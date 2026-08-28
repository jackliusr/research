# Migrating from BMC Control-M to Apache Airflow — the same batch, a new breath

> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
> **Topic:** End-to-End Migration Playbook — from BMC Control-M workload automation to Apache Airflow orchestration
> **Audience:** Solution Architects, Batch Operations Managers, Data Platform Engineers, Banking IT Decision-Makers
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Version:** 1.0 — August 2026

---

## Table of Contents

1. [Executive Summary — Why Banks Migrate Batch from Control-M to Airflow](#1-executive-summary--why-banks-migrate-batch-from-control-m-to-airflow)
2. [Control-M Platform Profile](#2-control-m-platform-profile)
3. [Apache Airflow Platform Profile](#3-apache-airflow-platform-profile)
4. [Semantic Mapping — Control-M to Airflow](#4-semantic-mapping--control-m-to-airflow)
5. [Migration Approach — Phased Methodology](#5-migration-approach--phased-methodology)
6. [Managed Airflow Options](#6-managed-airflow-options)
7. [Risks and Banking-Specific Concerns](#7-risks-and-banking-specific-concerns)
8. [Cymbal Bank Worked Example — EOD Batch Migration](#8-cymbal-bank-worked-example--eod-batch-migration)
9. [Claims Audit](#9-claims-audit)
10. [What Could Not Be Verified](#10-what-could-not-be-verified)
11. [Glossary](#11-glossary)
12. [References and Appendix](#12-references-and-appendix)

---

## 1. Executive Summary — Why Banks Migrate Batch from Control-M to Airflow

### Purpose of This Guide

This guide is the migration playbook for moving Cymbal Bank's end-of-day (EOD) batch estate from BMC Control-M, the enterprise workload automation (WLA) platform that has run our batch for two decades, to Apache Airflow, the open-source workflow orchestration platform. It is written for architects, batch operations managers, and platform engineers who need to understand *what* maps to *what*, *how* to sequence the migration safely, and *where the traps are* — especially in a regulated banking environment where the EOD batch is a control, not a convenience.

Control-M and Airflow both "run the batch," but they think about the problem differently:

- **Control-M is calendar- and condition-centric.** Jobs are defined in a database, attached to run cycles and calendars, and chained with IN-COND/OUT-COND conditions. The scheduler decides what runs next based on time and on which conditions have been set or deleted.
- **Airflow is code- and DAG-centric.** Workflows are Python files that declare a directed acyclic graph (DAG) of tasks and their dependencies. The scheduler parses those files and instantiates runs; dependencies are edges in the graph, not conditions in a database.

A migration is therefore not a lift-and-shift of definitions. It is a **semantic translation** — every Control-M construct must be re-expressed as an Airflow construct, and the translation must preserve the *intent* of the batch (what must complete, in what order, by what deadline) even when the mechanism differs. Sections 4 and 5 are devoted to exactly that translation and to a safe, phased way of doing it.

### Why Banks Migrate

| Driver | Detail |
|---|---|
| **Licensing cost** | Control-M is priced per agent and per job at enterprise scale; annual maintenance typically runs 20–25% of license cost. Airflow is Apache-2.0 open source; the main costs are the platform (self-hosted or managed) and the engineering team. |
| **Cloud and modernization strategy** | Banks are moving workloads to cloud and container platforms. Control-M supports cloud agents and a SaaS edition, but its center of gravity remains the classic scheduler. Airflow was born in the cloud/data-engineering era and deploys naturally to Kubernetes, with first-class operators for AWS, GCP, and Azure. |
| **Code-first development and DevOps** | Airflow DAGs live in Git, are reviewed in pull requests, are unit-tested with pytest, and are deployed through CI/CD. Control-M definitions live in a configuration database, exported and promoted through its own tooling. For banks standardizing on GitOps, Airflow aligns with the platform direction (see the [AI Platform Engineering Guide](ai_platform_engineering_guide.md)). |
| **Talent and ecosystem** | Airflow is the de facto standard for data orchestration, with a very large community, hundreds of provider packages, and abundant engineering talent. Control-M skills are rarer and concentrated in the WLA niche. |
| **Single orchestration plane** | Many banks run Control-M for batch *and* Airflow (or similar tools) for data pipelines, paying for two schedulers and two operation teams. Consolidating on Airflow removes the duplication. |
| **Vendor and platform risk** | BMC's portfolio has been through private-equity ownership and a 2024 split (BMC Software / BMC Helix); Control-M continues to be developed and sold, but some banks use the migration as an opportunity to reduce reliance on a legacy WLA vendor. |

None of this is an argument that Control-M is a bad product — it is the most battle-tested batch scheduler in banking, and for mainframe-centric shops it remains excellent. The argument is strategic: for a bank whose batch is increasingly distributed, whose data pipelines are already moving to open source, and whose cloud strategy is accelerating, Airflow is the more attractive long-term home for the same batch.

### What This Guide Contains

- Section 2 profiles Control-M (history, architecture, core constructs).
- Section 3 profiles Apache Airflow (history, concepts, execution model).
- Section 4 is the semantic mapping table — the heart of the translation.
- Section 5 is the phased migration methodology, including coexistence and cutover.
- Section 6 compares managed Airflow offerings (Amazon MWAA, Google Cloud Composer, Astronomer) against self-hosting.
- Section 7 covers banking-specific risks: scheduling drift, batch-window pressure, audit, mainframe boundary, data residency, secrets.
- Section 8 is a worked example: Cymbal Bank's real-world-shaped EOD migration of roughly 200 jobs.
- Sections 9–12 provide the claims audit, unverified items, glossary, and references.

---

## 2. Control-M Platform Profile

### Origins and History

Control-M was created by **New Dimension Software**, an Israeli software company, as a batch scheduling product for **IBM mainframes**. It was designed for the MVS/OS/390/z/OS world — scheduling JCL jobs, managing the batch window, and chaining jobs with conditions. BMC's own product documentation states that the product "was acquired from New Dimension Software in 1999, and was originally designed for IBM mainframe computers," and that it now also runs on distributed platforms including Unix, Windows, and Linux.

Key milestones (see the claims audit in Section 9 for sources):

- **1980s** — Control-M developed by New Dimension Software for mainframe batch scheduling.
- **1999** — BMC Software acquires New Dimension Software; Control-M becomes the core of BMC's workload automation portfolio.
- **2000s** — Control-M expands across distributed platforms (Unix, Windows, Linux, OpenVMS) and gains the Enterprise Manager (EM) console, calendars, SLA management, and Managed File Transfer capabilities.
- **2019** — BMC announces containerized deployment of Control-M (Docker) to simplify installation and cloud deployment.
- **2020** — BMC launches a SaaS edition of Control-M (BMC Helix Control-M) unifying on-premises and cloud deployments.
- **2024** — BMC splits into BMC Software (including the Digital Business Automation unit that owns Control-M) and BMC Helix; Control-M development continues under BMC Software.

The mainframe heritage is not trivia: it explains Control-M's vocabulary (run cycles, calendars, conditions), its reliability posture (designed to run a bank's overnight batch without fail), and its strengths (mature SLA handling, agent-based execution across heterogeneous platforms, deep JCL/z/OS integration).

### Architecture

| Component | Role |
|---|---|
| **Control-M/Server** | The scheduling engine: holds job definitions, calendars, dependencies, and the execution queue; decides what runs when. |
| **Control-M/Agent** | Lightweight agent installed on every execution host (z/OS via the mainframe agent, or distributed on Unix/Windows/Linux). Executes jobs and reports status back to the server. |
| **Control-M/EM (Enterprise Manager)** | The administration and monitoring console: GUI, dashboards, job status, alerts, and reporting. |
| **Control-M/Database** | Backend database (Oracle, MSSQL, DB2, or similar) storing definitions, run history, and statistics. |
| **Automation API** | REST API (introduced with Control-M 9.0.x) for programmatic control: define jobs, order jobs, query status, and manage the environment without the GUI. |
| **Control-M for MFT** | Managed File Transfer capability: SFTP/FTP/FTPS/AS2-style transfers defined as first-class job steps, monitored and SLA'd like any job. |
| **Control-M for Batch** | The core batch scheduling product; "Control-M" in its traditional sense. |

### Core Constructs

| Construct | Description |
|---|---|
| **Job** | The unit of work: a command, script, JCL member, executable, or file transfer. |
| **Folder / Job Group** | Logical grouping of jobs for management, permissions, and monitoring. |
| **Calendar** | Date rules for when jobs are eligible to run: working days, holidays, month-end, fiscal calendars. |
| **Run Cycle** | The scheduling pattern attached to a job — e.g., daily, weekday, month-end, every Nth weekday. Run cycles reference calendars. |
| **IN-COND / OUT-COND** | The dependency mechanism. A job has IN-CONDitions it *waits for* and OUT-CONDitions it *sets* (or deletes) when it completes. Job B starts when condition X (set by job A) exists. |
| **Quantitative SLA** | A deadline on a job or job group (e.g., must finish by 07:00) with escalation and alerting when missed. |
| **Resource** | A shared, limited-capacity asset (e.g., two concurrent DB license slots) that jobs acquire and release. |
| **Agent Pool** | A set of agents where a job can run on any available host. |
| **Order** | An instance of a job scheduled to run on a given day; ordering is how jobs are triggered (manually, by run cycle, or by API). |
| **Rerun / Force Run** | Operational actions to re-execute a job, typically re-setting its OUT-CONDitions. |

### The Control-M Model in One Paragraph

A Control-M production batch is a **condition graph layered over a calendar**. Every morning (or overnight) the scheduler *orders* the jobs whose run cycles match the day; each job waits on its IN-CONDitions, executes on an agent, and sets or deletes OUT-CONDitions as it finishes; jobs downstream wake up when their conditions are satisfied. Time and conditions are the two triggers, and everything — who ran, when, how long, with what return code — is recorded in the Control-M database for reporting and audit. It is a mature, deterministic model that has carried bank EOD processing for decades.

The full Control-M background is documented in the repo's [Control-M Guide](control_m_guide.md) and [Control-M Migration Options Guide](control_m_migration_options_guide.md); this section only recaps what the migration mapping in Section 4 needs.

---

## 3. Apache Airflow Platform Profile

### Origins and History

Apache Airflow was created at **Airbnb** by Maxime Beauchemin to manage the company's increasingly complex data workflows; Wikipedia dates the start to **October 2014**. It was made open source in **2015** (first release 1.0.0 on June 3, 2015; Beauchemin's public "Airflow: a workflow management platform" post is dated June 2, 2015). The project entered the **Apache Incubator in March 2016** and became a **top-level Apache Software Foundation project in January 2019**. Airflow 2.0 was released on **December 17, 2020**, and Airflow 3.0 on **April 22, 2025**. As of this writing (August 2026) the current stable line is 3.3.x (3.3.1 released August 12, 2026), with the 2.x line still maintained for enterprises that have not yet upgraded (2.11.2 released March 14, 2026).

Airflow is written in Python, licensed under Apache License 2.0, and designed around the principle of **configuration as code**: workflows are Python programs, so they can use loops, conditionals, libraries, and tests — none of which are available to XML- or database-defined schedulers.

### Core Concepts

| Concept | Description |
|---|---|
| **DAG (Directed Acyclic Graph)** | The workflow itself: a Python object declaring tasks and their dependency edges. A DAG has a schedule, an owner, retry policies, and a start date. |
| **Task** | A unit of work inside a DAG, instantiated from an Operator (or a decorated Python function). |
| **Operator** | The class that defines *what* a task does: `BashOperator`, `PythonOperator`, `SSHOperator`, `S3CopyObjectOperator`, and hundreds more from provider packages. |
| **Sensor** | A special operator that *waits*: `FileSensor` waits for a file, `ExternalTaskSensor` waits for a task in another DAG, `TimeSensor` waits until a time. Sensors are how Airflow expresses event-driven preconditions. |
| **XCom** | A small key-value message store that lets tasks pass data (e.g., a generated file name) to downstream tasks. |
| **Scheduler** | The process that parses DAG files, creates DAG runs and task instances per the schedule, and hands ready tasks to an executor. |
| **Executor** | The component that actually runs tasks: `SequentialExecutor` (one task at a time, for testing), `LocalExecutor` (multi-process on one host), `CeleryExecutor` (distributed workers via Celery), `KubernetesExecutor` (one pod per task). |
| **Metadata database** | A relational DB (PostgreSQL/MySQL) holding DAG runs, task instances, variables, connections, and run history. |
| **Webserver / UI** | The web interface: DAG graphs, grids, gantt, logs, and admin. |
| **Triggerer / Deferrable operators** | Since 2.2, tasks can *defer* and free their worker slot while waiting (async triggers), which is important for long waits like file arrivals. |
| **REST API** | Stable REST API (stable since Airflow 2.0) for programmatic control: trigger DAG runs, query status, list DAGs. |
| **Catchup / Backfill** | Mechanisms to run a DAG for past schedule dates: `catchup=True` runs all missed intervals; `backfill` explicitly replays a date range. |
| **TaskGroup** | A visual/logical grouping of tasks inside a DAG (a "sub-DAG" without the complexity). |

### The Airflow 2.x Scheduler Improvements

Airflow 2.0 (December 2020) was a major rewrite of the scheduler and execution model:

- The scheduler was rebuilt to **parse DAGs in a separate process** (the DAG processor) with a much faster, more reliable scheduling loop — a response to the 1.x scheduler's flakiness at scale.
- **Stable REST API** became a supported interface (it was experimental in 1.10).
- The **UI was rebuilt** (React-based) with a new grid view.
- TaskFlow API (Python decorators, `@task`) made data passing between tasks first-class without manual XCom plumbing.
- Later 2.x releases added deferrable operators/triggerer (2.2), the `airflow upgrade_check` tool (2.0 already shipped it), datasets/data-aware scheduling (2.4+), and stability improvements throughout.

Airflow 3.0 (April 2025) continued the trajectory: a split CLI, the Task SDK for multi-language/isolated task execution, an event-driven scheduling model, and a faster UI — while keeping the DAG-authoring model stable.

### A Minimal DAG

```python
from datetime import datetime, timedelta

from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

default_args = {
    "owner": "batch-ops",
    "retries": 1,
    "retry_delay": timedelta(minutes=5),
}

with DAG(
    dag_id="eod_payments_extract",
    schedule="15 1 * * *",          # 01:15 Asia/Singapore, daily
    start_date=datetime(2026, 1, 1),
    catchup=False,
    default_args=default_args,
    sla_miss_callback=notify_batch_ops,   # SLA-style deadline alerting
    tags=["eod", "payments"],
) as dag:

    extract = BashOperator(task_id="extract_payments", bash_command="run_extract.sh")
    validate = PythonOperator(task_id="validate_file", python_callable=validate_file)
    handoff = BashOperator(task_id="handoff_to_posting", bash_command="signal_posting.sh")

    extract >> validate >> handoff
```

Note how the last lines *declare the edges* — `extract >> validate >> handoff` — where Control-M would express the same relationship with OUT-COND `PAYMENTS_READY` and IN-COND on the downstream job.

### The Airflow Model in One Paragraph

Airflow is a **DAG-native orchestrator**: each workflow is a Python file that declares tasks and edges, and the scheduler materializes runs for each schedule interval. Everything is code, versioned in Git, deployed through CI/CD, and testable with pytest. Where Control-M centralizes definitions in a database and chains jobs with named conditions, Airflow distributes the definitions to the DAG repository and makes dependencies explicit edges in the graph. That explicitness is the migration's main opportunity — and its main discipline: **what Control-M expressed as a condition string, Airflow expresses as an edge, and the edge must be correct.**

---

## 4. Semantic Mapping — Control-M to Airflow

### The Mapping Table

This is the translation dictionary for the migration. Every Control-M construct maps to one or more Airflow constructs; the "semantic difference" column captures why the mapping is not mechanical.

| Control-M construct | Airflow construct | Semantic difference |
|---|---|---|
| Run cycle + calendar | `schedule` (cron expression or preset), `catchup`, start date; custom **Timetables** (Airflow 2.4+) for non-cron patterns | Control-M run cycles are calendar-aware definitions stored in the DB ("every weekday, except bank holidays, plus month-end"); Airflow schedules are cron expressions evaluated by the scheduler. Holidays must be encoded (e.g., a holiday-check task or a custom timetable), not assumed. |
| IN-COND / OUT-COND (job chaining) | Task dependency edges (`>>`), **trigger rules** (`all_success`, `all_done`, `one_failed`, `none_failed`), TaskGroups, `ExternalTaskSensor` | Control-M conditions are global named tokens set/deleted by jobs across the whole estate; Airflow edges are local to a DAG. Cross-DAG dependencies must be re-expressed as `ExternalTaskSensor` or as a shared upstream DAG. |
| Control-M Variables | Airflow **Variables**, `params`, XCom, or environment/secrets | Control-M variables are scheduler-side substitution values; Airflow Variables are stored in the metadata DB, `params` travel with a DAG run, XCom passes data between tasks. Decide scope early (variable vs secret — see Section 7). |
| Precondition / file-watch jobs (e.g., "wait for file F") | **Sensors**: `FileSensor`, `ExternalTaskSensor`, `TimeSensor`, `DateTimeSensor`, custom deferrable sensors | Control-M models a wait as a job (often a script loop); Airflow models it as a first-class sensor task. Deferrable sensors (triggerer) avoid holding a worker slot for the whole wait. |
| Control-M for MFT transfer steps | Transfer operators from provider packages: `SFTPOperator`, `SSHOperator`, cloud storage copy operators (`S3CopyObjectOperator`, `GCSToGCSOperator`), plus the repo's Axway transfer patterns (see [Axway CFT – Control-M Integration](axway_cft_controlm_integration.md)) | Control-M embeds MFT as monitored job steps with SLA; Airflow needs explicit operators and its own file-movement monitoring. Transfer steps must be re-tested end-to-end with the bank's file gateways (see [SWIFT / FileAct context in the banking guides](../banking/swiftnet_fileact_guide.md)). |
| Quantitative SLA (deadline + escalation) | Task/DAG-level `sla` (timedelta) + `sla_miss_callback`, plus alerting integrations (email, Slack, PagerDuty, ServiceNow) | Control-M SLAs are monitored continuously by EM with escalation rules; Airflow evaluates SLAs when a task completes and fires a callback on misses. The callback must implement the bank's escalation path. |
| Job | **Task** (operator or `@task` function) | One-to-one at first; later consolidation into multi-step tasks is possible because Airflow tasks are code, not DB rows. |
| Job flow / job chain (a folder's dependency network) | **DAG** | The condition graph becomes the DAG's edge set. This is the core of the translation. |
| Folder / job group | DAG folder (repository directory) + `tags` + **TaskGroups** | Control-M folders are permission/monitoring boundaries; in Airflow, directories and tags organize the DAG repo, and TaskGroups organize tasks *within* a DAG. |
| Order / manual rerun / force run | DAG run triggering: `trigger_dag_run` via UI/CLI/**REST API**, `clear` task instances, `backfill` | Control-M orders are scheduler-side instances; Airflow DAG runs are schedule-interval instances. Reruns are "clear and rerun" of task instances — cleaner, because state is explicit. |
| Control-M database (history, statistics) | Airflow **metadata database** (DAG runs, task instances, logs) + log storage + statsd metrics | Both keep audit-relevant history; Airflow's is open (PostgreSQL) and queryable, and logs are files/objects per task. Plan retention and archiving explicitly (Section 7). |
| Control-M/EM monitoring console | Airflow **UI** + **REST API** + metrics (statsd → Prometheus/Grafana) and alerting integrations | EM is a proprietary console; Airflow's UI is standard, and the REST API allows building custom dashboards. Ops teams need retraining either way. |
| Control-M/Agent | **Executor** workers: Celery worker nodes or Kubernetes pods (KubernetesExecutor); one pod per task | Control-M agents are long-lived daemons on fixed hosts; Airflow workers are ephemeral and horizontally scalable. Host-bound jobs (e.g., "must run on the z/OS LPAR") need explicit placement logic (Section 7, mainframe boundary). |
| Control-M resources / agent pools | Airflow **Pools** (concurrency limits per pool) + worker concurrency settings | Both limit parallelism; Airflow pools are per-task-instance slot counters, simpler and code-defined. |
| Control-M calendars for holidays | Holiday logic in code: a Python set of holiday dates, a custom timetable, or a `BranchPythonOperator` gate | This is where scheduling-semantics drift bites (Section 7): a bank's holiday calendar must be ported exactly, including SG public holidays and MAS-observed days. |

### Reading the Table — The Semantic Differences

**Calendars and run cycles.** Control-M's run cycle is a rich, calendar-aware concept: "run every weekday, skip bank holidays, also run on month-end regardless of weekday." Airflow's native scheduler is cron-based. For most daily batches a cron expression plus `catchup=False` is enough, but anything with holiday logic, business-day offsets, or fiscal month-end needs either an explicit holiday gate task or a custom timetable class. Budget for this; it is the most common source of "the batch ran on the wrong day" incidents after migration.

**Conditions and edges.** IN-COND/OUT-COND is a *global token system*: any job in any folder can wait on a condition set by any other job, even across servers, and conditions persist until deleted. Airflow's dependency model is *local and structural*: an edge between two tasks in the same DAG, or an `ExternalTaskSensor` for cross-DAG waits. The migration consequence: **global condition names become DAG-local edges**, and the rare cross-folder conditions become cross-DAG dependencies, which must be designed carefully (sensor timeouts, `external_task_id` matching, execution-date alignment). The repo's [Control-M External Conditions Guide](control_m_external_conditions_guide.md) is directly relevant when reconstructing these.

**Variables.** Control-M variables are substituted at order time from the scheduler; Airflow Variables are stored values read at task runtime. The practical difference is freshness and scope — and the security rule that credentials must go to a secrets backend, never to Variables (Section 7; see the [Cybersecurity Guide](cybersecurity_guide.md)).

**Preconditions and file-watch.** A Control-M estate is full of "wait for file" jobs that poll with shell loops. Airflow's native answer is sensors, and deferrable sensors make waiting nearly free. Migration guidance: replace every polling loop with a sensor, and set sensible `timeout` and `poke_interval` — a sensor that pokes every 10 seconds for 6 hours is a self-inflicted DDOS on the scheduler.

**MFT.** Control-M for MFT steps become transfer operators. Airflow's SSH/SFTP providers cover the common bank patterns (pulling from/ pushing to file gateways), and the repo's [Axway CFT – Control-M Integration](axway_cft_controlm_integration.md) documents the Axway side of the file-transfer boundary. Transfer steps carry their own risk profile: test them early, against the real gateways, with the real certificates.

**SLA and alerting.** Control-M's quantitative SLA is monitored continuously by EM with multi-level escalation. Airflow's `sla` mechanism is evaluated per task (and DAG) when instances complete, firing `sla_miss_callback`. The bank's escalation workflow (batch ops → on-call → incident management) must be implemented *in* the callback and in the alerting integration; this is a real gap to close, not a config toggle.

**Ordering and rerun.** Control-M's "order" is a scheduler action; Airflow's DAG run is a first-class, queryable object created by the scheduler per interval (or via API/UI for manual runs). Reruns are `clear` (delete task instance state) + rerun, which is safer than Control-M's rerun semantics because downstream state is recomputed by trigger rules rather than by condition bookkeeping.

**Monitoring.** EM's dashboards become the Airflow UI plus metrics. Airflow exposes statsd metrics (scheduler heartbeats, task states, durations) that feed Prometheus/Grafana; the UI covers graph, grid, gantt, and logs per task instance. Both are auditable, but the *habits* of the batch ops team must be retrained (Section 7, audit and change management).

### Worked Translation: A Condition Chain Becomes a DAG

Consider a classic Control-M chain in folder `PAYMENTS`:

| Job | Run cycle | IN-COND | OUT-COND | Action |
|---|---|---|---|---|
| PAYMENTS.EXTRACT | Daily 22:00 | — | `PAY_RAW_READY` | Extract raw payment file |
| PAYMENTS.VALIDATE | — | `PAY_RAW_READY` | `PAY_VALID` | Validate records |
| PAYMENTS.SWIFT_GEN | — | `PAY_VALID` | `PAY_SWIFT_READY` | Generate SWIFT MT103 file |
| PAYMENTS.SWIFT_SEND | — | `PAY_SWIFT_READY` | — | Send via gateway (SLA 06:00) |

The same flow in Airflow — three named conditions collapse into three edges, and the intermediate "wait" jobs disappear entirely:

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator
from airflow.providers.sftp.operators.sftp import SFTPOperator

with DAG(
    dag_id="payments_eod",
    schedule="0 22 * * *",          # daily 22:00, the run cycle
    start_date=datetime(2026, 1, 1),
    catchup=False,
    sla_miss_callback=notify_batch_ops,   # the quantitative SLA, 06:00
    tags=["eod", "payments"],
) as dag:
    extract = BashOperator(task_id="extract", bash_command="extract_payments.sh")
    validate = PythonOperator(task_id="validate", python_callable=validate_payments)
    swift_gen = BashOperator(task_id="swift_gen", bash_command="gen_swift_mt103.sh")
    swift_send = SFTPOperator(
        task_id="swift_send",
        ssh_conn_id="swift_gateway",            # connection from the secrets backend
        local_filepath="/data/out/{{ ds_nodash }}_MT103.txt",
        remote_filepath="/inbox/MT103_{{ ds_nodash }}.txt",
    )
    extract >> validate >> swift_gen >> swift_send
```

Three observations worth internalizing:

1. **Conditions became edges.** `PAY_RAW_READY`, `PAY_VALID`, and `PAY_SWIFT_READY` have no Airflow counterpart; the dependency information they carried is now the `>>` chain. Nothing is lost — but nothing is implicit either, so the reconstruction in Section 5c must be exhaustive.
2. **Scheduling moved to the DAG.** The "wait for condition" jobs vanished because Airflow's scheduler orders tasks by edges. Only the chain head (`extract`) carries the schedule; the rest inherit it.
3. **Naming and time references are native.** `{{ ds_nodash }}` replaces what Control-M would have done with variables and date tokens; the SLA becomes a DAG-level `sla` with the same escalation callback. The file date comes from the run's execution date, which is also what reconciliation (Section 5d) compares against Control-M's run history.

---

## 5. Migration Approach — Phased Methodology

The methodology is deliberately conservative: **inventory → map → reconstruct → coexist → cut over**, with Control-M as the source of truth until each wave is proven in Airflow. Total elapsed time for a mid-size bank estate (~200 jobs) is typically 6–9 months (see the Cymbal Bank timeline in Section 8).

### (a) Inventory and Discovery

You cannot migrate what you have not enumerated. The discovery phase produces a complete, machine-readable inventory of the Control-M estate. Primary extraction sources:

- **Control-M Configuration Manager export** — exports job definitions, folders, calendars, and run cycles as structured files; the backbone of the inventory.
- **Control-M/EM reporting** — run history, SLA compliance, and frequency reports; tells you what *actually* runs vs what is merely defined (dead definitions are common and should be flagged, not migrated).
- **Automation API (REST)** — programmatic enumeration of jobs, folders, conditions, and calendars; useful for building the inventory pipeline and for later reconciliation during coexistence.

Data collected per job:

| Field | Purpose |
|---|---|
| Job name + folder | Identity; drives naming in Airflow (Section 5b). |
| Run cycle + calendar(s) | Schedule translation (Section 4: calendars/run cycles). |
| IN-COND / OUT-COND lists | Dependency reconstruction (Section 5c). |
| Job type (command, JCL, script, MFT, SAP, etc.) | Operator selection and platform boundary. |
| MFT steps (source/destination, protocol) | Transfer operator mapping. |
| Quantitative SLA | `sla` + callback design. |
| Agent/host affinity | Worker placement constraints (mainframe boundary, Section 7). |
| Variables used | Variable/param/secrets classification. |
| Historical run data (last 90 days: start, end, duration, rc, failures) | Baseline for validation and throughput sizing. |

Deliverables: a job inventory table (or database), a condition map (who sets/deletes what), a calendar inventory, and a baseline of run durations and failure rates. The inventory is also the input to the risk register (Section 8d).

### (b) Job-to-DAG Mapping Patterns

Two canonical patterns, plus a hybrid:

1. **One DAG per batch flow (recommended).** Group jobs by business flow — payments, posting, reconciliation, reporting, regulatory — and build one DAG per flow, with tasks for the jobs and TaskGroups for sub-flows. Pros: dependencies are visible in one graph; scheduling and SLAs are flow-level; matches how batch operations already think ("the payments chain"). Cons: large DAGs (60+ tasks) need discipline to keep readable.
2. **One DAG per Control-M folder.** A literal 1:1 translation. Pros: fast to produce, easy traceability during coexistence. Cons: preserves Control-M's folder boundaries even where they are arbitrary, fragments cross-folder dependencies into `ExternalTaskSensor` chains, and can multiply DAG count and scheduler load.
3. **Hybrid.** One DAG per flow for the core chains; small utility/legacy folders stay as one-DAG-per-folder until decommissioned.

Naming conventions (adopt one, enforce it in CI):

| Control-M artifact | Airflow convention |
|---|---|
| Folder `PAYMENTS` | DAG id `payments_eod` (or `eod_payments`), file `dags/payments/eod_payments.py` |
| Job `PAYMENTS.EXTRACT` | Task id `extract` (prefix-free; keep ids short, they appear in logs and URLs) |
| OUT-COND `PAYMENTS_READY` | Edge `extract >> validate >> handoff`; external condition becomes `ExternalTaskSensor` in the consuming DAG |
| Calendar `SG_BANK_HOLIDAY` | Python module `calendars/sg_bank_holidays.py` with a frozenset of dates, imported by DAGs |

### (c) Dependency and Ordering Reconstruction

This is the highest-risk technical step: rebuilding the condition graph as DAG edges.

1. **Build the condition graph.** From the inventory, construct a directed graph: job A → job B if B has an IN-COND that A sets (or a date-based trigger).
2. **Classify edges.** Time-based triggers (run cycle at time T) become scheduling attributes; condition-based triggers become edges; mixed triggers (job waits for condition *and* earliest time) become edges with a `TimeSensor` or a time gate task.
3. **Detect orphans and dead conditions.** Conditions that are set but never consumed, or consumed but never set, reveal jobs that can never run (or run spuriously). In Control-M these are quiet pathologies; in Airflow they become visible as never-scheduled tasks or unsatisfied sensors. Decide per case: fix, drop, or flag for business confirmation.
4. **Handle fan-in and fan-out.** Many-to-one (fan-in): several jobs set the same condition; the downstream task uses trigger rule `all_done`/`none_failed` or an explicit join task. One-to-many (fan-out): one job's OUT-COND fans out to many consumers; in Airflow this is a single task with many downstream edges — natural.
5. **Reconstruct cross-DAG edges.** Where a condition crosses folder boundaries, choose: merge the flows into one DAG (preferred), or use `ExternalTaskSensor` with aligned `execution_date` semantics and generous `timeout`. Document every cross-DAG edge; they are the coexistence-reconciliation hotspots.
6. **Add explicit failure semantics.** Control-M conditions often only encode success; Airflow trigger rules let you express `one_failed` (run a compensation task), `all_done` (always proceed, even on failure), and `none_failed`. Use this to *improve* the batch: failure paths that Control-M expressed as separate alerting jobs become real edges to compensation or notification tasks.

### (d) Parallel-Run and Coexistence Strategy

During migration, both schedulers run the same batch. The rules:

- **Control-M remains the source of truth** (the one that executes production side effects) until a wave is formally cut over. Airflow instances of the same flows run in **shadow mode**: scheduled, executed against non-production targets or read-only copies, and *never* allowed to produce side effects (no real payments, no real regulatory submissions).
- **Reconciliation** is a daily automated job comparing, per flow and per job: started/ended on time, exit status, duration delta, and output artifact fingerprints (file names, sizes, hashes). Flow-level reconciliation (did the whole chain complete) matters more than job-level; job-level mismatches are the diagnosis, flow-level mismatch is the incident.
- **Baseline first.** Capture 90 days of Control-M run history before shadow mode starts, so reconciliation has a comparison set.
- **Guardrails:** shadow DAGs must fail closed (any ambiguity → do not execute the side-effectful step); alerts from shadow runs go to a dedicated channel, never the production pager; and Control-M orders for migrated flows are kept running until the wave's exit criteria pass.

Coexistence ends per wave, not globally: when a wave's reconciliation shows N consecutive clean days (typically 10–15), the wave cuts over and Control-M orders for those flows are disabled (see (e)).

**Reconciliation implementation sketch.** A daily reconciliation DAG (itself an Airflow DAG, `batch_reconciliation`) pulls three inputs — Control-M run history (via the Automation API), Airflow run history (via the Airflow REST API), and output-artifact listings (file names, sizes, hashes from the file gateways and data stores) — and produces a per-job and per-flow comparison:

| Comparison dimension | Control-M source | Airflow source | Match rule |
|---|---|---|---|
| Scheduled time | Job order time | DAG run schedule | Same business date; tolerance ±5 min |
| Start / end time | EM run report | Task instance start/end | Within ±15 min (informational) |
| Exit status | Job return code | Task instance state | rc 0 ↔ success; non-zero ↔ failure |
| Output artifacts | File registry / MFT log | File names from DAG logs + file system scan | Same names, sizes, SHA-256 hashes |
| SLA met | Quantitative SLA report | `sla` evaluation + completion time | Both met or both missed |

Flow-level rollup: a flow is **clean** when every job matches and the chain completed; **drifted** when any job mismatches; **blocked** when a job never started. Alerts: drifted/blocked goes to the shadow channel (never the production pager); three consecutive drifted days on the same job escalates to the migration lead — that job's mapping is wrong and must be fixed before the wave can cut over. The reconciliation DAG also produces the sign-off pack that the go/no-go review (Section 5e) consumes, which keeps the evidence trail consistent across waves.

### (e) Cutover

Two strategies, one recommended:

- **Phased cutover by business area or batch-window segment (recommended).** Wave 1 migrates low-risk flows (reporting), wave 2 reconciliation, wave 3 payments/posting, wave 4 regulatory — see Section 8c. Each wave has its own go/no-go criteria, rollback plan, and freeze window.
- **Big-bang cutover.** One weekend, everything. Only viable for small estates; for a bank's EOD batch it multiplies rollback risk and is not recommended.

Per-wave cutover procedure:

1. **Freeze window**: no batch changes (either side) for the wave's validation period, typically 5–10 business days.
2. **Go/no-go review**: reconciliation clean, SLA metrics met, runtimes within budget, ops team signed off on runbooks.
3. **Switch**: disable Control-M orders for the wave's flows; enable production mode in Airflow (side effects allowed).
4. **Rollback plan (flip-back)**: if the wave fails in production, re-enable Control-M orders for those flows within the rollback window (typically 1–2 business days), then rerun the affected day's batch from Control-M. Because Control-M definitions were never deleted, flip-back is a re-order, not a rebuild — which is exactly why Control-M stays configured until the whole migration closes.

### (f) DAG Testing

- **Unit tests (pytest):** test DAG integrity (no cycles, unique task ids, valid schedules), test helper functions (holiday calendars, file-name logic), and use `dagbag`/`dag.test()` to run a DAG in-process. Aim for CI-enforced tests on every DAG (Section 5g).
- **Backfill for historical validation:** run migrated DAGs with `backfill` over the 90-day baseline window and compare outcomes against Control-M run history. Discrepancies (a task that never runs, a sensor that times out) are caught here, before shadow mode.
- **Catchup validation:** verify `catchup` behavior matches intent — for daily DAGs with `catchup=False` deployed mid-day, no phantom historical runs may appear.
- **Staging instance:** all of the above happens in a staging Airflow environment that mirrors production (same Airflow version, same provider packages, same executor sizing relative to load). Production DAGs only after staging sign-off.
- **Comparison against Control-M run history** is the acceptance test: same inputs → same outputs, within tolerance (durations may differ; results may not).

### (g) CI/CD for DAGs

DAGs are code and must travel the same path as any banking application change. We do not re-derive platform practices here — the repo's [AI Platform Engineering Guide](ai_platform_engineering_guide.md) is the authority for CI/CD, environments, and deployment. Apply its practices to DAGs: linting (ruff on DAG code), unit tests in CI, DAG parsing checks (`airflow dags list-import-errors` in a test container), and deployment via Git-sync (MWAA/Composer native) or image-based deploys (Astronomer style). Change management for DAGs then inherits the bank's existing PR/approval/audit pipeline — which is a major compliance win over database-defined schedules.

### (h) Observability

- **Metrics:** Airflow emits statsd metrics (scheduler loop timing, task instances by state, DAG parsing times) — forward them to the bank's Prometheus/Grafana stack. Alert on scheduler heartbeats, DAG import errors, and task failures, not just on SLA misses.
- **Logs:** task logs per task instance, shipped to the central logging platform; retention aligned with audit requirements (Section 7).
- **Alerting:** SLA-miss callbacks, task-failure alerting via the alerting integration, and the shadow-mode reconciliation alerts during coexistence.
- The [AI Platform Engineering Guide](ai_platform_engineering_guide.md) covers the platform-level observability practices; this guide adds only the Airflow-specific instrumentation points above.

---

## 6. Managed Airflow Options

For Cymbal Bank, the operational choice is between running Airflow ourselves (self-hosted, typically on the bank's Kubernetes platform) and buying a managed service. The three mainstream managed options are Amazon MWAA, Google Cloud Composer, and Astronomer; there is also the option of a managed Airflow from other vendors or a SaaS WLA migration target, but the market has consolidated on these three plus self-hosting.

### Amazon Managed Workflows for Apache Airflow (MWAA)

- **Launched:** November 2020 — announced by AWS on November 24, 2020 ("Introducing Amazon Managed Workflows for Apache Airflow (MWAA)").
- **What it is:** a fully managed Airflow service on AWS. AWS runs the scheduler, workers, and webserver; you bring DAGs (stored in an S3 bucket, synchronized to the environment), plugins, and requirements files.
- **Positioning:** the natural choice for banks whose batch and data estate is AWS-centric. It supports Celery-based workers and, since later releases, the Kubernetes executor option; it integrates with AWS services (S3, Secrets Manager, EMR, Redshift, Step Functions).
- **Banking considerations:** VPC-only deployment, IAM-based access, Secrets Manager integration for credentials, and CloudWatch metrics/logs — all align well with an AWS landing zone.

### Google Cloud Composer (Managed Service for Apache Airflow)

- **Launched:** 2018 — Google announced Cloud Composer in May 2018 (TechCrunch, May 1, 2018); it is now branded "Managed Service for Apache Airflow (formerly Cloud Composer)."
- **What it is:** Google's fully managed Airflow, tightly integrated with GCP (BigQuery, Dataflow, Dataproc, Cloud Storage, Pub/Sub), supporting hybrid and multi-cloud orchestration. Google is an active contributor to Apache Airflow.
- **Banking considerations:** strong if the bank's data platform is GCP-based; supports private IP environments, Shared VPC, VPC Service Controls, and CMEK encryption — the controls a Singapore bank's cloud governance would require.

### Astronomer (Astro)

- **Founded:** 2015 (Cincinnati, Ohio; later New York City) — the company most identified with commercial Airflow.
- **What it is:** Astro, Astronomer's platform, offers managed Airflow (Astro on public cloud or private/hybrid) plus the Astro CLI, Astro Runtime (a maintained, tested Airflow image), and deployment-as-code. Astronomer was the creator of Astronomer Certified, the older tested distribution of Apache Airflow, which has been superseded by Astro Runtime in current guidance.
- **Banking considerations:** private/hybrid deployment options suit banks that cannot put the scheduler in a public cloud multi-tenant plane; the image-based deploy model fits GitOps (Section 5g).

### Comparison Table

| Dimension | Amazon MWAA | Google Cloud Composer | Astronomer (Astro) | Self-hosted (Kubernetes) |
|---|---|---|---|---|
| Launched | November 2020 | 2018 | Company founded 2015; Astro GA 2020s | n/a (open source since 2015) |
| Run model | Managed service on AWS | Managed service on GCP | Managed (public/private/hybrid) | You operate scheduler, workers, DB, webserver |
| Airflow version support | Vendor-managed versions (2.x line; 3.x as offered) | Vendor-managed versions (2.x; Airflow 3 in preview as of 2026) | Astro Runtime images (2 years maintenance); Airflow 3 support | Any version you choose to run |
| DAG deployment | Git-sync from S3 (or CLI) | Cloud Composer environments sync from Cloud Storage/Git | Image-based deploys (astro deploy) or Git | Git-sync / image-based per your CI/CD |
| Integrations | AWS-native (S3, Secrets Manager, EMR, Redshift) | GCP-native (BigQuery, Dataflow, Dataproc, GCS, Pub/Sub) | Cloud-agnostic; strong multi-cloud | Whatever you install (all providers) |
| Cost model | Pay per environment size (workers, scheduler) | Consumption-based (vCPU/hour, GB/month, GB transferred) | Subscription per deployment | Infrastructure + engineering time |
| Ops burden | Low (AWS runs the plane) | Low (Google runs the plane) | Low–medium (platform managed; you own DAGs) | High (you own everything) |
| Banking fit | Best for AWS-centric banks | Best for GCP-centric banks | Best for hybrid/private-cloud mandates | Best when full control and custom hardening are required |

### Managed vs Self-Hosted — Guidance

For a bank, the decision hinges on four questions:

1. **Where does the data plane live?** If the bank's batch and data estate is already on AWS or GCP, the matching managed service wins on integration and operational simplicity.
2. **Can the scheduler be in a shared cloud plane?** Banks with strict data-residency or control mandates (Section 7) may prefer Astronomer's private/hybrid model or self-hosting on the bank's own Kubernetes platform.
3. **Who runs the platform?** Managed services trade a subscription for most of the Airflow operations burden (upgrades, scaling, HA). Self-hosting keeps cost variable but demands Airflow-fluent platform engineers — a real constraint given how few banks staff for it.
4. **Version strategy.** Managed services dictate upgrade cadence; self-hosting lets the bank stay on a certified version for longer (useful when a DAG fleet depends on 2.x behavior).

The claims audit (Section 9) marks the launch dates and their sources; treat vendor "GA" dates beyond those listed as unverified unless confirmed.

---

## 7. Risks and Banking-Specific Concerns

### Scheduling-Semantics Drift

The biggest class of post-migration incidents is not "Airflow broke" but "Airflow ran on a different day/time than Control-M would have."

- **Run cycles vs cron:** Control-M run cycles encode business calendars ("first business day of month", "last weekday before month-end"). A naive cron translation runs on the wrong day for month-end and holiday weeks. Mitigation: holiday gate tasks, custom timetables, and a per-DAG schedule test matrix covering the bank's full year (including SG public holidays and MAS-observed days).
- **Time zones and DST:** Control-M and Airflow both need an explicit time zone. Set Airflow's `default_timezone` to `Asia/Singapore` (the bank has no DST, but upstream/downstream systems may); never leave schedules in UTC and assume the ops team will translate.
- **Calendar holidays:** port the bank's holiday calendar exactly — a missing holiday makes the batch run on a public holiday; an extra one makes it miss a working day. Both are regulatory-adjacent incidents.

### Retry and Alerting Differences

- **Control-M retries** are per-job definitions with escalation rules driven by EM. **Airflow retries** are per-task (`retries`, `retry_delay`, `max_retry_delay`), executed by the scheduler — with the important caveat that a retried task re-executes the operator, so tasks must be **idempotent** (re-runnable without double side effects). This is a behavioral change for jobs that were never idempotent under Control-M.
- **Alerting:** Control-M's EM console and SLA escalation is a mature ops surface. Airflow's equivalents are `sla_miss_callback`, failure alerting via integrations, and metrics-based alerting — all of which must be wired and tested *before* cutover, with the bank's incident-management path. Plan a "quiet period" rule: during shadow mode, shadow alerts go to a non-production channel.

### EOD Batch Window Pressure

- **Deadlines and SLAs:** the EOD batch must finish by a hard time (e.g., 07:00 SGT for start-of-day position). Airflow introduces no inherent slowdown, but scheduler and executor sizing determine throughput. Size using the baseline: peak concurrent task count from the 90-day Control-M history, times headroom.
- **Executor sizing:** with KubernetesExecutor, one pod per task — a burst of 40 parallel tasks needs 40 pods; with CeleryExecutor, workers must cover the peak. Right-size from the baseline, and use pools to cap runaway parallelism.
- **Scheduler throughput:** Airflow's scheduler parses DAGs on an interval; thousands of DAGs slow parsing. Keep DAG count and file size disciplined (Section 5b), and monitor scheduler loop timing via statsd.
- **Deadline safety net:** add a DAG-level "deadline guard" task pattern — a `PythonOperator` scheduled late in the window that checks batch completion and pages batch ops if the chain is behind, mirroring Control-M's quantitative SLA.

### Audit Trail and Change Management

For a MAS-regulated bank, the batch is a control, and its operation is auditable evidence. The compliance angle is fully covered in the repo's [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) and the [Cybersecurity Guide](cybersecurity_guide.md); the migration-specific points are:

- **Who changed what, when:** DAGs in Git with PR review give a *better* change trail than database-defined schedules. Enforce branch protection, signed commits, and audit-friendly PR descriptions.
- **Run evidence:** Airflow's metadata DB plus task logs are the audit record. Define retention (typically 7 years for MAS-relevant records, per the regulations guide) and archive task logs to object storage with immutability.
- **Access control:** RBAC in the Airflow UI (roles per team), scoped service accounts for API access, and no shared admin credentials. See the [Cybersecurity Guide](cybersecurity_guide.md) for the bank's identity and access standards.
- **Change windows:** batch changes ride the bank's change-management process; freeze windows during cutover waves are part of the plan (Section 5e).

### Mainframe Integration Boundary

Control-M's z/OS agent is one of its crown jewels: JCL jobs on the mainframe are scheduled, monitored, and SLA'd from the same plane as distributed jobs. Airflow has no native z/OS executor, so the boundary must be designed explicitly:

- **What stays on the mainframe:** the JCL jobs themselves. Cymbal Bank's core banking runs on the mainframe; those jobs are not being rewritten.
- **How Airflow reaches the mainframe:** SSH-based job submission (SSHFineOp-style automation, or z/OSMF workflows), or a bridge where the mainframe's own scheduler (or a z/OS agent shim) exposes job completion. Common pattern: an Airflow `SSHOperator` submits the JCL via a z/OSMF REST call or SSH to a submit service, then a sensor polls job completion (e.g., via z/OSMF job status or a sentinel file).
- **Design rule:** treat the mainframe as a *remote execution platform with a thick wall*. One integration point (a dedicated submit-and-poll service), timeouts and retries on that boundary, and no Airflow worker ever running on the mainframe.
- This boundary is where Control-M's z/OS heritage is hardest to replace; budget the most engineering time here and validate with the mainframe team early.

### Data Residency

- Airflow itself holds metadata and logs, not the bank's data — but DAGs and variables may embed data references, and logs may capture file names, paths, and (if careless) payload snippets.
- Keep Airflow environments within the bank's approved regions (Singapore primary; DR region per the bank's recovery plan). Managed services: choose regions explicitly (MWAA/Composer are regional services). Logs and metrics must stay in-region or in the bank's approved log estate.
- The [Cybersecurity Guide](cybersecurity_guide.md) covers the bank's data-classification and residency standards; apply them to DAG repositories and Airflow log storage.

### Secrets Management

- **Never** store credentials in Airflow Variables or DAG code. Use Airflow's secrets backend (AWS Secrets Manager, GCP Secret Manager, or HashiCorp Vault) with the `connections` and `variables` secret backends configured.
- Connections (databases, SFTP, APIs) live in the secrets backend; DAGs reference them by connection id.
- Rotation: secrets backends support rotation without DAG redeploys; bake rotation into the runbook.
- See the [Cybersecurity Guide](cybersecurity_guide.md) for the bank's secrets-management standards and tooling.

---

## 8. Cymbal Bank Worked Example — EOD Batch Migration

Cymbal Bank is a Singapore-headquartered bank running its end-of-day batch on BMC Control-M across a distributed estate (Linux application servers, an Oracle data warehouse) and a mainframe core (z/OS). The EOD batch must complete by 07:00 SGT; regulatory submissions (MAS Notice 610 / TRM-related reporting and the associated MAS returns) have hard deadlines and cannot slip. The migration follows the methodology of Section 5, with a target of 6–9 months.

### (a) Inventory Summary

Discovery (Section 5a) found **201 active jobs** in **18 Control-M folders** (a further 34 defined-but-never-run jobs were flagged and excluded from migration).

| Job family | Jobs | Control-M constructs in use | Notes |
|---|---|---|---|
| Payments (SWIFT, MEPS+/FAST files, nostro instructions) | 55 | Run cycles, heavy IN/OUT-COND chains, Control-M for MFT steps, agent pools, SLA 06:45 | Highest fan-in/fan-out; MFT to/from gateways |
| Posting (ledger posting, interest, fees, statement lines) | 45 | Calendars (month-end runs), conditions, quantitative SLA | Month-end calendar logic is critical |
| Reconciliation (GL vs clearing vs nostro, suspense) | 30 | Conditions, SLA, alerting jobs | Failure paths matter; compensation logic |
| Reporting (MI packs, customer statements, board packs) | 40 | Run cycles, calendars, file generation + MFT distribution | Lowest risk; ideal first wave |
| Regulatory (MAS returns, TRM-related controls, audit extracts) | 20 | Fixed calendars, strict SLA, minimal conditions | Hard deadline; last wave |
| Utilities and housekeeping (log purges, temp cleanup) | 11 | Simple run cycles | Migrate opportunistically |
| **Total** | **201** | | |

### (b) Mapping Table Excerpt

A sample of the job-to-DAG mapping produced in Section 5b (naming per the conventions table):

| Control-M folder | Control-M job | Run cycle / conditions | Airflow DAG | Task id | Mapping notes |
|---|---|---|---|---|---|
| PAYMENTS | PAYMENTS.EXTRACT | Daily 22:00; OUT-COND `PAY_RAW_READY` | `payments_eod` | `extract` | BashOperator on app server |
| PAYMENTS | PAYMENTS.VALIDATE | IN-COND `PAY_RAW_READY`; OUT-COND `PAY_VALID` | `payments_eod` | `validate` | PythonOperator; idempotent |
| PAYMENTS | PAYMENTS.SWIFT_GEN | IN-COND `PAY_VALID`; MFT step | `payments_eod` | `swift_gen` | SSHOperator + file generation |
| PAYMENTS | PAYMENTS.SWIFT_SEND | IN-COND `PAY_SWIFT_READY`; SLA 06:00 | `payments_eod` | `swift_send` | SFTPOperator to gateway (see [Axway CFT – Control-M Integration](axway_cft_controlm_integration.md)) |
| POSTING | POSTING.POST_GL | Month-end calendar; IN-COND `RECON_CLEAN` | `posting_eod` | `post_gl` | Holiday gate task upstream |
| POSTING | POSTING.INTEREST | Month-end run cycle | `posting_eod` | `calc_interest` | Custom timetable for month-end |
| POSTING | POSTING.STATEMENTS | Daily 03:00 | `posting_eod` | `gen_statements` | PythonOperator |
| RECON | RECON.NOSTRO | IN-COND `PAY_VALID` + `POST_GL_DONE`; OUT-COND `RECON_CLEAN` | `recon_eod` | `recon_nostro` | Cross-DAG input via `ExternalTaskSensor` on `payments_eod`/`posting_eod` |
| RECON | RECON.SUSPENSE_ALERT | Triggered on failure (one_failed pattern) | `recon_eod` | `suspense_alert` | trigger_rule=`one_failed`; notifies ops |
| REPORTING | REP.MI_PACK | Daily 04:30 | `reporting_eod` | `mi_pack` | PythonOperator |
| REPORTING | REP.STATEMENT_DIST | MFT distribution | `reporting_eod` | `distribute` | SFTPOperator |
| REGULATORY | REG.MAS_RETURNS | Fixed calendar (business day +1); SLA 06:30 | `regulatory_eod` | `mas_returns` | Runs last; hard deadline |
| REGULATORY | REG.TRM_CONTROLS | Monthly; IN-COND `MAS_RETURNS_OK` | `regulatory_eod` | `trm_controls` | Monthly timetable |
| UTIL | UTIL.TEMP_PURGE | Daily 05:30 | `housekeeping` | `temp_purge` | BashOperator |

Cross-DAG edges: `recon_eod` waits on `payments_eod` and `posting_eod` (ExternalTaskSensor, aligned execution dates); `regulatory_eod` waits on `recon_eod` and `reporting_eod`. Four explicit cross-DAG edges in total — the reconciliation hotspots during shadow mode.

### (c) Phased Cutover Plan

| Wave | Flows | Shadow mode start | Cutover target | Exit criteria |
|---|---|---|---|---|
| Wave 1 | Reporting + utilities (51 jobs) | Month 2 | Month 3 | 10 consecutive clean reconciliation days; runtime within budget |
| Wave 2 | Reconciliation (30 jobs) | Month 3 | Month 4–5 | Cross-DAG edges stable; suspense alerts exercised |
| Wave 3 | Payments + posting (100 jobs) | Month 4 | Month 6 | MFT steps green against real gateways; 07:00 SLA met |
| Wave 4 | Regulatory (20 jobs) | Month 6 | Month 7–8 | MAS returns submitted on time for 15 consecutive business days; audit sign-off |

Each wave follows the cutover procedure of Section 5e: freeze window (5–10 business days), go/no-go review, switch, and a 1–2 business-day flip-back window to Control-M. Control-M definitions remain in place (but un-ordered for cut-over flows) until Wave 4 closes, after which the decommissioning phase begins.

### (d) Risk Register (Excerpt)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Schedule drift: month-end DAG runs on wrong day | Medium | High | Custom timetables + holiday gate tasks; per-DAG schedule test matrix over the full year |
| Cross-DAG sensor misalignment (execution date mismatch) | Medium | High | Aligned execution dates, documented per edge, covered by unit tests; reconciliation catches in shadow mode |
| MFT step breaks against real gateway (cert/protocol) | Medium | High | Early end-to-end transfer tests in staging with real certificates (Wave 1 includes a pilot transfer) |
| Non-idempotent task re-executes side effects on retry | Medium | High | Idempotency review per task during mapping; retry policy set conservatively; compensation tasks where needed |
| Batch window overrun (throughput) | Low–Medium | High | Sizing from 90-day baseline; executor autoscaling; deadline guard task pages ops |
| Ops team unfamiliarity with Airflow UI/runbooks | High | Medium | Training in Month 1–2; runbooks per wave; shadow-mode alert channel for practice |
| Regulatory submission late during Wave 4 | Low | Critical | Hard deadline guard; flip-back window; MAS-required evidence preserved (metadata DB + logs archived) |
| Secrets leaked into DAG code or Variables | Low | Critical | Secrets backend enforced; CI secret scan; code review (see [Cybersecurity Guide](cybersecurity_guide.md)) |

### (e) Timeline

| Phase | Duration | Milestone |
|---|---|---|
| 0. Preparation (training, staging environment, baseline capture) | Weeks 1–4 | Staging Airflow up; 90-day baseline archived |
| 1. Inventory & discovery (Section 5a) | Weeks 3–6 | 201-job inventory + condition map signed off |
| 2. Mapping & DAG development (Sections 5b–5c) | Weeks 6–14 | All DAGs in Git; unit tests in CI |
| 3. Wave 1 shadow + cutover | Weeks 12–16 | Reporting + utilities live in Airflow |
| 4. Wave 2 shadow + cutover | Weeks 16–22 | Reconciliation live |
| 5. Wave 3 shadow + cutover | Weeks 20–28 | Payments + posting live; 07:00 SLA met |
| 6. Wave 4 shadow + cutover | Weeks 28–36 | Regulatory live; audit sign-off |
| 7. Decommissioning (Control-M wind-down) | Weeks 36–40 | Control-M orders retired; final report |

Total: **~40 weeks (≈9 months)** including buffer; with a leaner estate and earlier staging readiness, 6–7 months is achievable.

---

## 9. Claims Audit

Every factual claim of consequence in this guide is audited below. Status legend: ✅ = verified at a primary or directly fetched source; ⚠ = plausible and widely documented but not directly verified at a primary source during this research pass; ❌ = contradicted or failed verification.

| Claim | Status | Source |
|---|---|---|
| Control-M was originally designed for IBM mainframe computers | ✅ | BMC product report: [bmc.com TrustRadiusReport_BMCControlM_ProductReport_v6.pdf](https://www.bmc.com/content/dam/bmc/migration/pdf/TrustRadiusReport_BMCControlM_ProductReport_v6.pdf) |
| BMC acquired Control-M from New Dimension Software in 1999 | ✅ | BMC product report (same PDF): "acquired from New Dimension Software in 1999"; also [Semantic Scholar topic: BMC Control-M](https://www.semanticscholar.org/topic/BMC-Control-M/8832754) |
| New Dimension Software was an Israeli software company | ✅ | [Semantic Scholar topic: BMC Control-M](https://www.semanticscholar.org/topic/BMC-Control-M/8832754) ("Israel's New Dimension Software") |
| Control-M runs on z/OS and distributed platforms (Unix, Windows, Linux) | ✅ | BMC product report (same PDF) |
| Control-M/Server, Control-M/Agent, Control-M/EM, Automation API component names and roles | ⚠ | Standard BMC product vocabulary; docs.bmc.com could not be scraped from this environment. Consistent with the repo's [Control-M Guide](control_m_guide.md) and [Control-M Migration Options Guide](control_m_migration_options_guide.md) |
| Automation API is a REST API introduced in Control-M 9.0.x | ⚠ | Per BMC documentation knowledge and the repo's Control-M guides; not directly fetched this session |
| Control-M containerized deployment (Docker) announced 2019 | ✅ | [Wikipedia: BMC Software](https://en.wikipedia.org/wiki/BMC_Software) citing Cloud Native Now (Nov 1, 2019) |
| Control-M SaaS edition (BMC Helix Control-M) launched 2020 | ✅ | [Wikipedia: BMC Software](https://en.wikipedia.org/wiki/BMC_Software) citing SiliconANGLE (Dec 2, 2020) |
| Airflow started at Airbnb in October 2014 | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) |
| Airflow open-sourced in 2015 (first release June 3, 2015) | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) (release 1.0.0, 2015-06-03; Beauchemin's "Airflow: a workflow management platform", June 2, 2015) |
| Airflow entered the Apache Incubator in March 2016 | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) + [airflow.apache.org/announcements](https://airflow.apache.org/announcements/) (March 19, 2016: "we applied for Airflow's entry to the Apache Incubator") |
| Airflow became a top-level Apache project in January 2019 | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) |
| Airflow 2.0 released December 17, 2020 | ✅ | [airflow.apache.org/announcements](https://airflow.apache.org/announcements/) (entry dated December 17, 2020) |
| Airflow 3.0 released April 22, 2025 | ✅ | [airflow.apache.org/announcements](https://airflow.apache.org/announcements/) (entry dated April 22, 2025) |
| Current stable line 3.3.x; 3.3.1 released August 12, 2026; 2.11.2 released March 14, 2026 | ✅ | [airflow.apache.org/announcements](https://airflow.apache.org/announcements/) |
| Airflow is written in Python under Apache License 2.0; original author Maxime Beauchemin | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) |
| Airflow is the de facto data-engineering orchestration tool (per VentureBeat, 2025) | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) citing VentureBeat (April 22, 2025) |
| Airflow 2.0 introduced a stable REST API | ⚠ | Stated in Airflow 2.0 release communications (announcement post linked from the 2.0.0 announcements entry); not directly fetched this session |
| Airflow 2.0 rebuilt the scheduler (DAG processor, faster scheduling loop) | ⚠ | Same as above; widely documented in Airflow 2.0 release notes |
| Deferrable operators and the triggerer were introduced in Airflow 2.2 | ⚠ | Per Airflow 2.2 release notes/blog; not directly fetched this session |
| Amazon MWAA launched November 2020 (announced November 24, 2020) | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) citing AWS blog "Introducing Amazon Managed Workflows for Apache Airflow (MWAA)", 2020-11-24: [aws.amazon.com/blogs/aws/introducing-amazon-managed-workflows-for-apache-airflow-mwaa/](https://aws.amazon.com/blogs/aws/introducing-amazon-managed-workflows-for-apache-airflow-mwaa/) |
| Google Cloud Composer launched in 2018 (announced May 2018) | ✅ | TechCrunch (May 1, 2018): "Google launches Cloud Composer, a new workflow automation tool for developers", cited by [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) |
| Cloud Composer is now branded "Managed Service for Apache Airflow" | ✅ | [cloud.google.com/composer](https://cloud.google.com/composer) (fetched directly) |
| Astronomer was founded in 2015 | ✅ | Multiple independent company profiles: [PrivCo](https://www.privco.com/company/astronomer), [ZoomInfo](https://www.zoominfo.com/c/astronomer-inc/371535935), yespress.io profile; no Wikipedia article exists |
| Astro Runtime is Astronomer's maintained Airflow image (2 years of maintenance) | ✅ | [astronomer.io](https://www.astronomer.io/) (fetched directly) |
| Astronomer Certified was the older distribution, superseded by Astro Runtime | ⚠ | Consistent with Astronomer's current documentation and guidance; not directly fetched this session |
| Airflow supports CeleryExecutor and KubernetesExecutor | ✅ | Core Airflow documentation ([airflow.apache.org/docs](https://airflow.apache.org/docs/apache-airflow/stable/executor/index.html)); standard, stable executor set |

---

## 10. What Could Not Be Verified

The following items could not be confirmed at a primary source during this research pass, and are therefore flagged rather than asserted as verified facts:

- **docs.bmc.com product documentation pages** could not be retrieved from this environment (the site blocked automated scraping). Consequently, Control-M-specific details — the Automation API version introduction (9.0.x), Control-M for Batch vs Control-M for MFT product naming, EM console feature specifics, and the z/OS agent's exact capability set — rest on the BMC product report PDF, the repo's existing Control-M guides, and general BMC product knowledge rather than a directly fetched BMC documentation page. The BMC documentation portal is listed in Section 12 as the authoritative reference to consult for these details.
- **Exact general-availability date of Google Cloud Composer.** The May 2018 announcement (TechCrunch) is verified; the specific GA date later in 2018 is not confirmed here.
- **Astronomer's exact founding date** (month/day). The year 2015 is verified across multiple independent profiles; the specific day is not.
- **Astronomer Certified deprecation timeline.** That Astro Runtime is the current maintained image is verified; the formal deprecation of Astronomer Certified and its dates are not.
- **Airflow 2.0 scheduler internals and stable REST API specifics.** The release date is verified; the detailed feature claims (DAG processor, stable REST API) come from release communications that were not fetched in full this session.
- **Wikipedia coverage of the Control-M product.** Wikipedia's "Control-M" title redirects to the carriage-return control character article, and there are no Wikipedia articles for New Dimension Software or Astronomer — so Wikipedia could not serve as a source for those items.
- **Cymbal Bank's estate numbers** (job counts, SLAs, timelines) are a worked example for this guide, not verified external facts.

---

## 11. Glossary

### Control-M Terms

| Term | Definition |
|---|---|
| Control-M | BMC's workload automation platform for scheduling, monitoring, and managing batch jobs and file transfers across mainframe, distributed, and cloud platforms. |
| Control-M/Server | The scheduling engine that holds definitions and decides what runs when. |
| Control-M/Agent | Daemon installed on execution hosts (z/OS or distributed) that runs jobs and reports status. |
| Control-M/EM | Enterprise Manager — the administration and monitoring console. |
| Job | A unit of work: command, script, JCL member, or transfer. |
| Folder / Job Group | Logical grouping of jobs for management and permissions. |
| Calendar | Date rules (working days, holidays, fiscal calendars) governing when jobs may run. |
| Run Cycle | The scheduling pattern attached to a job (daily, weekday, month-end, etc.), referencing calendars. |
| IN-COND / OUT-COND | Dependency tokens: a job waits for IN-CONDitions and sets/deletes OUT-CONDitions. |
| Order | An instance of a job scheduled to run on a given day. |
| Quantitative SLA | A deadline on a job or group with escalation and alerting. |
| Resource / Agent Pool | Shared limited-capacity assets / sets of agents for load distribution. |
| Automation API | Control-M's REST API for programmatic definition, ordering, and status queries. |
| Control-M for MFT | Managed File Transfer capability integrated with batch flows. |

### Airflow Terms

| Term | Definition |
|---|---|
| DAG | Directed acyclic graph — a workflow declared in Python with tasks and dependency edges. |
| Task / Task Instance | A unit of work in a DAG / one scheduled execution of that task. |
| Operator | Class defining what a task does (Bash, Python, SSH, SFTP, etc.). |
| Sensor | Operator that waits for a condition (file, time, external task). |
| XCom | Small message store for passing data between tasks. |
| DAG Run | An instance of a DAG for a given schedule interval or manual trigger. |
| Scheduler | Process that parses DAGs, creates runs, and schedules task instances. |
| Executor | Component that runs tasks (Sequential, Local, Celery, Kubernetes). |
| Metadata Database | Relational store of runs, task instances, variables, and connections. |
| Trigger Rule | Rule deciding when a task may run based on upstream states (all_success, all_done, one_failed, none_failed). |
| Catchup / Backfill | Mechanisms for running a DAG over past schedule intervals. |
| Timetable | Airflow 2.4+ mechanism for custom scheduling logic beyond cron. |
| TaskGroup | Logical grouping of tasks within a DAG. |
| Deferrable Operator / Triggerer | Async task pattern that frees worker slots while waiting. |
| Pool | Concurrency limit on a named set of task instances. |
| SLA | `sla` timedelta + `sla_miss_callback` for deadline monitoring. |
| REST API | Stable HTTP API for triggering and querying DAGs and runs. |

---

## 12. References and Appendix

### Primary Sources

**BMC / Control-M**

- BMC Control-M product report (history and platform facts): [TrustRadiusReport_BMCControlM_ProductReport_v6.pdf](https://www.bmc.com/content/dam/bmc/migration/pdf/TrustRadiusReport_BMCControlM_ProductReport_v6.pdf)
- BMC Control-M documentation portal (authoritative product docs): [docs.bmc.com/docs/controlm/](https://docs.bmc.com/docs/controlm/)
- [Wikipedia: BMC Software](https://en.wikipedia.org/wiki/BMC_Software)

**Apache Airflow**

- Project site: [airflow.apache.org](https://airflow.apache.org/)
- Release and project announcements (2.0.0, 3.0.0, current versions): [airflow.apache.org/announcements](https://airflow.apache.org/announcements/)
- Airflow 2.0 announcement: [airflow.apache.org/blog/airflow-two-point-oh-is-here/](https://airflow.apache.org/blog/airflow-two-point-oh-is-here/)
- Core documentation (DAGs, operators, sensors, executors, timetables, REST API, SLA): [airflow.apache.org/docs](https://airflow.apache.org/docs/apache-airflow/stable/)
- [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow)

**Managed services**

- Amazon MWAA: [aws.amazon.com/mwaa](https://aws.amazon.com/mwaa/); launch announcement: [Introducing Amazon Managed Workflows for Apache Airflow (MWAA)](https://aws.amazon.com/blogs/aws/introducing-amazon-managed-workflows-for-apache-airflow-mwaa/) (Nov 24, 2020)
- Google Cloud Managed Service for Apache Airflow (formerly Cloud Composer): [cloud.google.com/composer](https://cloud.google.com/composer); launch coverage: [TechCrunch, May 1, 2018](https://techcrunch.com/2018/05/01/google-launches-cloud-composer-a-new-workflow-automation-tool-for-developers/)
- Astronomer: [astronomer.io](https://www.astronomer.io/); documentation: [docs.astronomer.io](https://docs.astronomer.io/)

### Related Guides in This Repository

- [Control-M Guide](control_m_guide.md) — full Control-M platform background and terminology.
- [Control-M Migration Options Guide](control_m_migration_options_guide.md) — evaluation framework for Control-M replacement options (this guide is the Airflow-specific playbook).
- [Control-M External Conditions Guide](control_m_external_conditions_guide.md) — IN-COND/OUT-COND and cross-system condition handling.
- [Axway CFT – Control-M Integration](axway_cft_controlm_integration.md) — MFT/file-transfer boundary patterns for the migration.
- [AI Platform Engineering Guide](ai_platform_engineering_guide.md) — CI/CD, environments, and observability practices for DAG development and deployment.
- [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) — compliance, audit, and retention requirements for regulated batch processing.
- [Cybersecurity Guide](cybersecurity_guide.md) — secrets management, access control, and data-residency standards.
- [SWIFT FileAct Guide](../banking/swiftnet_fileact_guide.md) — file-gateway context relevant to MFT steps.

### Closing

Migrating the EOD batch from Control-M to Airflow is not a tool swap; it is the translation of two decades of scheduling intent — calendars, conditions, SLAs, and hard-won failure handling — into code that a new generation of engineers can read, test, and evolve. Done wave by wave, with Control-M as the source of truth until each flow is proven, the bank keeps its batch window, its audit trail, and its regulatory obligations intact while gaining a platform that finally fits its cloud and engineering strategy. The same batch runs every night, but it runs on new terms: the same batch, a new breath.