# Graph Database Concepts & Neo4j — A Comprehensive Guide

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Crédit Agricole CIB  
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Audience:** Engineers and architects building production graph applications in financial services

---

## 1. What Are Graph Databases?

### 1.1 Definition and Core Concepts

Graph databases are NoSQL databases that use **nodes** (vertices), **relationships** (edges), and **properties** (key-value attributes) to represent and store data. Unlike relational databases where relationships are inferred through foreign keys and JOIN operations, graph databases store relationships as first-class citizens — traversal is pointer-chasing, not table joining.

- **Nodes** represent entities (person, account, transaction, document).
- **Relationships** represent connections (OWNS, SENT_TO, IDENTIFIES). Every relationship has direction, a single type, and optional properties.
- **Properties** are key-value pairs on both nodes and relationships.
- **Labels** group nodes into categories (`:Customer`, `:Account`, `:Transaction`).

### 1.2 Why Graph Databases Matter

Graph databases excel at **connected data** and **relationship-heavy queries** that are expensive in relational databases:

| Query Depth | Relational (JOINs) | Graph (Traversal) |
|-------------|-------------------|-------------------|
| 1 hop | 1 JOIN | 1 pointer dereference |
| 3 hops | Exponential JOIN cost | 3 pointer dereferences |
| N hops | N self-JOINs or recursive CTEs | Linear traversal cost |

A variable-depth relationship query in SQL requires recursive CTEs or N self-JOINs, each scanning the join table. In a graph database, the same query follows adjacency pointers — cost grows linearly with path length, not exponentially with join fan-out.

### 1.3 Property Graph Model vs. RDF/SPARQL

Neo4j uses the **labeled property graph (LPG) model**:

| Aspect | Property Graph (Neo4j) | RDF / SPARQL |
|--------|----------------------|--------------|
| **Data unit** | Nodes + relationships with properties | Triples (subject → predicate → object) |
| **Schema** | Schema-optional, labels for grouping | Ontology-driven, OWL/RDFS |
| **Query language** | Cypher (declarative) | SPARQL (W3C standard) |
| **Relationships** | Named, directed, can have properties | Named predicates only (no edge properties) |
| **Performance** | Optimized for graph traversal | Optimized for triple pattern matching |
| **ACID** | Fully ACID | Varies by implementation |

The LPG model is preferred for **transactional graph workloads** where relationship properties (amount, timestamp, status) and traversal performance are critical.

### 1.4 ACID Compliance

Neo4j is fully **ACID compliant**:

- **Atomicity:** Transactions are all-or-nothing — partial failures roll back fully.
- **Consistency:** Constraints (unique, node key, existence) are enforced at commit time.
- **Isolation:** Read-committed by default; dirty reads, non-repeatable reads, and phantom reads are prevented.
- **Durability:** Write-ahead logging ensures committed transactions survive crashes. `dbms.tx_log.rotation.retention_policy` controls log retention.

This makes Neo4j suitable for financial services workloads where data integrity is non-negotiable.

### 1.5 Key Use Cases

| Use Case | Description | Why Graph |
|----------|-------------|-----------|
| **Fraud Detection** | Real-time scoring, synthetic identity detection | Multi-hop pattern matching across accounts, devices, IPs |
| **Recommendation Engines** | Product/content/connection recommendations | Collaborative + content-based filtering via traversal |
| **Knowledge Graphs** | Entity resolution, semantic search | Heterogeneous entities with rich semantic relationships |
| **Network Analysis** | Social, telecom, IT infrastructure | Deep traversal, centrality, community detection |
| **IAM** | Role hierarchies, permission propagation | Hierarchical traversal, recursive access checks |
| **Supply Chain** | Bill of materials, supplier networks | Multi-level dependency traversal, impact analysis |
| **Master Data Mgmt** | Golden record resolution, data lineage | Entity deduplication, cross-system relationship tracing |
| **AML Transaction Monitoring** | SAR detection, transaction chain analysis, UBO | Multi-hop flow analysis, circular flow detection, pattern matching |

---

## 2. Neo4j Data Modeling Principles

### 2.1 Nodes and Labels

**Nodes** represent entities. **Labels** group nodes into categories; a node can have multiple labels.

```cypher
// Single label
CREATE (:Customer {id: 'CUST001', name: 'Acme Corp', risk_rating: 'HIGH'})
// Multiple labels
CREATE (:Customer:VipCustomer:Wholesale {id: 'CUST002'})
```

**Conventions:** PascalCase for labels (`:Customer` not `:customer`). Every node should have at least one label — unlabeled nodes force `AllNodesScan`. Labels are automatically indexed (lookup index).

**When to use multiple labels:** Taxonomy membership (`:Vehicle:Car:Sedan`), multiple roles (`:Person:Employee:Manager`), subgraph access control. **When not to:** If the combination is rare (<1%), use a property instead.

### 2.2 Relationships and Types

Relationships have **direction**, a **single type**, and optional **properties**.

```cypher
CREATE (c:Customer)-[:OWNS {since: datetime('2024-01-01'), ownership_pct: 100}]->(a:Account)
```

**Conventions:** UPPER_SNAKE_CASE (`:OWNS` not `:Owns`). Types are semantically verbs or prepositions. A relationship has exactly ONE type — use separate types for different semantics.

**Direction matters:** Directed traversals (`(a)-[:OWNS]->(b)`) are faster than undirected (`(a)-[:OWNS]-(b)`) because undirected must check both directions, doubling expansion work. Always specify direction when known.

### 2.3 Properties

| Type | Description | Example |
|------|-------------|---------|
| `String` | UTF-8 text | `'Singapore'` |
| `Integer` | 64-bit signed | `42` |
| `Float` | 64-bit IEEE 754 | `3.14159` |
| `Boolean` | true/false | `true` |
| `ByteArray` | Binary (max 2GB) | `$binaryData` |
| `Date` | ISO 8601 date | `date('2024-06-15')` |
| `DateTime` | Date+time+timezone | `datetime('2024-06-15T10:30:00+08:00')` |
| `LocalDateTime` | Date+time, no tz | `localdatetime('2024-06-15T10:30:00')` |
| `LocalTime` | Time only | `localtime('10:30:00')` |
| `Duration` | ISO 8601 duration | `duration('P1Y2M3D')` |
| `Point` | 2D/3D spatial point | `point({latitude:1.35, longitude:103.82})` |
| `List` | Homogeneous list | `['a','b','c']` |

**Conventions:** camelCase keys (`firstName` not `first_name`). Consistent across labels and relationship types.

### 2.4 Common Modeling Patterns

**Adjacency List** — direct connections:
```cypher
(:Account)-[:SENT_TO]->(:Account), (:Person)-[:KNOWS]->(:Person)
```

**Aggregation** — counting across relationships:
```cypher
MATCH (a:Account)-[:PERFORMS]->(t:Transaction)
WHERE t.date >= date('2024-01-01') AND t.date < date('2024-01-02')
RETURN a.id, count(t) AS tx_count, sum(t.amount) AS total_amount
```

**Time Series** — event timelines:
```cypher
(:Event {timestamp: datetime('2024-01-15T10:00:00')})-[:NEXT]->(:Event {timestamp: datetime('2024-01-15T10:01:00')})
```

**Hierarchies** — parent-child, org structures:
```cypher
(:Employee)-[:REPORTS_TO]->(:Employee)
MATCH (e:Employee {id: 'EMP001'})-[:REPORTS_TO*0..5]->(manager)
RETURN manager.name, length(path) AS depth
```

**Geospatial** — location-based:
```cypher
(:Branch {location: point({latitude: 1.35, longitude: 103.82})})
MATCH (b:Branch) WHERE point.distance(b.location, point({latitude: 1.35, longitude: 103.82})) < 5000
```

**Recommendation** — user-item interactions:
```cypher
MATCH (p:Product {id: $pid})<-[:PURCHASED]-(u:User)-[:PURCHASED]->(rec:Product)
WHERE rec.id <> $pid RETURN rec.id, count(*) AS freq ORDER BY freq DESC LIMIT 10
```

### 2.5 Anti-Patterns to Avoid

**1. "Nodelification" of properties** — Don't create a node for a value that should be a property. Rule: if the "entity" has no relationships of its own and only one node references it, use a property.

**2. Over-relationship** — Too many semantically redundant types (`:KNOWS`, `:FRIEND_OF`, `:ACQUAINTED_WITH`). Use one type + property qualifier unless semantics fundamentally change traversal.

**3. Over-labeling** — More than 5 labels per node suggests the taxonomy should be properties or separate nodes. 2–3 labels per node is typical.

**4. Disconnected subgraphs** — Isolated nodes with no relationships are unreachable via traversal. Every node should participate in at least one relationship.

**5. Relationships for everything** — Use a relationship when traversal across the connection is meaningful. Use a property when you only need to filter or return the value.

**6. Fan pattern (supernode)** — A single node with millions of relationships. See section 2.6 for mitigation.

### 2.6 Dense Nodes — Strategies

A **supernode** has disproportionately high degree (e.g., a central bank clearing account). It causes slow expansions, memory pressure, and skewed cardinality estimates.

**Strategy 1: Skip/Limit Pagination**
```cypher
MATCH (hub:Account {id: $hubId})-[r:SENT_TO]->(target)
RETURN target, r.amount ORDER BY r.timestamp DESC SKIP $skip LIMIT $limit
```

**Strategy 2: PageRank Sampling** — Use GDS algorithms instead of exhaustive traversal:
```cypher
CALL gds.pageRank.stream('myGraph') YIELD nodeId, score
RETURN gds.util.asNode(nodeId).id AS node_id ORDER BY score DESC LIMIT 100
```

**Strategy 3: Relationship Type Partitioning** — Each relationship type is a separate adjacency list:
```cypher
(:Account)-[:CREDIT_TRANSFER]->(:Account)
(:Account)-[:DEBIT_TRANSFER]->(:Account)
```

**Strategy 4: Splitting Supernodes** — Decompose into physically distinct partition nodes:
```cypher
(:Account)-[:HAS_BATCH]->(:TxBatch {date: date('2024-01-15')})-[:CONTAINS]->(:Transaction)
```

**Rule of thumb:** >100K relationships → evaluate partitioning. >1M → partitioning essential.

---

## 3. Neo4j Indexing

### 3.1 Schema Index Types

| Index Type | Best For | Example |
|-----------|----------|---------|
| **B-tree** (default) | Equality, range, ORDER BY, IS NOT NULL | `CREATE INDEX FOR (c:Customer) ON (c.id)` |
| **Range** | Numeric/string range queries (optimized) | `CREATE RANGE INDEX FOR (t:Transaction) ON (t.amount)` |
| **Text** (Lucene) | CONTAINS, ENDS WITH, full-text search | `CREATE TEXT INDEX FOR (c:Customer) ON (c.name)` |
| **Point** | Spatial distance/bbox queries | `CREATE POINT INDEX FOR (b:Branch) ON (b.location)` |
| **Lookup** | Label/rel-type lookups (auto-created) | Internal — created automatically |

**Text vs. B-tree for string matching:**

| Operation | B-tree | Text Index |
|-----------|--------|------------|
| `= 'exact'` | ✅ Fast | ❌ Not supported |
| `STARTS WITH 'prefix'` | ✅ Fast | ✅ |
| `CONTAINS 'substring'` | ❌ Full scan | ✅ Fast |
| `ENDS WITH 'suffix'` | ❌ Full scan | ✅ Fast |
| Full-text (stemming) | ❌ | ✅ Fast |

### 3.2 Index Management

```cypher
// Single-property B-tree
CREATE INDEX FOR (c:Customer) ON (c.id)
// Composite
CREATE INDEX FOR (t:Transaction) ON (t.date, t.amount)
// Text
CREATE TEXT INDEX FOR (c:Customer) ON (c.name)
// Full-text across multiple labels/properties
CALL db.index.fulltext.createNodeIndex('customerSearch', ['Customer'], ['name', 'email'])
// Drop
DROP INDEX index_name
```

**Indexes from constraints:**
```cypher
// Creates unique constraint + B-tree index
CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.id IS UNIQUE
// Creates existence constraint + index
CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.email IS NOT NULL
// Creates node key constraint + composite index
CREATE CONSTRAINT FOR (c:Customer) REQUIRE (c.id, c.country) IS NODE KEY
```

### 3.3 Index Usage

Neo4j's planner considers index usage for: equality comparisons, range queries, IN lists, STARTS WITH, IS NOT NULL checks, ORDER BY (pre-sorted), and distance queries (point index only).

**Index hints** — force the planner to use a specific index:
```cypher
MATCH (t:Transaction) USING INDEX t:Transaction(date, amount)
WHERE t.date >= date('2024-01-01') AND t.amount > 10000 RETURN t
```

**Monitoring:**
```cypher
SHOW INDEXES               -- List all indexes with status
SHOW CONSTRAINTS           -- List all constraints
CALL db.indexes()          -- Legacy API
CALL db.constraints()      -- Legacy API
```

In query plans: `NodeIndexSeek` ✅ (good), `NodeIndexScan` ⚠️ (OK for large sets), `NodeByLabelScan` ⚠️ (no index), `AllNodesScan` ❌ (bad).

### 3.4 Index Maintenance

Indexes are built **asynchronously**. States: `POPULATING` → `ONLINE` | `FAILED`.

**Composite index property order matters:** An index on `(a, b)` supports `WHERE a = $v1 AND b = $v2` and `WHERE a = $v1`, but NOT `WHERE b = $v2` (skipping leading property). Put the most selective property first.

**Selection guidelines:**
| Pattern | Recommended Index |
|---------|------------------|
| Single-property equality | Single B-tree |
| Multi-property equality | Composite B-tree |
| Range + equality | Composite with range property last |
| Full-text | Text index |
| Spatial | Point index |
| ORDER BY + equality | Index on (filter, sort) |

---

## 4. Cypher Query Optimization

### 4.1 Query Planning Pipeline

```
Cypher → Parser → AST → Rewriter → Logical Plan → Cost-Based Optimizer → Execution Plan → Runtime
```

The optimizer generates multiple physical plans and selects the lowest-cost based on **cardinality estimates** (label statistics, index selectivity, degree distributions). Stale statistics after large data loads can cause poor plan selection. Run `CALL db.stats.retrieve('GRAPH')` to refresh.

### 4.2 Reading Query Plans

**EXPLAIN** — shows plan without executing:
```cypher
EXPLAIN MATCH (c:Customer {id: 'CUST001'})-[:OWNS]->(a:Account) RETURN a
```

**PROFILE** — executes and returns profiling info:
```cypher
PROFILE MATCH (c:Customer {id: 'CUST001'})-[:OWNS]->(a:Account) RETURN a
```

**Key operators:**

| Operator | Meaning |
|----------|---------|
| `NodeIndexSeek` | Index lookup (~constant time) ✅ |
| `NodeIndexScan` | Full index scan ⚠️ |
| `NodeByLabelScan` | Scans all nodes of a label ⚠️ |
| `AllNodesScan` | Scans every node ❌ |
| `Expand(All)` | Relationship traversal ✅ |
| `CartesianProduct` | Cross-join ❌ |
| `Filter` | Row filtering ⚠️ |
| `Aggregation` | GROUP BY / COUNT / SUM ⚠️ |
| `Sort` | ORDER BY ⚠️ |
| `Skip`/`Limit` | Paging ✅ |

High DB hits on `Filter` after `Expand` = missing index on the filtered property. High DB hits on `AllNodesScan` = always bad.

### 4.3 Common Optimization Patterns

**Filter early** — reduce cardinality before complex operations:
```cypher
// GOOD
MATCH (c:Customer {risk_rating: 'HIGH'})
MATCH (c)-[:OWNS]->(a:Account)
WITH c, count(a) AS account_count
WHERE account_count > 5 RETURN c.id, account_count
```

**Avoid CartesianProduct** — every unconnected MATCH creates an implicit cross-join:
```cypher
// BAD — every Customer × every Transaction
MATCH (c:Customer), (t:Transaction)
// GOOD — connect patterns
MATCH (c:Customer)-[:OWNS]->(a:Account)-[:PERFORMS]->(t:Transaction)
```

**Use index-supported operations** — ensure `NodeIndexSeek` appears in PROFILE, not `NodeByLabelScan`:
```cypher
MATCH (c:Customer {id: 'CUST001'})  → NodeIndexSeek
MATCH (c:Customer) WHERE c.id = 'CUST001'  → NodeIndexSeek if indexed
MATCH (c) WHERE c.id = 'CUST001'  → AllNodesScan (no label)
```

**Prefer directed relationships** — `(a)-[:OWNS]->(b)` over `(a)-[:OWNS]-(b)`.

**Bound variable-length traversals:**
```cypher
// GOOD — bounded
MATCH path = (a:Account)-[:SENT_TO*1..5]->(b:Account)
// BAD — unbounded (heap exhaustion risk)
MATCH path = (a:Account)-[:SENT_TO*]->(b:Account)
```

### 4.4 MATCH Optimization

**Most selective MATCH first:**
```cypher
// GOOD — start with the rarest entity
MATCH (t:Transaction {amount: 999999999, currency: 'SGD'})
MATCH (t)<-[:PERFORMS]-(a:Account) MATCH (a)<-[:OWNS]-(c:Customer)
// BAD — millions of customers scanned first
MATCH (c:Customer) MATCH (c)-[:OWNS]->(a:Account)-[:PERFORMS]->(t:Transaction {amount: 999999999})
```

Always use labels: `MATCH (n:Person)` not `MATCH (n)`. Without labels, Neo4j falls back to `AllNodesScan`.

### 4.5 OPTIONAL MATCH Caveats

Can produce CartesianProducts if unconnected to the outer scope. Ensure the optional pattern joins on an existing variable:
```cypher
// GOOD
MATCH (c:Customer) OPTIONAL MATCH (c)-[:OWNS]->(a:Account)
// BAD — implicit cross-join
MATCH (c:Customer) OPTIONAL MATCH (a:Account)
```

### 4.6 Aggregation, WITH, UNION, CALL Subquery

**Aggregate early:**
```cypher
MATCH (c:Customer)-[:OWNS]->(a:Account)
WITH c, count(a) AS account_count WHERE account_count > 3
MATCH (c)-[:HAS_LOAN]->(l:Loan)
RETURN c.id, account_count, count(l) AS loan_count
```

**WITH** — pipeline break that materializes intermediate results:
```cypher
MATCH (c:Customer)-[:OWNS]->(a:Account)
WITH c, count(a) AS cnt ORDER BY cnt DESC LIMIT 10
MATCH (c)-[:HAS_LOAN]->(l:Loan)
RETURN c.id, cnt, count(l)
```

**UNION:** Prefer `UNION ALL` (no dedup) over `UNION` (dedup overhead). Column names/types must match.

**CALL subquery** — correlated subqueries with explicit scoping:
```cypher
MATCH (c:Customer)
CALL {
  WITH c
  MATCH (c)-[:PERFORMS]->(t:Transaction) WHERE t.amount > 10000
  RETURN count(t) AS large_tx_count
}
RETURN c.id, large_tx_count
```

### 4.7 Performance Pitfalls Summary

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| AllNodesScan | High DB hits in PROFILE | Add label + index |
| Eager aggregation before filter | High memory | Filter before aggregation |
| No label on MATCH | AllNodesScan | Add label |
| Unbounded variable-length | Heap exhaustion | Add upper bound: `*1..10` |
| Large IN lists | Slow index seek | Use temp node instead |
| Not using parameters | Recompilation every execution | Use `$param` |
| Unconnected MATCH | CartesianProduct | Connect patterns |

### 4.8 Query Optimization Patterns

**Pattern 1 — Index lookup + expand:**
```cypher
MATCH (a:Account {id: $accountId})-[r:SENT_TO]->(b:Account)
RETURN b.id, r.amount, r.timestamp
```

**Pattern 2 — Traversals with limits:**
```cypher
MATCH path = (a:Person)-[:KNOWS*1..3]-(b:Person) RETURN b.name LIMIT 100
```

**Pattern 3 — Shortest path** (bidirectional BFS, dramatically faster than manual expansion):
```cypher
MATCH p = shortestPath((a:Account {id: $from})-[:SENT_TO*]-(b:Account {id: $to}))
RETURN p
```
Use only for the **shortest** path, not all paths. Supports max length: `*..10`.

**Pattern 4 — Fan pattern with seek pagination** (better than large SKIP):
```cypher
MATCH (hub:Account {id: $hubId})-[r:SENT_TO]->(n)
WHERE r.timestamp < $lastSeen
RETURN n.id, r.amount, r.timestamp ORDER BY r.timestamp DESC LIMIT $limit
```

**Pattern 5 — Relationship type filtering:**
```cypher
MATCH (a:Account)-[:OWNS|MANAGES]->(b:Account) RETURN a.id, b.id
```

**Pattern 6 — Property existence:**
```cypher
MATCH (c:Customer) WHERE c.email IS NOT NULL RETURN c.id
```

**Pattern 7 — Relationship existence check** (faster than OPTIONAL MATCH for boolean checks):
```cypher
MATCH (c:Customer) WHERE exists((c)-[:OWNS]->(:Account)) RETURN c.id
```

**Pattern 8 — Subquery for complex filtering:**
```cypher
MATCH (c:Customer)
CALL {
  WITH c
  MATCH (c)-[:OWNS]->(a:Account)-[:PERFORMS]->(t:Transaction)
  WHERE t.amount > 10000 AND t.currency = 'USD'
  RETURN count(t) AS high_value_tx
}
WHERE high_value_tx > 5 RETURN c.id, c.name, high_value_tx
```

---

## 5. Performance Tuning

### 5.1 Hardware Considerations

**Memory:** Two critical regions:
- **Heap** (`dbms.memory.heap.max_size`): For query execution, sorting, aggregation, transaction state. 4–32 GB typical. Too large → GC pauses.
- **Page cache** (`dbms.memory.pagecache.size`): Memory-maps store files — nodes, relationships, properties. **Aim for 60–70% of available RAM.** If the graph fits in page cache, queries run at memory speed.

**Formula:** Page cache = (nodes × ~15B) + (rels × ~45B) + property store + ~20% overhead. A 1B-nodes, 5B-rels graph needs ~200–400 GB page cache.

**CPU:** 8–32 cores. Parallel execution benefits concurrent query throughput.

**Storage:** NVMe SSDs essential. Page cache misses fall to disk; random read performance is critical.

### 5.2 Configuration Tuning

```ini
dbms.memory.heap.initial_size=4G
dbms.memory.heap.max_size=12G
dbms.memory.pagecache.size=40G
dbms.tx_log.rotation.retention_policy=7 days
db.logs.query.enabled=true
db.logs.query.threshold=1000ms
dbms.connector.bolt_thread_pool_max_size=400
```

### 5.3 Page Cache Monitoring

```cypher
CALL db.queryJmx("org.neo4j:instance=kernel#0,name=Page cache")
```
- **Hit ratio >99%** → Excellent
- **95–99%** → Good, but increase page cache if possible
- **<95%** → Poor performance, increase page cache

### 5.4 Connection Pooling

Use connection pools (HikariCP, neo4j-driver built-in pool). Pool size = expected concurrent queries × connections/query. Avoid creating a connection per query — the Bolt handshake is expensive.

```python
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"),
    max_connection_pool_size=50, connection_acquisition_timeout=60)
```

### 5.5 Monitoring Commands

**Neo4j Browser:** `:sysinfo` (system info), `:queries` (running queries), `:config` (effective config), `:monitor` (metrics).

**Cypher:**
```cypher
CALL dbms.listActiveTransactions()  -- Active transactions
CALL dbms.listQueries()             -- Running queries
CALL dbms.killQuery('query-id')     -- Kill long-running query
CALL db.schema.visualization()      -- Visual schema
CALL db.indexes()                   -- Index status
```

**Slow query identification:** Enable `db.logs.query.enabled=true` with `db.logs.query.threshold=1000ms`. Review `logs/query.log`. PROFILE suspicious queries.

### 5.6 Profiler Output Reading

Read bottom-to-top. Key metrics: **DB hits** (page accesses), **Rows** (actual cardinality), **Estimated Rows** (planner estimate — large discrepancy = stale stats).

Red flags:
- `AllNodesScan` with >0 DB hits → missing label/index
- `CartesianProduct` → unconnected MATCHes
- `Filter` processing 10× more rows than child → filter not pushed to index
- `Sort` on millions of rows → consider index-backed ordering
- `EagerAggregation` before restrictive filter → restructure WITH

---

## 6. Neo4j for Banking

### 6.1 Common Banking Graph Models

**Transaction Network Analysis:**
```cypher
(:Account)-[:SENT_TO {amount, timestamp, currency, channel, status}]->(:Account)
// Or with explicit Transaction nodes for rich data:
(:Account)-[:PERFORMS]->(:Transaction {id, amount, currency, timestamp})-[:RECEIVED_BY]->(:Account)
```
Use cases: AML SAR detection, circular transaction discovery, velocity checks, flow visualization.

**Customer Relationship Management:**
```cypher
(:Customer)-[:HOLDS]->(:Account)-[:PERFORMS]->(:Transaction)
(:Customer)-[:IS_PRIMARY_HOLDER]->(:Account)
(:Customer)-[:IS_JOINT_HOLDER]->(:Account)
```
Use cases: Customer 360, cross-sell, relationship pricing, household discovery.

**KYC Entity Resolution:**
```cypher
(:Party {id: 'LEI123', type: 'CORPORATE'})-[:RELATED_TO {relationship_type: 'BENEFICIAL_OWNER', ownership_pct: 75}]->(:Party)
(:Document {type: 'PASSPORT'})-[:IDENTIFIES]->(:Party)
```
Use cases: UBO chain discovery, sanctions screening, PEP detection, corporate structure analysis.

**Multi-hop beneficial ownership:**
```cypher
MATCH path = (i:Party {type: 'INDIVIDUAL'})-[:RELATED_TO*1..10]->(e:Party {type: 'CORPORATE'})
WHERE all(r IN relationships(path) WHERE r.relationship_type = 'BENEFICIAL_OWNER')
RETURN i.name, e.name, reduce(acc = 1.0, r IN relationships(path) | acc * r.ownership_pct / 100) AS effective_ownership
ORDER BY effective_ownership DESC
```

**Trade Finance:**
```cypher
(:Trade {id: 'TF2024001', type: 'LC', amount: 500000})
(:Trade)-[:INVOLVES {role: 'ISSUING_BANK'}]->(:Party)
(:Document)-[:SUPPORTS]->(:Trade)
(:Shipment)-[:LINKED_TO]->(:Trade)
```
Use cases: LC lifecycle tracking, document discrepancy analysis, trade exposure aggregation.

**Regulatory Reporting:**
```cypher
(:Report {type: 'BASEL_III'})-[:COVERS]->(:Entity {lei: 'LEI123'})
(:Regulation {code: 'BCBS239'})-[:APPLIES_TO]->(:Entity)
```
Use cases: Regulatory obligation mapping, BCBS 239 data lineage, change impact analysis.

### 6.2 Data Modeling for Compliance

**Audit trails on relationships:**
```cypher
(c)-[:OWNS {since: datetime('2024-01-01'), until: datetime('2024-06-30'),
            created_by: 'ADMIN', created_at: datetime(), version: 2}]->(a)
```

**Change history as event nodes:**
```cypher
CREATE (e:ChangeEvent {entity_type: 'Customer', entity_id: 'CUST001', field: 'risk_rating',
    old_value: 'LOW', new_value: 'HIGH', changed_by: 'COMPLIANCE_USER',
    changed_at: datetime('2024-03-15T10:00:00Z')})
CREATE (e)-[:CHANGED]->(c)
```

**Temporal relationships for point-in-time queries:**
```cypher
MATCH (c:Customer)-[r:OWNS]->(a:Account)
WHERE r.valid_from <= datetime($asOfDate) AND (r.valid_to IS NULL OR r.valid_to >= datetime($asOfDate))
RETURN c.id, a.id, r.ownership_pct
```

**Immutable transaction records:** Transactions are append-only — corrections create new `:CorrectionTransaction` nodes linked via `[:CORRECTS]`, never update the original.

**Data retention:** Soft-delete with `retention_status = 'PURGE_PENDING'`, purge in scheduled batches. Regulatory requirements typically mandate 5–10 year retention for transaction data.

### 6.3 Performance Considerations for Banking

| Factor | Impact | Mitigation |
|--------|--------|------------|
| 100M–1B+ nodes | Index size, page cache pressure | Time/region/entity-type partitioning |
| High-degree nodes (10M+ rels) | Expansion perf, memory | Rel-type partitioning, skip/limit, supernode splitting |
| Multi-hop path queries | Exponential traversal cost | Bounded patterns, shortestPath, subgraph projections |
| Sub-second fraud detection | Plan stability, page cache | Index coverage, plan caching, pre-warmed page cache |
| Batch AML reporting | Resource contention | Read replicas, APOC periodic commit |

### 6.4 Security

```ini
dbms.security.auth_enabled=true
dbms.security.ldap.authentication_enabled=true
dbms.security.ldap.server=ldap://ldap.bank.internal:389
dbms.security.encryption_at_rest.enabled=true
```

**Granular subgraph access control:**
```cypher
CREATE ROLE customer_service
GRANT MATCH {*} ON GRAPH * NODES Customer, Account TO customer_service
GRANT MATCH {*} ON GRAPH * RELATIONSHIPS OWNS, HOLDS TO customer_service
DENY MATCH {*} ON GRAPH * NODES Transaction TO customer_service
CREATE USER alice SET PASSWORD 'changeme' CHANGE NOT REQUIRED
GRANT ROLE customer_service TO alice
```

### 6.5 HA and Clustering

**Causal clustering:** RAFT consensus among core servers (min 3). Read replicas scale query throughput. Bookmarks ensure causal consistency.

```ini
// Core server
dbms.mode=CORE
dbms.cluster.discovery.type=LIST
dbms.cluster.discovery.endpoints=core1:5000,core2:5000,core3:5000
// Read replica
dbms.mode=READ_REPLICA
```

**Multi-DC replication:** Deploy core servers across two data centers. Keep RAFT majority in one DC to avoid cross-DC write latency. Read replicas in each DC serve local reads.

**Backup:**
```bash
neo4j-admin backup --from=localhost:7687 --backup-dir=/backup/neo4j  # Online
neo4j-admin database dump neo4j --to=/backup/neo4j-dump             # Offline
```
Daily full + hourly incremental backups. Quarterly restore tests. Encrypted off-site storage per regulatory requirements.

---

## 7. Query Tuning Checklist

### 7.1 Profile & Analyze
- [ ] Run `PROFILE <query>`, identify the operator with highest DB hits
- [ ] Compare estimated rows vs. actual — large gap = stale statistics
- [ ] Check for `AllNodesScan`, `CartesianProduct`, or `EagerAggregation`

### 7.2 Check Index Coverage
- [ ] Every `MATCH (n:Label)` with WHERE clauses has an index on filtered properties
- [ ] Composite index available for multi-property filters
- [ ] Text index for CONTAINS/ENDS_WITH, point index for distance queries
- [ ] All critical indexes in `ONLINE` state (not `POPULATING` or `FAILED`)

### 7.3 Verify MATCH Patterns
- [ ] Every MATCH includes at least one node label
- [ ] Most selective MATCH is first
- [ ] All MATCH clauses are connected (no CartesianProduct)
- [ ] Relationship direction is specified
- [ ] Variable-length relationships have an upper bound
- [ ] `shortestPath` used when only the shortest path is needed

### 7.4 Optimize Data Flow
- [ ] Filter early — use WITH to reduce cardinality before complex ops
- [ ] Aggregate early to reduce row count
- [ ] LIMIT for exploratory/paginated queries
- [ ] Avoid large SKIP — use seek pagination (`WHERE id > $last ORDER BY id LIMIT $n`)
- [ ] Split complex queries into WITH stages

### 7.5 Parameters & Literals
- [ ] All filter values use `$param` not literals
- [ ] IN lists >1000 items replaced with temp node lookup
- [ ] LIMIT and SKIP use parameters

### 7.6 Anti-Pattern Check
- [ ] No AllNodesScan
- [ ] No CartesianProduct
- [ ] No unbounded variable-length paths
- [ ] No unconnected OPTIONAL MATCH producing cross-join
- [ ] No redundant filtering

### 7.7 System-Level
- [ ] Page cache hit ratio >99%
- [ ] No long-running queries blocking tx log cleanup
- [ ] Database statistics up to date
- [ ] Connection pool adequately sized
- [ ] No contention from long-running write transactions

### 7.8 Post-Tuning Validation
- [ ] Re-profile — DB hits significantly reduced
- [ ] Compare execution time before/after
- [ ] Verify correctness (same results)
- [ ] Benchmark under realistic load
- [ ] Document tuned query and schema changes

### 7.9 Operator Red Flags Quick Reference

| Operator Issue | What It Means | Action |
|----------------|---------------|--------|
| `AllNodesScan` | No label or index | Add label + index |
| `NodeByLabelScan` | No index on filtered property | Add index |
| `CartesianProduct` | Unconnected MATCH | Connect patterns |
| `Filter` with 10× rows of child | Filter not pushed to index | Add composite index |
| `EagerAggregation` before filter | Aggregation consuming memory | Filter before aggregation |
| `Sort` on large row count | No index for ordering | Add index on ORDER BY column |
| `Skip` with large value | Scanning skipped rows | Use seek pagination |

### 7.10 Quick Wins
1. **Update statistics:** `CALL db.stats.retrieve('GRAPH')`
2. **Restart database** (clears plan cache)
3. **Check disk I/O** — saturated SSDs from page cache misses
4. **Increase page cache** — most impactful single config change
5. **Use read replicas** — isolate reporting from transactional load
6. **APOC periodic commit** for large batches:
   ```cypher
   CALL apoc.periodic.iterate(
     'MATCH (t:Transaction) WHERE t.amount > 100000 RETURN t',
     'SET t.flagged = true', {batchSize: 10000})
   ```

---

## References

- [Neo4j Cypher Manual](https://neo4j.com/docs/cypher-manual/current/) — Official Cypher documentation
- [Neo4j Operations Manual](https://neo4j.com/docs/operations-manual/current/) — Configuration, clustering, tuning
- [Neo4j Graph Data Science](https://neo4j.com/docs/graph-data-science/current/) — PageRank, community detection, centrality
- *Graph Databases* by Robinson, Webber, Eifrem — O'Reilly (canonical text)
- [Neo4j Developer Blog](https://neo4j.com/blog/) — Production case studies

---

*This guide is maintained by Jack Liu Shurui. For corrections, open an issue or PR on [github.com/jackliusr/research](https://github.com/jackliusr/research).*
