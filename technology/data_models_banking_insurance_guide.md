# Data Models for Banking and Insurance: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Core Data Architecture / Domain Modeling — Banking, Insurance, Enterprise Data Platforms  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** July 2026

---

## Table of Contents

1. [The Canonical Data Model (CDM)](#1-the-canonical-data-model-cdm)
   - 1.1 [What Is a Canonical Data Model?](#11-what-is-a-canonical-data-model)
   - 1.2 [The Integration Problem: N-to-N vs. N-to-1](#12-the-integration-problem-n-to-n-vs-n-to-1)
   - 1.3 [How a CDM Works in Practice](#13-how-a-cdm-works-in-practice)
   - 1.4 [Benefits of a Canonical Data Model](#14-benefits-of-a-canonical-data-model)
   - 1.5 [CDMs in the Modern Data Platform](#15-cdms-in-the-modern-data-platform)
2. [Banking Data Models](#2-banking-data-models)
   - 2.1 [BIAN — Banking Industry Architecture Network](#21-bian--banking-industry-architecture-network)
   - 2.2 [IBM Banking Data Warehouse (BDW)](#22-ibm-banking-data-warehouse-bdw)
   - 2.3 [Teradata Financial Services Logical Data Model (FSLDM)](#23-teradata-financial-services-logical-data-model-fsldm)
   - 2.4 [FIS Banking Data Model](#24-fis-banking-data-model)
   - 2.5 [SAS Banking Data Model](#25-sas-banking-data-model)
   - 2.6 [Oracle Financial Services Data Model (OFSDFM)](#26-oracle-financial-services-data-model-ofsdfm)
   - 2.7 [Banking Model Comparison](#27-banking-model-comparison)
3. [Insurance Data Models](#3-insurance-data-models)
   - 3.1 [ACORD — Insurance Data Standards](#31-acord--insurance-data-standards)
   - 3.2 [IBM Insurance Information Warehouse (IIW)](#32-ibm-insurance-information-warehouse-iiw)
   - 3.3 [SAS Insurance Data Model](#33-sas-insurance-data-model)
   - 3.4 [Guidewire InfoCenter](#34-guidewire-infocenter)
   - 3.5 [Insurance Model Comparison](#35-insurance-model-comparison)
4. [Modern Data Modeling Approaches](#4-modern-data-modeling-approaches)
   - 4.1 [Data Vault 2.0](#41-data-vault-20)
   - 4.2 [Data Vault vs. 3NF vs. Dimensional](#42-data-vault-vs-3nf-vs-dimensional)
   - 4.3 [Data Mesh](#43-data-mesh)
   - 4.4 [Data Fabric](#44-data-fabric)
   - 4.5 [Data Fabric vs. Data Mesh](#45-data-fabric-vs-data-mesh)
5. [Regulatory Data Model Requirements](#5-regulatory-data-model-requirements)
   - 5.1 [BCBS 239 — Risk Data Aggregation and Reporting](#51-bcbs-239--risk-data-aggregation-and-reporting)
   - 5.2 [MAS Guidelines (Singapore)](#52-mas-guidelines-singapore)
   - 5.3 [GDPR and PDPA — Privacy Regulations](#53-gdpr-and-pdpa--privacy-regulations)
   - 5.4 [Solvency II (EU Insurance)](#54-solvency-ii-eu-insurance)
   - 5.5 [IFRS 17 — Insurance Contracts](#55-ifrs-17--insurance-contracts)
   - 5.6 [IFRS 9 / CECL — Expected Credit Loss](#56-ifrs-9--cecl--expected-credit-loss)
   - 5.7 [Regulatory Requirements Summary](#57-regulatory-requirements-summary)
6. [Data Model Implementation Patterns](#6-data-model-implementation-patterns)
   - 6.1 [Reference Data Management](#61-reference-data-management)
   - 6.2 [Transaction Data Modeling](#62-transaction-data-modeling)
   - 6.3 [Balance and Position Modeling](#63-balance-and-position-modeling)
   - 6.4 [Agreement Lifecycle Modeling](#64-agreement-lifecycle-modeling)
   - 6.5 [Customer 360 Model](#65-customer-360-model)
   - 6.6 [Worked Example: Account and Transaction Model](#66-worked-example-account-and-transaction-model)
   - 6.7 [Event-Driven Data Models](#67-event-driven-data-models)
7. [Tools for Data Modeling](#7-tools-for-data-modeling)
   - 7.1 [erwin Data Modeler](#71-erwin-data-modeler)
   - 7.2 [ER/Studio](#72-erstudio)
   - 7.3 [SAP PowerDesigner](#73-sap-powerdesigner)
   - 7.4 [Hackolade](#74-hackolade)
   - 7.5 [SQLDBM](#75-sqldbm)
   - 7.6 [dbt (Data Build Tool)](#76-dbt-data-build-tool)
   - 7.7 [Tool Comparison](#77-tool-comparison)
8. [Data Modeling Best Practices for Banking and Insurance](#8-data-modeling-best-practices-for-banking-and-insurance)
9. [Conclusion and Selection Guide](#9-conclusion-and-selection-guide)
   - 9.1 [How to Choose a Data Model](#91-how-to-choose-a-data-model)
   - 9.2 [The Modern Recommendation](#92-the-modern-recommendation)
   - 9.3 [Banking-Specific Guidance](#93-banking-specific-guidance)
   - 9.4 [Insurance-Specific Guidance](#94-insurance-specific-guidance)
10. [References](#10-references)

---

## 1. The Canonical Data Model (CDM)

### 1.1 What Is a Canonical Data Model?

A **canonical data model (CDM)** is a standardized, enterprise-wide data model that defines common data structures, definitions, and relationships across multiple systems and domains. It is the single agreed-upon representation of the enterprise's core data — customers, products, accounts, transactions, parties, agreements — independent of how any individual source system chooses to store that data.

A CDM is more than a schema. It is a **contract** that includes:

- **Entities and attributes** — the core business objects (e.g., `Customer`, `Account`, `Transaction`) and their standardized fields, with agreed data types, lengths, and nullability.
- **Definitions** — a business glossary entry for every entity and attribute, so "customer", "exposure", and "available balance" mean the same thing in every system and every report.
- **Relationships** — how entities relate to each other (a customer holds accounts; an account posts transactions; a transaction references a counterparty), including cardinality rules.
- **Reference data** — standardized code sets: currency codes (ISO 4217), country codes (ISO 3166), transaction types, product categories, status values, and their mappings.
- **Business rules** — validation rules, uniqueness constraints, and integrity rules that all data must satisfy.

The CDM acts as a **common language** between systems. Where two applications historically each spoke their own dialect — one calling a client a `CUST`, another calling it `BORROWER`, a third storing it in a flat file as `party_name` — the CDM establishes one vocabulary: `Party` / `Customer`, with one definition, one identifier strategy, and one set of attributes that every system maps to and from.

Crucially, a CDM is a *target*, not a *starting point*. Most banks do not design a greenfield model and rebuild their core banking systems around it. Instead, they define the canonical layer and then build **mappings** from each existing source system's data into the canonical structures. The CDM becomes the enterprise's "Esperanto" — no system is forced to change its internal storage, but every system can communicate through the common model.

### 1.2 The Integration Problem: N-to-N vs. N-to-1

The primary motivation for a CDM is **integration complexity**. A bank runs dozens — often hundreds — of systems: core banking, payments, cards, trade finance, CRM, risk, finance/GL, regulatory reporting, AML, data warehouses, and more. Every pair of systems that needs to exchange data requires a mapping between their respective formats.

**Without a CDM (point-to-point integration):** each integration is a bespoke mapping between two systems' native schemas.

```
  Core Banking ──┬──▶ CRM
                 ├──▶ Risk
                 ├──▶ GL / Finance
                 └──▶ Regulatory Reporting
  Payments ──────┬──▶ AML
                 ├──▶ Risk
                 └──▶ Regulatory Reporting
  Cards ─────────┬──▶ CRM
                 ├──▶ AML
                 └──▶ Data Warehouse
```

With `N` systems, this yields up to `N × (N-1)` point-to-point mappings. A bank with 40 systems can end up maintaining 1,500+ individual mappings. Each mapping is a small piece of bespoke code or configuration that must be built, tested, documented, and maintained. Every time a source system changes its schema — a new field, a renamed attribute, a changed code value — every downstream mapping must be checked. The result is a brittle spider web where data definitions drift, the same concept is represented differently in each silo, and regulators (BCBS 239 in particular) cannot trace data from source to report.

**With a CDM (hub-and-spoke):** every source maps *once* into the canonical model, and every consumer reads *from* the canonical model.

```
                    ┌──────────────┐
  Core Banking ────▶│              │───▶ CRM
  Payments ─────────▶│              │───▶ Risk
  Cards ────────────▶│     CDM      │───▶ GL / Finance
  Trade Finance ────▶│              │───▶ AML
  CRM ──────────────▶│              │───▶ Regulatory Reporting
                    └──────────────┘    └──▶ Data Warehouse / Analytics
```

Integration complexity drops from `N × M` mappings to `N + M`: `N` source-to-CDM mappings plus `M` CDM-to-consumer mappings. A **new source system** only needs a mapping into the CDM — it does not need to be wired to every consumer. A **new consumer** only needs to understand the CDM — it does not need to learn the quirks of every source. This is the single largest architectural payoff of canonical modeling, and it compounds as the system landscape grows.

### 1.3 How a CDM Works in Practice

The typical implementation approach is to transform source data into the CDM at the **ingestion or curation layer**, and have downstream systems consume the CDM:

1. **Ingest** — raw data is captured from source systems (via file feeds, APIs, change data capture, or message queues) into a landing/staging area. At this point the data is still in the source's native format.
2. **Map and transform** — the curation/standardization layer maps source fields to canonical fields: renaming, re-typing, converting codes, reconciling identifiers, applying reference data mappings, and validating against CDM business rules.
3. **Publish** — the standardized data is written to the canonical layer (the CDM physical store, typically in the enterprise data warehouse or lakehouse).
4. **Consume** — downstream systems — analytics, risk engines, regulatory reporting, data marts, APIs — read from the CDM. Downstream systems can be fed via batch extracts, virtualized views, or event streams of CDM-structured records.

A mapping specification is the unit of governance. For each CDM entity, the enterprise maintains a **source-to-target mapping document** that records, per attribute:

| Mapping attribute | Example |
|---|---|
| CDM entity / attribute | `Account.opened_date` |
| Source system | Core Banking (TCS BaNCS) |
| Source table / field | `ACCT_MSTR.OPEN_DT` |
| Transformation | `TO_DATE(OPEN_DT, 'YYYYMMDD')` |
| Code mapping | Source status `A` → CDM `ACTIVE`; `C` → `CLOSED` |
| Data quality rule | `opened_date` must not be null; must be ≤ today |
| Owner / steward | Retail Banking Data Steward |

These mappings are themselves governed artifacts — versioned, reviewed, and auditable — because they are exactly what regulators inspect when they ask "where does this number in the report come from?"

The CDM can be implemented **physically** (a materialized set of canonical tables that data is loaded into) or **logically** (canonical views defined over source data, with virtualization providing unified access). Most enterprises start physical — materialized canonical tables give performance, history, and a clean audit trail — and add a logical/virtualized layer for real-time or federated queries. We return to this distinction in Section 4 (data fabric).

### 1.4 Benefits of a Canonical Data Model

| Benefit | What it means in practice |
|---|---|
| **Single source of truth** | One authoritative representation of key entities — customer, product, account, transaction — with one definition, one identifier, one owner. Reports and analytics stop arguing over whose "customer count" is right. |
| **Decouples source systems from consumers** | Source systems evolve independently of consumers. A core banking upgrade changes the source schema but not the CDM, so downstream systems are untouched. |
| **Simplifies integration** | New source systems need only one mapping (to the CDM), not one per consumer. Integration effort and maintenance cost drop from O(N×M) to O(N+M). |
| **Enables cross-domain analytics** | Customer, product, and transaction data can be combined consistently: "revenue per customer across all product lines", "risk-weighted assets by customer segment", "cross-sell propensity". This is impossible when each domain speaks its own dialect. |
| **Regulatory alignment** | BCBS 239 and MAS guidelines require consistent risk data aggregation and lineage. A CDM is the natural substrate: consistent definitions, traceable mappings, one path from source to report. |
| **Drives data governance** | A CDM forces the organization to agree on standard definitions, assign data ownership, and codify quality rules. It turns governance from a policy document into an executable artifact. |
| **Reduces reconciliation burden** | When all systems speak the same language, the number of inter-system reconciliation breaks drops. Differences that remain are genuine data issues, not format mismatches. |
| **Accelerates onboarding** | M&A is constant in banking: each acquired bank's systems must be integrated. With a CDM, a new entity's data is mapped once and immediately available to every consumer. |

The costs are real and worth stating plainly: building and maintaining a CDM requires **upfront design effort**, **ongoing mapping maintenance**, and **governance discipline**. A CDM that is not governed decays into just another schema. The benefits above are realized only when the canonical layer is treated as a product with an owner, SLAs, and a change process — not as a one-time integration project.

### 1.5 CDMs in the Modern Data Platform

In a modern lakehouse or cloud data platform, the CDM typically maps to the **"silver" layer** of a medallion architecture:

- **Bronze (raw):** ingested source data, exactly as received — no transformations. This preserves the audit trail and allows re-processing.
- **Silver (canonical / curated):** the CDM. Source data transformed into standardized, conformed, validated structures. This is where the CDM lives in the modern platform.
- **Gold (presentation):** dimensional marts, aggregates, feature stores, and reporting views built *on top of* the CDM for specific consumption needs.

The CDM is therefore not an alternative to the lakehouse — it is the **semantic contract** that the lakehouse enforces in its curated layer. The same principle applies regardless of the underlying technology: the CDM is a *logical* concept (entities, definitions, relationships, rules) that can be physically realized in a relational warehouse, a lakehouse with Delta/Iceberg tables, a cloud data warehouse (Snowflake, BigQuery, Databricks), or a virtualized access layer.

Two implications follow. First, the CDM and the **data vault** (Section 4.1) are complementary: a data vault can be the physical backbone of the silver layer (hubs for business keys, satellites for history), with the CDM's canonical business entities defined as standardized hub/link/satellite patterns plus conformed views. Second, the CDM coexists with **data mesh** and **data fabric** (Sections 4.3–4.5): mesh organizes *who* owns the data (domains), fabric automates *how* it is integrated (metadata, virtualization), and the CDM defines *what* it means — the shared vocabulary that makes both work. A CDM without domain ownership and active metadata is just a database; domain ownership and metadata without a CDM produce federated chaos.

---

## 2. Banking Data Models

A number of industry-standard banking data models have evolved over the past three decades. They differ in origin (vendor vs. consortium), purpose (service design vs. warehousing vs. analytics), structure (3NF vs. dimensional vs. object-based), and breadth. This section covers the major ones: BIAN, IBM BDW, Teradata FSLDM, FIS, SAS, and Oracle OFSDFM.

### 2.1 BIAN — Banking Industry Architecture Network

**BIAN** (Banking Industry Architecture Network, bian.org) is a **semantic standard for banking services**, developed by a consortium of banks, vendors, and consultancies. Where most banking data models answer "how should banking *data* be structured?", BIAN answers "how should banking *capabilities* and the data they operate on be structured?" — a service-oriented view of the bank that has become a reference for API design and architectural standardization in banking.

#### 2.1.1 Structure of the BIAN Standard

BIAN organizes the banking business into a **service landscape** with several layers:

- **8 business domains** — the top-level classification of banking activity: *Business Operations*, *Product & Customer Management*, *Financial & Technical Services*, *Sales & Service*, *Risk Management*, and related domains. Each business domain groups a set of service domains.
- **~400 service domains** — the functional building blocks of a bank. A service domain is a well-defined area of banking capability with a clear responsibility, e.g., *Customer Information Management*, *Product Directory*, *Party Management*, *Account Management*, *Transaction Services*, *Payments*, *Cards*, *Lending*, *Deposits*, *Trade Finance*, *Reference Data Management*.
- **Business objects** — the data entities a service domain manages (e.g., the *Account Management* service domain manages business objects such as `Account`, `Account Balance`, `Account Limit`).
- **Behavioral qualifiers** — attributes or sub-structures that describe a business object's behavior or state (e.g., an account's `status`, `type`, `product instance reference`).
- **Control records** — the root objects of a service domain's data model: the "master record" that ties the domain's business objects and qualifiers together (e.g., the control record for account management is the account itself).

The BIAN model is a **5-layer business object model**: from the service landscape (business domains) down through service domains, business objects, and behavioral qualifiers to the attributes. This layered structure is what makes BIAN simultaneously a *capability map* (for architecture) and a *data model* (for data design).

#### 2.1.2 Key BIAN Concepts

| Concept | Definition | Example |
|---|---|---|
| **Service Domain** | A functional area of the bank with defined responsibilities and capabilities | `Account Management`, `Payments`, `Cards` |
| **Business Object** | A data entity managed within a service domain | `Account`, `Transaction`, `Customer` |
| **Behavior Qualifier** | An attribute or sub-structure describing a business object's behavior or state | Account `status`, `product instance reference`, `account type` |
| **Control Record** | The root object of a service domain's model — the anchor that business objects relate to | The `Account` record in Account Management |

#### 2.1.3 BIAN for Data Modeling

BIAN defines **logical data structures** — business objects and behavioral qualifiers — that can be mapped to physical data models. The **BIAN Service Domain Model (SDM)** standardizes each service domain's business objects and behavior qualifiers, giving a data architect a ready-made catalog of what entities exist in each functional area and how they relate. When a bank adopts BIAN:

- The service landscape becomes the **map of the business** — every data domain and system can be located on it.
- Service domain business objects become **candidate canonical entities** for the CDM (Section 1). A bank building a CDM can take BIAN's `Party`/`Customer`, `Account`, `Product`, and `Transaction` business objects as the starting skeletons of its canonical customer, account, product, and transaction models.
- BIAN's behavioral qualifiers become **standardized attributes** — reducing the "one bank, ten definitions of account status" problem.
- BIAN's service-domain boundaries become **data domain boundaries** — aligning data ownership with capability ownership, which fits naturally into a data-mesh operating model (Section 4.3).
- BIAN's API definitions (the standard exposes service operations per service domain) align the **interface contract** with the **data contract**: the API payloads carry the same business objects the data platform stores.

#### 2.1.4 Assessment

| Dimension | Assessment |
|---|---|
| **Strengths** | Vendor-neutral and industry-backed (a banking consortium, not a single vendor); comprehensive coverage of banking domains; service-oriented view that aligns architecture, API design, and data; active, evolving standard with broad international adoption (widely used in Europe and Asia-Pacific, including Singapore). |
| **Weaknesses** | Complex — the full standard has a significant learning curve; abstract — business objects and qualifiers need interpretation before they become a physical schema (BIAN describes *what*, not necessarily *how to store*); still maturing in some areas (coverage and granularity vary across service domains). |
| **Best for** | Banks that want an industry-standard, service-oriented data model and alignment with banking architecture standards — especially those pursuing API-led modernization, core banking transformation, or a target architecture that must reconcile business capabilities with data. |

**Verdict:** BIAN is the natural choice when the goal is *architectural alignment* — a common language across business, applications, APIs, and data. It is not a physical warehouse schema out of the box; it is the semantic backbone onto which a physical model (e.g., a data vault or 3NF warehouse model) is built.

### 2.2 IBM Banking Data Warehouse (BDW)

The **IBM Banking Data Warehouse (BDW)** is IBM's reference data model for banking data warehousing. It has been continuously developed and shipped for over three decades, making it one of the most mature banking data models in the industry. It is delivered as two related artifacts: the classic **IBM Banking Data Warehouse** model and the newer **IBM Banking Data Model for Analytics**.

#### 2.2.1 Structure and Scope

BDW covers **~20 subject areas** spanning the full banking business, and is delivered in both **3NF (third normal form)** and **dimensional** flavors — the 3NF model for the warehouse core, dimensional variants for analytics marts. In full, the model contains **~2,000 entities and 7,000+ attributes**, a scale that reflects 30+ years of accumulated banking requirements.

The core subject areas:

| Subject Area | Contents |
|---|---|
| **Party** | Customers, employees, organizations, party relationships — the "who" of banking: individuals, legal entities, relationship hierarchies (group → subsidiary), roles (account holder, guarantor, beneficial owner). |
| **Agreement** | Accounts, contracts, and product agreements — the "what the customer holds": deposit accounts, loans, cards, and the terms attached to each agreement. |
| **Product** | Product definitions, product hierarchy, product pricing — the bank's catalog of offerings and how agreements relate to products. |
| **Location** | Geographic entities: countries, regions, branches, addresses, and the hierarchy of places. |
| **Channel** | Delivery channels: branch, ATM, internet banking, mobile, call center, and channel-related events. |
| **Event** | Transaction events and event types: deposits, withdrawals, payments, fees — the activity that flows through the bank. |
| **Condition** | Terms and conditions: rates, limits, fees, and contractual conditions attached to agreements and products. |
| **Asset** | Physical and financial assets: collateral, securities, fixed assets. |
| **Finance** | General ledger, chart of accounts, financial accounting structures, and the mapping from transactions to accounts. |
| **Risk** | Credit, market, and operational risk data: exposures, ratings, limits, losses. |
| **Marketing** | Campaigns, responses, segments — the data needed for customer acquisition and retention analytics. |

#### 2.2.2 Assessment

| Dimension | Assessment |
|---|---|
| **Strengths** | Comprehensive — an industry-standard model with unusually broad coverage of banking subject areas; mature — 30+ years of evolution across hundreds of client implementations; deep IBM ecosystem integration (works with IBM InfoSphere DataStage, IBM Db2, IBM InfoSphere Warehouse, IBM Data Governance tools); strong support for financial services analytics, including risk, finance, and marketing. |
| **Weaknesses** | IBM-centric — the model is documented and optimized around IBM tooling; heavy — with ~2,000 entities, implementing the full model is a significant program of work (most banks implement a subset); licensing costs — BDW is a commercial, licensed model, not open source. |
| **Best for** | IBM shops — banks with an existing IBM data platform — and banks that want a comprehensive, proven reference data model to jump-start a banking warehouse without designing subject areas from scratch. |

**Verdict:** BDW is the classic "big reference model" play: buy a comprehensive blueprint, then tailor and subset it. Its value is highest where the surrounding stack is IBM's, but the logical model itself is a useful reference for any bank building a 3NF warehouse core.

### 2.3 Teradata Financial Services Logical Data Model (FSLDM)

The **Teradata Financial Services Logical Data Model (FSLDM)** is Teradata's comprehensive logical data model for financial services. It is based on **3NF**, covers **banking, insurance, securities, and wealth management** domains, and contains **~2,000 entities**. It is organized around **6 major subject areas**: *Party* (customers, employees, organizations), *Product* (offerings), *Agreement* (contracts and accounts), *Location*, *Channel*, and *Event* (transactions and activities).

FSLDM is notable for being **multi-domain**: a single logical model spans retail and corporate banking, insurance policies, securities holdings, and wealth management relationships — which makes it attractive to bancassurance groups and financial conglomerates that want one model across the group. It is well documented, with entity definitions, relationships, and usage guidance, and it has been implemented in production at scale at Teradata customer sites for two decades.

#### 2.3.1 Structure

| Subject Area | Contents |
|---|---|
| **Party** | Individuals, organizations, households, and the relationships among them — the foundation of customer data. |
| **Product** | The bank's or insurer's offerings: product definitions, features, and pricing structures. |
| **Agreement** | The contracts that link parties to products: accounts, policies, loans — with terms, status, and roles. |
| **Location** | Geographic and address data. |
| **Channel** | Interaction and delivery channels. |
| **Event** | Transactions, activities, and interactions between the enterprise and its customers. |

#### 2.3.2 Assessment

| Dimension | Assessment |
|---|---|
| **Strengths** | Multi-domain coverage (banking + insurance + securities + wealth) under one logical model; well-documented with detailed entity/attribute definitions; deep Teradata ecosystem integration (FSLDM is the reference model for Teradata Vantage data platforms); proven in production at large financial institutions. |
| **Weaknesses** | Teradata-centric — the model and its documentation assume the Teradata platform; commercial licensing; heavy implementation — like BDW, the full ~2,000-entity model is a large program, and 3NF logical models require dimensional marts on top for BI performance. |
| **Best for** | Teradata shops and financial services firms — especially bancassurance or diversified groups — that want a broad, proven logical model spanning multiple financial domains. |

### 2.4 FIS Banking Data Model

The **FIS banking data model** (originally the FRS banking data warehouse model, from FRSGlobal, which FIS acquired) is a banking data warehouse model covering **retail and corporate banking**, built around **customer, account, and transaction** models. FIS's heritage in regulatory reporting (FRSGlobal's flagship product was the FRS regulatory reporting suite, widely used in Europe for COREP/FINREP) means the model carries a strong reporting-oriented lineage: the customer/account/transaction core is designed to feed both management information and regulatory returns.

It is best understood as a **practical warehouse model delivered with FIS's reporting and data management tooling**, rather than a standalone industry standard like BIAN. Banks using FIS (or FRS-derived) reporting platforms typically inherit the model as the substrate of their reporting data warehouse. Coverage is strong in the retail and corporate banking core; it is less broad in capital-markets-specific subject areas than BDW or FSLDM.

| Dimension | Assessment |
|---|---|
| **Strengths** | Proven in the regulatory reporting context; retail and corporate banking core (customer/account/transaction) is clean and well tested; integrated with FIS reporting products. |
| **Weaknesses** | FIS-centric tooling; narrower domain coverage than BDW/FSLDM; commercial licensing; less community documentation than the larger reference models. |
| **Best for** | FIS/FRS customers — banks running FIS core banking or reporting platforms that want a warehouse model aligned with their vendor stack. |

### 2.5 SAS Banking Data Model

The **SAS Banking Data Model** is SAS's data model for banking analytics, focused on **credit risk, financial crime (AML), and marketing analytics**. It is designed to integrate with **SAS Risk** (SAS's risk management suite — credit risk, market risk, IFRS 9/CECL, stress testing) and **SAS Fraud** (SAS's financial crime and fraud detection framework).

Where BDW and FSLDM are enterprise warehouse models, SAS's model is an **analytics-oriented model**: it organizes data the way analytical workloads need it — customer/account/transaction structures feeding risk measures (PD, LGD, EAD, exposure), AML alerting and investigation data, and marketing segments and campaigns. Its strengths are the analytical depth in the risk and fraud domains, and seamless integration with SAS Viya and the SAS Risk/Fraud application suites. Its weaknesses mirror the vendor-model pattern: SAS-centric, commercial licensing, and narrower breadth outside the analytics domains.

| Dimension | Assessment |
|---|---|
| **Strengths** | Deep risk and financial-crime coverage; native integration with SAS Risk and SAS Fraud frameworks; strong analytical structures (measures, scores, scenarios) built in. |
| **Weaknesses** | SAS-centric; licensing; limited value outside the SAS ecosystem; not a general enterprise warehouse model. |
| **Best for** | SAS shops — banks running SAS Risk, SAS Fraud, or SAS Viya analytics that need a data model aligned with those products. |

### 2.6 Oracle Financial Services Data Model (OFSDFM)

The **Oracle Financial Services Data Model (OFSDFM)** is Oracle's banking data model, part of the **Oracle Financial Services Analytical Applications (OFSAA)** suite. It covers **risk, finance, and compliance** data and is delivered in **3NF plus dimensional** form, with **~2,000+ entities**.

OFSDFM is the substrate for OFSAA's analytical applications — funds transfer pricing, profitability, balance sheet planning, ALM, credit risk, Basel capital, IFRS 9/CECL, and regulatory reporting. Its structure is organized around the financial services data that those applications consume: customers/parties, accounts/agreements, financial contracts, balances and flows, and the dimensional structures (legal entity, product, geography, time) used to aggregate them.

| Dimension | Assessment |
|---|---|
| **Strengths** | Deep Oracle ecosystem integration; strong risk and compliance focus — the model is explicitly built to feed OFSAA risk, finance, and regulatory applications; regulatory reporting alignment (Basel, IFRS 9/CECL, local returns); ~2,000+ entities of proven coverage. |
| **Weaknesses** | Oracle-centric — the model is documented and tuned for Oracle databases and OFSAA; commercial licensing; the risk/finance focus means it is less of a general-purpose enterprise warehouse model than BDW. |
| **Best for** | Oracle shops — banks running Oracle databases and OFSAA applications that want risk, finance, and compliance data modeled consistently with those tools. |

### 2.7 Banking Model Comparison

| Model | Vendor | Domains Covered | Entity Count | Format | Strengths | Weaknesses | Best For |
|---|---|---|---|---|---|---|---|
| **BIAN** | Consortium (bian.org) | Full banking service landscape (8 business domains, ~400 service domains) | ~400 service domains; business objects + qualifiers | Service-oriented logical model (business objects, behavioral qualifiers) | Vendor-neutral, industry-backed, aligns architecture/APIs/data | Complex, abstract, needs physical interpretation | Banks standardizing architecture & APIs |
| **IBM BDW** | IBM | ~20 banking subject areas (Party, Agreement, Product, Location, Channel, Event, Condition, Asset, Finance, Risk, Marketing) | ~2,000 entities, 7,000+ attributes | 3NF + dimensional | Comprehensive, mature (30+ yrs), IBM ecosystem | IBM-centric, heavy, licensing | IBM shops, comprehensive warehouse |
| **Teradata FSLDM** | Teradata | Banking + insurance + securities + wealth (6 subject areas) | ~2,000 entities | 3NF logical model | Multi-domain, well-documented, proven | Teradata-centric, licensing, heavy | Teradata shops, diversified FS groups |
| **FIS** | FIS | Retail & corporate banking (customer/account/transaction) | Core banking warehouse scope | 3NF warehouse model | Reporting-proven, clean core | FIS-centric, narrower scope | FIS/FRS platform customers |
| **SAS Banking Model** | SAS | Credit risk, financial crime, marketing analytics | Analytics scope | Analytics-oriented model | Deep risk/fraud coverage, SAS Risk/Fraud integration | SAS-centric, licensing | SAS Risk/Fraud/Viya shops |
| **Oracle OFSDFM** | Oracle | Risk, finance, compliance (OFSAA) | ~2,000+ entities | 3NF + dimensional | Oracle ecosystem, risk/compliance focus, regulatory alignment | Oracle-centric, licensing | Oracle/OFSAA shops, risk analytics |

**Reading the table:** the models cluster into three purposes. (1) *Architecture standards* (BIAN) tell you what the bank's capabilities and business objects *are*; they do not ship a physical schema. (2) *Enterprise warehouse reference models* (IBM BDW, Teradata FSLDM, FIS) give you a ready-made 3NF blueprint of the whole bank that you subset and implement; they assume a warehouse platform and require marts on top for BI. (3) *Application-aligned models* (SAS, Oracle OFSDFM) organize data for a specific vendor's analytical applications. Your choice is therefore driven less by "which is best" and more by "what is my platform, and what am I modeling for" — see the selection guide in Section 9.

---

## 3. Insurance Data Models

Insurance has its own set of industry-standard data models, led by **ACORD** — the industry's data standards body — plus vendor warehouse models from IBM, SAS, and Guidewire. The insurance data landscape differs from banking in one important respect: policy/claims/billing data is highly **product-shaped** (life vs. property/casualty vs. health have very different structures), so the industry leans heavily on exchange standards (ACORD XML/JSON) to normalize communication between carriers, agencies, and MGAs, and on warehouse models to consolidate the mess.

### 3.1 ACORD — Insurance Data Standards

**ACORD** (Association for Cooperative Operations Research and Development, acord.org) is the **insurance industry's data standards organization**. It defines the data standards used across the insurance value chain — policy administration, claims, billing, and agency-carrier communication — and is the closest thing insurance has to a canonical data model standard.

#### 3.1.1 ACORD Standards Portfolio

| Standard | Purpose |
|---|---|
| **ACORD Data Standards** | The core data standards for policy, claims, and billing — the canonical definitions of insurance business objects and their attributes. |
| **ACORD XML** | XML message schemas for policy administration, claims, and billing exchanges — the dominant machine-to-machine format in insurance. |
| **ACORD LOMA Systems** | Standards for life, annuity, and health insurance (developed jointly with LOMA, the life insurance industry association). |
| **ACORD RPC** | Real-time policy lookup — a lightweight service protocol for instant policy verification (e.g., at point of sale or for electronic proof of insurance). |
| **ACORD Forms** | Digitized versions of the industry's paper forms — the standard form library (ACORD 125, ACORD 126, etc.) used across US agencies and carriers. |

#### 3.1.2 Key ACORD Structures

ACORD's data standards define the canonical insurance business objects:

| Structure | Contents |
|---|---|
| **Policy** | Policy number, insured party, coverage, premium, policy period (effective/expiration dates), product/line of business, policy status. |
| **Claim** | Claim number, policy reference, claimant, loss details (date, cause, location, description), coverage reference, payment/payout information, claim status. |
| **Party** | The roles and entities in the insurance relationship: insured, agent, broker, adjuster, attorney, and their contact/identity data. |
| **Coverage** | Coverage type, limits, deductibles, endorsements — the detailed terms that define what is and isn't covered. |

#### 3.1.3 ACORD for Data Modeling

ACORD XML schemas effectively define **canonical structures for insurance data exchange**. For a data architect, ACORD serves several roles:

- **Exchange contract:** ACORD XML/JSON messages define what a policy, claim, or billing transaction looks like when it crosses system boundaries. Carrier-agency communication (the classic ACORD use case — agencies submit applications and claims to carriers) runs on these schemas.
- **Canonical reference:** when an insurer builds a CDM (Section 1), ACORD's policy/claim/party/coverage structures are the natural starting skeleton — the same way BIAN's business objects serve banking.
- **Field-level vocabulary:** ACORD's attribute definitions (e.g., the exact meaning of `PolicyEffectiveDate`, `Limit`, `Deductible`) provide a ready-made glossary that reduces definitional drift.
- **Regulatory alignment:** ACORD structures align with state (US) and federal regulatory reporting needs, and increasingly with IFRS 17 data collection requirements.

#### 3.1.4 Assessment

| Dimension | Assessment |
|---|---|
| **Strengths** | Industry standard — set by the industry body itself, not a vendor; comprehensive coverage of policy, claims, and billing; very widely adopted across carriers, agencies, MGAs, and software vendors; regulatory alignment baked into the standard. |
| **Weaknesses** | Complex — the XML schemas are notoriously large and deep (thousands of elements across the portfolio); US-centric — ACORD's forms and much of its framing target the US market; evolving — still XML-heavy, with newer JSON standards (ACORD's JSON efforts) still maturing. |
| **Best for** | Insurance data exchange, policy administration, claims processing, and agency-carrier integration — anywhere insurance data must cross a system boundary in a standard way. |

### 3.2 IBM Insurance Information Warehouse (IIW)

The **IBM Insurance Information Warehouse (IIW)** is IBM's reference data model for insurance data warehousing — the insurance counterpart of BDW (Section 2.2). It covers **policy, claim, party, product, channel, finance, and risk** subject areas with **~1,000+ entities**, and supports **life, annuity, health, and property/casualty** lines of business.

IIW follows the same design philosophy as BDW: a comprehensive 3NF reference model that the insurer subsets and tailors, delivered with IBM tooling (DataStage, Db2, InfoSphere). Its subject areas map cleanly onto the insurance business:

| Subject Area | Contents |
|---|---|
| **Policy** | Policy master data: policy number, product/line, policy period, status, premium schedules. |
| **Claim** | Claims data: claim number, policy reference, claimant, loss, reserves, payments. |
| **Party** | Insureds, policyholders, agents, brokers, beneficiaries, and their relationships. |
| **Product** | Product definitions and hierarchies across life, annuity, health, and P&C lines. |
| **Channel** | Distribution channels: agency, broker, bancassurance, direct. |
| **Finance** | Premium accounting, commissions, general ledger structures. |
| **Risk** | Underwriting, actuarial, and reserving data. |

| Dimension | Assessment |
|---|---|
| **Strengths** | Comprehensive insurance warehouse model; multi-line support (life, annuity, health, P&C); mature and proven in IBM implementations; consistent with BDW for bancassurance groups running both. |
| **Weaknesses** | IBM-centric; commercial licensing; ~1,000+ entities means significant tailoring effort. |
| **Best for** | IBM shops and insurers (or bancassurance groups) wanting a comprehensive insurance warehouse reference model. |

### 3.3 SAS Insurance Data Model

The **SAS Insurance Data Model** is SAS's insurance data model for analytics, focused on **actuarial, underwriting, and claims analytics**. Like the SAS Banking Data Model (Section 2.5), it is analytics-oriented and integrates with SAS's analytics platform (SAS Viya) and industry solutions — SAS Insurance Risk, SAS Claims, and actuarial tooling.

Its value is in the analytical structures: policy/claim data organized to feed loss ratios, reserving analyses, pricing and underwriting models, and claims analytics. The model is narrower than IIW as an enterprise warehouse model but deeper in the actuarial/analytics structures that SAS customers actually run. The same vendor-model trade-offs apply: SAS-centric, commercial licensing, best value inside the SAS ecosystem.

### 3.4 Guidewire InfoCenter

**Guidewire InfoCenter** is the data warehouse model for the **Guidewire platform** — the dominant policy administration and claims system vendor in property/casualty insurance. InfoCenter is a pre-built data warehouse (with ETL) for **PolicyCenter, ClaimCenter, and BillingCenter** data, covering **policy, claim, and billing** data in a dimensional warehouse structure.

For a P&C insurer running Guidewire, InfoCenter is effectively the *de facto* canonical layer: it standardizes the Guidewire source data into warehouse structures (facts and dimensions) and provides out-of-the-box reporting. Its strengths: deep Guidewire ecosystem integration, fast time-to-value for Guidewire shops, and consistent structures for policy/claim/billing analytics. Its weaknesses: it covers only Guidewire data (other sources still need their own models), and it ties the insurer to the Guidewire toolchain. It is best for Guidewire-centric P&C insurers that want a warehouse without building the policy/claim/billing models themselves.

### 3.5 Insurance Model Comparison

| Model | Vendor | Domains Covered | Entity Count | Format | Strengths | Weaknesses | Best For |
|---|---|---|---|---|---|---|---|
| **ACORD** | ACORD (industry body) | Policy, claims, billing, party, coverage (all lines) | Standard portfolio (XML/JSON schemas + forms) | Exchange standards (XML/JSON) | Industry standard, comprehensive, widely adopted, regulatory alignment | Complex schemas, US-centric, XML-heavy | Insurance data exchange, agency-carrier integration |
| **IBM IIW** | IBM | Policy, claim, party, product, channel, finance, risk | ~1,000+ entities | 3NF warehouse model | Comprehensive, multi-line (life/annuity/health/P&C) | IBM-centric, licensing | IBM shops, insurance warehouse |
| **SAS Insurance** | SAS | Actuarial, underwriting, claims analytics | Analytics scope | Analytics-oriented model | Deep actuarial/claims analytics | SAS-centric, licensing | SAS analytics shops |
| **Guidewire InfoCenter** | Guidewire | Policy, claim, billing (PolicyCenter/ClaimCenter/BillingCenter) | Warehouse scope | Dimensional warehouse + ETL | Deep Guidewire integration, fast time-to-value | Guidewire data only | Guidewire-centric P&C insurers |

---

## 4. Modern Data Modeling Approaches

The classical industry models above answer "what should the enterprise data look like". The modern approaches in this section answer "how should we build and operate the data platform that holds it" — and they change the physical modeling strategy significantly. Data Vault 2.0 is a physical modeling methodology; data mesh is an organizational operating model; data fabric is a technology architecture. All three are compatible with the CDM concept from Section 1, and all three are being adopted in banking and insurance today.

### 4.1 Data Vault 2.0

**Data Vault 2.0** is an enterprise data warehouse modeling methodology built on the **hub-link-satellite** pattern. It was designed to solve the problems that 3NF and dimensional models create in large, multi-source, history-hungry enterprises — exactly the banking and insurance situation.

#### 4.1.1 The Core Pattern

| Component | Purpose | Example |
|---|---|---|
| **Hub** | Stores the **business keys** — the natural identifiers of core business entities — with no descriptive attributes. | `H_CUSTOMER` (business key: customer ID), `H_ACCOUNT` (account ID), `H_PRODUCT` (product code) |
| **Link** | Models **relationships between hubs** — many-to-many associations with no attributes. | `L_CUSTOMER_ACCOUNT` (customer ↔ account), `L_ACCOUNT_PRODUCT` (account ↔ product) |
| **Satellite** | Stores **descriptive attributes with history** — the context and changes over time for a hub or link, keyed by load date and source. | `S_CUSTOMER_DEMOGRAPHICS` (name, address, segment, with history), `S_ACCOUNT_BALANCE`, `S_PRODUCT_ATTRIBUTES` |

A simplified banking example:

```
H_CUSTOMER (CustomerID)  ◀── L_CUSTOMER_ACCOUNT ──▶  H_ACCOUNT (AccountID)
      │                                                  │
S_CUSTOMER_DEMOGRAPHICS                             S_ACCOUNT_BALANCE
S_CUSTOMER_KYC                                       S_ACCOUNT_TERMS
```

Satellites capture **full history**: every change to a customer's address or an account's terms is recorded as a new satellite row with its load timestamp and source system. Nothing is overwritten — which is precisely what regulators want when they ask for an audit trail (BCBS 239 lineage and change history).

#### 4.1.2 Why Data Vault Fits Banking and Insurance

| Advantage | What it means in practice |
|---|---|
| **Scalable** | Hubs, links, and satellites can be added incrementally — a new source system or new subject area means adding components, not redesigning the model. |
| **Auditability** | Full history of every key, relationship, and attribute. Every row carries `LOAD_DTS` (load timestamp) and `RECORD_SOURCE` — a built-in audit trail that directly supports BCBS 239 and MAS requirements. |
| **Parallel loading** | Satellites load independently and in parallel — no re-processing of the warehouse when a new satellite is added. Load windows shrink dramatically. |
| **Source-agnostic** | Multiple source systems can feed the same hub without conflict; conflicts are resolved in satellites (each source gets its own satellite, e.g., `S_CUSTOMER_CORE`, `S_CUSTOMER_CRM`), preserving the audit trail of which system said what. |
| **Resilient to change** | Schema changes in sources map to new satellites — the model absorbs change instead of breaking. |

| Disadvantage | Mitigation |
|---|---|
| **Complex** | A data vault has more tables than an equivalent 3NF or dimensional model (often 2–3×). Requires trained modelers and disciplined conventions. |
| **Not for direct querying** | Raw vault tables are not business-friendly; analytics need a **presentation layer** (dimensional marts or business views) on top — the "gold" layer in medallion terms. |
| **Requires tooling discipline** | Load patterns must be consistent (hash keys, load dates, record sources). Automation tooling (dbt packages, WhereScape, custom frameworks) is strongly recommended. |

**Where it sits:** the data vault is the physical backbone of the **silver/canonical layer** — the hub/link/satellite structures *implement* the CDM's entities and relationships, while satellites carry the history the CDM's consumers need. Presentation marts (gold) serve BI and reporting.

### 4.2 Data Vault vs. 3NF vs. Dimensional

| Dimension | Data Vault 2.0 | 3NF (e.g., IBM BDW, FSLDM) | Dimensional (Star Schema) |
|---|---|---|---|
| **Structure** | Hubs (keys), links (relationships), satellites (attributes+history) | Fully normalized relational model | Facts (measures) + dimensions (context) |
| **Normalization** | Hybrid — keys normalized, attributes de-normalized into satellites | Fully normalized (3NF) | Deliberately de-normalized |
| **History** | Full history by design (every change recorded) | Limited — history only if explicitly modeled (type-2 patterns) | Limited — typically type-1/type-2 on dimensions |
| **Loading** | Incremental, parallel, no re-processing | Complex incremental loads, often full refresh for big tables | Incremental ETL into facts/dimensions |
| **Querying** | Not suitable for direct querying — needs presentation layer | Direct queryable but complex joins across many tables | Excellent for BI — pre-joined, aggregatable |
| **Auditability** | Built-in (load timestamps, record source, hash keys) | Requires explicit audit fields and design effort | Requires explicit audit fields |
| **Best for** | Enterprise warehouse foundation with regulatory history requirements (BCBS 239 audit trails) | Transactional systems (OLTP) and reference-model warehouses | BI, reporting, analytics marts |

**In practice, these are complementary, not competing:** OLTP systems stay 3NF; the enterprise warehouse core is often data vault (for history and auditability); BI consumes dimensional marts built from the vault. The industry reference models (BDW, FSLDM) provide the *domain vocabulary* for all three layers.

### 4.3 Data Mesh

**Data mesh** is an organizational and architectural approach that treats data as a product and distributes ownership to the domains that know the data best. Its four principles:

1. **Domain-oriented data ownership** — the business domain (payments, cards, risk, compliance) owns its data end to end, instead of a central data team owning everything.
2. **Data as a product** — each domain publishes its data as a *product* with SLAs, versioning, documentation, quality guarantees, and a clear owner — the same care a software product gets.
3. **Federated computational governance** — global standards (naming, security, privacy, interoperability) are set centrally, while domains retain autonomy in how they implement them.
4. **Self-serve data platform** — a central platform team provides the tools (ingestion, storage, transformation, cataloging, access control) so domain teams can build and operate their own data pipelines.

#### 4.3.1 Banking Application

In a bank, the mesh maps onto the business's natural domains:

| Domain | Data Product | Example Contents |
|---|---|---|
| Customer | Customer 360 product | Party master, relationships, segments, KYC — published with a contract and SLA |
| Transactions | Transaction product | Posted transactions, events, enriched with reference data |
| Risk | Risk product | Exposures, limits, PD/LGD/EAD, stress results |
| Compliance | Compliance product | AML alerts, sanctions hits, surveillance findings |
| Finance | Finance product | GL, cost allocation, P&L |
| Payments | Payments product | Payment messages, statuses, exceptions |

Each domain **publishes well-defined data products** to the shared platform; the central platform provides tools and governance; **cross-domain joins happen at consumption time** — a risk analyst joins the customer product with the transaction product in their own workspace, rather than a central warehouse team building one giant integrated schema.

**Why it appeals to banks:** domain teams know their data best; ownership and accountability are explicit (who owns customer data quality? the customer domain — not "IT"); regulatory ownership maps to business ownership; and M&A integration is localized (acquired banks become new domains publishing products). **Why it is hard:** federated governance is genuinely difficult in regulated industries — a central authority (CDO office) must still enforce BCBS 239 lineage, data quality standards, and access control; and consumers must accept joining data themselves, which requires a mature self-serve platform. Mesh without a **shared vocabulary (the CDM!)** produces incompatible products that no one can join.

### 4.4 Data Fabric

**Data fabric** is a technology-centric architecture for **metadata-driven data integration**. Where mesh changes *who* owns data, fabric changes *how* data is integrated — using automation, active metadata, and virtualization rather than manual ETL and physical consolidation.

Core elements:

| Element | Description |
|---|---|
| **Active metadata** | Metadata is treated as a first-class, continuously updated asset — lineage, profiles, quality scores, usage — not static documentation. |
| **Knowledge graphs of data assets** | A graph of data assets and their relationships (systems → tables → columns → reports → owners), enabling automated discovery and impact analysis. |
| **Automated discovery, classification, and governance** | Machine-driven identification of data, auto-classification (PII, confidential), and policy application. |
| **Virtualization layer** | A unified access layer that lets consumers query across heterogeneous sources without physically copying data. |
| **Data preparation and orchestration** | Automated pipelines that can materialize data when virtualization is not enough (performance, history). |

#### 4.4.1 Banking Application

- A fabric layer sits **over heterogeneous systems** — core banking, risk, CRM, payments — abstracting their differences.
- Consumers get a **unified view without physical consolidation**: a risk analyst queries "customer exposures" and the fabric federates across the risk warehouse, core banking, and derivatives systems in real time.
- **Regulatory reporting via the fabric layer**: reports are defined once against the logical view; the fabric resolves source access, applies lineage (BCBS 239), and can physically materialize results where regulators require it.
- **Automated lineage and classification** directly support BCBS 239 (traceability) and GDPR (find all PII for a data subject across the estate).

**Why it appeals to banks:** speed — no multi-year consolidation project before you can answer a cross-system question; agility for regulatory change (new reporting requirements resolve against the fabric's logical layer); and lower storage/replication costs. **Why it is hard:** virtualization performance on high-volume banking data (transactions, positions) is a real constraint — fabric designs usually end up *hybrid*, virtualizing reference/semantic data while physically materializing high-volume transaction data; and a fabric still needs governance (the metadata is only as good as the curation behind it).

### 4.5 Data Fabric vs. Data Mesh

| Dimension | Data Fabric | Data Mesh |
|---|---|---|
| **Orientation** | Technology-centric | Organization-centric |
| **Core mechanism** | Automated metadata, knowledge graphs, virtualization | Domain ownership, data products, federated governance |
| **Primary goal** | Automate integration and access across systems | Distribute ownership and accountability to domains |
| **Who does the work** | Platform technology (metadata engine, virtualization) | Domain teams building data products |
| **Governance** | Centrally automated (policy as code, auto-classification) | Federated (global standards + local autonomy) |
| **Data movement** | Minimized — virtualize where possible | Domains publish products (physical or logical) |
| **Banking example** | Virtualized risk-reporting layer over core banking + risk + CRM | Risk domain publishing an exposure data product |

Both address **data sprawl** — the proliferation of ungoverned, disconnected data across the enterprise — but from opposite ends: fabric attacks it with automation and a unified access layer; mesh attacks it with ownership and product discipline. They are complementary: a mesh's self-serve platform is often built *on* a fabric (the platform team provides virtualization and metadata automation to the domains), and both require a **CDM** as the shared vocabulary — mesh to keep domain products interoperable, fabric to define the canonical views it exposes.

---

## 5. Regulatory Data Model Requirements

Regulation is the single strongest driver of data modeling decisions in banking and insurance. The models and approaches in Sections 2–4 are chosen, at least in part, because of what regulators require: consistent definitions, complete and accurate data, lineage, history, and timely aggregation. This section covers the regulations that most shape the data model: BCBS 239, MAS guidelines, GDPR/PDPA, Solvency II, IFRS 17, and IFRS 9/CECL.

### 5.1 BCBS 239 — Risk Data Aggregation and Reporting

**BCBS 239** (*Principles for effective risk data aggregation and risk reporting*, issued by the Basel Committee after the 2008 financial crisis) is the foundational regulatory driver for banking data architecture. It applies to globally systemically important banks (G-SIBs) and, through local adoption, to a much wider set of banks — including MAS-regulated banks in Singapore.

Its **14 principles** cluster into four themes that are, in effect, data model requirements:

| Theme | Principles | Data Model Implication |
|---|---|---|
| **Governance & infrastructure** | Data architecture and IT infrastructure (P3–P5) | Consistent data definitions; a **single source of truth**; enterprise-wide data model; controlled data ownership |
| **Risk data aggregation** | Accuracy, completeness, timeliness, adaptability (P6–P9) | Complete, validated data; aggregation along any dimension (product, customer, legal entity, geography) at any time |
| **Risk reporting** | Accuracy, comprehensiveness, clarity, frequency, distribution (P10–P13) | Reports traceable to source data; drill-down capability |
| **Supervisory review** | Remediation and supervisory tools (P1, P2, P14) | Evidence of lineage and quality — an audit trail |

#### 5.1.1 The Four Capabilities Regulators Inspect

- **Data lineage** — the ability to trace every number in a report back through transformations to the source systems and records. This requires *documented, machine-readable mappings* — exactly the source-to-target mapping artifacts of a CDM (Section 1.3) — and a lineage tool that captures them.
- **Data quality** — accuracy, completeness, timeliness, and adaptability of risk data. Requires a **data quality framework**: business rules, validation checks, and dashboards that monitor the canonical layer continuously. Quality rules are attached to CDM attributes (e.g., "exposure amount must reconcile to source systems within tolerance").
- **Data architecture** — consistent data definitions and a single source of truth: a CDM. Regulators will ask "where is the authoritative definition of exposure?" and the answer must be "the enterprise data model", not "each system has its own".
- **Risk data aggregation** — the ability to produce timely, accurate, complete risk reports (COREP, FINREP, MAS returns, CCAR in the US, and internal risk reports) — including **on-demand aggregation** for stress testing and ad-hoc supervisory requests. This drives the modeling of risk data structures (below).

#### 5.1.2 Canonical Risk Data Structures

BCBS 239 makes the following structures effectively mandatory in the enterprise data model:

| Structure | Contents |
|---|---|
| **Exposure** | The amount at risk for each counterparty/transaction: current exposure, potential future exposure, netting sets, collateral-adjusted exposure. |
| **Collateral** | Collateral assets, valuation, haircuts, margin calls, collateral agreements (CSA terms). |
| **Counterparty** | The party data for each counterparty: legal entity, group structure, credit ratings, jurisdiction, sector. |
| **Position** | Positions per instrument/product/portfolio at a point in time, with valuation inputs. |

These are the entities that risk aggregation, stress testing, and supervisory reporting all consume — and they must be modeled consistently across credit, market, and operational risk (the "risk data hub" concept).

#### 5.1.3 Audit Trail

BCBS 239 demands an audit trail: data lineage (source-to-report), change history, and versioning of data and reports. This is where the modeling choice matters most: a **data vault** (Section 4.1) provides full history natively (load timestamps, record source, satellites never overwritten); a 3NF model requires explicit bi-temporal/audit fields; a dimensional model requires careful type-2 handling. In practice, many banks moving toward BCBS 239 compliance adopt data vault precisely for this built-in auditability.

### 5.2 MAS Guidelines (Singapore)

For a Singapore-based bank, the **Monetary Authority of Singapore (MAS)** guidelines layer local requirements on top of the Basel framework:

| Guideline | Focus | Data Model Implication |
|---|---|---|
| **MAS Notice 637** | Risk management (board oversight, risk governance, stress testing, risk data aggregation aligned to BCBS 239) | Consistent risk data model; BCBS 239-style lineage and aggregation capabilities |
| **MAS Notice 612** | AML/CFT (customer due diligence, transaction monitoring, record keeping) | KYC data modeled as first-class entities (identity documents, risk rating, PEP flags — see Section 6.5); transaction data retained and queryable for monitoring and investigation |
| **Data residency requirements** | Certain data must remain within Singapore (or approved jurisdictions) | Data platform design must respect residency boundaries — physical placement of the canonical layer, replication policies, and access controls become data-model-adjacent architecture decisions |
| **MAS Technology Risk Management (TRM)** | Outsourcing, cyber resilience, business continuity | The data platform itself (including the CDM) must be resilient, recoverable, and tested — influencing where and how the model is operated |

**Practical consequence:** a Singapore bank's data model must satisfy both global (BCBS 239, IFRS 9) and local (MAS 637, MAS 612, residency, TRM) requirements simultaneously — which argues for a single enterprise CDM with clear regulatory mappings, rather than per-regulation silos.

### 5.3 GDPR and PDPA — Privacy Regulations

**GDPR** (EU) and **PDPA** (Singapore's Personal Data Protection Act) constrain what personal data can be stored, for how long, and what rights data subjects have. These constraints interact with the data model in specific ways:

| Requirement | Data Model Implication |
|---|---|
| **Right to be forgotten / erasure** | Data subject erasure requests must be executable. This is a **challenge for data vault's full-history design** — history that never deletes conflicts with erasure. Practical responses: erasure flags + access-control suppression (logical deletion), privacy-safe data zones, and explicit archival/erasure processes on satellites holding personal data. |
| **Consent management** | Consent must be modeled as a **data entity**: consent records (what was consented to, when, by whom, through which channel, revocation status) linked to the party. Every downstream use of personal data checks consent. |
| **Data minimization** | Store only the data necessary for the stated purpose — which pushes back against "capture everything" warehouse designs. Data models should distinguish *required* attributes from *nice-to-have* and justify retention. |
| **Data subject access requests (DSAR)** | Locating all of a person's data across the estate requires **cross-domain tracing** — a catalog and lineage layer (or fabric knowledge graph, Section 4.4) that can enumerate all data about a party, including in satellites, archives, and backups. |
| **Purpose limitation & classification** | The model must tag personal data (PII classification) and its permitted purposes — a driver for metadata-driven classification in the fabric, and for row-level/column-level access control in the platform. |

**Design response:** the modern pattern is *tiered storage with erasure capability* — an operational zone (limited retention), a warehouse zone (pseudonymized where possible), and an archive zone (encrypted, with strict access) — with the data model recording retention class, consent status, and erasure triggers per party.

### 5.4 Solvency II (EU Insurance)

**Solvency II** is the EU's risk-based capital regime for insurers, structured in three pillars that each impose data requirements:

| Pillar | Contents | Data Model Implication |
|---|---|---|
| **Pillar 1 — Quantitative** | Technical provisions, Solvency Capital Requirement (SCR), Minimum Capital Requirement (MCR) | **Risk data aggregation**: underwriting, claims, and reserving data must support actuarial calculations; **valuation data**: market data, discount curves, risk-free rate term structures; cash-flow projections per policy group |
| **Pillar 2 — Governance** | ORSA (Own Risk and Solvency Assessment), risk management, internal controls | **Governance data**: risk registers, ORSA documentation, model validation evidence — modeled as auditable artifacts with lineage |
| **Pillar 3 — Disclosure** | SFCR (Solvency and Financial Condition Report), RSR (Regular Supervisory Report) | Reporting data must be traceable (lineage) and reconcilable — the same BCBS 239-style discipline applied to insurance |

**Data model implications in practice:**

- **Risk data aggregation** — underwriting, claims, and reserving data must roll up by line of business, segment, and entity with the completeness and timeliness a supervisor expects.
- **Valuation data** — market data, discount curves, and risk-free rates must be stored with history (curve versions, sources) — a classic reference-data-plus-history modeling problem (Section 6.1).
- **Governance data** — risk registers, ORSA documentation, and validation evidence become modeled entities with owners, versions, and approval states.

### 5.5 IFRS 17 — Insurance Contracts

**IFRS 17** (effective 2023) replaced IFRS 4 as the accounting standard for insurance contracts. It fundamentally changes how insurers must measure and report contracts, and it is one of the most data-intensive accounting changes the industry has seen:

**Key concepts:**

| Concept | Meaning |
|---|---|
| **Expected cash flows** | Probability-weighted, discounted future cash flows from the contract group (premiums, claims, expenses). |
| **Discount rates** | The rate curves used to discount those cash flows — including the illiquidity premium adjustment. |
| **Risk adjustment** | The compensation for non-financial risk (explicit, modeled per group). |
| **Contractual Service Margin (CSM)** | The unearned profit recognized over the coverage period — the "liability" that makes IFRS 17 distinctive. |
| **Measurement models** | **BBA** (Building Block Approach — the general model, also called the General Measurement Approach, GMA), **PAA** (Premium Allocation Approach — a simplification for short-duration contracts), and **VFA** (Variable Fee Approach — for direct participating contracts). |

**Data model needs:**

- **Cash flow projections per contract group** — full projection grids (cash flows by period, by driver: claims, premiums, expenses) per group of contracts, recomputed at each reporting date.
- **Discount rate curves** — the full term structure per currency and per cohort, with history and source.
- **CSM tracking** — the CSM balance per group, its roll-forward (new business, interest accretion, changes in estimates, recognition), and its allocation to P&L.
- **Measurement model support** — the model must support BBA, GMA/PAA, and VFA cohorts simultaneously, with the election per group stored as reference data.

**Practical consequence:** IFRS 17 essentially forced insurers to build **granular, well-governed data models** — contract-group-level data with full history and lineage — because the calculations are re-run at every reporting date and must be auditable. The same canonical-model discipline (Section 1) applies: policy/claim data from administration systems is mapped into IFRS-17-specific canonical structures, and the results feed both the general ledger and disclosures.

### 5.6 IFRS 9 / CECL — Expected Credit Loss

**IFRS 9** (banks, effective 2018) and **CECL** (US, effective 2023) replaced incurred-loss provisioning with **expected credit loss (ECL)** — forward-looking provisioning based on probability-weighted outcomes.

**Data model needs:**

| Need | Contents |
|---|---|
| **PD / LGD / EAD inputs** | Probability of Default, Loss Given Default, Exposure at Default per facility/obligor — including historical data used to estimate them (default histories, recovery data, exposure profiles). |
| **Staging** | The three-stage model: **Stage 1** (performing — 12-month ECL), **Stage 2** (significant increase in credit risk — lifetime ECL), **Stage 3** (credit-impaired — lifetime ECL). The data model must store stage per facility per period, plus the triggers (SICR criteria, days past due, forbearance). |
| **Macroeconomic scenarios** | Multiple forward-looking scenarios (baseline, upside, downside) with their weights — GDP, unemployment, rates — and the scenario-to-PD mapping (the "through-the-cycle to point-in-time" transformation). |
| **ECL computation results** | ECL per facility per stage per scenario, the probability-weighted ECL, and the roll-forward (movements between stages, model changes) — all stored with lineage for audit. |

**Practical consequence:** IFRS 9/CECL made the **risk data model** a finance-critical asset: the same exposures, counterparties, and collateral structures required by BCBS 239 (Section 5.1) are the inputs to the ECL engine, and the results must reconcile to the general ledger. This is a powerful argument for one canonical risk/finance data model rather than separate risk and finance warehouses.

### 5.7 Regulatory Requirements Summary

| Regulation | Sector | Core Data Model Demand |
|---|---|---|
| **BCBS 239** | Banking | Enterprise CDM, lineage, quality framework, on-demand aggregation, audit trail |
| **MAS 637 / 612 / TRM** | Banking (SG) | BCBS 239 alignment, KYC/AML data structures, data residency, resilience |
| **GDPR / PDPA** | Both | Erasure capability, consent as entity, minimization, DSAR tracing, PII classification |
| **Solvency II** | Insurance (EU) | Risk data aggregation, valuation data, governance artifacts, Pillar 3 traceability |
| **IFRS 17** | Insurance | Cash-flow projections per group, discount curves, CSM roll-forward, measurement models |
| **IFRS 9 / CECL** | Banking | PD/LGD/EAD, staging, macro scenarios, ECL results with reconciliation |

The common thread: **consistent definitions (CDM), complete history (data vault / bi-temporal modeling), and traceability (lineage)** are no longer optional — they are the price of doing regulated business.

---

## 6. Data Model Implementation Patterns

This section covers the recurring **modeling patterns** that appear in every banking and insurance data model, regardless of whether the underlying approach is 3NF, dimensional, data vault, or an industry reference model. These are the patterns a data architect reaches for when designing the canonical layer.

### 6.1 Reference Data Management

Reference data is the "lookup" data that everything else depends on — and getting it canonical is a prerequisite for everything else. Four reference domains dominate:

**Customer master data (party model).** The party model is the anchor of the enterprise data model. Core structures:

- **Party** — the super-type covering `Person` and `Organization` (the classic party model pattern: one `Party` table with type-specific subtype tables, or a single table with a party-type discriminator).
- **Party relationships** — how parties relate: group→subsidiary, customer→guarantor, employer→employee, beneficial ownership. Relationship modeling (recursive party-to-party links with role and validity dates) is what makes group-level risk aggregation and KYC beneficial-owner lookups possible.
- **Party roles** — a party plays roles: account holder, signatory, authorized user, beneficial owner, guarantor, agent.

**Product reference data.** The product catalog: product definitions, product hierarchy (product line → product group → product → product variant), pricing attributes, and product-to-agreement relationships. Both BIAN (Product Directory service domain) and the warehouse models (BDW/FSLDM `Product` subject area) treat product as a first-class entity. Product data drives profitability analytics, cross-sell, and regulatory segmentation (product-based risk weights).

**Geography.** Country, region, state/province, city, and address structures with the ISO code sets (ISO 3166 countries, ISO 4217 currencies) as the canonical values. Location hierarchies support legal-entity, branch, and jurisdiction-based reporting (residency, sanctions, tax).

**Currency.** ISO 4217 currency codes, currency hierarchies (currency → currency group), and FX rate reference data (rate types, rate sources, rate dates). Every financial measure in the model references currency, and rate tables must be history-preserving for point-in-time conversion.

**Calendar.** Business calendars (working days per jurisdiction, holiday calendars), reporting periods (monthly close calendar, regulatory reporting periods), and date-dimension structures used to align time across systems.

**Reference data pattern in the model:** reference data is usually modeled as slowly-changing dimensions (SCD type 2 for history) or as data-vault satellites off reference hubs. The key discipline is **central ownership**: one system of record for each reference domain, published to all consumers — a reference data management (RDM) function that the CDM's governance model depends on.

### 6.2 Transaction Data Modeling

Transaction data is the highest-volume, highest-value data in a bank. Core patterns:

**Account model.** The account is the container of value and activity:

- **Account** — the account master: account identifier, product reference, customer/party reference, status, currency, open date.
- **Account balance** — point-in-time balances: balance date, balance type (available, ledger, cleared), balance amount.
- **Account transaction** — the posting history against the account.
- **Account limit** — limits and their utilization: credit limits, withdrawal limits, overdraft limits.

**Transaction model.** The transaction is the atomic financial fact:

| Attribute | Meaning |
|---|---|
| Transaction ID | Unique identifier (often surrogate + business reference) |
| Account reference | The account(s) the transaction affects (debit/credit sides) |
| Transaction type | Deposit, withdrawal, payment, fee, interest, transfer — from a canonical code set |
| Amount | Signed or unsigned amount with direction |
| Currency | ISO 4217 code; original currency vs. settlement currency |
| Timestamp | Value date, booking date, timestamp of capture |
| Status | Posted, pending, reversed, failed |
| Reference | The business reference: payment reference, cheque number, authorization code |
| Counterparty | The other side: payee/payer account, counterparty party reference, or clearing counterparty |

**Event model.** At a higher level, transactions are one kind of *event*: the event model generalizes to event type (transaction, interaction, status change, alert), event time (occurred, captured, processed), event source (system, channel, batch job), and event payload. Event-based modeling (Section 6.7) treats events as the primary records and derives state from them.

### 6.3 Balance and Position Modeling

Balances and positions are *derived, time-varying* facts — the modeling challenge is capturing time correctly:

| Pattern | Description |
|---|---|
| **Point-in-time balances** | A balance as of a specific date/time: `AccountBalance(AccountID, BalanceDate, BalanceAmount, BalanceType)`. The canonical pattern — daily balance snapshots per account per balance type. |
| **Time series of balances** | The sequence of balances over time — for interest accrual, average-balance calculations, liquidity reporting (LCR/NSFR), and balance-sheet history. |
| **Position hierarchy** | Positions roll up: instrument → product → portfolio → desk → legal entity → group. The model must support aggregation along every hierarchy level. |
| **Aggregated positions** | Pre-aggregated position stores per product, per customer, per business unit — the materialized facts that make regulatory aggregation (BCBS 239) fast and on-demand. |

Key modeling rules: balances are **point-in-time facts** (never update history — append new snapshots); balance types are canonical (ledger, available, cleared, accrued); and **reconciliation** between transaction history and balance history must be possible (transactions + opening balance = closing balance — a rule regulators audit).

### 6.4 Agreement Lifecycle Modeling

An **agreement** (account, loan, card, deposit, policy) has a lifecycle. Modeling it properly requires:

| Element | Example |
|---|---|
| **Agreement states** | `PENDING` → `ACTIVE` → `SUSPENDED` → `CLOSED` (with product-specific variants: `DRAWDOWN`, `MATURED`, `LAPSED` for policies) |
| **Lifecycle events** | Origination, drawdown, repayment, rollover, maturity, renewal, cancellation, closure |
| **State transitions** | A governed state machine: which transitions are legal (ACTIVE → SUSPENDED yes; CLOSED → ACTIVE typically no), who can trigger them, and what data changes accompany them |
| **Audit trail** | Every transition recorded: event, timestamp, actor, system, before/after state — the substrate for regulatory audit and for reconstructing the agreement's history |

Lifecycle state drives almost everything downstream: accruals run only on ACTIVE agreements; ECL staging reacts to state changes (forbearance, default); reporting counts agreements by state. Modeling state as *derived from events* (event sourcing, Section 6.7) is increasingly common because it makes the audit trail intrinsic.

### 6.5 Customer 360 Model

The customer 360 model is the canonical representation of everything the bank knows about a customer:

| Element | Contents |
|---|---|
| **Customer entity** | `Person` (individuals) or `Organization` (companies), with the party super-type pattern (Section 6.1) |
| **Customer relationships** | Related parties and roles: account holder, signatory, beneficial owner, guarantor, authorized user — including group structures for corporate customers (parent → subsidiaries) |
| **Customer segments** | Retail, SME, corporate, institutional (and sub-segments: affluent, private banking) — segment drives product eligibility, pricing, and risk treatment |
| **KYC data** | Identity documents (passport, NRIC, company registration), risk rating (customer risk score), PEP flag, sanction status, source of funds — modeled as KYC entities with review dates and versions (MAS Notice 612 / AML) |
| **Customer consent** | Consent records (GDPR/PDPA — Section 5.3) linked to the customer and to the purposes/uses they cover |

The customer 360 model is where the CDM's "single source of truth" claim is won or lost: if party identifiers are not unified, customer analytics, group risk aggregation, and KYC all break. Identifier resolution (matching customers across systems into one canonical party) is a data-quality program in its own right, usually supported by a party master / MDM system feeding the canonical party model.

### 6.6 Worked Example: Account and Transaction Model

A concrete canonical schema for the core banking pattern — the account/transaction/balance/limit model from Sections 6.2–6.3 — in SQL DDL:

```sql
-- Canonical account model (simplified; CDM silver layer)

CREATE TABLE dim_account (
    account_id          BIGINT          NOT NULL,          -- surrogate key
    account_number      VARCHAR(34)     NOT NULL,          -- natural key (IBAN-like)
    product_id          BIGINT          NOT NULL,          -- FK to dim_product
    customer_id         BIGINT          NOT NULL,          -- FK to dim_party (account holder)
    account_status      VARCHAR(20)     NOT NULL,          -- ACTIVE | SUSPENDED | CLOSED
    currency_code       CHAR(3)         NOT NULL,          -- ISO 4217
    open_date           DATE            NOT NULL,
    close_date          DATE,
    source_system       VARCHAR(30)     NOT NULL,
    created_at          TIMESTAMP       NOT NULL,
    updated_at          TIMESTAMP       NOT NULL,
    PRIMARY KEY (account_id),
    UNIQUE (account_number, source_system)
);

CREATE TABLE fact_transaction (
    transaction_id      BIGINT          NOT NULL,          -- surrogate key
    transaction_ref     VARCHAR(50)     NOT NULL,          -- business reference
    account_id          BIGINT          NOT NULL,          -- FK to dim_account
    transaction_type    VARCHAR(30)     NOT NULL,          -- canonical code set
    amount              DECIMAL(18,2)   NOT NULL,
    currency_code       CHAR(3)         NOT NULL,
    direction           CHAR(1)         NOT NULL,          -- D (debit) | C (credit)
    value_date          DATE            NOT NULL,
    booking_timestamp   TIMESTAMP       NOT NULL,
    status              VARCHAR(20)     NOT NULL,          -- POSTED | PENDING | REVERSED | FAILED
    counterparty_ref    VARCHAR(50),                       -- other side's account/party ref
    source_system       VARCHAR(30)     NOT NULL,
    created_at          TIMESTAMP       NOT NULL,
    PRIMARY KEY (transaction_id),
    UNIQUE (transaction_ref, source_system),
    FOREIGN KEY (account_id) REFERENCES dim_account (account_id)
);

CREATE TABLE fact_account_balance (
    account_id          BIGINT          NOT NULL,
    balance_date        DATE            NOT NULL,
    balance_type        VARCHAR(20)     NOT NULL,          -- LEDGER | AVAILABLE | CLEARED
    balance_amount      DECIMAL(18,2)   NOT NULL,
    currency_code       CHAR(3)         NOT NULL,
    created_at          TIMESTAMP       NOT NULL,
    PRIMARY KEY (account_id, balance_date, balance_type),
    FOREIGN KEY (account_id) REFERENCES dim_account (account_id)
);

CREATE TABLE dim_account_limit (
    account_id          BIGINT          NOT NULL,
    limit_type          VARCHAR(20)     NOT NULL,          -- OVERDRAFT | CREDIT | WITHDRAWAL
    limit_amount        DECIMAL(18,2)   NOT NULL,
    currency_code       CHAR(3)         NOT NULL,
    utilization_amount  DECIMAL(18,2)   NOT NULL,
    effective_date      DATE            NOT NULL,
    expiry_date         DATE,
    created_at          TIMESTAMP       NOT NULL,
    PRIMARY KEY (account_id, limit_type, effective_date),
    FOREIGN KEY (account_id) REFERENCES dim_account (account_id)
);
```

**Design notes:**

- **Surrogate keys** (`account_id`, `transaction_id`) for performance and stable references; **natural keys** (`account_number`, `transaction_ref`) retained for business meaning and deduplication — `UNIQUE (transaction_ref, source_system)` is the idempotency guard that prevents double-posting when a source re-sends data.
- **Audit fields** (`created_at`, `updated_at`, `source_system`) on every table; in a data-vault implementation these become `LOAD_DTS` / `RECORD_SOURCE` on satellites.
- **Balance history is append-only** — `fact_account_balance` is keyed by `balance_date`, never updated in place.
- **Canonical code sets** for status, type, and direction — enforced via reference tables or check constraints, with mapping tables back to source-system codes.
- **Currency on every financial fact** — never assume the account currency; a transaction may be booked in a different currency than the account.

### 6.7 Event-Driven Data Models

The final pattern family moves from *state* to *events* as the primary modeling artifact — increasingly standard in modern banking platforms:

| Pattern | Description |
|---|---|
| **Event sourcing** | Transaction events (and lifecycle events, KYC changes, limit changes) are the **source of truth**; current state (account balance, agreement status) is **derived from events** by replay. Every balance is a projection — which gives perfect auditability: any historical state can be reconstructed. |
| **CQRS** | **Command and Query Responsibility Segregation** — writes go through the command model (validating business rules), reads go through optimized query models/projections. Banking platforms use CQRS to keep high-volume transaction ingestion separate from read-heavy analytics. |
| **Kafka event streams as the canonical data bus** | Events are published to Kafka topics as the enterprise's canonical event backbone — the streaming counterpart of the CDM. The topic schemas (Avro/JSON/Protobuf) carry the same canonical definitions as the stored CDM, so the "common language" spans real-time and batch. |
| **Outbox pattern** | For **reliable event publishing**: a transactional outbox table in the source database records the events to publish; a relay reads the outbox and publishes to Kafka. This guarantees the event is published exactly once (at-least-once) when the business transaction commits — no dual-write inconsistency. |
| **Change Data Capture (CDC)** | **CDC** captures source-system changes (inserts/updates/deletes on source tables) and publishes them as events — the standard way to feed the canonical layer from core banking without modifying the source. CDC events + event-sourced canonical store = source systems change freely while the enterprise model stays consistent. |

**Event-driven + canonical in practice:** the modern banking data platform ingests via CDC/events into the canonical layer (silver), stores full history (data vault or event store), and serves both real-time consumers (streaming, CQRS projections) and batch analytics (marts). The CDM's definitions govern the event schemas just as they govern the tables — one vocabulary, two representations (event and state).

---

## 7. Tools for Data Modeling

Data modeling tools span traditional ER modeling, NoSQL/multi-model design, and the code-first transformation world (dbt). This section covers the mainstream tools and where each fits in a banking/insurance data platform.

### 7.1 erwin Data Modeler

**erwin Data Modeler** (formerly ERwin) is the classic ER modeling tool, in use for decades in financial services. It supports:

- **ER modeling** — conceptual, logical, and physical models with forward and reverse engineering (reverse from existing databases, forward to DDL).
- **Compare/merge** — model-to-model and model-to-database comparison with merge/update scripts — the core workflow for managing schema change.
- **Model-driven data governance** — erwin positions models as the governance backbone: business glossary linkage, data lineage, and collaboration around the model.

**Best for:** enterprises running classic ER modeling disciplines — banks whose warehouse models (BDW-style) are managed in erwin; teams needing robust reverse engineering of legacy core-banking schemas.

### 7.2 ER/Studio

**ER/Studio** (now IDERA) is ERwin's main competitor, with a similar feature set plus distinctive strengths:

- **ER modeling** — logical/physical modeling, reverse/forward engineering, compare/merge.
- **Data dictionary** — a central dictionary of definitions shared across models.
- **Lineage** — source-to-target lineage for ETL mappings (ER/Studio's data lineage module).
- **Collaboration** — team repositories, model versioning, and review workflows.
- **Business glossary integration** — linking model entities to glossary terms, supporting the "definition-first" discipline of a CDM.

**Best for:** teams that want ER modeling with strong metadata/dictionary and lineage capabilities out of the box — a good fit for governing a canonical model across many projects.

### 7.3 SAP PowerDesigner

**SAP PowerDesigner** is the most enterprise-architecture-oriented of the traditional tools:

- **Multiple model types** — conceptual, logical, physical, and **dimensional** models in one tool.
- **Metadata management** — a repository of models and their relationships across the enterprise.
- **Impact analysis** — "what breaks if I change this column?" analysis across models, mappings, and linked artifacts — essential for a CDM that dozens of mappings depend on.
- **Enterprise architecture integration** — PowerDesigner integrates with SAP EA tooling and can link data models to business capabilities and applications (a natural pairing with BIAN-style capability maps, Section 2.1).

**Best for:** large enterprises wanting modeling plus impact analysis and EA integration — common in European banks running SAP landscapes.

### 7.4 Hackolade

**Hackolade** is the modern tool for **NoSQL and multi-model databases**:

- **NoSQL modeling** — MongoDB, Cassandra, DynamoDB, Couchbase, Elasticsearch, and more — with native structures (documents, arrays, nested objects, keyspaces) instead of forcing NoSQL into ER shapes.
- **JSON schema support** — generates and validates JSON Schemas, Avro, and Protobuf — directly relevant to **event schema design** for Kafka (Section 6.7) and API contracts.
- **Schema migration and comparison** — compares models/schemas and generates migration scripts across target platforms.
- **Multi-model in one tool** — relational + document + columnar models can coexist, reflecting the reality of a modern data platform.

**Best for:** teams modeling non-relational stores (event streams, document databases, key-value caches) alongside relational ones — increasingly relevant as banks adopt Kafka schemas and document stores.

### 7.5 SQLDBM

**SQLDBM** is a web-based, collaborative data modeling tool:

- **Online, web-based** — no desktop install; browser access for the whole team.
- **Collaboration** — shared models, comments, and review workflows in the browser.
- **Version control** — model versioning and history.
- **Auto-generate SQL DDL** — forward-engineering to SQL for major platforms (Snowflake, BigQuery, Redshift, SQL Server, Postgres, and others).
- **Cloud-warehouse focus** — SQLDBM is popular with teams on Snowflake/BigQuery because it supports their dialects and modern storage.

**Best for:** cloud-native teams that want lightweight, collaborative modeling without a heavyweight desktop tool — a common choice for lakehouse/warehouse projects.

### 7.6 dbt (Data Build Tool)

**dbt** is the code-first transformation tool that has become the standard for **modeling in the modern data platform** — not an ER tool, but the place where canonical models are actually *built and governed* in SQL:

| Capability | Role in data modeling |
|---|---|
| **SQL models as code** | Models are versioned SQL (or Python) in git — the physical CDM tables are defined, reviewed, and deployed like software. |
| **dbt tests** | Declarative data-quality tests on models (uniqueness, not-null, accepted values, referential integrity, custom assertions) — the executable version of CDM quality rules (Section 5.1). |
| **dbt docs** | Auto-generated model documentation with lineage graphs — the modern replacement for static data dictionaries. |
| **dbt lineage (DAG)** | The dependency graph of models is computed automatically — source → staging → canonical → marts is a real, queryable DAG — directly supporting BCBS 239 lineage requirements. |
| **dbt data contracts** | `dbt contracts` (model contracts: defined columns, types, constraints) enforce the producer-consumer agreement at build time — schema-level governance for the canonical layer. |
| **dbt Semantic Layer** | **dbt semantic models** define metrics (revenue, balances, counts) once, in the model, and serve them consistently to BI tools — the metric layer on top of the CDM. |
| **Packages** | dbt packages exist for **data vault 2.0** (e.g., `dbtvault`, `automate_dv`) — building hub/link/satellite structures with the discipline the methodology requires. |

**dbt best practices for banking data models:**

- Structure the project as **staging → intermediate → canonical (marts)** — mirroring bronze/silver/gold; keep the canonical layer free of business logic, with logic pushed to marts.
- Model **one layer per source system** in staging (raw parity), then conform to canonical in intermediate.
- Write **tests for every quality rule** a regulator would ask about: uniqueness of transaction references, not-null on exposure amounts, accepted values on status/type codes, relationships between account and party.
- Use **model contracts** on canonical models so downstream teams cannot silently break the schema.
- Define **metrics once** in semantic models so "customer count" or "total ECL" cannot be computed differently in different tools.
- Version-control everything; review models in PRs like code — the model is the product.

### 7.7 Tool Comparison

| Tool | Model Types | Key Features | Platform | Licensing | Best For |
|---|---|---|---|---|---|
| **erwin Data Modeler** | ER (conceptual/logical/physical) | Reverse/forward engineering, compare/merge, governance, glossary | Desktop (+ web) | Commercial | Classic ER modeling, legacy schema reverse engineering |
| **ER/Studio** | ER, lineage | Data dictionary, lineage, collaboration, glossary integration | Desktop (+ repository) | Commercial | ER modeling with metadata & lineage governance |
| **SAP PowerDesigner** | Conceptual, logical, physical, dimensional | Metadata management, impact analysis, EA integration | Desktop + repository | Commercial | Large enterprises, EA-linked modeling, SAP shops |
| **Hackolade** | NoSQL, multi-model, JSON Schema | NoSQL native modeling, JSON/Avro/Protobuf schemas, migration/compare | Desktop (+ web) | Commercial | Event schemas, document/columnar stores, multi-model |
| **SQLDBM** | Relational (cloud dialects) | Web-based, collaboration, versioning, auto-DDL | Cloud (SaaS) | Commercial (SaaS) | Cloud warehouse teams, collaborative modeling |
| **dbt** | Code-first (SQL models, vault, semantic) | Tests, docs, lineage DAG, contracts, semantic layer | Open-source core + cloud | OSS / SaaS | Building & governing canonical models in the data platform |

**Practical guidance:** use an ER tool (erwin/ER/Studio/PowerDesigner) for the **design-time** logical/physical models and governance documentation; use Hackolade for **event/NoSQL schemas**; use SQLDBM for lightweight collaborative cloud modeling; and use **dbt as the runtime home of the canonical layer** — where the physical CDM is built, tested, and documented in code. Design-time and runtime models must be kept in sync (compare/merge tools help), otherwise the documented model drifts from the implemented one — a common and regulator-visible failure.

---

## 8. Data Modeling Best Practices for Banking and Insurance

Distilled from the preceding sections, the practices that separate successful banking/insurance data models from failing ones:

1. **Start with the logical model before the physical.** Design business-first: entities, definitions, and relationships agreed with the business before any DDL. Align every logical entity with the **business glossary**; get **domain expert input** (product, risk, finance, compliance) so the model reflects the business, not the source systems.

2. **Use standard naming conventions.** Consistent prefixes (e.g., `dim_`/`fact_`, or data-vault `h_`/`l_`/`s_`), consistent suffixes, and a controlled abbreviation list. Code sets (statuses, types) use canonical values with mapping tables back to source codes.

3. **Model for the query patterns.** 3NF for OLTP, dimensional for BI/analytics, data vault for the enterprise warehouse core (Section 4.2). Do not force one style onto every workload.

4. **Maintain data lineage.** Document source-to-target mappings, transformation logic, and a data dictionary — as governed artifacts, not afterthoughts. Machine-readable lineage (dbt DAG, lineage tools) directly serves BCBS 239.

5. **Version-control your models.** Track changes, review changes in PRs, approve changes with an owner sign-off. The model is a product; treat it like one.

6. **Align with regulatory requirements up front.** BCBS 239 lineage and aggregation, MAS guidelines, GDPR consent and erasure, IFRS 17/IFRS 9 structures — bake these into the model rather than retrofitting (Section 5).

7. **Document data quality rules.** Business rules, validation rules, and quality thresholds attached to attributes — and *executed* as tests (dbt tests, data-quality tooling) with dashboards for monitoring.

8. **Include audit fields.** `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `source_system` on every table; in data vault, `LOAD_DTS`/`RECORD_SOURCE` are the equivalents. Regulators and auditors will look for them.

9. **Use surrogate keys for performance, natural keys for business meaning.** Surrogates for stable joins and slow-changing references; natural keys retained for deduplication, reconciliation, and business lookup — with uniqueness enforced on natural keys.

10. **Model temporal data explicitly.** Effective dating (when a fact is true in the business), transaction dating (when it was recorded), and **bi-temporal modeling** (both) where regulators require reconstruction of history as it was known at a point in time.

11. **Plan for data retention.** Archival, deletion, and GDPR erasure processes designed into the model — retention classes per data type, not a single "keep everything" policy.

12. **Involve business stakeholders.** Business glossary ownership, data owners, and data stewards per domain — a CDM without owners decays.

13. **Balance normalization vs. denormalization.** 3NF for flexibility and single-sourcing; dimensional/denormalized for analytics performance; document the trade-off per layer.

14. **Use data contracts.** Schema agreements between producer and consumer (model contracts, schema registries for events, API contracts) so changes are negotiated, not breaking.

15. **Modernize iteratively.** Start with the core entities (party, product, agreement, transaction), expand gradually, and refactor as needed — a 2,000-entity big-bang implementation fails far more often than an incremental one that starts at the center of gravity.

---

## 9. Conclusion and Selection Guide

### 9.1 How to Choose a Data Model

| If your situation is… | Choose… | Because… |
|---|---|---|
| IBM data platform, comprehensive banking warehouse | **IBM BDW** | Mature, broad, IBM-integrated reference model (~2,000 entities) |
| Teradata platform; banking + insurance + securities group | **Teradata FSLDM** | Multi-domain 3NF logical model, proven in production |
| Service-oriented architecture / API standardization program | **BIAN** | Industry-standard banking capability and business-object semantics |
| Oracle / OFSAA risk & compliance analytics | **Oracle OFSDFM** | Risk, finance, and compliance structures aligned to OFSAA |
| Insurance data exchange / agency-carrier integration | **ACORD** | The insurance industry's canonical exchange standards |
| Enterprise warehouse needing history + auditability (BCBS 239) | **Data Vault 2.0** | Full-history, source-agnostic, parallel-loading foundation |
| BI and analytics consumption | **Dimensional marts** | Pre-joined, aggregatable structures for reporting tools |
| Transactional OLTP systems | **3NF** | Normalized integrity for operational processing |

### 9.2 The Modern Recommendation

For a bank or insurer building a data platform today, the winning combination is **not one model but a layered composition**:

1. **Data Vault 2.0 as the enterprise warehouse foundation** — full history, auditability (BCBS 239), parallel loading, source-agnosticism. This is the physical backbone of the silver/canonical layer.
2. **Dimensional marts for analytics** — star schemas built from the vault for BI, reporting, and regulatory marts (COREP/FINREP, MAS returns, ECL results).
3. **An industry model (BIAN / FSLDM / BDW) as the reference for domain structures** — use its subject areas and business objects to define *which* entities belong in the vault and what they mean, without importing the full 2,000-entity model wholesale.
4. **Data contracts for interface governance** — model contracts (dbt/DBT-style), schema registries for Kafka events, and API contracts so the canonical vocabulary is enforced at every boundary.
5. **Event-driven ingestion (CDC + Kafka + outbox)** — feed the vault and the streaming consumers from the same canonical event definitions.

Plus a **CDM governance layer** — definitions, owners, quality rules, lineage — running through all of it (Section 1). The industry reference model gives you the *vocabulary*; the vault gives you the *history*; the marts give you the *consumption*; contracts give you the *discipline*.

### 9.3 Banking-Specific Guidance

- **BIAN** for service-oriented architecture alignment — if the bank is standardizing APIs and capabilities (core banking transformation, open banking), BIAN gives the business-object semantics that both APIs and data should share.
- **IBM BDW or Teradata FSLDM** for a comprehensive warehouse — if the platform is IBM or Teradata and the bank wants a proven reference model to subset.
- **Data vault for BCBS 239 lineage and history** — the audit-trail and full-history requirements of BCBS 239 (and MAS 637) are the strongest argument for vault-based foundations in risk data aggregation.
- **IFRS 9/CECL and regulatory reporting** drive canonical exposure/collateral/counterparty/position structures shared by risk and finance (Section 5.6).

### 9.4 Insurance-Specific Guidance

- **ACORD** for exchange standards — carrier-agency communication, policy/claims/billing integration, and any insurance CDM should adopt ACORD's policy/claim/party/coverage vocabulary.
- **IBM IIW** for an insurance warehouse reference model — the multi-line (life, annuity, health, P&C) warehouse blueprint.
- **IFRS 17** drives the contract-group-level cash-flow, discount-curve, and CSM structures that dominate modern insurance data modeling; **Solvency II** drives risk-data aggregation and valuation data (Section 5.4–5.5).

**Final thought.** The data model is the bank's memory and its evidence. In a regulated industry, a well-governed canonical model is not an IT artifact — it is the mechanism by which the institution knows itself consistently, reports truthfully, and proves both to regulators. Choose the vocabulary (industry model), the spine (vault/3NF), the consumption shape (marts), and the governance (contracts, lineage, quality), and the rest is execution.

---

## 10. References

- BIAN — Banking Industry Architecture Network: [bian.org](https://www.bian.org)
- ACORD — Association for Cooperative Operations Research and Development: [acord.org](https://www.acord.org)
- IBM Banking Data Warehouse & IBM Banking Data Model for Analytics — IBM documentation
- IBM Insurance Information Warehouse — IBM documentation
- Teradata Financial Services Logical Data Model (FSLDM) — Teradata documentation
- Oracle Financial Services Analytical Applications (OFSAA) / OFSDFM — Oracle documentation
- SAS Banking Data Model, SAS Insurance Data Model — SAS documentation
- Guidewire InfoCenter — Guidewire documentation
- BCBS 239 — *Principles for effective risk data aggregation and risk reporting*, Basel Committee on Banking Supervision, 2013
- MAS Notice 637 (Risk Management), MAS Notice 612 (AML/CFT), MAS Technology Risk Management — Monetary Authority of Singapore
- GDPR (EU 2016/679); PDPA — Personal Data Protection Act (Singapore)
- Solvency II — Directive 2009/138/EC (EU insurance regulation)
- IFRS 17 — Insurance Contracts; IFRS 9 — Financial Instruments (IASB)
- CECL — Current Expected Credit Losses, FASB ASU 2016-13
- Data Vault 2.0 — Dan Linstedt & Michael Olschimke, *Building a Scalable Data Warehouse with Data Vault 2.0* (2015)
- Zhamak Dehghani, *Data Mesh: Delivering Data-Driven Value at Scale* (2022)
- dbt Labs — dbt documentation and dbtvault/automate_dv packages

---

*This guide is part of the [jackliusr/research](https://github.com/jackliusr/research) series on data architecture for financial services.*
