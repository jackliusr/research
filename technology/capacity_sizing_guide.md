# Capacity Sizing: The Sizing Discipline — A Comprehensive Guide

*A deep dive into capacity sizing — the engineering discipline that answers "how big does this system need to be, and how do I prove it?" — covering the overview (the definition, the overview table, the SRE/ITIL frame), the sizing process (demand forecasting, workload profiling, benchmark-based sizing, rule-of-thumb sizing), the sizing math (Little's Law, the utilization targets, peak-vs-average, headroom and growth margin), the per-dimension sizing (compute/CPU, memory, storage, network, GPU, database, queues, app servers), the cloud sizing (rightsizing, autoscaling, reserved capacity), the capacity-management process (the ITIL 4 practice, the monitor→forecast→plan→review loop), the banking context (batch windows, payment peaks, regulatory-reporting deadlines), a worked Cymbal Bank payments-service sizing with the actual math, the one-page summary — the final word is **the sized for the peak** — and the glossary.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** github.com/jackliusr/research · **Category:** Technology Series — the Sizing Discipline · **Date:** August 2026

> **Companion guides (this guide is the dedicated capacity-sizing deep-dive — the repo owns the related disciplines but not the sizing methodology):** the availability/24-7 side in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (the nines math, the batch windows of §9); the cloud-cost/rightsizing side in [finops_guide.md](finops_guide.md) (§3.2 right-sizing thresholds, §6 the purchase options); the cost-modeling side in [tco_modeling_guide.md](tco_modeling_guide.md) (the TCO layer the sizing feeds); the GPU/inference-sizing side in [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) (§4.7 the hardware backdrop, §7 concurrency); the cluster-sizing side in [spark_tuning_guide.md](spark_tuning_guide.md); the back-of-envelope estimates in [google_system_design_interview_guide.md](google_system_design_interview_guide.md), [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) and [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md); the ITIL 4 capacity-and-performance practice in [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) (§2.4 the 34 practices); the queue sizing in [../banking/kafka_guide.md](../banking/kafka_guide.md) (§7.3 throughput/partitions, §10.4 the topic catalog); the DB sizing in [oracle_database_guide.md](oracle_database_guide.md) and [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md); the payment peaks in [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) (§2.4 the peak problem) and [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md); the regulatory-reporting frame in [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); and the investment angle in [../management/business_case_development_guide.md](../management/business_case_development_guide.md) (the sizing budget is a business-case line item).

> **Primary sources (verified this pass where the degraded web backend allowed):** the queueing-theory canon — Little's Law (John D.C. Little's 1961 proof; **L = λW**, verified via the queueing-theory literature this pass) and the M/M/1 results (ρ = λ/μ, stability ρ < 1, mean number in system ρ/(1−ρ) — verified via the M/M/1 reference material); the ITIL 4 canon — the *capacity and performance management* practice, one of ITIL 4's 34 practices (17 service-management practices; ITIL 4 launched February 2019 — verified via the ITIL reference wiki this pass, cross-ref the operational-support guide's ledger); the Google SRE canon — "100% is the wrong reliability target" (✅ via [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §1.2), the SRE Workbook's *Managing Load* and *Identifying and Recovering from Overload* chapters (✅ chapter structure verified on sre.google this pass), the SRE Book's *Handling Overload*; the benchmark canon — TPC-C (approved July 1992, OLTP, five transaction types, nine table types, measured in tpmC — verified on tpc.org this pass); the AWS Well-Architected Performance Efficiency pillar (six-pillar framework, five PE focus areas, Nov 2024 revision — verified on docs.aws.amazon.com this pass); the banking facts — the NETS Gateway's >100 tps, the Visa ~8,500 tps average / >65,000 tps peaks, the FAST 24/7 rail (✅ cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) — **cross-referenced, not re-derived**). **NOTE on this pass's web access: the search backend was degraded and direct extraction was used; anything that could not be pinned to a primary source or a sibling's own verified ledger is flagged ⚠ honestly.** The repo's integrity convention applies: ✅ = verified this pass or cross-referenced from a sibling's verified ledger; ⚠ = flagged — commonly cited but not pinned to a single authoritative public source this pass; ⚠-structural = industry-standard practice, widely documented, not attributable to one primary source. **Never fabricated: every number in the worked example (§8) is either sourced or explicitly labelled as an illustrative planning figure with the arithmetic shown.**

---

## Cross-References (how this guide maps into the repo)

This guide is the **dedicated capacity-sizing treatment** — the discipline the siblings use as an input. Cross-ref heavily rather than re-derive:

- [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — **the availability twin**: the nines math (§2), the 24-7 payments reality (§9), the batch windows and cut-offs (§9.5). Capacity sizing decides *how much*; zero-downtime design decides *how it stays up*. The sized-for-the-peak rule of §3.5 below is what makes the availability targets of that guide achievable.
- [finops_guide.md](finops_guide.md) — **the cloud-cost twin**: §3.2 right-sizing (the <40% / >80% utilization thresholds), §6 the purchasing options (RIs up to 72%, Savings Plans up to 66%, spot up to 90% — verified in that guide's ledger). Sizing decides *what you need*; FinOps decides *what you pay for it*.
- [tco_modeling_guide.md](tco_modeling_guide.md) — the cost layer the sizing feeds: a sizing that over-provisions 2× inflates the §4 model's every line. Cross-ref lightly.
- [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) — **the GPU dimension is cross-referenced, not re-derived**: TTFT/TPOT, continuous batching, KV-cache sizing, the H100/B200 hardware backdrop (§4.7). This guide's §4.5 points there and stops.
- [spark_tuning_guide.md](spark_tuning_guide.md) — the cluster-sizing side for the batch/analytics estate; cross-ref lightly in §4 and §7.
- [google_system_design_interview_guide.md](google_system_design_interview_guide.md) · [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) · [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) — the back-of-envelope estimation discipline (QPS → machines, storage growth) that §3's math formalises.
- [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) — **the ITIL 4 home**: §2.4's verified 34-practice taxonomy (capacity and performance management among the 17 service-management practices), §6 the SRE overlay. This guide's §6 operates that practice.
- [../banking/kafka_guide.md](../banking/kafka_guide.md) — the queue/broker sizing (partitions, retention, the §10.4 topic catalog: 24 partitions for `payments.payment.executed`, 7-year tier-1 retention); this guide's §4.7 cross-refs it.
- [oracle_database_guide.md](oracle_database_guide.md) · [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) — the DB dimension (the PDB/Exadata capacity-pooling model of the Oracle guide §5; the access-pattern sizing of the NoSQL guide); cross-ref lightly.
- [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) · [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) — **the peak-load payments context** (§2.4 "throughput is a peak problem" — Visa ~8,500 tps average with peaks >65,000 tps; the NETS Gateway's >100 tps per-instance floor); this guide's §7.2.
- [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) — the regulatory-reporting frame (MAS returns, BCBS 239) behind §7.3's flagged deadlines.
- [../management/business_case_development_guide.md](../management/business_case_development_guide.md) — the investment frame: every sizing decision is a budgeted line; cross-ref lightly in §6 and §8.

**Link convention (repo standard):** siblings in `technology/` are plain filenames; `ai_llm/` subdir guides are prefixed `ai_llm/`; `banking/` and `management/` guides are prefixed `../banking/` and `../management/`.

---

## How to Use This Guide

Section 1 is the overview — the definition of capacity sizing, the aspect/description table, and the SRE/ITIL frame it sits in. Section 2 is the sizing process — demand forecasting, workload profiling, benchmark-based sizing, rule-of-thumb sizing (flagged), and the process table. Section 3 is the sizing math — Little's Law (verified, 1961), the utilization targets (flagged), peak-vs-average, headroom and growth margin, and the math table. Section 4 is the per-dimension sizing — compute/CPU, memory, storage, network, GPU (cross-referenced), database, queues, app servers, and the dimensions table. Section 5 is the cloud sizing — rightsizing, autoscaling, reserved capacity, and the cloud table. Section 6 is the capacity-management process — the ITIL 4 practice (verified), the monitor→forecast→plan→review loop, and the process table. Section 7 is the banking context — batch windows, payment peaks, regulatory-reporting deadlines (flagged), and the banking table. Section 8 is the worked example — a Cymbal Bank payments-service sizing with the actual math (QPS → cores, Little's Law → queue depth and threads, storage → IOPS) and the lessons. Section 9 is the one-page summary — the final word is **the sized for the peak**. The glossary, the verification ledger, and the cross-references recap close the file.

---

## Table of Contents

1. [The Overview](#1-the-overview) — the definition (verified), the overview table, the SRE/ITIL context
2. [The Sizing Process](#2-the-sizing-process) — demand forecasting, workload profiling, benchmark-based sizing, rule-of-thumb sizing (flagged), the process table
3. [The Sizing Math](#3-the-sizing-math) — Little's Law (verified — 1961), the utilization targets (flagged), peak-vs-average, headroom + growth margin, the math table
4. [The Per-Dimension Sizing](#4-the-per-dimension-sizing) — compute/CPU, memory, storage, network, GPU, database, queues, app servers, the dimensions table
5. [The Cloud Sizing](#5-the-cloud-sizing) — rightsizing (cross-ref), autoscaling, reserved capacity, the cloud table
6. [The Capacity-Management Process](#6-the-capacity-management-process) — the ITIL 4 practice (verified), the monitor→forecast→plan→review loop, the process table
7. [The Banking Context](#7-the-banking-context) — batch windows, payment peaks, regulatory-reporting deadlines (flagged), the banking table
8. [The Worked Example](#8-the-worked-example) — a Cymbal Bank payments-service capacity-sizing design with the actual math
9. [The Summary — One Page](#9-the-summary--one-page) — the final word: **the sized for the peak**
10. [Glossary](#10-glossary)
11. [The Verification Ledger](#11-the-verification-ledger)
12. [Cross-References Recap](#12-cross-references-recap)

---

## 1. The Overview

### 1.1 The Definition — What Capacity Sizing Is (verified)

**Capacity sizing is the disciplined estimation of the compute, memory, storage, network, and service resources a system must be provisioned with to meet its agreed performance targets under its forecast demand — including the peak, the growth, and the failure modes.** It is the engineering answer to the procurement question "how big do we buy it?", and it is the *input* to the ongoing management question "how big does it stay?" (the ITIL capacity-and-performance practice of §6) and the cost question "how big is it worth?" (the FinOps/TCO twins). ✅-structural: the definition is the standard industry understanding, consistent across the AWS Well-Architected performance-efficiency material (verified this pass — the PE pillar's capacity-planning guidance: evaluate capacity needs, match capacity with demand, use elasticity where possible) and the capacity-planning canon; the synthesis is this guide's.

Three distinctions keep the discipline honest, and the repo's own cluster draws them cleanly:

1. **Capacity sizing vs capacity planning.** Sizing is the *point-in-time estimation act* — the numbers you put on a design before you buy or build. Planning is the *rolling activity* — the quarterly/monthly process of forecasting demand, tracking utilization, and deciding when to buy more (the SRE workbook's capacity planning is this: a recurring, risk-managed activity, not a one-off estimate — ⚠-structural, from the SRE canon; the workbook's *Managing Load* chapter ✅ verified on sre.google this pass covers the traffic-management half). Sizing feeds planning: every capacity plan is a sequence of sizing decisions.
2. **Capacity sizing vs capacity management.** Management is the ITIL 4 practice (✅ §6.1): the *organizational capability* that ensures service performance meets agreed current and future requirements. Sizing is the *method* inside that practice — the estimation, profiling, and benchmarking work the practice runs on a schedule.
3. **Capacity sizing vs performance tuning.** Tuning makes the existing footprint faster; sizing decides the footprint. The two meet in the benchmark: you cannot size honestly until you know what one unit of hardware does with your workload — which is exactly the workload-profiling and benchmark work of §2.2–§2.3.

The sizing discipline's core claim, stated once and proved by the rest of this guide: **capacity is not a guess — it is a derived quantity.** Given a workload (arrival rate, service time, growth), a target (latency, utilization), and a failure mode (N+1, peak), the size of every dimension is arithmetic (§3). The parts that are not arithmetic — the forecast, the benchmark, the judgement — are made explicit and flagged, so the arithmetic is never silently polluted by opinion.

### 1.2 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **The definition** | The disciplined estimation of resources (compute, memory, storage, network, services) against forecast demand, performance targets, and failure modes (§1.1) |
| **The process** | Demand forecasting → workload profiling → benchmark-based sizing (with rule-of-thumb as the flagged shortcut) → review (§2) |
| **The math** | Little's Law (L = λW, ✅ 1961), the M/M/1 utilization curve (ρ = λ/μ, ✅), peak-vs-average, headroom and growth margin (§3) |
| **The dimensions** | Compute/CPU, memory, storage (IOPS + capacity), network (bandwidth), GPU, database (connections/QPS/IOPS), queues, app servers (threads/heap) (§4) |
| **The cloud** | Rightsizing, autoscaling, reserved capacity — the elasticity layer that changes the *when* of sizing (§5) |
| **The process home** | The ITIL 4 capacity-and-performance management practice (✅ one of the 34 practices / 17 service-management), run as the monitor→forecast→plan→review loop (§6) |
| **The banking reality** | Batch windows, payment peaks, regulatory-reporting deadlines — the moments the sizing must survive (§7) |
| **The worked example** | A Cymbal Bank payments-service sizing: 1,000 TPS peak → 15 cores, Little's Law → the queue depth and the thread pools, storage → 6,000 IOPS (§8) |
| **The SRE/ITIL frame** | SRE: "100% is the wrong reliability target" (✅ cross-ref zero_downtime §1.2) — so capacity targets are budgeted, not maximal; ITIL: the practice that makes sizing a recurring, evidenced activity (§6) |
| **The final word** | **The sized for the peak** — size to the peak, not the average; everything else follows (§9) |

### 1.3 The SRE/ITIL Context — Where the Discipline Lives

Capacity sizing sits at the intersection of two management traditions, and an architect should be able to name both:

- **The SRE tradition (risk-budgeted capacity).** The Google SRE canon contributes the *target* philosophy: reliability is a budget, not a maximum — "100% is the wrong reliability target" because the last fraction of availability costs super-linearly while users cannot perceive it (✅ cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §1.2, whose ledger verified the SRE-book framing). Capacity gets the same treatment: you size to a *target utilization* that buys headroom for the spikes you can predict and the ones you cannot — not to "as much as possible" and not to "exactly the average". The SRE Workbook's *Managing Load* chapter (✅ chapter verified on sre.google this pass) and the SRE Book's *Handling Overload* chapter (⚠-structural — the chapter exists in the SRE Book's table of contents, widely documented) are the operational half: what the system does when the sizing is exceeded anyway — load shedding, admission control, graceful degradation. Sizing sets the line; overload handling draws what happens past it.
- **The ITIL tradition (evidenced capacity).** ITIL 4's **capacity and performance management** is one of the 34 management practices — in the 17-practice *service management* category, alongside availability management, service level management, and monitoring and event management (✅ verified this pass via the ITIL reference wiki; the taxonomy matches [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) §2.4's verified ledger). The practice's job — ensuring service performance meets agreed *current and future* performance requirements — is the management wrapper around the sizing act: a sizing that is not revisited is a guess; a sizing that is revisited on a schedule, with measurements, is a practice. That is the monitor→forecast→plan→review loop of §6.

The two traditions answer the same question from opposite ends: SRE asks "what is the *right* amount of capacity for the risk we accept?", ITIL asks "what is the *evidenced* amount of capacity for the agreements we made?". The sizing discipline of this guide is the machinery both questions run on.

### 1.4 The Sizing Mindset — What Capacity Sizing Is Not

Three refusals define the discipline as sharply as its positive content:

- **Sizing is not "the biggest instance we can afford".** Over-provisioning is not a safety strategy — it is a cost decision with its own failure mode: a fleet sized at 10× the peak is a FinOps incident (cross-ref [finops_guide.md](finops_guide.md) §3.2's over-provisioned threshold) and hides the *real* limit (the database, the IOPS, the contention point) until the wrong moment. The sizing answers "how much is *derived*", and the derivation is what makes the over-provision visible as what it is: a choice, not a safety.
- **Sizing is not a spreadsheet's output.** The arithmetic of §3 is checkable and small; the *inputs* — the service times, the IOPS counts, the peak ratios — are measurements and judgements. A sizing that presents only its conclusion (a node count, a disk count) without its inputs and assumptions is not a sizing; it is a guess with a footer. The repo's own evidence convention — every number sourced or labelled, every assumption flagged — is this guide's rule too (the §8 figures are illustrative *with the arithmetic shown*).
- **Sizing is not a one-time deliverable.** A sizing done at design time and never revisited is obsolete by the first month-end it survives (§7.2's peaks) or fails. The ITIL practice of §6 exists because capacity is a *rate of change* — the forecast moves, the workload drifts, the estate grows — and the sizing must be re-run on the cadence, not re-written on the incident.

### 1.5 The Sizing Disciplines That Surround It

The sizing act sits in a small ecosystem of adjacent disciplines this guide deliberately does *not* absorb, each owned by a sibling: **availability engineering** (the nines, the failover — [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md)) decides how the sized capacity stays up; **performance engineering** (profiling, tuning — the low-latency guide's latency anatomy, [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §2) decides how fast the sized capacity runs; **cost engineering** (FinOps, TCO — the twins) decides what the sized capacity costs; and **the system-design estimation craft** (the three SDI guides' back-of-envelope) is sizing's scoping form. This guide's boundary rule: **sizing produces the numbers — how much, of what, for which peak — and the siblings consume them.** An architect who keeps that boundary clean never has to re-derive a sibling's discipline, only to cross-reference it.

---

## 2. The Sizing Process

The sizing process is five steps, of which the first three are the estimation machinery and the last two are the discipline that keeps it honest:

1. **Demand forecasting** — what will the workload be? (arrival rates, growth)
2. **Workload profiling** — what does one unit of demand *cost*? (service times, resource consumption per request)
3. **Sizing method** — benchmark-based (measured) or rule-of-thumb (flagged shortcut), per dimension
4. **Headroom and growth margin applied** — the arithmetic of §3.4
5. **Review and iteration** — benchmark vs reality, quarterly recalibration (§6's loop)

### 2.1 Demand Forecasting (verified in method)

**Demand forecasting is the estimation of the future workload the system must serve — volumes, arrival rates, peaks, and growth — from historical data, business plans, and explicitly stated assumptions.** ✅-structural: the *methods* below are the standard forecasting taxonomy (the time-series/regression/judgment split is universal in the forecasting literature; the SRE canon's capacity-planning practice uses exactly this mix — historical trend extrapolation plus planned-product changes — ⚠-structural on the exact chapter wording since the SRE workbook's capacity-planning content could not be live-extracted this pass; its *Managing Load* chapter was ✅). The banking-specific inputs are cross-ref'd in §7.

The forecasting methods, in order of reliability:

| Method | What it does | When it is the right tool | Notes |
|---|---|---|---|
| **Historical extrapolation** | Fit trend/seasonality to measured traffic (daily, weekly, monthly, yearly cycles) | The workload has a stable history and no step-change coming | The foundation of every forecast; banking traffic has strong seasonality (month-end, paydays, 1 Jan — §7.2) |
| **Business-plan-driven** | Add the volume from planned products, campaigns, M&A, new client onboarding | A launch or change is announced | The forecast is only as good as the business case behind it (cross-ref [../management/business_case_development_guide.md](../management/business_case_development_guide.md)) |
| **Analogue / industry benchmark** | Borrow the shape and ratios from a comparable system | Greenfield, no history | The system-design-interview guides' back-of-envelope is this method at interview scale (cross-ref the three SDI guides) |
| **Judgment / scenario** | Expert estimates, scenario planning (base / stress / extreme) | History is unreliable or the future is discontinuous | The honest label for "we don't know"; bank stress-testing culture makes this normal (cross-ref ERM guide §8) |

The three outputs a sizing needs from the forecast: (1) **the peak arrival rate** (transactions/second at the worst hour of the worst day), (2) **the average arrival rate** (for capacity economics and the peak/average ratio), and (3) **the growth curve** (the CAGR or step-changes the sizing's growth margin must absorb). A forecast that does not state its peak explicitly has not finished its job — §3.3 exists because average-only forecasts are the most common sizing failure in the industry.

### 2.2 Workload Profiling (verified in method)

**Workload profiling is the measurement of what one unit of demand costs the system — the service times, resource consumptions, and contention points that turn "requests per second" into "cores, gigabytes, IOPS, and Mbps".** ✅-structural: profiling is the standard performance-engineering practice (load-testing and profiling methodology is universal — the measurement of per-request CPU, memory, I/O, and network against increasing concurrency); the specific metrics below are the industry-standard set. The repo's own performance canon — [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) (TTFT/TPOT as the profiling metrics of the GPU dimension, §4.5 below) — is profiling wearing its latency clothes.

The profile captures, per transaction class:

- **Service time (CPU)** — the p50/p95/p99 CPU-seconds per request, measured under load, *with* the platform overhead (GC, framework, middleware). This is the number that becomes cores in §4.1.
- **Memory footprint** — the live-set bytes per instance and per in-flight request (heap, caches, buffers). This is the number that becomes heap in §4.8.
- **I/O profile** — reads/writes per transaction, block sizes, IOPS demand, and the cache hit rate. This becomes storage sizing in §4.3.
- **Network profile** — bytes in/out per request, plus fan-out (requests to DB, cache, downstream APIs per inbound request). This becomes bandwidth in §4.4.
- **Contention points** — locks, DB connection limits, thread pools, and the saturation behavior past the knee (the M/M/1 curve of §3.2: where latency starts climbing non-linearly).

The profiling instrument is the **load test**: drive the workload class at increasing concurrency against a reference instance, record the throughput/latency curve, and extract the per-unit costs. The output is the **workload profile document** — the single most valuable artefact the sizing produces, because every dimension's math in §4 consumes it, and every future re-sizing (§6's loop) re-runs it.

The profiling checklist — the ten lines the profile document must contain before the sizing starts (⚠-structural — the checklist is this guide's synthesis of the standard profiling practice):

1. The transaction classes (payment instruction, balance lookup, statement, admin — each profiled separately, because each has a different s, I/O count, and W).
2. Per class: **service time s** (p50/p95/p99 CPU-seconds) and the **in-flight time W** (end-to-end with downstream calls) under load.
3. Per class: **I/O count** (reads/writes/index/log per transaction — the §4.3 input) and the **cache hit rate** assumption.
4. Per class: **bytes in/out** and the **fan-out** (downstream calls per inbound request — the §4.4 input).
5. The **memory working set** per in-flight request and the steady-state live set (§4.2).
6. The **concurrency sweep**: throughput and latency at ρ = 0.3, 0.5, 0.7, 0.9 on the reference node — the measured version of the §3.2 curve.
7. The **bottleneck identification**: which resource saturates first (CPU, pool, IOPS, NIC) — the answer predicts the estate's real ceiling.
8. The **configuration record**: instance type, runtime flags, tuning — a profile without its configuration is unrepeatable (§2.3's rule).
9. The **degradation behaviour**: what happens past the knee — queueing, timeouts, retries — the overload-handling input (cross-ref the SRE workbook's overload chapter, §1.3).
10. The **date and the test conditions** — so the §6 review loop knows when the profile went stale.

### 2.3 Benchmark-Based Sizing (verified)

**Benchmark-based sizing is the sizing method that uses measured performance — either your own load tests (the §2.2 profile) or published, standardized benchmarks — to derive the required hardware, rather than guessing.** ✅-structural on the method (measure-then-derive is the sizing canon's core practice, consistent with the AWS Well-Architected performance-efficiency guidance verified this pass); the benchmark facts below are ✅-verified individually.

The two benchmark families an architect must know:

1. **Your own load tests** — the gold standard: your workload, your code, your hardware. A load test on a reference instance (e.g., "a 4-vCPU/8-GB node sustains 350 TPS of the payment-initiation mix at p95 < 200 ms") is the most defensible sizing input that exists, because it is the actual workload. Everything in §8's worked example runs on this.
2. **Published industry benchmarks** — standardized, comparable, but workload-approximate. The canonical OLTP benchmark is **TPC-C** (✅ verified on tpc.org this pass): approved **July 1992**, an on-line transaction processing benchmark with **five concurrent transaction types** and a **nine-table database**, measured in **transactions per minute (tpmC)**. TPC-C's value to a sizer is the *comparability* it provides — a vendor's "X tpmC on Y hardware" is a reproducible reference point — and its honest limitation is the workload gap: TPC-C portrays a wholesale supplier's order-entry mix, not your payment flow (✅ tpc.org's own description: "represents any industry that must manage, sell, or distribute a product or service" — i.e., approximate). The repo's DB guides (oracle_database_guide.md, nosql_data_modelling_guide.md) carry the database-specific capacity discussion; this guide's §4.6 applies it.

The benchmark discipline has three rules: **benchmark the actual workload** (or admit the proxy), **record the configuration** (a benchmark without the hardware/software/tuning context is a number with no meaning), and **benchmark the peak, not the demo** (a 10-minute run at 30% utilization proves nothing about the 2-hour peak at 90% — §3.3).

### 2.4 Rule-of-Thumb Sizing (verified — flagged ⚠)

**Rule-of-thumb sizing is the shortcut method: per-user, per-transaction, or per-metric unit-cost estimates ("one core per 100 TPS", "4 GB per million users", "1 Gbps per 10,000 concurrent users") used when no measurement exists.** The rules exist — every experienced architect carries a set — and they are useful for *scoping* (the SDI guides' back-of-envelope is precisely this, and it is the right tool for a design conversation). But the *specific numbers* are the problem: they are **⚠-flagged this pass** because no authoritative primary source publishes a canonical table of them — they are folklore, vendor guidance, and personal experience, varying by workload class by an order of magnitude (a 100-TPS-per-core rule is right for one workload and 10× wrong for another — a payment switch at >100 tps per gateway instance, ✅ NETS §2.4, vs a JSON-transforming API service at 1,000+).

The honest use of rules of thumb: (1) **scope only** — get within a factor of 2–3 for a budget conversation, then benchmark; (2) **document the source** — "one core per 100 TPS" is a claim with an author, and the author's workload is probably not yours; (3) **apply the safety factor** — a rule-of-thumb sizing without a benchmark behind it deserves *more* headroom, not less, because its error bars are wide (the headroom math of §3.4 exists to absorb exactly this uncertainty); and (4) **never ship rule-of-thumb as the final number for a tier-1 system** — a bank's payment path sized on folklore is a regulatory incident waiting for a peak (§7.2). The verification ledger records this flag: **rule-of-thumb sizing: ⚠ — the method is real and useful for scoping; the specific coefficients are unverifiable folklore and are flagged accordingly.**

### 2.5 The Process Table

| Step | Method | Input | Output | Verification |
|---|---|---|---|---|
| **1. Demand forecasting** | Historical extrapolation + business-plan deltas + scenario | Traffic history, product roadmap, business case | Peak rate, average rate, growth curve | ✅-structural (methods are the standard taxonomy) |
| **2. Workload profiling** | Load testing on a reference instance; per-transaction-class measurement | The workload, a reference node, the targets | Service times, memory/I/O/network per request, contention map | ✅-structural (standard performance-engineering practice) |
| **3. Sizing method** | Benchmark-based (own load tests; TPC-class published benchmarks) or rule-of-thumb (⚠ scoping only) | The §2.2 profile | Per-dimension resource demand | ✅ TPC-C facts (tpc.org); ⚠ rule-of-thumb coefficients |
| **4. Headroom + growth margin** | The §3.4 arithmetic: design peak = peak × (1+growth) × (1+headroom) | Peak rate, CAGR, N+1 policy | The provisioned target per dimension | ✅-structural (the arithmetic is shown in §3.4) |
| **5. Review and iterate** | The §6 loop: monitor actuals vs forecast, recalibrate quarterly | Utilization telemetry, real volumes | Updated sizing, capacity plan entries | ✅ ITIL 4 practice (verified); the loop is §6.2 |

### 2.6 The Common Sizing Failure Modes

The discipline is best learned from its failure modes — each one named, with the § that prevents it. Every failure below has been observed in production estates, and every one is a *method* failure, not a bad-luck failure:

| Failure mode | The symptom | The prevention |
|---|---|---|
| **Average-sizing** | The system dies at the peak it "should have handled" | §3.3: size to the sustained peak; the peak/average ratio is a design input |
| **Folklore-sizing** | The cores/IOPS/GB numbers came from "what everyone uses" — an order of magnitude off the workload | §2.4: rule-of-thumb is scoping-only; benchmark or flag |
| **Single-dimension-sizing** | Cores are perfect; the DB connection pool or the disk IOPS is the actual ceiling | §4.9: every dimension sized; the node count is the *max* of the dimension counts |
| **No-N+1-sizing** | One node lost → the fleet drops below the design peak at the worst moment | §3.4/§4.1: the N+1 check is a second sizing, done explicitly |
| **Static-sizing** | The design-time numbers are quoted three years later, through 40% growth | §6.2: the monitor→forecast→plan→review loop re-derives on the cadence |
| **Unlabelled-sizing** | The numbers cannot be traced to a measurement, a benchmark, or an assumption | §1.4: every input sourced or flagged; the arithmetic shown |

---

## 3. The Sizing Math

The math is where sizing stops being opinion. Three verified results do most of the work, plus one arithmetic pattern (headroom × growth) that is pure convention. The M/M/1 utilization curve then explains *why* the targets exist.

### 3.1 Little's Law (verified — the 1961 theorem)

**Little's Law: the long-term average number of customers (requests) in a stationary system, L, equals the average arrival rate, λ, multiplied by the average time each customer spends in the system, W — L = λW.** ✅ Verified this pass via the queueing-theory reference material: the theorem is named for **John D.C. Little**, who **published his proof in 1961** (the law's form was in use before — assumed without proof in a 1954 paper, and published in the L=λW form by Morse, who challenged readers to find a counterexample — and Little's 1961 proof showed none exists). The relationship is remarkable precisely because it holds **regardless of the arrival-process distribution, the service-time distribution, and the scheduling discipline** — only ergodicity (long-run stability) is required (✅ same source).

The law's power for sizing is that it converts *measured* quantities into *design* quantities:

- **Response time from occupancy and throughput:** if the average number of jobs in the system is L and the throughput is λ, the average response time is W = L/λ. The reference example: an average queue depth of 9 plus 1 in service (L = 10) at 50 jobs/second gives mean response time 10/50 = 0.2 s (✅ the worked example in the Little's Law reference).
- **Concurrency (in-flight requests) from rate and latency:** the number of requests *in flight* at a server or database is λ × W. A service doing 1,000 TPS with a 200 ms downstream call keeps 1,000 × 0.2 = **200 requests in flight** — which is precisely the connection-pool and thread-count sizing of §4.6/§4.8. This is the law's most-used sizing form.
- **Queue depth from rate and tolerated backlog:** a queue absorbing 1,000 TPS with a tolerated 2-second backlog holds 2,000 messages — the §4.7 queue-capacity number.

The one constraint the law imposes on every sizing: **a stationary system requires the service capacity μ to exceed the arrival rate λ (ρ = λ/μ < 1).** If arrivals outrun service on average, the queue grows without bound — no amount of tuning fixes an undersized system, only more μ (more cores, more connections, more brokers).

### 3.2 The Utilization Targets (verified math — the percentage flagged ⚠)

**The relationship between utilization and latency is the M/M/1 result: for a single-server queue with arrival rate λ and service rate μ, the utilization is ρ = λ/μ, stability requires ρ < 1, and the mean number in the system is L = ρ/(1−ρ); by Little's Law the mean response time is W = 1/(μ(1−ρ)) = 1/(μ − λ).** ✅ Verified this pass: the M/M/1 reference material states ρ = λ/μ with stability ρ < 1 and the mean number in the system ρ/(1−ρ) explicitly; W = 1/(μ(1−ρ)) follows immediately from Little's Law (L = ρ/(1−ρ), W = L/λ, λ = ρμ).

The consequence is the **hockey-stick curve** every capacity discussion is really about: at 50% utilization the response time is only 2× the service time; at 80% it is 5×; at 90% it is 10×; at 95% it is 20× — and beyond 100% it is undefined, because the queue grows forever. The knee of the curve is somewhere around 60–80% utilization, and that is the entire mathematical content of the utilization-target guidance. The numbers, from W = 1/(μ(1−ρ)) with service time normalized to 1:

| Utilization ρ | Mean response time (× service time) | Reading |
|---|---|---|
| 0.50 | 2.0× | Comfortable; queueing barely visible |
| 0.60 | 2.5× | The curve starts to bend |
| 0.70 | 3.3× | The common design target's neighbourhood |
| 0.80 | 5.0× | The knee — the FinOps upsize trigger's neighbour (cross-ref finops §3.2) |
| 0.90 | 10.0× | The vertical section — latency pain, queue growth |
| 0.95 | 20.0× | Saturation; one arrival-rate wiggle from instability |
| 1.00 | undefined | ρ ≥ 1: no stationary state — the queue grows without bound (§3.1) |

**The ~70–80% target itself is flagged ⚠:** the *curve* is verified math, but the *specific "keep CPU at 70–80%" guidance* is practice folklore — widely cited by virtualization, capacity-planning, and FinOps material (including the repo's own [finops_guide.md](finops_guide.md) §3.2, whose <40% downsize / >80% upsize thresholds are that guide's verified-in-its-own-ledger practice numbers), but not pinned this pass to a single authoritative primary source. It is a reasonable target *because* the verified curve says response-time pain accelerates past ~70–80% — and because it leaves 20–30% for the arrival-rate variance that real workloads exhibit (the peak-vs-average gap of §3.3) — but the exact band is convention, not theorem. An architect should say "we target 70% CPU under design peak because the M/M/1 curve bends there and it leaves failover headroom", not "the rule says 70%".

Two honest refinements to the target: (1) **the target is per-resource, not per-system** — a CPU target of 70% says nothing about the DB connection pool, the disk IOPS, or the network link; each dimension gets its own target (a disk should usually sit *lower* than CPU, because IOPS spikes are spikier; a memory allocation target is about GC pressure, not utilization); (2) **the target is for the peak, not the average** — sizing to 70% of the *average* load means 95%+ at the peak, which is the hockey stick's vertical section (the single most common sizing error; §3.3 and §8's lesson 1).

### 3.3 Peak-vs-Average (verified in method)

**Size to the peak, not the average — and know the ratio between them.** ✅-structural: the peak-vs-average discipline is universal in the capacity literature (the NETS guide's "throughput is a peak problem" — ✅ cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §2.4, with Visa averaging ~8,500 tps but peaking above 65,000 tps, a ~7.7× ratio — is the industry's own statement of it; the queueing math of §3.1–§3.2 is why).

The mechanics: a workload's average rate hides its shape. A service averaging 200 TPS over the day may run at 50 TPS overnight and 600 TPS in the lunch-hour peak — a 3× peak/average ratio. Because response time grows as 1/(1−ρ), the system must be sized so that ρ stays under target at the *peak* rate, not the average. The peak/average ratio is itself a design input: banking retail flows run 2–5× (month-end, paydays — §7.2); the NETS-style national-switch example runs ~7.7× on the Visa figures; a marketing-driven consumer service can run 10×+.

The two errors to name and avoid: **sizing to the average** (the peak lands on the hockey stick's vertical section — p95 latency explodes, queueing begins, the §8 queue-depth math runs away) and **sizing to a single "peak" without defining its duration** (a 1-second spike needs burst capacity or queueing discipline — the low-latency guide's admission control, cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §7.2 — while a 2-hour plateau needs real capacity). The sizing input is therefore the *sustained peak* — the worst hour (or worst 30-minute window for payment cut-offs, §7.1) of the worst day.

### 3.4 Headroom and the Growth Margin (verified in arithmetic)

**Headroom is the unallocated capacity kept for the unpredictable — the failover case, the traffic surprise, the forecast error. The growth margin is the capacity reserved for the planned future — the forecast's own growth curve. The sizing formula is: design peak = measured/forecast peak × (1 + growth margin) × (1 + headroom), and every dimension is provisioned for the design peak, with utilization targets applied on top.** ✅-structural: the *pattern* is universal capacity-planning practice (the SRE canon's capacity planning treats headroom as the buffer against forecast uncertainty — ⚠-structural on the exact chapter text, the SRE workbook's capacity-planning chapter could not be live-extracted this pass; the *Managing Load* and *Identifying and Recovering from Overload* chapters were ✅ and cover the operational half); the arithmetic is shown below so it is checkable.

Conventions worth naming (each is a policy decision, not a law):

- **Growth margin** typically reflects the 12–36-month forecast: a 3-year CAGR of ~9% compounds to ~30% (1.09³ ≈ 1.295) — the common "size for +30% growth" convention is this arithmetic wearing a suit (⚠ — the specific percentages are practice convention; the compounding arithmetic is exact).
- **Headroom** typically carries the failure mode: N+1 (one node can be lost without losing capacity) is a 1/(n−1) adder; a 25% adder is the common N+1 approximation for a 5-node tier; a 100% adder (2×) is the traditional "active/standby" or "dual-site" posture. Cloud changes the game (§5): autoscaling turns *steady-state* headroom into *reactive* headroom — you keep less standing, but the scaling must be faster than the peak's ramp (the §5.2 predictive-scaling note).
- **The two multiply, they do not add:** a 30% growth margin and a 25% headroom give 1.3 × 1.25 = 1.625× the measured peak — not 1.55×. The worked example in §8 uses exactly this arithmetic.

### 3.5 The Math Table

| Result | Formula | Status | Sizing use |
|---|---|---|---|
| **Little's Law** | L = λW | ✅ 1961 theorem (Little's proof, 1961) | Response time from occupancy; in-flight requests; queue depth |
| **Utilization** | ρ = λ/μ; stable iff ρ < 1 | ✅ M/M/1 result | The fundamental stability constraint on every dimension |
| **Mean number in system** | L = ρ/(1−ρ) | ✅ M/M/1 result | Queue-depth expectation at a given utilization |
| **Mean response time** | W = 1/(μ(1−ρ)) | ✅ via Little's Law (derived) | The hockey-stick: latency vs utilization |
| **The ~70–80% target** | Target ρ under design peak ≈ 0.7–0.8 | ⚠ percentage flagged (the curve is ✅) | The per-dimension utilization target convention |
| **Peak-vs-average** | Size to sustained peak; know the ratio | ✅-structural (universal practice; NETS §2.4 figures ✅) | The design input: the worst hour, not the mean |
| **Design peak** | peak × (1+growth) × (1+headroom) | ✅-structural (arithmetic shown) | The number every dimension is provisioned against |

### 3.6 The Sizing Identity — the Formulas at a Glance

The whole of §3, in the five equations an architect should be able to write from memory — each derived or verified above, each used in §8:

1. **Little's Law** — L = λW (✅ 1961): in-flight work = rate × time. The skeleton of the sizing (threads, connections, queue depth, memory's in-flight term).
2. **The stability constraint** — ρ = λ/μ < 1 (✅): arrivals must lose to service capacity, or the queue grows without bound. The sizing's first check on every dimension.
3. **The latency curve** — W = 1/(μ(1−ρ)) (✅ derived): response time = service time ÷ spare capacity. Why the hockey stick exists and why the targets sit where they do.
4. **The design peak** — λ_design = λ_peak × (1+growth) × (1+headroom) (✅-structural, arithmetic shown): the future and the failure mode, multiplied onto the peak.
5. **The dimension identities** — cores = λ_design·s/ρ_target (§4.1); threads = λ·W_blocking (§4.8); connections = λ·W_DB (§4.6); IOPS = λ_design × I/O-per-tx (§4.3); Mbps = Σ(bytes × rate) (§4.4); queue depth = λ × tolerated delay (§4.7). Six identities, one law: **everything is λ times something, divided by a target, plus the margins.**

An architect who can write these five lines and name the status of each (three verified theorems, one convention with its arithmetic shown, one family of identities) can size any system honestly — and can defend the sizing in the review, because every number traces to a line.

---

## 4. The Per-Dimension Sizing

The dimensions table is the §3 math applied dimension by dimension. Each subsection names the method, shows the arithmetic pattern, and flags what is convention. The sizing inputs (λ, W, bytes, IOPS-per-transaction) all come from the §2.2 workload profile; the targets come from §3.2–§3.4.

### 4.1 Compute / CPU (verified in method — the core counts)

**Method: cores = design-peak rate ÷ per-core sustainable rate at the utilization target.** The per-core rate comes from the profile: if a reference node's load test shows service time s seconds of CPU per request (p50, including overhead), one core sustains 1/s requests/sec at 100% — and target_ρ × 1/s at the target utilization. Formally:

`cores = λ_design / (target_ρ / s)`  where λ_design = λ_peak × (1+growth) × (1+headroom), and s = p50 CPU-seconds per request.

The arithmetic example (the §8 numbers): s = 4 ms → 250 req/s/core at 100%, 175 at the 70% target; λ_design = 1,625 TPS → 1,625/175 = 9.3 → **10 cores minimum, 15 provisioned across 3 × 5-vCPU nodes** so that losing one node leaves 10 cores (1,750 TPS ≥ 1,625 — the N+1 check, §3.4). ✅-structural: the cores-from-service-time pattern is standard capacity engineering (it is the direct application of the verified §3.1–§3.2 math — per-request CPU seconds and the ρ = λ/μ constraint; the specific s-values are always workload-measured, never assumed).

The caveats that keep the count honest: **cores are not all equal** (a vCPU is a scheduling slice; the benchmark must run on the *target* instance family — the AWS PE pillar's compute-and-hardware focus area, ✅-structural, says exactly this: match the hardware choice to the workload); **saturation is not linear across cores** (lock contention and the GC stop-the-world raise the effective s as cores grow — profile at the target core count); and **burst vs sustained** (a bursty tier can overcommit via cloud burst credits; a payment switch cannot — §7.2).

### 4.2 Memory (verified in method)

**Method: memory = live set + per-request working set × in-flight requests (Little's Law) + OS/GC overhead, sized so the runtime operates below its pressure threshold.** The Little's Law term is the one most sizings forget: at λ = 1,000 TPS with W = 200 ms in-flight time, the service holds 200 requests' worth of transient state — at, say, 8 KB of buffers and objects per request, that is 1.6 MB of transient memory *on top of* the live set; the number becomes large only with heavy per-request payloads (document processing, GenAI contexts — cross-ref the low-latency guide's KV-cache sizing, §4.5 below).

The Java/GC shape is the classic case: **heap ≥ 4× the live set** is the conventional working rule so the generational collector has room to operate without thrashing (⚠ — the 4× is practice convention, not theorem; the *mechanism* — GC pause frequency rising as the heap fills toward the live set — is verified runtime behavior, ⚠-structural). The memory dimension is also where the *caching* decision lives: a 90% cache hit rate at 1 KB per entry is 9 KB per 10 requests of *saved* downstream I/O — the §4.3 IOPS and §4.6 QPS numbers drop when the cache is sized honestly (and the cache's own memory must be in the §4.2 total).

### 4.3 Storage (verified — the IOPS/capacity split)

**Method: two independent sizes — IOPS (performance: the peak I/O rate the workload generates) and capacity (the bytes the retention policy requires). The provisioned volume must satisfy both, and they are rarely satisfied by the same purchase.** ✅-structural on the split (capacity vs performance provisioning is standard storage engineering; the IOPS-per-transaction derivation below is the §3 arithmetic applied to I/O). The two arithmetic patterns:

- **IOPS = λ_design × I/O-per-transaction:** a transaction class issuing 4 I/Os (1 read, 1 write, 1 index update, 1 log/commit) at 1,000 TPS demands 4,000 IOPS at the peak — and 4,000 × 1.25 headroom = **5,000 IOPS provisioned**. The IOPS-per-transaction count comes from the §2.2 profile (reads vs writes, hit ratios, batching); it is the single most commonly *underestimated* line in banking sizings because a payment's audit/journal writes multiply the naive count (§8 row 5).
- **Capacity = daily volume × retention days × replication factor:** 15M transactions/day × 2.5 KB stored each ≈ 40 GB/day ≈ 14.6 TB/year — and a 7-year regulatory retention (cross-ref the Kafka guide's tier-1 "7 years ⚠" retention, [../banking/kafka_guide.md](../banking/kafka_guide.md) §10.4) with 2× replication and 20% index overhead is 14.6 × 7 × 2 × 1.2 ≈ **245 TB** — which is why the tiering decision (hot vs archive) is a storage-sizing decision, not a storage-ops decision: keep 2 years hot, 5 years archived, and the hot tier's cost drops 3–4× (cross-ref [tco_modeling_guide.md](tco_modeling_guide.md) and [finops_guide.md](finops_guide.md) §5 for the cost side).

The two constraints interact: a volume sized for capacity at 3 IOPS/GB (the gp2-style ratio — ⚠ instance-family specific, the *ratio pattern* is verified storage practice) may fail the IOPS test (245 TB at 3 IOPS/GB ≈ 735 IOPS — far below the 5,000 needed), which is why high-IOPS/low-capacity tiers and high-capacity/low-IOPS tiers are separate products. The DB guide's PDB-pooling note ([oracle_database_guide.md](oracle_database_guide.md) §5 — capacity pooled and shared on Exadata-class platforms) is the consolidated answer at bank scale.

### 4.4 Network (verified — the bandwidth math)

**Method: bandwidth = Σ (per-message bytes × message rate) across all flows, summed at the peak, then multiplied for the failure modes — and the math must be done on the *total* fan-out, not the inbound rate.** The arithmetic: an inbound payment instruction of 2 KB that triggers a 1 KB DB round-trip, a 1 KB cache check, a 3 KB downstream authorization call, and a 1 KB response is 8 KB of *internal* traffic per inbound request — at 1,000 TPS, 8 MB/s ≈ 64 Mbps of east-west traffic *before* the 3× replication/aggregation multiplier of a 3-node tier. ✅-structural: the bytes-times-rate summation is the standard bandwidth-engineering formula; the fan-out accounting (inbound rate × amplification factor) is the standard networking practice — and it is the exact math the low-latency guide applies to the GenAI tier (a 20 KB prompt + 5 KB response per request at N requests/s is a bandwidth line item before it is a latency one, cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §8.3).

The bandwidth sizing's three honest notes: **NIC provisioning is not bandwidth provisioning** (a 10 Gbps NIC is the floor, not the plan — the plan is the summed flows above); **the WAN/API edge is sized separately from the east-west fabric** (client-facing ingress/egress at the payment peak vs the internal fan-out at the same peak — both use §3.3's peak, but different bytes); and **the latency budget and the bandwidth budget are the same equation** (a 10 ms DC-internal hop — cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) — is a latency number *and* a bandwidth number: the bytes per second over the hop's link must fit the link at the peak, or queueing at the NIC is a latency failure before the application ever sees the request).

### 4.5 GPU (verified — cross-referenced, not re-derived)

**The GPU dimension is owned by [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) — this subsection cross-refs it and stops.** The sizing-relevant facts from that guide's verified ledger: GPU capacity is dominated by **KV-cache memory** (the vLLM/PagedAttention insight — "the KV cache memory for each request is huge and grows and shrinks dynamically", ✅ that guide §3.3), not raw FLOPs; **throughput and latency are the same resource** (continuous batching means batch size and per-request latency fight over GPU memory — ✅ §3.1); and the hardware backdrop is the H100/B200 class (✅ §4.7). The sizing pattern the architect takes away: **GPU count = f(concurrent in-flight requests × context size, target TTFT/TPOT, batch policy)** — the same Little's Law shape as §4.1 (L = λW in-flight requests) with the per-request working set measured in KV-cache tokens instead of heap bytes, and the utilization target set by the latency budget (TPOT degrades as batch grows — ✅ §3.2) rather than a CPU-style 70% line. For a bank's copilot tier, cross-ref that guide's §9 worked example for the concrete instance counts; for the cost side, cross-ref [finops_guide.md](finops_guide.md) §9's GPU-spend notes. ⚠-structural: the *numbers* of instances are workload- and model-dependent by an order of magnitude; the *shapes* above are that guide's verified content.

### 4.6 Database (verified — connections/QPS/IOPS)

**Method: three independent sizes — connections (from Little's Law: pool ≥ λ × W_downstream + margin), QPS/CPU (from the profile's query mix), and IOPS (from §4.3's I/O count) — plus the capacity plan of §4.3, all validated against the engine's own limits.** ✅-structural: connection-pool sizing from in-flight concurrency is the standard database-capacity practice (it is the §3.1 law applied to the DB client); the engine-specific limits (PostgreSQL `max_connections`, Oracle's process/connection model, the PDB-pooled capacity of [oracle_database_guide.md](oracle_database_guide.md) §5) are cross-referenced, not re-derived.

The three arithmetic patterns:

- **Connections:** a 1,000-TPS workload with a 200 ms average DB call holds 200 in-flight connections (Little's Law); with 3 app nodes, per-node pools of 100–150 (200/3 × 1.5–2× margin) and a DB-side `max_connections` of 400+ (pooled total × 1.3 for admin/ops connections) — and a hard ceiling rule: **the DB must never be the connection bottleneck under failover** (one node lost, two nodes' pools × 2 retry spike — the N+1 connection check).
- **QPS/CPU:** the profile's query mix (e.g., 3 queries per payment instruction) at the design peak gives the DB QPS; divided by the per-core query rate from the DB load test (or the TPC-C-class reference, §2.3) gives DB cores. 3,000 QPS ÷ 1,500 QPS/core = 2 cores → provisioned 4–6 with the §3.4 multipliers.
- **IOPS:** §4.3's count, with the database's own amplification (index maintenance, WAL/redo writes, autovacuum/backup bursts) added — the "banking write multiplier" of §8 row 5.

The engine's own saturation signals (connection waits, lock waits, IOPS queue depth, WAL lag) are the §6 monitor loop's DB-specific inputs; the NoSQL side (partition/request-unit math) is cross-ref'd to [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md).

### 4.7 Queues (verified — cross-referenced)

**The queue dimension is owned by [../banking/kafka_guide.md](../banking/kafka_guide.md) — cross-ref its §7.3 (throughput/partitions) and §10.4 (the topic catalog); this subsection supplies the sizing math that guide assumes.** Three numbers size a queue:

- **Depth (Little's Law):** the backlog the queue must absorb = λ × tolerated_delay. 1,000 TPS with a 2-second tolerated recovery backlog = 2,000 messages in flight; with 1 KB messages, 2 MB of buffered state — and the *consumer-side* in-flight count is the same law: a consumer group processing 1,000 TPS at 50 ms per message holds 50 in flight per member.
- **Partitions:** ≥ λ_peak ÷ per-partition sustainable throughput. At 1,000 TPS and ~100 TPS per partition-consumer, 10+ partitions — which is why the Kafka guide's payments topics use **24 partitions** (✅ [../banking/kafka_guide.md](../banking/kafka_guide.md) §10.4: `payments.payment.executed` at 24 partitions) — the bank's own numbers already encode the peak-margin arithmetic.
- **Retention capacity:** λ × bytes × retention (7 years for tier-1 payment facts, ✅ cross-ref) — the §4.3 capacity math applied to the event stream, with the compaction/retention policy deciding hot vs archived (✅ cross-ref the Kafka guide's taxonomy: facts retained, reference data compacted).

The queue is also the *headroom device*: a bounded queue with admission control absorbs the §3.3 burst that would otherwise hit the DB (§7.2's peak, §5.2's autoscaling lag) — cross-ref the low-latency guide's §7.2 bounded-queue pattern. ⚠-structural on the per-partition throughput figure (workload-dependent by design; the *shape* — partitions scale with peak rate ÷ per-partition capacity — is the standard Kafka sizing practice ✅ via that guide's ledger).

### 4.8 App Servers (verified — threads/heap)

**Method: threads from Little's Law (threads ≥ λ × W_blocking for a blocking service; or the async model's in-flight count), heap from §4.2, nodes from the §4.1 core count with the N+1 check.** The blocking-service arithmetic: 333 TPS per node (1/3 of the 1,000 peak) × 200 ms of blocking downstream time = 67 busy threads per node → a **150-thread pool** (67 × 2× margin + admin threads) keeps queueing at the pool edge negligible; the async/event-loop model replaces thread count with in-flight request count and explicit backpressure (cross-ref the low-latency guide's §7 concurrency treatment for the async shape). Heap per node follows §4.2: live set + (in-flight × per-request working set) + runtime overhead, with the 4×-live-set GC convention (⚠) applied.

The app-server dimension is where §4.1–§4.4 *converge*: the node count must satisfy cores (§4.1), the per-node memory must satisfy the heap target (§4.2), the node's NIC must carry the per-node share of the fan-out (§4.4), and the node's connection pools must carry the §4.6 share. The final node count is therefore the *max* of the per-dimension counts — 3 × 5-vCPU/16-GB nodes in §8 satisfy cores (15 ≥ 10), memory (16 GB ≥ the heap need), and N+1 (10 cores after a loss ≥ the 1,625-TPS design peak) simultaneously.

### 4.9 The Dimensions Table — Dimension / Method / Notes

| Dimension | Method (arithmetic) | Notes / flags |
|---|---|---|
| **Compute/CPU** | cores = λ_design × s ÷ target_ρ | s from the profile (p50 CPU-sec/req); target_ρ 0.7–0.8 ⚠; N+1 check on the node count |
| **Memory** | live set + (λ × W × per-request working set) + runtime | Heap ≥ 4× live set ⚠ convention; cache memory counted here, saved IOPS counted in storage |
| **Storage** | IOPS = λ_design × I/O-per-tx; capacity = daily bytes × retention × replication | Two independent sizes; tiering (hot/archive) is a sizing decision; the banking write multiplier (§8) |
| **Network** | Mbps = Σ(bytes × rate) over all flows at the peak | Fan-out accounting; NIC ≠ bandwidth; the same equation as the latency budget |
| **GPU** | Cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §3–§4, §9 | KV-cache-dominated; latency/throughput are the same resource; not re-derived here |
| **Database** | connections = λ × W_DB + margin; QPS → cores; IOPS per §4.3 | Engine limits cross-ref'd (oracle_database_guide.md, nosql_data_modelling_guide.md); never the failover bottleneck |
| **Queues** | depth = λ × tolerated delay; partitions = λ_peak ÷ per-partition rate | Cross-ref [../banking/kafka_guide.md](../banking/kafka_guide.md) §7.3/§10.4; the burst-absorption device |
| **App servers** | threads = λ × W_blocking (× margin); heap per §4.2; nodes = max of dimension counts | The convergence point: node count must satisfy all dimensions at once, with N+1 |

### 4.10 The Cross-Dimension Checks

A sizing that sizes each dimension in isolation has not finished. The §8 worked example's table passes the five checks below, and every real sizing should be run against them explicitly:

1. **The max-not-sum rule:** the node count is the *maximum* of the per-dimension counts (cores → nodes, memory → nodes, network per node → nodes), never the sum — the §8 app tier is 3 nodes because 15 cores needed 3 × 5 vCPU, and the same 3 nodes carry the memory and network shares. Summing dimensions is how sizings over-provision by 2–3× (and then get "rightsized" into a surprise — cross-ref [finops_guide.md](finops_guide.md) §3.2).
2. **The N+1 re-check per dimension:** the failover check is not just cores — losing one node must not push the *connections* (pools × 2), the *network* (the remaining nodes carry the fan-out), or the *memory* (rebalanced state) past their own limits. §8 checks cores (10 ≥ 9.3 ✓) and connections (300 ≥ 200 ✓) explicitly; the others are the same arithmetic.
3. **The cache-IOPS loop:** a cache hit *is* saved IOPS and QPS — the §8 fraud-check cache's 90% hit rate is what keeps the §4.3 IOPS and §4.6 QPS numbers at their stated levels; the sizing must state the hit-rate assumption and re-derive the I/O dimensions when it drifts (the §6 monitor loop's cache-hit-rate metric).
4. **The queue-as-buffer check:** the queue's depth (§4.7) must absorb the *difference* between the arrival peak and the consumer capacity during the autoscaling lag (§5.2) — the §8 queue's 2-second tolerated backlog is exactly the scale-up window. A queue sized only for steady-state flow is a queue that overflows exactly when the fleet is growing.
5. **The latency-budget loop:** every dimension's *size* has a latency consequence (the network hop's bandwidth is a latency number, §4.4), and the latency budget (the < 500 ms p95 of §8) must be re-checked against the sized footprint — the low-latency guide's budget discipline, cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §2.3 and §8.3.

---

## 5. The Cloud Sizing

The cloud changes the *when* of capacity — and therefore the *how much*. The on-prem sizing question is "how big do we buy, for how long?"; the cloud sizing question is "how much do we *hold* standing, and how fast can we *grow* reactively — and what do we commit to, given both?" The elasticity layer of §5.2 is the new variable; the §3 math is unchanged.

### 5.1 The Rightsizing (verified — cross-ref finops_guide.md)

**Rightsizing is the ongoing correction of provisioned capacity toward actual demand — the FinOps twin of the sizing discipline, owned by [finops_guide.md](finops_guide.md).** ✅ via that guide's ledger (§3.2): the FinOps practice uses utilization thresholds — **over-provisioned below ~40% utilization: downsize; under-provisioned above ~80%: upsize; decide on 14+ days of utilization data**. The two guides' relationship is exact: this guide's §3.4 headroom says *provision for the design peak*; FinOps rightsizing says *don't keep provisioning for a peak that has not shown up for 14 days* — the standing allocation is the design-peak number, the *billed* allocation is whatever rightsizing has converged it to, and the gap is the cost of over-provisioning that the FinOps and TCO guides quantify (cross-ref [tco_modeling_guide.md](tco_modeling_guide.md) and [finops_guide.md](finops_guide.md) §9 for the banking cost reality).

The sizing-relevant note: rightsizing thresholds and §3.2's utilization targets are *the same band* from two directions — 70–80% as the design target (⚠ as flagged) and 80% as the upsize trigger — which is exactly why the FinOps guide's <40%/>80% thresholds are the operating counterpart of this guide's math. A cloud estate that rightsizes on the same targets the sizing used is a closed loop; one that rightsizes against a different band is a system slowly drifting from its design assumptions (§6's review loop catches this).

### 5.2 The Autoscaling (verified in method)

**Autoscaling is the mechanism that converts §3.4's standing headroom into *reactive* headroom: the fleet grows and shrinks with measured demand, so the design peak becomes a *ceiling the scaling races toward* rather than a floor the estate sits at.** ✅-structural: autoscaling is the standard cloud capacity mechanism (the repo's FinOps guide documents the modes in its own ledger — ✅ cross-ref [finops_guide.md](finops_guide.md) §7: **target-tracking** (e.g., scale at 60% CPU), **schedule-based** (predictable patterns), **predictive/ML** (forecast-driven); the AWS PE pillar's elasticity guidance — ✅-structural, verified the pillar's existence and focus areas this pass — is the framework home).

The sizing math changes shape with the mode:

- **Target-tracking:** the autoscaler holds utilization at a target (60% CPU is the common default, ⚠ — same flagged-convention status as §3.2's 70–80%), so the *standing* fleet is sized for the *current* peak and the scaler absorbs growth — the §3.4 growth margin shifts from standing capacity to *scaling speed*: the fleet must grow faster than the peak's ramp (a 15-minute scale-up lag vs a 10-minute traffic ramp is a missed peak — the §8 lessons' cloud row).
- **Schedule-based:** banking's predictable peaks (month-end, cut-off hours — §7) are *schedulable* — the scale-up happens before the peak by policy, not by reaction. This is the cloud-native answer to the §7.2 peak problem for the parts of the estate that are forecastable.
- **Predictive:** ML forecast-driven scaling (✅ cross-ref finops §7) pre-positions capacity for the pattern the target-tracking would lag — the §2.1 forecasting discipline, operationalized.

The hard constraints an architect must state: **autoscaling does not cover the stateful core** (a database cannot be scaled by adding a node and waiting for rebalancing — the DB dimension keeps standing capacity, §4.6; the queue absorbs the ramp — §4.7); **autoscaling does not cover the failover instant** (the N+1 headroom of §3.4 stays standing — a scaler that grows the fleet in 15 minutes does not help the 5-second node loss); and **the peak the scaler cannot outrun still needs standing capacity** (Visa's >65,000 tps surge — ✅ §7.2 — is not a scale-up race; it is provisioned headroom or load shedding). The honest summary: **autoscaling moves headroom from "standing" to "fast", but the speed of the peak sets how much must stay standing** — which is the sizing discipline's own §3.3 peak analysis applied to the scaler.

### 5.3 The Reserved Capacity (verified — cross-ref finops_guide.md)

**Reserved capacity is the committed-use purchase (Reserved Instances, Savings Plans, committed-use discounts) that buys the standing fleet at a discount — the procurement half of the cloud sizing decision.** ✅ via [finops_guide.md](finops_guide.md) §6's verified ledger: **RIs up to ~72% off, Savings Plans up to ~66%, spot up to ~90%** (the repo's own documented figures), with the portfolio rules — RIs for steady-state predictable workloads (the always-on DB tier, the standing app fleet), Savings Plans for the variable mix, spot for batch/CI/stateless (the §4.7 queue consumers are the classic spot candidate).

The sizing interaction is the portfolio split: the *standing* fleet (the N+1 headroom of §3.4, the stateful core of §4.6, the always-on tier of §7's 24-7 rail) is the RI/SP-covered base; the *elastic* fleet (§5.2) is on-demand/spot. The split is a sizing output, not a finance input: **commit only to capacity the sizing proves is standing** — an RI on a fleet that rightsizing (§5.1) will shrink is a locked-in over-provision, which is why the FinOps portfolio guidance (✅ cross-ref finops §6's "30-60% coverage" bands and the mix tables) and this guide's design-peak math must be run on the same numbers. The banking note: the 24/7 payments estate (cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.1 — the FAST rail has no maintenance window) has a large, provably-standing base — which is precisely the shape that makes committed use the right instrument for the payments tier.

### 5.4 The Cloud Table

| Cloud mechanism | What it does to the sizing | Cross-ref / status |
|---|---|---|
| **Rightsizing** | Corrects provisioned → actual on utilization thresholds (<40% downsize, >80% upsize, 14+ days data); the operating twin of §3.2's targets | ✅ [finops_guide.md](finops_guide.md) §3.2 |
| **Autoscaling — target-tracking** | Holds utilization at a target (~60% CPU ⚠); the growth margin moves from standing capacity to scaling speed | ✅-structural; modes per finops §7 |
| **Autoscaling — schedule-based** | Pre-positions capacity for banking's predictable peaks (month-end, cut-offs) | ✅-structural; §7.2 of this guide |
| **Autoscaling — predictive** | ML forecast-driven pre-scaling; the §2.1 forecast operationalized | ✅ finops §7 |
| **Autoscaling's limits** | Stateful core, failover instant, and un-outrunnable peaks still need standing capacity | ⚠-structural (the constraints are the §3 math) |
| **Reserved capacity** | Committed-use discounts (RI ≤72%, SP ≤66%, spot ≤90%) on the *standing* fleet only | ✅ finops §6 |
| **The portfolio split** | Standing = RI/SP; elastic = on-demand/spot; the split is a sizing output | ✅-structural per finops portfolio rules |

### 5.5 The Cloud in Banking — the Hybrid Reality

The bank's cloud sizing is not the textbook's cloud sizing, and the differences are worth stating as a table in their own right (the estate frame cross-ref'd: [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) and [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) for the mainframe/on-prem reality; [finops_guide.md](finops_guide.md) §11 for the bank's FinOps layer):

| Banking reality | The cloud-sizing consequence |
|---|---|
| **The payments rail is 24/7 and non-negotiable** (✅ [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.1) | The payments tier's standing base is provably large — committed use fits; autoscaling *down* below the standing base is never allowed |
| **Data residency and MAS expectations** | The Singapore-resident footprint (cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md)) limits which regions can carry the elastic fleet; the sizing's geography is a constraint, not a preference |
| **The stateful core stays stateful** | The DB tier keeps standing capacity (§4.6) — the cloud's elasticity applies to the stateless tiers and the queue consumers, not to the ledger |
| **Regulatory reporting has hard deadlines** (§7.3) | The reporting tier is a *schedule-based* autoscaling customer — the deadline calendar *is* the scale-up schedule, and the headroom policy is stricter |
| **The mainframe estate does not "scale down"** | The legacy tiers' capacity is a procurement cycle (cross-ref the TCO guide's mainframe-vs-cloud section, [tco_modeling_guide.md](tco_modeling_guide.md) §4.3); the cloud sizing coexists with a CapEx rhythm it cannot change |

The summary of the hybrid reality: **the bank's cloud sizing is the §5 mechanisms applied to the §7 moments** — autoscaling around the batch calendar, reserved capacity on the always-on rail, rightsizing where the estate is genuinely elastic, and standing capacity wherever a deadline, a ledger, or a regulator sits.

---

## 6. The Capacity-Management Process

A sizing is a snapshot; capacity management is the discipline that keeps the snapshot honest. The ITIL 4 practice provides the frame; the monitor→forecast→plan→review loop is the machinery.

### 6.1 The ITIL 4 Capacity-and-Performance Management Practice (verified)

**ITIL 4's capacity and performance management is one of the 34 management practices — specifically one of the 17 *service management* practices — defined as a set of organizational resources (roles, activities, information, tools) whose purpose is to ensure that service performance meets agreed current and future performance requirements.** ✅ Verified this pass: the ITIL reference wiki documents the 34 practices in three categories — 14 general, 17 service (including **capacity and performance management** alongside availability management, service level management, and monitoring and event management), 3 technical — and ITIL 4 itself launched **February 2019** as the first major update since 2007. The ownership and adoption facts are cross-referenced from [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) §2's verified ledger (Axelos → PeopleCert 2021 ownership; the 37%/58%/18% ITIL/DevOps/SRE adoption survey figures ⚠-flagged there).

The practice's key activities (⚠-structural — the standard ITIL 4 Foundation description, synthesized from the practice's documented scope; the *count and category* are verified above, the activity decomposition is the widely documented standard shape):

- **Performance and capacity analysis** — the §2 profiling and §3 math on a schedule: what is the demand, what does it cost, where is the headroom?
- **Performance and capacity planning** — the forecast and the plan: when do we buy/scale, how much, against which targets (the §6.2 loop's "plan" step).
- **Monitoring and reporting** — utilization, latency, and saturation telemetry against the targets (the "monitor" step), reported to the right owners.
- **Demand management** — influencing the demand side: chargeback, throttling, workload scheduling (moving batch to the low-traffic corridor — §7.1), so capacity is a negotiated resource, not a firefight.

The ITIL placement is the architect's licence to run sizing as a *recurring, evidenced activity* rather than a one-off design deliverable: the practice sits beside availability management and service level management, which is exactly right — the SLAs (the §3.2 latency targets, contractual) and the availability targets (cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2) are the *requirements* the capacity practice serves.

### 6.2 The Monitor → Forecast → Plan → Review Loop (verified in structure)

The loop is the practice's heartbeat — four steps, one cadence (monthly review, quarterly re-forecast, annual re-baseline are the common convention, ⚠-structural; the *loop shape* is the standard continual-improvement structure, and ITIL 4's own continual-improvement model is its generic form — ✅ cross-ref [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) §2.2):

1. **Monitor** — utilization, latency, saturation, and error telemetry per dimension against the §3.2 targets; the §4.9 dimension list is the monitoring checklist. The instrument is the existing observability stack (cross-ref the low-latency guide's §2.3 metric contracts for the latency side, [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §5).
2. **Forecast** — the §2.1 methods on the fresh data: trend, seasonality, business deltas. The forecast is re-run with every material input change (a new client, a product launch — cross-ref the business-case guide for the volume side, [../management/business_case_development_guide.md](../management/business_case_development_guide.md)).
3. **Plan** — the decision step: does the current footprint meet the forecast at the design peak, with headroom? If not, the plan names the action — scale up/down (§5.2), purchase (§5.3), or re-architect (a new partition, a cache, a queue — §4) — with the date and the owner. The plan output is the capacity plan: per service, per dimension, the current utilization, the forecast, the trigger threshold, and the action.
4. **Review** — the honest step: compare the last forecast against the actuals, compare the sizing's assumptions (service time, IOPS-per-transaction, peak ratio) against the measurements, and feed the deltas back into the next cycle. A forecast that misses by 2× is not a failure — it is data; a review that does not record the miss is.

The loop's relationship to this guide's other halves is the point: **monitoring is §4's dimensions instrumented; forecasting is §2.1; planning is §3's math and §5's cloud mechanics; reviewing is the closed-loop that makes the whole thing a *discipline* rather than a document.** The banking estate runs the loop with extra teeth — the §7 batch window and regulatory deadlines make a missed forecast a missed *obligation* (a capacity shortfall at cut-off is a value-date failure, cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.5), so the loop's cadence in banking is tied to the batch calendar, not the fiscal calendar.

### 6.3 The Process Table

| Loop step | Activities | Inputs | Outputs | Status |
|---|---|---|---|---|
| **Monitor** | Utilization/latency/saturation telemetry per dimension (§4.9); threshold alerts at the §3.2 targets | Observability stack, the dimension checklist | Utilization reports, breach alerts | ✅-structural (instrumentation practice) |
| **Forecast** | §2.1 methods: trend extrapolation, business deltas, scenarios | Traffic history, roadmap, business case | Peak/average/growth numbers | ✅-structural (methods standard) |
| **Plan** | Design-peak arithmetic (§3.4); scale/purchase/re-architect decision | Forecast, targets, §3 math | The capacity plan (per service × dimension) | ✅-structural (the arithmetic is §3) |
| **Review** | Forecast-vs-actual, assumptions-vs-measurements, delta feeding | Plan, actuals | Recalibrated sizing and assumptions | ✅-structural (continual-improvement loop) |
| **Practice frame** | ITIL 4 capacity-and-performance management: analysis, planning, monitoring, demand management | The ITIL 4 practice | An evidenced, recurring capability | ✅ practice verified (§6.1); activities ⚠-structural |

### 6.4 The Capacity Plan — the Artefact

The loop's output is the **capacity plan** — the living document the §8-style sizing table becomes once it is put on a schedule. The artefact's anatomy (⚠-structural — the *structure* is the standard capacity-planning practice; the exact template is this guide's synthesis):

| Section | Contents |
|---|---|
| **The service register** | Every service in scope: owner, SLA class (cross-ref the Kafka guide's tier-1/2/3 classes for the event platform, [../banking/kafka_guide.md](../banking/kafka_guide.md) §10.3), the latency budget, the availability target (cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2) |
| **The workload baselines** | The §2.2 profile per service: λ, W, service times, I/O counts, peak/average ratios — the measured inputs, with dates |
| **The dimension sizing** | The §4.9 table per service: provisioned, utilization at current peak, utilization at design peak, the bottleneck dimension |
| **The forecast** | The §2.1 numbers per quarter: expected peak, growth, the named events (month-end, cut-offs — §7) |
| **The triggers** | Per dimension: the utilization threshold that fires the plan's action (the §3.2 targets; the <40%/>80% rightsizing bands cross-ref'd) |
| **The actions** | The decision queue: scale/purchase/re-architect per trigger, with owner, date, and the §5 mechanism (autoscale, RI, re-partition) |
| **The review log** | The §6.2 step 4 honesty record: forecast vs actual, assumptions vs measurements, the deltas fed forward |

The capacity plan is the artefact that makes the ITIL practice *auditable* — in a bank, it is also the artefact a regulator's examination can ask for (§7.3's frame; cross-ref the ERM guide's evidenced-obligation culture, [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §8), which is why the review log is not optional: an un-reviewed plan is a guess the bank is presenting as a practice.

---

## 7. The Banking Context

Banking is where capacity sizing stops being an engineering preference and becomes a regulatory and commercial obligation: the batch window, the payment peak, and the reporting deadline are *hard moments* — and the sizing that misses them is not a performance problem, it is an incident (§7's moments are the capacity equivalents of the zero-downtime guide's "the system is up at the moments the bank's obligations fire", ✅ cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.5).

### 7.1 The Batch Windows (verified — cross-ref zero_downtime_system_design_guide.md)

**The end-of-day batch — interest accrual, position marking, settlement runs, statement production — runs in a nightly corridor, and the corridor is itself a capacity budget: everything the batch must finish before the morning cut-off is a throughput requirement with a deadline.** ✅ cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.5, whose ledger verified the shapes: payment cut-offs are hard boundaries (an instruction at 17:59 clears today, at 18:01 tomorrow); the nightly batch is both the enemy (a scheduled load) and the ally (the reconciliation heartbeat); standing instructions and direct debits generate work on their own overnight schedule; and the modernisation direction — ISO 20022, event-driven settlement — compresses the batch corridors toward real time (✅ cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §8), which *removes* the low-traffic refuge the batch once had.

The sizing consequence is the **batch-window arithmetic**: a settlement batch of 2M items that must finish in a 2.5-hour window needs 2,000,000 ÷ 9,000 s ≈ **222 TPS of sustained batch throughput** — a number that is *sized, not guessed*, and that competes with the online peak for the same estate (the classic banking capacity conflict: the batch at 02:00–05:00 and the morning open at 08:00 share the fleet, and the batch's late finish is the morning's queueing). The capacity plan's batch rows (§6.2) carry: the window (start, hard deadline), the item count, the derived TPS, the engines (cross-ref [spark_tuning_guide.md](spark_tuning_guide.md) for the cluster side), and the contention policy (batch QoS vs online QoS — which tier yields when both are hot). The cross-ref'd guide's §9.5 lesson is this section's summary: **in banking, capacity is measured at the moments the bank's obligations fire — and the batch window is the largest such moment on a schedule.**

### 7.2 The Payment Peaks (verified — cross-ref nets_software_systems_guide.md)

**Payment traffic is a peak problem, not an average problem — and the peaks are schedulable, named events.** ✅ cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §2.4, whose ledger verified the industry figures: Visa averages ~8,500 tps with peaks above 65,000 tps; Mastercard handles surges of ~70,000 tps; the NETS Gateway's own product page documents a >100 tps per-instance middleware floor; and the domestic debit switch's architecture must survive its own peak — the 1-January ERP-gantry rush or a national promotion (⚠-structural there, per that guide's ledger). FAST runs 24/7 (✅ §3.2 of that guide) — the rail has no low-traffic night, which is why §5.3's standing-capacity base is provably large for the payments tier.

The sizing pattern the peaks demand:

- **The peak/average ratio is a named input, not a surprise.** Singapore retail banking sees month-end salary crediting, paydays, the CNY/ang-pao days, and the 1-January gantry rush — each a 2–5× multiple of the normal day (⚠-structural: the *existence and shape* of these peak days is standard banking-industry understanding, cross-ref'd to the NETS guide's peak discussion; the exact multiples are estate-measured, which is the §6 loop's job).
- **The sustained peak is the sizing input (§3.3).** A 3× day at a 2-hour plateau sizes the fleet; a 1-minute spike sizes the queue (§4.7) and the admission control (cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §7.2).
- **The peak is schedulable capacity (§5.2).** Month-end is *known*; schedule-based autoscaling pre-positions the fleet, and the schedule comes from the §2.1 forecast's calendar. The one thing the schedule cannot do is cover the *un*scheduled peak (a national promotion, a competitor outage) — which is what the §3.4 headroom and the §4.7 queue exist for.

The cross-ref'd guide's own final word — "the switch that never sleeps" (✅ NETS guide §10) — is the capacity reading of the same sentence: the switch's sizing has no off-peak exemption, because the counter's tap at 23:59 and the ERP gantry's rush on 1 January are the same requirement.

### 7.3 The Regulatory-Reporting Deadlines (verified — flagged ⚠)

**Regulatory reporting runs on hard deadlines — the MAS returns, the BCBS 239 risk-data submissions, the audit and record-keeping windows — and a reporting platform that misses its deadline is a supervisory incident, not a schedule slip.** The *frame* is verified: MAS supervision and BCBS 239's accuracy-completeness-timeliness principles make reporting a recurring, deadline-bound obligation (✅ cross-ref [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §8 — "regulatory reporting (COREP, MAS returns)… BCBS 239's reporting principles bind: accurate, complete, timely"; ✅ cross-ref the zero-downtime guide's §9.1 for MAS TRM / Notice 637). The *specific deadline figures are ⚠-flagged*: exact submission windows are jurisdiction-, instrument-, and regulator-specific (MAS notice schedules, COREP/EBA calendars, the local returns), and no single public table is authoritative across all of them — the architect's correct move is to source the deadlines from the compliance/legal function and encode them as *the* scheduling constraints of the reporting platform's capacity plan.

The sizing consequence: **the reporting batch has a hard window like the §7.1 settlement batch, with the extra constraint that its deadline is non-negotiable and its data volume grows with the *entire* estate** (the reporting platform ingests from every system — the §4.7 queues' 7-year tier-1 retention exists for exactly this replay, ✅ cross-ref [../banking/kafka_guide.md](../banking/kafka_guide.md) §10.4). The capacity plan for the reporting tier therefore carries: the deadline calendar (sourced, flagged ⚠), the volume curve (month-end reporting is the peak of the peak — the same 2–5× day that hits payments, one day later), and the headroom policy (a reporting platform gets *more* headroom than a normal batch tier, because the failure mode is supervisory, not commercial — cross-ref the ERM guide's risk-appetite frame, [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §2).

### 7.4 The Banking Table

| Banking reality | The capacity consequence | Cross-ref / status |
|---|---|---|
| Cut-offs are hard boundaries; 17:59 vs 18:01 is a value-date change | The seconds around cut-off are a peak the sizing must cover; no graceful degradation allowed | ✅ [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.5 |
| The nightly batch (settlement, statements) has a finish-before-open window | Batch TPS is derived from window ÷ items (2M items / 2.5 h ≈ 222 TPS); batch and online share the fleet | ✅ same guide §9.5; [spark_tuning_guide.md](spark_tuning_guide.md) |
| Payment peaks are named events: month-end, paydays, CNY, 1-January | The peak/average ratio is a planning input; schedule-based scaling pre-positions the fleet; the queue absorbs the burst | ✅ [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §2.4 |
| Visa ~8,500 tps avg / >65,000 tps peak; NETS Gateway >100 tps per instance | Sizing is to the sustained peak; the rail has no off-peak exemption | ✅ same guide's ledger |
| FAST runs 24/7 | The standing base is provably large — committed-use purchase fits the payments tier (§5.3) | ✅ NETS guide §3.2; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.1 |
| Regulatory reporting has hard deadlines (MAS returns, BCBS 239) | The reporting tier is sized to a deadline calendar with extra headroom; the failure mode is supervisory | ✅ frame (ERM guide §8); ⚠ specific deadline figures |
| Modernisation compresses batch toward real time (ISO 20022, event-driven settlement) | The low-traffic corridor shrinks; online and batch sizing converge — no refuge left | ✅ NETS guide §8; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.3 |

### 7.5 The Singapore Angle

The Singapore operating reality shapes the sizing in three concrete ways (cross-ref the Singapore cluster: [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md), [singapore_data_centres_guide.md](singapore_data_centres_guide.md)):

- **The rail is the floor.** FAST — Singapore's real-time interbank rail — runs 24/7 (✅ cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §3.2), so a Cymbal Bank-SG payments service's sizing has a permanent, non-negotiable standing base: the §5.3 committed-use story writes itself, and the §3.4 headroom cannot be borrowed from the night (there is no night on the rail).
- **The peaks have names and dates.** Month-end salary crediting, the 1-January ERP-gantry rush, and the CNY/ang-pao days are schedulable events — the §2.1 forecast's calendar and §5.2's schedule-based autoscaling are the same artefact. The sizing that does not know the gantry rush's date is a sizing that will meet it surprised (⚠-structural — the peak-day *shape* is standard industry understanding; the *dates* are estate-sourced).
- **The geography is a constraint.** Data-residency expectations and MAS supervision keep the payments and reporting footprint Singapore-resident (cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) for the DC reality — power, cooling, the moratorium→green-criteria story), which means the elastic fleet of §5.2 cannot casually burst to another region: the sizing's geography is fixed before the sizing starts, and the DC-internal network budget (the low-latency guide's ~10–20 ms line, cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §8.5) is part of the §4.4 bandwidth math, not separate from it.

---

## 8. The Worked Example

### 8.1 The Scenario — a Cymbal Bank Payments Service

**The familiar context:** Cymbal Bank's Singapore payment-initiation hub — the service that takes a client's instruction (FAST/PayNow transfer, GIRO setup, or corporate payment), validates it, runs the fraud checks, writes the ledger, and emits the event — the estate the Kafka guide's §10 designed the event platform for, and the zero-downtime guide's §10 migrated. This section sizes it.

The estate context the sizing inherits (all cross-referenced, not re-derived): the hub produces `payments.payment.executed` onto a 24-partition topic with 7-year tier-1 retention (✅ [../banking/kafka_guide.md](../banking/kafka_guide.md) §10.4); the ledger DB is the PDB-pooled Oracle-class estate (✅ cross-ref [oracle_database_guide.md](oracle_database_guide.md) §5); the rail it feeds — FAST — is 24/7 with no maintenance window (✅ [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §3.2); and the availability target is the nines discipline of the zero-downtime guide (cross-ref §2), which the sizing's N+1 policy serves. The sizing below is the §3–§5 machinery applied to that context; the inputs are the §2.2 measurements the reader is expected to replace with their own load-test results.

**The workload (all figures are the illustrative planning numbers of this worked example — the *arithmetic* is the discipline, the *inputs* are placeholders the reader replaces with their own §2 measurements):**

- **Current volumes:** 15M transactions/day; busy-hour average 600 TPS; **sustained peak 1,000 TPS** (the worst hour of the month-end salary-crediting day); peak/average ratio ≈ 1.7 on a normal day, 3× on the named peak days (§7.2).
- **The workload profile (from the §2.2 load test on a 4-vCPU/8-GB reference node):** p50 CPU service time **4 ms** per instruction (validation + fraud-check orchestration + framework overhead); 3 DB queries and 1 DB write per instruction; **200 ms** p50 downstream (ledger write + fraud-service call) in-flight time per instruction; 8 KB of internal network traffic per inbound request (2 KB in, 1 KB DB round-trip, 1 KB cache, 3 KB downstream auth, 1 KB out); 4 storage I/Os per instruction (read + write + index + journal); 2.5 KB stored per transaction.
- **The targets:** p95 latency < 500 ms end-to-end; CPU utilization target 70% under design peak (⚠ as flagged in §3.2); N+1 (one app node lost must not reduce capacity below the design peak); growth margin 30% (3-year CAGR ≈ 9% → 1.09³ ≈ 1.295); headroom 25% (the N+1 + surprise buffer).

**The design peak (the §3.4 arithmetic):** 1,000 TPS × 1.3 (growth) × 1.25 (headroom) = **1,625 TPS** — every dimension below is provisioned against 1,625 TPS, not 1,000, and certainly not 600.

### 8.2 The Workload × Dimension Sizing Table (the actual math)

| Dimension | The math | The number | The check |
|---|---|---|---|
| **CPU → cores** | Per-core ceiling = 1 ÷ 0.004 s = 250 TPS at 100%; usable at the 70% target = 175 TPS; cores = 1,625 ÷ 175 = 9.3 | **10 cores minimum** → provision **3 × 5 vCPU = 15 cores** | N+1: one node lost → 10 cores → 1,750 TPS ≥ 1,625 ✓ (utilization at design peak: 1,625 ÷ (15 × 250) = 43%; under failover: 65% — both under the 70% line ✓) |
| **App-server threads** | Per node at peak: 1,625 ÷ 3 ≈ 542 TPS × 0.2 s blocking = 108 busy threads | **Pool of 200 threads per node** (108 × ~1.8 margin) | At the pool edge: 542 × 0.2 = 108 in flight vs 200 pool — negligible queueing (Little's Law: W = L/λ; L = 108, λ = 542 → W ≈ 0.2 s ✓) |
| **App-server memory/heap** | Live set 1.5 GB + (108 in-flight × 8 KB ≈ 1 MB) + caches 1 GB + runtime → ~4 GB working | **16 GB per node** (4 GB × 4× GC convention ⚠ + OS/page cache) | 3 × 16 GB = 48 GB total; heap utilization ≈ 25% of 16 GB — room for the cache to grow |
| **Queue depth (Little's Law)** | Backlog = 1,000 TPS × 2 s tolerated delay = **2,000 messages** (2 MB at 1 KB each); consumer in-flight = 1,000 × 0.05 s = 50 | **Topic: 24 partitions** (✅ the Kafka guide's `payments.payment.executed` layout) | 24 partitions × ~100 TPS/partition = 2,400 TPS ≥ the 1,625 design peak ✓ (the sibling's 24 already encodes the margin) |
| **Database connections** | In-flight = 1,000 × 0.2 s = 200 at the peak (Little's Law); per-node pools: 200 ÷ 3 × 2× ≈ 133 | **Pools of 150 per node; DB max_connections 500** (450 pooled + ops headroom) | N+1: 2 nodes × 150 = 300 in-flight capacity ≥ 200 + retry spike ✓ (the DB is never the failover bottleneck) |
| **Database QPS → cores** | 1,000 TPS × 3 queries = 3,000 QPS at the peak; 3,000 ÷ 1,500 QPS/core (reference load test) = 2 cores | **4–6 vCPU DB** (2 × 2× margin) | Under the 70% target at 3,000 QPS: 3,000 ÷ (4 × 1,500) = 50% ✓ |
| **Storage → IOPS** | 1,000 TPS × 4 I/Os = 4,000 IOPS at the peak; × 1.25 headroom = 5,000 | **6,000 IOPS provisioned** (high-IOPS tier) | The banking write multiplier: journal + index writes make the naive 1–2 I/O estimate 4 — the §4.3 warning in action |
| **Storage → capacity** | 15M × 2.5 KB ≈ 40 GB/day ≈ 14.6 TB/yr; 2 years hot + 5 archived, × 1.2 index, × 2 replication | **Hot ~70 TB; archive ~180 TB** (7-year retention ✅-cross-ref the Kafka tier-1 rule) | The capacity-vs-IOPS split: the hot tier is IOPS-shaped (6,000 IOPS), the archive is capacity-shaped (cheap) |
| **Network** | 1,000 TPS × 8 KB × 8 = 64 Mbps internal fan-out at the peak; ×3 replication/aggregation ≈ 190 Mbps | **1 Gbps per node, 10 Gbps fabric** | The NIC is not the bandwidth — the summed flows at the §3.3 peak are; 190 Mbps at <20% of 1 Gbps ✓ |
| **GPU** | None in the payments path (cross-ref the two-speed rule: the copilot tier is separate — [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §8.1) | **Not in this tier** | The GenAI assistant never sits in the execution path (✅ that guide §8.1) |

### 8.3 The Sizing Summary

| Tier | Provisioned | Utilization at the 1,625-TPS design peak | Utilization at the 1,000-TPS current peak |
|---|---|---|---|
| **App tier** | 3 × 5 vCPU / 16 GB | 43% CPU | 27% CPU |
| **DB tier** | 4–6 vCPU, pools of 150/node, max_connections 500 | 50% QPS | 31% QPS |
| **Storage** | 6,000 IOPS + 70 TB hot / 180 TB archive | 83% of the 5,000-IOPS need ✓ | 67% |
| **Queue** | 24 partitions | 68% of partition throughput | 42% |
| **Network** | 1 Gbps/node | <20% | <12% |

The estate sits between 20% and 68% across dimensions at the design peak — the headroom band the §3 math produces when the targets are applied honestly. The one line at 83% (storage IOPS) is the dimension to watch: IOPS is the spikiest resource and the hardest to scale reactively (you cannot autoscale a disk's IOPS contract at the same speed as compute — §5.2's limits), so §6's monitor loop watches storage queue depth first.

### 8.4 The Lessons

1. **Size to the peak — the average is a cost figure, not a sizing figure.** Sizing to the 600-TPS average would have produced 6 cores, and the 1,000-TPS peak would have landed at 95%+ utilization — the hockey stick's vertical section (§3.2) — with p95 latency in seconds and the queue growing without bound (§3.1's ρ < 1 violated in spirit). The peak-vs-average discipline is the difference between a design and an incident plan.
2. **Little's Law is the sizing's skeleton.** It produced the thread pools (108 busy per node), the DB connections (200 in flight), the queue depth (2,000 messages), and the memory's in-flight term — four dimensions from one verified 1961 theorem (§3.1). A sizing that cannot state its λ and W per dimension has not done the math.
3. **The N+1 check is a second sizing.** Losing one of three nodes must not drop capacity below the design peak: 15 cores → 10 cores → 1,750 TPS ≥ 1,625 ✓. The headroom is *for* the failure mode — this is where §3.4's headroom and the zero-downtime guide's redundancy (§5 of that guide) become the same number.
4. **The banking write multiplier is real.** The naive 1–2 I/Os per transaction became 4 once the journal, index, and audit writes were counted — storage IOPS was the tightest dimension (§8.3's 83%), exactly where underestimates cause production incidents. Count the writes the regulator will audit.
5. **The cloud moves the headroom, not the math.** Rightsizing (§5.1) and autoscaling (§5.2) would converge the standing fleet toward the *current* peak (1,000 TPS) — but the stateful DB tier, the N+1 instant, and the un-outrunnable month-end peak stay standing, and the reserved-capacity commitment (§5.3) is sized against the standing base. The §8.3 numbers are the standing estate; the cloud layer trims and races around them, it does not replace them.
6. **The batch window is a deadline, not a preference.** The settlement batch's 222 TPS in a 2.5-hour window (§7.1) is sized like a payment peak with a hard finish — and the reporting tier's deadline calendar (§7.3, flagged ⚠) is sourced from compliance, then enforced by the §6 loop.

### 8.5 The Sizing Runbook — the Ten-Step Checklist

The whole discipline, compressed into the execution checklist the architect runs (each step cross-refs the section that owns it; the worked example above is the finished product of this runbook):

1. **Name the moments** — the §7 calendar first: batch windows, cut-offs, peak days, reporting deadlines. The sizing exists to serve these; nothing is sized until they are listed.
2. **Forecast the demand** (§2.1) — peak, average, growth, from history + business deltas + scenarios; state the peak's duration.
3. **Profile the workload** (§2.2) — the ten-line profile document on a reference node; record the configuration.
4. **Pick the method per dimension** (§2.3/§2.4) — benchmark-based for the tier-1 paths; rule-of-thumb (⚠) for scoping only, flagged in the document.
5. **Compute the design peak** (§3.4) — peak × (1+growth) × (1+headroom); show the arithmetic.
6. **Size every dimension** (§4.1–§4.8) — cores, memory, storage IOPS *and* capacity, network, GPU (cross-ref), DB connections/QPS/IOPS, queues, threads/heap; run the §4.10 cross-checks.
7. **Apply the cloud layer** (§5) — which headroom stays standing, which goes elastic, what the reserved-capacity commitment covers.
8. **Write the capacity plan** (§6.4) — the §8-style table with triggers and actions, on the §6.2 cadence.
9. **Instrument the monitors** (§6.2) — every dimension's threshold wired to the observability stack before go-live.
10. **Review on the schedule** (§6.2 step 4) — forecast vs actual, profile vs production, deltas fed forward; a sizing that is never reviewed is a guess with a date.

---

## 9. The Summary — One Page

**Capacity sizing is the discipline of deriving, not guessing, the resources a system needs: forecast the demand, profile the workload, apply the math, size every dimension — compute, memory, storage, network, GPU, database, queues, app servers — to the design peak, and then manage the result as a practice.**

The math is small and verified: **Little's Law (L = λW, ✅ 1961)** turns arrival rates and service times into threads, connections, queue depths, and memory; **the utilization curve (ρ = λ/μ, L = ρ/(1−ρ), W = 1/(μ(1−ρ)) ✅)** explains why the 70–80% target (⚠ convention, not theorem) exists; **peak-vs-average** says the worst hour, not the mean, is the sizing input; **headroom × growth** (1.3 × 1.25 = 1.625× in the worked example) is the arithmetic of the future and the failure mode.

The process is five steps — forecast, profile, benchmark (rule-of-thumb ⚠ for scoping only), apply the margins, review — and the management wrapper is ITIL 4's capacity-and-performance practice (✅ one of the 34 practices), run as the monitor→forecast→plan→review loop. The cloud adds rightsizing, autoscaling, and reserved capacity — mechanisms that move headroom from standing to fast, but never remove the stateful core's standing capacity or the peak that cannot be outrun.

Banking makes the discipline a regulatory fact: the **batch window** is a throughput deadline (2M items ÷ 2.5 h = 222 TPS), the **payment peaks** are named events the sizing must survive (Visa's >65,000 tps; the switch that never sleeps, ✅), and the **reporting deadlines** (⚠ figures, sourced from compliance) are supervisory moments. The worked example sized a Cymbal Bank payments service to 1,625 TPS of design peak: 15 cores, 200-thread pools, 500 DB connections, 24 queue partitions, 6,000 IOPS — every number derived, every derivation shown, every convention flagged.

The one-page rule that carries the whole discipline: **the sized for the peak** — size to the sustained peak of the worst day, with the growth margin and the headroom the failure modes demand, and let the average take care of itself. A system sized for the average is a system that fails at the peak; a system sized for the peak is a system the bank can schedule around — the batch, the reporting, and the month-end all land inside the capacity that was derived, not guessed. That is the whole guide in one sentence: **capacity sizing is the discipline of being sized for the peak — every dimension derived from the workload, every number checkable, every assumption flagged, so the system the bank runs on is the system the arithmetic promised.**

---

## 10. Glossary

| Term | Definition |
|---|---|
| **Capacity sizing** | The disciplined estimation of the compute, memory, storage, network, and service resources a system must be provisioned with to meet its performance targets under its forecast demand — including the peak, the growth, and the failure modes (§1.1). The point-in-time estimation act that feeds planning and management. |
| **Capacity planning** | The rolling activity of forecasting demand, tracking utilization against targets, and deciding when to acquire more capacity — the SRE-canon treatment of capacity as a recurring, risk-managed exercise (§1.1, §6.2). A sequence of sizing decisions over time. |
| **Capacity management** | The ITIL 4 practice (one of the 34 management practices; one of the 17 service-management practices, ✅ verified) that ensures service performance meets agreed current and future performance requirements — the organizational capability that runs sizing on a schedule (§6.1). |
| **Demand forecasting** | The estimation of the future workload — volumes, arrival rates, peaks, growth — from historical data, business plans, and explicitly stated assumptions; the §2.1 methods: historical extrapolation, business-plan-driven, analogue/industry benchmark, judgment/scenario. |
| **Workload profiling** | The measurement of what one unit of demand costs the system — service times, memory footprint, I/O and network per request, contention points — captured by load testing on a reference instance (§2.2). The input every dimension's math consumes. |
| **Benchmark** | A standardized or self-run performance measurement used to derive sizing: your own load tests (the gold standard) or published industry benchmarks such as TPC-C (✅ approved July 1992, OLTP, measured in tpmC, tpc.org) (§2.3). |
| **Little's Law** | The queueing-theory theorem (✅ John D.C. Little's 1961 proof): the long-term average number in a stationary system L equals the arrival rate λ times the mean time in the system W — L = λW — regardless of arrival/service distributions or scheduling (§3.1). |
| **Utilization** | The fraction of a resource's capacity that is busy: ρ = λ/μ. The M/M/1 result ties it to latency (W = 1/(μ(1−ρ))); the ~70–80% design target is convention (⚠) sitting on that verified curve (§3.2). |
| **Headroom** | The unallocated capacity kept for the unpredictable — failover, traffic surprise, forecast error — applied as a multiplier on the peak (§3.4). |
| **Growth margin** | The capacity reserved for the planned future — the forecast's own growth curve — applied as a multiplier on the peak (§3.4). Design peak = peak × (1+growth) × (1+headroom). |
| **Peak** | The sustained maximum demand — the worst hour of the worst day — the sizing's primary input; distinct from the average, which is a cost figure (§3.3). |
| **Average** | The mean demand over the period; the peak/average ratio measures the workload's shape and is a design input, not a sizing basis (§3.3). |
| **IOPS** | Input/output operations per second — the performance axis of storage sizing: IOPS = design-peak rate × I/O operations per transaction (§4.3). |
| **QPS** | Queries per second — the database demand rate (a transaction generates a query mix: 3 queries per payment instruction in §8) used with per-core query rates to size DB CPU (§4.6). |
| **TPS** | Transactions per second — the arrival-rate unit of the sizing (the payments estate's 1,000 TPS peak, §8; the NETS Gateway's >100 tps per-instance floor, ✅ cross-ref). |
| **Cores** | The compute unit of the sizing: cores = design-peak rate × service time ÷ utilization target; a vCPU is a scheduling slice — benchmark on the target instance family (§4.1). |
| **Threads** | The concurrency unit of the app tier: threads = per-node rate × blocking time (Little's Law) with a margin; the async model replaces threads with in-flight counts and backpressure (§4.8). |
| **Heap** | The managed memory of a runtime (e.g., the JVM): sized as live set + in-flight working set + runtime overhead; the 4×-live-set GC convention is ⚠ practice (§4.2, §4.8). |
| **Rightsizing** | The ongoing correction of provisioned capacity toward actual demand on utilization thresholds (<40% downsize, >80% upsize, 14+ days of data — ✅ cross-ref [finops_guide.md](finops_guide.md) §3.2); the operating twin of the sizing targets (§5.1). |
| **Autoscaling** | The cloud mechanism that converts standing headroom into reactive headroom: target-tracking, schedule-based, and predictive modes; it cannot cover the stateful core, the failover instant, or an un-outrunnable peak (§5.2). |
| **Reserved capacity** | Committed-use purchases (Reserved Instances, Savings Plans, committed-use discounts) on the standing fleet only — RIs up to ~72%, Savings Plans up to ~66%, spot up to ~90% (✅ cross-ref finops §6); the portfolio split is a sizing output (§5.3). |
| **ITIL** | The IT Infrastructure Library — the service-management framework; ITIL 4 (✅ February 2019) defines 34 practices including capacity and performance management; owned by Axelos/PeopleCert (✅ cross-ref [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) §2). |
| **Batch window** | The scheduled corridor (e.g., the 02:00–05:00 end-of-day cycle) in which batch work must finish — a throughput deadline: items ÷ window seconds = required TPS (§7.1). |
| **Regulatory deadline** | The hard submission window of a regulatory report (MAS returns, BCBS 239 data) — non-negotiable, estate-wide, and the sizing constraint of the reporting tier; specific figures ⚠-flagged, sourced from compliance (§7.3). |

---

## 11. The Verification Ledger

**What was verified this pass (✅) vs flagged (⚠).** The web-search backend was degraded during this pass (empty result sets); verification was done by direct URL extraction plus cross-reference to sibling guides' own verified ledgers. Nothing was fabricated; every illustrative figure in §8 is labelled as such with the arithmetic shown.

| Claim | Status | Evidence |
|---|---|---|
| Little's Law: L = λW; theorem by John D.C. Little; proof published 1961; holds regardless of arrival/service distributions | ✅ verified | Queueing-theory reference material (Little's law article, extracted live this pass; cites Little's 1961 publication; the 1954 use-without-proof and Morse's earlier form) |
| M/M/1: ρ = λ/μ; stability requires ρ < 1; mean number in system ρ/(1−ρ); response time 1/(μ(1−ρ)) | ✅ verified (response time derived via Little's Law) | M/M/1 reference material (extracted live this pass); W = 1/(μ(1−ρ)) follows from L = ρ/(1−ρ) and λ = ρμ |
| The ~70–80% CPU utilization target | ⚠ flagged — the curve is verified math; the specific band is practice convention, not pinned to one authoritative source this pass | The M/M/1 curve above; the FinOps thresholds (<40%/>80%) are that guide's own verified practice numbers |
| Rule-of-thumb sizing coefficients (cores per TPS, GB per user, etc.) | ⚠ flagged — method real for scoping; coefficients are folklore, workload-dependent by an order of magnitude | No authoritative canonical table found this pass |
| ITIL 4: launched February 2019; 34 practices = 14 general + 17 service + 3 technical; capacity and performance management among the 17 service-management practices | ✅ verified | ITIL reference wiki (extracted live this pass); taxonomy matches [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) §2.4's ledger |
| ITIL 4 ownership (Axelos → PeopleCert 2021) and adoption survey figures (37% ITIL / 58% DevOps / 18% SRE) | ✅ cross-referenced (survey ⚠ there) | [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) §2's own ledger |
| TPC-C: approved July 1992; OLTP benchmark; five concurrent transaction types; nine table types; measured in tpmC | ✅ verified | tpc.org TPC-C homepage (extracted live this pass) |
| SRE: "100% is the wrong reliability target" | ✅ cross-referenced | [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §1.2 (verified in that ledger) |
| SRE Workbook chapters: Managing Load (ch 11), Identifying and Recovering from Overload (ch 17); SRE Book: Handling Overload, Appendix B on capacity redundancy | ✅ chapter structure verified (workbook TOC extracted live); chapter content ⚠-structural | sre.google workbook table of contents (extracted live this pass) |
| AWS Well-Architected: six pillars; Performance Efficiency pillar (Nov 2024 revision) with five focus areas; capacity-planning guidance | ✅ pillar structure verified; the capacity-planning detail ⚠-structural | docs.aws.amazon.com PE pillar page (extracted live this pass) |
| NETS Gateway >100 tps per instance; Visa ~8,500 tps avg / >65,000 tps peak; Mastercard ~70,000 tps surges; FAST 24/7 ISO 20022; BCS operator since 1976; "the switch that never sleeps" | ✅ cross-referenced | [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §2.4/§3.2/§10 ledger |
| Batch windows, cut-offs (17:59 vs 18:01), the nightly batch, standing instructions | ✅ cross-referenced | [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.5 ledger |
| Kafka: `payments.payment.executed` at 24 partitions; tier-1 7-year retention; RF=3/min.insync=2 | ✅ cross-referenced | [../banking/kafka_guide.md](../banking/kafka_guide.md) §10.3–§10.4 ledger |
| FinOps: rightsizing thresholds (<40%/>80%, 14+ days); RI up to 72% / SP up to 66% / spot up to 90%; autoscaling modes | ✅ cross-referenced | [finops_guide.md](finops_guide.md) §3.2/§6/§7 ledger |
| GPU sizing shapes (KV-cache dominance, latency/throughput as the same resource, H100/B200) | ✅ cross-referenced, not re-derived | [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) §3–§4, §7 ledger |
| Oracle PDB capacity pooling | ✅ cross-referenced | [oracle_database_guide.md](oracle_database_guide.md) §5 |
| MAS supervision, BCBS 239 reporting principles, MAS TRM/Notice 637 | ✅ cross-referenced (frame) | [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §8; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.1 |
| Specific regulatory-reporting deadline figures | ⚠ flagged — jurisdiction- and instrument-specific; source from compliance/legal | No authoritative single public table this pass; the *frame* is verified |
| Demand-forecasting method taxonomy; workload-profiling practice; peak-vs-average discipline; headroom/growth-margin pattern; per-dimension sizing methods | ⚠-structural — industry-standard practice, widely documented, not pinned to one primary source this pass | The verified §3 math the methods consume; the sibling ledgers cited inline |
| All worked-example numbers (§8) | Labelled illustrative planning figures; the arithmetic shown | The arithmetic is checkable; the inputs are placeholders for the reader's own §2 measurements |

**The honest close:** the *mathematical* core of this guide (Little's Law, the M/M/1 curve, the TPC-C benchmark facts, the ITIL 4 practice structure) is verified this pass; the *practice* layer (targets, ratios, methods, banking shapes) is cross-referenced to sibling ledgers where the repo owns them, and flagged ⚠/⚠-structural where it is convention or could not be pinned to a primary source given the degraded web backend. No claim in this guide should be repeated as fact without checking its status in this ledger.

---

## 12. Cross-References Recap

- **The availability twin:** [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — the nines, the 24-7 rail, the batch windows (§7.1 of this guide), the "100% is the wrong target" frame (§1.3).
- **The cloud-cost twin:** [finops_guide.md](finops_guide.md) — rightsizing thresholds, purchase options, autoscaling modes (§5 of this guide).
- **The cost layer:** [tco_modeling_guide.md](tco_modeling_guide.md) — what the sizing's footprint costs, all-in.
- **The GPU dimension:** [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) — TTFT/TPOT, KV-cache, continuous batching (§4.5 of this guide cross-refs and stops).
- **The cluster dimension:** [spark_tuning_guide.md](spark_tuning_guide.md) — the batch/analytics cluster sizing.
- **The back-of-envelope estimates:** [google_system_design_interview_guide.md](google_system_design_interview_guide.md) · [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) · [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) — the scoping layer §2.4 formalizes.
- **The ITIL home:** [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) — the 34-practice taxonomy, the SRE overlay (§6 of this guide operates the practice).
- **The queue dimension:** [../banking/kafka_guide.md](../banking/kafka_guide.md) — partitions, retention, the §10.4 catalog (§4.7 of this guide).
- **The DB dimensions:** [oracle_database_guide.md](oracle_database_guide.md) · [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) (§4.6).
- **The payments peaks:** [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) · [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) (§7.2).
- **The regulatory frame:** [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) (§7.3).
- **The investment frame:** [../management/business_case_development_guide.md](../management/business_case_development_guide.md) — the sizing budget as a business-case line (§6.2, §8).

---

*Capacity Sizing: The Sizing Discipline — the repo's dedicated deep-dive on the sizing methodology. The final word: **the sized for the peak.***
