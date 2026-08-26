# Neo4j — The Leading Graph Database Platform (Complete Reference)

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Cymbal Bank  
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Audience:** Engineers, architects, and data practitioners building production graph applications  
> **Part of:** research/technology/ — ties together neo4j_graph_database_guide, neo4j_bloom_alternatives_guide, graph_engineering_guide, and llm_frameworks_comparison_guide

---

## Table of Contents

1. [What Is Neo4j?](#1-what-is-neo4j)
2. [The Property Graph Model](#2-the-property-graph-model)
3. [Cypher Query Language](#3-cypher-query-language)
4. [Neo4j Deployment Options](#4-neo4j-deployment-options)
5. [Use Cases](#5-use-cases)
6. [Graph Data Science (GDS) Library](#6-graph-data-science-gds-library)
7. [Neo4j for GenAI and LLM](#7-neo4j-for-genai-and-llm)
8. [Ecosystem and Tools](#8-ecosystem-and-tools)
9. [Getting Started Checklist](#9-getting-started-checklist)

---

## 1. What Is Neo4j?

### 1.1 Definition

Neo4j is the world's leading **graph database** — a native graph storage and processing engine that stores data as **nodes** (entities), **relationships** (connections), and **properties** (attributes). It is fully ACID-compliant and uses the **labeled property graph model** queried via **Cypher**, a declarative pattern-matching query language purpose-built for graphs.

Unlike relational databases that infer relationships through foreign keys and JOIN operations, Neo4j stores relationships as first-class, pointer-chased objects. Graph traversals follow adjacency pointers at memory speed rather than computing join cartesian products — linear cost with path depth instead of exponential JOIN fan-out.

### 1.2 Company History & Enterprise Adoption

Founded 2007 (Emil Eifrem, Malmö, Sweden), HQ in San Mateo, CA. Cypher released 2011, open-sourced as openCypher in 2015 (adopted by Amazon Neptune, Memgraph, RedisGraph, SAP HANA Graph). Neo4j 4.0 (2019) introduced causal clustering; Neo4j 5.x (2022+) added vector search, GDS 2.x, and native GenAI integrations. AuraDB launched as fully managed cloud service.

Used by enterprises including **eBay** (fraud), **UBS** (AML), **Crédit Agricole** (transaction analysis), **Walmart** (supply chain), **NASA** (data lineage), **Comcast** (network), **Adobe**, **Cisco**, and many more.

### 1.4 How Neo4j Differs from Other Database Paradigms

#### vs. Relational Databases (SQL)

| Dimension | Relational (SQL) | Neo4j (Graph) |
|-----------|-----------------|---------------|
| **Relationship handling** | Foreign keys + JOINs at query time | Direct pointers stored with the data |
| **Depth-N queries** | N self-JOINs or recursive CTEs (exponential cost) | Pointer traversal (linear cost) |
| **Schema** | Rigid — ALTER TABLE requires migration | Schema-optional — add labels/properties on the fly |
| **Performance model** | Cost grows with join fan-out × depth | Cost grows linearly with path length |
| **Connected data** | Expensive beyond 2–3 JOINs | Optimal (designed for it) |
| **Agility** | Schema changes require migration tooling | Schema evolution is additive, non-breaking |

#### vs. Document Databases (MongoDB, DynamoDB)

| Dimension | Document DB | Neo4j |
|-----------|-------------|-------|
| **Relationships** | Embedded/nested or manual references | First-class, directed, typed, property-bearing |
| **Joins** | `$lookup` / application-side | Native traversal operators |
| **Multi-hop queries** | N nested lookups ($graphLookup) | Single Cypher pattern |
| **Schema** | Flexible documents | Flexible property graph |

#### vs. RDF Triple Stores (SPARQL)

| Dimension | RDF / SPARQL | Neo4j (Property Graph) |
|-----------|-------------|----------------------|
| **Data unit** | Subject → predicate → object triples | Nodes + relationships with properties |
| **Relationships** | Named predicates (no edge properties) | Typed, directed, can carry properties |
| **Schema** | Ontology-driven (OWL/RDFS) | Schema-optional with labels |
| **Query language** | SPARQL (W3C standard) | Cypher (pattern-matching, declarative) |
| **Performance** | Triple-pattern matching | Graph traversal (adjacency pointers) |
| **ACID** | Varies by implementation | Fully ACID |
| **Best for** | Semantic web, ontologies, inference | Transactional graph workloads, real-time queries |

---

## 2. The Property Graph Model

Neo4j implements the **labeled property graph (LPG) model** — the most widely adopted graph database data model, also used by Amazon Neptune, Memgraph, and others.

### 2.1 Nodes

**Nodes** represent entities — a person, an account, a transaction, a document, a device. Every node has:

- A unique internal ID (assigned by the database, not user-controlled)
- One or more **labels** that group it into categories
- Zero or more **properties** (key-value pairs)

```cypher
// A node with a single label
CREATE (:Person {name: 'Alice', dateOfBirth: date('1990-05-15')})

// A node with multiple labels
CREATE (:Person:Employee:Manager {employeeId: 'EMP001', department: 'Trading'})
```

**Labels** are the primary grouping mechanism:
- Used for indexing (`CREATE INDEX FOR (p:Person) ON (p.name)`)
- Used in queries to restrict pattern matching (`MATCH (p:Person)`)
- A node can have 0+ labels (unlabeled nodes cause `AllNodesScan` — avoid)
- Conventions: PascalCase (`:Person`, not `:person`)

**When to use multiple labels:** Taxonomy membership (`:Vehicle:Car:Sedan`), role composition (`:Person:Employee:Manager`), subgraph access control. **When not to:** If the combination affects <1% of nodes, use a property instead.

### 2.2 Relationships

**Relationships** are directed, typed connections between a source node and a target node. Every relationship has:

- **Direction** — required; always from source node to target node
- **A single type** — semantically names the connection (`KNOWS`, `OWNS`, `SENT_TO`)
- **Optional properties** — key-value pairs carrying metadata about the connection

```cypher
// Relationship with properties
CREATE (a:Person {name: 'Alice'})-[r:KNOWS {since: date('2010-06-01'), trustLevel: 0.85}]->(b:Person {name: 'Bob'})

// Relationship types are UPPER_SNAKE_CASE by convention
CREATE (c:Customer)-[:OWNS {since: datetime('2024-01-01'), ownershipPct: 100}]->(acct:Account)
```

**Key characteristics:**
- Every relationship has exactly ONE type — use separate types for different semantics
- Direction matters for performance: `(a)-[:OWNS]->(b)` is faster than `(a)-[:OWNS]-(b)` because undirected checks both directions
- Relationships in Neo4j are stored as doubly linked lists per relationship type per node — expanding a typed, directed traversal is O(1) pointer dereference per relationship

### 2.3 Properties

Properties are key-value pairs on both nodes and relationships. Native property types:

| Type | Description | Example |
|------|-------------|---------|
| `String` | UTF-8 text | `'Singapore'` |
| `Integer` | 64-bit signed | `42` |
| `Float` | 64-bit IEEE 754 | `3.14159` |
| `Boolean` | true/false | `true` |
| `ByteArray` | Binary (max 2GB) | `$binaryData` |
| `Date` | ISO 8601 date | `date('2024-06-15')` |
| `DateTime` | Date+time+timezone | `datetime('2024-06-15T10:30:00+08:00')` |
| `LocalDateTime` | Date+time, no timezone | `localdatetime('2024-06-15T10:30:00')` |
| `LocalTime` | Time only | `localtime('10:30:00')` |
| `Duration` | ISO 8601 duration | `duration('P1Y2M3D')` |
| `Point` | 2D/3D spatial point | `point({latitude:1.35, longitude:103.82})` |
| `List` | Homogeneous list | `['a','b','c']` |

**Conventions:** camelCase for property keys (`firstName` not `first_name`). Consistent across labels and relationship types.

### 2.4 Paths

A **path** is a sequence of alternating nodes and relationships — the result of a graph traversal. Paths are first-class citizens in Cypher:

```cypher
// Variable-length path from Alice to connections up to 3 hops away
MATCH path = (a:Person {name: 'Alice'})-[:KNOWS*1..3]->(b:Person)
RETURN path, length(path) AS hops

// Shortest path (bidirectional BFS)
MATCH p = shortestPath((:Account {id: $from})-[:SENT_TO*]-(:Account {id: $to}))
RETURN p
```

Path functions: `length(path)` (number of relationships), `nodes(path)`, `relationships(path)`, `startNode(path)`, `endNode(path)`.

### 2.5 Schema Flexibility

Neo4j is **schema-optional** — you can create labels and properties on the fly without DDL:

```cypher
// First query creates a Person with a property
CREATE (p:Person {name: 'Alice'})
// Later query adds an entirely new property on a different Person
CREATE (p:Person {name: 'Bob', employeeId: 'EMP002', department: 'Risk'})
// Properties coexist — no migration needed
```

Optional **constraints and indexes** add data integrity and performance guarantees:
- Unique constraints
- Existence constraints
- Node key constraints (composite unique + existence)
- Property indexes (B-tree, range, text, point)

### 2.6 Graph Model Comparison Summary

| Dimension | Property Graph (Neo4j) | RDF / SPARQL | Document + $graphLookup | Relational + Recursive CTE |
|-----------|----------------------|-------------|------------------------|---------------------------|
| **Entity** | Nodes with labels + properties | Triple subjects | Documents with `_id` | Rows in tables |
| **Relationship** | Directed, typed, property-bearing | Predicates (no edge properties) | Embedded refs / `$lookup` | Foreign keys |
| **Multi-hop** | `*1..N` pattern (native) | SPARQL property paths | `$graphLookup` (bounded) | Recursive CTEs |
| **Schema** | Schema-optional with optional constraints | Ontology-driven (OWL/RDFS) | Flexible schema | Rigid schema |
| **Performance** | Pointer-chase traversal | Triple-pattern matching | Nested aggregation lookups | Join-based |
| **ACID** | Fully ACID | Varies by implementation | Document-level atomicity | Fully ACID |
| **Best for** | OLTP traversals, real-time queries, fraud, KGs | Semantic web, reasoning, linked data | Document-oriented apps with occasional graph needs | Traditional OLTP with reporting |

---

## 3. Cypher Query Language

### 3.1 Overview

**Cypher** is a declarative, pattern-matching query language for graphs, originally developed by Neo4j in 2011 and open-sourced as **openCypher** in 2015. It is now used by multiple databases:

- **Neo4j** (native implementation)
- **Amazon Neptune** (via openCypher support)
- **Memgraph** (Cypher-compatible)
- **RedisGraph** (Redis module, Cypher subset)
- **SAP HANA Graph** (Cypher-inspired)

Cypher's syntax is inspired by ASCII art of graph patterns: `(node)-[:REL]->(node)`. It is designed to be intuitive — you draw the pattern of data you want to find.

### 3.2 Basic Syntax Structure

Every Cypher query follows a pipeline pattern:

```cypher
MATCH    -- Find patterns in the graph
WHERE    -- Filter results
WITH     -- Pipe and transform intermediate results
RETURN   -- Project final results
ORDER BY -- Sort
SKIP     -- Paginate: skip N rows
LIMIT    -- Paginate: limit to N rows
```

**Core clauses:**

| Clause | Purpose |
|--------|---------|
| `MATCH` | Find patterns in the graph (required for reads) |
| `OPTIONAL MATCH` | Like LEFT JOIN — returns null when pattern not found |
| `WHERE` | Filter results (AND, OR, NOT, comparisons, string matching) |
| `RETURN` | Project, alias, aggregate, order, limit |
| `CREATE` | Create nodes and relationships unconditionally |
| `MERGE` | Match or create (upsert pattern) |
| `DELETE` / `DETACH DELETE` | Delete nodes (DETACH also deletes relationships) |
| `SET` | Update properties or add labels |
| `REMOVE` | Remove properties or labels |
| `WITH` | Pipe intermediate results, filter aggregates (HAVING equivalent) |
| `CALL` | Invoke subqueries or built-in procedures |
| `UNION` / `UNION ALL` | Combine results from multiple queries |

### 3.3 Pattern Matching

Cypher patterns use parentheses for nodes and square brackets for relationships. Arrow `->` means directed, bare `-` means undirected:

```cypher
// Directed     (a:Person)-[:KNOWS]->(b:Person)
// Undirected   (a:Person)-[:KNOWS]-(b:Person)
// Any direction (a)--(b)
// 1-3 hops     (a)-[:KNOWS*1..3]->(b)
// Unbounded    (a)-[:KNOWS*]->(b)  — use with caution!
// Multi-type   (a)-[:KNOWS|OWNS]->(b)
// Shortest path shortestPath((a)-[:KNOWS*]-(b))
```

**Path variables** capture the entire matched path:

```cypher
MATCH path = (a:Person {name: 'Alice'})-[:KNOWS*1..3]->(b:Person)
RETURN path, length(path) AS hops,
       [n IN nodes(path) | n.name] AS node_names
```

### 3.4 Clauses in Detail

#### MATCH

Find patterns in the graph. Every `MATCH` should include a label to avoid `AllNodesScan`. Patterns are connected automatically — unconnected MATCH produce `CartesianProduct`:

```cypher
// GOOD — connected pattern with label
MATCH (c:Customer {id: 'CUST001'})-[:OWNS]->(a:Account) RETURN a
// BAD — implicit cross join
MATCH (c:Customer), (t:Transaction)
```

#### OPTIONAL MATCH

Like SQL `LEFT JOIN` — returns `null` for non-matching patterns. Must connect to an existing variable:

```cypher
MATCH (c:Customer) OPTIONAL MATCH (c)-[:OWNS]->(a:Account)
RETURN c.id, a.id  -- a.id is null if no account
```

#### WHERE

Filtering with `AND`, `OR`, `NOT`, parentheses for grouping:

```cypher
MATCH (t:Transaction)
WHERE t.amount > 10000
  AND t.currency IN ['USD', 'SGD']
  AND t.timestamp >= datetime('2024-01-01')
  AND t.status IS NOT NULL
  AND t.reference STARTS WITH 'SWIFT'
  AND t.description CONTAINS 'urgent'
RETURN t
```

**WHERE clause capabilities:**

| Operation | Syntax |
|-----------|--------|
| Equality | `=`, `<>` |
| Comparison | `<`, `>`, `<=`, `>=` |
| Range | `BETWEEN` (use `>= AND <=`) |
| List membership | `IN [...]`, `NOT IN [...]` |
| String prefix | `STARTS WITH` |
| String contains | `CONTAINS` |
| String suffix | `ENDS WITH` |
| Regex (limited) | `=~ 'regex'` |
| Existence | `IS NOT NULL`, `IS NULL` |
| Boolean logic | `AND`, `OR`, `NOT`, `()` |
| Pattern existence | `WHERE exists((n)-[:OWNS]->(:Account))` |

#### RETURN

Project, alias, order, aggregate, paginate:

```cypher
MATCH (c:Customer)-[:OWNS]->(a:Account)
RETURN c.id AS customerId,
       c.name AS customerName,
       count(a) AS accountCount,
       collect(a.id) AS accountIds
ORDER BY accountCount DESC
SKIP 10
LIMIT 10
```

**RETURN capabilities:**
- **Aliasing:** `AS` keyword
- **Ordering:** `ORDER BY property ASC|DESC` (multiple: `ORDER BY a, b DESC`)
- **Limiting:** `SKIP n`, `LIMIT n`
- **Aggregation:** `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `COLLECT`, `stDev`, `percentileCont`

#### Aggregation

Grouping in Cypher is **implicit** — non-aggregated RETURN columns become grouping keys:

```cypher
// Groups by c.id and c.name automatically
MATCH (c:Customer)-[:OWNS]->(a:Account)
RETURN c.id, c.name, count(a) AS accountCount,
       sum(a.balance) AS totalBalance,
       avg(a.balance) AS avgBalance,
       collect(a.id) AS accountIds
```

| Aggregate Function | Description |
|--------------------|-------------|
| `count(*)` | Count rows |
| `count(expr)` | Count non-null values |
| `count(DISTINCT expr)` | Count distinct non-null values |
| `sum(expr)` | Sum |
| `avg(expr)` | Average |
| `min(expr)` | Minimum |
| `max(expr)` | Maximum |
| `collect(expr)` | Collect values into a list |
| `stDev(expr)` | Sample standard deviation |
| `stDevP(expr)` | Population standard deviation |
| `percentileCont(expr, p)` | Continuous percentile |
| `percentileDisc(expr, p)` | Discrete percentile |

#### CREATE and MERGE

**CREATE** — unconditionally creates nodes and relationships:

```cypher
CREATE (p:Person:Employee {name: 'Alice', employeeId: 'E001'})
CREATE (p)-[:WORKS_IN {since: date('2020-01-01')}]->(d:Department {name: 'Trading'})
```

**MERGE** — match or create (upsert). Checks existence before creating:

```cypher
// Match existing Person or create one
MERGE (p:Person {name: 'Alice'})
// If created, set additional properties
ON CREATE SET p.createdAt = datetime()
ON MATCH SET p.lastSeenAt = datetime()

// MERGE a relationship
MATCH (a:Person {name: 'Alice'}), (b:Person {name: 'Bob'})
MERGE (a)-[r:KNOWS]->(b)
ON CREATE SET r.since = date()
```

**MERGE patterns:**
- `MERGE` checks the FULL pattern — partial matches do NOT create partial patterns
- For relationship properties, `MERGE ... ON CREATE SET` is the standard upsert pattern
- Avoid `MERGE` on high-concurrency patterns — deadlocks are possible; use `CREATE` + error handling for insert-mostly workloads

#### SET and REMOVE

```cypher
// Set/update properties
SET c.riskRating = 'HIGH'
SET c += {lastReviewed: date(), reviewer: 'COMPLIANCE_TEAM'}

// Add label
SET c:VipCustomer

// Remove property
REMOVE c.tempFlag

// Remove label
REMOVE c:Provisional
```

#### DELETE

```cypher
// Delete node (FAILS if it has relationships)
DELETE n

// Delete node and all its relationships
DETACH DELETE n

// Delete all nodes and relationships (careful!)
MATCH (n) DETACH DELETE n
```

#### WITH and Subqueries

**WITH** — pipes intermediate results between query parts. Acts as a pipeline break, materializing results. Used for:
- Filtering aggregated results (HAVING equivalent)
- Trimming properties before complex operations
- Paginating traversals

```cypher
MATCH (c:Customer)-[:OWNS]->(a:Account)
WITH c, count(a) AS accountCount
WHERE accountCount > 5  -- HAVING equivalent
MATCH (c)-[:HAS_LOAN]->(l:Loan)
RETURN c.id, accountCount, count(l) AS loanCount
```

**CALL subquery** — correlated and uncorrelated subqueries (Neo4j 4.1+):

```cypher
MATCH (c:Customer)
CALL {
    WITH c  -- correlated: pass outer variable
    MATCH (c)-[:PERFORMS]->(t:Transaction)
    WHERE t.amount > 10000
    RETURN count(t) AS largeTxCount
}
RETURN c.id, largeTxCount
```

**CALL procedure** — invoke built-in or custom procedures:

```cypher
CALL db.schema.visualization()
CALL db.indexes()
CALL dbms.listQueries()
```

### 3.5 Indexes and Constraints

#### Index Types

| Index Type | Best For | Example |
|-----------|----------|---------|
| **B-tree** (default) | Equality, range, ORDER BY, STARTS WITH | `CREATE INDEX FOR (c:Customer) ON (c.id)` |
| **Range** | Numeric/string range queries (optimized) | `CREATE RANGE INDEX FOR (t:Transaction) ON (t.amount)` |
| **Text** (Lucene) | CONTAINS, ENDS WITH, full-text search | `CREATE TEXT INDEX FOR (c:Customer) ON (c.name)` |
| **Point** | Spatial distance/bbox queries | `CREATE POINT INDEX FOR (b:Branch) ON (b.location)` |
| **Lookup** | Label/rel-type lookups (auto-created) | Internal — automatically created |
| **Full-text** | Stemming, fuzzy search across labels | `CALL db.index.fulltext.createNodeIndex(...)` |

```cypher
// Single-property B-tree   CREATE INDEX FOR (c:Customer) ON (c.id)
// Composite                CREATE INDEX FOR (t:Transaction) ON (t.date, t.amount)
// Text                     CREATE TEXT INDEX FOR (c:Customer) ON (c.name)
// Range                    CREATE RANGE INDEX FOR (t:Transaction) ON (t.timestamp)
// Point                    CREATE POINT INDEX FOR (b:Branch) ON (b.location)
// Full-text                CALL db.index.fulltext.createNodeIndex('customerSearch', ['Customer'], ['name', 'email'])
// Drop                     DROP INDEX index_name IF EXISTS
```

#### Constraint Types

```cypher
// Unique (creates B-tree index)     CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.id IS UNIQUE
// Existence (creates index)         CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.email IS NOT NULL
// Node key (composite unique + existence) CREATE CONSTRAINT FOR (c:Customer) REQUIRE (c.id, c.country) IS NODE KEY
// Show constraints                  SHOW CONSTRAINTS
```

### 3.6 Functions Reference

**String functions:**
`toUpper()`, `toLower()`, `substring()`, `replace()`, `trim()`, `lTrim()`, `rTrim()`, `split()`, `reverse()`, `toString()`, `left()`, `right()`, `size()` (string length)

**Aggregate functions:**
`count()`, `sum()`, `avg()`, `min()`, `max()`, `collect()`, `stDev()`, `stDevP()`, `percentileCont()`, `percentileDisc()`

**List functions:**
`size()`, `reverse()`, `head()`, `tail()`, `last()`, `range()`, `reduce()`, `filter()`, `extract()`

**Mathematical functions:**
`abs()`, `ceil()`, `floor()`, `round()`, `sqrt()`, `rand()`, `sign()`, `sin()`, `cos()`, `tan()`, `asin()`, `acos()`, `atan()`, `atan2()`, `exp()`, `log()`, `log10()`, `pi()`

**Temporal functions:** `date()`, `datetime()`, `localtime()`, `localdatetime()`, `time()`, `duration()`. Support creation (`date('2024-06-15')`), arithmetic (`date() + duration('P1D')`), truncation (`date.truncate('month', date())`), and formatting (`.toString()`, `.epochMillis`).

**Spatial functions:**
`point()`, `distance()`, `withinBBox()`, `point.distance()`, `point.withinBBox()`

**Predicate functions:**
`exists()`, `all()`, `any()`, `single()`, `none()`

### 3.7 Built-in Procedure Reference

| Procedure | Purpose |
|-----------|---------|
| `CALL db.schema.visualization()` | Return graph schema (node labels + relationship types) |
| `CALL db.indexes()` | List all indexes with status |
| `CALL db.constraints()` | List all constraints |
| `CALL db.labels()` | Return all node labels in use |
| `CALL db.relationshipTypes()` | Return all relationship types in use |
| `CALL db.propertyKeys()` | Return all property keys in use |
| `CALL db.schema.nodeTypeProperties()` | Schema per node label/property |
| `CALL dbms.listConfig()` | Show effective configuration |
| `CALL dbms.listQueries()` | List running queries with metadata |
| `CALL dbms.killQuery(queryId)` | Terminate a running query |
| `CALL dbms.listActiveTransactions()` | Show active transactions |
| `CALL db.stats.retrieve('GRAPH')` | Refresh cardinality statistics |
| `CALL db.index.fulltext.createNodeIndex(...)` | Create full-text search index |
| `CALL db.index.fulltext.queryNodes(...)` | Query full-text index |

### 3.8 Cypher Best Practices

- **Use parameters** (`$param`) instead of literals — prevents recompilation on every execution
- **Always specify labels** in MATCH — prevents `AllNodesScan`
- **Use directed relationships** — `(a)-[:OWNS]->(b)` is faster than undirected
- **Create indexes** on frequently filtered properties
- **Add existence constraints** for required fields to enforce data quality at commit time
- **Use LIMIT** for exploratory queries to avoid accidental full-graph scans
- **Use PROFILE/EXPLAIN** for query tuning — check for `NodeIndexSeek` vs `AllNodesScan`
- **Bound variable-length patterns** — `*1..5` prevents heap exhaustion
- **Avoid CartesianProduct** — always connect MATCH patterns
- **Most selective MATCH first** — start with the pattern that filters the most rows
- **Prefer `UNION ALL`** over `UNION` when dedup is not needed
- **Use `WITH` to pipe** intermediate results and filter aggregates (HAVING equivalent)
- **Use `shortestPath` for shortest paths** — bidirectional BFS is dramatically faster than manual expansion

---

## 4. Neo4j Deployment Options

Neo4j offers three deployment models: self-managed (Community/Enterprise), fully managed cloud (AuraDB), and self-managed on cloud infrastructure.

### 4.1 Neo4j Graph Database (Self-Managed)

#### Community Edition — GPL v3, single instance (no clustering), basic monitoring, manual backup, basic auth. Best for dev/learning.

#### Enterprise Edition — Commercial license, causal clustering (RAFT, min 3 cores + read replicas), advanced monitoring (JMX), RBAC/LDAP/SSO, online backups, full-text indexing, SLAs. Best for production workloads requiring HA and compliance.

#### Hardware Considerations

| Resource | Sizing Guidance |
|----------|----------------|
| **Memory** | Two regions: **Heap** (4–32 GB, query execution) + **Page cache** (60–70% of total RAM). If active graph fits in page cache, queries run at memory speed. |
| **Page cache formula** | (nodes × ~15B) + (rels × ~45B) + property store + 20% overhead |
| **CPU** | 8–32 cores. Parallel execution benefits concurrent query throughput. |
| **Storage** | NVMe SSD essential — page cache misses fall to disk; random read performance is critical. Separate disk for transaction logs. |
| **Operating system** | Linux (production), Windows/macOS (development) |

### 4.2 Neo4j AuraDB (Fully Managed Cloud)

| Feature | AuraDB Professional | AuraDB Enterprise |
|---------|-------------------|-------------------|
| **Max storage** | Up to 1 TB | Custom (multi-TB) |
| **Clustering** | HA with automatic failover | Multi-region HA |
| **Backups** | Automated, point-in-time recovery | Automated, point-in-time recovery |
| **Encryption** | At rest and in transit | At rest and in transit |
| **Compliance** | SOC 2, ISO 27001 | SOC 2, ISO 27001, HIPAA, PCI DSS |
| **Deployment** | AWS, GCP | AWS, GCP, multi-region |
| **Scaling** | Vertical | Vertical and horizontal |
| **Monitoring** | Query insights, metrics | Query insights, advanced metrics |
| **Support** | Standard SLA | Premium SLA |
| **Pricing** | Daily billing | Custom enterprise pricing |
| **Best for** | Production workloads without ops overhead | Regulated enterprises requiring compliance |

**AuraDB advantages:** Zero ops overhead, automated PITR backups, encryption at rest and in transit by default, query insights, and the same Neo4j 5.x engine as self-managed.

### 4.3 Neo4j on Cloud Infrastructure

Deploy Neo4j Enterprise on AWS, Azure, or GCP via marketplace (CloudFormation/Terraform templates). For teams needing cloud elasticity with full config control, VPC isolation, or existing monitoring tooling.

### 4.4 Deployment Comparison

| Dimension | Self-Managed Community | Self-Managed Enterprise | AuraDB Professional | AuraDB Enterprise |
|-----------|----------------------|----------------------|-------------------|-------------------|
| **Management overhead** | High | High | None | None |
| **Scalability** | Vertical only | Vertical + Read replicas | Vertical | Vertical + Horizontal |
| **HA/DR** | None | RAFT clustering, read replicas | Automatic failover | Multi-region |
| **Security** | Basic auth | RBAC, LDAP, SSO, encryption | Encryption, SOC 2 | Encryption, SOC 2, HIPAA, PCI |
| **Cost model** | Free (OSS) | License + infrastructure | Consumption-based | Enterprise subscription |
| **Best for** | Dev/learning | Production with dedicated ops | Ops-free production | Regulated enterprises |

---

## 5. Use Cases

### 5.1 Fraud Detection and Financial Services

Neo4j is the dominant graph database for financial crime detection, used by global banks, payment processors, and fintech companies.

**Real-time fraud detection:**
```cypher
// Score a transaction in sub-second time
MATCH (a:Account {id: $accountId})-[r:SENT_TO]->(target:Account)
WHERE r.timestamp >= datetime() - duration('PT1H')
RETURN count(r) AS txCount, sum(r.amount) AS totalAmount,
       count(DISTINCT target.id) AS uniqueCounterparties
```

**Ring detection (circular transaction patterns):**
```cypher
// Detect circular flows (money laundering)
MATCH path = (a:Account)-[:SENT_TO*3..6]->(a)
WHERE all(r IN relationships(path) WHERE r.amount > 10000)
RETURN path, length(path) AS cycleLength
```

**Capabilities:**
- **Ring detection** — circular patterns in transaction networks (AML, trade-based laundering)
- **Identity resolution** — entity resolution across customer data (synthetic identity detection)
- **Money laundering detection** — transaction flow analysis, beneficial ownership graphs (UBO chains)
- **Credit risk analysis** — borrower network analysis, connected exposure
- **Real-time scoring** — transaction graph queries in sub-second for authorization decisions

### 5.2 Knowledge Graphs

Enterprise knowledge graphs unify data across silos for semantic search, metadata management (data lineage, catalog), ontology management, and Q&A over enterprise knowledge.

### 5.3 Recommendation Engines

**Collaborative filtering** — "customers who bought this also bought":
```cypher
MATCH (p:Product {id: $pid})<-[:PURCHASED]-(u:User)-[:PURCHASED]->(rec:Product)
WHERE rec.id <> $pid RETURN rec.id, count(*) AS freq ORDER BY freq DESC LIMIT 10
```
**Hybrid** — collaborative + content-based can also be expressed in a single Cypher traversal.

### 5.4 Network IT, Supply Chain, Life Sciences

| Domain | Use Cases |
|--------|-----------|
| **IT/Network** | Topology management, asset dependency mapping, cloud infra management, root cause analysis |
| **Supply Chain** | Bill of materials, supplier network analysis, logistics optimization, inventory, impact analysis |
| **Life Sciences** | Drug discovery (protein interactions), patient journey analysis, clinical trials, disease pathways |

### 5.7 GenAI and LLM

See [Section 7](#7-neo4j-for-genai-and-llm) for detailed coverage of Neo4j's role in the GenAI ecosystem.

---

## 6. Graph Data Science (GDS) Library

### 6.1 Overview

The **Neo4j Graph Data Science (GDS) library** provides 50+ graph algorithms optimized for in-database execution. It runs algorithms directly on the stored graph without exporting data to external processing frameworks.

| Aspect | Community Edition | Enterprise Edition |
|--------|-----------------|-------------------|
| **Algorithms** | Limited subset | Full 50+ algorithms |
| **Concurrency** | Limited | Configurable parallel execution |
| **Pipeline builder** | Not available | Available (GraphSAGE, Node2Vec pipelines) |
| **Arrow Flight** | Not available | Available (fast graph export) |
| **Tiered storage** | Not available | Available |
| **Multi-graph** | Single graph | Multiple named graphs |

### 6.2 Algorithm Categories

| Category | Algorithms | Use Case |
|----------|-----------|----------|
| **Centrality** | PageRank, Betweenness, Closeness, Degree, Eigenvector, ArticleRank, Harmonic | Find important nodes (influencers, hubs, key accounts) |
| **Community Detection** | Louvain, Leiden, Label Propagation, Weakly Connected Components, Mutually Connected Components, Triangle Counting, K-1 Coloring | Find clusters (fraud rings, customer segments, topic communities) |
| **Path Finding** | Shortest Path, All Pairs Shortest Path, K Shortest Paths (Yen's), A*, Minimum Weight Spanning Tree, DijkstraSourceTarget | Find optimal routes (logistics, network routing) |
| **Node Embeddings** | Node2Vec, FastRP, GraphSAGE, Weisfeiler-Lehman, HashGNN | Generate dense vector representations for ML |
| **Similarity** | Node Similarity, K-Nearest Neighbors (KNN), Cosine, Pearson, Jaccard, Overlap | Find similar nodes (recommendations, dedup) |
| **Topology** | DijkstraSourceTarget, RandomWalk | Structural analysis |

### 6.3 GDS Workflow

The standard GDS pipeline follows these steps:

1. **Project graph** — Load subgraph into memory: `CALL gds.graph.project('myGraph', 'Account', 'SENT_TO', {relationshipProperties: ['amount', 'timestamp']})`
2. **Run algorithm** — Execute on projected graph: `CALL gds.pageRank.stream('myGraph') YIELD nodeId, score RETURN gds.util.asNode(nodeId).id AS accountId, score ORDER BY score DESC LIMIT 100`
3. **Write results** — Persist as node properties: `CALL gds.pageRank.write('myGraph', {writeProperty: 'pagerank'})`
4. **Mutate** — Apply to projected graph without DB write: `CALL gds.pageRank.mutate('myGraph', {mutateProperty: 'pagerank'})`
5. **Train** — ML model training (Enterprise): `CALL gds.beta.pipeline.nodeClassification.train('myGraph', {pipeline: 'myPipeline', targetNodeLabels: ['Account'], modelName: 'fraudModel'})`

### 6.4 GDS Output Modes

| Mode | Behavior | Best For |
|------|----------|----------|
| `stream()` | Returns results as rows | Analysis, visualization, export |
| `write()` | Writes property to Neo4j graph | Persistent storage for later queries |
| `mutate()` | Writes to projected in-memory graph (no DB write) | Intermediate steps in pipelines |
| `train()` | Trains an ML model (GraphSAGE, Node2Vec) | Production ML pipelines |

### 6.5 GDS and GenAI

- **Node embeddings for RAG** — Use FastRP or GraphSAGE embeddings in a vector index for graph-aware RAG
- **Graph features for LLM context** — PageRank, community membership, centrality as graph context signals
- **GraphSAGE for inductive learning** — Predict properties on new nodes without retraining
- **HashGNN** — Deep learning node embeddings with GNN architecture, optimized for GDS

### 6.6 GDS Enterprise Features

- **Arrow Flight integration** — Columnar data export at millions of rows/second for external ML frameworks
- **Tiered Storage** — Transparent migration of older/cold edges to slower storage, keeping hot graph in memory
- **Multi-graph support** — Multiple named projected graphs for different workloads
- **Pipeline framework** — Train/test split, feature extraction, algorithm chaining for graph ML
- **Workload management** — Configure concurrency per algorithm, queue management

---

## 7. Neo4j for GenAI and LLM

### 7.1 Vector Search

Neo4j 5.x+ includes native vector index support using **HNSW (Hierarchical Navigable Small World)** indexes:

```cypher
// Create a vector index
CREATE VECTOR INDEX moviePlots IF NOT EXISTS
FOR (m:Movie) ON (m.plotEmbedding)
OPTIONS {indexConfig: {
  `vector.dimensions`: 1536,
  `vector.similarity_function`: 'cosine'
}}

// Query vector index
MATCH (m:Movie)
WHERE m.plotEmbedding IS NOT NULL
WITH m, vector.similarity.cosine(m.plotEmbedding, $queryEmbedding) AS score
ORDER BY score DESC LIMIT 10
RETURN m.title, score
```

**Vector index features:**
- Supported similarity functions: cosine, euclidean, dot
- Hybrid search: combine vector similarity with graph filters (`WHERE m.genre = 'Action'`)
- HNSW parameters: dimensions (1–4096), neighbors, min/max connections

### 7.2 LangChain Integration

The **langchain-neo4j** package provides deep integration:

| Integration | Capability |
|-------------|------------|
| `Neo4jVector` | Full CRUD vector store, similarity search, MMR, metadata filtering |
| `LLMGraphTransformer` | Extract entities and relationships from text → construct knowledge graph |
| `Neo4jDocumentGraph` | Store document chunks with rich graph metadata |
| `GraphCypherQAChain` | Natural language → Cypher query → execution → answer synthesis |
| **GraphRAG** | Hybrid retrieval: combine vector search with graph traversal context |
| **Checkpoint Saver** | LangGraph state persistence in Neo4j (time-travel, branching) |

**Example: GraphCypherQAChain**
```python
from langchain_neo4j import GraphCypherQAChain, Neo4jGraph

graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="password")
chain = GraphCypherQAChain.from_llm(
    llm=ChatOpenAI(model="gpt-4"),
    graph=graph,
    verbose=True,
    allow_dangerous_requests=True
)
chain.invoke({"query": "What accounts did Customer CUST001 transact with in January 2024?"})
```

### 7.3 LlamaIndex Integration

The **llama-index-graph-neo4j** package provides:

| Integration | Capability |
|-------------|------------|
| `Neo4jVectorIndex` | Embeddings storage, similarity search, metadata filtering |
| `KnowledgeGraphIndex` | Extract entities/relationships into Neo4j KG |
| `PropertyGraphIndex` | Modern property graph index with Neo4jPropertyGraphStore |
| `KnowledgeGraphRAGRetriever` | Graph-aware retrieval for RAG |

### 7.4 GraphRAG

**GraphRAG** — retrieval augmented generation with graph context — combines vector similarity search with graph traversal to provide richer context to LLMs. A simplified flow:

```
User Query → Hybrid Retrieval (Vector Search + Graph Traversal + Graph Algorithms) → LLM Prompt (documents + graph context) → Final Answer
```

**GraphRAG advantages over pure vector RAG:**
- **Multi-hop reasoning** — graph traversal connects entities that are semantically distant but structurally close
- **Relationship context** — not just what entities are similar, but HOW they are connected
- **Aggregated context** — graph algorithms summarize neighborhoods (community membership, centrality)
- **Fact verification** — cross-reference LLM claims against ground-truth graph data

### 7.5 LLM Integration Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| **Text2Cypher** | LLM generates Cypher queries from natural language | "Show me all high-value transactions in January" → `MATCH (t:Transaction) WHERE ...` |
| **LLM summarization** | LLM summarizes graph query results for end users | Graph returns 50 transactions → LLM produces "Your top 3 transactions this month..." |
| **Entity extraction** | LLM extracts entities/relationships from documents → populates KG | `LLMGraphTransformer` in LangChain |
| **Conversational graph** | LLM drives interactive graph exploration | "Who are the suppliers of Acme Corp?" → traverse → "Show me their risk ratings" |
| **GraphRAG** | Graph context + vector retrieval → better RAG | Financial report RAG with account ownership context |
| **MCP tools** | LLM agents access Neo4j via Model Context Protocol | Agent queries KG for fact-checking, entity resolution |

### 7.6 Recent Developments

- **Vector + graph hybrid search** for RAG — combining semantic similarity with graph connectivity for substantially better retrieval relevance
- **Cypher generation with LLMs** — models increasingly capable of generating correct Cypher from schema context; fine-tuned models emerging
- **Graph-based fact checking** — verify LLM claims against a knowledge graph to reduce hallucination
- **LLM agents with Neo4j tools** — MCP servers providing Neo4j read/write access to agent frameworks

---

## 8. Ecosystem and Tools

### 8.1 Drivers and Client Libraries

**Official drivers** (Bolt protocol):

| Language | Package | Use Case |
|----------|---------|----------|
| Java | `neo4j-java-driver` | JVM apps, Spring Boot |
| JavaScript/TypeScript | `neo4j-driver` | Node.js, browser (with caution) |
| Python | `neo4j` | Data science, ML, web apps |
| .NET/C# | `Neo4j.Driver` | .NET applications |
| Go | `neo4j-go-driver` | Go services, microservices |
| C/C++ | `neo4j-cpp-driver` | High-performance systems |

**Community drivers:** Rust (`neo4j-rs`), Ruby, PHP, R, Elixir (`bolt_sips`), Haskell, Clojure, Julia, Erlang, Dart, Swift.

### 8.2 Connectors

| Connector | Purpose |
|-----------|---------|
| **Neo4j Connector for Apache Spark** | Read/write graph data from Spark jobs; DataFrame integration |
| **Neo4j Kafka Connector** | Stream data into Neo4j from Kafka topics (source + sink) |
| **Neo4j Connector for BI** | ODBC/JDBC interface for BI tools (Tableau, Power BI, Looker) |

### 8.3 Development Tools

| Tool | Purpose | Model |
|------|---------|-------|
| **Neo4j Browser** | Web-based query editor, visualization, schema explorer, built-in guides | Free (web app, bundled) |
| **Neo4j Bloom** | No-code graph visualization, natural language search, perspective-based exploration | Licensed (Enterprise/AuraDB Enterprise) |
| **Neo4j Workspace** | Data modeling, import, query development, visualization, collaboration | Cloud-based (AuraDB) |
| **Neo4j Console** | Free online sandbox, pre-loaded datasets, instant cloud instance | Free |
| **Neo4j Desktop** | Local dev environment, manage multiple DB instances, plugin ecosystem, graph apps | Free |

### 8.4 Integration Tools

**APOC (Awesome Procedures On Cypher)** — standard library of 450+ procedures and functions for data integration, conversion, and graph operations:

```cypher
CALL apoc.load.json('https://api.example.com/data') YIELD value CREATE (:Data {payload: value})
CALL apoc.load.csv('data.csv') YIELD list CREATE (:Row {id: list[0]})
CALL apoc.periodic.iterate('MATCH (t:Transaction) RETURN t', 'SET t.flagged = true', {batchSize:10000})
```
| APOC Category | Examples |
|--------------|----------|
| **Data integration** | `apoc.load.json`, `apoc.load.csv`, `apoc.load.xml`, `apoc.load.jdbc` |
| **Graph operations** | `apoc.periodic.iterate`, `apoc.create.node`, `apoc.create.relationship` |
| **Data conversion** | `apoc.convert.toJson`, `apoc.convert.fromJsonMap` |
| **Utility** | `apoc.map.merge`, `apoc.text.*`, `apoc.date.*` |
| **Triggers** | `apoc.trigger.*` — reactive procedures on data changes |

**GraphQL — Neo4j GraphQL Library:** Generate a GraphQL API from your Neo4j schema using `@neo4j/graphql` for Node.js or `neo4j-graphql-java`. Define types with `@relationship` directives — the library auto-generates CRUD queries, mutations, and resolvers.

**Other integration tools:** Spring Data Neo4j (SDN6 — repository pattern, `@Node` annotations), Cypher DSL (type-safe Java query builder), Neo4j ETL Tool (relational DB extraction), Neo4j Data Importer (CSV/JSON UI).

### 8.5 Enterprise Tools

| Tool | Purpose |
|------|---------|
| **Neo4j Ops Manager** | Backup, restore, monitoring, cluster management |
| **Neo4j ETL Tool** | Extract data from relational databases (JDBC) into Neo4j |
| **Neo4j Data Importer** | UI-based CSV/JSON import with mapping configuration |
| **Neo4j Connector for BI** | ODBC/JDBC bridge for Tableau, Power BI, Looker |

### 8.6 GenAI Tools

| Tool/Integration | Description |
|-----------------|-------------|
| **langchain-neo4j** | LangChain integration package (vector store, GraphCypherQAChain, LLMGraphTransformer) |
| **llama-index-graph-neo4j** | LlamaIndex integration (vector index, KnowledgeGraphIndex, PropertyGraphIndex) |
| **Spring AI Neo4j Store** | Spring AI vector store for Java/Spring ecosystem |
| **Neo4j + ChatGPT Plugins** | Connect ChatGPT to Neo4j for conversational graph querying |
| **MCP Server for Neo4j** | Model Context Protocol server exposing Neo4j read/write tools to LLM agents |

---

## 9. Getting Started Checklist

### Step 1: Install or Create Instance

- [ ] **Option A:** Download and install [Neo4j Desktop](https://neo4j.com/download/) for local development
- [ ] **Option B:** Create a free [AuraDB instance](https://console.neo4j.io/) (no install, cloud-hosted)
- [ ] **Option C:** Try the [Neo4j Sandbox](https://sandbox.neo4j.com/) with pre-loaded datasets

### Step 2: Learn Cypher Basics

- [ ] Complete the built-in guides in **Neo4j Browser** (type `:play start`)
- [ ] Work through the **Movie Graph** tutorial — create nodes, MATCH patterns, traverse relationships
- [ ] Practice filtering (`WHERE`), path matching (`*1..3`), aggregation (`COUNT`, `COLLECT`)

### Step 3: Practice with Sample Datasets

- [ ] **Northwind** — classic RDBMS-to-graph migration (products, categories, suppliers, orders)
- [ ] **Movie Recommendations** — user-movie ratings graph with collaborative filtering queries
- [ ] **Flight Routes** — airports, routes, airlines with shortest-path queries
- [ ] **Financial Transactions** — accounts, transactions, fraud pattern queries

### Step 4: Model Your Domain

- [ ] Sketch your domain as nodes (entities) and relationships (connections)
- [ ] Define labels (`:Customer`, `:Account`, `:Transaction`, `:Party`)
- [ ] Define relationship types (`:OWNS`, `:SENT_TO`, `:RELATED_TO`)
- [ ] Identify properties on both nodes and relationships
- [ ] Load real data using `LOAD CSV` or APOC's `apoc.load.json`

### Step 5: Add Indexes and Constraints

- [ ] Add **unique constraints** on entity identifiers: `CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.id IS UNIQUE`
- [ ] Add **indexes** on frequently filtered properties: `CREATE INDEX FOR (t:Transaction) ON (t.date)`
- [ ] Add **existence constraints** on required fields: `CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.name IS NOT NULL`
- [ ] Verify indexes are `ONLINE`: `SHOW INDEXES`

### Step 6: Back Up Your Database

- [ ] Configure automated backups (AuraDB: built-in; self-managed: cron + `neo4j-admin backup`)
- [ ] Test restore procedure
- [ ] Set retention policy for transaction logs

### Step 7: Deploy for Production

- [ ] **Learning/dev:** Neo4j Desktop or AuraDB free tier
- [ ] **Production with ops team:** Self-managed Enterprise cluster (3+ nodes)
- [ ] **Production without ops:** AuraDB Professional or Enterprise
- [ ] **Regulated industry:** AuraDB Enterprise (SOC 2, HIPAA, PCI DSS) or self-managed with RBAC/LDAP/encryption

### Step 8: Explore Graph Data Science

- [ ] Install GDS plugin: `CALL gds.list()` to verify installation
- [ ] Run PageRank to find influential entities: `CALL gds.pageRank.stream('myGraph')`
- [ ] Run community detection to find clusters: `CALL gds.louvain.stream('myGraph')`
- [ ] Explore node embeddings with FastRP or Node2Vec

### Step 9: Integrate with Applications

- [ ] Install the official driver for your language (`pip install neo4j`, `npm install neo4j-driver`)
- [ ] Write a simple CRUD app (connect, run query, process results)
- [ ] Test connection pooling and configure query logging

### Step 10: Explore GenAI Integration

- [ ] **Vector search:** Create a vector index, generate embeddings, run similarity search
- [ ] **GraphCypherQA:** Try `GraphCypherQAChain` from `langchain-neo4j`
- [ ] **GraphRAG:** Implement hybrid retrieval combining vector search + graph traversal
- [ ] **Knowledge graph from text:** Use `LLMGraphTransformer` to extract entities from documents
- [ ] **MCP server:** Set up a Neo4j MCP server for agent-based graph querying

---

## References

- [Neo4j Cypher Manual](https://neo4j.com/docs/cypher-manual/current/) — Official Cypher documentation and reference
- [Neo4j Operations Manual](https://neo4j.com/docs/operations-manual/current/) — Configuration, clustering, performance tuning
- [Neo4j Graph Data Science Manual](https://neo4j.com/docs/graph-data-science/current/) — GDS algorithms, pipelines, ML workflows
- [Neo4j AuraDB Documentation](https://neo4j.com/docs/aura/) — Managed cloud service reference
- [APOC Library](https://neo4j.com/docs/apoc/current/) — Standard procedures and functions
- [Neo4j GraphQL Library](https://neo4j.com/docs/graphql/current/) — GraphQL-to-Cypher compiler
- [LangChain Neo4j Integration](https://python.langchain.com/docs/integrations/vectorstores/neo4jvector/) — Vector store, GraphCypherQA, LLMGraphTransformer
- [LlamaIndex Neo4j Integration](https://llamahub.ai/l/neo4j) — Vector store, property graph index
- [Neo4j GenAI Documentation](https://neo4j.com/docs/cypher-manual/current/indexes-for-vector-search/) — Vector indexes, hybrid search
- *Graph Databases* by Robinson, Webber, Eifrem — O'Reilly Media (canonical reference)
- [Neo4j Developer Blog](https://neo4j.com/blog/) — Case studies, guides, community contributions

---

## Related Guides in This Repository

| Guide | Path | Focus |
|-------|------|-------|
| Graph Database Concepts & Neo4j | `technology/neo4j_graph_database_guide.md` | Data modeling, indexing, query optimization, performance tuning |
| Neo4j Bloom Alternatives | `technology/neo4j_bloom_alternatives_guide.md` | Graph visualization tools beyond Bloom |
| Graph Engineering | `technology/graph_engineering_guide.md` | Graph systems, pipelines, ML engineering |
| LLM Frameworks Comparison | `technology/ai_llm/llm_frameworks_comparison_guide.md` | LangChain, LlamaIndex, Spring AI Neo4j integration depth |

---

*This guide is maintained by Jack Liu Shurui. For corrections, open an issue or PR on [github.com/jackliusr/research](https://github.com/jackliusr/research).*
