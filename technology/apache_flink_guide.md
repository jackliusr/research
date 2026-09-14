# Apache Flink: The Stream's State — Runtime, Checkpointing and Operational Reality

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Architecture — stream processing engines, stateful runtime, regulated-enterprise operations
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Facts checked against primary sources:** 14 September 2026 (UTC). Every version, date and status claim is dated and sourced; the ones I could not verify are listed in §13.

---

## Table of Contents

1. [Overview and Version History](#1-overview-and-version-history)
2. [Runtime Architecture](#2-runtime-architecture)
3. [The State Machinery](#3-the-state-machinery)
4. [Exactly-Once and the End-to-End Guarantee](#4-exactly-once-and-the-end-to-end-guarantee)
5. [The APIs](#5-the-apis)
6. [Connectors and the Table Ecosystem](#6-connectors-and-the-table-ecosystem)
7. [Deployment and Operations](#7-deployment-and-operations)
8. [The Known Hard Parts and the Critiques](#8-the-known-hard-parts-and-the-critiques)
9. [The Comparison, Cross-Referenced Not Re-Derived](#9-the-comparison-cross-referenced-not-re-derived)
10. [The Banking and Regulated-Estate Angle](#10-the-banking-and-regulated-estate-angle)
11. [Cymbal Bank Worked Example](#11-cymbal-bank-worked-example)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)

---

## Scope: What This Guide Owns, and What Its Siblings Own

This is the **engine** guide: how Apache Flink's runtime, state store, checkpoint machinery, APIs and connector layer actually behave, what the official documentation guarantees, where the guarantees stop, and what an operator inherits by adopting it.

It deliberately does **not** re-explain the discipline. `technology/event_stream_processing_guide.md` (1218 lines) owns: §1 what event stream processing is; §2 paradigm evolution; §3 core concepts; §4 ESP patterns; **§5 the cross-technology comparison — §5.2 is the Flink positioning entry and §5.7 the comparison table**; §6 time semantics and windowing (6.4 watermarks, 6.5 late data); §7 state management and fault tolerance (7.1 state types, 7.2 state backends, 7.3 exactly-once, 7.4 checkpointing, 7.5 savepoints vs checkpoints, 7.6 consistency trade-offs, 7.7 state at scale); §8 ESP in the event-driven architecture; §9 ESP for banking; §10 CEP in banking. `technology/complex_event_processing_guide.md` owns CEP pattern theory; `banking/financial_fraud_detection_at_scale_guide.md` owns fraud/AML theory; `banking/kafka_guide.md` owns Kafka platform mechanics; `technology/cloud_object_storage_lakehouse_guide.md` and `technology/data/paimon_iceberg_delta_comparison.md` own the storage and table-format architecture.

What this guide owns instead is the implementation, and the difference matters: that Flink's default state backend is the in-memory `HashMapStateBackend`; that `ForStStateBackend` ships but is documented as *experimental and not fully available for production*; that an unaligned checkpoint cannot survive a Flink minor-version upgrade while a canonical savepoint can; that `SourceFunction`, `SinkFunction` and the `DataSet` API were **removed** in 2.0 rather than deprecated. Where a concept is needed, this guide names the sibling section rather than restating it — for example, watermark theory is the sibling's §6.4; what appears here is what Flink's idleness detection, watermark alignment and per-split watermark metrics actually do, including the 2.0 fix for back-pressured sources wrongly marked idle.

---

## 1. Overview and Version History

### 1.1 What Flink Is

The official documentation still describes Flink in the same terms it has used for a decade: a *framework and distributed processing engine for stateful computations over unbounded and bounded data streams*, designed to run in all common cluster environments and to perform computations at in-memory speed and at any scale (flink.apache.org docs, checked 2026-09-14). Three properties distinguish it from its nearest alternatives, and all three are engine properties rather than marketing ones: **streaming is the primitive** and batch is the bounded special case, with no micro-batch scheduler underneath; **state is first-class and checkpointed**, so keyed state, operator state, timers and stream positions are snapshotted together and recovery can be *consistent* rather than merely resumable (mechanics in §3, theory in the sibling's §7); and **the consistency guarantee is bounded by the sinks** — Flink can guarantee exactly-once state and, with the right sinks, exactly-once output, but nothing more than the weakest sink in the pipeline allows (§4). Positioning against Kafka Streams, Spark Structured Streaming, Beam, ksqlDB and RisingWave is the sibling's §5.2 and §5.7; §9 below keeps only a short positioning table for readers arriving here first.

### 1.2 Origin and Governance

Flink did not begin as Flink. It began as **Stratosphere**, a research project the Apache Software Foundation's own top-level-project announcement dates to **2009 at TU Berlin**, together with the Berlin and later European data-management community (HU Berlin, Hasso Plattner Institute, KTH Stockholm, ELTE Budapest and others). TU Berlin's research page confirms the project was funded by the **Deutsche Forschungsgemeinschaft (DFG)**. Releases up to 0.5 shipped under the *Stratosphere* name; **Flink 0.6-incubating (2014-08-26)** was, in the project's own words, "the first release of the system inside the Apache Incubator and under the name Flink." **Flagged:** secondary sources frequently say Stratosphere "started in 2010"; the ASF says 2009. This guide follows the ASF and records the discrepancy rather than settling it.

| Milestone | Date | Source |
|---|---|---|
| Stratosphere begins at TU Berlin (DFG-funded) | 2009 | ASF TLP announcement 2015-01-12; TU Berlin DIMA research page |
| Last Stratosphere-named release (0.5.x) | before 2014-08 | Flink 0.6 release note |
| **Enters Apache Incubator** | **2014-04-14** | Apache Incubator status page for Flink |
| Flink 0.6-incubating — first release under the Flink name | 2014-08-26 | flink.apache.org/2014/08/26/apache-flink-0.6-available/ |
| **Graduates from the Apache Incubator** | **2014-12-17** | Apache Incubator status page |
| ASF announces Flink as a Top-Level Project (TLP) | 2015-01-12 | news.apache.org announcement; Stephan Ewen named VP of Apache Flink |
| Flink 1.0.0 / 2.0.0 / 2.3.0 (current stable) | 2016-03-08 / 2025-03-24 / 2026-06-25 | downloads archive; release announcements |

Two governance notes explain the current project shape. The company founded by Flink committers, **data Artisans**, was acquired by Alibaba in 2019 and became **Ververica**, which remains an active commercial vendor and the origin of the **ForSt** project that Flink 2.0's disaggregated state backend is based on (`ForStStateBackend` is documented as "based on ForSt project", linking to `github.com/ververica/ForSt`). And Flink SQL's table format lineage runs through **Apache Flink Table Store** — still on the downloads archive at 0.1.0 (2022-05-11), 0.2.0 (2022-08-29) and 0.3.0 (2023-01-13) — which became **Apache Paimon**, now an independent ASF project with its own release train (§6.3).

### 1.3 Version Table

Dates below are read from the flink.apache.org downloads/archive listing, which is generated from the release archive itself; feature notes come from the release announcements and the per-version release notes under `nightlies.apache.org/flink/flink-docs-release-2.3/release-notes/`.

| Version | Date | Headline content (as documented) |
|---|---|---|
| 0.6-incubating | 2014-08-26 | first Apache release under the Flink name |
| 1.0.0 | 2016-03-08 | first 1.x; the API era that lasted nine years |
| 1.1.0 / 1.5.0 / 1.9.0 | 2016-08-08 / 2018-05-25 / 2019-08-22 | 1.9 opens the Table API/SQL consolidation era |
| 1.10.0 / 1.12.0 | 2020-02-11 / 2020-12-08 | |
| 1.13.0 | 2021-04-30 | **savepoint binary format unified across backends** — why a savepoint can now change backend |
| 1.14.0 | 2021-09-29 | buffer debloating introduced |
| 1.15.0 | 2022-05-05 | native-format savepoints; intermediate savepoints stop being used for recovery |
| 1.16.0 / 1.17.0 / 1.18.0 / 1.19.0 | 2022-10-28 / 2023-03-23 / 2023-10-25 / 2024-03-18 | |
| 1.20.0 | 2024-08-02 | final 1.x line; still maintained in 2026 (1.20.5, 2026-06-08) |
| **2.0.0** | **2025-03-24** | disaggregated state management (FLINK-32070), asynchronous execution model, ForSt backend, materialized tables, deprecated-API removal, deep Paimon integration |
| 2.0.1 | 2025-11-10 | first 2.0 bug-fix release; 51 fixes |
| 2.0.2 | 2026-05-11 | 34 bug and vulnerability fixes |
| **2.1.0** | **2025-07-31** | 116 contributors, 16 FLIPs, 220+ issues; `ML_PREDICT`; `VARIANT` type; Delta Join on by default; opt-in multi-join operator |
| 2.1.1 / 2.1.2 / 2.1.3 | 2025-11-10 (25 fixes) / 2026-05-11 (45 fixes) / 2026-06-14 (5 fixes) | |
| **2.2.0** | **2025-12-04** | 73 contributors, 9 FLIPs, 220+ issues; `VECTOR_SEARCH`; materialized-table flexibility; balanced task scheduling; protobuf 4.x |
| 2.2.1 | 2026-05-15 | 44 fixes |
| **2.3.0** | **2026-06-25** | 15 FLIPs; `FROM_CHANGELOG`/`TO_CHANGELOG`; materialized-table `START_MODE`; `SinkUpsertMaterializer` rework; experimental native S3 filesystem |

Also current on the downloads page on 2026-09-14: **2.3.0, 2.2.1, 2.1.3, 1.20.5** — one current line, one previous line, an older 2.1 line and the final 1.x line maintained simultaneously. That fan-out is itself an operational fact: a bank choosing "the N-1 release" has a supported target, but a long-lived 1.20 pipeline sits on a line that now receives bug fixes only.

### 1.4 What Actually Changed in 2.0

Flink 2.0 is the only release the community itself describes as a break: "the first major release since Flink 1.0 launched nine years ago", the product of "two years of meticulous preparation", with 165 contributors, 25 FLIPs and 369 issues. Five changes matter to an operator.

1. **Disaggregated state management (FLINK-32070).** Remote storage (DFS/object store) becomes *primary* state storage rather than only the checkpoint target. The stated motivation is a cloud-native argument, not a performance argument: local-disk constraints in containerization; spiky resource usage caused by compaction; fast rescaling for jobs with hundreds of TB of state; and light, fast checkpointing natively.
2. **An asynchronous execution model.** The release notes are candid that extending the state store to remote DFS is "insufficient due to Flink's existing blocking execution model", so async state access was introduced (out-of-order record processing, async state APIs) while preserving watermark propagation, timer handling and key ordering semantics.
3. **ForSt** ("For Streaming") — the purpose-built disaggregated backend, an LSM-tree store built on RocksDB that can hold its SST files on HDFS, S3 and other Flink-supported filesystems. **The 2.3 documentation still labels it experimental and not fully available for production** (§3.2). Seven critical SQL operators (joins, group/window aggregations) were re-implemented against the async state APIs, gated by `table.exec.async-state.enabled`; 11 of 14 stateful Nexmark queries were compatible and the rest of the stateful operator set is still being migrated.
4. **Materialized tables** — engine-refreshed, declaratively managed tables uniting real-time and historical views; 2.1, 2.2 and 2.3 each extend them (DDL parity, `FRESHNESS` becoming optional, `START_MODE`, `DISTRIBUTED BY`).
5. **A deprecated-API removal pass with backward-incompatible consequences** — the part that breaks upgrades; quantified in §5.1 and §6.1.

**Benchmark caveat.** The 2.0 announcement publishes Nexmark numbers: heavy-I/O stateful queries reaching 75–120% of traditional local-state throughput with a 1 GB cache (~50% with no cache), and small-state queries trailing local state by ≤10% on average. Those are project-published community/vendor benchmarks, not independent measurements — an order of magnitude, not a capacity plan.

---

## 2. Runtime Architecture

### 2.1 The Components

Flink's runtime is small in concept and fiddly in practice. The **JobManager** is the coordinator, and in modern versions the historical monolith is explicitly split into three roles: the **Dispatcher** (accepts job submissions, starts a JobMaster per job, exposes the REST endpoint and Web UI), the **ResourceManager** (owns task slots and requests containers from the underlying resource provider) and the **JobMaster** (one per running job; holds the ExecutionGraph, schedules tasks, coordinates checkpoints, applies the failover strategy). The production-readiness checklist is blunt about the consequence: "The JobManager serves as a central coordinator for each Flink deployment... it is a single point of failure within the cluster, and if it crashes, no new jobs can be submitted, and running applications will fail." HA — ZooKeeper-based or Flink's Kubernetes-based service — is "highly recommended for production setups", which in a regulated estate should be read as mandatory.

**TaskManagers** are the workers. Each runs one or more **task slots**, and a slot is the unit of resource scheduling: memory divides into framework heap, task heap, network memory, managed memory (the pool RocksDB and ForSt buffers and caches draw from) and JVM overhead. There is **no default** for total memory — one of `taskmanager.memory.process.size`, `taskmanager.memory.flink.size`, or the explicit heap-plus-managed pair must be configured or the process refuses to start. Slot sharing lets subtasks of different operators in the same slot-sharing group occupy one slot; `taskmanager.numberOfTaskSlots` and `parallelism.default` are the two knobs most often mismatched during sizing. The **client** builds the JobGraph and submits it. Since 2.0 the configuration file is `config.yaml` (YAML 1.2); `flink-conf.yaml` is no longer supported and `bin/migrate-config-file.sh` converts it.

```
                    +-------------------------------------------------+
   client / CLI --->|                 JobManager process            |
   REST / WebUI     |  Dispatcher     ResourceManager                 |
                    |   - receives jobs   - owns task slots           |
                    |   - starts JobMaster - requests containers      |
                    |  JobMaster (one per job)                        |
                    |   - holds the ExecutionGraph                    |
                    |   - schedules tasks, coordinates checkpoints    |
                    |   - triggers restarts / failover                |
                    +--------------------+----------------------------+
                                         |  RPC (task deployment, slots)
              +--------------------------+--------------------------+
     +--------v--------+        +--------v--------+        +--------v--------+
     |  TaskManager A  |        |  TaskManager B  |        |  TaskManager C  |
     |  [slot 1][slot 2]|       |  [slot 1][slot 2]|       |  [slot 1][slot 2]|
     |  memory / network / local state dirs (cache for ForSt)            |
     +-----------------+        +-----------------+        +-----------------+
```

### 2.2 Job Graph, Execution Graph, Physical Graph

The three-graph progression is where most "why is my job doing that" questions resolve. The **StreamGraph** is the logical topology the API layer produced. The **JobGraph** is the optimiser's and client's version, after operator **chaining**, and it is the unit of submission and the place where operator **UIDs** attach. The **ExecutionGraph**, built by the JobMaster, is the parallelism-expanded form — tasks, result partitions, intermediate result partitions — and it is the unit of scheduling, checkpointing and failover. Two consequences carry into operations: chaining decides whether two operators share a thread, and therefore whether back-pressure between them is even separately observable; and the JobGraph is where `uid()` is set, which is what makes state restorable. The checklist puts it plainly — auto-generated UIDs are "very fragile, as changes to the JobGraph (e.g., exchanging an operator) results in new UUIDs" — so stable, explicit UIDs are not optional in production.

### 2.3 Deployment Modes

**Session** mode runs one long-lived cluster hosting many jobs: fast submission, at the cost of shared libraries and one bad job starving its neighbours. **Per-job** mode dedicates a cluster to a single job for isolation, at higher startup cost, and was historically bound to YARN. **Application** mode runs one application per cluster with `main()` executed on the cluster; it is the production default and what the Kubernetes Operator orchestrates. The 2.0 cleanup removed the CLI shortcut `flink-client run-application`, replaced by `run -t kubernetes-application` — small, but exactly the class of change that breaks a hardened deployment runbook.

### 2.4 Schedulers and Failover Strategies

Two separate mechanisms are routinely confused, and both live in `config.yaml`. The **restart strategy** decides *when* to restart. Verified current values of `restart-strategy.type`: `disable`/`off`/`none`, `fixed-delay`, `exponential-delay` and `failure-rate` — with **no default** (the reference shows `(none)`), and per-strategy defaults of `fixed-delay.attempts` = 1 and `fixed-delay.delay` = 1 s; `exponential-delay.initial-backoff` = 1 s, `max-backoff` = 1 min, `backoff-multiplier` = 1.5, `jitter-factor` = 0.1, `reset-backoff-threshold` = 1 h; `failure-rate.max-failures-per-interval` = 1 per 1 min interval.

The **failover strategy** decides *which tasks* are restarted. `jobmanager.execution.failover-strategy` is documented with a default of `"region"` and two accepted values: `full` restarts all tasks (simple, predictable, most expensive — one bad subtask forces a whole-pipeline replay from the last checkpoint); `region` restarts only tasks that could be affected, computed from the ExecutionGraph's *pipelined regions*, so recovery is cheaper but depends on the topology actually having separable regions.

**Flagged:** older releases also documented `restart-pipelined-region` (and earlier, `restart-individual`) as failover-strategy values; both appear to have been consolidated into `region`, but their removal could not be re-verified in the 2.3 documentation in this pass, and whether the **AdaptiveScheduler** exposes region failover as its own mode is likewise **unverified**. What *is* verified for the AdaptiveScheduler in 2.0: an efficiency change letting it synchronise checkpointing and rescaling to minimise reprocessing (FLINK-35549); a new `jobmanager.adaptive-scheduler.executing.resource-stabilization-timeout`; the deprecation of `jobmanager.adaptive-scheduler.min-parallelism-increase`; and a behaviour change — it now respects `execution.state-recovery.from-local`, which **defaults to false**, so task-local recovery must be explicitly opted into (FLINK-36201).

### 2.5 Back-Pressure: How an Operator Actually Sees It

Flink's data exchange is credit-based, and the practical consequence is that back-pressure is not an exception but a state that propagates upstream and appears in three places. First, the **Web UI back-pressure tab**, which samples stack traces; the sampling is configurable through the `web.backpressure.*` keys (`num-samples`, `delay-between-samples`, `refresh-interval`, `cleanup-interval`), which still exist in the current configuration reference. Second, **per-task metrics** — the per-second busy/back-pressured/idle family, including `backPressuredTimeMsPerSecond` and `idleTimeMsPerSecond` — which is the machine-readable version of the same signal and what an alerting rule should use, never the UI. Third, **checkpoint behaviour**: when a job is permanently back-pressured, checkpoint barriers travel slowly, so checkpoint *start delay* and *alignment duration* rise even though nothing is wrong with the state store. The tuning guide names those two numbers and warns that a persistently high trigger time "typically indicates that the system is operating under a constant backpressure."

2.1 added **split-level watermark metrics** — `currentWatermark`, `activeTimeMsPerSecond`, `pausedTimeMsPerSecond`, `idleTimeMsPerSecond`, plus accumulated per-split counters (FLINK-37410) — precisely so that back-pressure, watermark-alignment pausing and idleness can be told apart per split rather than inferred from one indistinguishable symptom. Related: 2.0 fixed idleness detection so that a source or split that is back-pressured or blocked by watermark alignment is no longer wrongly switched to idle, which had produced incorrectly calculated watermarks and erroneous late data (FLINK-35886). 2.3 adds adaptive partition selection for back-pressure handling (a 2.3 release-note item; see §1.3 and §12).

---

## 3. The State Machinery

The concepts — keyed versus operator state, what exactly-once means, checkpointing theory — are the sibling's §7.1–§7.4. What follows is what Flink's runtime actually does and what each mechanism costs to operate.

### 3.1 State Primitives, and the One-Way Door

Flink distinguishes **keyed state** (scoped to a key in a keyed stream: `ValueState`, `ListState`, `MapState`, `ReducingState`, `AggregatingState`) from **operator state** (scoped to a subtask: `ListState`, `BroadcastState`, and the union/redistribution variants used to make operator state rescalable). Timers — the mechanism behind windows, session windows and `ProcessFunction` — are stored in the state backend by default rather than in a separate timer service, which is why they show up in checkpoint size. The design decision to internalise before any sizing conversation: state is scoped by operator UID and the key space is partitioned into **key groups**, whose number is fixed by **maximum parallelism**, set per operator. The production-readiness checklist states the consequence in a way nobody should discover in production: **"There is currently no way to change the maximum parallelism of an operator after a job has started without discarding that operator's state."** The documented bounds are `0 < parallelism <= max parallelism <= 2^15`, with the default derived as `128` for parallelism ≤ 128 and `MIN(nextPowerOfTwo(parallelism + parallelism/2), 2^15)` above that. Flink keeps metadata proportional to max parallelism in order to make rescaling possible, so "set it to 32768 just in case" trades state metadata and per-key overhead for headroom that may never be used. This is the most common irreversible mistake in Flink adoption.

### 3.2 The Current State Backend Set (verified 2026-09-14)

| Backend | Where state lives | Incremental checkpoints | Async state access | Documented status | Cost |
|---|---|---|---|---|---|
| **`HashMapStateBackend`** | Java heap, as objects | **No** | No | **the default**; "encouraged for jobs with large state, long windows, large key/value states" | fastest access, no (de)serialisation; bounded by heap; object reuse is **unsafe**; docs recommend setting managed memory to zero |
| **`EmbeddedRocksDBStateBackend`** | RocksDB, by default on TaskManager local data dirs, as serialised byte arrays | **Yes — the only backend offering incremental checkpoints** | No | "encouraged for jobs with very large state" | documented as "an order of magnitude slower than the memory state backends"; every access pays (de)serialisation; key comparison is byte-wise rather than Java `equals`/`hashCode`; per key and per value capped at 2^31 bytes by the RocksDB JNI bridge, with merge operations (e.g. `ListState`) able to silently exceed it and fail on next retrieval |
| **`ForStStateBackend`** | LSM on RocksDB, but **SST files on remote filesystems** (HDFS/S3); local disk holds only a file cache | Always incremental (only mode) | **Yes — the only backend supporting asynchronous state access**, exploited via State API V2 | **"still in the experimental stage and is not fully available for production"** | unbounded state size, light checkpoints and fast recovery, at the price of network latency on state access; **no canonical savepoint, no full snapshot, no changelog and no file-merging checkpoints** |

The documented decision rule is performance versus scalability: `HashMapStateBackend` is "very fast... however, state size is limited by available memory"; RocksDB "can scale based on available disk space" but is slower; and if state exceeds available disk, or a fast rescale under cloud-native conditions is required, `ForStStateBackend` is the recommendation. Three further documented details matter more than they look. **Legacy names are aliases, not separate backends**: `MemoryStateBackend` ≡ `HashMapStateBackend` + `JobManagerCheckpointStorage`; `FsStateBackend` ≡ `HashMapStateBackend` + `FileSystemCheckpointStorage`; `RocksDBStateBackend` ≡ `EmbeddedRocksDBStateBackend` + `FileSystemCheckpointStorage`. The documented rationale (Flink 1.13) is that the rework "does not affect the runtime implementation or characteristics... it is simply to communicate intent better" — the old names were always two orthogonal choices wearing one label, so a runbook saying "we use FsStateBackend" really means "heap state, checkpoints to a filesystem." **The savepoint format was unified in 1.13**, so a savepoint taken with one backend can be restored with another — provided you upgrade first, take a savepoint with the new version, and only then switch backends. And **managed memory is the RocksDB budget**: `state.backend.rocksdb.memory.managed` defaults to true, drawing write buffers and block cache from the managed-memory fraction (default 0.4), which is why the tuning guide's first recommendation is to raise managed memory before touching low-level RocksDB options.

### 3.3 Checkpoint Storage

Orthogonal to the state backend, **checkpoint storage** decides where a snapshot is written. `JobManagerCheckpointStorage` keeps snapshots in the JobManager heap, is limited to **5 MB per individual state by default**, cannot exceed the Pekko frame size, and requires the aggregate state to fit in JM memory — documented for "local development and debugging" and very small state. `FileSystemCheckpointStorage` writes snapshots to a filesystem or object-store URL (`hdfs://`, `s3://`, `file:///`), keeps only minimal metadata in JM memory, and is documented for "all high-availability setups". If a checkpoint directory is configured, filesystem storage is used; otherwise JobManager storage. The on-disk layout is `/{job-id}/` with `shared/`, `taskowned/` and per-checkpoint `chk-N/` directories — and the documentation warns explicitly that this layout "is not part of a public API and can be changed in the future release", so nothing outside Flink should parse it. Retention is controlled by `ExternalizedCheckpointRetention`: `RETAIN_ON_CANCELLATION` (keep it on cancel, clean up yourself) or `DELETE_ON_CANCELLATION` (state available only if the job fails). The default is **not retained** — checkpoints are deleted when a program is cancelled, so a DR plan that assumes they persist without setting retention has a hole in it.

### 3.4 Alignment, Unaligned Checkpoints and Buffer Debloating

In an **aligned** exactly-once checkpoint, barriers are injected at the sources and flow in-band; a subtask that has received a barrier on one input channel blocks that channel until every input has delivered theirs, then snapshots. That blocking window is the alignment time. In an **unaligned** checkpoint (available since Flink 1.11), the snapshot also contains the in-flight buffered records, so barriers may overtake those buffers and checkpoint duration becomes largely independent of throughput. The documentation is honest about the limits of that fix, and every one of them matters to an operator: unaligned checkpoints **do not solve back-pressure** ("this does not solve the underlying problem that's causing the backpressure in the first place, and end-to-end records latency will remain high"); there are **no concurrent unaligned checkpoints**, and a savepoint cannot be taken concurrently with one; they are **incompatible with task-local recovery**; **watermark semantics differ on recovery**, because Flink regenerates watermarks as the first step of recovery and with unaligned checkpoints generates them *after* restoring in-flight data, so an operator applying the latest watermark to each record can produce different results than under aligned checkpoints (workaround: store the watermark per key group in union state); **rescaling of unaligned checkpoints is restricted for pointwise and broadcast connections**, since forward-channel records carry no key context and broadcast channels give no guarantee that all subtasks apply the same broadcast events in the same order; and a single long-running record — a fan-out `flatMap`, a huge serialised record, a burst of timers — cannot be interrupted, so it can still delay an unaligned checkpoint.

The latency lever alongside this is **buffer debloating** (`taskmanager.network.memory.buffer-debloat.enabled`, introduced in Flink 1.14), which automatically bounds in-flight data; it helps both aligned and unaligned checkpoints but is most visible with aligned ones. `execution.checkpointing.aligned-checkpoint-timeout` offers a middle path: a checkpoint starts aligned and is promoted to unaligned if it exceeds the timeout.

### 3.5 The Tuning Knobs That Actually Get Used

`execution.checkpointing.interval` is the base period and should be set deliberately — the checklist frames it as an SLA question ("how much data can you tolerate reprocessing?"). `execution.checkpointing.min-pause` sets the minimum gap between the end of one checkpoint and the start of the next, and is the fix for a job that spends all its time snapshotting when checkpoints exceed the interval. `execution.checkpointing.timeout` should be raised only with the back-pressure cause understood. `execution.checkpointing.unaligned` is for back-pressure-dominated checkpoint durations and only when state/sink I/O is not already the bottleneck. `state.backend.incremental` is the first thing to try on large state (RocksDB/ForSt only). `execution.checkpointing.storage` and `.dir` move snapshots to an object store. `state.checkpoints.num-retained` keeps extra successful checkpoints, which matters when a corrupted newest checkpoint would otherwise force a long replay. `state.backend.local-recovery` enables task-local recovery — **default false**, and re-defaulted to false for the AdaptiveScheduler in 2.0. Note also that checkpoints may be configured to overlap: applications can allow multiple concurrent checkpoints, but with large state that ties up too many resources, and a manually triggered savepoint may run concurrently with an ongoing checkpoint.

### 3.6 Savepoints: What They Enable, and What They Cannot Survive

A savepoint is a consistent image of the execution state created through the checkpointing mechanism, but owned by the user rather than by Flink. Two formats exist: **canonical**, the backend-independent unified format (unified in 1.13), the most portable and the slowest to take and restore; and **native**, a snapshot in the state backend's own format (for RocksDB, SST files), available since Flink 1.15, much faster and correspondingly less portable. The documented capability matrix is the clearest statement in the whole documentation of what a savepoint survives, and it belongs in every upgrade runbook:

| Operation | Canonical savepoint | Native savepoint | Aligned checkpoint | Unaligned checkpoint |
|---|---|---|---|---|
| Change state backend | ✓ | ✗ | ✗ | ✗ |
| State Processor API (write) | ✓ | ✗ | ✗ | ✗ |
| State Processor API (read) | ✓ | ! | ! | ✗ |
| Self-contained and relocatable | ✓ | ✓ | ✗ | ✗ |
| Schema evolution of state type | ✓ | ! | ! | ! |
| Arbitrary job upgrade | ✓ | ✓ | ✓ | ✗ |
| Non-arbitrary job upgrade | ✓ | ✓ | ✓ | ✓ |
| Flink minor-version upgrade | ✓ | ✓ | ✓ | ✗ |
| Flink bug/patch version upgrade | ✓ | ✓ | ✓ | ✓ |
| Rescaling (change parallelism) | ✓ | ✓ | ✓ | ✓ |

(✓ fully supported; ✗ not supported; ! works today but Flink does not officially guarantee it.) The load-bearing row for a regulated estate is the minor-version upgrade: **a canonical savepoint survives a 1.x → 1.y upgrade; an unaligned checkpoint does not.** The rules that go with the matrix: **assign UIDs to every stateful operator**, because a savepoint is literally a map of `Operator ID -> State` and auto-generated IDs "depend on the structure of your program and are sensitive to program changes"; **deleting a stateful operator fails the restore** unless `--allowNonRestoredState` (`-n`) is passed, which deliberately discards that state and is therefore a decision rather than a fix; **adding a stateful operator is safe** (it starts empty); **changing parallelism is safe**, with key groups redistributed; **savepoints are relocatable** except with S3 entropy injection enabled (absolute path references) or with task-owned state such as the `GenericWriteAheadLog` sink; and since Flink 1.15 **intermediate savepoints are not used for recovery and commit no side effects**, with a documented hazard when several jobs share a checkpointing timeline — resuming one job from a savepoint while another fell back to an earlier checkpoint can commit transactions that never happened, and the advised mitigation is to change the UIDs of transactional sinks. With `state.backend.type: jobmanager`, metadata *and* state live in the `_metadata` file, so a savepoint directory with no data files is expected rather than broken.

### 3.7 Where Large State Actually Hurts

State size is not a storage problem; it is a *time* problem that appears in four places. **Checkpoint duration** — the snapshot must be produced and shipped, with incremental checkpointing as the primary mitigation, and 2.0 adding native file copy via `s5cmd` for S3 recovery downloads, documented as "at least a factor of 2" faster (FLINK-35739). **Recovery time** — the snapshot must be pulled back, which on a pod restarted in another availability zone is the difference between a blip and an outage. **Rescaling** — the re-partitioning of key groups across the new parallelism, which is the stated fourth motivation for Flink 2.0's disaggregated state work ("fast rescaling for jobs with large states (hundreds of Terabytes)"). And **compaction spikes** — the second stated motivation, "spiky resource usage caused by compaction in the current state model", familiar to anyone who has watched a RocksDB-backed job's CPU and latency tail correlate with compaction. Two controls deserve naming: **state TTL** (`StateTtlConfig`) for keyed state, the only way to stop an unboundedly growing per-key state from becoming an incident; and the **`SinkUpsertMaterializer`** problem, where the old implementation "kept the full history of updates for the primary key, giving unbounded state growth and poor performance", reworked in 2.3 (FLIP-558) so that an upsert key differing from the sink primary key now **fails at planning time** unless you write `ON CONFLICT DO NOTHING | DO ERROR | DO DEDUPLICATE` explicitly. Making that failure explicit turns a slow-motion state explosion into a planning error, which is the improvement.

### 3.8 The State Model in One Picture

```
   event stream  ->  [ source ]  ->  [ keyBy ]  ->  [ operator with keyed state ]
                                                        |
                          +-----------------------------+------------------------------+
                          |                                                            |
              managed by the STATE BACKEND (runtime)                snapshotted by CHECKPOINTING
                          |                                                            |
     heap objects (HashMapStateBackend)              checkpoint STORAGE (JM heap, or filesystem /
     RocksDB local disk (EmbeddedRocksDB)              object store)
     remote SST files (ForSt, experimental)                 |
                          |                                  |
                 keyed state + operator state          full snapshot or incremental delta
                 + timers + in-flight buffers          (RocksDB / ForSt)
                                                             |
                                          USED FOR: automatic failure recovery
                                          FORMAT:   aligned / unaligned (see the §3.6 matrix)
                                          ALSO:     savepoints, on demand, in canonical or
                                                    native format — the upgrade mechanism
```

The two columns are the point. **The state backend is a runtime decision** — it decides what state access costs, whether incremental checkpointing is available, and whether asynchronous state access is possible at all. **Checkpoint storage is a durability decision** — it decides where snapshots go and, through retention settings, whether they outlive the job. Getting one right and the other wrong is the most common configuration mistake, and it is entirely possible to get both right in an unusual combination: `HashMapStateBackend` (heap state) with filesystem checkpoint storage is a reasonable small-state configuration, and it is what the legacy `FsStateBackend` name always meant.

---

## 4. Exactly-Once and the End-to-End Guarantee

The concept — what exactly-once means, and why it is achievable only as a combination of replayable sources, checkpointed state and transactional or idempotent sinks — is the sibling's §7.3 and §7.6. This section is about what Flink's implementation delivers and where the guarantee stops.

### 4.1 What Checkpointing Alone Guarantees

Flink's checkpointing provides *consistent state recovery*: on restart, operators resume with the state of the last completed checkpoint and sources resume from the offsets recorded in that same snapshot. The documentation's summary is precise enough to quote — state and the corresponding stream positions are recovered, "thereby giving the application the same semantics as a failure-free execution." That sentence describes the pipeline's *internal* view. It says nothing about the outside world and cannot: the moment data leaves Flink's managed state, Flink is no longer the authority on whether it appeared once, twice or not at all.

### 4.2 How the Sink Half Works

For output to be exactly-once, the external system must participate. Three documented mechanisms exist: **two-phase commit**, where the sink stages output and holds a transaction open, then commits when notified that the checkpoint completed (Flink's `CheckpointListener` is the hook, and the sink must be able to abort on recovery); **write-ahead-log style staging**, where output is written to a temporary location and atomically moved into place on checkpoint completion (the pattern behind the filesystem sink); and **idempotent writes**, where the same write can be applied twice with the same effect — an upsert on a primary key, a deterministic-key delete-and-insert, or a compaction-aware table format. With idempotence, Flink only needs at-least-once delivery and the destination supplies deduplication. One documented property belongs in every architectural review: **exactly-once sinks make results visible only on checkpoint completion.** The production-readiness checklist says so directly of Kafka and FileSink, framing it as a delivery-latency decision — "Shorter checkpoint intervals make results available more quickly but may also put additional pressure on these systems." Exactly-once output is therefore purchased with latency of roughly one checkpoint interval, which for a 150 ms fraud decision path is usually an unacceptable trade; the correct design is a low-latency at-least-once alerting path alongside an exactly-once audit or ledger path.

### 4.3 The Guarantee Matrix — What Is Actually Documented

| Connector | Documented guarantee | Verification |
|---|---|---|
| **Kafka sink** | Flink "provides an Apache Kafka connector for reading data from and writing data to Kafka topics **with exactly-once guarantees**", implemented via Kafka transactions (`transactional.id`, `ProducerFencedException` handling documented) | **Verified** — 2.3 Kafka connector docs |
| **Filesystem sink** (`FileSink`) | Named as an exactly-once sink whose results "only make results visible on checkpoint completion" | **Verified** — production-readiness checklist |
| **Paimon** | Deep Flink integration; 2.0 shipped "deep Apache Paimon integration" / "Streaming Lakehouse", CDC 3.5.0 bumped Paimon to 1.2.0 | Relationship **verified**; the per-version exactly-once statement for Paimon 2.x was **not** re-verified — see §13 |
| **Iceberg / Hudi** | Both are supported table formats; CDC 3.6.0 added an Oracle Source and a **Hudi Sink** pipeline connector | Hudi sink **verified**; guarantee level per format **not** verified |
| **JDBC** (`flink-connector-jdbc`) | Official, separate from the engine repo; 4.1.0 targets Flink 2.1.x/2.2.x | Release/compat **verified**; the **documented delivery guarantee was not verified** — do not assume exactly-once |
| **Elasticsearch** | 4.0.0 targets Flink 2.0.x | Release/compat **verified**; guarantee level not verified |
| **Kafka source** | Offset commit on checkpoint (`commit.offsets.on.checkpoint`), replay from committed or timestamp offsets, dynamic partition discovery | **Verified** |

The two unverified rows are the practically important ones, because JDBC is exactly what a bank reaches for when a pipeline must write to a core system's database. The safe engineering position, and the one this guide takes: **treat a sink as at-least-once unless its own documentation states exactly-once, and design the business logic to tolerate a duplicate** — with an idempotent upsert on a deterministic business key, or a downstream deduplication step.

### 4.4 Where the Guarantee Stops

Three failures of the guarantee are worth naming, because each is a design error rather than a Flink limitation. A **non-replayable source** makes exactly-once impossible before Flink is involved: if the source cannot be re-read from the checkpointed position — a fire-and-forget socket, a REST feed with no history, a system that deletes acknowledged messages — Flink records the offset but the source must honour it. A **weaker sink in a multi-sink job** means a pipeline writing to both an exactly-once ledger and an at-least-once notification sink has one exactly-once output and one at-least-once output; that is a legitimate design, but the guarantee must be stated per sink, never per job. And **side effects inside operators** — an HTTP call, a metrics increment, a message published from a `ProcessFunction` — are not part of checkpointed state and *will* be re-executed on replay, so anything with an external effect belongs in a sink or must itself be idempotent. 2.3's `FROM_CHANGELOG` / `TO_CHANGELOG` operators (FLIP-564) are relevant here: converting retract/upsert streams into append-only form is what makes archival and audit sinks writable as pure appends.

### 4.5 Three Ways to Design Around an Imperfect Sink

Because the guarantee is bounded by the sink, the design work is usually about *the sink*, not the engine. Three patterns recur, in increasing order of strength. **Accept at-least-once and deduplicate downstream**: cheapest, works with any sink, and shifts the correctness burden to a component whose behaviour you must also test; the alert path in §11 uses this. **Make the write idempotent at the destination**: write an upsert on a deterministic business key (`decision_id`, `payment_id`), or a merge into a compaction-aware table format, so a replay overwrites the same row instead of adding one; this is the strongest pattern available when the destination is a legacy system that cannot participate in a transaction, and it requires the key to be genuinely deterministic across replays — including across a savepoint restore. **Use a transactional sink and accept the latency floor**: strongest where available, since output becomes visible exactly at checkpoint completion, which means the sink's visibility lag is the checkpoint interval by construction.

A fourth, easily overlooked option is to **move the side effect out of the pipeline entirely**: publish the decision to a durable, replayable log (which is a job for the Kafka tier, `banking/kafka_guide.md`) and let a separate, idempotent consumer perform the write at its own pace. That decouples the streaming engine's recovery semantics from the target system's transaction capabilities — and it means a Flink restart cannot corrupt the target, only re-deliver to a consumer that is built to handle it.

### 4.6 The Honest One-Line Statement

> **A Flink pipeline's end-to-end guarantee is exactly the minimum of three things: source replayability, state consistency, and sink transactionality or idempotence.** Flink supplies the middle term reliably. The first and the third are the architect's problem, and they are where real incidents come from.

---

## 5. The APIs

Flink ships several API layers that are not alternatives: they are different altitudes, with different maturity levels and, in two cases, different futures. Choosing wrongly at the start of a programme is expensive, so each is stated here with its status.

### 5.1 The API Surface in 2.3

The 2.3 documentation landing page lists **Flink SQL**, **Table API**, **DataStream API**, and **DataStream API (V2)** as reference entry points. That last entry is the consequential one.

| API | Position | Documented maturity (2.3, checked 2026-09-14) |
|---|---|---|
| **Flink SQL** | declarative, highest level; gained its own top-level docs section in the 2.3 restructure (FLIP-561) | production; the 2.x release train is SQL-heavy |
| **Table API** | programmatic relational API (Java/Python) | production; DDL parity with SQL |
| **DataStream API** | imperative, per-record; the long-standing default for custom logic | production; **not labelled deprecated** in the 2.3 docs |
| **DataStream API (V2)** | a new set of APIs intended to *gradually replace* the original | **"currently in the experimental stage and is not fully available for production"** (quoted from the V2 overview page) |

The V2 page splits the API into **fundamental primitives** (data stream, partitioning, process function, state, processing timer service, watermark) and **high-level extensions** (event timer service, window, join) — windows and joins become sugars on top of the primitives rather than being baked into the stream type. Sources are wired in with `DataStreamV2SourceUtils.wrapSource(...)` for FLIP-27 sources, sinks with `DataStreamV2SinkUtils.wrapSink(...)`, and the environment changes shape: `ExecutionEnvironment.getInstance()` rather than `StreamExecutionEnvironment`, with `NonKeyedPartitionStream` and keyed streams as distinct types so keyed state is only reachable where it is legal. **Guidance:** for anything intended to run for five years starting in 2026, write against SQL/Table where the logic allows and the *original* DataStream API where it does not. Do not start a greenfield production pipeline on V2 in this release cycle — the documentation says so itself. Watch it, prototype with it, do not commit.

**What 2.0 actually removed.** This is the section that matters for upgrade planning, and the removals are not deprecations — the classes are gone. The **`DataSet` API** was removed (migrate to DataStream or Table/SQL). The **Scala DataStream and DataSet API** was removed (migrate to the Java DataStream API); the 2.3.0 binary is still published as `flink-2.3.0-bin-scala_2.12.tgz`, which this guide reads as the bundled shaded Scala version rather than a supported Scala programming API — an interpretation, not a documented statement (see §13), and nothing in the 2.x documentation offers a Scala API. **`SourceFunction`, `SinkFunction` and Sink V1** were removed, with the instruction "please migrate to Source and Sink V2"; the release notes add that because these were removed, "existing connectors depending on these APIs will not work on the Flink 2.x series." The `addSource(SourceFunction)` and `addSink(SinkFunction)` overloads went with them, along with many other `@Public` symbols that the notes confirm had been deprecated for at least two minor releases, and `@PublicEvolving` symbols deprecated for at least one. That stated deprecation policy is a planning input rather than trivia: a bank can budget upgrades against release notes instead of against surprise, and it also means the 2.x series will keep changing underneath a pinned version — pinning is not a substitute for reading the release notes at each minor upgrade.

### 5.2 Stateful Functions (StateFun)

Stateful Functions is a separate Flink sub-project that models an application as independently addressable, stateful *functions* invoked by messages or timers — an actor-like abstraction over Flink's state and messaging, versioned separately from the engine (`flink-statefun-3.3.0` on the download archive). Its natural fit is the case where the *entity* (a customer, an account, a card) is the unit of state and events arrive from many directions. The engine-level fact to note is that StateFun depends on the same state and checkpoint machinery described in §3, so its recovery characteristics are Flink's. It is also an additional framework to learn and operate, and should be adopted deliberately rather than as a side effect.

### 5.3 Python

PyFlink is a first-class API for Table/SQL and a supported API for the DataStream model, and the release record shows investment rather than abandonment — which is the useful signal. 2.1 added Python 3.12 support and **removed** Python 3.8 (FLINK-37823, FLINK-37776). 2.2 added async functions to the Python DataStream API (FLINK-38190), explicitly motivated by querying external services such as a large model on a separate GPU cluster, with concurrency limiting and retry support. And the 2.3 documentation restructure merged the Python docs into the API sections (FLIP-561), documenting Python as an API rather than a sibling. Two caveats carry forward: the Python DataStream surface still lags Java — the Kafka connector's deserialiser customisation, for example, is documented as value-only in Python — and any Python pipeline inherits a Python/JVM serialisation boundary that is a real throughput consideration.

### 5.4 SQL versus Code — A Decision Rule

| Choose SQL/Table when | Choose DataStream when |
|---|---|
| the logic is joins, aggregations, windows and filters | you need per-record control flow, custom state machines or CEP-style matching |
| latency and (with Delta Join) state size are the main risks | you must interact with an external system from inside the operator |
| the team maintains more analysts than JVM engineers | you need the asynchronous I/O or custom-timer surface |
| you want the newest engine features first | you are pinned to a release where those SQL features had not landed |

The gap between SQL and code is narrowing but not closed (§8.5). The pattern that recurs in production estates is SQL for the pipeline shape and UDFs (Java UDFs, SQL UDFs, or Process Table Functions) for the parts that resist declarative expression.

### 5.5 The AI-adjacent SQL Surface (dated, not durable)

The 2.2 announcement describes Flink as advancing "Real-Time Data + AI", and the concrete items are worth recording with their dates because this surface is moving fastest of anything in the engine. **2.1**: AI model DDLs, letting models be defined as catalog objects and invoked like functions, plus the `ML_PREDICT` table-valued function for real-time inference in SQL, with a built-in OpenAI provider and an interface for custom providers — the release notes frame this as evolving Flink "from a real-time data processing engine to a unified realtime AI platform." **2.2**: `VECTOR_SEARCH` for streaming vector similarity search and real-time context retrieval in SQL, plus Table API support for model inference so models can be called from Java and Python pipelines. **2.3**: Process Table Function enhancements including late-data reaction in SQL and `ORDER BY` on table arguments, with improved PTF state access and broadcast state explicitly deferred. Two caveats. None of these are labelled experimental in the release notes, but they are new enough that their operational characteristics — external model latency inside an operator, back-pressure when the model endpoint slows — have not been through the production pressure the core operators have. And calling an external model service from inside a pipeline reintroduces exactly the side-effect problem of §4.4 unless the call is bounded, retried and idempotent, so any pipeline depending on this surface must include an external service in its latency budget and in its failure modes.

---

## 6. Connectors and the Table Ecosystem

### 6.1 The Connector Model and Its Evolution

| Era | Source interface | Sink interface | Status in 2.x |
|---|---|---|---|
| legacy | `SourceFunction` | `SinkFunction`, `Sink V1` | **removed in 2.0** — connectors depending on them "will not work on the Flink 2.x series" |
| current | **FLIP-27 `Source`** (split enumerator + split reader, bounded and unbounded capable, checkpointable per split) | **FLIP-143 `Sink V2`** (`Sink`, `SinkWriter`, `Committer` — the committer is what enables transactional sinks) | the supported interface set |

Three consequences follow. FLIP-27 sources expose **splits** as the unit of work and checkpoint state per split, which is why per-split watermark metrics became possible in 2.1 and why dynamic partition discovery works without a restart; FLIP-143 sinks separate writing from committing, which is what lets a sink implement two-phase commit and participate in exactly-once output (§4.2); and connectors are **released separately from the engine and lag it**.

| Connector | Latest as of 2026-09-14 | Flink compatibility (downloads page) | Note |
|---|---|---|---|
| Kafka | 5.0.0 (2026-06-02); 4.0.1; 3.4.0 | 1.20.x / 2.0.x / newest line | exactly-once documented for the sink; "universal" client tracking the latest Kafka client; Kafka clients are backwards compatible with brokers 2.1.0+ |
| AWS (Kinesis, Firehose, …) | 6.0.1; 5.1.0 | 2.0.x / 1.20.x | the newest AWS connector supporting the newest engine is the pattern to watch |
| Elasticsearch | 4.0.0; 3.1.0 | 2.0.x / 1.18–1.20.x | search/observability sink |
| JDBC | 4.1.0; 4.0.0; 3.4.0 | 2.1.x–2.2.x / 2.0.x / 1.20.x | dim-table lookups and result sinks; guarantee level not verified here (§4.3) |
| HBase | 4.0.0 | 1.18.x / 1.19.x | **no 2.x build listed** — an upgrade blocker |
| Hive | 3.0.0 | 1.20.x | catalog integration |
| Cassandra | 3.2.0 | 1.18.x / 1.19.x | **no 2.x build listed** |
| GCP PubSub | 3.1.0 | 1.18.x / 1.19.x | **no 2.x build listed** |
| HTTP | 1.0.0 | 2.2.x | new; external REST calls without custom code |
| RabbitMQ | 3.0.1 | (archive listing) | |

The pattern in that table is the most useful operational observation in this section: **the long-tail connectors hold an estate back.** Core connectors (Kafka, JDBC, filesystem, Paimon) keep up; HBase, Cassandra, PubSub and Hive stop in the 1.18–1.20 window. The lag is visible in the engine's own documentation — the 2.3 Kafka connector page states, in its dependency section, "There is no connector (yet) available for Flink version 2.3." Anyone planning "we will run the newest Flink" must first confirm that every connector their pipeline depends on has a build for it. **Treat the connector release train, not the engine release train, as the binding constraint on how fast a pipeline can move:** an engine upgrade is a maintenance window; a missing connector build is a redesign.

### 6.2 The Table Ecosystem and the Lakehouse

Flink's relationship to the open table formats is asymmetric and worth stating plainly. **Apache Paimon** is the Flink-lineage format: Flink's own Table Store project (0.1.0 on 2022-05-11, 0.2.0 on 2022-08-29, 0.3.0 on 2023-01-13, per the downloads archive) became Apache Paimon, now an independent ASF project whose documentation tree carries releases 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4, **2.0** and `master` (checked 2026-09-14). Paimon's releases page describes **Apache Paimon 2.0.0** as "the result of a broad community effort", a PyPaimon 2.0.0 package exists on PyPI, and Paimon's LSM structure is what makes partial updates, streaming reads and changelog production work inside a lake format. **Iceberg and Delta** are supported through their own connectors and are first-class in the sense that the community maintains them, but they are not Flink's native format in the way Paimon is — the 2.0 announcement's "Streaming Lakehouse" positioning is explicitly about Paimon. The releases keep deepening that relationship: materialized tables in 2.0; `VARIANT` support in Paimon via Flink SQL and sink reuse for multiple partial-update `INSERT INTO` statements in 2.1; materialized-table control in 2.2 and 2.3. Storage-side architecture — file layout, catalog choice, compaction strategy, object-storage economics — belongs to `technology/cloud_object_storage_lakehouse_guide.md` and `technology/data/paimon_iceberg_delta_comparison.md`; this guide stops at the Flink side of the boundary.

### 6.3 Flink CDC

Flink CDC is the sharpest example of "related project, separate release train" in the ecosystem. It lives in a **separate repository, `github.com/apache/flink-cdc`**, under the Apache Flink umbrella, having been accepted through **FLINK-34180 ("Accept Flink CDC project as part of Apache Flink")**; it was **not** merged into the main `apache/flink` repository. The module layout (`flink-cdc-runtime`, `flink-cdc-pipeline-model`, `flink-cdc-cli`, plus explicit `flink-cdc-flink1-compat` and `flink-cdc-flink2-compat` modules) shows the compatibility split being handled deliberately. Current releases on the downloads page (2026-09-14) are **CDC 3.6.0** and 3.5.0. **3.6.0** was announced **2026-03-30**: it extends Flink version support to **1.20.x and 2.2.x**, upgrades the required JDK to 11, adds an **Oracle Source** and an **Apache Hudi Sink** pipeline connector, adds lenient-mode schema evolution for the Fluss (Incubating) connector and PostgreSQL schema evolution, and adds `VARIANT` and JSON parsing to the transform framework. **3.5.0** (2025-09-26) added Fluss as a sink and PostgreSQL as a source, bumped **Paimon to 1.2.0**, and fixed a MySQL CDC GTID out-of-order data-loss bug (FLINK-37065) — a reminder that CDC correctness bugs are the ones that quietly corrupt downstream tables. The API layers are a declarative **YAML Pipeline API** driven by `flink-cdc.sh` (whole-database synchronisation with schema evolution) plus lower-level source/sink APIs. The version-alignment caveat is the same as for connectors: **CDC 3.6.0 supports Flink 1.20.x and 2.2.x, not 2.3**, so the CDC artifact's support matrix decides what a whole-database ingestion pipeline can run on.

**CDC and temporal-join patterns, at the engine level.** A CDC stream is a changelog (`+I`, `-U`, `+U`, `-D`), and Flink SQL's temporal joins (`FOR SYSTEM_TIME AS OF` against a versioned table, and regular-join-to-lookup-join conversions) are the declarative expression of the classic enrichment pattern. The planner will also convert suitable regular joins into **delta joins** in 2.1+ (FLIP-486), removing the join state entirely and re-reading the source tables via bidirectional lookup, with 2.2 adding caching to reduce external requests. The 2.3 `SinkUpsertMaterializer` change (§3.7) is directly relevant, since that operator reconciled out-of-order changelog events before an upsert sink and its old unbounded-state behaviour was the classic answer to "why is my CDC pipeline's state growing forever". Temporal joins against a CDC-fed dimension table are still state, subject to §3.7 in full. And whole-database replication into a regulated estate inherits a data-protection problem as well as an engineering one: the pipeline's state contains raw source data, and so do its checkpoints.

---

## 7. Deployment and Operations

### 7.1 The Kubernetes Story and the Operator

Kubernetes is the target Flink's own 2.0 roadmap was written around — the disaggregated state work exists because "containerized deployments on Kubernetes [have] become standard" — and the **Apache Flink Kubernetes Operator** is the blessed way to run Flink there. Developed under the Apache Flink umbrella, it extends the Kubernetes API with custom resources (`FlinkDeployment`, `FlinkSessionJob`, `FlinkStateSnapshot`, `FlinkBlueGreenDeployment`) and runs the whole operational lifecycle: deployment, stateful upgrades, rollbacks, self-healing, blue/green cutovers, autoscaling and memory right-sizing.

| Operator version | Date | Notable |
|---|---|---|
| 1.0.0 | 2022-06-04 | first stable `v1beta1` |
| 1.9.0 / 1.10.0 / 1.11.0 | 2024-07-02 / 2024-10-25 / 2025-03-03 | |
| 1.12.0 / 1.12.1 / 1.13.0 | 2025-05-28 / 2025-07-08 / 2025-09-29 | |
| **1.14.0** | **2026-02-13** (announcement 2026-02-15) | **native Blue/Green deployment support** |
| **1.15.0** | **2026-05-26** | Kubernetes-native Conditions on `FlinkDeployment`, Logback logging, bundled metric reporters, **Flink 2.2 compatibility**, reliability fixes across session jobs, savepoints and the mutating webhook. The downloads page still names **1.15 as the latest stable operator on 2026-09-14** |

**Flagged — 1.16.** The operator documentation at the `-docs-stable` alias links into `flink-kubernetes-operator-docs-release-1.16`, whose compatibility page lists Flink **2.3 and 2.4** as supported (alongside 1.19, 1.20, 2.0, 2.1, 2.2), operator JDK **17**, Kubernetes **1.21+**, Helm **3 and 4**, and a bundled internal Flink version of **1.20.4**. But `downloads.apache.org/flink/` contained no `flink-kubernetes-operator-1.16.0` directory on 2026-09-14 and the downloads page still names 1.15 as latest stable. The conservative reading this guide takes: **1.16 is documented ahead of release**, and its content describes the imminent shape of the next operator. The 1.16.0 release date is **not verified** (§13).

Three operator facts belong in a risk register. The operator and its webhook **target JDK 17**, and custom plugins must be compiled for it; Job and TaskManager pods run separately and may use a different JRE depending on the Flink image. The operator's Java types (`org.apache.flink.kubernetes.operator.*`, `org.apache.flink.autoscaler.*` and the `-api` module classes) are all `@Experimental` with **no source or binary compatibility guarantee**, so a plugin may need rebuilding on an operator upgrade — the Kubernetes-level backward-compatibility guarantee protects already deployed *resources*, not Java code compiled against the model classes. And CRDs are installed by Helm's `crds/` convention, which means **upgrades and uninstalls leave them untouched**, making CRD upgrades a manual `kubectl replace` step (or `apply` if the CRD does not yet exist); skipping it produces a new operator reconciling against an old schema. A fourth, from the Flink side: the 2.0 cleanup removed REST response fields (`host` on taskmanager/vertex/subtask responses, `execution-mode` on the job config, memory-segment metrics), so monitoring integrations that scrape those fields **break on the 2.0 upgrade**.

### 7.2 Other Deployment Targets

Standalone clusters on bare metal or VMs remain supported and documented. YARN was the historic per-job mode and the YARN option still appears in the savepoint CLI (`-yid`). Docker images are provided; `FLINK_PROPERTIES` passes configuration on Docker-based deployments, and `FLINK_CONF_DIR` selects a configuration directory for non-session resource providers — with the documented caveat that per-job configuration directories are **not** supported in Docker or standalone Kubernetes deployments.

### 7.3 High Availability and Job Recovery

JobManager HA — ZooKeeper-based or Flink's Kubernetes-based HA service — is required for any production claim; without it the JobManager is a single point of failure for both new submissions and running jobs. Recovery restarts from the most recent completed checkpoint, so **the checkpoint interval is the maximum reprocessing window**, and the checklist's framing is the sentence to take to a change board: "A checkpoint interval of 5 minutes implies that Flink will never reprocess more than 5 minutes worth of data after a failure." **Task-local recovery** — reusing the local state copy instead of pulling everything from remote storage — is **off by default** and, since 2.0, the AdaptiveScheduler respects `execution.state-recovery.from-local`, also default false, so it must be explicitly enabled. Its documented limits: keyed state only (not operator state or timers), **not supported with unaligned checkpoints**, and the RocksDB implementation's hard links require the state and local-recovery directories to be on the same physical device. **Allocation-preserving scheduling** is the mechanism that makes local recovery work — each task requests its previous slot and, if the slot is gone, a fresh one, so recovering tasks do not push each other out of their slots.

### 7.4 Metrics and Observability

Flink exposes metrics at job, task, operator and subtask scope; the families that matter operationally are checkpointing (duration, size, failures, alignment time), network and back-pressure, watermark progress, and RocksDB native metrics — the last documented but **disabled by default**. Reporters ship for Prometheus, JMX, Graphite, InfluxDB, StatsD, Datadog and OpenTelemetry. 2.3 hardened the OTel exporter specifically because large metric payloads were being rejected by the OTel gRPC endpoint and **silently dropped in production**, adding `metrics.reporter.otel.exporter.compression` (gzip|none) and `metrics.reporter.otel.batch.size` (FLIP-553) — silent metric loss deserves naming as a failure mode, because a pipeline can look healthy while nothing is arriving. The **HistoryServer** stores completed-job archives, and 2.2 added a time-based retention policy (`historyserver.archive.retained-ttl`) to combine with the existing quantity cap. Drive alerting off metrics, never off the UI: the back-pressure tab is a diagnostic with configurable sampling windows (`web.backpressure.*`), not a monitoring primitive.

### 7.5 The Savepoint-Based Upgrade Workflow

```bash
# 1. stop with a savepoint (default format is canonical; --type native for speed)
bin/flink stop --savepointPath s3://flink-savepoints/my-pipeline :jobId

# 2. deploy the new job version, restoring from the savepoint
bin/flink run -s s3://flink-savepoints/my-pipeline/savepoint-<id> -c com.example.Pipeline new-version.jar

# 3. only if stateful operators were legitimately removed:
#    -n / --allowNonRestoredState   (discards their state — a decision, not a fix)
```

On Kubernetes this is expressed as a `FlinkDeployment` suspension with `job.upgradeMode: savepoint` plus `job.initialSavepointPath` — the same sequence the operator's own v1alpha1→v1beta1 upgrade guide uses. The Kafka connector documents extra steps for a *connector* upgrade that generalise well: **do not upgrade Flink and the connector at the same time**; ensure offsets are committed on checkpoint before stopping; and change the operator `uid` of the source/sink so the new connector does not read the old connector's state, starting with `--allow-non-restored-state`.

### 7.6 Day-2 Realities an Operator Must Accept

A Flink pipeline is a running program, not a job: it does not finish, and it is monitored, versioned and upgraded like a service, with state migration in the upgrade path. The checkpoint interval is an SLA commitment that determines both reprocessing on failure and output latency for transactional sinks, and someone outside engineering should agree it. Maximum parallelism is a one-way door that must be chosen before the first production start (§3.1). **State is data**: checkpoints and savepoints contain production data and therefore inherit retention, encryption, access-control and cross-border transfer obligations, which puts legal in the design review rather than on the go-live checklist. Capacity planning must cover recovery, not just steady state — the tuning guide's second precondition for reliability is that after a failure there must be enough resources to *catch up* with the input stream, not merely to keep up with it. Back-pressure is the default explanation for most symptoms: high checkpoint duration, high end-to-end latency and rising consumer lag are usually one problem, not three. And version drift is continuous: with a 2.x minor release roughly every 6–9 months plus quarterly bug-fix waves across three or four maintained lines, an unrehearsed upgrade path is an outage waiting for a CVE.

### 7.7 Managed Offerings — Verified Live in 2026

| Offering | Status as of 2026-09-14 | Verification |
|---|---|---|
| **Amazon Managed Service for Apache Flink** | Live; renamed from **Amazon Kinesis Data Analytics** on **2023-08-30**; serverless Flink applications with a Studio for interactive development | AWS News Blog rename announcement; AWS migration docs |
| **Confluent Cloud for Apache Flink** | Live and serverless — Flink SQL, Table API (Java/Python) and UDFs over Kafka topics, integrated with Schema Registry, RBAC, private networking and autoscaling; available in AWS, Azure and GCP | Confluent documentation, checked 2026-09-14 |
| **Confluent Platform for Apache Flink** | Live — the on-premises distribution, for institutions that cannot use SaaS | Confluent documentation |
| **Ververica Platform** | Live and commercially active in 2026 (blog posts July 2026); offers fully managed cloud, **BYOC** (customer cloud account) and self-managed; explicitly marketed at banking. Its "Forrester Wave Leader, Q4 2025" and benchmark claims are **vendor claims** | ververica.com, checked 2026-09-14 |
| **Alibaba Cloud Realtime Compute for Apache Flink** | **Partially verified** — the research digest cites a 2026-07-29 release note on help.aliyun.com, but the international product page `alibabacloud.com/product/flink` returns 404 today | Flagged — §13 |
| **Apache Flink Agents** | A **new Flink sub-project** for LLM/AI agents: 0.1.0/0.1.1 (0.1.1 on 2025-12-09), 0.2.0 (2026-02-06), 0.2.1 (2026-03-26), **0.3.0 (2026-06-19)**, **0.3.1 (2026-07-25, adds Flink 2.3 distribution support)**. Every announcement labels it a **preview** whose "APIs and configuration options are experimental and may undergo non-backward compatible changes" | Flink blog; `downloads.apache.org/flink/flink-agents-*` |

For a regulated institution the managed-versus-self-managed decision is usually made on data residency and supervision rather than capability (§10.3). What the table establishes is that the market now offers all three postures — SaaS (Confluent, Amazon, Alibaba), vendor-run BYOC or self-managed (Ververica), and fully self-managed open source — so the choice can be made on policy grounds without giving up the engine.

---

## 8. The Known Hard Parts and the Critiques

This is deliberately the least comfortable section. Where a claim is documented it is cited; where it is practitioner consensus rather than documentation, it is labelled as such.

### 8.1 Operational Complexity and Skills Scarcity

Flink is a distributed stateful system with its own scheduler, memory model, state store and a compatibility matrix spanning engine, connector, table-format and operator versions. Nothing in the documentation claims otherwise — the production-readiness checklist exists precisely because sensible defaults do not cover production, and the memory guide states that "the full breadth of applications that users deploy on Flink means this isn't always possible." *Practitioner consensus, not a documented claim:* the scarcity of engineers who can read a Flink checkpoint dashboard, distinguish alignment time from start delay, size managed memory for RocksDB and safely execute a stateful upgrade is the dominant adoption cost — larger than licence or infrastructure cost. That is a market observation consistent with the relative scarcity of Flink skills in generalist hiring, not a measured fact.

### 8.2 State Size and Rescaling Pain

The documentation here is unusually candid, and the admissions collect into a picture: "max parallelism... determines the maximum parallelism to which a stateful operator can scale" and "there is currently **no way to change** the maximum parallelism of an operator after a job has started without discarding that operator's state"; RocksDB state access is "an order of magnitude slower than the memory state backends"; ForSt is "still in the experimental stage and is not fully available for production", and when most active state is remote, "the performance of state access may be affected by the network latency"; task-local recovery covers keyed state only and is incompatible with unaligned checkpoints; and unbounded state growth is a known defect class serious enough to require FLIP-558 and a breaking change in 2.3. The honest summary: **Flink 2.x is mid-migration from compute-coupled local state to disaggregated remote state, and both models ship simultaneously.** That is exactly why ForSt is labelled experimental — an operator choosing it in 2026 is choosing to be early.

### 8.3 The Checkpoint-Cost / Back-Pressure Interaction

The tuning documentation states the coupling directly: when checkpoint barriers take a long time to reach operators, "that typically indicates that the system is operating under a constant backpressure." The documented fix ordering is (1) remove the back-pressure source by optimising, reconfiguring or scaling, (2) reduce in-flight buffered data, (3) enable unaligned checkpoints — and unaligned checkpoints are explicitly *not* a fix, since they leave end-to-end latency high. *Practitioner consensus:* the most common production failure pattern is not a crashed node but a job that is measurably behind and checkpointing too slowly to recover if it fails — a stable-looking process with a growing recovery time. Because both symptoms share one cause, teams frequently buy time by raising the checkpoint timeout or disabling checkpoints for an hour, and both actions remove the last thing standing between the business and data loss.

### 8.4 Debugging a Distributed Stateful Pipeline

2.1 added a capability that implicitly admits the difficulty: a **SQL connector for keyed state** that lets you "query keyed state directly from checkpoint or savepoint using Flink SQL, making it easier to inspect, debug, and validate the state of Flink jobs without custom tooling", "especially useful for analyzing long-running jobs and validating state migrations" (FLINK-36929). Before 2.1 that required writing against the **State Processor API**. The general picture: a bug may only reproduce with a specific key distribution, arrival ordering or window boundary, and may vanish on restart because restart resets in-flight state; the tool set is a mix of engine-level (metrics, HistoryServer, checkpoint UI, the keyed-state connector, the State Processor API) and sink-level evidence; and no single tool answers "what did the operator believe at event time T". The strongest mitigation is design rather than tooling — keep business logic in pure functions testable outside Flink, and keep state shapes small and explicit.

### 8.5 The SQL-versus-Code Capability Gap

The 2.x release train is overwhelmingly SQL-facing (delta joins, multi-join operator, materialized tables, changelog conversion, AI functions, PTFs), with two consequences. **Features arrive in SQL first**, so a code-first team may have to wait for a DataStream equivalent or implement it themselves, while a SQL-first team receives the state-reduction work (delta join, zero-intermediate-state multi-join) that is hardest to build by hand. And **the escape hatch matters**: Process Table Functions are being extended — 2.3 added late-data reaction and `ORDER BY` on table arguments (FLIP-565) — while improved PTF state access and broadcast state were explicitly **deferred to a future release**. A design depending on an announced-but-deferred capability should be planned around its absence. *Practitioner consensus:* the gap has narrowed materially since 1.x, but "Flink SQL can do it" and "Flink SQL can do it at the latency and state size we need" remain different statements.

### 8.6 Upgrade Friction

Quantified from the release notes rather than asserted: 2.0 removed the `DataSet` API, the Scala DataStream/DataSet API, `SourceFunction`, `SinkFunction`, Sink V1, the `addSource`/`addSink` overloads, REST response fields and CLI options, replaced `flink-conf.yaml` with `config.yaml`, and broke every connector built on the removed interfaces; 2.2 introduced a **breaking change** to the `StreamingMultiJoinOperator`'s state management (UniqueKeys instead of UpsertKeys, FLINK-38209) after shipping it experimental in 2.1 precisely because breaking changes were expected, and changed row-type `NOT NULL` semantics with `table.legacy-nested-row-nullability` as the escape hatch and an explicit recommendation to fix queries instead; 2.3 made the `SinkUpsertMaterializer` fail at planning time where it previously degraded silently, which is correct and will still break a job that "worked"; and the operator's CRD remains `v1beta1`, with the docs stating that a future `v1` will not guarantee new resources can be deployed with the old API version even though existing resources keep running. None of that is unreasonable for a system this old, and each item is a named, dated change that can go into an upgrade plan.

### 8.7 Total Cost of Ownership

| Component | Flink's profile |
|---|---|
| Infrastructure | commodity Kubernetes; disaggregated state makes local disk and ephemeral pods viable; object-store checkpoint traffic is a real and often under-estimated line item |
| Software | Apache-2.0 open source; the licence cost of the managed alternatives trades against their operational savings |
| **People and process** | the dominant term: stateful upgrade rehearsals, checkpoint and state review, state-size governance, and the on-call skill to tell back-pressure from a broken sink |

The managed offerings exist largely because the third row is where organisations struggle, and they convert it into a line item. That is a legitimate purchase decision rather than a failure of engineering — but it should be made explicitly, and it changes the residency and control posture (§10.3).

### 8.8 What the Critiques Are Not

Flink is not fragile. The checkpointing mechanism, the state backends, the exactly-once sink protocol and the Kubernetes operator are production-grade and have been for years; the limitations above are the costs of doing genuinely hard things — keeping a distributed stateful program consistent across failures. The critique is that those costs are **concentrated in operations** and are routinely invisible at proof-of-concept time, when a pipeline runs for an hour on one node with a two-minute checkpoint interval and looks perfect.

---

## 9. The Comparison, Cross-Referenced Not Re-Derived

The discipline-level comparison — what each technology *is*, its strengths, weaknesses and best-fit use case — is **`technology/event_stream_processing_guide.md` §5** (the Flink positioning entry is §5.2; the full table is §5.7). What ESP is, its paradigm history, the theory of time semantics, windowing and watermarks, and the concepts of state and fault tolerance are that sibling's §1–§4, §6 and §7. None of it is repeated here. What follows is only a **positioning table** for a reader who arrived at this engine guide first, stating how each alternative differs from Flink at the engine level, with the competitors' states verified as of 2026-09-14.

| Engine | Engine model | State / checkpoint model | Where it diverges from Flink |
|---|---|---|---|
| **Apache Flink** (2.3.0, 2026-06-25) | true streaming; event-time first-class; batch as bounded streams | keyed/operator state with pluggable backends (heap / RocksDB / ForSt), asynchronous checkpointing, savepoints, State Processor API | the reference for this guide — richest windowing and CEP, most knobs, highest operational cost |
| **Kafka Streams / ksqlDB** | embedded library (Streams) or SQL over it (ksqlDB) on Kafka | RocksDB local state, changelog topics in Kafka, `exactly_once_v2` via Kafka transactions | no separate cluster, no savepoints, tightly coupled to Kafka partitions; simpler and cheaper for Kafka-native logic, less capable for complex event-time work |
| **Spark Structured Streaming** | micro-batch by default; continuous mode experimental | state store with checkpointing to a location; watermarks; `mapGroupsWithState` / `flatMapGroupsWithState` | shares the batch engine, so unified batch/stream code; ~100 ms scale latency floor; more limited state APIs and event-time sophistication |
| **Apache Beam** | portable model, runner-agnostic (runs *on* Flink, Spark, Dataflow and others) | delegated to the runner | portability trades against runner-specific capability; using Flink's backends or savepoints is not the Beam model |
| **RisingWave** | SQL streaming database, PostgreSQL-compatible, Kubernetes-native, storage/compute separated | system-managed state; exactly-once claimed | lower operational burden and PostgreSQL familiarity; smaller ecosystem and fewer connectors; less control over state internals |

Two framing points the sibling covers in depth matter for engine selection. **Positioning is not ranking**: Flink wins on event-time correctness, stateful expressiveness and control of state; Kafka Streams wins on deployment simplicity when the data is already in Kafka; Spark wins on team reuse where Spark already exists. And **the comparison that decides most cases is the source and the sink, not the engine** — a pipeline whose sink can only be written at-least-once and whose source cannot be replayed will not be exactly-once on any engine (§4.4).

---

## 10. The Banking and Regulated-Estate Angle

### 10.1 The Use Cases That Justify the Engine

| Use case | What Flink specifically contributes | Where the theory lives |
|---|---|---|
| **Fraud and AML screening** | per-key state (velocity, cumulative amount, beneficiary graph) over event-time windows; low-latency alerting; replayable audit of a decision | `banking/financial_fraud_detection_at_scale_guide.md`; sibling §9.1, §9.2; CEP theory in `technology/complex_event_processing_guide.md` |
| **Real-time risk and position streaming** | continuous aggregation over trade and position events with exactly-once state; temporal joins against reference data; delta joins to remove join state | sibling §9.6 |
| **Trade surveillance and pattern detection** | event-time pattern matching across long sequences with allowed lateness; CEP-style operators | `technology/complex_event_processing_guide.md`; sibling §10 |
| **Real-time reconciliation** | checkpointed state as the reconciliation ledger; exactly-once sinks where the destination is transactional | sibling §9.4 |
| **Payments processing** | exactly-once state plus transactional or idempotent sinks; savepoint-based zero-loss upgrades | sibling §9.3 |
| **Market data enrichment** | high-throughput event-time joins and windowing with back-pressure-aware tuning | sibling §9.5 |

The engine-level argument for Flink in a bank is narrow and specific: **it is the only widely deployed open-source engine that lets an institution reason precisely about what its state contained at a given event time, and therefore what a decision was based on.** That is what makes it defensible to an auditor, and it is not the same as "it is fast."

### 10.2 Exactly-Once and Auditability

A regulated institution needs to answer three questions, and only the first is a streaming question. **Was each event processed once, or is the pipeline idempotent?** §4 answers this, and the answer depends on the sink. The architectural conclusion for a bank is that **idempotence is the safer default than transactionality**, because an idempotent write keyed on a deterministic business key survives replays, savepoint restores and operator bugs alike; transactional exactly-once is stronger where available but requires the sink's participation and cannot be retrofitted to a legacy target system. **Can you reconstruct why the pipeline produced a given output?** That requires inspectable state, and Flink's answers are the **State Processor API** (read and write savepoint state) and, since 2.1, the **SQL connector for keyed state**, which queries keyed state directly from a checkpoint or savepoint (FLINK-36929) — both audit tools, not debugging conveniences. **Can the state be retained and produced as evidence?** Checkpoints and savepoints are files in an object store containing production data, so retention, immutability and legal hold are data-governance requirements on Flink's checkpoint directory, and `ExternalizedCheckpointRetention` decides whether a cancelled job leaves anything behind at all. A fourth question follows from the mechanism and is easy to miss: a savepoint is a point-in-time image, so "replay from yesterday's savepoint to reproduce this decision" is a supported forensic technique **provided the sources retain the data** — a requirement on the Kafka tier (`banking/kafka_guide.md`), not on Flink.

### 10.3 Data Residency and Deployment Constraints

The deployment posture is a policy decision first and an engineering decision second. **Self-managed on bank infrastructure on Kubernetes** gives full sovereignty at the highest operational cost (you own HA, upgrades, capacity and patching), and is the usual choice when data-residency rules, supervisory expectations on third-party risk, or a prohibition on production data leaving the estate apply. **Vendor-run BYOC** keeps data in the institution's cloud account with the vendor operating the control plane — high sovereignty, medium burden, and explicitly offered by Ververica. **Fully managed SaaS** (Confluent Cloud, Amazon MSF, Alibaba) has the lowest burden and provider-region-dependent sovereignty, and suits small teams, non-core pipelines or jurisdictions with adequate coverage. All three postures have a live 2026 supplier (§7.7); the practical constraint in most banks is not product availability but **which environment each posture is admissible in** — very often SaaS for non-production analytics, self-managed or BYOC for anything touching customer data. One engine-level constraint interacts with residency: Flink is "intentionally designed to support remote code execution", which the checklist confirms, instructing operators to restrict access to the company intranet or secure clusters with TLS, authentication and RBAC, and never to expose them to the public internet. In a regulated estate that is the boundary condition for the entire deployment, not a hardening recommendation.

### 10.4 Resilience Expectations

Regulatory operational-resilience regimes expect an institution to state and evidence its recovery objectives, and Flink's model translates as follows. **RPO** is set by the checkpoint interval plus whatever the source can replay — a dial, stated explicitly rather than assumed. **RTO** is dominated by recovery time, which is a function of state size, checkpoint-storage bandwidth and whether task-local recovery is enabled; it is *not* a constant. **Change safety** is the savepoint workflow (§7.5), which is what makes a stateful upgrade a rehearsal rather than a gamble. **Capacity after failure** must include the catch-up burst, not just steady state. And **test evidence** should include a rehearsed, timed and recorded restore from a savepoint in a lower environment, because an untested restore is an assumption.

### 10.5 The Organisational Cost

Adopting Flink changes the shape of a data team, not only its toolchain. It creates a **streaming platform team** that owns the cluster or operator, the state-store decisions, the upgrade train and the checkpoint strategy — a role that does not exist in a batch-oriented estate. It introduces **new disciplines**: state-size governance (states have owners, budgets and TTLs), checkpoint SLOs, upgrade rehearsals, and change management that understands "this release changes state semantics". It changes **testing**: unit-testing pure functions is easy while testing recovery is hard, so the credible approach is automated restore tests in CI against a real savepoint plus a small number of end-to-end replay tests against recorded event streams. And it requires a **shared language with risk and audit** — "exactly-once", "at-least-once", "checkpoint interval", "savepoint" and "reprocessing window" need definitions the second line of defence accepts; §14 is written to be quotable in that conversation. The cost is real, and the alternative — batch reconciliation with an hourly or daily lag — is often the correct answer for a use case that only *sounds* real-time.

---

## 11. Cymbal Bank Worked Example

**Everything numeric in this section is ILLUSTRATIVE AND FICTIONAL.** The volumes, latency targets, state sizes and cost figures are invented to make the design decisions concrete; they are not measured values from any system and must not be quoted as benchmarks. The *engine configuration options* named below are real and documented in the sections cited; the *values* are this hypothetical design's choices.

### 11.1 The Pipeline

Cymbal Bank wants a real-time signal path for **instant payment fraud screening**: every payment instruction is scored against per-account and per-beneficiary behavioural state, decisions above a threshold are raised as alerts to the fraud operations console inside a stated latency budget, and every decision — including the passes — is written to an immutable decision ledger for audit and model review.

```
   payment-instructions (Kafka, partitioned by account id)
              |
              v
   +---------------------------+   event-time watermarks (5 s bounded out-of-orderness)
   | FLIP-27 Kafka Source      |
   +-------------+-------------+
                 |
   +-------------v-------------+        +------------------------------+
   | keyBy(account_id)         |        | beneficiary reference state  |
   |   - velocity state        |<------>| (delta/lookup join, re-read  |
   |   - rolling amount state  |        |  from the source table)      |
   |   - last-N merchant state |        +------------------------------+
   |   - scoring UDF (pure)    |
   +------+--------------+-----+
          |              +-------------------------------+
          v                                              v
  +-------------------+                        +-----------------------+
  | alert sink        |                        | decision ledger sink  |
  | LOW LATENCY       |                        | EXACTLY-ONCE,         |
  | at-least-once     |                        | append-only           |
  +-------------------+                        +-----------------------+
```

### 11.2 Source and Sink Choices, and the Exact Guarantee Each Gives

| Element | Choice | Documented guarantee | Consequence accepted |
|---|---|---|---|
| Source | Kafka topic, FLIP-27 `KafkaSource`, `commit.offsets.on.checkpoint` enabled, `OffsetsInitializer.committedOffsets(EARLIEST)` | replayable from the offsets recorded in the checkpoint | replayability holds only while Kafka retention exceeds the maximum reprocessing window, so retention is sized against the checkpoint interval and not the other way round |
| Alert sink | HTTP/console sink, delivered as soon as the decision is made | **at-least-once** — a restart after an alert is emitted but before the checkpoint re-emits it | alerts carry a deterministic `decision_id` so the console deduplicates; operations staff are told duplicates are possible |
| Decision ledger sink | append-only, exactly-once (transactional sink, or an append-only table format written via two-phase commit) | **exactly-once output, visible only on checkpoint completion** | ledger latency is roughly one checkpoint interval — acceptable for audit, not for the alert path |
| (rejected) core ledger update | direct JDBC write to a core banking table | guarantee **not verified** for the JDBC connector in this pass (§4.3) | excluded: a non-replayable, non-idempotent core-system write is the classic source of duplicates in a payments corridor |

The design deliberately uses **two sinks with two different guarantees** — the honest way to express "fast alerts, exact books" — and the guarantee is documented per sink, never per job (§4.4).

### 11.3 State Design and Its Size

| State | Type | Illustrative size | Retention control |
|---|---|---|---|
| Account velocity counters (count and sum over trailing 1 h / 24 h) | keyed `ValueState`/`AggregatingState`, RocksDB | ~1 KB per active account × ~8 M active accounts ≈ **8 GB** | keys expire when inactive; `StateTtlConfig` background cleanup enabled |
| Recent-merchant ring buffer (last 20 merchants and amounts) | keyed `ListState` | ~2 KB per active account ≈ **16 GB** | bounded by construction (ring buffer); TTL for dormant keys |
| Beneficiary graph edges (account → beneficiary over 90 days) | keyed `MapState` | ~3 KB per active account ≈ **24 GB** | TTL 90 days; explicitly reviewed because it grows fastest |
| Watermark, timers, in-flight buffers | framework | a few GB | — |
| **Peak keyed state (illustrative)** | | **~50 GB at parallelism 64** | |

The decisions that follow from §3 are where the design earns its keep. **`EmbeddedRocksDBStateBackend` with incremental checkpoints**, not heap state, because 50 GB exceeds a sane heap budget and incremental checkpointing is the single highest-value tuning lever at this size; the cost — roughly an order of magnitude slower state access — is acceptable because state reads per record are few and the scoring UDF dominates CPU. **`ForStStateBackend` was considered and rejected for the first production release**, because it is documented as experimental, not fully available for production, without canonical savepoints or full snapshots and with state access latency-dependent on the object store; it is recorded as a roadmap item to revisit once it leaves experimental status, and the appetite is real because it is the only backend supporting asynchronous state access and the only one that scales state beyond local disk. **Maximum parallelism is set explicitly to 256 at design time** and parallelism starts at 64: this is the one-way door of §3.1, and the headroom exists only because the checklist says it cannot be changed later without discarding state. **Join state is avoided where possible** — beneficiary enrichment is a lookup/delta join re-reading the reference table rather than a regular streaming join retaining both sides, which is the most common way an enrichment join becomes an unbounded-state incident. **Timers live in RocksDB**, the documented default and the more robust and scalable choice; with session windows in the design, heap timers would raise checkpoint size.

### 11.4 Checkpoint, Savepoint and Storage Configuration

| Setting | Illustrative value | Rationale |
|---|---|---|
| `execution.checkpointing.interval` | **60 s** | the reprocessing window on failure and the output delay for the ledger; chosen as an SLA decision, per the checklist's framing |
| `state.backend.incremental` | `true` | required to make a 50 GB state's checkpoints tractable |
| `execution.checkpointing.storage` / `.dir` | filesystem on an in-region object store | HA-grade checkpoint storage; the layout is not a public API, so nothing outside Flink reads it |
| `state.checkpoints.num-retained` | 3 | two spare successful checkpoints, so one corrupted snapshot does not force a long replay |
| `ExternalizedCheckpointRetention` | `RETAIN_ON_CANCELLATION` | an operator's mistake leaves something to resume from, with manual cleanup accepted |
| `execution.checkpointing.timeout` | 10 min | generous, but only after the back-pressure cause is understood; raising a timeout to mask slow barriers is explicitly the wrong fix (§8.3) |
| `execution.checkpointing.min-pause` | 30 s | stops the pipeline doing nothing but checkpointing if checkpoints start exceeding the interval |
| `execution.checkpointing.unaligned` | **off** initially | unaligned checkpoints are incompatible with task-local recovery, cannot run concurrently, and change watermark-on-recovery semantics; adopted only if measurement shows barrier propagation dominating duration |
| `state.backend.local-recovery` | `true`, with allocation-preserving scheduling | local state survives a pod restart in the same zone; accepted limits: keyed state only, not with unaligned checkpoints, requires same-device hard links |
| Savepoints | **canonical format, always** | the only format that survives a state-backend change, a State Processor API write and a Flink minor-version upgrade (§3.6); native is faster but loses all three, and speed is not the constraint in a change window |
| Operator UIDs | assigned to **every** operator | the docs recommend all operators because it is not obvious which built-ins are stateful |

### 11.5 Parallelism, Rescaling and the Failover Position

**Sizing (illustrative):** 64 parallel subtasks on a 12-node Kubernetes pool with autoscaling, each TaskManager sized with `taskmanager.memory.process.size` and a managed-memory fraction raised above the conservative 0.4 default, because RocksDB's buffers and block cache are drawn from managed memory and raising managed memory is the tuning guide's first recommendation. **Rescaling** uses savepoint → deploy at new parallelism → verify; the 2.0 AdaptiveScheduler change that synchronises checkpointing with rescaling (FLINK-35549) is noted as the reason an in-place rescale no longer costs an extra checkpoint cycle. **Failover** keeps the default `jobmanager.execution.failover-strategy: region` so one degraded subtask does not force a whole-pipeline restart, while the restart strategy is `failure-rate` (1 failure per 1 min interval, 1 s delay) rather than `fixed-delay`, so a persistent sink outage fails the job visibly instead of looping against a downstream system; as stated in §2.4, whether the AdaptiveScheduler supports region failover directly could not be verified, so the design relies on the default behaviour rather than configuring it. **Maximum parallelism 256** caps future stateful scale-out and is stated as an accepted constraint rather than a hidden one. The node pool is sized so that after a restart the pipeline can consume the Kafka backlog *and* keep up with live traffic — the tuning guide's second precondition.

### 11.6 DR Position

| Scenario | Response | Illustrative RTO | Illustrative RPO |
|---|---|---|---|
| Single TaskManager lost | region failover, restart from the last completed checkpoint, task-local recovery if the pod lands in the same zone | seconds to low minutes | ≤ checkpoint interval |
| Availability-zone loss | restart in the surviving zone, state pulled from the in-region object store | minutes, dominated by state download | ≤ checkpoint interval |
| Whole-region loss | redeploy from the savepoint replicated to the secondary region; Kafka is the recovery source for the gap | hours (savepoint transfer plus Kafka replay) | savepoint age plus Kafka retention |
| Corrupted checkpoint | resume from a retained previous checkpoint (hence `num-retained: 3`); `recover-without-channel-state.checkpoint-id` is documented as a **last resort that loses in-flight data** and is not part of this design | minutes to hours | larger than the interval — an accepted residual risk |

### 11.7 Observability, Audit Evidence and Runbook Implications

Metrics and alerts cover: checkpoint health (completed/failed counts, duration distribution, **start delay** and **alignment duration** — the two documented numbers that expose barrier-propagation problems — and size); per-second busy/idle/back-pressured task metrics plus the 2.1+ split-level watermark metrics, so "watermark not advancing" can be attributed to idleness, alignment pausing or genuine back-pressure instead of guessed at; watermark lag per source split against event time; and sink health, where a ledger-commit failure is a data-integrity event rather than a performance event. RocksDB native metrics are enabled deliberately instead of left at their documented default of off.

The audit evidence is: the **decision ledger** (append-only, exactly-once) as the primary artefact, one row per decision with the model or rule version and the event time; **savepoints** taken before every change, retained and labelled with the deployed job version, so the state at any change boundary can be reconstructed; **state-inspection evidence** for specific investigations via the State Processor API or, since 2.1, keyed state queried from the checkpoint or savepoint through the SQL keyed-state connector; **rehearsal records** — timestamped restore-from-savepoint drills in pre-production, because an untested restore is an assumption (§10.4); and a **change record** capturing engine version, connector versions, checkpoint configuration and the reason for each change, since the guarantee depends on configuration as much as on code.

The runbook consequences are equally concrete: every change follows the savepoint workflow, and a hot deploy of a changed jar with a new `uid` is treated as a state-loss event rather than a deployment; the connector train is the binding constraint, so the pipeline cannot move to a Flink version for which the Kafka connector, the ledger sink and the CDC reference feed have no build (§6.1); alert-sink duplicates are documented so the console's deduplication is trusted rather than re-investigated; a "checkpoint duration creeping up" alert routes to a back-pressure investigation, never to a timeout increase; and state-size growth has a named owner and a monthly review, because unbounded state is a documented failure mode (§3.7, §8.2).

### 11.8 What This Design Cannot Guarantee

It cannot guarantee **no duplicate alert**: the alert path is at-least-once by construction, because sub-checkpoint-interval latency and exactly-once output are mutually exclusive with the documented mechanisms. It cannot guarantee a decision was based on a **complete view of the account's history**: watermarks bound out-of-orderness, late events beyond the watermark fall outside the window, and the allowed-lateness policy — a business decision expressed as configuration — decides whether they are side-output-recovered or dropped. It cannot guarantee the **absence of a data-protection incident**: checkpoints and savepoints contain customer data in an object store, and their residency, encryption and retention are governed by policy outside the engine (§7.6, §10.3). It cannot guarantee recovery within the **RTO** if the recovery path has never been exercised — the RTO figures above are illustrative targets, and the evidence for them is the rehearsals, not the table. And it cannot guarantee that the **fraud model is right**: Flink moves events and state correctly, which says nothing about whether the score is any good; detection theory belongs to `banking/financial_fraud_detection_at_scale_guide.md` and the sibling.

---

## 12. The Claims Audit

Every load-bearing version, date and status claim in this guide, with its verdict, source and check date. All checks were performed on **2026-09-14 (UTC)** against primary sources unless stated. **Verified** = read from a primary source with a traceable URL in this pass or in the dispatch digest. **Flagged** = true in substance but carrying a caveat, a source conflict or a qualifier that must travel with the claim. **Rejected / not asserted** = a claim I found reason to distrust, or could not support, and therefore did not make.

| # | Claim | Verdict | Source |
|---|---|---|---|
| 1 | **Flink 2.3.0 is the latest stable release; released 2026-06-25** | **Verified** | downloads page; release announcement (Apache Flink PMC) |
| 2 | **Current maintained lines: 2.3.0, 2.2.1, 2.1.3, 1.20.5** | **Verified** | downloads page, 2026-09-14 |
| 3 | **Flink 2.0.0 released 2025-03-24**; first major since 1.0; 165 contributors, 25 FLIPs, 369 issues; disaggregated state management, materialized tables, async execution model, Paimon "Streaming Lakehouse", deprecated-API cleanup | **Verified** | 2.0 announcement; 2.0 release notes; archive listing |
| 4 | **Flink 2.1.0 released 2025-07-31**; 116 contributors, 16 FLIPs, 220+ issues; `ML_PREDICT`, `VARIANT`, Delta Join default-on, multi-join opt-in, SQL keyed-state connector, PyFlink 3.12/3.8 | **Verified** | release announcement 2025-07-31; archive listing; 2.1 release notes |
| 5 | **Flink 2.2.0 released 2025-12-04**; 73 contributors, 9 FLIPs, 220+ issues; `VECTOR_SEARCH`, materialized-table flexibility, SinkUpsertMaterializer V2, balanced task scheduling, protobuf 4.x | **Verified** | release announcement 2025-12-04; archive listing; 2.2 release notes |
| 6 | 2026 bug-fix releases: 2.0.2 (2026-05-11, 34 fixes), 2.1.2 (2026-05-11, 45), 2.2.1 (2026-05-15, 44), 2.1.3 (2026-06-14, 5), 1.20.5 (2026-06-08, 4), 1.20.4 (2026-04-22, 41) | **Verified** | flink.apache.org blog index, pages 1–2 |
| 7 | **Default state backend is `HashMapStateBackend`**; backends are HashMap / EmbeddedRocksDB / ForSt; **RocksDB is the only backend offering incremental checkpoints** | **Verified** | 2.3 state-backend docs |
| 8 | **`ForStStateBackend` is "still in the experimental stage and is not fully available for production"**; the only backend supporting asynchronous state access (State API V2); no canonical savepoint, full snapshot, changelog or file-merging checkpoints | **Verified** (quoted) | 2.3 state-backend docs |
| 9 | Legacy aliases: `MemoryStateBackend` ≡ HashMap + JobManagerCheckpointStorage; `FsStateBackend` ≡ HashMap + FileSystemCheckpointStorage; `RocksDBStateBackend` ≡ EmbeddedRocksDB + FileSystemCheckpointStorage | **Verified** | 2.3 state-backend docs, "Migrating from Legacy Backends" |
| 10 | Checkpoint storage: `JobManagerCheckpointStorage` (5 MB per-state default) and `FileSystemCheckpointStorage`; a configured checkpoint dir implies filesystem storage; checkpoints are not retained by default | **Verified** | 2.3 checkpoints docs |
| 11 | Unaligned checkpoints exist since **1.11**; do not fix back-pressure; no concurrent unaligned checkpoints; incompatible with task-local recovery; rescaling restricted for pointwise/broadcast connections; watermark semantics differ on recovery | **Verified** | 2.3 checkpointing-under-backpressure and large-state-tuning docs |
| 12 | Buffer debloating introduced in **1.14** (`taskmanager.network.memory.buffer-debloat.enabled`) | **Verified** | 2.3 checkpointing-under-backpressure docs |
| 13 | Savepoint capability matrix, including **"unaligned checkpoints do not support a Flink minor-version upgrade"** | **Verified** | 2.3 checkpoints-vs-savepoints docs |
| 14 | Native-format savepoints introduced in **1.15**; intermediate savepoints since 1.15 are not used for recovery and commit no side effects | **Verified** | 2.3 savepoints docs |
| 15 | Savepoint binary format unified across backends in **1.13** | **Verified** | 2.3 state-backend docs |
| 16 | **Failover strategy default is `region`; documented values are `full` and `region`** | **Verified** | 2.3 configuration reference |
| 17 | Legacy `restart-pipelined-region` / `restart-individual` failover values consolidated into `region` | **Flagged** | not re-verified in the 2.3 docs |
| 18 | Whether the **AdaptiveScheduler** exposes region failover as its own mode | **Flagged / not asserted** | could not verify |
| 19 | Restart strategies are `disable`/`off`/`none`, `fixed-delay`, `exponential-delay`, `failure-rate`, with **no default**; documented per-strategy defaults as quoted | **Verified** | 2.3 configuration reference |
| 20 | AdaptiveScheduler in 2.0 syncs checkpointing with rescaling (FLINK-35549); new `…resource-stabilization-timeout`; `…min-parallelism-increase` deprecated; `execution.state-recovery.from-local` defaults to **false** (FLINK-36201) | **Verified** | 2.0 release notes |
| 21 | **Flink 2.0 supports Java 21, defaults to and recommends Java 17, minimum Java 11; Java 8 no longer supported** | **Verified** (quoted) | 2.0 release notes |
| 22 | **The Scala DataStream and DataSet API were removed in 2.0**; migration target is the Java DataStream API | **Verified** | 2.0 release notes (removed-API list) |
| 23 | The 2.3.0 binary is still published as `flink-2.3.0-bin-scala_2.12.tgz` | **Verified** (artifact name). The interpretation that this names the bundled shaded Scala version rather than a supported Scala API is **labelled as an interpretation** | downloads page |
| 24 | **`DataSet`, `SourceFunction`, `SinkFunction` and Sink V1 were removed in 2.0**; connectors depending on them "will not work on the Flink 2.x series" | **Verified** (quoted) | 2.0 release notes |
| 25 | 2.0 removed REST response fields (`host`, memory segments, `execution-mode`) and CLI options (`sql-client -u/--update`, `flink-client run-application`) | **Verified** | 2.0 release notes |
| 26 | Since 2.0 the configuration file is `config.yaml`; `flink-conf.yaml` is unsupported; a migration script is provided | **Verified** | 2.3 configuration docs; 2.0 release notes |
| 27 | **DataStream API V2 is documented for 2.3 and is "currently in the experimental stage and is not fully available for production"**, intended to "gradually replace the original DataStream API" | **Verified** (quoted) | 2.3 DataStream V2 overview |
| 28 | The original DataStream API is **not labelled deprecated** in the 2.3 documentation | **Flagged** (absence of evidence) — stated as "not labelled deprecated", not as "will not be deprecated" | 2.3 docs |
| 29 | **Kafka source/sink connector documents exactly-once** for Kafka topics | **Verified** (quoted) | 2.3 Kafka connector docs |
| 30 | **`FileSink` is an exactly-once sink; exactly-once sinks make results visible only on checkpoint completion** | **Verified** (quoted) | production-readiness checklist |
| 31 | The **JDBC connector's documented delivery guarantee**; per-version exactly-once support for **Paimon**; guarantee levels for Iceberg, Hudi, Elasticsearch | **Flagged / not asserted** — the JDBC README retrieved contains no guarantee statement, so the guide treats JDBC as at-least-once by default and says so | §4.3, §13 |
| 32 | **Connector compatibility**: Kafka 5.0.0/4.0.1/3.4.0; AWS 6.0.1/5.1.0; Elasticsearch 4.0.0/3.1.0; JDBC 4.1.0/4.0.0/3.4.0; HBase 4.0.0 (1.18/1.19 only); Hive 3.0.0 (1.20.x); Cassandra 3.2.0; PubSub 3.1.0; HTTP 1.0.0 (2.2.x); Kafka connector 5.0.0 dated **2026-06-02** | **Verified** | downloads page connector sections + archive listing |
| 33 | **The 2.3 Kafka connector docs state "There is no connector (yet) available for Flink version 2.3"** | **Verified** (quoted) | 2.3 Kafka connector docs |
| 34 | Flink 2.3 added `FROM_CHANGELOG`/`TO_CHANGELOG` (FLIP-564), materialized-table DDL parity (FLIP-550) and `START_MODE` (FLIP-557), SinkUpsertMaterializer rework (FLIP-558), PTF enhancements (FLIP-565), `CREATE FUNCTION … USING ARTIFACT` (FLIP-559), experimental native S3 filesystem, OTel exporter controls (FLIP-553), docs restructure (FLIP-561), 15 FLIPs | **Verified** | 2.3 announcement; 2.3 release notes |
| 35 | 2.2 changed row `NOT NULL` semantics with `table.legacy-nested-row-nullability`; `StreamingMultiJoinOperator` moved to UniqueKeys (FLINK-38209, breaking, experimental in 2.1) | **Verified** | 2.2 release notes |
| 36 | 2.1 added a **SQL connector for keyed state** to inspect state from checkpoint/savepoint (FLINK-36929) and **split-level watermark metrics** (FLINK-37410) | **Verified** (quoted) | 2.1 release notes |
| 37 | **Maximum parallelism: `0 < parallelism <= max parallelism <= 2^15`; cannot be changed after job start without discarding state**; default 128 or `MIN(nextPowerOfTwo(p*1.5), 2^15)` | **Verified** (quoted) | production-readiness checklist |
| 38 | UIDs should be assigned to all operators; auto-generated UIDs are fragile; the JobManager is a single point of failure and HA is highly recommended; Flink supports remote code execution and must not be exposed to the internet | **Verified** (quoted) | production-readiness checklist |
| 39 | Task-local recovery is off by default, keyed state only, incompatible with unaligned checkpoints; allocation-preserving scheduling enables it; RocksDB managed memory defaults to true with a 0.4 managed-memory fraction | **Verified** | large-state tuning docs |
| 40 | Native file copy for S3 recovery via `s5cmd`, "at least a factor of 2" faster (FLINK-35739) | **Verified** (quoted) | 2.0 release notes |
| 41 | **Operator 1.15.0 released 2026-05-26** (Flink 2.2 compatibility); **1.14.0 released 2026-02-13** (native Blue/Green); **1.15 is the latest released operator on 2026-09-14** | **Verified** | 1.15.0 announcement; downloads page; `downloads.apache.org/flink/` listing (no 1.16.0 directory) |
| 42 | **Operator 1.16 release date / existence** | **Flagged — not verified.** The `-docs-stable` alias points at release-1.16 docs (Flink 2.3/2.4 support, JDK 17, k8s 1.21+, Helm 3/4, bundled Flink 1.20.4) but no 1.16.0 artifact was published on 2026-09-14 | operator docs; downloads listing |
| 43 | Operator CRD is still `v1beta1`; a future `v1` will not guarantee old-version submission of new resources; operator Java API types are `@Experimental` with no compatibility guarantee; CRD upgrades are a manual step under Helm's `crds/` convention | **Verified** (quoted) | operator compatibility and upgrade docs |
| 44 | **Flink CDC lives in the separate repo `apache/flink-cdc` under the Flink umbrella, accepted via FLINK-34180, and was NOT merged into `apache/flink`** | **Verified** | repository structure; FLINK-34180; `flink-cdc-docs-stable` |
| 45 | **CDC 3.6.0 released 2026-03-30** (Flink 1.20.x and 2.2.x, JDK 11, Oracle Source, Hudi Sink, Fluss lenient mode, PostgreSQL schema evolution, VARIANT/JSON transforms); **CDC 3.5.0 released 2025-09-26** (Fluss sink, PostgreSQL source, Paimon bumped to 1.2.0, MySQL GTID fix FLINK-37065) | **Verified** | CDC 3.6.0 blog post; CDC 3.5.0 announcement; downloads CDC section |
| 46 | **Apache Paimon is the successor to Flink Table Store** (0.1.0 2022-05-11, 0.2.0 2022-08-29, 0.3.0 2023-01-13) and an independent ASF project with docs releases 0.8–2.0 | **Verified** | downloads archive; `paimon.apache.org/docs/` listing |
| 47 | **Apache Paimon 2.0.0 is released** | **Verified in substance, flagged on date** — the releases page and PyPI confirm 2.0.0; the release **date** could not be read | paimon.apache.org/releases; PyPI `pypaimon 2.0.0` |
| 48 | **Stratosphere started 2009 at TU Berlin, DFG-funded; incubator entry 2014-04-14; graduation 2014-12-17; TLP announced 2015-01-12; Flink 0.6-incubating 2014-08-26 as the first release under the Flink name** | **Verified** | ASF TLP announcement 2015-01-12; Apache Incubator status page; Flink 0.6 blog post; TU Berlin DIMA page |
| 49 | "Stratosphere started in 2010" | **Flagged — conflicting secondary sources** (ASF says 2009); recorded, not adopted | §1.2 |
| 50 | `ForStStateBackend` is "based on ForSt project", linking to `github.com/ververica/ForSt` | **Verified** (quoted) | 2.3 state-backend docs |
| 51 | **Amazon Managed Service for Apache Flink is live and was renamed from Amazon Kinesis Data Analytics on 2023-08-30** | **Verified** | AWS News Blog rename announcement; AWS migration docs |
| 52 | **Confluent Cloud for Apache Flink and Confluent Platform for Apache Flink are live** | **Verified** | Confluent documentation, 2026-09-14 |
| 53 | **Ververica Platform is live and commercially active in 2026**, offering managed cloud, BYOC and self-managed | **Verified** (product activity); its performance and Forrester Wave claims are **vendor claims**, labelled as such | ververica.com, 2026-09-14 |
| 54 | **Alibaba Cloud Realtime Compute for Apache Flink is live** | **Flagged** — the digest cites a 2026-07-29 help.aliyun.com release note, but `alibabacloud.com/product/flink` 404s on 2026-09-14 | §7.7, §13 |
| 55 | **Flink Agents**: 0.1.0/0.1.1 (0.1.1 on 2025-12-09), 0.2.0 (2026-02-06), 0.2.1 (2026-03-26), 0.3.0 (2026-06-19), 0.3.1 (2026-07-25, Flink 2.3 distribution support); every release labelled preview/experimental | **Verified** | Flink blog; `flink-agents-*` distribution directories |
| 56 | Flink's official self-description as a framework and distributed engine for stateful computations over unbounded and bounded streams | **Verified** (quoted) | Flink docs, 2026-09-14 |
| 57 | 2.0 Nexmark figures (75–120% of local-state throughput with 1 GB cache, ~50% without, ≤10% average deficit for small state) | **Verified as published — labelled community/vendor benchmark** | 2.0 announcement |
| 58 | OTel metric payloads were being dropped in production; 2.3 added compression and batch-size controls (FLIP-553); HistoryServer gained time-based retention in 2.2 (FLINK-38229); `web.backpressure.*` sampling keys exist in 2.x | **Verified** | 2.3 and 2.2 release notes; configuration references |
| 59 | The "stream's state" framing, the Cymbal Bank worked example and **all illustrative figures in §11** | **Fictional / illustrative — explicitly labelled as such** | §11 preamble; no external source claimed |

---

## 13. What Could Not Be Verified

Honest list of the gaps: what is missing, what I did instead, and how a reader closes it. Nothing in this guide was invented to fill these.

**Version and release gaps.** The **Flink Kubernetes Operator 1.16.0 release** is unresolved: the `-docs-stable` alias links into `flink-kubernetes-operator-docs-release-1.16`, whose compatibility page lists Flink 2.3 and 2.4 as supported, operator JDK 17, Kubernetes 1.21+, Helm 3/4 and a bundled Flink 1.20.4 — yet `downloads.apache.org/flink/` had no `flink-kubernetes-operator-1.16.0` directory on 2026-09-14, the downloads page still names 1.15 as latest stable, and no 1.16.0 announcement appeared in the blog index. The concurrent evidence (a `FlinkBlueGreenDeployment` CRD, a `v1beta1 → v1` migration plan, documented 2.4 support) suggests the release is imminent. Resolution: watch the downloads page and the operator blog; treat the release-1.16 documentation as forward-looking until an artifact appears. Relatedly, the **operator→Flink compatibility matrix for *released* pairs** was not read for 1.15, so "operator 1.15 supports Flink 2.2" rests on the release announcement's wording rather than a matrix. **Apache Paimon 2.0.0's release date** is unread (existence is confirmed by the releases page, the `docs/2.0/` tree and `pypaimon 2.0.0` on PyPI). **Precise dates for many 1.x feature introductions** are not attributed here: only the changes the current documentation explicitly versions (savepoint format unification in 1.13, native savepoints and intermediate-savepoint behaviour in 1.15, buffer debloating in 1.14, unaligned checkpoints in 1.11) are dated. Two dates the dispatcher's digest flagged as unread were resolved from the blog index during the final check of this guide and are now in §1.3: **2.0.1 (2025-11-10, 51 fixes)** and **2.1.1 (2025-11-10, 25 fixes)**; **1.19.3 (2025-07-10, 14 fixes)** is the most recent 1.19 bug-fix release seen.

**Technical gaps.** Whether the legacy `restart-pipelined-region` and `restart-individual` failover values still exist anywhere in 2.x, and whether the **AdaptiveScheduler** exposes region failover as its own configurable mode, are unverified — what is verified is that `full` and `region` are documented, the default is `region`, and the AdaptiveScheduler gained checkpoint/rescale synchronisation in 2.0. **Per-connector delivery guarantees beyond Kafka and FileSink** are unverified: the JDBC connector README on `main` contains no guarantee statement and the per-sink documentation for Iceberg, Hudi, Elasticsearch, Cassandra and HBase was not read, so §4.3 names only the two guarantees the primary documentation states. **Paimon's exactly-once mechanism per version** was not verified; the sibling `technology/data/paimon_iceberg_delta_comparison.md` covers the format comparison. **Task-local recovery semantics under the disaggregated (ForSt) model** are unclear: the docs say local recovery covers keyed state and that the local copy is a cache of remote SST files, but whether local recovery is meaningful rather than merely harmless in a fully disaggregated deployment was not established. The **exact current names of the per-task per-second busyness metrics** should be confirmed against the 2.3 metrics reference before writing an alerting query — the 2.1 release notes verified the split-level *watermark* metric names only. The **full Scala situation** is unverified beyond two facts (the Scala DataStream/DataSet API was removed in 2.0; the distribution is still named `…-bin-scala_2.12.tgz`), and this guide's reading of that filename as the bundled shaded Scala version is labelled an interpretation. Whether the **legacy state-backend configuration keys** still load with a deprecation warning or are rejected outright in 2.3 was not verified. **`table.exec.async-state.enabled` coverage beyond the seven re-implemented SQL operators** was not verified: the 2.0 announcement names seven operators and says the rest are still being migrated, so the operative reading is "do not assume your SQL job is async-state-ready without checking the current documentation". No independent reproduction of the **2.0 Nexmark figures** was located, which is why they are labelled a community/vendor benchmark. And the **exact Flink versions Kafka connector 5.0.0 declares support for** were not read from its own documentation, only its release date and the note that no 2.3 connector is yet available.

**Historical and vendor gaps.** **Alibaba Cloud Realtime Compute for Apache Flink** is confirmed only indirectly: the service is long-standing and the digest cites a 2026-07-29 release note on help.aliyun.com, but the international product page returned 404 on 2026-09-14. **Other regional or vendor offerings were not investigated at all** — the brief asked whether Huawei, Tencent Oceanus, Cloudera or similar Flink-based services remain current in 2026, and this guide names only Amazon, Confluent (Cloud and Platform), Ververica and Alibaba. That absence is an absence of research, not evidence of discontinuation. **Contributor/company/user counts as a maturity metric** are not used as evidence beyond the per-release contributor counts the announcements state. And the **exact date and terms of the Flink Table Store → Paimon renaming and donation** were not established, although the Table Store release dates and Paimon's current documentation tree are verified.

**What is deliberately not claimed.** No throughput, latency, state-size or cost figure is presented as a property of Flink; the only numbers quoted (Nexmark, the "factor of 2" for `s5cmd`, the 5 MB per-state JobManager limit, the 2^31-byte RocksDB per-key/value limit, the 0.4 managed-memory fraction, the 2^15 max-parallelism ceiling, the 1.20.x/2.2.x CDC windows) are documented or published values with their status labelled. No specific bank, regulator or supervisor is named in connection with Flink adoption; the one institution in this guide, Cymbal Bank, is a fictional persona and §11's numbers are illustrative. And no feature is called "production-ready" on the strength of a release note: where the documentation says experimental — ForSt, DataStream API V2, Flink Agents, the native S3 filesystem, the operator's Java API types, the native-format and intermediate savepoint guarantees — this guide repeats "experimental" and does not soften it.

---

## 14. Glossary

Terms are defined as **Flink uses them**; the discipline-level vocabulary is assumed from `technology/event_stream_processing_guide.md` §3 rather than repeated.

| Term | Meaning in Flink |
|---|---|
| **Aligned checkpoint** | Barriers flow in-band and each subtask blocks channels until all inputs have delivered a barrier. Fast to restore and portable; vulnerable to back-pressure. |
| **Asynchronous state access** | The 2.0 execution model in which operators issue state accesses without blocking, absorbing remote-state latency through parallel I/O. Supported only by `ForStStateBackend`, via State API V2. |
| **Barrier** | The marker a source injects at checkpoint time; its passage through the graph defines what is inside the snapshot. |
| **Canonical savepoint** | The backend-independent, unified savepoint format. Survives backend changes, State Processor API writes and Flink minor-version upgrades; slowest to take and restore. |
| **Checkpoint** | An automatic, Flink-owned snapshot of state and stream positions used for recovery. Deleted on cancellation by default. |
| **Checkpoint storage** | Where a snapshot lands: `JobManagerCheckpointStorage` (JM heap, 5 MB per-state default) or `FileSystemCheckpointStorage`. Orthogonal to the state backend. |
| **Chaining** | Fusing consecutive operators into one task to avoid serialisation between them; determines where back-pressure is separately observable. |
| **Delta join** | SQL join strategy (FLIP-486, default from 2.1) replacing join state with bidirectional lookup against the source tables — state size traded for source reads. |
| **Disaggregated state** | The 2.0 model in which remote storage is the primary state storage, not only the checkpoint destination. |
| **ExecutionGraph** | The parallelism-expanded, schedulable form of the job, held by the JobMaster; the unit of checkpointing and failover. |
| **Failover strategy** | Which tasks restart after a failure: `full` (all) or `region` (only those affected — the default). |
| **FLIP-27 Source** | The current source interface: split enumerator plus split reader, checkpointable per split, bounded and unbounded capable. |
| **FLIP-143 Sink / Sink V2** | The current sink interface (`Sink`, `SinkWriter`, `Committer`); the committer is what makes transactional sinks possible. |
| **ForSt** | "For Streaming" — the disaggregated LSM/KV state backend based on Ververica's ForSt project, holding SST files on remote filesystems. **Experimental, not fully available for production** in the 2.3 documentation. |
| **Incremental checkpoint** | A snapshot recording only the delta against the previous checkpoint. Offered only by `EmbeddedRocksDBStateBackend`, and always by `ForStStateBackend`. |
| **JobGraph** | The submitted operator topology after chaining; where operator UIDs are set. |
| **Key group** | A bucket of the key space and the unit by which keyed state is distributed and redistributed on rescaling. Its count is fixed by maximum parallelism. |
| **Keyed state** | State scoped to a key (`ValueState`, `ListState`, `MapState`, `ReducingState`, `AggregatingState`). |
| **Materialized table** | A declaratively defined table whose refresh pipeline the engine derives and manages, blending real-time and historical views (`FRESHNESS` from 2.0; `START_MODE` in 2.3). |
| **Maximum parallelism** | Per-operator cap on how finely keyed state can be split. `0 < parallelism <= max parallelism <= 2^15`; **cannot be changed after job start without discarding state**. |
| **Native savepoint** | A savepoint in the state backend's own format (e.g. SST files). Faster; loses backend portability, State Processor API write support and minor-version upgrade portability. |
| **Operator state** | State scoped to a subtask or subtask set rather than a key: `ListState`, `BroadcastState` and the union/redistribution variants used for rescaling. |
| **State backend** | How state is represented and where it lives during execution: `HashMapStateBackend` (heap), `EmbeddedRocksDBStateBackend` (local disk plus managed memory), `ForStStateBackend` (remote). |
| **State Processor API** | The library for reading and writing savepoint/checkpoint state offline — the auditable way to inspect or bootstrap state. |
| **State API V2** | The asynchronous state access API introduced with disaggregated state; the only API through which `ForStStateBackend`'s async access can be exploited. |
| **State TTL** | Per-state configuration expiring entries by age, on read/write or via background cleanup — the standard defence against unbounded keyed state. |
| **Task slot** | The scheduling unit on a TaskManager. |
| **Task-local recovery** | Restarting a task on the same slot so it can reuse local state instead of pulling from remote storage. **Off by default**; keyed state only; incompatible with unaligned checkpoints. |
| **Unaligned checkpoint** | A checkpoint including in-flight buffered data so barriers may overtake it; duration largely independent of throughput. Cannot be taken concurrently, cannot combine with task-local recovery, does not survive a Flink minor-version upgrade, and changes watermark-on-recovery semantics. |
| **UID** | The stable operator identifier (`uid(String)`) that scopes state in checkpoints and savepoints. Auto-generated UIDs change when program structure changes. |
| **Watermark** | The event-time progress marker (theory: sibling §6.4). Engine specifics here: per-split watermark metrics (2.1+), watermark-alignment pausing, idleness detection, and the 2.0 fix (FLINK-35886) for sources wrongly marked idle while back-pressured or alignment-paused. |

---

## 15. Cross-References and Further Reading

### 15.1 Sibling guides in this repository

| Guide | What it owns that this guide deliberately does not |
|---|---|
| `technology/event_stream_processing_guide.md` | §1 what ESP is; §2 paradigm evolution; §3 core concepts; §4 ESP patterns; **§5 technologies comparison (§5.2 Flink positioning, §5.7 comparison table)**; §6 time semantics and windowing (6.4 watermarks, 6.5 late data); §7 state management and fault tolerance (7.1 state types, 7.2 state backends, 7.3 exactly-once, 7.4 checkpointing, 7.5 savepoints vs checkpoints, 7.6 consistency trade-offs, 7.7 state at scale); §8 ESP in the EDA; §9 ESP for banking; §10 CEP in banking |
| `technology/complex_event_processing_guide.md` | CEP pattern theory, pattern languages, NFA semantics, CEP versus rule engines |
| `banking/financial_fraud_detection_at_scale_guide.md` | Fraud/AML detection theory, feature design, model governance, alert economics |
| `banking/kafka_guide.md` | Kafka platform mechanics — brokers, replication, transactions, retention, rebalancing, Schema Registry. Relevant here because checkpoint retention must be sized against source retention |
| `technology/cloud_object_storage_lakehouse_guide.md` | Lakehouse and object-storage architecture, storage tier economics, table maintenance |
| `technology/data/paimon_iceberg_delta_comparison.md` | Table-format comparison — Paimon vs Iceberg vs Delta; this guide states only the Flink-side relationship |

### 15.2 Primary Apache sources (all checked 2026-09-14 unless noted)

- Downloads and version archive (also carries connector, CDC, operator, Agents and historical Table Store releases with dates) — `https://flink.apache.org/downloads/`; release archive directory — `https://downloads.apache.org/flink/`
- Current documentation tree — `https://nightlies.apache.org/flink/flink-docs-release-2.3/`, including release notes per version (`/release-notes/flink-2.0/`, `/flink-2.1/`, `/flink-2.2/`, `/flink-2.3/`), `/docs/ops/state/state_backends/`, `/docs/ops/state/checkpoints/`, `/docs/ops/state/savepoints/`, `/docs/ops/state/checkpoints_vs_savepoints/`, `/docs/ops/state/large_state_tuning/`, `/docs/ops/state/checkpointing_under_backpressure/`, `/docs/ops/production_ready/`, `/docs/deployment/config/`, `/docs/dev/datastream-v2/overview/`, `/docs/connectors/datastream/kafka/`
- Flink blog — `https://flink.apache.org/blog/`, `https://flink.apache.org/posts/page/2/`, `…/page/3/`
- Kubernetes Operator docs — `https://nightlies.apache.org/flink/flink-kubernetes-operator-docs-stable/` (currently aliasing the `release-1.16` tree) and `…/flink-kubernetes-operator-docs-release-1.16/`
- Flink CDC — `https://github.com/apache/flink-cdc`; docs `https://nightlies.apache.org/flink/flink-cdc-docs-stable/`
- FLIPs, including FLIP-27, FLIP-143, FLIP-486, FLIP-550, FLIP-553, FLIP-557, FLIP-558, FLIP-559, FLIP-561, FLIP-564, FLIP-565 — `https://cwiki.apache.org/confluence/display/FLINK`
- Apache Incubator record for Flink — `https://incubator.apache.org/projects/flink.html`; ASF TLP announcement (2015-01-12) — `https://news.apache.org/foundation/entry/the_apache_software_foundation_announces69`

### 15.3 Vendor and adjacent projects

Apache Paimon (`https://paimon.apache.org/` and its versioned docs tree); Amazon Managed Service for Apache Flink (AWS product and developer docs, plus the 2023 rename announcement on the AWS News Blog); Confluent Cloud for Apache Flink and Confluent Platform for Apache Flink (Confluent documentation); Ververica Platform (`https://www.ververica.com/` — treat benchmark and analyst claims as vendor claims); Apache Flink Agents (Flink blog and the `flink-agents-*` distribution directories).

### 15.4 How to read this guide's date-stamps

Every version number, date and status here was true on **2026-09-14**. Flink's cadence is roughly a minor release every 6–9 months plus quarterly bug-fix waves across several maintained lines, and the operator, connector and CDC trains move independently. Before relying on any version or status claim, re-read §12 for what was verified and §13 for what was not, then check the primary source named for that claim. That is not a disclaimer; it is the operating discipline the rest of the guide argues for.

---

## 16. Closing Summary

Apache Flink in 2026 is a mature engine in the middle of a deliberate architectural migration. The stable release is **2.3.0 (2026-06-25)**, and the 2.x series has spent its first three releases moving state out of TaskManager local disk and into remote storage: the disaggregated state model introduced in 2.0.0 (2025-03-24), the asynchronous execution model that had to accompany it, and the ForSt backend that implements it. That work is not finished — the documentation still labels `ForStStateBackend` experimental and not fully available for production, only seven SQL operators had been re-implemented against the async state APIs at 2.0, and DataStream API V2 is explicitly experimental and not ready for production in the 2.3 docs. An architect adopting Flink in this cycle is adopting a system whose *current* behaviour is well documented and whose *next* behaviour is visible but not yet dependable.

What the engine does reliably, and what a guide can therefore promise, is narrower and more valuable than the marketing: it takes a stream of events, keeps structured state about them, and gives that state a consistent, inspectable, replayable history. Checkpointing is automatic and asynchronous; keyed state can be queried from a savepoint through SQL since 2.1 (FLINK-36929); the capability matrix in the checkpoints-vs-savepoints documentation says exactly which snapshot format survives which upgrade; and the savepoint workflow makes a stateful production change a rehearsable procedure rather than a gamble. Those properties put Flink in a bank — not raw throughput.

What it does not promise is where this guide spent most of its effort: that a pipeline is exactly-once end to end. That depends on the source being replayable and the sinks being transactional or idempotent, and the two sinks a bank most often wants — a legacy JDBC target and a low-latency alert path — are exactly the two whose guarantees are weakest or unverified. A Flink pipeline's guarantee is the minimum of its source, its state and its sinks; Flink supplies the middle term.

The costs are real, concentrated in operations, and quantifiable: maximum parallelism is a one-way door, the checkpoint interval is an unspoken SLA that decides what a failure costs, state size is a time problem expressed as checkpoint duration and recovery time, the connector and CDC release trains bind harder than the engine release train, and the operator's own Java API types carry no compatibility guarantee. Sixteen months of 2.x releases have also produced a steady stream of breaking changes — removed APIs in 2.0, changed `NOT NULL` semantics and a breaking multi-join state change in 2.2, a planning-time failure in 2.3 — which is what a project looks like when it is still making design decisions rather than defending them.

The practical recommendation for a regulated institution is not "adopt Flink" or "do not adopt Flink" but a sequencing: start with a use case whose value depends on event-time correctness over state; choose the source and the sinks and write down the guarantee each gives before writing a line of job code; set maximum parallelism deliberately; choose canonical savepoints; rehearse a restore; and treat the platform team as the real deliverable. Where a use case merely sounds real-time, batch reconciliation is cheaper and will be supported for longer.

Flink is the most precise instrument available for reasoning about what a distributed system believed at a given instant — and precision is exactly what it charges for. The engine is not the hard part. Keeping faith with what it can actually guarantee, across every version bump, connector lag and upgrade window for the next five years, is the hard part — and that, in the end, is the stream's state.
