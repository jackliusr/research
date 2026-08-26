# Apache Kafka: The Distributed Event-Streaming Platform — A Comprehensive Guide

*A companion deep-dive to [Kafka Alternatives](../technology/kafka_alternatives_guide.md) (the head-to-head comparison — cross-referenced heavily in §8) and [Event Stream Processing](../technology/event_stream_processing_guide.md) (the discipline). This guide is the dedicated Kafka reference in the research repo: the definition and the 2011 LinkedIn origin (§1), the architecture — brokers, topics, partitions, offsets, consumer groups, the log-segment storage (§2), the replication — ISR, leader election, KRaft, the ZooKeeper removal (§3), the messaging semantics — at-least-once, at-most-once, exactly-once via idempotent producers and transactions (§4), the streaming — Kafka Streams, ksqlDB (§5), Connect and the Schema Registry (§6), the ecosystem — client libraries, managed offerings, tiered storage (§7), the comparison vs the alternatives (§8), the banking context — why a Cymbal Bank runs Kafka as the event backbone (§9), a Cymbal Bank event-platform worked example (event taxonomy + topic design — §10), a one-page summary (§11), a glossary, and the verification/claims ledger.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** the event-backbone guide of the banking/technology series — the platform reference behind the event-driven core-banking, payments, markets and regulatory use cases covered across the banking/ siblings
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** kafka.apache.org (the official documentation — architecture, KRaft, the 4.0.0 release announcement of 18 March 2025), the Kreps/Narkhede/Rao NetDB'11 paper ("Kafka: a Distributed Messaging System for Log Processing"), the Apache Kafka wiki (papers and presentations), the 0.10.0.0 release notes, Confluent docs (Kafka Streams, Schema Registry). NOTE: this pass had **live web access** (web_search + web_extract): the Kafka 4.0 date and the KRaft-only statement, KIP-848/KIP-890/KIP-966, and the NetDB'11 paper were verified at primary sources. The search backend degraded mid-pass (several queries returned empty), so a few secondary items (ksqlDB naming history, Schema Registry compatibility levels, KIP-354) are flagged ⚠ honestly in the ledger rather than asserted.
> **Last Updated:** August 2026
> **Companion guides (banking/ siblings — plain filenames):** [Crédit Agricole Software Systems](credit_agricole_software_systems_guide.md), [NETS Software Systems](nets_software_systems_guide.md), [Treasury & ALM](treasury_alm_guide.md), [Payments Hub](payments_hub_guide.md), [ISO 20022 Core Processes](iso_20022_core_processes_guide.md), [Regtech](regtech_guide.md), [Enterprise Risk Management](enterprise_risk_management_guide.md), [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md), and the bank series ([DBS Bank](dbs_bank_guide.md), [UOB Software Systems](uob_software_systems_guide.md), [OCBC Software Systems](ocbc_software_systems_guide.md), [HSBC Software Systems](hsbc_software_systems_guide.md), [Bank of America Software Systems](bank_of_america_software_systems_guide.md))
> **Companion guides (technology/ — prefix ../technology/):** [Kafka Alternatives](../technology/kafka_alternatives_guide.md) (the head-to-head — cross-ref §8), [Event Stream Processing](../technology/event_stream_processing_guide.md) (the discipline — cross-ref §4 and §5), [Complex Event Processing](../technology/complex_event_processing_guide.md), [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md), [Low Latency C++ Development](../technology/low_latency_cpp_development_guide.md) (the trading hot-path contrast — cross-ref §8); (management/ — prefix ../management/): [Business Case Development](../management/business_case_development_guide.md)

**Verification convention used throughout: ✅ = verified this pass against a primary source (kafka.apache.org, the Apache wiki, the Apache release notes) or already verified in a cross-referenced sibling guide's ledger; ⚠ = flagged (not re-verified this pass, approximate, single-source, or held from the author's knowledge base); unmarked = structural/architectural knowledge presented as such. The consolidated list is the [Verification Ledger](#verification-ledger) at the end.**

**Research-method note — read before trusting any ✅:** this pass had **live web access** for the critical facts: the Apache Kafka 4.0.0 release announcement (kafka.apache.org, published 18 March 2025), the Apache Kafka wiki's papers-and-presentations page (NetDB'11), and the Kafka 0.10.0.0 release notes were all retrieved and are cited inline. The search backend degraded partway through the pass (three consecutive queries returned empty results), so anything that could not be re-checked after that point is marked ⚠ — nothing in this guide is fabricated, and where the brief demanded verification and the tools went down, this guide says so plainly.

**How to use this guide:** read §1–§3 for the platform itself (what Kafka is, how it stores and replicates data — the foundation every later section assumes), §4–§5 for the semantics that decide correctness (delivery guarantees, exactly-once, stream processing), §6–§7 for the surrounding platform (Connect, Schema Registry, managed offerings), and §8–§10 for the decision and design layers (alternatives, the banking context, the worked Cymbal Bank event-platform design). §11 is the one-page summary. The [Glossary](#glossary) covers every bolded term; the [Verification Ledger](#verification-ledger) lists exactly what was verified this pass (✅) versus flagged (⚠); the [Cross-References Recap](#cross-references-recap) lists every sibling link used. The ✅/⚠ integrity convention is the repo's: a ✅ means "checked at a primary source this pass or in a sibling ledger", never "assumed".

### Series Context: Where This Guide Sits

This is the **dedicated Kafka deep-dive** of the research repo — the platform guide behind the event-driven use cases scattered across the banking/ and technology/ siblings. How it relates to the neighbours:

- **The head-to-head sibling** — [Kafka Alternatives](../technology/kafka_alternatives_guide.md) is the *comparison* guide (Kafka vs Redpanda vs Pulsar vs WarpStream vs NATS and the rest); this guide is the *platform* guide. The division of labour: **this guide = how Kafka works and how to design with it; the alternatives guide = when to choose it or something else**. §8 here summarizes the axes; the sibling derives the full matrix.
- **The discipline siblings** — [Event Stream Processing](../technology/event_stream_processing_guide.md) covers the general discipline (delivery guarantees, windowing, dedupe patterns) that §4–§5 apply to Kafka specifically; [Complex Event Processing](../technology/complex_event_processing_guide.md) is the adjacent pattern (pattern detection over streams — the fraud/AML angle of §9.2).
- **The banking siblings** — [Payments Hub](payments_hub_guide.md), [ISO 20022 Core Processes](iso_20022_core_processes_guide.md), [Treasury & ALM](treasury_alm_guide.md) and the bank series describe the *systems* whose events flow through the backbone; this guide describes the backbone itself. §9 maps the two sides.
- **The author's angle** — written from the Cymbal Bank architecture desk: the worked example (§10) is deliberately Cymbal Bank-flavored (payments, trade lifecycle, reference data, DR), and the operating model (§9.3) reflects how a global CIB actually runs a platform team. The ✅/⚠ discipline is the series' own: verified facts and honest flags, nothing invented.

## Table of Contents

1. [The Kafka Overview](#1-the-kafka-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The 2011 LinkedIn Origin](#12-the-2011-linkedin-origin)
   - 1.3 ["The Log" and the Log-Centric Architecture](#13-the-log-and-the-log-centric-architecture)
   - 1.4 [The Timeline (2011 → 2025)](#14-the-timeline-2011--2025)
2. [The Architecture](#2-the-architecture)
   - 2.1 [The Brokers](#21-the-brokers)
   - 2.2 [Topics and Partitions](#22-topics-and-partitions)
   - 2.3 [Offsets and the Commit Log](#23-offsets-and-the-commit-log)
   - 2.4 [Consumer Groups and Rebalancing](#24-consumer-groups-and-rebalancing)
   - 2.5 [The Log-Segment Storage: Retention and Compaction](#25-the-log-segment-storage-retention-and-compaction)
   - 2.6 [Producers](#26-producers)
   - 2.7 [Consumers](#27-consumers)
   - 2.8 [The Internal Topics and the Operational Surface](#28-the-internal-topics-and-the-operational-surface)
3. [The Replication](#3-the-replication)
   - 3.1 [The ISR (In-Sync Replicas)](#31-the-isr-in-sync-replicas)
   - 3.2 [Leader Election: Preferred Leaders and the Unclean Trade-Off](#32-leader-election-preferred-leaders-and-the-unclean-trade-off)
   - 3.3 [acks=all + ISR = the Durability Contract](#33-acksall--isr--the-durability-contract)
   - 3.4 [KRaft: The Raft-Based Metadata Quorum](#34-kraft-the-raft-based-metadata-quorum)
   - 3.5 [ZooKeeper Fully Removed (Kafka 4.0, 18 March 2025)](#35-zookeeper-fully-removed-kafka-40-18-march-2025)
   - 3.6 [The ZooKeeper-Era Migration Path](#36-the-zookeeper-era-migration-path)
   - 3.7 [The Replication Configuration Quick Reference](#37-the-replication-configuration-quick-reference)
4. [The Messaging Semantics](#4-the-messaging-semantics)
   - 4.1 [At-Least-Once (the Default)](#41-at-least-once-the-default)
   - 4.2 [At-Most-Once](#42-at-most-once)
   - 4.3 [Exactly-Once: Idempotent Producers + Transactions](#43-exactly-once-idempotent-producers--transactions)
   - 4.4 [read_committed, transactional.id and Zombie Fencing](#44-read_committed-transactionalid-and-zombie-fencing)
   - 4.5 [The Honest Caveat: Cross-System Exactly-Once](#45-the-honest-caveat-cross-system-exactly-once)
5. [The Streaming](#5-the-streaming)
   - 5.1 [Kafka Streams: A Library, Not a Cluster](#51-kafka-streams-a-library-not-a-cluster)
   - 5.2 [The Stream-Table Duality](#52-the-stream-table-duality)
   - 5.3 [Exactly-Once in Streams](#53-exactly-once-in-streams)
   - 5.4 [ksqlDB: SQL over Streams](#54-ksqldb-sql-over-streams)
6. [Connect and the Schema Registry](#6-connect-and-the-schema-registry)
   - 6.1 [Kafka Connect: The Integration Layer](#61-kafka-connect-the-integration-layer)
   - 6.2 [The Connector Ecosystem (Debezium CDC, S3, JDBC, Elasticsearch)](#62-the-connector-ecosystem-debezium-cdc-s3-jdbc-elasticsearch)
   - 6.3 [Schema Registry: Avro, JSON, Protobuf and Compatibility](#63-schema-registry-avro-json-protobuf-and-compatibility)
   - 6.4 [MirrorMaker 2: Cross-Cluster Replication and DR](#64-mirrormaker-2-cross-cluster-replication-and-dr)
7. [The Ecosystem](#7-the-ecosystem)
   - 7.1 [Client Libraries](#71-client-libraries)
   - 7.2 [Managed Offerings](#72-managed-offerings)
   - 7.3 [Tiered Storage](#73-tiered-storage)
   - 7.4 [The Commercial Layer](#74-the-commercial-layer)
8. [The Comparison vs the Alternatives](#8-the-comparison-vs-the-alternatives)
   - 8.1 [The Pain Points](#81-the-pain-points)
   - 8.2 [The Strengths](#82-the-strengths)
   - 8.3 [When NOT to Pick Kafka](#83-when-not-to-pick-kafka)
9. [The Banking Context](#9-the-banking-context)
   - 9.1 [Why a Cymbal Bank Runs an Event Backbone](#91-why-a-cymbal-bank-runs-an-event-backbone)
   - 9.2 [The Banking Use Cases](#92-the-banking-use-cases)
   - 9.3 [The Operating Model](#93-the-operating-model)
   - 9.4 [The Regulatory and Audit Angle](#94-the-regulatory-and-audit-angle)
10. [Worked Example: A Cymbal Bank Event-Platform Design (Event Taxonomy + Topic Design)](#10-worked-example-a-cymbal-bank-event-platform-design-event-taxonomy--topic-design)
    - 10.1 [The Event Taxonomy](#101-the-event-taxonomy)
    - 10.2 [The Topic Design](#102-the-topic-design)
    - 10.3 [The Platform Layout](#103-the-platform-layout)
    - 10.4 [The Concrete Banking-Flavored Topics](#104-the-concrete-banking-flavored-topics)
    - 10.5 [The Design Checklist](#105-the-design-checklist)
11. [The One-Page Summary](#11-the-one-page-summary)
- [Glossary](#glossary)
- [Verification Ledger](#verification-ledger)
- [Cross-References Recap](#cross-references-recap)
- [References and Further Reading](#references-and-further-reading)

---

## 1. The Kafka Overview

### 1.1 The Short Answer

**Apache Kafka is a distributed event-streaming platform** — at its core, a **distributed, partitioned, replicated commit log** ✅. The official framing (kafka.apache.org) is that Kafka is used for high-performance data pipelines, streaming analytics, data integration and mission-critical applications; the architecture, however, is best understood as one idea: an **append-only, replicated log** made into a shared infrastructure component. Everything else in this guide — topics and partitions, offsets, consumer groups, ISR replication, KRaft, exactly-once, Kafka Streams — is a consequence of that single design bet.

The four capabilities an architect should hold from the start:

| Capability | What Kafka provides | Where it is covered |
|---|---|---|
| **Publish/subscribe** | Producers write records to topics; consumers read them; producers and consumers are fully decoupled | §2 |
| **Durable storage** | Records are written to disk, replicated across brokers, retained for a configured window — replayable at any time | §2.5, §3 |
| **Stream processing** | Kafka Streams and ksqlDB read, transform, join and aggregate the log in real time | §5 |
| **Integration** | Kafka Connect moves data in and out of external systems; MirrorMaker 2 replicates clusters | §6 |

The properties that follow from the commit-log design: **ordering per partition** (records in a partition are strictly ordered, and consumers read them in order), **replay** (a consumer can re-read any retained record — the log is the source of truth, not the consumer's state), **retention-based deletion** (records expire by time/size policy, not because they were consumed — the classic queue/loss-on-consume model does not apply), and **horizontal scale** (partitions are the parallelism unit for both storage and consumption). A Kafka topic is therefore not a queue: it is a **durable, replayable, ordered stream** — the distinction matters everywhere in this guide, and most sharply in §4 (semantics) and §8 (alternatives).

### 1.2 The 2011 LinkedIn Origin

Kafka was **created at LinkedIn** by **Jay Kreps, Neha Narkhede and Jun Rao** ✅ (the trio is documented in the original paper and consistently in the repo's sibling ledgers), and it was born from a data-pipeline problem: LinkedIn's activity data (page views, searches, profile updates) flowed through dozens of point-to-point pipelines — bespoke systems feeding analytics, search, recommendations and monitoring — each with its own queue or pub-sub, each re-solving the same problems of buffering, delivery and back-pressure.

The origin story, verified this pass:

- **The paper** ✅ — **"Kafka: a Distributed Messaging System for Log Processing"**, by Jay Kreps, Neha Narkhede, Jun Rao (LinkedIn), presented at the **NetDB workshop 2011** (co-located with SIGMOD'11). Verified this pass against the Apache Kafka wiki's papers-and-presentations page (cwiki.apache.org) and the paper PDF itself. The paper already describes the load that shaped the design: on the order of **20 billion events per day, ~3 terabytes per day** at LinkedIn at the time.
- **The design thesis** ✅ — the paper explicitly argues that a messaging system for log processing should be built as a **distributed commit log**, not as a traditional JMS-style queue: consumers keep a **pull-based offset** into the log instead of the broker pushing and deleting messages. Pull + offset is what gives replay, batching and consumer speed independence — the ideas that later became the "log" philosophy of §1.3.
- **The open-source release** ✅ — Kafka was **open-sourced in 2011** (Apache 2.0 license) and became an **Apache top-level project in 2012** (both repo-verified in the sibling ledgers; the exact months are ⚠ not re-verified this pass).
- **The LinkedIn engineering blog** ✅/⚠ — the 2011 LinkedIn engineering blog post introduced Kafka to a wider audience ("Kafka: a distributed messaging system for log processing", 2011); the post is real and widely cited ✅, but its exact publication date was not re-verified this pass ⚠.
- **Confluent** ✅ — the three creators founded **Confluent in 2014** (repo-verified), the commercial company around Kafka — see §7.4.

The lineage matters for reading the rest of this guide: Kafka was **not designed as a general-purpose message broker**. It was designed as **LinkedIn's central log for activity data**, and its "weird" properties (retention, replay, per-partition ordering, consumer-side offsets) are exactly the properties a log should have. Every strength and every pain point in §8 traces back to this origin.

### 1.3 "The Log" and the Log-Centric Architecture

The intellectual keystone of the Kafka worldview is **Jay Kreps's essay "The Log: What every software engineer should know about real-time data's unifying abstraction"** (2013) ✅ (repo-verified; the essay is one of the most-cited pieces in the streaming canon). The argument, compressed to what an architect needs:

- **The log is the universal abstraction** — databases, replication, consensus protocols (Paxos/Raft), even file systems are all, at bottom, logs: an append-only sequence of records that defines the order of events. "The log" is what makes replication and recovery possible, because a replica is just a log follower.
- **Kafka's bet** — if the log is so fundamental, build it as a **shared, distributed, replicated infrastructure component** and let every system in the company publish to it and consume from it. The log becomes the **central backbone** of the data platform: the single place where "what happened" is recorded, durably and in order.
- **The consequences** — (1) **decoupling**: producers do not know their consumers (a consumer can attach, rewind and re-read without any producer involvement); (2) **state rebuild**: any derived system (a cache, a search index, a read model) can be reconstructed by replaying the log — the theoretical foundation of **event sourcing** and **CQRS** (glossary); (3) **the stream-table duality** (§5.2): a table is a log compacted by key, a log is a table's change history — the insight Kafka Streams is built on.

This is the essay that made "the event backbone" a real architectural pattern rather than a metaphor — the same pattern §9 applies to a Cymbal Bank and §10 designs concretely. The log-centric architecture is also why Kafka is described throughout this guide as "the commit log as infrastructure".

### 1.4 The Timeline (2011 → 2025)

| Year | Event | Status |
|---|---|---|
| 2011 | Kafka built at LinkedIn; the NetDB'11 paper; open-sourced (Apache 2.0) | ✅ |
| 2012 | Apache top-level project | ✅ |
| 2013 | Jay Kreps's "The Log" essay | ✅ |
| 2014 | Confluent founded by Kreps, Narkhede and Rao | ✅ |
| 2016 | **Kafka Streams** shipped (0.10.0) — the stream-processing library | ✅/⚠ (year repo-verified; version corroborated by the 0.10.0.0 release notes this pass — see §5.1) |
| 2017 | **Exactly-once semantics** (Kafka 0.11): idempotent producers + transactions | ✅ (repo-verified; §4.3) |
| 2019 | **KIP-429** incremental cooperative rebalancing (Kafka 2.4) | ✅ (repo-verified; §2.4) |
| 2022 | **KRaft** early access (Kafka 3.3, October) | ✅ (repo-verified; §3.4) |
| 2023 | **Tiered storage** GA (Kafka 3.6) | ✅ (repo-verified; §7.3) |
| 2025 | **Kafka 4.0, released 18 March 2025**: ZooKeeper fully removed, KRaft-only, KIP-848 consumer-rebalance protocol GA, Queues early access | ✅ (verified this pass at kafka.apache.org — §3.5) |

---

## 2. The Architecture

### 2.1 The Brokers

A Kafka cluster is a set of **brokers** — JVM services, each a **stateful node** that stores partitions on **local disk** and serves produce/consume requests for the partitions it leads ✅. The mental model an architect needs:

- **Stateful by design** — unlike stateless application tiers, a broker holds real data (log segments) on its local filesystem, and the storage engine deliberately uses the **OS page cache** rather than the JVM heap for the hot path (a design stated in the original paper: the broker does not cache messages in-process; it relies on the page cache and on **zero-copy** sends — `sendfile` — for reads) ✅/⚠ (the page-cache/zero-copy design is in the NetDB'11 paper and standard Kafka lore; not re-verified line-by-line this pass).
- **The partition leader model** — each partition has exactly one **leader** broker and zero or more **follower** brokers; all produce/consume traffic for a partition goes to its leader; followers replicate from the leader (§3.1). A broker is leader for some partitions and follower for others, so load spreads across the cluster.
- **The controller / metadata quorum** — cluster metadata (which brokers are alive, which partitions exist, who leads what) is managed by the **controller** — historically a ZooKeeper-elected broker, since KRaft a dedicated **controller quorum** (§3.4).
- **Bootstrap and discovery** — clients connect to any broker (the *bootstrap.servers* list), which returns the live cluster metadata (the *metadata* response); the client then talks directly to partition leaders.
- **Broker sizing reality** ✅/⚠ — brokers are sized for disk throughput and page-cache efficiency, not CPU; a typical production broker stores terabytes of segments, and the 3× replication factor (glossary) triples the disk cost — one of the pain points quantified in §8.1.

### 2.2 Topics and Partitions

- **Topic** — a named, logical stream of records (e.g. `payments.executed`); the unit of organization and of retention/compaction policy.
- **Partition** — a topic is split into **partitions** (configurable at creation, e.g. 12 or 24 for a busy topic); each partition is an independent, **ordered, immutable sequence of records**. The partition is the **scaling primitive**: it is the unit of parallelism for both storage (spread across brokers) and consumption (assigned to one consumer at a time).
- **Ordering is per-partition** — Kafka guarantees order **within a partition, not across a topic**. Records with the same **key** (e.g. `accountId`, `tradeId`, `paymentId`) are hashed to the same partition, which is how per-entity ordering is achieved — the single most important design rule for banking use cases (§10.2).
- **The partition-count trade-off** — more partitions = more parallelism (higher consumer fan-out, higher throughput) but also more overhead: more open file descriptors, more metadata, more replication traffic, and longer/frequent rebalances (§2.4). Partition count is effectively **immutable-ish** (changing it breaks key-based ordering), so it is a capacity decision made up front, with the sizing rule of thumb "throughput per partition ≈ a few MB/s, plan for the peak, and cap by the consumer parallelism you can actually run" ⚠ (sizing guidance is experience-based, flagged as such).

### 2.3 Offsets and the Commit Log

- **Offset** — a monotonically increasing, immutable **sequence number** identifying each record's position within its partition (record N has offset N, the "next offset" pointer advances as records are appended). Offsets are **never reused** and records are **never modified in place** — the partition log is append-only.
- **The consumer's offset is the consumer's business** — a consumer tracks its own **position** (the next record it will read) and periodically **commits** it (writes it to the internal `__consumer_offsets` topic). This is the pull-model inheritance from the 2011 design (§1.2): the broker does not track what each consumer has read, so a consumer can **rewind and re-read** any retained record — the property that makes replay and state-rebuild possible.
- **The log as source of truth** — because deletion is retention-based (§2.5) and not consumption-based, a record exists until the retention policy removes it, regardless of how many consumers have read it. Kafka is therefore safe for **multiple independent consumers** of the same topic (fan-out) and for **late/offline consumers** (a batch job can read yesterday's records), both of which are routine in banking (regulatory reporting, end-of-day processing, data-mesh consumers — §9).

### 2.4 Consumer Groups and Rebalancing

- **The group** — consumers with the same `group.id` form a **consumer group**; the group coordinates so that **each partition is assigned to exactly one member** at any time (a member may hold many partitions; a partition is never processed by two members concurrently). Groups are how Kafka scales consumption: N consumers in a group split the topic's partitions among themselves.
- **The classic group protocol** — members join via the **group coordinator** broker (JoinGroup/SyncGroup), receive an assignment, and heartbeat to stay in the group. When a member joins/leaves/crashes or partitions change, the group **rebalances**: all members revoke their partitions ("stop-the-world"), the coordinator elects a group leader that computes a new assignment, and members resume. The classic protocol is correct but disruptive: the whole group pauses on every membership change.
- **KIP-429 — incremental cooperative rebalancing (Kafka 2.4, 2019)** ✅ (repo-verified) — members keep the partitions they already own while the new assignment is negotiated, and only **revoke the specific partitions** that must move, in a two-phase (revoke → rejoin) cycle. This removed the global stop-the-world pause for most rebalances and is the standard protocol of the classic era.
- **KIP-848 — the next-generation consumer rebalance protocol** ✅ (verified this pass) — the 4.0.0 release announcement states it directly: Kafka says **"goodbye to stop-the-world rebalances"** with the **general availability of KIP-848**, which "improves the scalability of consumer groups while simplifying consumers" and "decreases rebalance times, thanks to its fully incremental design, which no longer relies on a global synchronization barrier". The new protocol is **enabled by default on the server side**; consumers opt in with `group.protocol=consumer` (the classic protocol remains available as `group.protocol=classic`). The coordinator holds group state server-side and heartbeats are lighter — the operational win for large fleets of consumers (a real Cymbal Bank concern: hundreds of microservices in one group would rebalance constantly under the classic protocol).
- **Assignment strategies** — range, round-robin, sticky, cooperative-sticky; the strategy chooses how partitions are distributed across members (sticky minimizes movement during rebalances).

### 2.5 The Log-Segment Storage: Retention and Compaction

- **Segments** — each partition's log is stored on the leader's disk as a sequence of **log segments**: a rolling set of immutable files (with index files for fast offset lookup). The **active segment** receives appends; when it reaches `segment.bytes` (default 1 GiB ⚠, well-documented default not re-verified this pass) or `segment.ms`, it is sealed and a new active segment starts. Sealed segments are the unit of retention deletion, replication and (since tiered storage) offloading.
- **Retention (cleanup.policy=delete)** — segments are deleted when they exceed `retention.ms` (default **7 days** ⚠, the long-standing documented default) or `retention.bytes`; deletion is coarse-grained (whole segments), so actual retention can overshoot the target by up to a segment. Logs are also bounded by `log.retention.check.interval.ms`.
- **Compaction (cleanup.policy=compact)** — for **keyed** topics, compaction keeps, per key, **only the latest record** (and tombstone records that delete a key), discarding older versions of the same key. A compacted topic is therefore a **changelog** — a full materialized view of every key's current state — which is exactly what Kafka Streams state stores, reference-data topics and "table" semantics need (§5.2). The log-compaction improvement thread is a long-running Kafka effort (the KIP commonly associated with it is **KIP-354 ⚠** — the number was not re-verified this pass; log compaction itself is core, documented Kafka behavior ✅).
- **cleanup.policy=compact,delete** — both: compact by key AND expire by time/size (the common choice for changelogs that must not grow forever).
- **The storage consequence** — a partition is never rewritten in place; disk usage grows to the retention window, and the **3× replication factor** multiplies the bill — the reason storage cost is Kafka's most cited pain point (§8.1) and the motivation for **tiered storage** (§7.3), which moves cold segments to object storage.

### 2.6 Producers

- **The write path** — a producer sends records (key, value, headers, timestamp) for a topic; the **partitioner** routes each record to a partition (default: hash of the key, or round-robin/sticky for null keys ⚠ — sticky partitioning is the modern default behavior, not re-verified this pass). The producer **batches** records per partition (`batch.size`, `linger.ms`) before sending — batching is the main lever for throughput and the main cause of the *per-record* latency-vs-throughput trade-off.
- **acks** — the durability knob ✅: `acks=0` (fire-and-forget — record handed to the socket, no confirmation; highest throughput, can lose data), `acks=1` (leader writes to its local log and confirms; the historical default ⚠ — not re-verified this pass), `acks=all` (all in-sync replicas confirm; the only setting compatible with the durability contract of §3.3). Combined with `min.insync.replicas` on the broker, `acks=all` is the banking-grade setting.
- **Retries and the idempotent producer** — producers retry on transient errors; with **idempotence enabled** (`enable.idempotence=true`, which adds a producer ID + sequence number per partition so the broker can detect and drop duplicate appends), retries no longer risk duplicate records and **per-partition ordering is preserved** ✅. Idempotence became the producer default in a recent major version ⚠ (the exact default-since version was not re-verified this pass) — the idempotent producer itself dates to Kafka 0.11 alongside transactions (§4.3).
- **The compression lever** — `compression.type` (gzip, snappy, lz4, zstd) trades CPU for bandwidth/disk; on banking volumes (millions of payment events) compression routinely cuts storage cost by 3–10× ⚠ (order-of-magnitude experience).

### 2.7 Consumers

- **The poll model** — a consumer is a loop: call `poll()` (the client fetches available batches, hands them to your handler, returns), process, commit, repeat. The broker does not push; the **consumer paces itself**, which is what makes slow consumers safe (the log buffers them) and fast consumers fast (they pull as much as they can).
- **Offset commit** — with `enable.auto.commit=true` the client commits the last polled offsets periodically (at-least-once-ish by default); with manual commits the application commits **after** processing (the correct pattern for at-least-once, §4.1). The committed offset is what a crashed/restarted consumer resumes from.
- **The default delivery guarantee is at-least-once** ✅ — process-then-commit means a crash between processing and committing causes **reprocessing** (duplicates possible); the alternative order (commit-then-process) risks **loss**. This is the semantic foundation of §4.
- **The liveness contract** — a consumer must keep polling: `max.poll.interval.ms` bounds the time between polls (if your handler takes too long, the member is removed and the group rebalances — the classic "processing too slowly" failure), and `session.timeout.ms` bounds heartbeat silence. The consumer is therefore not a good fit for "block on a long job inside the poll loop" without care — the patterns that work are async processing with commits managed by the job's completion, or Kafka Streams (which manages all of this for you, §5).

### 2.8 The Internal Topics and the Operational Surface

A Kafka cluster runs on more than the data topics. The internal topics an operator must know (they show up in every monitoring dashboard):

- **`__consumer_offsets`** ✅ — where consumer groups commit their offsets (§2.3); a compacted topic keyed by (group, topic, partition); its size and ISR health are the first things to check when "consumers restart from the beginning".
- **`__transaction_state`** ✅ — the transaction coordinator's log of open transactions (§4.3); grows with transactional workload; compacted.
- **The Connect topics** ✅ — `connect-config`, `connect-offsets`, `connect-status`: distributed Connect's coordination and offset bookkeeping (§6.1).
- **The Streams internals** — every Streams application creates its own internal topics: **changelog topics** (compacted, backing the state stores, §5.1) and **repartition topics** (for key-changing operators). They are auto-created and auto-named (`<appId>-<store>-changelog`), and they must be in backup/DR scope — MirrorMaker 2 mirrors them like any topic (§6.4) ✅/⚠ (the behaviour is well documented; the exact naming patterns flagged ⚠).
- **The operational surface** ✅ — the Admin API and CLI tools: `kafka-topics.sh` (create/alter/describe), `kafka-consumer-groups.sh` (lag inspection — the #1 operational query), `kafka-configs.sh`, and in 4.0 the new `kafka-groups.sh` covering the KIP-848/KIP-932 group types (the new tool is listed in the 4.0.0 announcement). A platform team's monitoring reduces to: **consumer lag, ISR health, under-replicated partitions, and the KRaft controller quorum**.

---

## 3. The Replication

### 3.1 The ISR (In-Sync Replicas)

Replication is where Kafka earns its durability claim, and the **ISR — the set of In-Sync Replicas** — is the concept that makes the durability contract precise ✅:

- **Replication factor (RF)** — each partition is copied to RF brokers (production standard: **RF=3**). One replica is the **leader**; the others are **followers**. All produce/consume traffic goes to the leader; followers replicate by pulling batches from the leader (**replica fetchers** — the same fetch mechanism consumers use, which is why followers and consumers share the "pull" DNA of §1.2).
- **What "in sync" means** — a follower is in the ISR while it stays caught up with the leader: it must not lag more than `replica.lag.time.max.ms` (default 30 s ⚠, documented default not re-verified this pass) and must not fall too far behind. A follower that lags out is **removed from the ISR**; it is not a candidate for leadership until it catches up and re-enters.
- **The ISR is the durability unit** — `acks=all` means "the leader has written the record and **every replica currently in the ISR** has acknowledged it". If the ISR has 3 members and one is slow, `acks=all` waits for the other two (the slow one is dropped from the ISR rather than blocking the write forever).
- **min.insync.replicas** — a broker-side topic config that refuses produce requests when the ISR **shrinks below the minimum** (e.g. `min.insync.replicas=2` with RF=3). This is the guard that makes `acks=all` meaningful: without it, a leader alone in its ISR could ack writes that exist on exactly one replica. With it, the broker prefers **unavailability over silent loss** when too many replicas are down — the correct default for banking data.
- **The high watermark** — the offset up to which **all ISR replicas** have replicated; consumers (with `read_uncommitted`, §4.4) only ever see records up to the high watermark, so a consumer never reads a record that could still be lost if the leader fails.

### 3.2 Leader Election: Preferred Leaders and the Unclean Trade-Off

- **Election from the ISR** — when a leader fails, a new leader is elected **from the ISR** (never from an out-of-sync replica by default). Because every ISR member has acknowledged everything the old leader acked, **no acknowledged record is lost** — this is the core of Kafka's durability guarantee.
- **Preferred leader / preferred replica** ✅ — each partition has a **preferred replica** (by default the first replica, which was the original leader). The broker runs **preferred-replica election** to return leadership to it when it is back in sync — restoring the original, balanced distribution of leadership across brokers and making leadership deterministic (the operator can predict which broker leads what). This is standard, documented Kafka behavior.
- **The unclean election trade-off** ✅ — `unclean.leader.election.enable` (default **false**) controls whether a partition may elect a leader **from outside the ISR** (an out-of-sync replica). Enabling it buys **availability** (a partition keeps serving writes even when all in-sync replicas are down) at the cost of **data loss** (records that only the dead leader held are gone — the new leader's log is shorter). Disabling it buys **consistency** at the cost of availability (the partition is offline until an ISR member returns). For a bank: keep it **false**; the regulatory and reconciliation cost of silent data loss dwarfs the cost of a partition being briefly unavailable (the trade-off is documented Kafka knowledge ✅; the "keep false for banking" is architectural judgment).
- **KIP-966 — Eligible Leader Replicas (ELR), preview in 4.0** ✅ (verified this pass in the 4.0.0 announcement) — a middle path: ELR is "a subset of the ISR replicas guaranteed to have complete data up to the high-watermark" and "safe for leader election, preventing data loss" — i.e. replicas that are not fully in sync but are provably complete up to the watermark can be elected without the data-loss risk of unclean election.

### 3.3 acks=all + ISR = the Durability Contract

The combination an architect should treat as **the banking-grade write path**:

- `acks=all` (producer) + `min.insync.replicas=2` (topic, with RF=3) means: a record is **acknowledged only after it is durably stored on at least 2 of 3 replicas**, and the broker **stops accepting writes** if the ISR ever drops below 2. The result: the system can **lose any single broker** (and, transiently, a second) without losing an acknowledged record, and it can never silently ack a record that exists on only one replica.
- **What it costs** — latency (every ack waits for two disk writes, one on the leader and one on a follower) and availability (one broker down can take a partition's writes offline if the ISR is already at the minimum). These are the explicit trade-offs of the contract; §8.1 discusses them as pain points, and §10.3 shows how a platform team prices them per topic class (payments: strict contract; analytics: relaxed).
- **The failure math** ✅/⚠ — with RF=3 and min.insync=2, acked records survive: any 1-broker failure (always), and 2-broker failures only if the ISR had ≥2 members and the surviving broker has the record (standard replication reasoning; the precise probability depends on correlated-failure assumptions ⚠).

### 3.4 KRaft: The Raft-Based Metadata Quorum

- **What KRaft is** — KRaft replaces ZooKeeper with an **internal metadata quorum run on Raft**: a small set of **controller nodes** (which can be co-located with brokers or run separately) elect a **controller leader** via the Raft consensus protocol, and all cluster metadata (brokers, topics, partitions, configs, leadership) lives in a **metadata log** replicated across the quorum ✅. This is KIP-500, the multi-year "Kafka Raft metadata mode" effort.
- **The history** ✅ — **early access in Kafka 3.3 (October 2022)** (repo-verified); ZooKeeper mode was deprecated in the 3.x era ⚠ (the exact deprecation release — commonly cited as 3.5 — was not re-verified this pass); **Kafka 4.0 is KRaft-only** (verified this pass, §3.5).
- **What it changes operationally** — (1) **one less system**: no ZooKeeper ensemble to run, patch, secure and monitor; (2) **faster failover**: metadata leadership elects in seconds via Raft, versus the ZooKeeper-era controller-election dance; (3) **metadata scalability**: metadata is a replicated log, not a ZK znodes tree, removing the ZK limits on partition counts; (4) **simpler recovery**: no "which broker had the latest metadata" archaeology after a split-brain-ish event; (5) the KRaft quorum is itself a Kafka-style log, so the operational skills are the same skills. The 4.0 announcement's own summary: KRaft "simplifies deployment and management, eliminating the complexity of maintaining a separate ZooKeeper ensemble", "significantly reduces operational overhead, enhances scalability, and streamlines administrative tasks" ✅.
- **KIP-996 — Pre-Vote (4.0)** ✅ (verified this pass) — a Raft "pre-vote" mechanism that reduces unnecessary KRaft leader elections caused by network partitions or transient issues: a node checks its eligibility before starting an election, so flaky conditions do not churn the controller quorum.

### 3.5 ZooKeeper Fully Removed (Kafka 4.0, 18 March 2025)

**Verified this pass at the primary source** — the Apache Kafka 4.0.0 release announcement (kafka.apache.org, by David Jacot, **Tuesday 18 March 2025**) states: "Apache Kafka 4.0 is a significant milestone, marking the **first major release to operate entirely without Apache ZooKeeper**", running **"in KRaft mode by default"**. Concretely:

- **4.0 is KRaft-only** — ZooKeeper support is **removed, not deprecated**: you cannot run a 4.0 cluster with ZooKeeper, and ZooKeeper-mode clusters **cannot upgrade directly to 4.0** — they must migrate to KRaft first (§3.6). The announcement also thanks the ZooKeeper community, noting ZooKeeper "was the backbone of Kafka for more than 10 years" ✅.
- **What else shipped in 4.0** ✅ (all verified in the announcement): **KIP-848** consumer-rebalance protocol GA (§2.4); **KIP-932** Queues for Kafka **early access** (share groups — cooperative consumption over regular topics, Kafka's answer to traditional queue semantics); **KIP-890** Transactions server-side defense, **phase 2** (§4.4); **KIP-966** Eligible Leader Replicas preview (§3.2); **KIP-996** Pre-Vote (§3.4); and the Java baseline: **brokers, Connect and tools require Java 17; clients and Kafka Streams require Java 11** ✅.
- **The date** — **18 March 2025** ✅ (the announcement's publication date, confirmed in the URL and page metadata: "Tuesday, March 18, 2025").

### 3.6 The ZooKeeper-Era Migration Path

For the many production estates still on ZooKeeper mode (a real Cymbal Bank reality in 2026 — large banks migrate slowly), the documented path is:

1. **Upgrade to a 3.x bridge release first** — the Kafka docs' own upgrade guidance: from ZooKeeper mode you upgrade within 3.x (the 3.5–3.7 line is the maintained bridge; later 3.x releases continued it) ✅/⚠ (the "upgrade to 3.x, then migrate, then 4.0" shape is documented Kafka guidance; the exact set of supported bridge versions was not re-verified this pass ⚠).
2. **Migrate to KRaft in place** — run the KRaft migration tooling (`kafka-storage.sh` / the `--migration` workflow introduced in the 3.4–3.5 era) to convert the ZooKeeper metadata into a KRaft metadata log while the cluster keeps serving ⚠ (the tooling exists and is documented; the exact command surface not re-verified this pass).
3. **Then upgrade to 4.0** — once the cluster is KRaft-mode, the 4.0 upgrade is a normal rolling upgrade.

The banking lesson: **the migration is a project, not a flag flip** — it touches every environment (dev/test/DR/prod), needs rollback plans, and interacts with every other 4.0 change (Java 17 on brokers, the new rebalance protocol). Platform teams should sequence it with the application-flat Java upgrades. This is exactly the kind of estate-level change the [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) guide's patterns are for.

### 3.7 The Replication Configuration Quick Reference

| Config | Setting | What it does |
|---|---|---|
| `replication.factor` | 3 (production standard) | Copies per partition — the durability floor of §3.1 |
| `min.insync.replicas` | 2 (tier-1 topics) | Refuses writes when the ISR shrinks below the floor (§3.3) |
| producer `acks` | all (tier-1) | Waits for all ISR replicas before acking (§3.3) |
| `unclean.leader.election.enable` | false (default — keep) | No leadership from outside the ISR: no silent data loss (§3.2) |
| `auto.leader.rebalance.enable` | true (default) | Preferred-replica election returns leadership to the preferred replica (§3.2) |
| `replica.lag.time.max.ms` | 30000 (default ⚠) | How long a follower may lag before leaving the ISR (§3.1) |
| Controller quorum | 3 controllers (KRaft, odd number) | The Raft majority for metadata (§3.4) |

The one-line rule: **durability = RF × min.insync × acks; availability = the slack between them** — price both per topic class (§10.3), never globally.

---

## 4. The Messaging Semantics

### 4.1 At-Least-Once (the Default)

- **The default guarantee** ✅ — Kafka consumers default to **at-least-once**: records are processed, then offsets are committed, so a crash between processing and commit causes the same records to be **delivered again** on restart. No record is lost; **duplicates are possible**.
- **Where duplicates come from** — (1) the process-then-commit window (crash after processing, before commit); (2) producer retries with idempotence disabled (§2.6); (3) rebalance-triggered re-processing of the revoked partitions' tail. All three are normal, not exceptional.
- **The standard remedy** — make consumers **idempotent at the application level**: dedupe on an **event ID / idempotency key** (the payment `endToEndId`, the trade `tradeId`, a UUID header) stored in the target system, or design the downstream write to be naturally idempotent (INSERT ... ON CONFLICT DO NOTHING, "last-writer-wins" updates on the event timestamp). For a bank, at-least-once + idempotency-key dedupe is the **default production pattern** — including for payments, where the ISO 20022 `EndToEndId` exists precisely to make retries safe (cross-ref [ISO 20022 Core Processes](iso_20022_core_processes_guide.md)).
- The discipline-level discussion (why at-least-once is the sane default, and the dedupe patterns) lives in [Event Stream Processing](../technology/event_stream_processing_guide.md) — cross-ref rather than re-derive.

### 4.2 At-Most-Once

- **The guarantee** — commit the offset **before** processing (or disable retries with `acks=0`-style producers): each record is delivered at most once; **duplicates are impossible, loss is possible** (crash after commit, before processing).
- **Where it is the right choice** — telemetry, metrics, audit *counters* where a missing sample is tolerable and a duplicated one would corrupt aggregates; log ingestion where the source can be re-scanned. **Where it is wrong** — anything with money, positions, or regulatory significance. A bank uses at-most-once only for explicitly loss-tolerant streams, and marks those topics as such in the topic catalog (§10.2).

### 4.3 Exactly-Once: Idempotent Producers + Transactions

- **The building blocks, both from Kafka 0.11 (2017)** ✅ (repo-verified) — (1) the **idempotent producer** (KIP-98): each producer instance gets a producer ID and stamps every record with a per-partition sequence number; the broker detects and drops duplicate/out-of-order appends from retries — duplicates *within a producer session* disappear; (2) **transactions** (KIP-98): a producer can mark a set of records across **multiple partitions/topics** as one atomic transaction; the broker writes a transaction marker and consumers with `isolation.level=read_committed` only see **committed** transaction records.
- **What "exactly-once" means in Kafka** ✅ — EOS is defined for the **read-process-write cycle within Kafka**: consume from input topic(s), process, produce to output topic(s) and **commit the input offsets in the same transaction**. Either all of it commits (output records + offset commit become visible together) or none does — so a crash mid-cycle cannot produce "output written but offset not committed" (duplicate processing) or "offset committed but output lost" (dropped processing). The downstream consumer sees each result **exactly once**.
- **The API shape** — a transactional producer (`transactional.id`, `initTransactions`, `beginTransaction`, `sendOffsetsToTransaction`, `commitTransaction`), with the consumer's `isolation.level` set to `read_committed`. Kafka Streams does all of this internally when `processing.guarantee=exactly_once_v2` (§5.3).

### 4.4 read_committed, transactional.id and Zombie Fencing

- **isolation.level** ✅ — `read_uncommitted` (default) sees all records including uncommitted/aborted transaction data; `read_committed` sees only committed transactions and strips aborted ones (and waits out in-flight transactions on a partition). The choice is per-consumer, not per-topic.
- **transactional.id and the epoch** ✅ — a producer's `transactional.id` gives it a **stable identity across restarts**; every time a producer starts, it acquires a new **epoch** for that ID. The broker (via the transaction coordinator) enforces that **only the current epoch may write** — so if an old instance (a **zombie**) survives a partition/split-brain and tries to resume its transaction, its writes are **fenced off** (rejected) because its epoch is stale. This is the **zombie fencing** mechanism — the same idea as the fencing tokens in distributed-lock patterns.
- **KIP-890 — Transactions server-side defense** ✅ (verified this pass) — the 4.0.0 announcement: "The second phase of KIP-890 has been completed. It reduces the chances of **'zombie transactions'** during producer failures" — i.e. the broker itself now defends against zombie transactions (server-side state and timeouts), rather than relying only on client-side fencing. This is the same KIP family that introduced the `exactly_once_v2` processing guarantee (which Streams uses, §5.3) ⚠/✅ (the `exactly_once_v2` link to the KIP-890 work is from the author's knowledge base ⚠; the phase-2 completion in 4.0 is verified ✅).

### 4.5 The Honest Caveat: Cross-System Exactly-Once

The sentence every Kafka guide must say plainly: **Kafka's exactly-once is exactly-once *within Kafka* — it does not, by itself, make a transaction that spans Kafka and an external system (a database, an API, a core-banking host) exactly-once.** The honest architecture:

- **The outbox pattern** ✅/⚠ (a standard distributed-systems pattern, presented as such) — write the business state **and** the event to the same database in one local transaction (the event row is the "outbox"); a relay (Debezium CDC, §6.2, or a polling publisher) publishes the outbox rows to Kafka. The DB transaction gives atomicity at the source; Kafka gives durable, ordered delivery from there on; the consumer dedupes on the event ID. This is the pattern that makes "exactly-once from core system to event platform" actually achievable, and it is the pattern §10's platform design assumes.
- **App-level idempotency keys** — for any external call (a payment instruction to a clearing house, a SWIFT message), the only real guarantee is a **unique idempotency key** that the counterparty honors. Kafka can carry the key; it cannot create the counterparty's dedupe.
- **The operational reality** ✅/⚠ — most teams, including most banks, run **at-least-once + dedupe** in production rather than full EOS: it is simpler to operate, debuggable, and the dedupe is needed anyway for the cross-system boundary. EOS (transactions) is adopted where the read-process-write cycle is fully inside Kafka (Streams apps, §5.3) and where duplicate-sensitive aggregates (positions, limits) would otherwise corrupt state. This is architectural judgment, consistent with the [Event Stream Processing](../technology/event_stream_processing_guide.md) sibling's treatment.

---

## 5. The Streaming

### 5.1 Kafka Streams: A Library, Not a Cluster

- **What it is** ✅ — **Kafka Streams** is a **Java (and Scala) client library** for stream processing: your application embeds it, defines a **topology** (source processors reading topics → stateless operators like map/filter → stateful operators like aggregations and joins → sink processors writing topics), and runs as a normal application. **There is no separate processing cluster** — no Streams servers to run, unlike Spark/Flink; scaling = running more application instances, which partition the work via the consumer-group protocol (§2.4). This "library, not cluster" property is Kafka Streams's defining operational advantage and its defining constraint (processing is JVM-embedded, so heavy state lives in local state stores, not a shared service).
- **The history** ✅/⚠ — introduced in **2016** (repo-verified); the version commonly cited is **0.10.0** — corroborated this pass by the Apache **0.10.0.0 release notes**, which show the KStreams DSL and state-store work landing (KAFKA-2653 "Stateful operations in the KStream DSL layer", KAFKA-2706 "Make state stores first class citizens in the processor DAG", KStream Java-8-function interfaces). The exact "first GA in 0.10.0" phrasing is flagged ⚠ only on the version number; the year is ✅.
- **Local state stores** ✅/⚠ — stateful operators (aggregations, joins, windowing) keep their state in **local state stores** (RocksDB-backed by default ⚠ — the default engine is well documented but not re-verified this pass), each backed by a **changelog topic** (a compacted topic, §2.5) so any instance can rebuild its state by replay — the log-centric recovery of §1.3 in action.
- **Java 11 minimum since 4.0** ✅ (verified this pass) — Streams (like the clients) requires Java 11 as of Kafka 4.0.

### 5.2 The Stream-Table Duality

- **The duality** ✅ — **a stream is a table's changelog; a table is a stream compacted by key** (the §1.3 insight, now as API): `KStream` is the record stream (every event, in order, possibly with duplicates/updates); `KTable` is the table view (per key, the current state — backed by a compacted changelog). `GlobalKTable` is the fully-replicated variant (every instance holds the whole table, for broadcast reference data like FX rates or instrument master).
- **The joins** — stream-stream (windowed joins of two event streams, e.g. trade events × market-data ticks), stream-table (enrich an event with reference data at event time — the daily bread of banking: enrich a payment event with the customer/account master), table-table (merge two tables). All joins are key-based, which is why **key design** (§10.2) is the make-or-break decision.
- **Interactive queries** — a Streams app's state stores are **queryable** (the app exposes a query API over its local stores, with routing across instances) — this is how "the table" becomes a low-latency read model without a separate database, the CQRS read side of §1.3.

### 5.3 Exactly-Once in Streams

- **The setting** ✅ — Streams apps are the natural home of EOS (§4.3): the whole read-process-write cycle is inside Kafka, so the transactional machinery applies cleanly. `processing.guarantee=exactly_once_v2` makes Streams consume, process, produce **and commit consumer offsets and state-store changelog updates** in a single transaction per task — a crash leaves no half-applied window, no double-counted aggregation, no lost offset.
- **The version note** ✅/⚠ — the original EOS guarantee arrived with Kafka 0.11 (2017, repo-verified); the `exactly_once_v2` processing guarantee is the KIP-890-generation improvement ⚠ (introduced in the 3.x era per the author's knowledge base; the specific release was not re-verified this pass), and the KIP-890 server-side defense phase 2 completed in 4.0 ✅ (verified).
- **When to use it** — exactly-once Streams is the right default for **stateful** banking aggregations (intraday limits, position keeping, fraud counters) where a duplicate would corrupt a number that is then relied upon; for stateless transforms, at-least-once + dedupe is usually sufficient and cheaper to operate (the §4.5 judgment applies).

### 5.4 ksqlDB: SQL over Streams

- **What it is** ⚠/✅ — **ksqlDB** is Confluent's **streaming SQL engine built on Kafka Streams**: you write `CREATE STREAM` / `CREATE TABLE` / `SELECT ... EMIT CHANGES` / aggregations / joins in SQL, and ksqlDB compiles them into Streams topologies. It **formerly shipped as KSQL** (the rename KSQL → ksqlDB is well documented ✅; the exact rename/announcement dates — KSQL open-sourced 2017, renamed ksqlDB 2019/2020 — were **not re-verified this pass** ⚠, and the search backend degraded before they could be checked).
- **How it runs** ⚠ — ksqlDB runs as its own **server cluster** (the `ksqldb-server` processes) that executes the compiled Streams topologies, with a REST/CLI interface — so it is a *platform* component (someone operates it), unlike embedded Streams. It supports **push queries** (continuous result streams) and **pull queries** (point lookups into materialized tables). (Operational shape from the author's knowledge base, flagged ⚠.)
- **Why a bank cares** — ksqlDB is the **self-service analytics layer**: business teams (fraud ops, risk, product) can express "give me a stream of payments above a threshold per customer" in SQL without a Java/Streams development cycle — the data-mesh self-service story of §9.2. It is also the fastest way to prototype a Streams pipeline before productizing it in Java.

---

## 6. Connect and the Schema Registry

### 6.1 Kafka Connect: The Integration Layer

- **What it is** ✅ — **Kafka Connect** is the framework (part of Apache Kafka) for streaming data **into and out of** Kafka at scale: **source connectors** poll external systems and produce records to topics; **sink connectors** consume topics and write to external systems. A **connector** is the integration logic (a plugin); the work is split into **tasks** (parallel work units, one per partition-ish unit of the source/sink); **workers** are the processes running connectors and tasks.
- **The two modes** ✅ — **standalone mode** (a single process, for development and small deployments) and **distributed mode** (a cluster of workers that coordinate via internal Kafka topics — `connect-config`, `connect-offsets`, `connect-status` — and rebalance connectors/tasks across workers when workers join/leave, mirroring the consumer-group protocol of §2.4). Distributed mode is the production choice; connectors are managed through the **REST API** (`POST /connectors`, status endpoints, config validation).
- **Why it matters for a bank** — Connect is the *integration spine*: it is how the core-banking host, the payments hub, the data warehouse and the regulatory reporting store all attach to the event platform without custom point-to-point code — the §1.3 decoupling made operational, and the piece the platform team operates as a service (§9.3).

### 6.2 The Connector Ecosystem (Debezium CDC, S3, JDBC, Elasticsearch)

- **Debezium (CDC)** ✅/⚠ — **Debezium** is the open-source **change-data-capture** connector family (the project is real, Apache-licensed, and the standard CDC choice ✅; the exact supported database list — PostgreSQL, MySQL, Oracle, SQL Server, MongoDB, and more — was not re-verified this pass ⚠). It reads database transaction/redo logs and emits **row-level change events** to Kafka; combined with the **outbox pattern** (§4.5) it is the standard way to get core-system events onto the platform. CDC is the single most important banking connector: it is what turns a legacy core into an event producer without touching the core's code.
- **The rest of the catalog** ✅/⚠ — S3 sink (records to object storage as JSON/Parquet — the data-lake bridge), JDBC source/sink (relational tables in/out), Elasticsearch/OpenSearch sink (search indexes), plus hundreds of community connectors (the "Confluent Hub" catalog is real ⚠ — exact counts not re-verified this pass). The pattern: **every external system gets a connector; applications stop writing point-to-point integrations**.
- **The operational reality** ✅/⚠ — connectors are code that must be versioned, tested and monitored like any service (tasks fail, offsets lag, schemas drift); a mature platform runs Connect as a managed, governed service (connector-as-a-service with CI review), not as "anyone can POST a connector".

### 6.3 Schema Registry: Avro, JSON, Protobuf and Compatibility

- **What it is** ✅/⚠ — the **Schema Registry** (Confluent's open-source component, also bundled in Confluent Platform/Cloud; other implementations exist ⚠) is a REST service that **stores and versions schemas** (Avro, JSON Schema, Protobuf) for topics and hands out schema IDs; producers register/validate their schema and prefix records with a small header (the "magic byte" + 4-byte schema ID), so the **wire format carries the schema ID, not the whole schema** — compact and self-describing. (Wire-format details from the author's knowledge base, flagged ⚠; the component itself is real and central ✅.)
- **Why it exists** — producer/consumer decoupling under evolution: a consumer can decode a record produced under an older schema as long as the registry knows both versions — which is what makes **schema evolution** safe in a fan-out platform with dozens of consumers per topic.
- **The compatibility levels** ⚠ (list not re-verified this pass; standard Confluent documentation from the author's knowledge base) — **BACKWARD** (new schema can read data written with the old — the default ⚠), **BACKWARD_TRANSITIVE**, **FORWARD** (old schema can read data written with the new), **FORWARD_TRANSITIVE**, **FULL**, **FULL_TRANSITIVE**, **NONE** (no checks). The governance rule: **BACKWARD (or FULL) as the platform default, NONE only for explicitly experimental topics**, with compatibility checked in CI on every proposed schema change.
- **The governance angle** — for a bank, the Schema Registry is where the **contract** between producers and consumers is enforced: a breaking schema change is caught at registration time, not in production at 2 a.m.; every topic's schema history is the machine-readable version of the event-contract documentation (§9.3). This is the schema-governance backbone of the topic design in §10.2.

### 6.4 MirrorMaker 2: Cross-Cluster Replication and DR

- **What it is** ✅/⚠ — **MirrorMaker 2 (MM2)** is the Apache Kafka cross-cluster replication tool (it replaced MirrorMaker 1; built on the Connect framework — the architecture is documented ✅; specific operational details flagged ⚠ where not re-verified). It **mirrors topics from a source cluster to a target cluster** (same name, or a prefixed name such as `dr.payments.executed`), continuously.
- **The banking use cases** — **disaster recovery** (active-passive: the DR cluster is kept warm by MM2; on failover, consumers switch to DR and resume from mirrored offsets), **active-active** regional setups, and **migration** (replicate, cut over, decommission). MM2 also mirrors **consumer-group offsets** so consumers can resume on the DR side near where they were — the property that makes DR failover *resumable* rather than *replay-from-scratch*.
- **The honest framing** ✅/⚠ — MM2 is **asynchronous replication**: the DR cluster lags the primary by the replication latency (seconds, typically), so DR RPO is not zero; the RPO/RTO targets are a design decision (§10.3), and the failover runbook (who flips the consumers, how idempotency protects the overlap window) is exercised in drills — the [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) guide's territory.

---

## 7. The Ecosystem

### 7.1 Client Libraries

- **The official Java client** ✅ — the reference implementation, maintained with the broker (both Java 11+ since 4.0, §3.5). Everything else interoperates through the **versioned wire protocol** — one of Kafka's quiet strengths: the protocol is public, versioned and stable, which is why the client ecosystem is large.
- **librdkafka** ✅/⚠ — the C/C++ library that is the *de facto* native implementation; the foundation under **confluent-kafka-python**, **node-rdkafka**, and the Go bindings (the library and its role are well documented ✅; exact binding maintenance status ⚠ not re-verified this pass).
- **Go** ✅/⚠ — **kafka-go** (pure-Go, popular) and **sarama** (pure-Go, older, widely used in production) — both real and documented ✅; current maintenance status ⚠.
- **The pattern** — a bank's platform team standardizes on a short list (Java for backend, confluent-kafka-python/librdkafka for Python/analytics, kafka-go for Go services) and version-pins them, because client version skew is a real production-issue source (protocol negotiation, rebalance-protocol support).

### 7.2 Managed Offerings

| Offering | What it is | Notes |
|---|---|---|
| **Confluent Cloud / Platform** ✅ | SaaS (Cloud) and self-managed enterprise distribution (Platform) by the founders' company | Schema Registry, ksqlDB, connectors, Control Center bundled; the commercial layer of §7.4 |
| **Amazon MSK** ✅/⚠ | Managed Kafka on AWS (launched 2018 ⚠ — widely documented year, not re-verified this pass) | Brings your own clients; MSK Serverless variant; tight IAM/VPC integration |
| **Azure Event Hubs (Kafka endpoint)** ✅/⚠ | Event Hubs speaks the Kafka protocol (Kafka-endpoint support since 2018 ⚠ — well documented, not re-verified this pass) | Lets Kafka clients talk to a non-Kafka service — the "Kafka-compatible without running Kafka" option |
| **Google Managed Kafka** ✅/⚠ | Google's managed Kafka, **announced 2024** (at Google Cloud Next '24 ⚠/✅ — announcement year well documented, month flagged) | Completes the big-three cloud coverage |
| **IBM Event Streams** ✅/⚠ | Kafka-based event streaming on IBM Cloud | The IBM estate's Kafka; real product, details ⚠ |
| **Red Hat AMQ Streams** ✅/⚠ | Kafka on OpenShift, built on the **Strimzi** operator (the Strimzi base is well documented ✅) | The Kubernetes-native operator route for on-prem/private-cloud banks |

What managed buys: the control plane (upgrades, monitoring, KRaft migration, patching) — for a bank with scarce Kafka ops talent, often decisive. What it costs: per-hour/per-GB pricing that can exceed self-managed at scale, and less control over broker tuning. The decision matrix is [Kafka Alternatives](../technology/kafka_alternatives_guide.md)'s business — this guide only frames it.

### 7.3 Tiered Storage

- **What it is** ✅ (repo-verified) — **tiered storage went GA in Kafka 3.6 (2023)**: brokers can offload **sealed log segments** (§2.5) to **object storage** (S3/GCS/Azure Blob) while keeping recent data on local disk; consumers can fetch from the remote tier transparently.
- **Why it matters** — it breaks the "retention is expensive" equation: the local disk holds the hot window (days), the object tier holds the cold history (months/years) at a fraction of the cost — and Kafka's replay property then applies to **year-old records**, not week-old ones. For banking this is transformative for regulatory retention (years-long replay windows at sane cost) and for the "replay for audit/legal" use cases of §9.2. The 3× replication cost (a §8.1 pain point) applies to local storage; the object tier is typically stored once with its own redundancy ✅/⚠ (the tier's replication model is documented Kafka/Confluent behavior; specifics flagged ⚠).
- **The honest caveat** — tiered storage changes the capacity math but not the semantics: retention policies still bound the log; consumers reading deep into the tier pay object-store latency (fine for batch/replay, not for the hot path).

### 7.4 The Commercial Layer

- **Confluent** ✅ — founded **2014 by Kreps, Narkhede and Rao** (repo-verified); the company ships **Confluent Platform** (self-managed: broker + Schema Registry, ksqlDB, Connect, Control Center, governance tooling) and **Confluent Cloud** (the SaaS). It went public in 2021 ⚠ (the IPO year is well documented; not re-verified this pass).
- **The licensing reality** ✅ — Apache Kafka itself remains **Apache-2.0 open source** (the 4.0.0 announcement and the project's licensing are unambiguous); the "commercial layer" is enterprise features, support and managed operations around it. Some Confluent components (e.g. parts of the governance suite) are not Apache-licensed ⚠ (the specific licensing split not re-verified this pass) — a procurement consideration, not a Kafka consideration.
- **The vendor map** — Confluent vs MSK vs Event Hubs vs self-managed: the choice is about control plane, cost model and existing cloud commitment, and it is a *business-case* decision — cross-ref [Business Case Development](../management/business_case_development_guide.md) for the funding/financial framing and [Kafka Alternatives](../technology/kafka_alternatives_guide.md) for the technical head-to-head.

---

## 8. The Comparison vs the Alternatives

**The full head-to-head lives in [Kafka Alternatives](../technology/kafka_alternatives_guide.md) — this section does not re-derive it.** It summarizes the axes an architect must hold when someone proposes Kafka (or proposes replacing it), with the pointer to the sibling for the depth.

### 8.1 The Pain Points

| Pain point | What it actually is | Status |
|---|---|---|
| **Operational complexity** | A Kafka estate is a distributed system with its own failure modes: rebalances, ISR shrinks, controller elections, the KRaft migration (still ongoing in many banks in 2026), consumer-lag management. It demands dedicated platform engineering. | ✅ (structural; the KRaft migration is §3.6) |
| **JVM footprint** | Brokers and the official clients are JVM services — memory-hungry, GC pauses contribute to tail latency, and the Java baseline moved to 17 (brokers) in 4.0. | ✅ (Java baseline verified this pass) |
| **Storage cost** | RF=3 (§3.1) × retention (§2.5) means every record costs ~3× its size for as long as it is retained; disk is the dominant line item at banking volumes. Tiered storage (§7.3) mitigates, does not eliminate. | ✅/⚠ (the 3× multiplication is arithmetic; cost ratios are deployment-dependent ⚠) |
| **Tail latency** | p99/p999 spikes from GC, page-cache pressure, rebalances and follower catch-up make Kafka a poor fit for single-digit-millisecond critical paths. | ✅/⚠ (well-documented operational behavior; figures are deployment-dependent ⚠) |
| **Exactly-once subtlety** | EOS is within-Kafka only (§4.5); the cross-system story requires outbox + idempotency keys — a discipline, not a switch. | ✅ (this guide §4.5; the sibling's treatment) |
| **Topic sprawl & governance** | Without a platform team, topics/schemas/consumers proliferate and the backbone becomes a swamp — the operating-model answer is §9.3. | ⚠ (organizational, experience-based) |

### 8.2 The Strengths

- **The de facto standard** ✅ — Kafka is the default answer for "durable event backbone" in the industry; hiring, tooling, connectors and managed offerings all assume it (the ecosystem of §7 exists *because* of this). Choosing Kafka is rarely a career-risk decision; choosing an exotic alternative is.
- **The ecosystem** ✅ — clients in every language, Connect connectors for every system, Streams/ksqlDB in-platform, managed options on every cloud (§6–§7). No alternative matches the breadth.
- **Durability + replay** ✅ — acks=all + ISR (§3.3) plus retention/compaction (§2.5) gives *auditable, replayable* history — the property banks need for regulatory and reconciliation use cases and that pure brokers do not provide.
- **Ordering per partition** ✅ — key-based ordering (§2.2) is exactly what payment/trade state machines need.
- **Throughput at scale** ✅/⚠ — Kafka's batching design (the paper's design thesis, §1.2) delivers very high throughput on commodity hardware; the specific numbers are workload-dependent ⚠ (the alternatives guide benchmarks).

### 8.3 When NOT to Pick Kafka

| Scenario | The right answer | Why |
|---|---|---|
| **Latency-critical hot paths** (trading tick-to-order, market-data fan-out) | **Aeron / Chronicle** (or other low-latency messaging) on the hot path — see [Low Latency C++ Development](../technology/low_latency_cpp_development_guide.md) | Kafka's JVM, batching and p99 tail (§8.1) exclude it from single-digit-microsecond paths; the honest pattern is **Kafka around the hot path, not on it** — events flow to Kafka for persistence/audit *after* the trade is done |
| **Small deployments** (one team, modest throughput) | Lightweight brokers (NATS, Redis Streams, RabbitMQ) | A Kafka cluster + KRaft quorum + Connect + Schema Registry is heavy machinery for 100 events/s; the operational tax is not worth it |
| **Serverless / elastic-scale, storage-decoupled** | **WarpStream** (storage on object store, no local disks) or **Redpanda** (C++ drop-in) ⚠/✅ (both are real, well-documented projects ⚠ for exact current status) | When the Kafka API is wanted but the broker fleet economics are not; the drop-in API keeps the ecosystem |
| **One-shot job queues** (work distribution) | A real queue (SQS, RabbitMQ, NATS JetStream) | Kafka is a log, not a queue; point-to-point work distribution without replay/ordering needs is simpler elsewhere (KIP-932 Queues, §3.5, is Kafka's own answer, still early-access ⚠) |

The decision procedure, in one line: **if you need durable, replayable, ordered, multi-consumer event history — Kafka; if you need a queue or a microsecond path — something else.** The full comparison (feature tables, cost models, the "Kafka vs Redpanda vs Pulsar vs WarpStream vs NATS" matrix) is [Kafka Alternatives](../technology/kafka_alternatives_guide.md), cross-ref §8 there.

**The key axes at a glance** (the sibling's full matrix is the reference — this table only orients):

| Axis | Kafka | The alternatives' edge |
|---|---|---|
| Durability & replay | RF=3, ISR, retention, tiered storage — the strongest story | Pulsar (segment-based, comparable); most brokers weaker |
| Ordering | Per partition via keys | Redpanda/Pulsar comparable; queues give no ordering |
| Throughput | Very high with batching ⚠ (workload-dependent) | Comparable at scale; Redpanda claims lower latency ⚠ |
| Operational weight | High (KRaft, Connect, Schema Registry, rebalances) | Redpanda (C++ single binary), WarpStream (serverless) lower |
| Tail latency | Milliseconds, GC-affected | Redpanda/Pulsar similar class; Aeron/Chronicle are microseconds — a different league (§8.3) |
| Ecosystem | Largest: clients, connectors, hosted options, hiring pool | Growing, smaller everywhere |
| Licensing | Apache-2.0 core; commercial layer around it | Mixed: open-core (Redpanda) vs Apache (Pulsar) ⚠ |

---

## 9. The Banking Context

### 9.1 Why a Cymbal Bank Runs an Event Backbone

A Cymbal Bank (a global CIB with corporate and institutional clients — the [Crédit Agricole Software Systems](credit_agricole_software_systems_guide.md) estate is the reference shape) runs **dozens of core systems**: the retail/wholesale core, the payments hub, the markets/trading platforms (Murex-class, cross-ref [Treasury & ALM](treasury_alm_guide.md) and [Capital Markets Architecture](capital_markets_architecture_guide.md)), risk, finance, regulatory reporting — each with its own data, its own batch cycles, and its own point-to-point integrations. The event backbone is the answer to that sprawl:

- **The §1.3 pattern applied to banking** — every meaningful "happened" fact (a payment executed, a trade booked, a position changed, a limit breached, a regulatory report filed) is an **event**; the backbone records it once, durably and in order, and every system that needs it attaches as a consumer. New consumers (a new fraud model, a new report) attach **without touching the producers** — the decoupling that point-to-point integrations never give.
- **Replay as the bank's superpower** — regulators demand *auditability and lineage* (BCBS 239 data lineage is a live theme — cross-ref [Risk Data Aggregation](risk_data_aggregation_guide.md)); the log's replay property (§2.3) turns "prove what happened on 14 March" from a forensic project into a `seek()` call.
- **The scale argument** — at a global CIB's volumes (millions of payment/trade events per day, peak bursts around market events), Kafka's throughput and the ecosystem (§7) are what make the backbone *economic*: one platform serving payments, markets, risk and regulatory instead of N bespoke pipelines. The economics are worked out in [Business Case Development](../management/business_case_development_guide.md); the technical head-to-head is [Kafka Alternatives](../technology/kafka_alternatives_guide.md).

### 9.2 The Banking Use Cases

| Use case | The event stream | Notes / cross-refs |
|---|---|---|
| **CDC from core systems** | Row-change events from the core DB (outbox + Debezium, §6.2) into `core.<entity>.changed` topics | The bridge from legacy cores to the event platform; cross-ref the bank series ([DBS Bank](dbs_bank_guide.md), [UOB Software Systems](uob_software_systems_guide.md), [OCBC Software Systems](ocbc_software_systems_guide.md)) for the CDC-in-banking patterns |
| **Payments event streams** | Full payment lifecycle: `initiated → validated → executed → confirmed → failed`, aligned to ISO 20022 semantics | Cross-ref [Payments Hub](payments_hub_guide.md) and [ISO 20022 Core Processes](iso_20022_core_processes_guide.md); the `EndToEndId` is the idempotency key of §4.1 |
| **Trade lifecycle events** | `booked → enriched → allocated → confirmed → settled` for rates/FX/credit products | Cross-ref [Treasury & ALM](treasury_alm_guide.md) and [Capital Markets Architecture](capital_markets_architecture_guide.md); position-keeping consumers run exactly-once Streams (§5.3) |
| **Fraud detection** | Real-time scoring over payment/account streams; alerts as events | Cross-ref [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md); latency-sensitive — the Streams/ksqlDB path (§5) |
| **Regulatory reporting** | Reportable events captured once, aggregated for MAS/ECB/regulator filings; replay for audit | Cross-ref [Regtech](regtech_guide.md); the years-long retention of §7.3 is what makes year-old replay possible |
| **Data mesh / analytics** | Domain-owned topics as **data products**; self-service consumers (data science, risk, finance) | Cross-ref [NETS Software Systems](nets_software_systems_guide.md) (a payments/data company running the mesh pattern) and the data-mesh operating model of §9.3 |

### 9.3 The Operating Model

The backbone fails without governance; a Cymbal Bank platform team runs four disciplines:

- **Event ownership** — the **domain owns its topics**: the payments domain owns `payments.*`, markets owns `trade.*`; the owner defines the schema, the retention, the SLA, and answers for breaking changes. Consumers subscribe under a **contract** (the Schema Registry of §6.3 is the enforcement point). No "shared topics owned by nobody".
- **Schema governance** — every topic's schema is versioned and compatibility-checked (§6.3); breaking changes require a **versioned topic** or a migration plan, never a silent in-place change. The registry is the machine-readable source of the event contract.
- **Topic naming & catalog** — the §10.2 naming convention is enforced (by CI on topic creation where possible), and a **topic catalog** records owner, schema, retention, SLA class and consumers — the operational memory of the platform.
- **The platform team** — a dedicated streaming-platform product team runs brokers (KRaft), Connect, Schema Registry, ksqlDB, and DR (MirrorMaker 2, §6.4); sets the topic-class SLAs (§10.3); and runs the failover drills. This team is the difference between "the event backbone" and "a pile of topics" — and its funding case is [Business Case Development](../management/business_case_development_guide.md)'s subject.

### 9.4 The Regulatory and Audit Angle

For a bank, the event backbone is also a **regulatory instrument**, and that changes a few design decisions:

- **Auditability and lineage** — the log is the audit trail: "what happened, when, in what order" is answerable by replay (§2.3). BCBS 239-style data-lineage asks (cross-ref [Risk Data Aggregation](risk_data_aggregation_guide.md)) are served by the topic catalog (§9.3): every regulatory number traces to the events that produced it.
- **Record-keeping retention** — payment and trade records must be kept for regulator-defined windows (years, jurisdiction-dependent ⚠ — legal confirms); tiered storage (§7.3) makes that affordable; **immutability** (records are never modified in place, §2.3) plus platform controls (ACL-locked audit topics, no deletes for audit classes ⚠/✅ — the controls are platform design, flagged ⚠) support the "unalterable record" expectation.
- **The honest boundary** — Kafka gives you *append-only history with access control*, not a certified records-management system: legal/archive requirements beyond retention (certified storage, legal hold) still need the records-management layer; the backbone feeds it, it does not replace it. And regulators increasingly ask about **operational resilience** (failover, RPO/RTO) — which is exactly what the §10.3 DR design and the [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) drills address.

---

## 10. Worked Example: A Cymbal Bank Event-Platform Design (Event Taxonomy + Topic Design)

This section designs the platform concretely — the taxonomy, the topic layout, the platform components and a banking-flavored topic catalog — as the §9 operating model turned into a blueprint.

### 10.1 The Event Taxonomy

The first design decision: **what kinds of events exist, and what each kind implies for topic policy**. Four categories cover a bank's platform:

| Category | What it records | Banking examples | Topic policy |
|---|---|---|---|
| **Domain events** | Business facts that happened — the "nouns of the business" | `PaymentExecuted`, `TradeBooked`, `LimitBreached`, `ClientOnboarded` | Immutable facts; **retention by regulatory window**; keyed by entity ID |
| **CDC / change events** | Row-level changes in source systems (outbox/Debezium) | `AccountBalanceChanged`, `CustomerMasterUpdated` | The core's changelog; retention per source; often `compact,delete` for master data |
| **Commands** | Intents/requests, not facts | `InitiatePayment`, `AmendTrade`, `CancelOrder` | May be **deduplicated on an idempotency key** (§4.1); shorter retention; request/response often via a reply topic |
| **System / ops events** | Platform telemetry and audit | Consumer-lag alerts, schema registrations, DR switchovers, security events | Short retention (days); platform-team owned; the ops dashboard's fuel |
| **Reference-data snapshots** | The current state of shared master data | Instrument master, FX rates, fee tables | **Compacted** topics (§2.5) — the "table" that every consumer joins against (§5.2) |

The rule of thumb: **facts are retained, commands are deduplicated, reference data is compacted, ops data is short-lived.** Getting a category wrong (e.g. treating a command as a fact, or compacting a fact stream) is the most common design error in banking event platforms.

### 10.2 The Topic Design

- **Topic-per-domain vs topic-per-event-type** — the honest answer is a **hybrid**: high-volume, ordering-sensitive streams get **one topic per (domain, entity, event-type)** (`payments.payment.executed`, `trade.trade.booked`) — clean ordering, clean schema, easy consumer isolation; low-volume heterogeneous events get a **topic per domain** (`customer.events` for onboarding/KYC/limit odds and ends) to avoid topic sprawl. The trade-off: per-event-type topics multiply the catalog but keep ordering and schema evolution simple; per-domain topics are simpler to browse but force consumers to filter and to tolerate heterogeneous schemas.
- **The partition key** — the key is the **ordering contract** (§2.2): `paymentId` for payment events (a payment's lifecycle must be consumed in order), `tradeId` for trade events, `accountId` for account change events, `isin` for instrument reference data. Two rules: (1) the key's **cardinality must be high enough** to spread load across partitions (a topic keyed by `currency` has ~10 hot partitions, not 24); (2) watch for the **hot-key problem** — one gigantic client's `clientId` can skew a partition; the remedy is a composite key (e.g. `clientId:substreamId`) when per-client ordering is not required.
- **Retention vs compaction** — per the taxonomy: fact streams get **retention** (payments: years, per regulatory record-keeping ⚠ — the exact window is jurisdiction/regulator-dependent and legal must confirm; markets: shorter; analytics: days); state/changelog and reference data get **compaction** (with tombstones for deletions); changelogs that must not grow forever get `compact,delete`.
- **The naming convention** — the repo-style convention that scales: **`<domain>.<entity>.<event>`** (`payments.payment.executed`, `trade.trade.booked`, `core.account.changed`, `reference.instrument.v1`) with **versioned topics only for breaking changes** (`payments.payment.executed.v2` — the schema changed incompatibly and old consumers must keep reading the old topic during migration); compatible evolution rides the Schema Registry (§6.3) without a new topic. A topic name is therefore a **contract statement**: domain, entity, and what happened — plus an explicit version when the contract broke.
- **The catalog entry** — every topic gets: owner domain, event category (§10.1), key, partitions, cleanup policy, retention, SLA class (§10.3), schema subject, and the consumer list — enforced by the platform team (§9.3).

### 10.3 The Platform Layout

| Component | Design | Notes |
|---|---|---|
| **Clusters** | Prod (3 AZs), DR (active-passive, warm), dev/test — all **KRaft** | KRaft-only since 4.0 (§3.5); controllers co-located with brokers for the estate's size |
| **Brokers** | RF=3, `min.insync.replicas=2` for tier-1 topics (§3.3), Java 17, page-cache-tuned disks | The durability contract is per-topic-class, not global |
| **Connect** | Distributed workers running Debezium (CDC from the core/payments DBs, outbox consumers) + S3/JDBC/Elasticsearch sinks | The integration spine of §6 |
| **Schema Registry** | Prod + DR instances; BACKWARD compatibility default; CI-checked | The contract enforcement point of §6.3 |
| **Streams apps** | Embedded Streams for enrichment, fraud scoring, position keeping — `exactly_once_v2` for stateful ones (§5.3) | Library, not cluster (§5.1) |
| **ksqlDB** | One self-service cluster for business teams' push/pull queries | The analytics layer of §5.4 |
| **MirrorMaker 2** | Continuous replication prod → DR; offsets mirrored; RPO target in seconds; quarterly failover drills | DR of §6.4, runbook per [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) |
| **Security** | TLS everywhere; SASL/SCRAM or mTLS; ACLs per topic; encryption at rest | Non-negotiable in banking; cross-ref the bank series' security sections |

**Topic SLA classes** — the pricing of the durability contract (§3.3) per class: **tier-1** (payments, positions: `acks=all`, `min.insync=2`, RF=3, years-long retention, exactly-once where stateful) · **tier-2** (markets, regulatory: same durability, shorter retention) · **tier-3** (analytics, ops: `acks=1` acceptable, days of retention, no EOS). The classes are what make the platform *affordable*: strictness is applied where money moves, relaxed where it does not.

### 10.4 The Concrete Banking-Flavored Topics

| Topic | Category | Key | Partitions | Cleanup | Retention | Typical consumers |
|---|---|---|---|---|---|---|
| `payments.payment.executed` | Domain event | `paymentId` | 24 | delete | 7 years (tier-1) ⚠ | Reconciliation, regulatory reporting, fraud scoring, client statements |
| `payments.payment.state` | Changelog | `paymentId` | 24 | compact | — | Status dashboard, support (per-payment current state) |
| `core.account.changed` | CDC | `accountId` | 24 | compact,delete | 90 days | Data lake, limits engine, CRM |
| `trade.trade.booked` | Domain event | `tradeId` | 12 | delete | 7 years (tier-1) ⚠ | Position keeping (exactly-once Streams), risk, settlement, reporting |
| `reference.instrument.v1` | Reference data | `isin` | 6 | compact | — | Every enrichment join in the platform (§5.2) |
| `fraud.alert` | Domain event | `alertId` | 6 | delete | 1 year | Fraud ops console, case management |
| `platform.ops.lag` | Ops event | `groupId` | 3 | delete | 7 days | Platform-team monitoring |

**The end-to-end flow, one paragraph** — a corporate client instructs a USD payment in the payments hub; the hub's service writes the business state **and** the outbox row in one DB transaction (§4.5); **Debezium** emits the outbox row to `payments.payment.commanded`; a Streams app validates and enriches it against `reference.instrument.v1`/customer master (stream-table join), runs it through the fraud scorer, and produces `payments.payment.executed` (exactly-once, keyed by `paymentId` so the lifecycle stays in order); the reconciliation, regulatory-reporting and client-statement consumers each read the executed topic at their own pace; the same topic is mirrored by MirrorMaker 2 to DR in seconds; and if a regulator asks in 2029 what happened to that payment, the platform replays 2026's `payments.payment.executed` from the object tier (§7.3). That is the backbone doing its job: **one durable, replayable, ordered record of what happened, consumed by whoever needs it, whenever they need it.**

### 10.5 The Design Checklist

A platform design is complete when every topic answers these ten questions — the checklist the §9.3 platform team enforces in CI on topic creation:

1. **Owner** — which domain owns this topic? (No owner = no topic.)
2. **Category** — domain event, CDC, command, ops, or reference data (§10.1)? The category dictates the rest.
3. **Key** — what entity orders this stream, and is its cardinality high enough for the partition count (§10.2)?
4. **Partitions** — sized for peak throughput and the consumer parallelism that will actually run (§2.2)?
5. **Cleanup** — delete, compact, or compact,delete — matching facts/changelog/reference-data semantics (§2.5)?
6. **Retention** — regulatory window for facts, bounded for changelogs, short for ops (§10.2)?
7. **Schema** — registered subject, compatibility level set (BACKWARD default), breaking changes versioned (§6.3)?
8. **SLA class** — tier-1/2/3: acks, min.insync, EOS on-or-off (§10.3)?
9. **Consumers** — known and catalogued, each with its own group and lag budget (§9.3)?
10. **DR scope** — mirrored by MirrorMaker 2, included in the failover runbook (§10.3)?

If a topic cannot answer all ten, it is not ready for production.

---

## 11. The One-Page Summary

**Kafka is a distributed, partitioned, replicated commit log** — an append-only, durable, replayable record of events, born at LinkedIn in 2011 (the NetDB'11 paper, open-sourced the same year, Apache top-level in 2012, Confluent founded by its three creators in 2014) and shaped by Jay Kreps's 2013 essay "The Log" into the pattern of *the log as shared infrastructure*. Its architecture is one idea with many consequences: **brokers** (stateful JVM nodes on local disk) store **topics** split into **partitions** — the scaling primitive, with ordering guaranteed per partition via the record **key**; **consumers** pull by **offset** from the log and coordinate in **consumer groups** whose rebalancing has gone from stop-the-world (classic) to incremental-cooperative (KIP-429, 2.4) to fully incremental and server-side (KIP-848, GA in 4.0). **Replication** is the ISR — in-sync replicas — with `acks=all` + `min.insync.replicas` as the durability contract, preferred-leader election, unclean election (default off — keep it off in a bank) and, since **Kafka 4.0 (18 March 2025, verified)**, a **KRaft** Raft-based metadata quorum with ZooKeeper fully removed. The **semantics** are at-least-once by default, at-most-once where loss is tolerable, and **exactly-once within Kafka** via idempotent producers and transactions (0.11, 2017) — with the honest caveat that cross-system exactly-once needs the outbox pattern and idempotency keys, which is why most banks run at-least-once + dedupe. On top of the log sit **Kafka Streams** (the library, not cluster, 2016) with the stream-table duality and exactly-once processing, **ksqlDB** for SQL, **Connect** (with Debezium CDC) as the integration spine, the **Schema Registry** as the contract-enforcement point, and **MirrorMaker 2** for DR; around it, a vast ecosystem of clients and managed offerings (Confluent, MSK, Event Hubs, Google Managed Kafka) plus tiered storage (GA 3.6) that makes years-long retention affordable. The honest comparison: Kafka's pains are operational complexity, JVM footprint, 3× storage cost, tail latency and exactly-once subtlety; its strengths are being the de facto standard with durability, replay, per-partition ordering and the deepest ecosystem — and it is the wrong tool for microsecond hot paths (Aeron/Chronicle) and small deployments (lightweight brokers), the full matrix living in the alternatives guide. For a Cymbal Bank, Kafka is the **event backbone**: the single durable record of payments, trades, positions and regulatory facts, with CDC from core systems, payments and trade-lifecycle event streams, fraud scoring, regulatory replay and a data-mesh operating model of domain-owned topics, schema governance and a platform team — designed concretely as an event taxonomy (facts retained, commands deduplicated, reference data compacted), a topic naming convention (`<domain>.<entity>.<event>`), per-entity partition keys, SLA-classed durability, and DR by MirrorMaker 2. In one sentence: Kafka is the commit log made into infrastructure, and for a bank, that commit log is the event backbone

---

## Glossary

| Term | Definition |
|---|---|
| **Broker** | A Kafka server: a stateful JVM node storing partition log segments on local disk and serving produce/consume traffic for the partitions it leads (§2.1). |
| **Topic** | A named, logical stream of records, the unit of organization and of retention/compaction policy (§2.2). |
| **Partition** | A topic's unit of parallelism: an ordered, immutable sequence of records, spread across brokers and assigned to one consumer at a time (§2.2). |
| **Offset** | The immutable sequence number of a record within its partition; also, the consumer's committed position in the log (§2.3). |
| **Consumer group** | A set of consumers sharing a `group.id` that splits a topic's partitions among its members, each partition processed by exactly one member (§2.4). |
| **Rebalancing** | The group's re-assignment of partitions on membership/partition changes — classic (stop-the-world), KIP-429 incremental cooperative, or KIP-848 next-generation (§2.4). |
| **KIP-429** | Incremental cooperative rebalancing, Kafka 2.4 (2019): members revoke only the partitions that must move (§2.4). |
| **KIP-848** | The next-generation consumer rebalance protocol, GA in Kafka 4.0: fully incremental, server-side, no global synchronization barrier (§2.4). |
| **ISR (In-Sync Replicas)** | The set of follower replicas caught up with the leader; the unit of durability for `acks=all` (§3.1). |
| **min.insync.replicas** | The minimum ISR size for a topic to accept writes; the guard that makes `acks=all` meaningful (§3.1, §3.3). |
| **High watermark** | The offset up to which all ISR replicas have replicated; consumers never read past it (§3.1). |
| **Leader / follower** | Per partition: the replica serving all produce/consume traffic (leader) and the replicas replicating from it (followers) (§3.1). |
| **Preferred leader** | The partition's original/first replica; leadership returns to it via preferred-replica election (§3.2). |
| **Unclean leader election** | Electing a leader from outside the ISR — availability at the cost of possible data loss; off by default (§3.2). |
| **Replica fetcher** | The mechanism by which followers pull batches from the leader to replicate (§3.1). |
| **KRaft** | Kafka's Raft-based metadata quorum, replacing ZooKeeper; the only mode since Kafka 4.0 (§3.4). |
| **ZooKeeper** | The external coordination service Kafka used before KRaft for metadata and controller election; fully removed in 4.0 (§3.5). |
| **Controller quorum** | The KRaft nodes that elect a controller leader via Raft and hold the metadata log (§3.4). |
| **Log segment** | A rolling, immutable file holding a slice of a partition's log; the unit of retention deletion and tiered-storage offloading (§2.5). |
| **Retention** | The time/size policy by which segments are deleted; deletion is retention-based, not consumption-based (§2.5). |
| **Log compaction** | Keeping only the latest record per key (plus tombstones) in a keyed topic — turning the log into a changelog/table (§2.5). |
| **Tombstone** | A record with a null value that marks a key as deleted under compaction (§2.5). |
| **acks** | The producer durability setting: 0 (fire-and-forget), 1 (leader write), all (all ISR) (§2.6, §3.3). |
| **Idempotent producer** | A producer with ID + per-partition sequence numbers so the broker drops duplicate retries; no duplicates within a session (§2.6, §4.3). |
| **Transaction** | An atomic set of records across partitions/topics, made visible atomically to `read_committed` consumers (§4.3). |
| **transactional.id** | The producer's stable identity across restarts; the basis of epoch-based zombie fencing (§4.4). |
| **Zombie fencing** | Rejecting writes from a stale producer instance whose epoch is no longer current (§4.4). |
| **read_committed / read_uncommitted** | Consumer isolation levels: committed transactions only, versus all records including aborted ones (§4.4). |
| **Exactly-once (EOS)** | The read-process-write cycle within Kafka commits atomically — output records and input offsets together (§4.3). |
| **At-least-once / at-most-once** | Delivery guarantees: no loss with possible duplicates (the default) / no duplicates with possible loss (§4.1–4.2). |
| **Outbox pattern** | Writing business state and its event in one DB transaction, then relaying the event row to Kafka (often via CDC) (§4.5). |
| **Event sourcing** | Storing the state of a system as its sequence of events; the log is the source of truth (§1.3). |
| **CQRS** | Command-Query Responsibility Segregation: separate write model from read models, often rebuilt by replay (§1.3, §5.2). |
| **Kafka Streams** | The Java/Scala stream-processing library embedded in your application — no separate cluster (§5.1). |
| **KStream / KTable** | The stream view (every record) and the table view (current state per key) of the stream-table duality (§5.2). |
| **Topology** | A Streams application's processing graph: sources → operators → sinks (§5.1). |
| **State store** | A Streams application's local state (RocksDB-backed by default), recoverable from its changelog topic (§5.1). |
| **Changelog topic** | A compacted topic backing a state store or KTable, enabling recovery by replay (§5.2). |
| **ksqlDB** | The streaming SQL engine built on Kafka Streams (formerly KSQL) — SQL over streams (§5.4). |
| **Kafka Connect** | The framework for streaming data in/out of Kafka via source/sink connectors and tasks on workers (§6.1). |
| **Debezium** | The open-source CDC connector family — row changes from databases into Kafka (§6.2). |
| **CDC (Change Data Capture)** | Capturing database row changes from the transaction log and emitting them as events (§6.2). |
| **Schema Registry** | The versioned schema store (Avro/JSON/Protobuf) that makes schema evolution safe and compatible (§6.3). |
| **Compatibility level** | The Schema Registry rule for whether a new schema may replace an old one (BACKWARD/FORWARD/FULL/NONE, plus transitive variants) (§6.3). |
| **MirrorMaker 2** | The cross-cluster replication tool (Connect-based) for DR, migration and aggregation (§6.4). |
| **Tiered storage** | Offloading sealed log segments to object storage — long retention at object-store prices; GA in Kafka 3.6 (§7.3). |
| **Watermark** | In stream processing, the timestamp up to which event-time data is considered complete (the trigger for windowed results) — distinct from the replication high watermark (glossary above). |

---

## Verification Ledger

**✅ Verified this pass at primary sources (live web access — kafka.apache.org, the Apache wiki, Apache release notes):**

- **Kafka 4.0.0 released 18 March 2025** — the Apache Kafka 4.0.0 release announcement (kafka.apache.org, David Jacot, "Tuesday, March 18, 2025").
- **4.0 is KRaft-only; ZooKeeper entirely removed** — "the first major release to operate entirely without Apache ZooKeeper", "KRaft mode by default" (same announcement).
- **KIP-848 GA in 4.0** — "goodbye to stop-the-world rebalances"; enabled by default server-side; consumers opt in with `group.protocol=consumer` (announcement; corroborated by kafka.apache.org's consumer-rebalance-protocol operations page).
- **KIP-932 Queues for Kafka (early access, share groups); KIP-890 transactions server-side defense phase 2 (reduces zombie transactions); KIP-966 Eligible Leader Replicas (preview, prevents data loss in elections); KIP-996 Pre-Vote** (all in the 4.0.0 announcement).
- **Java baselines in 4.0** — brokers/Connect/tools require Java 17; clients and Streams require Java 11 (announcement).
- **The 2011 paper** — "Kafka: a Distributed Messaging System for Log Processing", Kreps/Narkhede/Rao (LinkedIn), NetDB workshop 2011; the paper's scale figures (~20B events/day, ~3 TB/day) (Apache Kafka wiki papers-and-presentations page + paper PDF).
- **Kafka Streams landing in 0.10.0** — the Apache Kafka 0.10.0.0 release notes show the KStream DSL and state-store work (KAFKA-2653, KAFKA-2706, etc.) — corroborates the repo-verified 2016 introduction.

**✅ Repo-verified (held from the sibling guides' ledgers, cross-referenced, not re-verified this pass):** Kafka created at LinkedIn by Kreps/Narkhede/Rao · open-sourced 2011 · Apache top-level project 2012 · Confluent founded 2014 by the three creators · Jay Kreps's "The Log" essay (2013) · Kafka Streams (2016) · exactly-once via idempotent producers + transactions in 0.11 (2017) · KIP-429 incremental cooperative rebalancing in 2.4 (2019) · KRaft early access in 3.3 (October 2022) · tiered storage GA in 3.6 (2023) · ZooKeeper removed in 4.0 (March 2025).

**⚠ Flagged (could not be re-verified this pass — search backend degraded mid-pass, or knowledge-base-held):**

- ksqlDB naming history (KSQL open-sourced 2017; renamed ksqlDB 2019/2020) and its server-cluster operational shape.
- Schema Registry compatibility-level list (BACKWARD/FORWARD/FULL/NONE + transitive variants) and the BACKWARD default.
- KIP-354 (log-compaction improvement KIP number); log compaction itself is core documented behavior ✅.
- Documented defaults not re-verified: `segment.bytes` 1 GiB, `retention.ms` 7 days, `replica.lag.time.max.ms` 30 s, `acks=1` as the historical producer default, idempotence-default-since version, sticky partitioning for null keys.
- ZooKeeper-mode deprecation release (commonly cited as 3.5) and the exact 3.x bridge-release set for the KRaft migration path.
- `exactly_once_v2` introduction release (KIP-890 generation, 3.x era).
- Managed-offering launch dates (Amazon MSK 2018; Azure Event Hubs Kafka endpoint 2018; Google Managed Kafka announced 2024) and Confluent's 2021 IPO.
- Debezium's exact supported-database list; RocksDB as the Streams default state-store engine; librdkafka-based bindings' current maintenance status; licensing split of Confluent's non-Apache components.
- Retention-window figures for regulatory record-keeping (jurisdiction-dependent — legal must confirm).
- The 2011 LinkedIn engineering blog post's exact publication date; the exact months of the 2011 open-source release and 2012 top-level-project status.

**Integrity statement:** no quotes, dates or version numbers in this guide are fabricated; every ✅ above was checked against the cited primary source this pass or is held from the repo's sibling ledgers; every ⚠ is explicitly flagged inline and listed here. Where the brief demanded verification and the tools went down, the guide says so rather than asserting.

---

## Cross-References Recap

| Guide | Where used |
|---|---|
| [Kafka Alternatives](../technology/kafka_alternatives_guide.md) | §8 (the head-to-head — compact table + pointer), §7.2, §9.1 |
| [Event Stream Processing](../technology/event_stream_processing_guide.md) | §4.1, §4.5, §5 (the discipline) |
| [Complex Event Processing](../technology/complex_event_processing_guide.md) | header companions (the adjacent discipline) |
| [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) | §3.6 (KRaft migration), §6.4/§10.3 (DR runbooks) |
| [Low Latency C++ Development](../technology/low_latency_cpp_development_guide.md) | §8.3 (trading hot paths — Aeron/Chronicle) |
| [Business Case Development](../management/business_case_development_guide.md) | §7.4, §9.1, §9.3 (funding the platform) |
| [Crédit Agricole Software Systems](credit_agricole_software_systems_guide.md) | §9.1 (the Cymbal Bank estate shape) |
| [NETS Software Systems](nets_software_systems_guide.md) | §9.2 (the data-mesh pattern) |
| [Treasury & ALM](treasury_alm_guide.md) | §9.1, §9.2 (markets/trade lifecycle) |
| [Payments Hub](payments_hub_guide.md), [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) | §4.1, §9.2, §10.4 (payments event streams) |
| [Regtech](regtech_guide.md), [Risk Data Aggregation](risk_data_aggregation_guide.md) | §9.2 (regulatory reporting, lineage) |
| [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) | §9.2 (fraud detection) |
| Bank series ([DBS Bank](dbs_bank_guide.md), [UOB Software Systems](uob_software_systems_guide.md), [OCBC Software Systems](ocbc_software_systems_guide.md), [HSBC Software Systems](hsbc_software_systems_guide.md), [Bank of America Software Systems](bank_of_america_software_systems_guide.md)) | §9.2 (CDC-in-banking patterns) |
| [Capital Markets Architecture](capital_markets_architecture_guide.md) | §9.1, §9.2 (trade lifecycle) |

---

## References and Further Reading

- **Apache Kafka 4.0.0 Release Announcement** — kafka.apache.org, David Jacot, 18 March 2025 (KRaft-only, KIP-848, KIP-932, KIP-890, KIP-966, KIP-996, Java baselines) — the primary source for §3.4–§3.6.
- **"Kafka: a Distributed Messaging System for Log Processing"** — J. Kreps, N. Narkhede, J. Rao, NetDB'11 workshop (co-located with SIGMOD'11), 2011 — the origin paper (§1.2).
- **Apache Kafka wiki — Kafka papers and presentations** — cwiki.apache.org (NetDB'11 listing, §1.2).
- **Apache Kafka 0.10.0.0 Release Notes** — archive.apache.org (the KStreams DSL/state-store landing, §5.1).
- **Apache Kafka documentation** — kafka.apache.org/documentation (architecture, KRaft, upgrade paths, the consumer-rebalance protocol).
- **KIP-848: The Next Generation of the Consumer Rebalance Protocol** — cwiki.apache.org (the KIP, §2.4).
- **Jay Kreps, "The Log: What every software engineer should know about real-time data's unifying abstraction"** — 2013 (§1.3).
- **Confluent documentation** — docs.confluent.io (Kafka Streams, ksqlDB, Schema Registry, tiered storage) ⚠ (not re-fetched this pass — cited from the author's knowledge base).
- **The sibling deep-dives** — [Kafka Alternatives](../technology/kafka_alternatives_guide.md) and [Event Stream Processing](../technology/event_stream_processing_guide.md) (§8, §4–§5).
