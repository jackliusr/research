# Apache Airflow — the orchestrated day

> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
> **Topic:** Apache Airflow Platform Deep-Dive — architecture, DAG authoring, operations, and ecosystem for the data-orchestration era
> **Audience:** Solution Architects, Data Platform Engineers, Batch Operations Managers, DevOps/Platform Teams, Banking IT Decision-Makers
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Version:** 1.0 — August 2026

---

## Table of Contents

1. [Executive Summary — The Orchestrated Day](#1-executive-summary--the-orchestrated-day)
2. [Lineage — From Airbnb to Apache 3.x](#2-lineage--from-airbnb-to-apache-3x)
3. [Architecture — Components and Execution Model](#3-architecture--components-and-execution-model)
4. [DAG Authoring — Core Concepts](#4-dag-authoring--core-concepts)
5. [DAG Authoring — Data, Configuration, and the Providers Ecosystem](#5-dag-authoring--data-configuration-and-the-providers-ecosystem)
6. [Scheduling Semantics — Trigger Rules, Catchup, Backfill, and Timetables](#6-scheduling-semantics--trigger-rules-catchup-backfill-and-timetables)
7. [Testing and CI/CD for DAGs](#7-testing-and-cicd-for-dags)
8. [Operations — Retries, Deadlines, Alerting, and Observability](#8-operations--retries-deadlines-alerting-and-observability)
9. [Capacity Planning and Managed Services](#9-capacity-planning-and-managed-services)
10. [Security, Secrets, and Audit](#10-security-secrets-and-audit)
11. [The Banking Angle — EOD Orchestration at Cymbal Bank](#11-the-banking-angle--eod-orchestration-at-cymbal-bank)
12. [Cymbal Bank Worked Example — payments_eod in Production](#12-cymbal-bank-worked-example--payments_eod-in-production)
13. [Claims Audit and What Could Not Be Verified](#13-claims-audit-and-what-could-not-be-verified)
14. [Glossary and References](#14-glossary-and-references)

---

## 1. Executive Summary — The Orchestrated Day

### Purpose of This Guide

This guide is the **platform deep-dive** for Apache Airflow: what it is, how it is built, how workflows are authored, how it is operated, and how it behaves in a regulated banking environment. It is the technical companion to two sibling guides in this repository and does not re-derive their content:

- The [Control-M to Airflow Migration Guide](control_m_to_airflow_migration_guide.md) covers the *migration angle* — semantic mapping, phased cutover, coexistence, and the Cymbal Bank migration scenario (201 active jobs, 18 Control-M folders).
- The [Open-Source Workload Automation Alternatives Guide](open_source_workload_automation_alternatives_guide.md) covers the *landscape angle* — how Airflow compares with Dagster, Prefect, Temporal, Kestra, and the commercial WLA incumbents, and why it is the recommended target.

This guide answers a different question: **once Airflow is the platform, how does it actually work?** It is written as internal platform documentation for Cymbal Bank, a Singapore-headquartered bank whose end-of-day (EOD) batch must complete by 07:00 SGT, with MAS Notice 610 and TRM regulatory submissions carrying hard deadlines.

### The Platform in One Paragraph

Apache Airflow is an open-source workflow orchestration platform in which **workflows are Python programs** — a directed acyclic graph (DAG) of tasks and dependency edges declared in code, versioned in Git, tested with pytest, and deployed through CI/CD. A scheduler parses the DAG files, materializes a *DAG run* for each schedule interval, and hands ready tasks to an *executor* (local processes, Celery workers, or Kubernetes pods) that runs them. Every run, task state, variable, and connection is recorded in a relational metadata database; a webserver UI and a stable REST API expose the same state for humans and machines. The result is an orchestrator whose scheduling intent — calendars, deadlines, retries, failure paths — lives in reviewable, testable code rather than in a configuration database.

### What This Guide Contains

- Sections 2–3: lineage and release history; architecture (scheduler, webserver, metastore, triggerer, executors, DAG bundles).
- Sections 4–8: DAG authoring, scheduling semantics, testing/CI-CD, and operations (retries, deadline alerting, metrics).
- Sections 9–12: capacity planning, managed services, security/secrets/audit, the banking angle, and a production `payments_eod` DAG.
- Sections 13–14: claims audit, unverified items, glossary, and references.

### Version Assumptions

As of August 2026 the current stable line is **Airflow 3.3.x** (3.3.1 released August 12, 2026), with the 2.x line still maintained (2.11.2, released March 14, 2026) for enterprises that have not yet upgraded. This guide is written against the **3.x documentation set** (stable docs = 3.3.1), and explicitly flags where Airflow 3 removed or replaced 2.x behavior — most importantly the removal of the classic `sla`/`sla_miss_callback` mechanism in favor of **Deadline Alerts** (Section 8). Banks migrating from Control-M via the sibling migration guide should read its Section 3 (which profiles Airflow against the 2.x vocabulary) together with this guide's Airflow-3 corrections.

---

## 2. Lineage — From Airbnb to Apache 3.x

### Origins

Apache Airflow was created at **Airbnb** by **Maxime Beauchemin** starting in **October 2014**, to manage the company's increasingly complex data workflows. It was open-sourced in **2015** — Beauchemin's public post "Airflow: a workflow management platform" is dated June 2, 2015, and the first release, **1.0.0**, is dated **June 3, 2015**. Airflow is written in **Python**, licensed under **Apache License 2.0**, and designed around the principle of *configuration as code*: because workflows are Python, they can use loops, imports, libraries, conditionals, and tests — capabilities no XML- or database-defined scheduler offers. The project entered the **Apache Incubator in March 2016** (the project's own announcement of March 19, 2016 records the application) and became a **top-level Apache Software Foundation project in January 2019**.

### The Release Timeline That Matters

| Date | Event | Why it matters |
|---|---|---|
| Oct 2014 | Project started at Airbnb (Maxime Beauchemin) | Birth of the DAG-native model |
| Jun 3, 2015 | Open-sourced; release 1.0.0 | First public release |
| Mar 2016 | Apache Incubator entry | ASF governance, community growth |
| Jan 2019 | Top-level Apache project | Maturity milestone |
| Dec 17, 2020 | **Airflow 2.0** | TaskFlow `@task` (AIP-31), stable REST API (AIP-32), scheduler HA (AIP-15), Task Groups (AIP-34), new UI |
| Oct 11, 2021 | **Airflow 2.2** | **Deferrable operators/triggerer** (AIP-40), custom **Timetables** (AIP-39), `logical_date`/`data_interval` naming |
| Sep 2022 | **Airflow 2.4** | Datasets / data-aware scheduling (precursor of 3.x Data Assets) |
| Oct 2024 | **Airflow 2.10** | Multi-executor configuration; audit-log era in the 2.x line |
| Apr 22, 2025 | **Airflow 3.0** | The biggest release in the project's history (below) |
| Jul 6, 2026 | **Airflow 3.3.0** | Task/Asset State Store (AIP-103), Language Task SDK for Java/Go (AIP-108), pluggable retry policies |
| Aug 12, 2026 | **Airflow 3.3.1** | Current stable (as of this writing) |

### Airflow 2.0 — The Rewrite (December 17, 2020)

Airflow 2.0 was a deliberate reset of the 1.x platform. Verified from the official 2.0 announcement:

- **TaskFlow API (AIP-31)** — `@task`-decorated Python functions replace manual `PythonOperator` + XCom plumbing; data flows through function return values.
- **Fully specified REST API (AIP-32)** — the previously experimental API became a supported interface with a comprehensive OpenAPI specification.
- **Scheduler performance and HA (AIP-15)** — the scheduler was significantly re-architected (faster task starts, a DAG-processing pipeline) and became **HA-compatible**: multiple scheduler instances cooperate through the database (requiring PostgreSQL or MySQL 8+).
- **Task Groups (AIP-34)** — visual/logical grouping of tasks inside a DAG.

### Airflow 3.0 — The Architecture Shift (April 22, 2025)

Airflow 3.0 was announced as the biggest release in the project's history — the official announcement reports **over 30 million monthly downloads** (up 30× since 2020), **80,000 organizations** using Airflow (up from 25,000 in 2020), over 30% of users using it for MLOps, and ~10% for GenAI workflows. Verified headline changes:

- **DAG versioning (AIP-65/AIP-66)** — a DAG run executes the DAG version that was current at start time, even if a new version is uploaded mid-run; the UI associates every run with its DAG version, code, and logs. A *major* auditability upgrade for banks (Section 10).
- **Scheduler-managed backfills (AIP-78)** — backfills run inside the scheduler, startable and monitorable from the UI/API.
- **Task Execution Interface / Task SDKs (AIP-72)** — Airflow evolves toward a client-server architecture with an **API server**; the Python Task SDK preserves backward compatibility, Task SDKs for other languages (Go first, then Java) arrived in 3.3, and the interface underpins **task isolation** for multi-team security.
- **Event-driven scheduling and Data Assets (AIP-74/75/82)** — Datasets evolve into **Data Assets** with asset-centric syntax and *Watchers*; external event-driven scheduling triggers DAGs from a "Common Message Bus" (AWS SQS ships out of the box).
- **UI and CLI modernization (AIP-38/79/81/84)** — UI rewritten on **React and FastAPI** (REST API moved to FastAPI); Flask AppBuilder moved into a provider package; the CLI was split with remote, API-driven operations in the `airflowctl` provider package.
- **Executor landscape changes** — statically-coded hybrids (`LocalKubernetesExecutor`, `CeleryKubernetesExecutor`) are no longer supported (use multi-executor configuration); the **Edge executor** (AIP-69) arrived as a provider package; `SequentialExecutor` is gone from the 3.x docs (Section 3).
- **SLA removed (AIP-86)** — the classic `sla`/`sla_miss_callback` was removed in 3.0 and replaced in **3.1 by Deadline Alerts** (Section 8). The single most important behavioral change for Control-M-style quantitative SLAs.

### The 3.x Cadence

The 3.x line has shipped steadily: 3.0.6, 3.1.4–3.1.8 (through March 2026), 3.2.0–3.2.2 (April–May 2026), 3.3.0 (July 6, 2026), and **3.3.1 (August 12, 2026)** — the current stable. The 2.x line continues to receive maintenance releases (2.11.2, March 14, 2026) for enterprises that have not yet moved; the provider ecosystem is versioned independently of core (Section 5).

## 3. Architecture — Components and Execution Model

### The Component Map

Airflow is a set of cooperating processes around one relational database. The 3.x component set:

| Component | Role |
|---|---|
| **Scheduler** | The orchestration brain: parses DAG code, creates DAG runs per schedule, evaluates dependencies, hands ready tasks to the executor. Executor logic runs *inside* the scheduler process. HA-compatible since 2.0 — multiple schedulers cooperate through the metadata DB. |
| **DAG processor** | The parsing pipeline that reads DAG files/bundles on an interval (`dag_dir_list_interval`, `min_file_process_interval`), with parallel parse processes (`parsing_processes`). In 3.x it operates on **DAG bundles** and feeds the scheduler serialized DAGs. |
| **Webserver / UI** | The React interface (FastAPI-backed in 3.x): grid, graph, gantt, logs, admin, asset views. Serves the stable REST API (`/api/v1`, OpenAPI-specified). |
| **Metadata database** | Relational store of DAGs, runs, task instances, variables, connections, XComs, audit/event logs. Backends per the docs: **PostgreSQL 13–17**, **MySQL 8.0/8.4/Innovation**, **SQLite 3.15+ (dev only)**. Production needs PostgreSQL or MySQL. |
| **Executor** | The pluggable mechanism that runs tasks: local processes, Celery workers, Kubernetes pods, or provider executors (below). |
| **Workers** | Celery-based execution: long-lived processes that pull tasks from the broker queue and run them. |
| **Triggerer** | Lightweight process (since 2.2) running async **triggers** for deferrable operators, freeing worker slots while tasks wait. HA with optional trigger queues (`--queues`). |
| **API server** | New in 3.x: the FastAPI server implementing the Task Execution Interface; the CLI and Task SDKs talk to it. |
| **DAG bundles** | The Airflow 3 replacement for the single `DAGS_FOLDER`: versioned sources of DAG code — local directories, **Git repositories (git-sync)**, cloud object storage, container images — refreshed and parsed by the DAG processor. Bundles make DAG versioning possible. |

### How a Task Actually Runs

1. The DAG processor parses each DAG file in the bundle and stores a serialized DAG in the metadata DB.
2. The scheduler materializes a **DAG run** per due interval, creates **task instances**, and computes which are *runnable* (upstream done, trigger rule satisfied, pool slot free, executor capacity available).
3. Runnable tasks go to the executor: `LocalExecutor` forks processes on the scheduler host; `CeleryExecutor` publishes to a broker queue (**Redis or RabbitMQ**) that workers consume; `KubernetesExecutor` launches **one pod per task**.
4. Workers/pods run the operator's `execute()`, write task logs, push XComs, and report state back through the metadata DB (or the Task Execution API in 3.x) — the same state the UI, REST API, and metrics observe, and the transitions that fire callbacks.

### The Executor Landscape in Airflow 3

Verified against the 3.3.1 executor documentation, which groups executors into *local* and *remote* (queued/batch and containerized):

| Executor | Type | Where it ships | Notes |
|---|---|---|---|
| **LocalExecutor** | Local | Core (default) | Runs task processes in the scheduler host; simplest production option for single-machine installs. |
| **CeleryExecutor** | Remote, queued | `apache-airflow-providers-celery` | Distributed workers pulling from a **Redis/RabbitMQ** broker; persistent workers, low latency, noisy-neighbor trade-off. |
| **KubernetesExecutor** | Remote, containerized | `apache-airflow-providers-cncf-kubernetes` | One pod per task; per-task isolation and resource control; pod startup latency. |
| **BatchExecutor / EcsExecutor** | Remote | `apache-airflow-providers-amazon` | AWS Batch or ECS as the execution plane. |
| **EdgeExecutor** | Remote | `apache-airflow-providers-edge3` | New in the 3.0 era (AIP-69): tasks on edge devices outside the DC/cloud, built on the Task Execution Interface. |
| ~~SequentialExecutor~~ | Local | Removed in 3.x | The 1.x/2.x single-process executor no longer appears in the 3.x executor docs; `LocalExecutor` is now the default. |
| ~~LocalKubernetes / CeleryKubernetes~~ | Hybrid | **Unsupported from 3.0.0** | The statically-coded hybrids misused the `queue` field; the documented replacement is multi-executor configuration. |

Two Airflow-3 facts matter for capacity work: **multi-executor configuration (since 2.10)** — `executor = LocalExecutor,CeleryExecutor` in `[core]` — lets a deployment run different executors per task/DAG (per-task `executor=`, aliases, per-team executors), with metrics published per executor; and executor choice drives the EOD window (Section 9) — Celery means sizing persistent workers, Kubernetes means sizing pod quota and startup time (a burst of 40 parallel tasks needs 40 pods or 40 worker slots).

### Deferrable Operators and the Triggerer

Introduced in **Airflow 2.2 (AIP-40, "Deferrable Tasks")**: a task that would otherwise *poll* for a condition (a file, an external job, a time) instead **defers** — it releases its worker slot and registers an async **trigger** with the lightweight **triggerer** process, which resumes the task when the condition occurs. The triggerer runs multiple instances for HA, optionally partitioned by `--queues`. Sensors expose this via `deferrable=True` (with `mode="reschedule"` as the older middle ground); the docs' comparison:

| mode="reschedule" | deferrable=True |
|---|---|
| Continuously reschedules itself until condition met | Pauses execution when idle, resumes on condition change |
| Higher resource use (repeated execution) | Lower resource use (frees worker slots while waiting) |
| Built into `BaseSensorOperator` | Requires the operator to implement deferral |

For Cymbal Bank's file-arrival and cross-DAG waits, deferrable sensors are the default choice (Section 11).

---

## 4. DAG Authoring — Core Concepts

### The DAG Object

A DAG is a Python object (or `@dag`-decorated function) declaring tasks and edges. Essential parameters: `dag_id`, `schedule` (cron, timedelta, timetable, or `None` for manual/event-triggered), `start_date`, `catchup`, `default_args` (retries, delays, owner, callbacks), `tags`, `max_active_runs`, and — new in 3.1 — `deadline` (Section 8). Note the 3.x import layout: core definitions come from `airflow.sdk`, and the classic operators live in the `apache-airflow-providers-standard` package (`airflow.providers.standard.operators.*`), installed as part of the default set.

```python
from datetime import datetime, timedelta
from airflow.sdk import DAG
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.standard.operators.python import PythonOperator

default_args = {
    "owner": "batch-ops",
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "retry_exponential_backoff": True,
    "max_retry_delay": timedelta(hours=1),
}
with DAG(
    dag_id="posting_eod",
    schedule="0 1 * * *",                 # 01:00 Asia/Singapore
    start_date=datetime(2026, 1, 1),
    catchup=False,
    default_args=default_args,
    tags=["eod", "posting"],
) as dag:
    extract = BashOperator(task_id="extract_ledger", bash_command="run_extract.sh")
    post = PythonOperator(task_id="post_gl", python_callable=post_to_gl)
    extract >> post
```

### Tasks, Operators, and the `>>` Edge

- **Task** — one unit of work in a DAG (an operator instance, a decorated function, or a sensor).
- **Operator** — the class that defines *what* a task does. Core/standard operators: `BashOperator`, `PythonOperator`, `EmptyOperator`, `EmailOperator`, plus the hundreds shipped by provider packages (Section 5).
- **Dependencies** — the `>>` operator (and its inverse `<<`) declares edges: `extract >> post >> report`. Dependencies are structural edges in the graph, not database conditions — the Control-M-to-Airflow translation of IN-COND/OUT-COND (see the [migration guide](control_m_to_airflow_migration_guide.md), Section 4).

### TaskFlow: `@task` and `@dag`

Since 2.0 (AIP-31), decorated functions make data passing first-class. Return values become XComs automatically and are passed as arguments downstream; the DAG structure is the call graph:

```python
from airflow.sdk import DAG
from airflow.decorators import task
from datetime import datetime

@dag(dag_id="recon_eod", schedule="0 3 * * *", start_date=datetime(2026, 1, 1), catchup=False)
def recon_dag():
    @task
    def pull_gl_balance() -> dict:
        return {"gl_total": 1234567.89}
    @task
    def pull_nostro_balance() -> dict:
        return {"nostro_total": 1234567.89}

    @task
    def reconcile(gl: dict, nostro: dict) -> str:
        delta = abs(gl["gl_total"] - nostro["nostro_total"])
        return "MATCH" if delta == 0 else f"DRIFT:{delta}"
    reconcile(pull_gl_balance(), pull_nostro_balance())
recon_eod = recon_dag()
```

### Sensors — Waiting as a First-Class Task

A sensor is an operator that *waits*: `FileSensor` (a file appears), `ExternalTaskSensor` (a task in another DAG completes), `TimeSensor`/`DateTimeSensor` (a wall-clock time), plus hundreds of provider sensors. For EOD flows, sensors replace the shell-polling "wait for file" jobs of the Control-M era. Long waits should use deferrable variants:

```python
from airflow.providers.standard.sensors.filesystem import FileSensor
wait_for_swift_ack = FileSensor(
    task_id="wait_for_swift_ack",
    filepath="/data/inbox/SWIFT_ACK_{{ ds_nodash }}.txt",
    fs_conn_id="file_gateway",
    poke_interval=30,
    timeout=3600,                 # 60 minutes, then fail — the runbook takes over
    mode="reschedule",            # or deferrable=True with a deferrable sensor
)
```

`ExternalTaskSensor` is the sanctioned mechanism for **cross-DAG edges** (e.g., `recon_eod` waiting on `payments_eod`): it matches a task in another DAG by `external_dag_id`/`external_task_id` and aligns on `execution_date` (in 3.x terminology, `logical_date`). Document every cross-DAG edge, set generous-but-bounded `timeout`, and cover alignment in unit tests (Section 7).

### Task Groups

`TaskGroup` (2.0+, AIP-34) groups tasks into a collapsible sub-graph in the UI without the overhead of a sub-DAG:

```python
from airflow.utils.task_group import TaskGroup

with DAG("payments_eod", ...) as dag:
    with TaskGroup("swift_leg") as swift_leg:
        gen = BashOperator(task_id="generate_mt103", bash_command="gen_mt103.sh")
        send = BashOperator(task_id="send_mt103", bash_command="send_mt103.sh")
        gen >> send
    with TaskGroup("meps_leg") as meps_leg:
        build = BashOperator(task_id="build_fast_file", bash_command="build_fast.sh")
        submit = BashOperator(task_id="submit_fast", bash_command="submit_fast.sh")
        build >> submit
    swift_leg >> meps_leg
```

Task-group ids become task-id prefixes in the UI and logs (`swift_leg.generate_mt103`) — naming discipline matters.

### Dynamic and Config-Driven DAGs

Because DAGs are code, they can be generated from configuration: a YAML/JSON job registry becomes one DAG per flow (the migration guide's "one DAG per batch flow" pattern) via a factory function, or a single DAG fans out per record via **dynamic task mapping** (`task.expand(...)`, 2.3+). The bank's discipline: generated DAGs must be deterministic (same config → same graph), the config is pinned in Git, and CI validates both — a non-deterministic DAG is an audit problem (Section 10).

### DAG Versioning (Airflow 3)

Airflow 3 ties every DAG run to the exact DAG version that started it — code, task structure, and logs — so a mid-run deploy cannot corrupt an in-flight EOD batch, and a rerun can target the original version. For regulated batch, this closes a real gap that 2.x left open (Section 10, auditability).

## 5. DAG Authoring — Data, Configuration, and the Providers Ecosystem

### XCom — Passing Data Between Tasks

XCom ("cross-communication") is Airflow's small key-value message store: a task pushes a value (often via `return` in TaskFlow, or `ti.xcom_push`), and a downstream task pulls it (`ti.xcom_pull(task_ids=..., key=...)`). The docs' standing advice is unambiguous: XComs are **only designed for small amounts of data; do not use them to pass around large values, like dataframes**. Concretely:

- The default backend (`BaseXCom`) stores values in the metadata DB. The practical ceiling is the DB column type — historically cited as roughly **48 KB on SQLite** (a legacy figure; the current docs no longer quote it, ⚠) and far larger on PostgreSQL/MySQL — but the *design* rule is kilobytes, not megabytes.
- **XComs are cleared on retry** if the producing task was not successful, so they cannot persist state across retries.
- For larger payloads, Airflow 3 documents the **Object Storage XCom backend** (from `apache-airflow-providers-common-io`) and custom backends via the `xcom_backend` configuration option.

```python
@task
def generate_filename(business_date: str) -> str:
    return f"MT103_{business_date.replace('-', '')}.txt"
@task
def send_file(name: str):
    # name arrives as a string pulled from the upstream task's XCom
    print(f"Sending {name}")
```

The bank's rule of thumb: **XCom for filenames, keys, counts, and checksums; object storage (or a shared file system) for payloads.** Never push transaction data through XCom — the metadata DB is not a data bus.

### Variables and Params

- **Variables** — key-value settings stored in the metadata DB (or a secrets backend), readable at runtime (`Variable.get("env")`). They are configuration, not secrets: credentials must never live in Variables (Section 10).
- **Params** — run-scoped parameters attached to a DAG run, settable via UI/API (`dag_run.conf`); the idiomatic way to parameterize a manual run ("backfill date X", "skip the SWIFT leg").

```python
with DAG("reporting_eod", schedule="0 4 * * *", start_date=..., catchup=False,
         params={"as_of_date": Param(default="{{ ds }}", type="string")}) as dag:
    ...
```

### Connections

Connections are named, typed credential bundles (host, port, login, password, extras) that operators reference by `conn_id` — `ssh_conn_id="swift_gateway"`, `postgres_conn_id="edw"`. They are resolved through the **secrets backend search path**: the metadata DB by default, or external secret stores (AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault, and others) — Section 10. Airflow 3's connection test feature verifies a connection from the UI/API before a DAG run depends on it.

### Pools — Controlling Parallelism

A **pool** is a named concurrency counter: tasks assigned to a pool occupy slots, and when the pool is full, runnable tasks queue until slots free up (ordered by priority weight). Verified facts from the pools documentation:

- Tasks without an explicit pool land in `default_pool`, initialized with **128 slots** (modifiable via UI/CLI, not removable).
- A task can occupy more than one slot via `pool_slots` (useful for "heavy" tasks that are computationally worth two light ones).
- Pools are the Airflow counterpart of Control-M *resources/agent pools* — see the mapping table in the [migration guide](control_m_to_airflow_migration_guide.md), Section 4.

```python
submit_to_gateway = BashOperator(
    task_id="submit_to_gateway",
    bash_command="submit.sh",
    pool="gateway_slots",
    pool_slots=2,          # this task is worth two slots
)
```

For Cymbal Bank, pools are the safety valve of the EOD window: `gateway_slots` caps concurrent MFT submissions to the file gateways, `mainframe_submit` caps concurrent z/OS submissions, and `regulatory_parallel` caps the MAS-return generation fan-out.

### The Providers Ecosystem

Airflow's reach comes from **provider packages** — independently versioned PyPI distributions (`apache-airflow-providers-*`) that add operators, hooks, sensors, transfer operators, secret backends, and even executors. Verified from the providers documentation: the Apache Airflow community **develops and maintains more than 80 providers**. Highlights for a bank:

| Provider package | What it brings |
|---|---|
| `apache-airflow-providers-standard` | The classic core operators (`BashOperator`, `PythonOperator`, `EmptyOperator`, `EmailOperator`) in the 3.x layout |
| `apache-airflow-providers-cncf-kubernetes` | `KubernetesPodOperator` (run a task in its own pod), `KubernetesExecutor` |
| `apache-airflow-providers-postgres` / `-mysql` / `-common-sql` | `PostgresOperator`, `SQLExecuteQueryOperator`, SQL check operators |
| `apache-airflow-providers-sftp` / `-ssh` | `SFTPOperator`, `SSHOperator` — the file-gateway boundary |
| `apache-airflow-providers-amazon` / `-google` / `-azure` | Cloud-native operators, plus the AWS executors and secret backends |
| `apache-airflow-providers-celery` / `-redis` | CeleryExecutor and its broker support |
| `apache-airflow-providers-common-io` | Object storage XCom backend, filesystem abstractions |
| `apache-airflow-providers-smtp` | `SmtpNotifier` for email alerting |
| Community notifiers | Slack, PagerDuty-style webhook notifiers via the notifications framework (Section 8) |

Two ecosystem facts matter for platform planning: providers version independently of core (a 3.3.1 deployment may run provider X.Y.Z), and provider selection must be pinned in the image/requirements with the same discipline as core — the migration guide's CI/CD section treats exactly this.

The `KubernetesPodOperator` deserves special mention as the escape hatch for "this task needs its own environment": it runs an arbitrary container image as a task pod, which is how a bank runs legacy utilities, vendor CLIs, or language runtimes that do not belong in the Airflow image:

```python
from airflow.providers.cncf.kubernetes.operators.pod import KubernetesPodOperator

run_swift_client = KubernetesPodOperator(
    task_id="run_swift_client",
    image="registry.cymbal.internal/swift-client:3.4.2",
    cmds=["swift_client", "send", "--file", "/data/out/{{ ds_nodash }}.mt103"],
    namespace="batch",
    in_cluster=True,
    pool="gateway_slots",
    get_logs=True,
    execution_timeout=timedelta(minutes=20),
)
```

---

## 6. Scheduling Semantics — Trigger Rules, Catchup, Backfill, and Timetables

### Trigger Rules — When a Task May Run

A task's `trigger_rule` decides when it becomes runnable given the states of its *direct upstreams*. Verified from the 3.3.1 DAGs documentation, the current rule set (11 rules — two more than the classic eight):

| Trigger rule | Runs when… |
|---|---|
| `all_success` (default) | All upstream tasks succeeded |
| `all_failed` | All upstream tasks are `failed` or `upstream_failed` |
| `all_done` | All upstream tasks finished (any state) |
| `all_done_setup_success` | Like `all_done`, but if the task has upstream *setup* tasks, at least one must have succeeded (default rule for teardown tasks) |
| `all_done_min_one_success` | All non-skipped upstream tasks are done and at least one succeeded |
| `one_failed` | At least one upstream failed (does not wait for the rest) |
| `one_success` | At least one upstream succeeded (does not wait for the rest) |
| `none_failed` | No upstream is `failed`/`upstream_failed` (success *or skipped* counts) |
| `none_failed_min_one_success` | No upstream failed **and** at least one succeeded (the branch-join workhorse) |
| `none_skipped` | No upstream is `skipped` |
| `always` | No dependencies — run at any time |

Three rules form the canonical banking pattern set: `one_failed` for compensation/alerting tasks ("if anything failed, page ops"), `none_failed_min_one_success` for joins after branches, and `always` for standalone monitors; the two newer rules (`all_done_setup_success`, `all_done_min_one_success`) support the **setup/teardown** pattern (2.6+), useful for test-like DAGs but rarely for production EOD.

```python
suspense_alert = BashOperator(
    task_id="suspense_alert",
    bash_command="page_batch_ops.sh",
    trigger_rule="one_failed",     # fire only when an upstream leg failed
)
```

### Catchup vs Backfill

- **Catchup** — a DAG-level behavior: with `catchup=True` (and a past `start_date`), the scheduler creates DAG runs for every missed interval since `start_date` on deploy. For an EOD bank batch, `catchup=False` is the default posture (deploying a DAG mid-day must not create phantom historical runs), with explicit backfill used when historical runs are actually wanted.
- **Backfill** — the deliberate replay of a date range. In **Airflow 3, backfills are scheduler-managed (AIP-78)**: started from the UI or API, monitored in the UI, and run within the scheduler for control and diagnostics — replacing the 2.x `airflow dags backfill` CLI workflow as the primary mechanism. This is the tool the migration guide's Section 5f uses to validate migrated DAGs against 90 days of Control-M history.

### Timetables — Beyond Cron

Custom **Timetables** (2.2, AIP-39) define *when* a DAG runs as a pluggable Python class, which is how a bank encodes its holiday-aware calendar — the single most common source of "the batch ran on the wrong day" incidents after migration (see the migration guide, Section 7). A timetable returns the next run time and the data interval for each run; the docs demand timetables be idempotent and fast, since the scheduler evaluates them on every loop. A bank holiday calendar is typically a timetable (or a holiday-gate task) that consumes the SG public-holiday list — the [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) documents the compliance context.

### Event-Driven Scheduling and Data Assets (Airflow 3)

Airflow 3 made scheduling event-native: **Data Assets** (the evolution of 2.4 datasets) with *Watchers* let DAGs react to data being produced or updated, and **external event-driven scheduling (AIP-82)** triggers DAG runs from a message bus (AWS SQS ships as the first integration). For Cymbal Bank this is the natural mechanism for *intraday* flows (a file lands → a DAG fires) without polling — while the EOD batch itself stays on its deterministic cron/timetable schedule, because 07:00 SGT does not move.

## 7. Testing and CI/CD for DAGs

DAGs are code and must travel the same path as any banking application change. The platform-level practices — branching strategy, environments, deployment pipelines, change management — are the property of the repo's [AI Platform Engineering Guide](ai_platform_engineering_guide.md); this section adds only the Airflow-specific testing and CI mechanics, and does not re-derive the platform guidance.

### Unit Testing with pytest

The standard stack is pytest with the Airflow test fixtures. The canonical checks per DAG:

- **Import, parse, and structure**: the DAG file imports cleanly (`DagBag(dag_folder=..., include_examples=False)`; assert no import errors), has no cycles (`dag.validate()`), unique task ids, and the expected edges — and contains no top-level I/O (a DAG file must be pure declaration; side effects at import time are a scheduler hazard).
- **Schedule validity**: the cron/timetable produces the expected run dates for a full-year matrix including SG public holidays (the migration guide's Section 5f calls this the acceptance test for schedule translation).
- **Behavioral**: `dag.test()` runs the DAG in-process against a test database; `dag.test(use_executor=True)` runs it against the configured executor for integration-style checks.

```python
from airflow.models.dagbag import DagBag

def test_payments_eod_structure():
    dagbag = DagBag(dag_folder="dags/payments/", include_examples=False)
    dag = dagbag.get_dag("payments_eod")
    assert dag is not None, f"import errors: {dagbag.import_errors}"
    dag.validate()  # no cycles
    assert dag.get_task("swift_leg.generate_mt103").downstream_task_ids == {"swift_leg.send_mt103"}
    assert dag.next_dagrun_info(None) is not None  # timetable produces a run
```

The docs' `dag.test()` pattern — ending a DAG module with `if __name__ == "__main__": dag.test()` — gives local smoke-testing, and the Airflow pytest plugin's `conf_vars` fixture lets tests flip configuration (e.g., point at a test metadata DB) without touching production settings.

### CI for DAGs

The minimum CI gate for a banking DAG repo (applied per pull request, per the AI Platform Engineering Guide's pipeline model):

| Stage | Command / check |
|---|---|
| Lint | `ruff check dags/` (plus type checking where adopted) |
| Parse | `airflow dags list-import-errors` in a test container that mirrors the production image — catches import-time failures before deploy |
| Unit tests | pytest suite above (structure + behavioral) |
| Bundle build | Build the DAG bundle/image (git-sync target or container image) and assert it is deterministic |
| Secret scan | Scan for credentials in DAG code and Variables defaults (see [Cybersecurity Guide](cybersecurity_guide.md)) |

Deployment is then either **git-sync** (the scheduler pulls the DAG bundle from the Git repo — native to Airflow 3 bundles, and the MWAA/Composer model) or **image-based** deploys (Astronomer-style: a new image per deploy, which also carries provider pinning). With Airflow 3's DAG versioning, a deploy mid-batch is no longer a corruption risk (Section 4), but the change-management freeze windows from the migration guide still apply during cutover waves.

---

## 8. Operations — Retries, Deadlines, Alerting, and Observability

### Retries and Exponential Backoff

Retries are per-task (via `default_args` or on the task): `retries`, `retry_delay`, `retry_exponential_backoff`, and `max_retry_delay`. With exponential backoff, the delay after attempt *n* grows (2^n × `retry_delay`, capped at `max_retry_delay`). The rule that matters most in banking is **idempotency**: a retried task re-executes the operator from scratch, so every task must be safe to re-run without double side effects (no double payments, no duplicate MAS submissions). The migration guide's Section 7 flags this as a behavioral change from Control-M; the platform answer is an idempotency review per task plus conservative retry policies on side-effectful tasks:

```python
default_args = {
    "retries": 3,
    "retry_delay": timedelta(minutes=2),
    "retry_exponential_backoff": True,
    "max_retry_delay": timedelta(minutes=30),
}
swift_send.retries = 1          # never auto-replay a live SWIFT submission more than once
swift_send.retry_delay = timedelta(minutes=10)
```

### SLAs → Deadline Alerts: The Airflow 3 Change

The classic 2.x mechanism — DAG/task `sla` timedelta plus `sla_miss_callback` — was **removed in Airflow 3.0** (AIP-86) and **replaced in 3.1 by Deadline Alerts**. This is the single most consequential Airflow-3 change for Control-M-style quantitative SLAs, and the sibling migration guide (written against 2.x vocabulary) must be read with it in mind. The official migration doc contrasts the two paradigms:

| Dimension | SLA (2.x) | Deadline Alerts (3.1+) |
|---|---|---|
| When checked | When the DAG run **finishes** — if it never finishes, the SLA is never checked | When the run **starts** — the scheduler stores the deadline and checks it continuously (default ~5s, `scheduler_heartbeat_sec`) |
| Reference point | `logical_date + sla` | `DeadlineReference` (e.g., `DAGRUN_LOGICAL_DATE`, `DAGRUN_QUEUED_AT`) + `interval` |
| Fires | `sla_miss_callback` | A callback (`AsyncCallback` in the triggerer, or `SyncCallback` in the executor) |
| Semantics | Miss detected only in hindsight | Fires "immediately" when the deadline passes — even mid-run |

For a bank, the Deadline Alert model is strictly better: a 06:45 SGT deadline that fires *at 06:45* while the batch is still running is an actionable page, whereas the 2.x model would only notice after the run completed (or never, if it hung). The pattern for Cymbal Bank:

```python
from airflow.sdk.definitions.deadline import DeadlineAlert, DeadlineReference
from airflow.providers.smtp.notifications.smtp import SmtpNotifier

with DAG(
    dag_id="regulatory_eod",
    schedule="0 2 * * *",
    start_date=datetime(2026, 1, 1),
    catchup=False,
    deadline=DeadlineAlert(
        reference=DeadlineReference.DAGRUN_LOGICAL_DATE,
        interval=timedelta(hours=4, minutes=30),   # 02:00 + 4h30m = 06:30 SGT hard deadline
        callback=SmtpNotifier(
            to="batch-ops-oncall@cymbal.internal",
            subject="🚨 {{ dag_run.dag_id }} missed its 06:30 SGT deadline",
            html_content="Run {{ dag_run.dag_run_id }} exceeded its deadline; batch ops must intervene.",
        ),
    ),
) as dag:
    ...
```

(Note: DAG-run *completion* deadlines — "must be done by 06:45" — are expressed the same way; the bank's batch window is a set of Deadline Alerts on the critical DAGs, mirroring the Control-M quantitative SLAs at 06:00–06:45 SGT.)

### Alerting — Callbacks and Notifiers

Airflow 3 formalizes **Notifiers** (classes usable inside `on_*_callback` arguments) alongside plain callable callbacks. The callback surface, verified from the 3.3.1 docs:

| Callback | Fires on |
|---|---|
| `on_execute_callback` | Task execution start |
| `on_success_callback` | Task (or DAG, at DAG level) success |
| `on_failure_callback` | Task (or DAG) failure — the primary alerting hook |
| `on_retry_callback` | A task retry |
| `deadline` callback | A Deadline Alert expiration (above) |

Notifiers include `SmtpNotifier` (email) and community-provided webhook notifiers (Slack-style) — PagerDuty-style incident routing is implemented as a custom Notifier posting to the bank's incident-management webhook. The design rule for batch ops: **the failure callback is the first line (page batch ops), the deadline callback is the second (page on-call), and metrics-based alerting is the third (the platform pages itself)** — Section 8b.

```python
def page_batch_ops(context):
    send_incident_webhook(
        summary=f"EOD task failed: {context['task_instance'].task_id}",
        severity="high",
    )
with DAG("payments_eod", default_args={"on_failure_callback": page_batch_ops}, ...) as dag:
```

### Observability — Metrics, Logs, Traces

Airflow emits a rich metrics surface. Verified from the 3.3.1 metrics documentation:

- **StatsD and OpenTelemetry** are both first-class metric transports (the docs cover `Setup - StatsD` and `Setup - OpenTelemetry`).
- Useful counters/gauges/timers include `scheduler.scheduler_loop_duration`, `scheduler.critical_section_duration`, `dagrun.duration.success/failed.{dag_id}`, `dagrun.schedule_delay`, per-executor slot metrics (`executor.open_slots.<executor>` in multi-executor mode), and `kubernetes_executor.pod_creation` — plus OpenLineage emit/extract timers.
- **Traces** are configurable (the docs have a dedicated Traces Configuration page) for distributed tracing of task execution.
- **Logs**: per-task-instance log files, shippable to external logging systems (CloudWatch, ELK, Splunk) — the docs' audit-log page shows the Elasticsearch shipping pattern; the bank's retention requirements come from the [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md).

The monitoring *themes* — golden signals, alerting tiers, SLOs — are the property of the sibling guides and the AI Platform Engineering Guide; the Airflow-specific instrumentation points above are what this guide adds. Minimum production alerts: scheduler heartbeat missing, DAG import errors, task failure rate, deadline misses, and batch-window overrun (via `dagrun.schedule_delay` and the deadline callbacks).

## 9. Capacity Planning and Managed Services

### Scheduler and Parsing Tuning

The scheduler is the heartbeat of the platform; the DAG processor is its digestion. Verified tuning surface from the 3.3.1 scheduler and DAG-file-processing docs:

| Knob | What it controls | EOD guidance |
|---|---|---|
| `parsing_processes` | Number of parallel DAG-parse processes | Raise for large DAG fleets; parsing is CPU-bound |
| `min_file_process_interval` | Minimum seconds between re-parses of a file | Lower = fresher DAGs, higher CPU; keep default unless deploy latency hurts |
| `dag_dir_list_interval` | How often the DAG folder/bundle is scanned | Bounds how quickly a new DAG appears |
| `scheduler_heartbeat_sec` | Scheduler loop cadence (also drives Deadline Alert checking) | Default 5s is the deadline-alert latency bound |
| `parallelism` / `dag_concurrency` / `max_active_runs` | Global / per-DAG / per-DAG-run task concurrency ceilings | Set from the 90-day peak-concurrency baseline, not from hope |

The sizing method is the migration guide's: **baseline from 90 days of Control-M run history** — peak concurrent task count, peak per-hour starts, longest chains — then size executor capacity to the peak with headroom, and use pools to cap runaway parallelism (Section 5). Monitor `scheduler.scheduler_loop_duration` and DAG import errors as the leading indicators of a scheduler approaching its ceiling.

### Executor Choice

| Scenario | Executor |
|---|---|
| Single host, small estate, testing | LocalExecutor |
| Distributed workers, steady throughput, existing Redis/RabbitMQ | CeleryExecutor |
| Bursty, heterogeneous workloads; per-task isolation; Kubernetes shop | KubernetesExecutor |
| Mixed / AWS-centric estates | Multi-executor configuration (per-task `executor=`); BatchExecutor/EcsExecutor or EdgeExecutor (provider-packaged) |

For Cymbal Bank's EOD window (peak concurrency from the 90-day baseline, one pod per task under Kubernetes), KubernetesExecutor with a warm pod pool is the current target; the migration guide's Section 7 sizing notes apply verbatim.

### Managed Services — Condensed

The full comparison — launch dates, integration profiles, cost models, banking fit — lives in the [migration guide](control_m_to_airflow_migration_guide.md), Section 6, and is not re-derived here. The platform-relevant facts, verified at primary sources:

- **Amazon Managed Workflows for Apache Airflow (MWAA)** — launched **November 2020** (announced November 24, 2020). Fully managed Airflow on AWS; DAGs synced from S3; VPC-only deployment, IAM, Secrets Manager integration.
- **Google Cloud Managed Service for Apache Airflow (formerly Cloud Composer)** — announced **May 2018** (TechCrunch, May 1, 2018); now branded "Managed Service for Apache Airflow". Tight GCP integration (BigQuery, Dataflow, Dataproc), private IP / Shared VPC / CMEK controls.
- **Astronomer (Astro)** — company founded **2015**; offers managed Airflow (public/private/hybrid) and **Astro Runtime**, its maintained Airflow image; image-based deploys fit GitOps.
- ⚠ **Microsoft "Apache Airflow Job"** — a SaaS managed Airflow integrated with Microsoft Fabric, listed in Wikipedia's managed-providers section; not otherwise verified this pass.

The decision questions (data-plane location, shared-plane acceptability, ops burden, version strategy) are the migration guide's Section 6 closing list. One platform note from this guide: **managed services lag core releases** — as of mid-2026, Airflow 3 support varies by vendor, so a bank standardizing on 3.x behavior (Deadline Alerts, DAG bundles, Task SDK) should verify the vendor's 3.x roadmap before committing (see the migration guide's comparison table).

---

## 10. Security, Secrets, and Audit

### Access Control

- The Airflow UI enforces **RBAC** with role-based permissions (roles per team: viewer, editor, admin, plus custom roles); in 3.x the auth layer is the pluggable **auth manager**, with the Flask-AppBuilder implementation moved into a provider package (AIP-79) while remaining backward compatible. **SSO/LDAP** enterprise federation rides the same mechanism (the bank's identity standards are the [Cybersecurity Guide](cybersecurity_guide.md)'s property).
- **API security**: the stable REST API supports token/JWT authentication, and the split CLI (airflowctl) routes remote operations through the API rather than direct DB access (AIP-81) — real hardening for multi-team deployments.
- **Task isolation** (Airflow 3, via the Task Execution Interface) keeps one team's task code from seeing another's process space — the documented security benefit of the 3.0 architecture shift.

### Secrets Backends

Credentials must never live in DAG code or Airflow Variables. The secrets-backend framework (verified from the 3.3.1 docs) resolves connections and variables through a **search path** of configured backends:

- **Core backend**: local filesystem secrets backend (file-based, for self-managed deployments).
- **Community-provided backends** (shipped by providers): AWS Secrets Manager, AWS SSM Parameter Store, GCP Secret Manager, HashiCorp Vault, Azure Key Vault, and others — the bank's Vault integration is the [Cybersecurity Guide](cybersecurity_guide.md)'s standard. **Custom backends** subclass `BaseSecretsBackend` (`get_connection()`/`get_conn_value()`, `get_variable()`, `get_config()`) and are wired via the `[secrets]` section (`backend` + `backend_kwargs`).
- **At-rest encryption**: values stored in the metadata DB (Variables, connection extras) are encrypted with a **Fernet** key — the encryption-at-rest layer under Security → Secrets → Encryption at rest.

Rotation works without DAG redeploys because DAGs reference connection *ids*, not credentials; bake rotation into the runbook (the migration guide's Section 7 has the same rule from the migration side).

### Audit Logs and Event Logs

Airflow 3 documents a dedicated **audit-logging surface** (Security → Audit Logs in Airflow) that distinguishes *audit logs* (who did what: user actions, task-instance events, CLI events, custom events) from *event logs* (operational logs). Verified facts:

- The metadata `log` table records events like `post_variable`, `patch_connection`, `delete_variable`, DAG-run and task-instance transitions, with `owner` and `extra` columns — directly queryable for compliance reporting; the docs publish patterns like "all actions by a user in the last 24 hours" that map onto MAS-style audit requests.
- Task logs are retrievable per try number via the REST API (`/api/v1/dags/{dag_id}/dagRuns/{dag_run_id}/taskInstances/{task_id}/logs/{try_number}`) and shippable to external logging systems.

### Metadata Integrity and Retention

For a regulated bank, the Airflow metadata DB plus task logs *are* the audit record of the batch:

- **DAG versioning in Git** — every DAG change travels PR review, branch protection, and signed commits (per the AI Platform Engineering Guide); Airflow 3's DAG versioning then ties each run to the exact code that produced it.
- **Metadata integrity and retention** — the metadata DB must be backed up and point-in-time recoverable like any system of record; the ERD/schema docs make the schema inspectable for auditors. Define retention per the [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) (typically multi-year for regulatory records); archive task logs to immutable object storage and archive DAG bundles so historical runs remain reproducible (3.x bundle versioning supports exactly this).
- **Segregation** — restore, staging, and DR environments must not share the production metadata DB; the platform's environment model in the AI Platform Engineering Guide governs this.

## 11. The Banking Angle — EOD Orchestration at Cymbal Bank

### The Batch as a Control

For Cymbal Bank, the EOD batch is not a convenience — it is a control. The EOD batch must complete by **07:00 SGT** for start-of-day position; **MAS Notice 610** and **TRM-related regulatory submissions** carry hard deadlines that cannot slip. The full regulatory context is in the [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) and the [Cybersecurity Guide](cybersecurity_guide.md); the migration scenario (201 active jobs in 18 Control-M folders) is in the [migration guide](control_m_to_airflow_migration_guide.md), Section 8. This section and Section 12 add the *platform* view: how those flows are built and operated once Airflow is the platform.

### DAG Naming and Layout Conventions

The repo-wide convention from the migration guide, applied as platform standards:

| Artifact | Convention | Example |
|---|---|---|
| DAG id | Lowercase, `<flow>_eod` suffix | `payments_eod`, `posting_eod`, `recon_eod`, `reporting_eod`, `regulatory_eod`, `housekeeping` |
| DAG file | `dags/<flow>/<flow>_eod.py` (one flow per directory) | `dags/payments/payments_eod.py` |
| Task id | Short, prefix-free | `extract`, `validate`, `swift_gen`, `swift_send` |
| TaskGroup id | `<leg>_leg` | `swift_leg`, `meps_leg` |
| Cross-DAG edge | `ExternalTaskSensor` in the consuming DAG | `recon_eod` waits on `payments_eod` and `posting_eod` |
| Holiday calendar | Python module `calendars/sg_bank_holidays.py`, consumed by timetables/holiday gates | imported by DAGs |
| Pool | `gateway_slots`, `mainframe_submit`, `regulatory_parallel`, `default_pool` | caps concurrency per boundary |

### The EOD Window as a Set of Deadlines

The batch window is expressed as a deadline map. Control-M's quantitative SLAs (06:00–06:45 SGT for critical flows, per the migration guide) become **Deadline Alerts** on the 3.x platform (Section 8):

| Flow | Scheduled (SGT) | Deadline (SGT) | Mechanism |
|---|---|---|---|
| Payments (`payments_eod`) | 22:00 D−1 | 06:45 | DeadlineAlert (logical-date reference) + failure callbacks |
| Posting (`posting_eod`) | 01:00 | 06:15 | DeadlineAlert |
| Reconciliation (`recon_eod`) | 03:00 | 06:30 | DeadlineAlert + `one_failed` suspense alert |
| Reporting (`reporting_eod`) | 04:00 | 06:30 | DeadlineAlert |
| Regulatory (`regulatory_eod`) | 02:00 | 06:30 | DeadlineAlert (hard deadline; MAS) |
| Housekeeping | 05:30 | none | opportunistic |

Cross-DAG edges are the reconciliation hotspots: `recon_eod` waits on `payments_eod`/`posting_eod`, `regulatory_eod` waits on `recon_eod`/`reporting_eod` — four explicit edges, each with aligned `logical_date` semantics, bounded sensor timeouts, and unit tests (Section 7).

### Data Pipelines, Lineage, and Backfills

The orchestration layer sits above the bank's data platform, and the repo's data guides provide the platform context:

- [Data Pipeline Guide](data/data_pipeline_guide.md) — pipeline patterns that Airflow DAGs execute (extract, validate, transform, load).
- [Backfill Data Engineering](data/backfill_data_engineering.md) — the replay semantics that Airflow 3's scheduler-managed backfills operationalize (Section 6).
- [Data Lineage Tools](data/data_lineage_tools.md) and [DataOps Guide](data/dataops_guide.md) — lineage tooling consuming Airflow's OpenLineage events (the `ol.*` metrics in Section 8), and the operating model DAG design must serve.

The platform rule: **Airflow orchestrates; it does not transform**. Heavy transforms belong to the data platform (SQL engines, Spark, the data pipeline guide's stack); Airflow tasks should be thin — start jobs, move files, check results, and page when the world misbehaves.

### Auditability on the 3.x Platform

The banking-specific audit story (Section 10) maps onto the platform features as follows: DAG changes are Git-reviewed (change trail), Airflow 3 DAG versioning ties every run to its code (run evidence), audit logs capture who changed Variables/connections (compliance queries), the metadata DB plus archived logs form the retention record (typically 7 years per the MAS guide), and bundle versioning keeps historical runs reproducible. Nothing in that chain is a bolt-on; it is the platform's native shape on 3.x.

---

## 12. Cymbal Bank Worked Example — payments_eod in Production

This is the platform companion to the migration guide's Section 8 scenario: the same `payments_eod` flow, designed as a production DAG on Airflow 3.x — task groups, retry policy, deadline alerting, secrets handling, cross-DAG signaling, and the mainframe boundary — not a re-derivation of the migration plan.

### The DAG

```python
"""payments_eod — EOD payments chain (SWIFT + MEPS+/FAST).

Scheduled 22:00 SGT, must complete by 06:45 SGT. Produces MT103 -> SWIFT
gateway and FAST/MEPS+ -> MAS-adjacent gateway. recon_eod waits on the
'swift_ack' task via ExternalTaskSensor.
"""
from datetime import datetime, timedelta
from airflow.sdk import DAG
from airflow.decorators import task
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.standard.operators.python import PythonOperator
from airflow.providers.standard.sensors.filesystem import FileSensor
from airflow.providers.sftp.operators.sftp import SFTPOperator
from airflow.providers.smtp.notifications.smtp import SmtpNotifier
from airflow.utils.task_group import TaskGroup
from airflow.sdk.definitions.deadline import DeadlineAlert, DeadlineReference
from common.notifiers import page_batch_ops, page_oncall
from common.secrets import get_conn_id  # resolves from Vault-backed secrets backend
BATCH_OPS = "batch-ops-oncall@cymbal.internal"
default_args = {
    "owner": "payments-batch",
    "retries": 2,
    "retry_delay": timedelta(minutes=3),
    "retry_exponential_backoff": True,
    "max_retry_delay": timedelta(minutes=20),
    "on_failure_callback": page_batch_ops,          # first line: page batch ops
}
with DAG(
    dag_id="payments_eod",
    schedule="0 22 * * *",                           # 22:00 Asia/Singapore
    start_date=datetime(2026, 1, 1),
    catchup=False,                                   # never phantom-run on deploy
    default_args=default_args,
    tags=["eod", "payments"],
    max_active_runs=1,                               # one EOD instance at a time
    deadline=DeadlineAlert(                          # quantitative SLA: 06:45 SGT
        reference=DeadlineReference.DAGRUN_LOGICAL_DATE,
        interval=timedelta(hours=8, minutes=45),     # 22:00 + 8h45m = 06:45
        callback=SmtpNotifier(
            to=BATCH_OPS,
            subject="DEADLINE: {{ dag_run.dag_id }} past 06:45 SGT",
            html_content="EOD payments chain exceeded its 06:45 SGT deadline; on-call must intervene.",
        ),
    ),
) as dag:
    # ---- intake -----------------------------------------------------------
    extract = BashOperator(task_id="extract", bash_command="extract_payments.sh")
    validate = PythonOperator(task_id="validate", python_callable=validate_payments)
    # ---- SWIFT leg --------------------------------------------------------
    with TaskGroup("swift_leg") as swift_leg:
        swift_gen = BashOperator(task_id="generate_mt103", bash_command="gen_mt103.sh")
        swift_send = SFTPOperator(
            task_id="send_mt103",
            ssh_conn_id=get_conn_id("swift_gateway"),        # from secrets backend
            local_filepath="/data/out/MT103_{{ ds_nodash }}.txt",
            remote_filepath="/inbox/MT103_{{ ds_nodash }}.txt",
        )
        swift_ack = FileSensor(                               # deferrable: frees the slot
            task_id="wait_ack",
            filepath="/data/inbox/SWIFT_ACK_{{ ds_nodash }}.txt",
            fs_conn_id="file_gateway",
            poke_interval=30,
            timeout=timedelta(hours=6).total_seconds(),
            deferrable=True,
        )
        swift_gen >> swift_send >> swift_ack
    # ---- MEPS+/FAST leg --------------------------------------------------
    with TaskGroup("meps_leg") as meps_leg:
        fast_build = BashOperator(task_id="build_fast_file", bash_command="build_fast.sh")
        fast_submit = BashOperator(
            task_id="submit_fast",
            bash_command="submit_fast.sh {{ ds }}",
            pool="gateway_slots",                              # cap gateway concurrency
        )
        fast_build >> fast_submit
    # ---- mainframe boundary (z/OS stays on the mainframe) ----------------
    submit_zos = BashOperator(task_id="submit_zos_jobs", bash_command="zos_submit.sh {{ ds }}")
    zos_done = FileSensor(
        task_id="wait_zos_done",
        filepath="/data/inbox/ZOS_DONE_{{ ds_nodash }}.txt",
        fs_conn_id="file_gateway",
        poke_interval=60,
        timeout=timedelta(hours=5).total_seconds(),
        deferrable=True,
    )
    submit_zos >> zos_done
    # ---- compensation: if ANY leg fails, page on-call ---------------------
    escalate = PythonOperator(
        task_id="escalate",
        python_callable=page_oncall,
        trigger_rule="one_failed",
    )
    # ---- edges -------------------------------------------------------------
    extract >> validate >> [swift_leg, meps_leg, submit_zos]
    [swift_leg, meps_leg, zos_done] >> escalate
    # cross-DAG: recon_eod's ExternalTaskSensor waits on swift_leg.wait_ack
```

### Design Notes

| Concern | Design decision | Why |
|---|---|---|
| Schedule & concurrency | `0 22 * * *`, `catchup=False`, `max_active_runs=1` | Deterministic EOD trigger; no phantom runs; no overlapping instances |
| Retries | 2 retries, exponential backoff, 3 min → 20 min cap; boundary tasks keep `retries=1` conservatism | Absorbs transient gateway hiccups; never auto-replay a live SWIFT submission |
| Deadline | `DeadlineAlert` @ 06:45 SGT (logical-date reference) | Fires *at* 06:45 even mid-run — the actionable page (Section 8) |
| Alerting | `on_failure_callback` (batch ops) + `one_failed` escalate task (on-call) + deadline callback | Three escalating lines, matching the ops runbook |
| Waiting & parallelism | Deferrable sensors for file acks; pools on gateway-bound tasks | Frees worker/pod slots during multi-hour waits; caps MFT concurrency |
| Secrets | All `conn_id`s resolved from the Vault-backed secrets backend via `get_conn_id()` | No credentials in code or Variables (Section 10) |
| Mainframe & cross-DAG | SSH/z/OSMF submit + sentinel-file sensor; `recon_eod`'s `ExternalTaskSensor` targets `swift_leg.wait_ack` | One integration point, thick wall, timeouts; documented, unit-tested edge |

### The Sibling Relationship

The migration guide's Section 8 covers the *wave plan* (reporting → recon → payments/posting → regulatory), shadow-mode reconciliation, and the risk register; this DAG is the *shape of the end state* for one flow. `posting_eod`, `recon_eod`, `reporting_eod`, and `regulatory_eod` follow the identical skeleton — task groups per leg, deadline alerts per SLA, pools at boundaries, deferrable waits, and the same alerting ladder — so the platform pattern is one, and the migration only ever translates business logic, never platform mechanics.

## 13. Claims Audit and What Could Not Be Verified

### Claims Audit

Every factual claim of consequence in this guide is audited below. Status legend: ✅ = verified at a primary or directly fetched source; ⚠ = plausible and widely documented but not directly verified at a primary source during this research pass; ❌ = contradicted or failed verification.

| Claim | Status | Source |
|---|---|---|
| Airflow started at Airbnb in October 2014 | ✅ | [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow) |
| Open-sourced 2015; first release 1.0.0 on June 3, 2015; Beauchemin's "Airflow: a workflow management platform" post June 2, 2015 | ✅ | Wikipedia + [github.com/apache/airflow/releases/tag/1.0.0](https://github.com/apache/airflow/releases/tag/1.0.0) |
| Apache Incubator entry March 2016 | ✅ | [airflow.apache.org/announcements](https://airflow.apache.org/announcements/) (March 19, 2016: "we applied for Airflow's entry to the Apache Incubator") + Wikipedia |
| Top-level Apache project January 2019 | ✅ | Wikipedia |
| Airflow 2.0 released December 17, 2020 | ✅ | [Announcements](https://airflow.apache.org/announcements/) + 2.0 blog ("Thu, Dec 17, 2020") |
| Airflow 2.0: TaskFlow API (AIP-31), stable REST API (AIP-32), scheduler HA/performance (AIP-15), Task Groups (AIP-34) | ✅ | [Airflow 2.0 announcement blog](https://airflow.apache.org/blog/airflow-two-point-oh-is-here/) |
| Deferrable operators and the triggerer introduced in Airflow 2.2 (AIP-40); timetables (AIP-39) | ✅ | [Airflow 2.2 blog](https://airflow.apache.org/blog/airflow-2.2.0/) (Oct 11, 2021) + deferring docs available since the 2.2.x docs set |
| Airflow 3.0 released April 22, 2025 | ✅ | [Announcements](https://airflow.apache.org/announcements/) + [3.0 blog](https://airflow.apache.org/blog/airflow-three-point-oh-is-here/) |
| Airflow 3.0: DAG versioning (AIP-65/66), scheduler-managed backfills (AIP-78), Task Execution Interface/Task SDK (AIP-72), event-driven scheduling (AIP-82), Data Assets (AIP-74/75), React/FastAPI UI (AIP-38/84), FAB → provider (AIP-79), airflowctl (AIP-81), Edge executor (AIP-69) | ✅ | Airflow 3.0 blog |
| 3.0 usage stats: 30M+ monthly downloads, 80,000 organizations, 30%+ MLOps, ~10% GenAI | ✅ | Airflow 3.0 blog |
| Current stable line 3.3.x; 3.3.1 released August 12, 2026; 3.3.0 July 6, 2026; 2.11.2 released March 14, 2026 | ✅ | [Announcements](https://airflow.apache.org/announcements/) (fetched directly) |
| Airflow 3.3.0: Task/Asset State Store (AIP-103), Language Task SDK (Java/Go, AIP-108), pluggable retry policies | ✅ | [3.3.0 blog](https://airflow.apache.org/blog/airflow-3.3.0/) (linked from 3.0 blog "Read also") |
| Written in Python; Apache License 2.0; original author Maxime Beauchemin | ✅ | Wikipedia |
| Metadata DB: PostgreSQL 13–17, MySQL 8.0/8.4/Innovation, SQLite 3.15+ (dev only) | ✅ | [Set up a Database Backend](https://airflow.apache.org/docs/apache-airflow/stable/howto/set-up-database.html) |
| Executor set in 3.3.1: LocalExecutor (default, runs inside the scheduler process), CeleryExecutor + KubernetesExecutor (provider-packaged), BatchExecutor/ECS (amazon), EdgeExecutor (edge3); SequentialExecutor removed and hybrid LocalKubernetes/CeleryKubernetes executors unsupported from 3.0.0; multi-executor config since 2.10 | ✅ | [Executor docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html) |
| Celery broker is Redis or RabbitMQ | ✅ | Celery provider docs (referenced from executor docs) |
| Trigger rules: 11 current rules incl. `all_done_setup_success` and `all_done_min_one_success`; original eight retained | ✅ | [DAGs docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html) |
| SLA feature removed in 3.0; replaced by Deadline Alerts in 3.1 (AIP-86) | ✅ | [AIP-86 (cwiki)](https://cwiki.apache.org/confluence/spaces/AIRFLOW/pages/323488182/AIP-86+Deadline+Alerts+Formerly+SLA) + [Migrating from SLA to Deadline Alerts](https://airflow.apache.org/docs/apache-airflow/stable/howto/sla-to-deadlines.html) |
| XComs are for small values; cleared on retry; object-storage backend available | ✅ | [XComs docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/xcoms.html) |
| XCom ~48 KB limit on SQLite | ⚠ | Historical guidance; the current docs no longer quote a size figure — flag per instructions |
| `default_pool` has 128 slots; `pool_slots` supported | ✅ | [Pools docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/pools.html) |
| Secrets backends: local filesystem core backend; community backends (AWS SM/SSM, GCP SM, Vault, Azure KV); custom backends; Fernet at-rest encryption | ✅ | [Secrets Backend docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/security/secrets/secrets-backend/index.html) + [Fernet page](https://airflow.apache.org/docs/apache-airflow/stable/security/secrets/fernet.html) |
| Audit logs: dedicated audit-log surface (user actions, task-instance events, CLI events, custom events), queryable `log` table | ✅ | [Audit Logs in Airflow](https://airflow.apache.org/docs/apache-airflow/stable/security/audit_logs.html) |
| Metrics via StatsD and OpenTelemetry; traces configurable; OpenLineage metrics | ✅ | [Metrics Configuration (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/logging-monitoring/metrics.html) |
| Callbacks: on_execute/on_success/on_failure/on_retry; Notifiers (SmtpNotifier etc.); Deadline Alert callbacks | ✅ | [Callbacks docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/logging-monitoring/callbacks.html) |
| `dag.test()` / `dag.test(use_executor=True)`; pytest `conf_vars` fixture | ✅ | DAGs docs (3.3.1) |
| DAG bundles replace the DAG folder in 3.x; GitDagBundle; bundle versioning | ✅ | [Dag Bundles docs (3.3.1)](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/dag-bundles.html) |
| Community maintains more than 80 providers | ✅ | [Providers docs](https://airflow.apache.org/docs/apache-airflow-providers/) |
| Amazon MWAA launched November 2020 (announced Nov 24, 2020) | ✅ | Wikipedia citing [AWS blog](https://aws.amazon.com/blogs/aws/introducing-amazon-managed-workflows-for-apache-airflow-mwaa/) |
| Cloud Composer announced May 2018; now "Managed Service for Apache Airflow" | ✅ | Wikipedia citing [TechCrunch](https://techcrunch.com/2018/05/01/google-launches-cloud-composer-a-new-workflow-automation-tool-for-developers/) + cloud.google.com/composer |
| Astronomer founded 2015; Astro Runtime is its maintained Airflow image | ✅ | Multiple independent profiles + [astronomer.io](https://www.astronomer.io/) |
| Microsoft "Apache Airflow Job" SaaS on Fabric | ⚠ | Wikipedia managed-providers section only; not otherwise verified this pass |
| KubernetesPodOperator ships in apache-airflow-providers-cncf-kubernetes | ✅ | cncf-kubernetes provider docs (referenced from executor docs, v10.21.0) |

### What Could Not Be Verified

The following items could not be confirmed at a primary source during this research pass, and are therefore flagged rather than asserted as verified facts:

- **The 48 KB XCom size limit.** The historical figure (a SQLite column-type artifact from early docs) is no longer quoted by the current documentation, which now advises "small amounts of data" and points to the object-storage XCom backend for larger values. Treat 48 KB as legacy folklore; the design rule (kilobytes, not megabytes) is what the docs actually enforce.
- **Microsoft "Apache Airflow Job" details** (GA date, feature set, pricing). Only a Wikipedia mention was found; no vendor page was verified this pass.
- **Exact GA dates for managed-service Airflow 3.x support.** The migration guide's comparison table notes vendor 3.x availability as of 2026; specific vendor GA dates were not verified here.
- **Airflow 2.x stable-line end-of-support date.** The 2.11.2 release (March 14, 2026) is verified; the project's formal sunset date for the 2.x line is not stated in the fetched sources.
- **Cymbal Bank's estate numbers** (job counts, deadlines, DAG designs) are a worked example for this guide, not verified external facts.

---

## 14. Glossary and References

### Glossary

| Term | Definition |
|---|---|
| DAG | Directed acyclic graph — a workflow declared in Python with tasks and dependency edges. |
| Task / Task Instance | A unit of work in a DAG / one scheduled execution of that task. |
| Operator | Class defining what a task does (Bash, Python, SFTP, KubernetesPod, etc.). |
| Sensor | Operator that waits for a condition (file, time, external task); can be deferrable. |
| DAG Run | An instance of a DAG for a given schedule interval or manual trigger. |
| Scheduler | Process that parses DAGs, creates runs, and schedules task instances to an executor. |
| DAG Processor | The parsing pipeline (part of the scheduler's world) that turns DAG files/bundles into serialized DAGs. |
| Executor | Component that runs tasks (Local, Celery, Kubernetes, plus provider executors). |
| Triggerer | Lightweight process running async triggers for deferrable operators (since 2.2). |
| Metadata Database | Relational store (PostgreSQL/MySQL) of runs, task instances, variables, connections, XComs, audit logs. |
| XCom | Small key-value store for passing data between tasks. |
| Trigger Rule | Rule deciding when a task may run based on upstream states (11 rules in 3.x). |
| Catchup / Backfill | Mechanisms for running a DAG over past schedule intervals; backfills are scheduler-managed in 3.x. |
| Timetable | Pluggable scheduling logic beyond cron (2.2+, AIP-39). |
| TaskGroup | Logical grouping of tasks within a DAG (2.0+, AIP-34). |
| Pool | Named concurrency limit on task instances (`default_pool` = 128 slots). |
| Variable / Param | Stored key-value configuration / run-scoped DAG-run parameter. |
| Connection | Named credential bundle referenced by `conn_id`, resolved via secrets backends. |
| Provider | Independently versioned package (`apache-airflow-providers-*`) adding operators, hooks, sensors, executors, secret backends. |

### Primary Sources

**Apache Airflow**

- Project site: [airflow.apache.org](https://airflow.apache.org/)
- Release and project announcements (2.0.0, 2.2.0, 3.0.0, 3.3.x, 2.11.x): [airflow.apache.org/announcements](https://airflow.apache.org/announcements/)
- Release blogs (2.0, 2.2, 3.0, 3.3.0): [airflow.apache.org/blog](https://airflow.apache.org/blog/)
- Core documentation (3.3.1): DAGs, executors, XComs, pools, deferring, callbacks, metrics, secrets, audit logs, dag bundles, databases: [airflow.apache.org/docs/apache-airflow/stable/](https://airflow.apache.org/docs/apache-airflow/stable/)
- Migrating from SLA to Deadline Alerts: [airflow.apache.org/docs/apache-airflow/stable/howto/sla-to-deadlines.html](https://airflow.apache.org/docs/apache-airflow/stable/howto/sla-to-deadlines.html)
- Providers documentation: [airflow.apache.org/docs/apache-airflow-providers/](https://airflow.apache.org/docs/apache-airflow-providers/)
- AIP-86 (Deadline Alerts, formerly SLA): [cwiki.apache.org/confluence/spaces/AIRFLOW/pages/323488182/](https://cwiki.apache.org/confluence/spaces/AIRFLOW/pages/323488182/AIP-86+Deadline+Alerts+Formerly+SLA)
- Repository: [github.com/apache/airflow](https://github.com/apache/airflow)
- [Wikipedia: Apache Airflow](https://en.wikipedia.org/wiki/Apache_Airflow)

**Managed services**

- Amazon MWAA: [aws.amazon.com/mwaa](https://aws.amazon.com/mwaa/); launch announcement: [aws.amazon.com/blogs/aws/introducing-amazon-managed-workflows-for-apache-airflow-mwaa/](https://aws.amazon.com/blogs/aws/introducing-amazon-managed-workflows-for-apache-airflow-mwaa/) (Nov 24, 2020)
- Google Cloud Managed Service for Apache Airflow (formerly Cloud Composer): [cloud.google.com/composer](https://cloud.google.com/composer); launch coverage: [TechCrunch, May 1, 2018](https://techcrunch.com/2018/05/01/google-launches-cloud-composer-a-new-workflow-automation-tool-for-developers/)
- Astronomer: [astronomer.io](https://www.astronomer.io/); documentation: [docs.astronomer.io](https://docs.astronomer.io/)

### Related Guides in This Repository

- [Control-M to Airflow Migration Guide](control_m_to_airflow_migration_guide.md) — the migration playbook; this guide's platform companion (semantic mapping, waves, reconciliation).
- [Open-Source Workload Automation Alternatives Guide](open_source_workload_automation_alternatives_guide.md) — the WLA landscape and target selection; this guide's ecosystem companion.
- [AI Platform Engineering Guide](ai_platform_engineering_guide.md) — CI/CD, environments, observability, and platform practices applied to DAG development and deployment.
- [Data Pipeline Guide](data/data_pipeline_guide.md) — pipeline patterns executed by Airflow DAGs.
- [Backfill Data Engineering](data/backfill_data_engineering.md) — replay semantics operationalized by Airflow 3 scheduler-managed backfills.
- [Data Lineage Tools](data/data_lineage_tools.md) — lineage platform consuming Airflow's OpenLineage events.
- [DataOps Guide](data/dataops_guide.md) — operating model (monitoring, quality gates, runbooks) for DAG operations.
- [MAS Regulations Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) — compliance, audit, and retention requirements for regulated batch processing.
- [Cybersecurity Guide](cybersecurity_guide.md) — secrets management, access control, and data-residency standards.

### Closing

Apache Airflow has grown from an Airbnb internal tool into the de facto standard for data orchestration — and, for Cymbal Bank, into the home of the EOD batch. The platform's real product is not the scheduler loop or the executor fleet; it is the discipline those mechanisms enforce: workflows as reviewable code, deadlines as first-class alerting, waits that cost nothing, failures that page the right human, and an audit trail that regulators can follow from a MAS return back to the exact DAG version that produced it. The migration guides in this repository explain how to get there; this guide explains what "there" is — a platform where every night's batch is declared, tested, versioned, and observed, so that the 07:00 SGT deadline is met not by heroics but by design. That is the promise of Airflow: not merely that the batch runs, but that the whole organization can see, trust, and improve how the day is orchestrated — the orchestrated day.
