# Capacity Planning for OpenSearch — Sizing and Scaling a Cluster

*The dedicated deep-research guide to the capacity discipline for OpenSearch: how to characterise the workload, how to size the shards and the heap that actually decide the cluster's fate, how to choose node roles and storage tiers, how to plan ingest, where the scaling model really stops, what it costs, and how the monitoring loop keeps the plan honest. Written for regulated-enterprise and banking estates, with the managed service kept strictly separate from the self-managed product.*

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** github.com/jackliusr/research · **Category:** Technology Series · **Date:** September 2026

**Verification posture for this guide.** Every product fact below is labelled by provenance. Three labels are used throughout: **[OS]** = stated by OpenSearch's own documentation, repositories, or release pages; **[MGD]** = stated by a cloud provider's managed-service documentation (principally Amazon OpenSearch Service on docs.aws.amazon.com) and **not** automatically true of self-managed OpenSearch; **[ES-inherited]** = a fact that originates with Elasticsearch or Elastic's documentation, is widely quoted as if it were OpenSearch's, and is carried here only with that provenance stated. Figures that could not be pinned to a primary source this pass are marked ⚠ and listed in §15 and §16. No node name, tier name, instance type, version number, licence term, or sizing figure in this guide was invented; where a number is a worked illustration it is labelled as such in the same paragraph.

**A note on the current web pass.** `web_search` returned empty result sets for every query attempted in this work (recorded in §16 as a tool limitation, not as evidence that no such material exists), so all research was done by direct extraction of primary URLs — docs.opensearch.org, opensearch.org, docs.aws.amazon.com — and by reading the repo's own sibling guides. Every extraction date is stated where it matters.

---

### Table of Contents

1. The Overview, the Thesis and the Decoder
2. The Fork, and What It Changes for a Planner
3. The Sizing Method
4. The Shard Strategy
5. Node Roles and Topology
6. Heap, Memory and the JVM
7. Storage and the Data Tiers
8. Ingest and Indexing Capacity
9. The Scaling Model and the Real Limits
10. The Cost Model
11. Monitoring and the Feedback Loop
12. The Regulated-Enterprise and Banking Angle
13. The Cymbal Bank Worked Example
14. The Anti-Patterns
15. The Claims Audit
16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

---

## 1. The Overview, the Thesis and the Decoder

**The thesis, in one line:** capacity planning for OpenSearch is a shard-and-heap problem wearing a hardware costume — the hardware is the easy part, and the two least reversible decisions are the shard count and the heap size.

Everything else in this guide is commentary on that sentence. You can change an instance type in an afternoon, add a node in an hour, or move an index between tiers with a policy edit. You cannot change the primary shard count of an existing index without reindexing it, and you cannot buy your way out of a heap that was sized for the wrong thing. Pick those two badly and the cluster will be expensive, fragile, and slow — and the fix will be a data migration, not a scaling event.

**OpenSearch is a fork of Elasticsearch** — derived from Elasticsearch 7.10.2 — and this guide distinguishes inherited facts from OpenSearch-specific facts in every section, using the three labels defined above. That distinction is not pedantry: the two products have diverged for years, several concepts were renamed, and the most-quoted sizing numbers in the industry come from Elastic's documentation, not OpenSearch's.

### 1.1 What this guide owns, and what it does not

This guide owns the **capacity discipline** for OpenSearch: the sizing method, cluster-level shard strategy, node roles and topology, heap and JVM, storage tiers and lifecycle policy, ingest capacity, scaling limits, cost, and the monitoring loop that closes the circle.

It deliberately does not re-derive:

- **Data modelling and schema design** — owned by [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) (the repo's Elasticsearch data-modelling and schema-design guide: mappings, text analysis, indexing strategy, nested-vs-join-vs-flattened patterns, aggregations-friendly schema, reindexing and schema evolution, its own §6.1 "Shard Sizing" subsection, and an e-commerce worked example). That guide has zero capacity-planning and zero node-role content; this guide cites it for modelling and does not repeat it. The critical boundary note: **its modelling knowledge largely transfers to OpenSearch, because mapping, analysis and query semantics were inherited; its operational and product facts do not transfer, because those are the parts that diverged.**
- **Generic cloud capacity sizing** — owned by [capacity_sizing_guide.md](capacity_sizing_guide.md) (the repo's 700-line generic sizing discipline: Little's Law, utilisation targets, peak-vs-average, headroom conventions, per-dimension sizing, the monitor→forecast→plan→review loop, and a worked Cymbal Bank payments-service sizing). This guide treats that as the generic methodology this guide specialises, and does not re-derive it.
- **OpenSearch as a vector store** — owned by the RAG and vector-search guides under [ai_llm/rag/](ai_llm/rag/), in particular [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md). k-NN index sizing, graph memory, and recall/throughput trade-offs are cross-referenced here only where they change the capacity arithmetic (they do: see §6.3 and §7.4).
- **Observability and logging pipelines** — owned by the repo's observability and logging guides; this guide plans the cluster those pipelines write into, not the pipeline designs themselves.
- **Adjacent datastores** — [clickhouse_guide.md](clickhouse_guide.md) and [dragonflydb_guide.md](dragonflydb_guide.md) own their own datastores; they appear here only as alternative-store contrasts at the architecture-decision boundary (§9.4).

### 1.2 The decoder

Nine terms carry most of the weight in this guide. Learn them once.

| Term | What it is | Why it matters to a planner |
|---|---|---|
| **Cluster** | A set of nodes sharing a `cluster.name` that act as one search endpoint **[OS]** | The unit you size, price, monitor, and fail over. Not the node. |
| **Node and node roles** | A single OpenSearch process holding a set of roles — `cluster_manager`, `cluster-manager-eligible`, `data`, `ingest`, `coordinating`, `dynamic`, `warm`, `search` are the roles OpenSearch's own cluster documentation enumerates **[OS]**. By default every node is cluster-manager-eligible, data, ingest and coordinating **[OS]** | Role mix is a design decision with a capacity consequence: a coordinating node absorbs fan-out work, an ingest node absorbs pipeline CPU, a warm node trades disk for cache. §5. |
| **Primary and replica shard** | An index is split into primary shards; each primary can have replicas that are full copies **[OS]**. Replicas multiply both storage and indexing work by (1 + replica count) | The single biggest multiplier on storage, write throughput cost, and recovery time. Replicas are not "extra capacity" — they are extra *copies*, and they consume the same resources. §3.2, §4.3. |
| **Index** | The logical collection of documents, with a fixed `number_of_shards` at creation **[OS]**; defaults are discussed in §4.1 and differ between self-managed OpenSearch and the managed service | **The shard count of an index is fixed at creation** — the least reversible decision in the whole stack. This is the guide's thesis. |
| **Segment** | The immutable Lucene unit a shard is built from; segments are created by refresh and combined by merges **[OS, merges covered on the indexing-tuning page]** | Segment count drives per-shard overhead and search cost; merge pressure is a real resource consumer (§8.4). |
| **Heap and garbage collection** | The JVM heap OpenSearch runs in; garbage collection reclaims heap the JVM no longer needs | Heap size determines how many shards, aggregations and caches fit. Breach it and you get GC pauses or `OutOfMemoryError`, which is why circuit breakers exist (§6.4). |
| **The storage tier** | The class of storage an index lives on — hot, warm, cold, frozen-style classes — with different cost, latency and read-only semantics. Self-managed OpenSearch implements the separation via node attributes and dedicated warm/`search` nodes **[OS]**; the managed service adds named tiers (UltraWarm, cold) **[MGD]** | Tiering is the largest infrastructure cost lever after retention, and it changes what queries are answerable at what latency (§7). |
| **The lifecycle policy** | In OpenSearch this is **Index State Management (ISM)** — policies of states, actions and transitions that roll over, move, force-merge, and delete indexes **[OS]** | The mechanism that makes tiering and retention automatic instead of aspirational. Renamed from Elasticsearch's ILM — see §2. |
| **The deployment** | **Self-managed OpenSearch** (you own nodes, JVM flags, disks) versus **Amazon OpenSearch Service** (a managed service with its own defaults, limits, tiers and tuning knobs) **[MGD]** | Many constraints differ: default shard counts, default limits, heap caps, node-role choices, and who can change them. This guide keeps the two apart throughout. §2.4, §5.6. |

### 1.3 How to read the sections

§2 fixes the fork discipline. §3 gives the method. §4 is the section this guide exists for. §5–§8 are the four sizings that follow from §3–§4 (topology, memory, storage, ingest). §9 says where it all stops working. §10 prices it. §11 keeps it honest. §12 constrains it for regulated estates. §13 works one full example. §14 lists the failure modes. §15 audits the claims. §16 states what could not be verified.

Every section ends with a reference table rather than a trailing paragraph, so a reader can skim the tables and get the argument.
## 2. The Fork, and What It Changes for a Planner

### 2.1 The fork point and its date

OpenSearch is **derived from Elasticsearch 7.10.2**, and OpenSearch Dashboards is **derived from Kibana 7.10.2** — both statements are the project's own, from the OpenSearch FAQ **[OS, opensearch.org/faq, retrieved September 2026]**. That is the fork point: **7.10.2**, the last version released under the Apache 2.0 licence.

On dates, the verified milestone is the generally-available release: **OpenSearch 1.0 GA on 12 July 2021**, listed as the initial GA release of the 1.x line on the project's release-schedule page, and the FAQ's own statement that OpenSearch "became ready for production use in July of 2021 with the generally available release of OpenSearch 1.0" **[OS, opensearch.org/releases and opensearch.org/faq, retrieved September 2026]**. ⚠ The commonly-cited *announcement* date of the fork project (April 2021) was **not** verified against a primary source this pass and is not asserted here. What is safely asserted: the fork point is version 7.10.2; the first production-ready OpenSearch release was 1.0 in July 2021.

The FAQ also states the compatibility surface that follows from taking the fork at 7.10: OpenSearch provides backwards REST APIs for ingest, search and management, uses the same query syntax and responses, and can read indices from Elasticsearch versions 6.0 up to 7.10 **[OS]**. The two projects are explicitly described as distinct, with future investment "independent of Elasticsearch" **[OS]**.

### 2.2 The licence difference

This is the reason the fork exists, and the project states it plainly: Elastic "ceased making open source options available for Elasticsearch and Kibana, releasing them under the Elastic license, with source code available under the Elastic License or SSPL", which the OpenSearch project characterises as "not open source and do not offer users the same freedoms"; OpenSearch is therefore distributed under the **Apache License, Version 2.0 (ALv2)**, which the project describes as granting rights to use, modify, extend, embed, monetise and resell **[OS, opensearch.org/faq, retrieved September 2026]**.

For a planner, the licence difference is not a legal footnote — it is an architectural constraint:

| Consequence | Practical effect on capacity design |
|---|---|
| ALv2 with permissive rights **[OS]** | You may self-manage, fork, embed, and re-port the platform, which means the self-managed path is a genuine strategic option and the capacity engineer can tune the JVM, the disks and the node roles directly. |
| The managed service is a separate product with its own licence model **[MGD]** | Choosing Amazon OpenSearch Service trades that freedom for operated infrastructure — and for a set of service-level defaults and hard limits that override several of the choices this guide recommends (§5.6, §9.3). |
| Versioning and roadmap are independent **[OS]** | Feature availability is not aligned with Elasticsearch's release train. Any assumption of the form "Elasticsearch version X has feature Y, therefore OpenSearch does too" is unsafe; verify against OpenSearch's own version-specific docs. |

### 2.3 The renamed concepts — verify these before you write a runbook

This is the fork hazard that bites hardest in practice. Two concepts that every search engineer knows by their Elasticsearch names were renamed or replaced in OpenSearch, and using the old name in an OpenSearch runbook is a correctness bug.

| Concept | Elasticsearch-lineage name | OpenSearch name | Verification |
|---|---|---|---|
| Node that manages the cluster and tracks cluster state | "master node" | **cluster manager node** — OpenSearch's own documentation states "The former 'master node' is now referred to as the cluster manager node", and the role is `cluster_manager` **[OS]** | Verified on the cluster-creation page, retrieved September 2026 |
| The API parameter for a cluster-manager timeout | `master_timeout` | `cluster_manager_timeout` — `master_timeout` is **deprecated since 2.0** "to promote inclusive language" **[OS]** | Verified on the Cluster Settings API page, retrieved September 2026 |
| Lifecycle management of indexes over time | Index Lifecycle Management (ILM) | **Index State Management (ISM)** — a plugin exposing `_plugins/_ism/policies`, with *policies*, *states*, *actions*, *transitions* and `ism_template` **[OS]** | Verified on the Index State Management page, retrieved September 2026 |
| The load-testing / benchmarking tool | Rally | **OpenSearch Benchmark** — "a macrobenchmark utility provided by the OpenSearch Project", in the `opensearch-project/opensearch-benchmark` repository **[OS]** | Verified on the OpenSearch Benchmark documentation page, retrieved September 2026 |
| Ingestion/agent tooling | Logstash/Beats as the default path | **Data Prepper** is the project's own tool, with Fluentd, Fluent Bit and OpenTelemetry Collector also supported **[OS]** | Verified on the FAQ and the cluster page's traffic-ordering note |
| Legacy plugin namespaces | `_opendistro` prefixes | Still present in places for compatibility but **deprecated** — e.g. `opendistro.index_state_management.policy_id` is documented as deprecated in favour of the ISM template field **[OS]** | Verified on the ISM page |
| Wire compatibility flag | n/a | `compatibility.override_main_response_version` exists to satisfy clients that check version numbers **[OS]** | Verified on the configuration page |

Two consequences follow, and both are planning-relevant:

1. **The tool name matters because §3.5's whole discipline depends on it.** The benchmarking tool a planner must use for OpenSearch is **OpenSearch Benchmark**, not Rally. It carries a further versioning subtlety: the documentation states that the current pages reflect "the updated terminology in OpenSearch Benchmark 2.X", that **1.15 is the last supported version in the 1.X series**, and that a migration-assistance page exists **[OS, retrieved September 2026]**. A benchmark harness written against 1.x terminology may need migrating before it can be trusted for a sizing exercise.
2. **The lifecycle name matters because §7 and §13 are built on it.** A "move to warm at 30 days" requirement is expressed in OpenSearch as an **ISM policy** whose state transition changes `index.routing.allocation.require.temp` (self-managed, using node attributes) or invokes the tier-migration API (managed service) **[OS for the ISM/attribute mechanism; MGD for the managed tier APIs]**.

### 2.4 The current version line, and where behaviour is version-dependent

As of the September 2026 retrieval of the project's release-schedule page:

| Line | Status | Initial GA | Note |
|---|---|---|---|
| **3.x** | **Current** — the major line in active development **[OS]** | 3.0 GA on **6 May 2025** | The release history at retrieval shows 3.7.0 (9 June 2026) as the most recent release in the table, with the schedule listing 3.8.0, 3.9.0, 3.10.0 and 3.11.0 to follow through 2027. |
| **2.x** | **Maintenance** — bug fixes and security patches, no new features **[OS]** | 2.19.x is the latest minor line; 26 May 2022 was the 2.0 GA | Maintenance window runs until the GA of 4.0. |
| **1.x** | **End-of-life** — maintenance window ended 6 May 2025 **[OS]** | 12 July 2021 | Do not plan a new capacity design onto 1.x. |
| **4.x** | Not released at retrieval **[OS]** | — | Named in the maintenance policy as the trigger that ends the 2.x window. The project ties major versions to critical masses of breaking changes, and notes these "tend to be tied to Lucene major version releases" **[OS]**. |

The project follows semantic versioning: minor versions are compatible within a major, and breaking changes only arrive at major versions **[OS]**.

**Version-dependent behaviour a planner must check rather than assume.** These are verified, and each one changes a sizing or topology decision:

| Behaviour | Version boundary | Capacity consequence |
|---|---|---|
| Nodes hosting **searchable snapshots** must carry the `warm` node role | **Changed in 3.0** — "As of OpenSearch 3.0, nodes that use the searchable snapshots feature must have the `warm` node role instead of the `search` role" **[OS]** | A topology designed for 2.x (search-role nodes serving snapshots) is invalid on 3.x. Re-plan the warm tier at upgrade. |
| **Shard-count limit per node** | Self-managed default **1,000** with `cluster.max_shards_per_node` **[OS setting, verified in the Cluster Settings API example]**; on the managed service, 1,000/node for Elasticsearch 7.x and OpenSearch up to 2.15, but **1,000 per 16 GB of heap up to a max of 4,000** from OpenSearch 2.17 — and on the managed service **the default limit can't be changed** **[MGD]** | The ceiling on total shard count moved at 2.17, and on the managed service it is not a tunable. This is the hard backstop behind §4's over-sharding argument and the ceiling in §9.3. |
| **k-NN indexes and tiering** | Managed service: k-NN indexes can move to warm storage (UltraWarm) and to cold storage **from version 2.17 and later**; indexes created on versions earlier than 2.x cannot migrate **[MGD]** | A vector workload's tiering plan is version-gated as well as architecture-gated. |
| **k-NN searchable snapshots** | Supported for the NMSLIB and Faiss engines **starting with 2.18** **[OS]** | Frozen-style access to vector indexes has a version floor. |
| **Remote-backed storage** | **Introduced in 2.10**; requires segment replication; cluster-level and only settable at bootstrap **[OS]** | A durability strategy that must be chosen before the cluster is built, not retrofitted. |
| **OpenSearch Benchmark terminology** | 2.X terminology in current docs; 1.15 last of the 1.X series **[OS]** | Migrate the harness before trusting old results. |

### 2.5 The practical framing — what transfers and what does not

The honest rule, stated once and applied throughout:

| Knowledge class | Transfers from Elasticsearch? | Why |
|---|---|---|
| **Data modelling and schema design** — mappings, field types, analyzers, nested vs join vs flattened, doc_values, index sorting, aggregations-friendly design, reindexing strategy | **Yes, largely** — because the schema and query semantics were inherited at the 7.10.2 fork and OpenSearch keeps backwards REST APIs, the same query syntax, and the same response shapes **[OS]** | This is exactly why the repo's modelling guide [elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) remains the correct reference for the modelling half, and why this capacity guide does not re-derive it. |
| **Operational and product facts** — feature names, lifecycle-policy APIs, tiering mechanisms, node-role names, defaults, limits, release behaviour | **No** — establish each from OpenSearch's own documentation, and from the managed service's own documentation when the deployment is managed | The products diverged; the API that implements lifecycle management has a different name and different JSON, the benchmark tool has a different name and repository, and the managed service applies defaults the open-source product does not. |
| **Widely-quoted sizing numbers** — per-shard size targets, the heap rule | **Partly, and only with stated provenance** — see §4.2, §6.1 and §15 | Some figures originate with Elastic's documentation and are quoted as though they were OpenSearch's; others are genuinely present in OpenSearch's own docs but framed differently. The claims audit sorts them. |
## 3. The Sizing Method

There is no single correct sizing method. The managed service's own sizing documentation concedes the point — "There's no perfect method of sizing Amazon OpenSearch Service domains" — and then makes the right argument: start with storage, then test with representative workloads and monitor **[MGD, docs.aws.amazon.com sizing-domains, retrieved September 2026]**. Adopt that posture; distrust anyone who hands you a finished node count.

### 3.1 Step one — characterise the workload

Sizing errors are usually classification errors. Declare the class in writing, because it decides which of the four sizings dominates.

| Class | Dominant resource | Sizing implication |
|---|---|---|
| **Search-dominated** | CPU and heap (query fan-out, per-shard overhead) | Favours more, smaller shards with good replica coverage; coordinating capacity matters; per-shard overhead dominates (§4.3) |
| **Ingest-dominated** | CPU (segment construction) plus disk write throughput | Favours dedicated ingest capacity, larger shards, delayed refresh, zero-replica windows (§8) |
| **Aggregation-heavy** | Heap and field data (or doc_values on disk) | Aggregation memory is bounded by circuit breakers, not disk (§6.5) — the class most often needing *fewer, larger* shards and more heap headroom |
| **Vector / k-NN** | Heap or native memory for graphs, plus cache | Cross-referenced, not re-derived: sizing vector indexes belongs to [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) and the other RAG guides. What matters *here* is that k-NN changes tier behaviour (version-gated, §2.4) and cache consumption (§7.4) |

Most estates are hybrids. Declare a **primary** and a **secondary** class, size for the primary, and verify the secondary does not breach a resource limit. A cluster serving both interactive search and log analytics is the classic hybrid (§13).

### 3.2 Step two — derive the data volume

The generic arithmetic: **document size × document count × retention × (1 + replica count)**, plus indexing overhead and reserved space. The most explicit published form is the managed service's, which documents **[MGD, bp-storage, retrieved September 2026]**:

- **Replicas** — "each replica is a full copy of the primary shard"; default one, and the service recommends at least one to prevent data loss, noting replicas also improve search performance for read-heavy workloads.
- **Indexing overhead** — "the total size of the source data plus the index is often 110% of the source, with the index up to 10% of the source data", measurable afterwards with `_cat/indices?v` and the `pri.store.size` value, summarised by `_cat/allocation?v`.
- **OS reserved space** — Linux reserves 5% of the file system for `root` by default.
- **Managed overhead** — the service reserves 20% of each instance's storage, up to 20 GiB, for segment merges, logs and internal operations.

```
Source data × (1 + replicas) × (1 + indexing overhead)
  ÷ (1 − Linux reserved space) ÷ (1 − service overhead) = minimum storage

Simplified (managed): Source × (1 + replicas) × 1.45
Their worked example: 66 GiB × 2 × 1.1 / 0.95 / 0.8 = 191 GiB
```

Be careful with three things. **The 1.45 multiplier is a managed-service simplification** built on that service's own 20% reservation — self-managed OpenSearch has no equivalent reservation but does have filesystem overhead, merges and translog, so the honest self-managed version uses *your* measured overheads. **Retention is the multiplier that dwarfs the others**: for rolling data the service's instruction is simply to multiply a representative period's volume by the retention period — 200 MiB/hour becomes 4.7 GiB/day, which becomes 66 GiB at two weeks **[MGD]**. And **the replica count is a second multiplier, not a spare wheel**: 60 TiB of primaries with one replica needs 120 TiB of usable shard storage before overheads, which is why replica policy belongs in the capacity conversation and not only in the resilience one. Long-lived, updated indexes are the harder case — you can measure the source data on disk easily **[MGD]**, but you must then forecast *update churn*, because reindex-and-replace workloads write far more bytes than they retain.

### 3.3 Step three — derive node count from a measured per-node capacity

1. **Establish per-node capacity for your workload class by measurement** — a node doing *your* queries against *your* mappings with *your* replica count. OpenSearch's own docs frame node-count choice the same way: it "depends on your use case", taking into account how long you hold data, average document size, typical workload (indexing, searches, aggregations), price-performance ratio and risk tolerance **[OS, cluster page, retrieved September 2026]**.
2. **Divide the requirement by that number**, then round up to satisfy topology constraints (zone multiples, replica placement, quorum — §5).
3. **Verify against the non-CPU/disk constraints**: shard count per node (§4.2, §9.3), circuit-breaker headroom (§6.5), disk headroom (§11.4).

The managed service offers one published starting ratio, to be read as exactly that: for clusters with many shards, taxing aggregations, frequent updates or many queries, "starting with a configuration closer to 2 vCPU cores and 8 GiB of memory for every 100 GiB of your storage requirement", with the warning that "some OpenSearch users report that they need many times those resources" **[MGD, bp-instances]**. It is a *floor for light workloads* from one provider's documentation, dated to this retrieval. Treating it as a rule is anti-pattern #6 (§14). Instance types and sizes change constantly: every hardware figure in this guide carries its source and date, and none should be reused without re-checking the provider's current pages.

### 3.4 The headroom convention

Three separate headrooms are needed; conflating them is a common error.

| Headroom | Protects against | Convention | Enforced by |
|---|---|---|---|
| **Disk** | Allocation and writes stopping when a shard cannot be placed | Keep usable disk well below the watermark thresholds; the managed service's own worked example operates below 80% usage **[MGD]** | Watermark settings (§11.4) |
| **Heap** | GC pressure and `OutOfMemoryError` | Never plan to the breaker limits — the parent breaker defaults to 95% of heap with real-memory accounting, so the last 5% is a wall **[OS]** | Circuit breakers (§6.5) |
| **Query/CPU** | Latency under peak concurrency and fan-out | Size for peak with the generic discipline ([capacity_sizing_guide.md](capacity_sizing_guide.md) owns peak-vs-average and utilisation targets), then verify by load test | Load test (§3.5) |

The managed service's own iteration advice is the cleanest usable statement of the convention: "because it's easier to measure the excess capacity in an overpowered cluster than the deficit in an underpowered one, we recommend starting with a larger cluster than you think you need. Next, test and scale down to an efficient cluster that has the extra resources to ensure stable operations during periods of increased activity" **[MGD, bp-instances]**. That bias is right for an initial build with no telemetry. For an existing cluster it reverses: measure first, because you already have the data.

### 3.5 The load-testing discipline — OpenSearch Benchmark

The project's own instruction is unambiguous: after assessing requirements, "we recommend you use a benchmark testing tool like **OpenSearch Benchmark** to provision a small sample cluster and run tests with varying workloads and configurations. Compare and analyze the system and query metrics for these tests to design an optimum architecture" **[OS, cluster page, retrieved September 2026]**. Three properties matter: it is a **macrobenchmark utility** for tracking a cluster's overall performance, informing upgrades and measuring workflow changes such as mapping or query edits **[OS]**; it is **the OpenSearch project's tool, not Rally** (§2.3); and its terminology changed at **2.X**, with 1.15 the last 1.X release **[OS]**.

A load-testing discipline that survives audit has five properties:

1. **Representative data.** The project's own indexing-tuning experiment used the StackOverflow dataset shipped with OpenSearch Benchmark on `r7iz.2xlarge` Intel EC2 instances with benchmark clients on a separate node **[OS]**. Note what that implies: even the project's published results cite specific instance types, a specific dataset and an indexing-only scope — evidence for the *shape* of an improvement, not a transferable capacity figure.
2. **Representative queries at representative concurrency**, including the tail, not a synthetic average.
3. **A stated pass criterion** — latency percentiles at target concurrency, error and rejection rates, resource ceilings. A test that cannot fail proves nothing.
4. **Separate ingest and search runs, then at least one run together** — because they contend (§8.6).
5. **A re-run cadence** (§11.6).

### 3.6 The honest order of operations, including when you cannot measure

| Situation | What to do | What you accept |
|---|---|---|
| **Production cluster exists** | Measure first — heap, GC, latency percentiles, rejections, watermarks, shard sizes (§11) — then size the change against measured throughput per node | Measurement bias: the current cluster may be misconfigured, so calibrate before extrapolating |
| **Migrating from Elasticsearch** | Use the existing cluster as baseline, then re-run the same OpenSearch Benchmark workload against a candidate OpenSearch build. Do not assume parity — the products diverged | Migration changes defaults, versions and tiers; wire compatibility does not imply operational equivalence |
| **Greenfield with representative data** | Benchmark on a small cluster against a sample of real data; extrapolate by resource saturation, not by linear QPS | Extrapolation error grows with cluster size; re-test at each order of magnitude |
| **Greenfield with no representative data** | Say so explicitly in the sizing document. Build the smallest cluster that holds the data, follow the start-large bias of §3.4, and defer the node count until first measured load. **Ship the shard strategy (§4) and the heap rule (§6) anyway**, because those two are not safely deferrable | Unknown workload shape — mitigate by choosing a shard strategy that tolerates being wrong (rollover-based) rather than a fixed count that does not |
| **Vendor supplies a node count with no method** | Reject it: per the source discipline, sizing numbers without a stated method are not evidence | A delay, which is cheaper than a re-index |

---

## 4. The Shard Strategy

This is the section the guide exists for. Everything else is recoverable; this is not.

### 4.1 The immutability that drives everything

An index's `number_of_shards` is set at creation and cannot be changed afterwards — the repo's modelling guide states it plainly ("`number_of_shards` is immutable after creation"), and the managed service's sharding page says outright that "because you can't easily change the number of primary shards for an existing index, you should decide about shard count *before* indexing your first document" **[MGD, bp-sharding, retrieved September 2026]**. The remedy for a wrong shard count is a reindex: a data migration with downtime risk, compute cost and a validation burden. That is why shard count, alongside heap, is one of the two least reversible decisions in the stack.

**The defaults differ between the products, and this is a classic fork trap.** OpenSearch's own documentation of index creation states that "indexes default to one primary shard and one replica" **[OS, retrieved September 2026]**. The managed service states that each index is divided into **five primary shards and one replica** (ten shards in total) and notes explicitly that "this behavior differs from open source OpenSearch, which defaults to one primary and one replica shard" **[MGD]**. Same API, same version line, a fivefold difference the moment an index is created by accident.

### 4.2 The documented size guidance, and exactly whose it is

This is where most published OpenSearch sizing advice quietly launders Elasticsearch's numbers. The honest ledger:

| Guidance | Exact statement | Source | Provenance verdict |
|---|---|---|---|
| **Shards-per-node distribution** | "Number of shards for index = k × (Number of data nodes)", with the example that 24 shards across 8 data nodes gives 3 per node | docs.opensearch.org indexing-tuning | **Genuinely OpenSearch's own** — and a *balance* rule, not a size rule |
| **Per-shard size, latency-sensitive** | "A general guideline is to try to keep shard size between 10–30 GiB for workloads where search latency is a key performance objective" | docs.aws.amazon.com bp-sharding | **Managed service guidance** — not a statement in the OpenSearch project's documentation, and not automatically binding on self-managed OpenSearch |
| **Per-shard size, write-heavy** | "30–50 GiB for write-heavy workloads such as log analytics" | docs.aws.amazon.com bp-sharding | **Managed service guidance**, same caveat |
| **Shard arithmetic** | "(Source data + room to grow) × (1 + indexing overhead) / desired shard size = approximate number of primary shards", overhead at 1.1 | docs.aws.amazon.com bp-sharding | **Managed service guidance** |
| **Shards per heap** | "On a given node, have no more than 25 shards per GiB of Java heap" (example: a 4-GiB-heap `m5.large.search` → no more than 100 shards) | docs.aws.amazon.com bp-sharding | **Managed service guidance** |
| **Shards per heap, inherited** | Fewer than **20 shards per GB of heap** per node — carried in the sibling guide as "Max ~20 shards per GB of heap" | Elasticsearch lineage; [elasticsearch guide](data/elasticsearch_data_modeling_schema_design.md) §6.1 | **[ES-inherited]**. Note the number *differs* from the managed service's 25; neither measures your workload |
| **Per-shard size, general** | The industry's most-quoted figure is **10–50 GB per shard** — the sibling guide §6.1 states "Target: 10–50 GB per shard. Below 1 GB → coordinator overhead. Above 100 GB → merge overhead", alongside "50% system RAM (max 32 GB)" for heap | Elasticsearch lineage; sibling guide §6.1 | **[ES-inherited]** ⚠ — Elastic's own page was **not** re-fetched this pass, so this is attributed as lineage, not as a re-verified Elastic citation |
| **A per-shard size figure in OpenSearch's own project docs** | **Not found this pass**, across cluster creation, index management, indexing tuning, ISM, searchable snapshots, remote-backed storage, snapshot-restore, cluster settings API and benchmark | — | **Absence in the pages consulted**, not proof of absence (§16) |

The conclusion is not "there is no guidance" but "the size target you will be quoted is either the managed service's, or Elastic's, or nobody's" — which is why §4.5 works the arithmetic rather than announcing a number. What is safe to plan on, combining only verified rules: distribute evenly (`shards = k × data nodes` **[OS]**); choose a size inside a defensible range (10–30 GiB for latency-sensitive search, 30–50 GiB for log analytics **[MGD]**); check the per-node ceiling (≤25 shards per GiB of heap as the managed service frames it **[MGD]**, and never near the 1,000-shard-per-node default **[OS]** or the 2.17+ rule of 1,000 shards per 16 GB of heap up to 4,000 **[MGD]**); and roll over rather than grow for any append-only time-series index.

### 4.3 Over-sharding — the canonical planning error

Over-sharding is the default failure of well-intentioned teams, because "more shards" sounds like "more parallelism". Each shard is an independent Lucene index with its own files, its own share of node memory, and metadata that is part of the cluster state published to cluster-manager-eligible nodes.

| Cost | Mechanism | Where you feel it |
|---|---|---|
| **Per-shard overhead** | Every shard carries its own structures, segment files and merge activity; a tiny shard costs nearly as much memory as a useful one | Heap pressure and GC — the reason both the 25-shards/GiB and ~20-shards/GB rules exist |
| **Cluster-state growth** | Shard and index metadata is part of cluster state, published to cluster-manager-eligible nodes | Cluster-manager CPU and cluster-state publication latency; the practical ceiling on total shard count (§9.3) |
| **Small-shard search penalty** | Queries fan out to every shard in the target, adding coordination work for a smaller slice of data | Query latency at fixed data volume — the sibling guide's "below 1 GB → coordinator overhead" |
| **Longer recovery and rebalance** | More, smaller work units plus more per-shard setup cost | Time-to-green after a node loss or scaling event — the availability window that matters |
| **Worse memory efficiency per unit of data** | Heap is consumed per shard, so the same data over more shards needs more heap for the same work | Circuit-breaker headroom shrinks and rejections appear earlier (§11.3) |

The managed service names the mechanism exactly: "because each shard uses some amount of CPU and memory, having too many small shards can cause performance issues and out of memory errors" **[MGD]**. It also demonstrates the growth trap worth reproducing, because it is the most common real planning mistake: with 66 GiB that will quadruple in a year, sizing at 30 GiB gives `(66 + 198) × 1.1 / 30 = 10` shards for the future — but today those hold `66 × 1.1 / 10 = 7.26 GiB` each, "below the recommended size range". The service's own alternatives are six shards (12-GiB today, 48-GiB in future) or three shards with a reindex later when shards exceed 50 GiB **[MGD]**. **The rule: never size shards for a future you are not living in.** For append-only data, future volume is absorbed by *new* indexes via rollover, not by over-sized shard counts today.

### 4.4 Under-sharding, and the recovery-time consequence

- **Recovery time scales with shard size.** A large shard is rebuilt or copied as a unit; the managed service states that "large shards can make it difficult for OpenSearch to recover from failure" **[MGD]**. If availability is expressed as time-to-green after a node loss (§12.5), shard size is an input to that number.
- **Merge overhead grows with shard size** — the sibling guide's "above 100 GB → merge overhead" captures the shape.
- **Fewer shards means coarser parallelism** for search and, more importantly, for indexing: one large primary absorbs a slice of the write stream no other shard can help with.
- **Rebalance granularity worsens.** With few large shards, adding nodes cannot spread existing data evenly — the cluster may be unable to rebalance at all in reasonable time.

The transition cost is asymmetric and that is the whole argument for getting it right first: under-sharded to correct requires a reindex; over-sharded to correct also requires a reindex.

### 4.5 The shard arithmetic at two sizes

Both examples use the same corpus so the difference is visible rather than asserted. **Every figure is an illustrative arithmetic construction, not a measured capacity figure for any product.** Corpus: 66 GiB of source data, one replica, indexing overhead 1.1 (the managed service's own example corpus **[MGD]**); candidate sizes 30 GiB and 6 GiB.

Method: `primaries = (source × 1.1) / target size`, then `total shards = primaries × (1 + replicas)`.

| Step | Strategy A — 30 GiB | Strategy B — 6 GiB |
|---|---|---|
| Data to place | 66 × 1.1 = **72.6 GiB** | 72.6 GiB |
| Primaries | 72.6 / 30 = 2.42 → **3** | 72.6 / 6 = 12.1 → **13** (12 would give 6.05 GiB each) |
| Total shards (1 replica) | 3 × 2 = **6** | 13 × 2 = **26** |
| Relative shard count | 1× | **≈4.3×** |
| Cluster-state entries | 6 | 26 |
| Balance across 3 data nodes | 6 / 3 = 2 per node — clean | 26 / 3 = 8.67 — **uneven; one node takes 9** |
| Search fan-out for a query | 6 shard targets | 26 shard targets |
| Recovery units after a node loss | 2 shards to re-replicate | ~9 — more work units, each with fixed setup cost |

The point is not that A is correct; it is the row showing a **4.3× change in shard count at identical data volume and identical service level** — and every downstream cost (per-shard heap, cluster-state entries, fan-out, recovery units, merge activity) moved with it. That factor is the entire difference between a comfortable cluster and one hitting circuit breakers at 60% of intended load.

**Now the time-series case, where the guidance actually gets used.** A log estate at 200 MiB/hour with 90-day retention:

| Parameter | Value (illustrative) |
|---|---|
| Daily volume | 200 MiB × 24 = **4.7 GiB/day** (the managed service's own worked rate) **[MGD]** |
| 90-day volume | 4.7 × 90 = **423 GiB** |
| With replicas and overhead (×1.45) | ≈ **613 GiB** of storage |
| At 30 GiB per shard | 423 × 1.1 / 30 ≈ **16 primaries** for the whole 90 days |
| One index sized for 90 days | 16 primaries, each growing daily, none rollable |
| **Daily rollover at ~5.2 GiB/index** | 4.7 × 1.1 / 5.2 ≈ **1 primary per daily index**; 90 indexes; shard size flat forever |

The daily-rollover pattern uses 90 primaries against the single index's 16 — *more shards in total* — and is nonetheless correct, because each shard stays healthy, the shard count is a function of retention rather than volume growth, and old indexes move down a tier or delete as whole units (§7). **Shard count is not the objective; shard size stability is.** That inversion is the most important idea in this section.

### 4.6 Rollover and alias patterns for time-series data

OpenSearch implements this with **ISM** (§2.3). The documented mechanics: **rollover** is an ISM *action* and the plugin advises specifying an existing rollover alias when applying such a policy **[OS]**; policies **auto-attach via templates**, using `ism_template` `index_patterns` with a `priority` resolving conflicts, per the docs' "sample policy with ISM template for auto rollover" **[OS]**; attaching implicitly via an index template's `policy_id` is the **older, deprecated** mechanism **[OS]**; triggers are conditions, since ISM automates operations "by triggering them based on changes in the index age, index size, or number of documents" **[OS]**; and **ISM jobs run every 5 minutes by default** and — critically — **"ISM does not run jobs if the cluster state is red"** **[OS]**, which has a compliance consequence (§12.2).

| Decision | Choice | Reason |
|---|---|---|
| Rollover trigger | Size-based primary, age as safety net | Size protects shard stability; age prevents a stalled ingest freezing the write index |
| Alias strategy | One write alias per data-stream family | Application code decoupled from index naming; rollover invisible to writers |
| Initial shard count for a rollover index | Small and fixed — often 1 primary per index, with replicas doing resilience work | Growth is absorbed by new indexes, so pre-provisioning is pure overhead (§4.3) |
| Retention | An ISM age-triggered state transition with deletion as an action | The lifecycle policy *is* the retention control (§7.5, §12.2) |
| Tier migration | An ISM transition setting `index.routing.allocation.require.temp` (self-managed **[OS]**) or invoking the managed tier API **[MGD]** | Automatic, auditable, reversible by policy edit |
| Force-merge on read-only indexes | An ISM `force_merge` action, off-peak | The docs suggest off-peak force-merge to improve search performance in peak hours **[OS]** |

### 4.7 The shard decision procedure

1. Classify the workload (§3.1); choose the size range from that class, citing the range's source.
2. Decide whether the index is **append-only time-series** or **long-lived and updated**.
3. Time-series: roll over. Choose the rollover unit so one period lands inside the target range, then let retention determine the total index count.
4. Long-lived: compute `(source + growth you will actually reach) × 1.1 / target size`, then pick the count that divides evenly across data nodes per `k × data nodes` **[OS]**.
5. Check both ceilings — shards per GiB of heap, and the absolute per-node shard limit for your version and deployment **[OS]/[MGD]**.
6. Verify by load test **at the intended shard count**, never at a convenient one: shard count changes search cost, so testing a 6-shard design and deploying a 60-shard one invalidates the test.

---

## 5. Node Roles and Topology

### 5.1 The role set, as OpenSearch names it

These are the node types OpenSearch's own cluster documentation enumerates, quoted from its own descriptions and best-practice column **[OS, docs.opensearch.org cluster-creation page, retrieved September 2026]**. Do not import Elasticsearch's tier-role vocabulary as if it were OpenSearch's; where the managed service offers tiering the product does not, that is separated in §5.5.

| Role (OpenSearch name) | What it does | Documented production best practice |
|---|---|---|
| **Cluster manager node** | Manages overall cluster operation and cluster state: creating and deleting indexes, tracking nodes joining and leaving, checking node health by ping, allocating shards | "Three dedicated cluster manager nodes in three different zones is the right approach for almost all production use cases" — two of the three idle except when one fails or is maintained |
| **Cluster-manager-eligible node** | Elects one node among its peers as cluster manager by voting | For production, use *dedicated* cluster managers, achieved by marking all other node types `false` on the other nodes |
| **Data node** | Stores and searches data; performs all data-related operations (indexing, searching, aggregating) on local shards; the worker nodes, needing more disk than any other type | Keep additions balanced between zones — with three zones, add data nodes in multiples of three; "we recommend using storage and RAM-heavy nodes" |
| **Ingest node** | Pre-processes data, running an ingest pipeline that transforms it before it is added to an index | For heavy ingest and complex pipelines use dedicated ingest nodes, which can also offload indexing from data nodes so those serve searching and aggregating exclusively |
| **Coordinating node** | Delegates client requests to shards, collects and aggregates results, returns the final result | "A couple of dedicated coordinating-only nodes is appropriate to prevent bottlenecks for search-heavy workloads"; prefer CPUs with as many cores as possible |
| **Dynamic node** | Delegated for custom work such as machine-learning tasks, keeping those resources off data nodes | (No production guidance given on the page) |
| **Warm node** | Provides access to searchable snapshots, caching frequently used segments and removing least-used ones to access the snapshot index in remote long-term storage (for example Amazon S3 or Google Cloud Storage) | Warm nodes hold an index allocated as a snapshot cache, so use "dedicated nodes with more compute (CPU and memory) than storage capacity" |
| **Search node** | Dedicated nodes hosting only search replica shards, separating search from indexing workloads | Use dedicated memory-optimized instances |

Two further documented facts complete the picture. **The default is everything**: "by default, each node is a cluster-manager-eligible, data, ingest, and coordinating node" **[OS]** — so a three-node starter cluster is a three-way manager vote, three data nodes, three ingest nodes and three coordinators at once. And **traffic has a documented order**: external sources such as Dashboards and Data Prepper should be directed at nodes "in the following order of availability: ingest node, coordinating node, data node", and "we do not recommended sending traffic directly to the cluster manager node" **[OS]**.

### 5.2 The dedicated cluster-manager question and the quorum arithmetic

The recommendation is unambiguous — three dedicated cluster managers in three zones for almost all production cases **[OS]** — but a planner should understand *why*, because the arithmetic is what justifies the cost. Cluster-manager-eligible nodes elect one of their number as cluster manager by a vote, and the cluster cannot make progress without that election succeeding. The arithmetic below is derived from the majority-election mechanism rather than quoted as a product figure:

| Eligible nodes | Majority required | Losses tolerated while retaining quorum | Comment |
|---|---|---|---|
| 1 | 1 | 0 | Any loss halts cluster-state changes |
| 2 | 2 | 0 | **No tolerance at all** — an even number buys resilience without providing it |
| **3** | **2** | **1** | The documented recommendation |
| 4 | 3 | 1 | Same tolerance as three, at higher cost and an extra node to publish to |
| 5 | 3 | 2 | For larger estates; three dedicated managers remains the documented general answer |

**An odd number gives tolerance at the lowest cost; an even number wastes a node.** Two managers tolerate zero failures; four tolerate exactly what three do. That is why the platform recommends three specifically, and why adding a fourth "for safety" is a mistake. Three qualifications:

- **Dedication is a separate decision from count.** A dedicated cluster manager is achieved by marking the other node types `false` on it; the role is set with `node.roles: [ cluster_manager ]` **[OS]**.
- **Manager capacity is cheap but not free.** Managers hold cluster state, and cluster state scales with shard and index count (§4.3, §9.3) — so a pathological shard count can make a "tiny" manager set the bottleneck. The knobs that surface when this happens appear in the Cluster Settings API's own example response: `cluster.routing.allocation.load_awareness.flat_skew`, `cluster.max_voting_config_exclusions`, `cluster.auto_shrink_voting_configuration`, `cluster.thread_pool.generic.max` **[OS]**.
- **Manager problems look like data problems.** A cluster that stops allocating shards, stops running ISM jobs, or refuses index creation is often suffering manager overload or lost quorum, not data-node saturation (§11.2).

### 5.3 Zone awareness and replica placement

Zone awareness converts a replica from "a copy on another machine" into "a copy that survives a zone outage". Three related mechanisms are documented as their own sections on OpenSearch's cluster page **[OS]**:

| Mechanism | Purpose | Planning consequence |
|---|---|---|
| **Shard allocation awareness** | Prevents all copies of a shard landing in one zone or rack by spreading copies across awareness domains | Set it before populating the cluster; retrofitting triggers rebalancing |
| **Forced awareness** | Prevents allocation at all when the awareness configuration cannot achieve the spread (e.g. not enough zones for the replica count) | Fails loudly at creation rather than silently losing redundancy — and surfaces topology mistakes immediately |
| **Replica count enforcement** | Enforces that a minimum number of copies is always allocated in each zone | Makes "one replica in each of three zones" a rule rather than an intention |

The practical arithmetic follows from the data-node guidance: "if you have three zones, add data nodes in multiples of three, one for each zone" **[OS]**. That one rule satisfies even shard distribution (`k × data nodes`, §4.2), zone-spread replica placement, and the three-managers-in-three-zones recommendation simultaneously. Note the replica/zone relationship: with `N` replicas and `N+1` zones, every shard has a copy in a distinct zone and one zone can be lost without data loss; with fewer zones, some zone holds two copies, so losing it costs more redundancy than the replica count suggests. That is precisely why forced awareness refuses to pretend.

### 5.4 Collapsing roles onto fewer nodes — when it is correct, when it is a mistake

The discriminator is contention and blast radius: collapse roles while the cluster is small and the workload uniform; separate them when a specific contention is measured.

| Collapse | Correct when | A mistake when |
|---|---|---|
| Data + coordinating + ingest (the default) | Small clusters, moderate ingest, no evidence of ingest/search contention | Heavy ingest or heavy fan-out search — the documented remedy for both is dedicated nodes **[OS]** |
| Data + cluster-manager-eligible | Development, or small clusters with bounded shard counts | Production with many shards or indexes: allocation work, cluster-state publication and data work share one JVM and CPU budget, so an overloaded data node can cause an election crisis |
| Cluster-manager-only node also serving queries | Never as a deliberate design — the documentation advises against sending traffic directly to the cluster manager **[OS]** | Any traffic reaching it; a coordinator workload on a manager is a self-inflicted quorum risk |
| Warm role collapsed onto data nodes | Data nodes have spare CPU and local disk for the snapshot cache | The documented rationale for warm nodes is compute over storage plus cache isolation **[OS]**; sharing them with hot data makes the cache compete with page cache and merges |
| Search role collapsed onto data nodes | Mixed workloads with a modest search share | The role exists to keep search replicas off the indexing path **[OS]**; collapsing it re-creates the contention it was introduced to solve |

### 5.5 Where the managed service constrains the choice

Managed-service documentation must not be presented as product behaviour. Amazon OpenSearch Service makes several topology decisions for you and hard-limits others:

| Constraint | Statement |
|---|---|
| The term is **"dedicated master nodes"**, not "dedicated cluster manager nodes" | Documented as improving performance and cluster reliability and recommended for production or complex-state clusters **[MGD, bp-instances]** |
| Minimum node guidance | A minimum of three nodes avoids issues such as split-brain; with three dedicated master nodes, a minimum of two **data** nodes is still recommended for replication **[MGD, bp-instances]** |
| UltraWarm prerequisites | Domains must have dedicated master nodes **[MGD, ultrawarm]** |
| Cold storage prerequisites | Requires warm storage enabled on the same domain plus dedicated master nodes **[MGD, cold-storage]** |
| Instance families excluded from tiering | T2/T3 data nodes cannot use warm storage, and likewise cannot use cold storage **[MGD]** |
| Warm-node count tied to zones | Under Multi-AZ with Standby, "the number of warm nodes must be a multiple of the number of Availability Zones being used" **[MGD, ultrawarm]** |
| Permitted node counts and family limits | Per-AZ limits (334 hot / 250 warm at one AZ, 668/500 at two, 1002/750 at three, overall limit on hot + warm) plus per-family totals that differ by version and carry a lower default requiring a quota increase **[MGD, limits]** |
| Tier types you cannot invent | The service exposes hot (instance store or EBS), UltraWarm and cold storage — not an arbitrary coordinating-only fleet or an open-source-style frozen tier **[MGD]** |

The lesson: **on the managed service, topology is partly a product property, and several options are gated behind prerequisites — dedicated master nodes, warm before cold, instance-family restrictions — that must be decided at domain creation.** Those are exactly the decisions that are expensive to change later, the same structural argument as the shard count.

---

## 6. Heap, Memory and the JVM

### 6.1 The documented heap rule, with its reason

This is the second of the two irreversible decisions, and the figure is widely misattributed — so here is the evidence precisely.

**What OpenSearch's own documentation says:** "A larger Java heap size is useful for indexing. **Setting the Java min and max heap sizes to 50% of the RAM size shows better indexing performance on EC2 instances**" **[OS, docs.opensearch.org indexing-tuning page, retrieved September 2026]**. That is a genuine OpenSearch statement of the 50% rule.

**Its stated scope:** a roughly 60% throughput improvement for an **indexing-only** workload on **`r7iz.2xlarge` Intel EC2 instances**, using the **StackOverflow dataset** via OpenSearch Benchmark, with only the OpenSearch server process on the machines and benchmark clients on a separate node **[OS]**. The page frames it as an improvement "compared to the out-of-the-box experience" for that workload.

**What that means.** The 50% figure is an OpenSearch-published, measured, *indexing-workload* finding — not a timeless rule for every workload. It converges with the wider search-engine consensus and matches what the repo's Elasticsearch modelling guide already carries in its §6.1 ("Heap: 50% system RAM (max 32 GB). Rest for OS page cache"). Hold it as **a rule of thumb with a stated reason, validated by measurement for your workload.**

**The reason, which is the part that survives translation.** The JVM heap is not the whole of memory, and treating it as such is the actual error. The remainder does essential work: the **filesystem and page cache**, since Lucene reads segments from disk and a heap that has swallowed the RAM leaves nothing to cache them, so searches become disk-bound; **garbage collection behaviour**, since a larger heap postpones collections but makes each longer, and long pauses convert a throughput problem into a latency problem, which for interactive search is usually worse; and **off-heap structures**, which live outside the Java heap entirely — so "heap = memory" is doubly wrong.

⚠ **A provenance caution.** The commonly attached justification for a ceiling — that JVM compressed ordinary object pointers (compressed oops) stop being used above roughly 32 GB, so heap beyond that costs more memory per object — is **Lucene/JVM lineage**, not verified from OpenSearch's documentation in this pass. It is real JVM behaviour, flagged here rather than asserted as an OpenSearch statement. The managed service does impose a practical ceiling of its own, verified in §6.7.

### 6.2 What the remaining memory is for

| Memory region | Serves | Sizing implication |
|---|---|---|
| **JVM heap** | Shard-level structures, per-segment readers, aggregation working memory, caches, translog buffers | The 50%-of-RAM starting point, bounded by the breakers below |
| **OS page cache** | Lucene segment reads for search | The reason heap is capped rather than maximised: a heap-maximised node is a disk-bound node |
| **Index buffer** | The in-memory buffer documents are indexed into before becoming a segment | Default 10% of JVM heap; up to 25% suggested for further indexing improvement **[OS]** |
| **Searchable-snapshot cache** | Locally cached segments downloaded from object storage for warm-tier access | On a dedicated `warm`-role node this defaults to **80%** of available storage; otherwise set with `node.search.cache.size` **[OS]** |
| **Off-heap / native memory** | Engine-level and (for k-NN) native structures | Invisible to heap monitors, so node-level memory pressure can exist while the JVM heap dashboard looks healthy |

**The k-NN dimension** is cross-referenced rather than re-derived — see [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) — but two facts belong here because they are tier and version decisions: managed-service documentation notes that k-NN indexes moving to warm storage are **not** force-merged to a single segment, specifically to avoid OOM on hot and warm nodes from graphs too big for in-memory engines, with the consequence that more segments per shard may consume more local cache and allow fewer indexes to migrate **[MGD, ultrawarm]**; and k-NN indexes became migratable to warm and cold **from 2.17**, with older 2.x-created k-NN indexes unable to migrate **[MGD]**.

### 6.3 Garbage collection, in capacity terms

The planner's job on GC is not to tune it but to ensure the cluster never depends on GC behaviour that cannot be guaranteed. Three observations: **GC pauses are latency incidents** — a pause stopping the world on a search node appears as a spike at the percentile your service level measures, so a cluster routinely near-full in heap will have that percentile dominated by GC; **rising GC frequency at fixed load is a capacity signal**, one of the clearest early warnings that shard count, heap size or query mix has outgrown the node, and it is measured, so it belongs in the monitoring loop (§11) rather than the reactive queue; and **killing a node for exceeding heap costs more than the pause**, because a JVM exiting on `OutOfMemoryError` removes a data node and triggers shard reallocation and recovery. Circuit breakers exist to prevent exactly that, which is why they are a capacity concept and not merely a safety feature.

### 6.4 The circuit breakers — what they protect

Circuit breakers "prevent OpenSearch from causing a Java `OutOfMemoryError`": the parent breaker governs the total memory available for all child breakers, and each child governs itself **[OS, docs.opensearch.org circuit-breaker page, retrieved September 2026]**.

| Breaker | Setting | Default | Protects |
|---|---|---|---|
| Parent | `indices.breaker.total.limit` | **95% of heap** when `indices.breaker.total.use_real_memory` is `true` (the default); **70%** otherwise | The whole heap — the last wall before `OutOfMemoryError` |
| Field data | `indices.breaker.fielddata.limit` | **40% of heap** | Loading a field into the field-data cache (the aggregation-heavy workload's exposure) |
| Request | `indices.breaker.request.limit` | **60% of heap** | Data structures built for a request, e.g. aggregation computation |
| In-flight requests | `network.breaker.inflight_requests.limit` | **100% of heap**, so effectively governed by the parent | All currently running incoming requests, sized from content length |
| Script compilation | `script.context.<context>.max_compilations_rate` | **75 compilations per 5 minutes per context**; `script.max_compilations_rate` replaces per-context limits with one cluster-wide limit | Runaway script compilation; exceeding it returns a `circuit_breaking_exception` |
| Regular expressions | `script.painless.regex.enabled`, `script.painless.regex.limit-factor` | `limited` with a limit factor of **6** | Regex complexity inside Painless scripts |

Three planning consequences. **The parent breaker's 95% with real-memory accounting means there is no comfortable margin up top** — a cluster running at high heap utilisation hits breaker rejections before `OutOfMemoryError`, which is the desired behaviour but still a failed request, so plan heap utilisation to leave room for peak. **Aggregation-heavy workloads are bounded by the field-data and request breakers, not by disk**: a capacity plan that says "we have plenty of disk" has said nothing about whether the aggregations fit, which is the most common gap in aggregation-heavy sizing. And **rejections are a capacity datum** — a breaker event tells you exactly where the limit is, so capture it (§11.3) instead of treating it as noise.

### 6.5 Other memory settings that move the needle

| Setting | Default / guidance | Capacity effect |
|---|---|---|
| Index buffer | 10% of JVM memory; up to 25% for heavy indexing **[OS]** | Helps ingest throughput at the cost of heap headroom for search |
| Translog flush threshold | `flush_threshold_size` default **512 MB**; the docs suggest considering **25% of the Java heap** for pure indexing, with a 1024 MB example called "ideal for instances that have more than 32 GB of memory" **[OS]** | Fewer, larger segments and fewer flushes lift indexing throughput — but the docs warn translog completion takes longer and "if a shard fails, then recovery takes more time because the translog is larger" **[OS]**. A throughput gain purchased with recovery time is a capacity decision, not a free win |
| Refresh interval | Default **1 second**; OpenSearch "only refreshes indexes that have received at least one search request in the last 30 seconds"; raising it is recommended to prevent **429 errors**, with `30s` or disabling it in a pure-indexing scenario **[OS]** | Trades search freshness for ingest throughput and rejection rate |
| Searchable-snapshot cache | 80% of available storage on a dedicated warm node; otherwise `node.search.cache.size` **[OS]** | Determines how much snapshot-backed data is served at local speed before object-storage round trips dominate (§7.3) |
| Concurrent merges | `max_merge_count`; the concurrent merge scheduler runs merges in separate threads and further merges wait when the thread maximum is reached **[OS]** | Merge threads consume CPU and I/O that would otherwise serve search |

### 6.6 The managed service's heap constraints

The managed service constrains this layer in ways self-managed OpenSearch does not: **Java process memory is limited to 50% of total available memory, up to 32 GiB** **[MGD, limits]**; **the 32 GB upper limit does not apply to r7g and OpenSearch-optimized instances**, but "the increased heap allocation is not automatic. The default heap allocation remains 32 GB even on these instance types. Heap above 32 GB may be enabled via Auto-Tune … or by opening a support case with proper justification" **[MGD, limits]**. Consequently, any heap planned above 32 GiB in a managed domain is a request to the provider rather than a configuration change — a scheduling risk in a capacity project.

**The synthesis, in one defensible sentence:** the heap is sized to serve the working set in memory while leaving the machine enough memory to cache segments and keep collections short, and it is validated by measurement for the workload at hand. Any heap figure defended with "it's the standard number" and no reason is anti-pattern #3 in §14.

---

## 7. Storage and the Data Tiers

### 7.1 The tier model as OpenSearch itself names it

The self-managed product does not hand you four named tiers; it hands you **node-level building blocks** and expects you to compose them. Verified from OpenSearch's own documentation: **tiering is expressed as a node attribute** — the cluster page shows `index.routing.allocation.require.temp` being set to `hot` so that "OpenSearch stores your most recent data on your hot nodes", then shows ISM changing that setting to `warm` when an index reaches a certain age, moving data automatically **[OS, retrieved September 2026]**; **tiering is enforced by node roles**, with `warm` existing to serve searchable snapshots from remote long-term storage and `search` hosting search replica shards **[OS]**; and **the remote tier is object storage** — searchable snapshots read index data "on demand in real time (at search time) rather than downloading all index data to cluster storage at restore time", from sources "for example, Amazon Simple Storage Service \[Amazon S3\] or Google Cloud Storage" **[OS]**.

Crucially, **what self-managed OpenSearch does not name**: in the pages consulted this pass, OpenSearch's own documentation does **not** define a named "cold" or "frozen" tier for the self-managed product. Those names belong to the managed service (cold storage) and to the Elasticsearch lineage (frozen tier). Do not write "the OpenSearch frozen tier" into a runbook without checking the version's own documentation.

### 7.2 What each tier is for

| Tier | Mechanism | Should hold | Read/write |
|---|---|---|---|
| **Hot** | Local instance store or attached block storage; full indexing and search | Anything being written, plus everything with a tight latency objective | Read-write |
| **Warm** (self-managed `warm` role, or an attribute-tiered warm node) | Server-side cache in front of remote object storage, or local storage with reduced compute | Data no longer written but still queried regularly | Snapshot-backed warm indexes are **inherently read-only** — "any attempt to write to a searchable snapshot index results in an error" **[OS]** |
| **Cold** | **Managed-service tier**: object-storage-backed, queried by selectively attaching indexes to existing warm nodes **[MGD]** | Infrequently accessed data, compliance-retained data, periodic research and forensic analysis | Read-only until returned to a warmer tier |
| **Snapshot / backup** | A registered repository: shared file system, Amazon S3, HDFS or Azure Storage **[OS]** | The durability copy — not a queryable tier in the self-managed product | Restore-only, except searchable snapshots |

### 7.3 The documented mechanisms behind the warm and remote tiers

Both are version-gated, which makes them decisions to be taken early.

**1. Searchable snapshots.** Index data stays in the repository in snapshot format; segments are downloaded and cached on demand at search time, with the node's local storage used for caching. The documentation states that "the computing capacity of cluster nodes is shared between indexing, local search, and data segments on a snapshot residing on lower-cost object storage", that node resources are used "much more efficiently", and — the cost — that "the high number of tasks results in slower and longer snapshot searches" **[OS]**. Version-relevant facts: a node using searchable snapshots must have the **`warm`** role **as of 3.0**, previously `search` **[OS]**; restoring with `storage_type: remote_snapshot` requires at least one `warm`-role node, the default being `local`, which downloads everything **[OS]**; the index's store type becomes `remote_snapshot` **[OS]**; k-NN searchable snapshots are supported for NMSLIB and Faiss **from 2.18** **[OS]**; and cache sizing is the performance control, defaulting to 80% of available storage on a dedicated warm node **[OS]**.

**2. Remote-backed storage.** "Introduced 2.10" **[OS]**. The translog is uploaded to remote storage when a write lands on the primary, and segments are uploaded after refresh, flush and merge, with replicas sourcing copies from the same remote segment store rather than receiving write requests directly. It requires **segment replication**, and is a **cluster-level setting settable only at bootstrap** — "after bootstrapping completes, the remote-backed storage cannot be enabled or disabled" **[OS]**. Azure Blob Storage, Google Cloud Storage and Amazon S3 are compatible repository implementations, with separate repositories for segments, translog and cluster state (they may share one) **[OS]**. It is therefore a **capacity-changing durability choice**: it moves write amplification off the replica path and makes restore-to-last-acknowledged-write possible "regardless of replica count, if `index.translog.durability` is set to `request`" **[OS]** — and because it is bootstrap-only, it is a design-phase decision or none, the same structural property as shard count.

### 7.4 Storage-type trade-offs, labelled and dated

Catalogues change constantly; treat the table as illustrative and re-check the provider's current limits before committing.

| Option | Verified facts | Caveat |
|---|---|---|
| **EBS gp2** | Minimum volumes from 10 GiB; maximum sizes documented per instance type **[MGD, limits, retrieved September 2026]** | Older general-purpose SSD generation |
| **EBS gp3** | Maximum supported **80,000 IOPS and 2,000 MB/s throughput**, effective limits possibly lower depending on instance type **[MGD, limits]** | The reason gp3 is usually the default general-purpose choice |
| **EBS magnetic** | 100 GiB maximum for all instance types except t2.small/t2.medium; unsupported on Graviton **[MGD, limits]** | Legacy; not a serious hot-tier candidate |
| **Instance store** | OpenSearch's own tuning page recommends "use an instance type that has SSD instance store volumes (such as I3)", stating those deliver better ingestion performance than gp2 EBS volumes **[OS]** | Ephemeral by design: node loss loses local data unless replicas or remote-backed storage cover it |
| **EBS io2 / st1 / sc1-style families** | ⚠ Provisioned-IOPS and throughput-optimised/cold-HDD families are generally available, but were **not** verified on the OpenSearch Service limits page this pass | Confirm current support, size limits and IOPS behaviour before planning around them |
| **Managed object tiers** | UltraWarm warm nodes and cold storage are both backed by **Amazon S3** with a caching layer **[MGD]** | A product feature, not a disk choice — a distinction self-managed OpenSearch does not share |
| **Warm-node storage ceilings (managed)** | UltraWarm maximums documented at **1.5 TiB** (ultrawarm1.medium.search) and **20 TiB** (ultrawarm1.large.search); for OpenSearch Optimized OI2, 80% of local storage is cache and maximum addressable warm storage is **5 × cache** (468 GB local → 375 GB cache → 1,875 GB addressable) **[MGD, limits]** | Determines how much warm data a fixed node count can hold — a hard ceiling, unlike EBS which scales with the volume |

### 7.5 The lifecycle mechanics that move data between tiers

Tiers are only useful if data moves without a human in the loop, and in OpenSearch that is ISM's job. A policy is "a set of rules that describes how an index should be managed", composed of **states** each with **actions** executed on entry and **transitions** whose conditions move the index onward; the first state created is the initial state **[OS]**. ISM triggers on changes in index age, index size, or number of documents **[OS]** — so a move-to-warm can be age-driven while a rollover is size-driven. The move itself, in self-managed OpenSearch, is a change to `index.routing.allocation.require.temp` **[OS]**; on the managed service the equivalent is the tier-migration API with ISM automation, with named documentation topics for automating migrations to UltraWarm and to cold storage, migration tuning, and the ability to **cancel** a queued or failed migration **[MGD]**. Force-merge before low-tier storage is both a self-managed ISM action (recommended off-peak) **[OS]** and a managed migration-tuning concern — the service deliberately does **not** force-merge k-NN indexes on migration, to avoid OOM from oversized graphs, and offers the setting to override before migrating **[MGD]**. Deletion is an ISM action: a policy can, for example, "move your index into a `read_only` state after 30 days and then delete it after a set period of 90 days", optionally notifying on deletion **[OS]**.

Two further behaviours that surprise production changes: **a red cluster stops the clock**, since ISM does not run jobs when cluster state is red **[OS]** — a compliance issue, not just an operational one (§12.2); and on the managed service, **cold storage requires warm enabled on the same domain plus dedicated master nodes**, cold indexes are queried by attaching them to existing UltraWarm nodes, a deleted cold index can be restored only to warm before migrating back, and the service "retains cold indexes for 14 days after they've been deleted" **[MGD, cold-storage]**.

### 7.6 The honest economics

Two levers dominate the cost of an OpenSearch estate, and **neither is an engineering decision**. **Retention** — how long data must be kept — is set by a business requirement or a regulator (§12.2); once fixed it multiplies volume, shard count, tier mix and snapshot footprint, and engineering can make a fixed retention cheap but cannot make a long retention free. **Tiering policy** — how quickly data drops to a cheaper tier and what query latency the business will accept for older data — is a service-level negotiation: the mechanism is free, the decision is political.

Everything else (instance families, gp2 versus gp3, cache sizing, replica counts) is engineering optimisation against a retention-and-tiering decision already made. **A capacity plan that optimises storage types before fixing retention and tier boundaries is optimising the small term.** The useful corollary for a design review: if the cost is wrong, look at the retention rule and the tier boundaries first, and only then at the hardware catalogue.

---

## 8. Ingest and Indexing Capacity

### 8.1 The ingest-pipeline cost

Ingest is not a passive write. OpenSearch's documentation frames the ingest node as one that "pre-processes data before storing it" and "runs an ingest pipeline that transforms your data before adding it to an index", with the standing advice that heavy ingest and complex pipelines justify dedicated ingest nodes, which "can also optionally offload your indexing from the data nodes so that your data nodes are used exclusively for searching and aggregating" **[OS, cluster page, retrieved September 2026]**. Three capacity consequences follow: **pipeline CPU is a first-class line item**, since parsing, extraction and enrichment consume CPU and latency a naive documents-per-second estimate omits; **pipeline latency adds to backpressure**, because a slow pipeline slows the ingest thread pool, raising queue depth and producing the classic self-inflicted rejection incident (§8.5); and **field extraction affects storage, not just CPU** — what the pipeline emits is what gets indexed, so pipeline design and mapping design meet here, and mapping is owned by [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md).

### 8.2 Bulk-indexing discipline

The documented `_bulk` facts **[OS, index-management page, retrieved September 2026]**:

- **Two APIs exist**: the Index API for incremental arrivals and the `_bulk` API for less frequent, larger flows — for large numbers of documents, "lumping requests together and using the `_bulk` API offers superior performance. If your documents are exceptionally large, however, you might need to index them individually."
- **Format is strict**: a newline at the end of every line, including the last, with the action-and-metadata line followed by an optional document line.
- **Partial failure is silent by design**: if any action fails, OpenSearch continues with the others, so detecting failure falls to `items` in the response, in the order of the requests — a plan that ignores this sizes for throughput the cluster does not achieve, because failed items are re-sent.
- **Auto-generated IDs cost you updates**: without an explicit ID you "can't easily update the document at a later time", and re-running the same request indexes the document repeatedly — duplicate documents inflate volume and storage.
- **Bulk size is found by experiment with a documented start**: "start with a bulk request size of 5 MiB to 15 MiB. Then slowly increase the request size until the indexing performance stops improving."
- **Document `_id` is bounded**: "the document `_id` must be 512 bytes or less in size" — a real constraint for pipelines building composite keys.

### 8.3 Refresh and durability settings, with their trade-offs

These settings together are the main throughput-versus-safety dial on the ingest path, and every one has a documented downside — which is what makes them capacity decisions rather than tuning preferences.

| Setting | Default / guidance | Gain | What you give up |
|---|---|---|---|
| `index.refresh_interval` | 1 s default; refreshes only indexes searched in the last 30 s; increasing recommended to prevent **429 errors**; `30s` or disable in pure indexing **[OS]** | Fewer, larger segments; higher throughput | Search freshness |
| `index.translog.flush_threshold_size` | 512 MB default; **25% of heap** suggested for pure indexing; page example 1024 MB for instances with more than 32 GB of memory **[OS]** | Fewer flushes; larger segments that merge less often, leaving more threads for indexing | The docs warn translog completion takes longer and "if a shard fails, then recovery takes more time because the translog is larger" **[OS]** |
| `index.number_of_replicas` | Default 1; set to **0** for heavy indexing, then reactivate **[OS]** | Each replica duplicates the indexing process, so zeroing them improves throughput | "If a node fails while replicas are disabled, you might lose data" **[OS]** — a risk acceptance, not a tweak |
| `index.translog.durability` | `request` referenced as the mode under which remote-backed storage can recover to the last acknowledged write "regardless of replica count" **[OS]** | Durability | Throughput |
| Index buffer size | 10% of JVM memory default; up to 25% for heavy indexing **[OS]** | Ingest throughput | Heap headroom for search |

**The rule:** every ingest-throughput gain here is paid for in freshness, recovery time, heap headroom or durability. A capacity plan listing the optimisation without the corresponding service-level concession is incomplete.

### 8.4 Merge pressure and its resource cost

Segments are immutable; updates and deletes are handled by writing new segments and merging later, so merges are a permanent consumer of CPU and I/O whose cost is proportional to how badly the segment layout is managed. Documented handles **[OS, indexing-tuning]**: `max_merge_count` bounds concurrent merges and "further merges will wait until a merge thread becomes available" when the thread maximum is reached; where index throttling is the issue, the docs suggest considering more merge threads; and the refresh and translog settings matter *because* of merges, since larger flushes "create fewer large segments instead of multiple small segments", and "large segments merge less often, and more threads are used for indexing instead of merging". Force-merge is the blunt instrument for read-only data — and note the direction of the managed-service trade, where single-segment force-merge is deliberately *disabled* for k-NN migrations to avoid OOM from oversized graphs **[MGD]**: a merge that helps query performance can make a tier migration fail. **Merge capacity is invisible in a QPS estimate and visible in a node's CPU graph**, which is one reason a small cluster can saturate while reporting modest query load.

### 8.5 Backpressure and queue behaviour

Backpressure is how the cluster refuses work, and a planner should know the documented signals because they are how an under-sized ingest path announces itself: **HTTP 429 rejections**, whose avoidance is the stated rationale for raising the refresh interval **[OS]**; **thread-pool saturation**, for which the platform provides a dedicated logging category, `org.opensearch.threadpool`, described as "helpful for understanding queuing and pool saturation" **[OS]**; **search backpressure**, present in the Cluster Settings API's documented example as a `search_backpressure` block with `mode: monitor_only` **[OS]** — meaning the default in that configuration does not shed, so confirm the mode before assuming protection; **circuit-breaker rejections**, i.e. `circuit_breaking_exception` from the request, field-data and script-compilation breakers **[OS]**; and the presence of **admission control** and **workload-group scheduling** settings, shown in that same response with a `wlm.workload_group.mode` of `disabled` **[OS]** — facilities that exist to protect workloads from each other. The design discipline: decide in advance whether the cluster should **shed** (reject early, keep survivors fast) or **queue** (accept, absorb latency) when overloaded, and configure toward it. A cluster that does neither has taken the worst of both, because it will do whatever its defaults happen to do under a load it was not sized for.

### 8.6 Sizing ingest independently — and the contention you cannot size away

Ingest is sized with its own arithmetic: target documents or MiB per second at peak, divided by a *measured* per-node ingest rate for the same pipeline and document shape (OpenSearch Benchmark, §3.5), plus headroom for merges and for the chosen refresh interval. That is a genuinely separate calculation and belongs as its own line in the sizing document. **But it is not separate hardware.** In self-managed OpenSearch, ingest and search contend for CPU, disk I/O, page cache and above all heap; the managed service's own suggestion that lower-priority indexes be moved to a warmer tier to "provide isolation of resources consumed between the queries of low and high priority indexes so they don't impact each other" **[MGD, ultrawarm]** is an acknowledgement of the same contention, solved with tiers rather than promises.

Three honest conclusions. **Sizing ingest and search separately is correct method and insufficient practice** — the two rates must also be tested together (§3.5, test 4). **Isolation is the only real answer at scale**: dedicated ingest nodes, search-role nodes for search replicas, or separate clusters (§9.6). And **a cluster serving both a periodic bulk load and interactive search will show its worst latency during the bulk load** — not a tuning failure but the contention being visible, so plan the window or separate the workload.

---

## 9. The Scaling Model and the Real Limits

### 9.1 Vertical scaling, and where it stops

Vertical scaling means a bigger node. It works until one of four ceilings arrives, and only one is about CPU.

| Ceiling | Why it stops | Basis |
|---|---|---|
| **Managed heap quota** | Java process memory is limited to 50% of total memory up to 32 GiB; the cap does not apply to r7g and OpenSearch-optimized instances, but heap above 32 GB is not automatic and requires Auto-Tune or a support case **[MGD]** | A vertical plan crossing 32 GiB of heap becomes procurement, not configuration |
| **Heap-versus-cache equilibrium** | Past the point where heap has taken the memory the page cache needs, more heap makes searches slower (§6.1–§6.2) | The 50% rule and its reason |
| **Storage per node** | Managed EBS volumes have documented maximum sizes per instance type; instance-store families have fixed local capacity **[MGD]** | Beyond it, the only directions are more nodes or fewer/smaller shards |
| **Shard density per node** | More data on a node means more shards on it; the 1,000-shard default and the 2.17+ rule of 1,000 shards per 16 GB of heap up to 4,000 are hard ceilings **[OS]/[MGD]** | Adding disk does not lift the shard ceiling |

**Vertical scaling is the right first move for a cluster with too few shards and the wrong move for one with too many.** If the problem is shard density or the shard-to-heap ratio, a bigger node relocates the limit rather than removing it.

### 9.2 Horizontal scaling, and where it stops

Adding nodes works while three conditions hold: **shards are granular enough to spread** (a cluster of six 200 GiB shards cannot distribute itself across twelve nodes); **the shard count divides usefully across the new node count** — the documented balance rule of shards per index as a multiple of data nodes **[OS]**, with zone-aware placement wanting the node count in zone multiples **[OS]**; and **recovery and rebalance time stay inside the availability window** (§9.5). It stops at the managed service's documented node limits: per-AZ data-node maxima of 334 hot/250 warm at one AZ, 668/500 at two and 1002/750 at three, with the overall limit applying to hot + warm combined, plus per-instance-family totals that differ by version and carry a lower default requiring a quota increase **[MGD, limits]**. Those are dated, catalogue-specific figures, but the lesson generalises: **every managed service has a node ceiling, and it is discoverable before you hit it.**

### 9.3 The cluster-state limit and the ceiling it implies for shard count

Cluster state holds index and shard metadata and is published to cluster-manager-eligible nodes, which makes shard count a *cluster-manager* problem long before it is a storage problem. The verified numbers:

| Constraint | Value |
|---|---|
| `cluster.max_shards_per_node` default | **1,000** — per the Cluster Settings API's documented example values (the example sets 500 and shows 1000 as the transient value) **[OS]** |
| Elasticsearch 7.x | 1,000, changeable via the setting **[MGD, limits]** |
| OpenSearch 1.x to 2.15 | 1,000, changeable via the setting **[MGD, limits]** |
| OpenSearch 2.17 and above (managed) | **1,000 shards per every 16 GB of heap, up to a maximum of 4,000 per node** — and "the default limit can't be changed" **[MGD, limits]** |
| Design guidance | No more than **25 shards per GiB of Java heap** **[MGD, bp-sharding]** |

Two consequences. **The 2.17 change is a genuine unlock for large-shard-count estates** — a node with 32 GB of heap can host 2,000 shards instead of 1,000 in a managed domain — and it is version-gated, so the same domain sizes differently before and after 2.17 (§2.4). And **the guidance binds far earlier than the hard limits**: anyone near 1,000 shards per node is already wrong, and the managed service's own sanity check makes the point — "if you size your shards appropriately, you typically run out of disk space long before encountering this limit", with its illustration that an `m6g.large.search` instance (documented 512 GiB maximum disk) can hold roughly 20 shards of 20 GiB below 80% disk usage **[MGD]**.

**The ceiling, stated plainly:** total primary shard count is bounded by `nodes × shards-per-node-limit`, and that product caps how far a single cluster can be scaled by sharding alone. When the plan needs more shards than the ceiling allows, the answer is fewer shards (§4.6), a second cluster (§9.6), or a different store (§9.6) — not a bigger cluster.

### 9.4 The hot-shard and skew problems

Even distribution is an assumption, and it is frequently false.

| Skew | Cause | Symptom | Remedy |
|---|---|---|---|
| **Routing** | A custom routing key (tenant, account) concentrates documents; the sibling modelling guide's §6.2 notes skewed routing causes hot shards | One node at high CPU/IO while peers idle | Re-evaluate the routing key or shard by a higher-cardinality key; a reindex is usually required because shard count is fixed |
| **Shard-count** | A count that does not divide evenly across nodes or zones | A node carrying 9 of 26 shards (§4.5's illustration) | Choose counts that divide; scale in zone multiples |
| **Size** | Uneven data per shard from non-uniform documents or non-uniform time buckets | A few large shards dominating recovery and merge cost | Roll over on size, not only age (§4.6) |
| **Query** | A query pattern hitting one index or tenant disproportionately | Periodic latency spikes correlated with a report or tenant | Separate the heavy query path, or budget for it explicitly |

The platform acknowledges the general problem in its settings — `cluster.routing.allocation.load_awareness.flat_skew` appears in the Cluster Settings API's own example response **[OS]**, a load-awareness facility for rebalancing away from loaded nodes. But **load-awareness settings treat the symptom; they cannot fix a routing key that puts 40% of the data on one shard.**

### 9.5 Rebalancing and recovery cost at scale

Every scaling event has a recovery-shaped tail. **Adding nodes moves shards** — the managed service documents that as you add instances OpenSearch "automatically rebalances the distribution of shards throughout the cluster" **[MGD]**, which is both mechanism and cost. **Losing a node re-replicates every shard it held**: with many shards that is many small units, with few large shards it is few huge ones, and §4.5's arithmetic is how to see which you have. **Translog size directly extends recovery time** — "if a shard fails, then recovery takes more time because the translog is larger" **[OS]** — so an ingest decision from §8.3 reappears here as an availability number. **Restores are incremental but not free**: snapshots "only store data that has changed since the last successful snapshot", frequent snapshots often cost little extra disk and complete faster, and some users snapshot as often as every 30 minutes **[OS]** — making snapshot cadence a recovery-time decision as much as a storage one. And **snapshot deletion must go through the API**: "if you need to delete a snapshot, be sure to use the OpenSearch API rather than navigating to the storage location and purging files", because incremental snapshots share data and the API removes only what no other snapshot needs **[OS]** — a retention policy that deletes objects directly corrupts the repository.

### 9.6 When the architecture, not the hardware, is the constraint

Past a certain point, adding nodes or disks makes the system worse — more shards to coordinate, more cluster state to publish, longer recoveries, more expensive snapshots — and no larger instance fixes it. The signals:

1. **Shard count is at or near the ceiling while disk is not full** (§9.3). The remedy is fewer shards, not more nodes.
2. **Two workloads with incompatible service levels share one cluster.** Interactive search and bulk analytics want different refresh intervals, replica counts, heap profiles and tier policies. At some scale the honest answer is **two clusters**, joined by cross-cluster search or an application-level fan-out. The managed service's tier-based isolation suggestion **[MGD]** is the same idea inside one cluster, and it has limits.
3. **The query pattern, not the data volume, is the problem.** Deep pagination, unbounded aggregation or high-cardinality terms aggregation is a schema-and-query problem owned by the modelling guide, not a capacity problem this guide can size away.
4. **The workload is a warehouse in disguise.** A log estate written once and read with heavy analytical aggregations may be cheaper and faster in a columnar store — see [clickhouse_guide.md](clickhouse_guide.md) for that alternative and [dragonflydb_guide.md](dragonflydb_guide.md) for the in-memory-datastore alternative. OpenSearch is excellent at search, time-series retrieval and moderate analytics; it is not a substitute for a columnar warehouse at warehouse scale.
5. **The durability model is the constraint.** If the requirement is zero-loss recovery regardless of replica count, the architecture must be chosen at bootstrap (remote-backed storage, §7.3), not sized.

**The honest statement:** the architecture becomes the constraint when the shard count, workload mix, query shape or durability requirement — rather than CPU, disk or node count — is what fails. At that moment capacity planning's job changes from "how much" to "how many clusters, and what belongs in each".

---

## 10. The Cost Model

### 10.1 The components of the bill

An OpenSearch estate's bill has seven lines, and only the first two are usually modelled.

| Component | Driven by | Scales with |
|---|---|---|
| **Hot-tier compute** | Data nodes running searches and indexing | Data volume, query load, replica count, heap generosity, node count |
| **Hot-tier storage** | Attached block volumes or instance-store families | Volume × (1 + replicas) × overhead; provisioned IOPS/throughput if provisioned |
| **Warm/cold-tier compute and storage** | Warm nodes and their cache, plus object-storage bytes below the hot tier | Volume held below hot; the storage-to-cache ratio on warm nodes |
| **Snapshot / backup storage** | Repository bytes | **Not** the full data volume — snapshots are incremental, so the marginal cost of frequent snapshots is often small **[OS]**; still grows with retention and churn |
| **Inter-zone and cross-region traffic** | Replication across zones, cross-cluster replication, cross-region restores, client traffic crossing zones | Replica count, topology spread, restore frequency, DR design — the line most often omitted from a first model |
| **Egress and client traffic** | Requests leaving the region or provider network | Query volume and response size; the tuning docs' "reduce response size" advice **[OS]** is a cost lever as well as a latency one |
| **Managed-service premium** | The operated-service component: patching, snapshots, Multi-AZ, dedicated master nodes, Auto-Tune, support | Domain configuration — a premium buys operational labour you would otherwise staff, usually a deliberate trade in a regulated estate |

Two structural notes. **The managed service reserves 20% of each instance's storage up to 20 GiB for its own overhead** **[MGD]** — capacity you pay for and cannot use, and the reason the service's simplified storage formula carries a 1.45 multiplier (§3.2). And **managed warm nodes have hard storage ceilings** (documented at 1.5 TiB and 20 TiB for the UltraWarm instance types, and 5× cache for OI2) **[MGD]**, so warm capacity is bought in node-shaped increments, not by attaching another volume.

### 10.2 How each cost line scales with the decisions in §4, §6 and §7

| Decision | Cost consequence | Direction |
|---|---|---|
| Shard size target (§4.2) | Smaller shards → more shards → more heap per unit of data → more nodes for the same heap budget, plus more cluster state and recovery units | Over-sharding is a *hardware* cost, not just a performance cost |
| Shard-count ceiling (§9.3) | Hitting the per-node ceiling forces additional nodes even when disk and CPU are idle | Cost with no workload reason behind it |
| Rollover design (§4.6) | Enables tiering and deletion at whole-index granularity — which is what makes cheap tiers usable | The cheapest architectural decision in the guide |
| Heap size (§6.1) | Heap is capped by RAM and the managed 32 GiB quota; the *node size* needed follows from heap plus cache plus OS | Over-sized heap does not itself cost more; over-sized **instances** do |
| Node roles (§5) | Dedicated ingest, coordinating, search and warm nodes each add nodes carrying no storage | A resilience and isolation cost paid in node count |
| Dedicated cluster managers (§5.2) | Three small nodes | The cheapest high-value spend in the design |
| Tiering policy (§7.6) | Moving N% of bytes from hot to warm/cold multiplies the cheap-tier share | The largest available reduction in a steady-state bill |
| Retention (§7.6) | A linear multiplier on every storage line | The largest *absolute* number, and a policy variable |
| Refresh and translog settings (§8.3) | More aggressive ingest settings can *reduce* the node count needed for a fixed ingest rate | An ingest win is a cost win |
| Remote-backed storage (§7.3) | Adds object-storage cost, reduces write amplification on the replica path | A durability purchase with a capacity side-effect |

### 10.3 A worked comparison of two sizing strategies

**Everything in this subsection is illustrative.** Unit costs are expressed as relative indices, not currency: no figure here is an AWS list price, and any real comparison must be rebuilt from the provider's current pricing page and instance catalogue on the date of the decision. The service levels assumed are identical in both strategies — same data, retention, latency objective, and resilience (one replica, three zones).

**Scenario (illustrative):** 60 TiB of primary log data, 90-day retention, one replica, hourly ingest with a peak window, interactive search over the last 7 days and occasional analytical queries over 90 days.

| Line | Strategy A — hot-heavy | Strategy B — tiered |
|---|---|---|
| Hot tier | All 120 TiB (primary + replica) on hot nodes | 30 TiB hot (last ~7 days plus headroom); 90 TiB warm/cold |
| Warm/cold tier | — | 90 TiB on object-backed tiers, replicas per tier policy |
| Shard strategy | Daily rollover at ~30 GiB/shard | Same |
| Snapshot repository | Daily incrementals per policy | Weekly full plus daily incrementals |
| Dedicated cluster managers | 3 small nodes | 3 small nodes |
| Ingest capacity | Dedicated ingest nodes sized to peak | Same (ingest rate unchanged) |
| Relative compute cost | 1.00 (index) | ~0.45–0.60 of A's hot-tier compute, plus warm-node compute |
| Relative storage cost | 1.00 (index) | Substantially lower — most bytes move to object storage |
| Latency for 60–90-day-old data | Same as hot | Slower and more variable: the documented "slower and longer snapshot searches" **[OS]** |
| Operational complexity | Low | Higher: ISM policies, tier transitions, cache to size, migrations to monitor |
| Risk | Steady-state cost grows linearly with retention | A tier policy that moves data a query cannot tolerate (§14, anti-pattern #5) |

**How to read it.** Strategy B is not "better" — it is cheaper at the same service level *only if the service level genuinely permits slower access to old data*. If the business requires all 90 days at one latency, B's tiering is a service-level breach disguised as a saving and A is correct. **The cost model's job is to price the service level, not to minimise the number** — the same argument as §7.6, in money: retention and tiering are policy decisions, and the engineer's contribution is to state the price of each option clearly enough for someone else to choose. Two further points: **retention dominates**, since doubling it roughly doubles both strategies and the absolute cost of the retention rule far exceeds the difference between them, so negotiate retention before optimising architecture; and **the warm tier is not free**, because it needs warm-node compute and cache, and a tier policy that pushes data to warm without shrinking the hot working set produces both costs without removing hot nodes.

### 10.4 Labelling discipline for cost figures

Every cost figure entering a decision document carries four things: the **unit** (node-hours, GiB-months, request count), the **date checked**, the **source** (provider pricing page, contract rate, internal chargeback rate), and the **assumption set** (instance families, purchase options, retention, replica policy). A figure missing any of the four is not evidence; it is a number. ⚠ All §10.3 figures are deliberately illustrative indices, not sourced to a pricing page, because prices change continuously and a stale price is worse than no price.

---

## 11. Monitoring and the Feedback Loop

### 11.1 Heap pressure and GC behaviour

The two managed-service metrics its own sizing page names as evidence of an inadequate configuration are **`CPUUtilization`** and **`JVMMemoryPressure`**: "if performance isn't acceptable, tests fail, or `CPUUtilization` or `JVMMemoryPressure` are high, you might need to choose a different instance type (or add instances) and continue testing" **[MGD, bp-instances, retrieved September 2026]**. Those two names are the minimum viable alerting set on a managed domain. On a self-managed cluster the equivalents come from the platform's own surfaces, and OpenSearch's logging documentation is explicit that the categories exist for this: `org.opensearch.threadpool` is "helpful for understanding queuing and pool saturation", and `org.opensearch.indices` is the category to enable "during shard recovery and indexing pressure analysis" **[OS, logs page, retrieved September 2026]**.

| Signal | Indicates | Capacity action |
|---|---|---|
| Heap utilisation trend at flat load | Working set outgrowing the node | Re-examine shard count first (§4.3), then heap and node size |
| GC frequency and pause duration rising | Closer to the breaker wall; latency will follow | Add headroom, or reduce per-shard memory cost |
| Off-heap / native memory on vector nodes | Pressure invisible to heap dashboards | Track node-level memory, not just JVM heap (§6.2) |
| Breaker rejections | The exact point at which the cluster refused work | Sizing evidence, not noise (§11.3) |

### 11.2 Latency percentiles

Percentiles, not averages: averages hide the tail that violates a service level, and the generic treatment belongs to [capacity_sizing_guide.md](capacity_sizing_guide.md). OpenSearch supplies the instrumentation — **search request slow logs** and **shard slow logs** are documented categories alongside task logs and deprecation logs **[OS, logs page section list]**, and slow logs are how you catch the query whose multi-second p99 is invisible in a dashboard of averages; **task logs** show long-running tasks, the mechanism behind a latency spike caused by one pathological query rather than a capacity deficit; **per-node `_cat/nodes?v`** is the documented check for confirming node roles and cluster formation **[OS]**; and **per-index and per-shard sizing** comes from `_cat/indices?v` with the `pri.store.size` value, summarised by `_cat/allocation?v` — the exact pair the managed service documents for confirming actual indexing overhead against the 110% assumption **[MGD]**.

### 11.3 Rejection and circuit-breaker events

Treat every rejection as a data point. The verified surfaces: **HTTP 429** responses, whose avoidance is the stated reason for the refresh-interval recommendation **[OS]**; **`circuit_breaking_exception`**, documented for the script compilation breaker specifically and produced by the request and field-data breakers in general **[OS]**; **thread-pool queue saturation**, for which the platform provides a dedicated logging category **[OS]**; and **search-backpressure events**, if the backpressure mode is configured to act rather than monitor — the documented example shows `search_backpressure.mode: monitor_only`, meaning that configuration does not shed by default **[OS]**. The planning use: a rejection rate that is non-zero at normal peak is a sizing failure; one that is non-zero only during an abnormal event is a *behaviour* and should be documented as such. Both belong in the monitoring record, and the distinction is a standing question in a design review.

### 11.4 Disk watermark states and their consequences

Disk watermarks are thresholds on disk usage that change cluster behaviour — typically reducing allocation as usage rises and ultimately making indexes read-only. The commonly cited defaults are around 85% (low), 90% (high) and 95% (flood stage), but **⚠ these values were not re-verified from OpenSearch's own documentation in this pass** and should be confirmed against the configuration reference for the version in use. What matters more than the exact numbers is verified: **the managed service recommends operating below 80% disk usage** in its own worked example of shard capacity per instance **[MGD]** — a stricter operating convention than the watermark thresholds, which is the right way round. **Watermarks change semantics, not just performance**: above the flood stage the cluster may stop accepting writes to affected indexes, which is the difference between "slow" and "down", and why disk headroom is a separate headroom from heap headroom (§3.4). And **watermark states are a retention and shard-size symptom** — a cluster repeatedly reaching the high watermark while shard sizes are also above target is telling you the rollover design (§4.6) or the retention rule (§7.6) needs revisiting, not that the disks need to be bigger.

### 11.5 Shard-size drift

The plan assumes a shard size range; only monitoring says whether reality matches it.

| Drift | Detection | Remedy |
|---|---|---|
| Shards growing past target because rollover is not firing | `_cat/indices?v` / `pri.store.size`, trended per rolled-over index | Check ISM job health — and note that a red cluster stops ISM entirely **[OS]** |
| Shards far smaller than target because too many primaries were created | Same measurements against the intended size | Reduce primaries for *new* indexes so the drift stops spreading; older indexes need a reindex |
| Shard count per node climbing towards the ceiling | Node-level shard counts against `cluster.max_shards_per_node` **[OS]** and the 2.17+ managed rule **[MGD]** | Fewer shards, not more nodes (§9.3) |
| Index count growing from too-fine a rollover cadence | Index counts and cluster-state size trended | Coarsen the rollover period: the retention requirement has not changed, only the granularity |
| Skew between nodes | Per-node shard counts and disk usage | §9.4 |

**A shard-size dashboard is the single highest-value OpenSearch capacity instrument**, because it detects all four of §4's failure modes — over-sharding, under-sharding, growth-without-rollover and skew — before any becomes an incident.

### 11.6 Load-test cadence, and what triggers a re-plan

| Trigger | Test to run | Why |
|---|---|---|
| Before admitting a new workload class | That workload's own OpenSearch Benchmark workload | A new class invalidates the old per-node capacity number |
| Before a version upgrade | The existing workload, before and after | The fork's divergence means behaviour can change (§2.4) |
| When any §4 input changes | Full search + ingest test | Retention, shard size target and replica count changes are capacity changes |
| Quarterly, or after a >20% change in any sizing input | Steady-state replay at peak | Catches drift the dashboards smooth over |
| After any rejection, breaker or watermark incident | Reproduction at the incident's concurrency | Turns an incident into a measurement |
| After any scaling event | Verification run at peak | Confirms the new steady state and the new recovery time |

**And the honest conclusion of the whole section: capacity planning is a loop, not a deliverable.** The shard strategy and the heap rule are decided once and then defended; the node count, tier boundaries, retention window and cost are re-planned. The signals that force a re-plan are measurable and few: (1) heap utilisation or GC implying the working set has outgrown the node; (2) rejections or breaker events at normal peak; (3) disk watermark states during normal operation; (4) shard sizes drifting outside the design range; (5) cluster state or shard count per node approaching the ceiling; (6) a change to retention, the latency objective, or the workload mix; (7) a version change that alters a documented behaviour (§2.4). Anti-pattern #7 (§14) is the cluster re-planned only after an incident, and this list is the remedy: **re-plan on a signal, not on a failure.**

---

## 12. The Regulated-Enterprise and Banking Angle

A regulated estate does not get to optimise freely: several of this guide's most powerful levers — retention, tier placement, replica counts, where data physically sits — are constrained before the engineer arrives. The honest framing is that the regulator sets the boundary conditions and the engineer sizes inside them.

The resilience half of this section is **cross-referenced, not re-derived**: [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (*Operational Resilience: The Resilient Bank* — the repo's operational-resilience anchor, covering impact tolerances, important business services, BIA, BCP and DR tiers and the RTO/RPO machinery) owns what "resilient" means as a set of requirements; [../management/resilience_engineering_guide.md](../management/resilience_engineering_guide.md) owns the engineering-discipline half; and [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) is a worked DR design in the same repo. What follows is only the OpenSearch-specific part: **which of those requirements land on a capacity plan.**

### 12.1 Data residency and where each tier physically sits

The capacity decision that looks purely technical and is not: **tiering moves data between physical locations, and a tier's location is a residency fact.** A hot tier in region A and a warm tier backed by object storage in region B is cross-border data movement whatever the diagram calls it; if residency says the data stays in jurisdiction A, the warm/cold tier must be implemented there too, removing the option of the cheapest available object-storage location. **The managed tier model makes this explicit**, since warm and cold storage are backed by object storage (Amazon S3 in that service's case) while hot tiers use attached volumes or instance store **[MGD]** — different physical substrates with different control sets to evidence. **Snapshots are a copy in another place**: a repository is a shared file system, Amazon S3, HDFS or Azure Storage **[OS]**, so "backup to another region" and "backup inside the jurisdiction" are different designs with different costs. The pragmatic rule: draw the residency boundary on the architecture diagram, place every tier and repository inside it, and record which tier lives where as a governance artefact rather than an operational detail.

### 12.2 Retention: the requirement that sizes the estate

Retention is the largest single input to cost and shard count, and in a regulated firm engineering does not get a vote on the number — only on how cheaply a fixed retention can be served.

| Retention decision | Origin | Capacity consequence |
|---|---|---|
| How long data must be kept | Records-retention rules and the firm's schedule | Linear multiplier on every storage line (§7.6, §10.3) |
| Whether it must be *queryable* throughout, or merely *retrievable* | The specific obligation: an investigation power implies retrievability; an ongoing analytical need implies queryability | Tier policy — retrievable-only allows the coldest tier; queryable implies a latency-bearing tier |
| Whether it must be *immutable* | Evidentiary and audit obligations | Read-only tier semantics; searchable-snapshot indexes are inherently read-only, which fits **[OS]** |
| Whether deletion must be *provable* | Privacy and data-minimisation obligations | Deletion must be an auditable policy action — an ISM state that deletes **[OS]** — not an operational script |

**The compliance trap a capacity plan must catch: ISM does not run jobs when the cluster state is red** **[OS]**. If retention enforcement, including deletion, is implemented as an ISM policy — the documented mechanism **[OS]** — then a red cluster is also a retention-policy *failure*, silently. A plan for a regulated estate should therefore state that shard and node sizing has a compliance consequence, not merely a performance one, and that cluster health is part of the retention control set.

### 12.3 Encryption and key management for the store

Encryption at rest and in transit are platform features rather than capacity features, but key management adds latency and operational dependencies that affect tier choice: a tier whose data is encrypted under a key held in a different jurisdiction or account creates a dependency the availability design must reflect. **Repository encryption matters for snapshots**, because the repository is a *copy* of the data and inherits its classification. And the access-control model reaches into tier operations: on the managed service, users making UltraWarm API calls must be mapped to the `ultrawarm_manager` role and users managing cold indexes to `cold_manager` under fine-grained access control **[MGD]** — so tiering automation needs an identity with the right role and an audit trail for its actions. Platform support for encryption is not a substitute for a decision about the key: who holds it, what happens when it is unavailable, and whether the store can be opened without it are resilience questions owned by the security and resilience guides, and the capacity plan records the dependency.

### 12.4 Audit and access control

Auditing an OpenSearch store in a regulated estate involves at least four surfaces: **who can query what** — index-level access control, so residency or need-to-know boundaries are enforced in the store and not only in the application; **what was done to the cluster** — audit events for index creation, deletion, mapping changes and policy changes, which is where the capacity plan's decisions become auditable facts, since a retention policy is evidence and so is the shard layout supporting it; **what the cluster did** — deprecation, slow, task and error logs are documented categories **[OS]**, with slow logs the record of query behaviour a capacity investigation needs; and **which identity performed tier operations** — the `ultrawarm_manager` and `cold_manager` mappings **[MGD]** being the managed example of tiering as a privileged, auditable action. A capacity plan that changes shard layout, tier boundaries or retention policy should route through the same change record as any other control change, because in a regulated estate those changes *are* control changes.

### 12.5 Resilience expectations: backup, restore and cross-region behaviour

Requirements arrive as RTO and RPO targets (owned by the resilience guides above); the OpenSearch-specific conversation is which mechanisms can meet them.

| Requirement | Mechanism | Capacity consequence |
|---|---|---|
| Point-in-time backup | Snapshots to a registered repository — shared file system, Amazon S3, HDFS or Azure Storage; incremental, so frequent snapshots cost little extra and complete faster **[OS]** | Snapshot storage grows with churn and retention; cadence is a recovery-time decision (§9.5) |
| Restore of a deleted or corrupted index | Restore from the repository through the API **[OS]** | A restore consumes cluster resources while it runs — a DR drill is a load test |
| Zero/minimal loss regardless of replica count | Remote-backed storage, requiring segment replication, settable only at bootstrap **[OS]** | Must be chosen at design time; adds object-storage cost and changes the write path |
| Recovery of a red cluster or red indexes | Remote-backed storage is documented for restoring red clusters or indexes, recovering to the last acknowledged write with `index.translog.durability: request` **[OS]** | An availability mechanism purchased as capacity |
| Keeping older data queryable without hot-tier cost | Searchable snapshots from a `warm` node (as of 3.0), at the documented cost of slower, longer snapshot searches **[OS]** | Tier design participates in the DR design, not only the cost model |
| Zone-outage tolerance | Replica placement across zones with shard allocation awareness, forced awareness and replica count enforcement **[OS]** | Replica count is a storage multiplier (§3.2); zone count constrains how redundancy distributes (§5.3) |
| Cross-zone or cross-region standby (managed) | Multi-AZ with Standby, which also constrains warm-node counts to multiples of the AZ count **[MGD]** | A topology constraint to absorb at design time, not discover |

One nuance routinely missed: these mechanisms move **copies**, and copies consume capacity in the destination as well as the source. A standby region sized at a fraction of production is a valid DR target only if the failure scenario it is sized for is the one that actually occurs — the analysis the resilience guides own and this guide does not repeat.

### 12.6 Which planning decisions a regulator effectively fixes

| Decision | Who effectively decides | What the engineer still controls |
|---|---|---|
| **How long data is retained** | The regulator and the firm's retention schedule | The cost of serving that retention: tiers, shard strategy, replica policy |
| **Where data may physically sit** | Residency rules | Which storage substrate satisfies residency inside the permitted envelope |
| **Whether data may be deleted early to save money** | Forbidden by the retention rule | Nothing — early deletion is not an optimisation, it is a breach |
| **What must remain retrievable versus queryable** | The specific obligation | Whether retrieval is a cold-tier query or a repository restore |
| **Backup and recovery expectations** | RTO/RPO from resilience obligations | Which mechanisms meet them, at what cost (§12.5) |
| **Auditability of changes** | Change-control and audit obligations | Making shard, tier and retention changes auditable by design |
| **Node count, instance family, storage type, shard target, heap size, replica count** | **The engineer** | Everything — which is why this guide exists |

**A regulator fixes the boundary conditions — retention, residency, retrievability, resilience — and the capacity engineer sizes inside them.** A plan that treats retention as a variable has misunderstood its problem statement; one that treats instance family as fixed by compliance has misunderstood the other half. Knowing which is which is the whole job.

---

## 13. The Cymbal Bank Worked Example

**Cymbal Bank is a fictional bank — the only bank persona used in this repository.** The scenario, volumes, node counts and costs below are **illustrative constructions** produced to demonstrate the method. They are not figures from any real institution, no real bank is described, and no provider's list prices are quoted.

### 13.1 The scenario and its constraints

| Constraint | Illustrative value |
|---|---|
| **Workload A — internal search** | Internal document/reference search: ~4 M documents, ~6 KiB each, low churn, interactive latency objective |
| **Workload B — log analytics** | Application and infrastructure logs: ~24 GiB/day indexed, 90 days of active analysis, **7-year regulatory retention** |
| **Deployment** | Managed service (Amazon OpenSearch Service) for operational simplicity; three Availability Zones; small platform team, so simplicity is a real design input |
| **Resilience** | One replica in the hot tier; time-to-green after a single-node loss inside the firm's tolerance — set by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), not re-derived here |
| **Residency** | All data and repositories stay in the approved jurisdiction, so the warm/cold substrate must be an approved in-jurisdiction object store |

### 13.2 Workload characterisation

| Dimension | Workload A | Workload B |
|---|---|---|
| Primary class (§3.1) | **Search-dominated** — many small latency-sensitive queries, moderate fan-out | **Ingest-dominated** — continuous writes; search is retrieval, not relevance |
| Secondary class | Aggregation-light (facet counts) | Aggregation-heavy on recent windows (dashboards, error rates) |
| Freshness | Second-level expected | Minutes-to-hours acceptable for most consumers |
| Latency objective | Interactive (<~200 ms p95, illustrative) | Interactive for the last 30 days; **slower is acceptable for older windows** |
| Retention | Long-lived, occasionally updated | **7 years, regulatory** |
| Replicas | 1, zone-spread | 1 in the hot window; tier-appropriate copies below |

The tension is explicit: A wants low-latency search over a tiny corpus; B wants cheap bulk storage over an enormous one. They share a cluster only if tiering keeps them from contending.

### 13.3 Volume arithmetic (illustrative)

```
Workload A:  4,000,000 docs × 6 KiB = 22.9 GiB source
             × 1.1 overhead           = 25.2 GiB shard data
             × (1 + 1 replica)        = 50.4 GiB total store

Workload B:  24 GiB/day × 365        = 8,760 GiB/year ≈ 8.6 TiB/year
             × 7 years                ≈ 60 TiB of primary shard data
```

The tier split, driven by the retention rule and the latency concession:

| Window | Age | Primary volume | Tier | Query expectation |
|---|---|---|---|---|
| Active | 0–30 d | 24 × 30 = **720 GiB** | Hot | Interactive |
| Analytical | 31–180 d | 24 × 150 = **3,600 GiB** | Warm | Slower, acceptable |
| Retained | 181 d – 7 y | ≈ **55.7 TiB** | Cold / retrievable | Forensic and investigative only |

The shape is the point: **the hot tier holds about 1.2% of the 7-year corpus.** The 7-year figure is a compliance number; the 1.2% is the engineering achievement.

### 13.4 The shard decision at two candidate sizes

Worked for Workload B's 24 GiB daily index at two targets (§4.5 method: `primaries = 24 × 1.1 / target`):

| | **B1 — 30 GiB target (chosen)** | B2 — 6 GiB target |
|---|---|---|
| Primaries per daily index | 26.4 / 30 = 0.88 → **1 primary** | 26.4 / 6 = 4.4 → **5 primaries** |
| Shards per index (1 replica) | **2** | **10** |
| Hot-window shards (30 indexes) | 30 × 2 = **60** | 30 × 10 = **300** |
| Shards per node on 6 hot data nodes | **10** | **50** |
| Heap-equivalent of the hot window at ≤25 shards/GiB-heap | Trivial | 300 shards ≈ 12 GiB of heap-equivalent at the guidance ratio, before any query work |
| Query fan-out for a 30-day search | **30 shard targets** | **150 shard targets** |
| Shard size vs the 10–30 GiB latency-sensitive range **[MGD]** | 24 GiB equivalent — **inside** | ~4.8 GiB — **below**, i.e. the over-sharding condition |
| Recovery work per failed node | ~10 shards | ~50 shards |

**Decision: B1, one primary per daily log index.** B2 buys nothing: identical data, five times the fan-out, five times the heap cost, and a shard size pushed below the documented range — textbook over-sharding (§4.3). Rollover keeps each index at a stable healthy size permanently, so no shard count needs to be pre-provisioned for growth (§4.6).

**Workload A gets a different answer for a different reason.** At a 30 GiB target, 25.2 GiB yields one primary — but one primary is one query thread for a service expecting concurrency. Sizing for parallelism instead of size: `25.2 / 2 = 12.6 GiB per shard`, inside the range, so **two primaries with one replica**. That is the "give the workload what it needs" move of §4.7, safe here only because the corpus is small and shard size stays in range.

**Rollover and retention design:** size-based rollover as the primary trigger (a burst day rolls early, so shard size never exceeds target) with age as a safety net (a stalled ingest cannot freeze the write index), one write alias per log family, `ism_template` for automatic policy attachment **[OS]**, then ISM states transitioning `index.routing.allocation.require.temp` to warm at 30 days, to cold at 180 days, and deleting at 7 years (2,555 days) **[OS]**.

### 13.5 Heap and topology

| Decision | Choice | Reason |
|---|---|---|
| Heap | 50% of RAM, inside the managed 50%-of-memory / 32 GiB quota **[MGD]**; 16 GiB heap classes (32 GiB RAM class) on the hot tier | The OpenSearch-documented 50% rule **[OS]**; 16 GiB heap allows ~400 shards at the 25-shards-per-GiB guidance, versus the 10 per node this design needs |
| Cluster managers | **3 dedicated masters**, one per AZ | Documented managed prerequisite for warm and cold **[MGD]**; §5.2 quorum arithmetic |
| Hot data nodes | **6** (two per AZ) as the initial estimate | Zone multiples **[OS]**; 60 hot-window shards ÷ 6 = 10 per node, inside every per-node shard limit |
| Warm nodes | **3 or 6** — a multiple of the AZ count — sized by the warm storage ceiling, not by CPU | Documented Multi-AZ with Standby constraint **[MGD]**; warm nodes documented as needing more compute than storage proportionally **[OS]** |
| Coordinating nodes | Absorbed by data nodes initially; a dedicated layer deferred | The "couple of coordinating-only nodes" guidance targets search-heavy fan-out **[OS]**; Cymbal's fan-out is 4 shards, not 400 |
| Ingest nodes | Deferred until the pipeline is measured | A documented dedicated-ingest candidate **[OS]**, but pipeline cost is unknown until tested |
| Search-role nodes | Not used | Their purpose — separating search replicas from indexing — is already achieved by the tier split |

### 13.6 Tiering and lifecycle driven by the retention rule

The retention rule determines the policy; the policy determines the cost. In that order:

| Age | ISM state | Action | Bytes (primary) |
|---|---|---|---|
| 0–29 d | `hot` | None — active write index and recent reads | 720 GiB |
| 30 d | `warm_transition` | Set `index.routing.allocation.require.temp` to `warm` **[OS]** (or the managed migration) | — |
| 30–179 d | `warm` | Optional replica reduction; serve analytical queries | 3,600 GiB |
| 180 d | `cold_transition` | Migrate to cold storage **[MGD]** | — |
| 180 d – 7 y | `cold` | Retained, retrievable by attaching to warm nodes **[MGD]** | ≈55.7 TiB |
| 2,555 d | `expire` | Delete via policy — an auditable action (§12.2) | — |

Three documented behaviours the policy must respect: **cold requires warm enabled first and dedicated masters** **[MGD]**; **ISM does not run when cluster state is red** **[OS]**, so a red cluster delays a *regulatory* deletion schedule and cluster health is therefore a compliance control; and **a deleted cold index is retained 14 days by the service and restores only to warm first** **[MGD]**, which the firm should understand before calling a deletion irrevocable. With residency confining every tier and the snapshot repository to the approved jurisdiction, the cost plan starts from a constraint rather than a price.

### 13.7 The load-test plan

| Test | Data | Pass criterion (illustrative) |
|---|---|---|
| Search-only, Workload A | Representative extract of the 4 M-document corpus | p95 objective met at target concurrency |
| Ingest-only, Workload B | 3–7 days of representative log volume at peak rate | Target GiB/day sustained with no sustained 429s and no breaker events |
| **Mixed** | Both workloads at peak simultaneously | Both criteria met; A's p95 degradation inside tolerance |
| Recovery drill | Node loss on a hot/warm boundary | Time-to-green inside the resilience tolerance |
| Restore drill | Restore a 30-day-old index from the repository | Restore inside the RTO |
| Tier-transition drill | Force an ISM transition on a synthetic index | Data moves, query behaviour changes as expected, policy idempotent |

All run with **OpenSearch Benchmark** **[OS]**. The mixed run decides the topology, because it is the only test that measures the contention of §8.6.

### 13.8 The cost comparison and the recommendation

Two strategies at the same service level, using the relative indices of §10.3 — **illustrative, dated September 2026, not provider prices**:

| | Strategy A — all-hot | **Strategy B — tiered (recommended)** |
|---|---|---|
| Hot bytes carried for 7 years | ~120 TiB | ~1.4 TiB (720 GiB primaries + replica) |
| Warm/cold bytes | — | ~59.3 TiB |
| Hot data nodes | A large multiple of B's | 6, revisable after testing |
| Latency, 0–30 d | Same | Same |
| Latency, >30 d | Same | Slower — the documented searchable-snapshot cost **[OS]** |
| Relative steady-state cost | 1.00 (index) | A fraction of A, dominated by the cold tier's per-GiB rate |
| Operational complexity | Low | Higher: ISM policies, tier prerequisites, cache sizing, migration monitoring |

**Recommendation:** adopt Strategy B with the B1 shard design (one primary per daily log index), two primaries for the search index, three dedicated masters, six hot data nodes as an *initial* estimate, warm nodes in AZ multiples, and the 30-day/180-day/7-year ISM policy. Not because tiering is fashionable, but because Strategy A means paying the most expensive tier's rate for **98.8% of the estate** in order to keep forensic data interactive.

### 13.9 Deferred decisions and what was deliberately not optimised

| Deferred until measured | Trigger |
|---|---|
| Final hot data-node count (6 is an estimate) | Ingest-only and mixed load tests |
| Whether dedicated ingest nodes are needed | Ingest-only test showing pipeline CPU contending with search |
| Whether dedicated coordinating nodes are needed | Mixed test showing latency degradation at peak |
| Warm node count and instance choice | Warm-tier query measurements against the documented warm storage ceiling **[MGD]** |
| Whether A and B ultimately need separate clusters | The §9.6 signal — the mixed test failing repeatedly after role separation |
| The exact rollover threshold inside the 30 GiB target | A week of real daily volume variance |
| Whether remote-backed storage is adopted | It is bootstrap-only (§7.3), so it is a build-time decision not yet taken |

**Deliberately not optimised, with reasons:** **storage type** — general-purpose volumes are adequate at 10 shards per node on an ingest-dominated workload, so provisioning IOPS would optimise a resource that is not the constraint; **heap beyond the 50% rule** — the working set fits and the page cache is doing useful work, and no node approaches the managed quota; **latency on cold data** — a *business* decision recorded in the design, exactly the split §7.6 and §10.3 argue for; **shard count below two primaries for Workload A** — a bounded, recorded exception to the "fewer shards" instinct, bought for query parallelism; **zero-replica bulk windows** — declined on risk grounds despite the documented throughput benefit **[OS]**, because the resilience tolerance does not permit the documented data-loss window; and **shard size itself** — with one primary per daily index and size-based rollover, shard size is a derived quantity that stays stable by construction, which is the correct end state for this guide's central argument.

---

## 14. The Anti-Patterns

Eight failure modes, each as **symptom / cause / guardrail**. The first seven are the required set; the eighth is included because it is what survives the first seven being fixed.

### 14.1 Over-sharding by default

- **Symptom:** hundreds or thousands of small shards; heap pressure and `OutOfMemoryError` risk at modest volumes; latency climbing even as hardware is added; slow recoveries; elevated cluster-manager CPU.
- **Cause:** a shard count copied from a template, an old project or a default; migrating from a system where "more partitions" meant "more throughput"; pre-provisioning primaries for future volume today.
- **Guardrail:** derive the count from a stated target size and the documented arithmetic (§4.2, §4.5) *before* creating the index, because it is immutable. Verify against the balance rule `shards = k × data nodes` **[OS]** and both ceilings — ≤25 shards per GiB of heap **[MGD]** and the per-node shard limit for your version and deployment **[OS]/[MGD]**. Put a shard-size dashboard in place from day one (§11.5).

### 14.2 Shards sized to today's volume with no rollover strategy

- **Symptom:** shards growing past target over months; a reindex project "to fix shard sizes"; ingest slowing as individual shards become very large; recovery times lengthening monthly.
- **Cause:** a single index designed for launch-day volume with no append strategy; over-provisioning shard count at creation to "leave room" instead of rolling over — the documented trap where preparing for quadrupling creates shards below the recommended range today **[MGD]**.
- **Guardrail:** for append-only data, roll over — size as the primary trigger, age as a safety net, `ism_template` for automatic attachment, one write alias per family **[OS]**. Shard size then becomes a derived quantity that stays stable by construction, and retention becomes a policy action rather than a migration (§4.6).

### 14.3 The heap rule broken for a reason nobody can state

- **Symptom:** a heap set to a fraction nobody can justify; GC pauses moving latency percentiles; searches slower *after* an instance upgrade; breaker rejections with free memory on the box.
- **Cause:** optimising the wrong bottleneck. The heap is not the whole of memory, and the remainder is what caches Lucene segments; a heap sized until collections become long turns a throughput problem into a latency problem (§6.1).
- **Guardrail:** start from OpenSearch's documented 50%-of-RAM finding, with its scope stated — an indexing-only result on `r7iz.2xlarge` with a specific dataset **[OS]** — and adjust by measurement. State the reason in the design: the heap holds the working set while leaving the machine memory to cache segments and keep collections short. Never accept "it's the standard number", and never exceed the managed service's 50%-of-memory, 32 GiB Java-process quota without knowing it requires Auto-Tune or a support case **[MGD]**.

### 14.4 A node role collapsed onto the cluster-manager node

- **Symptom:** cluster-state operations slowing or stalling; shard allocation stopping; ISM — and therefore retention — silently pausing; election or quorum problems under load; a manager node at high CPU while data nodes idle.
- **Cause:** a topology chosen for node economy. The default node is cluster-manager-eligible *and* data *and* ingest *and* coordinating **[OS]**, so a small cluster quietly runs the coordination plane on the data plane; adding query traffic to a manager is worse, and the documentation explicitly advises against sending traffic directly to it **[OS]**.
- **Guardrail:** three dedicated cluster managers in three zones **[OS]**, achieved by setting the other node types `false` on them. The quorum arithmetic favours an odd number: an even number wastes a node without buying tolerance (§5.2). Monitor manager health separately from data-node health, and treat a stalled allocation or paused ISM job as a coordination symptom first (§11).

### 14.5 A lifecycle policy that moves data to a tier that cannot serve the required query

- **Symptom:** queries that were interactive become slow — or fail — at a predictable index age; a "cost optimisation" that produced complaints instead of savings; dashboards timing out on 60-day data.
- **Cause:** tiering designed against a storage-cost model instead of a query model. Searchable-snapshot-backed indexes are read-only and documented to produce "slower and longer snapshot searches" **[OS]**; warm and cold tiers are for data you are not actively writing and do not need the same performance from **[MGD]**. Moving data the business still queries interactively converts a saving into a service-level breach.
- **Guardrail:** map every query pattern to a maximum tolerable age and latency *before* drawing tier boundaries, and have the business sign the concession — which is why §7.6 and §10.3 call tiering a policy decision. Verify the direction of travel is possible: searchable snapshots need a `warm`-role node as of 3.0 **[OS]**, and cold storage is one-way without a warm round trip **[MGD]**. Test the transition with real queries (§13.7).

### 14.6 Sizing from generic vendor guidance without a load test

- **Symptom:** a cluster chronically overloaded or embarrassingly over-provisioned; sizing documents whose only evidence is a vendor table; per-node capacity numbers nobody has measured for this workload.
- **Cause:** treating published guidance as a specification. A guidance ratio is a starting point with a stated domain — the documented "2 vCPU and 8 GiB per 100 GiB of storage" is explicitly a floor for lighter workloads, with the service noting some users need many times those resources **[MGD]**.
- **Guardrail:** benchmark with **OpenSearch Benchmark** — the OpenSearch project's own tool **[OS]** — against representative data and queries, with a stated pass criterion, re-testing at each order of magnitude (§3.5, §11.6). Cite the source and date of every guidance figure used, label hardware catalogues as illustrative and dated, and record any sizing number whose method cannot be stated as rejected rather than as evidence (§3.6).

### 14.7 The cluster re-planned only after an incident

- **Symptom:** capacity work that only ever happens in response to an outage; a sizing document dated years ago that no longer describes the cluster; surprise at a shard-count or disk figure monitoring could have shown months earlier.
- **Cause:** capacity planning treated as a project deliverable rather than a loop. The information needed to re-plan early is already being collected; nobody set thresholds or a cadence.
- **Guardrail:** re-plan on a signal, not a failure — the seven signals in §11.6 (heap and GC trends, rejections and breaker events, watermark states, shard-size drift, approach to the ceiling, a change to retention or service level, and a version change altering documented behaviour). Set a quarterly steady-state test, a test before every upgrade, and a re-test after every scaling event.

### 14.8 Separate ingest and search sizings with no joint test

- **Symptom:** both capacities "sufficient" on paper, yet latency spikes at the same time daily — during the bulk load; rejections appearing only in the mixed window.
- **Cause:** ingest and search contending for the same CPU, disk I/O, page cache and heap while being modelled as independent line items. Sizing them separately is correct method and insufficient practice (§8.6).
- **Guardrail:** always run the mixed test (§3.5). Where contention is confirmed, isolate — dedicated ingest nodes or search-role nodes as documented **[OS]**, tier-based workload separation **[MGD]**, or two clusters (§9.6). Do not try to tune contention away with settings when the architecture is the problem.

### 14.9 Reference table — §14

| # | Anti-pattern | The single most useful guardrail |
|---|---|---|
| 1 | Over-sharding by default | Derive the count before creation; it is immutable |
| 2 | Sized to today's volume, no rollover | Roll over; make shard size a derived, stable quantity |
| 3 | Heap rule broken with no stated reason | 50% of RAM, with the reason and scope stated |
| 4 | Role collapsed onto the cluster manager | 3 dedicated managers in 3 zones |
| 5 | Tier policy serving an unservable query | Map query patterns to tiers before drawing boundaries |
| 6 | Guidance without a load test | OpenSearch Benchmark with a pass criterion |
| 7 | Re-planned only after an incident | Re-plan on the seven signals of §11.6 |
| 8 | Independent sizings, no joint test | Always run the mixed test |

---

## 15. The Claims Audit

Every high-risk claim in this guide, sorted into **verified**, **flagged**, and **rejected**. Dates are the retrieval dates of the primary sources in this work (September 2026, single research pass). "Quality" describes the strength of the evidence, not the importance of the claim. The highest-risk classes — the fork and its licence, the renamed concepts, every documented sizing figure, every node and tier name, and every managed-service figure — are all represented here explicitly.

### 15.1 The fork, the licence, and the version line

| Claim | Verdict | Source | Date | Quality |
|---|---|---|---|---|
| OpenSearch is derived from Elasticsearch **7.10.2** | **Verified** | opensearch.org/faq ("derived from Elasticsearch 7.10.2"; OpenSearch Dashboards "derived from Kibana 7.10.2") | Sept 2026 | Primary — the project's own FAQ. Statement is explicit and unambiguous. |
| OpenSearch is licensed under **Apache License 2.0 (ALv2)** | **Verified** | opensearch.org/faq | Sept 2026 | Primary. The FAQ states all software in the project is ALv2. |
| Elasticsearch/Kibana moved to the **Elastic License or SSPL**, described as not open source | **Verified** as the project's characterisation | opensearch.org/faq | Sept 2026 | Primary *for the characterisation*. The licence terms of Elastic products were **not** independently verified against Elastic's own repository in this pass — ⚠ note this is a partisan source describing a competitor's licensing. |
| OpenSearch **1.0 GA on 12 July 2021**; production-ready July 2021 | **Verified** | opensearch.org/releases (maintenance table, 1.x Initial GA); opensearch.org/faq | Sept 2026 | Primary, two independent pages agreeing. |
| The fork was **announced in April 2021** | **Flagged** ⚠ | commonly cited, not verified this pass | — | Not asserted anywhere in this guide. Do not present it as verified. |
| Current major line is **3.x**; 3.0 GA **6 May 2025**; 2.x in maintenance; 1.x end-of-life 6 May 2025 | **Verified** | opensearch.org/releases | Sept 2026 | Primary. Note the 3.x "latest minor" cell and the release history table are not perfectly synchronised on the page; the guide states the major-line facts and the release-history latest, and treats the next releases as scheduled rather than shipped. |

### 15.2 The renamed concepts (the highest fork-discipline risk)

| Claim | Verdict | Source | Date | Quality |
|---|---|---|---|---|
| "master node" → **cluster manager node** | **Verified** | docs.opensearch.org cluster-creation page: "The former 'master node' is now referred to as the cluster manager node"; role name `cluster_manager` | Sept 2026 | Primary, explicit quotation. |
| `master_timeout` → **`cluster_manager_timeout`**; old form deprecated since 2.0 | **Verified** | docs.opensearch.org Cluster Settings API query-parameter table | Sept 2026 | Primary, including the deprecation date and the stated reason. |
| ILM → **Index State Management (ISM)** | **Verified** | docs.opensearch.org/im-plugin/ism: the plugin, its `_plugins/_ism/policies` API, the policy/state/action/transition model, and the ISM-template mechanism | Sept 2026 | Primary. The Elasticsearch name is deliberately **not** used as if it were OpenSearch's. |
| Rally → **OpenSearch Benchmark** | **Verified** | docs.opensearch.org/benchmark: "a macrobenchmark utility provided by the OpenSearch Project", repository `opensearch-project/opensearch-benchmark`; the cluster page instructs planners to benchmark with it | Sept 2026 | Primary. Also verified: the 2.X terminology change and 1.15 as the last 1.X release. |
| `opendistro.index_state_management.policy_id` is deprecated in favour of the ISM template field | **Verified** | docs.opensearch.org ISM page | Sept 2026 | Primary. |
| Data Prepper / Fluentd / Fluent Bit / OpenTelemetry Collector as the supported ingestion path | **Verified** | opensearch.org/faq; docs.opensearch.org cluster page | Sept 2026 | Primary. |
| `compatibility.override_main_response_version` exists for version-checking clients | **Verified** as an existing setting | docs.opensearch.org configuration page | Sept 2026 | Primary for existence; no claim made here about its exact semantics. |

### 15.3 Every documented sizing figure, with provenance

This is the table that matters most, because it is where published advice most often misattributes Elasticsearch's numbers to OpenSearch.

| Figure | Verdict | Source | Date | Quality / provenance |
|---|---|---|---|---|
| **Heap: 50% of RAM** | **Verified as OpenSearch's own statement** | docs.opensearch.org indexing-tuning page: "Setting the Java min and max heap sizes to 50% of the RAM size shows better indexing performance on EC2 instances" | Sept 2026 | Primary — but **scoped**: indexing-only workload, `r7iz.2xlarge`, StackOverflow dataset, ≈60% improvement. Presented in §6.1 with that scope. |
| **Heap ceiling of ~32 GB (compressed oops)** | **Flagged** ⚠ | not verified from OpenSearch documentation this pass | — | Reported in §6.1 as Lucene/JVM lineage behaviour with the provenance labelled, never as an OpenSearch statement. Note the *managed service* documents a 32 GiB Java-process quota, which is a separate and verified fact. |
| **Shards per node = k × data nodes** | **Verified as OpenSearch's own** | docs.opensearch.org indexing-tuning, "Shard distribution", with the 24-shards/8-nodes example | Sept 2026 | Primary. A balance rule, not a size rule. |
| **Per-shard size 10–30 GiB (latency-sensitive) / 30–50 GiB (write-heavy)** | **Verified, but as MANAGED-SERVICE guidance** | docs.aws.amazon.com bp-sharding | Sept 2026 | Primary for Amazon OpenSearch Service. **Explicitly not attributed to the OpenSearch project's documentation.** |
| **Per-shard size 10–50 GB** | **Flagged as Elasticsearch-lineage** ⚠ | carried in this repo's sibling guide §6.1; the Elasticsearch page itself was not re-fetched this pass | Sept 2026 (sibling guide) | **[ES-inherited]**. Stated as lineage, never as OpenSearch's figure. |
| **Shards per heap: ≤25 per GiB** | **Verified, MANAGED-SERVICE guidance** | docs.aws.amazon.com bp-sharding, with the `m5.large.search` 4-GiB-heap → 100-shard example | Sept 2026 | Primary for the managed service. |
| **Shards per heap: ~20 per GB** | **Flagged as Elasticsearch-lineage** | Elasticsearch's long-standing guidance; carried in the sibling guide §6.1 | Sept 2026 | **[ES-inherited]**. Deliberately contrasted with the managed service's 25 in §4.2 so the reader sees they differ. |
| **Shard-count formula `(source + growth) × 1.1 / desired size`** | **Verified, MANAGED-SERVICE guidance** | docs.aws.amazon.com bp-sharding | Sept 2026 | Primary for the managed service; presented as such with the full worked example. |
| **Storage formula `source × (1+replicas) × 1.45`** | **Verified, MANAGED-SERVICE simplification** | docs.aws.amazon.com bp-storage | Sept 2026 | Primary. The 20% managed overhead inside it is a service reservation with no self-managed equivalent — stated in §3.2. |
| **Indexing overhead ≈10% (source + index ≈110%)** | **Verified, MANAGED-SERVICE** | docs.aws.amazon.com bp-storage | Sept 2026 | Primary, with the measurement method (`pri.store.size`, `_cat/allocation?v`). |
| **Linux reserved space 5%** | **Verified, MANAGED-SERVICE** | docs.aws.amazon.com bp-storage | Sept 2026 | Primary; also a general Linux filesystem default. |
| **Compute ratio ~2 vCPU / 8 GiB per 100 GiB** | **Verified, MANAGED-SERVICE, and scoped** | docs.aws.amazon.com bp-instances | Sept 2026 | Primary; the page itself qualifies it as a light-workload floor and warns some users need many times more. |
| **Bulk request size 5–15 MiB** | **Verified, OpenSearch's own** | docs.opensearch.org indexing-tuning | Sept 2026 | Primary. |
| **Index buffer 10%, up to 25%** | **Verified, OpenSearch's own** | docs.opensearch.org indexing-tuning | Sept 2026 | Primary. |
| **Translog flush 512 MB default; 25%-of-heap suggestion; 1024 MB example** | **Verified, OpenSearch's own** | docs.opensearch.org indexing-tuning | Sept 2026 | Primary, including the recovery-time trade-off warning. |
| **Refresh interval 1 s default; 30 s recommended** | **Verified, OpenSearch's own** | docs.opensearch.org indexing-tuning | Sept 2026 | Primary. |
| **Circuit-breaker defaults (95/70%, 40%, 60%, 100%, 75-per-5-min, regex factor 6)** | **Verified, OpenSearch's own** | docs.opensearch.org circuit-breaker | Sept 2026 | Primary, quoted per setting. |
| **Warm-node snapshot cache default 80%** | **Verified, OpenSearch's own** | docs.opensearch.org searchable snapshots | Sept 2026 | Primary, with the `node.search.cache.size` override. |
| **Disk watermark defaults ~85%/90%/95%** | **Flagged** ⚠ | not verified this pass | — | §11.4 states the mechanism and flags the values for confirmation. |
| **≈60% indexing throughput improvement from documented tuning** | **Verified, OpenSearch's own, and scoped** | docs.opensearch.org indexing-tuning | Sept 2026 | Primary; scoped to indexing-only on `r7iz.2xlarge` with the StackOverflow dataset and benchmark clients on a separate node. |

### 15.4 Every node and tier name

| Name | Verdict | Source | Date | Notes |
|---|---|---|---|---|
| `cluster_manager`, cluster-manager-eligible | **Verified** | docs.opensearch.org cluster page | Sept 2026 | OpenSearch's own naming; the guide never uses "master" as a role name. |
| `data`, `ingest`, `coordinating`, `dynamic` | **Verified** | docs.opensearch.org cluster page (node-type table) | Sept 2026 | Table includes each with a description and a production best-practice column. |
| `warm`, `search` | **Verified** | docs.opensearch.org cluster page; searchable-snapshots page | Sept 2026 | `warm` role required for searchable snapshots as of 3.0 (previously `search`). |
| Default node roles | **Verified** | docs.opensearch.org cluster page | Sept 2026 | "By default, each node is a cluster-manager-eligible, data, ingest, and coordinating node." |
| Tier control in self-managed OpenSearch | **Verified** | docs.opensearch.org cluster page | Sept 2026 | `index.routing.allocation.require.temp` with `hot`/`warm` values, driven by ISM. |
| A named self-managed "cold" or "frozen" tier in OpenSearch's own docs | **Rejected / absent** | not found in the pages consulted | Sept 2026 | The guide states the absence and attributes "cold storage" to the managed service. Do not use "OpenSearch frozen tier" as a product term. |
| Managed tiers: **UltraWarm**, **cold storage** | **Verified** | docs.aws.amazon.com ultrawarm / cold-storage | Sept 2026 | Managed-service names, clearly separated from product terms throughout. |
| Elasticsearch data-tier role names (`data_hot`, `data_warm`, `data_cold`, `data_content`, `data_frozen`) | **Not asserted** | — | — | Deliberately **not** used as OpenSearch role names anywhere in this guide, because they were not verified as OpenSearch's own naming in this pass. |

### 15.5 Every managed-service figure

| Claim | Verdict | Source | Date | Quality |
|---|---|---|---|---|
| Default index = **5 primaries + 1 replica** on the managed service, differing from open source | **Verified** | docs.aws.amazon.com bp-sharding | Sept 2026 | Primary; the page itself makes the contrast with open source. |
| 20% storage reservation per instance, up to 20 GiB | **Verified** | docs.aws.amazon.com bp-storage | Sept 2026 | Primary. |
| Java process memory 50% of total, max 32 GiB | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary. |
| r7g and OpenSearch-optimized instances exempt from the 32 GB cap, but heap not auto-raised | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary, including the Auto-Tune/support-case path. |
| Shard limits: 1,000/node for ES 7.x and OpenSearch ≤2.15; **1,000 per 16 GB heap, max 4,000** for 2.17+, default not changeable | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary. Version-gated; flagged as managed-service behaviour. |
| Per-AZ node limits 334/668/1002 hot and 250/500/750 warm | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary and dated; explicitly labelled illustrative because the catalogue changes. |
| Warm storage maximums 1.5 TiB / 20 TiB; OI2 addressable = 5× cache | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary, with the worked OI2 example. |
| gp3 maximums 80,000 IOPS / 2,000 MB/s, effective limits instance-dependent | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary. |
| gp2 / gp3 / magnetic as managed volume types; magnetic capped at 100 GiB and unsupported on Graviton | **Verified** | docs.aws.amazon.com limits | Sept 2026 | Primary. |
| io2 / st1 / sc1-style families for OpenSearch Service | **Flagged** ⚠ | not verified on the limits page this pass | Sept 2026 | §7.4 flags them as general EBS families requiring confirmation. |
| Warm requires dedicated master nodes; cold requires warm plus dedicated masters | **Verified** | docs.aws.amazon.com ultrawarm / cold-storage | Sept 2026 | Primary prerequisites. |
| T2/T3 data nodes cannot use warm or cold storage | **Verified** | docs.aws.amazon.com ultrawarm / cold-storage | Sept 2026 | Primary. |
| k-NN migrates to warm and cold from **2.17** | **Verified** | docs.aws.amazon.com ultrawarm / cold-storage | Sept 2026 | Primary, including the pre-2.x-created-index caveat. |
| Cold indexes retained **14 days** after deletion; restore only via warm | **Verified** | docs.aws.amazon.com cold-storage | Sept 2026 | Primary. |
| Multi-AZ with Standby requires warm node count as a multiple of AZ count | **Verified** | docs.aws.amazon.com ultrawarm | Sept 2026 | Primary. |
| `ultrawarm_manager` / `cold_manager` role mappings under fine-grained access control | **Verified** | docs.aws.amazon.com ultrawarm / cold-storage | Sept 2026 | Primary. |
| `CPUUtilization` and `JVMMemoryPressure` as the adequacy metrics | **Verified** | docs.aws.amazon.com bp-instances | Sept 2026 | Primary. |
| Actual AWS list prices for any instance type or storage class | **Deliberately excluded** | — | — | No price is quoted anywhere in this guide. §10.3 uses relative indices labelled illustrative. |

### 15.6 Rejected claims — things this guide refuses to assert

| Rejected claim | Why it is rejected |
|---|---|
| "OpenSearch's documentation says to size shards at 10–50 GB." | The 10–30/30–50 GiB figures are the **managed service's**; the 10–50 GB figure is **Elasticsearch lineage**. Neither was found in the OpenSearch project's own documentation in this pass. |
| "Rally is the OpenSearch benchmarking tool." | Fork-discipline error. The tool is **OpenSearch Benchmark**. |
| "Use ILM policies in OpenSearch." | Fork-discipline error. The feature is **ISM**. |
| "Add a master node." | The role is **cluster manager**; the term changed. |
| "OpenSearch defaults to 5 shards per index." | True of the **managed service**, false of open-source OpenSearch, which defaults to 1 primary + 1 replica. |
| "The frozen tier is an OpenSearch tier." | Not verified as a self-managed OpenSearch product tier this pass; "cold storage" is a managed-service tier. |
| "25 shards per GiB of heap is Elasticsearch's rule." | Provenance is reversed: 25/GiB is the **managed service's** figure; ~20/GB is the **Elasticsearch-lineage** figure. |
| "Heap must be 50% of RAM and never more than 32 GB, per OpenSearch." | Half true and therefore rejected as stated: the 50% is OpenSearch's own **scoped** finding; the 32 GB ceiling is a **managed-service quota** plus **JVM lineage**, not a general OpenSearch documentation statement. |
| Any specific AWS price. | Prices change continuously; a stale price is worse than no price. §10.4 states the labelling discipline instead. |
## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What could not be verified in this pass

**A tool limitation, recorded as such.** `web_search` returned **empty result sets for every query attempted**, including site-restricted queries against docs.opensearch.org. This is a **tool limitation, not evidence of absence** — no conclusion here rests on a search returning nothing. All research was direct extraction of primary URLs (docs.opensearch.org, opensearch.org, docs.aws.amazon.com) plus this repository's own guides. A future pass with a working search backend should re-attempt the ⚠ items.

| Item | Status | What would resolve it |
|---|---|---|
| The fork project's **announcement date** (commonly cited as April 2021) | ⚠ not verified | A dated primary announcement |
| A **per-shard size figure in OpenSearch's own project documentation** | Not found in the pages consulted (cluster creation, index management, indexing tuning, ISM, searchable snapshots, remote-backed storage, snapshot-restore, cluster settings API, benchmark) | A project page stating one, or confirmation that none exists. Size figures are therefore attributed to their managed-service or Elasticsearch-lineage sources |
| Elastic's own **"size your shards" page** carrying the 10–50 GB figure | ⚠ not re-fetched | A direct extraction, to convert the lineage attribution into a verified citation |
| The **compressed-oops / ~32 GB heap ceiling** rationale in OpenSearch's words | ⚠ not verified | An OpenSearch page on the heap ceiling, or acceptance that it is JVM/Lucene lineage |
| **Disk watermark defaults** (~85%/90%/95%) | ⚠ not verified | The configuration reference for the version in use |
| `data_hot` / `data_warm` / `data_cold` / `data_content` / `data_frozen` as **OpenSearch role names** | Not asserted — deliberately | OpenSearch's own node-role documentation enumerating them |
| A named self-managed **"cold" or "frozen" tier** | Not found in the pages consulted | A project page naming one for the self-managed product |
| **EBS io2 / st1 / sc1-style families** on the managed service | ⚠ not verified on the limits page | The current limits and pricing pages |
| **Account- and region-specific quotas** (as opposed to documented service maxima) | Not asserted | The provider's quota table for the account in question |
| **Exact instance catalogue and prices** | Deliberately excluded; catalogues described as illustrative | The provider's pricing page on the date of decision |
| **Cross-cluster replication specifics** (it exists; topology and capacity implications not researched) | Not covered | The availability-and-recovery documentation for the version in use. §12.5 cites only mechanisms verified in this pass |

The thesis does not depend on any flagged item: both irreversible decisions — shard count and heap — rest on verified facts.

### 16.2 Glossary

| Term | Definition as used here |
|---|---|
| **Circuit breaker** | A memory guard preventing `OutOfMemoryError`; the parent breaker governs the total heap available and children govern their own allocations **[OS]**. |
| **Cluster manager node** | The node that manages cluster operation and cluster state — OpenSearch's name for the role formerly called "master" (`cluster_manager`) **[OS]**. |
| **Cluster state** | Cluster metadata — indexes, shards, mappings, settings — held and published by cluster-manager-eligible nodes. Its size scales with shard and index count, which is why shard count has a ceiling. |
| **Cold storage** | A **managed-service** tier backed by object storage for infrequently accessed or compliance-retained data, queried by attaching indexes to warm nodes **[MGD]**. |
| **Coordinating node** | Delegates requests to shards and assembles the final result; dedicated coordinating-only nodes are recommended for search-heavy workloads **[OS]**. |
| **Data node** | Stores and searches data, performing indexing, searching and aggregating on local shards **[OS]**. |
| **Data tier** | A class of storage an index lives on (hot/warm/cold-style) with different cost and latency; in self-managed OpenSearch it is implemented with node attributes and ISM **[OS]**. |
| **Dedicated master node** | The **managed-service** term for a dedicated cluster-manager node, and a prerequisite for warm and cold storage **[MGD]**. |
| **Elasticsearch-lineage ([ES-inherited])** | A fact or figure originating with Elasticsearch or Elastic's documentation, widely quoted as if it were OpenSearch's; carried here only with that provenance stated. |
| **Headroom** | The margin between capability and peak demand; three kinds are needed — disk, heap, query/CPU (§3.4). |
| **Heap** | The JVM memory OpenSearch runs in; a rule of thumb at 50% of RAM, with the reason stated (§6.1) **[OS]**. |
| **Index** | The logical collection of documents, with `number_of_shards` fixed at creation **[OS]**. |
| **Index State Management (ISM)** | OpenSearch's lifecycle-management feature: policies of states, actions and transitions that roll over, move, force-merge and delete indexes **[OS]** — the renamed successor to Elasticsearch's ILM. |
| **Ingest node** | Runs ingest pipelines to transform data before indexing **[OS]**. |
| **Managed service ([MGD])** | A provider-operated OpenSearch offering, principally Amazon OpenSearch Service, whose defaults, limits, tiers and constraints do **not** automatically apply to self-managed OpenSearch. |
| **OpenSearch Benchmark** | The OpenSearch project's macrobenchmark utility (`opensearch-project/opensearch-benchmark`) **[OS]** — the renamed successor to Rally. |
| **Primary shard** | One of the fixed set of shards an index is divided into **[OS]**. |
| **Remote-backed storage** | A segment-replication-based durability feature (2.10) uploading translogs and segments to remote storage; cluster-level and bootstrap-only **[OS]**. |
| **Replica shard** | A full copy of a primary shard, multiplying storage and indexing work by (1 + replica count). |
| **Rollover** | The ISM action creating a new write index when a trigger (age, size, document count) fires, keeping shard size stable **[OS]**. |
| **Searchable snapshot** | An index served from a snapshot repository in object storage, read-only with local caching; requires a `warm`-role node as of 3.0 **[OS]**. |
| **Segment** | The immutable Lucene unit from which a shard is built; created by refresh and combined by merges. |
| **Self-managed OpenSearch** | A deployment you operate: nodes, JVM flags, disks, roles and tiers are yours to choose. |
| **Shard allocation awareness** | Spreads shard copies across zones or racks; **forced awareness** refuses allocation when the spread cannot be achieved; **replica count enforcement** guarantees a minimum number of copies per zone **[OS]**. |
| **UltraWarm** | The **managed-service** warm tier: object-storage-backed warm nodes with a caching layer, best suited to immutable data such as logs **[MGD]**. |
| **Warm node** | A node with the `warm` role serving searchable snapshots from remote storage with a segment cache **[OS]**. |
| **Watermark** | A disk-usage threshold that changes allocation and write behaviour, converting a performance problem into a capacity and availability problem (§11.4). |

### 16.3 Cross-references

**Cited, never re-derived:**

- [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) — the repo's Elasticsearch **data modelling and schema design** guide: mappings, text analysis, indexing strategy, data-modelling patterns, aggregations-friendly schema, reindexing and schema evolution, its own §6.1 "Shard Sizing". **No capacity or node-role content** — that is this guide's territory. The boundary rule: its modelling knowledge largely transfers to OpenSearch (schema and query semantics were inherited at the 7.10.2 fork); its operational and product facts do not.
- [capacity_sizing_guide.md](capacity_sizing_guide.md) — the generic **cloud capacity-sizing** discipline (Little's Law, utilisation targets, peak-vs-average, headroom, per-dimension sizing, the monitor→forecast→plan→review loop). The generic method this guide specialises; no OpenSearch content, not re-derived.
- [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) and the RAG guides under [ai_llm/rag/](ai_llm/rag/) — own **OpenSearch as a vector store**. Cross-referenced in §3.1, §6.3 and §7.3 only where vector workloads change the capacity arithmetic.
- [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) — *Operational Resilience: The Resilient Bank*, the operational-resilience anchor owning impact tolerances, important business services, BIA, BCP/DR tiers and RTO/RPO. Cited in §12 as the source of requirements this guide's mechanisms must satisfy.
- [../management/resilience_engineering_guide.md](../management/resilience_engineering_guide.md) and [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) — the engineering-discipline counterpart and a worked DR design in the same repo (§12).
- [clickhouse_guide.md](clickhouse_guide.md) and [dragonflydb_guide.md](dragonflydb_guide.md) — own their datastores; cited in §9.6 as alternative-store contrasts at the architecture decision boundary.
- The repo's observability and logging guides — own their pipelines; this guide plans the cluster those pipelines write into (§8.1, §12).

**Primary sources, retrieved September 2026:** docs.opensearch.org (cluster creation, index management, ISM, indexing tuning, circuit breakers, searchable snapshots, remote-backed storage, snapshot/restore, benchmark, logs, cluster settings API, configuration); opensearch.org/faq and /releases; docs.aws.amazon.com/opensearch-service (sizing-domains, bp-storage, bp-sharding, bp-instances, ultrawarm, cold-storage, limits); the `opensearch-project/opensearch-benchmark` repository as referenced from the project's benchmark docs.

### 16.4 Closing summary

**The thesis, restated because everything above is commentary on it:** capacity planning for OpenSearch is a shard-and-heap problem wearing a hardware costume. The hardware is the easy part — instance families, volume types and node counts are catalogue choices, revisable in an afternoon, and every provider documents them. The shard count and the heap size are not: the first is fixed when the index is created and changes only by migrating the data, and the second decides whether the machine's memory serves the search or merely the JVM.

**The eight conclusions to carry into a design review:**

1. **Characterise the workload before sizing anything** — search-dominated, ingest-dominated, aggregation-heavy or vector decides which sizing dominates, and misclassification is the root of most sizing errors.
2. **Derive volume with the documented arithmetic, then divide by a measured per-node capacity.** Published ratios are starting points with stated domains; a number whose method cannot be stated is not evidence.
3. **Roll over. Shard count is not the objective; shard size stability is.** For append-only data it also turns retention into a policy action instead of a migration.
4. **Over-sharding is the canonical error and a hardware cost, not just a performance cost** — a factor of 4.3 in shard count for identical data, demonstrated in §4.5.
5. **Frame the heap as a rule of thumb with a stated reason.** 50% of RAM is OpenSearch's own documented finding, scoped to an indexing workload; the reason is that the remainder caches segments and keeps collections short.
6. **Three dedicated cluster managers, in three zones, and an odd number** — the cheapest high-value spend in the design, since an even number wastes a node without buying tolerance.
7. **Retention and tiering are policy decisions and the two largest cost levers.** If the cost is wrong, look there first; if a tier cannot serve a query the business still runs, the saving was a service-level breach.
8. **Capacity planning is a loop** — re-plan on the seven measurable signals of §11.6, not after an incident, and test ingest and search together because they contend even though they are sized separately.

**The fork discipline, one last time, because it is the hazard most likely to recur:** OpenSearch is a fork of Elasticsearch taken at 7.10.2 and licensed under Apache 2.0; the lifecycle feature is **Index State Management**, the benchmark tool is **OpenSearch Benchmark**, the coordinating role is the **cluster manager**, and the per-shard size figures and heap ceilings most often quoted belong to a managed service or to Elastic, not to the OpenSearch project. State which is which every time, because the modelling knowledge transfers and the operational facts do not.

**The final word, and the reason this guide has the shape it has:** you can re-size a node, re-tier an index, re-price a strategy and re-run a benchmark — but you cannot re-shard an index in place, so plan the shard before the hardware, measure the heap before the instance, and respect the shard you cannot re-shard.

