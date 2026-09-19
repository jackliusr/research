# YugabyteDB: The Distributed PostgreSQL Fork — Architecture, Transactions, Licensing and Operations

> A deep dive into the distributed SQL database that chose to **fork** PostgreSQL and embed it in a Raft-replicated document store rather than reimplement SQL. This guide resolves the half-verified YugabyteDB rows left open by `cockroachdb_guide.md`.

**Jack Liu Shurui, Solution Architect**

| Field | Value |
| --- | --- |
| Research date | 19 September 2026 |
| Primary sources checked this pass | `github.com/yugabyte/yugabyte-db` (`LICENSE.md`, README), `docs.yugabyte.com/stable/*`, `yugabyte.com` (About, News) |
| Documented target release | v2026.1 (STS, released 29 June 2026) — current stable series at time of writing |
| Scope | The database: identity, architecture, YSQL, YCQL, transactions, geo-distribution, licensing, operations, comparisons |
| Out of scope | Consensus/replication theory (`distributed_systems_engineering_guide.md`); the distributed-SQL taxonomy and CockroachDB deep dive (`cockroachdb_guide.md`); data modelling (`nosql_data_modelling_guide.md`) |
| Supersedes | The ⚠ and ⚠-structural YugabyteDB rows in `cockroachdb_guide.md` §9.3 and §9.5, itemised and resolved in §10.4 and §14.4 below |

Markers used throughout: ✅ verified this pass against a live primary source; ⚠ flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source quoted here.

## Table of contents

1. [Overview, identity and the decoder](#1-overview-identity-and-the-decoder)
2. [Identity and corporate history](#2-identity-and-corporate-history)
3. [Architecture: DocDB, tablets, Raft and the clock](#3-architecture-docdb-tablets-raft-and-the-clock)
4. [The YSQL layer: a PostgreSQL fork, not a reimplementation](#4-the-ysql-layer-a-postgresql-fork-not-a-reimplementation)
5. [The YCQL layer and the multi-API story](#5-the-ycql-layer-and-the-multi-api-story)
6. [Transactions and consistency](#6-transactions-and-consistency)
7. [Geo-distribution and multi-region](#7-geo-distribution-and-multi-region)
8. [Licensing and the commercial model](#8-licensing-and-the-commercial-model)
9. [Operations](#9-operations)
10. [Comparisons: resolving the sibling guide's flags](#10-comparisons-resolving-the-sibling-guides-flags)
11. [The regulated-enterprise and banking angle](#11-the-regulated-enterprise-and-banking-angle)
12. [The Cymbal Bank worked example](#12-the-cymbal-bank-worked-example)
13. [Gotchas and anti-patterns](#13-gotchas-and-anti-patterns)
14. [Claims audit](#14-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary](#16-glossary)
17. [Cross-references and further reading](#17-cross-references-and-further-reading)
18. [Closing summary](#18-closing-summary)

## 1. Overview, identity and the decoder

**YugabyteDB is a distributed, horizontally scalable, ACID-transactional SQL database whose SQL surface is a fork of PostgreSQL rather than a reimplementation of SQL.** It splits every table into *tablets*, replicates each tablet with a Raft group, stores rows in *DocDB* (a document store on a hardened RocksDB), and exposes two query APIs over one substrate: YSQL (PostgreSQL wire and grammar) and YCQL (Cassandra CQL).

### 1.1 The framing insight

| Approach | What you do | Who took it |
| --- | --- | --- |
| **Reimplement SQL** | New optimiser, executor, catalog, type system, protocol — and permanent compatibility debt against PostgreSQL semantics | CockroachDB (own SQL layer over own KV store) — see `cockroachdb_guide.md` |
| **Fork PostgreSQL** | Keep PostgreSQL's parser, planner, catalog, type system and wire protocol; replace the storage and execution substrate underneath | **YugabyteDB** |

The vendor states its camp plainly: *"YugabyteDB's YSQL API reuses a fork of the query layer of PostgreSQL and hence is fully compatible with PostgreSQL"* ✅, adding that this *"allows YSQL to support most PostgreSQL features, such as data types, queries, expressions, operators and functions, stored procedures, triggers, extensions"* ✅ (docs.yugabyte.com, SQL features page, checked 19 Sep 2026).

That one decision cascades: it is why a PostgreSQL/PL-pgSQL estate can consider the product at all; it is why upgrades are coupled to a fork base (§4.3); and it is why "PostgreSQL compatible" needs a feature-level audit, not a yes/no answer (§4.4, §13).

### 1.2 Company versus product naming

| Name | What it actually is |
| --- | --- |
| **Yugabyte** | The company (formerly capitalised *YugaByte*; the site and repo use *Yugabyte*) |
| **YugabyteDB** | The database — the Apache-2.0 core project |
| **YSQL** / **YCQL** | The PostgreSQL-compatible and Cassandra-compatible APIs of the same database |
| **YugabyteDB Anywhere (YBA)** | The self-managed control plane / orchestration platform (separately licensed — §8) |
| **YugabyteDB Aeon** | The vendor's managed DBaaS (originally *YugabyteDB Managed*), introduced 2021 ✅ |
| **YugabyteDB Voyager** | Migration tooling brand |
| ***universe*** | A deployed instance — one primary cluster plus any read-replica clusters |

### 1.3 The decoder table

| Term | Decoder |
| --- | --- |
| **DocDB** | *"DocDB is the underlying document storage engine of YugabyteDB and is built on top of a highly customized and optimized version of RocksDB"* ✅; it manages *"multiple RocksDB instances which are created one per tablet"* ✅ |
| **Tablet** | The unit of sharding *and* replication — *"tablets a.k.a shards"* ✅. Each has its own Raft group, log and RocksDB instance; splits automatically past a size threshold ✅ |
| **Sharding** | *"The mapping of a row to a tablet is deterministic and this process is known as sharding"* ✅; hash and range both supported natively ✅ |
| **Hash sharding** | Rows spread by hash of the primary key — even write spread; suits high-volume point access |
| **Range sharding** | Rows in key order across tablets — efficient ordered scans; suits time-series and ordered reporting |
| **Colocated tables** | Small tables placed in a shared tablet so each does not pay for its own Raft group ✅ |
| **YB-TServer** | Tablet server: hosts tablets, serves queries, participates in Raft |
| **YB-Master** | *"responsible for keeping system metadata, coordinating system-wide operations, such as creating, altering, and dropping tables, as well as initiating maintenance operations such as load balancing"* ✅ |
| **Raft group** | One per tablet; *"Amongst the tablet replicas, one tablet is elected leader as per the Raft protocol"* ✅ |
| **Replication factor (RF)** | *"providing a fault tolerance of `ft` requires replicating data across `2ft + 1` domains"* ✅ — RF 3 tolerates one domain failure |
| **Hybrid logical clock (HLC)** | The timestamp mechanism: (physical, logical) tuple; **not** TrueTime ✅ (§3.7) |
| **MVCC** | Multi-Version Concurrency Control, *"without the need to lock rows"* ✅; uncommitted values held as *provisional records* ✅ |
| **Follower reads** | Reads served from a replica at a configurable staleness; explicitly not linearizable ✅ (§7.4) |
| **Read replica** | Async read-only copy of the whole universe (observer node); does not vote in Raft; *"timeline consistency"* ✅ |
| **xCluster** | Async cross-cluster replication, unidirectional or bidirectional, per table; RPO *"Some data loss"* ✅ |
| **Geo-partitioning** | Row-level pinning of data to regions by policy, for residency or latency ✅ |
| **Universe** | *"Sometimes the term cluster is used interchangeably with the term universe. However, the two are not always equivalent"* ✅ — a universe can hold primary plus read replicas |
| **STS / LTS / preview** | Two stable releases a year; the `.1` release is STS, the `.2` is LTS; preview is not production-supported ✅ |
| **TP / EA / GA** | Feature maturity: Tech Preview, Early Access, General Availability ✅ |
| **Apache License 2.0** | Licence of the database core — all features, one edition ✅ |
| **Polyform Free Trial License 1.0.0** | Licence of YugabyteDB Anywhere: *"permits evaluation use for up to 32 consecutive calendar days"*; commercial licence required for production ✅ |

### 1.4 Boundary statement

- **This guide does not re-derive the distributed-SQL taxonomy, the SQL-vs-NoSQL spectrum or CockroachDB's architecture** — those live in `cockroachdb_guide.md` §1–§9. Its §9.5 head-to-head table flagged several YugabyteDB rows it could not verify; **resolving those rows is this guide's job** (§10.4, §14.4).
- **This guide does not re-explain Raft, quorum arithmetic, PACELC or the consistency spectrum** — those live in `distributed_systems_engineering_guide.md` §5–§7, cited by section rather than repeated.
- **This guide owns** YugabyteDB's identity, architecture, YSQL and YCQL surfaces, transaction and isolation behaviour, geo-distribution, licensing, operations and a fully sourced CockroachDB comparison.
- **Per-database consistency knobs** across products belong to the repo's dedicated consistency material; this guide states only YugabyteDB's own defaults and flags (§6).

## 2. Identity and corporate history

Corporate facts decay fastest, so everything here is dated. Sources checked **19 September 2026**.

### 2.1 Founding and founders — the claim the sibling guide flagged

The sibling's ⚠-structural founding claim is now **verified** ✅, though the vendor's wording differs from the sibling's paraphrase:

> *"Yugabyte was founded by three former Facebook engineers: Kannan Muthukkaruppan, Karthik Ranganathan, and Mikhail Bautin."* ✅

> *"In February 2016, the founders started building YugabyteDB, a global-scale distributed SQL database designed for cloud-native transactional applications."* ✅

> *"They have a proven track record in building and operating distributed databases, including Cassandra — one of the first web-scale databases — and HBase, a popular non-relational database."* ✅

| Fact | Value | Date checked | Marker |
| --- | --- | --- | --- |
| Founders | Kannan Muthukkaruppan, Karthik Ranganathan, Mikhail Bautin | 19 Sep 2026 | ✅ |
| Prior employer / lineage | Facebook; Cassandra and HBase | 19 Sep 2026 | ✅ |
| Founding year | 2016 (engineering started February 2016) | 19 Sep 2026 | ✅ |
| Current leadership | Ranganathan and Muthukkaruppan both listed **Co-Founder & Co-CEO** | 19 Sep 2026 | ✅ |

**House analysis:** the exact "ex-Facebook engineers" framing is accurate but incomplete — the lineage the vendor advertises is **Cassandra and HBase**, which is the intellectual ancestry of the YCQL API (§5) and of the wide-column storage model underneath DocDB. A team that ran HBase at Facebook scale would reach naturally for "RocksDB + tablets + Raft" rather than "own LSM + own consensus + own SQL".

### 2.2 Naming history

| Name | Status |
| --- | --- |
| *YugaByte* (mixed case) | Historical capitalisation in early material; current repo and site use **Yugabyte** ✅ |
| *yuga* + *byte* | The vendor states the name *"blends the Sanskrit word 'yuga,' meaning 'era' (or long period of time) with 'byte'"* ✅ |
| *YSQL* / *YCQL* | The two query APIs — PostgreSQL grammar and Cassandra grammar respectively ✅ |
| *universe* | Vendor term for a deployed instance: primary cluster plus read replicas ✅ |
| *YugabyteDB Managed* → *YugabyteDB Aeon* | Managed DBaaS *"introduced in 2021"*, later renamed ✅ |
| The "Kudu"-related earlier project name | **Not confirmed** on the vendor's own pages; deliberately not restated here ⚠ |

### 2.3 Funding, ownership and current corporate status

The vendor's own investor block ✅ lists: Lightspeed Venture Partners; Sapphire Ventures; Dell Technologies Capital (VC arm of Dell Technologies); 8VC; Alkeon Capital Management; Meritech Capital; **Wells Fargo** (*"strategic investment arm… focusing on technology partnerships"*); Wipro Ventures (*"strategic investment arm of Wipro Limited"*). Board: Ravi Mhatre (Lightspeed), Deepak Jeevankumar (Dell Tech Capital), Bhaskar Ghosh (8VC), Jai Das (Sapphire), plus both co-CEOs; Scott McNealy is listed as an advisor ✅.

**Current corporate status as of 19 September 2026 — the highest-decay fact in this guide, stated explicitly:**

- Yugabyte continues to operate as an **independent, venture-backed private company** under its own name, with its own leadership, board and investors ✅.
- **No acquisition, merger or restructuring was found on the vendor's own site or news page as of 19 September 2026 — the trail shows none.** This is a *negative finding from the sources checked*, not proof of absence (§15). The vendor is still publishing first-party product and partnership announcements under its own name, which is the observable signal of an independent operating company ✅.
- **No total-raised figure or round-by-round history could be verified from a primary source this pass.** Funding databases are secondary aggregators and were not reachable with confidence; **no funding total is quoted anywhere in this guide** ⚠.

Recent first-party signals ✅ (vendor news page, 19 Sep 2026): "Yugabyte Accelerates Agentic AI Momentum with New Product Lines, Global Expansion, and Industry Recognition" (Sep 2026); "Yugabyte Unveils Global Partner Program…" (Sep 2026); a study on "the Hidden Cost of Legacy Database Architectures" (Jul 2026). **House analysis:** the market framing has moved from "distributed SQL for transactional apps" toward "PostgreSQL for AI/agent workloads" — a positioning shift, not a technical change, but it tells you where go-to-market energy is directed.

**Vendor marketing figures** ✅ (about page): *"1M+ transactions per second"*, *"10K GitHub stars"*, *"100+ countries"*, *"11,000+ community members"*. Recorded only so a reader knows their origin. **No performance or scale claim in this guide rests on them.**

## 3. Architecture: DocDB, tablets, Raft and the clock

Two logical layers ✅: the **query layer** (handles user requests; routes queries to the right tablets) and the **storage layer** (stores data on disk; manages replication and consistency). Two APIs sit over both: **YSQL** and **YCQL**.

### 3.1 DocDB, the storage core

- *"DocDB is the underlying document storage engine of YugabyteDB"*, built on a *"highly customized and optimized version of RocksDB, a log-structured merge tree (LSM)-based key-value store"* ✅.
- It is an **ordered** persistent key-value store: *"It supports ordered data operations, allowing efficient range queries and iteration over keys"* ✅ — which is what makes range sharding possible.
- LSM storage: data lands in sorted string tables (SSTs); *"DocDB periodically compacts data by merging and sorting multiple SST files"* ✅. The consequence is the standard LSM trade — excellent write throughput, compaction as a background cost to capacity-plan (⚠-structural).
- Written in C++ ✅. **One RocksDB instance per tablet** ✅ — the sentence that makes the architecture click: a tablet is not a logical range in a shared store, it is a separately managed storage instance with its own Raft log.

### 3.2 The tablet: unit of sharding *and* replication

- *"These shards are distributed across multiple server nodes… in a shared-nothing architecture. The application interacts with a SQL table as one logical unit and remains agnostic to the physical placement of the shards"* ✅.
- **Automatic splitting**: *"Once a tablet reaches a threshold size, it automatically splits into two. These 2 new tablets can now be placed in other nodes to keep the load on the system balanced"* ✅.
- **Cluster balancing** moves both data and leaders after scale-in/out, outages, and table creation/deletion ✅.

**Why this matters operationally:** every tablet carries the fixed cost of a consensus group. A schema with 5,000 small tables sharded naively produces tens of thousands of Raft groups, each with heartbeats, leases and leader bookkeeping. That is why colocated tables exist ✅. This is the single most consequential schema-design decision in a migration (§4.4, §12, §13).

### 3.3 Hash versus range sharding

| Dimension | Hash sharding | Range sharding |
| --- | --- | --- |
| Row placement | By hash of the primary key — deliberately unordered | In key order, contiguous across tablets |
| Writes | Evenly spread; avoids append-tail hotspots | Can concentrate on a "last" tablet if the key is monotonic |
| Ordered scans | Not cheap — ordering destroyed by hashing | Efficient — the reason to choose it |
| Typical fit | High-volume OLTP point access | Time-ordered data, ledger-style range reporting |
| Guardrail | Poor for range-scan-heavy access | A monotonically increasing key can create a persistent hot tablet |

**House analysis:** the choice is baked into the primary key's physical mapping and is not cheaply reversible. The wrong choice surfaces months later as either a hot tablet or a slow range query — and "PostgreSQL compatible" offers no protection, because the SQL looks identical either way.

### 3.4 The Raft group per tablet

*"Amongst the tablet replicas, one tablet is elected leader as per the Raft protocol"* ✅; fault tolerance follows *"`2ft + 1`"* domains ✅; and *"Normally, only the tablet leader can process user-facing write and read requests"* ✅.

Two consequences to internalise: **(1) write latency is leader latency** — a write is acknowledged only after a Raft majority replicates it, so spreading replicas across regions spreads that round trip (§7); **(2) failure costs a re-election per tablet** — a node loss affects every tablet it hosted, in parallel, so blast radius is counted in tablets. Raft mechanics themselves belong to `distributed_systems_engineering_guide.md` §6.

### 3.5 A write's path, and a node failure, in words

```
write:  client (psql / pg driver / ycqlsh) → any YB-TServer (query layer)
          → parse/plan (PostgreSQL fork for YSQL); resolve key → tablet
          → tablet LEADER: append to that tablet's Raft log
               follower: append + ack
               follower: append + ack        ← MAJORITY replicated = committed
          → DocDB applies into the tablet's RocksDB; HLC assigns the timestamp → ack

failure: T-Server N dies
          → tablets N led: followers time out; a majority elects a new leader; writes resume
          → tablets where N was only a follower: leader keeps serving; N catches up on return
          → balancer re-replicates under-replicated tablets and rebalances leader counts
```

Multi-tablet writes add a transaction layer: uncommitted values are written as **provisional records**, because written as ordinary values they *"would then become visible at different times to clients reading through different tablet servers, allowing a client to see a partially applied transaction and thus breaking atomicity"* ✅.

- **Unavailability is per-tablet, not cluster-wide** — tablets with a majority intact re-elect quickly.
- **Fault domain choice is a deployment decision**: *"The fault domain can be at the level of individual nodes, availability zones, or entire regions"* ✅. RF 3 across three zones tolerates a zone loss; RF 3 across two zones makes the second zone loss a full outage for affected tablets — the vendor's own arithmetic ✅.

### 3.7 The clock: hybrid logical clocks, and what they do *not* give you

- **There is no TrueTime.** *"TrueTime, used by Google Cloud Spanner, is an example of such a clock with tight error bounds. However, this type of clock is not available in many deployments. Physical time clocks (or wall clocks) cannot be perfectly synchronized across nodes"* ✅.
- Instead, an **HLC** based on the hybrid time algorithm — *"combines the advantages of local real-time (physical) clocks and Lamport clocks that track causal relationships"* ✅. It is a tuple (physical component, logical component): *"HLCs generated on any node are strictly monotonic"*, physical takes precedence, the physical component is initialised from `CLOCK_REALTIME` and *"can only be updated to a higher value"*, and the logical component resets to 0 when physical time advances ✅.
- **Skew propagates on every RPC**: *"On any RPC communication between two nodes, HLC values are exchanged. The node with the lower HLC updates its HLC to the higher value"* ✅ — so a fast clock drags the cluster forward.
- The vendor states the cost under a heading called "Caveat": *"the conflict resolution depends on the maximum clock skew in the cluster. This leads to a higher number of transaction conflicts or a higher latency of the transaction."* ✅

| Question | Answer |
| --- | --- |
| Spanner-style externally consistent timestamps? | **No** — not without the bounded-uncertainty clock the vendor explicitly lacks ✅ |
| Still correct? | Yes — correctness comes from **Raft majority replication plus MVCC** over hybrid time: *"If an update has safely been replicated onto a majority of nodes, as per the Raft protocol… it is safe to serve all reads up to that HLC"* ✅ |
| Does skew matter? | Yes, measurably — it is named as the driver of more conflicts and higher latency ✅. Clock synchronisation is an availability-relevant requirement, not housekeeping |
| Is there a knob? | A maximum-clock-skew bound parameter exists (commonly documented as `max_clock_skew_usec`); name and default were **not** re-verified this pass, so recorded as ⚠ (§15) |

**This guide's analysis:** the sibling's flagged row "TiDB/YugabyteDB → similar HLC approaches" ⚠-structural is **substantively correct but must be read carefully**. It is a statement about the *timestamp mechanism*, not about the *guarantee*. Neither product offers external consistency, and both depend on bounded skew for conflict behaviour. The row is resolved as **true-but-not-load-bearing**.

## 4. The YSQL layer: a PostgreSQL fork, not a reimplementation

### 4.1 What "fork" means here

*"YugabyteDB's YSQL API reuses a fork of the **query layer** of PostgreSQL and hence is fully compatible with PostgreSQL"* ✅. The repo's own licence file agrees: `src/postgres/` is *"Modified PostgreSQL fork for YSQL compatibility"* ✅.

| Layer | Origin |
| --- | --- |
| Parser, planner, type system, catalog, pg_catalog, wire protocol, PL/pgSQL VM | PostgreSQL source (forked) ✅ |
| Storage engine | DocDB over RocksDB ✅ |
| Replication and consensus | Raft group per tablet ✅ |
| Distributed transaction manager, provisional records, MVCC on hybrid time | YugabyteDB-native ✅ |
| Sharding, placement, balancing, cluster metadata | YB-Master + YB-TServer ✅ |

**House analysis:** this is the inverse of the CockroachDB bet. CockroachDB keeps its own SQL layer and pays a continuous compatibility tax; YugabyteDB keeps PostgreSQL and pays a periodic *fork-base* tax (§4.3). Neither is free; they fall due at different times.

### 4.2 The fork base: 11.2 → 15.0

| Property | Value | Marker |
| --- | --- | --- |
| Original fork base | **PostgreSQL 11.2** | ✅ |
| Current fork base | **PostgreSQL 15.0** | ✅ |
| Pre-PG-15 line | *"all versions prior to v2.25"* | ✅ verbatim |
| PG-15 line | v2.25 or later (preview); **v2025.1 or later (stable)** | ✅ verbatim |
| Significance of v2025.1 | repo README: *"the first stable release featuring a PostgreSQL fork rebase from version 11.2 to 15.0"* | ✅ verbatim |

The rebase brought **stored generated columns**, **foreign keys on partitioned tables** and **non-distinct NULLs in unique indexes** ✅.

**This is the most under-appreciated fact about running YugabyteDB in a bank.** For most of the 2023–2025 window the stable product shipped with the SQL semantics of a PostgreSQL released in **February 2019**. Any migration assessment that says "it's PostgreSQL" without naming the fork base is not an assessment.

### 4.3 Maintaining and moving the fork base

| Dimension | Normal upgrade | YSQL major upgrade (PG 11 → 15) |
| --- | --- | --- |
| Maturity | GA path | **EARLY ACCESS** ✅ |
| Online? | — | *"The upgrade is fully online. While the upgrade is in progress, you have full and uninterrupted read and write access to your cluster."* ✅ |
| DDL | Available | **Blocked** — all DDL except temp-table DDL and Refresh Materialized View; DMLs allowed ✅ |
| Minimum source version | — | Cluster must run **v2024.2.3.0 or later** ✅ |
| Drivers | — | Must be upgraded; backward compatible across the two DB versions ✅ |
| Dedicated masters | — | A superuser named `yugabyte_upgrade` must exist with credentials in `.pgpass` on each master node ✅ |
| PITR | — | Configuration deleted before the upgrade, recreated after finalize/rollback ✅ |
| `pg_stat_monitor` | — | Dropped before, re-created after ✅ |
| Precheck | — | `yugabyted upgrade check_version_compatibility --base_dir=...` prints *"Clusters are compatible for upgrade."* ✅ |
| Tooling | — | `pg_upgrade` is integrated into the distributed architecture rather than replaced ✅ |

Read together: **the DDL freeze alone makes this a change-window project with a schema-change embargo**, even though reads and writes continue.

### 4.4 What the fork buys, and what it costs

| Dimension | Buys | Costs |
| --- | --- | --- |
| Grammar and semantics | PostgreSQL parser, planner, types, expressions, operators, functions ✅ | The fork's *vintage* of PostgreSQL, not upstream's current behaviour (§4.2) |
| Procedures and triggers | PL/pgSQL and trigger machinery are part of the fork ✅ — a bank's procedural estate transfers far more readily than to a reimplemented SQL layer | Procedures assuming single-node execution, local sequencing or heavy temp-table use still break |
| Extensions | *"robust support for PostgreSQL extensions"* ✅ | Not all extensions work distributed; validate each against the current matrix |
| Drivers | Standard PostgreSQL wire protocol and drivers ✅ | Must be version-validated for the major upgrade ✅ |
| Types | *"INTEGER, VARCHAR, BOOLEAN, TIMESTAMP… arrays, JSONB, and XML"* ✅ | Sharding key, colocation and cross-tablet join cost are new decisions |
| Feature parity | *"a large portion of the documentation and examples written for PostgreSQL would work against YSQL"* ✅ | "Most" and "a large portion" are the vendor's own hedges — read the compatibility matrix per version ⚠ |

The authoritative unsupported/differing list is the vendor's PostgreSQL-compatibility feature matrix (docs.yugabyte.com …/ysql-language-features/postgresql-compatibility/) — a document to open, date and diff per release, **not** to copy into a guide. It failed to load during this pass, so **no claim here rests on its contents** (§15).

### 4.5 Drivers, pooling and operational shape

YSQL speaks the PostgreSQL wire protocol, so ordinary PostgreSQL drivers connect ✅. The repo's licence map shows `src/odyssey/` — a PostgreSQL connection pooler with its **own BSD-style licence** ✅ — as a core component, plus `java/` (client libraries) and `python/` (build/test utilities) ✅.

**This guide's analysis:** pooling matters more, not less, in a distributed database; application servers opening hundreds of connections per instance behave differently against a cluster, and the failure mode (connection exhaustion during a partial outage) looks exactly like a classic database incident. Plan the pooling layer explicitly (§12).

## 5. The YCQL layer and the multi-API story

YCQL is the Cassandra-Query-Language-compatible API of the same DocDB substrate — an ancestry traceable to the founders' HBase/Cassandra lineage ✅.

| Property | YCQL | YSQL |
| --- | --- | --- |
| Grammar | Cassandra CQL | PostgreSQL |
| Isolation | **Snapshot only** ✅ (*"Although YugabyteDB supports only Snapshot isolation level in the YCQL API, it supports three levels of isolation in the YSQL API"*) | Serializable, Snapshot, Read Committed ✅ |
| Transactions | Opt-in **per table**: `WITH transactions = { 'enabled' : true }` ✅ | Always available — every DocDB update is a transaction ✅ |
| Syntax | `BEGIN TRANSACTION … END TRANSACTION` ✅ | `BEGIN … COMMIT/ROLLBACK`, autocommit per statement ✅ |
| Model | Wide-column, keyspaces | Relational schemas, constraints, indexes ✅ |
| Follower reads | *"looser consistency model"* than YSQL ✅ | Staleness-bounded and strictly ordered ✅ |
| Ecosystem | Cassandra drivers and tooling | PostgreSQL drivers, ORMs, extensions ✅ |

Two details to carry forward. **First, transactions are opt-in per table in YCQL** ✅ — a table without the `transactions` property is not giving you the guarantees you may assume from the YSQL side (the vendor's worked example creates `banking.accounts` explicitly with `transactions = {'enabled': true}` ✅). **Second, client-side retries can break linearizability even though the database is not broken**: *"Automatic retries can break linearizability of operations from the client point of view… the client gets a successful response to a retried request and treats the operation as completed, but the value might get overwritten by an older operation due to retries"* ✅; the remedy is a driver policy (`NoRetryOnClientTimeoutPolicy`) with timeouts handled in the application layer ✅.

| Position | Argument |
| --- | --- |
| **Advantage** | One cluster, one runbook, one backup story, one upgrade story, two application compatibility surfaces — an estate with both PostgreSQL and Cassandra workloads consolidates *infrastructure* without rewriting the Cassandra-facing services |
| **Complexity** | Two dialects, two isolation models (three levels vs one ✅), two transaction idioms (always-on vs per-table opt-in ✅), two driver ecosystems, two skill pools. Shared storage, **unshared semantics** |
| **Deciding question** | Does the estate genuinely have both workloads under one blast radius, and staff who can hold both models? If either is no, the second API is pure surface area |

**House analysis:** the multi-API story is *consolidation of infrastructure, not of semantics*. Data-modelling questions belong to `nosql_data_modelling_guide.md` and are not re-derived here.

## 6. Transactions and consistency

### 6.1 Isolation levels and their defaults

Three levels exist in the transactional layer: **Serializable, Snapshot, Read Committed** ✅ (checked 19 Sep 2026). The headline default, verbatim:

> *"The default isolation level for the YSQL API is effectively Snapshot (that is, the same as PostgreSQL's REPEATABLE READ) because, by default, Read Committed, which is the YSQL API and PostgreSQL syntactic default, maps to Snapshot isolation."* ✅

And the change that supersedes it for new deployments:

> *"However, for new universes running v2025.2 or later, Read Committed is enabled by default when you deploy using yugabyted, YugabyteDB Anywhere, or YugabyteDB Aeon."* ✅

| Version / deployment situation | Effective default for a `BEGIN` naming no level |
| --- | --- |
| v2025.2+ deployed via yugabyted, YBA or Aeon | **Read Committed** ✅ |
| Earlier than v2025.2, **or** any manually deployed universe, without the flag | **Snapshot** — `yb_enable_read_committed_isolation` defaults `false` and Read Committed *"falls back to the stricter Snapshot isolation"* ✅ |
| Same, with `yb_enable_read_committed_isolation=true` | Read Committed ✅ |

**This is a real migration hazard.** Read Committed is the SQL standard's and PostgreSQL's *syntactic* default, so applications that name no level get it by name — but on older or manually deployed clusters that name resolves to the stricter **Snapshot**, which produces *more* aborts in workloads written for Read Committed. The vendor's own tip ✅:

> *"To avoid serializable errors (that is, to run applications with no retry logic), keep the default Read Committed isolation (`--ysql_default_transaction_isolation`), and set the YB-TServer `--yb_enable_read_committed_isolation` flag to true."*

Per-transaction control is via `SET TRANSACTION` ✅.

### 6.2 The anomaly mapping, as the vendor documents it

| SQL standard | YugabyteDB equivalent | Dirty read | Non-repeatable read | Phantom read | Serialization anomaly |
| --- | --- | --- | --- | --- | --- |
| Read Uncommitted | Read Committed | Allowed, but not in YSQL | Possible | Possible | Possible |
| Read Committed | Read Committed | Not possible | Possible | Possible | Possible |
| Repeatable read | Snapshot | Not possible | Not possible | Allowed, but not in YSQL | Possible |
| Serializable | Serializable | Not possible | Not possible | Not possible | Not possible |

✅ verbatim. The "Allowed, but not in YSQL" cells mean the standard permits the anomaly and the product does not exhibit it. The cost is documented too: Snapshot *"detects only write-write conflicts; it does not detect read-write conflicts"* and *"Applications using this level must be prepared to retry transactions due to serialization failures"* ✅; Serializable *"can detect read-write conflicts in addition to write-write conflicts. This is accomplished by writing provisional records for read operations as well"* ✅ — serializability is bought with extra writes on the read path.

### 6.3 The transaction machinery

| Component | What the vendor says |
| --- | --- |
| Atomicity across tablets | Uncommitted values written as **provisional records**, since ordinary values *"would then become visible at different times to clients reading through different tablet servers"* ✅ |
| Ordering | HLC determines the read point; reads safe up to a hybrid time once a Raft majority has replicated ✅ |
| Concurrency control | MVCC *"without the need to lock rows"* ✅; each transaction works against a version of the data as of a hybrid timestamp |
| Version storage | *"The last part of each key is a timestamp, which enables quick navigation to a particular version of a key in the RocksDB key-value store"* ✅ |
| Single-row writes | Every DocDB update is a transaction, *"including operations that update only one row"* ✅; with autocommit, each statement is one transaction ✅ |
| Conflict behaviour | Depends on **maximum clock skew**, producing *"a higher number of transaction conflicts or a higher latency"* ✅ |

### 6.4 What a reader can rely on

**This guide's analysis:**

1. **Rely on ACID and Raft-majority durability** ✅ — the durability point is majority replication, not local disk.
2. **Do not assume the isolation default without checking version *and* deployment method** ✅ — the Read Committed change landed in v2025.2 and depends on how the cluster was deployed.
3. **Do not assume serializability unless you ask for it** — and if you ask for it you must handle retries ✅.
4. **Do not model conflict rates without modelling clock skew** ✅.

Cross-references: quorum, linearizability and the consistency spectrum are `distributed_systems_engineering_guide.md` §5–§7; the "a quorum is not automatically linearizable" result belongs to this repo's dedicated consistency material and is not re-derived here. **How this differs from the sibling product:** CockroachDB builds around serializable-by-default semantics over its own SQL layer; YugabyteDB exposes three selectable levels ✅ with a default that has changed within the last twelvemonth (v2025.2 ✅). For a house that wants one answer to "what is our isolation level", that matters more than any benchmark.

## 7. Geo-distribution and multi-region

### 7.1 The topology menu

The vendor's own summary ✅ (checked 19 Sep 2026):

| | Default (synchronous) | Geo-partitioning | xCluster | Read replicas |
| --- | --- | --- | --- | --- |
| **Replication** | Synchronous | Synchronous | Asynchronous (uni- and bidirectional) | Asynchronous (unidirectional only) |
| **Data residency** | All data replicated across regions | Partitioned across regions; replicated inside region | All data inside region; per-table cross-region configurable | All data in primary region; cluster-wide async to replicas |
| **Consistency** | Transactional | Transactional | Transactional | **Timeline consistency** |
| **Write latency** | High | Low | Low | N/A |
| **Read latency** | High | Low (when queried from nearby geography) | Low | Low |
| **Schema changes** | Transparently managed | Transparently managed | Transparently managed (with limitations) | Transparently managed |
| **RPO** | No data loss | No data loss (partial unavailability possible) | **Some data loss** | No data loss |

### 7.2 The latency arithmetic of a cross-region quorum

Writes require a Raft **majority** ✅, so write latency is bounded below by the round trip to the majority.

| Deployment shape | Where the majority lives | Conceptual write cost |
| --- | --- | --- |
| 3 replicas, one region, three zones | Same region | One cross-zone round trip — the standard baseline |
| 3 replicas across 3 regions | Two of three regions | A cross-region round trip; the third merely keeps up |
| 5 replicas across 5 regions | Three of five regions | Wider tail, lower probability an outage takes the majority |
| Geo-partitioned, data pinned to a region | Inside that region | Local write latency; cost is *"partial unavailability possible"* ✅ if that region is isolated |

**House analysis:** this is arithmetic doable before design sign-off, not a performance detail to discover later. Put a region 150 ms away and the commit path inherits it. The only escapes are a local quorum (geo-partitioning), not requiring a quorum for the read (follower reads), or accepting async replication and its RPO (xCluster, read replicas) — every one a *guarantee* trade, not a tuning knob. That is the PACELC trade formalised in `distributed_systems_engineering_guide.md` §5.

### 7.3 Read replicas, and what they weaken

Read replicas are *"also known as observer nodes"* that *"do not participate in writes but get a timeline consistent copy of the data in an asynchronous manner"* ✅. *"Read replicas do not add to the write latencies"* ✅; because they do not participate in consensus, *"an odd number of replicas is not required for correctness"* — an RF of 2 is valid ✅. Schema changes propagate transparently, and writes sent to a replica are *"internally redirected to the source of truth"* ✅.

| Guarantee | Primary cluster | Read replica |
| --- | --- | --- |
| Transactional consistency on reads | Yes ✅ | **No** — *"Timeline consistency"* ✅ |
| Write latency impact | Baseline | None added ✅ |
| RPO | No data loss ✅ | No data loss ✅ (reads may lag the truth) |
| Role in consensus | Full Raft member | Not a Raft member ✅ — cannot break or help write availability |

The vendor's defence — *"still strictly better than eventual consistency, because with the latter the application's view of the data can move back and forth in time and is hard to program"* ✅ — is fair **and** a boundary marker. Timeline consistency means *no out-of-order data* ✅; it does **not** mean the read reflects the latest committed write. Read-your-writes paths must not be pointed at a replica without an application-level design.

### 7.4 Follower reads: the precise contract

| Knob | Behaviour |
| --- | --- |
| `yb_read_from_followers` | Enables reading from followers. **Default `false`** ✅ |
| `yb_follower_read_staleness_ms` | Read executes at hybrid time ≈ `now − staleness`. **Default 30000 (30 s)** ✅ |

| Conditions | Expected behaviour |
| --- | --- |
| `yb_read_from_followers` true **AND** transaction read only | Read from the closest replica of the tablet, leader or follower ✅ |
| `yb_read_from_followers` false **OR** statement not read only | Read from the leader ✅ |

Deciding details: **(1)** it applies only to read-only transactions ✅, marked via `SET TRANSACTION READ ONLY`, `SET SESSION CHARACTERISTICS AS TRANSACTION READ ONLY`, or `SET default_transaction_read_only = TRUE` ✅. **(2)** YSQL follower reads are hard-bounded: *"Unlike YCQL follower reads, which support a looser consistency model, YSQL follower reads do not return data newer than this staleness interval, even if available at a local replica, in order to ensure consistency"* ✅ — even *"if the tablet leader is on the closest node"* ✅. **(3)** Do not set staleness below 2× `raft_heartbeat_interval_ms` (default 500 ms) ✅. **(4)** Redirects are observable via `consistent_prefix_failed_reads`, and a shorter staleness means more redirects ✅. **(5)** **Leader leases** underpin the strong-read path — *"an elected node member is guaranteed to be the leader until its lease expires"* ✅ — preventing a stale leader serving reads.

**For a bank:** follower reads are a *contract*, not a switch — they convert a read from "latest" to "at least 30 seconds old, and never out of order". Right for reference data, rate tables and dashboards; wrong for anything a customer acts on against a fresh write.

### 7.5 xCluster and geo-partitioning in one line each

- **xCluster**: asynchronous cross-cluster replication, uni- or bidirectional, per table, documented RPO *"Some data loss"* ✅ — the tool for cross-cluster read-local and for patterns where a regional quorum is unacceptable.
- **Geo-partitioning**: row-level pinning so data is *"partitioned across regions; partitions replicated inside region"* ✅ — residency compliance and low latency, at the cost of *"partial unavailability possible"* ✅. For a bank with residency obligations this is usually the topology that actually satisfies the constraint; synchronous global replication to every region may not be lawful in the first place.

## 8. Licensing and the commercial model

### 8.1 The two licences, from the primary files

Read this pass from the repository's own licence file and the vendor's own legal page ✅ (`github.com/yugabyte/yugabyte-db/blob/master/LICENSE.md`, master branch; `docs.yugabyte.com/stable/legal/`). Checked **19 September 2026**.

| Component tree | Licence |
| --- | --- |
| `src/yb/` — core C++ storage engine: DocDB, consensus, tablets, servers | **Apache License 2.0** ✅ |
| `src/postgres/` — *"Modified PostgreSQL fork for YSQL compatibility"* | **Apache License 2.0** ✅ |
| `src/odyssey/` — connection pooling | **its own BSD-style licence** ✅ |
| `java/`, `python/`, `build-support/`, `bin/`, `cmake_modules/` | Apache License 2.0 ✅ |
| `managed/` — YugabyteDB Anywhere backend (Scala/Java), UI (React), CLI, installers, DevOps tooling | **Polyform Free Trial License 1.0.0** ✅ |
| `troubleshoot/` | **Polyform Free Trial License 1.0.0** ✅ |

The vendor's legal page states the distinction verbatim ✅:

> *"The entire database with all its features (including the enterprise ones) are licensed under the Apache License 2.0."*

> *"The binaries that contain `-managed` in the artifact and help run a managed service are licensed under the Polyform Free Trial License 1.0.0."*

By default, build options generate **only the Apache-2.0 binaries** ✅. Contributions require a contributor licence agreement via cla-assistant.io/yugabyte/yugabyte-db ✅.

### 8.2 The 32-day limit — the sentence procurement needs

> The Polyform Free Trial License 1.0.0 *"permits evaluation use for up to 32 consecutive calendar days. For production use of YugabyteDB Anywhere, a commercial license is required."* ✅

| Question | Answer |
| --- | --- |
| Run the **database** in production under a free licence? | **Yes** — the core, including enterprise features, is Apache-2.0 ✅ |
| Run **YugabyteDB Anywhere** in production under a free licence? | **No** — Polyform Free Trial; commercial licence required ✅ |
| How long may Anywhere be evaluated? | 32 consecutive calendar days ✅ |
| Is there a separate "enterprise edition" of the database to buy? | **No** — this is the correction to a widely repeated claim (§8.4) |
| CLA needed to contribute? | Yes, via cla-assistant.io/yugabyte/yugabyte-db ✅ |

**Practical shape:** self-hosting the database without Anywhere is lawful and free — but the control plane (multi-cluster orchestration, upgrade automation, backup scheduling, fleet monitoring, the UI) is the commercial product. The realistic pattern is *free database, paid management*. That is materially different from an "open core with a paid closed-source edition" model.

### 8.3 The licence change, and its date

| Fact | Detail | Marker |
| --- | --- | --- |
| Announced | **16 July 2019**, by Karthik Ranganathan, *"Founder and co-CEO"*, in *"Why We Changed YugabyteDB Licensing to 100% Open Source"* | ✅ |
| Effective from | the **v1.3** release (15 July 2019) | ✅ |
| What changed | previously closed-source commercial features — **Distributed Backups, Data Encryption, Read Replicas** — folded into the open-source project; the two-edition split removed | ✅ |
| Result, in the vendor's words | *"There is only one edition of YugabyteDB now and that is fully open source"* | ✅ verbatim |
| Management software | released under the Polyform Project's source-available free-trial-only licence | ✅ |
| The contrast drawn | MongoDB (AGPL→SSPL, Oct 2018), Cockroach Labs (Apache 2.0→BSL, June 2019), Confluent (Apache 2.0→CCL, Dec 2018), Elastic (→Elastic License) | ✅ |

The About page restates it in the present tense ✅: *"In July 2019, we went against the industry's shift to proprietary licensing models by making our previously commercial features open source."* **Because that blog post is dated 2019, the current position was re-verified this pass against the live licence file and legal page, not against the blog** ✅.

### 8.4 Correcting a claim that is now partly wrong

The sibling guide's §9.3 row reads, verbatim: **"YugabyteDB → Apache-2.0 core with source-available enterprise ⚠-structural"**. **That is now partly wrong:**

- There is **no source-available enterprise *edition* of the database** — *"the entire database with all its features (including the enterprise ones)"* is Apache-2.0 ✅, and *"there is only one edition of YugabyteDB now"* ✅.
- The Polyform Free Trial licence attaches to **YugabyteDB Anywhere**, the *management platform* ✅.
- Accurate formulation: **"Apache-2.0 database (all features, one edition) + source-available free-trial-only management platform."**

This matters for vendor risk: the thing you cannot self-support for free is the **tooling**, not the **engine**. The engine has no editions to upgrade out of.

### 8.5 Honest positioning against the industry's source-available turn

| Vendor | Trajectory | Consequence |
| --- | --- | --- |
| YugabyteDB | Community/Enterprise → **100% Apache-2.0 core** (16 Jul 2019) ✅ | No feature editions; management platform source-available |
| CockroachDB | Apache 2.0 → **BSL** (June 2019) ✅ | Commercial-use restrictions on newer versions (see `cockroachdb_guide.md`) |
| MongoDB | AGPL → **SSPL** (Oct 2018) ✅ | Service-provider restrictions |
| Confluent | Apache 2.0 → **CCL** (Dec 2018) ✅ | Managed-service restrictions |
| Elastic | → **Elastic License** ✅ | Managed-service restrictions |

**This guide's analysis:** the vendor's self-positioning is factually accurate — it moved opposite to the 2018–2019 industry current, and early rather than retrospectively. Two caveats keep it honest: **(1)** the commercial tension has not disappeared, it has *moved* to the management layer and the managed service, so the frictionless fleet-scale paths sit behind the commercial product and should be budgeted for; **(2)** a permissive licence is not a support contract — Apache-2.0 guarantees the right to run and fork, nothing about response times, CVE timelines, or the upgrade assistance a fork-base rebase will need (§4.3).

## 9. Operations

### 9.1 Deployment topologies

| Topology | When it fits | Marker |
| --- | --- | --- |
| Single-region, multi-zone, RF 3 | The default sane production shape — tolerates one zone loss | ⚠-structural |
| Multi-region synchronous | When *"No data loss"* ✅ and transactional consistency outweigh write latency | ✅ |
| Geo-partitioned | Residency obligations; latency-sensitive regulated workloads | ✅ |
| xCluster | Cross-cluster read-local, DR, migration; accepts *"Some data loss"* RPO ✅ | ✅ |
| Primary + read replicas | Distance reads at *"Timeline consistency"* ✅ | ✅ |
| Managed (Aeon) | Teams that do not want to run a control plane; *"introduced in 2021"* ✅ | ✅ |
| Kubernetes | Standard for cloud-native estates | ⚠-by-pointer |

Because the default build produces **only Apache-2.0 binaries** ✅, an enterprise can run the engine without the commercially licensed tooling — a legitimate architectural choice, but it means writing your own fleet tooling.

### 9.2 The upgrade path, and why a fork changes its shape

| Motion | Frequency | Nature |
| --- | --- | --- |
| Maintenance/patch within a stable series (`YYYY.N.MAINTENANCE.PATCH` ✅) | Maintenance roughly every 1–2 months ✅ | Ordinary rolling upgrade; backward-compatible bug fixes ✅ |
| **YSQL major upgrade** (fork-base rebase) | Rare — once so far in the stable history ✅ | A project: **Early Access** ✅, **DDL frozen** ✅, PITR torn down and rebuilt ✅, `pg_stat_monitor` dropped ✅, drivers upgraded ✅, optional master-side `yugabyte_upgrade` superuser ✅ (§4.3) |

| Series | Type | Released | Maintenance ends | EOL | Status |
| --- | --- | --- | --- | --- | --- |
| v2026.1 | STS | 29 Jun 2026 | 29 Jun 2027 | 29 Dec 2027 | **Current stable** ✅ |
| v2025.2 | LTS | 11 Dec 2025 | 11 Dec 2027 | 11 Jun 2028 | Supported |
| v2025.1 | STS | 23 Jul 2025 | 23 Jul 2026 | 23 Jan 2027 | Maintenance ended |
| v2024.2 | LTS | 9 Dec 2024 | 9 Dec 2026 | 9 Jun 2027 | Supported |

Policy ✅: **STS** series *"receive maintenance updates for at least 1 year (365 days)"* plus *"an additional 180 days"* — *"1.5 years of support… in total"*; **LTS** *"2 years (730 days)"* plus 180 — *"2.5 years… in total"*. *"Yugabyte supports production deployments on stable YugabyteDB releases"* ✅; preview releases are *"not supported for production deployments"* and there is *"no migration path from a preview release to a stable release"* ✅.

**This guide's analysis:** an LTS series gives 2.5 years while a new LTS lands every 12 months, so a bank on LTS must complete a major-version upgrade roughly every 24 months (every ~18 on STS). Layer the once-per-fork-base YSQL major upgrade on top, and you have **two upgrade calendars, one of which is an Early Access project with a DDL freeze**. That is the real operational cost of the fork and it belongs in the budget from day one.

### 9.3 Backup, restore and disaster recovery

| Capability | Position | Marker |
| --- | --- | --- |
| Distributed Backups | In the open-source project — folded in with the v1.3 licence change (16 Jul 2019) ✅ | ✅ |
| Backup/restore documentation | Maintained as a first-class vendor docs section | ✅-by-pointer |
| PITR | Exists; interacts with the YSQL major upgrade (deleted before, recreated after) ✅ | ✅ |
| xCluster as DR | Async, per-table, *"Some data loss"* RPO ✅ | ✅ |
| Third-party BR tooling breadth | **Not verified** this pass | ⚠ |

**House analysis:** the sibling flagged "TiDB/YugabyteDB → BR/backup tooling" ⚠. For YugabyteDB, distributed backup is **not** a commercial add-on — it was named among the features made open source in 2019 ✅ — which resolves the substance of the flag (backups ship with the engine). Ecosystem breadth versus PostgreSQL's remains ⚠. Broader DR design (RTO/RPO framing, HA versus DR, recovery testing) is owned elsewhere in the repo (`ibm_mq_disaster_recovery_guide.md` is the nearest technology-tree sibling).

### 9.4 Observability and operational skills

Verified monitoring surfaces already surfaced in this guide: **`consistent_prefix_failed_reads`** counts follower reads redirected to the leader ✅; **cluster balancing** is continuous with documented monitoring guidance ✅. The metric surface is broad because the tablet/leader/compaction model generates many series — an asset and a capacity-planning problem (⚠-structural). **House analysis:** observability is how you find the *one tablet* out of fifty thousand that is hot, under-replicated or failing to split — budget metric volume and dash tablets, not just "the database". General operations discipline sits in `zero_downtime_system_design_guide.md` and `chaos_engineering_guide.md`.

| Skill | Why needed |
| --- | --- |
| PostgreSQL depth | YSQL is a fork ✅; you operate PostgreSQL semantics against a fork base you must track ✅ |
| Distributed-systems literacy | Raft-per-tablet, quorum latency, clock skew and its effect on conflicts ✅ |
| Tablet-level thinking | Hot tablets, split thresholds, leader distribution, colocation ✅ |
| Topology design | Four topologies with four different guarantees ✅ |
| Capacity/compaction planning | LSM/RocksDB storage behaviour ✅ |

**This guide's analysis:** the skills story is *asymmetric*. Application-side transfer is high — a PostgreSQL-fluent team is productive in days ✅. Operations-side requirements are materially higher than running PostgreSQL, because the failure modes are distributed-systems failure modes. A bank that reads "it's PostgreSQL" and staffs accordingly has mis-read the product.

## 10. Comparisons: resolving the sibling guide's flags

### 10.1 YugabyteDB vs CockroachDB — verified head-to-head

Every row sourced; claims about the other product come from the vendor's own contrast statements or from `cockroachdb_guide.md`, never asserted.

| Dimension | YugabyteDB | CockroachDB | Source |
| --- | --- | --- | --- |
| SQL layer strategy | Fork of the PostgreSQL query layer ✅ | Own SQL layer over own KV store | vendor docs ✅; sibling guide |
| Storage engine | DocDB on customised RocksDB, one instance per tablet ✅ | Own storage layer | vendor docs ✅; sibling §4–§6 |
| Consensus | Raft group per tablet ✅ | Raft-based | vendor docs ✅; `distributed_systems_engineering_guide.md` §6 |
| Unit of sharding/replication | Tablet (shard) ✅ | Range | vendor docs ✅; sibling guide |
| Sharding modes | Hash **and** range natively ✅ | Range-oriented | vendor docs ✅ |
| Timestamp mechanism | HLC ✅; **no TrueTime** ✅ | HLC-class | vendor docs ✅; §3.7 |
| Isolation levels exposed | Three: Serializable, Snapshot, Read Committed ✅ | Serializable-by-default model | vendor docs ✅; sibling guide |
| Default isolation (YSQL) | Snapshot historically; **Read Committed for v2025.2+** via yugabyted/YBA/Aeon ✅ | Serializable by default | vendor docs ✅ |
| Second query API | YCQL (Cassandra CQL), Snapshot isolation only ✅ | None | vendor docs ✅ |
| Major upgrade | *"fully online"* but **DDL blocked** for the duration ✅ | n/a (own SQL layer) | vendor docs ✅ |
| Fork/upgrade coupling | Coupled to the PostgreSQL fork base; YSQL major upgrade Early Access ✅ | Not coupled to a PostgreSQL fork base | vendor docs ✅ |
| Database licence | **Apache License 2.0, all features, one edition** ✅ | Apache 2.0 → BSL (June 2019) | repo `LICENSE.md` + legal page ✅ |
| Management-platform licence | Polyform Free Trial 1.0.0; 32-day evaluation; commercial licence for production ✅ | See sibling guide | repo `LICENSE.md` + legal page ✅ |
| Managed offering | YugabyteDB Aeon (2021, formerly Managed) ✅ | Managed cloud offering | vendor About page ✅ |
| Self-hosted | Yes — Apache-2.0 binaries built by default ✅ | Yes | repo licence ✅ |
| Multi-region menu | Synchronous / geo-partitioning / xCluster / read replicas ✅ | See sibling guide | vendor topology table ✅ |
| Follower reads | Default off; 30 s default staleness ✅ | See sibling guide | vendor docs ✅ |
| CDC | Transactional CDC — *"changes across tables are captured together"* ✅ | Vendor-documented | key-concepts ✅ |
| Ecosystem inheritance | PostgreSQL drivers, extensions, PL/pgSQL ✅ | Own dialect knowledge required | vendor docs ✅; sibling guide |

**Vendor-comparison caution:** the vendor publishes a comparison page and testimonials asserting advantages over CockroachDB (one customer claim of *"3x better performance with fewer resources"* ✅, quoted on the vendor's own About page). **That is a vendor benchmark and a testimonial, not an independent measurement.** No performance claim in this guide rests on either.

### 10.2 vs PostgreSQL

| Dimension | Plain PostgreSQL | YugabyteDB |
| --- | --- | --- |
| Scalability | Vertical, plus read replicas, plus manual sharding/extensions | Horizontal by tablet, RF ≥ 3 ✅ |
| Consistency | Single-node ACID; single-primary replication | Distributed ACID with quorum durability ✅ |
| Failure tolerance | Failover to a standby | Per-tablet Raft; `ft` domain losses at RF `2ft+1` ✅ |
| Vendor coupling | Community + multiple vendors | One principal engine vendor; source is Apache-2.0 ✅ |
| Feature surface | The reference implementation, current version | A **fork of PostgreSQL 15.0** (was 11.2) ✅ — check the base before assuming a feature exists |
| Operational skill | Well understood | App side well understood; ops side distributed ✅ |

**This guide's analysis:** YugabyteDB is *not* "PostgreSQL that scales". It is a different engine that speaks PostgreSQL, with the query layer inherited and storage, replication and transactions replaced ✅. The compatibility dividend is large; the behavioural dividend is partial (isolation defaults, sharding keys, colocation, distributed join costs, fork base).

### 10.3 vs TiDB, managed cloud, and classical sharding

| Alternative | The trade against YugabyteDB |
| --- | --- |
| TiDB | MySQL-compatibility strategy rather than PostgreSQL; HLC-class clock as a *mechanism* parallel, not a guarantee parallel; its backup tooling was not verified this pass ⚠ |
| Spanner-class managed services | TrueTime-class external consistency and no operational burden, in exchange for cloud lock-in — the exact contrast the vendor draws when explaining its own clock ✅ |
| Managed PostgreSQL (RDS-class) | Far simpler and cheaper at single-region scale; loses horizontal write scale and multi-region consistency |
| Classical sharding (see `oracle_sharding_guide.md`) | Retains full vendor feature surface and skills; pays with manual shard management, cross-shard transaction limits, application-level routing |
| MPP/analytical platforms (`data/mpp_databases_guide.md`) | A different job — the MPP-vs-OLTP distinction is owned there |
| Pooled/shared-nothing relational alternatives (`polardb_vs_oceanbase_guide.md`) | Those products' compatibility strategies and guarantees are their own; not conflated here |

**The one decision-relevant line:** if the estate is PostgreSQL-oriented, the fork buys more than MySQL compatibility would; if MySQL-oriented, the reverse. That discriminates more than any architectural parallel.

### 10.4 The sibling guide's flagged rows

The sibling guide's each ⚠/⚠-structural YugabyteDB row is reconciled one-by-one in the claims-audit sub-table at **§14.4** (founding claim; DocDB and Raft-per-tablet architecture; "Postgres-code-wrap"; PostgreSQL wire compatibility; HLC approaches; Raft in DocDB; zone/region placement; CDC; BR/backup tooling; self-hosted plus managed; and the §9.3 licensing row). The short version: **two are resolved with a caveat**, **one is only partly resolved**, and **one needed outright correction** — the licensing row, which stated "Apache-2.0 core with source-available enterprise" when in fact the entire database including its enterprise features is Apache-2.0 with a single edition, and the source-available licence attaches to the Anywhere management platform (§8.4).

### 10.5 Decision framework (this guide's own analysis, explicitly labelled)

| Your situation | Lean toward |
| --- | --- |
| PostgreSQL-centric estate, PL/pgSQL procedures, PostgreSQL-fluent staff, single-region scale-out | **YugabyteDB** — the fork buys maximum inheritance |
| Need a permissively licensed engine with no feature editions | **YugabyteDB** — Apache-2.0 in full ✅ |
| Need serializable-by-default with least configuration ambiguity | Test both against your retry strategy |
| Need Cassandra compatibility **and** SQL in one cluster | **YugabyteDB** — the only one of the two offering YCQL ✅ |
| Need external consistency with no clock assumptions | **Neither** — both HLC-class ✅ (§3.7) |
| Need DDL available during a major engine upgrade | Weight the fork-base DDL freeze ✅ heavily |
| Need a managed service with no self-operated control plane | Compare the managed offerings on terms, not architecture |
| Need maximum operational simplicity on a modest workload | **Neither** — a managed PostgreSQL service is almost certainly cheaper |

## 11. The regulated-enterprise and banking angle

### 11.1 Which banking workloads fit

| Workload | Fit | Reasoning |
| --- | --- | --- |
| Payments/transactional posting with multi-region continuity | Strong | Distributed ACID ✅, per-tablet Raft ✅, geo-partitioning for residency ✅ |
| Core product/account masters with PostgreSQL lineage | Strong | Fork-based PL/pgSQL and type compatibility ✅ |
| Reference and static data services | Strong | Follower reads at bounded staleness ✅ suit this exactly |
| Regulatory reporting / range-heavy analytics | Moderate | Needs range sharding ✅ and designed tablets; heavy analytics belongs on an MPP platform (`data/mpp_databases_guide.md`) |
| Cross-shard aggregates on live OLTP data | Weak | Tablet-spread scans and cross-tablet joins carry distributed cost; keep intraday aggregation off the hot path |
| High table-count microservice schemas | Conditional | Requires colocation ✅; one-table-per-service multiplies Raft groups |

### 11.2 The PostgreSQL-compatibility dividend

| Asset in the existing estate | Transfers? |
| --- | --- |
| PostgreSQL drivers, connection pools, ORMs | Yes — wire protocol ✅ |
| PL/pgSQL and stored procedures | Substantially — the fork carries the procedural machinery ✅, subject to distributed caveats (§4.4) |
| Data model and DDL | Substantially ✅, but sharding key and colocation decisions are new (§3.3) |
| Developer skills | Yes, quickly ✅ |
| DBA operational skills | Partially — everything distributed is new ✅ |
| Backup/restore muscle memory | Partially — distributed backup exists ✅ but behaves differently |
| Upgrade cadence expectations | No — the fork adds a second, heavier calendar ✅ |

**House analysis:** the dividend lands on the *application* side; the cost lands on the *operational* side. That asymmetry is the single most useful sentence for a bank business case, because it says which cost centre to load and which to relieve.

### 11.3 Audit, consistency and controls

| Control question | Position |
| --- | --- |
| Are transactions ACID? | Yes, stated by the vendor for all transactions ✅ |
| Is the isolation guarantee consistent across environments? | **It depends on version and deployment method** ✅ — an audit-relevant hazard (§6.1). A control asserting "we run Read Committed" must specify version *and* how the cluster was deployed |
| Data loss in DR? | Synchronous: *"No data loss"* ✅. xCluster: *"Some data loss"* ✅. Read replicas: *"No data loss"*, reads may lag ✅. The RPO claim must name the topology |
| External consistency for globally ordered ledgers? | **No** — no TrueTime-class clock ✅. Correctness comes from Raft majority plus MVCC; cross-region ordering comes from topology, not a clock |
| Can data be pinned to a jurisdiction? | Yes — geo-partitioning ✅ |
| Is clock synchronisation a control? | **Yes** — skew is named as the driver of conflicts and latency ✅, so NTP/chrony discipline and skew monitoring belong in the control set |

### 11.4 Licence and vendor-risk questions (condensed)

| Question | Answer |
| --- | --- |
| Permissively licensed engine? | Yes — Apache License 2.0, all features, one edition ✅ |
| What is commercially restricted? | YugabyteDB Anywhere: Polyform Free Trial 1.0.0, 32 consecutive calendar days, commercial licence required for production ✅ |
| Escape hatch? | The engine source is Apache-2.0 ✅ — you may fork and self-support the *database*; you may **not** keep running Anywhere in production without a licence ✅ |
| Concentration risk | One principal engine vendor, plus a fork-base dependency on PostgreSQL upstream ✅ |
| What a vendor-management framework should score | Fork-base upgrade responsiveness, CVE timelines, the maintenance/EOL calendar ✅, and whether the commercial control plane is required for your operating model |

Repo cross-references rather than duplication: `../management/vendor_management_guide.md` for vendor scoring and exit planning; `../banking/risk_data_aggregation_guide.md` for the data-side risk framing.

### 11.5 What is NOT yet proven

- **No independent performance or scale figure is established here.** Vendor marketing (§2.3) and testimonials (§10.1) are recorded as claims, never as evidence.
- **No production-scale banking case study is cited** — that would require naming an institution, which this guide deliberately does not do (§12 uses a fictional persona).
- **The completeness of the PostgreSQL feature gap is not fixed here** — that is the vendor's version-specific matrix, which failed to load this pass (§15).
- **The fork-base upgrade at scale is Early Access** ✅ — its behaviour on a large, heavily procedural banking schema is not attested here.

## 12. The Cymbal Bank worked example

**Cymbal Bank is a fictional institution.** Every figure in this section is **illustrative and fictional**. Nothing here is a benchmark, a vendor quote, or a real financial statement.

### 12.1 The estate and the workload assessment

| Attribute | Value (fictional) |
| --- | --- |
| Current platform | 12-year-old sharded relational platform: application-level sharding, four shards, PostgreSQL-derived procedural logic |
| Pain | Manual shard rebalancing; cross-shard transactions avoided by application workarounds; no true active second site; shrinking reporting windows |
| Skills | Strong PostgreSQL/PL-pgSQL; moderate distributed-systems experience |
| Regulatory context | Data residency in two jurisdictions; intraday liquidity reporting; audit requires a defensible isolation statement |
| Candidate under evaluation | YugabyteDB — precisely because the team's skills and stored procedures are PostgreSQL-based |

| Workload | Rows (fictional) | Access pattern | Verdict |
| --- | --- | --- | --- |
| Account master | 80 M | Point lookup + update by account id | Hash sharding candidate ✅-fit |
| Posting/journal | 4 B | Time-ordered writes, range reporting | Range sharding candidate ✅-fit |
| Product/rate reference | 40 K | Read-mostly, wide fan-out | Follower-reads candidate ✅ |
| Case management (300 tiny tables) | small | Point access | **Colocation required** ✅, else Raft-group explosion (§3.2) |
| Intraday regulatory aggregates | n/a | Cross-shard scans on live data | **Keep off the OLTP cluster**; use an MPP platform (`data/mpp_databases_guide.md`) |

### 12.2 The compatibility check, including what does NOT port

| Item | Outcome |
| --- | --- |
| PostgreSQL drivers and ORMs | Port unchanged ✅ |
| PL/pgSQL procedures with local temp-table pipelines | **Do not port cleanly** — rewrite to set-based distributed operations |
| Procedures relying on sequences for global ordering | Re-review — sharding changes the meaning of a monotonic global sequence |
| Extensions (fictional list: text search, encryption, an indexing extension) | Each validated individually against the version-specific matrix — **not assumed** ⚠ (matrix not retrieved this pass) |
| Features requiring PostgreSQL above the fork base | **Blocked until the base moves** ✅ — anything introduced upstream after PG 15 |
| `SELECT … FOR UPDATE` heavy locking patterns | Re-review against MVCC/provisional-record semantics ✅ |
| Cross-shard joins in reporting code | Rewrite or relocate |

### 12.3 Topology, consistency and operations

| Decision | Choice | Why |
| --- | --- | --- |
| Base topology | Three availability zones, RF 3 | Tolerate one zone loss (`2ft+1`, ft=1) ✅ |
| Second jurisdiction | **Geo-partitioned** data for residency-constrained products | Residency ✅ and local write latency ✅ |
| Reference-data reads | **Follower reads enabled selectively** | Bounded staleness, timeline-consistent ✅; wrong for anything a customer acts on |
| Isolation | Pin explicitly with `SET TRANSACTION`; enable Read Committed where the application has no retry logic ✅ | Avoids the "the default is what I think it is" hazard (§6.1) |
| DR | xCluster to a secondary cluster, accepting *"Some data loss"* RPO ✅ | Cheaper than a synchronous second-region quorum |
| Clock | Chrony with monitored skew | Skew drives conflicts and latency ✅ |

| Area | Impact (fictional but reasoned) |
| --- | --- |
| Application teams | ~2–4 weeks to productivity — PostgreSQL familiarity pays immediately |
| Platform/DB team | Substantially new: tablet/leader reasoning, splits and balancing, follower-read semantics, geo-partition ops, fork-base upgrade planning |
| Runbooks | New: hot-tablet triage, follower-read redirect triage (`consistent_prefix_failed_reads` ✅), quorum loss, region isolation |
| Upgrade planning | Two calendars ✅, one Early Access with a DDL freeze ✅ |
| Tooling | Decide explicitly: license Anywhere, or build fleet tooling on the Apache-2.0 binaries ✅ |

### 12.4 Licence and support questions

| Question | Cymbal's answer |
| --- | --- |
| Run the engine without buying anything? | Yes — Apache License 2.0, all features, one edition ✅ |
| Need Anywhere? | Only for the vendor control plane; note the 32-day evaluation limit and the production commercial-licence requirement ✅ |
| Still need a support contract? | Yes — the licence grants rights, not response times (§8.5); price fork-base upgrade assistance specifically |

### 12.5 Migration sequencing and risks

1. **Build the compatibility inventory** — every procedure, extension, sequence and dependency, classified port / rewrite / retire.
2. **Prototype the hardest schema first** (the 300-small-table domain) to force the colocation decision early ✅.
3. **Stand up a three-zone RF-3 cluster** and validate isolation behaviour explicitly, including the version/deployment-method matrix ✅.
4. **Move the reference-data service to follower reads first** — lowest risk, immediate latency evidence ✅.
5. **Migrate the account master to hash sharding**, posting/journal to range sharding ✅.
6. **Bring up geo-partitioning** for residency-constrained products; test region isolation, accepting *"partial unavailability possible"* ✅.
7. **Layer DR last** (xCluster), once steady state is understood.

| Risk | Guardrail |
| --- | --- |
| Hot tablet from a monotonic posting key | Range sharding ✅ plus pre-splitting and per-tablet monitoring |
| Raft-group explosion from table sprawl | Colocation ✅, reviewed at schema gate |
| Applications silently running the wrong isolation level | Pin isolation explicitly; assert it in CI ✅ |
| Fork-base upgrade arriving as an unplanned project | Put the rebase in the 24-month plan from day one ✅ |
| Deferred workload testing | Use the repo's chaos and deterministic-testing discipline (`chaos_engineering_guide.md`, `zero_downtime_system_design_guide.md`) |

### 12.6 Cost comparison — ILLUSTRATIVE AND FICTIONAL

**All figures are fabricated for reasoning only.** They are not vendor pricing, not quoted from any source, and must not be cited.

| Cost line | Current sharded platform | YB option A (self-hosted engine, own tooling) | YB option B (engine + licensed Anywhere) |
| --- | --- | --- | --- |
| Hardware/infrastructure | 100 (index) | 110 | 110 |
| Engine licence | 100 | 0 — Apache-2.0 ✅ | 0 — Apache-2.0 ✅ |
| Management platform licence | 0 (in-house) | 0 (build in-house) | 60 (illustrative) |
| Internal platform engineering | 40 | 90 | 45 |
| Application remediation | 0 | 70 (one-off) | 70 (one-off) |
| Rehearsal/migration project | 0 | 25 (one-off) | 25 (one-off) |
| Skills uplift | 10 | 30 | 30 |
| Support contract | 30 | 35 | 35 |
| **Indicative 3-year total** | **~280** | **~360** | **~375** |

**Reading (this guide's analysis):** the option with the lowest *licence* cost is not the option with the lowest *total* cost. Option A avoids the platform fee and pays it back in engineering; Option B pays the fee and buys back engineer time. Either way, the one-off remediation line dwarfs the licence delta.

### 12.7 Recommendation, and what the team would give up

**Recommendation (this guide's analysis): a scoped pilot, not an estate-wide mandate.** The pilot should cover one residency-constrained product domain (forcing geo-partitioning), one small-table domain (forcing colocation), and one reference-data service (forcing the follower-read contract). Success criteria should be *isolation behaviour asserted explicitly*, *no hot tablets under the modelled posting rate*, and *a written fork-base upgrade plan* — not throughput numbers alone.

**What Cymbal Bank would give up:** (1) the freedom to pick the PostgreSQL version — semantics come from the fork base (15.0, formerly 11.2) ✅; (2) a single unambiguous isolation default — it depends on version and deployment method ✅; (3) DDL availability during major engine upgrades ✅; (4) simplicity of the operating model — distributed skills become mandatory; (5) a vendor-free operating model unless it self-builds tooling ✅; (6) any pretence of Spanner-equivalent external consistency ✅.

**What it gains:** per-tablet horizontal write scale, distributed ACID with quorum durability ✅, a residency-compliant topology ✅, a genuinely permissive engine licence ✅, and the ability to keep its PostgreSQL skills, PL/pgSQL code and driver estate largely intact while changing the substrate underneath.

## 13. Gotchas and anti-patterns

Format: **symptom → cause → guardrail**.

| # | Symptom | Cause | Guardrail |
| --- | --- | --- | --- |
| 1 | "It's PostgreSQL, so our feature works" — then it doesn't | Assuming compatibility means the *full* feature surface; the vendor's own hedges are *"most PostgreSQL features"* and *"a large portion of the documentation"* ✅ | Audit **per feature** against the vendor's version-specific matrix before design sign-off, and re-audit on every fork-base move (§4.4) |
| 2 | An upstream PostgreSQL feature is missing locally | Treating the fork's version as equal to upstream's — base was **11.2**, now **15.0** ✅ | Pin the fork base in architecture standards; anything newer than the base is a *no* until it moves (§4.2) |
| 3 | A migrated Cassandra workload returns wrong results | Treating YCQL as Cassandra semantics: **Snapshot isolation only** ✅, transactions **opt-in per table** ✅, *"looser consistency model"* ✅ | Re-run consistency and retry analysis per migrated table; verify `WITH transactions = {'enabled': true}` where atomicity is assumed (§5) |
| 4 | A "latency-optimised" topology still misses the budget | Topology chosen without quorum arithmetic; writes need a Raft **majority** ✅ | Compute the quorum round trip at design time (§7.2); if it fails, change the topology, not a parameter |
| 5 | The cross-region write that quietly did not perform | Synchronous multi-region replication on a latency-critical commit path; nobody modelled the second region's RTT | Model it; consider local quorums, and be honest that xCluster/read replicas trade *"Some data loss"* or timeline consistency ✅ for latency (§7) |
| 6 | The licence understood only from a comparison blog | Acting on second-hand summaries — including the sibling guide's own corrected §9.3 row | Read `LICENSE.md` and the legal page directly (§8.1); the distinctions that matter are invisible in blog summaries |
| 7 | The upgrade deferred until it became a project | Fork-base rebases are rare, **Early Access** ✅, and freeze **DDL** ✅ — deferral turns a window into a programme | Put the rebase in a rolling 24-month plan from day one; keep a standing compatibility inventory (§9.2, §12.5) |
| 8 | An operation reported complete but the value was overwritten | Client-side auto-retry on timeout, which *"can break linearizability… from the client point of view"* ✅ | Use the no-retry-on-timeout driver policy and handle timeouts in the application ✅ (§5) |
| 9 | Tens of thousands of Raft groups; a struggling cluster | Schema sprawl — one table per microservice, each getting its own consensus group | Colocate small tables ✅; make colocation a schema-gate item (§3.2, §12.1) |
| 10 | Reads that miss a write the same user just made | Pointing read-your-writes paths at follower reads or read replicas; YSQL follower reads *"do not return data newer than this staleness interval"* ✅ | Classify reads: reference/telemetry → followers; anything a customer acts on against a fresh write → leader (§7.3, §7.4) |
| 11 | Serialization errors appearing after a platform change | Snapshot resolving underneath an application written for Read Committed, with no retry logic | Enable Read Committed explicitly ✅ and ensure retry handling exists before raising isolation (§6.1) |
| 12 | Conflicts and latency rising with no workload change | Clock skew — the vendor names *"the maximum clock skew in the cluster"* as the driver ✅ | Treat NTP/chrony discipline and skew monitoring as production controls (§3.7, §11.3) |

**Testing technique cross-reference:** the discipline for deliberately breaking a distributed system to surface these failure modes is in `chaos_engineering_guide.md`, with availability framing in `zero_downtime_system_design_guide.md`.

## 14. Claims audit

Legend: **Verified** = read from a primary source this pass; **Flagged** = recorded but not independently confirmed; **Rejected** = asserted elsewhere but shown to be wrong or unsupportable, and therefore not carried.

### 14.1 Verified claims (highest confidence)

| # | Claim | Source | Source date | Marker |
| --- | --- | --- | --- | --- |
| 1 | Founded 2016 by Muthukkaruppan, Ranganathan and Bautin; three former Facebook engineers with Cassandra/HBase lineage | yugabyte.com/about | checked 19 Sep 2026 | ✅ |
| 2 | Database core under **Apache License 2.0**, *"all its features (including the enterprise ones)"*, one edition | repo `LICENSE.md` + stable/legal | checked 19 Sep 2026 | ✅ |
| 3 | **Polyform Free Trial License 1.0.0** covers `managed/` and `troubleshoot/`; evaluation up to **32 consecutive calendar days**; commercial licence for YBA production | repo `LICENSE.md` + legal page | checked 19 Sep 2026 | ✅ |
| 4 | 100% Apache-2.0 core announced **16 July 2019**; effective with **v1.3**; folded in Distributed Backups, Data Encryption, Read Replicas | vendor blog | 16 Jul 2019 | ✅ |
| 5 | YSQL is a **fork of the PostgreSQL query layer**; `src/postgres/` = *"Modified PostgreSQL fork for YSQL compatibility"* | vendor docs + repo `LICENSE.md` | checked 19 Sep 2026 | ✅ |
| 6 | Fork base **PostgreSQL 11.2 → 15.0**; boundary: prior to v2.25 = PG 11; v2.25+ preview / v2025.1+ stable = PG 15 | docs major-upgrade page + README | checked 19 Sep 2026 | ✅ |
| 7 | YSQL major upgrade is **Early Access**, fully online but **DDL blocked**; requires v2024.2.3.0+, driver upgrade, PITR delete/recreate, `pg_stat_monitor` drop/recreate, `yugabyte_upgrade` superuser, precheck command | docs major-upgrade page | checked 19 Sep 2026 | ✅ |
| 8 | Three YSQL isolation levels (Serializable, Snapshot, Read Committed); YCQL **Snapshot only** | vendor docs | checked 19 Sep 2026 | ✅ |
| 9 | YSQL default **effectively Snapshot** historically; **Read Committed by default for v2025.2+** via yugabyted/YBA/Aeon; `yb_enable_read_committed_isolation` defaults false | docs isolation-levels | checked 19 Sep 2026 | ✅ |
| 10 | DocDB built on a customised RocksDB, **one RocksDB instance per tablet**; LSM/SST; C++ | docs architecture/docdb | checked 19 Sep 2026 | ✅ |
| 11 | Deterministic row→tablet mapping; hash and range native; automatic tablet splitting; cluster balancing of data and leaders | docs docdb-sharding | checked 19 Sep 2026 | ✅ |
| 12 | **One Raft group per tablet**, elected leader; fault tolerance `ft` needs `2ft+1` domains | docs key-concepts | checked 19 Sep 2026 | ✅ |
| 13 | **No TrueTime**: HLC = (physical, logical), strictly monotonic; skew affects conflict resolution and latency | docs transactions-overview | checked 19 Sep 2026 | ✅ |
| 14 | Four multi-region topologies with documented consistency/RPO: synchronous (No data loss); geo-partitioning (No data loss, partial unavailability possible); xCluster (**Some data loss**); read replicas (**timeline consistency**) | docs multi-region-deployments | checked 19 Sep 2026 | ✅ |
| 15 | Follower reads: `yb_read_from_followers` default **false**; `yb_follower_read_staleness_ms` default **30000**; read-only transactions only; never newer than the staleness interval; 2× heartbeat floor; `consistent_prefix_failed_reads` | docs follower-reads-ysql | checked 19 Sep 2026 | ✅ |
| 16 | Read replicas are observer nodes, do not vote in Raft, add no write latency, may use even RF, receive schema changes transparently, redirect writes to source | docs read-replicas | checked 19 Sep 2026 | ✅ |
| 17 | Release calendar: v2026.1 STS (29 Jun 2026, EOL 29 Dec 2027); v2025.2 LTS (11 Dec 2025, EOL 11 Jun 2028); v2025.1 STS (23 Jul 2025); v2024.2 LTS (9 Dec 2024) | docs releases page | checked 19 Sep 2026 | ✅ |
| 18 | Support policy: STS ≥ 365 days + 180; LTS ≥ 730 days + 180; preview unsupported for production; no preview→stable migration path | docs releases/versioning | checked 19 Sep 2026 | ✅ |
| 19 | Aeon (formerly YugabyteDB Managed) introduced 2021 | yugabyte.com/about | checked 19 Sep 2026 | ✅ |
| 20 | Transactional CDC — *"changes across tables are captured together"* | docs key-concepts | checked 19 Sep 2026 | ✅ |

### 14.2 Flagged claims (recorded, not relied upon)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | Vendor marketing scale figures: *"1M+ transactions per second"*, 10K GitHub stars, 100+ countries, 11,000+ community members | Vendor marketing with no methodology published on the cited page (§2.3); recorded, never used as evidence |
| 2 | Testimonial claiming *"3x better performance with fewer resources"* than a competitor | Vendor-published testimonial, not an independent benchmark (§10.1) |
| 3 | Total capital raised, per-round funding history, valuation | **Not established** — funding aggregators are secondary and were not reachable with confidence (§2.3, §15) |
| 4 | The `max_clock_skew_usec` flag name and default | The existence of a skew bound is implied by the vendor's own caveat ✅, but the flag name/default were not re-read this pass (§3.7, §15) |
| 5 | Third-party backup/BR tooling breadth | Not verified this pass (§9.3) |
| 6 | The vendor's PostgreSQL-compatibility feature matrix | Page failed to load this pass; no claim here rests on its contents (§4.4, §15) |
| 7 | The sibling's "Kudu"-related naming history | Not confirmed on the vendor's own pages; not restated (§2.2) |
| 8 | Current patch version within the 2026.1 series | Secondary aggregator reporting; series-level facts came from the vendor's own releases page (§15) |

### 14.3 Rejected / corrected claims

| # | Claim as commonly seen | Verdict |
| --- | --- | --- |
| 1 | "YugabyteDB offers an **enterprise edition** of the database under a source-available licence" — including the sibling's §9.3 row *"Apache-2.0 core with source-available enterprise ⚠-structural"* | **REJECTED / CORRECTED.** No enterprise *edition* of the database exists: *"the entire database with all its features (including the enterprise ones) are licensed under the Apache License 2.0"* ✅, *"there is only one edition of YugabyteDB now and that is fully open source"* ✅. Polyform Free Trial attaches to **YugabyteDB Anywhere**, the management platform ✅ (§8.4) |
| 2 | "YugabyteDB has Spanner-style TrueTime / externally consistent timestamps" | **REJECTED.** The vendor states a TrueTime-class clock *"is not available in many deployments"* and that HLC is used instead ✅ (§3.7) |
| 3 | "It implements PostgreSQL 15, so it matches PostgreSQL 15 semantics in full" | **REJECTED as stated.** The **fork base** is 15.0 ✅, but storage, transactions and sharding are replaced; compatibility is feature-by-feature, not wholesale (§4.2, §4.4) |
| 4 | Any specific throughput, latency or scale number attributed to this guide | **NOT OFFERED.** No independent benchmark was performed or found this pass; vendor numbers are labelled as vendor numbers (§2.3, §10.1, §11.5) |

### 14.4 The sibling guide's flagged rows — sub-table

| Sibling flag | Row | Disposition |
| --- | --- | --- |
| ⚠-structural | Founding / founders claim | **RESOLVED** — founded 2016, named founders, Cassandra/HBase lineage ✅ |
| ⚠-structural, widely documented | DocDB + Raft-per-tablet architecture | **RESOLVED** — DocDB on customised RocksDB, one instance per tablet ✅; Raft group per tablet ✅ |
| ⚠-structural | Postgres-code-wrap claim | **RESOLVED** — genuine fork of the PostgreSQL query layer ✅ |
| ⚠ | Wire compatibility "PostgreSQL (via Postgres core)" | **RESOLVED WITH CAVEAT** — wire protocol ✅, but against the fork base (15.0, formerly 11.2) ✅ |
| ⚠-structural | "TiDB/YugabyteDB → similar HLC approaches" | **RESOLVED, NARROWED** — true as mechanism ✅; not a guarantee claim; no TrueTime ✅ |
| ⚠-structural | "YugabyteDB → Raft (DocDB)" | **RESOLVED** — per-tablet Raft leader election ✅; `2ft+1` ✅ |
| ⚠ | "YugabyteDB → zone/region placement" | **RESOLVED** — fault domains at node/zone/region level ✅; geo-partitioning ✅ |
| ⚠ | "YugabyteDB → CDC" | **RESOLVED** — transactional CDC ✅ |
| ⚠ | "TiDB/YugabyteDB → BR/backup tooling" | **PARTLY RESOLVED** — distributed backups are in the open-source project ✅; third-party ecosystem ⚠ |
| ⚠ | "YugabyteDB → self-hosted + managed" | **RESOLVED** — both exist ✅ |
| ⚠-structural | §9.3 licensing row "Apache-2.0 core with source-available enterprise" | **CORRECTED** — Apache-2.0 database in full, one edition ✅; Polyform Free Trial applies to Anywhere ✅ |

## 15. What Could Not Be Verified

Stated plainly, because a guide that reports only its successes is not an audit trail.

1. **Funding and ownership trajectory** — no total-raised figure, per-round history or cap table from a primary source; the investor list and board were read ✅ but the *amounts* were not. ⚠
2. **Confirmed absence of any acquisition, merger or restructuring** — the trail shows **none** as of 19 September 2026, but this is a *negative finding from the sources checked*, not proof of absence; a private company can transact without a website announcement. ⚠
3. **The vendor's PostgreSQL-compatibility feature matrix** — the page failed to load this pass (scrape failure, not a confirmed 404), so **no feature-by-feature compatibility claim is made here**; §12.2 is explicitly marked as requiring that matrix to be read. ⚠
4. **The `max_clock_skew_usec` flag name and default** — the vendor's caveat confirms skew matters ✅, but the configuration reference was not re-read, so the specific flag name/default is **not quoted as fact**. ⚠
5. **Independent performance or scale evidence** — none located or run; every performance-flavoured number in circulation is vendor marketing (§2.3) or a vendor-published testimonial (§10.1). ⚠
6. **Third-party backup/restore tooling breadth** versus the PostgreSQL ecosystem. ⚠
7. **YCQL consistency semantics beyond the isolation level** — Snapshot-only ✅ and the *"looser consistency model"* for YCQL follower reads ✅ were verified; the full per-statement consistency surface was not. ⚠
8. **The exact current patch release within the 2026.1 series** — secondary aggregator reporting, unconfirmed against the vendor's own page. ⚠
9. **The "Kudu"-related naming-history claim** carried in the sibling guide — not confirmed, not restated. ⚠
10. **YugabyteDB Anywhere's own release-notes series and support windows** as distinct from the database's — the vendor maintains a separate YBA releases page ✅-by-pointer, but its contents were not read this pass. ⚠

## 16. Glossary

| Term | Definition |
| --- | --- |
| **Aeon (YugabyteDB Aeon)** | The vendor's managed DBaaS, introduced 2021 as *YugabyteDB Managed* and renamed ✅ |
| **Apache License 2.0** | The permissive licence covering the database core — all features, one edition ✅ |
| **Anywhere (YugabyteDB Anywhere, YBA)** | The self-managed management and orchestration platform; Polyform Free Trial 1.0.0 ✅ |
| **CDC** | Change Data Capture; transactional, with changes across tables captured together ✅ |
| **Colocated table** | A small table inside a shared tablet, so it does not get its own Raft group ✅ |
| **DocDB** | The storage engine beneath everything, on a customised RocksDB, one instance per tablet ✅ |
| **Fault domain** | A potential point of failure: node, rack, zone or region ✅ |
| **Follower read** | A read served from a replica rather than the leader, bounded by configured staleness; YSQL default 30 s, off by default ✅ |
| **Geo-partitioning** | Row-level pinning of data to a region for residency or latency ✅ |
| **Hybrid logical clock (HLC)** | Timestamp mechanism combining a physical clock component with a monotonic logical counter; **not** TrueTime ✅ |
| **Leader lease** | A guarantee that an elected tablet leader stays leader until its lease expires, preventing stale-leader reads ✅ |
| **LSM tree / SST** | Log-structured merge tree storage with sorted string tables; the model beneath DocDB ✅ |
| **MVCC** | Multi-Version Concurrency Control; concurrent access without row locking ✅ |
| **Polyform Free Trial License 1.0.0** | Source-available licence covering Anywhere; 32 consecutive calendar days of evaluation, commercial licence required for production ✅ |
| **Provisional record** | The representation of an uncommitted transactional value, so a partially applied transaction is never visible as ordinary data ✅ |
| **Raft group** | The consensus group replicating one tablet; one per tablet, with an elected leader ✅ |
| **Read replica (observer node)** | An async, read-only copy of a universe; does not vote in Raft, adds no write latency, gives timeline consistency ✅ |
| **Replication factor (RF)** | Copies of each tablet; fault tolerance `ft` needs `2ft+1` domains ✅ |
| **Sharding** | Deterministic mapping of a row to a tablet; hash or range ✅ |
| **Snapshot isolation** | The stricter non-serializable level; detects write-write conflicts only, not read-write ✅ |
| **STS / LTS** | Release trains: the `.1` release per year is standard-term support, the `.2` is long-term support ✅ |
| **Tablet** | The unit of sharding and replication — its own Raft group and its own RocksDB instance ✅ |
| **TrueTime** | Google's bounded-uncertainty clock used by Cloud Spanner; **not** available in YugabyteDB deployments ✅ |
| **Universe / TP-EA-GA / xCluster / YB-Master / YB-TServer** | Respectively: a deployed instance (primary cluster plus read replicas) ✅; feature maturity tags — Tech Preview, Early Access, General Availability ✅; async cross-cluster per-table replication with RPO *"Some data loss"* ✅; cluster metadata, DDL coordination and balancing ✅; the tablet server that hosts tablets, serves queries and participates in Raft ✅ |
| **YCQL** | The Cassandra-Query-Language-compatible API; Snapshot isolation only ✅ |
| **YSQL** | The PostgreSQL-compatible API, built on a fork of the PostgreSQL query layer ✅ |
| **YSQL major upgrade** | The procedure that rebases the PostgreSQL fork (11 → 15); Early Access, online, DDL blocked ✅ |

## 17. Cross-references and further reading

### 17.1 Sibling guides in this repository

| Guide | Why you want it |
| --- | --- |
| `cockroachdb_guide.md` | Owns the distributed-SQL/NewSQL taxonomy and the CockroachDB deep dive; its §9.3 and §9.5 YugabyteDB rows are resolved by §10.4 here |
| `distributed_systems_engineering_guide.md` | Owns the theory — consistency spectrum (§5), consensus (§6), replication and partitioning (§7), PACELC, quorum arithmetic. Cited throughout, never duplicated |
| `nosql_data_modelling_guide.md` | Owns data modelling, including the wide-column model relevant to YCQL |
| `chaos_engineering_guide.md` | Deliberately breaking a distributed system — the testing technique referenced in §13 |
| `zero_downtime_system_design_guide.md` | Availability and change-without-downtime framing for §9 |
| `oracle_database_guide.md`, `oracle_sharding_guide.md` | The incumbent-relational and classical-sharding comparison points in §10.3 |
| `polardb_vs_oceanbase_guide.md` | Alternative distributed-relational strategies and their own compatibility trades |
| `data/mpp_databases_guide.md` | The MPP-vs-OLTP distinction; where heavy analytical workloads belong (§12.1) |
| `schema_evolution_data_drift_guide.md` | Schema-change discipline under a distributed DDL freeze (§4.3) |
| `ibm_mq_disaster_recovery_guide.md` | DR framing adjacent to §9.3 |
| `../management/vendor_management_guide.md` | Vendor-risk scoring and exit planning (§11.4) |
| `../management/it_strategy_guide.md` | IT strategy framing for a platform-replacement business case |
| `../banking/risk_data_aggregation_guide.md` | Data-side risk aggregation framing (§11.4) |
| `../banking/end_to_end_banking_processes.md`, `../banking/policy_administration_systems_guide.md` | Domain context for the banking workloads in §11.1 |
| `../banking/bian_cash_management_domains_guide.md`, `../banking/treasury_alm_guide.md` | Cash-management and treasury domain context where OLTP-class workloads live |

**Note on a referenced-but-absent guide:** a dedicated per-database consistency-knob guide (`tunable_consistency_databases_guide.md`) is referenced in this repo's material but **is not present in the tree as of 19 September 2026** (checked). That material currently lives in `distributed_systems_engineering_guide.md` §5; this guide's §6 covers YugabyteDB's own knobs only.

### 17.2 Primary sources used this pass (all checked 19 September 2026 unless dated otherwise)

- `github.com/yugabyte/yugabyte-db/blob/master/LICENSE.md` — per-component licence map (Apache-2.0 trees vs Polyform trees). Repo README — v2025.1 as the first stable PG 15 fork base.
- `docs.yugabyte.com/stable/legal/` — two-licence statement; 32-day evaluation limit; CLA.
- `docs.yugabyte.com/stable/architecture/`, `…/architecture/docdb/`, `…/architecture/key-concepts/`, `…/architecture/docdb-sharding/`, `…/architecture/transactions/transactions-overview/`, `…/architecture/docdb-replication/read-replicas/` — the layer split; DocDB on a customised RocksDB, one instance per tablet; tablets, fault domains, RF, follower reads, universe, isolation levels, YB-Master, CDC; hash/range sharding, tablet splitting, balancing, colocation; HLC, TrueTime absence, the skew caveat, MVCC, provisional records; read-replica mechanics and guarantees.
- `docs.yugabyte.com/stable/explore/transactions/isolation-levels/`, `…/explore/multi-region-deployments/`, `…/explore/going-beyond-sql/follower-reads-ysql/`, `…/explore/ysql-language-features/`, `…/develop/learn/transactions/acid-transactions-ycql/` — isolation levels, defaults, anomaly matrix and the Read Committed change; the four-topology comparison table; follower-read flags, defaults and caveats; the PostgreSQL-fork statement and supported feature families; YCQL Snapshot-only, per-table transactions and the retry/linearizability caveat.
- `docs.yugabyte.com/stable/releases/versioning/`, `…/releases/ybdb-releases/`, `…/manage/ysql-major-upgrade-yugabyted/` — versioning model, support policy, feature maturity, per-series dates; the fork-base upgrade procedure and constraints.
- `yugabyte.com/about/` — findings, founders, history, licence-change summary, Aeon, leadership, investors, vendor claims. `yugabyte.com/news/` — 2026 operating signals. `yugabyte.com/blog/why-we-changed-yugabyte-db-licensing-to-100-open-source/` — the 16 July 2019 licence change (dated 2019).

### 17.3 Suggested further reading (primary, not read this pass)

- `docs.yugabyte.com/stable/explore/ysql-language-features/postgresql-compatibility/` — the definitive per-feature compatibility matrix; **required reading before any migration assessment** (§15).
- `docs.yugabyte.com/stable/reference/configuration/yb-tserver/` — T-server configuration reference, including the clock-skew bound (§15).
- `docs.yugabyte.com/stable/releases/yba-releases/` — YugabyteDB Anywhere releases and support windows.
- `docs.yugabyte.com/stable/additional-features/change-data-capture/`, `…/manage/backup-restore/`, `…/explore/observability/` — CDC, backup/restore, and metrics detail.
- `yugabyte.com/compare-products/` — the vendor's own comparison material (vendor benchmark; label accordingly).

## 18. Closing summary

YugabyteDB is the distributed SQL database that made one decisive bet and has lived with its consequences ever since: **keep PostgreSQL rather than reimplement it.** The parser, planner, type system, catalog, wire protocol and PL/pgSQL machinery are a genuine fork — the repository's own licence file describes `src/postgres/` as a *"Modified PostgreSQL fork for YSQL compatibility"* ✅ — and everything underneath is replaced: DocDB on a customised RocksDB, one instance per tablet ✅; a Raft group per tablet with an elected leader and a `2ft+1` replication-factor rule ✅; hash and range sharding, automatic tablet splitting and continuous cluster balancing ✅; MVCC over hybrid logical clocks with provisional records for uncommitted values ✅.

What that buys is unusually concrete: a PostgreSQL-fluent team is productive immediately, drivers and procedures largely carry over, and the engine sits under the Apache License 2.0 in full — one edition, all features, no enterprise edition to upgrade into ✅. What it costs is equally concrete, and it is not what the marketing leads with. The SQL semantics you inherit are the fork base's, not upstream's — 15.0 today, 11.2 until v2025.1 ✅ — so feature availability is a matrix to be read rather than a version number to be assumed. The isolation default is not one answer but several, depending on both version and deployment method ✅. Upgrading means two calendars, one of which is an Early Access procedure that freezes DDL for its duration ✅. Writes cost a quorum round trip wherever the quorum lives ✅, and there is no TrueTime-class clock to make cross-region ordering free ✅.

For a regulated enterprise the honest summary is this: the compatibility dividend lands on the application side and the operating burden lands on the platform side. The database earns its place when an estate is PostgreSQL-shaped, needs horizontal write scale, needs a residency-compliant topology, and is prepared to staff for distributed-systems operations and to plan a fork-base rebase as a project rather than a patch. It is the wrong answer when the requirement is Spanner-grade external consistency, when the workload is analytical, or when a managed service on a single-node technology would do the job at a fraction of the effort.

The sibling guide's flagged rows are now resolved or corrected (§10.4, §14.4) — including the licensing row, which needed outright correction because the thing under a source-available licence is the management platform, not the database. What remains genuinely open is the feature-by-feature compatibility matrix, the funding history and every performance claim, all flagged rather than filled in (§15). A reader making a decision should treat those three gaps as the assessment work and take the rest of this guide as the map. The product's whole strategic bet, and the source of both its greatest appeal and its sharpest operational edge, is that what you are really running is a distributed database with the PostgreSQL inside.

