# Alibaba PolarDB vs Alibaba OceanBase: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Data Architecture / Database Engineering — the head-to-head comparison of Alibaba Cloud PolarDB (cloud-native, shared-storage) and Ant Group OceanBase (native distributed, shared-nothing): architectures, compatibility, ecosystems, deployments, performance, and the selection framework (Database / Data-Engineering series)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The Two Databases at a Glance](#1-the-two-databases-at-a-glance)
2. [PolarDB in Depth](#2-polardb-in-depth)
3. [OceanBase in Depth](#3-oceanbase-in-depth)
4. [Head-to-Head Comparison](#4-head-to-head-comparison)
5. [Use Cases](#5-use-cases)
6. [Selection Framework](#6-selection-framework)
7. [Market and Ecosystem](#7-market-and-ecosystem)
8. [Worked Example: A Bank Evaluating Both for a New Core-Banking Platform](#8-worked-example-a-bank-evaluating-both-for-a-new-core-banking-platform)
9. [The Future: 2026 and Beyond](#9-the-future-2026-and-beyond)
10. [Glossary](#10-glossary)
11. [References and Verification Notes](#11-references-and-verification-notes)

---

## 1. The Two Databases at a Glance

### 1.1 PolarDB in One Paragraph

**Alibaba Cloud PolarDB** is Alibaba Cloud's flagship **cloud-native relational database**: a MySQL- and PostgreSQL-compatible engine built on a **compute-storage separation (shared-storage) architecture** in which a cluster of stateless compute nodes — one read-write primary plus up to 15 read replicas — attach to a single distributed storage pool called **PolarStore**. Launched in 2017 as Alibaba Cloud's answer to AWS Aurora, PolarDB is "the cloud database": it exists to make relational databases elastic, serverless, and operationally trivial *on the Alibaba Cloud*, with storage replication, backup, and failover pushed down into the storage layer so that compute can scale in seconds. The family also includes **PolarDB-X** (the distributed, shared-nothing scale-out edition — the former DRDS) and the Oracle-compatible **PolarDB-O**, making PolarDB a spectrum from "a single database that behaves like one MySQL" to "a sharded cluster that behaves like many."

### 1.2 OceanBase in One Paragraph

**OceanBase** is the **distributed relational database** built inside the Alibaba ecosystem (originally at Taobao/Alibaba, today owned by **Ant Group**'s subsidiary **OceanBase Inc.**) and born from the hardest OLTP workload on earth: Alipay's payment core and the **Double 11 (双11, Singles' Day)** transaction spikes. It is a **shared-nothing** system: data is partitioned across peer **OBServer** nodes, every partition is replicated with a **Multi-Paxos** consensus protocol (RPO = 0, failover in seconds), and cross-node transactions are handled with two-phase commit — giving it both **write scale-out** (partition-level leaders mean many nodes can accept writes concurrently) and strong consistency. OceanBase holds two TPC-C world records (60.88M tpmC in 2019, 707M tpmC in 2020 — the first Chinese database to top the benchmark), is MySQL- and Oracle-compatible, was open-sourced in 2021 under the Mulan license, and now runs 400+ customer deployments globally, with Chinese bank core systems — including ICBC — as its marquee references.

### 1.3 The Alibaba Family Tree

Both databases come from the same family, and the family history explains why they are different:

| Era | Event | Consequence |
|---|---|---|
| 2009–2010 | Alibaba's e-commerce core hits the Oracle + IBM + EMC ceiling; the **去IOE** ("de-IOE") movement begins inside Alibaba | The mandate to replace foreign stack with self-built, commodity-x86 technology |
| 2010 | **Yang Zhenkun (阳振坤)** joins Taobao and starts OceanBase to replace Oracle for Taobao's shopping cart | OceanBase's origin as an internal project (see [chinese_bank_core_systems_guide.md §5.1](../banking/chinese_bank_core_systems_guide.md)) |
| 2011–2014 | OceanBase 0.x/1.x enters Taobao/Tmall production; **Alipay** adopts it for the payment core | OceanBase becomes the transaction engine of Alibaba's financial arm |
| 2014 | Alipay restructures into **Ant Group**; OceanBase moves with it | OceanBase's corporate home becomes Ant, not Alibaba Cloud |
| 2017 | **Alibaba Cloud launches PolarDB** as its cloud-native relational database | PolarDB's origin as a *cloud* product, architected for the cloud era |
| 2020 | **Ant Group splits from Alibaba** (the aborted Ant IPO; Alibaba and Ant formally separate their shareholding and business ties) | OceanBase becomes an Ant subsidiary, increasingly independent from Alibaba Cloud |
| 2021 | **OceanBase Inc.** is incorporated as an Ant Group company; OceanBase 3.x is open-sourced | OceanBase becomes an independent commercial company and open-source project |
| 2025 | **PolarDB retakes the TPC-C world record** (2.055B tpmC, "Limitless" multi-master mode) | The family rivalry is now also a benchmark rivalry |

The shared heritage is real: both are products of **Alibaba tech DNA** — the engineering culture forged by Taobao/Tmall scale, Double 11, and 去IOE. The split is also real: since 2020–2021 they are corporate cousins (Alibaba Cloud vs Ant Group) that increasingly compete head-to-head for the same China and international database deals.

### 1.4 Positioning in One Line Each

- **PolarDB** = the cloud-native **shared-storage** relational database for the cloud era: "one logical MySQL/PostgreSQL, elastically scaled, managed by Alibaba Cloud."
- **OceanBase** = the **shared-nothing** native-distributed relational database born for extreme OLTP: "many commodity servers acting as one strongly-consistent, horizontally write-scalable database."

A useful shorthand: **PolarDB scales reads (and, via PolarDB-X/Limitless, writes) by adding compute on shared storage; OceanBase scales writes by partitioning data across peer nodes.** One is an Aurora-style architecture; the other is a NewSQL/Google Spanner-style architecture. Section 4 explores the trade-offs.

### 1.5 The Double 11 Crucible

Both products were forged in the same fire: **Double 11 (双11, Singles' Day, 11 November)** — the world's largest online shopping event, and the single most brutal OLTP stress test in commercial history. The numbers that matter:

- Alibaba's Tmall/Taobao handle **hundreds of millions of concurrent shoppers** and peak order/payment rates in the **hundreds of thousands of TPS** during the event window (Alipay's widely reported peak reached ~610,000 TPS in 2021; per-year figures vary by source).
- **PolarDB-X** was created specifically to scale the **Tmall Double 11 core transaction system** — its sharding design is the direct descendant of that requirement.
- **OceanBase** carried the **Alipay payment core** through Double 11 for a decade — the 707M tpmC TPC-C record was, in part, the public proof of what the internal event had already demonstrated.
- When Alibaba Cloud promoted PolarDB's 2025 TPC-C record, it framed the 2.055B tpmC result as **~59× the peak Tmall 11.11 transaction volume of 2020** — the benchmark, in Alibaba's telling, is a stress test of the stress test.

For architects evaluating either product, the Double 11 heritage is the single most important credibility signal: these are not lab databases. They have survived the highest sustained OLTP load ever observed in production — with money on the line (see [chinese_bank_core_systems_guide.md §1.3](../banking/chinese_bank_core_systems_guide.md) for the banking-side framing of peak-season loads).

### 1.6 Why This Comparison Matters

For a solutions architect, this is the rare head-to-head where both candidates are credible for the same workload class (ACID OLTP) yet differ on every architectural axis: storage (shared vs shared-nothing), replication (storage-side log replay vs consensus), compatibility surface (MySQL/PG/Oracle vs MySQL/Oracle), deployment (cloud-first vs cloud + on-prem + open source), and corporate home (Alibaba Cloud vs Ant Group). The choice drives years of data-centre, licensing, and skills decisions — especially in banking, where the China context (信创/xinchuang, 去IOE) makes the domestic-database question strategic (see [chinese_bank_core_systems_guide.md §7.1](../banking/chinese_bank_core_systems_guide.md) and §7.5). This guide is the dedicated deep-dive; the relational baseline it competes against is covered in [oracle_database_guide.md](oracle_database_guide.md).

---

## 2. PolarDB in Depth

### 2.1 Overview and History

- **Launch:** PolarDB was announced and launched by Alibaba Cloud in **September 2017** (at the Apsara conference), positioned explicitly as the cloud-native alternative to AWS Aurora — "the database architected for the cloud." Its design is documented in academic papers (e.g., the PolarDB Serverless / cloud-native VLDB publications), which describe it as a shared-storage cloud-native database with a single primary handling read/write and secondaries for reads.
- **Versioning:** PolarDB does not use a "PolarDB 1.0/2.0" public product versioning. The engines track their upstream compatibility targets: **PolarDB for MySQL** offers engine versions 8.0.2, 8.0.1 (stable LTS, fully compatible with MySQL Community Edition 8.0.13 and earlier), 5.7 and 5.6; **PolarDB for PostgreSQL** tracks PostgreSQL (14/15/16-era compatibility in current releases). The "2.0" the task-brief era sometimes cites refers to Alibaba Cloud's marketing of the second-generation PolarDB (the 2021-era PolarDB 2.0 announcements around PolarDB-X) — **do not rely on "PolarDB 2.0" as a version number; check the engine's MySQL/PG compatibility version instead.**
- **Positioning:** compute-storage separation, seconds-level elastic scaling, multi-AZ high availability, and "100% MySQL / 100% PostgreSQL compatibility" are the marketing pillars; PolarDB is the flagship of the Alibaba Cloud database portfolio (with RDS, AnalyticDB, Lindorm alongside it) and is marketed internationally on alibabacloud.com, not just in China.

### 2.2 The PolarDB Family

| Edition | Engine basis | What it is | Status notes |
|---|---|---|---|
| **PolarDB for MySQL** | MySQL 8.0 / 5.7 / 5.6-compatible | The flagship: shared-storage cluster (1 RW primary + up to 15 read nodes), HTAP, serverless | GA on Alibaba Cloud; the main international product |
| **PolarDB for PostgreSQL** | PostgreSQL-compatible | Same shared-storage architecture on the PG engine; strong for Oracle-style workloads migrating to open-source SQL | GA |
| **PolarDB-O (PolarDB for Oracle)** | Oracle-compatible (PG-based engine with Oracle compatibility features) | Migration target for Oracle estates on Alibaba Cloud (mainly the China region) | Status to verify on aliyun.com — Alibaba Cloud's strategic emphasis since ~2023 has shifted to the MySQL/PG engines and PolarDB-X; check current availability before planning an Oracle migration on it |
| **PolarDB-X** | MySQL-compatible, distributed | The scale-out edition: **the former DRDS** (Distributed Relational Database Service), re-branded PolarDB-X ("PolarDB for Xscale") in 2021 and open-sourced (Apache-2.0 SQL layer + polardbx-engine, a MySQL fork); shared-nothing, compute/storage decoupled, 2PC distributed transactions, Multi-Paxos on data nodes | GA; strong for sharding workloads; see §2.5 |
| **PolarDB (Apsara Stack) / PolarDB Stack** | MySQL/PG | The private-cloud edition delivered via Alibaba Cloud's **Apsara Stack** for on-premises/专有云 deployments | Exists for enterprise/信创 private clouds (mainly China); verify current SKUs |

### 2.3 Architecture: Compute-Storage Separation on PolarStore

PolarDB's architecture is the "shared-storage" model made famous by Aurora, taken to an Alibaba-specific extreme:

```
        Application (one logical database endpoint)
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   RW Primary         RO Replica 1      RO Replica 15      ← stateless compute (SQL layer)
   (writes)            (reads)            (reads)             (up to 15 read nodes)
        │                 │                 │
        └────────┬────────┴────────┬────────┘
                 ▼                 ▼
        ┌──────────────────────────────────────┐
        │   PolarStore (shared distributed     │
        │   storage pool: triple-replicated,   │
        │   SSD-based, log-structured)         │
        └──────────────────────────────────────┘
```

- **Compute layer:** stateless database nodes (the SQL/transaction engine). One node is the **read-write primary**; the rest are **read replicas**. Alibaba Cloud documents the topology as "write once, read many": **1 RW primary + 1 to 15 read nodes** — the cluster is read-scale-out by adding compute nodes on demand, in seconds (this corrects the "15/16 replicas" figure sometimes quoted: the documented maximum is 15 read nodes).
- **Storage layer (PolarStore):** a distributed storage system with **triple replication** (three copies across availability zones by default), very high throughput and low latency to the compute layer, and **"log-as-the-database"** operation: the primary ships its **redo log** to storage, and **the storage layer applies the log and materialises pages** (storage-side replication). Compute nodes do not share buffer pools via cache-coherence traffic the way RAC does — replicas read committed pages directly from PolarStore. PolarStore also performs **compression and compaction** at the storage tier (a 2025 ArXiv paper describes PolarStore's dual-layer compression for large-scale cloud-native databases).
- **What the application sees:** one database endpoint. Read replicas are exposed via a load-balanced endpoint; there is no sharding, no re-partitioning, no cross-node transaction problem — the app keeps its single-node mental model while the cloud handles the rest.

### 2.4 Read Scale-Out, Failover, and Elasticity

- **Read scale-out:** read replicas (up to 15) can be added in seconds; replica lag is typically in the low milliseconds because replicas consume the same redo stream the storage layer already applies (some materials cite sub-10ms typical lag).
- **Failover:** on primary failure, Alibaba Cloud promotes a replica (or rebuilds the primary against PolarStore); because the redo log lives in shared storage, there is **no data loss on failover (RPO = 0 within the storage replication domain)** and the recovery time is in the **seconds-to-tens-of-seconds** class (vendor-stated; exact RTO depends on detection and promotion configuration — verify against the current SLAs).
- **Elasticity:** compute can scale up/down without data migration; storage scales automatically. **PolarDB Serverless** (available for both MySQL and PostgreSQL engines) scales cluster nodes elastically within seconds to handle workload surges, charging per PCU (capacity unit) and scaling to zero/sleep during idle periods — Alibaba Cloud's answer to Aurora Serverless.
- **Multi-master ("Limitless"):** since ~2023 Alibaba Cloud has shipped a **multi-master scale-out mode for PolarDB for MySQL (the "Limitless" cluster)** that lets multiple RW nodes write concurrently by combining the shared-storage engine with distributed extensions (sharded tables across multiple masters). This is the mode in which PolarDB set the 2025 TPC-C record on 2,340 read/write nodes (§4.4). It effectively blurs the line between the "single logical DB" PolarDB and the distributed PolarDB-X.

### 2.5 Compatibility: MySQL, PostgreSQL, Oracle, and the Distributed PolarDB-X

- **MySQL-compatible:** PolarDB for MySQL targets near-100% MySQL compatibility (the engine tracks MySQL 8.0; the LTS 8.0.1 line is fully compatible with MySQL Community 8.0.13 and earlier). For most MySQL applications, migration is a connection-string change plus minor parameter tuning. This is the default entry point for the huge MySQL ecosystem (Spring/JDBC, ORMs, binlog-based CDC, DataX, etc.).
- **PostgreSQL-compatible:** PolarDB for PostgreSQL offers the same shared-storage architecture on a PG-compatible engine — the choice for PG estates and for teams that want Aurora-PostgreSQL-style behaviour on Alibaba Cloud.
- **Oracle-compatible (PolarDB-O):** the Oracle-compatibility edition targets Oracle migrations (PL/SQL, Oracle data types, and SQL dialect differences). **Verify its current availability and roadmap** on aliyun.com — as of 2026 the Oracle-migration story is more prominently carried by PolarDB PostgreSQL's compatibility features and by OceanBase's Oracle mode (§3.7) than by a separately marketed PolarDB-O.
- **PolarDB-X (distributed):** PolarDB-X is the distributed member of the family — **the successor of DRDS**. It is a **shared-nothing** distributed SQL database: stateless **CN (Compute Node)** for parsing/optimisation/routing/2PC coordination, **DN (Data Node)** for persistence with Multi-Paxos strong consistency, and a **GMS (Global Meta Service)** for metadata. It supports hash/range sharding, distributed transactions, global secondary indexes, HTAP, cross-data-centre deployment, and Table Group/geo-locality optimisations; it is highly MySQL-compatible (protocol, most SQL syntax, isolation levels, binlog). PolarDB-X was originally built for **Tmall's Double 11 core transaction system** and is open source (community version on GitHub). **Relationship to PolarDB:** PolarDB-X is the "scale-out MySQL" option of the family — if a workload outgrows a single PolarDB cluster's write capacity, PolarDB-X (or PolarDB Limitless multi-master) is the next step; the two share tooling and the MySQL compatibility surface but are architecturally distinct (shared-storage vs shared-nothing).

### 2.6 Features: HTAP, Serverless, AI

- **HTAP:** PolarDB for MySQL provides an **in-memory column index (IMCI)** for analytical acceleration on the same row store — writes continue on the row format while a columnar index serves OLAP queries; **elastic Parallel Query (ePQ)** pushes parallel execution across nodes for analytical SQL. The PostgreSQL engine adds parallel-query and columnar features over time.
- **Serverless:** see §2.4 — seconds-level elastic scaling and pay-per-PCU on both engines.
- **AI:** Alibaba Cloud's database line is being repositioned around AI-era features — PolarDB integrates with the **Alibaba Cloud AI ecosystem** (vector support on the PG engine, integration with Qwen/Model Studio for natural-language database interaction, AI-optimised autoscaling/self-tuning, and the **PolarDB agentic tooling** published on GitHub by Alibaba Cloud). Treat the "AI database" feature set as fast-moving marketing + genuinely shipping pieces (vector types, AI assistant, AI-driven O&M); evaluate against your concrete workload.
- **Operational features:** transparent data encryption (TDE), SSL, audit, backups/PITR at storage level, binlog for CDC, Data Transmission Service (DTS) for migrations, and integration with the full Alibaba Cloud observability stack.

### 2.7 Deployment: Cloud and On-Premises

- **Public cloud (the main story):** PolarDB is a managed service on **Alibaba Cloud (aliyun.com / alibabacloud.com)** — international regions included, which is the primary way non-China customers consume it.
- **Serverless:** available on the public cloud (§2.4).
- **Private/on-premises:** Alibaba Cloud offers its **Apsara Stack** private-cloud line, which includes **PolarDB Stack** for on-premises/专有云 deployments — primarily sold in China for 信创 and regulated-industry estates. Availability, versions and licensing outside China are limited: **PolarDB's centre of gravity is the public cloud.** For an on-premises-first bank, PolarDB is the weaker of the two candidates on this axis; OceanBase (§3.8) was designed for the opposite bias.
- **Licensing:** PolarDB is a **proprietary managed service** (no public open-source core; PolarDB-X is the open-source exception). You consume it as a cloud SKU — which means pricing transparency, but also cloud lock-in and no self-managed option for the main engines.

### 2.8 PolarDB Timeline and Adoption

| Year | Milestone |
|---|---|
| 2017 | PolarDB launched (Apsara conference, September) as the Aurora-class cloud-native database |
| 2019 | PolarDB 2.0-era features: HTAP/IMCI columnar acceleration, parallel query; TPC-C results begin appearing for the MySQL engine |
| 2020–2021 | **PolarDB-X rebranded from DRDS** (PolarDB for Xscale); PolarDB-X open-sourced (October 2021, community edition) |
| 2021 | PolarDB Serverless announced; PolarDB positions as the "fourth-generation" database (compute-storage separation + HTAP + distributed) |
| 2023–2024 | **Multi-master "Limitless"** mode for PolarDB for MySQL; AI-era features (vector, natural-language database, agentic tooling) |
| 2025 | **TPC-C world record: 2.055B tpmC on 2,340 read/write nodes (Limitless), $0.11/tpmC** — surpassing OceanBase's 707M tpmC record |

**Adoption footprint:** PolarDB is the default relational database for new workloads on Alibaba Cloud (China and international regions), powering large internet/SaaS estates; Alibaba's own Double 11 systems run PolarDB-family engines. Its banking footprint is thinner than OceanBase's — PolarDB appears in Chinese banks mainly as the cloud/MySQL workload database and via Apsara Stack deployments, not as the core-ledger engine (the core-ledger slot belongs to the shared-nothing crowd — OceanBase, GaussDB, GoldenDB; see [chinese_bank_core_systems_guide.md §7.1](../banking/chinese_bank_core_systems_guide.md)).

---

## 3. OceanBase in Depth

### 3.1 Overview and History

OceanBase is the distributed relational database that began inside Taobao in 2010 and is now the flagship of Ant Group's tech ecosystem. Timeline:

| Year | Milestone |
|---|---|
| 2010 | **Yang Zhenkun (阳振坤)** — a Peking University computer-science professor-turned-industry researcher (he worked at Lenovo and Microsoft Research Asia before joining Taobao in 2010) — starts OceanBase at Taobao to replace Oracle for the **shopping-cart system**; the first OceanBase 0.x prototypes target exactly that workload |
| 2011–2012 | OceanBase 0.x deployed for Taobao's shopping cart; the project moves into Alipay |
| 2012–2016 | **OceanBase 1.x** in Taobao/Tmall production (transaction, order, and account domains); Alipay's core payment systems migrate |
| 2017 | **OceanBase 2.0**: a near-complete rewrite into the modern architecture — native distributed, Multi-Paxos, LSM-tree, shared-nothing (§3.3) |
| 2019 (Oct) | TPC-C world record #1: **60.88M tpmC** — the first Chinese database ever to top TPC-C, beating the previous (Oracle) record |
| 2020 (May) | TPC-C world record #2: **707M tpmC (7.07亿)** at ¥3.98/tpmC — 11.6× the 2019 result (the task-brief's "2021 7.07亿 tpmC" is a one-year mis-dating; the record was published **20 May 2020**, documented in the VLDB 2022 paper "OceanBase: A 707 Million tpmC Distributed Relational Database System") |
| 2020 | Ant Group splits from Alibaba (aborted IPO); OceanBase's corporate path separates from Alibaba Cloud |
| 2021 | **OceanBase 3.x** and the **Community Edition** open-sourced under the **Mulan Public License 2.0** on GitHub and Gitee; **OceanBase Inc.** incorporated as an Ant Group company |
| 2022 (Aug) | **OceanBase 4.0 ("Xiaoyu" 小鱼)** — the "**single-machine distributed**" (一体化/单机分布式) generation: a distributed database that runs efficiently even on a single machine, radically lowering the entry threshold; multi-tenant resource isolation, cloud-native K8s operator, cost optimisation |
| 2023–2026 | **4.x series** (4.2 → 4.3 → 4.4+): HTAP (columnar store), OB Cloud (public cloud), international expansion; a **5.x line** has been signalled in vendor/community materials — **not confirmed in official release notes as of August 2026; verify** before citing a 5.0 GA |

### 3.2 The OceanBase Company

- **Corporate structure:** OceanBase is run by **OceanBase Inc.** (北京奥星贝斯科技有限公司), a subsidiary of **Ant Group** (which itself separated from Alibaba Group in 2020). The company is headquartered in Beijing, with engineering also in Hangzhou (Ant's home) and a growing international organisation.
- **Leadership:** CEO **Yang Bing (杨冰)** (previously Ant's CTO of the tech platform line) — publicly active internationally (e.g., ATxSummit in Singapore).
- **Funding/valuation:** OceanBase was funded internally by Alibaba/Ant for its first decade. Per SCMP reporting (late 2023), OceanBase was **seeking its first external equity financing** as part of Ant's spin-off-driven "independent growth" strategy; no confirmed Series A valuation or round size is verifiable as of August 2026 — **treat any specific valuation figure as unverified**.
- **IPO:** no confirmed IPO plans as of August 2026 (Ant Group's own IPO remains frozen since the November 2020 suspension). The eventual Ant IPO is the most plausible catalyst; watch OceanBase's funding announcements for a concrete timeline.

### 3.3 Architecture: Shared-Nothing with Paxos

OceanBase's architecture is the opposite end of the spectrum from PolarDB:

```
   App ──► SQL routing (OBProxy / OB Cloud endpoint)
                  │
   ┌──────────────┼──────────────┐
   ▼              ▼              ▼
OBServer 1    OBServer 2    OBServer 3        ← peer nodes (shared-nothing)
 partition A   partition B   partition A'      (each node owns its data slice)
 leader        leader        follower
   ▲              ▲              ▲
   └── Multi-Paxos log replication (RPO = 0) ──┘
        (each partition's Paxos group spans nodes/AZs)
```

- **Shared-nothing:** the database is horizontally partitioned into **tablets/partitions** spread across **OBServer** nodes; each node owns its data on local storage — no shared disk, no shared storage pool. Scale-out = add servers; data rebalances automatically (partition migration), and the system supports **hundreds of nodes per cluster** (Alipay's estate runs tens of thousands of OBServers across clusters — per the VLDB 707M-tpmC paper, "tens of thousands of OceanBase servers have been deployed in Alipay").
- **Multi-Paxos replication:** every partition's log is replicated to a Paxos group (typically 3 replicas, by default across AZs/sites in production). Writes commit when a quorum (2 of 3) acknowledges. This gives **RPO = 0** (a committed transaction's log exists on ≥2 nodes before the client is told it committed) and automatic failover in seconds; vendor-stated RTO is **under 30 seconds** (some materials cite ~8s for single-region failures) — verify per configuration.
- **Multi-writer by design:** each partition has a **leader** that accepts writes for its slice; because there are many partitions, **many nodes accept writes concurrently** — write scale-out is native (unlike PolarDB's single-primary topology, where write capacity is capped at one node unless Limitless/multi-master is engaged). Reads can be served by followers (with consistency options from strong to session/eventual) for read scale-out.
- **Single-machine distributed (4.x):** the 4.0-generation architecture removed the "distributed tax": a 4.x cluster can run on **one server** (or a laptop for development) and scale to hundreds of nodes without a switch of product — the "一体化" (integration of distributed and stand-alone) positioning that makes it viable for mid-tier banks and enterprises, not just giants.

### 3.4 Storage: LSM-Tree

- OceanBase's storage engine is an **LSM-tree** design: writes go to an in-memory **memtable**; when the memtable fills it is flushed to immutable, sorted **SSTables** on disk, and background **compaction** merges SSTables. Consequences: **writes are sequential append-only** (no in-place page updates, no random writes, no double-write), which is why OceanBase can sustain extreme write throughput on commodity hardware — the key to its TPC-C results and to Alipay's payment loads.
- The LSM design is paired with **block-level compression and encoding** (multi-layer compression; OceanBase 4.x added real-time/advanced compression options) to cut storage cost, and with a **columnar store (HTAP)** since 4.3 for analytical scans on the same data.
- Read amplification and compaction storms are the classic LSM trade-offs; OceanBase mitigates them with tiered compaction, adaptive compression, and the memtable/SSTable balance — operational tuning differs materially from a B-tree engine like PolarDB's (PolarStore pages) or Oracle's (see [oracle_database_guide.md §2](oracle_database_guide.md)).

### 3.5 Distributed Transactions and Consistency

- **Strong consistency:** OceanBase transactions are **ACID with strong consistency across nodes**. A transaction touching multiple partitions runs **two-phase commit (2PC)** coordinated by the transaction's home/coordinator node, with each participating partition's log protected by Paxos. The result: the application sees a single strongly-consistent database, not an eventually-consistent sharded mess — the property that lets banks run account postings and transfers across shards safely.
- **Isolation:** supports Read Committed and Repeatable Read (and Oracle-mode semantics such as snapshot/read-only); MVCC via multi-version storage.
- **Failover semantics:** RPO = 0; RTO seconds (see §3.3); **geo-distributed deployment** (Paxos groups across AZs/regions) supports multi-AZ and even multi-region active-active configurations — the same 单元化 (cell-based) pattern the Alibaba family pioneered (see [chinese_bank_core_systems_guide.md §5.3](../banking/chinese_bank_core_systems_guide.md)).

### 3.6 Performance: The TPC-C Records

| Record | Date | Result | Note |
|---|---|---|---|
| TPC-C #1 | Oct 2019 | **60.88M tpmC** | First Chinese database to top TPC-C; first to beat Oracle's long-held record (Oracle had held the top spot since 2011) |
| TPC-C #2 | 20 May 2020 | **707M tpmC (7.07亿)** | 11.6× improvement; ¥3.98/tpmC; documented in the VLDB 2022 paper; the benchmark also demonstrated fault tolerance (a node was killed during the run and throughput recovered within ~2 minutes) |
| (Crown lost) | 2025 | PolarDB 2.055B tpmC | OceanBase's record was surpassed by its cousin PolarDB in "Limitless" multi-master mode (§4.4) — the family rivalry now owns the top of the TPC-C leaderboard |

Benchmark caveats worth stating: TPC-C results are tuned, scale-dependent, and published under strict audit, but they measure *capacity* (tpmC) on a *specific* configuration — they do not tell you what you will get at your transaction mix on your hardware. Use them as existence proofs of scale, not as sizing tables.

### 3.7 Compatibility: MySQL Mode and Oracle Mode

- **MySQL mode:** OceanBase's primary compatibility surface — MySQL protocol, most MySQL SQL syntax, transaction isolation levels, JDBC/ODBC drivers, and ecosystem tools. Most MySQL applications migrate with modest SQL rewriting. The community edition is MySQL-mode-first.
- **Oracle mode:** OceanBase offers an **Oracle-compatibility mode** (PL/SQL, Oracle data types, packages, sequences, hints, and much of the Oracle dialect) specifically to absorb **Oracle migrations — the 去IOE heart of the China banking story** (see [chinese_bank_core_systems_guide.md §5.1](../banking/chinese_bank_core_systems_guide.md)). This is OceanBase's wedge into Chinese banks replacing Oracle core systems: the same application code, ported with migration tooling (OMS — OceanBase Migration Service), onto a domestic distributed database.
- **Not PostgreSQL:** OceanBase deliberately does not offer a PG-compatibility mode (PolarDB PG and GaussDB own that niche in China). Its compatibility story is "MySQL and Oracle," nothing else.

### 3.8 Deployment: On-Prem, Cloud, Open Source, International

- **Ant production (the reference):** OceanBase runs **all of Alipay's mission-critical systems** — the payment core, accounts, settlement. Double 11 (双11) is the stress test: peak payment-processing rates during the Singles' Day sales events are widely reported at **~610,000 TPS (2021)** with several hundred thousand TPS sustained over the peak window; the exact per-year numbers vary by source — treat as "widely reported" rather than audited.
- **Banks:** OceanBase's banking references include **ICBC** (adoption announced September 2020, verified), plus Bank of Nanjing, Zheshang Bank, Changshu Rural Commercial Bank, Hongta Bank and others (per OceanBase materials and the landscape table in [chinese_bank_core_systems_guide.md §7.1](../banking/chinese_bank_core_systems_guide.md)); **MYbank (网商银行)** — Ant's digital bank — runs its core on OceanBase, including the **310 model** (3-minute application, 1-second approval, 0 human intervention) for SME lending (see [chinese_bank_core_systems_guide.md §6.2](../banking/chinese_bank_core_systems_guide.md)). The company states **400+ customers globally**.
- **On-premises/private:** OceanBase is fully deployable **on-premises** (the bank pattern): bare metal or VMs, x86/ARM (Kunpeng etc.), K8s operator, and it is a first-class citizen in **信创/xinchuang** certified stacks (domestic chips, Kylin/UOS OS, domestic middleware — see [chinese_bank_core_systems_guide.md §7.3–7.5](../banking/chinese_bank_core_systems_guide.md)). For Chinese banks, on-prem + 信创 is the default requirement, and OceanBase is one of the two or three default answers.
- **Cloud (OB Cloud):** **OceanBase Cloud (OB Cloud)** is the managed SaaS: available on **Alibaba Cloud**, **AWS** (Marketplace listing since October 2022), **Google Cloud** (Marketplace), and other platforms; the company claims **60+ regions / 240+ AZs** of OB Cloud coverage (company statement — verify per platform). It supports the 4.x line.
- **Open source:** the **OceanBase Community Edition** (MySQL mode) is on GitHub/Gitee under the **Mulan Public License 2.0** — fully self-manageable, deployable anywhere, with the commercial/Enterprise edition adding Oracle mode, advanced O&M, and support. This makes OceanBase the only one of the two with a genuine open-source, self-managed path.
- **International:** OceanBase is the more internationally ambitious of the two: a global support centre opened in Malaysia (April 2026), the "GO GLOBAL GO" international program (October 2025), Gartner Peer Insights recognition, and a Singapore-based leadership presence (CEO appearances at ATxSummit). Markets targeted: Southeast Asia, Japan, the Middle East, and other Oracle-heavy regions where the Oracle-compatibility story lands.

### 3.9 Operations and Tooling

Operating OceanBase differs from operating a single-node engine — the tooling matters as much as the engine:

| Tool | Role |
|---|---|
| **OBProxy** | The SQL routing/proxy layer: connection management, load balancing, routing to partition leaders; the app's entry point |
| **OCP (OceanBase Cloud Platform)** | The O&M control plane: cluster lifecycle, upgrades, monitoring, alerts, backup/restore orchestration |
| **OMS (OceanBase Migration Service)** | Schema + data migration from MySQL/Oracle (and other sources) with minimal downtime — the tool that makes Oracle-mode migrations tractable |
| **ODC (OceanBase Developer Center)** | Developer/DB tooling: SQL workspace, PL/SQL debugging, schema management, data comparison |
| **OBClient / drivers** | MySQL-protocol CLI; JDBC/ODBC/Go/Python drivers; binlog-compatible CDC for stream integration |
| **K8s operator / Ansible** | Containerised and scripted deployment for private clouds and 信创 stacks |
| **Community ecosystem** | 20+ supported third-party tools (monitoring, BI, backup) plus GitHub/Gitee community builds |

For a bank DBA team coming from Oracle, the learning curve is real but bounded: SQL and PL/SQL knowledge transfers (Oracle mode), while the distributed concepts (partitions, Paxos groups, compaction, 2PC) are new. OceanBase's certification tracks (OCA/OCP/OCM) and its Singapore/SEA presence shorten that curve for regional teams.

### 3.10 Editions Compared: Community vs Enterprise

| Capability | **Community Edition** | **Enterprise Edition** |
|---|---|---|
| License | Mulan Public License 2.0 (open source, free) | Commercial (per-node/per-core licensing) |
| Compatibility modes | MySQL mode | MySQL mode + **Oracle mode** |
| Deployment | Self-managed anywhere (bare metal/VM/K8s); no vendor support | Self-managed + vendor support (OCP enterprise, global support centres) |
| OB Cloud | — | Yes (managed SaaS on Aliyun/AWS/GCP) |
| Enterprise features | Core engine (LSM, Paxos, 2PC, HTAP basics), 20+ third-party tools | Oracle mode, advanced O&M tooling, security/compliance packages, migration services (OMS enterprise), SLAs |
| Typical use | Development, mid-tier OLTP, open-source-first enterprises, escape-hatch/escrow | Bank cores, regulated workloads, supported production |

The pragmatic reading: **the Community Edition is the try-before-buy and the escrow** (the full engine source is available even for Enterprise customers' risk mitigation), while **Enterprise + Oracle mode is the bank-core product**. For a bank evaluating OceanBase, run the POC on Community, contract Enterprise for production — the architecture is identical.

---

## 4. Head-to-Head Comparison

### 4.1 The Comparison Table

| Dimension | **PolarDB** | **OceanBase** |
|---|---|---|
| **Architecture** | Shared-storage (compute-storage separation); Aurora-style | Shared-nothing (native distributed); Spanner/NewSQL-style |
| **Origin** | Alibaba Cloud, 2017 (cloud-native product) | Taobao/Alipay internal project, 2010; Ant Group company since 2021 |
| **Compatibility** | MySQL + PostgreSQL (+ PolarDB-O Oracle-compatible edition; verify current status) | MySQL mode + Oracle mode (no PG mode) |
| **Storage** | PolarStore distributed storage, triple-replicated, log-applied pages | Local storage per node, LSM-tree (memtable + SSTables + compaction) |
| **Replication** | Redo-log shipping to shared storage (storage-side replication) | Multi-Paxos consensus per partition (RPO = 0) |
| **Scale** | 1 RW primary + up to 15 read nodes; PolarDB-X/Limitless for write scale-out (2,340 nodes in the 2025 TPC-C run) | Hundreds of nodes per cluster; partition-level write scale-out; tens of thousands of servers across Alipay |
| **Transactions** | Single-node transactions on the primary; replicas read-only (multi-master via Limitless) | Distributed 2PC + Paxos; strong consistency across nodes natively |
| **Consistency** | Strong within the cluster (single logical DB) | Strong across all nodes (ACID at any scale) |
| **Deployment** | Aliyun public cloud (incl. international); PolarDB Stack on Apsara Stack for private cloud | Cloud (OB Cloud on Aliyun/AWS/GCP) + on-prem + open-source self-managed |
| **Licensing** | Proprietary managed service (PolarDB-X is the open-source exception) | Proprietary enterprise + open-source Community Edition (Mulan 2.0) |
| **Best fit** | Cloud-native OLTP, read-heavy, elastic; MySQL/PG teams on Alibaba Cloud | Financial-grade distributed OLTP, extreme write scale, Oracle migrations, on-prem/信创 |
| **Target** | Alibaba Cloud customers; internet/SaaS workloads | Banks and enterprises (China first), then global Oracle-replacement deals |
| **International** | Available on Alibaba Cloud international regions; limited standalone presence | Expanding: OB Cloud 60+ regions/AZs, AWS/GCP marketplaces, SEA/Middle East push |
| **Community** | Closed (managed service); PolarDB-X open source | Open source since 2021 (GitHub/Gitee, Mulan 2.0), active community |
| **TPC-C crown** | Current holder: 2.055B tpmC (2025, Limitless mode) | Former holder: 707M tpmC (2020); first Chinese record-holder (2019) |

### 4.2 Architecture in Depth: Shared-Storage vs Shared-Nothing

**PolarDB (shared-storage):** one write node + many read replicas over a shared storage pool.

- **Elasticity:** compute is stateless — scale read capacity by adding replicas in seconds, scale storage without migration. The app keeps the "single database" mental model.
- **Read scale:** excellent (up to 15 replicas; more via PolarDB-X routing).
- **Write scale:** inherently limited to the primary's CPU/storage bandwidth. Alibaba Cloud's answers are (a) **PolarDB-X** (true sharding) and (b) **PolarDB Limitless multi-master** (multiple RW nodes over shared storage with distributed extensions) — the latter is newer, workload-dependent, and not the default topology.
- **Simplicity:** the least-distributed thing that looks distributed — a MySQL/PG DBA can operate it; application code is unchanged.

**OceanBase (shared-nothing):** peer nodes, each owning partitions, replicating by Paxos.

- **Write scale:** native — partition-level leaders mean many nodes write concurrently; add nodes to add write capacity; the 707M tpmC record was a horizontal-expansion exercise.
- **Read scale:** follower reads, plus the same Paxos groups; read scale-out is configurable.
- **Multi-writer:** yes — every partition has a leader, so "multi-writer" is the default (the answer to the "does OceanBase support multi-writer?" question: it is multi-writer at partition granularity by design; a single partition has one leader at a time).
- **Cost:** you now operate a distributed system — partition design, 2PC behaviour on cross-partition transactions, compaction, and Paxos-group placement are real DBA concerns, though 4.x's single-machine mode removes the minimum-size tax.

**The trade-off in one line:** PolarDB is read-scalable and write-single-node (simple, cloud-native); OceanBase is write-scalable and read-scalable (partition-level leaders), at the price of distributed-system complexity. For workloads whose write rate exceeds one powerful node, only the shared-nothing design (OceanBase, or PolarDB-X/Limitless within the PolarDB family) applies.

### 4.3 Compatibility Comparison

| Compatibility axis | PolarDB | OceanBase |
|---|---|---|
| **MySQL** | Yes — flagship (engine tracks MySQL 8.0; 8.0.1 LTS fully compatible with MySQL Community 8.0.13) | Yes — main mode, both editions |
| **PostgreSQL** | Yes — PolarDB for PostgreSQL | No |
| **Oracle** | PolarDB-O (Oracle-compatible edition) — availability to verify; otherwise PG-based migration path | Yes — Oracle mode is a strategic, bank-proven migration surface (the 去IOE workhorse) |
| **Ecosystem tooling** | MySQL/PG ecosystems; binlog CDC; Alibaba Cloud DTS; DataWorks | MySQL/Oracle ecosystems; binlog; OMS migration service; ODC developer tooling; third-party tools (20+ supported) |
| **Migration story** | MySQL/PG → PolarDB is low-friction; Oracle → PolarDB-O/PG requires more work | MySQL → low-friction; **Oracle → Oracle mode** is the flagship migration path for Chinese banks replacing Oracle cores |

The Oracle-migration question (去IOE) is where the two diverge strategically: OceanBase's Oracle mode is battle-tested in Chinese bank core migrations (ICBC and others — [chinese_bank_core_systems_guide.md §5.4](../banking/chinese_bank_core_systems_guide.md)); PolarDB's Oracle story is thinner and its current emphasis sits on MySQL/PG. If the workload is Oracle and the destination must be domestic, OceanBase Oracle mode is the conservative default; if the destination is PostgreSQL (or the cloud), PolarDB PG is the modernisation play (see [oracle_database_guide.md §5](oracle_database_guide.md) for the incumbent's view).

### 4.4 Performance Comparison

| Benchmark | PolarDB | OceanBase |
|---|---|---|
| TPC-C | **2.055B tpmC** (2025), 2,340 RW nodes, $0.11/tpmC — current world record, ~2.9× OceanBase's 2020 result | 60.88M tpmC (2019) → **707M tpmC** (2020, ¥3.98/tpmC) — first Chinese DB to top TPC-C; held the crown 2019–2025 |
| Peak-load proof | Tmall Double 11 core transaction system (via PolarDB-X heritage) | Alipay payment core; Double 11 peaks ~610,000 TPS (2021, widely reported) |
| Headline strength | Elastic read scale; storage-side log replay; serverless | Write scale-out; LSM write throughput; RPO=0 failover; single-machine-to-hundreds-of-nodes range |

Both are elite OLTP engines by global standards — the TPC-C leaderboard has been an Alibaba-family property since 2019. Read the caveats in §3.6 before quoting either number in a procurement.

### 4.5 Deployment and Licensing Comparison

| | PolarDB | OceanBase |
|---|---|---|
| Public cloud | Alibaba Cloud (international incl.) — native managed service | OB Cloud on Alibaba Cloud, AWS, GCP (+ others); managed |
| Private/on-prem | PolarDB Stack via Apsara Stack (China-focused 专有云; verify SKUs) | Fully on-prem: the bank default; bare metal/VM/K8s; 信创-certified stacks |
| Open source | PolarDB-X only | Community Edition (Mulan 2.0) — full self-managed path |
| Cost model | Cloud SKUs (pay per compute/storage/PCU); serverless pricing; cloud lock-in | Enterprise license (per-node/per-core) + cloud SaaS; open-source free tier for self-managed |
| Support | Alibaba Cloud support org | OceanBase Inc. support; global support centres (Malaysia 2026) |

### 4.6 High Availability and Disaster Recovery in Depth

| HA/DR axis | PolarDB | OceanBase |
|---|---|---|
| **Replication mechanism** | Redo-log shipping to PolarStore (triple-replicated, AZ-scatter by default); storage materialises pages | Multi-Paxos quorum per partition (typically 3 replicas across AZs/sites) |
| **RPO** | 0 within the storage replication domain (committed log is durable before ack) | 0 by consensus (quorum ack before commit returns) |
| **RTO** | Seconds-to-tens-of-seconds on primary failure (replica promotion against shared storage); vendor-stated | Seconds (vendor-stated ~8–30s by configuration); automatic leader re-election |
| **Read availability during failover** | Replicas keep serving reads; primary promotion is orchestrated | Followers keep serving reads; new leader elected by Paxos |
| **Multi-AZ / multi-region** | Multi-AZ via PolarStore replication; DR via DTS cross-region replication (async) | Multi-AZ native; Paxos groups can span regions for active-active/geo-distributed (单元化 pattern) |
| **Site failure (whole AZ/region)** | Storage quorum (2 of 3 copies) keeps the cluster alive; RPO=0 within 3-AZ deployments | Paxos majority keeps partitions alive; RPO=0 with 3-site quorum |

The architectural difference shows up in the DR story: **PolarDB's HA is a storage property** (the shared log is the source of truth; any compute node can pick it up), while **OceanBase's HA is a consensus property** (the log lives on the peers themselves, so a surviving majority can always form a new leader). Both achieve RPO=0 and seconds-level RTO in their native deployments — for banking, the practical differentiator is the proven *pattern*: OceanBase has run bank-grade, cross-site, RPO=0 cores in production for years; PolarDB's strongest HA proof is the cloud managed-service SLA (see [chinese_bank_core_systems_guide.md §5.3](../banking/chinese_bank_core_systems_guide.md) for the 单元化/dual-active context).

### 4.7 Migration Paths in Detail

| From → To | Typical journey | Key risks |
|---|---|---|
| **MySQL → PolarDB for MySQL** | DTS one-time sync + binlog incremental; connection-string cutover | Minor version/parameter differences (test 8.0.1 vs 8.0.2); no schema redesign expected |
| **PostgreSQL → PolarDB for PG** | DTS/dump-restore; extension compatibility check | Extension availability differences (postgis, etc.) — verify each extension |
| **Oracle → OceanBase Oracle mode** | OMS schema/PL-SQL assessment → migration → dual-run; package-by-package PL/SQL porting | Dialect edge cases (hints, packages, sequences, datatypes); the long tail — budget 3–6 months of testing per package inventory |
| **Oracle → PolarDB (O/PG)** | Assessment via Alibaba Cloud tools → DTS → rewrite PL/SQL to PG/PLpgSQL where needed | Heavier rewrite than OceanBase Oracle mode; PolarDB-O availability caveats |
| **MySQL → OceanBase MySQL mode** | OMS migration; partition/sharding design for large tables | Distributed design decisions (partition keys, table groups) must be made upfront |
| **Sharded legacy (分库分表) → PolarDB-X / OceanBase** | Consolidate shards onto the distributed engine's native sharding | Transaction semantics across former shards; global secondary index design |

Rule of thumb from real migrations: **compatibility claims transfer the schema, not the edge cases** — every migration budget should include a dialect-difference testing phase measured in weeks, not days, regardless of which product wins (see the worked example in §8 for the bank-scale version of this).

---

## 5. Use Cases

### 5.1 PolarDB Use Cases

- **Cloud-native OLTP on Alibaba Cloud:** e-commerce, SaaS, internet applications where the deployment is (or will be) Alibaba Cloud and the team wants managed MySQL/PG with elastic scaling and serverless economics. Read-heavy, elastic traffic (flash sales, promotions, seasonal spikes) is the sweet spot — add read replicas in seconds, scale to zero at night.
- **MySQL/PostgreSQL modernisation to the cloud:** lift-and-shift or re-platform MySQL/PG estates onto a managed, Aurora-class engine without rewriting SQL.
- **Oracle/legacy migration to the cloud (PolarDB-O / PolarDB PG):** Oracle estates moving to Alibaba Cloud can target PolarDB-O (verify availability) or modernise onto PolarDB PostgreSQL.
- **HTAP-lite:** OLTP with a columnar/IMCI assist for operational analytics — when you want to avoid a separate warehouse for modest analytical loads.
- **Distributed MySQL workloads (PolarDB-X):** high-concurrency, high-volume MySQL workloads needing sharding — Double 11-style transactional scale on a MySQL-compatible surface.

### 5.2 OceanBase Use Cases

- **Financial core systems:** banking core (deposits, loans, accounts, payments), insurance policy systems, securities/clearing — the reference domain (ICBC, MYbank, and the banking landscape in [chinese_bank_core_systems_guide.md §7.1](../banking/chinese_bank_core_systems_guide.md)).
- **Extreme OLTP:** payment platforms, e-commerce transaction cores, any workload with sustained high write TPS and strict consistency — Alipay/Double 11-grade.
- **Oracle replacement (去IOE):** Oracle-core → OceanBase Oracle-mode migrations, the canonical China-banking pattern (§4.3).
- **Enterprise core systems:** telco BSS/OSS, logistics, retail transaction cores, government platforms — anywhere ACID + scale + on-prem/信创 are required simultaneously.
- **Hybrid cloud / multi-cloud:** OB Cloud makes it deployable on AWS/GCP/Aliyun with the same engine, for cloud-first banks and enterprises.

### 5.3 Use-Case Comparison Table

| Workload | PolarDB fit | OceanBase fit |
|---|---|---|
| Read-heavy cloud OLTP, elastic (SaaS, e-commerce, internet) | **Excellent** — 1+15 replicas, serverless, seconds-level scale | Good — follower reads, but the distributed design is overkill for pure read scale |
| High write TPS on one logical DB (payments, transaction cores) | Moderate — single-primary write ceiling; needs Limitless/PolarDB-X | **Excellent** — partition-level write scale-out is the design point |
| Banking core (deposits/loans/ledger), ACID + RPO=0 + on-prem | Moderate — PolarDB Stack on Apsara; less banking core provenance | **Excellent** — the reference domain (ICBC, MYbank) |
| Oracle → domestic migration | Moderate — PolarDB-O/PG (availability caveats) | **Excellent** — Oracle mode is the flagship migration path |
| MySQL estate on Alibaba Cloud | **Excellent** — lowest friction, managed | Good — MySQL mode, but you carry distributed ops |
| PostgreSQL estate (any cloud) | **Excellent** — PolarDB PG | Not supported |
| On-prem + 信创 compliance | Moderate — Apsara Stack 专有云; verify | **Excellent** — on-prem-first design, 信创-certified stacks |
| Open-source, self-managed, portable | PolarDB-X only (distributed edition) | **Excellent** — Community Edition (Mulan 2.0) |
| Hybrid/multi-cloud managed DBaaS | Aliyun-centric | **Excellent** — OB Cloud across Aliyun/AWS/GCP |
| HTAP (OLTP + operational analytics) | Good — IMCI columnar index, ePQ | Good — 4.3+ columnar store |

### 5.4 When NOT to Choose Each (Anti-Patterns)

- **Don't choose PolarDB for:** sustained multi-hundred-thousand-TPS *writes* on one logical database (single-primary ceiling — you will be forced into Limitless/PolarDB-X mid-project); on-prem-first estates outside China (PolarDB Stack is China-专有云-centric); workloads needing PostgreSQL on non-Aliyun clouds; teams that must self-manage and escape cloud lock-in.
- **Don't choose OceanBase for:** pure read-heavy elastic web workloads with modest writes (the distributed design buys nothing and costs DBA complexity); PostgreSQL workloads (not supported — pick PolarDB PG or open-source PG); small teams with no distributed-DB experience and no budget for training/certification; workloads where the app cannot tolerate even the small operational overhead of partition design and 2PC tuning.
- **Both, in common:** neither is right for document/JSON-at-scale or graph workloads (see [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md)); neither replaces a dedicated analytical warehouse for heavy BI (HTAP features help, but a warehouse/OLAP engine remains the answer for serious analytics); both are wrong choices for teams whose real problem is application design, not database capacity.

---

## 6. Selection Framework

### 6.1 The Decision Tree

```
Is the workload relational OLTP (ACID)?
├─ No → see [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md)
└─ Yes → Where will it run?
   ├─ Alibaba Cloud, managed, MySQL/PG, read-heavy/elastic
   │   └─ → POLARDB (MySQL or PG; Serverless if spiky)
   ├─ Cloud, but multi-cloud or AWS/GCP, or open-source self-managed
   │   └─ → OCEANBASE (OB Cloud or Community Edition)
   ├─ On-premises (bank/enterprise/信创) with extreme write scale
   │   └─ → OCEANBASE (on-prem, MySQL or Oracle mode)
   ├─ Oracle core being decommissioned (去IOE)
   │   ├─ Destination domestic distributed → OCEANBASE Oracle mode
   │   └─ Destination cloud/PG modernisation → POLARDB (PolarDB-O/PG)
   ├─ Single-node write volume is enough; read scale is the need
   │   └─ → POLARDB (shared-storage read replicas)
   ├─ Write volume exceeds one powerful node; strong consistency across shards
   │   └─ → OCEANBASE (native), or POLARDB-X / PolarDB Limitless within the family
   └─ Open-source + self-managed + MySQL-compatible
       └─ → OCEANBASE Community Edition (or PolarDB-X if the estate is Aliyun-flavoured)
```

### 6.2 Decision Factors

| Factor | What to ask | PolarDB bias | OceanBase bias |
|---|---|---|---|
| **Deployment** | Cloud-only on Aliyun? On-prem? Hybrid/multi-cloud? | Cloud | On-prem + any cloud |
| **Compatibility** | MySQL, PostgreSQL, or Oracle surface? | MySQL/PG | MySQL/Oracle |
| **Scale** | Read-heavy or write-heavy? Will one primary's write ceiling bind? | Read scale | Write scale |
| **Consistency** | ACID strictness across shards; RPO/RTO targets | Strong in-cluster; single-writer | Strong across nodes; RPO=0 |
| **Licensing/cost** | Managed-cloud economics vs enterprise license vs open source; exit/portability | Pay-as-you-go cloud SKUs | License + cloud + free community tier |
| **Support** | Alibaba Cloud relationship vs Ant/OceanBase relationship | Alibaba Cloud org | OceanBase Inc. org |
| **信创/xinchuang** | Domestic certification and stack compatibility for China deployments | Certified via Alibaba Cloud/Apsara (verify catalogue) | Deep 信创 banking provenance ([chinese_bank_core_systems_guide.md §7.5](../banking/chinese_bank_core_systems_guide.md)) |
| **Skills** | MySQL skills (both), PG skills (PolarDB only), Oracle skills (OceanBase Oracle mode) | MySQL/PG DBA pool | MySQL + Oracle DBA pool |
| **Risk of lock-in** | Cloud platform lock-in (PolarDB) vs vendor lock-in with on-prem portability (OceanBase) | High (managed service) | Moderate (self-managed possible) |

### 6.3 The Decision Table

| Scenario | Recommendation | Rationale |
|---|---|---|
| Aliyun cloud-native OLTP, MySQL/PG, read-heavy, elastic | **PolarDB** | Managed, serverless, 15 read replicas, lowest friction |
| Extreme write TPS + strict ACID (payments, transaction core) | **OceanBase** | Partition-level write scale-out; RPO=0; proven at Alipay scale |
| Bank core on-prem + 信创 | **OceanBase** | The reference domain; on-prem-first; ICBC/MYbank provenance |
| Oracle decommissioning, domestic destination | **OceanBase Oracle mode** | Battle-tested migration path; PL/SQL compatibility |
| Oracle decommissioning, cloud/PG destination | **PolarDB (PolarDB-O/PG)** | Modernisation onto PG; cloud ops |
| MySQL estate, cloud-agnostic or AWS/GCP | **OceanBase (OB Cloud / Community)** | Runs where your cloud is; open-source option |
| MySQL estate, committed to Alibaba Cloud | **PolarDB for MySQL** | Native integration; simplest O&M |
| PostgreSQL estate | **PolarDB for PostgreSQL** | Only one of the two with PG |
| Open-source self-managed, no cloud preference | **OceanBase Community** | Mulan 2.0, full deployment freedom |
| "We may outgrow one node's writes in 3–5 years" | **OceanBase**, or PolarDB with a PolarDB-X exit path | Design for the ceiling, not the current load |

### 6.4 The Architect's Evaluation Checklist

A reusable pre-decision checklist for either product (lift into your decision record):

1. **Deployment reality:** cloud-only, on-prem, or hybrid? Is the destination cloud fixed (Aliyun, AWS, GCP)? → PolarDB only fits Aliyun-centric; OceanBase fits all three.
2. **Compatibility surface:** enumerate the SQL dialect, stored procedures, and drivers your estate actually uses; run a compatibility assessment (PolarDB: MySQL/PG test suite; OceanBase: MySQL/Oracle mode test suite) before believing "100% compatible" claims.
3. **Write ceiling:** model peak TPS × 5-year growth. If it exceeds one powerful node, the shared-nothing option (or PolarDB-X/Limitless) is mandatory, not optional.
4. **Consistency/HA contract:** write down RPO/RTO targets per system tier; ask each vendor for the *configuration* that achieves them (AZ count, quorum, DR site), not the headline.
5. **Migration path:** for Oracle estates, run a PL/SQL package inventory through the vendor's migration assessment tooling (OceanBase OMS vs PolarDB-O/PG tools); budget for the long tail of dialect edge cases.
6. **信创/compliance:** confirm the product is in the relevant domestic catalogues for China deployments; for non-China, confirm data-residency and support SLAs in your region.
7. **Skills:** audit the DBA/SQL skills pool — MySQL and Oracle skills transfer to both; PostgreSQL skills only to PolarDB; distributed-DB skills are scarce everywhere and need certification investment.
8. **TCO model:** build the 5-year model with *your* scale — managed-cloud per-PCU (PolarDB), enterprise license + hardware + DBA headcount (OceanBase on-prem), or open-source community + support contract (OceanBase CE).
9. **Exit plan:** what does leaving look like? PolarDB → dump/CDC to another cloud (costly); OceanBase on-prem → standard MySQL-protocol tooling and open-source code access ease exit.
10. **Reference check:** visit a same-industry, same-scale reference (banking: ICBC/MYbank for OceanBase; large Aliyun SaaS estates for PolarDB) and ask about operational incidents, not marketing slides.

---

## 7. Market and Ecosystem

### 7.1 The China Database Market

- **The context is 信创 (xinchuang).** China's database market is driven by the domestic-substitution programme: foreign databases (Oracle, IBM Db2, Microsoft SQL Server) are being replaced in new builds and migrated in legacy estates, with finance among the priority industries (the "2+8" scope — see [chinese_bank_core_systems_guide.md §7.5](../banking/chinese_bank_core_systems_guide.md)). This creates a multi-billion-yuan market that is effectively reserved for domestic vendors — the same force that made OceanBase and PolarDB household names in Chinese IT.
- **Market structure:** per IDC China's relational-database market reports of 2024–2025, the China market is led by a small group of domestic vendors — **OceanBase and Alibaba Cloud (PolarDB + RDS)** at the top, with **Huawei GaussDB** and **Tencent TDSQL** close behind — followed by TiDB, GoldenDB, Dameng, KingbaseES and openGauss. **Exact shares vary by report vintage and category (on-prem vs cloud, OLTP vs OLAP); verify against the current IDC China release before quoting a percentage in a deck.** Gartner has ranked Alibaba Cloud among the leaders in Asia-Pacific DBMS by revenue; OceanBase's own marketing cites top-tier China share figures from IDC.
- **The 去IOE tailwind:** both products are children of 去IOE ([chinese_bank_core_systems_guide.md §5.1](../banking/chinese_bank_core_systems_guide.md)) — PolarDB as the cloud-native successor to MySQL/Oracle estates on Alibaba Cloud, OceanBase as the direct Oracle replacement in bank cores. The two now compete for the same 信创 database contracts, which is precisely why this head-to-head matters.

### 7.2 The Chinese Database Vendors

| Vendor | Flagship DB | Architecture / compatibility | Position |
|---|---|---|---|
| **Alibaba Cloud** | **PolarDB** (+ RDS) | Shared-storage; MySQL/PG | The cloud-native incumbent; strongest on Alibaba Cloud estates |
| **Ant Group / OceanBase Inc.** | **OceanBase** | Shared-nothing; MySQL/Oracle | The financial-grade distributed leader; TPC-C records |
| **Tencent** | **TDSQL** (+ TDSQL-C) | Distributed, MySQL-compatible (Raft-based) | WeBank/Tencent ecosystem; reported at several banks |
| **Huawei** | **GaussDB** (+ openGauss) | Distributed, PG-based; openGauss open-source sibling | The 信创 stack default with Kunpeng hardware; ICBC/PSBC-scale references |
| **ZTE** | **GoldenDB** | Distributed, shared-nothing, MySQL-compatible | CITIC Bank core "heart transplant" |
| **PingCAP** | **TiDB** | NewSQL (Raft), MySQL-compatible, open source | Open-source darling; fintech/regional banks; now AI-agent positioning |

(Banking footprint details for each are in the landscape table of [chinese_bank_core_systems_guide.md §7.1](../banking/chinese_bank_core_systems_guide.md).) The strategic point: **the Alibaba family holds two of the top slots with two different architectures** — an unusual competitive asset, and an unusual internal rivalry.

### 7.3 International Landscape

- **The global baseline:** the world OLTP market is still Oracle/PostgreSQL/MySQL-shaped (see [oracle_database_guide.md §1](oracle_database_guide.md) and §5). Chinese databases are the challengers, with three international plays: (1) **OceanBase** — OB Cloud on AWS/GCP/Aliyun, 60+ regions claimed, SEA/Middle East expansion, Oracle-compatibility-driven; (2) **PolarDB** — international regions of Alibaba Cloud, consumed mainly by Alibaba Cloud customers; (3) **TiDB** — the open-source global favourite. For a Singapore-based bank or fintech, OceanBase is the one most likely to show up in a vendor shortlist today; PolarDB appears when the platform is Alibaba Cloud.
- **The Oracle-replacement wedge internationally:** OceanBase's Oracle mode travels well — Southeast Asian and Middle Eastern banks running Oracle cores are the same migration profile as Chinese banks, minus the 信创 mandate. This is the most plausible vector for either product to win significant non-China market share.

### 7.4 Ecosystem: Tooling, Marketplaces, and Partnerships

| Ecosystem layer | PolarDB | OceanBase |
|---|---|---|
| **Cloud marketplaces** | Native to Alibaba Cloud; consumed via the Aliyun console | OB Cloud on Alibaba Cloud, AWS Marketplace (2022), GCP Marketplace |
| **Migration tooling** | Alibaba Cloud DTS (Data Transmission Service); PolarDB-X PXD/K8s tools; binlog CDC | OMS (MySQL/Oracle/others); binlog CDC; OBLOADER/OBDUMPER |
| **Dev/ops tooling** | Aliyun console, Cloud DBA, DMS; serverless console | OCP, ODC, OBProxy, K8s operator, Ansible |
| **Open source** | PolarDB-X (Apache-2.0 SQL layer + polardbx-engine); no PolarDB core | OceanBase Community Edition (Mulan 2.0); full engine source |
| **Analytics/HTAP** | IMCI columnar index, ePQ; AnalyticDB integration | 4.3+ columnar store; vector features; integration with Flink/Spark |
| **AI ecosystem** | Qwen/Model Studio integration; PolarDB agent tooling (GitHub); vector support (PG engine) | LangChain integration package; vector/LLM-era positioning |
| **Certifications/community** | Alibaba Cloud certification tracks; PolarDB open-source community events | OCA/OCP/OCM certifications; 20+ third-party tools; GitHub/Gitee community; Gartner Peer Insights |

Partnership note for architects: **PolarDB's ecosystem is the Alibaba Cloud ecosystem** — its value compounds if you are already on Aliyun (IaaS, DTS, DataWorks, MaxCompute, Observability). **OceanBase's ecosystem is deliberately multi-cloud and open-source** — it attaches to whichever cloud you choose, plus your own data centre. The ecosystem question is therefore really the platform question in disguise.

---

## 8. Worked Example: A Bank Evaluating Both for a New Core-Banking Platform

### 8.1 The Requirements

A mid-tier Asian bank (for realism: a Chinese joint-stock bank with a Singapore regional subsidiary) is replacing an Oracle-based core-banking platform. Requirements, as agreed with the architecture board:

| # | Requirement | Weight | Why this weight |
|---|---|---|---|
| R1 | Full ACID, strong consistency for accounts/ledger/postings | 5 | Non-negotiable for core; regulator-visible |
| R2 | High availability: **RPO = 0**, RTO < 60s | 5 | Loss of committed transactions is a compliance event |
| R3 | Write scale: peak 200k+ TPS, growth to 500k+ TPS; horizontal write scale-out expected | 5 | The reason the old Oracle/RAC design is being replaced |
| R4 | **Oracle compatibility** for migrating the existing PL/SQL application estate | 4 | Reuse of the existing core application, not a rewrite |
| R5 | Deployment: **on-prem (信创 stack) now, cloud option later** | 4 | Regulator requires domestic on-prem for the China entity; Singapore entity wants cloud optionality |
| R6 | **Xinchuang compliance** (domestic certification, domestic chips/OS) | 5 | Procurement gate for the China entity |
| R7 | MySQL skills availability in the team | 3 | Existing DBA pool is Oracle-skilled; MySQL is a bonus, not a need |
| R8 | TCO over 5 years (licenses, hardware, O&M) | 3 | Budget-constrained |
| R9 | Vendor support and roadmap stability | 3 | Both are Alibaba-family; still scored |

Weights sum to **37**. Scoring: 1 (poor) to 5 (excellent).

### 8.2 The Scored Comparison

| # | Weight | OceanBase score (w×s) | PolarDB score (w×s) | Score justification |
|---|---|---|---|---|
| R1 | 5 | 5 → 25 | 5 → 25 | Both full ACID. OceanBase across nodes; PolarDB in-cluster (Limitless adds distributed ACID but is newer) |
| R2 | 5 | 5 → 25 | 4 → 20 | Both claim RPO=0; OceanBase's Paxos quorum + seconds failover is the bank-proven pattern; PolarDB's storage-side failover is solid but less core-bank-proven |
| R3 | 5 | 5 → 25 | 4 → 20 | OceanBase: native partition-level write scale-out (707M tpmC proof). PolarDB: single-primary ceiling; needs Limitless/PolarDB-X — workable but not the default topology |
| R4 | 4 | 5 → 20 | 4 → 16 | OceanBase Oracle mode is the de-facto 去IOE migration surface (ICBC). PolarDB-O exists but with availability caveats; PG-based migration is heavier |
| R5 | 4 | 5 → 20 | 3 → 12 | OceanBase on-prem-first with OB Cloud optionality. PolarDB is cloud-first; PolarDB Stack on Apsara is China 专有云, less flexible |
| R6 | 5 | 5 → 25 | 4 → 20 | Both domestic; OceanBase has the deeper banking 信创 track record (on-prem bank cores); PolarDB's certification is via Alibaba Cloud/Apsara |
| R7 | 3 | 4 → 12 | 5 → 15 | MySQL mode in both; PolarDB also offers PG; neither replaces the Oracle-skilled team's main need |
| R8 | 3 | 3 → 9 | 4 → 12 | PolarDB's managed-cloud economics are cheaper at this scale; OceanBase on-prem requires servers + licenses + distributed-DBA ops |
| R9 | 3 | 4 → 12 | 4 → 12 | Both Alibaba-family, both invest heavily; OceanBase Inc. is the more focused database company |
| **Σ** | **37** | **173 → 4.68** | **152 → 4.11** | |

Arithmetic check (worked per candidate, Σ(w×s)/Σw): **OceanBase** = (25+25+25+20+20+25+12+9+12)/37 = 173/37 ≈ **4.68**. **PolarDB** = (25+20+20+16+12+20+15+12+12)/37 = 152/37 ≈ **4.11**.

### 8.3 Recommendation and Rationale

**Primary recommendation: OceanBase (MySQL mode for new services, Oracle mode for the migrated core application), on-prem for the China entity and OB Cloud for the Singapore entity**, with the migration executed via OMS and a dual-core parallel run (双轨运行 — see [chinese_bank_core_systems_guide.md §5.5](../banking/chinese_bank_core_systems_guide.md)). Rationale: OceanBase wins on the four requirements that cannot be compromised (R2 RPO=0, R3 write scale, R4 Oracle compatibility, R6 xinchuang) — a 0.57-point weighted gap that is driven entirely by core-critical factors, not by preferences. The team's Oracle skills carry over through Oracle mode, and the "on-prem now, cloud later" path is native.

**Contingent recommendation:** if the bank's strategy were instead "cloud-only on Alibaba Cloud, no China entity, greenfield MySQL, moderate write scale," PolarDB for MySQL (with PolarDB-X as the scale-out escape hatch) would be the better value — §8.4 quantifies the flip.

**Deployment shape regardless of winner:** accounts/customer data on the winner; read-heavy channel and CRM workloads can sit on PolarDB (or any read-scalable engine) beside the core; a Kafka/RocketMQ event pipeline decouples hot path from analytics; DR via the winner's geo-replication.

**Residual risks:** Oracle-mode PL/SQL edge cases (test every package); OceanBase DBA skills are scarce outside China (plan certification/training); TCO of the on-prem distributed estate (server count grows with scale); vendor-concentration — the bank is betting the core on an Ant subsidiary whose external funding/IPO path is still unproven (mitigate with escrow/source-access clauses — Community Edition source is available).

**Lesson of the exercise:** the winner won on *requirements*, not on product quality — both are excellent engines. When RPO=0, write scale-out, Oracle migration and xinchuang are all hard constraints, the shared-nothing, on-prem-first, Oracle-compatible product wins; the shared-storage cloud-native product wins the complementary set.

### 8.4 Sensitivity Analysis

| Perturbation | New context | Result | Verdict |
|---|---|---|---|
| **A. Xinchuang/on-prem weights soften** (R6 5→3, R5 4→3; Σ=33) | Same bank, less regulatory pressure | OceanBase (25+25+25+20+15+15+12+9+12)/33 = **4.79** vs PolarDB (25+20+20+16+9+12+15+12+12)/33 = **4.27** | OceanBase's lead **grows** — its advantage sits in the core requirements, not the policy ones |
| **B. Cloud-only Aliyun SaaS, greenfield MySQL, no Oracle legacy, moderate writes** (R3 5→2, R4 4→0, R6 5→2, R7 3→4, cloud preference scores reversed; Σ=28) | A different bank: no China entity, no legacy | OceanBase (25+25+10+0+16+10+16+9+12)/28 = **4.39** vs PolarDB (25+20+8+0+20+8+20+12+12)/28 = **4.46** | **PolarDB wins narrowly** — a knife-edge. Resolution: decide on cloud strategy and team skills (Alibaba Cloud commitment + MySQL/PG pool → PolarDB), not on the score |

The ranking is robust for the bank-core scenario (A) and flips only for a fundamentally different workload+deployment profile (B) — exactly the behaviour a decision framework should have. State this in the decision record: *"OceanBase for distributed core OLTP with strict HA; PolarDB for cloud-native elastic OLTP on Alibaba Cloud — the two are complements, not substitutes, across the portfolio."*

---

## 9. The Future: 2026 and Beyond

### 9.1 PolarDB Evolution

- **PolarDB-X growth:** expect PolarDB-X to carry more of the family's scale story — deeper MySQL compatibility, more automation (K8s operator, auto-sharding), and tighter integration with the Limitless multi-master line; the open-source community version keeps it a credible open option.
- **Limitless multi-master:** the 2.055B-tpmC "Limitless" topology will normalise multi-writer on shared storage — watch for it becoming the recommended topology for high-write workloads on Alibaba Cloud, which would narrow the architectural gap with OceanBase.
- **AI integration:** vector support, Qwen-powered natural-language database interaction, AI-driven self-tuning and agentic O&M tooling (Alibaba Cloud already publishes PolarDB agent tooling on GitHub) — the "AI database" marketing will accelerate through 2026–2027.
- **Serverless maturity:** wider serverless adoption (both engines), finer-grained scaling, and "scale-to-zero with instant wake" economics.
- **International:** PolarDB remains Alibaba-Cloud-bound; international growth tracks Alibaba Cloud's own expansion rather than standalone selling.

### 9.2 OceanBase Evolution

- **Version line:** 4.x remains the mainstream (4.3/4.4+ with HTAP columnar store); a **5.x line has been reported in vendor and community materials but is not confirmed in official release notes as of August 2026 — verify** before citing it in an architecture deck.
- **Corporate trajectory:** the spin-off from Ant continues (first external financing sought per SCMP, late 2023); an **Ant Group IPO — and with it OceanBase's own funding/IPO narrative — remains the big unknown**; a successful OceanBase round would fund the international push.
- **International:** OB Cloud's 60+ regions claim, the Malaysia support centre (2026), and the GO GLOBAL GO program signal sustained SEA/Middle East/Japan expansion; Oracle-mode migrations are the wedge.
- **HTAP and AI:** 4.x columnar/HTAP features mature; OceanBase's AI positioning (vector store, AI-era scalability claims) will follow the industry pattern.

### 9.3 Convergence and Competition

- **Alibaba-family synergy:** Alibaba Cloud hosts **OceanBase Cloud** on its marketplace even as PolarDB competes with it — the family sells both. Expect continued co-existence: Aliyun wins either way; the rivalry is managed, not resolved.
- **External competition:** **TDSQL** (Tencent) and **GaussDB** (Huawei) are the strongest challengers in China bank cores; **TiDB** leads the open-source/global narrative; internationally, Oracle's 23ai/26ai (see [oracle_database_guide.md §9](oracle_database_guide.md)) and the cloud hyperscalers' Aurora/Cloud Spanner/AlloyDB remain the incumbents to displace.
- **AI databases:** both vendors are rebranding around AI (vector search, AI agents, natural-language SQL). For architects, evaluate AI features as product features, not as reasons to pick a database — the OLTP engine decision remains what it has always been.
- **Trends summary:** (1) the two architectures are converging on a middle ground (PolarDB adding multi-writer scale-out; OceanBase adding single-machine entry); (2) the strategic battleground moves from "which engine" to "which deployment and which compliance story"; (3) in China, 信创 keeps both inside the domestic winners' circle; internationally, Oracle-compatibility is the winning wedge; (4) open source (OceanBase Community, PolarDB-X) increasingly anchors both ecosystems.

### 9.4 What to Watch (Signals for 2026–2028)

| Signal | What it would mean |
|---|---|
| OceanBase 5.x official GA | Confirms the next architecture generation (HTAP/AI direction); update all version claims |
| OceanBase external funding round closes | Funded international push; validates the spin-off thesis; watch valuation |
| Ant Group IPO relaunch | The biggest corporate catalyst for OceanBase's independence and pricing power |
| PolarDB Limitless as default topology | Confirms multi-writer on shared storage as the family's answer to write scale-out — narrows the architectural gap |
| PolarDB-O roadmap clarity | Determines whether PolarDB is a credible Oracle-migration target or strictly MySQL/PG |
| OB Cloud on Azure / more regions | OceanBase's multi-cloud reach widens the international addressable market |
| 信创 2.0 catalogue updates | Both products' China ceiling/floor moves with the domestic-substitution scope (see [chinese_bank_core_systems_guide.md §10.1](../banking/chinese_bank_core_systems_guide.md)) |
| TDSQL/GaussDB bank-core wins | Competitive pressure on OceanBase's China core-ledger position; watch share reports |
| AI-database feature shipping (not slides) | Vector/agent features that actually run bank workloads will shift greenfield decisions |

---

## 10. Glossary

| Term | Definition |
|---|---|
| **PolarDB** | Alibaba Cloud's cloud-native relational database: shared-storage architecture (compute-storage separation), MySQL/PostgreSQL-compatible, launched 2017 |
| **PolarStore** | The shared distributed storage layer of PolarDB: triple-replicated, SSD-based, applies the redo log to materialise pages (log-as-the-database) |
| **PolarDB-X** | PolarDB's distributed (shared-nothing) edition, the former DRDS: sharding, 2PC distributed transactions, Multi-Paxos data nodes, MySQL-compatible, open source |
| **OceanBase** | Ant Group's native distributed relational database (shared-nothing, Multi-Paxos, LSM-tree), MySQL/Oracle-compatible; TPC-C record-holder 2019–2025 |
| **OBServer** | An OceanBase server node: owns partitions (tablets), runs SQL + storage locally; peers form Paxos groups |
| **Paxos** | Consensus protocol family for agreeing on a value (log entry) across replicas; the basis of fault-tolerant replication |
| **Multi-Paxos** | Paxos optimised for log replication (leader-elected, chained rounds); used by OceanBase and PolarDB-X data nodes |
| **LSM-tree** | Log-structured merge tree: writes to an in-memory memtable, flushed to immutable SSTables, merged by compaction; write-optimised storage |
| **Memtable** | The in-memory write buffer of an LSM-tree engine (OceanBase's write path) |
| **SSTable** | Sorted immutable on-disk table file produced by flushing a memtable |
| **Shared-storage architecture** | Many compute nodes attach to one replicated storage pool (PolarDB, Aurora); storage does replication |
| **Shared-nothing architecture** | Each node owns its data locally; scale-out by partitioning across nodes (OceanBase, Spanner) |
| **Compute-storage separation** | Decoupling the SQL/transaction engine from storage so each scales independently — the defining cloud-native move |
| **Read replica** | A read-only copy of the database serving reads; PolarDB supports up to 15 read nodes per cluster |
| **2PC** | Two-phase commit: prepare + commit across participating nodes; how distributed databases make cross-shard transactions atomic |
| **TPC-C** | The Transaction Processing Performance Council's OLTP benchmark (order-entry workload); the industry's flagship OLTP yardstick |
| **tpmC** | TPC-C throughput unit: transactions per minute |
| **Double 11 / 双11** | Singles' Day (11 November): the world's largest online shopping event; Alibaba's extreme-scale stress test that forged PolarDB-X and OceanBase |
| **Ant Group** | Alibaba's fintech arm (Alipay's parent), separated from Alibaba Group in 2020; owner of OceanBase Inc. |
| **Alipay** | Ant Group's payments platform; OceanBase runs all of its mission-critical systems |
| **MYbank (网商银行)** | Ant Group's digital bank, built on OceanBase; famous for the 310 model (3-minute application, 1-second approval, 0 manual intervention) |
| **去IOE** | "Remove IBM, Oracle, EMC": the Chinese movement to replace the foreign platform stack; started at Alibaba, now national policy-adjacent |
| **Xinchuang (信创)** | 信息技术应用创新: the national programme to replace foreign IT in critical industries with domestic technology; finance is a priority sector |
| **TDSQL** | Tencent's distributed MySQL-compatible database; WeBank's engine; a PolarDB/OceanBase competitor |
| **GaussDB** | Huawei's distributed database (PG-based), with the openGauss open-source sibling; a leading 信创 bank-core choice |
| **TiDB** | PingCAP's open-source NewSQL database (Raft-based, MySQL-compatible) |
| **Mulan license** | Mulan Public License v2 — the Chinese open-source license (OSI-approved) under which OceanBase Community Edition is released |
| **RPO** | Recovery Point Objective: maximum acceptable data loss (OceanBase: 0; PolarDB: 0 within the storage domain) |
| **RTO** | Recovery Time Objective: maximum acceptable downtime (both: seconds-to-tens-of-seconds, vendor-stated) |
| **HTAP** | Hybrid Transactional/Analytical Processing: one engine serving OLTP + OLAP (PolarDB IMCI/ePQ; OceanBase 4.3+ columnar store) |
| **Serverless** | Database that scales compute elastically on demand and bills for usage (PolarDB Serverless, PCU-based) |
| **Cloud-native** | Designed for cloud delivery: separation of compute/storage, elasticity, managed operations |
| **Distributed SQL** | A database that speaks standard SQL across many nodes with distributed transactions (OceanBase, PolarDB-X, TiDB, Spanner) |

---

## 11. References and Verification Notes

**Sibling guides in this repo (cross-referenced above):**

- [chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md) — the China banking landscape: 去IOE (§5.1), flagship distributed-core projects incl. ICBC/OceanBase (§5.4), dual-core parallel run (§5.5), MYbank and the 310 model (§6.2), the domestic database landscape (§7.1), 信创 framework (§7.5)
- [oracle_database_guide.md](oracle_database_guide.md) — the relational incumbent: overview (§1), architecture (§2), competitive position (§5), banking context (§7)
- [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) — the non-relational complement when ACID OLTP is not the requirement
- [cloud_providers_guide.md](cloud_providers_guide.md) — the cloud platform context for Alibaba Cloud deployments

**Primary sources consulted (August 2026):** Alibaba Cloud product/help documentation for PolarDB (PolarDB for MySQL engine release notes 8.0.2/8.0.1/5.7/5.6; PolarDB Serverless docs; PolarDB-X product pages and GitHub), the VLDB paper "OceanBase: A 707 Million tpmC Distributed Relational Database System" (PVLDB Vol 15, 2022; also documents the Mulan license and Alipay deployment scale), OceanBase official site/blog (400+ customers, OB Cloud, GO GLOBAL GO), TPC-published TPC-C results, SCMP reporting on Ant spin-offs, Alibaba Cloud TPC-C record announcements (2.055B tpmC, 2,340 nodes, $0.11/tpmC), PolarStore compression paper (arXiv 2025).

**Claims-status table** (task-brief claims vs verification outcome):

| # | Claim (from brief) | Status | Note |
|---|---|---|---|
| 1 | PolarDB launched 2017 | ✅ | Announced September 2017 at Apsara; Aurora-alternative positioning |
| 2 | PolarDB "1.0/2.0" versioning | ⚠ | No public 1.0/2.0 product versions; engines are versioned by MySQL/PG compatibility (8.0.2/8.0.1/5.7/5.6); "PolarDB 2.0" was a marketing banner (~2021), not a version |
| 3 | PolarDB read replicas "15/16" | ✅ (≈) | Documented topology: 1 RW primary + **1 to 15** read nodes |
| 4 | PolarDB-O (Oracle-compatible) | ⚠ | Exists in Alibaba Cloud documentation, but current availability/roadmap must be verified on aliyun.com; PolarDB PG now carries much of the Oracle-migration story |
| 5 | PolarDB-X = former DRDS | ✅ | Confirmed; shared-nothing, CN/DN/GMS, Multi-Paxos, 2PC, open source (Apache-2.0 SQL layer + MySQL-fork engine) |
| 6 | PolarDB multi-master | ✅ | "Limitless" multi-master mode for PolarDB for MySQL (sharded tables across multiple RW nodes) |
| 7 | PolarDB Serverless | ✅ | Confirmed for MySQL and PostgreSQL engines (PCU-based, seconds-level) |
| 8 | PolarDB on-prem / "PolarDB Stack" | ⚠ | PolarDB Stack exists via Apsara Stack (专有云, China-focused); SKUs/versions to verify |
| 9 | PolarDB TPC-C record | ✅ (new fact) | 2.055B tpmC, 2,340 RW nodes, $0.11/tpmC (2025) — PolarDB now holds the world record, surpassing OceanBase's 707M tpmC |
| 10 | OceanBase origin 2010, Taobao shopping cart, Yang Zhenkun | ✅ | Verified (official history + VLDB paper + OceanBase blog) |
| 11 | OceanBase 2.0 rewrite 2017 | ✅ | Widely documented; 2017 rewrite into the native distributed architecture |
| 12 | OceanBase open source 2021, Mulan license | ✅ | Community Edition 2021, Mulan Public License 2.0, GitHub + Gitee |
| 13 | OceanBase 4.x "single-machine distributed" | ✅ | 4.0 "Xiaoyu" (August 2022); 4.2–4.4+ current; 5.x **not confirmed** in official release notes as of Aug 2026 |
| 14 | TPC-C 60.88M tpmC 2019 (first Chinese DB) | ✅ | October 2019 result |
| 15 | TPC-C 7.07亿 tpmC "2021" | ⚠ | **Record is May 2020** (707M tpmC, ¥3.98/tpmC), not 2021 — brief mis-dated by one year |
| 16 | OceanBase RPO=0, RTO<30s | ⚠ | Vendor-stated (RPO=0 standard; RTO figures vary ~8s–30s by configuration) — verify per deployment |
| 17 | OceanBase scale "hundreds of nodes" | ✅ | Hundreds per cluster; tens of thousands of OBServers across Alipay (VLDB paper) |
| 18 | Double 11 "610,000 tps" | ⚠ | Widely reported Alipay peak (2021 vintage); per-year figures vary — treat as widely reported |
| 19 | ICBC on OceanBase | ✅ | Announced September 2020 (also verified in sibling guide §5.4) |
| 20 | OceanBase spin-off from Ant ~2021, funding/IPO | ⚠ | OceanBase Inc. incorporated 2021 (Ant subsidiary); first external financing sought per SCMP (2023); no confirmed valuation or IPO plans as of Aug 2026 — treat specific figures as unverified |
| 21 | OceanBase international/global | ✅ | OB Cloud on AWS (Oct 2022) + GCP + Aliyun; 60+ regions/240+ AZs claimed (company statement); Malaysia support centre (Apr 2026); GO GLOBAL GO (Oct 2025) |
| 22 | China DB market shares (PolarDB/OceanBase vs TDSQL/GaussDB/TiDB) | ⚠ | OceanBase and Alibaba Cloud lead China relational-DB share per IDC China 2024–2025 reporting; exact percentages vary by vintage — verify before quoting |

**Honesty footer:** All product facts above are as of **August 2026** and were verified against vendor documentation, published benchmark results, and industry reporting where possible; items marked ⚠ in the claims table could not be fully verified in this session (search budget limited) and should be re-checked against the official sources — aliyun.com / alibabacloud.com for PolarDB, en.oceanbase.com for OceanBase, tpc.org for benchmark results — before any procurement or architecture decision. Benchmark numbers measure specific audited configurations, not your workload. This guide is analysis, not an endorsement.


