# Zero-Downtime System Design

**The Availability Discipline Deep-Dive — the Zero-Downtime Aspiration and the Availability Math (the Nines, MTBF/MTTR, the SLO/SLA/Error-Budget), the Deployment Strategies (Rolling, Blue-Green, Canary, A/B, Feature Flags, Dark Launches), the Data Layer (Expand-Contract, Dual-Writes, CDC), the Stateful Systems (Failover, Quorum, Multi-AZ), the Infrastructure (Load Balancers, Health Checks, DNS Failover), the Chaos Engineering (Chaos Monkey, GameDays), the Resilience Patterns (Circuit Breakers, Bulkheads, Retries, Idempotency, Graceful Degradation), the Zero-Downtime Reality in Banking (the 24-7 Payments Switch), a Zero-Downtime Migration Worked Example, and the One-Page Summary — the Zero-Downtime Illusion**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Reliability & System Design — the DEDICATED zero-downtime deep-dive of the reliability cluster. Where [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) covers the *platform* side (Kubernetes/OpenShift workload availability) and [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) covers the *migration runbook*, this guide is the *discipline* itself: the availability math, the deployment and data-layer techniques that keep systems up through change, the stateful and infrastructure mechanisms, the chaos-engineering practice that proves it, and the banking reality that demands it. It is written to stand alone and to cross-ref the cluster heavily.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the availability canon — the nines/MTBF/MTTR availability math references (calcoi.com, calculatorhub.com, mechcodex.com), the Google SRE book — *Site Reliability Engineering: How Google Runs Production Systems* (O'Reilly, 2016; Beyer, Jones, Petoff, Murphy) — and the SRE Workbook (sre.google/workbook — the error-budget-policy and implementing-SLOs chapters); the deployment-strategy literature — the Martin Fowler bliki (BlueGreenDeployment 2010, CanaryRelease, FeatureFlag/FeatureToggle 2010, ParallelChange, CircuitBreaker 2014) and the Continuous Delivery book lineage (Humble & Farley); the parallel-change/expand-contract pattern literature (the Prisma Data Guide, the Fowler bliki); the chaos-engineering canon (Netflix's Chaos Monkey — Wikipedia-documented 2011 origin — the Principles of Chaos, the GameDay practice from the SRE literature); the reliability references (Michael Nygard's *Release It!* for circuit breakers and bulkheads, the AWS Architecture Blog on timeouts/retries/backoff with jitter, the transactional-outbox/CDC literature — Chris Richardson's microservices.io outbox pattern, Debezium); the AWS documentation (RDS Multi-AZ failover, Route 53 health checks and DNS failover); and the repo's own verified cluster (the banking series, the NETS guide, the OpenShift guide). NOTE: this pass had **live web access** (self-hosted Firecrawl backend) — the key claims were verified on 2026-08-24 via web search and direct page extraction (Fowler's bliki pages extracted live). Anything that could not be verified is flagged ⚠ honestly.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [OpenShift Workload Availability](openshift_workload_availability_guide.md) (**THE platform-side companion — HA/DR/BC, RPO/RTO, probes, multi-AZ topology — cross-ref constantly in §5–§6**), [Monolith to Microservices](monolith_to_microservices_guide.md) (**THE migration runbook — cross-ref §10's migration path**), [Domain-Driven Design](domain_driven_design_guide.md) (the decomposition angle — light cross-ref in §10), [Kafka Alternatives](kafka_alternatives_guide.md) and [Message Queue Data Loss](message_queue_data_loss_guide.md) (the messaging-reliability angle — at-least-once and idempotency in §4 and §8), [Late-Arriving Data](late_arriving_data_guide.md) (the data-lag angle in §4), [Distributed Rate Limiter](distributed_rate_limiter_guide.md) (the resilience angle — light), [Distributed Auth](distributed_auth_guide.md) (the auth-availability angle — light), [Event Stream Processing](event_stream_processing_guide.md) (the streaming-availability angle in §4 and §10), [Grokking System Design Companion](grokking_system_design_companion_guide.md), [DDIA Study Companion](ddia_study_companion_guide.md) and [System Design Interview Insiders](system_design_interview_insiders_guide.md) (the study-guide genre — cross-ref the availability chapters), [Data Centres in Singapore](singapore_data_centres_guide.md) (the DC/DR frame in §6)
> **Companion guides (banking/, prefix `../banking/`):** [NETS Software Systems](../banking/nets_software_systems_guide.md) (**THE banking cross-ref — the switch that never sleeps — the 24-7 payments reality of §9**), [Core Banking Systems](../banking/core_banking_systems_guide.md), [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) (the trading-day availability angle), [Crédit Agricole Software Systems](../banking/credit_agricole_software_systems_guide.md) (the house context for §10), and the bank software-systems series
> **Companion guides (management/, prefix `../management/`):** [Business Case Development](../management/business_case_development_guide.md) (the RTO/RPO investment angle — light cross-ref in §2 and §9)

---

**How to use this guide:** Section 1 is the availability overview — what the zero-downtime aspiration actually is, and the overview table (aspect / description). Section 2 is the availability math — the nines, MTBF/MTTR, the SLO/SLA/error-budget machinery (cross-ref the Google SRE book), and the math table. Section 3 is the deployment strategies — rolling, blue-green, canary, A/B, feature flags, dark launches (cross-ref the Fowler bliki), and the strategies table (strategy / mechanics / notes). Section 4 is the data layer — expand-contract/parallel-change schema migrations, dual-writes and the outbox, CDC, and the data table. Section 5 is the stateful systems — database failover, quorum, multi-AZ, and the stateful table. Section 6 is the infrastructure — load balancers, health checks, DNS failover, and the infra table. Section 7 is the chaos engineering — the Chaos Monkey, the Principles of Chaos, GameDays, and the chaos table. Section 8 is the resilience patterns — circuit breakers, bulkheads, retries-with-backoff, idempotency, graceful degradation, and the patterns table. Section 9 is zero-downtime in banking — the 24-7 payments reality (cross-ref the NETS guide — the switch that never sleeps) and the banking table. Section 10 is the worked example — a zero-downtime migration of a Cymbal Bank system. Section 11 is the one-page summary — the final word is **the zero-downtime illusion**. The glossary, the claims-status ledger and the cross-references close the file. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`; guides in the `ai_llm/` subdirectory are prefixed `ai_llm/`. **Integrity convention:** ✅ = verified this pass (live web or the cross-referenced guide's ledger); ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source. The single most important honesty rule of this guide: **"zero downtime" is an aspiration, not an absolute — every technique below reduces or hides downtime, and the guide says so plainly (§1.2, §11).**

---

## Table of Contents

1. [The Availability Overview](#1-the-availability-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Zero-Downtime Aspiration — What It Actually Means](#12-the-zero-downtime-aspiration--what-it-actually-means)
   - 1.3 [The Overview Table — Aspect / Description](#13-the-overview-table--aspect--description)
   - 1.4 [Reading the Overview Table](#14-reading-the-overview-table)
   - 1.5 [The Canon — Where the Discipline Comes From](#15-the-canon--where-the-discipline-comes-from)
2. [The Availability Math](#2-the-availability-math)
   - 2.1 [The Nines](#21-the-nines)
   - 2.2 [MTBF and MTTR — the Availability Formula](#22-mtbf-and-mttr--the-availability-formula)
   - 2.3 [SLO, SLA and the Error Budget — the Google SRE Framework](#23-slo-sla-and-the-error-budget--the-google-sre-framework)
   - 2.4 [The Math Table](#24-the-math-table)
   - 2.5 [The Math in Practice — Worked Numbers](#25-the-math-in-practice--worked-numbers)
3. [The Deployment Strategies](#3-the-deployment-strategies)
   - 3.1 [The Deployment Problem — Why Change Is the Outage](#31-the-deployment-problem--why-change-is-the-outage)
   - 3.2 [Rolling Deployment](#32-rolling-deployment)
   - 3.3 [Blue-Green Deployment](#33-blue-green-deployment)
   - 3.4 [Canary Release](#34-canary-release)
   - 3.5 [A/B Testing — and How It Differs from Canary](#35-ab-testing--and-how-it-differs-from-canary)
   - 3.6 [Feature Flags and Dark Launches](#36-feature-flags-and-dark-launches)
   - 3.7 [The Strategies Table — Strategy / Mechanics / Notes](#37-the-strategies-table--strategy--mechanics--notes)
   - 3.8 [Choosing a Strategy — and Combining Them](#38-choosing-a-strategy--and-combining-them)
4. [The Data Layer](#4-the-data-layer)
   - 4.1 [The Data Problem — Schema and State](#41-the-data-problem--schema-and-state)
   - 4.2 [Expand-Contract (Parallel Change) Schema Migrations](#42-expand-contract-parallel-change-schema-migrations)
   - 4.3 [Dual-Writes — and the Transactional Outbox](#43-dual-writes--and-the-transactional-outbox)
   - 4.4 [Change Data Capture (CDC)](#44-change-data-capture-cdc)
   - 4.5 [The Data Table](#45-the-data-table)
   - 4.6 [The Schema-Migration Playbook — Concrete Rules](#46-the-schema-migration-playbook--concrete-rules)
5. [The Stateful Systems](#5-the-stateful-systems)
   - 5.1 [Why State Is the Hard Part](#51-why-state-is-the-hard-part)
   - 5.2 [Database Failover](#52-database-failover)
   - 5.3 [Quorum — the Consensus Core](#53-quorum--the-consensus-core)
   - 5.4 [Multi-AZ — the Failure-Domain Frame](#54-multi-az--the-failure-domain-frame)
   - 5.5 [The Stateful Table](#55-the-stateful-table)
   - 5.6 [The Stateful Decision Map](#56-the-stateful-decision-map)
6. [The Infrastructure](#6-the-infrastructure)
   - 6.1 [The Infrastructure Stack — Routing Around Failure](#61-the-infrastructure-stack--routing-around-failure)
   - 6.2 [Load Balancers](#62-load-balancers)
   - 6.3 [Health Checks](#63-health-checks)
   - 6.4 [DNS Failover](#64-dns-failover)
   - 6.5 [The Infra Table](#65-the-infra-table)
   - 6.6 [The Routing Ladder — One Request's Journey](#66-the-routing-ladder--one-requests-journey)
7. [The Chaos Engineering](#7-the-chaos-engineering)
   - 7.1 [The Chaos Monkey — the Origin](#71-the-chaos-monkey--the-origin)
   - 7.2 [The Principles of Chaos](#72-the-principles-of-chaos)
   - 7.3 [GameDays — the Deliberate Drill](#73-gamedays--the-deliberate-drill)
   - 7.4 [The Chaos Table](#74-the-chaos-table)
   - 7.5 [Building the Practice — the Blast-Radius Ramp](#75-building-the-practice--the-blast-radius-ramp)
8. [The Resilience Patterns](#8-the-resilience-patterns)
   - 8.1 [Circuit Breakers](#81-circuit-breakers)
   - 8.2 [Bulkheads](#82-bulkheads)
   - 8.3 [Retries with Backoff](#83-retries-with-backoff)
   - 8.4 [Idempotency](#84-idempotency)
   - 8.5 [Graceful Degradation](#85-graceful-degradation)
   - 8.6 [The Patterns Table](#86-the-patterns-table)
   - 8.7 [The Pattern Stack — How They Compose](#87-the-pattern-stack--how-they-compose)
9. [The Zero-Downtime in Banking](#9-the-zero-downtime-in-banking)
   - 9.1 [The 24-7 Payments Reality](#91-the-24-7-payments-reality)
   - 9.2 [The Switch That Never Sleeps — the NETS Cross-Ref](#92-the-switch-that-never-sleeps--the-nets-cross-ref)
   - 9.3 [What Zero Downtime Means Inside a Bank](#93-what-zero-downtime-means-inside-a-bank)
   - 9.4 [The Banking Table](#94-the-banking-table)
   - 9.5 [The 24-7 Shapes — Cut-Offs and the Nightly Batch](#95-the-247-shapes--cut-offs-and-the-nightly-batch)
10. [The Worked Example — A Zero-Downtime Migration](#10-the-worked-example--a-zero-downtime-migration)
    - 10.1 [The Scenario — a Cymbal Bank System](#101-the-scenario--a-cymbal-bank-system)
    - 10.2 [The Migration Design](#102-the-migration-design)
    - 10.3 [The Lessons](#103-the-lessons)
11. [The Summary — One Page](#11-the-summary--one-page)
12. [Glossary](#12-glossary)
13. [Claims Status and Verification Notes](#13-claims-status-and-verification-notes)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)

---

## 1. The Availability Overview

### 1.1 The Short Answer

**Zero-downtime system design is the discipline of changing and failing a system without its users noticing.** It is not a single technique and it is not a literal guarantee — it is a layered stack of mathematics (what availability is actually measurable as), deployment mechanics (how new code reaches production without a cut-over gap), data techniques (how schemas and state evolve under a live workload), stateful mechanisms (how databases survive their own failures), infrastructure routing (how traffic flows around broken components), chaos practice (how you prove the whole chain works), and resilience patterns (how one component's failure is prevented from becoming everyone's). The banking industry is the purest forcing function for the discipline: a payment switch that authorises the counter's card tap has no maintenance night — it must be "the switch that never sleeps" (cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §10). The whole guide in one paragraph: **availability is a number, downtime is a budget, change is the risk, state is the hard part, redundancy is the mechanism, chaos is the proof, and "zero downtime" is the illusion you engineer so carefully that it becomes the reality your users experience.**

### 1.2 The Zero-Downtime Aspiration — What It Actually Means

The phrase "zero downtime" (also "no downtime", "seamless cutover", "zero-downtime deployment/migration") describes the **aspiration that a system remains continuously available to its users while it is deployed, upgraded, migrated or partially failed**. Verified against the deployment-strategy and migration literature (the Fowler bliki ✅, the expand-contract/parallel-change pattern literature ✅, the zero-downtime-migration engineering literature ✅), the honest definition has three parts:

1. **It is an aspiration, not an absolute.** Mathematically, "100% availability" is a target the SRE canon explicitly calls wrong — the Google SRE book's "Embracing Risk" chapter argues that **100% is the wrong reliability target** because the cost of the last fraction of availability grows super-linearly while users cannot perceive the difference (✅ SRE book framing, §2.3). What the industry means by "zero downtime" is: *no downtime the user perceives and no downtime caused by our own change process*. Planned changes are made invisible (blue-green switch, rolling drain, feature-flag flip); unplanned failures are absorbed by redundancy (failover, quorum, multi-AZ). Both are "designed-away" downtime, which is the real content of the discipline.
2. **It is measured in the gap between what breaks and what users see.** The availability math (§2) is the *perceived-availability* lens: a database failover that takes 90 seconds (✅ AWS-documented RDS Multi-AZ failover times, §5.2) is invisible to a user whose request is retried or queued; the same 90 seconds is a full outage to a synchronous API caller with no retry. Zero-downtime design therefore works on *both sides of the gap*: reduce the failure duration (redundancy, faster failover) **and** hide it (retries, queues, stand-in processing, idempotency).
3. **It is a property of the change process, not of the steady state.** A system that never changes can be made arbitrarily available with enough redundancy; the discipline earns its name when *change* — deployments, schema migrations, platform migrations, version upgrades — happens without a service gap. That is why §3 (deployment strategies) and §4 (the data layer) carry most of the weight in this guide: **the outage you can schedule is the outage you must design out first.**

A note on the term's limits, flagged honestly: some sources use "zero downtime" loosely as marketing for near-zero or "single-digit-seconds" availability; no credible source claims literal zero seconds of unavailability over a long horizon (⚠-structural — the honest reading of the entire reliability literature). This guide uses "zero downtime" in the disciplined sense above and says so in §11's final word, **the zero-downtime illusion**.

### 1.3 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **The aspiration** | Continuous user-visible availability through planned change and unplanned failure; "zero downtime" as a disciplined target, not a literal guarantee (§1.2) |
| **The math** | Availability as a measurable number — the nines, MTBF/MTTR, SLO/SLA/error budgets — the scoreboard the discipline is played against (§2) |
| **The deployment strategies** | Rolling, blue-green, canary, A/B, feature flags, dark launches — how new code reaches production without a cut-over gap (§3) |
| **The data layer** | Expand-contract/parallel-change migrations, dual-writes and the outbox, CDC — how schemas and data evolve under a live workload (§4) |
| **The stateful systems** | Database failover, quorum, multi-AZ — how the stateful core survives its own failures (§5) |
| **The infrastructure** | Load balancers, health checks, DNS failover — how traffic is routed around broken components (§6) |
| **The chaos engineering** | Chaos Monkey, the Principles of Chaos, GameDays — how you prove the resilience chain actually works (§7) |
| **The resilience patterns** | Circuit breakers, bulkheads, retries-with-backoff, idempotency, graceful degradation — how one failure is prevented from becoming everyone's (§8) |
| **The banking reality** | The 24-7 payments switch — the industry where "the switch that never sleeps" is a regulatory and commercial fact, not a slogan (§9) |
| **The worked example** | A Cymbal Bank zero-downtime migration — the whole discipline applied to one familiar system (§10) |

### 1.4 Reading the Overview Table

The ten aspects are deliberately ordered as a **dependency chain**: the math (§2) tells you what target you are engineering for; the deployment strategies (§3) and data layer (§4) make *change* safe; the stateful mechanisms (§5) and infrastructure (§6) make *failure* survivable; chaos engineering (§7) proves the combination; the resilience patterns (§8) are the connective tissue that lets each layer absorb the layer below it; the banking section (§9) is the forcing-function case study; and the worked example (§10) ties every technique to one system. Read §2 first if you want the numbers, §9 first if you want the motivation, and §10 first if you want the whole discipline in one narrative — the rest of the guide unpacks what the worked example uses.

### 1.5 The Canon — Where the Discipline Comes From

Zero-downtime design did not arrive as one doctrine; it is the convergence of four verified lineages, and knowing which lineage a technique comes from explains why it works and where its edges are:

1. **The availability-measurement lineage** — the nines, MTBF/MTTR and the availability formula come from classical reliability engineering (✅ the availability-math references verified this pass), and were absorbed into software operations through the SRE movement — the Google SRE book (O'Reilly, 2016) turned them into the SLI/SLO/error-budget management system of §2.3 ✅.
2. **The deployment-and-delivery lineage** — blue-green, canary and feature flags were codified in the continuous-delivery tradition: the Fowler bliki entries (✅ 2010–2014, verified live this pass) and the Humble & Farley *Continuous Delivery* book that the blue-green bliki builds on. This lineage's core claim: **deployment is a routine, automated, reversible act — and the release is not the risky moment, the *exposure* is.**
3. **The database-evolution lineage** — expand-contract/parallel change comes from the evolutionary-database-design tradition (✅ Fowler's ParallelChange bliki; ✅ the Prisma Data Guide), which holds that **schemas are code and must evolve with the same reversibility discipline as code** — additive, backward-compatible, contract-last.
4. **The resilience-and-chaos lineage** — circuit breakers and bulkheads come from Michael Nygard's *Release It!* (✅ via Fowler's CircuitBreaker bliki attribution), retries-with-jitter from the AWS reliability canon (✅ Brooker, 2015), and chaos engineering from Netflix's Chaos Monkey (✅ 2011) and the Principles of Chaos ✅. This lineage's claim: **failure is normal, and the system that survives is the one that has rehearsed.**

The convergence is the point: the availability lineage says *what* to guarantee, the delivery lineage says *how to change without breaking it*, the database lineage says *how to move the state underneath*, and the resilience lineage says *how to behave when it breaks anyway*. Every section of this guide maps to at least one lineage, and the worked example of §10 uses all four at once.

---

## 2. The Availability Math

### 2.1 The Nines

Availability is conventionally expressed as a percentage of time a system is up, and the "nines" shorthand counts the leading nines in that percentage. **The canonical table** (✅ verified this pass against the availability-math references — calcoi.com, calculatorhub.com, conyso.com, mechcodex.com; the numbers are pure arithmetic on 365.25 days/year and are consistent across all sources):

| Availability | Downtime per year | Downtime per month | Downtime per week | Downtime per day |
|---|---|---|---|---|
| 99% (two nines) | 3.65 days | 7.31 hours | 1.68 hours | 14.4 minutes |
| 99.9% (three nines) | 8.77 hours | 43.8 minutes | 10.1 minutes | 1.44 minutes |
| 99.95% | 4.38 hours | 21.9 minutes | 5.04 minutes | 43.2 seconds |
| 99.99% (four nines) | 52.6 minutes | 4.38 minutes | 1.01 minutes | 8.64 seconds |
| 99.999% (five nines) | 5.26 minutes | 26.3 seconds | 6.05 seconds | 0.86 seconds |
| 99.9999% (six nines) | 31.5 seconds | 2.63 seconds | 0.605 seconds | 0.086 seconds |

**What the table teaches the architect:** the jump from 99.9% to 99.99% is not a 10× improvement in effort — it is a ~100× tightening of the *allowed* annual downtime (8.77 h → 52.6 min), and each additional nine typically costs more than the previous one because it forces you up the failure-domain ladder (single server → rack → AZ → region → multi-region). A "five-nines" target means **the entire system — including your deployments, your migrations and your failure recovery — may be unavailable for about five minutes a year**. That single sentence is why the rest of this guide exists: you cannot spend five minutes on a scheduled maintenance window and still claim five nines; the maintenance must be invisible (§3), the schema change must be live (§4), and the failover must be automatic (§5–§6).

### 2.2 MTBF and MTTR — the Availability Formula

The nines are the *output*; the inputs are the two classical reliability quantities (✅ verified this pass — the availability-math references all present the same formula):

- **MTBF — Mean Time Between Failures**: the average elapsed time between consecutive failures of a system (total operating time ÷ number of failures). It is a *reliability* measure — how long the system runs before breaking.
- **MTTR — Mean Time To Repair** (sometimes "Mean Time To Recover" or "Mean Time To Restore"): the average time to restore service after a failure — and in practice it includes **detection time, response time and repair/restore time**, not just the hands-on fix.

**The availability formula** (✅ consistent across sources):

```
Availability A = MTBF / (MTBF + MTTR)
```

which is the same fraction as *uptime / (uptime + downtime)*. The formula is the discipline in miniature: **availability is improved by increasing MTBF (reliability — build things that fail less) and by decreasing MTTR (resilience — recover faster when they fail)**. For a given availability target, the MTTR budget is brutally small: to hold 99.99% with an MTBF of 30 days, MTTR must be ≤ ~4.3 minutes — which is why automatic failover, health-check-driven routing and self-healing (§5–§6) matter more than heroic manual recovery. The RTO/RPO pair (Recovery Time Objective / Recovery Point Objective — the DR-side targets: how fast you must be back, and how much data loss you may accept) is the business-facing translation of the same trade-off (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §3 for the HA/DR/BC framing, and [../management/business_case_development_guide.md](../management/business_case_development_guide.md) for the investment case — the cost of an extra nine must be justified by the business value of the avoided downtime).

### 2.3 SLO, SLA and the Error Budget — the Google SRE Framework

The modern operational machinery for *managing* availability is the SLI/SLO/error-budget framework from the Google SRE book — *Site Reliability Engineering: How Google Runs Production Systems*, O'Reilly, 2016, edited by **Beyer, Jones, Petoff and Murphy** (✅ verified — the 2016 O'Reilly edition is consistently cited, including by the SRE Workbook at sre.google and by the secondary literature; the framework's home chapters are "Service Level Objectives" and "Embracing Risk"). The terms, verified against the SRE book and the sre.google/workbook chapters:

- **SLI — Service Level Indicator**: the *measured* quantity — e.g. request success rate, latency percentile, availability fraction. The raw metric before any target is applied.
- **SLO — Service Level Objective**: the *target* you set on the SLI — e.g. "99.9% of requests succeed over a 30-day window". The SLO is the engineering contract with yourself; **an SLO is deliberately not 100%** — the book's "Embracing Risk" chapter argues 100% is the wrong target because the cost of chasing the last fraction is disproportionate to user benefit (✅ SRE book).
- **SLA — Service Level Agreement**: the *external* contract with customers, almost always looser than the SLO (you promise the customer 99.9% while you engineer to 99.99%), because the SLA carries consequences (credits, penalties) and must leave headroom for the failures you cannot foresee (✅ standard SRE framing).
- **Error budget**: the allowed failure within the SLO — **error budget = 1 − SLO** (✅ SRE book). A 99.9% SLO over 30 days buys an error budget of ~43 minutes of failure per month. The operational consequence is the famous one: **unplanned outages and planned changes *spend the same budget*** — a maintenance window that causes user-visible failures consumes error budget exactly like an outage does, so "scheduled downtime" is not free. When the budget is spent, the SRE practice is to **freeze launches** — slow down change until the budget regenerates (✅ SRE Workbook, the error-budget-policy chapter — the policy mechanics: budget consumption → release gating).

The error budget is the bridge between §2 and the rest of this guide: **every deployment strategy in §3 and every data technique in §4 exists to make change not spend the budget** — and the budget, in turn, is the honest accounting that exposes "zero-downtime" claims: if your changes cause user-visible failures, no amount of marketing makes them zero-downtime.

### 2.4 The Math Table

| Concept | Definition (verified) | The Architect's Reading |
|---|---|---|
| **Nines** | The count of leading nines in availability: 99.9% = three nines ≈ 8.77 h/year downtime; 99.99% ≈ 52.6 min/year; 99.999% ≈ 5.26 min/year ✅ | Each nine is ~10× tighter annual allowance at super-linear cost; the target must be chosen, not inherited |
| **MTBF** | Mean Time Between Failures — operating time ÷ failures ✅ | The reliability lever: fail less often (design, testing, chaos-proven robustness) |
| **MTTR** | Mean Time To Repair/Recover — detection + response + restore ✅ | The resilience lever: recover faster (automation, failover, runbooks) — usually the cheaper lever |
| **A = MTBF/(MTBF+MTTR)** | The availability formula ✅ | The whole discipline in one fraction: reliability × resilience |
| **SLI** | The measured indicator (success rate, latency) ✅ SRE book | Instrument first; you cannot SLO what you cannot measure |
| **SLO** | The internal target, deliberately < 100% ✅ SRE book | The engineering contract; 100% is the wrong target ✅ |
| **SLA** | The external contract, looser than the SLO ✅ SRE framing | Leave headroom between what you promise and what you engineer |
| **Error budget** | 1 − SLO; the allowed failure in the window ✅ SRE book | Change and outages share one budget; spent budget → freeze launches ✅ Workbook |
| **RTO / RPO** | Recovery Time Objective / Recovery Point Objective — DR targets ✅ (openshift guide §3) | The business translation of the math; the investment case lives here |

### 2.5 The Math in Practice — Worked Numbers

The formula-level math of §2.1–§2.4 composes into the numbers that actually drive architecture decisions:

- **Series availability — the chain is only as strong as its weakest link.** When a request must pass through N components in series (LB → app → database → downstream), the combined availability is the *product* of the components' availabilities ✅-structural (the standard reliability-engineering result; the availability-math references present the same rule): 0.999 × 0.999 × 0.999 ≈ 0.997 — three components at three nines each deliver 99.7% end-to-end (~1.1 days/year down), not 99.9%. This is the mathematical reason §8's patterns exist: **you cannot stack nines by adding components; you must add redundancy and degradation so the chain does not multiply.**
- **Redundant (parallel) availability — the mirror image.** Two independent paths, each 99% available, deliver 1 − (0.01 × 0.01) = 99.99% — the redundancy math that justifies §5's failover and §6's routing: **availability is bought by making the failure of any single component non-fatal, not by making components infallible.**
- **The MTTR budget, concretely.** Holding 99.99% availability with an MTBF of 30 days requires MTTR ≤ ~4.3 minutes (§2.2's formula) — which is why the 60–120 s automatic failover of §5.2 *fits* a four-nines target while a 30-minute manual recovery procedure does not, no matter how good the runbook is.
- **The error budget, concretely.** A 99.95% SLO over a 30-day window buys 0.05% × 30 days ≈ **21.6 minutes** of budget per month ✅-arithmetic (the §2.3 framework). A single 10-minute user-visible maintenance window — the kind the pre-discipline estate ran "off-peak" — spends nearly half the month's entire budget. This is the exact number that converts §9's "the business refuses the maintenance window" from an opinion into an accounting fact.

The lesson across all four: **the availability math is not academic — it sets the budget every technique in §3–§8 exists to protect, and it exposes which "zero-downtime" claims are arithmetic and which are aspiration.**


---

## 3. The Deployment Strategies

### 3.1 The Deployment Problem — Why Change Is the Outage

The deployment is the moment the zero-downtime discipline is most likely to break: the old version must stop, the new version must start, and in between there is a gap where either nothing serves or two versions disagree. The strategies in this section are the canonical answers to that gap, each with a different trade-off between **risk exposure** (how many users see a broken release), **speed** (how fast the release reaches everyone), **cost** (how much idle capacity you pay for) and **rollback** (how fast you can retreat). All four deployment shapes are verified against the Martin Fowler bliki (✅ — [BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html), 1 March 2010; [CanaryRelease](https://martinfowler.com/bliki/CanaryRelease.html); the Continuous Delivery book lineage — Humble & Farley — that the blue-green bliki builds on) and the wider deployment-strategy literature; the feature-flag material is verified against Fowler's [FeatureFlag](https://martinfowler.com/bliki/FeatureFlag.html) bliki (29 October 2010, revised 2016 and 2023) and Pete Hodgson's companion article "Feature Toggles" ✅.

### 3.2 Rolling Deployment

**Mechanics:** the new version is deployed to a subset of instances (nodes, pods, servers) at a time, while the rest keep serving the old version; traffic shifts progressively as instances pass health checks, until every instance runs the new version (✅ standard — this is the default strategy of Kubernetes/OpenShift `Deployment` objects and of cloud auto-scaling groups; cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) for the platform-side mechanics — rolling updates, maxSurge/maxUnavailable, readiness gates).

**Zero-downtime properties:** no idle capacity is required (unlike blue-green) — instances are reused, so the cost is minimal; availability is preserved as long as **the two versions can coexist** and the load balancer only sends traffic to healthy instances (health checks, §6.3). The failure mode is the slow burn: a bad release spreads instance by instance, and if the health check is too shallow, the bad version is already half-deployed before anyone notices.

**Notes:** the version-coexistence requirement is the hidden tax — rolling is only safe when old and new can serve traffic side by side, which forces backward-compatible APIs and the expand-contract data discipline of §4. Rollback is a reverse roll — fast, but it re-exposes the same coexistence constraint.

### 3.3 Blue-Green Deployment

**Mechanics (✅ Fowler bliki, 1 March 2010):** two production environments, "as identical as possible" — blue live, green idle. The new release is fully deployed and tested in green; then **the router is switched so all incoming requests go to green**, and blue becomes idle — kept as the instant rollback target ("if anything goes wrong you switch the router back to your blue environment"). The two environments cycle roles release after release: green goes live, blue becomes the staging/rollback environment for the next release. Fowler credits the name to "some foggy combination of Dan North and Jez Humble" ✅.

**Zero-downtime properties:** the cut-over is a single routing switch, not an application restart — so the *deployment* gap is eliminated; and the rollback is the same switch in reverse, which is the fastest possible retreat. Fowler's bliki explicitly calls out the **rapid rollback** as the headline benefit ✅.

**Notes and the database caveat (✅ Fowler, verbatim in substance):** "Databases can often be a challenge with this technique, particularly when you need to change the schema to support a new version of the software. The trick is to separate the deployment of schema changes from application upgrades" — apply the database refactoring so the schema supports both versions, deploy it, verify, then deploy the new application version, and only later remove the old-version support. That is the expand-contract discipline of §4.2, named from the deployment side. The cost: you pay for two full environments (or two slices — Fowler notes the pair can be VMs, hardware, or zones of one environment with separate IPs ✅). Fowler also notes the same switch mechanism is exactly what a hot-standby needs — "this allows you to test your disaster-recovery procedure on every release" ✅ — a genuinely valuable side effect (§7's chaos mindset).

### 3.4 Canary Release

**Mechanics (✅ Fowler bliki — CanaryRelease):** "a canary release occurs when you roll out a new version of some software to a small subset of your user base to see if there are any problems before you make it available to everyone." The name comes from the coal-mine canary — the small, expendable early-warning system. In practice the canary is a small percentage of traffic (or a slice of users) routed to the new version while the majority stays on the old; the release proceeds (or rolls back) based on the canary's observed error rates and latency against the SLOs of §2.3.

**Zero-downtime properties:** unlike blue-green's binary switch, canary is *graduated exposure* — risk is metered: a broken release hurts 1–5% of traffic, not 100%. The canary is also the honest production test: it exercises the new version against real traffic, real data and real dependencies, which staging can never fully replicate.

**Notes:** canary requires **traffic-shaping capability** (weighted routing at the load balancer or service mesh) and **decisive metrics** (the canary's SLIs vs the baseline's — the SRE Workbook's canarying guidance ✅-structural). The two failure modes: a canary that is too small to be statistically meaningful, and a canary that stays small forever because nobody dares to ramp it — both are discipline failures, not tooling failures. Fowler's bliki also notes the canary is not a substitute for good testing; it is the last gate, not the first ✅.

### 3.5 A/B Testing — and How It Differs from Canary

**Mechanics:** A/B testing routes different *user cohorts* to two (or more) versions — usually deliberately different experiences — and compares **business outcomes** (conversion, engagement, revenue) rather than operational health. It is verified in the same lineage: Fowler's FeatureFlag bliki explicitly lists **"experiment toggles" for A/B testing** ✅, and the standard reading (⚠-structural, consistent across the deployment literature) is that A/B is a *product experiment*, while canary is a *release-validation technique*.

**The distinction that matters:** canary asks "is the new version safe to ship to everyone?" and routes a *traffic slice*; A/B asks "which version performs better for users?" and routes *defined cohorts* with instrumentation for outcome measurement. They share machinery (weighted routing, flags) and are often run together — a canary is frequently implemented as an A/B-style cohort split with operational metrics — but conflating them is a classic mistake: **using A/B for release safety exposes you to a statistically meaningless canary; using canary for product decisions gives you a business answer nobody asked for.** Both are zero-downtime in the sense that neither requires a service gap — the versions coexist and routing decides who sees what.

### 3.6 Feature Flags and Dark Launches

**Feature flags (✅ Fowler FeatureFlag bliki, 29 October 2010):** a configuration-driven switch in the application that decides whether a feature is visible or active. Fowler's taxonomy ✅: **release toggles** (hide a half-built feature until it is ready — the original use case), **experiment toggles** (A/B), **ops toggles** (runtime controls for operations staff), and **permissioning toggles** (feature access by user subset). The bliki's cautions ✅ are the real content: keep **toggle points** minimal (flag only the entry points, not every code path); **remove flags when the feature is stable** — "if you find that creating, maintaining, or removing the flags takes significant time, then that's a sign that you have too many toggle tests"; and watch for **accidental exposure** when a UI element is forgotten unwrapped. The deeper article is Pete Hodgson's "Feature Toggles" (✅ referenced by Fowler) — the standard reference for flag lifetimes and the "toggle debt" of stale flags.

**Dark launch:** deploying code to production **while it is invisible and inert** — the feature is live but switched off (release toggle off), or worse, exercised with shadow traffic (requests mirrored to the new service, responses discarded, outcomes compared) without any user seeing it. The verified anchor: the dark-launch *concept* is the direct application of release toggles (✅ Fowler's release-toggle mechanism), and the shadow-traffic variant is the industry-standard way to warm caches, validate against production data, and build confidence before the flag flips (⚠-structural — widely documented in the deployment literature; no single canonical primary source). Dark launch is the deployment strategy that most literally delivers "zero downtime": **the change ships with zero user exposure, and the exposure is itself a flag flip** — the §3.3 blue-green cut-over reduced to a configuration change.

### 3.7 The Strategies Table — Strategy / Mechanics / Notes

| Strategy | Mechanics | Notes |
|---|---|---|
| **Rolling** | Deploy new version instance-by-instance; load balancer shifts traffic as health checks pass ✅ standard (OpenShift/K8s default) | No idle capacity; requires version coexistence (backward-compatible APIs, expand-contract data); slow-burn failure mode; reverse-roll rollback |
| **Blue-green** | Two identical production environments; deploy to idle one; switch the router; old one kept for rollback ✅ Fowler bliki 2010 | Instant cut-over and instant rollback ✅; costs two environments; DB caveat — separate schema changes from app upgrades ✅ Fowler |
| **Canary** | Route a small traffic/user slice to the new version; proceed or roll back on observed SLIs ✅ Fowler bliki | Metered risk (1–5% blast radius); needs weighted routing + decisive metrics; not a substitute for testing ✅ |
| **A/B testing** | Cohort-split routing comparing business outcomes; experiment toggles ✅ Fowler | Product experiment, not release validation — don't conflate with canary; shares flag machinery |
| **Feature flags** | Config-driven toggles in the app: release/experiment/ops/permissioning ✅ Fowler | Ship dark, flip live; keep toggle points minimal; remove flags when stable ✅; flag debt is the tax |
| **Dark launch** | Deploy inert (release toggle off) or shadow-traffic the new version before exposure ⚠-structural (mechanism verified via Fowler release toggles) | Zero user exposure by construction; warms caches and proves the new path on production data before the flip |

### 3.8 Choosing a Strategy — and Combining Them

The selection logic, verified against the deployment-strategy literature ✅-structural (the comparison treatments verified this pass — the handbook/technical sources comparing blue-green, canary and rolling — are consistent with the Fowler originals):

- **Choose rolling** when capacity is tight (no idle environments), the release is backward-compatible by construction, and the platform gives you health-check-gated instance replacement — the default for routine service deployments on Kubernetes/OpenShift (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) for the platform mechanics).
- **Choose blue-green** when the release touches infrastructure or state (schema-coupled changes, config, version-skew-sensitive upgrades) and you want the *certainty* of a binary switch and a binary rollback — paying the price of two environments. Fowler's own framing ✅: the rapid rollback is the benefit; the database is the challenge, solved by separating schema changes from app upgrades (§3.3).
- **Choose canary** when the release's risk is *behavioral* — new logic, new dependency, new algorithm — and you have the traffic-shaping and metrics to make a small slice statistically meaningful. The SRE Workbook's canarying guidance (✅-structural, sre.google) treats the canary as the decision gate before full rollout.
- **Choose A/B** when the question is business outcome, not release safety — and remember it shares machinery with canary but answers a different question (§3.5).
- **Choose feature flags** when you need to decouple *shipping* from *exposure* entirely — the flag is the deployment, and the rollout is a configuration change (§3.6). This is the strategy that most literally delivers "zero downtime" and the one the Fowler canon warns to keep clean (toggle points minimal, flags removed when stable ✅).

**The combination stack — the way serious systems actually deploy:** a feature flag ships the code dark (§3.6); a canary ramps real traffic against it under SLO burn alerts (§3.4); the ramp completes and the flag flips to full exposure; and a blue-green pair (or a warm previous-version pool) sits behind the router as the instant retreat (§3.3). Each mechanism answers a different question — *is it deployed? is it safe? is it live? can we retreat?* — and the stack composes them without any single mechanism being forced to do a job it is bad at. That stack is exactly what §10's worked example deploys at estate scale.

---

## 4. The Data Layer

### 4.1 The Data Problem — Schema and State

Deployments (§3) are the easy half. The data layer is where zero-downtime ambitions go to die, because **the schema is shared state between versions that cannot be rolled back independently**: a new application version reading an old schema, or an old version writing to a new schema, breaks immediately. The canonical solution family is verified against the Fowler bliki's [ParallelChange](https://martinfowler.com/bliki/ParallelChange.html) pattern and the expand-contract literature (✅ — the Prisma Data Guide's expand-and-contract treatment; the Fowler ParallelChange bliki: "most database refactorings follow the parallel change pattern, where the migrate phase is the transition period between the original and the new schema, until all database access code has been updated to work with the new schema"). The additional problems — writes that must reach two systems, and events that must not be lost — have their own canonical answers: the **transactional outbox** (✅ Chris Richardson's microservices.io outbox pattern; widely verified) and **CDC** (✅ Debezium and the CDC literature, §4.4).

### 4.2 Expand-Contract (Parallel Change) Schema Migrations

**The pattern (✅ Fowler ParallelChange; ✅ Prisma Data Guide; ✅ the zero-downtime-migration literature):** every schema change proceeds in three phases, each independently deployable and independently reversible:

1. **Expand:** add the new schema element (column, table, index) **alongside** the old one. Nothing is removed; the old element keeps working. Old code and new code can both run — new code may start *writing* the new element while old code ignores it.
2. **Migrate (the parallel period):** backfill the new element from the old data; run both versions of the application against the dual schema; compare outcomes. This is the phase Fowler's ParallelChange bliki calls the transition period "until all database access code has been updated to work with the new schema" ✅. For a code-visible example: adding a `status_v2` column, writing both, reading `status_v2` in new code while old code reads `status`, reconciling the two in the migration window.
3. **Contract:** once no live code reads the old element, drop it — a second, separate deployment. The contract is the point of no return and must be the *last* step, weeks after the expand, never the same release ✅ (Fowler's blue-green bliki says the same from the deployment side: "when the upgrade has bedded down remove the database support for the old version" ✅).

**Why it is zero-downtime:** every step is additive or delayed-removal — no step requires stopping traffic, and every step is individually reversible (contract = restore the column; expand = drop the column). The discipline constraints, verified across the sources: **backward-compatible migrations only** (additive DDL; never rename-in-place — rename is expand-new + migrate + contract-old); **backfills are batched and throttled** to avoid lock storms; and **old and new code must coexist for the whole parallel period** — which is exactly the version-coexistence requirement §3.2 and §3.3 demanded. The Prisma guide's framing ✅: expand-contract is "a way to migrate data and clients to a new schema" without a big-bang cut-over; the matthewpalma.dev treatment ✅ adds the honest framing that "zero-downtime migrations are not a single tool... usually expressed as expand → migrate data → contract".

### 4.3 Dual-Writes — and the Transactional Outbox

**The dual-write problem (✅ verified — microservices.io, the outbox literature):** whenever two systems must both be updated — a database *and* a message broker (publish-event-on-write), or a legacy database *and* a new database during migration (§10) — the naive code path is two writes. If the second write fails after the first succeeds, the systems disagree: the event is lost (DB updated, broker missed) or duplicated (broker got it, DB rolled back). No ordering of the two writes removes the atomicity gap without a third mechanism.

**The transactional outbox (✅ Chris Richardson's outbox pattern, microservices.io; ✅ the outbox/CDC literature):** write the business record **and** the event into the *same database transaction* — the event goes to an `outbox` table in the same commit. A separate relay (a **polling publisher** or a **CDC capture**, §4.4) then delivers outbox rows to the broker, exactly-once-ish (at-least-once + consumer idempotency, §8.4). The pattern converts "two atomic writes to two systems" (impossible) into "one atomic write to one system + a durable, retryable relay" (possible). Verified framing across the sources ✅: the outbox solves the dual-write problem by making the *database* the source of truth for the event, and the broker a derived consumer.

**The migration-specific variant — the dual-write with reconciliation:** when migrating state between two databases (legacy → new, §10), the dual-write is *both* systems accepting writes, with the order and failure handling governed by a reconciliation job that detects and repairs divergence. This is the data-layer shape of the shadow-traffic pattern (§3.6): the migration's correctness is judged by a comparison pipeline, not by hope. Cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) for the at-least-once/durability framing and [late_arriving_data_guide.md](late_arriving_data_guide.md) for the data-lag consequences when the two sides are eventually consistent.

### 4.4 Change Data Capture (CDC)

**The mechanism (✅ verified — Debezium and the CDC literature):** CDC reads the **database transaction log (WAL/binlog)** and emits the committed changes as an event stream — inserts, updates, deletes, with before/after images. Because it reads the log, not the application, CDC is: **non-invasive** (no application code change), **accurate** (it sees exactly what committed), and **low-latency** (events stream as the log advances). Debezium is the canonical open-source implementation (✅ verified — the CDC/outbox literature consistently centres Debezium + Kafka); the log-tailing approach is the transaction-log-tailing alternative to the polling publisher in the outbox pattern (✅ microservices.io).

**Uses in zero-downtime design:** (1) the **outbox relay** — CDC tails the outbox table and publishes to the broker with near-zero polling delay (§4.3); (2) **schema-change propagation** — a CDC stream carries the expand-contract migration's writes to downstream consumers, letting *data* migrate ahead of *code*; (3) **database-to-database replication** — the §10 migration's live copy of legacy writes into the new database is CDC at its purest; (4) **event sourcing of the operational state** — the NETS-style "the stream is the state" pattern (cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §9.2's D7). The caveats, flagged honestly: CDC consumers must be **idempotent** (log replay can redeliver — §8.4), the log must be **retained** long enough for the slowest consumer, and CDC is not a substitute for application-level validation — it is a transport, not a contract ✅-structural.

### 4.5 The Data Table

| Technique | What it does (verified) | Zero-downtime role | Key constraint |
|---|---|---|---|
| **Expand-contract / parallel change** | Expand (additive change) → migrate (dual-run, backfill) → contract (drop old) ✅ Fowler ParallelChange, Prisma | Every schema change becomes additive and reversible; old + new code coexist | Backward-compatible DDL only; contract is a separate, later deployment ✅ |
| **Backward-compatible migrations** | Additive columns/tables/indexes; no in-place renames ✅ structural | Old and new versions read the same schema | Never rename-in-place; batch and throttle backfills |
| **Dual-writes** | Write two systems from one code path ✅ structural | Migration-time coexistence of legacy + new stores | The atomicity gap — needs outbox or reconciliation |
| **Transactional outbox** | Event written in the same DB transaction as the record; relay delivers ✅ Richardson/microservices.io | Solves the dual-write problem; durable event delivery | Relay is at-least-once → consumers must be idempotent |
| **CDC** | Transaction-log tailing → event stream ✅ Debezium/CDC literature | Outbox relay; live DB-to-DB replication; schema propagation | Log retention; idempotent consumers; transport not contract |

### 4.6 The Schema-Migration Playbook — Concrete Rules

The §4.2 pattern, reduced to the rules that every team actually follows (✅-structural — the consistent distilled practice across the verified sources: the Fowler bliki, the Prisma Data Guide, and the zero-downtime-migration literature):

1. **Additive DDL only, always.** New column, new table, new index — never alter-in-place, never rename-in-place. A rename is expand (add the new name) → migrate (dual-write and backfill) → contract (drop the old name) ✅.
2. **Make new columns nullable or defaulted.** A NOT NULL column with no default breaks every old-version insert the moment it is added — the classic expand-phase outage. Add nullable; backfill; then tighten the constraint in the contract phase ✅-structural.
3. **Never backfill in one transaction.** Batched, throttled, resumable backfills with a checkpoint — a single UPDATE of 100M rows is a lock storm and a replication lag spike (and a §2.1 budget spender), not a migration step.
4. **The application deploys are the migration's heartbeat.** Each expand/migrate/contract step pairs with a deploy of the code that uses it; the sequence is *schema → code → code → schema*, never both schemas in one night (Fowler's blue-green bliki says it from the deployment side: apply the schema refactoring, verify, deploy the app, then remove the old support ✅).
5. **The contract is a separate release, weeks later.** The drop is the only irreversible step — it must be the most-rehearsed, most-boring step in the sequence, gated by evidence that no live code path touches the old element ✅.
6. **Every step reversible, every step observable.** The migration runs under the §2.3 error-budget monitoring; a step that spends budget is rolled back — which is only possible because steps 1–5 made each step individually reversible.

The playbook is the data-layer twin of §3.8's combination stack: no single rule is exotic, and the discipline is in the *ordering* — which is why §10's worked example treats the data plan as the schedule (§10.3 lesson 4).

---

## 5. The Stateful Systems

### 5.1 Why State Is the Hard Part

Stateless application instances can be killed, replaced and rebalanced freely — the platform does it for you (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §4–§6). **State is the part that cannot be recreated from a template**, so the availability of a stateful system is the availability of its *consensus*: where is the authoritative copy of the data, who may write, and what happens when the node holding it fails. The three mechanisms in this section — failover, quorum, multi-AZ — are the canonical answers, each verified below.

### 5.2 Database Failover

**The mechanism (✅ AWS-documented):** a primary database instance is **synchronously replicated** to a standby, and on failure the standby is promoted automatically. The AWS RDS Multi-AZ documentation (✅ docs.aws.amazon.com, verified this pass) states the properties precisely: "a primary DB Instance is synchronously replicated to a standby instance in a different Availability Zone for automatic failover," and "failover times are typically 60–120 seconds" ✅. Azure's failover groups (✅ learn.microsoft.com) provide the managed cross-region analogue. The key numbers: **synchronous replication means zero data loss on failover (RPO ≈ 0)** — the standby has every committed transaction — and the 60–120 s promotion window is the RTO the application must be designed to ride out (retries, queues, connection-pool reconfiguration — §8.3 and §6.3).

**The architect's reading:** failover moves the *role*, not the data — and role moves are exactly where split-brain (two primaries) becomes possible, which is why automatic failover must be driven by **quorum-based election, not by "the primary looks dead"** (§5.3). Failover also has a **planned** form — a switchover — which is the stateful analogue of the blue-green router flip (§3.3): you fail over deliberately during low traffic to rehearse the mechanism (the GameDay mindset, §7.3) so the unplanned version is boring.

### 5.3 Quorum — the Consensus Core

**The mechanism (✅ verified — the consensus literature: Raft and Paxos, etcd's majority-based operation):** replicated state machines (etcd, ZooKeeper, Consul, the control planes of every HA database and of Kubernetes itself) decide by **majority**: a write is committed when a quorum of nodes — **more than half (N/2 + 1) of the cluster** — acknowledges it, and a new leader is elected only when a quorum agrees. Quorum is what makes split-brain *impossible by construction*: at most one partition can hold a majority, so at most one leader can exist.

**The trade-off, verified ✅ (etcd/Raft documentation and the distributed-systems canon; cross-ref [ddia_study_companion_guide.md](ddia_study_companion_guide.md) for the replication chapters):** quorum buys consistency at the price of availability under partition — a 3-node cluster tolerates **1** node failure (2 of 3 still form a majority); a 5-node cluster tolerates **2**. Lose more than the tolerated number and the cluster stops accepting writes rather than risk divergence — it **fails closed**. This is the honest heart of the zero-downtime illusion in the stateful layer: **the consensus core cannot serve during a quorum-loss partition; the design goal is to make that window as small and as rare as the math allows** (odd node counts, 3 or 5; cross-AZ placement so a single AZ failure never costs the majority — §5.4).

### 5.4 Multi-AZ — the Failure-Domain Frame

**The mechanism (✅ AWS RDS Multi-AZ docs; ✅ the OpenShift guide's topology section):** an availability zone (AZ) is a physically isolated failure domain — separate power, cooling, network — inside a region. Multi-AZ placement spreads replicas across AZs so that **a single-AZ failure cannot take out the quorum or the standby**: the RDS pattern is primary in one AZ, synchronously replicated standby in another (✅ docs); the Kubernetes/etcd pattern is control-plane replicas across three AZs (✅ [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §4.2 — the recommended production topology). The frame's rule: **failure domains must be respected by the placement of every redundant component** — three replicas in one AZ are a single point of failure with extra steps; three AZs with one replica each survive an AZ loss. Cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) for the DC-level frame (the Singapore estate — 70+ data centres, Tier III norm ⚠) and the regional/DR tier above AZs.

### 5.5 The Stateful Table

| Mechanism | What it does (verified) | Availability property | The honest limit |
|---|---|---|---|
| **Database failover** | Synchronous replication to standby; automatic promotion on failure ✅ AWS RDS docs | RPO ≈ 0 (sync replication); automatic recovery | Promotion window 60–120 s ✅ — the app must ride it out (retries, queues) |
| **Quorum** | Majority (N/2+1) commits and elections; Raft/Paxos consensus ✅ etcd/Raft canon | Split-brain impossible by construction; 3 nodes → 1 failure, 5 nodes → 2 ✅ | Fails closed on quorum loss — availability sacrificed for consistency |
| **Multi-AZ** | Replicas placed in physically isolated AZs ✅ AWS docs, OpenShift guide §4.2 | One AZ failure cannot take the quorum/standby | Region loss still hurts — that is the DR tier's job (cross-ref openshift guide §3) |
| **Planned switchover** | Deliberate role flip during low traffic ✅ structural (the planned form of failover) | Rehearses the unplanned path; the blue-green analogue for state | Only as valuable as the rehearsal discipline (§7.3) |

### 5.6 The Stateful Decision Map

The stateful choices reduce to four questions, each with a verified-mechanism answer from §5.2–§5.4:

1. **How much data loss is acceptable?** If RPO = 0 (the payment-instruction case of §9–§10), the replication must be **synchronous** — the primary waits for the standby's ack before committing ✅ (AWS RDS Multi-AZ's documented property). If a small RPO is acceptable (a cache, a session store, a reporting replica), **asynchronous** replication buys lower write latency at the price of losing the tail of writes on failover ✅-structural (the standard replication trade-off; cross-ref [ddia_study_companion_guide.md](ddia_study_companion_guide.md) for the replication chapters).
2. **How many simultaneous failures must be tolerated?** One node → 3-node quorum (tolerates 1); two nodes → 5-node quorum (tolerates 2) ✅ (the §5.3 majority math). The answer sets the cluster size — and the rule "odd numbers only" falls out of the majority definition.
3. **Which failure domains exist?** If a whole AZ can vanish, the replicas must not share an AZ — the §5.4 placement rule. The count of AZs you spread across must exceed the number of failures you tolerate, or the quorum math and the placement math contradict each other.
4. **Who decides the primary is dead?** Not the primary (it cannot be trusted to report its own death — the split-brain trap); the **quorum** decides, and the losing side must be fenced (denied writes) so a partitioned old-primary cannot accept writes while a new primary exists ✅-structural (the fencing/lease mechanism from the consensus canon; cross-ref the OpenShift guide's etcd framing). The fencing step is what makes automatic failover safe rather than reckless — and why §7.3's GameDays must rehearse exactly this scenario.

The map's output is a concrete configuration — *synchronous replication, 3-node quorum, spread across 3 AZs, fencing enforced, failover rehearsed* — which is the stateful core of every serious zero-downtime estate, and precisely what §10.2's Phase 3 specifies.

---

## 6. The Infrastructure

### 6.1 The Infrastructure Stack — Routing Around Failure

The infrastructure layer's zero-downtime job is **routing**: keep each request flowing to a *healthy* instance of a *healthy* service, and when something breaks, route around it before the user notices. The three mechanisms — load balancers, health checks, DNS failover — form a ladder of increasing scope: the load balancer handles instance-level failure in milliseconds, health checks feed every layer's decisions, and DNS failover handles site/region-level failure at TTL speed. All three are verified below against the AWS documentation and the standard infrastructure canon.

### 6.2 Load Balancers

**The mechanism (✅ standard, AWS-documented class):** a load balancer sits in front of a pool of instances and distributes traffic — L4 (TCP/UDP, connection-level) or L7 (HTTP, content-aware routing). Its zero-downtime properties come from two features: **draining** (stop sending new connections to an instance marked for removal while letting in-flight requests finish — the graceful-shutdown handshake that makes rolling deployments §3.2 seamless) and **health-check-driven membership** (an instance failing its health check is removed from the pool automatically — the balancer *is* the first line of failure routing). Verified anchors ✅: AWS ELB/ALB health-check and connection-draining behavior (AWS docs class), the Kubernetes/OpenShift Service + readiness-probe mechanism (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §6 — probes as the platform's health-check contract).

### 6.3 Health Checks

**The mechanism (✅ standard; ✅ OpenShift guide cross-ref):** a health check is a probe the orchestrator or balancer runs against an instance — **liveness** (is the process alive? restart if not) vs **readiness** (is the instance able to serve traffic? remove from the pool if not) vs **startup** (is it still initializing? — the Kubernetes variant). The distinction is the zero-downtime core: a process can be *alive* and *unready* (warming caches, draining, degraded) — and the routing layer must know the difference. Verified anchors: the Kubernetes probe contract (✅, cross-ref the OpenShift guide's probes section), the AWS target-group health-check configuration (✅ AWS docs). The design cautions, flagged honestly: **shallow checks** (TCP connect only) miss the failure modes that matter (the process accepts connections but returns 500s — the classic "half-dead" state), while **overly aggressive checks** (deep dependency probes) can cascade — every instance fails its check because the *database* is slow, and the balancer empties the pool precisely when you need it full (the health-check cascade; ⚠-structural, well-documented failure mode). The discipline: check what the *user* would experience, with a dependency-aware but not dependency-blinded probe.

### 6.4 DNS Failover

**The mechanism (✅ AWS Route 53 docs, verified this pass):** DNS records point at a service's endpoints; **health checks attached to the records** detect failure and the DNS service stops returning the failed endpoint, or switches the record to the standby. AWS documents the two shapes ✅: **active-active** (all healthy endpoints served, traffic distributed — the Route 53 "any routing policy other than failover") and **active-passive** (the failover routing policy — primary served while healthy, standby on failure). The constraint is the honest one: **DNS failover is TTL-bound** — clients cache DNS answers for the record's TTL, so failover takes at least the TTL (commonly 30–300 s) even after the health check fires. DNS failover is therefore the **coarse, last-resort rung** — right for site/region loss, wrong for instance failure (that is the load balancer's milliseconds). Cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) for the DC-level reality and the GSLB (global server load balancing) variant ⚠-structural.

### 6.5 The Infra Table

| Mechanism | What it does (verified) | Scope / speed | The honest limit |
|---|---|---|---|
| **Load balancer** | Distributes traffic across a pool; drains and removes unhealthy instances ✅ AWS/ELB class | Instance-level; milliseconds | L7 smarts add latency; shallow health checks miss half-dead instances |
| **Health checks** | Liveness vs readiness vs startup probes feeding routing decisions ✅ K8s/OpenShift probes, AWS target groups | Feeds every layer | Shallow = blind; too deep = cascade — check what the user experiences |
| **DNS failover** | Health-checked records; active-active or active-passive routing ✅ Route 53 docs | Site/region-level; TTL-bound (30–300 s typical) | Too slow for instance failure — the last-resort rung, not the first |

### 6.6 The Routing Ladder — One Request's Journey

The three mechanisms of §6.2–§6.4 are not alternatives — they are **rungs of one ladder**, and a request climbs down it as failures escalate (✅-structural synthesis of the verified mechanisms; the time scales are the documented properties):

1. **The DNS rung (region/site level, seconds-to-minutes).** The client resolves the service name; Route 53-class health-checked records return only healthy endpoints, active-passive failing over to the standby site when the primary's checks fail ✅ (§6.4). This rung only moves when a *site* is at stake — its TTL cost makes it the wrong tool for anything smaller.
2. **The load-balancer rung (instance level, milliseconds).** The resolved endpoint is a balancer; it holds the pool of instances, drains the ones marked for removal, and stops sending traffic to instances that fail their health checks ✅ (§6.2). Instance death, node drain, rolling deployments — all handled here, invisibly, per-request.
3. **The application rung (dependency level, per-call).** Inside the instance, the §8 patterns take over: the timeout bounds the call, the retry-with-jitter rides through transient windows, the circuit breaker fails fast when the dependency is genuinely down, and the bulkhead keeps the damage contained. **This rung is what makes the other two invisible** — the retry is what hides the 90-second database failover of §5.2 from the user.
4. **The data rung (state level).** The request's write lands on a primary that fails over via quorum (§5.3); the connection pool re-establishes; the idempotency key ensures the retried request does not double-apply (§8.4).

The ladder is the whole discipline in miniature: **each rung handles the failures the rung above it is too slow or too coarse to catch, and the composition — not any single mechanism — is what makes a 5-minute-per-year budget achievable** (§2.1). Cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) for the DC-level frame beneath the ladder, and the OpenShift guide for the platform's own probe/routing rungs.


---

## 7. The Chaos Engineering

### 7.1 The Chaos Monkey — the Origin

**The origin (✅ verified — Wikipedia's chaos-engineering article dates the tool; the Netflix engineering blog and the Principles of Chaos corroborate the framing):** Netflix created **Chaos Monkey** around 2011 as it migrated to AWS, and the tool "intentionally disabl[es] computers in Netflix's production network to test how the remaining systems respond to the outage" ✅ (Wikipedia; the tool's own README ✅: "Chaos Monkey randomly terminates virtual machine instances and containers that run inside of your production environment. Exposing engineers to failures more frequently incentivizes them to build resilient services"). The insight is the famous one: **if you do not cause failures yourself, production will cause them for you — on its own schedule; the Monkey simply makes failure a routine event the architecture must survive.** Chaos Monkey grew into the **Simian Army** — a family of failure-injection tools (Latency Monkey, Conformity Monkey, etc.) ✅-structural (documented in the Netflix tech-blog era and the chaos literature). The deeper point, verified ✅ against the Principles of Chaos: chaos engineering is not vandalism — "breaking things on purpose" is only valid when it is a *controlled experiment* with a hypothesis and observability (§7.2).

### 7.2 The Principles of Chaos

**The Principles of Chaos** (✅ — the principlesofchaos.org document, the canonical statement produced by the chaos-engineering community with Netflix's participation; corroborated by the 2026 chaos-engineering literature verified this pass) state the practice as four principles:

1. **Build a hypothesis around steady-state behavior** — define what "normal" looks like (e.g. the SLOs of §2.3: error rate < X, p99 latency < Y) *before* injecting failure; the experiment is "does the system still hold steady state under this perturbation?"
2. **Vary real-world events** — inject the failures that actually happen: instance death, AZ loss, network partition, latency, certificate expiry — not theatrical ones.
3. **Run experiments in production** — staging cannot reproduce production's traffic, data and topology; the discipline's distinctive claim is that production is the only truthful environment (the same argument as canary, §3.4).
4. **Automate experiments to run continuously** — chaos as a *permanent* practice (the Monkey runs continuously), not a quarterly exercise.

The companion concept from the same canon ✅: **blast radius** — the discipline of bounding how much of the system a single experiment may disturb (a percentage of instances, a single AZ, a specific service) and ramping from small to large. ⚠-structural note: the *adoption* claim sometimes made for chaos engineering in the industry at large (specific adoption percentages) is not verifiable from a single authoritative source and is flagged rather than repeated here.

### 7.3 GameDays — the Deliberate Drill

**The practice (✅ — the GameDay is documented in the Google SRE book and the SRE Workbook's game-day chapters; the sre.google/workbook material verified this pass covers the practice's shape):** a GameDay is a **scheduled, bounded, rehearsed failure drill** — a team deliberately takes down a dependency, fails over a database, kills a region's traffic, or replays a past incident, inside a defined window, with observers, runbooks and a post-exercise review. Unlike the Monkey's continuous automation, the GameDay is the *deliberate* form: it tests **people and process** (runbooks, decision-making under time pressure, communication) as much as the architecture. The SRE framing ✅: GameDays convert the unplanned into the rehearsed — the same logic as Fowler's blue-green note that the switch mechanism "allows you to test your disaster-recovery procedure on every release" (✅ §3.3). The banking regulator's version of the GameDay is the **BCM/DR exercise** — MAS Notice 637-style business-continuity testing that banks must run and evidence (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §3 for the regulatory frame; the NETS guide §7 for the switch's stand-in/DR chain).

### 7.4 The Chaos Table

| Practice | What it is (verified) | Zero-downtime role | Discipline constraint |
|---|---|---|---|
| **Chaos Monkey** | Randomly terminates production instances/containers ✅ Netflix tool, Wikipedia 2011 origin | Makes instance failure routine so the architecture must survive it; proves §5–§6 automatic recovery | Random ≠ uncontrolled — the system must hold steady state under the perturbation ✅ |
| **Simian Army** | The family of failure-injection tools around the Monkey ⚠-structural | Widens the injected-failure catalogue (latency, conformity, etc.) | Same hypothesis discipline as the Monkey |
| **Principles of Chaos** | Hypothesis → vary real events → run in production → automate ✅ principlesofchaos.org | The *method* that turns breaking things into an experiment | Steady-state hypothesis first; blast radius bounded and ramped ✅ |
| **GameDay** | Scheduled, bounded, rehearsed failure drill ✅ SRE book / Workbook | Tests people + process + architecture; converts unplanned into rehearsed | Post-exercise review is mandatory; no GameDay without a runbook |
| **BCM/DR exercise** | The regulated drill banks must run (MAS-style) ✅ (openshift guide §3 frame) | The GameDay with regulatory teeth | Evidence of the exercise is the deliverable |

### 7.5 Building the Practice — the Blast-Radius Ramp

A chaos practice is built in stages, each with a bounded blast radius (✅-structural synthesis of the Principles of Chaos and the 2026 chaos-engineering literature verified this pass — the four principles of §7.2 and the blast-radius concept are the verified anchors):

1. **Stage 0 — GameDays before Monkeys (months 1–3).** The first drills are scheduled, bounded and manual (§7.3): fail over the database deliberately, kill a node, replay a past incident. This stage builds the runbooks and the observability — and it is where the §2.3 SLOs prove their worth, because the drill's pass/fail criterion is "did steady state hold?" ✅ (Principle 1).
2. **Stage 1 — one service, small percentages.** The first automated chaos targets a *single* service with a tiny blast radius (one instance of a pool of twenty — the Monkey's own random-termination shape ✅, bounded by the pool). The hypothesis is written first; the experiment is a yes/no question about an SLI, not a vague "did it survive?" ✅ (Principle 1 again).
3. **Stage 2 — the dependency and infrastructure failures.** Inject the real-world events of Principle 2 ✅: network partitions, latency spikes, certificate expiry, AZ loss — against the services whose failure would matter most (the stateful core of §5, the routing of §6).
4. **Stage 3 — continuous, in production, automated.** The practice runs on a schedule (Principle 4 ✅), in production (Principle 3 ✅), with the blast radius ramped gradually from instance → service → AZ, each ramp gated by the previous stage's clean results.

The honest notes: (1) **production chaos requires the production-equivalent of consent** — the §7.2 principles and the banking context (§9.1's evidenced-resilience obligations) both argue for starting with GameDays and shadow environments before touching live traffic; (2) **adoption claims** — specific industry adoption statistics for chaos engineering ⚠ could not be verified from a single authoritative source and are deliberately not quoted here; and (3) the practice's value is precisely that it **converts the §11 "zero-downtime illusion" from hope into rehearsal** — the illusion holds because the failure has been seen before, on purpose.

---

## 8. The Resilience Patterns

### 8.1 Circuit Breakers

**The pattern (✅ Fowler bliki, 6 March 2014 — verified live this pass):** Fowler's CircuitBreaker bliki is explicit about both the origin and the mechanics: **Michael Nygard popularized the pattern in *Release It!*** — the book that gave the industry its failure-mode vocabulary — to prevent the "cascading failures across multiple systems" that occur when "many callers on an unresponsive supplier... run out of critical resources." The mechanism ✅: wrap the protected call in a breaker object that monitors failures; "once the failures reach a certain threshold, the circuit breaker trips, and all further calls... return with an error, without the protected call being made at all"; and the self-resetting variant tries the protected call again after an interval and closes the circuit on success (the **half-open** state, though Fowler's bliki describes the behavior — "the breaker itself detect if the underlying calls are working again" ✅). Fowler's "Further Reading" ✅ names Netflix's **Hystrix** ("a sophisticated tool for dealing with latency and fault tolerance for distributed systems. It includes an implementation of the circuit breaker pattern with the thread pool limit") — the production-grade reference implementation of the era.

**Why it is a zero-downtime pattern:** without a breaker, a failing dependency converts a localized problem into a global one — every caller's threads block on timeouts until the whole service is exhausted (the cascading failure Fowler describes ✅). The breaker **fails fast** (returns immediately when open), preserving the caller's capacity for the healthy parts of its workload — the stateful-system equivalent of "don't let the quorum loss take down the read path" (§5.3). The design constants ✅: threshold, timeout, reset interval, and a monitor alert when the breaker trips.

### 8.2 Bulkheads

**The pattern (✅ Nygard's *Release It!* — the bulkhead is one of its signature patterns; ⚠-structural for the exact framing, consistent across the reliability literature):** named after ship bulkheads — the watertight compartments that keep a hull breach from sinking the whole vessel — the bulkhead pattern **isolates failure by partitioning resources**: each dependency (or customer class, or workload type) gets its own bounded pool of threads/connections/queues, so one dependency exhausting its pool cannot starve the others. The canonical production example is again Netflix's Hystrix ✅ (Fowler's bliki quote above: "the thread pool limit" — Hystrix's per-dependency thread pools are bulkheads implemented in code). In container platforms the same idea appears as **resource quotas and limit ranges per namespace/tenant** (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) — the platform-side bulkhead). The pattern's relation to the circuit breaker ✅-structural: the bulkhead bounds *how much* a failure can consume; the breaker bounds *how long* the system keeps trying; they are complementary, not alternative.

### 8.3 Retries with Backoff

**The pattern (✅ verified — the AWS Architecture Blog's canonical treatment, "Timeouts, retries, and backoff with jitter" (Marc Brooker, 2015), and the standard reliability literature):** transient failures (a connection dropped, a 503 during a deploy, a failover window — §5.2's 60–120 s) are worth retrying; permanent failures are not. The mechanics: **exponential backoff** (wait 1s, 2s, 4s, 8s... between attempts, capped) and — the Brooker insight ✅ — **jitter** (randomize each wait), because without jitter, synchronized retry storms (the thundering herd) can take down the very service being retried, and can turn a failover into a self-inflicted outage. The zero-downtime coupling ✅-structural: retries are what let applications ride through the failover and deployment windows that §3, §5 and §6 create but cannot eliminate; and **retries are only safe when the operation is idempotent** (§8.4) — otherwise a retried payment authorizes twice, which in banking is not an abstraction. The caps are the discipline: max attempts, total deadline, and "retry only what is transient" — a 4xx is never retried; a 5xx or network error is, with backoff.

### 8.4 Idempotency

**The pattern (✅ verified — the idempotency literature: Stripe's idempotency-keys design is the canonical production reference; the messaging literature's at-least-once + idempotent-consumer framing — cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) — is the queue-side version):** an operation is idempotent when executing it N times has the same effect as executing it once. The mechanisms: **idempotency keys** (the client sends a unique key; the server deduplicates on it — Stripe's `Idempotency-Key` header pattern), **natural-key uniqueness constraints** (the database rejects the duplicate insert), and **state-machine deduplication** (an event processor checks "have I already processed this event ID?" before applying — the outbox/CDC consumer contract of §4.4). The zero-downtime role ✅-structural: idempotency is the *enabler* of every other pattern in this section — retries (§8.3), at-least-once messaging (cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)), dual-write reconciliation (§4.3), and the NETS-style reversal discipline where "reversals that are idempotent and retried" are a design invariant (cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §10's pattern list). Without idempotency, every retry is a potential duplicate; with it, retries become free.

### 8.5 Graceful Degradation

**The pattern (✅ SRE-book class; ⚠-structural for the exact taxonomy — consistent across the reliability literature):** when a component cannot deliver its full function, it should deliver **a reduced function rather than nothing** — serve cached or stale data instead of 500s, disable the non-critical feature while protecting the critical path (read-only mode during a migration, the recommendations carousel dropping while the search box stays up), or fail-open where the risk allows and fail-closed where it does not. The SRE framing ✅: availability is defined by the SLO on the *user-visible* function (§2.3), so degradation that protects the SLO'd path *is* availability. The banking flavour (cross-ref §9): a payment switch in **stand-in mode** authorizes within risk limits while the issuer link is down rather than declining everything — the NETS guide's §7.2 chain (cross-ref [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md)); the trading system degrades to read-only market data during a feed outage rather than showing stale prices as executable (cross-ref [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) — the trading-day availability frame). The design discipline ⚠-structural: **decide in advance** what degrades, what never degrades, and how the degraded state is signalled — improvisation during an incident is how degradation becomes outage.

### 8.6 The Patterns Table

| Pattern | What it does (verified) | Zero-downtime role | Key constraint |
|---|---|---|---|
| **Circuit breaker** | Trips after a failure threshold; fails fast; half-open self-reset ✅ Fowler bliki (Nygard's *Release It!*) | Stops cascading failures; preserves capacity for healthy work | Thresholds + timeout + reset interval; alert on trip ✅ |
| **Bulkhead** | Per-dependency bounded resource pools ✅ Nygard; Hystrix thread-pool limit ✅ Fowler quote | Bounds how much a failure can consume | Pools sized per dependency; complements the breaker ✅ |
| **Retries + backoff + jitter** | Retry transients with exponential backoff, randomized (jitter) ✅ AWS Architecture Blog (Brooker) | Rides through failover/deploy windows (§5.2, §3) | Only transient failures; capped; only safe with idempotency |
| **Idempotency** | N executions = 1 execution; keys/uniqueness/dedup ✅ Stripe pattern; messaging canon | Makes retries and at-least-once delivery safe | Key generation, storage and expiry need design; the enabler pattern |
| **Graceful degradation** | Reduced function instead of none; stand-in, read-only, cached ✅ SRE class; NETS stand-in cross-ref | Protects the SLO'd path; degradation *is* availability | Decide the degradation policy in advance; fail-open vs fail-closed by risk |

### 8.7 The Pattern Stack — How They Compose

The §8 patterns are usually described one at a time and always deployed as a stack — each one assumes the others (✅-structural synthesis of the verified pattern canon; the composition is the standard reliability-engineering reading of Nygard/Fowler/Brooker). The canonical call path, in order:

1. **Timeout** — the outermost bound: every remote call gets a deadline, because a call without a timeout is an unbounded resource lease (✅-structural, the reliability canon's most-repeated rule; Fowler's CircuitBreaker bliki opens with the "hang without a response until some timeout limit" failure ✅).
2. **Retry with backoff and jitter** — transient failures (the §5.2 failover window, the §3 deploy drain) are retried, exponentially backed off, jittered against thundering herds ✅ (§8.3).
3. **Circuit breaker** — when retries stop helping (the failure is not transient), the breaker trips and the call fails fast, protecting the caller's threads ✅ (§8.1).
4. **Bulkhead** — the caller's resources are partitioned per dependency, so the tripped dependency cannot starve the healthy ones ✅ (§8.2).
5. **Graceful degradation** — with the call failing fast, the *user-facing* layer serves the reduced path — cached data, stand-in authorization, read-only mode ✅ (§8.5).
6. **Idempotency underneath everything** — every retry, every replay, every dual-write comparison is safe only because the operation carries the key that makes N executions equal one ✅ (§8.4).

The stack is why the patterns appear *together* in every serious reliability review: a breaker without a retry budget trips on blips; retries without idempotency duplicate money movements; a bulkhead without a breaker lets a slow (not dead) dependency hold its pool open forever. And the stack is the §8-shaped answer to §2.5's series-availability math: **the patterns break the multiplicative chain — a failure in one component degrades, rather than multiplies, the whole.**

---

## 9. The Zero-Downtime in Banking

### 9.1 The 24-7 Payments Reality

Banking is where the zero-downtime discipline stops being an engineering preference and becomes a regulatory and commercial fact. The verified anchors from the repo's own cluster: **FAST — Singapore's real-time interbank rail — runs 24/7** (✅ the NETS guide's §3.2 — "the FAST operations — real-time 24/7"; FAST is ISO 20022-based and BCS-operated, ✅ the NETS guide's verified ledger); the **NETS switch authorizes in real time against the issuing banks** — every tap at the counter is a synchronous decision that cannot wait for a maintenance window (✅ NETS guide §1–§2); and the estate is designed with **stand-in processing, store-and-forward, and active/active dual-site resilience** so that even issuer outages and site losses do not stop the counter (✅ NETS guide §7.2 — the resilience chain). The regulatory frame ✅ (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §1 and §3): MAS TRM (Technology Risk Management), MAS Notice 637 (BCM), and BCBS 239 (data lineage and risk reporting) make resilience and recoverability *evidenced obligations* — the bank must prove its RTO/RPO, run its BCM exercises (§7.3's GameDay with teeth), and keep the payments estate available through change.

### 9.2 The Switch That Never Sleeps — the NETS Cross-Ref

The sibling guide's final word is the industry's own summary of §9: **"the switch that never sleeps"** (✅ [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §10). The NETS guide's patterns are this guide's patterns wearing a bank's clothes: **dual-message shape** (online authorization + batched clearing) = the deployment/data discipline of separating the synchronous path from the batch path; **correlation as identity** (STAN/RRN through authorize→clear→settle) = the idempotency key of §8.4 made into a national-payments invariant; **reversals that are idempotent and retried** = §8.3+§8.4 in production; **store-and-forward in two flavours** = the queue-behind-the-failure pattern of §8.5's stand-in; **active/active resilience** = §5.4's multi-AZ carried to dual sites; and the **modernization rule** — "coexistence is a feature, not a phase" (✅ NETS guide §9.4) — = §4's expand-contract applied at estate scale: ISO 8583 and ISO 20022 in one estate *by design*, with the middleware as the translation layer, because a national switch cannot be switched off to be upgraded. The NETS guide's worked example (§9) is the §10 of this guide in the payments domain: **dual-run from day one, shadow settlement, one-issuer-at-a-time cutover, every slice reversible** — the exact deployment/state/data pattern stack of §3–§5.

### 9.3 What Zero Downtime Means Inside a Bank

Inside a bank (cross-ref [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md), and the house guide [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md)), the discipline acquires bank-specific shapes:

- **The trading day is the availability window.** Capital-markets systems run to the market's calendar — Asia open, Europe, US — and the "night" is a real but short maintenance corridor that must never be assumed (cross-ref [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) — the trading-day availability frame; weekend and holiday calendars differ by market and by product).
- **The payment switch has no window at all.** FAST-class rails are 24/7/365; the counter, the ATM and the PayNow transfer do not observe bank holidays ✅ (NETS guide §10: "No bank holiday, no maintenance night, no outage at the issuer changes the requirement").
- **Batch is the friend of the window.** The parts of banking that *are* batch — end-of-day processing, settlement runs, statement generation — create natural low-traffic corridors, but the modernisation direction (ISO 20022, event-driven settlement — ✅ NETS guide §8) is precisely the compression of those corridors toward real time, which *removes* the windows and raises the zero-downtime bar.
- **The regulator audits the evidence.** MAS-style supervision means the RTO/RPO targets, the BCM exercise results and the incident postmortems are artefacts the bank must produce (✅ frame from the OpenShift guide §1/§3; cross-ref [../management/business_case_development_guide.md](../management/business_case_development_guide.md) for the investment framing — every rung of the resilience chain is a budgeted decision).

### 9.4 The Banking Table

| Banking reality (verified) | The zero-downtime consequence | Cross-ref |
|---|---|---|
| FAST runs real-time 24/7 ✅ (NETS guide §3.2) | No maintenance window exists for the rail; change must be invisible (§3–§4) | [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) |
| The switch authorizes synchronously against issuers ✅ (NETS guide §1–§2) | Stand-in processing and store-and-forward are availability features, not workarounds (§8.5) | same + §7.2 chain |
| "The switch that never sleeps" — no bank holiday changes the requirement ✅ (NETS guide §10) | Zero downtime is the product spec; the aspiration of §1.2 is the industry's default | same, §10 |
| MAS TRM / Notice 637 / BCBS 239 ✅ (OpenShift guide §1/§3 frame) | Resilience is evidenced: RTO/RPO, BCM drills (GameDays), postmortems | [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) |
| The trading day is the availability window (markets calendars) ✅-structural | Maintenance corridors are short, market-dependent, never assumed | [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) |
| Modernisation compresses batch toward real time (ISO 20022, event-driven settlement) ✅ (NETS guide §8) | Windows disappear; the zero-downtime bar rises with every rail upgrade | NETS guide §8; [event_stream_processing_guide.md](event_stream_processing_guide.md) |

### 9.5 The 24-7 Shapes — Cut-Offs and the Nightly Batch

The banking day has shapes the generic zero-downtime literature does not cover, and an architect in the industry must know them (✅-structural — the operational shapes below are the standard industry understanding; cross-ref [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) and [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) for the systems-side frames):

- **Payment cut-offs are availability events, not business hours.** A client's instruction accepted at 17:59 must clear with today's value date; at 18:01 it settles tomorrow. The cut-off is a *hard* boundary the systems enforce — which means the seconds around it are the highest-stakes availability window of the day: a 30-second blip at cut-off is a value-date failure for the client, while the same blip at 03:00 is invisible. Zero-downtime design in banking therefore prioritises the *calendars* — the cut-off times, the market holidays, the weekend rails (FAST's 24/7 vs the batch rails' T+1) — as first-class availability inputs.
- **The nightly batch is both the enemy and the ally.** The end-of-day cycle — interest accrual, position marking, statement production, settlement runs — is the classic scheduled-maintenance corridor, and the modernisation direction (§9.1's ISO 20022/event-driven shift) is compressing it. But the batch is also the *reconciliation heartbeat*: the day-end comparison is the banking version of §10's parity pipeline, and a bank that keeps its day-end reconciliation honest has the migration conscience the NETS guide demands ✅ (§9.4 lesson 3).
- **Standing instructions and direct debits never sleep either.** The recurring-payment estate (GIRO-class rails — cross-ref the NETS guide's §3.4) generates work on its own schedule, independent of any human maintenance plan; a failed overnight run is a missed salary payment by 08:00, not a ticket by 09:00.
- **The trading floor is the extreme case.** For capital markets, "availability" is measured in the market's microseconds of opportunity (cross-ref [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)); the maintenance window is the weekend, and even that is shared with the global market's other timezones.

The unified lesson: **in banking, the availability requirement is not "the system is up" — it is "the system is up at the moments the bank's obligations fire": the cut-off, the batch, the standing instruction, the market open.** The zero-downtime discipline of §2–§8 is what lets those moments never notice the change happening underneath them.


---

## 10. The Worked Example — A Zero-Downtime Migration

### 10.1 The Scenario — a Cymbal Bank System

**The familiar context:** a Cymbal Bank-style payments-and-trade platform — call it the **Payments Hub** — a 15-year-old Java/ESB estate that routes client payment instructions (SWIFT MT/MX, ISO 20022, domestic rails, the house's internal formats) through validation, sanctions screening, liquidity checks and booking into the general ledger (cross-ref [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) for the house systems frame; [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §9 for the same migration in the payments-operator's clothes). The estate has a nightly maintenance window that the business is now refusing to grant — clients' payments must flow 24/7 (the §9 reality), the regulators expect evidenced resilience (the MAS-style frame of §9.1), and the platform must move to a new event-driven microservices architecture (the [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) target shape) without a single user-visible gap. The constraints, stated as the business would state them:

- **RTO ≤ 15 minutes, RPO = 0** for the instruction pipeline (no lost payment instructions, ever — cross-ref §2.4's math table and the [../management/business_case_development_guide.md](../management/business_case_development_guide.md) investment framing).
- **The migration itself must not consume error budget** — the SLO is 99.95% on the instruction API (§2.3: that is ~4.4 hours/year of *allowed* failure; the migration is expected to contribute ~zero).
- **Dual-running with the legacy estate is mandatory** — the new platform must prove parity transaction-by-transaction before it takes over (the NETS lesson: "the reconciliation pipeline is the migration's conscience" ✅ [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) §9.4).
- **Every cut-over slice must be reversible** — no irreversible switch, no big-bang weekend, no "the old system is now read-only" without a retreat path.

### 10.2 The Migration Design

**Phase 0 — the observability and SLO baseline (months 1–2).** Instrument the legacy estate to the §2.3 standard: SLIs (instruction success rate, p95 end-to-end latency, queue depth), the SLOs, and the error-budget burn-rate alerts (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) for the platform monitoring stack). **Nothing migrates until "normal" is measurable** — the steady-state hypothesis that §7.2's Principles of Chaos demand.

**Phase 1 — the adapter wall and dark launch (months 2–6).** Stand the new platform up *beside* the legacy estate, wired through the same inbound channels as a passive observer: every instruction is **mirrored** into the new platform's validation, screening and booking services in **shadow mode** (§3.6's dark launch) — results computed, persisted to a shadow store, compared with the legacy outcome, never returned to the client. The comparison pipeline (the reconciliation service — the [event_stream_processing_guide.md](event_stream_processing_guide.md) matching machinery) is built *first*, because it is the only honest proof of parity. The new platform learns the house's dialects against recorded production traffic; the adapter wall makes the legacy estate's formats *configuration* at the edges (the NETS Gateway pattern ✅ — cross-ref the NETS guide §2 and §9.4).

**Phase 2 — expand: schema and state coexistence (months 6–12).** The data layer follows §4.2's expand-contract: the shared stores gain additive columns and the new-platform tables are created **alongside** the legacy tables; the new platform's writes go to both (the §4.3 dual-write, ordered and journaled); the outbox table carries every instruction event in the same transaction as the business record (§4.3); CDC tails the log and streams the events into the new platform's stores (§4.4). The dual-write is judged by the reconciliation pipeline: **until the new store's view of every instruction matches the legacy view — event by event, including the failure cases — the cut-over does not start** (the NETS §9.3 discipline ✅). This is the longest phase on purpose: the parallel period is where the migration's correctness is proven, and the expand step is individually reversible at any moment.

**Phase 3 — canary cut-over, corridor by corridor (months 12–18).** Production traffic moves to the new platform in **slices the business can name** — one client corridor (say, one large corporate's SWIFT MT103 flow), then a payment type, then a region — each slice a §3.4 canary ramped from 1% to 100% under the §2.3 error-budget burn alerts, with the legacy estate kept warm behind the router (§3.3's blue-green retreat path: the router flip is the rollback). The stateful core follows §5: the new platform's database runs **synchronously replicated across three AZs with automatic failover** (RDS-class, §5.2's 60–120 s window ridden out by the §8.3 retries-with-jitter in the API layer), and the instruction state machine treats **event ID as the idempotency key** (§8.4) so that retries, replays and the dual-run comparison never double-book a payment.

**Phase 4 — contract and retirement (months 18–24).** Once every corridor is live on the new platform and the reconciliation pipeline has held parity for a full reporting cycle, the *contract* phase begins (§4.2's third step): the legacy write path is frozen to read-only for one quarter (the bank's own audit wants a burn-in period), the legacy tables are dropped, the adapter wall thins to the residual dialects that still need it (the NETS lesson — "the adapter wall never comes down; it just gets thinner" ✅ NETS §9.3), and the nightly maintenance window — the one the business refused — is formally retired, because there is nothing left that needs it. The GameDays run throughout: the §7.3 drills (kill the primary AZ, fail over the database, replay a past production incident against the new platform) happen *before* each cut-over slice, not after.

**The design, mapped to this guide's sections:** the SLOs and error budget of §2 govern every gate; the deployment mechanics of §3 (dark launch → canary → blue-green retreat) govern every exposure; the data discipline of §4 (expand-contract, outbox, CDC, reconciliation) governs every state change; the stateful mechanisms of §5 and the routing of §6 carry the runtime; the chaos practice of §7 proves each phase; the patterns of §8 (breaker, bulkhead, retries, idempotency, degradation) are the connective tissue that lets the phases overlap safely; and §9 is the reason the business demanded the whole thing.

**The phase timeline at a glance** — the schedule the business signs off, with the exit gate of each phase:

| Phase | Duration | What happens | Exit gate |
|---|---|---|---|
| 0 — Observability baseline | Months 1–2 | SLIs, SLOs, error-budget burn alerts on the legacy estate (§2.3) | "Normal" is measurable — the §7.2 steady-state hypothesis exists |
| 1 — Adapter wall + dark launch | Months 2–6 | New platform mirrors traffic in shadow mode; reconciliation pipeline built first (§3.6, §10.2) | Parity on recorded production traffic; dialects are configuration |
| 2 — Expand: schema and state | Months 6–12 | Dual-write with outbox + CDC; backfills; the parallel period (§4.2–§4.4) | Event-by-event parity, including the failure cases — the NETS "conscience" gate ✅ |
| 3 — Canary cut-over by corridor | Months 12–18 | Corridor canaries 1%→100% under burn alerts; GameDay before each slice (§3.4, §7.3) | Every corridor live; error budget intact; retreat path warm |
| 4 — Contract and retirement | Months 18–24 | Legacy read-only quarter, drops, maintenance window retired (§4.2, §10.2) | Audit sign-off; parity held a full reporting cycle |

**The migration risk table** — the honest pre-mortem every phase gate is checked against:

| Risk | Mechanism that contains it | Section |
|---|---|---|
| The new platform mis-validates an instruction | Shadow mode + event-by-event reconciliation; no production traffic until parity holds | §3.6, §10.2 Phases 1–2 |
| Dual-write divergence (legacy and new stores disagree) | Transactional outbox for ordering + reconciliation pipeline + idempotent replay | §4.3–§4.4, §10.2 Phase 2 |
| Schema change breaks the old version mid-migration | Expand-contract: additive-only DDL, contract separated by months | §4.2 |
| A bad slice takes down client traffic | Corridor canaries ramped under error-budget burn alerts; blue-green router retreat | §3.4, §3.3, §2.3 |
| Database failover during cut-over loses instructions | Synchronous multi-AZ replication (RPO=0), automatic failover, retries-with-jitter ride the 60–120 s window | §5.2–§5.4, §8.3 |
| The dependency chain cascades during the migration | Circuit breakers + bulkheads + graceful degradation on every new call path | §8.1–§8.2, §8.5 |
| The team discovers the failure mode on go-live day | GameDays before each slice: AZ kill, failover drill, incident replay | §7.3 |
| The migration overruns and the business loses patience | Reversible slices: every phase can pause at a consistent, reconciled state; the contract phase is the only irreversible step and it is last | §4.2, §10.2 Phase 4 |

### 10.3 The Lessons

1. **The reconciliation pipeline is the migration.** Every phase is gated by parity, not by schedule — the NETS guide's lesson verbatim in spirit ✅ (§9.4). If you build only one thing first, build the comparison.
2. **Zero downtime is a budget discipline, not a technique.** The error budget (§2.3) turns "the migration must not hurt users" into a measurable gate; every slice spends a little budget or none, and the gates enforce it.
3. **Expand first, contract last, and never in the same release.** The §4.2 sequence is the whole data migration in three words; every shortcut (rename-in-place, same-release contract, big-bang switch) is how migrations become outages.
4. **State is the schedule.** The application can be canaried in an afternoon; the database dual-write and the reconciliation burn-in set the real timeline (§10.2 Phase 2). The project plan is the data plan.
5. **The retreat path is a feature.** Blue-green's router flip (§3.3), the warm legacy estate, the reversible expand steps — the migration's confidence comes from knowing every step can be undone, and that confidence is what lets you go faster, not slower.
6. **Chaos before cut-over, always.** The GameDay that fails in rehearsal is a gift; the GameDay that fails in production is an incident (§7.3).
7. **Banking adds the evidence requirement.** The RTO/RPO artefacts, the BCM exercise records, the parity reports — the migration's by-products are regulatory deliverables (§9.1). Engineer them as deliverables from day one, not as after-the-fact documentation.

---

## 11. The Summary — One Page

**Zero-downtime system design is the discipline of changing and failing a system without its users noticing — and it is built from ten stacked layers.** The **math** (§2) sets the scoreboard: the nines are downtime budgets (three nines = 8.77 hours a year, five nines = 5.26 minutes ✅), availability is MTBF/(MTBF+MTTR) ✅, and the SRE framework's SLI/SLO/error budget (✅ the 2016 Google SRE book) turns reliability into an accounting system where **every planned change and every unplanned outage spend the same budget** — which is why the rest of the guide exists. The **deployment strategies** (§3) make change invisible: rolling reuses capacity but demands version coexistence; blue-green (✅ Fowler 2010) buys instant cut-over and instant rollback with two environments; canary (✅ Fowler) meters risk to a small slice; feature flags and dark launches (✅ Fowler 2010) ship code with zero exposure and flip it live. The **data layer** (§4) makes state changeable: expand-contract/parallel change (✅ Fowler's ParallelChange; ✅ Prisma) turns every schema change into additive, reversible steps; the transactional outbox (✅ Richardson) solves dual-writes; CDC (✅ Debezium class) streams the log. The **stateful systems** (§5) make failure survivable: synchronous failover (✅ AWS — RPO≈0, 60–120 s promotion), quorum consensus (✅ Raft/etcd — split-brain impossible, fails closed), multi-AZ placement (✅ AWS/OpenShift frames). The **infrastructure** (§6) routes around failure: load balancers in milliseconds, health checks that know liveness from readiness (✅ K8s probes), DNS failover at TTL speed (✅ Route 53). The **chaos engineering** (§7) proves it: the Chaos Monkey (✅ Netflix, 2011) makes failure routine, the Principles of Chaos ✅ make it experimental, GameDays (✅ SRE) make it rehearsed. The **resilience patterns** (§8) are the connective tissue: circuit breakers (✅ Fowler/Nygard), bulkheads (✅ Nygard; Hystrix), retries with backoff and jitter (✅ AWS/Brooker), idempotency (✅ Stripe-class), graceful degradation. The **banking reality** (§9) is the forcing function: the payment switch has no maintenance night — "the switch that never sleeps" ✅ (NETS guide §10) — FAST runs 24/7 ✅, and MAS-style regulation demands the evidence. The **worked example** (§10) ties it together: a Cymbal Bank payments hub migrates by dark launch, dual-run, corridor canaries and reconciliation-gated cut-over — with the retreat path always warm.

**The final word — the zero-downtime illusion.** Here is the honest truth the whole discipline rests on: **zero downtime is never actually achieved — it is engineered so well that the failure becomes imperceptible, and the imperceptible failure is the illusion that makes the achievement real.** The database fails over in ninety seconds and no user notices, because the retry with jitter rode out the window; the schema changes under a live workload and no request fails, because expand-contract made the change additive; the platform migrates for eighteen months and the client's MT103 still clears, because the reconciliation pipeline held parity and the canary ramped under the error budget. The switch that never sleeps does sleep — a node at a time, an AZ at a time, a version at a time — and the *discipline* is that its sleeping is scheduled, bounded, rehearsed and invisible. That is the zero-downtime illusion, and it is the finest trick in the reliability canon: **you do not eliminate downtime, you design it out of the user's experience — the aspiration of §1.2, made real by the math of §2, the mechanics of §3–§6, the proof of §7, the patterns of §8, the necessity of §9, and the discipline of §10.** When the counter keeps accepting the tap through the migration, the outage you never saw was the one you designed. The whole guide in one sentence: **zero downtime is not the absence of failure — it is the absence of *perceived* failure, engineered by budget, redundancy, and rehearsal — the zero-downtime illusion, and the most valuable illusion a systems architect can build.**

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Zero-downtime** | The aspiration that a system remains continuously available to users through planned change and unplanned failure; in the disciplined sense, no *user-perceived* downtime caused by the change process (§1.2) |
| **Availability** | The fraction of time a system is able to perform its function; measured as uptime/(uptime+downtime) or MTBF/(MTBF+MTTR) (§2) |
| **Nines** | The count of leading nines in an availability percentage — 99.9% = three nines ≈ 8.77 h/year downtime; 99.999% = five nines ≈ 5.26 min/year (§2.1) |
| **MTBF** | Mean Time Between Failures — operating time divided by the number of failures; the reliability measure (§2.2) |
| **MTTR** | Mean Time To Repair/Recover — detection + response + restore time; the resilience measure (§2.2) |
| **SLO** | Service Level Objective — the internal reliability target set on an SLI, deliberately below 100% (✅ SRE book, §2.3) |
| **SLA** | Service Level Agreement — the external customer contract, normally looser than the SLO, with consequences attached (§2.3) |
| **Error budget** | 1 − SLO — the allowed failure in a window; spent by outages *and* by user-visible change; launch-freeze when exhausted (✅ SRE, §2.3) |
| **Rolling deployment** | Deploying a new version instance-by-instance while traffic shifts through health checks; no idle capacity; needs version coexistence (§3.2) |
| **Blue-green** | Two identical production environments with a router switch between them; instant cut-over and instant rollback (✅ Fowler 2010, §3.3) |
| **Canary** | Rolling a new version out to a small traffic/user subset first, proceeding or rolling back on observed SLIs (✅ Fowler, §3.4) |
| **A-B testing** | Cohort-split routing comparing *business* outcomes between versions; experiment toggles; distinct from canary's release validation (§3.5) |
| **Feature flag** | A config-driven toggle in the application controlling feature visibility; release/experiment/ops/permissioning types (✅ Fowler 2010, §3.6) |
| **Dark launch** | Deploying code to production while inert (flag off) or shadow-trafficed, with zero user exposure until the flip (§3.6) |
| **Expand-contract** | The three-phase schema migration: expand (additive) → migrate (dual-run/backfill) → contract (drop old, later) (§4.2) |
| **Parallel change** | Fowler's name for the same pattern — the migrate phase is the transition until all code works with the new schema (✅, §4.2) |
| **Schema migration** | The controlled evolution of a database schema; zero-downtime migrations require backward-compatible, additive steps (§4.2) |
| **Dual-writes** | Writing two systems from one code path; has an atomicity gap that the outbox or reconciliation must close (§4.3) |
| **CDC** | Change Data Capture — tailing the database transaction log to emit committed changes as an event stream (✅ Debezium class, §4.4) |
| **Failover** | Automatic promotion of a standby on primary failure; synchronous replication gives RPO≈0, promotion typically 60–120 s (✅ AWS, §5.2) |
| **Quorum** | The majority (N/2+1) of a consensus cluster required to commit and elect; makes split-brain impossible, fails closed on loss (✅ Raft/etcd, §5.3) |
| **Multi-AZ** | Placing replicas across physically isolated availability zones so one AZ failure cannot take the quorum or standby (✅, §5.4) |
| **Load balancer** | The traffic-distribution layer that drains, health-checks and removes instances — the first rung of failure routing (§6.2) |
| **Health check** | A probe distinguishing liveness (restart) from readiness (remove from pool); the routing layer's eyes (§6.3) |
| **DNS failover** | Health-checked DNS records switching endpoints; active-active or active-passive; TTL-bound speed — the last-resort rung (✅ Route 53, §6.4) |
| **Chaos engineering** | The practice of injecting failures as controlled experiments to prove resilience; hypothesis, production, automation (✅ Principles of Chaos, §7) |
| **Chaos Monkey** | Netflix's tool (c. 2011) that randomly terminates production instances to make failure routine (✅, §7.1) |
| **GameDay** | A scheduled, bounded, rehearsed failure drill testing people, process and architecture (✅ SRE, §7.3) |
| **Circuit breaker** | The failure-threshold trip that fails fast and self-resets (half-open); prevents cascading failures (✅ Fowler/Nygard, §8.1) |
| **Bulkhead** | Per-dependency bounded resource pools isolating failure compartments (✅ Nygard; Hystrix thread pools, §8.2) |
| **Retry** | Re-attempting a transiently failed operation; only safe when idempotent (§8.3) |
| **Backoff** | The waiting discipline between retries — exponential growth, capped, with jitter to prevent retry storms (✅ AWS/Brooker, §8.3) |
| **Idempotency** | The property that N executions have the same effect as one; enabled by keys, uniqueness constraints, dedup (§8.4) |
| **Graceful degradation** | Serving reduced function rather than none — stand-in, read-only, cached — protecting the SLO'd path (§8.5) |
| **RTO** | Recovery Time Objective — how fast service must be restored after a disaster; the DR-side time target (§2.4, openshift guide §3) |
| **RPO** | Recovery Point Objective — how much data loss is acceptable; the DR-side data target; RPO=0 means no accepted loss (§2.4) |

---

## 13. Claims Status and Verification Notes

**Verification pass: 2026-08-24, live web access via the self-hosted Firecrawl backend (web search + direct page extraction, both working throughout this pass).** Primary sources extracted/verified live: **martinfowler.com** (BlueGreenDeployment — 1 March 2010, the two-environment/router-switch/rapid-rollback mechanics, the database-changes caveat, the Dan North/Jez Humble naming note; CanaryRelease — the small-subset definition; FeatureFlag — 29 October 2010, the release/experiment/ops/permissioning taxonomy, the toggle-points and flag-debt cautions, the Pete Hodgson reference; CircuitBreaker — 6 March 2014, the Nygard *Release It!* attribution, the threshold-trip and half-open self-reset mechanics, the Hystrix reference; ParallelChange — the migrate-phase framing), **docs.aws.amazon.com** (RDS Multi-AZ — synchronous replication to a standby in another AZ, automatic failover, 60–120 s typical promotion times; Route 53 — health checks, active-active vs active-passive failover), **sre.google** (the SRE Workbook's error-budget-policy and implementing-SLOs chapters), the availability-math references (calcoi.com, calculatorhub.com, conyso.com, mechcodex.com — the nines table and the MTBF/(MTBF+MTTR) formula, all consistent), the chaos-engineering sources (Wikipedia's chaos-engineering article — the 2011 Chaos Monkey origin; the Netflix/chaosmonkey GitHub README; principlesofchaos.org), the outbox/CDC literature (microservices.io's outbox pattern; the Debezium-centred CDC treatment), the AWS Architecture Blog (timeouts/retries/backoff-with-jitter — Marc Brooker), and the repo's own verified cluster (the NETS guide's ✅ ledger — FAST 24/7, the resilience chain, the "switch that never sleeps" final word; the OpenShift guide's HA/DR/BC and MAS-regulatory frame).

**✅ Verified this pass (live sources):** the nines table and its arithmetic; A = MTBF/(MTBF+MTTR); the Google SRE book's identity (O'Reilly 2016, Beyer/Jones/Petoff/Murphy) and the SLI/SLO/error-budget framework including "100% is the wrong target" and the launch-freeze policy (Workbook); all four Fowler bliki mechanics cited in §3 and §8 (BlueGreenDeployment 2010, CanaryRelease, FeatureFlag 2010 with the taxonomy, CircuitBreaker 2014 with the Nygard attribution); the ParallelChange/expand-contract framing; the RDS Multi-AZ failover properties (sync replication, 60–120 s); Route 53 health-check and failover-routing shapes; the Chaos Monkey origin (2011) and the tool's purpose statement; the four Principles of Chaos; the GameDay practice (SRE); the outbox/dual-write framing (Richardson, microservices.io) and Debezium-class CDC; the backoff-with-jitter canon (Brooker).

**⚠ Flagged / unverified this pass (honest ledger):** the **dark-launch** concept beyond the feature-flag mechanism — the shadow-traffic variant is widely documented industry practice but has no single canonical primary source (⚠-structural, §3.6); the **bulkhead** framing beyond Nygard's pattern and Hystrix's thread-pool implementation (⚠-structural, §8.2); **graceful-degradation** taxonomy specifics (⚠-structural, §8.5); any **industry adoption statistics** for chaos engineering or zero-downtime practices (⚠ — not verifiable from a single authoritative source; deliberately not quoted); the **health-check cascade** failure mode (⚠-structural — well-documented but attributed to practice literature, §6.3); the trading-day availability framing for capital markets (⚠-structural — cross-ref the repo's capital-markets guide for its own ledger); specific Cymbal Bank-internal system details (⚠ — the §10 worked example is an illustrative scenario in the house's context, not a description of any real Cymbal Bank system).

**Repository convention:** ✅ = verified this pass or in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = standard industry practice widely documented, not attributable to a single primary source.

---

## 14. Cross-References and Further Reading

**The reliability/system-design cluster (sibling, technology/):**
- **[openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)** — **THE platform-side companion.** Its §1–§3 (HA/DR/BC/RPO/RTO), §4 (multi-AZ topology) and §6 (probes) are this guide's §5–§6 at the Kubernetes layer; its MAS TRM / Notice 637 / BCBS 239 framing feeds §9.1. Read the two together: this guide is the discipline, that guide is the platform.
- **[monolith_to_microservices_guide.md](monolith_to_microservices_guide.md)** — the migration runbook; §10's strangler path, adapter wall and corridor cut-overs are its patterns in zero-downtime clothes.
- **[domain_driven_design_guide.md](domain_driven_design_guide.md)** — the decomposition angle behind §10's bounded-context design (light cross-ref).
- **[kafka_alternatives_guide.md](kafka_alternatives_guide.md)** and **[message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)** — the messaging-reliability angle: at-least-once delivery and consumer idempotency underpin §4.3–§4.4 and §8.4.
- **[late_arriving_data_guide.md](late_arriving_data_guide.md)** — the data-lag consequences of §4's eventual-consistency windows.
- **[event_stream_processing_guide.md](event_stream_processing_guide.md)** — the streaming machinery behind §4.4's CDC and §10.2's reconciliation pipeline.
- **[distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md)** and **[distributed_auth_guide.md](distributed_auth_guide.md)** — the resilience and auth-availability angles (light cross-refs, §8).
- **[singapore_data_centres_guide.md](singapore_data_centres_guide.md)** — the DC/DR frame under §5.4 and §6.4 (70+ DCs, Tier III norm ⚠).
- **[grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md)**, **[ddia_study_companion_guide.md](ddia_study_companion_guide.md)**, **[system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md)** — the study-guide genre; cross-ref their availability/replication chapters for the interview-shaped version of §2 and §5.

**The banking cluster (prefix `../banking/`):**
- **[../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md)** — **THE banking cross-ref.** Its §3.2 (FAST 24/7), §7.2 (the resilience chain — stand-in, active/active, DR), §9 (the switch-modernization worked example) and §10 (the final word — "the switch that never sleeps") are §9 of this guide in full.
- **[../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)** — the bank-core interface for §9.3's batch-vs-real-time frame.
- **[../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)** — the trading-day availability window (§9.3).
- **[../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md)** — the house systems context for §10's scenario.
- The bank software-systems series (HSBC, BNP Paribas, Deutsche Bank, UOB, OCBC, Bank of America...) — the estate-structure and 24-7-banking precedents.

**The management cluster (prefix `../management/`):**
- **[../management/business_case_development_guide.md](../management/business_case_development_guide.md)** — the RTO/RPO investment angle: the cost of an extra nine vs the business value of avoided downtime (§2.4, §9.1 — light cross-ref).

**Primary sources and further reading:** the Google SRE book (*Site Reliability Engineering*, O'Reilly 2016 — Beyer, Jones, Petoff, Murphy) and the SRE Workbook (sre.google) — the SLI/SLO/error-budget canon; the Martin Fowler bliki (BlueGreenDeployment, CanaryRelease, FeatureFlag, CircuitBreaker, ParallelChange — martinfowler.com) and Pete Hodgson's "Feature Toggles"; Jez Humble & Dave Farley, *Continuous Delivery*; Michael Nygard, *Release It!* (the circuit-breaker and bulkhead canon); the AWS Architecture Blog — Marc Brooker, "Timeouts, retries, and backoff with jitter" (2015); Chris Richardson, microservices.io (the transactional outbox); Debezium documentation; the AWS documentation (RDS Multi-AZ, Route 53 health checks and failover); principlesofchaos.org and the Netflix chaos-engineering lineage (chaosmonkey GitHub, Wikipedia's chaos-engineering article); the Prisma Data Guide (expand-and-contract); the availability-math references (calcoi.com, calculatorhub.com, conyso.com, mechcodex.com); and the repo's own verified cluster listed above.

---

*End of guide — the zero-downtime illusion: the outage you never saw was the one you designed.*
