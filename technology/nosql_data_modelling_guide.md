# NoSQL Data Modelling: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Data Architecture / Data Modeling — NoSQL Data Models, Query-Driven Design, Banking  (Data Engineering series)  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [NoSQL Data Modelling Foundations](#1-nosql-data-modelling-foundations)
2. [The Document Data Model](#2-the-document-data-model)
3. [The Key-Value Data Model](#3-the-key-value-data-model)
4. [The Wide-Column Data Model](#4-the-wide-column-data-model)
5. [The Graph Data Model](#5-the-graph-data-model)
6. [Cross-Cutting NoSQL Modelling Patterns](#6-cross-cutting-nosql-modelling-patterns)
7. [Modelling for Specific Systems](#7-modelling-for-specific-systems)
8. [The NoSQL Modelling Process and Methodology](#8-the-nosql-modelling-process-and-methodology)
9. [Schema Evolution and Migration](#9-schema-evolution-and-migration)
10. [Banking Context: NoSQL in Financial Services](#10-banking-context-nosql-in-financial-services)
11. [Worked Examples](#11-worked-examples)
12. [The Future: 2026 and Beyond](#12-the-future-2026-and-beyond)
13. [Glossary](#13-glossary)

---

## 1. NoSQL Data Modelling Foundations

### 1.1 What NoSQL Data Modelling Is

**NoSQL data modelling** is the discipline of designing the structure of data for non-relational (NoSQL) storage engines. Where relational modelling asks *"what are the entities and how do they relate?"*, NoSQL modelling asks *"what queries will this store serve, and what shape of data makes those queries cheap?"* The philosophy is often compressed into two slogans:

- **Data model first, query first** — you do not start from entities and normalize; you start from the access patterns (the queries, the reads, the writes, the SLAs) and derive the physical shape of the data from them.
- **Model FOR the queries, not the entities** — the data structure is a projection of the application's workload, not a platonic description of the business domain.

A relational schema answers *"what is true about the business?"* A NoSQL schema answers *"how will this data be read and written?"* The same business fact can legitimately be stored in several different NoSQL shapes depending on who reads it, how often, and at what latency.

### 1.2 Fundamental Differences from Relational Modelling

The relational model — see [data_model_resource_book_guide.md](data_model_resource_book_guide.md) for the canonical treatment — optimizes for *data integrity and flexibility of ad-hoc query*: normalize into entities, store facts once, join at query time, and enforce the schema on write. NoSQL inverts nearly every one of those choices.

| Dimension | Relational modelling | NoSQL modelling |
|---|---|---|
| Starting point | Entities → tables (data-driven) | Queries → data shape (query-driven) |
| Normalization | Normalize to 3NF; store each fact once | Denormalize deliberately; duplicate facts across read models |
| Unit of storage | Row in a table | Aggregate (document), key-value pair, row in a partition, node+relationship |
| Related data | Foreign keys, joined at query time | Embedded / duplicated / application-joined; joins avoided |
| Schema | Schema-on-write, enforced by DDL | Schema-on-read, flexible, often implicit in application code |
| Transactions | ACID across tables | Single-unit atomicity (document, key, partition); multi-unit ops via application or limited transactions |
| Query model | SQL, joins, arbitrary predicates | Pre-defined access paths; query-by-key, scan, traversal |
| Consistency | Strong by default | Eventual / tunable per operation |
| Scaling | Vertical (or sharded with pain) | Horizontal by design (partition/row-key distribution) |

**The query-driven design is the opposite of relational's data-driven design.** In relational modelling you normalize first and let the query engine figure out how to serve any query. In NoSQL the query engine is deliberately weak (no joins, no general predicates) and the data shape *is* the query plan: if you can name the query, you can build a data shape that serves it in one read; if you cannot, no amount of indexing will save you.

### 1.3 The NoSQL Modelling Process

The canonical NoSQL modelling workflow is a four-step loop:

1. **Understand the queries.** Enumerate every access pattern: the reads (by what key? what range? what filter? what sort order? what latency?), the writes (frequency, size, atomicity needs), and the volume/velocity (rows per second, data per key). This is the requirements contract for the model.
2. **Choose the data model (family).** Decide whether the store should be document, key-value, wide-column, or graph — driven by the query shape, not fashion (see §8.2 for the selection decision tree).
3. **Denormalize.** Fold the data into the shape that the queries want: embed related data, duplicate read-mostly fields, pre-join, pre-aggregate. Duplication is the *point*, not a defect.
4. **Optimize for access.** Design keys, partitions, indexes, TTLs, and bucket sizes so every hot query hits exactly one unit (one document, one key, one partition) with minimal data read.

The loop is iterative: a new query discovered in step 5 (validation) usually forces a change to the shape — which is why NoSQL schemas evolve continuously (see §9).

The most common failure mode is **relational-mindset leakage**: a team with a relational background maps entities to collections/tables, normalizes, and reaches for joins — and then discovers the store cannot serve the resulting queries efficiently. The tell-tale symptoms are N+1 read storms, full scans, and application code doing joins in memory. The remedy is not better indexing; it is redoing the model from the access-pattern catalogue. Every design review in this guide's methodology (§8) starts by checking that each query hits exactly one unit — if it doesn't, the model leaked.

### 1.4 The Four NoSQL Families

The "NoSQL" umbrella covers four radically different storage paradigms. The modelling techniques are family-specific; a document model and a wide-column model share almost nothing except the query-first philosophy.

| Family | Data unit | Query model | Best for | Examples |
|---|---|---|---|---|
| **Document stores** | Self-contained JSON/BSON document (aggregate) | Query by document key, field indexes, aggregation pipelines; some join support | Semi-structured records read/written as wholes; flexible schemas; customer 360s, orders, catalogs | MongoDB, Couchbase, DynamoDB (also KV), Firestore, Cosmos DB |
| **Key-value stores** | Opaque or structured value addressed by a single key | Get/put/delete by key; range scans (some); app-side indexing | Ultra-low-latency lookups, caches, sessions, rate limits, leaderboards | Redis, Memcached, DynamoDB, etcd, RocksDB |
| **Wide-column stores** | Row in a column family, addressed by partition key + clustering columns | Query by partition key, ordered scan within partition; no joins | Massive write scale, time-series, event/transaction logs, query-by-key with range | Cassandra, HBase, ScyllaDB, Bigtable, Cloud Spanner (also relational) |
| **Graph databases** | Node + relationship + properties | Traversal from a start node over typed relationships | Connected data: link analysis, fraud rings, entitlements, network paths | Neo4j, Amazon Neptune, ArangoDB, JanusGraph |

> **Cross-reference:** graph modelling is covered in depth in [neo4j_graph_database_guide.md](neo4j_graph_database_guide.md); this guide covers the graph family at the modelling-pattern level in §5 and defers the deep-dive.

### 1.5 When NoSQL Modelling Applies (and When It Doesn't)

NoSQL modelling is the right discipline when the workload has *one or more* of these properties:

- **A known, bounded set of access patterns.** The queries are enumerable in advance and stable enough to bake into the physical shape. Ad-hoc, unpredictable analytical queries are the relational/data-warehouse domain (see [data_model_resource_book_guide.md](data_model_resource_book_guide.md) and the dimensional treatment in [data/types_of_dimensions_data_warehousing.md](data/types_of_dimensions_data_warehousing.md)).
- **Scale or latency that a single relational node cannot meet** — millions of writes/sec (wide-column), sub-millisecond reads on the transaction path (key-value), or data volumes that shard naturally (partition key / document key as the shard).
- **Semi-structured or polymorphic data** whose shape evolves faster than DDL cycles — the document store's schema-on-read absorbs this.
- **A natural aggregate** — data that is always read and written as a unit (orders, customer profiles, device telemetry windows).

It does **not** apply when the workload is: multi-entity ACID transactions with arbitrary joins (relational SoR), complex ad-hoc reporting (warehouse/OLAP), or metrics/telemetry time series at scale (purpose-built TSDB, §6.4). The selection decision tree in §8.2 formalizes this boundary.

---

## 2. The Document Data Model

### 2.1 The Document as the Aggregate

The document store's unit of modelling is the **document**: a self-contained, self-describing JSON (or BSON, in MongoDB) structure. The defining modelling move is that **a document is an aggregate** — related data that is read together is *physically* stored together, inside one document, in one read.

```
// The aggregate principle: one read returns the whole business object
{
  "orderId": "ORD-88213",
  "customerId": "CUST-4401",
  "placedAt": "2026-08-01T10:24:00Z",
  "status": "CONFIRMED",
  "total": 1249.50,
  "currency": "SGD",
  "lineItems": [                      // embedded — read with the order, always
    { "sku": "FX-SPOT-01", "qty": 1, "unitPrice": 1200.00 },
    { "sku": "FEE-TT-02",  "qty": 1, "unitPrice": 49.50 }
  ]
}
```

The document is the **atomic unit of consistency and the atomic unit of performance**: one write commits the whole document, one read fetches the whole document. This makes the central document-modelling question *"what belongs inside the document?"* — i.e., *"what is the aggregate?"* — which is the same question DDD asks (see §6.3).

### 2.2 The Classic Decision: Embedding vs Referencing

Document modelling reduces to one recurring decision: for any two related pieces of data, do you **embed** them in one document, or do you **reference** between documents (store the related document's ID)?

**Embed** — put the related data inside the parent document:
- One read returns everything; no second lookup, no join.
- Updates to the whole aggregate are atomic (single-document writes).
- Cost: the parent document grows; every update rewrites the embedded copy; data that is shared (embedded in many parents) is duplicated and can drift.

**Reference** — store only the ID of the related document:
- Related data lives once; it can be large, unbounded, and independently updated.
- Reads of parent + related require two (or N) queries; the application joins.
- Cost: N+1 reads, and no atomicity across the two documents (unless the platform supports multi-document transactions, e.g., MongoDB 4.0+).

### 2.3 Embedding vs Referencing Decision Table

| When to **embed** | When to **reference** |
|---|---|
| Data is always read together with the parent (the read-together rule) | Data is read independently, or with many different parents |
| One-to-many where the "many" is small and bounded | One-to-many where the "many" is unbounded (comments on a post, transactions on an account) |
| The child has no life of its own (line items, addresses, attributes) | The child is shared/referenced by many parents (a customer, a product master, a rate) |
| Updates must be atomic across parent + child | Child updates frequently and independently (updating a child would rewrite the parent) |
| Data volume is bounded by the document limit (e.g., 16 MB in MongoDB) | Data volume can exceed the document size limit |
| Query wants the whole object in one shot | Query wants subsets, aggregations across many children, or cross-entity analytics |

The rule of thumb: **embed by default, reference when the relationship is unbounded, shared, or independently updated.** And when in doubt, remember that the read pattern decides: data read together lives together.

### 2.4 Document Structure Patterns

**One-to-one** — always embed. A customer's profile and its KYC flags are one document. There is no scenario where a 1:1 relationship justifies a reference except document-size pressure.

**One-to-many** — two options:
- *Embed an array* when the many-side is bounded and always read with the parent (order → line items, customer → addresses).
- *Reference* when the many-side is unbounded or independently queried (customer → transactions: you never want the transaction history inside the customer document).

**Many-to-many** — reference, on both sides. An order references products; a product does not embed its orders. The application resolves the links (or a junction collection with IDs on both ends).

**Tree / hierarchy** — three classic encodings:

| Pattern | Shape | Read cost | Write cost | Best for |
|---|---|---|---|---|
| Nested documents | Children embedded recursively in the parent | One read for a whole subtree | Rewrites the whole subtree | Small, bounded hierarchies (org chart of 20 people) |
| Materialized paths | Each node stores its full ancestor path (`"a.b.c.d"`) | Regex/prefix query for descendants | Single-node write | Mid-size trees with path queries |
| Adjacency (references) | Each node stores its parent ID | Recursive/N+1 reads, or graph traversal | Single-node write | Deep, unbounded, frequently mutated trees |

The materialized-path encoding, concretely — a product category tree in a `categories` collection:

```
{ "_id": "cat:fx",   "name": "FX",         "path": "fx" }
{ "_id": "cat:spot", "name": "FX Spot",    "path": "fx.spot" }
{ "_id": "cat:fw",   "name": "FX Forwards","path": "fx.fw" }
// All descendants of "fx":  find({ path: /^fx\./ })
// Depth of a node:          path.split('.').length
```

Nested documents give the same "subtree in one read" property but force rewrites of the whole subtree on any mutation; materialized paths move that cost to a regex/prefix query while keeping writes single-node. For hierarchies that are deep *and* frequently restructured, or where you need arbitrary path queries, a graph model (§5) is often the better fit.

**Polymorphic documents** — one collection holds documents of different shapes, distinguished by a `type` (or `schemaVersion`) field. This is the document store's answer to inheritance and to the "one table for many similar records" problem (e.g., a `payments` collection holding `MT103`, `MT202`, `SEPA` documents with different fields). The schema-on-read contract is: *the application branches on `type` before touching fields* — the model must never assume a field exists.

```
{ "_id": "PAY-1", "type": "MT103", "amount": 1000.00, "ccy": "USD",
  "orderingParty": {...}, "beneficiary": {...}, "charges": "SHA" }
{ "_id": "PAY-2", "type": "SEPA", "amount": 250.00, "ccy": "EUR",
  "iban": "DE89...", "bic": "COBADEFFXXX", "endToEndId": "E2E-991" }
```

**Document versioning** — when a document is updated in place, history is lost. For audit-relevant documents (contracts, limits, KYC records), the patterns are: (a) keep a `version` field and store previous versions in a separate `_history` collection; (b) append-only event documents (see event sourcing, §6.5); (c) immutable documents with new IDs per version plus a `supersedes` link. Banking audit requirements (see §10.2) usually force one of these.

### 2.5 MongoDB Modelling Specifics

- **The 16 MB document limit.** MongoDB caps a single document (BSON) at 16 MB. This is the hard ceiling on embedding: an unbounded child collection (transactions, audit events) can *never* be embedded — it must be referenced or bucketed (see the bucket pattern below). The limit also means embedded arrays must be size-budgeted: 100 line items × 1 KB = fine; 100,000 events × 200 B = 20 MB — too big.
- **BSON types.** Documents are stored as BSON (Binary JSON) with typed values: string, int32/int64, double, decimal128 (important for money — binary floating point is not acceptable for currency), ObjectId, Date, embedded documents, arrays, and more. Modelling choice: use `decimal128` for monetary fields, `Date` for timestamps, and `ObjectId`/natural keys deliberately (natural business keys are often better for idempotency in banking).
- **Atomicity.** Writes are atomic at the *single document* level. MongoDB 4.0+ added multi-document ACID transactions (with a replica set, then sharded clusters in 4.2+), but the design implication stands: **keep the atomic unit in one document**. If an operation must be all-or-nothing (debit + credit, order + inventory decrement), either put both in one document or use a transaction — and the latter carries real performance cost in sharded clusters.
- **Indexes are query-driven.** Index design starts from the query list: every recurring query gets a supporting index (equality fields first, then sort field, then range field). MongoDB only uses one index per query clause, so composite index *order* matters: `{customerId: 1, timestamp: -1}` serves "transactions of customer X, newest first" in one index scan.
- **The aggregation pipeline.** MongoDB's `$match → $sort → $group → $project` pipeline is a mini-ETL inside the database. Modelling implication: you can keep documents in a write-optimized shape and derive read shapes (rollups, joins via `$lookup`) in the pipeline — but pipelines that run per-request are a smell; materialize frequent aggregations (see CQRS, §6.2).

### 2.6 Worked Document Patterns

**The e-commerce order (the aggregate pattern).** The order document embeds its line items, shipping address, and totals — the whole order is one document, one write, one read. This is the canonical aggregate: the order is the consistency boundary; nothing about the order exists outside it.

**Customer + orders (the embed vs reference decision).** The customer document embeds profile data and *references* orders — because orders are unbounded, independently updated, and queried separately ("recent orders", "orders by status"). The decision table (§2.3) resolves this instantly: unbounded → reference.

```
// customers collection
{ "_id": "CUST-4401", "name": "...", "segment": "CORP", "addresses": [ ... ] }
// orders collection — referenced by customerId, NOT embedded
{ "_id": "ORD-88213", "customerId": "CUST-4401", "placedAt": "...", "lineItems": [ ... ] }
// Query "orders of customer, newest first" -> index { customerId: 1, placedAt: -1 }
```

The reference costs one extra index and one extra query per screen; it buys unbounded order counts, independent order lifecycle updates, and order-centric queries. If the product instead needed "the customer with their last 5 orders on one screen", the model would embed a *bounded* summary array (`recentOrders: [{orderId, total, date}]`) alongside the reference — the read-together rule applied twice, at two granularities.

**Time-series (the bucket pattern).** Raw event-per-document (one document per sensor reading per second) explodes document counts and index sizes. The **bucket pattern** groups time-ordered events into fixed windows inside one document:

```
// Bucket: one document per device per hour
{ "deviceId": "ATM-014", "bucketStart": ISODate("2026-08-01T10:00:00Z"),
  "count": 3600, "min": 100.0, "max": 9999.0,
  "readings": [ { "t": "10:00:01", "v": 104.5 }, ... ] }
```

This cuts document count by orders of magnitude, enables cheap `$min/$max/$avg` rollups on the bucket, and naturally bounds document size (see the time-series note in §6.4 for the general treatment).

**The transactional outbox (event publishing).** When a write to the database must also publish an event (e.g., "order confirmed" → Kafka), the **outbox pattern** writes the event into an `outbox` collection *in the same transaction* as the business write — a separate process then publishes outbox rows to the broker and deletes them. The outbox gives atomicity between the database state and the event stream without distributed transactions. See [event_stream_processing_guide.md](event_stream_processing_guide.md) for the eventing side; the outbox also appears in §6.1 as the consistency-handling pattern.

---

## 3. The Key-Value Data Model

### 3.1 Key Design: The Key Is the Model

In a key-value store, **the key is the entire data model**. There is no schema beyond the key space; every access pattern must be expressible as "give me the value(s) for key(s) X". Key design therefore *is* query design:

- **Key naming conventions.** Keys are strings (Redis) or composite of partition/sort (DynamoDB). Conventional forms: `namespace:entity:id` (`rate:USD:SGD`, `session:usr-4401`), or hierarchical paths (`acct:88213:txn:20260801`) that exploit lexicographic ordering for range scans.
- **Composite keys.** Since most KV stores scan keys lexicographically, you can model a secondary dimension into the key: `txn:2026-08:day:01` sorts day 01 before day 02. The composite key is the poor man's index — its *order* is the query's sort order.
- **Hierarchical keys** extend the same idea across dimensions: `tenant:entity:year:month:day:id` gives you range scans at *any* prefix (`tenant:entity:2026:08:*` = all of August). The cost is that each level is fixed-width and zero-padded, so the key space must be planned before data lands — another reason key format changes are rebuilds (§9.3).
- **The primary key** is the only first-class access path. Everything else is derived.

### 3.2 Key Patterns

**Secondary indexes (application-managed).** KV stores have no secondary indexes, so you build them: for every non-key attribute you need to query by, maintain a *separate* key space whose key is that attribute and whose value is the primary key (or a set of primary keys).

```
// Primary space
customer:4401  ->  { ...customer JSON... }
// Application-managed secondary index (by email)
idx:email:jliu@bank.sg  ->  "customer:4401"
// Set-based secondary index (by segment)
idx:segment:CORP  ->  SET { "customer:4401", "customer:7712", ... }
```

Every index entry is a key-value pair written by the *application* on each mutation — which is exactly how DynamoDB GSI and Cassandra secondary indexes work internally, and why application-managed indexes are the source of the dual-write consistency problem (§6.1).

**Caching (cache-aside).** The canonical KV use: on read, check the cache (`GET key`); on miss, read the source of truth, store under the key with a **TTL** (time-to-live) so the cache self-heals; on write, invalidate (or update) the cache entry. The TTL is a modelling parameter: too short → cache misses; too long → stale data. See the Redis patterns in §3.5.

**Hot keys (the hotspot problem).** If one key absorbs a disproportionate share of reads/writes (a celebrity account, a viral instrument price, a single rate pair at fixings), that single node becomes a bottleneck: the partition containing the hot key saturates while its peers idle. Mitigations: (a) shard the hot key into N sub-keys (`rate:USD:SGD:0` … `rate:USD:SGD:63`) and merge on read; (b) replicate the hot key to more nodes (read replicas) — but only for idempotent reads; (c) move the hot data into an in-memory layer (Redis) in front of the persistent store. Hot keys are the same problem as hot partitions in wide-column stores (§4.5) and DynamoDB (§3.6).

### 3.3 Value Design

- **Opaque blob vs structured.** The value can be an opaque byte string (Memcached, Redis STRING) or structured (Redis HASH, JSON in DynamoDB, typed in etcd). Modelling rule: if the application *always* reads the whole value, an opaque blob is fastest; if it reads or updates *fields* (increment a counter, patch one field), use a structured value so the store can do the partial work without a read-modify-write in the client.
- **Value size.** Large values (multi-MB) are an anti-pattern in every KV store: they inflate memory (Redis), blow up request latency (DynamoDB 400 KB item cap), and amplify write amplification. Break large blobs into chunks or push them to object storage and keep only the reference + metadata in the KV store.

### 3.4 Redis Modelling

Redis is an in-memory key-value store whose "value" is one of several **data structures** — and choosing the structure *is* the modelling step.

| Structure | Shape | Model as | Example |
|---|---|---|---|
| STRING | scalar bytes | simple values, counters, flags, cached JSON | `rate:USD:SGD = 1.3492` |
| LIST | ordered collection | queues, recent-item logs (LPUSH/LRANGE) | `queue:payment:sepa` |
| SET | unordered unique members | membership, tags, dedup | `fraud:blocklist:card` |
| HASH | field-value map | objects (fields updated independently) | `customer:4401 { name, tier, limit }` |
| ZSET | sorted set (member + score) | rankings, leaderboards, sliding windows, scheduled work | `leaderboard:fx { "USD": 1.3492, ... }` |

**Modelling with the structures:**
- **HASH for objects** — a customer record as a HASH lets you update `limit` without rewriting name/address; memory-efficient vs many small STRINGs.
- **ZSET for rankings/leaderboards** — the score is the rank key; `ZRANGE` gives top-N in O(log n). Also used for the sliding-window rate limiter: each request is a ZSET member scored by timestamp; the window count is `ZCOUNT(now - window, now)`.
- **LIST for queues** — LPUSH/BRPOP implements a FIFO work queue; Redis Streams (a log structure) supersedes LIST for durable, consumer-group messaging — see [kafka_alternatives_guide.md](kafka_alternatives_guide.md) §on Redis Streams.

**Redis patterns:**
- **Cache-aside** — §3.2 with TTL; Redis is the default cache layer for reference data (rates, limits — see §10.1).
- **Leaderboard** — ZSET, described above.
- **Rate limiting (sliding window)** — ZSET per subject (`ratelimit:api:user-4401`) with score = epoch ms; count requests in `[now-60s, now]`; expire the key after the window. Fixed-window counters (INCR + EXPIRE) are cheaper but bursty at boundaries.
- **Session store** — key = session token, value = HASH (subject, expiry, consent flags), TTL = session lifetime; reads are single GETs at auth time.
- **Distributed lock (Redlock)** — `SET lock:key token NX PX 30000` acquires a lock that expires; the token (a UUID) is verified on release so a lock holder can't release someone else's lock. Redlock is controversial (it is not safe under clock jumps/GC pauses without fencing tokens — the classic Martin Kleppmann vs Salvatore Sanfilippo debate); for banking-grade correctness use fencing tokens or a consensus store (etcd, ZooKeeper) instead.
- **Pub/sub** — fire-and-forget fan-out (`PUBLISH`/`SUBSCRIBE`); no persistence, no replay — fine for cache invalidation signals, not for event delivery guarantees (that's Kafka's job; see [event_stream_processing_guide.md](event_stream_processing_guide.md)).

### 3.5 DynamoDB Modelling

DynamoDB is a key-value *and* document store (items are attribute maps) with a distinctive modelling discipline.

**The composite primary key.** Every table has a **partition key** (hash) and optionally a **sort key** (range). The partition key determines physical placement (which partition/shard); the sort key determines order *within* the partition. Query = exact partition key + optional sort-key range. This one decision — "what is my partition key, what is my sort key" — is the whole DynamoDB model.

**Single-table design (the "one table" pattern).** DynamoDB's philosophy is **"one table, many access patterns"**: instead of one table per entity (relational habit), you put *all* items in one table and disambiguate with a **type attribute** (item type) plus overloaded partition/sort keys. Access patterns are served by choosing the right (partition key, sort key) *prefix*:

```
// One table, many item types
PK            SK                      type      data
CUST#4401     CUST#4401               CUSTOMER  {name, tier, kyc}
CUST#4401     ACCT#88213              ACCOUNT   {balance, ccy}
ACCT#88213    TXN#2026-08-01#001      TXN       {amount, ...}
RATE#USD      RATE#SGD                RATE      {bid, ask, ts}
```

- "All accounts of customer 4401": `Query(PK=CUST#4401, SK begins_with "ACCT#")`
- "All transactions of account 88213 in August": `Query(PK=ACCT#88213, SK between "TXN#2026-08-01" and "TXN#2026-08-31")`

The single-table pattern exists because DynamoDB charges per table and throttles per partition — consolidating items concentrates access patterns and reduces the number of GSIs needed.

**GSIs vs LSIs.** A **global secondary index (GSI)** is a full re-projection of the table under a *different* partition/sort key, maintained asynchronously; you pay storage + write cost for it and it lets you query by any attribute (`GSI1PK = EMAIL#jliu@bank.sg`). A **local secondary index (LSI)** offers an alternative sort key *within the same partition key* — but LSIs are fixed at table creation and limited (20 GB per partition key value), so GSIs dominate modern designs. Rule: GSI for new access patterns, LSI only when you need consistent secondary reads within a partition and knew the pattern at creation time.

**Access patterns as the model.** The DynamoDB modelling workflow is: list every access pattern → assign each a (PK, SK) shape → add GSIs only for patterns the base key can't serve → *never* scan. If a query needs a full scan, the model is wrong.

A GSI in the single-table design, concretely — "find the customer by email" (login/onboarding) is not servable by `CUST#`-prefixed keys, so it gets a GSI:

```
// GSI1: GSI1PK = EMAIL#<address>   (projected attributes: PK, SK, name, tier)
EMAIL#jliu@bank.sg  ->  { PK: "CUST#4401", SK: "CUST#4401" }
// Query: Query(GSI1, GSI1PK = "EMAIL#jliu@bank.sg") -> then GetItem(PK, SK)
```

Each GSI is a second write to the table (asynchronous, billed, eventually consistent) — so the rule is *one GSI per genuinely separate access pattern*, and never a GSI that merely re-orders the same pattern.

**Hot partitions.** A partition key value that receives too much traffic (one customer with 10× the activity) throttles *that partition* even when the table has spare capacity. Mitigations: shard the hot key (`CUST#4401#0..N`), spread by adding a random suffix, or restructure the model so hot entities are split across keys (e.g., current-day data under `TXN#2026-08-01` keys, then roll over daily).

**Capacity: on-demand vs provisioned.** Provisioned mode (read/write capacity units with auto-scaling) is cheaper for predictable workloads; on-demand (per-request billing) absorbs spikes but costs more at steady state. Modelling implication: bursty banking workloads (month-end, fixings) often justify on-demand, and hot partition design matters *more* under provisioned mode because a hot partition throttles while other partitions idle.

---

## 4. The Wide-Column Data Model

### 4.1 The Column-Family Model

Wide-column stores (Cassandra, HBase, ScyllaDB, Bigtable) look like tables but are physically **nested sorted maps**. The Cassandra model's vocabulary is the modelling vocabulary:

- **Keyspace** — the database; owns replication settings (replication factor, network topology strategy).
- **Table** — a column family; the unit of schema and of query design.
- **Partition key** — the first part of the primary key; the hash that decides *which node* holds the row(s). All rows with the same partition key live on one node, in one partition.
- **Clustering columns** — the rest of the primary key; they define the *sort order within the partition*. They are not hashed; they order.
- **Columns** — the remaining fields; physically stored per-row (wide-column = each row can have different columns, though schema-on-write defines the set).

```
CREATE TABLE txn_by_account (
  account_id   text,      -- partition key: WHERE account_id = ?
  txn_time     timestamp, -- clustering col 1: ORDER BY txn_time
  seq          int,       -- clustering col 2: uniqueness within same timestamp
  amount       decimal,
  counterparty text,
  status       text,
  PRIMARY KEY ((account_id), txn_time, seq)
);
```

The physical layout is: partition (sorted by partition key hash) → within partition, rows sorted by clustering columns. **The clustering order *is* the query's sort order** — that is the entire trick of wide-column modelling.

### 4.2 Query-First Design: One Table Per Query

Cassandra's golden rule: **design the table per query** — "one table per query", "denormalization by design". You do not model entities and hope; you take each query and build a table whose primary key serves it in exactly one partition read. If two queries need different shapes of the same data, you maintain *two tables* with the same data.

```
-- Query 1: "transactions of account X, newest first"
CREATE TABLE txn_by_account (account_id text, txn_time timestamp, ...,
  PRIMARY KEY ((account_id), txn_time)) WITH CLUSTERING ORDER BY (txn_time DESC);
-- Query 2: "transactions by counterparty" -> needs its own table
CREATE TABLE txn_by_counterparty (counterparty text, txn_time timestamp, ...,
  PRIMARY KEY ((counterparty), txn_time));
```

The application writes both tables (a dual-write — see §6.1 for the consistency handling). This is not a hack; it is the designed behaviour of the system.

### 4.3 Primary Key Design: Distribution + Order

The primary key has exactly two jobs, and they map to its two parts:

- **Partition key = data distribution.** Choose it so data spreads evenly across nodes (high cardinality, no hot entity) *and* queries hit one partition (the query always filters on it). Conflict: even distribution wants many small partitions; query efficiency wants the whole answer in one partition. Resolution: pick the natural query grain (account, device, customer) and accept the partition-size trade-off — see partition sizing below.
- **Clustering columns = sort order within the partition.** Choose them so the query's range filter and ORDER BY fall out of the physical order. Common patterns: `(entity, time)` for "all rows of entity ordered by time", with `DESC` clustering for newest-first (the 99% banking case).

### 4.4 The Modelling Rules

| Rule | Meaning |
|---|---|
| No joins | Related data is pre-joined into the queried shape; the application (or a second table) handles what a join would have |
| No aggregates | `COUNT`/`SUM`/`AVG` are not allowed in queries (Cassandra) — counters and rollups are *maintained* as data (counter tables, materialized views, application-side rollups) |
| Denormalize everything | The same fact appears in every table that needs it; duplication is normal, expected, and the design |
| Data duplication is normal | Storage is cheap; a query served from one partition is priceless |

### 4.5 Cassandra Patterns

- **Time-series (partition per time bucket).** Raw event rows clustered by timestamp grow a partition without bound — bad (partitions have practical size limits ~100 MB; large partitions slow reads and repairs). The pattern: bucket the partition key by time — `PRIMARY KEY ((account_id, day), txn_time)` — so each partition holds one account-day. Queries for a range span multiple partitions (one per day) and the application fans out; unbounded growth becomes bounded per-bucket.
- **Counter tables.** For "how many/ how much" (hits per endpoint, balances of aggregates): a table whose columns are counters, updated with `UPDATE ... SET c = c + 1`. Counters are the sanctioned way to do increments; they cannot be used in the same table as non-counter columns.
- **Materialized views.** Cassandra can maintain a second table automatically from a base table (base partition key + additional clustering), so a query shape that differs only in key/order gets a maintained projection. Caveats: limited column set (no multiple non-key columns in the key), known operational sharp edges (view rebuilds, write amplification) — many teams build the second table manually instead.
- **Collection types (SET/LIST/MAP).** Columns can be collections: tags as SET, event lists as LIST, attributes as MAP. Rules: collections are read whole (don't index into them), keep them small (a few hundred items), never use them for unbounded data — that's what rows are for.
- **User-defined types (UDTs).** A named composite type (`address`, `money_amount`) used as a column type — gives structure without a table; good for "value objects" embedded per row (the wide-column equivalent of embedding).

```sql
CREATE TYPE money_amount (amount decimal, ccy text);
CREATE TABLE limit_events (
  limit_id text, event_time timestamp, seq int,
  delta money_amount,                        -- UDT column: structured value per row
  tags SET<text>,                            -- collection: small membership set
  attributes MAP<text,text>,                 -- collection: sparse attribute map
  PRIMARY KEY ((limit_id), event_time, seq)
);
```

Rules that follow: collections are read whole and must stay small (a few hundred entries); anything that grows unbounded belongs in *rows* (clustering), not in a collection column.
- **The wide row.** The "wide" in wide-column: a single partition can hold millions of rows ordered by clustering columns — each row an event, the partition a time-ordered log (`PRIMARY KEY ((account_id), event_time, seq)` = a per-account event stream, served by one partition scan). Bigtable/HBase are designed for exactly this (a row = a key with many column families/versions); Cassandra achieves it via clustering.

### 4.6 The Cassandra Modelling Process

The workflow is a strict sequence — the reverse of relational design:

1. **Queries** — enumerate every query with its filter keys, range, sort order, and read SLA. No query, no table.
2. **Table design** — one table per query (or per query family sharing a shape).
3. **Partition key choice** — the query's equality filter; check cardinality and evenness of distribution; decide bucketing (time buckets for time-series).
4. **Clustering** — the query's range + sort; set `CLUSTERING ORDER BY` to match the hot sort direction.
5. **Test** — load-test with production-like data volumes: partition size (aim ≤ ~10–50 MB, or ≤ 100K rows), read latency at p99, write amplification from the dual tables, tombstone pressure (see §7.3).

### 4.7 The HBase / Bigtable Model

HBase and Bigtable are sorted-map stores: a **row key** is a byte string; rows are ordered *lexicographically* by row key across the table; each row holds column families of cells with versions. **The row key is the model** — there is no partition/clustering split, just one ordered key space.

- **Key ordering is physical.** Because rows sort by key, adjacent keys live on the same region/tablet and are scanned together. You model *range scans* into the key: `customerId|timestamp` lets you scan a customer's data over a time range in one contiguous read.
- **Time-series row keys.** Naive `timestamp`-first keys (`20260801T102400|ACCT88213`) cause **hotspotting**: all writes for the current minute land on one region. Fixes: **reverse the timestamp** (`maxTime - t`) so newest data sorts *before* old (still one region for new writes — actually the classic fix is), or **salt** the key (`partition|timestamp|id`, where `partition = hash(id) % N`) to spread writes across N regions, or **key-shard** by a business dimension. The canonical Bigtable guidance: monotonically increasing keys = write hotspot; randomize or bucket the leading component.
- **Column families** are the schema's top level (rarely more than a handful — each family is a separate storage unit); cells are versioned by timestamp, and TTL/version-count policies do retention.

---

## 5. The Graph Data Model

### 5.1 The Property Graph Model

Graph databases model data as **nodes** (entities), **relationships** (typed edges), and **properties** (attributes on either). The relationship is a first-class citizen — it is stored as data, not derived from foreign keys at query time:

```
(:Customer {id: "CUST-4401", name: "Jack Liu", segment: "CORP"})
  -[:HOLDS {since: "2019-03-01", role: "SIGNATORY"}]->(:Account {id: "ACCT-88213"})
  -[:DEBITED_FROM]->(:Transaction {id: "TXN-99", amount: 1200.00})
(:Transaction)-[:INVOLVES]->(:Counterparty {id: "CP-77", name: "Shurui Trading"})
```

### 5.2 Graph Modelling Patterns

- **Node vs property.** A thing that is *queried, filtered, or connected* becomes a node; a thing that is only *displayed* stays a property. A customer's phone number is a property; if you run "find all customers sharing a phone number" (fraud typology), the phone becomes a node. The test is always: *will I traverse through it, or filter on it?*
- **Relationship design.** Relationships are typed and can carry properties (since-date, role, weight, status). Modelling decisions: direction (store one direction, traverse either), cardinality (many relationships between the same pair are legal and often meaningful — separate *contracts* between two parties are separate `:HAS_CONTRACT` edges, not one), and whether an edge should really be a node (a *loan* between a bank and a customer is an edge with heavy state — promote it to a node when it has its own lifecycle, the "reification" pattern).

Reification, concretely: a *guarantee* between a borrower and a lender has its own state machine (issued → active → called → released). Model it as an edge and every state transition rewrites the edge; model it as a node (`(:Guarantee {status})` with `:GRANTED_BY` / `:GRANTED_TO` edges) and the state machine becomes node updates and queries — "all active guarantees where the bank is counterparty" is a node lookup, not an edge scan.
- **Labels/types.** Labels (`:Customer`, `:Account`) are the graph's schema: they enable index selection, constraint enforcement (uniqueness on `id`), and query planning. Use a small, stable label vocabulary; avoid label-per-subtype sprawl — use a `kind` property for fine-grained type distinctions and keep labels for structurally different node kinds.
- **Graph vs document modelling for connected data.** The decision is about *the query shape*: if the workload is "give me the record by key" (document/KV), or "give me everything about one aggregate" (document), the connectedness is incidental and a document model is simpler and faster. If the workload is *traversal* — "how is X connected to Y", "find rings/communities/links" — the graph model wins because each hop is an index-free adjacency lookup, not a join. See §8.2 for the decision tree.

### 5.3 Modelling for Graph Queries (Traversal-Friendly Design)

Graph modelling optimizes the **traversal shape**: you choose which relationships exist (and their direction), what indexes anchor the start node (`:Customer(id)` unique constraint), and how deep the typical traversal goes. The classic guidance — "the graph is only as good as its starting-point indexes and its relationship granularity" — means: index every node label's natural keys; model relationships at the *business* grain (a `:DEBITED_FROM` per transaction is a billion-edge pattern; model `:TRANSFERRED` between accounts and keep the transaction as a node attached to both); and prefer depth-bounded traversals (3–4 hops) which stay millisecond-cheap.

> **Cross-reference:** the graph family gets its full deep-dive in [neo4j_graph_database_guide.md](neo4j_graph_database_guide.md) — property graph vs RDF, Cypher modelling, indexing, and graph data science. The fraud/link-analysis application is in [../banking/financial_fraud_detection_at_scale_guide.md](../banking/financial_fraud_detection_at_scale_guide.md).

---

## 6. Cross-Cutting NoSQL Modelling Patterns

### 6.1 Denormalization: The Golden Rule

**Denormalize for the read patterns — duplication is intentional.** Every NoSQL family (except graph, which denormalizes into relationships) works by copying data into the shape each query wants. The golden rule: *duplicate the read-mostly, rarely-changing fields; never duplicate the write-hot, volatile fields.* Concretely: customer name/segment/tier is duplicated into every order, transaction, and account document (it almost never changes); customer *balance* is never duplicated (it changes every transaction).

**Consistency handling.** Duplication creates drift; you must decide how copies stay consistent:

- **Dual-write** — the application writes the primary record and every denormalized copy in the same request. Simple, but non-atomic: a crash between writes leaves copies stale. Acceptable when the copies are soft (display fields) or when the write path is a single service that owns both.
- **Event-driven sync** — the primary write publishes an event (via the **outbox pattern**, §2.6) and projection consumers rebuild the denormalized copies (see CQRS below). Eventual consistency by construction; the copies lag the primary by the processing latency.

**Eventual consistency** — the model *must tolerate lag*. A duplicated field is, by definition, momentarily inconsistent; the design question is not "how do we make it consistent" but "how stale can each copy be, and what happens when a user reads the stale copy". Read models that feed dashboards, analytics, and customer-360 UIs tolerate seconds of lag; a balance shown on a payment screen does not — so the payment screen reads the source of truth.

### 6.2 CQRS: Command/Query Separation

**CQRS** splits the data model into a **write model** (the authoritative, often normalized/aggregate-shaped store that accepts commands and enforces invariants) and **read models** (denormalized, read-optimized projections built from the write model's events). CQRS is the *architectural* answer to the NoSQL modelling tension: one shape cannot serve both the write path (atomicity, invariants) and every read path (shape-per-query), so you model *two* — and the read model becomes a set of **materialized views** maintained by event handlers.

NoSQL is the natural read-model technology: the write model stays in the relational core (ledger truth), and projections land in document/KV/wide-column stores shaped per query. This is precisely the pattern described for the OBMA data model in [../banking/oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md) (relational core + CQRS read projections) and the canonical "relational core + NoSQL edges" banking architecture (§10.3).

### 6.3 Aggregate Design: The Consistency Boundary

The **aggregate** (from Domain-Driven Design) is the unit of consistency and atomicity in the write path: a cluster of objects treated as one unit for invariants and transactions. In NoSQL, **the aggregate = the storage unit = the atomic unit**: the document, the key-value pair, the single partition's row set. The design trade-off is the heart of NoSQL modelling:

- **Bigger aggregates** → fewer transactions, fewer distributed operations, better read performance (one read returns the whole business object) — *but* higher write contention (every change to any part rewrites/locks the whole) and larger documents (16 MB limits, partition sizing).
- **Smaller aggregates** → independent updates, less contention, smaller units — *but* more cross-aggregate operations, N+1 reads, and application-level joins.

The modelling rule: **size the aggregate to the transactional invariant**, not to the entity. If a debit and its authorization must commit together, they are one aggregate (one document, one partition). If two records are merely *related*, they are separate aggregates joined by reference. Banking examples: an *order* with its line items is one aggregate; an *account* and its *transactions* are separate aggregates (transactions are unbounded); a *payment* and its *outbox event* are one aggregate (that's the outbox pattern).

| Aggregate sizing | Consequence | Banking example |
|---|---|---|
| Too big (everything in one document) | One read serves all; but every child update rewrites the whole, write contention rises, document/partition limits loom | Embedding *all* of a customer's transactions in the customer document |
| Too small (every record its own unit) | Independent updates, but N+1 reads and application joins | Storing an order and its line items as separate documents with no transaction |
| Right-sized (the invariant boundary) | Atomic where it matters, independent where it doesn't | Order + line items + outbox event in one document; transactions referenced (§11.1) |

### 6.4 Time-Series Modelling

Time-series data (market data, transaction flows, sensor/ATM telemetry) is the most common NoSQL workload after caching, and it has a standard modelling playbook:

- **Bucketing (partition per time period).** Never let a partition/document grow forever: key by `(entity, time_bucket)` — day, hour, or minute depending on write rate — so every partition is bounded (§4.5, §2.6).
- **Downsampling / rollups.** Keep raw data at high resolution for a short window; compute and store aggregates (1-min, 1-hour, daily OHLC/min/max/count) in separate tables/collections with longer retention. Rollups are maintained by stream processors (see [event_stream_processing_guide.md](event_stream_processing_guide.md)) or scheduled jobs.
- **Retention and TTL.** Old data is deleted by TTL (Redis, DynamoDB, MongoDB, Cassandra `TTL`, Bigtable cell TTLs) or by dropping whole time-bucketed partitions (Cassandra `DROP PARTITION`-via-`DELETE`, HBase region splits). Modelling decision: which granularity survives how long (raw 90 days, hourly 2 years, daily 7 years for banking audit).
- **Time-series databases vs NoSQL time-series modelling.** Purpose-built TSDBs — InfluxDB (columnar + inverted index), TimescaleDB (Postgres with hypertables), Prometheus (pull model, metric labels) — compress, downsample, and query time series far better than general NoSQL stores, at the cost of a narrower data model. The decision: *metric/telemetry* data → TSDB; *business event logs that happen to be time-ordered* (transactions, payments, audit) → wide-column/document with the patterns above, because the data is queried by *business key first, time second*.

| Decision | Pattern | Typical retention |
|---|---|---|
| Raw data at full resolution | Time-bucketed partitions/documents (§4.5, §2.6) | 90 days–1 year |
| Rollups (1-min → hourly → daily) | Downsampled tables/collections, stream-maintained | 1–7 years |
| Aggregates for dashboards | Counter/materialized tables (§4.5) | as long as the dashboard needs |
| Deletion | TTL (document/KV), time-bucketed partition deletes (wide-column) | per regulatory schedule |
| Metrics/telemetry at scale | Purpose-built TSDB | per monitoring policy |

### 6.5 Event-Sourced Models

**Event sourcing** stores state *as* its history: the event store is the source of truth (an append-only log of facts), and current state is a projection of replaying events. NoSQL modelling of event stores:

- Append-only collections/tables with an immutable event document/row per fact: `{eventId, aggregateId, type, payload, timestamp, sequence}`.
- The **aggregate key** (partition key) must group all events of one entity (Cassandra `PRIMARY KEY ((aggregate_id), seq)`, document `_id = aggregateId` with events as a bucketed series) so replay is one partition read.
- Projections (CQRS read models, §6.2) are materialized views over the event stream — the same pattern as outbox consumers.
- Event stores are also how **durable workflows** keep checkpoint state — see [durable_ai_agent_workflows_guide.md](durable_ai_agent_workflows_guide.md) and [event_stream_processing_guide.md](event_stream_processing_guide.md).

---

## 7. Modelling for Specific Systems

### 7.1 Platform-Specific Modelling Notes

**MongoDB** — document modelling per §2; the 16 MB ceiling, single-document atomicity (multi-doc transactions available but costly), schema validation (`$jsonSchema` — the document store's way to *choose* schema-on-write where regulation demands it), and **change streams** (the CDC mechanism that feeds outbox consumers, projections, and analytics — the event-driven sync of §6.1).

**DynamoDB** — single-table design (§3.5), partition/sort keys as the model, GSIs for every off-key access pattern, hot-partition avoidance, 400 KB item cap, on-demand vs provisioned capacity.

**Cassandra** — table-per-query (§4.2), partition sizing (keep partitions ≤ ~10–50 MB; oversized partitions degrade reads and repairs), and **tombstones**: deletes write tombstones that must outlive the data's TTL/gc_grace; high tombstone density makes reads scan dead cells (the classic "tombstone storm" after mass deletes — mitigate with TTLs, time-bucketed deletes, and compaction tuning).

**Redis** — structure choice is the model (§3.4); TTL everywhere; memory budgeting (each structure has a different memory profile; HASH/ZSET with small members use far less than many STRINGs); persistence (RDB/AOF) affects the modelling of *durable* data — don't store the only copy of critical data in Redis.

**Cosmos DB** — partition keys and **request units (RUs)**: every operation costs RUs (read = 1 RU per 1 KB, writes more), and RU consumption is the cost model. Modelling rules: choose the partition key so queries are single-partition (cross-partition queries cost more RUs and are slower); keep documents under 2 MB (max item size); design for the RU budget per operation (a 10 RU read budget rules out per-request aggregation pipelines).

**Firestore** — document model with *collections of documents* and composite indexes; the modelling unit is the collection + auto-indexed fields; rules: denormalize, keep documents small, beware of write hotspots on monotonically increasing IDs (distribute with sharded IDs), and design around its strong-consistency single-region vs multi-region trade-offs. Subcollections give hierarchical organization without nesting into one document.

**ScyllaDB** — Cassandra-compatible (same CQL, same modelling rules §4); per-node shard-per-core architecture makes it faster per node; modelling-wise identical to Cassandra, with the same partition-sizing and tombstone discipline.

**HBase** — row-key modelling (§4.7): key order = scan order, salt/reverse keys for write distribution, column families for storage grouping, versioned cells for "latest N" patterns.

### 7.2 Platform-Specific Modelling Pitfalls

| Platform | Pitfall | Mitigation |
|---|---|---|
| MongoDB | Unbounded embedded arrays (16 MB blow-up, document rewrite on every child update) | Reference or bucket (§2.3, §2.6) |
| MongoDB | Multi-document transactions on sharded clusters as the default consistency mechanism | Size aggregates to one document; use transactions only for genuine cross-aggregate invariants |
| DynamoDB | Hot partition throttling (one key saturates its partition) | Shard hot keys, spread by suffix, re-key (§3.5) |
| DynamoDB | Full-table scans sneaking into the model | Every access pattern must map to a (PK, SK) query or a GSI |
| Cassandra | Oversized partitions (multi-hundred-MB) | Time-bucket the partition key; monitor partition size (§4.5) |
| Cassandra | Tombstone storms after mass deletes | TTL instead of delete, time-bucketed deletions, compaction tuning (§7.1) |
| Cassandra | Write amplification from 5 denormalized tables | Cap the number of query shapes; use materialized views where safe |
| Redis | Storing the only copy of recoverable data (data loss on restart) | Redis for cache/hot data only; source of truth elsewhere; AOF for durability |
| Redis | One giant STRING for an object → partial updates become read-modify-write | HASH for field-updatable objects (§3.4) |
| Cosmos DB | Cross-partition queries eating RUs and latency | Partition key = the query's equality filter (§7.1) |
| Firestore | Write hotspot on sequential IDs | Sharded/random IDs; avoid monotonically increasing keys |
| HBase | Monotonic row keys concentrating writes on one region | Salt/reverse keys (§4.7) |
| Any | Assuming eventual consistency is "wrong" — then reading balances from a replica | Know each read's staleness tolerance; route money reads to the strong path (§10.2) |

---

## 8. The NoSQL Modelling Process and Methodology

### 8.1 Step-by-Step Methodology

**Step 1 — Requirement analysis.** Enumerate every query and access pattern: read/write ratio, keys and filters, sort orders, latency SLAs (p99), throughput (ops/sec), data volumes and growth, atomicity requirements, consistency requirements (how stale can a read be), retention. This step produces the *access pattern catalogue* — the contract every later step serves. In banking this is also where SLA/regulatory constraints (audit retention, residency) enter.

**Step 2 — Conceptual model.** At the *business* level, ignoring the store: entities, relationships, cardinalities (the Silverston-style universals from [data_model_resource_book_guide.md](data_model_resource_book_guide.md) apply unchanged — PARTY, AGREEMENT, TRANSACTION, etc.). This is the one step NoSQL shares with relational design; it gives you the domain vocabulary, not the storage shape.

**Step 3 — Logical model: family choice.** Decide document vs key-value vs wide-column vs graph vs (relational/TSDB) using the selection criteria (§8.2). The output is *"this workload is a document workload"*, not a schema.

**Step 4 — Physical model.** Per family: document/aggregate boundaries, embed/reference decisions, key design (KV: namespaces and composite keys; DynamoDB: PK/SK + GSIs; Cassandra: partition key + clustering + table-per-query; HBase: row keys), indexes, partitions/buckets, TTLs, and the write path for denormalized copies (dual-write vs outbox/projections).

**Step 5 — Validation.** (a) *Query test* — walk every access pattern in the catalogue against the physical model: does it hit one unit (document/key/partition)? Is there an index? (b) *Load test* — production-like volumes: p99 latency, partition sizes, hotspot detection, tombstone/TTL behaviour. (c) *Review* — schema-evolution plan (§9), consistency analysis, and a duplication audit (every denormalized copy has an owner and a refresh path).

The load test deserves specific parameters, because NoSQL performance is shape-dependent: test at *projected* data volume, not today's — an embedded-array document that is fine at 10 KB is broken at 12 MB. For wide-column: measure partition size distribution (the p99 partition must stay under the sizing budget), write latency under a *burst* (fixings, month-end), and read latency at p99 with concurrent readers. For key-value: measure memory (Redis) and eviction rates, and the cache-hit ratio at peak. For document: measure document size growth over the retention window and index size (indexes can exceed the data they index in document stores). A model that passes the query test but fails the load test is re-worked at step 4 — which is why validation is part of the modelling loop, not a gate at the end.

### 8.2 Data Model Selection (When to Use Which Family)

```
                        ┌─ Does the workload traverse relationships
                        │  (links, paths, rings, "how connected")?
                        │        YES → GRAPH (Neo4j, Neptune, ArangoDB)
                        ▼
          Is the query "give me record by key, ultra-fast"
          with a simple value (cache, session, rate, counter)?
                        YES → KEY-VALUE (Redis, Memcached, DynamoDB)
                        ▼
          Is the data semi-structured records read/written
          as wholes, with a flexible schema and rich queries?
                        YES → DOCUMENT (MongoDB, Couchbase, DynamoDB, Cosmos)
                        ▼
          Is the workload massive writes of event/row data,
          queried by key + ordered range, with no joins?
                        YES → WIDE-COLUMN (Cassandra, ScyllaDB, HBase, Bigtable)
                        ▼
          Multi-entity ACID transactions, ad-hoc SQL, complex
          joins, regulatory SoR? → RELATIONAL (see data_model_resource_book_guide.md)
          Pure metrics/telemetry? → TIME-SERIES DB (InfluxDB, TimescaleDB, Prometheus)
```

Selection criteria, in practice:

| Criterion | Favours |
|---|---|
| Connected/traversal queries (fraud link analysis, entitlements) | Graph |
| Read-heavy simple lookups, sub-millisecond, high QPS | Key-value (often as a cache layer) |
| Complex queries over semi-structured, schema-flexible records | Document |
| Massive write scale, query-by-key with ordered ranges, no joins, append-heavy | Wide-column |
| ACID multi-entity transactions, arbitrary joins, strong regulatory SoR | Relational |
| Metric/telemetry time series | Time-series DB |

### 8.3 The Modelling Checklist

Before a NoSQL model ships, run the checklist:

- **Query coverage** — every access pattern in the catalogue maps to one unit read (document/key/partition) + an index; no scans.
- **Denormalization completeness** — every read shape has its own data shape; read-mostly fields are duplicated where needed; every duplicate has an owner and a sync path (dual-write or event-driven).
- **Consistency requirements** — staleness is quantified per read; money/authoritative reads route to the strong path; the model tolerates the lag of event-driven copies.
- **Partition/key design** — partition keys are high-cardinality and even; clustering/sort keys match the hot sort; keys are never changed after go-live without a rebuild plan (§9.3).
- **Hot spots** — no hot keys/partitions in the access-pattern analysis; bursty patterns (month-end, fixings) have a plan (on-demand capacity, key sharding).
- **Growth** — partition/document size is bounded (bucketing); write rate per partition projected over the retention window.
- **TTL/retention** — every dataset has a retention policy; TTLs set; time-bucketed deletes for wide-column.
- **Schema evolution** — additive changes only at go-live; a version field and transform-on-read plan exist (§9).

---

## 9. Schema Evolution and Migration

### 9.1 Schema-on-Read Evolution

NoSQL's schema flexibility is a *runtime* property: the schema lives in the application's read logic, so evolution is a code-deploy, not a DDL. The discipline is the same as API versioning. See [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md) for the general schema-evolution treatment; the NoSQL specifics:

- **Document field additions (backward compatible).** New fields with defaults are safe: old documents lack the field, the reader supplies the default (`doc.riskScore ?? "LOW"`). The cardinal rule: **readers must tolerate missing fields** — never assume presence.
- **Data migration (backfill, dual-read).** When old documents must carry the new field (e.g., for an index to serve it), backfill in batches (cursor-based, throttled), or read-transform-and-rewrite lazily on access. **Dual-read** — write new format, read *both* formats during the transition — is the safe rollout for format changes.
- **Versioning.** A `schemaVersion` field makes evolution explicit: the reader branches on version and transforms old shapes in memory (transform-on-read). Old data is never rewritten for mere shape changes.

```js
// transform-on-read: one reader, two shapes
function readCustomer(doc) {
  if (doc.schemaVersion >= 2) return doc;                       // current shape
  return { ...doc, schemaVersion: 2,                           // v1 -> v2 transform
           kyc: { status: doc.kycStatus, level: "L1" },        // flattened -> nested
           legacy: true };                                     // tagged for backfill
}
```

The backfill (writing the v2 shape) then runs as a batch job whenever convenient — reads are correct from the moment the reader deploys.
- **Breaking changes (field renames).** Renaming a field is breaking — old documents have the old name. Patterns: write the new field and the old field during a transition window (dual-write at the field level), read either, then drop the old field after all readers move; or transform-on-read indefinitely for a tiny legacy population.

### 9.2 Wide-Column Schema Changes

- **Cassandra `ALTER TABLE`** supports adding columns (the new column is `null` for existing rows — fine with schema-on-read) and changing some table options; *changing or removing columns, or altering the primary key, is not supported* — that requires a new table + migration.
- **Adding a new query shape** in Cassandra means a *new table* (per the table-per-query rule) and a backfill of existing data into it — which is why the query catalogue (§8.1 step 1) must be complete *before* go-live.

### 9.3 Key Design Changes Are Irreversible

In KV and wide-column models the key is the physical layout: **changing the key design (partition key, row key, key format) is a full rebuild** — new keyspace/table, re-ingestion or backfill, dual-read during cutover, then drop. There is no in-place migration. The mitigation is design-time: get the key right in the access-pattern analysis, and version key *formats* (`v2:rate:USD:SGD`) rather than migrating in place.

### 9.4 Schema Evolution Patterns

| Pattern | When | Mechanics |
|---|---|---|
| **Additive** | New optional fields | Deploy readers first (tolerate absence), then writers, then optional backfill |
| **Transform-on-read** | Old shapes linger; no rewrite budget | `schemaVersion` branch; in-memory transform; old data untouched |
| **Dual-write migration** | Format/key change | Write new + old during transition; readers read new, fall back to old; backfill; then retire old |
| **Event-replay rebuild** | Projection/read-model change | Rebuild the CQRS read model from the event store (§6.2, §6.5) |

---

## 10. Banking Context: NoSQL in Financial Services

### 10.1 Use Cases

| Use case | Family | Why | Cross-reference |
|---|---|---|---|
| **Customer 360** | Document store | The customer aggregate (profile + KYC + relationships + preferences) read as one unit; flexible schema for segments; the canonical document model (§11.1) | [data_models_banking_insurance_guide.md](../banking/data_models_banking_insurance_guide.md) |
| **Transaction / event storage** | Wide-column / time-series | Massive append volume, query by account + time range, no joins, retention via TTL/buckets (§4.5, §11.2) | [event_stream_processing_guide.md](event_stream_processing_guide.md) |
| **Fraud detection** | Graph | Link analysis: mule rings, shared devices/phones, beneficiary networks; traversal is the query (§5) | [../banking/financial_fraud_detection_at_scale_guide.md](../banking/financial_fraud_detection_at_scale_guide.md) |
| **Reference data caching** | Key-value (Redis) | FX rates, limits, product parameters cached with TTL; sub-ms reads on the transaction path (§11.3) | [../banking/banking_limits_domain_guide.md](../banking/banking_limits_domain_guide.md) |
| **Session / consent stores** | Key-value | Session tokens and consent decisions keyed by subject, TTL'd (§3.4) | — |
| **Payment event stores** | Document / wide-column | Payment lifecycle events, iso20022 payloads as polymorphic documents (§2.4), query by payment ID | [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) |
| **Audit / ledger** | Append-only (wide-column / event store) | Immutable event log of who-did-what-when; replayable for forensics; never update in place (§6.5) | [core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) |

The pattern behind the table: **every NoSQL deployment in a bank is either a read-optimized projection of the core or a write-optimized log the core consumes** — rarely the source of truth itself. The customer 360 (§11.1) is a projection built from core events; the transaction history (§11.2) is a log the core appends to and the statement engine reads; the fraud graph is a derived analytical layer over transaction and entity data; the Redis tier is a cache in front of reference-data truth. Naming the *direction of data flow* for each store — who writes it, what feeds it, what reads it — is the first step of any banking NoSQL design, because it determines the consistency contract (§10.2).

### 10.2 Regulatory Considerations

- **Data governance — BCBS 239.** The risk-data-aggregation principles demand data lineage, accuracy, and completeness for risk reporting. A denormalized NoSQL copy is a *derived data product*: it needs documented lineage to its source, an owner, and validation — exactly the discipline of §6.1's sync paths. See [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md).
- **Consistency requirements — ACID vs eventual.** Financial data has a hard truth: **the ledger truth lives in the relational core** (see [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)) where ACID and regulatory SoR are non-negotiable. NoSQL serves the *read models, analytics, and experience layers* — the CQRS split (§6.2): authoritative writes go to the core; customer 360, reporting, and analytics read eventually-consistent projections. The rule: *any read that could be acted upon as a financial decision (balance for a payment, limit check for approval) must read the strong/authoritative path;* everything else may read the NoSQL projection with its documented lag.
- **Data residency.** Regulatory (MAS, GDPR, China DSL) constraints on where data lives apply to NoSQL clusters like any other store — multi-region replication and BYOC deployments must respect residency; see [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) and the MAS TRM discussion there.
- **Model risk (SR 11-7).** The NoSQL data model *is* a model in the SR 11-7 sense when it embodies business logic — a denormalized risk-aggregation projection, a fraud-scoring graph schema, a limit-check cache — and is subject to governance, validation, and change control like any model.
- **Operational resilience (MAS TRM).** NoSQL tiers must meet the same availability/recovery targets as the core: multi-AZ/multi-region replication, defined RPO/RTO for each NoSQL dataset, and tested failover — the platform notes in §7 (e.g., Cassandra's multi-DC topology, DynamoDB global tables) are resilience features you model *into* the deployment.
- **Audit and immutability.** Regulatory audit trails are the one place NoSQL's flexibility must be *disabled by design*: audit/ledger data is append-only, never updated in place, versioned (or event-sourced, §6.5), and retained per the regulatory schedule (§6.4). A document store used for audit should be written with immutable documents (new ID per version, `supersedes` link) and read-only roles for everything except the appender — schema validation (`$jsonSchema`) and database roles enforce what the application promises.

### 10.3 Bank NoSQL Adoption Patterns

The pragmatic banking pattern is **"relational core + NoSQL edges"**: the core banking system of record, general ledger, and regulatory reporting stay relational (or NewSQL); NoSQL wraps the edges where its properties pay:

- **Document store** for the customer-experience layer (customer 360, onboarding journeys, agreements in flexible shapes) — fed from the core by CDC/events (see [../banking/oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md) for the CQRS read-model split in a real core).
- **Redis** for hot reference data and session state on the transaction path.
- **Wide-column** for event/transaction history, audit trails, and risk data at scale.
- **Graph** for fraud/AML link analysis and entitlements.

Each edge has a clear owner, lineage, and consistency contract — and none of them ever becomes the ledger.

---

## 11. Worked Examples

### 11.1 Worked Document Model: Banking Customer 360 (MongoDB)

**Queries (the catalogue):**
1. Fetch the full customer profile by customer ID (single read, p99 < 50 ms) — the onboarding/KYC screen.
2. List the customer's accounts with balances (dashboard).
3. List the customer's cards (cards screen).
4. Count the customer's accounts/cards by type (dashboard tiles).
5. *Not* a query: the customer's full transaction history — that lives in the wide-column store (§11.2) and is referenced, not embedded.

**Design walkthrough.** The customer is the aggregate: profile + KYC + accounts + cards are read together on every customer screen. So the *bounded, read-together* parts embed; the *unbounded* part (transactions) references.

```
// customers collection — one document per customer (the aggregate)
{
  "_id": "CUST-4401",
  "schemaVersion": 3,
  "type": "CORPORATE",                       // polymorphic: retail docs have different fields
  "name": "Shurui Trading Pte Ltd",
  "segment": "CORP", "tier": "GOLD",
  "kyc": { "status": "APPROVED", "level": "L3", "reviewDue": "2027-01-15",
           "amlFlags": [] },
  "addresses": [ { "kind": "REGISTERED", "country": "SG" } ],      // 1:1 → embed
  "accounts": [                                // 1:many, bounded, read-together → embed
    { "accountId": "ACCT-88213", "ccy": "SGD", "kind": "CURRENT",
      "balance": 184250.00, "status": "ACTIVE" },
    { "accountId": "ACCT-88214", "ccy": "USD", "kind": "SAVINGS",
      "balance": 52000.00, "status": "ACTIVE" }
  ],
  "cards": [                                   // 1:many, bounded → embed
    { "cardId": "CARD-7712", "product": "CORP-DEBIT", "status": "ACTIVE", "masked": "4539 **** 2210" }
  ],
  "txnRefs": { "store": "TXN-HISTORY", "keys": ["ACCT-88213", "ACCT-88214"] }  // unbounded → reference
}
```

**Trade-offs considered:**
- *Embed vs reference for accounts:* embedded — bounded (a customer has a handful of accounts), read with the customer, and account *status/balance* must be near-consistent with the profile screen. If account counts could grow unbounded or accounts needed heavy independent updating, this would flip to references (IDs) + a separate `accounts` collection.
- *Why not embed transactions:* unbounded → reference, by the decision table (§2.3). The 16 MB limit makes it physically impossible anyway.
- *Balance freshness:* the balance here is a *projection* fed by events from the core (CQRS, §6.2); the authoritative balance lives in the core. The screen labels it "as of" — the model tolerates lag (documented).
- *Indexes:* `_id` (customer ID) serves query 1; no other index needed — every query is by customer ID against the aggregate. This is query-driven index design: the catalogue has no cross-customer queries.

### 11.2 Worked Wide-Column Model: Transaction History (Cassandra)

**Queries:**
1. "Recent transactions of account X, newest first" (p99 < 20 ms, the account screen).
2. "Transactions of account X between dates D1 and D2" (statement download).
3. "Daily totals per account for the last 30 days" (dashboard).

**Table design (one table per query shape):**

```sql
-- Query 1 & 2 share a shape (account, time range, newest first) → one table
CREATE TABLE txn_by_account (
  account_id  text,
  day         date,          -- partition bucket: one partition per account-day (bounded, §4.5)
  txn_time    timestamp,
  seq         int,           -- uniqueness within same millisecond
  amount      decimal,
  ccy         text,
  counterparty text,
  status      text,
  PRIMARY KEY ((account_id, day), txn_time, seq)
) WITH CLUSTERING ORDER BY (txn_time DESC, seq DESC);

-- Query 1:  SELECT * FROM txn_by_account WHERE account_id=? AND day=? ;        -- one partition
-- Query 2:  SELECT * FROM txn_by_account WHERE account_id=? AND day>=? AND day<=?; -- fan-out over days
-- Query 3 needs a different shape → its own table (or materialized view)
CREATE TABLE daily_totals_by_account (
  account_id text, day date, txn_count counter, total_amount counter,
  PRIMARY KEY ((account_id), day)
);
```

**Design decisions:** partition key = `(account_id, day)` — the query's equality filter *plus* a time bucket so no partition grows past one day of transactions (bounded growth, even for a 30-year account). Clustering = `txn_time DESC` so "newest first" is the physical order. The statement query fans out over ~31 partitions — acceptable, each read is still a single-partition scan. Query 3 is served by a *counter table* maintained in the write path — no aggregates in queries (§4.4). The application dual-writes (or a CDC consumer updates) the counters. Tombstones: statement deletion (regulatory retention expiry) is a *range delete* per day bucket, not per-row deletes — avoiding tombstone storms (§7.1).

### 11.3 Worked Key-Value Model: Rate Cache (Redis)

**Requirement:** FX spot rates for ~180 currency pairs; 40,000 reads/sec on the pricing path at p99 < 1 ms; rates refresh every 15 seconds from the market data feed; stale-but-bounded acceptable (max 60 s) on the pricing *display* path, strong on the execution path.

**Design:**

```
Key:      rate:USD:SGD          (namespace:entity:pair — composite, hierarchical)
Value:    HASH { bid: "1.3482", ask: "1.3492", mid: "1.3487", ts: "2026-08-01T10:00:15Z" }
TTL:      60 seconds            (twice the refresh interval — self-healing staleness bound)

Refresh:  HSET rate:USD:SGD bid ... ask ... (pipeline all 180 pairs in one round-trip)
Read:     HGETALL rate:USD:SGD  → 1 round-trip, ~µs
Hot key:  USD:SGD at fixings → shard the key if needed: rate:USD:SGD:0..63, read all, merge (§3.2)
```

**Decisions:** HASH not STRING — the pricing path reads bid/ask/mid separately and the feed updates fields independently (a STRING would force read-modify-write). TTL = 60 s makes staleness self-bounding: a dead feed means *expired* keys (cache miss → fall back to the last-known table in the relational reference-data store), never silently stale rates on the execution path. The execution path (order pricing) reads the rate *and* validates freshness (`ts` within 30 s) before use — the strong-read rule of §10.2.

---

## 12. The Future: 2026 and Beyond

### 12.1 Convergence: The Multi-Model Trend

The four families are converging into **multi-model databases** that store several paradigms behind one engine and one API: MongoDB (document + time-series + vector + graph-lite), ArangoDB (document + graph + KV), Cosmos DB (document + KV + wide-column + graph), and Oracle 23ai (relational + JSON + graph + vector). For the architect this *reduces* the family-selection stakes: the same cluster can serve a document aggregate and a graph traversal, and the modelling decision becomes "which paradigm does this query shape use" rather than "which vendor do we buy".

### 12.2 Postgres-ification: "Just Use Postgres"

Meanwhile the relational side absorbed NoSQL's best ideas: **JSONB** (schema-flexible documents inside a relational table, indexed with GIN, queried with SQL), pgvector (§in [ai_llm/vector_databases_guide.md](ai_llm/vector_databases_guide.md)), native partitioning, and increasingly graph (SQL/PGQ in Oracle 23ai, Apache AGE for Postgres). The "just use Postgres" trend is real for the *long tail* of workloads that were never truly NoSQL — moderate scale, semi-structured data, one team, and a preference for one engine. The trend does *not* retire the genuine NoSQL cases: sub-millisecond KV caching (Redis), massive write scale with bounded partitions (Cassandra-class), and traversal-native graph (Neo4j-class) remain structural advantages.

### 12.3 AI and Data Modelling

- **AI-assisted schema design.** LLM-assisted modelling tools generate candidate aggregates, keys, and partition designs from a natural-language access-pattern catalogue; the architect's job shifts to validating the query coverage and consistency analysis (§8.3) — the checklist becomes the prompt.
- **LLM-generated models** risk elegance over correctness: an LLM will happily produce a normalized relational model when asked for NoSQL, or invent access patterns that don't exist. The query-first discipline (§1.3) is exactly the guardrail AI assistance needs.
- **Schema-on-read maturity** — with AI agents and LLMs reading data directly, self-describing schemas (embedded type/version fields, JSON Schema) become the machine interface; schema evolution (§9) becomes an API-contract discipline for models as much as for services.

### 12.4 NoSQL in the AI Era

The AI stack made NoSQL *bigger*, not smaller: **vector stores** (the AI-era document/KV hybrid — embeddings as first-class data) are documented in [ai_llm/vector_databases_guide.md](ai_llm/vector_databases_guide.md), **feature stores** (the AI-era read model — denormalized, query-shaped, versioned features for online/offline serving) are the CQRS pattern applied to ML (see [feature_store_guide.md](feature_store_guide.md)), and RAG pipelines reuse every pattern in this guide: the retrieval index is a denormalized projection, the chunk cache is a KV store with TTL, the knowledge graph is a graph model (§5). See also [ai_llm/rag_vs_long_context_llms_guide.md](ai_llm/rag_vs_long_context_llms_guide.md) and [ai_llm/beyond_rag_guide.md](ai_llm/beyond_rag_guide.md).

### 12.5 Trends Summary

| Trend | What it means for modelling |
|---|---|
| Multi-model convergence | One engine, paradigm-per-query-shape; family choice becomes intra-engine |
| Postgres-ification | JSONB/pgvector/partitioning absorb the easy cases; "just use Postgres" for the long tail |
| AI-assisted schema design | Natural-language access-pattern catalogues → candidate models; the checklist becomes the prompt |
| Schema-on-read maturity | Self-describing documents as the machine interface; schema evolution as API discipline |
| NoSQL in the AI era | Vector stores, feature stores, knowledge graphs — the patterns of this guide *are* the AI data layer |

The through-line of the future is the through-line of this guide: **model for the queries** — whether the store is a document database, a sorted map, or a Postgres table with a JSONB column, the shape of the data is a projection of the workload, and the architect who starts from the access patterns ends up with the right model.

### 12.6 Practical Implications for Architects

- **Re-evaluate the family choice annually.** Multi-model and Postgres-ification mean the "right" store for a workload shifts; a model designed in 2023 for a document store may be better served in 2026 by JSONB + partitioning in the relational core — the modelling *discipline* (§1.3) carries over unchanged even when the engine changes.
- **Model the AI layer with the same rules.** Vector indexes, feature stores, and knowledge graphs are NoSQL models with new query shapes (similarity, feature-lookup, traversal) — the access-pattern catalogue and the one-unit-read test still apply.
- **Keep the schema-on-read contract explicit.** As LLMs and AI agents become readers of data, self-describing documents (`type`, `schemaVersion`) and published JSON Schemas become the interface — write them down, version them, and treat them as API contracts (§9.1).
- **The query catalogue is the deliverable.** In every engagement, the access-pattern catalogue (§8.1 step 1) — not the diagram — is what survives: it is the input to AI-assisted design, the validation checklist, and the migration plan when the engine changes.

---

## 13. Glossary

**NoSQL.** "Not only SQL": non-relational databases optimized for horizontal scale, flexible schema, and specific query shapes, trading general-purpose SQL/joins/ACID for performance and simplicity in their niche.

**Document store.** A NoSQL family storing self-contained JSON/BSON documents as aggregates; query by document key and field indexes (MongoDB, Couchbase, DynamoDB, Firestore, Cosmos DB).

**Key-value store.** A NoSQL family storing values addressed by a single key; the simplest and fastest access model (Redis, Memcached, etcd, RocksDB, DynamoDB).

**Wide-column store.** A NoSQL family storing rows in partitions ordered by clustering columns; massive write scale, query by partition key + ordered range (Cassandra, HBase, ScyllaDB, Bigtable).

**Graph database.** A NoSQL family storing nodes, typed relationships, and properties; optimized for traversal queries (Neo4j, Neptune, ArangoDB, JanusGraph).

**Aggregate.** A cluster of data treated as one unit for atomicity and reads (DDD); in NoSQL, the document/partition — the storage unit *is* the consistency boundary (§6.3).

**Embedding.** Storing related data inside the parent document/aggregate so it is read and committed together (§2.2).

**Referencing.** Storing only the ID of related data, resolved by the application or a second query (§2.2).

**Denormalization.** Deliberately duplicating data into the shape each query wants; the golden rule of NoSQL modelling (§6.1).

**Partition key.** The hash key deciding physical placement of a row/item; the query's equality filter (Cassandra, DynamoDB) (§4.3, §3.5).

**Clustering column.** The non-hashed part of a wide-column primary key defining sort order within the partition (§4.3).

**Sort key.** DynamoDB's term for the range component of the composite primary key, defining order within a partition (§3.5).

**GSI (Global Secondary Index).** A DynamoDB index re-projecting the table under a different partition/sort key, maintained asynchronously (§3.5).

**LSI (Local Secondary Index).** A DynamoDB index offering an alternative sort key within the same partition key; fixed at creation (§3.5).

**Single-table design.** The DynamoDB practice of storing many item types in one table, disambiguated by item-type prefixes in the keys (§3.5).

**Item type.** The attribute distinguishing item kinds in a single-table DynamoDB design (§3.5).

**Hot key / hot partition.** A key/partition receiving disproportionate traffic, bottlenecking its node while peers idle (§3.2, §3.5, §4.7).

**TTL (Time-to-live).** Per-record expiry; the self-healing staleness and retention mechanism of KV/document/wide-column stores (§3.2, §6.4).

**Tombstone.** A delete marker in wide-column stores that must outlive the deleted data; excessive tombstones degrade reads (§7.1).

**Materialized view.** A maintained, pre-computed projection of data in a query-friendly shape; the implementation of CQRS read models (§6.2).

**CQRS (Command Query Responsibility Segregation).** Splitting the write model (authoritative, invariant-enforcing) from read models (denormalized projections) (§6.2).

**Event sourcing.** Storing state as an append-only log of events; current state is a projection of replay (§6.5).

**Outbox.** Writing events to a table/collection in the same transaction as the business write, then publishing them; atomic state + event delivery without distributed transactions (§2.6, §6.1).

**Bucket pattern.** Grouping time-ordered events into fixed-window documents/partitions to bound size and enable rollups (§2.6, §4.5).

**Time-series.** Data indexed primarily by time; modelled via bucketing, downsampling, retention, and TTL (§6.4).

**Schema-on-read.** The schema is applied by the application when reading; writers may store any shape (§1.2, §9.1).

**Schema-on-write.** The schema is validated/enforced at write time (relational DDL, MongoDB `$jsonSchema` validation) (§1.2, §7.1).

**Multi-model.** A database supporting several data paradigms (document, graph, KV, relational, vector) behind one engine (ArangoDB, Cosmos DB, Oracle 23ai) (§12.1).

**JSONB.** Postgres's binary JSON type enabling schema-flexible documents inside relational tables with GIN indexes (§12.2).

**UDT (User-Defined Type).** A named composite type in Cassandra used as a column type; the wide-column equivalent of embedding (§4.5).

**BSON.** Binary JSON, MongoDB's storage format, with typed values including decimal128 for exact money (§2.5).

**RU (Request Unit).** Cosmos DB's cost unit; every operation consumes RUs, making RU budgeting part of the model (§7.1).

---

*Related guides: [data_model_resource_book_guide.md](data_model_resource_book_guide.md) (relational universals — the conceptual-model counterpart) · [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md) (schema evolution & drift) · [event_stream_processing_guide.md](event_stream_processing_guide.md) (eventing, outbox consumers, time-series feeds) · [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (event streaming platforms) · [neo4j_graph_database_guide.md](neo4j_graph_database_guide.md) (graph deep-dive) · [../technology/ai_llm/vector_databases_guide.md](../technology/ai_llm/vector_databases_guide.md) (vector stores, pgvector) · [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (relational SoR) · [../banking/oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md) (CQRS read models in a core) · [../banking/data_models_banking_insurance_guide.md](../banking/data_models_banking_insurance_guide.md) (canonical banking models) · [../banking/financial_fraud_detection_at_scale_guide.md](../banking/financial_fraud_detection_at_scale_guide.md) (graph link analysis) · [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) (BCBS 239, MAS TRM) · [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) (payment event stores) · [../banking/banking_limits_domain_guide.md](../banking/banking_limits_domain_guide.md) (Redis reference data)*
