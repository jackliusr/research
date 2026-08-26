# Apache Fineract: A Comprehensive Guide to the Open-Source Core Banking Platform

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Open-Source Core Banking / Fintech Platforms — Architecture, API, Deployment, Vendor Comparison
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is Apache Fineract?](#1-what-is-apache-fineract)
   - 1.1 [Definition and Positioning](#11-definition-and-positioning)
   - 1.2 [Origins: From Mifos to Apache Fineract](#12-origins-from-mifos-to-apache-fineract)
   - 1.3 [License, Governance, and Community](#13-license-governance-and-community)
   - 1.4 [Who Uses Fineract and at What Scale](#14-who-uses-fineract-and-at-what-scale)
2. [Platform Capabilities and Design Principles](#2-platform-capabilities-and-design-principles)
   - 2.1 [Functional Capabilities at a Glance](#21-functional-capabilities-at-a-glance)
   - 2.2 [Design Principles](#22-design-principles)
3. [Platform Architecture](#3-platform-architecture)
   - 3.1 [Overall Shape: A Modular Monolith](#31-overall-shape-a-modular-monolith)
   - 3.2 [Layered Architecture](#32-layered-architecture)
   - 3.3 [Package Structure (Vertical Slices)](#33-package-structure-vertical-slices)
   - 3.4 [Technology Stack](#34-technology-stack)
   - 3.5 [Instance Types and Horizontal Scaling](#35-instance-types-and-horizontal-scaling)
4. [Functional Modules](#4-functional-modules)
   - 4.1 [Organization Module](#41-organization-module)
   - 4.2 [Client and Group Management](#42-client-and-group-management)
   - 4.3 [Loan Portfolio](#43-loan-portfolio)
   - 4.4 [Savings Portfolio](#44-savings-portfolio)
   - 4.5 [Deposits (Fixed and Recurring)](#45-deposits-fixed-and-recurring)
   - 4.6 [Accounting and General Ledger](#46-accounting-and-general-ledger)
   - 4.7 [Tax Management](#47-tax-management)
   - 4.8 [Charges and Fees](#48-charges-and-fees)
   - 4.9 [Collateral and Guarantees](#49-collateral-and-guarantees)
   - 4.10 [Reporting](#410-reporting)
   - 4.11 [Self-Service](#411-self-service)
   - 4.12 [Notifications and Events](#412-notifications-and-events)
   - 4.13 [Credit Bureau Integration](#413-credit-bureau-integration)
5. [The Microfinance Lending Model](#5-the-microfinance-lending-model)
   - 5.1 [Interest Calculation Methods](#51-interest-calculation-methods)
   - 5.2 [Repayment Schedules and Frequency](#52-repayment-schedules-and-frequency)
   - 5.3 [Group Lending (JLGs and Self-Help Groups)](#53-group-lending-jlgs-and-self-help-groups)
   - 5.4 [Grace Periods, Rescheduling, and Delinquency](#54-grace-periods-rescheduling-and-delinquency)
6. [The Fineract REST API](#6-the-fineract-rest-api)
   - 6.1 [API Philosophy and Documentation](#61-api-philosophy-and-documentation)
   - 6.2 [Base URL, Tenancy, and Versioning](#62-base-url-tenancy-and-versioning)
   - 6.3 [Authentication and Authorization](#63-authentication-and-authorization)
   - 6.4 [Common API Patterns](#64-common-api-patterns)
   - 6.5 [Commands, Status Codes, and Errors](#65-commands-status-codes-and-errors)
7. [Hands-On API Walkthrough](#7-hands-on-api-walkthrough)
   - 7.1 [Authentication](#71-authentication)
   - 7.2 [Create a Client](#72-create-a-client)
   - 7.3 [Define a Loan Product](#73-define-a-loan-product)
   - 7.4 [Create, Approve, and Disburse a Loan](#74-create-approve-and-disburse-a-loan)
   - 7.5 [Loan Repayment](#75-loan-repayment)
   - 7.6 [Savings Accounts](#76-savings-accounts)
8. [The Loan Product Lifecycle](#8-the-loan-product-lifecycle)
9. [Accounting Integration](#9-accounting-integration)
   - 9.1 [Double-Entry Bookkeeping and the Chart of Accounts](#91-double-entry-bookkeeping-and-the-chart-of-accounts)
   - 9.2 [Product-to-GL Account Mapping](#92-product-to-gl-account-mapping)
   - 9.3 [Automatic Journal Entries](#93-automatic-journal-entries)
   - 9.4 [Accrual Accounting and Financial Activity Mappings](#94-accrual-accounting-and-financial-activity-mappings)
   - 9.5 [Financial Reporting](#95-financial-reporting)
10. [Multi-Tenancy and Scalability](#10-multi-tenancy-and-scalability)
    - 10.1 [Tenant Model](#101-tenant-model)
    - 10.2 [Data Isolation](#102-data-isolation)
    - 10.3 [Scaling Patterns](#103-scaling-patterns)
11. [Deployment and Operations](#11-deployment-and-operations)
    - 11.1 [Deployment Options](#111-deployment-options)
    - 11.2 [Database Setup and Migrations](#112-database-setup-and-migrations)
    - 11.3 [Configuration](#113-configuration)
    - 11.4 [Security Hardening](#114-security-hardening)
    - 11.5 [Monitoring, Backup, and Upgrades](#115-monitoring-backup-and-upgrades)
12. [Fineract CN: The Cloud-Native Variant](#12-fineract-cn-the-cloud-native-variant)
13. [Ecosystem and Integrations](#13-ecosystem-and-integrations)
    - 13.1 [The Mifos Community](#131-the-mifos-community)
    - 13.2 [Payments and Mobile Money](#132-payments-and-mobile-money)
    - 13.3 [Other Integrations](#133-other-integrations)
    - 13.4 [Commercial Support and Implementers](#134-commercial-support-and-implementers)
14. [Use Cases and Case Studies](#14-use-cases-and-case-studies)
15. [Fineract vs. Commercial Core Banking Systems](#15-fineract-vs-commercial-core-banking-systems)
    - 15.1 [Comparison Table](#151-comparison-table)
    - 15.2 [When to Choose Fineract](#152-when-to-choose-fineract)
    - 15.3 [When Not to Choose Fineract](#153-when-not-to-choose-fineract)
16. [Fineract in a Bank's Architecture](#16-fineract-in-a-banks-architecture)
17. [Production Considerations](#17-production-considerations)
18. [Getting Started](#18-getting-started)
19. [References and Further Reading](#19-references-and-further-reading)


## 1. What Is Apache Fineract?

### 1.1 Definition and Positioning

**Apache Fineract** is an open-source core banking platform for financial services. It provides the "engine room" of a bank or microfinance institution (MFI): customer and account management, loan and savings portfolio management, deposits, interest calculation, double-entry accounting, reporting, and a comprehensive REST API — packaged as a single, self-contained application.

Fineract is designed to make robust banking technology openly available, lowering the barriers for institutions and innovators to reach underserved and unbanked populations. It is explicitly positioned as a **cloud-ready core banking system** that enables digital financial services for everyone — from microfinance institutions serving rural borrowers to digital banks serving millions of mobile customers.

Typical consumers of Fineract include:

- **Digital banks** (particularly in Africa and Asia) that need an API-first core at a fraction of the cost of a commercial license.
- **Microfinance institutions (MFIs)** — Fineract's historical home — including Grameen-style group lenders and self-help group (SHG) programs.
- **Fintech startups** building lending or savings products who do not want to build a ledger from scratch.
- **Community banks, credit unions, and savings cooperatives**.
- **Government and NGO financial-inclusion programs** (national IDs to financial services, subsidized lending schemes, mobile-money-backed microcredit).
- **SaaS providers** that run Fineract multi-tenant and rent banking functionality to many institutions.

Fineract is a **verified Digital Public Good** (registered with the Digital Public Goods Alliance), which matters for institutions that must or prefer to use open, auditable technology in aid-funded or government-backed financial-inclusion programs.

### 1.2 Origins: From Mifos to Apache Fineract

Fineract's lineage goes back to **Mifos**, a microfinance platform founded in **2005** by the **Grameen Foundation** together with the open-source community, and driven since then by the **Mifos Initiative** (a non-profit founded in 2010 to sustain and grow the Mifos platform). Mifos was built to serve the "bottom of the pyramid" — group-lending MFIs following the Grameen Bank model — and was deployed by hundreds of MFIs worldwide.

Key milestones:

| Year | Milestone |
|---|---|
| 2005 | Mifos founded by the Grameen Foundation and the open-source community |
| 2010 | Mifos Initiative (mifos.org) founded to steward the platform |
| 2011–2015 | Mifos X rewritten as a modern, API-driven platform (Java, REST) |
| Dec 2015 | Mifos Initiative contributes the Fineract backend code to the Apache Software Foundation (incubation) |
| Apr 2017 | **Apache Fineract graduates from the Apache Incubator to a top-level project (TLP)** |
| 2018–2021 | Fineract 1.x matures; Fineract CN (cloud-native microservices rewrite) developed in parallel |
| 2022–2026 | Fineract 1.x continues as the production line (releases through 1.15.0, mid-2026); Fineract CN is archived; PostgreSQL becomes the officially supported database |

The current project family:

- **Apache Fineract 1.x** — the production platform: a Spring Boot monolith with a REST API. This is what "Fineract" means in production today.
- **Apache Fineract CN** — a cloud-native re-architecture (microservices + Kafka) attempted from ~2017 onward. Development has effectively ended and the repositories are archived; the community recommends Fineract 1.x for production. See [Section 12](#12-fineract-cn-the-cloud-native-variant).
- **Mifos X / community apps** — front-end and ecosystem projects built on top of Fineract (see [Section 13](#13-ecosystem-and-integrations)).

### 1.3 License, Governance, and Community

- **License:** Apache License 2.0 — permissive, allows commercial use, modification, and redistribution with attribution; no copyleft obligations.
- **Governance:** Apache-style "community over code" — a project management committee (PMC), committers elected by merit, open development on public mailing lists, and consensus-based decision making. No single vendor controls the roadmap.
- **Project home:** [fineract.apache.org](https://fineract.apache.org) · [github.com/apache/fineract](https://github.com/apache/fineract) · dev mailing list `dev@fineract.apache.org` · JIRA issue tracker · Matrix chat (`#apache-fineract-home:matrix.org`).
- **Leadership and contributors:** development is led by the **Mifos Initiative** and **Connexta** (a long-standing engineering contributor) together with a broad community of individual contributors from institutions, consultancies, and fintechs.
- **Release cadence:** active and frequent; 1.x releases have been shipping continuously (1.8, 1.9 … 1.15.0 as of mid-2026), each with new APIs, features, and security fixes.

### 1.4 Who Uses Fineract and at What Scale

Fineract powers a wide range of production institutions:

- **Digital banks in Africa and Asia** — most prominently **TymeBank** (South Africa, rebranded **GoTyme Bank** in 2026 as part of Tyme Group), whose digital banking platform is widely reported to be built on Fineract; Tyme Group serves ~20 million customers across its markets. Several other African and Asian digital banks and "neo-banks" run Fineract-based cores.
- **Microfinance institutions** — hundreds of MFIs worldwide, from Grameen-model group lenders to urban microfinance banks, historically in India, Bangladesh, Africa, Latin America, and Southeast Asia.
- **Government financial-inclusion programs** and NGO programs (the DPGA registration reflects heavy use in aid-funded digital-financial-services programs).
- **Fintech lenders and savings startups** using Fineract's loan/savings engine as their back office.
- The Mifos community reports **20+ million end customers** served through Fineract-based deployments (SourceForge project description for Mifos).

The platform is engineered to serve millions of customers and high transaction volumes when deployed with appropriate infrastructure — vertical scaling of the monolith plus horizontal scaling via read replicas and dedicated batch instances (see [Section 10](#10-multi-tenancy-and-scalability)).


## 2. Platform Capabilities and Design Principles

### 2.1 Functional Capabilities at a Glance

| Capability | Description |
|---|---|
| Client management | Individuals, corporate entities, client identifiers (national ID, passport), documents, KYC data tables, client status workflow (pending → active → closed) |
| Group management | Centers, groups, joint liability groups (JLGs), self-help groups; group-level loans and savings |
| Loan portfolio | Loan products, loan accounts, disbursement, repayment scheduling, interest (flat/declining, daily/monthly), fees, penalties, guarantors, collateral, rescheduling, transfers, delinquency, write-offs |
| Savings portfolio | Savings products and accounts, deposits, withdrawals, interest posting, overdrafts, dormancy, account closure |
| Deposits | Fixed deposits and recurring deposits with terms, interest rates, and maturity handling |
| Accounting | Chart of accounts, GL accounts, journal entries, accounting rules, product-to-GL mappings, financial activity mappings, accrual accounting |
| Tax | Tax components, tax groups, tax mappings applied to fees/interest |
| Charges & fees | Loan charges (disbursement, installment, overdues), savings charges, fee types and payment modes |
| Collateral & guarantees | Collateral registry per loan, guarantee tracking |
| Reporting | Client, loan, savings, accounting, and financial reports (balance sheet, income statement, trial balance), XBRL export, Pentaho-based report designer integration |
| Self-service | REST endpoints for mobile apps and customer self-service (balance inquiry, mini-statements, transfers) |
| Notifications & events | In-platform notifications, webhooks (hooks), business events to Kafka/JMS brokers, SMS/email via plugins |
| Administration | Offices/branches, staff, currencies, holidays, working days, roles & permissions (RBAC), maker-checker |
| Extensibility | Data tables (custom fields on any entity), hooks, plugins (JARs), custom REST modules |

### 2.2 Design Principles

The official documentation frames Fineract's architecture around a set of explicit principles that are worth understanding before touching the code or the API:

1. **Practically-RESTful API.** All functionality is exposed via a JSON REST API. It is "practically RESTful" rather than purist: stateless (no server-side session state → easy horizontal scaling), resource-oriented, and consistent (HTTP verbs, HTTP status codes, uniform JSON shapes).
2. **Multi-tenanted at the core.** One deployment can serve many institutions, each isolated in its own database/schema (see [Section 10](#10-multi-tenancy-and-scalability)). This makes both single-tenant bank deployments and SaaS offerings first-class.
3. **Extensible per tenant.** Beyond core tables, each tenant can add custom fields via *data tables*, attach documents, and register hooks — without code changes.
4. **Command/query separation.** State-changing requests are captured as *commands*: persisted for audit, re-runnable in maker-checker workflows, and executed through an object-oriented domain layer; read queries stay in the data paradigm. This is a lightweight, pragmatic CQRS — not a full event-sourced architecture.
5. **Maker-checker (four-eyes).** Any state-changing API can require a second user to approve the command before it takes effect. Fineract uses this as its control mechanism for sensitive operations (e.g., large disbursements).
6. **Fine-grained access control.** Every API endpoint carries a permission; administrators can build roles from individual permissions and assign them to users.
7. **Auditability.** Because commands are persisted and every state change flows through the command pipeline, the platform naturally produces an audit trail — a property regulators look for (relevant to MAS licensing discussions in [Section 17](#17-production-considerations)).


## 3. Platform Architecture

### 3.1 Overall Shape: A Modular Monolith

Fineract 1.x is a **monolithic Spring Boot application** that packages the entire platform into a single deployable artifact — a Spring Boot executable JAR (recommended) or a WAR for legacy servlet containers. The codebase is organized into *vertical slices* (business modules) rather than strict layers, so although it is deployed as one process, it is designed to be modular internally — and the community is incrementally splitting pieces into separate modules/plugins.

Why a monolith is the right default for Fineract's audience:

- **Operational simplicity** — one artifact, one database schema per tenant, one process to scale.
- **Transactional integrity** — loan disbursement + GL posting + interest booking happen in one database transaction, which is essential for money movement.
- **Lowest TCO** for MFIs and digital banks that do not need microservice-level elasticity.
- The monolith still scales horizontally through *instance types* (read/write/batch) — see 3.5.

### 3.2 Layered Architecture

```
Clients (Mifos X web app, mobile apps, custom channels)
        ↓
REST API layer (Jersey/JAX-RS + JSON) → Command layer (validation, maker-checker)
        ↓
Service layer → Domain layer (OO model) → Persistence (Spring Data JPA, Flyway/Liquibase)
        ↓
Databases — tenant DBs (PostgreSQL/MySQL/MariaDB) + tenants registry DB
```

The layers in more detail:

- **REST API layer:** JAX-RS resources (Jersey) in each module's `api` package — e.g., `ClientsApiResource`, `LoanAccountsApiResource`. Handles HTTP verbs, query parameters, JSON (de)serialization via Google Gson, and permission checks.
- **Command layer:** state-changing requests become `JsonCommand`s routed to `CommandHandler`s; supports validation, maker-checker gating, and audit persistence (the `m_portfolio_command_source` table records every command).
- **Service layer:** read services (queries, DTOs) and write services (business logic orchestration) per module.
- **Domain layer:** object-oriented model of banking concepts — loan accounts, savings accounts, GL accounts, journal entries, interest calculations.
- **Persistence layer:** Spring Data JPA repositories; each tenant's schema contains the platform tables plus tenant-specific *data tables* (custom fields).

### 3.3 Package Structure (Vertical Slices)

Source: `fineract-provider/src/main/java/org/apache/fineract` — organized by functional area, each with a common internal structure (`api`, `handler`, `service`, `domain`, `data`, `serialization`):

| Top-level package | Functional area |
|---|---|
| `accounting` | GL accounts, journal entries, accounting rules, financial activity mappings |
| `portfolio` | `charge` (fees), `client` (clients, groups, identifiers, documents), `fund`, `loanaccount` (loans), `savings` (savings + deposits), `self` (self-service) |
| `organization` | Offices, staff, currencies, holidays, working days, tellers |
| `tax` | Tax components, tax groups, tax mappings |
| `useradministration` | Users, roles, permissions, maker-checker settings |
| `infrastructure` | Data tables, hooks, document management, audit, jobs, configuration |

### 3.4 Technology Stack

| Component | Technology |
|---|---|
| Language | Java (JDK 17+) |
| Application framework | Spring Boot, Spring Framework, Spring Security |
| REST layer | JAX-RS / Jersey; JSON via Google Gson |
| Persistence | Spring Data JPA (EclipseLink or Hibernate); Flyway/Liquibase migrations |
| Databases | **PostgreSQL (officially supported)**; MySQL and MariaDB support exists but is deprecated (planned removal); MariaDB4j used for integration tests |
| Build | Gradle (multi-module: `fineract-provider`, `fineract-client`, docs, etc.) |
| Eventing | Business events to Apache Kafka or JMS (ActiveMQ) brokers, webhooks |
| Reporting | SQL-defined report templates; Pentaho (now a plugin, see 13.3); XBRL for regulatory-style export |
| Deployment | Spring Boot fat JAR / WAR, Docker image (`apache/fineract`), Kubernetes, Tomcat |
| Docs/API spec | Asciidoc docs + OpenAPI (Swagger) spec; generated client SDKs |

**Database note:** since FSIP-9 ("Standardize on PostgreSQL"), PostgreSQL is the officially supported database and the default in CI and the Docker image; MySQL/MariaDB remain functional but deprecated. New deployments should use PostgreSQL.

### 3.5 Instance Types and Horizontal Scaling

For high load, Fineract instances can be started in three roles (configurable via Spring profiles / environment):

| Capability | Read instance | Write instance | Batch instance |
|---|---|---|---|
| Read-only DB connection | Yes | No | No |
| Read APIs | Yes | Yes | No |
| Write APIs | No | Yes | No |
| Batch job APIs (CoB, interest posting) | No | No | Yes |
| Receives business/hook events | No | Yes | No |
| Sends business/hook events | No | Yes | Yes |
| Runs Liquibase migrations on startup | No | Yes | No |

A single-instance deployment enables all three roles (the default). A scaled deployment typically runs **1 write instance + 1+ batch instances + N read instances** pointing at read replicas, so read traffic and close-of-business (CoB) jobs do not contend with the write path. Master/worker batch instance topologies are supported for CoB job distribution. This is the officially documented scale-out story for the monolith.


## 4. Functional Modules

### 4.1 Organization Module

The organization of the institution itself: **offices** (branch hierarchy — loans and savings roll up by office), **staff** (loan officers, tellers), **currencies** (per-tenant base currency + multi-currency support), **holidays** and **working days** (used in repayment-date calculations), **tellers** (cash management), and **advanced accounting options**. Holiday calendars and working-day definitions directly affect schedule generation — a feature microfinance operators rely on for weekly and irregular repayment patterns.

### 4.2 Client and Management

- **Clients:** individuals and corporates with lifecycle states (`pending`, `active`, `closed`, `rejected`, `withdrawn`), client identifiers (national ID, passport, mobile number), client documents, client images, and client-level custom data tables (e.g., KYC/onboarding questionnaires).
- **Groups and centers:** hierarchical structures — centers contain groups; groups contain clients. Groups can guarantee loans (JLGs), and group accounts can carry loans or savings with group-level liability.
- **Family/relationship links** between clients (e.g., next of kin, account nominees) and **client surveys**.

### 4.3 Loan Portfolio

Fineract's richest module, purpose-built for microfinance and small-balance lending:

- **Loan products** define templates: currency, principal range, interest method and rate, repayment frequency and schedule, amortization, fees/penalties, accounting mappings, and workflow rules.
- **Loan accounts** instantiate a product for a client or group with a lifecycle: submitted → approved → disbursed → (repaying) → closed, with `rejected`/`withdrawn` states.
- **Loan transactions:** disbursement, repayments, interest, fees, penalties/overdue charges, waivers (interest/fee), write-offs, adjustments, reversals, and transfers between loan accounts.
- **Repayment scheduling:** equal-installment or declining-balance amortization; daily/weekly/monthly/irregular schedules; due-date-based scheduling honoring holidays and working days.
- **Delinquency:** overdue tracking, arrears aging (portfolio at risk), delinquency buckets and reports.
- **Rescheduling & restructuring:** modify interest rates, extend terms, rewrite repayment schedules.
- **Guarantors and collateral** per loan (see 4.9).
- **Loan accounts payable / transfers**, and **loan re-payment allocation rules** (which portion of a payment goes to fees, interest, principal — configurable order).

### 4.4 Savings Portfolio

- **Savings products** (deposit product templates): minimum/maximum balances, interest rate and calculation (daily balance, monthly posting), interest compounding/postings, overdraft facility, dormancy rules, fees, and accounting mappings.
- **Savings accounts:** lifecycle (submitted → approved → active → closed), **deposits and withdrawals**, **interest calculation and posting** (daily accrual, monthly/quarterly posting), **overdrafts** (with limit and interest), **dormancy/inactive** handling, transfers between accounts, and account-holder statements.
- **Group savings** and **savings linked to loan repayments** (e.g., compulsory savings for group lenders).

### 4.5 Deposits (Fixed and Recurring)

- **Fixed deposits:** a lump sum locked for a **term** at a **rate**; interest calculation options (simple/compounding), premature closure rules, maturity instructions (renew, transfer to savings, pay out), and maturity processing.
- **Recurring deposits:** periodic installment deposits toward a maturity goal, with interest computed on the running balance; maturity and closure processing.
- Both share the savings infrastructure (approval/activation workflows, accounting, charges) while adding term/maturity semantics.

### 4.6 Accounting and General Ledger

- **Chart of accounts** per tenant, GL account types (asset, liability, equity, income, expense) and usage classes (detailed, header, cash).
- **Journal entries** — the atomic record of every accounting movement, with the classic double-entry invariant (debits = credits).
- **Accounting rules** define how transaction types map to journal entry postings.
- **Product-to-GL mapping** — every loan/savings/deposit product maps its financial events (disbursement, repayment, interest income, fee income, write-offs, etc.) to specific GL accounts, so the GL is updated automatically as portfolio events occur.
- **Financial activity mappings** — map conceptual activities (e.g., "interest received", "write-off expense", "overpayment liability") to GL accounts, used to generate income statements and balance sheets consistently.
- **Accrual accounting** — periodic (monthly) accrual of interest income and expense via batch jobs, with deferred/receivable accounts, for institutions that report on an accrual rather than cash basis.
- Integration with **external GL systems** by exporting journal entries or consuming the journal-entry API.
- See [Section 9](#9-accounting-integration) for the deeper walkthrough.

### 4.7 Tax Management

Tax components (e.g., VAT rates), tax groups (bundles of components), and tax mappings that apply taxes to charges and interest in supported jurisdictions — used mainly in markets where loan fees or interest attract VAT/withholding-type taxes (e.g., parts of Africa and Asia).

### 4.8 Charges and Fees

- **Charge products** reusable across loan/savings/deposit products: disbursement charges, installment charges, overdue/penalty charges, withdrawal fees, annual fees, etc.
- **Charge payment modes** (regular vs. penalty-style), **charge time** (disbursement, installment due, overdue, specified date), **calculation types** (flat or percentage), and **waivers**.
- Charges appear in repayment schedules and generate fee income in the GL.

### 4.9 Collateral and Guarantees

- **Collateral:** per-loan collateral registry — collateral types, values, and documents (e.g., land title, gold, livestock in classic MFI practice), tracked from approval through closure.
- **Guarantees:** client or group guarantees backing loans — including **group/joint-liability guarantees** where group members guarantee each other's loans (core to JLG lending, see 5.3), and third-party guarantor tracking.

### 4.10 Reporting

- **SQL-defined report templates** registered in the platform (`m_reports` / `m_report_parameter`): client reports, loan reports (portfolio at risk, arrears, disbursement), savings reports, accounting reports (trial balance, balance sheet, income statement, GL accounts), and custom financial reports.
- **Report API** (`GET /reports`, `POST /reports/{id}`) returns JSON or CSV; supports runtime parameters.
- **XBRL** export for external/regulatory-style reporting.
- **Pentaho reporting** integration (historically a bundled BI engine; in current releases the Pentaho report runner is a plugin — see 13.3) for pixel-perfect, branded reports.
- **Batch jobs** produce scheduled outputs (e.g., daily delinquency reports, interest postings) — the same job framework that drives CoB processing.

### 4.11 Self-Service

REST endpoints under `/self/*` designed for customer-facing channels: balance inquiry, mini-statements, product catalogs, and (where enabled) transfers. This is what mobile banking apps and USSD channels talk to — separate from the back-office APIs, with restricted permissions and its own authentication considerations (often paired with OAuth2 for app users; see 6.3).

### 4.12 Notifications and Events

- **In-platform notifications** to users (alerts on approvals, due repayments).
- **Webhooks ("hooks")** — HTTP callbacks triggered by template events (e.g., loan repayment received), letting external systems subscribe without polling.
- **Business events** — publish domain events to **Apache Kafka** or **JMS brokers** (e.g., ActiveMQ); disabled by default, enabled via configuration. Useful for feeding analytics, CRM, or payment hubs.
- **SMS/email** delivery via gateway plugins.

### 4.13 Credit Bureau Integration

No built-in bureau engine, by design: Fineract exposes the portfolio data and APIs, and integrations with credit bureaus (pull credit reports, push repayment histories) are implemented as external services/plugins calling the Fineract API — a common pattern in African and Asian deployments where bureaus such as TransUnion, Experian, or CRIF operate.


## 5. The Microfinance Lending Model

Fineract's lending engine encodes a decade-plus of microfinance domain knowledge. Understanding these concepts matters even for digital banks, because they explain the product parameterization the API exposes.

### 5.1 Interest Calculation Methods

| Method | How interest accrues | Typical use |
|---|---|---|
| **Flat (fixed/flat rate)** | Interest is calculated on the **original principal** for the full term; the total interest is spread evenly across installments. Effective APR is higher than the nominal rate. | Classic microfinance (Grameen-model) products |
| **Declining balance** | Interest is calculated on the **outstanding principal** each period, so interest decreases as principal is repaid. | Standard retail loans, most digital-bank products |

Within each method, Fineract supports **interest periods** (monthly vs. daily) and **compounding options** (compounding on the whole principal vs. interest only, or no compounding), plus **interest recalculation** (on repayment, rescheduling, or arrears) and **interest capitalization**. The interest basis can be 360 or 365 days, and **interest days in year** settings follow market convention.

### 5.2 Repayment Schedules and Frequency

- **Amortization:** equal installments (annuity-style, fixed EMI) or equal principal with declining interest component.
- **Repayment frequency:** daily, weekly, monthly, or **irregular/adhoc schedules** (custom installment dates — common for agricultural loans tied to harvest cycles).
- **Repayment start date** (grace before first installment), **grace on principal**, **grace on interest**, and **moratorium periods**.
- Schedules are generated respecting the tenant's **holiday calendar and working days**, with configurable handling of due dates that fall on holidays.
- **Prepayment:** full or partial prepayment with configurable **prepayment penalty** and recalculated schedules.
- **Overdue/arrears:** penalties per day of delay, arrears aging buckets, and delinquency reports.

### 5.3 Group Lending (JLGs and Self-Help Groups)

- **Joint Liability Groups (JLGs):** a group of clients (e.g., 5–10) who cross-guarantee each other's loans. Fineract supports group-level loan accounts where the group is the borrower, with individual loan tracking within the group and the group's members as guarantors.
- **Self-Help Groups (SHGs):** savings-first groups that accumulate internal funds and then lend to members — supported via group savings accounts feeding group loans.
- **Centers:** higher-level gatherings (e.g., weekly village center meetings) that aggregate several groups for operational efficiency — a core Grameen operational concept Fineract models natively.
- This native group-lending capability is one of Fineract's clearest differentiators versus commercial retail cores (see [Section 15](#15-fineract-vs-commercial-core-banking-systems)).

### 5.4 Grace Periods, Rescheduling, and Delinquency

- **Grace:** principal-only grace, interest-only grace, or both; applied at product or account level.
- **Rescheduling/restructuring:** extend the term, change the rate, or rewrite the schedule (with configurable treatment of accrued interest), while preserving the audit trail of the original and new schedules.
- **Write-offs:** partial or full write-off with GL postings to the write-off expense account (via financial activity mappings).
- **Delinquency lifecycle:** due-date-based arrears tracking → delinquency bucket classification (e.g., 30/60/90+ days) → collection follow-up via batch jobs and reports → write-off recommendation.
- **Loan transfers** between offices/staff and **loan account transfers between clients** are supported.


## 6. The Fineract REST API

### 6.1 API Philosophy and Documentation

Fineract exposes *everything* through a single REST API — there is no hidden internal UI protocol; the Mifos X web app is itself just an API client. The API was historically defined in RAML and is now published as an **OpenAPI/Swagger specification**, with a live Swagger UI on every instance and generated client SDKs (`fineract-client` for Java; Swagger Codegen output for other languages).

- Live spec (running instance): `/fineract-provider/api-docs` (JSON) and Swagger UI at `/fineract-provider/swagger-ui/` (older releases: `/fineract-provider/api/v1/swagger-ui.html`).
- Sandbox instance operated by the community: `sandbox.mifos.community` (also hosts the Swagger spec at `/fineract-provider/swagger-ui/fineract.yaml`).

### 6.2 Base URL, Tenancy, and Versioning

```
https://<host>:8443/fineract-provider/api/v1/<resource>
```

- Default HTTPS port is **8443** (HTTPS is enforced by default — see 11.4).
- The API is versioned via the `/v1` path segment; backward compatibility is a stated project concern (documented in the `api-backward-compatibility` chapter).
- **Tenant selection:** every request carries the tenant identifier — as an HTTP header:

```
Fineract-Platform-TenantId: default
```

  or as a path/URL element (supported by some tooling). The platform resolves the tenant identifier against the **tenants database** (`fineract_tenants` → `tenant_server_connections`), which maps tenant identifiers to their dedicated database connection details (OLTP and optional reporting connections). See [Section 10](#10-multi-tenancy-and-scalability).

### 6.3 Authentication and Authorization

| Scheme | Details |
|---|---|
| **Basic auth** (default) | `Authorization: Basic base64(user:password)`; default bootstrap user `mifos` / `password` (change immediately in production). Every request is authorized against the user's **role permissions** (RBAC) — each API carries a permission name. |
| **OAuth 2.0** (since 1.8) | Enable via `FINERACT_SECURITY_OAUTH_ENABLED=true` (and optionally disable basic auth with `FINERACT_SECURITY_BASICAUTH_ENABLED=false`); the platform validates tokens against an OAuth2 resource server (e.g., Keycloak) configured via `FINERACT_SERVER_OAUTH_RESOURCE_URL`. Typical for customer-facing/self-service apps. |
| **Two-factor (2FA)** | Optional TOTP-style two-factor challenge for back-office users. |
| **API keys / tokens** | In addition to basic auth, self-service and app-to-app flows use bearer tokens obtained via the login endpoint (`POST /authentication` returns a token usable in the `Authorization: Bearer` header). |

**Permission model:** users belong to roles; roles aggregate fine-grained permissions (`READ_CLIENT`, `CREATE_LOAN`, `APPROVE_LOAN`, …). The maker-checker principle can be layered on top: certain commands can be configured so that one user creates them and another approves them.

### 6.4 Common API Patterns

- **Resource naming:** plural REST resources — `/clients`, `/loans`, `/loans/{loanId}/transactions`, `/savingsaccounts`, `/fixeddepositaccounts`, `/recurringdepositaccounts`, `/charges`, `/offices`, `/glaccounts`, `/journalentries`, `/reports`, `/datatables`, `/hooks`, `/jobs`, `/self/clients`, …
- **Pagination & filtering:** query parameters — `paged=true&offset=0&limit=50&orderBy=id&sortOrder=ASC`; list endpoints support `fields=…` projection and filtering parameters per resource.
- **Dates & numbers:** Fineract requires explicit formats — `dateFormat=dd MMMM yyyy&locale=en_GB` (e.g., `"01 January 2026"`) and `locale` for money amounts. Omitting these is the most common integration mistake.
- **Idempotency:** recent releases add idempotency-key support for sensitive write operations (a genuinely useful feature for payment/retry integrations).
- **Business date:** the platform supports a configurable **business date** for CoB-style processing, decoupling "today" from the processing date.

### 6.5 Commands, Status Codes, and Errors

State changes are triggered with `?command=` on the resource — Fineract's signature pattern:

```
POST /loans/{loanId}?command=approve|disburse|waiveinterest|writeoff|close|...
POST /loans/{loanId}/transactions?command=repayment|waivecharge|...
POST /savingsaccounts/{savingsId}?command=activate|close|...
POST /savingsaccounts/{savingsId}/transactions?command=deposit|withdrawal|...
```

**HTTP status codes:** `200 OK` (query/update), `202 Accepted` (command accepted, possibly maker-checker pending), `400 Bad Request` (validation), `401 Unauthorized`, `403 Forbidden` (permission), `404 Not Found`, `405`, `409 Conflict` (state-machine violation, e.g., disbursing an unapproved loan), `500 Internal Server Error`.

**Error body shape** (consistent across the API):

```json
{
  "developerMessage": "Validation errors occurred.",
  "defaultUserMessage": "Validation errors occurred.",
  "userMessageGlobalisationCode": "validation.msg.validation.errors.exist",
  "errors": [
    {
      "userMessageGlobalisationCode": "validation.msg.loan.disbursement.not.allowed",
      "defaultUserMessage": "Loan disbursement is not allowed in current state.",
      "parameterName": "loanId",
      "value": "42"
    }
  ]
}
```

**Maker-checker responses:** when enabled for a command, the response carries an `officeId` + `commandId` and the change is staged until a checker approves it (`POST /makercheckers/{commandId}`).


## 7. Hands-On API Walkthrough

The flow below mirrors the classic Fineract "hello world": login → client → loan product → loan → disburse → repay. All examples target `https://localhost:8443` (Docker quickstart, [Section 18](#18-getting-started)) with the default tenant and bootstrap credentials.

### 7.1 Authentication

```bash
# Basic auth (default user mifos / password, base64: bWlmb3M6cGFzc3dvcmQ=)
curl --insecure -u mifos:password \
  -H 'Fineract-Platform-TenantId: default' \
  https://localhost:8443/fineract-provider/api/v1/authentication
```

Returns an `accessToken` for bearer-token flows and the user's role/permission summary.

### 7.2 Create a Client

```bash
curl --insecure -u mifos:password \
  -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST https://localhost:8443/fineract-provider/api/v1/clients \
  -d '{
    "officeId": 1,
    "firstname": "Amina",
    "lastname": "Diallo",
    "legalFormId": 1,
    "dateFormat": "dd MMMM yyyy",
    "locale": "en_GB",
    "activationDate": "01 January 2026"
  }'
# → { "clientId": 1, "officeId": 1, "resourceId": 1 }
```

New clients are created directly in `active` state when an activation date is supplied.

### 7.3 Define a Loan Product

```bash
curl --insecure -u mifos:password \
  -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST https://localhost:8443/fineract-provider/api/v1/loanproducts \
  -d '{
    "name": "Micro Loan - Declining",
    "shortName": "MDL",
    "currencyCode": "KES",
    "digitsAfterDecimal": 2,
    "principal": 50000,
    "numberOfRepayments": 12,
    "repaymentEvery": 1,
    "repaymentFrequencyType": 2,
    "interestRatePerPeriod": 2.0,
    "interestRateFrequencyType": 2,
    "amortizationType": 1,
    "interestType": 1,
    "interestCalculationPeriodType": 1,
    "transactionProcessingStrategyCode": "mifos-standard-strategy",
    "accountingRule": 2
  }'
```

Notes: `amortizationType: 1` = equal installments, `interestType: 1` = declining balance; `accountingRule: 2` = accrual accounting with GL mappings (see [Section 9](#9-accounting-integration)).

### 7.4 Create, Approve, and Disburse a Loan

```bash
# Create loan account for client 1 against product 1
curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST https://localhost:8443/fineract-provider/api/v1/loans \
  -d '{
    "clientId": 1, "productId": 1, "loanTermFrequency": 12,
    "loanTermFrequencyType": 2, "numberOfRepayments": 12,
    "repaymentEvery": 1, "repaymentFrequencyType": 2,
    "principal": 50000, "loanType": "individual",
    "expectedDisbursementDate": "05 January 2026",
    "dateFormat": "dd MMMM yyyy", "locale": "en_GB"
  }'
# → { "loanId": 1 }

# Approve → 202 Accepted with maker-checker if enabled
curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST "https://localhost:8443/fineract-provider/api/v1/loans/1?command=approve" \
  -d '{"approvedOnDate": "06 January 2026", "dateFormat": "dd MMMM yyyy", "locale": "en_GB"}'

# Disburse
curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST "https://localhost:8443/fineract-provider/api/v1/loans/1?command=disburse" \
  -d '{"actualDisbursementDate": "07 January 2026", "dateFormat": "dd MMMM yyyy", "locale": "en_GB"}'
```

### 7.5 Loan Repayment

```bash
curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST "https://localhost:8443/fineract-provider/api/v1/loans/1/transactions?command=repayment" \
  -d '{
    "transactionDate": "07 February 2026",
    "transactionAmount": 4500.00,
    "dateFormat": "dd MMMM yyyy", "locale": "en_GB"
  }'
```

The repayment is allocated to fees → interest → principal per the product's transaction processing strategy, and (under accrual accounting) the corresponding journal entries are posted automatically.

### 7.6 Savings Accounts

```bash
# Create savings account for client 1 on savings product 1
curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST https://localhost:8443/fineract-provider/api/v1/savingsaccounts \
  -d '{
    "clientId": 1, "productId": 1,
    "submittedOnDate": "10 January 2026",
    "dateFormat": "dd MMMM yyyy", "locale": "en_GB"
  }'

# Activate, then deposit and withdraw
curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST "https://localhost:8443/fineract-provider/api/v1/savingsaccounts/1?command=activate" \
  -d '{"activatedOnDate": "12 January 2026", "dateFormat": "dd MMMM yyyy", "locale": "en_GB"}'

curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST "https://localhost:8443/fineract-provider/api/v1/savingsaccounts/1/transactions?command=deposit" \
  -d '{"transactionDate": "12 January 2026", "transactionAmount": 10000, "dateFormat": "dd MMMM yyyy", "locale": "en_GB"}'

curl --insecure -u mifos:password -H 'Content-Type: application/json' \
  -H 'Fineract-Platform-TenantId: default' \
  -X POST "https://localhost:8443/fineract-provider/api/v1/savingsaccounts/1/transactions?command=withdrawal" \
  -d '{"transactionDate": "20 January 2026", "transactionAmount": 2000, "dateFormat": "dd MMMM yyyy", "locale": "en_GB"}'
```


## 8. The Loan Product Lifecycle

Putting the API calls together, the canonical end-to-end lifecycle of a loan in Fineract is:

```
1. Create client (or group)       POST /clients
2. Define loan product            POST /loanproducts (interest method, schedule, fees, GL mappings)
3. Create loan account            POST /loans (product + principal + term + disbursement date)
4. Approve                        POST /loans/{id}?command=approve (maker-checker may interpose)
5. Disburse                       POST /loans/{id}?command=disburse (cash → loan asset postings)
6. Repay installments             POST /loans/{id}/transactions?command=repayment (fees → interest → principal)
7. Manage exceptions              waiveinterest · waivecharge · prepay · reschedule · writeoff · addcharge
8. Track delinquency              arrears aging / PAR via reports + batch jobs
9. Close / write-off              ?command=close / ?command=writeoff (final GL postings)
```

Throughout, every state change is a persisted command (audit trail), optionally maker-checker gated, and — under accrual accounting — mirrored by automatic double-entry journal entries.


## 9. Accounting Integration

Fineract is not just a portfolio tracker; it maintains a real double-entry ledger that makes the books balance. This is the piece that most distinguishes a core banking system from a loan-management CRM.

### 9.1 Double-Entry Bookkeeping and the Chart of Accounts

Each tenant has a **chart of accounts** (`gl_account`): GL accounts typed as **asset, liability, equity, income, expense**, with usage classes (detailed, header, cash). Every financial transaction produces **journal entries** (`gl_journal_entry`) with debits and credits that must balance. The trial balance, balance sheet, and income statement reports are computed directly from these entries.

### 9.2 Product-to-GL Account Mapping

Each loan, savings, or deposit product carries a mapping from its **financial events** to GL accounts. For example, a loan product with accrual accounting requires:

| Event | Typical GL account |
|---|---|
| Disbursement | Loan portfolio (asset) — debit; cash/bank (asset) — credit |
| Repayment — principal | Cash — debit; loan portfolio — credit |
| Repayment — interest | Cash — debit; interest income (income) — credit |
| Interest accrual (monthly) | Interest receivable (asset) — debit; interest income — credit |
| Fees | Cash — debit; fee income — credit |
| Write-off | Write-off expense (expense) — debit; loan portfolio — credit |
| Overpayment refund | Overpayment liability (liability) |

Savings products map interest expense, deposit/withdrawal cash movements, and overdraft positions similarly. If the mapping is incomplete, product creation fails — Fineract enforces accounting completeness at configuration time.

### 9.3 Automatic Journal Entries

Portfolio events (disbursement, repayment, interest posting, charge, write-off, transfer, savings deposit/withdrawal, fixed-deposit maturity) automatically generate journal entries through the accounting rules engine — inside the same transaction as the portfolio change, so the ledger can never drift from the portfolio. External GL systems can be fed by exporting journal entries or consuming the journal-entry reporting API.

### 9.4 Accrual Accounting and Financial Activity Mappings

- **Cash basis** (accounting rule 1): income/expense recognized only when cash moves — simpler, used by many MFIs.
- **Accrual basis** (accounting rule 2): income is recognized as it is earned; monthly **interest accrual batch jobs** post accruals to receivable accounts and recognize income in the period it belongs to. This is what formal banks and regulators expect.
- **Financial activity mappings** tie *conceptual* activities (interest income recognition, write-off expense, overpayment liability, savings interest expense, etc.) to GL accounts independently of individual products, keeping the income statement and balance sheet generation stable and auditable.

### 9.5 Financial Reporting

The reporting module (4.10) exposes the standard financial statements:

- **Trial balance** — all GL accounts with debit/credit totals (the integrity check).
- **Balance sheet** — assets vs. liabilities + equity per the chart of accounts.
- **Income statement** — income vs. expense, driven by financial activity mappings.
- **GL account statements** — per-account transaction history.
- Portfolio reports (portfolio at risk, arrears, aging) for credit management, plus XBRL export for external reporting.


## 10. Multi-Tenancy and Scalability

### 10.1 Tenant Model

Fineract was designed multi-tenant from the start, so the same codebase serves both a single bank and a SaaS platform:

- A **tenant** = one institution (or one business unit/legal entity of an institution).
- The **tenants database** (`fineract_tenants`) is the registry: table `tenant_server_connections` maps each tenant identifier (`default`, `bankA`, `mfiB`, …) to its **OLTP database connection** (and optionally a separate **reporting database connection**).
- Each tenant gets its **own database/schema** with the full platform schema (tables like `m_client`, `m_loan`, `m_savings_account`, `gl_account`, …), created automatically via Liquibase when a tenant is provisioned.
- **Tenant configuration** (currency, locale, date formats, holiday calendars, product catalogs, users) lives inside each tenant database — tenants are fully independent, including their chart of accounts.
- Clients address a tenant per request via the `Fineract-Platform-TenantId` header (see 6.2).

### 10.2 Data Isolation

Tenant isolation is at the **database level** — the strongest practical isolation short of separate physical infrastructure. Tenants cannot read each other's data through the API, and a compromised tenant's data is still separated at the storage layer. For stricter compliance (e.g., data-residency requirements), a SaaS operator can also place tenants on different database servers or even different regions — the tenants DB simply points each tenant at its connection.

### 10.3 Scaling Patterns

| Pattern | Mechanism |
|---|---|
| Vertical | Bigger JVM heap, more CPU — the monolith handles a large single-tenant portfolio on one node |
| Horizontal (read) | Multiple **read instances** pointed at **read replicas** of the tenant DBs (documented instance types, 3.5) |
| Horizontal (write/batch) | Dedicated **write instance** + **batch instances** for CoB jobs; master/worker batch topology |
| Caching | Spring cache abstraction (per-tenant caches for configuration lookups) |
| Event offload | Business events to Kafka/JMS for downstream analytics without blocking the API path |
| SaaS | Many tenants, each with its own DB, behind one (or several) Fineract clusters; per-tenant capacity planning via the tenants registry |

The documented reference topology for scale: 1 write instance + 1+ batch instances + N read instances with read replicas (see the multi-instance diagram in the official deployment docs). For workloads beyond this, the (now archived) Fineract CN microservices were the intended answer — but for the vast majority of Fineract deployments, the monolith with instance types is the practical production pattern.


## 11. Deployment and Operations

### 11.1 Deployment Options

| Option | Notes |
|---|---|
| **Docker (official image)** | `apache/fineract` on Docker Hub; quickest path to a running instance (see 18.1). The image entrypoint supports plugin JARs from `/app/libs/`. |
| **Spring Boot JAR** | `java -jar fineract-provider-<version>.jar` — the recommended distribution; supports `-Dloader.path=libs/` for plugins. |
| **WAR on Tomcat** | Legacy distribution for existing Tomcat estates; works, but the project recommends the JAR. |
| **Kubernetes** | Official docs cover K8s deployment (including the GKE guide); community **Helm charts** exist; the `liquibase-only` profile makes migrations a first-class deployment step. |
| **Cloud marketplaces** | AWS Marketplace and other marketplaces list community AMIs/appliances; docs include AWS and Google Cloud deployment chapters. |
| **Managed hosting** | Community providers (Mifos-aligned consultancies) offer managed Fineract hosting and support. |
| **On-premise** | Plain JAR/WAR behind a proxy on institution-controlled hardware — the dominant MFI pattern. |

### 11.2 Database Setup and Migrations

- **PostgreSQL is the supported production database** (MySQL/MariaDB deprecated since FSIP-9; MariaDB4j remains for integration tests).
- Two databases minimum: `fineract_tenants` (tenant registry) and one tenant DB per tenant (e.g., `fineract_default`).
- Migrations are handled by **Liquibase** (recent releases; Flyway in earlier 1.x). On startup, the write instance applies pending migrations; in Kubernetes, run migrations explicitly with the **`liquibase-only` Spring profile** (controlled by `FINERACT_LIQUIBASE_ENABLED`) before rolling out new application pods — this avoids multiple instances racing to migrate.
- New tenant databases are initialized automatically via Liquibase when first referenced.

### 11.3 Configuration

Configuration is via `application.properties`, Java system properties, or environment variables (`FINERACT_*`). Notable settings:

| Variable | Purpose |
|---|---|
| `FINERACT_SERVER_PORT` / `FINERACT_SERVER_SSL_ENABLED` | Port (default 8443) and SSL on/off |
| `FINERACT_SERVER_SSL_KEY_STORE` (+ password) | Replace the built-in dev certificate |
| `FINERACT_DEFAULT_TENANTDB_*` / `FINERACT_TENANTSDB_*` | JDBC URLs and credentials for tenant(s) and tenants DBs |
| `FINERACT_SECURITY_BASICAUTH_ENABLED` / `FINERACT_SECURITY_OAUTH_ENABLED` / `FINERACT_SERVER_OAUTH_RESOURCE_URL` | Authentication modes (6.3) |
| `FINERACT_LIQUIBASE_ENABLED` | Liquibase on/off (batch/read instances: off) |
| `SPRING_PROFILES_ACTIVE` | Profiles incl. `liquibase-only`, and instance-type profiles (read/write/batch) |

Many additional properties exist for database pooling, caching, Kafka/JMS eventing, metrics, Resilience4j, Tomcat, and SSL — documented in the deployment appendix of the official docs.

### 11.4 Security Hardening

- **HTTPS is enforced by default** — Fineract refuses plain HTTP even in dev. A self-signed cert ships for localhost development only. For production, terminate TLS at a managed cloud proxy (auto-rotating certs, e.g., cloud load balancer or NGINX + Let's Encrypt); Fineract honors `X-Forwarded-For` / `X-Forwarded-Proto` (FINERACT-914).
- **Change bootstrap credentials** (`mifos`/`password`) immediately; enforce password policies; use RBAC with least-privilege roles; enable maker-checker for sensitive commands.
- Choose **OAuth2/OIDC** for customer-facing APIs; keep basic auth internal where used.
- Run vulnerability scans and keep dependencies current — Fineract publishes security reports (fineract.apache.org/security.html); the ASF takes reports privately and fixes are released on a disclosed schedule. See the repo's container/security guides (`container_certificates_guide.md`, `openscap_guide.md`, `sbom_c_proc_cobol.md`) for SBOM and image-scanning practice.
- Backups of tenant DBs are the primary DR control; test restore regularly (see 11.5).

### 11.5 Monitoring, Backup, and Upgrades

- **Monitoring:** JMX (Spring Boot Actuator) health/metrics endpoints, standard JVM/log monitoring (Prometheus/Grafana via actuator), application logs, and the batch-job audit (each job run is logged with status).
- **Backup/restore:** database-level backups (pg_dump / managed DB snapshots) of `fineract_tenants` + every tenant DB; consistent backup strategy must account for the tenants registry pointing at tenant DBs. Restore drills are mandatory before go-live.
- **Upgrades:** Fineract publishes release notes and migration steps per version; Liquibase changesets carry schema evolution. Upgrade path: pin a version → test migrations in staging (`liquibase-only`) → roll application pods → run data-quality checks. The API has a documented backward-compatibility policy, easing client upgrade coordination.
- **Batch jobs:** close-of-business jobs (interest accrual/posting, delinquency, report generation) are managed via the jobs API (`/jobs`) and scheduler — schedule them outside peak hours in the tenant's timezone.


## 12. Fineract CN: The Cloud-Native Variant

**Fineract CN** was the project's attempt at a cloud-native re-architecture: a microservices rewrite of the core using **Kafka** as an event bus, **12-factor/event-driven** patterns, Docker/Kubernetes packaging, REST APIs per service, and **separate databases per service**. The service map:

| Service | Responsibility |
|---|---|
| `provisioner` | Tenant provisioning and lifecycle |
| `identity` | Authentication, roles, permissions |
| `organization` | Offices, staff, holidays, currencies |
| `customer` | Clients, groups, KYC |
| `accounting` | Chart of accounts, journal entries |
| `loan` | Loan products, accounts, schedules |
| `savings` | Savings products and accounts |
| `deposit` | Fixed/recurring deposit products and accounts |
| `portfolio` | Aggregation/orchestration across services |
| `notification` / `reporting` etc. | Ancillary services (SMS/email notifications, reporting) |

**Status and honest assessment:**

- Fineract CN **development has effectively stopped and the CN repositories are archived**; the official FAQ states Fineract CN is no longer part of the project's active line. It was always a preview/experimental architecture.
- The **community explicitly recommends Fineract 1.x for production**.
- Lessons from CN that did flow back into 1.x: cleaner modularization ambitions, Kafka/JMS business-event publishing (now in 1.x), and better multi-instance operations.
- For architects evaluating it today: do not build new production systems on Fineract CN; if you need microservices, run Fineract 1.x as a well-contained service behind your own BFF/integration layer, or pair it with event-driven middleware of your choice.


## 13. Ecosystem and Integrations

### 13.1 The Mifos Community

The Mifos Initiative (mifos.org) maintains the reference front-ends and adjacent products:

- **Mifos X (mifosx)** — the modern **Angular** web application: the standard back-office UI for Fineract (client onboarding, loan/savings operations, accounting, reports). It is a pure API client, so custom UIs can be built against the same API.
- **Legacy community-app** — the older AngularJS web UI (`openmf/community-app`), still deployed by some institutions.
- **Mifos mobile apps** — Android/iOS reference apps for field officers and customers, consuming the self-service API.
- **Mifos Payment Hub EE** — the enterprise payment orchestration layer (see 13.2).
- **Mifos X books/accounting** — day-to-day accounting operations handled through the Mifos X back office on top of Fineract's GL.
- **Fineract SDK / fineract-client** — generated Java client for programmatic access; Swagger Codegen supports other languages.

### 13.2 Payments and Mobile Money

- **Payment Hub EE** (openMF/payment-hub-ee) — Mifos's payment orchestration hub integrating with **Mojaloop** (the open-source interoperable payments platform, backed by the Gates Foundation) and **Interledger**. It routes payment requests between Fineract, mobile-money operators, and other DFSPs — the standard stack for interoperable digital-financial-services deployments.
- **Mobile money integrations** — M-Pesa, Airtel Money, MTN MoMo, and other wallets are integrated via Payment Hub or bespoke adapters (mobile-money operator APIs are typically REST/SMS/USSD-based).
- **SMS/USSD channels** for customer transactions in low-connectivity markets.
- **SMS/email gateways** for notifications (Twilio-style providers or local aggregators).

### 13.3 Other Integrations

- **Pentaho reporting** — the historical BI/report-designer integration; in current releases the Pentaho report runner is packaged as a **plugin JAR** (e.g., the community `fineract-pentaho` plugin) rather than bundled code.
- **Credit bureaus** — custom integrations over the API (pull reports during loan origination; push repayment history).
- **Biometric authentication** — field-level biometric enrolment/verification integrations (Aadhaar-style) implemented as external services calling Fineract APIs.
- **Core banking / external systems** — any system can integrate via the REST API; typical patterns: ERP/GL export, CRM, analytics warehouses (via Kafka business events), and government systems.
- **Extensibility mechanisms:** data tables (custom fields), hooks (webhooks), plugins (`-Dloader.path=libs/`), and custom REST modules built on Fineract's module infrastructure.

### 13.4 Commercial Support and Implementers

- **Connexta** — long-time core engineering contributor; provides commercial services around Fineract.
- **Mifos Initiative partners** — a network of implementation partners worldwide offering deployment, customization, and managed services for MFIs and digital banks.
- **Community consultancies** — numerous smaller firms (often founded by former Mifos/Fineract committers) offer implementation, hosting, and support.
- The practical support model: community (mailing lists, Matrix, JIRA) + a paid implementer for SLA-backed production support — the standard open-source-enterprise pattern.


## 14. Use Cases and Case Studies

| Use case | How Fineract fits |
|---|---|
| **Digital banks (Africa/Asia)** | API-first core for a mobile bank: customers onboard via app, transact via mobile money/cards, lending via declining-balance products. E.g., **TymeBank / GoTyme Bank** (South Africa; Tyme Group ~20M customers across markets) — widely reported to run its digital banking platform on Fineract; other African/Asian digital banks and fintechs similarly use Fineract cores. |
| **Traditional microfinance** | Grameen-model MFIs: centers + groups + JLG loans + weekly repayment schedules + flat interest — Fineract's native domain. Hundreds of MFIs; the historical SKS Microfinance-style high-volume group-lending model is the design center of the platform. |
| **Financial inclusion programs** | Government/NGO subsidized lending, national financial-inclusion initiatives, DPGA-backed programs — open source, auditable, and free of vendor lock-in, which is a procurement requirement in many aid programs. |
| **Fintech lending platforms** | Digital lenders use Fineract purely as the loan-management/ledger engine behind their own origination and collection apps (Fineract is the book of record; the fintech owns the customer experience). |
| **Savings groups / community savings** | Group savings accounts + internal lending — SHG programs and village savings and loan associations (VSLAs) digitized. |
| **Credit unions / community banks** | Savings + loans + GL for small member-owned institutions; some credit-union deployments exist, particularly where group structures matter. |

**The typical digital-bank stack on Fineract:**

```
Channels (mobile app · USSD/SMS · web · agents/ATM)
        ↓
Integration layer (BFF/API gateway, OAuth2/OIDC)
        ↓
Apache Fineract core (loans, savings, GL, clients, reporting) ← Payment Hub EE (mobile money, Mojaloop)
        ↓
Reporting & analytics (Pentaho / BI warehouse via Kafka events) · Back office (Mifos X web app)
```


## 15. Fineract vs. Commercial Core Banking Systems

Fineract competes with commercial cores such as **Temenos Transact (T24)**, **Mambu**, **Thought Machine Vault**, **Oracle Flexcube**, **FIS Profile/Systematics**, **Avaloq**, and (for markets/treasury) **Murex** (see the repo guides for Murex and other trading systems). The comparison is not "worse vs. better" — it is about *fit*.

### 15.1 Comparison Table

| Dimension | Apache Fineract (open source) | Commercial cores (T24, Mambu, Vault, Flexcube, FIS, Avaloq) |
|---|---|---|
| **Cost** | Free (Apache 2.0); costs = infrastructure + your own engineering or implementer fees | License + annual maintenance (often 20–25% of license); implementation and SI fees on top; typically 7–8 figures for tier-1 deployments |
| **Deployment** | On-prem or cloud, self-managed; Docker/K8s; full control | Vendor SaaS (Mambu, Vault) or on-prem (T24, Flexcube) with vendor-run upgrades |
| **Flexibility** | Full source access; customize anything; fork or extend via data tables/hooks/plugins | Configure within product boundaries; changes require vendor roadmap or professional services; some (Vault) offer code-level programmability but inside vendor platform |
| **Microfinance support** | Native: JLG/group lending, centers, flat & declining interest, weekly/irregular schedules, SHGs — best-in-class | General retail focus; group lending is typically custom work |
| **Digital-banking features** | API-first by design (the UI itself is an API client); ideal for mobile-first | Mambu/Vault are API-native; T24/Flexcube are richer but heavier to expose via APIs (Open Banking modules exist) |
| **Product breadth** | Loans, savings, deposits, GL, reporting — retail/MFI scope | Enterprise breadth: trade finance, treasury, wealth, cards, payments, channel banking, core plus satellite systems |
| **Scalability** | Moderate-to-good via instance types/read replicas; proven to millions of customers; not built for ultra-low-latency global cores | Enterprise-grade; T24/Flexcube run some of the world's largest banks; Vault built for cloud scale |
| **Support** | Community (mailing lists/Matrix/JIRA) + optional paid implementers; no vendor SLA | Vendor SLA, certification programs, global support network |
| **Regulatory compliance** | Configurable controls (audit trail, maker-checker, RBAC, accrual) — but **you** assemble and evidence compliance | Built-in/compliance-ready modules (regulatory reporting packs, Basel/IFRS tooling), vendor compliance certifications and reference implementations |
| **Target market** | MFIs, digital banks, fintechs, inclusion programs, cost-sensitive institutions | Tier-1 and mid-tier banks needing enterprise features, complex products, and certified vendor support |

### 15.2 When to Choose Fineract

- **Microfinance / group-lending / inclusion-focused institutions** — no commercial core matches its native JLG/SHG/center model.
- **Cost-sensitive startups and digital banks** — API-first core at near-zero license cost; spend budget on the customer experience instead.
- **Financial-inclusion and government programs** — open source, auditable, DPGA-verified, no vendor lock-in, local implementers allowed.
- **Custom core needs** — you intend to modify the core (full source access) or build heavily bespoke products.
- **API-first digital banks** — the API is the product; Fineract's REST surface is comprehensive and versioned.
- **Rapid prototyping** — spin up a working core in minutes (Docker) to validate a banking product idea.

### 15.3 When Not to Choose Fineract

- **Large tier-1 banks** needing enterprise breadth (trade finance, treasury, wealth, cards management, payments hub) — those are outside Fineract's scope; you'd be assembling a large ecosystem around it.
- **Complex product lines** — trade finance instruments, derivatives, treasury/ALM: Fineract is a retail portfolio core, not a wholesale banking platform.
- **Heavily regulated jurisdictions demanding certified cores** — if the regulator effectively requires vendor certification or packaged compliance modules, an open-source core means you must build and evidence those controls yourself (see [Section 17](#17-production-considerations)).
- **Organizations without Java/Spring engineering capacity** — you will need that skill for production hardening and customizations.
- **Ultra-high-throughput global cores** — Fineract's monolith scales well but is not architected for the extreme concurrency profiles of top-tier global transaction banks.


## 16. Fineract in a Bank's Architecture

For a banking architect, Fineract is best understood as a *portfolio-and-ledger engine with an API*, which can be slotted into several architectural roles:

| Role | Description |
|---|---|
| **Core for a digital bank subsidiary** | A conventional bank (or a large corporate like a retailer/telecom) launches a digital bank on Fineract as the subsidiary's core: accounts, loans, GL, reporting — with the parent's systems integrated via APIs. TymeBank is the canonical example. |
| **Lending engine (loan management as a service)** | Keep the existing core; run Fineract as the loan book engine for a new lending business line, feeding the GL via journal-entry export and the CRM via webhooks — a low-risk way to launch digital lending. |
| **Microfinance module within a banking group** | A commercial bank with a microfinance arm (community banking, rural lending, government-subsidized SME programs) runs Fineract for that portfolio, segregated from the main core. |
| **Financial-inclusion initiatives** | Government-subsidized lending, national ID-linked accounts, agricultural credit schemes — Fineract as the accountable system of record with full audit trails. |
| **Reference / learning platform** | Open source means your engineers can study a complete core-banking implementation (schedules, accruals, double-entry) — an excellent training ground for core banking concepts. |
| **Rapid prototyping** | Prove a product, pricing model, or integration pattern in days (Docker + Swagger), then decide whether to productionize on Fineract or move to a commercial core. |

**Architecture pattern that recurs in production:** Fineract is wrapped by an integration layer (API gateway / BFF) that owns authentication (OAuth2/OIDC), channel adaptation, and orchestration; Fineract owns the books; Payment Hub EE owns money movement; a warehouse consumes Kafka business events for analytics; Mifos X (or a custom back office) serves operations. This mirrors the "core engine + integration layer" pattern used with commercial cores — only the core engine is open source and self-hosted.


## 17. Production Considerations

Going to production on an open-source core is a governance exercise as much as an engineering one. Key considerations:

| Area | What to do |
|---|---|
| **Security review** | Open source = you can (and should) audit the code. Run dependency/vulnerability scanning (OWASP Dependency-Check, Trivy on the container image — see the repo's `container_certificates_guide.md`, `openscap_guide.md`, `vuln_scanning_c_proc_cobol.md` for patterns), review Fineract security advisories (fineract.apache.org/security.html), and consider a paid security assessment before go-live. |
| **Compliance / regulatory approval** | Regulatory treatment of open-source cores varies by jurisdiction. For a **MAS digital bank license application (Singapore)**, the regulator does not require a specific vendor, but you must demonstrate: robust **controls** (RBAC, maker-checker, segregation of duties), complete **audit trails**, **data residency** (Singapore data hosted in-region), business continuity, and operational resilience. Open source is defensible — MAS focuses on outcomes, not provenance — but you must document how you govern the codebase (who reviews changes, how CVEs are handled). |
| **Data residency** | Self-hosting gives full control over where data lives — directly relevant to **PDPA** (Singapore) and MAS data-residency expectations. Choose cloud region/on-prem accordingly; per-tenant DBs make multi-region placement tractable. |
| **Support strategy** | Decide between community-only, a paid implementer (Mifos partners, Connexta), or in-house expertise. For a licensed bank, an SLA-backed arrangement plus in-house capability is the prudent mix. |
| **Upgrade strategy** | Pin versions, run migrations in staging with the `liquibase-only` profile, subscribe to release/security announcements, and keep an upgrade runbook. The API backward-compatibility policy protects your channel clients. |
| **Skills** | You need Java/Spring engineers (for hardening, customizations, incident response) — plus DB administration (PostgreSQL). Budget for this; it is the main hidden cost of open source. |
| **Customizations** | Prefer non-invasive extension (data tables, hooks, plugins, new REST modules) over forking. If you fork, track upstream closely, minimize divergence, and plan to re-base each release — forks are the classic open-source trap. |
| **Data migration** | Plan migration from legacy systems (account, schedule, GL history) with cutover reconciliation; Fineract's import tooling (and community scripts) help, but expect real migration engineering. |
| **Testing** | Fineract ships an extensive integration-test suite (and CI runs on PostgreSQL); replicate key flows in your own non-prod tenant before upgrades. |


## 18. Getting Started

### 18.1 Run It in Five Minutes (Docker)

```bash
# Pull and run the official image (PostgreSQL included in the compose stack)
docker pull apache/fineract
docker run --name fineract -p 8443:8443 apache/fineract
```

The community publishes a Docker Compose setup (Fineract + PostgreSQL + Mifos X web app) that gives you a complete environment:

```yaml
services:
  fineract:
    image: apache/fineract
    ports: ["8443:8443"]
    environment:
      FINERACT_DEFAULT_TENANTDB_JDBC_URL: jdbc:postgresql://db/fineract_default
      # ... (see docs for full env list)
  webapp:
    image: openmf/mifosx-web-app   # or community-app for legacy UI
    environment:
      FINERACT_API_URL: https://localhost:8443
      FINERACT_PLATFORM_TENANT_IDENTIFIER: default
```

After startup (a minute or two — Liquibase initializes the schema), the API listens on **https://localhost:8443** (self-signed cert for local dev; use `-k`/insecure in curl). Default login: `mifos` / `password`, tenant `default`.

### 18.2 Explore the API

- Swagger UI: `https://localhost:8443/fineract-provider/swagger-ui/` (older releases: `/fineract-provider/api/v1/swagger-ui.html`); raw spec at `/fineract-provider/api-docs`.
- Community sandbox: `sandbox.mifos.community` (public demo tenant).
- Work through the walkthrough in [Section 7](#7-hands-on-api-walkthrough): login → client → loan product → loan → approve → disburse → repay → savings deposit. Each step is visible in the Mifos X web app too (same API, different client).

### 18.3 Developer Setup (Source)

```bash
git clone https://github.com/apache/fineract.git && cd fineract
./gradlew createPGDB -PdbName=fineract_tenants
./gradlew createPGDB -PdbName=fineract_default
./gradlew devRun        # API on port 8443
```

### 18.4 Key Resources

| Resource | URL |
|---|---|
| Project site & docs | fineract.apache.org · fineract.apache.org/docs/stable |
| Source & issues | github.com/apache/fineract · issues.apache.org/jira (FINERACT) |
| Mifos Initiative | mifos.org (community, front-ends, Payment Hub EE, partners) |
| Community channels | dev@fineract.apache.org mailing list · Matrix `#apache-fineract-home:matrix.org` |
| Fineract CN (archived) | github.com/apache/fineract-cn (reference only) |
| Docker image | hub.docker.com/r/apache/fineract |
| Sandbox | sandbox.mifos.community |
| Security advisories | fineract.apache.org/security.html |
| Pentaho reporting plugin | github.com/vorburger/fineract-pentaho |


## 19. References and Further Reading

1. Apache Fineract project site and documentation — fineract.apache.org; official docs (stable 1.15.0), including architecture, deployment, security, and FAQ chapters.
2. Apache Fineract GitHub repository and README (quickstart, API examples, CI status) — github.com/apache/fineract.
3. Mifos Initiative — mifos.org; "Apache Fineract Graduates to Top-Level Project" (Apr 2017).
4. Apache Fineract wiki — cwiki.apache.org/confluence/display/FINERACT (FAQ incl. Fineract CN status; FSIP-9 PostgreSQL standardization).
5. Tyme/GoTyme Bank — gotyme.co.za newsroom (Tyme Group, ~20M customers); public reporting on TymeBank's Fineract-based digital banking platform.
6. Digital Public Goods Alliance — digitalpublicgoods.net (Apache Fineract verified DPG).
7. Related guides in this repository: `data_models_banking_insurance_guide.md` (BIAN, banking data models), `end_to_end_banking_processes.md`, `financial_risk_compliance_systems_guide.md` (MAS, BCBS 239, IFRS 9), `financial_technology_overview.md`, `container_certificates_guide.md` and `openscap_guide.md` (image/OS hardening), and the Murex guide (trading/treasury systems).
8. Mojaloop / Payment Hub EE — mojaloop.io; openMF/payment-hub-ee (payments interoperability with Fineract).

---

*This guide is a technical research document for architecture and technology evaluation. Product facts reflect the Apache Fineract project as of August 2026 (1.15.x line); verify current versions, feature availability, and licensing details against the official project site before making procurement or architecture decisions.*
