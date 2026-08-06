# Data Governance: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Data Engineering / Data Governance — Frameworks, Operating Models, Data Quality, Metadata & Lineage, Regulatory Drivers (BCBS 239, MAS), Banking  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [Foundations: What Data Governance Is](#1-foundations-what-data-governance-is)
2. [Why Governance: Drivers and Value](#2-why-governance-drivers-and-value)
3. [Frameworks and Standards](#3-frameworks-and-standards)
4. [Governance Organization and Operating Model](#4-governance-organization-and-operating-model)
5. [Policies and Standards](#5-policies-and-standards)
6. [Data Quality Management](#6-data-quality-management)
7. [Metadata Management](#7-metadata-management)
8. [Reference and Master Data Management](#8-reference-and-master-data-management)
9. [Governance in the Data Platform](#9-governance-in-the-data-platform)
10. [Banking and Regulatory Context](#10-banking-and-regulatory-context)
11. [Implementation Roadmap](#11-implementation-roadmap)
12. [Worked Example: BCBS 239 Risk Data Program](#12-worked-example-bcbs-239-risk-data-program)
13. [The Future: 2026 and Beyond](#13-the-future-2026-and-beyond)
14. [Glossary](#14-glossary)
15. [References](#15-references)

---

## 1. Foundations: What Data Governance Is

Data governance is the discipline of **defining policies, standards, and responsibilities for managing data as an organizational asset** — answering, for every piece of data, *who can do what with it, under what rules, and who is accountable*. It is the answer to a deceptively simple question: when two departments disagree about what "customer revenue" means, or when a regulator asks who approved a data quality threshold, there must be a defined, accountable answer. Governance is the machinery that produces that answer and makes it stick.

### 1.1 Governance vs Management vs Engineering

The single most common confusion in the field is treating "governance" and "management" as synonyms. They are different layers of the same system:

| Layer | Question answered | Typical content | Who owns it |
|-------|-------------------|-----------------|-------------|
| **Governance** | The WHAT and the WHO | Decision rights, accountability, policies, standards, principles | CDO, data council, stewards |
| **Management** | The HOW | Processes, practices, procedures, quality programs, lifecycle operations | Data management teams, DGO |
| **Engineering** | The WITH WHAT | Technology: pipelines, catalogs, tools, platforms, infrastructure | Data engineers, platform teams |

Governance is the **decision rights and accountability framework** — it decides that a customer master exists, that the Risk division owns it, and that any change to the definition of "credit exposure" requires council approval. Management is the **operational execution** — running the stewardship program, profiling data, fixing quality issues, operating the catalog day to day. Engineering builds and runs the technology that makes both possible. A governance framework with no management capability is a set of empty words; management with no governance produces well-run chaos; engineering with no governance produces pipelines that nobody can explain to an auditor.

### 1.2 The Canonical Definition: DAMA-DMBOK

The industry-standard definition comes from the Data Management Body of Knowledge (DAMA-DMBOK, 2nd edition, 2017):

> **Data governance is the exercise of authority and control (planning, monitoring, and enforcement) over the management of data assets.**

Note the deliberate scope. Governance is *not* the same as doing data management — it is the **authority and control** over whoever does the management. The definition bundles three verbs: *planning* (deciding rules and decision rights in advance), *monitoring* (measuring whether the rules hold), and *enforcement* (acting when they do not). A governance program that plans but never enforces is documentation, not governance.

### 1.3 The DAMA Wheel and the 11 Knowledge Areas

DAMA-DMBOK organizes the discipline into **11 knowledge areas (KAs)**, usually drawn as a wheel with Data Governance at the center:

| # | Knowledge area | Scope |
|---|----------------|-------|
| 1 | **Data Governance** | Planning, policy, standards, stewardship, decision rights (the hub) |
| 2 | **Data Architecture** | Enterprise data architecture, blueprints, integration design |
| 3 | **Data Modeling & Design** | Conceptual/logical/physical models (see [data_model_resource_book_guide.md](data_model_resource_book_guide.md)) |
| 4 | **Data Storage & Operations** | Databases, warehouses, lakes, operations, backup/recovery |
| 5 | **Data Security** | Access, privacy, encryption, masking |
| 6 | **Data Integration & Interoperability** | Pipelines, ETL/ELT, streaming (see [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md)) |
| 7 | **Document & Content Management** | Unstructured data, records management |
| 8 | **Reference & Master Data** | Golden records, reference codes (see §8) |
| 9 | **Data Warehousing & BI** | Dimensional models, reporting, analytics |
| 10 | **Metadata** | Catalogs, glossaries, lineage (see §7) |
| 11 | **Data Quality** | Dimensions, measurement, issue management (see §6) |

The wheel is a *system*, not a menu: data quality depends on metadata definitions, which depend on architecture decisions, which depend on modeling choices — and every knowledge area's activities must be *governed*. That is why governance sits at the hub: it is the central function that all other areas feed into and receive authority from. In DAMA's context diagram, data governance is the hub that coordinates inputs (policies, business requirements, regulatory obligations) and outputs (standards, decision rights, accountabilities) to the other ten areas.

### 1.4 Governance Principles

Whatever framework an organization adopts, the same core principles recur:

- **Accountability** — every data asset has a named owner who can answer for it. No orphan data.
- **Transparency** — rules, decision rights, and data issues are visible and auditable; nothing operates as a hidden exception.
- **Integrity** — data is trustworthy, and the governance process itself is honest about gaps and failures.
- **Stewardship** — data is cared for on behalf of the organization, not owned by individuals or silos.
- **Compliance** — regulatory and contractual obligations are non-negotiable inputs to every decision.
- **Data as an asset** — data is valued, invested in, and managed like capital, with a business case and returns (see §2.3).

### 1.5 The Governance Escalation Path

The mechanism that makes governance *work* is the escalation path — a bounded, pre-agreed route for disputes and issues that operational teams cannot resolve:

```
Data issue / definition dispute
        │
        ▼
Data steward (triage, 5 days) ──► resolved: implement fix
        │ unresolved
        ▼
Domain data owner (decision, 2 weeks) ──► resolved: set standard
        │ unresolved
        ▼
Governance council (arbitration, next meeting) ──► decision is binding
```

The path matters more than any policy: it guarantees that no dispute can hide. A governance program with an escalation path but thin policies will outperform one with perfect policies and no path — because the path produces *decisions*, and decisions produce standards. Every domain and every issue category (definitions, quality tolerances, access disputes) should have its escalation path written down in the operating model, with SLAs at each step.

### 1.6 Governance vs Privacy vs Security

Three overlapping disciplines get conflated; their boundaries matter for organization design:

| Discipline | Question it answers | Typical home |
|-----------|---------------------|--------------|
| **Data governance** | Who may use data, for what purpose, under what rules? | CDO / DGO |
| **Data protection / privacy** | How is personal data handled lawfully (consent, purpose, retention, deletion)? | Privacy office / DPO |
| **Information security** | How is data protected from unauthorized access, loss, and breach? | CISO / security |

Governance *decides* the rules (classification, access policy, retention); security *enforces* them (controls, encryption, monitoring); privacy *constrains* them (lawful basis, minimization). A common failure is treating privacy as a subset of security (it is not — privacy is about legitimate use, security about unauthorized access) or treating governance as a subset of IT (it is not — it is a business discipline). Banks typically run all three with a defined interface: governance owns the data policies, privacy reviews them for PII obligations, security implements and audits the controls.

## 2. Why Governance: Drivers and Value

Governance programs are usually justified by one of two triggers: a regulatory deadline that forces the issue, or a business failure (a bad report, a failed migration, a customer-360 project that collapses) that makes trust in data the bottleneck. Both are real; the strongest programs are built on both.

### 2.1 Regulatory Drivers

Regulation is the original and still the most powerful driver — regulators can quantify the cost of bad data governance more precisely than any internal business case:

- **BCBS 239 (2013)** — the Basel Committee's *Principles for Effective Risk Data Aggregation and Risk Reporting*; 14 principles requiring banks to govern risk data with defined ownership, architecture, and demonstrable accuracy/completeness/timeliness. The single most influential data governance driver in banking (§10.1; full coverage in [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) §2.2 and §9.2).
- **MAS (Singapore)** — the Technology Risk Management (TRM) framework and its successor FSMA-era Notices (FSM-N05 Technology Risk Management, FSM-N06 Cyber Hygiene) plus data-management expectations in supervisory guidance; data governance is an explicit supervisory expectation for banks in Singapore (§10.2).
- **GDPR (EU) / PDPA (Singapore)** — data protection regimes that force *privacy* into the governance framework: lawful basis, retention limits, deletion rights, breach notification. Privacy is data governance with legal teeth.
- **DORA (EU, applied 17 January 2025)** — Digital Operational Resilience Act; requires financial entities to govern ICT risk, including data integrity and availability for critical functions, with a full ICT risk management framework.
- **IFRS 17 (insurance, effective 2023)** — actuarial reporting built on granular, well-governed contract-level data; forced insurance firms to build the same data-quality and lineage machinery banks built for BCBS 239.
- **SOX §404 (US, 2002)** — requires management to attest to internal controls over financial reporting, which auditors translate into *IT general controls* over data: access, change management, and evidence of control effectiveness.

The pattern is universal: every major post-2008 financial regulation is, at its core, a *data governance* regulation wearing different clothes.

### 2.2 Business Drivers

Beyond regulation, governance earns its keep operationally:

- **Decision quality** — "garbage in, garbage out." Data-driven decisions, dashboards, and KPIs are only as trustworthy as the underlying data; governance is what makes "trusted data" a design property rather than a hope.
- **Operational efficiency** — a governed single source of truth (SSOT) eliminates duplicated reconciliation, conflicting definitions, and rework. Unresolved definitional disputes silently cost more than any governance budget.
- **Risk management** — data risk is a first-class risk: bad data causes bad decisions, regulatory breaches, and model failures. Governance is the control that mitigates data risk.
- **Monetization** — governed data can be packaged as *data products* (internally or externally sold, §9.3); ungoverned data cannot be sold, shared, or safely exposed.
- **AI/ML enablement** — models are trained on data; model quality is downstream of data quality. "AI-ready data" (provenance, licensing, bias-aware, well-documented) is a governance output, not an engineering one (see §9.5 and the `ai_llm/` guide series).
- **Cost** — governance drives the data lifecycle: knowing what exists and what it is worth makes retention, tiering, and deletion decisions possible. Storage and compute costs in modern lakehouses are otherwise unbounded (§9.1).

### 2.3 The Governance Value Framework

The business case for governance is usually built on three value levers:

| Lever | Mechanism | Example metric |
|-------|-----------|----------------|
| **Risk reduction** | Fewer regulatory findings, fines, and audit issues; faster remediation | BCBS 239 principle ratings, audit issue closure rate |
| **Cost savings** | Less rework, lower storage spend, decommissioning, fewer reconciliation teams | % storage retired, FTE reduction in reconciliation |
| **Revenue enablement** | Faster product launch, data products, better cross-sell via customer 360 | Time-to-market for data products, uplift from 360 analytics |

The governance business case should quantify all three, because "compliance only" cases get defunded once the deadline passes, while "revenue only" cases get deprioritized by risk functions. The strongest framing: *governance is the insurance policy that makes every other data investment (warehouse, lakehouse, MDM, AI) safe to make* — its absence is what turns those investments into write-downs.

### 2.4 Funding the Program

Governance funding follows a recognizable arc: **regulatory remediation funding** (a supervisory finding frees budget fast) → **transformation funding** (platform modernization carries governance as a workstream) → **BAU funding** (governance as an operating cost, once the KPIs prove value). The funding model matters because the *value* of governance shows up in other budgets: the DGO's costs are visible, while the rework it eliminates is spread invisibly across the bank. That asymmetry is why governance programs get cut in downturns — unless the business case (risk reduction + cost savings + revenue enablement, §2.3) is refreshed annually with real numbers. Pragmatic rule: attach governance funding to the programs that need it (regulatory reporting, customer 360, AI enablement) rather than standing it up as an isolated cost center; isolated governance budgets are the first to die.

### 2.5 The Cost of No Governance

The business case is sharpened by naming what absence costs. The recurring failure catalog: **regulatory findings and capital add-ons** (BCBS 239 gaps are priced by supervisors); **the reconciliation tax** (finance and risk teams permanently staffing reconciliations that exist only because systems disagree); **failed migrations and mergers** (data mapping discovered at cutover, not governed upfront); **customer-360 projects that stall** (identity resolution is a governance problem, not a tool problem); **shadow IT data** (the business builds its own spreadsheets and databases because governed data is inaccessible — and the shadow data then feeds decisions); and **AI project failures** (models that cannot be deployed because training data provenance or bias cannot be demonstrated). Every one of these has a price tag; a governance program's budget is small change next to any one of them. This is the argument that converts CFOs: not "governance is good" but "these six known losses are governed-away."

## 3. Frameworks and Standards

A framework gives governance a shared vocabulary, a scope definition, and a maturity yardstick. No framework is mandatory; mixing two or three deliberately (a maturity model for assessment, a body of knowledge for content, a regulatory standard for banking specifics) is the norm.

### 3.1 The Major Frameworks

| Framework | Origin | Focus | Maturity model | Best for |
|-----------|--------|-------|----------------|----------|
| **DAMA-DMBOK** | DAMA International (2nd ed. 2017) | Body of knowledge: 11 knowledge areas, processes, activities | None built-in (used with other models) | Content: what to do and why; certification (CDMP) |
| **DGI Framework** | Data Governance Institute | 10 universal components of a governance *program* (mission, scope, decision rights, accountabilities, controls, etc.) | None built-in | Designing the governance *program* itself (org, process) |
| **CMMI DMM** | CMMI Institute (ISACA) | Data management capability maturity | 5 levels: Initial → Managed → Defined → Measured → Optimized | Maturity assessment and improvement roadmaps |
| **DCAM** | EDM Council | Data management *capability assessment* for financial services | 8 components, each scored 1–5 | Banks: objective, regulator-recognized assessment (the de facto banking standard) |
| **COBIT 2019** | ISACA | IT governance and management, enterprise-aligned | 6 capability levels (0–5) | IT governance integration; audit alignment |
| **TOGAF** | The Open Group | Enterprise architecture (business, data, application, technology domains) | Architecture Maturity Model | Where governance must integrate with architecture governance |
| **ISO 8000** | ISO | Data quality (part 8x series: quality of data, exchange, master data) | Conformance-based, not a maturity ladder | Data quality certification and exchange standards |
| **ISO/IEC 38505-1:2017** | ISO/IEC | Governance of data (principles for governing data, aligned with ISO 38500 IT governance) | None built-in | ISO-aligned governance of data as a corporate asset |
| **NIST** | US NIST | No standalone DG framework — NIST AI RMF (AI governance), NIST CSF (cybersecurity), NBDIF (big data interoperability) | NIST CSF tiers | US public sector; AI risk governance (§13.1) |

### 3.2 DCAM — the Banking Standard

The **EDM Council's Data Management Capability Assessment Model (DCAM)** deserves special attention: it is the framework most recognized by banking supervisors (and by firms responding to BCBS 239 reviews) because it was built *by* financial institutions and maps directly to supervisory expectations. DCAM v2 (2020) organizes data management into **8 components**:

1. **Data Management Strategy** — mission, goals, alignment to business strategy.
2. **Business Case and Funding Model** — value articulation and sustainable funding.
3. **Data Management Program** — operating model, roles, communication.
4. **Data Governance** — decision rights, accountability, policies.
5. **Data Architecture** — target architecture, data flows, integration.
6. **Technology Architecture** — the platform and tooling landscape.
7. **Data Quality Management** — dimensions, measurement, remediation.
8. **Data Operations and Control Environment** — BAU operations, controls, monitoring.

Each component is assessed against capability levels, producing a maturity scorecard that a bank can show to supervisors and to its own board. DCAM v3 (announced 2025/2026 by the EDM Council) extends the model into the AI/data-product era.

### 3.3 DAMA-DMBOK Deep Dive

DMBOK2 is the content backbone of most programs: it defines the *activities, inputs, outputs, roles, and techniques* for each knowledge area. Its governance chapter specifies the classic **governance activities**: define data policies and standards; establish decision rights and accountability; steward the data domains; monitor and enforce policy compliance; and manage escalations (the "governance escalation path" that turns a stuck definitional dispute into a council decision in a bounded time). The DAMA wheel's context diagram positions governance as the hub: policies and requirements flow *out* to the other ten KAs, and status, issues, and metadata flow *back* for oversight. DMBOK2 also popularized the roles vocabulary used in §4 (owner, steward, custodian), the quality dimensions in §6, and the metadata types in §7.

### 3.4 Choosing and Combining

A pragmatic combination seen across banks: **DCAM or CMMI DMM for the maturity assessment** (objective, regulator-friendly), **DAMA-DMBOK for the content** (what each function actually does), **DGI for the program design** (how the council, stewards, and processes fit together), and **COBIT for the IT/audit interface**. Frameworks are tools, not religions — the deliverable that matters is a functioning operating model, not framework purity.

### 3.5 CMMI DMM Maturity Levels

The CMMI Data Management Maturity model gives governance a simple, communicable maturity ladder (also the shape DCAM scoring follows):

| Level | Name | What it means in practice |
|-------|------|---------------------------|
| 1 | **Initial** | Ad hoc: data managed by heroes; no defined roles or standards; every report is a bespoke effort |
| 2 | **Managed** | Project-level: some domains have owners and processes; repeatable within projects, inconsistent across them |
| 3 | **Defined** | Organization-level: firm-wide standards, glossary, and roles; processes documented and trained |
| 4 | **Measured** | Quantitative: quality and coverage measured continuously; KPIs drive decisions; SLAs enforced |
| 5 | **Optimized** | Continuous improvement: data-driven optimization of the governance process itself; automation (policy-as-code) is normal |

Most banks assessed are at level 2–3; the jump from 3 to 4 (measurement) is where regulatory evidence becomes credible, and the jump from 4 to 5 is where governance becomes cheap (automated). Use the ladder to set *honest* targets: a level-1 organization promising level-5 in a year is a red flag, not a goal.

### 3.6 Choosing: A Decision Aid

| Your situation | Framework to lead with | Why |
|----------------|------------------------|-----|
| Supervisory finding on data (BCBS 239, MAS) | **DCAM** | The assessment language supervisors recognize; scores map to findings |
| No roles, no policies, no vocabulary | **DAMA-DMBOK** | The content backbone: what to build and in what order |
| Program exists but is drifting (no decisions, no teeth) | **DGI** | The 10 components fix program structure and decision rights |
| IT/audit wants control mapping (SOX, COBIT alignment) | **COBIT** | Enterprise IT governance and audit interface |
| Data quality certification or exchange with partners | **ISO 8000** | Conformance-based quality standards for exchange |
| AI governance program forming | **NIST AI RMF + EU AI Act mapping** | The AI-risk vocabulary and the legal requirements (§13.1) |
| Architecture governance needs a home | **TOGAF** | Data governance as a domain within architecture governance |

The table is a starting point, not a verdict: most enterprises end up with DAMA content + DCAM/CMMI assessment + DGI program design. The discipline that matters is *picking deliberately and saying which questions each framework answers* — that is what prevents framework soup.

## 4. Governance Organization and Operating Model

Governance lives or dies on **who decides, who owns, who does, and who watches**. The roles below are the standard vocabulary (DAMA-DMBOK and bank practice converge on it); the operating model determines how they are wired together.

### 4.1 Roles

| Role | Level | Accountability | Typical duties |
|------|-------|----------------|----------------|
| **Chief Data Officer (CDO)** | Executive (C-suite) | Enterprise data strategy, governance mandate, value realization | Owns the data strategy, chairs/attends council, owns the DGO, reports data value and risk to the board. The CDO role has evolved from "compliance fixer" (post-BCBS 239) to a revenue/value role (data products, AI enablement). |
| **Data Governance Council / Committee** | Executive cross-functional | Direction and escalation | Approves policies, resolves domain disputes, prioritizes initiatives; typically meets quarterly, chaired by CDO or COO with CRO/CFO/CISO representation. |
| **Data Owner** | Senior business leader | **Accountability** for a data domain's quality and use | Appoints the steward, approves standards for the domain, answers to the council and regulators for domain data. Ownership is a line-of-business role, not a technology role. |
| **Business Data Steward** | Business function | Day-to-day domain quality and definitions | Maintains the glossary, validates quality rules, triages issues, represents the domain in forums. |
| **Technical Data Steward / Custodian** | IT / platform | Implementation and custody | Builds and runs the technical controls: pipelines, catalog, lineage, masking, retention execution. Custodians *implement*; they do not own. |
| **Data Architect** | Architecture | Design and standards | Designs the data architecture the governance rules apply to (see [data_model_resource_book_guide.md](data_model_resource_book_guide.md) and [databricks_guide.md](databricks_guide.md)). |
| **Data Governance Office (DGO)** | Central team | Program management | Runs the operating model: policy administration, training, KPI reporting, tool administration, issue escalation logistics. The DGO is governance's "engine room." |

The single most common organizational failure: **a governance structure with stewards but no owners** — people who operate the process but nobody accountable for outcomes. Owners must be senior enough to be fired for their domain's data quality; that is what makes the structure real.

### 4.2 Operating Models

| Model | Structure | Pros | Cons | Best for |
|-------|-----------|------|------|----------|
| **Centralized** | One central DGO governs everything; stewards report into it | Consistent standards, fast policy creation, strong control | Distant from business realities; becomes a bottleneck; business treats it as "their" function | Small/mid-size firms; heavy regulatory pressure; greenfield |
| **Decentralized (federated)** | Each domain governs itself with light central coordination | Business-aligned, fast local decisions, high engagement | Inconsistent standards, weak cross-domain integration, governance drift | Highly diversified groups with mature domain teams |
| **Hybrid (hub-and-spoke)** | Central DGO/center of excellence sets frameworks; domain stewards operate within them | Consistency + business alignment; scalable; the most common bank model | Needs strong central authority and clear escalation to avoid "two masters" | Large enterprises and banks (the default choice) |

The hybrid model's working pattern: the center owns *the framework* (policy templates, tooling, training, KPIs, escalation), the spokes own *the execution* (domain glossaries, quality rules, issue resolution), and the council arbitrates. This is the model regulators implicitly expect at G-SIB scale: firm-wide standards with accountable domain owners.

### 4.3 RACI: Making Decision Rights Explicit

RACI (Responsible / Accountable / Consulted / Informed) is the mechanism that turns "decision rights" from a phrase into a testable artifact. Rules of thumb: exactly one **A** per row (an owner who answers for it), R does the work, C must be consulted before decisions, I is notified after. Two canonical examples:

| Activity: Data quality issue resolution | Business Owner | Data Steward | DGO | IT/Custodian | CRO |
|------------------------------------------|----------------|--------------|-----|--------------|-----|
| Log and triage the issue | I | **R** | A | I | I |
| Fix data at source | A | C | C | **R** | I |
| Verify fix and close | I | **R** | A | C | I |
| Escalate unresolved issue | A | R | R | I | **C** |

| Activity: Data access approval | Owner | Steward | DGO | Security | Requesting team |
|--------------------------------|-------|---------|-----|----------|-----------------|
| Request access | I | C | I | C | **R** |
| Approve business justification | **A** | R | I | C | I |
| Grant technical access | I | I | I | **R** | A |
| Periodic access review | **A** | R | I | R | I |

### 4.4 Forums and Cadence

Governance is a meeting discipline before it is a tooling discipline. The standard cadence: **council quarterly** (strategy, escalation, policy approvals), **DGO monthly** (program KPIs, issue backlog, policy compliance), **domain forums monthly/bi-weekly** (steward-level definition and quality discussions), and **ad-hoc working groups** for specific artifacts (a glossary build, a lineage remediation, an AI data governance review). Every forum needs a standing agenda, published minutes, and an action tracker — governance forums without actions are the first thing auditors notice.

### 4.5 Data Domains and Data Mesh

**Data domains** are the natural units of ownership: **Customer, Product, Finance, Risk, Counterparty, Operations, HR, Reference Data**. Each domain gets an owner, a steward, a glossary, and quality KPIs. Domain decomposition is also the organizing principle of the **data mesh** (Zhamak Dehghani, 2019), which pushes governance further toward the edges with four principles:

1. **Domain ownership** — domains own their data end to end (not just "own the definitions").
2. **Data as a product** — each domain publishes governed, documented data products (§9.3).
3. **Self-serve data platform** — a platform that makes publishing/consuming safe and easy.
4. **Federated computational governance** — governance rules encoded *in the platform* (policy-as-code, §13.4), automated rather than manual.

For banks, full mesh is rare: regulation demands firm-wide standards (BCBS 239 is inherently non-federated), so banks typically run a *domain-oriented hybrid* — mesh-style domain products with firm-wide governance standards and a strong central authority. The mesh vocabulary (data products, contracts, federated governance) has nonetheless become the industry's forward language (§9).

### 4.6 The Governance Charter

Every program needs a **charter** — the one-page constitution that answers: purpose and scope (what data, what geographies, what exclusions), roles and decision rights (who owns, who stewards, who decides what), forum structure and cadence, the escalation path, the KPI framework, and the change process for the charter itself. The charter is signed by the CDO and the council sponsors (in banking, typically the CRO and CFO — because risk data and finance data are the two domains where governance first pays). It is deliberately short and deliberately boring: its job is to end arguments about *who decides*, not to impress. When a new program or project touches governed data, the first question is "does the charter cover this?" — and if it does not, the answer is a charter change, not an exception.

### 4.7 Governance and the Three Lines of Defense

In banking, the governance organization maps onto the **three lines of defense** (a structure supervisors expect to see articulated):

| Line | Who | Governance role |
|------|-----|-----------------|
| **1st line** | Business divisions (owners, stewards) | Own the data, run the quality and access processes, answer for domain data |
| **2nd line** | Risk, compliance, privacy, DGO | Set frameworks, monitor adherence, challenge the 1st line, report to the council |
| **3rd line** | Internal audit | Independently assess whether governance is designed and operating effectively |

The mapping prevents the classic ambiguity: the DGO (2nd line) sets the framework and monitors; the divisions (1st line) execute; audit (3rd line) verifies. When a governance program is placed entirely in the 2nd line with no 1st-line owners, it becomes a policing function with no accountability for outcomes — the failure mode described in §4.1. When the 2nd line is absent, the 1st line drifts. The council sits above the model, arbitrating between lines.

## 5. Policies and Standards

Policies are governance's output product — the rules that operational teams and systems must obey. Standards make policies concrete; procedures and guidelines make them executable.

### 5.1 The Policy Hierarchy (Policy Pyramid)

```
Policy            "Risk data shall be accurate and complete"        (what, why — board-level)
  └─ Standard     "Risk data accuracy ≥ 99.5% per BCBS 239 P6"      (measurable rule)
       └─ Procedure "Monthly profiling run; issues > 48h escalate"   (step-by-step how)
            └─ Guideline "Use Soda checks for profiling"             (advisory, best practice)
```

Policies are deliberately short (one page), written in business language, and approved at council level. Standards carry the measurable detail. Procedures belong to the teams that execute them (and change frequently — they should *not* need council approval). The classic error is writing procedures at policy level: unchangeable detail that goes stale immediately.

### 5.2 Key Policies

- **Data Classification Policy** — defines levels and how data is labeled. Bank-standard levels: **Public / Internal / Confidential / Restricted** (restricted = regulated, PII, or market-sensitive; often further split per MAS/regulatory categories). Classification is the input to access, masking, and retention decisions; in banking it typically mirrors regulatory categories (customer data, payment data, risk data, salary/HR data).
- **Data Access Policy** — the **need-to-know** and **least-privilege** principles; approval workflows per classification (self-service for Internal, owner approval for Confidential, owner + security for Restricted); access reviews (quarterly for privileged/restricted); logging and monitoring of access.
- **Data Retention Policy** — retention periods per data class and disposal/destruction method. Regulatory anchors: Singapore AML/CFT requirements keep CDD and transaction records for at least **5 years** after the end of the business relationship; MAS and other supervisors impose similar floors; tax and trade-finance records can run longer. Retention policy is where governance meets storage cost (§2.2) — most banks hold far more than they are required to, and defensible deletion is a governance capability.
- **Data Quality Policy** — quality standards and tolerances per domain: which dimensions are measured, thresholds, SLAs for issue resolution (see §6).
- **Data Sharing Policy** — internal and external sharing rules; third-party/vendor data intake and exit; data-sharing agreements; the *receiving* side of the policy (what the bank may do with vendor data) is as important as the giving side.
- **Data Protection Policy** — PII handling under PDPA/GDPR: lawful basis, minimization, transfer rules, breach response; the bridge between the governance framework and the privacy office.
- **Data Ethics Policy** — ethical use of data, especially for AI: no discriminatory use, transparency about automated decisions, human oversight; increasingly referenced by AI governance frameworks (§13.1).

### 5.3 Standards

- **Naming standards** — consistent naming for tables, columns, files, and jobs (a naming convention is the cheapest lineage tool that exists).
- **Data definition standards** — the **business glossary** as the arbiter of meaning: one definition per term, with aliases and owners (§7.2).
- **Reference data standards** — authoritative sources for codes and enums: ISO 4217 currencies, ISO 3166 countries, internal product/cost-center codes (§8.3).
- **Data exchange standards** — message and file formats for internal and external exchange (ISO 20022 in payments is the canonical banking example; see the `iso_20022_core_processes_guide.md` in `../banking/`).

### 5.4 Policy Lifecycle and Enforcement

The policy lifecycle is: **draft → review → approval → communication → training → enforcement → periodic review** (typically annual, or on regulatory change). Three enforcement mechanisms make policies real:

1. **Controls** — preventive and detective: access rules in the platform, quality checks in pipelines, schema contracts (§9.2).
2. **Monitoring** — continuous measurement: policy-compliance dashboards (e.g., % of assets classified, % of access requests approved on time, % of data products with documented owners).
3. **Attestations** — periodic sign-offs: owners attest annually that their domain complies (or documents exceptions); attestation is how governance reaches the board and the auditor in a defensible form.

### 5.5 Example: A One-Page Policy Template

A well-formed policy is one page, in business language, with the five blocks below (example drawn from a real-world risk data policy):

| Block | Content (example) |
|-------|-------------------|
| **Purpose** | Ensure risk data used for regulatory reporting is accurate, complete, and timely, per BCBS 239 principles. |
| **Scope** | All data feeding COREP, MAS returns, and internal risk reporting; owned by the risk divisions. |
| **Roles** | Data owners (division heads) are accountable; stewards operate quality and definitions; DGO administers. |
| **Rules** | Quality tolerances per dimension (accuracy ≥ 99.5%, completeness 100% on regulatory fields); issue escalation path with SLAs; definitions changes require council sign-off. |
| **Compliance** | Controls (automated checks), monitoring (monthly scorecard), attestation (owners sign twice a year); breaches escalate per the incident path. |

The template test: if a new owner can read the policy and immediately know what they must do, what they may not do, and what happens if they do not — it is a policy. If they need a lawyer or a 40-page handbook to understand it, it is not.

### 5.6 Communication and Training

Policies fail silently when they are published but not understood. The operating rule: **every policy ships with a one-page "what this means for you" summary, a named training module, and a contact**. New joiner onboarding includes data governance basics (classification, access, glossary use); role-specific training covers stewards/owners (how to run a domain forum, how to use the catalog) and data engineers (quality-as-code, contracts); annual refreshers track completion per role, with completion rates reported to the council. The unglamorous truth: a bank with 20 well-communicated policies outperforms one with 200 published-but-unread ones. Communication is not a launch activity — it is the ongoing loop that converts policy text into behavior, which is what governance actually is.

## 6. Data Quality Management

Data quality is the most tangible output of governance: it is measurable, it fails visibly, and it is what regulators audit first. Governance defines *who owns quality and what the rules are*; quality management runs the measurement and remediation.

### 6.1 Quality Dimensions

DAMA-DMBOK's seven dimensions are the standard vocabulary (BCBS 239 uses a tighter regulatory subset — see §6.4):

| Dimension | Definition | Typical measurement |
|-----------|------------|---------------------|
| **Accuracy** | Data reflects the real-world object/event correctly | % of records matching the source of truth; error rate on sampled values |
| **Completeness** | All required data is present | % of rows/fields non-null vs. required set; % of expected records received |
| **Consistency** | Same value across systems, no contradictions | % of records where the same entity agrees across sources; referential integrity violations |
| **Timeliness** | Data is available when needed | Data latency vs. SLA (e.g., T+1 for risk reporting); staleness of the freshest record |
| **Validity** | Values conform to defined domains/format | % of values passing format/domain/reference checks (valid currency codes, valid dates) |
| **Uniqueness** | No unintended duplicates | Duplicate rate per entity; match-rate on entity resolution |
| **Integrity** | Data and relationships are structurally sound | Referential-integrity violations; orphaned records |

Measurement discipline: every dimension needs a **measurable rule with a threshold and an owner** ("accuracy ≥ 99.5% on risk exposure fields, owned by Risk Data Steward"). A dimension without a threshold is a topic, not a control.

### 6.2 The Quality Process

- **Assessment** — data profiling (statistics, distributions, null rates, duplicates, domain violations) to understand current state; the first profiling pass usually shocks the business into funding the program. See [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) §7 for quality in the integration context.
- **Issue management** — the quality issue lifecycle: **log → triage → assign → fix → verify → close**, with severity-based SLAs (critical regulatory-data issues in hours, cosmetic issues in weeks) and an escalation path to the council for stuck issues. An issue tracker with owners and dates is non-negotiable; email-based quality management does not exist.
- **Monitoring** — continuous, automated measurement with dashboards and alerts; quality SLAs reported to owners monthly and to the council quarterly; "quality scorecards" per domain (see the worked example §12).
- **Improvement** — root-cause fixes at the source (fix the system of record, not the report), process changes, and prevention (quality rules moved into the pipeline as tests — see §6.3).

### 6.3 Quality Tooling

| Tool | Model | Strengths | Notes |
|------|-------|-----------|-------|
| **Informatica / Talend** | Commercial E2E suites | Profiling, monitoring, MDM integration, enterprise scale | Traditional bank stack; heavy |
| **Collibra / Alation / Atlan** | Commercial catalog+quality | Quality rules co-located with glossary/lineage | Governance-first platforms (§7.3) |
| **Ataccama** | Commercial | Strong profiling + ML-assisted rule discovery | Mid-market favorite |
| **Great Expectations** | Open source | Declarative expectations as code, data docs, CI-friendly | The "testing framework for data"; Python |
| **Soda** | Open source + cloud | Lightweight checks, anomaly detection, dbt-friendly | Modern stack; strong SQL-first checks |
| **dbt tests** | Open source (dbt) | Built-in singular/generic tests on transformed models | Quality at the transformation layer |
| **Monte Carlo / Anomalo** | Commercial (observability) | ML-based anomaly detection, freshness/volume monitoring | "Data observability" category; complements rule-based tools |

The modern pattern: **quality as code** — rules versioned in the repo, run in CI/CD on every pipeline change, and monitored in production. Great Expectations/Soda/dbt tests form the OSS core of this; commercial catalogs add governance context (who owns the rule, what it means). In banking, both the OSS rule layer and the governed catalog layer are usually present, with the catalog layer carrying the regulatory evidence.

### 6.4 Data Quality in Banking

BCBS 239 defines the **regulatory quality dimensions**: **accuracy, completeness, timeliness, and adaptability** (principles 6–9; see §10.1 and [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) §2.2). Banks operationalize these as: accuracy and integrity of risk data (P6), completeness of risk data (P7), timeliness of aggregation (P8), and adaptability to changing requirements (P9 — can the bank produce new reports without re-architecting?). Supervisors also scrutinize **MAS reporting accuracy** (regulatory returns submitted to MAS must reconcile to the books) and the quality of **risk inputs** — credit risk data (exposures, PD/LGD inputs), market risk data (positions, curves, prices), liquidity data — because risk models are only as good as their inputs. The regulatory pattern: quality is not a "nice to have" metric; it is a *demonstrated control* with evidence, ownership, and a remediation track record.

### 6.5 Writing a Quality Rule

A quality rule is a small, testable contract. The modern tooling (§6.3) expresses rules as code — the same rule rendered in three styles:

```sql
-- Soda: SQL check — completeness of a regulatory field
SELECT COUNT(*) AS total, COUNT(credit_exposure) AS filled
FROM risk_credit.exposure_summary
WHERE report_date = :today;
-- fail if filled/total < 1.0

-- Great Expectations (Python): accuracy of currency codes
expect_column_values_to_be_in_set(
    column="currency",
    value_set=["SGD","USD","EUR","JPY","HKD","CNY","GBP","AUD"],
    mostly=1.00)

-- dbt test (YAML): uniqueness of the exposure key
models:
  - name: exposure_summary
    tests:
      - unique:
          column_name: exposure_id
          severity: error
```

Rule-writing discipline: one rule per dimension per critical field, owned by a steward, threshold tied to a policy tolerance (§5.2), and a *documented failure response* (what happens when the rule trips — alert, block, or quarantine). The three-part test for a rule: it is specific (exactly what is measured), it has a threshold with an owner, and tripping it has a defined consequence. Rules that fail the test are decoration.

### 6.6 The Data Quality Scorecard

The scorecard is the governance artifact that makes quality visible to the council. A domain scorecard (one per domain, monthly) looks like:

| Domain: Customer Master | Accuracy | Completeness | Consistency | Timeliness | Trend |
|-------------------------|----------|--------------|-------------|------------|-------|
| Critical fields (name, ID, address) | 99.1% ✓ | 98.4% ✓ | 96.2% ▲ | T+1 ✓ | 2 of 4 at target |
| Regulatory fields (tax ID, country) | 99.8% ✓ | 100% ✓ | 99.5% ✓ | T+1 ✓ | at target |
| Reference fields (segment, tier) | 94.0% ▲ | 95.1% ▲ | 92.3% ✗ | T+3 ▲ | remediation owner named |

Scorecard discipline: same format every month (trend matters more than level), every red cell has a named remediation owner and a due date, and the scorecard rolls up into the governance KPI pack (§11.4). A scorecard with no red cells is either perfect or lying — auditors know which one it is. The scorecard is also the natural place to record *quality SLAs* per domain (the Data Quality Policy's tolerances, §5.2) so target-vs-actual is always visible.

## 7. Metadata Management

Metadata is "data about data," and metadata management is the foundation everything else sits on: you cannot govern, quality-check, or report on data you cannot find, name, and trace. The governing insight — *govern the metadata first* — is that a catalog with definitions and lineage is the substrate on which every other governance control operates.

### 7.1 Metadata Types

| Type | What it is | Examples |
|------|-----------|----------|
| **Technical metadata** | Structure of the data asset | Schemas, columns, data types, partitions, database catalogs |
| **Business metadata** | Meaning and usage | Glossary terms, definitions, owners, business rules, tags |
| **Operational metadata** | How data moves and performs | Run logs, job status, pipeline durations, freshness, error rates |

All three must be captured and linked; a catalog that stores schemas but no definitions is a dictionary without meaning, and one with definitions but no lineage cannot answer "where did this number come from?"

### 7.2 The Business Glossary

The glossary is the governance foundation: a controlled vocabulary where every business term has **one canonical definition, aliases (including bad/legacy names), a domain owner, and an approval workflow** (new term → steward drafts → owner approves → published). Glossary-driven development means new reports, models, and pipelines must reference glossary terms rather than inventing local definitions — the mechanism that kills "same name, different meaning" disputes. In banking, the glossary is also the natural place to encode regulatory definitions (what "credit exposure" means for BCBS 239 reporting vs. for the credit-risk model vs. for finance) — one term, multiple governed definitions, each scoped to its use.

### 7.3 The Data Catalog

The catalog is the metadata repository with a UI. Core functions:

- **Discovery & search** — find assets by name, term, tag, or column.
- **Profiles & previews** — see schemas, sample data, statistics without querying.
- **Tagging & classification** — apply classification labels (public/confidential/restricted) and custom tags; the operational arm of the classification policy.
- **Certification** — designated "certified" assets that have passed governance review and can be consumed with confidence; certified-vs-uncertified is the catalog's quality signal.
- **Access requests** — request access with justification, routed to the owner (ties into the access policy §5.2).

Tooling landscape:

| Tool | Open source? | Strengths | Notes |
|------|--------------|-----------|-------|
| **Collibra** | No | Deep governance workflow, glossary-first, banking-heavy install base | The enterprise governance platform |
| **Alation** | No | Strong search/ML-assisted tagging, BI integration | Popular in analytics orgs |
| **Atlan** | No | Modern UX, developer-friendly, active metadata | Fast-growing; strong for lakehouse stacks |
| **DataHub** | Yes (LinkedIn) | Metadata platform, automated lineage, ML model metadata | Apache-2.0; strong for engineering teams |
| **Amundsen** | Yes (Lyft) | Search/discovery-focused | Lighter governance; less active development |
| **Apache Atlas** | Yes | Hadoop-era governance, lineage, classification | Legacy Hadoop/Hive stacks |
| **OpenMetadata** | Yes | Full catalog + glossary + lineage + quality in one | Actively developed, broad feature set |
| **Unity Catalog** | Yes (Databricks, OSS since June 2024) | Lakehouse-native: tables, models, volumes; fine-grained ACLs | See [databricks_guide.md](databricks_guide.md) §7 |

### 7.4 Data Lineage

**Lineage** traces data from source through transformations to consumption. Types: **table-level** (which tables feed which), **column/field-level** (which columns map to which), and **end-to-end** (source system → transformation → report/metric — the form regulators want). Lineage comes from: manual mapping, pipeline/job metadata, and **automated parsing** of SQL and ETL jobs (the modern catalogs do this continuously).

Lineage use cases:

- **Impact analysis** — before changing a source schema or job, see every downstream report, model, and KPI that would break (see [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md)).
- **Root-cause analysis** — when a number is wrong, walk the lineage to the broken step.
- **Audit and regulatory** — BCBS 239 requires banks to demonstrate that reported risk numbers can be traced to source data (see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) §9.2); lineage is the evidence.
- **Data product documentation** — lineage is the "how was this made" section of any data product's documentation (§9.3).

### 7.5 Metadata Strategy

The strategy question is not "which tool" but **"govern the metadata first"**: build the glossary and inventory before the quality rules, because rules without defined terms and owners are unenforceable. Practical sequence: inventory critical assets → define glossary terms for them → capture lineage on critical paths → attach owners and quality rules → certify. Metadata coverage (what % of critical data assets are catalogued, defined, owned, and traced) is itself a governance KPI (§11.4).

### 7.6 Lineage in Practice: A Worked Trace

End-to-end lineage for a single reported number (the form a BCBS 239 auditor asks for) looks like this — every hop documented with the transformation that produced it:

```
COREP C.07 "Total credit risk exposure" (report line R180)
        ▲  mapped from: SUM(exposure) GROUP BY legal_entity, exposure_class
        │
risk_credit.exposure_summary (warehouse table, certified, owner: Credit Risk)
        ▲  ECL engine output: exposure = EAD × (1 - CCF); joins trade + collateral
        │
credit_risk_dataset (integration layer, daily load T+1 02:00)
        ▲  CDC from trade capture + collateral system
        │
front_office.trades  (system of record: trade capture, deal table)
```

The column-level mappings, transformation logic, and job names live in the catalog; the diagram is the human-readable view. Two tests for good lineage: **walk it** — can a new analyst follow a number to its source in under an hour? — and **reverse-walk it** — if the source table changes, does the catalog tell you every report line that moves?

### 7.7 The Metadata Operating Model

Metadata needs an owner too — otherwise the catalog decays. The working model: the **DGO owns the catalog platform and standards** (naming conventions, glossary workflow, certification criteria); **domain stewards own their domain's metadata** (terms, definitions, lineage accuracy); **platform/engineering teams keep technical and operational metadata flowing** (automated ingestion, job metadata, schema updates); and **data owners approve certification**. The daily reality: metadata decays fastest at the *business* layer (definitions go stale, owners move on), so the glossary refresh is a standing steward activity with a coverage KPI, not a one-time project. The test of the operating model: six months after launch, is catalog coverage *rising* (healthy) or *flat/falling* (decaying)? Decay is the default state of metadata; the operating model exists to fight it.

## 8. Reference and Master Data Management

Master and reference data are the highest-value, highest-dispute data in any organization: everyone uses the customer, the product, the currency code — and everyone has their own version. Governance of these is where the "single source of truth" promise is actually won or lost.

### 8.1 Master Data Management (MDM)

**Master data** describes the core business entities: **Party/Customer, Product, Account, Counterparty, Employee, Location, and Agreement/Contract** (the canonical entity set; see [data_model_resource_book_guide.md](data_model_resource_book_guide.md) and [data_models_banking_insurance_guide.md](../banking/data_models_banking_insurance_guide.md) §2 for the models). MDM creates and maintains the **golden record** — the single, best-quality, agreed version of each entity, surviving across all source systems. MDM architecture styles:

| Pattern | How it works | Pros | Cons | Typical use |
|---------|--------------|------|------|-------------|
| **Registry** | Sources stay in place; MDM holds matched keys and a minimal golden record | Fast to deploy, no system rewrites | Limited enrichment; quality not enforced at source | Rapid consolidation of customer identity |
| **Consolidation** | MDM aggregates master data from sources into a central store; writes back | Good for analytics and 360 views | Stale between refreshes; not real-time | Customer 360 for analytics/CRM |
| **Coexistence** | MDM distributes the golden record to subscribing systems; sources continue to operate | Balance of central control and system autonomy | Complex sync and ownership rules | Bank-wide customer master |
| **Transaction-based (centralized)** | MDM is the system of record; all creates/updates go through it | Strongest control and consistency | Slowest, most disruptive to deploy | New greenfield platforms; regulatory-driven entities |

### 8.2 Customer Data Management and the Golden Record

Customer master is the flagship MDM use case in banking (the **customer 360** — everything the bank knows about a customer joined through one identity; see [data_models_banking_insurance_guide.md](../banking/data_models_banking_insurance_guide.md) for the customer entity model and identifier-resolution discussion). The core technical machinery is **entity resolution**:

1. **Matching** — decide two records refer to the same entity (deterministic rules + probabilistic scoring on name, ID, address, phone, tax ID).
2. **Merging/survivorship** — build the golden record by selecting the best value per field from contributing sources (survivorship rules: which source wins for name, for address, for risk rating).
3. **Deduplication** — collapse duplicates in a single source; ongoing monitoring keeps the duplicate rate in check (the *uniqueness* quality dimension, §6.1).

The golden record carries **source and confidence metadata** (which system supplied each field, at what confidence) — regulators and risk teams need to know the provenance of the customer identity behind an exposure. MDM tooling: **Informatica MDM, IBM InfoSphere MDM, SAP Master Data Governance, Reltio, Semarchy** (commercial); the open-source MDM landscape is thin (no mainstream OSS MDM — banks treat MDM as a buy decision, often integrated with the governance catalog).

### 8.3 Reference Data Management (RDM)

**Reference data** is the controlled vocabulary of codes and values the organization runs on: currencies (ISO 4217), countries (ISO 3166), business days/calendars, internal product codes, cost centers, holiday calendars. RDM governs the **golden source** (which system is authoritative for each reference set), **change control** (who can add a currency or product code, with what approval — adding codes without governance is how reporting chaos starts), and **distribution** (how updates propagate to every consuming system with versioning and timeliness). In banking, reference data splits into:

- **Regulatory reference data** — legal entity identifiers (**LEI**, ISO 17442, mandatory for OTC derivative reporting), **BIC** (ISO 9362), **ISIN** (ISO 6166); quality and timeliness of this data is itself regulated.
- **Market reference data** — instruments, prices, curves, corporate actions — the lifeblood of market risk and trading systems (see the Calypso/Murex coverage in the `banking/` guides, e.g. [nasdaq_calypso_guide.md](../banking/nasdaq_calypso_guide.md)).

RDM is deceptively simple and chronically under-funded: a bad currency code or a stale corporate action propagates into every downstream system, and reference data quality issues are a recurring BCBS 239 finding.

### 8.4 Master Data in Banking: Regulatory Master Data

Banking adds a regulatory layer on top of generic MDM: certain master data entities are *themselves* governed by regulation. The **counterparty master** is the prime example — the legal-entity view of every counterparty (with LEI linkage, country of risk, group structure) is the input to large-exposure reporting, credit concentration limits, and BCBS 239 aggregation; a counterparty master with duplicates or stale LEIs produces wrong regulatory numbers regardless of how good the reporting pipeline is. Similarly, **product master** data (product type hierarchies, regulatory product classifications) drives risk-weighting and P&L attribution. In practice, regulatory master data programs pair the MDM tool with the governance machinery: the counterparty owner sits in credit risk, the golden record carries source/confidence metadata (§8.2), and the RDM function maintains the LEI/BIC/ISIN reference sets (§8.3). Banks that skip this layer find their "single customer view" projects succeed while their regulatory numbers still fail reconciliation — because the master was built for sales, not for risk.

### 8.5 MDM in the Lakehouse Era

The modern data platform changes where MDM logic lives. The **lakehouse catalog as the registry**: with Unity Catalog / Iceberg metadata (§9.1), the *catalog* can serve as a lightweight registry-style MDM layer — entity keys, golden-record pointers, and survival rules as governed metadata, without a heavyweight MDM appliance. The **"MDM as a data product"** pattern: the golden record is published as a certified data product (§9.3) — versioned, contracted, and consumed by both operational systems (via API) and analytics (via the lakehouse) — rather than being locked in a proprietary MDM repository. In practice banks run a spectrum: heavyweight MDM (Informatica/IBM/Reltio) where operational write-back and real-time entity resolution are needed, lakehouse-native registry for analytic 360 views, and contracts between them. The governance requirement is the same in both: *one owned golden record with provenance*, regardless of where the code runs.

## 9. Governance in the Data Platform

Governance has moved from a back-office function to a **platform capability**: the modern data platform (lakehouse, mesh, AI platform) encodes governance rules directly — access, lineage, quality, and policy enforced by the platform rather than by manual process.

### 9.1 Lakehouse Governance

The lakehouse (see [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) and [databricks_guide.md](databricks_guide.md)) reintroduced warehouse-grade governance to data lakes:

- **Central catalog** — Unity Catalog (Databricks, open-sourced June 2024) or Iceberg/Hudi metadata layers give tables, schemas, and fine-grained ACLs a single governed namespace across engines.
- **Table-level governance** — schema enforcement (schema-on-write), partitioning, and table properties; Iceberg/Hudi metadata enables time travel and controlled evolution (see [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md)).
- **Data quality in the pipeline** — checks run as part of ingestion/transformation (Great Expectations/Soda/dbt tests, §6.3), so bad data is stopped at the boundary, not discovered in the dashboard.
- **The lakehouse governance trio** — *catalog* (find it), *lineage* (trace it), *quality* (trust it) are the three capabilities supervisors ask about; a lakehouse with all three is the modern answer to "where is your data warehouse governance?"

### 9.2 Data Contracts

A **data contract** is the explicit producer–consumer agreement: **schema, semantics (glossary terms), quality SLAs, freshness/availability, and contact owners**, versioned and machine-readable. Contract-first data sharing means producers publish against contracts and consumers build against them, so schema and quality changes are negotiated, not discovered (deep coverage in [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md) §10). For governance, contracts are the enforcement point: quality checks, schema validation, and access rules hang off the contract, making policy *structural* rather than aspirational.

### 9.3 Data Products

**Data product thinking** treats governed datasets as products with: a **product owner** (accountable domain role), **SLAs** (freshness, quality, availability), **documentation** (glossary, lineage, usage examples), **discoverability** (in the catalog, with certification), and **versioning** (contract-based evolution). The data product is the unit that reconciles mesh-style domain autonomy (§4.5) with governance standards: products must meet firm-wide governance requirements (classification, quality thresholds, lineage) to be published — that is the "federated computational governance" principle in action.

### 9.4 Data Access Governance

Access control is where governance meets security engineering:

- **RBAC** — role-based access: users get roles, roles get permissions; the baseline model.
- **ABAC** — attribute-based access: decisions from attributes (user location, data classification, time) — finer-grained, policy-driven, increasingly the model for zero-trust data (§13.5).
- **Row-level security (RLS) and column-level security** — data-level controls: a risk analyst sees only their desk's rows, or a masked PII column.
- **Masking and tokenization** — dynamic data masking (mask at query time based on viewer), static masking (test/dev environments), tokenization (replace PII with reversible tokens — the foundation of test-data management and, increasingly, of clean-room analytics).
- **Data residency** — data localization constraints (e.g., China's Data Security Law / PIPL require certain data to remain in-country; see [chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md) §7); residency is a governance decision (which data can live in which region/cloud) enforced by platform policy.
- **Clean rooms** — data clean rooms let two parties run queries/analytics on joined data **without either seeing the other's raw records** (privacy-enhancing technologies: secure multiparty computation, differential privacy); increasingly used for shared analytics with customers/vendors and for marketing data collaboration.

### 9.5 The AI/Data Governance Intersection

AI is now the sharpest edge of data governance:

- **Training data governance** — provenance, licensing, quality, and bias of training data are becoming regulated requirements (EU AI Act, §13.1); "AI-ready data" means documented, licensed, bias-audited, and versioned training/evaluation sets (see the `ai_llm/` guide series).
- **Model governance** — model risk management (US guidance SR 11-7; see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) §12.4) treats models as regulated assets: inventory, validation, monitoring, retirement — the model analogue of data governance.
- **Feature and embedding governance** — features and embeddings are derived data assets with their own lineage, quality, and versioning needs (see [feature_store_guide.md](feature_store_guide.md) §11).
- **Synthetic data** — synthetically generated data (for testing, model training, sharing) needs governance too: generation provenance, fidelity metrics, and re-identification risk controls. Synthetic data is a governance *solution* (share without exposing PII) and a governance *problem* (is it representative? is it safe?).

### 9.6 The Platform Operating Rhythm

Platform-era governance runs on a **rhythm of automated controls with human escalation**, not meetings:

- **Continuous** — quality checks run with every pipeline; access policy evaluated on every query; lineage captured on every job. No human in the loop.
- **Daily** — data quality alerts triaged by stewards; failed contracts quarantined; incident response for data issues per severity SLA.
- **Monthly** — steward forums review scorecards; catalog certification reviews; access review exceptions.
- **Quarterly** — council reviews KPIs, approves policy changes, resolves escalations; owners attest.

The platform makes the *continuous* layer cheap (policy-as-code, §13.4) and the *human* layers focused: humans stop inspecting data and start governing rules. The operating rhythm is also the answer to the classic audit question "how do you know your controls are working?" — the answer is a dashboard showing checks run, rules tripped, and escalations closed, all with evidence.

## 10. Banking and Regulatory Context

Regulated industries — and banking above all — are where data governance stops being optional. This section deep-dives the two frameworks most relevant to Singapore banking (BCBS 239 and MAS), then maps the wider regulatory landscape.

### 10.1 BCBS 239 in Depth

**BCBS 239** — the Basel Committee's *Principles for Effective Risk Data Aggregation and Risk Reporting* (January 2013) — is the single most consequential data governance regulation ever written. It exists because the 2008 crisis exposed that banks could not aggregate their own risk positions accurately or quickly: supervisors could not get comparable data, and banks could not tell their own exposure in time. Its 14 principles:

| Group | Principles | Focus |
|-------|-----------|-------|
| **Governance & infrastructure** | 1–5 | P1: governance (data governance and risk data oversight); P2: data architecture and IT infrastructure; P3: accuracy and integrity; P4: completeness; P5: timeliness |
| **Risk data aggregation** | 6–9 | P6: accuracy/integrity of aggregated risk data; P7: completeness; P8: timeliness of aggregation; P9: adaptability (ability to meet new reporting demands) |
| **Risk reporting** | 10–13 | P10: accuracy; P11: comprehensiveness; P12: clarity/usefulness; P13: frequency |
| **Supervisory review** | 14 | P14: supervisory review of compliance — supervisors assess and enforce |

Key principles in practice: **P1 (governance)** requires a formal data governance framework for risk data — ownership, stewardship, and escalation — making BCBS 239 effectively a *mandate to build data governance*. **P2** requires architecture that supports complete, accurate, timely aggregation (no shadow-Spreadsheet risk reporting). **P6/P7/P8/P9** define the four regulatory quality dimensions (accuracy, completeness, timeliness, adaptability — §6.4). **P13** requires frequent, on-demand reporting (daily/weekly for key metrics, with intra-day drill-down). **P14** is the enforcement mechanism: supervisors rate each principle and escalate persistent gaps into capital add-ons.

**The compliance reality.** Compliance has been genuinely hard: the Basel Committee's November 2023 progress report (covering 31 G-SIBs) found **only 2 of 31 G-SIBs fully compliant with all principles, and no single principle fully implemented by all banks** — ten years after publication. Repeated supervisory findings concentrate on data architecture (P2), governance (P1), and aggregation accuracy (P6). The lesson for practitioners: BCBS 239 is not a project with an end date; it is a permanent capability, and the banks that treat it as such (governance operating model + lineage + quality evidence) are the ones supervisors stop repeating findings about. Full BCBS 239 coverage: [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) §2.2 and §9.2.

**BCBS 239 as a governance driver.** For most banks, BCBS 239 created the CDO role, funded the first enterprise data catalog, and built the first firm-wide risk data lineage. It is the reference point for every governance decision in banking: if a governance artifact (policy, RACI, quality rule, lineage map) can be pointed at a BCBS 239 principle, it survives budget cycles; if it cannot, it is at risk.

### 10.2 The MAS Context (Singapore)

MAS has built a layered, notice-based expectation set for technology and data:

- **TRM framework evolution** — the 2013 *Technology Risk Management Guidelines* (voluntary guidance applying across FIs) were succeeded by **binding Notices under the Financial Services and Markets Act 2022**: **Notice FSM-N05 (Technology Risk Management)** and **Notice FSM-N06 (Cyber Hygiene)**, effective for specified FIs from **10 May 2024**. These make technology risk management — including data integrity, availability, and incident management — a legal requirement, not guidance. (Note: MAS **Notice 658** is the *outsourcing/third-party risk* notice (issued December 2023, effective 11 December 2024), not a cyber-hygiene notice — a common mix-up.)
- **Data management expectations** — MAS supervisory guidance and thematic reviews have repeatedly flagged data quality and governance in regulatory reporting, AML systems, and risk management; MAS expects documented data governance for material data, with ownership, quality monitoring, and evidenced remediation.
- **SG banking practice** — Singapore banks (DBS, OCBC, UOB) run mature data offices: group CDOs, domain-steward operating models, enterprise catalogs, and BCBS 239 programs for their risk data; their governance models are effectively the reference implementations of the hybrid operating model (§4.2) in Asia. DBS in particular has publicized its data-driven transformation with a data/analytics operating model that pairs a central data office with domain-aligned squads.
- **PDPA** — the *Personal Data Protection Act* (enforced by the PDPC) governs PII: consent, purpose limitation, retention limits, and data-breach notification; PDPA obligations are woven into bank data governance through the data protection policy (§5.2). See also the privacy-angle coverage of GDPR/PDPA in the data-protection sections of the banking guides.

### 10.3 The Regulatory Landscape

| Regulation | Jurisdiction | Data governance requirement | Applies to |
|-----------|--------------|-----------------------------|------------|
| **BCBS 239** | Global (Basel) | Risk data governance, architecture, quality (accuracy/completeness/timeliness/adaptability), lineage, reporting | G-SIBs (and, via local adoption, large banks) |
| **MAS TRM Notices (FSM-N05/N06)** | Singapore | Technology risk framework, data integrity/availability, cyber hygiene | Banks and specified FIs under FSMA |
| **MAS Notice 658 / 1121** | Singapore | Third-party/outsourcing risk management, including data shared with vendors | Banks and merchant banks |
| **PDPA** | Singapore | PII governance: consent, purpose, retention, breach notification | All organizations handling SG PII |
| **GDPR** | EU | PII governance, lawful basis, data minimization, deletion rights, breach notification | Any organization processing EU residents' data |
| **DORA** | EU | ICT risk management framework; data integrity and availability for critical functions | EU financial entities (applied 17 Jan 2025) |
| **EU AI Act** | EU | Data governance for high-risk AI: training data quality, provenance, bias (Art. 10) | Providers/deployers of AI systems in the EU (see §13.1) |
| **IFRS 17** | Global (accounting) | Granular, governed contract-level data for insurance accounting | Insurers (effective 2023) |
| **SOX §404** | US | IT general controls over financial data: access, change management, evidence | US-listed companies |
| **SR 11-7** | US | Model risk management (model inventory, validation, governance) | US banks (model governance — §9.5) |
| **China DSL / PIPL** | China | Data classification, localization, cross-border transfer controls | Organizations handling China data (see [chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md) §7) |

The convergence is unmistakable: privacy, resilience, AI, and prudential regulation are all landing on the same set of data governance capabilities — ownership, classification, quality, lineage, and access control. A bank that builds those capabilities once, well, can answer most of this table with the same evidence.

### 10.4 BCBS 239 Principle → Governance Response Map

The practical value of BCBS 239 for a governance program is that each principle maps to a concrete governance capability — a useful self-assessment device:

| Principle | Supervisory question | Governance response |
|-----------|----------------------|---------------------|
| P1 Governance | Who governs risk data? | Council + risk data owners/stewards; charter; escalation path |
| P2 Architecture | Can data be aggregated without shadow systems? | Target architecture; platform governance; decommissioning of spreadsheets |
| P3/P4/P5 | Is risk data accurate, complete, timely? | Quality rules + scorecards on risk data domains |
| P6–P9 | Is *aggregated* risk data accurate/complete/timely/adaptable? | End-to-end lineage; aggregation-layer quality; report-level SLAs; change management |
| P10–P13 | Are reports accurate, comprehensive, clear, frequent? | Report certification; glossary-driven definitions; on-demand reporting capability |
| P14 | Can supervisors verify all of the above? | Evidence pack: lineage maps, quality dashboards, attestations, audit trail |

### 10.5 What Supervisors Actually Look For

Across BCBS 239 reviews and MAS thematic inspections, the same evidence requests recur: **ownership** (who owns this data — a name, not a team), **definitions** (what does this term mean — glossary, with regulatory scoping), **quality evidence** (measurements over time, not a one-off report), **lineage** (trace a reported number to source), **change control** (what happens when a source or definition changes), and **remediation track record** (issues found, fixed, closed, on time). Programs that prepare this evidence *continuously* (because it is the byproduct of their operating model) pass reviews with minor findings; programs that prepare it *for the review* get the "sustainability" finding — the most common supervisory criticism of governance programs.

## 11. Implementation Roadmap

Governance programs fail from over-scoping more often than from under-funding. The proven path is: assess where you are, design a target operating model, then build in value-ordered phases — starting with foundations, not tools.

### 11.1 Maturity Assessment and Target Operating Model

Start with a **current-state assessment** against a maturity model (CMMI DMM or DCAM — §3): score each capability area, identify the gaps that matter (usually: no owners, no glossary, no lineage, no quality evidence, no policy enforcement). The assessment produces the evidence base for the business case (§2.3). Then design the **target operating model (TOM)**: operating model choice (hybrid is the default, §4.2), role definitions and staffing, forum cadence, policy inventory, tooling footprint, and KPI framework. The TOM is the one document the program must get right; everything else follows from it.

### 11.2 Phases

| Phase | Focus | Typical duration | Key deliverables |
|-------|-------|------------------|------------------|
| **1. Foundations** | Roles, policies, glossary seed, pilot domain | Months 1–3 | CDO/council/stewards appointed; classification + access + retention policies; first 100 glossary terms; one pilot domain fully owned |
| **2. Metadata & catalog** | Inventory, lineage, certification | Months 3–9 | Critical assets inventoried; end-to-end lineage on critical paths (risk/finance reporting); catalog live with search + access requests |
| **3. Data quality** | Measurement, issue management, monitoring | Months 6–14 | Quality rules + dashboards on critical domains; issue lifecycle live; quality SLAs in place; BCBS 239 evidence pack starts |
| **4. MDM / RDM** | Golden records, reference data control | Months 9–18 | Customer master (coexistence pattern) live for priority entities; reference data golden sources + change control |
| **5. Continuous governance** | Culture, automation, expansion | Months 12–24+ | Attestations annualized; policy-as-code on the platform; governance KPIs in the board pack; program becomes BAU |

The phases overlap deliberately — quality monitoring (3) starts as soon as the catalog (2) can host its rules, and MDM (4) is often accelerated by a customer-360 business case. A realistic journey is **12–24 months to "credible BAU"** and 3+ years to maturity; anyone promising governance-in-a-year is selling tooling.

### 11.3 Success Factors

- **Executive sponsorship that spends** — the council must include budget holders; a governance program sponsored only by the CTO is an IT project (see pitfalls).
- **Business involvement from day one** — stewards and owners are business roles; if the program is staffed entirely by IT, it is already dead.
- **Start small** — one pilot domain (Risk or Customer), fully governed, visibly working, before enterprise rollout; pilots convert skeptics.
- **Deliver incremental value** — every phase must produce something the business can use (a glossary that answers a dispute, a lineage map that saves a migration, a quality dashboard that fixes a report).
- **Communication and training** — governance is a behavior change; named training, FAQ, and a visible "ask the DGO" channel are part of the design, not afterthoughts.
- **Metrics that measure the program, not the tool** — see §11.4.

### 11.4 Governance KPIs

| Category | Example KPIs |
|----------|--------------|
| **Coverage** | % of critical data assets inventoried / glossary-defined / lineage-traced / certified |
| **Quality** | Domain quality scores by dimension; % of critical rules passing; issue backlog age |
| **Process health** | % access requests approved within SLA; % policies attested; issue closure rate; council decision turnaround |
| **Regulatory** | BCBS 239 principle ratings; audit findings count; remediation on-time % |
| **Value** | Rework hours saved; storage retired; data product count and adoption |

KPI discipline: report the same numbers monthly to the DGO and quarterly to the council, and let the numbers (not anecdotes) drive the agenda.

### 11.5 Pitfalls

- **Governance without value ("process theater")** — councils that meet, policies that are filed, and nothing that changes; the anti-pattern parallel to methodology theater in [project_management_methodologies_guide.md](project_management_methodologies_guide.md) §27. Symptom: the program cannot name what it has *changed*.
- **Governance as an IT project** — no business owners, only custodians; the classic failure. Governance without accountable business owners is a documentation exercise.
- **Tool-first** — buying Collibra/DataHub and expecting governance to appear; tools amplify a process that exists and accelerate the failure of one that does not.
- **Over-governance** — bureaucracy that kills data democracy: every access request takes weeks, every field needs a policy, and the business routes around the process (shadow data returns, worse than before).
- **Under-governance** — no teeth: policies with no enforcement, owners who never answer, quality issues that never escalate. Chaos with a glossary.

The balance is deliberate: govern what is *material* (regulatory data, customer data, risk data, finance data) tightly, and leave the long tail lightly governed but discoverable.

### 11.6 The 12–24 Month Journey at a Glance

```
Month    1   3   6   9   12  15  18  21  24
         │   │   │   │   │   │   │   │   │
Phase 1  ▓▓▓▒▒▒
Foundations (roles, policies, pilot)
Phase 2      ▒▒▒▓▓▓▒▒▒
Catalog & lineage (critical paths)
Phase 3          ▒▒▒▓▓▓▓▓▒▒▒
Data quality (rules, issues, SLAs)
Phase 4              ▒▒▒▓▓▓▓▓▒▒▒
MDM/RDM (customer, reference data)
Phase 5                  ▒▒▒▒▒▒▒▒▒▓▓▓
Continuous governance (attestation, automation, BAU)
         │   │   │   │   │   │   │   │   │
Gates:   ▲   ▲       ▲       ▲           ▲
      charter pilot  catalog quality    attestation
      signed done    live     SLAs      round 1
```

▓ = active, ▒ = ramping/overlap. The gates are the governance checkpoints: each gate requires the previous phase's KPIs to be visibly moving before the next phase's spend is approved — the mechanism that keeps a governance program from becoming a 3-year tooling project with no decisions.

### 11.7 Embedding Governance in Delivery

Governance becomes durable when it is embedded in how projects are delivered, not bolted on afterward. The standard instrument is the **data impact assessment (DIA)** — a mandatory step in the project lifecycle for anything that creates, changes, or consumes data: What data does this touch? Who owns it? Does it use glossary terms or invent new ones? Does it change a source that feeds regulatory reporting (lineage impact)? What quality rules apply? What classification and retention apply? The DIA is the practical embodiment of "govern the metadata first" (§7.5): it forces projects to identify themselves against the governance framework before they build, and it feeds the catalog, lineage, and access requests automatically. Programs that adopt DIAs find governance issues are caught in design review (cheap) instead of audit (expensive) — the same economics that drive schema contracts (§9.2) and quality-as-code (§6.3).

## 12. Worked Example: BCBS 239 Risk Data Program

A mid-size wholesale bank (say, a Singapore-based regional bank with a modest trading book) responds to a supervisory finding on BCBS 239 P1/P2/P6. This is how a realistic program is structured — the artifacts are the ones practitioners actually produce.

### 12.1 Program Structure

- **Governance council** — chaired by the CDO; CRO, CFO, COO, CISO; quarterly; approves the Risk Data Policy and the data-quality tolerance framework.
- **Risk data stewards** — business stewards per risk domain (Credit, Market, Liquidity, Counterparty) plus technical stewards from the risk platform team; monthly domain forums.
- **DGO** — program management, KPI reporting, tool administration, issue escalation.
- **Owners** — the Head of Credit Risk owns credit risk data; the Head of Market Risk owns market risk data; accountability sits in the first line, not in IT.

### 12.2 Risk Data Domains and Quality Metrics

Domains: **Credit (exposures, ECL inputs), Market (positions, curves, prices), Liquidity (cash flows, collateral), Counterparty (netting sets, CVA inputs)**. The risk data quality dashboard (monthly, to the council) shows:

| Metric | Definition | Target |
|--------|-----------|--------|
| Accuracy | % of key exposure fields matching source of truth (reconciliation rate) | ≥ 99.5% |
| Completeness | % of required records/fields present at aggregation time | 100% for regulatory fields |
| Timeliness | % of reports available by T+1 08:00 | ≥ 98% |
| Lineage coverage | % of regulatory report line items with end-to-end lineage | 100% for material reports |
| Issue aging | Open quality issues by severity and age | 0 critical > 48h |

### 12.3 Lineage and Evidence

The program builds **end-to-end lineage from regulatory reports to source systems**: each COREP/risk report line item is traced report → aggregation layer → warehouse → source system (front office, collateral, payments), with column-level mappings and transformation logic documented in the catalog. Impact analysis on source changes becomes a routine step in change management (see [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md) and [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) §9.2). The lineage map plus the quality dashboard *is* the BCBS 239 evidence pack supervisors ask to see.

### 12.4 Policies

The **Risk Data Policy** (one page, council-approved): risk data is owned by the risk divisions; quality standards per BCBS 239 dimensions with the tolerance framework; issue escalation path (steward → owner → council) with SLAs; change management for risk data definitions requires council sign-off; attestation by owners twice a year.

### 12.5 Example Artifacts

- **Governance charter** — purpose, scope (risk data domains), roles and decision rights, forum cadence, KPI framework, escalation path; signed by the CDO and CRO.
- **RACI** — the §4.3 quality-issue RACI, instantiated for the risk domain (owner = Head of Credit Risk; steward = Credit Data Steward; DGO = program office).
- **Quality scorecard** — the §12.2 dashboard per domain, with trend lines and remediation owners.
- **Catalog entry** — e.g., for the `CREDIT_EXPOSURE` table:

| Field | Value |
|-------|-------|
| Asset | `risk_credit.exposure_summary` (certified) |
| Glossary terms | "Credit Exposure" (BCBS 239 definition, owner: Head of Credit Risk) |
| Classification | Restricted (regulatory risk data) |
| Lineage | Front office deals → trade capture → ECL engine → exposure_summary → COREP C.07 |
| Quality rules | Accuracy ≥ 99.5%; completeness 100% on regulatory fields; freshness T+1 |
| Consumers | COREP reporting, credit risk dashboard, stress testing |
| Access | Owner-approved; RLS by legal entity; masking: none (non-PII) |

The program's first-year result (realistic): P1 and P2 ratings improved, the June COREP submission passed MAS validation with zero material breaks, and the council's standing agenda shifted from "why is data bad" to "which data products to certify next" — which is exactly the trajectory governance programs are supposed to follow.

### 12.6 Lessons from the Example

Three transferable lessons from the worked program. **First, regulatory evidence is a byproduct, not a deliverable** — the program succeeded because lineage and quality were built as operating machinery, so the evidence pack assembled itself; banks that build "evidence packs" as projects are always behind. **Second, the domain structure is the program structure** — the risk domains (credit, market, liquidity, counterparty) each got an owner + steward + scorecard, and the council governed the *interfaces* between them; this is the hybrid operating model (§4.2) instantiated. **Third, the first-year metric that mattered was the COREP submission** — one clean regulatory return convinced the business more than any KPI deck. Governance programs should always find their "first clean COREP" — the visible, external proof that the machinery works.

## 13. The Future: 2026 and Beyond

Governance is being reshaped by AI — both as the thing being governed and as the tool doing the governing. The 2026+ agenda:

### 13.1 AI-Era Data Governance

- **EU AI Act data governance requirements** — Regulation (EU) 2024/1689 (in force 1 August 2024, phased application through 2025–2027) makes training-data governance a *legal* requirement for high-risk AI systems: **Article 10** mandates that training, validation, and test data sets be relevant, representative, free of errors and bias to the extent reasonably possible, and appropriately documented (provenance, purpose, governance practices). This turns "AI-ready data" from a best practice into a compliance obligation for EU-facing systems — and the same expectations are migrating into banking supervision globally.
- **AI data governance frameworks** — the governance field is extending its vocabulary to model/data/AI assets: registries of AI use cases, AI risk assessment, model cards, and data-sheet-for-datasets documentation practices; NIST AI RMF is the US anchor, the EU AI Act the regulatory anchor (see §9.5 and the `ai_llm/` guide series).
- **Governance for foundation models** — the training-data provenance and licensing questions for LLMs (what was scraped, from where, under what license, with what PII) are forcing new governance artifacts — corpus manifests, license audits, and bias evals as governed assets.

### 13.2 Data Contracts Adoption

Data contracts (§9.2) are moving from practice to standard: schema registries, contract versioning, and quality SLAs negotiated at the contract boundary are becoming the default way governed data moves between teams — especially as mesh-style domain products proliferate. Expect contract tooling (registry + validation + monitoring) to merge into the catalog layer.

### 13.3 Automated Governance

Governance labor is increasingly automated: **AI-driven metadata** (auto-profiling, auto-classification of sensitive data, auto-tagging), **auto-lineage** (continuous SQL/job parsing), and **anomaly-driven quality** (observability tools, §6.3). The direction of travel: humans write the *policies and rules*, machines do the *labeling, tracing, and monitoring* — with humans auditing the machines. Governance agents (LLM-driven assistants that answer "who owns this?" or "what breaks if I change this?") are the frontier.

### 13.4 Governance as Code

**Policy-as-code** encodes governance rules in versioned, testable artifacts: data access policies as code (Open Policy Agent — OPA — is the reference implementation for fine-grained, attribute-based authorization), quality rules as code (§6.3), contracts as code (§9.2). The payoff is *federated computational governance* (§4.5): policy enforced by the platform at the point of access, with the audit trail as a side effect — no human check needed, no shadow process possible.

### 13.5 Zero-Trust Data

Access governance is converging on **zero-trust data access**: never trust location, always verify identity and context — ABAC over RBAC, per-query policy evaluation, continuous verification, and least-privilege by default (§9.4). Combined with masking, tokenization, and clean rooms, this is the access model regulators and CISOs both accept.

### 13.6 Trends Summary

| Trend | What it means for governance |
|-------|------------------------------|
| AI governance becomes regulated | Training-data governance is a legal requirement (EU AI Act); AI registries become governance artifacts |
| Data products proliferate | Governance standardizes the *product* layer (contracts, SLAs, certification) |
| Automation eats the metadata work | Machines label, trace, and monitor; humans govern the rules and audit the machines |
| Policy-as-code | Enforcement moves into the platform; audit trails become side effects |
| Zero-trust data access | ABAC, per-query policy, continuous verification replace static roles |
| Convergence of regulatory drivers | Privacy, resilience, AI, and prudential rules demand the same core capabilities — ownership, classification, quality, lineage, access |

## 14. Glossary

| Term | Definition |
|------|-----------|
| **Data governance** | The exercise of authority and control (planning, monitoring, enforcement) over the management of data assets (DAMA-DMBOK). |
| **Data management** | The operational execution of governing, stewarding, and engineering data: processes, practices, and day-to-day operations. |
| **Data engineering** | The technology layer: building and running pipelines, platforms, and tools that move, store, and transform data. |
| **CDO** | Chief Data Officer — the executive accountable for enterprise data strategy, governance, and value. |
| **DGO** | Data Governance Office — the central team that runs the governance operating model. |
| **Data steward** | The role that operates governance day to day for a domain (business steward: definitions/quality; technical steward: implementation). |
| **Data owner** | A senior business leader accountable for a data domain's quality and use. |
| **Data custodian** | The IT role that implements and operates technical controls over data (custody, not ownership). |
| **DAMA-DMBOK** | The Data Management Body of Knowledge — the industry-standard body of data management knowledge (11 knowledge areas). |
| **DAMA wheel** | DAMA's visual map of the 11 knowledge areas with data governance at the hub. |
| **DGI** | The Data Governance Institute — its framework defines 10 universal components of a governance program. |
| **CMMI DMM** | CMMI Institute's Data Management Maturity model — 5 maturity levels (Initial → Managed → Defined → Measured → Optimized). |
| **DCAM** | The EDM Council's Data Management Capability Assessment Model — the banking-standard maturity/capability framework (8 components). |
| **COBIT** | ISACA's IT governance and management framework. |
| **BCBS 239** | Basel Committee principles for effective risk data aggregation and risk reporting (2013, 14 principles). |
| **MAS TRM** | MAS Technology Risk Management framework — now the FSMA-era Notices FSM-N05 (TRM) and FSM-N06 (Cyber Hygiene). |
| **PDPA** | Singapore's Personal Data Protection Act (enforced by the PDPC). |
| **Data quality** | The degree to which data meets requirements — measured across dimensions (accuracy, completeness, consistency, timeliness, validity, uniqueness, integrity). |
| **Data dimension** | A measurable characteristic of data quality (e.g., accuracy, completeness). |
| **Profiling** | Analyzing data to discover its actual structure, content, and quality (the first step of quality assessment). |
| **Metadata** | Data about data: technical (schema), business (meaning), and operational (how it runs). |
| **Business glossary** | The governed vocabulary of business terms with canonical definitions and owners. |
| **Data catalog** | The metadata repository with discovery, search, tagging, certification, and access-request functions. |
| **Data lineage** | The trace of data from source through transformations to consumption. |
| **Impact analysis** | Using lineage to determine what a change to a source or job would affect downstream. |
| **MDM** | Master Data Management — creating and maintaining golden records for core business entities. |
| **CDM** | Customer Data Management — the customer master / customer 360 capability (also: canonical data model in other contexts). |
| **Golden record** | The single agreed best-quality version of a master data entity. |
| **Entity resolution** | Matching, merging, and deduplicating records that refer to the same real-world entity. |
| **Reference data** | The controlled vocabulary of codes and values (currencies, countries, product codes). |
| **LEI** | Legal Entity Identifier (ISO 17442) — the regulatory identifier for legal entities in transactions. |
| **Data contract** | A versioned producer–consumer agreement: schema, semantics, quality SLAs, freshness. |
| **Data product** | A governed, documented, productized dataset with an owner, SLAs, and discoverability. |
| **Data mesh** | A decentralized architecture: domain ownership, data as a product, self-serve platform, federated computational governance. |
| **RACI** | Responsible / Accountable / Consulted / Informed — the decision-rights matrix. |
| **RBAC / ABAC** | Role-based / attribute-based access control. |
| **RLS** | Row-level security — data-level restrictions on which rows a viewer can see. |
| **Masking / tokenization** | Hiding or replacing sensitive values (dynamic masking at query time; tokenization with reversible tokens). |
| **Data residency** | The constraint that data must remain within a specified jurisdiction. |
| **Clean room** | A technology for joint analytics without exposing raw data to the counterparty. |
| **SR 11-7** | US supervisory guidance on model risk management (model inventory, validation, governance). |
| **EU AI Act** | Regulation (EU) 2024/1689 — the EU's AI regulation; Article 10 governs high-risk training data. |
| **Policy-as-code** | Encoding governance rules in versioned, machine-enforced artifacts (e.g., OPA). |
| **OPA** | Open Policy Agent — the open-source policy engine for attribute-based authorization. |

## 15. References

**Sibling guides in this repository** (cross-referenced above):

- [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — BCBS 239 (§2.2, §9.2), MAS, SR 11-7/model risk (§12.4)
- [data_model_resource_book_guide.md](data_model_resource_book_guide.md) — universal data models, the entity library
- [data_models_banking_insurance_guide.md](../banking/data_models_banking_insurance_guide.md) — banking/insurance data models, customer 360, regulatory data models
- [databricks_guide.md](databricks_guide.md) — lakehouse, Unity Catalog (§7)
- [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) — lakehouse architecture, retention and resilience
- [feature_store_guide.md](feature_store_guide.md) — feature store governance (§11), banking (§12)
- [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) — integration patterns, data quality in integration (§7)
- [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md) — schema evolution, data contracts (§10), BCBS 239 (§13)
- [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) — NoSQL modeling, banking context (§10)
- [chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md) — China data localization (§7)
- [nasdaq_calypso_guide.md](../banking/nasdaq_calypso_guide.md) — market reference data in trading platforms
- [project_management_methodologies_guide.md](project_management_methodologies_guide.md) — methodology anti-patterns (§27)
- `ai_llm/` series — training data governance, embeddings, AI governance

**Primary sources and standards:**

- DAMA International. *DAMA-DMBOK: Data Management Body of Knowledge*, 2nd ed., 2017.
- Basel Committee on Banking Supervision. *Principles for Effective Risk Data Aggregation and Risk Reporting (BCBS 239)*, January 2013; *Progress in adopting the Principles* (report, November 2023 — d559).
- EDM Council. *DCAM — Data Management Capability Assessment Model*, v2 (2020); v3 announced.
- Data Governance Institute. *The DGI Data Governance Framework* (10 universal components).
- CMMI Institute. *Data Management Maturity (DMM) Model*.
- ISACA. *COBIT 2019*.
- ISO/IEC 38505-1:2017. *Governance of IT — Part 1: Governance of data*; ISO 8000 series (data quality).
- Monetary Authority of Singapore. Notices FSM-N05 (Technology Risk Management), FSM-N06 (Cyber Hygiene), Notice 658/1121 (outsourcing); *Technology Risk Management Guidelines* (2013, superseded).
- Regulation (EU) 2022/2554 (DORA); Regulation (EU) 2024/1689 (EU AI Act).
- Federal Reserve / OCC. *SR 11-7: Model Risk Management* (2011).

*Product, regulation, and vendor facts as of August 2026; verify against official sources (DAMA, BIS, EDM Council, MAS, ISACA) before procurement or architecture decisions.*

---

*End of guide.*
