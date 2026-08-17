# S3 Architecture — Amazon Simple Storage Service: A Comprehensive Guide

**Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB (Singapore)
**Series:** Cloud Storage / AWS Architecture
**Repo:** https://github.com/jackliusr/research — `technology/cloud-storage`
**Audience:** Solution architects, platform engineers, data engineers, SREs
**Scope:** The S3 data model, internal architecture, durability and consistency guarantees, storage classes, lifecycle and versioning, replication, security, performance, API/integrations, enterprise patterns, and a worked bank-platform example.

---

## Table of Contents

1. [S3 Overview](#1-s3-overview)
2. [The Data Model](#2-the-data-model)
3. [Internals: How S3 Actually Works](#3-internals-how-s3-actually-works)
4. [Durability and Availability](#4-durability-and-availability)
5. [Consistency](#5-consistency)
6. [Storage Classes](#6-storage-classes)
7. [Lifecycle and Versioning](#7-lifecycle-and-versioning)
8. [Replication](#8-replication)
9. [Security](#9-security)
10. [Performance](#10-performance)
11. [API and Integrations](#11-api-and-integrations)
12. [S3 in the Enterprise](#12-s3-in-the-enterprise)
13. [Worked Example: A Bank's Data Platform on S3](#13-worked-example-a-banks-data-platform-on-s3)
14. [Summary: S3 in One Page](#14-summary-s3-in-one-page)
15. [Glossary](#glossary)

---

## 0. How to Read This Guide

This is the dedicated deep-dive on S3 in the research repo. It sits alongside, and cross-references, several sibling guides — the intent is one authoritative S3 reference that the others point into, rather than duplicating their material:

- **[`aws_sap_c02_guide.md`](aws_sap_c02_guide.md)** — exam-oriented survey of AWS services, including S3 storage classes, S3 Object Lambda, S3 Storage Lens, and S3 Select (§4.16 Storage Design). Use it for the certification view; this guide is the architectural deep-dive behind those bullets.
- **[`cloud_object_storage_lakehouse_guide.md`](cloud_object_storage_lakehouse_guide.md)** — object storage as the foundation of lakehouse architectures (open table formats, S3-compatible API as a universal interface). Cross-reference for the data-lake/lakehouse story.
- **[`cloud_providers_guide.md`](cloud_providers_guide.md)** — where S3 sits in the AWS portfolio and vs. Azure Blob / GCS (§3.3 Storage Services).
- **[`cephfs_alternatives_guide.md`](cephfs_alternatives_guide.md)** and **[`dell_objectscale_guide.md`](dell_objectscale_guide.md)** — on-premises and private-cloud object storage (Ceph RADOS/RGW, ObjectScale); the "build vs. buy vs. on-prem" comparison for S3-compatible storage.
- **[`finops_guide.md`](finops_guide.md)** — the cost-optimization discipline that S3 lifecycle rules and storage-class selection serve.
- **[`audit_as_code_guide.md`](audit_as_code_guide.md)** — CloudTrail data events, S3 access logging, and evidence collection for audit trails on S3.
- **[`advanced_analytics_solutions_guide.md`](advanced_analytics_solutions_guide.md)** — analytics engines (Athena, Redshift Spectrum, EMR) running queries directly against S3 data lakes, including banking use cases.
- **[`event_stream_processing_guide.md`](event_stream_processing_guide.md)** — event-driven pipelines; S3 event notifications are the ingestion trigger for many of them.
- **`data/` guides** (`on_prem_lakehouse_guide.md`, `delta_lake_vs_iceberg.md`) — lakehouse table formats (Delta Lake, Iceberg) whose data lives on S3.

**A note on verification.** Where a fact is officially documented by AWS (launch date, SLA numbers, API limits), it is stated as fact. Where the source is AWS engineering talks, blog posts, or third-party analysis — and especially where AWS has *not* published details (e.g., exact erasure-coding parameters) — the guide says so explicitly with a ⚠ **Verify** flag. Pricing figures are approximate and dated; always check the AWS price list before committing numbers to a business case.

---

## 1. S3 Overview

### 1.1 What S3 Is

**Amazon Simple Storage Service (Amazon S3)** is AWS's object storage service: a web-scale key-value store that lets you store and retrieve any amount of data, from any location, over HTTP(S), paying only for what you use. AWS's own one-line description from the launch announcement has aged remarkably well:

> "Amazon S3 is storage for the Internet. It is designed to make web-scale computing easier for developers." — AWS launch announcement, March 14, 2006.

**Verified launch facts:**
- S3 launched **March 14, 2006** (Pi Day), announced on the AWS News Blog and via press release. It was the first AWS infrastructure service to reach general availability (EC2 followed later that year, in August 2006).
- The original announcement described "a simple storage service" with SOAP and REST interfaces, where "each block can be up to 5 GB in length, and is associated with a user-defined key and additional key/value metadata pairs" — the core data model (key → value + metadata) is unchanged twenty years later.
- S3 was built on the same scalable storage infrastructure Amazon.com used for its e-commerce platform; the service turned internal infrastructure into a product.

### 1.2 Why S3 — The Rationale

S3 answered a question that dominated the mid-2000s: *how do you store data for "web-scale" applications without building your own storage infrastructure?* Before S3, a team needing storage had to buy and operate its own servers, RAID arrays, and backup systems — capacity planning, replication, and durability were all your problem. S3 inverted the model:

- **Object storage at scale.** Objects (not files in a directory tree) in a flat namespace, addressed by a key. Scale is horizontal and effectively unbounded — there is no capacity ceiling you must plan for.
- **Utility pricing.** Pay per GB-month and per request. No upfront hardware, no minimum commitment.
- **Operationally invisible durability.** Replication, checksums, and self-healing are AWS's problem, not yours — you get a contract (11 nines design durability) instead of a maintenance burden.
- **HTTP everywhere.** Any client that speaks HTTP(S) can read and write; no special protocols, no mounting, no licensing.
- **Separation of storage from compute.** You pay for storage and compute independently, and can spin compute up and down around a persistent data store — the architectural move that made data lakes, serverless, and analytics-on-storage possible.

Twenty years later the pitch is the same; only the scale is bigger. AWS states S3 stores **hundreds of trillions of objects** and processes **tens of millions of requests per second** across its regions — ⚠ **Verify** — these figures come from AWS marketing/engineering talks (e.g., the "S3 at 20" retrospective, March 2026) and are approximate, but the order of magnitude is not in dispute.

### 1.3 S3's Position in the Storage Landscape

S3 is *the* reference implementation of **object storage** and, more importantly, the *de facto standard API* for it. Its position:

| Dimension | Position |
|---|---|
| Storage paradigm | Object storage (vs. block storage like EBS, file storage like EFS/FSx) |
| Interface | RESTful HTTP(S) API over a flat key-value namespace |
| Scope | Global service: region-scoped data, region-wide namespace for general-purpose buckets |
| Ecosystem role | Default landing zone for AWS data: backups, logs, data lakes, ML datasets, static assets, archival |
| Industry role | The S3 API is the common language of object storage — MinIO, Ceph RGW, Dell ObjectScale, NetApp StorageGRID, and competitors' services (Azure Blob, GCS) all emulate it (see `cloud_object_storage_lakehouse_guide.md`) |
| Economics | ~$0.021–0.023/GB-month for Standard in us-east-1 (⚠ approximate, changes frequently; see `finops_guide.md`) |

In the classic storage taxonomy: **files** are byte streams with a hierarchical path; **blocks** are fixed-size addressed sectors presented as raw devices; **objects** are self-contained (data + metadata + key) units in a flat namespace. S3 is the object store that made this category mainstream.

### 1.4 The Overview Table

| Aspect | Description |
|---|---|
| Service name | Amazon Simple Storage Service (Amazon S3) |
| Launch | March 14, 2006 — first GA AWS service |
| Category | Object storage (key-value store of objects) |
| Data model | Buckets → objects (key + value + metadata + version ID) |
| Interface | REST/SOAP (SOAP retired), HTTPS, SDKs, CLI, console |
| Durability design target | 99.999999999% (11 nines) for multi-AZ classes |
| Availability SLA (Standard) | 99.99% |
| Consistency | Strong read-after-write for all operations since Dec 2, 2020 |
| Max object size | 5 TB historically; **50 TB since re:Invent 2025** (⚠ new limit, Dec 2025) |
| Storage classes | Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Flexible Retrieval, Glacier Deep Archive, Express One Zone |
| Typical use | Data lakes, backups, archives, static web content, ML datasets, application storage |
| Compliance | SOC 1/2/3, ISO 27001, PCI DSS, HIPAA, FedRAMP, and regional frameworks (relevant for a bank's control environment) |

---

## 2. The Data Model

S3's data model is deliberately minimal: **buckets** contain **objects**, and each object is a **key**, a **value** (the data), **metadata**, and optionally a **version ID**. That is the entire model. Everything else — folders, tags, lifecycle, replication — is layered on top.

### 2.1 Buckets

A **bucket** is a container for objects and the unit of configuration: policies, encryption, lifecycle rules, versioning, logging, replication, and access controls are all attached at bucket level. Buckets are also the unit of billing and quota (Soft Limit of 100 buckets per account by default — a hard *soft* limit you can raise via Service Quotas).

**Verified bucket facts:**
- **General-purpose bucket names live in a global namespace**: bucket names must be *globally unique across all AWS accounts and all regions* (with the exception of S3 Express directory buckets, which use a regional namespace — see §10.3). If anyone, anywhere, has taken `acme-data`, you cannot use it. This is the most commonly hit operational constraint in S3 design.
- Naming rules: 3–63 characters, lowercase letters, numbers, dots, and hyphens; must begin and end with a letter or number; must be DNS-compliant (no underscores, no adjacent dots, no IP-address-form names). A sane enterprise convention: `<company>-<env>-<purpose>-<region>` e.g. `cacib-prod-txnlake-eu-west-1`.
- Buckets are created in a **region**; the bucket's data lives in that region (except with cross-region replication). The namespace is global, the data is regional.
- You can create up to 100 buckets per account by default (raised on request) — one common reason teams move from "one bucket per project" to "one bucket per environment/purpose with prefixes."

### 2.2 Objects

An **object** is the fundamental stored entity. Its anatomy:

| Part | Description |
|---|---|
| **Key** | The unique identifier (name) of the object within the bucket — effectively a string, up to 1,024 bytes (UTF-8) |
| **Value** | The data itself — a byte sequence up to 5 TB (historically) / 50 TB (since Dec 2025) |
| **System metadata** | Created and managed by S3: Content-Type, Content-Length, Last-Modified, ETag, x-amz-version-id, storage class, checksum values |
| **User metadata** | `x-amz-meta-*` headers you supply, up to 2 KB total per object (key + value combined); not queryable — you must read the object head to see them |
| **Object tags** | Up to 10 key-value tags per object; *are* queryable and can drive lifecycle rules, access policies, and cost allocation |
| **Version ID** | Assigned when versioning is enabled; distinguishes versions of the same key (see §7.2) |

**Verified limits:**
- Single `PUT` request: maximum **5 GB**.
- Multipart upload: for objects **5 MB to 5 TB** (per classic documentation; AWS now documents "5 MB to 50 TB" following the 2025 limit increase), with up to 10,000 parts and a minimum part size of 5 MB (except the last part).
- Maximum object size: **5 TB** for most of S3's life; **raised to 50 TB at re:Invent 2025** (Dec 2, 2025) — ⚠ **Verify** — the 10x increase was announced in the re:Invent 2025 keynote; adoption across SDKs/tools was rolling out through 2026.
- An object is atomic: a write is all-or-nothing. There is no partial object visible to readers (this is what makes the data model safe for concurrent writers).

### 2.3 Keys and Prefixes

The **key** is a plain string; S3 has *no real folder hierarchy*. What looks like a directory —

```
s3://cacib-prod-txnlake/2026/08/17/trades/part-00001.parquet
```

— is just a key containing `/` characters. The "folders" are a console convenience (a "prefix" is the key up to the last `/`). This has two architectural consequences:

1. **The flat namespace is the source of scale.** There is no directory tree to lock, move, or rebalance; S3 can shard purely on key ranges (see §3).
2. **Listing and grouping are prefix operations.** `ListObjectsV2` with `prefix=` gives you all keys sharing a prefix; "folders" are derived. Tools like Athena partition pruning rely on prefix conventions (`dt=2026-08-17/`), so key design doubles as query optimization.

A **prefix** is therefore not a first-class entity but a naming convention — yet it is the most important convention in S3 architecture, because lifecycle rules, replication rules, bucket policies, and Athena/Glue partition pruning all key off it.

### 2.4 The Data Model Table

| Component | Description | Example |
|---|---|---|
| Bucket | Global-namespace container; unit of config, policy, billing | `cacib-prod-txnlake` |
| Object | Key + value + metadata (+ version ID); atomic unit of read/write | A Parquet file, an image, a database backup |
| Key | Flat-namespace identifier (≤1,024 bytes) | `2026/08/17/trades/part-00001.parquet` |
| Prefix | Key substring used for grouping/listing/policy | `2026/08/17/` |
| Value | The object data (≤5 TB / 50 TB) | 128 MB of Parquet rows |
| Metadata | System + user (`x-amz-meta-*`), ≤2 KB; tags ≤10/object | `Content-Type: application/x-parquet` |
| Version ID | Distinguishes versions when versioning is on | `3/L4kqtJlcpXroDTDmJ+rmSpXd3dIbrHY` |

---

## 3. Internals: How S3 Actually Works

AWS has never published a full internal design of S3 — no whitepaper describes the complete system. What we know comes from engineering talks (notably by Andy Warfield, VP of Storage at AWS), re:Invent deep-dive sessions, and AWS's own blog retrospectives. This section synthesizes that public material and flags what is inference.

### 3.1 The Architecture: Partitions and Shards

The canonical public description is that **S3 is a distributed system built from two planes**:

1. **The metadata plane** — a sharded, replicated store of object metadata (key → location, version, checksums, attributes). Every object's metadata lives in a shard; the shard is the unit of consistency decisions.
2. **The data plane** — clusters of storage nodes ("cells") that hold the actual object bytes, replicated or erasure-coded across devices and **Availability Zones (AZs)**.

The scaling unit is the **partition** (often called a shard in talks): a contiguous range of keys managed by a metadata partition plus its associated storage. When a partition becomes hot — too many requests, too much data, or too much traffic on a key range — S3 **splits** it into smaller partitions automatically. This is why the *old* performance advice ("randomize your key prefixes") existed pre-2018, and why it became unnecessary in July 2018 when S3 began handling partition splitting transparently at much higher rates (see §10.2).

⚠ **Verify — the exact design.** The partition/shard model is well documented in AWS talks, but precise numbers (partition size thresholds, split criteria, replication factors, erasure-coding parameters) are **not published**. AWS states only that S3 is designed for 99.999999999% durability across a *minimum of three Availability Zones*; the underlying coding scheme is proprietary. Sources: Andy Warfield's re:Invent talks (e.g., "Amazon S3 Storage" sessions), the re:Invent 2024 "Deep Dive on Amazon S3" (STG302), and the March 2026 "S3 at 20" retrospective. Any blog that gives you an exact "RS(6,3)" or "3x replication" figure for S3 is speculating — treat it as such.

### 3.2 The Storage Layout

The durable-storage design, as described in AWS talks and consistent across decades of S3 engineering discussion:

- **Erasure coding, not simple replication.** AWS has confirmed S3 uses erasure coding internally (it tolerates device and node failures with less overhead than 3x replication). ⚠ **Verify** — the exact scheme (code rate, stripe width) is unpublished. What is published: S3 "redundantly stores data across multiple geographically separated AZs" and is designed to sustain the loss of an entire AZ without data loss.
- **End-to-end checksums.** Objects are checksummed from upload through storage to retrieval; AWS has mentioned on the order of billions of checksum calculations per second across the fleet (⚠ approximate, from re:Invent deep dives). Checksums detect silent corruption, which is then repaired from redundant copies.
- **Self-healing.** Background processes continuously verify data and rebuild/repair anything degraded — a "data hygiene" loop that is a large part of how 11 nines is actually operated rather than merely designed.
- **Shuffle sharding.** Traffic and failures are spread across shards so that correlated failures (e.g., a buggy software rollout hitting a subset of nodes) affect only a small slice of customers, not a shared fate — a pattern AWS engineers have described in reliability talks. ⚠ **Verify** — this is described in S3 reliability sessions (e.g., STG302 at re:Invent 2024) but the deployment details are internal.
- **Quorum-based consistency.** Write operations are acknowledged only after the data is durably stored on enough replicas across AZs; this is the mechanism behind both durability and strong consistency (see §5).

### 3.3 The Internals Table

| Component | Role | Notes |
|---|---|---|
| Metadata plane | Stores key → location/attributes; the consistency authority | Sharded; shard = unit of split/scale |
| Data plane / storage cells | Holds object bytes | Erasure-coded across devices and AZs |
| Partition (shard) | Contiguous key range; unit of horizontal scaling | Splits automatically under load |
| Erasure coding | Redundancy with less overhead than 3x replication | Exact parameters unpublished ⚠ |
| Checksums | Corruption detection end-to-end | Billions/sec fleet-wide (approx.) ⚠ |
| Self-healing | Continuous verification and repair | Operates the 11-nines design |
| Shuffle sharding | Blast-radius isolation | Described in reliability talks ⚠ |
| Quorum writes | Durability + consistency gate | Acknowledge only after durable write |

**Why this matters to architects:** you do not operate any of this — but you *design against* it. The flat namespace + automatic sharding means you never partition data by hand; the erasure-coded multi-AZ design means "one AZ is down" is a normal day, not a disaster; and the consistency gate means you can build stateful pipelines on S3 without inventing your own read-after-write handling.

---

## 4. Durability and Availability

Durability and availability are the two guarantees people most often conflate. **Durability** is "will the data still be there in N years" (loss prevention). **Availability** is "can I read/write right now" (uptime). S3's numbers are famously asymmetric: 99.999999999% durability vs. 99.99% availability.

### 4.1 The 11 Nines — 99.999999999%

**Verified:** AWS documents S3 (multi-AZ classes) as **designed for 99.999999999% (11 nines) durability**. The number is a *design target*, not an SLA — AWS does not offer an SLA on durability, because the implied failure rate (losing one object out of 10 million in 10,000 years, per AWS's own illustration) is not something a service credit can meaningfully remedy. What AWS does offer is a **Service Level Agreement (SLA)** on *availability* (uptime), with credits if breached.

How the design target is met:

- Data is redundantly stored across a **minimum of three Availability Zones** (for multi-AZ classes), where each AZ is an isolated failure domain (independent power, network, cooling).
- **Erasure coding** provides redundancy with lower overhead than full replication; ⚠ **Verify** — the exact scheme is unpublished (see §3.2).
- **End-to-end checksums** catch silent bit-rot; **self-healing** repairs it.
- **Versioning** protects against accidental deletion/overwrite (operator error is, in practice, the biggest durability threat — a fact worth stating loudly in any enterprise design).

The annualized expected loss for 1 PB at 11 nines: on the order of 10⁻³ bytes/year — i.e., effectively "never," which is why S3 is treated as the durability anchor of most enterprise architectures.

### 4.2 Replication and Erasure Coding

Two different mechanisms, two different jobs:

- **Cross-AZ replication (internal, synchronous):** every write is synchronously replicated to at least two other AZs *before* S3 acknowledges the write. This is the mechanism that makes "lose an entire AZ, lose nothing" true, and it is invisible to you — there is no knob, and it is not the same as the *optional* S3 Replication feature (§8), which is asynchronous and cross-bucket.
- **Erasure coding (internal):** within and across AZs, object bytes are striped with parity across many devices/nodes. This is how S3 tolerates *device* failures (the frequent case) with less storage overhead than 3x replication, and it is the reason a "degraded" S3 still returns correct data.

⚠ **Verify — the boundary of public knowledge.** AWS has confirmed: erasure coding is used; data spans ≥3 AZs for multi-AZ classes; writes are acknowledged only after durable, multi-AZ persistence. AWS has *not* published: the code parameters, the number of copies, or the exact probability model behind 11 nines. Third-party "exact" descriptions are reconstructions, not documentation.

### 4.3 Availability — the 99.99% SLA

**Verified SLA numbers** (per AWS service-level agreements, multi-region service credits apply):

| Class | Availability SLA |
|---|---|
| S3 Standard | 99.99% |
| S3 Intelligent-Tiering | 99.9% |
| S3 Standard-IA | 99.9% |
| S3 One Zone-IA | 99.5% |
| S3 Glacier Instant Retrieval | 99.9% |
| S3 Glacier Flexible Retrieval | 99.99% |
| S3 Glacier Deep Archive | 99.99% |
| S3 Express One Zone | 99.95% |

Note the deliberate trade: **One Zone-IA drops both durability (99.99% — data lives in one AZ) and availability (99.5%)** in exchange for the lowest cost of the non-archive classes. It is for reproducible data (thumbnails, derived artifacts) where loss is an inconvenience, not an event.

### 4.4 The Durability Table

| Aspect | Design | SLA / Contract |
|---|---|---|
| Durability (multi-AZ classes) | 99.999999999% (11 nines), design target | No durability SLA; credits are availability-based |
| Durability (One Zone-IA) | 99.99% (single AZ) | No durability SLA |
| Availability (Standard) | Redundant infrastructure, auto-scaling | 99.99% SLA with service credits |
| Replication | Synchronous cross-AZ (internal, mandatory) | Not separately contracted; underpins durability |
| Erasure coding | Internal redundancy scheme | Parameters unpublished ⚠ |
| Corruption defense | End-to-end checksums + self-healing | Implicit in durability design |
| Human-error defense | Versioning, MFA delete, Object Lock (WORM) | Optional features, you enable them |

---

## 5. Consistency

### 5.1 The December 2020 Change

**Verified:** On **December 2, 2020**, AWS announced that **Amazon S3 delivers strong read-after-write consistency automatically for all applications, in all regions, at no additional cost, with no opt-in**. The change was applied to *all* existing buckets — a rare, fleet-wide semantic upgrade with zero customer action.

What became strongly consistent on that date: **all GET, PUT, and LIST operations, plus operations that change object tags, ACLs, or metadata** — and, critically, *overwrite* PUTs and *DELETE* operations.

### 5.2 The Consistency Model

The model, in plain terms:

| Operation | Guarantee (since Dec 2, 2020) |
|---|---|
| Write a new object, then read it | You will see the new object (read-after-write) |
| Overwrite an existing object, then read it | You will see the *new* content, never the old (or a mix) |
| Delete an object, then read it | You will get 404 — the deleted object does not reappear |
| List a bucket/prefix | The list reflects all completed writes and deletes |
| Read after a failed/concurrent write | Atomic: readers see either the old or the new complete object, never a partial one |
| Headers/metadata/tags changes | Strongly consistent immediately |
| Cross-region replication | *Asynchronous* — eventual consistency applies to replicated copies (see §8) |

Before December 2020, S3 was: read-after-write consistent for *new* object PUTs (since roughly 2015 for all new objects, and effectively from the start for most), but **eventually consistent for overwrite PUTs and DELETEs** — a stale read could return the old object for a bounded window. This was the most cited S3 limitation for a decade, and its removal in 2020 was a genuinely large engineering statement: it means S3's metadata plane now serializes overwrites and deletes across the fleet.

### 5.3 Implications for Architects

- **No more write-then-read workarounds.** Code that slept, retried, or read-your-own-writes via auxiliary stores can be simplified.
- **Stateful pipelines are safe on S3.** Upsert-then-read patterns, leader-election via conditional writes, and "object as source of truth" designs no longer need a side database for consistency.
- **The consistency table**

| Operation | Before Dec 2020 | After Dec 2020 |
|---|---|---|
| PUT new object → GET | Read-after-write (strong) | Strong |
| PUT overwrite → GET | Eventually consistent (stale reads possible) | Strong |
| DELETE → GET | Eventually consistent (object could reappear briefly) | Strong |
| LIST | Eventually consistent (delayed visibility) | Strong |
| GET after modifying tags/ACL/metadata | Eventually consistent | Strong |
| Replicated copy (CRR/SRR) | Eventually consistent | Still eventually consistent (async replication) |

---

## 6. Storage Classes

S3 is not one product; it is a ladder of eight storage classes trading cost against access latency and redundancy. The right architecture question is rarely "should we use S3?" but "which class, and what's the transition plan?" (see §7 on lifecycle).

### 6.1 The Classes

- **S3 Standard** — hot, frequently accessed data. 11 nines durability across ≥3 AZs, 99.99% availability, millisecond access. The default and the reference point. Use for active data lakes, application data, content delivery.
- **S3 Intelligent-Tiering** — automates class selection. Monitors access per object and moves data between a frequent-access tier, an infrequent-access tier, and — optionally — archive tiers (Archive Access, Deep Archive Access). No retrieval fees; a **monitoring/automation charge per 1,000 objects** (⚠ ~$0.0025/1k objects, verify current price). Best for *unknown or varying* access patterns; bad for predictable hot data (you pay the monitoring fee for no benefit).
- **S3 Standard-IA** — infrequent but immediate access. Same durability, 99.9% availability, **retrieval fee per GB** on top of lower storage price. The "backups and older data" class.
- **S3 One Zone-IA** — like Standard-IA but in a single AZ: 99.99% durability, 99.5% availability, cheapest of the IA pair. For reproducible data only.
- **S3 Glacier Instant Retrieval** (added Nov 2021) — archive-class storage with millisecond retrieval; ~4x cheaper than Standard. For long-lived data that *might* be needed immediately (quarterly reports, media masters).
- **S3 Glacier Flexible Retrieval** (the original "S3 Glacier," renamed Nov 2021) — archival with retrieval in **minutes to hours** (Expedited 1–5 min, Standard 3–5 h, Bulk ≤12 h). Note the 40 KB minimum billable object size (metadata overhead) and retrieval costs.
- **S3 Glacier Deep Archive** — the bottom rung: ~$0.00099/GB-month, retrieval in **12–48 hours** (Standard/Bulk only). For regulatory archives where "never, but legally required" is the access pattern.
- **S3 Express One Zone** (GA Nov 2023, re:Invent 2023) — the top rung in performance: **up to 10x faster** than Standard, single-digit-millisecond latency, millions of requests per minute, delivered via **directory buckets** colocated with the compute using them (see §10.3). Single-AZ (one AZ), 99.95% availability, and *expensive* per GB (~7x Standard ⚠ approximate). For latency-critical workloads: ML training data loading, HPC, real-time ingestion.

### 6.2 The Class Comparison Table

| Class | Durability | Availability | Access latency | Relative cost (us-east-1, approx ⚠) | Retrieval fee |
|---|---|---|---|---|---|
| Standard | 11 nines | 99.99% | ms | 1.0x (~$0.023/GB) | None |
| Intelligent-Tiering | 11 nines | 99.9% | ms | ~1.0x FA tier + per-object fee | None (per-object monitoring fee) |
| Standard-IA | 11 nines | 99.9% | ms | ~0.55x | Yes (per GB) |
| One Zone-IA | 99.99% (1 AZ) | 99.5% | ms | ~0.43x | Yes |
| Glacier Instant | 11 nines | 99.9% | ms | ~0.17x | Yes |
| Glacier Flexible | 11 nines | 99.99% | min–h | ~0.16x | Yes (+40 KB min object) |
| Glacier Deep Archive | 11 nines | 99.99% | 12–48 h | ~0.04x | Yes |
| Express One Zone | 99.99% (1 AZ) | 99.95% | sub-ms–ms | ~7x | No |

*Prices are order-of-magnitude, not quotes — always check the current price list; see `finops_guide.md` for the cost-governance playbook.*

### 6.3 Class Selection — Which Class for Which Data

The decision procedure, in practice:

1. **Is it latency-critical?** → Standard (or Express if the workload is colocated and performance-bound).
2. **Unknown/variable access pattern?** → Intelligent-Tiering (let it decide).
3. **Known infrequent access, needs ms?** → Standard-IA.
4. **Reproducible data, cost-sensitive?** → One Zone-IA (thumbnails, re-derivable artifacts, staging).
5. **Cold but might be needed in minutes?** → Glacier Instant Retrieval.
6. **Regulatory/cold archive, hours OK?** → Glacier Flexible Retrieval.
7. **Never-but-keep-forever?** → Glacier Deep Archive.
8. **Never decide by hand** — encode this ladder as **lifecycle rules** (§7) so data *drifts* down the classes as it ages.

---

## 7. Lifecycle and Versioning

### 7.1 Lifecycle Rules

A **lifecycle rule** is a declarative policy on a bucket (or prefix/tag subset) that S3 executes automatically: **transition** objects to cheaper classes as they age, and **expire** (delete) them. Rules are defined in days since creation (or since a specified date).

**Verified capabilities:**

- *Transitions:* Standard → IA → Glacier Instant → Glacier Flexible → Deep Archive. Minimum ages apply (e.g., 30 days in Standard before transitioning to IA; 90 days to Deep Archive; Standard-IA minimum 30 days, etc. — ⚠ check current minimums, they change).
- *Expiration:* delete current versions, noncurrent versions, incomplete multipart uploads (a real cost leak if forgotten — orphaned parts bill forever).
- *Filters:* prefix and/or object tags; a rule can target `logs/` or `env=prod`.
- *Noncurrent-version handling:* expire noncurrent versions after N days — this is how you prevent versioned buckets from growing unboundedly.
- *Delivered as:* a bucket-level configuration (XML/JSON), managed via console, CLI, SDK, or IaC (Terraform/CloudFormation).
- *Cost guardrail:* transitions are one-way in the common case (you can transition back manually, but the lifecycle engine only moves data down the cost ladder). Design for the lifecycle — never plan to "rescue" data out of Deep Archive.

### 7.2 Versioning

**Versioning** keeps every version of an object under its key. It is **off by default**; when enabled, each write gets a unique **version ID** and old versions are retained. States:

- **Enabled** — all writes create new versions; DELETEs create a **delete marker** (a tombstone) rather than removing prior versions. Enabling is irreversible as a state choice (you can only *suspend* afterwards).
- **Suspended** — new writes go to the current version (version ID `null`), but existing versions are retained.
- **Never enabled** — one version per key; a DELETE removes it permanently. This is the state in which "oops" means "gone."

**Why versioning is a durability feature:** it is the primary defense against operator error (accidental overwrite/delete) and ransomware-style mass deletion. Combined with **MFA Delete**, a delete requires a second factor and cannot be scripted away.

⚠ **Verify — MFA Delete details:** MFA Delete requires the bucket owner's root credentials (or an MFA-enabled session) to *permanently delete object versions* or to change versioning state; it cannot be applied via IAM-user credentials alone, which is a governance quirk to know when designing operational runbooks.

**Cost implication:** every version is stored and billed. A versioned bucket with hot overwrite patterns grows without bound unless lifecycle expires noncurrent versions — the classic "our S3 bill doubled" story. Always pair versioning with a noncurrent-version expiration rule.

### 7.3 The Lifecycle Table

| Rule | Action | Example |
|---|---|---|
| Transition to IA | Move to Standard-IA after N days | `logs/*` → Standard-IA after 30 days |
| Transition to Glacier | Move to archive class | after 90 days → Glacier Flexible |
| Transition to Deep Archive | Regulatory retention | after 365 days → Glacier Deep Archive |
| Expire current version | Delete the live object | delete raw uploads after 90 days |
| Expire noncurrent versions | Prune version history | keep noncurrent for 90 days, then delete |
| Abort incomplete multipart | Clean up orphaned parts | abort uploads older than 7 days |
| Object Lock (WORM) | Compliance mode: objects immutable | retention 7 years for trade records; legal hold for litigation |

---

## 8. Replication

S3 **Replication** is an *optional, asynchronous, cross-bucket* feature (distinct from the mandatory internal cross-AZ replication of §4.2). It copies objects from a source bucket to one or more destination buckets, in the same or another region. **Versioning must be enabled on both buckets.**

### 8.1 Same-Region Replication (SRR)

- Copies objects between buckets **in the same region** — different accounts, different storage classes, different buckets.
- Use cases: **compliance copies** (a locked copy for auditors), **aggregation** (multiple app buckets → one analytics bucket), **class separation** (hot bucket → replicated into a colder class directly), **test/prod isolation**, and **deletion protection** (replicate to a bucket the operational team cannot touch).

### 8.2 Cross-Region Replication (CRR)

- Copies objects between buckets **in different regions**.
- Use cases: **disaster recovery / geo-redundancy** (a second region holds a copy; if the primary region is impaired you have data in-region elsewhere), **latency locality** (users in APAC read from ap-southeast-1, EU from eu-west-1), and **compliance** (data residency — e.g., a copy inside the EU for GDPR while primary processing stays in Singapore).

### 8.3 Replication Semantics (verified highlights)

- **Asynchronous:** replication typically completes within minutes; there is no consistency SLA on the replica. **S3 Replication Time Control (RTC)** provides a 15-minute objective for 99.99% of objects (⚠ an objective/SLA on the *service*, added Nov 2019).
- Replicates **new objects by default**; existing objects need a *backfill* (S3 Batch Operations `S3ReplicateObject`).
- Delete markers can be replicated (configurable); permanent deletes are **not** replicated (unless you use S3 Object Lock with retention).
- Storage-class transitions on the destination can be set independently (e.g., replicate into Glacier-class directly for DR archives).
- **Replication metrics and events** (`s3:Replication:OperationCompleted*`) feed CloudWatch and EventBridge, so you can monitor lag and failures.
- Encrypted objects replicate; SSE-KMS objects require KMS-key permissions on both sides.

### 8.4 The Replication Table

| Type | Use case | Notes |
|---|---|---|
| SRR | Compliance copy, aggregation, cold-class copy, deletion protection | Same region; versioning required; different account possible |
| CRR | DR, latency locality, data residency | Cross-region; async; RTC 15-min objective |
| RTC | Contractual replication speed | Adds cost; 99.99% of objects within 15 min |
| Backfill | Copy pre-existing objects | S3 Batch Operations |
| Replication events | Monitoring, pipelines | CloudWatch/EventBridge integration |

---

## 9. Security

S3 security is layered: **identity** (who), **policy** (what they may do), **encryption** (how data is protected at rest and in transit), and **access delivery** (how you hand out time-boxed access).

### 9.1 Encryption at Rest — SSE-S3, SSE-KMS, SSE-C

| Option | Key management | Notes |
|---|---|---|
| **SSE-S3** | AWS manages the keys (AES-256) | Default for new buckets since Jan 2023; free; per-object unique keys wrapped by a master key; you cannot rotate/audit keys |
| **SSE-KMS** | AWS KMS customer-managed keys (or AWS-managed `aws/s3`) | Envelope encryption; key rotation, audit via CloudTrail, fine-grained key policies, cross-account control; adds KMS API cost per operation; the default choice for regulated industries |
| **SSE-C** | **You** provide the keys on every request (HTTPS only) | S3 never stores the key; you must supply it per call; key rotation = re-upload or re-encrypt; niche — used when the customer must hold keys |

Plus **in transit:** TLS/HTTPS end-to-end, and **S3 Object Lock** for WORM (write-once-read-many) compliance retention.

**Verified nuance:** since **January 5, 2023**, *all new buckets* get **SSE-S3 (AES-256) encryption by default**, with the ability to specify SSE-KMS at creation; new objects uploaded without an encryption header inherit the bucket default.

### 9.2 Access Control — Bucket Policies, IAM, ACLs

- **IAM policies** — attached to users/roles/groups; the *identity* side ("this role may do X on these buckets"). Used for cross-account access via role assumption.
- **Bucket policies** — attached to the bucket; the *resource* side ("this bucket allows/denies these principals these actions"). Can grant cross-account access directly, enforce conditions (TLS, source IP, VPC endpoint, MFA), and require encryption headers (`s3:x-amz-server-side-encryption`).
- **ACLs** — legacy per-object/per-bucket grants; **disabled by default for all new buckets since April 2023**; AWS recommends IAM + bucket policies instead. Unless you have legacy S3 (2006-era) workloads, do not use ACLs.
- **Access Points** — named network endpoints with their own policies, useful for sharing a bucket across many teams/accounts with distinct permissions (e.g., one bucket, an access point per business unit).
- **Block Public Access** — the belt-and-braces guardrail: four toggles that block public ACLs/policies at bucket and account level. **Enable at account level by default** — it is the single most effective misconfiguration prevention.
- **S3 Object Ownership / bucket owner enforced** — ensures objects written by other accounts become owned by the bucket owner (prevents "orphaned" objects the bucket owner cannot manage).

### 9.3 Presigned URLs

A **presigned URL** is a URL that embeds a valid AWS Signature V4 credential, valid for a limited time, granting whoever holds it the permission to perform one specific operation (GET, PUT, etc.) on one specific object — **without** giving them any AWS credentials.

**Verified facts:**

- Default validity window: **15 minutes to 7 days maximum** (the signature's `X-Amz-Expires` is capped at 7 days when generated via SDK).
- Generated by a principal who already has permission; the URL inherits *their* permissions at generation time.
- Use cases: private-object sharing (reports, invoices), direct-to-S3 uploads from browsers/apps (the client PUTs straight to S3, bypassing your servers), temporary access for partners.
- The signature binds the HTTP method and the exact object key — a presigned PUT URL cannot be reused as a GET.
- ⚠ Security note: presigned URLs are bearer tokens — anyone with the URL can use it until expiry. Do not put them in logs, do not email them around casually, and use short expirations for uploads.

### 9.4 The Security Table

| Control | Purpose | Example |
|---|---|---|
| SSE-S3 | Default encryption, zero effort | default for all new buckets since Jan 2023 |
| SSE-KMS | Auditable, rotatable, fine-grained keys | bank's customer-data bucket with a CMK + key policy limiting access to specific roles |
| SSE-C | Customer-held keys | regulatory requirement to hold keys in a local HSM |
| IAM policy | Identity-side permissions | `s3:GetObject` on `cacib-prod-*/reports/*` for the reporting role |
| Bucket policy | Resource-side + conditions | deny all non-TLS requests; require SSE-KMS header |
| ACL | Legacy grants | off by default (Apr 2023); avoid |
| Block Public Access | Misconfiguration shield | account-level `BlockAllPublicAccess = true` |
| Presigned URL | Time-boxed, credential-free access | 1-hour GET link for an account statement PDF |
| Object Lock (WORM) | Compliance immutability | 7-year retention on trade records |
| CloudTrail + access logs | Auditability | `s3:PutObject` data events → `audit_as_code_guide.md` |

---

## 10. Performance

### 10.1 Multipart Upload

**Multipart upload** splits an object into parts (1–10,000, each 5 MB–5 GB) uploaded **in parallel, in any order**, then completed with a single `CompleteMultipartUpload` call that assembles the object from its parts. It is:

- **Required** for objects > 5 GB (a single PUT caps at 5 GB) — and with the new 50 TB object limit, effectively *mandatory* for anything large (⚠ note: the 50 TB ceiling requires multipart; verify SDK support status for very large objects).
- **Recommended** for anything > ~100 MB: parallel part uploads give near-linear throughput scaling, and a failed part can be retried in isolation without restarting the whole upload.
- **The right tool for streaming and resumable uploads**: you can upload parts as data arrives, pause, resume, and abort cleanly (aborting removes the parts — remember to lifecycle-expire *incomplete* multipart uploads, a classic cost leak).

Each part carries its own ETag; the final object's ETag is derived from the parts' ETags. There is a *minimum* part size of 5 MB (except the last part) — parts smaller than that are rejected at completion time.

### 10.2 Prefix Design and Partitioning

**The 2018 change.** Before **July 17, 2018**, S3 partitioned by key prefix and you had to *randomize* prefixes (e.g., hash prefixes) to spread load — the famous "add random hex to your keys" advice. On that date AWS announced S3 **automatically scales to very high request rates** with no prefix constraints: up to **3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests per second per prefix**, with no limits on the number of prefixes. AWS still recommends a date-based scheme (`2026/08/17/...`) for listing and lifecycle convenience rather than for throughput.

**Modern guidance:**

- **Do not overthink prefixes for performance** — S3 shards and splits automatically (§3.1). Optimize prefixes for *listing, lifecycle, policy, and Athena/Glue partition pruning* instead: `s3://bucket/env=prod/dt=2026-08-17/<data>`.
- **Very hot keys** (one object hammered by millions of requests) will trigger partition splits; the 3,500/5,500 per-prefix guidance still applies per prefix — spread genuinely hot keys across prefixes or put CloudFront in front for read-heavy content.
- **Throughput is a function of parallelism, not object size**: use multipart (write) and ranged GETs / parallel ranged reads (read) with sufficient concurrency; S3 scales with concurrent connections. AWS reports multi-Gbps per client with enough concurrency on modern instances.
- **Express One Zone** removes most of this discussion for the workloads that need it (below).

### 10.3 S3 Express One Zone and Directory Buckets

**Verified (GA November 2023, announced at re:Invent 2023):**

- **S3 Express One Zone** is a storage class + new bucket type (**directory bucket**) designed for latency-critical workloads: **single-digit-millisecond latencies, up to 10x faster data access than S3 Standard**, and support for **millions of requests per minute**.
- Directory buckets are created **within a single AZ**, and you pick an AZ that is *colocated with your compute* (EC2, EKS, SageMaker, Lambda in the same AZ) — the entire design is "put the storage next to the compute."
- Directory buckets use a **regional namespace** (the one exception to the global bucket-name namespace) and a distinct naming scheme (`<bucket-name>--<azid>--x-s3`, e.g., `cacib-ml-data--use1-az4--x-s3`).
- Costs more per GB (⚠ roughly an order-of-magnitude premium over Standard; verify current pricing) and is **single-AZ** (99.95% availability SLA) — it is a *performance* product, not a *durability* one. Replicate or regenerate the data if it must survive an AZ loss.
- Use cases: ML training-data loading, HPC checkpointing, real-time analytics ingestion, ETL shuffles — workloads where data is accessed within seconds-to-minutes of being written, in the same AZ.

### 10.4 The Performance Table

| Technique | Use case | Benefit |
|---|---|---|
| Multipart upload | Objects > 100 MB / > 5 GB | Parallel throughput, resumability, retry isolation |
| Parallel ranged GETs | Large reads, partial extraction | Higher read throughput; supports S3 Select-style filtering |
| Prefix/lifecycle-friendly keys | Listing, retention, Athena pruning | Operational + query efficiency (not throughput, since 2018) |
| Express One Zone + directory buckets | Latency-critical, colocated compute | 10x faster, ms latency, millions of req/min |
| CloudFront in front | Hot read-heavy content | Edge caching, TLS termination, DDoS absorption |
| S3 Transfer Acceleration | Cross-continent uploads | Faster long-distance uploads via edge locations (⚠ extra cost) |
| S3 Object Lambda | Transform on read | No ETL copy; compute only when data is consumed |

---

## 11. API and Integrations

### 11.1 The REST API

S3's API is **HTTP(S)-based REST** (the original SOAP interface was retired in 2006–2007). Every operation is a well-formed HTTP request signed with **AWS Signature Version 4** (except anonymous/public reads):

- `PUT /bucket/key` — upload; `GET /bucket/key` — download; `HEAD` — metadata; `DELETE` — remove.
- `POST /bucket?uploads` / `PUT ?partNumber=` / `POST ?uploadId=` — multipart lifecycle.
- `GET /bucket?list-type=2&prefix=...` — ListObjectsV2.
- `PUT /bucket?lifecycle`, `?versioning`, `?replication`, `?policy`, `?encryption`, `?tagging` — configuration subresources.
- Status codes carry semantics: `200/204`, `206 Partial Content` (ranged GET), `301/307` (region redirect), `403` (denied), `404` (missing), `409` (conflict), `503 SlowDown` (throttle — retry with backoff).

**Verified practical facts:**

- Objects up to 5 GB in a single PUT; everything larger goes through multipart (see §10.1).
- `Range` headers enable partial reads — the basis of resumable downloads, video streaming, and S3 Select-style scanning.
- Conditional requests (`If-Match`/`If-None-Match` with ETags) give you **conditional writes** — the foundation of optimistic concurrency patterns ("only overwrite if the object is still version X").
- Checksums: S3 now defaults to **additional checksums (SHA-256, CRC32/CRC32C/CRC64NVME)** on upload/retrieval, giving end-to-end integrity verification at the API level (the internal checksums of §3.2 are separate).

### 11.2 Event Notifications

S3 can emit **event notifications** when objects change:

- Event types: `s3:ObjectCreated:*` (Put, Post, Copy, CompleteMultipartUpload), `s3:ObjectRemoved:*`, `s3:ObjectRestore:*`, `s3:ObjectTagging:*`, `s3:ObjectAcl:*`, `s3:LifecycleExpiration:*`, `s3:Replication:*`, and `s3:ObjectAccessTierChanged`.
- Destinations: **SNS topics, SQS queues, and Lambda functions** — or the richer **Amazon EventBridge** integration (added 2022) for rule-based routing (filter by bucket/prefix/suffix/size), and S3 EventBridge is now the recommended destination for complex pipelines.
- **At-least-once, near-real-time delivery** (typically within seconds); handlers must be idempotent.
- Notifications are the trigger for most ingestion pipelines: file lands → Lambda validates/transforms → writes catalog entry → downstream analytics (`event_stream_processing_guide.md`).
- ⚠ Note: as of 2025, event notifications to Lambda are generally available for both general-purpose and directory buckets (Express) with some configuration differences; verify current support matrix.

### 11.3 Lambda and SDKs

- **Lambda + S3** is the canonical serverless pair: S3 as the durable event source, Lambda as the compute. Patterns: image/video processing on upload, format conversion, malware scanning, schema validation, fan-out to queues. Lambda can also mount S3 via **Mountpoint for Amazon S3** (2023) for file-interface access, and invoke S3 Object Lambda for read-time transforms (redaction, resizing — see `aws_sap_c02_guide.md` §4.16).
- **SDKs** exist for every mainstream language (Python `boto3`, Java, Go, .NET, JS/TS, Ruby, PHP, Rust, C++), plus the **AWS CLI**, `curl`-able SigV4 signing, and IaC via CloudFormation/Terraform. The S3 API's stability is itself a feature: code written against the 2006 API still runs.
- **S3-compatible tooling** (rclone, s3cmd, MinIO client, most BI tools) means the API is a portable skill, not a vendor lock-in — relevant to the on-prem comparisons in `cephfs_alternatives_guide.md` and `cloud_object_storage_lakehouse_guide.md`.

### 11.4 The API Table

| Feature | Description | Example |
|---|---|---|
| REST API | Signed HTTP operations | `GET /bucket/key` with SigV4 |
| Multipart API | Parallel upload of large objects | 1,000 parts × 100 MB in parallel |
| Conditional writes | ETag-based optimistic concurrency | `If-Match` on config objects |
| Range reads | Partial object retrieval | `Range: bytes=0-1048575` for streaming |
| Event notifications | Object-change events → SNS/SQS/Lambda/EventBridge | `s3:ObjectCreated:*` → Lambda ingest |
| S3 Select / S3 Tables | Query without full download; managed Iceberg tables (Dec 2024) | SQL filter on CSV/Parquet in place |
| SDKs / CLI / IaC | Programmatic and infrastructure control | `aws s3 cp`, `boto3` |
| S3 Object Lambda | Transform data on read | Redact PII per requester role |
| Mountpoint | File interface to S3 for compute | ECS/EKS pods reading S3 as a filesystem |

---

## 12. S3 in the Enterprise

### 12.1 Data Lakes

S3 is the default home of the modern **data lake**: raw and curated data in open formats (Parquet, ORC, Iceberg/Delta tables) with **compute separated from storage** — Athena, Redshift Spectrum, EMR, Glue, and Spark all query S3 directly. Enterprise patterns:

- **Zone structure by prefix:** `raw/` (landing), `curated/` (validated/transformed), `consumption/` (aggregated, analytics-ready) — the medallion architecture, enforced by lifecycle + IAM boundaries per prefix.
- **Open table formats:** Iceberg/Delta/Hudi catalogs stored on S3, with S3 as the single source of truth (`delta_lake_vs_iceberg.md`, `cloud_object_storage_lakehouse_guide.md`). AWS's own managed option, **S3 Tables** (re:Invent 2024), provides serverless Iceberg tables on S3.
- **Governance:** Glue Data Catalog, Lake Formation, tags for cost allocation, and prefix-based retention policies. S3 buckets are the *data estate*; treat bucket layout as schema.

### 12.2 Backups

S3 is the enterprise backup target of choice — either directly (application-to-S3) or via AWS Backup (which stores EBS snapshots, RDS snapshots, FSx, and VMware backups in S3 under the hood):

- **Versioning + lifecycle + cross-region replication** is the canonical backup pattern: current data in Standard, versions retained for N days, noncurrent versions expired after M, a CRR copy in a second region for DR, and a final transition to Glacier/Deep Archive for long-term retention.
- **Cost discipline:** choose the class by RTO — hot backups (Standard-IA), cold backups (Glacier Flexible), regulatory archives (Deep Archive). See `finops_guide.md` for the optimization loop.

### 12.3 Archival and Compliance

- **Regulatory retention** (trade records, statements, audit evidence) maps naturally to Glacier Flexible/Deep Archive + **Object Lock** (WORM) with compliance-mode retention or legal holds — objects cannot be deleted or overwritten by anyone, including root, until the retention period expires.
- **Data residency:** region selection + CRR/SRR give you control over *where* copies live (e.g., an EU-only copy for GDPR while the primary data stays in Asia).
- **Auditability:** CloudTrail *data events* (`s3:PutObject`, `s3:GetObject`), S3 server access logs, and S3 Storage Lens (free 14-day metrics; paid for deep history) feed the evidence pipeline described in `audit_as_code_guide.md`.
- **Electronically stored information (ESI) discovery** in banking/finance typically requires: immutable retention, indexed searchability (Athena over the archive metadata), and defensible deletion — all expressible on S3.

### 12.4 The Enterprise Table

| Use case | Pattern | Classes |
|---|---|---|
| Data lake (raw→curated→consumption) | Prefix zones, Iceberg/Delta, Athena/EMR | Standard / Intelligent-Tiering / IA |
| Backup & DR | Versioning + lifecycle + CRR | Standard-IA + Glacier |
| Regulatory archive | Object Lock WORM, 7-year retention | Glacier Flexible / Deep Archive |
| Compliance copies | SRR to a locked, restricted bucket | Standard or Glacier, immutable |
| Audit trail | CloudTrail data events → S3 → Athena | IA → Glacier |
| Application content | Static assets, uploads | Standard / IA by age |

---

## 13. Worked Example: A Bank's Data Platform on S3

*Scenario: CACIB-style global bank building a **regulatory + analytics data platform** on S3. Constraints: MAS/ECB-style record-keeping (7-year retention for trade records), data residency (Singapore primary, EU copy for European entities), strict access control, and a hard cost budget (storage doubling every 18 months if unmanaged).*

### 13.1 Bucket Design

One bucket per *purpose*, never per team; teams get prefixes and access points. Directory buckets reserved for latency-critical ML ingestion (Express).

| Bucket | Purpose | Layout (prefixes) | Class plan |
|---|---|---|---|
| `cacib-prod-txnlake` | Trade and transaction lake | `raw/`, `curated/`, `consumption/`, `audit/` | Standard → IA → Glacier by age |
| `cacib-prod-backup` | System + DB backups | `rds/`, `ebs/`, `app/` | IA → Glacier; CRR to EU |
| `cacib-prod-archive` | Regulatory WORM archive | `trade-records/`, `statements/`, `evidence/` | Glacier Flexible/Deep Archive + Object Lock |
| `cacib-prod-artifacts` | Builds, reports, shared files | `ci/`, `reports/` | Standard-IA; presigned URLs for distribution |
| `cacib-ml-data--use1-az4--x-s3` (directory bucket) | Feature-store staging, training loads | `features/`, `checkpoints/` | Express One Zone (regenerate-able) |

Naming: `<org>-<env>-<purpose>`; all buckets **SSE-KMS** with per-bucket CMKs; **Block Public Access = on** at account level; versioning **enabled** everywhere with noncurrent-version expiry.

### 13.2 Lifecycle

One lifecycle rule set per bucket, encoding the cost ladder:

1. `raw/` data: Standard for 30 days → Standard-IA at day 30 → Glacier Flexible at day 365 → **expire at day 2,555 (7 years)** to match MAS record-keeping.
2. `curated/` (recomputable): Intelligent-Tiering — access is unpredictable (regulatory queries).
3. `backup/`: IA immediately (restores are rare), Glacier Flexible at day 90, expire at 7 years; noncurrent versions expire after 30 days.
4. `archive/`: **Object Lock, compliance mode, 7-year retention**; Glacier Flexible for the first year (queries possible), Deep Archive after (queries unlikely), with a manual expedited-restore runbook for regulatory requests.
5. All buckets: **abort incomplete multipart uploads after 7 days** — a forgotten 500 GB upload bills forever.

### 13.3 Security

- **SSE-KMS** everywhere, CMK per bucket, key policies restricting `kms:Decrypt` to the analytics/data-platform roles; CloudTrail on `kms:Decrypt` for audit.
- **IAM roles, not users**: `data-platform-analytics` role gets `s3:GetObject` on `raw/`+`curated/`; `ingest` role gets `s3:PutObject` on `raw/` only; `audit` role gets read on `audit/` and nothing else. Bucket policy enforces **TLS only** and denies cross-account access except via defined roles.
- **Presigned URLs** (max 1 hour) for report distribution to business users — no credentials, time-boxed.
- **MFA delete** on the archive bucket (root-credential quirk noted in §7.2 — runbook documented).
- **SRR compliance copy**: `txnlake` replicates to a locked `cacib-prod-txnlake-compliance` bucket (different account) for the auditors; **CRR** to `eu-west-1` for the European entities' data-residency copy.

### 13.4 Cost (approximate — ⚠ flag)

Illustrative 1 PB/year growth at rough us-east-1 prices (verify before use):

| Component | Volume | Class | ~$/GB-mo | Monthly |
|---|---|---|---|---|
| Hot lake (raw 30d, curated) | 80 TB | Standard ~0.023 / IT ~0.02 | ~0.02 | ~$1,600 |
| Warm (IA) | 150 TB | Standard-IA ~0.0125 | 0.0125 | ~$1,900 |
| Cold archive | 700 TB | Glacier Flexible ~0.0036 | 0.0036 | ~$2,500 |
| Deep archive | 70 TB | Deep Archive ~0.00099 | 0.001 | ~$70 |
| Requests/retrievals | — | — | — | ~$300 |
| KMS (SSE-KMS per-op) | — | — | — | ~$150 |
| **Total** | ~1 PB | — | — | **~$6,500/mo** |

*Real figures depend on region, request mix, and the 40 KB minimum-object overhead in Glacier classes (many small objects inflate archive bills — batch small records into larger files before transition). The same data in pure Standard would be ~$20k+/mo: **lifecycle is the FinOps lever** (`finops_guide.md`).*

### 13.5 Lessons

- **Design for the lifecycle.** The bucket layout, key scheme, and class ladder were decided *before* the first byte landed. Retro-fitting retention to an ungoverned lake is expensive and risky.
- **The namespace is global, the data is regional.** Name collisions and residency requirements shape naming and replication from day one.
- **Versioning is a durability feature and a cost feature** — pair it with noncurrent-version expiry or the bill grows forever.
- **Assume the operators are fallible.** MFA delete, Object Lock, Block Public Access, and least-privilege roles are the actual protection; 11 nines does not protect you from a careless recursive delete.
- **Test the restore path.** Glacier restore runbooks and RTC-monitored CRR should be exercised, not assumed — an archive you cannot retrieve within the regulatory deadline is not an archive.

---

## 14. Summary: S3 in One Page

**The model.** Buckets (globally named containers) hold objects (key + value + metadata + version ID) in a flat, prefix-conventioned namespace. There is no hierarchy, no locking, no file system — which is exactly why it scales: S3 shards key ranges automatically and acknowledges writes only once durably stored.

**The guarantees.** 99.999999999% durability (design target) via multi-AZ synchronous replication + internal erasure coding + end-to-end checksums + self-healing; 99.99% availability (SLA) for Standard; strong read-after-write consistency for all operations since December 2020. One AZ loss is survivable by design; operator error is not — versioning, MFA delete, and Object Lock close that gap.

**The classes.** Eight rungs from Standard (ms, hot) through Intelligent-Tiering (automatic), IA pair (cheap warm), Glacier trio (minutes-to-days archives), to Express One Zone (10x faster, single-AZ, expensive). Lifecycle rules move data down the ladder automatically; versioning and replication protect it; SSE-KMS + IAM/bucket policies + presigned URLs + Block Public Access control it.

**The final word.** S3 is the storage that scales to the internet — the twenty-year-old service whose original design (flat namespace, HTTP API, utility pricing, operational invisibility) turned out to be the right abstraction for data lakes, backups, archives, AI training sets, and most of what "the cloud" stores. Architecting on S3 is not about the service; it's about *deciding the lifecycle, security, and naming* — and letting the platform do the rest.

---

## Glossary

- **S3 / Amazon Simple Storage Service** — AWS's object storage service, launched March 14, 2006; "storage for the Internet."
- **Bucket** — container for objects; global namespace (general-purpose), regional data; unit of config/billing.
- **Object** — the stored entity: key + value + metadata (+ version ID); atomic read/write unit; up to 5 TB (50 TB since Dec 2025).
- **Key** — the object's unique name in a bucket (flat string, ≤1,024 bytes).
- **Prefix** — key substring used for grouping, listing, policies, lifecycle, and query pruning (a "folder" convention).
- **Metadata** — system (Content-Type, ETag, …) + user (`x-amz-meta-*`, ≤2 KB) attributes of an object.
- **Version ID** — unique identifier of an object version when versioning is enabled.
- **Durability** — the probability data survives (no loss); S3's design target is 99.999999999%.
- **Availability** — the probability the service is usable; S3 Standard SLA is 99.99%.
- **11 nines** — 99.999999999% durability, the multi-AZ S3 design target.
- **Erasure coding** — redundancy scheme (data + parity stripes) used internally by S3; exact parameters unpublished ⚠.
- **AZ (Availability Zone)** — an isolated failure domain (power/network/cooling) within an AWS region; S3 multi-AZ classes span ≥3 AZs.
- **Strong consistency** — reads reflect completed writes; S3-wide since Dec 2, 2020 (read-after-write for GET/PUT/LIST/DELETE).
- **Read-after-write consistency** — a read immediately after a successful write returns the new data.
- **Storage class** — pricing/performance tier of an object (Standard … Deep Archive … Express).
- **S3 Standard** — hot, ms-access, 11-nines/99.99% default class.
- **Intelligent-Tiering** — auto-tiers by access pattern; per-object monitoring fee.
- **Standard-IA** — infrequent access, ms latency, retrieval fee.
- **One-Zone-IA** — single-AZ IA class; 99.99% durability, 99.5% availability; reproducible data only.
- **Glacier** — the original archive class, renamed **Glacier Flexible Retrieval** (Nov 2021); retrieval minutes–hours.
- **Glacier-Flexible Retrieval** — archive class with Expedited/Standard/Bulk retrieval options.
- **Glacier-Deep Archive** — cheapest archive (~$0.00099/GB-mo); 12–48 h retrieval; regulatory "never but keep."
- **S3 Express One Zone** — 2023 performance class: 10x faster, ms latency, single AZ, directory buckets.
- **Directory bucket** — Express bucket type, regional namespace, colocated with compute in one AZ.
- **Lifecycle** — bucket rules that transition objects between classes and expire them by age.
- **Transition** — lifecycle action moving an object to a cheaper storage class.
- **Versioning** — retaining all versions of a key (enabled/suspended); DELETE creates a delete marker.
- **MFA delete** — multi-factor requirement to permanently delete versions / change versioning state.
- **Replication** — async, cross-bucket copy of objects (SRR/CRR); requires versioning.
- **SRR (Same-Region Replication)** — replication within one region (compliance copies, aggregation).
- **CRR (Cross-Region Replication)** — replication across regions (DR, latency locality, residency).
- **SSE-S3** — server-side encryption with AWS-managed keys (AES-256); default for new buckets.
- **SSE-KMS** — server-side encryption with KMS keys (envelope encryption, auditable, rotatable).
- **SSE-C** — server-side encryption with customer-provided keys (supplied per request; not stored).
- **Bucket policy** — resource-side JSON access policy on a bucket (conditions, cross-account).
- **IAM** — identity and access management: users/roles/groups with permissions (identity side).
- **ACL** — legacy access control list (per-object grants); disabled by default for new buckets since Apr 2023.
- **Presigned URL** — time-limited (≤7 days) signed URL granting one operation on one object, no credentials.
- **Multipart upload** — parallel, resumable upload (parts 5 MB–5 GB, ≤10,000 parts); required > 5 GB.
- **Prefix (performance)** — historically a partitioning axis; since July 2018, automatic scaling removed the random-prefix requirement.
- **Event notification** — S3 → SNS/SQS/Lambda/EventBridge message on object events (`s3:ObjectCreated:*` etc.).
- **Lambda** — serverless compute; the canonical S3 event consumer and processing engine.
- **REST API** — HTTP(S) + SigV4 interface for all S3 operations.
- **SDK** — language client library (boto3, etc.) and CLI for S3.
- **Data lake** — central repository of raw/curated data in open formats, with compute separated from storage; S3 is the canonical home.
- **Backup** — durability copy of data (versioning + lifecycle + replication patterns on S3).
- **Archival** — long-term, low-cost retention (Glacier classes, Object Lock WORM).

---

*End of guide. Verified facts: S3 launch (Mar 14, 2006); strong consistency (Dec 2, 2020); storage-class names and availability SLAs; Express One Zone GA (Nov 2023, directory buckets); multipart limits (5 GB PUT, 5 MB–5 TB/50 TB objects, 10,000 parts); SSE default encryption (Jan 2023); ACLs off by default (Apr 2023); max object size 50 TB (re:Invent 2025). Flagged as approximate/unverified: exact erasure-coding parameters and internal partition thresholds, fleet scale figures, and all prices. Cross-references: `aws_sap_c02_guide.md`, `cloud_providers_guide.md`, `cloud_object_storage_lakehouse_guide.md`, `cephfs_alternatives_guide.md`, `dell_objectscale_guide.md`, `finops_guide.md`, `audit_as_code_guide.md`, `advanced_analytics_solutions_guide.md`, `event_stream_processing_guide.md`, and the `data/` lakehouse guides.*
