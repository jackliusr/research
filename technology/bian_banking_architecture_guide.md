
# BIAN — The Banking Industry Architecture Network (Banking's Open Architecture Standard)

> **Scope:** This guide is the dedicated deep-dive on **BIAN** (the Banking Industry Architecture
> Network) as a standard, framework, and movement. It covers the BIAN Service Landscape, the
> Service Domain model, the Business Object Model (BOM), the meta model, design principles,
> release evolution (9.x → 14.0), adoption journeys, tools and certification, the vendor
> landscape, and BIAN's role in service-oriented / composable / cloud-native banking
> architecture.
>
> The companion guide [`data_models_banking_insurance_guide.md`](data_models_banking_insurance_guide.md)
> covers BIAN in passing as one of the canonical banking *data* models (alongside IBM BDW,
> Teradata FSLDM, FIS, Silverston). This guide goes deeper on **BIAN itself** and treats the data
> comparison only in summary — see §11 for the cross-reference.
>
> **Audience:** Solution/enterprise architects in banking (Jack Liu, Crédit Agricole CIB) who need
> to use BIAN as a capability map, integration standard, vendor-evaluation lens, and target
> architecture blueprint.

---


1. [What is BIAN?](#1-what-is-bian)
2. [Why BIAN Exists — the Problem It Solves](#2-why-bian-exists--the-problem-it-solves)
3. [The BIAN Service Landscape](#3-the-bian-service-landscape)
4. [The Service Domain — the Elemental Building Block](#4-the-service-domain--the-elemental-building-block)
5. [The Standard Operations Vocabulary (Action Terms)](#5-the-standard-operations-vocabulary-action-terms)
6. [Business Objects — the BIAN Business Object Model (BOM)](#6-business-objects--the-bian-business-object-model-bom)
7. [The BIAN Meta Model and Design Artifacts](#7-the-bian-meta-model-and-design-artifacts)
8. [BIAN Design Principles](#8-bian-design-principles)
9. [The BIAN API Standard — Semantic APIs](#9-the-bian-api-standard--semantic-apis)
10. [Release Evolution: 8.x → 14.0 and Beyond](#10-release-evolution-8x--140-and-beyond)
11. [BIAN vs Other Standards (Data Models, ISO 20022, TM Forum, Open Banking)](#11-bian-vs-other-standards--data-models-iso-20022-tm-forum-open-banking)
12. [BIAN in Practice — How Banks Use It](#12-bian-in-practice--how-banks-use-it)
13. [The Adoption Journey — Levels, Methodology, Pitfalls](#13-the-adoption-journey--levels-methodology-pitfalls)
14. [BIAN and Cloud-Native / Composable Banking](#14-bian-and-cloud-native--composable-banking)
15. [BIAN and Core Banking Vendors](#15-bian-and-core-banking-vendors)
16. [The BIAN Role in a Bank Architect's Toolbox](#16-the-bian-role-in-a-bank-architects-toolbox)
17. [Singapore / Asia Context](#17-singapore--asia-context)
18. [Getting Started with BIAN](#18-getting-started-with-bian)
19. [The Future of BIAN (2026+) — and the Honest Relevance Question](#19-the-future-of-bian-2026--and-the-honest-relevance-question)
20. [Key Takeaways](#20-key-takeaways)
21. [References and Further Reading](#21-references-and-further-reading)

---

## 1. What is BIAN?

**BIAN — the Banking Industry Architecture Network e.V.** — is a global, independent,
member-owned, **not-for-profit association** (registered association / `e.V.` in Germany) that
creates and promotes an **open, standardized framework for banking IT architecture**.

| Attribute | Detail |
|---|---|
| **Founded** | 2008 |
| **Legal form** | Registered association (e.V.), not-for-profit, member-owned |
| **HQ** | Frankfurt, Germany (Friedrich-Ebert-Anlage 36) |
| **Members** | ~113 (as of 2026): ~40 banks, ~59 software/tech partners, consultancies, academics; **18 active working groups** |
| **Mission** | "Establish and promote a common architectural framework for enabling banking interoperability"; lower integration cost; speed innovation |
| **Output** | The **BIAN standard** — a service-oriented reference model for banking: Service Landscape, Business Object Model, meta model, design guidelines, and OpenAPI-based Semantic API specifications |
| **Executive Director** | Hans Tesselaar |
| **Website / Portal** | bian.org / portal.bian.org / github.com/bian-official/public |

**In one sentence:** BIAN is banking's attempt to build the "Rosetta Stone" of banking
architecture — a **common vocabulary and blueprint** that lets any bank, any vendor, and any
consultancy describe banking capabilities the same way.

BIAN's core belief: a bank's IT estate is a collection of **~300–400 discrete business
capabilities** (a payment execution capability, a customer onboarding capability, a collateral
administration capability…). If the industry agrees on what those capabilities **are** and how
they **interoperate**, then:

- banks can compose and re-compose their architecture from reusable building blocks,
- software vendors can build products that slot into any bank's landscape,
- integration cost (historically point-to-point and proprietary) collapses,
- legacy systems can be mapped, wrapped, and progressively replaced against a stable target.

The underlying architectural style is **service-oriented architecture (SOA)**, updated over the
years with **API-first**, **domain-driven design (DDD)**, **event-driven** and **cloud-native**
thinking (see §10, §14).

### The "banking standards" landscape — where BIAN sits

BIAN deliberately does **not** compete with message formats, data warehouse models, or
regulatory regimes. Its lane is **the capability/service architecture** of the bank:

- **BIAN** → *What capabilities does a bank have, and how do they inter-operate?* (service/capability architecture + the data those services exchange)
- **ISO 20022** → *How are financial messages formatted?* (message syntax/semantics) — BIAN maps to it
- **IBM BDW / Teradata FSLDM / FIS / Oracle** → *How is data structured for warehousing/analytics?* (data models)
- **Open Banking (PSD2 / UK / SG)** → *Which APIs must be exposed to third parties, under what rules?* (regulatory/commercial API regime)
- **TM Forum** → the telecom equivalent of BIAN

See §11 for the detailed comparison.

---

## 2. Why BIAN Exists — the Problem It Solves

### 2.1 The legacy bank IT problem

Banks grew by acquisition, product silo, and decades of bolting-on. The result, by BIAN's own
mission statement, is a landscape that is:

- **Complex and inflexible** — intertwined monoliths, duplicated capabilities in every silo,
- **Proprietary** — every integration is bespoke; nothing is reusable across products, channels,
  or banks,
- **Slow to change** — "strategic change" takes years because the architecture cannot be re-sequenced
  or re-composed,
- **Vendor-locked** — switching or blending vendors means re-integrating everything.

### 2.2 The diagnosis: no shared language

When a bank says "we need a new onboarding capability", the software vendor hears that in its own
product's terms; the bank's own legacy teams hear it in host-system terms; the integration vendor
hears it in interface terms. Every project renegotiates meaning from scratch. BIAN's founding
insight: the industry needs a **shared semantic framework** — a standard way to identify and define
banking IT services.

### 2.3 The BIAN answer

BIAN defines a **canonical decomposition of the bank** into discrete, non-overlapping
**Service Domains** — each a self-contained business capability with a defined responsibility,
a standard set of service operations, and named business objects. The decomposition is:

- **MECE** (mutually exclusive, collectively exhaustive) — every banking activity falls in exactly
  one Service Domain, and the whole bank is covered,
- **Elemental** ("right-sized") — a Service Domain is either adopted wholly or not at all; it
  cannot be split, which is what makes it canonical,
- **Stable** — Service Domains change rarely, which makes them a reliable target for migration
  and a reliable contract for procurement.

### 2.4 What the bank gets

| Benefit | Mechanism |
|---|---|
| **Lower integration cost** | Standardized service interfaces instead of bespoke point-to-point |
| **Composability** | Capabilities reusable across products, channels, and processes |
| **Vendor interoperability** | Vendors map products to Service Domains; banks compare on the same map |
| **Faster change** | Re-sequence service calls rather than re-engineer systems |
| **A migration blueprint** | A stable target architecture for legacy wrapping and replacement |
| **Organizational alignment** | Service Domains map to business units (the "aligned operating model") |

> **The "Rosetta Stone" framing:** a bank describes its current and target architecture in BIAN
> terms; every vendor maps its products to BIAN Service Domains; the two views can then be
> compared directly — "this vendor covers our required Service Domains, that one doesn't", "this
> gap is a build, that gap is a buy". BIAN turns architecture negotiation into a common-language
> exercise.

---

## 3. The BIAN Service Landscape

### 3.1 What it is

The **BIAN Service Landscape** is the core, most famous artifact of the standard: a **reference
framework / map of the entire bank's business capabilities**, expressed as the full set of BIAN
Service Domains. The BIAN Semantic API Practitioner Guide calls it "a reference structure that
categorizes and organizes BIAN Service Domains for ease of access".

Sizes (approximate, by release):

- ~**320+** Service Domains in the 8.x/9.x-era standard,
- **322** completed Service Domains in **Service Landscape 11.0** (Dec 2022),
- ~**326** Service Domains / 38 business domains in **12.0** (per SunTec),
- **14.0** (2026): rationalized; some domains renamed/removed/added (especially payments) plus
  three new insurance domains.

### 3.2 The hierarchy: Business Area → Business Domain → Service Domain

The BIAN meta model (a UML model) defines three classification levels (Wikipedia / BIAN metamodel):

1. **Business Area** — the highest-level classification; "aspects of business activity that have
   similar supporting application and information-specific needs".
2. **Business Domain** — a coherent collection of capabilities within an area, "associated with
   skills and knowledge recognizable in the banking business" (the "~55–60 business domains" level;
   12.0 lists 38 in the public model).
3. **Service Domain** — the finest partition; "the elemental building blocks of the service
   landscape". Service Domains are **stable and layout-independent**; Business Areas/Domains are
   **classifications that can be rearranged into different landscape layouts**.

**Important (verified):** the task assumption of "9 business areas named Customer/Product/Channel/
…" is **not** the actual published structure. BIAN publishes **two coexisting layouts** of the same
Service Domain set (per the BIAN guide, §3.1):

- **Matrix layout** — organizes Service Domains by *predominantly technical properties* into
  business areas such as (V8.0 matrix view): **Reference Data, Sales & Service, Operations &
  Execution, Risk & Compliance, Business Support, Business Direction**.
- **Value Chain layout** — a more *enterprise/organizational* view, preferred by business
  practitioners, with business areas such as: **Operations, Products, Customers, Channels,
  Finance & Risk Management, Business Development, Marketing & Development, Resource
  Management, IT Management / Corporate Services**.

Both layouts contain **exactly the same Service Domains** — only the grouping differs. This
illustrates a key BIAN property: **the Service Domain set is canonical; the grouping is a view.**
A bank should therefore anchor on Service Domains, not on a specific landscape grouping.

The "8 functional categories" seen in the official GitHub README's DeepWiki docs (Payment
Services, Account Management, Lending Services, Trading & Capital Markets, Customer Services,
Risk & Compliance, Product Management, Infrastructure) is a **third, implementation-oriented
grouping** used for the API repository — further proof that groupings are presentation, not
substance.

### 3.3 Example Service Domains (the real ones)

Extracted from BIAN's actual API repository (release 14.0 names) — payments, cards, lending,
trade, customer, and support areas:

| Area | Example Service Domains |
|---|---|
| **Payments** | Payment Execution, Payment Instruction, Payment Order, Direct Debit, Direct Debit Mandate, Standing Order, Disbursement, Cheque Processing, Cheque Lock Box, ACH Fulfilment, Financial Gateway, Correspondent Bank Operations, Transaction Engine |
| **Cards** | Card Authorization, Card Capture, Card Billing & Payments, Card Clearing, Card Financial Settlement, Card Network Participant Facility, Card Transaction Switch, Card Terminal Administration/Operation, Card eCommerce Gateway, Merchant Relations/Acquiring, Credit/Charge Card |
| **Accounts** | Current Account, Corporate Current Account, Savings Account, Deposit Account, Position Keeping, Securities Position Keeping, Reward Points Account, Customer Position |
| **Lending** | Loan, Consumer Loan, Corporate Loan, Mortgage Loan, Syndicated Loan, Leasing, Factoring, Credit Facility, Merchandising Loan, Underwriting, Credit Risk Operations, Collections, Card Collections, Delinquent Account Handling, Account Recovery |
| **Trade & Capital Markets** | Trade Finance, Letter of Credit, Bank Guarantee, Bank Drafts & Travelers Checks, Project Finance, Public Offering, Private Placement, ECM/DCM, Market Making, Program Trading, Dealer Workbench/Desk, Trade Settlement, Trade/Price Reporting, Custody Administration |
| **Customer** | Customer Relationship Management, Customer Offer, Customer Agreement, Customer Position, Customer Tax Handling, Customer Access Entitlement, Party Lifecycle Management, Party Authentication, Customer Case Management, Prospect Campaign Design/Execution/Management, Lead/Opportunity Management, Sales Planning |
| **Reference Data & Support** | Party Reference Data Directory, Product Directory, Location Data Management, Financial Instrument Reference Data Management, Market Data Switch, Document Services, Archive Services, Enterprise Architecture, Service Directory, Business Unit Management |

### 3.4 The InSite / model browser

The current Service Landscape is published via the **BIAN InSite portal** (e.g.
`bian.org/servicelandscape-13-0-0/`), offering:

- **Value Chain View** and **Matrix View** of the Service Domain Landscape,
- **Overview Diagrams** per business area (Bank Relations, Business Development, Card Products,
  Channels, Corporate Banking Products, Corporate Finance, Lending, Payments, Product and Price,
  Retail Banking and Consumer, Wealth),
- **Business Scenarios** (+ "generally usable scenario snippets"),
- **Business Capability Map** (top-level view + views per business domain),
- **BIAN Meta Model Overview**, **Information Architecture Overview**, **Wireframes Overview**,
  and a **Dashboard**.

---

## 4. The Service Domain — the Elemental Building Block

### 4.1 Anatomy of a Service Domain

A Service Domain is defined as the application of **one pattern of behavior ("Functional Pattern")**
to **instances of one type of asset**, for the **complete lifecycle**, as many times as required.
It "does something to something, from start to finish."

A Service Domain comprises:

| Component | Description |
|---|---|
| **Functional Pattern** | One of **19** standardized commercialization behaviors (see §4.2) |
| **Asset Type** | The kind of thing the domain acts on — one of ~250–300 classified asset types (e.g., a current account facility, an ATM network, a customer relationship) |
| **Control Record** | The domain's "master record" — the main business information it governs; in object terms, "a type of class"; in DDD terms, the **aggregate root** (CR) |
| **Behavior Qualifiers (BQs)** | Finer partitions of the control record giving service operations an unambiguous scope — in DDD terms, the **sub-aggregates** |
| **Service Operations** | The API-like operations the domain exposes (see §5) |
| **First-Order Connections** | Known service dependencies to/from other Service Domains (caller→called with a specific operation), often tied to a **business event** |
| **Information Profile** | The complete information make-up if the domain runs as a standalone service center (control records + reference data + configuration/status + activity records) |

### 4.2 The 19 Functional Patterns

The "verbs" of banking. Every Service Domain applies exactly one. They group into four families
(the grouping is BIAN's own):

| Family | Functional Pattern | Generic Artifact (resulting "noun") |
|---|---|---|
| **Direct** (make plans, design, solutions) | DIRECT | Strategy |
| | MANAGE | Management Plan |
| | ADMINISTER | Administrative Plan |
| | DESIGN | Specification |
| | DEVELOP | Development |
| | PROCESS | Procedure |
| **Initiate** (process work, operate tooling) | OPERATE | Operating Session |
| | MAINTAIN | Maintenance Arrangement |
| | FULFILL | Arrangement |
| | TRANSACT | Transaction |
| | ADVISE | Advice |
| | MONITOR | State |
| | TRACK | Log Record |
| **Register** (catalogue & enroll) | CATALOG | Directory Entry |
| | ENROLL | Membership |
| | AGREE TERMS | Agreement |
| **Evaluate** (perform tests, checks, analysis) | ASSESS | Assessment |
| | ANALYSE | Analysis |
| **Provide** | ALLOCATE | Allocation |

**Why it matters for architects:** the functional pattern instantly tells you the *nature* of a
Service Domain — `Current Account` is a FULFILL domain (it fulfils an arrangement), `Card
Authorization` is a TRANSACT domain, `Party Reference Data Directory` is a CATALOG domain,
`Market Analysis` is an ANALYSE domain, `Customer Case Management` is a PROCESS domain. It also
determines the **default set of service operations** the domain offers (§5.4).

### 4.3 Control Records and the fractal pattern

The **control record** (CR) is the heart of the domain's data. It holds everything needed to
control processing, everything referenced, and everything produced across the lifecycle.
Behavior Qualifiers (BQs) partition it using a **behavior qualifier type** derived from the
functional pattern (e.g., a PROCESS domain decomposes into *work steps*; a CATALOG domain into
*properties*; a TRANSACT domain into *tasks/steps*). The partitions have the same characteristics
as their parent — BIAN explicitly calls this a **fractal pattern**: the action-term vocabulary
applies uniformly at CR, BQ, and sub-qualifier level. Partitions must stay **MECE** (mutually
exclusive, collectively exhaustive).

**Real example (MortgageLoan service domain, from the official OpenAPI specs):**

```
Control Record (aggregate root):  MortgageLoanFacility   — the loan facility itself
Behavior Qualifiers (sub-aggregates):
  - Billing        (interest calculations & billing schedules)
  - Collateral     (property & valuation)
  - Fees           (fee structures & charges)
  - Lien           (legal claim management)
  - Payments       (principal & interest processing)
  - Sweep          (automated fund movement)
  - Withdrawals    (withdrawal processing)
```

**Real example (DirectDebit):** CR = `DirectDebitFacility`; BQs = `PaymentTracking`,
`FundsAvailableCheck`, `PaymentInitiation`, `VerifyMandate`.

**Real example (CustomerOffer):** CR = `CustomerOfferProcedure`; BQs = `FacilityApplication`,
`Disclosures`, `Credit`, `Underwriting`, `Compliance` — orchestrating the whole mortgage offer
evaluation (see §7.3).

### 4.4 Right-sizing Service Domains ("elemental" design)

BIAN's "right-sizing" technique — the formal method the working groups use — decomposes asset
types to the lowest granularity at which they retain **unique business context or ownership**.
Below that level, functions become generic *utilities* (e.g., "register an entity") rather than
assignable business responsibilities. This is the key distinction:

- **Service Domains** = discrete **operational capabilities** assignable to a responsible party
  and reused by other parts of the business (e.g., Customer Relationship Management, Document
  Services).
- **Utilities** = standard actions/behaviors encoded as reusable code modules (procedures,
  microservices) that execute independently (e.g., a pricing calculation).

The rule prevents both over-granular services (which break uniqueness) and mega-domains (which
split responsibility). A Service Domain should pass the "outsourcing test": it is sized so that it
*could* be outsourced as a whole.

---

## 5. The Standard Operations Vocabulary (Action Terms)

### 5.1 The canonical action terms

BIAN defines a **controlled vocabulary of action terms** characterizing the purpose of any
service exchange. The current standard set (17 terms, per the BIAN guide; slightly revised since
Service Landscape 7.0 to align with REST):

| Category | Action Terms |
|---|---|
| **Control the Service Domain overall** | Activate, Configure, Feedback |
| **Create a new control-record instance (new lifecycle)** | Create, Initiate, Register, Evaluate, Provide |
| **Act on / change an existing instance** | Update, Control, Exchange, Capture, Execute, Request, Grant |
| **Obtain or subscribe to information (read-only, no state change)** | Retrieve, Notify |

Note the deliberate read/write split (Retrieve/Notify vs the rest) — BIAN explicitly frames this
as supporting **CQRS-style deployments**.

### 5.2 Semantics of the key terms

| Operation | Meaning | HTTP (REST mapping) |
|---|---|---|
| **Initiate** | Instantiate a new CR/BQ (replaces the older "Create") | POST `/{SD}/Initiate` |
| **Register** | Create a directory/catalog entry (CATALOG/ENROLL domains) | POST `/{SD}/Register` |
| **Evaluate** | Establish an agreement/assessment requiring evaluation logic | POST `/{SD}/Evaluate` |
| **Update** | Modify an existing instance; idempotent | PUT `/{SD}/{id}/Update` |
| **Control** | Manage processing lifecycle (suspend/resume/terminate); idempotent | PUT `/{SD}/{id}/Control` |
| **Exchange** | Accept/verify/acknowledge — multi-party verification (correspondent/clearing) | PUT `/{SD}/{id}/Exchange` |
| **Execute** | Trigger an automated action (vs. Request = human intervention) | PUT `/{SD}/{id}/Execute` |
| **Capture** | Record activity/event/audit information against an instance | PUT `/{SD}/{id}/Capture` |
| **Request** | Request manual intervention / human decision (escalation) | PUT `/{SD}/{id}/Request` |
| **Grant** | Grant permission/approval | PUT `/{SD}/{id}/Grant` |
| **Retrieve** | Query current state (read-only) | GET `/{SD}/{id}/Retrieve` |
| **Notify** | Event notification / subscription | GET / event subscription (AsyncAPI) |

### 5.3 How operations scope a request

A BIAN service operation is scoped by **three concerns**:

1. **The Service Domain's core functionality** — which control record (or partition) it acts on,
2. **The action term** — which filters the control record attributes to the input/output messages,
3. **Optionally the Behavior Qualifier** — narrowing to a sub-partition of the control record.

This is why BQs exist: an "Execute" against a current account is ambiguous (execute a payment vs
execute a deposit?); scoping it to a BQ makes the service operation unambiguous.

### 5.4 Default operations per functional pattern

BIAN provides a **default mapping** of action terms to each of the 19 functional patterns
(used to seed the Semantic API Portal). Notable patterns:

- Activate/Configure/Feedback apply to **all** domains,
- exactly **one** creation term applies per pattern (Initiate for FULFILL/TRANSACT/PROCESS/
  OPERATE/MONITOR/TRACK/ADVISE; Register for CATALOG/ENROLL; Evaluate for ASSESS/ANALYSE/
  AGREE TERMS; etc.),
- Retrieve and Notify apply in all cases.

A service exchange is a **semantic dependency**, not a prescribed choreography — BIAN does not
assume request/response vs one-way vs iterative dialogue; that is an implementation decision.

### 5.5 REST mapping of action terms

For the Semantic APIs, action terms are converted to **noun form** for REST endpoints
(Activate→Activation, Initiate→Initiation, Register→Registration, Evaluate→Evaluation,
Provide→Provision, Execute→Execution, Request→Requisition; Update/Control/Exchange/Capture/
Grant stay as-is; Retrieve maps to GET; Notify to subscriptions). The consolidated modern
implementation taxonomy (OpenAPI-era) is **11 primary operations**: Initiate, Register, Evaluate,
Update, Control, Exchange, Execute, Request, Retrieve, Notify, Capture.

---

## 6. Business Objects — the BIAN Business Object Model (BOM)

### 6.1 BIAN's three information descriptions

BIAN deliberately separates *semantic business information* from *physical data formats*. The
standard comprises three related descriptions:

1. **Service Domain Information Model** — the semantic attributes making up each Service
   Domain's information profile (primarily the control record definition);
2. **BIAN Business Vocabulary** — descriptions of the individual information attributes (using
   industry-accepted definitions where available);
3. **BIAN Business Object Model (BOM)** — maps the information attributes to **conceptual
   business objects** for definitional consistency.

### 6.2 The BOM as the "common data language"

The BOM is BIAN's answer to "what does a *Party*, a *Product*, an *Account*, an *Agreement*, a
*Transaction* actually mean across the whole bank?". BIAN defines these as business objects at a
**conceptual level** — e.g., an attribute named `customer reference` means "a unique reference to
a bank customer", but BIAN does **not** dictate whether the implementation uses an IBAN-like code,
a bank-specific key, or a UUID. This is what makes BIAN **implementation- and vendor-agnostic**.

**Mapping to the canonical/universe data models** (see
[`data_model_resource_book_guide.md`](data_model_resource_book_guide.md) and
[`data_models_banking_insurance_guide.md`](data_models_banking_insurance_guide.md)):

| Concept | Silverston universal | BIAN BOM analog | BIAN service domains that own it |
|---|---|---|---|
| The party | **Party** | Party, Customer, Relationship | Party Reference Data Directory, Party Lifecycle Management, Customer Relationship Management |
| The offer | **Product** (as offered) | Product, Product Directory entry, Product instance | Product Directory, Product Design, Customer Offer |
| The contract | **Agreement** | Agreement, Customer Agreement | Customer Agreement, AGREE TERMS-type domains |
| The money container | — | Account, Financial Facility, Position | Current Account, Position Keeping, Savings Account, Deposit Account |
| The movement | **Financial Transaction** | Transaction, Payment, Trade | Payment Execution, Transaction Engine, Trade Settlement, Card Authorization |

**Architect's use of the BOM:** the BOM gives a data architect a ready-made catalog of canonical
entities per functional area. A bank building a canonical data model (CDM) can start from BIAN's
Party/Customer, Account, Product, Agreement, and Transaction business objects as skeletons, then
layer physical attributes. This is exactly the "BIAN as input to CDM" pattern described in
[`data_models_banking_insurance_guide.md`](data_models_banking_insurance_guide.md) §2.1.

### 6.3 Relationship between Service Domains and Business Objects

**Each Service Domain owns (governs) specific business objects** — captured in its control
record. The fundamental design rule is the **encapsulation** of business information:

- The **control record** is the aggregate root — the anchor that business objects relate to
  (e.g., in Account Management the control record is the Account itself).
- Business objects are the "shared vocabulary" passed as **service operation payloads** between
  Service Domains.
- The *internal* data schema of a Service Domain is explicitly **not** standardized — only what
  crosses the service boundary is. This is what enables two implementations of the same Service
  Domain to interoperate while remaining internally different.

This yields "two overlapping views of business information": (a) the high-level conceptual
vocabulary exchanged across service boundaries, and (b) the detailed internal processing
logic/schema. BIAN standardizes (a).

### 6.4 BOM vs ISO 20022

BIAN's policy is **not to develop competing content** with prevailing standards. Where possible it
maps its semantic attributes to the **ISO 20022 Business Model**. Because ISO 20022's business
model has gaps outside payments/securities, BIAN maintains its **own intermediate conceptual
object model (the BOM)** to bridge. This is the intellectual core of the BIAN-BOM/ISO20022
alignment: **BIAN = service/capability architecture + high-level business objects; ISO 20022 =
message-level financial semantics.**

---

## 7. The BIAN Meta Model and Design Artifacts

### 7.1 The meta model

The BIAN **meta model** is a detailed UML model defining all BIAN design structures. Its core
elements:

| Meta model element | Meaning |
|---|---|
| **Business Area** | Highest-level classification (grouping of capabilities with similar support needs) |
| **Business Domain** | Coherent capability collection within an area |
| **Service Domain** | The elemental building block |
| **Business Object / Control Record** | The data the domain owns/governs |
| **Business Behavior (service operations)** | The action-term-based operations the domain exposes |
| **Behavior Qualifier** | Sub-partition of the control record narrowing operation scope |
| **Business Actor / Role** | Who engages the domain (implicit; surfaced in scenarios) |
| **Business Context / Business Event** | The trigger/business situation for service exchanges |

The meta model draws on parts of the **ISO 20022 meta model** and is aligned with **TOGAF**
(there is a published TOGAF↔BIAN mapping; each TOGAF ADM step is annotated with the BIAN
deliverables that support it).

### 7.2 Design artifacts / deliverable set

Beyond the Service Landscape, the standard publishes:

1. **Service Domain specifications** — for each domain: functional pattern, generic artifact,
   control record definition, behavior qualifiers, default service operations, first-order
   connections, information profile.
2. **Business Scenarios** — canonical, cross-domain interaction diagrams showing how Service
   Domains collaborate to deliver an end-to-end business outcome (e.g., mortgage application,
   customer-initiated case, account opening, payments processing). Scenarios reveal the Service
   Domains in play and their message sequence; they are the developer's "how to frame requirements
   as services, not processes" aid.
3. **Wireframes** — diagrammatic views of a Service Domain network for a specific business area;
   the stable "enterprise blueprint" used for migration planning.
4. **The Business Capability Map** — a capability view (what the bank *can do*), complementary to
   the Service Landscape (the organizational capacities / Service Domains that deliver it).
   Tooling such as Ardoq imports both as two workspaces: *BIAN Business Capabilities v12.0*
   (functional breakdown) and *BIAN Business Domains v12.0* (Service Landscape, Business Area L1 /
   Business Domain L2).
5. **The Business Object Model / Information Architecture** (§6).
6. **The BIAN Semantic API specifications** — OpenAPI-format service domain APIs (§9).
7. **Design guidelines / adoption guides** — the "Guide to Adoption", "How-to Guide: Applying
   the Standard", the Semantic API Practitioner Guide.

### 7.3 A worked example — the mortgage application scenario

The canonical "Customer Mortgage Application" business scenario (from the BIAN guide) shows the
component (not process) mindset:

```
Customer Offer  →  Retrieve  Party Reference Data Directory     (check known customer / ref data)
                →  Retrieve  Product Directory                  (get offer-processing rules)
                →  Retrieve  Credit Administration              (current customer credit assessment)
                →  Create    Collateral Asset Administration     (property + valuation record)
                →  Evaluate  Underwriting                        (underwriting decision)
                →  Register  Document Services                   (file offer & related documents)
                →  Initiate  (via current account)               (disburse / fund the facility)
                →  Initiate  Mortgage Loan                        (start the mortgage lifecycle)
```

Each of those calls is a **delegated service exchange** whose *externalization* is justified by
the lifecycle of the target's control record: the customer relationship, the product spec, the
collateral asset, and the loan facility all have **independent lifecycles** governed by their own
Service Domains — so they are not embedded in Customer Offer. This "consider the control record
lifecycle" test is BIAN's core **externalization** decision rule, and it is the same rule used to
map legacy applications onto components.

---

## 8. BIAN Design Principles

The principles below are the foundation of the standard (codified in the official architecture
principles and the practitioner guide):

1. **Service orientation** — banking is modeled as a set of discrete, reusable services
   (Service Domains) exchanging standardized service operations. *Banking = a set of reusable
   services, not a set of siloed processes.*
2. **Business capability focus** — a Service Domain is a *business capability*, not an IT
   system. It can be implemented by people, procedures, and systems in any combination, and is
   sized to be **outsourceable**.
3. **Standardization** — a common vocabulary: MECE Service Domains, a controlled action-term
   vocabulary, standard semantics for business objects, standard API shapes.
4. **Decoupling / loose coupling** — Service Domains interact only through defined service
   operations; internal implementation and data are encapsulated. Encapsulation is the principle
   that preserves decoupling (and is why, as the guide notes, no formal mapping is maintained
   between a domain's offered services and its delegated dependencies).
5. **Reusability** — capabilities are designed for reuse across products, channels, and
   processes (e.g., Credit Administration reused in mortgage offers, product matching, and
   relationship management).
6. **Semantic interoperability** — business information means the same thing everywhere (BOM +
   Business Vocabulary + mapping to ISO 20022).
7. **Technology neutrality** — the model is implementation-agnostic: any stack, any data format,
   any protocol. BIAN defines *semantic dependencies*, not choreography/protocol.
8. **Right-sizing / elemental design** — Service Domains are atomic; they cannot be split. This
   is the property that makes the standard canonical.
9. **Externalization** — determine precisely what belongs inside a Service Domain vs. what is
   delegated, using the control-record-lifecycle test.
10. **Componentization ≠ service enablement** — an often-misunderstood principle: the component
    (capability) view is valuable even if you do *not* service-enable everything. For high-
    throughput back-office transactions, hard-wired point-to-point interfaces between components
    are often better than generalized service calls. BIAN blesses both.
11. **API-first and event-driven-ready** (modern releases) — from 10.0/12.0 onward the standard
    is published API-first (OpenAPI 3.x, then AsyncAPI 3.x) and DDD-annotated.
12. **The "business blueprint" concept** — BIAN is the blueprint connecting *business strategy*
    to *IT architecture*: the capability map lets strategy (enter new country, launch a product
    line) be translated into "which Service Domains must change/be added", and the service
    landscape gives program management a stable decomposition.

---

## 9. The BIAN API Standard — Semantic APIs

### 9.1 What a Semantic API is

A **BIAN Semantic API** = the collection of service operations offered by a **single Service
Domain**, formatted for REST (and now event-driven) implementation. The Service Domain maps to
the application boundary ("A" of API); its service operations are the program interfaces ("PIs").

### 9.2 REST mapping

- CRUD-style mapping: action terms → noun-form endpoints; Retrieve→GET; Initiate/Register/
  Evaluate→POST; Update/Control/Exchange/Execute/Request/Capture→PUT.
- **REST archetypes**: BIAN generic artifacts map to REST resource archetypes — Documents
  (Strategy, Management Plan, Specification, Agreement, Assessment, Analysis, Allocation),
  Controllers (Procedure, Operating Session, Maintenance Arrangement, Fulfilment Arrangement,
  Transaction, Advice, State, Log), Collections (Directory Entry, Membership).
- **Path structure** (canonical, from the official specs):
  `/{ServiceDomainName}/{cr-ref-id}/{BehaviorQualifierName}/{bq-ref-id}/{Operation}`
  e.g. `/MortgageLoan/12345/Collateral/67890/Retrieve`, `/DirectDebit/11111/Initiate`.
- **Endpoints are not implementation specs** — they are "REST-shaped" semantic definitions
  sufficient to switch providers without destabilizing up/downstream dependencies. Banks extend
  them with implementation-specific reference attributes.

### 9.3 The official repository (`github.com/bian-official/public`)

Machine-readable, Apache-2.0-licensed, organized by release and variant:

| Release | Format | Variants | Service-domain API specs |
|---|---|---|---|
| 9.1.0 | Swagger 2.x | early BOM concepts | 186 |
| 10.0.0 | OpenAPI 3.x | BOM (`semantic-apis`) | 244 |
| 11.0.0 | OpenAPI 3.x | BOM | ~245 |
| 12.0.0 | OpenAPI 3.x + **AsyncAPI 3.x** | BOM + **ISO20022+DDD** | 247 |
| 13.0.0 | OpenAPI 3.x + AsyncAPI 3.x | BOM + ISO20022+DDD | 251 |
| **14.0.0** | OpenAPI 3.x + AsyncAPI 3.x | BOM + ISO20022+DDD | **259** |

- **Variant choice:** `semantic-apis` (BIAN's own BOM-extended semantic model) vs
  `apis-iso20022_ext-ddd` (ISO 20022-mapped, DDD-annotated) — choose based on whether you want
  BIAN-native semantics or international standards alignment.
- Sync (OpenAPI) + async (AsyncAPI) + hybrid domains; from 12.0 each domain spec carries DDD
  annotations (CR = "DDD Aggregate", operations = "DDD Service").
- All releases are maintained indefinitely; the **BIAN Portal** (portal.bian.org) is the
  web-based browser with "250+ APIs and more than 5,000 service definitions" available to members
  and non-members.

### 9.4 API governance value

For API governance (see [`spec_driven_development_frameworks_guide.md`](spec_driven_development_frameworks_guide.md)):
BIAN gives you a **naming convention, operation vocabulary, and path structure** out of the box.
A BIAN-aligned API strategy means: your public/internal APIs are named and shaped per Service
Domain standards, your OpenAPI specs are BIAN-conformant, and your API portfolio maps one-to-one
to the capability map — which makes API catalog, capability map, and target architecture the same
picture.

---

## 10. Release Evolution: 8.x → 14.0 and Beyond

| Version | Date | Highlights (verified) |
|---|---|---|
| 8.0 | ~2020 | Matrix + draft Value Chain layouts; ~320 Service Domains; the era of the Practitioner Guide V8.1 |
| 9.1 | ~2021 | Public release via new portal; +40 domains; Swagger 2.x semantic APIs (186 specs); BOM diagrams |
| 10.0 | ~2022 | **OpenAPI 3.x migration**; BOM extension; 244 service-domain APIs; the "10+" API-first direction begins |
| 11.0 | Dec 2022 | **All 322 Service Domains completed**; +5,000 service definitions; +250 Semantic APIs; **Event-Driven Design**; Business Capability Model; Information Architecture/BOM; Coreless Bank pilots; Product Certification; integrated Dev/Ops pipeline; ServiceNow integration; wireframe generation |
| 12.0 | Nov 2023 | **Bank-verified content** (working-group-driven descriptions for Business Capabilities, Scenarios, Business Objects, Service Domains); **AsyncAPI 3.x event specs**; ISO20022+DDD variant; ~247 API specs; Coreless 3.0 scenarios; DDD annotations |
| 13.0 | Jun 2025 | More content/quality; extra **Wireframe Diagrams** as starting points for organization-specific Business Scenarios (from the Coreless 4 PoC); 251 API specs |
| **14.0** | **Feb–Mar 2026** | "**AI-ready**" release: rationalization of Service Domain definitions (unnecessary operations eliminated, some domains renamed/removed/added — especially in payments); **more ISO 20022 links**; three new Insurance Service Domains; expanded standards work into payments and insurance; new domains supporting AI-enabled use cases; updated Behavior Qualifiers |

**Direction of travel:** API-first → event-driven → DDD-annotated → AI-ready. BIAN 10/11+ is
explicitly a cloud-native, API-first, event-driven standard; the old "SOA artifact" framing is
not what current releases ship.

---

## 11. BIAN vs Other Standards — Data Models, ISO 20022, TM Forum, Open Banking

### 11.1 BIAN vs canonical banking data models

See [`data_models_banking_insurance_guide.md`](data_models_banking_insurance_guide.md) §2 for the
full landscape. The one-line distinction:

| Model | Owner | Answers |
|---|---|---|
| **BIAN BOM** | BIAN consortium | "How should banking *capabilities* and the data they exchange be structured?" — service/component architecture + high-level business objects |
| **IBM BDW** | IBM | "How should banking *data* be structured for the warehouse/analytics?" — 3NF enterprise data warehouse model |
| **Teradata FSLDM** | Teradata | Same question as BDW — FSLDM is the financial-services logical data model (warehousing/analytics, 6 subject areas in v11 vs BIAN's service view) |
| **Silverston** | Len Silverston (Universal Data Models) | Generic industry-agnostic universals: Party, Product, Agreement, Financial Transaction… — the "kernel" business objects |

**BIAN = service/capability architecture + data objects; BDW/FSLDM = data warehouse models;
Silverston = universal data models.** Practically: use BIAN for capability decompositions,
service boundaries, and canonical *service payload* objects; use BDW/FSLDM for the analytic
store; use Silverston as the generic reference when building the CDM core. BIAN BOM's Party/
Account/Product/Agreement/Transaction objects line up with Silverston's universals and with the
canonical models' core entities — which is why BIAN maps so cleanly onto CDM programs.

### 11.2 BIAN vs ISO 20022

- **BIAN** = architecture/capability layer ("which service performs a payment, who owns the
  data").
- **ISO 20022** = message layer ("the exact structure of a payment instruction").
- BIAN has a **'category D' liaison with ISO** for ISO 20022 semantic models; it maps its
  information attributes onto the ISO 20022 business model and publishes **ISO20022-extended
  API variants** (since 12.0). Release 14.0 further strengthens ISO 20022 alignment.
- In practice: a BIAN-aligned payment service (see [`payments_hub_guide.md`](payments_hub_guide.md))
  exposes BIAN-style operations (Retrieve/Execute/… on Payment Execution) with ISO 20022 payloads.

### 11.3 BIAN vs TM Forum

TM Forum (telecom) is the closest cross-industry analogue: it defines Open APIs and a business
process/application framework for telecom operators. **TM Forum ~ "the BIAN of telecom"** — same
idea (standardized capability map + Open APIs), different industry. BIAN's Semantic APIs and
TM Forum's Open APIs are conceptually parallel; both are vendor-neutral, member-driven,
framework-style standards.

### 11.4 BIAN vs Open Banking

**BIAN ≠ open banking compliance.** Open Banking (PSD2, Open Banking UK, Singapore's SGFinDex/
MAS ecosystem initiatives) is a *regulatory/commercial API regime* — which APIs must be exposed,
to whom, under what consent/security rules. BIAN is an *architecture standard* — how the bank's
own capabilities are decomposed and interfaced. The relationship:

- BIAN Service Domains provide the **service layer underneath open banking APIs** — the open
  banking API is the channel-facing surface; the BIAN-aligned service layer is what it calls.
- A bank doing both gets: regulatory APIs (open banking) built on standardized internal services
  (BIAN) — no duplication, consistent semantics.
- BIAN's value to open banking: rapid onboarding of third parties onto well-defined, stable
  internal capabilities; and "BIAN Applied to Open Banking" is a recognized BIAN topic.

---

## 12. BIAN in Practice — How Banks Use It

### 12.1 As an architecture blueprint (target architecture mapping)

The Service Landscape becomes the **target capability decomposition**: every current and future
system is located on the map ("which Service Domains does system X cover?"). Programs are
scoped in Service Domain terms ("replace the 6 overlapping systems covering these 4 Service
Domains with one BIAN-conformant platform").

### 12.2 Vendor evaluation

The classic procurement use: **write RFPs in BIAN terms.** "We require these 40 Service Domains
across payments, cards, lending, customer management — map your product to them." Vendors respond
with conformance matrices; you compare coverage, gaps, and overlaps *objectively*. "Can the
vendor cover our required Service Domains?" becomes a first-class evaluation criterion. (See the
RFP/sales-methodology context in [`sales_methodology_frameworks_guide.md`](sales_methodology_frameworks_guide.md).)

### 12.3 Integration design

Service-domain-based integration: interfaces are defined as BIAN service operations between
BIAN-aligned components. First-order connections from the standard give you a reference network
of dependencies; business scenarios give you canonical message flows. New integrations copy
standard patterns instead of inventing new ones.

### 12.4 API strategy

BIAN-aligned APIs (§9.4): naming, operation vocabulary, path structure, OpenAPI specs. The API
catalog becomes a projection of the capability map; API governance and architecture governance
merge.

### 12.5 Organization design ("BIAN-aligned operating model")

Because Service Domains are "assignable business responsibilities", banks align **IT/operations
organization to Service Domains**: platform teams own Service Domains, funding follows Service
Domains, and the org chart mirrors the architecture. This is a recognized, high-leverage adoption
pattern — and also one of the more culturally difficult (see pitfalls, §13.4).

### 12.6 Data architecture

BOM-aligned data models: canonical entities derived from BIAN business objects; data domains
mapped to Service Domains (data ownership = Service Domain ownership of its control record).
Banks building a CDM use the BOM as skeleton and the landscape as the data-domain map (see
[`data_models_banking_insurance_guide.md`](data_models_banking_insurance_guide.md)).

### 12.7 Cloud migration / re-platforming

BIAN as the **target decomposition for cloud-native re-platforming**: the Service Landscape gives
the "service cut" for a modular core; the parallel-core migration technique (§14/§15 of the
practitioner guide) lets you build a BIAN-aligned cloud platform alongside the legacy host, sync
data, and retire the host incrementally. AWS and others publish BIAN-based banking data/architecture
blueprints on the cloud.

---

## 13. The Adoption Journey — Levels, Methodology, Pitfalls

### 13.1 Adoption levels

BIAN's own guidance describes a maturity progression (consistent with the industry's "Level 0–4"
framing):

| Level | Name | What happens |
|---|---|---|
| **0** | Awareness | Understand BIAN, its artifacts, and business case; leadership buy-in |
| **1** | Mapping | Map existing systems/applications to BIAN Service Domains; build the current-state capability map |
| **2** | Alignment | Align new interfaces/APIs and integration to BIAN service operations and semantics; BIAN-shaped API standards |
| **3** | Transformation | Restructure architecture (and organization) around Service Domains; start componentizing/replacing legacy |
| **4** | BIAN-native | New capabilities are built as BIAN Service Domains from the start; composable, event-driven, API-first |

### 13.2 Adoption methodology

The canonical path: **assess → map → pilot → standardize → scale**

1. **Assess** — pick a business area with pain (integration cost, vendor lock-in, slow change);
   educate stakeholders (business + IT).
2. **Map** — map the current application portfolio to Service Domains; produce the current-state
   landscape and gap view.
3. **Pilot** — run a **focused pilot in one business area** (Customer Onboarding is the classic
   starter: it is bounded, cross-domain, and strategically visible). Build the pilot per BIAN:
   service operations, BQ-scoped payloads, BIAN-shaped APIs.
4. **Standardize** — convert pilot learnings into **internal BIAN standards**: naming,
   API patterns, data conventions, governance process, RFP language.
5. **Scale** — roll out area by area; iterate legacy wrapping/migration against the stable
   blueprint; institutionalize governance.

### 13.3 Typical timeframes

- Mapping an application portfolio to BIAN: weeks–months.
- A credible pilot: 3–12 months.
- Full BIAN-aligned transformation of a large bank's core: **multi-year (3–7+ years)** program —
   which is why the *stable landscape* matters (the blueprint does not move while the program runs).

### 13.4 Common adoption pitfalls

| Pitfall | Description / antidote |
|---|---|
| **BIAN as documentation exercise** | Map everything, then file it. Antidote: tie adoption to funded change (a pilot with measurable outcomes) — "map *and* transform". |
| **Over-standardization / forcing legacy into BIAN** | Trying to make every legacy system "conform" rather than mapping it as a provider of multiple Service Domains via wrapping. Antidote: wrap first, conform later; use the legacy-wrapping patterns (§12 of guide). |
| **Under-governance** | No architecture authority to arbitrate Service Domain ownership, naming, and API conformance. Antidote: a standing BIAN governance board with teeth. |
| **Missing tooling support** | Hand-maintained maps rot. Antidote: use the model browser/portal, import the model into EA tooling (Ardoq, Sparx, LeanIX, ServiceNow), keep the landscape as a governed model. |
| **IT-only initiative** | BIAN fails when IT adopts it and the business doesn't. Antidote: business involvement in capability mapping and operating-model design; the "aligned operating model" is a business change. |
| **Skipping the BOM/data side** | Using BIAN only for API naming while data stays chaotic. Antidote: adopt the BOM alongside — semantic consistency is the point. |

---

## 14. BIAN and Cloud-Native / Composable Banking

### 14.1 BIAN as the blueprint for composable banking

The composable banking debate (see [`core_banking_systems_guide.md`](core_banking_systems_guide.md)):
banking should be assembled from modular, replaceable business capabilities rather than one
monolithic core. BIAN is, in effect, **the industry's pre-built capability decomposition for
composable banking** — its Service Domains are a ready-made list of candidate packages.

### 14.2 Service Domain → microservice mapping

The practical, widely-used mapping:

| BIAN concept | Implementation |
|---|---|
| **Service Domain** | **Microservice / bounded context** (each Service Domain = one deployable, business-aligned service) |
| **Control Record (CR)** | The microservice's aggregate root / primary data entity |
| **Behavior Qualifier (BQ)** | Sub-aggregate / sub-resource within the service |
| **Service Operation** | **API endpoint** |
| **Business Object** | The data model / schema |
| **First-order connection** | Service-to-service call / event dependency |

This is the "service domain as DDD bounded context" framing the BIAN repository itself uses from
12.0 onward (explicit DDD annotations: CR = Aggregate, operations = DDD Services).
**Benefits:** business-aligned microservices (not tech-sliced), standardized interfaces,
vendor-neutral, and a migration path (legacy wrapped as service providers; new capabilities built
BIAN-native).

**Caveat (honest):** 300+ fine-grained services is too many to run as microservices directly.
Sensible practice: **cluster** Service Domains into deployable units (a "product" or "payment"
group of 3–8 domains per service), keep the BIAN domain boundaries inside/at the API layer, and
apply your own sizing rules. DeepWiki's own docs caution that the 98+ API spec count is the
repository's current published set — the *landscape* has ~320+, and you will not implement all of
them.

### 14.3 BIAN + event-driven architecture

- Since 11.0/12.0 BIAN supports **event-driven design**: Service Domains emit and consume
  events; the standard publishes AsyncAPI 3.x specs for event channels.
- The practitioner guide describes the event-driven SOA vision: state changes cascade through
  Service Domains ("referential dependencies" + "service domain events") until a stable state is
  reached; exchanges must be **idempotent and commutative**; defensive operations required.
- For an architect: BIAN gives the *event vocabulary* (which domain owns which state changes,
  what the events mean) — the plumbing is your event backbone (Kafka etc., see
  [`event_stream_processing_guide.md`](event_stream_processing_guide.md)).

### 14.4 BIAN + APIs + OpenAPI

Covered in §9 — BIAN is effectively a **spec-driven development framework for banking** (see
[`spec_driven_development_frameworks_guide.md`](spec_driven_development_frameworks_guide.md)):
standardized OpenAPI specs per service domain, generated from the model, so code, docs, and tests
all derive from the standard.

### 14.5 The "Coreless Banking" initiative

BIAN's flagship proof-of-concept movement: a consortium of banks and vendors co-developing
**compatible, BIAN-based banking microservices** to replace the monolithic core ("coreless" =
the core is no longer a single platform). Timeline: initial PoCs (savings/loans/checking APIs) →
**Coreless 3.0** business scenarios in SL 12.0 → **Coreless Banking 4.0 (Oct 2024)** adding AI
for customer retention and personalization, running with **Wells Fargo, Bangkok Bank, Bantrab,
IBM, Salesforce, Zafin, TCS, Temenos** among others; its wireframes feed SL 13.0.

---

## 15. BIAN and Core Banking Vendors

### 15.1 The vendor value proposition

"**BIAN-aligned**" is an explicit selling point: it signals composable architecture, standardized
integration, and lower integration cost. Vendors map their product suites to BIAN Service Domains
(BOM mapping) and publish BIAN conformance/alignment assessments; some undergo BIAN product
certification (introduced around SL 11.0). For the buyer, this turns vendor selection into a
coverage comparison on the same map (§12.2).

### 15.2 Vendor alignment snapshot

| Vendor / product | BIAN posture (verified directions) |
|---|---|
| **Temenos** (Transact) | **BIAN member; actively BIAN-aligned** — publishes Transact↔BIAN Service Domain mappings; participant in Coreless Banking 4.0 (with TCS). One of the most explicit BIAN-aligned core vendors. |
| **Oracle** (FLEXCUBE) | Publishes **OBMA (Oracle Banking Microservices Architecture)** with BIAN mapping and a BIAN-aligned service decomposition — see [`oracle_banking_microservices_architecture_guide.md`](oracle_banking_microservices_architecture_guide.md). |
| **SAP** | BIAN member (long-standing; SAP community publishes BIAN positioning); SAP Banking Services aligned with BIAN-era service thinking. |
| **Mambu** | Cloud-native, API-first composable banking platform; API-native rather than BIAN-certified, but its capability model maps readily onto BIAN domains. |
| **Thought Machine (Vault)** | Explicitly **less BIAN-aligned**: Vault is API-native with its own model (product logic "Vault Core", no BIAN conformance emphasis). Fine to use; you do the mapping to BIAN yourself. |
| **Finastra** | Member; maps Fusion product lines to BIAN (FusionFabric.cloud APIs are BIAN-aligned in parts). |
| **FIS** | Member; maps core (Profile, DNA) and digital products to BIAN domains; also markets the FIS Banking Data Model. |
| **IBM** | BIAN-aligned references (e.g., Banking Process and Service Models aligned to the BIAN Service Landscape), participant in Coreless 4.0. |
| **Microsoft** | Publishes BIAN-based industry reference architectures for banking (and AWS publishes BIAN-based banking data strategies). |

**Buyer's rule:** treat "BIAN-aligned" claims as a mapping you can verify — ask for the vendor's
Service Domain coverage matrix, BOM mapping, and API conformance evidence. The *map* is the
contract.

---

## 16. The BIAN Role in a Bank Architect's Toolbox

How a practicing banking architect (the audience of this repo) uses BIAN day-to-day:

1. **Target architecture** — the Service Landscape as the target blueprint: every domain,
   system, and data domain gets located on it; the target state is expressed as a subset of
   Service Domains with owners.
2. **API governance** — BIAN-aligned naming and operations: endpoints are
   `/{ServiceDomain}/{cr-id}/{BehaviorQualifier}/{bq-ref}/Operation`; operational vocabulary is
   the action-term set; OpenAPI specs derive from the model.
3. **Integration standards** — service-domain-based integration: define interfaces between
   BIAN-aligned components, reuse business scenarios / first-order connections as reference
   flows instead of bespoke point-to-point design.
4. **Data architecture** — BOM-aligned: canonical entities from BIAN business objects; data
   ownership aligned to Service Domain control records.
5. **Vendor management** — BIAN mapping in RFPs: capability coverage matrices, conformance
   assessment, gap analysis (buy vs build vs partner).
6. **Roadmap** — the "BIAN-aligned core": the long-term roadmap runs legacy → wrapped legacy →
   BIAN-shaped components → BIAN-native services, executed incrementally against the stable
   blueprint (parallel-core migration).
7. **Operating model** — BIAN-aligned org: platform/domain ownership aligned to Service Domains.

**BIAN in the architecture roadmap** — the honest sequence most banks follow:
- Year 0–1: landscape + portfolio mapping; pick pilot area.
- Year 1–3: pilot APIs/integration on the BIAN pattern; internal standards; begin wrapping the
  worst legacy around Service Domain boundaries.
- Year 3+: BIAN-aligned platforms replace hosts area by area; new-build capabilities are
  BIAN-native; the org follows.

---

## 17. Singapore / Asia Context

- **Adoption in Asia:** BIAN's membership is global but historically European/North American-led;
  Asian adoption is visible among large regional banks and their architecture practices. SG banks
  (DBS, UOB, OCBC) pursue **BIAN-aligned initiatives** in the sense of capability-based
  architecture, microservices decomposition, and API standardization — some explicitly reference
  BIAN in their architecture practices. There is active BIAN interest in the SG banking
  community (architecture practice, vendor-partner work in the region).
- **MAS and BIAN:** there is **no direct MAS mandate for BIAN**. BIAN is voluntary. MAS's
  technology-risk focus is **TRM** (Technology Risk Management) and **FEAT** (Fairness, Ethics,
  Accountability, Transparency) principles — see [`financial_risk_compliance_systems_guide.md`](financial_risk_compliance_systems_guide.md).
  BIAN's relevance to MAS-regulated banks is *indirect*: a clean, well-governed architecture
  (which BIAN supports) makes TRM/FEAT compliance and audit easier.
- **Practical SG angle:** BIAN's value locally is (a) vendor-neutral architecture dialogue in a
  region where banks run many vendor cores, (b) a stable map for cloud/host migration, and
  (c) a common language for ecosystem/embedded-finance initiatives.

---

## 18. Getting Started with BIAN

1. **Explore free material** — the Service Landscape is publicly browsable: bian.org →
   Deliverables → Service Landscape; the InSite portal (e.g. `bian.org/servicelandscape-13-0-0/`);
   the **BIAN Portal** (portal.bian.org) with 250+ APIs / 5,000+ definitions; the official
   **GitHub repo** (bian-official/public, Apache 2.0) — clone `release14.0.0/` and inspect the
   OpenAPI/AsyncAPI YAMLs.
2. **Read the two key guides** — the *BIAN Semantic API Practitioner Guide* and the *Guide to
   Adoption* (both on bian.org).
3. **Pilot** — map one business area (Customer Onboarding is the classic) onto Service Domains;
   implement one scenario (mortgage application is the canonical worked example) as BIAN-shaped
   APIs.
4. **Training & certification** — the **BIAN Banking Architecture Foundation Certification**
   (delivered via Van Haren Certify): 60 multiple-choice questions / 1 hour / 70% pass; valid
   2 years; no prerequisites. There is also a v3 refresh of the Foundation certification (2026).
5. **Join / community** — membership tiers for banks, vendors, consultancies; **18 active
   working groups** (content, APIs, payments, insurance, coreless, architecture…); annual
   conference, webinars, hackathons, and the **BIAN Transformation Awards** (2025 winners
   announced Nov 2025); the BIAN LinkedIn group.
6. **Tooling** — the model browser (InSite), BIAN Portal APIs, the GitHub JSON/XML/YAML model,
   EA-tool imports (Sparx, Ardoq, LeanIX, ServiceNow integration from 11.0), BIAN Services GmbH
   (training/consulting arm).

---

## 19. The Future of BIAN (2026+) — and the Honest Relevance Question

### 19.1 Direction (verified from BIAN's own 2026 activity)

- **AI/GenAI** — SL 14.0 is explicitly "AI-ready": new Service Domains for AI-enabled use cases,
  updated behavior qualifiers for AI-orchestrated workflows; BIAN positioning as the blueprint
  for integrating AI into banking operations (see the repo's LLM/AI guides for the broader
  context, e.g. [`on_prem_llm_deployment_guide.md`](on_prem_llm_deployment_guide.md) — BIAN is
  the capability/taxonomy layer, not the AI stack itself).
- **Cloud-native** — service domains as cloud-native microservices remain the coreless banking
  thrust (Coreless 4.0+ with AI); wireframes and PoCs feed each release.
- **Broader scope** — formal expansion into **payments and insurance** (3 new insurance domains
  in 14.0) toward "a unified reference model spanning the financial services ecosystem".
- **ISO 20022** — deepened alignment (more links in 14.0; ISO20022-extended API variants are
  now the primary published variant).
- **API standardization** — 259 OpenAPI/AsyncAPI specs in 14.0, all releases published and
  continuously improved.
- **Embedded finance / ecosystem banking** — BIAN Service Domains as the API layer for embedded
  banking (banking-as-a-service): the capability map provides the stable service surface that
  embedded/ecosystem APIs call into (see [`core_banking_systems_guide.md`](core_banking_systems_guide.md)
  on embedded finance).
- **Adoption growth** — digital banks and cross-industry ecosystems are natural BIAN consumers.

### 19.2 The relevance question: future standard or legacy SOA idea?

Honest assessment:

- **The value is the map and the vocabulary.** The **Service Landscape is the most complete,
  vendor-neutral banking capability map that exists** (300+ MECE service domains, refined by a
  global consortium over nearly two decades). That value is *independent* of the SOA-era framing
  and survives any technology change. The BOM + action-term vocabulary give you a canonical
  language for capability, API, and data discussions that nothing else provides at this scale.
- **The risk is SOA-era formalism.** The deep meta-model (functional patterns, generic
  artifacts, fractal qualifiers, 19 patterns × 17 action terms) is heavy. Banks that adopt BIAN
  as a religion — full classification, full conformance, no pragmatism — burn time and credibility.
  And a 300+ domain model is *not* a microservice topology; naive one-domain-per-microservice
  adoption is a well-known misread.
- **The synthesis (recommended):** **use BIAN as the capability map and vocabulary; implement
  pragmatically.** Service Domains → bounded contexts → (clustered) microservices; adopt the
  API naming and operation vocabulary; use the BOM to seed your canonical data model; don't
  certify-conform everything. BIAN gives banking architects what "bounded context" alone cannot:
  an industry-agreed set of context boundaries to start from. That is a future-proof asset
  regardless of whether the next wave is microservices, event-driven, or AI-orchestrated.

---

## 20. Key Takeaways

1. **BIAN is a standard, not software** — a member-owned, not-for-profit association
   (est. 2008, Frankfurt) publishing an open architectural framework for banking.
2. **The core artifact is the Service Landscape** — ~320–326 MECE Service Domains covering the
   whole bank, organized (in two interchangeable layouts: matrix and value chain) under Business
   Areas → Business Domains → Service Domains.
3. **The Service Domain is the building block** — functional pattern + asset type + control
   record + behavior qualifiers + standard service operations + first-order connections.
4. **The vocabulary is controlled** — 19 functional patterns, ~17 action terms (modern API
   taxonomy: 11 primary operations), 250–300 asset types.
5. **The BOM is the data language** — conceptual business objects (Party, Account, Product,
   Agreement, Transaction) that map to ISO 20022 where possible and to the universal/canonical
   data models (Silverston, BDW, FSLDM) for CDM work.
6. **It is API-first and event-driven today** — 259 OpenAPI 3.x + AsyncAPI 3.x service-domain
   specs in release 14.0 (Feb 2026), with BOM and ISO20022+DDD variants, all on GitHub (Apache 2.0).
7. **Adoption is a journey** — assess → map → pilot → standardize → scale; Levels 0–4; multi-year
   for a big bank; the classic failures are map-only adoption, over-standardization, under-
   governance, missing tooling, and IT-only sponsorship.
8. **It is a vendor lens** — "BIAN-aligned" is the industry's integration selling point
   (Temenos most explicitly; Oracle OBMA, Finastra, FIS, IBM, SAP; Thought Machine's Vault the
   notable non-aligned, API-native choice).
9. **It is not open-banking compliance and not ISO 20022** — it is the capability/service layer
   that makes those regimes coherent.
10. **Relevance survives** — even if the "SOA" label is dated, the capability map + vocabulary
    is the industry's best shared blueprint for composable, cloud-native, AI-ready banking.

---

## 21. References and Further Reading

### Official sources
- BIAN website: bian.org (Membership, Deliverables, News Room, FAQs)
- **Service Landscape page** (release history 11.0→14.0): bian.org/deliverables/service-landscape/
- **InSite portal v13**: bian.org/servicelandscape-13-0-0/ (v12: .../servicelandscape-12-0-0/)
- **BIAN Portal** (APIs & model browser): portal.bian.org
- **Official GitHub repo**: github.com/bian-official/public (API specs, all releases, Apache 2.0)
- **BIAN Semantic API Practitioner Guide V8.1** (PDF on bian.org) — the authoritative design
  deep-dive (functional patterns, action terms, control records, REST mapping, implementation
  approaches, legacy wrapping, parallel core)
- **BIAN Guide to Adoption**; **Coreless Banking** deliverables
- BIAN news: "BIAN Unveils New Service Landscape 14.0 to Accelerate AI-Ready Banking
  Architecture" (Mar 2026); "BIAN advances Coreless Banking initiative… AI" (Oct 2024)
- Wikipedia: "Banking Industry Architecture Network" (est. 2008, Frankfurt, mission, metamodel,
  certification details)

### In this repository (cross-references)
- [`data_models_banking_insurance_guide.md`](data_models_banking_insurance_guide.md) — BIAN as a
  canonical banking model, alongside IBM BDW, Teradata FSLDM, FIS, SAS, Oracle
- [`data_model_resource_book_guide.md`](data_model_resource_book_guide.md) — Silverston
  universals (Party, Product, Agreement, Financial Transaction) ↔ BIAN BOM
- [`payments_hub_guide.md`](payments_hub_guide.md) — payments architecture; BIAN payment service
  domains and ISO 20022
- [`core_banking_systems_guide.md`](core_banking_systems_guide.md) — composable banking, core
  system replacement, embedded finance
- [`spec_driven_development_frameworks_guide.md`](spec_driven_development_frameworks_guide.md) —
  OpenAPI/spec-driven API practice that BIAN APIs plug into
- [`event_stream_processing_guide.md`](event_stream_processing_guide.md) — event plumbing for
  BIAN event-driven design
- [`oracle_banking_microservices_architecture_guide.md`](oracle_banking_microservices_architecture_guide.md) —
  Oracle OBMA BIAN mapping
- [`financial_risk_compliance_systems_guide.md`](financial_risk_compliance_systems_guide.md) —
  MAS TRM/FEAT context (BIAN is voluntary)
- [`on_prem_llm_deployment_guide.md`](on_prem_llm_deployment_guide.md) — AI deployment context
  for BIAN's AI-ready direction

---

