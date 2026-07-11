# Massively Parallel Processing (MPP) Databases — A Comprehensive Guide

> **Author**: Jack Liu Shurui  
> **Last Updated**: July 2026  
> **Part of**: research/ — standalone reference on MPP database architecture, systems, and practical usage

---

## Table of Contents

1. [What is MPP?](#1-what-is-mpp)
2. [Core Architecture](#2-core-architecture)
3. [Key Concepts](#3-key-concepts)
   - 3.1 [Data Distribution (Partitioning)](#31-data-distribution-partitioning)
   - 3.2 [Query Execution](#32-query-execution)
   - 3.3 [Key Design Decisions](#33-key-design-decisions)
4. [Major MPP Databases](#4-major-mpp-databases)
   - 4.1 [Greenplum](#41-greenplum)
   - 4.2 [Amazon Redshift](#42-amazon-redshift)
   - 4.3 [Snowflake](#43-snowflake)
   - 4.4 [Apache Doris](#44-apache-doris)
   - 4.5 [ClickHouse](#45-clickhouse)
   - 4.6 [Google BigQuery](#46-google-bigquery)
5. [Comparison Table](#5-comparison-table)
6. [Distribution Key Selection](#6-distribution-key-selection)
7. [When to Use MPP vs Alternatives](#7-when-to-use-mpp-vs-alternatives)
8. [Performance Optimization](#8-performance-optimization)
9. [Data Loading Patterns](#9-data-loading-patterns)
10. [Future Trends](#10-future-trends)
11. [Data Sources](#11-data-sources)

---

## 1. What is MPP?

### 1.1 Definition

**Massively Parallel Processing (MPP)** is a coordinated computing architecture in which many independent processors or nodes simultaneously work on different parts of the same problem. In the database context, an MPP system splits a large dataset across multiple nodes and executes queries in parallel across all of them, then assembles the results into a single response.

The core principle is simple but powerful: **divide the data, divide the work, and combine the results**. By partitioning data across dozens, hundreds, or even thousands of nodes, an MPP database can process terabytes to petabytes of data using commodity hardware.

### 1.2 Shared-Nothing Architecture

MPP databases are built on a **shared-nothing architecture**:

- **Each node** has its own dedicated CPU, memory, and local disk storage
- **No resource sharing** between nodes — no shared memory, no shared disk
- **Communication** happens only over the network interconnect (InfiniBand, high-speed Ethernet)
- **Failure isolation** — a failed node affects only its data partition; the cluster can continue operating

This contrasts sharply with traditional SMP (Symmetric Multi-Processing) systems where all processors share a common memory bus and disk array.

```
┌──────────────────────────────────────────────────────┐
│                  Shared-Nothing MPP                    │
├────────────┬────────────┬────────────┬───────────────┤
│   Node 1   │   Node 2   │   Node 3   │    Node N     │
│  ┌──────┐  │  ┌──────┐  │  ┌──────┐  │  ┌──────┐     │
│  │ CPU  │  │  │ CPU  │  │  │ CPU  │  │  │ CPU  │     │
│  ├──────┤  │  ├──────┤  │  ├──────┤  │  ├──────┤     │
│  │ RAM  │  │  │ RAM  │  │  │ RAM  │  │  │ RAM  │     │
│  ├──────┤  │  ├──────┤  │  ├──────┤  │  ├──────┤     │
│  │ DISK │  │  │ DISK │  │  │ DISK │  │  │ DISK │     │
│  └──────┘  │  └──────┘  │  └──────┘  │  └──────┘     │
└─────┬──────┴─────┬──────┴─────┬──────┴───────┬───────┘
      │            │            │              │
      └────────────┴────────────┴──────────────┘
                 Network Interconnect
```

### 1.3 MPP vs SMP

| Dimension | SMP | MPP |
|-----------|-----|-----|
| **Memory** | Shared across all CPUs | Each node has private memory |
| **Disk** | Shared storage (SAN/NAS) | Local disks per node |
| **Scaling** | Vertical (add more CPUs to the same box) | Horizontal (add more nodes) |
| **Cost** | Expensive proprietary hardware | Commodity hardware |
| **Max scale** | ~64–256 CPUs typically | Thousands of nodes |
| **Fault tolerance** | Single system — full outage on failure | Node failure impacts only its partition |
| **Example DBs** | Oracle RAC, SQL Server | Greenplum, Redshift, ClickHouse |

SMP systems hit a scalability wall because the shared memory bus and disk I/O become bottlenecks. MPP eliminates these by giving each node its own resources — but pays the cost of network communication for data shuffling.

### 1.4 MPP vs Distributed SQL Databases

Distributed SQL databases (CockroachDB, YugabyteDB, TiDB) are often confused with MPP databases, but they serve fundamentally different workloads:

| Dimension | MPP Database | Distributed SQL Database |
|-----------|-------------|------------------------|
| **Primary workload** | OLAP (analytics) | OLTP (transactions) |
| **Query pattern** | Large scans, aggregations, multi-table joins | Point lookups, short transactions, index scans |
| **Data distribution** | Hash/range across all nodes, optimized for scan | Range-based or hash sharding, optimized for locality |
| **Consistency model** | Typically read-committed or snapshot isolation | Strong consistency (serializable, linearizable) |
| **Transactions** | Bulk operations, limited multi-statement transactions | Full ACID, distributed transactions (2PC, Raft) |
| **Node coordination** | Coordinator node (single point for query planning) | Peer-to-peer (Raft/Paxos consensus) |
| **Storage** | Columnar or hybrid (optimized for compression) | Row-oriented (optimized for single-row access) |

**The key distinction**: MPP databases are designed to **scan and aggregate large amounts of data quickly**. Distributed SQL databases are designed to **serve many concurrent small transactions with strong consistency guarantees**. Trying to run OLTP on Greenplum or Redshift would be inefficient; trying to run petabyte-scale analytics on CockroachDB would be impractical.

### 1.5 How MPP Works at a High Level

```
User submits SQL query
        │
        ▼
  ┌─────────────────┐
  │   Coordinator   │  Parse → Optimize → Plan → Distribute
  │   (Leader Node) │
  └────────┬────────┘
           │
   ┌───────┴───────┐
   ▼               ▼
┌────────┐   ┌────────┐      ┌────────┐
│ Node 1 │   │ Node 2 │  ... │ Node N │  ← Each reads its local partition
│ Scan   │   │ Scan   │      │ Scan   │
│ Filter │   │ Filter │      │ Filter │
│ Agg    │   │ Agg    │      │ Agg    │
└───┬────┘   └───┬────┘      └───┬────┘
    │            │               │
    └────────────┴───────────────┘
                 │
            Shuffle / Exchange
                 │
    ┌────────────┴───────────────┐
    │        Coordinator         │  Gather → Merge → Sort → Return
    └────────────────────────────┘
                 │
                 ▼
           Result Set
```

---

## 2. Core Architecture

### 2.1 Architecture Diagram

```
                     ┌─────────────────────┐
                     │  Client / BI Tool    │
                     │  (Tableau, Power BI, │
                     │   JDBC/ODBC client)  │
                     └──────────┬──────────┘
                                │
                     ┌──────────▼──────────┐
                     │ Query Coordinator    │
                     │ (Frontend / Leader)  │
                     │                      │
                     │ • SQL Parsing        │
                     │ • Query Optimization │
                     │ • Plan Distribution  │
                     │ • Result Aggregation │
                     └──────────┬──────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
  │   Compute      │   │   Compute      │   │   Compute      │
  │   Node 1       │   │   Node 2       │   │   Node N       │
  │                │   │                │   │                │
  │  ┌──────────┐  │   │  ┌──────────┐  │   │  ┌──────────┐  │
  │  │ CPU Cores│  │   │  │ CPU Cores│  │   │  │ CPU Cores│  │
  │  ├──────────┤  │   │  ├──────────┤  │   │  ├──────────┤  │
  │  │ Memory   │  │   │  │ Memory   │  │   │  │ Memory   │  │
  │  │ (RAM)    │  │   │  │ (RAM)    │  │   │  │ (RAM)    │  │
  │  ├──────────┤  │   │  ├──────────┤  │   │  ├──────────┤  │
  │  │ Local    │  │   │  │ Local    │  │   │  │ Local    │  │
  │  │ SSD/NVMe │  │   │  │ SSD/NVMe │  │   │  │ SSD/NVMe │  │
  │  └──────────┘  │   │  └──────────┘  │   │  └──────────┘  │
  └───────┬────────┘   └───────┬────────┘   └───────┬────────┘
          │                    │                     │
          └────────────────────┼─────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Shared Network     │
                    │  (InfiniBand / 25GbE │
                    │   / 100GbE / RoCE)   │
                    └─────────────────────┘
```

### 2.2 Component Breakdown

#### Coordinator Node (Leader / Frontend)

The coordinator is the brain of the MPP system. It:

1. **Parses SQL** — validates syntax, resolves table/column references
2. **Optimizes queries** — builds an execution plan, choosing join order, aggregation strategies, and data movement patterns
3. **Distributes work** — slices the plan into fragments and dispatches them to compute nodes
4. **Monitors execution** — tracks progress, handles failures and retries
5. **Aggregates results** — collects partial results from compute nodes, applies final sorting/aggregation, and returns the result set to the client

The coordinator itself typically does not store user data; it is a stateless or lightly-stateful routing and planning engine. In Greenplum, the coordinator is a modified PostgreSQL process. In Redshift, it is the leader node. In BigQuery, this role is distributed across the Dremel tree's mixer nodes.

#### Compute Nodes (Segments / Workers / Slots)

Compute nodes do the actual data processing. Each compute node:

- Stores a portion of the dataset on local disk (in pure shared-nothing)
- Executes query fragments — scans local data, applies filters, performs local aggregations
- Participates in data shuffling — sends/receives intermediate results to/from other nodes during joins and aggregations
- Reports results back to the coordinator

The number of compute nodes determines the degree of parallelism. More nodes means faster scans (up to the point where data per node is small enough that overhead dominates).

#### Interconnect

The interconnect is the high-speed network binding the cluster together. Since MPP databases shuffle large volumes of data between nodes during query execution, interconnect performance is critical:

- **InfiniBand** — Low latency, high bandwidth (up to 400 Gbps HDR), RDMA support. Common in on-premise Greenplum deployments
- **25/100/400 GbE** — Ethernet with RoCE (RDMA over Converged Ethernet) for RDMA-like performance
- **NVLink / NVSwitch** — GPU-direct interconnects gaining traction for GPU-accelerated MPP

The interconnect is often the first bottleneck in a healthy MPP cluster. A fast interconnect can make a 10-node cluster perform like 8 nodes; a slow one can make it perform like 3.

#### Storage Layer

Storage varies by architecture generation:

| Generation | Storage Architecture | Examples |
|------------|---------------------|----------|
| **1st Gen** (Pure shared-nothing) | Local SSDs/NVMe per node, data tied to physical node | Greenplum (classic), Redshift (DC/DS nodes) |
| **2nd Gen** (Hybrid) | Local cache + managed cloud storage, compute nodes read from local cache when possible | Redshift RA3, Snowflake (shared storage model) |
| **3rd Gen** (Fully separated) | Object store (S3/GCS/Azure Blob) with no local data affinity | Snowflake, BigQuery, Redshift Spectrum |

Second and third generation architectures trade some scan performance (local NVMe is faster than S3) for operational flexibility — you can scale compute independently of storage and nodes can be added/removed without data rebalancing.

---

## 3. Key Concepts

### 3.1 Data Distribution (Partitioning)

How data is spread across nodes is the single most important design decision in an MPP database. Choose wrong, and a 100-node cluster can perform like a 3-node one.

#### Hash Distribution

Data is distributed by computing a hash of the distribution key column(s) and mapping the hash to a node.

```sql
-- Greenplum syntax
CREATE TABLE orders (
    order_id BIGINT,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(12,2)
)
DISTRIBUTED BY (customer_id);
```

**Pros**: Even distribution for high-cardinality keys. Natural colocation for joins on the same key.  
**Cons**: Data skew if the key has low cardinality or uneven value distribution.  
**Best for**: Fact tables joined on a common key (e.g., `customer_id`, `product_id`).

**How the hash function works** (simplified):

```
hash(customer_id) → 42
42 % num_nodes → Node 3
```

Greenplum uses `hash_uint32()` internally. Redshift uses an internal hash function for `DISTKEY`. The hash modulus maps to a segment or slice.

#### Random Distribution (Round-Robin / EVEN)

Data is distributed sequentially across nodes without any hash or key.

```sql
-- Redshift syntax
CREATE TABLE staging_events (
    raw_line VARCHAR(65535)
)
DISTSTYLE EVEN;

-- Greenplum syntax
CREATE TABLE staging_events (
    raw_line TEXT
)
DISTRIBUTED RANDOMLY;
```

**Pros**: Guaranteed even distribution. No data skew. Good for staging/landing tables where distribution key is not yet known.  
**Cons**: Joins require data shuffling. No data locality for any query pattern.  
**Best for**: Temporary tables, staging areas, ETL landing zones.

#### Replicated Distribution

The entire table is copied to every node.

```sql
-- Greenplum
CREATE TABLE dim_customer (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    segment VARCHAR(50)
)
DISTRIBUTED REPLICATED;

-- Redshift
CREATE TABLE dim_customer (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    segment VARCHAR(50)
)
DISTSTYLE ALL;
```

**Pros**: Joins on replicated tables never require data shuffling — every node already has the full table. Dramatically improves join performance for small dimension tables.  
**Cons**: Storage cost multiplies by number of nodes. Table size limited by per-node memory. UPDATE/DELETE must propagate to all nodes.  
**Best for**: Small dimension tables in star schemas (typically < 10 million rows or < 10 GB compressed).

**Rule of thumb**: A replicated table should fit in memory on the smallest node. If the table is too large to replicate, use hash distribution on the join key instead.

#### Range Distribution

Data is split by value ranges of the distribution key.

```sql
-- Conceptual (varies by system: often via partition + sub-partition)
CREATE TABLE events (
    event_id BIGINT,
    event_timestamp TIMESTAMP,
    event_data JSONB,
    user_id INT
)
DISTRIBUTED BY (event_timestamp)
-- Ranges: Jan 2025 → Node 1, Feb 2025 → Node 2, etc.
```

**Pros**: Natural for time-series data. Enables partition pruning at the distribution level. Good for range scans.  
**Cons**: Can cause hot spots if new data concentrates on a single node (e.g., today's data all goes to one node). Data skew is common.  
**Best for**: Time-series data with evenly distributed write patterns. Less common in pure MPP — more common in systems like ClickHouse (shard key) or Apache Doris (partition + bucket).

### 3.2 Query Execution

Understanding how a query flows through an MPP system is essential for optimization.

#### Execution Phases

**Phase 1: Parse and Plan**
The coordinator parses SQL and builds a distributed execution plan. The plan is a DAG (Directed Acyclic Graph) of operations:

```
SQL: SELECT d.segment, SUM(s.amount)
     FROM sales s JOIN dim_customer d ON s.customer_id = d.customer_id
     WHERE s.sale_date >= '2025-01-01'
     GROUP BY d.segment
     ORDER BY SUM(s.amount) DESC
     LIMIT 10;

Plan (simplified):
1. Scan sales (all nodes, local date filter)
2. Scan dim_customer (broadcast or local, depending on distribution)
3. HashJoin on customer_id
4. Partial Aggregation (GROUP BY segment)
5. Shuffle by segment hash
6. Final Aggregation
7. Sort + Limit
8. Gather to coordinator → Return
```

**Phase 2: Scan**
Each compute node reads its local data partition in parallel. For columnar storage, only the columns referenced in the query are read. Zone maps and min/max indexes filter out irrelevant row groups at this stage.

```
Node 1: Reads 500,000 rows from sales where sale_date >= '2025-01-01'
Node 2: Reads 480,000 rows from sales where sale_date >= '2025-01-01'
Node 3: Reads 510,000 rows from sales where sale_date >= '2025-01-01'
```

**Phase 3: Local Processing**
Each node applies filters, performs local joins (if all join data is local), and does partial aggregation. This reduces the data volume before the shuffle.

```
Node 1: After JOIN + GROUP BY → 15 groups, partial SUM for each
Node 2: After JOIN + GROUP BY → 12 groups
Node 3: After JOIN + GROUP BY → 18 groups
```

**Phase 4: Shuffle / Exchange**
Data is redistributed across nodes. This is the most expensive phase in terms of network I/O.

- For **joins**: Rows are redistributed so matching keys land on the same node
- For **aggregations**: Rows are redistributed by GROUP BY key
- For **sorting**: Rows are redistributed by sort key range

```
Each node sends/receives data for redistribution:
Node 1 → Node 2: 3 groups (segment='Consumer')
Node 1 → Node 3: 4 groups (segment='Corporate')
Node 2 → Node 1: 2 groups (segment='Home Office')
...
```

**Phase 5: Final Processing**
Each node performs its final computation on the redistributed data.

```
Node 1: Full SUM for 'Consumer' segment = $1.2M
Node 2: Full SUM for 'Corporate' segment = $3.4M
Node 3: Full SUM for 'Home Office' segment = $0.8M
```

**Phase 6: Gather and Return**
The coordinator collects all final results, applies global sort/limit, and returns the result set to the client.

```
Coordinator receives:
  Consumer: $1.2M
  Corporate: $3.4M
  Home Office: $0.8M

→ Sort DESC → Return top 10 (3 in this case)
```

#### Data Movement Types

| Movement Type | Description | When It Occurs |
|---------------|-------------|----------------|
| **Broadcast** | Send all rows from one table to all nodes | Small table joins (dimension broadcast), subquery in WHERE IN |
| **Redistribute (Shuffle)** | Hash-redistribute rows across nodes by a key | Joins where join keys differ from distribution keys, GROUP BY on non-distribution key |
| **Gather** | Send all rows from all nodes to the coordinator | ORDER BY with LIMIT, final aggregation |
| **Unicast** | Send rows from one node to a specific other node | Targeted lookups, certain subquery patterns |

### 3.3 Key Design Decisions

#### Distribution Key Selection

The distribution key is the single most impactful design decision. Get it wrong, and no amount of tuning will fix the performance.

**Criteria for a good distribution key**:

1. **High cardinality** — thousands to millions of distinct values (e.g., `customer_id`, `order_id`)
2. **Used in JOIN conditions** — ideally the same key used to distribute both fact and dimension tables
3. **Even value distribution** — no single value dominates (e.g., a `status` column with 99% 'completed' and 1% 'pending' is terrible)
4. **Stable** — values don't change frequently (UPDATE on distribution key requires data movement)

**Anti-patterns**:

- `gender` (2-3 values → data goes to 2-3 nodes)
- `year` alone (only as many values as years; terrible skew)
- Nullable columns with many NULLs (all NULLs hash to the same node)
- Surrogate keys not used in joins (data is evenly distributed but every join requires a shuffle)

#### Columnar Storage

Most MPP databases use columnar storage (or a hybrid row/column approach). In columnar storage, each column's values are stored contiguously, rather than storing each row contiguously.

```
Row-oriented:
Row 1: [1, 'Alice', 100.00, '2025-01-01']
Row 2: [2, 'Bob',   50.00, '2025-01-02']
Row 3: [3, 'Carol', 75.00, '2025-01-03']

Column-oriented:
id:       [1, 2, 3]
name:     ['Alice', 'Bob', 'Carol']
amount:   [100.00, 50.00, 75.00]
date:     ['2025-01-01', '2025-01-02', '2025-01-03']
```

**Benefits of columnar for MPP**:

- **Column pruning**: Only read columns referenced in the query. A `SELECT COUNT(*)` reads only the row count metadata, not the data.
- **Better compression**: Same-typed values stored together compress far better (run-length encoding, dictionary encoding, delta encoding).
- **SIMD-friendly**: Modern CPUs can process columnar data with SIMD instructions for vectorized scans.

#### Compression

Columnar storage enables much better compression than row storage because adjacent values are of the same type:

| Encoding | Best For | Compression Ratio | Notes |
|----------|----------|-------------------|-------|
| **Run-length (RLE)** | Low-cardinality columns (status, segment) | 10:1 to 100:1 | Stores value + count: `('Active', 50000), ('Inactive', 2000)` |
| **Dictionary** | Moderate-cardinality (strings, category codes) | 5:1 to 20:1 | Maps distinct values to integer codes |
| **Delta** | Sequential numbers, timestamps | 3:1 to 8:1 | Stores differences instead of absolute values |
| **LZO / LZ4 / Zstd** | General-purpose | 2:1 to 5:1 | Fast compression/decompression, good balance |
| **Gorilla** | Floating-point time-series | 5:1 to 15:1 | XOR-based, used in ClickHouse |

#### Indexing in MPP

MPP databases typically do not use traditional B-tree indexes (which are designed for point lookups in row-oriented OLTP).

Instead, they use:

- **Zone maps / Min-max indexes**: For each row group (typically ~1 million rows), store the min and max values of each column. Queries can skip entire row groups if the WHERE clause falls outside the range.
- **Block indexes / Bloom filters**: Used for equality predicates. Bloom filters quickly determine whether a value might exist in a block.
- **Local indexes**: Some MPP databases (Greenplum, ClickHouse) support local indexes per node — but they are not global across the cluster.
- **Sort keys**: Define the physical sort order of data on disk (see Section 8).

---

## 4. Major MPP Databases

### 4.1 Greenplum

**Overview**: Greenplum is one of the most mature open-source MPP databases, originating as a fork of PostgreSQL. It was acquired by EMC in 2010 and later spun off to VMware. In 2015, it was open-sourced under the Apache 2.0 license as the **Greenplum Database**.

**Architecture**:

```
[Coordinator] — PostgreSQL process, no user data
     │
     ├── [Segment 1] — Primary (active) + Mirror (standby)
     ├── [Segment 2] — Primary (active) + Mirror (standby)
     ├── [Segment 3] — Primary (active) + Mirror (standby)
     └── [Segment N] — Primary (active) + Mirror (standby)
     
     [Standby Coordinator] — Failover for coordinator
     [ETL hosts] — gpfdist, PXF for external data
```

- Each segment is a full PostgreSQL instance with its own data directory
- Segments are paired 1:1 with mirrors for HA at the segment level
- The coordinator holds catalog metadata but no user data
- Data is distributed hash/random/replicated across segments

**Key Features**:

| Feature | Details |
|---------|---------|
| **SQL Compatibility** | PostgreSQL-based — extensive SQL standard support, window functions, CTEs, PL/pgSQL |
| **Storage** | Heap (row), Append-Optimized (AO), Append-Optimized Columnar (AOCS) |
| **Partitioning** | Range + List partition, sub-partition |
| **External Tables** | gpfdist (parallel file load), PXF (Hadoop, S3, Hive, JDBC), Web external tables |
| **Extensions** | PostGIS, MADlib (in-database ML), GPORCA optimizer |
| **Data Loading** | gpfdist (parallel), gpload (declarative YAML-based), COPY |

**Ecosystem Tools**:

- **gpfdist**: Parallel file distribution server — run multiple instances for higher throughput
- **gpload**: YAML-based wrapper around gpfdist with transformation support
- **PXF**: Platform Extension Framework — query external data sources (HDFS, S3, Hive, HBase, JDBC)
- **GPORCA**: Modern cost-based query optimizer (replaces the legacy planner)
- **Madlib**: In-database machine learning library (regression, classification, clustering)
- **Greenplum Command Center**: Web-based monitoring and management console

**When to Use Greenplum**:

- Large-scale enterprise data warehousing (10TB to 10PB+)
- Organizations that need open-source flexibility with PostgreSQL ecosystem compatibility
- Hybrid transactional/analytical workloads (HTAP) — Greenplum supports concurrent reads and modest writes
- On-premise or private cloud deployments
- Financial services, telecom, and other regulated industries

**Limitations**:

- Coordinator is a single point of failure (mitigated by standby coordinator)
- Adding/removing segments requires data redistribution (can take hours for large datasets)
- PostgreSQL version lags behind upstream (Greenplum 7 is based on PostgreSQL 12)
- Requires careful distribution key selection — no automatic redistribution

**Version History**:

| Greenplum Version | PostgreSQL Base | Release Year |
|-------------------|----------------|--------------|
| 4.x | 8.2 | 2010 |
| 5.x | 8.3 | 2016 |
| 6.x | 9.4 | 2019 |
| 7.x | 12 | 2023 |

### 4.2 Amazon Redshift

**Overview**: Amazon Redshift is AWS's fully managed MPP data warehouse service. Launched in 2013, it was one of the first cloud-native MPP databases and remains one of the most widely deployed.

**Architecture**:

```
[Leader Node] — Coordinates query execution
     │
     ├── [Compute Node 1]
     │    ├── [Slice 0]
     │    ├── [Slice 1]
     │    └── [Slice N]
     ├── [Compute Node 2]
     │    └── ...
     └── [Compute Node N]
```

- **Leader Node**: SQL endpoint, query planning, result aggregation, does NOT store data
- **Compute Nodes**: Data storage + processing. Each node is divided into **slices** (typically 2-16 per node, depending on node type)
- **Slices**: The fundamental processing unit. Each slice has dedicated CPU, memory, and disk. The degree of parallelism equals the total number of slices
- **Node types**: DC2 (dense compute, local SSD), DS2 (dense storage, HDD), RA3 (managed storage, local SSD cache)

**Node Family Comparison**:

| Family | Storage | Compute-Storage Separation | Best For |
|--------|---------|---------------------------|----------|
| **DC2** | Local SSD (1.6 TB - 2.56 TB) | Tied | High-performance workloads |
| **DS2** | Local HDD (16 TB - 32 TB) | Tied | Large datasets, cost-sensitive |
| **RA3** | Managed storage (up to 64 PB) + local SSD cache | ✅ Separated | Flexible scaling, large datasets |

**Distribution Styles**:

```sql
-- KEY: Hash distribution on a column
CREATE TABLE sales (
    sale_id INT,
    customer_id INT DISTKEY,
    amount DECIMAL(12,2)
) SORTKEY(sale_date);

-- EVEN: Round-robin
CREATE TABLE staging (
    event_id INT,
    data VARCHAR(65535)
)
DISTSTYLE EVEN;

-- ALL: Replicated (Redshift term)
CREATE TABLE dim_customer (
    customer_id INT DISTKEY,
    name VARCHAR(255)
)
DISTSTYLE ALL;
```

**Sort Keys**:

```sql
-- Compound sort key (precedence-based)
CREATE TABLE orders (
    order_id BIGINT,
    customer_id INT,
    order_date DATE,
    status VARCHAR(20)
)
SORTKEY (order_date, status);
-- Best when WHERE clause often includes order_date

-- Interleaved sort key (equal importance)
CREATE TABLE logs (
    log_date DATE,
    severity VARCHAR(10),
    source VARCHAR(50),
    message VARCHAR(1024)
)
SORTKEY INTERLEAVED (log_date, severity, source);
-- Best when any column in the sort key may appear in WHERE
```

**Key Features**:

| Feature | Details |
|---------|---------|
| **Redshift Spectrum** | Query data directly in S3 via external tables — no loading required |
| **Auto WLM** | Automatic workload management — queues and prioritizes queries |
| **Concurrency Scaling** | Auto-provisions additional clusters for peak concurrency |
| **AQUA** | Advanced Query Accelerator — hardware-accelerated caching for certain node types |
| **RA3 Managed Storage** | Independent scaling of compute and storage |
| **Materialized Views** | Pre-computed results for complex aggregations |
| **Automatic Vacuum Delete** | Auto reclaims storage space from deleted rows |
| **Data Sharing** | Share live data across Redshift clusters (readers) |

**Limitations**:

- No support for `UPDATE`/`DELETE` on some distribution styles (ALL tables need special handling)
- `COPY` from S3 is the preferred loading method — individual inserts are very slow
- `VACUUM` needed after bulk deletes to reclaim space
- Sort key choice is critical and cannot be changed without recreating the table
- Single-node deployment can't be resized without downtime (snapshot → restore)
- No native support for JSONB or nested data types (though Super/JSON type was added in 2024)

**Pricing**:

| Node Type | Price (On-Demand, us-east-1) | Notes |
|-----------|------------------------------|-------|
| dc2.large | $0.25/hr | 1 slice, 160GB SSD |
| dc2.8xlarge | $4.80/hr | 16 slices, 2.56TB SSD |
| ra3.xlplus | $0.85/hr | Managed storage, 32TB |
| ra3.16xlarge | $13.04/hr | Managed storage, 128TB |
| Spectrum | $5/TB scanned | External table queries |

### 4.3 Snowflake

**Overview**: Snowflake introduced a fundamentally different architecture for MPP — **shared-disk storage with MPP compute**. While not a pure shared-nothing system, its compute layer is massively parallel and warrants inclusion here due to its position as the leading cloud data warehouse.

**Architecture**:

```
┌──────────────────────────────────┐
│        Cloud Storage Layer       │
│  (S3 / GCS / Azure Blob)         │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌───┐  │
│  │ Data│ │Data │ │Data │ │...│  │
│  └─────┘ └─────┘ └─────┘ └───┘  │
└──────────────┬───────────────────┘
               │
┌──────────────▼───────────────────┐
│        Compute Layer             │
│  ┌──────────────────────────┐    │
│  │ Virtual Warehouse (VW) 1 │    │
│  │ X-Small ... 6XL         │    │
│  ├──────────────────────────┤    │
│  │ Virtual Warehouse (VW) 2 │    │
│  │ (for different workload) │    │
│  ├──────────────────────────┤    │
│  │ Virtual Warehouse (VW) N │    │
│  └──────────────────────────┘    │
└──────────────┬───────────────────┘
               │
┌──────────────▼───────────────────┐
│        Cloud Services Layer      │
│  • Authentication                │
│  • Query optimization            │
│  • Metadata management           │
│  • Transaction management        │
│  • Auto-scaling / auto-suspend   │
└──────────────────────────────────┘
```

**Three-Layer Architecture**:

1. **Storage Layer**: Data stored in cloud object storage (S3/GCS/Azure Blob) in Snowflake's proprietary columnar format. This layer is shared across all compute clusters.

2. **Compute Layer**: Virtual Warehouses — independent MPP clusters. Each VW is a pool of EC2/Azure VM/GCE instances that cache data from storage and execute queries. VWs can be:
   - **Scaled up** (larger nodes — X-SMALL to 6XL)
   - **Scaled out** (more nodes — auto-scaling from 1 to ~10 nodes per XL size, up to ~100+ for 6XL)
   - **Multi-cluster** (multiple VWs of the same size behind a single endpoint for concurrency)

3. **Cloud Services Layer**: Persistent, stateless services handling authentication, query optimization, metadata, transaction management, and infrastructure orchestration.

**Key Architectural Differences from Pure MPP**:

| Aspect | Pure MPP (Greenplum/Redshift) | Snowflake |
|--------|-------------------------------|-----------|
| **Data locality** | Data lives on local node SSD | Data is in object store; nodes cache it |
| **Distribution keys** | User-defined, critical for performance | Automatic — no user control |
| **Scaling compute** | Requires data redistribution | Add/remove nodes instantly |
| **Concurrency** | Shared resources across queries | Isolated VWs per workload |
| **Storage scaling** | Add/rebalance nodes | Pay for what you store |
| **Performance for hot data** | Excellent (local SSD) | Good (cache hit) to moderate (cache miss) |

**Key Features**:

| Feature | Details |
|---------|---------|
| **Virtual Warehouses** | Independent, auto-scaling, auto-suspend compute clusters |
| **Time Travel** | Access historical data up to 90 days back |
| **Zero-Copy Cloning** | Instant read/write clone without copying data |
| **Data Sharing** | Share live data across Snowflake accounts (no data movement) |
| **Snowpipe** | Continuous auto-ingestion via event notifications |
| **Streams & Tasks** | Change data capture and scheduling for pipelines |
| **Dynamic Tables** | Declarative incremental refresh for continuous pipelines |
| **Snowpark** | DataFrame-style programming in Python, Java, Scala |
| **Iceberg Tables** | Native Apache Iceberg table format support |

**Pricing Model**:

Snowflake charges based on **compute credits** consumed:

| VW Size | Credits/Hour | Typical Use |
|---------|--------------|-------------|
| X-Small | 1 | Light development |
| Small | 2 | BI dashboards |
| Medium | 4 | ETL, moderate analytics |
| Large | 8 | Heavy analytics |
| X-Large | 16 | Data science |
| 2X-Large | 32 | Production workloads |
| 3X-Large | 64 | Large-scale ETL |
| 4X-Large | 128 | Enterprise |
| 5X-Large | 256 | Extreme |
| 6X-Large | 512 | Maximum |

Storage: ~$40/TB/month (compressed). Cloud service: no separate charge.

### 4.4 Apache Doris

**Overview**: Apache Doris is an open-source MPP database designed specifically for real-time analytics. Originally developed at Baidu and donated to the Apache Software Foundation in 2018, Doris has gained significant adoption in Asia and is growing globally.

**Architecture**:

```
┌────────────────────────────────────────────┐
│              Frontend (FE)                  │
│  • SQL parsing & optimization              │
│  • Metadata management (BDB JE)            │
│  • Query coordination                      │
│  • Load balancing                          │
│  Multiple FEs for HA (Paxos-based)         │
└────────────────────┬───────────────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
    ▼                ▼                ▼
┌──────────┐  ┌──────────┐  ┌──────────────┐
│ BE Node 1│  │ BE Node 2│  │ BE Node N    │
│ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────────┐ │
│ │Tablet│ │  │ │Tablet│ │  │ │ Tablet   │ │
│ │  1-3 │ │  │ │ 4-6  │ │  │ │  N-M     │ │
│ ├──────┤ │  │ ├──────┤ │  │ ├──────────┤ │
│ │Local │ │  │ │Local │ │  │ │ Local    │ │
│ │  SSD │ │  │ │  SSD │ │  │ │   SSD    │ │
│ └──────┘ │  │ └──────┘ │  │ └──────────┘ │
└──────────┘  └──────────┘  └──────────────┘
```

- **Frontend (FE)**: Handles SQL parsing, query planning, metadata management (using BerkeleyDB Java Edition), and load balancing. Multiple FEs can be deployed for HA using Paxos consensus.
- **Backend (BE)**: Stores data in **Tablets** (the unit of data distribution). Each BE manages multiple tablets. Tablets are replicated for fault tolerance. BE nodes perform local query processing.

**Distribution Model**:

Doris uses a **two-level partitioning** approach:

```sql
CREATE TABLE events (
    event_date DATE,
    event_hour TINYINT,
    event_type VARCHAR(50),
    user_id BIGINT,
    event_data JSON
)
PARTITION BY RANGE(event_date) (
    PARTITION p_202501 VALUES LESS THAN ('2025-02-01'),
    PARTITION p_202502 VALUES LESS THAN ('2025-03-01')
)
DISTRIBUTED BY HASH(user_id) BUCKETS 10;
```

- **First level**: Partition by range (typically time) — enables partition pruning
- **Second level**: Hash into buckets within each partition — enables parallel processing

**Key Features**:

| Feature | Details |
|---------|---------|
| **Real-time Ingestion** | Stream Load (HTTP), Broker Load (HDFS/S3), Routine Load (Kafka), Insert Into |
| **Storage** | Columnar, highly compressed (LZ4, Zstd) |
| **Rollup Tables** | Pre-computed aggregation tables — materialized views at ingestion time |
| **Colocation** | Data from joined tables placed on the same BE nodes |
| **Indexing** | Min/max, Bloom filter, inverted index (v2.0+), bitmap index |
| **Materialized Views** | Automatic rewrite and refresh |
| **MySQL Compatibility** | MySQL wire protocol — works with most MySQL clients and ORMs |

**Data Model Types**:

| Model | Description | Use Case |
|-------|-------------|----------|
| **Duplicate** | Raw data storage, no deduplication | Logs, raw events |
| **Aggregate** | Keys define grouping; measures aggregated on ingestion | Pre-computed metrics, rollups |
| **Unique** | Upsert semantics — last write wins on key | Real-time dimension updates |
| **Primary Key** | Delete + update by primary key | CDC streams, mutable dimensions |

**When to Use Apache Doris**:

- Real-time analytics dashboards with sub-second query latency
- High-ingestion-rate scenarios (millions of rows/sec)
- BI workloads on data that is minutes to hours old
- Organizations preferring open-source with MySQL compatibility

### 4.5 ClickHouse

**Overview**: ClickHouse is an open-source columnar MPP database optimized for real-time analytics on large datasets. Developed by Yandex for web analytics (released 2016, open-source), it is known for its exceptional single-table scan performance.

**Architecture**:

```
┌─────────────────────────────────────────┐
│        ClickHouse Server Cluster         │
│                                          │
│  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ Shard 1  │  │ Shard 2  │  │Shard N │ │
│  │ ┌──────┐ │  │ ┌──────┐ │  │┌─────┐ │ │
│  │ │Replica│ │  │ │Replica│ │  ││Rep..│ │ │
│  │ │  A    │ │  │ │  A    │ │  ││A    │ │ │
│  │ ├──────┤ │  │ ├──────┤ │  │├─────┤ │ │
│  │ │Replica│ │  │ │Replica│ │  ││Rep..│ │ │
│  │ │  B    │ │  │ │  B    │ │  ││B    │ │ │
│  │ └──────┘ │  │ └──────┘ │  │└─────┘ │ │
│  └──────────┘  └──────────┘  └────────┘ │
└─────────────────────────────────────────┘
```

- **Shards**: Data is horizontally partitioned across shards (which can be individual servers or groups of replicas)
- **Replicas**: Within a shard, data is replicated for fault tolerance and read scalability
- **Distributed Tables**: A logical view that queries across all shards — routes queries to the appropriate shards
- **Local Tables**: Physical tables on each shard

**MergeTree Engine Family**:

ClickHouse's power comes from the MergeTree engine family:

| Engine | Description |
|--------|-------------|
| **MergeTree** | Base engine — sorted table, partitionable, replication support |
| **ReplacingMergeTree** | Deduplicates rows with same sort key (async) |
| **SummingMergeTree** | Pre-aggregates numeric columns on merge |
| **AggregatingMergeTree** | Stores intermediate aggregation states |
| **CollapsingMergeTree** | Handles mutable state via sign/cancel rows |
| **VersionedCollapsingMergeTree** | Collapsing with version support |
| **GraphiteMergeTree** | Optimized for Graphite monitoring data |

Each table is configured with:
- **ORDER BY**: Sort key (defines physical order)
- **PARTITION BY**: Partition key (enables partition pruning)
- **PRIMARY KEY**: Primary key (subset of ORDER BY, used for indexing)
- **TTL**: Time-to-live for automatic data expiration
- **SAMPLE BY**: Sampling expression for approximate queries

**Key Features**:

| Feature | Details |
|---------|---------|
| **Vectorized Execution** | Processes data in CPU SIMD vector batches (1024 rows at a time) |
| **Real-time Ingestion** | Batch inserts (recommended: 1000+ rows per insert), Kafka engine, materialized views |
| **SQL Subset** | Extensive SQL support but limited UPDATE/DELETE (mutations are async and expensive) |
| **Materialized Views** | Push-based — data flows into views as it is inserted |
| **Full-text Search** | Inverted index support (experimental → stable in recent versions) |
| **External Data Sources** | MySQL, PostgreSQL, MongoDB, Kafka, S3, HDFS, JDBC table engines |
| **Approximate Query Processing** | Aggregation combinators (uniq, quantile, etc.) with controllable precision |
| **Window Functions** | Supported since recent versions |

**Performance Characteristics**:

- **Scan throughput**: 1-10 GB/s per server on typical hardware
- **Ingestion rate**: Millions of rows per second per server
- **Compression ratio**: 3:1 to 10:1 typically (columnar + codec)
- **Query latency**: Milliseconds to seconds for most queries (sub-second for single-table scans)

**When to Use ClickHouse**:

- Observability (logging, tracing, monitoring)
- Time-series analytics (metrics, IoT sensor data)
- Web and product analytics (user event streams)
- High-ingestion-rate workloads (100K+ inserts/sec)
- Approximate or real-time analytics where sub-second response matters

**When NOT to Use ClickHouse**:

- Full ACID transactional workloads
- High-frequency point updates or deletes
- Complex multi-table joins at high concurrency
- General-purpose OLTP

### 4.6 Google BigQuery

**Overview**: Google BigQuery is a serverless, fully managed MPP data warehouse built on Google's Dremel technology. It pioneered many of the concepts now standard in cloud data warehousing — separation of compute and storage, automatic scaling, and columnar execution.

**Architecture**: Dremel Tree

```
Client / API
     │
     ▼
┌────────────┐
│   Root     │  ← Root Mixer (query planning, metadata)
│  Server    │
└─────┬──────┘
      │
┌─────▼──────┐
│   Mixer    │  ← Intermediate mixers (sub-plan coordination)
│  Layer     │
└─────┬──────┘
      │
┌─────▼──────┐
│   Leaf     │  ← Storage and computation (reads Capacitor format)
│  Servers   │
└────────────┘
     │
     ▼
Cloud Storage (Colossus)
```

- **Root Server**: Coordinates query execution, receives SQL, builds the distributed execution plan
- **Mixer Layer**: Intermediate nodes that shuffle and aggregate intermediate results in a tree-like fashion (allows massive parallelism)
- **Leaf Servers**: Read data from Colossus (Google's distributed file system) in Capacitor columnar format and perform local computations
- **Colossus**: Underlying storage system — replicated, compressed, automatically partitioned

BigQuery does not use a shared-nothing model. Instead, compute (slots) and storage are fully separated, and the Dremel tree dynamically assigns work to available slots.

**Key Features**:

| Feature | Details |
|---------|---------|
| **Serverless** | No clusters, no nodes, no scaling — just pay for queries and storage |
| **Automatic Optimization** | No distribution keys, sort keys, or indexes to manage |
| **Columnar Storage (Capacitor)** | Proprietary columnar format with nested/repeated field support |
| **Clustering** | Auto- or manual clustering of data within partitions (similar to sort keys) |
| **Partitioning** | Ingestion time, column range, integer range — with automatic pruning |
| **BigQuery ML** | Train and run ML models directly in SQL |
| **BigQuery Omni** | Query data across AWS/Azure (multi-cloud) |
| **BigLake** | Unified data lake with fine-grained access control on GCS |
| **Streaming Ingestion** | Streaming insert API (billions of rows/day) |
| **Time Travel** | Access data as it existed up to 7 days ago |
| **Materialized Views** | Automatic incremental refresh |

**Slots and Pricing**:

BigQuery pricing has two dimensions:

1. **Storage**: ~$0.02/GB/month for active data, $0.01/GB/month for long-term (90+ days untouched)
2. **Compute (Slots)**:
   - **On-demand**: $5/TB scanned (first 1TB free/month)
   - **Flat-rate**: Commitment to a fixed number of slots (50-2000+), makes cost predictable

A slot is a unit of compute capacity (roughly equivalent to one virtual CPU with associated memory). Queries consume slots proportional to their complexity. With flat-rate reservations, slots can be allocated to different project/team workloads.

**Limitations**:

- No traditional indexes — performance depends on data pruning via partitioning and clustering
- Slot contention under on-demand pricing (shared pool across all GCP customers)
- Quotas on concurrent queries, DML statements, and API requests
- No ACID transactions across tables (only snapshot isolation per table)
- Streaming inserts have a 90-minute window before being available for export/table copy
- Cannot control storage format or compression directly

**BigQuery Comparison**: BigQuery is best when you want to run ad-hoc analytics on large datasets with zero infrastructure management. It is weaker for workloads requiring predictable low-latency or high-frequency micro-queries.

---

## 5. Comparison Table

| Feature | Greenplum | Redshift | Snowflake | ClickHouse | BigQuery | Apache Doris |
|---------|-----------|----------|-----------|------------|----------|-------------|
| **Architecture** | Pure shared-nothing | Shared-nothing | Shared-storage + MPP compute | Shared-nothing | Dremel tree | Pure shared-nothing |
| **Open Source** | ✅ Apache 2.0 | ❌ Proprietary | ❌ Proprietary | ✅ Apache 2.0 | ❌ Proprietary | ✅ Apache 2.0 |
| **Distribution** | Manual (hash/random/replicated) | Manual (KEY/EVEN/ALL) | Automatic | Manual (shard key) | Automatic | Two-level (partition + bucket) |
| **Storage** | Local SSD | Local SSD (RA3: managed + cache) | Cloud object store (S3/GCS/Azure) | Local SSD | Colossus (GFS) | Local SSD |
| **Compute-Storage Separation** | ❌ Tied | ⚠️ RA3 only | ✅ Full | ❌ Tied | ✅ Full | ❌ Tied |
| **Real-time Ingestion** | Moderate | Moderate | Good | Excellent | Good | Excellent |
| **Latency (single-table scan)** | Moderate | Good | Good | Excellent | Good | Good |
| **Full SQL** | ✅ PostgreSQL-based | ✅ | ✅ | ⚠️ Limited DML | ✅ | ✅ MySQL-compatible |
| **ACID Transactions** | ✅ Snapshot isolation | ✅ Serializable (single-table) | ✅ Snapshot isolation | ❌ No | ⚠️ Snapshot per table | ✅ (Aggregate model) |
| **Concurrency** | Moderate | Moderate | Excellent | Good | Excellent | Good |
| **Best Workload** | Enterprise DWH | AWS-native DWH | Multi-cloud DWH | Real-time analytics | Serverless analytics | Real-time analytics |
| **Pricing Model** | BYOL / Subscription | Node-hour | Credit / compute | Open source (free) + Cloud | Per-slot / on-demand | Open source (free) + Cloud |
| **Indexing** | B-tree, bitmap, GiST, min/max | Block-based, sort key | Automatic clustering | Min/max, Bloom, inverted | Partition + cluster (auto) | Min/max, Bloom, inverted, bitmap |
| **Fault Tolerance** | Standby + segment mirror | Automated backup + replication | Continuous data protection | Shard replication | Replicated storage | Tablet replication |
| **Notable Users** | Financial institutions, telecom | Airbnb, Nasdaq, Lyft | Capital One, Instacart | Spotify, Cloudflare, eBay | All GCP customers | Baidu, Meituan |
| **Max Scale** | ~100s of TB | ~PB+ (RA3 managed) | ~PB+ | ~TB-PB | ~EB (theoretically) | ~TB-PB |

---

## 6. Distribution Key Selection

### 6.1 Choosing the Right Distribution Key

The distribution key determines which node stores each row. A good key ensures even data distribution and minimizes data movement during queries.

**Decision tree**:

```
Is the table a small dimension (< 10M rows, < 10GB)?
    │
    ├─ Yes → Use REPLICATED / ALL distribution
    │
    └─ No (fact table, large table)
         │
         Is there a natural join key used in most queries?
              │
              ├─ Yes → Hash distribute on that key
              │
              └─ No → Consider EVEN / RANDOMLY
```

**Practical examples**:

```sql
-- Star schema: Replicate dimensions, distribute fact on join key
CREATE TABLE fact_sales (
    sale_id BIGINT,
    customer_id INT,
    product_id INT,
    store_id INT,
    sale_date DATE,
    quantity INT,
    amount DECIMAL(12,2)
)
DISTRIBUTED BY (customer_id);  -- matches how sales are queried

CREATE TABLE dim_customer (
    customer_id INT,
    name VARCHAR(100),
    segment VARCHAR(50)
)
DISTRIBUTED REPLICATED;  -- small, all nodes need it

-- Time-series: Hash on a high-cardinality key, partition by time
CREATE TABLE events (
    event_id BIGINT,
    event_time TIMESTAMP,
    user_id INT,
    event_type VARCHAR(50),
    payload JSONB
)
DISTRIBUTED BY (user_id)
PARTITION BY RANGE (event_time);

-- Landing table: Random while discovering access patterns
CREATE TABLE raw_import (
    row_id BIGINT,
    raw_data TEXT
)
DISTRIBUTED RANDOMLY;
```

### 6.2 Distribution Key Pitfalls

#### Data Skew

If one or a few nodes receive a disproportionate share of data, the query runs only as fast as the slowest node.

**Identifying skew**:

```sql
-- Greenplum
SELECT gp_segment_id, COUNT(*)
FROM fact_sales
GROUP BY gp_segment_id
ORDER BY gp_segment_id;

-- Redshift
SELECT node, slice, COUNT(*)
FROM stv_slices s
JOIN stv_blocklist b ON s.slice = b.slice
WHERE b.tbl = :table_id
GROUP BY node, slice
ORDER BY node, slice;
```

**Good distribution** (uniform spread):
```
 segment_id | count 
------------+-------
          0 | 1,024,312
          1 | 1,018,755
          2 | 1,032,188
          3 | 1,019,994
          4 | 1,027,451
```

**Bad distribution** (severe skew):
```
 segment_id | count 
------------+-------
          0 | 3,852,401
          1 | 112,345
          2 | 98,234
          3 | 134,567
          4 | 102,456
```

**Common causes of skew**:

1. **Low cardinality key**: `gender` (M/F) → 2 nodes get all data; `status` with 99% 'active' → 1 hot node
2. **Null distribution**: All NULLs hash to the same node (solution: use COALESCE with a random value)
3. **Sequential key Insert**: Bulk loading sequential IDs can cause uneven initial distribution in some systems
4. **Frequent value**: A single customer with 40% of all orders overloads one node

**Fixing skew**:

- Change distribution key (requires table recreation or redistribution)
- Use a compound distribution key (e.g., `(customer_id, sale_date)` instead of `customer_id` alone)
- Add a salt column for problematic high-frequency values
- Consider EVEN/RANDOMLY distribution if no better key exists

#### Join Skew

Even when data is well-distributed, **join skew** can occur if one key value dominates the join.

```sql
-- If customer_id 123 represents 30% of all orders:
-- The node handling customer_id 123 does 30% of the join work
-- Query completes only when that node finishes
```

**Solutions**:

- Distribute both tables on the join key (colocated join → no shuffle)
- Use replicated distribution for the smaller table
- Break the problematic key into a separate query path

### 6.3 Testing Distribution Plans

Before committing to a distribution key, test with representative data:

```sql
-- 1. How many segments will have data?
SELECT COUNT(DISTINCT gp_segment_id) 
FROM staging_table;

-- 2. What's the data spread?
SELECT 
    gp_segment_id,
    COUNT(*) as row_count,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 2) as pct
FROM staging_table
GROUP BY gp_segment_id
ORDER BY gp_segment_id;

-- If any segment has <5% or >30% of data, you have a problem.
```

---

## 7. When to Use MPP vs Alternatives

### 7.1 Decision Matrix

| Use Case | Recommended Systems | Rationale |
|----------|-------------------|-----------|
| **Enterprise data warehouse (10TB–10PB), complex SQL** | Greenplum, Redshift, Snowflake | Full SQL support, mature query optimizers, enterprise features (RLS, workload management) |
| **Real-time analytics, high ingestion rate (>1M rows/sec)** | ClickHouse, Apache Doris | MergeTree engine handles high-throughput ingestion; sub-second scan latency |
| **Serverless, zero infrastructure management** | BigQuery, Snowflake | No cluster management, auto-scaling, pay-per-query |
| **Open source, self-managed, full control** | Greenplum, ClickHouse, Apache Doris | Apache 2.0 licensed, full control over hardware and configuration |
| **AWS-native, existing AWS investment** | Redshift | Tight integration with S3, IAM, VPC, Glue, QuickSight |
| **Multi-cloud / data sharing across organizations** | Snowflake | Native data sharing, cross-cloud replication (AWS↔Azure↔GCP) |
| **Ad-hoc analytics on data lake (S3/ADLS/GCS)** | Snowflake, BigQuery, Redshift Spectrum | Query data in-place without loading; schema-on-read |
| **Detailed OLTP with some analytics** | SingleStore (not pure MPP, hybrid) | Row-store for OLTP + columnar for analytics in one system |
| **Cost-sensitive, predictable workloads** | ClickHouse (self-managed), Redshift (reserved instances) | Open source or reserved pricing reduces costs |
| **Machine learning on warehouse data** | BigQuery ML, Redshift ML, Snowpark ML | In-database ML avoids data movement |

### 7.2 When NOT to Use MPP

Despite their power, MPP databases are not universal:

| Scenario | Better Alternative |
|----------|-------------------|
| **High-frequency single-row lookups** (e.g., user profile by ID) | PostgreSQL, MySQL or a key-value store (Redis, DynamoDB) |
| **Complex OLTP with many small transactions** | PostgreSQL, MySQL, or distributed SQL (CockroachDB, YugabyteDB) |
| **Full-text search at scale** | Elasticsearch, OpenSearch, Meilisearch |
| **Small data (<100GB)** | Single-node PostgreSQL (simpler, cheaper) |
| **Graph traversal queries** (e.g., social network, recommendations) | Neo4j, ArangoDB, or specialized graph DBs |
| **Embedded/edge analytics** (e.g., on mobile device) | DuckDB (embedded OLAP, not MPP-distributed) |
| **Real-time streaming** (event processing, alerting) | Kafka Streams, Flink, Spark Streaming |

### 7.3 Cost Comparison (Approximate)

Monthly cost for 10TB data warehouse, moderate query volume:

| System | Monthly Cost (Approx.) | Notes |
|--------|----------------------|-------|
| **Greenplum (on-prem, self-managed)** | $5K–$15K | Hardware amortized over 3yr; enterprise license varies |
| **Greenplum (cloud, VMware)** | $8K–$20K | Depends on instance size and managed services |
| **Redshift (RA3.4xlarge, 3 nodes)** | ~$9,000 | On-demand; reserved saves ~60% |
| **Snowflake (Medium VW, ~50TB storage)** | ~$12K–$20K | Varies with auto-scaling and concurrency |
| **ClickHouse (self-managed on AWS)** | ~$2K–$5K | 3-5 c5.4xlarge nodes, EBS for storage |
| **BigQuery (on-demand)** | ~$2K–$10K | Storage + query costs; flat-rate for heavy users |
| **Apache Doris (self-managed)** | ~$2K–$6K | Similar hardware to ClickHouse |

**Key cost drivers**: Compute cost, storage cost, data scanned (on-demand), and egress/network charges.

---

## 8. Performance Optimization

### 8.1 Distribution Key (Revisited)

- **High cardinality**: Aim for distribution key with ≥100x distinct values than number of nodes
- **Used in JOIN conditions**: Colocated joins eliminate data shuffling
- **Test for skew**: Run distribution tests before production deployment
- **Avoid single-column low-cardinality keys**: Prefer compound keys `(col1, col2)`

### 8.2 Sort / Order Keys

Sort keys determine the physical order of data on disk. They primarily benefit **range-restricted queries** and **compression**.

#### Redshift Sort Keys

```sql
-- Compound sort key: Prefix optimization
CREATE TABLE orders (
    order_date DATE,
    region VARCHAR(20),
    product_id INT,
    amount DECIMAL
)
SORTKEY (order_date, region);
-- Queries filtering on order_date benefit most
-- Queries filtering on region alone (without order_date) do NOT benefit
-- Query: WHERE order_date >= '2025-01-01' AND region = 'APAC' → Excellent pruning
-- Query: WHERE region = 'APAC' → No pruning (order_date prefix missing)

-- Interleaved sort key: Balanced optimization
CREATE TABLE orders_interleaved (
    order_date DATE,
    region VARCHAR(20),
    product_id INT,
    amount DECIMAL
)
SORTKEY INTERLEAVED (order_date, region);
-- All columns treated equally
-- Both queries above benefit from pruning
-- Cost: Slower COPY and VACUUM operations
```

**Compound vs Interleaved Decision**:

| Factor | Choose Compound | Choose Interleaved |
|--------|----------------|--------------------|
| Query patterns | Predictable, prefix-driven | Unpredictable, varied |
| Sort key columns | 1-2 columns | 3+ columns with equal importance |
| Write frequency | Frequent writes | Read-heavy, infrequent writes |
| Performance cost | Low overhead | Higher maintenance cost |

#### ClickHouse Order Keys

ClickHouse sort keys (ORDER BY) are more fundamental — they define the entire storage layout:

```sql
CREATE TABLE events (
    event_date Date,
    event_time DateTime,
    user_id UInt64,
    event_type String,
    metric Float64
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, event_type, user_id);
-- Data is sorted by event_date first, then event_type, then user_id
-- Primary key is automatically (event_date, event_type, user_id)
```

ClickHouse creates a sparse primary index from the ORDER BY columns. Each index entry marks the first row of a "granule" (typically 8192 rows). Queries that filter on prefix columns of the ORDER BY can skip entire granules.

### 8.3 Compression Tuning

**General rules**:

- **Low-cardinality strings** (status, country, segment): RLE → 20:1 compression
- **Booleans/enums**: RLE or Bitmap → 50:1+
- **Float/integer columns**: Delta + LZ4 → 5:1 to 10:1
- **Text/JSON columns**: Zstd → 3:1 to 5:1
- **Timestamps**: Delta + LZ4 → 8:1 to 15:1 (especially with regular intervals)

**Analyze compression in Greenplum**:

```sql
-- Check compression ratios for AO/CO tables
SELECT 
    relname,
    pg_size_pretty(pg_relation_size(oid)) as raw_size,
    pg_size_pretty(pg_total_relation_size(oid)) as total_size,
    ROUND(100.0 * pg_total_relation_size(oid) / NULLIF(pg_relation_size(oid), 0), 2) as pct
FROM pg_class
WHERE relname LIKE 'sales%';
```

**Analyze compression in Redshift**:

```sql
-- Check column encoding and compression ratios
SELECT 
    tablename, 
    "column", 
    type, 
    encoding,
    ROUND(100.0 * estimated_compressed_size / NULLIF(pg_total_relation_size(tablename::regclass), 0), 2) as pct
FROM pg_table_def t
JOIN svv_table_info i ON t.tablename = i.table_name
WHERE t.schemaname = 'public';
```

**Analyze compression in ClickHouse**:

```sql
SELECT
    table,
    name,
    formatReadableSize(total_bytes) AS compressed_size,
    formatReadableSize(data_uncompressed_bytes) AS uncompressed_size,
    ROUND(100.0 * data_compressed_bytes / NULLIF(data_uncompressed_bytes, 0), 2) AS compression_ratio_pct,
    compression_codec
FROM system.columns
WHERE database = 'default'
ORDER BY total_bytes DESC;
```

### 8.4 Query Optimization Patterns

#### Use Distribution Keys in JOIN Conditions

```sql
-- BAD: Full redistribution required
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2025-01-01';

-- GOOD: Colocated join (both distributed by customer_id)
CREATE TABLE orders () DISTRIBUTED BY (customer_id);
CREATE TABLE customers () DISTRIBUTED BY (customer_id);
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2025-01-01';
-- No data movement for the join; each node joins its local data
```

#### Leverage Replicated Tables

```sql
-- DIM table replicated avoids shuffle
CREATE TABLE dim_region DISTRIBUTED REPLICATED AS
SELECT * FROM external_regions;

-- Fact table distributed on join key
CREATE TABLE fact_sales DISTRIBUTED BY (region_id) AS
SELECT * FROM external_sales;

-- Join: No shuffle required (dimension is everywhere)
-- Only the fact table's distribution matters
SELECT r.region_name, SUM(s.amount)
FROM fact_sales s
JOIN dim_region r ON s.region_id = r.region_id
GROUP BY r.region_name;
```

#### Avoid SELECT *

```sql
-- BAD: Scans all columns, defeats columnar pruning
SELECT * FROM fact_sales WHERE sale_date >= '2025-01-01';

-- GOOD: Only scans 4 columns
SELECT sale_date, customer_id, amount, product_id
FROM fact_sales
WHERE sale_date >= '2025-01-01';
```

This matters more in columnar MPP databases. Reading 5 columns instead of 50 reduces I/O by an order of magnitude.

#### Use Approximate Functions

```sql
-- Exact count distinct (expensive — requires hash set across all nodes)
SELECT COUNT(DISTINCT customer_id) FROM fact_sales;

-- Approximate (much faster, ~1-2% error)
-- Redshift / Greenplum
SELECT APPROXIMATE COUNT(DISTINCT customer_id) FROM fact_sales;

-- ClickHouse
SELECT uniq(customer_id) FROM fact_sales;  -- exact approximate (HyperLogLog)

-- BigQuery
SELECT APPROX_COUNT_DISTINCT(customer_id) FROM fact_sales;
```

#### Leverage Partition Pruning

```sql
-- BAD: Full table scan
SELECT * FROM events WHERE event_date > '2025-06-01';

-- GOOD: Partition key in WHERE clause enables pruning
CREATE TABLE events (...) PARTITION BY RANGE(event_date);
SELECT * FROM events WHERE event_date > '2025-06-01';
-- Only scans partitions Jun-Dec 2025 instead of all partitions
```

### 8.5 Workload Management

#### Redshift Workload Management (WLM)

```sql
-- Auto WLM (default, recommended)
-- Automatically manages query queues based on concurrency and memory

-- Manual WLM (legacy, configurable via parameter group)
-- Queue 1: 5 slots, 50% memory (for short queries)
-- Queue 2: 10 slots, 50% memory (for long ETL queries)
```

#### Greenplum Resource Management

```sql
-- Resource groups (GP7+)
CREATE RESOURCE GROUP rg_etl WITH (
    CPU_RATE_LIMIT = 40,
    MEMORY_LIMIT = 40,
    CONCURRENCY = 10
);

CREATE RESOURCE GROUP rg_bi WITH (
    CPU_RATE_LIMIT = 60,
    MEMORY_LIMIT = 60,
    CONCURRENCY = 5
);

ALTER ROLE etl_user RESOURCE GROUP rg_etl;
ALTER ROLE bi_user RESOURCE GROUP rg_bi;
```

#### Snowflake Resource Management

```sql
-- Virtual warehouse isolation
CREATE WAREHOUSE wh_etl
    WITH WAREHOUSE_SIZE = 'MEDIUM'
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 3
    AUTO_SUSPEND = 300  -- 5 min idle
    AUTO_RESUME = TRUE;

CREATE WAREHOUSE wh_bi
    WITH WAREHOUSE_SIZE = 'LARGE'
    MIN_CLUSTER_COUNT = 1
    MAX_CLUSTER_COUNT = 5
    AUTO_SUSPEND = 600;
```

---

## 9. Data Loading Patterns

### 9.1 Greenplum — Parallel Load with gpfdist

gpfdist is Greenplum's parallel file distribution server. Multiple gpfdist instances enable concurrent reads across segments.

**Setup**:

```bash
# Start gpfdist on ETL host(s)
gpfdist -d /data/sales_export -p 8081 -l /var/log/gpfdist_8081.log &
gpfdist -d /data/sales_export -p 8082 -l /var/log/gpfdist_8082.log &
```

**SQL Load**:

```sql
-- Define external table pointing to gpfdist locations
CREATE EXTERNAL TABLE ext_sales (
    LIKE fact_sales
)
LOCATION (
    'gpfdist://etl_host:8081/*.csv',
    'gpfdist://etl_host:8082/*.csv'
)
FORMAT 'CSV' (DELIMITER '|' NULL 'NULL')
LOG ERRORS SEGMENT REJECT LIMIT 100;

-- Parallel insert into target table
INSERT INTO fact_sales SELECT * FROM ext_sales;

-- Check load errors
SELECT * FROM gp_read_error_log('ext_sales');
```

**Performance tuning**:

- Use 1 gpfdist instance per 4-8 segment nodes
- Compress files with gzip and use `PROTOCOL 'gpfdist'` with compression
- Split large files into 64MB-256MB chunks for even distribution
- Use `gpload` YAML for declarative ETL with transformation

```yaml
# gpload YAML example
VERSION: 1.0.0.1
DATABASE: dw_prod
USER: etl_user
HOST: mdw
PORT: 5432
GPLOAD:
  INPUT:
    - SOURCE:
        LOCAL_HOSTNAME:
          - etl_host
        PORT: 8081
        FILE:
          - /data/sales_202501.csv
    - FORMAT: csv
    - DELIMITER: '|'
    - ERROR_LIMIT: 100
    - ERROR_TABLE: public.sales_load_errors
  OUTPUT:
    - TABLE: public.fact_sales
    - MODE: INSERT
  PRELOAD:
    - TRUNCATE: true
```

### 9.2 Redshift — COPY from S3

Redshift's COPY command is designed for high-throughput parallel loads from S3.

```sql
-- Simple COPY
COPY fact_sales
FROM 's3://data-lake-prod/sales/'
IAM_ROLE 'arn:aws:iam::123456789:role/RedshiftCopyRole'
CSV DELIMITER '|'
IGNOREHEADER 1;

-- With compression
COPY fact_sales
FROM 's3://data-lake-prod/sales/'
IAM_ROLE 'arn:aws:iam::123456789:role/RedshiftCopyRole'
GZIP
CSV;

-- With manifest (exact file list)
COPY fact_sales
FROM 's3://data-lake-prod/sales/manifest.json'
IAM_ROLE 'arn:aws:iam::123456789:role/RedshiftCopyRole'
MANIFEST
CSV;

-- From multiple S3 prefixes in parallel
COPY fact_sales
FROM 's3://data-lake-prod/sales/year=2025/month=01/'
IAM_ROLE 'arn:aws:iam::123456789:role/RedshiftCopyRole'
CSV;

COPY fact_sales
FROM 's3://data-lake-prod/sales/year=2025/month=02/'
IAM_ROLE 'arn:aws:iam::123456789:role/RedshiftCopyRole'
CSV;
```

**COPY Performance Tips**:

- Split files into **equal-sized chunks** (ideally 64MB-1GB compressed)
- Number of files should be a **multiple of the slice count** (e.g., for 32 slices, use 32 or 64 files)
- Use **columnar formats** like Parquet if data originates from data lakes
- Use **STL_LOAD_ERRORS** and **STL_LOAD_COMMITS** for troubleshooting

```sql
-- Monitor COPY progress
SELECT query, trim(filename) as file, line_number, 
       colname, type, raw_line, raw_field_value
FROM stl_load_errors
WHERE query = :last_copy_query_id;

-- Check COPY success
SELECT query, starttime, endtime, rows_loaded,
       file_format, files_loaded
FROM stl_load_commits
WHERE query = :last_copy_query_id;
```

### 9.3 ClickHouse — High-Volume Ingestion

ClickHouse excels at high-ingestion-rate workloads.

**Batch insert (recommended)**:

```sql
-- Insert 1000+ rows per statement
INSERT INTO events (event_date, event_time, user_id, event_type, metric) VALUES
    ('2025-06-01', '2025-06-01 10:00:00', 1234, 'page_view', 1.0),
    ('2025-06-01', '2025-06-01 10:00:01', 5678, 'click', 2.5),
    ...
    (1000+ rows);
```

**Kafka Engine (real-time streaming)**:

```sql
-- Create Kafka source table
CREATE TABLE events_kafka (
    event_date Date,
    event_time DateTime,
    user_id UInt64,
    event_type String,
    metric Float64
)
ENGINE = Kafka
SETTINGS
    kafka_broker_list = 'kafka:9092',
    kafka_topic_list = 'events',
    kafka_group_name = 'clickhouse_consumer',
    kafka_format = 'JSONEachRow';

-- Create target MergeTree table
CREATE TABLE events (
    event_date Date,
    event_time DateTime,
    user_id UInt64,
    event_type String,
    metric Float64
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, event_type, user_id);

-- Materialized view pipes Kafka → MergeTree
CREATE MATERIALIZED VIEW events_mv TO events AS
SELECT * FROM events_kafka;
```

**HTTP Bulk Load**:

```bash
# Using ClickHouse HTTP interface
curl -X POST 'http://localhost:8123/?query=INSERT%20INTO%20events%20FORMAT%20CSV' \
  --data-binary @events.csv

# Using clickhouse-client
clickhouse-client --query="INSERT INTO events FORMAT CSV" < events.csv
```

### 9.4 Apache Doris — Stream Load

Doris supports HTTP-based Stream Load for real-time ingestion:

```bash
# Stream Load via HTTP
curl --location-trusted -u admin:admin_password \
    -H "label: sale_load_20250601" \
    -H "column_separator:|" \
    -T /data/sales_20250601.csv \
    http://fe_host:8030/api/db_name/fact_sales/_stream_load
```

**Response**:

```json
{
    "Status": "Success",
    "NumberTotalRows": 5000000,
    "NumberLoadedRows": 5000000,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 2147483648,
    "Label": "sale_load_20250601"
}
```

### 9.5 BigQuery — Streaming and Batch

```sql
-- Batch load (recommended for large datasets)
-- Use bq command-line tool:
-- bq load --source_format=CSV --autodetect dataset.fact_sales gs://bucket/sales/*.csv

-- Streaming insert (for real-time, billable per row)
INSERT INTO `project.dataset.fact_sales` (sale_id, customer_id, amount, sale_date)
VALUES (1, 1001, 250.00, '2025-06-01');
```

---

## 10. Future Trends

### 10.1 Compute-Storage Separation

The trend is clear: move from tied compute-storage to fully separated architectures.

- **Redshift RA3** managed storage already separates compute from storage with local SSD caching
- **Snowflake** and **BigQuery** pioneered this model — it is now the standard for cloud data warehouses
- **Greenplum** is moving in this direction with integration with cloud object stores
- **ClickHouse** supports S3-backed storage via its S3 table engine and the `StorageConfiguration` for MergeTree

**Benefit**: Elastic scaling — add compute without data movement, and only pay for storage independently.

### 10.2 Real-Time Analytics Convergence

The boundary between "real-time" and "batch" analytics is blurring:

- **ClickHouse** and **Doris** lead with native streaming ingestion (Kafka, HTTP Stream Load)
- **Snowflake** added Dynamic Tables for declarative incremental refresh
- **Redshift** added materialized views with automatic refresh
- **BigQuery** added continuous queries and real-time ingestion improvements

Expect MPP databases to increasingly support **streaming ingestion** as a first-class feature rather than an add-on.

### 10.3 Cloud-Native Open Source

Open-source MPP databases are getting cloud-managed offerings:

- **Greenplum**: VMware Tanzu Greenplum on GCP and AWS
- **ClickHouse**: ClickHouse Cloud (managed by ClickHouse Inc.)
- **Apache Doris**: SelectDB Cloud (managed by Doris developers)
- **StarRocks**: CelerData cloud offering (StarRocks fork)

This gives enterprises the flexibility of open-source with the operational convenience of managed services.

### 10.4 AI/ML Integration

In-database ML is becoming standard:

| Feature | System | Description |
|---------|--------|-------------|
| **BigQuery ML** | BigQuery | Train/run ML models with SQL (linear regression, XGBoost, deep learning) |
| **Redshift ML** | Redshift | SageMaker integration for model training + SQL inference |
| **Snowpark ML** | Snowflake | DataFrame-based ML workflows with scikit-learn, XGBoost |
| **Madlib** | Greenplum | In-database ML library (regression, classification, graph) |

The direction is clear: **bring ML to the data** rather than moving data to ML frameworks.

### 10.5 Apache Iceberg / Delta Lake / Lakehouse Support

MPP databases are adding native support for open table formats:

- **Snowflake Iceberg Tables**: Native Apache Iceberg support (2023)
- **BigQuery** supports Iceberg tables via BigLake
- **Redshift Spectrum** queries Iceberg/Delta tables in S3
- **ClickHouse** has experimental Iceberg table engine
- **Doris** supports Iceberg and Hive external tables

This enables a **lakehouse architecture** where the same data is accessible by Spark, Trino, and the MPP database with consistent semantics.

### 10.6 Elastic Scaling

- **Snowflake** and **BigQuery** already offer instant scaling — add/remove compute in seconds
- **Redshift Concurrency Scaling** adds transient clusters for burst workloads
- **ClickHouse** supports dynamic cluster expansion with automatic rebalancing (via `Distributed` engine)
- **Greenplum** segment expansion requires manual redistribution but is being automated

Future MPP systems will treat **elasticity** as table stakes — automatically scaling compute up and down based on workload demand.

### 10.7 GPU-Accelerated MPP

GPUs are increasingly used for data processing:

- **RAPIDS cuDF** + dask-cudf enables GPU-accelerated MPP-like processing
- **ClickHouse** supports GPU acceleration via specialized extensions
- **HeavyDB** (formerly OmniSci) is a GPU-native MPP database
- **Elasticsearch** + NVIDIA GPUs for accelerated vector search

As GPU memory and programmability improve, expect more MPP systems to leverage GPU parallelism for compute-intensive operations (filtering, aggregation, sorting).

### 10.8 Serverless MPP for Everyone

The serverless model (no capacity planning, no cluster management) is expanding:

- **BigQuery** remains the archetype — truly serverless
- **Snowflake** Multi-cluster Warehouses get close with auto-scaling
- **Redshift Serverless** (launched 2022) eliminates node management
- **ClickHouse Cloud** offers serverless-like experience with auto-scaling compute groups

By 2030, it is reasonable to expect that **most new MPP deployments will be serverless**, with self-managed clusters being the exception for cost optimization or compliance reasons.

---

## 11. Data Sources

### Official Documentation

- **Greenplum Database**: https://docs.vmware.com/en/VMware-Greenplum/index.html
- **Amazon Redshift**: https://docs.aws.amazon.com/redshift/latest/dg/welcome.html
- **Snowflake**: https://docs.snowflake.com/en/
- **ClickHouse**: https://clickhouse.com/docs
- **Apache Doris**: https://doris.apache.org/docs/
- **Google BigQuery**: https://cloud.google.com/bigquery/docs

### Architecture Papers

- Melnik, S., et al. "Dremel: Interactive Analysis of Web-Scale Datasets." VLDB 2010. *[BigQuery/Dremel architecture]*
- Korotkov, A. "Greenplum Architecture and Query Optimization." PGCon 2021.
- Yandex. "ClickHouse Architecture." ClickHouse Documentation. https://clickhouse.com/docs/en/development/architecture
- Apache Doris. "Design Documents." https://doris.apache.org/community/design/
- AWS. "Amazon Redshift Cluster Management Guide." https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html

### Industry Comparisons and Benchmarks

- Gartner. "Magic Quadrant for Cloud Database Management Systems." (Annual)
- Fivetran. "Modern Data Warehousing: Redshift vs Snowflake vs BigQuery." (2024)
- ClickHouse Benchmark. "ClickHouse vs Greenplum vs Redshift." https://benchmark.clickhouse.com/
- Apache Doris. "Doris vs ClickHouse vs StarRocks." (Doris Documentation)
- TPC-H / TPC-DS Benchmarks. Transaction Processing Performance Council. http://www.tpc.org/

### Books and Courses

- "The Data Warehouse Toolkit" (3rd Edition) — Ralph Kimball, Margy Ross
- "Designing Data-Intensive Applications" — Martin Kleppmann
- "Greenplum: Building Data Warehousing Solutions" — Alexander Denesov
- "ClickHouse: Principles and Practices" — Chen Yue (Chinese/English editions)

### Community Resources

- Greenplum Community: https://greenplum.org/
- ClickHouse GitHub: https://github.com/ClickHouse/ClickHouse
- Apache Doris GitHub: https://github.com/apache/doris
- Redshift Developer Forums: https://repost.aws/tags/TAKEdk1weQ3elP4JfBcCj9mA/amazon-redshift

---

> **About this guide**: This is a living document maintained as part of the research repository. It aims to provide a comprehensive, neutral reference on MPP database architecture, major systems, and practical best practices. Updates and corrections are welcome.
