# Oracle FLEXCUBE Universal Banking (FCUBS) and Oracle Banking Microservices Architecture (OBMA): Data Models — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Core Banking / Data Architecture — FLEXCUBE Data Model (the "CUBS" Core), OBMA Data Model, Product/Agreement/Account Modeling, GL and Accounting, Event-Driven Data, Migration, Banking Data Modeling  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [What This Guide Covers](#1-what-this-guide-covers)
2. [FLEXCUBE Universal Banking Data Model Overview](#2-flexcube-universal-banking-data-model-overview)
3. [The FLEXCUBE Customer Model (Party)](#3-the-flexcube-customer-model-party)
4. [The FLEXCUBE Account and Product Model](#4-the-flexcube-account-and-product-model)
5. [The FLEXCUBE Transaction and Accounting Model](#5-the-flexcube-transaction-and-accounting-model)
6. [FLEXCUBE Data Model Characteristics](#6-flexcube-data-model-characteristics)
7. [The OBMA Data Model](#7-the-obma-data-model)
8. [FCUBS vs OBMA: Comparison and Migration](#8-fcubs-vs-obma-comparison-and-migration)
9. [The Banking Context: Working with These Data Models](#9-the-banking-context-working-with-these-data-models)
10. [Worked Example: A Savings Account Opened and Funded](#10-worked-example-a-savings-account-opened-and-funded)
11. [Glossary](#11-glossary)
12. [References and Related Guides](#12-references-and-related-guides)

---

## 1. What This Guide Covers

### 1.1 Two Data Models, One Story

Oracle's banking story spans two very different data architectures under the same vendor roof:

- **Oracle FLEXCUBE Universal Banking (FCUBS)** — the traditional, monolithic core banking system built on a large, highly normalized relational schema (the "CUBS" core, §2.1). One database, thousands of tables, one integrated general ledger, batch-driven end-of-day processing.
- **Oracle Banking Microservices Architecture (OBMA)** — Oracle's cloud-native successor, where the same banking domains (party, account, product, transaction, limit, collateral) are decomposed into microservices that **each own their data**, communicate through **events**, and persist in per-service schemas with JSON where flexibility is needed.

This guide is the **data-model deep-dive** for both platforms: the FLEXCUBE data dictionary and table conventions, the customer/account/product/transaction entities, the accounting model, the characteristics that make the schema recognizable (authorization statuses, audit columns, branch on every table), and then how OBMA deliberately inverts most of those choices (data ownership, events as the source of truth, read models). It complements, rather than duplicates, the platform-level architecture coverage in `oracle_banking_microservices_architecture_guide.md`.

### 1.2 How This Guide Relates to the Sibling Guides

- `oracle_banking_microservices_architecture_guide.md` — OBMA architecture, service topology, product factory, event-driven design, and the FLEXCUBE → OBP → OBMA evolution (the data layer is only summarized there in §5.3; this guide expands it into a full data-model treatment).
- `core_banking_systems_guide.md` — the discipline-level umbrella: core data model patterns (§7.8), the ledger and double-entry accounting (§7.3), product factory patterns (§7.4), transaction processing (§7.6).
- `data_model_resource_book_guide.md` — Silverston's universal PARTY/PRODUCT/AGREEMENT/FINANCIAL TRANSACTION model and the banking worked examples (§9) that this guide maps FCUBS and OBMA onto.
- `data_models_banking_insurance_guide.md` — canonical banking data models (BIAN, IBM BDW, FSLDM, Oracle OFSDFM), balance/position modeling (§6.3), agreement lifecycle modeling (§6.4), event-driven data models (§6.7).
- `bian_banking_architecture_guide.md` — BIAN service domains and business objects (the §6 BOM is the vocabulary OBMA aligns to).
- `banking_limits_domain_guide.md` — the limits domain: tables, engines, and integration (FLEXCUBE's limits/collateral module vs OBMA's Limits service).
- `financial_risk_compliance_systems_guide.md` — audit context, BCBS 239 data lineage, regulatory reporting systems.
- `temenos_guide.md` and `apache_fineract_guide.md` — the vendor comparisons (Temenos T24/Transact data model, Fineract's relational model) for contrast.

### 1.3 A Note on Verification (Read This First)

FLEXCUBE's schema is documented in Oracle's per-release **data dictionary / table-description documents** (delivered to licensed customers) and in module user guides, but there is **no public, complete table reference**. The facts in this guide fall into three buckets, marked where it matters:

1. **Verified against Oracle's public documentation** — e.g., `STTM_CUSTOMER`, `STTM_CUST_ACCOUNT`, `ACTB_DAILY_LOG`, `STTM_PRODUCT`, `STTM_TRN_CODE`, the Customer Information File (CIF) module, the `TM`/`TB`/`TW` table-type convention.
2. **Widely documented field knowledge** — e.g., the maker/checker audit columns, `AUTH_STAT`/`REC_STAT` status fields, the module prefixes (`ACTB_`, `DETB_`, `LNTB_`, `CLTB_`), balance-component terminology. These are consistent across practitioner references and integration guides, but table names and column sets **vary by release and by bank customization**, so treat any specific table as "check the release data dictionary."
3. **Folklore flagged as such** — the "CUBS" acronym expansion (§2.1) is community terminology, not an Oracle-documented name.

Where a claim is release-dependent, the guide says so. That honesty matters: a data architect quoting FLEXCUBE table names to a client must always validate against the **installed release's dictionary**, not a blog.

---

## 2. FLEXCUBE Universal Banking Data Model Overview

### 2.1 FCUBS and the "CUBS" Core — Verifying the Name

**FLEXCUBE Universal Banking (FCUBS)** is Oracle's traditional core banking platform, developed by Oracle Financial Services Software (OFSS, formerly **i-flex Solutions** — a Citibank IT spinoff, acquired by Oracle in 2005). It has been in production since the late 1990s and is one of the largest installed bases in core banking, especially in Asia, the Middle East, Africa, and Eastern Europe.

**The "CUBS" naming — verified status: unverified, treat as community shorthand.** The task of pinning down "CUBS" leads to a folklore trail rather than an Oracle-documented name:

- Oracle's official documentation names the product **FLEXCUBE Universal Banking Solutions (FCUBS)** and its siblings (FLEXCUBE Investor Servicing, FLEXCUBE Lending, FLEXCUBE Core). The name "FLEXCUBE" itself is widely explained as **"FLEXible CUBE"** — the "CUBE" being the core engine concept, variously expanded in trade articles and practitioner blogs as *Comprehensive Universal Banking Environment* or *Comprehensive Universal Banking System* (hence "CUBS").
- In the field, practitioners use **"CUBS"** loosely to mean *the classic FLEXCUBE UBS core engine and its data model* — the monolithic schema that this guide describes — to distinguish it from the newer Oracle Banking Platform (OBP) / OBMA lineage. You will also see it in job ads and forum posts ("CUBS developer", "CUBS tables").
- **No Oracle documentation reviewed for this guide uses "CUBS" as an official product or schema name.** If the acronym matters in a deliverable, cite it as informal terminology: *"CUBS — the classic FCUBS core/data model, informally 'Comprehensive Universal Banking System'; not an Oracle-documented product name."*

What is certain and verifiable: FCUBS runs on **Oracle Database**, is delivered as a **single integrated schema per bank** (with per-branch data inside it), and its module/user manuals (CIF, CASA, General Ledger (FGL), Deposits, Retail Lending, Funds Transfer, Limits & Collateral, Messaging, etc.) describe the entities this guide covers.

### 2.2 The FLEXCUBE Data Dictionary: Scale

Oracle does not publish a single official table count, and the schema grows with each release (plus each bank's customization adds tables). Realistic figures from the field:

- A fresh FCUBS **14.x schema contains on the order of 2,000–3,500 tables**, views, and sequences; practitioner references commonly cite "3,000+ tables." The task's "~3000+" figure is therefore in the right range — state it as *"roughly 3,000 tables, release-dependent."*
- The dictionary is organized by **module**: each functional module (static data, CASA, deposits, loans, collateral, funds transfer, accounting/GL, limits, messaging, security) owns a family of tables, buffers, and views.
- Tables are overwhelmingly **normalized** (hundreds of tables per module), with child tables carrying multi-valued attributes (addresses, UDFs, rate schedules) that a modern design would store as JSON or repeated rows.

The scale has real architectural consequences: the FLEXCUBE schema is the bank's system of record for everything — customers, accounts, balances, transactions, the GL — so reporting and integration teams spend their lives joining module tables through **views** (e.g., `ACVW_ALL_AC_ENTRIES`, a documented view joining the daily and history accounting logs) rather than querying base tables directly.

### 2.3 Table Naming Conventions: The Prefix Grammar

FLEXCUBE table names follow a consistent, readable grammar. The verified convention (documented in practitioner references and consistent with Oracle's own docs):

```
[MM] [T] [X] _ [content]
 │    │   │      └── content (6th char onward): what the table holds
 │    │   └── 4th char: table type  (M = Maintenance/Master, B = Buffer/Internal,
 │    │                             W = Work/Temporary)
 │    └── 3rd char: always 'T' (Table)
 └── 1st–2nd chars: module prefix (ST, AC, DE, LN, CL, FT, CS, SM, …)
```

So `STTM_CUSTOMER` reads: **ST** (static module) + **T** (table) + **M** (maintenance) + `_CUSTOMER` (content). The commonly seen prefixes:

| Prefix | Module / Type | Verified examples | What it holds |
|---|---|---|---|
| `STTM_` | **ST**atic, Table, **M**aintenance | `STTM_CUSTOMER`, `STTM_CUST_ACCOUNT`, `STTM_CURRENCY`, `STTM_BRANCH`, `STTM_PRODUCT`, `STTM_TRN_CODE`, `STTM_ACCOUNT_CLASS`, `STTM_FIN_CYCLE`, `STTM_PERIOD_CODES` | Master/maintenance data: customers, accounts, currencies, branches, products, transaction codes, account classes, financial cycles |
| `STTMS_` | STatic, Table, Maintenance, **S**ub | `STTMS_CUST_PERSONAL` | Multi-row child/sub-tables of a master (e.g., per-customer personal details rows) |
| `STTB_` | STatic, Table, **B**uffer | `STTB_ACCOUNT`, `STTB_VALUE_DESC` | Internal/buffer tables: working copies, value descriptions (code translations) |
| `ACTB_` | **AC**counting, Table, **B**uffer | `ACTB_DAILY_LOG`, `ACTB_HISTORY_LOG`, `ACTB_DAILY_GHOST_LOG`, `ACTB_RECON_MASTER` | The accounting engine's transaction/entry logs and reconciliation masters |
| `DETB_` | **DE**posits, Table, **B**uffer | `DETB_*` (term deposit buffers) | Deposit module buffers/transaction tables |
| `LNTB_` | **LN** (lending), Table, **B**uffer | `LNTB_*` (loan buffers) | Loan module buffers |
| `CLTB_` | **CL** (collateral), Table, **B**uffer | `CLTB_ACCOUNT_APPS_MASTER` | Collateral module buffers |
| `FTMB_` / `FTTB_` | **FT** (funds transfer), Table, M/B | `FT*_*` (payment/message tables) | Funds-transfer maintenance and buffers — **prefix varies by release; check the dictionary** |
| `CSTB_` | **CS** (cash), Table, **B**uffer | `CSTB_CONTRACT` | Cash module contract buffers |
| `ACVW_` | **AC**counting, **V**ie**W** | `ACVW_ALL_AC_ENTRIES` | Accounting views (not tables) |

Key rules to remember:

- **`TM` = master/maintenance, `TB` = buffer/internal, `TW` = temporary work tables.** The 3rd–4th characters are the type discriminator; the first two are the module.
- **Every table name has an underscore as the 5th character** (in the classic naming); the 6th character onward names the content.
- **The same content word appears across types**: `STTM_CUSTOMER` (master) vs the API-staging `RATM_CUSTOMER`; `STTB_ACCOUNT` (buffer) vs `STTM_CUST_ACCOUNT` (master).
- **`STTM_CUST_ACCOUNT` vs `STTM_CUSTOMER_ACCOUNT`**: the account master is most widely documented as **`STTM_CUST_ACCOUNT`** in modern releases (the task's `STTM_CUSTOMER_ACCOUNT` spelling appears in older/alternate documentation); always resolve against the installed release's dictionary.

### 2.4 System vs User Tables

FLEXCUBE distinguishes (practically, not always formally):

- **System/master tables** — the delivered dictionary (`STTM_*`, `ACTB_*`, etc.), owned by the FLEXCUBE schema and maintained through the application's maintenance screens with full maker-checker and audit.
- **User-defined / customization tables** — every master supports **UDFs (User-Defined Fields)**: banks extend `STTM_CUSTOMER`, `STTM_CUST_ACCOUNT`, etc. with custom columns or child UDF tables (e.g., `STTMS_*_UDF`) without touching delivered code. This is the sanctioned extension mechanism and a major source of schema drift between banks on the same release.
- **Staging / API tables** — newer releases pair REST services with staging tables; Oracle's REST documentation shows pairs like `RATM_CUSTOMER` / `STTM_CUSTOMER`, `RATM_PRODUCT` / `STTM_PRODUCT`, `RATM_TRN_CODE` / `STTM_TRN_CODE`, `RATM_ACCOUNT` / `STTB_*` — the `RATM_` ("REST API table, maintenance"?) side receives API traffic before it lands in the business tables.
- **Work/temporary tables** (`*TW_*`) — batch and EOD scratch space, safe to truncate.
- **Security tables** — `SMTB_*` (e.g., `SMTB_USER_GLEXCEPT`) for security-management user/GL-exception data.

For an architect the practical split is: **business-of-record tables** (never touch directly in production — go through the application or its APIs) vs **reporting/integration surfaces** (views and EOD output tables that are fair game for ETL).

### 2.5 Multi-Currency, Multi-Language, Multi-Branch Design

Three orthogonal dimensions are baked into the schema:

- **Multi-currency** — currency is a first-class column (`CCY`) on every account, balance, and rate entity; `STTM_CURRENCY` is the currency master. Balances are stored in **account currency (ACY)** and, where needed, the reporting currency; the accounting engine generates both the local-currency and the reporting-currency legs of every posting (the `ACY_OPENING_BAL`-style columns on `STTM_CUST_ACCOUNT` show the pattern). Cross-currency transactions flow through a **revaluation / FX module** that revalues positions at the day's rates.
- **Multi-language** — language is data, not code. Text that must display in the user's language is maintained per-language through description/value tables — the `STTB_VALUE_DESC`-style tables exist precisely to hold *translations of codes* for display; master records carry a `LANGUAGE` column (e.g., `customer_name1` vs the localized description). There is no per-language schema duplication.
- **Multi-branch / multi-entity** — **the branch code (`BRANCH_CODE`) is a column on virtually every business table**, and composite keys are typically `(BRANCH_CODE, entity_key)`. A single FCUBS database can serve hundreds of branches; branch is the unit of the financial cycle (each branch runs its own EOD), of user access, and of GL. This is the *multi-entity* design discussed in §6.4.

The three dimensions interact in ways that shape every design decision:

| Dimension | Where it lives in the schema | What the architect must respect |
|---|---|---|
| Currency | `CCY` column on accounts/balances/rates; `STTM_CURRENCY` master | Balances are stored in account currency; reporting-currency conversions are generated at posting time, not at read time |
| Language | `LANGUAGE` column on masters; per-language description/value tables (`STTB_VALUE_DESC`-style) | Text you see is *data*; don't hard-code display text in integrations — resolve codes through the value-description tables |
| Branch | `BRANCH_CODE` on every business table; composite PKs | Always scope queries by branch; a "customer" spans branches but an account/GL entry never does |

A consequence worth internalizing: because currency, language, and branch are **columns rather than separate schemas**, FCUBS can serve a multi-country bank from one database — but every report, extract, and API must carry the branch and currency dimensions explicitly or it silently produces wrong aggregates.

---

## 3. The FLEXCUBE Customer Model (Party)

### 3.1 STTM_CUSTOMER — The Customer Master

The customer master is `STTM_CUSTOMER`, keyed by `CUSTOMER_NO` (plus `BRANCH_CODE` in the classic composite-key style). Verified/documented columns include:

- `CUSTOMER_NO` — the bank-wide customer identifier (the CIF number in FLEXCUBE terminology, §3.3)
- `CUSTOMER_TYPE` — the customer type code (§3.2)
- `CUSTOMER_NAME1` / `CUSTOMER_NAME2` — name fields (the `LANGUAGE` column selects the display name)
- `BRANCH_CODE` — home branch
- `CCY` — preferred currency
- Address, contact, and identification detail held in **child tables** (STTMS-style sub-tables: addresses, phones, emails, identity documents)
- Audit/status columns: `MAKER_ID`, `MAKER_DT_STAMP`, `CHECKER_ID`, `CHECKER_DT_STAMP`, `MOD_NO`, `REC_STAT`, `AUTH_STAT`, `ONCE_AUTH` (see §5.4 and §6.5)

Every customer-facing entity — accounts, loans, deposits, limits, collateral, guarantees — references `CUSTOMER_NO`, which is what makes the customer the hub of the model.

### 3.2 Customer Types

FLEXCUBE classifies customers by `CUSTOMER_TYPE`, with the standard set covering:

- **Individual / retail** — natural persons (with sub-types for joint holdings and minor accounts)
- **Corporate** — companies and other legal entities (with corporate structure links: parent/holding, subsidiaries, group relationships)
- **Bank** — financial institutions and counterparty banks (used for nostro/vostro relationships, interbank limits)
- **Non-individual / non-resident variants** — partnerships, associations, government bodies, non-resident individuals — plus **group** customers: FLEXCUBE's CIF manual is explicit that a *group* (e.g., "Customer A and Customer B belong to group Z") is itself defined as a separate customer record, so a group can own accounts and guarantees exactly like any other customer.

The type drives validation rules, KYC requirements, and which relationship types are legal (e.g., only individuals can be authorized signatories in many jurisdictions).

### 3.3 The CIF (Customer Information File) Concept — Verified

**CIF terminology is verified as core FLEXCUBE vocabulary.** Oracle ships a dedicated **"FLEXCUBE Customer Information File (CIF) User Manual"**, and the customer module's maintenance screens are CIF functions (e.g., `CIM09 — Customer Information Master Maintenance`). In FLEXCUBE:

- The **CIF is the customer master record** — the single file that consolidates identity, addresses, contacts, KYC, and relationships for one customer.
- The **CIF number is the customer identifier** — in many FLEXCUBE banks this is the `CUSTOMER_NO` itself (banks often map it to the local regulatory "CIF number" concept familiar from Asian banking, where a passbook shows a CIF number linking all of a customer's accounts).
- CIF is the legacy-flavored name for what modern modeling calls the **party**: FLEXCUBE's CIF is effectively a pre-PARTY model — a customer-centric hub with typed relationships, which is exactly the shape Silverston's PARTY formalizes (§3.5).

### 3.4 The Customer 360: Customer as Hub

The "customer 360" in FLEXCUBE is not a separate store — it is the **join graph** centered on `CUSTOMER_NO`:

```
STTM_CUSTOMER (CUSTOMER_NO)
  ├── STTM_CUST_ACCOUNT (CUST_NO)        → CASA accounts (current/savings)
  ├── DETB_* / deposit masters (CUST_NO)  → term deposits, recurring deposits
  ├── LNTB_* / loan masters (CUST_NO)     → loans and facilities
  ├── Limits & collateral (CUST_NO)       → CLTB_* collateral, limit utilization
  ├── Relationship tables (CUSTOMER_NO)   → guarantor, joint holder, signatory, beneficiary
  └── KYC / CIF extension tables          → risk rating, documents, screening results
```

Any customer-facing screen (account statement, loan summary, limit utilization, KYC dossier) is a query fanning out from this hub. The 360 *view* is assembled at query time; there is no denormalized customer aggregate in the core (that is exactly what OBMA's Party service and read models change, §7.4–7.5).

### 3.5 Mapping to the Silverston PARTY Model

Mapping FLEXCUBE's CIF to Silverston's universal model (see `data_model_resource_book_guide.md` §9.1) is direct, with one caveat: FLEXCUBE predates the strict supertype/subtype discipline:

| Silverston construct | FLEXCUBE equivalent | Gap |
|---|---|---|
| PARTY (supertype: PERSON / ORGANIZATION) | `STTM_CUSTOMER` with `CUSTOMER_TYPE` | Type is a code column, not a subtype table; the supertype/subtype split is implicit |
| PARTY ROLE (Customer, Guarantor, Signatory…) | `CUSTOMER_TYPE` + relationship tables | Roles beyond the primary customer role are modeled as **relationships**, not as a role entity |
| PARTY RELATIONSHIP (guarantees, holds-jointly-with, is-employer-of) | CIF relationship tables (guarantor, joint holders, signatories, group membership) | Typed and directional, with validity dates — close to the universal pattern |
| PARTY IDENTIFIER (NRIC, passport, tax ID, LEI) | Identification child tables per customer | Multiple identifiers supported, but not always date-effective in older releases |
| CONTACT MECHANISM (address, phone, email) | Address/contact child tables (STTMS-style) | Supported; purpose/validity typing varies by release |

The architect's takeaway: **FLEXCUBE's CIF is a pragmatic, relationship-capable PARTY model without Silverston's formal supertype discipline.** When mapping FCUBS data into a canonical model (for a data warehouse, or for OBMA migration), the `CUSTOMER_TYPE` column and the relationship tables carry almost everything needed to populate PARTY/PARTY ROLE/PARTY RELATIONSHIP.

### 3.6 Customer Data Entities

The customer data entities, in FLEXCUBE terms:

- **Customer master** — `STTM_CUSTOMER`: identity, type, name(s), language, home branch, preferred currency, risk attributes.
- **Addresses** — child tables per customer (residential, correspondence, registered office), each with purpose and (in newer releases) validity dates.
- **Contact details** — phone, fax, email, telex, with **media** and delivery preferences (the `DEFAULT_MEDIA` column appears in documented account queries — media drives statement/advice delivery).
- **KYC data** — CIF extension: customer KYC status, risk rating, PEP/sanctions screening results (often interfaced from a dedicated AML system — see `financial_risk_compliance_systems_guide.md`), documents (via the image/document tables, e.g., `STTM_CUST_IMAGE`).
- **Relationships** — the relationship graph that makes the model bankable:
  - **Guarantor** — guarantor → customer relationship with limits and validity (drives loan collateral and limits modules)
  - **Joint holders** — joint account ownership with mandate rules (any-one / all / survivorship)
  - **Authorized signatories** — individuals authorized to operate corporate accounts, with signing instructions
  - **Group/hierarchy** — group customers, parent/subsidiary links, employer/employee links

These relationships are **typed, directional, and date-effective** — which is why FLEXCUBE can answer AML/FATCA questions like "who is connected to whom and why" without a graph database. The relationship set in practice:

| Relationship | Direction | What it enables |
|---|---|---|
| Guarantor → customer | Guarantor `CUSTOMER_NO` → borrower `CUSTOMER_NO`, with guaranteed amount/validity | Loan collateralization, limits utilization |
| Joint holder ↔ joint holder | Mutual, with mandate (any-one / all / survivorship) | Joint CASA accounts |
| Authorized signatory → corporate customer | Individual → corporate, with signing limits/instructions | Corporate account operations |
| Group membership | Group customer ↔ member customers | Group-level limits, consolidated statements |
| Parent/subsidiary | Corporate ↔ corporate | Group structures, consolidated reporting |
| Employer/employee | Corporate → individual | Salary processing, affinity products |

Each relationship row carries its own maker/checker and validity dates, so the relationship history is fully auditable — a requirement when regulators ask "who could sign on this account on date X."

---

## 4. The FLEXCUBE Account and Product Model

### 4.1 The Product-Instance Pattern

FLEXCUBE implements the universal **definition → instance** pattern (see `data_model_resource_book_guide.md` §9.2): a **product** is the parameterized definition; an **account** is the product instance carrying the balance. In FLEXCUBE the chain is:

```
Product (definition, STTM_PRODUCT + product parameter tables)
   → Account Class (STTM_ACCOUNT_CLASS — groups of accounts sharing attributes)
      → Account (instance, STTM_CUST_ACCOUNT — the customer account with the balance)
         → Transactions (ACTB_DAILY_LOG entries, §5)
```

The **account class** (`STTM_ACCOUNT_CLASS`) is a distinctive FLEXCUBE intermediate layer: accounts *inherit properties from an account class* (interest rules, fee rules, GL linkage, cheque-book attributes), and account classes are in turn attached to products — so the bank can tune behavior at either level.

### 4.2 Product Definition Tables (The Product Catalog)

- **`STTM_PRODUCT`** — the product master (verified in Oracle's REST documentation as the business table behind the product service). One row per product: product code, product type (CASA / deposit / loan / etc.), currency applicability, GL linkage.
- **Product parameter tables** — the product definition is spread across many module-specific parameter tables (interest rate schedules, fee schedules, tenor/maturity rules, posting/GL mappings, channel availability). Rates and fees are themselves **data with effective dates** (rate tables), which is the temporal pattern (§6.3).
- The product catalog is **configuration, not code**: a new savings product is a new product row plus its parameter rows, not a code change. This is the *product factory* idea in its classic, schema-bound form (see `core_banking_systems_guide.md` §7.4 and the OBMA product factory, §4.8 below).

### 4.3 Account Instance Tables

- **`STTM_CUST_ACCOUNT`** — the customer account master (the "ACCOUNT" entity; older docs spell it `STTM_CUSTOMER_ACCOUNT`). Verified/documented columns include: `CUST_AC_NO` (account number), `CUST_NO` (customer), `CCY`, `ACCOUNT_CLASS`, `ACY_OPENING_BAL` (account-currency opening balance), `ACC_STATUS` (statuses like `NORM` normal / `OVER` overdrawn), `BRANCH_CODE`, plus audit/status columns.
- Per-product-type account tables extend the master: deposit-specific attributes live in the deposits module's buffers (`DETB_*`), loan-specific attributes in the lending module's buffers (`LNTB_*`), collateral in `CLTB_*`.
- **Account numbering** is bank-configurable (product-prefixed number series), which is why integrations should join on the internal key, not parse account numbers.

### 4.4 Account Subtypes

- **CASA (Current And Savings Accounts)** — the demand-deposit book: current accounts and savings accounts on `STTM_CUST_ACCOUNT`, differentiated by account class/product (interest-bearing or not, cheque book, overdraft linkage).
- **Term deposits** — deposit contracts with tenor and maturity (deposits module: booking, rollover, premature-withdrawal, interest accrual/capitalization), held in the deposits tables/buffers and linked to the customer.
- **Loans** — retail/corporate loan accounts (lending module): disbursement, repayment schedule, interest (simple/compound, fixed/floating), fees/penalties, rescheduling — held in the lending buffers (`LNTB_*`).
- **GL / internal accounts** — the bank's own accounts (nostro, suspense, clearing, expense/income) that live in the accounting module rather than the customer account master.

### 4.5 Account Attributes: Balances and Balance Components

Balances are the performance-critical heart of the model. FLEXCUBE maintains, per account, a **set of balance components** (the terminology maps to `core_banking_systems_guide.md` §7.8):

| Balance component | Meaning |
|---|---|
| **Ledger / book balance** | The accounting truth — what the GL says the account holds |
| **Cleared balance** | Ledger balance excluding uncleared items (cheques in clearing) |
| **Uncleared** | Funds deposited but not yet cleared (float) |
| **Available balance** | What the customer can actually use: ledger − holds − liens − uncleared debits |
| **Lien** | Amount frozen by the bank (security for another product/limit) |
| **Hold / block** | Temporary reservations (card authorizations, cheque holds) |

Balances are maintained in **account currency (ACY)** and derived aggregates (average balance, period balances) are stored/updated by the accounting engine; balance history tables support statements and interest calculation. The daily log/balance tables (`ACTB_DAILY_LOG`, balance-period tables such as `ACTB_DAILY_BAL` in some releases — **check the dictionary**) are partitionable by date and are the largest tables in the schema.

A simplified view of how the components sit on an account record:

```text
Account 0012345678 (SGD)
├── Ledger/book balance ........ 10,000.00   ← the GL truth
├── Cleared balance ............  9,500.00   ← excludes uncleared items
├── Uncleared (float) ..........    500.00   ← cheque deposited, not cleared
├── Holds / blocks .............    200.00   ← card authorization reservation
├── Lien .......................  1,000.00   ← security for a loan product
└── Available balance ..........  8,800.00   ← ledger − holds − lien (usable)
```

Every posting adjusts the component set according to the transaction type and the account's rules: a cheque deposit increases uncleared first and clears later; a card authorization places a hold that releases or converts to a posting; a lien is raised and released by the collateral/limits module. Getting these semantics right is the difference between a balance that reconciles and one that doesn't — the single most common integration defect in FLEXCUBE shops.

### 4.6 Account Statuses

Every account carries a status lifecycle. Documented examples on `STTM_CUST_ACCOUNT` include `NORM` (normal) and `OVER` (overdrawn); the fuller status set (release-dependent) covers: active/opened, dormant, inoperative, frozen/blocked, lien-marked, closed. Status is **state with audit** — every transition is maker-checked and stamped, which matters for both compliance (see `financial_risk_compliance_systems_guide.md` audit context) and for integration (a channel reading `ACC_STATUS` must handle the full code set).

### 4.7 Account Relationships to Customers and Products

- **Account → Customer**: `STTM_CUST_ACCOUNT.CUST_NO` → `STTM_CUSTOMER.CUSTOMER_NO`; joint accounts add relationship rows (joint holders with mandates).
- **Account → Product/Class**: `ACCOUNT_CLASS` → `STTM_ACCOUNT_CLASS` → product.
- **Account → Branch**: `BRANCH_CODE` (the account is domiciled in one branch even if serviced anywhere).
- **Account → GL**: each account maps (via product/class) to GL heads — the account is a *subsidiary ledger* whose sum must equal the GL control account (§5.2).

### 4.8 The Product Factory Pattern (FCUBS vs OBMA)

FLEXCUBE's product configuration and OBMA's product factory are the same idea at different architectural ages:

- **FCUBS**: products as **rows in a fixed schema** — `STTM_PRODUCT` plus hundreds of module parameter tables; creating a product is a data-entry task in maintenance screens, but *changing the shape* of products (new parameter, new behavior) requires schema/code change and a release.
- **OBMA**: the product factory is a **service** where products are **documents/definitions assembled via APIs** — a product definition bundles type, pricing, fees, limits, GL mappings, and regulatory attributes as data, and new product shapes can be introduced through configuration rather than schema change (see `oracle_banking_microservices_architecture_guide.md` §7). See §7.5 for the OBMA product data model.

### 4.9 Mapping to Silverston PRODUCT/AGREEMENT/ACCOUNT

FLEXCUBE collapses Silverston's four-level chain (see `data_model_resource_book_guide.md` §9.2) into three:

| Silverston level | FLEXCUBE | Note |
|---|---|---|
| PRODUCT (definition) | `STTM_PRODUCT` + parameter tables | Present, schema-bound |
| AGREEMENT (the customer's contract/terms) | **No first-class agreement entity** — the customer's specific terms (rate override, fee waiver, limit) live as columns/overrides on the account | This is the biggest modeling gap: FCUBS bakes the contract into the account instance |
| ACCOUNT (instance with balance) | `STTM_CUST_ACCOUNT` (+ module buffers) | Present |
| FINANCIAL TRANSACTION | `ACTB_DAILY_LOG` / `ACTB_HISTORY_LOG` entries | Present, immutable-by-convention (§5) |

OBMA, by contrast, restores the agreement concept in its data model (§7.5) — a direct consequence of the Silverston/BIAN discipline the OBP/OBMA design followed.

---

## 5. The FLEXCUBE Transaction and Accounting Model

### 5.1 Transaction Tables

Every movement of money becomes **accounting entries** in the FLEXCUBE accounting engine:

- **`ACTB_DAILY_LOG`** — the daily transaction/entry log: one row per *accounting entry* (each debit/credit leg of a transaction), written in real time during the day. This is *the* transaction table of FCUBS. Verified: Oracle's own Database Practices guide uses `ACTB_DAILY_LOG` as the canonical volatile table for partitioning/statistics guidance.
- **`ACTB_HISTORY_LOG`** — the history log: entries moved out of the daily log at EOD (the documented view `ACVW_ALL_AC_ENTRIES` joins daily + history to give the full entry picture).
- **`ACTB_DAILY_GHOST_LOG`** — "ghost" (back-valued / future-dated) entries held outside the main log until their value date (§5.4).
- **`ACTB_RECON_MASTER`** — reconciliation master for matching entries (nostro reconciliation, internal matching).

Transaction *capture* happens in the functional modules before posting: teller transactions, EFT/remittance (`FT*` funds-transfer tables), SWIFT/MT messages (messaging module), standing orders/direct debits, and the interest/fee engines all generate entries into this accounting core. The transaction code (`STTM_TRN_CODE` — the transaction-code master) classifies every entry and drives GL mapping and reporting.

### 5.2 Double-Entry Accounting and the GL

FLEXCUBE's accounting model is textbook double-entry:

- **Every transaction posts balanced DEBIT/CREDIT entries** into `ACTB_DAILY_LOG` (the sum of debits equals the sum of credits per transaction and per branch-day). A customer deposit, for example, debits the cash/nostro account and credits the customer account; interest accrual posts accrual entries; fees post fee entries.
- **The GL (general ledger) is integrated in the core** — the "General Ledger (FGL)" module maintains the **chart of accounts (COA)**: a GL master (GL code hierarchy, e.g., `STTM_GL`-style GL master tables — **check the release dictionary for the exact GL master name**) with parent/child GL heads, currency-specific GLs, and per-branch GL books.
- **Subsidiary-ledger control**: customer accounts, deposit accounts, and loan accounts are subsidiary ledgers; their totals must equal the GL control accounts. The trial balance (per branch, per currency, per GL) is produced from the GL and reconciles against the sum of account balances — the daily reconciliation that keeps the core honest.
- **GL mapping** is product/transaction-code-driven: each product/event maps to debit and credit GL heads, which is why product setup and GL setup are inseparable in implementations.

The trial balance, in essence:

```sql
-- The daily trial balance (per branch, per currency, per GL): the sum of
-- authorized entries by GL head must net to zero across D and C.
SELECT branch_code, gl_code, ccy,
       SUM(CASE WHEN drcr = 'D' THEN amount ELSE 0 END) AS total_dr,
       SUM(CASE WHEN drcr = 'C' THEN amount ELSE 0 END) AS total_cr
FROM   actb_daily_log
WHERE  auth_stat = 'A'                       -- authorized entries only
GROUP  BY branch_code, gl_code, ccy
HAVING total_dr <> total_cr;                 -- anything here is a break
```

The GL's control accounts tie the subsidiary ledgers (customer accounts, deposit contracts, loan accounts) to the GL: each product/account-class maps to a control GL, and the sum of account balances must equal the control GL balance. Finance runs this reconciliation every day; a core replacement or migration stands or falls on reproducing it (§8.2).

### 5.3 Multi-Book Accounting — Verified Status

The task asks about "accounting books: statutory, tax, management, IFRS." **Verified status: the classic FCUBS core does not have a first-class multi-book (multi-GAAP) accounting architecture.** What FCUBS actually provides:

- A **single integrated GL per branch/entity** — one chart of accounts, one set of books, with statutory reporting derived from it.
- **IFRS/regulatory views are produced outside the core**, primarily in **Oracle Financial Services Analytical Applications (OFSAA)**, which ingests the core's GL and accounting-entry data and applies IFRS 9 / regulatory reclassification (see `data_models_banking_insurance_guide.md` §2.6 on OFSDFM, the OFSAA data foundation). FCUBS supplies the underlying data (GL balances, accounting entries, product/contract attributes); it does not maintain parallel GAAP books.
- In the **OBP/OBMA lineage**, accounting services are more granular and some products expose book-level accounting structures — but treat "books" as a **feature to verify against the specific product release**, not a platform guarantee.

Architectural consequence: a bank needing statutory + tax + IFRS books on FCUBS builds the second/third books in the analytics/reporting layer (OFSAA or the warehouse), not in the core — a material fact for finance-transformation programs.

### 5.4 The Transaction Lifecycle

**Maker-checker authorization.** Every FLEXCUBE master and transaction record carries an authorization status — the maker-checker discipline is structural, not procedural. The verified field set (documented on `STTM_CUSTOMER` and standard across tables):

- `MAKER_ID`, `MAKER_DT_STAMP` — who created the record and when
- `CHECKER_ID`, `CHECKER_DT_STAMP` — who authorized it and when (null until authorized)
- `AUTH_STAT` — the authorization status: **`U` (UNAUTH — created, awaiting authorization) and `A` (AUTH — authorized); some tables add further states (e.g., reversed/`R`, deleted/`D`)** — the exact code set is per-table/release, but U/A are the universal pair
- `ONCE_AUTH` — whether the record has *ever* been authorized (prevents silent re-edits after authorization)
- `MOD_NO` — modification counter (concurrency/audit), incremented on each change
- `REC_STAT` — record status (e.g., open/closed), orthogonal to authorization

A transaction or master is **not visible to downstream processing until authorized** — interest runs, EOD, and reports all filter on `AUTH_STAT = 'A'`. This is the single most important integration rule for FLEXCUBE data: *always filter on authorization status; unauthorized rows are drafts.*

**Value dating.** Every entry carries at least two dates:

- **Booking date** — when the entry was actually posted/recorded.
- **Value date** — when the entry takes economic effect (the date from which interest/balances count).
- **Back-value entries** — entries with a value date *before* the booking date (e.g., a correction booked today for value three days ago) are handled as ghost entries (`ACTB_DAILY_GHOST_LOG`) until their value date arrives, then merged into the daily log. Back-value dating is a first-class, audited mechanism — not an afterthought.

**Reversals.** Errors are not deleted; they are **reversed**. A reversal posts offsetting entries (with reversal codes and a reference to the original entry), and the original entries remain in the log for audit. Reversal handling matters for reconciliation: reporting must decide whether to show gross entries, net of reversals, or both.

### 5.5 The End-of-Day Data Flow

The batch/EOD cycle is where the FCUBS data model does its heaviest lifting — and where most integration surprises live. The canonical overnight sequence:

1. **Cut-off** — the branch stops accepting same-value-date transactions; late entries are booked for the next day (or as back-value entries).
2. **Entry rollover** — `ACTB_DAILY_LOG` rows are moved to `ACTB_HISTORY_LOG` (the daily log starts each day clean); ghost entries whose value date has arrived are merged into the new daily log.
3. **Interest and charges** — the interest engine computes accruals and postings per account (using the balance-history tables and rate schedules); fees post; the resulting entries are *authorized internally* and land in the accounting logs.
4. **Revaluation** — multi-currency positions are revalued at the day's rates (FX module), producing revaluation entries.
5. **GL close** — the GL module aggregates entries into the trial balance per branch/currency; control accounts are reconciled against subsidiary ledgers; period-end (EOM/EOY) processes close GL periods and open the next.
6. **Output generation** — statements, advices, regulatory extracts, and the warehouse feeds are produced from the closed-day data (with `AUTH_STAT = 'A'` filters).

Data-model consequences: the **history log and balance-history tables grow without bound** (partition by date, archive per retention policy); **EOD output tables** are the sanctioned reporting surface (extract from them, not from live tables); and any real-time channel that needs same-day data must read *during* the day, because after EOD the day's truth lives in the history tables. OBMA's answer to this whole cycle is §7.3 — the cycle doesn't disappear, but the *data* is available as events in real time, and EOD shrinks to period-close and reporting.

---

## 6. FLEXCUBE Data Model Characteristics

### 6.1 Normalization vs Denormalization

FLEXCUBE is **heavily normalized** — hundreds of tables per module, with child tables for every multi-valued attribute (addresses, contacts, UDF values, rate schedules, relationship rows). This is the classic 1990s OLTP design: it maximizes integrity and flexibility at the cost of query complexity. Denormalization is deliberately avoided in the core; the **views** (e.g., `ACVW_ALL_AC_ENTRIES`) and the EOD/batch output tables provide the denormalized surfaces for reporting and integration. The ETL rule of thumb: never join 20 base tables in the warehouse — use the shipped views and the accounting logs.

### 6.2 Generic vs Specific Tables

Two coexisting styles:

- **Specific tables** — most module tables are purpose-built (a customer master, an account master, a deposit buffer).
- **Generic/type-coded tables** — a significant minority are *multi-purpose tables with a type code*, where one physical table serves many business objects distinguished by a `TYPE`/`CLASS` column (e.g., value-description tables that hold translations for *any* code; generic parameter tables; `STTM_PERIOD_CODES`-style financial-period definitions). This "generic table + type code" pattern is how FLEXCUBE absorbs product variety without a table per product — and it is precisely the pattern OBMA generalizes into JSON documents (§7.2).

### 6.3 Date-Effective Data (The Temporal Model)

Master and parameter records are **date-effective**: effective-from/effective-to dates on rate schedules, product parameters, customer attributes, and relationship rows. Rate changes are *inserts of new effective-dated rows*, not overwrites — the temporal pattern Silverston formalizes (see `data_model_resource_book_guide.md` §4.3). This is what makes historical interest recalculations and audit possible. The same discipline extends to the two-date transaction model (booking vs value date, §5.4).

### 6.4 Multi-Entity Design (Branch on Every Table)

`BRANCH_CODE` on virtually every business table (§2.5) is the multi-entity mechanism: branch is the unit of books, of the financial cycle (per-branch EOD), of GL, and of user authorization. Legal-entity separation (multiple legal entities in one bank) is layered on top via branch groupings/entity parameters — a design that serves branch networks well and multi-legal-entity groups less well (a common FCUBS pain point that OBMA's per-service tenancy addresses differently).

### 6.5 Audit Fields (The Audit Trail)

Beyond the maker/checker columns (§5.4), the schema's audit posture is:

- **Every record** carries maker/checker stamps, `MOD_NO`, `AUTH_STAT`, and `REC_STAT` — authorization and audit are *columns on the row*, not a separate audit log.
- **Reversal and back-value entries** leave the original entries intact (immutable-by-convention ledger).
- **Amendment tracking** — `GWTB_AMEND_FIELDS`-style tables (documented in Oracle's extensibility docs) record which fields changed in an amendment, supporting field-level audit.
- This column-level audit is the FCUBS answer to the audit-trail requirements discussed in `financial_risk_compliance_systems_guide.md`; for BCBS 239-style lineage it is the *source* that the bank's lineage tooling must capture (see §9.2).

### 6.6 Localization (Country-Specific Tables and Regulatory Parameters)

Localization is data-driven: country/region parameter tables, holiday calendars (per branch/currency), currency-specific rules (decimal places, rounding), regulatory parameters (statutory reserve ratios, reporting thresholds), and per-language description tables. Banks add country-specific tables and UDFs during implementation; the delivered dictionary plus the bank's customization *is* the deployed schema. This is why two FCUBS banks on the same release can have materially different dictionaries — and why a data dictionary inventory is a mandatory migration/reporting artifact (§9.6).

### 6.7 The Characteristics at a Glance

| Characteristic | FCUBS posture | Where covered |
|---|---|---|
| Normalization | Highly normalized; hundreds of tables per module; views as the reporting surface | §6.1 |
| Generic vs specific | Specific tables dominate; generic type-coded tables absorb product variety | §6.2 |
| Temporal data | Date-effective rows everywhere (rates, parameters, relationships); booking vs value date | §6.3 |
| Multi-entity | `BRANCH_CODE` on every table; branch = books/GL/EOD unit | §6.4 |
| Audit | Maker/checker columns, `AUTH_STAT`, `REC_STAT`, `MOD_NO`, `ONCE_AUTH` on every record | §6.5 |
| Localization | Country/currency/language as data; bank customization adds tables | §6.6 |
| Authorization | Structural maker-checker — nothing processes until `AUTH_STAT = 'A'` | §5.4 |
| Extensibility | UDFs and user tables on masters; no schema-free escape hatch | §2.4 |

Read together, these characteristics define the FCUBS *personality*: a strongly governed, strongly typed, audit-first relational core. Every one of them is either inverted or softened in OBMA (§7), which is the cleanest way to understand what the two platforms actually differ on.

---

## 7. The OBMA Data Model

### 7.1 How OBMA Differs: Data Ownership and Database-Per-Service

OBMA inverts the FCUBS design at the architectural root. Where FCUBS has **one schema for everything**, OBMA follows **microservices data ownership**:

- **Database-per-service (schema-per-service)**: each domain service owns its data exclusively — the Party service owns party records, the Accounts service owns account records, the Transactions service owns transaction records. No other service reads another service's tables directly; cross-domain reads happen through **APIs or events**.
- **Shared data is service-owned too**: reference data (currencies, rates, holidays), product definitions, and security data live in shared/common services (the OBMA **Common Core** provides shared data services — see `oracle_banking_microservices_architecture_guide.md` §5.3), not in a global schema.
- **Tenancy**: in Oracle Banking Cloud Service (SaaS) mode, isolation is schema- or database-per-tenant.
- The price of this decomposition: **no cross-table joins across domains** — an architect who could join `STTM_CUSTOMER` to `STTM_CUST_ACCOUNT` in one query must now call two services (or consume two event streams) and join in the consumer. This is the fundamental data-architecture trade of OBMA, and it is why the event backbone (§7.3) and read models (§7.4) exist.

### 7.2 OBMA Data Stores — Verified Picture

Verified against the OBMA architecture guide (§5.3) and Oracle's current database portfolio:

- **Oracle Database is the system of record** — account balances, transactions, party records, product definitions, limits — on **Autonomous Database (ATP)** for managed deployments or **Exadata** for high-end/on-prem. OBMA does **not** run its system of record on a separate NoSQL cluster.
- **JSON where flexibility is needed**: where FCUBS uses generic tables with type codes, OBMA uses **Oracle's JSON type / JSON collection tables** inside Oracle Database — including **MongoDB-compatible document access** via Autonomous JSON Database for workloads that want a document API. The honest answer to "does OBMA use MongoDB or Oracle NoSQL?" is: *Oracle's converged database gives it document-store capability without a second database; a bank should verify per service which tables are relational vs JSON in the release's schema documentation.*
- **Redis-compatible cache (OCI Cache)** — for hot reads: balances, product parameters, rate lookups. Cache is *derived*, never authoritative; the database remains the source of truth.
- **OCI Streaming** (Kafka-compatible) — the event backbone (§7.3); events also feed the data lake (OFSAA / OCI Data Lake / Object Storage) for analytics.
- **Object storage** — statements, advices, documents (Document Generation Service), files.

### 7.3 Event-Driven Data: Events as the Source of Truth

The most profound data-model difference: **in OBMA, events are first-class data** (see `event_stream_processing_guide.md` for the general pattern):

- **Domain events as business facts** — `account.opened`, `account.funded`, `payment.executed`, `loan.disbursed`, `limit.breached` (event taxonomy in the OBMA guide §8). Each event is a **JSON payload** describing a business fact with its own schema (versioned), carrying the aggregate id, the changed attributes, and timestamps.
- **Transactional outbox pattern** — to guarantee no lost events, the event is written to an **outbox table in the same database transaction** as the business state change, and a relay publishes outbox rows to the event bus (verified as OBMA's documented reliability pattern in the architecture guide §8.3). This gives *at-least-once* delivery and per-aggregate ordering (Kafka-style partitioning by account/party key).
- **Events as the integration contract** — downstream systems (analytics, notifications, data lake, regulatory reporting) consume events instead of querying the core. The event stream *is* the data backbone: the same facts that FCUBS would expose only as tables after EOD are available to consumers in real time.
- **Event sourcing vs event-notification**: OBMA is primarily **event-notification with state in the database** (the service persists current state; events announce changes), not full event sourcing where the event log is the only store — but the outbox + replayable stream gives many event-sourcing benefits (audit, replay into the data lake). An architect should verify per service whether full event sourcing is used for any aggregate.

**The outbox in practice.** The transactional outbox is a small, high-leverage data structure — worth sketching because it is the OBMA equivalent of the FCUBS maker/checker discipline (both exist to make data trustworthy):

```sql
-- In the SAME database transaction as the business change:
-- 1. UPDATE account SET balance = balance - 100 WHERE account_id = :id;
-- 2. INSERT INTO outbox (event_id, aggregate_type, aggregate_id,
--                        event_type, payload_json, created_at)
--    VALUES (:uuid, 'ACCOUNT', :id, 'payment.executed',
--            '{"txnId":"...","amount":100,"ccy":"SGD","valueDate":"2026-08-05"}',
--            SYSTIMESTAMP);
-- COMMIT;   -- state and event are atomic
-- A relay process then reads the outbox (in order), publishes each row to the
-- event bus, and marks it published. At-least-once delivery + replay on failure.
```

**A domain event payload** (illustrative, versioned JSON):

```json
{
  "eventId": "evt_9f2c…",
  "eventType": "account.funded",
  "eventVersion": "1.0",
  "occurredAt": "2026-08-05T09:31:22.482Z",
  "aggregate": { "type": "ACCOUNT", "id": "acct_88412001" },
  "payload": {
    "accountId": "acct_88412001",
    "partyId": "pty_100234",
    "productId": "savings-plus",
    "currency": "SGD",
    "amount": 10000.00,
    "balanceAfter": { "ledger": 10000.00, "available": 10000.00 },
    "valueDate": "2026-08-05", "bookingDate": "2026-08-05"
  }
}
```

Consumers subscribe by `eventType`; the schema is the contract. Adding a field is a version bump; breaking changes require a new event version and a migration of consumers — the same discipline as changing a FLEXCUBE table, but governed by schemas instead of DDL.

### 7.4 Read Models and CQRS

To square "each service owns its data" with "queries must be fast and cross-domain":

- **CQRS** — commands (state changes) go to the owning service's write model; queries hit **read-optimized projections**. The write model is normalized for integrity; the read model is denormalized for query speed.
- **Read models** are built by **consuming the event stream** (the projection pattern): e.g., a Customer 360 read model that joins party + accounts + loans + limits facts by subscribing to all domain events and maintaining a denormalized document per customer. This is the OBMA answer to the FCUBS customer-360 join graph of §3.4 — assembled asynchronously, in the query path, from events.
- **Eventual consistency is accepted and managed**: an API may return slightly stale read-model data while the write is in flight; balance-critical reads (e.g., pre-payment authorization) go to the owning service, not the read model. The architect's job is deciding *which* reads may be eventually consistent (customer 360, statements, dashboards) and which must be strongly consistent (balance check, limit check).

**A read-model sketch** — the OBMA customer 360 as an event projection:

```text
Customer 360 read model (per partyId, stored as a JSON document or denormalized rows)
  partyId: pty_100234
  party:   { name, type, kycStatus, ... }              ← from party.created/updated events
  accounts: [ { accountId, productId, currency, status,
                balances: { ledger, available } } ]     ← from account.opened/funded events
  deposits: [ { depositId, tenor, rate, maturity } ]    ← from deposit.booked events
  loans:    [ { loanId, facilityId, outstanding } ]     ← from loan.disbursed events
  limits:   [ { limitId, type, utilization } ]          ← from limit.utilization events
```

The projection is *built* by consuming the event stream and *kept fresh* by the same stream; rebuilding it is a replay, not a migration. This is the CQRS answer to the FCUBS join graph of §3.4 — and it is why OBMA can serve a 360 view without cross-service table access.

### 7.5 The Data Model per Domain

OBMA's domain decomposition (party, product, account, transaction, limit, collateral — see the service catalog in the OBMA guide §6) with its data shapes:

| Domain service | Data owned (persisted) | Notes |
|---|---|---|
| **Party** | Party records (individual/organization, type), addresses, contacts, identifiers, KYC status, relationships (joint, guarantor, signatory) | The Silverston PARTY model made explicit — including the supertype/subtype that FCUBS approximates with `CUSTOMER_TYPE` |
| **Product (Product Factory)** | Product definitions: type, pricing (rates with effective dates), fees, limits, tenors, GL mappings, regulatory attributes | Products as **documents/JSON** assembled via API (the product factory, OBMA guide §7) |
| **Accounts** | Account/agreement records: account number, party links, product link, status, **balance components** (ledger, available, cleared, lien, hold), holds/liens | Restores the **AGREEMENT** concept: the customer's contract (rate overrides, fee waivers, terms) is data distinct from the account instance — closing the FCUBS gap of §4.9 |
| **Transactions** | Transaction records, entries, history, reversals, audit | The immutable entry log, but service-owned and event-emitted rather than a shared `ACTB_DAILY_LOG` |
| **Limits** | Limit definitions, limit utilization, limit check results | See `banking_limits_domain_guide.md` §12 for the limit-engine data patterns |
| **Collateral** | Collateral objects, valuations, allocations, margins | Owned by the collateral/limits service, not a `CLTB_*` family |

### 7.6 Mapping OBMA to Silverston and BIAN

- **Silverston**: OBMA's domain model maps *cleanly* onto the universal model — PARTY (Party service), PRODUCT (Product Factory), AGREEMENT + ACCOUNT (Accounts service), FINANCIAL TRANSACTION (Transactions service). Where FCUBS collapses AGREEMENT into the account, OBMA restores the four-level chain (§4.9 table). This is no accident: the OBP/OBMA design was built with exactly the definition→instance→activity discipline that `data_model_resource_book_guide.md` §9.2 describes.
- **BIAN**: OBMA's service domains align to **BIAN service domains and business objects** (see `bian_banking_architecture_guide.md` §6, the Business Object Model): Party ↔ `Party`/`Customer` BOM, Accounts ↔ `Account` (with `AccountBalance`, `AccountLimit` objects), Product Factory ↔ `Product`/`ProductDirectory`, Transactions ↔ `Payment`/`FinancialTransaction`, Limits ↔ `LimitCheck`/`LimitDefinition`. The API payloads (§7.7) are the practical BIAN alignment: OBMA's OpenAPI data definitions are the vendor's concrete take on BIAN's semantic APIs (BIAN guide §9).

| BIAN service domain / BOM object (illustrative) | OBMA service | FCUBS equivalent |
|---|---|---|
| `Party` / `PartyRelationship` | Party Management | `STTM_CUSTOMER` + CIF relationship tables |
| `Product` / `ProductDirectory` | Product Factory | `STTM_PRODUCT` + parameter tables |
| `Account` / `AccountBalance` / `AccountLimit` | Accounts | `STTM_CUST_ACCOUNT` + balance components |
| `PaymentExecution` / `PaymentOrder` | Payments / Transactions | `FT*` tables + `ACTB_DAILY_LOG` |
| `LimitCheck` / `LimitDefinition` | Limits | FLEXCUBE limits & collateral module |
| `FinancialTransaction` / `LedgerPosting` | Transactions / Accounting | `ACTB_DAILY_LOG` / `ACTB_HISTORY_LOG` |

The mapping is not cosmetic: a bank that has adopted BIAN as its architecture vocabulary (common in Singapore/Asia modernization programs) can map OBMA services onto BIAN service domains almost one-for-one, whereas FCUBS requires translation at every layer (screens → tables → APIs).

### 7.7 The OBMA Data Dictionary: API Payloads vs Persisted Data

OBMA's "data dictionary" has two layers that must not be conflated:

- **The API data model** — every REST/OpenAPI operation carries **JSON schemas** (request/response data definitions): the contract consumers see. These are versioned, published (Swagger/OpenAPI), and are what a bank's integration teams build against. Field names and structures follow the domain vocabulary (e.g., `partyId`, `accountId`, `balanceComponents[].type/value`).
- **The persisted data model** — the tables/JSON documents inside each service's schema, which **do not have to match the API shape**. Services translate between the two; persistence is optimized for integrity/performance, APIs for semantics. Unlike FCUBS (where the API layer is a thin veneer over `STTM_*` tables — the `RATM_*` staging tables of §2.4), OBMA's API contract is the *primary* integration surface and the persisted model is an implementation detail.

For a data architect this split is the single biggest OBMA mindset shift: **you design against JSON schemas and events, not against a published table dictionary.** There is no OBMA equivalent of "the `STTM_CUSTOMER` column list"; there is an OpenAPI catalog and a Kafka event catalog.

---

## 8. FCUBS vs OBMA: Comparison and Migration

### 8.1 The Comparison Table

| Dimension | FCUBS (classic core) | OBMA (microservices core) |
|---|---|---|
| **Architecture** | Monolithic app, single integrated schema per bank | Microservices suite, **database-per-service / schema-per-service** data ownership |
| **Schema** | Fixed relational schema, ~2,000–3,500 tables (release-dependent), highly normalized | Per-service relational tables **+ JSON documents**; no global schema; OpenAPI/JSON-schema contracts |
| **Data dictionary** | Published per-release table descriptions; the bank's deployed dictionary includes customizations | API payload schemas + event schemas are the dictionary; persisted model is per-service and largely internal |
| **Accounting** | **Integrated GL in the core** — `ACTB_DAILY_LOG` entries, chart of accounts, trial balance, single set of books | **Accounting as services** — entries posted by the owning services; GL integration via events/APIs into the finance/GL systems (often OFSAA or an external ledger); verify book structures per product |
| **Events** | Batch/EOD-centric; change data is discovered by reading tables after EOD; back-value/ghost entries | **Event-driven, real-time** — domain events with JSON payloads, transactional outbox, replayable stream; batch shrinks to reporting |
| **Consistency** | Strong ACID within one database (cross-table joins, single transaction) | Strong ACID **per service**; **eventual consistency across services** via events and read models |
| **Audit/authorization** | Maker-checker columns on every row (`AUTH_STAT` U/A, maker/checker stamps) | Authorization workflows in services; audit via event stream + service audit records (no universal column set) |
| **Product model** | Products as rows in `STTM_PRODUCT` + parameter tables; product shape changes need schema/release | Products as **data definitions in the Product Factory** (JSON); new product shapes via configuration |
| **Agreement model** | No first-class agreement — contract terms live on the account | AGREEMENT restored as data distinct from the account instance |
| **Multi-entity** | `BRANCH_CODE` on every table; branch = books/GL/EOD unit | Tenancy at schema/database level (SaaS); branch/legal-entity as service data |
| **Customer 360** | Query-time join graph on `CUSTOMER_NO` | **Read model** projected from events (CQRS), eventually consistent |

### 8.2 Migration: FCUBS → OBMA

The migration is a **data migration plus a paradigm migration** (the OBMA guide's §14 covers the program shape; this section is the data angle):

**Data mapping — CUBS tables → OBMA domain services.** The mapping follows the domain decomposition:

| FCUBS (source) | OBMA (target) | Mapping notes |
|---|---|---|
| `STTM_CUSTOMER` + CIF child tables | **Party service** | `CUSTOMER_TYPE` → party type/roles; addresses/contacts/identifiers → party data objects; relationships → party relationship objects |
| `STTM_PRODUCT` + parameter tables | **Product Factory** | Product definitions *re-created* as product-factory definitions (not copied row-by-row — the factory is the new product catalog) |
| `STTM_CUST_ACCOUNT` + module buffers | **Accounts service** | Account master → account/agreement records; `ACC_STATUS`, balance components → account status/balance objects; contract overrides → agreement terms |
| `ACTB_DAILY_LOG` / `ACTB_HISTORY_LOG` | **Transactions service** | Entries → transaction records; history window decision (typically migrate 12–24 months; older data archived and queryable from the legacy system) |
| Limits/collateral tables | **Limits / Collateral services** | Limit definitions and utilization → limit service data (see `banking_limits_domain_guide.md`) |
| GL master/entries | Accounting/GL integration | GL mappings re-created per product/event; the GL itself may stay in FCUBS or OFSAA during coexistence |

**Migration approaches — big-bang vs incremental/parallel run.** The OBMA guide's §14 pattern is the industry reality:

| Approach | What happens | Data implications | Verdict for FCUBS→OBMA |
|---|---|---|---|
| **Big-bang** | All books migrate in one cutover; legacy frozen | One-time full extract/transform/load; GL continuity risk concentrated in one weekend | High risk; rarely chosen |
| **Incremental (wave-by-wave)** | Party/accounts → payments → loans/deposits → cards/limits/trade (OBMA guide §14.3) | Per-wave mappings; the legacy system keeps serving unmigrated books; reconciliation runs daily across the boundary | The norm |
| **Parallel run / coexistence** | Dual processing: new business on OBMA, legacy books on FCUBS, synchronized via APIs/events (OBMA guide §14.2, §14.5) | Two systems of record for overlapping books; the consolidated customer 360/balances live in the integration layer (§9.1); daily balance/GL reconciliation is mandatory | The pragmatic hybrid most banks actually run |

- **Big-bang** (all books in one cutover) — rarely chosen for FCUBS→OBMA; the data volumes, the GL continuity requirement (the trial balance must reconcile on day one), and the product-factory re-creation effort make it high-risk.
- **Incremental / parallel run (the norm)** — wave-by-wave migration (party and accounts → payments → loans/deposits → cards/limits/trade), **dual processing** with daily reconciliation of balances and positions, per-book cutover on weekends with rollback plans, and **coexistence** where FLEXCUBE keeps the legacy books while OBMA fronts new business (the OBMA guide §14.5 calls this the pragmatic hybrid — and it is also precisely the "customer 360 aggregates across systems" integration problem of §9.1).

**Data quality and reconciliation.** The non-negotiables:

- **Balance reconciliation** — every migrated account's ledger balance in OBMA must match FCUBS at cutover instant; balance components (lien, hold, uncleared) migrate as data, not just the net figure.
- **GL reconciliation** — the migrated book's trial balance must reconcile; GL mappings are re-created per product/event and validated by re-posting a sample of historical transactions.
- **Product parity** — product definitions in the factory must reproduce FCUBS pricing/fee behavior; a product-behavior comparison matrix (rate tiers, fee triggers, rounding) is the standard validation artifact.
- **Cutover hygiene** — freeze legacy, migrate residual transactions (in-flight, back-value, un-reconciled items), archive, and keep the legacy system queryable for the statutory history window.

---

## 9. The Banking Context: Working with These Data Models

### 9.1 Data Integration Patterns

**From FCUBS to the warehouse (ETL/ELT).** The classic pattern: extract from the shipped views and accounting logs (`ACVW_ALL_AC_ENTRIES`-style views, `ACTB_DAILY_LOG`/`ACTB_HISTORY_LOG`, GL trial-balance tables, customer master) into a **banking data warehouse model** — BIAN/BDW/FSLDM/OFSDFM shape (see `data_models_banking_insurance_guide.md` §2). Hard rules:

- Filter on `AUTH_STAT = 'A'` and `REC_STAT` — unauthorized rows are drafts.
- Respect value date vs booking date semantics in every fact table.
- Use incremental extraction on `MAKER_DT_STAMP`/`CHECKER_DT_STAMP` or the EOD tables; account for **back-value and reversal entries** in balance facts (net vs gross).
- Treat the bank's customization (UDFs, country tables) as first-class extract scope — inventory it before designing the mappings.

**From OBMA to everything (real-time).** The pattern inverts: instead of polling tables, **subscribe to the event stream** (outbox-published domain events) for the data lake, analytics, and downstream systems; use the OpenAPI services for on-demand reads. The event payloads carry the business facts; the read models (§7.4) carry the aggregated views; the data lake is built by event replay into object storage/OFSAA (the OBMA guide §8.4).

**The coexistence integration** (FCUBS + OBMA in parallel, §8.2): the customer 360 and the balance picture span both systems — reconcile daily via a comparison job, and build the consolidated view in the warehouse/read-model layer, not in either core.

### 9.2 Regulatory Reporting Data (MAS, BCBS 239, Data Residency)

- **MAS context (Singapore)** — regulatory returns (MAS 610/649, ABS reporting, large-exposure, liquidity (MAS 649) and capital frameworks) are sourced from the core's GL and product/account data; the bank's regulatory reporting platform consumes FCUBS EOD output or OBMA events per the design (see `financial_risk_compliance_systems_guide.md` §9 for the regulatory reporting system pattern).
- **BCBS 239 data lineage** — the risk-data aggregation rules demand **end-to-end lineage** from source systems to regulatory reports. For FCUBS, lineage starts at the row level: maker/checker columns, entry logs, and GL postings are the audit source; the bank's lineage tooling must capture the FCUBS-to-warehouse-to-report path. For OBMA, lineage is *easier by construction*: the event stream is the audit trail — replaying events reconstructs any position and proves the path from business fact to report. (BCBS 239 coverage: `financial_risk_compliance_systems_guide.md` and `data_models_banking_insurance_guide.md` §5.1.)
- **Data residency** — with FCUBS, residency is a data-center question (on-prem/Exadata). With OBMA/OBCS on OCI, residency is a **cloud-region design question**: choose OCI regions and tenancy models that keep customer data in the required jurisdiction (e.g., Singapore region for MAS-licensed banks), and use the platform's tenancy/isolation options (§7.1) to enforce it. The practical residency checklist: which OCI region holds the system of record; where event streams and backups replicate (avoid cross-border replication of customer data unless permitted); where the data lake/OFSAA resides; and contractual data-processing terms with Oracle for SaaS (OBCS). MAS TRM expectations on data location and business continuity apply to both platforms (see `financial_risk_compliance_systems_guide.md` §11 and `core_banking_systems_guide.md` §9.5).

### 9.3 Reference Data (Currency, Rates, Holidays)

Shared reference data — the "static data" of FCUBS (`STTM_CURRENCY`, rate tables, holiday calendars, `STTM_TRN_CODE`) — is the same *content* in OBMA but owned by the **Common Core / reference-data services** (§7.1). For an architect, the practical questions are the same on both platforms: who owns the golden copy (core vs a bank-wide RDM tool), how rate/calendar changes propagate (EOD refresh in FCUBS; events in OBMA), and how reference data is versioned (date-effective rows in FCUBS; versioned definitions in the factory). See `data_models_banking_insurance_guide.md` §6.1 for the reference-data management pattern.

### 9.4 Limits Data

FLEXCUBE's limits and collateral module (enterprise limits, customer/product/facility limits, utilization) and OBMA's Limits service both implement the limit hierarchy and lifecycle of `banking_limits_domain_guide.md` (§4, §5): limit *definitions* vs *utilization* vs *check results*. Integration notes:

- **FCUBS**: limit utilization is computed from account balances/transactions in the core; expose via views/APIs; real-time pre-transaction checks are batch-adjacent in the classic core (see the limits guide §13 on real-time vs batch checking).
- **OBMA**: the Limits service owns limit data, checks limits in the transaction flow (pre/post), and emits `limit.breached` events; balances feed utilization via events/APIs from the Accounts/Transactions services.
- **Migration**: limit definitions and outstanding utilization map to the Limits service; utilization must be re-computed from migrated balances, not copied blindly.

### 9.5 Data for Analytics (Customer 360, Profitability, Risk)

The mart designs on top of these models follow the canonical patterns in `data_models_banking_insurance_guide.md` (§6.5 customer 360, §6.3 balance/position):

- **Customer 360 mart** — from FCUBS: join graph on `CUSTOMER_NO` (§3.4) flattened into a customer dimension with account/balance aggregates; from OBMA: the party read model + event-fed aggregates.
- **Profitability** — product × account × transaction facts with GL-based cost/income allocation; both platforms supply the raw facts (entries, balances, product attributes); the allocation logic lives in the mart/OFSAA.
- **Risk data** — credit risk (exposures from loans/limits/collateral), liquidity (cash-flow from dated instruments), market risk (from treasury/trading modules) — sourced from the core's product/transaction data and aggregated per the bank's risk data model (BCBS 239: see §9.2).

**A mart sketch** (star schema on top of either platform):

```text
fact_account_daily            dim_customer            dim_product
├── date_key                  ├── customer_key         ├── product_key
├── branch_key                ├── customer_no (CIF)    ├── product_code
├── customer_key              ├── type, segment        ├── type, currency
├── account_key               └── kyc_risk_rating      └── rate_tier
├── product_key
├── ledger_balance, available_balance, lien_amount
└── nbr_txns, txn_amount_sum          ← from ACTB_DAILY_LOG (FCUBS) or
                                         transaction events (OBMA)
```

The customer 360 mart is the `dim_customer` hub with `fact_*` satellites (accounts, transactions, limits); profitability adds cost/income facts from GL allocations; risk adds exposure/limit facts. The same star can be fed from FCUBS tables (ETL, §9.1) or from OBMA events (streaming into the lake, §7.3) — the mart is platform-agnostic, which is exactly the point of building it on a canonical model (`data_models_banking_insurance_guide.md` §2).

### 9.6 The Architect's Data Checklist

A practical checklist when working with FCUBS/OBMA data:

1. **Data governance** — who owns the golden copy of each data domain; the core is the system of record for customer/account/balance/transaction data; the warehouse is *derived*.
2. **Data dictionary** — for FCUBS: obtain and version the installed release's table descriptions, and inventory the bank's customizations (UDFs, country tables) before any extraction design. For OBMA: version the OpenAPI catalog and the event catalog (payload schemas) — they are the integration contract.
3. **Data lineage** — record source→mapping→target for every critical report (BCBS 239); for OBMA, lineage is the event path; for FCUBS, the table/column path through views.
4. **Test data** — synthetic data for testing: generate realistic, **masked** data (names, NRIC, account numbers) so non-production environments are safe; for FCUBS, masking applies at the extract/refresh layer (the core itself is production data); for OBMA, the event streams must also be masked in non-prod Kafka/streaming environments. Never copy production PII into test environments (PDPA/GDPR — see `data_models_banking_insurance_guide.md` §5.3).
5. **Reconciliation** — daily balance/GL reconciliation between core and warehouse (FCUBS), and between FCUBS and OBMA during coexistence (§8.2); automated, with difference thresholds and investigation workflow.
6. **Retention and archiving** — statutory history windows (transaction history, statements) drive archive design on both platforms; FCUBS history tables grow fast (partition by date); OBMA event streams must be retained/replayed per regulatory periods.

---

## 10. Worked Example: A Savings Account Opened and Funded

### 10.1 The FCUBS Walkthrough: Tables Touched

Scenario: *"Ms. Tan opens a savings account (product 'Savings Plus') at branch 001 and deposits SGD 10,000 in cash."* The tables touched, in order:

| Step | Table | What happens |
|---|---|---|
| 1. Customer exists | `STTM_CUSTOMER` | `CUSTOMER_NO` = 100234 (CIF record created earlier, `AUTH_STAT` = 'A') |
| 2. Product/class lookup | `STTM_PRODUCT`, `STTM_ACCOUNT_CLASS` | Product "Savings Plus" → account class → interest/fee/GL rules |
| 3. Account opened | `STTM_CUST_ACCOUNT` | Insert account row: `CUST_AC_NO` (new series), `CUST_NO` = 100234, `CCY` = SGD, `ACCOUNT_CLASS`, `ACC_STATUS` = 'NORM', opening balance 0; `AUTH_STAT` = 'U' → checker authorizes → 'A' (maker/checker stamps set) |
| 4. Cash deposit | teller → accounting engine | Teller transaction captured (teller module); posting engine generates **two entries** into `ACTB_DAILY_LOG`: DR cash/nostro GL 10,000 / CR customer account 10,000 (each row: branch, account, GL, amount, `TRN_CODE`, booking date, value date, maker/checker) |
| 5. Balances updated | balance tables (account balance components) | Ledger/available balance = 10,000; balance components updated in the balance tables (e.g., `ACTB_DAILY_BAL`-style period balances — check release dictionary) |
| 6. EOD | EOD batch | Entries moved from `ACTB_DAILY_LOG` to `ACTB_HISTORY_LOG`; interest accrual computed; trial balance produced; GL control accounts agree with the customer-account subsidiary ledger |

### 10.2 SQL Sketch (Simplified Structures)

Illustrative DDL — **not** the real dictionary (real tables have hundreds of columns); it shows the shape:

```sql
-- Customer master (CIF) — simplified
CREATE TABLE sttm_customer (
    customer_no      VARCHAR2(14)  PRIMARY KEY,
    branch_code      VARCHAR2(6)   NOT NULL,
    customer_type    VARCHAR2(1)   NOT NULL,   -- I=individual, C=corporate, B=bank, N=non-individual
    customer_name1   VARCHAR2(105),
    language         VARCHAR2(2),
    maker_id         VARCHAR2(30),  maker_dt_stamp  DATE,
    checker_id       VARCHAR2(30),  checker_dt_stamp DATE,
    mod_no           NUMBER(4),     rec_stat  VARCHAR2(1),  auth_stat VARCHAR2(1),  once_auth VARCHAR2(1)
);

-- Customer account master — simplified (verified column names in brackets)
CREATE TABLE sttm_cust_account (
    cust_ac_no       VARCHAR2(20)  PRIMARY KEY,
    branch_code      VARCHAR2(6)   NOT NULL,
    cust_no          VARCHAR2(14)  REFERENCES sttm_customer(customer_no),
    ccy              VARCHAR2(3)   NOT NULL,
    account_class    VARCHAR2(16)  NOT NULL,   -- links to STTM_ACCOUNT_CLASS / product
    acy_opening_bal  NUMBER(24,3)  DEFAULT 0,
    acc_status       VARCHAR2(4)   DEFAULT 'NORM',   -- NORM | OVER | ... 
    maker_id         VARCHAR2(30),  maker_dt_stamp  DATE,
    checker_id       VARCHAR2(30),  checker_dt_stamp DATE,
    mod_no           NUMBER(4),     rec_stat  VARCHAR2(1),  auth_stat VARCHAR2(1)
);

-- Product master — simplified (verified table)
CREATE TABLE sttm_product (
    product_code     VARCHAR2(4)   PRIMARY KEY,
    product_type     VARCHAR2(1),              -- CASA / DEP / LN ...
    currency         VARCHAR2(3)
    -- ... hundreds of product-parameter rows live in module parameter tables
);

-- Accounting entry log — simplified (verified table)
CREATE TABLE actb_daily_log (
    branch_code      VARCHAR2(6)   NOT NULL,
    ac_entry_sr_no   NUMBER(10)    PRIMARY KEY,
    cust_ac_no       VARCHAR2(20),             -- customer account leg
    gl_code          VARCHAR2(20),             -- GL leg
    trn_code         VARCHAR2(3)   NOT NULL,   -- STTM_TRN_CODE transaction code
    drcr             VARCHAR2(1)   NOT NULL,   -- D / C
    amount           NUMBER(24,3)  NOT NULL,
    booking_date     DATE          NOT NULL,
    value_date       DATE          NOT NULL,
    maker_id         VARCHAR2(30),  maker_dt_stamp  DATE,
    checker_id       VARCHAR2(30),  checker_dt_stamp DATE,
    auth_stat        VARCHAR2(1)   DEFAULT 'U'   -- U=UNAUTH, A=AUTH
);

-- The deposit as entries:
-- INSERT INTO actb_daily_log (branch_code, gl_code, trn_code, drcr, amount, booking_date, value_date) 
--   VALUES ('001', 'GL.CASH',   'CDP', 'D', 10000, SYSDATE, SYSDATE);  -- debit cash
-- INSERT INTO actb_daily_log (branch_code, cust_ac_no, trn_code, drcr, amount, booking_date, value_date)
--   VALUES ('001', '0012345678', 'CDP', 'C', 10000, SYSDATE, SYSDATE);  -- credit account
```

**Balance components** (simplified — the real tables store per-component values and history):

```sql
CREATE TABLE acct_balance_components (          -- simplified
    cust_ac_no     VARCHAR2(20)  NOT NULL,
    branch_code    VARCHAR2(6)   NOT NULL,
    ccy            VARCHAR2(3)   NOT NULL,
    component      VARCHAR2(20)  NOT NULL,      -- LEDGER | CLEARED | UNCLEARED
                                                -- | AVAILABLE | LIEN | HOLD
    amount         NUMBER(24,3)  DEFAULT 0,
    as_of_date     DATE          NOT NULL,
    PRIMARY KEY (cust_ac_no, branch_code, component, as_of_date)
);

-- Reading the customer's position (with authorization discipline):
SELECT c.cust_ac_no,
       SUM(CASE WHEN b.component = 'LEDGER'   THEN b.amount END) AS ledger,
       SUM(CASE WHEN b.component = 'AVAILABLE' THEN b.amount END) AS available,
       SUM(CASE WHEN b.component = 'LIEN'     THEN b.amount END) AS lien
FROM   sttm_cust_account c
JOIN   acct_balance_components b
  ON   b.cust_ac_no = c.cust_ac_no
WHERE  c.cust_no = '100234'
  AND  c.auth_stat = 'A'
  AND  b.as_of_date = (SELECT MAX(as_of_date) FROM acct_balance_components
                       WHERE cust_ac_no = c.cust_ac_no)
GROUP  BY c.cust_ac_no;
```

### 10.3 The OBMA Equivalent

The same scenario on OBMA touches **services, not tables**:

1. **Party service** — `POST /parties` (or the party already exists): party record for Ms. Tan with KYC status; response JSON carries `partyId`.
2. **Product Factory** — `GET /product-factory/products/savings-plus` (or via the product catalog service): the savings product definition (pricing tiers, fees, limits, GL mappings) is read as data.
3. **Accounts service** — `POST /accounts` with `{partyId, productId, currency: "SGD", branch}` → creates the account + the **agreement** (Ms. Tan's specific terms); persists in the Accounts service's schema; publishes **`account.opened`** (JSON: `accountId`, `partyId`, `productId`, `currency`, `status`, `timestamp`) to OCI Streaming via the **transactional outbox**.
4. **Transactions service** — `POST /transactions` (cash deposit) → validates, posts the entries (DR cash GL / CR account), updates balance components, publishes **`transaction.posted`** (JSON: `transactionId`, `accountId`, `amount`, `currency`, `drCr`, `valueDate`, `bookingDate`).
5. **Consumers react asynchronously**: Notifications sends the welcome SMS (`account.opened` consumer); the Customer 360 **read model** updates (event consumer projecting party + account + transaction facts); OFSAA/Data Lake ingests the events for analytics; the accounting/GL integration posts the GL entries (or the bank's finance ledger consumes the events).
6. **Audit** — the event stream + service audit records are the audit trail; no universal `AUTH_STAT` column — authorization is workflow-driven at the API layer (maker-checker can still be enforced per operation).

**The API payloads** (illustrative, simplified):

```json
POST /accounts  →  201
{
  "partyId": "pty_100234", "productId": "savings-plus",
  "currency": "SGD", "branch": "001"
}
{
  "accountId": "acct_88412001", "agreementId": "agr_5530",
  "status": "ACTIVE",
  "balances": { "ledger": 0.00, "available": 0.00 }
}

POST /transactions  →  201
{
  "accountId": "acct_88412001", "type": "CASH_DEPOSIT",
  "amount": 10000.00, "currency": "SGD",
  "valueDate": "2026-08-05", "bookingDate": "2026-08-05"
}
{
  "transactionId": "txn_7712003", "status": "POSTED",
  "entries": [
    { "drCr": "D", "glCode": "GL.CASH", "amount": 10000.00 },
    { "drCr": "C", "accountId": "acct_88412001", "amount": 10000.00 }
  ],
  "balancesAfter": { "ledger": 10000.00, "available": 10000.00 }
}

Event on the bus (outbox-published): account.funded
{
  "eventType": "account.funded", "eventVersion": "1.0",
  "aggregate": { "type": "ACCOUNT", "id": "acct_88412001" },
  "payload": { "transactionId": "txn_7712003", "amount": 10000.00,
               "currency": "SGD", "valueDate": "2026-08-05" }
}
```

**The contrast in one line** — FCUBS = write once to `ACTB_DAILY_LOG`, read everywhere from tables; OBMA = write to your service, publish the fact, let everyone project.

---

## 11. Glossary

| Term | Meaning |
|---|---|
| **CUBS** | Community shorthand for the classic FLEXCUBE UBS core engine/data model, informally expanded "Comprehensive Universal Banking System"; **not an Oracle-documented product name** (§2.1) |
| **FCUBS** | FLEXCUBE Universal Banking (Solutions) — Oracle's monolithic core banking platform (OFSS) |
| **CIF** | Customer Information File — FLEXCUBE's customer master module; the CIF number is the customer identifier (`CUSTOMER_NO`) |
| **STTM_** | Static Table, Maintenance prefix — master tables (`STTM_CUSTOMER`, `STTM_CURRENCY`, `STTM_CUST_ACCOUNT`, `STTM_PRODUCT`) |
| **STTB_** | Static Table, Buffer prefix — internal/working tables (`STTB_ACCOUNT`, `STTB_VALUE_DESC`) |
| **ACTB_** | Accounting Table, Buffer prefix — the accounting engine's tables (`ACTB_DAILY_LOG`, `ACTB_HISTORY_LOG`) |
| **DETB_ / LNTB_ / CLTB_** | Deposits / Lending / Collateral Table Buffer prefixes — module transaction/buffer tables |
| **FTMB_ / FTTB_** | Funds-transfer table prefixes — **prefix varies by release; check the dictionary** |
| **Product** | The parameterized definition (rates, fees, terms, GL mapping) from which accounts are created — configuration, not code |
| **Agreement** | The customer's contract for a product (terms, overrides) — a first-class entity in OBMA; folded into the account in FCUBS |
| **Account** | The product instance that carries the balance (FCUBS: `STTM_CUST_ACCOUNT`) |
| **Balance component** | The balance set per account: ledger, cleared, uncleared, available, lien, hold |
| **Chart of accounts (COA)** | The GL head hierarchy the core posts to; the GL master |
| **GL** | General ledger — the integrated double-entry ledger in FCUBS (trial balance, control accounts) |
| **Maker-checker** | The two-person authorization discipline enforced structurally by `AUTH_STAT`/maker/checker columns |
| **AUTH / UNAUTH** | Authorized / Unauthorized record states (`AUTH_STAT` = 'A' / 'U') — filter on 'A' when reading data |
| **Value date** | The date an entry takes economic effect (interest/balance) |
| **Booking date** | The date an entry was actually posted |
| **Back value / ghost entry** | An entry valued before its booking date; held in `ACTB_DAILY_GHOST_LOG` until its value date |
| **Reversal** | Offsetting entries that cancel an erroneous posting; originals remain for audit |
| **Event sourcing** | Making the event log the source of truth (OBMA uses outbox + replayable streams; verify per service) |
| **CQRS** | Command/query separation — writes to the owning service, reads from projections |
| **Outbox** | The transactional-outbox pattern: event written with the business transaction, then published (no lost events) |
| **Database-per-service** | Each microservice owns its data exclusively; no cross-service table access |
| **Read model** | A denormalized, event-projected view optimized for queries (e.g., customer 360) |
| **JSON schema** | The versioned payload contract of an OBMA API or event |
| **Data lineage** | The documented path from source data to report (BCBS 239) |
| **Test data / masking** | Synthetic, anonymized data for non-production environments (never copy production PII) |

---

## 12. References and Related Guides

### Sibling guides in this repository

- `oracle_banking_microservices_architecture_guide.md` — OBMA architecture, service catalog (§6), product factory (§7), event-driven design (§8), data layer (§5.3), migration (§14), OBMA-vs-vendors comparison (§15)
- `core_banking_systems_guide.md` — core data model (§7.8), ledger and double-entry (§7.3), product factory patterns (§7.4), transaction processing (§7.6), vendor landscape (§5.3 Oracle)
- `data_model_resource_book_guide.md` — Silverston universals: PARTY (§3.1, §9.1), PRODUCT/AGREEMENT/ACCOUNT/FINANCIAL TRANSACTION (§3.2, §9.2), temporal/classification patterns (§4)
- `data_models_banking_insurance_guide.md` — BIAN/BDW/FSLDM/OFSDFM (§2), balance and position modeling (§6.3), agreement lifecycle (§6.4), customer 360 (§6.5), event-driven data models (§6.7), BCBS 239 (§5.1)
- `bian_banking_architecture_guide.md` — BIAN service domains, Business Object Model (§6), semantic APIs (§9), BIAN vs core vendors (§15)
- `banking_limits_domain_guide.md` — the limits domain: hierarchy (§4), lifecycle (§5), limit-engine architecture (§12), real-time vs batch (§13)
- `financial_risk_compliance_systems_guide.md` — audit context, regulatory reporting systems (§9), BCBS 239 and MAS (§2, §9), cloud data management (§11)
- `temenos_guide.md` — the main competitor's data model (T24 applications/versions, MultiValue, jBASE, COB) for contrast
- `apache_fineract_guide.md` — the open-source relational core (accounting integration) for contrast
- `event_stream_processing_guide.md` — event-driven patterns: outbox, replay, partitioning, state (the general theory behind §7.3)

### External sources consulted (verified facts)

- Oracle FLEXCUBE Universal Banking documentation (docs.oracle.com) — module user guides: Customer Information File (CIF) User Manual, Customer Accounts (CASA) User Guide, General Ledger (FGL) User Manual, Database Practices guides (partitioning guidance citing `ACTB_DAILY_LOG`)
- Oracle FLEXCUBE REST API compilation/build documentation — `RATM_*`/`STTM_*` table pairs (`STTM_CUSTOMER`, `STTM_PRODUCT`, `STTM_TRN_CODE`, `CLTB_ACCOUNT_APPS_MASTER`)
- Oracle FLEXCUBE Web Service Development / extensibility documentation — `STTMS_CUST_PERSONAL`, `GWTB_AMEND_FIELDS`
- Oracle FLEXCUBE table-naming conventions and table references (practitioner references, e.g., VSL's FLEXCUBE notes) — `TM`/`TB`/`TW` convention, `STTB_VALUE_DESC`, `STTM_FIN_CYCLE`, `STTM_PERIOD_CODES`, `ACVW_ALL_AC_ENTRIES`
- Oracle Banking Microservices Architecture documentation family — `docs.oracle.com/en/industries/financial-services/microservices-common/` (platform foundation, common core)
- Oracle Database JSON capabilities — JSON Collection Tables, Autonomous JSON Database, MongoDB-compatible document APIs (for the OBMA data-store discussion)
- Oracle Financial Services Software (OFSS) product pages and Wikipedia — i-flex lineage, Oracle acquisition (2005)

> **Field-knowledge caveat (repeated):** FLEXCUBE table names and column sets vary by release and bank customization. Validate every table referenced in this guide against the installed release's data dictionary before quoting it in a design. The "CUBS" acronym expansion and the exact FT-prefix naming are flagged in-text as unverified.
