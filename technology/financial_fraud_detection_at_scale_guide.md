# Financial Fraud Detection at Scale

> A comprehensive guide to designing, building, and operating real-time fraud detection systems for large-scale financial transaction volumes — covering architectures, ML models, data pipelines, feature engineering, model lifecycle management, adversarial defense, regulatory compliance, and operational best practices.

**Audience**: Solution architects, ML engineers, fraud operations teams, and financial technologists at banks, payment processors, and fintechs. Written with focus on the core banking context — retail payments, cards, real-time payments, and digital banking channels.

---

## 1. What Financial Fraud Detection at Scale Means

Financial fraud detection at scale is the ability to screen, score, and flag fraudulent transactions in real-time (sub-100ms decision latency) across high-volume transaction streams — millions per day, peak tens of thousands per second (TPS) — while maintaining high detection rates (85-99% on known fraud types) and low false positive rates (sub-2-5%, mature systems targeting sub-1%).

This differs fundamentally from simpler fraud detection. At scale, several additional capabilities become mandatory:

- **Distributed stream processing** — a single server cannot handle peak loads; the pipeline must horizontally scale across clusters.
- **Low-latency ML inference** — model scoring must complete in single-digit milliseconds; every microsecond affects customer experience and authorization success rates.
- **Feature engineering pipelines** — hundreds to thousands of features must be computed in real-time with sub-second freshness from streaming and batch sources.
- **Model lifecycle management** — models must be versioned, A/B tested, monitored for drift, and seamlessly rolled forward or rolled back.
- **Regulatory compliance** — regulators (MAS FEAT in Singapore, SR 11-7 in the US) require explainability for declined transactions, model governance documentation, bias monitoring, and audit trails.
- **Concept drift and adversarial adaptation** — fraudsters continuously probe and evolve; models must keep pace.

A bank processing 10 million transactions daily with a 99.9% legitimate rate and 0.1% fraud rate still faces 10,000 fraudulent transactions per day. Each one must be caught in under 100 milliseconds while avoiding falsely declining 10,000 legitimate customers for every percentage point increase in false positive rate. This is the core tension at scale: maximize fraud capture while minimizing customer friction.

---

## 2. The Fraud Detection Landscape

Financial fraud manifests in many forms, each requiring different detection strategies. Understanding this taxonomy is essential before designing detection systems.

### 2.1 Payment Card Fraud

**Card-Not-Present (CNP) Fraud**: the dominant form in e-commerce. Fraudsters use stolen card details (number, expiry, CVV) for online purchases without physical possession. Detection relies on device fingerprinting, IP geolocation, velocity checks, and behavioral analysis.

**Card-Present (CP) Fraud**: fraud at physical POS using counterfeit or stolen cards. Detection leverages EMV chip authentication data, transaction timing, and merchant category analysis.

**Card Cloning / Skimming**: copying magnetic stripe data via compromised POS terminals or ATMs. Detection through duplicate transaction detection, geographic impossibility checks, and CP-vs-CNP pattern analysis.

**Application Fraud**: applying for a card using stolen or synthetic identity. Detection via identity verification, credit bureau cross-referencing, and device/IP consistency.

### 2.2 Account Takeover (ATO)

Fraudsters gain unauthorized access through: credential theft (phishing, data breaches, credential stuffing), session hijacking (malware, MITM, XSS), social engineering (tricking customer support into resetting credentials or bypassing MFA), or SIM swapping (porting victim's phone number to intercept SMS-based MFA codes). Detection relies on device fingerprint changes, IP geolocation shifts, behavioral biometrics (typing speed, mouse patterns), and unusual transaction patterns.

### 2.3 Identity Fraud

**Synthetic Identity Fraud**: the most challenging variant. Fraudsters blend real information (legitimate SSN/NRIC) with fabricated details to create a fictional persona, build credit over months or years, then "bust out" by maxing out credit lines. Detection requires graph analysis (link analysis across accounts sharing attributes), application pattern analysis, and longitudinal behavior monitoring.

**Identity Theft**: using a real person's complete identity without their knowledge. Detection via identity verification services, credit bureau alerts, and application consistency checks.

**New Account Fraud**: opening accounts with fabricated or stolen identities. Detection through device reputation, IP analysis, application velocity, and identity verification.

**First-Party Fraud (Friendly Fraud / Chargeback Abuse)**: legitimate account holders dispute legitimate transactions. Detection through dispute pattern analysis, customer history, merchant relationships, and transaction context.

### 2.4 Money Laundering

**Layering**: complex transaction chains obscuring fund origins. Detection via transaction graph analysis, circular transaction detection, and multi-hop tracing.

**Smurfing / Structuring**: breaking large amounts into smaller transactions to avoid reporting thresholds (e.g., $10K in the US). Detection via velocity monitoring, amount distribution analysis, and deposit pattern analysis.

**Trade-Based Laundering**: over-invoicing or under-invoicing goods/services. Detection requires trade document analysis, price benchmarking, and transaction-to-trade reconciliation.

**Shell Companies and Correspondent Banking Abuse**: legal entities obscuring beneficial ownership. Detection depends on entity resolution, ownership graph analysis, and KYC enrichment.

**Crypto Laundering**: using exchanges, mixers/tumblers, and DeFi protocols. Detection uses blockchain analytics, exchange pattern analysis, and transaction graph tracing.

### 2.5 Insider Fraud

Employees with legitimate access perpetrating fraud: collusion with external fraudsters, data theft (customer data for fraud rings), rogue trading (unauthorized positions), policy violation (overriding fraud flags). Detection via user behavior analytics (UBA), privileged access monitoring, system access pattern analysis, and separation-of-duties enforcement.

### 2.6 Loan Fraud

**Application Fraud**: falsifying income, assets, or employment. Detection via income verification, employment validation, cross-referencing with internal/external sources.

**Bust-Out Fraud**: build positive credit history (6-18 months), then rapidly max out all credit. Detection via credit velocity monitoring, utilization pattern analysis, and payment behavior trajectory.

**Straw Buyer**: person with good credit applies for a loan on behalf of someone who cannot qualify. Detection via relationship analysis, cross-application pattern matching, and KYC consistency.

### 2.7 Merchant Fraud

**Collusion Fraud**: merchant and fraudster collude to process fraudulent transactions. Detection via merchant behavior analysis, refund-to-sale ratio monitoring, and chargeback pattern analysis.

**Refund Fraud**: fake refunds to fraudster-controlled cards. Detection through refund velocity monitoring, refund-to-original-sale matching, and amount distribution analysis.

**Triangulation Fraud**: fraudster sets up a legitimate-looking storefront, accepts orders (and payments), uses stolen cards to fulfill orders at another merchant. Detection via drop-ship pattern detection, MCC mismatch analysis, and fulfillment timing.

### 2.8 Real-Time Payments Fraud

Instant payment systems (UPI, FAST, FedNow, SEPA Instant) create new vectors because payments are irrevocable:

- **APP Fraud**: trick victim into authorizing payment to fraudster's account. The transaction is legitimate from the bank's perspective — the fraud is in the social engineering. Detection via payment context analysis, beneficiary account reputation scoring, and value-date-pattern analysis.
- **Invoice Fraud**: fake invoices from legitimate-looking suppliers. Detection via payee verification, invoice pattern analysis, and supplier communication verification.
- **BEC / CEO Fraud**: impersonating a senior executive to authorize urgent payments. Detection via email authentication analysis, payment authorization pattern deviation, and beneficiary account change detection.
- **BNPL Fraud**: synthetic identities for buy-now-pay-later services. Detection via identity verification, spending pattern analysis, and repayment behavior monitoring.
- **Digital Wallet Fraud**: compromised Apple Pay, Google Pay, Alipay wallets. Detection via device binding verification, biometric auth analysis, and wallet token velocity monitoring.
- **P2P Payment Fraud**: exploitation of peer-to-peer platforms for scams and unauthorized transfers. Detection via social graph analysis, payment purpose classification, and reported scam pattern matching.

---

## 3. The Fraud Detection Lifecycle

A production fraud detection system follows a continuous lifecycle with six phases:

```
Data Ingestion ──► Feature Engineering ──► Model Inference ──► Decision ──► Case Management ──► Model Retraining
   │                       │                     │                │               │                   │
   ▼                       ▼                     ▼                ▼               ▼                   ▼
  TX Stream              TX Features          Real-time         Approve        Investigation       Label Data
  (real-time)            Customer Feats       Scoring           Review         & Decision          Feature Re-eng
  Customer Profile       Device Feats         Batch Scoring     Decline         Feedback Loop       Retrain
  (batch)                Network Feats        Ensemble Models   Step-Up         (labels back)       Deploy
  Device Fingerprint     Behavioral Feats     Rule Engine       Auth
  (real-time)            External Feats
  Network Graph
  (real-time+batch)
  External Signals
  (batch+real-time)
```

**Data Ingestion**: real-time transaction streams, batch customer profile updates, device fingerprint telemetry, network graph data, and external signals (credit bureaus, sanctions, threat intelligence) converge into the pipeline.

**Feature Engineering**: raw transaction data is transformed into predictive features — transaction attributes, customer historical patterns, device reputation scores, network topology metrics, and behavioral biometrics.

**Model Inference**: features are scored against an ensemble of models (rules + gradient-boosted trees + neural networks + graph neural networks) to produce risk scores.

**Decision**: risk scores are combined with business rules and risk appetite thresholds to produce outcomes: approve, review (manual), decline, or step-up authentication.

**Case Management**: flagged transactions become cases for fraud analysts. Investigators review evidence, make decisions, and produce outcomes that become labeled training data.

**Model Retraining**: labeled data from case outcomes, chargeback results, and investigation decisions feeds back into feature reengineering and model retraining, closing the loop.

---

## 4. Scale Requirements

### 4.1 Latency

| Requirement | Target | Context |
|---|---|---|
| End-to-end decision latency | <100ms (<50ms target) | Card-present and online payment authorization |
| Model inference latency | <10ms per model (1-5ms target) | Across the model ensemble |
| Feature retrieval latency | <5ms | From feature store |
| Stream processing lag | <1 second | Windowed aggregations |
| Batch scoring latency | Hours | Overnight non-real-time fraud detection |

Card-present transactions (Visa, Mastercard) have the tightest latency budgets — typically under 300ms for the entire authorization flow including network transit, issuer processing, and fraud detection. The fraud detection system must leave at least 50-100ms headroom within that window.

### 4.2 Throughput

- **Daily transaction volume**: millions to tens/hundreds of millions for large banks and processors
- **Peak transaction rate**: 10,000-100,000+ TPS
- **Model inference throughput**: thousands of QPS per node
- **Feature computation throughput**: matching TPS for real-time features
- **Case volume**: hundreds to thousands per day requiring human investigation

### 4.3 Data Volume

- **Training data**: terabytes to petabytes of historical transaction data
- **Daily feature storage**: gigabytes of real-time features
- **Feature store size**: tens to hundreds of GB for online serving
- **Graph database**: billions of nodes and edges for large institutions
- **Case management**: millions of historical cases with investigation artifacts

### 4.4 Feature Count and Model Complexity

| Dimension | Range |
|---|---|
| Per-transaction features | 100-10,000+ |
| Real-time computed features | 50-500 per transaction |
| Pre-computed (feature store) features | 100-5,000 per entity |
| Embedding features | 32-256 dimensions per embedding |
| Model inference time (tree) | <1ms (CPU) |
| Model inference time (NN) | 1-5ms (GPU) |
| Model inference time (GNN) | 5-50ms (graph-size dependent) |

---

## 5. Real-Time Fraud Detection Architecture

The architecture is layered to separate concerns and allow independent scaling of each component.

### 5.1 Architectural Overview

```
Transaction Streams (ISO 8583, ISO 20022, JSON, Avro, Protobuf)
       │
┌──────▼────────────────────────────────────────────────┐
│   INGESTION LAYER                                     │
│   Kafka / Event Hubs + Schema Registry + Enrichment   │
│   (device fingerprint, geolocation, IP intelligence,  │
│    velocity counters, customer profile lookup)        │
└──────┬────────────────────────────────────────────────┘
       │
┌──────▼────────────────────────────────────────────────┐
│   STREAM PROCESSING LAYER                             │
│   Apache Flink / Kafka Streams                        │
│   (windowed aggregations, velocity checks,            │
│    CEP pattern detection, stateful processing)        │
└──────┬────────────────────────────────────────────────┘
       │
┌──────▼────────────────────────────────────────────────┐
│   INFERENCE LAYER                                     │
│   Model Serving + Graph Inference + Rule Engine       │
│   (ensemble scoring, voting/stacking, fallback)       │
└──────┬────────────────────────────────────────────────┘
       │
┌──────▼────────────────────────────────────────────────┐
│   DECISION LAYER                                      │
│   Decision Orchestration + Threshold Mgmt             │
│   (approve/decline/review/step-up per merchant/card/  │
│    channel with risk appetite)                        │
└──────┬────────────────────────────────────────────────┘
       │
┌──────▼────────────────────────────────────────────────┐
│   STORES & DATABASES                                  │
│   Feature Store (Redis/S3) + Graph DB (Neo4j) +       │
│   Profile Store + Case Mgmt DB                        │
└───────────────────────────────────────────────────────┘
```

### 5.2 Ingestion Layer

**Kafka / Event Hubs** serve as the transaction event bus. Key requirements:
- **High throughput**: tens of thousands of messages per second per partition
- **Low latency**: sub-5ms publish and consume
- **Guaranteed delivery**: at-least-once or exactly-once semantics
- **Replay capability**: retain and replay past transactions for model retraining and backtesting
- **Partition key strategy**: partition by card hash, account ID, or customer ID to ensure ordering and enable stateful stream processing per entity

**Schema Registry** (Confluent, Apicurio): enforces contract compatibility between producers (payment gateways, POS terminals, online checkout) and consumers (stream processors, feature computation, model inference) via Avro, Protobuf, or JSON Schema.

**Protocols**:

| Protocol | Use Case | Characteristics |
|---|---|---|
| ISO 8583 | Legacy card payments (ATM, POS) | Fixed-field binary |
| ISO 20022 | Modern payments (SEPA, SWIFT, FAST) | XML, rich data model |
| JSON | Internal APIs, mobile SDK telemetry | Flexible, human-readable |
| Avro | Kafka-native, schema evolution | Compact binary with schema |
| Protobuf | High-performance internal pipelines | Compact binary, strong typing |

**Transaction Enrichment**: raw transaction messages are enriched before feature computation:
- **Device fingerprint**: browser fingerprint, device ID, OS version, screen resolution, installed fonts, WebGL renderer, audio fingerprint
- **Geolocation**: IP geolocation, GPS from mobile SDK, Wi-Fi BSSID
- **IP intelligence**: proxy/VPN detection, ASN, ISP, connection type, risk score
- **Velocity counters**: pre-computed counters from streaming aggregations
- **Customer profile lookup**: account age, historical spending patterns, current balance
- **External data**: sanctions lists, credit bureau scores, device reputation

### 5.3 Stream Processing Layer

**Apache Flink** is the preferred stream processor for real-time fraud detection because of:
- **Rich windowing**: sliding windows (5min, 1hr, 24hr), tumbling windows, session windows, custom window assigners
- **State management**: managed keyed state with RocksDB backend for billions of keys
- **Exactly-once semantics**: with event-time processing and watermarking for out-of-order events
- **Complex Event Processing (CEP)**: pattern matching on event streams (sequences, time-constrained patterns)
- **High throughput**: millions of events per second per Flink cluster

Typical Flink job DAG for fraud feature computation:

```
Kafka Source (transactions, partitioned by card_hash)
  │
  ├──FlatMap: Normalize, Validate, Reject malformed
  │     │
  │     ├── KeyBy(card_hash)
  │     │     ├── SlidingWindow(5min) → count, sum(amount), max(amount)
  │     │     └── SlidingWindow(1hr) → count, sum(amount), distinct merchants
  │     │
  │     ├── KeyBy(merchant_id)
  │     │     └── SlidingWindow(1hr) → count, distinct cards, distinct geos
  │     │
  │     ├── KeyBy(device_id)
  │     │     ├── SlidingWindow(24hr) → count, distinct accounts
  │     │     └── SlidingWindow(7d) → count, distinct IPs
  │     │
  │     ├── KeyBy(ip_address)
  │     │     └── SlidingWindow(1hr) → count, distinct cards, geo variance
  │     │
  │     ├── KeyBy(customer_id)
  │     │     ├── SlidingWindow(24hr) → count, sum(amount)
  │     │     ├── SlidingWindow(30d) → avg, median, std(amount), top MCC
  │     │     └── ValueState: time_since_last_txn, inter-arrival times
  │     │
  │     ├── CEP Pattern: RapidTxSequence
  │     │     (3+ txns at diff merchants within 60s → flag as velocity_burst)
  │     │
  │     └── CEP Pattern: CardTestingPattern
  │           (small auth → large decline → immediate retry → flag as card_test)
  │
  └── RichSink: Kafka Topic (enriched_features) + Side output for alerts
```

**Kafka Streams** is a lighter-weight alternative: runs as a library (no separate cluster), supports exactly-once via Kafka transactions, state stores via RocksDB, and basic windowing. Comparison:

| Factor | Flink | Kafka Streams |
|---|---|---|
| Windowing complexity | Rich (custom, CEP) | Basic |
| State management | RocksDB, TB-scale | RocksDB, smaller state |
| Exactly-once | Advanced | Via Kafka transactions |
| Event-time | Advanced watermarking | Basic |
| CEP / Pattern matching | Native | Manual state machine |
| Ops complexity | Cluster management | App-embedded |
| Throughput | Millions/sec | Hundreds K/sec |

**Spark Streaming** (micro-batch, 5s-min intervals): suitable for near-real-time fraud (overnight detection, batch AML screening) but **not** for card-present authorization where sub-second latency is required.

### 5.4 Inference Layer

**Real-Time ML Serving**: low-latency (sub-10ms per model), high-throughput (thousands of QPS per node), ensemble of 3-8 models per transaction, graceful degradation on failure.

**Serving Technologies**:

| Technology | Strengths | Weaknesses |
|---|---|---|
| ONNX Runtime | Cross-framework, optimized CPU/GPU | Limited graph model support |
| NVIDIA Triton | Multi-framework, GPU optimized, dynamic batching | Complex setup |
| SageMaker | Managed, A/B testing, auto-scaling | Vendor lock-in, cost |
| TorchServe | PyTorch native, custom handlers | Limited graph model support |
| MLflow | Model registry + serving, experiment tracking | Less optimized for latency |
| BentoML | Python-native, custom runners, observability | Newer ecosystem |
| Custom C++ | Minimum latency, maximum control | High development cost |

**Model Ensemble Strategy**:

```
Transaction Features
  │
  ├── Rule Engine (deterministic) ─────────► Risk Flags
  ├── LightGBM (gradient boosted) ─────────► Score_GB
  ├── DNN (wide & deep) ──────────────────► Score_NN
  ├── GraphSAGE (graph) ──────────────────► Score_Graph
  ├── Isolation Forest (anomaly) ──────────► Score_Anomaly
  │
  └── Ensemble Meta-Model / Stacking ─────► Final Risk Score
                                               │
                                          Decision Layer
```

**Rule Engine** (deterministic logic complementary to ML): velocity limits (max 5 txns/10min per card, $2K cumulative/24hr), country blocks (high-risk countries, first-txn-from-new-country), blacklists (known fraud cards/devices/IPs), whitelists (trusted merchants, verified devices), MCC blocks (gambling for certain card types, cash advance limits), amount thresholds (flag >$10K, round-dollar amounts), time-based rules (block overnight for daytime spenders).

**Voting and Stacking**: simple majority (3 of 5 flag → decline), weighted voting (model weights from validation performance), stacking (meta-model on model outputs — usually logistic regression or LightGBM).

**Cascading**: evaluate fast models first (rules → tree → NN), escalate to complex models (graph) only when uncertainty is high. ~70% of transactions resolved by first-stage models.

**Fallback Strategies**:
- **Model timeout**: if exceeds latency budget, fall back to rules-only
- **Model failure**: if unhealthy (error rate > threshold), route around it
- **Graceful degradation**: reduce ensemble size during traffic spikes
- **Circuit breaker**: if upstream sources flap, use cached features with degraded confidence
- **Load shedding**: drop non-critical enrichment, skip expensive models

### 5.5 Decision Layer

Combines model scores + rules + business risk appetite:

```python
def make_decision(transaction, model_scores, rules, risk_appetite):
    ensemble_score = meta_model.predict(model_scores)
    rule_flags = rule_engine.evaluate(transaction)
    threshold = risk_appetite.get_threshold(
        channel=transaction.channel, merchant_mcc=transaction.mcc,
        card_type=transaction.card_type, amount=transaction.amount)

    if rule_flags and ensemble_score > threshold.high:
        return DECLINE
    elif ensemble_score > threshold.medium:
        if transaction.channel in [ECOMMERCE, P2P]:
            return STEP_UP_AUTH
        return REVIEW
    elif rule_flags:
        return REVIEW
    elif ensemble_score > threshold.low:
        return DECLINE  # for high-risk channels
    else:
        return APPROVE
```

**Decision Outcomes**: APPROVE (proceed), DECLINE (rejected), REVIEW (manual investigation), STEP-UP AUTH (additional verification, adds 2-30s).

**Threshold Management** varies by: merchant category (high-risk MCCs: gambling, crypto, money transfer — stricter), card type (credit vs debit vs prepaid), channel (e-commerce vs POS vs mobile vs P2P vs ATM), customer segment (premium vs new), transaction amount, geographic region.

### 5.6 Stores and Databases

**Feature Store** (online + offline):

The feature store is the central nervous system of a fraud detection system. It decouples feature computation from model inference, enabling teams to develop features independently and serve them consistently across training and serving.

- **Online store**: Redis, Aerospike, or DynamoDB for sub-millisecond key-value reads. Stores pre-computed features: velocity counters (tx_count_1hr, amount_sum_24hr), customer profile metrics (avg_amount_30d, account_age), device reputation scores. Keyed by entity ID (card_hash, customer_id, device_id, merchant_id). Supports bulk multi-get for reading all entities involved in a transaction. Local LRU cache on each inference node for hot features (sub-ms access, tens of MB per node).
- **Offline store**: S3, ADLS, or GCS with Parquet files partitioned by date and entity. Stores historical snapshots and enables point-in-time correct training data generation. Feature values are stored with `as_of` timestamps so training data joins use the feature version active at the transaction timestamp.
- **Feature Serving API**: gRPC or REST endpoints: `get_features(entity_id, feature_list)`, `get_features_batch(entity_ids, feature_list)`, `get_historical_features(entity_id, timestamp)`. Implemented with Feast or Tecton for feature registry, versioning, and lineage tracking.
- **Feature registry**: metadata catalog with feature definitions, data sources, owners, versions, transformation logic, and upstream/downstream lineage. Enables discovery, governance, and impact analysis.

**Graph Database** (Neo4j, Memgraph):

Used for real-time graph queries at scoring time:

- **Entity resolution**: resolve that device_id-xyz, ip-456, and phone-789 all belong to the same synthetic identity across multiple accounts
- **Fraud ring detection**: find accounts that share devices, phones, addresses, and merchants in patterns consistent with organized fraud rings using Louvain or Leiden community detection
- **Shortest path**: compute distance (in hops) between transaction entities and known fraud nodes — shorter paths indicate higher risk
- **Personalized PageRank**: measure how much of a random walk from transaction entities lands on known fraud nodes — higher probability = stronger association
- **Entity linking**: connect transactions to fraud rings via shared attributes across the graph

Real-time graph query constraints: queries must complete in <10ms. Achieved through:
- **Node property indexing**: entities indexed by ID for O(1) lookups
- **Pre-computed scores**: community membership, PageRank, and centrality scores computed offline and stored as node properties
- **Limited traversal depth**: real-time queries constrained to 2-3 hops; deeper analysis performed asynchronously
- **Graph partitioning**: large graphs partitioned across database shards; queries routed to relevant shards
- **Caching**: frequent query results (e.g., shortest path to known fraud for high-volume entities) cached with TTL

**Profile Store**: customer behavior profiles maintained for real-time scoring:
- Spending patterns: average/median amounts, typical merchants with frequency, spending cadence (time between transactions), preferred channels
- Velocity baselines: typical transaction count per hour/day/week, by channel and merchant category, with seasonal adjustments
- Device history: known devices with first/last seen timestamps, device age, linked accounts, anomaly flags
- Behavioral biometric baselines: typical keystroke dynamics, mouse movement patterns, session duration, inter-action timing
- Session behavior norms: typical login frequency, navigation paths, page dwell times

**Case Management Database**: relational (PostgreSQL, Aurora) or document store (DynamoDB, MongoDB):
- Cases: investigation records with fraud analyst decisions, evidence artifacts, notes, timestamps
- Outcomes: confirmed fraud, false positive, inconclusive, with detailed reason codes
- Labels: ground truth for model retraining, including feedback from chargebacks, customer disputes, and investigator review
- Audit trail: all actions taken on each case — who did what and when, system overrides, rule changes

---

## 6. Feature Engineering for Fraud Detection

Feature engineering is the highest-impact activity in a fraud detection system. Good features consistently outperform model architecture changes.

### 6.1 Types of Features

**Transaction Features**: amount (raw, log-transformed, binned), currency (ISO code), merchant category code (MCC), transaction type (purchase/refund/cash advance/balance inquiry), channel (e-commerce/POS/mobile/ATM/P2P/recurring), time of day (cyclical sin/cos encoding), day of week, distance from home (Haversine), entry mode (chip/swipe/contactless/manual), auth type (3DS/biometric/password/none).

**Customer Features**: avg/median transaction amount (30d/90d/1yr), max single amount, top 10 merchants by frequency and amount, typical tx count per day/week, account age (days), total historical transactions, spending entropy (merchant category diversity), GeoIP country vs transaction country, distinct devices used, distinct IPs used, time since first transaction.

**Device Features**: device ID, IP address (with geolocation and proxy/VPN flags), device fingerprint (canvas/WebGL/audio/font fingerprinting), browser/OS/screen resolution/language, known device indicator, device velocity (unique accounts associated in 24hr), device age (time since first seen), device reputation score (aggregate across accounts), high-risk attribute count (headless browser, automation tools, timezone mismatch).

**Network/Graph Features**: degree centrality (connections per entity), proximity to known fraud (shortest path distance), shared attribute ratio (% of devices/phones/emails shared with other accounts), community size and density, (device, merchant) bi-gram velocity, entity linkage density, historical fraud ratio (fraud % on shared devices), two-hop fraud concentration (% of entities two hops away flagged as fraud).

**Behavioral Features**:
- **Keystroke dynamics**: typing speed, key hold time, inter-key latency, error rate, backspace frequency
- **Mouse dynamics**: movement speed and acceleration, click patterns, scroll patterns, hovering behavior
- **Navigation patterns**: page flow sequence (login→payment→confirm vs jump to payment), session duration, time between actions, page dwell time
- **Touch gestures** (mobile): swipe velocity, tap pressure, multi-touch patterns
- **Field completion order**: tab order through form fields, autofill detection vs manual entry
- **Copy/paste behavior**: pasted vs typed data in form fields, clipboard monitoring

**External Features**: credit bureau scores and fraud alerts, sanctions lists (OFAC, UN, EU, MAS), PEP lists, negative news/adverse media, fraud ring threat intelligence feeds (LexisNexis, Feedzai, iovation, ThreatMetrix), social media analysis (emerging), open banking data (account aggregation, transaction verification).

**Temporal Features**: time since last transaction/password change/device change, hour/day cyclical encoding (sin, cos), holiday indicator (public holidays, shopping days, festive periods), seasonality (month, quarter), transaction frequency (txns per unit time).

**Rolling Window Features** (computed via Flink stateful processing):

| Window | Features |
|---|---|
| 1hr sliding | tx count, amount sum, distinct merchants, distinct devices |
| 24hr sliding | tx count, cumulative amount, distinct IPs, distinct countries |
| 7d sliding | average amount, std deviation, total count, distinct merchants |
| 30d sliding | running average amount, median amount, baseline velocity |
| Rate of change | 3d-vs-30d velocity ratio, trend direction (linear regression slope) |

### 6.2 Streaming Feature Computation Patterns

**Pattern 1: Sliding Window Aggregation** (Flink SQL):
```sql
SELECT card_hash, COUNT(*) AS tx_count_1hr, SUM(amount) AS amount_sum_1hr,
       COUNT(DISTINCT merchant_id) AS distinct_merchants_1hr
FROM transactions WHERE card_hash IS NOT NULL
GROUP BY card_hash, TUMBLE(rowtime, INTERVAL '5' MINUTE)
```

**Pattern 2: Stateful Counters with Per-Transaction Emission**:
Maintain `ValueState<VelocityState>` per key. On each transaction, purge expired windows, increment counters, emit current velocity values inline. This avoids window boundary latency — features are always fresh.

**Pattern 3: CEP Pattern Detection** (Flink CEP):
- Rapid successive transactions: 3+ in 60 seconds at different merchants
- Card testing: small amounts → large decline → large attempt
- Geographic hopping: geographically impossible sequence in short time
- Profile change: password reset → device change → transaction

### 6.3 Challenges in Feature Engineering

**Class Imbalance**: fraud is typically 0.1-2% of transactions. Standard accuracy optimization trivializes the problem (99%+ by always predicting "legitimate"). Rare but highly predictive features (device across 50+ accounts) are informative but hard to learn. Sampling strategies affect feature importance and model calibration.

**Concept Drift**: fraud patterns evolve continuously. Features that were highly predictive 6 months ago may be worthless today. Fraudsters adapt to detection. Feature distribution shifts may be gradual (seasonality) or sudden (new fraud technique). New payment methods introduce new feature spaces.

**Feedback Loop Latency**: ground truth is delayed — chargebacks can take 30-90 days; authorization vs settlement amounts differ; manual review outcomes are delayed; false positives often never discovered. Feature engineering for training data must use point-in-time correct values (features as they existed at prediction time, not today's values).

**Feature Backfill**: historical features must use the same logic as online features. Point-in-time correctness via time-travel queries on the feature store (Feast/Tecton `as_of_timestamps`). Backfill for large datasets can take days.

### 6.4 Feature Validation and Monitoring

- **Feature drift detection**: PSI and KS test on current vs baseline feature distributions
- **Feature importance tracking**: SHAP values over time to detect shifts in what drives predictions
- **Missing data handling**: null values must be explicit, never silently cast to 0; monitor for unexpected null rates
- **Data quality**: schema validation, range checks, cardinality bounds, referential integrity checks

### 6.5 Anti-Adversarial Features

Features designed to resist gaming by adaptive fraudsters:
- **Velocity decoys**: track phantom lookups — failed OTP attempts, password reset requests, rate-limited endpoint queries
- **Randomization detection**: measure entropy of transaction attributes — fraudsters often randomize amounts, timestamps, and IPs differently from legitimate patterns
- **Pattern similarity**: align transaction feature vectors against known fraud patterns using sequence alignment or temporal similarity scoring
- **Graph coherence**: does the transaction's graph neighborhood have natural structure or appear artificially constructed for a fraud ring?
- **Behavioral consistency**: cross-reference biometric features with transaction attributes — does typing speed match typical speed for this customer?

---

## 7. Model Architecture for Fraud Detection

### 7.1 Supervised Models

**Gradient Boosted Trees (XGBoost, LightGBM, CatBoost)**

The dominant model class for tabular fraud data. Why GBTs consistently win fraud benchmarks:
- Best-in-class tabular performance on heterogeneous feature spaces
- SHAP interpretability (essential for regulatory compliance)
- Native missing data handling (XGBoost learns best direction)
- High-cardinality categorical handling (CatBoost ordered target encoding, LightGBM GOSS)
- Handles class imbalance via `scale_pos_weight`, sample weights, `min_child_weight`
- Sub-millisecond inference (100-500 trees)

```python
params = {
    'objective': 'binary', 'metric': 'auc', 'boosting_type': 'gbdt',
    'num_leaves': 63, 'max_depth': 8, 'learning_rate': 0.05,
    'feature_fraction': 0.8, 'bagging_fraction': 0.8, 'bagging_freq': 5,
    'min_data_in_leaf': 50, 'scale_pos_weight': 100,
    'lambda_l1': 0.1, 'lambda_l2': 0.1, 'device': 'gpu'
}
```

**Random Forest**: ensemble of decision trees on bootstrapped data. More robust to overfitting than boosting. Handles high-dimensional features well. Good baseline — often similar performance to GBT with less hyperparameter tuning. Weakness: tends to underperform GBT on highly imbalanced data.

**Logistic Regression**: simple, interpretable, regulatory-compliant. Cannot capture non-linear relationships or feature interactions without manual engineering. Still used in heavily regulated environments where explainability is paramount.

**Neural Networks**:
- **Wide & Deep**: combines memorization (wide cross-product features) with generalization (deep learned feature interactions). Excellent for high-cardinality categoricals via entity embeddings.
- **Transformers for sequence modeling**: capture transaction sequence context — order matters (small test → large purchase, rapid fire across merchants).
- **Autoencoders**: trained to reconstruct legitimate transactions; high reconstruction error signals anomaly.

Wide & Deep architecture:
```
Input Features
  ├── Categoricals → Embeddings → Concatenated
  ├── Numericals → Normalization → Dense
  │
  ├── [Wide] Cross-product Features → Linear Layer ──┐
  ├── [Deep] Dense(256) → Dense(128) → Dense(64) ────┤
  │                                                   │
  └────────────────────────────────────── Concatenate → Output (sigmoid)
```

### 7.2 Graph Neural Networks (GNNs)

Capture relational patterns across the transaction network — entity resolution, fraud ring detection, relational reasoning that transactional models miss.

**Why GNNs for fraud**: a transaction flagged because it shares a device with another fraudulent transaction — even though the transaction itself looks clean. Graph features are harder for fraudsters to game.

**Architectures**:
- **GraphSAGE**: inductive learning with neighbor sampling (10-50 neighbors, 2-3 hops). No retraining for new nodes. Best for large transaction graphs.
- **GAT**: attention-weighted neighbor aggregation; learns which neighbors are most relevant.
- **RGCN**: multiple edge types (transaction, device-account, shared-IP, shared-phone) with relation-specific transformations.
- **Heterogeneous GNNs**: handle Account, Device, Merchant, IP, Phone node types with type-specific message passing.

**Training at scale**: neighbor sampling, mini-batch training on sampled subgraphs, distributed graph partitioning (METIS).

### 7.3 Anomaly Detection Models

For detecting unknown fraud patterns that supervised models haven't seen:
- **Isolation Forest**: isolates anomalies via random feature splitting. Fast, scalable, go-to anomaly detector.
- **Autoencoders**: reconstruction error as anomaly score. Can detect novel fraud patterns.
- **VAE**: reconstruction + KL divergence. More robust for high-dimensional features.
- **One-Class SVM**: boundary around legitimate class. Less scalable but complex boundaries with RBF kernel.
- **Local Outlier Factor**: local density deviation. Computationally expensive at scale.
- **Adversarial Autoencoders**: generate features for unknown attack patterns.

### 7.4 Ensemble Strategies

- **Stacking**: meta-model (logistic regression or LightGBM) on base model predictions from non-overlapping training folds. Learns optimal combination weights.
- **Bagging**: same architecture on bootstrapped samples. Reduces variance. Different models capture different fraud subpopulations.
- **Boosting**: sequential error correction. Focuses on hardest cases.
- **Voting**: simple majority or weighted (by validation AUC or cost metric).
- **Cascading**: fast models first (rules → tree → NN), escalate to complex (graph) only when uncertainty high.

**Multi-model orchestration**:
```
Txn arrives ───► Rule Engine (<0.1ms)
  ├── Clear pass → APPROVE (exit)
  ├── Clear fail → DECLINE (exit)
  │
  ├── LightGBM (<1ms)
  │     ├── Low risk + low uncertainty → DECISION
  │     └── Medium/high risk or uncertainty → escalate
  │
  ├── DNN + Autoencoder (2-5ms GPU)
  │     ├── Combined passes threshold → DECISION
  │     └── Still uncertain / high value → escalate
  │
  └── Graph Model (5-50ms)
        └── Final decision with full relational context
```

### 7.5 Sequence Models

LSTM/GRU for transaction sequences: temporal ordering matters (small test → large purchase, rapid fire). Transformer-based behavioral models: self-attention over sequences captures cross-transaction relationships. Attention mechanisms identify which past transactions drive predictions (also provides explainability).

### 7.6 Graph Models for Fraud Ring Detection

**Community Detection**: Louvain (fast modularity optimization), Leiden (guarantees connected communities, higher quality), Weakly Connected Components (fast first pass for connected subgraphs).

**Proximity Scoring**: shortest path to known fraud nodes, Katz centrality (walks weighted by path length), personalized PageRank (random walk from transaction entities to known fraud).

**Entity Resolution**: resolve identities across accounts (shared devices/IPs/phones/emails), build unified entity graph, link transactions to fraud rings, construct knowledge graphs for multi-hop relationships.

---

## 8. Model Training at Scale

### 8.1 Handling Class Imbalance (0.1-2% fraud rate)

The extreme class imbalance is the defining challenge of fraud ML. Multiple approaches are used in combination:

**Resampling**
- **Oversampling minority**: SMOTE generates synthetic fraud samples by interpolating between k-nearest neighbor fraud samples in feature space. ADASYN adaptively focuses on harder-to-learn samples by generating more synthetic examples near the decision boundary.
- **Undersampling majority**: random undersampling of legitimate transactions. Can discard useful information, so typically combined with oversampling.
- **Hybrid**: SMOTE + undersampling, SMOTE + Tomek links (remove borderline majority samples), or SMOTE + Edited Nearest Neighbors.

**Cost-Sensitive Learning**
- Assign different misclassification costs: C_FN (cost of allowing a fraudulent transaction) is typically 10-100× C_FP (cost of blocking a legitimate transaction).
- Implemented via `scale_pos_weight` in XGBoost/LightGBM (ratio of negative to positive class weights), `class_weight='balanced'` in sklearn models, or custom loss functions with per-sample weights.

**Ensemble of Undersampled Models**
- **EasyEnsemble**: train multiple models (e.g., 10-20), each on a different randomly undersampled subset of the majority class combined with all minority samples. Final prediction is the average across all models.
- **BalanceCascade**: sequential ensemble where correctly classified majority samples are removed after each model. Progressively focuses on the hardest cases.

**Custom Loss Functions**
- **Focal loss**: down-weights well-classified examples, focuses learning on hard, misclassified cases. γ parameter controls how much to focus (γ=2 is common). Particularly effective for neural network training with extreme imbalance.
- **Asymmetric loss**: different gradient magnitudes for positive vs negative errors. Allows asymmetric treatment without class weighting.
- **AUC loss / ranking loss**: directly optimize the ranking of fraud above legitimate transactions (pairwise or listwise). Bypasses threshold issues during training.
- **Margin-based losses**: enforce a minimum margin between fraud and legitimate scores in embedding space.

**Anomaly Detection Approach**
- For extreme imbalance (<0.1%): treat fraud detection as an anomaly detection problem
- Train on predominantly legitimate data; flag outliers as potential fraud
- Use labeled fraud data only for threshold calibration, not model training
- Semi-supervised: combine unsupervised anomaly score with supervised fine-tuning on available labels

### 8.2 Distributed Training

| Technique | Description | When to Use |
|---|---|---|
| Data parallelism | Split data across workers, sync/async gradient updates (all-reduce NCCL) | Large datasets, moderate model size |
| Model parallelism | Split large model across workers | Very large models (GNNs, massive embeddings) |
| Pipeline parallelism | Pipeline stages across GPUs | Deep NNs with sequential computation |
| Parameter servers | Centralized/distributed parameter storage | Parameters > single GPU memory |

### 8.3 Hyperparameter Optimization

Bayesian optimization (Hyperopt, Optuna) outperforms grid/random search for fraud models. Early stopping (20-50 rounds no improvement). Learning rate schedules: warmup (linear 0→target over N steps) → decay (cosine annealing, step decay, exponential decay). Cyclic LR to escape local minima.

Regularization: L1 (sparse feature selection), L2 (weight decay), dropout (0.2-0.5, MC dropout at inference for uncertainty), batch normalization, label smoothing, gradient clipping (max norm 1.0-5.0). Curriculum learning: easy → hard examples.

### 8.4 Calibration

Fraud models must output well-calibrated probabilities for threshold tuning. Platt scaling (logistic regression on model outputs), isotonic regression (non-parametric, better for large data), temperature scaling (single parameter for NN softmax). Monitor expected calibration error (ECE) and reliability diagrams.

---

## 9. Evaluation Metrics

Accuracy is a meaningless metric for fraud detection (99.9% accuracy is achievable by always predicting "legitimate"). The right metrics focus on the model's ability to rank fraud above legitimate transactions and operate at business-optimal thresholds.

### 9.1 Precision and Recall

| Metric | Formula | Relevance |
|---|---|---|
| **Precision** | TP / (TP + FP) | Measures false positive cost — each FP is a blocked legitimate customer |
| **Recall (TPR)** | TP / (TP + FN) | Measures missed fraud cost — each FN is a fraudulent transaction that passes through |
| **F1 Score** | 2 × (P × R) / (P + R) | Harmonic mean of precision and recall |
| **F2 Score** | 5 × (P × R) / (4P + R) | Weighs recall higher — appropriate when catching fraud is more important than avoiding false positives |
| **F-beta** | (1+β²) × (P × R) / (β²P + R) | Generalization; β=2 for recall-focused |

Precision and recall are threshold-dependent. The threshold should be tuned to optimize business metrics (expected cost), not F1.

### 9.2 Area Under Curves

- **Precision-Recall AUC (PR-AUC)**: the most important curve for fraud detection. PR curves are sensitive to class imbalance and show model performance at all thresholds. Better than ROC-AUC for fraud because ROC curves can look optimistic when the negative class (legitimate transactions) dominates.
- **ROC-AUC**: less sensitive to class imbalance. Useful for comparing across datasets, but can be misleadingly optimistic for fraud — a model can achieve high ROC-AUC with poor precision at low FPR.
- **Lift**: fraud capture rate improvement over random selection. Lift@10% = (fraud captured in top 10% scored transactions) / (10% random capture). Intuitively communicates model value to business stakeholders.

### 9.3 Operational Metrics

- **False Positive Rate (FPR)**: FP / (FP + TN). The most important operational metric. Each 0.1% increase in FPR means thousands of legitimate customers inconvenienced. Mature systems target FPR < 1% while maintaining high capture rates.
- **Fraud Capture Rate**: detected fraud $ / total fraud $. The bottom-line metric — measures actual financial loss prevention.
- **TPR at Low FPR**: TPR at 0.1% FPR, TPR at 0.5% FPR, TPR at 1% FPR. The standard fraud benchmarks. A production model must achieve high TPR at extremely low FPR — this is the real test of discrimination ability.
- **False Positive Cost**: estimated cost of blocked legitimate transactions = lost revenue + customer friction + support call volume + reputational damage.
- **Mean Average Precision (mAP)**: average precision across different recall thresholds.

### 9.4 Cost-Based Metrics

The business-optimal metric combines the costs of different error types:

```
Expected Cost per Transaction = C_FN × FN_rate + C_FP × FP_rate

Where:
  C_FN = cost of missed fraud (fraud amount + operational costs + regulatory fines)
  C_FP = cost of declined legitimate transaction (lost revenue + customer friction + support cost)
```

The optimal threshold minimizes expected cost. This is the metric that aligns model performance with business outcomes. Threshold tuning is a separate step from model training — the threshold is chosen on a holdout validation set after the model is trained.

### 9.5 Additional Metrics

- **Gini Coefficient**: 2 × AUC - 1. Common in credit modeling. Measures model's ability to rank-order risk.
- **Kolmogorov-Smirnov (KS)**: maximum separation between cumulative distributions of fraud and legitimate scores. Higher KS = better separation.
- **Brier Score**: mean squared error between predicted probabilities and actual outcomes. Measures calibration + discrimination together.
- **Expected Calibration Error (ECE)**: measures how well predicted probabilities match actual frequencies. Important for threshold tuning.

---

## 10. Regulatory Requirements for Models

### 10.1 MAS FEAT (Monetary Authority of Singapore)

Fairness, Ethics, Accountability, Transparency:
- **Fairness**: no bias against protected groups. Requires stratified performance evaluation (TPR, FPR, precision per demographic segment).
- **Ethics**: documented intended scope; not used beyond it. Ethical review for model applications.
- **Accountability**: attributable decisions — model ownership, governance committees, approval workflows.
- **Transparency**: SHAP/LIME explanations for every declined transaction, reason codes, top contributing features. Model documentation includes bias analysis.

Every declined transaction must produce an explanation. Periodic fairness monitoring with reporting to risk committees. Independent model validation before production.

### 10.2 SR 11-7 (Federal Reserve / OCC)

Model risk management framework requiring:
- Three lines of defense: development (1st), independent validation (2nd), internal audit (3rd)
- Model documentation: development, validation, implementation, monitoring, governance
- Independent validation: conceptual soundness, outcomes analysis, ongoing monitoring
- Backtesting: historical performance vs predictions
- Benchmarking: comparison against alternative models or industry standards
- Board-level oversight, documented approval process, annual review cycle

### 10.3 Model Explainability

- **SHAP**: game-theoretic feature attribution. Global + local explanations. Every declined transaction produces a SHAP force plot.
- **LIME**: local surrogate model. Faster but less stable than SHAP.
- **PDP**: marginal feature effects for regulator review of model behavior.
- **Feature importance**: permutation importance (drop in performance when shuffled), gain importance (information gain from splits).
- **Counterfactuals**: "if amount was $50 less, transaction would have been approved." Intuitive for customers but computationally expensive.
- **Exception reporting**: automated reporting of model behavior changes, feature drift, performance degradation.

### 10.4 Bias and Fairness Monitoring

Disparate impact (disproportionate decline rates across groups), equal opportunity (equal fraud sensitivity across groups). Metrics: demographic parity, equalized odds, predictive parity. Requires stratified TPR/FPR/precision evaluation per demographic segment, with reporting cadence defined in model governance documentation.

---

## 11. Model Monitoring and Retraining

### 11.1 Monitoring Metrics

**PSI (Population Stability Index)**: measures score distribution shift.
- PSI < 0.1: no significant shift
- PSI 0.1-0.25: moderate shift — investigate
- PSI > 0.25: significant shift — retrain needed

**CSI (Characteristic Stability Index)**: per-feature drift detection.

**Model Performance**: precision/recall/FPR/TPR at fixed thresholds (daily monitoring), PR-AUC/ROC-AUC (weekly), fraud capture rate in $ (daily), false positive volume and rate (hourly).

**Feature Drift**: per-feature PSI and KS, null rate monitoring, cardinality drift (new categorical values), distribution comparisons (KDE plots, summary statistics).

**Concept Drift**: distribution monitoring + performance monitoring + error analysis + adversarial pattern detection. Bayesian Structural Time Series (BSTS) models expected performance with uncertainty bounds and flags deviations from the Bayesian posterior.

### 11.2 Retraining Triggers

| Trigger | Action | Timeframe |
|---|---|---|
| FPR > 3% threshold breached | Urgent retraining | Hours-days |
| Concept drift detected | Quality retraining | Days-weeks |
| New fraud pattern identified | Targeted retraining + new features | Days |
| Time-based (weekly/monthly) | Scheduled retraining | Scheduled |
| Feature distribution shift (PSI > 0.25) | Investigate features + retrain | Days |
| Seasonal effects (holidays, sales events) | Proactive seasonal retraining | Before event |
| Regulatory requirement (annual review) | Full model redevelopment | Months |

### 11.3 Retraining Pipeline

```
Data Extract → Feature Engineering → Model Train/Validate → Champion/Challenger → A/B Test → Canary Deploy → Rollout
```

**Feedback Loop**: investigation outcomes (confirmed fraud/FP/inconclusive) form labels; chargeback results (30-90 day delayed, definitive); review decisions (intermediate, less reliable); customer disputes (self-reported flags).

### 11.4 Champion/Challenger Framework

- **Champion**: current production model
- **Challenger(s)**: candidates with new features, architecture, or training data
- **Evaluation**: challenger shadows champion for N days (minimum 7-14). Compare PR-AUC, TPR@lowFPR, PSI, expected cost
- **Promotion**: challenger promoted on statistically significant improvement (t-test, Bayesian A/B test)
- **Rollback**: champion kept warm for at least one retraining cycle

### 11.5 Deployment Patterns

**Canary**: 1% → 5% → 25% → 100% traffic with guardrails at each step. Automatic rollback if FPR/latency bounds breached.

**A/B Testing**: disjoint traffic splits (by transaction ID hash), champion vs challenger on separate transaction sets. Monitor all metrics for 7+ days with statistical significance testing.

**Real-time Model Switching**: models registered in MLflow/SageMaker, switched via config push without deployment downtime.

**Push-button Rollback**: previous champion version always deployable on demand.

---

## 12. Adversarial Fraud Detection

Fraudsters are adaptive adversaries who actively probe and reverse-engineer detection systems.

### 12.1 How Fraudsters Adapt

1. **Test detection boundaries**: incremental amount hikes, time-based rule probes, IP/device variations, feature boundary exploration.
2. **Probe model responses**: submit similar transactions varying one feature at a time; measure approve/decline pattern to infer decision boundary; use model as oracle for adversarial example generation.
3. **Reverse engineer rules**: test each rule independently via edge cases; build evasion profile.
4. **Mimic legitimate behavior**: slow and low (small amounts, long intervals, days-weeks spacing); copy real transaction patterns from compromised accounts.
5. **Automate with AI**: ML-generated synthetic identities that pass verification; automated credential stuffing at scale; generative AI for phishing and social engineering scripts.

### 12.2 Adversarial ML Attacks

- **Evasion attacks**: craft transaction features to avoid detection (add noise near decision boundary, split amounts, use random IPs). Black-box via API querying; white-box via insider threat.
- **Poisoning attacks**: submit fake fraud reports to corrupt training data during retraining. E.g., fraudster reports their legitimate transactions as fraud to reduce FPR on attack patterns.
- **Model extraction**: query detection API to reconstruct surrogate model; transaction features in, approve/review/decline out; surrogate reveals decision boundaries for evasion crafting.

### 12.3 Adversarial Attack Patterns

Slow and low (days between $5-20 txns, perfect mimicry), synthetic identity arrays (thousands of identities algorithmically generated, months of credit building), collusion networks (hundreds of accounts across devices/IPs/merchants sharing limited phones/addresses), cross-channel attacks (card + digital wallet + P2P + ATM — each channel sees only a fragment), behavioral mimicry (bots replicating human mouse movements, typing, navigation; incremental behavior changes that avoid anomaly detectors).

### 12.4 Detection of Adversarial Attacks

**Model-based**: detect probe queries (count of near-boundary transactions from same entity), adversarial patterns (extreme on one feature, normal on all others), high uncertainty clusters (high entropy, high ensemble variance).

**Graph anomaly detection**: unusually connected communities, high shared-attribute-to-account ratio, entities "too connected" for natural social graphs.

**Feature consistency**: device fingerprint anomalies (UA inconsistent with device capabilities), behavioral consistency (typing speed vs device type — mobile vs desktop), cross-reference velocity across independent signals.

**Transaction pattern monitoring**: incremental feature changes over time from fraud ring, clustering similar feature vectors for pattern identification, sequence alignment against known fraudulent sequences.

### 12.5 Defenses

- **Adversarial training**: train on adversarial examples generated during training; regular retraining to incorporate new patterns; ensemble diversity makes evasion harder.
- **Ensemble diversity**: different algorithms, feature sets, training windows. Model disagreements flagged for review.
- **Robust features**: graph features (harder to game than linear/transaction features), behavioral biometrics (require human behavior, hard to automate), multi-window velocity (evade 1hr window but caught by 7d window), entropy features (catch artificially randomized attributes).
- **Online learning**: continuous adaptation (online gradient descent on each new labeled example), adaptive thresholds (adjust based on recent fraud rate and FPR), bandit-based model selection (dynamically choose model based on recent performance).
- **Human-in-the-loop**: escalation triggers for suspicious patterns (uncertainty, probe detection); tight analyst feedback loop (new patterns → features → rule); collaboration with fraud intelligence teams for pattern sharing.

---

## 13. Deployment and Operations

### 13.1 MLOps for Fraud Detection

| Component | Technology | Purpose |
|---|---|---|
| Feature Store | Feast/Tecton + Redis/Aerospike | Online + offline feature serving |
| Model Registry | MLflow / SageMaker | Versioning, lineage, artifacts |
| Model Serving | Triton / ONNX / SageMaker | Low-latency inference |
| Model Monitoring | WhyLabs / Arize / Evidently / Custom | Drift, performance, data quality |
| A/B Testing | Custom framework | Champion/challenger comparison |
| Retraining Pipeline | Airflow / Kubeflow / Prefect | Automated retraining |
| Orchestration | Kubernetes | Resource management, scaling |

### 13.2 Model Serving Challenges

**Inference Budget**:

| Model Type | Inference Time | Notes |
|---|---|---|
| Tree models (LightGBM, XGBoost) | <1ms (CPU) | No GPU needed, batch 1 |
| Neural networks | 1-5ms (GPU) | With dynamic batching: 0.5-2ms |
| Graph models (GNN) | 5-50ms | Depends on neighbor sampling depth |
| Full ensemble (3-5 models) | 2-10ms | Parallel independent model execution |

**Optimization**: ONNX Runtime (cross-framework graph optimization), Triton (dynamic batching, GPU optimization), INT8 quantization (2-4× speedup, <1% accuracy loss), model pruning (30-50% feature reduction), micro-batching (2-5 txns when latency permits), custom C++ serving for latency-critical paths.

**Feature Freshness**: real-time features computed within decision window (<50ms). Pre-computed from feature store (<5ms). Local LRU cache for hot features (sub-ms). Cache warming for expected high-volume customers. Stale feature handling with degraded confidence and explicit staleness signals.

**High Availability**: multi-AZ deployment, fallback to rule-only on model server failure, load shedding (drop enrichment, skip expensive models, reduce feature count), circuit breakers (route around flapping data sources for N seconds), health checks (/health endpoint, LB removal), rate limiting at the inference layer.

### 13.3 Deployment Strategies

- **Shadow scoring**: new model runs alongside champion, no customer impact. Shadow predictions logged for comparison. Minimum 7-14 days.
- **Gradual rollout**: 1% → 5% → 25% → 100% with automatic rollback guardrails (FPR > threshold, latency > target).
- **Canary**: new model deployed to subset of serving nodes; real-time metrics comparison; automated promotion gate.
- **Blue/Green**: entire new stack alongside current; atomic traffic switch; immediate rollback.
- **A/B testing**: random traffic split by txn ID hash; champion vs challenger on disjoint sets; 7+ days with statistical significance testing.

### 13.4 Production Monitoring

**System Metrics**:

| Metric | Target | Alert |
|---|---|---|
| Decision latency p50 | <30ms | >50ms |
| Decision latency p95 | <80ms | >150ms |
| Decision latency p99 | <100ms | >200ms |
| Model inference time | <5ms per model | >10ms |
| Feature retrieval time | <5ms | >20ms |
| Throughput (TPS) | Per node target | >90% capacity |
| Error rate | <0.01% | >0.1% |
| Queue depth | <100 | >1000 |
| Timeout rate | <0.001% | >0.01% |

**Model Metrics**: score distribution (p50, p95, mean, std), decision distribution (% approve/review/decline/step-up per channel/MCC/card type), model invocation count per ensemble component, threshold breach alerts.

**Data Pipeline**: source availability, data freshness (txn-to-feature latency), quality failure rates (schema errors, null rate anomalies), stream processing lag (Kafka consumer lag per partition).

**Feature Drift Dashboard**: real-time PSI per feature (7d vs baseline), KDE plots (current vs baseline), top-N drift features, SHAP importance trend, alert history.

### 13.5 Scaling the Pipeline

**Flink**: increase task slots (parallelism with more CPU cores), rebalance key groups (more partitions = finer parallelism), RocksDB on SSD, incremental checkpointing.

**Kafka**: add partitions (rebalance consumer groups), adjust replication factor, tune batch.size and linger.ms, tiered storage for long retention.

**Feature Store**: Redis Cluster with consistent hashing sharding, read replicas, TTL management, cache warming.

**Inference**: horizontal scaling (more nodes behind LB), GPU auto-scaling (by queue depth), model partitioning (split large NNs across GPUs).

**Handling Bursty Loads** (2-10× spikes during holiday shopping, Black Friday, Double 11, flash sales, payday): predictive scaling (schedule scale-up before known events from historical patterns), reactive scaling (CPU/memory/queue/latency metrics), cache pre-warming for expected customers, capacity reservation, load testing with historical peak volumes.

### 13.6 Incident Response

| Trigger | Action |
|---|---|
| FPR > 3% for 5 minutes | Rollback to previous champion model. Verify FPR returns to baseline in 10 minutes. |
| Latency p99 > 200ms for 3 min | Reduce ensemble (skip graph model), increase model server replicas. |
| Feature store error rate > 5% | Route through rule-only engine, reduce risk thresholds. |
| Feature retrieval error rate > 5% | Fall back to cached features, degrade confidence. |
| Fraud capture rate drops >20% vs 7d avg | Investigate missed transactions for common pattern; deploy deterministic rule; fast-track feature dev + retraining. |

---

## 14. Case Management

Case management is the human-in-the-loop component — where flagged transactions become investigation cases for fraud analysts.

### 14.1 Analyst Investigation Flow

```
Alert → Case Created (auto-routed) → Investigation → Decision → Feedback (labels)
  ┌────────────────── Invesigation Tools ──────────────────┐
  │ Customer 360 view (profile, accounts, devices, cases)   │
  │ Transaction history (filterable, sortable, visual)      │
  │ Network visualization (interactive entity graph)        │
  │ Timeline view (events: txns, logins, password changes)  │
  │ Model explanations (SHAP force plot, top features)      │
  │ External data (credit bureau, device intel, sanctions)  │
  └─────────────────────────────────────────────────────────┘
  Decision: Confirm Fraud / False Positive / Inconclusive
  → Labels fed back into model retraining pipeline
```

### 14.2 Case Management System Features

**Case Routing**: rules-based (high-value→senior, type→specialist), skill-based (complex→expert), round-robin (standard), supervisor override for workload balancing.

**Investigation Tools**: Customer 360 (unified profile, accounts, devices, IPs, transaction history, previous cases, notes), full transaction history (filters, aggregation, spending visualization), interactive graph visualization (accounts-devices-IPs-phones-merchants relationships), timeline view (chronological events), document management (evidence, screenshots, communications).

**Decision Support**: SHAP force plot per transaction, similar case search (feature vector k-NN across historical cases), risk scoring breakdown (by model and by risk category: amount/velocity/device/network), automated compliance report generation.

### 14.3 At-Scale Case Management

**Automated Triage**: high-confidence auto-decline (score >0.99 + multiple rules → decline without review), high-confidence auto-approve (score < threshold + zero rules → approve without review), only edge cases reach humans. Triage model (LightGBM) predicts whether a case benefits from human review. Reduces case volume from thousands to hundreds daily.

**Investigator Productivity**: batch actions (approve/decline/block on multiple cases), templated responses (pre-written notes and communications), AI-assisted investigation (LLM summarizing case details for rapid review, natural language querying of transaction history), similar case search.

**Quality Assurance**: 5-10% of analyst decisions reviewed by QA team; per-analyst precision/recall tracking; calibration training on new fraud patterns and model behavior changes; inter-rater reliability measurement.

### 14.4 Tooling Integration

SIEM integration (alert export to security tools), threat intelligence feeds (fraud ring indicators, compromised device data, credential leaks), external data sources (credit bureaus: Experian/Equifax/CTS; sanctions screening: World-Check/LexisNexis; device intelligence: iovation/ThreatMetrix), customer communication channels (automated SMS/email/in-app verification prompts, fraud alerts, dispute processing integration), alerting and notification for high-value fraud and system issues.

### 14.5 Payment Network Integrations

| Network | Integration | Description |
|---|---|---|
| **Visa** | VAA (Visa Advanced Authorization) | Transaction scoring + risk data exchange |
| **Visa** | TC40 | Chargeback notification and fraud reporting |
| **Mastercard** | GCMS (Global Card Mgmt System) | Card management, fraud reporting |
| **Mastercard** | SAFE (System to Automate Fraud Email) | Fraud case reporting, intelligence sharing |
| **SWIFT** | ISO 20022 / ISO 8583 | Cross-border payment fraud, card authorization |
| **All** | Fraud Indicator Codes (FIC) | Standardized fraud codes |
| **All** | Network alerts | Compromised cards, merchant breaches, new patterns |

Chargeback data (30-90 day delayed) provides definitive ground truth labels for model retraining — the gold standard feedback loop.

---

## 15. Conclusion

Financial fraud detection at scale is one of the most demanding ML engineering challenges in production. Success requires:

1. **Sub-100ms decision latency** at tens of thousands of TPS — Kafka + Flink + optimized ML serving (ONNX/Triton) are mandatory.
2. **Hundreds to thousands of real-time features** — feature stores (Redis + S3) with point-in-time correctness, drift monitoring, and anti-adversarial design.
3. **Ensemble of models** — gradient boosted trees for tabular features, neural networks for high-cardinality and sequences, GNNs for fraud ring detection, anomaly detectors for unknown types.
4. **Continuous retraining** — weekly/biweekly champion/challenger cycles with A/B testing and automated rollback.
5. **Adversarial resilience** — adversarial training, diverse ensembles, robust features (graph, behavioral, multi-window), online learning.
6. **Regulatory compliance** — MAS FEAT: SHAP explainability for every decline, fairness monitoring, model governance; SR 11-7: independent validation, documentation, annual review.
7. **Operational excellence** — multi-AZ HA, graceful degradation, load shedding, circuit breakers, comprehensive p50/p95/p99 monitoring with automated runbooks.

The frontier continues to evolve: transformer-based sequence modeling, heterogeneous GNNs for multi-entity graphs, online learning with bandit algorithms for adaptive thresholds, and LLM-assisted case investigation for analyst productivity. Institutions investing in data infrastructure, feature engineering pipelines, and model lifecycle management will maintain the detection advantage as fraudsters become increasingly sophisticated with AI-driven attack methods.

The architecture described in this guide provides a blueprint for building systems that catch the fraudulent few while letting the legitimate millions flow uninterrupted — at scale, in real-time, under regulatory scrutiny, and against adaptive adversaries.

---

> **Author's Note**: This guide was written for financial technology professionals, solution architects, ML engineers, and fraud operations teams designing, building, or operating fraud detection systems. The principles and architectures are broadly applicable across retail banking, card issuing, payment processing, digital banking, and fintech environments.
