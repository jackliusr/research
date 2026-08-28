# Open-Source Workload Automation Alternatives: A Landscape and Decision Guide

**The enterprise workload automation (WLA) landscape being displaced — BMC Control-M, IBM Workload Scheduler, CA-7/AutoSys, Stonebranch, Redwood RunMyJobs, ActiveBatch, JAMS — and the open-source alternatives (Apache Airflow, Dagster, Prefect, Temporal, Kestra, Apache DolphinScheduler, Azkaban, Apache Oozie, Rundeck, Luigi, Windmill, n8n): the comparison framework (DAG-based batch orchestration vs stateful durable execution; data-pipeline vs general scheduling; code-first vs config-driven), the migration decision framework for a Control-M estate (selection criteria, target-selection matrix, the condensed Control-M concepts mapping, the phased inventory → map → parallel-run → cutover pattern, coexistence and reconciliation), the banking angle (EOD batch windows, audit trails, change management, the mainframe boundary), and the Cymbal Bank worked example — a WLA estate rationalization consolidating Control-M + TWS + CA-7 workloads onto Apache Airflow and Temporal, from the 1980s mainframe schedulers to the open-source rhythm.**

> **Author:** Jack Liu Shurui, Solution Architect — Cymbal Bank, Singapore
> **Context:** Technology / Workload Automation — the landscape-and-decision companion to the Control-M cluster: what enterprise WLA actually is (the mainframe-born batch schedulers of the 1980s and their distributed descendants), who owns the incumbent products today (KKR/BMC, IBM, Broadcom, Stonebranch, Redwood, Advanced Systems Concepts, MVP/JAMS), which open-source platforms are credible targets (Airflow, Dagster, Prefect, Temporal, Kestra, DolphinScheduler, Azkaban, Oozie, Rundeck, Luigi, plus the lightweight tier Windmill and n8n), how to compare them (DAG-batch vs stateful durable execution; data-pipeline vs general scheduling; code-first vs config-driven; self-hosted vs managed; license and community health; maturity; banking-appropriateness), and how to migrate a Control-M estate (selection criteria, the target-selection matrix, the condensed Control-M concepts mapping — the *full* Control-M → Airflow mapping lives in the companion [Control-M → Airflow Migration Guide](control_m_to_airflow_migration_guide.md), cross-ref, do not re-derive — the phased inventory → map → parallel-run → cutover pattern, and coexistence/reconciliation during parallel run). The banking angle cross-references the [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide (audit, outsourcing, operational resilience) and the [Cybersecurity](cybersecurity_guide.md) guide (change management, privileged access, secrets). The worked example is a Cymbal Bank WLA estate rationalization: Control-M + IBM Workload Scheduler + CA-7 workloads consolidated onto Apache Airflow (batch/DAG workloads) and Temporal (stateful long-running flows), with the decision matrix used, a quarterly phased timeline, and the risk register.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** verified live this pass — the KKR acquisition of BMC (completed 2 October 2018 — media.kkr.com and bmc.com newsroom), the Broadcom acquisition of CA Technologies (announced 11 July 2018, US$18.9B cash — investors.broadcom.com), Tivoli Systems founded 1989 and merged into IBM 1996 (en.wikipedia.org/wiki/Tivoli_Software), CA-7's UCCEL/UCC-7 lineage (en.wikipedia.org/wiki/CA-7_(software)), JAMS released 1987 and the June 2025 Fortra spin-out (jamsscheduler.com/about), Airflow's Airbnb-2014/Incubator-2016/TLP-2019 history (en.wikipedia.org/wiki/Apache_Airflow), Oozie's TLP status August 2012 and retirement February 2025 (attic.apache.org/projects/oozie.html), DolphinScheduler's 17 March 2021 graduation (incubator.apache.org/projects/dolphinscheduler.html), Temporal's Cadence-fork lineage (github.com/temporalio/temporal README), and GitHub API repo-creation dates for Azkaban (2012), Luigi (2012), Rundeck (2010), Dagster (2018), Prefect (2018), Kestra (2019), n8n (2019), Windmill (2022). All URLs are repeated in the claims audit (§13). Everything verified this pass is marked ✅; anything not confirmed is flagged ⚠ honestly. No dates, founders, or origins were invented.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [Control-M → Airflow Migration Guide](control_m_to_airflow_migration_guide.md) (**the full Control-M → Airflow mapping — cross-ref, do not re-derive**), [Control-M Comprehensive Guide](control_m_guide.md) (**the Control-M deep-dive**), [Control-M Migration Options](control_m_migration_options_guide.md) (**the earlier general options survey; this guide supersedes it in depth for open-source targets**), [Temporal Workflow](temporal_workflow_guide.md) (**the Temporal deep-dive**), [Cybersecurity](cybersecurity_guide.md) (**the change-management/security companion — cross-ref §10**)
> **Companion guides (banking/, prefix `../banking/`):** [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) (**the regulatory companion — cross-ref §10**)

---

**How to use this guide:** Section 1 is the incumbent landscape — what WLA is, why banks run it, and the seven enterprise products being displaced, each verified at primary sources. Section 2 is the open-source landscape — each target platform verified at its project site, GitHub repository, or Wikipedia. Section 3 is the comparison framework — the axes that actually separate the tools (DAG-batch vs stateful durable execution; data-pipeline focus vs general scheduling; code-first vs config-driven; self-hosted vs managed; license and community health; maturity; banking-appropriateness). Section 4 is the target-selection matrix — concrete scenarios and which platform wins each. Section 5 is the condensed Control-M concepts mapping — one table only (the full Airflow mapping lives in the companion guide). Section 6 is the phased migration pattern — inventory → map → parallel-run → cutover. Section 7 is coexistence and reconciliation between old and new schedulers during parallel run. Section 8 is the banking angle — EOD batch windows, audit trails, and change management. Section 9 is the mainframe boundary — what open source can and cannot do on z/OS. Section 10 is the Cymbal Bank worked example — the WLA estate rationalization with the decision matrix, the quarterly timeline, and the risk register. Section 11 is the claims audit (✅/⚠/❌). Section 12 is "What Could Not Be Verified". Section 13 is the glossary and references. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` are prefixed `../banking/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (URL given in §13); ⚠ = flagged/unverified/approximate; ⚠-knowledge = well-documented industry knowledge not re-verified live this pass. No dates, founders, or origins were invented.

---

## Table of Contents

1. [The Enterprise WLA Landscape Being Displaced](#1-the-enterprise-wla-landscape-being-displaced)
   - 1.1 [What WLA Is — and Why Banks Run It](#11-what-wla-is--and-why-banks-run-it)
   - 1.2 [BMC Control-M](#12-bmc-control-m)
   - 1.3 [IBM Workload Scheduler (TWS/IWS)](#13-ibm-workload-scheduler-twsiws)
   - 1.4 [CA Workload Automation — CA-7 and AutoSys](#14-ca-workload-automation--ca-7-and-autosys)
   - 1.5 [Stonebranch — Universal Controller](#15-stonebranch--universal-controller)
   - 1.6 [Redwood RunMyJobs](#16-redwood-runmyjobs)
   - 1.7 [ActiveBatch](#17-activebatch)
   - 1.8 [JAMS](#18-jams)
   - 1.9 [The Incumbent Landscape at a Glance](#19-the-incumbent-landscape-at-a-glance)
2. [The Open-Source Alternatives](#2-the-open-source-alternatives)
   - 2.1 [Apache Airflow](#21-apache-airflow)
   - 2.2 [Dagster](#22-dagster)
   - 2.3 [Prefect](#23-prefect)
   - 2.4 [Temporal](#24-temporal)
   - 2.5 [Kestra](#25-kestra)
   - 2.6 [Apache DolphinScheduler](#26-apache-dolphinscheduler)
   - 2.7 [Azkaban](#27-azkaban)
   - 2.8 [Apache Oozie](#28-apache-oozie)
   - 2.9 [Rundeck](#29-rundeck)
   - 2.10 [Luigi](#210-luigi)
   - 2.11 [The Lightweight Tier — Windmill and n8n](#211-the-lightweight-tier--windmill-and-n8n)
   - 2.12 [The Open-Source Landscape at a Glance](#212-the-open-source-landscape-at-a-glance)
3. [The Comparison Framework](#3-the-comparison-framework)
   - 3.1 [DAG-Based Batch Orchestration vs Stateful Workflow Engines](#31-dag-based-batch-orchestration-vs-stateful-workflow-engines)
   - 3.2 [Data-Pipeline Focus vs General Scheduling](#32-data-pipeline-focus-vs-general-scheduling)
   - 3.3 [Code-First vs Config-Driven](#33-code-first-vs-config-driven)
   - 3.4 [Self-Hosted vs Managed Offerings](#34-self-hosted-vs-managed-offerings)
   - 3.5 [Community Health and License Notes](#35-community-health-and-license-notes)
   - 3.6 [Maturity](#36-maturity)
   - 3.7 [Banking-Appropriateness — the Five Questions](#37-banking-appropriateness--the-five-questions)
4. [The Target-Selection Matrix](#4-the-target-selection-matrix)
5. [The Condensed Control-M Concepts Mapping](#5-the-condensed-control-m-concepts-mapping)
6. [The Phased Migration Pattern — Inventory, Map, Parallel-Run, Cutover](#6-the-phased-migration-pattern--inventory-map-parallel-run-cutover)
   - 6.1 [Phase 1 — Inventory](#61-phase-1--inventory)
   - 6.2 [Phase 2 — Map](#62-phase-2--map)
   - 6.3 [Phase 3 — Parallel-Run](#63-phase-3--parallel-run)
   - 6.4 [Phase 4 — Cutover](#64-phase-4--cutover)
7. [Coexistence and Reconciliation During Parallel Run](#7-coexistence-and-reconciliation-during-parallel-run)
8. [The Banking Angle — EOD Batch Windows, Audit Trails, Change Management](#8-the-banking-angle--eod-batch-windows-audit-trails-change-management)
9. [The Mainframe Boundary — What Open Source Can and Cannot Do on z/OS](#9-the-mainframe-boundary--what-open-source-can-and-cannot-do-on-zos)
10. [Cymbal Bank Worked Example — WLA Estate Rationalization](#10-cymbal-bank-worked-example--wla-estate-rationalization)
    - 10.1 [The Starting Estate](#101-the-starting-estate)
    - 10.2 [The Decision Matrix Used](#102-the-decision-matrix-used)
    - 10.3 [The Phased Timeline (Quarters)](#103-the-phased-timeline-quarters)
    - 10.4 [The Risk Register](#104-the-risk-register)
11. [The Claims Audit](#11-the-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary and References](#13-glossary-and-references)

---

## 1. The Enterprise WLA Landscape Being Displaced

### 1.1 What WLA Is — and Why Banks Run It

Workload automation (WLA) — historically "job scheduling" or "batch scheduling" — is the discipline of planning, executing, monitoring, and recovering scheduled IT workloads: the overnight batch that settles payments, the EOD (end-of-day) processing that closes books, the regulatory file generation that lands in MAS's hands by a deadline, the feeder jobs that move data between core banking and the data warehouse. The category was born on the IBM mainframe: enterprises needed something smarter than a cron table to express *dependencies* ("run job B only after job A completes successfully, but only on weekdays, and rerun it with these overrides if it fails"), and the products that delivered this became mission-critical systems of record in their own right.

The WLA estate in a typical large bank is a **multi-vendor patchwork**: a mainframe scheduler (Control-M z/OS or CA-7) for the JES2/JES3 batch, a distributed scheduler (Control-M/Agent, TWS, AutoSys) for the UNIX/Linux/Windows job streams, and one or more modern orchestration layers for the data platform. The incumbents below are what that patchwork is made of — and why a bank would want to reconsider each of them in the 2020s: private-equity ownership and price pressure, mainframe-centric architectures that predate the cloud, per-CPU/agent licensing that punishes scale-out, and an operations model (calendar-based, agent-managed) that fits batch windows far better than event-driven, API-first pipelines.

### 1.2 BMC Control-M

Control-M is the flagship of **BMC Software**, now owned by **KKR**. The acquisition history is well documented: BMC was taken private by a Bain Capital–led investor group (with Golden Gate Capital, GIC, Insight Venture Partners, Elliott Management) in 2013, and **KKR completed its acquisition of BMC on 2 October 2018** (announced 29 May 2018) — verified at media.kkr.com and bmc.com newsroom. ✅

The product lineage predates BMC: **Control-M was created in the 1980s by New Dimension Software (Israel) for mainframes**, and BMC acquired New Dimension in 1999 for approximately US$675M — per this repository's [Control-M Comprehensive Guide](control_m_guide.md) (cross-ref; the deep-dive lives there). ✅-repo Control-M today spans the z/OS mainframe product, the distributed Control-M/Agent model, Control-M Workbench, the Automation API, and a SaaS offering; it remains the most widely deployed enterprise WLA product in banking, and the de-facto reference point for every migration discussion in this guide.

### 1.3 IBM Workload Scheduler (TWS/IWS)

IBM Workload Scheduler (formerly IBM Tivoli Workload Scheduler, TWS; the "IWS" shorthand follows the 9.3 rebrand) is the Tivoli-lineage product. The lineage verifies cleanly at Wikipedia: **Tivoli Systems Inc. was founded in Austin, Texas in 1989 by Bob Fabbio** (with Peter Valdes, Todd Smith and Steve Marcie, all ex-IBM), and **merged into IBM in 1996**; the Tivoli name became an IBM brand in 2002 and was phased out from 2013, with TWS renamed IBM Workload Scheduler as of release 9.3. ✅ The distributed scheduler itself traces further back: the TWS engine originated as **Maestro, from Unison Software**, which Tivoli acquired in 1997 — reported at ~US$170M in stock — ⚠ (secondary source: softpanorama.org; not re-verified at a primary source this pass). TWS/IWS remains the standard scheduler across much of the IBM-centric banking world, often paired with IBM Z mainframe batch (JES) via the z/OS version of the product.

### 1.4 CA Workload Automation — CA-7 and AutoSys

The CA lineage is now the **Broadcom** lineage. **Broadcom announced the acquisition of CA Technologies for US$18.9 billion in cash on 11 July 2018** — verified at investors.broadcom.com — and completed it later that year. ✅ CA Technologies itself began as Computer Associates International, founded 1976 by Charles B. Wang and Russell Artzt (Wikipedia). ✅ The two WLA products that matter:

- **CA-7** — the mainframe scheduler. Wikipedia verifies the lineage: **CA-7 was originally UCC-7 from UCCEL Corporation; Computer Associates took ownership in 1987 when it acquired UCCEL and renamed the product CA-7**. ✅ CA-7 is "commonly used by banks and other large enterprises with IBM mainframe IT computing platforms" — Wikipedia cites its role in the 2012 RBS/NatWest batch-processing failure, a cautionary tale that recurs in §8.
- **AutoSys** (CA Workload Automation AE) — the distributed scheduler. It originated at **AutoSystems Corporation**, which CA acquired in 1997 — ⚠ (widely cited industry history, not re-verified at a primary source this pass). AutoSys has been sold under successive names: Unicenter AutoSys Job Management → CA Workload Automation AE (AutoSys Edition). ✅-knowledge

Broadcom now sells CA-7 Workload Automation and CA Workload Automation AE as part of its mainframe software portfolio (broadcom.com product pages).

### 1.5 Stonebranch — Universal Controller

Stonebranch's Universal Controller (UC) is the modern, agentless-capable WLA platform, with Universal Automation Center (UAC) as the platform umbrella. The company celebrated **25 years of innovation in IT automation in September 2024** — implying a founding around 1999 — ⚠ (bizwire-distributed press release headline verified; the article body did not render live this pass). Industry directories place the headquarters in the Atlanta area ⚠; the task brief describes the company as German-rooted ⚠ — Stonebranch's deep European presence and German founding narrative are commonly cited but were not re-verified at a primary source this pass (the stonebranch.com about page verified live states "with decades of experience" but gives no founding year ✅-partial). The UC product is positioned as a hybrid IT orchestration layer that sits *above* schedulers (it can orchestrate Control-M, TWS, AutoSys and modern pipelines alike) — a different competitive posture from the batch schedulers.

### 1.6 Redwood RunMyJobs

Redwood Software's RunMyJobs is the SaaS/job-as-a-service WLA offering (also sold as RunMyJobs Process Automation), positioned heavily at finance teams who want batch automation without running a scheduler estate. **Founding year could not be verified this pass**: Redwood Software has no English Wikipedia article, and the redwood.com about/company URLs returned 404 or blocked responses ⚠. The company is widely reported as a Dutch-founded (Utrecht) software firm from the early 1990s, but that is flagged ⚠ unverified here rather than asserted. For the decision framework what matters is the *category* — managed, finance-friendly, low-code batch automation with strong SAP/ERP connectors — not the founding year.

### 1.7 ActiveBatch

ActiveBatch (Advanced Systems Concepts, Inc.) is the Windows-centric "workload automation and job scheduling" platform with a visual workflow designer, popular in mid-market enterprise IT. **Founding details could not be verified this pass**: the Wikipedia "ActiveBatch" title redirects to a trademark-litigation case (Network Automation v. Advanced Systems Concepts, 9th Cir. 2011 — which at least confirms the company name and product as of 2011 ✅-partial), and the vendor about pages blocked automated access ⚠. The company is widely reported as founded in the early 1980s (1981 per industry directories), flagged ⚠ unverified here.

### 1.8 JAMS

JAMS (Job Access & Management System) is the scheduler with the clearest verifiable lineage of the mid-tier group. The vendor's own about page states: **"the same JAMS Scheduler that was released in 1987"**, developed as part of **MVP Systems Software**, and — a notable 2025 development — **in June 2025 the JAMS business separated from Fortra and became an independent company, JAMS Software, owned by growth equity firms PSG and 2ndWave plus employees**. ✅ (jamsscheduler.com/about, verified live). So: JAMS released 1987 under MVP Systems Software; MVP's scheduler line passed through the HelpSystems → Fortra portfolio; and the product is now independent again. ✅

### 1.9 The Incumbent Landscape at a Glance

| Product | Vendor (now) | Core platform | Lineage highlights | Verification |
|---|---|---|---|---|
| Control-M | BMC Software (owned by KKR since Oct 2018) | z/OS + distributed agents + SaaS | New Dimension Software (Israel), 1980s; BMC acquired 1999 | ✅ KKR press release; ✅-repo |
| IBM Workload Scheduler (TWS/IWS) | IBM | z/OS + distributed | Tivoli Systems (1989) → IBM 1996; engine from Unison's Maestro (1997) | ✅ Wikipedia; ⚠ Maestro |
| CA-7 | Broadcom (ex-CA) | z/OS (JES) | UCC-7 from UCCEL; CA acquired UCCEL 1987 | ✅ Wikipedia |
| AutoSys (CA WLA AE) | Broadcom (ex-CA) | Distributed | AutoSystems Corp; CA acquired 1997 | ⚠ |
| Universal Controller | Stonebranch | Distributed/hybrid, agentless option | Founded ~1999; 25th anniversary 2024 | ⚠ |
| RunMyJobs | Redwood Software | SaaS, finance-facing | Founding unverified | ⚠ |
| ActiveBatch | Advanced Systems Concepts | Windows/enterprise IT | Founding unverified (1980s per directories) | ⚠ |
| JAMS | JAMS Software (ex-Fortra, ex-MVP) | Windows/enterprise IT | Released 1987; Fortra spin-out June 2025 | ✅ |

### 1.10 The Displacement Drivers

Why is a bank in 2026 even *looking* at open source for something as mission-critical as its batch estate? Five drivers, all structural rather than ideological:

1. **Ownership churn and price pressure.** The incumbents have spent the last decade inside private-equity portfolios (BMC → KKR in 2018 ✅; CA → Broadcom in 2018 ✅ — both verified in §1.2/§1.4) or inside mega-vendor rationalizations (Tivoli → IBM brand phase-out from 2013 ✅). PE owners optimize cash yield; banks experience that as per-agent/per-MIPS license growth and slower roadmap responsiveness. Open source removes the *per-unit* pricing model entirely — the license cost becomes an engineering cost.
2. **The cloud and the data platform.** The modern data estate (warehouse, lakehouse, Kafka, Kubernetes) does not speak Control-M's language — it speaks DAGs, sensors, and API triggers. Every bank's data team already runs Airflow or its kin for the *new* workloads; the WLA estate is the legacy island that has not joined them. The migration is less "replace a tool" and more "connect the island to the mainland."
3. **The operator-skill cliff.** The scheduler-operator generation that reads Control-M forecasting screens and CA-7 control statements is retiring; the incoming generation writes Python and Git. §3.3 is the honest treatment of that skill shift — but the direction is not in doubt.
4. **Audit and change-management expectations.** Regulators and internal audit increasingly expect definition-level change control, CI-tested promotion, and queryable run histories — which are *native* to code-in-Git platforms and *retrofitted* onto vendor control databases (§8).
5. **Mainframe economics.** z/OS MLC pricing makes every incremental mainframe workload expensive; moving the *orchestration* of mainframe batch off Control-M z/OS/CA-7 (while leaving execution on z/OS — §9) is one of the few ways to shrink the mainframe bill without touching the JCL.

None of these drivers requires abandoning the mainframe or the batch model — they require changing *who decides what runs when*, which is exactly the boundary §9 draws.

## 2. The Open-Source Alternatives

### 2.1 Apache Airflow

**Apache Airflow** is the de-facto standard for open-source DAG-based batch orchestration, and it is the primary Control-M replacement target for data-platform workloads. It started at Airbnb in **October 2014** (author: Maxime Beauchemin), entered the Apache Incubator in **March 2016**, and became an Apache top-level project in **January 2019**; Apache-2.0 licensed. ✅ (Wikipedia; verified live) Workflows are DAGs authored in Python; execution is distributed across a scheduler, workers, and a metadata database. The *full* Control-M → Airflow mapping (jobs→DAGs, run cycles→schedules, dependencies→task dependencies, agents→workers, forecasting→run history) lives in the companion [Control-M → Airflow Migration Guide](control_m_to_airflow_migration_guide.md) — this guide cross-refs it and does not re-derive it.

### 2.2 Dagster

**Dagster** is the asset-based data orchestration platform: instead of scheduling *tasks*, you declare *data assets* and the platform derives the compute graph from asset dependencies, with software-defined assets, asset lineage, and the Dagster UI for observation. The GitHub repository (dagster-io/dagster, Apache-2.0) was created **30 April 2018** ✅ — consistent with the widely reported founding of the commercial steward Elementl in 2018 by **Nick Schrock** (ex-Facebook, co-creator of GraphQL) ⚠-knowledge (company founding not re-verified at a primary source this pass). Market note: **Prefect's site announced an acquisition of Dagster Labs** (prefect.io banner, seen August 2026) ⚠ date unverified — see §2.3 and the claims audit; consolidation is happening in this category.

### 2.3 Prefect

**Prefect** is the Python-native workflow orchestration framework, founded 2018 by **Jeremiah Lowin** ⚠-knowledge, with the GitHub repository (PrefectHQ/prefect, Apache-2.0) created **29 June 2018** ✅. The defining architectural event is the **Prefect 2.0 / "Orion" rewrite** — a from-scratch engine (dynamic workflows, native async, first-class task caching, a far simpler deployment model) released in 2022 ⚠-knowledge (widely documented; the launch blog URL 404'd this pass). As of August 2026 the prefect.io site carries an acquisition announcement banner for Dagster Labs ⚠ (see §2.2) — a sign of the consolidation in the Python orchestration tier.

### 2.4 Temporal

**Temporal** is the stateful durable-execution engine — a different category from the DAG batch schedulers (see §3.1). The project's own README verifies the lineage: **"Temporal is a mature technology that originated as a fork of Uber's Cadence. It is developed by Temporal Technologies, a startup by the creators of Cadence."** ✅ The server repository (temporalio/temporal, **MIT license**) was created **16 October 2019** ✅. Temporal Technologies was founded in 2019 by **Maxim Fateev and Samar Abbas**, the engineers who built Cadence at Uber — ⚠-knowledge (founding pair widely documented, not re-verified at a primary source this pass). This repository's [Temporal Workflow](temporal_workflow_guide.md) guide is the deep-dive companion; here Temporal matters as the open-source answer to *stateful, long-running, exactly-once-effect* flows — payment orchestration, settlement runs, KYC case workflows — that batch schedulers express badly.

### 2.5 Kestra

**Kestra** is the declarative, event-driven orchestration platform: workflows are YAML (with a UI builder), scheduled and event-driven triggers are first-class, and the platform leans on Git-based versioning and a rich plugin ecosystem. The GitHub repository (kestra-io/kestra, Apache-2.0) was created **24 August 2019** ✅; the README verified live describes it as "an open-source, event-driven orchestration platform for data, AI, and infrastructure workflows… declarative YAML interface". ✅ The company (Kestra Technologies) is widely reported as founded around 2020 by **Ludovic Dehon** ⚠ (founder and founding year not re-verified at a primary source this pass). For Control-M refugees, Kestra's YAML-declarative model is the closest open-source analogue to Control-M's job-definition-as-configuration style.

### 2.6 Apache DolphinScheduler

**Apache DolphinScheduler** is the distributed, low-code data orchestration platform with a drag-and-drop workflow UI, multi-master/multi-worker HA, and tens-of-millions-of-tasks-per-day positioning. The Apache Incubator page verifies the timeline: the project **graduated as an Apache top-level project on 17 March 2021**. ✅ The repository (apache/dolphinscheduler, Apache-2.0) was created **1 March 2019** — the donation date. ✅ The pre-Apache origin is widely reported as **Analysys (易观), the Chinese data-analytics company, which donated the project to Apache in 2019** — ⚠ (origin not re-verified at a primary source this pass; the incubator page verified live confirms the graduation but the sponsor/origin detail did not render). Its UI-driven, dependency-heavy model makes it one of the more Control-M-like open-source options for teams that do not want to write Python.

### 2.7 Azkaban

**Azkaban** is the batch workflow scheduler created at **LinkedIn** for Hadoop job orchestration, with a distinctive web UI for scheduling and visualizing job flows, and job files (`.job` properties) as the definition format. The repository (azkaban/azkaban, Apache-2.0) was created **18 October 2012** — the open-sourcing date ✅ (GitHub API, primary). The LinkedIn origin is ⚠-knowledge (extensively documented, not re-verified at a primary source this pass). Azkaban's mainframe-era parallels are strong — jobs, dependencies, scheduled flows — which makes it a *conceptually* easy read for Control-M operators, but its Hadoop-centric heritage and comparatively small community limit its banking appeal today.

### 2.8 Apache Oozie

**Apache Oozie** is the Hadoop workflow scheduler, donated to Apache by Yahoo, with workflows defined as XML (control-flow and action nodes forming a DAG). The Apache Attic page verifies the timeline precisely: **Oozie became a Top Level Project in August 2012 and was retired in February 2025**, with the move to the Attic completed in April 2025. ✅ (Note: the commonly repeated "top-level 2015" is wrong — the Attic minutes say August 2012; the task brief's guess was corrected against the primary source.) The repository mirror (apache/oozie) dates to **14 September 2011** (the donation era) ✅. Oozie is now a *retired* Apache project — which is itself a decision-framework data point: XML-config-driven schedulers of the Hadoop era are being sunset, and any estate still on Oozie should treat migration as mandatory, not optional.

### 2.9 Rundeck

**Rundeck** is the open-source operations/runbook automation platform: jobs defined as ordered steps (commands, scripts, workflows) with an access-controlled web UI, scheduled or on-demand, aimed at IT operations self-service rather than data pipelines. The repository (rundeck/rundeck, Apache-2.0) was created **3 September 2010** ✅. The product is widely attributed to **Damon Edwards and Alex Honor** (SimplifyOps) ⚠-knowledge, and the commercial steward was **acquired by PagerDuty in 2020** ⚠ (widely reported April 2020; the PagerDuty press pages 404'd this pass, so it is flagged rather than asserted). Rundeck is a credible target for the *ops-runbook* slice of a WLA estate (file transfers, restarts, runbook automation) that doesn't belong in a data orchestrator.

### 2.10 Luigi

**Luigi** is the Python pipeline framework created at **Spotify** for long-running batch pipelines, built around tasks with `requires()` dependencies and a central scheduler with a web UI. The repository (spotify/luigi, Apache-2.0) was created **20 September 2012** ✅. The Spotify origin is ⚠-knowledge (well documented). Luigi is the historical ancestor of the Python-DAG school (Airflow's direct predecessor in spirit) and remains in production in many data teams, but it is effectively in maintenance mode relative to Airflow/Prefect/Dagster — fine to know, rarely the right *target* for a 2026 migration.

### 2.11 The Lightweight Tier — Windmill and n8n

Two lightweight/lower-code tools round out the landscape for the *long tail* of automation that neither a batch scheduler nor a durable-execution engine should own:

- **Windmill** — an open-source developer platform that turns scripts into workflows, cron jobs, webhooks and UIs (repo created **5 May 2022**, windmill-labs/windmill, source-available license — ⚠ license class not Apache; "NOASSERTION" on GitHub API this pass). ✅-partial
- **n8n** — the fair-code, node-based workflow automation platform (repo created **22 June 2019**, n8n-io/n8n, fair-code/sustainable-use license, not Apache-2.0) ✅. n8n is the open-source answer to Zapier-style integrations — exactly the glue that a WLA estate accumulates informally (email-to-ticket, file-watch-to-notify) and that should never be scheduled on the mainframe.

Both are verified by GitHub API creation dates ✅; neither is a Control-M *replacement* — they are complements for the edges of the estate (see §3.2 and the worked example §10).

### 2.12 The Open-Source Landscape at a Glance

| Platform | Category | Definition style | Repo created (GitHub API) | License | Control-M affinity |
|---|---|---|---|---|---|
| Apache Airflow | DAG batch orchestration | Python code | 2015 (started at Airbnb Oct 2014; TLP Jan 2019) | Apache-2.0 ✅ | Highest — the primary target |
| Dagster | Asset-based data orchestration | Python code + asset graph | 2018-04-30 ✅ | Apache-2.0 ✅ | High (data assets) |
| Prefect | Python workflow orchestration | Python code | 2018-06-29 ✅ | Apache-2.0 ✅ | High (dynamic flows) |
| Temporal | Stateful durable execution | Code (Go/Java/Python/TS…) | 2019-10-16 ✅ | MIT ✅ | Different category — complements |
| Kestra | Declarative event-driven orchestration | YAML + UI | 2019-08-24 ✅ | Apache-2.0 ✅ | High (config-driven like Control-M) |
| Apache DolphinScheduler | Distributed low-code data orchestration | UI drag-drop | 2019-03-01 ✅ (TLP 2021-03-17 ✅) | Apache-2.0 ✅ | High (UI/dependency model) |
| Azkaban | Batch workflow scheduler | .job properties files | 2012-10-18 ✅ | Apache-2.0 ✅ | Medium (conceptually close) |
| Apache Oozie | Hadoop workflow scheduler | XML | 2011-09-14 ✅ (TLP Aug 2012 ✅; **retired Feb 2025 ✅**) | Apache-2.0 ✅ | Low — retired |
| Rundeck | Ops runbook automation | Job steps (commands/scripts) | 2010-09-03 ✅ | Apache-2.0 ✅ | Medium (ops slice) |
| Luigi | Python pipeline framework | Python tasks | 2012-09-20 ✅ | Apache-2.0 ✅ | Low (maintenance mode) |
| Windmill | Scripts→workflows/cron/UI | Scripts + low-code | 2022-05-05 ✅ | Source-available ⚠ | Low — edge automation |
| n8n | Node-based integration automation | Visual nodes | 2019-06-22 ✅ | Fair-code ⚠ | Low — glue/integrations |

### 2.13 Managed Tiers at a Glance

For the bank evaluating *deployment posture* (the §3.4 decision), the managed options per platform, with the notable launch facts verified this pass:

| Platform | Managed tier(s) | Verified launch facts | Self-host viable? |
|---|---|---|---|
| Airflow | Amazon MWAA, Google Cloud Composer, Azure managed Airflow, Astronomer | MWAA November 2020 ✅; Cloud Composer May 2018 ✅ (Wikipedia) | ✅ — the default banking posture |
| Dagster | Dagster Cloud (Dagster Labs) | — | ✅ |
| Prefect | Prefect Cloud | — | ✅ |
| Temporal | Temporal Cloud | — | ✅ (server is MIT, self-hostable) |
| Kestra | Kestra Cloud / Enterprise | — | ✅ |
| DolphinScheduler | No first-party managed tier of note | — | ✅ (designed self-hosted) |

### 2.14 Evaluating an Open-Source Target — the Bank Checklist

Before a target platform is allowed anywhere near the WLA estate, the §3.7 five questions get operationalized into a checklist the Cymbal Bank WLA committee uses (and that any bank can copy):

- [ ] License file reviewed by legal: Apache-2.0/MIT preferred; fair-code/source-available (n8n, Windmill) requires a documented exception decision ✅-review
- [ ] Apache TLP status or equivalent governance durable (Airflow TLP 2019 ✅, DolphinScheduler TLP 2021 ✅, Oozie retired 2025 ✅ — governance *changes*, re-check at adoption time)
- [ ] HA architecture documented and load-tested for the bank's month-end profile
- [ ] Run history and logs exportable to the bank's SIEM/log archive (§8 audit trail)
- [ ] Secrets integration (vault) supported — no credentials in workflow definitions
- [ ] Backup/restore and DR runbook for the scheduler's own metadata store
- [ ] Version-upgrade path and compatibility policy understood (Airflow's breaking-change discipline is real — pin and test)
- [ ] Community support channels mapped to the bank's support model (vendor SLA for the managed tier, or an internal SME rotation for self-hosted)
- [ ] Exit cost documented: self-host the open core, export definitions, re-point workers

### 2.15 Why Apache TLP Status Matters to a Bank

The difference between "a GitHub project with Apache-2.0" and "an Apache top-level project" is not legal — the license is the same — it is *governance durability*: a TLP is owned by a foundation with a board, a PMC, release processes, trademark protection, and a documented path that keeps the project alive if the commercial steward disappears. For a bank whose batch estate will run on the platform for a decade, that durability is worth real money. The evidence in this pass is concrete: **Oozie, a TLP since August 2012, was retired by the ASF in February 2025** ✅ — the foundation *can* and *will* retire projects, but it does so with years of notice, an Attic home, and no license disruption; and DolphinScheduler's graduation (17 March 2021 ✅) shows the path in the other direction. Airflow's TLP status since January 2019 ✅, plus its scale of adoption, is why it is the default target in this guide rather than merely the most popular one. The corollary for the checklist: verify TLP status at adoption time, and treat "Apache-2.0 but not an Apache project" (Kestra, Prefect, Dagster, Temporal, n8n, Windmill) as a *steward-risk* item — fine when the steward is healthy, worth an exit plan regardless (§3.7 Q5, §2.14).

---

## 3. The Comparison Framework

The open-source landscape above is not one market — it is four overlapping ones. Comparing a Control-M estate against it fails if you treat all targets as "schedulers". The framework below names the axes that actually separate the tools; §4 turns the axes into a selection matrix.

### 3.1 DAG-Based Batch Orchestration vs Stateful Workflow Engines

This is the single most important distinction in the guide. 

- **DAG-based batch orchestration** (Airflow, Dagster, Prefect, DolphinScheduler, Azkaban, Oozie, Luigi, Kestra): a workflow is a *static or semi-static graph* of tasks; the scheduler fires task instances on a schedule or trigger; each task is a *stateless* unit that reads inputs, does work, writes outputs; the graph is re-evaluated per run. This maps almost one-to-one onto Control-M's mental model — jobs, dependencies, calendars, run cycles. Batch, EOD, and data-pipeline workloads belong here.
- **Stateful durable-execution engines** (Temporal — and, in the commercial world, AWS Step Functions, Azure Durable Functions): a workflow is *code that runs as a long-lived process*, and the engine persists every step's state so the process survives restarts, crashes, and multi-hour waits. The engine guarantees that activities execute with at-least-once semantics and effects are idempotent-friendly. This maps onto the *long-running business flows* that batch schedulers fake with "condition" loops and sleep jobs: payment-settlement orchestration, loan-origination journeys, KYC case flows, compensation/reconciliation sweeps, cross-system saga flows.

A Control-M estate contains both kinds of workload. The classic error is to force the second kind into the first (a batch DAG whose "tasks" are actually long-running state machines, held together with file flags and timeouts — precisely the Control-M pattern §5 maps away from).

### 3.2 Data-Pipeline Focus vs General Scheduling

- **Data-pipeline-first** (Airflow, Dagster, Prefect, DolphinScheduler): built around data assets, datasets, sensors, and warehouse/object-store integration. Dagster is the most asset-centric (software-defined assets, lineage); Prefect is the most dynamic-flow-centric; Airflow is the most mature and generic.
- **General scheduling / operations** (Rundeck, n8n, Windmill, and to a large degree Kestra): run scripts, commands, runbook steps, integrations — not data assets. Rundeck is the ops-runbook tool; n8n and Windmill are the glue tier.

The WLA estate's *data* slices (feeds, warehouses, regulatory reporting) and its *ops* slices (file transfers, restarts, housekeeping) need different homes. A single "one scheduler to rule them all" is exactly the enterprise-WLA assumption the open-source world rejects.

### 3.3 Code-First vs Config-Driven

- **Code-first** (Airflow, Dagster, Prefect, Temporal, Luigi): workflows are programs — Python (or Go/Java/TS for Temporal), versioned in Git, tested in CI. The cost is a steeper learning curve for operations staff; the payoff is expressiveness, testability, and reviewable change management (which banking audit teams like — code review *is* change control).
- **Config-driven / declarative** (Kestra YAML, Azkaban `.job` properties, Oozie XML, DolphinScheduler UI, n8n/Windmill visual): workflows are configuration or drag-drop artifacts. This is the closest match to Control-M's job-definition model (definitions in a control file, calendar rules, dependencies) and to the *operator* skill set — but config-only systems tend to hit a complexity wall on branching, error handling, and dynamic logic, which is why Kestra pairs YAML with code plugins and why the config-driven pure-play (Oozie) was retired.

Control-M teams migrating should expect a *skill shift*, not a like-for-like tool swap: the biggest hidden cost of any code-first target is retraining scheduler operators into Python/Go developers or, alternatively, choosing a config-driven target (Kestra, DolphinScheduler) to preserve the operator profile.

### 3.4 Self-Hosted vs Managed Offerings

Every major open-source target has a commercial managed tier — the "open-core" pattern:

- Airflow: Amazon MWAA (since November 2020 ✅, per Wikipedia), Google Cloud Composer (2018 ✅), Azure's managed Airflow offering, plus Astronomer.
- Dagster: Dagster Cloud (steward: Dagster Labs / Elementl lineage).
- Prefect: Prefect Cloud.
- Temporal: Temporal Cloud.
- Kestra: Kestra Enterprise/Cloud.
- DolphinScheduler: no first-party managed tier of note; it is primarily self-hosted.

For a bank the managed-vs-self-hosted decision is *not* primarily about cost — it is about **data residency, outsourcing classification, and operational resilience** (see §8 and the [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide). Self-hosting an Apache-2.0 scheduler in your own region/VPC keeps the boundary clean but puts the HA burden on you; a managed tier moves that burden to a vendor but adds a regulated-outsourcing arrangement. Both are defensible; the mistake is to decide without the compliance overlay.

### 3.5 Community Health and License Notes

- **Apache-2.0** (permissive, patent-grant, banking-lawyer-friendly): Airflow, Dagster, Prefect, Kestra, DolphinScheduler, Azkaban, Oozie, Rundeck, Luigi — all verified in §2 via GitHub API license metadata ✅.
- **MIT**: Temporal ✅ (verified in its README/license badge).
- **Source-available / fair-code** (NOT Apache-2.0 — read the license before adopting): n8n (fair-code/sustainable-use) and Windmill (source-available) ⚠ — verified only as "NOASSERTION" in the GitHub API this pass; treat as non-Apache until the license file is reviewed.
- **Community health signals**: Airflow (~46.6k stars), Prefect (~23.7k), Temporal (~22.6k), n8n (~200k — the largest of all), Kestra (~27.9k), Windmill (~17.7k), Dagster (~16.1k), DolphinScheduler (~14.5k), Luigi (~18.8k), Rundeck (~6.3k), Azkaban (~4.5k), Oozie (~0.7k and retired). Star counts are a *popularity* proxy, not a governance proxy — the governance facts that matter for banking are: Apache TLP status (Airflow 2019 ✅, DolphinScheduler 2021 ✅, Oozie retired 2025 ✅), the steward company's viability (Prefect's announced acquisition of Dagster Labs ⚠), and the commercial tier's longevity (open-core vendors do fail; the Apache TLP projects are the ones the foundation will keep alive).

### 3.6 Maturity

| Tier | Platforms | Signal |
|---|---|---|
| Battle-tested decade+ | Airflow (2014), Luigi (2012), Azkaban (2012), Rundeck (2010) | Production at scale for 10+ years |
| Established second wave | Temporal (2019), Prefect (2018), Dagster (2018) | Production-proven, actively consolidated |
| Younger but fast | Kestra (2019/2020), DolphinScheduler (2019/2021), Windmill (2022), n8n (2019) | Growing fast; validate before banking-critical use |
| Retired | Oozie (TLP 2012, retired Feb 2025) | Do not adopt; migrate off |

### 3.7 Banking-Appropriateness — the Five Questions

When a bank's WLA committee evaluates any target platform, five questions decide everything (elaborated with the compliance cross-refs in §8):

1. **Audit trail** — is every run, trigger, parameter, and state transition recorded immutably and queryable? (Airflow's run history, Temporal's event history, Dagster's run/asset logs all qualify *if* the logs are shipped to the bank's SIEM/log archive.)
2. **Retries and recovery** — are retry policies explicit, bounded, and idempotency-aware, and is restart/resume from failure controlled (no silent duplicate effect)?
3. **SLA guarantees** — does the platform give you *observable* SLAs (scheduled-vs-actual start, duration percentiles, missed-run detection) rather than best-effort cron? (Control-M's forecasting → the open-source world's run history and SLA dashboards — see §5.)
4. **Run visibility** — can operations see the full estate on one screen, with per-run drill-down, without vendor-proprietary consoles?
5. **Exit cost** — if the steward company or the managed tier disappears, what is the cost to self-host or move? (Apache-2.0 + no proprietary lock-in = low exit cost; this is the single biggest structural advantage of open source over the incumbents in §1.)

### 3.8 The Governance Reality — Who Answers the Phone at 3 a.m.

The question that ends more open-source evaluations in banks than any technical gap is operational ownership. A Control-M estate has a vendor support contract and a named account team; an Airflow estate has — what? The honest answer, and the pattern the Cymbal Bank worked example adopts:

- **Self-hosted core (the banking default):** the bank owns the platform — scheduler HA, metadata DB backups, upgrades, and the 3 a.m. page go to an internal support rotation, exactly as they would for any Tier-1 application. This is *not* a downgrade; it is the same operating model the bank already runs for its core databases and middleware. The open-source community provides the knowledge base; the bank provides the duty roster. Budget for it: roughly one platform engineer per scheduler platform, plus the operator team that owns the workloads.
- **Managed tier:** the vendor owns the platform and the SLA; the bank owns the *workloads* (DAGs, retries, calendars). This is cleaner operationally but adds a regulated-outsourcing arrangement (vendor due diligence, exit clauses, data-residency checks — per the [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide's outsourcing overlay).
- **The hybrid that most banks actually choose:** self-host the core schedulers inside the bank's VPC/region (residency and resilience control), use managed tiers only for *non-critical* analytics workloads, and keep a documented self-host fallback for every managed tier in use (§2.14 exit-cost checklist).

The 3 a.m. test has a second half: *who can change a definition at 3 a.m. under incident conditions, and what is the audit trail of that change?* Git-reviewed DAGs with emergency-deploy procedures answer it better than vendor consoles do — but only if the emergency procedure is written down and rehearsed before the first incident, not after.

## 4. The Target-Selection Matrix

| Scenario (concrete) | Wins | Why |
|---|---|---|
| Batch/EOD job streams with fixed dependencies and calendars (the classic Control-M estate) | **Airflow** | Mature DAG model, huge community, every integration exists; the companion [Control-M → Airflow Migration Guide](control_m_to_airflow_migration_guide.md) maps it 1:1 |
| Data-asset pipelines where lineage and data-ops matter (warehouse, lakehouse, regulatory reporting assets) | **Dagster** | Asset graph, lineage, asset-level observability |
| Dynamic, branching, event-driven Python pipelines (per-customer flows, parameterized runs) | **Prefect** | Dynamic workflow model, native async, simple deployments |
| Stateful long-running business flows (settlement orchestration, KYC journeys, saga compensations) | **Temporal** | Durable execution, event history, exactly-once-effect activities; complements rather than replaces the batch layer |
| Operators-first shop that wants config/UI-driven definitions, minimal Python | **Kestra** or **DolphinScheduler** | Declarative YAML / drag-drop UI with Control-M-like dependency model |
| Ops runbook automation (restarts, file transfers, self-service ops) | **Rundeck** | Access-controlled runbook jobs, audit-friendly |
| Glue/integration long tail (notifications, ticket automation, lightweight webhooks) | **n8n** / **Windmill** | Low-code, fast, cheap — keep it off the schedulers |
| Hadoop-era XML workflows | **Anything but Oozie** | Oozie is retired (Feb 2025 ✅); migrate off, typically to Airflow |
| Mainframe-boundary workloads | See §9 | No open-source agent runs *inside* z/OS JES; use bridges |
| Calendar-heavy, holiday-aware job streams (bank holidays, T+1/T+2 settlement calendars) | **Airflow** (custom `Timetable`) or **Kestra** (declarative schedules) | Holiday calendars become data (a Git-backed holiday table) consumed by the scheduler — the Control-M calendar model survives as configuration |
| File-transfer-heavy cycles (SFTP sweeps, vendor file pickup/drop) | **Airflow** with `SFTPOperator`/`FileSensor`, or **Rundeck** for ad-hoc ops transfers | File-watch sensors replace Control-M's file-wait conditions 1:1 (§5) |
| AI/ML pipeline orchestration (training, evaluation, feature backfills) | **Airflow** or **Dagster** | Dagster's asset model fits ML lineage; Airflow's maturity fits the batch core |
| Disaster-recovery / rerun discipline (re-run yesterday's batch, replay a failed stream) | **Airflow** (backfill, rerun, `catchup` semantics) + **Temporal** (workflow replay) | The Control-M "rerun/restart" discipline maps to backfill and replay — design the replay policy up front (§6.4) |
| Cyclic/wait jobs and "run until condition" streams | **Temporal** (durable loop with signals) | The #1 mis-fitted workload in a batch estate (§5 last row) |
| Multi-team, multi-tenant scheduling with per-team quotas and RBAC | **Airflow** (teams/folders/RBAC) or **DolphinScheduler** (project/tenant model) | Control-M's folder/role model maps to either; decide on the RBAC matrix before pilot |

**The two-platform default** (and the one the worked example uses): **Airflow for batch/DAG workloads + Temporal for stateful long-running flows**, with Rundeck or n8n as optional ops/glue complements. Everything else is a specialization you justify with the criteria in §3.

**How to read the matrix.** The rows are *workload shapes*, not tools you already own — classify the estate by shape (§6.1's inventory), then let each shape pick its platform; the §10.2 scoring matrix is what you produce when two platforms tie on shape (as Airflow and Kestra do for operator-fit). Two rows deserve emphasis because they are the ones most estates get wrong: *cyclic/wait jobs* do not belong in a DAG scheduler at all (they are Temporal-shaped, §3.1), and *mainframe-boundary workloads* are not "scheduler problems" — they are bridge-engineering problems (§9). A matrix that maps every Control-M job stream onto one of these rows, with no leftovers and no forced fits, is the sign the estate has been understood rather than merely renamed.

## 5. The Condensed Control-M Concepts Mapping

One table — the *full* Control-M → Airflow mapping (job definition YAML, run cycles, forecasting, agent topology, condition handling, the Automation API) lives in the companion [Control-M → Airflow Migration Guide](control_m_to_airflow_migration_guide.md) and is not re-derived here.

| Control-M concept | Open-source equivalent (primary target in bold) | Notes |
|---|---|---|
| Job (the schedulable unit) | **Airflow task** (DAG node); Dagster op/asset; Temporal activity | One Control-M job → one task/activity; one Control-M job stream → one DAG/workflow |
| Run cycle / calendar (when it runs) | **Schedule** (`schedule_interval`/cron, `timetable`); Temporal `Schedule`/cron workflow | Calendar exclusions (holidays) become `timezone`/custom `Timetable` logic or a holiday table |
| Job dependency / in-condition (success, failure, completion) | **Task dependency** (`>>`, `set_downstream`); Airflow `TriggerRule` | Control-M's *out-condition* style ("job X sets condition Y") → explicit DAG edges or `ExternalTaskSensor` |
| Condition / quantum / prompt | **Sensor / trigger / event** (Airflow `BaseSensorOperator`; Kestra trigger; Temporal signal) | File-watch, time-window, and cross-flow conditions become sensors or Temporal signals |
| Control-M Agent (the execution endpoint) | **Worker / executor** (Airflow Celery/Kubernetes executors; Temporal worker) | Agent install/registration → worker pools, queues, and Kubernetes worker nodes |
| Control-M forecasting (projected run times, "what if") | **Run history / observability** (Airflow run list, Gantt, SLA callbacks; Temporal event history) | Forecasting is replaced by scheduled-vs-actual telemetry plus SLA monitoring |
| Control-M/EM (enterprise manager console) | **Web UI** (Airflow UI, Dagster UI, Temporal Web UI) | No single "enterprise manager" — observability is per-platform, aggregated in the bank's own monitoring stack |
| Control-M job definition (JSON/YAML via Automation API) | **DAG definition as code** (Python; Kestra YAML) | Definitions move into Git — this is the change-management win (§8) |
| Cyclic job (runs until condition) | **Temporal workflow** (durable loop) or Airflow backfill/poke | Cyclic/wait jobs are the #1 sign the workload belongs in Temporal, not a DAG |

### 5.1 A Worked Mapping — One Control-M Job Stream to One Airflow DAG

A concrete example fixes the pattern. Consider a classic Cymbal Bank job stream, `PAY_EXT_SWIFT_EOD` (Control-M notation):

- **Job 1** `SWIFT_HEADER_EXTRACT` — runs daily 22:30 SGT, extracts the day's SWIFT messages from the payments store to a landing file.
- **Job 2** `SWIFT_MT103_BUILD` — waits on Job 1's success (in-condition), builds the MT103 outbound file.
- **Job 3** `SWIFT_FTP_TO_MEPS` — waits on Job 2's success, SFTPs the file to the MEPS+ exchange; on failure, sets an alert condition `SWIFT_EOD_FAILED` that pages the payments on-call.
- **Job 4** `SWIFT_ACK_RECONCILE` — runs at 23:45, waits on the *out-condition* `SWIFT_FTP_OK` set by Job 3, reconciles the ack file.

The Airflow DAG is the same graph, expressed as code:

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.sensors.filesystem import FileSensor
from airflow.providers.sftp.operators.sftp import SFTPOperator

with DAG(
    "pay_ext_swift_eod",
    schedule="30 22 * * *",            # Control-M run cycle → cron, Asia/Singapore
    start_date=datetime(2026, 1, 1),
    catchup=False,                     # no silent backfill of missed days
    tags=["payments", "eod", "swift"],
) as dag:
    extract = PythonOperator(task_id="swift_header_extract", python_callable=extract_swift_headers)
    build = PythonOperator(task_id="swift_mt103_build", python_callable=build_mt103)
    ftp = SFTPOperator(task_id="swift_ftp_to_meps", ssh_conn_id="meps_sftp", remote_filepath="/inbound/MT103.dat")
    ack = PythonOperator(task_id="swift_ack_reconcile", python_callable=reconcile_acks)

    extract >> build >> ftp >> ack      # the in-conditions become DAG edges
    ack >> FileSensor(task_id="wait_ack_file", filepath="/landing/swift_ack/*.dat", timeout=3600)
```

The translation notes that matter for the mapping review: the run cycle became a cron schedule in the bank's timezone; the in-conditions became `>>` edges; the file-wait condition became a `FileSensor`; the alert condition (`SWIFT_EOD_FAILED`) became Airflow alerting/on-call integration on task failure; and the 23:45 "run after FTP regardless" timing became a *dependency* (ack waits on ftp), not a wall-clock time — which is exactly the semantic upgrade the migration delivers: **the DAG encodes *why* a job runs, not just *when*.**

## 6. The Phased Migration Pattern — Inventory, Map, Parallel-Run, Cutover

The pattern is deliberately conservative: **inventory → map → parallel-run → cutover**, with each phase producing an artifact the next phase consumes and a regulator can review.

### 6.1 Phase 1 — Inventory

Produce the machine-readable inventory of the entire WLA estate: every job/job stream, its scheduler of origin (Control-M, TWS, CA-7, AutoSys…), its run cycle, its dependencies (including *implicit* ones — file handoffs, out-conditions, time windows), its owner, its criticality tier, and its last-run data. The inventory is the single most valuable deliverable of the whole migration; most estates discover 20–40% more jobs than the license inventory claimed. Export via each scheduler's API (Control-M Automation API, TWS command line, CA-7 control statements) into a Git-backed catalog.

### 6.2 Phase 2 — Map

Classify every job stream against the target model (§3.1): **batch/DAG-shaped → Airflow; stateful/long-running/cyclic → Temporal; ops runbook → Rundeck; glue → n8n; mainframe-resident → stays or bridges (§9)**. Then produce the per-stream mapping artifacts: the DAG sketch, the schedule/calendar translation, the dependency translation, the retry/error-handling policy, and the data contract (what files/tables the stream consumes and produces — the reconciliation key for parallel run). Define the translation conventions once (naming, folder structure, tagging, owner metadata, SLA metadata) so every DAG looks like it came from the same factory — the Control-M "job stream" discipline survives as a *naming and packaging* convention.

### 6.3 Phase 3 — Parallel-Run

Run the mapped streams on the new platform *while the old scheduler still runs them*, comparing outcomes run-by-run (see §7 for the reconciliation machinery). The rule of thumb: a stream graduates from parallel-run to cutover when it has N consecutive identical outcomes (N typically 10–20 runs, or one full month-end cycle — month-end is the hardest case and the best test). Never cut over a stream that has not survived a month-end/quarter-end parallel run.

### 6.4 Phase 4 — Cutover

Cut over per stream or per business domain (not per scheduler): freeze changes to the old definition, flip the trigger source, keep the old scheduler in *observe-only* mode for one full cycle, then retire the old definition. The scheduler *platforms* are retired last, only after the final stream leaves — and only after the audit evidence (parallel-run reconciliation reports) is archived.

### 6.5 The Month-End Special Case

Month-end (and quarter-end, year-end) is not just a bigger batch — it is a *different workload shape*: more streams, longer critical paths, currency/FX revaluation steps, regulatory submissions that cannot slip, and usually a calendar that collides with the first-of-month settlement runs. The migration rules that fall out:

1. **Map month-end streams first.** If month-end is the hardest run, it is the one you want in parallel-run longest. The Cymbal Bank timeline (§10.3) deliberately puts the first full month-end parallel run on *migrated domains only*, then a second month-end with *all* domains, before any cutover.
2. **Load-test the month-end profile, not the daily profile.** A platform that handles the daily batch with 50% headroom can fail month-end with 300% load. Test the December profile in September — with time to fix.
3. **Treat month-end as a release freeze window.** No cutovers on month-end day, ever; cutovers happen on the weekend after a successful month-end, not before an unproven one.
4. **Keep the old scheduler's month-end definitions warm until the new platform has survived two month-ends.** Observe-only mode exists for exactly this: the definitions stay, the triggers are off, and the month-end parallel evidence accrues.

The metric that gates everything: **month-end batch completion parity** — same completion time or better, same outputs, zero manual interventions on the new platform — for two consecutive month-ends, per domain.

### 6.6 Roles and Operating Model

A migration of this shape changes the operating model, and the RACI should be written before the first DAG is. The Cymbal Bank pattern, generalizable to any bank:

| Role | Responsibility during migration | Responsibility after cutover |
|---|---|---|
| **WLA platform team** (new) | Stand up and run Airflow + Temporal platforms (HA, upgrades, capacity) | Own the platforms end-to-end (the 3 a.m. rotation of §3.8) |
| **Scheduler operators** (existing 14) | Run the parallel-run reconciliation ledger, observe both engines, investigate ⚠ diffs | Become the workload owners: calendars, SLA metadata, runbook discipline, on-call for batch incidents |
| **Data engineers / DAG authors** (new) | Write and review DAGs, own the translation conventions | Own definitions-as-code through the normal SDLC |
| **Mainframe team** | Own the z/OSMF bridge contract and the z/OS side of the boundary (§9.1) | Keep owning z/OS execution; the bridge is a Tier-1 service |
| **Compliance & audit** | Gate the phases: sign off the inventory, the mapping, the ledger, the cutover evidence | Ongoing: audit the scheduler logs as part of the normal IT audit cycle |
| **Vendor account teams (BMC, IBM, Broadcom)** | Support the old schedulers through parallel run; no definition changes after freeze | Retired with the licenses |

The operating-model rule that prevents the migration from stalling: **the operators' job is the ledger, not the DAGs.** If the existing scheduler operators are also expected to become Python authors overnight, the parallel run stops being a comparison and becomes a rewrite — §6.2's dual-track skill plan (bootcamp for some, Kestra fallback for the rest) exists precisely to keep the two jobs separate.

## 7. Coexistence and Reconciliation During Parallel Run

During parallel run the bank operates two schedulers against the same business calendar — this is where migrations quietly fail. The controls that keep it honest:

- **Single source of truth for the calendar**: both schedulers must derive their run schedules from the same holiday calendar and timezone rules, or "identical outcome" comparisons are meaningless.
- **Outcome comparison, not log comparison**: compare *business outcomes* (the file arrived, the table was loaded, the count matched, the SLA met) rather than raw scheduler logs — the two engines will never produce identical logs, and they should not.
- **A reconciliation ledger**: each stream gets a run-ID pair (old-run vs new-run) with a comparison status (✅ identical / ⚠ diff — investigated / ❌ failed on one side). The ledger is the audit artifact regulators ask for; it doubles as the cutover checklist.
- **Dependency handoff discipline**: while both schedulers run, cross-scheduler dependencies (a Control-M job waiting on an Airflow task's output, or vice versa) must be expressed as *file/condition handoffs* with a written contract — never as scheduler-to-scheduler calls. The bridging pattern: the upstream scheduler writes a completion marker (file/table row/HTTP callback); the downstream scheduler consumes it via a sensor/trigger (§5).
- **Failure ownership**: when the two engines disagree, the *old* scheduler remains the system of record until cutover — the parallel run exists to find the new platform's gaps, not to double the blast radius. Documented per-incident.
- **Observability layering**: aggregate both engines' run status into the bank's existing monitoring/alerting stack (the [Cybersecurity](cybersecurity_guide.md) guide's change-management and monitoring conventions apply unchanged), so operations sees one estate, not two consoles.

### 7.1 The Reconciliation Ledger in Practice

The ledger is a table, not a system — it starts as a spreadsheet and becomes a small database-backed report once the run volumes justify it. Per stream, per run date:

| Stream | Run date | Old-run ID (Control-M) | New-run ID (Airflow) | Outcome comparison | Status | Investigation |
|---|---|---|---|---|---|---|
| PAY_EXT_SWIFT_EOD | 2026-09-01 | 20260901-2230-1142 | dag_run_id=2026-09-01T22:30+08 | File hash + row count + ack arrival identical | ✅ | — |
| REG_MAS610_SUBMIT | 2026-09-01 | 20260901-0200-8819 | dag_run_id=2026-09-01T02:00+08 | Submission timestamp 02:47 vs 02:52 (SLA 03:00) | ✅ | New platform faster; no action |
| GL_CLOSE_EOD | 2026-09-01 | 20260901-2330-0071 | dag_run_id=2026-09-01T23:30+08 | GL trial balance row count differs by 12 rows | ⚠ | Investigating — suspected calendar difference (Sep public holiday rule); old scheduler is system of record until resolved |
| DEP_ACCRUAL_CYCLE | 2026-09-01 | — | dag_run_id=2026-09-01T01:15+08 | New-only run (stream not yet in parallel-run scope) | ❌ | Out of scope — ledger entry is a checklist reminder, not a failure |

Three ledger disciplines keep it honest: (1) every ⚠ row gets an owner and a 24-hour investigation SLA; (2) ❌ rows are *scope* markers (streams not yet in parallel-run), never silent; (3) the ledger's ✅-rate per domain is the objective cutover-readiness metric presented to the Technology Committee each month — the same number the §10.3 gates consume.

---

## 8. The Banking Angle — EOD Batch Windows, Audit Trails, Change Management

Banks run WLA under a compliance and operational-resilience overlay that the open-source world does not provide out of the box — it must be *built* around the platform. Three obligations dominate (the regulatory deep-dive lives in the [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide; cross-ref, do not re-derive):

- **EOD batch windows and operational resilience.** The overnight batch is a regulated business process: MAS operational-resilience expectations (and the equivalent regimes in other jurisdictions) require that critical batch processing complete within defined windows, recover from failure within defined RTOs, and never silently lose a run. The migration implications: (a) the target platform's HA (Airflow's scheduler HA with a shared metadata DB, Temporal's clustered history service) must be engineered to the same standard as the incumbent's; (b) missed-run detection (an Airflow `SlaMiss`/`DagRun` monitor, a Temporal schedule heartbeat) replaces Control-M forecasting as the "did the batch start and finish" alarm; (c) month-end/quarter-end load must be load-tested *before* parallel run, not discovered during it.
- **Audit trails.** Everything a scheduler does is auditable: run starts/ends, parameters, retries, manual interventions, definition changes. Control-M's audit log and CA-7's SMF-based records set the bar. On the open-source side the raw material exists (Airflow's `dag_run`/`task_instance` tables and logs, Temporal's complete event history per workflow, Dagster's run/asset event log) — but the bank must ship those logs to its enterprise log archive (SIEM), retain them per record-keeping policy, and prove *tamper-evidence* the same way it does for any other system log. This is an integration deliverable, not a platform feature.
- **Change management.** This is where open source *wins* structurally: Control-M definitions live in a vendor control database with its own change workflow; an Airflow/Dagster/Kestra definition is *code in Git*, which means the bank's existing change-management pipeline (branch → review → CI tests → approval → deploy) applies with no new machinery. The [Cybersecurity](cybersecurity_guide.md) guide's change-management and privileged-access conventions (§9 of that guide) carry over unchanged: definitions and secrets (DB credentials, API keys — via a vault, never in DAG code) follow the same controls as application code. The RBS/NatWest 2012 failure that Wikipedia cites under CA-7 — a batch-definition change that cascaded through a weekend — is precisely the class of incident that Git-reviewed, CI-tested DAG changes are designed to prevent.

### 8.1 The MAS-Resilience Angle in Practice

The [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide is the deep-dive; the scheduler-specific reading of it, for the migration program, is:

- **Critical systems inventory**: the batch schedulers (old *and* new, during parallel run) are critical systems; their RTO/RPO, DR testing, and incident reporting obligations apply to both engines simultaneously — the parallel-run period *doubles* the resilience surface, and the DR plan must say which scheduler is primary and how the other is quiesced in a failover.
- **Outsourcing**: a managed-tier vendor (MWAA, Temporal Cloud, Astronomer) is a material outsourcing arrangement — vendor due diligence, data-residency confirmation (Singapore region or approved jurisdiction), exit provisions, and notification obligations per the MAS outsourcing guidelines. Self-hosting inside the bank's VPC avoids the outsourcing classification for the *platform* (the workloads and their data stay inside the bank's boundary) — a real, and often decisive, argument for the self-hosted default of §3.8.
- **Business continuity**: the scheduler's own metadata store (Airflow's database, Temporal's history/visibility stores) is a recovery point — its backup/restore and the "rebuild a scheduler from Git + metadata backup in X hours" runbook are resilience artifacts that auditors will ask for, alongside the batch-completion SLAs of §8 bullet 1.
- **Change governance**: the migration itself is a change program — the quarterly gates of §10.3 are change tickets with evidence, not project milestones; the reconciliation ledger (§7.1) is the change record that survives the program.

**The mainframe boundary is the caveat on all of the above**, and it is treated in §9.

## 9. The Mainframe Boundary — What Open Source Can and Cannot Do on z/OS

The honest boundary, stated plainly: **no mainstream open-source WLA platform runs natively inside z/OS, and none integrates natively with JES2/JES3 batch**. Control-M z/OS and CA-7 exist because the mainframe batch world speaks JCL/JES, SMF, and z/OS security (RACF/ACF2/Top Secret); an Airflow worker or a Temporal worker is a Linux process and cannot be one of those. What open source *can* do:

| Capability | Can open source do it? | How |
|---|---|---|
| Schedule and trigger z/OS batch jobs | ✅ (at the boundary) | From Linux: submit JCL via z/OSMF jobs REST API, SSH/3270 automation, or an integration layer; the open-source scheduler owns the *orchestration*, z/OS owns the *execution* |
| Monitor z/OS job completion | ✅ | Poll z/OSMF job status or consume SMF-derived events via a bridge; Airflow sensors / Temporal activities wrap the poll |
| File handoff to/from the mainframe | ✅ | SFTP (FTPS/connect:direct-style) bridges, file-watch sensors on landing zones, MQ message handoffs — the standard pattern |
| Run *inside* JES as a first-class batch job | ❌ | No open-source agent installs into the z/OS address space; there is no "open-source Control-M Agent for z/OS" |
| Replace Control-M z/OS's mainframe-native features | ❌ | Native z/OS console automation, SMF-based audit integration, and deep JCL introspection stay proprietary-mainframe territory |
| Provide SLA/run visibility across the whole estate including mainframe | ✅ | The open-source scheduler's run history + the bank's monitoring stack aggregate mainframe jobs via the bridge — this *replaces* Control-M forecasting at the estate level |

The architectural consequence: **in a post-migration estate the mainframe is an external dependency of the open-source schedulers, not a peer scheduler**. The batch *orchestration* layer (what runs when, with what dependencies, with what retries) moves to Airflow/Temporal; the mainframe contributes *execution services* (the JCL batch that only z/OS can run) exposed through z/OSMF/MQ/SFTP endpoints. Control-M z/OS and CA-7 either remain as z/OS-local schedulers behind the bridge, or — where the JCL streams themselves are being modernized — are retired along with the workloads they schedule. This is a *strategy* decision a bank must make explicitly; the migration pattern of §6 supports either outcome, because the bridge contract (trigger + completion marker) is the same.

### 9.1 The Bridge Pattern in Sequence

The bridge is the mechanism that lets an Airflow DAG own a z/OS job. The canonical sequence, with the contract points a bank must freeze:

1. **Trigger** — the Airflow task calls the z/OSMF Jobs REST API (`POST /zosmf/restjobs/jobs`) with the JCL (either stored on z/OS or submitted from a template in Git) and the substituted parameters. Alternatively, for environments without z/OSMF exposure, an MQ message or an SFTP-drop of a control file triggers a z/OS-side agent (e.g., a started task or an existing CA-7/Control-M z/OS job scheduled to "wait for trigger").
2. **Wait** — the Airflow task polls the job status (`GET .../jobs/{jobname}/{jobid}`) on a bounded interval, or a `FileSensor` waits for a completion marker; in Temporal the equivalent is an activity that polls with retries and timeouts.
3. **Complete** — on normal completion the bridge writes the completion marker the *next* task depends on; on abnormal completion it writes a failure marker with the job's return code and step, and the DAG's retry/alert policy takes over (the Control-M "on failure, notify and hold" behavior becomes Airflow retries + alerting).
4. **Audit** — the bridge logs the request/response pairs (job ID, submit time, end time, return code) into the run's log stream; the z/OS side's SMF records remain the authoritative execution record, and the reconciliation between the two (SMF job list vs bridge ledger) is part of month-end parity checks.

The failure modes to engineer for: z/OSMF unavailability (the bridge must fail *loudly* with an alert, never silently skip), duplicate submission (the bridge must be idempotent — a retried task must not submit the JCL twice; the z/OS job name + date stamp is the dedup key), and security (the bridge's z/OS credentials are vault-held and scoped — per the [Cybersecurity](cybersecurity_guide.md) guide's privileged-access conventions).

## 10. Cymbal Bank Worked Example — WLA Estate Rationalization

### 10.1 The Starting Estate

Cymbal Bank (Singapore) runs a three-scheduler WLA estate, the product of two decades of acquisition and organic growth:

- **BMC Control-M** — the dominant scheduler: ~4,800 distributed jobs (Control-M/Agents on Linux/Windows) covering EOD feeds, regulatory reporting (MAS 610, MAS 1003-style submissions), the data warehouse load, and SWIFT/MEPS+ file cycles; plus Control-M z/OS for ~1,200 mainframe batch jobs (core banking EOD, GL close).
- **IBM Workload Scheduler (TWS/IWS)** — ~1,100 jobs in the payments and cards domains, legacy of a regional acquisition.
- **CA-7 (Broadcom)** — ~900 z/OS jobs in the deposits and lending domains, legacy of an older core-system migration.

Total: ~8,000 job definitions across three consoles, four agent fleets, and two mainframe images — with the classic symptoms: duplicated calendars, implicit file-handoff dependencies nobody documents, per-agent licensing renewals that grow every year, and a scheduler-operator team of 14 whose tribal knowledge is the real system of record. The mandate from the Cymbal Bank Technology Committee: **consolidate the WLA estate onto open-source targets, eliminate the commercial scheduler licenses by end of 2027, and keep the mainframe batch running without regression** — under the Cymbal Bank governance conventions (the [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide's outsourcing/audit overlay and the [Cybersecurity](cybersecurity_guide.md) guide's change-management overlay apply).

The estate in one table, as the inventory phase (§6.1) would capture it:

| Domain | Scheduler | Jobs | Shape | Migration target |
|---|---|---|---|---|
| Regulatory reporting | Control-M | ~1,400 | Fixed EOD DAGs, strict SLAs, month-end peaks | Airflow (P0 — mapped first) |
| Payments & SWIFT/MEPS+ | Control-M + TWS | ~1,600 | Fixed DAGs + long-running settlement journeys | Airflow (DAGs) + Temporal (journeys) |
| Cards | TWS | ~1,100 | Daily/real-time mix, cyclic reconciliation loops | Airflow + Temporal (cyclic loops) |
| Data warehouse & lakehouse | Control-M | ~1,300 | Data-pipeline DAGs, backfills | Airflow (asset-aware DAGs; Dagster evaluated) |
| Deposits & lending | CA-7 (z/OS) | ~900 | Mainframe JCL batch | Bridged via z/OSMF (Airflow owns orchestration) |
| Core banking EOD & GL close | Control-M z/OS | ~1,200 | Mainframe JCL batch, month-end critical | Bridged via z/OSMF; CA-7/Control-M z/OS retirement decision at Q3 2027 |
| Ops runbooks & glue | ad-hoc scripts | ~500 | Restarts, transfers, notifications | Rundeck + n8n (edge complements) |

### 10.2 The Decision Matrix Used

The committee scored candidate targets against the §3 framework with weights reflecting Cymbal Bank's constraints (weight 5 = critical). Scores are 1–5; the matrix is the *reasoned record* of the decision, kept as an audit artifact:

| Criterion (weight) | Airflow | Temporal | Dagster | Prefect | Kestra | DolphinScheduler |
|---|---|---|---|---|---|---|
| DAG/batch fit for the Control-M estate (5) | 5 | 2 | 4 | 4 | 4 | 4 |
| Stateful long-running flow fit (4) | 2 | 5 | 2 | 3 | 2 | 2 |
| Operator skill match — Python vs config/UI (3) | 3 | 2 | 3 | 3 | 5 | 5 |
| Mainframe-boundary bridgeability (5) | 5 | 4 | 3 | 3 | 4 | 3 |
| Banking audit/observability maturity (4) | 5 | 4 | 4 | 3 | 3 | 3 |
| Community/ecosystem size (3) | 5 | 4 | 4 | 4 | 4 | 3 |
| License posture — Apache-2.0/MIT (3) | 5 | 5 | 5 | 5 | 5 | 5 |
| Managed-tier optionality (2) | 5 | 4 | 4 | 4 | 3 | 1 |
| **Weighted total** | **169** | **139** | **132** | **131** | **139** | **128** |

**Decision:** Airflow (batch/DAG workloads — the Control-M/TWS job streams and the z/OS bridge orchestration) + **Temporal** (stateful flows — settlement orchestration, the payments-domain long-running journeys currently faked with TWS cyclic jobs). Kestra scored well on operator fit and is the documented fallback if the Python skill ramp stalls; DolphinScheduler was the runner-up on UI fit but lost on ecosystem depth. Rundeck and n8n were approved as *edge complements* (ops runbooks, glue) without entering the core consolidation scope.

### 10.3 The Phased Timeline (Quarters)

The §6 pattern, scheduled in quarters from Q1 2026 (kickoff) to Q4 2027 (license exit):

| Quarter | Phase | Deliverables / gates |
|---|---|---|
| Q1 2026 | Inventory | Machine-readable estate catalog (~8,000 jobs), owner/criticality tagging, implicit-dependency discovery; baseline "batch completion by 06:30 SGT" SLA metrics |
| Q2 2026 | Map | Translation conventions (naming, folder, tagging, SLA metadata); per-domain mapping artifacts; mainframe bridge design (z/OSMF + SFTP/MQ handoffs); **gate: 100% of P0 streams mapped and reviewed** |
| Q3 2026 | Map + pilot parallel-run | Airflow and Temporal platforms stood up (self-hosted, bank VPC, HA); first two domains (regulatory reporting, payments) in parallel-run; reconciliation ledger live |
| Q4 2026 | Parallel-run scale-out | 6 of 12 domains in parallel-run; month-end 2026 full parallel run on the migrated domains; **gate: month-end parity on migrated domains** |
| Q1 2027 | Parallel-run completion + first cutovers | All domains in parallel-run; cutover of regulatory reporting and payments domains; Control-M z/OS bridge in production |
| Q2 2027 | Cutover wave 2 | Deposits, cards, GL-close domains cut over; TWS estate empty and decommissioned (license exit #1) |
| Q3 2027 | Cutover wave 3 + mainframe decision | Remaining domains cut over; CA-7 z/OS workloads either bridged or retired with their legacy core jobs; Control-M estate empty (license exit #2) |
| Q4 2027 | Platform retirement + closeout | Control-M and remaining agents decommissioned; parallel-run evidence archived for audit; runbooks and runbooks' runbooks (the ops tribal knowledge) documented; **gate: 12 consecutive weeks of full-estate SLA attainment** |

### 10.4 The Risk Register

Follows the Cymbal Bank risk-register convention (risk, likelihood, mitigation — see the repo's banking guides):

| Risk | Likelihood | Mitigation |
|---|---|---|
| Month-end batch overruns on the new platforms under peak load | Medium-High | Load-test month-end profiles in Q3 2026 *before* the December parallel run; HA metadata DB and worker pools sized to 2× peak; SLA dashboards from day one |
| Operator skill gap — scheduler operators → Python authors | High | Dual-track plan: Python bootcamp for 8 of 14 operators (Q2–Q3 2026); Kestra fallback documented for the domains that cannot staff Python; hiring 2 data engineers |
| Mainframe bridge instability (z/OSMF/SFTP handoffs) breaks cross-boundary dependencies | Medium | Bridge contract frozen in Q2 2026; completion-marker protocol (file + heartbeat) with idempotent consumption; bridge monitored like a Tier-1 service |
| Reconciliation drift during parallel run (old/new outcomes diverge) | Medium | Outcome-based comparison (not log comparison); reconciliation ledger per stream with ⚠-diff SLA of 24h to investigate; old scheduler remains system of record until cutover (§7) |
| Regulator/audit scrutiny of the migration itself | Medium | Every phase produces an auditable artifact (inventory, mapping, ledger, cutover evidence); decision matrix (§10.2) retained as the reasoned record; MAS outsourcing/audit overlay per the [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md) guide |
| Steward-company risk in the open-core ecosystem (e.g., consolidation moves) | Low-Medium | Apache TLP targets for the core (Airflow) plus MIT/Apache-2.0 self-hostable Temporal; exit cost to self-host documented; vendor lock-in is structurally low (§3.7 Q5) |
| Scope creep — "while we're at it" job rewrites during mapping | Medium | Change freeze on job definitions during Q2–Q4 2026 except regulatory-mandated changes; rewrites are a separate program, logged not absorbed |

### 10.5 The End State

At the close of Q4 2027, if the gates hold, the Cymbal Bank WLA estate looks like this:

- **Two open-source platforms, self-hosted in the bank's VPC**: Airflow (the batch/DAG core — ~6,500 distributed jobs migrated from Control-M and TWS) and Temporal (the stateful flows — settlement journeys, cyclic reconciliation loops, and the z/OS bridge's long-running waits).
- **The mainframe as an external dependency**: z/OS still runs the JCL batch (core banking EOD, GL close, deposits/lending), now triggered and monitored through the z/OSMF bridge; Control-M z/OS and CA-7 are either in observe-only retirement or decommissioned per the Q3 2027 mainframe decision — the orchestration of mainframe work is no longer a mainframe product's job.
- **Edge automation** on Rundeck (ops runbooks) and n8n (glue), deliberately out of the core platforms' scope.
- **One observability estate**: run status from both platforms plus the bridge aggregated into the bank's monitoring stack; Control-M forecasting replaced by scheduled-vs-actual SLA dashboards; the reconciliation ledger archived as the migration's audit record.
- **The commercial license line**: Control-M, TWS, and CA-7 licenses — agents, MIPS, and support — removed from the budget; the platform cost is now engineering time, which is a cost the bank can *reduce* by standardization instead of a cost that grows per server.

The end state is not "no mainframe" and not "no batch" — it is **no vendor-controlled batch**, which is the entire point of the exercise: the same jobs, the same calendars, the same month-ends, owned by the bank's own code and the bank's own operating model, on the open-source rhythm.

---

## 11. The Claims Audit

✅ = verified live this pass against a primary/cited source. ⚠ = flagged/unverified/approximate. ⚠-knowledge = well-documented industry knowledge not re-verified live this pass. ❌ = claim checked and found incorrect.

| # | Claim | Status | Source |
|---|---|---|---|
| 1 | KKR completed its acquisition of BMC Software on 2 October 2018 (announced 29 May 2018) | ✅ | media.kkr.com/news-details (KKR press release); bmc.com/newsroom |
| 2 | Control-M created in the 1980s by New Dimension Software (Israel); BMC acquired New Dimension 1999 (~US$675M) | ✅-repo | [control_m_guide.md](control_m_guide.md) §2 (repository's own verified Control-M guide) |
| 3 | Tivoli Systems Inc. founded 1989 in Austin, TX by Bob Fabbio; merged into IBM in 1996 | ✅ | en.wikipedia.org/wiki/Tivoli_Software |
| 4 | IBM Tivoli Workload Scheduler renamed IBM Workload Scheduler as of release 9.3; Tivoli brand phased out from 2013 | ✅ | en.wikipedia.org/wiki/Tivoli_Software |
| 5 | TWS distributed engine originated as Maestro from Unison Software; Tivoli acquired Unison 1997 (~US$170M) | ⚠ | softpanorama.org (secondary source only) |
| 6 | Broadcom announced acquisition of CA Technologies for US$18.9B cash on 11 July 2018 | ✅ | investors.broadcom.com (press release) |
| 7 | Computer Associates founded 1976 by Charles B. Wang and Russell Artzt; defunct 2018 (acquired by Broadcom) | ✅ | en.wikipedia.org/wiki/CA_Technologies |
| 8 | CA-7 was UCC-7 from UCCEL; Computer Associates acquired UCCEL in 1987 and renamed the product CA-7 | ✅ | en.wikipedia.org/wiki/CA-7_(software) |
| 9 | CA-7 commonly used by banks (Wikipedia cites the 2012 RBS/NatWest batch failure) | ✅ | en.wikipedia.org/wiki/CA-7_(software) |
| 10 | AutoSys originated at AutoSystems Corp, acquired by CA in 1997 | ⚠ | widely cited; not re-verified at a primary source this pass |
| 11 | Stonebranch founded ~1999 (celebrated 25 years in September 2024) | ⚠ | bizwire-distributed press release headline (markets.financialcontent.com) — article body did not render |
| 12 | Stonebranch is a German-founded company, HQ Atlanta area | ⚠ | task brief + industry directories; not re-verified at a primary source this pass |
| 13 | Redwood Software founding year | ⚠ | could not verify — no Wikipedia article; redwood.com pages 404/blocked |
| 14 | ActiveBatch (Advanced Systems Concepts) founding year | ⚠ | could not verify — Wikipedia title redirects to a 2011 trademark case; vendor pages blocked |
| 15 | JAMS Scheduler released in 1987, developed within MVP Systems Software | ✅ | jamsscheduler.com/about ("the same JAMS Scheduler that was released in 1987… as part of MVP Systems Software") |
| 16 | JAMS separated from Fortra in June 2025, becoming JAMS Software owned by PSG, 2ndWave and employees | ✅ | jamsscheduler.com/about |
| 17 | Airflow started at Airbnb in October 2014; Apache Incubator March 2016; top-level January 2019; Apache-2.0 | ✅ | en.wikipedia.org/wiki/Apache_Airflow |
| 18 | Amazon MWAA launched November 2020; Google Cloud Composer launched May 2018 | ✅ | en.wikipedia.org/wiki/Apache_Airflow (cites AWS/Google announcements) |
| 19 | Dagster repository created 30 April 2018, Apache-2.0 | ✅ | api.github.com/repos/dagster-io/dagster |
| 20 | Elementl founded 2018 by Nick Schrock (Dagster's commercial steward) | ⚠-knowledge | widely documented; company founding not re-verified at a primary source this pass |
| 21 | Prefect repository created 29 June 2018, Apache-2.0 | ✅ | api.github.com/repos/PrefectHQ/prefect |
| 22 | Prefect 2.0 ("Orion" rewrite) released 2022 | ⚠-knowledge | widely documented; the launch blog URL 404'd this pass |
| 23 | Prefect's site announces acquisition of Dagster Labs | ⚠ | prefect.io banner seen August 2026; date/details not verified |
| 24 | Temporal "originated as a fork of Uber's Cadence; developed by Temporal Technologies, a startup by the creators of Cadence" | ✅ | github.com/temporalio/temporal README (verified live) |
| 25 | Temporal server repository created 16 October 2019, MIT license | ✅ | api.github.com/repos/temporalio/temporal |
| 26 | Temporal Technologies founded 2019 by Maxim Fateev and Samar Abbas | ⚠-knowledge | widely documented; not re-verified at a primary source this pass |
| 27 | Kestra repository created 24 August 2019, Apache-2.0; declarative YAML event-driven orchestration | ✅ | api.github.com/repos/kestra-io/kestra; kestra-io/kestra README |
| 28 | Kestra founded ~2020 by Ludovic Dehon | ⚠ | not re-verified at a primary source this pass |
| 29 | Apache DolphinScheduler graduated as top-level project on 17 March 2021 | ✅ | incubator.apache.org/projects/dolphinscheduler.html |
| 30 | DolphinScheduler repository created 1 March 2019 (donation date), Apache-2.0 | ✅ | api.github.com/repos/apache/dolphinscheduler |
| 31 | DolphinScheduler originated at Analysys (易观), donated to Apache in 2019 | ⚠ | not re-verified at a primary source this pass |
| 32 | Azkaban repository created 18 October 2012, Apache-2.0 | ✅ | api.github.com/repos/azkaban/azkaban |
| 33 | Azkaban created at LinkedIn | ⚠-knowledge | extensively documented; not re-verified this pass |
| 34 | Apache Oozie became a Top Level Project in August 2012 | ✅ | attic.apache.org/projects/oozie.html (**note: the commonly repeated "2015" is incorrect — corrected against the Attic minutes**) |
| 35 | Apache Oozie retired by the ASF in February 2025; Attic move completed April 2025 | ✅ | attic.apache.org/projects/oozie.html; en.wikipedia.org/wiki/Apache_Oozie |
| 36 | Rundeck repository created 3 September 2010, Apache-2.0 | ✅ | api.github.com/repos/rundeck/rundeck |
| 37 | Rundeck created by Damon Edwards and Alex Honor (SimplifyOps); acquired by PagerDuty in 2020 | ⚠ | widely reported; PagerDuty press pages 404'd this pass |
| 38 | Luigi repository created 20 September 2012, Apache-2.0 | ✅ | api.github.com/repos/spotify/luigi |
| 39 | Luigi created at Spotify | ⚠-knowledge | well documented; not re-verified this pass |
| 40 | n8n repository created 22 June 2019; fair-code license (not Apache-2.0) | ✅ / ⚠ | api.github.com/repos/n8n-io/n8n (creation ✅; license shows "NOASSERTION" — fair-code class flagged ⚠ pending license-file review) |
| 41 | Windmill repository created 5 May 2022; source-available license | ✅ / ⚠ | api.github.com/repos/windmill-labs/windmill (creation ✅; license class ⚠) |
| 42 | GitHub star counts as a popularity proxy (Airflow ~46.6k, n8n ~200k, Kestra ~27.9k, Temporal ~22.6k, Prefect ~23.7k, Dagster ~16.1k, DolphinScheduler ~14.5k, Windmill ~17.7k, Luigi ~18.8k, Rundeck ~6.3k, Azkaban ~4.5k, Oozie ~0.7k) | ✅ | api.github.com (same pass as the creation dates) |
| 43 | No mainstream open-source WLA platform runs natively inside z/OS/JES | ✅-knowledge | architectural fact (Linux-based workers); consistent with all sources in this pass |

## 12. What Could Not Be Verified

- **AutoSys → CA acquisition year (1997)** (#10) — the most-cited gap; the CA Technologies Wikipedia history section did not mention AutoSys, and searches were rate-limited this pass.
- **Stonebranch founding year and German founding narrative** (#11–12) — the 25-years press-release headline verified, but the article body did not render; the vendor about page gives no founding year.
- **Redwood Software founding year** (#13) — no Wikipedia article exists; vendor site URLs 404'd or blocked automated access.
- **ActiveBatch / Advanced Systems Concepts founding year** (#14) — Wikipedia has no product article (the title redirects to a court case); vendor pages blocked access.
- **Prefect 2.0/Orion exact release date** (#22) and the **Prefect–Dagster Labs acquisition details** (#23) — the launch blog 404'd; the acquisition is attested only by the prefect.io banner seen this pass (date, terms, and completion status unknown).
- **Elementl (Dagster) and Temporal founding specifics** (#20, #26) — founders and years are widely documented but no primary source was reachable this pass.
- **DolphinScheduler's Analysys (易观) origin** (#31) — the incubator page verified the graduation but the origin narrative did not render.
- **Rundeck's PagerDuty acquisition date** (#37) — press pages 404'd; flagged rather than asserted.
- **n8n and Windmill license classes** (#40–41) — GitHub API returned "NOASSERTION"; the fair-code/source-available labels come from vendor materials that were not re-extracted this pass; a license-file review is the documented next step before adoption.
- **Kestra's founding year and founder** (#28) — flagged ⚠ pending a primary source.
- **Unison/Maestro acquisition details** (#5) — only a secondary source (softpanorama) was reachable.

## 13. Glossary and References

### Glossary

| Term | Definition |
|---|---|
| **WLA** | Workload automation — the planning, execution, monitoring and recovery of scheduled IT workloads (job scheduling, batch scheduling); the category the enterprise products in §1 belong to |
| **DAG** | Directed acyclic graph — the workflow structure used by Airflow/Dagster/Prefect/Oozie/Azkaban: tasks as nodes, dependencies as edges, no cycles |
| **EOD batch** | End-of-day batch processing — the overnight run (settlement, GL close, reporting) that defines a bank's batch window |
| **JES2/JES3** | z/OS job entry subsystems — the mainframe batch execution environment; the boundary open source cannot cross natively (§9) |
| **JCL** | Job Control Language — the z/OS batch job definition language |
| **SMF** | System Management Facilities — z/OS records (including job accounting) that feed mainframe audit trails |
| **Run cycle / calendar** | Control-M's scheduling rule: when and how often a job runs, including holiday exclusions |
| **Condition / quantum** | Control-M's dependency token: a job sets or waits on conditions that gate other jobs |
| **Forecasting** | Control-M's projected-run-time/"what-if" planning feature — replaced by run-history and SLA telemetry in the open-source world (§5) |
| **Control-M Agent** | The endpoint process that executes jobs on a managed host — the analogue of an Airflow worker/Temporal worker |
| **Sensor** | An Airflow (and Dagster/Prefect) task type that waits for an external event (file, table, API) before downstream tasks run |
| **Durable execution** | The Temporal model: workflow code whose state is persisted by the engine, surviving process restarts and long waits (§3.1) |
| **Activity** | A Temporal unit of work executed by a worker, with retries and idempotency contracts |
| **Open-core** | The business model: an open-source core (Apache-2.0/MIT) plus a paid managed/commercial tier (§3.4) |
| **Fair-code / source-available** | Licenses that allow source visibility but restrict use (n8n, Windmill) — not open source in the OSI sense; review before adoption (§3.5) |
| **Reconciliation ledger** | The parallel-run artifact pairing old-scheduler runs with new-scheduler runs and their outcome comparison status (§7) |
| **z/OSMF** | z/OS Management Facility — the REST API surface used to submit/monitor z/OS jobs from Linux in the bridge pattern (§9) |
| **Control-M/EM** | Control-M Enterprise Manager — the product's central console, replaced by per-platform web UIs plus the bank's aggregated monitoring (§5) |
| **Out-condition / in-condition** | Control-M's dependency tokens: a job's completion *sets* out-conditions that downstream jobs consume as in-conditions — the ancestor of DAG edges (§5) |
| **Catchup / backfill** | Airflow's re-run of past schedule slots — the disciplined form of Control-M's "rerun/restart" (§6.4, §4 DR row) |
| **SLA (scheduled-vs-actual)** | The run-telemetry comparison (scheduled start vs actual start, duration, missed runs) that replaces Control-M forecasting in the open-source estate (§5, §8) |
| **RTO / RPO** | Recovery time objective / recovery point objective — the resilience targets the scheduler's own DR design must meet (§8.1) |
| **MEPS+** | MAS Electronic Payment System — Singapore's interbank payment and settlement system; a typical mainframe-adjacent file-exchange endpoint in Cymbal Bank's estate (§10.1) |

### References

1. KKR completes acquisition of BMC — https://media.kkr.com/news-details?news_id=7e493384-45ad-4dd6-966c-a00e2ec9d69f&type=1 ; https://www.bmc.com/newsroom/releases/kkr-completes-acquisition-bmc-software.html
2. Broadcom to acquire CA Technologies, US$18.9B (11 July 2018) — https://investors.broadcom.com/news-releases/news-release-details/broadcom-acquire-ca-technologies-189-billion-cash
3. Tivoli Software history (1989 founding, 1996 IBM merger, 9.3 rebrand) — https://en.wikipedia.org/wiki/Tivoli_Software
4. CA-7 (UCC-7/UCCEL 1987; banking usage) — https://en.wikipedia.org/wiki/CA-7_(software)
5. Computer Associates / CA Technologies (1976 founding; Broadcom fate) — https://en.wikipedia.org/wiki/CA_Technologies
6. JAMS about (1987 release; MVP Systems; June 2025 Fortra spin-out) — https://www.jamsscheduler.com/about/
7. Apache Airflow history (Airbnb Oct 2014; Incubator Mar 2016; TLP Jan 2019) — https://en.wikipedia.org/wiki/Apache_Airflow
8. Apache Oozie status (TLP Aug 2012; retired Feb 2025; Attic Apr 2025) — https://attic.apache.org/projects/oozie.html ; https://en.wikipedia.org/wiki/Apache_Oozie
9. Apache DolphinScheduler incubation status (graduated 2021-03-17) — https://incubator.apache.org/projects/dolphinscheduler.html
10. Temporal README (Cadence fork lineage; Temporal Technologies) — https://github.com/temporalio/temporal (README, master)
11. Kestra README (declarative YAML, event-driven, Apache-2.0) — https://github.com/kestra-io/kestra (README, develop)
12. GitHub API repository metadata (creation dates, licenses, stars for azkaban/azkaban, spotify/luigi, apache/dolphinscheduler, kestra-io/kestra, dagster-io/dagster, PrefectHQ/prefect, temporalio/temporal, apache/airflow, apache/oozie, rundeck/rundeck, n8n-io/n8n, windmill-labs/windmill) — https://api.github.com/repos/{owner}/{repo}
13. Stonebranch 25-years press release headline — https://markets.financialcontent.com/stocks/article/bizwire-2024-9-10-stonebranch-celebrates-25-years-of-innovation-in-it-automation
14. Prefect site (acquisition banner, August 2026) — https://www.prefect.io/
15. Tivoli Workload Scheduler product documentation (IBM) — https://www.ibm.com/docs/en/workload-automation/9.2.0
16. Repo companions — [Control-M Comprehensive Guide](control_m_guide.md), [Control-M → Airflow Migration Guide](control_m_to_airflow_migration_guide.md), [Control-M Migration Options](control_m_migration_options_guide.md), [Temporal Workflow](temporal_workflow_guide.md), [Cybersecurity](cybersecurity_guide.md), [MAS Regulations Guidelines](../banking/mas_regulations_guidelines_guide.md)

---

**The final word.** The enterprise WLA incumbents of §1 are not going away — banks will run Control-M, TWS and CA-7 for years, and the mainframe boundary in §9 guarantees it. But the direction of travel is unmistakable: definitions move from vendor control databases into Git, dependencies move from conditions and quantums into DAG edges and durable workflow state, forecasting moves from a proprietary planning console into run-history telemetry on the bank's own dashboards, and the per-agent licensing bill that grows with every server becomes an Apache-2.0 platform the bank can self-host or buy managed. The decision framework in this guide — the comparison axes of §3, the selection matrix of §4, the mapping of §5, the phased pattern of §6–§7, and the compliance overlay of §8–§9 — is the map a bank like Cymbal Bank follows to get from a three-scheduler, three-license estate to a two-platform open-source estate without a single missed month-end. The worked example of §10 shows the destination is reachable in two years with the right gates and the right risk register. The journey from the 1980s mainframe schedulers ends in the open-source rhythm.
