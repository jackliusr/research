# On-Premises vs Cloud Lakehouse: A Critical Analysis

*A decision framework for when to build, when to buy, and when to do both.*

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [The TCO Trap: What Everyone Gets Wrong](#the-tco-trap-what-everyone-gets-wrong)
3. [TCO Comparison by Scale (Real Numbers)](#tco-comparison-by-scale-real-numbers)
4. [When On-Prem Makes Sense](#when-on-prem-makes-sense)
5. [When On-Prem Does NOT Make Sense](#when-on-prem-does-not-make-sense)
6. [The Grey Zone: Indeterminate Workloads](#the-grey-zone-indeterminate-workloads)
7. [Hybrid Approaches: Best of Both Worlds](#hybrid-approaches-best-of-both-worlds)
8. [Decision Framework with Scoring Matrix](#decision-framework-with-scoring-matrix)
9. [Case Studies](#case-studies)
10. [Hidden Costs and Risk Factors](#hidden-costs-and-risk-factors)
11. [Decision Flowchart](#decision-flowchart)
12. [Conclusion](#conclusion)

---

## Executive Summary

The on-premises vs cloud lakehouse debate is not a technical question — it is a financial and strategic one. Both architectures run the same software stack (Iceberg, Spark, Trino, MinIO). Both deliver ACID transactions on object storage. The difference is who owns the hardware, who bears the operational risk, and how costs scale with utilization.

**The short answer:** On-premises lakehouses win on TCO above ~500 TB of steady-state data when utilisation exceeds 60-70%, or when regulatory constraints rule out cloud. Cloud lakehouses win below ~100 TB, for variable workloads, or when organisational velocity (experimentation, ML, time-to-market) matters more than unit economics.

**The nuanced answer:** This guide provides a decision framework with real-world cost data, a scoring matrix, and hybrid architecture patterns so you can rationalise your choice with your CFO and your CISO in the same meeting.

---

## The TCO Trap: What Everyone Gets Wrong

### The Cloud Vendor Framing

Cloud providers frame the comparison as CapEx (on-prem) vs OpEx (cloud). This is misleading. The real trade-off is:

| | Cloud Framing | Reality |
|---|---|---|
| **On-prem** | High upfront, inflexible | Predictable cost at scale, zero egress |
| **Cloud** | Pay-as-you-go, elastic | Bill shocks from egress, API calls, storage class transitions |

### Hidden Cloud Costs

- **Egress fees:** $0.05-0.12/GB outbound. 1 PB/yr = $50K-120K/yr invisible cost.
- **API request costs:** S3 PUT/GET/LIST add $500-5K/mo at petabyte scale.
- **Storage class transitions:** $0.01-0.03/1,000 transitions when lifecycle-expiring S3 Standard → Glacier.
- **Multi-region replication:** $0.02-0.06/GB cross-region for DR.
- **Support plan overhead:** Enterprise support at 10% of spend.

### Hidden On-Prem Costs

- **Staffing:** 2-3 platform + 1 storage/network engineer minimum.
- **Power/cooling density:** NVMe + 25GbE nodes push 15-20 kW/rack.
- **Hardware refresh:** 3-5 year lifecycle, nodes replaced in lockstep.
- **Spare capacity:** Must over-provision 30-50% for peak + failover.
- **Procurement friction:** 8-16 week lead times for enterprise storage hardware.

### The Real Comparison Framework

| Item | On-Prem (3-yr amortised) | Cloud (pay-as-you-go) |
|---|---|---|
| Storage (raw, per TB/yr) | $35-60 | $276 (S3 Standard) |
| Compute (per vCPU/yr, 50% util) | $150-250 | $350-600 (EC2 + markup) |
| Network (per Gbps/mo) | $15-30 | $20-50 (NAT GW + DX) |
| **Total at 1 PB active (3 yr)** | **$1.2-2.0M/yr** | **$2.8-4.5M/yr** |

---

## TCO Comparison by Scale (Real Numbers)

All figures are annualised TCO including hardware (3-year amortisation), software licensing, staffing, facilities (power, cooling, rack space), and cloud services (compute, storage, egress, support). Excludes existing DC sunk costs.

### 100 TB Scale (Small)

| Cost Component | On-Premises (Annual) | Cloud (Annual) | Notes |
|---|---|---|---|
| Storage (raw + parity) | $15,000-25,000 | $28,000-35,000 | On-prem: 6 nodes × 4x4TB SSD, EC:4:2. Cloud: S3 Standard |
| Compute (steady) | $30,000-45,000 | $48,000-90,000 | On-prem: 4 Trino+Spark nodes. Cloud: 4-6 nodes on-demand |
| Staffing (fractional) | $60,000-100,000 | $30,000-60,000 | On-prem needs dedicated infra engineer |
| Network + facilities | $8,000-12,000 | $5,000-10,000 | Cloud includes inter-AZ; on-prem needs switches + UPS |
| Support/licensing | $5,000-15,000 | $15,000-30,000 | Cloud support at 10% of compute+storage |
| Egress/data transfer | $0 | $6,000-18,000 | Assumes 1-2 TB/mo analytics output |
| **Total Annual** | **$118K-197K** | **$132K-243K** | |
| **$/TB/yr** | **$1,180-1,970** | **$1,320-2,430** | |

**Verdict:** Cloud is competitive at 100 TB. The staffing delta is the deciding factor. Teams without existing DC operations should choose cloud. Teams with existing DC ops and unused rack capacity may prefer on-prem.

### 1 PB Scale (Mid-Large)

| Cost Component | On-Premises (Annual) | Cloud (Annual) | Notes |
|---|---|---|---|
| Storage (raw + parity) | $50,000-80,000 | $280,000-350,000 | On-prem: 12 nodes × 12x8TB SSD, EC:8:4. Cloud S3 Standard |
| Compute (steady 60%) | $120,000-180,000 | $360,000-720,000 | On-prem: 16 Spark + 8 Trino nodes. Cloud: reserved instances at 40% discount |
| Staffing (dedicated) | $180,000-280,000 | $100,000-150,000 | On-prem: 1.5-2 FTE platform + 0.5 FTE storage |
| Network + facilities | $25,000-40,000 | $15,000-25,000 | On-prem: 25GbE leaf-spine, UPS, cooling |
| Support/licensing | $30,000-60,000 | $80,000-150,000 | Cloud: Enterprise support + DBU/warehouse markup |
| Egress/data transfer | $0 | $50,000-120,000 | 50-100 TB/yr BI output + inter-region replication |
| **Total Annual** | **$405K-640K** | **$885K-1,515K** | |
| **$/TB/yr** | **$405-640** | **$885-1,515** | |

**Verdict:** On-prem wins 40-70% TCO advantage at 1 PB steady-state. The gap widens with egress-heavy workloads (BI/reporting). Cloud still viable for organisations prioritising speed over cost (e.g., fast-growing startups, AI-first companies).

### 10 PB+ Scale (Enterprise)

| Cost Component | On-Premises (Annual) | Cloud (Annual) | Notes |
|---|---|---|---|
| Storage (raw + parity) | $350,000-600,000 | $2,800,000-3,500,000 | On-prem: 48 nodes × 20TB HDD + SSD cache tier. Cloud S3 Standard, potential bulk discounts |
| Compute (steady 70%) | $800,000-1,400,000 | $2,400,000-4,800,000 | On-prem: 200+ compute nodes across multiple racks |
| Staffing (team) | $500,000-800,000 | $250,000-400,000 | On-prem: 4-6 FTE (platform, storage, network, security) |
| Network + facilities | $180,000-300,000 | $80,000-150,000 | On-prem: 100GbE spine-leaf, dedicated DC space, power |
| Support/licensing | $200,000-400,000 | $500,000-1,000,000 | Enterprise license agreements; cloud vendor negotiate |
| Egress/data transfer | $0 | $500,000-1,200,000 | >500 TB/yr egress; cross-region replication costs |
| **Total Annual** | **$2.03M-3.50M** | **$6.53M-11.05M** | |
| **$/TB/yr** | **$200-350** | **$650-1,100** | |

**Verdict:** On-prem dominates at 10 PB+. The cost advantage is 3-5x. Cloud only makes sense if (a) the workload is highly variable, (b) the organisation has no DC footprint, or (c) regulatory arbitrage makes cloud storage of this magnitude infeasible.

### Scale Summary

**Key insight:** The cross-over point is ~300-500 TB of active managed data. Below this, cloud's operational elasticity compensates for unit-cost premium. Above it, on-prem's structural cost advantage becomes decisive — provided utilisation stays above 50-60%.

---

## When On-Prem Makes Sense

### 1. Data Sovereignty & Regulatory Compliance

**Regimes where on-prem is effectively mandatory:**

| Regulation | Jurisdiction | Requirement | Lakehouse Impact |
|---|---|---|---|
| GDPR | EU/EEA | Data stays in EU or adequacy jurisdiction | Cloud regions exist but CLOUD Act risk remains |
| Data Security Law | China | Critical data stays in-country | Must use Chinese cloud (Alibaba, Huawei) or on-prem |
| DPDP Act | India | Personal data stored in India | Domestic cloud available but audit access is limited |
| PDPL | Saudi Arabia | Personal data within KSA | On-prem or Saudi-hosted cloud (limited vendor choice) |
| MAS Guidelines | Singapore | Financial data controlled locally | Cloud approved with stringent contractual safeguards |
| BaFin/ECB | EU Banking | Audit-proof data processing | On-prem preferred; cloud requires MaRisk-compliant review |
| **Fed/OCC** | US Banking | Third-party risk management | Cloud permitted, exam expectations high |

**The CLOUD Act Risk:** The US CLOUD Act (2018) permits US law enforcement access to data held by US-based cloud providers regardless of DC location. For EU financial institutions, this is a board-level concern. On-prem infrastructure by a non-US entity is outside CLOUD Act reach.

**Financial sector reality:** Most tier-1 European banks run hybrid — on-prem for regulated customer data, cloud for non-regulated analytics.

### 2. >1 PB Active Data with Steady Utilisation

At petabyte scale, the math is undeniable:

- Cloud: $650-1,100/TB/yr at 10 PB
- On-prem: $200-350/TB/yr at 10 PB
- Savings at 10 PB: $4.5-7.5M/year

The savings fund the team, the hardware refresh, and the DC expansion — with enough left over to buy the CFO a data centre tour.

### 3. Existing Data Centre Investment

If you already own:
- DC space with spare power/cooling/capacity
- Network fabric (25/100 GbE)
- Storage area network or existing server fleet
- Staff with storage/networking/ops skills

The incremental cost of adding a lakehouse is dramatically lower than greenfield on-prem. The marginal cost of adding MinIO nodes to an existing rack is ~40-50% of the full loaded cost.

**Sunk cost fallacy warning:** Just because you have a DC does not mean you should fill it. If the DC is at capacity, has poor PUE (>1.6), or is due for lease renewal, cloud may still be cheaper.

### 4. Air-Gapped Environments

Zero internet = zero cloud. Defence, intelligence, critical infrastructure, nuclear, and certain government systems operate without external connectivity. The on-prem lakehouse (MinIO + Iceberg + Trino on internal registries) is the only architecture available.

**Unique requirements:**
- All container images must come from internal registries
- No phoning home for license validation
- Hardware maintenance contracts must include on-site spares
- Software updates delivered via physical media or validated VPN

### 5. Sustained High-Throughput Workloads

Workloads that run 24/7 at high utilisation (trading systems, risk calculations, continuous ingestion from IoT/SCADA) benefit from on-prem because:

- **Deterministic performance.** No noisy-neighbour variance on bare metal.
- **Predictable cost.** Flat power + amortised hardware beats per-query billing.
- **Latency.** When data sources are on-prem, cloud round-trip adds 5-50ms of unnecessary latency.
- **Throughput.** 100 GbE on bare metal delivers 12.5 GB/s sustained. Equivalent cloud instance networking is rate-limited and shared.

---

## When On-Prem Does NOT Make Sense

### 1. Variable or Bursty Workloads

If your workload runs 2 hours a day, once a week, or spikes seasonally (e.g., Black Friday analytics, monthly regulatory reporting), on-prem forces you to pay for idle hardware 80-90% of the time.

| Workload Pattern | On-Prem | Cloud |
|---|---|---|
| 24/7 sustained (80%+ util) | ✅ Best fit | ❌ Overpaying |
| Business hours (40% util) | ⚠️ Wastes 60% | ✅ Auto-scale down |
| Ad-hoc / experimental (<10% util) | ❌ Terrible | ✅ Pay per query |
| Seasonal spikes (e.g., holiday retail) | ❌ Over-provision for peak | ✅ Burst then shrink |

### 2. <100 TB Active Data

Below 100 TB, the economics don't favour on-prem:

- Hardware minimum viable cluster: 4-6 nodes ≈ $80-120K
- Staffing still needs a human (even part-time)
- Cloud offers $0 upfront, immediate provisioning, zero procurement
- Cloud's per-TB premium is small in absolute dollars ($15-25K/yr delta)

The break-even analysis: at 100 TB, on-prem costs $118-197K/yr vs cloud $132-243K/yr. The delta is ~$15-45K/yr — less than one junior engineer. Unless you already have a DC, choose cloud.

### 3. Rapid Experimentation and Data Science

On-prem provisioning cycles (8-16 week hardware procurement) kill velocity for:
- Data science proof-of-concepts
- ML model training on new datasets
- Ad-hoc exploration of unstructured data
- Evaluating new tools (e.g., trying Polaris vs Nessie)

Cloud lets you spin up a lakehouse in minutes, test for a week, and tear it down. On-prem requires a capital request, a purchase order, a racking party, and a configuration sprint before you can run your first query.

### 4. AI/ML-Heavy Workloads

GPU scarcity on-prem is a real problem. Training clusters (NVIDIA H100/B200) face 26-52 week lead times, while cloud GPU instances are available on demand. Even with on-prem data lakes:

- **Training:** Cloud GPUs with data plane connectivity back to on-prem storage
- **Inference:** On-prem for latency-sensitive; cloud for burst
- **Fine-tuning:** Cloud for experimentation; on-prem for production

The sweet spot: data lake on-prem, ML training in cloud (hybrid pattern — see below).

### 5. Small Team (≤3 Data Engineers)

An on-prem lakehouse needs:
- 1 platform engineer (K8s, networking, storage ops)
- 1 data engineer (Spark, Trino, Iceberg)
- 0.5 security/compliance engineer

Without this headcount, the on-prem lakehouse becomes a snowflake (irony intended) — a fragile, under-maintained, single-point-of-failure system. Small teams should absolutely choose a fully managed cloud lakehouse.

### 6. Multi-Region Active-Active

Cloud's global infrastructure (S3 cross-region replication, Route53 global endpoints, CloudFront edge caching) is vastly cheaper and easier to operate than building equivalent on-prem across multiple DCs or regions. If you need truly global active-active with <1 second RPO, cloud wins.

---

## The Grey Zone: Indeterminate Workloads

### 500 GB-3 TB/day Ingestion

| Factor | Leans On-Prem | Leans Cloud |
|---|---|---|
| Growth trajectory | Predictable (10-20%/yr) | Exponential (50-100%/yr) |
| Retention period | >5 years | <1 year |
| Query pattern | Repetitive, scheduled | Ad-hoc, varied |
| Compliance | Regulated (PII, financial) | Non-regulated |

**Recommendation:** Start cloud, build on-prem business case at ~300 TB. Re-evaluate every 6 months.

### Data Warehouse Migration (Teradata/Netezza/Oracle Exadata)

| Factor | Modernise In-Place | Migrate to Cloud |
|---|---|---|
| Hardware refresh due | Yes (3+ year old) | Yes (3+ year old) |
| Existing staff skills | Teradata/Netezza | Spark/Trino |
| Budget model | CapEx available | OpEx preferred |

**Recommendation:** If hardware refresh is due and staff skills transferable, cloud migration is a clean break opportunity. If staff retention is a concern, modernise in-place with Iceberg on MinIO.

---

## Hybrid Approaches: Best of Both Worlds

### Pattern 1: Hot On-Prem → Cold Cloud (Tiered Storage)

```
┌──────────────────┐          ┌──────────────────┐
│   On-Prem DC     │          │   Cloud Region    │
│                  │          │                  │
│  MinIO (Hot)     │  async   │  S3 (Cold/Archive)│
│  Iceberg tables  │ ───────> │  Glacier / Deep   │
│  Active data     │          │  Archive          │
│  <90 days        │          │  >90 days data    │
│                  │          │                  │
│ Trino / Spark    │          │ (No compute)     │
└──────────────────┘          └──────────────────┘
```

**Use case:** On-prem stores hot data (<90 days) for fast analytics. Data older than 90 days transitions to S3 Glacier via lifecycle policies. Queries on cold data are rare and tolerate 5-10 minute restore latency.

**Cost advantage:** Combines on-prem economics for hot data ($200-350/TB/yr) with cloud archival ($12-24/TB/yr for Glacier Deep Archive). Overall blended cost approaches on-prem-only figures while gaining unlimited cloud archival capacity.

**Implementation:** Iceberg supports multi-hadoop configuration — set `io-impl` to MinIO for active tables and S3 for archival tables. Or use Nessie catalog with per-table storage configuration.

### Pattern 2: Cloud Burst for ML Training

```
┌──────────────────┐          ┌──────────────────┐
│   On-Prem DC     │          │   Cloud Region    │
│                  │          │                  │
│  MinIO + Iceberg │  VPN/DX  │  GPU Cluster     │
│  (all data)      │ ◄──────> │  (H100/B200)     │
│                  │          │                  │
│  Spark ETL       │          │  PyTorch/TF      │
│  Trino SQL       │          │  Spark on K8s    │
│  Airflow         │          │  (ephemeral)     │
└──────────────────┘          └──────────────────┘
```

**Use case:** On-prem stores the canonical dataset. For ML training, ephemeral GPU clusters in cloud read Iceberg tables over a dedicated connection (AWS Direct Connect / Azure ExpressRoute). Cloud compute shuts down when training completes.

**Cost advantage:** No GPU idle cost on-prem. Cloud GPUs are expensive but only used for active training. Data stays on-prem for compliance.

**Performance considerations:**
- Data transfer over Direct Connect: 10-40 Gbps sustained
- Alluxio cache on cloud side reduces repeated WAN reads
- Prefer co-located cloud region (latency <10 ms RTT)
- For 10+ PB datasets, consider cloud-side MinIO replica with async sync

### Pattern 3: Active-Passive DR

```
┌──────────────────┐          ┌──────────────────┐
│   On-Prem DC     │          │   Cloud Region    │
│   (Primary)      │          │   (DR)            │
│                  │          │                  │
│  MinIO + Iceberg │  async   │  MinIO / S3      │
│  Trino + Spark   │ ───────> │  (warm standby)  │
│  Nessie catalog  │          │  Trino (stopped)  │
│                  │          │                  │
│  RPO: seconds    │          │  RTO: 30 min     │
└──────────────────┘          └──────────────────┘
```

**Use case:** On-prem is primary; cloud is DR site. No secondary DC needed. Pay only for cloud storage (cold/warm) plus infrequent DR test compute.

**Cost:** Cloud storage for a petabyte replica at S3 Standard costs ~$23K/mo (infrequent access class reduces this). Compared to a second DC with identical hardware ($50-100K/mo for rack + power + networking), this is a 50-75% DR cost reduction.

### Pattern 4: Federated Query Across Both

```
┌──────────────────┐          ┌──────────────────┐
│   On-Prem DC     │          │   Cloud Region    │
│                  │          │                  │
│  Trino Cluster   │  federated│ Trino / Athena   │
│  on MinIO        │ ◄──────> │ on S3 / Redshift │
│  (core data)     │          │ (SaaS data)      │
│                  │          │                  │
│  Nessie Catalog  │          │ AWS Glue Catalog │
└──────────────────┘          └──────────────────┘
```

**Use case:** On-prem holds authoritative data; cloud holds SaaS data (Salesforce, Marketo, Zendesk, etc.). Trino on-prem has both Iceberg and JDBC/Athena connectors, allowing cross-platform joins in a single SQL query. Cloud data is never copied on-prem; on-prem data never leaves.

**Implementation:** Trino `iceberg` catalog on Nessie + `delta_lake` or `jmx` or `postgresql` catalog pointing to cloud sources. Avoid full cross-cloud scans — push predicates aggressively.

---

## Decision Framework with Scoring Matrix

### Scoring Methodology

Rate each criterion from 1 (strongly favours cloud) to 5 (strongly favours on-prem). Sum the scores and compare to the decision bands below.

| # | Criterion | Weight | 1 (Cloud) | 2 | 3 (Neutral) | 4 | 5 (On-Prem) |
|---|---|---|---|---|---|---|---|
| 1 | **Active data volume** | 3× | <50 TB | 50-200 TB | 200-500 TB | 500 TB-2 PB | >2 PB |
| 2 | **Workload variability** | 2× | >80% variable | 60-80% | 40-60% | 20-40% | <20% variable |
| 3 | **Regulatory constraints** | 3× | None | Light (SOC2) | Moderate (HIPAA) | Strict (GDPR+) | Air-gapped/classified |
| 4 | **Existing DC investment** | 2× | No DC | Renting colo | Own DC, full | Own DC, spare capacity | Own DC, underutilised |
| 5 | **AI/ML intensity** | 2× | Heavy ML, GPU | ML moderate | Balanced | Analytics heavy | Batch/ETL only |
| 6 | **Staff availability** | 2× | <2 data eng | 2-3 data eng | 3-5 engineers | 5-8 engineers | 8+ engineers |
| 7 | **Time-to-market need** | 1× | Days (startup) | Weeks | Months | Quarters | Years (stable) |
| 8 | **Growth rate** | 2× | >100%/yr | 50-100%/yr | 20-50%/yr | 10-20%/yr | <10%/yr |
| 9 | **Latency sensitivity** | 1× | <100ms tolerate | 100-500ms | 500ms-1s | 1-5s | Sub-millisecond |
| 10 | **Budget model** | 2× | OpEx only | Mostly OpEx | Mixed | Mostly CapEx | CapEx available |
| 11 | Data egress volume | 2× | <10 TB/yr | 10-50 TB/yr | 50-200 TB/yr | 200 TB-1 PB/yr | >1 PB/yr |

### Scoring

1. Multiply each score (1-5) × weight (1-3). Sum all weighted scores. Divide by sum of weights (21). Multiply by 100 for percentage.

**Maximum (all 5s):** `(5×3)+(5×2)+(5×3)+(5×2)+(5×2)+(5×2)+(5×1)+(5×2)+(5×1)+(5×2)+(5×2)` = 15+10+15+10+10+10+5+10+5+10+10 = **110**

**Minimum (all 1s):** 3+2+3+2+2+2+1+2+1+2+2 = **22**

### Decision Bands

| Score % | Recommendation | Strategy |
|---|---|---|
| **0-35%** | **Strong Cloud** | Use managed lakehouse. Do not build on-prem. |
| **35-50%** | **Cloud-Leaning Hybrid** | Cloud primary; reserve on-prem for specific workloads. |
| **50-65%** | **Neutral / Hybrid** | Conduct detailed TCO analysis. Consider tiered hybrid. |
| **65-80%** | **On-Prem-Leaning Hybrid** | On-prem primary; cloud for burst/DR. |
| **80-100%** | **Strong On-Prem** | Build on-prem. Use cloud only for DR/archival. |

### Example 1: Regulated Bank with 2 PB Active Data

Scores: Active data(4×3=12) + Variability(4×2=8) + Regulatory(5×3=15) + DC invest(4×2=8) + ML low(4×2=8) + Staff(4×2=8) + Time-to-market(3×1=3) + Growth(4×2=8) + Latency(3×1=3) + CapEx available(4×2=8) + Egress 200TB(4×2=8) = **88/110 = 80% → Strong On-Prem**

### Example 2: AI Startup, 50 TB, 100%/yr Growth

Scores: Active data(1×3=3) + Variability(1×2=2) + Regulatory(1×3=3) + DC invest(1×2=2) + ML heavy(1×2=2) + Staff(2×2=4) + Time-to-market(1×1=1) + Growth(1×2=2) + Latency(2×1=2) + OpEx(1×2=2) + Egress(1×2=2) = **25/110 = 23% → Strong Cloud**

---

## Case Studies

### Case Study 1: European G-SIB Bank (Banking)

**Profile:** €2T+ AUM, 50+ country operations. Multi-PB Teradata + Hadoop estate. 20%/yr growth.

**Situation:** Regulatory pressure (PSD2, GDPR, French *secret bancaire*, BaFin/ECB rules). Cloud vendor full-migration TCO: €18M/3yr vs on-prem modernisation: €7.5M/3yr.

**Decision:** On-prem lakehouse (MinIO + Iceberg + Trino) as primary. Cloud (Azure) for market data feeds, ML GPU training, and DR warm standby.

**Outcome:** 1,200+ tables migrated. Live in 14 months. 3-year TCO actual: €8.2M (within 10% of estimate). Cloud DR tested quarterly. Staff Spark/SQL skills transferred directly; no 6-9 month cloud retraining needed.

**Lessons:** Hardware procurement was the critical path (8 months). Cloud burst for ML introduced €45K/mo unplanned egress costs. Nessie branching adopted by compliance for audit isolation.

### Case Study 2: Federal Statistical Office (Government)

**Profile:** European national statistical agency. 2,000 employees. Census, economic indicators. 800 TB, 10-15%/yr growth with decennial 5× spikes.

**Situation:** Mandated to keep all citizen data within national borders. Cloud offered domestic DCs but procurement rules created jurisdictional tangle.

**Decision:** On-prem lakehouse for all PII/regulated data. Cloud (Poland region) for anonymised public datasets. MinIO (16 nodes × 8×7.68TB SSD, EC:8:4) + Trino (20 workers) + Spark on K8s. Polaris catalog. Pre-provisioned 12 extra nodes for 3-month census burst.

**Outcome:** Operational in 16 months. 2025 census processed in 6 weeks vs 14 weeks (previous Hadoop). Ongoing cost: €850K/yr vs projected €2.1M/yr AWS equivalent.

**Lessons:** Census over-provisioning wasted €320K in idle hardware — cloud burst would have been cheaper. Staff retention risk: 2 of 5 platform engineers left within 18 months. Polaris catalog needed 2 emergency metadata-corruption patches.

### Case Study 3: National Healthcare System (Healthcare)

**Profile:** Single-payer system, 65M lives. EHR, claims, genomics, imaging. 5 PB, 30%/yr growth.

**Situation:** Cloud migration paused after data protection authority flagged US-cloud access to health data under HIPAA-like regime. Research compute needed GPUs unavailable on-prem.

**Decision:** Tiered hybrid — on-prem for active clinical data (last 2 years, MinIO all-flash), on-prem HDD for 6-24 month data, Azure Cool/Archive for >2 year data. Trino on-prem for clinical queries; Azure Databricks for genomic research (burst). Nessie (on-prem) + Azure Purview dual catalog.

**Outcome:** Blended cost €2.95M/yr vs €5.8M/yr (all-cloud) or €4.2M/yr (all-on-prem). 85% of clinical queries served from hot tier with <100ms P99. Cold tier queries tolerate 30-60s archive restore.

**Lessons:** Dual-catalog operational complexity was significant — two permission models, two backends. Data lifecycle (on-prem → cloud) required custom scripting. Research teams strongly preferred Databricks notebooks over on-prem Jupyter.

### Case Study 4: Global Streaming Platform (Media)

**Profile:** Top-5 streaming service, 250M+ subscribers. 15+ PB analytics data (clickstream, viewing, content metadata). Monthly cloud bill: $2.8M, growing 15% QoQ.

**Situation:** Content metadata is stable, high-value, margin-sensitive. Clickstream is ephemeral (90-day retention), highly variable (10× new-release spikes).

**Decision:** Partial repatriation — 6 PB (content metadata + user behavior) moved on-prem. 9 PB (clickstream, ad logs, experiments) stayed in AWS. On-prem: MinIO (48 nodes × 12×20TB HDD + NVMe cache) in LA metro DC (existing CDN presence). Cloud: S3 + Athena + EMR. Trino on-prem federates to AWS Glue for cross-platform joins. Recommendation models train on cloud GPUs, read MinIO over Direct Connect.

**Outcome:** Analytics bill dropped from $2.8M/mo to $1.1M/mo (on-prem $0.5M + cloud $0.6M). 18-month payback on $8.5M hardware investment. Content metadata queries 3× faster (deterministic bare-metal vs burstable cloud).

**Lessons:** Training data egress (on-prem → cloud GPU) added $65K/mo unanticipated cost. Cross-platform joins were awkward without unified catalog — Nessie with S3 connector added in phase 2. On-prem staffing needed 3 additional engineers beyond initial estimate. 6-month repatriation was too aggressive; 9 months would have been realistic.

---

## Hidden Costs and Risk Factors

### On-Prem Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Hardware lead times | 8-16 weeks servers, 16-32 weeks GPUs | Maintain buffer stock; negotiate pre-positioned inventory |
| Staff turnover | Single points of failure | Cross-train; document; run DR drills |
| Capacity planning | Under/over-provision waste | Monitor utilisation; build +20% headroom annually |
| Hardware failure | 1-5%/yr disk failure rate | Erasure coding; hot spares; 4-hour support |
| Security compliance | Infra-layer responsibility | Automated patching; vulnerability scanning; access controls |

### Cloud Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Bill shock | Uncontrolled ad-hoc query spend | Budget alerts; usage governance; committed-use discounts |
| Lock-in | Proprietary APIs (Glue, Athena, Redshift) | Use Iceberg; avoid vendor-specific features |
| CLOUD Act exposure | US access to EU-based US cloud data | On-prem for regulated data; contractual clauses |
| Noisy neighbour | Shared infra variance | Provisioned IOPS; dedicated instances |
| Egress costs | Underestimated data movement cost | Cache on-prem; negotiate egress discounts |
| Provider dependency | Pricing power at scale | Multi-cloud; maintain on-prem capability |

---

## Decision Flowchart

```
                    ┌─────────────────────────────┐
                    │ Do you have regulatory       │
                    │ constraints (GDPR, banking,   │
                    │ health, air-gap, classified)? │
                    └─────────────┬───────────────┘
                              YES│        NO
                                ▼           ▼
                 ┌─────────────────────┐  ┌──────────────────────┐
                 │ Can cloud provider  │  │ Active data > 500TB  │
                 │ satisfy in-region?  │  │ (steady utilisation)?│
                 └──────┬──────┬──────┘  └──────┬──────┬───────┘
                     YES│    NO│           YES│     NO│
                        ▼       ▼              ▼        ▼
             ┌──────────────────┐  ┌───────────────────┐   ┌──────┐
             │ Consider hybrid: │  │ ON-PREM           │   │CLOUD │
             │ on-prem for      │  │ (regulatory       │   │      │
             │ regulated data   │  │  override)        │   │      │
             │ cloud for rest   │  └───────────────────┘   └──────┘
             └──────────────────┘
                        │
                        ▼
              ┌─────────────────────┐
              │ Evaluate scoring    │
              │ matrix (page X)     │
              └─────────────────────┘
                        │
            ┌───────────┴───────────┐
         <50%                  50-65%                  >65%
            ▼                       ▼                       ▼
     ┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐
     │  Cloud       │    │  Hybrid (tiered) │    │  On-Prem         │
     │  Primary     │    │  Evaluate TCO    │    │  Primary         │
     └──────────────┘    │  at scale        │    └──────────────────┘
                         └──────────────────┘
```

---

## Conclusion

The cloud vs on-prem lakehouse decision is not a religion — it is an economic and regulatory calculation that changes with scale, workload pattern, and organisational maturity.

**Three rules to remember:**

1. **Scale is the primary driver.** Below 300 TB, cloud wins on simplicity. Above 1 PB on-prem wins on cost (if utilisation >50%). Between, it depends on everything else.

2. **Regulation can override economics.** If data cannot legally leave your DC, the cloud debate is academic. Design for on-prem with a cloud escape hatch for non-regulated workloads.

3. **Hybrid is sophistication, not failure.** The most mature organisations tier: on-prem for hot/regulated, cloud for burst/DR/archival/ML. The complexity is real but manageable with the right team.

Run the scoring matrix every 12 months. Re-evaluate at every hardware refresh (every 3 years). Cloud pricing, hardware costs, and regulations change — the right answer today may be wrong in 2028.

---

*This analysis complements the [On-Premises Lakehouse Architecture Guide](./on_prem_lakehouse_guide.md) in this repository. That guide covers HOW to build an on-prem lakehouse; this guide covers WHEN and WHY to build one. Both are living documents — update them as the landscape evolves.*

*Author: Jack Liu Shurui, Solution Architect, Crédit Agricole CIB. Based on real engagements, industry benchmarks, and public pricing as of mid-2026. All dollar figures are USD unless otherwise noted.*
