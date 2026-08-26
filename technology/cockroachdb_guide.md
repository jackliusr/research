# CockroachDB — the Distributed SQL Database

**The Distributed-SQL/NewSQL Deep-Dive — the Overview (Distributed SQL, NewSQL, the SQL-vs-NoSQL Spectrum), the History (the 2015 Founding, the Ex-Google Engineers, the Spanner Heritage, the Licensing Pivot), the Architecture (Nodes, Ranges, Replicas, Raft Consensus, Leaseholders), the SQL Layer (PostgreSQL Wire Compatibility, Distributed Execution), the Transactions (Serializable Isolation, Parallel Commits, Commit-Wait), the Multi-Region Features (Regional Tables, Survival Goals, Data Domiciling), the Data Features (CDC Changefeeds, Point-in-Time Backups), the Cloud and the Licensing (Dedicated, Serverless, the BSL Journey), the Comparisons (Spanner, TiDB, YugabyteDB, PostgreSQL), a Multi-Region Banking Worked Example, and the One-Page Summary — the Un-Killable Database**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Databases — the DEDICATED distributed-SQL/NewSQL deep-dive of the database cluster. Where [oracle_database_guide.md](oracle_database_guide.md) is the *classical relational* deep-dive, [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) is the *NoSQL modelling* deep-dive and [neo4j_graph_database_guide.md](neo4j_graph_database_guide.md) is the *graph* deep-dive, this guide is the *distributed relational* member of the family: the database that keeps SQL, ACID transactions and joins, and distributes them across a cluster of commodity machines with Raft consensus — the Spanner-heritage category the industry calls distributed SQL / NewSQL. It is written to stand alone and to cross-ref the cluster heavily.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the CockroachDB official documentation (docs.cockroachlabs.com — the Architecture Overview and the Replication/SQL/Transaction layer pages; the Transactions and Serializable-Isolation pages; the Multi-Region Capabilities Overview, Table Localities and Multi-Region Survival Goals pages; the Change Data Capture Overview; the Backup and Restore Overview; the PostgreSQL Compatibility page; the Benchmarking Overview), the Cockroach Labs product pages (cockroachlabs.com — the Cloud product page, the pricing page), the GitHub repository (github.com/cockroachdb/cockroach — stars, forks, commits, language, design docs), Wikipedia's CockroachDB article (the 2015 founding, the founders' Google background, the 2019 Apache→BSL relicensing, the press lineage) and the press citations it carries (Wired, the Wall Street Journal, VentureBeat), the Tracxn funding record, and the repo's own verified cluster (the database series, the sharding guide, the event-streaming guide, the zero-downtime guide). NOTE: this pass had **live web access** (self-hosted Firecrawl backend) — the key claims were verified on 2026-08-24 via web search and direct page extraction of the docs and the repo. Anything that could not be verified is flagged ⚠ honestly.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [Oracle Database](oracle_database_guide.md) (**THE classical-relational companion — the SQL-compatibility angle — cross-ref §4 constantly**), [Oracle Sharding](oracle_sharding_guide.md) (**THE sharding-patterns companion — the horizontal-scale angle — its §on sharding is §3 of this guide in classical clothes; it already cross-refs CockroachDB**), [NoSQL Data Modelling](nosql_data_modelling_guide.md) (**THE SQL-vs-NoSQL-spectrum companion — where this guide's §1.5 sits**), [Neo4j Graph Database](neo4j_graph_database_guide.md) (the database-category map — cross-ref §1.5), [Databricks](databricks_guide.md) and [Cloud Object Storage Lakehouse](cloud_object_storage_lakehouse_guide.md) (the analytical side — light cross-ref §7), [Event Stream Processing](event_stream_processing_guide.md) (**THE streaming companion — cross-ref the changefeed/CDC angle in §7.1 constantly**) and [Kafka Alternatives](kafka_alternatives_guide.md) (the sink/streaming angle), [Zero-Downtime System Design](zero_downtime_system_design_guide.md) (**THE availability companion — cross-ref the multi-region survivability angle in §6 constantly**), [Distributed Auth](distributed_auth_guide.md) (light), [DDIA Study Companion](ddia_study_companion_guide.md) and [Grokking System Design Companion](grokking_system_design_companion_guide.md) (the study-guide genre — cross-ref the Spanner/NewSQL/consensus chapters), [Singapore Data Centres](singapore_data_centres_guide.md) (the DC/DR frame for §10)
> **Companion guides (banking/, prefix `../banking/`):** [NETS Singapore](../banking/nets_singapore_guide.md) and [Singapore Fintech Payments](../banking/singapore_fintech_payments_guide.md) (**THE payments cross-refs — the 24-7, multi-entity, real-time context for §10's banking scenario**), [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) (the trading-day availability angle — light), the bank software-systems series (the estate context for §10)
> **Companion guides (management/, prefix `../management/`):** [Business Case Development](../management/business_case_development_guide.md) (the licensing/cloud-pricing investment angle — light cross-ref in §8 and §10)

---

**How to use this guide:** Section 1 is the overview — what CockroachDB is, what distributed SQL and NewSQL mean, and the overview table (aspect / description). Section 2 is the history — the 2015 founding by the ex-Google engineers, the Spanner heritage, the funding, the licensing pivot, and the history table (year / event / notes). Section 3 is the architecture — nodes, ranges, replicas, Raft consensus, leaseholders, the layer stack, and the architecture table (component / role / notes). Section 4 is the SQL layer — the PostgreSQL wire compatibility, the SQL→KV pipeline, distributed execution, online schema changes, and the SQL table. Section 5 is the transactions — the serializable default, write intents, parallel commits, commit-wait, retries, and the transactions table. Section 6 is the multi-region — regional tables, regional-by-row, global tables, survival goals, super regions, and the multi-region table. Section 7 is the data features — CDC changefeeds and point-in-time backups, and the data table. Section 8 is the cloud and the licensing — Dedicated, Serverless, self-hosted, the BSL journey, and the cloud table. Section 9 is the comparisons — Spanner, TiDB, YugabyteDB, PostgreSQL, and the comparison table (dimension / CockroachDB / alternative). Section 10 is the worked example — a multi-region banking deployment in the house's context. Section 11 is the one-page summary — the final word is **the un-killable database**. The glossary, the claims-status ledger and the cross-references close the file. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`; guides in the `ai_llm/` subdirectory are prefixed `ai_llm/`. **Integrity convention:** ✅ = verified this pass (live web or the cross-referenced guide's ledger); ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [Distributed SQL and NewSQL — the Definitions](#12-distributed-sql-and-newsql--the-definitions)
   - 1.3 [The Overview Table — Aspect / Description](#13-the-overview-table--aspect--description)
   - 1.4 [Reading the Overview Table](#14-reading-the-overview-table)
   - 1.5 [The Database Spectrum — Where CockroachDB Sits](#15-the-database-spectrum--where-cockroachdb-sits)
2. [The History](#2-the-history)
   - 2.1 [The 2015 Founding — Cockroach Labs](#21-the-2015-founding--cockroach-labs)
   - 2.2 [The Spanner Heritage](#22-the-spanner-heritage)
   - 2.3 [The Name and the Mission](#23-the-name-and-the-mission)
   - 2.4 [The Funding and the Growth](#24-the-funding-and-the-growth)
   - 2.5 [The Licensing Pivot — Apache to BSL](#25-the-licensing-pivot--apache-to-bsl)
   - 2.6 [The History Table — Year / Event / Notes](#26-the-history-table--year--event--notes)
3. [The Architecture](#3-the-architecture)
   - 3.1 [The Big Picture — a Database in Layers](#31-the-big-picture--a-database-in-layers)
   - 3.2 [Nodes — the Symmetric Process](#32-nodes--the-symmetric-process)
   - 3.3 [Ranges — the Shards of Keyspace](#33-ranges--the-shards-of-keyspace)
   - 3.4 [Replicas — the Copies](#34-replicas--the-copies)
   - 3.5 [Raft — the Consensus Core](#35-raft--the-consensus-core)
   - 3.6 [Leaseholders — the Single Writer](#36-leaseholders--the-single-writer)
   - 3.7 [The Architecture Table — Component / Role / Notes](#37-the-architecture-table--component--role--notes)
   - 3.8 [A Read and a Write, End to End](#38-a-read-and-a-write-end-to-end)
4. [The SQL Layer](#4-the-sql-layer)
   - 4.1 [The PostgreSQL Wire Compatibility](#41-the-postgresql-wire-compatibility)
   - 4.2 [The SQL-to-KV Pipeline](#42-the-sql-to-kv-pipeline)
   - 4.3 [Distributed Execution](#43-distributed-execution)
   - 4.4 [Online Schema Changes](#44-online-schema-changes)
   - 4.5 [The SQL Table](#45-the-sql-table)
5. [The Transactions](#5-the-transactions)
   - 5.1 [Serializable by Default](#51-serializable-by-default)
   - 5.2 [How a Distributed Transaction Works — Write Intents and Transaction Records](#52-how-a-distributed-transaction-works--write-intents-and-transaction-records)
   - 5.3 [Parallel Commits — and Commit-Wait](#53-parallel-commits--and-commit-wait)
   - 5.4 [Retries — the 40001 Discipline](#54-retries--the-40001-discipline)
   - 5.5 [The Transactions Table](#55-the-transactions-table)
6. [The Multi-Region](#6-the-multi-region)
   - 6.1 [The Multi-Region Vocabulary — Cluster Regions, Database Regions, Localities](#61-the-multi-region-vocabulary--cluster-regions-database-regions-localities)
   - 6.2 [Regional Tables](#62-regional-tables)
   - 6.3 [Regional by Row Tables](#63-regional-by-row-tables)
   - 6.4 [Global Tables](#64-global-tables)
   - 6.5 [Survival Goals — Zone vs Region](#65-survival-goals--zone-vs-region)
   - 6.6 [Super Regions and Data Domiciling](#66-super-regions-and-data-domiciling)
   - 6.7 [The Multi-Region Table](#67-the-multi-region-table)
7. [The Data Features](#7-the-data-features)
   - 7.1 [CDC — Changefeeds](#71-cdc--changefeeds)
   - 7.2 [Backups — Full, Incremental, Scheduled, Point-in-Time](#72-backups--full-incremental-scheduled-point-in-time)
   - 7.3 [The Data Table](#73-the-data-table)
8. [The Cloud and the Licensing](#8-the-cloud-and-the-licensing)
   - 8.1 [CockroachDB Cloud — Dedicated](#81-cockroachdb-cloud--dedicated)
   - 8.2 [CockroachDB Cloud — Serverless](#82-cockroachdb-cloud--serverless)
   - 8.3 [Self-Hosted — and the Deployment Shapes](#83-self-hosted--and-the-deployment-shapes)
   - 8.4 [The Licensing Journey — Apache 2.0, BSL, CCL](#84-the-licensing-journey--apache-20-bsl-ccl)
   - 8.5 [The Cloud Table](#85-the-cloud-table)
   - 8.6 [The Pricing Angle](#86-the-pricing-angle)
9. [The Comparisons](#9-the-comparisons)
   - 9.1 [CockroachDB vs Google Spanner](#91-cockroachdb-vs-google-spanner)
   - 9.2 [CockroachDB vs TiDB](#92-cockroachdb-vs-tidb)
   - 9.3 [CockroachDB vs YugabyteDB](#93-cockroachdb-vs-yugabytedb)
   - 9.4 [CockroachDB vs PostgreSQL](#94-cockroachdb-vs-postgresql)
   - 9.5 [The Comparison Table — Dimension / CockroachDB / Alternative](#95-the-comparison-table--dimension--cockroachdb--alternative)
   - 9.6 [The Rest of the Spectrum — Oracle and NoSQL](#96-the-rest-of-the-spectrum--oracle-and-nosql)
10. [The Worked Example — a Multi-Region Banking Deployment](#10-the-worked-example--a-multi-region-banking-deployment)
    - 10.1 [The Scenario — a Banking Workload](#101-the-scenario--a-banking-workload)
    - 10.2 [The Regional Design](#102-the-regional-design)
    - 10.3 [The Schema Sketch](#103-the-schema-sketch)
    - 10.4 [The Failure Drills](#104-the-failure-drills)
    - 10.5 [The Lessons](#105-the-lessons)
11. [The One-Page Summary — the Un-Killable Database](#11-the-one-page-summary--the-un-killable-database)
12. [The Glossary](#12-the-glossary)
13. [Claims Status and Verification Notes](#13-claims-status-and-verification-notes)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)

---

## 1. The Overview

### 1.1 The Short Answer

**CockroachDB is a source-available, distributed SQL database built by Cockroach Labs** (New York, founded 2015). In one sentence: it is a PostgreSQL-wire-compatible relational database whose relational functionality is built on top of a distributed, transactional, consistent key-value store, replicated across a cluster of commodity machines with Raft consensus, so that it behaves like a single SQL database while being spread across many nodes — often across data centres and cloud regions. It is the flagship of the **distributed SQL** category, which the industry also calls **NewSQL** (✅ Wikipedia categories list CockroachDB under both *Cloud databases* and *NewSQL*; the CockroachDB docs describe the product as "the source-available database... both scalable and consistent" — ✅ architecture-overview page).

The name is the mission: cockroaches are famously hard to kill, and the database was built to survive node failures, zone failures, region failures and even whole-data-centre loss while continuing to serve consistent reads and writes — "the un-killable database" (the press of the day described it as "almost impossible to take down" — ✅ Wired via Wikipedia's citation lineage).

Three facts anchor everything that follows:

1. **It is a SQL database.** Most of the ANSI SQL standard, joins, constraints, foreign keys, ACID transactions, `BEGIN`/`COMMIT`, indexes — the full relational experience, over the PostgreSQL wire protocol so that PostgreSQL drivers, ORMs and tools work largely unchanged (✅ docs, §4.1).
2. **It is a distributed database.** Data is automatically split into ranges and replicated across nodes; every node can accept reads and writes; the cluster rebalances itself; and consensus (Raft) — not a master/slave arrangement — decides what is committed (✅ docs, §3).
3. **It is consistent.** Serializable isolation by default, no stale reads from the leaseholder, and multi-region data placement you can control with SQL — the Spanner-heritage consistency model without Spanner's special hardware (✅ docs, §5–§6).

### 1.2 Distributed SQL and NewSQL — the Definitions

**Distributed SQL** (✅-structural — the term is widely used across vendor and analyst literature, with CockroachDB and Spanner as its canonical exemplars) is the category of databases that offer the full relational model — SQL, ACID transactions, joins — while scaling horizontally across many nodes like a NoSQL store. The defining properties, as the category has settled:

| Property | What it means |
| --- | --- |
| **SQL interface** | The relational model and SQL as the primary API — not key-value or document APIs bolted on |
| **ACID transactions** | Serializable (or strong) isolation across rows, tables, even regions — not per-key atomicity |
| **Horizontal scale** | Data sharded across many nodes; add nodes to add capacity, automatically rebalanced |
| **Consensus replication** | Copies of data kept consistent by a consensus protocol (Raft/Paxos), not by async replication |
| **Elasticity** | Nodes can join and leave; no manual sharding or resharding by the application |

**NewSQL** (✅-structural; the term was coined by 451 Research's Matthew Aslett around 2011) is the older umbrella label for "the new generation of SQL databases that keep the relational model and ACID while scaling like NoSQL" — a reaction to the early-2010s orthodoxy that scale required abandoning SQL. The category's founding references were Google Spanner (2012) and the VoltDB/H-Store academic lineage; CockroachDB is one of the purest expressions of the NewSQL thesis because it rebuilt the entire stack — storage, replication, transactions, SQL — from the ground up for distribution rather than layering distribution onto a single-node engine. The repository's own [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) covers the other side of the spectrum; this guide covers the answer to it.

### 1.3 The Overview Table — Aspect / Description

| Aspect | Description |
| --- | --- |
| **What it is** | A distributed SQL (NewSQL) database: a relational database over a distributed, transactional, consistent key-value store (✅ Wikipedia: "relational functionality is built on top of a distributed, transactional, consistent key-value store") |
| **Category** | Distributed SQL / NewSQL; source-available software (✅ Wikipedia categories: NewSQL, Cloud databases, Distributed data stores) |
| **Company** | Cockroach Labs, Inc. — New York City, founded 2015 by Spencer Kimball, Peter Mattis, Ben Darnell (✅ Wikipedia) |
| **SQL compatibility** | PostgreSQL wire protocol v3.0 and the majority of PostgreSQL syntax; works with most PostgreSQL drivers, ORMs and tools (✅ docs §4.1) |
| **Consistency model** | Serializable isolation by default — the strongest SQL-standard level; linearizable writes, no stale reads from the leaseholder (✅ docs §5.1) |
| **Replication** | Raft consensus per range; 3 replicas by default; quorum commits; (RF−1)/2 tolerated failures (✅ docs §3.5) |
| **Scalability model** | Data automatically split into ranges and rebalanced across nodes; symmetric nodes — any node serves any request; no manual sharding (✅ docs §3.2–§3.3) |
| **Multi-region** | SQL-level locality control: regional tables, regional-by-row, global tables; zone and region survival goals; super regions for data domiciling (✅ docs §6) |
| **Data features** | CDC changefeeds (Kafka, cloud storage, webhook sinks), full/incremental/scheduled backups, point-in-time restore with revision history (✅ docs §7) |
| **Deployment** | Self-hosted (bare metal, VMs, containers, Kubernetes) or CockroachDB Cloud (Dedicated single-tenant, Serverless usage-based) (✅ docs/product §8) |
| **Language** | Written in Go (~119k commits, 32.4k GitHub stars, 4.1k forks as of the 2026-08-24 pass — ✅ GitHub) |
| **License** | Apache License 2.0 until mid-2019; Business Source License (BSL) since CockroachDB 19.2; enterprise features under the CockroachDB Community License (✅ Wikipedia + press; §8.4) |
| **Position in the estate** | The horizontal-scale relational answer to the classic single-node relational (Oracle, PostgreSQL) and the sharded-relational (Oracle Sharding) patterns; the consistency answer to NoSQL's eventual consistency |

### 1.4 Reading the Overview Table

The row to read twice is **the consistency row**. The entire product thesis is that you should not have to choose between the relational/ACID experience and the scale/availability of a distributed system. CockroachDB's wager — inherited from Spanner — is that with consensus + carefully managed clocks you can have both, and that the operational cost (extra replicas, network hops on commits, retry discipline in applications) is worth the simplification of everything above the database. Every later section of this guide is a footnote to that row: §3 is *how* the replication works, §5 is *how* the transactions stay correct while distributed, §6 is *what it costs* when the replicas span continents.

### 1.5 The Database Spectrum — Where CockroachDB Sits

The repository's database cluster maps the space: [oracle_database_guide.md](oracle_database_guide.md) is the classical single-node relational; [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) is the NoSQL side (eventual consistency, flexible schema, horizontal scale by design); [neo4j_graph_database_guide.md](neo4j_graph_database_guide.md) is the specialised-graph corner; [oracle_sharding_guide.md](oracle_sharding_guide.md) is the classical database's attempt to scale horizontally by manual sharding. CockroachDB occupies the quadrant that historically was empty: **horizontal scale + relational model + strong consistency**. It does not fit the old trade-off triangle — you give up neither consistency (like NoSQL) nor the relational model (like most scale-out stores), and the price you pay is paid in infrastructure (more replicas), latency physics (quorum across the network) and application discipline (transaction retries). It is also, importantly, **not** a cloud-only product: unlike Spanner, it is software you can run anywhere — which is precisely what the licensing section (§8.4) made legally complicated for the cloud vendors who wanted to run it for you.

---

## 2. The History

### 2.1 The 2015 Founding — Cockroach Labs

**Cockroach Labs was founded in 2015 by three ex-Google engineers: Spencer Kimball, Peter Mattis and Ben Darnell** (✅ Wikipedia, ✅ the company's own About page: "Cockroach Labs was founded in 2015"). The pedigrees matter:

- **Spencer Kimball** and **Peter Mattis** had been key members of the **Google File System (GFS)** team at Google (✅ Wikipedia, citing Wired's 2012 "Colossus" article). Kimball had earlier co-created **GIMP**, the GNU Image Manipulation Program, while at university (✅ Spencer Kimball's Wikipedia article) — an early signal of the open-source instinct.
- **Ben Darnell** had been a key member of the **Google Reader** team (✅ Wikipedia, citing TechCrunch 2009).
- All three had used Google's internal database systems — **Bigtable** and its successor **Spanner** — while at Google, and after leaving they wanted to build something in that family for the outside world (✅ Wikipedia: "After leaving Google, they wanted to design and build something similar").

The chronology of the project itself predates the company: Kimball wrote the first iteration of the design in **January 2014** and began the open-source project on GitHub in **February 2014** (✅ Wikipedia). The company followed in 2015. Development on GitHub attracted substantial early contributions — enough to win the **Open Source Rookie of the Year** award from Black Duck Software in early 2015 (✅ Wikipedia, citing Wired).

### 2.2 The Spanner Heritage

CockroachDB is, openly and by design, **the Spanner model built without Spanner's prerequisites**. Google's **Spanner** (published as *"Spanner: Google's Globally-Distributed Database"*, OSDI 2012) is a globally distributed, externally consistent database that uses **Paxos** for replication, **two-phase commit** across participants, and — crucially — **TrueTime**, a time-synchronisation service built on GPS receivers and atomic clocks, to assign globally meaningful commit timestamps. Spanner's consistency guarantees rest on that special hardware.

CockroachDB's founders wanted the same externally consistent, geo-replicated, transactional database for everyone else — but refused to depend on atomic clocks. Their answer, designed from the start, was a **hybrid logical clock (HLC)** that captures both physical time and causality, combined with a **commit-wait / uncertainty-interval** discipline that makes serializable behaviour provable without TrueTime (✅ the company's own design writing — Spencer Kimball's author page describes the Spanner basis and its "atomic clocks and GPS clocks" inspiration; the design-document lineage "Living Without Atomic Clocks" is in the repo's key-documents list). The trade-off is that CockroachDB's transaction histories are not strictly linearizable in the Spanner sense, but provide **external consistency for transactions with overlapping keys** — the guarantee applications actually rely on (✅ Wikipedia's discussion of the consistency model and its cited sources).

The heritage shows in the architecture (§3): ranges + replicas + consensus + a transaction protocol over a KV store is exactly the Spanner shape. What CockroachDB changed: commodity clocks instead of atomic clocks, Raft instead of Paxos (simpler to implement and explain), **parallel commits** to avoid the classic commit-wait latency penalty of Spanner-style two-phase commit (✅ transaction-layer docs, §5.3), and open source instead of a Google-internal service.

### 2.3 The Name and the Mission

The cockroach is the deliberate mascot: a blast-resistant organism that survives nearly anything. The founders' branding was intentional — "the idea being a database engine built to be blast-resistant and virtually indestructible" (✅ The Stack's interview with Spencer Kimball). The press reception at launch leaned heavily on this: Wired's 2014 coverage was literally titled "Out in the Open: Ex-Googlers Building Cloud Software That's Almost Impossible to Take Down" (✅ via Wikipedia's citation list), and InformationWeek called it "CockroachDB: Ultimate in Database Survival" (✅, June 2015). The mission statement in the docs is more sober: an always-on database that accepts reads and writes on all nodes, offers industry-leading consistency at massive scale, is low-touch for operators and simple for developers, and runs anywhere (✅ architecture-overview page).

### 2.4 The Funding and the Growth

The funding record, as verified this pass (✅ WSJ via Wikipedia for the seed; ✅ Tracxn for the total):

- **June 2015** — $6.3M seed round, led by Benchmark's **Peter Fenton** (✅ WSJ "CockroachDB Scampers Off With $6.3 Million"; ✅ VentureBeat "Peter Fenton's latest investment is a database startup called Cockroach"). The Wired headline of the same day: "Ex-Googlers Get Millions to Help You Build the Next Google."
- **Through 2021** — a total of **~$633M across 7 rounds**, ending with a **$278M Series F in December 2021** (✅ Tracxn). Investors across the arc included Benchmark, Index Ventures, Redpoint, Altimeter Capital, GV, Tiger Global and others (⚠-structural — the full investor list varies by aggregator).
- **Valuation** — ⚠ reported figures vary by source and point in time: Tracxn records ~$5B (post-Series-F), WOWLS ~$2B, secondary-market trackers ~$2.4B. The valuation trajectory is real; the precise number depends on the source and the date. Flagged rather than asserted.
- **Scale/impact metrics** — ⚠ specific customer counts and revenue figures were not verified this pass (they change constantly and are marketing-reported); what is verifiable is the product's engineering footprint: ~119,000 commits, 32.4k stars and 4.1k forks on GitHub as of 2026-08-24 (✅ GitHub page, read live this pass).

### 2.5 The Licensing Pivot — Apache to BSL

CockroachDB launched under the **Apache License 2.0** — a genuine open-source licence. In **June 2019** the company announced it would **relicense the database under the Business Source License (BSL)** — the source-available licence MariaDB had pioneered — with **CockroachDB 19.2** as the first BSL release (✅ Wikipedia; ✅ OpenSourceForU, June 2019: "Cockroach Labs Adopts Permissive Version of Business Source License"; ✅ SD Times' contemporaneous coverage; ✅ the founders' own post "Why We're Relicensing CockroachDB", June 4, 2019). The BSL forbids offering a commercial version of CockroachDB **as a service** (the "cloud-provider loophole": AWS or others packaging the OSS database into a managed offering) without buying a licence, while remaining free for community use. The stated motivation was explicitly defensive — the same wave that pushed MongoDB (SSPL) and Redis (RSAL) to restrict their licences (✅-structural; the SD Times coverage quotes OSI weighing in on the industry conflict). The enterprise features later moved under Cockroach Labs' own **CockroachDB Community License (CCL)** (✅ It's FOSS's summary of the sequence; ⚠ details of exactly which features sit under which licence change between versions — check the current licensing FAQs). The practical consequence for this repo's audience: **CockroachDB is source-available, not OSI open source, since mid-2019**, and any vendor wanting to run it as a managed service for third parties needs a commercial agreement — a consideration that matters for the banking-estate procurement angle (§8.4, §10).

### 2.6 The History Table — Year / Event / Notes

| Year | Event | Notes |
| --- | --- | --- |
| **2012** | Google publishes Spanner (OSDI 2012) | The model CockroachDB would later "democratise" — Paxos + 2PC + TrueTime atomic clocks (✅-structural; the paper is canonical) |
| **Jan 2014** | Spencer Kimball writes the first CockroachDB design | Pre-company; the design was written before the code (✅ Wikipedia) |
| **Feb 2014** | The project is open-sourced on GitHub | Apache License 2.0; "CockroachDB: A Scalable, Geo-Replicated, Transactional Datastore" appears on InfoQ the same year (✅ Wikipedia/InfoQ citation) |
| **2014** | Wired: "almost impossible to take down" | The cockroach framing goes mainstream (✅ via Wikipedia citations) |
| **2015** | **Cockroach Labs is founded** — Kimball, Mattis, Darnell | Ex-Google: GFS team (Kimball, Mattis), Reader team (Darnell) (✅ Wikipedia) |
| **Jun 2015** | $6.3M seed from Benchmark (Peter Fenton) | WSJ, Wired, VentureBeat same-day coverage (✅ WSJ via Wikipedia) |
| **2015** | Black Duck "Open Source Rookie of the Year" | Early community momentum (✅ Wikipedia, citing Wired) |
| **2017** | First public release (v1.0 era) | Wikipedia dates the first release to 2017 (✅ Wikipedia infobox) |
| **Jun 2019** | **Relicensed: Apache 2.0 → Business Source License (BSL)** | 19.2 first BSL release; the anti-cloud-provider pivot, same wave as MongoDB SSPL and Redis RSAL (✅ Wikipedia + press) |
| **2019–2021** | Series rounds through **Series F ($278M, Dec 2021)** | Total ~$633M over 7 rounds (✅ Tracxn; ⚠ valuation figures vary: ~$2B–$5B by source) |
| **2021** | Non-voting replicas (v21.1) — the multi-region read story matures | Read-only replicas that follow the Raft log without participating in quorum (✅ replication-layer docs) |
| **2022–2025** | READ COMMITTED isolation; multi-region SQL abstractions mature; Cloud Serverless/Dedicated; enterprise features under CCL | The product surface this guide describes (✅ docs; ⚠ exact version-to-feature mapping varies) |
| **2026** | v26.x current; 32.4k stars, ~119k commits | As observed live on 2026-08-24 (✅ GitHub) |

---
## 3. The Architecture

### 3.1 The Big Picture — a Database in Layers

CockroachDB's architecture is a clean layer stack, each layer treating the one below as a black-box API (✅ architecture-overview page — the layer table). From the top:

| Layer | Order | Purpose (as the docs state it) |
| --- | --- | --- |
| **SQL** | 1 | Translate client SQL queries into KV operations |
| **Transactional** | 2 | Allow atomic changes to multiple KV entries |
| **Distribution** | 3 | Present replicated KV ranges as a single entity |
| **Replication** | 4 | Copy data between nodes; ensure consistency via consensus (Raft) |
| **Storage** | 5 | Write and read data to/from disk, keeping track of timestamps (MVCC) |

The mental model that makes everything else fall into place: **the whole cluster is one giant sorted key-value map, split into contiguous chunks (ranges), each chunk replicated with Raft, and the SQL layer presents that KV map to you as tables, rows and columns.** SQL never touches a single node's storage directly; the SQL layer compiles your query into KV operations, the transaction layer makes them atomic, the distribution layer finds the right ranges, the replication layer gets quorum, and the storage layer persists with MVCC timestamps (which is what powers `AS OF SYSTEM TIME` historical reads — ✅ architecture overview).

### 3.2 Nodes — the Symmetric Process

A **node** is one running `cockroach` process, typically one per machine. The critical property is **symmetry**: every node is identical in capability, every node can accept SQL from clients (the node you connect to becomes the *gateway* for your request), and no node is the "master" of the cluster (✅ docs: "the symmetrical behavior of all nodes in a cluster"; you can "send SQL requests to any node; this makes CockroachDB easy to integrate with load balancers"). The cluster forms when nodes join via `cockroach start --join=<other nodes>` and a one-time `cockroach init` (✅ docs). Nodes can be labelled with **locality** information (`--locality=region=us-east-1,zone=us-east-1b`) — the hook that the multi-region features (§6) hang on. Add nodes and the cluster automatically rebalances data onto them; lose nodes and the cluster automatically re-replicates to restore survivability (✅ replication-layer docs).

### 3.3 Ranges — the Shards of Keyspace

The KV map is divided into contiguous spans of keys called **ranges** — each range is the unit of both replication and movement. When a range grows past its configured size, it **splits** into two; the ranges are then placed so that load and data spread evenly across nodes (✅ distribution-layer docs, referenced from the architecture overview: "breaking the data up into chunks that we call ranges"; "if a table and its indexes grow too large for a single range, it is split into two ranges"). Range size is bounded by zone-configurable thresholds (⚠ the exact default thresholds have changed across versions — historically ranges split around 64 MiB; current versions allow larger ranges, on the order of hundreds of MiB to ~512 MiB maximum; the precise default is version-dependent and not asserted here).

The key architectural consequence: **a table is not stored on one node.** A table is a set of keys (one key per row, plus one key-space per index); those keys fall into many ranges; each range lives on several nodes. "Sharding" is therefore automatic, invisible, and continuous — the application never specifies a shard key, and the cluster reshards itself as it grows. This is the direct descendant of Spanner's *tablet* model and the thing that distinguishes CockroachDB from manual sharding (cross-ref [oracle_sharding_guide.md](oracle_sharding_guide.md) — where sharding is a design activity the DBA performs, here it is a background process of the engine).

### 3.4 Replicas — the Copies

Each range is replicated: by default **3 replicas on 3 different nodes** (✅ docs; controlled by replication zones at cluster/database/table level). The replicas of one range form a **Raft group**. Two replica types matter (✅ replication-layer docs):

- **Voting replicas** — participate in Raft elections and quorum; the number of failures a range can survive is **(replication factor − 1) / 2** — e.g., 3x replication tolerates 1 failure, 5x tolerates 2. This is why "high availability requires 3 nodes" — 3 is the smallest number that can achieve a quorum of 2.
- **Non-voting replicas** (added in v21.1) — follow the Raft log and can serve *follower reads* locally, but do not vote, so they add read capacity and cross-region read locality **without** adding write-latency to quorum (✅ docs: "They have almost no impact on write latencies"). These are the mechanism behind fast multi-region reads (global tables, §6.4) and are the same idea as Spanner's *read-only replicas*.

When a node dies, the cluster notices, and the ranges whose quorum is intact automatically re-replicate onto surviving nodes — "CockroachDB automatically realizes nodes have stopped responding and works to redistribute your data to continue maximizing survivability" (✅ replication-layer docs). This self-healing is the availability core that the repo's [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) discusses as *quorum* in the abstract; here it is concrete machinery.

### 3.5 Raft — the Consensus Core

**Raft is the consensus protocol that makes the replicas agree** (✅ replication-layer docs; cross-ref the Raft/consensus concepts in [ddia_study_companion_guide.md](ddia_study_companion_guide.md) and [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §5.3). The mechanics, as implemented per range:

- Each range's replicas form a **Raft group**; each replica is a **leader** or a **follower** (a third role, the non-voting replica, participates in neither election nor quorum).
- All writes are proposed to the **leader**, appended to the **Raft log**, and **committed only when a majority (quorum) of voting replicas has the entry** — for 3 replicas, 2 must agree. Followers who miss entries are caught up by replaying the log; a replica that fell behind can be rebuilt from a snapshot.
- The leader **heartbeats** followers; if followers stop hearing from it, randomized **election timeouts** trigger a new election. The randomisation is what prevents election ties (✅ docs: the election timeout is "multiplied by a random factor of 1–2 to avoid election ties").
- The Raft log is the range's **source of truth for consistent replication**: "an ordered set of commands that the replicas agreed on... because this log is treated as serializable, it can be replayed to bring a node from a past state to its current state" (✅ replication-layer docs).

Raft is per-range, not per-cluster: a 30-node cluster runs thousands of independent Raft groups, one per range. This is the scalability trick — consensus is kept local to each chunk of data, so the protocol cost doesn't grow with the cluster. It is also the reason the docs can promise that "data is safely stored on multiple nodes and those nodes agree on the current state even if some of them are temporarily disconnected" (✅ architecture overview — Raft protocol definition).

### 3.6 Leaseholders — the Single Writer

For each range, one replica holds the **lease** — the **leaseholder** (✅ architecture overview + replication-layer docs):

- The leaseholder **receives and coordinates all read and write requests for the range**; it is "the only replica that can serve consistent reads (reads that return 'the latest' data)" for most table types.
- The leaseholder is **the same replica as the Raft leader** (except briefly during lease transfers) — the docs' *leader leases* arrangement collapses the "who writes" and "who reads-consistently" roles into one node per range.
- Leases are **epoch-based** for ordinary ranges (tied to the node's liveness; they don't expire while the node is healthy) with **expiration-based leases** reserved for meta and system ranges (✅ replication-layer docs — the lease-type split).
- When the leaseholder fails, the lease is transferred/acquired by another replica and service continues — the liveness machinery detects the dead node, and per-replica **circuit breakers** fail requests to temporarily unavailable ranges fast instead of hanging (✅ replication-layer docs).

The leaseholder concept is the bridge between the consensus layer and performance: consensus decides *what* is committed; the lease decides *where* the consistent reads and writes are served from, which is what lets data have a "home" — and the multi-region feature set (§6) is essentially a set of dials for *where leaseholders and voting replicas sit*.

### 3.7 The Architecture Table — Component / Role / Notes

| Component | Role | Notes |
| --- | --- | --- |
| **Node** | One `cockroach` process (one machine); the symmetric unit of the cluster | Any node serves any request; locality labels (`region`/`zone`) attach geography (✅ §3.2) |
| **Range** | A contiguous span of the KV keyspace; the unit of data distribution, replication and movement | Auto-splits when large; auto-rebalances across nodes; "sharding without a shard key" (✅ §3.3) |
| **Replica** | A copy of a range on a node | 3 by default (voting); tolerates (RF−1)/2 failures; non-voting replicas (v21.1+) serve local reads without quorum cost (✅ §3.4) |
| **Raft group** | The set of replicas of one range | Thousands of independent groups — consensus stays local to each range (✅ §3.5) |
| **Raft leader** | The replica that coordinates writes for the group | Proposes to the Raft log; commits on majority; heartbeats followers; elections on failure (✅ §3.5) |
| **Raft log** | The time-ordered, agreed log of writes per range | The source of truth for consistent replication; replayable for catch-up (✅ §3.5) |
| **Quorum** | Majority of voting replicas agreeing to commit | 2-of-3 default; the (RF−1)/2 failure-tolerance math (✅ §3.4–§3.5) |
| **Leaseholder** | The replica holding the range lease — serves consistent reads and coordinates writes | Same replica as the Raft leader (leader leases); epoch-based by default; where "home region" is pinned (✅ §3.6) |
| **Gateway node** | The node a client connects to | It routes/executes the request across the cluster — symmetry in action (✅ §3.2, §4.3) |
| **Storage engine** | The MVCC key-value store on disk | Keeps timestamps per value → `AS OF SYSTEM TIME`, follower reads, changefeeds, PITR all lean on it (✅ §3.1) |

### 3.8 A Read and a Write, End to End

**A write:** your app sends `UPDATE accounts SET balance = balance - 100 WHERE id = 42` to any node (the gateway). The SQL layer parses, plans and compiles it into KV operations; the transaction layer turns the write into a *write intent* (§5.2); the distribution layer locates the range(s) holding account 42's keys; the write intent goes to the range's **leaseholder**, which proposes it to the **Raft group**; a **majority** of replicas append it to their Raft logs; the commit is acknowledged; followers apply it. If the range's leaseholder is on a different node than the gateway, the gateway forwards to the leaseholder — you never need to know where the data lives (✅ architecture overview: "If a node receives a read or write request it cannot directly serve, it finds the node that can handle the request").

**A read:** consistent reads go to the **leaseholder**, which returns the latest committed value at the transaction's timestamp. Cheaper *follower reads* can read from a local replica at a timestamp no higher than the range's **closed timestamp** — slightly stale, zero cross-region round-trip (✅ transaction-layer docs; the mechanism behind §6.4's global tables). Both paths are serializable-consistent under the hood; only the freshness/performance trade differs.

---

## 4. The SQL Layer

### 4.1 The PostgreSQL Wire Compatibility

CockroachDB's SQL surface is deliberately PostgreSQL-shaped (✅ PostgreSQL Compatibility page, extracted live this pass):

- **Wire protocol:** it supports **version 3.0 of the PostgreSQL wire protocol (pgwire)** — the protocol that PostgreSQL clients, drivers and tools speak. This is what makes "existing applications built on PostgreSQL can often be migrated to CockroachDB without changing application code" (✅ docs).
- **Drivers and ORMs:** "CockroachDB also works with most PostgreSQL drivers and ORMs" (✅ docs) — JDBC, psycopg, libpq, GORM, Hibernate, etc. — plus tools such as DBeaver and IntelliJ.
- **Version surface:** it reports PostgreSQL **18** in `server_version`, and the `pg_catalog` system catalogs are aligned with the PostgreSQL 18 catalogs; startup status parameters (`search_path`, `default_transaction_read_only`, `in_hot_standby`, `scram_iterations`) are sent as drivers expect (✅ docs). Because CockroachDB has no primary/standby distinction, `in_hot_standby` is always `off`.
- **The honest caveat:** the compatibility is *broad, not total*. The docs maintain an explicit, versioned list of differences — unsupported features include PostgreSQL **range types**, **events**, **dropping a primary key** (every table must have one), **XML functions**, **column-level privileges**, **XA syntax**, **foreign data wrappers**, and session-scoped advisory locks (transaction-scoped ones are supported); there are also behavioural differences (e.g., `INT` is always 64-bit, bit-shift widths differ, `INSERT ON CONFLICT` CHECK-validation timing differs) (✅ docs, v26.3 list). The message for the estate: **porting a PostgreSQL application is usually a migration, not a rewrite — but the differences list must be read before committing** (cross-ref the SQL-compatibility angle of [oracle_database_guide.md](oracle_database_guide.md)).

### 4.2 The SQL-to-KV Pipeline

The SQL layer is itself a pipeline of sublayers (✅ architecture/sql-layer page):

1. **SQL API** — the user interface; the client connection over pgwire.
2. **Parser** — SQL text → abstract syntax tree (AST), via a `yacc` grammar describing the supported syntax.
3. **Cost-based optimizer** — AST → optimized *logical* query plan (join ordering, index selection, etc.).
4. **Physical planner** — logical plan → *physical* plan for execution across one or more nodes.
5. **SQL execution engine** — executes the plan (vectorized, columnar execution in modern versions ⚠-structural — the vectorized execution engine has been the norm since v21.x) by issuing KV read/write requests down to the transaction layer.

The relational structure is real: tables of rows and columns, databases, constraints, foreign keys — "application developers can trust that the database will ensure consistent structuring of the application's data" (✅ SQL-layer docs). The trick is that the relational surface is *compiled* onto the KV store: every row is a key, every index is a key-space, and the cost-based optimizer chooses how to satisfy your SQL with as few KV touches as possible. A query that in PostgreSQL reads a heap and an index here reads a set of range spans — the planner's job is to make those spans cheap.

### 4.3 Distributed Execution

Because any node can be a gateway and the data is spread across ranges on many nodes, **query execution itself is distributed** (✅ SQL-layer docs: the physical planner produces "a physical query plan for execution by one or more nodes in the cluster"). A `SELECT` with a filter on a big table is planned so that each participating node processes the ranges it holds locally (local scan + filter), and results are streamed/merged back to the gateway — the classic shared-nothing *parallel query* shape. The gateway node coordinates; the plan is a tree whose leaves touch local data and whose interior nodes aggregate. This is why CockroachDB can execute joins and aggregations across tables that physically live on different nodes — and why single-row point lookups (the banking hot path) are optimized to touch exactly one range and one leaseholder, keeping them fast. The `EXPLAIN`/`EXPLAIN (VEC)` output shows the distributed plan; the DB Console shows the per-node execution stats (✅-structural, from the docs' monitoring surface).

### 4.4 Online Schema Changes

Schema changes — `ALTER TABLE ADD COLUMN`, `CREATE INDEX`, and the rest — run **online**: the table keeps serving reads and writes while the change progresses (✅ SQL-layer docs). The mechanism is a **staged schema-change protocol** that decomposes each change into incremental versions and enforces "at most two successive versions of this schema used in the cluster at all times" to keep the cluster consistent — an approach the docs explicitly attribute to Google's *F1* paper, "Online, Asynchronous Schema Change in F1" (✅ docs). For the banking estate this is the difference between a maintenance window and a continuous operation: adding an index to a 24-7 table is a background job, not an outage (cross-ref the zero-downtime data-layer discussion in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §4 — expand-contract migrations are still your application-level pattern, but the schema engine does its part natively).

### 4.5 The SQL Table

| SQL-layer aspect | What it is | Notes |
| --- | --- | --- |
| **Wire protocol** | PostgreSQL wire protocol v3.0 (pgwire) | Most PostgreSQL drivers/ORMs/tools work unchanged (✅ §4.1) |
| **SQL dialect** | PostgreSQL-flavoured; most of ANSI SQL | Explicit, versioned differences list; `INT` always 64-bit; no range types, no FDWs (✅ §4.1) |
| **System catalogs** | `pg_catalog` aligned with PostgreSQL 18 | Reports `server_version` 18; `in_hot_standby` always off (✅ §4.1) |
| **Pipeline** | Parser (yacc→AST) → cost-based optimizer → physical planner → execution engine | SQL compiled to KV operations on the distributed store (✅ §4.2) |
| **Distributed execution** | Physical plans run across nodes; local scans + streamed aggregation | Gateway node coordinates; point lookups hit one range (✅ §4.3) |
| **Online schema changes** | Staged, versioned schema transitions (F1-based) | Tables stay online; at most two schema versions live (✅ §4.4) |
| **Historical reads** | `AS OF SYSTEM TIME` via MVCC timestamps | Follower reads and PITR lean on the same mechanism (✅ §4.4/§3.1) |
| **Language coverage** | SQL, PL/pgSQL-style functions ⚠-partial, triggers ⚠ (not supported in the same form as PostgreSQL) | Check the feature-support matrix per version (✅ docs link) |

---

## 5. The Transactions

### 5.1 Serializable by Default

CockroachDB defaults to **`SERIALIZABLE` isolation** — the strongest of the four SQL-standard isolation levels, "stronger than the SNAPSHOT isolation level developed later" (✅ docs, serializable-demo page: "SERIALIZABLE isolation guarantees that even though transactions may execute in parallel, the result is the same as if they had executed one at a time"). This is a deliberate product statement: most databases default to READ COMMITTED and let you opt up; CockroachDB defaults to serializable and lets you opt *down* to **READ COMMITTED** (supported since v22.2) when your workload needs higher concurrency with fewer retries and can tolerate it (✅ docs, read-committed page: "READ COMMITTED is one of two transaction isolation levels supported on CockroachDB. By default, CockroachDB uses the SERIALIZABLE isolation level"). ACID semantics span "arbitrary tables and rows, even when data is distributed" (✅ transactions page) — a transaction that touches ten rows in three regions either commits everywhere or nowhere, and no other transaction sees a partial state.

### 5.2 How a Distributed Transaction Works — Write Intents and Transaction Records

The mechanism that makes cross-range atomicity possible without a single locking authority (✅ transaction-layer page, extracted live):

- **Write intents:** a write doesn't touch the committed value directly. It lays down a *write intent* — a provisional value that doubles as an exclusive lock, containing a pointer to the transaction's *transaction record*. Intents are replicated through Raft like any other write.
- **Transaction record:** stored in the range of the transaction's first write, carrying the state: `PENDING` → `STAGING` → `COMMITTED`, or `ABORTED`.
- **Conflict resolution:** when a transaction encounters another's intent or lock on a key, it's a *transaction conflict* — resolved by aborting the loser, waiting, or restarting, depending on priority and timestamp. When a transaction encounters a newer committed value than it read, it may be **restarted** with a bumped timestamp — the classic serializable-retry path.
- **Unreplicated locks:** the concurrency-control machinery also keeps in-memory per-node locks (an optimization surface; the *Select for Update* statement is supported for explicit row locking) (✅ transaction-layer docs; ✅ PostgreSQL-compatibility page: "CockroachDB supports the `SELECT FOR UPDATE` statement").

The system is, in the company's own phrase, "**Serializable, Lockless, Distributed**" — reads never block on reads (MVCC gives every transaction a consistent snapshot at its timestamp), and the only blocking is intent-vs-intent contention, resolved by abort/retry rather than lock waits.

### 5.3 Parallel Commits — and Commit-Wait

The classic distributed-commit problem: a transaction spanning multiple ranges must not become visible before all its writes are durable — the two-phase-commit (2PC) answer involves a coordinator and a **commit-wait** (the coordinator waits a bounded period after commit to be sure the commit decision survives coordinator failure — this is precisely the latency Spanner pays, and why Spanner's commit path needs TrueTime to keep the wait short).

CockroachDB's answer is **Parallel Commits** (✅ transaction-layer docs: "it achieves correctness using a distributed, atomic commit protocol called Parallel Commits"): the transaction is moved to `STAGING` when *all* its write intents are durable across the cluster, and the commit is acknowledged as soon as the transaction record reaches `COMMITTED` — because every intent already carries a pointer to the record, no participant can act on a write without first checking the record, so the protocol never needs to wait for a "prepare" round-trip or a commit-wait window in the common case. **Commit-wait does not disappear entirely**: the docs describe it as a targeted fallback — "a transaction that reads a future-time write to this range can have its commit timestamp bumped into the future... Such transactions (a.k.a. 'conflicting readers') may also need to commit-wait" (✅ transaction-layer docs). In short: **parallel commits made the commit-wait rare; the uncertainty window made it correct when it happens.** The same machinery powers **non-blocking transactions** for global tables (§6.4), which write with a timestamp in the future so that cross-region reads never block (✅ table-localities docs).

Two more timestamps matter for correctness: the **closed timestamp** (per range — the watermark below which replicas may serve follower reads) and the **maximum clock offset** (`--max-offset`, 500ms default, 250ms recommended for multi-region) which bounds the uncertainty window that forces commit-wait (✅ transaction-layer + multiregion docs).

### 5.4 Retries — the 40001 Discipline

Serializable isolation in a distributed system means **transactions sometimes lose races and must be retried**. CockroachDB's contract with the application (✅ transactions page):

- **Automatic retries** — single-statement implicit transactions can be retried silently by the server.
- **Client-side retries** — multi-statement transactions must be written against the *advanced client-side retry protocol*: open with `SAVEPOINT cockroach_restart`, and on a **transaction retry error** (SQLSTATE `40001`, message `restart transaction` — meaning the transaction "could not be placed in a serializable ordering"), `ROLLBACK TO SAVEPOINT cockroach_restart` and re-run, then `RELEASE SAVEPOINT` and `COMMIT` (✅ docs, with the exact syntax on the transactions page). Retries happen "only rarely under READ COMMITTED" isolation and under contention at serializable (✅ docs).
- **Ambiguous errors** (`40003`) — the state of the commit is unknown; the application must resolve ambiguity (e.g., by idempotency keys) rather than blindly retry (✅ docs error table).
- For frameworks without retry logic built in, the docs prescribe an application-level retry loop with exponential backoff (✅ docs).

This is the single biggest *application* consequence of choosing CockroachDB: **your transaction code must be retry-aware** — the repo's [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §8 (retries and idempotency) is not optional reading here, it is the programming model.

### 5.5 The Transactions Table

| Transaction aspect | What it is | Notes |
| --- | --- | --- |
| **Default isolation** | SERIALIZABLE — strongest SQL-standard level, stronger than snapshot | Result "the same as if they had executed one at a time" (✅ §5.1) |
| **Alternative isolation** | READ COMMITTED (v22.2+) | For high-concurrency workloads that tolerate it and want fewer retries (✅ §5.1) |
| **Write mechanism** | Write intents (provisional values + exclusive locks) + transaction records (PENDING/STAGING/COMMITTED/ABORTED) | Intent conflicts resolved by abort/restart; MVCC keeps reads lock-free (✅ §5.2) |
| **Commit protocol** | Parallel Commits — STAGING when all intents durable; no coordinator prepare round-trip | The classic 2PC commit-wait avoided in the common case (✅ §5.3) |
| **Commit-wait** | Still occurs for "conflicting readers" — transactions that read future-time writes | Bounded by the clock-offset uncertainty window (✅ §5.3) |
| **Consistency framing** | External consistency for transactions with overlapping keys; not strictly linearizable histories | The Spanner difference, honestly documented (✅ §2.2) |
| **Retry contract** | `40001`/`restart transaction` → `ROLLBACK TO SAVEPOINT cockroach_restart`; `40003` ambiguous | Client-side retry protocol is mandatory for multi-statement transactions (✅ §5.4) |
| **Historical reads** | `AS OF SYSTEM TIME`; follower reads below the closed timestamp | The storage-layer MVCC timestamps made visible in SQL (✅ §3.1) |
| **Locking** | `SELECT ... FOR UPDATE` supported; session-scoped advisory locks unsupported | Differences list (✅ §4.1) |

---

## 6. The Multi-Region

### 6.1 The Multi-Region Vocabulary — Cluster Regions, Database Regions, Localities

Multi-region is where CockroachDB's distributed-DNA shows its full value, and it is controlled in **SQL**, not in config files (✅ multiregion-overview page, extracted live):

- **Cluster region** — a geographic region declared at node startup via `--locality=region=...,zone=...` (e.g., `region=us-east-1,zone=us-east-1b`).
- **Database region** — a region added to a specific database with `ALTER DATABASE ... PRIMARY REGION <r>` (first region) and `ALTER DATABASE ... ADD REGION <r>` (more). A database with a primary region is a *multi-region database*; its data is stored only within its assigned regions.
- **Table locality** — how CockroachDB optimizes access to a table's data: **regional tables** (fast in one home region), **regional by row tables** (per-row home regions), **global tables** (fast reads everywhere).
- **Survival goal** — how many simultaneous failures the database survives: **zone** (default) or **region**.

The design flow the docs recommend is: set node localities → set the database's primary region → optionally tune table localities → optionally raise the survival goal (✅ multiregion-overview). Underneath, these SQL statements compile down to replication-zone placements of leaseholders, voting replicas and non-voting replicas — the multi-region abstractions are the user-friendly face of the zone-config machinery (✅ docs: "most users should control non-voting replica placement with the high-level multi-region SQL features instead" of raw zone configs).

### 6.2 Regional Tables

A **regional table** (default locality for multi-region databases) homes the whole table in **one region**: the leaseholders of its ranges — and the voting replicas required by the survival goal — are placed in that home region, so reads and writes from that region are local, and access from other regions is slower (✅ table-localities docs: "access to the table will be fast in the table's home region and slower in other regions"). The home region defaults to the database's primary region but can be set per table with `ALTER TABLE ... SET LOCALITY REGIONAL BY TABLE IN <region>`. This is the right choice for data with a natural single home — a bank's Singapore-ledger tables homed in `asia-southeast1`, for example (§10).

### 6.3 Regional by Row Tables

A **regional by row** table homes **each row** in a region: every row carries a hidden `crdb_region` column (defaulting to the gateway node's region at insert time), and the table and all its indexes are **automatically partitioned by `crdb_region`** as the partition-key prefix (✅ table-localities docs). Rows are co-located with the users/accounts that access them — the canonical example is a `users` table where European users' rows live in Europe and Asian users' in Asia. Supporting machinery: **locality-optimized searches** (the optimizer looks in the gateway's region first, only fanning out if the local rows don't satisfy the query) and **auto-rehoming** (an `UPDATE`/`UPSERT` can move a row's home to the writer's region automatically) (✅ docs). The bank-estate version: a *customer* table where each customer's row is homed in their regulatory jurisdiction.

### 6.4 Global Tables

A **global table** is optimized for **low-latency reads from all regions** (✅ table-localities docs). The mechanism is the non-blocking-transaction protocol: writes carry a timestamp "in the future" (within `--max-offset`), which lets every region serve consistent reads locally without cross-region coordination — at the price of higher write latency and the `--max-offset` dependency (✅ docs; this is why multi-region clusters are advised to set `--max-offset=250ms`). The intended use is **read-mostly reference data** — currency tables, holiday calendars, instrument masters, branch directories — rarely updated, needed everywhere. In §10's banking example, the reference-data tables are global.

### 6.5 Survival Goals — Zone vs Region

The **survival goal** is the availability promise, expressed per database (✅ multiregion-survival-goals page, extracted live):

- **SURVIVE ZONE FAILURE (default)** — the database stays fully available for reads and writes if a *zone* (cloud availability zone) goes down; it may not survive multiple zones failing in the same region. Cheap: replicas can stay within a region.
- **SURVIVE REGION FAILURE** — the database stays fully available even if an **entire region** goes down. The cost is explicit and documented: "write latency will be increased by at least as much as the round-trip time to the nearest region. Read performance will be unaffected." Requirements: at least 3 database regions; setting the goal on a 3-region cluster automatically raises the replication factor of the database's ranges **from 3 to 5**, spread 2+2+1 across the three regions, so that any one region's loss leaves a quorum (2 of the remaining 3 voters) intact (✅ docs — the exact 2+2+1=5 spread is stated). Writes then need coordination across 2 of the 3 regions.

The mental model for the estate: **zone survival is DR-within-a-region; region survival is DR-across-regions, and you pay for it in write latency** — the classic availability-vs-latency trade dressed in SQL (cross-ref the RTO/RPO and quorum discussion in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §5).

### 6.6 Super Regions and Data Domiciling

**Super regions** let you confine replicas of specified data to a subset of regions — the *data domiciling* feature (✅ multiregion-overview docs: "The primary use case for super regions is data domiciling"). A super region is a set of database regions such that all replicas of regional tables homed in it (and rows of regional-by-row tables homed in it) are stored **only** in member regions; a super region with ≥3 regions and a REGION survival goal keeps domiciled data available through one region's failure. The regulatory resonance is immediate for a bank: MAS/SG data that must not leave Singapore, EU data that must not leave the EU — expressed as `ALTER DATABASE ... ADD SUPER REGION` rather than as deployment folklore (✅ docs). The docs carry one honest caveat: system-database ranges are not covered by super-region placement, so cluster-level availability still depends on the system ranges' quorum (✅ docs).

### 6.7 The Multi-Region Table

| Multi-region aspect | What it is | Notes |
| --- | --- | --- |
| **Cluster region** | Region/zone declared at node start (`--locality=region=...,zone=...`) | The geographic vocabulary of the cluster (✅ §6.1) |
| **Database region** | Region added to a database (`PRIMARY REGION` / `ADD REGION`) | Data stored only in assigned regions (✅ §6.1) |
| **Regional table** | Table homed in one region — fast local, slower remote | Default locality; home region settable per table (✅ §6.2) |
| **Regional by row table** | Per-row home regions via hidden `crdb_region`; auto-partitioned by region | Locality-optimized searches; auto-rehoming; the user/account pattern (✅ §6.3) |
| **Global table** | Low-latency reads everywhere; non-blocking writes with future timestamps | Read-mostly reference data; write latency depends on `--max-offset` (✅ §6.4) |
| **Survival goal — ZONE** | Survives a zone failure; default | Multiple same-region zone failures may take the DB down (✅ §6.5) |
| **Survival goal — REGION** | Survives a full region failure; needs 3+ regions; RF 3→5 spread 2+2+1 | Write latency += round-trip to nearest region; reads unaffected (✅ §6.5) |
| **Super region** | Replica placement confined to a region subset — data domiciling | Regulatory data-residency in SQL; system ranges excluded (✅ §6.6) |
| **Secondary region** | A designated failover region for regional tables | `SET SECONDARY REGION`; incompatible with regional-by-row tables (✅ docs) |
| **Follower reads** | Local reads at/below the closed timestamp | The read-side mechanism under global tables and RBR (✅ §3.6, §6.4) |

---
## 7. The Data Features

### 7.1 CDC — Changefeeds

**Change Data Capture (CDC)** detects row-level changes and emits them as messages for downstream systems — "While CockroachDB is an excellent system of record, CDC allows it to integrate with other systems in your data ecosystem" (✅ change-data-capture-overview page, extracted live). The mechanism is the **changefeed**: a customizable *job* that watches one or more tables and emits every row change in real time to a **sink** (✅ docs):

- **Sinks:** Kafka, Google Cloud Pub/Sub, webhooks, and cloud storage (S3/GCS/Azure); plus **sinkless changefeeds** that stream to the SQL client for prototyping (✅ docs — the sink table).
- **Delivery semantics:** each emitted change is delivered **at least once**, and "the first emit of every event for the same key is ordered by timestamp" (✅ docs) — the classic CDC contract that downstream consumers must deduplicate/idempotently apply (cross-ref [event_stream_processing_guide.md](event_stream_processing_guide.md) and [kafka_alternatives_guide.md](kafka_alternatives_guide.md) — at-least-once and consumer idempotency are the standing themes there).
- **Formats and control:** configurable message formats (including Avro with schema registry — ⚠-structural, the format list is version-dependent), **CDC queries** to filter/transform data before it leaves the database, scheduled changefeeds, execution-locality pinning (emit from the region closest to the sink in multi-region clusters), and monitoring via `SHOW CHANGEFEED JOBS` and the DB Console's changefeed dashboard (✅ docs).
- **Safety:** changefeeds take **protected timestamps** — data being streamed is shielded from garbage collection until it has been emitted (✅ docs: "Protect Changefeed Data from Garbage Collection").
- **Use cases the docs name:** streaming to Kafka for application notifications, mirroring into search/analytics engines, exporting snapshots to backfill new applications, feeding ML feature stores (✅ docs).

For the banking estate, the changefeed is the sanctioned path from OLTP system-of-record to the analytics side — the row-level stream into the lakehouse/analytics cluster (cross-ref [databricks_guide.md](databricks_guide.md) and [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) lightly) — and to the event backbone (cross-ref [event_stream_processing_guide.md](event_stream_processing_guide.md) §CDC). It is also the natural implementation of the transactional-outbox-without-the-outbox: the database's own log is the outbox.

### 7.2 Backups — Full, Incremental, Scheduled, Point-in-Time

The backup surface (✅ backup-and-restore-overview page, extracted live) covers the full DR menu:

- **Full backups** — an un-replicated copy of the cluster, database or table; the base of the chain.
- **Incremental backups** — changes since a base backup (full or full+incremental).
- **Scheduled backups** — periodic, automated; scheduled backups protect their data from GC until the backup succeeds, decoupling backup cadence from the GC TTL.
- **Revision history + point-in-time restore (PITR)** — a backup *with revision history* records every change within the garbage-collection period up to the backup timestamp; **point-in-time restore** then restores "from an arbitrary point in time within the revision history" (✅ docs) — the *undo a bad deployment / dropped table / erroneous batch* feature.
- **Encrypted backups** (KMS or passphrase) and **locality-aware backups** (each node writes to the storage closest to it; locality-restricted execution pins the backup job to chosen nodes) (✅ docs).
- **Operations:** `BACKUP`/`RESTORE` run as **distributed jobs** with `SHOW JOBS`/`PAUSE`/`RESUME`/`CANCEL`, `SHOW BACKUP` inspection, and backup **validation** with three levels of verification (✅ docs). The docs are explicit that operational backups should use `BACKUP`/`RESTORE`, not `pg_dump` (which is supported for schema export — ✅ §4.1).

RPO framing for the estate: the engine's MVCC timestamps + revision-history backups give you *point-in-time* RPO≈0-in-window recovery, and the cross-region replication (§6.5) covers the availability axis; backups cover the *logical/operational* disaster axis (corruption, fat-finger, ransomware) that replication cannot (cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2 RPO/RTO and §5 DR frames).

### 7.3 The Data Table

| Data feature | What it is | Notes |
| --- | --- | --- |
| **CDC changefeeds** | Row-level change streams emitted to sinks in real time | Kafka, Pub/Sub, webhook, cloud-storage sinks; at-least-once, per-key timestamp ordering (✅ §7.1) |
| **CDC queries** | SQL-defined filtering/transformation of change data before emission | Schema stabilisation for downstream systems (✅ §7.1) |
| **Changefeed safety** | Protected timestamps shield streaming data from GC | Backups and changefeeds both use protected timestamps (✅ §7.1–§7.2) |
| **Full/incremental backups** | Un-replicated base copies + change deltas | Cluster/database/table scope; job-based (✅ §7.2) |
| **Scheduled backups** | Periodic automated backup jobs | GC-safe cadence independent of GC TTL (✅ §7.2) |
| **Revision history** | Backup captures every change inside the GC window | The prerequisite for PITR (✅ §7.2) |
| **Point-in-time restore (PITR)** | Restore to an arbitrary timestamp within revision history | The dropped-table/bad-batch undo (✅ §7.2) |
| **Encryption & locality** | KMS/passphrase-encrypted backups; locality-aware placement/execution | DR + data-residency alignment (✅ §7.2) |
| **Backup validation** | Three levels of restorability verification | Faster than a full restore drill (✅ §7.2) |

---

## 8. The Cloud and the Licensing

### 8.1 CockroachDB Cloud — Dedicated

**CockroachDB Cloud Dedicated** is the fully managed, **single-tenant** offering: your own cluster on AWS, GCP or Azure, operated by Cockroach Labs (✅ product/cloud page: "a fully managed distributed SQL database with effortless scale and up to 99.999% availability"; ✅ cloudpicker/startupik summaries: Dedicated = "isolated, customizable clusters... priced based on custom enterprise agreements"). Single-tenant isolation is the enterprise story: no noisy neighbours, custom node sizing, VPC peering/private networking, and the managed service handles upgrades, backups and monitoring. The **99.999% availability** figure is the vendor's SLA-level claim (⚠ vendor claim — the actual SLA terms and achievable numbers depend on configuration and region topology; flagged rather than asserted). Dedicated is the natural fit for the regulated estate: isolation, compliance tooling, and a procurement conversation rather than a self-operated fleet (cross-ref [../management/business_case_development_guide.md](../management/business_case_development_guide.md) for the buy-vs-build economics).

### 8.2 CockroachDB Cloud — Serverless

**CockroachDB Cloud Serverless** is the fully managed, **usage-based** offering: clusters that scale to zero, auto-scale with traffic, and bill per use, with a **free tier** (✅ cloudpicker: "Offers Serverless for usage-based scaling... CockroachDB Cloud offers a free tier for its Serverless product, followed by usage-based pricing"). Serverless shares the same SQL/consistency surface as Dedicated — the database is the database — but it is multi-tenant under the hood and aimed at development, staging, lightweight production, and bursty workloads. The trade to flag honestly for a solutions architect: ⚠ third-party analyses argue Serverless can be *more* expensive per transaction than Dedicated at sustained high throughput (the "pay-as-you-go" premium), so the Serverless-vs-Dedicated decision should be workload-shaped, not convenience-shaped (⚠-flagged third-party claim, not verified against Cockroach Labs' own pricing sheets this pass — pricing changes; check cockroachlabs.com/pricing).

### 8.3 Self-Hosted — and the Deployment Shapes

The self-hosted story is the one the architecture section already implies: **run CockroachDB anywhere** — bare metal, VMs, containers, Kubernetes (the operator is the standard path), private data centres or any cloud — with the same codebase (✅ Wikipedia: "can be run in a variety of environments such as bare metal, VMs, containers and Kubernetes... both in private data centers and in the cloud"; ✅ docs deploy guides). This is a genuine differentiator vs Spanner (Google-only) and a real consideration for banks with on-prem/MAS-residency requirements (§10). The operational surface is the `cockroach` CLI, the DB Console, and the Kubernetes Operator; the docs' recommended production settings (load balancers in front, `--join` flags, zone configs) are the deployment discipline (✅ docs).

### 8.4 The Licensing Journey — Apache 2.0, BSL, CCL

The license timeline (✅ verified this pass; the sequence is also flagged here because it is the most commonly misremembered part of the story):

1. **Apache License 2.0** — from the Feb 2014 open-source launch through mid-2019. Genuinely OSI open source.
2. **Business Source License (BSL), June 2019** — announced June 4, 2019 ("Why We're Relicensing CockroachDB" — Kimball, Mattis, Darnell); **CockroachDB 19.2 was the first BSL release** (✅ Wikipedia + OpenSourceForU + SD Times). The BSL (based on MariaDB's BSL 1.1 — ✅ hardcracked/OpenSourceForU) is **source-available, not OSI open source**: free to use and modify internally, but you may not "offer a commercial version of CockroachDB as a service without buying a license" (✅ Wikipedia's phrasing) — the anti-cloud-provider clause. The stated motivation: protect the project from cloud vendors packaging it as a managed service — the same 2019 wave as MongoDB's SSPL and Redis's RSAL (✅-structural).
3. **CockroachDB Community License (CCL)** — enterprise features subsequently moved under Cockroach Labs' own CCL (✅ It's FOSS's summary of the sequence: Apache → BSL → CCL for enterprise offerings; ⚠ the precise feature/license mapping is version-dependent and should be checked against the current licensing FAQs before procurement).

Practical reading for this repo's audience: **"open-source CockroachDB" is, since mid-2019, actually source-available CockroachDB.** Self-hosting, modification, and internal use are fine; *commercially offering it as a service to third parties* requires a licence; and the enterprise feature set (multi-region enterprise features, backups, CDC, etc. — ⚠ feature-to-edition mapping varies by version) may sit under CCL. Any banking procurement must read the current licensing FAQs — the guide's ⚠ stands until that check is done.

### 8.5 The Cloud Table

| Offering | What it is | Notes |
| --- | --- | --- |
| **Cloud Dedicated** | Fully managed, single-tenant clusters on AWS/GCP/Azure | Custom sizing, private networking, enterprise pricing; vendor claims up to 99.999% availability (⚠ vendor claim) (✅ §8.1) |
| **Cloud Serverless** | Fully managed, multi-tenant, usage-based clusters | Free tier; scales to zero; per-use billing; per-transaction cost premium at sustained high throughput (⚠ third-party analysis) (✅ §8.2) |
| **Self-hosted** | Run the same code anywhere: bare metal, VMs, containers, Kubernetes, private DCs, any cloud | The Spanner-differentiator: not cloud-locked; Kubernetes Operator is the standard path (✅ §8.3) |
| **Core license** | BSL since 19.2 (June 2019); Apache 2.0 before | Source-available; no commercial DBaaS without a licence; enterprise features under CCL (⚠ version-dependent mapping) (✅ §8.4) |
| **Free tier / trial** | Serverless free tier; self-hosted has no licence fee for internal use | The low-friction adoption path (✅ §8.2) |
| **Pricing model** | Usage-based (Serverless) vs custom enterprise agreements (Dedicated) vs self-hosted infra cost | ⚠ current numbers change; consult cockroachlabs.com/pricing (✅ §8.2, §8.6) |

### 8.6 The Pricing Angle

The pricing model is three-shaped: Serverless is metered usage with a free tier; Dedicated is custom enterprise agreements; self-hosted is your own infrastructure cost with a commercial licence conversation for the enterprise feature set (✅ pricing page exists with this Dedicated-vs-self-hosted framing; ⚠ specific numbers deliberately not quoted — they change and are negotiation-dependent). For the [../management/business_case_development_guide.md](../management/business_case_development_guide.md) angle: the honest TCO comparison is not licence-fee-vs-free, it is **operating-cost-vs-engineered-cost** — a self-hosted 3-region, region-survivable cluster is real engineering (nodes, networking, backups, upgrades, runbooks); Dedicated trades that for subscription; Serverless trades it for metered usage. Each is a different point on the same buy-vs-build curve.

---

## 9. The Comparisons

### 9.1 CockroachDB vs Google Spanner

The comparison is the family comparison — Spanner is the parent, CockroachDB the independent child (✅ §2.2):

- **Same family:** globally distributed relational database, replicated ranges (Spanner *tablets* / CockroachDB *ranges*), consensus replication (Paxos / Raft), strong consistency, externally consistent transactions, geo-partitioning controls.
- **Clock strategy — the defining difference:** Spanner uses **TrueTime** — GPS receivers + atomic clocks per data centre — giving externally consistent, linearizable transactions with bounded commit-wait; CockroachDB uses **hybrid logical clocks** + uncertainty intervals (commit-wait for conflicting readers), achieving external consistency for overlapping-key transactions without special hardware (✅ Wikipedia's consistency-model discussion + §2.2). The trade: Spanner's linearizability is stricter; CockroachDB runs on commodity hardware anywhere.
- **Deployment:** Spanner is **Google-Cloud-only, a managed service** (⚠-structural — Google's product; no self-hosted Spanner exists); CockroachDB is software you run anywhere or consume as CockroachDB Cloud. For a bank that cannot put data on Google Cloud, the comparison ends there.
- **SQL dialect:** Spanner speaks GoogleSQL/PostgreSQL-interoperable dialects ⚠-structural; CockroachDB is PostgreSQL-wire-compatible (§4.1).
- **Cost shape:** Spanner is metered managed service; CockroachDB has self-hosted, Dedicated and Serverless shapes (§8).

### 9.2 CockroachDB vs TiDB

**TiDB** (PingCAP, founded 2015, open source — ✅-structural; cross-ref the Wikipedia "See also" which lists TiDB and YugabyteDB as the comparison neighbours) is the other major open-source distributed SQL database, and the closest in architecture: SQL front end (TiDB server) over a distributed transactional KV store (TiKV) with Raft, plus a Placement Driver for shard management. The differences that matter:

- **SQL compatibility:** TiDB is **MySQL-wire-compatible** (MySQL protocol and dialect, ⚠-structural — well documented); CockroachDB is PostgreSQL-wire-compatible. For the estate this is usually the deciding axis: which legacy does your application code speak?
- **Transaction protocol:** TiKV uses the **Percolator-style two-phase commit** model (from Google's Percolator paper) with Raft for replication (⚠-structural, widely documented); CockroachDB uses write intents + **parallel commits** with the same Raft substrate (§5). Both are strongly consistent; the commit-latency behaviours differ.
- **Ecosystem/company:** TiDB is China-rooted (PingCAP, Beijing/Silicon Valley), with a strong APAC presence and TiDB Cloud managed offering (⚠-structural); CockroachDB is New York-rooted (§2). Both have enterprise/cloud tiers.
- **Both** are genuinely distributed SQL: sharded, Raft-replicated, ACID, elastic. The honest summary: **same category, different wire dialect and commit-protocol lineage.**

### 9.3 CockroachDB vs YugabyteDB

**YugabyteDB** (Yugabyte, founded 2016 by ex-Facebook engineers Kannan Muthukkaruppan, Karthik Ranganathan and Mikhail Bautin — the team behind the Facebook Cassandra fork and Amazon's DynamoDB lineage ⚠-structural) is the PostgreSQL-compatible challenger:

- **Architecture:** YugabyteDB = YSQL (PostgreSQL-compatible SQL) + YCQL (Cassandra-style API) over a shared document-store core (**DocDB**) with **Raft** per tablet (⚠-structural, widely documented). CockroachDB = one SQL surface over a KV store with Raft (§3).
- **PostgreSQL story:** YugabyteDB is built *on* PostgreSQL code (the YSQL layer wraps Postgres); CockroachDB reimplemented the PostgreSQL wire protocol and dialect against its own distributed core (§4.1). Both advertise PostgreSQL compatibility; the difference is in how much of the PostgreSQL feature surface survives distribution — read each vendor's compatibility matrix (⚠-structural — both maintain such lists).
- **Multi-API:** YugabyteDB offers the Cassandra-style YCQL API alongside SQL; CockroachDB is SQL-only (the KV API is internal). If you need to migrate a Cassandra workload without rewrites, YugabyteDB's YCQL is the argument; if you want one consistent relational system, CockroachDB is the cleaner shape.
- **Licensing:** both moved to source-available licensing for their core (YugabyteDB → Apache-2.0 core with source-available enterprise ⚠-structural; CockroachDB → BSL/CCL §8.4). Verify current terms per version.

### 9.4 CockroachDB vs PostgreSQL

This is the "why not just use the thing it's compatible with?" comparison (cross-ref [oracle_database_guide.md](oracle_database_guide.md) for the classical-relational framing):

- **PostgreSQL** is a superb single-node relational database: richer feature surface (range types, FDWs, triggers, extensions), lower per-node overhead, no consensus overhead. **It does not scale horizontally on its own** — read replicas, partitioning and external sharding tools (like Citus) are add-ons, and there is no built-in consensus-based multi-writer story.
- **CockroachDB** trades some PostgreSQL features (the unsupported list in §4.1) for: automatic sharding, Raft consensus, serializable-by-default distributed transactions, multi-region SQL controls, self-healing, CDC and PITR built in.
- **The honest migration note:** an application written to the PostgreSQL compatibility surface (§4.1) will often port with near-zero code change — but the *operational* contract changes completely: retry-aware transactions (§5.4), no sequences with gaps-guarantees ⚠ (CockroachDB sequences behave differently under distribution), and multi-region latency physics (§6.5).
- **The decision rule:** if your scale is one node and your feature need is the full PostgreSQL surface, use PostgreSQL. If your need is *the database must survive regions and scale out while staying relational*, CockroachDB is the PostgreSQL-compatible way to get it. The two are complementary members of the same SQL dialect family, not substitutes at the same price point.

### 9.5 The Comparison Table — Dimension / CockroachDB / Alternative

| Dimension | CockroachDB | The alternative |
| --- | --- | --- |
| **Category** | Distributed SQL / NewSQL — SQL over a Raft-replicated KV store (§3) | Spanner (managed global SQL, Paxos+TrueTime); TiDB (MySQL-wire, Percolator-style 2PC); YugabyteDB (Postgres-based YSQL + Cassandra-style YCQL) |
| **Wire compatibility** | PostgreSQL wire v3.0 (§4.1) | TiDB → MySQL wire ⚠; YugabyteDB → PostgreSQL (via Postgres core) ⚠; Spanner → GoogleSQL/Postgres-interoperable dialects ⚠ |
| **Clock/consistency** | Hybrid logical clocks; external consistency for overlapping keys; commit-wait only for conflicting readers (§5.3) | Spanner → TrueTime atomic clocks, linearizable (✅ §2.2); TiDB/YugabyteDB → similar HLC approaches ⚠-structural |
| **Consensus** | Raft per range; leaseholder = leader (§3.5–§3.6) | Spanner → Paxos; TiDB → Raft (TiKV); YugabyteDB → Raft (DocDB) ⚠-structural |
| **Commit protocol** | Write intents + Parallel Commits (§5.2–§5.3) | TiDB → Percolator-style 2PC ⚠; Spanner → 2PC (TrueTime-bounded commit-wait) ⚠-structural |
| **Default isolation** | SERIALIZABLE (READ COMMITTED optional) (§5.1) | PostgreSQL → READ COMMITTED default; TiDB → snapshot-like repeatable read ⚠; Spanner → strong/external consistency ⚠-structural |
| **Multi-region control** | SQL-level: regional/RBR/global tables, survival goals, super regions (§6) | Spanner → config-level placement (not SQL); TiDB → placement rules ⚠; YugabyteDB → zone/region placement ⚠ |
| **Data residency** | Super regions for domiciling (§6.6) | Spanner → config-based; others → placement rules ⚠ |
| **CDC** | Built-in changefeeds (Kafka/PubSub/webhook/storage sinks) (§7.1) | PostgreSQL → logical decoding + Debezium ecosystem ⚠-structural; TiDB → TiCDC ⚠; YugabyteDB → CDC ⚠ |
| **Backup/PITR** | Full/incremental/scheduled, revision history, PITR, encryption, validation (§7.2) | PostgreSQL → pg_dump/WAL archiving + PITR ⚠-structural; TiDB/YugabyteDB → BR/backup tooling ⚠ |
| **Deployment** | Anywhere: self-hosted, Kubernetes, Dedicated, Serverless (§8) | Spanner → Google Cloud only ⚠-structural; TiDB → self-hosted + TiDB Cloud ⚠; YugabyteDB → self-hosted + managed ⚠ |
| **License** | Apache 2.0 → BSL (2019) → enterprise under CCL (§8.4) | PostgreSQL → PostgreSQL License (permissive OSS); TiDB → Apache 2.0 core ⚠; YugabyteDB → Apache 2.0 core + source-available enterprise ⚠; Spanner → proprietary SaaS |
| **Classical alternative** | The horizontal-scale relational answer | Oracle (single-node scale-up + manual sharding — see [oracle_sharding_guide.md](oracle_sharding_guide.md)); PostgreSQL (single-node, §9.4) |
| **NoSQL alternative** | Strong consistency + SQL where NoSQL gives eventual consistency | Cassandra/DynamoDB-style stores — see [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) |

### 9.6 The Rest of the Spectrum — Oracle and NoSQL

Against **Oracle** (the house's classical reference — [oracle_database_guide.md](oracle_database_guide.md)): CockroachDB is not a feature-for-feature Oracle replacement — the Oracle feature surface (PL/SQL, advanced analytics, RAC's shared-cache clustering) is a different world. The comparison that matters is the *scaling pattern*: Oracle Sharding ([oracle_sharding_guide.md](oracle_sharding_guide.md)) is manual, DBA-driven sharding of a mature engine; CockroachDB is automatic, engine-driven sharding with consensus. Where the sharding guide's pattern needs a shard key, a shard catalogue and application routing, CockroachDB's ranges do all of that internally (§3.3). Against **NoSQL** ([nosql_data_modelling_guide.md](nosql_data_modelling_guide.md)): the position is the mirror — NoSQL traded consistency and SQL for scale; CockroachDB refuses the trade and prices it in infrastructure. Both positions are legitimate; the category map is complete only with both ([neo4j_graph_database_guide.md](neo4j_graph_database_guide.md) holds the graph corner).

---

## 10. The Worked Example — a Multi-Region Banking Deployment

### 10.1 The Scenario — a Banking Workload

**The scenario (illustrative — in the house's familiar context, not a description of any real Cymbal Bank system ⚠):** a cross-border payments-and-accounts platform serving retail accounts and corporate payments across three jurisdictions — Singapore (MAS-regulated, the home base), Hong Kong and London. Requirements:

- **24-7 availability** — payments never stop (cross-ref [../banking/nets_singapore_guide.md](../banking/nets_singapore_guide.md) — the switch that never sleeps — and [../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md)); the platform must survive a full regional failure.
- **Strong consistency** — account balances, payment statuses and standing instructions must be serializable; no eventual-consistency surprises on a balance read.
- **Data residency** — Singapore customer data must not leave Singapore (MAS data-residency expectations); the same logic applies per jurisdiction.
- **Reference data** — currency rates, holiday calendars, BIC directories: read-mostly, needed in all regions.
- **Integration** — every payment event must stream to the analytics lakehouse and the real-time risk/AML pipelines.

That requirement set is almost a checklist of the preceding sections: consensus for availability (§3.5), serializable transactions for the balances (§5), regional-by-row for domiciled customer data (§6.3), global tables for reference data (§6.4), region survival for the DR promise (§6.5), changefeeds for the event streams (§7.1), PITR backups for the operational-disaster undo (§7.2).

### 10.2 The Regional Design

**Topology:** 3 cloud regions — `ap-southeast-1` (Singapore, primary), `ap-east-1` (Hong Kong), `eu-west-2` (London) — each with ≥3 availability zones. Nodes started with `--locality=region=...,zone=...`; a CockroachDB Cloud Dedicated topology or a self-hosted Kubernetes deployment — the design is identical.

**Database-level settings (the SQL that encodes the design):**

```sql
CREATE DATABASE payments;
ALTER DATABASE payments PRIMARY REGION "ap-southeast-1";      -- home base
ALTER DATABASE payments ADD REGION "ap-east-1";
ALTER DATABASE payments ADD REGION "eu-west-2";
ALTER DATABASE payments SURVIVE REGION FAILURE;               -- RF 3 → 5, spread 2+2+1
```

The `SURVIVE REGION FAILURE` line is the availability contract: any one region can vanish and reads/writes continue, at the documented cost of cross-region write coordination (§6.5). The 2+2+1 replica spread means London's loss leaves quorum in Singapore+HK; Singapore's loss leaves quorum in HK+London — and the *leaseholders* fail over to the surviving replicas automatically (§3.6). The read side never pays the price — reads stay local (follower reads at the closed timestamp, §3.6/§6.4).

**Table localities (the data-placement contract):**

| Table | Locality | Why |
| --- | --- | --- |
| `customers` | `REGIONAL BY ROW` | Each customer row homed in its jurisdiction — domiciled by construction; `crdb_region` defaults to the insert gateway's region; auto-rehoming if the customer moves |
| `accounts`, `transactions` | `REGIONAL BY TABLE IN "ap-southeast-1"` | The core ledger is Singapore-homed; all ledger writes and balance reads are local to SG |
| `fx_rates`, `holiday_calendars`, `bic_directory` | `GLOBAL` | Read-mostly reference data served at local latency in all three regions; writes are rare and accept the future-timestamp cost |
| `audit_log` | `REGIONAL BY TABLE IN "ap-southeast-1"` | Append-heavy; domiciled; streamed out via changefeed |
| `payment_events` (to analytics) | CDC changefeed → Kafka → lakehouse | The outbox-less event backbone (§7.1) |

And the **super region** for the SG data: `ALTER DATABASE payments ADD SUPER REGION "sg_region" VALUES ("ap-southeast-1")` — an explicit, auditable statement that SG customer data's replicas live only in SG (the repo's [singapore_data_centres_guide.md](singapore_data_centres_guide.md) frame; §6.6).

**The operational layer:** scheduled full+incremental backups with revision history to immutable cloud storage, validated on a cadence (§7.2); changefeeds with execution locality pinned to the sink's region (§7.1); per-replica circuit breakers giving fast failures instead of hangs (§3.6); and the app layer written to the retry protocol — every multi-statement transaction uses `SAVEPOINT cockroach_restart` and handles `40001` with backoff, and payment idempotency keys resolve the `40003` ambiguous-commit class (§5.4; cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §8).

### 10.3 The Schema Sketch

```sql
CREATE TABLE customers (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name          STRING NOT NULL,
    jurisdiction  STRING NOT NULL CHECK (jurisdiction IN ('SG','HK','UK')),
    status        STRING NOT NULL DEFAULT 'ACTIVE'
) LOCALITY REGIONAL BY ROW;                      -- domiciled per row

CREATE TABLE accounts (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id  UUID NOT NULL REFERENCES customers(id),
    currency     STRING NOT NULL,
    balance_cents BIGINT NOT NULL DEFAULT 0,
    version      INT NOT NULL DEFAULT 0
) LOCALITY REGIONAL BY TABLE IN "ap-southeast-1"; -- the ledger lives in SG

CREATE TABLE fx_rates (
    pair  STRING NOT NULL,
    rate  DECIMAL(18,6) NOT NULL,
    ts    TIMESTAMPTZ NOT NULL,
    PRIMARY KEY (pair, ts)
) LOCALITY GLOBAL;                                 -- read locally everywhere

-- The money movement (must be retry-safe in the app):
BEGIN;
SAVEPOINT cockroach_restart;
UPDATE accounts SET balance_cents = balance_cents - 100000, version = version + 1
 WHERE id = $1 AND version = $2;                  -- optimistic concurrency
INSERT INTO transactions (account_id, amount_cents, status) VALUES ($1, -100000, 'BOOKED');
RELEASE SAVEPOINT cockroach_restart;
COMMIT;
```

### 10.4 The Failure Drills

- **A node dies** — its ranges' quorum holds (3 voting replicas, 1 loss OK); leases move; no visible outage. Self-healing re-replicates (§3.4–§3.6). This is the everyday case.
- **An entire zone dies** — within-region availability holds (zone survival is baked into the region-survivable design); the region keeps serving.
- **An entire region dies (the drill that matters)** — with `SURVIVE REGION FAILURE`, each range still has quorum across the two surviving regions (2+2+1); leaseholders transfer; reads stay local in the surviving regions; writes coordinate across two regions instead of one — degraded but **available**, which is the promise. The drill proves the RPO=0 availability axis. Simultaneously, the CDC changefeed to the analytics sink keeps flowing from the surviving emission locality (§7.1).
- **A bad deployment / dropped table (the drill the availability design does NOT cover)** — replication does not protect against logical damage; the **PITR restore from revision history** is the answer: restore the table to the pre-change timestamp and reconcile the delta (§7.2). This is why backups are scheduled and validated *independent* of the replication story.

### 10.5 The Lessons

1. **The design is SQL, not engineering folklore.** Every availability and placement decision — region survival, row domiciling, reference-data locality — is a declarative `ALTER DATABASE`/`ALTER TABLE` statement (§6). That is the product's deepest operational advantage: the intent is in versioned, reviewable, auditable DDL.
2. **Availability and write-latency are the same knob.** `SURVIVE REGION FAILURE` buys region survival with cross-region write coordination; the design should reserve that cost for the data that needs it (ledgers), not for everything (reference data stays global, cheap) (§6.5, §6.4).
3. **The application contract changed even though the SQL didn't.** PostgreSQL drivers plugged in unchanged, but transaction code must be retry-aware and idempotent (§5.4) — the migration is 90% SQL-compatible and 100% discipline-required.
4. **Domiciling is auditable.** Super regions turn a regulatory requirement into a queryable fact (§6.6) — the compliance story is the design, not an afterthought.
5. **The event backbone is a changefeed.** The same committed rows that move money are the event stream (§7.1) — no outbox table, no dual-write gap (cross-ref [event_stream_processing_guide.md](event_stream_processing_guide.md) and the outbox discussion in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §4.3).
6. **Operational disasters are a different axis from hardware disasters.** Region survival covers the datacentre; PITR covers the human; you run both drills (§10.4).

---

## 11. The One-Page Summary — the Un-Killable Database

**CockroachDB is the database that refuses the old trade-off.** The 2015-era orthodoxy said you could have scale or you could have SQL — NoSQL took scale and eventual consistency; the classical relational engines kept SQL and ACID but scaled up, not out. CockroachDB, built by ex-Google engineers who had lived inside Bigtable and Spanner, rebuilt the whole stack for distribution: a SQL layer that speaks the PostgreSQL wire protocol, compiled onto a transactional key-value store, split into ranges, replicated by Raft, served by leaseholders, committed by parallel commits, and serializable by default. The result is a database you can shard without a shard key, fail a region without a failover runbook, and geo-place with `ALTER TABLE` — and that is why the press called it "almost impossible to take down" and why the category it founded is called distributed SQL.

The honest ledger balances the story: the compatibility with PostgreSQL is broad but has a differences list; serializable correctness demands retry-aware applications; region survival is bought with write-latency; and since June 2019 the licence is source-available, not OSI open source — the price of refusing to be a cloud vendor's free SKU. Spanner gave the world the model; CockroachDB gave the model commodity clocks, open code, and the ability to run anywhere — your data centre, your cloud, or a managed cloud of its own.

For the solutions architect's estate, the final word is the one the founders chose as the name: **the un-killable database** — the one that survives the node, the zone, the region, and the human, and keeps serving consistent SQL through all of them. That is the pitch, and — with the retries, the latency physics and the licence read in full — it is also the engineering reality. The cockroach, after all, was the only organism that outlived the asteroid.

---

## 12. The Glossary

| Term | Definition |
| --- | --- |
| **CockroachDB** | The source-available distributed SQL database: a PostgreSQL-wire-compatible relational database over a distributed, transactional, consistent KV store with Raft replication (✅ §1.1) |
| **Cockroach Labs** | The New York company founded 2015 by Spencer Kimball, Peter Mattis and Ben Darnell; maker of CockroachDB and CockroachDB Cloud (✅ §2.1) |
| **Distributed SQL** | The database category: full SQL + ACID transactions + horizontal scale across nodes with consensus replication (✅-structural, §1.2) |
| **NewSQL** | The umbrella term (c. 2011) for the new generation of relational databases that scale like NoSQL while keeping SQL and ACID; CockroachDB's historical category label (✅-structural, §1.2) |
| **Spanner** | Google's globally distributed database (OSDI 2012): Paxos + 2PC + TrueTime atomic clocks; the architectural parent of CockroachDB (✅ §2.2) |
| **Node** | One `cockroach` process/machine; the symmetric, interchangeable unit of the cluster (✅ §3.2) |
| **Range** | A contiguous span of the KV keyspace — the unit of distribution, replication and movement; auto-splits and rebalances (✅ §3.3) |
| **Replica** | A copy of a range on a node; 3 voting replicas by default; non-voting replicas (v21.1+) serve reads without quorum cost (✅ §3.4) |
| **Raft** | The consensus protocol: leader election, replicated log, majority commit — used per range (✅ §3.5; cross-ref the consensus concepts in the DDIA/zero-downtime guides) |
| **Consensus** | The distributed-systems agreement machinery (Raft/Paxos) ensuring replicas agree on committed state despite failures (✅ §3.5) |
| **Leaseholder** | The replica holding the range lease: serves consistent reads and coordinates writes; same replica as the Raft leader (✅ §3.6) |
| **SQL** | Structured Query Language — the relational interface CockroachDB presents to clients (✅ §4) |
| **PostgreSQL** | The open-source relational database whose wire protocol and dialect CockroachDB is compatible with (✅ §4.1) |
| **Wire protocol** | The byte-level client-server protocol (pgwire v3.0) that lets PostgreSQL drivers/tools talk to CockroachDB (✅ §4.1) |
| **Serializable** | The strongest SQL-standard isolation level — results as if transactions ran one at a time; CockroachDB's default (✅ §5.1) |
| **Isolation** | The property controlling how concurrent transactions see each other's uncommitted/committed state (✅ §5.1) |
| **Commit-wait** | The bounded wait some distributed-commit protocols require for correctness; CockroachDB's parallel commits avoid it in the common case, keeping it only for "conflicting readers" (✅ §5.3) |
| **Multi-region** | Deploying a cluster across geographic regions with SQL-controlled data placement and survivability (✅ §6) |
| **Regional table** | A table homed in a single region — fast local, slower remote (✅ §6.2) |
| **Survivability** | The failure-tolerance promise of a database — zone vs region survival goals (✅ §6.5) |
| **CDC** | Change Data Capture — emitting committed row changes as streams (changefeeds) (✅ §7.1) |
| **Changefeed** | The CDC job that watches tables and emits changes to a sink (Kafka, Pub/Sub, webhook, storage) at least once (✅ §7.1) |
| **Backup** | A recoverable copy of data — full, incremental, scheduled, encrypted (✅ §7.2) |
| **PITR** | Point-in-time recovery — restoring to an arbitrary timestamp using revision-history backups (✅ §7.2) |
| **Point-in-time recovery** | Same as PITR; the restore-from-arbitrary-timestamp capability (✅ §7.2) |
| **Cloud** | CockroachDB Cloud — the managed offerings: Dedicated (single-tenant) and Serverless (usage-based) (✅ §8) |
| **Dedicated** | The fully managed single-tenant cloud offering on AWS/GCP/Azure (✅ §8.1) |
| **Serverless** | The fully managed usage-based cloud offering with a free tier (✅ §8.2) |
| **BSL** | Business Source License — the source-available licence (MariaDB lineage) CockroachDB moved to in June 2019; forbids offering the DB as a commercial service without a licence (✅ §8.4) |
| **Business Source License** | Same as BSL; CockroachDB's licence since 19.2 (✅ §8.4) |
| **TPC-C** | The industry-standard OLTP benchmark; Cockroach Labs publishes its own TPC-C results (vendor-run — ⚠ not audited TPC-council results; v21.1: 1.68M tpmC, 140,000 warehouses, 95% efficiency, per the vendor docs) (✅ numbers sourced from the vendor's benchmarking page; ⚠ the "world record" framing is vendor marketing) |
| **Benchmark** | A standardized workload measurement; TPC-C the OLTP canonical one (§9.5 note; §13) |
| **YugabyteDB** | The PostgreSQL-compatible distributed SQL challenger (DocDB + Raft, YSQL/YCQL, ex-Facebook founders) (⚠-structural, §9.3) |
| **TiDB** | PingCAP's MySQL-wire-compatible distributed SQL database (TiDB/TiKV/Percolator-style 2PC) (⚠-structural, §9.2) |
| **Sharding** | Splitting data across nodes; manual in classical systems (see [oracle_sharding_guide.md](oracle_sharding_guide.md)), automatic via ranges in CockroachDB (✅ §3.3) |

---

## 13. Claims Status and Verification Notes

**Verification pass: 2026-08-24, live web access via the self-hosted Firecrawl backend (web search + direct page extraction, both working; some search queries returned empty results mid-pass and were retried or replaced with direct page extraction — a backend flakiness note, not a source failure).** Primary sources extracted/verified live: **docs.cockroachlabs.com** (Architecture Overview — the layer table, the goals, the node/range/replica/leaseholder/Raft definitions, the 3-replica default, "industry-leading consistency"; Replication Layer — Raft mechanics, the (RF−1)/2 failure math, non-voting replicas v21.1, epoch/expiration leases, circuit breakers, the leader-leaseholder identity; SQL Layer — the parser/optimizer/planner/executor pipeline, the PostgreSQL wire protocol, online schema changes and the F1 attribution, the gateway node; Transaction Layer — write intents, transaction records (PENDING/STAGING/COMMITTED/ABORTED), Parallel Commits, the commit-wait language for "conflicting readers", closed timestamps, lock types; Transactions — the SERIALIZABLE default, READ COMMITTED, the 40001/40003 error contract, the SAVEPOINT cockroach_restart protocol; Serializable Transactions demo page — "strongest of the four isolation levels"; Multi-Region Capabilities Overview — cluster/database regions, the setup flow, super regions and domiciling; Table Localities — regional / regional-by-row / global definitions, crdb_region, auto-rehoming, locality-optimized search; Multi-Region Survival Goals — ZONE vs REGION goals, the 3-region RF 3→5 2+2+1 spread, the write-latency cost; Change Data Capture Overview — changefeeds, sinks, at-least-once, CDC queries, scheduling, protected timestamps; Backup and Restore Overview — full/incremental/scheduled, revision history, PITR, encryption, locality-aware, validation; PostgreSQL Compatibility — pgwire v3.0, PG-18-aligned catalogs, the unsupported-features list; Benchmarking Overview (v25.4 docs) — v21.1 TPC-C: 1.68M tpmC, 140,000 warehouses, 95% efficiency, 40% over 19.2), **cockroachlabs.com product pages** (the Cloud product page — "fully managed distributed SQL database with effortless scale and up to 99.999% availability"; the pricing page's Dedicated-vs-self-hosted framing), **github.com/cockroachdb/cockroach** (32.4k stars, 4.1k forks, 657 watchers, ~118,988 commits, Go, the key design-document list), **en.wikipedia.org/wiki/CockroachDB** (the 2015 founding, the GFS/Reader backgrounds, the Jan 2014 design / Feb 2014 open-source chronology, the Black Duck award, the June 2019 Apache→BSL change with the founders' post citation, the seed round via WSJ, the NewSQL/Cloud-database categories, the consistency-model discussion), **Tracxn** (total funding ~$633M over 7 rounds, $278M Series F Dec 16, 2021, ~$5B valuation), and the press citations the Wikipedia article carries (Wired 2014 "almost impossible to take down", WSJ 2015 "$6.3 Million", VentureBeat).

**✅ Verified this pass (live sources):** the 2015 founding and the founders' identities and Google pedigrees; the Spanner-heritage framing and the "living without atomic clocks" design stance (the company's own writing); the architecture vocabulary and mechanics in §3 (nodes/ranges/replicas/Raft/leaseholders/quorum/leader-lease identity/non-voting replicas); the SQL-layer pipeline and PostgreSQL wire v3.0 compatibility including the versioned differences list; online schema changes with the F1-paper attribution; the SERIALIZABLE default and READ COMMITTED option; write intents/transaction records/Parallel Commits and the existence of commit-wait for conflicting readers; the 40001/40003 retry contract and the SAVEPOINT protocol; the multi-region vocabulary and all four table localities; ZONE vs REGION survival goals and the RF 3→5 2+2+1 mechanics; super regions for data domiciling; changefeed sinks/delivery semantics/CDC queries/protected timestamps; the backup catalogue including revision history and PITR; the BSL change (June 2019, 19.2 first BSL release, the anti-DBaaS clause, the founders' post); the GitHub repository metrics; the vendor-published TPC-C numbers (1.68M tpmC / 140k warehouses / 95% efficiency for v21.1, 40% improvement over 19.2) as *vendor-published benchmark results*; the funding arc ($6.3M seed 2015; ~$633M total; $278M Series F Dec 2021).

**⚠ Flagged / unverified this pass (honest ledger):** the **"world-record" TPC-C framing** — the numbers above are from Cockroach Labs' own benchmarking pages and are not audited, published TPC-C council results; treat comparisons against audited Oracle/SQL Server TPC-C results with care (⚠ §13, §9.5); **company valuation** figures vary by source and date (Tracxn ~$5B vs WOWLS ~$2B vs secondary ~$2.4B — ⚠ §2.4); **customer counts/revenue** (marketing-reported, not verified); the **99.999% availability** claim (vendor claim ⚠ §8.1); **Serverless-vs-Dedicated cost analyses** (third-party, ⚠ §8.2); **current pricing numbers** (deliberately not quoted — consult cockroachlabs.com/pricing; ⚠ §8.6); the **CCL enterprise feature/licence mapping** (version-dependent; check the current licensing FAQs ⚠ §8.4); **range-size default thresholds** (version-dependent, not asserted ⚠ §3.3); TiDB/YugabyteDB/Spanner architectural specifics beyond the widely documented level (⚠-structural §9.1–§9.3); the **§10 worked example** is an illustrative scenario in the house's context, not a description of any real Cymbal Bank system (⚠ §10.1); the vectorized-execution-engine claim (⚠-structural §4.2); sequence-gap behaviour under distribution (⚠ §9.4).

**Repository convention:** ✅ = verified this pass or in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = standard industry practice widely documented, not attributable to a single primary source.

---

## 14. Cross-References and Further Reading

**The database cluster (sibling, technology/):**
- **[oracle_database_guide.md](oracle_database_guide.md)** — **THE classical-relational companion.** Its SQL/PL-SQL and single-node scale-up world is what §1.5 and §9.4 position against; cross-ref the SQL-compatibility angle in §4.
- **[oracle_sharding_guide.md](oracle_sharding_guide.md)** — **THE horizontal-scale companion.** Its manual sharding patterns (shard keys, catalogues, routing) are §3.3's automatic ranges in classical clothes — the sibling already mentions CockroachDB; cross-ref §3.3 and §9.6.
- **[nosql_data_modelling_guide.md](nosql_data_modelling_guide.md)** — **THE spectrum companion.** Where NoSQL gives up consistency/joins for scale, CockroachDB keeps them; cross-ref §1.2 and §1.5.
- **[neo4j_graph_database_guide.md](neo4j_graph_database_guide.md)** — the database-category map; cross-ref §1.5.
- **[event_stream_processing_guide.md](event_stream_processing_guide.md)** — **THE streaming companion.** Its CDC/change-data-capture machinery is §7.1's changefeeds; cross-ref the at-least-once/idempotency themes constantly.
- **[kafka_alternatives_guide.md](kafka_alternatives_guide.md)** — the sink/streaming angle behind §7.1's Kafka changefeeds.
- **[zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md)** — **THE availability companion.** Its quorum (§5.3), failover (§5.2), RPO/RTO (§2.4), outbox/CDC (§4.3–§4.4) and retry/idempotency (§8) sections are §3.5, §5.4, §6.5, §7 and §10 of this guide in the abstract; cross-ref the multi-region survivability angle in §6 and §10 constantly.
- **[ddia_study_companion_guide.md](ddia_study_companion_guide.md)** and **[grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md)** — the study-guide genre; their replication/consensus/Spanner-NewSQL chapters are §3 and §5 of this guide in interview shape.
- **[distributed_auth_guide.md](distributed_auth_guide.md)** — the auth-availability angle (light cross-ref).
- **[databricks_guide.md](databricks_guide.md)** and **[cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md)** — the analytical side that §7.1's changefeeds feed (light cross-ref).
- **[singapore_data_centres_guide.md](singapore_data_centres_guide.md)** — the DC/DR frame under §10's Singapore-primary design.

**The banking cluster (prefix `../banking/`):**
- **[../banking/nets_singapore_guide.md](../banking/nets_singapore_guide.md)** and **[../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md)** — **THE payments cross-refs.** The 24-7 switch, the real-time payments context and the multi-entity settlement reality are what §10's scenario is built on.
- **[../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)** — the trading-day availability angle (light cross-ref in §10).
- The bank software-systems series — the estate context for §10's migration and procurement framing.

**The management cluster (prefix `../management/`):**
- **[../management/business_case_development_guide.md](../management/business_case_development_guide.md)** — the licensing/cloud-pricing investment angle behind §8 and §10.5 (buy-vs-build, TCO).

**Primary sources and further reading:** docs.cockroachlabs.com (Architecture Overview; Replication/SQL/Transaction layer pages; Transactions; Multi-Region Capabilities Overview; Table Localities; Multi-Region Survival Goals; Change Data Capture Overview; Backup and Restore Overview; PostgreSQL Compatibility; Benchmarking Overview; the licensing FAQs); the Cockroach Labs blog — "Why We're Relicensing CockroachDB" (June 4, 2019), "Living Without Atomic Clocks", "Serializable, Lockless, Distributed: Isolation in CockroachDB", "How CockroachDB Does Distributed Atomic Transactions", "Consensus, Made Thrive"; the design document (github.com/cockroachdb/cockroach/blob/master/docs/design.md); the Spanner paper (Corbett et al., *Spanner: Google's Globally-Distributed Database*, OSDI 2012) and the F1 schema-change paper (Rae et al., *Online, Asynchronous Schema Change in F1*, VLDB 2013) — both cited by the CockroachDB docs; the Raft paper (Ongaro & Ousterhout, *In Search of an Understandable Consensus Algorithm*, 2014); Wikipedia's CockroachDB article and its press citations (Wired 2014/2015, WSJ 2015, VentureBeat 2015); Tracxn's Cockroach Labs funding record; the NewSQL-category literature (Matthew Aslett / 451 Research, c. 2011 — ⚠-structural); and the repo's own verified cluster listed above.

---

*End of guide — the un-killable database: the cockroach that outlives the asteroid, the region, and the cloud-vendor licence wars.*
