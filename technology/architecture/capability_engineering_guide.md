# Capability Engineering: A Comprehensive Guide

> **Audience:** Enterprise Architects, Solution Architects, Business Architects, Technology Leaders  
> **Author context:** Jack Liu Shurui, Solution Architect at Crédit Agricole CIB  
> **Domain focus:** Financial services / banking (primary), cross-industry patterns (secondary)  
> **Reference frameworks:** BIAN, TOGAF, BIZBOK, CMMI, DAMA-DMBOK, SFIA

---

## Table of Contents

1. [What Is Capability Engineering?](#1-what-is-capability-engineering)
2. [Core Concepts](#2-core-concepts)
3. [Capability Modeling Frameworks](#3-capability-modeling-frameworks)
4. [Capability Lifecycle](#4-capability-lifecycle)
5. [Capability Decomposition](#5-capability-decomposition)
6. [Capability Heat Mapping](#6-capability-heat-mapping)
7. [Capability-Based Planning](#7-capability-based-planning)
8. [Capability vs Process vs Function Matrices](#8-capability-vs-process-vs-function-matrices)
9. [Capability in Enterprise Architecture](#9-capability-in-enterprise-architecture)
10. [Capability Engineering in Banking](#10-capability-engineering-in-banking)
11. [Tools and Platforms](#11-tools-and-platforms)
12. [Capability Maturity Models](#12-capability-maturity-models)
13. [Capability Metrics and KPIs](#13-capability-metrics-and-kpis)
14. [Common Pitfalls](#14-common-pitfalls)
15. [Best Practices](#15-best-practices)
16. [Appendix A: Banking Capability Decomposition Example](#16-appendix-a-banking-capability-decomposition-example)
17. [Appendix B: Capability Heat Map Template](#17-appendix-b-capability-heat-map-template)
18. [References and Further Reading](#18-references-and-further-reading)

---

## 1. What Is Capability Engineering?

Capability Engineering is the **systematic discipline of defining, designing, modeling, analyzing, and evolving organizational capabilities** to align business strategy with execution. It answers the question: *"What must the organization be able to do — today and in the future — to execute its strategy?"*

### 1.1 Definition

A **capability** is the ability of an organization to achieve a specific outcome through the orchestration of people, processes, technology, and information. Capabilities are *stable abstractions* that outlive the specific processes, systems, or organizational structures used to deliver them.

### 1.2 Capability Engineering vs Enterprise Architecture

| Dimension | Capability Engineering | Enterprise Architecture |
|---|---|---|
| **Primary lens** | *What* the organization can do | *How* the organization is structured |
| **Unit of analysis** | Business/technical capabilities | Architecture domains (business, data, app, tech) |
| **Focus** | Outcome, ability, strategic alignment | Structure, standardization, interoperability |
| **Time horizon** | Strategic (3-5+ year capability evolution) | Tactical to strategic (1-5 year roadmaps) |
| **Governance** | Capability investment governance | Architecture review board |
| **Outputs** | Capability maps, heat maps, roadmaps | Architecture principles, reference architectures, blueprints |

Capability Engineering is **not** a replacement for EA — it is a complementary lens that connects strategy to the EA layers. TOGAF's Architecture Capability (Phase A/B) explicitly calls for capability-based planning as the bridge between enterprise strategy and architecture work.

### 1.3 Why Capability Engineering Matters

- **Strategy-to-execution bridge:** Capabilities are the units through which strategy is translated into investment decisions.
- **Business-IT alignment:** Both business and technology stakeholders reason about the same capability model.
- **Change impact analysis:** When a capability needs to change, the model reveals what processes, systems, data, and people are affected.
- **M&A integration:** Merging capability maps from two organizations reveals overlaps, gaps, and synergies faster than comparing org charts or application portfolios.
- **Regulatory compliance:** Regulators increasingly expect banks to demonstrate capability-level controls (e.g., BCBS 239 data capabilities, MAS risk capabilities).
- **Digital transformation:** Transformation becomes a portfolio of capability-building initiatives rather than a vague "modernization program."

---

## 2. Core Concepts

### 2.1 Capability vs Process vs Function vs Service

These terms are frequently conflated. Clear definitions are essential.

| Concept | Definition | Example (Banking) | Scope | Stability |
|---|---|---|---|---|
| **Capability** | *What* an organization can do — a stable ability to produce an outcome | "Onboard Customer" | Strategic / stable | Long-lived (3-10+ years) |
| **Process** | *How* work flows — the sequence of activities that produces a result | "Customer Onboarding Process" (steps from application to account activation) | Operational / temporal | Medium (1-5 years) |
| **Function** | *Who* does the work — an organizational unit or role specialization | "KYC Operations Team" | Organizational | Variable (restructuring changes this) |
| **Service** | *What interface* is exposed — a contract for consuming capability output | "KYC Verification API" / "Account Opening Portal" | Technical / boundary | Medium (2-5 years) |

**Relationships:**
- A **capability** is delivered by one or more **processes**.
- A **process** is executed by one or more **functions** (roles/teams).
- A **service** exposes a **capability** through a contractual interface.
- Capabilities are *stable* — processes and services may change without changing the underlying capability.
- A single capability may span multiple functions, processes, and services.

### 2.2 Business Capability Modeling

Business Capability Modeling is the practice of creating a **capability map** — a hierarchical decomposition of everything an organization does to achieve its mission.

**Key principles:**
- Capabilities are **noun-based** ("Credit Assessment"), not verb-based ("Assess Credit") — though both forms appear in practice.
- Capabilities are **mutually exclusive** at each level (a capability belongs to exactly one parent).
- Capabilities are **collectively exhaustive** across the business domain.
- Capabilities are **stable** — they change slowly, even as processes and technology evolve.
- Capabilities describe **what**, not **how well** (maturity is assessed separately).

### 2.3 Capability Maturity Assessment

Maturity assessment answers: *How well does the organization currently perform this capability?*

- Each capability is scored against a maturity model (see §12).
- Assessments are conducted through interviews, evidence review, and benchmarking.
- Results feed heat maps and investment prioritization.

### 2.4 Capability Heat Mapping

A visual representation of capability health across multiple dimensions (see §6).

### 2.5 Capability-Based Planning

An investment planning approach where capabilities — not projects or systems — are the unit of investment (see §7).

---

## 3. Capability Modeling Frameworks

Several established frameworks provide capability models, taxonomies, and methodologies. The choice depends on industry, scope, and organizational maturity.

### 3.1 BIAN Service Landscape (Banking)

The **Banking Industry Architecture Network (BIAN)** Service Landscape is the de-facto standard for banking capability modeling.

**Key facts:**
- **400+ service domains** organized across **10 business areas**
- Each service domain is a granular business capability (e.g., "Customer Onboarding," "Loan Origination," "Payment Execution")
- Service domains are defined with standard inputs, outputs, and behaviors
- Covers retail banking, corporate banking, wealth management, payments, risk, and operations

**10 BIAN Business Areas:**
1. **Customer** — Relationship management, customer data, marketing
2. **Sales & Service** — Product distribution, channels, service management
3. **Products** — Product definition, manufacturing, management
4. **Operations** — Transaction processing, fulfillment, settlement
5. **Risk** — Credit risk, market risk, operational risk, compliance
6. **Finance** — Accounting, treasury, financial management
7. **Legal & Compliance** — Regulatory reporting, KYC/AML, legal
8. **Technology** — IT infrastructure, data management, security
9. **Channel** — Branch, digital, mobile, call center, ATM
10. **Corporate** — HR, procurement, strategy, facilities

**BIAN service domain example:** `Service Domain: CustomerOnboarding`
- **Purpose:** Manage the process of bringing new customers into the bank
- **Standard behaviors:** Onboard, check status, update, terminate
- **Associated domains:** KYC/AML checks, Document Management, Product Eligibility

### 3.2 TOGAF

The **Open Group Architecture Framework (TOGAF)** integrates capability engineering primarily in:

- **Preliminary Phase:** Defines the "Architecture Capability" — the organization's ability to do architecture itself.
- **Phase A (Architecture Vision):** Business capability mapping establishes the baseline and target states. The Architecture Vision document includes a capability assessment.
- **Phase B (Business Architecture):** Business capabilities are decomposed, mapped to value streams and organization units.
- **Architecture Capability Framework:** A meta-capability — the framework for governing and improving the architecture practice itself.

**TOGAF Capability-Based Planning:**
- Capabilities are the basis for transition architectures (Phase E-F)
- The Capability Increment is the unit of delivery across architecture roadmaps
- Each capability increment bundles architecture work packages into a measurable delivery

### 3.3 CMMI Capability Levels

The **Capability Maturity Model Integration (CMMI)** provides capability levels that can be applied to any business or technical capability:

| Level | Name | Description |
|---|---|---|
| 0 | Incomplete | Work is not performed or does not meet defined goals |
| 1 | Performed | Work is performed but may not be stable or repeatable |
| 2 | Managed | Work is planned, monitored, and controlled |
| 3 | Defined | Work is defined at organizational standards |
| 4 | Quantitatively Managed | Work is measured and controlled statistically |
| 5 | Optimizing | Work is continuously improved through quantitative feedback |

**CMMI for Development (CMMI-DEV)** and **CMMI for Services (CMMI-SVC)** provide capability-level assessments for product development and service delivery capabilities respectively.

### 3.4 BIZBOK (Business Architecture Guild)

The **Business Architecture Body of Knowledge (BIZBOK)** provides structured guidance for business capability modeling:

- **Capability Map Blueprint:** The core deliverable — a Level 1-3 map showing all business capabilities
- **Capability-to-Value Stream Mapping:** Links capabilities to value streams (end-to-end customer journeys)
- **Capability-to-Organization Mapping:** Shows which organizational units own or contribute to each capability
- **Capability-to-Resource Mapping:** Links capabilities to people, process, technology, and information resources

**BIZBOK conventions:**
- Capability maps typically go 3-4 levels deep in initial models
- Heat maps overlay maturity, performance, or strategic importance
- The Business Architecture Guild maintains industry-specific reference models (including banking)

### 3.5 DAMA-DMBOK (Data Capabilities)

The **Data Management Body of Knowledge (DAMA-DMBOK)** provides a data-specific capability framework:

**11 Data Management Knowledge Areas (as capabilities):**
1. Data Governance
2. Data Architecture
3. Data Modeling and Design
4. Data Storage and Operations
5. Data Security
6. Data Integration and Interoperability
7. Document and Content Management
8. Reference and Master Data
9. Data Warehousing and Business Intelligence
10. Metadata Management
11. Data Quality

Each knowledge area includes defined activities, deliverables, roles, and best practices — effectively a capability model for the data domain.

### 3.6 SFIA (Skills Capabilities)

The **Skills Framework for the Information Age (SFIA)** models individual-level skills as capabilities:

- **121 professional skills** across 6 categories (Strategy, Design, Development, Service, People, Governance)
- **7 levels of responsibility** (from Follow to Set strategy)
- Used for workforce planning, skills gap analysis, and individual development planning
- Complements organizational capability models by providing the "people capability" dimension

**SFIA in capability engineering:**
- SFIA skills are the *individual* level of capability
- Organizational capability = orchestration of SFIA-skilled individuals
- Heat maps can overlay SFIA skill availability against capability requirements

---

## 4. Capability Lifecycle

Capabilities follow a defined lifecycle with governance gates at each stage.

### 4.1 Lifecycle Stages

```
 Identify → Define → Model → Assess → Plan → Build → Operate → Evolve
    ↑                                                               │
    └─────────────────────── (continuous loop) ────────────────────┘
```

| Stage | Description | Key Activities | Governance Gate |
|---|---|---|---|
| **1. Identify** | Discover and inventory current and needed capabilities | Stakeholder interviews, strategy review, regulatory scan, competitor analysis | Capability identification approved by architecture board |
| **2. Define** | Specify the capability's name, scope, outcome, and boundaries | Write capability definition, identify parent/child, document stakeholders | Definition reviewed with business owners |
| **3. Model** | Decompose and link capability into hierarchy and relationship map | Build capability map, link to processes, org units, systems, and data | Model validated by domain experts |
| **4. Assess** | Evaluate current maturity, performance, risk, and cost | Conduct maturity assessment, gather metrics, create heat map | Assessment results reviewed with leadership |
| **5. Plan** | Plan investments and roadmap to close gaps or build new capabilities | Capability-based investment planning, roadmap creation, gap analysis | Investment plan approved by steering committee |
| **6. Build** | Execute the work packages that deliver or improve the capability | Project execution, system implementation, process redesign | Delivery milestones reviewed at stage gates |
| **7. Operate** | Run and sustain the capability in production | Operations handover, SLA monitoring, incident management | Operational acceptance sign-off |
| **8. Evolve** | Continuously improve or retire the capability as strategy changes | Periodic reassessment, trigger re-identification, decommission obsolete capabilities | Evolution decisions at planning cycles |

### 4.2 Governance Gates

Each stage entry/exit requires specific artifacts:

- **Gate 1 (Identify → Define):** Capability catalog entry drafted, strategic alignment confirmed
- **Gate 2 (Define → Model):** Capability definition signed off by business owner
- **Gate 3 (Model → Assess):** Capability map reviewed by architecture review board
- **Gate 4 (Assess → Plan):** Assessment results reviewed, investment prioritization criteria set
- **Gate 5 (Plan → Build):** Funding approved, work packages defined
- **Gate 6 (Build → Operate):** Capability delivered, operational readiness confirmed
- **Gate 7 (Operate → Evolve):** Periodic review triggers re-identification or retirement

---

## 5. Capability Decomposition

Capabilities are decomposed into hierarchical levels. The number of levels varies by organizational scope and tooling constraints, but a 5-level model is standard for large enterprises.

### 5.1 Standard Decomposition Levels

| Level | Name | Scope | Example (Banking) |
|---|---|---|---|
| L1 | Strategic Capability | Enterprise-wide strategic domain | Manage Customer Relationships |
| L2 | Core/Business Capability | Major business function | Onboard Customer |
| L3 | Operational Capability | Specific operational ability | KYC/CDD |
| L4 | Functional Capability | Concrete functional activity | Document Verification |
| L5 | Sub-Capability / Feature | Atomic capability unit | Biometric ID Check |

### 5.2 Decomposition Rules

- **Each child fully supports the parent** — the parent's existence depends on its children.
- **Children are mutually exclusive** at the same level — no overlap.
- **A capability at any level can be assessed for maturity**, but assessments are typically done at Level 3-4.
- **Too many levels** (6+) become administratively burdensome. **Too few** (2-3) lack actionable granularity. **5 levels** is the practical sweet spot for large financial institutions.

### 5.3 Banking Capability Decomposition (Full Example)

**Domain: Customer Lifecycle Management**

```
L1: MANAGE CUSTOMER RELATIONSHIPS
├── L2: Acquire Prospects
│   ├── L3: Lead Management
│   │   ├── L4: Lead Capture (from branches, digital, third-party)
│   │   ├── L4: Lead Scoring & Qualification
│   │   └── L4: Lead Distribution & Tracking
│   └── L3: Marketing Campaign Management
│       ├── L4: Campaign Design & Targeting
│       ├── L4: Campaign Execution (multi-channel)
│       └── L4: Campaign Performance Measurement
│
├── L2: Onboard Customer
│   ├── L3: Customer Identification
│   │   ├── L4: Identity Verification
│   │   └── L4: Biometric ID Check (L5)
│   ├── L3: KYC/CDD
│   │   ├── L4: Document Verification
│   │   │   ├── L5: Biometric ID Check
│   │   │   ├── L5: PoA Document Validation
│   │   │   └── L5: Digital Document Forensics
│   │   ├── L4: Beneficial Ownership Identification
│   │   ├── L4: PEP Screening
│   │   ├── L4: Sanctions Screening
│   │   ├── L4: Adverse Media Check
│   │   └── L4: Risk Rating (customer risk profiling)
│   ├── L3: Account Setup
│   │   ├── L4: Account Opening (current/savings/deposit)
│   │   ├── L4: Product Selection & Eligibility
│   │   ├── L4: Credit Initiation (if applicable)
│   │   └── L4: Service Activation (online banking, cards)
│   ├── L3: Onboarding Monitoring
│   │   ├── L4: Onboarding Progress Tracking
│   │   └── L4: Onboarding Quality Assurance
│   └── L3: Regulatory Onboarding Reporting
│       ├── L4: Regulatory Filing (MAS, ACRA, etc.)
│       ├── L4: FATCA/CRS Reporting Setup
│       └── L4: Tax Profile Establishment
│
├── L2: Serve Customer
│   ├── L3: Transaction Processing
│   │   ├── L4: Payment Execution (domestic, cross-border)
│   │   ├── L4: Trade Execution
│   │   ├── L4: Treasury Settlement
│   │   └── L4: Card Transaction Processing
│   ├── L3: Customer Service
│   │   ├── L4: Inquiry Management
│   │   ├── L4: Complaint Resolution
│   │   ├── L4: Dispute Management
│   │   └── L4: Branch & Digital Service
│   ├── L3: Portfolio Management
│   │   ├── L4: Investment Portfolio Reporting
│   │   ├── L4: Rebalancing & Optimization
│   │   └── L4: Performance Attribution
│   └── L3: Advisory Services
│       ├── L4: Financial Planning
│       ├── L4: Investment Advisory
│       └── L4: Wealth Planning
│
├── L2: Manage Risk & Compliance (Customer-Facing)
│   ├── L3: Ongoing KYC Refresh
│   │   ├── L4: Periodic Review
│   │   ├── L4: Event-Driven Review
│   │   └── L4: Trigger-Based Reassessment
│   ├── L3: Transaction Monitoring
│   │   ├── L4: AML Transaction Screening
│   │   ├── L4: Suspicious Activity Detection
│   │   └── L4: STR/SAR Filing
│   ├── L3: Credit Risk Management
│   │   ├── L4: Credit Assessment
│   │   ├── L4: Limit Management
│   │   └── L4: Collateral Management
│   └── L3: Regulatory Reporting
│       ├── L4: MAS Returns Filing
│       ├── L4: BCBS 259 Reporting
│       ├── L4: Capital Adequacy Reporting
│       └── L4: Audit Trail Provision
│
├── L2: Retain & Grow Customer
│   ├── L3: Loyalty & Retention
│   │   ├── L4: Loyalty Program Management
│   │   ├── L4: Churn Prediction & Prevention
│   │   └── L4: Win-Back Campaigns
│   ├── L3: Cross-Sell & Upsell
│   │   ├── L4: Product Recommendation Engine
│   │   ├── L4: Next-Best-Action Orchestration
│   │   └── L4: Offer Management
│   └── L3: Customer Feedback & Voice
│       ├── L4: NPS Survey Management
│       ├── L4: Sentiment Analysis
│       └── L4: Feedback-Driven Improvement
│
└── L2: Manage Customer Data
    ├── L3: Customer Data Governance
    │   ├── L4: Data Quality Management
    │   ├── L4: Data Privacy Compliance (PDPA/GDPR)
    │   └── L4: Data Lineage & Provenance
    ├── L3: Customer Reference Data
    │   ├── L4: Master Customer Record Management
    │   ├── L4: Customer Data Integration (multi-system)
    │   └── L4: Identity Resolution & Matching
    └── L3: Customer Analytics
        ├── L4: Segmentation & Profiling
        ├── L4: Behavior Analytics
        └── L4: Predictive Modeling
```

### 5.4 Cross-Domain Capability Dependencies

Capabilities do not exist in isolation. Dependencies must be explicitly modeled:

```
Onboard Customer
  depends on → Manage Customer Data (for customer reference data)
  depends on → Manage Risk & Compliance (for KYC/CDD outcomes)
  depends on → Manage Identity & Access (for digital identity verification)
  depends on → Manage Documents & Records (for document management)

KYC/CDD
  depends on → Manage Screening Data (for sanctions/PEP lists)
  depends on → Manage Identity Services (for biometric verification)
  depends on → Manage Case Management (for review workflows)
```

Dependency modeling enables impact analysis: *"If we decommission System X (supporting Capability Y), what capabilities are affected?"*

---

## 6. Capability Heat Mapping

A capability heat map is a visual matrix that overlays assessment results onto the capability map. Each capability cell is colored to represent its health across one or more dimensions.

### 6.1 Common Heat Map Dimensions

| Dimension | What It Measures | Data Sources | Typical Scale |
|---|---|---|---|
| **Maturity** | How mature/defined the capability is | Maturity assessment | 1-5 (see §12) |
| **Performance** | How well the capability delivers outcomes | KPIs, SLAs, customer satisfaction | 1-5 / RAG |
| **Cost** | Total cost to operate the capability | TCO analysis, activity-based costing | $ range / RAG |
| **Risk** | Operational, regulatory, or technology risk exposure | Risk assessments, audit findings | RAG (Low/Med/High) |
| **Strategic Importance** | How critical this capability is to strategy | Strategy decomposition, executive input | 1-5 |
| **Regulatory Compliance** | Whether the capability meets regulatory requirements | Compliance assessments, audit results | Pass/Warning/Fail |
| **Technology Health** | Health of underlying systems | Application portfolio assessment | RAG |
| **Talent Availability** | Availability of skilled resources | Workforce planning, SFIA assessments | RAG |
| **Automation Level** | Degree of process automation | Process mining, automation audits | Manual/Assisted/Automated/Autonomous |

### 6.2 Heat Map Example (Banking)

**Capability Heat Map — Customer Onboarding Domain (Q2 2025)**

| L1 | L2 | L3 | Maturity | Performance | Cost | Risk | Compliance | Strategic Importance |
|---|---|---|---|---|---|---|---|---|
| **Manage Customer Relationships** | **Acquire Prospects** | Lead Management | 🟢 4 | 🟢 5 | 🟢 $2M | 🟢 Low | 🟢 Pass | 🟢 High |
| | | Campaign Mgmt | 🟢 4 | 🟢 4 | 🟢 $1.5M | 🟢 Low | 🟢 Pass | 🟡 Medium |
| | **Onboard Customer** | Customer Identification | 🟢 4 | 🟢 4 | 🟢 $1M | 🟡 Medium | 🟢 Pass | 🟢 High |
| | | **KYC/CDD** | **🟡 3** | **🔴 2** | **🔴 $8M** | **🔴 High** | **🟡 Warning** | **🟢 High** |
| | | Account Setup | 🟢 4 | 🟢 4 | 🟡 $3M | 🟢 Low | 🟢 Pass | 🟢 High |
| | | Onboarding Monitoring | 🟡 3 | 🟡 3 | 🟢 $500K | 🟢 Low | 🟢 Pass | 🟡 Medium |
| | | Regulatory Onboarding Reporting | 🟡 3 | 🟡 3 | 🟡 $2M | 🔴 High | 🟡 Warning | 🟢 High |
| | **Serve Customer** | Transaction Processing | 🟢 5 | 🟢 5 | 🟢 $10M | 🟡 Medium | 🟢 Pass | 🟢 High |
| | | Customer Service | 🟢 4 | 🟡 3 | 🟢 $5M | 🟢 Low | 🟢 Pass | 🟢 High |
| | ... | ... | ... | ... | ... | ... | ... | ... |

**RAG Key:** 🟢 Green = Good (on target) | 🟡 Amber = Needs attention (below target) | 🔴 Red = Critical (action required)

**Observations from this heat map:**
- **KYC/CDD** is the top investment priority: low maturity (3), poor performance (2), high cost ($8M), high risk, compliance warnings, and high strategic importance.
- **Regulatory Onboarding Reporting** also requires investment — high risk and compliance warning.
- **Transaction Processing** is mature and well-performing — a candidate for optimization rather than transformation.
- **Lead Management** and **Account Setup** are healthy — maintain current investment levels.

### 6.3 Heat Map Creation Process

1. **Score each capability** against selected dimensions (use RAG or 1-5 numbering)
2. **Weight dimensions** if creating a composite score (e.g., strategic importance × 2, compliance × 3)
3. **Color-code** based on thresholds (Green ≥ 4, Amber 2-3, Red < 2 for 1-5 scales)
4. **Visualize** using the capability hierarchy as the grid
5. **Validate** with business owners and domain experts
6. **Refresh** quarterly or at each planning cycle

### 6.4 Heat Map Template (Markdown / Spreadsheet-Usable)

```markdown
# Capability Heat Map — [Domain Name] — [Period]

## Scoring Legend
| Score | Maturity | Performance | Cost-to-Serve | Risk Exposure | Compliance |
|---|---|---|---|---|---|
| 🟢 4-5 | Defined/Managed/Optimizing | Exceeding targets | Under budget | Low | Full compliance |
| 🟡 2-3 | Repeatable/Ad Hoc | Below targets | On budget | Medium | Partial / warnings |
| 🔴 1-2 | Incomplete / Performed | Well below targets | Over budget | High | Non-compliant |

## Heat Map Grid
| Capability (L1) | Capability (L2) | Capability (L3) | Maturity | Performance | Cost | Risk | Compliance | Strategic Import |
|---|---|---|---|---|---|---|---|---|
| [L1 Name] | [L2 Name] | [L3 Name] | 🟢 4 | 🟢 4 | 🟢 $1M | 🟢 Low | 🟢 Pass | 🟢 High |
| | | [L3 Name 2] | 🟡 3 | 🔴 2 | 🔴 $5M | 🔴 High | 🟡 Warning | 🟢 High |
| | [L2 Name 2] | [L3 Name] | ... | ... | ... | ... | ... | ... |

## Key Insights
- **Critical gaps:** [Capabilities with 🔴 in critical dimensions]
- **Top investment priorities:** [Capabilities needing most urgent action]
- **Strengths to leverage:** [Capabilities with all 🟢 scores]

## Action Items
| Priority | Capability | Issue | Recommended Action | Owner | Target |
|---|---|---|---|---|---|
| P1 | KYC/CDD | Performance, cost, risk | Transform KYC platform | [Name] | Q3 2025 |
| P2 | ... | ... | ... | ... | ... |
```

---

## 7. Capability-Based Planning

Capability-Based Planning (CBP) uses capabilities — not projects, systems, or organizational units — as the primary unit of investment and portfolio management.

### 7.1 Core Principles

1. **Capabilities are the investment unit.** Funding is allocated to improving, building, or retiring capabilities — not to projects or systems.
2. **Strategy decomposes to capabilities.** Strategic goals are translated into required capability improvements.
3. **Roadmaps are capability-based.** The roadmap shows which capabilities will improve by when, not which projects will deliver.
4. **Portfolio management is capability-aligned.** The portfolio is organized by capability domain, enabling cross-silo investment decisions.

### 7.2 Capability-Based Roadmap

```
                      +--------------------------------------------------+
                      |            CAPABILITY ROADMAP (2025-2028)        |
+--------------------+------+------------+------------+------------+------+
|   Capability       | 2025 |    2026    |    2027    |    2028    | Target|
+--------------------+------+------------+------------+------------+-------+
| KYC/CDD            | L2→L4|   (sustain)|   (sustain)|   (sustain)|  L4   |
|   Investment       | $20M |    $2M     |    $1M     |    $1M     |       |
+--------------------+------+------------+------------+------------+-------+
| Customer Identity   | L2→L3|   L3→L4    |   (sustain)|   (sustain)|  L4   |
|   Investment       | $8M  |    $5M     |    $2M     |    $2M     |       |
+--------------------+------+------------+------------+------------+-------+
| Digital Account    | L3→L4|   (sustain)|   (sustain)|   (sustain)|  L4   |
| Opening            |      |            |            |            |       |
|   Investment       | $10M |    $3M     |    $2M     |    $2M     |       |
+--------------------+------+------------+------------+------------+-------+
| Payment Processing | L4→L5|   L4→L5    |   (sustain)|   (sustain)|  L5   |
|   Investment       | $5M  |    $5M     |    $3M     |    $3M     |       |
+--------------------+------+------------+------------+------------+-------+
| Legacy System X    |     Retire       |            |            | Gone  |
|   (Capability       | (merge into     |            |            |       |
|   absorbed)         | Digital Acct    |            |            |       |
+--------------------+------------------+------------+------------+-------+
| TOTAL INVESTMENT   | $43M |    $15M    |    $8M     |    $8M     |       |
+--------------------+------+------------+------------+------------+-------+
```

### 7.3 Linking Strategy to Capabilities

**Example — Strategic Goal: "Achieve 3-day time-to-account for SME clients"**

1. **Strategic goal** → Break down into required capability improvements
2. **Identify capabilities** needed: Onboard Customer, KYC/CDD, Account Setup, Document Management
3. **Assess current maturity** for each:
   - KYC/CDD: Level 2 (Repeatable) — manual checks take 5-7 days
   - Account Setup: Level 3 (Defined) — 2 days
   - Document Management: Level 2 (Repeatable) — manual uploads and verification
4. **Define target maturity**: KYC/CDD → Level 4, Document Management → Level 4
5. **Identify gaps**: KYC automation, digital identity verification, automated document processing
6. **Plan investment**: $15M across 3 initiatives targeting these capabilities
7. **Roadmap**: Q2 2025: KYC automation platform | Q3 2025: Digital ID integration | Q4 2025: Automated document processing
8. **Measure**: Time-to-account reduced from 14 days → 5 days (Q3) → 3 days (Q4)

### 7.4 Capability-Based Investment Prioritization

**Prioritization criteria (weighted scoring):**

| Criterion | Weight | Description |
|---|---|---|
| Strategic Alignment | 30% | How directly does improving this capability serve top strategic objectives? |
| Business Value | 25% | What is the expected ROI, revenue impact, or cost savings? |
| Regulatory Necessity | 20% | Is improvement required by regulation (compliance deadline)? |
| Dependency Impact | 10% | How many other capabilities depend on this one? |
| Feasibility | 10% | Can we realistically improve this capability given constraints? |
| Risk Reduction | 5% | How much does this capability contribute to enterprise risk exposure? |

---

## 8. Capability vs Process vs Function Matrices

Cross-mapping capabilities to other EA artifacts enables traceability and impact analysis.

### 8.1 Capability-to-Process Matrix

```
                          Onboard Customer   KYC/CDD   Account Setup   Payments   ...
Total Customer Journey
├── Marketing Journey    Touch/Optimization  -         -               -
├── Sales Journey        -                   -         -               -
├── Onboarding Journey   ✦ Primary            ✦ Primary  ✦ Primary      -
│   ├── Submit App       Support              -         -               -
│   ├── Verify Identity  -                    ✦ Primary  -               -
│   ├── Credit Assess    -                    Support   Support         -
│   ├── Open Account     -                    -         ✦ Primary       -
│   └── Activate Service -                    -         Support         ✦ Primary
├── Transact Journey     -                   -         -               ✦ Primary
├── Service Journey      Touch               -         -               Support
└── Retain Journey       -                   -         -               -
```

**Legend:** ✦ Primary (capability is primary enabler) | Support (capability supports but is not primary) | Touch (capability is involved at specific touchpoint) | - (no relationship)

### 8.2 Capability-to-Application Matrix

```
Application \ Capability    Onboard Customer   KYC/CDD   Account Setup   Payments
Salesforce CRM              ✦ Primary           Support    Support         -
KYC Platform (Pega)         -                   ✦ Primary  -               -
Core Banking System (T24)   Touch               -         ✦ Primary       ✦ Primary
Payment Hub (A498)          -                   -         -               ✦ Primary
Document Mgmt (Alfresco)    Support             ✦ Primary  Support         -
Identity Platform (ForgeRock) ✦ Primary          ✦ Primary  ✦ Primary       -
Data Warehouse (Teradata)   Support             Support   Support         Support
```

**Impact analysis:** If the KYC Platform is decommissioned, the KYC/CDD capability is directly impacted (✦ Primary), with secondary effects on Onboard Customer (via dependency).

### 8.3 Capability-to-Data Matrix

```
Data Entity \ Capability    Onboard Customer   KYC/CDD   Account Setup   Payments
Customer Master             ✦ Create/Update     ✦ Read    ✦ Read          ✦ Read
KYC Documentation           -                   ✦ CRUD    -               -
Account Record              -                   Touch     ✦ Create/Update  ✦ Read
Transaction Record          -                   ✦ Read    -               ✦ CRUD
Product Catalog             ✦ Read              -         ✦ Read          -
Risk Profile                Touch               ✦ CRUD    Touch           -
```

**Legend:** CRUD = Full create/read/update/delete | Read-only | Touch = occasional access

### 8.4 Multi-Dimensional Traceability Matrix

A consolidated view linking capabilities, processes, applications, data, and organization:

```
Capability  │ Process(es)    │ Application(s)   │ Data Entity(ies)        │ Owner Org
────────────┼────────────────┼──────────────────┼─────────────────────────┼─────────────
KYC/CDD     │ Verify Identity│ KYC Platform     │ KYC Documentation       │ Compliance
            │ Assess Risk    │ Sanctions System │ Risk Profile            │ KYC Ops
            │ Screen PEPs    │ WorldCheck       │ PEP List / Sanctions    │ FCC
            │ Ongoing Review │ CRM (Salesforce) │ Customer Master         │
────────────┼────────────────┼──────────────────┼─────────────────────────┼─────────────
Account     │ Open Account   │ Core Banking     │ Account Record          │ Retail Ops
Setup       │ Select Product │ Product Catalog  │ Product Catalog         │ Digital
            │ Activate Srvc  │ Channel Platform │ Service Subscriptions   │ Channels
```

---

## 9. Capability in Enterprise Architecture

Capability models are the connective tissue across the four TOGAF architecture domains.

### 9.1 TOGAF ADM and Capability Engineering

```
┌─────────────────────────────────────────────────────────────┐
│                 TOGAF ADM with Capabilities                  │
├─────────────────────────────────────────────────────────────┤
│ Preliminary Phase: Define Architecture Capability           │
│ Phase A: Business Capability Mapping (baseline & target)    │
│ Phase B: Capability → Business Architecture mapping         │
│ Phase C: Capability → Data Architecture mapping              │
│ Phase C: Capability → Application Architecture mapping       │
│ Phase D: Capability → Technology Architecture mapping        │
│ Phase E: Capability-based transition architecture            │
│ Phase F: Capability-based roadmap & migration planning       │
│ Phase G: Capability governance & implementation oversight    │
│ Phase H: Capability evolution & change management            │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 Capability-to-Business Architecture

- **Value streams** are mapped to capabilities (a value stream step invokes one or more capabilities)
- **Organization units** are mapped to capabilities (who owns or contributes)
- **Business processes** realize capabilities (how the capability is executed)
- **Business information** is consumed and produced by capabilities (what data the capability uses)

### 9.3 Capability-to-Data Architecture

- Capabilities define what data must exist
- Data entities are classified by which capabilities create, read, update, or delete them
- Data governance is organized by capability domain (e.g., "Customer Data Governance" supports the "Manage Customer Data" capability)
- Critical data elements are traceable to the capabilities that depend on them

### 9.4 Capability-to-Application Architecture

- Applications are mapped to the capabilities they enable
- Application portfolios are rationalized by capability: "Which applications support the KYC/CDD capability?"
- Redundancy is exposed: "Three applications claiming to support the same capability — which is the system of record?"
- Gaps are identified: "No application explicitly supports the Biometric ID Check capability — we need one."

### 9.5 Capability-to-Technology Architecture

- Technology platforms are mapped to the capabilities they underpin
- Infrastructure consolidation decisions consider capability criticality
- Cloud migration priority is derived from capability strategic importance
- Technology debt is assessed per capability: "The KYC/CDD capability runs on 15-year-old infrastructure"

### 9.6 Integration Example: Impact Analysis

**Question:** *What is the impact of migrating the KYC platform from on-premise to cloud?*

Using the capability model, the architect traces:

1. **Capability affected:** KYC/CDD (L3), Document Verification (L4), Customer Onboarding (L2)
2. **Processes affected:** Verify Identity, Assess Risk, Screen PEPs, Ongoing Review
3. **Applications affected:** KYC Platform (primary), WorldCheck (integrated), CRM (receives data from KYC)
4. **Data affected:** KYC Documentation, Risk Profile, Customer Master (updates from KYC)
5. **Organization affected:** KYC Ops, Compliance, FCC (Financial Crime Compliance)
6. **Dependencies:** Identity Platform (provides authentication data), Document Mgmt (stores KYC docs)

This comprehensive view enables the architect to plan migration in waves, ensuring no capability gap during transition.

---

## 10. Capability Engineering in Banking

Banking has the most mature capability engineering discipline outside defense/government, driven by regulatory complexity, M&A activity, and digital transformation pressure.

### 10.1 Financial Services Capability Model (FS-PCA)

The **Financial Services Process Classification Agreement (FS-PCA)** originated from BIAN in collaboration with APQC and industry consortia.

**Key characteristics:**
- Industry-standard capability model for retail and corporate banking
- Aligned with BIAN service domain taxonomy
- Used by major banks worldwide for benchmarking and transformation
- Covers 13 process/capability categories across 400+ activities

**FS-PCA top-level categories:**
1. Develop Vision and Strategy
2. Develop and Manage Products and Services
3. Market and Sell Products and Services
4. Deliver Customer Service
5. Manage Customer Accounts
6. Process Payments and Transactions
7. Manage Financial Resources
8. Manage Risk and Compliance
9. Manage Information Technology
10. Manage Human Resources
11. Manage Legal and Regulatory
12. Manage Enterprise Resources
13. Manage External Relationships

### 10.2 Regulatory Compliance Capabilities

Banking regulation requires specific capabilities, often with formal assessment requirements:

| Regulation | Required Capability | Key Requirements |
|---|---|---|
| **BCBS 239** (Risk Data Aggregation) | Risk Data Management, Risk Reporting | 11 principles for data accuracy, completeness, timeliness; audit trails; data lineage |
| **MAS 644 / 749** | Technology Risk Management | Cybersecurity capabilities, business continuity, incident response |
| **MAS 626** | AML/CFT | KYC/CDD capabilities, transaction monitoring, screening, suspicious activity reporting |
| **GDPR / PDPA** | Data Privacy | Data inventory, consent management, data subject access request, breach notification |
| **FATCA / CRS** | Tax Reporting | Customer tax classification, automated reporting to tax authorities |
| **SFA / FAA (Singapore)** | Compliance Monitoring | Trade surveillance, market abuse detection, record-keeping |
| **BCBS 258 / 259** | Capital Management | Capital adequacy assessment, stress testing, ICAAP/ILAAP |

**Regulatory capability mapping approach:**
1. Identify all regulations applicable to the bank
2. Decompose each regulation into required capabilities
3. Map existing bank capabilities to regulatory requirements
4. Identify gaps (capabilities that don't meet regulatory standards)
5. Assign risk ratings based on gap severity

### 10.3 M&A Integration — Merging Capability Maps

When two banks merge, capability models enable structured integration planning:

**Step 1: Map both banks' capabilities to a common model (e.g., BIAN)**
```
Bank A                          Bank B                          BIAN Reference
┌──────────────────────┐       ┌──────────────────────┐        ┌──────────────────────┐
│ Customer Onboarding  │───┐   │ New Client Setup     │───┐    │ CustomerOnboarding   │
│ (Level 2, mature L3) │   │   │ (Level 2, mature L4) │   │    │ Service Domain       │
├──────────────────────┤   ├───┼──────────────────────┤   ├────┼──────────────────────┤
│ KYC Checks (Level 3) │   │   │ AML/KYC (Level 3)    │   │    │ KYC/AML Checks       │
│   - Manual process   │   │   │   - Automated        │   │    │                      │
│   - 5-day cycle      │   │   │   - 1-day cycle      │   │    │                      │
└──────────────────────┘   │   └──────────────────────┘   │    └──────────────────────┘
                           └───────────────────────────────┘
```

**Step 2: Identify overlaps, gaps, and best-of-breed**
- **Overlap:** Both banks have onboarding capabilities — choose Bank B's as the target (more mature)
- **Gap:** Bank A has a Trade Finance capability not present in Bank B — must build or retain
- **Best-of-breed:** Bank A's Risk Management capability is stronger — adopt as the target

**Step 3: Plan integration roadmap**
- Target capability map = merged best-of-breed capabilities
- Transition plan = 18-24 months with defined capability increments
- Each increment delivers a set of capabilities at target maturity on the target platform

**Step 4: Execute and measure**
- Track integration by capability, not by system or department
- Measure: capability maturity convergence, cost synergies per capability, regulatory compliance per capability

### 10.4 Digital Transformation — Build vs Retire vs Inhabit

Digital transformation involves three capability strategies:

| Strategy | When to Use | Banking Example |
|---|---|---|
| **Build new capability** | Strategic gap, no viable external solution | Build a real-time KYC verification capability using biometrics and AI (not available as package) |
| **Buy / partner for capability** | Standard capability available externally | Buy a cloud-based sanctions screening service (WorldCheck, LexisNexis) |
| **Transform existing capability** | Existing capability is strategically important but underperforming | Transform account opening from branch-led to fully digital with eKYC |
| **Retire capability** | Capability no longer strategic, being phased out | Retire legacy cheque processing capability as digital payments replace cheques |
| **Consolidate capability** | Duplicate capabilities across business units | Consolidate 3 separate KYC capabilities (retail, corporate, wealth) into a single enterprise KYC capability |

**Digital maturity progression:**
```
Paper-based Manual    →  Digitized (document-centric)   →  Automated (workflow)
Automated (workflow)  →  Intelligent (AI/ML-enhanced)   →  Autonomous (self-optimizing)
```

**Banking digital capability example — Account Opening:**
- **Level 1 (Paper):** Customer walks into branch, fills physical forms, officer verifies physically
- **Level 2 (Digitized):** Customer fills PDF, emails to RM, documents scanned into DMS
- **Level 3 (Automated):** Web portal with form filling, OCR document upload, automated data extraction
- **Level 4 (Intelligent):** AI-powered identity verification, liveness detection, predictive risk scoring
- **Level 5 (Autonomous):** Account fully opened in <60 seconds with zero human touch, auto-funded

### 10.5 Banking-Specific Capability Examples

**Corporate Banking Capabilities:**
- Trade Finance Capabilities (Letter of Credit, Guarantees, Supply Chain Finance)
- Cash Management Capabilities (Pooling, Sweeping, Virtual Accounts)
- Loan Syndication Capabilities (Origination, Underwriting, Distribution)
- FX & Treasury Capabilities (Currency trading, Hedging, Derivatives)

**Wealth Management Capabilities:**
- Portfolio Management Capabilities (Asset allocation, Rebalancing, Performance reporting)
- Advisory Capabilities (Financial planning, Estate planning, Tax optimization)
- Discretionary Mandate Capabilities (Model management, Trade execution, Rebalancing)

**Operations Capabilities:**
- Settlement & Clearing Capabilities (Domestic, Cross-border, Securities)
- SWIFT Messaging Capabilities (MT/MX message processing, compliance screening)
- Reconciliation Capabilities (Cash, Securities, Nostro/Vostro)
- Regulatory Reporting Capabilities (MAS, HKMA, ECB, Fed)

---

## 11. Tools and Platforms

### 11.1 Enterprise Capability Modeling Platforms

| Tool | Strengths | Best For | Typical Use Case |
|---|---|---|---|
| **LeanIX** | Easy-to-use SaaS, strong integration, excellent heat maps, pre-built banking reference models | Quick-start, continuous capability management | Enterprise-wide capability maps with automated data collection |
| **Ardoq** | Dynamic visualization, scenario modeling, strong impact analysis | Collaborative modeling, what-if analysis | M&A integration planning, digital transformation impact analysis |
| **BizzDesign** (BiZZdesign Enterprise Studio) | Full EA repository, ArchiMate support, advanced analysis | Comprehensive EA + capability modeling | Large banks with mature EA practice, TOGAF/BIAN integration |
| **SAP Signavio** | Process mining + capability modeling, SAP integration | Process-to-capability alignment | Banks with heavy SAP footprint, process transformation |
| **Sparx EA** (Enterprise Architect) | Low cost, highly customizable, UML/ArchiMate/BPMN | Technical teams, custom frameworks | EA teams building bespoke capability repositories |
| **Abacus** (Avolution) | Capability-based planning, portfolio management | Investment planning | Government/defense (origin), financial services |
| **MEGA HOPEX** | Full lifecycle, GRC integration | Governance-heavy environments | Banks requiring integrated risk + capability management |
| **QualiWare** | Extended EA + business process + quality management | Regulated industries | Banks needing traceability from regulation through capability to implementation |

### 11.2 BIAN Service Landscape Portal

The **BIAN Service Landscape Portal** provides:
- Interactive browsing of all 400+ service domains
- Service domain definitions, behaviors, and inputs/outputs
- Mapping to business areas and other standards (ISO 20022, etc.)
- Downloadable capability reference models in Excel/CSV

### 11.3 Capability Modeling in General-Purpose Tools

| Tool | Approach | Limitations |
|---|---|---|
| **Microsoft Excel / Google Sheets** | Tabular capability hierarchies, conditional formatting for heat maps | No impact analysis, no dependency tracking, manual updates |
| **Draw.io / diagrams.net** | Visual capability maps as diagrams | Static, no analysis capability |
| **Atlassian Confluence + Gliffy** | Documented capability maps with embedded diagrams | Reference only, not analyzable |
| **Notion / SharePoint** | Wiki-style capability catalogs | Limited to documentation |

### 11.4 Tool Selection Criteria

| Criterion | Weight | Why |
|---|---|---|
| Banking reference models pre-loaded | High | BIAN alignment reduces manual modeling effort by 60-80% |
| Heat mapping and analytics | High | The primary value of capability models is in analysis |
| Integration with EA repository | High | Capability model must connect to applications, data, and technology |
| Collaboration and workflow | Medium | Multiple architects contribute to the model |
| API / integration capability | Medium | Import/export with CMDB, ITSM, and portfolio management tools |
| Cost | Medium | Full enterprise tools are $50-200K/year; Excel is free |

---

## 12. Capability Maturity Models

### 12.1 The Standard 5-Level Maturity Model

Each capability in the map can be assessed at one of five levels:

| Level | Name | Description | Banking Example (KYC/CDD) |
|---|---|---|---|
| **1** | **Ad Hoc** | Capability is performed reactively and inconsistently. No standard process, no defined roles, no metrics. | KYC checks done differently in every branch. Some use paper forms, some use excel. No defined process. |
| **2** | **Repeatable** | Capability has a defined basic process but is inconsistently applied. Some controls exist. Repeatable at team level but not enterprise-wide. | KYC process documented in SOP but not enforced. Some branches follow it, others don't. Manual quality checks exist. |
| **3** | **Defined** | Capability is standardized across the enterprise. Process is documented, roles are defined, training exists. Metrics are collected. | Enterprise-wide KYC standard with defined procedures, roles (KYC Officer, Reviewer, Approver). Training program in place. Audit confirms process adherence. |
| **4** | **Managed** | Capability is quantitatively managed. Metrics are systematically collected and used for decision-making. Performance is predictable. | KYC turnaround time, accuracy rate, and exception rate are tracked in dashboards. Management decisions use these metrics. Continuous monitoring in place. |
| **5** | **Optimizing** | Capability is continuously improved through quantitative feedback. Innovation is systematic. Lessons learned drive process evolution. | AI/ML improves KYC accuracy automatically. A/B testing of process changes. Industry benchmarking drives continuous improvement. New regulation triggers proactive process update. |

### 12.2 Domain-Specific Maturity Models

| Domain | Maturity Model | Applicable Capabilities |
|---|---|---|
| Data Management | DAMA-DMBOK Maturity Model | Data Governance, Data Quality, Data Architecture |
| Business Architecture | BIZBOK Maturity Model | Business Capability Modeling, Value Stream Mapping |
| IT Service Management | ITIL Maturity Model | Incident Management, Service Desk, Change Management |
| Software Development | CMMI-Dev | Development Capabilities (Requirements, Design, Test, Deploy) |
| Cyber Security | NIST Cybersecurity Framework | Security Monitoring, Incident Response, Identity Management |
| Cloud Readiness | Cloud Maturity Model | Cloud Infrastructure Management, Cloud Security |
| AI/ML | AI Maturity Model | Model Development, Model Operations, AI Governance |

### 12.3 Maturity Assessment Process

1. **Define assessment scope:** Which capabilities (typically L3-L4) and domains
2. **Select maturity model:** Use standard 5-level model or domain-specific
3. **Gather evidence:** Interviews, process documentation, metrics, audit reports, system analysis
4. **Score each capability:** Assign level 1-5 with supporting evidence
5. **Validate scores:** Review with business owners and domain experts
6. **Aggregate and visualize:** Roll up to parent capabilities (e.g., L3 average determines L2 score)
7. **Set target levels:** Based on strategy, competitive benchmarking, and regulatory requirements
8. **Track over time:** Reassess annually or quarterly

---

## 13. Capability Metrics and KPIs

### 13.1 Core Capability Metrics

| Metric | Definition | Unit | Data Source | Reporting Frequency |
|---|---|---|---|---|
| **Capability Maturity Score** | Current maturity level (1-5) | Level (1-5) | Maturity assessment | Annual |
| **Capability Performance Score** | Composite of operational KPIs | Score (1-5) or RAG | Operations dashboard | Monthly |
| **Cost per Capability** | Total cost to operate the capability | $ USD | Activity-based costing, TCO | Quarterly |
| **Cost Trend** | Quarter-over-quarter cost change | % | Financial systems | Quarterly |
| **Revenue per Capability** | Revenue attributable to the capability | $ USD | Revenue attribution | Monthly |
| **Risk Exposure** | Aggregate risk rating across dimensions | Score (1-5) or RAG | Risk register, ORMF | Monthly |
| **Compliance Status** | Whether capability meets regulatory requirements | Pass/Warning/Fail | Compliance assessments | Quarterly |
| **Process Efficiency** | Time/cost per capability output | Hours, $ | Process mining, operations | Monthly |
| **Automation Rate** | % of capability activities automated | % | Process mining, automation platforms | Quarterly |
| **System Health** | Health of applications supporting the capability | RAG | Application portfolio assessment | Monthly |
| **Talent Gap** | Availability of skilled resources for the capability | % filled | Workforce planning | Annual |
| **Customer Satisfaction** | CSAT/NPS scores linked to capability outcomes | Score 1-10 | Customer surveys, VOC | Monthly |

### 13.2 Banking-Specific Capability KPIs

| Capability | Leading KPI | Lagging KPI |
|---|---|---|
| **KYC/CDD** | Cases processed per day, automation rate | Cycle time (days), accuracy rate, regulatory findings |
| **Account Opening** | Application start rate, abandonment rate | Time-to-account (days), approval rate, NPS |
| **Transaction Monitoring** | Alerts generated, alert closure rate | False positive ratio, suspicious activity reports filed |
| **Payment Processing** | Transaction throughput | Straight-through processing rate, settlement time, fallouts |
| **Credit Assessment** | Applications received | Approval cycle time, default rate, risk-adjusted return |
| **Regulatory Reporting** | Data quality score, filing completeness | Filing accuracy, regulatory fines, audit findings |

### 13.3 Metric-Driven Decision Framework

```
Observability: We measure the capability
    ↓
Accountability: We assign owners to capability metrics
    ↓
Comparability: We benchmark against industry peers
    ↓
Actionability: Metrics drive investment decisions
    ↓
Improvement: Metrics trend positively over time
```

---

## 14. Common Pitfalls

### 14.1 Capability Model Too Abstract to Be Useful

**Problem:** Capability definitions are so broad that they could describe any organization. "Manage Customer Relationships" at L1 is fine, but if every L2 capability is equally abstract ("Deliver Value," "Optimize Operations"), the model cannot inform decisions.

**Solution:** Ensure that L3-L4 capabilities describe specific, recognizable organizational abilities. Validate with business stakeholders: *"Do you recognize this capability as something your team does?"*

### 14.2 Over-Engineering the Capability Hierarchy

**Problem:** Capability maps with 7-10 hierarchical levels, hundreds of interdependencies, and complex relationship matrices. The model becomes an end in itself, maintained by a dedicated team but never used for decision-making.

**Solution:** Aim for 5 levels maximum. Stop decomposing when capabilities become features or activities (process territory, not capability territory). A good test: *"If this capability were outsourced or automated, would the next level still make sense as a distinct organizational ability?"*

### 14.3 Disconnected from Strategy and Investment Decisions

**Problem:** The capability map sits in a repository, reviewed by the architecture team quarterly, but investment decisions continue to be made by project or by system. The capability model is wallpaper — nice to look at, but nobody acts on it.

**Solution:** Embed capability models in the investment planning process. Require every initiative above a threshold to reference the capabilities it improves. Use capability heat maps as the primary input to budget allocation.

### 14.4 Treating Capability Maps as Static Documents

**Problem:** The capability map is created once during a transformation program, published as a PDF, and never updated. Three years later, the map describes capabilities that no longer exist and omits capabilities that have been built.

**Solution:** Treat the capability model as a living artifact. Assign capability owners. Refresh assessments quarterly. Update the model when strategy changes, new products launch, or acquisitions close.

### 14.5 Not Validating with Business Stakeholders

**Problem:** Architects build the capability model in isolation, based on documentation and interviews with IT leadership. When line-of-business leaders see the model, they don't recognize it as their business.

**Solution:** Validate every level of the capability map with the people who actually do the work. Use workshop formats (capability mapping sessions, card sorting) rather than top-down design.

### 14.6 Confusing Capabilities with Organization Structure

**Problem:** The capability map mirrors the org chart because it's easier to model. "Manage Retail Banking" (L1) maps to the Retail Banking division, then L2 maps to each department within that division.

**Solution:** Capabilities should be stable across organizational restructures. A capability that disappears because a department was renamed is not a capability — it was an organizational unit. Test: *"If we reorganize, does this capability still exist?"*

### 14.7 Ignoring Dependencies Between Capabilities

**Problem:** Each capability is modeled in isolation. The KYC/CDD capability depends on the Document Verification capability, which depends on the Document Management capability — but the model doesn't capture these dependencies, so impact analysis is impossible.

**Solution:** Explicitly model dependencies between capabilities (at L3-L4). Use a matrix or graph structure. Identify critical path dependencies: *"When the Document Management capability fails, what else breaks?"*

### 14.8 Using Capability Models for Performance Management

**Problem:** Capability maturity scores are used to evaluate team leaders, creating perverse incentives to inflate scores and avoid honest assessments.

**Solution:** Disconnect capability assessment from individual performance evaluation. Use capability assessments for investment planning and strategic decision-making, not for appraising people.

---

## 15. Best Practices

### 15.1 Align Capability Models to Strategy

- **Start with strategy.** Before building the capability map, articulate the strategic goals. Every capability should trace to a strategic objective.
- **Use strategic importance as a dimension** on the heat map. Capabilities that are highly important but low maturity are the top priorities.
- **Review alignment annually** when strategy is refreshed. If strategy changes, capability priorities change.

### 15.2 Keep 5-7 Levels Deep Maximum

- **L1:** Strategic (4-8 capabilities covering the enterprise)
- **L2:** Core business domains (15-30 capabilities)
- **L3:** Operational capabilities (60-120 capabilities)
- **L4:** Functional capabilities (200-400 capabilities)
- **L5:** Sub-capabilities (500+ capabilities — only if tool supports it)
- **Total:** A large bank capability map typically has 300-500 capabilities at L3-L4. That's enough.

### 15.3 Link to Measurable Outcomes

- Every capability should have at least one measurable outcome.
- Example: KYC/CDD → *"Reduce client onboarding KYC cycle from 14 days to 3 days"*
- **If a capability cannot be measured, it is not well-defined.** Revisit the definition.

### 15.4 Embed in Investment Planning

- Make capability improvement the justification for investment.
- "This $10M project improves the KYC/CDD capability from maturity Level 2 to Level 4, enabling 3-day onboarding."
- Use capability-based portfolio management to avoid funding redundant capabilities or underfunding critical ones.

### 15.5 Maintain as a Living Model

- Assign a **capability owner** for every L3 capability (business stakeholder)
- Conduct **quarterly heat map reviews** with capability owners
- Update the model when:
  - Strategy changes
  - New products/regulations require new capabilities
  - Acquisitions add capabilities
  - Capabilities are retired or consolidated

### 15.6 Integrate with EA Repository

- Capability models should be in the same repository as applications, data, and technology
- Enable impact analysis: "If this application is decommissioned, what capabilities are affected?"
- Auto-populate heat map dimensions from EA data where possible (application health → capability health)

### 15.7 Standardize Naming Conventions

- **Use noun phrases:** "Customer Onboarding," "KYC Checks," "Payment Processing"
- **Avoid verbs:** "Onboard Customer" (capability) vs "Onboarding" (noun form preferred) — though both appear in practice
- **Be consistent:** Choose a convention and apply it globally
- **Avoid jargon:** "KYC/CDD" is standard in banking; general capabilities should use plain language

### 15.8 Start Small and Iterate

- **Phase 1:** Build a Level 1-2 map for one business domain (e.g., Customer Lifecycle)
- **Phase 2:** Validate with business stakeholders, refine definitions
- **Phase 3:** Add Level 3 for the prioritized domain
- **Phase 4:** Extend to adjacent domains
- **Phase 5:** Add heat maps and maturity assessments
- **Phase 6:** Integrate with investment planning

**Don't try to build the perfect enterprise-wide capability map in one pass.**

### 15.9 Use Reference Models to Accelerate

- **Banking:** Start with BIAN Service Landscape (400+ service domains)
- **Generic:** Start with BIZBOK or APQC Process Classification Framework
- **Technology:** Use SFIA for skills, DAMA-DMBOK for data capabilities
- **Adapt, don't adopt:** Reference models should be tailored — 70% adoption, 30% customization is a healthy ratio

### 15.10 Govern the Capability Model

- Establish a **Capability Model Governance Board** with business and IT representation
- Define the **change process**: request, evaluate, approve, publish
- Publish a **Capability Model Standard** document defining:
  - Naming conventions
  - Decomposition rules
  - Assessment methodology
  - Update frequency
  - Owner responsibilities

---

## 16. Appendix A: Banking Capability Decomposition Example

### Complete Decomposition — Trade Finance Domain

```
L1: MANAGE PRODUCTS & SERVICES
├── L2: Manage Trade Finance
│   ├── L3: Letter of Credit (LC) Management
│   │   ├── L4: LC Issuance
│   │   │   ├── L5: LC Application Processing
│   │   │   ├── L5: LC Terms & Conditions Setup
│   │   │   ├── L5: LC Collateral Management
│   │   │   └── L5: LC Issuance Confirmation & Advising
│   │   ├── L4: LC Amendment
│   │   │   ├── L5: Amendment Request Processing
│   │   │   ├── L5: Amendment Approval Workflow
│   │   │   └── L5: Amendment Notification
│   │   ├── L4: LC Document Check
│   │   │   ├── L5: Document Presentation Receipt
│   │   │   ├── L5: Discrepancy Checking
│   │   │   ├── L5: Document Acceptance/Rejection
│   │   │   └── L5: SWIFT MT700/MT707 Message Handling
│   │   └── L4: LC Settlement
│   │       ├── L5: Payment Processing (at sight / deferred)
│   │       ├── L5: Acceptance Processing
│   │       ├── L5: Negotiation Processing
│   │       └── L5: Reimbursement Claim Processing
│   │
│   ├── L3: Guarantee & Standby LC Management
│   │   ├── L4: Guarantee Issuance
│   │   ├── L4: Guarantee Amendment
│   │   ├── L4: Guarantee Claim Handling
│   │   └── L4: Guarantee Expiry & Release
│   │
│   ├── L3: Supply Chain Finance (SCF)
│   │   ├── L4: Receivables Finance
│   │   ├── L4: Payables Finance (Reverse Factoring)
│   │   ├── L4: Inventory Finance
│   │   └── L4: SCF Platform Management
│   │
│   ├── L3: Documentary Collections
│   │   ├── L4: Collection Instruction Processing
│   │   ├── L4: Document Handling (D/P, D/A)
│   │   ├── L4: Collections Payment Processing
│   │   └── L4: Unpaid/Negotiation Handling
│   │
│   └── L3: Trade Finance Operations
│       ├── L4: Trade Limit Management
│       ├── L4: Trade Confirmation & Settlement
│       ├── L4: SWIFT Network Connectivity (MT messages)
│       ├── L4: Regulatory Reporting (trade data to MAS)
│       └── L4: Trade Finance Reconciliation

L1: MANAGE RISK & COMPLIANCE
├── L2: Manage Financial Crime Compliance
│   ├── L3: AML/CFT
│   │   ├── L4: Customer Risk Assessment & Profiling
│   │   ├── L4: Transaction Screening (real-time, batch)
│   │   ├── L4: Sanctions Screening
│   │   │   ├── L5: Name Screening (fuzzy matching)
│   │   │   ├── L5: Payment Message Screening
│   │   │   └── L5: Sanctions List Management
│   │   ├── L4: Suspicious Activity Monitoring
│   │   ├── L4: STR/SAR Filing
│   │   └── L4: AML Regulatory Reporting
│   │
│   ├── L3: Fraud Management
│   │   ├── L4: Fraud Detection (rules engine, ML models)
│   │   ├── L4: Fraud Investigation & Case Management
│   │   ├── L4: Fraud Prevention (device fingerprinting, behavioral)
│   │   └── L4: Fraud Loss Recovery
│   │
│   └── L3: Trade-Based Money Laundering (TBML) Detection
│       ├── L4: Trade Document Anomaly Detection
│       ├── L4: Pricing Discrepancy Detection
│       ├── L4: Dual-Use Goods Screening
│       └── L4: Circular Trade Detection
│
├── L2: Manage Credit Risk
│   ├── L3: Credit Origination
│   │   ├── L4: Credit Application Processing
│   │   ├── L4: Financial Spreading & Analysis
│   │   ├── L4: Credit Scoring & Rating
│   │   └── L4: Credit Approval Workflow
│   ├── L3: Credit Monitoring
│   │   ├── L4: Covenant Monitoring
│   │   ├── L4: Early Warning Indicator Tracking
│   │   ├── L4: Limit Utilization Tracking
│   │   └── L4: Portfolio Credit Risk Reporting
│   └── L3: Credit Remediation
│       ├── L4: Watchlist Management
│       ├── L4: Restructuring & Workout
│       └── L4: Provisioning & Impairment Calculation
│
├── L2: Manage Market & Liquidity Risk
│   ├── L3: Market Risk Measurement
│   │   ├── L4: VaR Calculation & Backtesting
│   │   ├── L4: Stress Testing & Scenario Analysis
│   │   ├── L4: Risk Limit Monitoring
│   │   └── L4: P&L Attribution & Valuation
│   ├── L3: Liquidity Risk Management
│   │   ├── L4: LCR Monitoring & Reporting
│   │   ├── L4: NSFR Monitoring & Reporting
│   │   ├── L4: Liquidity Stress Testing
│   │   └── L4: Contingency Funding Plan Management
│   └── L3: IRRBB Management
│       ├── L4: Interest Rate Sensitivity Measurement
│       ├── L4: Earnings-at-Risk Monitoring
│       └── L4: Hedge Effectiveness Testing
│
├── L2: Manage Operational Risk
│   ├── L3: Operational Risk Identification & Assessment
│   │   ├── L4: RCSA (Risk & Control Self-Assessment)
│   │   ├── L4: Key Risk Indicator (KRI) Monitoring
│   │   └── L4: Scenario Analysis
│   ├── L3: Operational Risk Events Management
│   │   ├── L4: Loss Data Collection
│   │   ├── L4: Root Cause Analysis
│   │   └── L4: Remediation Tracking
│   └── L3: Business Continuity Management
│       ├── L4: BIA (Business Impact Analysis)
│       ├── L4: Disaster Recovery Planning
│       ├── L4: BCP Testing & Exercising
│       └── L4: Crisis Management
│
└── L2: Manage Regulatory Compliance
    ├── L3: Regulatory Change Management
    │   ├── L4: Regulatory Horizon Scanning
    │   ├── L4: Regulatory Impact Assessment
    │   └── L4: Regulatory Implementation Planning
    ├── L3: Regulatory Reporting
    │   ├── L4: MAS 610 / MAS 1003 Reporting
    │   ├── L4: BASEL Reporting (Pillar 1, 2, 3)
    │   ├── L4: FATCA / CRS Reporting
    │   ├── L4: Trade Repository Reporting (MAS)
    │   └── L4: Regulatory Data Quality Management
    ├── L3: Audit & Assurance
    │   ├── L4: Internal Audit Coordination
    │   ├── L4: Regulatory Exam Management
    │   └── L4: Remediation Tracking
    └── L3: Licensing & Registration
        ├── L4: License Application Management
        ├── L4: Regulatory Notifications
        └── L4: Register of Representatives (CMCAS for Singapore)
```

---

## 17. Appendix B: Capability Heat Map Template

### Markdown Version (for Git-based repositories)

```markdown
# Capability Heat Map: [Domain Name]
**Period:** [Quarter/Year] | **Owner:** [Name]

## Scoring Legend

| Score | Maturity (L1-L5) | Performance | Cost | Risk | Compliance |
|---|---|---|---|---|---|
| 🟢 Good | 4-5 (Managed/Optimizing) | Exceeding target | Under budget | Low | Fully compliant |
| 🟡 Watch | 2-3 (Repeatable/Defined) | Meeting target | On budget | Medium | Minor issues |
| 🔴 Critical | 1 (Ad Hoc/Incomplete) | Below target | Over budget | High | Non-compliant |

## Heat Map Grid

| L1 | L2 | L3 | Maturity | Perf | Cost | Risk | Comp | Strategic | Priority |
|---|---|---|---|---|---|---|---|---|---|
| [Strategic] | [Core] | [Operational] | 🟢 4 | 🟢 5 | 🟢 $2M | 🟢 L | 🟢 Pass | 🟢 H | 🟢 Maintain |
| | [Core 2] | [Operational A] | 🟡 3 | 🟡 3 | 🟡 $5M | 🔴 H | 🟡 Warn | 🟢 H | 🔴 Transform |
| | | [Operational B] | 🔴 2 | 🔴 2 | 🔴 $8M | 🔴 H | 🔴 Fail | 🟢 H | 🔴 Critical Fix |
| | | [Operational C] | 🟢 4 | 🟡 3 | 🟢 $1M | 🟢 L | 🟢 Pass | 🟡 M | 🟢 Sustain |
| | [Core 3] | [Operational D] | 🟡 3 | 🟢 4 | 🟡 $3M | 🟡 M | 🟢 Pass | 🟡 M | 🟡 Improve |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
```

### Excel/Spreadsheet Version

A tabular template compatible with Excel, Google Sheets, or any spreadsheet tool:

| Column | Purpose | Example |
|---|---|---|
| L1 | Strategic capability domain | Manage Customer Relationships |
| L2 | Core/Business capability | Onboard Customer |
| L3 | Operational capability | KYC/CDD |
| L4 | Functional capability (optional) | Document Verification |
| Maturity Score | Numeric score (1-5) | 3 |
| Maturity Level | Text label based on score | Defined |
| Performance Score | Numeric score (1-5) | 2 |
| Cost (Annual) | Dollar value | $8,000,000 |
| Cost Score | RAG based on budget variance | Red |
| Risk Score | RAG from risk assessment | Red |
| Compliance Status | Pass/Warning/Fail | Warning |
| Strategic Importance | High/Medium/Low | High |
| Priority | Critical/High/Medium/Low | Critical |
| Capability Owner | Business owner name | [Name] |
| Improvement Initiatives | Linked projects or work packages | KYC Automation Program |
| Target Maturity (N+1) | Target level for next planning cycle | 4 |
| Target Date | Target date for maturity level | Q4 2025 |
| Notes | Free text | Manual processes, aging platform |

### Conditional Formatting Rules (for spreadsheets)

- **Maturity Score:** < 2 = Red, 2-3 = Amber/Yellow, > 3 = Green
- **Performance Score:** < 2 = Red, 2-3 = Amber/Yellow, > 3 = Green
- **Cost:** > 20% over budget = Red, within ±20% = Amber, under budget = Green
- **Risk:** High = Red, Medium = Amber, Low = Green
- **Compliance:** Fail = Red, Warning = Amber, Pass = Green
- **Priority:** Critical = Red, High = Amber, Medium = Yellow, Low = Green

### Composite Scoring Formula

For an overall capability health score:

```
Overall Score = (Maturity × 0.15) + (Performance × 0.20) + (Cost × 0.15) + (Risk × 0.20) + (Compliance × 0.30)

Where:
  - Each dimension is normalized to 1-5
  - Compliance is weighted highest (30%) for regulated industries
  - Adjust weights based on organizational priorities
```

**Composite RAG mapping:** Score ≥ 4.0 = 🟢 Green | 2.5-3.9 = 🟡 Amber | < 2.5 = 🔴 Red

---

## 18. References and Further Reading

### Standards and Frameworks

- **BIAN Service Landscape 12.0** — bian.org (banking capability reference model)
- **TOGAF Standard 10th Edition** — The Open Group (capability-based planning, architecture capability)
- **BIZBOK Guide 10.0** — Business Architecture Guild (business capability modeling methodology)
- **CMMI V2.0** — ISACA / CMMI Institute (capability maturity levels)
- **DAMA-DMBOK 2nd Edition** — DAMA International (data management capabilities)
- **SFIA 8** — SFIA Foundation (skills and competency capabilities)
- **FS-PCA** — APQC / BIAN (Financial Services Process Classification Agreement)
- **ISO 42010** — Systems and software engineering — Architecture description

### Books

- *Business Architecture: A Practical Guide* — Jonathan Whelan, Graham Meaden
- *Enterprise Architecture at Work* — Marc Lankhorst (ArchiMate, capability modeling)
- *Capability-Based Planning for Defense* — Paul K. Davis (origins of CBP in defense)
- *The Business Architecture Body of Knowledge™ (BIZBOK) Guide* — Business Architecture Guild
- *Data Management Body of Knowledge (DAMA-DMBOK)* — DAMA International

### Articles and Papers

- "Capability-Based Planning" — MITRE Systems Engineering Guide
- "Business Capability Modeling: A Practical Guide" — LeanIX / Forrester
- "The Role of Business Capabilities in Digital Transformation" — Gartner
- "BIAN Service Landscape: A Banking Industry Standard" — BIAN White Paper
- "Architecture Capability Maturity Models" — The Open Group

### Tools

- **LeanIX** — leanix.net (SaaS capability modeling with heat maps)
- **Ardoq** — ardoq.com (collaborative capability mapping)
- **BizzDesign** — bizzdesign.com (full EA + capability repository)
- **SAP Signavio** — signavio.com (process + capability modeling)
- **Sparx Enterprise Architect** — sparxsystems.com (customizable EA tool)
- **Abacus (Avolution)** — avolution.com (capability-based planning)
- **BIAN Service Landscape Portal** — bian.org/service-landscape
- **MEGA HOPEX** — mega.com (integrated EA + GRC + capability modeling)

---

> **Document version:** 1.0  
> **Last updated:** July 2026  
> **Author context:** Jack Liu Shurui, Solution Architect — Crédit Agricole CIB  
> **License:** This work is licensed under Creative Commons Attribution 4.0 International (CC BY 4.0)
