# Interest Background Jobs in Apache Fineract: A Comprehensive Guide

> **A comprehensive guide to the scheduler/batch machinery that computes and posts interest in Apache Fineract — the loan interest jobs, the savings interest posting, the accrual transactions, the scheduler configuration, the job internals, and the operational reality. Verified against the Apache Fineract source (develop branch, August 2026) and the official documentation.**

**Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB, Singapore
**Path:** `banking/fineract_interest_background_jobs_guide.md` | **Version:** 1.0 — August 2026
**Classification:** Technology Research — Core Banking Engineering

**Companion guides:** [Apache Fineract](apache_fineract_guide.md) (the platform umbrella) · [Interest Engines in Core Banking](interest_engines_core_banking_guide.md) (the theory + vendor comparison) · [Posting Engine](posting_engine_core_banking_guide.md) (the double-entry machinery) · [Core Banking Processes](core_banking_processes_guide.md) (the EOD/batch pipeline) · [jBASE & UniVerse](../technology/jbase_universe_guide.md)

---

## Table of Contents

1. [The Fineract Scheduler Overview](#1-the-fineract-scheduler-overview)
2. [The Job Catalog: Interest-Related Jobs](#2-the-job-catalog-interest-related-jobs)
3. [The Loan Interest Jobs](#3-the-loan-interest-jobs)
4. [The Savings Interest Jobs](#4-the-savings-interest-jobs)
5. [The Accrual and Accounting Jobs](#5-the-accrual-and-accounting-jobs)
6. [The Scheduler Configuration](#6-the-scheduler-configuration)
7. [The Job Internals: Interest Logic and Execution](#7-the-job-internals-interest-logic-and-execution)
8. [The Scheduler API](#8-the-scheduler-api)
9. [The Worked Examples](#9-the-worked-examples)
10. [Troubleshooting and Best Practices](#10-troubleshooting-and-best-practices)
11. [The Future (2026+)](#11-the-future-2026)
12. [Glossary](#12-glossary)
13. [References and Verification Notes](#13-references-and-verification-notes)

---

## 1. The Fineract Scheduler Overview

### 1.1 What the Scheduler Is

Apache Fineract is the open-source core-banking platform for financial inclusion (see [Apache Fineract](apache_fineract_guide.md)). A large part of its day-to-day banking behaviour — especially **interest** — does not happen in real time at transaction time. Instead, it happens in **background jobs**: scheduled, batch-style processes that compute interest, post accruals, update balances, and generate accounting entries.

The Fineract scheduler is the machinery that runs these jobs:

- **Batch execution** is implemented with **Spring Batch** — each job is a Spring Batch job with steps, chunk-oriented processing, and per-chunk transactions.
- **Scheduling** (when jobs run) is handled by the **Quartz Scheduler** — cron-triggered, per-tenant schedulers.

The official Fineract documentation states it directly: *"batch jobs … implemented using Spring Batch … automatic scheduling is done by the Quartz Scheduler."*

> **Verified (August 2026, develop branch):** package `org.apache.fineract.infrastructure.jobs` contains the job framework. `JobRegisterServiceImpl` builds per-tenant Quartz schedulers (named `Scheduler{tenantId}`), using `MethodInvokingJobDetailFactoryBean` with `concurrent=false` — meaning **jobs of the same definition do not run concurrently** — and cron triggers configured in the **tenant timezone**.

### 1.2 The Scheduler Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Fineract Application                      │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐   │
│  │  Quartz Scheduler (per tenant: Scheduler{tenantId})    │   │
│  │  • cron triggers (tenant timezone)                     │   │
│  │  • manual/application triggers (API-driven)            │   │
│  │  • default 7 threads (1 reserved for scheduler groups) │   │
│  └───────────────────────┬───────────────────────────────┘   │
│                          │ fires                             │
│  ┌───────────────────────▼───────────────────────────────┐   │
│  │  JobStarter.run()                                     │   │
│  │  • MethodInvokingJobDetailFactoryBean (concurrent=false)│  │
│  └───────────────────────┬───────────────────────────────┘   │
│                          │ launches                          │
│  ┌───────────────────────▼───────────────────────────────┐   │
│  │  Spring Batch JobLauncher                            │   │
│  │  • chunk-oriented steps (read → process → write)      │   │
│  │  • per-chunk transactions                            │   │
│  │  • Spring Batch 5 state tables                       │   │
│  └───────────────────────┬───────────────────────────────┘   │
│                          │ executes                          │
│  ┌───────────────────────▼───────────────────────────────┐   │
│  │  Job steps (loan interest, savings posting, accruals…) │  │
│  └───────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

Key architectural facts (verified from source):

- **Per-tenant schedulers:** each tenant (in the multi-tenant Fineract model — see [Apache Fineract](apache_fineract_guide.md) §10) gets its own Quartz scheduler instance, so tenant A's job runs do not interfere with tenant B's.
- **No concurrent execution:** `concurrent=false` on the job detail factory means the same job will not be re-entered while a previous run is still executing — a critical safeguard against double-posting.
- **Thread pool:** the default scheduler configuration provides 7 threads, with 1 reserved for the scheduler's own group management.
- **Cron in tenant timezone:** the cron expressions are evaluated in the tenant's configured timezone — important for "end of day" jobs in global deployments.

### 1.3 The Scheduler vs the Batch

Two distinct concepts that are easy to confuse:

| Concept | What it is | Fineract implementation |
|---|---|---|
| **Scheduler** | *When* jobs run (the trigger) | Quartz — cron + manual triggers |
| **Batch** | *How* jobs execute (the machinery) | Spring Batch — steps, chunks, transactions |

The scheduler fires the trigger; the batch framework executes the job. When you see "scheduler job" in Fineract documentation, it means a Spring Batch job registered with the Quartz scheduler.

### 1.4 Scheduler Status

The scheduler as a whole can be started and stopped:

- `GET /v1/scheduler` → `{ "active": true/false }` — is the scheduler running?
- `POST /v1/scheduler?command=start` or `?command=stop` — starts/stops the scheduler (requires the `UPDATE_SCHEDULER` permission; the read permission is `READ_SCHEDULER`).

> **Operational note:** if the scheduler is stopped, **no background jobs run** — interest is not posted, accruals are not booked, loan summaries are not updated. This is the first thing to check when "interest didn't post."

---

## 2. The Job Catalog: Interest-Related Jobs

Fineract's jobs are defined in the `job` table via Liquibase changelogs (verified: `0002_initial_data.xml`, `0014`, `0015`, `0037`, `0040`, `0049`, `0053`, `0075`, `0143`, `0145`, `0201`, `0239`, `2001_add_savings_accrual_job.xml`). Each job has:

- **A display name** (e.g., "Post Interest For Savings")
- **A short name** (e.g., `SA_PINT`) — used in the API
- **A cron expression** (the default schedule)
- **A scheduler group** (ordering/priority among jobs)
- **An active flag** (some jobs ship disabled)

### 2.1 The Interest-Related Jobs Table

The following table lists the jobs most relevant to interest processing, with their exact names, short names, default crons, and groups as verified from the develop branch (August 2026):

| Display name | Short name | Default cron | Group | Active | Purpose |
|---|---|---|---|---|---|
| Post Interest For Savings | `SA_PINT` | `0 0 0 1/1 * ? *` (daily 00:00) | 1 | Yes | Posts accrued interest to savings accounts |
| Transfer Interest To Savings | — | `0 2 0 1/1 * ? *` (daily 00:02) | — | Yes | Transfers interest to savings (interest-on-interest) |
| Add Accrual Transactions | `ACC_AATR` | `0 1 0 1/1 * ? *` (daily 00:01) | 3 (prio 3) | Yes | Adds accrual transactions (loans) |
| Add Periodic Accrual Transactions | `ACC_APTR` | `0 2 0 1/1 * ? *` (daily 00:02) | 3 (prio 2) | Yes | Periodic accrual (loans, periodic basis) |
| Add Accrual Transactions For Loans With Income Posted As Transactions | `ACC_AATI` | — | 3 | Yes | Accrual variant when income is posted as transactions |
| Add Accrual Transactions For Savings | `ADD_ATFS` | — | — | **No** (inactive) | Savings accrual — ships disabled |
| Recalculate Interest For Loans | `LA_RIL` | `0 1 0 1/1 * ? *` (daily 00:01) | — | Yes | Recalculates loan interest |
| Update loan Summary | `LA_USUM` | `0 0 22 1/1 * ? *` | — | **Deleted** | ⚠️ Removed in `0014_remove_unused_jobs.xml` — loan summary is updated in real time now |
| Apply Holidays To Loans | `LA_AHOL` | `0 0 12 * * ?` (daily noon) | — | Yes | Applies holiday calendars to loan schedules |
| Loan COB | `LA_ECOB` | `0 0 0 * * ?` (daily midnight) | — | No (inactive by default) | End-of-day loan processing — partitionable |
| Loan Delinquency Classification | `LA_DELQ` | `0 0 22 1/1 * ? *` | — | Yes | Classifies loan delinquency |
| Accrual Activity Posting | — | `0 0 1 * * ?` (01:00) | — | No (inactive) | Posting of accrual activity |
| Journal Entry Aggregation | `JRNL_AGG` | — | — | — | Aggregates journal entries |
| Retained Earning Job | `RE_ERNG` | — | — | — | Retained-earnings processing |

> **⚠️ Version-sensitivity warning:** job names, short names, and default crons **change between versions**. The table above is verified against the develop branch as of August 2026 (and the 1.13.0 docs). Always verify against the current source and your deployed version before relying on a specific job name or schedule.

### 2.2 The Three Interest Groups

The jobs cluster into three interest-processing concerns:

1. **Loan interest** — schedule generation, recalculation, accrual (income-side), delinquency
2. **Savings interest** — posting, transfer, accrual (expense-side)
3. **General accrual/accounting** — periodic accruals, journal aggregation, retained earnings

---

## 3. The Loan Interest Jobs

### 3.1 How Loan Interest Works in Fineract

Fineract's lending model is the **microfinance declining-balance model** (see [Apache Fineract](apache_fineract_guide.md) §5): interest is computed on the outstanding principal according to a repayment schedule (EMI-style), with configurable:

- **Interest rate** (per annum, per period)
- **Repayment frequency** (daily, weekly, monthly, etc.)
- **Interest calculation period** — `DAILY` or `SAME_AS_REPAYMENT_PERIOD` (verified enum)
- **Day-count conventions** — days-in-year: `ACTUAL` / `DAYS_360` / `DAYS_364` / `DAYS_365`; days-in-month: `ACTUAL` / `DAYS_30` (verified enums)

The **loan schedule** (the EMI table) is generated at loan disbursement and defines exactly how much interest accrues in each period. The background jobs then *recognise* that interest in the books over time.

### 3.2 The Loan Interest Jobs in Detail

| Job | What it does | When it runs (default) |
|---|---|---|
| **Recalculate Interest For Loans** (`LA_RIL`) | Recomputes loan interest — used when a loan's terms change (rescheduling, interest-rate change, waivers) | Daily 00:01 |
| **Add Accrual Transactions** (`ACC_AATR`) | Books the periodic interest accrual for loans — DR interest receivable, CR interest income | Daily 00:01 |
| **Add Periodic Accrual Transactions** (`ACC_APTR`) | The periodic (per-accounting-period) variant of the accrual | Daily 00:02 |
| **Add Accrual Transactions For Loans With Income Posted As Transactions** (`ACC_AATI`) | Accrual variant for the accounting configuration where income is posted as transactions rather than accrued | Daily |
| **Apply Holidays To Loans** (`LA_AHOL`) | Shifts loan schedules when a holiday calendar applies | Daily noon |
| **Loan COB** (`LA_ECOB`) | The end-of-day loan processor — partitionable, configurable business steps (see §3.3) | Inactive by default |
| **Loan Delinquency Classification** (`LA_DELQ`) | Classifies loans into delinquency buckets | Daily 22:00 |
| ~~Update loan Summary~~ (`LA_USUM`) | ~~Updated loan balances~~ — **removed**: loan summary is now maintained in real time | — |

### 3.3 The Loan COB Job (End of Day)

The **Loan COB** job is the modern end-of-day processor for loans:

- **Partitionable:** the job splits loan processing across partitions (and supports remote partitioning via Kafka/JMS).
- **Configurable business steps:** the sequence of processing steps is stored in `m_batch_business_steps` and can be configured via `GET/PUT /v1/jobs/{jobName}/steps`.
- **Inline trigger:** `POST /v1/jobs/LOAN_COB/inline` triggers an inline run.
- **Loan locking:** loans are soft-/hard-locked during processing (`m_loan_account_locks`); concurrent attempts return HTTP 409.
- Ships **inactive by default** — banks enable it as part of their EOD design.

> **Architect's note:** the Loan COB's step-configuration and partitioning model is the template for how modern Fineract deployments design their EOD — see [Core Banking Processes](core_banking_processes_guide.md) for the EOD pipeline context.

---

## 4. The Savings Interest Jobs

### 4.1 How Savings Interest Works in Fineract

Savings interest is configured per **savings product** (verified enums from `SavingsAccountInterestPostingServiceImpl` and the interest enums):

- **Interest rate** (per annum)
- **Interest type (calculation method):** `DAILY_BALANCE` or `AVERAGE_DAILY_BALANCE` (verified)
- **Compounding:** `DAILY`, `WEEKLY`, `MONTHLY`, `QUARTERLY`, `BI_ANNUALLY`, `ANNUAL`, or `NO_COMPOUNDING` (verified enum)
- **Posting period & frequency:** how often interest is posted to the account (monthly is typical; daily is possible)
- **Days-in-year:** savings supports only `DAYS_360` / `DAYS_365` (verified — note the difference from loans, which also support `ACTUAL` and `DAYS_364`)

### 4.2 The Savings Interest Jobs in Detail

| Job | What it does | When it runs (default) |
|---|---|---|
| **Post Interest For Savings** (`SA_PINT`) | Posts the accrued interest to savings accounts — the principal interest-posting job | Daily 00:00 |
| **Transfer Interest To Savings** | Handles interest-on-interest (compounding) transfers | Daily 00:02 |
| **Add Accrual Transactions For Savings** (`ADD_ATFS`) | Books savings interest accrual — **ships inactive** (adds the accrual transactions for savings) | — (inactive) |

### 4.3 The Posting Mechanics

The `SA_PINT` job (verified in `SavingsAccountInterestPostingServiceImpl`):

1. **Reads** the savings accounts due for interest posting (based on their product's posting period/frequency).
2. **Computes** the interest due for the posting period, using the product's interest type (`DAILY_BALANCE` / `AVERAGE_DAILY_BALANCE`), rate, and day-count convention.
3. **Handles compounding** — if the product compounds, the computed interest is added to the balance before the next period's calculation.
4. **Posts** the interest to the savings account (CR the savings account) with the corresponding accounting entry (DR interest expense) when the account's product has accounting enabled.
5. **Updates** the account's accrued-interest tracking so the same period is not posted twice.

> **Double-posting safeguard (verified):** the job tracks what has already been posted (per account, per period). Combined with `concurrent=false` on the Quartz trigger, the design prevents the classic "interest posted twice" failure. See §10 for the operational side of this.

---

## 5. The Accrual and Accounting Jobs

### 5.1 Accrual vs Posting — the Distinction

From [Interest Engines in Core Banking](interest_engines_core_banking_guide.md):

- **Accrual** = the *recognition* of interest income/expense over time (accounting concept — the income is earned even though cash hasn't moved).
- **Posting** = the *actual movement* — crediting the customer account (savings) or booking the receivable (loans).

Fineract separates these into different jobs so banks can choose their accounting posture:

| Job | Accounting effect |
|---|---|
| **Add Accrual Transactions** (`ACC_AATR`) | DR **interest receivable** / CR **interest income** (loans) |
| **Post Interest For Savings** (`SA_PINT`) | DR **interest expense** / CR **savings account** (savings) |

### 5.2 The Accounting Jobs

| Job | What it does |
|---|---|
| **Add Accrual Transactions** (`ACC_AATR`) | Books loan interest accruals (income recognition) |
| **Add Periodic Accrual Transactions** (`ACC_APTR`) | Periodic accrual variant — aligns accruals with the accounting period |
| **Add Accrual Transactions For Loans With Income Posted As Transactions** (`ACC_AATI`) | Accrual when the product posts income as transactions |
| **Add Accrual Transactions For Savings** (`ADD_ATFS`) | Savings-side accrual (ships inactive) |
| **Accrual Activity Posting** | Posting of accrual activity (ships inactive) |
| **Journal Entry Aggregation** (`JRNL_AGG`) | Aggregates journal entries (e.g., for reporting efficiency) |
| **Retained Earning Job** (`RE_ERNG`) | Period-end retained-earnings processing |

The accounting entries follow the double-entry machinery described in [Posting Engine](posting_engine_core_banking_guide.md) — every accrual/posting job ultimately generates balanced DR/CR entries into the general ledger.

---

## 6. The Scheduler Configuration

### 6.1 Configuring a Job

Each job in the `job` table carries:

- **`cron_expression`** — the Quartz cron (e.g., `0 0 0 1/1 * ? *` = daily at midnight)
- **`active`** — whether the job is enabled
- **`scheduler_group`** — the ordering group (jobs in lower-numbered groups run first; group 3 is the accrual group with priorities 2-3)
- **`scheduler_group_priority`** — priority *within* the group
- **`short_name`** — the API identifier

### 6.2 Cron Examples

| Expression | Meaning |
|---|---|
| `0 0 0 1/1 * ? *` | Daily at 00:00 |
| `0 1 0 1/1 * ? *` | Daily at 00:01 |
| `0 0 22 1/1 * ? *` | Daily at 22:00 |
| `0 0 12 * * ?` | Daily at noon |
| `0 0 1 * * ?` | Daily at 01:00 |

Crons are evaluated in the **tenant timezone** — the same expression means different local times for different tenants.

### 6.3 The Job Groups

Jobs are grouped so that dependent processing happens in order. The verified grouping includes:

- **Group 1** — savings interest posting (`SA_PINT`)
- **Group 3** — accrual transactions (`ACC_AATR` priority 3, `ACC_APTR` priority 2)

> **Design intent:** accruals must run *after* the transactions they accrue on, and interest posting must run *before* the downstream accounting. The group/priority mechanism encodes these dependencies. Review the group ordering whenever you change a job schedule.

### 6.4 Updating a Job

`PUT /v1/jobs/{jobId}` updates the job (cron, active flag) and triggers a `rescheduleJob` on the Quartz scheduler. This is the API-equivalent of editing the `job` table directly.

---

## 7. The Job Internals: Interest Logic and Execution

### 7.1 The Execution Path

Verified from `JobStarter.java` and `JobRegisterServiceImpl.java`:

1. **Quartz fires** the trigger (cron or manual/application trigger — `TRIGGER_TYPE_CRON` / `TRIGGER_TYPE_APPLICATION`).
2. **`JobStarter.run()`** is invoked (via `MethodInvokingJobDetailFactoryBean`).
3. The Spring Batch **`JobLauncher`** launches the batch job.
4. The job executes its **steps** — chunk-oriented (read → process → write), with **per-chunk transactions** (a failed chunk rolls back only that chunk's work).
5. Spring Batch 5 maintains the **batch state tables** (job instance, job execution, step execution) — the source of the run history shown in the API.
6. **Idempotency:** `JobParametersBuilder.getNextJobParameters` generates the next job parameters, so re-runs create new job instances rather than replaying a completed one.

### 7.2 The Interest Logic

**Loan interest** (declining balance — see [Apache Fineract](apache_fineract_guide.md) §5):

- Interest for a period = outstanding principal × periodic rate, where the periodic rate derives from the annual rate and the day-count convention.
- Day-count enums (verified): days-in-year `ACTUAL` / `DAYS_360` / `DAYS_364` / `DAYS_365`; days-in-month `ACTUAL` / `DAYS_30`.
- Calculation period: `DAILY` or `SAME_AS_REPAYMENT_PERIOD`.

**Savings interest** (balance-based):

- `DAILY_BALANCE` — interest on each day's closing balance.
- `AVERAGE_DAILY_BALANCE` — interest on the period's average daily balance.
- Day-count: `DAYS_360` / `DAYS_365` only (verified).
- Compounding: `DAILY` … `ANNUAL` / `NO_COMPOUNDING` (verified).

**Rounding (verified):** `Money` scales amounts to the currency's decimal places; `MoneyHelper` provides a configurable per-tenant `RoundingMode`. Rounding drift is a known concern — see [Interest Engines](interest_engines_core_banking_guide.md) §rounding-drift for the theory and the golden-test pattern for verification.

### 7.3 Transactionality and Errors

- **Per-chunk transactions:** a job failure mid-run leaves completed chunks committed and uncompleted chunks rolled back — the job can be re-run to finish the remainder.
- **Run history:** every run records `jobRunStartTime`, `jobRunEndTime`, `status`, `triggerType`, `jobRunErrorMessage`, `jobRunErrorLog` (verified from `SchedulerJobRunnerReadServiceImpl`).
- **Failed runs** appear in the run history with the error message — the operational starting point for any "interest didn't post" investigation.

---

## 8. The Scheduler API

### 8.1 The Jobs API (Verified from Source)

> **⚠️ Correction to the common documentation:** the widely-cited `POST /v1/jobs/{jobId}?runType=execute` is **outdated**. The verified endpoint is `POST /v1/jobs/{jobId}?command=executeJob` (returns HTTP 202, requires batch-manager mode and the `EXECUTEJOB_SCHEDULER` permission).

| Endpoint | Purpose |
|---|---|
| `GET /v1/jobs` | List all jobs (name, short name, cron, active, group) |
| `GET /v1/jobs/{jobId}` | Get one job |
| `GET /v1/jobs/short-name/{shortName}` | Get a job by short name (e.g., `SA_PINT`) |
| `GET /v1/jobs/{jobId}/runhistory?offset=&limit=` | Run history (times, status, error message, trigger type) |
| `POST /v1/jobs/{jobId}?command=executeJob` | Trigger a manual run (202; needs `EXECUTEJOB_SCHEDULER`) |
| `PUT /v1/jobs/{jobId}` | Update cron/active (triggers reschedule) |
| `GET /v1/scheduler` | Scheduler status (`{active: bool}`) |
| `POST /v1/scheduler?command=start` / `?command=stop` | Start/stop the scheduler (needs `UPDATE_SCHEDULER`) |
| `GET/PUT /v1/jobs/{jobName}/steps` | Read/configure Loan-COB business steps |
| `POST /v1/jobs/LOAN_COB/inline` | Trigger an inline Loan-COB run |

### 8.2 Permissions

- `READ_SCHEDULER` / `UPDATE_SCHEDULER` — scheduler start/stop
- `EXECUTEJOB_SCHEDULER` — manual job execution (batch-manager mode)

---

## 9. The Worked Examples

### 9.1 Worked Example 1: The Loan Interest Accrual Job

**Scenario:** a bank runs `Add Accrual Transactions` nightly at 00:01. A loan with S$100,000 outstanding at 5% p.a. (actual/365) accrues daily.

**The flow:**

1. The job reads loans with accrual due (accrual not yet booked for the period).
2. Daily interest = 100,000 × 0.05 × (1/365) ≈ **S$13.70**.
3. The job books the accrual entry (per loan, per day — or aggregated per period):
   - **DR** Interest Receivable (asset) — S$13.70
   - **CR** Interest Income (income) — S$13.70
4. The account's "accrued interest" tracking is updated so the day is not double-booked.

**The entries** follow the double-entry pattern of [Posting Engine](posting_engine_core_banking_guide.md) — balanced DR = CR, with the receivable growing over the period until repayment.

### 9.2 Worked Example 2: The Savings Interest Posting Job

**Scenario:** a savings account with a S$10,000 average daily balance, product rate 2% p.a. (365), monthly posting, no compounding.

**The flow:**

1. `SA_PINT` runs at 00:00 on the posting day.
2. Monthly interest = 10,000 × 0.02 × (30/365) ≈ **S$16.44**.
3. The job posts:
   - **CR** Savings Account — S$16.44 (the customer's balance increases)
   - **DR** Interest Expense — S$16.44
4. The accrued-interest tracking is reset for the next period.

### 9.3 Worked Example 3: The API Walkthrough

**List the jobs:**

```bash
curl -X GET "https://fineract.example.com/fineract-provider/api/v1/jobs" \
  -H "Authorization: Basic base64(user:password)"
```

**Trigger a manual savings-interest posting:**

```bash
curl -X POST "https://fineract.example.com/fineract-provider/api/v1/jobs/SA_PINT?command=executeJob" \
  -H "Authorization: Basic base64(user:password)"
# → 202 Accepted
```

**Check the run history:**

```bash
curl -X GET "https://fineract.example.com/fineract-provider/api/v1/jobs/short-name/SA_PINT/runhistory?offset=0&limit=5" \
  -H "Authorization: Basic base64(user:password)"
```

**Result** (abridged):

| jobRunStartTime | jobRunEndTime | status | triggerType | jobRunErrorMessage |
|---|---|---|---|---|
| 2026-08-13T00:00:02 | 2026-08-13T00:00:41 | SUCCESS | cron | — |
| 2026-08-12T00:00:01 | 2026-08-12T00:00:39 | SUCCESS | cron | — |
| 2026-08-11T00:00:02 | 2026-08-11T00:00:44 | SUCCESS | cron | — |

---

## 10. Troubleshooting and Best Practices

### 10.1 The Common Failure Modes

| Symptom | Likely cause | First check |
|---|---|---|
| Interest didn't post | Scheduler stopped | `GET /v1/scheduler` — is `active: true`? |
| Job shows FAILED in run history | Data issue, configuration, or a thrown exception | `jobRunErrorMessage` in the run history |
| Job never runs | Cron wrong, job inactive, tenant timezone | `GET /v1/jobs` — cron + active flag; review the timezone |
| Manual trigger rejected | Missing permission / not in batch-manager mode | `EXECUTEJOB_SCHEDULER` permission; batch-manager config |
| 409 on loan processing | Loan locked by Loan COB | The loan is being processed — wait or inspect `m_loan_account_locks` |

### 10.2 The Re-run and Idempotency

- Re-running a job is **safe by design**: completed work is tracked (per-account, per-period), and Spring Batch's `getNextJobParameters` creates a new job instance rather than replaying the old one.
- **Double-posting risk** is mitigated by (a) the tracked posting state, (b) `concurrent=false`, and (c) per-chunk transactions. The residual risk is a partially-failed chunk — always inspect the run history before re-running after a failure.

### 10.3 Best Practices

- **Schedule off-peak:** interest jobs are EOD/Early-morning work — align with the bank's EOD design (see [Core Banking Processes](core_banking_processes_guide.md)).
- **Respect the group ordering:** accruals (group 3) after transactions; interest posting (group 1) before downstream accounting.
- **Test in staging:** run the full job sequence against staging data before every release — interest jobs are the highest-regret place to find a bug.
- **Reconcile the interest:** periodically reconcile posted interest against independently computed expectations (the golden-test pattern of [Interest Engines](interest_engines_core_banking_guide.md)).
- **Monitor the run history:** alert on FAILED status; review the daily run times for drift.
- **Version-pin the job catalog:** job names/short-names/crons change between Fineract versions — record your version's catalog in your runbook.

---

## 11. The Future (2026+)

- **Fineract CN:** the cloud-native variant re-architects the platform (see [Apache Fineract](apache_fineract_guide.md) §12). Its job framework continues the Spring Batch lineage with cloud-native operationalisation — verify the CN-specific scheduler against the CN source when planning a CN deployment.
- **The Loan COB model:** partitionable, step-configurable, remotely-partitionable (Kafka/JMS) EOD processing is the direction of travel — interest work is increasingly expressed as configurable COB business steps rather than fixed jobs.
- **Event-driven processing:** the outbox/event patterns (see the event-streaming guides) are the natural complement — events for "interest posted" enable downstream real-time consumers without polling.
- **Real-time interest:** the industry trend toward real-time accrual (see [Interest Engines](interest_engines_core_banking_guide.md) §future) will gradually shrink the batch window — but the batch machinery will remain the accounting backbone for the foreseeable future.

**The trends summary:** batch-first today → configurable COB steps + events tomorrow → real-time accrual eventually. Design the integration layer so it survives all three.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Scheduler** | The machinery that decides *when* jobs run (Quartz in Fineract) |
| **Job** | A named, scheduled batch task (Spring Batch job in Fineract) |
| **Trigger** | The event that fires a job — cron or application (manual) |
| **Cron** | A time-expression (e.g., `0 0 0 1/1 * ? *` = daily at midnight) |
| **Short name** | The API identifier for a job (e.g., `SA_PINT`) |
| **Quartz** | The open-source scheduler library Fineract uses for cron triggering |
| **Spring Batch** | The batch framework Fineract uses for job execution (steps, chunks, transactions) |
| **runType / command** | The manual-trigger parameter — verified current form: `command=executeJob` |
| **runhistory** | The record of job runs (times, status, errors) |
| **jobParameters** | Spring Batch job parameters (used for idempotent re-runs) |
| **EMI** | Equated Monthly Instalment — the repayment schedule |
| **Declining balance** | Interest on the outstanding principal (the MFI model) |
| **Accrual** | Income/expense recognition over time (DR receivable / CR income) |
| **Posting** | The actual balance movement (CR the savings account, etc.) |
| **Compounding** | Adding interest to the balance so it earns interest |
| **Day-count** | The convention for the year/month length: 30/360, actual/365, etc. |
| **`DAYS_IN_YEAR`** | Fineract's day-count enum family (ACTUAL, DAYS_360, DAYS_364, DAYS_365) |
| **Rounding** | Scaling to the currency's decimal places (`Money`/`MoneyHelper`) |
| **Idempotency** | Re-run safety — running twice produces the same end state |
| **Double-posting** | The classic failure where interest is posted twice |
| **Loan COB** | The end-of-day loan processor (partitionable, step-configurable) |
| **Fineract CN** | The cloud-native variant of Fineract |

---

## 13. References and Verification Notes

**Primary sources (verified August 2026, Apache Fineract develop branch):**
- `org.apache.fineract.infrastructure.jobs` — `JobRegisterServiceImpl.java`, `JobStarter.java`, `SchedulerJobApiResource.java`, `SchedulerApiResource.java`, `SchedulerServiceConstants.java`, `JobName.java`, `SchedulerJobRunnerReadServiceImpl.java`
- Liquibase changelogs: `0002_initial_data.xml`, `0014/0015/0037/0040/0049/0053/0075/0143/0145/0201/0239`, `2001_add_savings_accrual_job.xml`
- Interest enums + `SavingsAccountInterestPostingServiceImpl.java`, `Money.java`
- Official Fineract 1.13.0 documentation (batch jobs + Quartz statement)

**Honesty flags:**
- Job names, short names, crons, and active flags are **version-sensitive** — verified against develop/1.13.0 (August 2026); always re-verify against your deployed version.
- The `POST /v1/jobs/{jobId}?runType=execute` form found in older documentation is **outdated**; the verified form is `?command=executeJob`.
- `Update loan Summary` was **removed** (changelog `0014`) — loan summaries are now maintained in real time.

*End of guide. Interest background jobs are where the accounting theory meets the batch reality — the jobs are simple in isolation, and the discipline is in the ordering, the idempotency, and the reconciliation.*
