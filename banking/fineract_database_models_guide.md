# The Database Models of Apache Fineract: The Fineract Schema Deep-Dive — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Banking / Data Architecture — the Apache Fineract relational schema: `fineract_default`, table families, `m_` naming conventions, client/loan/savings/accounting/audit tables, ER relationships, and a worked loan lifecycle
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [What This Guide Covers](#1-what-this-guide-covers)
2. [The Fineract Schema: Overview](#2-the-fineract-schema-overview)
3. [The Table Families](#3-the-table-families)
4. [The Client Tables](#4-the-client-tables)
5. [The Loan Tables](#5-the-loan-tables)
6. [The Savings Tables](#6-the-savings-tables)
7. [The Accounting Tables](#7-the-accounting-tables)
8. [The Audit Tables](#8-the-audit-tables)
9. [The Relationships: Key ER Links](#9-the-relationships-key-er-links)
10. [Worked Example: A Loan Lifecycle in the Schema](#10-worked-example-a-loan-lifecycle-in-the-schema)
11. [Summary: The Schema Behind the Microfinance Core](#11-summary-the-schema-behind-the-microfinance-core)
12. [Glossary](#12-glossary)
13. [References and Related Guides](#13-references-and-related-guides)

---

## 1. What This Guide Covers

### 1.1 The Purpose of This Guide

**Apache Fineract** is the Apache Software Foundation's open-source core banking platform for microfinance institutions, financial inclusion providers, and digital banks. Its heart — the thing every API call, every report, and every integration eventually touches — is a **relational database schema** that has grown organically from the **MIFOS** microfinance platform of the mid-2000s into a modern, Liquibase-managed core banking data model.

This guide is the **dedicated database-model deep-dive** for Fineract. The umbrella guide `apache_fineract_guide.md` covers the platform end-to-end (architecture, API, deployment) and touches the schema only in passing (~6 schema/table mentions across 984 lines). This guide takes that gap and expands it to full depth: the schema overview, the table families, the client/loan/savings/accounting/audit tables, the key entity-relationship links, and a worked example that walks a loan from origination to repayment row-by-row through the schema.

It follows the same pattern as the other data-model deep-dive in this repository, `oracle_flexcube_data_model_guide.md` (the FLEXCUBE schema: prefix grammar, module families, GL logs, maker/checker audit). Where FLEXCUBE is the commercial, closed, thousand-table monolith, Fineract is the open, inspectable, ~two-hundred-plus-table platform — which makes it the best **teaching schema** in core banking: you can read every DDL, every migration, and every constraint in the public GitHub repository.

### 1.2 How This Guide Relates to the Sibling Guides

- `apache_fineract_guide.md` — the Fineract umbrella: platform architecture (§3), modules (§4), the API walkthrough (§7), the loan lifecycle (§8), accounting integration (§9). This guide deep-dives the *schema* those sections only name.
- `oracle_flexcube_data_model_guide.md` — the FLEXCUBE data-model deep-dive; the structural twin of this guide (prefix grammar, module families, GL logs). Cross-reference for the commercial-core contrast.
- `data_models_banking_insurance_guide.md` — canonical banking data models (BIAN, IBM BDW, FSLDM); balance/position modeling and agreement lifecycle modeling, which Fineract's `_derived` balance columns and status enums instantiate.
- `core_banking_systems_guide.md` — the discipline umbrella: core data-model patterns (§7.8), double-entry ledger (§7.3), product factory (§7.4), transaction processing (§7.6).
- `interest_engines_core_banking_guide.md`, `posting_engine_core_banking_guide.md`, `core_banking_processes_guide.md` — the mechanics guides: interest calculation, posting/journalling, and core processes. The worked example in §10 shows the same mechanics as table rows.
- `banking_limits_domain_guide.md` — the limits domain; Fineract's `m_loan_arrears_aging`, delinquency buckets, and loan-limits fields are the schema-side of that domain.
- `universal_banking_model_guide.md` — the universal banking data model; Fineract maps cleanly onto the party/agreement/product/financial-transaction split.
- `temenos_data_model_guide.md` — the other open-data-model reference for contrast (T24 schema conventions).
- `../technology/oracle_database_guide.md` — the database platform layer (Fineract typically runs on MySQL/MariaDB; see §2.1).

### 1.3 A Note on Verification (Read This First)

Fineract is open source, so unlike FLEXCUBE almost every schema fact here is *in principle* checkable. This guide was verified against:

1. **The published SchemaSpy analysis of `fineract_default`** at `fineract.apache.org/docs/database/` — an official ASF-hosted snapshot of the default tenant schema (generated **Sun May 08 2022**), with per-table pages (`m_client.html`, `m_loan.html`, `m_loan_transaction.html`, `m_savings_account.html`, `acc_gl_journal_entry.html`, `m_portfolio_command_source.html`, `m_staff.html`, …) and summary counts.
2. **The Fineract source repository** (`github.com/apache/fineract`) — the schema DDL lives under `fineract-provider/src/main/resources/sql/` (legacy core DDL files plus the Liquibase `migrations/` tree, master manifest `changelog-tenant.xml`).
3. **The MIFOS heritage** — the old Mifos X core DDL (`0001a-mifosplatform-core-ddl-latest.sql`) that Fineract's schema evolved from, and the Apache project wiki's own history notes.
4. **The Fineract documentation** (`fineract.apache.org/docs/stable/`) and practitioner references (Finecko, DeepWiki) where they corroborate the above.

Where a fact is verified it is marked **verified**. Where a fact is version-dependent, community folklore, or could not be confirmed against a primary source, it is **flagged** — the guide says so explicitly. Two headline flags up front:

- **The table count is version-dependent.** The May 2022 SchemaSpy snapshot shows **222 tables, 0 views, 2,094 columns, 368 constraints** (verified). The current `develop` branch has grown considerably via hundreds of Liquibase changelog parts, so any single round number ("~400+") is at best a rough estimate for a recent release — treat exact counts as "check the installed version." See §2.2.
- **The `r_` and `c_` prefix claims are not confirmed.** `m_` (core/master) and `acc_gl_` (accounting) are verified; the reporting tables in the MIFOS lineage are actually `stretchy_report` / `stretchy_parameter`, and configuration lives in `m_configuration` / `m_global_configuration`, not a `c_*` table. See §2.3.

---

## 2. The Fineract Schema: Overview

### 2.1 The Database: MySQL, `fineract_default`, and the Tenant Registry

**Fineract is a database-per-tenant, MySQL-first system (verified).** The platform documentation and the operational guides agree on the shape:

- **Primary engine: MySQL** (the project's default and best-tested database; the Docker images ship MySQL). **MariaDB** is used interchangeably by many deployments, and **PostgreSQL** has been supported since the Liquibase migration era (the official docs describe the MariaDB/MySQL → PostgreSQL migration path). The schema DDL itself is written against MySQL dialect (engine InnoDB, `BIGINT` identity columns, `DATETIME` timestamps).
- **Two databases per installation (verified):**
  - `fineract_tenants` — the **tenant registry**. A small database holding one row per tenant with its connection details (host, port, database name, credentials) plus the tenant's timezone/locale. Fineract *always* connects to this first to resolve tenant connections (the docs: "Apache Fineract® is always connecting to this database to fetch tenant details and connection informations first"). Deployment variables are named `FINERACT_DEFAULT_TENANTDB_*`.
  - `fineract_default` — the **default tenant's schema**: the entire portfolio — offices, staff, clients, loans, savings, accounting, configuration, audit — as a single MySQL schema. This is the database the official SchemaSpy analysis documents, and it is the subject of this guide.
- **Multi-tenancy is at the database level (verified).** DeepWiki's summary of the openMF/fineract codebase confirms: each tenant gets its own isolated schema; HTTP requests are routed to the correct tenant from a request header, a tenant-specific connection is established, and tenant context is kept in a `ThreadLocal` for the request's lifetime. Schema-wise this means: **one schema per tenant, no `tenant_id` column pollution** — the same DDL, N copies.
- **Schema management is Liquibase (verified).** Since the 1.7.0 line, schema evolution is version-controlled Liquibase changelogs under `fineract-provider/src/main/resources/sql/migrations/` (master manifest `changelog-tenant.xml`, with hundreds of change-set parts — audit tables, Spring Batch structures, external-event tables, and feature migrations). The 1.6.0 release was the last Flyway-driven line.

> **Database facts — verification summary (verified):** MySQL primary / MariaDB + PostgreSQL supported; `fineract_tenants` registry + `fineract_default` tenant schema; database-per-tenant multi-tenancy; Liquibase migrations since 1.7.0. All confirmed against the ASF docs, the migration guide, and the DeepWiki codebase analysis.

### 2.2 The Schema Scale: 222 Tables (May 2022 Snapshot) and Counting

**The official SchemaSpy summary of `fineract_default` (verified — generated Sun May 08 2022):**

| Metric | Count (verified, 2022 snapshot) |
|---|---|
| Tables | **222** |
| Views | 0 |
| Columns | **2,094** |
| Constraints | 368 |
| Anomalies | 7 |
| Routines | 0 |

Notes for anyone quoting Fineract schema scale:

- **The task's "~400+" figure is not supported by the official snapshot — flag as unverified.** The published SchemaSpy analysis shows **222 tables** as of May 2022. Since then the project has added migrations continuously (loan re-financing, account transfers, external events, Spring Batch, credit bureau, tax, new charge types…), so a current release plausibly sits in the **250–400+ range depending on version and enabled features** — but no canonical current count is published. State it as: *"222 tables in the May 2022 published schema snapshot; materially more in current releases; count is version-dependent."*
- **Views: 0 (verified).** Fineract deliberately does not push reporting into database views; reporting runs through its own `stretchy_report` / `stretchy_parameter` report-definition framework, executing SQL at runtime. (Contrast: FLEXCUBE ships hundreds of `ACVW_*` views — Fineract chose the opposite route.)
- **Columns: 2,094 (verified).** The portfolio tables are wide — `m_loan` alone carries dozens of columns, including the `_derived` balance columns (see §5), because Fineract caches computed balances in the row rather than recomputing them from transactions on every read.
- **Constraints: 368 (verified)** — a healthy set of FKs and uniques, though by design **not** exhaustive: several "links" are logical rather than physical (see §9 on the polymorphic `entity_id` pattern).
- **Anomalies: 7 (verified)** — SchemaSpy's own orphan/constraint anomalies on the 2022 sample schema; a reminder that the schema is a living, partially organic artifact, not a freshly designed one.

**Scale implications (this guide's reading, flagged as interpretation):** 222+ tables is small for a core banking system (FLEXCUBE: ~2,000–3,500; T24: similar) precisely because Fineract is *narrow and deep on one product family* — microfinance lending and savings — rather than a universal bank. The schema is also *flat*: no per-module schemas, no tenant_id, no partitioning; one namespace, `m_`/`acc_gl_` prefixes doing the organizing work that schemas do in other systems.

### 2.3 Naming Conventions: The `m_` Prefix Grammar

Fineract's table names follow a readable, consistent grammar inherited from MIFOS. The verified conventions:

```
<m_|acc_gl_|ref_>_<domain>_<detail>[_<detail>...]
```

| Prefix | Meaning | Verified examples |
|---|---|---|
| `m_` | **Master / core module tables** — the "m" of MIFOS's "mifosplatform" lineage. Everything that is not accounting or pure reference data lives here. | `m_office`, `m_staff`, `m_client`, `m_group`, `m_loan`, `m_loan_transaction`, `m_savings_account`, `m_savings_account_transaction`, `m_configuration`, `m_global_configuration`, `m_appuser`, `m_code`, `m_code_value`, `m_charge`, `m_product_loan`, `m_product_savings`, `m_currency`, `m_portfolio_command_source` |
| `acc_gl_` | **Accounting / General Ledger** — the double-entry engine. | `acc_gl_account` (the chart of accounts), `acc_gl_journal_entry` (the ledger), `acc_gl_closure` (period closure), plus the mapping/rule tables `acc_product_mapping` and `acc_accounting_rule` |
| `ref_` | **Reference data** — small lookup tables used by business logic. | `ref_loan_transaction_processing_strategy` (the repayment allocation strategies: FIFO, weighted-average, etc.) |
| `stretchy_` | **Reporting framework** (MIFOS heritage) — report definitions and parameters. | `stretchy_report`, `stretchy_parameter` |

**Flag — two prefix claims from the brief could not be verified and are likely wrong:**

- **`r_` (reports): not confirmed.** The MIFOS/Fineract reporting tables are `stretchy_report` and `stretchy_parameter` (the "stretchy reports" framework), not `r_*` tables. If a deployment shows `r_*` tables, they are custom.
- **`c_` (configuration): not confirmed.** Configuration lives in **`m_configuration`** (MIFOS lineage; name/value/enabled property rows) and, in later releases, **`m_global_configuration`** (the rename/expansion). There is no `c_configuration` in the core schema.

**Content-word grammar (verified by example):** domain nouns stack left-to-right from general to specific: `m_loan` → `m_loan_transaction` → `m_loan_charge` / `m_loan_charge_paid_by` → `m_loan_collateral` → `m_loan_repayment_schedule` → `m_loan_arrears_aging` → `m_loan_recalculation_details` → `m_loan_rate`; `m_savings_account` → `m_savings_account_transaction` / `_charge` / `_interest_rate_chart` / `_interest_rate_slab`. This makes table families *discoverable by prefix scan* — one of the schema's best operational properties (`SHOW TABLES LIKE 'm_loan%'` gives you the whole loan portfolio family).

**Column conventions (verified by example):** foreign keys are `<owner>_id` (`client_id`, `office_id`, `loan_id`, `savings_account_id`); status is `status_enum` (integer enum, see §5); audit columns are `createdby_id` / `lastmodifiedby_id` → `m_appuser` and `created_date` / `lastmodified_date` (the `acc_gl_journal_entry` FKs to `m_appuser` on both columns are visible in the migration history); computed balances carry the `_derived` suffix (`principal_outstanding_derived`, `total_outstanding_derived`); and external system keys are `external_id` (with unique constraints where the integration needs them).

### 2.4 The Overview Table

| Aspect | Description (verified unless flagged) |
|---|---|
| **Platform** | Apache Fineract — open-source (Apache-2.0) core banking engine for microfinance / financial inclusion / digital banking; Java + Spring modular monolith; REST API on top of the schema |
| **Database** | MySQL (primary), MariaDB and PostgreSQL supported; two databases per install: `fineract_tenants` (registry) and `fineract_default` (the default tenant's full portfolio schema) |
| **Multi-tenancy** | Database-per-tenant: same DDL, one schema per tenant; request-header routing + ThreadLocal tenant context; no tenant_id columns |
| **Schema scale** | 222 tables, 0 views, 2,094 columns, 368 constraints, 7 anomalies (SchemaSpy snapshot, May 2022 — verified); larger in current releases — exact count version-dependent (flag) |
| **Naming** | `m_` master tables, `acc_gl_` accounting, `ref_` reference data, `stretchy_` reports; `<entity>_<detail>` stacking; `status_enum`, `*_derived`, `*_id` FK columns |
| **Schema management** | Liquibase changelogs under `fineract-provider/src/main/resources/sql/migrations/` (master `changelog-tenant.xml`); Flyway before 1.7.0 |
| **Heritage** | MIFOS (Grameen Foundation initiative; development from ~2005, in production from 2006); Mifos platform code contributed to Apache in December 2015; Fineract 1.x since (verified via ASF project wiki) |
| **Current version** | 1.15.0 (per the ASF docs site at the time of writing; the 1.14.x line preceded it) — verify against the release list when quoting (flag as date-sensitive) |
| **Character** | Normalized core with cached derived balances; no DB views (reporting via `stretchy_report`); audit via command journal rather than row triggers; double-entry enforced in application code, not DB triggers |

### 2.5 Where the DDL Lives: Reading the Schema from the Repository (verified)

Because Fineract is open source, the schema is not a black box — it is a directory tree you can read. The verified layout under `fineract-provider/src/main/resources/sql/`:

- **`migrations/`** — the modern, Liquibase-managed tree (since the 1.7.0 line). The master manifest is **`changelog-tenant.xml`**, which includes hundreds of change-set parts: core DDL parts, audit structures, Spring Batch tables (the batch job execution tables for background jobs), external-event/outbox tables, and feature migrations (credit bureau, tax, account transfers, loan re-financing, …). Each migration is a `databaseChangeLog` part with `<changeSet>` elements; the history is queryable at runtime via Liquibase's own `DATABASECHANGELOG` table (present in every Fineract schema).
- **`migrations/sample_data/`** — seed data for a working out-of-the-box instance (e.g., `barebones_db.sql`), which is why the published SchemaSpy snapshot shows small row counts (15 rows in `m_portfolio_command_source`, 0 rows in many portfolio tables — it is a *fresh sample schema*, not a production one).
- **Legacy/core DDL files** — the older consolidated DDL (MIFOS-platform-era, e.g., the `0001a-mifosplatform-core-ddl-latest.sql` lineage) that defined the `m_office`/`m_client`/`m_loan` core before Liquibase took over. Useful for seeing the *original* intent of the core tables without migration noise.

**Practical reading order for an architect (flagged as this guide's recommendation):** start with the SchemaSpy site (§1.3) for the shape, then grep the `migrations/` tree for the table you care about (`grep -r "CREATE TABLE m_loan" fineract-provider/src/main/resources/sql/migrations/`), then read the newest change-sets touching it to see how the table evolved. This is a capability no commercial core offers.

### 2.6 Fineract vs FLEXCUBE: The Two Data Models at a Glance

The sibling guide `oracle_flexcube_data_model_guide.md` documents the commercial-core extreme; Fineract is the open-core extreme. The comparison (this guide's reading of both schemas; all table names verified in their respective guides' sources):

| Dimension | Fineract (`fineract_default`) | FLEXCUBE (FCUBS) |
|---|---|---|
| Scale | ~222 tables (2022 snapshot), ~2,094 columns | ~2,000–3,500 tables (release-dependent) |
| Engine | MySQL / MariaDB / PostgreSQL, one schema per tenant | Oracle Database, one integrated schema per bank |
| Naming | `m_`/`acc_gl_`/`ref_` prefixes; `<entity>_<detail>` stacking | `[MM]T[M/B/W]_<content>` grammar (e.g., `STTM_CUSTOMER`, `ACTB_DAILY_LOG`) |
| Customer | `m_client` (one table, identifiers/addresses as satellites) | CIF module: `STTM_CUSTOMER` + multi-row sub-tables (`STTMS_CUST_PERSONAL`) |
| Ledger | Single convergence table `acc_gl_journal_entry`, linked back to portfolio transactions | Module logs: `ACTB_DAILY_LOG` / `ACTB_HISTORY_LOG`, views like `ACVW_ALL_AC_ENTRIES` |
| Audit | Centralized command journal `m_portfolio_command_source` (maker/checker as a mode) | Distributed maker/checker + `AUTH_STAT`/`REC_STAT` columns on every table |
| Reporting | No DB views; `stretchy_report` framework executes SQL at runtime | Hundreds of `ACVW_*` views over base tables |
| Views into history | `_derived` cached balances + `is_reversed` soft deletes | History logs and buffers per module |
| Best for | Teaching, microfinance/digital-bank cores, extensible open platform | Universal banking at scale, deep integration with Oracle stack |

---

## 3. The Table Families

### 3.1 Family Overview

Every table in `fineract_default` belongs to one of seven recognizable families. The families mirror the functional modules of the platform (organization, client, loan, savings, accounting, configuration, audit) and — because of the prefix grammar — are *self-identifying*: the first token of the table name tells you the family.

### 3.2 The Organization Family: `m_office`, `m_staff` (verified)

- **`m_office`** — the institution's **organizational hierarchy**. Every office is a row; offices form a tree through a self-referencing `parent_id` column: Head Office → Regional Office → Area/Branch Office → Field Office. `m_client.office_id` binds every client to a branch, which is how Fineract segments the portfolio and how reports roll up the hierarchy. Verified via the MIFOS core DDL lineage (`CREATE TABLE m_office` in `0001a-mifosplatform-core-ddl-latest.sql`) and the schema pages.
- **`m_staff`** — **employees / loan officers**. Verified columns from the schema page: `office_id` (FK to `m_office` — every staff member belongs to an office), `mobile_no` (unique), plus the usual `display_name`, `firstname`/`lastname`, `joining_date`, `is_loan_officer`, `status_enum`. The loan portfolio links to staff via `m_loan.loan_officer_id` (FK verified in the migration history as `FK_m_loan_m_staff`).
- Supporting tables in the family: `m_role`, `m_appuser` (system users), `m_permission` (the permission matrix backing the API security model), `m_organisation_currency`, `m_currency`.

### 3.3 The Client Family: `m_client` (verified)

- **`m_client`** — the **borrower / account holder master**. One row per customer. Carries the client's branch (`office_id`), loan officer (`staff_id`), names and contact data, and the client lifecycle state (`status_enum` with its own state machine: pending → active → closed, etc.). Full treatment in §4.
- Supporting tables: `m_group` (group/JLG lending — clients group into centers and groups, the microfinance model), `m_client_identifier` (multiple identifiers per client: national ID, mobile number, etc.), `m_client_address`, `m_client_contact`, `m_client_charge`, plus the **datatable** mechanism — `m_app_table`-driven custom fields where institutions attach arbitrary extra tables to `m_client` ("apptableName: m_client" per the Finecko datatables documentation).

### 3.4 The Loan Family: `m_loan` (verified)

- **`m_loan`** — the **loan account / agreement**. One row per loan, from application through closure; holds the terms snapshot (principal, rate, term, repayment frequency, processing strategy), the status state machine, and the cached `_derived` balances. Full treatment in §5.
- **`m_loan_transaction`** — the **loan transaction journal**: disbursements, repayments, waivers, write-offs, accruals, transfers. Full treatment in §5.
- Supporting tables: `m_loan_repayment_schedule` (the amortization schedule rows), `m_loan_charge` / `m_loan_charge_paid_by` (loan-level fees and penalties), `m_loan_collateral` (FK `FK_collateral_m_loan` verified), `m_loan_arrears_aging` (delinquency buckets, FK `m_loan_arrears_aging_ibfk_1` verified), `m_loan_recalculation_details`, `m_loan_rate` (rate tables), `m_product_loan` (the loan product factory definition), `m_loan_product_*` satellites (interest rate charts/slabs for variable products), `ref_loan_transaction_processing_strategy`.

### 3.5 The Savings Family: `m_savings_account` (verified)

- **`m_savings_account`** — the **savings account / agreement**, structurally a twin of `m_loan` (same status lifecycle pattern, same product-linkage pattern). Full treatment in §6.
- **`m_savings_account_transaction`** — deposits, withdrawals, interest postings, fee deductions, transfers (the GitHub issue quote in the audit section of the research trail explicitly names this table and its `created_date` anomaly — see §8).
- Supporting tables: `m_savings_account_charge` / `m_savings_account_charge_paid_by` (fee/penalty instances), `m_savings_account_interest_rate_chart` / `_slab` (tiered interest rates), `m_savings_product` (product factory), `m_deposit_account` (fixed/recurring deposits in the wider deposits family).

### 3.6 The Accounting Family: `acc_gl_*` (verified)

- **`acc_gl_account`** — the **chart of accounts** (COA): one row per GL account, organized as a tree (`parent_id`), with `gl_code`, `name`, `classification_enum` (1 asset, 2 liability, 3 equity, 4 income, 5 expense), and usage flags.
- **`acc_gl_journal_entry`** — the **general ledger**: every double-entry posting. Debit and credit legs are separate rows with `entry_type_enum`, `amount`, `transaction_date`, `entry_date`, and the FK links back to the originating portfolio transaction (`loan_transaction_id`, `savings_account_transaction_id` — FK `FK_acc_gl_journal_entry_m_savings_account_transaction` verified).
- **`acc_gl_closure`** — GL **period closure** (open/close accounting periods per office).
- **`acc_product_mapping`** — the **product-to-GL mapping** that turns every portfolio event into journal entries (the "lane one: automatic" of the Finecko analysis: *"a loan or savings transaction goes through the product GL mapping, acc_product_mapping"*).
- **`acc_accounting_rule`** — **manual/frequent posting rules** (the "lane two: manual" — *"an accounting rule, acc_accounting_rule, is applied via the Frequent Postings screen"*).
- Full treatment in §7.

### 3.7 The Configuration Family: `m_configuration` (verified by lineage)

- **`m_configuration`** (later releases: **`m_global_configuration`**) — the **platform configuration store**: name/value property rows (with an `enabled` flag) that switch platform behavior — e.g. force-pending-loan-status, maker-checker enforcement, rescheduling rules, client-naming display format. Verified by the MIFOS core-DDL lineage; the exact column set and the `m_configuration` → `m_global_configuration` rename are version-dependent (flag: check the installed version's DDL).
- Related: `m_currency`, `m_organisation_currency`, `m_external_service` (external service credentials), `m_permission` toggles.

### 3.8 The Audit Family: `m_portfolio_command_source` (verified)

- **`m_portfolio_command_source`** — the **command/audit journal**: one row per API command (create client, approve loan, post repayment, …), capturing who, what, when, and the full JSON payload, plus maker-checker state and processing result. Full treatment in §8.
- Supporting: `m_appuser` (the actors the audit references), plus the newer external-event tables (`m_external_event` / event store) added with the outbox pattern in later releases.

### 3.9 The Families Table

| Family | Prefix | Key tables (verified) | Purpose |
|---|---|---|---|
| Organization | `m_` | `m_office`, `m_staff`, `m_appuser`, `m_role`, `m_permission`, `m_currency` | The institution's structure: office hierarchy (head office → branch), employees/loan officers, users, roles/permissions, currencies |
| Client | `m_` | `m_client`, `m_group`, `m_client_identifier`, `m_client_address`, `m_client_contact` | Borrowers and their groupings; identifiers, addresses, contacts; datatable extension points |
| Loan | `m_` | `m_loan`, `m_loan_transaction`, `m_loan_repayment_schedule`, `m_loan_charge`, `m_loan_collateral`, `m_loan_arrears_aging`, `m_product_loan` | The lending portfolio: loan agreements, their terms, schedules, charges, collateral, delinquency, and the loan product factory |
| Savings | `m_` | `m_savings_account`, `m_savings_account_transaction`, `m_savings_account_charge`, `m_savings_account_interest_rate_chart`/`_slab`, `m_savings_product` | The savings portfolio: accounts, deposit/withdrawal/interest transactions, fees, tiered rates, product factory |
| Accounting | `acc_gl_` | `acc_gl_account`, `acc_gl_journal_entry`, `acc_gl_closure`, `acc_product_mapping`, `acc_accounting_rule` | Double-entry general ledger: chart of accounts, journal entries, period closure, product→GL mapping, posting rules |
| Configuration | `m_` | `m_configuration` / `m_global_configuration` | Platform behavior flags and property values (name/value/enabled); currency setup |
| Audit | `m_` | `m_portfolio_command_source`, `m_appuser` | Command audit journal: every API command with payload, maker/checker state, and result; the actors |

---

## 4. The Client Tables

### 4.1 `m_client` — Structure (verified)

`m_client` is the customer master. Its structure is documented on the schema page and in the API documentation, and its columns are the ones every integration touches first. Representative core columns (verified where marked, otherwise "check the installed version's DDL"):

| Column | Meaning | Verification |
|---|---|---|
| `id` | Surrogate PK (BIGINT identity) | verified |
| `account_no` | Human-readable client account number (unique) | verified |
| `external_id` | Integration key from external systems (unique when used) | verified |
| `office_id` | FK → `m_office` — the client's branch (reporting/segmentation anchor) | verified |
| `staff_id` | FK → `m_staff` — the client's loan officer | verified |
| `firstname`, `middlename`, `lastname` | Name parts | verified |
| `display_name` | Computed display name (format configurable via configuration) | verified |
| `mobileno`, `emailaddress` | Contact | verified |
| `status_enum` | Client state machine: 100 submitted/pending, 300 active, 400 closed, 500 rejected, 600 withdrawn (MIFOS client-status convention; exact enum values version-dependent — flag) | verified (column), flag (values) |
| `activation_date` | When the client became active | verified |
| `submittedon_date`, `submittedon_userid` | Application date and submitting user | verified |
| `closedon_date` | Closure date (death, write-off of client, exit) | verified |
| `createdby_id`, `lastmodifiedby_id`, `created_date`, `lastmodified_date` | Standard audit columns → `m_appuser` | verified (pattern) |

**Design notes (flagged as this guide's reading):** `m_client` deliberately stores the *current* state only — history lives in the command journal (§8) and in datatables. The `staff_id` link is the loan-officer assignment used by portfolio-at-risk reports; the `office_id` link is what makes every client row reportable to a branch and up the office tree. Fineract clients are **persons** in the microfinance sense (borrowers), but the same table serves digital-bank account holders — the platform is used both ways.

### 4.2 The Client Table

| Table | Purpose | Notes |
|---|---|---|
| `m_client` | Customer/borrower master: one row per client, with branch, loan officer, identity, contact, and lifecycle state | The anchor of the client family; every loan (`m_loan.client_id`) and savings account (`m_savings_account.client_id`) hangs off it |
| `m_client_identifier` | Multiple identifiers per client (national ID, passport, mobile money ID) | One-to-many; supports KYC and deduplication; each identifier has its own `document_type` |
| `m_group` | Groups, centers, and JLGs (joint liability groups) | The microfinance group-lending model: clients belong to groups; loans can be group-level (`m_loan.group_id`) |
| `m_group_client` | Membership join: which clients are in which group | The classic M:N join table |
| `m_client_address` / `m_client_contact` | Addresses and contact records | One-to-many per client |
| `m_client_charge` | Client-level charges (e.g., annual fees) | Mirrors the loan/savings charge-instance pattern |
| Datatables (e.g., custom `m_client_*_datatable` extensions) | Institution-defined extra fields ("apptableName: m_client") | The extensibility mechanism: arbitrary columns without touching core DDL — verified via Finecko datatables documentation |

---

## 5. The Loan Tables

### 5.1 `m_loan` — The Lifecycle (verified)

`m_loan` is the heart of the Fineract schema. One row per loan; the row's `status_enum` drives the lifecycle state machine, and the API enforces the legal transitions:

```
SUBMITTED (100) ──approve──▶ APPROVED (200) ──disburse──▶ ACTIVE (300) ──repay──▶ CLOSED_OBLIGATIONS_MET (400)
      │                          │                            │
      ├─reject (800)             ├─withdraw (700)             ├─write-off (500)
      └─withdraw (700)           │                            └─reschedule (600) CLOSED_RESCHEDULE_OUTSTANDING
                                 └─undo approval → 100
```

- **100 SUBMITTED_AND_PENDING_APPROVAL** — application created; terms and schedule generated.
- **200 APPROVED** — credit decision recorded (`approvedon_date`).
- **300 ACTIVE** — disbursed; money moved; interest starts running (`disbursedon_date`, `actual_disbursement_date`).
- **400 CLOSED_OBLIGATIONS_MET** — repaid in full; the happy end state (`closedon_date`).
- **500 CLOSED_WRITTEN_OFF**, **600 CLOSED_RESCHEDULE_OUTSTANDING_AMOUNT** — bad-loan exits.
- **700 WITHDRAWN_BY_CLIENT**, **800 REJECTED** — pre-disbursement exits.

The enum values (100/200/300/400/…) are the documented `LoanStatus` convention used across the API responses and the schema's `status_enum` column — **verified as the platform convention, but flag as version-dependent in detail** (later releases add states such as transfer-in-progress and partial-application variants; always check the `LoanStatus`/`LoanAccountStatus` enum of the installed version).

**The terms live on the row (verified pattern):** `principal_amount`, `approved_principal_amount`, `nominal_interest_rate_per_period` + `interest_period_frequency_enum`, `annual_nominal_interest_rate`, `number_of_repayments`, `repayment_every` + `repayment_frequency_enum`, `loan_term_*` (frequency + type), `amortization_method_enum` (equal principal vs equal installments), `interest_method_enum` (declining balance vs flat), `interest_calculated_in_period_enum`, `loan_transaction_processing_strategy_id` → `ref_loan_transaction_processing_strategy` (FK verified), `loanpurpose_cv_id` → `m_code_value` (FK `FK_m_loanpurpose_codevalue` verified), `loan_officer_id` → `m_staff` (FK `FK_m_loan_m_staff` verified), `client_id` / `group_id` → `m_client` / `m_group`, `fund_id` → `m_fund`, `currency_code` → `m_currency`. This is the **product-factory snapshot pattern**: the terms are copied onto the loan row at origination so later changes to the product never rewrite history.

**The balances live on the row too (verified pattern):** the `_derived` columns — `principal_outstanding_derived`, `interest_outstanding_derived`, `fee_charges_outstanding_derived`, `penalty_charges_outstanding_derived`, `total_outstanding_derived`, `total_expected_repayment_derived`, `total_repayment_derived`, `total_charges_due_at_disbursement_derived`, etc. Fineract recomputes and caches these on every transaction rather than deriving them in queries — a pragmatic choice that makes read paths fast and reporting trivial, at the cost of keeping the derived state consistent in application code.

### 5.2 `m_loan_transaction` — The Loan Journal (verified)

`m_loan_transaction` records every monetary event on a loan. Verified as a table; the column and enum details are the documented platform convention (flag for version-specific detail):

| Column | Meaning |
|---|---|
| `id`, `loan_id` (FK → `m_loan`), `external_id` | Identity and linkage |
| `transaction_type_enum` | 1 DISBURSEMENT, 2 REPAYMENT, 3 CONTRA, 4 WAIVE_INTEREST, 5 ACCRUAL, 6 WRITE_OFF, 7 TRANSFER, 8 CHARGE_PAYMENT (documented `LoanTransactionType` convention; version-dependent — flag) |
| `transaction_date` | The business date of the event (schedule-relevant) |
| `submitted_on_date` | When the event was posted to the system |
| `amount` | Total transaction amount |
| `principal_portion_derived`, `interest_portion_derived`, `fee_charges_portion_derived`, `penalty_charges_portion_derived` | The **allocation** of the payment across the four buckets, computed by the processing strategy |
| `is_reversed` | Soft-delete flag for reversed/undone transactions (Fineract reverses by flagging, never by deleting — audit integrity) |
| `createdby_id`, `lastmodifiedby_id`, `created_date`, `lastmodified_date` | Audit columns |

The four `_portion_derived` columns are the schema's fingerprint of the microfinance repayment model: every repayment is split into principal, interest, fees, and penalties, and the **processing strategy** (`ref_loan_transaction_processing_strategy`: FIFO, weighted average, early repayment, etc.) decides the order in which the portions are consumed. When a repayment arrives, the engine allocates the amount across the buckets, updates the `m_loan` `_derived` balances, writes the `m_loan_transaction` row, and posts the corresponding GL entries (§7).

### 5.3 The Loan Table

| Table | Purpose | Notes |
|---|---|---|
| `m_loan` | Loan agreement master: terms snapshot + lifecycle state + derived balances | The state machine (§5.1); one row per loan |
| `m_loan_transaction` | Loan monetary-event journal | Disbursements, repayments, waivers, write-offs, accruals, transfers; portion allocation per processing strategy |
| `m_loan_repayment_schedule` | The amortization schedule: one row per installment (due date, principal/interest/fee/penalty due and paid, completed flag) | Generated at origination from terms + processing strategy; recomputed on reschedule/recalculate |
| `m_loan_charge` / `m_loan_charge_paid_by` | Loan-level charge instances (fees/penalties applied to the loan) and the transactions that paid them | Charge types: disbursement fee, installment fee, overdue penalty |
| `m_loan_collateral` | Collateral items pledged against the loan | FK `FK_collateral_m_loan` (verified); links to `m_collateral` types |
| `m_loan_arrears_aging` | Delinquency: per-loan aging buckets (30/60/90 days…) with outstanding amounts | FK `m_loan_arrears_aging_ibfk_1` (verified); feeds portfolio-at-risk reports |
| `m_loan_recalculation_details` / `m_loan_rate` | Variable-rate and schedule-recalculation support | Added in the interest-rate-chart era (verified table names) |
| `m_product_loan` + `m_product_loan_*` | The loan **product factory**: templates for terms, accounting rules, charges, and rate charts | New loans copy from the product (snapshot pattern); `acc_product_mapping` links product → GL accounts |
| `ref_loan_transaction_processing_strategy` | Repayment allocation strategies | FK from `m_loan.loan_transaction_processing_strategy_id` (verified) |

### 5.4 The Loan Product Factory in the Schema

The product factory is where a bank configures what *kind* of loan can exist. In the schema (verified table names; column detail flagged as version-dependent):

- **`m_product_loan`** — one row per loan product. Carries the currency (`currency_code`), the allowed principal range (`min_principal_amount` / `max_principal_amount`), the term range (`min_*`/`max_*`/`default_*` per frequency), the default interest settings (rate, method — flat vs declining balance, amortization — equal installments vs equal principal), the default repayment frequency and number of repayments, the default processing strategy, whether the product allows arrears tolerance / variable installments / recalculation, and the accounting rule it uses (`accounting_rule` — NONE / CASH / ACCRUAL_PERIODIC / ACCRUAL_UPFRONT).
- **`m_product_loan_charge`** — the charges that every loan of this product carries (link table to `m_charge`, with the charge's timing and calculation).
- **`m_product_loan_interest_rate_chart`** / **`m_product_loan_interest_rate_slab`** — variable/floating-rate products: the rate chart and its tiered slabs (the loan twin of the savings interest-rate tables).
- **`m_product_loan_variable_installment_config`** — products that allow flexible installments (verified table name).
- **`acc_product_mapping`** — the product's accounting slots (§7), which is what makes the product *bookable*.

**The factory pattern in action (flagged as this guide's reading):** at loan origination the engine copies the product's defaults onto the `m_loan` row and materializes the schedule; afterwards the loan lives independently of the product. Changing a product affects only *new* loans — a property banks rely on, and the reason the schema stores terms on `m_loan` rather than joining to `m_product_loan` for every balance computation.

---

## 6. The Savings Tables

### 6.1 `m_savings_account` — The Structure (verified)

`m_savings_account` is the structural twin of `m_loan` — same lifecycle pattern, same product-linkage, same derived-balance caching. Verified as a table on the schema pages (with its full satellite family: `m_savings_account_charge`, `m_savings_account_charge_paid_by`, `m_savings_account_interest_rate_chart`, `m_savings_account_interest_rate_slab`).

Representative columns (verified pattern; flag for version-specific detail):

| Column | Meaning |
|---|---|
| `id`, `account_no`, `external_id`, `client_id` / `group_id` | Identity and owner linkage (verified) |
| `product_id` | FK → `m_savings_product` — the savings product factory row (verified) |
| `status_enum` | Savings lifecycle: 100 submitted, 200 approved, 300 active, 400 closed (plus withdrawn/rejected variants) — the savings state machine (flag values) |
| `submittedon_date`, `approvedon_date`, `activatedon_date`, `closedon_date` | Lifecycle timestamps (verified pattern) |
| `currency_code` | Account currency (verified) |
| `nominal_interest_rate_per_period` + `interest_period_frequency_enum` | Base rate (verified pattern) |
| `interest_calculation_type_enum` | Daily balance vs average daily balance (verified pattern) |
| `interest_calculation_days_in_year_type_enum` | 360 vs 365 day basis — the classic banking convention stored explicitly (verified pattern) |
| `min_required_opening_balance`, `lockin_period_frequency` (+ `lockin_period_frequency_enum`), `withdrawal_fee_for_transfer` | Account constraints (verified pattern) |
| `min_balance_derived`, `total_deposits_derived`, `total_withdrawals_derived`, `total_interest_earned_derived`, `account_balance_derived` | The cached balance set (verified pattern) |

**Savings-specific design points (flagged as this guide's reading):** savings interest is **tiered and chart-driven** — the `m_savings_account_interest_rate_chart` / `_slab` tables hold the per-account rate chart (or reference the product's chart), and interest is **posted** (capitalized) into the account by a background job at the configured frequency (daily/monthly/quarterly) — the schema counterpart of the mechanics in `interest_engines_core_banking_guide.md`. Interest postings are ordinary `m_savings_account_transaction` rows of type interest-posting, which keeps the balance logic uniform: *everything that moves a balance is a transaction row*.

### 6.2 The Savings Table

| Table | Purpose | Notes |
|---|---|---|
| `m_savings_account` | Savings account master: terms + lifecycle + derived balances | Twin of `m_loan`; the savings state machine |
| `m_savings_account_transaction` | Deposit, withdrawal, interest-posting, fee-deduction, transfer journal | Each row moves the balance; carries the same portion/derived discipline as loan transactions; `created_date` on this table is a known reporting shortcut (see §8 note) |
| `m_savings_account_charge` / `m_savings_account_charge_paid_by` | Account-level charges and their payment events | FK `fk_savings_account_charge_charge` (verified) |
| `m_savings_account_interest_rate_chart` / `m_savings_account_interest_rate_slab` | Per-account tiered interest charts and slabs | FK `savings_account_interest_rate_chart_id` (verified) |
| `m_savings_product` | The savings product factory | Template for rates, charges, accounting mappings |
| `m_deposit_account` (family) | Fixed deposits and recurring deposits | The wider deposits family sharing the account/transaction pattern |

### 6.3 The Wider Deposits Family

Beyond passbook-style savings, Fineract models **fixed deposits (FD)** and **recurring deposits (RD)** in the same schema family (verified family names; details version-dependent — flag):

- **`m_deposit_account`** — the fixed/recurring deposit agreement: the same status lifecycle and derived-balance pattern as `m_savings_account`, plus deposit-type specifics (maturity date, maturity amount, pre-mature-closure rules, interest compounding frequency).
- **`m_deposit_account_transaction`** — the deposit account's transactions (deposits into the FD, interest postings, premature withdrawals, maturity payout).
- **`m_deposit_product`** — the deposit product factory (the FD/RD twin of `m_savings_product` / `m_product_loan`), with its own interest rate charts and slabs and its own `acc_product_mapping` slots (e.g., deposit liability accounts, interest expense).
- **`m_deposit_account_interest_rate_chart`** / **`_slab`** — per-account tiered rate charts for deposits.

The design point: **Fineract treats savings, fixed deposits, and recurring deposits as three flavors of one "deposit account" pattern** — agreement row + transaction rows + product factory + GL mapping. Once you can read `m_savings_account`, you can read the whole deposits family; only the interest/term mechanics differ (and those are exactly the mechanics covered in `interest_engines_core_banking_guide.md`).

---

## 7. The Accounting Tables

### 7.1 The `acc_gl` Journal — The Single Ledger (verified)

Fineract's accounting is a textbook **double-entry general ledger**, and the schema makes the convergence explicit: **all three posting lanes end in one table, `acc_gl_journal_entry`**. The Finecko analysis states it directly: *"Three lanes converge on a single ledger table, acc_gl_journal_entry. Lane one, automatic: a loan or savings transaction goes through the product GL mapping, acc_product_mapping. Lane two, manual: an accounting rule, acc_accounting_rule, is applied via the Frequent Postings screen."* (Lane three is the manual journal entry API.)

- **`acc_gl_account`** — the chart of accounts (COA). Columns (verified pattern): `id`, `name`, `gl_code` (unique), `parent_id` (COA tree), `classification_enum` (1 asset, 2 liability, 3 equity, 4 income, 5 expense), `account_usage_enum` (1 detail/leaf vs 2 header/parent), `manual_entries_allowed`, `disabled`, plus audit columns.
- **`acc_gl_journal_entry`** — the ledger. Columns (verified pattern): `id`, `account_id` (FK → `acc_gl_account`), `office_id` (FK → `m_office` — entries are branch-scoped), `currency_code`, `transaction_date`, `entry_date`, `entry_type_enum` (DEBIT / CREDIT), `amount`, `reference_number`, `manual_entry` (flag), `loan_transaction_id` (FK → `m_loan_transaction`), `savings_account_transaction_id` (FK → `m_savings_account_transaction` — **FK `FK_acc_gl_journal_entry_m_savings_account_transaction` verified**), `createdby_id` / `lastmodifiedby_id` (FK → `m_appuser` — verified in the migration history).
- **`acc_gl_closure`** — GL period closure per office: `office_id`, `closing_date`, `is_deleted`, `createdby_id`. Closing a period prevents postings into it.
- **`acc_product_mapping`** — product → GL account mapping: `product_type_enum` (loan/savings/deposit), `product_id`, `financial_account_type` (the semantic slot: fund source, loan portfolio, interest income, fee income, penalty income, …), `gl_account_id`. This table is what makes portfolio events *bookable*: the engine looks up the mapping, then writes the debit/credit pair.
- **`acc_accounting_rule`** — frequent/manual posting rules: `name`, `office_id`, `account_to_debit`, `account_to_credit`, `debit_account_type`, `credit_account_type`, `system_defined`.

**The double-entry discipline is application-enforced, not database-enforced (verified by inspection — flag as interpretation).** There are no DB triggers writing `acc_gl_journal_entry`; the posting engine (see `posting_engine_core_banking_guide.md`) writes balanced pairs in a transaction. The schema records the *links back* to the originating portfolio transaction (`loan_transaction_id`, `savings_account_transaction_id`), which is what makes the ledger reconcilable to the portfolio — and is exactly the same design choice as FLEXCUBE's `ACTB_DAILY_LOG` entry-linking columns.

### 7.2 The Accounting Table

| Table | Purpose | Notes |
|---|---|---|
| `acc_gl_account` | Chart of accounts (COA tree) | Asset/liability/equity/income/expense classification; header vs detail usage |
| `acc_gl_journal_entry` | The single general-ledger journal | Every debit and credit as a row; linked back to `m_loan_transaction` / `m_savings_account_transaction` / `m_office` / `m_appuser` |
| `acc_gl_closure` | GL period closure per office | Blocks postings into closed periods |
| `acc_product_mapping` | Product → GL account mapping | The automatic-posting lane; the schema of the product factory's accounting rules |
| `acc_accounting_rule` | Manual frequent-posting rules | The manual-posting lane; used by the Frequent Postings screen |
| `acc_gl_account` self-relations / `m_code`-backed enums | COA hierarchy + entry-type/classification enumerations | Classification values backed by code tables |

### 7.3 Posting Mechanics: From Portfolio Event to Journal Rows

The mapping from a portfolio event to ledger rows is deterministic, and it is the schema's most important accounting property (verified pattern; the fine-grained behavior of each `financial_account_type` slot is version-dependent — flag):

1. **A portfolio event occurs** — e.g., the repayment transaction in §10: `INSERT m_loan_transaction` (type REPAYMENT, amount 91,000).
2. **The product's accounting rule decides the regime** — cash basis vs accrual (periodic/upfront). Cash: entries are booked when money moves. Accrual: interest is recognized over time via accrual transactions (type 5 ACCRUAL) and the repayment then *reverses* accrued interest before booking income.
3. **The engine resolves GL accounts** — via `acc_product_mapping` for the loan's product and the event's financial-account type (fund source, loan portfolio, interest income, fee income, penalty income, receivable, …).
4. **Balanced rows are written** — debit and credit legs as separate `acc_gl_journal_entry` rows in the same transaction as the portfolio update, each carrying `office_id`, `currency_code`, `transaction_date`/`entry_date`, `entry_type_enum`, `amount`, and the back-link `loan_transaction_id` (or `savings_account_transaction_id`).
5. **Reconciliation is a join** — `SELECT … FROM acc_gl_journal_entry WHERE loan_transaction_id = ?` returns the full posting for any event; summing by `account_id` and `office_id` over a period gives the GL trial balance; the `acc_gl_closure` rows define which periods are still open.

The design consequence: **the ledger never invents money**. Every debit has its credit in the same application transaction, and every journal row can be traced to a portfolio transaction that itself traces to a command in `m_portfolio_command_source`. That chain — command → transaction → journal — is the schema's answer to the auditability question regulators ask first ("show me the entries for this loan").

---

## 8. The Audit Tables

### 8.1 `m_portfolio_command_source` — The Command Audit (verified)

Fineract's audit model is **command-based, not trigger-based**. Instead of database triggers or row-version columns, every mutating API call is journaled as a *command* in **`m_portfolio_command_source`** — one row per command (CREATE CLIENT, APPROVE LOAN, POST REPAYMENT, …). This is verified directly:

- The schema page documents `m_portfolio_command_source` (the published sample schema even ships with ~15 seeded rows).
- The Mifos X source (the lineage ancestor) maps the JPA entity `CommandSource` with `@Table(name = "m_portfolio_command_source")` and a column `action_name` (nullable, length 100) — verified from the historical code reference.
- Community discussion (openMF GitHub issue #1329) confirms the design consequence: transaction tables historically did **not** carry `created_date` because "avoiding having to make a join between transaction tables and m_portfolio_command_source table" was the *reason* the anomaly (a `created_date` column on `m_savings_account_transaction`) was introduced — i.e. the command-source table is the canonical provenance store.

Representative columns (verified pattern from the MIFOS lineage; flag for version-specific detail):

| Column | Meaning |
|---|---|
| `id` | Command PK |
| `action_name` | The API action (CREATE, UPDATE, DELETE, APPROVE, DISBURSE, REPAY, …) |
| `entity_name` | The resource entity (CLIENT, LOAN, SAVINGSACCOUNT, …) |
| `entity_id` | The affected business row (`m_client.id`, `m_loan.id`, …) — **polymorphic**: no FK, points at whichever entity table the `entity_name` says |
| `product_id` | Product context when applicable |
| `office_id` | Branch context |
| `api_operation` | The HTTP method (POST/PUT/DELETE) |
| `version` | Optimistic-concurrency version of the affected row |
| `command_as_json` | The **full request payload** as JSON — the who/what/why audit trail |
| `made_on_date` / `made_on_datetime` | When the maker issued the command |
| `maker_id` | FK → `m_appuser` — the maker (verified pattern) |
| `checked_on_date` / `checker_id` | When the checker approved it (maker-checker mode) |
| `processing_result_enum` | 1 SUCCESS / 2 FAILURE — the outcome of executing the command |
| `command_serial_number` | Sequence for replay/ordering |

**How it works (verified pattern):** with **maker-checker** enabled, a command is first *made* (maker_id set, `processing_result_enum` pending) and only executed once a second user *checks* it (checker_id + checked_on_date). With maker-checker off, the command is executed immediately and the row records the outcome. Either way, **the audit is the command stream** — you can reconstruct exactly what was requested, by whom, when, and whether it succeeded. This is the MIFOS answer to the maker/checker audit columns FLEXCUBE scatters across its tables (`AUTH_STAT`, maker/checker stamps) — Fineract centralizes the same discipline into one journal.

**Related audit machinery (verified names, details flagged):** `m_appuser` (the actors — users, with their office and roles); `m_audit`-style row-change tables in some releases (version-dependent); the **external event store** (`m_external_event` and friends) added with the transactional-outbox pattern in the 1.7+ era for emitting domain events to integrations. The command journal remains the *authoritative* audit, and `m_loan_transaction.is_reversed` / soft-delete flags everywhere preserve the "never delete money" audit rule.

### 8.2 The Audit Table

| Table | Purpose | Notes |
|---|---|---|
| `m_portfolio_command_source` | Command/audit journal: every API command with actor, action, entity, JSON payload, maker/checker state, and result | The canonical provenance store; polymorphic `entity_id`; replayable command stream |
| `m_appuser` | Application users (the actors) | Referenced by `maker_id`/`checker_id` and by the `createdby_id`/`lastmodifiedby_id` audit columns on portfolio tables |
| `m_external_event` (family) | Outbox/event store for domain events (later releases) | Feeds integrations without blocking the transaction; version-dependent (flag) |
| Transaction soft-delete flags (`is_reversed` on `m_loan_transaction`, etc.) | Reversal instead of deletion | Keeps the monetary audit trail complete |
| `stretchy_report`-backed audit reports | Regulatory/user audit reporting | Reporting runs over the command journal and transaction tables |

### 8.3 Maker-Checker in the Schema

Maker-checker is a control Fineract inherits directly from the MIFOS banking-culture heritage (the same discipline FLEXCUBE encodes in `AUTH_STAT`/`REC_STAT` columns on every table). In Fineract it is a **mode**, not a column set: the platform configuration (`m_configuration` / `m_global_configuration`, property `enable-maker-checker`) plus per-permission settings decide whether commands are executed immediately or held for a second actor. The schema's side of the pattern (verified pattern; flag for version-specific column names):

- **Maker phase:** the command is journaled in `m_portfolio_command_source` with `maker_id` = the requesting user, `made_on_date`/`made_on_datetime`, the full `command_as_json` payload, and `processing_result_enum` left in the pending state. **No business tables are touched yet.**
- **Checker phase:** a second user (with the check permission for that `action_name`/`entity_name`) approves or rejects the command. On approval, the business logic executes (the `m_client`/`m_loan`/`m_loan_transaction` rows are written), and the command row is updated with `checker_id`, `checked_on_date`, and `processing_result_enum=1` (success). On rejection, the row records the failed/skipped result and nothing executes.
- **Audit payoff:** because the payload is stored as JSON on the command row, the *intent* is auditable even when execution failed — and because `processing_result_enum` distinguishes success from failure, auditors can separate "what was asked" from "what happened". Reports over the command journal are standard regulator-friendly output.

---

## 9. The Relationships: Key ER Links

### 9.1 The ER Shape in One Diagram

```
m_office (parent_id ──▶ m_office)                ┌── m_client_identifier
   │  ▲                                            │
   │  └── m_staff (office_id)                      ▼
   │         ▲                               m_client (office_id, staff_id)
   │         │ loan_officer_id                    │ │
   │         │                                    │ └─────────────┐
   │   m_loan (client_id / group_id)              │               │
   │      │  │  │  │  │                           ▼               ▼
   │      │  │  │  │  └── m_loan_repayment_schedule      m_savings_account (client_id)
   │      │  │  │  └── m_loan_charge / _paid_by             │  │  │
   │      │  │  └── m_loan_collateral                       │  │  └── m_savings_account_charge / _paid_by
   │      │  └── m_loan_arrears_aging                       │  └── m_savings_account_interest_rate_chart ──▶ _slab
   │      └── m_loan_transaction ──┐                         └── m_savings_account_transaction ──┐
   │                              │ loan_transaction_id                           │ savings_account_transaction_id
   │                              ▼                                                 ▼
   │                    acc_gl_journal_entry ◀── acc_product_mapping ◀── m_product_loan / m_savings_product
   │                              ▲    (debit/credit rows; account_id)
   │                              └── acc_gl_account (parent_id COA tree)
   │
   └── m_portfolio_command_source (maker_id/checker_id ──▶ m_appuser; entity_id ──▶ polymorphic)
```

### 9.2 The Key ER Relationships (verified)

| # | Relationship | Cardinality | Columns / FK (verified where marked) | Semantics |
|---|---|---|---|---|
| 1 | `m_office` → `m_office` (self) | 1:N | `parent_id` | The office hierarchy tree: head office → region → branch → field office |
| 2 | `m_office` → `m_staff` | 1:N | `m_staff.office_id` | Every employee belongs to exactly one office |
| 3 | `m_office` → `m_client` | 1:N | `m_client.office_id` | Every client belongs to a branch — the reporting/segmentation anchor |
| 4 | `m_staff` → `m_client` | 1:N | `m_client.staff_id` | Loan-officer assignment for the client |
| 5 | `m_staff` → `m_loan` | 1:N | `m_loan.loan_officer_id` (FK `FK_m_loan_m_staff` verified) | Loan officer responsible for the loan |
| 6 | `m_client` → `m_loan` | 1:N | `m_loan.client_id` | A client's loans (a client can have many loans over time) |
| 7 | `m_group` → `m_loan` | 1:N | `m_loan.group_id` | Group/JLG loans (microfinance group lending) |
| 8 | `m_loan` → `m_loan_transaction` | 1:N | `m_loan_transaction.loan_id` | The loan's monetary events (disbursements, repayments, …) |
| 9 | `m_loan` → `m_loan_repayment_schedule` | 1:N | `m_loan_repayment_schedule.loan_id` | The amortization installments |
| 10 | `m_loan` → `m_loan_charge` / `m_loan_collateral` / `m_loan_arrears_aging` | 1:N | `loan_id` FKs (FK `FK_collateral_m_loan`, `m_loan_arrears_aging_ibfk_1` verified) | Charges, collateral, delinquency buckets |
| 11 | `m_loan` → `ref_loan_transaction_processing_strategy` | N:1 | `m_loan.loan_transaction_processing_strategy_id` (verified) | Repayment allocation strategy |
| 12 | `m_loan` → `m_code_value` | N:1 | `m_loan.loanpurpose_cv_id` (FK `FK_m_loanpurpose_codevalue` verified) | Loan purpose from the code table (`m_code`/`m_code_value`) |
| 13 | `m_savings_account` → `m_savings_account_transaction` / `_charge` / `_interest_rate_chart` → `_slab` | 1:N | `savings_account_id` FKs (chart/slab FK verified) | The savings portfolio satellites |
| 14 | `m_loan_transaction` → `acc_gl_journal_entry` | 1:N | `acc_gl_journal_entry.loan_transaction_id` | Ledger entries linked back to the loan event that caused them |
| 15 | `m_savings_account_transaction` → `acc_gl_journal_entry` | 1:N | `acc_gl_journal_entry.savings_account_transaction_id` (FK verified) | Same provenance link for savings |
| 16 | `acc_gl_account` → `acc_gl_journal_entry` | 1:N | `acc_gl_journal_entry.account_id` | The GL account each entry posts to |
| 17 | `acc_gl_account` → `acc_gl_account` (self) | 1:N | `parent_id` | The COA tree |
| 18 | `m_product_loan` / `m_savings_product` → `acc_product_mapping` → `acc_gl_account` | 1:N / N:1 | `acc_product_mapping.product_id`, `gl_account_id` | Product → GL slot mapping (the automatic posting lane) |
| 19 | `m_appuser` → everything | 1:N (audit) | `maker_id`, `checker_id`, `createdby_id`, `lastmodifiedby_id` | Every write is attributable to a user |
| 20 | `m_portfolio_command_source` → `m_appuser`; → polymorphic `entity_id` | N:1 / logical | `maker_id`, `checker_id`, `entity_name`+`entity_id` | The command audit trail (polymorphic — no FK; flag) |

**Two relationship design lessons (flagged as this guide's reading):**

1. **The schema is deliberately "under-constrained" in places.** `m_portfolio_command_source.entity_id` is polymorphic (points at `m_client`, `m_loan`, or anything else depending on `entity_name`), and some portfolio links are enforced in application code rather than by FK. That is a pragmatic trade-off in a platform with many entity types and frequent schema evolution — but it means referential integrity is only as good as the application layer. Data architects integrating against Fineract should not assume every link is a physical FK.
2. **Office is on everything that matters.** `m_client.office_id`, `m_staff.office_id`, `acc_gl_journal_entry.office_id` — the office dimension is the universal segmentation key, exactly as `BRANCH` is in FLEXCUBE's account tables. Branch-scoped reporting is a join on `office_id` away.

### 9.3 How to Explore the Schema Yourself

For an architect who wants to verify or extend anything in this guide, the exploration kit (all verified techniques — this is how the facts here were checked):

- **The SchemaSpy site** — browse `fineract.apache.org/docs/database/`: the column listing per table, the relationships tab per table (which FKs point in and out), the orphan/anomaly lists, and the full constraint index. This is the fastest way to see the ER shape (§9.1) interactively.
- **`SHOW TABLES LIKE` / `SHOW CREATE TABLE`** — against any running Fineract instance: `SHOW TABLES LIKE 'm_loan%'` lists the whole loan family; `SHOW CREATE TABLE m_loan_transaction` shows the exact columns, indexes, and FKs of the installed version. Always check the *installed* version — column sets drift between releases.
- **`information_schema`** — `SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema='fineract_default' ORDER BY table_rows DESC` gives live row counts (useful for spotting which tables are hot in production); `information_schema.key_column_usage` lists every FK in the schema.
- **The `migrations/` tree on GitHub** — grep for `CREATE TABLE`/`ALTER TABLE` on the table of interest to see birth and evolution; the `changelog-tenant.xml` master manifest is the index of everything.
- **The runtime `DATABASECHANGELOG` table** — in any deployed schema, it tells you exactly which change-sets ran, in what order, and when — the deployment's schema-version fingerprint.

---

## 10. Worked Example: A Loan Lifecycle in the Schema

### 10.1 The Scenario: Loan Origination to Repayment

A microfinance institution on Fineract takes a borrower through the full lifecycle: **client onboarding → loan application → approval → disbursement → scheduled repayment → closure**. Below is the same story told twice: once as business steps, once as table rows (which tables get INSERTs/UPDATEs at each step, and what lands in the ledger).

Setup (assumed already in the schema):

- `m_office`: Head Office (id=1), **Branch "Kampala Central"** (id=2, `parent_id=1`).
- `m_staff`: loan officer **Amina** (id=10, `office_id=2`).
- `m_product_loan`: product **"Group Agri Loan"** (id=100) — 12 monthly installments, 18% p.a. declining balance, FIFO processing strategy (`ref_loan_transaction_processing_strategy` id=1), disbursement fee 1%.
- `acc_gl_account`: COA with at least — 10000 **Loan Portfolio** (asset), 11000 **Cash / Fund Source** (asset), 40000 **Interest Income** (income), 41000 **Fee Income** (income), 42000 **Penalty Income** (income).
- `acc_product_mapping`: product 100 → GL slots — fund source → 11000, loan portfolio → 10000, interest income → 40000, fee income → 41000.
- `m_appuser`: user **maker** (id=1000) and **checker** (id=1001) — maker-checker enabled.

### 10.2 The Table Flow (verified pattern — the tables are verified; the row-level detail is an illustrative trace)

| # | Business step | Tables touched | What happens in the schema |
|---|---|---|---|
| 1 | **Client onboarding** | `m_client`, `m_client_identifier`, `m_portfolio_command_source` | `INSERT m_client` (office_id=2, staff_id=10, names, `status_enum=100` submitted) → activate: `UPDATE m_client SET status_enum=300, activation_date=…`. Identifier row in `m_client_identifier`. Each command (CREATE, ACTIVATE) journaled in `m_portfolio_command_source` (action_name=CREATE/UPDATE, entity_name=CLIENT, entity_id=new client id, command_as_json=payload, maker_id=1000; if maker-checker, executed on checker's approval) |
| 2 | **Loan application (origination)** | `m_loan`, `m_loan_repayment_schedule`, `m_loan_charge`, `m_portfolio_command_source` | `INSERT m_loan` (client_id, product_id=100, loan_officer_id=10, status_enum=100, principal_amount=1,000,000, nominal_interest_rate_per_period=1.5, number_of_repayments=12, repayment_every=1 month, amortization_method_enum=declining, interest_method_enum=declining-balance, loan_transaction_processing_strategy_id=1, currency_code=UGX). The engine generates **12 schedule rows** in `m_loan_repayment_schedule` (due dates + principal/interest per installment). Disbursement fee 1% → `m_loan_charge` row (charge type=disbursement fee, amount=10,000). Command CREATE LOAN journaled |
| 3 | **Approval** | `m_loan`, `m_portfolio_command_source` | `UPDATE m_loan SET status_enum=200, approved_principal_amount=1,000,000, approvedon_date=…`. Command APPROVE LOAN journaled (maker→checker) |
| 4 | **Disbursement** | `m_loan`, `m_loan_transaction`, `m_loan_charge`/`_paid_by`, `acc_gl_journal_entry`, `m_portfolio_command_source` | `UPDATE m_loan SET status_enum=300, disbursedon_date=…`. `INSERT m_loan_transaction` (loan_id, `transaction_type_enum=1` DISBURSEMENT, amount=1,000,000). Disbursement fee collected → charge paid-by row + a charge-payment transaction. **Journal entries** (per `acc_product_mapping`): Debit 10000 Loan Portfolio 1,000,000 / Credit 11000 Cash 1,000,000; and Debit 11000 Cash 10,000 / Credit 41000 Fee Income 10,000 — four rows in `acc_gl_journal_entry`, each with `loan_transaction_id` pointing back to the disbursement transaction and `office_id=2`. Command DISBURSE LOAN journaled |
| 5 | **Interest accrual** (if accrual accounting) | `m_loan_transaction`, `acc_gl_journal_entry` | Periodic accrual: `m_loan_transaction` rows `transaction_type_enum=5` ACCRUAL; entries Debit 10000 Loan Portfolio / Credit 40000 Interest Income (per accrual-based accounting config) |
| 6 | **Repayment #1** | `m_loan_transaction`, `m_loan` (derived columns), `m_loan_repayment_schedule`, `acc_gl_journal_entry`, `m_portfolio_command_source` | `INSERT m_loan_transaction` (`transaction_type_enum=2` REPAYMENT, amount=91,000, transaction_date=due date). The processing strategy allocates: principal_portion_derived≈83,000 / interest_portion_derived≈8,000 (declining balance, month 1). `UPDATE m_loan` derived balances (principal_outstanding_derived≈917,000, …). Schedule row marked paid. **Journal entries**: Debit 11000 Cash 91,000 / Credit 10000 Loan Portfolio 83,000 / Credit 40000 Interest Income 8,000 — each linked via `loan_transaction_id`. Command REPAY LOAN journaled |
| 7 | **Missed payment → delinquency** | `m_loan_arrears_aging` | Overdue installment → `m_loan_arrears_aging` rows with aging buckets (30/60/90 days); feeds portfolio-at-risk reporting |
| 8 | **Final repayment → closure** | `m_loan_transaction`, `m_loan`, `m_portfolio_command_source` | Last repayment zeroes `total_outstanding_derived` → `UPDATE m_loan SET status_enum=400 CLOSED_OBLIGATIONS_MET, closedon_date=…`. Command CLOSE LOAN journaled. Final state: 13 `m_loan_transaction` rows (1 disbursement + 12 repayments), 12 schedule rows paid, 2×n journal-entry rows, one complete command trail |
| 9 | **(Alternative exit) Write-off** | `m_loan`, `m_loan_transaction`, `acc_gl_journal_entry` | If the loan goes bad: `m_loan_transaction` `transaction_type_enum=6` WRITE_OFF; entries Debit write-off-expense / Credit 10000 Loan Portfolio; `status_enum=500` |

### 10.3 The Disbursement Step in SQL Sketches

Step 4 of the flow, sketched as the SQL the schema would hold (illustrative — column names per the conventions of this guide; check the installed DDL for the exact set):

```sql
-- 1. The loan goes ACTIVE
UPDATE m_loan
   SET status_enum = 300, disbursedon_date = '2026-01-15'
 WHERE id = 5001;

-- 2. The disbursement transaction row
INSERT INTO m_loan_transaction
  (loan_id, transaction_type_enum, transaction_date, submitted_on_date,
   amount, createdby_id)
VALUES
  (5001, 1 /* DISBURSEMENT */, '2026-01-15', '2026-01-15 10:12:00',
   1000000.00, 1000);

-- 3. The ledger: balanced pair, traced back to the transaction
INSERT INTO acc_gl_journal_entry
  (account_id, office_id, currency_code, transaction_date, entry_date,
   entry_type_enum, amount, loan_transaction_id, createdby_id)
VALUES
  (10000, 2, 'UGX', '2026-01-15', '2026-01-15', 1 /* DEBIT */, 1000000.00, 501, 1000),
  (11000, 2, 'UGX', '2026-01-15', '2026-01-15', 2 /* CREDIT */, 1000000.00, 501, 1000);

-- 4. The command audit row (maker-checker on: executed after checker approves)
INSERT INTO m_portfolio_command_source
  (action_name, entity_name, entity_id, office_id, api_operation,
   command_as_json, made_on_date, maker_id, processing_result_enum)
VALUES
  ('DISBURSE', 'LOAN', 5001, 2, 'POST',
   '{"loanId":5001,"actualDisbursementDate":"2026-01-15"}',
   '2026-01-15 10:11:00', 1000, 1 /* SUCCESS */);
```

The four statements are one application transaction: the loan state, the portfolio transaction, the balanced ledger rows, and the command audit move together or not at all. That atomicity — not a DB trigger — is what keeps the schema consistent.

### 10.4 The Lessons

1. **One row per loan, everything else hangs off it.** The loan is the aggregate root; its schedule, charges, collateral, aging, and transactions are all `loan_id` children. This is the agreement-centric pattern from `universal_banking_model_guide.md` made concrete.
2. **Snapshot at origination, derive afterward.** Terms are copied from the product onto `m_loan` (the product factory pattern — §5.1); balances are recomputed into `_derived` columns; the schedule is materialized. History is never rewritten by product changes.
3. **Transactions are the only writers of balances.** Every money movement is a `m_loan_transaction` / `m_savings_account_transaction` row; derived balances and the ledger are updated in the same application transaction. Read the mechanics in `core_banking_processes_guide.md` and `posting_engine_core_banking_guide.md`.
4. **The ledger is a convergence point, not a source.** `acc_gl_journal_entry` rows always trace back to a portfolio transaction via `loan_transaction_id` / `savings_account_transaction_id` — audit and reconciliation are joins, not archaeology.
5. **The command journal is the real audit.** If you want to know *why* the schema looks the way it does at any moment, read `m_portfolio_command_source`: it is the complete, replayable story of every API command, maker, checker, and payload.
6. **Reversals, not deletions.** Transactions are flagged (`is_reversed`) rather than removed — the money trail is never broken.

---

## 11. Summary: The Schema Behind the Microfinance Core

One page on the `fineract_default` schema:

- **It is a MySQL schema with a registry.** Two databases per installation — `fineract_tenants` (tenant connections) and `fineract_default` (the default tenant's portfolio). Database-per-tenant multi-tenancy, Liquibase-managed evolution since 1.7.0.
- **It is ~222 tables and counting.** 222 tables / 2,094 columns / 368 constraints in the official May 2022 SchemaSpy snapshot (verified); materially larger in current releases. No views — reporting runs through the `stretchy_report` framework instead.
- **It speaks a prefix grammar.** `m_` for master tables, `acc_gl_` for the general ledger, `ref_` for reference data — so the schema is self-documenting and families are discoverable by `SHOW TABLES LIKE`. (The `r_` and `c_` prefixes sometimes quoted are not confirmed.)
- **It is organized into seven families** — organization (`m_office`/`m_staff`), client (`m_client`), loan (`m_loan`), savings (`m_savings_account`), accounting (`acc_gl_*`), configuration (`m_configuration`), audit (`m_portfolio_command_source`) — mirroring the platform's modules.
- **It models agreements, not products.** Products live in the factory tables (`m_product_loan`, `m_savings_product`); agreements snapshot terms onto `m_loan` / `m_savings_account` and drive everything from a `status_enum` state machine.
- **It caches what it derives.** `_derived` balance columns and `_portion_derived` transaction allocations make reads and reporting trivial at the cost of discipline in the posting engine.
- **It converges on one ledger.** Every posting lane — automatic product mappings (`acc_product_mapping`), manual rules (`acc_accounting_rule`), manual entries — ends in `acc_gl_journal_entry`, linked back to the portfolio transaction that caused it.
- **It audits by command, not by trigger.** `m_portfolio_command_source` is the replayable story of every API call: actor, action, entity, JSON payload, maker/checker, result.
- **It is a microfinance schema first** — group lending (`m_group`), JLGs, disbursement fees, four-bucket repayments (principal/interest/fees/penalties), arrears aging, declining-balance interest — and a general core banking schema second, which is exactly why it has been a fixture of financial inclusion for two decades.

**The final word: the schema behind the microfinance core.** Fineract's data model is the rare core banking schema you can actually read end-to-end — every DDL, every migration, every constraint is public. Its design vocabulary (prefix families, agreement snapshots, derived balances, single-ledger convergence, command audit) is the same vocabulary used by FLEXCUBE, T24, and every other core — but expressed in ~250 tables instead of ~3,000. For a solution architect, it is the best possible *teaching* schema: small enough to hold in your head, complete enough to be a real bank's system of record, and honest enough about its trade-offs (cached derived state, application-enforced integrity, polymorphic links) to teach you what a production core schema actually is.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Fineract** | Apache Fineract — the Apache Software Foundation's open-source (Apache-2.0) core banking engine for microfinance institutions and financial inclusion providers; Java/Spring modular monolith exposing a REST API over a relational schema |
| **Schema** | The structure of a database: its tables, columns, constraints, and relationships; here, the `fineract_default` relational schema that Fineract persists its portfolio in |
| **MySQL** | The open-source relational database that is Fineract's primary engine (MariaDB and PostgreSQL also supported); the schema DDL is written in MySQL dialect |
| **fineract_default** | The default tenant's schema in a Fineract installation — the full portfolio (offices, staff, clients, loans, savings, accounting, configuration, audit) as one MySQL database; documented by the official SchemaSpy analysis |
| **m_client** | The client/customer master table: one row per borrower or account holder, with branch, loan officer, identity, contact, and lifecycle state |
| **m_loan** | The loan agreement table: one row per loan, holding the terms snapshot, the status state machine, and the cached derived balances |
| **m_loan_transaction** | The loan transaction journal: disbursements, repayments, waivers, write-offs, accruals, transfers, each allocated into principal/interest/fee/penalty portions |
| **m_savings_account** | The savings account table: the twin of `m_loan` for the savings portfolio, with its own status lifecycle, terms, and derived balances |
| **m_office** | The organization table: one row per office, forming the institution's hierarchy through a self-referencing `parent_id` (head office → branch) |
| **m_staff** | The staff/employee table: employees and loan officers, each bound to an office via `office_id` |
| **acc_gl** | The accounting prefix and family: the general-ledger tables (`acc_gl_account` chart of accounts, `acc_gl_journal_entry` ledger, `acc_gl_closure`, `acc_product_mapping`, `acc_accounting_rule`) |
| **m_configuration** | The platform configuration table (later releases: `m_global_configuration`): name/value/enabled property rows switching platform behavior |
| **m_portfolio_command_source** | The command/audit journal: one row per API command with actor, action, entity, JSON payload, maker/checker state, and processing result |
| **Audit** | The trail of who did what and when; in Fineract, implemented as the command journal (`m_portfolio_command_source`) plus `createdby_id`/`lastmodifiedby_id` columns, rather than database triggers |
| **Journal** | The record of entries; here, both the portfolio transaction tables (`m_loan_transaction`, `m_savings_account_transaction`) and the general ledger (`acc_gl_journal_entry`) |
| **Table family** | A group of tables sharing a domain and prefix (e.g., the `m_loan*` family: loan, transactions, schedule, charges, collateral, aging) |
| **Prefix** | The leading token of a table name identifying its family (`m_` master, `acc_gl_` accounting, `ref_` reference data) |
| **ER** | Entity-Relationship: the model of entities and their relationships in a schema; "ER relationships" here = the foreign-key and logical links between Fineract tables |
| **Entity relationship** | A link between tables, typically 1:N through a foreign key (e.g., `m_client` → `m_loan` via `client_id`) |
| **Lifecycle** | The state machine an agreement passes through — loan: submitted → approved → active → closed (or rejected/withdrawn/written-off/rescheduled); driven by `status_enum` |
| **Origination** | The creation of a loan: application submitted, terms snapshotted, schedule generated — the `m_loan` INSERT |
| **Repayment** | A borrower payment: a `m_loan_transaction` row allocated across principal/interest/fees/penalties, updating derived balances and posting GL entries |
| **MIFOS** | The precursor platform (Grameen Foundation initiative; development from ~2005, in production from 2006) whose code was contributed to Apache in December 2015 and became Fineract; the source of the `m_` prefix and much of the schema's design |
| **Microfinance** | Financial services for low-income clients: small loans, group lending, frequent small repayments, simple savings — the domain Fineract's schema is optimized for |
| **Core banking** | The system of record for a bank's accounts, transactions, and ledger; Fineract is an open-source core banking engine (a "core" in the sense of `core_banking_systems_guide.md`) |

---

## 13. References and Related Guides

### Primary Sources (verified against)

- Apache Fineract — official site and docs: [fineract.apache.org](https://fineract.apache.org) / [docs/stable](https://fineract.apache.org/docs/stable/) (version 1.15.0 at the time of writing)
- The official SchemaSpy analysis of `fineract_default`: [fineract.apache.org/docs/database](https://fineract.apache.org/docs/database/) — 222 tables / 2,094 columns / 368 constraints (May 2022 snapshot); per-table pages for `m_client`, `m_loan`, `m_loan_transaction`, `m_savings_account`, `acc_gl_journal_entry`, `m_portfolio_command_source`, `m_staff`, and more
- The source repository: [github.com/apache/fineract](https://github.com/apache/fineract) — schema DDL and Liquibase migrations under `fineract-provider/src/main/resources/sql/` (master manifest `changelog-tenant.xml`)
- Apache project wiki — Fineract history: [cwiki.apache.org/confluence/display/FINERACT/Fineract+Home](https://cwiki.apache.org/confluence/display/FINERACT/Fineract+Home) (MIFOS in production since 2006; code contributed to Apache December 2015)
- MIFOS heritage DDL (e.g., `0001a-mifosplatform-core-ddl-latest.sql`) — the lineage of `m_office`, `m_client`, `m_portfolio_command_source` column conventions
- DeepWiki codebase analysis of Fineract — multi-tenancy and Liquibase migration structure: [deepwiki.com/apache/fineract](https://deepwiki.com/apache/fineract)
- Finecko — accounting lanes analysis and multi-tenancy/PostgreSQL guides: [finecko.com](https://finecko.com)

### Sibling Guides in This Repository

- `apache_fineract_guide.md` — the Fineract umbrella (architecture, API, lifecycle, accounting integration; this guide's schema deep-dive is its missing chapter)
- `oracle_flexcube_data_model_guide.md` — the FLEXCUBE data-model deep-dive (the commercial-core contrast: prefix grammar, module families, GL logs, maker/checker)
- `temenos_data_model_guide.md` — the T24 data-model reference
- `data_models_banking_insurance_guide.md` — canonical banking data models (BIAN, IBM BDW, FSLDM)
- `universal_banking_model_guide.md` — the universal banking data model
- `core_banking_systems_guide.md` — the core-banking discipline umbrella
- `interest_engines_core_banking_guide.md` — interest calculation mechanics (the savings interest-posting job)
- `posting_engine_core_banking_guide.md` — posting/journalling mechanics (the `acc_gl_journal_entry` writer)
- `core_banking_processes_guide.md` — core processes (the lifecycle the worked example walks)
- `banking_limits_domain_guide.md` — limits/delinquency domain (the `m_loan_arrears_aging` side)
- `../technology/oracle_database_guide.md` — the database platform layer

---

*End of guide. The schema behind the microfinance core: 222 tables (and growing), one ledger, one command journal, one state machine per agreement — and every DDL public.*


