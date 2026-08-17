# How to Avoid Data Loss When Using Message Queues: The Message-Queue Reliability Guide

*A comprehensive deep-dive on message-queue reliability: the data-loss problem and its three loss points, the delivery semantics (at-most-once, at-least-once, exactly-once), the producer-side acks and retries, the broker-side replication and durability, the consumer-side offsets and commit discipline, idempotency and dedupe keys, dead-letter queues, exactly-once mechanisms (Kafka transactions, SQS FIFO), broker guarantees (SQS, RabbitMQ, Pulsar), design patterns (transactional outbox, event sourcing), monitoring (lag, retries), and a worked banking payment pipeline.*

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Messaging / Distributed-Systems Reliability (technology/)
> **Audience:** Solution architects, platform engineers, event-driven / integration teams
> **Last Updated:** August 2026

**Cross-references:** [event_stream_processing_guide.md](event_stream_processing_guide.md) (the at-least-once discipline and Kafka durability in streaming pipelines — the foundation this guide builds on), [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (broker selection; the guarantees table in §9 here is the reliability lens on that comparison), [apache_seata_guide.md](apache_seata_guide.md) (distributed transactions, saga, exactly-once-adjacent coordination), [complex_event_processing_guide.md](complex_event_processing_guide.md) (event-pattern detection on reliable streams — lightly), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (backpressure and admission control as the flow-control cousin of delivery guarantees), [camel_camelk_research.md](camel_camelk_research.md) and [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) (integration tooling that must inherit these guarantees — lightly), [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md) and [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md) (the banking event pipelines that motivate §12's worked example), [banking/financial_fraud_detection_at_scale_guide.md](banking/financial_fraud_detection_at_scale_guide.md) (fraud event pipelines — the same reliability requirements, different consumers — lightly).

---

## Table of Contents

1. [The Data-Loss Problem](#1-the-data-loss-problem)
2. [The Delivery Semantics](#2-the-delivery-semantics)
3. [The Producer-Side: Acks, Retries, Idempotent Producers](#3-the-producer-side-acks-retries-idempotent-producers)
4. [The Broker-Side: Replication, Durability, Election](#4-the-broker-side-replication-durability-election)
5. [The Consumer-Side: Offsets and Commit Discipline](#5-the-consumer-side-offsets-and-commit-discipline)
6. [The Idempotency: Dedupe Keys and the Dedupe Store](#6-the-idempotency-dedupe-keys-and-the-dedupe-store)
7. [The DLQ: Dead-Letter Queues](#7-the-dlq-dead-letter-queues)
8. [The Exactly-Once: Kafka Transactions and SQS FIFO](#8-the-exactly-once-kafka-transactions-and-sqs-fifo)
9. [The Broker Guarantees: SQS, RabbitMQ, Pulsar, Kafka](#9-the-broker-guarantees-sqs-rabbitmq-pulsar-kafka)
10. [The Design Patterns: Transactional Outbox and Event Sourcing](#10-the-design-patterns-transactional-outbox-and-event-sourcing)
11. [The Monitoring: Lag, Retries, DLQ Depth](#11-the-monitoring-lag-retries-dlq-depth)
12. [The Worked Example: A Banking Payment Pipeline](#12-the-worked-example-a-banking-payment-pipeline)
13. [The Summary: Reliability in One Page](#13-the-summary-reliability-in-one-page)
14. [Verification and Claims-Status](#14-verification-and-claims-status)
15. [Glossary](#15-glossary)

---

## 1. The Data-Loss Problem

### 1.1 The Problem: The Three Loss Points

**"The message queue lost my message" is almost never one bug — it is one of three broken hand-offs.** A message in transit passes through exactly three ownership boundaries, and every one of them is a place where a message can silently vanish:

1. **The producer → broker hand-off.** The application produces a message; the broker never receives it, or receives it and fails to persist it. Causes: network failure, producer crash before send, broker crash before the write is durable, or — most commonly — a configuration that *accepts* the loss (`acks=0`). The producer believes the message is "sent"; nothing anywhere has it.
2. **The broker → consumer hand-off.** The broker has the message but the consumer never gets it, or gets it and loses it. Causes: consumer crash mid-batch, offset committed before processing completed, retention/expiry deleting messages before consumption, or an unclean leader election discarding un-replicated data (see §4).
3. **The consumer → downstream hand-off.** The consumer processed the message but the *effect* was lost — the database write failed silently, the API call was dropped, the state update was applied twice. This is the loss point people forget: the queue delivered perfectly, and the data still disappeared because the consumer's own output was not durable or not idempotent.

The essential framing, verified across the Kafka and messaging literature: **a message is "safe" only when it is durably recorded in at least two independent places** (e.g., producer's source of truth + broker log; broker log + consumer's output store). Until the hand-off to the next owner completes durably, the message exists in exactly one place, and a crash at that instant loses it. The entire reliability discipline in this guide is a set of mechanisms that shrink the windows in which a message exists in only one place — and a set of recovery mechanisms (retries, replay, dedupe) for when a crash still happens.

This is the same decomposition the event-streaming guide uses for stream processing fault tolerance ([event_stream_processing_guide.md](event_stream_processing_guide.md) §on fault tolerance): the difference here is we zoom into the *messaging layer* itself — the three hand-offs — rather than the processing engine.

### 1.2 Why It Matters: Payments, Orders, Events

Why does this deserve a dedicated guide? Because for the systems banks actually run, "probably delivered" is not a guarantee:

- **Payments.** A payment instruction that is produced but never reaches the execution engine is not "late" — it is *missing money movement*. In a real-time payments context (the instant-payment rails, the payment-hub integrations), a lost credit is a customer-facing incident, a regulatory matter (the dispute/chargeback machinery exists precisely because losses happen), and a reconciliation break. Payment pipelines are the canonical *must-not-lose* workload, and the worked example in §12 is a payment pipeline for exactly this reason.
- **Orders.** An order event lost between the order service and the inventory/fulfillment service means an accepted order that never ships — a customer-visible failure that cascades into refunds, support load, and revenue leakage. Order pipelines are the canonical *must-not-lose, must-not-duplicate* workload (duplicate order = double shipment, double charge).
- **Events (the event backbone).** In event-driven banking architecture — the core-banking event streams, the posting-engine events, the fraud-detection events (see [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md), [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md), [banking/financial_fraud_detection_at_scale_guide.md](banking/financial_fraud_detection_at_scale_guide.md)) — events are the *system of record* for what happened. Lose the event and you lose the audit trail, the ability to rebuild derived state, the ability to answer "what actually occurred?" in a dispute. Event loss is silent data corruption of the ledger itself.

The common thread: **a lost message in a financial pipeline is not a retry problem, it is a money/trust problem.** That is why this guide exists — the stakes justify the configuration discipline in the checklists.

### 1.3 The Loss Taxonomy: Silent Drop, Crash Loss, Duplicate

The three loss points manifest as three observable *loss modes* — and note that the third one is not "loss" in the naive sense:

- **Silent drop.** The message is acknowledged as handled but never durably stored or processed. Classic causes: `acks=0` producers (fire-and-forget), auto-committed offsets when processing fails after the commit, retention windows shorter than processing time, broker crashes before fsync. *Signature: no error anywhere; data just isn't there.* This is the most dangerous mode because monitoring usually doesn't catch it — you need end-to-end reconciliation (§11) to see it.
- **Crash loss.** The message existed in one place and the owner crashed before the hand-off completed. Producer crashed before the broker acked; broker crashed before replicating; consumer crashed after processing but before committing (this one is a *duplicate*, not a loss — see below). Crash loss is the mode that replication, acks, and commit discipline address.
- **Duplicate (the "loss" that is actually a double).** At-least-once delivery *guarantees* duplicates: any retry after an ambiguous outcome (did the broker commit before the connection died? did the consumer commit before it crashed?) produces a second copy. Duplicates are data-loss-adjacent: a duplicated debit is as bad as a lost one. The fix is not to prevent duplicates (in at-least-once you cannot) but to make them harmless via idempotency (§6).

### 1.4 The Problem Table

| Loss point | Loss mode | Impact | Typical root cause |
|---|---|---|---|
| **Producer → broker** | Silent drop / crash loss | Message never enters the pipeline | `acks=0` or `acks=1`, no retries, no idempotent producer |
| **Broker (storage)** | Crash loss | Acknowledged messages vanish | Replication factor 1, `min.insync.replicas=1`, fsync disabled, unclean leader election |
| **Broker → consumer** | Silent drop | Message never processed | Auto-commit before processing, retention/expiry too short, consumer crash mid-batch |
| **Consumer → downstream** | Silent drop / duplicate | Effect lost or applied twice | Non-idempotent writes, no dedupe, failed DB write ignored |

The table is the diagnostic map: when a message goes missing, walk the three hand-offs and ask *which ownership boundary was the message in when it disappeared* — that determines which section of this guide fixes it.

### 1.5 The Message's Life: Where Each Mechanism Fits

To make the three loss points concrete, here is one message's journey through a correctly-configured pipeline, with the §3–§7 mechanism that protects each step:

| Step | What happens | The mechanism that protects it | Failure if the mechanism is missing |
|---|---|---|---|
| 1. Business fact occurs | Payment booked; event written to outbox table in the same DB transaction | Transactional outbox (§10.1) | Dual-write: event published without the fact, or fact committed without the event |
| 2. Relay publishes | Outbox relay sends to Kafka with `acks=all` | `acks=all` + idempotent producer (§3) | Producer told "sent" while only the leader (or nobody) has it |
| 3. Broker persists | Leader + ≥1 follower (ISR) durably hold the record; producer acked | RF=3 + `min.insync.replicas=2` (§4) | Acked write lives on one disk; one crash loses it |
| 4. Broker serves | Consumer polls; offset still at previous position | Offset = uncommitted until processing done (§5) | Auto-commit advances past unprocessed messages |
| 5. Consumer processes | DB write applied; dedupe row inserted atomically | Idempotency (§6) | Redelivery after a crash double-applies the effect |
| 6. Consumer commits | Offset committed after processing | Commit-after-process (§5.3) | Crash before commit is fine (redelivery); crash after commit means the effect must already be durable |
| 7. Message fails forever | Retries exhausted; routed to DLQ | DLQ + alerting + redrive (§7) | Poison message stalls the partition; failure invisible |
| 8. Audit | Nightly reconciliation compares outbox vs. applied | Reconciliation (§11.3) | Residual loss invisible until a customer or regulator finds it |

Read the table top to bottom and it *is* the guide: each row's mechanism closes exactly one loss window, and the stack is only as strong as its weakest row — which is why the checklists in §3–§7 are stated as non-negotiables rather than suggestions.

---

## 2. The Delivery Semantics

### 2.1 The Semantics: At-Most-Once, At-Least-Once, Exactly-Once

Every queue/broker product implements one of three *delivery semantics* — the contract between producer, broker, and consumer about what happens to a message across failures. These are the vocabulary of every reliability discussion, so they are worth stating precisely (verified — this is the standard tripartite classification used across Kafka, AWS, and RabbitMQ documentation):

- **At-most-once.** A message is delivered zero or one times. The broker (or producer) gives up on the first failure. Implemented by fire-and-forget sends and by committing offsets *before* processing. *The contract: you will never see a duplicate, but you may see nothing at all.* Data loss is a *feature of the contract* — acceptable only for telemetry/analytics where a dropped sample is harmless.
- **At-least-once.** A message is delivered one or more times. The producer retries until the broker acks; the consumer commits the offset only *after* processing completes; on any ambiguous failure the message is redelivered. *The contract: you will never lose a message, but you will see duplicates.* This is the default for every serious broker (Kafka with `acks=all` + retries, SQS standard, RabbitMQ with publisher confirms + manual acks, Pulsar persistent topics).
- **Exactly-once.** A message is delivered and its *effect* is applied exactly once — no loss, no duplicates. Either the broker/protocol provides it (SQS FIFO's dedup window, Kafka transactions *within the cluster*) or the application *simulates* it (at-least-once + idempotent consumers — the subject of §6). True exactly-once across arbitrary external systems is impossible in the general case (see §8 for the honest boundary).

The key insight, verified: **at-least-once is the only semantic that is both achievable and safe for financial data without exotic machinery.** At-most-once loses data by design; exactly-once is either limited (within one broker/cluster) or simulated (idempotency). At-least-once + idempotency is the industrial default — see §2.2.

### 2.2 The Semantics Choice: At-Least-Once + Idempotency (The Default)

Given the three options, what should a payments/orders/events pipeline actually use? The industry consensus (and the recommendation this entire guide is built around) is:

> **At-least-once delivery + idempotent consumers + dead-letter queues. Always. For anything that matters.**

The reasoning, in three steps:

1. **At-most-once is disqualified** for anything financial — it *permits* loss. It survives only in metrics/audit-telemetry where a lost sample is noise.
2. **True exactly-once is expensive and bounded.** Kafka transactions give exactly-once only within one Kafka cluster (consume→process→produce on the *same* cluster, with `read_committed` consumers); SQS FIFO gives exactly-once only within its 5-minute dedup window and per message-group; and *neither* covers the final hand-off to an external database or API — the last-mile write is still at-least-once (or worse). So even "exactly-once" systems end up needing idempotency at the boundary (see §8.4 for the exact-once table's honest rows).
3. **At-least-once + idempotency covers everything.** The queue guarantees "no loss, duplicates possible"; the application makes duplicates harmless by deduplicating on a key and making the write idempotent. The cost is a dedupe store (§6) — a small, well-understood price for not having to trust a distributed transaction.

The verification status of this choice: it is the *de facto* standard across the industry — AWS's own guidance for SQS ("design your consumers to be idempotent"), the Kafka documentation's "exactly-once is for special cases, at-least-once is the default," and every serious banking integration pattern. Flag: it is a *convention*, not a formal standard — but it is the convention every broker vendor documents.

### 2.3 The Semantics Table

| Semantic | Guarantee | Trade-off | Use case |
|---|---|---|---|
| **At-most-once** | 0 or 1 deliveries | May lose messages silently; no duplicates, minimal overhead | Telemetry, metrics, logs, analytics samples, cache warming |
| **At-least-once** | 1+ deliveries; no loss | Duplicates possible; needs idempotent consumers | Payments, orders, events, anything financial — the default |
| **Exactly-once** | Effect applied exactly once | Complex/bounded: within-cluster transactions, dedup windows, or app-level idempotency | Ledger-critical flows where duplicates are catastrophic and you control both ends (Kafka Streams pipelines, SQS FIFO) |

---

## 3. The Producer-Side: Acks, Retries, Idempotent Producers

### 3.1 The Acks: Kafka's `acks` (0, 1, all)

The producer's first decision is *how much confirmation it demands before believing the broker has the message*. In Kafka this is the `acks` producer config — the single most important data-loss knob in the ecosystem (verified — the 0/1/all trichotomy is standard Kafka producer documentation):

- **`acks=0`** — fire-and-forget. The producer sends and does not wait for any acknowledgment; it does not even know the broker received the record. Maximal throughput, *guaranteed* data loss on any network hiccup or broker failure. There is no retry and no error surfaced. For anything financial: never.
- **`acks=1`** — the broker acknowledges as soon as the *leader* replica has written the record to its local log (before fsync, before replication). If the leader crashes before the followers replicate, the record is lost and the producer is not told. This is the historic default (`acks=1`); it is *at-least-once-with-a-hole*: fine for tolerant workloads, dangerous for payments.
- **`acks=all`** (alias `-1`) — the broker acknowledges only after *all in-sync replicas* have accepted the record. Combined with `min.insync.replicas` (see §4.3), this is the durable configuration: the record exists on multiple brokers before the producer moves on. If the ISR shrinks below `min.insync.replicas`, the broker *rejects* the write (`NotEnoughReplicas`) instead of silently accepting it — availability drops, data integrity holds. **This is the banking default.**

The essential mental model: `acks` buys you *knowledge* — the difference between "I hope it arrived" (`0`), "the leader has it" (`1`), and "a quorum durably has it" (`all`). Data loss happens in the gap between the acknowledgment and the actual durability; `acks=all` + `min.insync.replicas` closes that gap to the extent physics allows.

### 3.2 The Retries

Acks alone are not enough — the producer must also *retry* when a send fails. Verified producer-retry doctrine:

- **`retries`** — how many times the producer resends a record that failed transiently (network errors, leader election in progress, `NotLeaderForPartition`). With the idempotent producer enabled (below), the default is effectively unlimited retries (`Integer.MAX_VALUE` since Kafka 2.1 when idempotence is on); without idempotence, unlimited retries risk *duplicate* records (a retried send that actually committed on the first attempt is delivered twice).
- **`retry.backoff.ms`** — pause between attempts; without a backoff, retries hammer a struggling broker and make the outage worse (the classic retry-storm — see also the rate-limiter guide's backpressure discussion, [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) §1.2).
- **The retry pitfall:** retries only help if the failure is *retryable*. A permanent failure (invalid topic, oversized record, auth error) retries forever in vain — which is why a *bounded* retry policy plus a failure path (log + alert + DLQ at the consumer side, or a dead-letter topic at the producer side) is the production shape. Unbounded silent retries are how "lost" messages hide in plain sight: the producer is still retrying hours later while the business already asked "where is my payment?"

### 3.3 The Idempotent Producers: `enable.idempotence`

The producer-side answer to the retry-duplicate problem is the **idempotent producer** — `enable.idempotence=true` (verified: this is Kafka's idempotent-producer mechanism, default `true` since Kafka 3.0):

- **How it works:** each producer session gets a unique Producer ID (PID); every record carries the PID plus a monotonically increasing sequence number per partition. The broker tracks the highest sequence number it has accepted per partition; a retried record with an already-seen sequence number is recognized as a duplicate and *discarded* rather than appended twice.
- **What it guarantees:** exactly-once *delivery to the broker per partition per producer session* — no duplicate appends from retries, no out-of-order appends. It does **not** make the whole pipeline exactly-once (consumers can still see duplicates after a consumer crash — §5.3), and it does not span producer restarts (a new PID after a restart means the dedupe state resets).
- **Config interaction (verified):** enabling idempotence forces `acks=all` (the producer config validator rejects incompatible combinations) and requires `retries>0`. Since Kafka 3.0, `enable.idempotence=true` is the *default* — so modern Kafka producers are idempotent out of the box. The pragmatic rule: **leave idempotence on; it is nearly free** (a few extra bytes per record, negligible throughput cost) and it removes an entire class of silent duplicates.

### 3.4 The Producer Checklist (Verified Best Practice)

- [ ] `acks=all` — never `0`, never `1`, for anything financial.
- [ ] `enable.idempotence=true` — the default since Kafka 3.0; keep it on.
- [ ] `retries` high (with idempotence on, the default is fine) + sane `retry.backoff.ms`.
- [ ] `max.in.flight.requests.per.connection` — with idempotence on, 5 is safe (ordering is preserved by the sequence numbers); without it, keep it at 1 to avoid reordering.
- [ ] `delivery.timeout.ms` — bound the total send time so a permanently failing record eventually *surfaces* instead of retrying forever.
- [ ] A failure path for non-retryable errors: log, alert, and (for critical topics) write to a producer-side dead-letter topic.
- [ ] Monitor `record-error-rate`, `request-latency-avg`, and retry counts (§11).

### 3.5 The Producer Table

| Config | Value | Effect on data loss |
|---|---|---|
| `acks` | `0` | Fire-and-forget — loss on any failure. Never for financial data |
| `acks` | `1` | Leader-only ack — loss window if leader dies before replication |
| `acks` | `all` | All in-sync replicas ack — no ack without durability (with `min.insync.replicas`) |
| `enable.idempotence` | `true` (default ≥ 3.0) | PID + sequence numbers kill retry duplicates at the broker |
| `retries` | high / unlimited with idempotence | Transient failures retried instead of lost |
| `retry.backoff.ms` | e.g. 100–500 | Avoids retry storms during broker trouble |
| `delivery.timeout.ms` | bounded (e.g. 120s) | Permanent failures surface as errors, not infinite silent retries |
| `max.in.flight.requests` | 5 with idempotence (1 without) | Preserves order; with idempotence, safe to pipeline |

---

## 4. The Broker-Side: Replication, Durability, Election

### 4.1 The Replication: Replication Factor

The broker's core defense against losing acknowledged data is **replication** — keeping copies of every record on multiple brokers (verified: Kafka's replication-factor concept; the same principle underlies RabbitMQ quorum queues, Pulsar's bookie replication, SQS's internal 3-AZ replication):

- **Replication factor (RF)** — the number of replicas that must hold each partition/queue. RF=1 is a single point of failure: the broker disk dies, the data is gone, and *the producer was already told it was safe*. RF=3 is the standard production setting (and the standard for managed services: SQS replicates across three AZs, MSK defaults to RF=3, RabbitMQ quorum queues default to 3 replicas, Pulsar defaults to 3 bookie replicas).
- **The ISR (in-sync replicas)** — the subset of replicas that are fully caught up with the leader. Writes go to the leader and are replicated to ISR followers. A follower that lags beyond `replica.lag.time.max.ms` is dropped from the ISR — which matters because the durability contract is defined against the ISR, not against "all brokers" (see `min.insync.replicas`).
- **The replication table of truth:** with RF=3 and `min.insync.replicas=2`, a record is "safe" once the leader and one follower hold it — you can lose any *one* broker with zero data loss, and the producer was only acked after durability was real. That is the exact property a bank wants.

### 4.2 The Durability: fsync

Replication across brokers is worthless if the copies exist only in memory. **fsync** — flushing the OS page cache to physical disk — is the broker-side durability primitive (verified: Kafka's `log.flush.interval.messages` / `log.flush.interval.ms`; the general principle applies to every broker):

- A broker that acknowledges a write sitting in the page cache has acknowledged *volatile* memory: a power loss or kernel panic loses it. The broker crashed "after acking" and the record is gone — the classic invisible loss.
- Kafka's design deliberately **does not fsync on every record** — it relies on replication as the durability mechanism (if the leader dies with unflushed data, the ISR followers still have it). That is the correct division of labor: *replication covers broker crashes; fsync covers the whole-machine/power-loss case where all replicas of a partition might live on machines that die together (rare, but that's what `unclean.leader.election.enable=false` + ISR discipline exists for).*
- Managed services handle this for you (SQS, MSK, RabbitMQ MQ, Pulsar cloud all persist durably); self-managed Kafka still lets you tune flush intervals. The rule: **do not trade fsync away for throughput on a cluster that stores anything you cannot afford to lose** — and know that replication, not fsync frequency, is the primary durability mechanism in Kafka's design.

### 4.3 The min.insync.replicas

`acks=all` says "the leader must wait for all in-sync replicas" — but *who is in the ISR?* If only the leader remains in the ISR (followers lagging or down), `acks=all` degrades to `acks=1` in practice. **`min.insync.replicas` is the guardrail that prevents this silent degradation** (verified — this is exactly the documented purpose):

- **`min.insync.replicas=2`** (with RF=3) means: a write with `acks=all` succeeds only if at least 2 replicas are in sync. If the ISR shrinks to 1, the broker *rejects* writes with `NotEnoughReplicasException` instead of acking a write that only one machine holds.
- **The trade-off is availability, not data:** rejecting writes when the ISR is too small means the topic is briefly *unavailable* (producers get errors, consumers see no new data) — but no write is ever acked as durable when it isn't. The alternative — acking anyway — is how "we never lose messages" becomes "we lost messages and didn't know."
- **The classic failure this prevents:** one broker dies; its partitions' leaders move to the remaining brokers; the ISR shrinks; with `min.insync.replicas=2` the pipeline *stops* and alerts fire, and you fix the cluster. Without it, the pipeline keeps "working" with single-copy durability until the second broker dies — and then the data is gone. Availability-first teams sometimes set `min.insync.replicas=1` deliberately for non-critical topics; financial topics should never.

### 4.4 The Unclean Leader Election: The Risk

The most dangerous broker-side knob is **`unclean.leader.election.enable`** (verified — the risk is well documented across Confluent/Conduktor/AWS guidance):

- **What it does:** when the current leader dies and no in-sync replica remains (all followers lagged or died with it), the broker may elect a leader from the *out-of-sync* replicas — a replica that is missing records — rather than declaring the partition unavailable. The partition stays writable/readable... and every record the old leader had but the new leader lacks is **permanently lost, silently**, as if it never existed.
- **The trade-off:** availability vs. data integrity. `unclean.leader.election.enable=true` keeps the partition available during an extreme outage (a follower with stale data becomes leader) at the cost of *guaranteed data loss of the missing tail*. `false` (the modern default) chooses *unavailability over data loss*: the partition's leader cannot be elected, producers fail, and operators must decide how to recover (including accepting loss explicitly, with eyes open).
- **The banking rule (verified consensus):** `unclean.leader.election.enable=false`. "Unavailability over data loss" is the only defensible position for payments/orders/ledgers. If you ever see it set to `true`, someone has traded your data for uptime — usually unknowingly.

### 4.5 The Broker Checklist

- [ ] Replication factor 3 for critical topics (never 1).
- [ ] `min.insync.replicas=2` (with RF=3) so `acks=all` cannot degrade silently.
- [ ] `unclean.leader.election.enable=false` — unavailability over data loss.
- [ ] Producer `acks=all` to match (the producer and broker sides must agree on the durability contract).
- [ ] Flush/fsync settings left at sane defaults; never disable durability for critical topics.
- [ ] `replica.lag.time.max.ms` sane so lagging replicas leave the ISR promptly (a stale follower in the ISR is a false durability promise).
- [ ] Monitor ISR shrink (`IsrShrinksPerSec`), offline partitions, and under-replicated partitions — these are the leading indicators of an imminent loss event (§11).

### 4.6 The Broker Table

| Config | Recommended | Effect on data loss |
|---|---|---|
| `replication.factor` | 3 (min 2 for non-critical) | Copies survive broker loss; RF=1 is one disk away from total loss |
| `min.insync.replicas` | 2 | Writes rejected when durability can't be met — no silent degradation of `acks=all` |
| `unclean.leader.election.enable` | `false` | No out-of-sync replica can become leader — availability loss instead of data loss |
| `log.flush.interval.*` | sane defaults (don't disable) | Durable beyond the page cache; replication remains the primary crash protection |
| `replica.lag.time.max.ms` | e.g. 30s | Lagging followers exit the ISR — the ISR stays an honest durability set |
| `retention.ms` | ≥ downstream reprocessing horizon | Messages not deleted before consumers can reprocess (§5.4) |

---

## 5. The Consumer-Side: Offsets and Commit Discipline

### 5.1 The Offsets: The Consumer's Bookmark

Every consumer group tracks its **offset** — the position in the partition log up to which messages have been *committed* (verified: Kafka's offset-commit mechanism; the equivalent concept exists in RabbitMQ as message acks, in SQS as message deletion, in Pulsar as cursor/ack marks). The offset is the consumer's bookmark: on restart or rebalance, consumption resumes from the committed offset. **The offset is also where most consumer-side data loss is manufactured** — because committing an offset is a *statement about the past* that the broker trusts, whether or not the processing behind it actually succeeded.

The fundamental asymmetry, verified and worth internalizing: **the broker commits offsets; it cannot know what your application did with the message.** "Committed" means "my consumer group says it is done" — and the reliability of that claim is entirely the consumer's responsibility. This asymmetry is the root of both loss (commit before process) and duplication (process before commit, then crash) in the two patterns below.

### 5.2 The Auto-Commit: The Risk

`enable.auto.commit=true` (the Kafka default) tells the consumer to commit the current position automatically on a timer (default `auto.commit.interval.ms=5000`) — *independent of whether processing succeeded* (verified — this is the documented behavior, and the New Relic/Conduktor analyses of the failure modes are explicit):

- **The loss mode:** auto-commit fires on the poll cycle. If a message is polled, auto-commit advances past it, and the *processing* of that message then fails (crash, exception, DB down), the offset is already committed — on restart, the consumer resumes *after* the failed message. The message is gone: never processed, never retried, no error surfaced. **Silent drop at the consumer, caused by configuration.**
- **The duplicate mode:** the flip side — processing succeeds but the commit timer hasn't fired when the consumer crashes; on restart, the message is redelivered. Duplicate, not loss (and idempotency handles it, §6).
- **The verdict (verified consensus):** auto-commit is acceptable only for best-effort workloads (log collection, metrics). For anything financial, **`enable.auto.commit=false` + explicit, post-processing commit** is the at-least-once discipline. Note the subtlety: even with auto-commit off, a *synchronous* commit-after-process that fails leaves the offset uncommitted → redelivery (duplicate, safe with idempotency) — which is exactly the failure mode we want: **fail toward duplicates, never toward loss.**

### 5.3 The Manual Commits: Commit-After-Process

The at-least-once consumer pattern (verified):

1. Poll a batch of records.
2. Process each record **fully** — write to the database, call the downstream API, update state — and only then
3. Commit the offset (synchronously, per record or per batch) so the broker knows the work is done.

The invariant: **the commit must never precede the effect.** If the effect and the commit cannot be atomic (and across a DB and a broker they cannot, without exotic machinery — see §8), then the commit-after-process ordering guarantees that any crash produces *redelivery* (duplicate) rather than *loss*. Duplicates are a known, bounded, idempotency-handled problem; loss is not recoverable. This is the single most important consumer-side rule, and it is the direct consumer-side expression of "at-least-once."

The honest caveat: commit-after-process is *at-least-once on the message*, not a guarantee about the downstream effect. If the consumer's own write is not durable (DB transaction rolled back, API call dropped), the effect is lost even though the message is "processed." Closing that gap is the job of §6 (idempotency) and of making the consumer's output durable — the third loss point from §1.1.

### 5.4 The Failure Mode: Crash-After-Process-Before-Commit

The canonical consumer failure, walked through (verified — this exact scenario is the standard Kafka/SQS/Spring-Kafka documentation example):

1. Consumer polls record M (offset 42).
2. Consumer processes M successfully — the payment is booked in the ledger.
3. Consumer **crashes before committing** offset 43.
4. The group rebalances; the partition is reassigned (or the consumer restarts).
5. Consumption resumes from offset 42 — **M is redelivered and processed a second time.**

The outcome is a **duplicate effect** — a double booking, a double API call — unless the consumer is idempotent. This is not a bug in the queue; it is the *defined behavior* of at-least-once. The design consequence, verified across the industry: **you cannot design a financial consumer that never reprocesses; you can only design one that makes reprocessing harmless** (idempotency, §6) and *visible* (DLQ + metrics, §7, §11). Teams that try to eliminate duplicates by committing *before* processing (or by "remembering" processed messages in memory) have simply moved the loss from the consumer to the pipeline — the loss taxonomy's silent drop.

### 5.5 The Consumer Checklist

- [ ] `enable.auto.commit=false` for anything financial; commit **after** processing completes.
- [ ] Commit synchronously (or handle commit failure explicitly); on commit failure, stop and retry — never process-and-continue on an uncommitted offset.
- [ ] Handle rebalances: commit before losing the partition; if using cooperative rebalancing, keep the "commit only processed work" invariant.
- [ ] Make every downstream write idempotent (§6) — reprocessing after a crash is *guaranteed* to happen.
- [ ] Bound processing time vs. session timeouts (`max.poll.interval.ms`, `session.timeout.ms`) so a slow consumer is detected and rebalanced, not silently left half-dead.
- [ ] Route repeatedly-failing messages to a DLQ after bounded retries (§7) — a poison message must not block the partition forever (offset never committed → partition lag grows → everyone behind it stalls).

### 5.6 The Consumer Table

| Aspect | Setting / practice | Effect on data loss |
|---|---|---|
| `enable.auto.commit` | `false` for financial consumers | No commit-before-process → no silent loss via auto-commit timer |
| Commit timing | After processing completes | Crash ⇒ redelivery (duplicate), never loss |
| Commit mode | Synchronous, explicit | Commit failures surface instead of being swallowed |
| `max.poll.interval.ms` | Bounded (e.g. 5 min) | Dead/slow consumers detected and rebalanced |
| Processing design | Idempotent writes + dedupe (§6) | Redeliveries are harmless by construction |
| Failure handling | Bounded retries → DLQ (§7) | Poison messages don't stall partitions or hide failures |
---

## 6. The Idempotency: Dedupe Keys and the Dedupe Store

### 6.1 The Pattern: Dedupe Keys

Since at-least-once delivery *guarantees* duplicates (§2.2, §5.4), the consumer's defense is **idempotency** — making the second application of a message indistinguishable from the first. The core mechanism is the **dedupe key** (verified — this is the standard consumer-side idempotency pattern across AWS, Kafka, and messaging guidance):

- **A dedupe key is a stable, unique business identifier carried in the message** — not the message ID assigned by the broker (which changes on redelivery in some systems — e.g., SQS's `MessageId` differs for a redelivered message in a standard queue, so it must not be the dedupe key), but a business-level key the *producer* assigns once and forever.
- **Examples:** `payment_id` for a payment instruction; `order_id` for an order; `account_transaction_id` for a posting; `loan_repayment_ref` for a repayment event. The key must be the same on every retry/redelivery of the same logical event — which means it must be generated at the *source* (when the business fact happens), not at the queue boundary.
- **What it buys:** the consumer checks "have I already applied this key?" before applying the effect. If yes, it skips the write (or returns the prior result) and still commits the offset — the redelivery is absorbed.

The pattern is one half of the industry-standard pair: **outbox (producer-side, §10.1) gives every event a stable key and a durable home; inbox/dedupe (consumer-side) makes redeliveries harmless.** Together they are the "reliable messaging" stack that banks actually run.

### 6.2 The Idempotency Design: Keys and Store

The dedupe check needs a **dedupe store** — state that records which keys have been applied. Verified design guidance:

- **Where the store lives:** the same database the consumer writes to — the dedupe check and the business write must be **atomic** (one transaction: `INSERT INTO processed_events(key) VALUES (?)` + the business write, or an `INSERT ... ON CONFLICT DO NOTHING` guard). If the dedupe check and the write are two separate non-atomic steps, a crash between them recreates the duplicate. The atomic pair is the whole point — this is the "transactional inbox" pattern (see the inbox/outbox discussion in the apache_seata_guide.md transaction-patterns material and the softwaremill/AWS guidance).
- **What the store contains:** the key plus a status/result (so a replayed message can return the original outcome — "payment already booked, here is booking id 123") and a timestamp. TTL/retention: keep entries at least as long as the maximum redelivery horizon (retries + consumer downtime + reprocessing window); after that, an old duplicate could slip through — so retention should comfortably exceed the DLQ reprocessing horizon (§7).
- **Store options:** a unique-indexed table in the operational DB (standard), Redis with `SET NX EX` for high-throughput non-ledger cases (with the caveat that Redis is not the durability anchor — fine for dedupe of tolerant effects, not for money), or a Kafka compacted topic as the dedupe log. For banking: the DB table in the same transaction as the write.
- **Scaling:** the dedupe table is keyed by business key, so partitioning the consumer by key-hash keeps dedupe checks local to one consumer instance — no cross-node coordination needed.

The one caveat to flag honestly: dedupe stores are only as good as their atomicity with the business write. "Check the cache, then write" without a transaction is not idempotency — it is a race. If you cannot make the check atomic with the write, you must make the *write itself* idempotent instead (e.g., a unique constraint on the business table that makes the second insert fail benignly — `payment_id` UNIQUE in the payments table is the cheapest dedupe store that exists).

### 6.3 The Idempotency Table

| Aspect | Design | Example |
|---|---|---|
| **Dedupe key** | Stable business key set by the producer, identical across redeliveries | `payment_id = "PAY-2026-00012345"` |
| **Dedupe store** | Table with unique index on the key, in the same DB/transaction as the effect | `processed_events(key PK, status, result, processed_at)` |
| **Atomicity** | Dedupe insert + business write in one transaction | `INSERT INTO processed_events ... ; INSERT INTO payments ...` (or `ON CONFLICT DO NOTHING`) |
| **Replay behavior** | Skip effect, return stored result, commit offset | Redelivered payment → "already processed, booking_id=42" |
| **Retention** | ≥ max redelivery + reprocessing horizon | Keep dedupe rows 30+ days; DLQ reprocessing window is the floor |
| **Fallback** | Idempotent writes via unique constraints when a dedupe store is not possible | `payments.payment_id UNIQUE` — second insert fails benignly, handled as "already done" |

---

## 7. The DLQ: Dead-Letter Queues

### 7.1 The Pattern: The Dead-Letter Queue

A **dead-letter queue (DLQ)** is a separate queue/topic that receives messages the consumer could not process successfully after bounded retries (verified — the pattern is universal: SQS has native DLQ redrive, RabbitMQ has DLX/dead-letter exchanges, Kafka has no native DLQ so it is implemented as a second topic, Pulsar has `maxDeliveries`/DLQ topics):

- **Why it exists:** without a DLQ, a single poison message (malformed payload, missing reference data, a bug in the consumer, a downstream that is down for hours) blocks its partition: the consumer keeps failing, never commits the offset, lag grows, and *every message behind the poison message stalls* — the whole pipeline backs up. With a DLQ, the poison message is removed from the hot path (after bounded retries), the offset commits, the pipeline continues, and the failure becomes *visible and fixable at leisure*.
- **What it is not:** a DLQ is not a data-loss device — the message is *preserved* in the DLQ. It is a *visibility and separation* device: it trades "one bad message stalls everything silently" for "one bad message sits in a well-named, well-monitored queue that a human or an automated repair job can inspect."
- **The DLQ as the third leg:** the reliability stack is *at-least-once (no loss) + idempotency (duplicates harmless) + DLQ (failures visible and isolated)*. Remove the DLQ and the first two legs make the system *correct but blind* — a message that can never succeed (schema evolution, corrupted data) will retry forever, and no one will know.

### 7.2 The DLQ Design: Routing and Reprocessing

Verified DLQ design decisions:

- **Routing — when does a message go to the DLQ?** After a bounded retry budget: e.g., 3 attempts with exponential backoff for transient errors (downstream 503s, DB timeouts), then route. Non-retryable errors (validation failure, schema mismatch, unknown message type) should route *immediately* — retrying them is pure waste. Distinguish the two classes in the consumer; this split is what separates a good DLQ design from a lazy one.
- **Carry the failure context:** the DLQ message should wrap the original payload plus `original_topic`, `original_partition`, `original_offset`, `failure_reason`, `retry_count`, `first_failed_at`. Without this envelope, reprocessing is archaeology.
- **Reprocessing — the DLQ is a queue, not a dump:** a DLQ consumer (or a manual replay tool) reads the DLQ, applies the fix (deploy the schema fix, repair the reference data, correct the payload), and **redrives** the message back into the main topic (or reprocesses it directly) with the original dedupe key intact — so the idempotency store absorbs any overlap with already-processed messages. AWS SQS calls this "redrive" (native `StartMessageMoveTask`); Kafka shops build a small replay job; the shape is the same everywhere.
- **Alerting:** DLQ depth > 0 (or growing) must alert (§11). A silent DLQ is a shame-queue — messages rotting invisibly. The DLQ contract is *"fail loudly, preserve, and provide a path back."*

### 7.3 The DLQ Table

| Aspect | Design | Notes |
|---|---|---|
| **Trigger** | Bounded retries (e.g. 3× exponential backoff) then DLQ; immediate for non-retryable | Transient vs. permanent error split is the key design decision |
| **Envelope** | Original payload + topic/partition/offset + reason + retry count | Makes reprocessing and root-cause possible |
| **DLQ consumer** | Separate group/worker that inspects, repairs, redrives | Never block the main partition; never auto-delete |
| **Redrive** | Re-inject with original dedupe key; idempotency absorbs overlap | Repairs must be applied before redrive |
| **Monitoring** | Alert on DLQ depth > 0 / growth; track redrive success rate | A DLQ nobody watches is a data-loss vector in disguise |
| **Retention** | Long (≥ reprocessing SLA); treated as data, not garbage | DLQ contents are evidence — disputes, audits |

---

## 8. The Exactly-Once: Kafka Transactions and SQS FIFO

### 8.1 The Kafka Transactions: EOS — What It Is and Its Limitations

**Kafka transactions** (KIP-98, shipped in Kafka 0.11, 2017 — the "exactly-once semantics" / EOS feature) let a producer mark a set of records as one atomic unit: either all records in the transaction are visible to consumers (with `isolation.level=read_committed`) or none are (verified — KIP-98 is the authoritative source). Combined with the idempotent producer, this gives **exactly-once within the Kafka cluster** for the classic consume→transform→produce flow (Kafka Streams' `processing.guarantee=exactly_once_v2` is the flagship user, per KIP-129). 

**The limitations (verified — and the reason this guide treats EOS as a tool, not a religion):**

- **Within-cluster only.** Transactions guarantee atomicity of writes *to Kafka topics* — they do **not** cover the consumer's writes to a database or an external API. The last mile (the DB insert, the payment call) is still at-least-once, and the classic "exactly-once + external side effect" problem remains unsolved in general. The standard mitigation is still the transactional outbox (§10.1) or idempotency (§6) at the boundary.
- **Cost and complexity.** Transactions add a coordinator, fencing, `transactional.id` management, and latency/throughput overhead (industry measurements commonly cite single-digit-percent to ~10–20% throughput reduction and a few ms of added latency — verified via Conduktor's EOS trade-offs summary). For many pipelines the cost buys nothing that idempotency didn't already buy.
- **Configuration surface.** `transactional.id` stability across restarts, `read_committed` consumers, transaction timeouts (`transaction.timeout.ms`), and the KIP-447 (2020) scalability refinements all have sharp edges. Note on the "2020" reference some sources cite for EOS: EOS itself shipped in 2017 (KIP-98, Kafka 0.11); 2020's KIP-447 addressed *producer scalability* for transactions — the distinction matters when reading older material.
- **Verdict (flagged as a judgment call, not a fact):** use Kafka transactions for *stream-internal* exactly-once (Kafka Streams joins/aggregations where a half-applied state would corrupt downstream) — and use at-least-once + idempotency for anything whose final effect lands outside Kafka. The banking pattern is overwhelmingly the latter.

### 8.2 The SQS FIFO: Exactly-Once Processing, Bounded

**SQS FIFO queues** are AWS's exactly-once offering (verified — AWS documents FIFO as "exactly-once processing" within a message group):

- **How it works:** FIFO queues are strictly ordered per *message group ID* and deduplicate on a **message deduplication ID** (or content hash) within a **5-minute deduplication window**. A duplicate send within the window is dropped; a message is delivered to one consumer at a time per group and is not delivered again until the consumer deletes it (or the visibility timeout expires — after which it *is* redelivered, i.e., at-least-once at the edge).
- **The honest nuance (flagged):** FIFO is best described as *strict ordering + built-in producer-side dedup + single-in-flight delivery per group* — which in practice yields "each message processed once, in order," for the common case. It is not a distributed-transaction primitive: the final consumer-side effect (the DB write) still needs idempotency, and the dedup window means an old duplicate *outside* the window can slip through.
- **Costs (verified):** throughput caps (default 300 messages/s per API action with batching, 3,000/s in high-throughput mode), and the per-group single-in-flight constraint serializes processing within a group — you pay for the ordering guarantee.
- **When to choose it:** single-writer, ordered, moderate-throughput flows where the ordering + dedup combination removes the need for a custom dedupe store — e.g., a bank's per-account event stream (all events for one account = one message group). When throughput must scale beyond the caps or ordering is not needed, standard queues + idempotency is the flexible path.

### 8.3 The Exactly-Once Table

| Mechanism | Scope of exactly-once | Bounds / limitations | When to use |
|---|---|---|---|
| **Kafka transactions (EOS)** | Within one Kafka cluster (consume→produce, `read_committed`) | No coverage of external DB/API writes; ~10–20% throughput cost; complex config | Kafka Streams stateful pipelines; atomic multi-topic writes |
| **SQS FIFO** | Per message group, within 5-min dedup window | Throughput caps; single-in-flight per group; last-mile write still needs idempotency | Ordered per-key flows, moderate throughput, managed-service preference |
| **At-least-once + idempotency** | Effect exactly-once *given a correct dedupe store* | Requires app-level dedupe discipline (§6) | The default — everything not covered by the two rows above |
| **Exactly-once across arbitrary external systems** | **Impossible in general** (two-phase commit over heterogeneous systems; the classic distributed-systems result) | Any claim of global exactly-once deserves scrutiny | Never rely on it; design for at-least-once + idempotency |

### 8.4 The Honest Boundary: Why Cross-System Exactly-Once Is Impossible

Worth stating once, plainly, because "exactly-once" is the most abused phrase in messaging: **no message queue can guarantee that the effect of a message on a system outside the queue happens exactly once.** The reason is the classic distributed-systems impossibility result (the two-generals problem / the fundamental limit of atomic commit over an unreliable network): for the queue to know the external write succeeded, the external system must acknowledge; for the external system's ack to be trustworthy, *it* must be durably recorded; and the ack itself can be lost in transit, forcing a retry — which re-executes the write, or leaves the queue guessing. Every "exactly-once" mechanism you will meet is a *bounded* version of this problem:

- **Within one system** (Kafka transactions within one cluster; SQS FIFO within its dedup window) the queue can hold both ends of the transaction and enforce atomicity — because there is no external party to coordinate with.
- **Across systems** (queue + database + API), the only honest designs are: (a) make the external write *idempotent* so retries are harmless (§6), or (b) make the queue a *projection* of the authoritative store (outbox/event sourcing, §10) so a lost message is re-emitted rather than re-created.

This is why the guide's recommendations never say "use exactly-once"; they say "use at-least-once, make duplicates harmless, and make loss impossible at the source." The engineering community that claims otherwise is selling a boundary condition as a universal.

---

## 9. The Broker Guarantees: SQS, RabbitMQ, Pulsar, Kafka

### 9.1 The SQS: Standard (At-Least-Once) and FIFO (Exactly-Once-Processing)

**SQS standard queues** deliver **at-least-once** (verified — AWS documentation is explicit: "Standard queues provide at-least-once delivery"; duplicates are possible, ordering is best-effort). The consumer must therefore be idempotent — AWS's own guidance says exactly that. **SQS FIFO queues** provide exactly-once *processing* within a message group (deduplication window, strict ordering — §8.2). Configuration notes: SQS is a managed service — durability (replication across three AZs) and retention (default 4 days, up to 14) are AWS's contract; the *consumer-side* reliability (visibility timeouts, DLQ redrive, idempotent handling) is the operator's job. The visibility timeout is the SQS-specific reliability knob: too short ⇒ premature redelivery (duplicates); too long ⇒ slow failure detection.

### 9.2 The RabbitMQ: Publisher Confirms + Consumer Acks

**RabbitMQ** achieves at-least-once when *both* sides use acknowledgements (verified — RabbitMQ's "confirms" and "consumer acknowledgements" documentation, and the DeepWiki/RabbitMQ-docs analyses):

- **Publisher confirms** (`channel.confirmSelect()` + waiting for `basic.ack`): the broker confirms a message only after it has accepted it for routing/persistence. A producer that treats "no confirm" as "retry" closes the producer-side loss window. (RabbitMQ also has a **mandatory** flag + `basic.return` for unroutable messages — the "the message matched no queue" case, a silent-loss vector many teams miss.)
- **Consumer acks** (manual `basic.ack` after processing): the message is deleted from the queue only when the consumer acks. No ack (crash, `basic.nack`/reject) ⇒ redelivery (at-least-once). Auto-ack mode (`autoAck=true`) is the at-most-once foot-gun.
- **Durability stack:** durable queues + persistent messages (`delivery_mode=2`) so messages survive broker restarts; **quorum queues** (RabbitMQ 3.8+, 2020) replicate across nodes and survive node loss — the equivalent of Kafka's replication factor. Without quorum queues (or mirrored/classic queue replication), a single-node RabbitMQ is a single point of failure.

### 9.3 The Pulsar: At-Least-Once by Default — Flagged

**Apache Pulsar** delivers **at-least-once** for persistent topics by default (verified via Pulsar documentation/community sources; flagged: the nuance below is where the claims get fuzzy):

- Persistent topics + producer acks + consumer acknowledgements (with ack-timeout redelivery and negative acks) give at-least-once; Pulsar's ledger-based storage (BookKeeper) replicates (default 3) and survives broker/bookie failure — durability is built-in rather than configured.
- **Exactly-once-ish:** Pulsar transactions (matured ~2022) provide *effectively-once* semantics for consume→produce flows within Pulsar — the same "within the system" boundary as Kafka EOS.
- **At-most-once exists too:** *non-persistent* topics are explicitly at-most-once (message loss on broker failure by design) — the trap is assuming every Pulsar topic is durable.
- **Flagged:** Pulsar's delivery-guarantee documentation is less crisply canonical than Kafka's or SQS's, and semantics depend on the ack model (cumulative vs. individual acks, ack timeout behavior) — verify against your Pulsar version's docs before banking on a specific claim.

### 9.4 The Guarantee Table

| Broker | Default guarantee | Config/mechanism that makes it real | Loss vectors to close yourself |
|---|---|---|---|
| **Kafka** | At-least-once | `acks=all` + `enable.idempotence` + `min.insync.replicas` + RF≥3 + no unclean election | Consumer commit discipline (§5), dedupe (§6), DLQ (§7) |
| **SQS standard** | At-least-once (managed) | AWS-replicated storage; visibility timeout; DLQ redrive | Idempotent consumers; don't use `MessageId` as dedupe key |
| **SQS FIFO** | Exactly-once processing per group | Dedup ID + message group + 5-min window | Last-mile writes; throughput caps |
| **RabbitMQ** | Depends on config (at-least-once with confirms+acks) | Publisher confirms, manual consumer acks, persistent messages, quorum queues | Unroutable messages (`mandatory` flag); auto-ack consumers; single-node deployments |
| **Pulsar** | At-least-once (persistent topics) | Producer acks + consumer acks, BookKeeper replication (default 3), transactions for effectively-once | Non-persistent topics are at-most-once; ack-model nuances (flagged) |
| **Redis Streams / NATS (light mention)** | At-least-once-ish, config-dependent | Consumer groups + acks (Redis); JetStream with replication (NATS) | Smaller guarantees surface; verify per version — see [kafka_alternatives_guide.md](kafka_alternatives_guide.md) for the full landscape |

The cross-reference: [kafka_alternatives_guide.md](kafka_alternatives_guide.md) compares brokers on features and operations; this table adds the *reliability contract* axis — the guarantees each broker actually makes and the knobs that make them real.

---

## 10. The Design Patterns: Transactional Outbox and Event Sourcing

### 10.1 The Transactional Outbox: The Dual-Write Fix

The **transactional outbox pattern** solves the **dual-write problem**: an application that must (a) commit a business change to its database *and* (b) publish an event to a message queue cannot do both atomically. Whatever order it tries — DB first, then publish (crash ⇒ event lost; the classic "the DB has the payment, the queue doesn't"); publish first, then DB (crash ⇒ phantom event for a payment that doesn't exist) — a crash between the two leaves the system inconsistent. Verified: this is the problem the pattern exists to solve, and the pattern is documented on microservices.io (Chris Richardson) and in AWS Prescriptive Guidance as the standard fix.

**The pattern (verified):**

1. Write the event to an **outbox table in the same database transaction** as the business change: `BEGIN; INSERT INTO payments(...); INSERT INTO outbox(event_id, payload, status); COMMIT;` — the business fact and the event are now atomic.
2. A **relay** (a poller that scans `outbox WHERE status='pending'`, or a CDC stream — Debezium-style change data capture on the outbox table) publishes the events to the broker, then marks them `published`.
3. The relay retries until the broker acks; the outbox row is the *source of truth* for "events that still must go out" — the broker can lose the message, and the relay simply re-publishes it.

**Why it is the producer-side anchor of this guide:** it moves the producer's durability problem from "hope the network works" to "the event is durable in my own database the moment the business fact is" — the outbox is a *broker-independent* guarantee. Every event also gets a stable `event_id` (the dedupe key for consumers, §6). Origin note, flagged: the pattern is widely credited to Chris Richardson's microservices.io catalog (mid-2010s, c. 2015–2016, with the 2018 *Microservices Patterns* book as the canonical written reference); I could not pin down a specific single "2015 blog post" origin in my research — treat "popularized by Richardson, mid-2010s" as the safe citation.

### 10.2 The Event Sourcing: The Ledger of Facts

**Event sourcing** stores the *events* — the immutable facts — as the system of record, and derives current state by replaying them (verified: the standard definition, per Fowler's "Event Sourcing" write-up and the DDD/microservices literature). In reliability terms it is the natural *complement* to messaging: the message queue *transports* events; event sourcing makes the events themselves the durable, replayable source of truth. If the queue loses a message, the event-sourced store still has the fact and can re-emit it — the queue becomes a *projection* of the store rather than the only copy.

For banking, event sourcing is a heavy architectural decision (see the banking guides: [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md), [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md) — the posting/interest engines are natural event-sourced domains: a posting is a fact, the balance is a projection). The cross-reference to make here: **event sourcing + outbox** is the strongest producer-side data-loss posture that exists — facts are atomic with the ledger (outbox) and the ledger is a replayable event log (sourcing) — and it composes with at-least-once + idempotency downstream. The cost: schema evolution, snapshotting, replay tooling, and a cultural shift to "events are the data."

### 10.3 The Pattern Table

| Pattern | Problem it solves | Solution | Reliability contribution |
|---|---|---|---|
| **Transactional outbox** | Dual-write: DB commit and event publish can't be atomic | Event written to outbox table in the same DB transaction; relay publishes + marks done | Producer-side: no event published without its business fact durably existing; stable `event_id` for dedupe |
| **Inbox (consumer-side dedupe)** | At-least-once redelivery duplicates effects | Dedupe table in the consumer's DB, atomic with the write (§6) | Consumer-side: duplicates harmless |
| **Event sourcing** | Current state is a lossy projection; the past is recoverable | Facts stored as immutable event log; state derived by replay | The event log is the durable source of truth — queue loss is recoverable by re-emission |
| **Saga / distributed transactions** | Multi-service consistency | Choreographed/orchestrated compensation (see [apache_seata_guide.md](apache_seata_guide.md)) | Reliability *across* services; messaging is the transport — messages must still be at-least-once + idempotent |
| **CDC (change data capture)** | Capturing DB changes as events | Debezium-style log-based capture; powers outbox relays | No dual-write at all when the DB log itself is the event source |

### 10.4 The Outbox Implementation Variants

The outbox pattern has three well-established implementation variants (verified across the microservices.io and AWS guidance) — the choice is an operational one, not a semantic one:

- **Poller (query-based).** A scheduled job selects `outbox WHERE status='pending' ORDER BY id` (with `FOR UPDATE SKIP LOCKED` to avoid duplicate dispatch between poller instances), publishes, and marks `published`. Simplest to build and reason about; adds a polling interval of latency and needs an index on `(status, id)` to stay fast. The 2015–2016-era default.
- **Transaction-log tailing (CDC).** Debezium (or the platform's CDC) streams the outbox table's inserts from the database's own transaction log — no polling, sub-second latency, and the relay cannot miss rows the poller's query might skip. Requires CDC infrastructure and careful handling of the log-position watermark (the "transaction log tailing" pattern in the microservices.io catalog).
- **Dual-write elimination (event-first).** Some teams go further and make the *database log itself* the event source (the outbox *is* the DB's CDC stream, with no separate table) — the strongest form, at the cost of coupling the event contract to the DB schema.

All three converge on the same property: the event is durable the moment the business fact is, and the relay retries until the broker confirms. The implementation note worth flagging: whichever variant you choose, the outbox row must carry the stable `event_id` (the consumer's dedupe key, §6.1) and the relay must be idempotent about publishing (broker-side duplicates from relay retries are absorbed by the consumer's dedupe store — the layers compose).

---

## 11. The Monitoring: Lag, Retries, DLQ Depth

### 11.1 The Lag: The Primary Health Signal

**Consumer lag** is the difference between the newest offset in a partition and the offset the consumer group has committed — the number of messages waiting to be processed (verified: the standard Kafka metric; MSK/Confluent/Burrow all center on it). Lag is the single most informative reliability metric in a messaging pipeline:

- **Lag growing ⇒ the consumer cannot keep up** (slow processing, downstream blocked, a poison message stalling the partition because the offset never commits). Lag *exploding on one partition* while others are fine ⇒ a partition-level problem (hot key, poison message, rebalance issue).
- **Lag near zero ⇒ the pipeline is keeping up** — but beware the false negative: zero lag means "consumed up to the committed offset," *not* "all effects applied." Lag is a *throughput* signal, not a *correctness* signal; correctness is checked by reconciliation (§11.4).
- **Tooling (verified):** `kafka-consumer-groups --describe --group <g>` for an instant snapshot; Burrow (LinkedIn's lag evaluator) or kafka_exporter + Prometheus for continuous monitoring; MSK/Confluent Cloud expose lag natively. Alert on **per-partition** max lag, not the group average — an average hides a single stalled partition (verified: Conduktor's guidance is explicit — "aggregated alerts hide problems"; alert when *any* partition exceeds the threshold).

### 11.2 The Retries: The Hidden Failure Signal

Retries are the pipeline's *subclinical* symptom: every retry is a failure that almost happened. Monitor:

- **Producer retry rate / record-error-rate** — rising retries ⇒ broker trouble, network trouble, or a misconfigured `acks` contract; sustained retries with delivery-timeout errors ⇒ messages *are* being dropped at the producer (§3.4).
- **Consumer redelivery rate** — rising redeliveries ⇒ processing instability (crashes, timeouts, commit failures); a redelivery *storm* is the first sign of the crash-after-process-before-commit loop from §5.4.
- **Commit failures** — a consumer that cannot commit is a consumer that is about to reprocess everything since the last commit.

The pattern to internalize: **retries are the warning light, errors are the engine failure.** Alert on retry-rate trends long before the errors start.

### 11.3 The DLQ Depth and Reconciliation

- **DLQ depth > 0** must alert immediately (§7.2). A non-empty DLQ is the queue saying "I have messages I cannot process" — every hour it sits is an hour the business fact is delayed. Alert on growth rate, not just absolute depth; track redrive success.
- **End-to-end reconciliation** is the only true data-loss detector (verified as best practice across banking event-pipeline literature): periodically compare the count/checksum of business facts in the source system (the outbox table, the event-sourced ledger) against what the queue delivered and what consumers applied. A mismatch of one in a million is invisible to lag and retries — reconciliation finds it. For payments, this is the *control/GL reconciliation* function every bank already runs; the message-queue version is the same discipline applied to the event backbone.

### 11.4 The Monitoring Table

| Metric | Alert when | Action |
|---|---|---|
| **Consumer lag (per partition)** | Any partition's lag > threshold (e.g. 10k messages, or age > SLA) | Scale consumers, investigate the stalled partition, check for poison messages → DLQ (§7), check downstream health |
| **Producer retry rate / errors** | Sustained retries; `record-error-rate` > 0 over window | Check broker health/ISR, network, config contract (`acks`/idempotence), delivery-timeout failures |
| **Consumer redelivery rate** | Rising trend / storm | Find crash-looping consumer, commit-failure causes, fix then rely on idempotency to absorb duplicates |
| **Commit failures** | Any | Fix consumer commit path; expect reprocessing (idempotency absorbs) |
| **DLQ depth** | > 0 or growing | Inspect, repair, redrive (§7.2); root-cause the poison message class |
| **ISR shrink / offline partitions** | `IsrShrinksPerSec` > 0, under-replicated partitions > 0 | Broker capacity/network; the durability contract is at risk (§4) |
| **Reconciliation delta** | Any mismatch source-vs-queue-vs-applied | End-to-end data-loss hunt; replay from source of truth (§10) |

---

## 12. The Worked Example: A Banking Payment Pipeline

### 12.1 The Scenario: Payment Events

Meridian Bank (the recurring worked-example bank of this research series — see [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) §11 and the banking guides) runs a **real-time payment pipeline**: the payments API accepts a payment instruction, books it in the core ledger, and emits payment events (`PaymentCreated`, `PaymentAuthorized`, `PaymentSettled`, `PaymentRejected`) to Kafka for downstream consumers — fraud detection ([banking/financial_fraud_detection_at_scale_guide.md](banking/financial_fraud_detection_at_scale_guide.md)), notifications, reconciliation, and the regulatory reporting feed. The requirement is absolute: **a payment event must never be lost** — a lost `PaymentSettled` means a settled payment that the reporting system, the customer notification, and the reconciliation engine never saw. Duplicates are tolerable only if harmless (double-notification is annoying; double-booking is not).

The pipeline: `payments API → payments DB (ledger) → Kafka (payment-events topic) → consumers (fraud, notify, reconcile) → downstream stores`.

### 12.2 The Design: At-Least-Once + Idempotency Keys + DLQ

The design applies this guide's stack end-to-end:

1. **Producer side — transactional outbox (§10.1).** The payments API writes the payment row *and* the `PaymentCreated` event into an outbox table in **one DB transaction**. A relay (CDC on the outbox table) publishes to the `payment-events` topic with the stable `payment_id` as the message key and a global `event_id`. The outbox is the durability anchor: even if Kafka is down for an hour, no event is lost — the relay catches up.
2. **Kafka configuration — the §3/§4 contract.** Producers use `acks=all`, `enable.idempotence=true`, bounded `delivery.timeout.ms`. The topic has `replication.factor=3`, `min.insync.replicas=2`, `unclean.leader.election.enable=false`. The ISR floor of 2 means the pipeline *stops* (producer errors, alert fires) rather than acking single-copy writes — availability risk accepted, data integrity guaranteed.
3. **Consumer side — commit-after-process (§5).** Every consumer (fraud, notify, reconcile) runs `enable.auto.commit=false`, processes each message fully (writes to its own store), and commits *after*. A crash mid-batch ⇒ redelivery ⇒ duplicate — absorbed by step 4.
4. **Idempotency (§6).** Every consumer keeps a `processed_events(event_id PK, status, result)` table; the dedupe insert and the business write are **one transaction** (for the ledger-adjacent consumers, the `payment_id` unique constraint in the payments table is itself the dedupe store). Redelivered `PaymentSettled` → dedupe hit → return stored result, commit, move on.
5. **DLQ (§7).** After 3 retries with exponential backoff (transient: downstream 503, DB contention) — or immediately for non-retryable (schema mismatch, unknown payment type) — the consumer routes to `payment-events-dlq` with an envelope carrying `payment_id`, `event_id`, `original_topic/partition/offset`, `failure_reason`, `retry_count`. The DLQ alerts on depth > 0; the ops runbook says: inspect → fix (repair reference data, deploy schema fix) → **redrive with the original `event_id`** → idempotency absorbs anything already applied.

### 12.3 The Configs: acks=all, min.insync, Replication (The Sketch)

The concrete config sketch (Kafka, producer + topic + consumer):

```properties
# Producer (payments API / outbox relay)
acks=all                      # §3.1 — no ack without all in-sync replicas
enable.idempotence=true       # §3.3 — PID + sequence numbers kill retry duplicates
delivery.timeout.ms=120000    # §3.4 — bound the retry horizon; surface permanent failures
retry.backoff.ms=200

# Topic (payment-events)
replication.factor=3          # §4.1 — survive two-broker loss of any one copy
min.insync.replicas=2         # §4.3 — reject writes when durability can't be met
unclean.leader.election.enable=false   # §4.4 — unavailability over data loss
retention.ms=1209600000       # 14 days — replay/reconciliation horizon

# Consumer (fraud, notify, reconcile)
enable.auto.commit=false      # §5.2 — commit-after-process only
max.poll.interval.ms=300000
isolation.level=read_committed  # if using transactions anywhere in the flow (§8.1)
```

The shape to defend in any design review: **acks=all + min.insync.replicas=2 + RF=3 + idempotent producer + commit-after-process + dedupe store + DLQ + lag/DLQ/reconciliation monitoring.** Every piece closes one of the three loss points from §1.1; the stack is the point.

### 12.4 The Lessons: "At-Least-Once + Idempotency + DLQ"

- **The message can always be redelivered; design for it.** Every consumer in the pipeline is written as if every message arrives twice. The double-booking incident Meridian avoided would have come from a consumer that assumed exactly-once.
- **The outbox is what makes "the queue lost it" recoverable.** When Kafka lost a batch in a pre-production drill (leader killed mid-write), the relay re-published from the outbox and the pipeline self-healed — no manual replay, no reconciliation break. The queue is a projection; the outbox is the truth.
- **DLQ discipline is the difference between an incident and a non-event.** A malformed payload from a legacy integration (a `PaymentCreated` with a null currency) used to stall the notify consumer's partition for hours. Now: 3 retries, DLQ, alert, fix the producer, redrive — the partition never blocked, the failure was visible in minutes.
- **The config is the contract.** The single most dangerous moment in the project was a proposal to set `acks=1` "for throughput." The design review killed it by pointing at §3.1's loss window: one leader crash between ack and replication, and a settlement event is gone with no error anywhere. `acks=all` + `min.insync.replicas=2` is non-negotiable for this topic class.
- **Monitoring closes the loop.** Lag alerts caught a throttled consumer; the DLQ alert caught the null-currency producer; the reconciliation check (outbox count vs. applied count, run nightly) is the final backstop that would catch any residual loss the other layers missed.

### 12.5 The Failure Drills: What the Stack Actually Did

The design was validated with failure drills before go-live — worth summarizing because each drill maps one-to-one to a loss mode from §1.3 and a mechanism from the stack:

- **Drill 1: "Kill the leader mid-write."** An engineer killed the partition leader during a load test. With `acks=all` + `min.insync.replicas=2`, in-flight writes were rejected (producers retried), a follower took over, and **zero events were lost** — the outbox relay simply re-sent the few events whose acks never arrived, and consumer dedupe absorbed the re-sends. The measured outcome: a blip in producer error metrics, no reconciliation delta.
- **Drill 2: "Crash a consumer after processing, before commit."** A notify consumer was SIGKILLed mid-batch. On restart, the batch was redelivered; the dedupe store (transactional with the notification write) recognized the `event_id`s, skipped the notifications, committed. Outcome: **duplicates delivered to the store, zero duplicate effects** — the idempotency row of §1.5 doing its job.
- **Drill 3: "Inject a poison message."** A null-currency `PaymentCreated` was published directly to the topic. The consumer retried 3× (transient-error budget), classified the failure as non-retryable, routed to `payment-events-dlq` with the envelope. The partition kept flowing; the alert fired; ops repaired the producer and redrove the message after fixing the payload. Outcome: **one visible, isolated failure instead of a stalled partition and a silent backlog.**
- **Drill 4: "The audit."** The nightly reconciliation (outbox rows published vs. events applied across consumers, keyed on `event_id`) ran a zero-delta check against the drill artifacts. This is the layer that would have caught any *silent* loss the other mechanisms missed — and the reason the pipeline owners sleep at night.

The meta-lesson, stated as the guide's thesis: **each drill's outcome was determined in advance by configuration, not by heroics.** The stack converted every failure mode into either a retry, a harmless duplicate, or a visible DLQ entry — which is the entire definition of a data-loss-proof pipeline.

---

## 13. The Summary: Reliability in One Page

### 13.1 The One-Page Reliability Model

**The problem** (from §1): a message crosses three ownership boundaries — producer→broker, broker→consumer, consumer→downstream — and each boundary is a place where a message can silently vanish (silent drop, crash loss, or the duplicate that behaves like loss).

**The semantics** (from §2): three delivery contracts exist — at-most-once (loss by design), at-least-once (no loss, duplicates possible), exactly-once (bounded: within-cluster transactions, dedup windows, or simulated). The only sane default for financial data: **at-least-once + idempotency.**

**The configs** (from §3–§5) — the mechanical core:

- *Producer:* `acks=all`, `enable.idempotence=true`, bounded retries with backoff — the producer never believes a message is safe until a quorum durably has it.
- *Broker:* RF=3, `min.insync.replicas=2`, `unclean.leader.election.enable=false` — replication covers crashes; the ISR floor refuses to ack fragile writes; no out-of-sync replica can ever become leader.
- *Consumer:* `enable.auto.commit=false`, commit **after** processing, idempotent writes — failures produce redelivery (safe), never loss.

**The patterns** (from §6–§10):

- *Idempotency:* stable business keys (producer-assigned `event_id` / `payment_id`) checked atomically in a dedupe store with the business write — duplicates become no-ops.
- *DLQ:* bounded retries, then route poison messages to a monitored, replayable dead-letter queue — failures become visible and isolated instead of stalling partitions silently.
- *Outbox:* events written in the same DB transaction as the business fact, relayed to the broker — the producer-side guarantee becomes "durable in my DB," not "hope the network worked."
- *Monitoring:* per-partition lag, retry rates, DLQ depth, and end-to-end reconciliation — the pipeline's health signals, with reconciliation as the only true loss detector.

### 13.2 The Final Word: "At-Least-Once + Idempotency + DLQ"

Every mechanism in this guide collapses into one sentence:

> **"At-least-once + idempotency + DLQ" — accept that messages will be redelivered, make redelivery harmless, and make failures visible and recoverable.**

At-least-once is the delivery contract that refuses to lose data. Idempotency is the application contract that refuses to let redelivery corrupt state. The DLQ is the operational contract that refuses to let failure hide. Exactly-once machinery (Kafka transactions, SQS FIFO) earns its complexity only in narrow within-system cases; for the last mile — the database write, the API call, the payment itself — there is no substitute for the three-part stack. Configure the queue for durability, discipline the consumer toward duplicates-not-loss, dedupe on stable keys, dead-letter the poison, monitor the lag, and reconcile the totals. Do that, and "the message queue lost my message" becomes a phrase you never hear.

---

## 14. Verification and Claims-Status

This guide's factual claims were verified against primary documentation and industry sources during research (August 2026). Status per claim cluster:

| Claim | Status | Source |
|---|---|---|
| Delivery semantics tripartite (at-most-once / at-least-once / exactly-once) | **Verified** — standard classification in Kafka, AWS, and RabbitMQ docs | Apache Kafka docs, AWS SQS docs, RabbitMQ docs |
| Kafka `acks` 0/1/all semantics | **Verified** | Apache Kafka producer configuration docs |
| `enable.idempotence=true` default since Kafka 3.0; PID + sequence numbers; forces `acks=all` | **Verified** | Apache Kafka docs; Kafka 3.0 release notes; Data Vidhya producer-semantics analysis |
| `min.insync.replicas` rejects writes when ISR too small | **Verified** | Apache Kafka docs; Stack Overflow/Conduktor/AutoMQ analyses |
| `unclean.leader.election.enable` data-loss risk; default `false` in modern Kafka | **Verified** (risk); default value stated conservatively — check broker version | Conduktor pitfalls, Kafka docs |
| Kafka transactions = KIP-98, Kafka 0.11 (2017); within-cluster scope; throughput cost ~10–20% (industry-reported, varies) | **Verified** | KIP-98 (cwiki.apache.org), Conduktor EOS glossary, KIP-447 (2020, scalability — flagged as the likely source of the "2020" citation) |
| SQS standard = at-least-once; FIFO = exactly-once processing within dedup window (5 min) and message group; throughput caps (300/3,000 msg/s) | **Verified** | AWS SQS FIFO developer guide |
| RabbitMQ publisher confirms + consumer acks → at-least-once; quorum queues (3.8+, 2020) | **Verified** | RabbitMQ confirms docs, DeepWiki analysis |
| Pulsar: persistent topics at-least-once; non-persistent at-most-once; transactions ~2022 effectively-once | **Partially verified — flagged.** Pulsar's guarantee documentation is less crisply canonical; ack-model nuances vary by version | Pulsar docs, community analyses (Medium/PhoenixAI) — treat specifics with version checks |
| Auto-commit (`enable.auto.commit=true`) risks: offset committed on poll timer independent of processing → silent loss; crash-after-process-before-commit → duplicates | **Verified** | New Relic Kafka consumer analysis, Spring Kafka/Stack Overflow consensus, Kafka docs |
| Transactional outbox pattern solves dual-write; outbox table in same DB transaction + relay | **Verified** (pattern); **origin flagged** — credited to Chris Richardson's microservices.io catalog (mid-2010s, c. 2015–2016; 2018 book); a specific single "2015 blog post" origin could not be confirmed in research | microservices.io, AWS Prescriptive Guidance, softwaremill |
| DLQ pattern: bounded retries → DLQ with envelope → redrive; SQS native redrive | **Verified** | AWS SQS DLQ/redrive docs, industry practice |
| Consumer-lag monitoring; alert per-partition not average; Burrow/kafka_exporter tooling | **Verified** | Confluent/MSK monitoring docs, Conduktor alerting guidance |
| "At-least-once + idempotency + DLQ" as the industry default for reliable messaging | **Verified as convention** (not a formal standard) — consistent with AWS, Kafka, and RabbitMQ guidance | Vendor guidance, this guide's synthesis |

**Flagged/unverifiable items:** (1) the specific "2015 blog" origin of the transactional outbox pattern; (2) the "2020" date sometimes attached to Kafka EOS (EOS = 2017 KIP-98; 2020 = KIP-447 scalability work — distinct); (3) Pulsar delivery-guarantee nuances; (4) exact performance overhead percentages of Kafka transactions (vendor-measured, environment-dependent); (5) the version at which `unclean.leader.election.enable` defaulted to `false` (state conservatively: modern Kafka defaults to `false`).

### 14.1 References (Consulted During Research, August 2026)

- **Kafka producer semantics / idempotence:** Data Vidhya — *Producer Delivery Semantics* (datavidhya.com); Apache Kafka documentation — *Producer Configs* (`acks`, `enable.idempotence`, `retries`).
- **Kafka transactions / EOS:** KIP-98 — *Exactly Once Delivery and Transactional Messaging* (cwiki.apache.org); KIP-447 — *Producer scalability for exactly once semantics*; Conduktor — *Exactly-Once Semantics in Kafka* (trade-offs: latency/throughput); Akka Alpakka Kafka — *Transactions* (same-cluster scope).
- **Kafka broker durability:** Conduktor — *Kafka in Production: 11 Pitfalls* (unclean leader election); AutoMQ — *Kafka Replication: Concepts & Best Practices*; Stack Overflow — *min.insync.replicas vs. unclean.leader.election*.
- **SQS:** AWS SQS Developer Guide — *FIFO queues* (exactly-once processing, dedup window, TPS limits); AWS Prescriptive Guidance — *Transactional outbox pattern*.
- **RabbitMQ:** RabbitMQ documentation — *Consumer Acknowledgements and Publisher Confirms*; DeepWiki rabbitmq-demo — *At Least Once Delivery*.
- **Pulsar:** Pulsar documentation and community analyses (persistent vs. non-persistent topics; effectively-once via transactions) — flagged per §9.3.
- **Consumer offsets:** New Relic — *Kafka consumer auto-commit: data loss and duplication*; Spring Kafka / Stack Overflow consensus on manual commits; Kafka docs on `enable.auto.commit`.
- **Lag monitoring:** Confluent Cloud — *Monitor Kafka Consumer Lag*; AWS MSK — *Monitor consumer lags*; Conduktor — *Kafka Consumer Lag Alert Thresholds* (per-partition alerting); Burrow / kafka_exporter.
- **Outbox pattern:** microservices.io — *Pattern: Transactional outbox* (Chris Richardson); AWS Prescriptive Guidance; softwaremill — *Microservices 101: Transactional Outbox and Inbox*.
- **Domain cross-references (this repo):** [event_stream_processing_guide.md](event_stream_processing_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md), [apache_seata_guide.md](apache_seata_guide.md), [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md), [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md), [banking/financial_fraud_detection_at_scale_guide.md](banking/financial_fraud_detection_at_scale_guide.md), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md).

---

## 15. Glossary

- **Message queue** — a broker-mediated transport that decouples producers from consumers, holding messages until consumers acknowledge them. Includes classic queues (RabbitMQ, SQS), stream/log brokers (Kafka, Pulsar), and their managed variants.
- **Data loss** — a message (or its effect) that is acknowledged as handled but never durably stored or applied — the sum of silent drops and crash losses (§1).
- **At-most-once** — delivery semantic: each message delivered zero or one times; loss permitted, duplicates impossible (§2.1).
- **At-least-once** — delivery semantic: each message delivered one or more times; no loss, duplicates possible — the default for serious brokers (§2.1).
- **Exactly-once** — delivery semantic: each message's effect applied exactly once; achievable only within bounded scopes (single cluster, dedup windows) or simulated via idempotency (§2.1, §8).
- **Delivery semantics** — the contract between producer, broker, and consumer about message fate across failures (§2).
- **Producer** — the component that sends messages to the broker (§3).
- **Broker** — the server(s) that receive, persist, and serve messages (§4).
- **Consumer** — the component that reads and processes messages (§5).
- **Acks** — acknowledgements the broker sends the producer; Kafka's `acks` config (0 = none, 1 = leader only, all = all in-sync replicas) sets the durability bar for a write (§3.1).
- **Retries** — the producer's (and consumer's) resend of a failed operation; with idempotence, retries are duplicate-safe (§3.2).
- **Idempotent producer** — a producer that tags records with a PID + sequence numbers so the broker can discard retried duplicates; Kafka `enable.idempotence` (§3.3).
- **enable.idempotence** — Kafka producer config that activates idempotent-producer deduplication; default `true` since Kafka 3.0 (§3.3).
- **Replication factor** — the number of copies of each partition/queue held across brokers; RF=3 is the production standard (§4.1).
- **fsync** — flushing data from OS memory to physical disk; the durability primitive behind a broker's "durable write" claim (§4.2).
- **min.insync.replicas** — the minimum number of in-sync replicas required for an `acks=all` write; below it, writes are rejected rather than weakly acked (§4.3).
- **Unclean leader election** — electing a leader from out-of-sync replicas; keeps availability but permanently loses the missing records — disable for financial data (§4.4).
- **Offset** — a consumer group's committed position in a partition log; the bookmark from which consumption resumes (§5.1).
- **Auto-commit** — committing offsets on a timer independent of processing; the silent-loss foot-gun (§5.2).
- **Manual commit** — explicit, post-processing offset commit; the at-least-once consumer discipline (§5.3).
- **Duplicate** — a message delivered/processed more than once; guaranteed by at-least-once, neutralized by idempotency (§2.1, §5.4).
- **Idempotency** — the property that applying an operation twice equals applying it once; achieved via dedupe keys and stores (§6).
- **Dedupe key** — a stable, producer-assigned business identifier used to detect and absorb redeliveries (§6.1).
- **DLQ / dead-letter queue** — a separate queue receiving messages that failed after bounded retries; makes failures visible and isolated, with a redrive path (§7).
- **Reprocessing** — redriving DLQ messages (or replaying retained messages) after a fix; idempotency makes it safe (§7.2).
- **Kafka transactions** — Kafka's EOS feature (KIP-98, 2017): atomic multi-topic produce/consume within one cluster with `read_committed` consumers (§8.1).
- **EOS** — exactly-once semantics; the umbrella term for Kafka's transactional machinery (§8.1).
- **SQS** — Amazon Simple Queue Service; standard queues (at-least-once) and FIFO queues (exactly-once processing per message group) (§8.2, §9.1).
- **FIFO** — SQS's first-in-first-out queue type: strict ordering, dedup window, single-in-flight per message group (§8.2).
- **RabbitMQ** — a classic AMQP message broker; at-least-once via publisher confirms + manual consumer acks + quorum queues (§9.2).
- **Publisher confirms** — RabbitMQ's broker acknowledgment to producers; "no confirm ⇒ retry" closes the producer loss window (§9.2).
- **Pulsar** — Apache's unified streaming/messaging platform; at-least-once for persistent topics, BookKeeper replication, effectively-once transactions (§9.3).
- **Outbox pattern / transactional outbox** — writing events to a DB table in the same transaction as the business change, then relaying them to the broker; the dual-write fix (§10.1).
- **Event sourcing** — storing immutable facts/events as the system of record, deriving state by replay; the queue becomes a projection of the ledger (§10.2).
- **Lag** — the gap between the newest offset and the consumer's committed offset; the primary pipeline health metric (§11.1).
- **Monitoring** — the lag/retry/DLQ/reconciliation observability layer that makes data loss visible instead of silent (§11).
