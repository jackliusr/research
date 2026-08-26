# The Transaction Engine in BIAN — The Transaction-Processing Service-Domain Deep-Dive

> **Scope:** This guide is the **dedicated deep-dive on the Transaction Engine service
> domain** in BIAN (the Banking Industry Architecture Network) and on the service-domain view
> of transaction processing: what the Transaction Engine *is* in the BIAN landscape (verified
> from bian.org and the official API repository), where the transaction-related service
> domains sit in the Service Landscape, the Transaction Engine's anatomy (control record,
> functional pattern, operations — and the honest flags where the expected qualifiers do not
> exist), the transaction lifecycle (initiation → authorisation → execution → clearing →
> settlement), the transaction types (funds transfer, card, FX), the service-domain
> orchestration around the engine (product domains, channels, rules), the transaction-to-
> posting flow, the Cymbal Bank banking context, a worked BIAN-aligned transaction-engine
> design, and the one-page summary — **the transaction at the centre**.
>
> **How it relates to the BIAN pair already in the repo:** the umbrella
> [`bian_banking_architecture_guide.md`](bian_banking_architecture_guide.md) is the framework
> guide (the Service Landscape, the service-domain model, the action terms, the BOM, the API
> standard, release evolution, adoption) — read it first. The companion
> [`bian_cash_management_domains_guide.md`](bian_cash_management_domains_guide.md) is the
> cash-management-corner deep-dive (the account/payment/cash domains, the verified r14 SD
> catalog). **This guide deepens the *transaction-processing* corner**: the engine that
> schedules, orchestrates, and executes the movements those domains initiate —
> cross-referencing the payments cluster ([`payment_rails_guide.md`](payment_rails_guide.md),
> [`payments_hub_guide.md`](payments_hub_guide.md),
> [`nets_software_systems_guide.md`](nets_software_systems_guide.md),
> [`iso_20022_core_processes_guide.md`](iso_20022_core_processes_guide.md)) and the
> accounting pair ([`posting_engine_core_banking_guide.md`](posting_engine_core_banking_guide.md),
> [`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md)) for the
> transaction-to-posting seam.
>
> **Verification convention (consistent with the repo series):** ✅ = verified this pass
> against primary sources (bian.org InSite Service Landscape v12/v13 pages — the Transaction
> Engine SD pages — and the official `github.com/bian-official/public` repository — the
> release 13.0 and 14.0 OpenAPI 3.x ISO20022+DDD service-domain specs); ⚠ = flagged
> (inferred, single-source, or not re-verifiable this pass — the web search backend degraded
> mid-pass); **unmarked** = structural/industry knowledge presented as such. **Where a
> task-expected name, code, or qualifier turned out not to exist in BIAN's published
> landscape, this guide says so plainly and names the real BIAN element instead** — nothing
> here is fabricated.
>
> **Audience:** Solution/enterprise architects in banking (Jack Liu, Cymbal Bank) who need
> the *actual* BIAN transaction-processing service domains — names, control records,
> operations, qualifiers, positions — to build a BIAN-aligned transaction engine or to
> evaluate vendor BIAN claims.

---

## Table of Contents

1. [The Overview — the Transaction Engine in the BIAN Landscape](#1-the-overview--the-transaction-engine-in-the-bian-landscape)
2. [The BIAN Landscape — the Transaction-Related Service Domains](#2-the-bian-landscape--the-transaction-related-service-domains)
3. [The Transaction Engine Domain](#3-the-transaction-engine-domain)
4. [The Transaction Lifecycle](#4-the-transaction-lifecycle)
5. [The Transaction Types](#5-the-transaction-types)
6. [The Service-Domain Orchestration](#6-the-service-domain-orchestration)
7. [The Transaction-to-Posting Flow](#7-the-transaction-to-posting-flow)
8. [The Banking Context — the Cymbal Bank Transaction Estate](#8-the-banking-context--the-cymbal-bank-transaction-estate)
9. [The Worked Example — a BIAN-Aligned Transaction-Engine Design](#9-the-worked-example--a-bian-aligned-transaction-engine-design)
10. [The Summary — One Page](#10-the-summary--one-page)
11. [Glossary](#11-glossary)
12. [References and Further Reading](#12-references-and-further-reading)
13. [The Verification Ledger — Verified vs Flagged](#13-the-verification-ledger--verified-vs-flagged)

### Reading Map — how this guide connects to the series

| This guide's section | Umbrella anchor (read first) | Sibling anchors |
|---|---|---|
| §1 Overview | umbrella §3 (landscape), §6.2 (BOM: Financial Transaction) | CMAS guide §1 (the cluster), §3.2 (payment table) |
| §2 Landscape | umbrella §3.1–3.4, §10 (releases) | CMAS guide §3.1–3.4 (payment domains; r14 rationalization) |
| §3 Transaction Engine domain | umbrella §4 (SD anatomy), §5 (action terms), §9 (APIs) | CMAS guide §3.2 (the row), §6.2 (BQ table) |
| §4 Lifecycle | umbrella §5 (operations), §11.2 (ISO 20022) | payments_hub §4 (state machine); iso_20022 §4–§7 |
| §5 Transaction types | umbrella §3.3 (Payments/Cards rows), §6.2 (BOM movements) | payment_rails §2; nets §2 (ISO 8583 flows) |
| §6 Orchestration | umbrella §4.1, §7.3 (delegated exchanges), §13 (adoption) | CMAS §3.3 (payment chain); banking_limits_domain_guide |
| §7 Transaction-to-posting flow | umbrella §6 (Transaction vs Account/Position) | posting_engine §2–§4; posting_rules §2, §9 |
| §8 Banking context | umbrella §15 (vendors), §17 (Singapore) | nets §2–§3; payment_rails §7–§8; posting_rules §11 |
| §9 Worked example | umbrella §7.3, §13.1 (adoption levels) | CMAS §8 (worked design); payment_rails §8 |
| §10 Summary | umbrella §20 (key takeaways) | CMAS §9 (one-page pattern) |
| §11–§13 | umbrella §21 (references) | the claims-ledger convention of the series |

**The one-paragraph orientation:** the umbrella gives you the framework — MECE service
domains, control records, action terms, behavior qualifiers, semantic APIs. The CMAS guide
gives you the cash-management corner. This guide answers the *transaction-processing*
questions a banking architect actually asks: *what exactly is the BIAN Transaction Engine —
the real name, the control record, the operations, the qualifiers (or the honest absence of
them)? Where do the transaction-related service domains sit? Which BIAN domains own each
stage of a transaction's life? How does a BIAN-aligned transaction flow become postings?*
The honest headline, verified below: **the Transaction Engine is a real BIAN service domain —
a FULFILL-pattern scheduling/execution utility that product-fulfillment domains delegate
their scheduled payment transactions to — and it has no published behavior qualifiers, no
official code, and a role text that says more about product fulfillment than about a
"transaction processing hub"**. This guide documents exactly that, and maps the rest of the
transaction estate around it.

---

## 1. The Overview — the Transaction Engine in the BIAN Landscape

### 1.1 The definition, verified

**The Transaction Engine is a real BIAN service domain.** Verified this pass against three
independent primary sources that agree:

- **bian.org InSite, Service Landscape v13.0** — the SD Overview page (`view_54373`) lists
  the meta-model attributes (✅): `«ServiceDomain» Transaction Engine`,
  `«AssetType» Transaction Schedule`, `«ControlRecord» Transaction Schedule Facility`,
  `«FunctionalPattern» Fulfill`, `«GenericArtifact» Facility`, `«BehaviorQualifierType»
  Fulfillment`, `«BehaviorQualifier» NA`, and the service groups `Transaction Engine_SD_
  Operations`, `Transaction Schedule Facility_Instantiation` / `_Invocation` / `_Reporting`.
  It appears on the **Clearing And Settlement** view of the Payments area.
- **bian.org InSite, Service Landscape v12.0** — the SD documentation page (`view_118013`)
  states the role (✅): *"The Service Domain provides a utility/background operational support
  service to orchestrate a schedule of payment transaction and reporting activities for the
  fulfillment of certain long term instruments or structured facilities. The Service Domain
  maintains a link to the associated fulfillment Service Domain and the applicable product
  instance for reporting and processing issues that may arise."* The example of use is
  explicit (✅): *"The mortgage fulfillment Service Domain delegates repayment processing to
  the Transaction Engine for its active mortgage product instances."* Key features (✅):
  establish the processing schedule for a product instance; process transaction and reporting
  tasks; escalate issues to the product fulfillment Service Domain; report on fulfillment
  activity.
- **bian.org InSite, Service Landscape v8.0** — the origin note (✅): *"…can service enable
  the structured cash flows that many products might wish to execute. It can be implemented
  to execute the financial transactions, or simply notify the calling service center of the
  need to do so… an anticipated service capability that has yet to be confirmed in use (it
  could be used in structured loan and mortgage fulfillment for example)."*
- **The official API repository** (`github.com/bian-official/public`) — `TransactionEngine.yaml`
  exists in **both** `release13.0.0` and `release14.0.0` under `apis-iso20022_ext-ddd/oas3/
  yamls/` (✅, verified by listing and fetch this pass): *"…Orchestrate a schedule of payment
  transaction and reporting activities for the fulfillment of certain long term instruments
  or structured facilities."* The tag set is a single CR aggregate: `CR (DDD Aggregate) -
  TransactionScheduleFacility`.

**What that definition means, read carefully:** the Transaction Engine is **not** the
posting engine, **not** the payment instruction validator, and **not** a generic
"transaction hub" domain. In BIAN's own words it is a **utility/background operational
support service**: it holds a *schedule* (the `Transaction Schedule` asset type, the
`TransactionScheduleFacility` control record) of payment-transaction and reporting activities
that long-term instruments or structured facilities need executed on their behalf, runs them
when due, and escalates back to the owning fulfillment domain when something goes wrong. The
v8 note even says it may only *notify* the caller that a transaction is due — execution can
be delegated onward or the engine can hand back. The BOM position agrees (umbrella §6.2, ✅):
the **Financial Transaction** business object ("the movement") is owned by *"Payment
Execution, Transaction Engine, Trade Settlement, Card Authorization"* — BIAN itself
classifies the Transaction Engine among the movement domains, alongside the payment
execution and card authorization domains this guide cross-references.

**The honest naming correction (verified):** the task brief expected a "Transaction Engine"
whose qualifiers cover *transaction initiation / execution / notification / update*. The
verified reality: the engine's behaviors are **operations on the control record** (`Initiate`,
`Update`, `Control`, `Exchange`, `Execute`, `Retrieve` — §3), **not behavior qualifiers** —
the v13 landscape explicitly lists `«BehaviorQualifier» NA`, and the OpenAPI spec has no BQ
paths. Initiation, execution, and reporting are the *service groups* and *operations* of the
domain, and the "notify the calling service center" behavior is a documented design option,
not a named qualifier (§3.3 flags this precisely).

### 1.2 The overview table — aspect / description

| Aspect | Description (verified) |
|---|---|
| **What the Transaction Engine is in BIAN terms** | A real service domain (✅, r9.1–r14, retained in r14): a **FULFILL-pattern utility** that "orchestrates a schedule of payment transaction and reporting activities for the fulfillment of certain long term instruments or structured facilities" (✅ bian.org v12/v13) |
| **The control record** | `TransactionScheduleFacility` (✅ v13 InSite + r13/r14 OpenAPI specs) — the schedule for a product instance is the aggregate root |
| **The asset type** | `Transaction Schedule` (✅ v13 InSite) — the domain acts on schedules, not on accounts |
| **The functional pattern** | **Fulfill** / Generic Artifact **Facility** / BQ type **Fulfillment** (✅ v13 InSite) — an arrangement-style domain, not a process or an operating session |
| **The behavior qualifiers** | **None published** — `«BehaviorQualifier» NA` (✅ v13 InSite); the spec exposes only the CR aggregate (✅ r13/r14 yamls). Flagged: the expected initiation/execution/notification *qualifiers* do not exist; those behaviors are *operations* (§3.3) |
| **The service operations** | `Initiate` (POST), `Update` / `Control` / `Exchange` / `Execute` (PUT), `Retrieve` (GET) — 6 paths, identical in r13 and r14 (✅ yamls) |
| **The position in the landscape** | Payments area → the **"Clearing And Settlement"** view (✅ v13 InSite), alongside Order Allocation, ACH Operations, Payment Order, Correspondent Bank Operations, Payment Execution, Cheque Processing, Payment Instruction, the card settlement domains, Payment Rail Operations |
| **The role in the estate** | The **scheduling/execution utility behind product fulfillment**: product domains (loan, mortgage, structured facilities) delegate their scheduled transaction runs to it; it executes them or notifies the caller (✅ v8–v13 role texts) |
| **What it is not** | Not the posting engine (§7), not a payment initiation domain (Payment Order / Payment Order Initiation), not the rail connectivity (Financial Gateway / Payment Rail), not the authorisation domain (Transaction Authorization, §4) |
| **Why it matters** | BIAN's answer to "who runs the scheduled movements that products generate?" — standing-order runs, loan repayments, structured-facility cash flows — and the domain where a bank's scheduling/execution capability maps onto the BIAN map (cross-ref [`../technology/capacity_sizing_guide.md`](../technology/capacity_sizing_guide.md) for the throughput view) |

> **Cross-ref:** the Service Landscape hierarchy and "the Service Domain set is canonical,
> the grouping is a view" are umbrella **§3.1–3.2**; the service-domain anatomy is umbrella
> **§4.1**; the BOM's Financial Transaction object is umbrella **§6.2**; the action terms are
> umbrella **§5**; the Semantic API standard is umbrella **§9**; the r14 payments
> rationalization is CMAS **§3.4**.

---

## 2. The BIAN Landscape — the Transaction-Related Service Domains

### 2.1 The catalog, verified

The task asked to verify the transaction-related service domains and their position in the
BIAN catalog. Verified result: **there is no dedicated "Transaction" business area, but the
transaction-processing estate is a coherent, verified set of service domains** clustered
under the **Payments** area (and its neighbours):

- **The Payments area — the "Clearing And Settlement" view (v13, ✅ bian.org InSite
  `view_97164`/`view_102450`):** the view documentation reads *"Clear and settle market
  trades, allocate orders, maintain counterparty and correspondent details. Handle check
  processing, correspondent bank and ACH interactions and payment fulfillment activities."*
  The view contains exactly these 14 service domains (✅, both the v12.0 and v13.0 overview
  diagrams): **Order Allocation, ACH Operations, Payment Order, Transaction Engine,
  Correspondent Bank Operations, Correspondent Bank Directory, Counterparty Administration,
  Payment Execution, Cheque Processing, Payment Instruction, Card Financial Settlement, Card
  Clearing, Card eCommerce Gateway, Payment Rail Operations.**
- **The r14 changes (✅ CMAS guide §3.4, re-verified against the r14 API listing):** the
  release-14.0 rationalization removed `Payment Execution`, `Payment Instruction`, and
  `Payment Order` and added **Payment Order Initiation** (CR `PaymentOrderInitiation
  Transaction`, BQs `Compliance`/`Confirmation`/`OrderInitiation`), **Payment Rail** (CR
  `PaymentRailOperatingSession`, BQs `InboundTransaction`/`OutboundTransaction`/
  `PaymentAccountReconciliation`/`PaymentClearingandSettlement`), and **Payment
  Orchestration** (portal ✅, not in the oas3 set ⚠). **The Transaction Engine is retained
  in r14** (✅ `TransactionEngine.yaml` present in `release14.0.0`).
- **The wider transaction estate (✅ verified names, umbrella §3.3 + CMAS §2–§4):** the
  account/position spine (**Current Account, Corporate Current Account, Virtual Account,
  Internal Bank Account, Position Keeping, Account Reconciliation**); the instrument domains
  that *generate* transactions (**Standing Order, Direct Debit, Direct Debit Mandate,
  Disbursement, Cheque Processing, Cash Concentration, Currency Exchange**); the card
  cluster (**Card Authorization, Card Capture, Card Clearing, Card Financial Settlement,
  Card Transaction Switch, Card eCommerce Gateway, Merchant Relations/Acquiring, Credit/
  Charge Card**).
- **The authorisation domain (✅, verified this pass from the specs — a find the brief did
  not expect):** **Transaction Authorization** exists in both r13 and r14
  (`TransactionAuthorization.yaml` ✅). Description: *"This service domain handles risk based
  authorization for interactive customer transactions. This combines the context (channel)
  transaction, customer details and recent activity analysis as appropriate. The
  authorization may require a specific level of party/customer authentication to get
  approval."* CR `InteractiveTransactionAssessment`; 7 paths (`Evaluate`, `Exchange`,
  `Execute`, `Grant`, `Request`, `Retrieve`, `Update`). **Its exact landscape position was
  not re-verified this pass** ⚠ (search backend degraded) — the domain is real, its
  business-area placement in the published overview diagrams is flagged.
- **The rules corner (flagged):** the task asked about a "Rule Set" interplay. The honest
  verified result: **no `RuleSet`/`Rule Set` service-domain spec exists in the r12–r14
  ISO20022+DDD API sets** (⚠, verified by listing the r13/r14 yaml directories — zero
  rule-named files). BIAN expresses rules as **behavior qualifiers** on the domains that
  enforce them (`ComplianceCheck` on Payment Instruction, `VerifyMandate`/`FundsAvailableCheck`
  on Direct Debit — ✅ CMAS §6.2) and as the bank's own rule/limits engines behind the APIs
  (§6.3).

### 2.2 The landscape table — the transaction-related service domains

Verified names from the sources above (r13 names unless noted; every name is a real BIAN
service domain):

| Service domain | Area / position | Role in transaction processing | Verification |
|---|---|---|---|
| **Transaction Engine** | Payments → Clearing And Settlement (v13) | Schedules and executes the payment-transaction and reporting activities of long-term instruments / structured facilities | ✅ this pass (InSite v8/v12/v13 + r13/r14 yamls) |
| **Payment Order** (r13) / **Payment Order Initiation** (r14) | Payments → Clearing And Settlement | The customer-facing initiation of a payment transaction (r14: CR `PaymentOrderInitiationTransaction`, BQs `Compliance`/`Confirmation`/`OrderInitiation`) | ✅ (CMAS §3.2/§3.4) |
| **Payment Instruction** (r13, removed r14) | Payments → Clearing And Settlement | Validation and preparation of the instruction (BQs `AgreementConfirmation`, `ComplianceCheck`, `PaymentInstruction`); 29 paths | ✅ (CMAS §3.2) |
| **Payment Execution** (r13, removed r14) | Payments → Clearing And Settlement | Back-end execution of the funds movement; `PaymentMechanism` selection | ✅ (CMAS §3.2) |
| **Payment Orchestration** (r14) | Payments (portal only ⚠) | Coordinates the multi-step execution of a payment instruction | ✅ portal / ⚠ not in oas3 set |
| **Payment Rail** (r14) | Payments | The rail-facing operating session (outbound/inbound, clearing/settlement, reconciliation BQs); 36 paths | ✅ (CMAS §3.4) |
| **Financial Gateway** (r13) | Payments | The r13-era connectivity domain (BQs `Inbound`/`Outbound`) | ✅ (CMAS §3.2) |
| **Transaction Authorization** | (position ⚠) | Risk-based authorisation of interactive customer transactions — CR `InteractiveTransactionAssessment`; 7 paths | ✅ domain verified this pass; position ⚠ |
| **ACH Operations / Correspondent Bank Operations / Payment Rail Operations / Cheque Processing / Order Allocation / Correspondent Bank Directory / Counterparty Administration** | Payments → Clearing And Settlement | The rail- and instrument-specific operational domains of the clearing/settlement view | ✅ (v13 InSite view) |
| **Card Authorization / Card Capture / Card Clearing / Card Financial Settlement / Card Transaction Switch / Card eCommerce Gateway** | Cards | The card transaction estate: authorisation (0100/0110), capture, clearing, scheme settlement | ✅ (umbrella §3.3; yaml listings) |
| **Standing Order / Direct Debit / Direct Debit Mandate / Disbursement** | Payments | The recurring/collection instruments that generate transactions on schedules — the Transaction Engine's natural counterparties | ✅ (CMAS §3.2) |
| **Current Account / Corporate Current Account / Virtual Account / Internal Bank Account** | Accounts | The money containers the transactions move; their BQs (`Payment`, `Deposit`, `Sweep`, r14 `DebitandCredit`) are the transaction events on the facility | ✅ (CMAS §2.2) |
| **Position Keeping** | Accounts | The position log; `FinancialTransactionCapture` BQ captures every movement — the transaction→position seam | ✅ (CMAS §2.2) |
| **Account Reconciliation** | Accounts | Reconciles internal vs external transaction records (BQs `AccountAssessment`/`AccountResolution`) | ✅ (CMAS §2.2) |
| **Currency Exchange** | Cash management / corporate products | The FX conversion domain for cross-currency transactions | ✅ (CMAS §4.1, r14 list) |
| **Cash Concentration** | Cash management | Sweeps/zero-balance transfers — scheduled transaction runs on balances | ✅ (CMAS §4.2) |

### 2.3 The catalog position — flags and honest notes

- **No official "transaction" business area.** The transaction domains are distributed across
  Payments, Cards, Accounts, and cash/corporate products; the Payments-area "Clearing And
  Settlement" view is the closest thing to a transaction-processing cluster (✅ v13 InSite).
- **No official service-domain codes.** BIAN's published artifacts identify domains by name
  and API resource path (`/TransactionEngine/…`), not by a code registry (⚠ — §3.2).
- **Transaction Authorization's position is flagged ⚠** (domain verified; placement not
  re-verifiable this pass).
- **"Rule Set" is not a service domain in the current API set ⚠** — rules live in
  qualifiers and in the bank's rule engines (§6.3).

> **Cross-ref:** the landscape hierarchy is umbrella **§3.1–3.4**; the r13 payment chain and
> the r14 rationalization are CMAS **§3.2–3.4**; the card estate is umbrella **§3.3**; the
> rails those domains connect to are [`payment_rails_guide.md`](payment_rails_guide.md)
> **§1–§2** and the hub that hosts them is [`payments_hub_guide.md`](payments_hub_guide.md)
> **§3**.

---

## 3. The Transaction Engine Domain

### 3.1 The name, verified

**The exact name is "Transaction Engine"** — no hyphen, no qualifier — and it is a **service
domain**, not an engine product. Verified this pass:

- bian.org InSite v13.0 SD Overview page: `«ServiceDomain» Transaction Engine` ✅
- bian.org InSite v12.0 SD documentation page title: "Transaction Engine" ✅
- bian.org InSite v8.0 landscape: "Transaction Engine (Class)" / "(Capability)" ✅
- `github.com/bian-official/public` — `release13.0.0/.../TransactionEngine.yaml` and
  `release14.0.0/.../TransactionEngine.yaml`, both titled **"Transaction Engine"** ✅
- The repo's verified catalog agrees: CMAS §3.1 lists **Transaction Engine** as a ✅ real
  domain (r9.1–r14); umbrella §3.3 lists it in the Payments row.

**Retention across releases (✅):** present in r9.1-era specs, retained through r13, **still
present in r14** — while its neighbours (Payment Execution/Instruction/Order) were removed in
the r14 rationalization, the Transaction Engine survived unchanged in shape (identical 6-path
spec in r13 and r14, verified by comparing the two yamls this pass).

### 3.2 The code / abbreviation — flagged

**BIAN does not publish an official code or abbreviation for the Transaction Engine** ⚠. The
sources verified this pass (the InSite SD pages, the OpenAPI specs, the landscape views)
identify the domain by name only; the API resource path is `/TransactionEngine/...`; there is
no "TE" code in any BIAN artifact examined. **"TE" is therefore an informal shorthand only —
do not treat it as an official BIAN code.** (This guide uses it at most as a local
abbreviation after first use, and flags it as such.) If a vendor presents a "BIAN TE-123"
catalog code, treat the claim as unverified.

### 3.3 The behavior qualifiers — verified and flagged

The task brief expected the Transaction Engine to carry behavior qualifiers such as
*transaction initiation, execution, notification, update*. The honest verified result:

| Expected (task brief) | Status | What BIAN actually has |
|---|---|---|
| **Initiation qualifier** | ❌ No such BQ | Initiation is the **`Initiate` operation** on the CR (service group `Transaction Schedule Facility_Instantiation`) ✅ |
| **Execution qualifier** | ❌ No such BQ | Execution is the **`Execute` operation** on the CR (service group `Transaction Schedule Facility_Invocation`) ✅ |
| **Notification qualifier** | ❌ No such BQ | Notification is a **documented behavior of the domain's role** ("or simply notify the calling service center of the need to do so", v8 ✅), delivered operationally via the escalation link to the owning fulfillment domain — not a named qualifier |
| **Update qualifier** | ❌ No such BQ | The **`Update` and `Control` operations** on the CR cover schedule maintenance ✅ |
| **Any qualifier at all** | `«BehaviorQualifier» NA` (✅ v13 InSite); BQ type "Fulfillment" declared but no BQ names published; the OpenAPI specs expose **only** the tag `CR (DDD Aggregate) - TransactionScheduleFacility`, zero BQ paths (✅ r13/r14) | The domain is **CR-only**: a single aggregate, six operations |

**The architect's reading of the empty qualifier set:** a FULFILL-pattern domain with one
aggregate and no sub-aggregates is BIAN's way of saying the domain is *elemental* — the
schedule facility is the whole capability, and the six operations cover its lifecycle. The
fractal BQ pattern of umbrella §4.3 is not violated; it simply is not needed where the asset
(schedule) does not decompose into independently-owned sub-assets in BIAN's model. The
*behaviors* the brief expected to find as qualifiers are present as **operations** — exactly
the "action term + control record" scoping rule of umbrella §5.3.

### 3.4 The domain table — the verified anatomy

| Attribute | Verified value | Source |
|---|---|---|
| **Name** | Transaction Engine | ✅ InSite v12/v13; r13/r14 yamls |
| **Functional pattern** | Fulfill | ✅ v13 InSite (`«FunctionalPattern» Fulfill`) |
| **Generic artifact** | Facility | ✅ v13 InSite (`«GenericArtifact» Facility`) |
| **Asset type** | Transaction Schedule | ✅ v13 InSite (`«AssetType» Transaction Schedule`) |
| **Control record** | `TransactionScheduleFacility` | ✅ v13 InSite; r13/r14 spec tag |
| **Behavior qualifier type** | Fulfillment | ✅ v13 InSite (`«BehaviorQualifierType» Fulfillment`) |
| **Behavior qualifiers** | NA — none published | ✅ v13 InSite (`«BehaviorQualifier» NA`); zero BQ paths in the specs |
| **Analytics object** | Transaction Schedule Facility Analytics Object | ✅ v13 InSite |
| **Service groups** | `Transaction Engine_SD_Operations`; `Transaction Schedule Facility_Instantiation`; `…_Invocation`; `…_Reporting` | ✅ v13 InSite |
| **Role** | "Utility/background operational support service to orchestrate a schedule of payment transaction and reporting activities for the fulfillment of certain long term instruments or structured facilities… maintains a link to the associated fulfillment Service Domain and the applicable product instance" | ✅ v12 InSite |
| **Example of use** | "The mortgage fulfillment Service Domain delegates repayment processing to the Transaction Engine for its active mortgage product instances" | ✅ v12 InSite |
| **Key features** | Establish the processing schedule for a product instance; process transaction and reporting tasks; escalate issues to the product fulfillment SD; report on fulfillment activity | ✅ v12 InSite |
| **Operations** | `Initiate` (POST `/TransactionEngine/Initiate`); `Update`/`Control`/`Exchange`/`Execute` (PUT `/TransactionEngine/{id}/…`); `Retrieve` (GET `/TransactionEngine/{id}/Retrieve`) — **6 paths** | ✅ v12 InSite (InCR/UpCR/CoCR/EcCR/ExCR/ReCR) + r13/r14 yamls |
| **ISO 20022 mapping** | Spec carries the ISO20022+DDD expansion; the scheduled transactions it runs surface as pain/pacs messages downstream (§4) | ✅ r13/r14 spec header; ⚠ no message-level mapping published for this domain |
| **Position** | Payments area → "Clearing And Settlement" view | ✅ v13 InSite view_97164/view_102450 |
| **Retention** | r9.1 → r14, unchanged shape | ✅ CMAS §3.1/§3.4; r13 vs r14 yaml identical in paths |
| **Reference information** | Product and Service Pricing | ✅ v13 InSite |
| **Official code** | None published — "TE" is informal shorthand only | ⚠ flagged §3.2 |

### 3.5 The six operations, in the domain's own words (✅ v12 InSite)

| Operation (v12 label) | Meaning (v12 InSite text) | REST | When it fires |
|---|---|---|---|
| **Initiate** (InCR) | "Initiate delegated transaction processing for a product instance" | POST `/TransactionEngine/Initiate` | A product domain hands a product instance's schedule to the engine |
| **Update** (UpCR) | "Update details for an active transaction processing arrangement" | PUT `/TransactionEngine/{id}/Update` | Schedule terms change (rate, dates, amounts) |
| **Control** (CoCR) | "Control an active transaction processing arrangement" | PUT `/TransactionEngine/{id}/Control` | Suspend/resume/terminate a schedule (umbrella §5.2) |
| **Exchange** (EcCR) | "Accept, reject a proposed action" | PUT `/TransactionEngine/{id}/Exchange` | The multi-party accept/reject step with the fulfillment domain (umbrella §5.2) |
| **Execute** (ExCR) | "Execute an automated function for a transaction processing arrangement" | PUT `/TransactionEngine/{id}/Execute` | The scheduled run — process the due transactions/reports now |
| **Retrieve** (ReCR) | "Retrieve details about delegated transaction processing for a product instance" | GET `/TransactionEngine/{id}/Retrieve` | Query the schedule state, the run history, the escalations |

**Why `Exchange` sits in this set:** the engine is a *delegated* utility — every scheduled
action is proposed to (or accepted from) the owning fulfillment domain, and `Exchange` is the
BIAN verb for exactly that accept/reject verification dialogue (umbrella §5.2, ✅). The
domain is not a fire-and-forget batch job; it is a contractual partner of the product
domains, and the operation set mirrors that contract.

---

## 4. The Transaction Lifecycle

### 4.1 The five stages, verified

Every transaction — a funds transfer, a card purchase, an FX conversion, a scheduled
repayment — passes through the same five-stage spine: **initiation → authorisation →
execution → clearing → settlement**. The stages are verified as *states* in the repo's
payments-hub state machine ([`payments_hub_guide.md`](payments_hub_guide.md) §4, ✅:
`initiated → validated → authorized → routed → submitted → acknowledged → cleared → settled →
reconciled → reported`), as *BIAN service domains* (verified this pass and in the CMAS
guide), and as *ISO 20022 message families* ([`iso_20022_core_processes_guide.md`](iso_20022_core_processes_guide.md)
§4–§7, ✅).

**1. Initiation** — the transaction is created and captured. BIAN: the customer-facing
initiation domains — **Payment Order** (r13, CR `PaymentOrderProcedure`, BQs
`ExecutionInitiation`/`OrderConfirmation`) → **Payment Order Initiation** (r14, CR
`PaymentOrderInitiationTransaction`, BQs `Compliance`/`Confirmation`/`OrderInitiation`) ✅;
the instrument domains (Standing Order, Direct Debit, Cheque Processing's
`FinancialTransactionInitiation` BQ) ✅; and — for scheduled product transactions — the
**Transaction Engine's `Initiate`**, which establishes the schedule on behalf of a product
instance ✅ (v12 InSite). ISO 20022: the **pain** family — `pain.001` (credit transfer
initiation), `pain.008` (direct debit initiation) ✅ (iso_20022 guide §4). Hub state:
`initiated → validated`.

**2. Authorisation** — the transaction is checked and approved. BIAN: the dedicated
**Transaction Authorization** domain (✅ this pass): *"risk based authorization for
interactive customer transactions… combines the context (channel) transaction, customer
details and recent activity analysis… may require a specific level of party/customer
authentication"* — CR `InteractiveTransactionAssessment`, operations
`Evaluate`/`Grant`/`Request`/`Exchange`/`Execute`/`Update`/`Retrieve`. For cards, **Card
Authorization** (0100/0110, funds hold — cross-ref [`nets_software_systems_guide.md`](nets_software_systems_guide.md)
§2.3 ✅). The checks are qualifiers on the payment domains (`ComplianceCheck`, `VerifyMandate`,
`FundsAvailableCheck` — ✅ CMAS §6.2) plus the bank's limits and fraud engines
([`banking_limits_domain_guide.md`](banking_limits_domain_guide.md) ✅). Hub state:
`validated → authorized`; status via `pacs.002` ✅.

**3. Execution** — the movement is processed. BIAN: **Payment Execution** (r13, CR
`PaymentExecutionProcedure`, BQ `PaymentMechanism`) ✅; in r14 the orchestration view —
**Payment Orchestration** (portal ⚠) driving the **Payment Rail** session ✅; and the
**Transaction Engine's `Execute`** for scheduled runs ✅. Hub state: `routed → submitted →
acknowledged`. ISO 20022: the **pacs** family — `pacs.008` (FIToFI customer credit
transfer), `pacs.009` (FI credit transfer / cover) ✅ (iso_20022 guide §5, §12).

**4. Clearing** — the transaction's obligations are established between the parties, before
final funds movement. BIAN: the entire **"Clearing And Settlement" view** is the clearing
cluster (✅ §2.1) — **ACH Operations** (batch clearing), **Cheque Processing**, **Card
Clearing** (presentments), **Correspondent Bank Operations**, **Order Allocation** (trades);
the r14 **Payment Rail** carries a `PaymentClearingandSettlement` BQ ✅. The mechanics:
clearing houses, RTGS vs deferred net settlement (DNS) — cross-ref
[`payment_rails_guide.md`](payment_rails_guide.md) §5 ✅. Hub state: `cleared`.

**5. Settlement** — funds actually move and the accounts are posted. BIAN: **Card Financial
Settlement** (scheme settlement), **Position Keeping** with the `FinancialTransactionCapture`
BQ (the movement captured into the position log — ✅ CMAS §2.2), **Internal Bank Account**
(nostro/vostro control), **Account Reconciliation** ✅. The mechanics: RTGS finality vs DNS
end-of-cycle settlement, liquidity (cross-ref [`payment_rails_guide.md`](payment_rails_guide.md)
§5.2–5.3 ✅); hub state: `settled → reconciled → reported`. ISO 20022: settlement advice and
statements — the **camt** family (`camt.053` statement, `camt.054` notification) ✅
(iso_20022 guide §8). The postings themselves are the accounting pair's machinery — §7.

**Where the Transaction Engine sits in the lifecycle:** the engine is the *scheduler for
stage 3 of scheduled transactions* — it does not initiate (stage 1 belongs to the customer/
product domains), it does not authorize (stage 2 belongs to Transaction Authorization / Card
Authorization), and it does not clear or settle (stages 4–5 belong to the clearing/
settlement domains). It **executes what a product schedule says is due**, hands the movement
to the execution chain (Payment Execution / Payment Orchestration → Payment Rail), and
escalates issues back to the owning fulfillment domain (✅ v12 role text). In hub terms: the
engine is the *trigger source* that injects scheduled transactions into the
`initiated`/`submitted` states on schedule.

### 4.2 The lifecycle table — stage / BIAN domain / ISO 20022 / hub state

| Stage | What happens | BIAN service domains (verified) | ISO 20022 (cross-ref) | Hub state (payments_hub §4) |
|---|---|---|---|---|
| **Initiation** | The transaction is created and captured from a channel, instrument, or schedule | Payment Order (r13) / Payment Order Initiation (r14); Standing Order, Direct Debit, Cheque Processing (`FinancialTransactionInitiation` BQ); Transaction Engine `Initiate` (schedules) | `pain.001`, `pain.008`, `pain.002` | initiated → validated |
| **Authorisation** | Risk-based checks, limits, authentication; approval granted or refused | Transaction Authorization (CR `InteractiveTransactionAssessment`); Card Authorization (cards, 0100/0110); the compliance/mandate/funds qualifiers; the limits engine | `pacs.002`; card-network auth messages (ISO 8583 0100/0110) | validated → authorized |
| **Execution** | The movement is processed: mechanism selected, rail engaged | Payment Execution (r13) / Payment Orchestration (r14); Payment Rail (r14); Financial Gateway (r13); Transaction Engine `Execute` (scheduled runs) | `pacs.008`, `pacs.009` (cover); rail formats (FAST API, ISO 8583 0200) | routed → submitted → acknowledged |
| **Clearing** | Obligations established; batches netted; presentments exchanged | The Clearing And Settlement view: ACH Operations, Cheque Processing, Card Clearing, Correspondent Bank Operations, Order Allocation; Payment Rail `PaymentClearingandSettlement` BQ | `pacs.008`/`pacs.009` clearing legs; ACH/cheque/card clearing files | cleared |
| **Settlement** | Funds move; positions and accounts posted; books reconciled | Card Financial Settlement; Position Keeping (`FinancialTransactionCapture`); Internal Bank Account (nostro); Account Reconciliation | `camt.053`, `camt.054`, settlement advices; RTGS/DNS mechanics | settled → reconciled → reported |

**The failure paths (✅ payments_hub §4.2):** rejected (any stage up to cleared), returned
(after clearing), reversed, charged back (cards), recalled — each with its BIAN counterpart:
`Request` operations for human intervention (umbrella §5.2 ✅), the `Control` lifecycle
operations, the posting engine's reversals/storno ([`posting_engine_core_banking_guide.md`](posting_engine_core_banking_guide.md)
§3.7 ✅), and the card reversal family 0400/0410/0420 ([`nets_software_systems_guide.md`](nets_software_systems_guide.md)
§2.6 ✅).

> **Cross-ref:** the hub state machine is [`payments_hub_guide.md`](payments_hub_guide.md)
> **§4**; the clearing/settlement mechanics are [`payment_rails_guide.md`](payment_rails_guide.md)
> **§5**; the pain/pacs/camt lifecycle is [`iso_20022_core_processes_guide.md`](iso_20022_core_processes_guide.md)
> **§4–§8**; the card authorisation and financial flows are [`nets_software_systems_guide.md`](nets_software_systems_guide.md)
> **§2.3–§2.4**.

---

## 5. The Transaction Types

### 5.1 Funds transfer (verified)

The classic transaction type — move money between accounts, intra-bank or inter-bank. BIAN's
r13-era chain (✅ CMAS §3.2–3.3): **Payment Order** (customer order) → **Payment
Instruction** (validate: `AgreementConfirmation`, `ComplianceCheck`) → **Payment Execution**
(select `PaymentMechanism`, process the movement) → **Financial Gateway** (`Outbound` session
to SWIFT/ACH/RTGS) → **Position Keeping** (`FinancialTransactionCapture`) → **Account
Reconciliation**. The r14 shape (✅ CMAS §3.4): **Payment Order Initiation** → **Payment
Orchestration** → **Payment Rail** (`OutboundTransaction`, `PaymentClearingandSettlement`).
Intra-bank transfers short-circuit the rail (the posting pair's four-leg internal pattern —
[`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) §9, P3/P6 ✅);
inter-bank transfers engage the rails and the clearing/settlement stage (§4). ISO 20022:
`pain.001` → `pacs.008` (customer credit transfer), `pacs.009` (bank-to-bank/cover) ✅
(iso_20022 guide §5, §12). The recurring variant is the **Standing Order** domain (CR
`StandingOrderFacility`, BQ `StandingOrderPayment` — each generated payment a sub-record ✅),
the natural client of a Transaction Engine schedule; the collection variant is **Direct
Debit** (✅ CMAS §3.2); the paper variant **Cheque Processing** (✅).

### 5.2 Card (verified)

The card transaction type — the four-party model (cardholder → issuer → network → acquirer →
merchant, ✅ [`payment_rails_guide.md`](payment_rails_guide.md) §2.1). BIAN's card estate (✅
umbrella §3.3 + yaml listings): **Card Authorization** (the 0100/0110 authorisation —
authorize now, hold the funds), **Card Capture** (the presentment of the authorized
transaction), **Card Clearing** (presentment exchange between issuer and acquirer via the
scheme), **Card Financial Settlement** (the net settlement between the parties), **Card
Transaction Switch** (the routing/switch domain — the BIAN shape of the NETS-style switch
that [`nets_software_systems_guide.md`](nets_software_systems_guide.md) §2 documents), and
**Card eCommerce Gateway** (online card presentment). The mechanics are ISO 8583 (✅ nets
guide §2.3–2.4): authorization 0100/0110, financial 0200/0210 (single-message), completion
0220/0230, reversal 0400/0410/0420; dual-message vs single-message is the fundamental
architectural fork (✅ nets guide §2.5). The Transaction Engine's card relevance: card
products with structured cash flows (instalment plans, card-loan repayments) delegate their
scheduled runs to the engine, exactly like the mortgage example of §3.1.

### 5.3 FX (verified)

The FX transaction type — convert value across currencies. BIAN's domain: **Currency
Exchange** (✅ CMAS §4.1, r14 list — the FX conversion domain for cross-currency sweeps and
payments). Cross-currency transactions add the conversion leg to the execution stage: the
movement is priced, converted, and balanced per currency — the posting pair's FX mechanics
([`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) §6: FX leg
construction, rate capture, round-trip balancing, P5/P14 ✅) and the hub's FX enrichment
([`payments_hub_guide.md`](payments_hub_guide.md) §8.2 ✅). In the BOM (umbrella §6.2 ✅), FX
trades are the **Trade** business object (Trade Settlement domain), and the treasury/FX
front-office runs on Murex-class platforms in the Cymbal Bank estate (⚠ per
[`credit_agricole_software_systems_guide.md`](credit_agricole_software_systems_guide.md) §3 —
cross-ref [`murex_mx3_platform_guide.md`](murex_mx3_platform_guide.md)). The Transaction
Engine's FX relevance: structured FX facilities (forward schedules, multicurrency repayment
runs) use the same delegated-schedule pattern, each scheduled leg executed as a
Currency-Exchange-scoped conversion before the payment leg.

### 5.4 The types table — type / BIAN domains / messages / mechanics home

| Transaction type | BIAN service domains (verified) | Message standards | Mechanics home (cross-ref) |
|---|---|---|---|
| **Funds transfer** (intra-bank) | Payment Order / Payment Order Initiation (r14); the account domains' `Payment`/`DebitandCredit` BQs; Position Keeping capture | Internal canonical → posting legs (no rail message) | posting_rules §9 (P3, P6, P9); posting_engine §4 |
| **Funds transfer** (inter-bank) | r13: Payment Order → Payment Instruction → Payment Execution → Financial Gateway; r14: Payment Order Initiation → Payment Orchestration → Payment Rail | `pain.001` → `pacs.008`/`pacs.009`; rail formats (FAST, SEPA, SWIFT, ACH) | iso_20022 §4–§5; payment_rails §2; payments_hub §4–§6 |
| **Card** | Card Authorization, Card Capture, Card Clearing, Card Financial Settlement, Card Transaction Switch, Card eCommerce Gateway | ISO 8583 0100/0110/0200/0220/0400; clearing files | nets §2 (flows, dual/single-message); payment_rails §2.1 |
| **FX** | Currency Exchange; Trade Settlement (trades); the payment chain for the converted leg | FX spot/forward conventions; the converted payment continues as pacs.008 | posting_rules §6 (P5, P14); payments_hub §8.2 |
| **Scheduled product transactions** (the Transaction Engine's own type) | **Transaction Engine** (schedule + run); Standing Order (recurring); the product-fulfillment domains (Loan/Mortgage `Billing`/`Payments` BQs) | Whatever the generated movement needs (pain/pacs/card/cheque) | umbrella §4.3 (MortgageLoan example); §3 of this guide |

> **Cross-ref:** the rails taxonomy is [`payment_rails_guide.md`](payment_rails_guide.md)
> **§2**; the ISO 8583 anatomy and flows are [`nets_software_systems_guide.md`](nets_software_systems_guide.md)
> **§2**; the pain/pacs/camt catalogue is [`iso_20022_core_processes_guide.md`](iso_20022_core_processes_guide.md)
> **§1–§2**; the card estate's BIAN names are umbrella **§3.3**.

---

## 6. The Service-Domain Orchestration

### 6.1 The Transaction Engine ↔ the product domains (verified)

The engine's reason to exist is delegation from product-fulfillment domains. Verified from
bian.org's own words: *"The Service Domain maintains a link to the associated fulfillment
Service Domain and the applicable product instance for reporting and processing issues that
may arise"* and *"The mortgage fulfillment Service Domain delegates repayment processing to
the Transaction Engine for its active mortgage product instances"* (✅ v12 InSite). The
pattern, in BIAN terms:

- **The product domain owns the facility; the engine owns the schedule.** A **Loan** /
  **Mortgage Loan** facility (umbrella §4.3's verified example: CR `MortgageLoanFacility`,
  BQs `Billing`, `Payments`, `Withdrawals`, `Sweep`…) generates a repayment schedule. The
  product domain `Initiate`s a `TransactionScheduleFacility` with the engine, and the engine
  runs the schedule (✅).
- **The `Exchange` operation is the contract.** Every proposed action is accepted/rejected
  between the engine and the fulfillment domain (✅ v12: "Accept, reject a proposed action")
  — the umbrella §7.3 delegated-service-exchange test applied: the schedule has an
  independent lifecycle from the product instance, so it gets its own domain (✅ umbrella
  §7.3).
- **Escalation is designed in.** The v12 key features include *"escalate issues to the
  product fulfillment Service Domain as necessary"* (✅) — the engine's failure path is a
  first-order connection back to the product domain, not a silent break.
- **The account domains are the movement endpoints.** When a scheduled run fires, the actual
  movements land on the account facilities' BQs (`Payment`, `DebitandCredit`, `Sweep` — ✅
  CMAS §2.2) and the position log (`FinancialTransactionCapture` — ✅).

### 6.2 The channels (verified structure, flagged detail)

Transactions enter from channels — the umbrella's **Channels** business area (✅ umbrella
§3.4; the channel domains include eBanking, Contact Center, ATM Network, POS, Card Terminal
Administration/Operation — ✅ umbrella §3.3). The channel angle of the transaction estate:

- **Transaction Authorization explicitly consumes channel context** (✅, verified this pass):
  *"This combines the context (channel) transaction, customer details and recent activity
  analysis…"* — BIAN's authorisation domain is channel-aware by definition.
- **The channels hand off to the transaction domains** (⚠ structural, the standard BIAN
  consumption pattern — a channel API call is an `Initiate` on Payment Order / Payment Order
  Initiation or an `Evaluate` on Transaction Authorization). The exact first-order
  connections from each channel domain were not re-verified this pass ⚠.
- **Scheduled transactions have no channel** — the Transaction Engine's `Initiate` comes
  from a product domain, not a channel (✅ v12: "delegated transaction processing for a
  product instance") — the channel/schedule split is one of the cleanest seams in the BIAN
  map: interactive transactions are channel-born and authorization-checked; scheduled
  transactions are product-born and engine-run.

### 6.3 The rules — the Rule-Set interplay (verified and flagged)

- **No "Rule Set" service domain in the current API set (⚠ flagged, verified by listing the
  r13 and r14 yaml directories — zero rule-named specs).** If a vendor or a brief cites a
  BIAN "Rule Set" service domain, treat the *current-catalog* claim as unverified. (A rules
  catalogue may exist in the wider landscape documentation — not re-verifiable this pass ⚠.)
- **Rules are expressed as behavior qualifiers where they are enforced (✅):** `ComplianceCheck`
  (Payment Instruction), `VerifyMandate` + `FundsAvailableCheck` (Direct Debit), the
  `Compliance` BQ (r14 Payment Order Initiation) — the rule *checks* are sub-aggregates of
  the domains that run them (✅ CMAS §6.2).
- **The engine's own rule interplay is the schedule + the escalation link (✅):** the
  Transaction Engine's rules are its schedule parameters (what runs when) and its
  acceptance/escalation dialogue with the fulfillment domain — the `Control` (suspend/
  resume) and `Exchange` (accept/reject) operations are the rule-enforcement surface (✅ v12
  operation texts).
- **The bank's rule engines sit behind the APIs (⚠ structural, cross-ref the repo):** BIAN
  does not standardise the bank's internal limits, pricing, or fraud rule engines — those
  are the [`banking_limits_domain_guide.md`](banking_limits_domain_guide.md) domain (limits
  checks on every balance-moving transaction, ✅) and the fraud/risk systems
  ([`financial_fraud_detection_at_scale_guide.md`](financial_fraud_detection_at_scale_guide.md)),
  wired into the authorisation stage (§4) and the payment domains' compliance qualifiers.

### 6.4 The orchestration table — the engine and its neighbours

| Orchestration pair | Interaction (verified) | Evidence |
|---|---|---|
| **Transaction Engine ↔ product domains** (Loan/Mortgage/structured facilities) | Product domain delegates its schedule: `Initiate` → engine runs it; `Exchange` accept/reject; escalations flow back | ✅ v12 InSite (role, example, key features) |
| **Transaction Engine ↔ the execution chain** | The engine's `Execute` triggers the generated movements into the payment chain (r13: Payment Execution; r14: Payment Orchestration → Payment Rail) | ✅ v12 InSite (`Execute` text); CMAS §3.3/§3.4 |
| **Product domains ↔ account domains** | The movements land on the facility BQs (`Payment`, `DebitandCredit`, `Sweep`) | ✅ CMAS §2.2 |
| **Channels ↔ Transaction Authorization** | Interactive transactions are channel-context + risk assessed (auth before execution) | ✅ TransactionAuthorization.yaml description (this pass) |
| **Execution ↔ rails** | Payment Rail `OutboundTransaction` / `PaymentClearingandSettlement` (r14); Financial Gateway `Outbound` (r13) | ✅ CMAS §3.2/§3.4 |
| **Execution ↔ Position Keeping** | Every movement captured: `FinancialTransactionCapture/Capture` | ✅ CMAS §2.2 |
| **Execution ↔ rules** | Compliance/mandate/funds qualifiers on the payment domains; the bank's limits engine at the authorisation stage | ✅ CMAS §6.2; banking_limits_domain_guide |
| **Clearing/settlement ↔ reconciliation** | Payment Rail `PaymentAccountReconciliation`; Account Reconciliation `AccountAssessment`/`AccountResolution` | ✅ CMAS §3.4/§2.2 |

**The orchestration principle (✅ umbrella §7.3):** every hop above is a *delegated service
exchange* justified by the control-record-lifecycle test — the schedule, the order, the
validated instruction, the executed movement, the outbound session, and the position log each
own an independent lifecycle, so each gets its own domain. The Transaction Engine is the
*least-coupled* node in the map: it speaks only to its owning fulfillment domain and the
execution chain — which is exactly what a "utility/background operational support service"
should be (✅ v12).

---

## 7. The Transaction-to-Posting Flow

### 7.1 Transaction vs posting — the two-layer model (verified)

The transaction is the *event*; the posting is the *accounting record* the event produces.
The repo's accounting pair states it precisely: "[`posting_engine_core_banking_guide.md`](posting_engine_core_banking_guide.md)
§2.1 — Posting vs Transaction" (✅): a transaction is what the business sees (a payment, a
card purchase, a repayment); a posting is the set of DR/CR legs that move balances and GL
accounts. The BIAN map expresses the same split structurally: the **Financial Transaction**
business object (BOM, umbrella §6.2 ✅) is owned by the movement domains (Payment Execution,
Transaction Engine, Trade Settlement, Card Authorization), while the **Account / Position**
objects are owned by the account domains and **Position Keeping** — and the seam between
them is the **`FinancialTransactionCapture` BQ** on `PositionKeeping` (✅ CMAS §2.2): the
transaction is *captured into* the position, then the posting machinery turns it into legs.

### 7.2 The flow, verified stage by stage

1. **The transaction is initiated and authorized** (§4) — a channel-born payment, a card
   authorization, or a scheduled run (`TransactionEngine/Execute`). The result is an
   authorized, executable movement.
2. **The movement is executed** — the payment chain (r14: Payment Order Initiation →
   Payment Orchestration → Payment Rail) moves the value; the rail acknowledges.
3. **The movement is captured into the position** — `PositionKeeping/FinancialTransactionCapture/
   Capture` (✅ CMAS §2.2): the position log records the movement — the transaction→position
   seam, the BIAN twin of the ledger's transaction-table → position-table split
   ([`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md) cross-ref).
4. **The posting engine builds the legs** — the posting rules determine sides and accounts
   ([`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) §2.1: the
   debit/credit determination matrix; §2.2 account determination; §2.3 GL-string generation,
   ✅): e.g. a funds transfer → DR payer / CR payee (+ fee legs); a loan repayment → the EMI
   split of the sibling's P10 (principal CR / interest CR / fee — ✅ posting_rules §9); a
   card authorization hold → the soft hold of P13, hardened on capture (✅ posting_rules §9
   P13; nets guide §2.3's "funds earmarked (hold)").
5. **The balances and the GL update** — the balance engine applies the legs atomically
   ([`posting_engine_core_banking_guide.md`](posting_engine_core_banking_guide.md) §4.5:
   atomicity, concurrency, hot accounts ✅), the entry engine commits the journal entry (§4.6
   ✅), GL integration posts the control accounts (§4.7 ✅), value dates are applied (§4.9 —
   the three dates: transaction, posting, value ✅).
6. **The books are reconciled** — `AccountReconciliation` (BQs `AccountAssessment`/
   `AccountResolution` ✅) and the hub's reconciliation layer
   ([`payments_hub_guide.md`](payments_hub_guide.md) §10 ✅) match the transaction across
   hub, rail, core, and nostro records; breaks land in suspense with an owner
   ([`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) §4, P8 ✅).

**The invariant that binds the two layers (✅ posting_rules §9, the golden discipline):**
every executed transaction produces exactly one balanced journal set — Σ DR = Σ CR — and the
transaction id (plus the idempotency key, posting_rules §7 ✅) is the lineage link between
the BIAN transaction record and its postings.

### 7.3 The flow table — transaction step / BIAN domain / posting mechanic

| Flow step | BIAN domain (verified) | Posting mechanic (cross-ref) |
|---|---|---|
| Initiate (incl. scheduled) | Payment Order / Payment Order Initiation; Transaction Engine `Initiate` | Posting-lifecycle initiation (posting_engine §3.1) |
| Authorise | Transaction Authorization (`Evaluate`/`Grant`); Card Authorization; the compliance/mandate/funds qualifiers; limits engine | Limits guard the debit legs (banking_limits_domain_guide); maker-checker authorization (posting_engine §3.3) |
| Execute | Payment Execution (r13) / Payment Orchestration + Payment Rail (r14); Transaction Engine `Execute` | — (the movement happens; the posting follows) |
| Capture into position | Position Keeping `FinancialTransactionCapture` | Transaction→position seam; transaction tables vs position tables (cockroachdb_guide) |
| Post the legs | (the accounting pair's machinery) | DR/CR matrix + account determination + GL strings (posting_rules §2); entry engine + atomic commit (posting_engine §4.6, §4.5.1) |
| Update balances & GL | Internal Bank Account (nostro/GL control accounts); the account facilities' BQs | Balance engine, hot accounts, GL integration, three dates (posting_engine §4.5, §4.7, §4.9) |
| Reconcile | Account Reconciliation (`AccountAssessment`/`AccountResolution`); Payment Rail `PaymentAccountReconciliation` | Sub-ledger-vs-GL balancing, control accounts, suspense (posting_rules §8, §4) |
| Handle failures | `Request` operations (human intervention); `Control` lifecycle ops; the reversal flows | Storno/reversals (posting_engine §3.7; posting_rules §3), card reversals (nets §2.6), idempotency keys (posting_rules §7) |

> **Cross-ref:** the posting lifecycle is [`posting_engine_core_banking_guide.md`](posting_engine_core_banking_guide.md)
> **§3**; the posting engine architecture is **§4**; the posting rules are
> [`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) **§2**; the worked
> postings library (P1–P14) is **§9**; the card hold→capture mechanics are the sibling's
> **P13** and [`nets_software_systems_guide.md`](nets_software_systems_guide.md) **§2.3–2.4**.

---

## 8. The Banking Context — the Cymbal Bank Transaction Estate

### 8.1 The estate, verified from the corpus

Cymbal Bank is the Singapore bank whose architecture the corpus documents (the persona's
employer; see [`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) §11 for
the canonical framing — "the Singapore retail and SME bank" whose flows the posting library
serves, plus the APAC corporate/investment arm of the Cymbal Bank group). The transaction
estate that the BIAN map of this guide describes:

- **The payments cluster (✅ cross-ref):** Cymbal Bank runs PayNow/FAST (the real-time rail
  over MEPS+), the NETS switch estate for card-present debit (the 0100/0110 authorisation
  flows, dual-message batch settlement — [`nets_software_systems_guide.md`](nets_software_systems_guide.md)
  §2–§3 ✅), SWIFT for cross-border with SEPA/STET in Europe (⚠ per
  [`credit_agricole_software_systems_guide.md`](credit_agricole_software_systems_guide.md)
  §7), and the CB card rails — the "one bank, many rails, one hub" pattern of
  [`payment_rails_guide.md`](payment_rails_guide.md) §7.1 (✅). The BIAN-shaped boundary in
  front of all of it is the r14 **Payment Rail** domain (CMAS §3.4 ✅).
- **The transaction instruments (✅ cross-ref):** standing orders, GIRO direct debits, FAST
  transfers, NETS debit, cheque clearing — each with its BIAN counterpart from §2's table.
- **The accounting underneath (✅ cross-ref):** every one of those transactions becomes the
  posting pair's mechanics — the four-leg transfer, the suspense park for unmatched inward
  credits, the storno on same-day recall, the card hold→capture (P13), the EMI split on loan
  repayment (P10) — [`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md)
  §9/§11 ✅; the interest engine books the accruals and capitalizations
  ([`interest_calculation_engine_guide.md`](interest_calculation_engine_guide.md) ✅).
- **The control layers (✅ cross-ref):** the limits engine guards every balance-moving debit
  ([`banking_limits_domain_guide.md`](banking_limits_domain_guide.md)); the fraud/risk layer
  screens at the authorisation stage; the batch window sizes the EOD runs
  ([`../technology/capacity_sizing_guide.md`](../technology/capacity_sizing_guide.md) ✅) and
  the 24/7 rails demand the availability design of
  [`../technology/zero_downtime_system_design_guide.md`](../technology/zero_downtime_system_design_guide.md)
  ✅.
- **The scheduled-transaction gap (the Transaction Engine's seat):** Cymbal Bank's standing
  orders, loan repayment runs, structured-facility cash flows, and instalment schedules are
  exactly the "long term instruments or structured facilities" of the BIAN role text — the
  **Transaction Engine** is the BIAN home for that scheduling capability, which in vendor
  cores lives inside the product modules (Temenos Transact's arrangement-level schedules,
  FLEXCUBE's product schedules — cross-ref [`core_banking_systems_guide.md`](core_banking_systems_guide.md)
  and [`t24_programming_guide.md`](t24_programming_guide.md) lightly ⚠-structural).

### 8.2 The banking table — the Cymbal Bank flows × the BIAN map

| Cymbal Bank flow | BIAN service domains (this guide's map) | Repo home |
|---|---|---|
| PayNow/FAST transfer (real-time, MEPS+) | Payment Order Initiation → Payment Orchestration → Payment Rail (`OutboundTransaction`); Position Keeping capture; Account Reconciliation | payment_rails §3.9; payments_hub §4; posting_rules §9 P6/P8/P9 |
| NETS debit card purchase (0100/0110) | Card Authorization → Card Capture → Card Clearing → Card Financial Settlement; Card Transaction Switch | nets §2.3–2.5; posting_rules §9 P13 |
| Standing order run (monthly) | **Transaction Engine** (schedule) → Standing Order (`StandingOrderPayment`) → execution chain | CMAS §3.2 (Standing Order); §3/§4 of this guide |
| Loan repayment (EMI, monthly) | **Transaction Engine** (delegated schedule, the mortgage example ✅) → Loan/Mortgage Loan fulfillment → payment chain → postings | umbrella §4.3 (MortgageLoan BQs); posting_rules §9 P10 |
| Cross-border supplier payment (SWIFT/SEPA) | Payment Order Initiation → Payment Rail; Currency Exchange (if cross-currency); Account Reconciliation | payment_rails §2.5/§8; iso_20022 §13 (CBPR+) |
| Cheque clearing | Cheque Processing (`ChequeExtraction`, `ChequeExceptionHandling`, `FinancialTransactionInitiation`) | CMAS §3.2; nets §3.5 |
| FX conversion in the app | Currency Exchange + the payment chain; the posting pair's FX legs | posting_rules §6 (P5/P14); payments_hub §8.2 |
| EOD interest + GL close | Position Keeping / Internal Bank Account; Account Reconciliation | interest_calculation_engine_guide; posting_rules §9 P1–P2/P11 |

**The architect's read:** Cymbal Bank's transaction estate *is* the BIAN transaction
landscape of §2, running on vendor cores and the NETS/payment rails — the BIAN map gives the
bank a common vocabulary to describe all of it (the same vocabulary a vendor evaluation uses:
ask each vendor for its Service Domain coverage matrix, umbrella §12.2 ✅). The one capability
the estate documents only implicitly — *scheduled transaction runs* — is precisely the
Transaction Engine's seat.

---

## 9. The Worked Example — a BIAN-Aligned Transaction-Engine Design

### 9.1 The scenario — a Cymbal Bank transaction flow (the familiar context)

**The bank:** Cymbal Bank's Singapore APAC hub (the corpus's familiar Cymbal Bank context —
[`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) §11,
[`payment_rails_guide.md`](payment_rails_guide.md) §8.1). It runs the retail/SME core plus the
APAC corporate arm; the estate is a legacy host core wrapped by a service layer, the NETS/
FAST/MEPS+ rails, SWIFT for cross-border, and a payments hub in front of the rails. The
adoption posture is umbrella §13.1 **Level 2–3**: *align new interfaces to BIAN service
operations and semantics; begin componentizing around service domains* — not a full Level-4
BIAN-native rewrite (✅ umbrella §13.1).

**The flow — one day, three transactions, one design.** The brief for the transaction-engine
program: (1) a **scheduled mortgage repayment** — a Cymbal Bank mortgage client's monthly
EMI of S$4,200, due today, generated by the loan product's repayment schedule; (2) an
**interactive card purchase** — a NETS debit purchase of S$86.50 at a merchant, authorized in
~200 ms through the switch; (3) an **instant funds transfer** — a PayNow transfer of
S$480.00 between two Cymbal Bank customers. The design target: a **BIAN-aligned transaction
engine** that owns the scheduled-run capability (the gap §8.1 identified) and orchestrates
the three flows through the verified domains of this guide. Every domain, control record,
operation, and path below is a **verified BIAN name** from §2–§3 and the CMAS guide —
nothing invented.

### 9.2 The service-domain orchestration design — endpoint by endpoint

**Leg A — the scheduled mortgage repayment (the Transaction Engine's own flow):**

```
Step A1  Loan fulfillment establishes the schedule with the engine:
         POST /TransactionEngine/Initiate
           { "productInstance": "MortgageLoanFacility/…/ML-1001",
             "schedule": [ { "dueDate": "2026-08-28", "amount": 4200.00, "currency": "SGD",
                             "type": "EMI" }, … ],
             "fulfillmentDomain": "MortgageLoan" }
         → 201 { "transactionEngineId": "TE-5521", "status": "Active" }
         (CR: TransactionScheduleFacility — verified §3.4)

Step A2  The due date arrives; the engine runs the scheduled action:
         PUT /TransactionEngine/TE-5521/Execute
         → 200 { "action": "Execute", "result": "Initiated", "movementRef": "PAY-…" }
         (Execute = "execute an automated function for a transaction processing
         arrangement" — verified v12 text)

Step A3  The repayment enters the payment chain (r14 shape):
         POST /PaymentOrderInitiation/Initiate        // payer: ML-1001, amount: 4200.00, INTRA
         GET  /PaymentOrderInitiation/POI-…/Compliance/{id}/Retrieve
         PUT  /PaymentRail/{railId}/OutboundTransaction/{otId}/Exchange   // intrabank → no rail hop
         (r14 domains verified — CMAS §3.4; intrabank short-circuit per posting_rules §9 P3)

Step A4  Capture into the position and post:
         PUT  /PositionKeeping/{posId}/FinancialTransactionCapture/{ftcId}/Capture
              { "debit": "…/ML-1001", "credit": "…/DDA-2210", "amount": 4200.00 }
         → posting engine: the EMI split — DR Mortgage Loan / CR client DDA,
           principal vs interest legs (posting_rules §9 P10 — verified mechanics)
```

**Leg B — the NETS debit purchase (the card flow, cross-ref the switch guide):**

```
Step B1  Card Authorization (the 0100/0110 round trip through the switch):
         POST /CardAuthorization/…   ← funds earmarked (hold)
         (nets_software_systems_guide.md §2.3 — the hold; posting_rules §9 P13 — soft hold)
Step B2  Capture + clearing + settlement at end of day:
         Card Capture → Card Clearing → Card Financial Settlement
         (verified card domains, §5.2; dual-message batch settlement, nets guide §2.5)
Step B3  The hold hardens into the posting: DR customer DDA / CR merchant settlement (P13 ✅)
```

**Leg C — the PayNow transfer (the instant flow):**

```
Step C1  POST /PaymentOrderInitiation/Initiate          // payer DDA-A, payee DDA-B, 480.00, FAST
Step C2  PUT  /TransactionAuthorization/{id}/Evaluate   // channel-context risk check
Step C3  PUT  /PaymentRail/{railId}/OutboundTransaction/{otId}/Exchange   // FAST via MEPS+
         ← rail ack in real time (the hub's acknowledged state — payments_hub §4 ✅)
Step C4  PUT  /PositionKeeping/{posId}/FinancialTransactionCapture/{ftcId}/Capture
         → the four-leg posting (DR A / CR B — posting_rules §9 P6-family ✅),
           idempotency key PAY-20260828-88412 (posting_rules §7 ✅)
```

**The full sequence, in one line (BIAN vocabulary):**

```
MortgageLoan → TransactionEngine.Initiate → TransactionEngine.Execute → PaymentOrderInitiation.Initiate
→ PaymentRail.OutboundTransaction.Exchange → PositionKeeping.FinancialTransactionCapture.Capture
→ (posting legs: EMI split)   ‖   CardAuthorization → CardCapture → CardClearing → CardFinancialSettlement
→ (posting legs: hold→capture)   ‖   PaymentOrderInitiation.Initiate → TransactionAuthorization.Evaluate
→ PaymentRail.OutboundTransaction.Exchange → PositionKeeping.FinancialTransactionCapture.Capture
→ (posting legs: four-leg transfer)
```

### 9.3 The lessons

1. **The Transaction Engine is a schedule owner, not a transaction hub.** The verified role
   text says it: the engine orchestrates *scheduled* payment-transaction and reporting
   activities for *long-term instruments and structured facilities*. Design it as the
   product-fulfillment scheduling capability — not as a re-branded payment hub. The hub is
   the payment chain's job; the engine is the scheduler that feeds it.
2. **No qualifiers, no code — design to the operations.** The domain has six operations and
   zero BQs (verified §3.3–3.4). Your API contract for the engine is `Initiate`/`Update`/
   `Control`/`Exchange`/`Execute`/`Retrieve` on the `TransactionScheduleFacility` — and the
   behaviors you wanted as "initiation/execution/notification qualifiers" are exactly those
   operations plus the escalation link to the fulfillment domain. Do not invent BQs to make
   the spec prettier; the elemental CR-only shape is the BIAN design.
3. **The `Exchange` and `Control` verbs are the operational contract.** Schedule changes
   (`Update`), suspension (`Control`), and accept/reject dialogues (`Exchange`) with the
   product domains are first-class — the engine is a contractual partner of fulfillment.
   Build the accept/reject and escalation flows from day one.
4. **The r14 payment chain is the downstream target.** The engine's `Execute` feeds the
   r14 shape — Payment Order Initiation → Payment Orchestration → Payment Rail — with the
   r13 specs as the semantic reference (CMAS §3.4 ✅). Pin the release, keep both
   vocabularies.
5. **The transaction-to-posting seam is `FinancialTransactionCapture`.** The BIAN map ends
   at the position log; the posting legs are the accounting pair's machinery (posting_rules
   §2, §9 ✅). Wire the engine's executed movements into `PositionKeeping/…/Capture`, and let
   the posting engine apply the DR/CR matrix, the idempotency keys, and the suspense
   landing zones. The transaction id + idempotency key is the lineage link.
6. **Authorisation is a separate domain — use it.** Interactive transactions get
   `TransactionAuthorization` (channel context + risk + authentication, ✅); scheduled
   product transactions are pre-authorized by their product terms. Do not bolt
   authorisation onto the engine; the BIAN split mirrors the real operational split.
7. **Rules live in qualifiers and in your engines.** No Rule Set domain in the current API
   set (⚠ §6.3) — express rules as the compliance/mandate/funds qualifiers on the payment
   domains and as the bank's limits/fraud engines at the authorisation stage.
8. **Vendor evaluation becomes a diff.** Ask Temenos/FLEXCUBE-class vendors where their
   schedule-driven transaction runs map to the Transaction Engine (⚠-structural; the vendor
   cores implement this inside product modules — cross-ref core_banking_systems_guide.md).
   The gaps on this guide's tables are the gaps in your platform (umbrella §12.2 ✅).

---

## 10. The Summary — One Page

**The map.** BIAN's transaction-processing estate is a verified set of service domains
clustered under the Payments area's **Clearing And Settlement** view (Order Allocation, ACH
Operations, Payment Order, Transaction Engine, Correspondent Bank Operations, Payment
Execution, Cheque Processing, Payment Instruction, Card Financial Settlement, Card Clearing,
Card eCommerce Gateway, Payment Rail Operations — ✅ v13 InSite), surrounded by the account/
position spine (Current Account… Position Keeping, Account Reconciliation), the card estate
(Card Authorization, Card Capture, Card Clearing, Card Financial Settlement, Card
Transaction Switch), the authorisation domain (**Transaction Authorization** — ✅ verified
this pass), the FX domain (Currency Exchange ✅), and the r14 payment chain (Payment Order
Initiation → Payment Orchestration → Payment Rail ✅).

**The domain.** The **Transaction Engine** is a real BIAN service domain (✅, r9.1–r14,
retained unchanged in r14): a **FULFILL-pattern utility** whose asset is the **Transaction
Schedule**, whose control record is the **`TransactionScheduleFacility`**, whose six
operations are `Initiate`/`Update`/`Control`/`Exchange`/`Execute`/`Retrieve` (✅ v12 InSite +
r13/r14 OpenAPI specs), whose role is to *"orchestrate a schedule of payment transaction and
reporting activities for the fulfillment of certain long term instruments or structured
facilities"* — executing them or notifying the calling fulfillment domain (✅ v8/v12). It has
**no published behavior qualifiers** (`«BehaviorQualifier» NA`, ✅ v13) and **no official
code** (⚠) — the expected initiation/execution/notification *qualifiers* are *operations*,
and "TE" is informal shorthand only.

**The lifecycle and the types.** Initiation (Payment Order / Payment Order Initiation; the
engine's `Initiate`) → Authorisation (Transaction Authorization; Card Authorization; the
compliance/mandate/funds qualifiers; the limits engine) → Execution (Payment Execution r13 /
Payment Orchestration + Payment Rail r14; the engine's `Execute`) → Clearing (the Clearing
And Settlement cluster; pacs.008) → Settlement (Card Financial Settlement; Position Keeping
capture; camt.053; RTGS vs DNS) — each stage verified against the payments hub's state
machine, the ISO 20022 guide, and the rails/NETS guides. The types: funds transfer
(pain.001→pacs.008), card (ISO 8583 0100/0200, dual/single-message), FX (Currency Exchange +
the posting pair's FX legs) — verified. **The flow to postings:** the movement is captured
into the position (`FinancialTransactionCapture`), then the posting rules build the DR/CR
legs, the entry engine commits them atomically, the balances and GL update, and
reconciliation closes the loop — the accounting pair's machinery, cross-referenced.

**The banking context.** Cymbal Bank's estate — PayNow/FAST, NETS debit, SWIFT/SEPA, the
vendor cores — is the BIAN landscape in production; the one capability documented only
implicitly (scheduled transaction runs) is the Transaction Engine's seat.

**The final word — the transaction at the centre.** BIAN's gift to the transaction
architect is not an engine product — it is a **map with the transaction at the centre**:
every service domain in this guide exists to serve one movement — the product domains
generate it, the Transaction Engine schedules it, the authorisation domain gates it, the
payment chain executes it, the rails clear and settle it, Position Keeping captures it, the
posting engine books it, and reconciliation proves it. Learn where each stage lives, use the
verified names, and the transaction — the most important object in banking — finally has a
home in the architecture: not buried in a product silo, but scheduled, executed, captured,
and booked by the domains that own its journey.

---

## 11. Glossary

| Term | Definition |
|---|---|
| **BIAN** | Banking Industry Architecture Network — the open, member-owned standard for banking IT architecture (est. 2008, Frankfurt); publisher of the Service Landscape, the BOM, the meta model, and Semantic API specifications (umbrella §1). |
| **Service Domain** | The elemental building block of the BIAN Service Landscape — one functional pattern applied to one asset type for its complete lifecycle; a bounded context in DDD terms (umbrella §4). |
| **Service Landscape** | BIAN's core artifact: the complete map of the bank as ~320+ MECE service domains under Business Area → Business Domain → Service Domain (umbrella §3). |
| **Transaction Engine** | The verified BIAN service domain (CR `TransactionScheduleFacility`, FULFILL pattern) that "orchestrates a schedule of payment transaction and reporting activities for the fulfillment of certain long term instruments or structured facilities" (§3). |
| **Qualifier / Behavior Qualifier (BQ)** | A sub-aggregate partition of a control record that scopes service operations — in DDD terms the sub-aggregates (umbrella §4.3). The Transaction Engine has none published (`NA` — §3.3). |
| **Behaviour** | (BIAN usage) The functional behaviour of a domain — its pattern of activity over the lifecycle, expressed through operations and, where present, BQs (umbrella §4.1–4.3). |
| **Lifecycle** | The stages of a transaction's (or domain instance's) life from creation to completion — for transactions: initiation → authorisation → execution → clearing → settlement (§4). |
| **Initiation** | The stage at which a transaction is created and captured — Payment Order / Payment Order Initiation; the Transaction Engine's `Initiate` for schedules (§4). |
| **Authorisation** | The stage at which a transaction is risk-checked and approved — Transaction Authorization (CR `InteractiveTransactionAssessment`), Card Authorization, the compliance/mandate/funds qualifiers, the limits engine (§4). |
| **Execution** | The stage at which the movement is processed — Payment Execution (r13) / Payment Orchestration + Payment Rail (r14); the Transaction Engine's `Execute` for scheduled runs (§4). |
| **Clearing** | The stage at which the transaction's obligations are established between parties, before final funds movement — the Clearing And Settlement cluster; RTGS vs DNS (payment_rails §5). |
| **Settlement** | The stage at which funds actually move and accounts are posted — Card Financial Settlement, Position Keeping capture, RTGS finality / DNS cycle settlement (§4). |
| **Orchestration** | The coordination of service domains around a business outcome — delegated service exchanges between domains (product → engine → payment chain → position → reconciliation), each justified by the control-record-lifecycle test (umbrella §7.3; §6). |
| **Rule Set** | ⚠ No service domain of this name exists in the r12–r14 BIAN API set; rules are expressed as qualifiers (`ComplianceCheck`, `VerifyMandate`, `FundsAvailableCheck`) and in the bank's own rule/limits engines (§6.3). |
| **Payments** | The BIAN business area holding the payment transaction domains — the Clearing And Settlement view, the r14 payment chain, the instrument domains (umbrella §3.3; §2). |
| **Posting** | The accounting record a transaction produces: the set of DR/CR legs that move balances and GL accounts (posting_engine_core_banking_guide §2.1; §7). |
| **ISO 20022** | The international standard for financial message semantics (pain/pacs/camt families) that BIAN maps to and the rails use — cross-ref iso_20022_core_processes_guide (§4–§7). |
| **Business object** | A conceptual business entity in the BIAN BOM (Party, Customer, Account, Product, Agreement, **Transaction**…) exchanged across service boundaries (umbrella §6). |
| **API / Semantic API** | The REST-shaped (and event-shaped) program interface of one BIAN service domain — `/{ServiceDomain}/{cr-ref-id}/{BQ}/{bq-ref-id}/{Operation}` (umbrella §9; §3.4). |
| **Control record** | The domain's aggregate root / master record — e.g. `TransactionScheduleFacility` (umbrella §4.1; §3.4). |

---

## 12. References and Further Reading

### Primary sources (verified this pass)
- **BIAN Service Landscape v13.0 InSite** — bian.org/servicelandscape-13-0-0/ — the
  Transaction Engine SD Overview page (`views/view_54373.html`: ServiceDomain, AssetType
  Transaction Schedule, CR Transaction Schedule Facility, FunctionalPattern Fulfill,
  GenericArtifact Facility, BQType Fulfillment, `BehaviorQualifier NA`, the service groups)
  and the Clearing And Settlement views (`views/view_97164.html` v12 diagram,
  `view_102450.html` v13 diagram — the 14-domain list)
- **BIAN Service Landscape v12.0 InSite** — bian.org/servicelandscape-12-0-0/ — the
  Transaction Engine SD documentation page (`views/view_118013.html`: role, example of use,
  key features, the InCR/UpCR/CoCR/EcCR/ExCR/ReCR operation texts)
- **BIAN Service Landscape v8.0 InSite** — bian.org/servicelandscape-8-0/ — the Transaction
  Engine class/capability origin note (`object_14.html?object=26440`)
- **Official GitHub repository** — github.com/bian-official/public — `release13.0.0/` and
  `release14.0.0/` `apis-iso20022_ext-ddd/oas3/yamls/TransactionEngine.yaml` (verified: 6
  paths, single CR tag, identical r13/r14) and `TransactionAuthorization.yaml` (verified: 7
  paths, CR `InteractiveTransactionAssessment`); yaml directory listings (verified: no
  rule-named specs; CardAuthorization/CardClearing/CardFinancialSettlement present)
- **The repo's verified BIAN catalog** — [`bian_cash_management_domains_guide.md`](bian_cash_management_domains_guide.md)
  (the r13 payment domains, the r14 rationalization, the BQ table, the API table)

### In this repository (cross-references)
- [`bian_banking_architecture_guide.md`](bian_banking_architecture_guide.md) — **the
  umbrella**: §3 landscape, §4 service-domain model, §5 action terms, §6 BOM, §7.3 delegated
  service exchanges, §9 API standard, §10 releases, §12 vendor evaluation, §13 adoption
- [`bian_cash_management_domains_guide.md`](bian_cash_management_domains_guide.md) — the
  verified r14 SD catalog; the payment domains and the r13/r14 chains (§3), the BQ table
  (§6), the API table (§7)
- [`payments_hub_guide.md`](payments_hub_guide.md) — the payment lifecycle state machine
  (§4), routing/orchestration (§6), reconciliation (§10)
- [`payment_rails_guide.md`](payment_rails_guide.md) — the rails taxonomy (§2), clearing and
  settlement (§5), the Cymbal Bank rail selection (§7–§8)
- [`nets_software_systems_guide.md`](nets_software_systems_guide.md) — the ISO 8583
  authorization/financial/reversal flows and the dual/single-message fork (§2), the
  settlement rails (§3)
- [`iso_20022_core_processes_guide.md`](iso_20022_core_processes_guide.md) — the pain/pacs/
  camt lifecycle (§4–§8), CBPR+ (§13)
- [`posting_engine_core_banking_guide.md`](posting_engine_core_banking_guide.md) — posting
  vs transaction (§2.1), the posting lifecycle (§3), the engine architecture (§4)
- [`posting_rules_mechanics_guide.md`](posting_rules_mechanics_guide.md) — the posting rules
  (§2), the worked postings P1–P14 (§9), the Cymbal Bank context (§11)
- [`interest_calculation_engine_guide.md`](interest_calculation_engine_guide.md) — the
  interest postings (lightly)
- [`banking_limits_domain_guide.md`](banking_limits_domain_guide.md) — the limits/rules
  interplay at the authorisation stage
- [`core_banking_systems_guide.md`](core_banking_systems_guide.md) and
  [`t24_programming_guide.md`](t24_programming_guide.md) — the vendor cores' transaction
  modules (lightly, ⚠-structural)
- [`../technology/capacity_sizing_guide.md`](../technology/capacity_sizing_guide.md) — the
  transaction-throughput sizing; [`../technology/zero_downtime_system_design_guide.md`](../technology/zero_downtime_system_design_guide.md)
  — the 24/7 rails (lightly); [`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md)
  and [`../technology/event_stream_processing_guide.md`](../technology/event_stream_processing_guide.md)
  — the DDD and event seams (lightly); [`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md)
  — the ledger/persistence angle (lightly)
- [`../management/business_case_development_guide.md`](../management/business_case_development_guide.md)
  — the investment case for a transaction-platform program (lightly)

### Vendor/adoption material (flagged ⚠ where not re-verified this pass)
- Temenos Transact / Oracle FLEXCUBE BIAN mappings — schedule-driven transaction runs live
  inside product modules; the Transaction Engine is the BIAN home for that capability
  (⚠-structural; cross-ref umbrella §15.2)
- BIAN white papers on transaction processing — referenced via the umbrella's §21
  bibliography; not re-fetched this pass ⚠

---

## 13. The Verification Ledger — Verified vs Flagged

### Verified this pass (✅ — primary sources)
| Claim | Source |
|---|---|
| Transaction Engine is a real BIAN service domain, retained r9.1→r14 | bian.org InSite v8/v12/v13; `TransactionEngine.yaml` in r13 and r14 |
| Exact name "Transaction Engine"; CR `TransactionScheduleFacility`; AssetType Transaction Schedule; FunctionalPattern Fulfill; GenericArtifact Facility; BQType Fulfillment; `BehaviorQualifier NA` | v13 InSite SD Overview page (view_54373) |
| Role text, example of use (mortgage fulfillment delegates repayment processing), key features (establish schedule, process tasks, escalate, report) | v12 InSite SD documentation (view_118013) |
| The six operations and their meanings (Initiate/Update/Control/Exchange/Execute/Retrieve) | v12 InSite operation texts + r13/r14 OpenAPI specs (6 paths, identical) |
| Position: Payments area → "Clearing And Settlement" view (14 domains) | v13 InSite views view_97164/view_102450 |
| Transaction Authorization domain exists (CR `InteractiveTransactionAssessment`, 7 paths, risk-based authorization description) | r13/r14 `TransactionAuthorization.yaml` |
| No Rule Set service-domain spec in the r12–r14 API set | r13/r14 yaml directory listings |
| Card domains (Card Authorization/Capture/Clearing/Financial Settlement/Transaction Switch) and Currency Exchange in the catalog | umbrella §3.3 + yaml listings + CMAS guide §4.1 |
| The r13 payment chain, the r14 rationalization, the BQ table, the API table | CMAS guide §3–§7 (repo's verified catalog) |
| The payments lifecycle state machine; the posting lifecycle and rules; the NETS ISO 8583 flows; the Cymbal Bank estate | payments_hub §4; posting_engine §3–§4; posting_rules §2/§9/§11; nets §2–§3; payment_rails §7–§8 |

### Flagged this pass (⚠ — honest gaps)
| Claim | Status |
|---|---|
| An official code/abbreviation for Transaction Engine ("TE") | ⚠ **No official code published** in the verified sources; "TE" is informal shorthand only — do not treat as official (§3.2) |
| The expected behavior qualifiers (initiation/execution/notification/update) | ⚠ **Do not exist as BQs** — the v13 landscape publishes `BehaviorQualifier NA`; those behaviors are the domain's *operations* (§3.3) |
| Transaction Authorization's exact position in the landscape overview diagrams | ⚠ Domain verified; business-area placement not re-verifiable this pass (search backend degraded) (§2.3) |
| A "Rule Set" service domain | ⚠ Not in the r12–r14 API set; rules live in qualifiers and the bank's rule engines (§6.3) |
| ISO 20022 message-level mapping for the Transaction Engine itself | ⚠ Spec header confirms the ISO20022+DDD expansion; no per-message mapping published for this domain (§3.4) |
| Channel-domain first-order connections | ⚠ The channel → transaction handoff is structural knowledge; the InSite connection diagrams were not fetched (§6.2) |
| Vendor cores' schedule/transaction-run implementation details | ⚠-structural; cross-ref the vendor guides (§9.3, lesson 8) |

*Verification honesty note: every BIAN service-domain name, control record, operation, and
qualifier statement in this guide was verified this pass against bian.org InSite (v8/v12/v13)
and/or the official GitHub repository (r13/r14 ISO20022+DDD OpenAPI specs), or is taken from
the repo's already-verified BIAN catalog (the CMAS guide and the umbrella). Where a
task-expected element does not exist in BIAN's published landscape (the qualifiers, the code,
the Rule Set domain), it is flagged ⚠ and the real BIAN element is named. Nothing is
fabricated.*
