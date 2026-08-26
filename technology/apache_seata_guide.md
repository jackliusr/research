# Apache Seata: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Distributed Systems / Data Engineering — Apache Seata (incubating), the open-source distributed transaction framework: the distributed transaction problem, Seata architecture, the AT / TCC / SAGA / XA transaction modes, ecosystem, comparisons, and practical adoption (Distributed Systems series)  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [The Distributed Transaction Problem](#1-the-distributed-transaction-problem)
2. [Apache Seata Overview](#2-apache-seata-overview)
3. [Seata Architecture](#3-seata-architecture)
4. [Transaction Modes I: AT Mode](#4-transaction-modes-i-at-mode)
5. [Transaction Modes II: TCC Mode](#5-transaction-modes-ii-tcc-mode)
6. [Transaction Modes III: SAGA Mode](#6-transaction-modes-iii-saga-mode)
7. [Transaction Modes IV: XA Mode](#7-transaction-modes-iv-xa-mode)
8. [Mode Comparison and Selection](#8-mode-comparison-and-selection)
9. [Ecosystem and Integrations](#9-ecosystem-and-integrations)
10. [Adoption and Best Practices](#10-adoption-and-best-practices)
11. [Comparison with Alternatives](#11-comparison-with-alternatives)
12. [Worked Example: Order–Inventory–Account](#12-worked-example-orderinventoryaccount)
13. [The Future: 2026 and Beyond](#13-the-future-2026-and-beyond)
14. [Glossary](#14-glossary)

---

## 1. The Distributed Transaction Problem

### 1.1 What a Distributed Transaction Is

A **distributed transaction** is a transaction whose participants span more than one independently managed resource — multiple databases, multiple microservices, or a mix of databases, message queues, caches, and third-party APIs. The defining property is that **no single local transaction manager can see all the participants**, so atomicity ("all or nothing"), consistency, isolation, and durability — the **ACID** properties that a single database engine provides natively — must be *rebuilt* by application- or middleware-level coordination.

This guide assumes the reader already knows ACID at the single-database level (see [oracle_database_guide.md](oracle_database_guide.md) and [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) for the database-side foundations) and how transactions behave *within* one engine. The problem addressed here is the one that starts at the *boundary*: the moment an atomic unit of work touches two databases or two services.

A local transaction is atomic because one engine owns the log, the locks, and the recovery. A distributed transaction has no such owner. It is the difference between:

```
Local transaction:                  Distributed transaction:
  BEGIN;                              Service A: BEGIN; UPDATE orders ...; COMMIT;
  UPDATE orders ...;                  Service B: BEGIN; UPDATE inventory ...; COMMIT;
  UPDATE inventory ...;               Service C: BEGIN; UPDATE accounts ...; COMMIT;
  COMMIT;                             ??? who guarantees all three commit together ???
```

In a microservices architecture (see [oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) for the banking context), the *database-per-service* pattern makes this the default situation rather than the exception: each service owns its schema, so a single business action — place an order, transfer money, book a seat — fans out into several local transactions that must behave as one.

### 1.2 ACID in the Distributed World

Mapping ACID onto a distributed setting immediately reveals what is lost:

- **Atomicity.** With N independent databases, "all or nothing" requires a *global* decision protocol. If service A commits and service B fails, somebody must undo A's commit — a **compensation** — because a database cannot un-commit a committed transaction on command. This is the fundamental break: local rollback becomes *global compensation*.
- **Consistency.** Each database enforces its own constraints, but cross-database invariants (e.g., "sum of account balances is conserved" across two banks' ledgers) are invisible to any single engine. Consistency moves from the database to the *coordination layer*.
- **Isolation.** A local database serializes concurrent transactions touching the same rows. Across databases, no shared lock manager exists — a transaction in service A can commit rows that service B's in-flight transaction has not seen, producing the distributed equivalent of dirty reads, non-repeatable reads, and write skew.
- **Durability.** Durability survives per-participant, but the *global record* of "did this distributed transaction happen?" must live somewhere recoverable — typically in a **transaction coordinator's log** — or a crash between participants leaves the system in an indeterminate state.

The honest summary: **distributed transactions trade the guarantees a single engine gives you for guarantees you must construct.** Every framework in this space — Seata included — is an engineering answer to that reconstruction problem.

### 1.3 CAP and the Distributed Transaction Trade-Off

The **CAP theorem** states that a distributed data system can guarantee at most two of: **Consistency** (every read returns the latest write), **Availability** (every request receives a response), and **Partition tolerance** (the system continues operating despite network partitions). Since partitions are a physical fact of networks, real systems choose between **CP** (consistency-first, refuse responses during partitions) and **AP** (availability-first, accept staleness). See [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) for the full CAP treatment.

Distributed transactions sit at the CP end of this spectrum by construction:

- A global transaction that holds locks across participants until a global decision is made is *choosing consistency over availability*: during the decision window, some participants are blocked (unavailable) to preserve correctness.
- The **trade-off axis is usually latency/blocking vs. consistency**: strict 2PC gives strong consistency but blocks; sagas give availability and liveness but only eventual consistency, with a *window* in which intermediate states are visible.

This is why the distributed transaction debate is never purely technical — it is a **business-risk decision** about how long the system may show inconsistent intermediate state, and what the cost of a wrong-but-eventually-corrected outcome is. Money movement and inventory are domains where that cost is high; content moderation, recommendations, and analytics are domains where it is low.

### 1.4 Two-Phase Commit (2PC)

**Two-phase commit** is the classic protocol for atomic commitment across multiple resources, and it is the intellectual baseline for everything in this guide. A **coordinator** (also called the transaction manager or "commit arbiter") drives two phases over all participants:

```
Phase 1 — PREPARE:
  Coordinator ── prepare(Tx) ──▶ Participant A ──▶ write UNDO log, lock rows, vote
  Coordinator ── prepare(Tx) ──▶ Participant B ──▶ write UNDO log, lock rows, vote
  Participants reply: YES (ready to commit) or NO (abort)

Phase 2 — COMMIT / ABORT:
  Coordinator ── commit(Tx) ──▶ Participant A ──▶ release locks, commit locally
  Coordinator ── commit(Tx) ──▶ Participant B ──▶ release locks, commit locally
  (if ANY participant voted NO, coordinator sends abort(Tx) everywhere)
```

2PC is covered in depth in the cross-shard transaction sections of [oracle_sharding_guide.md](oracle_sharding_guide.md) — Oracle's native XA / grid infrastructure is a production-grade 2PC implementation. The protocol's properties are well understood:

- **Guaranteed atomicity** — provided the coordinator and participants all obey the protocol and the coordinator survives long enough to deliver the final decision.
- **Blocking at prepare.** After a participant votes YES it holds locks and its resources are *in doubt*: it cannot unilaterally decide. It waits for the coordinator.
- **Coordinator failure = blocked resources.** If the coordinator crashes after phase 1, participants holding locks cannot proceed until the coordinator (or a recovery log) returns. This is the famous "2PC blocking problem" — solved in practice by a durable coordinator log plus recovery, but never eliminated.
- **Performance cost.** Two network round-trips per participant, locks held across both phases, and a single point of coordination serialization. Latency scales poorly.

These three limitations — blocking, coordinator-failure exposure, and performance — are precisely the problems the *alternative* distributed transaction patterns, and Seata's four modes, were designed to address.

### 1.5 The Distributed Transaction Pattern Landscape

The industry has converged on a small family of patterns. Each trades something (blocking, code burden, consistency strength) for something else:

| Pattern | Core idea | Consistency | Blocking | Business code | Typical use |
|---|---|---|---|---|---|
| **2PC / XA** | Coordinator + prepare/commit on all resources | Strong | Yes (prepared resources locked) | None (middleware handles it) | Short, high-value transactions over XA-capable DBs |
| **TCC** | Business-level Try-Confirm-Cancel on each participant | Strong-ish (via business reservation) | Short (reservation window) | **Heavy** (3 methods per participant) | Heterogeneous resources, non-DB participants |
| **SAGA** | Sequential local transactions + compensation chain | Eventual (intermediate states visible) | No (locks released per step) | Medium (compensation logic) | Long-running business processes, orchestration |
| **Outbox** | Write event + business data in one local txn; relay publishes | Eventual | No | Low (relay process) | Event-driven decoupling; see [event_stream_processing_guide.md](event_stream_processing_guide.md) |
| **Event-driven / compensation** | Async messages, local transactions, compensating events on failure | Eventual | No | Medium | The microservices default; see the saga and outbox sections of [event_stream_processing_guide.md](event_stream_processing_guide.md) |

Key relationships to keep in mind:

- **TCC is 2PC with the business as the resource manager.** Instead of the database preparing, the *application* implements Try (reserve), Confirm (commit), Cancel (compensate). It buys non-blocking-ness for heterogeneous resources at the price of hand-written idempotent business code.
- **SAGA is the availability-first answer.** No global locks; each step commits and releases its resources; on failure, earlier steps are compensated. Strong liveness, eventual consistency.
- **The outbox pattern** (transactional outbox) is not a transaction protocol at all — it guarantees *one* local atomicity (DB row + event row in the same local transaction) and relies on a relay to publish events, which downstream services consume and compensate as needed. It is the pragmatic default for most event-driven systems and is covered in detail in [event_stream_processing_guide.md](event_stream_processing_guide.md) and the outbox-focused sections of [kafka_alternatives_guide.md](kafka_alternatives_guide.md).

### 1.6 The Microservices Orthodoxy — and the Reality

**The orthodoxy.** Mainstream microservices guidance — from Fowler's saga article through most platform teams' architecture principles — is blunt: *don't use distributed transactions in microservices; use sagas (or events) instead.* The reasoning is sound for the common case:

- Distributed transactions require a **global coordinator** — a new single point of failure and a coupling point between services.
- They **hold locks across service boundaries and network calls**, which is how a small order service takes down an entire platform under load.
- They **assume all participants are reachable and XA-capable**, which is false for caches, search indexes, third-party APIs, and NoSQL stores.
- They fight the database-per-service ownership model — services lose control of their own transactions.

So the received wisdom: keep local transactions local, publish events, and let each service react; when a multi-service flow needs a business-level rollback, choreograph or orchestrate a saga. This is exactly the philosophy of [event_stream_processing_guide.md](event_stream_processing_guide.md) and the microservices transaction patterns in [oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md).

**The reality.** The orthodoxy is right for *most* flows but not *all*. There are domains where the business cannot tolerate eventual consistency — where an interim state like "order created but inventory not deducted" or "account debited but not credited" is a *real, costly, audit-visible event*:

- **Money movement.** Transfers, settlements, and ledger postings where intermediate states are regulated, reported, or irreconcilable.
- **Inventory and ticketing.** Overselling a flight or a product because two services decremented the same stock independently, then discovering it after the customer is committed.
- **Banking and fintech.** The "transfer" use case that this guide's worked example (§12) is built around — the domain of the reader's employer — where regulators expect strong consistency and precise audit trails.

For these flows, teams face three options: (a) collapse the flow into **one database** (or one logical database, e.g., Oracle sharding with XA across shards — see [oracle_sharding_guide.md](oracle_sharding_guide.md)); (b) implement **business-level compensation** carefully (TCC/saga with rigorous idempotency); or (c) accept a **middleware-managed distributed transaction** such as Seata. The rest of this guide is about option (c): what Seata is, how it works, and — honestly — when it is and is not the right answer.

---

## 2. Apache Seata Overview

### 2.1 What Seata Is

**Apache Seata** (incubating) is an open-source distributed transaction solution for microservices architectures, licensed under Apache-2.0. Its own description: *"an open source distributed transaction solution that delivers high performance and easy to use distributed transaction services under a microservices architecture."* It is the de-facto standard distributed transaction framework in the Java ecosystem, and it explicitly positions itself as covering the whole consistency spectrum — strong and eventual — through **four transaction modes**: **AT, TCC, SAGA, and XA** (§4–§7).

The name is an acronym: **S**imple **E**xtensible **A**utonomous **T**ransaction **A**rchitecture. The "autonomous" is deliberate — the design philosophy is that each participant (branch) manages its own local transaction autonomously, and the framework coordinates the *global* outcome, so services do not need to hand their transaction management over to a heavyweight external engine. "Extensible" reflects the pluggable mode/registry/config/store design that has let the project grow from an Alibaba internal tool into an Apache project.

### 2.2 History: From Alibaba's Fescar to Apache Seata

The lineage is one of the clearest examples of "open-sourced production middleware" in the industry — Seata was not built to be open source; it was battle-tested inside Alibaba and Ant Group first.

```
2014–2018  Alibaba runs internal distributed transaction middleware TXC
           ("Taobao Transaction Constructor" lineage) across its e-commerce
           systems; Ant Group runs its own GTS (Global Transaction Service)
           for financial workloads.

Jan 2019   Alibaba open-sources Fescar (Fast & EaSy Commit And Rollback),
           built on the TXC/GTS experience. GitHub: github.com/fescar-group/fescar.

Apr 2019   Ant Group's Data Middleware Team joins the community. To make the
           project neutral, Fescar is renamed Seata (Simple Extensible
           Autonomous Transaction Architecture); the repo moves from the
           Alibaba org to an independent Seata org.

2019–2023  Seata grows rapidly in the Chinese internet ecosystem (Spring
           Cloud Alibaba integration, Dubbo support, mode expansions),
           becomes the default answer to "distributed transaction" in Java.

Oct 2023   Seata enters the Apache Incubator as apache/incubator-seata.

2023–2026  Incubation continues: 2.x release line, seata-go, MCP-enabled
           console, raft clustering in beta. Still "incubating" at the time
           of writing (August 2026) — graduation to a top-level Apache
           project had not yet been announced (verified against
           seata.apache.org, Aug 2026).
```

Two naming facts worth pinning down precisely, since they are frequently garbled:

- **Fescar** = **F**ast **&** **E**a**S**y **C**ommit **A**nd **R**ollback (the stylized capitalization "Fast & EaSy" is intentional in the original branding).
- **Seata** = **S**imple **E**xtensible **A**utonomous **T**ransaction **A**rchitecture.

**Apache Incubator status (verified, August 2026):** Seata entered the Apache Incubator in **October 2023**. The project is still **incubating** — the official site styles itself "Apache Seata™ (incubating)" and the repository lives at `apache/incubator-seata`. Any claim of a specific graduation date would be false; as of this writing there is none to report. This is worth tracking: graduation is a governance/quality milestone, not a feature milestone — the software has been in production use for years regardless of its ASF status.

### 2.3 Current Status and Version Line

Verified as of August 2026:

- **Latest release: 2.6.0** — the current download-page headline ("2.6.0 latest version with a major upgrade"). Notable 2.6.0 changes: **MCP server support for the Seata Console**, netty protocol optimization with simplified configuration, YAML compatibility fixes, and **branch transaction consolidation for multi-data-source** scenarios.
- **2.5.0** was released **2025-08-05** (per the project site's release notes).
- Version lines **2.0–2.3 and 1.8** have reached end-of-life (no more security updates), so production users should be on **2.4.x / 2.5.x / 2.6.x**. The earlier 1.x line ended at 1.8.
- **Community size (approximate, August 2026):** ~**25.9k stars** and ~8.9k forks on GitHub (the count moves daily — treat "~26k stars" as the right order of magnitude; the task brief guessed ~30k, which is now too high).
- **Security:** CVE-2024-22399 — a critical deserialization vulnerability affecting Seata up to 1.8.0 / 2.0.0. Fixed in later 1.8.x/2.0.x+ patches; this is the single most important reason to stay off EOL versions (see §10.6).

### 2.4 Positioning: Seata vs. the Alternatives at a Glance

Seata occupies a specific niche, and understanding the niche explains the project's design:

- **vs. saga frameworks (Axon, Eventuate Tram):** Axon and Eventuate Tram are *event-sourcing / CQRS / saga* frameworks — they assume eventual consistency and give you the infrastructure for events, sagas, and compensation. Seata's AT and XA modes additionally provide *strong* consistency with near-zero business code, which pure saga frameworks cannot. Seata's SAGA mode is its answer on the saga side of the spectrum — so Seata is deliberately *wider*: one framework, four consistency levels.
- **vs. classic 2PC middleware (Atomikos, Narayana):** Atomikos and Narayana are JTA transaction managers for classic 2PC/XA over JDBC/JMS resources. They are mature, but they are *local-process* coordinators — the app server embeds them, and distributed scenarios need XA-capable resources throughout. Seata's AT mode gives strong consistency *without* requiring the resources to support XA (it fakes the "prepare" with undo logs and global locks) and coordinates over the network via a dedicated server — a different deployment model built for microservices.
- **vs. the outbox pattern:** The outbox (see [event_stream_processing_guide.md](event_stream_processing_guide.md)) is not a competing framework but a competing *strategy*: it guarantees one local atomic write and accepts eventual consistency downstream. For many flows outbox + events is the better engineering choice (cheaper, no coordinator). Seata is the answer when the flow needs *stronger* guarantees than outbox gives — §10.7 covers when that trade is worth it.
- **vs. cloud offerings:** AWS has **no managed RDBMS 2PC / distributed transaction middleware** — the frequently-cited "AWS Mithra" is a *malicious-domain detection* system, not a transaction service (verified; see §11.2). Cloud-native alternatives are Step Functions (saga orchestration) or platform-native transactions (DynamoDB transactions, Spanner). Seata's differentiator is being *database-agnostic and self-hosted* — which is precisely what a bank's private cloud / on-prem estate needs.

The bottom line on positioning: **Seata is the strongest mainstream answer for "strong consistency across independently deployed Java services and heterogeneous databases, without writing compensation business code."** Its AT mode is the headline feature; everything else (TCC, SAGA, XA) extends the same TC/TM/RM architecture (§3) to cover the rest of the consistency spectrum.

---

## 3. Seata Architecture

### 3.1 The Three Roles: TC, TM, RM

Seata's architecture decomposes every distributed transaction into three roles — the same roles appear in all four modes, which is the key to the framework's coherence:

| Role | Full name | Where it lives | Responsibility |
|---|---|---|---|
| **TC** | **Transaction Coordinator** | The **Seata Server** (standalone service) | Maintains the state of global and branch transactions; drives global commit or rollback; owns the global lock table and the transaction log |
| **TM** | **Transaction Manager** | In the **application** (the service that starts the global transaction) | Defines the global transaction boundary: `begin`, `commit`, `rollback`; propagates the **XID** |
| **RM** | **Resource Manager** | In the **application** (each service touching a resource) | Manages *branch* transactions: registers branches with the TC, reports branch status, executes the branch's commit/rollback work (e.g., undo-log apply in AT mode, confirm/cancel in TCC) |

The TM and RM run **inside the client application** (as interceptors on top of your data access layer — see §3.6); only the TC is a separate process. This is a deliberate contrast with classic JTA/XA, where the transaction manager also lives in the application process: Seata moves the *global* brain out into a dedicated, clusterable server, which is what makes the AT mode's non-invasive coordination over plain JDBC possible.

**XID.** Each global transaction is identified by a globally unique **XID** — typically `<TC-address>:<transaction-id>` (e.g., `192.168.1.10:8091:9091462348341965889`). The XID is the token that links all branches to their global transaction; it must travel with every downstream call (§3.3).

### 3.2 Architecture Diagram and Interaction Flow

```
                     ┌──────────────────────────────────────┐
                     │            SEATA SERVER (TC)         │
                     │  Transaction Coordinator             │
                     │  ┌────────────┐  ┌────────────────┐  │
                     │  │ Global txn │  │ Global lock    │  │
                     │  │ log/store  │  │ table (AT)     │  │
                     │  └────────────┘  └────────────────┘  │
                     └───────▲───────────────▲──────────▲───┘
                             │ register /     │ begin /   │ branch
                             │ report         │ commit    │ commit/
                             │ branches       │ /rollback │ rollback
                 ┌───────────┴───────┐   ┌────┴────┐   ┌───┴───────────┐
                 │  Service A (TM+RM)│   │Service B│   │  Service C    │
                 │  @GlobalTransactional (RM)    │   │  (RM)          │
                 └───────────────────┘   └─────────┘   └───────────────┘
                          │  XID propagates with every downstream call
                          ▼
                 ┌──────────────────────────────────────────┐
                 │   Database A / B / C  (each with undo_log │
                 │   table in AT mode; XA-capable in XA mode)│
                 └──────────────────────────────────────────┘
```

The canonical interaction sequence (AT mode; the other modes differ only in what a "branch commit/rollback" executes):

```
 1. TM calls TC:    begin global transaction          → TC returns XID
 2. TM stores XID in the current thread's context
 3. Service A executes business SQL (JDBC)
 4. RM (A) calls TC: register branch (XID, resource, lock keys)
    → TC records the branch and (AT mode) acquires global locks
 5. Service A commits its LOCAL transaction (branch done locally)
 6. ... downstream calls propagate the XID; services B, C do steps 3–5 ...
 7. TM calls TC:    global commit / global rollback
 8. TC drives phase 2 for every branch:
      - commit  → tell each RM to do its branch-commit work
                  (AT: delete undo logs asynchronously)
      - rollback → tell each RM to do its branch-rollback work
                  (AT: apply reverse SQL from the undo log)
```

### 3.3 The Global Transaction Lifecycle and XID Propagation

A global transaction moves through five states managed by the TC:

```
Begin ──▶ BranchRegister ──▶ (branch local txns run) ──▶ BranchReport
                                                          │
                          ┌───────────────────────────────┤
                          ▼                               ▼
                   Global Commit                    Global Rollback
                   (branch commit phase 2)          (branch rollback phase 2)
                          │                               │
                          ▼                               ▼
                     Finished                          Finished
```

**XID propagation** is the plumbing that makes this work across services. Because TM and RM are interceptors, the framework injects the XID into every outbound call and reads it on every inbound call, transparently:

- **Spring Cloud / HTTP:** the Feign/RestTemplate interceptor adds the XID as an HTTP header (default header name `TX_XID`); the server-side interceptor restores it into the thread context.
- **Dubbo:** the filter attaches the XID to the RPC invocation's attachments (RpcContext), which is why Seata integrates with Dubbo out of the box (Dubbo 2.x and Dubbo 3 both supported in current versions).
- **gRPC:** the gRPC interceptor propagates the XID via gRPC metadata.
- **Within a thread:** the XID lives in a `ThreadLocal`-style context; the framework manages its scope around the `@GlobalTransactional` boundary and across async/thread-pool handoffs (the `io.seata.core.context.GlobalLockConfigHolder` / `RootContext` machinery).

If the XID fails to propagate (custom HTTP clients, manual threads, non-integrated frameworks), the downstream service simply runs its branch **outside** the global transaction — a silent, dangerous failure mode that §10.6 flags as a top integration pitfall.

### 3.4 Branch Transactions

A **branch transaction** is one participant's local unit of work inside the global transaction: the local SQL plus its undo/compensation capability, identified in the TC by a **branch ID** scoped under the global XID. The RM registers the branch, reports its status (registered / phase-1 done / phase-2 done / failed), and — critically — the *local* transaction commits or rolls back *independently* at the end of its own method. The global transaction's atomicity is then achieved by the TC's phase-2 instructions: in AT mode by applying the undo log, in TCC by invoking the business Confirm/Cancel, in SAGA by stepping the state machine's compensation, in XA by driving the XA prepare/commit on the resource.

This "branch commits locally, global outcome decided later" design is what gives Seata its non-blocking character in the compensation modes (AT/TCC/SAGA): resources are not held open across the whole global transaction the way classic 2PC holds them across prepare.

### 3.5 The Seata Server (TC): Deployment and High Availability

The Seata Server is a standalone Java process (the `seata-server` distribution, port 8091 by default). Deployment considerations:

- **Standalone:** one server, `file` or `db` storage — fine for development, demos (§12), and low-criticality environments.
- **Cluster / HA:** the standard production pattern is *N servers behind a load balancer (or VIP)*, all registered in a **registry** (Nacos, Eureka, Consul, etcd, ZooKeeper, or Redis — Nacos is the de-facto default in the Alibaba ecosystem), with clients discovering them. For a cluster, the transaction store must be shared:
  - **`db` storage** — global/branch/lock records in a shared database (the classic HA setup; MySQL/PostgreSQL/Oracle etc.),
  - **`redis` storage** — Redis-backed store for higher throughput,
  - **Raft cluster mode** — a built-in Raft-based HA mode (consensus without an external store) that the project describes as **in beta**; the 2.x release notes advertise "cluster mode with separate storage and computing, where computing nodes can be horizontally scaled, and storage supports both databases and Redis."
- **Operations surface:** the server exposes JMX-style management, a **Seata Console** (web dashboard; 2.6.0 adds an **MCP server** interface to the console — a notable 2026-era development, see §13), and metrics endpoints for Prometheus scraping (§10.5).

A rule of thumb: the TC is a *stateful, latency-sensitive* coordinator — run it close to your services (same region/network), give it a durable store, and treat it with the same operational care as a database, because it *is* effectively the transaction log of your distributed state.

### 3.6 The Seata Client: Framework Integration

The client side is a set of interceptors and data-source proxies shipped as libraries. The main integration surfaces (see §9 for the full ecosystem map):

- **Spring Boot:** `io.seata:seata-spring-boot-starter` (or `org.apache.seata:seata-spring-boot-starter` since the 2.x Apache groupId) auto-configures the TM/RM, the `@GlobalTransactional` interceptor, the XID propagation filters, and the proxied data sources. `@GlobalTransactional` on a method is the entire programming model for AT mode.
- **Spring Cloud / Spring Cloud Alibaba:** Spring Cloud Alibaba's `spring-cloud-starter-alibaba-seata` bundles the integration; XID propagates over Feign/RestTemplate automatically.
- **Dubbo:** the Dubbo filter integration (`seata-dubbo` / built into the starter) propagates XID via attachments; Dubbo 3 is supported.
- **Other stacks:** gRPC interceptor, plain HTTP via the `seata-http` integration, and native **Go** via **seata-go** (the Apache incubating Go implementation, interoperable with Java Seata — see §9.5).
- **Under the hood:** Seata wraps your `DataSource` in a proxy (`DataSourceProxy` → `ConnectionProxy` → `StatementProxy`) that intercepts SQL, generates undo logs (AT mode), acquires global locks, and reports to the TC — this is why AT mode is "zero-code": the *framework* does the RM work, not the developer.

---

## 4. Transaction Modes I: AT Mode

### 4.1 What AT Mode Is

**AT** stands for **Automatic (Branch) Transaction**. It is Seata's flagship mode and its reason for existing: a **non-invasive, compensation-based** mode that gives *strong consistency for database resources* with **no business code** — the developer writes ordinary SQL inside a method annotated `@GlobalTransactional`, and the framework handles everything else.

The contract is simple and remarkable at the same time:

```java
@GlobalTransactional(name = "create-order", timeoutMills = 30000)
public void createOrder(Order order) {
    orderDao.insert(order);              // Service A: plain SQL
    inventoryClient.deduct(order.sku()); // Service B: plain SQL
    accountClient.debit(order.amount()); // Service C: plain SQL
}
```

No Try/Confirm/Cancel methods, no saga steps, no XA datasources — just the annotation. The framework's proxy (RM) intercepts each local transaction, records what changed, and makes the global outcome atomic. This is why AT mode is the reason most teams adopt Seata at all.

### 4.2 Mechanics: Undo Log and Global Lock

AT mode rests on two mechanisms — one in the *database* (undo log) and one in the *TC* (global lock).

**The undo log.** Each database participating in AT mode must contain a framework-managed table:

```sql
CREATE TABLE `undo_log` (
  `id`            BIGINT       NOT NULL AUTO_INCREMENT,
  `branch_id`     BIGINT       NOT NULL,           -- branch of the global txn
  `xid`           VARCHAR(128) NOT NULL,           -- global transaction id
  `context`       VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB     NOT NULL,           -- serialized before/after images
  `log_status`    INT          NOT NULL,           -- 0=normal, 1=rollback done
  `log_created`   DATETIME(6)  NOT NULL,
  `log_modified`  DATETIME(6)  NOT NULL,
  UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB;
```

When a branch executes a business SQL statement, the RM proxy:

1. Parses the SQL to identify affected rows and columns.
2. Reads the **before image** of those rows (the state *before* the statement) — using `SELECT ... FOR UPDATE` where needed to lock the rows locally.
3. Executes the business statement.
4. Reads the **after image** of the same rows.
5. Serializes both images into a `rollback_info` blob in `undo_log` (JSON by default; Kryo and other serializers are pluggable), **in the same local transaction** as the business statement.
6. Registers the branch with the TC (including the *lock keys* for the affected rows) and acquires the **global lock**.
7. Commits the local transaction — business change + undo log row commit atomically.

**Rollback via the undo log.** If the TC later decides global rollback, each branch's phase-2 rollback reads its `undo_log` row, *validates* that the current row state still matches the after image (if not, data was changed after phase 1 — a conflict the framework reports rather than silently overwrites), and generates **reverse SQL** from the before image to restore the original state. The undo log row is then marked/logged as handled.

**The global lock.** In the TC's store there is a global lock table keyed by resource + row (the "lock keys" the RM reported). Before any branch commits a change to a row, it must hold the global lock for that row. This is Seata's substitute for a distributed lock manager: it guarantees that **two global transactions never concurrently modify the same row** across services — the distributed analogue of a database write lock.

### 4.3 Isolation Semantics (Important — Read Carefully)

Seata's documentation is precise about what AT mode's isolation actually is, and it is *weaker* than many assume:

- **Dirty writes: prevented.** Because every writer must hold the global lock for the rows it changes, two global transactions cannot interleave writes to the same data — there is no dirty-write problem.
- **Dirty reads: possible by default.** The Seata docs state the default global isolation level is effectively **read uncommitted**: the local databases run at read-committed or above, but a global transaction's uncommitted branches *have already committed locally* (phase 1 is a local commit), so another transaction can read data that will later be rolled back. The docs literally say: *"The isolation level of local database is read committed or above, so the default isolation level of the global transaction is read uncommitted."*
- **Upgrading to read committed:** to prevent dirty reads, the reading side must check the global lock before reading — Seata provides the **`@GlobalLock`** annotation (with `SELECT ... FOR UPDATE` in the method) which *checks the global lock without starting a global transaction*, and the `@GlobalTransactional`-scoped `select for update` pattern. This is a documented, deliberate performance trade: lock checking costs RPCs, so it is opt-in for reads.

This nuance matters enormously in banking (see §10.6): **AT mode does not give you read-committed isolation for free.** Reads of data that other global transactions may still roll back must be explicitly protected with `@GlobalLock` / `SELECT FOR UPDATE`, or the application must tolerate reading in-flight branch states.

### 4.4 AT Workflow (End to End)

```
Service A (TM+RM)                 TC (Seata Server)              Service B (RM)
      │  begin global txn (XID)        │                              │
      ├────────────────────────────────▶│  (global txn created)        │
      │  SQL: UPDATE inventory         │                              │
      │  write undo_log (same local    │                              │
      │  txn) → commit local txn       │                              │
      │  register branch + lock keys   │                              │
      ├────────────────────────────────▶│  (branch A + global locks)   │
      │                                 │                              │
      │   ... downstream call with XID ...                              │
      │                                 │                 SQL: UPDATE accounts
      │                                 │                 write undo_log → commit
      │                                 │                 register branch (B)
      │                                 ◀────────────────────────────────┤
      │  global commit                  │                              │
      ├────────────────────────────────▶│  phase 2: branch A → delete    │
      │                                 │  undo log (async); branch B →  │
      │                                 │  delete undo log (async)       │
      │                                 │                              │
   ROLLBACK PATH (any branch failed):   │                              │
      │  global rollback                │  phase 2: branch A → apply     │
      ├────────────────────────────────▶│  reverse SQL from undo log;    │
      │                                 │  branch B → apply reverse SQL  │
```

### 4.5 AT Mode Limitations

AT mode is not free, and the costs are structural:

- **Undo-log overhead.** Every business statement costs extra reads (before/after images), extra storage (the `undo_log` rows, which must be cleaned up — the branch-commit path deletes them asynchronously, but a backlog is a real operational concern; see §10.6), and extra serialization. Rule of thumb: expect AT mode to add meaningful write-path overhead (commonly cited in the community in the tens-of-percent range on write-heavy tables) and to roughly **double** the SQL traffic on affected statements.
- **Global lock contention.** The TC's global lock table is a serialization point. Hot rows (a single product's inventory row, a single account row) become contention bottlenecks across *all* global transactions touching them — precisely the rows that matter in e-commerce and banking. Lock timeout tuning (§10.4) is a core AT-mode operational skill.
- **Dirty reads by default** (above) — the isolation surprise.
- **SQL coverage limits.** Undo-log generation requires parseable SQL: complex DDL, certain stored-procedure patterns, and exotic SQL may not be supported or may be handled conservatively (Seata's documentation lists supported/unsupported statement types per version). Databases need to be supported by the AT parser set (see §9.3).
- **Performance ceiling.** Every branch does 2 RPCs to the TC (register + report) plus lock operations; the TC is on the hot path for every transaction. AT mode suits short, high-frequency but *not ultra-hot* transactions.

### 4.6 AT vs. XA (The Fundamental Comparison)

| Dimension | **AT** | **XA** |
|---|---|---|
| Protocol | Compensation-based (undo log + global lock) | Real 2PC (X/Open XA prepare/commit) |
| Who does "prepare" | The framework, via undo log | The database itself (XA resource) |
| Resource requirement | Any JDBC DB supported by the parser | DB must implement the XA spec |
| Locking | Global locks on TC + short local locks; local locks released after phase 1 | Resource locks held from prepare until global decision (blocking) |
| Business code | None (`@GlobalTransactional` + plain SQL) | None (JTA-style) |
| Consistency | Strong for writes; dirty reads possible by default | Strong everywhere (isolation guaranteed by the resource) |
| Failure handling | Undo-log reverse SQL | Native rollback in each resource |
| Best fit | Microservices over ordinary MySQL/PostgreSQL; the default choice | Short transactions needing maximal isolation guarantees |

AT mode is effectively *"2PC semantics without the blocking and without XA-capable databases"* — the framework manufactures the prepare/commit behavior from undo logs and a global lock table. The trade is the isolation gap and the undo-log machinery.

---

## 5. Transaction Modes II: TCC Mode

### 5.1 What TCC Is

**TCC** — **Try-Confirm-Cancel** — is the business-level two-phase pattern: the *application* implements the two phases, not the framework or the database. It is Seata's mode for participants that **are not databases** (or not XA/undo-log-able databases): remote APIs, legacy systems, caches, third-party services. TCC is 2PC with the business logic as the resource manager.

The three methods every TCC participant must implement:

| Method | Phase | Semantics | Example (account service) |
|---|---|---|---|
| **Try** | 1 | **Reserve** the resource — check availability and lock/park the amount without finalizing | `tryDebit(acct, amount)` — check balance, park `amount` in a "frozen" state |
| **Confirm** | 2 (commit) | **Finalize** — turn the reservation into the real change | `confirmDebit(acct, amount)` — move frozen → debited |
| **Cancel** | 2 (rollback) | **Compensate** — release the reservation, no-op if Try never ran | `cancelDebit(acct, amount)` — unfreeze, no-op if nothing frozen |

### 5.2 TCC Mechanics Under Seata

The TM begins the global transaction as usual; each TCC participant's RM registers a branch when `Try` runs, and the TC invokes `Confirm` or `Cancel` on **all** branches during phase 2 — the branch's `Try` is the registration trigger, and the TC holds the mapping between branch and the Confirm/Cancel methods. In Seata, a TCC participant is declared with the `@TwoPhaseBusinessAction` annotation:

```java
@TwoPhaseBusinessAction(name = "debitTcc", commitMethod = "confirmDebit", rollbackMethod = "cancelDebit")
public boolean tryDebit(BusinessActionContext ctx, @BusinessActionContextParameter(paramName = "accountId") Long accountId,
                        @BusinessActionContextParameter(paramName = "amount") BigDecimal amount) {
    // 1. check balance; 2. park "amount" in a frozen column; 3. return true
}

public boolean confirmDebit(BusinessActionContext ctx) {
    // move frozen -> debited (idempotent)
}

public boolean cancelDebit(BusinessActionContext ctx) {
    // unfreeze (idempotent; must no-op if Try never succeeded)
}
```

`BusinessActionContext` carries the branch id, XID, and the `@BusinessActionContextParameter`-annotated arguments, so Confirm/Cancel can reconstruct what Try reserved — this is how the framework hands business context across the two phases without you re-serializing it yourself.

### 5.3 TCC Design Requirements: Idempotency, Empty Confirmation, Hanging

TCC shifts the complexity from the framework to the business developer, and the classic failure modes are well catalogued — every one of them must be handled in the *business methods*:

- **Idempotency.** Confirm and Cancel may be invoked more than once (retries after network failures, TC retries). They must be safe to repeat — usually by checking a status flag or a unique business key in the same local transaction.
- **Empty confirmation.** The TC may call Confirm for a branch whose Try **never ran** (e.g., Try timed out but the framework decided to commit anyway, or the Try call itself was lost). Confirm must detect "nothing reserved" and no-op cleanly.
- **Hanging / cancel-without-try.** The reverse: Cancel arriving when Try never executed. The reservation record (or its absence) is the ground truth — "reserved?" is a queryable state, and both Confirm and Cancel branch on it.
- **Try must not partially commit.** Try, Confirm, and Cancel should each be *single local transactions* — a Try that commits half its reservation is a data bug, not a framework issue.
- **Reservation modeling.** The resource design itself must have a "reserved/parked" state (a frozen column, a hold record, a lock table) — TCC forces you to model your data in two-phase shape, which is a real schema design cost.

The "empty confirmation" and "hanging" cases are exactly why TCC is called the *hardest* mode to get right — but it is also the only mode that works when the participant is an arbitrary external system with no transaction protocol at all.

### 5.4 TCC vs. AT

| Dimension | **AT** | **TCC** |
|---|---|---|
| Who implements the two phases | Framework (undo log + global lock) | **Business developer** (Try/Confirm/Cancel) |
| Business code | Zero (annotation + SQL) | Three methods + reservation modeling per participant |
| Resource types | JDBC databases only (parser-supported) | **Anything**: DBs, REST APIs, MQ, legacy systems |
| Consistency | Strong for writes; dirty reads possible | Strong, *business-defined* (reservation semantics) |
| Locking | Global lock table in TC | Business-level reservation (e.g., frozen amount) |
| Development cost | Low | High |
| Operational risk | Undo-log/lock-table issues | Business idempotency/empty-confirm/hanging bugs |

The practical guidance: **AT when the participant is a supported database; TCC when it is not.** Most real Seata deployments are mixed — AT for the database-heavy core services, TCC for the odd non-database dependency (payment gateways, allocation engines, external loyalty systems).

---

## 6. Transaction Modes III: SAGA Mode

### 6.1 The Saga Pattern

The **saga** pattern sequences a long business process as a chain of local transactions, each with a compensating action, and — on failure — executes the compensations of the completed steps in reverse order (the pattern is discussed in the saga sections of [event_stream_processing_guide.md](event_stream_processing_guide.md); the orchestration-vs-choreography debate there applies directly). Sagas have no global locks and no coordinator-held resources: each step commits and releases immediately, so sagas are the natural fit for **long-running processes** (hours, days) where holding locks is impossible.

### 6.2 Seata SAGA: State-Machine-Driven

Seata's SAGA mode implements the *orchestrated* saga as a **state machine**, defined declaratively (JSON) and executed by the Seata runtime — with a visual designer available for authoring the state machines. This is a different flavor from hand-coded saga choreography: the *flow itself* becomes configuration.

A SAGA definition is a graph of states and transitions:

```json
{
  "name": "order-saga",
  "comment": "Place order: create order → deduct inventory → debit account",
  "startState": "createOrder",
  "states": {
    "createOrder": {
      "type": "ServiceTask",
      "ServiceName": "order-service",
      "ServiceMethod": "createOrder",
      "CompensateState": "compensateCreateOrder",
      "Next": { "Success": "deductInventory" }
    },
    "deductInventory": {
      "type": "ServiceTask",
      "ServiceName": "inventory-service",
      "ServiceMethod": "deduct",
      "CompensateState": "compensateDeduct",
      "Next": { "Success": "debitAccount" }
    },
    "debitAccount": {
      "type": "ServiceTask",
      "ServiceName": "account-service",
      "ServiceMethod": "debit",
      "Next": { "Success": "succeed" }
    },
    "compensateDeduct":   { "type": "ServiceTask", "ServiceName": "inventory-service", "ServiceMethod": "compensateDeduct", "Next": { "Success": "compensateCreateOrder" } },
    "compensateCreateOrder": { "type": "ServiceTask", "ServiceName": "order-service", "ServiceMethod": "compensateCreateOrder", "Next": { "Success": "fail" } }
  }
}
```

Key elements: **ServiceTask** states call remote methods; **CompensateState** declares each step's compensation; transitions route on success/failure; and there are additional state types (Choice for branching, Loop, ScriptTask, CompensationTrigger, etc.) for complex flows. The state machine is executed and persisted by the SAGA engine, which gives **recoverability**: an interrupted saga resumes from its last persisted state — the state machine's log *is* the transaction log.

### 6.3 Compensation Semantics

- **Forward path:** steps execute in order; each step's local transaction commits immediately.
- **On failure at step N:** the engine walks the *completed* steps backwards, invoking each one's `CompensateState` (N-1, N-2, … 0) — the reverse chain.
- **Compensation idempotency** is again the developer's responsibility: compensations may be retried after engine crashes and must tolerate double execution.
- The engine also supports **forward recovery / retry** of failed steps and custom retry policies per state — an operational lever the other modes lack.

### 6.4 SAGA vs. the Other Modes

| Dimension | **SAGA** | AT | TCC |
|---|---|---|---|
| Consistency | Eventual (intermediate states visible) | Strong (writes) | Strong (business-defined) |
| Locking | None (each step commits) | Global locks | Business reservations |
| Duration | **Long-running OK** (hours/days) | Short (lock window) | Short-to-medium (reservation) |
| Business code | Per-step methods + compensation; flow defined in state machine | Zero | Heavy (Try/Confirm/Cancel) |
| Flow control | Declarative state machine + designer | None (implicit) | Implicit |
| Best fit | Cross-team business processes, workflows with human/async steps | Standard database CRUD across services | Non-database participants |

The selection rule from Seata's own documentation and community practice: **short transactions → AT; business-level two-phase or non-database resources → TCC; long-running multi-step processes → SAGA; maximal isolation → XA.**

---

## 7. Transaction Modes IV: XA Mode

### 7.1 The XA Protocol

**XA** is the X/Open distributed transaction standard: a real 2PC protocol (prepare/commit/rollback) implemented *by the transaction resource itself* — the database — under the direction of a transaction manager. Oracle Database's XA support and its cross-shard transaction machinery are covered in [oracle_sharding_guide.md](oracle_sharding_guide.md); XA is the industry's only standardized strong-consistency protocol for heterogeneous databases.

### 7.2 Seata's XA Mode

Seata added XA mode (introduced around v1.2.0, 2020) specifically to cover the **strong-consistency scenarios** its compensation modes cannot fully serve. In XA mode:

- The branch transaction is a real **XA transaction** on the resource: the RM starts the XA transaction, executes business SQL, and **prepares** it; the TC decides the global outcome and drives each branch's commit or rollback through the XA protocol.
- **Isolation is native and total** — the database enforces it. The Seata documentation is explicit: *"the XA protocol requires transaction resources themselves to support the specification and protocol. This ensures effective isolation of data from any perspective, meeting global data consistency."*
- **No undo log, no global lock table, no reverse SQL** — the resource does the rollback work natively.
- **Supported resources:** the project currently documents XA support for **MySQL, Oracle, and MariaDB** (the set varies by version; PostgreSQL support has been a long-standing community request — verify against the docs for your version). The classic 2PC limitations apply as usual: **blocking** (prepared resources hold locks until the global decision) and the performance cost of the prepare phase.

### 7.3 XA vs. AT (Recap) and When to Choose XA

| | **XA mode** | **AT mode** |
|---|---|---|
| Consistency | Strong, isolation guaranteed by the DB | Strong for writes; dirty reads possible by default |
| Blocking | Yes (prepare holds locks) | No (local commit in phase 1, locks released) |
| Performance | 2PC round-trips + lock hold | Undo-log overhead + global lock contention |
| DB requirement | XA-capable (MySQL/Oracle/MariaDB per current docs) | Any parser-supported JDBC DB |
| Failure handling | Native DB rollback | Reverse SQL from undo log |

**Use XA mode when:** the transaction is short, the databases support XA, and the isolation guarantee is non-negotiable (e.g., the money-movement core where "read committed or better, guaranteed by the engine" is an audit requirement). Use **AT** when you want most of the strong consistency without XA-capable databases or blocking. In practice XA mode is the least-used of Seata's four modes — most teams choose AT for database work and TCC for everything else — but it is the correct answer for the strictest domains, and its existence completes Seata's spectrum.

---

## 8. Mode Comparison and Selection

### 8.1 The Four-Mode Comparison Table

| Mode | Type | Consistency | Isolation guarantee | Invasiveness (business code) | Performance profile | Blocking | Best for |
|---|---|---|---|---|---|---|---|
| **AT** | Compensation (framework-managed) | Strong for writes; **read uncommitted by default** (dirty reads possible) | Global write locks; reads need `@GlobalLock`/`FOR UPDATE` for RC | **Zero** — `@GlobalTransactional` + plain SQL | Undo-log overhead ×2 SQL; global lock contention on hot rows | No (locks released after phase 1) | The default: short DB transactions across microservices on MySQL/PostgreSQL etc. |
| **TCC** | Business-level two-phase | Strong, **business-defined** (reservation semantics) | Business reservation state | **Heavy** — Try/Confirm/Cancel + idempotency per participant | One extra phase-1 call; no undo log | No (short reservation window) | Non-database participants: APIs, legacy systems, MQ, caches |
| **SAGA** | Long-running orchestration (state machine) | **Eventual** (intermediate states visible) | None by design | Medium — step methods + compensations; flow in JSON DSL | No global locks; engine-persisted state machine | No | Long-running multi-step processes, workflows, cross-team flows |
| **XA** | Real 2PC (X/Open) | **Strong**, guaranteed by the resource | Full DB isolation (native) | Zero (JTA-style) | 2PC round-trips; prepare holds locks | **Yes** | Short transactions with non-negotiable isolation on XA-capable DBs (MySQL/Oracle/MariaDB) |

### 8.2 Selection Guidance

A pragmatic decision sequence used in production adoptions:

```
Is the whole flow over supported JDBC databases?
   ├─ YES → Is isolation "read committed guaranteed" an audit requirement?
   │         ├─ YES → XA mode (if DBs are XA-capable) 
   │         └─ NO  → AT mode  ← DEFAULT
   └─ NO (any non-database participant)
         ├─ Short-lived business operation → TCC
         └─ Long-running multi-step process  → SAGA
```

Refinements: use SAGA for anything that must survive hours or involve humans; use TCC for the *mixed* case (database core + one external API) — a common and sensible architecture is **AT for the DB branches + TCC for the external branch inside one global transaction**; use XA only when auditors/regulators force engine-level isolation. And remember §10.7: for many flows the right answer is *no Seata at all* (outbox + events).

---

## 9. Ecosystem and Integrations

### 9.1 Framework Integrations

Seata's client libraries plug into the dominant Java stacks:

- **Spring Boot** — `seata-spring-boot-starter` (artifact `org.apache.seata:seata-spring-boot-starter` in the 2.x Apache coordinate space) auto-configures data-source proxying, the `@GlobalTransactional` interceptor, XID propagation filters, and the TM/RM. Spring Boot 3 and JDK 17 are supported in current versions.
- **Spring Cloud / Spring Cloud Alibaba** — `spring-cloud-starter-alibaba-seata` wires Seata into the Alibaba microservices stack; XID propagates automatically over Feign and RestTemplate. This is the most common production pairing.
- **Dubbo** — the Dubbo filter propagates XID via RPC attachments; **Dubbo 3** is supported.
- **gRPC** — an interceptor propagates the XID via gRPC metadata.
- **HTTP / Feign** — XID travels in a dedicated HTTP header (default `TX_XID`) via interceptors on both sides.
- **Service mesh?** — Seata is *not* a mesh-level solution: it coordinates at the application/framework layer (the TC is an application-level service). Mesh-based approaches (Istio-style distributed transactions are generally not a thing; mesh solves traffic, not atomicity) are not a direct competitor — worth saying plainly, because "Seata in the mesh" is a recurring confusion. The 2.x direction of separating TC compute from storage (§13.2) is Seata's cloud-native answer.

### 9.2 Registry and Configuration Centers

The Seata Server and clients are pluggable across the standard discovery/config stacks — a practical boon for estates that standardize on one of them:

- **Registry:** Nacos, Eureka, Consul, etcd, ZooKeeper, Redis, plus a plain `file` config for standalone setups. **Nacos** is the de-facto default in the Alibaba ecosystem and the most common in production.
- **Configuration center:** Nacos, Apollo, Consul, etcd, ZooKeeper, `file`. Apollo is common in Chinese enterprises; Nacos covers both registry and config in one.

### 9.3 Database Support

- **AT mode** works with any JDBC database the SQL parser supports. The community-supported matrix includes **MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, and several Chinese commercial databases (DM/DaMeng, OceanBase, GBase, Kingbase)** — exact per-version support lives in the Seata docs' "supported databases" table; multi-primary-key support for Oracle/PostgreSQL and MySQL variants has been steadily expanded in 2.x.
- **XA mode** currently documents **MySQL, Oracle, MariaDB** (check your version's docs; PostgreSQL XA support is a recurring community request).
- Every AT-mode database needs the `undo_log` table (§4.2); the DDL is provided for each database in the project docs. For Oracle XA, the classic Oracle XA configuration applies (see [oracle_database_guide.md](oracle_database_guide.md) and the cross-shard 2PC discussion in [oracle_sharding_guide.md](oracle_sharding_guide.md)).

### 9.4 Message Queues and Transactional Messaging

Two distinct integration patterns are often conflated:

1. **Seata coordinating MQ as a participant** — an MQ send can be a TCC/SAGA participant (Try = prepare message, Confirm = send, Cancel = discard). This is the "Seata wraps your MQ" pattern for flows that need the message send to be part of the global transaction.
2. **RocketMQ's own transactional messages** — RocketMQ (also an Apache project with Alibaba lineage) has *native* transactional messaging (half-message + local transaction checkback), which solves "write DB + send event atomically" *without* Seata — the same problem the outbox pattern solves. Kafka similarly has *no* native transactional-outbox-equivalent (Kafka transactions exist but serve exactly-once stream processing, not DB+event atomicity) — so teams on Kafka typically use the outbox pattern instead ([event_stream_processing_guide.md](event_stream_processing_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md)).

The architectural guidance: if the flow's consistency requirement is "DB write and event publish must be atomic, downstream handles its own state", use **outbox or RocketMQ transactional messages** — cheaper than a global transaction. Use **Seata** when the *business state across services* (not just the event) must be atomic. Many production systems do both: Seata for the strong-consistency core, outbox/MQ for everything else.

### 9.5 Multi-Language Support and Community

- **Java** is the first-class citizen (the entire framework, server, and ecosystem).
- **Go**: **seata-go** (`apache/incubator-seata-go`) is the official Go implementation, targeting **interoperability between Java and Go services in one global transaction** — Go services can participate in a global transaction coordinated by a Java-started TM. It reached the **2.1.0** release line (module `seata.apache.org/seata-go`). Maturity is behind the Java client — treat Go branches as "core modes work, ecosystem edges may lag."
- **Community ports**: there are community implementations/sdks for other languages (PHP, .NET, Node, Python have all seen community efforts over the years), but none is Apache-official and maintenance varies — verify before depending on them. The *server* (TC) is always the Java distribution; only the client SDKs are ported.
- **Community shape**: ~25.9k GitHub stars, an active issue tracker (the famous *"Wanted: who's using Seata"* issue #1246 has ~200 comments from adopting companies), a Chinese-language-heavy but bilingual community, and the standard Apache incubation governance (dev@ list, community-driven releases) since October 2023.

---

## 10. Adoption and Best Practices

### 10.1 Canonical Use Cases

- **E-commerce: order → inventory → account.** The canonical Seata demonstration (and §12's worked example): placing an order must atomically create the order record, deduct inventory, and debit the customer account. Without a global transaction, a crash between the three services leaves "order exists but stock not deducted" or "stock deducted but payment not taken" — both commercially unacceptable.
- **Banking: cross-service transfer.** Debit account A (account service) + credit account B (account service or a second ledger system) must be atomic. This is the money-movement archetype that motivates strong consistency; the reader's own domain (see the banking guides, e.g. [oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) and [data_models_banking_insurance_guide.md](../banking/data_models_banking_insurance_guide.md)) is exactly where this debate lives.
- **Fintech:** wallet top-ups, coupon issuance + order application, loan disbursement + ledger postings, payment gateway reservations (a natural TCC participant).
- **Logistics / reservations:** seat/room inventory + booking + payment, where overselling is the cardinal sin.
- **Inventory-heavy retail:** the "deduct stock" step that every marketplace runs into the moment services are split.

### 10.2 Production Adoption (Verified Facts and Honest Limits)

Publicly verifiable adoption signals:

- **Alibaba** — the origin: TXC ran inside Alibaba's e-commerce systems for years before Fescar was open-sourced; Seata's internal predecessors are the project's own origin story (verified via the project history and Apache proposal).
- **Ant Group (Ant Financial)** — operated GTS, the financial-grade sibling that joined forces with Fescar in April 2019; Seata inherits the GTS design lineage.
- **Meituan** — Meituan's infrastructure team built its internal distributed transaction solution **"Swan" based on open-source Seata** to serve multiple Meituan businesses (documented in the project's own user-case blog, June 2023).
- **Hema (Alibaba's fresh-retail chain)** — Hema Town uses Seata for game-interaction flows; the same blog cites a development cycle reduction from 20 days to 5 days for a Seata-based integration.
- **Community self-reports** — the GitHub issue *"Wanted: who's using Seata"* (#1246) collects adoption statements from many Chinese internet companies (e-commerce, logistics, fintech, gaming); it is the canonical crowd-sourced list, but it is *self-reported and unverified*.

Honest caveats: there is **no official, audited customer roster** (typical for open-source projects), most public adoption stories come from Chinese internet companies, and Western enterprise adoption is real but less publicized. Claims like "JD.com uses Seata" circulate in blogs but could not be verified from primary sources for this guide — treat specific vendor names beyond those above as unverified. For a Crédit Agricole-type environment the relevant question is not "who uses it" but "does it meet our transaction guarantees and support contracts" — which is what §10.3–§10.7 address.

### 10.3 Mode Selection (Summary)

- **AT** for the default database-centric flow (zero business code, strong-enough consistency, non-blocking).
- **TCC** for non-database or business-defined two-phase participants; expect the most engineering effort.
- **SAGA** for long-running, multi-step, human-in-the-loop processes that cannot hold locks.
- **XA** when engine-guaranteed isolation is a regulatory/audit requirement and the DBs are XA-capable.
- Mixed-mode global transactions (AT branches + TCC branch) are supported and common.

### 10.4 Performance Tuning

- **Global lock timeouts.** Set `lock.retryInterval` / `lock.retryTimes` (client) and TC-side lock-hold policies so hot-row contention fails fast instead of queueing silently; monitor global-lock-wait metrics as a top-3 operational signal.
- **Global transaction timeout.** `timeoutMills` on `@GlobalTransactional` bounds how long a global transaction may live; too long → undo logs pile up and locks are held (in AT) or reservations stale (TCC); too short → legitimate slow flows get rolled back mid-flight.
- **Undo-log hygiene.** Branch-commit deletes undo logs asynchronously by default — ensure the async delete backlog is monitored, and schedule cleanup for stale rows (crash leftovers) so `undo_log` tables don't balloon (§10.6).
- **TC capacity.** The TC is on every transaction's critical path: size it (heap, network) for peak branch-registration rate, keep it in the same region, and use `db` or `redis` storage in HA mode; the 2.x storage/compute separation (§13.2) is designed for horizontal TC scaling.
- **Branch reporting batching / async commit.** Tune phase-2 commit processing (async commit is the default in AT) to keep the TC's phase-2 queue from becoming the bottleneck.
- **Serializer choice.** Undo-log serialization (JSON default vs. Kryo) trades debuggability for size/speed; benchmark on large-image tables.

### 10.5 Monitoring and Operations

- **Metrics:** the Seata Server exposes metrics (Prometheus endpoint) covering global/branch transaction counts, active transactions, global-lock wait times, and phase-2 queues; clients expose their own counters. Dashboards exist in the community (Prometheus + Grafana) and the built-in **Seata Console** provides a management UI (2.6.0 adds an **MCP server interface** to the console — §13.4).
- **Logs:** XID-tagged log lines make cross-service tracing of a global transaction feasible with plain grep; for full tracing, correlate the XID with your APM tooling.
- **Health signals to alarm on:** rising global-lock wait time (contention), undo_log table growth (cleanup lag), TC store growth, failed branch reports (network or XID propagation issues), and phase-2 retry backlog.
- **Release/security cadence:** track Seata releases and CVEs (the deserialization CVE-2024-22399 hit ≤1.8.0/2.0.0) — stay on supported 2.4+/2.5+/2.6.x lines (§2.3).

### 10.6 Pitfalls (The Ones That Bite in Production)

- **XID propagation gaps.** Custom HTTP clients, raw threads, async frameworks, or non-integrated RPC paths silently run branches *outside* the global transaction. Test XID propagation explicitly on every new call path; the symptom is "global rollback didn't roll back service X."
- **AT dirty reads.** Default global isolation is read-uncommitted (§4.3). Reporting/read paths that observe in-flight branch data must use `@GlobalLock` + `SELECT ... FOR UPDATE` — in banking, an unpaid/rolled-back debit read by a balance-check service is a production incident.
- **Undo-log explosion.** High write throughput × large rows × slow async cleanup = unbounded `undo_log` growth and, worse, *undo-log validation failures* on rollback (data changed since phase 1). Monitor, tune async delete, and design rollbacks to be rare.
- **Global lock contention on hot rows.** The single inventory row / single account row is a global bottleneck across all transactions (§4.5). Mitigations: split hot rows (inventory buckets, sub-accounts), shorten transaction time, or move the hot path out of global transactions.
- **SAGA compensation failures.** Compensations that are not idempotent, that fail permanently, or that are missing for a step leave the saga stuck; build compensation retries with dead-letter handling and alerting (the state-machine engine supports retry policies — use them).
- **TCC empty-confirm / hanging / non-idempotent methods** (§5.3) — the classic source of "we rolled back but the money still moved" bugs; the reservation-state check must be in the same local transaction as the state change.
- **Blocking issues in XA mode.** Prepared XA branches hold DB locks until the global decision; a slow TC or a stuck participant blocks the whole resource. Keep XA windows short and the TC healthy.
- **Global rollback of long transactions.** The longer the global transaction, the more likely some branch's data moved between phase 1 and phase 2 — AT mode will report undo-validation conflicts rather than silently corrupt data, but each conflict is a manual reconciliation job.
- **Security:** stay off EOL versions; CVE-2024-22399 (deserialization) is the documented critical example.

### 10.7 When NOT to Use Seata (The Honest View)

Seata is a serious tool with a serious operational footprint (a stateful TC cluster, undo logs, lock tables, new failure modes). It is the right answer only when the flow genuinely needs *stronger* guarantees than the alternatives give:

- **If eventual consistency is acceptable** — and for most flows it is — the **outbox pattern** (+ event streaming) is cheaper, simpler, and more resilient: no coordinator, no locks, no undo logs. See the outbox deep-dives in [event_stream_processing_guide.md](event_stream_processing_guide.md) and [kafka_alternatives_guide.md](kafka_alternatives_guide.md), and the CQRS/saga references in [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md). A saga you can *write in business code* is often better than a saga framework plus a TC.
- **If the flow can be collapsed into one database** — a single logical database with native transactions (or Oracle sharding's XA across shards, [oracle_sharding_guide.md](oracle_sharding_guide.md)) — do that first. Distributed transactions are a *splitting* cost; don't pay it for data that didn't need to be split.
- **If the transaction rate on hot rows is extreme** — global lock contention will cap throughput below what a single-DB design or an outbox design achieves.
- **If your estate is polyglot or non-Java** — seata-go exists but lags Java; a Java TC + Java-centric ecosystem is a real constraint.
- **Seata is not a silver bullet:** it does not make network failures disappear, it adds a stateful infrastructure dependency, and it changes your failure-mode vocabulary (undo conflicts, empty confirms, XID gaps). Adopt it like the middleware it is — with runbooks, monitoring, and an owner — or not at all.

---

## 11. Comparison with Alternatives

### 11.1 The Landscape Table

| Solution | Type | Consistency | Invasiveness | Language | Maturity | Best for |
|---|---|---|---|---|---|---|
| **Apache Seata** | Distributed transaction framework (4 modes: AT/TCC/SAGA/XA) | Strong (AT/XA/TCC) to eventual (SAGA) | Low (AT: annotation only) to high (TCC) | Java (client); Go via seata-go; Java server | Very high in Java ecosystem; Apache incubating since Oct 2023 | Java/Spring microservices needing strong consistency across DBs without XA |
| **Outbox pattern** (self-built or via frameworks) | Messaging/event pattern | Eventual | Low (one table + relay) | Any | Pattern: proven; impl-dependent | Event-driven decoupling; the default "good enough" answer |
| **RocketMQ transactional messages** | Native transactional messaging | Eventual (downstream) | Low | Java (client) | High (Apache TLP) | "Write DB + publish event" atomicity without a coordinator |
| **Axon Framework** | Event sourcing + CQRS + saga framework | Eventual | Medium-high (event-sourced model) | Java | High (commercial + OSS) | DDD/event-sourced systems; sagas over events |
| **Eventuate Tram** | Saga/outbox framework (Chris Richardson) | Eventual | Medium | Java (also other langs in Eventuate platform) | Medium-high | Message-based sagas and transactional outbox in Spring/Java |
| **Atomikos** | JTA/XA transaction manager (2PC) | Strong | Low (JTA datasources) | Java | High (commercial) | Classic JTA/XA inside a process or app server |
| **Narayana** (JBoss/WildFly) | JTA/XA transaction manager (2PC) | Strong | Low (JTA) | Java | High (Red Hat) | JTA/XA in WildFly/JBoss estates |
| **Oracle XA / grid infrastructure** | Native 2PC across Oracle databases | Strong | Low | SQL/Java | Highest | Cross-shard Oracle transactions — see [oracle_sharding_guide.md](oracle_sharding_guide.md) |
| **AWS Step Functions** | Managed saga orchestration | Eventual | Medium (workflow definition) | Any (SDK) | High (AWS) | Long-running orchestrations on AWS; no strong-consistency claim |
| **DynamoDB transactions / Spanner** | Platform-native transactions | Strong (single service) | Low | Any | High | When you can live *inside* one cloud data service |
| **"AWS Mithra"** | — does not exist as a transaction service — | — | — | — | — | See §11.2 |

### 11.2 Clearing Up the "AWS Mithra" Confusion

The task brief asked to verify **"AWS Mithra / AWS Transactional Outbox"** as a cloud distributed-transaction offering. Verified result: **AWS Mithra is not a distributed transaction service at all.** It is AWS's **malicious-domain detection system** — a neural-network graph model (publicly described as ~3.5 billion nodes / 48 billion edges) that scores and blocks malicious domains across AWS's DNS infrastructure. There is also **no managed "Transactional Outbox" service** on AWS. The accurate cloud landscape is:

- **Saga orchestration:** AWS **Step Functions** (managed, durable, but *not* strong-consistency).
- **Native transactions inside one service:** **DynamoDB transactions** (up to 100 items, 4 MB) and **Spanner** (Google) — strong consistency *only if you live inside that single data service*.
- **Classic 2PC:** there is no AWS-managed RDBMS 2PC middleware; if you need XA across RDBMS in the cloud you run it yourself (Atomikos/Narayana/Oracle XA) — or use Seata.

This matters for the verdict below: the "cloud-native distributed transaction middleware" slot is largely **empty** — which is precisely the gap Seata (self-hosted) fills for Java estates.

### 11.3 The Verdict: Seata's Niche

Seata's defensible niche, stated without hype:

1. **The Java/Spring ecosystem's default strong-consistency option.** If your estate is Spring Boot + MySQL/PostgreSQL + (optionally) Dubbo/Spring Cloud Alibaba, Seata's AT mode delivers distributed atomicity with less business code than any alternative — *including hand-rolled sagas* — because the compensation machinery is framework-generated.
2. **The AT mode's non-invasiveness is the moat.** "One annotation, plain SQL, strong consistency" is a combination no saga framework (Axon, Eventuate) and no JTA manager (Atomikos, Narayana — which need XA resources) matches for ordinary databases. Teams adopt Seata for AT and stay for the mode breadth.
3. **"Alibaba-grade" provenance.** The framework was extracted from systems doing real e-commerce and financial volumes at Alibaba/Ant scale; that pedigree (and the Chinese-internet adoption base) is a genuine signal of production hardening, even if the Apache graduation is still pending.
4. **Where it loses:** to outbox/events on cost and simplicity when eventual consistency suffices; to platform-native transactions when you can consolidate into one cloud data service; to classic JTA when you're already all-in on XA-capable resources; to Step Functions for pure long-running orchestration. And it is a Java-centric bet — Go support exists but is not parity.

For a Crédit Agricole-style estate (Java, Spring, Oracle/PostgreSQL, strong governance): Seata is a credible candidate for the "cross-service money movement" flows — with the honest caveats of §10.7, and with XA mode or single-database consolidation evaluated first where the isolation bar is regulatory.

---

## 12. Worked Example: Order–Inventory–Account

### 12.1 Scenario

Classic e-commerce ordering, split across three services, each with its own database (database-per-service):

- **Order service** (`order_db`): inserts the order row (status = CREATED).
- **Inventory service** (`inventory_db`): decrements stock for the SKU.
- **Account service** (`account_db`): debits the customer's balance.

Business invariant: *all three happen, or none.* A partial outcome (order created, stock not deducted) is unacceptable. This is the exact scenario Seata's demo (and most tutorials) use, and it is the AT-mode showcase.

### 12.2 Setup

1. **Deploy the Seata Server** (TC) — standalone for dev: `sh seata-server.sh -p 8091 -m file` (or `-m db` with a shared schema for HA, §3.5). Register it with Nacos (or use `file` registry for the demo).
2. **Add the client dependency** to all three services: `org.apache.seata:seata-spring-boot-starter:2.6.0` (plus Spring Cloud Alibaba's `spring-cloud-starter-alibaba-seata` if on that stack).
3. **Create the `undo_log` table** in `order_db`, `inventory_db`, and `account_db` (§4.2 DDL — MySQL version shown there).
4. **Annotate the entry point** with `@GlobalTransactional` and add Feign clients for the cross-service calls. Nothing else is required — AT mode does the rest.

### 12.3 The AT Flow (Diagram)

```
Client
  │  POST /order
  ▼
┌─────────────────────────── ORDER SERVICE (TM + RM) ───────────────────────────┐
│ @GlobalTransactional createOrder()                                            │
│   1. TM ──begin(XID)──────────────────────────────▶ TC (Seata Server)          │
│   2. INSERT INTO orders ...                                                    │
│      RM: before-image → INSERT → after-image → undo_log row                    │
│      register branch + global lock on order row ──▶ TC                        │
│      commit local txn (order + undo_log together)                              │
│   3. inventoryClient.deduct(sku)  ──(XID in HTTP header)──▶ INVENTORY SERVICE  │
│        UPDATE inventory SET qty = qty - 1 WHERE sku=?                          │
│        RM: undo_log + branch register + local commit                           │
│   4. accountClient.debit(amount) ──(XID)──────────▶ ACCOUNT SERVICE            │
│        UPDATE account SET balance = balance - ? WHERE id=?                     │
│        RM: undo_log + branch register + local commit                           │
│   5. TM ──global commit──────────────────────────▶ TC                          │
│        TC: phase-2 → each branch deletes its undo_log (async)                  │
│        return 200 OK (order placed)                                            │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 12.4 Rollback Scenario (Inventory Insufficient)

```
   3'. inventoryClient.deduct(sku)  ──(XID)──▶ INVENTORY SERVICE
        UPDATE inventory SET qty = qty - 1 ...  →  0 rows affected (out of stock)
        RM reports branch FAILED (or throws) ──▶ TC
        exception propagates back to OrderService with the XID context
   4'. TM catches exception ──global rollback──▶ TC
        TC: phase-2 rollback for EVERY registered branch:
          branch (order): read undo_log → validate after-image → reverse SQL:
                          DELETE FROM orders WHERE id = ?   (restores before-image)
          branch (inventory): no change was committed → nothing to undo
        undo_log rows marked handled; global txn = ROLLED_BACK
        return 500 "Insufficient inventory"  (order never partially exists)
```

The point to internalize: **order creation was already locally committed in step 2 — the global rollback is what removes it**, via the undo log, *after* the fact. That is AT mode's compensation-based atomicity in action.

### 12.5 Code Sketch

```java
// ---- OrderService (the TM: opens the global transaction) ----
@Service
public class OrderService {

    @Autowired private OrderDao orderDao;
    @Autowired private InventoryClient inventoryClient;
    @Autowired private AccountClient accountClient;

    @GlobalTransactional(name = "create-order", timeoutMills = 30000,
                         rollbackFor = Exception.class)
    public void createOrder(Order order) {
        order.setStatus("CREATED");
        orderDao.insert(order);                    // branch 1: local txn + undo_log
        inventoryClient.deduct(order.getSku(), 1); // branch 2 (XID auto-propagated)
        accountClient.debit(order.getAccountId(),
                            order.getAmount());    // branch 3 (XID auto-propagated)
    }
}

// ---- Feign clients (Spring Cloud; XID rides the TX_XID header automatically) ----
@FeignClient(name = "inventory-service")
public interface InventoryClient {
    @PostMapping("/inventory/deduct")
    void deduct(@RequestParam("sku") String sku, @RequestParam("qty") int qty);
}

@FeignClient(name = "account-service")
public interface AccountClient {
    @PostMapping("/account/debit")
    void debit(@RequestParam("accountId") Long accountId,
               @RequestParam("amount") BigDecimal amount);
}

// ---- InventoryService (pure RM: plain SQL, nothing Seata-specific) ----
@RestController
public class InventoryController {
    @PostMapping("/inventory/deduct")
    public void deduct(String sku, int qty) {
        int updated = inventoryDao.deductStock(sku, qty);  // plain SQL
        if (updated == 0) throw new InsufficientStockException(sku); // → global rollback
    }
}
```

Note: `rollbackFor = Exception.class` matters — by default the interceptor rolls back on runtime exceptions; checked exceptions need explicit configuration.

### 12.6 Configuration (application.yml, Seata 2.x)

```yaml
seata:
  enabled: true
  application-id: order-service
  tx-service-group: my_tx_group          # group name; maps to a TC cluster in registry
  service:
    vgroup-mapping:
      my_tx_group: default               # registry name of the TC cluster
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: SEATA_GROUP
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
  data-source-proxy-mode: AT             # AT | TCC | XA | SAGA(per-datasource choice)
  client:
    rm:
      lock:
        retry-interval: 10               # ms between global-lock retries
        retry-times: 30                  # total lock-retry attempts
        retry-policy-branch-rollback-on-conflict: true
    tm:
      default-global-transaction-timeout: 30000   # ms
  # NOTE: the @GlobalTransactional timeoutMills overrides the default above
```

The `undo_log` DDL (§4.2) must exist in each participating database, and each service's `DataSource` must be the proxied one (the starter does this automatically when `seata.enabled: true`).

### 12.7 Verifying the Demo

- **Happy path:** create order → all three tables updated; `undo_log` rows briefly appear then are deleted (async cleanup); TC logs show `GlobalCommit`.
- **Rollback path:** order a SKU with zero stock → HTTP 500; verify `orders` table has **no** new row (the inserted order was undone via the undo log) and the account was **not** debited.
- **XID propagation check:** add a fourth, deliberately non-Feign HTTP call (e.g., raw `RestTemplate` without the interceptor) and observe that its changes are *not* rolled back — the classic "silent branch escape" (§10.6) demonstrated in five minutes.

---

## 13. The Future: 2026 and Beyond

### 13.1 Apache Graduation

As of August 2026 Seata remains an **incubating** project (entered October 2023); no graduation announcement had been made. Graduation to an Apache top-level project is expected eventually (the project is mature, the community is active, and the ASF proposal/incubation process is well along), but no date can be verified — treat "TLP status" as a governance milestone to track, not a technical dependency. (Re-verify before citing: this is the single most likely item in this guide to go stale.)

### 13.2 Cloud-Native Direction

- **Storage/compute separation** is the headline 2.x architectural theme: TC compute nodes horizontally scalable, with storage in shared DB or Redis — the design that makes Seata a genuinely deployable cluster on Kubernetes (stateful TC is the thing that makes K8s deployment awkward; separating compute from store fixes it).
- **Raft-based HA mode** is in beta — built-in consensus would remove the external-store dependency for HA entirely.
- **Operator/Helm patterns** for the Seata Server are maturing in the community; expect the server to become an ordinary K8s workload.
- Seata remains an *application-layer* coordinator — mesh-based transaction support is not a real market category (mesh solves traffic, not atomicity) — so the cloud-native story is "deploy the TC better", not "disappear into the mesh".

### 13.3 Multi-Language

**seata-go** (Apache incubating, 2.1.x) is the official path to Java↔Go interoperability in one global transaction; expect Go parity to keep closing but not to fully match Java. Community ports (PHP/.NET/Node/Python) remain unofficial and uneven — the realistic multi-language story for 2026+ is **Java core + Go participation**, with everything else via HTTP/XID propagation or outbox.

### 13.4 Observability and Tooling

- The **Seata Console** is becoming the operations center; the 2.6.0 **MCP server support for the console** is a notable signal — AI-tooling interfaces (Model Context Protocol) arriving in transaction infrastructure means agents will soon query/inspect global transaction state conversationally.
- Metrics (Prometheus), XID-correlated logging, and the phase-2/queue telemetry (§10.5) will keep improving; expect deeper integration with OpenTelemetry-style tracing in the 2.x line.

### 13.5 AI-Assisted Transaction Decisions (Speculative)

Two honest observations, not predictions: (1) the 2.6.0 MCP console integration shows the project embracing LLM-era tooling — a natural next step is LLM-assisted *analysis* (explain a stuck global transaction, suggest lock-contention fixes, draft reconciliation queries); (2) AI *deciding* transaction outcomes (e.g., choosing commit vs. compensate based on learned risk) is not on any roadmap and is arguably a bad idea for money movement — regulation and auditability demand deterministic protocols. Expect AI around the edges (ops, analysis, DSL authoring), not in the commit path.

### 13.6 Trends Summary

- Strong-consistency middleware stays relevant wherever money/inventory move across services — a category Seata dominates in the Java world.
- The pattern debate (2PC vs. saga vs. outbox) is not converging on one answer; frameworks that *cover the whole spectrum* (Seata's four modes) fit the real world better than single-pattern tools.
- Cloud vendors still don't offer managed RDBMS distributed transactions — leaving the slot Seata occupies unfilled by hyperscalers.
- Watch items for 2026–2027: Apache graduation, Raft HA leaving beta, seata-go parity, and whether the MCP/console work becomes an agent-friendly ops layer.

### 13.7 Fact-Check Notes (August 2026)

Verified against primary sources (seata.apache.org, the Apache Incubator proposal, GitHub, project blog): Fescar acronym and 2019 open-sourcing; April 2019 rename to Seata; October 2023 Apache Incubation entry; **still incubating**; latest release 2.6.0 (2.5.0 on 2025-08-05); ~25.9k stars / ~8.9k forks; AT-mode isolation semantics (read-uncommitted default, `@GlobalLock`); XA-mode resource list (MySQL/Oracle/MariaDB); seata-go existence and 2.1.x line; Meituan (Swan) and Hema adoption cases; CVE-2024-22399 scope; **AWS Mithra is a malicious-domain detection system, not a transaction service**. Not verifiable from primary sources: any specific graduation date (none exists), a definitive production-user roster beyond the documented cases, and per-version database-support minutiae (check the docs for your exact Seata version).

---

## 14. Glossary

- **Seata** — Apache Seata (incubating): the open-source distributed transaction framework; acronym for **S**imple **E**xtensible **A**utonomous **T**ransaction **A**rchitecture.
- **Fescar** — Seata's original name; acronym for **F**ast **&** **E**a**S**y **C**ommit **A**nd **R**ollback; open-sourced by Alibaba in January 2019, renamed to Seata in April 2019.
- **TC (Transaction Coordinator)** — the Seata Server component that maintains global/branch transaction state and drives global commit/rollback; owns the global lock table and transaction store.
- **TM (Transaction Manager)** — the application-side component that defines the global transaction boundary (begin/commit/rollback) and propagates the XID.
- **RM (Resource Manager)** — the application-side component managing branch transactions: registration, status reporting, and phase-2 execution (undo-log apply, TCC confirm/cancel, XA calls).
- **XID** — the global transaction identifier (typically `<TC-addr>:<txn-id>`), propagated across service calls to tie branches to their global transaction.
- **Global transaction** — the Seata-managed unit spanning all participants; states: begin → branch register → branch report → global commit/rollback.
- **Branch transaction** — one participant's local unit of work (local transaction + its undo/compensation capability) inside the global transaction.
- **AT mode** — Automatic Transaction mode: compensation-based strong consistency via undo logs and global locks; non-invasive (`@GlobalTransactional` + plain SQL).
- **Undo log** — the `undo_log` table holding before/after images of changed rows, used by AT mode to reverse phase-1 changes on global rollback.
- **Global lock** — the TC-held write lock on rows modified by AT branches; prevents dirty writes across global transactions.
- **TCC** — Try-Confirm-Cancel: business-level two-phase pattern; Try reserves, Confirm finalizes, Cancel compensates.
- **Try** — TCC phase 1: check and reserve the resource (e.g., freeze funds) without finalizing.
- **Confirm** — TCC phase 2 (commit): turn the reservation into the real business change.
- **Cancel** — TCC phase 2 (rollback): release the reservation; must be idempotent and tolerate "empty cancel" (Try never ran).
- **SAGA** — the long-running-transaction pattern: sequential local transactions with compensating actions executed in reverse on failure; Seata implements it as a persisted state machine.
- **State machine** — the JSON/DSL graph of states and transitions that defines a Seata SAGA flow (ServiceTask, CompensateState, transitions, retry policies).
- **XA** — the X/Open distributed transaction standard: real two-phase commit (prepare/commit/rollback) implemented by the transaction resource itself.
- **2PC (two-phase commit)** — the classic atomic-commit protocol: prepare all participants, then commit or abort all; strong consistency but blocking and coordinator-dependent.
- **Compensation** — the business or framework action that undoes an already-committed local change (undo-log reverse SQL, TCC Cancel, SAGA compensation step).
- **Eventual consistency** — the guarantee that, absent new failures, all replicas/participants converge to the same state over time; intermediate states may be inconsistent.
- **Strong consistency** — the guarantee that any read reflects the latest committed write and that participants never observe partial outcomes.
- **Outbox** — the transactional-outbox pattern: business change + event row written in one local transaction, with a relay publishing events; the main alternative to distributed transactions (see [event_stream_processing_guide.md](event_stream_processing_guide.md)).
- **Saga (pattern)** — see SAGA; also used generically for any compensation-chain design, orchestrated or choreographed.
- **Spring Cloud** — the Spring ecosystem's microservices framework; Seata integrates via Spring Cloud Alibaba with automatic XID propagation over Feign/RestTemplate.
- **Dubbo** — Alibaba's high-performance Java RPC framework; Seata propagates the XID via Dubbo attachments (Dubbo 3 supported).
- **Nacos** — Alibaba's service discovery + configuration platform; the default Seata registry/config center.
- **Spring Boot** — the Java application framework; `seata-spring-boot-starter` provides turnkey Seata client integration.
- **@GlobalTransactional** — the Seata annotation marking a method as a global transaction boundary (TM begin/commit/rollback).
- **@GlobalLock** — the Seata annotation that checks the global lock for reads without starting a global transaction; the tool for read-committed behavior in AT mode.
- **Dirty read** — reading data committed by a branch whose global transaction later rolls back; possible by default in AT mode.
- **Lock contention** — competition among global transactions for the same global locks/rows; the main AT-mode scaling bottleneck on hot data.
- **Idempotency** — the property that executing an operation multiple times has the same effect as once; required of TCC Confirm/Cancel and SAGA compensations.
- **Distributed transaction** — a transaction spanning multiple independently managed resources (databases/services), whose atomicity must be constructed via coordination (2PC, TCC, SAGA, or Seata-style compensation).
