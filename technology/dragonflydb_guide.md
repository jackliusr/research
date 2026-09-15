# DragonflyDB — The Dash Table

> **Author:** **Jack Liu Shurui, Solution Architect**
> **Context:** Technology Architecture — in-memory data stores, Redis-compatible caching infrastructure, regulated-enterprise operations
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Facts checked against primary sources:** 15 September 2026 (UTC). Every version, date, licence and funding claim is dated and sourced; the ones I could not verify are listed in §13.
> **Scope:** Dragonfly the product and the system: what it is and what it is not, the origin and the thesis, the shared-nothing multi-threaded architecture and its DashTable, the honest compatibility surface, the licensing position and its landmines, the performance evidence examined properly, the Redis/Valkey/Dragonfly landscape, deployment and operations, the use cases, and the assessment a regulated buyer must make.
> **Out of scope:** caching patterns and cache-design discipline (`technology/copilot_data_cache_guide.md`, `technology/ai_llm/agent_runtime_cache_design_guide.md`), rate-limiting patterns (`technology/distributed_rate_limiter_guide.md`), alternatives-landscape framing (`technology/kafka_alternatives_guide.md`), and data-modelling discipline (`technology/nosql_data_modelling_guide.md`). Those siblings own the patterns; this guide owns the engine.

**Related guides in this series:** [Copilot Data Cache](copilot_data_cache_guide.md) and [Agent Runtime Cache Design](ai_llm/agent_runtime_cache_design_guide.md) own caching patterns and eviction discipline; [Distributed Rate Limiter](distributed_rate_limiter_guide.md) owns the rate-limiting pattern that usually lands on a store like this one; [Kafka Alternatives](kafka_alternatives_guide.md) owns alternatives-landscape framing; [NoSQL Data Modelling](nosql_data_modelling_guide.md) owns key-design and access-pattern discipline; [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) owns RTO/RPO and disaster-recovery doctrine; [Billion-User System Architecture](architecture/billion_user_system_arch.md) owns the large-scale systems context in which a datastore choice sits.

---

## Table of Contents

1. [The Overview and the Identity](#1-the-overview-and-the-identity)
   - 1.1 [What Dragonfly Is](#11-what-dragonfly-is)
   - 1.2 [What Dragonfly Is Not](#12-what-dragonfly-is-not)
   - 1.3 [The Company, the Founders and the Funding](#13-the-company-the-founders-and-the-funding)
   - 1.4 [The One-Page Orientation](#14-the-one-page-orientation)
2. [The Origin and the Thesis](#2-the-origin-and-the-thesis)
   - 2.1 [The Limitation the Founders Identified](#21-the-limitation-the-founders-identified)
   - 2.2 [The Multi-Core Hardware Argument](#22-the-multi-core-hardware-argument)
   - 2.3 [The Timeline from Founding to Managed Service](#23-the-timeline-from-founding-to-managed-service)
3. [The Architecture](#3-the-architecture)
   - 3.1 [Shared-Nothing, Multi-Threaded Execution](#31-shared-nothing-multi-threaded-execution)
   - 3.2 [DashTable: the Data Structure Behind the Claims](#32-dashtable-the-data-structure-behind-the-claims)
   - 3.3 [Memory Efficiency and Compact Encodings](#33-memory-efficiency-and-compact-encodings)
   - 3.4 [Forkless Snapshotting and Persistence](#34-forkless-snapshotting-and-persistence)
   - 3.5 [Replication and Horizontal Scaling](#35-replication-and-horizontal-scaling)
   - 3.6 [SSD Tiering as It Actually Stands](#36-ssd-tiering-as-it-actually-stands)
4. [The Compatibility Surface](#4-the-compatibility-surface)
5. [The Licensing](#5-the-licensing)
6. [The Performance Claims Examined Properly](#6-the-performance-claims-examined-properly)
7. [The Landscape It Sits In](#7-the-landscape-it-sits-in)
8. [Deployment and Operations](#8-deployment-and-operations)
9. [The Use Cases](#9-the-use-cases)
10. [The Regulated-Enterprise Angle](#10-the-regulated-enterprise-angle)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)

---

## Scope: What This Guide Owns, and What Its Siblings Own

This guide owns Dragonfly as a system. It explains how the project replaces Redis's single-threaded event loop with a shared-nothing, multi-threaded execution model, what its own hash table actually is, how it snapshots without forking, what its command-compatibility matrix really says, what its licence lets you do, and what a bank should require before depending on it. Where a topic is already owned elsewhere in this repository, this guide names the sibling and moves on rather than restating it.

The boundary is deliberate and it is worth stating up front, because the collision is unavoidable. A Redis-compatible store is, by construction, used for caching, and this repository already has two cache guides: `technology/copilot_data_cache_guide.md` and `technology/ai_llm/agent_runtime_cache_design_guide.md`. **This guide does not re-derive cache-aside, write-through, TTL jitter, stampede protection or eviction policy design.** It assumes that discipline and asks only whether Dragonfly is the right engine to run it on. Likewise, `technology/distributed_rate_limiter_guide.md` owns the token-bucket and sliding-window patterns; §9 here only notes that Dragonfly ships a compatible rate-limiter command family and leaves the algorithm to the sibling.

One consequence of that boundary is worth stating plainly at the top. The difference between this guide and a vendor landing page is not tone, it is what gets included. Dragonfly's throughput story is genuine and architecturally explicable. Its licensing position is **not open source**, and the vendor's own FAQ admits it. Its compatibility is "fully compatible with the Redis ecosystem" as a slogan and a documented matrix of partial and unsupported commands as a fact. §4, §5 and §6 are where those three statements get separated from each other.

---

## 1. The Overview and the Identity

### 1.1 What Dragonfly Is

Dragonfly is an in-memory data store that speaks the Redis and Memcached wire protocols and replaces the incumbent's single-threaded execution model with a multi-threaded, shared-nothing one. The project describes itself in a single dense sentence: "Dragonfly is a modern in-memory datastore, fully compatible with Redis and Memcached APIs. Dragonfly implements novel algorithms and data structures on top of a multi-threaded, shared-nothing architecture. As a result, Dragonfly reaches 25X performance compared to Redis and supports millions of QPS on a single instance" (dragonflydb.io/docs, fetched 15 September 2026).

That sentence contains the whole product: a compatibility promise, a machine model, and a performance multiple. Only the first and second are load-bearing; §6 examines the third.

The problem class it targets has a specific shape, and the specificity matters because it is where the product either earns its place or does not:

| Property | What it means operationally | Where Dragonfly sits |
|---|---|---|
| **Single-instance vertical scale** | One process must use the whole box — 64+ cores, hundreds of GB of RAM — without a cluster | Thread-per-core, shared-nothing shards; shard data is not shared between threads |
| **Predictable tail latency under write load** | P99 on the write path, not the average, is the SLO | Vendor benchmarks show a flatter write-latency distribution than Valkey 9.0 (§6) |
| **Memory efficiency per key** | Bytes per entry determines the instance class you rent | DashTable reports ~127 bytes/entry against Valkey's ~149–150 (§3.3, §6) |
| **Capacity beyond RAM** | Dataset outgrows the memory budget without resharding | SSD data tiering, GA since June 2026 for strings (§3.6) |
| **Cluster-free multi-key semantics** | Multi-key and transactional operations must behave as they do on a single node | Single-node semantics preserved; Dragonfly's own cluster mode is young (§3.5, §4) |

The essential claim is the fourth column's first row: most systems that serve this workload scale *out* by sharding into a cluster of small nodes, and Dragonfly's thesis is that the modern cloud instance — 64 cores, 768 GB — makes scaling *up* both cheaper and simpler. The vendor states this as a belief: "Modern Cloud hardware is extremely powerful. Infrastructure should scale vertically first to utilize this amazing power. Less moving parts means more peace of mind" (dragonflydb.io/about, fetched 15 September 2026). That is marketing, but it is marketing that describes a real engineering decision, and the engineering decision has consequences both good (§6, §8) and bad (§8, §3.5).

### 1.2 What Dragonfly Is Not

Being precise here is more useful than being enthusiastic, because the failure mode is a team reading "drop-in replacement" as a specification rather than a marketing claim and discovering the gap after the cutover.

**It is not open source.** This is the single most important correction to make at the top of any evaluation, and it is not buried in a footnote. Dragonfly is released under the **Business Source License 1.1**, and the SPDX page for BUSL-1.1 states the position without ambiguity: "The Business Source License (this document, or the 'License') is not an Open Source license." The vendor's own FAQ confirms the licence and defends it: "We believe that a BSL license is more permissive than AGPL-like licenses" (dragonflydb.io/docs/about/faq, fetched 15 September 2026). §5 is devoted to the terms and to what they mean for a regulated buyer. Any procurement document that records Dragonfly as "open-source Redis alternative" is factually wrong.

**It is not a 100% compatible Redis.** The vendor claims "fully compatible with the Redis ecosystem" and "requires no code changes to implement". The vendor's *own command matrix* contradicts the absolute reading of that claim with dozens of unsupported and partially supported entries, including whole structural areas: `FUNCTION`/`FCALL`, `MODULE LOAD`, `MIGRATE`, `WAITAOF`, `SWAPDB`, `OBJECT ENCODING`, the `LATENCY` family, `CLUSTER ADDSLOTS`/`SETSLOT`/`FAILOVER`, and the TDigest, Time Series and Auto Suggest module families (dragonflydb.io/docs/command-reference/compatibility, verified against Dragonfly v1.40.0 and Redis 8.6.4, fetched 15 September 2026). §4 reproduces the honest table.

**It is not a like-for-like cluster replacement.** Its own cluster-mode management surface is thin by design: `CLUSTER ADDSLOTS`, `DELSLOTS`, `MEET`, `REPLICATE`, `FAILOVER`, `RESET`, `SAVECONFIG` and `SET-CONFIG-EPOCH` are all listed Unsupported, and Dragonfly substitutes its own `DFLYCLUSTER` family for control-plane work (dragonflydb.io/docs, command reference, fetched 15 September 2026). A team whose automation drives Redis Cluster slot management will not find that automation working unchanged.

**It is not a persistence-first or durability-first store.** It is an in-memory cache and data-structure server that persists via snapshot and replication. The tiering documentation says it outright in passing: "Durability is the same as standard Dragonfly — tiering is a capacity extension, not a persistence layer. Use snapshots or HA replicas for durability" (dragonflydb.io/blog, 1 June 2026). It is not a system of record.

**It is not a mature ecosystem play.** The project is a rewrite, not a fork, so it inherits Redis's *protocol* but not Redis's decade of module ecosystem, client-library edge-case fixes, managed-service ubiquity, or the operational folk knowledge that surrounds the incumbent. As of the fetch date the repository showed 31.5k stars and 1.3k forks (github.com/dragonflydb/dragonfly, fetched 15 September 2026) — healthy, and materially behind Redis's ecosystem gravity.

### 1.3 The Company, the Founders and the Funding

This section is written against the primary source — the vendor's own leadership page — because the secondary aggregators disagree with each other and at least one widely-repeated claim about the founders does not survive contact with the vendor's own site.

**The founders, verified.** The Dragonfly leadership team as published by the company is: **Oded Poncz — Co-Founder & CEO**, and **Roman Gershman — Co-Founder**, alongside Nicholas Gottlieb (VP of Marketing) and Ari Shotland (VP of Cloud Engineering) (dragonflydb.io/about, fetched 15 September 2026). TechCrunch's launch coverage adds the technical lineage and is consistent: "Oded Poncz and Roman Gershman experienced the pain of managing and scaling Redis … in their previous engineering roles. The pair worked together both at Google and at Ubimo, and Roman was a principal on Amazon's ElastiCache service" (TechCrunch, 21 March 2023).

**The Redis-lineage claim, examined and rejected as stated.** It is often repeated that Dragonfly was founded by Redis alumni, and specifically that **Ofer Bengal** — co-founder and CEO of Redis from 2011 to 2023 — founded or co-founded Dragonfly after leaving Redis. The first half of that is fully verifiable at primary source and the second half is not supported. Redis's own team page states that Bengal "co-founded Redis in 2011 and served as its CEO until 2023", and Redis's press release of 5 December 2022 announces that Rowan Trollope would become CEO **effective 1 February 2023, with Bengal moving to Chairman of the Board** (redis.io/company/team/ofer-bengal and redis.io/press/redis-ceo-succession, both fetched 15 September 2026). What could not be established from any primary source is any founder, officer or investor relationship between Bengal and DragonflyDB Ltd. The company's own leadership page does not list him. **The Dragonfly success story and the Redis succession story are two separate events in the same market; they must not be conflated.** The reported second name, **Yair Gottdenker**, could not be verified in connection with Dragonfly at all (§13).

So the accurate lineage sentence is narrower than the folklore and still interesting: Dragonfly was founded by two engineers who had operated Redis at scale in production — one of them as a principal on AWS ElastiCache, the managed Redis service — rather than by Redis's commercial founders. That is an *operator's* lineage, not a *commercial founder's* lineage, and it explains the shape of the product: a deep-fix of the execution model, plus a managed service, but no attempt to inherit the incumbent's brand or governance.

**Geography.** The company is reported as dual-based in San Francisco and Tel Aviv. DragonflyDB appears in Tel Aviv in the Israeli venture databases and in San Francisco in US-oriented profiles, and secondary profiles describe both; the vendor's own about page names neither city (dragonflydb.io/about; vcbacked.co/company/dragonflydb; startupintros.com/orgs/dragonflydb, all fetched 15 September 2026). Treat "San Francisco and Tel Aviv" as **verified-secondary**, and note that the more consequential geography for a buyer is not the headquarters but the **cloud regions** the managed service runs in (§8).

**Founding year.** Reported as 2022 by the aggregators, and consistent with the product timeline: Dragonfly reached **version 1.0 in the week of 21 March 2023**, at which point the company had "seven-person workforce" and planned to double it by year end (TechCrunch, 21 March 2023). The **2022 founding** is therefore **verified-secondary** — no primary incorporation record was retrieved, but the launch-state evidence is consistent with it.

**The funding position, and the conflict resolved as far as it can be.** Here is the primary-source position and then the noise around it.

| Claim | Figure | Source | Status |
|---|---|---|---|
| Funding to date at launch | **US$21M across seed and Series A** | TechCrunch, 21 March 2023: "raised $21 million to date from Redpoint and Quiet Capital across seed and Series A rounds" | **Verified (press)** |
| Funding round headline | **US$21m in new funding** | Businesswire headline, "DragonflyDB Announces $21m in New Funding and General Availability", 21 March 2023 (URL retrievable; body blocked to automated fetch) | **Verified (headline)** |
| Lead investors | **Redpoint Ventures** (lead Series A; Satish Dharmaraj, Managing Director, quoted in the launch coverage) and **Quiet Capital** | TechCrunch, 21 March 2023; dragonflydb.io/about investor logos | **Verified** |
| Aggregator "Series A, US$21.0M, March 2023" | US$21M | vcbacked.co/company/dragonflydb (page last updated 21 March 2023) | **Verified-secondary, consistent** |
| Aggregator "US$42.0M across 2 rounds" | US$42M | startupintros.com/orgs/dragonflydb, which separately lists a "$21M Seed (21 Mar 2023)" and a "$21M Series A (1 Mar 2023)" | **Rejected — double-count** |
| Reported conflicting figure **US$6M** | US$6M | No primary source located | **Unresolved — flagged (§13)** |
| Dealroom valuation "under US$100M" | <US$100M | Vendor/aggregator secondary; no primary round valuation was ever disclosed | **Unverified (§13)** |

The resolution is this. **US$21M is the total raised across both rounds and it is the figure the launch coverage and the press-release headline agree on.** The US$42M figure is an aggregator artefact — the same $21M event appears twice under two different round labels and is summed. The US$6M figure appears in secondary databases and **could not be reconciled to any primary source**; the most plausible reading is that it refers to the seed tranche alone, but I could not verify that, so it is reported as an unresolved conflict rather than silently dropped. Critically, **no post-2023 round, and no Series B, could be found for DragonflyDB.** The last funding event with a date is March 2023. Whether that means the company has been growing on that capital, has raised quietly, or has raised only venture debt is not something any primary source I fetched states — and a bank doing diligence on supplier viability must ask the company directly rather than infer from an absence (§10, §13).

### 1.4 The One-Page Orientation

```
        Dragonfly: one process, one machine, many cores

  clients ─► [ RESP over TCP | Memcached protocol | HTTP | TLS ]
                        │
                        ▼
          ┌─────────────────────────────────────────┐
          │  proactor / io_uring  (async I/O)        │
          └──────────────┬──────────────────────────┘
                         │ dispatch by key hash
     ┌───────────────────┼───────────────────┐
     ▼                   ▼                   ▼
 ┌──────────┐       ┌──────────┐        ┌──────────┐
 │ shard 0   │      │ shard 1   │  ...  │ shard N   │   thread-per-core,
 │ DashTable │      │ DashTable │       │ DashTable │   shared-nothing:
 │ own mem   │      │ own mem   │       │ own mem   │   no shared keyspace
 └────┬─────┘       └────┬─────┘       └────┬─────┘
      │                  │                  │
      └──────────┬───────┴──────────────────┘
                 ▼
   ┌──────────────────────────────┐     ┌─────────────────────────┐
   │ snapshot (RDB / DFS), no fork│     │ replication: REPLICAOF, │
   │ each shard serialises its own │     │ ROLE; lag = max unacked │
   └──────────────┬───────────────┘     └─────────────────────────┘
                  ▼
        optional second tier: local NVMe via io_uring
        (keys and metadata stay in RAM; values offload)
```

Read the diagram as three sentences. Data is partitioned across shards by key, and each shard owns its own memory and its own share of the keyspace — there is no shared table, so there is no cross-core lock on the hot path. A command executes on the shard that owns its key, on that shard's thread, which is why the engine scales with core count where a single-threaded engine does not. Persistence and replication are then built on top of that partition: because no memory is shared, each shard can serialise itself independently and no `fork()` is needed to obtain a consistent point in time.

The design's price is paid in two places, and honest reading requires both. First, operations that span many keys must still be correct when those keys live on different shards, and Dragonfly handles that with cross-shard coordination rather than a single-threaded guarantee. Second, anything that needs one global view — a single global index, for example — fits the model badly. §3.6 and §3.2 show the project discovering exactly that with its own vector search implementation, and rewriting the index because of it.

---

## 2. The Origin and the Thesis

### 2.1 The Limitation the Founders Identified

The founders' stated diagnosis is specific and worth quoting rather than paraphrasing, because it is the entire thesis of the product. Speaking to TechCrunch at launch, Poncz put it this way: "Redis was created 14 years ago, and while the amount of data the average application consumes has grown dramatically since then, the processing capabilities of Redis' … processing model have not kept pace." He added that Dragonfly "can support millions of operations per second and terabyte size data volumes from each instance" (TechCrunch, 21 March 2023).

The diagnosis has two halves, and it is worth separating them because only the first is a hard technical constraint:

1. **The execution-model half.** Redis executes commands on a single thread. Memory, network and CPU keep improving at different rates; a single core does not get faster at the rate instance cores multiply. So the ceiling on a Redis instance is the ceiling of one core, and the way the ecosystem works around that is to shard into a cluster. This is a structural observation and it is correct as far as it goes — the vendor's own comparison against Valkey 9.0 makes the point empirically, showing Valkey's throughput barely moving between 8 vCPU and 16 vCPU while Dragonfly's roughly doubles (§6).
2. **The operational-complexity half.** Clustering is not free. A cluster imposes limitations on multi-key and transactional operations, needs its own slot management, and multiplies the number of moving parts an operator must reason about. Poncz's framing in the same interview is about developer burden: "Developers are fed up with hand holding their infrastructure"; "the primary challenge is keeping pace with the performance demands of modern applications while keeping operations simple enough for the majority of developers."

**Attribution matters here.** Both halves of that diagnosis are the company's own telling, delivered in a launch interview, and the second half in particular is a value judgement rather than a measurement. What is verifiable is the architectural consequence: Redis's documented behaviour is that command execution runs on a single thread and newer releases add I/O threading that lifts network handling off that thread, which the vendor's own benchmark write-up describes accurately and uses as the basis for its comparison (dragonflydb.io/blog, 17 June 2026, quoting Valkey's `io-threads` behaviour and observing that "command execution still runs on a single core"). That sentence is the thesis restated by a competitor's own measurement; the framing around it is the vendor's opinion.

### 2.2 The Multi-Core Hardware Argument

The hardware argument is the part of the thesis that is not opinion, so it deserves its own paragraph.

Cloud instances and on-premise servers have grown cores and memory far faster than they have grown single-thread clock speed. A store whose throughput is bounded by one core therefore leaves most of a large machine idle by construction, and the industry's answer — run many small instances and shard across them — trades that idle CPU for network hops, cluster metadata, more failure domains, and operations work. Dragonfly's answer is to make the engine itself parallel: partition the keyspace across threads, give every thread its own memory and its own data structures, and issue I/O asynchronously so a thread never blocks waiting for the network or the disk (dragonflydb.io/docs and blog, fetched 15 September 2026).

The operational consequence is the argument's real content: **capacity scales with instance size**, not with instance count, so the natural unit of growth is a bigger box rather than another node. That is genuinely simpler when it works, and §8 shows where it stops working — because a single large box is also a single failure domain, and vertical scaling has a ceiling that a cluster does not.

### 2.3 The Timeline from Founding to Managed Service

| Milestone | Date | Source |
|---|---|---|
| Company founded (reported) | 2022 | Aggregators (startupintros, vcbacked); consistent with launch-state evidence — **verified-secondary** |
| First public release and open repository; project gained traction through 2022 | 2022 | dragonflydb.io/blog/2022-recap, referenced in secondary profiles |
| **Dragonfly v1.0 — general availability, and US$21M funding announced** | **21 March 2023** | TechCrunch, 21 March 2023 ("reached version 1.0 this week"); Businesswire headline, same date |
| **SSD data tiering first shipped in v1.35** | v1.35.0 / v1.35.1, November 2025 (v1.35.1 dated 25 November 2025 on the releases page) | github.com/dragonflydb/dragonfly/releases, checked 15 September 2026; dragonflydb.io/docs/managing-dragonfly/tiering |
| Vector search redesigned around a single global HNSW index in v1.37 | 26 February 2026 (post date) | dragonflydb.io/blog/vector-search-just-got-faster, fetched 15 September 2026 |
| **SSD data tiering declared GA** (Community Edition), string values | **1 June 2026** | dragonflydb.io/blog/ssd-data-tiering-is-generally-available, fetched 15 September 2026 |
| **Dragonfly v1.40.0** — Cuckoo Filter data type, tiered-storage and replication fixes, vector search native-width storage | **4 August 2026** | github.com/dragonflydb/dragonfly/releases, checked 15 September 2026 |
| **Current release: v1.40.2** (marked Latest); advised upgrade for a journaling correctness bug | **3 September 2026** | github.com/dragonflydb/dragonfly/releases, checked 15 September 2026 |

Two things in that timeline deserve comment. First, the gap between v1.0 (March 2023) and tiering (November 2025) is where a young engine sits longest: paying down correctness debt while the core features are stable. The v1.40.x patch notes are dominated by tiering, replication and journal-consistency fixes rather than headline features, and the maintainers flagged a **journaling correctness bug** with an explicit recommendation to upgrade — the kind of advisory that tells you the software is being exercised hard in production and still finding its footing. Second, the release notes for v1.40.0 describe it as having "the highest number of external contributors in the last two years", which is a useful, uncomfortable signal: external contribution is arriving, and the project is not yet carried by it.

**The managed service.** Dragonfly Cloud is the commercial offering: "an in-memory data store service fully managed by the Dragonfly team", deploying "the exact same technology without gated features", hosted on **AWS, GCP and Microsoft Azure**, with BYOC (bring-your-own-cloud) available alongside the vendor-hosted model (dragonflydb.io/docs/cloud, fetched 15 September 2026). The vendor's positioning line for it is unusual and worth recording verbatim, because it is both a promise and a liability: **no gated features**. The managed service differentiates on operations, not on software capability, which means the open-core risk profile for a self-hosting buyer is different from the usual one (§5, §10).

---

## 3. The Architecture

### 3.1 Shared-Nothing, Multi-Threaded Execution

The architecture is a single sentence and a long list of consequences. Dragonfly partitions the keyspace across threads; each thread owns a disjoint slice of the keys and its own memory, and there is no shared hash table and therefore no global lock on the command path. The project's own description: "Dragonfly implements novel algorithms and data structures on top of a multi-threaded, shared-nothing architecture" (dragonflydb.io/docs, fetched 15 September 2026). Each such thread is driven by an asynchronous I/O loop, and on modern Linux the disk side of that loop is `io_uring` rather than blocking syscalls (dragonflydb.io/blog, 1 June 2026).

| Design choice | Mechanism | Operational consequence |
|---|---|---|
| **Keyspace partitioned by thread** | A key hashes to a shard; that shard's thread executes the command | Throughput scales with core count instead of being capped at one core |
| **No shared mutable state on the hot path** | Each shard owns its own DashTable and its own allocator | Fewer cross-core synchronisation stalls; simpler reasoning about data races |
| **Asynchronous, non-blocking I/O** | Proactor-style event loop; `io_uring` for disk | A disk read does not stop the thread; other requests continue while a value is fetched |
| **Multi-key operations cross shards** | Coordinated rather than executed under one global lock | Single-node semantics are preserved for `MULTI`/`EXEC` and multi-key commands, at the cost of coordination work |
| **Emulated cluster mode by default** | `--cluster_mode=emulated` is the default when no flag is given | Redis Cluster-protocol clients connect unchanged to a single node |

The fifth row is the one that surprises people and it is documented plainly: "By default, if the `--cluster_mode` server flag is not specified, Dragonfly runs in this emulated cluster mode" (dragonflydb.io/docs/managing-dragonfly/cluster-mode, fetched 15 September 2026). A single Dragonfly node answers `CLUSTER SLOTS`, `CLUSTER SHARDS` and `CLUSTER KEYSLOT` as though it were a one-shard cluster, which is precisely what makes a Redis Cluster client library work against it without a reconfiguration step.

The execution model's price is structural and worth stating: **anything that needs a single global view fits badly.** A per-shard design has no place to put a structure that must be traversed as a whole — and §3.6 records the project hitting exactly that wall with its own vector index and rewriting it.

### 3.2 DashTable: the Data Structure Behind the Claims

The single most substantive thing Dragonfly does differently is not the threading, it is the hash table the threading runs on. The project calls it **DashTable**, and the vendor's own engineering write-up is unusually candid about where it came from: the algorithm is **extendible hashing**, "invented by Fagin, Nievergelt, Pippenger, and Strong in 1979 and published in the Journal of the ACM", adapted in a 2020 VLDB paper titled *DASH: Scalable Hashing on Persistent Memory*, which Dragonfly's implementation builds on (dragonflydb.io/blog/from-dict-to-dashtable, dated 30 July 2026 per the post, fetched 15 September 2026).

The mechanism, in the project's own terms:

| Element | What it is | Why it matters |
|---|---|---|
| **Directory** | Top-level array of pointers, indexed by the first K bits of a key's hash; K is the *global depth* | Growth in the directory moves pointers, not keys |
| **Segment** | A fixed-size mini-hash-table; its *local depth* says how many hash-prefix bits identify it | Capacity is bounded, so the cost of any single operation is bounded |
| **Segment internals** | 56 regular buckets plus 4 stash buckets; each bucket has 14 slots | Up to **840 entries** per segment — the unit of redistribution |
| **Split** | On a failed placement, one prefix range splits and one sibling segment is allocated | Only that segment's ≤840 entries move; the rest of the keyspace is untouched |
| **Directory growth** | If local depth equals global depth, the directory doubles first | Doubling the pointer array is cheap; no key data moves |

Two consequences follow directly and both are operational rather than aesthetic.

**First, there is no rehash spike.** Redis's `dict` is a separate-chaining hash table that grows by allocating `ht[1]` at twice the current capacity while `ht[0]` is still live and migrating keys incrementally. The vendor's write-up does the arithmetic: with `ht[0]` costing 8N for the bucket array and 24N for `dictEntry` nodes, and `ht[1]` adding 16N, peak memory reaches "48N, before a single key has moved" — and the cycle repeats each time the table fills. That produces the sawtooth memory profile operators know well, and it forces memory headroom to be provisioned for the *next* resize rather than for the data. DashTable's split is bounded and its growth curve is smooth, which the vendor measures: at 100 million keys, "Redis consumes roughly 7.5 GB of memory, while Dragonfly uses about 4.3 GB — a reduction of approximately 43%" (same post). **This is a vendor measurement of a vendor data structure, and it should be read as such (§6), but the mechanism it describes is verifiable in the source** (`src/core/dash.h`).

**Second, iteration does not require a fork.** This is the design decision that unlocks §3.4. Because the table never needs a stop-the-world or copy-on-write-memory view to be traversed safely — segments are independent and the table can be walked segment by segment — a snapshot can serialise the live table in place, with per-entry versioning to establish the point in time, instead of forking the process to get a private copy of memory.

### 3.3 Memory Efficiency and Compact Encodings

Per-key overhead is where an in-memory store's bill is actually decided, and there are three distinct mechanisms at work.

**The base table overhead.** §3.2 covers the headline: roughly 127 bytes per entry for Dragonfly against roughly 149–150 for Valkey 9.0 in the vendor's own head-to-head, "about 15% less RAM for the same dataset … roughly 9 GB you do not have to provision" on a 60 GB working set (dragonflydb.io/blog, 17 June 2026). The same write-up attributes the difference explicitly to "Dragonfly's dashtable layout, which has lower per-entry overhead than Valkey's hashtable plus dictEntry structure."

**Compact encodings for collection types.** Dragonfly reuses Redis's efficient binary encodings — the tiering write-up describes lists as "doubly linked lists of listpack nodes", where listpack is "a redis-specific efficient binary encoded format" — and adds compression on top. In July 2026 the project shipped dictionary compression for lists aimed specifically at job-queue payloads, claiming it "Cuts Celery & Sidekiq Queue Memory by 3-4x" (dragonflydb.io/blog, 20 July 2026). The same release train reduced JSON document memory and cut vector-search memory 40% for hash-map documents (v1.37 notes).

**Where efficiency is claimed and where it is not.** The 25x/80% figures on the landing pages are throughput and cost marketing. The defensible memory claim is the per-entry byte count, because it is a structural property of the table and it is measured on a fully populated keyspace. That distinction matters for capacity planning: you can plan from bytes-per-entry, you cannot plan from a percentage saving on an unspecified workload.

### 3.4 Forkless Snapshotting and Persistence

"Forkless" is the claim, and the documentation supports it in detail rather than in slogan form. The snapshot algorithm is described as follows: "The algorithm utilizes the shared-nothing architecture of Dragonfly and makes sure that each shard-thread serializes only its own data" (dragonflydb.io/docs/managing-dragonfly/snapshotting, fetched 15 September 2026).

The mechanism, as documented:

1. An `RdbSave` object creates one blocking channel to gather blobs from all shards, and instantiates a thread-local **SnapshotShard** in each shard thread.
2. Each SnapshotShard serialises its own keys into the Redis RDB binary format. Blobs are emitted **at bucket granularity** — never a blob that partially covers a bucket — "in order to guarantee snapshot isolation."
3. Serialised data is buffered in memory and flushed to the channel once large enough, then written to the output file or network socket through an `AlignedBuffer`, because Dragonfly uses direct I/O and direct I/O requires page-aligned buffers.

Point-in-time isolation is achieved with **entry versions and a captured epoch**, not with process forking. Each SnapshotShard records its shard's current epoch before scanning; the scan serialises entries whose version is at or below that cut, and a write hook serialises the pre-write value of any entry mutated during the scan. The documented variants are **conservative** snapshotting (the cut is captured when the snapshot *starts*, so concurrent writes are excluded — used for backups) and **relaxed** point-in-time snapshotting (the cut includes data up to when the snapshot *finishes*, used for replication so the changelog need not be retained during full sync).

What this buys, operationally: a snapshot does not double the resident set through copy-on-write page duplication, and it does not stall the engine behind a `fork()` of a multi-gigabyte address space. What it costs, also operationally: serialisation competes for CPU and memory on the same threads that serve traffic, only one of these formats is used for backups on disk versus replication over sockets, and the "Dragonfly Snapshot (TBD)" and "Relaxed point-in-time (TBD)" headings in the very documentation that describes the algorithm are marked as design notes rather than finished references. **Read that documentation for what it says the code does, not for what it promises to document.** The v1.40.2 release notes show the cost side directly: a fix adds `tiered_serialization_inflight_bytes_cap` because "snapshot serialization now throttles/waits when there are too many outstanding tiered reads", and the maintainers issued an explicit upgrade advisory for a **journaling correctness bug** (github.com/dragonflydb/dragonfly/releases, v1.40.2, 3 September 2026).

### 3.5 Replication and Horizontal Scaling

Replication is deliberately conventional on the surface. Dragonfly "supports a primary-replica high-availability model, similarly to Valkey replication", managed through the Redis-compatible `ROLE` and `REPLICAOF` commands; `REPLICAOF host port` points an instance at a new primary, and `REPLICAOF NO ONE` promotes a replica without discarding its dataset (dragonflydb.io/docs/managing-dragonfly/replication, fetched 15 September 2026). Replication lag is defined as "the maximal amount of unacknowledged database among all shards", exposed as `dragonfly_connected_replica_lag_records` in Prometheus metrics and as `lag` in `INFO REPLICATION`.

Three documented limits matter more than the happy path:

- **Redis→Dragonfly replication is capped at Redis OSS 6.2.** "We currently support the data structures and replication protocol of Redis OSS up to version 6.2." Migration from a newer source must therefore use snapshot-and-restore rather than live replication.
- **Cascading replication is experimental and off by default.** As of v1.40.0 it requires `--experimental_cascaded_partial_sync=true` on every node in the chain, and "if an intermediate replica performs a full synchronization after its upstream primary changes, it forces its downstream replicas to fully resynchronize." A chain is not a one-way upgrade.
- **Replicas may delete expired keys on their own.** By default a replica proactively deletes expired keys when they are read; `--replica_delete_expired=false` restores primary-driven deletion. This is a documented behavioural difference from a naive reading of replication as a pure stream.

**Horizontal scaling is where the architecture is most honest about itself.** Dragonfly has two cluster modes: **emulated single-shard** (the default) and **multi-shard**. The multi-shard documentation contains the sentence that should appear in every evaluation: "Dragonfly only provides a *data plane* (which is the Dragonfly server), but it does **NOT** provide a *control plane* to manage cluster deployments. Tasks like node health monitoring, automatic failover, slot migration for data rebalancing, and others are out of the scope of Dragonfly server functionality and are provided as part of the Dragonfly Cloud service" (dragonflydb.io/docs/managing-dragonfly/cluster-mode, fetched 15 September 2026).

The practical consequences of that sentence are sharp:

| Constraint | Documented behaviour | What a team must do about it |
|---|---|---|
| No control plane in the OSS server | Cluster config is pushed to each node; "Dragonfly nodes do not communicate with each other, except for replication and slot migration" | Build or buy orchestration; `tools/cluster_mgr.py` in the repository is a reference, not a product |
| Manual configuration | `DFLYCLUSTER MYID` then `DFLYCLUSTER CONFIG` with a JSON topology string over the admin port | Treat the topology as configuration-as-code, and push the *same* string to every node |
| Config drift is destructive | "Once a Dragonfly server is configured, it will only handle the slots it owns… any keys not owned by it will be deleted" | A wrong or stale config is a data-destruction event, not a routing error |
| Configuration is not persistent across change | "You'll need to resend the configuration to nodes after restart and to update all nodes with the new configuration upon any changes to the cluster" | Restart orchestration must re-apply config before the node serves traffic |
| Single database only | "Multi-shard cluster mode supports only database 0" | Code that uses `SELECT db` beyond 0 cannot use multi-shard mode |
| Cloud control plane is the commercial answer | **Dragonfly Swarm** — "sharding, slot migration, data rebalancing, and everything else are automatically managed" | The managed service is where the operational burden is actually discharged (§5, §10) |

The strategic reading is unavoidable and it is the crux of §5 and §10: **the OSS server is designed to be scaled vertically, and the thing that makes horizontal scaling painless is the vendor's managed service.** That is not a bug; it is a coherent business model. It is also a structural dependency a regulated buyer must price in.

### 3.6 SSD Tiering as It Actually Stands

Tiering is Dragonfly's answer to the one problem vertical scaling cannot solve: the dataset that outgrows RAM. It first shipped in **v1.35 (November 2025)**, was declared **generally available in the Community Edition on 1 June 2026**, and became **available on Dragonfly Cloud on 12 August 2026 for AWS and GCP** (github.com/dragonflydb/dragonfly/releases; dragonflydb.io/blog/ssd-data-tiering-is-generally-available, 1 June 2026; dragonflydb.io/blog/ssd-data-tiering-is-now-on-dragonfly-cloud, 12 August 2026 — both fetched 15 September 2026).

The design is documented precisely enough to reason about:

| Property | Documented behaviour |
|---|---|
| **What is offloaded** | Values only. "Keys, TTLs, and metadata always remain in Dragonfly's DASH table." `EXISTS` and `TTL` therefore never touch disk |
| **What qualifies** | Strings longer than 64 characters by default (`tiered_min_value_size` raises the bar); lists and small hashes are **experimental** |
| **Mechanism** | Linux `io_uring` (kernel **5.19+**) with `O_DIRECT` page-cache bypass and registered buffers for zero-copy I/O |
| **Data model** | Three states — hot (RAM), cold (SSD, pointer in RAM), cooled (on SSD with a temporary RAM copy so a read-after-write needs no disk read) |
| **Small-value handling** | Values are batched to fill a 4 KB page, because that is the smallest unit NVMe reads or writes |
| **Backpressure** | Actively writing clients are throttled once the offload threshold is crossed; new values are written directly to disk |
| **Metrics** | `INFO TIERED` exposes `tiered_entries`, `tiered_ram_hits`, `tiered_ram_misses`, `tiered_pending_read_cnt`, `tiered_pending_stash_cnt` and others |
| **Capacity claim** | "potentially achieving a 2x-5x improvement" over RAM-only, at sub-millisecond average latency |

**The restrictions are the important part, and the documentation states them.** "Data tiering is not supported for BitMap and HyperLogLog operations." Experimental list and hash tiering "are not supported by snapshotting or replication", with full support "available only for the String datatype". And the durability framing is explicit: tiering "is a capacity extension, not a persistence layer. Use snapshots or HA replicas for durability."

The vendor's own supporting evidence for tiering is instructive about what to trust. In the docs, an AWS comparison claims Dragonfly absorbed 200K RPS of writes into a 90 GB dataset where Memcached/Extstore dropped ~18% of the workload and ElastiCache throttled to 66.5K RPS, and that Dragonfly at 60K read RPS held 5 ms P99 where both alternatives reached 190 ms P99 (dragonflydb.io/docs/managing-dragonfly/tiering). In the GA announcement, the customer evidence is a single named case: **Meesho**, with a "300GB working set and 45GB RAM capacity running at ~30K ops/sec", reporting "20-25% lower latencies at equivalent load, alongside roughly 70% cost savings for the same cache footprint" after two months in production, quoted through an engineer identified as Yash Shah (dragonflydb.io/blog, 1 June 2026). That is one named customer with an attributed quote — weak evidence in absolute terms, but materially stronger than an unattributed percentage, and §12 records it that way.

**The architectural honesty in that write-up deserves credit and also sets a boundary.** The vendor describes its own limitation directly: "Traditional datastores are notoriously bottlenecked by disk I/O… Dragonfly bypasses these synchronous limitations by leveraging asynchronous `io_uring` APIs." The boundary is that this only exists on Linux ≥ 5.19 with a genuinely fast local NVMe device, and it is a per-node capacity extension rather than a shared storage tier — so a tiered node is still a single node with a local disk, and losing that disk loses the cold values.

---

## 4. The Compatibility Surface

### 4.1 The Claim and the Matrix

The vendor's compatibility claim and the vendor's compatibility *documentation* are not the same document, and reading them together is the whole exercise. The claim: "Dragonfly is fully compatible with the Redis ecosystem and requires no code changes to implement" (dragonflydb.io, landing pages and about page, fetched 15 September 2026), reinforced by Dragonfly Cloud's "provides full API compatibility with Redis, Valkey, and Memcached" (dragonflydb.io/docs/cloud). The documentation's own framing is more careful and is the sentence to quote in a design review: the matrix "tracks command-surface compatibility: whether Dragonfly accepts a command and its documented options or subcommands. **'Fully supported' does not imply byte-for-byte identical behavior.** See each command page for Dragonfly-specific precision, limits, and other behavioral differences" (dragonflydb.io/docs/command-reference/compatibility, fetched 15 September 2026).

That page carries an explicit verification stamp: **"Verification: Dragonfly v1.40.0; Redis 8.6.4; modules: BF, CF, CMS, FT, JSON, TDIGEST, TOPK, TS."** That is the single most useful line on the site for an evaluator, because it dates the matrix against a named Redis version.

### 4.2 What Works Unchanged

The coverage is genuinely broad, and the categories where nothing meaningful is missing are the ones most workloads actually use.

| Area | Verified status | Note |
|---|---|---|
| **Connection and protocol** | `PING`, `AUTH`, `HELLO`, `RESET`, `SELECT`, `ECHO`, `QUIT` all Fully supported; RESP2/RESP3 and the Memcached protocol on the same deployment | Client libraries connect without modification |
| **Strings** | Fully supported across the family (`SET`/`GET`/`MSET`/`MGET`/`INCR`/`APPEND`/`GETEX`/`GETDEL`/`SETRANGE`…) | `SET` is Partially supported, missing `IFEQ`/`IFNE`/`IFDEQ`/`IFDNE`, the newest Redis conditional-set options |
| **Hashes** | Fully supported including field-level TTL (`HEXPIRE`, `HGETEX`, `HPEXPIRETIME`) | Field TTLs are implemented, not stubbed |
| **Lists** | Fully supported (`LPUSH`/`RPUSH`/`LMOVE`/`BLMPOP`/`LPOS`…), all blocking variants present | Queue and stack patterns port directly |
| **Sets and Sorted Sets** | Fully supported across both families including `ZRANGE`/`ZRANGEBYLEX`/`ZRANGESTORE`/`ZMPOP`/`SINTERCARD` | Leaderboards and rank queries port directly |
| **Geospatial** | All query commands Fully supported | `GEOSEARCHSTORE` is the one gap |
| **Bitmap** | Fully supported | `BITOP` partially (see below) |
| **HyperLogLog** | `PFADD`/`PFCOUNT`/`PFMERGE` Fully supported | `PFDEBUG`/`PFSELFTEST` unsupported (diagnostics only) |
| **Pub/Sub** | Every listed command Fully supported, including sharded channels (`SPUBLISH`/`SSUBSCRIBE`) | Also a documented area of v1.40 stability fixes |
| **Transactions** | `MULTI`, `EXEC`, `DISCARD`, `WATCH`, `UNWATCH` — all Fully supported | Optimistic locking works; this is the differentiator against a Redis cluster |
| **Scripting (Lua)** | `EVAL`, `EVALSHA`, `EVAL_RO`, `EVALSHA_RO`, `SCRIPT LOAD`, `SCRIPT EXISTS` Fully supported | Lua scripts run |
| **Geo-distributed JSON** | All 27 listed `JSON.*` commands Fully supported | JSON ships in the server, not as a bolt-on module |
| **ACL** | The full `ACL` family Fully supported | Including `ACL DRYRUN`, `ACL LOG`, `ACL SAVE` |
| **Bloom / Cuckoo / CMS / Top-K** | Full command sets, with Cuckoo Filters added in v1.40.0 | Probabilistic structures are in-tree, mirroring RedisBloom |

### 4.3 What Is Not Supported, or Behaves Differently

This is the part that matters, and it is best organised by *what kind of application breaks* rather than alphabetically.

| Gap | Evidence (dragonflydb.io/docs/command-reference/compatibility, v1.40.0 vs Redis 8.6.4) | What it breaks |
|---|---|---|
| **Redis Functions** — the successor to Lua scripting | `FUNCTION *` Unsupported; `FCALL` Unsupported; `FUNCTION FLUSH` Unsupported | Any library or feature built on `FUNCTION LOAD` + `FCALL` (the modern Redis scripting model) does not run |
| **Server-side modules** | `MODULE LOAD` and `MODULE LOADEX` Unsupported; `MODULE UNLOAD` Unsupported (`MODULE LIST` works) | Third-party Redis modules cannot be loaded at all. This is a hard wall, not a gap |
| **Time Series** | `TS` family: "TBD — Unsupported" (RedisTimeSeries) | Metrics, IoT and time-series workloads have no path |
| **T-Digest** | "TBD — Unsupported" | Percentile-heavy analytics |
| **Auto Suggest** | "TBD — Unsupported" | Autocomplete built on `FT.SUGADD`/`FT.SUGGET` |
| **Graph** | `GRAPH` — "Not supported. Unsupported" | RedisGraph-style workloads |
| **Cluster control plane** | `CLUSTER ADDSLOTS`, `ADDSLOTSRANGE`, `DELSLOTS`, `DELSLOTSRANGE`, `MEET`, `FORGET`, `REPLICATE`, `FAILOVER`, `RESET`, `SAVECONFIG`, `SET-CONFIG-EPOCH`, `SETSLOT`, `SET-CONFIG-EPOCH` Unsupported; `ASKING` Unsupported; `CLUSTER REPLICAS`/`MYSHARDID`/`LINKS`/`COUNT-FAILURE-REPORTS` Unsupported | Redis Cluster automation, provisioning tooling and manual failover commands. `CLUSTER INFO`, `NODES`, `SLOTS`, `SHARDS`, `KEYSLOT`, `MYID` work |
| **Migration and server-to-server** | `MIGRATE` Unsupported; `WAITAOF` Unsupported; `SWAPDB` Unsupported; `FAILOVER` Unsupported | Tools that move keys between instances, and any code that blocks on `WAITAOF` durability acknowledgement, will not run |
| **Introspection and tuning** | `OBJECT ENCODING`, `OBJECT FREQ`, `OBJECT IDLETIME`, `OBJECT REFCOUNT` all Unsupported; entire `LATENCY` family Unsupported (`DOCTOR`, `GRAPH`, `HISTOGRAM`, `HISTORY`, `LATEST`, `RESET`); `MEMORY DOCTOR` Unsupported; `COMMAND DOCS`/`GETKEYS`/`GETKEYSANDFLAGS`/`LIST` Unsupported | Encoding is no longer observable, so textbook `ziplist`/`listpack`-aware sizing advice does not transfer. Redis's latency-monitor tooling has no equivalent |
| **Client tracking / caching** | `CLIENT TRACKING` Partially supported, "Missing: BCAST, PREFIX, REDIRECT"; `CLIENT TRACKINGINFO` Unsupported; `CLIENT GETREDIR` Unsupported | Client-side caching invalidation is only partially implemented — relevant if an application relies on server-assisted client caches |
| **Some option-level divergences** | `BITOP` Missing `ANDOR`, `DIFF`, `ONE`; `FLUSHALL`/`FLUSHDB` Missing `ASYNC`; `COPY` Missing `DB`; `SCRIPT FLUSH` Missing `ASYNC`/`SYNC`; `SET` Missing the conditional flags; `MEMORY USAGE` accepts but ignores `SAMPLES` | Option-level surprises in otherwise-supported commands |
| **Streams, mostly present with gaps** | `XADD` Missing `ACKED`, `DELREF`, `IDMP`, `IDMPAUTO`, `KEEPREF`; `XREADGROUP` Missing `CLAIM`; `XTRIM` Missing `ACKED`/`DELREF`/`KEEPREF`; `XSETID` Missing `ENTRIESADDED`/`MAXDELETEDID` | Streams are supported and mostly complete, but the newest Redis 8.x stream options are absent |
| **Search module, partially supported** | `FT.CREATE` Missing `NOFIELDS`, `NOHL`, `SCORE`, `SCORE_FIELD`, `INDEXALL` and more; `FT.SEARCH` Missing `HIGHLIGHT`, `SUMMARIZE`, `INFIELDS`, `INKEYS`, `GEOFILTER`, `TAGS`, `VERBATIM`, `WITHPAYLOADS`, `EXPLAINSCORE` and more; `FT.AGGREGATE` Missing `WITHCURSOR`, `RANDOM_SAMPLE`, `QUANTILE` and more | Full-text search works, but result presentation, highlighting and cursor-based pagination do not. `FT.HYBRID` (vector + filter) exists and is partially supported |

### 4.4 How to Read the Compatibility Claim

Four rules, in order of importance:

1. **Convert the application's actual command usage into a checklist and test it against the matrix line by line.** The matrix is public and version-stamped (v1.40.0 against Redis 8.6.4), so this is an afternoon's work, not a research project. Anything an application calls that appears as Unsupported is a blocker; anything Partially supported needs its specific missing options checked against how the code calls it.
2. **Separate "command missing" from "command different."** The documentation's own warning — "fully supported does not imply byte-for-byte identical behaviour" — means a command can appear as Fully supported and still return different precision, different memory accounting, or different empty-result semantics. Only per-command pages settle that, and the honest position is that **a compatibility matrix is a filter, not a proof.**
3. **Treat modules as the categorical gap.** `MODULE LOAD` being unsupported is not an edge case; it means the entire Redis module ecosystem — including several Redis first-party features that ship as modules upstream — is unavailable. Dragonfly compensates by implementing a substantial set of those features *in-tree* (JSON, Bloom, Cuckoo, CMS, Top-K, search, ACL), which is a genuine achievement and also a *maintenance liability*: every one of those implementations is now the project's own code to keep in sync with a moving Redis surface.
4. **Run the application's own test suite against Dragonfly before believing any of it**, and include the negative path: failover, restart, replica promotion, and what happens to a partially-completed `MULTI`. The compatibility matrix says nothing about behaviour under failure, and failure is where §8's operational gotchas live.

---

## 5. The Licensing

### 5.1 What the Licence Actually Is

This is the section a bank's legal and procurement functions should read first, and it begins with a correction rather than a description.

**Dragonfly is not open source.** The repository's licence file reads, verbatim: **"License: Business Source License 1.1 (BSL 1.1); Licensor: DragonflyDB, Ltd.; Licensed Work: Dragonfly including the software components, or any portion of them, and any modification; Change Date: Nov 1, 2030; Change License: Apache License, Version 2.0"** (github.com/dragonflydb/dragonfly/blob/main/LICENSE.md, fetched 15 September 2026). The SPDX entry for BUSL-1.1 states the consequence in its own notes: **"The Business Source License (this document, or the 'License') is not an Open Source license"** (spdx.org/licenses/BUSL-1.1.html, fetched 15 September 2026). The vendor does not dispute this; its FAQ is titled with the question "Is it open source?" and answers with the licence name and a defence of it.

The operative terms, quoted from the licence file:

| Term | Value | What it means for a self-hosting enterprise |
|---|---|---|
| **Licence** | Business Source License 1.1 | Source-available, not OSI-approved |
| **Licensor** | DragonflyDB, Ltd. | The commercial entity controls the grant |
| **Change Date** | **1 November 2030** | On that date (or four years after a given version's first public distribution, whichever is earlier) the version converts to the Change License automatically |
| **Change License** | **Apache License 2.0** | The eventual destination is genuinely open source — in 2030, or four years per version |
| **Additional Use Grant — permitted** | Use "(i) only as part of your own product or service, provided it is not an in-memory data store product or service; and (ii) provided that you do not use, provide, distribute, or make available the Licensed Work as a Service" | Internal production use, including hosting for other internal divisions, is permitted |
| **Additional Use Grant — prohibited** | Providing it as a Service: "a commercial offering, product, hosted, or managed service, that allows third parties … to access and/or use the Licensed Work … as a software-as-a-service, platform-as-a-service, infrastructure-as-a-service or other similar services that compete with Licensor products or services" | Building a competing managed cache on Dragonfly is the prohibited act |
| **Cure provision** | "If your use of the Licensed Work does not comply … you must purchase a commercial license from the Licensor … or you must refrain from using the Licensed Work" | Non-compliance is a licensing conversation, not a silent breach |

### 5.2 The Per-Version Detail That Nobody Reads and Everybody Should

Two clauses in that licence are easy to miss and materially change the risk picture.

**The change date is per version.** The licence says the Change Date "may vary for each version of the Licensed Work released by Licensor", and the conversion trigger is "the Change Date, or the fourth anniversary of the first publicly available distribution of a specific version of the Licensed Work under this License, **whichever comes first**." The practical reading: the version you deploy today becomes Apache-2.0-licensed four years after its own release, which is well before the 1 November 2030 headline date for anything shipped now. **The vendor is on the record committing to a genuine open-source destination for every version, on a rolling four-year clock.** That is a materially better position than a licence with no change date at all, and it is the strongest thing that can be said about this licence.

**Rights terminate automatically on breach.** "Any use of the Licensed Work in violation of this License will automatically terminate your rights under this License for the current and all other versions of the Licensed Work." Note the words "**all other versions**" — a breach on one version terminates rights across the project, retroactively including the change-date-converted ones. For a bank that must demonstrate continuous licence compliance to an auditor, this is the clause to have counsel read.

### 5.3 What Is Licensed How

| Component | Licence | Source |
|---|---|---|
| **Dragonfly server (Community Edition)** | BSL 1.1, Licensor DragonflyDB, Ltd. | github.com/dragonflydb/dragonfly/LICENSE.md, 15 Sep 2026 |
| **Dragonfly documentation** | Creative Commons Attribution-ShareAlike 4.0 International | dragonflydb.io/docs/about/license, 15 Sep 2026 |
| **Adapted Redis documentation content** | CC BY-SA 4.0, with attribution to "Copyright © 2009-2014, Salvatore Sanfilippo; Copyright © 2023, DragonflyDB Ltd." | dragonflydb.io/docs/about/license, 15 Sep 2026 |
| **Dragonfly Cloud (managed service)** | Commercial subscription; not the BSL. The Cloud docs point to Cloud-specific terms, a support-plan structure and a usage-based pricing model rather than to the BSL | dragonflydb.io/docs/cloud, /cloud/pricing, /cloud/support, 15 Sep 2026 |
| **Third-party Redis client libraries** | Unchanged — these are the client projects' own licences (MIT/BSD/Apache), not Dragonfly's | redis.io/legal/licenses (client table), 15 Sep 2026 |

One point of precision that matters and is frequently got wrong: the vendor states that Dragonfly Cloud runs "the exact same technology **without gated features**" (dragonflydb.io/docs/cloud, fetched 15 September 2026). **There is therefore no feature-gated Enterprise Edition of the server to escalate to.** The commercial model is operations, support, control plane (Dragonfly Swarm), BYOC and enterprise contracting — not withheld software capability. That is unusual, it is genuinely customer-friendly, and it also means **the vendor's monetisation depends entirely on the managed service remaining attractive**, which is the correct lens for §10's supplier-risk analysis.

### 5.4 The Commercial Model in Numbers

The pricing model is documented and is usage-based, with three components: **provisioned memory size**, a **compute performance tier** (Standard, Enhanced, Extreme — "the CPU-to-memory ratio"), and **cloud provider and region** (dragonflydb.io/docs/cloud/pricing, fetched 15 September 2026). Data transfer is charged "at public data transfer fee rates set by cloud vendors", and backup storage at **US$0.10 per GB per month**, prorated hourly. The documentation's own worked example — 12.5 GB on the Enhanced tier, two zones, AWS `us-east-1`, active 50 hours — uses **US$11.00 per GB per month** for Enhanced in that region and arrives at US$18.84. SSD tiering is priced at **1.5× the standard RAM rate** (US$11 → US$16.50 per GB/month in that example) and is described as including the SSD capacity, giving "8X the memory" — with the vendor's own illustration claiming 81% savings on a 400 GB dataset with a 50 GB hot set. Plan floors are quoted as **Flex at US$36/month and up** and **Business at US$2,000/month and up** (dragonflydb.io/blog, 12 August 2026). **All of these are vendor list prices retrieved on 15 September 2026 and none of them should be treated as a quote**; §11 uses clearly-labelled illustrative figures for that reason.

**Any hard commitment should be tested against a list price that has moved at least once already.** The vendor's 2025 summary advertised "a guaranteed minimum of 30% cost savings" as a Dragonfly Cloud Enterprise promise (dragonflydb.io/blog/2025-recap, 18 December 2025) — a contractual-sounding claim that a bank should require in writing, with a defined baseline, before relying on it.

### 5.5 Redis and Valkey, for Contrast

The following positions are verified at the vendors' own legal and project sources, and the contrast with Dragonfly is the whole point of the comparison.

| Project | Current licence (verified) | History | Governance |
|---|---|---|---|
| **Redis 7.2.x and earlier** | BSD-3-Clause | — | Redis Ltd. (single vendor) |
| **Redis Community Edition 7.4.x–7.8.x** | RSALv2 **or** SSPLv1 (dual, source-available) | **Announced 20 March 2024** | Redis Ltd. |
| **Redis Open Source 8.0+** | **Tri-license: RSALv2 or SSPLv1 or AGPLv3** | **Redis 8 GA 1 May 2025** added AGPLv3; product renamed from "Community Edition" to "Redis Open Source" | Redis Ltd. |
| **Redis modules (RedisJSON, Redis Stack, etc.)** | Dual RSALv2/SSPLv1 | Dual-licensed from **15 November 2022**; earlier modules used "Apache v2.0 modified with Commons Clause" (e.g. RediSearch 1.4–1.4.3, RedisGraph <1.0.14) — the 2018/2019 Commons Clause era | Redis Ltd. |
| **Valkey** | **BSD-3-Clause, "open source, forever"** | Fork of **Redis 7.2.4** announced by the Linux Foundation on **28 March 2024**, in response to the Redis licence change | **Linux Foundation**, open governance; participants include AWS, Google Cloud, Oracle, Ericsson, Snap Inc. (original) and Percona, Alibaba Cloud, Aiven, Tencent Cloud, NetApp Instaclustr, ByteDance and others today |

The Redis history is documented by Redis itself and is worth quoting because it is the cautionary tale in this guide's §10. Redis's own FAQ acknowledges that "some community members were frustrated by our March 2024 license change to the dual-license RSALv2 and SSPLv1, neither of which are OSI-approved licenses", explains that the change targeted "managed service providers who used Redis 7.2 and prior versions under the BSD3 license but provided limited contributions", and concedes that "some chose to move on" (redis.io/legal/licenses, fetched 15 September 2026). The AGPLv3 option added with Redis 8 is presented not as a licence change "but rather the inclusion of an option to provide more flexibility", and Redis states plainly that it has "no plans to change or add to the current Redis Open Source license configuration."

**The three-way summary.** Redis: source-available with an OSI-approved option, vendor-controlled, and with a *demonstrated willingness to re-licence* twice in six years. Valkey: unambiguously open source under foundation governance, with the specific design goal of eliminating "surprise license changes that break trust" (Chris Aniszczyk, CTO, Linux Foundation, 28 March 2024). Dragonfly: source-available, vendor-controlled, with a contractual change date to Apache-2.0 per version, no feature gating, and no re-licensing history — because it launched under the BSL and has never needed to change. **Evaluated purely on licence, Dragonfly sits between the two, closer to Redis than to Valkey, and its mitigation is the per-version change date rather than its governance.**

---

## 6. The Performance Claims Examined Properly

### 6.1 The Headline Numbers, Dated and Attributed

Dragonfly's performance claims are numerous, and they are all vendor-published. Recording them precisely is the first step to reading them honestly.

| Claim | Number | Source and date | Status |
|---|---|---|---|
| Landing-page headline | "up to a 25X boost in performance and 80% reduction in cost" | dragonflydb.io (site-wide footer and about page), fetched 15 Sep 2026 | **Vendor marketing, unattributed** |
| Docs headline | "reaches 25X performance compared to Redis and supports millions of QPS on a single instance" | dragonflydb.io/docs, fetched 15 Sep 2026 | **Vendor marketing** |
| Launch-era benchmark | "25x QPS and 12x faster snapshotting compared to Redis", as of July 2023 | Reported via launch coverage, March 2023; TechCrunch's own note: "TechCrunch can't independently confirm those claims" | **Vendor claim, third-party-noted as unverified** |
| Aggregator restatement | "3.9 million QPS per instance versus Redis's 150,000" | startupintros.com (aggregating vendor sources), fetched 15 Sep 2026 | **Secondary; superseded by the vendor's own newer figures** |
| GCP, Valkey 8.0.2 vs Dragonfly v1.26.1 | **2.4× throughput on 16 vCPU; 4.5× on 48 vCPU**; `ZADD` up to **29×** on 48 vCPU | dragonflydb.io/blog/dragonfly-vs-valkey-benchmark-on-google-cloud, **4 March 2025** | **Vendor-published, method disclosed** |
| AWS Graviton, Valkey 9.0 vs Dragonfly | m7g.2xlarge: 816K write / 848K read QPS vs Valkey's 548K / 811K. m7g.4xlarge: 1,563K / 1,581K vs 599K / 844K. **c7gn.metal single process: 6.66M write / 6.66M read QPS** | dragonflydb.io/blog/dragonfly-vs-valkey-90-on-aws-graviton-an-honest-head-to-head, **17 June 2026** | **Vendor-published, method fully disclosed** |
| AWS Graviton memory | **~127 bytes/entry** (Dragonfly) vs **~149–150** (Valkey 9.0) — "about 15% less RAM" | Same post, 17 June 2026 | **Vendor-published measurement** |
| GCP memory, sorted sets | 12.6 KiB per sorted-set entry vs Valkey's 23.1 KiB — "45% reduction" | dragonflydb.io/blog, 4 March 2025 | **Vendor-published measurement** |
| Tiering, read path vs ElastiCache | 335,011 vs 21,995 ops/sec reads — "15.2X"; write 655,863 vs 292,235 ops/sec — "2.2X" | dragonflydb.io/blog, 12 August 2026 | **Vendor-published, June 2026 test run** |
| Vector search vs Valkey | 3,136 QPS vs 2,104 QPS at EF_RUNTIME=64; 44% less memory than its own v1.35 and 47% less than Valkey | dragonflydb.io/blog/vector-search-just-got-faster, 26 February 2026 | **Vendor-published, method disclosed** |

### 6.2 The Methodology, Read Carefully

The benchmark write-ups are, to their credit, unusually explicit — and the explicitness is what makes a critical reading possible. The 2026 Graviton post states the setup fully: servers `m7g.2xlarge`, `m7g.4xlarge` and `c7gn.metal`; clients always larger than the server so the generator is not the bottleneck; Ubuntu 24.04.2; NIC IRQ affinity tuned because "a benchmark that does not address this is measuring the kernel, not the database"; `--save "" --appendonly no` on Valkey (no persistence); `--dbfilename=` on Dragonfly (likewise). And the scope limit is stated up front: "This benchmark covers string SET/GET workloads with 64-byte values, no persistence, no replication, and no cluster mode." **That single sentence disqualifies these numbers from use as a capacity-planning input for a persistent, replicated production deployment**, and the vendor says so itself.

Four methodological facts decide how much weight the numbers carry:

1. **The load generator is the vendor's own tool.** `dfly_bench` is described as "a load-testing program developed by our team for our internal benchmarking needs", "very similar to memtier_benchmark", and — this is the load-bearing sentence — "We've also seen it reach higher throughput rates for the same setup by being slightly more efficient. While the last point is less relevant for testing Valkey, it's very important for Dragonfly — we do not want our load-test program to become a bottleneck." **A generator that is measurably more efficient at driving one of the two systems is a conflict of interest in the measurement instrument itself.** It is disclosed, which is the honest thing to do; it is not neutral, which is the honest thing to remember.
2. **The Valkey configuration is tuned on the vendor's own judgement.** The 2025 GCP post states: "For Dragonfly, we chose higher numbers for THREADS and CONN_PER_THREAD. For Valkey, we had to choose lower numbers in order not to overload the server." The 2026 post is more careful — it justifies capping Valkey's `io-threads` at 8 by reporting that raising it produced no further throughput, and states "We are not under-threading Valkey to make it look bad; we are reflecting where the engine actually plateaus." That explanation is plausible *and* it is exactly the kind of claim that only an independent party can validate.
3. **The asymmetry is architectural, not just configurational.** The `io-threads` plateau is a genuine property of Valkey's design, and the vendor's 2025 framing of it is accurate: I/O threading "does not enable Valkey to operate on the data store from multiple threads", so "the main thread that cannot be offloaded any further" is the true bottleneck. This is the legitimate architectural argument, and it is best evidenced by the *scaling curve* rather than by any single ratio: Dragonfly's throughput approximately doubles from 8 vCPU to 16 vCPU (816K → 1,563K writes) while Valkey's moves from 548K to 599K. **Scaling behaviour is much harder to fake than a headline ratio**, because it is a shape rather than a number.
4. **The vendor volunteers the results that go against it.** The 2026 post states that Valkey 9.0 reads are "close — about a 5% gap" at 8 vCPU, credits the Valkey team's I/O work explicitly, reports that `m7g.8xlarge` showed throughput "very close to m7g.4xlarge" (an AWS instance-family limit that undercuts the vertical-scaling story), and reports that its own vector search v1.35 had *higher* raw precision than v1.37 (0.9975 vs 0.95) before explaining why that number was misleading. A write-up that publishes its own regressions and its own hardware ceiling is more trustworthy than one that does not.

### 6.3 How to Read a Vendor Benchmark Honestly

Six disciplines, in order of usefulness:

1. **Find the sentence that says what was not tested.** "No persistence, no replication, no cluster mode" is the most important line in the 2026 post. Whatever a vendor excludes from the test, you must test yourself.
2. **Ask who tuned what.** If the vendor chose the parameters for both systems, the comparison measures the vendor's judgement as much as the engines.
3. **Prefer curves to ratios.** "6.66M QPS" is a claim; "throughput roughly doubles when cores double, and the incumbent's does not" is a finding. Ratios can be engineered by picking a small baseline; scaling curves are constrained by physics.
4. **Check whether the alternative was given its best configuration.** A fair reading here credits the vendor for publishing the `io-threads` sensitivity test — and still notes that only Valkey's *documented, supported* scaling mechanisms were exercised, not an alternative topology (e.g. a properly sized Valkey cluster) that a buyer would actually run at scale.
5. **Separate the engine comparison from the product comparison.** The Graviton and GCP posts compare *engines*. The ElastiCache tiering post compares *managed services*, and there the gap is stated as 15.2× on reads — a number that reflects ElastiCache's documented promotion-through-RAM behaviour for tiered values as much as it reflects Dragonfly's io_uring path. Both may be true; they are different claims.
6. **Never use a vendor benchmark as a sizing input.** Use it to decide *whether a pilot is worth running*, then size from your own workload.

### 6.4 The Legitimate Architectural Argument, Separated from the Marketing Multiple

Strip away the 25× and what remains is a real, defensible argument:

- **A single-threaded engine has a hard ceiling that more cores do not raise.** Valkey's own FAQ confirms the design: "Enable I/O threading to offload client communication to threads … By offloading this work to separate threads, the main thread can focus on executing commands" (valkey.io/topics/faq, fetched 15 September 2026). Command execution remains single-threaded, so the ceiling stands. The vendor's measurements of a flattened scaling curve are consistent with this, and so is Valkey's own architecture documentation.
- **The workaround for that ceiling is a cluster, and a cluster has costs.** More failure domains, slot management, multi-key restrictions, DNS-based discovery, and — as Instacart's account puts it — degraded performance during cluster mutations at scale (§8).
- **A shared-nothing multi-threaded engine converts cores into throughput.** This is demonstrated by the scaling curve, which is the one number in the benchmark suite that the vendor cannot easily manufacture.
- **The multiple is workload- and topology-dependent, and often large.** 25× is a marketing figure; a 1.5×–2× write-throughput advantage on a comparable instance, plus a dramatically better scaling slope, is what the disclosed evidence actually supports for the tested workload. Those two statements are not in conflict — the first is a headline, the second is the finding.

**Verdict on the claims:** vendor-published, methodologically disclosed to an unusual degree, measured with a vendor-owned load generator, and valid strictly within the stated test envelope. Treat every ratio as an upper bound with the stated exclusions attached, and treat the scaling slope as the durable result.

---

## 7. The Landscape It Sits In

This is a positioning section, not a re-derivation of caching mechanics — the patterns live in `technology/copilot_data_cache_guide.md`, `technology/ai_llm/agent_runtime_cache_design_guide.md` and `technology/distributed_rate_limiter_guide.md`, and the alternatives-landscape framing lives in `technology/kafka_alternatives_guide.md`.

### 7.1 The Three Positions, Verified

| Dimension | **Redis** | **Valkey** | **Dragonfly** |
|---|---|---|---|
| **Strategy** | Commercialise and control the core; re-license as needed | **Fork and preserve** — continue Redis 7.2.4 under a foundation | **Rewrite** — new engine, same wire protocol |
| **Licence** | Tri-license (RSALv2 / SSPLv1 / AGPLv3) since Redis 8, GA 1 May 2025 | **BSD-3-Clause** | **BSL 1.1**, Apache-2.0 per version after ~4 years; Change Date 1 Nov 2030 |
| **Governance** | Redis Ltd. (single vendor) | **Linux Foundation** (open governance, multi-vendor) | DragonflyDB, Ltd. (single vendor) |
| **Execution model** | Single-threaded commands + I/O threads | Single-threaded commands + I/O threads (rewritten in 8.x) | **Shared-nothing, multi-threaded** |
| **Scale-out story** | Redis Cluster / Redis Cloud | Valkey Cluster; **all the major clouds** as participants (AWS ElastiCache, Google Memorystore, Oracle OCI, Tencent, Vultr, IONOS, Aiven, Percona, NetApp Instaclustr) | OSS multi-shard data plane only, **no control plane**; control plane is **Dragonfly Swarm** in the vendor's cloud |
| **Module ecosystem** | First-party modules now in-core; third-party module API | Module plugins supported | **`MODULE LOAD` unsupported**; a set of module features reimplemented in-tree |
| **Vector search** | Redis Query Engine + vector set (beta since Redis 8) | valkey-search (global HNSW) | FT.* partially supported; **single global HNSW index since v1.37** |
| **Beyond-RAM** | No SSD tiering in OSS; Redis on Flash in commercial product | **"no plans to create an on disk backend for Valkey"** (official FAQ) | **SSD data tiering GA** for strings since 1 June 2026 |
| **Current release (15 Sep 2026)** | Redis 8.x (matrix verified against 8.6.4) | **9.1.2**, released 1 September 2026 (8.1.10 and 7.2.14 also patched same day) | **v1.40.2**, 3 September 2026 |

### 7.2 Fork Versus Rewrite, Which Is the Real Strategic Difference

Valkey's model is migration-free continuity: the same codebase, the same data structures, the same RDB/AOF formats, the same `fork()`-based persistence, and the same single-threaded execution model — with the licence problem fixed by moving to a foundation and the roadmap accelerated by a consortium. Its pitch is that nothing has to change except the licence.

Dragonfly's model is the opposite bet: **abandon the incumbent's codebase entirely and replace the execution model, while keeping the protocol**. The consequences run in both directions and are worth tabulating because they are the decision a buyer is actually making.

| | **Fork (Valkey)** | **Rewrite (Dragonfly)** |
|---|---|---|
| Migration risk | Lowest — same behaviour, same formats | Higher — new code paths, an honest compatibility matrix with real gaps (§4) |
| Headroom for improvement | Bounded by the inherited execution model | Unbounded — the reason to rewrite at all |
| Licence risk | Lowest — foundation-governed, BSD | Lowest-to-moderate — per-version Apache conversion, but vendor-controlled until then |
| Operational maturity | Inherited: a decade of operator knowledge and tooling | Must be earned; the control plane is commercial (§3.5) |
| Ecosystem | Modules, clients, managed services everywhere | Redis clients work; modules do not |
| Performance ceiling | The single-thread ceiling remains | Scales with cores, demonstrated non-linearly for Valkey (§6) |
| Evidence for the claim | Fork provenance is fully auditable (Redis 7.2.4) | Requires reading the source: DashTable, io_uring, per-shard serialisation |

### 7.3 What Actually Changed for Buyers in 2024–2026

Three structural shifts, each of which changed what a buyer should weigh:

1. **The licence question became a governance question (March 2024).** Before the Redis dual-licence change, choosing Redis was choosing an open-source commodity. After it, choosing Redis meant either accepting a source-available licence, exercising an AGPLv3 option, or depending on a vendor's stated intention not to change again. Redis's own FAQ argues licence stability — "we plan to keep Redis Open Source under the AGPLv3 license" — but stability is a policy, not a contractual term. **A buyer should now distinguish between "this is open source" and "this vendor has committed, in writing, to keep this open source."** Valkey is the only one of the three where the second statement is structurally true rather than promissory.
2. **The performance question became a topology question.** With the single-threaded ceiling intact in both Redis and Valkey, achieving more throughput still means more nodes — which means more cluster complexity, more failure domains, and (per Instacart's account) more fragility during cluster mutations. Dragonfly's contribution is to offer vertical headroom as an alternative to another shard. **The relevant comparison is no longer "how fast is one instance" but "what does the whole topology cost to run and to operate."**
3. **The beyond-RAM question acquired a real answer.** Redis and Valkey both require the dataset to fit in memory (Valkey's FAQ says so explicitly: "no plans to create an on disk backend for Valkey"). Dragonfly shipped SSD tiering to GA in June 2026 for strings. For a workload where the working set is 300 GB and RAM is 45 GB (§3.6), that changes the set of viable products rather than the price of one of them.

### 7.4 What a Buyer Should Weigh Now

| Question | Why it now matters | Where the answer sits |
|---|---|---|
| Will our workload fit the single-thread ceiling on a big box, or do we need a cluster? | This is the first real question, and it is now answerable per-vendor | §6 (scaling curves), §3 |
| What licence risk are we accepting, and is the governance a mitigation? | Two Redis re-licences in six years made this a board-level question | §5 |
| Does our application use any of the gaps — modules, Functions, Time Series, cluster control commands? | These are hard blockers, not degradations | §4 |
| Can we operate a datastore-level HA topology ourselves, or must we buy the control plane? | Dragonfly's OSS cluster has no control plane | §3.5, §8 |
| Is the dataset bigger than RAM, and is it mostly cold? | Tiering is a genuine differentiator, strings-only at full support | §3.6, §9 |
| How young is the vendor, and what is our exit cost if it changes direction? | No Series B located since March 2023; the exit cost is bounded by the licence and the protocol compatibility | §1.3, §5, §10 |

The honest one-line positioning: **Valkey is the lowest-risk choice for a workload the incumbent design already serves; Dragonfly is the right choice when vertical headroom, memory efficiency or beyond-RAM capacity is the binding constraint, and it is the higher-risk choice on ecosystem, control plane and vendor maturity.**

---

## 8. Deployment and Operations

### 8.1 Deployment Options and Platform Support

| Path | Verified detail | Source |
|---|---|---|
| **Docker / Docker Compose** | Documented install paths | dragonflydb.io/docs/getting-started, 15 Sep 2026 |
| **Kubernetes Operator** | CRD-based (`dragonflies.dragonflydb.io`), installed into `dragonfly-operator-system`; features: automatic failover, horizontal and vertical scaling with custom rollout strategy, auth and server TLS, snapshots to PVC and S3-compatible storage, Prometheus/Grafana monitoring | dragonflydb.io/docs/getting-started/kubernetes-operator, 15 Sep 2026 |
| **Helm chart** | Documented alternative to the operator | dragonflydb.io/docs/getting-started, 15 Sep 2026 |
| **Binary / Linux packages** | Both documented; flag files supported | dragonflydb.io/docs/getting-started, 15 Sep 2026 |
| **Dragonfly Cloud** | Managed, on **AWS, GCP and Microsoft Azure**; BYOC available; purchasable via AWS and GCP Marketplaces | dragonflydb.io/docs/cloud, 15 Sep 2026; 2025 recap, 18 Dec 2025 |

Platform support is stated concretely and is better than many young projects: **x86_64 and arm64 are both "officially supported and certified"**, with x86_64 requiring a minimum *sandybridge* architecture. Continuous testing runs on **Graviton2 in AWS, x86_64 instances in AWS and GCP**, and regression tests run on **Azure via GitHub Actions**. OS: **Linux 4.14 or later, with 5.10+ recommended** — and, separately, **kernel 5.19+ with `io_uring` if SSD tiering is enabled** (dragonflydb.io/docs/getting-started; /docs/managing-dragonfly/tiering; both fetched 15 September 2026). Note the two-kernel problem: a Kubernetes fleet on a 5.10 kernel can run Dragonfly but cannot run tiering, which is a capacity-planning constraint that surfaces late if it is not checked first.

### 8.2 Persistence and Backup

Three mechanisms, with one default that will surprise a Redis-literate operator:

| Mechanism | Detail |
|---|---|
| **Backup on shutdown** | Automatic when `dbfilename` is non-empty |
| **Scheduled** | `snapshot_cron` (Dragonfly ≥ 1.7.1); the older `save_schedule` is **deprecated and will be removed** |
| **On demand** | `SAVE` and `BGSAVE` |
| **Auto-load** | On start, from the configured `dir`; disabled by an empty `dbfilename` |
| **Default filename** | `dump-{timestamp}` — `{timestamp}` is expanded to a lexicographically sortable local timestamp |
| **File format** | **`df_snapshot_format` defaults to `true`, i.e. Dragonfly's own `.dfs` format**, not Redis RDB. Files come in `-0000.dfs` / `-summary.dfs` pairs |
| **Object storage** | `--dir s3://bucket/prefix` writes to AWS S3 or S3-compatible storage (MinIO). **Documented as a preview feature** |

**(dragonflydb.io/docs/managing-dragonfly/backups, fetched 15 September 2026.)**

Two consequences deserve to be written into a migration plan rather than discovered during one. First, **the default snapshot format is Dragonfly's own, not Redis's** — a backup taken with default settings is not directly loadable into a Redis or Valkey instance, so anyone who needs an escape hatch back to the incumbent must either set `df_snapshot_format=false` or plan an application-level migration path. Second, **S3 backup is a preview feature**, which in this project's vocabulary means it should not be the sole durability mechanism for a regulated workload. The documentation's own tiering caveat reinforces the point: "tiering is a capacity extension, not a persistence layer. Use snapshots or HA replicas for durability."

### 8.3 High Availability and Failover

Self-hosted, the OSS project supplies the *capability* and not the *automation*: primary-replica replication via `REPLICAOF`, promotion via `REPLICAOF NO ONE`, lag observable in `INFO REPLICATION` and in Prometheus. There is no Sentinel equivalent in the OSS project. Practical HA therefore rests on one of two things — the **Kubernetes Operator**, whose documented feature list begins with "Automatic Failover", or the **managed service**.

Dragonfly Cloud's HA is documented in detail and is architecturally different from Redis Sentinel, which Dragonfly does not use at all. Every node runs a local agent that pings the Dragonfly process once per second over the admin socket, and every node monitors its peers by ICMP. That yields two detection paths: **~1 second for a process failure** (crash, OOM kill, SIGKILL — with a higher four-ping threshold if the process is alive but hung) and **~10 seconds for hardware failure**, where the local agent has died with the VM and peer consensus takes over. The control plane then selects the replica with the **highest replication offset**, promotes it, and updates DNS; client applications point at a DNS endpoint rather than implementing Sentinel-aware connection logic. A replacement node is provisioned in roughly two minutes (dragonflydb.io/blog/no-sentinels-required, dated 20 April 2026, fetched 15 September 2026).

The operational consequences are worth stating plainly. **Applications get simpler** — no Sentinel clients, no manual master discovery, and no separate three-process monitoring tier to patch. **Recovery is faster than a default Sentinel deployment**, whose `down-after-milliseconds` defaults to 30 seconds. **And the failover quality is a property of the paid service, not the source-available engine**, which is the dependency identified in §3.5 and priced in §10.

### 8.4 The Observability Surface

Dragonfly "allows HTTP access through its main TCP port (i.e., 6379) and exposes Prometheus-compatible metrics at `:6379/metrics`", including connection and pipeline memory metrics and full replication information — role, connected replicas, backlog and lag. The repository ships a worked **Grafana monitoring stack** example. Three documented caveats matter in practice: **on Kubernetes, metrics are also available on admin port 9999**, and hitting `:6379/metrics` there may return `Received HTTP/0.9 when not allowed`; **batch I/O counters are available via `INFO stats` but are not exported by the Prometheus endpoint**; and tiering has its own metric family under `INFO TIERED` (dragonflydb.io/docs/managing-dragonfly/monitoring and /tiering, fetched 15 September 2026).

The observability *gaps* are the more consequential finding for a team migrating from Redis, because they are gaps in tooling the team already relies on: the entire `LATENCY` command family is unsupported, `MEMORY DOCTOR` is unsupported, and `OBJECT ENCODING`/`FREQ`/`IDLETIME`/`REFCOUNT` are unsupported (§4). That means **latency-histogram dashboards, memory diagnostics, and any alert or capacity model keyed on Redis's encoding transitions all have to be rebuilt rather than ported.**

### 8.5 Scaling Limits and the Upgrade Path

The scaling story has a clear boundary: scale vertically first, and when that stops being enough, the OSS multi-shard cluster is a **data plane that requires your own control plane** (§3.5), restricted to **database 0**, configured by pushing the *same* JSON topology string to every node through the admin port, and **re-pushed after every restart**. The vendor's own headline for horizontal scale — "beyond 100TB of memory and 100 million RPS" (dragonflydb.io/blog/2025-recap, 18 December 2025) — is a statement about **Dragonfly Swarm in the managed service**, not about the OSS cluster. Reading it as an OSS capability would be a mistake.

Upgrades follow a fast, patched cadence: v1.40.0 (4 August 2026), v1.40.1 (6 August 2026) and v1.40.2 (3 September 2026), on top of a train that has reached 13 pages of releases. The Kubernetes operator documents "scaling horizontally and vertically (with custom rollout strategy)", which is the mechanism for controlled rollout. The read-the-release-notes discipline is not optional: v1.40.2 carries an advisory to upgrade because of "a correctness bug with journaling."

### 8.6 Known Operational Gotchas Adopters Report

| Gotcha | Evidence | Mitigation |
|---|---|---|
| **Stale cluster config destroys data** | "any keys not owned by it will be deleted" (cluster-mode docs) | Topology as code, re-applied before a node serves traffic; never hand-edit |
| **Cluster config is not persisted across restart** | "You'll need to resend the configuration to nodes after restart" | Restart automation must apply config first |
| **Only database 0 in multi-shard mode** | Documented limitation | Audit for `SELECT n` usage before choosing multi-shard |
| **Redis→Dragonfly live replication stops at Redis OSS 6.2** | Documented | Plan snapshot-and-restore or dual-write for newer sources |
| **A newly started cluster node rejects user requests** | "it will reply with errors to any user requests until the cluster is properly configured" | Readiness probes must not report ready on process-start alone |
| **Experimental features are excluded from snapshot and replication** | List and hash tiering: "not supported by snapshotting or replication" | Do not enable experimental tiering on a workload needing either |
| **Backups default to the proprietary `.dfs` format** | `df_snapshot_format=true` by default | Set it false if an exit path to Redis matters |
| **Performance may need vendor engagement, not just tuning** | Instacart: reaching expected performance "took a combination of client-side tuning and engine-level improvements from the Dragonfly team, including changes to the compactor" | Treat the pilot as a joint engineering exercise with a support contract in place |
| **Cluster mutations at scale are a pain point in the incumbent too** | Instacart: at large cluster sizes, "any operation that modified the cluster … caused prolonged periods of degraded performance" | This is a reason to evaluate, not a Dragonfly-specific defect — and it is precisely what vertical scaling avoids |

---

## 9. The Use Cases

### 9.1 The Class, and Which Member Suits Dragonfly Specifically

The workloads an in-memory data store of this kind serves are well covered elsewhere in this repository, and the patterns are not repeated here: **caching and eviction discipline** belong to `technology/copilot_data_cache_guide.md` and `technology/ai_llm/agent_runtime_cache_design_guide.md`; **rate limiting and token-bucket or sliding-window counters** belong to `technology/distributed_rate_limiter_guide.md`; **queues and event-ish patterns** and their trade-offs against a log-based backbone belong to `technology/kafka_alternatives_guide.md`; and **key design, cardinality and access-pattern discipline** belong to `technology/nosql_data_modelling_guide.md`. What follows is only the question this guide owns: which of those workloads suit *Dragonfly specifically* rather than the class in general.

| Workload | Served by the class | Suits Dragonfly specifically when… |
|---|---|---|
| **Cache / read-through** | Yes | The working set is large enough that per-key overhead and instance count drive the bill — the ~127 vs ~150 bytes/entry difference multiplies (§3.3) |
| **Session state** | Yes | Session data is regenerable, so loose RPO is acceptable and the vertical-scaling simplicity pays; per-key overhead matters at hundreds of millions of sessions |
| **Rate limiting / counters** | Yes | Throughput on a single logical counter key is the bottleneck; single-node semantics for `MULTI`/`WATCH`-based limiter implementations avoid cluster-mode multi-key restrictions. The algorithm itself is the sibling guide's |
| **Leaderboards / sorted sets** | Yes | The scores are large and hot — the vendor's own sorted-set memory measurement (12.6 KiB vs 23.1 KiB per entry) is a real cost lever at scale |
| **Job queues / sidekiq-style** | Yes | The queue is memory-bound and large; list dictionary compression targets exactly this payload, and list tiering (experimental) fits the read-one-end/append-other pattern |
| **ML / feature store** | Yes | This is the strongest fit: dozens-of-GB datasets with a small hot slice, low-latency key lookups, and Instacart's production account of an ad-serving feature store is the best-documented adopter case (§8.6) |
| **Pub/Sub** | Yes | The command surface is complete and the sharded variants work, but nothing about this workload specifically rewards multi-threading over a Redis cluster |
| **Vector / semantic search** | Partly | **Verified as shipped, at partial maturity** — see §9.2 |
| **Time series** | No | `TS` family unsupported (§4) — Redis or a purpose-built store |
| **Graph / document query engine** | No | `GRAPH` unsupported; the JSON commands work but the query engine surface is the Redis Query Engine's, and it is not all present |

### 9.2 The AI and Vector Case, Verified as Shipped

This is the claim most likely to be read from an older announcement, so it is worth being precise about what exists today.

**Vector similarity search is genuinely shipped, and it has been rearchitected.** The relevant history is that v1.35 and earlier maintained **a separate HNSW index per shard** and merged results across shards, which the vendor now describes as a poor fit for its own architecture: "Inverse indexes like HNSW are fundamentally a poor fit for Dragonfly's original shared-nothing, thread-per-shard architecture… precision was unstable: it would shift as you scaled thread counts, and performance would actually degrade as more threads were added." **v1.37 (February 2026) moved to a single global HNSW index**, with the index holding references to the original memory locations rather than copying vector data (dragonflydb.io/blog/vector-search-just-got-faster, 26 February 2026). **v1.40.0 (August 2026)** added "native-width storage across all vector dtypes, EPSILON-tuned HNSW range queries, and EF_RUNTIME control" (github.com/dragonflydb/dragonfly/releases, v1.40.0).

The measured result, vendor-published on the `gist-960-euclidean` dataset (1M vectors, 960 dimensions, HNSW M=32, EF_CONSTRUCTION=128) on `m7g.4xlarge`: at EF_RUNTIME=64, **v1.37 reached 3,136 QPS at 0.75 precision versus Valkey's 2,104 QPS at 0.75 precision**, with RSS of **4.57 GB versus 8.65 GB**. The post is also candid that its predecessor had *higher raw precision* (0.9975 at EF_RUNTIME=512) but was unusable at scale for the reasons quoted above.

**The limits are documented, and they define the use case.** In the compatibility matrix, `FT.CREATE`, `FT.SEARCH`, `FT.AGGREGATE` and `FT.HYBRID` are all **partially supported**, with significant missing options — `FT.SEARCH` lacks `HIGHLIGHT`, `SUMMARIZE`, `INFIELDS`, `INKEYS`, `GEOFILTER`, `TAGS`, `VERBATIM`, `WITHPAYLOADS` and `EXPLAINSCORE`; `FT.AGGREGATE` lacks `WITHCURSOR`, `RANDOM_SAMPLE` and `QUANTILE`; and **Auto Suggest is entirely unsupported**. There is also **no separate vector-database comparison here** — that is owned by `technology/ai_llm/rag/vector_databases_guide.md`, and Dragonfly's appropriate placement in that landscape is as a low-latency similarity layer for in-memory embedding sets, **not** as a replacement for a dedicated vector database at scale. Vector search is a real, shipped, recently-improved capability with a partial command surface; it is not a mature standalone vector platform, and the vendor's own decision to rewrite the index rather than tune it is the clearest evidence that this area is still moving.

---

## 10. The Regulated-Enterprise Angle

This is the assessment a bank must make before depending on Dragonfly in production, and it is written as the questions a risk function should ask rather than as a recommendation.

### 10.1 Licence and Open-Core Risk

**The licence is not open source, and that is the first finding (§5).** The second finding is more useful: this is a *predictable* licence. The BSL carries a real change date to a real open-source licence — Apache-2.0, per version, at four years or 1 November 2030, whichever comes first — and the project has no re-licensing history because it launched under the BSL. That is structurally different from the Redis situation, where a permissively licensed project was re-licensed under commercial pressure.

**The Redis precedent must still be named as the cautionary tale**, because it is the reason this section exists. In six years Redis moved from BSD-3-Clause (its modules were moved to "Apache 2.0 with Commons Clause", then to RSALv2/SSPLv1 from 15 November 2022), then re-licensed the core itself from Redis 7.4 (announced 20 March 2024), then added AGPLv3 with Redis 8 (GA 1 May 2025) — three distinct licensing positions, each defensible on the vendor's own terms and each disruptive to adopters who had built on the previous one. **The lesson for a bank is not "avoid BSL" but "a vendor's licence promise is only as durable as the vendor's commercial incentive, so get it in contract."**

**What the open-core risk profile actually is here is unusual and worth stating:** the vendor commits that Cloud runs "the exact same technology without gated features" (§5.3). There is no Enterprise Edition with held-back engine capability. The consequence is that self-hosting does not put you on a feature-degraded path — and, correspondingly, **the vendor's revenue depends on the managed service, the control plane and support**, which is where its incentives will concentrate.

### 10.2 Self-Hosting Versus Managed Cloud Against Data Residency

| Model | Data location | Licence position | Residency implication |
|---|---|---|---|
| **Self-hosted OSS** | Your estate, your control | BSL 1.1 permits internal production use, including for other internal divisions; prohibits offering it as a service to third parties | Best position: data never leaves the bank's network |
| **Dragonfly Cloud, vendor-hosted** | AWS, GCP or Azure regions | Commercial subscription | Residency depends on region availability on those providers; data leaves the bank's account |
| **Dragonfly Cloud, BYOC** | Inside **your** AWS/GCP/Azure account | Commercial subscription | Strong position: vendor operates the service but the data plane sits in the bank's own cloud account and VPC |

**Independence requirement worth checking explicitly:** the Additional Use Grant prohibits providing Dragonfly "as a Service … that allows third parties (other than your own employees and contractors acting on your behalf) to access and/or use the Licensed Work". **Hosting it for the bank's own employees and for other internal divisions is permitted. Embedding it in a product or platform offered to external parties is not** — which is the precise point at which a bank that white-labels a platform to partner institutions would breach the licence. That analysis should be documented per deployment, not assumed.

### 10.3 Operational-Maturity and Support-Model Risk

The honest position: **this is a young company with a fast-moving engine and a commercial control plane.** The verifiable indicators are that the last funding event with a date is **March 2023** (US$21M, no Series B located); the project carries 31.5k stars and 1.3k forks; the server supports **x86_64 and arm64** and is continuously tested on three clouds; releases are frequent and include explicit correctness advisories; and the vendor stated in August 2026 that v1.40.0 had "the highest number of external contributors in the last two years" — meaning external contribution is arriving but the project is not yet carried by it.

Three specific consequences for a bank. **First, the operational control plane is commercial** (§3.5), so self-hosting HA and scaling means implementing or operating what the vendor sells. **Second, the engine is still finding correctness bugs in production-shaped code** — the tiering, replication and journal-consistency fixes concentrated in v1.40.x, and the journaling upgrade advisory, are the observable trace of a system under heavy real use. **Third, a bank should not treat the vendor's viability as an abstraction**: no disclosed round since 2023 means diligence must ask directly about runway and funding plans rather than infer (§13).

### 10.4 Migration Risk and the Substitution Question

Substituting a datastore beneath a running application is a different risk class from adopting a new one. The controls that matter, in order:

1. **Command-usage inventory against the published matrix**, version-stamped, gap-list closed before any commitment (§4.4).
2. **Full application test suite plus failure-path testing** — failover, restart, replica promotion, partial `MULTI`, and the behaviour of every in-flight request during each.
3. **Dual-run with a reversible cutover**, not a flag day. The Instacart account describes the shape that works: stand the new cluster beside the old, backfill with existing pipeline tooling, and shift traffic gradually from 0% to 100% (§8.6).
4. **An explicit rollback path that has been tested in the pilot** — and note that a default `.dfs` backup is not loadable into Redis (§8.2), so if rollback relies on data migration rather than dual-write, the backup format must be chosen deliberately.
5. **Replication-based cutover has a version ceiling**: Redis→Dragonfly live replication supports Redis OSS through 6.2 (§3.5), so a bank on Redis 7.x or 8.x must plan snapshot-and-restore or dual-write.

### 10.5 Skills and Ecosystem Maturity

The genuine advantage is that **Redis protocol skills transfer directly** — client libraries work unchanged, the connection semantics are the same, and an engineer who knows Redis is productive on Dragonfly quickly. The genuine costs are that **the module ecosystem does not transfer** (`MODULE LOAD` unsupported), so any capability the bank used as a Redis module must be re-implemented or moved; that **the operational tooling does not transfer fully** (`LATENCY` family, `MEMORY DOCTOR`, `OBJECT ENCODING` unsupported), so dashboards, alert thresholds and capacity models need rebuilding; and that **the OSS cluster's control plane is the bank's own problem**, which means cluster lifecycle work that was previously a Redis Cluster detail becomes bespoke engineering.

### 10.6 What an Institution Should Require Contractually Before Depending On It

| Requirement | Why | How to test it |
|---|---|---|
| **A written licence covenant** | Stability is a policy, not a term. The BSL permits the licensor to set future terms | Obtain a contractual commitment that the version supplied and its successor versions of the same major line remain under BSL-then-Apache terms, or a non-re-licensing covenant for a defined period |
| **Recorded per-version change date** | The Apache-2.0 conversion is the bank's structural exit | Record the release date of every pinned version and its four-year conversion date in the asset register |
| **A defined support SLA** | The engine's cadence includes correctness advisories; response time matters | Contractual response and resolution targets, security-fix timelines, and a named escalation path |
| **Roadmap commitment on the specific gaps you rely on** | A gap in a command your application calls is a blocker, not a deferral | Written commitment with dates for any matrix entry your design depends on |
| **A change-of-control and continuity clause** | Single-vendor projects carry continuity risk | Trigger rights on acquisition; source availability; the licence already provides the fork right after conversion |
| **Verification of any cost guarantee** | "A guaranteed minimum of 30% cost savings" was advertised for Cloud Enterprise | Require the baseline, the measurement method and the remedy in writing before relying on it |
| **An exit plan tested once** | Exit cost is the real measure of vendor lock-in | Prove RDB-format export (set `df_snapshot_format=false` if needed) and a restore into an open-source store, once, before go-live |

Cross-reference: `../banking/operational_resilience_framework_guide.md` owns the RTO/RPO doctrine, impact-tolerance analysis and third-party dependency registration this section applies; it is condensed here deliberately.

---

## 11. The Cymbal Bank Worked Example

> **This section is fiction.** Cymbal Bank is an illustrative persona. Every number, topology and organisational detail below is invented to demonstrate a method. The **only** real figures are those explicitly attributed to a source with a date; vendor list prices are labelled as such. Nothing here should be used as a sizing input.

### 11.1 The Workload

Cymbal Bank runs the shared cache and counter tier for its retail digital channels: mobile app, internet banking, and the call-centre agent desktop. The estate today is a Redis-compatible cache in cluster mode.

| Characteristic | Value |
|---|---|
| Working set | 480 GB |
| Hot subset | ~22% (roughly 105 GB) |
| Peak throughput | 1.4M ops/sec |
| Read : write | 9 : 1 |
| Median value size | ~300 bytes (session blobs, entitlement snapshots, rendered fragments) |
| TTL profile | 30 s (OTP and step-up challenges) to 24 h (sessions); counters with no TTL |
| Latency SLO | P99 ≤ 5 ms end-to-end at the channel, of which the cache gets ~1.5 ms |
| Current topology | 24 shards, each with 2 replicas |
| Regenerability | Session and entitlement data is fully regenerable from the system of record; **rate-limit and fraud counters are not** — losing them grants a window of unlimited attempts |
| Recovery objectives | RTO 5 minutes; RPO 30 seconds for counters |

Two properties of that table drive everything else. The working set is **larger than a single reasonable instance** but with only ~22% hot, which is exactly the tiering profile the vendor describes (§3.6). And the counter workload has a **tight RPO with weak regenerability**, which is the opposite of the session data and must be treated differently in the HA design.

### 11.2 Compatibility Verification Against the Bank's Actual Command Usage

The bank extracts every command its code path can issue, including those behind feature flags, and checks each against the published matrix (v1.40.0 vs Redis 8.6.4). The result, by class:

| Usage | Matrix result | Verdict |
|---|---|---|
| `GET`, `SET` (no conditional flags), `MGET`, `MSET`, `EXPIRE`, `TTL`, `PERSIST`, `DEL`, `EXISTS` | Fully supported | Clear |
| `INCR`, `INCRBY`, `DECRBY`, `EXPIRE` — the counter tier | Fully supported | Clear |
| `HSET`, `HGETALL`, `HGETEX`, `HEXPIRE`, `HSETNX` | Fully supported | Clear |
| `ZADD`, `ZRANGE`, `ZREVRANK`, `ZINCRBY` | Fully supported | Clear |
| `LPUSH`/`BRPOP`, `LMOVE`, `LMPOP` — queue consumers | Fully supported | Clear |
| `MULTI`/`EXEC`/`WATCH` — the limiter's atomic read-modify-write | Fully supported | Clear, and the reason the existing cluster-mode workarounds can be removed |
| `EVALSHA`/`SCRIPT LOAD` — the bank's Lua limiter | Fully supported | Clear |
| `SCAN` — the reconciliation job | Fully supported | Clear |
| `FT.SUGADD`/`FT.SUGGET` — **branch name autocomplete in the call-centre desktop** | **Auto Suggest: Unsupported** | **Blocker.** Must be rehosted (Redis, or a search service) before migration |
| `CLUSTER ADDSLOTS`/`DELSLOTS` — the provisioning automation | **Unsupported** | **Work required.** Must be replaced with `DFLYCLUSTER MYID` + `DFLYCLUSTER CONFIG` topology-as-code (§3.5) |
| `MIGRATE` — the legacy cache-warming tool | **Unsupported** | **Work required.** Rewrite to `DUMP`+`RESTORE` or dual-write |
| `WAITAOF` — one library waits for durability acknowledgement | **Unsupported** | **Blocker for that library.** Remove the wait or restructure the write path |
| `CLIENT TRACKING` with `BCAST`/`PREFIX` — one service uses server-assisted client caching | **Partially supported** (those options missing) | **Work required.** Disable or reimplement |
| No `FUNCTION`/`FCALL`, no `TS`, no `MODULE`, no `GRAPH` | Unsupported but unused | No impact — verified by inventory rather than assumed |

**The finding the bank cares about is the method, not the list:** two blockers and four work items were found with the matrix alone, before any code ran. The autocomplete and `WAITAOF` findings in particular would not have surfaced from a smoke test on a greenfield deployment; they surface from an exhaustive inventory against a version-stamped matrix.

### 11.3 Licensing and Support Analysis

| Question | Answer |
|---|---|
| May Cymbal Bank self-host Dragonfly for its own applications and for its other divisions? | **Yes** — the Additional Use Grant permits use as part of your own product or service, and internal use across divisions is permitted |
| Does Cymbal Bank sell a cache or data store to third parties? | **No** for the retail estate. **Escalation:** Cymbal's white-label digital platform is offered to two partner institutions. If Dragonfly were embedded in it, that could constitute providing it "as a Service", which the Additional Use Grant prohibits. **This is the licence decision the bank must take with counsel before any platform-scale rollout** |
| What happens in 2030? | Every version converts to Apache-2.0 four years after its own first public distribution, or on 1 November 2030, whichever is first. The bank records the conversion date for whichever version it pins, and treats the Apache-2.0 build as its structural exit |
| What if the licence changes again? | The BSL is set by the licensor per version. The bank requires a licence covenant (§10.6) rather than relying on intent |
| What support is being bought? | A commercial support plan plus, for the managed path, the Cloud subscription and the Swarm control plane. A pilot without support is not a representative test, given the Instacart experience that engine-level changes were part of reaching target performance (§8.6) |

### 11.4 Migration Approach and Rollback Plan

The bank cannot use live Redis→Dragonfly replication as its primary path: the incumbent is a **Redis 7.x**-compatible cluster, and Dragonfly's Redis replication support stops at **Redis OSS 6.2** (§3.5). The adopted approach:

| Stage | Action | Exit criterion |
|---|---|---|
| 0 — Inventory | Command inventory and matrix classification (§11.2) | All blockers closed or scheduled; zero unclassified commands |
| 1 — Stand up | Two Dragonfly clusters in the non-production estate: one single-shard, one multi-shard, to test both topologies | Both serve the full test suite |
| 2 — Backfill | Populate from the incumbent using the bank's existing pipeline tooling (`DUMP`/`RESTORE` or application-level replay) | Key count, TTL distribution and checksum reconciliation match |
| 3 — Dual-run | Application clients write to both; reads served from the incumbent | Both stores converge within tolerance; divergence monitor green for two weeks |
| 4 — Shift reads | Move read traffic 1% → 10% → 50% → 100% behind a feature flag, per channel | SLOs hold at each step, on the bank's own workload, not the vendor's |
| 5 — Flip writes | Make Dragonfly primary for writes; keep the incumbent warm as the older-version population | Session continuity verified against the system of record |
| 6 — Decommission | Retire the incumbent after a defined soak period | Rollback window formally closed, with sign-off |

**Rollback.** Stage 3's dual-write is what makes the plan reversible, and it must be exercised rather than assumed: the bank rehearses a mid-flight return to the incumbent in the pilot fleet, including the state of in-flight requests. Two mechanical hazards are recorded in the plan because they are documented behaviours, not speculation: **a default `.dfs` snapshot cannot be restored into a Redis instance**, so if rollback is data-driven rather than dual-write-driven the format flag must be set deliberately (§8.2); and **a stale cluster topology config causes a node to delete keys it thinks it does not own** (§8.6), so topology changes are gated behind a change record with a mandatory post-apply verification step.

### 11.5 HA and DR Against the Bank's Recovery Objectives

The counter tier sets the requirement, because it is the non-regenerable component: **RTO 5 minutes, RPO 30 seconds**.

| Design decision | Rationale |
|---|---|
| Split the estate: sessions and entitlements on one cluster, counters on another | The two have different RPO and different regenerability; a shared blast radius is a design defect |
| Counter cluster: 3 zones minimum, replica selection by replication offset | Highest-offset promotion minimises lost counter updates on failover — this is the mechanism that protects the RPO |
| Accept that RPO 30 s is a **target, not a guarantee** | Replication is asynchronous and lag is the maximal unacknowledged offset across shards; the bank must measure it under peak load, because `WAITAOF` (the durability acknowledgement command) is unsupported |
| Self-hosted path: Kubernetes Operator automatic failover; managed path: Cloud's embedded HA | The OSS project supplies no control plane; the operator or the service supplies the failover logic |
| Validate the vendor's failover timings against the bank's own SLO | Cloud is documented at ~1 s for process failure and ~10 s for hardware failure, with ~2 min to provision a replacement node — comfortably inside a 5-minute RTO, but the bank must observe it rather than accept it |
| DR: cross-region snapshot copies, and a documented rebuild procedure from the system of record | Backups are an interim mechanism; the counter tier's rebuild procedure is the real last resort |
| Explicit decision recorded on tiering for the counter workload | Tiering is a capacity extension, not a persistence layer, and its experimental list/hash support is excluded from snapshot and replication — so tiering is **not** proposed for counters |

Cross-reference: `../banking/operational_resilience_framework_guide.md` governs the impact-tolerance analysis, the RTO/RPO derivation and the third-party dependency register that this design must satisfy; the numbers above are the *inputs* to that process, not a substitute for it.

### 11.6 Observability and Runbook Implications

| Item | Change required |
|---|---|
| Latency dashboards | Rebuild — the `LATENCY` command family is unsupported, so any dashboard or alert derived from `LATENCY HISTOGRAM`/`LATENCY RESET` has no equivalent |
| Memory diagnostics | Rebuild — `MEMORY DOCTOR` unsupported; capacity models keyed on `OBJECT ENCODING` transitions are invalid, since encoding is not observable |
| New alert: replication lag | Use the `dragonfly_connected_replica_lag_records` metric and the `lag` field in `INFO REPLICATION`; alert on the *maximum across shards*, which is the definition used |
| Metrics scraping | Point at the right port — on Kubernetes, metrics are also served on admin port 9999, and `:6379/metrics` may return an HTTP/0.9 error |
| New runbook: topology drift | A mandatory post-apply verification after any `DFLYCLUSTER CONFIG` push, because a stale config deletes unowned keys |
| New runbook: cluster node readiness | A newly started cluster-mode node rejects user requests until configured — readiness must reflect configuration state, not process state |
| New runbook: tiering operations | If tiering is adopted for the session cluster, monitor `INFO TIERED` (ram hits/misses, pending read and stash counts, pending bytes) and record the kernel ≥ 5.19 and local-NVMe prerequisites as platform standards |
| Release-management change | Fast cadence with correctness advisories means a defined patch-ingest process and a tested rollback for the datastore itself, not just the application |

### 11.7 Illustrative Cost Comparison

> **All figures in this table are ILLUSTRATIVE and invented for method demonstration, except the unit rate, which is the vendor's published list price as documented on 15 September 2026 (US$11.00 per GB per month for the Enhanced tier in AWS `us-east-1`, per dragonflydb.io/docs/cloud/pricing). A real comparison requires current quotes, committed-use discounts, egress measurement and the bank's own instance pricing.**

| Approach | Provisioned | Basis | Illustrative monthly |
|---|---|---|---|
| **Incumbent: 24 shards × (1 primary + 2 replicas)** | 24 × 3 nodes | Bank's current instance class, illustrative | see note |
| **Dragonfly, vertical, self-hosted** | 3 × large instances per region, sized to hot set + headroom | Bank's own cloud pricing; no control-plane fee; support contract added | TBD from bank's rate card |
| **Dragonfly, vertical, on Cloud** | e.g. 3 × 3-zone data stores totalling ~480 GB | **US$11.00/GB/month × GB × zones** — vendor list rate | ~US$15,840 ≈ (480 × 11) at one zone equivalent; zone count multiplies |
| **Dragonfly + SSD tiering, on Cloud** | ~120 GB RAM + ~360 GB NVMe per region | **US$16.50/GB/month** on the RAM provisioned, SSD included, per the vendor's tiered-memory rate | materially lower than all-RAM; the bank's 22% hot ratio is the favourable case |
| **Costs that must be added to any Dragonfly column** | | Support subscription; control-plane cost for the self-hosted path (build or buy); engineering time for the six work items in §11.2; runbook and dashboard rebuild (§11.6); dual-run period running both estates | usually decisive |

The method matters more than the numbers: **the bank's 22% hot ratio is the single variable most likely to make the tiered path economically compelling**, and the honest conclusion is that the *self-hosted* case is less obviously cheaper than the headline 80% figure implies, because a self-hosted deployment has to pay for the control plane in engineering rather than in subscription.

### 11.8 Decision Criteria, and What the Bank Cannot Establish Before a Pilot

**Decision criteria, in priority order:** (1) the two blockers and four work items from §11.2 are closed with dated plans; (2) the licence analysis for the white-label platform is settled in writing; (3) the counter tier's RPO is *measured* under peak load, not assumed, given `WAITAOF` is unavailable; (4) failover and failback are observed inside the 5-minute RTO on the bank's topology; (5) the vendor's cost claim is converted from marketing to a quoted, baselined comparison; (6) the contract covers the licence covenant, the SLA, the roadmap commitments and continuity (§10.6).

**What the bank cannot establish before a pilot — stated honestly:**

- **Whether the bank's real P99 holds.** Every available benchmark is vendor-published, uses a vendor-owned load generator, and explicitly excludes persistence, replication and cluster mode (§6). The bank's workload is Lua-heavy, replicated, persistent and clustered — none of which has been measured publicly on this engine.
- **Whether engine changes will be needed.** Instacart's account is that reaching expected performance took "client-side tuning **and engine-level improvements from the Dragonfly team, including changes to the compactor**" (§8.6). The bank cannot know whether its workload needs the same, and therefore cannot schedule the project without a support relationship in place.
- **Whether tiering behaves under the bank's TTL pattern.** Tiering's full support is strings-only, its list and hash support is experimental and excluded from snapshot and replication, and the bank's session values carry a heavy mix of TTLs and small values. This is pilot-only knowledge.
- **Whether the vendor's commercial position is durable.** No funding round after March 2023 could be located (§13). Runway, profitability and future funding are questions for the vendor, not inferences from public data.
- **What a future licence change would cost.** The per-version Apache-2.0 conversion bounds the damage for a pinned version, but the cost of being stranded on a four-year-old engine if terms change badly is not something the bank can price in advance. It can only be bounded contractually.
- **Whether the cloud service passes the bank's own residency, exit and concentration requirements.** BYOC improves the position substantially (§10.2); whether it satisfies a specific regulator's view is a determination the bank must obtain rather than assume.

---

## 12. The Claims Audit

Every load-bearing claim in this guide, its verdict, the source it rests on and the source's quality. Version, funding and licence claims go stale fastest; the verification date for all of them is **15 September 2026**.

| Claim | Verdict | Source and quality |
|---|---|---|
| Dragonfly is an open-source drop-in Redis/Memcached replacement | **Rejected as stated.** "Drop-in" is broadly true for the common command surface; **"open source" is false** | dragonflydb.io/docs and /docs/about/faq; github LICENSE.md; SPDX BUSL-1.1 notes — primary and unambiguous |
| Founded by Redis alumni (Ofer Bengal, Yair Gottdenker) | **Rejected.** Founders are **Oded Poncz** and **Roman Gershman** (ex-Google, ex-Ubimo; Gershman a principal on AWS ElastiCache). Bengal co-founded **Redis** (CEO 2011–2023) with **no verifiable Dragonfly connection**; Gottdenker unverified | dragonflydb.io/about (primary); TechCrunch 21 Mar 2023; redis.io/company/team/ofer-bengal and redis.io/press/redis-ceo-succession (primary) |
| US$21M raised in March 2023 (Redpoint, Quiet Capital) | **Verified.** Total across seed **and** Series A | TechCrunch 21 Mar 2023, quoting the round; Businesswire headline 21 Mar 2023 |
| Conflicting Series A figures (US$21M vs US$6M) and a US$42M total | **US$21M correct; US$42M rejected** (same event double-counted by an aggregator). **US$6M unresolved** — no primary source | vcbacked.co (consistent); startupintros.com (double-count); no source for US$6M |
| A later round (Series B or later) exists | **Not verified — no funding event located after March 2023** | github/blog/dragonflydb.io/press search, 15 Sep 2026 — absence of evidence, stated as such |
| Valuation under US$100M | **Unverified.** No round valuation has been disclosed by the company | Dealroom-style secondary only |
| Licence is BSL 1.1 with Change Date 1 Nov 2030 → Apache-2.0 | **Verified verbatim from the licence file**, and confirmed by the vendor's FAQ | github.com/dragonflydb/dragonfly/LICENSE.md — primary |
| Dragonfly Cloud ships "without gated features" | **Verified as a vendor claim**; no Enterprise-only engine features were found | dragonflydb.io/docs/cloud |
| "25× performance" and "80% lower cost" | **Flagged as marketing.** Unattributed, workload-unspecified, and materially different from the disclosed benchmarks (2.4×–4.5× engine comparisons; 1.5×–2.2× writes on current instances) | dragonflydb.io landing pages vs vendor blog measurements |
| 6.66M QPS on a single process (c7gn.metal) | **Verified as a vendor measurement** under disclosed conditions: 64-byte SET/GET, no persistence, no replication, no cluster mode, vendor's own `dfly_bench` generator | dragonflydb.io/blog, 17 Jun 2026 (method disclosed) |
| Vector search is shipped | **Verified and current** — single global HNSW index since **v1.37 (Feb 2026)**; native-width storage, EPSILON range queries and EF_RUNTIME in **v1.40.0**; `FT.*` commands partially supported | github releases; dragonflydb.io/blog 26 Feb 2026; compatibility matrix |
| SSD tiering is GA | **Verified — Community Edition, 1 Jun 2026, strings only**; on Dragonfly Cloud for AWS/GCP from 12 Aug 2026; list and hash support experimental and excluded from snapshot/replication | dragonflydb.io/blog, 1 Jun 2026 and 12 Aug 2026; /docs/managing-dragonfly/tiering |
| Current release | **v1.40.2, 3 September 2026** (marked Latest), with an upgrade advisory for a journaling correctness bug | github.com/dragonflydb/dragonfly/releases, checked 15 Sep 2026 |
| Redis licensing history (Commons Clause → RSALv2/SSPLv1 → AGPLv3) | **Verified on all three stages**, with the module dual-licence dated 15 Nov 2022, the core change announced 20 Mar 2024, and AGPLv3 added with Redis 8 GA on 1 May 2025 | redis.io/legal/licenses; redis.io/blog 20 Mar 2024 and 1 May 2025 — primary |
| Valkey is a Linux Foundation BSD-3-Clause fork of Redis 7.2.4 | **Verified** — announced 28 March 2024, with AWS, Google Cloud, Oracle, Ericsson and Snap Inc. as original participants; current release 9.1.2, 1 Sep 2026 | linuxfoundation.org press release 28 Mar 2024; valkey.io |

## 13. What Could Not Be Verified

- **Any founder, officer, adviser or investor relationship between Ofer Bengal and DragonflyDB.** Redis's own sources confirm Bengal co-founded Redis and was CEO until 2023 and Chairman from 1 February 2023; **nothing verifies a Dragonfly connection**, and the vendor's leadership page does not list him. The name **Yair Gottdenker** could not be connected to Dragonfly at all. Neither name is asserted anywhere in this guide as a Dragonfly founder.
- **The reported US$6M funding figure.** No primary source located; it may be the seed tranche of the US$21M total, but that is inference, not verification.
- **Any funding round, valuation or funding disclosure after March 2023.** No Series B, extension or debt facility was located. Supplier viability must be established directly with the vendor.
- **The exact incorporation date, legal jurisdiction and current headquarters of DragonflyDB, Ltd.** The 2022 founding year is verified only through consistent secondary evidence plus launch-state reporting, and the "San Francisco and Tel Aviv" description rests on aggregators — the vendor's own about page names neither city.
- **The body of the 21 March 2023 Businesswire release.** The URL and headline were retrievable; the page body was blocked to automated retrieval, so the funding statement rests on the headline plus TechCrunch.
- **Independent, third-party performance measurements** for Dragonfly against Redis or Valkey. Every benchmark reviewed is vendor-published, using a vendor-owned load generator. No credible neutral benchmark was located, so §6's conclusions are constrained accordingly.
- **Dragonfly Cloud's contract terms and its governing law.** The Cloud documentation references Cloud terms, support plans and pricing but the actual terms of service were not retrieved or analysed here.
- **The commercial support price list.** Support plans are documented as existing structures; no rates were retrieved.
- **Whether the matrix's "TBD" module families (Time Series, T-Digest, Auto Suggest) are actively scheduled.** The vendor's guidance is to vote on GitHub issues; no committed roadmap dates were found.
- **Behaviour of the bank-relevant edge cases**: partial-`MULTI` outcomes under failover, and the exact RPO achievable under peak load with asynchronous replication. Documentation states the mechanism; it does not state an observed worst case.

## 14. Glossary

- **BSL 1.1 (Business Source License 1.1)** — the licence Dragonfly ships under: source-available, not OSI-approved, converting to a named Change License on a Change Date.
- **Change Date / Change License** — the Change Date is 1 November 2030 for Dragonfly, though per version it is the earlier of that date and the fourth anniversary of that version's first public distribution; the Change License it converts to is Apache License 2.0.
- **Additional Use Grant** — the BSL clause that permits internal production use and prohibits providing the software "as a Service" competing with the licensor.
- **DashTable** — Dragonfly's hash table: an extendible-hashing structure of segments addressed through a directory, with 56 regular and 4 stash buckets of 14 slots per segment (≤840 entries), adapted from a 2020 VLDB paper on DASH.
- **Extendible hashing** — the algorithmic family (Fagin et al., 1979) behind DashTable; growth splits one hash-prefix range rather than rehashing the whole table.
- **Shared-nothing** — an architecture in which each thread owns a disjoint slice of the keyspace and its own memory, with no shared mutable table.
- **Proactor thread / `--proactor_threads` / fibers** — the Dragonfly thread that runs an asynchronous event loop and owns shard data and I/O (the unit of vertical scaling), using stackful coroutines (fibers) so in-flight I/O can be suspended and resumed without blocking the thread.
- **`io_uring`** — the Linux asynchronous I/O interface Dragonfly uses for disk work; required for SSD tiering (kernel 5.19+).
- **Forkless snapshotting** — Dragonfly's point-in-time serialisation of each shard by its own thread, using entry versions and a captured epoch instead of forking the process.
- **`.dfs`** — Dragonfly's own snapshot file format, the default (`df_snapshot_format=true`), as distinct from Redis-compatible RDB.
- **SSD data tiering** — offloading values (not keys) to local NVMe, with hot/cold/cooled states; GA for strings since June 2026.
- **`DFLYCLUSTER CONFIG`** — the admin-port command that pushes a JSON topology to a node in multi-shard cluster mode; the replacement for Redis's `CLUSTER ADDSLOTS` family.
- **Dragonfly Swarm** — the vendor's managed control plane for Dragonfly clusters, sold as part of Dragonfly Cloud rather than shipped in the OSS server.
- **Smart client / emulated cluster mode** — the default single-node mode in which one Dragonfly instance answers Redis Cluster protocol commands as a one-shard cluster.
- **RSALv2 / SSPLv1 / AGPLv3** — Redis's current tri-licence; RSALv2 and SSPLv1 are source-available, AGPLv3 is OSI-approved. Earlier Redis releases were BSD-3-Clause.
- **`dfly_bench`** — the Dragonfly team's own load generator, "very similar to memtier_benchmark", used in the vendor's published benchmarks.
- **HNSW / EF_RUNTIME** — Hierarchical Navigable Small World, the graph index used for approximate nearest-neighbour vector search (Dragonfly moved from a per-shard index to a single global one in v1.37); EF_RUNTIME is the HNSW search-time candidate-list size that controls the throughput/precision trade-off.

## 15. Cross-References and Further Reading

**Within this repository:** `technology/copilot_data_cache_guide.md` and `technology/ai_llm/agent_runtime_cache_design_guide.md` own caching patterns, eviction and cache-design discipline; `technology/distributed_rate_limiter_guide.md` owns rate-limiting algorithms; `technology/kafka_alternatives_guide.md` owns alternatives-landscape framing; `technology/nosql_data_modelling_guide.md` owns key design and access-pattern discipline; `technology/ai_llm/rag/vector_databases_guide.md` owns vector-database comparison; `../banking/operational_resilience_framework_guide.md` owns RTO/RPO and third-party risk doctrine; `technology/architecture/billion_user_system_arch.md` and `technology/shell_testing_frameworks_comparison_guide.md` are the two guides in this repository that mention Dragonfly in passing.

**Primary sources, all retrieved 15 September 2026:** github.com/dragonflydb/dragonfly (repository, `LICENSE.md`, releases, compatibility matrix); dragonflydb.io/docs (compatibility, license, FAQ, snapshotting, replication, cluster mode, tiering, backups, monitoring, Kubernetes operator, Cloud, Cloud pricing); dragonflydb.io/blog (SSD tiering GA 1 Jun 2026 and Cloud 12 Aug 2026; vector search v1.37 26 Feb 2026; Graviton/Valkey 9.0 17 Jun 2026; GCP/Valkey 8.0.2 4 Mar 2025; failover 20 Apr 2026; from-dict-to-Dashtable 30 Jul 2026; Instacart 14 Jul 2026; 2025 recap 18 Dec 2025); dragonflydb.io/about; redis.io/legal/licenses, redis.io/blog/redis-8-ga (1 May 2025), redis.io/blog/redis-adopts-dual-source-available-licensing (20 Mar 2024), redis.io/company/team/ofer-bengal, redis.io/press/redis-ceo-succession (5 Dec 2022); valkey.io and valkey.io/topics/faq; linuxfoundation.org press release 28 Mar 2024; spdx.org/licenses/BUSL-1.1.html; TechCrunch 21 Mar 2023.

## 16. Closing Summary

Dragonfly is the most technically interesting entrant in the in-memory data store market and the one whose *marketing* is furthest from its *documentation*. The engine is real: a shared-nothing, thread-per-core design, a hash table built on extendible hashing that removes Redis's rehash spike and its `fork()`-based snapshot, asynchronous `io_uring` I/O, and a beyond-RAM tier that neither Redis nor Valkey offers. The compatibility is real but partial, and the project publishes an honest matrix that contradicts its own "no code changes" slogan in two blockers and a dozen lesser gaps. The licence is not open source, and the vendor says so while defending the choice.

For a regulated buyer the summary is narrower than the marketing and more useful than a rejection. Adopt when vertical headroom, memory efficiency per key, or a working set larger than RAM is the binding constraint — the feature-store and large-cache cases where the evidence, including the only named production accounts, is strongest. Do not adopt to replace a Redis module, a Redis Function, a Redis Query Engine feature set, or a cluster your tooling manages, because those are documented walls rather than gaps. And require the licence covenant, the measured RPO, the tested exit and the support contract before, not after, the cutover — because the durable difference between Redis's history and Dragonfly's licence is not the terms on the page but whether the vendor has put them beyond its own future discretion.

The design decisions that make all of this true are not in the marketing at all. They are in a hash table that splits one prefix range at a time instead of doubling a dictionary, and in the discipline that follows from it — no rehash spike, no fork, a snapshot that costs a version counter rather than a second copy of memory. Every throughput figure in every benchmark, every byte saved on every cloud bill, and every claim in this guide that survived scrutiny traces back to the choice of the dash table.
