# Complex Event Processing (CEP): A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Event-Driven Architecture / Real-Time Analytics — Banking, Financial Services, Fraud & AML, Trade Surveillance  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is Complex Event Processing?](#1-what-is-complex-event-processing)
2. [CEP vs Adjacent Technologies](#2-cep-vs-adjacent-technologies)
3. [Core CEP Concepts](#3-core-cep-concepts)
4. [The CEP Lifecycle](#4-the-cep-lifecycle)
5. [CEP Engines and Platforms](#5-cep-engines-and-platforms)
6. [CEP Pattern Catalog for Banking](#6-cep-pattern-catalog-for-banking)
7. [Implementing CEP in Banking](#7-implementing-cep-in-banking)
8. [CEP vs Alternative Detection Approaches](#8-cep-vs-alternative-detection-approaches)
9. [Operational Considerations](#9-operational-considerations)
10. [Engine Selection Guide](#10-engine-selection-guide)
11. [Anti-Patterns and Pitfalls](#11-anti-patterns-and-pitfalls)
12. [Future Directions](#12-future-directions)
13. [Conclusion](#13-conclusion)

---

## 1. What Is Complex Event Processing?

Complex Event Processing (CEP) is the real-time processing of event streams to detect patterns, correlations, and situations that span multiple events. Where a simple event processor reacts to a single event — "a card transaction of $500 was declined" — a CEP engine combines many events across time to answer a different class of question: *what is happening across the event flow?* "This card has been used at three merchants in four different countries within ten minutes." "A login from a new device was followed by a password change and then a large transfer within 24 hours." "Ten deposits just under the reporting threshold arrived in the same account within an hour, summing to over $90,000."

These situations cannot be seen in any single event. They emerge only when events are related to each other — by ordering, by time, by shared attributes, by cause and effect. CEP is the discipline and the technology for making those relationships explicit and acting on them in milliseconds to seconds.

### 1.1 Definition and Origins

CEP was formalized in the academic world of the 1990s, most prominently by **David Luckham** at Stanford University, whose 2001 book *The Power of Events* gave the field its name and its core vocabulary: events, event streams, patterns, and the idea of processing events at multiple levels of abstraction (raw events → derived events → situations). Around the same period, active-middleware research at MIT and the University of Vienna explored distributed event-based systems and rule-based event correlation, laying the groundwork for the commercial CEP engines (TIBCO, Apama, StreamBase) that emerged in the early 2000s for capital markets and algorithmic trading.

Three foundational ideas distinguish Luckham's framing from ordinary stream processing:

- **Events are the unit of meaning.** An event is a fact — something that happened at a point in time — and the system's job is to reason about collections of facts, not individual records.
- **Patterns express knowledge.** A pattern is an executable encoding of a domain expert's understanding: "when X happens, then Y, within T, unless Z." CEP makes domain knowledge directly runnable.
- **Hierarchies of events.** Raw events aggregate into derived events (e.g., "velocity breach"), which participate in higher-level patterns ("account takeover attempt"). This layering is how CEP systems scale from transaction-level rules to enterprise-level risk situations.

CEP today sits in a crowded space — event stream processing frameworks, streaming databases, rule engines, and ML-based anomaly detection all overlap with it. Understanding the precise boundaries is essential for choosing the right tool, which is the subject of Section 2.

### 1.2 What CEP Detects: The Pattern Taxonomy

CEP engines detect a small set of pattern primitives that compose into arbitrarily complex situations:

| Primitive | Meaning | Banking example |
|---|---|---|
| **Sequence** | A then B then C, in order | Login → password change → large transfer |
| **Correlation** | A and B together (any order, related by key) | Two logins to one account from different IPs at the same time |
| **Absence / negation** | A without B within a window | Deposit received with no expected confirmation event |
| **Aggregation with threshold** | 3+ events in 5 minutes, sum(amount) > X | Structuring: many deposits under the reporting threshold |
| **Temporal constraint** | Within / after / before / between times | Transfer within 24h of credential change |
| **Causality** | A caused B (A precedes and relates to B) | Market order followed by own-account fill — front-running |

What makes these "complex" is that they require **state across events** (remembering what was seen), **temporal reasoning** (did B happen within the window after A?), and **relational matching** (do the events share an account ID, card, device, or IP?). None of this is expressible as a single-event rule or a stateless filter.

### 1.3 The Spectrum: SEP → ESP → CEP

Event processing is a spectrum of increasing complexity:

- **Simple Event Processing (SEP):** one event in, one action out. IF transaction.amount > 10,000 THEN flag. Stateless, no memory, sub-millisecond. This is the "event trigger" pattern — every rules engine and most API-based fraud checks start here.
- **Event Stream Processing (ESP):** continuous queries and aggregations over streams — sliding windows, rolling averages, joins, counts. "Average transaction value per merchant over the last 5 minutes." ESP is about *summarizing* the flow. Apache Flink, Kafka Streams, Spark Structured Streaming, and ksqlDB are ESP-first.
- **Complex Event Processing (CEP):** pattern detection *across* events — stateful, temporal, relational matching that produces a new derived event (the "complex event") when a pattern completes. CEP is about *recognizing situations* in the flow. Esper, Siddhi, and the Flink CEP library are CEP-first.

The boundaries are blurring. Modern engines deliberately cover both: Flink CEP combines a full stream-processing runtime with a pattern library; Esper has both an EPL pattern language and SQL-like windows; Siddhi embeds both in a lightweight library. A useful mental model: **ESP computes what the stream looks like; CEP computes what the stream means.** Most production banking systems need both, and increasingly they are built in one engine.

---

## 2. CEP vs Adjacent Technologies

CEP is routinely confused with three neighboring technologies. The distinctions matter because they change the architecture, the team skills required, and what the system can express.

### 2.1 CEP vs Event Stream Processing (ESP)

| Dimension | ESP | CEP |
|---|---|---|
| Core operation | Continuous queries, aggregations, joins | Pattern matching over ordered events |
| Canonical example | "Rolling 5-min average of trade size per symbol" | "Large order followed by cancel within 2 min = spoofing" |
| State | Windowed aggregation state | Per-key partial-match state (sequences in progress) |
| Time | Window boundaries, watermarks | Temporal constraints between events (within/after/between) |
| Output | Updated aggregates, transformed streams | Derived events, alerts, situation notifications |
| Typical latency | Sub-second to seconds | Sub-second to seconds |

The practical difference shows up in what is easy to express. A velocity check — "more than 5 transactions on this card in 5 minutes" — is a windowed count, natural in both ESP and CEP. A sequence — "transaction on card A at merchant X, *then* a transaction on the same card at merchant Y, *then* a cash withdrawal" — requires ordered, stateful matching across event types, which is CEP territory. Esper and Siddhi are CEP-first but ship ESP capabilities (windows, aggregations) as first-class citizens; Flink and Kafka Streams are ESP-first and require more code for pattern semantics — which is exactly why the Flink CEP library exists.

### 2.2 CEP vs Rule Engines (Drools)

Rule engines such as Drools evaluate IF-THEN rules against a working memory of facts. The two technologies overlap heavily at the surface — both express "when conditions hold, do something" — but they operate on different models of time and state. See the companion guide [drools_rule_engine_alternatives_guide.md](drools_rule_engine_alternatives_guide.md) for the full rule-engine landscape.

| Dimension | Rule engines (Drools) | CEP |
|---|---|---|
| Data model | Facts in working memory, retracted/updated | Immutable events on ordered streams |
| Time | Mostly timeless; Drools Fusion adds temporal operators | Time is fundamental: windows, intervals, delays |
| Ordering | Rules fire on fact insertion, no inherent ordering | Sequences and ordering are first-class pattern primitives |
| State | Rule-engine agenda, salience, truth maintenance | Per-key partial matches, window state, TTL |
| Expressiveness | Arbitrary IF-THEN logic, decision tables | Temporal + relational pattern algebra |
| Best fit | Business rules, eligibility, pricing, workflow decisions | Situations across time: fraud, surveillance, monitoring |

The overlap: **fraud rules can be implemented either way.** A rule engine with Fusion-style temporal operators ("transaction after password change within 24 hours") can express many CEP patterns; a CEP engine can express many rules. The deciding factors are stream-native ordering and temporal constraints (CEP wins), and integration with an existing decisioning stack (rules engines win). In banking, a common architecture is CEP for *detection* (finding situations in streams) and a rule/decision engine for *response* (deciding what action to take: block, decline, review, allow).

### 2.3 CEP vs Stream Processing Frameworks

General stream processing frameworks — Apache Flink, Kafka Streams, Spark Structured Streaming — are not CEP engines, but they can *host* CEP logic:

- **Apache Flink** ships the Flink CEP library (`flink-cep`) with a dedicated Pattern API, plus SQL `MATCH_RECOGNIZE`. Flink is the strongest open-source option for CEP at scale because the pattern engine runs on the same runtime that provides exactly-once, checkpointing, and horizontal scaling.
- **Kafka Streams** has no pattern language; stateful pattern detection is written by hand with the Processor API, state stores, and punctuators.
- **Spark Structured Streaming** provides event-time windows, watermarking, and `mapGroupsWithState`/`flatMapGroupsWithState` — pattern logic is coded, not declared.

The trade-off is expressiveness vs. control: a pattern language (EPL, the Flink Pattern API, MATCH_RECOGNIZE) declares *what* situation to detect and lets the engine handle ordering, state, and time; hand-coded state machines give full control but require the team to build and maintain the stateful logic themselves.

**Esper/Siddhi are CEP-first** — the pattern is the unit of deployment. **Flink/Kafka Streams are stream-processing-first** — the pattern is one operator inside a larger pipeline (enrichment, joins, sinks). Choose based on whether the pattern workload is the core of the system or one component of a broader streaming architecture.
## 3. Core CEP Concepts

### 3.1 Events and Event Streams

An **event** is an immutable fact: something that happened, at a time, with attributes. In CEP systems an event is modeled with at least three parts:

- **Timestamp** — when the event occurred (event time) vs. when it was processed (processing time). Getting these right is Section 3.6.
- **Type** — the event's semantic kind: `Transaction`, `Login`, `OrderPlaced`, `OrderCancelled`, `PriceTick`, `PasswordChanged`.
- **Attributes** — the payload: `account_id`, `amount`, `merchant`, `device_id`, `ip_address`, `instrument`.

Events are immutable by design: they are facts, and facts cannot be edited, only superseded by later events (a transaction is not "updated" — a reversal event arrives). This immutability is what makes streams replayable, checkpoints meaningful, and audit trails trustworthy.

An **event stream** is an unbounded, ordered sequence of events. "Ordered" is subtle: ordering may be by timestamp, by arrival, or by partition offset, and different consumers may legitimately see different orders. CEP engines impose a working order (usually event time within a partition, guarded by watermarks) and match patterns against that order.

### 3.2 Event Patterns

An event pattern is a template describing a situation in terms of temporal and logical relationships between events. The core operators, using a pseudo-notation and the Flink CEP naming:

| Operator | Notation | Meaning |
|---|---|---|
| **Sequence** | `A → B → C` (`next`) | B must directly follow A; C follows B |
| **Sequence (skip-any)** | `A → B` (`followedBy`) | B after A, with unrelated events allowed in between |
| **Conjunction** | `A AND B` | Both A and B within the window, any order |
| **Disjunction** | `A OR B` | Either A or B |
| **Negation / absence** | `NOT A within T` | A does *not* occur within the window (`notNext`, `notFollowedBy`) |
| **Repetition** | `A*` / `A{2,4}` (`times`, `oneOrMore`) | A occurs 2+ times (optionally a range) |
| **Conditional** | `A → B where B.amount > 100` | Constraints on event attributes at each step |
| **Temporal** | `within(5 min)`, `timer:interval` | Time bounds on the whole match or between steps |
| **Causality** | `A caused B` | A precedes B and is related by key — modeled as sequence + correlation |

Patterns compose: `every (A → B) within 10 min where A.card = B.card and B.amount > 3 × A.amount` is a single declarative statement of a money-mule scenario.

### 3.3 Temporal Constraints and Windows

Time is the dimension that makes CEP "complex." The standard constraint forms:

- **Within** — the whole match must complete inside a window: "login → transfer within 24h."
- **After / before** — relative to a reference event: "withdrawal *at least 30 minutes after* deposit."
- **Between** — inside an interval between two events.
- **Timer** — Esper's `timer:within(n)`, `timer:interval(n)`, `timer:at(...)`; Siddhi's `within` clause.

**Windows** group events for aggregation and matching:

- **Time-based, tumbling** — fixed non-overlapping intervals (every 5 minutes).
- **Time-based, sliding** — overlapping intervals, updated continuously (last 5 minutes, refreshed per event).
- **Time-based, hopping** — sliding with a step (every 1 minute, window of 5 minutes).
- **Count-based** — last N events regardless of time.
- **Session** — events grouped by gaps of inactivity (a session ends after 30 idle minutes).

In pattern context, windows bound how long a partial match is allowed to wait for its remaining events — the mechanism behind "A within 5 min of B."

### 3.4 Correlation Keys

A pattern match is only meaningful when its events belong to the same *entity of interest*. Correlation keys define that: partition by `account_id`, `customer_id`, `card_token`, `device_id`, `ip_address`, `trader_id`, or combinations ("same card AND same merchant group"). In Flink, patterns run per key on keyed streams; in Esper, the `where a.account = b.account` clause ties events; in SQL MATCH_RECOGNIZE, `PARTITION BY` is the correlation key.

Getting correlation keys right is a domain decision, not a technical one: money-mule detection correlates by account; synthetic-identity detection correlates by phone or IP *across* accounts; spoofing detection correlates by trader and instrument. A pattern with the wrong key is either blind (never fires) or noisy (fires on coincidental matches).

### 3.5 Match Semantics and Event Selection

When a stream can produce overlapping or ambiguous matches, the engine needs rules to decide which events participate:

- **Greedy vs. reluctant vs. lazy** — repetition consumes as many qualifying events as possible (greedy) or as few (reluctant/lazy). A greedy `A{2,}` in `A{2,} B` will try to consume B's event into A; the engine backtracks or fails accordingly.
- **Event selection (Flink)** — `first`, `last`, `next`, `next-till` control which events are retained when several match the same pattern step. `first` keeps the earliest matching event, `last` the most recent, `next` the first event *after* the previous match, `next-till` the first event after the previous match that still allows the rest of the pattern.
- **Optional steps** — `A.optional()` lets a match succeed without A; combined with greedy/reluctant, it defines whether the engine prefers matches with or without the optional event.
- **AFTER MATCH SKIP (SQL)** — `SKIP PAST LAST ROW`, `SKIP TO NEXT ROW`, `SKIP TO FIRST/LAST` determine whether events consumed by one match can seed another. This is the single most common source of subtly wrong MATCH_RECOGNIZE results and needs explicit testing.

Concretely, in the Flink Pattern API, optionality and greediness shape matches:

```java
Pattern<Event, ?> p = Pattern.<Event>begin("a")
    .where(e -> e.type.equals("A"))
    .times(2, 4).greedy()        // consume as many "a" as possible
    .optional()                  // match may omit this step
    .followedBy("b").where(e -> e.type.equals("B"))
    .within(Time.minutes(10));
```

`greedy()` on a repetition makes the engine hold onto as many events as possible; `optional()` allows a match to succeed without the step. The combination defines whether overlapping windows produce one long match or several short ones — which directly changes alert counts and is a common source of "why did this fire twice" questions from investigators.

### 3.6 Time Semantics: Event Time vs Processing Time

CEP correctness hinges on which clock the engine trusts:

- **Processing time** — the engine's wall clock when the event arrived. Simple, low latency, but wrong under backlog or out-of-order arrival: events processed late are matched in the wrong order.
- **Event time** — the timestamp embedded in the event. Correct regardless of arrival order, but requires handling *late events*: events whose event time is older than already-emitted results.
- **Watermarks** — the engine's estimate of "we have now seen everything up to time T." Events older than the watermark are late; the engine can drop them, buffer them, or emit them to a side output for reconciliation. Watermark lag is the knob that trades latency against completeness.

Banking regulators care about this: a fraud pattern matched in processing-time order can accuse the wrong transaction, and a surveillance pattern that misses a late event can miss a real violation. Production banking CEP runs event-time semantics with explicit watermark and late-data policies.

### 3.7 State in Pattern Matching

Pattern matching is inherently stateful: the engine must remember partial matches — "we saw A for account X at 14:02, still waiting for B" — for every key, until the pattern completes, times out, or the state expires. Key state-management concerns:

- **Per-key state** — one partial-match set per correlation key; the engine holds all in-progress sequences.
- **State TTL / cleanup** — patterns with long windows (24h account-takeover, 30-day journey abandonment) hold state for hours or days; without TTL and idle-state retention, state grows without bound. Flink exposes `StateTtlConfig`; Esper expires events via `win:time`; Siddhi via `#window.time` and expiry policies.
- **Window timeouts** — a partial match that never completes must be discarded (and optionally reported: "login happened but no transfer followed — monitoring value in itself").
- **State backend** — in-memory for speed, RocksDB for scale; the trade-off is latency vs. memory footprint and recovery time.

### 3.8 Thresholds and Aggregates in Patterns

Patterns frequently need arithmetic over the events they match: "sum of deposits ≥ $90,000," "3+ transactions," "average interval between events < 60s." CEP engines support this in the pattern itself:

- Flink CEP: iterate conditions and `IterativeCondition` can inspect all previously matched events; aggregations are computed in the process function after a match.
- Esper EPL: patterns combine with `select sum(amount) from pattern [...]` and window aggregations (`win:length`, `win:time`) with `having sum(amount) > 90000`.
- SQL MATCH_RECOGNIZE: `MEASURES SUM(B.amount) AS total` over the matched rows.

Thresholds are the primary precision/recall lever (Section 4.2) and should be calibrated against real data, not set by intuition.

### 3.9 Notifications vs Actions

CEP output falls into three categories:

- **Alerts** — notifications to humans or downstream systems: "potential account takeover for customer C." Typically written to an alert topic/case-management queue, not acted on automatically.
- **Derived events** — new events emitted into the stream (enriched, aggregated, or situation-level) that feed higher-level patterns: a velocity breach event becomes input to an account-takeover pattern. This is the event-hierarchy idea from Section 1.1.
- **Triggers / actions** — direct effects: decline a transaction, block a card, halt an order, call a workflow API. Actions carry operational risk — a false trigger is a declined legitimate customer — so production systems route most detections to alerts and reserve triggers for high-confidence patterns.

---

## 4. The CEP Lifecycle

### 4.1 The End-to-End Pipeline

A CEP system in a bank is a loop, not a point solution:

```
Event sources ──> Ingestion ──> Pattern detection ──> Alerts/actions ──> Case management
 (Kafka: txns,      (stream        (Flink CEP /          (alert topic,         (investigators
  logins, market     processing,    Esper / Siddhi)       block/decline,        review, label,
  data, trades)      validation)                          dashboards)           feedback)
                                                                   │
                                                                   └── monitor: alert rates, latency, FP rate ──> tune patterns
```

1. **Event ingestion** — Kafka (or equivalent) as the backbone; events from core banking, cards, channels, market data, and order management systems. Schema governance (Avro/Protobuf, registry) is a prerequisite.
2. **Pattern detection** — the CEP engine consumes the relevant topics, correlates, matches, and emits situation events.
3. **Alerts and actions** — matched situations become alerts (to a case-management system or investigator queue) or actions (declines, blocks) depending on confidence and risk appetite.
4. **Case management** — investigators review alerts, gather evidence, and decide outcomes; their dispositions are the ground truth for tuning.
5. **Feedback loop** — alert dispositions (confirmed fraud / false positive) flow back to calibrate thresholds, add patterns, and retrain scoring models.

### 4.2 Detection Quality: Precision and Recall

CEP detection quality is measured like any detection system:

- **True positives** — confirmed situations the pattern caught.
- **False positives** — alerts that were not real situations; each one costs investigator time and, if it triggers an action, customer friction.
- **Precision** — TP / (TP + FP): of all alerts, how many were real. Low precision = alert fatigue.
- **Recall** — TP / (TP + FN): of all real situations, how many were caught. Low recall = missed fraud.

**Alert fatigue** is the signature failure of banking CEP: a bank running thousands of pattern rules fires so many alerts that investigators triage superficially or ignore classes of alerts, and genuine cases drown. The trade-off is structural — tightening thresholds raises precision and lowers recall, and vice versa. Mature operations manage this explicitly: precision/recall targets per pattern family, alert volume budgets, and regular review of low-yield patterns. This is where CEP meets ML (Section 7.4): pattern recall can be wide and cheap; an ML scorer filters the candidates to keep precision high.

**Worked example.** A velocity pattern with threshold 5 transactions/5 min fires 350 alerts on a day with 100 real fraud bursts and 40,000 legitimate cards: 90 real (10 missed), 260 false — precision ≈ 26%, and investigators drown. Raising the threshold to 8 cuts alerts to 120: 75 real (25 missed), 45 false — precision ≈ 62%, recall down from 90% to 75%. The right operating point depends on risk appetite, investigator capacity, and whether an ML scorer can separate the 45 false positives from the 75 real ones downstream. This is the calibration arithmetic behind every threshold decision.
## 5. CEP Engines and Platforms

### 5.1 Open-Source Engines

#### Apache Flink CEP

Apache Flink is the leading open-source stream processor, and its `flink-cep` library is the most widely used open-source CEP implementation in production. Patterns are declared with the Pattern API on keyed `DataStream`s:

```java
DataStream<Transaction> txns = ...;  // from Kafka

Pattern<Transaction, ?> aTO = Pattern.<Transaction>begin("login")
    .where(e -> e.type.equals("LOGIN") && e.newDevice)
    .next("pwd")
    .where(e -> e.type.equals("PASSWORD_CHANGE"))
    .followedBy("transfer")
    .where(e -> e.type.equals("TRANSFER") && e.amount > 10000)
    .within(Time.hours(24));

PatternStream<Transaction> patternStream = CEP.pattern(
    txns.keyBy(t -> t.accountId), aTO);
```

Key API elements: `begin()` / `next()` (strict contiguity) / `followedBy()` (non-strict) / `notNext()` / `notFollowedBy()` (negation), `.where()` and `.or()` conditions, `.times(2)`, `.times(2,4)`, `.oneOrMore()`, `.optional()`, `.greedy()`, `.until()`, and `.within(Time...)` window timeouts. Matches arrive in `PatternStream` and are processed via `select`/`process` (which receives the full `Map<String, List<Event>>` of matched events per key — the basis for match-level aggregates like `sum(amount)`).

Because it runs on the Flink runtime, CEP inherits exactly-once checkpointing, event-time/watermark support, RocksDB state backends, savepoints for upgrades, and horizontal scaling by key partitioning. For a deeper treatment of the surrounding runtime, see [event_stream_processing_guide.md](event_stream_processing_guide.md).

#### Apache Flink SQL MATCH_RECOGNIZE

Flink SQL implements `MATCH_RECOGNIZE`, the SQL:2016 standard row-pattern recognition clause — a declarative, SQL-standard alternative to the Java Pattern API:

```sql
SELECT *
FROM transactions
MATCH_RECOGNIZE (
  PARTITION BY account_id
  ORDER BY event_time
  MEASURES
    FIRST(A.event_time)   AS start_ts,
    LAST(C.event_time)    AS end_ts,
    SUM(B.amount)         AS total_between
  ONE ROW PER MATCH
  AFTER MATCH SKIP PAST LAST ROW
  PATTERN (A B+ C)
  DEFINE
    A AS A.type = 'DEPOSIT' AND A.amount < 10000,
    B AS B.type = 'DEPOSIT' AND B.amount < 10000,
    C AS C.type = 'WITHDRAWAL'
) AS m
WHERE m.total_between >= 90000;
```

Clauses: `PATTERN` (the match shape with quantifiers `*`, `+`, `?`, `{n,m}`), `DEFINE` (per-variable conditions), `MEASURES` (what to output, including aggregates over matched rows), `AFTER MATCH SKIP` (overlap handling), plus optional `WITHIN` for time bounds in some dialects. MATCH_RECOGNIZE is a subset of the SQL:2016 standard; dialect support varies (Flink, RisingWave, Materialize, Snowflake, Oracle, DuckDB, BigQuery, and Azure Stream Analytics all implement flavors). The declarative form is a major productivity win for SQL-first teams, at the cost of some Pattern-API flexibility (e.g., custom iterate conditions).

#### Esper

Esper (EsperTech, now open-sourced as Esper/EsperIO) is the classic standalone CEP engine for Java/.NET, and its **EPL (Event Processing Language)** is the most mature pattern language in the open-source world. EPL is SQL-like with a dedicated `pattern` syntax:

```sql
-- Pattern: login from new device, then large transfer within 24h
select a.accountId, b.amount
from pattern [
  every a=LoginEvent(newDevice=true) ->
    (b=TransferEvent(amount > 10000)
       where timer:within(24 hours))
]
```

Pattern operators: `every`, `and`, `or`, `->` (followed-by), `not`, `timer:within`, `timer:interval`, `timer:at`, `timer:schedule`, plus event-attribute filters and `where` correlation. Esper also does full ESP: `win:time(5 min)`, `win:length(10)`, `win:ext_timed`, sliding/hopping windows, `sum/avg/count` with `having`, and output-rate limiting. Deployment is embedded (library) or standalone server; scale is single-node to modest clusters — it is not a distributed engine, which is the usual reason teams graduate to Flink.

Esper's ESP side is equally mature — windows and aggregations are first-class EPL:

```sql
-- Rolling 5-minute average transaction amount per merchant
select merchantId, avg(amount) as avgAmt, count(*) as cnt
from TransactionEvent.win:time(5 min)
group by merchantId
having count(*) > 10
output last every 1 minute;

-- Session window: transactions grouped by 30-minute inactivity gaps
select customerId, sum(amount) as sessionTotal
from TransactionEvent.win:session(30 min)
group by customerId;
```

The same engine therefore covers both halves of the SEP→ESP→CEP spectrum (Section 1.3) with one language.

#### Siddhi

Siddhi is WSO2's open-source (Apache 2.0) CEP engine, distributed as a lightweight Java library, microservice runtime, or the WSO2 Streaming Integrator platform. Its **Siddhi Query Language** is SQL-like with pattern constructs:

```sql
-- Every login from a new device followed by a password change within 5 minutes
from every l=LoginEvent[newDevice == true] ->
     p=PasswordChangeEvent within 5 min
select l.accountId, p.deviceId
insert into AccountTakeoverStage1;
```

Pattern syntax: `every`, `sequence`, `and`, `or`, `not`, `->`, `within` (time bounds), with windows (`#window.time(5 min)`, `#window.length(10)`, session windows) and standard SQL-like queries for the ESP side. Siddhi shines where CEP must be embedded in a Java/Kafka microservice or inside a WSO2 stack (with Kafka, EI, and micro-integration). It supports incremental aggregation, partitions, and function extensions (ML, geo). Windows are inline in SiddhiQL:

```sql
-- Sliding time window with aggregation
from TransactionEvent#window.time(5 min)
select cardId, sum(amount) as total, count() as cnt
group by cardId
having cnt >= 3
insert into VelocityCheck;

-- Session window for journey tracking
from PageViewEvent#window.session(30 min)
select sessionKey(), count() as pageViews
insert into JourneySession;
```

Siddhi's embeddability (a few-MB library inside a JVM service) makes it a popular choice for per-service CEP within larger Kafka microservice architectures.

#### Drools Fusion

Drools Fusion (part of KIE) is a rule engine with a CEP mode: rules run over event streams with event semantics — event expiration, identity, and **temporal operators** (`after`, `before`, `coincides`, `during`, `includes`, `finishes`, `finishedby`, `meets`, `metby`, `overlaps`, `overlappedby`, `starts`, `startedby`). Sliding windows (`window:time`, `window:length`) appear directly in rule conditions:

```drl
rule "Transfer after credential change"
when
    $p: PasswordChangeEvent() from entry-point "CEP"
    TransferEvent(amount > 10000,
        this after $p over window:time(24h))
then
    insert(new AccountTakeoverAlert($p.accountId));
end
```

Drools Fusion suits organizations already invested in Drools business rules that need temporal reasoning on event streams — it is not stream-native (no partitioning, no watermarking, no distributed runtime), so it fits decisioning workloads more than high-throughput detection.

#### Apache Kafka Streams

Kafka Streams (KStreams) is Kafka's native stream-processing library. It has **no pattern language** — pattern detection is written by hand using the Processor API, state stores, and punctuators:

```java
// Sketch: stateful spoofing detection — order placed then cancelled
// by the same trader for the same instrument within 2 minutes.
processorContext.transitionTo(StateStore)      // store order by (trader, instrument)
punctuator.wallClockTime(60_000, ...)          // scan for stale orders
// on OrderCancelled: look up open order, compare size & time, emit alert
```

State stores (RocksDB-backed), KTables, windowing (`TimeWindows`, `SessionWindows`), and exactly-once are all available; the team builds the state machine. This is the right choice when patterns are few and simple, and the whole pipeline is Kafka-centric Java.

#### Apache Spark Structured Streaming

Spark Structured Streaming supports event-time windows, watermarking, and stateful `mapGroupsWithState`/`flatMapGroupsWithState` — again, **pattern detection is code, not a pattern language**. Teams implement sequence detection as explicit state machines in the state function. Spark's strength is batch+stream unification and scale; its weakness is per-event latency (typically hundreds of ms to seconds) and the absence of temporal pattern primitives. It fits CEP workloads that can tolerate seconds of latency and that already live in a Spark shop.

#### Streaming Databases: RisingWave and Materialize

**RisingWave** is a cloud-native streaming database exposing SQL over Kafka/Pulsar streams, and it implements `MATCH_RECOGNIZE` — pattern detection becomes a SQL view that stays incrementally updated. **Materialize** is an incremental SQL view engine with `MATCH_RECOGNIZE` support as well. Both are attractive when the team is SQL-first and wants CEP results queryable like tables, with the warehouse as the consumer. Pattern complexity and match semantics are bounded by their MATCH_RECOGNIZE implementations.

#### Hazelcast Jet, Samza, Storm

**Hazelcast Jet** is an in-memory distributed stream processor with a CEP-friendly API (windowing, stateful transforms) and low-latency in-memory architecture. **Apache Samza** provides stateful stream processing on Kafka/YARN with a low-level API. **Apache Storm** is the legacy pioneer (spouts/bolts, Trident) — pattern detection is hand-built; Storm is in maintenance mode and new builds should prefer Flink/Kafka Streams.

### 5.2 Commercial Engines

| Engine | Vendor | Positioning |
|---|---|---|
| **TIBCO Streaming (StreamBase)** | TIBCO | The classic CEP platform (StreamBase); visual StreamBase Studio, EventFlow language, low-latency, capital-markets heritage |
| **Apama** | Software AG | The other classic; EPL (Event Processing Language), streaming analytics, event-correlation; used heavily in banking/trading floors and now integrated into Cumulocity IoT |
| **SAS Event Stream Processing** | SAS | SAS ESP: rule engine + scoring + analytics on streams; used in fraud/risk with SAS Model Manager and Visual Investigator integration |
| **IBM Streams (InfoSphere Streams)** | IBM | Distributed stream processing with SPL language; strong in telecom/finance; being repositioned toward IBM Cloud Pak for Data |
| **Oracle Stream Analytics** | Oracle | Inside Oracle Event Processing (OEP), now part of Oracle Integration; SQL/EPL over Oracle-centric event sources |
| **Azure Stream Analytics** | Microsoft | Cloud SQL-like queries (`TumblingWindow`, `HoppingWindow`, `MATCH_RECOGNIZE`-like via `MATCH_RECOGNIZE` support) over Event Hubs/IoT Hub |
| **AWS Kinesis Analytics** | AWS | SQL over Kinesis streams plus Apache Flink (KDA) with Flink SQL `MATCH_RECOGNIZE` |
| **Google Cloud Dataflow** | Google | Apache Beam managed service; stateful/event-time processing, but no pattern language — patterns via DoFn state machines |
| **Confluent ksqlDB** | Confluent | SQL over Kafka with `PARTITION BY`, windowing, UDFs; pattern detection via SQL-isms (self-joins, windowed counts); native MATCH_RECOGNIZE only in recent versions |

### 5.3 Comparison Table

| Engine | Language | Pattern language | State mgmt | Deployment | Scale | Best for |
|---|---|---|---|---|---|---|
| **Flink CEP** | Java/Scala/SQL | Pattern API + MATCH_RECOGNIZE | RocksDB/memory, TTL, checkpoints | Cluster (K8s/YARN) | Millions evt/s | Complex stateful patterns at scale, exactly-once |
| **Flink SQL MATCH_RECOGNIZE** | SQL | SQL:2016 row patterns | Same as Flink | Cluster | Same | SQL-first teams on Flink |
| **Esper** | Java/.NET | EPL (richest) | In-memory, expiration | Embedded/server | 10⁵–10⁶ evt/s | Standalone CEP, mature EPL, Java shop |
| **Siddhi** | Java/SQL-like | SiddhiQL patterns | In-memory, windows, partitions | Embedded/microservice | 10⁵–10⁶ evt/s | Lightweight embedded CEP, WSO2 stack |
| **Drools Fusion** | DRL | Rules + temporal ops | Working memory, expiration | Embedded (KIE server) | 10⁴–10⁵ evt/s | Rules + events hybrid decisioning |
| **Kafka Streams** | Java | None (Processor API) | RocksDB stores | Library on Kafka | Millions evt/s | Kafka-centric pipelines, hand-coded patterns |
| **Spark SS** | Scala/Python/SQL | None (stateful APIs) | State stores, checkpoints | Cluster | Very high | Batch+stream unified, seconds latency |
| **RisingWave / Materialize** | SQL | MATCH_RECOGNIZE | Materialized state | Cloud/managed | High | SQL-first, queryable CEP results |
| **Apama / StreamBase** | EPL/EventFlow | Rich proprietary EPL | Commercial | Appliance/VM/cloud | High | Trading floors, enterprise support |
| **SAS ESP** | SAS lang/rules | Rules + scoring | Commercial | On-prem/cloud | High | SAS ecosystem, fraud/risk scoring |
| **Azure Stream Analytics** | SQL | SQL + windows (+MATCH_RECOGNIZE) | Managed | Azure | High | Cloud-native, SQL team, simpler patterns |
| **Kinesis Analytics** | SQL/Flink | SQL + Flink MATCH_RECOGNIZE | Managed | AWS | High | AWS-native streaming analytics |
| **ksqlDB** | SQL | SQL-isms, no native patterns | Managed/self-hosted | Kafka cluster | High | Kafka-centric, SQL-first, simpler patterns |

Rule of thumb: **Flink for scale and complex state; Esper/Siddhi for standalone pattern-language power; SQL engines (Flink SQL, RisingWave, ksqlDB, cloud) when the team is SQL-first; commercial engines when enterprise support and a regulated-vendor footprint matter.**
## 6. CEP Pattern Catalog for Banking

The catalog below is the banking canon of CEP patterns — each entry names the typology, the correlation key, the window, and the pattern shape. Section 6.8 gives worked code for six of them.

### 6.1 Fraud Detection

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Card-not-present fraud** | Multiple transactions on one card at *different* merchants/geographies in rapid succession | Card token | Minutes |
| **Account takeover (ATO)** | Login from new device → password change → large transfer | Account ID / device | 24h |
| **Money mule** | Deposit into low-activity account → rapid withdrawal/transfer onward | Account ID | Hours |
| **Bust-out fraud** | Credit limit increase → max utilization → no payment (build over weeks, then burn) | Account ID | Weeks–months |
| **Synthetic identity** | Multiple accounts sharing phone/IP/address with different names, small qualifying activity, then a spike | Phone / IP / device (cross-account) | Months |
| **Velocity breach** | N transactions on one card/customer in M minutes | Card / customer | Minutes |

### 6.2 AML / Financial Crime

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Structuring / smurfing** | Many deposits each below the reporting threshold, summing above it | Account ID | Hours–days |
| **Layering** | Rapid movement of funds between many accounts to obscure origin | Account graph | Days |
| **Placement** | Large cash deposit followed immediately by transfer to a different entity | Account ID | Minutes–hours |
| **Rapid in-out** | Funds arrive and leave within a short window, netting to ~zero | Account ID | Hours |
| **Unusual counterparty** | Transactions to/from sanctioned, PEP, or high-risk jurisdictions | Account / counterparty | Real-time + list join |

AML typologies map naturally onto CEP sequences; the financial_fraud_detection_at_scale_guide.md covers the full AML detection stack including Flink CEP pattern examples for these typologies, plus the case-management and regulatory side.

### 6.3 Trade Surveillance

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Wash trading** | Same counterparty buys and sells the same instrument, no change of beneficial ownership | Counterparty + instrument | Day |
| **Spoofing** | Large order placed to move the market, then cancelled before execution | Trader + instrument | Minutes |
| **Layering (market)** | Sequential orders at improving prices creating false liquidity, then cancelled | Trader + instrument | Minutes |
| **Front-running** | Large customer order followed by own-account trade in the same instrument | Trader + instrument | Seconds–minutes |
| **Insider trading** | Trade in instrument shortly before material news about the issuer (correlating trade stream with news events) | Trader + instrument (+ news feed) | Hours |

### 6.4 Market Risk

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Flash crash** | Rapid price decline (e.g., >5% in seconds) with no news | Instrument | Seconds |
| **Circuit breaker** | Price move exceeding threshold triggers trading halt logic | Instrument | Seconds–minutes |
| **Correlation breakdown** | Two historically correlated instruments diverge beyond a band | Instrument pair | Minutes–hours |
| **Liquidity dry-up** | Spread widening + volume drop beyond thresholds simultaneously | Instrument / venue | Minutes |

### 6.5 Payments

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Velocity check** | N payments in M minutes (cards, wallets, IBANs) | Payer / card / device | Minutes |
| **Unusual merchant category** | First-ever transaction in a category, or category switch pattern | Card + MCC | Real-time |
| **Cross-border burst** | Sudden cluster of cross-border payments from a normally domestic account | Account | Hours |
| **Round-trip payments** | Send → receive → send between the same parties (value movement without economic purpose) | Account pair | Days |

### 6.6 Operational Monitoring

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Failed-login sequence** | N failed logins → successful login (credential stuffing) | User / IP / device | Minutes |
| **API abuse** | Rate-limit evasion: bursts just under limits, token rotation | API key / IP | Minutes |
| **Failure cascade** | Error spike → retry storm → service outage (leading-indicator chain) | Service / host | Minutes |
| **SLA breach** | P95 latency above threshold for a sustained window | Service | Windowed |

### 6.7 Customer Journeys

| Pattern | Description | Correlation | Window |
|---|---|---|---|
| **Journey abandonment** | Application started → no completion within 30 days | Customer / session | 30 days |
| **Churn signal** | Decreased activity + support complaints within a window | Customer | Weeks |
| **Cross-sell trigger** | Milestone event (salary credit, mortgage anniversary) + product eligibility | Customer | Real-time |

### 6.8 Worked Pattern Examples

**Pattern 1 — Card-not-present velocity (Flink CEP).** N transactions, same card, distinct merchants, within 5 minutes:

```java
Pattern<Transaction, ?> cnP = Pattern.<Transaction>begin("first")
    .where(t -> t.type == CARD_PURCHASE)
    .times(3)
    .where((t, ctx) -> ctx.getEventsForPattern("first")
        .stream().map(Transaction::getMerchantId).distinct().count() >= 3)
    .within(Time.minutes(5));

CEP.pattern(txns.keyBy(Transaction::getCardToken), cnP)
   .process(new MatchAlertFunction("CNP_VELOCITY", 3, 5));
```

**Pattern 2 — Account takeover (Esper EPL).** New-device login, then password change, then transfer > $10k, all within 24h:

```sql
select a.accountId as acct, c.amount as amount
from pattern [
  every a=LoginEvent(newDevice=true) ->
    b=PasswordChangeEvent(accountId=a.accountId) ->
    c=TransferEvent(accountId=a.accountId, amount > 10000)
      where timer:within(24 hours)
]
```

**Pattern 3 — Structuring (Flink SQL MATCH_RECOGNIZE).** 3+ deposits under $10,000 within 1 hour summing over $25,000:

```sql
SELECT account_id, cnt, total
FROM (
  SELECT *
  FROM deposits
  MATCH_RECOGNIZE (
    PARTITION BY account_id
    ORDER BY ts
    MEASURES
      COUNT(A.amount) AS cnt,
      SUM(A.amount)   AS total
    ONE ROW PER MATCH
    AFTER MATCH SKIP PAST LAST ROW
    PATTERN (A{3,})
    WITHIN INTERVAL '1' HOUR
    DEFINE A AS A.amount < 10000
  )
)
WHERE total > 25000;
```

**Pattern 4 — Money mule (Siddhi).** Deposit into a low-activity account followed by a withdrawal of ≥80% of the deposit within 6 hours:

```sql
from every d=DepositEvent[accountAgeDays > 180 and priorTxns30d < 3] ->
     w=WithdrawalEvent[accountId == d.accountId and amount >= 0.8 * d.amount] within 6 hours
select d.accountId, d.amount as deposit, w.amount as withdrawal
insert into MoneyMuleAlert;
```

**Pattern 5 — Wash trading (Flink CEP).** Buy and sell of the same instrument by the same counterparty within the same day, crossing near the same price:

```java
Pattern<Order, ?> wash = Pattern.<Order>begin("buy")
    .where(o -> o.side == BUY && o.counterparty != null)
    .followedBy("sell")
    .where((o, ctx) -> o.side == SELL
        && o.instrument.equals(ctx.getEventsForPattern("buy").get(0).instrument)
        && o.counterparty.equals(ctx.getEventsForPattern("buy").get(0).counterparty)
        && Math.abs(o.price - ctx.getEventsForPattern("buy").get(0).price) < 0.01 * o.price)
    .within(Time.hours(24));
```

**Pattern 6 — Spoofing (pseudocode / Drools Fusion).** Large order placed, then cancelled before execution; repeat pattern is the signal:

```drl
rule "Spoofing: large order placed then cancelled"
when
    $o: OrderEvent(side == BUY, quantity >= 50000) from entry-point "Orders"
    OrderCancelEvent(orderId == $o.orderId,
        this after $o over window:time(2m),
        $o.executedQuantity == 0)
    not(OrderEvent(side == SELL, quantity >= 50000,
        this after $o over window:time(2m)))
then
    insert(new SpoofingAlert($o.traderId, $o.instrument));
end
```

**Pattern 7 — Round-trip payments (Flink SQL MATCH_RECOGNIZE).** A→B→A transfers where the return roughly equals the outbound:

```sql
SELECT payer_id, payee_id, outbound, return_amount
FROM payments
MATCH_RECOGNIZE (
  PARTITION BY payer_id, payee_id
  ORDER BY ts
  MEASURES
    A.amount AS outbound,
    C.amount AS return_amount
  ONE ROW PER MATCH
  AFTER MATCH SKIP PAST LAST ROW
  PATTERN (A B* C)
  WITHIN INTERVAL '7' DAY
  DEFINE
    A AS A.type = 'OUT',
    B AS B.type = 'OUT',
    C AS C.type = 'IN' AND C.amount >= 0.9 * A.amount AND C.amount <= 1.1 * A.amount
);
```

**Pattern 8 — Flash crash (aggregation + pattern).** Price drop >5% within 60 seconds on an instrument:

```sql
-- Esper: windowed aggregation driving a pattern condition
select symbol, min(price) as low, max(price) as high
from PriceTick.win:time(60 sec)
group by symbol
having (max(price) - min(price)) / max(price) > 0.05
output first every 60 seconds;
```

**Pattern 9 — Layering (Esper EPL).** Funds hopping through three accounts within 24 hours, each hop within the window:

```sql
select a.fromAccount as origin, c.toAccount as terminus, a.amount as amount
from pattern [
  every a=TransferEvent ->
    b=TransferEvent(fromAccount=a.toAccount) ->
    c=TransferEvent(fromAccount=b.toAccount)
      where timer:within(24 hours)
]
```

**Pattern 10 — Insider trading (Flink CEP, trade correlated with material news).** A trade in an instrument shortly before material news about its issuer — the news feed is a second Kafka topic joined into the same stream:

```java
Pattern<Event, ?> insider = Pattern.<Event>begin("trade")
    .where(e -> "TRADE".equals(e.type))
    .followedBy("news")
    .where((e, ctx) -> "NEWS".equals(e.type)
        && e.material                     // materiality flag from news NLP pipeline
        && e.issuer.equals(ctx.getEventsForPattern("trade").get(0).issuer))
    .within(Time.hours(2));
```

The suspicious ordering is trade *before* news: the pattern fires when an own-account or related-party trade precedes the public disclosure, and the alert trail (trade event + news event + timestamps) is exactly the evidence regulators ask for.

## 7. Implementing CEP in Banking

### 7.1 Reference Architecture

A production banking CEP platform has five tiers:

```
┌─ Sources ─────────────────────────────────────────────────────────────┐
│  Core banking (transactions, accounts)   Cards (authorizations)      │
│  Channels (login, onboarding)            Market data (ticks, quotes) │
│  OMS/EMS (orders, fills)                 Payments (SWIFT, FAST, SEPA)│
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌─ Ingestion ── Kafka event backbone ───────────────────────────────────┐
│  Schema registry (Avro/Protobuf) · validation · enrichment (geo,     │
│  device fingerprint, watchlists) · partitioning by correlation key    │
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌─ Pattern detection ── Flink CEP / Esper / Siddhi ─────────────────────┐
│  Pattern tiers: transaction → customer → network/graph                │
│  Event-time + watermarks · per-key state with TTL · ML scoring hook   │
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌─ Alerts & actions ────────────────────────────────────────────────────┐
│  Alert topic (Kafka) → case management (Actimize, Allegro, in-house) │
│  Action path: block card, decline auth, hold transfer, suspend order  │
│  Dashboards (alert volume, latency, FP rate)                          │
└───────────────┬──────────────────────────────────────────────────────┘
                ▼
┌─ Feedback loop ── investigator dispositions ──> threshold tuning ──┐
└────────────────────────────────────────────────────────────────────┘
```

Key design points:

- **Kafka as the backbone** with topics partitioned by correlation key (account, card, customer) so each partition's events are naturally ordered per key — the CEP engine consumes one partition per key range.
- **Pattern tiers** mirror the event hierarchy: tier 1 patterns emit derived events (velocity breaches) that tier 2 patterns (account takeover) consume. This keeps individual patterns simple and auditable.
- **Decision split:** detection (CEP) and action (a decision service — often a rules engine or ML scorer, per Section 2.2) are separate components so the action logic can change without touching patterns.
- **Monitoring** of the pipeline itself: alert rates per pattern, detection latency (event time → alert), false-positive rate per pattern, engine lag (watermark vs. wall clock), state size, backpressure — Section 9.1.
- **Schema governance** — events enter through a registry with versioned Avro/Protobuf schemas; the CEP engine binds to schema versions so a producer change cannot silently break patterns. Unknown or ill-formed events go to a dead-letter topic, never into the pattern engine.

### 7.2 Pattern Design Process

Turning a fraud/risk typology into a working pattern is a repeatable process:

1. **Understand the typology with domain experts.** Fraud investigators, AML officers, and surveillance analysts describe the *modus operandi*: what events occur, in what order, over what time, involving which entities. Do not skip this — patterns built from documentation alone miss the real-world variations.
2. **Translate the typology into an event pattern.** Choose the operators (sequence vs. correlation vs. absence), the correlation keys (account, card, customer, device, IP — and when to correlate *across* keys, e.g., synthetic identity by phone/IP), and the window.
3. **Define windows and thresholds, calibrated with data.** Analyze historical labeled data: how fast do real cases unfold? What do the amounts look like? Set thresholds from percentiles, not intuition; a threshold at the 99.9th percentile of legitimate behavior is a reasonable starting point, then tune.
4. **Handle event time vs. processing time.** Adopt event-time semantics; set watermark lag and late-data policy explicitly (Section 3.6).
5. **Design for state cleanup.** Set TTLs aligned to the longest window; cap per-key state; plan for idle keys (a pattern on `within 24h` still holds state for every key that saw step A). Flink `StateTtlConfig`, Esper expiration policies, and window caps are the tools.
6. **Test with historical data (replay).** Replay labeled historical streams through the pattern; measure precision/recall against ground truth before going live. Golden datasets (Section 9.3) make this repeatable.
7. **Tune precision/recall.** Adjust thresholds, add suppression conditions, layer an ML scorer on top (Section 7.4).
8. **Monitor and iterate.** Track each pattern's yield; kill patterns that never confirm; revise patterns as fraudsters adapt. The feedback loop from investigators is the system's immune system.

**Done-when checklist** for shipping a pattern:

| Step | Done when |
|---|---|
| Typology understood | Domain experts sign off on the modus-operandi description |
| Pattern translated | Pattern code reviewed; operators and correlation keys documented |
| Thresholds calibrated | Precision/recall measured on labeled historical data |
| Time semantics set | Watermark + late-data policy defined and tested |
| State bounded | TTLs and caps set; state-size test passes |
| Replay validated | Golden dataset and historical replay both pass |
| Tuned | Thresholds / score cutoff chosen against risk appetite |
| Monitored | Yield dashboard live; owner named for the feedback loop |

### 7.3 Performance and Scale Considerations

| Concern | Reality | Mitigation |
|---|---|---|
| **Throughput** | Banks run millions of events/sec across all streams | Partition by key; scale engine parallelism with partitions; only the patterns that need sub-second latency run in the hot path |
| **Latency** | Sub-second to seconds typical; trading surveillance needs ms | In-memory state, minimal enrichment in the pattern path, process-time mode only where event order is guaranteed |
| **State size** | Per-key partial matches × key count × window duration | TTLs, window caps, idle-state retention, RocksDB backend, state migration on upgrade |
| **Horizontal scaling** | Flink scales by parallelism; each key's pattern runs on one task | Align Kafka partitioning to parallelism; rebalance on hot keys |
| **Exactly-once vs at-least-once** | At-least-once + reprocessing = duplicate alerts | Alert dedup keys (pattern + correlation key + match start time); idempotent actions |
| **Backpressure** | Sink slower than detection → state backlog, watermark stall | Backpressure-aware design, bounded alert sinks, load shedding for non-critical patterns |
| **Checkpointing/recovery** | Failover must not lose in-flight matches | Flink checkpoints/savepoints; Esper requires external state persistence; test recovery with state restore |
| **Hot keys** | One account generating 10k events/sec skews one task | Split hot keys (suffix sharding), per-key rate caps, secondary correlation dimensions |
| **Event-time skew** | Producers with lagging clocks or delayed batch loads | Watermark strategy per source, timestamp extraction validation, skew monitoring |

**Load shedding** matters: when the engine falls behind, dropping *non-critical* pattern evaluations (while never dropping the stream itself) is preferable to letting watermark lag grow until every window fires late.

### 7.4 Integration with ML

CEP and ML are complementary, and production banking detection is a hybrid of both:

- **ML for scoring (the standard fraud pattern).** CEP detects candidate situations with wide, explainable patterns; an ML model (gradient boosting or neural network over features) scores each candidate for fraud probability; a threshold on the score decides the action. CEP provides recall coverage and explainability; ML provides precision and adaptivity. This is the dominant architecture in card fraud.
- **Feature extraction via CEP.** CEP aggregations — velocity counts, sum/avg amounts, pattern-match flags, interval statistics — feed the feature vector of ML models in real time (via a feature store or streaming feature pipeline). Many ML fraud models are largely *pattern features* underneath.
- **Pattern discovery via ML.** Unsupervised anomaly detection (isolation forests, autoencoders) on streaming features surfaces novel behaviors; analysts investigate and encode the confirmed ones as new CEP patterns — ML proposes, CEP codifies.
- **Model-in-loop.** ML model outputs are themselves events: score events, drift signals, and model-retraining triggers flow back into the CEP stream so patterns can consume them ("if score of entity > 0.9 and velocity pattern fires, escalate").

### 7.5 Case Study: Real-Time Fraud Detection Pipeline

A regional bank's card-fraud platform, in production shape:

- **Sources:** authorization stream (~40k TPS peak), channel login events, device-fingerprint events — all into Kafka, partitioned by card token and customer ID.
- **Detection:** Flink CEP with ~60 production patterns: CNP velocity, ATO sequences (login → change → transfer), mule sequences, bust-out trend patterns (weekly windowed utilization), synthetic-identity cross-account patterns keyed by phone/IP. Watermark lag 10s; late events routed to a reconciliation topic.
- **Scoring:** every matched situation gets ~120 features (CEP-computed velocity + ML-encoded entity features); a gradient-boosted model scores it; score > 0.85 → decline action, 0.5–0.85 → case review, below → log only.
- **Actions:** declines via the authorization API (idempotent, exactly-once keyed by auth ID); review cases to the case-management system with the full matched-event trail attached.
- **Feedback:** investigators label dispositions; labels feed (a) weekly threshold re-calibration and (b) model retraining; pattern yield reports kill the bottom decile of patterns each quarter.
- **Outcome metrics:** sub-100ms decision latency, ~92% precision on declined authorizations, ~85% recall on confirmed fraud, and investigator throughput roughly 4× vs. the pre-CEP rule engine. The explainable CEP trail (which pattern fired, which events matched) is the single biggest driver of investigator speed and of regulatory acceptance.
## 8. CEP vs Alternative Detection Approaches

CEP is one tool in the detection toolbox. Knowing when *not* to use it is as valuable as knowing when to.

### 8.1 Streaming SQL Aggregations

Many detection needs are simple velocity/aggregation checks, and a plain windowed aggregation in Flink SQL, ksqlDB, or a cloud stream service is often the right answer: "N payments in M minutes," "sum of deposits over threshold," "P95 latency above X." These need no pattern engine — they are ESP (Section 2.1). CEP earns its complexity when the situation is a **sequence with ordering and cross-event constraints**: not just "3 deposits" but "deposit, then withdrawal of ≥80% within 6 hours, in an account with low prior activity." Start with aggregations; escalate to CEP only for patterns aggregations cannot express.

### 8.2 ML-Only Detection

Anomaly detection models (isolation forests, autoencoders, deep sequence models) find *unusual* behavior without encoding what "fraudulent" means. Strengths: catches unknown patterns, adapts to drift, handles high-dimensional features. Weaknesses: opaque alerts (regulators and investigators demand explanations), no built-in temporal-sequence semantics, threshold instability. CEP and ML are complements, not substitutes — see Section 7.4.

### 8.3 Rule Engines and Heuristics

Rule engines (Drools and commercial BRMS) express rich IF-THEN decision logic but, outside Fusion-style extensions, lack stream-native ordering and temporal semantics. Lookalike/heuristic approaches (scoring "transactions like known-bad ones" by similarity) are cheap but brittle and hard to govern. CEP patterns sit between them: more structured than heuristics, more temporal than classic rules.

### 8.4 When CEP Is the Right Tool — and When ML Is Better

| Situation | Prefer |
|---|---|
| Known, explicit patterns; domain experts can describe the typology | **CEP** |
| Temporal sequences with ordering and windows are the core of the detection | **CEP** |
| Low-latency decisions (sub-second) with explainable alerts | **CEP** |
| Regulatory need to explain every alert in event terms | **CEP** |
| Unknown or evolving patterns; high-dimensional features | **ML** |
| Adaptive scoring to filter CEP candidates | **ML** |
| Both: known patterns + scoring | **Hybrid CEP + ML** |

### 8.5 Hybrid CEP + ML as the Standard

For banking fraud and AML, the hybrid is the de facto standard: CEP gives coverage of known typologies, real-time low-latency detection, and inherently explainable alerts; ML gives precision, adaptivity, and detection of pattern variations. The pipeline shape is fixed: wide CEP patterns → candidate situations → ML score → thresholded action → investigator feedback → retune both sides.

---

## 9. Operational Considerations

### 9.1 Monitoring

Run the detection platform like the risk system it is:

- **Alert volume per pattern** — absolute count and rate; a spike means either fraud activity or a broken pattern.
- **True positive / false positive rates** — per pattern, per family, from investigator dispositions; these are the health metrics.
- **Detection latency** — event time → alert time; watermark lag (engine clock vs. event time) is the leading indicator of trouble.
- **Engine lag and backpressure** — consumer lag per topic, task backpressure metrics, checkpoint duration.
- **State size** — per-key state totals, TTL effectiveness, RocksDB usage, state growth rate.

Dashboards plus alerting on these (with SLOs: e.g., p95 detection latency < 2s, engine lag < 30s, FP rate per pattern < target) are the operational contract.

### 9.2 Alert Management

- **Case management integration** — alerts flow into Actimize Case Manager, Allegro, or an in-house case system, carrying the matched-event trail (which pattern, which events, which timestamps). The trail is what makes CEP alerts actionable and auditable.
- **Alert triage** — tiered queues (high-confidence automatic-action, medium review, low informational); ML pre-screening can rank alerts within a queue.
- **False positive reduction** — threshold tuning per pattern, ML scoring of candidates, suppression rules (e.g., whitelisted merchants, repeated same-alert suppression), alert correlation and deduplication across patterns (one situation firing five patterns should produce one case, not five).
- **Investigator feedback loop** — dispositions (confirmed / false positive / undetermined) label the alert stream; labels drive threshold recalibration, pattern revision, and model retraining. Without this loop, patterns decay as behavior changes.

### 9.3 Testing

- **Replay historical data** — run labeled historical streams through new patterns before deployment; compare against ground truth.
- **Golden datasets** — a curated set of scenarios (one per pattern, plus edge cases) with expected alert outputs; run in CI on every pattern change.
- **Pattern unit tests** — feed synthetic event sequences covering match, non-match, boundary (exactly at threshold, exactly at window edge), overlapping matches, and negations.
- **Chaos testing** — inject late events, duplicates, gaps, and clock skew; verify the watermark/late-data policy behaves as designed and state doesn't corrupt.
- **Performance testing** — throughput and latency at 2–3× expected peak, with state growth; verify backpressure behavior and recovery time after forced failover.

### 9.4 Deployment

On-prem vs. cloud is a bank-specific decision (data residency, MAS TRM requirements). The mainstream target is **Kafka + Flink cluster on Kubernetes**: Flink as a Kubernetes-native operator, Kafka (or managed Kafka) as the backbone, patterns as versioned Flink jobs deployed via CI/CD with savepoints for zero-downtime upgrades. Autoscaling applies to the ingestion and scoring tiers; pattern jobs scale by parallelism and partition count. Commercial engines (Apama, StreamBase, SAS ESP) deploy as appliances/VM clusters with their own HA story.

### 9.5 Compliance and Governance

- **Audit trail** — every alert must answer: which pattern fired, which version, which events matched, which timestamps, which decision was taken. Pattern runs must be reproducible from stored streams.
- **Model/rule governance** — patterns are models under regulatory scrutiny: documentation, versioning, change approval, performance tracking (SR 11-7 in the US treats model components of detection systems as models; MAS TRM and FEAT in Singapore impose equivalent discipline on detection and explanation).
- **Explainability** — CEP's key advantage over pure ML: a pattern alert is explainable by construction ("this alert fired because login from new device at 09:01, password change at 09:04, transfer of $12,000 at 09:07 — matching pattern ATO-01 within 24h"). Preserve this in alert payloads and case records.
- **Data retention** — align stream retention, case data retention, and audit logs with local regulation; ensure replay capability within retention windows.
- **Model risk** — where ML scores are layered on CEP, the ML components carry their own SR 11-7 governance; CEP patterns themselves need versioned, approved, documented change control.

**Compliance checklist:**

| Control | Evidence |
|---|---|
| Audit trail | Every alert stores pattern ID, version, matched events, timestamps, decision |
| Pattern governance | Versioned, documented, change-approved; SR 11-7 / MAS-aligned review cycle |
| Explainability | Alert payload carries a human-readable match narrative |
| Data retention | Stream / case / audit retention aligned to regulation; replay possible in-window |
| Model risk | ML components governed separately (SR 11-7); thresholds and cutoffs documented |

---

## 10. Engine Selection Guide

### 10.1 When to Choose Each Engine

- **Apache Flink CEP** — already on the Flink/Kafka ecosystem; need scale (millions of events/sec), exactly-once, event-time rigor, and complex stateful patterns; Java team. The default for enterprise banking detection platforms.
- **Esper** — want the most mature, powerful pattern language (EPL) and a standalone CEP engine; Java shop; moderate scale (single node to tens of thousands of events/sec); fast time-to-pattern.
- **Siddhi** — lightweight, SQL-like CEP; WSO2 ecosystem; microservice-friendly and embeddable; good for per-service pattern logic and IoT-ish telemetry at modest scale.
- **Drools Fusion** — already using Drools business rules and need rules + events hybrid; rule-based patterns in a decisioning context; not for high-throughput stream detection.
- **ksqlDB** — Kafka-centric, SQL-first team, simpler patterns; no native MATCH_RECOGNIZE (as of recent versions it remains limited) — plan around SQL-isms and windowed aggregations.
- **Apama / StreamBase** — enterprise support, trading-floor heritage, lowest-latency requirements, commercial budget; classic capital-markets surveillance.
- **SAS ESP** — SAS ecosystem, scoring integration (SAS Model Manager/Visual Investigator), fraud/risk analytics shops already on SAS.
- **Azure Stream Analytics / AWS Kinesis Analytics** — cloud-native, SQL team, simpler patterns, managed infrastructure; Kinesis adds Apache Flink for heavier workloads.
- **RisingWave / Materialize** — SQL-first team that wants CEP results as queryable tables; MATCH_RECOGNIZE-driven patterns feeding analytics and applications.

### 10.2 Decision Table

| Requirement | Recommendation |
|---|---|
| Sub-millisecond latency, trading floor | Apama, TIBCO Streaming (commercial, proven) |
| Millions of events/sec, exactly-once, complex state | Flink CEP |
| SQL-first team, managed cloud | Azure Stream Analytics, Kinesis Analytics, ksqlDB |
| Standalone Java CEP, richest pattern language | Esper |
| Embedded/microservice CEP, lightweight | Siddhi |
| Rules + events hybrid, existing Drools | Drools Fusion |
| Queryable CEP results, warehouse integration | RisingWave, Materialize |
| SAS analytics/risk ecosystem | SAS ESP |
| Kafka-only pipeline, few hand-built patterns | Kafka Streams |
| Batch + stream unified, seconds latency OK | Spark Structured Streaming |
| Regulated vendor footprint, enterprise support | Apama, StreamBase, SAS ESP, IBM Streams |

Tie-breakers: team skills (SQL vs. Java), existing platform (Kafka/Flink vs. WSO2 vs. cloud), pattern complexity (declarative language vs. hand-coded state), compliance (vendor support, audit features, on-prem capability for MAS TRM data residency), and budget.

---

## 11. Anti-Patterns and Pitfalls

- **Pattern overload** — thousands of rules firing constantly → alert fatigue, investigators tune out, real cases drown. Budget alert volume per family; kill low-yield patterns quarterly.
- **State explosion** — unbounded per-key state (no TTLs, window caps, or idle-state retention) → memory/RocksDB blowup and checkpoint bloat. Align TTL to the longest window; monitor state size.
- **Event time ignored** — matching on processing time → wrong order under backlog → wrong matches, wrong alerts, regulatory exposure. Use event time with explicit watermarks and late-data policy.
- **Hot-key bottlenecks** — one account/card generating a disproportionate share of events skews one partition/task. Shard hot keys; use secondary correlation dimensions.
- **Duplicate alerts** — at-least-once reprocessing after failover duplicates alerts and double-declines customers. Dedup keys and idempotent actions are mandatory.
- **No threshold calibration** — default or guessed thresholds → either alert storms or blind spots. Calibrate with labeled historical data; revisit on behavior drift.
- **Ignoring the feedback loop** — no investigator dispositions flowing back → patterns go stale as fraudsters adapt. The loop is the system's immune system.
- **Pattern engine in the hot path without load shedding** — when the engine falls behind, everything backs up. Shed non-critical patterns; protect the stream and the action path.
- **Single point of failure** — one engine node/instance down → missed detections with no failover. HA deployment, checkpointing, and recovery drills.
- **No replay strategy** — cannot re-run history against a new pattern → cannot validate or recover missed detections. Keep streams within retention; automate replay.
- **Over-complex patterns** — 15-step mega-patterns are unmaintainable and unexplainable; break into tiers of derived events (Section 7.1).
- **Missing data quality checks** — late, duplicate, corrupt, or schema-violating events silently poison patterns. Validate at ingestion; quarantine bad events to a DLQ.
- **Not handling out-of-order events** — no watermark/late policy → matches computed on partial or shuffled views. Define and test the late-data behavior explicitly.
- **Patterns as the only line of defense** — no ML scoring, no feedback, no calibration → precision collapses. The hybrid (Section 7.4) is the standard for a reason.

---

## 12. Future Directions

### 12.1 CEP + GenAI

- **LLM-assisted pattern design** — describe the typology in natural language ("deposit, then rapid withdrawal of most of the funds within 6 hours, from an account with little prior activity") and generate Flink CEP / EPL / MATCH_RECOGNIZE code, with a golden-dataset test harness validating it. The bottleneck shifts from syntax to scenario validation.
- **Natural language alerts** — the LLM summarizes a pattern match into an investigator-ready narrative: "Why did this alert fire" paragraphs generated from the matched-event trail — dramatically faster triage.
- **GenAI for alert triage** — LLM pre-screening of alert queues (summarize, rank, group duplicates) reduces the false-positive load before a human looks.
- **Agentic CEP** — agents that observe pattern yield and adjust thresholds, suppression rules, and routing within governed guardrails — the feedback loop (Section 9.2) becomes automated.

### 12.2 CEP in the Lakehouse Era

Streaming and batch are converging: MATCH_RECOGNIZE now exists in warehouses and streaming databases (RisingWave, Materialize, Snowflake, BigQuery), Flink SQL runs everywhere, and the same pattern definition can run on streaming and historical data. The practical consequence: patterns are tested on the lakehouse, then deployed to the stream, with the lakehouse also serving as the replay/audit substrate.

### 12.3 CEP + Data Streaming + RAG

Real-time context feeds LLM applications: CEP-derived situation events (fraud alerts, risk events, market anomalies) become fresh context for RAG pipelines — see [rag_with_data_streaming_guide.md](ai_llm/rag_with_data_streaming_guide.md) for the streaming-RAG architecture. A compliance assistant can answer "what happened with account X in the last hour" from CEP-derived situation events rather than stale batch indexes.

### 12.4 Adaptive, Threshold-less CEP

The long-term direction is ML-driven thresholds: patterns that hold shape (the sequence, the correlation) but whose thresholds are set and re-set by models from labeled feedback — reducing the calibration burden and the threshold brittleness that Section 11 warns about. Self-tuning patterns plus agentic triage is the likely end-state of the hybrid architecture.

---

## 13. Conclusion

Complex Event Processing answers the question no single event can: *what is happening across the flow of events?* Born from academic event-algebra research in the 1990s and hardened on trading floors in the 2000s, CEP is now a mainstream banking capability — the temporal, stateful, explainable layer that detects card fraud, AML typologies, market abuse, and operational failure in real time. The practical architecture has converged: Kafka for ingestion, a pattern engine (Flink CEP first among equals for new builds) for detection, case management for investigation, and a feedback loop that keeps patterns calibrated — with ML scoring layered on top for precision, and GenAI moving in to help author, explain, and triage.

The recurring lesson for architects: CEP is a *discipline* as much as a technology. The pattern catalog, the calibration discipline, the event-time rigor, the state hygiene, and the investigator feedback loop decide whether a CEP platform is a precision detection instrument or another source of alert noise. Choose the engine by pattern complexity and team skills, keep patterns simple and tiered, and treat the feedback loop as non-negotiable. Done well, CEP gives banking what it needs most: real-time detection that is fast, scalable, and — critically — explainable to the people who must act on it and to the regulators who hold the bank to account.
