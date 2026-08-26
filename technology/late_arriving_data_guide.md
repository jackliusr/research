# Handling Late-Arriving Data: The Reliability Deep-Dive

*A comprehensive deep-dive on the late-data problem: what late data is (events arriving after their event time) and why it threatens completeness and correctness, the two clocks (event time vs processing time), the streaming foundations (event-time processing, watermarks, allowed lateness, event-time windows), the watermark deep-dive (Flink/Dataflow mechanics, periodic vs punctuated generation, watermark drift), the late-event handling toolkit (allowed lateness, side outputs, retractions and corrections), the batch and warehouse patterns (late-arriving dimensions and SCDs, corrections, backfill/reprocessing), CDC late arrivals, the measurement of lateness (on-time/late/completeness metrics and data SLAs), the design patterns (kappa vs lambda, upsert/merge), a worked example (a banking cross-border payment stream), and the one-page summary — with the final word: plan for the late, measure the late.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Streaming / Data-Engineering Reliability (technology/)
> **Audience:** Solution architects, streaming-platform engineers, data-platform and warehouse teams
> **Last Updated:** August 2026

**Cross-references:** [event_stream_processing_guide.md](event_stream_processing_guide.md) (the streaming umbrella — watermarks and lateness were subsections there; **this guide is the dedicated deep-dive** that gives the late-data problem the full treatment across streaming *and* batch), [complex_event_processing_guide.md](complex_event_processing_guide.md) (out-of-order event streams from the pattern-detection angle — the same disorder, different consumer), [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (Kleppmann Ch. 11 "Streams" — the theory of event-time vs processing-time and stream reprocessing), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (the reliability layer underneath: retries, at-least-once delivery and DLQs — retries are a *cause* of out-of-order arrival and the DLQ is the batch cousin of the side output), [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (the broker/log that makes replay and reprocessing possible), [data/delta_lake_vs_iceberg.md](data/delta_lake_vs_iceberg.md) (lakehouse time-travel and `MERGE` — the batch-side machinery for corrections and backfills), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (integration frameworks that must inherit late-data handling), [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable retries — lightly; the same "event happened long ago, effect applied now" skew appears in workflows), [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md) and [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md) (banking EOD and late-file handling — the batch half of this guide's story), [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md) (retro-dated postings — a native late-arriving-data workload), [banking/financial_fraud_detection_at_scale_guide.md](banking/financial_fraud_detection_at_scale_guide.md) (late fraud events — lightly; fraud scoring cares *when* an event happened, not when it arrived).

---

## Table of Contents

1. [The Late-Data Problem](#1-the-late-data-problem)
2. [The Streaming Foundations: Event-Time, Watermarks, Allowed Lateness](#2-the-streaming-foundations-event-time-watermarks-allowed-lateness)
3. [The Watermark Deep-Dive](#3-the-watermark-deep-dive)
4. [The Late-Event Handling: Allowed Lateness, Side Outputs, Updates](#4-the-late-event-handling-allowed-lateness-side-outputs-updates)
5. [The Batch and the Warehouse: SCDs, Corrections, Backfill](#5-the-batch-and-the-warehouse-scds-corrections-backfill)
6. [The CDC Late Arrivals](#6-the-cdc-late-arrivals)
7. [The Measurement: On-Time, Late, Completeness, SLAs](#7-the-measurement-on-time-late-completeness-slas)
8. [The Design Patterns: Kappa/Lambda, Upserts](#8-the-design-patterns-kappalambda-upserts)
9. [The Worked Example: A Banking Cross-Border Payment Stream](#9-the-worked-example-a-banking-cross-border-payment-stream)
10. [The Summary: Late Data in One Page](#10-the-summary-late-data-in-one-page)
11. [Verification and Claims-Status](#11-verification-and-claims-status)
12. [References](#12-references)
13. [Glossary](#13-glossary)

---

**How this guide fits together (a reading map).** §1 defines the problem and the vocabulary — the two clocks, out-of-order arrival, the lateness spectrum. §2–§4 are the streaming stack: the foundations (§2), the watermark mechanism in depth (§3), and the handling toolkit — allowed lateness, side outputs, retractions and corrections (§4). §5–§6 are the batch/warehouse half: late-arriving dimensions and SCDs, corrections, backfill (§5), and CDC's special flavor of lateness (§6). §7 is the measurement discipline — on-time/late/completeness metrics and data SLAs — that makes all of it auditable; §8 the architecture patterns (kappa/lambda, upserts) that house it. §9 ties everything into one worked example — a banking cross-border payment stream — and §10 compresses the whole guide to one page. Readers short on time: read §1, §9.4, §10, and the Glossary; then return to §3–§4 for the mechanism details.

---

## 1. The Late-Data Problem

### 1.1 What Late Data Is: Events That Arrive After Their Event Time

**Late data — late-arriving data — is any event whose *processing time* (when the system observed it) is significantly after its *event time* (when the event actually happened).** The event itself is perfectly valid, complete, and well-formed; it is simply *late to the party*: it arrives after the computation that was supposed to include it has already been emitted, or after the system's completeness estimate has moved past it.

The canonical definition comes from the stream-processing engines. In Apache Flink's documentation, **an element is late if its timestamp is older than the watermark at the time it arrives at an operator** — the watermark being the engine's running estimate of "how much event time has been seen." In Google Cloud Dataflow's model, **late data is data whose event time is behind the watermark when it arrives** — the documentation's own analogy is "a delayed flight still lands": the data arrives after its expected time, but it is not invalid.

The point of the definition: **lateness is a property of an event *relative to a processing pipeline's progress*, not a property of the event itself.** The same payment event is "on time" in a pipeline with a generous watermark and "late" in one with an aggressive watermark. Lateness is therefore something you *design for* — you choose how much lateness your system tolerates — not something you can eliminate.

### 1.2 Why It Matters: Completeness and Correctness

Late data matters because time-based computation is the core of stream processing, and time-based computation is *wrong* if events arrive after the computation has closed the books on a time interval. Two distinct failure modes:

- **Incompleteness.** A windowed aggregate (payments per hour, trades per day, sessions per minute) is emitted when the engine decides the window is done. If a late event for that window arrives afterwards and is dropped, the aggregate is *missing* an event. The number is published, consumed, and acted upon — and it is too small, permanently, unless something corrects it.
- **Incorrectness.** If the late event is *not* dropped but is processed anyway into whatever window is currently open (keyed by *processing* time rather than event time), it lands in the wrong bucket — the event is counted, but in the wrong hour/day, so both the bucket it should have been in and the bucket it landed in are wrong. Out-of-order arrival compounds this: the "latest" event seen by a naive counter is not the latest event that *happened*.

The completeness/correctness framing is the standard one in the streaming literature — the Flink and Dataflow docs both motivate watermarks and late-data handling explicitly in these terms: results must reflect *all* events whose event time falls in the window (completeness) and must attribute each event to the correct window (correctness). For a bank, this is not an academic nicety: a settlement-amount total that is 0.02% short, or an intraday liquidity report that buckets a payment into yesterday, is a reconciliation break and a regulatory question.

### 1.3 The Two Clocks: Event Time vs Processing Time

Every event carries (or can be assigned) two timestamps, and conflating them is the root of the late-data problem:

| Clock | What it records | Set by | Properties |
|---|---|---|---|
| **Event time** | When the event *happened* in the real world | The source system, embedded in the event (or assigned at ingestion from a trusted field) | Immutable, meaningful, may be in the past — sometimes far past |
| **Processing time** | When the event was *observed/processed* by the pipeline | The machine clock of the processing engine | Always "now", monotonically increasing, but meaningless for historical accuracy |

The two clocks diverge for mundane reasons: a mobile device offline overnight, a batch file from a correspondent bank generated at EOD and FTP'd hours later, a retry storm after a broker outage (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §on retries), a connector paused for maintenance, a human operator correcting a record two days after the fact. **The divergence is unbounded in theory and often large in practice** — seconds on an API stream, hours for batch files, days for manual amendments.

The design consequence: **if you compute on processing time, your "time-based" results are actually "arrival-order-based" results** — they describe when your pipeline happened to see things, not when things happened. The Dataflow Model paper (Akidau et al., 2015) and Kleppmann's DDIA (Ch. 11, see [ddia_study_companion_guide.md](ddia_study_companion_guide.md)) both make the same point: event time is the only clock that answers "what happened when?"; processing time only answers "when did we get around to looking?"

### 1.4 Out-of-Order Arrival

Late data almost always arrives *out of order* — the stream's arrival sequence differs from the events' chronological sequence. Mechanisms that produce disorder:

- **Network and broker delays** — different producers, different routes, different latencies; a fast path can overtake a slow path (two corridors of the same payment stream, a webhook vs a batch file).
- **Retries and redeliveries** — at-least-once delivery (the default for serious brokers, see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §2) means a failed-then-retried event can arrive *after* events that happened later. Redelivery is a direct generator of both duplicates and disorder.
- **Batching at the source** — EOD files, statement files, offline syncs: the source collects events over hours and emits them in one burst, timestamped hours in the past.
- **Partitioning and parallelism** — with multiple partitions/consumers, there is no global arrival order; per-key order is preserved (Kafka guarantees per-partition order), but cross-key order is not.
- **Human/system corrections** — amendments, reversals, adjustments are new events that *reference* old event times (see [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md) on amendments and reversals).

Out-of-order arrival is the *empirical* problem; watermarks (next section) are the *mechanism* for reasoning about it, and late-event handling is the *policy* for what to do with what's left. The [complex_event_processing_guide.md](complex_event_processing_guide.md) covers the same disorder from the pattern-detection perspective (a fraud pattern must not miss a payment that merely arrived late).

### 1.5 The Lateness Spectrum: From Milliseconds to Days

Lateness is not binary — it is a spectrum, and the *shape* of the spectrum dictates the design:

| Lateness class | Typical cause | Example | Design response |
|---|---|---|---|
| Milliseconds–seconds | Network jitter, broker retries, consumer rebalances | API-gateway payment event redelivered | Absorbed by the out-of-orderness bound; invisible to the design |
| Minutes | Source-side batching, slow connectors, small outages | Screening result queued behind a backlog | Covered by watermark bound + allowed lateness |
| Hours | EOD/batch files, correspondent statements, overnight sync | SWIFT MT103 file generated at 18:00, ingested 08:00 next day | Punctuated file watermarks; batch reconciliation |
| Days | Human corrections, regulatory restatements, retro-dated postings | Ops team amends a payment 3 days after value date | Side output + amendment chain; never windowed |
| Effectively unbounded | Legal/audit restatements, source-system archaeology | A year-old payment re-classified | Warehouse restatement; raw-log retention |

The design consequence: **choose the boundary between "handled by watermarks" and "handled by reconciliation" where the spectrum's probability mass drops** — measure the tail (your late-rate histogram, §7.1) and place allowed lateness past the bulk of it, with the side-output + batch path catching the rest. A design with no spectrum analysis is a design with a guessed boundary.

### 1.6 The Problem Table

| Aspect | Description | Why it's a problem | When it bites hardest |
|---|---|---|---|
| **Late arrival** | Event arrives after its event time; timestamp < watermark at arrival | Windowed results emitted without it are incomplete | Any windowed aggregation, any "close the books" boundary |
| **Event vs processing time** | Two clocks; conflation buckets events by arrival, not by fact | Results describe pipeline behavior, not reality | Historical reporting, reconciliation, regulatory queries |
| **Out-of-order arrival** | Arrival order ≠ event-time order | Naive counters and last-wins logic attribute events wrongly | Fast+slow source mixes, retries, batch-file + API hybrids |
| **Unbounded skew** | Lateness has no a priori maximum | No fixed wait is ever "safe"; hard cutoff = guaranteed loss | Corrections/amendments, mobile offline, correspondent files |
| **Silent drop** | Late event discarded, no trace | Incompleteness without visibility — the worst kind | No side output, no late-data metric, no reconciliation |

---

## 2. The Streaming Foundations: Event-Time, Watermarks, Allowed Lateness

### 2.1 Event-Time Processing

**Event-time processing means the computation is driven by the events' own timestamps, not by the pipeline's clock.** Every event is stamped (ideally at the source) with when it happened; operators — windowing, aggregation, joins, pattern detection — use that timestamp for all time-based decisions. In Flink, event time is one of the three time characteristics (processing time, event time, ingestion time); in Dataflow/Beam, event time is the default and primary notion of time.

Event-time processing is what makes late data *detectable and handleable* rather than silently mis-bucketed: because the event carries its own truth about when it happened, the pipeline can (a) recognize that an arriving event belongs to a window that was already emitted, (b) decide what to do with it — update, side-output, or drop — and (c) measure exactly how late it is. Without event-time processing, "late data" is not even a well-defined concept — everything is just "data that arrived."

Trade-off, honestly stated: event-time processing costs **state and latency**. The pipeline must buffer events and keep window state until it is confident windows are complete, instead of emitting the instant an event arrives. The guarantee of ordering is replaced by the burden of buffering.

A concrete contrast: a payment initiated at **10:00:03** that the pipeline observes at 10:00:07 (4 seconds of processing delay) is trivially on time in event-time processing — its event time is 10:00:03, so it belongs in the 10:00 hour window. The same event observed at **14:00:12** (a file re-issue) still belongs in the 10:00 hour window in event-time processing, because its event time never changed — which is exactly what makes late data *correctable* rather than *lost*. In processing-time processing, the 14:00 arrival would land in the 14:00 window: the payment would be counted for the wrong hour, the 10:00 window would be permanently short, and no amount of replay would fix the attribution — the damage is in the bucketing itself.

### 2.2 Watermarks: The Definition

**A watermark is a timestamp that the pipeline maintains as an estimate of "how far along" it is in event time: the pipeline assumes that all events with an event time earlier than the watermark have (probably) arrived.** Formally, both major engines define it the same way:

- **Flink** (documentation): a watermark carrying timestamp *t* signifies that events with a timestamp *t* or earlier have arrived; elements arriving with a timestamp older than the current watermark are considered late. Flink's docs are explicit that **watermarks are based on heuristics — "this is not a guarantee."**
- **Dataflow** (The Dataflow Model paper; documentation): a watermark is **a lower bound on the event times of future (not-yet-arrived) events**. Dataflow watermarks are computed by the service from the sources, and the model distinguishes *exact* watermarks (the source can prove nothing earlier will arrive) from *estimated* watermarks (the source is inferring).

The two definitions are the same coin: Flink's "everything ≤ T has arrived" and Dataflow's "nothing < T will arrive in the future" are dual statements. The essential truth: **a watermark is an estimate of completeness, not a guarantee of it.** It says "we *believe* we've seen everything up to here" — and the entire late-data discipline exists because that belief can be wrong.

Watermarks exist to answer the *when* question from the stream-processing four questions ([event_stream_processing_guide.md](event_stream_processing_guide.md) §1.3): **when is a window safe to emit?** A window is emitted (fired) when the watermark passes the end of the window — because at that point, the pipeline believes every event for that window has arrived.

### 2.3 Allowed Lateness: The Threshold

**Allowed lateness is a threshold, configured per pipeline, on how late (in event time, relative to the watermark) an event may arrive and still be processed into the window it belongs to.** It is the buffer of forgiveness between the watermark and total abandonment:

- **Flink**: `window.allowedLateness(Time.seconds(30))` — elements arriving after the watermark but within 30 seconds of *event time* still update the window result; window state is retained until the watermark passes window-end + allowed lateness, then purged. Default allowed lateness is 0 — any element after the watermark is late-dropped.
- **Dataflow/Beam**: `window.withAllowedLateness(Duration.standardMinutes(5))` — late data within the allowance still triggers the window and updates its panes; beyond it, late data is dropped or routed to the late-data collection.

The threshold is a **trade-off knob between completeness and latency/resource use**: a bigger allowance captures more late events but delays "final" results and holds window state (memory, RocksDB) longer. Note the subtlety: allowed lateness is measured in *event-time* units (the event is late relative to the watermark), which is a different axis from processing-time delays — an event can be 30 seconds late in event-time terms while the pipeline processes it in real time.

### 2.4 Event-Time Windows

**An event-time window groups events by their event-time timestamps into fixed intervals (tumbling: every hour), sliding (every hour, advanced every 5 minutes), or session windows (bursts separated by gaps).** The window's lifecycle, in both Flink and Dataflow:

1. **Open**: events with event times in `[windowStart, windowEnd)` are assigned to the window as they arrive.
2. **Fire**: when the watermark passes `windowEnd`, the window's result is emitted (the "first" or "on-time" result).
3. **Re-fire on late data**: within allowed lateness, a late event re-triggers the window; the result is re-emitted, updated (Flink fires again; Dataflow re-fires the window's trigger per its late-data policy).
4. **Close and purge**: when the watermark passes `windowEnd + allowedLateness`, state is dropped; any later event for that window is late beyond allowance — side output or drop.

Event-time windows are the *container* that the late-data problem lives inside: every strategy in this guide is ultimately about what happens to an event whose window has already fired (§4), or whose window boundary has already been closed in a batch warehouse (§5).

A window's lifecycle with numbers makes the mechanics legible. A tumbling 1-hour window `[10:00, 11:00)` on the payment stream: at 10:23 the watermark reads 10:18 — the window is open, accumulating events with event times inside the hour. At 11:05 the watermark passes 11:00 — the window fires: `total = 1,000,000`, emitted downstream as the on-time result. At 11:40 an event with event time 10:58 arrives — late, but within the 24 h allowed lateness: the window's state is still alive (not purged), the event is folded in, and the window re-fires: `total = 1,000,150`. At 11:00 next day the watermark passes `11:00 + 24 h` — the window is purged; an event with event time 10:59 arriving at 11:30 is beyond allowance and goes to the side output. Four lifecycle events — fire, re-fire, purge, side-output — each one a decision the design in §4 makes explicit.

### 2.5 The Foundations Table

| Concept | Description | Trade-off |
|---|---|---|
| **Event-time processing** | Compute on when events happened (embedded timestamps), not when observed | Correct attribution and historical accuracy vs. buffering state and added latency |
| **Watermark** | Estimate of event-time progress: "all events ≤ T have (probably) arrived" — the trigger for emitting windows | Aggressive (early) watermarks → low latency but more late data; conservative → fewer late events but slower results |
| **Allowed lateness** | Event-time threshold within which a late event still updates its window's result | Larger allowance → better completeness, more state retained, "final" results delayed |
| **Event-time window** | Groups events by event time (tumbling/sliding/session); fires on watermark, re-fires on in-allowance late data | Fine granularity → more windows/state; coarse → less state, coarser analysis |

---
## 3. The Watermark Deep-Dive

### 3.1 The Watermark Mechanics: Flink and Dataflow

**How watermarks are produced and propagated is the difference between "the window fired" and "the window fired at the right time."** Both Flink and Dataflow implement the same core mechanism — a running, monotonic estimate of event-time progress — but with different machinery:

**Flink.** Watermarks are generated by `WatermarkStrategy`s attached to sources, and each operator propagates them downstream:

1. **Per-source/per-partition generation.** Each parallel source subtask (each Kafka partition, for the Kafka connector) observes the event timestamps of the records it reads and produces a watermark. The standard strategy, `WatermarkStrategy.forBoundedOutOfOrderness(Duration)` — the workhorse for out-of-order streams — maintains **watermark = max(observed event time) − maxOutOfOrderness**: it assumes no event will be more than *maxOutOfOrderness* late, and subtracts that bound from the best timestamp seen. (Verified against the Flink docs and the strategy's source semantics; see §11.)
2. **Alignment across partitions.** A downstream operator receives one watermark per upstream subtask and **takes the minimum** — the stream's watermark only advances when *every* input (partition) has advanced. This is correct and conservative: one silent partition would otherwise let the watermark race ahead and declare windows complete while that partition's events are still unread.
3. **Idle-source handling.** A partition with no new records produces no new watermark, and the minimum stalls the whole stream — the classic **idle-partition problem**. Flink's fix is `withIdleness(Duration)`: a partition that has been silent longer than the idle timeout is excluded from the minimum, letting the watermark advance. (Verified — Flink docs, "dealing with idle sources.")
4. **Propagation and window firing.** Watermarks flow through operators; when a watermark passes a window's end timestamp, the window fires. Operators emit the watermark downstream as the min of their inputs' watermarks, so the estimate is global and monotonic.

**Dataflow.** Watermarks are **computed by the Dataflow service**, primarily at the sources, and the model is explicit about confidence:

1. **Source-based computation.** Each source reports its watermark — e.g., a Pub/Sub source infers a watermark from the oldest undelivered message; a file source advances it as files complete. The service derives each stage's watermark from its inputs.
2. **Exact vs estimated.** The Dataflow model formally distinguishes **exact watermarks** (the source can *prove* no earlier events will arrive — e.g., a log file that is complete) from **estimated watermarks** (inferred from observed timing). Estimated watermarks can be wrong — too early (marking things done that aren't) or too late (unnecessary delay) — and the model says so explicitly.
3. **Late-data accounting.** Dataflow tracks events that arrive behind the watermark ("late data") and, under the default trigger, windows fire again when late data arrives within allowed lateness — the service-side analogue of Flink's allowed-lateness re-fire.

The architectural commonality is documented in the VLDB survey *Watermarks in Stream Processing Systems* (Begoli et al., 2021): both engines implement event-time semantics via watermarking, differing mainly in *who* computes the watermark (user strategy in Flink vs. service-managed in Dataflow) and in how aggressively the two treat the estimate.

**The watermark in code.** The strategy behind the mechanics above, in Flink's API (illustrative — syntax per Flink's `WatermarkStrategy`):

```java
DataStream<Payment> payments = env.fromSource(
    kafkaSource,
    WatermarkStrategy.<Payment>forBoundedOutOfOrderness(Duration.ofMinutes(5))
        .withIdleness(Duration.ofMinutes(2))
        .withTimestampAssigner((event, ts) -> event.getEventTimeMillis()),
    "payments-kafka");
```

The three clauses are the whole design in miniature: `forBoundedOutOfOrderness(5 min)` sets the drift budget (§3.3), `withIdleness(2 min)` stops one silent partition from stalling the watermark (§3.1), and the timestamp assigner declares *which field is event time* — the declaration that turns the stream into an event-time stream at all (§2.1). The same shape exists in Dataflow/Beam: a per-source watermark policy, with the service computing and propagating the watermark.

### 3.2 The Watermark Types: Periodic vs Punctuated

Flink's `WatermarkGenerator` interface defines the two generation *styles*, and the distinction is worth knowing because it maps to real stream shapes:

- **Periodic watermarks** — emitted **on a timer, at a fixed processing-time interval** (Flink's `ExecutionConfig.setAutoWatermarkInterval`, default **200 ms**). Regardless of how many events arrive, the generator emits a new watermark every interval (if the estimate has advanced). This is the standard for high-throughput streams: cheap, steady, and decoupled from event arrival rate. (Verified — Flink docs "Periodic Watermark Generation"; default interval stated per Flink docs.)
- **Punctuated watermarks** — emitted **only when a special marker or condition is encountered in the stream** (e.g., every Nth element, or an explicit control record from the source). A punctuated generator emits a watermark *if and only if* the current element triggers one. Useful when the source can tell you *precisely* "everything before this marker is complete" — the natural home for exact watermarks (e.g., a file-source emitting a watermark when a file boundary passes, or a batch-job writing a "cutoff" marker into a topic). (Verified — Flink docs "Punctuated Watermark Generation.")

Dataflow's service-computed watermarks are conceptually closer to a hybrid: sources report progress events, and the service interpolates between them — but the two-generation-style vocabulary is Flink's, and it is the terminology most teams use when discussing watermark *design* (§9's worked example designs per-stream watermark strategies in exactly these terms).

### 3.3 The Watermark Drift

**Watermark drift is the gap between the watermark (what the pipeline believes has arrived) and the true event-time frontier (what has actually happened at the sources).** Drift is the mechanism by which lateness becomes visible: every late event is, by definition, evidence that the watermark ran ahead of reality.

Sources of drift (and their mitigations):

- **Conservative strategies.** `BoundedOutOfOrderness(10 min)` intentionally lags 10 minutes behind the newest observed event. Drift is *chosen* — the knob that trades latency for completeness.
- **Idle partitions/sources.** One silent partition stalls the min-watermark indefinitely (see §3.1). Mitigation: idle-timeout exclusions (Flink `withIdleness`), or periodic heartbeat/keepalive records so the source never looks idle.
- **Stragglers beyond the bound.** Events later than the strategy's assumption arrive anyway — the estimate was wrong. Mitigation: allowed lateness (§4.1), side outputs (§4.2), and batch reconciliation (§5) for the tail.
- **Source outages and backlog.** A paused connector (maintenance, crash) lets the watermark advance on the data that *was* read while the backlog behind it is stale. Mitigation: pause watermark advancement during outage (or rely on idle timeouts), and re-align via reprocessing (§5.3).
- **Watermark-clock skew.** The watermark is compared against wall-clock time for latency metrics; if the metric clock is wrong, "drift" is misread. Minor, but it confuses operations.

Drift is not a bug — it is the *design surface*: **your watermark strategy is a statement about how much lateness you expect and how much latency you accept.** The measurement story in §7 exists so drift and lateness are numbers you can see rather than surprises you discover in a reconciliation break.

A drift timeline makes it concrete. At 10:00:00 the pipeline's watermark reads 09:55:00 (5-minute bounded out-of-orderness): the pipeline believes "everything ≤ 09:55 has arrived," and the 09:50–10:00 window has not fired. At 10:00:02 an event with event time 09:58:31 arrives — on time, absorbed. At 10:00:07 one Kafka partition goes silent (a broker hiccup; the other 11 partitions keep producing) — without `withIdleness`, the min-aligned watermark freezes at ~09:55 while real time marches on: *drift grows*, window firings stall, and end-to-end latency balloons even though nothing is wrong with the data. With `withIdleness(2 min)`, at 10:02:07 the silent partition is excluded and the watermark resumes advancing; the backlog behind the recovering partition then arrives as a burst of events that are *not* late (their event times are older than the still-conservative watermark) — the idle-timeout plus a conservative strategy converted a would-be late-data storm into ordinary processing. This is drift management in action: the watermark is a control variable, and idle handling is its most important regulator.

### 3.4 The Watermark Table

| Aspect | Design choice | Example |
|---|---|---|
| **Strategy** | How much out-of-orderness to absorb at generation | `forBoundedOutOfOrderness(5 min)` on an API event stream; a file source emitting punctuated watermarks at file boundaries |
| **Granularity** | Per-source, per-partition, then min-aligned downstream | Kafka connector: per-partition watermarks, min across 12 partitions; `withIdleness(2 min)` so silent partitions don't stall |
| **Interval** | How often periodic watermarks are emitted | Flink default 200 ms; longer intervals for low-rate sources |
| **Confidence** | Exact vs estimated (Dataflow) / heuristic (Flink) | Exact: complete log files; estimated: network streams — assume the estimate is wrong sometimes and design for it |
| **Drift budget** | The chosen gap between watermark and reality | 5 min out-of-orderness + 24 h allowed lateness ⇒ 99.9% of events are "on time," 0.1% are handled as late |
| **Idle handling** | What happens when a source goes silent | `withIdleness(2 min)`; heartbeat records; pause-and-replay |

---

## 4. The Late-Event Handling: Allowed Lateness, Side Outputs, Updates

This section is the operational heart of the guide: *given that late events will arrive, what does the pipeline do with them?* There are exactly three fates for a late event, in increasing order of forgiveness: (a) **update the window it belongs to** (within allowed lateness), (b) **route it to a side output** for audit/reprocessing (beyond allowed lateness), (c) **drop it silently** — which should never be a deliberate design choice for financial data.

### 4.1 The Allowed-Lateness Mechanism

**Fate (a): the event still counts, and the result is corrected.** Configuration (Flink: `allowedLateness`; Dataflow/Beam: `withAllowedLateness`) keeps the window's state alive after the watermark passes window-end, for the configured event-time allowance. When a late event arrives inside the allowance:

1. The event is assigned to its (already-fired) window — the engine knows its event time.
2. The window's aggregation state is updated with the event.
3. The window result is **re-emitted with the corrected value** — the downstream consumer sees a second, updated output for the same window (Flink re-fires the window; Dataflow's trigger re-fires the window for late data under its late-data policy).
4. When the watermark passes `windowEnd + allowedLateness`, the window state is purged — after this point, events for that window can no longer be processed: they go to fate (b) or (c).

The mechanism is verified across the Flink and Dataflow documentation (allowed-lateness config, late-data re-firing, state retention until `windowEnd + lateness`). The key design fact: **allowed lateness turns "the result is final" into "the result is provisional until window-end + allowance."** Downstream consumers of windowed output must therefore expect *multiple outputs per window* — a first (on-time) value and one or more corrected values — which is exactly why the update patterns in §4.3 exist.

### 4.2 The Side Output: The Late Stream

**Fate (b): the event is preserved but separated.** Flink's `sideOutputLateData(tag)` on a windowed stream routes every event that arrives beyond allowed lateness into a **separate side-output stream** (a tagged `DataStream`), instead of dropping it; the main stream never sees it. Dataflow's equivalent is the late-data handling path — data beyond allowed lateness is written to a separate location/collection rather than silently discarded (the docs describe exactly this: "write late data to a separate location for analysis").

The side output is the *streaming* form of the dead-letter queue (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §7 — the DLQ): a failure path that is **visible, isolated, and redrivable** rather than silent. The late stream is typically:

- **Landed** to a topic/object store with the event's original timestamp intact — nothing is lost, auditability is preserved (a regulatory requirement for payment data);
- **Measured** — late-stream volume is a first-class metric (§7.1), because a growing late stream is a watermark-design failure signal;
- **Redriven** — a batch reconciliation job (§5.3) or a corrected re-ingestion absorbs the late stream with full event-time fidelity, so the final warehouse state is complete even though the real-time results were not.

The side output is *not* a place to dump problems and forget them — it is the *bounded-loss escape hatch* that converts a guaranteed data-quality incident into a measured, scheduled reconciliation item.

**The handling in code.** The allowed-lateness + side-output pair, in Flink's windowed API (illustrative):

```java
OutputTag<Payment> lateTag = new OutputTag<Payment>("late-payments") {};

SingleOutputStreamOperator<CorridorHourTotal> totals = payments
    .keyBy(Payment::getCorridor)
    .window(TumblingEventTimeWindows.of(Time.hours(1)))
    .allowedLateness(Time.hours(24))
    .sideOutputLateData(lateTag)
    .aggregate(new CorridorHourAggregate());

DataStream<Payment> latePayments = totals.getSideOutput(lateTag);
latePayments.sinkTo(latePaymentsTopic);   // land, measure, reconcile
```

Three lines carry the whole §4 design: `allowedLateness(24 h)` keeps the corridor-hour window's state alive for a day after the watermark passes (in-allowance late events re-fire the window with corrected totals); `sideOutputLateData` routes everything older than that to the `late-payments` stream instead of dropping it; and the sink to a topic (plus object storage) makes the beyond-allowance tail visible, measurable, and redrivable (§4.2). The same structure in Dataflow/Beam is `withAllowedLateness` on the window plus the late-data collection path.

### 4.3 The Update Patterns: Retraction and Correction

When late data updates a window (fate (a)), the pipeline must communicate *change* downstream. Two canonical patterns:

**Retraction (withdraw-then-replace).** The pipeline emits a **negative/withdrawal record for the previous value** followed by the new value — so a consumer that summed the output stream stays correct. This is the native semantics of Flink's Table API and SQL (`UPDATE` on a windowed aggregate produces a retraction row with the old value negated, then the new row; the changelog stream is exactly "insert / update-before+update-after / delete"). Dataflow/Beam's equivalent is the **"accumulating & retracting" accumulation mode** for triggers: later panes retract the previous pane's contribution before adding the new one. (Verified — Flink Table API changelog/retract documentation; Beam trigger accumulation modes; Confluent's streaming-SQL docs on retractions.) Retractions are the *mechanical* pattern — how the engine expresses "the old number is wrong, here's the new number" — and they are the foundation for exactly-once-style downstream materialization (an upsert store keyed by window+key simply applies the changelog).

**Correction (supersede-by-reference).** The *business* pattern: a late event that *corrects* a previously processed event arrives as a new event that references the original — an amendment, reversal, adjustment, or re-issue (banking: `PaymentAmended`, `PaymentReversed`; see [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md) and [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md) on amendments/reversals). Correction events are the *semantic* layer on top of retraction: the correction event says "the original event's effect is superseded," and the engine's retraction machinery is how that supersession is expressed in an aggregate. Corrections are also the *reason* lateness is unbounded: there is no watermark that can predict when a human will amend a record — which is why the correction path (side output → batch reconciliation → warehouse upsert, §5–§6) must exist outside the real-time window machinery.

The two patterns compose: **corrections are business facts; retractions are the engine's change language; allowed lateness decides which path a late event takes.** A late *correction* event within allowed lateness triggers a retraction+insert in the window; a late correction beyond it goes to the side output and eventually lands as a warehouse upsert (§5.2).

A worked retraction makes the pattern concrete. The 10:00 corridor-hour window fires at 10:05 (watermark passes) with **total = 1,000,000**. At 13:40 a confirmation arrives with event time 10:47 — inside the 24 h allowance. The window re-fires: the changelog emits a retraction row for the 10:00 key (**total −1,000,000, flagged as update-before**) followed by the new row (**total 1,000,150, update-after**). A downstream consumer that sums the changelog ends at 1,000,150 — correct — while a consumer that only *read* the first emission would be 150 short until it processes the retraction. The design lesson: **retraction only corrects consumers that speak the changelog language** — which is why the upsert materialization (§8.2) is the standard downstream shape, and why the "never silently overwrite" rule (§5.2) governs what the *business* sees: the 10:00 total is restated with a visible change, not quietly replaced.

### 4.4 The Handling Table

| Strategy | Mechanism | Use-case |
|---|---|---|
| **Allowed lateness** | Keep window state until watermark > window-end + allowance; re-fire window with updated result | Normal tail of lateness (seconds–hours): the bulk of late events, corrected in real time |
| **Side output** | Route beyond-allowance events to a separate late stream (Flink `sideOutputLateData`; Dataflow late-data path); land + measure + redrive | The unpredictable tail (hours–days): amendments, batch-file stragglers; audit + reconciliation |
| **Retraction** | Emit negative old row + new row on late update (Flink Table changelog; Beam accumulating-&-retracting) | Correcting aggregates downstream; materializing exactly-once output stores |
| **Correction event** | New event referencing/superseding the original (amendment, reversal) | Business-level corrections, unbounded lateness, regulatory restatements |
| **Silent drop** | Discard the event with no trace | Never for financial data — if used, it is a bug, not a strategy |

---

## 5. The Batch and the Warehouse: SCDs, Corrections, Backfill

The streaming half of this guide (watermarks, side outputs) handles lateness *while the pipeline runs*. The batch half handles lateness *after the fact* — in the warehouse, where the final, queryable, auditable state lives. **The two halves must agree, and the batch half is where completeness is ultimately guaranteed.** Late data that the stream handled gracefully is the easy case; late data that the stream *never saw* (a correspondent's EOD file arriving a day late) is a warehouse problem from the start.

### 5.1 The Late-Arriving Dimensions: SCDs

Warehouses model facts and dimensions, and **both can arrive late**. A *late-arriving fact* (a payment event for yesterday, loaded today) is handled by backfill (§5.3). A *late-arriving dimension* is subtler: the fact row references a customer/counterparty/corridor dimension row that hasn't arrived yet — or arrives with corrections after facts already reference it. The dimensional-modeling answer is the **slowly changing dimension (SCD)**, the Kimball pattern family for "dimension attributes change over time," with three canonical types (verified against Kimball Group and standard dimensional-modeling references):

- **SCD Type 1 — overwrite.** The new attribute value replaces the old in place; no history is kept. Simple, current-truth only; *destroys history* — reports that grouped on the old value silently change. Appropriate when the attribute is genuinely a correction (a data-entry fix), not a change.
- **SCD Type 2 — versioned rows.** Each change inserts a *new row* with a surrogate key, effective/expiry dates, and a current-flag; history is fully preserved and facts keep pointing at the version that was current at fact time (point-in-time correctness). This is the standard for dimensions whose history matters — regulatory "what did we know when?" queries. Cost: row growth, surrogate-key joins.
- **SCD Type 3 — original/current columns.** A fixed "original value" column plus "current value" column (e.g., `original_region`, `current_region`); limited, explicit history. Niche — used when exactly one level of history is required (e.g., "before/after a reorganization").

For **late-arriving dimensions**, the established Kimball practice is: **load the fact against a default/placeholder dimension row** (a "unknown"/"pending" member keyed for the missing entity), then **when the real dimension row arrives, re-point the fact** (Type 1-style attribute update on the fact's dimension key) or **add the real row and let the placeholder remain for the window in which it was genuinely unknown** (Type 2-style). The placeholder pattern is verified as the standard remedy in Kimball's late-arriving-dimension guidance — the alternative (rejecting facts until the dimension arrives) breaks fact loading, and the alternative to the alternative (blocking the batch) breaks the SLA.

The SCD types in one tiny example — a customer's `segment` attribute changing from RETAIL to CORPORATE on 2026-03-15:

| SCD type | Rows after change | What a report on 2026-01 data shows | Cost |
|---|---|---|---|
| **Type 1** (overwrite) | One row, segment = CORPORATE | January's rows now say CORPORATE — history silently rewritten | Cheapest; destroys history |
| **Type 2** (versioned) | Two rows: RETAIL (eff. 2024-01-01, end 2026-03-15), CORPORATE (eff. 2026-03-15, current) | January's facts join to RETAIL — "as-was" reporting works | Surrogate keys, row growth, join complexity |
| **Type 3** (original/current) | One row with `original_segment = RETAIL`, `segment = CORPORATE` | Can answer "before/after" at one fixed point only | One-level history only |

For a late-arriving dimension (the customer's master file arrived 2026-03-20 for a fact dated 2026-03-10): Type 1 re-points the fact's join to the corrected row; Type 2 keeps the placeholder version for the fact's date and adds the real row going forward — both verified Kimball practice, and the choice is driven by whether the *interim* "unknown" attribution must be reproducible (Type 2) or not (Type 1).

### 5.2 The Corrections: Warehouse Restatement

Batch corrections are the warehouse mirror of §4.3's retraction/correction patterns:

- **Delta/change files.** The source system re-sends the corrected records (full-file re-issue or a delta of changed keys). The warehouse applies them as *updates*, not inserts — hence the `MERGE`/upsert discipline (§8.2).
- **Amendment tables.** For regulated domains (banking), corrections are *not* applied in place; they are posted as amendment/reversal records that adjust the original (a ledger never erases — it posts offsets; see [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md)). The warehouse mirrors this: the fact table keeps the original row and a `corrected_by` reference, and aggregate views sum originals + adjustments.
- **Restatement.** When a correction changes published numbers (a daily report already sent), the warehouse *restates*: the corrected value is published with the change made visible (restatement flags, change notes), not silently replaced — because downstream consumers (treasury, risk, regulators) already acted on the old number. This is the batch analogue of the retraction: the old value is withdrawn *visibly*, the new value replaces it.

The design rule that unifies streaming and batch corrections: **never silently overwrite a published number.** Streaming expresses this as retractions; batch expresses it as restatements and amendment tables; both preserve the audit trail.

### 5.3 The Reprocessing: Backfill

**Backfill is re-running a computation over historical data to produce corrected/complete results** — the batch answer to "the stream missed it, or got it wrong." The plumbing is already in this repo's guides: the broker log retains and replays ([kafka_alternatives_guide.md](kafka_alternatives_guide.md); [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §7.2 on redrive), the lakehouse provides time-travel and `MERGE` for applying corrections idempotently ([data/delta_lake_vs_iceberg.md](data/delta_lake_vs_iceberg.md)), and Kleppmann's DDIA covers stream reprocessing semantics (see [ddia_study_companion_guide.md](ddia_study_companion_guide.md) Ch. 11).

The discipline that makes backfill *safe*:

1. **Idempotency.** Re-running a backfill must produce the same final state whether it runs once or twice — upserts keyed on natural keys (§8.2) and dedupe keys ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §6) are the mechanisms. Without idempotency, backfill *is* the data-quality incident.
2. **Immutable raw.** The raw layer (topic/object store) is append-only and never mutated by backfill; corrections are *new* records (or new table versions — lakehouse time-travel), so any state can be rebuilt from raw at any point in time.
3. **Reprocessing = replay, not recompute-from-memory.** Kappa-style (§8.1): the same streaming code re-runs over the replayed log with a corrected watermark/strategy, producing corrected output that lands via upsert. The 200-line job that replays yesterday's topic with today's fixes is the canonical backfill shape.
4. **Cutover discipline.** Backfill writes to a staging area/version, is reconciled against the live state (row counts, totals), then is swapped in — a mini "restatement" (§5.2) of the affected period.

Backfill is the *final safety net* in the late-data story: watermarks and side outputs bound the *streaming* exposure, but backfill is what guarantees that **the warehouse, by T+1 (or T+N), is complete regardless of how late anything arrived.**

The operational backfill checklist (each item maps to a section of this guide):
1. **Freeze the input.** Capture the raw log range (topic offsets or object-store prefixes) to replay; never backfill a moving target.
2. **Make the write idempotent.** Upsert by natural key (§8.2) — the target must converge regardless of double-runs; dedupe keys for at-least-once sources ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §6).
3. **Fix the code, not just the data.** A backfill that re-runs broken logic reproduces the break; replay with the corrected watermark/strategy/business rule (§9.2's retuning loop).
4. **Stage and reconcile.** Compute into staging/version, compare counts and totals against live state and against source-of-truth counts (§7.1's completeness), then cut over.
5. **Publish the restatement.** Mark the affected period's numbers as restated (§5.2) — consumers who already acted on the old numbers must see the change.
6. **Measure the result.** After cutover, the period's completeness metric should read 100% — the backfill's success criterion is a metric, not a job exit code.

### 5.4 The Batch Table

| Pattern | Mechanism | Example |
|---|---|---|
| **Late-arriving dimension (placeholder)** | Fact loads against "unknown" dimension key; real row arrives later and facts are re-pointed (T1) or versioned (T2) | Payment fact for a new counterparty whose master data file arrived a day late |
| **SCD Type 1 / 2 / 3** | Overwrite / versioned rows / original+current columns | Customer address: T2 rows with effective dates for audit; T1 for pure corrections |
| **Correction as amendment** | New offset/reversal records adjust the original; aggregates sum originals + adjustments | Posting-engine reversal posts a −X adjustment; warehouse keeps both rows |
| **Restatement** | Corrected numbers published with visible change flags | Daily liquidity report re-issued with "restated" marker after a late SWIFT file |
| **Backfill / reprocessing** | Replay raw log with fixed code; upsert into target; reconcile then cut over | Re-run yesterday's settlement totals after a watermark mis-config |

---

## 6. The CDC Late Arrivals

### 6.1 The Change Data Capture: Late CDC Events

**Change data capture (CDC) is the practice of extracting row-level changes from a database's transaction log (rather than polling or full dumps) and streaming them as events** — Debezium being the canonical open-source implementation (Kafka Connect–based; verified against the Debezium project docs). CDC events are different from application events in one crucial way: **their event time is the database commit/change time** (Debezium carries the source DB timestamp, e.g., `ts_ms`), which the CDC connector reads from the log *after* the change is durable — so a CDC event's event time is always in the past relative to its arrival.

Late CDC events are therefore routine, and they arrive late for structural reasons:

- **Connector lag and restarts.** The connector reads the log asynchronously; after a pause (maintenance, crash, Kafka outage — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §4 for the broker-side failure modes), it resumes from its last committed offset and replays a *backlog of committed changes that are all, by definition, late* — and they replay in log order, which is commit order, which is fine per-row but can interleave with newer rows from other tables.
- **Replica lag / multi-source skew.** CDC from replicas, or from multiple databases feeding one stream, produces commits that were *simultaneous in reality* arriving at different times — the same out-of-order problem as §1.4, at the database layer.
- **Long-running transactions.** A transaction that stays open for an hour commits one row that is an hour "late" relative to the first statement's time. Some CDC implementations emit pre/post images whose times span the transaction.
- **Snapshot + incremental overlap.** Initial snapshot loads (bulk) run concurrently with incremental changes; deduplication and ordering between the two paths is a classic late-data source (a change captured in the snapshot and again in the log).
- **Backdated writes.** The source application itself updates a row with a backdated timestamp (banking: retro-dated postings, see [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md)) — the *change* is new, but its business date is old.

The honest flag: **"CDC late-data handling" is less canonically documented than streaming late-data handling** — Debezium documents ordering/offset guarantees, but the *watermarking of CDC streams* is practitioner practice, not a documented Debezium feature. This guide treats the standard practice (below) as engineering consensus and flags it accordingly in §11.

A Debezium CDC event in its canonical shape (fields elided for brevity) makes the "commit time is event time" point concrete:

```json
{
  "op": "u",                      // operation: c=create, u=update, d=delete
  "ts_ms": 1755400000000,         // when Debezium read the change (processing time)
  "source": {
    "db": "payments", "table": "settlement",
    "ts_ms": 1755399600000        // when the DB committed the change (event time)
  },
  "before": {"id": "PX-8812", "status": "PENDING", "amount": 1000},
  "after":  {"id": "PX-8812", "status": "SETTLED", "amount": 1000}
}
```

The two timestamps side by side are the two clocks (§1.3) wearing CDC clothes: `source.ts_ms` (the DB's commit, the event time) can precede `ts_ms` (the connector's read, processing time) by seconds to hours depending on connector lag — and every row in a resumed backlog is exactly that far behind. The row-level structure also explains the handling in §6.2: the `id` is the partition key and the natural key for the merge, `op` tells the merge whether to insert/update/delete, and `source.ts_ms` is the versioning signal for last-write-wins — a late `u` for PX-8812 simply re-merges, and the target converges.

### 6.2 The CDC Handling

The practitioner pattern for late/out-of-order CDC:

1. **Key-based ordering, not global ordering.** Kafka topics partition CDC events by primary key (Debezium's default per-table partitioning), giving per-row order; cross-row order is not guaranteed and must not be assumed. Stateful consumers (Flink, Kafka Streams) key by the business key and apply per-key state — the same discipline as any out-of-order stream, with the key as the ordering domain.
2. **Upsert semantics (last-write-wins by commit time).** The warehouse target receives CDC as a stream of `INSERT`/`UPDATE`/`DELETE` events and applies them with a `MERGE` keyed on the natural key (§8.2) — *not* by appending. A late `UPDATE` for a row simply re-merges; a late `DELETE` re-merges as a delete. The merge is idempotent, so replaying a CDC topic (backfill, §5.3) is safe.
3. **Watermarking CDC streams (when needed).** For *windowed* consumers of CDC (e.g., "how many account updates per hour"), treat the CDC stream like any event stream: assign watermarks from the source commit timestamps, choose out-of-orderness from the connector-lag profile, and route the tail to a side output — exactly §2–§4. For *state mirroring* consumers (cache, search, warehouse sync), watermarks are unnecessary — the merge absorbs lateness by construction; what matters is per-key order and idempotency.
4. **Snapshot/incremental dedupe.** Track the snapshot cutoff (the log position at snapshot start) and drop incremental events at positions ≤ cutoff — or make the merge idempotent (option 2) and let duplicates collide harmlessly.
5. **Reconciliation as the backstop.** Periodically compare source DB row counts/checksums against the CDC target — the completeness metric (§7) applied to CDC. Debezium itself is designed for *complete* capture ("every change captured" is its documented guarantee, modulo retention), but *your* pipeline's application of the stream is what must be verified.

### 6.3 The CDC Table

| Aspect | Late-arrival scenario | Handling |
|---|---|---|
| **Connector lag/restart** | Backlog of commits replays, all event-time-old | Per-key order + merge; measure connector lag as the CDC health metric |
| **Long transaction** | One commit is minutes/hours "late" | Accept via merge; avoid windowing CDC by processing time |
| **Snapshot overlap** | Change appears in both snapshot and log | Dedupe at cutoff, or rely on idempotent merge |
| **Backdated write** | New change, old business date | Warehouse keeps change-time (CDC) and business-time separately; SCD2 for business-time history (§5.1) |
| **Multi-source skew** | Two DBs' changes interleave out of order | Key-based processing; no global order assumption; reconcile per source |
| **Windowed CDC consumer** | Late change to a closed window | Watermark + allowed lateness + side output (§2–§4) |

---
## 7. The Measurement: On-Time, Late, Completeness, SLAs

### 7.1 The Metrics: On-Time, Late, Completeness

The late-data discipline fails without measurement: **a pipeline with watermarks but no lateness metrics is a pipeline whose data-quality failures are invisible until a reconciliation break.** Three metrics form the minimum observability set (the definitions are the standard ones from data-quality practice — completeness is one of the classic data-quality dimensions — applied to streaming):

- **On-time rate.** The fraction of events whose event time is at or ahead of the watermark *when they arrive* — i.e., events that landed inside the pipeline's completeness estimate and were processed normally. On-time rate is the direct readout of whether your watermark strategy matches your actual arrival profile.
- **Late rate.** The fraction of events that arrive *after* the watermark (event time behind the watermark at arrival). Subdivided into (a) *recoverable late* — within allowed lateness, corrected the window in real time — and (b) *beyond-allowance late* — routed to the side output (§4.2), which is the number that matters most because it is the number that says "the real-time result for this window is now permanently incomplete until the batch reconciles it."
- **Completeness.** The fraction of *expected* events actually accounted for, per window/period. Completeness needs an *expectation* to measure against — from source-system counts, from a batch reconciliation of the same period (the batch half of the pipeline is the ground truth for the streaming half), or from row-count comparisons after backfill (§5.3). The canonical banking form: **reconciliation** — the streaming total vs the core-banking total for the same period (see [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md) on reconciliation).

Supporting diagnostics: **watermark age** (now − watermark: how far behind event time the pipeline believes it is — the direct drift readout, §3.3), **window-fire latency** (event time at fire vs window end), **consumer lag** (the processing-time-side cousin, [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §11), and **side-output volume per period**. The essential property all of these share: they are *ratios and durations over time*, trended and alerted on — not one-off checks.

A worked computation makes the definitions crisp. A corridor-hour window receives 1,000 events whose event time falls inside it. When the watermark passes window-end: 992 events had arrived at-or-ahead of the watermark (**on-time rate 99.2%**), and 8 arrived behind it (**late rate 0.8%**). Of those 8, 6 arrive within the 24 h allowance and re-fire the window with corrections (**in-allowance late — the finality metric tracks how often this happens**), and 2 arrive beyond it and land in the side output (**side-output rate 0.2% — the number the reconciliation job consumes**). The completeness check runs at T+1: the warehouse fact table holds 1,000 rows for the window — 992 on-time + 8 late, all absorbed (6 in real time, 2 via the nightly redrive) — **completeness 100%**, achieved by the batch half even though the streaming on-time rate was 99.2%. Every number in that story is a metric from this table; every one of them is an SLA input in §7.2.

### 7.2 The Data SLAs

A **data SLA is a measurable, contracted commitment to downstream consumers about a data product's delivery** — the data-reliability-engineering practice of turning "the data is fine" (an opinion) into "completeness ≥ 99.9%, freshness ≤ 15 min" (a metric). Data-pipeline SLA practice (verified against the data-reliability literature, e.g., dbt Labs' data-SLA guidance and streaming-SLA references such as Conduktor's) centers on four dimensions, each of which maps onto this guide:

| SLA dimension | What it commits | Late-data mapping |
|---|---|---|
| **Freshness / latency** | How recent the data is (batch: T+1 by 06:00; streaming: p99 latency ≤ seconds) | Watermark design + drift budget (§2–§3) |
| **Completeness** | What fraction of expected events are present | Side-output rate + batch reconciliation + backfill (§4–§5) |
| **Accuracy** | Results match ground truth | Retraction/correction discipline + restatement (§4.3, §5.2) |
| **Quality / validity** | Schema, constraints, referential integrity | Warehouse merge + SCD discipline (§5, §8.2) |

Streaming SLAs are distinct from batch SLAs in the dimension they measure: batch SLAs are about *schedule and completeness of a run* (the file/table arrived, complete, by the cutoff); streaming SLAs are about *continuous tail behavior* — p99/p99.9 latencies, late-rate ceilings, and completeness attainment over rolling windows, because a stream has no single "run" to check. The late-data-relevant commitments look like: *"≥ 99.5% of window results final within watermark + 5 min; ≥ 99.9% of events accounted for by T+1 reconciliation; late-rate < 1% rolling daily."* (Exact numbers are illustrative — §11 flags them; the *shape* of the SLA — a latency tail, a completeness floor, a late-rate ceiling — is the practice.)

The final SLA principle: **an SLA without a measurement is a hope** — which is why §7.1's metrics are the enforcement mechanism, and why this guide's summary ends where it does.

An SLA without a breach workflow is a wish. The late-data-shaped breach response, in order:
1. **Detect** — the metric trips (late-rate > ceiling, side-output volume spike, T+1 completeness below floor). Alerting is on the *ratio and the trend*, not a single event.
2. **Triage** — is this a *data* problem (a source stopped sending — check source health, consumer lag, connector state) or a *design* problem (the watermark strategy no longer matches the arrival profile — a new correspondent with slow files changed the tail)? The two fixes are different: operational recovery vs watermark redesign (§9.2).
3. **Contain** — the side output and raw log guarantee nothing is lost while the fix proceeds (bounded, measured exposure — §4.2); for batch, the affected period is quarantined from publication until reconciled.
4. **Correct** — re-run the affected windows/period: in-allowance events already corrected in real time; beyond-allowance absorbed by the side-output redrive or backfill (§5.3).
5. **Restate and review** — publish corrected numbers with change visibility (§5.2), update the SLA attainment record, and feed the root cause back into watermark/allowed-lateness tuning. The SLA's attainment number is the honest summary of how often this loop ran.

### 7.3 The Measurement Table

| Metric | Definition | Typical target (illustrative) | Where it's enforced |
|---|---|---|---|
| **On-time rate** | Events with event time ≥ watermark at arrival / total | ≥ 99% (per window, rolling) | Source operators, watermark-aware counters |
| **Late rate** | Events with event time < watermark at arrival / total | < 1%; subdivided into in-allowance vs side-output | Side-output tagging (§4.2) |
| **Completeness** | Expected events accounted for / expected (per period) | ≥ 99.9% by T+1 reconciliation | Batch reconciliation vs source counts (§5.3, §6.2) |
| **Watermark age** | now − watermark | ≤ out-of-orderness budget + tolerance | Pipeline metrics (Flink/Dataflow) |
| **Freshness/latency** | p99 event → result-visible time | Seconds for streaming; T+1 for batch | SLA monitoring (§7.2) |
| **Side-output volume** | Events beyond allowance per period | Trending flat; spike = watermark redesign trigger | Side-output landing + alerting |
| **Restatement rate** | Published numbers corrected per period | Low and falling; each one investigated | Warehouse amendment/restatement tracking (§5.2) |

---

## 8. The Design Patterns: Kappa/Lambda, Upserts

### 8.1 The Lambda and Kappa Architectures

The two canonical architectural answers to "how do I get both real-time results *and* correct historical results?" — and the late-data problem is the reason both exist:

**Lambda architecture** (Nathan Marz, c. 2011): two parallel paths over the same data — a **batch layer** that computes complete, correct results from all data (slowly), and a **speed layer** that computes approximate real-time results from recent data (quickly); a **serving layer** merges both. Late data is handled by the batch layer: whatever the speed layer missed, the next batch run corrects. The documented cost: **two codebases that must produce the same results and be kept in sync** — the speed layer is, by design, a lossy/approximate pre-computation of the batch layer, and reconciling the two is a permanent maintenance tax (this is the critique the kappa proponents cite, e.g., Jay Kreps' 2014 post and subsequent industry commentary — verified as the standard framing).

**Kappa architecture** (Jay Kreps, 2014): **one path** — all data flows through a single stream-processing pipeline over the immutable log; "batch" is just stream processing re-run over historical log data (replay = reprocessing, §5.3). Late data is handled by the same machinery as everything else: watermarks, allowed lateness, side outputs — and, when the watermark design itself was wrong, replay the log with a corrected strategy. Cost: the log must retain/replay data for the longest reprocessing horizon, and the *streaming* engine must be capable of full recomputation (stateful reprocessing at scale is not free — the "kappa is just lambda with one codebase" nuance is real: you still need a batch-speed equivalent, you've just unified the code).

For late-arriving data specifically, the architecture choice is a completeness-strategy choice: **lambda guarantees completeness by periodic recompute; kappa guarantees it by replay-with-same-code.** In practice, most financial platforms run a hybrid: kappa-style streaming for the real-time layer (Flink/Kafka Streams over the log) *plus* a warehouse batch layer for the T+1 reconciliation and regulatory reporting — which is, honestly, lambda with the batch layer as the completeness backstop rather than a parallel computation of the same query. (The [event_stream_processing_guide.md](event_stream_processing_guide.md) §2 covers the same evolution; this guide's contribution is the late-data lens.)

The practical selection table (banking context):

| Situation | Architecture | Why |
|---|---|---|
| Regulatory T+1 reporting + real-time ops | Hybrid (kappa streaming + batch warehouse backstop) | Completeness guaranteed by batch; latency bounded by streaming — the §9 design |
| Pure real-time with replayable log and no batch-reporting consumer | Kappa | One codebase; reprocessing = replay |
| Legacy batch estate, stream is a new add-on | Lambda-style (batch stays the source of truth) | The speed layer is explicitly approximate; batch recompute is the correction path |
| Exactly-once-style correctness for downstream stores | Either + upsert changelog (§8.2) | Correctness comes from the materialization pattern, not the architecture label |
| Small team, one codebase, limited ops | Kappa | Two codebases' reconciliation tax is a real operating cost (the lambda critique) |

### 8.2 The Upsert Patterns: Merge

Wherever late data lands — stream output store, warehouse fact table, CDC target — the write pattern must be **upsert, not append**: the late event *corrects or supersedes* a row that may already exist, and the target must reconcile by business key. The mechanisms, all standard and verified across warehouse/lakehouse documentation (Snowflake/Delta/BigQuery `MERGE`; CDC-pipeline guidance):

- **`MERGE` (SQL)** — `MERGE INTO target USING source ON target.key = source.key WHEN MATCHED THEN UPDATE WHEN NOT MATCHED THEN INSERT` — the warehouse-native upsert; idempotent (re-running the same source produces the same target), which is what makes backfill safe (§5.3).
- **Changelog materialization** — for streaming output: apply the retraction/insert changelog (§4.3) to a keyed store (Flink's keyed state → a sink table; a Kafka-compacted topic; a key-value store). The changelog *is* the upsert stream — each key's latest value is the current state.
- **CDC merge** — the CDC target (§6.2) is exactly a merge: `INSERT`/`UPDATE`/`DELETE` events applied by natural key, with commit timestamps as the versioning signal.
- **Dedupe keys** — the at-least-once companion: an idempotency key on the event (or the natural key itself) so replays and redeliveries collide with existing rows instead of duplicating ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §6).

The rule: **append-only is for the raw log; everything derived is upsert.** Late data is the reason.

The canonical shape, in warehouse SQL (Snowflake/Delta/BigQuery all support this form):

```sql
MERGE INTO payment_fact f
USING late_payments_stage s
  ON f.payment_id = s.payment_id
WHEN MATCHED THEN UPDATE SET f.status = s.status, f.amount = s.amount,
                             f.updated_at = s.updated_at
WHEN NOT MATCHED THEN INSERT (payment_id, status, amount, event_time, updated_at)
     VALUES (s.payment_id, s.status, s.amount, s.event_time, s.updated_at);
```

The `ON` clause is the whole point: the merge matches on the *business key* (`payment_id`), so a late event — whether it is a correction to an existing row or a brand-new row the stream missed — lands in exactly one row, and re-running the merge (backfill, redelivery) changes nothing that is already correct. Idempotency by construction, which is the property that makes every replay in §5.3 safe.

### 8.3 The Pattern Table

| Pattern | Problem it solves | Solution |
|---|---|---|
| **Lambda** | Need both fresh and complete results | Batch layer (complete) + speed layer (fresh) + serving merge; completeness by recompute |
| **Kappa** | Two codebases drift; reprocessing should reuse code | Single streaming path over the log; reprocessing = replay with same code |
| **Hybrid (common in banking)** | Real-time ops + regulatory T+1 completeness | Kappa-style streaming + warehouse batch reconciliation as backstop |
| **Upsert / MERGE** | Late events must correct, not duplicate | Keyed merge (SQL `MERGE`, changelog materialization, CDC merge) — idempotent writes |
| **Placeholder dimension** | Fact arrives before its dimension row | Load vs "unknown" key; re-point or version when the row arrives (SCD1/2) |
| **Side-output + backfill** | Bounded-loss escape hatch for the unpredictable tail | Route beyond-allowance events out; absorb them via replay/reconciliation |

---

## 9. The Worked Example: A Banking Cross-Border Payment Stream

### 9.1 The Scenario: Late Events in a Payment Stream

A bank runs a **cross-border payment stream**: every payment emits events — `PaymentInitiated` (customer/API), `SanctionScreened`, `PaymentSent` (to the correspondent), `ConfirmationReceived` (from the correspondent/SWIFT), `ValueDateSettled`. The business needs: **real-time intraday liquidity and settlement-amount totals per corridor per hour** (streaming), plus **T+1 regulatory reporting and reconciliation** (batch).

The stream is a *designed mixture of on-time and late sources* — the late-data problem is not hypothetical here, it is structural:

| Source | Event time | Typical lateness | Late mechanism |
|---|---|---|---|
| API gateway (initiation, screening) | Payment timestamp (ms) | Seconds | Retries after broker hiccups ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §3) |
| SWIFT MT103/MT202 confirmations | Message send/receive time | Hours | Correspondent banks batch and send EOD files; files can be re-issued |
| Nostro statements | Value/statement date | Hours–days | Statement cycles, holidays, correspondent delays |
| Operations amendments/reversals | Original payment timestamp | Hours–days (unbounded) | Human corrections — see [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md); retro-dated postings, [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md) |

The payments stream is therefore *two streams with different lateness profiles*: the real-time API stream (bounded, seconds-to-minutes lateness) and the correspondent-file stream (bounded but coarse, hours-to-days lateness), plus a correction stream with *unbounded* lateness. **Designing one watermark for all three is the classic mistake; the design below separates them.**

A concrete late payment walks through the design. **Payment P-7731**: a customer initiates a EUR→SGD transfer at **10:00:00** (event time). The API stream carries `PaymentInitiated` at 10:00:03 (on time, §9.2 stream 1). Sanction screening clears at 10:00:11. The payment is sent to the correspondent at 10:00:40. Then silence — the corridor's confirmation channel has a slow night. At **14:20:00** the SWIFT confirmation arrives (event time 10:00:40 — the *send* time, per §9.2 stream 2's file-boundary watermarking): the 10:00 corridor-hour window fired at 10:05 with P-7731 counted as `sent, unconfirmed`; the confirmation is 4 h late but *inside the 24 h allowed lateness*, so at 14:20 the window re-fires and the corridor's hourly "confirmed notional" retracts the unconfirmed contribution and adds the confirmed one (§4.3, §9.3). At **T+2**, operations posts `PaymentAmended` (a fee correction, event time 10:00:00, arrived 2 days later): beyond allowed lateness, it flows to the `late-payments` side output, lands in the object store, and the nightly reconciliation re-ingests it into the warehouse as an amendment row (§5.2) — the streaming totals for 10:00 stay as-is, the warehouse fact row grows a `corrected_by` chain, and T+1 reporting reconciles. Every mechanism in §2–§5 fired exactly once, in the order the design intended.

### 9.2 The Watermark Design: The Values Sketch

Three watermark strategies, one per lateness profile (values are a sketch — the tuning method matters more than the numbers):

1. **API stream (bounded, fine):** `forBoundedOutOfOrderness(Duration.ofMinutes(5))` — out-of-orderness absorbs retries and intra-broker disorder (5 min is generous for an API stream; tuned down by observing late-rate, §7.1). Periodic generation at the Flink default interval (200 ms). Kafka connector: **per-partition watermarks, min-aligned across the 12 payment partitions**, with `withIdleness(Duration.ofMinutes(2))` so a quiet corridor doesn't stall the watermark (§3.1, §3.3).
2. **SWIFT/statement file stream (coarse):** file-level event time = the file's statement/message date; **punctuated watermarks** (§3.2) — each completed file emits a watermark at its max event time, because file completion is a *precise* completeness signal. Subtract a 24 h bound for re-issued files and statements that arrive next day. (Punctuated is the natural choice: the source *knows* when a file is complete, which is the definition of an exact watermark, §3.1.)
3. **Correction stream (unbounded):** *no* watermark-based windowing at all. Corrections bypass the window machinery and flow directly to the upsert/amendment path (§9.3) — you cannot watermark human behavior.

The sketch's numbers, put on one line: **5 min out-of-orderness on API events, file-boundary watermarks on correspondent data, 24 h allowed lateness on settlement windows, and a side-output + T+2 batch path for everything later.** This is the §3.4 table applied to a real stream.

### 9.3 The Late Handling: Side Output + Correction Design

- **Side output.** Settlement windows use `allowedLateness(24h)` (a corridor's confirmation can legitimately arrive next day). Events beyond 24 h — the tail of human corrections and stale statements — go to a **`late-payments` side output** (Flink `sideOutputLateData`), which lands to a Kafka topic *and* object storage with original event times intact. A nightly reconciliation job (the DLQ-redrive cousin, [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §7) replays the side output against core-banking records and re-ingests genuine items with full event-time fidelity. The side output is measured: **side-output volume per corridor per day is a published metric** — a growing tail means the 24 h allowance or the file watermark needs redesign, not more ops heroics.
- **Correction design (retraction + upsert).** An operations team fixes a payment: `PaymentAmended(payment_id, supersedes=original_event, new_amount)`. In the streaming aggregate, the amendment lands as a **late event within the 24 h allowance → the corridor-hour window re-fires with a retraction of the original contribution and the corrected value** (Flink Table changelog / Beam accumulating-&-retracting, §4.3). Downstream, the materialized store (a keyed sink table per payment_id) applies the changelog — the payment row's amount is updated, the window total is corrected. In the warehouse, the same amendment lands as **an upsert by `payment_id` with a `corrected_by` chain** — the ledger never erases: original row + amendment rows, aggregates sum originals + adjustments (§5.2), and the payment's *status* dimension is versioned (SCD Type 2, §5.1) so "what did we know on day X" queries work for audit and disputes.

The design handles every case in §1.5: in-allowance lateness is corrected in real time (retraction), beyond-allowance lateness is preserved and reconciled (side output), corrections are visible facts (amendment chain), and nothing is silently dropped.

The before/after numbers for the corridor-hour window (the retraction above made visible):

| Emission | Trigger | Corridor 10:00 total | Correct? |
|---|---|---|---|
| 10:05:00 | Watermark passes 10:00 (first fire) | 1,000,000 (P-7731 counted unconfirmed) | Provisional |
| 14:20:00 | P-7731 confirmation arrives, in-allowance | 1,000,150 (retract 1,000,000; add confirmed) | Provisional (no further data expected) |
| T+2 06:00 | Nightly reconciliation absorbs side output (amendment) | Warehouse fact = 1,000,150 + fee-adjustment row | Final for T+1 reporting |

The column that matters is the last: **the streaming total was "wrong" (provisional) for 4 h 15 min, and the warehouse was right by T+2** — which is precisely what the SLA in §9.4 commits: finality ≥ 99% within watermark + 5 min for the *typical* event, and 100% completeness by T+1 via the batch half. The pipeline did not fail; it behaved per design, and the measurement (§7) is what lets the bank say so.

### 9.4 The Completeness SLA: The Numbers, Flagged

The SLA for the stream, with the shape of the practice and **numbers flagged as illustrative** (see §11 — set them from your own measurement):

- **On-time:** ≥ 99.5% of API-stream events arrive at or ahead of watermark (rolling 1 h windows). Measured: watermark-aware counters (§7.1).
- **Finality:** ≥ 99% of corridor-hour settlement totals are final within watermark + 5 min (i.e., no further in-allowance corrections) — the number that tells operations whether 24 h of allowed lateness is actually being used.
- **Completeness by T+1:** **100%** of payments with event time in day D are accounted for (count and notional) by the T+1 06:00 reconciliation run — where "accounted for" means present in the warehouse fact table, including everything the stream side-outputted. This is the SLA that matters to regulators, and it is met *by the batch half* (reconciliation + backfill), not by the stream — the honest framing: the stream bounds the latency of being wrong; the batch guarantees being right.
- **Late-rate ceiling:** < 1% of events arrive after the watermark (rolling daily); side-output rate trending flat. Breach = watermark redesign trigger.

The numbers are deliberately *checkable*: on-time rate, finality, T+1 completeness, and late-rate are the four numbers this guide's measurement section (§7) exists to produce. **An SLA you cannot measure is not an SLA — and the first version of this table is where you start the negotiation with the business.**

### 9.5 The Lessons: Plan for the Late, Measure the Late

1. **The late data will come; design for it before it does.** Separate the stream by lateness profile (API vs files vs corrections), pick a watermark strategy per profile, set allowed lateness from the *measured* tail — not from optimism. The two-clock confusion and the out-of-order reality are features of the world, not bugs you fix away.
2. **Give the beyond-allowance tail a first-class home.** Side output + landing + reconciliation job — the streaming DLQ. "Drop silently" is not a strategy for financial data.
3. **Make corrections visible facts.** Retractions in the stream, amendments in the ledger, restatements in reports. Never silently overwrite a published number.
4. **Guarantee completeness in batch, bound latency in streaming.** Watermarks make the stream *mostly right, fast*; reconciliation and backfill make the warehouse *exactly right, eventually*. Both halves are mandatory.
5. **Measure the late.** On-time rate, late rate, side-output volume, watermark age, T+1 completeness — trended, alerted, and used to retune the watermark. The pipeline that measures lateness is the pipeline that can *explain* its numbers to a regulator; the one that doesn't is the one that discovers them in a break.

---

## 10. The Summary: Late Data in One Page

**Late data is an event that arrives after its event time — after the pipeline's completeness estimate (the watermark) has moved past it.** It is caused by the two clocks (event time vs processing time) diverging under network delay, retries, batching, partitioning, and human correction — i.e., by the normal messiness of the world. It matters because time-based computation — windows, aggregates, joins, SLAs — is *wrong* when events are missing or mis-bucketed: incomplete numbers and incorrect attribution.

**The streaming response is a stack of four mechanisms.** *Event-time processing* makes lateness definable and correctable. *Watermarks* — estimates of "all events ≤ T have (probably) arrived," periodic or punctuated, min-aligned across partitions, prone to drift when sources idle — decide *when windows fire*. *Allowed lateness* decides *which late events still count*: within the allowance, the window re-fires with a corrected value; beyond it, the event goes to a *side output* — the streaming DLQ — preserved, measured, and redriven. *Retractions and corrections* express the change: the engine withdraws the old value (negative row, changelog, accumulating-and-retracting) and inserts the new one; the business sends amendments and reversals as new facts that supersede the old.

**The batch response completes the story.** In the warehouse, late-arriving dimensions meet the SCD family (overwrite, versioned rows, original/current columns) with placeholder keys for facts that arrive before their dimensions; corrections are posted as amendments and restatements, never silent overwrites; and backfill — replaying the immutable raw log with the same code — is the final safety net that makes the warehouse complete by T+1 no matter how late anything arrived. CDC streams add their own flavor: log-based change events whose event time is the commit time, handled by key-based ordering, idempotent merge, and reconciliation.

**The architecture question** (lambda's two-codebase recompute vs kappa's replay-with-one-codebase) is really a completeness-strategy question, and the common banking answer is a hybrid: kappa-style streaming for real time, a warehouse batch layer for the T+1 guarantee. **Everywhere derived state is written, it is upsert — `MERGE` by business key, idempotent, so late events correct instead of duplicate.**

**And none of it works unmeasured.** On-time rate, late rate, watermark age, side-output volume, T+1 completeness — trended and SLA'd — turn "we handle late data" from a claim into a number.

The one-page cheat sheet (the guide in twelve lines):

| When you see… | Do this | Where |
|---|---|---|
| A new stream with sources of varying speed | Measure the lateness spectrum first; then choose watermarks per profile | §1.5, §9.2 |
| A windowed aggregate that must be right | Event-time windows + watermark + allowed lateness; expect re-fires | §2, §4.1 |
| A source that can tell you "this batch is complete" | Punctuated/exact watermarks at the boundary | §3.2 |
| A silent partition stalling results | Idle-source handling (`withIdleness`) | §3.1, §3.3 |
| Events beyond your tolerance | Side output — land, measure, reconcile; never drop silently | §4.2 |
| A corrected event | Retraction in the stream; amendment/restatement in the ledger | §4.3, §5.2 |
| A fact without its dimension | Placeholder key + SCD1/2 re-point | §5.1 |
| A wrong result already published | Backfill with idempotent upserts; restate visibly | §5.3, §8.2 |
| CDC changes flowing downstream | Key-based order + merge by natural key; reconcile | §6 |
| A claim that "we handle late data" | Ask for the late-rate, on-time-rate, and T+1 completeness numbers | §7 |
| An architecture decision | Kappa/hybrid for one codebase + batch backstop; upsert everywhere derived | §8 |
| Any of the above in a bank | Plan for the late, measure the late | §9.5, §10 |

The whole guide in one sentence: **late data is unavoidable; what is avoidable is being surprised by it — so plan for the late, measure the late.**

---

## 11. Verification and Claims-Status

This guide's factual claims were verified against primary documentation and industry sources during research (August 2026). Status per claim cluster:

| Claim | Status | Source |
|---|---|---|
| Late element definition (timestamp behind watermark at arrival) | **Verified** — Flink and Dataflow docs define late data identically (event time behind the watermark) | Apache Flink docs (Event Time / Allowed Lateness); Google Cloud Dataflow docs (Watermarks & Late Data) |
| Event time vs processing time; event time is the only correct clock for time-based analysis | **Verified** | Flink docs; Dataflow Model paper (Akidau et al., 2015); DDIA Ch. 11 |
| Watermark = completeness estimate, "not a guarantee" (Flink) / lower bound on future event times (Dataflow) | **Verified** — both definitions; the heuristic-vs-exact nuance is real and flagged | Flink docs ("Generating Watermarks"); Dataflow docs; Begoli et al., *Watermarks in Stream Processing Systems* (VLDB 2021) |
| Flink watermark generation: `BoundedOutOfOrderness` = max observed event time − bound; min-alignment across partitions; idle-source handling (`withIdleness`); default auto-watermark interval 200 ms | **Verified** | Flink docs (Watermark Strategies; Dealing with Idle Sources); Confluent Flink course; strategy source semantics |
| Periodic vs punctuated watermark generation | **Verified** — two `WatermarkGenerator` styles documented | Flink docs ("Periodic Watermark Generation" / "Punctuated Watermark Generation") |
| Allowed lateness: window state retained until watermark > window-end + allowance; re-fire/update; default 0 | **Verified** (Flink `.allowedLateness`); Beam `withAllowedLateness` equivalent | Flink docs (Allowed Lateness); Beam docs |
| Side output for late data (`sideOutputLateData`); Dataflow "write late data to a separate location" | **Verified** | Flink docs (Side Outputs); Dataflow docs (Handling late data) |
| Retraction semantics: Flink Table/SQL changelog emits update-before (negative) + update-after; Beam "accumulating & retracting" | **Verified** | Flink Table API docs (retract mode); Beam trigger accumulation modes; Confluent streaming-SQL docs |
| SCD Type 1 (overwrite), Type 2 (versioned rows, surrogate keys, effective dates), Type 3 (original/current columns); late-arriving-dimension placeholder practice | **Verified** | Kimball Group ("Slowly Changing Dimensions"); standard dimensional-modeling references (flagged: practitioner-consensus wording) |
| Backfill/reprocessing via log replay; idempotency required; lakehouse time-travel/MERGE as the batch machinery | **Verified** (pattern) | DDIA Ch. 11; Delta/Iceberg docs; microservices/data-engineering consensus |
| CDC: Debezium log-based capture; event time = DB commit time; per-key partitioning/order; snapshot+incremental dedupe; **watermarking CDC streams is practitioner practice, not a documented Debezium feature** | **Verified** (CDC fundamentals); **flagged** (watermarking CDC = engineering consensus, not vendor-documented) | Debezium docs; CDC-pipeline practitioner guidance |
| Data SLAs: freshness/latency/completeness/accuracy dimensions; streaming SLAs measured on tails (p99), not schedules | **Verified** (as practice) — exact SLA numbers in §9.4 are **illustrative, flagged** | dbt Labs (data SLAs/SLOs); Conduktor (SLAs for streaming); data-quality KPI practice |
| Lambda vs kappa: two-codebase reconciliation cost vs single-pipeline replay; kappa credited to Jay Kreps (2014) | **Verified** (standard framing) | Kreps' kappa post; industry surveys (Streamkap, GeeksforGeeks); [event_stream_processing_guide.md](event_stream_processing_guide.md) |
| Upsert/MERGE as the idempotent write pattern for late data and CDC targets | **Verified** | Snowflake/Delta/BigQuery `MERGE` docs; CDC-pipeline guidance |
| "Plan for the late, measure the late" as the guide's synthesis | **Flagged as this guide's recommendation** — consistent with the sources, but a synthesis, not a citation | This guide (§9.5, §10) |

**Flagged/unverifiable items:** (1) the exact default auto-watermark interval in all Flink versions (docs state 200 ms — check the version in use); (2) Dataflow's exact-vs-estimated watermark behavior varies by source connector — treat per-source; (3) watermarking CDC streams is documented nowhere canonically — treated here as engineering consensus; (4) all numeric targets in §7.3/§9.4 are illustrative; (5) "never silently overwrite a published number" is a banking-domain regulatory-adjacent convention, not a codified standard — adopt per your control framework.

## 12. References

- **Apache Flink documentation** — *Event Time*; *Generating Watermarks* (Watermark Strategies; Periodic/Punctuated WatermarkGenerator); *Allowed Lateness*; *Side Outputs*; *Dealing with Idle Sources*; *Table API — retract mode / dynamic tables*. (flink.apache.org / nightlies.apache.org)
- **Google Cloud Dataflow documentation** — *Watermarks*; *Handling Late Data*; *Triggers*. Also: Akidau et al., *The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing* (VLDB 2015).
- **Begoli, Akidau, et al.** — *Watermarks in Stream Processing Systems: Semantics and Comparative Analysis* (VLDB 2021) — the Flink/Dataflow watermark comparison.
- **Apache Beam documentation** — *Windows*; `withAllowedLateness`; *Trigger accumulation modes* (discarding / accumulating / accumulating-and-retracting).
- **Confluent** — *Windowing and Watermarks in Flink* (course); *Handle Late-Arriving Data in Confluent Cloud for Apache Flink* (how-to guide).
- **Kimball Group** — *Slowly Changing Dimensions* (kimballgroup.com); Ralph Kimball & Margy Ross, *The Data Warehouse Toolkit* (dimensional modeling, late-arriving dimensions).
- **Jay Kreps** — *Questioning the Lambda Architecture* (O'Reilly Radar, 2014) — the kappa architecture origin; Nathan Marz, *Big Data* (2011–2015) — lambda.
- **Debezium documentation** — *Change Data Capture*; connector ordering/offset semantics (debezium.io).
- **Data-reliability / SLA practice** — dbt Labs, *Data SLAs and SLOs* (getdbt.com); Conduktor, *SLAs for Streaming*; data-quality KPI/scorecard practice (completeness as a quality dimension).
- **Warehouse/lakehouse merge** — Snowflake `MERGE`, Databricks Delta `MERGE`, BigQuery `MERGE` documentation.
- **Practitioner late-data patterns** — Software Patterns Lexicon, *Aggregation with Late Data*; oneuptime, *Late Data Handling* (allowed lateness, side outputs, retractions — the practitioner vocabulary this guide adopts); Confluent Cloud for Apache Flink, *Handle Late-Arriving Data* (how-to).
- **Apache Kafka documentation** — per-partition ordering (the ordering domain that out-of-order reasoning builds on).
- **Domain cross-references (this repo):** [event_stream_processing_guide.md](event_stream_processing_guide.md), [complex_event_processing_guide.md](complex_event_processing_guide.md), [ddia_study_companion_guide.md](ddia_study_companion_guide.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md), [data/delta_lake_vs_iceberg.md](data/delta_lake_vs_iceberg.md), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md), [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md), [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md), [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md), [banking/financial_fraud_detection_at_scale_guide.md](banking/financial_fraud_detection_at_scale_guide.md).

---

## 13. Glossary

- **Late data / late-arriving data** — an event that arrives at a processing system after its event time; more precisely, an event whose timestamp is behind the pipeline's watermark at arrival (§1.1).
- **Event time** — when an event actually happened, per the timestamp the source embedded in it; the only clock that answers "what happened when?" (§1.3).
- **Processing time** — when the pipeline observed/processed the event; the local machine clock, monotonically increasing but meaningless for historical accuracy (§1.3).
- **Out-of-order** — arrival order differing from event-time order; the empirical form of the late-data problem, caused by networks, retries, batching, and partitioning (§1.4).
- **Watermark** — the pipeline's running estimate of event-time progress: "all events ≤ T have (probably) arrived"; the trigger that fires windows; a heuristic, not a guarantee (§2.2, §3).
- **Allowed lateness** — the event-time threshold (relative to the watermark) within which a late event still updates its window's result; beyond it, the event is side-output or dropped (§2.3, §4.1).
- **Side output** — a separate stream receiving events that arrived beyond allowed lateness; the streaming DLQ — preserved, measured, redriven (§4.2).
- **Retraction** — withdrawing a previously emitted value (negative row / update-before) so downstream sums stay correct when a window result is corrected (§4.3).
- **Correction** — a new business event (amendment, reversal, adjustment) that supersedes an earlier event's effect; the semantic layer on top of retraction (§4.3, §5.2).
- **SCD / slowly changing dimension** — the Kimball pattern family for dimension attributes that change over time: Type 1 (overwrite), Type 2 (versioned rows), Type 3 (original/current columns) (§5.1).
- **Backfill** — re-running a computation over historical data to produce corrected/complete results; safe only with idempotent writes (§5.3).
- **Reprocessing** — re-applying a pipeline (typically by replaying the log) after a fix; the kappa path to "batch" (§5.3, §8.1).
- **CDC / change data capture** — extracting row-level database changes from the transaction log and streaming them as events; event time = DB commit time (§6.1).
- **On-time** — an event arriving at or ahead of the watermark; the numerator of the on-time rate metric (§7.1).
- **Late** — an event arriving behind the watermark; subdivided into in-allowance (recoverable) and beyond-allowance (side-output) (§7.1).
- **Completeness** — the fraction of expected events actually accounted for, per period; the quality dimension that late data most threatens (§7.1).
- **Data SLA** — a measurable commitment to consumers covering freshness/latency, completeness, accuracy, and quality; an SLA without measurement is a hope (§7.2).
- **Lambda architecture** — batch layer (complete) + speed layer (fresh) + serving merge; two codebases to reconcile; completeness by recompute (§8.1).
- **Kappa architecture** — a single streaming path over the immutable log; reprocessing = replay with the same code (§8.1).
- **Upsert** — insert-or-update by business key; the idempotent write pattern that lets late data correct instead of duplicate (§8.2).
- **Merge** — the SQL upsert (`MERGE INTO ... WHEN MATCHED THEN UPDATE ...`), the warehouse-native form of the upsert (§8.2).
- **Flink** — Apache Flink, the open-source stream processor whose watermark/late-data machinery (allowed lateness, side outputs, retract changelogs) is this guide's streaming reference (§2–§4).
- **Dataflow** — Google Cloud's managed stream processor, whose watermark/late-data model (exact vs estimated watermarks, late-data handling) is the other canonical reference (§2–§3).
- **Window / event-time window** — a grouping of events by event-time range (tumbling/sliding/session) that fires when the watermark passes its end and re-fires for in-allowance late data (§2.4).
- **Watermark drift** — the gap between the watermark and the true event-time frontier; the mechanism by which lateness becomes visible (§3.3).
- **Watermark strategy** — the per-source configuration (out-of-orderness bound, periodic/punctuated style, idle handling) that sets the drift budget (§3.4).
- **Late stream** — the side-output stream of beyond-allowance events, landed and reconciled rather than dropped (§4.2).
- **Placeholder dimension** — a default/"unknown" dimension row used when a fact arrives before its dimension; re-pointed or versioned when the real row arrives (§5.1).
- **Restatement** — publishing a corrected number with the change visible, never silently overwriting a published result (§5.2).
