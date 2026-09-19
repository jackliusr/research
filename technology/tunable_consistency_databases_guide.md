# Tunable Consistency Databases — what the knob actually buys

**A per-database field guide to the consistency configuration surface: which systems expose a per-operation knob, what each setting actually guarantees, what the stronger setting costs in latency, availability and money, and — the part vendor pages tend to mumble — where the guarantee delivered is weaker than the name of the setting implies.**

> **Author:** Jack Liu Shurui — Solution Architect
> **Byline:** Jack Liu Shurui, Solution Architect
> **Context:** The product-level companion to the theory in `distributed_systems_engineering_guide.md`; part of the Database / distributed-systems cluster, sitting between the CAP/PACELC theory and the per-vendor operational guides.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** Apache Cassandra architecture — *Dynamo* (`cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html`, consulted September 2026) and DataStax OSS 3.x *How is the consistency level configured?* (`docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlConfigConsistency.html`, consulted September 2026); Amazon DynamoDB *Read consistency* and *Read/write capacity modes* (`docs.aws.amazon.com/amazondynamodb/latest/developerguide/`, consulted September 2026); Azure Cosmos DB *Consistency levels* and *Consistency level choices* (`learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels`, consulted September 2026); MongoDB manual — *Read Concern*, *Read Preference*, *Write Concern*, *Causal Consistency and Read and Write Concerns* (`mongodb.com/docs/manual/reference/`, consulted September 2026); Riak KV *Strong Consistency* app guide (`docs.riak.com/riak/kv/latest/developing/app-guide/strong-consistency/index.html`, consulted September 2026); Couchbase Server *Durability*, *N1QL consistency (scan_consistency)* and *Storage modes* (`docs.couchbase.com/server/current/`, consulted September 2026); Aerospike *Consistency* (`aerospike.com/docs/server/develop/consistency`, consulted September 2026); CockroachDB *AS OF SYSTEM TIME* and *Follower Reads* (`cockroachlabs.com/docs/stable/`, consulted September 2026); YugabyteDB *Follower reads in YSQL* (`docs.yugabyte.com/preview/explore/ysql-language-features/going-beyond-sql/follower-reads-ysql/`, consulted September 2026). Papers: DeCandia et al., *Dynamo: Amazon's Highly Available Key-value Store*, SOSP 2007; Bailis et al., *Highly Available Transactions: Virtues and Limitations*, VLDB 2014; Abadi, *Consistency Tradeoffs in Modern Distributed Database System Design* (PACELC), IEEE Computer 45(2) 2012; Viotti & Vukolić, *Consistency in Non-Transactional Distributed Storage Systems*, ACM Computing Surveys 49(1) 2016; Lamport, *On Interprocess Communication*, 1986; Attiya, Bar-Noy & Dolev, *Sharing Memory Robustly in Message-Passing Systems*, JACM 42(1) 1995.
> **Last Updated:** September 2026
> **Companion guides:**
> - [`distributed_systems_engineering_guide.md`](distributed_systems_engineering_guide.md) — the theory this guide assumes: §4.2 CAP and its misreading, §4.4 PACELC, §4.5 the other impossibilities, §5 the consistency spectrum, §6.6 quorum arithmetic, §7 replication and partitioning. **Read §5 and §6.6 first if the vocabulary below is unfamiliar.**
> - [`nosql_data_modelling_guide.md`](nosql_data_modelling_guide.md) — data models and modelling; this guide is deliberately *not* about modelling.
> - [`cockroachdb_guide.md`](cockroachdb_guide.md) — the database that refuses the knob; used here as the contrast case (§11).
> - [`polardb_vs_oceanbase_guide.md`](polardb_vs_oceanbase_guide.md) and [`oracle_sharding_guide.md`](oracle_sharding_guide.md) — the sharded-relational neighbours with their own, narrower, consistency surface.
> - [`chaos_engineering_guide.md`](chaos_engineering_guide.md) and [`deterministic_engineering_guide.md`](deterministic_engineering_guide.md) — how to test that the knob does what you think (§15 cross-references these rather than re-deriving the technique).
> - [`../banking/kafka_guide.md`](../banking/kafka_guide.md) and the event-streaming guides — log semantics and delivery guarantees, a different axis from read/write consistency.
> - [`late_arriving_data_guide.md`](late_arriving_data_guide.md) — time and ordering in analytics, where "bounded staleness" is a data-arrival question rather than a replica-read question.
> **Integrity convention:** ✅ = verified this pass against the cited primary source; ⚠ = flagged (the source is ambiguous, hedged, or version-specific); ❌ = rejected (the claim is contradicted by a primary source or by a cited measurement).
> **Claim classes:** PROVEN RESULT (a theorem or a peer-reviewed measurement), ENGINEERING CONVENTION (a widely-used implementation choice that is not a theorem), VENDOR MARKETING (a vendor's own framing of its product). Every recommendation below is tagged with the class of claim it rests on.
> **Illustrative figures:** Every number attributed to **Cymbal Bank** in this guide is explicitly illustrative and fictional — Cymbal Bank is a fictional institution invented for worked examples. No figure in a Cymbal Bank example is a benchmark, a customer measurement, or production telemetry. Numbers attributed to vendors or papers are cited to their source and marked ✅ where verified this pass.
> **Verification method, stated up front:** Every URL in **Primary Sources** was fetched from this host during September 2026; where a page could not be retrieved, it is named in this paragraph and the affected material is marked ⚠ inline rather than being invented. Pages extracted successfully and used directly include the DataStax Cassandra 3.x consistency-level reference (the most complete published enumeration of Cassandra CL semantics found this pass), the Apache Cassandra *Dynamo* architecture page, the DynamoDB read-consistency page, and the Azure Cosmos DB consistency trade-offs page. Three pages in the seed list **redirected to vendor landing pages and returned no substantive content**: the Riak KV read-quorum page (`docs.riak.com/riak/kv/latest/developing/usage/read/index.html` → Riak product chooser), the Aerospike consistency page (`aerospike.com/docs/server/develop/consistency` → Aerospike documentation portal), and the Couchbase N1QL consistency page (`docs.couchbase.com/server/current/n1ql/n1ql-language-reference/consistency.html` → Couchbase documentation portal). For those systems the guide states what could be verified from the pages that *did* resolve and **marks the remainder ⚠ rather than inventing definitions**. `web_search` on this host repeatedly returned **empty result sets** for these topics; per the repository's convention that emptiness is recorded as a **tool limitation, not as a negative fact** — an empty search is not evidence that a thing does not exist. Where a definition could not be confirmed against a vendor primary source this pass, the guide says so inline instead of asserting it.

---

## How this guide is organised

The theory of consistency — linearizability, causal, session guarantees, the CAP and PACELC framings, quorum arithmetic — is owned by [`distributed_systems_engineering_guide.md`](distributed_systems_engineering_guide.md). This guide starts one level down, at the **configuration surface**. For each database that exposes a per-operation or per-request consistency knob, it answers four questions in a fixed order:

1. **What is the knob called** — the literal parameter names a caller types; 2. **what does each setting actually guarantee**, in the vocabulary of §5 of the theory guide rather than in the vendor's adjectives; 3. **what does the stronger setting cost**, in latency, availability under failure, money, and application complexity; 4. **where is the guarantee weaker than its name** — the specific failure mode under which the setting delivers less than a reader of its name would assume.

Sections 1–2 build the shared vocabulary: what "tunable" means and what a knob can and cannot move. Sections 3–10 are the per-database profiles, ordered roughly from the systems with the richest knob surface (Cassandra, ScyllaDB, DynamoDB, Cosmos DB) to the systems whose knob is narrower or differently shaped (MongoDB, Riak, Couchbase, Aerospike). Section 11 is the deliberate contrast case — the databases that **refuse** to expose a per-operation consistency knob (CockroachDB, YugabyteDB) and why that refusal is itself a design position. Sections 12–15 generalise: what strong settings cost across the board, the catalogue of misleading names, a per-operation decision procedure, and how to test that the knob does what you think. That is **Part I**, and it is complete on its own as a product reference.

**Part II (sections 16–26) states the result the whole guide turns on and then applies it.** Section 16 gives the quorum result precisely — what `R + W > N` actually buys, why it does not buy linearizability, and what does — with the primary sources and the independent measurements attached. Sections 17–18 turn the analysis outward: the traps consolidated into a per-database configuration checklist, and the regulated-payments and banking angle, where the display read and the decision read have opposite requirements. Section 19 audits vendor claims, marketing against documentation; section 20 works a single end-to-end example — Cymbal Bank, the fictional institution used throughout this repository — choosing a setting per operation and writing down the reason, including where the team deliberately chose the weaker setting. Sections 21–23 are the audit layer: the team-level gotchas and anti-patterns, the guide's own claims audit, and an honest list of what could not be verified. Sections 24–26 close with the glossary, the cross-references and further reading, and the summary.

Two reading conventions matter. First, **strongest-to-weakest ordering is always stated** in every table; the industry is not consistent about whether "level 0" is the strong end or the weak end, and a table without an explicit direction is a trap. Second, **the default is not the safe setting** — in most of these systems the default is the weak one, because the default is tuned for throughput and the vendor's marketing is tuned for the strongest setting the engine *can* do. This guide always names the default explicitly.

## Table of Contents

1. [What "tunable consistency" actually means](#1-what-tunable-consistency-actually-means)
   - 1.1 the one-sentence definition · 1.2 three things called "consistency" · 1.3 what a per-operation knob can and cannot move
2. [The knob taxonomy — a decode table for ten systems](#2-the-knob-taxonomy--a-decode-table-for-ten-systems)
   - 2.1 knob shapes · 2.2 the master decode table · 2.3 the defaults, and whose interests they serve
3. [Apache Cassandra — the reference implementation](#3-apache-cassandra--the-reference-implementation)
   - 3.1 write consistency levels · 3.2 read consistency levels · 3.3 QUORUM arithmetic and the multi-DC trap · 3.4 SERIAL, LWT and "linearizable" · 3.5 where CL names oversell
4. [ScyllaDB — the same knobs, a different failure tail](#4-scylladb--the-same-knobs-a-different-failure-tail)
   - 4.1 what carries over · 4.2 the shard-per-core cost curve · 4.3 settings that behave differently
5. [Amazon DynamoDB — the boolean that costs 2×](#5-amazon-dynamodb--the-boolean-that-costs-2)
   - 5.1 `ConsistentRead`, and where it is silently ignored · 5.2 global tables: MREC vs MRSC · 5.3 the pricing asymmetry
6. [Azure Cosmos DB — five levels and the scope nobody reads](#6-azure-cosmos-db--five-levels-and-the-scope-nobody-reads)
   - 6.1 the five levels · 6.2 dynamic quorum and what it does to "strong" · 6.3 bounded staleness · 6.4 the API-mapping gap
7. [MongoDB — read concern × write concern × read preference](#7-mongodb--read-concern--write-concern--read-preference)
   - 7.1 the three-axis model · 7.2 read concern levels · 7.3 write concern and the durability conflation · 7.4 causal sessions
8. [Riak KV — r/w/pr/pw and the notfound_ok surprise](#8-riak-kv--rwprpw-and-the-notfound_ok-surprise)
   - 8.1 the quorum properties · 8.2 strong consistency buckets and their cost · 8.3 sibling creation
9. [Couchbase — durability levels and scan_consistency](#9-couchbase--durability-levels-and-scan_consistency)
   - 9.1 key-value durability · 9.2 query consistency · 9.3 the index-storage-mode dependency
10. [Aerospike — read mode and commit level](#10-aerospike--read-mode-and-commit-level)
    - 10.1 `SC` vs `AP` read modes · 10.2 commit level and the write path · 10.3 what the knob does not cover
11. [The databases that refuse the knob — CockroachDB and YugabyteDB](#11-the-databases-that-refuse-the-knob--cockroachdb-and-yugabytedb)
    - 11.1 CockroachDB `AS OF SYSTEM TIME` · 11.2 follower reads in CockroachDB · 11.3 YugabyteDB YSQL follower reads · 11.4 why the refusal is a position
12. [What the stronger setting costs](#12-what-the-stronger-setting-costs)
    - 12.1 the latency cost of the third replica · 12.2 availability under partition · 12.3 the money cost · 12.4 the application-complexity cost
13. [Where the name oversells the guarantee](#13-where-the-name-oversells-the-guarantee)
    - 13.1 the misleading-names catalogue · 13.2 consistency is not isolation · 13.3 the single-key scope boundary
14. [Choosing a knob per operation](#14-choosing-a-knob-per-operation)
    - 14.1 the procedure · 14.2 the operation-to-level table · 14.3 mixed-consistency schemas and the upgrade trap
15. [Testing that the knob does what you think](#15-testing-that-the-knob-does-what-you-think)
    - 15.1 the five tests · 15.2 what to instrument · 15.3 the failure-injection minimum

**Part II — the result, the traps, the domain, and the audit**

16. [The quorum result, stated precisely](#16-the-quorum-result-stated-precisely)
    - 16.1 what the arithmetic actually buys · 16.2 why it does not buy linearizability · 16.3 the conditions under which a quorum read gives more · 16.4 what it takes to obtain linearizable reads from a leaderless store · 16.5 the independent measurements, by product and date · 16.6 the one-table answer
17. [The traps, consolidated](#17-the-traps-consolidated)
    - 17.1 the trap table · 17.2 the configuration-questions checklist, per database
18. [The regulated-payments and banking angle](#18-the-regulated-payments-and-banking-angle)
    - 18.1 what the customer sees versus what authorises the movement · 18.2 fraud's two horizons · 18.3 read-your-writes is a session problem · 18.4 the audit and evidence implication
19. [The vendor-claim audit](#19-the-vendor-claim-audit)
    - 19.1 where the marketing overstates the guarantee · 19.2 the sentences read carefully, and their resolutions · 19.3 the verdict on vendor documentation
20. [The Cymbal Bank worked example](#20-the-cymbal-bank-worked-example)
    - 20.1 the operation inventory · 20.2 the §17 checklist, applied · 20.3 one partition, walked through the operations · 20.4 the alternatives, including the database that refuses the knob · 20.5 the cost comparison · 20.6 the recommendation, including the weaker settings chosen on purpose
21. [The gotchas and anti-patterns](#21-the-gotchas-and-anti-patterns)
22. [The claims audit](#22-the-claims-audit)
    - 22.1 verified · 22.2 flagged · 22.3 rejected · 22.4 the risk statement
23. [What Could Not Be Verified](#23-what-could-not-be-verified)
24. [Glossary](#24-glossary)
25. [Cross-references and further reading](#25-cross-references-and-further-reading)
    - 25.1 within this repository · 25.2 primary sources, with dates consulted
26. [Closing summary](#26-closing-summary)

## 1. What "tunable consistency" actually means

### 1.1 The one-sentence definition

**Tunable consistency is the property that a single database exposes a parameter — settable per operation, per session, per table or per bucket — that selects between two or more points on a read-consistency/availability trade-off curve, without changing the schema.** ✅ (this is the sense in which Apache Cassandra's own architecture page uses the phrase: "Multi-master replication using versioned data and **tunable consistency**", `cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html`).

Three parts of that definition are load-bearing, and each is routinely dropped in vendor marketing:

- **Per operation, not merely per cluster.** A cluster-wide consistency setting is a *configuration*, not a knob. The distinguishing feature of the systems in this guide is that two operations in the same millisecond may run at different consistency levels. Cassandra states this explicitly: consistency "can be configured on a cluster, datacenter, or per individual read or write operation" ✅ (`docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlConfigConsistency.html`).
- **Along a genuine trade-off curve.** If the "stronger" setting costs nothing, there is no trade-off and no real knob — the vendor would simply ship the strong setting as the default. Every knob in this guide has a price; §12 itemises it.
- **Without changing the schema.** Re-architecting around a database is not turning a knob. The knob must be a runtime choice.

### 1.2 Three different things get called "consistency" in these docs

The single largest source of confusion in this space is that vendor documentation uses one word for at least three unrelated properties. Before any per-database table is meaningful, the reader has to know which of the three a given page is describing.

| What the page calls "consistency" | The actual property | Whose page does this | Litmus test |
| --- | --- | --- | --- |
| **Replica consistency** | Whether a read sees the effect of a completed write, given that writes go to multiple replicas | Cassandra CL, Riak `r`/`w`, DynamoDB `ConsistentRead`, Cosmos DB levels | "If I write once and read once, do I see my write?" |
| **Isolation** | Whether concurrent *transactions* see each other's intermediate states | MongoDB read concern, most SQL `SET TRANSACTION ISOLATION LEVEL` | "Do two concurrent users interfere in ways they should not?" |
| **Durability / persistence** | Whether a write survives a crash of the node that acknowledged it | Couchbase durability level, MongoDB write concern, Aerospike commit level | "If the node dies one millisecond after the ACK, is my write still there?" |

These three are **orthogonal axes**, and the vendor naming actively obscures the fact. The clearest example in the current product landscape is MongoDB, whose **write concern** is routinely described by users as a "consistency setting" when its actual job is durability plus replication acknowledgement — a point taken up in §7.3 and in the misleading-names catalogue in §13.1. The theory guide's §5.3 owns the isolation-anomaly catalogue (including write skew under Snapshot Isolation, Berenson et al., SIGMOD 1995); this guide's job is to stop the reader from buying an isolation level and believing they bought replica consistency.

A fourth, narrower meaning — **temporal/read-your-own-time consistency** — appears in CockroachDB's `AS OF SYSTEM TIME` and in Couchbase's `scan_consistency`. Those are covered where they arise (§11.1, §9.2) rather than being folded into the taxonomy.

### 1.3 What a per-operation knob can and cannot move

A tunable consistency knob is a **read-path and write-path acknowledgement rule**. It changes *how many replicas must answer before the coordinator replies*. It does not change what the replicas do with the data when they get it, and it does not create transactionality.

| The knob **can** move | The knob **cannot** move |
| --- | --- |
| How many replicas must acknowledge before the client gets an ACK | Whether the operation spans multiple partitions atomically |
| The probability that a single read sees the latest acknowledged write | Whether concurrent transactions serialise correctly (isolation) |
| Latency, because more replicas is more round-trips | Whether a *multi-key* read is consistent as a set |
| Availability, because requiring more replicas means more ways to fail | Whether a write is durable on disk (that is a separate durability knob) |
| Which replica class serves the read (leader vs follower) | The wall-clock convergence time of the anti-entropy/repair loop, except at the strongest settings |

The practical consequence, and the thesis of this guide: **almost every surprising production incident attributed to "we used the wrong consistency level" is really one of the three rows in the right-hand column** — a cross-partition atomicity assumption, an isolation assumption, or a durability assumption — that the knob could never have satisfied in the first place. §13 catalogues these. §14 turns the observation into a selection procedure.

A second consequence worth stating early: **weak settings do not fail loudly.** A `QUORUM` read that cannot reach quorum returns an error you can see. A `ONE` read that returns a stale value returns HTTP 200 with wrong data. The failure mode of the weak end of every knob in this guide is **silent**, which is precisely why these settings need the testing discipline of §15 rather than the ordinary integration test suite.
## 2. The knob taxonomy — a decode table for ten systems

### 2.1 Knob shapes: quorum counts, named levels, booleans, multi-dimensional tuples

The systems in scope do not merely expose different *values*; they expose different *shapes* of knob. The shape determines what an operator can express, and it determines which mistakes are even possible.

| Knob shape | How it reads | Systems | What it can express | The mistake it invites |
| --- | --- | --- | --- | --- |
| **Integer quorum count** | `r=2, w=2, n=3` | Riak KV, Dynamo (paper) | Arbitrary quorum arithmetic against a fixed replica count | Arithmetic that does not actually overlap — `r + w ≤ n` |
| **Named level, mapped to a count** | `QUORUM`, `ONE`, `LOCAL_QUORUM` | Cassandra, ScyllaDB, Cosmos DB (Cassandra/ Mongo APIs) | A fixed menu of counts, plus locality (`LOCAL_*`) | Assuming the name is a guarantee rather than a count |
| **Boolean** | `ConsistentRead = true` | DynamoDB | Exactly one step of strengthening | Assuming "strong" means "linearizable" |
| **Named level, account-scoped, request-overridable** | `Strong`, `Session`, `Eventual` | Azure Cosmos DB | Five points on a spectrum, selected per account and overridable per request | Forgetting that the *write* path stays synchronous at the account's level |
| **Multi-dimensional tuple** | `readConcern` × `writeConcern` × `readPreference` | MongoDB | Independent control of isolation, durability-acknowledgement, and replica routing | Believing these three are one setting |
| **Per-bucket flag plus quorum** | `consistent: true` on a bucket type | Riak KV strong-consistency mode | All-or-nothing per bucket, with the quorum knobs quietly ignored | Leaving `r`/`w` set and assuming they still apply |
| **Durability level + separate query consistency** | `majority` / `persistToMajority`; query-side `scan_consistency` | Couchbase | Durability on the KV path and staleness on the query path, controlled separately | Setting durability and believing the *index* query is now fresh |
| **Read mode enum + commit level** | `SC` / `AP`; `commitLevel` | Aerospike ⚠ | Consistency class on the read side plus a write-acknowledgement level on the write side | Reading "SC" as "strongly consistent reads everywhere" |
| **Historical timestamp clause** | `AS OF SYSTEM TIME <ts>` | CockroachDB, YugabyteDB | A *lower bound* on how stale a read may be, expressed in time | Treating it as a consistency level rather than a time-travel bound |

The taxonomy has a practical payoff: **the shape tells you where the documentation will be vague.** Systems with integer quorums have crisp arithmetic and vague failure semantics. Systems with named levels have crisp-sounding names and arithmetic the vendor sometimes declines to publish. Systems with booleans have almost no expressive power and therefore almost no room to misconfigure — and consequently the most commonly *misread* documentation, because a single boolean cannot carry the nuance that consumers assume it carries.

### 2.2 The master decode table

The following table is the guide's central reference. Ordering is **strongest-to-weakest on the left-to-right of the "strongest setting" column**, and the "weakest setting" column names the default where the default is the weakest setting. Claim-class tags apply per row.

| Database | Per-operation knob? | Strongest setting | Weakest setting / default | What "strongest" actually guarantees | What it costs |
| --- | --- | --- | --- | --- | --- |
| **Apache Cassandra** ✅ | Yes — per statement, per session, per cluster | `ALL` (reads and writes); `SERIAL`/`LOCAL_SERIAL` for LWT | `ONE` is the default ✅ | `ALL`: every replica must ack. `QUORUM`: a majority of *all* replicas cluster-wide | Availability falls with each replica required; cross-DC `QUORUM` adds inter-DC RTT |
| **ScyllaDB** ⚠ | Yes — same CQL surface as Cassandra | `ALL`, `SERIAL` | `ONE` | Same CL semantics as Cassandra (design lineage) | Same shape; the cost curve differs because of the shard-per-core engine |
| **Amazon DynamoDB** ✅ | Yes — `ConsistentRead` per request | Strongly consistent read | Eventually consistent (default), half the cost ✅ | "most up-to-date data, reflecting the updates from all prior write operations that were successful" ✅ | Strong reads cost 2× an eventually consistent read ✅; **not available on GSIs or streams at all** ✅ |
| **Azure Cosmos DB** ✅ | Yes — account default + per-request override | `Strong` (linearizability) | `Session` is the recommended/most used; `Eventual` is weakest ✅ | `Strong`: "reads are guaranteed to return the most recent committed version of an item" ✅ | `Strong`: write latency ≈ 2× RTT between farthest regions + 10 ms p99 ✅; read throughput halved vs. single-replica reads ✅; blocked by default beyond 8,000 km ✅ |
| **MongoDB** ✅ | Yes — per operation, per session, per transaction | `readConcern: "linearizable"` (+ `writeConcern: {w:"majority", j:true}`) | `readConcern: "local"` default for primary and secondaries ✅; global default write concern is `majority` ✅ | `"linearizable"`: "data that reflects all successful majority-acknowledged writes that completed before the start of the read" ✅ | Linearizable reads are **primary-only** and "might not read from a consistent snapshot" unless the query uniquely identifies a single document via an immutable, unique-indexed key ✅ |
| **Riak KV** ⚠ | Yes — `r`, `w`, `pr`, `pw`, `rw` per request; `consistent: true` per bucket type | `consistent: true` bucket (linearizable per key) ✅ | Default bucket properties (eventually consistent) | Strong mode: "a value is guaranteed readable by any client *immediately* after a successful write has occurred to a given key" ✅ | Strong mode is "experimental… not commercially supported or production-ready" per Riak's own warning ✅, incompatible with MDC replication, Search, Data Types, Commit Hooks ✅ |
| **Couchbase** ✅ | Yes — durability level per mutation; `scan_consistency` per query ⚠ | KV: `persistToMajority`; Query: `request_plus` ⚠ | KV: regular (asynchronous) write ✅ | `majority`: "A majority of Data Service nodes must store the mutation to memory" ✅ | "if you configure a bucket with one replica and a node fails, you cannot perform a durable write to any vBucket that has data on the failed node" ✅ |
| **Aerospike** ⚠ | Yes — read mode and commit level | `SC` read mode; strongest commit level | `AP` (availability-oriented) read mode | Page-level definitions could not be fetched this pass — see §10 and *Blocked sources* | Not established from a primary source this pass — deliberately left blank rather than guessed |
| **CockroachDB** ✅ (contrast) | **No** generic knob; only read-only historical bounds | Serializable (default, no knob needed) ✅ | n/a — `AS OF SYSTEM TIME` is the *only* relaxation | Bounded staleness reads: served locally even under partition, at "increasingly" larger staleness ✅ | Bounded staleness reads must be single-statement, must read a single row, and must not require an index join ✅ |
| **YugabyteDB** ✅ (contrast) | Yes but narrow — `yb_read_from_followers` on read-only transactions | Leader-leased strongly consistent read ✅ | `yb_read_from_followers = false` (default) ✅ | Follower read at `now - yb_follower_read_staleness_ms` | "the read is always stale, even if you are reading from a tablet leader" ✅ — the knob cannot be turned off for a single statement once enabled for the session |

Two rows deserve immediate comment because they invert the usual story.

**Riak KV's strongest setting is officially disclaimed by its own vendor.** The Riak documentation states plainly: "Riak KV's strong consistency is an experimental feature and may be removed from the product in the future. Strong consistency is not commercially supported or production-ready… We do not recommend its usage in any production environment." ✅ This is a rare case of a vendor being *more* honest than its users. A guide that lists Riak's knob without that warning would be actively misleading, so it is quoted in full in §8.2.

**CockroachDB is in the table precisely because it has no general knob.** It is the control group. Its `AS OF SYSTEM TIME` clause is not a consistency level in the sense of §1.1 — it is a *time-travel bound* whose semantics ("at least 4.2 seconds in the past", ✅ per the `follower_read_timestamp()` definition) are about historical snapshots, not about how many replicas acknowledge.

### 2.3 The defaults, and whose interests they serve

| Database | Default consistency | Default durability/acknowledgement | Default set to favour |
| --- | --- | --- | --- |
| Cassandra | `ONE` for reads and writes ✅ | — | Throughput and availability |
| ScyllaDB | `ONE` (CQL default) ⚠ | — | Throughput and availability |
| DynamoDB | Eventually consistent reads ✅ | Write returns after durable persistence on success ✅ | Cost (reads are half the price) |
| Cosmos DB | `Session` (recommended for most workloads) ✅ | `Local Majority` for all levels except `Strong` ✅ | Latency with a per-user guarantee |
| MongoDB | `readConcern: "local"` ✅ | `writeConcern: {w: "majority"}` ✅ | Locality for reads, safety for writes |
| Riak KV | Eventually consistent ✅ | Quorum per bucket defaults | Availability |
| Couchbase | Regular (asynchronous) write ✅ | None unless durability requested ✅ | Throughput |
| CockroachDB | Serializable ✅ | Raft majority | Correctness |
| YugabyteDB | Leader read (strong) ✅ | Raft majority | Correctness |

The pattern is worth naming: **the databases built as key-value or document stores default to the weak end, and defaulting reads to weak is a cost decision as often as a latency decision.** DynamoDB is the cleanest statement of this: eventually consistent reads are literally *half the price* ✅, so the default is the one that makes the bill smaller. The two distributed SQL engines default to the strong end, because their target workloads cannot tolerate silent staleness and because they price that in. A fourth, narrower meaning — **temporal/read-your-own-time consistency** — appears in CockroachDB's `AS OF SYSTEM TIME` and Couchbase's query consistency, and is covered where it arises (§11.1, §9.2) rather than folded into this taxonomy.

This is the first place where the guide's thesis shows up in the defaults themselves. **A default of `ONE` is not a statement that one replica is sufficient for your business process; it is a statement that the vendor does not know your business process.** §14 gives the procedure for deciding per operation.

## 3. Apache Cassandra — the reference implementation

Cassandra is the reference implementation for this whole guide because it has the richest and best-documented knob surface. Everything the other systems do, Cassandra does with more settings and more published detail. It is also the system where the gap between a consistency-level *name* and the guarantee it delivers is widest.

### 3.1 The write consistency levels

DataStax's Cassandra 3.x documentation publishes the write levels in strongest-to-weakest order ✅. The definitions below are quoted or closely paraphrased from that page.

| Level | Requirement (quoted/aligned with source) ✅ | Official usage note ✅ |
| --- | --- | --- |
| `ALL` | "A write must be written to the commit log and memtable on **all** replica nodes in the cluster for that partition." | "Provides the highest consistency and the lowest availability of any other level." |
| `EACH_QUORUM` | "Strong consistency. A write must be written to the commit log and memtable on a quorum of replica nodes in *each* datacenter." | Maintains the same level in every DC; use when a read must fail if a DC's quorum is unreachable |
| `QUORUM` | "A write must be written to the commit log and memtable on a quorum of replica nodes across *all* datacenters." | Single- or multi-DC clusters; "use if you can tolerate some level of failure" |
| `LOCAL_QUORUM` | "Strong consistency. A write must be written to the commit log and memtable on a quorum of replica nodes in the same datacenter as the coordinator." | Avoids inter-DC latency |
| `ONE` | "at least one replica node" | "Satisfies the needs of most users because consistency requirements are not stringent." |
| `TWO` | at least two replica nodes | "Similar to `ONE`." |
| `THREE` | at least three replica nodes | "Similar to `TWO`." |
| `LOCAL_ONE` | at least one replica in the local datacenter | Local-availability uses |
| `ANY` | "A write must be written to at least one node. If all replica nodes for the given partition key are down, the write can still succeed after a hinted handoff has been written." | "Provides low latency and a guarantee that a write never fails. Delivers the lowest consistency and highest availability." |

Three things about this table repay close reading.

**First, the multi-DC semantics of `QUORUM` are not what most readers assume.** `QUORUM` is a quorum of replicas *across all datacenters*, and the DataStax page spells out the arithmetic: "In a two datacenter cluster where each datacenter has a replication factor of 3, a quorum is 4 nodes" ✅. A write at `QUORUM` in that topology must reach four of six replicas spanning two DCs — which means the write carries inter-DC latency whether the application wanted it or not. `LOCAL_QUORUM` is the setting most multi-DC deployments actually want, and the fact that its name is the *longer* one is a small usability trap.

**Second, `ANY` is the level whose name is furthest from its meaning.** ✅ "ANY" does not mean "any replica holds the data." It means "at least one node — possibly not a replica of this partition — holds a hint." The Write Consistency Levels table adds an important and easily-missed sentence: "If all replica nodes are down at write time, an `ANY` write is not readable until the replica nodes for that partition have recovered." So `ANY` writes can be *acknowledged and invisible*. That is a durability-shaped hazard wearing a consistency-level name. An application that writes at `ANY` and immediately reads at `ONE` can get a successful write and a not-found read. The theory guide's §5.3 and §6.6 supply the vocabulary for why; the operational lesson is simply that `ANY` is for write-only telemetry pipelines, not for anything whose value a human will read back.

**Third, "strong consistency" appears as a phrase in the `EACH_QUORUM` and `LOCAL_QUORUM` rows** ✅. Used loosely, that phrase invites the reader to believe `LOCAL_QUORUM` writes are linearizable. They are not — `LOCAL_QUORUM` is a quorum within one datacenter, and a read at `LOCAL_QUORUM` in a *different* datacenter need not see that write for an arbitrary period. This is the classic multi-DC read-your-writes failure, and it is the reason §13.3 insists on stating the scope of every setting explicitly.

### 3.2 The read consistency levels

| Level | Requirement ✅ | Note |
| --- | --- | --- |
| `ALL` | "Returns the record after all replicas have responded. The read operation will fail if a replica does not respond." | "highest consistency of all levels and the lowest availability" |
| `EACH_QUORUM` | **Not supported for reads** ✅ | A write-only level |
| `QUORUM` | "a quorum of replicas from all datacenters has responded" | — |
| `LOCAL_QUORUM` | "a quorum of replicas in the current datacenter as the coordinator has reported" | "Fails when using `SimpleStrategy`" ✅ |
| `ONE` | "Returns a response from the closest replica, as determined by the snitch." | "The replicas contacted for reads may not always have the most recent write." ✅ |
| `TWO` / `THREE` | most recent data from two/three of the closest replicas | — |
| `LOCAL_ONE` | closest replica in the local datacenter | — |
| `SERIAL` | "Allows reading the current (and possibly uncommitted) state of data without proposing a new addition or update. If a `SERIAL` read finds an uncommitted transaction in progress, it will commit the transaction as part of the read." ✅ | "Similar to QUORUM" |
| `LOCAL_SERIAL` | Same as `SERIAL`, confined to the datacenter | "Used to achieve linearizable consistency for lightweight transactions." ✅ |

The asymmetry between the read and write tables is itself instructive: **`EACH_QUORUM` is meaningful for writes and rejected for reads** ✅. A reader who assumes the two lists are symmetric will eventually ship a `EACH_QUORUM` read that errors at runtime. The reverse asymmetry also matters — `SERIAL` and `LOCAL_SERIAL` exist only as read levels in this sense, because a `SERIAL` read is the read half of Cassandra's compare-and-set protocol.

### 3.3 QUORUM arithmetic and the multi-DC trap

The DataStax page publishes the formula ✅:

```
quorum = (sum_of_replication_factors / 2) + 1
sum_of_replication_factors = datacenter1_RF + datacenter2_RF + ... + datacentern_RF
```

with rounding down to a whole number. The worked examples from that page are reproduced here because they are the fastest way to build intuition about how *badly* `QUORUM` scales with datacenter count:

| Topology | `QUORUM` (cluster-wide) | `LOCAL_QUORUM` |
| --- | --- | --- |
| single DC, RF=3 | 2 nodes ✅ | 2 nodes (local) |
| single DC, RF=6 | 4 nodes ✅ | 4 nodes (local) |
| two DCs, RF=3 each | 4 nodes ✅ | 2 nodes per DC |
| five DCs (two at RF=3, three at RF=2) | 7 nodes ✅ | 2 nodes in the coordinating DC |

The DataStax page states the consequence directly: "The more datacenters, the higher number of replica nodes need to respond for a successful operation." ✅ Read together with §2.3's point about defaults, this is the single most common production misconfiguration in Cassandra: a globally-distributed keyspace written at `QUORUM`, where the operator intended "a majority in each region" and got "a majority of the whole planet." The correct expression of the intent is `LOCAL_QUORUM`, possibly with `EACH_QUORUM` where a cross-region write must not be acknowledged until every region has its own majority ✅.

There is a second, subtler property of the arithmetic worth stating for a reader who has not internalised the theory guide's §6.6: **`QUORUM` + `QUORUM` overlaps only if the replication factor is the same for both sides of the equation and the cluster is not changing membership mid-flight.** Cassandra's actual read-path guarantee at `QUORUM` read + `QUORUM` write rests on the fact that the read quorum and the write quorum of the same replica set necessarily intersect. When the replica set *changes* — a node added, a node replaced, a node temporarily out of the ring — intersection is no longer guaranteed for writes that landed under the old member set. Cassandra papers over this with read repair and hinted handoff, not with a proof. This is why the operational guidance in every serious Cassandra deployment includes periodic `nodetool repair`: **the consistency level is necessary but not sufficient for the guarantee, and the repair loop is where the "eventual" in eventual consistency actually lives.** ⚠ (the requirement for repair is documented behaviour; the framing that the CL alone is insufficient is this guide's synthesis, and is consistent with the read-repair note in the `ONE` read row, which says read repair "runs in the background to make the other replicas consistent" ✅).

### 3.4 SERIAL, LWT and what "linearizable" means here

Cassandra's lightweight transactions (LWT) — `INSERT ... IF NOT EXISTS`, `UPDATE ... IF` — are implemented as Paxos over a quorum and use the `SERIAL`/`LOCAL_SERIAL` read levels ✅. The DataStax page describes `LOCAL_SERIAL` as "Used to achieve linearizable consistency for lightweight transactions" ✅.

That sentence deserves unpacking, because it is the one place in Cassandra where the word *linearizable* is used and it is used with a specific, narrower sense than the theory guide's §5.1 (Herlihy & Wing, TOPLAS 12(3) 1990).

| What Cassandra calls "linearizable" ✅ | What linearizability means in the theory guide §5.1 |
| --- | --- |
| Applies to a **single partition** compare-and-set | Applies to the whole system's operation history |
| `LOCAL_SERIAL` permits the Paxos quorum to be local to one DC | No locality qualifier — one global real-time order |
| Requires the read level to be `SERIAL`/`LOCAL_SERIAL`, not `QUORUM` | Requires a single, agreed order |
| Does not span partitions — Cassandra declines cross-partition transactions by design ✅ | Says nothing about partitioning; a partitioned implementation can be linearizable per key |

The relevant Cassandra architecture statement is explicit: Cassandra "chooses not to provide cross-partition transactions," instead offering "fast, consistent, latency at any scale for **single partition** operations" and "single partition compare and swap functionality via the lightweight transaction CQL API." ✅ So `LOCAL_SERIAL` delivers linearizability *for one key, within one datacenter's Paxos quorum*. For a ledger key that is exactly what is needed. For anything a business process treats as an invariant across two keys, it is not, and no consistency level in Cassandra can make it so. That boundary — **one partition, one key** — is the most important sentence in the whole Cassandra section, and it recurs in §13.3 and §14.

A further caution on LWT that the documentation itself implies but does not dwell on: `SERIAL` reads "commit the transaction as part of the read" ✅. A read that mutates is not free — it adds Paxos rounds to the read path, and it means a `SERIAL` read can *fail* for reasons a `QUORUM` read would not. Teams that sprinkle `SERIAL` on reads to "make them safe" pay a latency tax for a protocol they did not need unless the read is genuinely part of a compare-and-set.

### 3.5 Where CL names oversell: ANY, ONE, and the repair dependency

Pulling the Cassandra section together, the four places where the level name is weaker than it reads:

| Level | What the name suggests | What it delivers ✅ | The gap |
| --- | --- | --- | --- |
| `ANY` | "some replica has it" | "at least one node", possibly via hint; **invisible until replicas recover** | The name hides that the write may be unreadable |
| `ONE` | "the data is there, on a replica" | "the closest replica", "may not always have the most recent write" | The name says nothing about recency |
| `QUORUM` | "a majority agreed" | a majority *across all datacenters*; recency depends on replica-set stability | The name omits the topology |
| `LOCAL_QUORUM` | "strong consistency, locally" | strong *within one DC*; a reader in another DC can miss the write | The word "strong" is doing more work than it can carry |
| `SERIAL` | "linearizable" | linearizable **for one partition**, and a read that can write | The name omits the partition-scope qualifier |

None of this makes Cassandra's design wrong. `ONE` and `LOCAL_QUORUM` are correct defaults for what Cassandra is *for* — always-available, low-latency, single-partition access with a tunable, honest durability story. The point of the table is that **the knob is a promise about behaviour under partition and replica lag, and the promise is always smaller than the name.** The rest of this guide applies that same analysis to nine more systems and finds the gap varies in *shape*, not merely in size.
## 4. ScyllaDB — the same knobs, a different failure tail

### 4.1 What carries over from Cassandra unchanged

ScyllaDB is a CQL-compatible reimplementation and its consistency surface is deliberately the same shape: per-statement, per-session, same vocabulary, same `QUORUM` family and `LOCAL_*` variants. ⚠ **Verification status, stated plainly:** the seed's direct ScyllaDB architecture/consistency path **404s**, and `manager.docs.scylladb.com/stable/` resolves to a navigation hub rather than a definitional page ✅ (fetched, navigational only). The per-level definitions are therefore **not quoted from a ScyllaDB primary source in this pass**; nothing in this section is a ScyllaDB-specific definition. What follows is the structural difference that follows from the engine architecture, flagged ENGINEERING CONVENTION.

### 4.2 What the shard-per-core rewrite changes about the cost curve

| Dimension | Cassandra (JVM, thread-per-request) | ScyllaDB (shard-per-core, C++) ⚠ |
| --- | --- | --- |
| Coordinator fan-out cost | Thread-pool contention affects the tail under high concurrency | Work largely confined to a shard; coordination explicit |
| The "cost of the third replica" | Higher concurrency worsens p99 relative to p50 | Tighter tail; the marginal cost is more predictable |
| Failure detection | GC pauses can make a live replica *look* failed | No GC pauses; detection sees the actual network |

The consequence is a claim about *how teams behave*: **consistency levels get lowered to fix latency problems, and many of those latency problems are engine artefacts rather than network physics.** When the artefact disappears, an operator who had been running `ONE` "for latency" can often afford `LOCAL_QUORUM` at the same service level. That is a real upgrade in guarantee obtained by changing the engine rather than the knob — and a reminder that "the knob is expensive" is always a statement about a specific implementation.

### 4.3 The knob settings that behave differently

`ALL`, `SERIAL`/LWT, `ANY` and LWT throughput are all worth re-testing on migration rather than assuming — availability behaviour, Paxos implementation details and hinted-handoff mechanics are implementation properties, not semantic ones. ⚠ All flagged ENGINEERING CONVENTION, unverified against a ScyllaDB primary source this pass. The general rule this implies, and which §15 formalises: **a consistency level is a semantic contract plus an implementation of that contract, and only the first half survives a database migration.**

## 5. Amazon DynamoDB — the boolean that costs 2×

### 5.1 ConsistentRead, and where it is silently ignored

DynamoDB's knob is the sparsest in this guide: one boolean. The AWS documentation states the surface in three sentences ✅:

> "Both tables and LSIs provide two read consistency options: *eventually consistent* (default) and *strongly consistent* reads. **All reads from GSIs and streams are eventually consistent.**"

> "Read operations such as `GetItem`, `Query`, and `Scan` provide an optional `ConsistentRead` parameter. If you set `ConsistentRead` to true, DynamoDB returns a response with the most up-to-date data, reflecting the updates from all prior write operations that were successful."

> "Eventually consistent reads are **half the cost** of strongly consistent reads."

Three properties of this knob differ from what a Cassandra-trained reader expects. **The knob is read-only** — there is no write-consistency parameter; a successful write that returns HTTP 200 "completed successfully and has been durably persisted" ✅, and the documentation names the isolation it comes with: "DynamoDB provides *read-committed* isolation… **Read-committed isolation does not prevent modifications of the item immediately after the read operation.**" ✅ That last sentence is the vendor stating the limitation of its own guarantee, and is quoted rather than tidied. **The strong setting is silently unavailable on two access paths** — GSIs and streams are *always* eventually consistent ✅, with no error and no effect from `ConsistentRead`. That is the most consequential DynamoDB consistency fact for design, because GSIs are the standard way to answer "find rows by a non-key attribute"; an application that maintains a GSI and then assumes it can read it strongly has a race no configuration change can fix. **"Strongly consistent" is per-item and single-region** — "the most up-to-date data, reflecting the updates from all prior write operations that were successful" ✅ is a statement about one item, and says nothing about a `Query` spanning multiple items being a consistent snapshot.

### 5.2 Global tables: MREC vs MRSC

| Mode | How writes replicate (AWS wording ✅) | Cross-Region read guarantee |
| --- | --- | --- |
| **MREC** (default) | "item changes are replicated to the other replicas **typically within a second** and are eventually consistent across Regions" ✅ | None beyond eventual |
| **MRSC** | "item changes are **synchronously** replicated to another Region before the write returns, and strongly consistent read operations on any replica always return the latest version of an item" ✅ | Strong, on any replica — at the cost of an inter-Region RTT on the write path |

**DynamoDB makes you choose the write path's latency to obtain cross-Region read strength.** "Synchronously replicated… before the write returns" ✅ is a direct statement that write latency now includes a cross-Region round trip — the same physics Cosmos DB prices explicitly (§6.4). Note also the careful wording "another Region," singular: a reader who assumes MRSC writes to every replica, by analogy with Cosmos DB's global-majority `Strong`, will mis-model the failure behaviour. Quote the vendor; do not extend it. ⚠ (the phrasing is unambiguous; this flag records that no quorum model is claimed for MRSC that AWS does not state.)

### 5.3 The pricing asymmetry as a design constraint

**First, the cost model biases teams toward weak reads for exactly the access patterns where staleness hurts most.** A GSI-backed query cannot be strengthened at any price; the paths that *can* be strengthened cost double. ⚠ The combined pressure under-consistencies user-visible queries and over-consistencies internal lookups, the inverse of what most business processes need. **Second, the cost model makes consistency changes capacity events.** Flipping a hot read from eventually to strongly consistent doubles its RCU draw — invisible in on-demand until the bill arrives, a throttling risk in provisioned tables. Any change to a production DynamoDB consistency setting belongs in the same review as a request-rate increase. This class of risk — *the consistency knob is also a capacity knob* — recurs in Cosmos DB (§6.4, where strong and bounded staleness reads consume twice the RU per read ✅) and does not arise in Cassandra, where the replica count is fixed and the cost appears as latency rather than consumption.

## 6. Azure Cosmos DB — five levels and the scope nobody reads

### 6.1 The five levels, strongest to weakest

| Level | Guarantee (vendor wording ✅) | Quorum reads | Quorum writes |
| --- | --- | --- | --- |
| **Strong** | "offers a linearizability guarantee… reads are guaranteed to return the most recent committed version of an item. A client never sees an uncommitted or partial write." ✅ | Local Minority ✅ | Global Majority ✅ |
| **Bounded staleness** | "the lag of data between any two regions is always less than a specified amount", in *K* versions or *T* time, "whichever is reached first" ✅ | Local Minority ✅ | Local Majority ✅ |
| **Session** | "within a single client session, reads are guaranteed to honor the read-your-writes, and write-follows-reads guarantees" ✅ | Single Replica (session token) ✅ | Local Majority ✅ |
| **Consistent prefix** | "Updates made as a batch within a transaction are returned consistent to the transaction in which they were committed. Write operations within a transaction of multiple documents are always visible together." ✅ | Single Replica ✅ | Local Majority ✅ |
| **Eventual** | "the client issues read requests against any one of the four replicas in the specified region. This replica could be lagging and could return stale or no data." ✅ | Single Replica ✅ | Local Majority ✅ |

Note the shape: four of five levels share an identical write quorum, so **moving between `Session`, `Consistent prefix` and `Eventual` changes nothing about durability or write latency** — the difference is entirely which replica class a read consults. The consistent-prefix definition is the easiest to misread, so the vendor's worked example is quoted in full ✅:

> "Assume two write operations are performed transactionally (all or nothing operations) on document Doc1 followed by document Doc2, within transactions T1 and T2. When client does a read in any replica, the user sees either 'Doc1 v1 and Doc2 v1' or 'Doc1 v2 and Doc2 v2' or neither document if the replica is lagging, but never 'Doc1 v1 and Doc2 v2' or 'Doc1 v2 and Doc2 v1' for the same read or query operation."

That forbids one specific inversion — "later write visible, earlier write invisible" — and guarantees nothing extra for two independently written items, because the example's guarantee is explicitly about a transaction's batch ✅.

### 6.2 Dynamic quorum and what it does to "strong"

> "Under normal circumstances, for an account with strong consistency, a write is considered committed when all regions acknowledge replication of the record. If your account has three or more regions, the system can lower the number of regions needed for a quorum when some regions are slow or not responding… This capability is known as 'dynamic quorum'…" ✅

> "When regions are removed from the quorum set as part of dynamic quorum, those regions are **no longer able to serve reads** until readded into the quorum." ✅

This is a vendor choosing availability-by-quorum-shrinkage *within* a setting named `Strong`, then stating that some replicas become read-unavailable as a result. **"Strong" means "no client sees stale data", not "all clients can always read."** Capacity-planning rule: size the region count for the quorum arithmetic, not the read distribution — in a three- or four-region account "only one region can be removed" ✅; in a five-region account "up to two unresponsive regions can be removed" ✅.

### 6.3 Bounded staleness: the K-versions / T-time bound

Three facts, all vendor ✅: the bound is expressible as *K* versions or *T* time, enforced against "whichever is reached first"; the bound is enforced with **backpressure, not errors** — "If the data lag in a region (determined per physical partition) exceeds the configured staleness value, **writes for that partition are throttled** until staleness is back within the configured upper bound"; and "staleness checks are made **only across regions and not within a region**."

The backpressure sentence is the most operationally important on the page: a reader expects staleness bounds to produce stale reads, and this one produces *write throttling* — a capacity-planning fact wearing a consistency-fact's clothes. The third fact is where the name is weakest, and the vendor says so ✓: reads "from a nonprimary region might not show the latest data from all regions. However, they always return the newest data available in that region, within the allowed staleness limit" ✅. The bound is on *replication lag*, not on how old the data can be. Two further vendor statements collapse part of the spectrum ✅: "Bounded Staleness in a multi-write account is an **anti-pattern**", and "For a single-region account, Bounded Staleness provides the same write consistency guarantees as Session and Eventual Consistency."

### 6.4 The API-mapping gap

Cosmos DB's scope limitation is stated once and easy to lose in the length of the page ✅:

> "Scope of read consistency: Read consistency applies to a **single read operation within a logical partition**."

Every guarantee in §6.1 is therefore per logical partition, per single read. A cross-partition query is outside both the consistency scope and the transaction scope (transactional batch is partition-scoped ✅). The cost model, from the vendor ✅:

| Fact (vendor wording ✅) | Implication |
| --- | --- |
| "for the same number of request units, read throughput for strong and bounded staleness is **half** that of the other consistency levels" | Strengthening reads is a capacity purchase, as in DynamoDB §5.3 |
| "the write throughput for request units is identical across all consistency levels" | The cost of `Strong` is latency and availability, not write RU |
| "For Azure Cosmos DB accounts configured with strong consistency with more than one region, the write latency is equal to **two times round-trip time (RTT) between any of the two farthest regions, plus 10 milliseconds** at the 99th percentile" | The most precise public statement of the price of strong consistency in this guide — the anchor for §12.1 |
| "Strong consistency for accounts with regions spanning more than 5,000 miles (8,000 kilometers) is **blocked by default** because of high write latency" | The strongest setting is gated behind a support request beyond a distance threshold |

Two vendor caveats round out the picture. **The per-request override applies to reads only** ✅: "An account configured for strong consistency by default still writes and replicates data synchronously to every region in the account. When the SDK client instance or request overrides this consistency with session or weaker consistency, reads are performed using a single replica" — so a read downgrade buys back read throughput but **cannot** buy back write latency. **Session tokens are partition-scoped and cache-scoped** ✅: "If the client didn't initiate a write to a physical partition, the client doesn't contain a session token in its cache and **reads to that physical partition behave as reads with Eventual Consistency**. Similarly, if the client is re-created, its cache of session tokens is also re-created."

That last sentence is the Cosmos analogue of Cassandra's `ANY` surprise: **the setting says `Session`, and the behaviour is `Eventual` under an ordinary condition.** The documented mitigation — cache and re-send the token, don't modify its contents ✅ — is an application responsibility. `Session` is the level the vendor recommends for most workloads ✅ and is the right default; the trap is treating it as free rather than as a contract the client is half responsible for honouring.
## 7. MongoDB — read concern × write concern × read preference

### 7.1 The three-axis model

MongoDB is the system in this guide where the knob surface is genuinely **three-dimensional**, and where the documentation's own framing ("the level of consistency and availability guarantees") is at its most likely to be misread. The three axes are orthogonal and independently settable:

| Axis | What it controls | Where it is set ✅ | Default ✅ |
| --- | --- | --- | --- |
| **`readConcern`** | Isolation and recency of the data a read returns — "the consistency and isolation of data read" | per operation, per session, per transaction, or globally via `setDefaultRWConcern` | `"local"` for reads against the primary and secondaries ✅ |
| **`writeConcern`** | "the level of acknowledgment requested from MongoDB for write operations" — replication and on-disk journal acknowledgement | per write, per transaction (transaction-level only ✅), or globally | `{ w: "majority" }` is the default for most deployments ✅ |
| **`readPreference`** | "how MongoDB clients route read operations to the members of a replica set" — which *member* answers, not what guarantee it offers | per operation, per client, per connection string | `primary` ✅ |

The crucial distinction, stated in MongoDB's own words on the read-preference page ✅:

> "**Read preference does not affect the visibility of data.** Clients can see the results of writes before they are acknowledged or have propagated to a majority of replica set members."

and ✅:

> "**Read preference does not affect causal consistency.**"

That is the vendor explicitly telling the reader that axis three is *not* a consistency axis. It is a routing axis. It changes latency and load distribution; it does not change what the returned document is guaranteed to reflect. A team that sets `readPreference: secondaryPreferred` "for consistency reasons" has misunderstood which axis they were turning.

The same page carries one more vendor warning that a reader should not skip, because it is the MongoDB analogue of Cassandra's `ANY` surprise ✅:

> "All read preference modes except `primary` may return stale data because secondaries replicate operations from the primary in an asynchronous process… **Ensure that your application can tolerate stale data if you choose to use a non-`primary` mode.**"

### 7.2 Read concern levels and what each actually isolates

| Level | Vendor definition ✅ | Availability constraints ✅ | The gap between the name and the guarantee |
| --- | --- | --- | --- |
| `"local"` (default) | "The query returns data from the instance with **no guarantee that the data has been written to a majority** of the replica set members. **Data may be rolled back.**" | Available with or without causal sessions and transactions | The name suggests "close to the truth"; it means "whatever this node happens to have, possibly rollback-able" |
| `"available"` | Same as `local` — no majority guarantee, may roll back | **Unavailable** with causally consistent sessions and transactions ✅ | In a sharded cluster it can return **orphaned documents** during chunk migration ✅ — a correctness failure, not merely a staleness one |
| `"majority"` | "The query returns the data acknowledged by a majority of the replica set members. **Returned documents are durable, even if a failure occurs.**" | Requires the WiredTiger storage engine ✅ | In transactions, gives its guarantees **only if the transaction commits with write concern `"majority"`** — "otherwise, `"majority"` provides **no guarantees** about data read in transactions" ✅ |
| `"linearizable"` | "returns data that reflects all successful majority-acknowledged writes that completed before the start of the read operation. The query **may wait** for concurrent writes to propagate to a majority… before returning results." | **Unavailable** with causally consistent sessions and transactions ✅; **primary only** ✅; cannot be combined with `$out` or `$merge` ✅ | The name is accurate but the *preconditions* are severe — see below |

The `"linearizable"` preconditions are the most important passage in the MongoDB section, because they mean the setting delivers its named guarantee only for a narrower query shape than most readers assume ✅:

> "Linearizable read concern guarantees only apply if read operations specify a query filter that **uniquely identifies a single document**. Additionally if none of the following criteria are met, linearizable read concern **might not read from a consistent snapshot, resulting in a document matching the filter not being returned**:
> - The query uses an **immutable field** as the search key of the query. For example, searching on the `_id` field or using `$natural`.
> - **No concurrent updates mutate the search key** of the query.
> - The search key has a **unique index** and the query uses that index."

Read that carefully: under the wrong query shape, `readConcern: "linearizable"` can return **no document at all** where a matching document exists. That is not staleness; that is a false negative on a point read. The vendor also advises ✅: "Always use `maxTimeMS` with linearizable read concern if a majority of data-bearing members are unavailable. `maxTimeMS` ensures that the operation returns an error if the read concern cannot be fulfilled, rather than **blocking indefinitely**." A read that can block indefinitely is a read that can exhaust a connection pool and take an application down — which is a reminder that the strongest setting has an availability cost measured in *threads*, not just in milliseconds.

### 7.3 Write concern and the durability/consistency conflation

`writeConcern` is the axis most often *called* a consistency setting and least often *used* as one. Its three fields ✅:

| Field | Meaning (vendor wording ✅) |
| --- | --- |
| `w` | "Requests acknowledgment that the write operation has propagated to a specified number of `mongod` instances or to `mongod` instances with specified tags." |
| `j` | "Requests acknowledgment that the write operation has been **written to the on-disk journal**." |
| `wtimeout` | "Specifies a time limit to prevent write operations from blocking indefinitely." |

For `w: "majority"` specifically, the semantics are precise ✅: "Requests acknowledgment that the calculated majority of **data-bearing voting members** have durably written the change to their local **oplog**. The members then **asynchronously** apply changes as they read them from their local oplogs."

Three consequences that are routinely missed:

**The majority is of data-bearing *voting* members, not of all members.** Hidden, delayed and priority-0 members with `votes > 0` can acknowledge majority writes ✅; delayed secondaries "can return write acknowledgment no earlier than the configured `secondaryDelaySecs`" ✅. A replica set topology with several non-voting members therefore has less write-acknowledgement headroom than its node count suggests.

**Acknowledgement is of the oplog, and oplog durability is configurable.** The linearizable-read row carries the decisive caveat ✅: "With `writeConcernMajorityJournalDefault` set to `false`, MongoDB does **not** wait for `w: "majority"` writes to be written to the on-disk journal before acknowledging the writes. As such, `"majority"` write operations **could possibly roll back** in the event of a transient loss (e.g. crash and restart) of a majority of nodes in a given replica set."

So `w: "majority"` guarantees majority *acknowledgement*; it guarantees **durability** only when journaling defaults are left on. That is precisely the durability/consistency conflation §1.2 warns about, and the vendor documents the escape hatch plainly. Any audit that reads "we use `w: majority`" as "our writes cannot be lost" needs to check `writeConcernMajorityJournalDefault` as well.

**If majority is not reached, the write may still stand.** ✅ "If the operation does not replicate to the calculated majority of replica set members before it returns a response, then the data **eventually replicates or rolls back**." A write under `w: "majority"` with a `wtimeout` that fires is not a failed write; it is an unresolvable-at-the-time write. `wtimeout` "does not abort the write" in the sense of rolling it back — it converts the wait into an error while the data may yet propagate. Applications must distinguish "definitely not written" from "unknown," and MongoDB gives them no such distinction beyond the error.

### 7.4 Causal consistency sessions

MongoDB's causal consistency is delivered by client sessions plus a specific *combination* of concerns, and the vendor publishes the combination matrix ✅. This is the most useful piece of documentation in the MongoDB consistency surface, because it converts "causal consistency" from an adjective into an auditable four-way test.

| `readConcern` | `writeConcern` | Read own writes | Monotonic reads | Monotonic writes | Writes follow reads |
| --- | --- | --- | --- | --- | --- |
| `"majority"` | `"majority"` | ✅ | ✅ | ✅ | ✅ |
| `"majority"` | `{ w: 1 }` | — | ✅ | — | ✅ |
| `"local"` | `{ w: 1 }` | — | — | — | — |
| `"local"` | `"majority"` | — | — | ✅ | — |

(Reproduced from the vendor's table ✅; the four guarantee names are Terry et al.'s session guarantees, PDIS 1994, whose taxonomy the theory guide's §5.1 owns.)

The vendor's own summary of the matrix ✅:

> "only read operations with `"majority"` read concern and write operations with `"majority"` write concern can guarantee all four causal consistency guarantees… The read concern `"majority"` and write concern `"majority"` ensure that the four causal consistency guarantees hold **even in circumstances (such as with a network partition) where two members in a replica set *transiently* believe that they are the primary**."

That is the clearest published statement in this guide of *why* the weak combinations fail, and it is mechanical rather than mysterious ✅: "while both primaries can complete writes with `{ w: 1 }` write concern, **only one primary will be able to complete writes with `"majority"` write concern**."

The mechanism deserves spelling out because it generalises (§13.1). During a partition, two nodes can each believe they are primary. Writes at `{w: 1}` **can succeed on both**, and the loser's writes roll back when the partition heals ✅. Reads at `"local"` can be served by the not-yet-demoted old primary and return data that will cease to exist ✅. The immediate, practical consequence the vendor gives is the one that hurts ✅: "Read 1 reads data from `S1` that **doesn't reflect a state after Write 1**." Read-your-own-writes is lost during a transient split-brain — not because MongoDB is broken, but because `{w: 1}` and `"local"` were never promises against that event.

A final structural caution about the axis interaction. **The write-concern default and the read-concern default are set on different axes with different strengths** ✅ — global default write concern is `majority`, global default read concern is `local`. An application that sets nothing therefore writes durably and reads rollback-able local data. That combination is row three of the matrix shifted by one: it delivers monotonic writes ✅ and nothing else. It is a defensible default for many workloads, and it is almost never the guarantee the team believes it has.

For a worked illustration, consider **Cymbal Bank** — the fictional institution used throughout this repo's examples — maintaining a fraud-scoring service on a three-member MongoDB replica set, with a scoring write and an immediate read-back for the decision record. *All figures in this example are illustrative and fictional.* If the decision path runs `readConcern: "local"` against a `secondaryPreferred` routing, the matrix says read-own-writes is ✗ ✅, so the read-back can legitimately miss a write that succeeded. The remedy is not a single setting but a *pair*: `readConcern: "majority"` plus `writeConcern: "majority"` on a session, which the matrix says returns all four guarantees ✅. The cost is that the read may now wait for majority propagation, and the write path is unchanged (it was already `majority` by default ✅) — so the entire cost lands on the read latency of one operation. That is a favourable trade, and it is only visible because the vendor published the matrix rather than an adjective.
## 8. Riak KV — r/w/pr/pw and the notfound_ok surprise

### 8.1 The quorum properties

Riak KV is the only system in this guide that exposes the **integer-quorum** shape, and it is the system whose documentation is most explicit about failure semantics because the quorum arithmetic is the whole interface. The replication-properties page publishes the full parameter set with defaults ✅:

| Parameter | Common name | Default ✅ | Vendor description ✅ |
| --- | --- | --- | --- |
| `n_val` | N | `3` | "Replication factor, i.e. the number of nodes in the cluster on which an object is to be stored" |
| `r` | R | `quorum` | "The number of servers that must respond to a read request" |
| `w` | W | `quorum` | "Number of servers that must respond to a write request" |
| `pr` | PR | `0` | "The number of primary vnodes that must respond to a read request" |
| `pw` | PW | `0` | "The number of primary vnodes that must respond to a write request" |
| `dw` | DW | `quorum` | "The number of servers that must report that a write has been successfully written to disk" |
| `rw` | RW | `quorum` | "If R and W are undefined, this parameter will substitute for both R and W during object deletes. It is extremely unlikely that you will need to adjust this parameter." |
| `notfound_ok` | | `true` | See §8.3 — the most surprising default in this guide |
| `basic_quorum` | | `false` | "If `notfound_ok` is set to `false`, Riak will be more thorough in looking for an object on multiple nodes. Setting `basic_quorum` to `true`… will instruct Riak to wait for only a `quorum` of responses to return a `notfound` error instead of N responses." |

Two defaults stand out. **`notfound_ok` defaults to `true`**, which the vendor explains in one devastating sentence ✅: "Setting to `true` (the default) is the equivalent to setting **R to 1**: if the first node to respond doesn't have a copy of the object, Riak will immediately return a `not found` error." A default that silently reduces your effective R to 1 is exactly the kind of thing this guide exists to surface, and it is covered in §8.3.

**`dw` exists and is easy to ignore**, and it is a durability knob rather than a consistency knob ✅ — "the number of servers that must report that a write has been successfully written to disk." Riak, like every system in this guide, separates the write-acknowledgement axis (`w`) from the disk-persistence axis (`dw`). An operator who sets `w=3` for "safety" has not asked for anything to be on disk. This is the same distinction as Couchbase's durability level versus regular write (§9.1) and MongoDB's `w` versus `j` (§7.3); the fact that it appears in three vendor documentation sets with three different spellings is the reason §1.2's three-axis table exists.

The vendor's framing of the whole exercise is worth quoting because it is unusually candid about what the knob is for ✅:

> "The most general trade-off to be aware of when setting these values is the trade-off between **data accuracy** and **client responsiveness**. Choosing higher values for N, R, and W will mean higher accuracy… but higher values will also entail degraded responsiveness, especially if one or more nodes is failing, because Riak has to wait for responses from more nodes."

and, on per-request setting ✅:

> "In addition to the bucket level, you can also specify replication properties **on the client side for any given read or write**."

Riak's per-operation surface is therefore genuinely per-operation, with the caveat that properties set at the bucket level via bucket types apply unless overridden ✅.

### 8.2 Strong consistency buckets and their cost

Riak's strong consistency is not a stronger quorum setting; it is a different subsystem. The reference page is explicit ✅:

> "Using Riak in this way is **fundamentally different** from adjusting replication properties and fine-tuning the availability/consistency trade-off, as it **sacrifices *all* availability guarantees** when necessary."

and the app-guide page relevant to implementation carries an unhedged vendor warning that must be quoted in full because it is the single most important sentence for anyone considering this setting ✅:

> "Riak KV's strong consistency is an **experimental feature and may be removed from the product in the future. Strong consistency is not commercially supported or production-ready.** Strong consistency is incompatible with Multi-Datacenter Replication, Riak Search, Bitcask Expiration, LevelDB Secondary Indexes, Riak Data Types and Commit Hooks. **We do not recommend its usage in any production environment.**"

That is VENDOR MARKETING's opposite: a vendor telling you not to buy the feature. The rest of the strong-consistency documentation is nonetheless coherent and, where it makes claims, makes them precisely — which is what makes the disclaimer more striking rather than less.

| Property ✅ | Statement | Why it matters |
| --- | --- | --- |
| Guarantee | "If you successfully write a value to a key in a strongly consistent system, the **next successful read of that key is guaranteed to show that write**. A client will never see out-of-date values." | This is the *per-key* linearizability claim, correctly scoped |
| Quorum | "Quorum is defined as N / 2 + 1, or `n_val` / 2 + 1. If N is set to 7, at least 4 object replicas must be available, 2 must be available if N=3" | Standard majority arithmetic |
| Failure mode | "If there is a network partition that leaves **less than a quorum** of object replicas available within an ensemble, strongly consistent operations against the keys managed by that ensemble will **fail**." | Explicit CP behaviour, and correctly stated as *fail*, not *stale* |
| Availability retained | "Consistent operations can still succeed when a **minority** of replicas in each ensemble can be offline, faulty, or unreachable." | The guarantee is not "always fails under partition" |
| Performance | "a complex consensus subsystem that typically requires more communication between Riak nodes than eventually consistent operations, which can entail a performance hit of **varying proportions**" | ⚠ "Varying proportions" is the vendor declining to quantify; recorded as-is |
| Scope | Applied "on a **per-key** basis" via bucket types ✅ | Not a cluster-wide mode |
| Quorum knobs | "these settings are **quietly ignored** for strongly consistent operations. These settings include `r`, `pr`, `w`, `rw`, and others." ✅ | **The biggest trap in this section** |

That last row deserves emphasis. **Enabling strong consistency on a bucket silently disables the quorum properties.** An operator who has spent a year tuning `r`, `w` and `pr` and then flips `consistent: true` on a bucket does not get "strong consistency plus my tuned quorums" — they get strong consistency and their tuning is dead code. The only two properties that survive are `n_val` and `return_body` ✅, and `n_val` is unusually consequential ✅:

> "Once the `n_val` property is set for a given bucket type, **it cannot be changed**. If you wish to change the `n_val` for one or more strongly consistent buckets… you will need to **create a new bucket type** with the desired `n_val`."

The vendor also recommends ✅: "We also recommend setting the `n_val` on strongly consistent buckets to at least 5." Read together with the quorum formula (N/2+1), an `n_val` of 5 means a quorum of 3, so the ensemble tolerates 2 replica failures while remaining available. An `n_val` of 3 tolerates 1. Riak's recommendation is a direct statement that a strongly consistent ensemble needs more replicas than an eventually consistent one to reach the same availability — which is the general shape of the cost in §12.2, expressed in replicate count rather than in milliseconds.

There is one more operational constraint that turns strong consistency from a performance decision into an application-architecture decision ✅:

> "**When modifying strongly consistent objects in Riak, you *must* attach a causal context.** If you attempt to modify a strongly consistent object without attaching a context to the request, the request will always fail."

and ✅:

> "**strongly consistent updates can fail even under normal conditions**, particularly in the event of concurrent updates."

The error surface is consequential. Riak's own documentation admits ✅ that "All of Riak's official client libraries currently convert errors returned by Riak into generic exceptions" and that strong-consistency-specific errors — including the *expected* failure of a conditional put under concurrent update — are indistinguishable from other errors; the recommended workaround is to "catch these exceptions on the application side and **parse server-side error messages**" and retry ✅, and critically ✅: "it is necessary to **retry the entire read/modify/put cycle**… If you perform a simple put over and over again, without reading the object, the update will continue to fail."

So Riak's strong mode imposes a **read-modify-write discipline with error-string parsing and full-cycle retry**. That is a high application-complexity price, and it is the clearest case in this guide of §12.4's point: the strongest setting is often cheap in millimetres of latency and expensive in lines of application code.

### 8.3 Sibling creation as the failure mode of a weak read

Riak is the system whose *weak* settings have the most interesting failure mode, and it is not staleness — it is **siblings**.

The replication-properties page documents the read and write scenarios explicitly ✅. Two of the read scenarios and one of the write scenarios create siblings:

| Scenario ✅ | Riak's behaviour |
| --- | --- |
| Read: "2 of 3 vnodes agree on the value, and those 2 are the first to reach the coordinating node" | "The value is returned to the client. Read repair will deal with the conflict per the later scenarios, which means that **a future read may return a different value or siblings**" |
| Read: "2 siblings or conflicting values reach the coordinating node, vector clocks indicate a fork in the object history, and `allow_mult` is set to `true`" | "All keys are returned as **siblings**, optionally with associated values" |
| Write: "A vector clock is not included with the write request and an object already exists, with `allow_mult` set to `true`" | "The new value is created as a **sibling** for future reads" |

This is the Riak-specific consequence of the `n`-way replication model with causal context: **a weak read does not merely return stale data; it can return a set of conflicting values, and the resolution is deferred to the application.** The theory guide's §5.1 owns the CRDT vocabulary (Shapiro et al., SSS 2011) for why siblings exist; the practical point here is that Riak's consistency knob has a *lower* floor than every other system in this guide. An `r=1` read on a bucket with `allow_mult = true` can hand the application two versions of the same key and require application code to merge them. That is not a bug and not a misconfiguration — it is the honest consequence of choosing availability over convergence-by-timestamp, and it is why Riak is the system that most rewards the §14 decision procedure being applied *before* the bucket type is created rather than after.

Combining §8.1's `notfound_ok` default with this section produces the sharpest trap in the Riak material: an application that sets `r=2` for read confidence, leaves `notfound_ok` at its default of `true`, and leaves `allow_mult` at `true` will sometimes get a single-stale value, sometimes a `not found`, and sometimes a sibling set — and the tunables responsible for all three behaviours are on three different settings pages. That is the cost of Riak's expressive power: **the knob surface is a quorum interface, and a quorum interface can be misconfigured in more ways than a named-level interface can.**

## 9. Couchbase — durability levels and scan_consistency

### 9.1 Key-value durability: majority vs majorityAndPersistActive

The Couchbase durability page resolved and provided substantive content ✅. Its central definition is that durability is a **write-acknowledgement-plus-persistence** setting, not a read-consistency setting ✅:

> "Couchbase Server allows you to set durability requirements for writes to buckets. These requirements make sure that Couchbase Server updates data **in memory, on disk, or both on multiple nodes** before it decides the write succeeded. The more replicas of data that Couchbase Server writes, the greater the level of data durability."

The three levels are defined in the vendor's own words ✅:

| Level ✅ | Definition ✅ | Availability ✅ |
| --- | --- | --- |
| `majority` | "A majority of Data Service nodes must store the mutation **to memory** for the write to be durable." | "You can use this level with both Couchbase and Ephemeral buckets." |
| `majorityAndPersistActive` | "A majority of Data Service nodes must store the mutation in-memory. **Also, the node that hosts the active vBucket must write and synchronize the mutation to disk.**" | "You can choose this level only for Couchbase buckets." |
| `persistToMajority` | "A majority of Data Service nodes must **save and synchronize the mutation to disk**." | "You can choose this level only for Couchbase buckets." |

Three facts about this surface are more important than the level definitions themselves.

**The levels are ordered by what happens to memory, not by replica count.** `majority` durability means "in memory on a majority"; `persistToMajority` means "on disk on a majority." An operator choosing `majority` for cost reasons gets memory-only durability on the replicas, and the vendor says exactly what that costs ✅: "If a node fails after a regular write, the data could be lost." The lineage from `majority` to `persistToMajority` is a change of *medium*, not of quorum.

**The achievable quorum is bounded by the replica count in a way the vendor documents with a table** ✅:

| Number of Replicas ✅ | Nodes required for majority ✅ |
| --- | --- |
| 0 | 1 |
| 1 | 2 |
| 2 | 2 |
| 3 or more | **Not supported** ✅ |

with the vendor's own consequence stated twice, plainly ✅:

> "As shown by the table, **if you configure a bucket with one replica and a node fails, you cannot perform a durable write to any vBucket that has data on the failed node.**"

> "If one node fails over, **durable writes fail until the failed node rejoins the cluster**."

This is a different shape of cost from every other system in the guide. In Cassandra, losing a replica raises the level needed and may fail the operation at high levels; the cluster degrades gracefully and reads continue at low levels. In Couchbase, **a one-replica bucket loses durable writes entirely on the affected vBuckets after a single failover** ✅ — and regular (asynchronous) writes keep working. So the effect of a node failure is not "latency rises" but "the durability feature is off for part of the keyspace, silently, while the application keeps writing."

The vendor then documents a footgun of its own making, and flags it with a data-loss warning ✅:

> "you can have Couchbase Server report that durable writes succeeded even if the majority of nodes are unavailable. To enable this behavior, set the `durabilityImpossibleFallback` setting for the bucket to `true`."

with the warning ✅:

> "**Potential Data Loss.** Enabling `durabilityImpossibleFallback` degrades the guarantee that durable writes offer… When enabled for a bucket, this setting makes durable writes to it during a failover **no more safe from data loss than regular asynchronous writes**. Also, because transactions require durable writes, enabling this setting means they **do not provide the same guarantees** as they do when `durabilityImpossibleFallback` is off. Use this setting only in special cases… Always turn off this setting as soon as possible."

and the effect table ✅:

| Situation ✅ | `durabilityImpossibleFallback` Disabled ✅ | Enabled ✅ |
| --- | --- | --- |
| All nodes available | Durable writes succeed | Durable writes succeed |
| One node fails over | Durable writes fail due to lack of majority | "Couchbase Server reports the durable write succeeded **without replication**" |
| Two nodes fail over | Durable writes fail | "Durable writes fail to vBuckets on failed-over nodes" |

`durabilityImpossibleFallback` is the purest example in this guide of a knob that makes a guarantee *inoperative* while leaving its name intact. It is also, to Couchbase's credit, documented with an explicit data-loss banner and the instruction "always turn off this setting as soon as possible" ✅. Any Couchbase durability audit should check this one bucket property before anything else, because a bucket with it enabled reports durable-write success for writes that have not left the node.

Two smaller facts complete the durability picture. First, **durability is composable across client and bucket and enforced at the maximum** ✅: "Either or both the client and the bucket can set durability requirements. When both specify durability, Couchbase Server enforces the **greater** of the two values." Second, **a durable write blocks concurrent writes to the same key** ✅: "If you try to write to a key that's undergoing a durable write, Couchbase Server returns a `SYNC_WRITE_IN_PROGRESS` message" and "the client can retry the operation." A key under durable write is therefore temporarily single-writer; applications that fan several writers at one key need retry logic, not just a durability setting.

### 9.2 Query consistency: not_bounded, request_plus, statement_plus

⚠ **This subsection could not be verified against a primary source this pass.** Couchbase documents a query-side consistency control — `scan_consistency` — distinct from the key-value durability levels of §9.1, and the seed URL for it (`docs.couchbase.com/server/current/n1ql/n1ql-language-reference/consistency.html`) **redirected to the Couchbase documentation portal** and returned no substantive content ✅ (fetched, empty). The `.md` variants of that page and of the index storage-modes page both returned scraper errors ✅.

Per this guide's integrity convention, the commonly-cited values are therefore **not enumerated here as fact**. What can be said, and is flagged ⚠ as unverified: the query path in Couchbase has its own consistency control because an index-based query is answered by an **index service** that is replicated separately from the data service, so an index can be behind the data even when the key-value read path is current. The *structural* reason for a separate query knob is therefore sound and follows from Couchbase's services architecture; the *specific level names and their exact guarantees* are not asserted. Any reader about to configure this should read the vendor page directly rather than trust a secondary summary.

That gap is itself worth naming as a finding rather than an inconvenience, because it is the mirror image of §2.2's observation about knob shapes. **Systems with two consistency axes — one for the key-value path and one for the query path — have a failure mode that single-axis systems do not: the two axes can be set inconsistently with each other.** A Couchbase deployment can have durable key-value writes (§9.1) and a stale index answering the queries over those very writes. The same structural hazard exists in Cosmos DB (per-request read override versus account-level write behaviour, §6.4) and in MongoDB (the read/write concern matrix, §7.4): **whenever a system exposes two knobs, the interesting bug lives in the gap between them.**

### 9.3 The index-storage-mode dependency

⚠ The storage-modes page also could not be fetched this pass ✅ (scraper error). What follows is structural and flagged accordingly. Couchbase's index service supports different storage modes, and the standard-mode indexer can serve a query from a snapshot of the index that is behind the data — which is precisely why §9.2's query-consistency control exists. The design consequence is the same one §9.2 draws: **the consistency of a query in an index-backed system is a property of the index's replication lag, not of the document store's write path** — the product-level appearance of a fact the theory guide's §7.5 (secondary indexes across partitions) owns in the abstract.
## 10. Aerospike — read mode and commit level

### 10.1 SC vs AP read modes

⚠ **This section is the least verified in the guide, and says so.** The seed URL (`aerospike.com/docs/server/develop/consistency`) resolved but returned the **Aerospike documentation portal** — quickstart, client and architecture links — without the consistency definitions ✅ (fetched, navigational only). A later attempt at `aerospike.com/docs/database/learn/architecture/consistency-model/` returned an explicit **404** ✅, consistent with the seed note that most `aerospike.com/docs` paths 404.

Per the integrity convention, the following **is not asserted as verified vendor behaviour**: the read-mode selection between a consistency-oriented mode (`SC`) and an availability-oriented mode (`AP`), and the separate `commitLevel` controlling how many replicas must acknowledge a write. The two-axis *shape* is consistent across the sources consulted; the exact enumerations are not reproduced here.

What can be stated with confidence is structural, and it is why Aerospike belongs in this guide. **Two-axis systems put read consistency and write acknowledgement under separate control** — the same structural split as MongoDB's read concern versus write concern (§7.3) and Couchbase's KV durability versus query consistency (§9.2). The consequence is identical in each case: **the interesting misconfiguration is a read mode stronger than the write path can support, or a commit level weaker than the read mode assumes.** A read at the strongest mode cannot recover information the write path never acknowledged to a majority; the read mode governs which replica answers and how the answer is validated, not whether the write reached enough replicas to be found anywhere.

The house position: **do not configure Aerospike consistency from this guide.** Read the vendor's current page directly, confirm the enumerations for the version in use, then apply §14 and §15. Inventing level definitions to fill the section would violate the convention stated at the top of the file.

### 10.2 Commit level and the write path

⚠ The specific commit-level values are not asserted. Four questions carry across every system in this guide and are the probe to put to the documentation when it is read:

| Question | Why it decides the guarantee |
| --- | --- |
| Is the commit level counted over *all* replicas or over a *master plus replicas* set? | Determines whether the acknowledgement is a majority quorum or a leader-echo |
| Does the strongest commit level include the in-memory write only, or the on-disk write? | Separates the consistency axis from the durability axis (§1.2) |
| Does a strong-mode read cross-check replicas, or trust one replica's copy? | Determines whether a divergent replica can serve a "strong" read from its own state |

### 10.3 What the knob does not cover

Two structural limits that appear in every two-axis system, to be assumed present in Aerospike until the documentation says otherwise. **A read mode does not create cross-record atomicity** — every system here with a strong single-record mode refuses to extend it across records without a separate transaction facility: Cassandra declines cross-partition transactions by design ✅, Cosmos DB scopes read consistency to "a single read operation within a logical partition" ✅, Riak's strong mode is "applied… on a per-key basis" ✅. **A commit level does not survive a topology change without a recovery mechanism** — exactly as Cassandra's quorum guarantee leans on read repair and `nodetool repair` (§3.3) and Couchbase's durability leans on the replica count staying stable (§9.1).

## 11. The databases that refuse the knob — CockroachDB and YugabyteDB

### 11.1 CockroachDB: serializable by default, AS OF SYSTEM TIME by exception

CockroachDB is the control group because it **declines to offer the knob** that §§3–10 describe. Its only general transaction isolation is serializable; serializability, parallel commits and commit-wait are owned by [`cockroachdb_guide.md`](cockroachdb_guide.md) §5 and are not re-derived here. What matters is the shape of the exception it does offer, because it is not a consistency level at all: `AS OF SYSTEM TIME` "causes statements to execute using the database contents 'as of' a specified time in the past" ✅, and its forms are a **lower bound on data age**, not a quorum count ✅.

| Form ✅ | Meaning ✅ |
| --- | --- |
| `INT` nanoseconds / negative `INTERVAL` / `STRING` | Absolute historical timestamp, or an offset "added to `statement_timestamp()`, and thus must be negative" |
| `follower_read_timestamp()` | "returns the `TIMESTAMP` `statement_timestamp() - 4.2s`… as close as possible to the present time while remaining safe for exact staleness follower reads" |
| `with_min_timestamp(TIMESTAMPTZ, [nearest_only])` | "The minimum timestamp at which to perform the bounded staleness read… **cannot be before** the provided timestamp" |
| `with_max_staleness(INTERVAL, [nearest_only])` | "The maximum staleness interval… equivalent to `with_min_timestamp(now() - INTERVAL)`" |

Three properties distinguish it from every knob in §§3–10. **It is a time bound, not an acknowledgement rule** — the mechanism that makes it safe is the **closed timestamp**, referenced in the follower-reads page as the thing a long-running `SELECT` can fall behind ✅; the read is served locally because the local replica can *prove* no future transaction will need to write before that timestamp. A proof, not a quorum. **It applies to a whole read-only transaction and never to a write** ✅: "Follower reads are 'read-only' operations; you **cannot** use them in read-write transactions", and the clause "cannot be used with" locking reads or mutation statements, which "return an error: `cannot execute {SQL STATEMENT} in a read-only transaction`" ✅. **Its strongest form is restricted to a narrow query shape** ✅: exact staleness reads need to be "at least 4.2 seconds in the past" for a high probability of local service; bounded staleness reads "must be used in a single-statement (aka implicit) transaction… must read from a single row… must not require an index join", with the vendor publishing the error text (`cannot use bounded staleness for queries that may touch more than one row or require an index join`) ✅. That restriction is the honest admission that a bounded-staleness read is a **point read by construction** — the moment a query touches more than one row, even with `LIMIT 1` as the vendor's own example shows ✅, it cannot be served under the bound.

### 11.2 Follower reads and bounded staleness in CockroachDB

| Read type ✅ | Served from ✅ | Strong or stale ✅ |
| --- | --- | --- |
| Plain `SELECT` | "Only From Leaseholder" | Strong |
| `SELECT` on a `GLOBAL` table, or a long-running `SELECT` whose read timestamp has fallen behind the closed timestamp | "From Nearest Replica" | Strong |
| `SELECT` with `AS OF SYSTEM TIME <historical-timestamp-function>` | From nearest replica | Stale |

The first two rows are the contrast with §§3–10: **CockroachDB achieves local reads at strong consistency through a mechanism other than lowering a consistency level.** `GLOBAL` tables are that mechanism ✅ — "optimized for low-latency reads from every region," with the trade in the same breath: "The tradeoff is that **writes will incur higher latencies from any given region**, since writes have to be replicated across every region to make the global low-latency reads possible." That is the same trade as Cosmos DB's `Strong` and DynamoDB's MRSC — pay on the write path, read locally — but expressed as a **per-table placement property** rather than a per-operation knob. CockroachDB's answer to "local reads without weakening my guarantee" is to move the data, not turn a dial. The dial is finer-grained and needs no schema decision; the placement is coarser and cannot be silently weakened by one careless query.

### 11.3 YugabyteDB: follower reads in YSQL

| Parameter ✅ | Default ✅ | Meaning ✅ |
| --- | --- | --- |
| `yb_read_from_followers` | `false` ✅ | "controls whether or not reading from followers is enabled" |
| `yb_follower_read_staleness_ms` | `30000` (30 s) ✅ | "sets the exact staleness to read at (the read executes at hybrid time ~`now - yb_follower_read_staleness_ms`)" |
| `ysql_follower_reads_avoid_waiting_for_safe_time` | — | "governs whether a stale read will wait at the follower, or be redirected to the leader immediately" ✅ |

The published behaviour table is two rows ✅: with `yb_read_from_followers` true **and** the transaction marked read only, "Read happens from the closest replica of the tablet, which could be leader or follower"; otherwise "Read happens from the leader." Three vendor statements each name a different cost. **The read is stale even when it need not be** ✅: "even if the tablet leader is on the closest node, you would still read from `Now() - yb_follower_read_staleness_ms`. Therefore, when follower reads are used, **the read is always stale, even if you are reading from a tablet leader**." Unlike Cosmos DB, which exposes a probabilistically-bounded-staleness metric for its weak levels ✅, YugabyteDB offers no "sometimes strong" — it offers "always at least this stale," which removes the case analysis and is strictly weaker per read. **The YSQL variant refuses fresher data on purpose** ✅: "Unlike YCQL follower reads, which support a looser consistency model, YSQL follower reads **do not return data newer than this staleness interval, even if available at a local replica, in order to ensure consistency**." **The staleness floor is bounded by the consensus heartbeat, not chosen freely** ✅: "You shouldn't set `yb_follower_read_staleness_ms` to less than **2× the `raft_heartbeat_interval_ms`** (which by default is 500 ms)." Below roughly one second the knob stops buying locality and starts buying leader round trips, transparently ✅: "If the follower is not yet caught up to `<current_time> - <staleness>`, the read is **redirected** to a different replica transparently from the end-user" — with a metric for exactly this, `consistent_prefix_failed_reads` ✅. This is the clearest example in the guide of a consistency parameter whose useful range is set by a *different* subsystem's timing constant, and of why the knob must be measured rather than reasoned about. Leader reads are made strong by lease rather than by quorum ✅: "an elected node member is guaranteed to be the leader until its lease expires… The leader lease mechanism guarantees to serve strongly consistent reads where a client can fetch reads directly from the leader, because the leader under lease will have the latest data."

### 11.4 Why the refusal is a position, not an omission

| | **Knob exposed** (Cassandra, ScyllaDB, DynamoDB, Cosmos DB, Riak, Couchbase, Aerospike) | **Knob refused** (CockroachDB, YugabyteDB) |
| --- | --- | --- |
| Consistency is | a per-operation application decision | a system property with a narrow staleness exception |
| Strength delivered by | counting acknowledgements (quorums) | time-based proofs (leases, closed timestamps) |
| Default guarantee | often the weak end (§2.3) | the strong end |
| Failure under partition | local reads may succeed and be stale | reads fail, or are served at a stated, increasing staleness ✅ |
| The characteristic bug | a read at the wrong level, failing silently | an application that cannot function with a region cut off, failing loudly |
| Cross-record atomicity | usually absent or a separate facility | present — and the reason the knob was refused |

The last row is causal, and it is the pivot of this guide. **CockroachDB refuses the per-operation knob because it also provides cross-record serializable transactions, and the two cannot coexist** — not an engineering opinion but the Bailis et al. result cited in the theory guide's §4.5: "serializable transactions—the gold standard of traditional ACID databases—are **not achievable with high availability** in the presence of network partitions" ✅, and "**besides serializability, Snapshot Isolation and Repeatable Read isolation are not HAT-compliant**, while most other isolation levels are achievable with high availability" ✅ (Bailis et al., *Highly Available Transactions*, VLDB 2014; derivation owned by the theory guide).

Two further lines from that paper frame everything in §§3–10. Its first pragmatic takeaway ✅: "**The default (and sometimes strongest) configurations of most widely deployed database systems expose a range of anomalies that can compromise application-level consistency.**" And its statement of what high availability cannot buy ✅: "highly available systems are **fundamentally unable to prevent concurrent updates to shared data items and cannot provide recency guarantees for reads**."

That second sentence is the theoretical form of the boundary this guide has walked along in every section. Recency guarantees for reads — exactly what `ConsistentRead`, `QUORUM`, `Strong` and `majority` claim — are, in the general case, not obtainable while remaining highly available. The systems in §§3–10 do not achieve the general case; they achieve **scoped** versions: recency for one key (`LOCAL_SERIAL`), one item (`ConsistentRead`), one logical partition (`Strong`), one client session (`Session`), one bucket (`consistent: true`). The scope is not a documentation detail; it is the whole of what the knob buys, and §13 turns that into a catalogue.
## 12. What the stronger setting costs

Every cost claim below is anchored to a vendor statement or a paper quoted earlier in the guide. Where a number is a vendor guarantee it is cited; where it is this guide's synthesis it is flagged.

### 12.1 The latency cost of the third replica

The pricing question is always *where does the extra round trip land*, and the answer differs by system. The single most precise public figure in this guide is Cosmos DB's: for a multi-region strong-consistency account, "the write latency is equal to **two times round-trip time (RTT) between any of the two farthest regions, plus 10 milliseconds** at the 99th percentile" ✅.

| System | Where the stronger setting's latency lands | Anchor |
| --- | --- | --- |
| Cassandra / ScyllaDB | On the operation's own wait time, and (for plain `QUORUM` in multi-DC) on the inter-DC RTT required to reach a cluster-wide quorum ✅ | §3.3 arithmetic; "Avoids latency of inter-datacenter communication" is the stated purpose of `LOCAL_QUORUM` ✅ |
| DynamoDB | On cross-Region writes under MRSC — "synchronously replicated to another Region **before the write returns**" ✅ | §5.2 |
| Cosmos DB | On cross-Region **writes** under `Strong`; per-level read latency is otherwise "less than 10 milliseconds at the 99th percentile" for all levels ✅ | §6.4; the 2×RTT + 10 ms formula ✅ |
| MongoDB | On `linearizable` reads, which "may **wait** for concurrent writes to propagate to a majority" ✅, and on `w: "majority"` writes which wait for journal acknowledgement unless `j` is relaxed ✅ | §7.2, §7.3 |
| Riak KV | "**Slightly slower performance**" plus a consensus subsystem whose hit is of "varying proportions" ✅ | §8.2 |
| CockroachDB / YugabyteDB | On writes for `GLOBAL`-style placement ✅; per-read staleness instead of latency, with the choice made at table/query level ✅ | §11.1–11.3 |

The structural point the table makes: **the latency cost of strengthening consistency is a function of *distance*, not of the setting's name.** `ALL` in a single rack is cheap; `LOCAL_QUORUM` in a single rack is cheap; the same level names across a continent are expensive. This is exactly PACELC's normal-operation trade, Abadi's subject in *Consistency Tradeoffs in Modern Distributed Database System Design* (IEEE Computer 45(2) 2012): "there is a fundamental tradeoff between consistency, availability, and latency… **This tradeoff exists even when there are no network partitions, and thus is completely separate from the tradeoffs CAP describes**" ✅.

### 12.2 The availability cost under partition

Abadi makes one further point that every capacity plan should absorb, because it undercuts the assumption that partition-behaviour tuning is the main event ✅:

> "in general, **network partitions are somewhat rare, and are often less frequent than other serious types of failure events** in DDBSs."

The consequence is that the *replication-induced* latency cost compounds every day, while the partition-availability cost is paid rarely. Yet the default settings in §2.3 are overwhelmingly explained by partition reasoning. Abadi names the error directly ✅: "it is wrong to assume that DDBSs that reduce consistency in the absence of any partitions are doing so due to CAP-based decision-making… CAP allows the system to make the complete set of ACID guarantees alongside high availability when there are no partitions."

| Setting | Availability behaviour stated by the vendor |
| --- | --- |
| Cassandra `ALL` | "Provides the highest consistency and the **lowest availability** of any other level." ✅ |
| Cassandra `ANY` | "a guarantee that a write **never fails**… Delivers the lowest consistency and highest availability." ✅ |
| Cosmos DB `Strong` + dynamic quorum | Regions dropped from the quorum set "are **no longer able to serve reads** until readded" ✅ |
| Couchbase durable write, 1 replica | "if you configure a bucket with one replica and a node fails, you **cannot perform a durable write**" ✅ |
| Riak strong consistency | "If there is a network partition that leaves less than a quorum… strongly consistent operations… will **fail**." ✅ |
| MongoDB `linearizable` | Can "**block indefinitely**" without `maxTimeMS` ✅ |
| CockroachDB bounded staleness | A partitioned replica "will continue to stay available for (**increasingly**) stale reads" ✅ |

Note the last row, which is the inverse of the others: CockroachDB's bounded-staleness read gets *more* available under partition, at the price of *growing* staleness. That is the only row in which the weak setting's cost is expressed as a rising quantity rather than as a failure.

### 12.3 The money cost

| System | The billing consequence of strengthening | Anchor |
| --- | --- | --- |
| DynamoDB | "Eventually consistent reads are **half the cost** of strongly consistent reads." ✅ Flipping a hot read doubles its RCU draw. | §5.3 |
| Cosmos DB | "for the same number of request units, read throughput for strong and bounded staleness is **half** that of the other consistency levels." ✅ Write RU cost is "identical across all consistency levels." ✅ | §6.4 |
| Cassandra / ScyllaDB / Riak / Couchbase / CockroachDB / YugabyteDB | No per-operation price; the cost appears as *provisioned replicas* and as *capacity headroom*. Riak's guidance to use `n_val >= 5` for strong buckets ✅ is a hardware bill. | §8.2 |

Two rules follow. **Rule one: in the metered systems, the consistency knob is a capacity knob, and a consistency change belongs in a capacity review.** **Rule two: in the unmetered systems, the cost is already sunk in replication factor, so the marginal cost of strengthening is chiefly latency — which makes experimentation cheap.** The two rules point to opposite deployment strategies, and which one applies is determined by the vendor's pricing model, not by the database technology.

### 12.4 The application-complexity cost nobody budgets

| System | The application-side obligation incurred by the strong setting |
| --- | --- |
| Riak KV strong buckets | Must attach causal context to every modify ✅; must parse error strings to distinguish expected conditional-put failure from a real error ✅; must retry the **entire read/modify/put cycle** ✅ |
| Couchbase durable writes | Must handle `SYNC_WRITE_IN_PROGRESS` and retry ✅; must handle "ambiguous" outcomes from timeouts, replica failure, or rebalance ✅ |
| Cassandra `SERIAL` / LWT | A `SERIAL` read "will commit the transaction as part of the read" ✅ — reads become writes |
| Cosmos DB `Session` | The **client** must cache, re-send, and not modify the session token, and re-build it after process restart, or reads silently degrade to `Eventual` ✅ |
| CockroachDB bounded staleness | Single statement, single row, no index join ✅; errors otherwise ✅ |
| YugabyteDB follower reads | Session- or transaction-level flag; no per-statement escape once set session-wide ✅; redirect metric to watch ✅ |

This is the cost that escapes every benchmark, because it is measured in code paths a benchmark does not exercise: the retry loop, the token cache, the error-string parser, the `maxTimeMS` bound. A consistency decision that does not allocate engineering time to its §12.4 row has not been fully costed.

## 13. Where the name oversells the guarantee

### 13.1 The misleading-names catalogue

This is the guide's central artefact. Each row is a setting, the guarantee a reasonable reader infers from the name, the guarantee the vendor actually documents, and the citation.

| Setting | What the name suggests | What the vendor documents | Cite |
| --- | --- | --- | --- |
| Cassandra `ANY` | "at least one replica holds it" | "written to at least one **node**"; via a hint; "**not readable until the replica nodes for that partition have recovered**" | DataStax CL page ✅ |
| Cassandra `ONE` | "a replica has the current value" | "The replicas contacted for reads **may not always have the most recent write**." | DataStax CL page ✅ |
| Cassandra `QUORUM` | "a majority of replicas agreed" | A majority **across all datacenters**; "the more datacenters, the higher number of replica nodes need to respond" | DataStax CL page ✅ |
| Cassandra `LOCAL_QUORUM` | "strong consistency" | Strong **within one datacenter**; "Fails when using `SimpleStrategy`" for reads | DataStax CL page ✅ |
| Cassandra `SERIAL` / `LOCAL_SERIAL` | "linearizable" | Linearizable **for one partition**; and a read that "will commit the transaction as part of the read" | DataStax CL page ✅; single-partition scope ✅ |
| DynamoDB `ConsistentRead: true` | "a consistent read" | "the most up-to-date data, reflecting the updates from all prior write operations that were successful" — **per item**; and **not available at all on GSIs or streams** | AWS read-consistency page ✅ |
| DynamoDB MRSC | "strong consistency across regions" | "synchronously replicated to **another** Region before the write returns" — singular | AWS read-consistency page ✅ |
| Cosmos DB `Strong` | "linearizable everywhere" | Linearizable per the TLA⁺ spec, **scoped to "a single read operation within a logical partition"**; dynamic quorum can remove regions from serving reads | Cosmos page ✅ |
| Cosmos DB `Bounded staleness` | "a system-wide bound on staleness" | "staleness checks are made **only across regions and not within a region**" | Cosmos page ✅ |
| Cosmos DB `Session` | "I read my writes" | Only if the client holds a valid session token for that partition; otherwise reads "**behave as reads with Eventual Consistency**" | Cosmos page ✅ |
| MongoDB `readConcern: "linearizable"` | "linearizable reads" | Preconditions apply; otherwise it "**might not read from a consistent snapshot, resulting in a document matching the filter not being returned**" | MongoDB read-concern page ✅ |
| MongoDB `"majority"` read concern | "durable data" | Durable in transactions **only if** the transaction commits with write concern `"majority"`; otherwise "**no guarantees**" | MongoDB read-concern page ✅ |
| MongoDB `writeConcern: "majority"` | "the write is durable" | Durability holds only if `writeConcernMajorityJournalDefault` is `true`; otherwise majority writes "**could possibly roll back**" | MongoDB read-concern page ✅ |
| MongoDB `readPreference` | "a consistency setting" | "**Read preference does not affect the visibility of data**"; "does not affect causal consistency" | MongoDB read-preference page ✅ |
| Riak `allow_mult` + weak `r` | "reads return the value" | Reads can return **siblings** — a set of conflicting values for one key | Riak replication-properties page ✅ |
| Riak `notfound_ok` (default `true`) | "R=2 means two replicas answered" | Equivalent "**to setting R to 1**" for the not-found path | Riak replication-properties page ✅ |
| Riak `consistent: true` | "strong consistency, plus my tuned quorums" | `r`, `pr`, `w`, `rw` are "**quietly ignored**"; "**not commercially supported or production-ready**" | Riak app-guide and reference pages ✅ |
| Couchbase `durabilityImpossibleFallback` | "durable writes succeed" | "**no more safe from data loss than regular asynchronous writes**"; transactions "do not provide the same guarantees" | Couchbase durability page ✅ |
| Couchbase durable write | "my write is safe" | With one replica, a single failover means "durable writes **fail until the failed node rejoins**", while regular writes keep working | Couchbase durability page ✅ |
| YugabyteDB follower reads | "local reads" | "the read is **always stale, even if you are reading from a tablet leader**" | YugabyteDB follower-reads page ✅ |

Three patterns run through the catalogue, and naming them is more useful than memorising the rows.

**Pattern one: the scope is smaller than the name.** `Strong`, `linearizable`, `majority`, `quorum` all read as system-wide and are documented as per-key, per-item, per-partition, per-session or per-region. Cassette-style mnemonics do not help; what helps is to *always ask "over what?"* and to find the sentence in the vendor's own page that answers it. Every such sentence is quoted above.

**Pattern two: the failure mode under degradation is a silent downgrade, not an error.** Cosmos DB `Session` degrades to `Eventual` for a partition a client never wrote to ✅. Couchbase durable writes stop being durable after one failover while regular writes continue ✅. MongoDB `{w:1}` writes succeed on both sides of a split brain and one side's work is later rolled back ✅. Cassandra `ANY` writes are acknowledged and unreadable ✅. In each case the application receives a success and the guarantee is absent. §14 and §15 exist because of this pattern.

**Pattern three: the vendor sometimes says it and the readers do not.** Riak's "not commercially supported or production-ready" ✅, Couchbase's "Potential Data Loss" banner ✅, MongoDB's "no guarantees about data read in transactions" ✅, AWS's "Read-committed isolation does not prevent modifications of the item immediately after the read operation" ✅, Cosmos DB's "Bounded Staleness in a multi-write account is an anti-pattern" ✅. These are not hedging phrases buried in appendices; they are on the primary pages. The failure is on the reader's side, and quoting rather than paraphrasing is the only defence.

### 13.2 Consistency is not isolation

Bailis et al. state the conflation precisely, and it is the theoretical backing for §1.2's three-axis table ✅:

> "despite its narrow scope, the CAP Theorem is **often misconstrued as a broad result regarding the ability to provide ACID database properties with high availability**; this misunderstanding has led to substantial confusion regarding **replica consistency, transactional isolation, and high availability**." ✅

and their taxonomy result, which is the operative boundary for every system in §11 ✅:

> "serializable transactions—the gold standard of traditional ACID databases—are **not achievable with high availability** in the presence of network partitions. However… **besides serializability, Snapshot Isolation and Repeatable Read isolation are not HAT-compliant**, while most other isolation levels are achievable with high availability." ✅

with the reason given ✅: "Snapshot Isolation and Repeatable Read isolation are not HAT-compliant because they require **detecting conflicts between concurrent updates** (as needed for preventing Lost Updates or Write Skew phenomena), which we show is unavailable." ✅

(Write skew under Snapshot Isolation, Berenson et al., SIGMOD 1995, and the full isolation-anomaly catalogue are owned by the theory guide §5.3 and are not re-derived here.)

| What the knob moves | What it does not move |
| --- | --- |
| How many replicas answered a read | Whether two concurrent transactions can each read a value the other invalidates (write skew) ✅ |
| Whether a read sees a specific completed write | Whether a multi-record invariant survives concurrent modifications — HAT systems "**cannot prevent concurrent updates**" at all ✅ |
| Recency of a single read | "recency guarantees for reads" in the general case, which HAT systems "**cannot provide**" ✅ |
| Which replica and what acknowledgement rule applies | The isolation level of any transaction the read participates in |

The practical test, and it is the one §14 formalises: **if the anomaly you fear involves two records and two concurrent operations, no consistency level in this guide can fix it.** Rows 1 and 4 of the table are the only things the knob can promise.

### 13.3 The single-key scope boundary

Collecting the scope statements from §§3–11 gives the guide's second reference table. Every "strong" setting in the industry is strong *somewhere*, and the somewhere is the whole of the purchase.

| System | Strongest setting's scope, in the vendor's own terms | Cite |
| --- | --- | --- |
| Cassandra | "**single partition** operations" and "**single partition** compare and swap functionality via the lightweight transaction CQL API" | Cassandra Dynamo page ✅ |
| DynamoDB | Per item; "**All reads from GSIs and streams are eventually consistent**" | AWS read-consistency page ✅ |
| Cosmos DB | "Read consistency applies to a **single read operation within a logical partition**" | Cosmos page ✅ |
| MongoDB | `linearizable` requires a filter that "**uniquely identifies a single document**", on the primary only | MongoDB read-concern page ✅ |
| Riak KV | Applied "on a **per-key basis**" | Riak strong-consistency reference ✅ |
| Couchbase | Durability is **per mutation**; query freshness is a separate axis | Couchbase durability page ✅ |
| CockroachDB / YugabyteDB | Statements and transactions are serializable, but the *staleness* escape hatch is confined to a "**single row**" (CockroachDB) or a read-only transaction under a time bound (YugabyteDB) | §11.1–11.3 ✅ |

The table reads as one sentence: **no system in this guide sells a cross-record, concurrent-safe recency guarantee through a per-operation consistency knob, and the reason is a theorem, not a vendor's laziness** ✅ (§13.2). Everything the knob buys is a scoped recency promise on one key or one item; everything involving multiple records and concurrency must come from transactions, from an application-level protocol, or from an idempotent/commutative data design. The theory guide's §5.4 ("which level does the business process actually need") and §7.6 (the cross-partition operation) own the design side of that boundary.
## 14. Choosing a knob per operation

### 14.1 The procedure

The theory guide's §5.4 owns "which level does the business process actually need" as a five-step procedure at the level of business process. This section does the same job at the level of the **individual operation**, because that is the granularity the knobs in §§3–10 actually accept. Apply it once per operation, not once per system.

| Step | Question | If the answer is yes | If the answer is no |
| --- | --- | --- | --- |
| 1 | **Does the anomaly involve two records, or two concurrent writers to one record?** | Stop. The knob cannot help (§13.2, §13.3). Reach for transactions, an idempotent protocol, or a single-key design. | Continue |
| 2 | **Is the read of a value this same actor just wrote?** | Session/causal guarantee, not a quorum: Cosmos `Session` ✅, MongoDB causal session with `majority`/`majority` ✅, or route to the primary. Cheapest fix for the most common complaint. | Continue |
| 3 | **Does the read cross a region or an index?** | Identify which axis is stale: replica lag (kv) or index lag (query) — Couchbase §9.2, MongoDB `readPreference` §7.1, Cosmos §6.4. Choices on one axis do not fix the other. | Continue |
| 4 | **What does the operation do when the strongest setting is unavailable?** | Decide now: fail the request, serve stale with a marker, or fall back to a weaker read plus a reread. Couchbase's `durabilityImpossibleFallback` ✅ is the anti-answer. | Continue |
| 5 | **Price the setting in all four currencies of §12** — latency, availability, money, application complexity. | If any currency is unaffordable, step down a level and re-run step 4. | Ship it |

Steps 1 and 2 are the ones that change the most decisions, and both are cheap. Step 1 removes the impossible requirements before any tuning starts. Step 2 removes the largest single class of *real* complaints — "my write didn't show up" — without requiring a stronger quorum at all, because read-your-own-writes is a session property, not a quorum property ✅ (Terry et al., PDIS 1994; theory guide §5.1).

### 14.2 The operation-to-level table

A default starting position per workload class, to be adjusted by §14.1. Every entry is a starting point, not an instruction.

| Operation class | Starting setting | Why | Where it can be wrong |
| --- | --- | --- | --- |
| Point read by primary key for display, own-user data | Session / read-your-writes level | The complaint is almost always "I don't see my own change", which is a session problem ✅ | If the display is shared, "own" no longer means anything |
| Point read for a decision (authorisation, balance check, fraud score) | Strongest affordable single-key read | The Cymbal Bank fraud-decision case in §7.4 ✅ — a stale authorisation is a wrong answer, not a stale display | Fails step 1 if the decision reads two records |
| List/query over an index | Query-consistency axis, with an explicit staleness budget | The staleness is index lag, not replica lag (Couchbase §9.2, DynamoDB GSI §5.1) | Unfixable on DynamoDB GSIs at any price ✅ |
| Analytical/aggregate over a large range | Bounded staleness by time (CockroachDB `with_max_staleness` ✅, YugabyteDB `yb_follower_read_staleness_ms` ✅) | Cheapest way to get locality and take load off the leader | Narrow query-shape limits (CockroachDB: single row, no index join ✅) |
| Write whose loss has financial consequence | Strongest durability/commit setting the topology supports | Durability axis, not consistency axis (§1.2) — MongoDB `w:"majority"` + journaling ✅, Couchbase `persistToMajority` ✅ | Couchbase: impossible with 1 replica after a failover ✅ |
| Write into a telemetry or event pipeline | Weakest setting, `ANY` if the pipeline tolerates unreadable-then-recovered writes | Throughput and availability ✅ | Never read at `ANY` and expect a value back ✅ |
| Cross-entity invariant (sum of postings = balance) | **Not a level.** Serialisable transaction or single-partition design | §13.2, §13.3 | Every attempt to solve this with a CL is a future incident |

### 14.3 Mixed-consistency schemas and the upgrade trap

Two systemic hazards appear whenever a system's real configuration is a mix of levels, which for every deployment in §§3–10 it is.

**Hazard one: the level that matters is the weakest one in the path, not the one on the statement.** A read at `QUORUM` over data whose last write was `ONE` ✅ is a quorum read of a possibly-absent value. A `persistToMajority` write ✅ followed by an index-backed query is durable and invisible (§9.2). A `secondaryPreferred` read ✅ of a `{w:1}` write can be served by a node that will roll back (§7.4). In each case the *stronger* setting is the one an inventory will find and the *weaker* one is the one that decides the behaviour. Any consistency inventory must record the weakest setting on each data path, and it should be derived from code, not from documentation.

**Hazard two: strengthening a write can silently strengthen the read path's obligation, and strengthening a read can silently change a timeout budget.** The upgrade trap in its sharpest form is MongoDB's, and it is a *pair*: read concern `"majority"` "provides its guarantees **only if the transaction commits with write concern `"majority"`**. Otherwise, `"majority"` provides **no guarantees** about data read in transactions" ✅. An operator who upgrades reads to `"majority"` and leaves writes at `{w:1}` has changed nothing. Conversely, the §14.1 step-5 discipline exists because the strongest read settings are the ones that can block indefinitely without a bound ✅ (MongoDB `maxTimeMS`) or wait on a quorum that no longer exists ✅ (Cassandra `ALL`, `LOCAL_QUORUM` ✅).

The operative rule: **change consistency settings as pairs (or tuples), with the interaction documented, in a change that also moves the timeout budget and the capacity estimate.** A consistency change is a behavioural change, not a configuration tidy-up.

## 15. Testing that the knob does what you think

The technique for the tests below — fault injection method, deterministic reproduction, harness design — is owned by [`chaos_engineering_guide.md`](chaos_engineering_guide.md) and [`deterministic_engineering_guide.md`](deterministic_engineering_guide.md) and is **not** re-derived here. What this section supplies is the five *assertions* specific to a consistency knob, which those guides' harnesses should carry.

### 15.1 The five tests

| # | Test | Assertion | Why the §13 catalogue makes it necessary |
| --- | --- | --- | --- |
| 1 | **Write-then-read, same client, immediate** | The client observes its own write at the configured setting | Catches the Cosmos `Session`-without-token degradation ✅ and the MongoDB `local`/`{w:1}` case ✅ |
| 2 | **Write-then-read, different client** | Either the value is visible, or the absence is *expected* under the documented scope | Distinguishes a design decision from a bug; makes §13.3's scope explicit in the test suite |
| 3 | **Read at the strongest setting with one replica unavailable** | The operation either succeeds with the current value, or fails loudly — never succeeds with a stale value | Catches dynamic-quorum-style read exclusion ✅; catches "strong read from a divergent replica" |
| 4 | **Read at the *default* setting with one replica unavailable** | The staleness observed is within the documented bound, and the success is *recorded* | Makes the silent downgrade of §13.1 pattern two visible |
| 5 | **Consistency-change as a capacity event** | After flipping the setting in a load test, RCU/RU consumption and p99 latency are as predicted | §12.3 — in metered systems the knob is a capacity knob ✅ |

Test 3 deserves emphasis because it is the one teams skip: it is easy to verify that a strong read returns current data when everything is up, and it is only under partial failure that the guarantee is doing any work. A consistency guarantee that has never been exercised with a replica down has not been tested; it has been *assumed*.

### 15.2 What to instrument

| Instrument | System(s) | What it reveals |
| --- | --- | --- |
| Redirect count for follower reads | YugabyteDB `consistent_prefix_failed_reads` ✅ | Whether the staleness bound is set below the Raft heartbeat floor and reads are silently going to the leader ✅ |
| Read-repair / anti-entropy activity, and `durabilityImpossibleFallback` per bucket | Cassandra, ScyllaDB (§3.3), Couchbase ✅ | Whether the "eventual" half of the guarantee is actually running, and whether a bucket is reporting durable-write success without replication ✅ |
| Write concern / read concern *provenance* | MongoDB | `clientSupplied` vs `customDefault` vs `implicitDefault` ✅ — tells you whether the application or a default decided the guarantee |
| Replication lag per region/partition | Cosmos DB (the portal's Consistency metrics ✅), DynamoDB global tables | Whether `Session` is degrading to `Eventual` and where |
| Error-path counters (conflict, timeout, `SYNC_WRITE_IN_PROGRESS`) | Riak, Couchbase, MongoDB | §12.4 — the application-complexity cost is measurable, and if these counters are zero the retry code is untested |

### 15.3 The failure-injection minimum

Three injections cover the majority of the §13 catalogue, and they are small enough to run routinely rather than as an event.

1. **Kill one replica, then run the read-heavy path at its production setting.** Assert test 3 and test 4 above. This exercises dynamic-quorum read exclusion (Cosmos ✅), Cassandra `ALL`/`LOCAL_QUORUM` failure, Couchbase majority-durability failure on the affected vBuckets ✅, and Riak's below-quorum failure ✅ — all with one fault.
2. **Partition a replica (or an index service) from the rest, then run the write-then-read pair.** Assert tests 1, 2 and 4. This is the injection that surfaces the silent downgrades: session tokens missing for that partition (Cosmos ✅), rolls of `{w:1}` writes on the losing primary (MongoDB ✅), index lag against durable data (Couchbase ✅).
3. **Change the setting under load, in both directions, and watch the four currencies.** Assert test 5. This catches a consistency change that is not actually a capacity event, and — the more dangerous direction — a capacity change that is also silently a consistency change. The latter is the `durabilityImpossibleFallback` shape: a settings flip made during a graceful failover that never gets flipped back ✅.

A note on interpreting the results, and it is the guide's closing operational point. **The tests can only fail in the direction of "weaker than documented".** No test can show that a setting is stronger than its documentation claims, and no test suite in this space will ever be evidence of a guarantee the vendor has not stated. What the tests *can* do is prevent the specific, enumerated, quotable gaps in §13.1 from becoming production incidents — and because each of those gaps has a vendor sentence attached to it, each can be given a named test with a citation in the assertion message. A test suite whose consistency assertions quote the vendor is a test suite that survives an engine upgrade. That is the practical form of everything above: the knob is a promise about behaviour under failure, and the only honest question is what the knob actually buys.

*End of Part I — the product reference. Part II follows: the quorum result stated precisely, the traps consolidated, the regulated-payments angle, the vendor-claim audit, the Cymbal Bank worked example, and the claims audit — because the knob is a promise about behaviour under failure, and the honest question is what the knob actually buys.*
## 16. The quorum result, stated precisely

This is the guide's headline finding, and it is the one claim that most teams in this field have wrong. **The belief that a read quorum plus a write quorum greater than the replication factor yields strong — linearizable — consistency is not generally true.** It is true in a narrow, checkable case, and every system in §§3–10 is outside that case in at least one respect. This section states exactly what the arithmetic buys, what it does not, and what has to be added before a quorum read may be called strong. ✅ (the *narrow* case and the *missing pieces* are both settled by the primary sources below — Attiya/Bar-Noy/Dolev, Lamport, and DeCandia et al. — and by the independent measurements in §16.5.)

### 16.1 What the arithmetic actually buys

Start with the statement that is true, because the whole section is a matter of not over-reading it.

> **Quorum-intersection result.** Fix one key `k`, a *fixed* set of `n` replicas, a write discipline that acknowledges only after `w` distinct replicas in that set have stored the new version, and a read discipline that contacts `r` distinct replicas in the same set and returns a version that at least one of them holds. If `r + w > n`, every read's replica set intersects every write's replica set, so at least one replica the read contacted holds the written version. The write is therefore *discoverable* by the read.

Two things follow, and no more than two. **First, read-after-write for a single key**: a read that begins after a write completed and that contacts any `r` replicas with `r + w > n` will contact at least one replica that stored the write, provided the write is still there and the replica set has not changed. **Second, the overlap is a guarantee about *contact*, not about *selection*** — the coordinator must actually compare what it received. If it returns the first response it gets, or if it never repairs the replicas it did not hear from, the intersection is present and unused.

The paper that establishes the constructive half of this is Attiya, Bar-Noy & Dolev, *Sharing memory robustly in message-passing systems*, JACM 42(1), 1995 — the "ABD" paper. Its abstract, verified this pass ✅ (bibliographic record via OpenAlex, consulted September 2026; DOI `10.1145/200836.200869`, JACM vol. 42 no. 1, pp. 124–142, January 1995, authors Attiya, Bar-Noy, Dolev; cited 530 times at that date):

> "Both are achieved by implementing a **wait-free, atomic, single-writer multi-reader register** in unreliable, asynchronous networks." … "Any wait-free algorithm based on … registers can be automatically emulated … **provided at least a majority of the processors are not faulty and remain connected**." ✅

Read the qualifiers, because they are the entire content of the "quorum reads are strong" claim:

| The ABD result gives | The ABD result assumes |
| --- | --- |
| **atomic** (in Lamport's sense, §16.2) behaviour for **one key** | a **single writer** — multi-writer atomicity is a different problem, and the paper's own abstract lists multi-writer registers as a *derived* application of the emulator rather than the emulated object itself ⚠ |
| that behaviour through **majority quorums** ("at least a majority … not faulty and remain connected") | **a majority of the processors**, which is the `r + w > n` arithmetic in different words |
| **wait-free** reads and writes | an **unreliable but fixed membership** network — the emulator is built over a processor set, not over "the first N nodes who happen to answer" |

⚠ Two precision flags. (i) The paper's full text could not be retrieved this pass (the standard mirrors 404 or are paywalled — see §23), so the *guarantee and fault model* above are quoted from the published abstract ✅ while the paper's internal protocol shape — the two-phase read that queries a quorum and then propagates the value it found — is characterised here from the standard literature, not from the paper's text this pass. (ii) The abstract is explicit that the emulated object is **single-writer multi-reader**; the multi-writer case appears in its list of things the emulator *enables*, and the classical single-writer register is the clean case.

Lamport's register hierarchy is the vocabulary that makes the gap visible, and it is quotable in his own words ✅ (*On Interprocess Communication*, Part II, §5 and §6; PDF `lamport.azurewebsites.net/pubs/interprocess.pdf`, consulted September 2026 — the report states it appeared as two articles in *Distributed Computing*, 1986):

> "The weakest possibility is a **safe** register, in which it is assumed only that a read not concurrent with any write obtains the correct value… No assumption is made about the value obtained by a read that overlaps a write…" ✅

> "The next stronger possibility is a **regular** register, which is safe … and in which a read that overlaps a write obtains either the old or new value." ✅

> "The final possibility is an **atomic** register, which is safe and in which reads and writes behave as if they occur in some definite order … there is some way of totally ordering the reads and writes so that the values returned by the reads are the same as if the operations had been performed in that order, with no overlapping." ✅

And the sentence that names the price of moving from regular to atomic ✅:

> "In general, if two successive reads overlap the same write, then a regular register allows the first read to obtain the new value and the second read the old value, while this is **forbidden with an atomic register**. In fact, Proposition 5 of Section 6 essentially states that a regular register is atomic if two successive reads that overlap the same write cannot obtain the new then the old value." ✅

That is the precise form of the widely-mumbled claim. **Intersecting quorums with a single writer give you a *regular* register: never an old value after a new one has been read, but old-then-new and new-then-old both permitted around a concurrent write.** To get the *atomic* register — a total order consistent with real time — you must forbid the new-then-old pattern, which is a statement about ordering the reads against the writes, i.e. exactly the mechanism §16.4 describes.

Lamport also names the boundary honestly, in the paper's own closing discussion ✅:

> "I have not addressed the question of **multiwriter shared registers**. It is not clear what assumptions one should make about the effect of overlapping writes." ✅

The industrial statement of the same arithmetic carries a hedge of its own. Dynamo's paper — DeCandia et al., *Dynamo: Amazon's Highly Available Key-value Store*, SOSP 2007 — says ✅ (PDF fetched this pass, consulted September 2026):

> "Setting R and W such that **R + W > N yields a quorum-like system**." ✅

*quorum-like*, not "linearly consistent". And the same paper shows that Dynamo's own read is not a pure read ✅: the coordinator "requests all existing versions of data for that key from the N highest-ranked reachable nodes … and then waits for R responses", and when it gathers multiple versions, "the divergent versions are then reconciled and **the reconciled version superseding the current versions is written back**." ✅ A Dynamo read can write. That is the mechanism §16.4 says is required — and it is present in the model while its *discipline* is left to the implementation.

### 16.2 Why it does not buy linearizability

Six independent reasons. Any one of them is sufficient to break the inference from `r + w > n` to "linearizable"; a real system in §§3–10 typically has three or four.

**1. There is no total order across concurrent writes.** Majority intersection says nothing about *ordering*. The systems in this guide resolve concurrent writes by last-write-wins or by timestamp, and LWW discards writes rather than ordering them. The measured consequence is not subtle. Jepsen's analysis of Cassandra 2.0.0 (published 24 September 2013, `aphyr.com/posts/294-call-me-maybe-cassandra`, consulted September 2026): a test mutating the same cell "using perfectly synchronized clocks, **QUORUM consistency, and a perfect lock service**" produced "1009 acknowledged / 724 survivors / **285 acknowledged writes lost**", followed by the verdict ✅:

> "Losing 28% of your supposedly committed data is **not linearizable by any definition**. Next question." ✅

Jepsen's Riak analysis (1.2.1, published 19 May 2013, `aphyr.com/posts/285-call-me-maybe-riak`) found last-write-wins losing "**71% of acknowledged writes** on a fully-connected, *healthy* cluster" ✅ and 91% across a partition. Both systems were doing exactly what the quorum arithmetic permits; the arithmetic simply never covered which of two concurrent writes survives. Lamport's Proposition 5 above is the same fact stated as a theorem rather than a measurement. VENDOR DOCUMENTATION + INDEPENDENT MEASUREMENT.

**2. Nothing is said about a read that *overlaps* a write.** The intersection argument is about a read that begins after the write finished. A read concurrent with a write is outside it — that is why the result is a *regular* register and not an atomic one (§16.1). Every production read that runs during a write is in this category. PROVEN RESULT.

**3. Sloppy quorums break the intersection invariant outright.** Dynamo's own §4.6 ✅: "To remedy this it does not enforce strict quorum membership and instead it uses a **'sloppy quorum'**; all read and write operations are performed on the **first N healthy nodes from the preference list**, which may not always be the first N nodes encountered while walking the consistent hashing ring." ✅ With hinting, a write can be counted as successful by a node that is not a replica of the key at all. The intersection is then between the read set and *whoever happened to be up*, which is not the same set. Cassandra inherits hinted handoff (`ANY` is its purest expression, §3.1/§3.5 ✅); Riak inherits fallback vnodes, and Jepsen's Riak run shows the size of the resulting loss ✅. VENDOR DOCUMENTATION + INDEPENDENT MEASUREMENT.

**4. An acknowledged write can be lost after acknowledgement.** If `w = 1`, or if the write landed with a hint on a node that never delivers it, or if a majority write was acknowledged without journal fsync (`writeConcernMajorityJournalDefault: false`, §7.3 ✅), then a completed write may not exist at read time. `r + w > n` constrains *how many replicas must answer*; it says nothing about whether the answered write survives. This is the durability/consistency conflation of §1.2, and it is the one most often mistaken for a consistency bug. INDEPENDENT MEASUREMENT (MongoDB rollback under default write concern; Jepsen, MongoDB 4.2.6, 15 May 2020 ✅) + VENDOR DOCUMENTATION.

**5. The write-back / read-repair path is optional, asynchronous, or defeated by a default.** Intersection produces a *discoverable* write; discoverability becomes visibility only if someone compares. Riak's `notfound_ok` default — "the equivalent to setting **R to 1**" ✅ (§8.1) — is the cleanest published example of a default that removes the comparison for the not-found path. Cassandra's read repair "runs in the background to make the other replicas consistent" ✅ — asynchronously, so it does not order the reads it repairs. `ONE` reads, which are the default ✅, do not intersect anything. VENDOR DOCUMENTATION.

**6. The scope is one key.** Every statement above is per key. A `Query` or `Scan` over several items is not a quorum read of anything; a cross-record invariant is untouched by any level in this guide (§13.2, §13.3 ✅). PROVEN RESULT (Bailis et al., VLDB 2014: high-availability systems "cannot provide recency guarantees for reads" in the general case ✅).

### 16.3 The conditions under which a quorum read does give more

Each condition is independently checkable, and all of them are required before the word "linearizable" may be used about a plain quorum read.

| # | Condition | Why it is required | How it fails in practice |
| --- | --- | --- | --- |
| 1 | **A single key** | The result is a per-register result (Lamport ✅; ABD ✅) | A multi-item `Query`/`Scan` is not a register read (§13.3) |
| 2 | **No sloppy quorum** — `w` and `r` count *actual replicas of that key*, and hints do not count as replicas | Intersection is with the replica set, not with "healthy nodes" (Dynamo §4.6 ✅) | Hinted handoff, `ANY`, fallback vnodes ✅ |
| 3 | **A bounded, deterministic read-repair discipline** — the read that finds an older version completes or reports the newer one | Discoverability ≠ visibility; regular→atomic requires the reads be ordered against the write (Lamport Prop. 5 ✅) | `notfound_ok` ✅; background-only repair ✅ |
| 4 | **A stable replica set across the write and the read** | The intersection argument is over one fixed set; membership change invalidates it | Cassandra ring changes ✅ (⚠ synthesis in §3.3); Scylla's membership is "based on Cassandra's gossip system rather than a consensus system", and "even though LWT operations go through Paxos, **nodes may disagree about the quorum required** for that Paxos operation" ✅ (Jepsen, Scylla 4.2-rc3, 23 December 2020) |
| 5 | **A consensus-backed path** — the read is not a plain quorum read at all, but a Paxos/Raft/leader-leased operation | Only consensus orders concurrent writes, which reason 1 above requires | Cassandra `SERIAL`/`LOCAL_SERIAL`; Riak `consistent: true` buckets; Aerospike `SC`; Cosmos DB `Strong`; MongoDB `linearizable` on the primary; CockroachDB leaseholder reads |

Condition 5 is the decisive one, and it is worth stating flatly: **in every system in this guide, the setting that actually delivers linearizable reads is a setting that routes the operation through a consensus or leader-lease path — not a setting that increases a quorum count.** Increasing the count raises the probability that a read sees a completed write; it does not create an order.

### 16.4 What it takes to obtain linearizable reads from a leaderless store

Three routes exist, and all three are more than a knob.

| Route | What the read must do | Where it is available |
| --- | --- | --- |
| **A read that may write** (read-modify-read-modify-writeback) | Contact a quorum, **discover an in-progress or unrepaired write, complete it, and return the value the completion produced**. This is the ABD shape: the read is only linearizable because it participates in ordering the write it meets | Designed into Dynamo's read path ("the reconciled version … is **written back**" ✅); realised in products only through a consensus path — Riak strong buckets require the client to attach a causal context and to retry the **entire read/modify/put cycle** ✅ (§8.2), and Cassandra's `SERIAL` read "will commit the transaction as part of the read" ✅ (§3.4) |
| **Route the read through consensus** | Use a per-key consensus protocol (Paxos/Raft) whose quorum both orders the write and answers the read | Cassandra `SERIAL`/`LOCAL_SERIAL` over LWT ✅; Riak strong-consistency buckets ✅ (with the vendor's own "not commercially supported or production-ready" warning, §8.2 ✅); Aerospike `SC` mode, which Jepsen confirmed "**does appear to provide linearizability through network partitions and process crashes**, but data loss due to process pauses and clock skew remains" ✅ (Jepsen, Aerospike 3.99.0.3, 7 March 2018) |
| **Use a leader with a lease** | Read from a single node that can prove it is still the leader, so no concurrent writer exists to order against | CockroachDB's leaseholder: the plain `SELECT` is served "Only From Leaseholder" and is strong; local *strong* reads come from a placement decision, not a level ✅ (§11.2); YugabyteDB: "an elected node member is guaranteed to be the leader until …" ✅ (§11.3). In the leader-full systems, MongoDB `readConcern: "linearizable"` is available **on the primary only** ✅ and Cosmos DB's `Strong` is a global-majority write plus a local-minority read ✅ |

The one-line version: **a leaderless store gives linearizable reads only when the read stops being a pure read** — either it completes the writes it meets, or it is served by a consensus order, or it is served by a leader whose lease excludes concurrent writers. Everything else the knob can do is on the regular-register rung of Lamport's ladder.

Kyle Kingsbury's consistency-model catalogue is the best available public scaffolding for placing a setting on that ladder rather than on a vendor's adjective: it defines the models, the phenomena that distinguish them, and the dependency graphs that generate them ✅ (`jepsen.io/consistency`, consulted September 2026; the linearizability entry there rephrases Herlihy & Wing's 1990 definition as three constraints — `SingleOrder`, `RealTime`, `RVal` — and notes both that "Linearizability is a **single-object model**, but the scope of 'an object' varies" and that the model "cannot be totally or sticky available; in the event of a network partition, some or all nodes will be unable to make progress" ✅). Both of those sentences are the theoretical form of everything in §13.3.

### 16.5 The independent measurements, by product and date

The quorum question has been answered empirically for most of this guide's systems, and the analyses are products of their version and date — which is why every row carries its version. All URLs and dates below were read from Jepsen's published index this pass ✅ (`jepsen.io/analyses`, consulted September 2026) and the findings are quoted from the reports themselves ✅.

| System (version) | Date | The finding, in the report's words or figures ✅ | Claim class |
| --- | --- | --- | --- |
| **Riak 1.2.1** | 2013-05-19 | Last-write-wins lost "**71% of acknowledged writes** on a fully-connected, *healthy* cluster"; **91%** across a partition. "Sloppy quorum … allows both components to continue writing data with full multi-node durability guarantees" — and the heal then discards one side's work | INDEPENDENT MEASUREMENT |
| **Cassandra 2.0.0** | 2013-09-24 | With synchronised clocks, `QUORUM` and a perfect lock service: **285 of 1009 acknowledged writes lost**. Counters "drift by up to 50% of the expected value" under partition. "Losing 28% of your supposedly committed data is not linearizable by any definition." ✅ | INDEPENDENT MEASUREMENT |
| **Aerospike 3.5.4** | 2015-05-04 | AP mode: "stale reads, dirty reads, and lost updates during network partitions"; the report also records that Aerospike's team "confirms that using consistency level `all` does not prevent consistency anomalies—only reduce their frequency" ✅ | INDEPENDENT MEASUREMENT |
| **Aerospike 3.99.0.3** | 2018-03-07 | SC mode confirmed per-record (per-key) linearizability **through network partitions and process crashes** ✅, with two documented losses: "it can lose updates when more than k nodes crash", and "when either process pauses or clock skew exceed 27 seconds, Aerospike could lose committed updates" ✅ | INDEPENDENT MEASUREMENT |
| **Scylla 4.2-rc3** | 2020-12-23 | "LWT exhibited split-brain in healthy clusters, and non-LWT operations were not isolated as claimed" ✅; split-brain also with LWT after membership changes (partially resolved), aborted reads with LWT (fixed 4.2.1). This is the strongest available independent evidence about ScyllaDB, which §4 flagged ⚠ as a documentation gap | INDEPENDENT MEASUREMENT |
| **MongoDB 4.2.6** | 2020-05-15 | "Even at the strongest levels of read and write concern, it failed to preserve snapshot isolation. … read skew, cyclic information flow, duplicate writes, and internal consistency violations" ✅; per-document linearizability with `linearizable` + `majority` ✅, but "MongoDB's default level of write concern was (and remains) acknowledgement by a single node, which means **MongoDB may lose data by default**" ✅ | INDEPENDENT MEASUREMENT |
| **Amazon DynamoDB** | — | **No analysis exists in Jepsen's published index** (consulted September 2026). Recorded as a fact about the index — an absence of coverage, not evidence of absence of defects (see §23) ✅ | — |
| **Azure Cosmos DB** | — | **No analysis exists in Jepsen's published index** (consulted September 2026). Same caveat ✅ | — |
| **Couchbase** | — | **No analysis exists in Jepsen's published index** (consulted September 2026). Same caveat ✅ | — |

Three reading rules for that table. **Rule one: the negative results are not endorsements.** Silence in the index means nobody has published an opaque-box test under fault injection; for DynamoDB and Cosmos DB the strongest evidence available in this guide is the vendor's own documentation, quoted in §§5–6 ✅. **Rule two: the rows are version-dated.** The Cassandra and Riak runs are 2013; both vendors subsequently changed behaviour and both report fixes, and the Cassandra findings were followed by DataStax's own testing in 2.1/2.2 (recorded in the report's comments, ⚠ vendor-stated in a third-party comment thread, not from a primary page this pass). The correct use of these rows is to establish *what the arithmetic permits*, not to score a current release. **Rule three: a passing analysis is evidence about one version under one workload.** To gain more confidence in a system's safety, the Aerospike report says plainly, "a formal specification and proof of the consensus algorithm would be helpful" ✅.

### 16.6 The one-table answer

| Family (examples) | What `R + W > N`-family reasoning actually gives | What it takes to get **linearizable reads** | The price |
| --- | --- | --- | --- |
| **Leaderless quorum store** — Cassandra/Scylla at `QUORUM`, Riak `r`+`w`>n, Dynamo-model stores | Quorum intersection: a *regular*-register standard, i.e. read-after-write for **one key** with a stable replica set and no sloppy quorum. No ordering of concurrent writes (LWW), no statement about overlapping reads ✅ | Nothing available from the quorum knob itself. Use `SERIAL`/`LOCAL_SERIAL` (Paxos) ✅, or a strong-consistency bucket ✅, or stop claiming it | Latency: Paxos rounds (Cassandra: four round trips per LWT transaction ✅). Application: causal context, full read/modify/put retry, error-string parsing ✅ |
| **Paxos-per-key / LWT store** — Cassandra LWT, Riak strong buckets, Aerospike `SC` | Linearizability **for the key**, and (in a single partition) strict serializability for an all-LWT history ✅ | This is the route. Keep the batch inside one partition; never extend the claim across keys | Availability under partition collapses by design: "if there is a network partition that leaves less than a quorum … strongly consistent operations … **will fail**" ✅; Riak recommends `n_val >= 5` ✅ |
| **Single-leader with synchronous replication** — MongoDB `linearizable` + `majority`, Cosmos DB `Strong`, DynamoDB global tables MRSC | Not a quorum argument at all: a leader orders writes and the read is served by the leader, or by a synchronously updated replica ✅ | Already there — subject to the preconditions: primary-only and a single-document, immutable-key filter for MongoDB ✅; "synchronously replicated to another Region **before the write returns**" for MRSC ✅; dynamic quorum and the 8,000 km write-latency block for Cosmos ✅ | Write latency carries an inter-region RTT ✅; read throughput halves in Cosmos DB ✅ and doubles in cost in DynamoDB ✅ |
| **Consensus-replicated log** — CockroachDB, YugabyteDB, etcd-style systems | The knob does not exist; the guarantee is the default ✅ | Already there. Local *strong* reads come from placement (`GLOBAL` tables ✅) or from a leader lease ✅, and the only relaxation offered is a **time bound**, not a level ✅ | Region-cut failure modes are loud (an application that cannot function cut off); bounded-staleness reads are restricted to a single row / single statement ✅ |
| **Bounded-staleness read** — Cosmos `Bounded staleness`, Cockroach `AS OF SYSTEM TIME`, Yugabyte follower reads | Nothing about consistency-strength: a *staleness budget* enforced against a replication property ✅ | Nothing here delivers linearizable reads. It is the correct setting for the *read-only analytics* case and the wrong one for a decision | Cosmos: the bound is enforced by **throttling writes**, and checked "only across regions and not within a region" ✅; YugabyteDB: "the read is **always stale**" ✅ |

The final line of this section is the sentence the rest of the guide elaborates: **`R + W > N` buys the intersection; linearizability costs a consensus round, a lease, or a read that writes — and no configuration value in any of these products substitutes for one of those three.**
## 17. The traps, consolidated

§13 catalogued the naming gaps. This section catalogues the **configuration traps** — the specific places where a product, set up in a way a reasonable team would set it up, delivers less than the team believes it bought. Each row gives the product, the configuration, the assumption the configuration violates, and the consequence. Where an earlier section already argued the point with its vendor quotation, the row cites that section rather than re-arguing it: **the citation is the evidence, and repeating it here would only add length.**

### 17.1 The trap table

| # | Trap | Product and configuration | The assumption it violates | Consequence in production | Source |
| --- | --- | --- | --- | --- | --- |
| **a** | **The index-backed query that cannot be strengthened at any price** | DynamoDB: a `Query` against a **GSI**, or any read of a **stream**; Couchbase: a N1QL query served from an index; Cassandra: a query served by a **local secondary index** | "I set the strong read flag, so this read is strong" | The strong setting is ignored — silently, with a success response. DynamoDB states it flatly: "**All reads from GSIs and streams are eventually consistent**" ✅. Couchbase's index service is replicated separately from the data service, so index lag is an axis of its own ✅ (⚠ the level names are unverified, §9.2). Cassandra index reads fan out to every node holding a matching value | §5.1 ✅; §9.2 ✅; §3.2 ✅ |
| **b** | **The secondary read that stays stale under a strong read preference** | MongoDB: `readConcern: "majority"` plus `readPreference: secondaryPreferred`; Cosmos DB: `Session` on **a partition the client has never written to** | "read concern sets the guarantee; read preference only picks a server" | MongoDB's own page: "**Read preference does not affect the visibility of data**" ✅ — a secondary can answer from a state that a majority-acknowledged write has not reached. Cosmos: "reads to that physical partition **behave as reads with Eventual Consistency**" ✅ | §7.1 ✅; §6.4 ✅ |
| **c** | **The quorum that is only local** | Cassandra: `LOCAL_QUORUM`, or plain `QUORUM` believed to mean "per region"; `EACH_QUORUM` writes read back at `LOCAL_QUORUM` in a different DC; DynamoDB global tables under MREC | "a majority agreed" ⟹ "every reader can see it" | A reader in another region can miss a completed write for an arbitrary period. `QUORUM` is a majority **across all datacenters** (`(sum of RFs / 2) + 1` ✅), so intent and arithmetic diverge; MREC replicates "typically within a second" with no cross-region guarantee ✅ | §3.3 ✅; §5.2 ✅ |
| **d** | **The session guarantee that does not survive a restart or a pool boundary** | Cosmos DB: the session token in the client's cache; MongoDB: a causal session on a **new or different connection**; Aerospike `SC` read modes ⚠ | "the session level is set on the account, so it holds" | Cosmos: "if the client is re-created, its cache of session tokens is also re-created" ✅ — the read silently becomes `Eventual`. MongoDB: the four causal guarantees require **both** `majority` concerns, and hold only for the session ✅ (§7.4 matrix; Jepsen confirmed that with weaker defaults the causal session "did not preserve causal consistency" ✅, 15 May 2020). Aerospike: the client-retry behaviour that Jepsen had to disable — "the Java client, by default, will retry operations, which could lead to operations being incorrectly applied multiple times" ✅ — is documented *by an independent measurement*, not by a page this guide could reach (⚠, §10) | §6.4 ✅; §7.4 ✅; §10.1 ⚠; Terry et al., PDIS 1994 ✅ for the model |
| **e** | **The staleness bound that is per region pair, not global** | Cosmos DB `Bounded staleness` | "the bound applies to the data" | "staleness checks are made **only across regions and not within a region**" ✅; a read "might not show the latest data from all regions. However, they always return the newest data available in that region" ✅. The bound is on *replication lag between regions*, and breaching it **throttles writes** rather than returning errors ✅ | §6.3 ✅ |
| **f** | **The write acknowledged before it is durable** | Cassandra `ANY` (and the periodic commitlog fsync default ⚠); MongoDB `{ w: 1 }` — **the default before the driver/deployment default was changed to `majority`** ✅; Riak `w=1`; Couchbase a durable write on a one-replica bucket after a failover | "the ACK means the write happened" | `ANY` writes are "**not readable until the replica nodes for that partition have recovered**" ✅; `{w:1}` writes on a stepped-down primary "may be rolled back" ✅ and are lost in practice (Jepsen MongoDB 4.2.6: "MongoDB may lose data by default" ✅); Riak separates `w` from `dw` so `w=3` asks for nothing on disk ✅; Couchbase "durable writes **fail until the failed node rejoins**" while regular writes keep succeeding ✅ | §3.1 ✅; §7.3 ✅; §8.1 ✅; §9.1 ✅ |
| **g** | **The read that is consistent on the primary path and stale on the derived path** | DynamoDB Streams; MongoDB change streams; Cassandra materialized views | "the database is consistent, so its change feed is too" | Every derived path is a separate replication with its own lag, and the strong read flag does not reach it: "All reads from … streams are eventually consistent" ✅. A consumer that reacts to a change and re-reads the item can see a state older than the change it just processed | §5.1 ✅; §5.3 ✅; §9.2 ✅ (the same two-axis hazard) |
| **h** | **The knob that has no effect on the query path the team actually uses** | MongoDB: `readConcern` on paths that do not accept it (and `snapshot` outside transactions ✅); DynamoDB: `ConsistentRead` on GSIs/streams; Cosmos DB: `Strong` **unavailable** where multi-region writes are enabled ✅; Couchbase: KV durability not covering index-backed reads | "I configured the knob, therefore my read path is covered" | The guarantee is absent from the path that matters, with no error anywhere. Cosmos also scopes every level: "Read consistency applies to a **single read operation within a logical partition**" ✅ | §7.2 ✅; §5.1 ✅; §6.4 ✅; §9.2 ✅ |

Three of these traps recur in the same shape (§9.2's "whenever a system exposes two knobs, the interesting bug lives in the gap between them", §12's four-currency cost, §14.3's "the level that matters is the weakest one in the path"), and rows **a**, **g** and **h** are all instances of one rule: **the setting that is absent from a path is more dangerous than the setting that is weak on it, because absence produces no error and no metric.**

### 17.2 The configuration-questions checklist, per database

The questions to put to the configuration *before* choosing a setting, and the ones to put to the change ticket afterwards. Each is written so that a "no" or a blank answer is a stop sign rather than a discussion.

| Database | Questions to ask before choosing a setting |
| --- | --- |
| **Apache Cassandra / ScyllaDB** | 1. Is this operation **single-partition**? If not, stop — no CL helps (§13.3 ✅). 2. How many datacenters, and what is the per-DC RF? A plain `QUORUM` is a majority of *all* replicas — did the team mean `LOCAL_QUORUM` (§3.3 ✅)? 3. Is the last write on this path at `ONE` or `ANY`? The weakest level in the path decides the behaviour (§14.3 ✅). 4. Does LWT appear on this path — and is anyone billing the Paxos round trips (§3.4 ✅)? 5. When did `nodetool repair` last complete, and is read repair enabled for this table? 6. For ScyllaDB: has the specific version been exercised under fault injection, or is the claim inherited from Cassandra's lineage (§4.1 ⚠; Jepsen Scylla 4.2-rc3, 23 December 2020 ✅)? |
| **Amazon DynamoDB** | 1. Does this read touch a **GSI or a stream**? Then it is eventually consistent and no flag changes that (§5.1 ✅). 2. Is the table global, and is it MREC or MRSC — and does the team understand that MRSC's write now carries a cross-region RTT (§5.2 ✅)? 3. Is the table provisioned or on-demand, and has the 2× RCU draw of the strong read been priced (§5.3 ✅)? 4. What isolation does the application think the single-item read provides? (Read-committed, and "**Read-committed isolation does not prevent modifications of the item immediately after the read operation**" ✅.) 5. Who consumes the stream, and how stale may that consumer be? |
| **Azure Cosmos DB** | 1. Is the account single-write or multi-write? (`Bounded staleness` in a multi-write account "is an **anti-pattern**" ✅, and `Strong` is unavailable with multi-region writes ✅.) 2. Which region pair does the staleness bound cover, and does the business care about the pair or the data (§6.3 ✅)? 3. Does the client hold a session token **for the partition it is reading** — and does it survive process restart (§6.4 ✅)? 4. Is the query single-partition, or is the read scope of the level being assumed to stretch across partitions (§6.4 ✅)? 5. How many regions can dynamic quorum drop before reads become unavailable, and is the capacity plan sized for the region count rather than the read traffic (§6.2 ✅)? |
| **MongoDB** | 1. Are the read concern and write concern **paired** as the causal matrix requires (§7.4 ✅)? 2. Is `readPreference` being set for a consistency reason? It does not affect data visibility (§7.1 ✅). 3. If `linearizable` is in use: is the filter a single document on an **immutable, uniquely indexed** key, and is there a `maxTimeMS` (§7.2 ✅)? 4. Is `writeConcernMajorityJournalDefault` still `true` ✅? 5. Does any transaction read with `snapshot`/`majority` while committing with a weaker write concern — which voids the guarantee ✅? 6. Are the concerns `clientSupplied` or `implicitDefault` (§15.2 ✅)? |
| **Riak KV** | 1. What is `notfound_ok`, and has anyone noticed the effective `R = 1` (§8.1 ✅)? 2. Is `allow_mult` set, and does the application actually merge siblings (§8.3 ✅)? 3. Is this a strong-consistency bucket — and therefore are `r`/`pr`/`w`/`rw` "**quietly ignored**" (§8.2 ✅)? 4. What `n_val` was fixed at bucket-type creation (it "**cannot be changed**"), and is it at least the recommended 5 (§8.2 ✅)? 5. Does the client attach a causal context and retry the **entire** read/modify/put cycle (§8.2 ✅)? 6. Does the team accept the vendor's own "**not commercially supported or production-ready**" ✅? |
| **Couchbase** | 1. What is the replica count, and therefore what happens to durable writes after **one** failover (§9.1 ✅)? 2. Is `durabilityImpossibleFallback` `true` on any bucket ✅? 3. Is the read an index-backed query rather than a KV read — and if so, which axis is stale (§9.2 ⚠)? 4. Are durable writes and index freshness configured consistently with each other (§14.3 ✅)? 5. Are `SYNC_WRITE_IN_PROGRESS` and ambiguous-timeout outcomes handled in code or only in the runbook (§12.4 ✅)? |
| **Aerospike** | 1. Which mode is the namespace in — AP or SC? 2. If AP: what happens to a read on the minority side of a partition (§10.1 ⚠; Jepsen's two analyses are the best available evidence ✅)? 3. If SC: which two-replica compromise is the deployment relying on, and what is the clock-skew and pause tolerance (§10.2 ⚠)? 4. Does any client library retry writes by default, and is the retry idempotent ✅ (Jepsen, Aerospike 3.99.0.3)? |
| **CockroachDB / YugabyteDB (the contrast case)** | 1. Is any relaxation being used at all — and if so, is it a historical bound rather than a consistency level (§11.1 ✅)? 2. For CockroachDB bounded staleness: single statement, single row, no index join ✅? 3. For YugabyteDB follower reads: is the flag set session-wide (there is no per-statement escape ✅), and is the staleness above 2× the Raft heartbeat ✅? 4. Is the application able to run when a region is cut off, and does it fail loudly or serve an increasing staleness ✅? |

The checklist is deliberately answerable from the configuration and the code, not from a vendor page — the last question in every row is the one that survives an engine upgrade.
## 18. The regulated-payments and banking angle

**Scope statement, and it is not a formality: this section is engineering guidance about which read path may tolerate which staleness. It is not a regulatory determination, not legal advice, and not a statement about any supervisor's expectations.** Where a payment system needs a compliance position — on record-keeping, on evidence, on the treatment of a provisioned entry — that position has to come from the institution's compliance and legal functions and from the applicable scheme rules, not from a consistency table. What this section does is make the *engineering* question precise, so that the compliance question can be asked about a real system rather than about a diagram.

The payment and reconciliation mechanics referenced below are owned by the neighbouring guides and are not re-derived here: [`../banking/payment_rails_guide.md`](../banking/payment_rails_guide.md), [`../banking/payments_hub_guide.md`](../banking/payments_hub_guide.md), [`../banking/iso_20022_core_processes_guide.md`](../banking/iso_20022_core_processes_guide.md), [`../banking/posting_engine_core_banking_guide.md`](../banking/posting_engine_core_banking_guide.md), [`../banking/end_to_end_banking_processes.md`](../banking/end_to_end_banking_processes.md), [`../banking/nets_singapore_guide.md`](../banking/nets_singapore_guide.md), [`../banking/singapore_fintech_payments_guide.md`](../banking/singapore_fintech_payments_guide.md), and — for the control and evidence layer — [`audit_as_code_guide.md`](audit_as_code_guide.md).

### 18.1 The distinction that decides almost every case: what the customer sees versus what authorises the movement

The single most useful cut in a banking context is not "payment versus non-payment". It is **whether the value is being shown to a human or used to decide something**.

| The read | Why eventual is usually tolerable | Why strong is usually not |
| --- | --- | --- |
| **The balance a customer SEES** on a mobile or web screen | It is a *display*: the number is presented with a timestamp, it is reconciled continuously, and the failure of the display is a support call, not a misposted transaction. The display read may use a session/read-your-writes guarantee so that the customer's own last action is visible (see §18.3) without any quorum cost | — |
| **The balance or limit used to AUTHORISE a payment** | — | The read **replaces a decision**. If a stale limit authorises a payment that the true limit prohibits, or a stale available-balance reports funds that two other in-flight authorisations have already consumed, then the system has produced a *wrong answer* and the wrong answer has a monetary consequence that cannot be reconciled away by waiting for convergence. The correct comparison is a comparison of *funds available to more than one concurrent writer* — which, per §13.2, no consistency level in this guide can provide ✅. This is the exact case where a single-key strong read is necessary and still insufficient, and where the answer is a serialising mechanism (a per-account serialisation, a reservation/hold, or a single-writer account actor) rather than a level |
| **The statement** (period-end, interest, fees) | It is computed from settled, reconciled data by a batch whose *whole point* is to be late | The statement must equal the ledger it is derived from, or the institution has two numbers for one balance |
| **The ledger** (the posting engine's authoritative record) | — | Nothing about a posted movement may be reconstructed from a possibly-rollback-able read. A read that feeds a balance must not be `local` or `{w:1}` — the MongoDB default-write-concern finding applies here: "MongoDB may lose data by default" ✅ (Jepsen, 15 May 2020; the vendor documents the rollback ✅, §7.3) |

The general statement: **display may be eventually consistent and reconciled; the authorisation check against a limit must not be, and cannot be made correct by a consistency knob alone.** The second half of that sentence is the one that matters, and it is the §13.2 theorem restated in banking language. A team that "fixes" a limit check by strengthening its read level has moved a stale read closer to the present without removing the two-concurrent-writers problem underneath it.

### 18.2 Fraud, where the two answers differ

A fraud architecture has two horizons and they want opposite settings.

| Horizon | Setting that fits | Reasoning, not rule of thumb |
| --- | --- | --- |
| **Pre-authorisation scoring** — a risk score computed while the payment waits | **Bounded staleness** is the right shape: the score may be computed from a feature set that is seconds or minutes old, and the latency budget is a product requirement (§11.1–11.3 ✅). What must *not* happen is an unbounded read: the acceptable answer is "score from data at most N seconds old, and say so" | A scoring model is a *probabilistic* judgement about a customer's behaviour; adding a bounded amount of staleness changes the model's inputs slightly, while adding a quorum round changes the payment's latency for every customer. Cosmos DB's `Bounded staleness` and CockroachDB's `with_max_staleness`/`AS OF SYSTEM TIME` express exactly this budget ✅; the fraud-domain material is in [`../banking/financial_fraud_detection_at_scale_guide.md`](../banking/financial_fraud_detection_at_scale_guide.md) |
| **Post-hoc reconciliation and case investigation** — what did we know, and when | **Strong/durable reads, and a recorded read provenance.** The reconciliation cannot be computed from data that may still move | A case file that says "the system showed a limit of X" is worthless if the read that produced X is now known to have been served from a replica that later rolled back. The controlling requirement is not freshness at read time; it is that the *read a decision used* is reproducible |

That second row is where the consistency question turns into an evidence question, and §18.4 takes it up.

### 18.3 Read-your-writes is genuinely user-facing — and the session guarantee is the correct and cheapest answer

The most common consistency complaint in a customer-facing payment journey is not "the ledger is wrong". It is **"I made a payment thirty seconds ago and my app still shows the old balance"** — and the correct fix is a *session* guarantee, not a strong read.

Three reasons the session answer is both correct and cheap:

1. **The complaint is exactly the property Terry et al. defined** ✅ — Read Your Writes: "read operations reflect previous writes" (*Session guarantees for weakly consistent replicated data*, Terry, Demers, Petersen, Spreitzer, Theimer & Welch, Xerox PARC, PDIS 1994; PDF fetched this pass, consulted September 2026). It is a property *of one actor's sequence of operations*, which is what a mobile session is.
2. **It is the cheapest thing on the menu.** Vogels states the model and the cost together ✅: "**Session consistency** … is a practical version of the previous model, where a process accesses the storage system in the context of a session. **As long as the session exists, the system guarantees read-your-writes consistency. If the session terminates because of a certain failure scenario, a new session needs to be created and the guarantees do not overlap the sessions.**" ✅ (*Eventually Consistent*, revised version, All Things Distributed, 23 December 2008, consulted September 2026.) Cosmos DB's `Session` is the level the vendor recommends for most workloads ✅; MongoDB delivers the same four guarantees with `majority`/`majority` on a session ✅; both cost one read-path change and no quorum change.
3. **It is also where the failure modes live, so it is the setting to test.** A session guarantee is not stored in the database — it is stored in the client (§6.4 ✅) or in the connection (§7.4 ✅). Every §17 row **d** case is a session guarantee evaporating at a process boundary. For a payments front end the practical requirement is therefore not "set `Session`" but "**prove that the token survives the deploy, the autoscale event, the pooled-connection reuse, and the failover**" — a testable assertion, in the shape of §15.1's tests 1 and 2.

### 18.4 The audit and evidence implication

When a read that fed a decision was served stale, the auditor's questions are not about consistency levels. They are, in order:

1. **Which data did the decision use?** (identify the record, the version, and the read path — including whether it was a KV read, an index query, or a derived/stream path, §17 rows **a**, **g**, **h**)
2. **At what consistency was that read served?** (the effective setting on that path — which is the *weakest* setting on it, §14.3 ✅, and which is often `implicitDefault` rather than what the application asked for, §15.2 ✅)
3. **Can you prove it — after the fact, for that specific request?** (a per-request record of the read path and its provenance, not a cluster-level configuration screenshot taken at the time of the incident review)

Three engineering consequences follow, and they are all cheap if designed in and expensive if retrofitted:

- **Record the read provenance with the decision.** MongoDB exposes `clientSupplied` / `customDefault` / `implicitDefault` provenance for read and write concern ✅ (§15.2) — that value, captured per decision, is the difference between evidence and reconstruction.
- **Where a decision must be reproducible, read the value that cannot move.** For a stored decision — an authorisation limit used, a balance shown at a moment of decline — read from the ledger's durable state with a majority/durable setting, or from an immutable append-only record (§14.2's "point read for a decision" row ✅), so that the value the decision used cannot be rolled back by a subsequent failover.
- **Instrument the stale-served counter, per path.** A system that can report "N reads on the limits path were served from a replica more than *T* behind" has an answer for question 1 above; one that cannot has a control gap that no consistency setting closes. The instrumentation pattern (assertion-with-citation tests, per-path counters, and an evidence trail) is developed in [`chaos_engineering_guide.md`](chaos_engineering_guide.md), [`deterministic_engineering_guide.md`](deterministic_engineering_guide.md) and [`audit_as_code_guide.md`](audit_as_code_guide.md).

**The summary position for a payments team:** use the weakest setting that satisfies the *business* question on each path, write down the answer and the reason, keep the display reads weak and the decision reads strong, buy read-your-writes with a session guarantee rather than a quorum, and store the read provenance next to the decision. That combination is cheaper, faster, and far easier to defend to an auditor than a uniform "we set everything to strong" — which, as §12.4 shows, is also the configuration most likely to have been left half-applied.
## 19. The vendor-claim audit

This section separates **documentation** from **marketing** for each product in scope, and it does so in both directions: where a vendor's marketing overstates what the setting delivers, and where a vendor's documentation is more honest than its users are. The second column is not a courtesy — the existing sections found several cases where the vendor's own page contains the sentence that condemns the assumption, and those sentences are the strongest evidence in this whole guide because they are not disputable.

**Claim classes here:** VENDOR MARKETING (the vendor's own framing, promotional register); VENDOR DOCUMENTATION (the vendor's technical definition, citable); INDEPENDENT MEASUREMENT (an opaque-box test by a third party, version-dated).

### 19.1 Where the marketing overstates the guarantee

| Product | The claim | The claim class | What the documentation actually defines | What the guide settled on |
| --- | --- | --- | --- | --- |
| **Cassandra** | `EACH_QUORUM` and `LOCAL_QUORUM` rows are labelled "**Strong consistency**" in the write-level table ✅ | VENDOR DOCUMENTATION (the word appears in the level table itself, not only in marketing) | A quorum *within one datacenter*, and — for plain `QUORUM` — a majority across **all** datacenters: "the more datacenters, the higher number of replica nodes need to respond" ✅ | "Strong" is doing more work than the level can carry; the guide treats `LOCAL_QUORUM` as "strong within one DC" and quotes the row rather than adopting its adjective (§3.1, §3.5) |
| **Cassandra (LWT)** | "Used to achieve **linearizable consistency** for lightweight transactions" ✅ | VENDOR DOCUMENTATION | The same page's architecture statement: Cassandra offers "**single partition** operations" and "single partition compare-and-swap functionality via the lightweight transaction CQL API" ✅ | Correct, and correctly scoped — the guide accepts the word *only* with the partition qualifier attached (§3.4) |
| **Scylla** | "In an UPDATE statement, all updates within the same partition key are applied **atomically and in isolation**" ✅ (quoted in the Jepsen report from Scylla's own DML documentation) | VENDOR DOCUMENTATION, since corrected | Jepsen: "non-LWT operations were not isolated as claimed. … Scylla's documentation **no longer claims** non-LWT operations are isolated" ✅ (Scylla 4.2-rc3, 23 December 2020) | The guide records the claim, its independent test, and the correction: a documentation change following a measurement is evidence the measurement was right |
| **MongoDB** | "**full ACID transactions**"; "the only database that fully combines the power of the document model and a distributed systems architecture with ACID guarantees"; "**strong consistency by design**" ✅ | VENDOR MARKETING | The ACID whitepaper's own clarification: transactions offer **snapshot isolation** ✅. Jepsen: even at the strongest read and write concern, MongoDB 4.2.6 "failed to preserve snapshot isolation" ✅, and "**MongoDB may lose data by default**" ✅ | The guide uses "snapshot isolation" and quotes the vendor's marketing only as an example of the gap (§7.2 original content, §19.1 here) |
| **MongoDB (reporting)** | The vendor's "MongoDB and Jepsen" page "discusses only passing results, makes no mention of read or write concern, buries the actual report in a footnote", and claims "among the strongest data consistency, correctness, and safety guarantees of any database available today" ✅ | VENDOR MARKETING (per an INDEPENDENT MEASUREMENT) | The Jepsen report it summarises ✅ | Recorded as a reader-side hazard: **a vendor's summary of an independent test is not the test** |
| **Aerospike** | "can guarantee **complete data consistency** by involving all replicas of a record during each transaction"; the term "ACID" applied to the AP mode ✅ | VENDOR DOCUMENTATION, since corrected | Jepsen 2018: "Aerospike's team confirms that using consistency level `all` does **not** prevent consistency anomalies—only reduce their frequency"; "These documentation errors were quickly addressed, and Aerospike's web site **now accurately describes** the current AP behavior" ✅ | Treated as a corrected-documentation case, with the measurement as the reference for the AP-mode failure modes ✅ (§10.1) |
| **DynamoDB** | The setting is named **"strongly consistent read"**, and the guarantee is "the most up-to-date data, reflecting the updates from all prior write operations that were successful" ✅ | VENDOR DOCUMENTATION | Same page: the guarantee is **per item**; "**All reads from GSIs and streams are eventually consistent**" ✅; and the isolation is read-committed, where "Read-committed isolation does not prevent modifications of the item immediately after the read operation" ✅ | The guide uses the vendor's exact sentence and adds the two scope limits it omits from the sentence itself (§5.1) |
| **Cosmos DB** | `Strong` "offers a **linearizability** guarantee" ✅ | VENDOR DOCUMENTATION | The same page scopes it: "Read consistency applies to a **single read operation within a logical partition**" ✅; `Strong` is blocked beyond 8,000 km by default ✅; dynamic quorum removes regions from serving reads ✅ | Accepted in full, with the scope attached every time the word appears (§6.1–6.4) |
| **Riak KV** | Strong consistency "a value is guaranteed readable by any client **immediately** after a successful write" ✅ | VENDOR DOCUMENTATION | "Strong consistency is **not commercially supported or production-ready**. … **We do not recommend its usage in any production environment.**" ✅ | Accepted and quoted in full; the guide's position is that no reader may cite the guarantee without the warning (§8.2) |

### 19.2 The sentences the guide had to read carefully, and what it settled on

These are the passages where a first reading and a second reading differ. Each is listed with the resolution the guide adopted, because a guide that quotes a sentence and then quietly ignores a clause in it is worse than one that never quoted it.

| Vendor sentence ✅ | The careful reading | The guide's resolution |
| --- | --- | --- |
| DynamoDB: "Read-committed isolation **does not prevent modifications of the item immediately after the read operation**" | A vendor stating the limitation of its own strongest read — it is a *timing* statement, not a defect | Quoted as the honest upper bound of `ConsistentRead`; used in §5.1 and §17 |
| MongoDB: `linearizable` "**might not read from a consistent snapshot, resulting in a document matching the filter not being returned**" | Not staleness — a false negative on a point read, and only under three named preconditions | Quoted verbatim; the preconditions are promoted to a checklist item in §17.2 |
| MongoDB: with `writeConcernMajorityJournalDefault: false`, majority writes "**could possibly roll back**" | Durability is not implied by the word *majority* | Quoted; §1.2's three-axis table exists partly because of this sentence |
| Riak: `notfound_ok` default `true` is "the equivalent to setting **R to 1**" | A default that silently overrides the quorum the operator set | Quoted; it is the paradigm case of the §17 class of "absent comparison" |
| Riak: strong mode's quorum properties are "**quietly ignored**" | Two settings, one of which is dead code, with no warning | Quoted; §17 row (h) |
| Couchbase: enabling `durabilityImpossibleFallback` makes writes "**no more safe from data loss than regular asynchronous writes**" | A bucket property that turns a guarantee off while keeping its name, plus the vendor's own "always turn off this setting as soon as possible" | Quoted with the data-loss banner (§9.1); made a §17.2 checklist question |
| Cosmos DB: "**Bounded Staleness in a multi-write account is an anti-pattern**"; "staleness checks are made **only across regions and not within a region**" | A top-level configuration that the vendor names as an anti-pattern, and a bound that is narrower than its name | Quoted; the per-region-pair narrowness becomes §17 row (e) |
| Cosmos DB: with a dropped quorum, regions "are **no longer able to serve reads** until readded into the quorum" | "Strong" means "no client sees stale data", not "all clients can always read" | Quoted; the capacity-planning rule in §6.2 |
| YugabyteDB: "the read is **always stale, even if you are reading from a tablet leader**" | Unlike probabilistic staleness, there is no "sometimes strong" case to reason about | Quoted; §11.3 |
| CockroachDB: bounded staleness "must be used in a single-statement … transaction" and "must not require an index join" ✅ | The strongest public statement that a bounded-staleness read is a point read by construction | Quoted; the §17.2 checklist item |
| AWS on MRSC: "synchronously replicated to **another** Region before the write returns" ✅ | Singular. Do not extend it to "all replicas" | Quoted; §5.2 explicitly warns against the extension |
| Jepsen on its own limitations: "tests are nondeterministic, and we **cannot prove correctness, only find errors**" ✅ (`jepsen.io/analyses`, consulted September 2026) | A measurement's absence is not a guarantee's presence | Quoted; drives §22's "rejected" column and §23's framing |

### 19.3 The verdict on vendor documentation as a class

Three findings, each of which changed how this guide is written.

**Finding one: the definitional pages are better than the marketing pages, and the difference is checkable.** Every scope limit this guide relies on — per partition, per item, per key, within one datacenter, primary-only, unavailable with multi-region writes — appears in the vendor's own **documentation** ✅. What the marketing pages add is the unqualified adjective. The practical rule for a design review: **cite the definition, never the adjective, and put the scope sentence in the design document next to the setting.**

**Finding two: the honest sentences are usually on the same page as the overstatement.** Riak's "not commercially supported or production-ready" sits on the strong-consistency app guide ✅; DynamoDB's read-committed caveat sits on the read-consistency page ✅; Cosmos DB's anti-pattern note sits on the consistency-levels page ✅. This means the overstatement is not a lie but a *selection* — and it means that a team can be wrong while having read the page.

**Finding three: an independent measurement is the only thing that moves a vendor's text.** Scylla removed its isolation claim after Jepsen tested it ✅; Aerospike "quickly addressed" its documentation errors after the 2018 analysis ✅; MongoDB updated its isolation documentation after the causal-consistency finding ✅. In each case the documentation became more precise in response to a measurement, not in response to customers. That is the argument for the testing discipline of §15, and it is also the argument for reading the version-dated measurement (§16.5) rather than the reputational summary.
## 20. The Cymbal Bank worked example

> **Illustrative figures, stated loudly and first.** Every number attributed to **Cymbal Bank** in this section is **invented for the shape of the comparison** — not a benchmark, not a customer measurement, not a capacity test, and not production telemetry. Cymbal Bank is a **fictional** institution invented for this repository's worked examples. Read every figure as "this is the *kind* of number that decides the question", never as a number to quote. Numbers attributed to vendors or to papers elsewhere in this guide are cited and tagged ✅; nothing in this section is.

**The scenario.** Cymbal Bank is launching a new customer-facing **Instant Payments Profile** service: a mobile app and an internal servicing console over an account's profile, balance, limits, and instant payment history. The team has one weekend of design and a mandate to choose a consistency setting **per operation**, in writing, before the service leaves design.

The procedure is not re-derived here. It is the theory guide's five-step business-process procedure ([`distributed_systems_engineering_guide.md`](distributed_systems_engineering_guide.md) §5.4) applied at operation granularity as §14.1's five questions, with §14.2's operation-to-level table as the starting position. The team's job is to fill in the fourth column — *where it can be wrong* — for each of its own operations.

### 20.1 The operation inventory

Constraint: the estate is **Apache Cassandra**, two datacenters, per-DC replication factor 3, six replicas of every partition cluster-wide ✅ (the arithmetic of §3.3). One DC is the primary for writes; the second is the regional read locality. The team has *not* been given a budget to re-platform the account store, but it is allowed to propose one.

| # | Operation | What the business question actually needs | Setting chosen | Cost, in the four currencies of §12 |
| --- | --- | --- | --- | --- |
| 1 | `GET /accounts/{id}/banner` — name, tier, marketing segment | A display string; a stale tier is invisible to the customer | `ONE` (session-level default for the path) | Nothing. This is the guide's "display read" row, §14.2 ✅ |
| 2 | `GET /accounts/{id}/balance/display` — the balance card | **Read-your-writes for this customer only.** The complaint to prevent is "it still shows my old balance after my transfer" (§18.3) | Session/read-your-writes: write the payment with a token, read with it — and **prove the token survives the deploy** | One client-side token store; the test in §15.1 #1 becomes a release gate |
| 3 | `GET /accounts/{id}/available-limit` for **authorise** | A *decision*: this read replaces a business decision (§18.1) | Single-key strong read — `LOCAL_QUORUM` — **plus** a per-account serialisation in the write path (LWT/conditional update on the account row) | Read: local majority, ~1 extra intra-DC hop in the tail. Write: a Paxos round ✅ (§3.4) on the account row only |
| 4 | `POST /payments` — accept and persist the payment intent | The write must not be silently discarded or rolled back | `LOCAL_QUORUM` write (durability handled by the commit log and `commitlog_sync` settings ⚠, checked separately, §17 row **f**) | Inter-DC replication is asynchronous, so the *other* DC is a staleness question, not a write-latency one |
| 5 | `GET /payments/{id}/status` — the payer polling their own payment | Read-your-writes, and the identifier is a **client-held** uuid | Session/read-your-writes on the same session token | Same token store as #2; the polling client is the same client |
| 6 | `GET /accounts/{id}/transactions/recent` — the paginated list | A **display list**. It may be one payment behind, if the UI says so | `LOCAL_ONE` read over a **partition-per-account** wide row (single-partition query, deliberately modelled that way) | Cheapest read on the path; the risk is exactly §14.3's weakest-level-in-path hazard, so the write path that feeds it is reviewed with it |
| 7 | `GET /payments/{id}` from the **servicing console** | A decision-adjacent read: an agent answering "has this payment settled?" | `LOCAL_QUORUM` (+ the console reads the same partition as #5 where possible) | The console is a low-QPS path; the extra quorum cost is immaterial |
| 8 | Pre-authorisation **fraud scoring** feature read | A probabilistic judgement with a stated age budget (§18.2) | Bounded staleness: read the feature store at a declared `now - N` | Modelled on the §11.1–11.3 family ✅ — the *shape* is the point; on Cassandra this is emulated by a replica-read with a documented lag bound ⚠ |
| 9 | **Post-hoc fraud reconciliation** over a window | Reproducible: what did the system know, and when | Strong/durable reads from the ledger path, with read provenance recorded per decision (§18.4) | Cost is on the batch, not on the customer path |
| 10 | **Statement generation** | Must equal the ledger | Batch read of reconciled, settled data at the strongest available setting | Runs off-peak; latency is free |
| 11 | **Ops dashboard** — payment volumes, error rates | Analytics; a five-second-old sum is correct enough to page on | Bounded staleness / local reads, never the primary's capacity (§11.2's `GLOBAL`-table trade ✅) | Cheap by design; explicitly **not** used for any business decision |
| 12 | **Change stream / event consumer** into the servicing data mart | A derived path with its own lag, not the store's guarantee | Eventual, with a **published lag SLO** and an alert on it | This is §17 row **g**: the derived path is where the strong setting does not reach ✅ |

Eleven of the twelve are decisions the team can defend from the vendor documentation quoted in §§3–10. The interesting work is the checks and the walk-through.

### 20.2 The §17 checklist, applied

| §17 row | Question | Cymbal's answer |
| --- | --- | --- |
| (a) index/derived path | Does any operation read a GSI-like secondary index or a stream and believe it is strong? | **Yes, and it was caught.** Operation 6 was originally `SELECT ... WHERE merchant = ?` — a table-wide scan for a customer-facing list. It was re-modelled to a partition-per-account row so that the query is a **single-partition** read; on Cassandra that converts an unpredictable scan into a scoped read ✅ (§13.3). Operation 12's consumer is now explicitly eventual, with its own lag SLO |
| (b) secondary read | Is any read routed to a replica while a strong *concern* is set? | Not in the account path (Cassandra's routing is not MongoDB's axis), but the **analytics replica** used by operation 11 was found to be serving a path that a "strong" label had been applied to in the runbook — removed from the decision paths entirely |
| (c) local-only quorum | Is any read relying on a quorum that is local to one DC while reading from the other? | The hard one. Operation 7's console is multi-region, and `LOCAL_QUORUM` in the read's own DC is the right setting — but the **payment may have been accepted in the other DC seconds earlier** ✅ (§3.3). Resolution: the console reads the partition in the region where the payment was accepted (the payer's home region), and displays the payment's own timestamp so an agent can tell "not yet visible" from "not there" |
| (d) session at a boundary | Does any session guarantee cross a restart or a pool? | **This is the release gate for #2/#5.** The token is stored server-side per customer session and re-issued on login; the acceptance test kills and restarts the client mid-journey and asserts test #1 of §15.1 ✅ |
| (e) per-pair staleness bound | Is the staleness bound per region pair rather than global? | Not applicable to the Cassandra path; recorded as a constraint that would bind if the service moved to a product that sells a bounded-staleness level ✅ (§6.3) |
| (f) acknowledged before durable | Does the write ACK mean the write happened? | Checked explicitly: no `ANY`, no `{w:1}`-equivalent, and the `commitlog_sync` / fsync settings ⚠ are on the audit list with the §17 row **f** citation attached to the assertion |
| (g) derived path | Is the change feed assumed as fresh as the store? | No — operation 12 has a lag SLO and an alert |
| (h) knob with no effect | Is any setting configured on a path it does not govern? | Yes: the team found `ALL` set on one internal reconciliation read "for safety", which made it fail whenever a single replica was down ✅ (§12.2) and bought nothing the batch did not already get from its strong ledger read |

### 20.3 One partition, walked through the operations

**The fault:** the second datacenter is cut off from the first for eleven minutes during a network maintenance window (an inter-DC partition — the case §12.2 says is rare, which is exactly why it is worth walking through once).

| # | Operation | What happens | Correct? |
| --- | --- | --- | --- |
| 1 | Banner | Served locally; older tier string possible | Yes — display, and §14.2's display row |
| 2 | Balance display (session) | Served locally **with** the token; the customer's own last write is visible because the token pins a version | Yes — this is the whole point of buying read-your-writes with a session rather than a quorum ✅ |
| 3 | Authorise (strong limit read + serialised update) | The write path needs a **`LOCAL_QUORUM` in the payer's DC** (available) and a **Paxos quorum** for the conditional update ✅. If the payer's DC is the cut-off one, the LWT fails — and the payment is **declined loudly** | Yes. A louder, rarer failure is the intended trade of §12.2 ✅ |
| 4 | Payment write | Accepted in the reachable DC at `LOCAL_QUORUM` | Yes. The partition is invisible to the write, by design ✅ |
| 5/6 | Status and recent list in the payer's own session | Status: visible via the session token. Recent list: `LOCAL_ONE` in the payer's DC shows the write (the write landed there) | Yes, and the two were deliberately placed in the same DC |
| 7 | Servicing console, other region | Reads `LOCAL_QUORUM` in its own DC — the payment written in the other DC is **not visible**, and the console cannot tell "recent" from "missing" | **Partially.** Accepted, with the timestamp displayed and the agent script saying so; the mitigation is a "read from the accepting region" action rather than a stronger level, because no Cassandra level makes a partitioned DC's write visible ✅ |
| 8 | Fraud scoring | Reads features at the declared bound; the bound itself is now violated on the cross-DC path, so the score degrades to "stale but flagged" | Yes — the score carries its data age, which is §18.2's requirement |
| 9/10 | Reconciliation and statements | Batch runs after the window closes; durable read of settled data | Yes |
| 11 | Dashboard | Degrades to local numbers with a visible lag indicator | Yes |
| 12 | Change-feed consumer | **Stops advancing for the cut-off region.** The lag alert in §20.2 row (g) is what turns this from a silent staleness into a page | Yes — this is the only place the fault should be *loud* |

**The one operation the walk-through changed:** #7's assumption. It is the §17 row (c) trap exactly — "a quorum agreed, so the other region must see it" — and no setting fixes it, because the correct answer is a product decision (read from the accepting region) rather than a level.

### 20.4 The alternatives, including one that refuses the knob

| Option | What it would buy | What it would cost | Why the team did or did not take it |
| --- | --- | --- | --- |
| **Stay on Cassandra, per-operation settings as above** | A per-operation knob for every operation, priced per §12, with the existing estate and skills | The decision-path correctness rests on LWT's single-partition scope ✅ and on the team *writing down* the session-token discipline (§12.4) | **Chosen.** Lowest total change, and every guarantee is quotable from the vendor's own page (§19) |
| **Move the account/limits store to a consensus-replicated SQL engine — the knob-refuser of §11** | Cross-record serializable transactions: the limit check and the posting could be *one* transaction, which is the §13.2 anomaly the knob cannot touch | A schema and API rewrite; local *strong* reads then require a placement decision (`GLOBAL`-style) or a leader read ✅, and the region-cut behaviour becomes a loud failure with no degraded mode ✅ | **Rejected for phase one, and recorded as the natural phase-two direction.** The decisive sentence is the §11.4 one: the refuser refuses the knob *because* it provides the cross-record atomicity that the decision path actually needs. The team's position is that the LWT-per-account workaround is correct for the account row and will not generalise to the fee-and-limit-together cases — so the trigger for phase two is written down (any operation that must read two records to authorise) |
| **Move to a fully managed multi-region store with a strong cross-region read** | Cross-region strong reads with one setting, plus a billing model that prices the change (§12.3) | The strong cross-region read is bought on the write path: "synchronously replicated to another Region **before the write returns**" ✅ — and the strong read is unavailable on the derived paths anyway (§17 rows a, g) | **Rejected.** It buys cross-region freshness the customer journey does not need (session tokens already cover the user-visible case) at the price of a cross-region RTT on every payment write |
| **Do nothing — leave the defaults** | No change cost | `ONE` reads on the decision path ✅, no token discipline, and §13.1's silent downgrades in production | **Rejected.** This is the configuration that produces the incidents the section exists to prevent |

### 20.5 The cost comparison

> **Everything in this table is invented.** The figures exist to show the *shape* of the comparison — which options move which line — and are not measurements of any system. Do not quote them.

| Line item (illustrative only) | Cassandra, per-operation settings | Consensus-replicated engine | Managed multi-region store |
| --- | --- | --- | --- |
| Marginal read cost of the decision path | ~0 (replicas already provisioned) | ~0 | 2× per strong read, billed ✅ *(direction from §12.3; the ratio is the vendor's, the absolute number is invented)* |
| Write latency on the payment path | +1 Paxos round on the account row only ✅ | +1 consensus round | +1 cross-region RTT ✅ |
| Monthly infrastructure | baseline | higher baseline (more replicas, cross-record transactions) | per-request, grows with the *decision* reads, not with the store |
| Engineering effort, first 90 days | moderate: token store, LWT retry, per-path tests | large: schema and API rewrite | small: configuration, then capacity review |
| The failure mode the team buys | loud decline when the payer's region is cut off ✅ | loud failure for the whole account | a full write-path dependency on the inter-region link |
| Risk of a silent downgrade | medium — mitigated only by §15's tests | low — nothing to downgrade | medium — derived paths and session tokens both stay weak ✅ |

Two rules from §12.3 govern the reading. **The unmetered option's cost is already sunk in replication factor**, so the marginal cost of the strong setting is latency and engineering time — which makes the Cassandra column's *decision-path* strengthening cheap and the *session-token* work (the real cost) easy to under-budget. **The metered option's cost is a capacity event**, so its column's risk is a bill rather than a latency figure.

### 20.6 The recommendation, including where the team deliberately chose weaker

1. **Decision path (#3, #7): strong, local, and serialised.** `LOCAL_QUORUM` reads plus an LWT on the account row. Justification in writing: the read *is* the decision (§18.1), the anomaly is a concurrent-writer anomaly that no level fixes (§13.2), and the failure mode when the region is cut off is a **loud decline**, which is the correct behaviour for a payment ✅.
2. **User-visible own-data paths (#2, #5): session, not strong.** Justification in writing: the complaint is read-your-writes, which is a session property ✅ (Terry et al., PDIS 1994); a quorum buys nothing the token does not, and costs a majority round trip on every poll. **The discipline — token issued at login, stored server-side, re-issued on restart — is written into the acceptance criteria, and the §15.1 #1 test is a release gate.**
3. **Display list (#6): deliberately weak, `LOCAL_ONE`, with the staleness made visible.** *This is the team's explicit choice of the weaker setting.* The written justification has three parts: (i) the business question is "show me my recent activity", for which a lag of at most one replication interval is acceptable if shown; (ii) the alternative (`LOCAL_QUORUM` on a hot list read) buys a freshness the customer cannot perceive while adding a majority round trip to the most frequently hit path; and (iii) the consequence is bounded and named — the list may omit the customer's most recent payment **until the write that fed it has replicated**, and the UI must therefore render the just-made payment from the session token's item rather than from the list. Condition (iii) is the reason the choice is defensible: a weak setting with a named, bounded consequence and a UI mitigation is an engineering decision; a weak setting with an unexamined consequence is the incident of §21.
4. **Analytics and derived paths (#11, #12): bounded staleness and eventual, with published lag SLOs.** Never used for a decision; the lag alert is the control.
5. **The phase-two trigger:** any operation that must read **two records** to authorise goes to a cross-record transactional engine — the knob-refuser — and is not solved with a consistency level. Written down so that the next team does not have to rediscover §13.2.

The last line is the one that belongs in the design record, and it is the whole of the worked example: **the team's strongest settings are on the reads that decide, its session guarantees are on the reads a customer perceives as their own, and its weakest settings are on the reads whose staleness it has written down and bounded.**
## 21. The gotchas and anti-patterns

§17 listed traps that are **in the products** — a setting, a path, a default. This section lists the traps that are **in the team**, which is a different failure class: each is a process habit whose symptom appears long before its cause is visible, and none of them is fixed by reading a vendor page. The testing technique for the last item is owned by [`chaos_engineering_guide.md`](chaos_engineering_guide.md) and [`deterministic_engineering_guide.md`](deterministic_engineering_guide.md) and is deliberately **not** re-derived here.

| # | Anti-pattern | Symptom you can see | Cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **Believing the quorum formula gives strong consistency** | The design document contains the sentence "`R + W > N`, therefore the reads are linearizable" — with no consensus path named anywhere in it | The arithmetic was learned from a summary; the paper's conditions (§16.1's table of qualifiers) and Lamport's *regular* vs *atomic* distinction (§16.1 ✅) were never read | Require the design document to name **which** mechanism gives the ordering: a `SERIAL`/LWT path, a strong-consistency bucket, a leader with a lease, or "none, and here is what that costs us". A document that cannot name one has answered the question |
| 2 | **Setting the knob globally rather than per operation** | Two operations in the same service have the same consistency setting although one displays a name and the other authorises a payment | Cluster- or account-level configuration is easier, appears in one place, and produces a screenshot that looks like governance | §14's per-operation procedure, with the *weakest setting on each path* recorded from **code**, not from documentation (§14.3 ✅). A global setting is a starting position, never a decision |
| 3 | **Reading a status page or a marketing page as a design document** | The architecture decision cites a vendor blog, a comparison table, or a "benchmark" page rather than a definitional page | Marketing pages are shorter, better illustrated, and rank higher; a definitional page qualifies everything it says | Cite the definition and paste the scope sentence into the design record (§19.3). One rule, checkable in review: **if the quoted sentence has no scope qualifier in it, it came from the wrong page** |
| 4 | **The consistency level copied from another team** | The setting arrives via a link to another service's repository, with the comment "this is what Payments uses" | Copying a working configuration is normally good engineering; here the setting is only meaningful relative to *that* workload's operation mix, DC layout, replica count and read/write ratio (§14.2 ✅) | Copy the *procedure* and the *test*, not the value. §17.2's checklist is portable between teams; `LOCAL_QUORUM` on a three-DC keyspace is not |
| 5 | **Assuming a managed service's default matches what the documentation's default implies** | A migration to a managed offering silently changes the effective setting — the driver default, the deployment default, or a service-level default the team never saw | Defaults are set by the deployment, the driver version, and the service's own recommendation, and the vendor's *documented* default may not be the one in force. MongoDB is the clean example: the documented default write concern became `majority` while "MongoDB may lose data by default" remained true of the effective behaviour on weaker settings ✅ (Jepsen, 15 May 2020), and the guide can report provenance — `clientSupplied` / `customDefault` / `implicitDefault` ✅ (§15.2) — precisely because this is a known failure | Assert the effective setting at startup and log it. A setting that is not read back from the running system is a hypothesis |
| 6 | **The migration or upgrade that changed the guarantee silently** | A version bump or a managed-service upgrade, followed weeks later by an anomaly review that cannot explain why behaviour changed | A consistency level is "a semantic contract **plus an implementation of that contract**, and only the first half survives a database migration" (§4.3 ✅). Scylla's post-measurement documentation change ✅, Aerospike's corrected consistency pages ✅ and MongoDB's documentation change after the causal-consistency finding ✅ all show the contract's *text* moving too | Re-run the §15.1 five tests on upgrade, as a gate, not as a follow-up. The assertions should carry the vendor citation in the message so that a changed sentence fails a test rather than an incident report (§15.3 ✅) |
| 7 | **The team that never tested its read path under a partition** | Every consistency assertion in the test suite passes, and every one of them runs against a healthy cluster | A consistency guarantee does no work while everything is up; its entire content is behaviour under partial failure (§15.1 test 3 ✅) | The three injections of §15.3 as routine, not as an event: kill a replica, partition a replica or index service, and flip the setting under load — with the fault-injection method itself owned by [`chaos_engineering_guide.md`](chaos_engineering_guide.md) and the deterministic reproduction harness by [`deterministic_engineering_guide.md`](deterministic_engineering_guide.md) |

Two observations tie the list together. **First, patterns 1–4 are documentation and review failures; 5–7 are operational failures.** The first group is cheap to fix (a checklist question, a citation rule) and the second is expensive, because it needs infrastructure and discipline. Most teams that believe they have the first group under control have the second group untested. **Second, every one of the seven has a written artefact as its guardrail** — a named mechanism in the design document, a per-operation table, a quoted scope sentence, a copied test, a logged effective setting, a citation in the assertion message, a scheduled injection. That is not a coincidence: this guide's subject is a promise about behaviour under failure, and the only durable way to hold a promise is to write down what it was.
## 22. The claims audit

The guide's claims, sorted by what happened to them when they were checked. **Three classes carry nearly all of the risk, and they are named first:** the **quorum result** (because the popular version of it is false in the general case, §16), **every vendor definition** (because a definition is only as good as the page it came from, and half the pages in §23 did not resolve), and **every cost figure** (because the only public numbers are the vendors' own, §12.3).

### 22.1 Verified this pass

| Claim | Source | Source quality | Date consulted |
| --- | --- | --- | --- |
| Quorum intersection gives a **regular**, not atomic, register; atomicity requires the reads to be ordered against the write (Lamport's Prop. 5) | Lamport, *On Interprocess Communication*, Part II §5–6 ✅ | PRIMARY (author's own PDF) | September 2026 |
| Majority quorums + a **single writer** yield a wait-free **atomic single-writer multi-reader** register, provided a majority of processors are not faulty ✅ | Attiya, Bar-Noy & Dolev, JACM 42(1):124–142, 1995; abstract ✅ | PRIMARY (bibliographic record + published abstract) | September 2026 |
| Dynamo: "R + W > N yields a **quorum-like** system"; sloppy quorum and hinted handoff replace strict membership; the read path reconciles and **writes back** ✅ | DeCandia et al., SOSP 2007 ✅ | PRIMARY (paper PDF) | September 2026 |
| Session guarantees (RYW, MR, WFR, MW) are per-session properties that "can be layered on existing systems that employ a read-any/write-any replication scheme" ✅ | Terry et al., PDIS 1994 ✅ | PRIMARY (paper PDF, university mirror) | September 2026 |
| Session guarantees do not overlap sessions; session consistency survives only as long as the session ✅ | Vogels, *Eventually Consistent* (revised), 23 December 2008 ✅ | PRIMARY (author's own page) | September 2026 |
| Cassandra quorum arithmetic and the multi-DC cost; `ANY` unreadable-until-recovery; `SERIAL`/`LOCAL_SERIAL` scope ✅ | DataStax Cassandra 3.x CL reference; Cassandra architecture page ✅ | VENDOR DOCUMENTATION | September 2026 (per the file's metadata paragraph) |
| DynamoDB: GSIs and streams are always eventually consistent; read-committed isolation caveat; 2× read cost ✅ | AWS read-consistency page ✅ | VENDOR DOCUMENTATION | September 2026 |
| Cosmos DB: five levels, dynamic quorum read exclusion, staleness checked only across regions, 2×RTT+10 ms, 8,000 km block, session-token degradation ✅ | Cosmos DB consistency pages ✅ | VENDOR DOCUMENTATION | September 2026 |
| MongoDB: the causal matrix, `linearizable` preconditions, journal-default rollback caveat ✅ | MongoDB manual pages ✅ | VENDOR DOCUMENTATION | September 2026 |
| Riak: `notfound_ok` default ≡ `R=1`; quorum properties "quietly ignored" in strong buckets; the vendor's own production-readiness warning ✅ | Riak KV pages ✅ | VENDOR DOCUMENTATION | September 2026 |
| Couchbase: durability levels, the one-replica failover consequence, `durabilityImpossibleFallback`'s data-loss banner ✅ | Couchbase durability page ✅ | VENDOR DOCUMENTATION | September 2026 |
| CockroachDB: `AS OF SYSTEM TIME` forms, bounded-staleness query-shape limits ✅; YugabyteDB: follower-read parameters and the "always stale" behaviour ✅ | Vendor docs ✅ | VENDOR DOCUMENTATION | September 2026 |
| Jepsen findings and **dates** for Riak 1.2.1 (2013-05-19), Cassandra 2.0.0 (2013-09-24), Aerospike 3.5.4 (2015-05-04) and 3.99.0.3 (2018-03-07), Scylla 4.2-rc3 (2020-12-23), MongoDB 4.2.6 (2020-05-15) ✅ | Jepsen's published index and the reports themselves ✅ | INDEPENDENT MEASUREMENT | September 2026 |
| Jepsen's published index lists **no** analysis of DynamoDB, Cosmos DB or Couchbase (a fact about the index, not about the products) ✅ | `jepsen.io/analyses` ✅ | INDEPENDENT (coverage statement) | September 2026 |
| The linearizability definition's three constraints (`SingleOrder`, `RealTime`, `RVal`) and the single-object scope ✅ | Jepsen consistency reference, citing Herlihy & Wing 1990 and Viotti & Vukolić ✅ | SECONDARY (well-curated tertiary reference) | September 2026 |

### 22.2 Flagged — quarantined, usable only with the flag attached

| Claim | Why flagged | What the guide did | Date |
| --- | --- | --- | --- |
| **ABD's protocol shape** (a read that queries a quorum and then propagates the value it found) | The paper's **full text did not resolve** this pass; the guarantee and fault model are from the published abstract ✅, the protocol structure from the standard literature ⚠ | Characterised, labelled ⚠, and separated from the quoted abstract | September 2026 |
| **ScyllaDB per-level definitions** | The seed path 404s; the documentation portal returns navigational content only ✅ | No Scylla-specific definition is asserted; the section is structural and tagged ENGINEERING CONVENTION ⚠ (§4.1) | September 2026 |
| **ScyllaDB's cost curve** (shard-per-core vs JVM) | Structural inference, not a measured comparison | Presented as a claim about *how teams behave*, flagged ⚠ (§4.2) | September 2026 |
| **Aerospike level enumerations** (`SC`/`AP`, commit-level values) | The definitions page resolves to a 404 or the documentation portal ✅ | No enumeration asserted; the section supplies the *probe questions* instead, and points the reader at the vendor's current page (§10.1–10.2) | September 2026 |
| **Aerospike client-retry behaviour** | Learned from an independent measurement, not from a page this pass could reach | Attributed to Jepsen ✅ with the ⚠ that the vendor page was unreachable (§17 row d) | September 2026 |
| **Couchbase `scan_consistency` level names and their guarantees** | The page redirected to the documentation portal ✅ and the `.md` variants errored ✅ | Level names not enumerated; the *structural* reason for a separate query axis is stated and flagged (§9.2) | September 2026 |
| **Couchbase index storage modes** | Page unreachable ✅ | Structural treatment only, flagged (§9.3) | September 2026 |
| **Cassandra's periodic commitlog fsync default** | Asserted in §17 row **f** with a ⚠; not re-verified against a page in this pass | Left ⚠ and marked as an audit item rather than a fact | September 2026 |
| **Cassandra `QUORUM`-plus-`QUORUM` overlap depending on a stable replica set** | Documented behaviour for repair; the *framing* that the CL alone is insufficient is this guide's synthesis | Flagged as synthesis in §3.3 and reused in §16.3 condition 4 | September 2026 |
| **YugabyteDB leader-lease wording** | Truncated in the extracted page during the earlier pass | Quoted only in part, marked ✅ for what was retrieved | September 2026 |
| **Scylla's post-Jepsen fixes** (split-brain fixed in 4.2 etc.) | Stated in the report's own summary ✅, but the version under test is a release candidate | Quoted with the version and date attached | September 2026 |
| **Cassandra fixes in 2.1/2.2 claimed by the vendor** | Appears in a third-party comment thread on the Jepsen report, not from a vendor primary page | Recorded as ⚠ vendor-stated-in-a-comment | September 2026 |

### 22.3 Rejected

| Claim | Rejected because | Citation | Date |
| --- | --- | --- | --- |
| "`R + W > N` gives strongly consistent (linearizable) reads" as a general statement | It gives a **regular** register for one key under the ABD conditions; ordering requires a consensus, a lease, or a read that writes ✅. The 28%-and-71%-of-acknowledged-writes-lost measurements are the empirical form | Lamport 1986 ✅; ABD 1995 ✅; Jepsen 2013 ✅ | September 2026 |
| "`Strong`/`ConsistentRead`/`majority` means the read is a consistent snapshot of the *database*" | Each vendor scopes its own strongest setting: one item, one logical partition, one document, one key ✅ (§13.3) | Vendor pages ✅ | September 2026 |
| "`readPreference` is a consistency setting" | The vendor states the opposite: "Read preference does not affect the visibility of data" ✅ | MongoDB ✅ | September 2026 |
| "`writeConcern: majority` means the write cannot be lost" | True only when journaling defaults are on; otherwise majority writes "could possibly roll back" ✅ | MongoDB ✅ | September 2026 |
| "Consistency level `all` prevents consistency anomalies" | Aerospike's own team: it "does not prevent consistency anomalies—**only reduce their frequency**" ✅ | Jepsen 2018 ✅ | September 2026 |
| "A passing Jepsen report means the system is safe" | "tests are nondeterministic, and we **cannot prove correctness, only find errors**" ✅ | Jepsen ✅ | September 2026 |
| "Any bounded-staleness level bounds the age of the data" | The bound is on replication lag between regions and is checked "only across regions and not within a region" ✅; and one product's follower read is "**always stale**", leader or not ✅ | Cosmos DB ✅; YugabyteDB ✅ | September 2026 |
| "A consistency knob can fix a two-record invariant" | HAT systems "cannot prevent concurrent updates" and "cannot provide recency guarantees for reads" in the general case ✅ | Bailis et al., VLDB 2014 ✅ (derivation owned by the theory guide) | September 2026 |

### 22.4 The risk statement

**The quorum result is the highest-risk claim in this guide** because it is the one most often repeated from a summary, and because the summary's version is false while the paper's version is true. **Vendor definitions are the second** because thirteen of the guide's conclusions are a *scope sentence* on a vendor page, and a page that re-words that sentence — or a portal that stops serving it, as three did in §23 — removes the evidence without changing the behaviour. **Cost figures are the third** because they are vendors' own numbers in all cases except the Cosmos write-latency formula, they are denominated in prices that change, and only two of them (the 2× read multipliers) are stated as ratios rather than absolutes. Nothing in this guide's cost material is a measurement by this author; §23 lists the four figures that are vendor-marketed rather than independently observed.
## 23. What Could Not Be Verified

Recorded honestly, because a guide whose gaps are invisible is a guide whose reader cannot calibrate it. Two conventions apply throughout: an **unresolved page is recorded as a retrieval failure, never as evidence that a claim is false**, and **an empty search result set is recorded as a TOOL LIMITATION, not as an absence of the thing searched for.**

### 23.1 Pages that did not resolve

**Carried forward from this file's existing metadata paragraph** (three seed pages, all from the original pass, all recorded ✅ as *fetched, no substantive content*): the Riak KV read-quorum page (`docs.riak.com/riak/kv/latest/developing/usage/read/index.html` → product chooser); the Aerospike consistency page (`aerospike.com/docs/server/develop/consistency` → documentation portal); and the Couchbase N1QL consistency page (`docs.couchbase.com/server/current/n1ql/n1ql-language-reference/consistency.html` → documentation portal). The `.md` variants of the Couchbase N1QL and storage-mode pages returned scraper errors.

**New this pass** — all attempted for the sections added above, all failed, none of them producing a claim:

| Attempted source | Why it was attempted | Outcome |
| --- | --- | --- |
| Attiya, Bar-Noy & Dolev, *Sharing memory robustly in message-passing systems*, full text — five candidate URLs tried, including `cs.utexas.edu/~lorenzo/corsi/cs380d/papers/ABD.pdf` (404 ✅), `cs.cornell.edu/courses/cs7412/2011sp/papers/ABD.pdf` (404 ✅), `groups.csail.mit.edu/tds/papers/Lynch/jacm95.pdf` (404 ✅), the Yale *pinewiki* page (404 ✅), and a Ben-Gurion University mirror (404 ✅) | Verify the paper's protocol shape and fault model | **Not retrieved.** The bibliographic record and the **published abstract** were retrieved instead via OpenAlex (`api.openalex.org`, DOI `10.1145/200836.200869`, JACM 42(1):124–142, 1995) ✅, and the protocol-shape sentence in §16.1 is flagged ⚠ as characterised from the standard literature rather than from the paper's text |
| `dl.acm.org/doi/10.1145/200836.200853` | I had this DOI recorded as the ABD paper's | Both wrong and unreachable. The lookup returned a **different** paper (Callahan & Kosaraju, k-nearest-neighbours, JACM 1995) ✅ — a reminder that a DOI is a claim and needs checking — and the ACM landing pages could not be scraped ✅. The correct DOI is `10.1145/200836.200869`, confirmed via OpenAlex ✅ |
| `api.semanticscholar.org/graph/v1/paper/search?query=…` | Machine-readable abstract retrieval for the paper above | Scraper failure ✅. (The DOI-keyed endpoint of the same API did respond, but echoed the wrong record — see above.) The OpenAlex API answered ✅ |
| The directory listing of a public mirror hosting the session-guarantee and ABD papers | Locate a resolvable ABD copy | Scraper failure ✅ |
| `decentralizedthoughts.github.io` ABD explainer | A candidate secondary explanation of the protocol | 404 ✅ — so §16.1's ⚠ on protocol shape rests on the literature generally, not on that page |

### 23.2 Definitions asserted from something other than a primary vendor page

| Definition | Status | Where it is recorded |
| --- | --- | --- |
| **ScyllaDB per-level consistency semantics** | Not asserted. The section is structural, flagged ENGINEERING CONVENTION, and directs the reader to the vendor's current page | §4.1–4.3, §22.2 |
| **Aerospike `SC`/`AP` read-mode and commit-level enumerations** | Not asserted. Only the two-axis *shape* is described, plus the acceptance-ceiling evidence from the two independent analyses ✅ and the client-retry behaviour ✅ (which comes from the independent report, not a vendor page) | §10.1–10.3, §17 row **d** |
| **Couchbase `scan_consistency` level names and guarantees** | Not asserted — the commonly cited values are deliberately not enumerated. Only the *structural* reason for a separate query axis is stated | §9.2–9.3 |
| **Couchbase index storage modes** | Not asserted | §9.3 |
| **Cassandra's periodic-commitlog-fsync default** | Carried at ⚠ as an audit item rather than as a quoted fact | §17 row **f**, §22.2 |
| **ABD's protocol mechanics** | ⚠ — abstract quoted ✅, mechanics characterised from the literature | §16.1 |
| **Terry et al.'s page numbers and exact venue string** | Not verified this pass: the PDF was retrieved ✅ and its author list and content confirmed ✅, but the standing citation "PDIS 1994" is carried from the existing file rather than re-derived | §16.3, §18.3, §25 |
| **YugabyteDB's leader-lease sentence** | Partially retrieved (the extracted page truncated it); quoted only in part | §11.3, §16.4 |
| **Any claim that a named product's *current* release behaves as the version-dated measurement shows** | Explicitly not made. Every measurement in §16.5 and §22.1 is attached to its version and date | §16.5, §22.1 |

### 23.3 Costs that are vendor-marketed rather than independently observed

These four are the guide's cost material, and in every case the number's *origin* is the vendor's own page or the vendor's own recommendation — not a benchmark by a third party, and not a measurement by this author.

| Figure | Origin | Status |
| --- | --- | --- |
| DynamoDB: strongly consistent reads cost **2×** eventually consistent reads | AWS's own pricing documentation ✅ | VENDOR MARKETING-ADJACENT (vendor's published price structure; a *ratio* rather than an observation) |
| Cosmos DB: strong-consistency write latency = **2×RTT between the two farthest regions + 10 ms at p99** | Microsoft's own consistency documentation ✅ | VENDOR DOCUMENTATION of a latency promise — the most precise public statement in the guide, and still a vendor's number |
| Cosmos DB: read throughput for strong/bounded-staleness is **half** that of other levels, for the same RUs | Microsoft ✅ | Same class |
| Riak: strong-mode performance hit of "**varying proportions**"; recommendation of `n_val >= 5` | Riak's own pages ✅ | VENDOR DOCUMENTATION that declines to quantify (the vendor's *refusal* to give a number is itself the honest data point) |
| *Everything attributed to Cymbal Bank* | Invented for shape | **ILLUSTRATIVE AND FICTIONAL.** Not a benchmark of anything |

### 23.4 Empty search result sets, recorded as tool limitations

`web_search` on this host returned **empty result sets** for: the phrase-query used to locate the Attiya/Bar-Noy/Dolev full text; and (in the earlier pass recorded in this file's metadata paragraph) a set of topics in this area generally. Per this repository's convention, an empty search is recorded as a **TOOL LIMITATION and not as a negative fact**: it does not mean the document, the analysis or the product behaviour does not exist. Where the guide needed a source and the search was empty, it either used `web_extract` against a primary URL directly (which is how the Lamport PDF, the Dynamo paper, the Vogels article, the session-guarantee paper, the two Aerospike analyses, the Riak and Cassandra analyses, the MongoDB and Scylla analyses, the Jepsen index and the consistency reference were all obtained ✅) or it said so in a ⚠ flag rather than inferring.

One further limitation worth stating plainly, because it bounds everything above: **this guide's verification is documentation-and-paper verification, not system verification.** Nothing here was measured on a running cluster. That is why §15 exists as a section rather than as a concluding flourish, and why §22's "verified" column means "verified against the cited source", not "verified against a deployment".
## 24. Glossary

Terms are defined here as **this guide uses them**, which for the consistency models means the theory guide's [`distributed_systems_engineering_guide.md`](distributed_systems_engineering_guide.md) §5 vocabulary rather than a vendor's. Where a definition is a vendor's own, the vendor is named.

| Term | Definition as used here |
| --- | --- |
| **Anti-entropy** | The background process by which replicas reconcile divergent state (Dynamo's and Cassandra's repair mechanisms). It is where the "eventual" in eventual consistency actually lives ✅ (§3.3, §8.1) |
| **Atomic register** | Lamport's strongest register class: "reads and writes behave as if they occur in some definite order" ✅ — i.e. a total order consistent with real time. The rung of the ladder that `R + W > N` alone does not reach (§16.1) |
| **Bounded staleness** | A read setting that guarantees data no more than a stated age/version distance behind, rather than recency. Enforced in one product by throttling writes ✅, expressed in another as a historical timestamp clause whose forms are a *lower bound on data age* ✅ (§6.3, §11.1) |
| **Causal consistency** | A per-session guarantee that operations related by a read-write dependency are seen in that order by the reader that has the dependency. In MongoDB it is delivered by a specific **pair** of concerns, not by the word "session" ✅ (§7.4) |
| **Consensus (Paxos / Raft)** | A protocol that produces agreement on a single ordered value among a quorum. It is the only mechanism in this guide that orders *concurrent writes*, and therefore the only thing that upgrades a quorum read from a regular to an atomic register (§16.3 condition 5) |
| **Claim classes** | The guide's tagging convention: **PROVEN RESULT** (a theorem or peer-reviewed measurement), **ENGINEERING CONVENTION** (a widely-used implementation choice that is not a theorem), **VENDOR MARKETING** (a vendor's own framing). Plus the integrity tags: **✅** verified this pass against the cited primary source; **⚠** flagged (ambiguous, hedged, or version-specific source); **❌** rejected |
| **Consistent prefix** | Cosmos DB's level guaranteeing that updates made as a transaction batch are never seen out of order, with no guarantee for independently written items ✅ (§6.1) |
| **ConsistentRead** | DynamoDB's per-request boolean. Per item, single-region, unavailable on GSIs and streams ✅ (§5.1) |
| **CRDT** | A data type whose merge function is associative, commutative and idempotent, making order-free convergence possible. Theoretically the right shape for a leaderless store; not a consistency level (§8.3, §13.3) |
| **Discovered vs visible write** | The distinction at the heart of §16: a quorum read *discovers* a write by intersecting the write set, and makes it *visible* only if the coordinator compares and repairs |
| **Durability level / write concern / commit level** | The **durability axis** — whether a write survives a node crash — as distinct from the replica-consistency axis. Called "consistency" by almost every user and by none of the vendors' own definitions ✅ (§1.2, §7.3, §9.1) |
| **Dynamic quorum** | Cosmos DB's ability to shrink the region set required for a `Strong` write when regions are slow, at the cost that removed regions "are no longer able to serve reads" ✅ (§6.2) |
| **Follower read** | A read served by a non-leader replica, strong only because the replica can *prove* it is safe to serve (a lease, a closed timestamp), or stale by a declared interval ✅ (§11.2, §11.3) |
| **GSI / LSI** | DynamoDB global and local secondary indexes. Both are on the derived path, and `ConsistentRead` does not reach the GSI ✅ (§5.1, §17 row a) |
| **HAT (highly available transactions)** | The class of isolation levels achievable with high availability; "serializable transactions … are **not achievable** with high availability" in the presence of partitions ✅ (Bailis et al., VLDB 2014) |
| **Hinted handoff** | Dynamo/Cassandra's mechanism for accepting a write on a node that is not a replica of the key, with a hint to deliver it later. It is what makes a quorum "sloppy" ✅ (§16.2 reason 3) |
| **Linearizability** | The strongest single-object model: every operation appears to take place atomically "in some order, consistent with the real-time ordering of those operations" ✅ (`SingleOrder` + `RealTime` + `RVal`). Single-object, and not available under partition. In this guide the word is used only with its scope attached |
| **LWT (lightweight transaction)** | Cassandra/Scylla's compare-and-set, implemented as Paxos over a quorum, single-partition only, with `SERIAL`/`LOCAL_SERIAL` on the read side ✅ (§3.4, §16.4) |
| **Last-write-wins (LWW)** | Conflict resolution by timestamp. It **discards** a concurrent write rather than ordering it, which is the mechanism behind the measured 28% and 71% loss figures ✅ (§16.2 reason 1) |
| **MREC / MRSC** | DynamoDB global tables: multi-region eventual consistency (default) and multi-region strong consistency, the latter writing synchronously to "another" region before returning ✅ (§5.2) |
| **Quorum** | The number of replicas that must answer for an operation to succeed. In Cassandra, `QUORUM` is a majority of **all** replicas cluster-wide ✅ |
| **Read concern / read preference / write concern** | MongoDB's three axes: isolation+recency of the data read; *which member answers* (explicitly "does not affect the visibility of data" ✅); and replication+journal acknowledgement. Orthogonal, independently settable ✅ (§7.1) |
| **Read repair** | The read-path reconciliation that repairs replicas after an inconsistency is observed. Its *discipline* (bounded, synchronous, comparison-enforcing) is what turns quorum intersection into visibility (§16.2 reason 5) |
| **Read-your-writes** | A session guarantee: "read operations reflect previous writes" ✅ (Terry et al., PDIS 1994). The cheapest correct answer to the most common user-facing complaint (§18.3) |
| **Regular register** | Lamport's middle class: a read overlapping a write returns "either the old or new value", but old-then-new and new-then-old are both permitted ✅. **This is what intersecting quorums buy** (§16.1) |
| **Safe register** | Lamport's weakest class: correct only for reads that do not overlap a write; an overlapping read may return any value the register can hold ✅ |
| **Scan consistency** | Couchbase's query-side consistency control, separate from KV durability. The level names are deliberately not asserted in this guide ⚠ (§9.2) |
| **Session token / session guarantee** | The client-held state that makes a database honour read-your-writes for one actor. It does not survive a process restart or a re-created client unless the application preserves it ✅ (§6.4, §18.3) |
| **Sloppy quorum** | A quorum counted over "the first N healthy nodes from the preference list" rather than the key's replicas ✅ — which removes the intersection invariant (§16.2 reason 3) |
| **Sibling** | Riak's representation of a conflict: several values for one key, returned to the application for merging when `allow_mult` is true ✅ (§8.3) |
| **Strict serializability** | Linearizability across multiple objects: the model to reach for when the invariant spans records. Not obtainable from a consistency knob ✅ |
| **Weakest-level-in-path** | The guide's operational rule: the last write and the first read on a data path determine the effective guarantee, and both must be inventoried from code, not documentation ✅ (§14.3) |
## 25. Cross-references and further reading

### 25.1 Within this repository

**The theory layer** — owned elsewhere, assumed here:

- [`distributed_systems_engineering_guide.md`](distributed_systems_engineering_guide.md) — §4.2 CAP and its misreading, §4.4 PACELC, §4.5 the other impossibilities (including the HAT result), §5 the consistency spectrum and the session guarantees, §5.3 the isolation-anomaly catalogue, **§5.4 the five-step procedure** applied at business-process level (§14.1 and §20 use its operation-level form), §6.6 quorum arithmetic, §7 replication and partitioning.
- [`nosql_data_modelling_guide.md`](nosql_data_modelling_guide.md) — the modelling decisions that make a consistency setting either necessary or irrelevant (the §20 phase-one re-modelling is a modelling change, not a consistency change).
- [`late_arriving_data_guide.md`](late_arriving_data_guide.md) — time and ordering on the analytics path, where bounded staleness is an arrival question rather than a replica-read question.

**The product companions** — the neighbouring guides this one sits between:

- [`cockroachdb_guide.md`](cockroachdb_guide.md) — serializability, parallel commits and commit-wait; the knob-refuser of §11 as a system, rather than as a contrast case.
- [`polardb_vs_oceanbase_guide.md`](polardb_vs_oceanbase_guide.md), [`oracle_sharding_guide.md`](oracle_sharding_guide.md) — the sharded-relational neighbours with their own, narrower, consistency surfaces.

**The testing and evidence layer** — referenced by §15 and §21 rather than re-derived:

- [`chaos_engineering_guide.md`](chaos_engineering_guide.md) — fault injection, nemeses, and the operational shape of a partition test.
- [`deterministic_engineering_guide.md`](deterministic_engineering_guide.md) — reproducible harnesses, so that §15.1's five assertions fail the same way twice.
- [`audit_as_code_guide.md`](audit_as_code_guide.md) — controls expressed as executable evidence, the enforcement mechanism for §18.4's provenance requirement.
- [`capacity_sizing_guide.md`](capacity_sizing_guide.md) — the capacity review that §12.3 says a consistency change belongs inside.
- [`event_stream_processing_guide.md`](event_stream_processing_guide.md) — the derived path of §17 row **g** as a discipline of its own.

**The banking and payments layer** — §18's application context, with the mechanics owned there:

- [`../banking/payment_rails_guide.md`](../banking/payment_rails_guide.md) and [`../banking/payments_hub_guide.md`](../banking/payments_hub_guide.md) — rails and hub architecture; the authorisation-versus-display distinction of §18.1 in process terms.
- [`../banking/posting_engine_core_banking_guide.md`](../banking/posting_engine_core_banking_guide.md) and [`../banking/end_to_end_banking_processes.md`](../banking/end_to_end_banking_processes.md) — the ledger and the statement, and why §18.1 insists the statement is derived from the ledger rather than from a read path.
- [`../banking/banking_limits_domain_guide.md`](../banking/banking_limits_domain_guide.md) — limits and available balance, the domain behind the operation that §20 refuses to solve with a consistency level.
- [`../banking/financial_fraud_detection_at_scale_guide.md`](../banking/financial_fraud_detection_at_scale_guide.md) — pre-authorisation scoring versus post-hoc investigation, §18.2's two horizons.
- [`../banking/iso_20022_core_processes_guide.md`](../banking/iso_20022_core_processes_guide.md), [`../banking/nets_singapore_guide.md`](../banking/nets_singapore_guide.md), [`../banking/singapore_fintech_payments_guide.md`](../banking/singapore_fintech_payments_guide.md) — message flows, clearing and the Singapore context.
- [`../banking/kafka_guide.md`](../banking/kafka_guide.md) — log semantics and delivery guarantees: a different axis from read/write consistency, and the layer where "the stream is one message behind" becomes an explicit ordering question.
- [`../management/vendor_management_guide.md`](../management/vendor_management_guide.md) — the procurement and assurance side of §19's vendor-claim audit.

### 25.2 Primary sources, with dates consulted

**Papers and results** (all consulted September 2026):

- Attiya, H., Bar-Noy, A. & Dolev, D. *Sharing memory robustly in message-passing systems*. JACM 42(1):124–142, January 1995. DOI `10.1145/200836.200869`. **Abstract ✅; full text not retrieved ⚠** (§23.1). The quorum-intersection result, the majority-of-processors condition, and the single-writer multi-reader scope (§16.1).
- Lamport, L. *On Interprocess Communication* (Parts I and II), published as two articles in *Distributed Computing*, 1986. PDF: `lamport.azurewebsites.net/pubs/interprocess.pdf` (report dated 25 December 1985; the report states the publication history). The safe/regular/atomic register hierarchy, Proposition 5, and the author's own note that multi-writer registers are not addressed (§16.1).
- DeCandia, G., Hastorun, D., Jampani, M., Kakulapati, G., Lakshman, A., Pilchin, A., Sivasubramanian, S., Vosshall, P. & Vogels, W. *Dynamo: Amazon's Highly Available Key-value Store*. SOSP 2007. "Quorum-like" framing, sloppy quorum and hinted handoff, and the read path's write-back reconciliation (§16.1–16.2).
- Terry, D. B., Demers, A. J., Petersen, K., Spreitzer, M. J., Theimer, M. M. & Welch, B. B. *Session guarantees for weakly consistent replicated data*. PDIS 1994 (venue and page range carried from the guide's existing citation ⚠). The four session guarantees (§16.3, §18.3).
- Vogels, W. *Eventually Consistent* (revised edition), ACM Queue / All Things Distributed, 23 December 2008. Session consistency's scope, the inconsistency window, user-perceived consistency (§18.3).
- Bailis, P. et al. *Highly Available Transactions: Virtues and Limitations*, VLDB 2014. Serializability is not HAT-compliant; HAT systems "cannot provide recency guarantees for reads" (§13.2, §22.3).
- Abadi, D. *Consistency Tradeoffs in Modern Distributed Database System Design* (PACELC), IEEE Computer 45(2), 2012. The latency-versus-consistency trade independent of partitions (§12.1–12.2).
- Herlihy, M. & Wing, J. *Linearizability: A Correctness Condition for Concurrent Objects*, TOPLAS 12(3), 1990; and Viotti, P. & Vukolić, M. *Consistency in Non-Transactional Distributed Storage Systems*, ACM Computing Surveys 49(1), 2016 — the definitional chain behind §16's use of the word, via Kingsbury's reference below.

**Independent measurements** (Jepsen; dates as published, all read from the index and the reports this pass):

- Kingsbury, K. *Jepsen: Riak* (Riak 1.2.1), 19 May 2013 — `aphyr.com/posts/285-call-me-maybe-riak`.
- Kingsbury, K. *Jepsen: Cassandra* (Cassandra 2.0.0), 24 September 2013 — `aphyr.com/posts/294-call-me-maybe-cassandra`.
- Kingsbury, K. *Jepsen: Aerospike* (3.5.4), 4 May 2015 — `aphyr.com/posts/324-call-me-maybe-aerospike`.
- Kingsbury, K. *Jepsen: Aerospike 3.99.0.3*, 7 March 2018 — `jepsen.io/analyses/aerospike-3-99-0-3`.
- Kingsbury, K. *Jepsen: MongoDB 4.2.6*, 15 May 2020 — `jepsen.io/analyses/mongodb-4.2.6`.
- Kingsbury, K. *Jepsen: Scylla 4.2-rc3*, 23 December 2020 — `jepsen.io/analyses/scylla-4.2-rc3`.
- *Analyses* (the published index, which is also the source for the coverage statement about DynamoDB, Cosmos DB and Couchbase) — `jepsen.io/analyses`.
- *Consistency* reference and *Linearizability* model entry — `jepsen.io/consistency`, `jepsen.io/consistency/models/linearizable`.
- Bibliographic verification for the 1995 JACM record — `api.openalex.org` (OpenAlex, work W2065180040).
## 26. Closing summary

Part I (sections 1–15) answered four questions per product and generalised them into a cost model, a misleading-names catalogue, a per-operation procedure, and a test suite. Part II (sections 16–26) closes the loop by doing four things Part I could not:

1. **It states the central result precisely rather than approximately** (§16). `R + W > N` buys quorum intersection, and quorum intersection buys a **regular** register for one key under a named set of conditions — not linearizability. Linearizability costs a consensus round, a leader lease, or a read that writes. Everything the field says loosely about "strong quorum reads" is either this theorem or a misreading of it, and the misreading has a measured cost: 28% of acknowledged writes in one 2013 test, 71% in another.
2. **It converts the naming gaps into configuration traps with a checklist** (§17), because a gap becomes an incident only through a specific setting on a specific path — the GSI that cannot be strengthened, the secondary read under a strong preference, the local quorum believed global, the session token that dies with the process, the bound that is per region pair, the write acknowledged before it is durable, the derived path, and the knob that governs nothing the team actually queries.
3. **It grounds the whole thing in the domain where being wrong is expensive** (§18–§20): display versus authorisation, statement versus ledger, pre-authorisation scoring versus post-hoc reconciliation, read-your-writes as a session problem rather than a quorum problem, and the read provenance an auditor will ask for. The Cymbal Bank walk-through ends with a design team choosing its settings per operation, deliberately keeping three of them weak, and writing down why — which is the only form in which a consistency decision is reviewable.
4. **It audits itself** (§19, §22, §23): which sentences are vendor marketing, which are vendor documentation, which are independently measured; which claims were verified, flagged or rejected, with source quality and date; and which pages did not resolve and which definitions rest on something other than a primary source.

Three findings are worth carrying out of the guide as a whole. **The first is that the knob is scoped, always** — one key, one item, one partition, one session, one bucket — and the scope, not the name, is the purchase (§13.3, §16.6). **The second is that the strongest setting in most of these systems is a setting that routes the operation somewhere else** — through Paxos, through a lease, or through a primary — rather than a setting that raises a count (§16.3). **The third is that the failure mode of the weak end is silent and the failure mode of the strong end is loud**, so the cheapest reliable posture is to be explicit per operation, to quote the vendor's own scope sentence in the assertion, and to test under a fault (§15, §21).

The reading order, if this is being used as a reference rather than read in sequence: §2.2 for the decode table, §16 for the headline finding, §17 for the trap checklist before any design review, §14 for the per-operation procedure, and §23 before quoting any number in this guide to somebody else.

*One sentence holds all of it, and it is the same sentence in every section, every table and every test: the knob is a promise about behaviour under failure, and the honest question is* what the knob actually buys.
