# Temenos Data Models: Transact, Payments, and Data Hub — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB
> **Repository:** https://github.com/jackliusr/research
> **Last Updated:** August 2026
> **Scope:** The data models of the three Temenos products — Transact (core banking system of record), Payments (payments processing), and Data Hub (batch scheduling, EOD positions, back-office workflow) — with verification notes, comparison to the FLEXCUBE/OBMA data model, and architect guidance.

---

## Table of Contents

1. [What This Guide Covers](#1-what-this-guide-covers)
2. [Transact Data Model Overview: The MultiValue/jBASE Heritage](#2-transact-data-model-overview-the-multivaluejbase-heritage)
3. [The ACCOUNT Application and the Arrangement Architecture (AA)](#3-the-account-application-and-the-arrangement-architecture-aa)
4. [The CUSTOMER Application and the Party Model](#4-the-customer-application-and-the-party-model)
5. [Transactions and Accounting: OFS, FT, STMT.ENTRY, and the GL](#5-transactions-and-accounting-ofs-ft-stmtentry-and-the-gl)
6. [Temenos Payments Data Model](#6-temenos-payments-data-model)
7. [Temenos Data Hub Data Model](#7-temenos-data-hub-data-model)
8. [Data Flow Between the Three Products and the Golden-Source Hierarchy](#8-data-flow-between-the-three-products-and-the-golden-source-hierarchy)
9. [Temenos vs FLEXCUBE: A Data-Model Comparison](#9-temenos-vs-flexcube-a-data-model-comparison)
10. [Worked Example: Savings Account + Incoming Transfer](#10-worked-example-savings-account--incoming-transfer)
11. [Glossary](#11-glossary)
12. [References](#12-references)

---

## 1. What This Guide Covers

This guide documents the **data models** behind the three Temenos products the user named, anchored on their stated roles:

| Product | Role (as framed) | Data-model anchor |
|---|---|---|
| **Temenos Transact** | System of record for accounts, deposits, loans, and general-ledger/back-office processing | The T24/Transact application model (jBASE MultiValue) — CUSTOMER, ACCOUNT, ARRANGEMENT, STMT.ENTRY, GL |
| **Temenos Payments** | Payments processing | The payment record, ISO 20022 message store, UETR, status lifecycle |
| **Temenos Data Hub** | Batch scheduling, end-of-day position visibility, back-office workflow | Extracts, EOD position data, batch job data, reporting/BI layer |

**Relationship to sibling guides:** the vendor/product/architecture context is in [temenos_guide.md](temenos_guide.md) (§12 Transact architecture, §6 Payments, §7 Data Hub). The comparable-vendor data-model pattern is in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md). The payments layer architecture is in [payments_hub_guide.md](payments_hub_guide.md); ISO 20022 message semantics in [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md); canonical banking models (Silverston PARTY/PRODUCT/AGREEMENT/FINANCIAL TRANSACTION) in [data_model_resource_book_guide.md](data_model_resource_book_guide.md) and [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md).

> **Verification note:** This guide's T24/Transact facts were verified against primary T24 documentation sources (the R14 "Application Structure and Files" training deck and the T24 Accounts User Guide), jBASE/TAFC documentation, and the sibling guides. Items that could not be conclusively verified are flagged in-line as **"flag: verify against installed release"** — T24 table names, field sets, and record layouts vary by release and by bank customization, exactly as with FLEXCUBE. The guide follows the same honesty discipline as the FLEXCUBE data-model guide.

---

## 2. Transact Data Model Overview: The MultiValue/jBASE Heritage

### 2.1 The MultiValue Paradigm

Temenos Transact (the evolution of T24) is built on the **jBASE MultiValue database** — a PICK-derived, non-relational data model. The fundamental difference from relational systems:

- **Relational (FLEXCUBE, OBMA):** data in fixed-column tables; relationships via joins; normalization as the design default.
- **MultiValue (Transact):** data in **dynamic arrays** — each "record" is a single string of attribute-delimited values; repeating data lives *inside* the record as **multi-value (MV)** and **sub-value (SV)** fields rather than in child tables.

The trade-offs are the core of every Temenos data discussion:

| Dimension | MultiValue (Transact) | Relational (FLEXCUBE) |
|---|---|---|
| Single-record access | Fast — the whole entity in one read | Requires joins across tables |
| Repeating data | MV/SV fields within the record | Normalized child tables |
| Reporting / ad-hoc SQL | Harder — needs the SQL surface or extracts | Natural |
| Concurrency | Record-level locking | Row/table-level |
| Skill base | Specialized (T24 BASIC, enquiries) | Mainstream SQL |

### 2.2 The "Application" Concept

Every business entity in Transact is an **application** — a file definition with a UPPERCASE dotted name. Examples:

- `CUSTOMER` — the customer master
- `ACCOUNT` — the account master
- `STMT.ENTRY` — the statement/accounting entries
- `FT.COMMISSION.TYPE` — a parameter application (dot-separated namespace)
- `AA.ARRANGEMENT` — the modern arrangement record
- `COMPANY`, `ACCOUNT.PARAMETER`, `LIMIT.PARAMETER`, `AC.BALANCE.TYPE`, `EB.FINANCIAL.SYSTEM` — parameter/control applications

**Application anatomy (verified):** each application is more than a data file. A T24 application consists of:

1. a **PGM.FILE** entry (the program/file definition),
2. a **FILE.CONTROL** entry (multi-company/file behaviour),
3. a **STANDARD.SELECTION** entry (the field list),
4. the **I_** file (the input/validation layer), and
5. **TEMPLATE/THE.TEMPLATE** code (the record layout).

Versions (screen definitions) and enquiries (query definitions) are themselves **records** in the system, not code.

### 2.3 Record Structure

A Transact record (in any application) has:

- **@ID** — the record key (customer ID, account number, etc.).
- **Fields** — numbered attributes; each field has a name. Bank-customized fields use the **`L.` prefix** (local fields).
- **Multi-value (MV) fields** — attributes holding repeating groups (e.g., a customer's multiple addresses in one MV field).
- **Sub-values (SV)** — the second dimension of repetition within an MV field (e.g., per-address street/city/country as SV sub-values).
- **Max 500 fields** per application (`R.NEW(C$SYSDIM)`), which shapes how banks add data (hence the local-field discipline).

### 2.4 Table Storage: Hashed Files (Not ".TAB" Files)

**Correction to a common misconception:** T24 data is **not** stored in ".TAB" files. jBASE/Pick stores data as **hashed files**:

- Each dataset = a **data file** + a separate **dictionary** (`]D` / DICT) file.
- Each application has **companion files**: `$HIS` (history — old versions of records), `$NAU` (unauthorised — records awaiting authorization), and the `]D` dictionary.
- File naming: **`F<MNE>.<APPLICATION>{$<SUFFIX>}`** — e.g. `FBNK.CUSTOMER`, `FBNK.ACCOUNT$NAU`, `FBNK.CUSTOMER.ACCOUNT$HIS` (where `BNK` is the company mnemonic).
- Application types H/U/L/T/W determine which companion files exist.
- **Concat files** (e.g., `CUSTOMER.ACCOUNT`) provide cross-reference lookup paths.

### 2.5 Audit, Maker–Checker, and Record Status

Every T24 record carries audit fields — this is the regulatory/audit backbone (see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) for the audit-trail context):

- **Audit fields:** `INPUTTER`, `AUTHORISER`, `CO.CODE` (company), `CURR.NO` (the "currency of the record" — the modification counter), `RECORD.STATUS`. Under the `Q` function, `AUDIT.DATE.TIME` and `AUDITOR.CODE` are also populated.
- **Record status codes (verified):** `INAU` (input unauth), `IHLD` (input hold), `INAO`, `INA2`, `RNAU` (reverse unauth), `RNA2`, `RNAO`, `REVE` (reverse), `HNAU` (hold unauth). A **live (authorized) record has no RECORD.STATUS** — its absence is the authorized state.
- **The authorization flow (verified):** a new record is created in `$NAU` → authorized → becomes live; amending a live record pushes the previous version to `$HIS` with the id suffix `;1` (the re-audit counter increments).

### 2.6 Multi-Company, Multi-Currency, Multi-Language

- **Multi-company (partially verified):** a single Transact instance hosts many companies via per-company file mnemonics (`F<MNE>`), the `CO.CODE` audit field, and ledger consolidation. The exact per-company field partitioning is release-dependent — **flag: verify against installed release**.
- **Multi-currency:** currency fields and multi-currency amounts are first-class (amounts are held with their currency; the FX revaluation process is part of the EOD cycle).
- **Multi-language:** language fields support localized descriptions/names in the record.

---

## 3. The ACCOUNT Application and the Arrangement Architecture (AA)

### 3.1 The ACCOUNT Application

The `ACCOUNT` application is the legacy account master. Key fields (verified against the Accounts User Guide):

| Field | Meaning |
|---|---|
| `@ID` | The account number (format/length/check-digit set in the `COMPANY` application) |
| `CUSTOMER` | The account-holder link (party) |
| `CATEGORY` | The **product/GL category** — drives GL mapping and product behaviour (see §5.2) |
| `CURRENCY` | The account currency |
| `WORKING.BALANCE` | The balance used for **limit/credit checks** |
| `ONLINE.ACTUAL.BAL` | The ledger balance (online) |
| `ONLINE.CLEARED.BAL` | The cleared balance |
| `OPEN.*` | Start-of-day balance variants |
| `AVAILABLE.BALANCE` | The available-funds ladder (with exposure-date splitting) |
| `ACCOUNT.STATUS` | open/closed/dormant states |
| `ACCOUNT.CLASS` | Internal-account classification (RECORD.TYPE-based) |

**NOSTRO accounts** are identified by `CATEGORY` + `LIMIT.REF='NOSTRO'`. `ACCOUNT.PARAMETER` controls credit-check options. The balance-component semantics (working vs cleared vs available) are the single most common integration point with surrounding systems — the same lesson as the FLEXCUBE balance-component model in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) §5.

### 3.2 The Arrangement Architecture (AA) — the Modern Product-Instance Model

The **Arrangement Architecture (AA)** is Temenos's flagship product-instance model and **the basis of new implementations** (greenfield and Infinity-integrated builds); the legacy `ACCOUNT`/`LENDING`/`DEPOSITS` modules remain for existing books.

The AA model is a three-level structure that maps directly onto the Silverston universals:

```
AA.PRODUCT (product definition — the catalog)
   └── AA.ARRANGEMENT (the agreement/instance — one per customer-product pairing)
         ├── AA.PROPERTY (product properties: interest property, fee property, limit property)
         └── AA.ARRANGEMENT.ACTIVITY (the transactions: disbursement, repayment, interest, accruals)
```

Key AA applications (verified): `AA.ARRANGEMENT`, `AA.ARRANGEMENT.ACTIVITY`, `AA.PROPERTY`, `AA.PRD.DES` (product design), `AA.PRODUCT`, `AA.PRODUCT.GROUP`.

- **Arrangement records are system-generated** when the initiating activity is authorized.
- The AA product families cover LENDING (loans), DEPOSITS (term deposits), and ACCOUNTS (current/savings) — plus the conditions that define behaviour (rates, fees, limits).
- **Silverston mapping:** AA.PRODUCT → the Silverston **PRODUCT** universal; AA.ARRANGEMENT → **AGREEMENT**; the activities → **FINANCIAL TRANSACTION**; CUSTOMER → **PARTY**. This is the cleanest vendor-native expression of the Silverston pattern in core banking — see [data_model_resource_book_guide.md](data_model_resource_book_guide.md) §3.

### 3.3 The Product Model

Products in Transact are **configuration records, not code**: product definitions, conditions (`ACCT.GROUP.CONDITION`, `ACCT.GEN.CONDITION`), rate schedules, and fee structures. The **Model Bank** ships a complete pre-configured product catalogue as the implementation starting point (see [temenos_guide.md](temenos_guide.md) §15 — config-over-code and the Model Bank).

---

## 4. The CUSTOMER Application and the Party Model

### 4.1 The CUSTOMER Application

The `CUSTOMER` application is the party master. Key elements:

- **Customer ID** — the @ID.
- **Customer type** — `CUSTOMER.TYPE`: PERSON, COMPANY, GROUP.
- **Name/address data** — including MV address fields.
- **Identification** — national ID, passport, registration numbers.
- **KYC-related data** — the fields/applications supporting onboarding (see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) for the KYC/AML context).
- **Relationships** — guarantor, joint holders, authorized signatories (via relationship/related-customer structures).
- **The customer 360** — accounts, loans, deposits, and arrangements linked to the customer via the `CUSTOMER` field and concat cross-references (e.g., `CUSTOMER.ACCOUNT`).

### 4.2 Silverston PARTY Mapping

The T24 customer model is a direct expression of the Silverston **PARTY** universal: CUSTOMER = Party, with party roles (holder, guarantor, signatory) realized through relationship data — see [data_model_resource_book_guide.md](data_model_resource_book_guide.md) §3 for the canonical pattern and [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) for the banking-industry data-model context.

---

## 5. Transactions and Accounting: OFS, FT, STMT.ENTRY, and the GL

### 5.1 OFS — The Transaction API

**OFS (Open Financial System)** is the core transaction interface: OFS messages (`OFS.<APPLICATION>`) perform debit/credit and maintenance transactions from external systems and internal modules. It is the transactional integration surface (the "API" of the T24 world) — see [temenos_guide.md](temenos_guide.md) for the modern REST/Open-API layer on top.

### 5.2 The FT Applications and STMT.ENTRY

- **`FUNDS.TRANSFER` (FT)** — the transfer/payment application (the legacy payment path; payments today route through Temenos Payments — see §6).
- **`STMT.ENTRY`** — the statement/accounting entries: the transaction entries posted to accounts. A STMT.ENTRY record carries the account, amount, value date, transaction code, and narrative — the core "what moved and why" record.
- **`CATEG.ENTRY`** — the category/GL entry counterpart.

### 5.3 The Category-Driven GL

Transact accounting is **category-driven**: each account's `CATEGORY` maps to a GL account (balance-sheet 'BS' or profit-and-loss 'PL' categories). The double-entry is realized through STMT.ENTRY/CATEG.ENTRY pairs posting to the GL; the chart of accounts ships with the Model Bank. This differs fundamentally from FLEXCUBE's explicit COA tables (see §9).

### 5.4 The Transaction Lifecycle

1. **Input** — via OFS message or a Version (screen); the record lands in `$NAU`.
2. **Authorization (maker–checker)** — status flow `IHLD` (input hold) / `INAU` (input unauth) → **AUTH** (live; RECORD.STATUS cleared). Reversals use the `RNAU`/`REVE` path.
3. **Value dating** — `VALUE.DATE` vs `BOOKING.DATE`; back-dated ("back value") entries are supported.
4. **Reversals** — reversal entries with their own audit trail.

### 5.5 Multi-Book Accounting

Multi-book accounting (regulatory/statutory books) is supported but release/configuration-dependent — **flag: verify the installed release's multi-book support** before designing around it.

---

## 6. Temenos Payments Data Model

### 6.1 Product Positioning

Temenos Payments is **the payments-hub product** (marketed as a hub; the reviewed documentation does not formally use "Temenos Payments Hub" — **flag: not conclusively named "Temenos Payments Hub"**). It is the ISO 20022 payments-processing layer (pain/pacs/camt messages — see [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)) and the modern successor to the T24 FT module's payment path. The payments-layer architecture context is in [payments_hub_guide.md](payments_hub_guide.md).

### 6.2 The Payment Transaction Entity

The core payment record carries:

- **Payment ID** (internal) and the **UETR** (unique end-to-end transaction reference — the cross-border tracking identifier; see [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) §6).
- **Amount and currency** (instructed amount vs interbank settlement amount where they differ).
- **Party data** — debtor/creditor (and their agents/banks).
- **Payment type** — credit transfer, direct debit, instant payment.
- **Channel data** — SWIFT, SEPA, FAST, ACH (channel-specific fields).

### 6.3 Message Store and Status Lifecycle

- **Message store:** inbound/outbound ISO 20022 payload persistence — `pain.001` (initiation), `pacs.008` (transfer), `pacs.002` (status), `camt.053` (statement) — the message logs are the audit/evidence layer.
- **Status lifecycle** — the ISO statuses: `ACTC` (accepted technical validation) → `ACSP` (accepted settlement in process) → `ACSC` (accepted settlement completed), with `RJCT` (rejected) and `PDNG` (pending) paths — see [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) §6 (state diagram).
- **Event log** — the payment event/status-transition history.
- **Exceptions** — returns, rejects, investigations (`camt.056` claims, `camt.029` resolutions) — see [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) §7.

### 6.4 Reconciliation and Integration Data

- **Reconciliation:** payment-to-account matching via reference/URN matching — the virtual-account pattern of [programmable_business_bank_guide.md](programmable_business_bank_guide.md) §3.
- **Transact integration:** authorized payments post to the core as `STMT.ENTRY`/GL entries (the §5 model) — the payments system is the message truth, Transact is the account/ledger truth (see §8).

---

## 7. Temenos Data Hub Data Model

### 7.1 Product Positioning

Temenos Data Hub is the **data integration/BI layer** — per the user's framing: batch scheduling, EOD position visibility, and back-office workflow efficiency. It is the reporting/analytics surface over the core (the "golden source" for reporting — see §8). **Flag: exact product positioning varies by release; treat as the data platform over Transact.**

### 7.2 Data Model Layers

| Layer | Content |
|---|---|
| **Staging** | Raw extracts from the core (nightly account/transaction/customer extracts) |
| **Data marts / warehouse** | Subject-area models for reporting and analytics |
| **EOD position data** | End-of-day balances/positions per account — the "position keeping" records |
| **Batch scheduling data** | Batch job definitions, schedules, job runs, job logs |
| **Back-office workflow data** | Workflow instances, task data, process definitions |
| **Reporting data** | Report definitions, output, regulatory reports |
| **Analytics** | BI cubes, dashboards |

### 7.3 The EOD Data Flow

The canonical overnight sequence feeding the Data Hub: cut-off → entry rollover → interest/charges postings → FX revaluation → GL close → output generation (statements, regulatory extracts, warehouse feeds) — the same EOD rhythm documented for FLEXCUBE in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) §5.5, which is why the Data Hub's EOD position records are the natural cross-vendor comparison point.

---

## 8. Data Flow Between the Three Products and the Golden-Source Hierarchy

### 8.1 The Data Flows

```
Transact (core) ── payment initiation ──▶ Temenos Payments
      ▲                                      │
      │ ISO 20022 message (pacs.008)         │ settlement/confirmation
      │                                      ▼
      └──────── STMT.ENTRY / GL postings ◀───┘

Transact ── nightly EOD extracts ──▶ Temenos Data Hub ──▶ reporting / BI
Temenos Payments ── payment data ──▶ Temenos Data Hub ──▶ analytics
```

- **Transact ↔ Payments:** a payment initiated in the core flows to the Payments hub, becomes an ISO 20022 message, settles, and the confirmation posts back to the core (STMT.ENTRY + GL).
- **Transact ↔ Data Hub:** EOD extracts (accounts, transactions, customers) feed the Data Hub for reporting/BI.
- **Payments ↔ Data Hub:** payment data flows to the Data Hub for analytics.

### 8.2 The Golden-Source Hierarchy

| Domain | System of Record |
|---|---|
| Accounts / transactions / GL | **Transact** (the accounting truth) |
| Payment messages / status / UETR | **Temenos Payments** (the message truth) |
| Reporting / analytics / EOD positions | **Temenos Data Hub** (the reporting truth) |

Architects should design integrations around this hierarchy: never treat the Data Hub as a transactional system of record, and never treat Payments as the ledger.

---

## 9. Temenos vs FLEXCUBE: A Data-Model Comparison

### 9.1 Comparison Table

| Dimension | Temenos Transact | FLEXCUBE (FCUBS) |
|---|---|---|
| **Paradigm** | MultiValue dynamic arrays (jBASE) | Relational, normalized tables |
| **Entity naming** | Dotted applications (`CUSTOMER`, `STMT.ENTRY`) | Prefixed tables (`STTM_`, `ACTB_`, `FTMB_`...) |
| **Account model** | AA arrangement (ARRANGEMENT → PROPERTY → ACTIVITY); legacy ACCOUNT | CIF/account master (`STTM_CUSTOMER`, account tables) |
| **Product instance** | AA.PRODUCT → AA.ARRANGEMENT | Product definition vs account instance |
| **GL** | Category-driven (`CATEGORY` → GL) | COA tables + control accounts |
| **Maker–checker** | `IHLD`/`INAU`/AUTH (RECORD.STATUS cleared when live) | `UNAUTH`/`AUTH` (`AUTH_STAT` field) |
| **Transaction API** | OFS messages | SQL/API layers |
| **Reporting** | Enquiries + Data Hub extracts | SQL views + warehouse feeds |
| **Audit trail** | `$HIS` history files + audit fields | History tables + audit fields |

### 9.2 The Architect's View

- **Data access patterns:** OFS for transactional integration; jBASE SQL/JDBC (read-oriented) for direct SQL; enquiries for operational queries; **the Data Hub for analytics**.
- **Integration design:** the modern REST/Open-API layer and events (see [temenos_guide.md](temenos_guide.md)); OFS for the classic path.
- **Migration:** data migration to/from Transact stands or falls on reproducing the balance/GL reconciliation (the same lesson as FLEXCUBE — see [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) §8).
- **Test data:** synthetic data with masking; **governance:** the data dictionary (the `]D` dictionaries and the application catalogue) and lineage.

---

## 10. Worked Example: Savings Account + Incoming Transfer

### 10.1 The Transact Flow

A customer opens a savings account and receives a transfer. The applications touched:

1. **CUSTOMER** — the party record (created/updated at onboarding).
2. **AA.ARRANGEMENT** — the savings arrangement (the agreement; system-generated when the opening activity is authorized).
3. **ACCOUNT** — the account master (linked to the arrangement and the customer).
4. **STMT.ENTRY** — the incoming-transfer entry (account, amount, value date, transaction code, narrative).
5. **OFS FUNDS.TRANSFER / the payment path** — the initiating transfer (via Temenos Payments for the message side).
6. **GL entries (CATEG.ENTRY)** — the category-driven double-entry postings.

Record sketches (simplified):

```
CUSTOMER: @ID=100123 | NAME=ACME PTE LTD | CUSTOMER.TYPE=COMPANY | ...
ACCOUNT:  @ID=50123456789 | CUSTOMER=100123 | CATEGORY=1001 | CURRENCY=SGD |
          WORKING.BALANCE=0 | ONLINE.ACTUAL.BAL=0 | ACCOUNT.STATUS=OPEN | ...
AA.ARRANGEMENT: @ID=... | PRODUCT=ACCOUNTS.SAVINGS | CUSTOMER=100123 | ...
STMT.ENTRY: @ID=... | ACCOUNT=50123456789 | AMOUNT=+10,000 | VALUE.DATE=today |
            TXN.CODE=... | NARRATIVE='INCOMING TRANSFER' | ...
```

### 10.2 The Payments Equivalent

Temenos Payments: the payment record (UETR, debtor/creditor, amount), the `pacs.008` message, the status transitions (`ACTC → ACSP → ACSC`), and the confirmation that triggers the core posting.

### 10.3 The Data Hub Equivalent

The EOD extract moves the account/position to the Data Hub: the position record (end-of-day balance, date) feeds the reporting and BI layers.

---

## 11. Glossary

| Term | Meaning |
|---|---|
| **jBASE** | The MultiValue database engine underlying Transact |
| **MultiValue / MV / SV** | The dynamic-array model; multi-value fields and sub-values for repeating data |
| **Application** | A T24 business entity/file definition (CUSTOMER, ACCOUNT, ...) |
| **@ID** | The record key |
| **`L.` fields** | Bank-customized local fields |
| **$HIS / $NAU / ]D** | History, unauthorised, and dictionary companion files |
| **OFS** | Open Financial System — the transactional message interface |
| **FT** | FUNDS.TRANSFER — the transfer/payment application |
| **STMT.ENTRY** | The statement/accounting entry record |
| **CATEGORY** | The product/GL category driving accounting |
| **AA / ARRANGEMENT / PROPERTY / ACTIVITY** | The Arrangement Architecture model |
| **IHLD / INAU / AUTH / RNAU / REVE** | Record status codes (maker–checker) |
| **UETR** | Unique end-to-end transaction reference |
| **Temenos Data Hub** | The data/BI layer (EOD positions, batch, reporting) |
| **Golden source** | The system-of-record hierarchy across the three products |

---

## 12. References

- Temenos T24 "Application Structure and Files" training deck (R14) — application anatomy, companion files, record structure.
- T24 Accounts User Guide — ACCOUNT fields, balance semantics, account classes.
- jBASE/TAFC documentation — hashed-file storage, jBASE SQL/JDBC (read-oriented).
- [temenos_guide.md](temenos_guide.md) — vendor, product portfolio, architecture, deployment.
- [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) — the comparable FLEXCUBE/OBMA data model.
- [payments_hub_guide.md](payments_hub_guide.md), [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) — payments layer and message semantics.
- [data_model_resource_book_guide.md](data_model_resource_book_guide.md), [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) — Silverston and canonical banking models.
- [programmable_business_bank_guide.md](programmable_business_bank_guide.md) — virtual-account reconciliation pattern.

---

> **Honesty footer:** T24/Transact table names, field sets, and record layouts vary by release and bank customization. Validate every application/field referenced here against the installed release's data dictionary before quoting it in a design. Items flagged in-text as "verify against installed release" (multi-book accounting, exact per-company partitioning, Data Hub product positioning, Temenos Payments naming) were not conclusively verifiable from the reviewed documentation as of August 2026.
