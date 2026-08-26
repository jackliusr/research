# Architecting Systems for Billion-Level Users

> A comprehensive reference on designing, scaling, and operating internet-facing systems
> that serve one billion or more users. Drawing on real-world architectures from Meta,
> Google, Netflix, Twitter/X, TikTok, WhatsApp, Uber, and LinkedIn.

---

## Table of Contents

1. [The Scale Challenge](#1-the-scale-challenge)
2. [Foundational Principles](#2-foundational-principles)
3. [Architecture Layers](#3-architecture-layers-top-to-bottom)
   - 3.1 Client & Edge Layer / 3.2 API Gateway & Load Balancing
   - 3.3 Application / Service Layer
   - 3.4 Data / Caching Layer
   - 3.5 Messaging & Async Processing / 3.6 Storage Layer
4. [Scaling Data: From Single Node to Billion Users](#4-scaling-data-from-single-node-to-billion-users)
5. [Real-World Case Studies](#5-real-world-case-studies)
   - 5.1 Meta (TAO) / 5.2 Google (Spanner & Borg)
   - 5.3 Netflix (Chaos Engineering) / 5.4 Twitter/X (Fanout)
   - 5.5 WhatsApp (Erlang) / 5.6 TikTok (PostgreSQL) / 5.7 Uber (DOMA)
6. [Technical Deep Dives](#6-technical-deep-dives)
7. [Key Design Patterns Reference](#7-key-design-patterns-reference)
8. [Cloud-Native Technologies Stack](#8-cloud-native-technologies-stack)
9. [Anti-Patterns to Avoid](#9-anti-patterns-to-avoid)
10. [Data Sources & Further Reading](#10-data-sources--further-reading)

---

## 1. The Scale Challenge

### What "Billion Users" Means in Numbers

| Metric | Value | Context |
|--------|-------|---------|
| **MAU** | 1,000,000,000+ | Facebook 3B+, YouTube 2.5B+, WhatsApp 2B+, WeChat 1.3B+, TikTok 1B+ |
| **DAU** | 300M–2B | 30–50% of MAU for social; 50–70% for messaging |
| **QPS (avg)** | 100K–10M+ | 1B users × 1 req/min = ~16.7M QPS |
| **Peak/Avg QPS ratio** | 3–5× (10–15× for events) | Super Bowl, New Year's, product launches |
| **Data storage** | PB–EB | 1 PB metadata; 100s PB media; EB-scale logs |
| **Bandwidth** | 10+ Tbps | Streaming: 100+ Tbps; social: 10+ Tbps |
| **P99 latency (web)** | <100 ms | First-byte for API responses |
| **P99 latency (real-time)** | <10 ms | Messaging, gaming, trading |
| **Availability target** | 99.99%+ | "Four nines" = 52.56 min downtime/year |
| **Read-to-write ratio** | 100:1 (feeds) / 1:1 (messaging) | Drives caching and sharding strategy |

### Back-of-Envelope Calculations

```
For 1B MAU, 500M DAU, each user makes 100 requests/day:
  Avg QPS = 500M × 100 / 86,400 ≈ 579K QPS
  Peak QPS = 579K × 3 ≈ 1.7M QPS
  Bandwidth (50 KB/response) = 1.7M × 50 KB × 8 = 680 Gbps peak
  Metadata storage (1 year) = 1B × 1 KB = 1 TB
  Media storage (1 year) = 1B × 100 KB × 12 = 1.2 PB
  Logs/analytics = 1B × 10 KB/day × 365 = 3.65 PB/year
```

**Key trade-off**: At 10M+ QPS, a 1ms CPU optimization per request saves thousands of server-hours per day.

---

## 2. Foundational Principles

**Horizontal scaling**: Add commodity machines, not bigger ones (48–128 cores, 256–512 GB RAM, NVMe SSDs). This gives linear cost scaling, fault isolation, and avoids the vertical ceiling.

**Stateless services**: Application servers hold zero durable state. All session data, cache, and persistence lives in dedicated data-tier services. Stateless services can be scaled up/down arbitrarily, restarted without data loss, and rolled out gradually.

**Asynchronous communication**: Prefer event-driven messaging over synchronous RPC for non-critical paths. Kafka/Pulsar decouples producers from consumers, absorbs traffic spikes, and enables independent scaling.

**Graceful degradation**: Serve cached/static versions when DB is unreachable. Disable non-critical features (comments, search, likes) under extreme load. Show "read-only mode" for financial systems.

**Defense in depth**: Rate limiting (edge → gateway → service), circuit breakers (fail fast on unhealthy dependencies), bulkheads (separate thread/connection pools), retry with exponential backoff + jitter (avoid thundering herd), request timeouts at every hop.

**Observability**: Metrics (Prometheus/Graphite), tracing (OpenTelemetry/Jaeger), logging (Loki/Elasticsearch) at every layer. SLO tracking with error budgets.

**Automation**: Infrastructure as Code (Terraform/Pulumi), auto-scaling (HPA/target tracking), self-healing (auto-restart, health checks), chaos engineering (proactive failure injection).

**Cost-aware engineering**: At billion-user scale, every micro-optimization has real P&L impact. Per-service cost dashboards, regular efficiency reviews, tiered storage strategies.

---

## 3. Architecture Layers (Top to Bottom)

### 3.1 Client & Edge Layer

**CDN**: Akamai, Cloudflare, Fastly, CloudFront, Cloud CDN. Serve static assets from edge nodes near the user. Advanced: dynamic content acceleration (Fastly VCL, Cloudflare Workers), API response caching. Akamai serves 15–30% of all web traffic; Cloudflare handles 25M+ HTTP requests/second.

**DNS load balancing**: Geo-based routing (nearest region), latency-based routing (lowest measured latency), weighted round-robin (gradual traffic shifting), Anycast (same IP advertised from multiple locations — BGP routes to nearest).

**Edge computing**: Cloudflare Workers (V8 isolates, 5ms cold start, 275+ cities), Fastly Compute@Edge (WASM, sub-ms cold start), Lambda@Edge (Node/Python on CloudFront events). Use cases: auth checks, A/B testing, header manipulation, bot detection.

**Mobile-specific**: Protocol Buffers (10× smaller than JSON), WebP/AVIF adaptive image compression, DASH/HLS adaptive video bitrate, offline-first (local SQLite/Realm → delta sync on reconnect).

### 3.2 API Gateway & Load Balancing

**Global LB (L4/L3)**: DNS-based → Anycast IP → regional balancers. Health-aware (removes unhealthy regions from DNS). DDoS scrubbing at edge.

**Regional LB (L7)**: Nginx, HAProxy, Envoy, AWS ALB/NLB, GCP HTTP LB. Functions: TLS termination (offload from app servers), path-based routing (`/api/*` → service A, `/static/*` → CDN), connection pooling.

**API Gateway**: Kong (50K+ RPM throughput, Lua/Go plugins), Apigee (enterprise API management), AWS API Gateway (100K+ RPS, WebSocket), Envoy (100K+ req/s/core). Functions: auth, rate limiting, request/response transformation, caching, quota management, API versioning.

**GraphQL federation / BFF**: Apollo Federation composes multiple graph services into a single endpoint. BFF provides one backend per client type (iOS, Android, Web). Reduces N+1 requests, eliminates over-fetching. Trade-off: gateway caching complexity, increased operational burden.

**Real-world**: Facebook uses Proxygen (custom C++ async HTTP server, millions of persistent connections). Google uses GFE (custom L7 LB with TLS, DDoS, routing). Netflix uses Zuul 2 (Netty async, hundreds of instances).

### 3.3 Application / Service Layer

**Microservices**: Domain-driven decomposition with bounded contexts. Each service owns its data (database-per-service). Services communicate via REST/gRPC (sync) or events (async). Independent deploy, scale, and failure domains.

**Service mesh**: Istio (Envoy sidecars — traffic mgmt, mTLS, observability), Linkerd (Rust-based "linkerd2-proxy", lightweight), Consul (service discovery + mesh). Handles retry/timeout, circuit breaking, traffic splitting (canary), fine-grained policy.

**Orchestration**: K8s (5K nodes/cluster default, pushable to 15K+). Meta runs 10K+ node K8s clusters. Google evolved Borg → Omega → K8s (Borg manages millions of tasks). Spotify runs 15K+ nodes, 1,000+ microservices. Scale K8s via federation (KubeFed) or cell-as-cluster model.

**Cell-based architecture**: Divide users into isolated full-stack "cells" (each with its own app tier, cache, DB shard, supporting services). Blast radius: failure in one cell doesn't affect others. Independent rollouts, canary testing (route 1% of users to new cell). Used by Facebook, YouTube, Uber. Typical cell size: 10M–50M users.

**Architecture evolution paths**: Google (B4 → Borg → Omega → K8s). Meta (PHP monolith → SOA → TAO → microservices). Twitter (Rails → Scala → JVM microservices). Netflix (Java monolith → cloud → microservices → chaos). Uber (Python/Node monolith → 2,200 microservices → DOMA layered domains).

### 3.4 Data / Caching Layer

#### Caching

**Multi-tier cache**: L1 (application memory, <1ms, GB) → L2 (local Redis/Memcached sidecar, 1–3ms, 10s GB) → L3 (distributed cluster, 3–10ms, TB) → L4 (CDN edge, 1–50ms, TB).

**Cache strategies**: Cache-aside (read cache → miss → read DB → populate cache; write DB, invalidate cache — most common for read-heavy). Read-through (cache loads from DB on miss). Write-through (cache writes DB synchronously). Write-behind (cache writes DB async — best write throughput). Write-around (write directly to DB, cache populates on read).

**Cache invalidation**: TTL-based (simplest, serves stale data). Event-driven (publish invalidation to message queue). Version-based (increment global counter, cache misses on mismatch). Meta uses TAO "lease" mechanism — cache-coherence protocol between TAO servers.

**Cache warming**: Pre-populate caches for hot keys (celebrities, trending content, viral posts). Use historical traffic patterns to identify hot keys. Gradual warm-up to avoid cache-stampede. Meta's `McDipper` tool warms Memcached clusters before traffic cutover.

**Real-world**: Meta runs thousands of Memcached servers, billions of req/s (100K+ QPS per server). Twitter runs Twemcache (forked Memcached) with hundreds of servers. LinkedIn uses Redis for real-time analytics aggregation.

#### Database

**Read replicas**: Scale read capacity horizontally. MySQL/PostgreSQL async replication (seconds lag). Aurora: 15 replicas, ~100ms lag. Trade-off: eventual consistency for replica reads.

**Database sharding strategies**:

| Strategy | Mechanism | Pros | Cons |
|----------|-----------|------|------|
| **Range-based** | By ID range (0–1M → shard 0) | Simple, range queries work | Hot spots, uneven, rebalancing hard |
| **Hash-based** | hash(user_id) % N | Even distribution | Full re-shard on N change |
| **Consistent hashing** | Hash ring with virtual nodes | Minimal re-sharding on add/remove | Complexity, slight unevenness |
| **Directory-based** | Lookup table key→shard | Max flexibility | Lookup overhead, mapper SPOF |
| **Geo-sharding** | By region | Latency, compliance (GDPR) | Cross-region queries expensive |

**Shard key**: `user_id` (most common — even distribution). `tenant_id` (B2B/SaaS co-location). `geo_hash` (location-based services). Wrong key → hot spots, cascading failures, painful re-sharding.

**Database-per-service**: Each microservice owns its database — no service reads another's DB directly. Communication only through the owning service's API.

**Polyglot persistence**: Relational (PostgreSQL, MySQL, Aurora — ACID, complex queries). Document (MongoDB, DynamoDB — flexible schema). Key-value (Redis, Memcached — high throughput). Wide-column (Cassandra, Scylla, Bigtable — time-series, high write). Graph (Neo4j — social graphs, fraud). Search (Elasticsearch — full-text). Time-series (InfluxDB, TimescaleDB). Blob (S3, GCS). Message queue (Kafka, Pulsar, RabbitMQ).

**Real-world stacks**: Meta — MySQL (sharded) + TAO graph cache + Memcached + Haystack (blob). Google — Spanner + Bigtable + Colossus (GFS). Twitter/X — Manhattan (KV) + MySQL + Redis + Blobstore. Netflix — DynamoDB + RDS + EVCache + S3. TikTok — PostgreSQL + Redis + S3. Uber — MySQL (Schemaless) + Cassandra + Redis. LinkedIn — MySQL (sharded) + Voldemort (KV) + Redis + Kafka (7T msg/day).

### 3.5 Messaging & Async Processing

**Message queue comparison**: Kafka (1M+ msg/s, ~2ms latency, durable, replayable by offset) — industry standard for event streaming. RabbitMQ (50K+ msg/s, <1ms latency) — reliable task distribution. Pulsar (1M+ msg/s, ~5ms latency, segment-centric storage, built-in geo-replication) — multi-tenant streaming. AWS SQS/SNS (unlimited scale, managed) — serverless apps.

**Event-driven architecture**: Services communicate via events (UserCreated, OrderPlaced). Producer doesn't know who consumes. Decouples services completely.

**Event sourcing**: Store sequence of state-changing events as source of truth. Current state derived by replaying events. Benefits: complete audit trail, temporal queries, easy to add read models. Cost: more storage, replay time.

**CQRS**: Separate read and write models. Writes → command model (normalized, consistent). Reads → read model (denormalized, optimized). Updated async via events. Trade-off: eventual consistency between models.

**Real-world**: LinkedIn's Kafka grew from 1B events/day (2011) to 7T messages/day (2019). Uber runs Kafka as central event backbone for dispatch, pricing, notifications. Netflix uses Kafka for async encoding, recommendations, telemetry.

### 3.6 Storage Layer

**Object storage**: AWS S3 (11 9's durability, ~$0.023/GB/mo), GCP GCS (~$0.020/GB/mo), Azure Blob (~$0.018/GB/mo), Cloudflare R2 (no egress fees). Max object size ~5 TB.

**Distributed file systems**: HDFS (100s PB, batch analytics). Ceph (unified block/file/object, self-healing). Meta's Haystack (exabyte-scale photo blob store — sequential needle/haystack format for optimal HDD). Google's Colossus / GFS v2 (10M+ disks, exabyte-scale, used by YouTube, Gmail, Search).

**Cold/archival storage**: AWS Glacier Deep Archive (~$0.001/GB/mo, 12–48 hr retrieval). GCP Archive (~$0.0012/GB/mo). Azure Archive (~$0.00099/GB/mo). Tape (LTFS) at ~$0.001/GB media cost.

**Data lifecycle**: Hot (CDN/cache, <10ms) → Warm (SSD/DB, 10–100ms, ~90 days) → Cold (HDD/object storage, 100ms–1s, 90d–1yr) → Archive (tape/Glacier, minutes–hours retrieval, compliance/legal hold).

---

## 4. Scaling Data: From Single Node to Billion Users

**Phase 1: Single Server (0–10K users)** — Monolith + single DB + everything in one process. Limit: ~10K concurrent.

**Phase 2: Vertical Split (10K–100K users)** — Separate web server + DB server. Add dedicated cache (Redis/Memcached). CDN for static assets. Limit: DB write bottleneck.

**Phase 3: Horizontal Split (100K–1M users)** — Multiple app servers behind load balancer. DB primary + read replicas. Distributed cache cluster (Redis Cluster). Static files → CDN + S3. Challenges: distributed session management, DB write bottleneck.

**Phase 4: Microservices & Sharding (1M–10M users)** — Domain-bounded microservices. DB sharding (horizontal partitioning). Service mesh. Async messaging (Kafka). Challenges: distributed transactions, observability complexity.

**Phase 5: Global Scale (10M–100M users)** — Multi-region deployment. Geo-distributed DB (Spanner, CockroachDB, Cassandra). Global CDN + edge computing. Cell-based architecture per region. Cross-region replication for DR. Challenges: cross-region latency, eventual consistency, regulatory compliance (GDPR).

**Phase 6: Billion+ Users** — Multi-cell, multi-region, multi-cloud. Custom-built infrastructure (proxies, caches, LBs). ML-driven optimization (traffic prediction, capacity planning, anomaly detection). Chaos engineering at scale (regular GameDays). Examples: Facebook (3B+), Google (multiple billion-user products), WeChat (1.3B+).

---

## 5. Real-World Case Studies

### 5.1 Meta (Facebook) — The TAO System

**Scale**: 3B+ MAU, billions of photos uploaded/month, petabytes/day.

**TAO (The Associations and Objects)**: Distributed graph cache sitting in front of sharded MySQL. Handles >1B reads/sec and millions of writes/sec. Objects = entities (users, photos), Associations = relationships (likes, tags). TAO caches the graph in-memory with a lease-based cache-coherence protocol.

**MySQL foundation**: Thousands of shards across thousands of servers. Logical sharding by `user_id` (~100GB/shard). Custom MySQL fork with async replication patches.

**Memcached**: Thousands of servers, billions of req/s. Custom fork (McRouter) provides consistent hashing + connection pooling.

**Proxygen**: Custom C++ async HTTP server — millions of concurrent connections, configurable LB, adaptive timeouts, QoS.

**Other infrastructure**: Scuba (real-time in-memory DB for operational analytics — 10K+ servers, sub-second on petabytes). Presto (distributed SQL on Hive/HDFS data lake). Haystack (exabyte-scale photo blob store).

**Key lesson**: Build custom infrastructure when off-the-shelf doesn't scale. Caching is the entire critical path.

### 5.2 Google — Spanner & Borg

**Scale**: Search (billions queries/day, 100B+ pages indexed), YouTube (2B+ MAU, 500+ hrs/min uploaded), Gmail (1.5B+ users), Maps (1B+ users).

**Spanner**: Globally distributed relational DB with external consistency. Uses TrueTime API (GPS + atomic clocks) for globally consistent timestamps. Auto-failover, geo-replication to 8+ regions. SQL + ACID at global scale.

**Bigtable**: Wide-column NoSQL — Search, Maps, YouTube, Analytics. Petabyte-scale. Original inspiration for HBase and Cassandra.

**Borg/Omega/K8s**: Borg (monolithic scheduler, millions of tasks across 10K+ servers). Omega (shared-state, optimistic concurrency). K8s (open-source evolution).

**GFE (Google Front End)**: Custom L7 LB — TLS, DDoS mitigation, protocol translation, request routing on commodity servers.

**Chubby**: Distributed lock service (Paxos-based). Leader election, config management, namespace coordination.

**Colossus (GFS v2)**: 10M+ disks, exabyte-scale. Replication, checksumming, auto-recovery. Foundation for all Google storage.

**Key lesson**: Strong consistency at global scale is possible (Spanner). Build your own stack at Google scale.

### 5.3 Netflix — Chaos Engineering at Cloud Scale

**Scale**: 260M+ subscribers, 2B+ hrs/month streamed, 100+ Tbps peak bandwidth.

**AWS-native**: Unusual at this scale. 1000s of EC2 instances, 100s of RDS/Aurora databases, S3 for media. CloudFront + Open Connect (CDN appliances at ISP peering — 95%+ traffic delivered within ISP networks, eliminates transit costs).

**Zuul 2**: Netty async gateway — 100+ instances, 10K+ req/s/instance. Routing, rate limiting, auth, observability.

**Eureka**: Service discovery — services register on startup, clients cache registry, refresh ~30s.

**Hystrix**: Circuit breaker — wraps remote calls, opens circuit at threshold (e.g., 50% errors in 5s), half-open for retry.

**Simian Army**: Chaos Monkey (randomly terminates EC2 instances in production). Latency Monkey (injects delays). Conformity Monkey (finds non-standard configs). Janitor Monkey (cleans unused resources). Security Monkey (finds vulnerabilities).

**Spinnaker**: Multi-cloud continuous delivery — deploy to 100s of instances, multi-region, canary analysis, auto-rollback.

**Key lesson**: Embrace failure (Chaos Monkey). Cloud works at extreme scale with right architecture. CDN appliances at ISPs slash bandwidth costs.

### 5.4 Twitter/X — Fanout Architecture

**Scale**: 500M+ tweets/day (peak), ~330M MAU, millions of QPS timeline delivery.

**Hybrid fanout**: Fanout-on-write (pre-populate tweet in all followers' timelines) for users with <1K followers. Fanout-on-read (fetch tweets at read time) for celebrities (100M+ followers). Merge cached celebrity tweets with fanout tweets. Threshold: ~1,500 followers. **Canonical example of optimizing write vs read paths by user characteristics.**

**Manhattan**: Custom distributed KV store on MySQL/Redis. Offers Redis-like API with persistent storage. Geo-replicated, auto-sharded.

**Twemcache**: In-house Memcached fork with auto-eviction, statistics, connection management.

**Timeline service**: Walks social graph, fetches tweets from fanout cache, merges with celebrity read-time tweets, applies visibility filtering. P99 <500ms.

**EventBus**: Kafka-based real-time event stream — billions of events/day for search indexing, analytics, ML, real-time signals.

**Key lesson**: Hybrid fanout strategies save massive compute and storage. Custom KV stores when cache+DB complexity peaks.

### 5.5 WhatsApp — Minimalist Erlang at 2B Users

**Scale**: 2B+ MAU, 100B+ messages/day, ~55 employees at acquisition (2014).

**Erlang/OTP**: Fault-tolerant telecom-grade concurrency. 2M+ concurrent connections per commodity server. Hot code swapping (zero-downtime upgrades). "Let it crash" philosophy — supervisor trees auto-restart.

**FreeBSD**: Custom-tuned kernel — ~1M TCP connections per server. TCP state in kernel instead of user-space.

**Custom Ejabberd (XMPP fork)**: Replaced XML with binary protocol. Optimized for mobile (low bandwidth, intermittent connectivity).

**End-to-end encryption** (Signal Protocol): Server stores encrypted blobs only. Scaling is about message routing, not data access.

**Small team efficiency**: ~50 engineers for 2B users = 40M users/engineer. Erlang's concurrency + simple product + deep optimization.

**Key lesson**: Language choice matters at scale. Simple product + deep optimization = billion-user scale with small teams.

### 5.6 TikTok/ByteDance — PostgreSQL at Hyperscale

**Scale**: 1B+ MAU, massive video upload/consumption, viral traffic patterns.

**Core stack**: PostgreSQL + Redis + S3. PG for relational data, Redis for cache/session, S3 for videos. Multi-region, multi-cell architecture.

**For You Page (FYP) algorithm**: Real-time ML recommendation pipeline. 10K+ GPU nodes for feature extraction, model inference, ranking.

**Cold start optimization**: Preload ML models, cache user preferences, predict content before first swipe.

**Content moderation**: AI-powered video scanning at upload time — processes millions of uploads/day with sub-second decision latency.

**Regional data compliance**: Data stored in-region per GDPR, India data localization, US requirements.

**Key lesson**: PostgreSQL works at billion-user scale with proper sharding/caching. ML at scale is a first-class infrastructure concern, not an add-on.

### 5.7 Uber — Domain-Oriented Microservices

**Scale**: 100M+ MAU, 25M+ trips/day, 2,200+ microservices.

**DOMA (Domain-Oriented Microservice Architecture)**: Layers — Infrastructure → Business → Product → Aggregate. Each layer only talks to the layer below through well-defined interfaces. Prevents chaotic microservice coupling.

**Schemaless (MySQL-based)**: Custom datastore on sharded MySQL. JSON columns, auto-sharding, auto-failover. Handles ride, geolocation, and pricing data.

**Dispatch optimization**: Geospatial search (find nearest drivers). Bipartite graph matching. Dynamic surge pricing. Compute-intensive real-time optimization.

**Ringo**: Custom geo-distributed KV store for real-time location — 100K+ location updates/second per cluster.

**Key lesson**: Microservices at 2K+ need governance — DOMA prevents chaos. Geospatial workloads require specialized databases. Monolith → microservices is a multi-year journey.

---

## 6. Technical Deep Dives

### 6.1 Database Sharding Strategies

**Range-based**: `user_id 0–1M → shard 0`. Simple, range queries efficient. Hot spots (new users in current range). Massive rebalancing on expansion.

**Hash-based**: `hash(user_id) % N`. Even distribution. Full re-shard on N change (all data re-hashed).

**Consistent hashing**: Hash ring with virtual nodes (e.g., 256 vnodes/node). Only ~1/N fraction moves on add/remove. Used by Cassandra, DynamoDB, Riak, ScyllaDB. Not perfectly even without enough vnodes.

**Directory-based**: Lookup table maps key → shard. Maximum flexibility — online re-sharding, per-key migration. Lookup overhead (extra query), mapper is SPOF without replication.

**Geo-sharding**: `shard_id = geo_region(major_city)`. Data locality, compliance-friendly. Cross-region queries expensive. Used by Spanner, CockroachDB.

### 6.2 Consistency Models

| Model | Guarantee | Examples |
|-------|-----------|----------|
| **Strong (linearizability)** | Read returns latest write | Spanner, ZooKeeper, etcd |
| **Read-your-writes** | User sees own writes | Most web apps |
| **Monotonic reads** | Reads never go backward | Cassandra (tunable) |
| **Eventual** | Converges over time, may be stale | DNS, Cassandra, DynamoDB |
| **Causal** | Causally-related events consistent | COPS, Eiger |

**CAP trade-offs**: Financial transactions → consistency. Social feed → availability. DNS → availability (old IP > no IP). User settings → consistency.

**Quorum-based (Dynamo)**: N=3, W=2, R=2 gives strong consistency (W+R > N). W=1,R=1 gives eventual consistency with max availability. N=3,W=3,R=1 = strong but low availability.

### 6.3 Rate Limiting at Scale

**Algorithms**: Token bucket (simple, burst-friendly). Leaky bucket (smooth output, no spikes). Fixed window (memory efficient, edge-case bursts). Sliding window log (precise, memory heavy). GCRA (very precise, complex).

**Distributed approaches**: Centralized Redis (accurate, adds latency). Local + background sync (zero overhead, slightly inaccurate). Fixed per-server quotas (no coordination, inefficient with imbalanced traffic).

**Multi-level**: Level 0 (edge/CDN → IP-based DDoS). Level 1 (global LB → per-IP coarse throttle). Level 2 (API gateway → per-user per-endpoint business rules). Level 3 (service → circuit breaker, backpressure). Level 4 (database → connection pool limits).

**Real-world**: Twitter uses Redis sliding-window counters per authenticated user. Google uses token bucket at GFE with per-project and per-user limits.

### 6.4 Observability at Scale

| Pillar | Collection | Storage | Cost at Scale |
|--------|-----------|---------|--------------|
| **Metrics** | Pull (Prometheus) / Push (StatsD) | TSDB (Thanos 3× compression) | Moderate |
| **Tracing** | OpenTelemetry SDK (sampled) | Jaeger, Tempo, Zipkin | High — sampling mandatory |
| **Logging** | Fluentd, Logstash, vector | ES, Loki | Very high at full fidelity |

**Sampling**: Head-based (always sample errors/slow requests — cheap, biased). Tail-based (store all traces temporarily, keep by criteria — accurate, expensive). Adaptive (100% at low load, 1% at peak). Rule-based (100% errors, 10% slow, 0.1% fast).

**SLOs/SLIs/SLAs**: SLI = measured metric (P99 latency). SLO = target (P99 < 300ms, 99.99%). SLA = contract (99.9% uptime). Error budget = 100% - SLO. Exhausting error budget = halt feature releases. Google's approach: set SLO below system capability — "system can do 99.99%, SLO is 99.9%" — 0.09% error budget allows experimentation.

**Incident management**: SEV1 (customer outage) → SEV4 (cosmetic). Blameless postmortems with "Five Whys." Follow-the-sun on-call rotation.

---

## 7. Key Design Patterns Reference

| Pattern | Problem Solved | Example Tech | Trade-offs |
|---------|---------------|-------------|------------|
| **CQRS** | Separate read/write models for independent optimization | Axon, Eventuate | Eventual consistency, code duplication |
| **Event Sourcing** | Immutable event log as authoritative truth | Kafka, EventStoreDB | Schema evolution, query complexity |
| **Saga Pattern** | Distributed transactions without 2PC | Temporal, Camunda | Compensation complexity |
| **Strangler Fig** | Incrementally replace a monolith | Feature flags, reverse proxy | Long migration, routing complexity |
| **Bulkhead** | Isolate failure domains | Separate thread/connection pools | Higher resource overhead |
| **Circuit Breaker** | Fail fast on unhealthy dependencies | Resilience4j, Envoy | False positives, tuning complexity |
| **Throttling** | Control resource consumption | Token bucket, leaky bucket | Can reduce throughput below demand |
| **Leader Election** | Ensure only one instance acts as leader | ZooKeeper, etcd | Split-brain risk, election latency |
| **Idempotency Key** | Safe retry without duplicate side effects | Custom middleware (Stripe-style) | Storage for processed keys |
| **2PC** | Distributed atomic commit | XA, transaction coordinators | Blocking protocol, not for high throughput |
| **Backpressure** | Slow producers when consumers can't keep up | Reactive Streams, Kafka consumer lag | Complexity, can cause tail latency |
| **Claim Check** | Store large payloads in blob; queue carries ref | S3 + Kafka | Extra latency for blob retrieval |
| **Sidecar** | Offload cross-cutting concerns to co-located process | Envoy (Istio), Dapr | Resource overhead per pod |
| **Ambassador** | Proxy external traffic through intermediary | Envoy, Nginx, HAProxy | Extra hop adds latency |

---

## 8. Cloud-Native Technologies Stack

| Layer | Open Source | AWS | GCP | Azure |
|-------|-------------|-----|-----|-------|
| **Container orchestration** | Kubernetes | EKS | GKE | AKS |
| **Serverless** | Knative, OpenFaaS | Lambda | Cloud Run | Functions |
| **Service mesh** | Istio, Linkerd, Consul | App Mesh | Traffic Director | Service Fabric |
| **API Gateway** | Kong, Envoy, KrakenD | API Gateway | Apigee | API Management |
| **Messaging** | Kafka, Pulsar, RabbitMQ | MSK / SQS / Kinesis | Pub/Sub | Event Hubs / Bus |
| **Cache / KV** | Redis, Memcached, Dragonfly | ElastiCache | Memorystore | Cache for Redis |
| **CDN** | — | CloudFront | Cloud CDN | Azure CDN / Front Door |
| **Object storage** | MinIO, Ceph | S3 | GCS | Blob Storage |
| **Relational DB** | PostgreSQL, MySQL, CockroachDB | RDS / Aurora | Cloud SQL / Spanner | SQL / PostgreSQL |
| **Document NoSQL** | MongoDB | DynamoDB | Firestore | Cosmos DB |
| **Wide-column NoSQL** | Cassandra, Scylla | Keyspaces | Bigtable | Cosmos DB |
| **Graph** | Neo4j, JanusGraph | Neptune | — | Cosmos DB (Gremlin) |
| **Search** | Elasticsearch, OpenSearch | OpenSearch Service | — | Elastic (managed) |
| **Time-series** | Prometheus + Thanos/Cortex | CloudWatch | Cloud Monitoring | Monitor |
| **Tracing** | OpenTelemetry + Jaeger/Tempo | X-Ray | Cloud Trace | App Insights |
| **Logging** | Grafana Loki / OpenSearch | CloudWatch / OpenSearch | Cloud Logging | Log Analytics |
| **Stream processing** | Flink, Kafka Streams | Kinesis Analytics | Dataflow | Stream Analytics |
| **Data lake / warehouse** | Spark, Trino, Iceberg | Athena / Redshift | BigQuery | Synapse / Databricks |
| **Feature flags** | LaunchDarkly, Flagsmith | AppConfig | — | Feature Management |
| **Secrets** | Vault, SOPS | Secrets Manager | Secret Manager | Key Vault |
| **DNS** | CoreDNS, Bind | Route 53 | Cloud DNS | DNS |
| **DDoS** | — | Shield | Cloud Armor | DDoS Protection |

---

## 9. Anti-Patterns to Avoid

**Single point of failure**: Every component must have redundancy — LBs (active-active), DBs (replicas), caches (cluster), gateways (multiple instances). The weakest link defines availability.

**Talking to database from UI**: Never connect browser/mobile directly to DB. Always API layer. Direct DB access breaks security, bypasses caching, leaks schema.

**Synchronous cross-service calls in request path**: Chain of sync RPCs makes the slowest service determine user latency. If A→B→C→D and D = 200ms, user waits 200ms for a simple page. Use async eventual consistency for non-critical data.

**Custom security/encryption**: Never roll your own crypto, auth, or authorization. Use well-tested libraries (TLS, OAuth2, OIDC, bcrypt/argon2). Custom = undiagnosed vulnerabilities.

**Ignoring cold start**: Cold caches, empty connection pools, JIT-compiled code cause latency spikes after deployment. Pre-warm everything — queries, model weights, caches.

**Forgetting data gravity**: Move compute to data, not data to compute. Data movement at billion-user scale takes hours and significant bandwidth. Place compute in same region/AZ as the data.

**Shared database across services**: Single DB shared by multiple services = tight coupling. Schema changes require multi-team coordination. One bad query degrades all. Use database-per-service.

**Too fine-grained microservices**: Each service should own a bounded context (user, payment, notification). Don't create a service per function. Sweet spot: 1 service per 3–8 person team covering a coherent business capability.

**Not planning for failure**: Assume network partitions, disk failures, memory corruption, CPU throttling, kernel panics, dependency unavailability, cert expiration, quota limits. Every component fails eventually. Design for it from day one.

**Premature optimization**: Don't build billion-user architecture before 1M users. Over-engineering kills velocity. Build for current scale + 2× growth with a clear migration path to next phase.

**Ignoring cost at scale**: A $0.000001/invocation function at 1M/month = $1. At 1B/month = $1,000. Features "free" at small scale become expensive at billion-user scale. Track cost-per-request and cost-per-user in dashboards.

---

## 10. Data Sources & Further Reading

### Books
- "Designing Data-Intensive Applications" — Martin Kleppmann
- "Site Reliability Engineering" / "The Site Reliability Workbook" — Google SRE team
- "Building Microservices" (2nd ed) — Sam Newman
- "The Art of Scalability" — Abbott & Fisher
- "System Design Interview" — Alex Xu

### Engineering Blogs
- Meta: engineering.fb.com
- Netflix: netflixtechblog.com
- Twitter: blog.twitter.com/engineering
- Uber: uber.com/blog/engineering
- LinkedIn: linkedin.com/blog/engineering
- Spotify: engineering.atspotify.com
- Cloudflare: blog.cloudflare.com
- High Scalability: highscalability.com

### Key Papers
- Spanner: OSDI 2012 (globally-distributed DB)
- Bigtable: OSDI 2006 (structured data storage)
- Dynamo: SOSP 2007 (highly-available KV store)
- TAO: ATC 2013 (Facebook's social graph data store)
- Kafka: NetDB 2011 (distributed messaging)
- Borg: EuroSys 2015 (cluster management)
- Dremel: VLDB 2010 (interactive web-scale analysis)

### Frameworks
- AWS Well-Architected Framework: docs.aws.amazon.com/wellarchitected
- Google Cloud Architecture Framework: cloud.google.com/architecture/framework
- Azure Well-Architected Framework: learn.microsoft.com/azure/well-architected

---

> **Document authored for research purposes.** Covers architectures serving 1B+ users with
> case studies from Meta, Google, Netflix, Twitter/X, WhatsApp, TikTok/ByteDance, Uber,
> and LinkedIn. Last updated: July 2026.
