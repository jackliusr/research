# Core BIAN Cash Management Domains — The Cash-Management Service-Domain Deep-Dive

> **Scope:** This guide is the **domain-level companion** to the BIAN umbrella guide
> [`bian_banking_architecture_guide.md`](bian_banking_architecture_guide.md) (1100+ lines — what
> BIAN is, the Service Landscape, the service-domain model, the action terms, the BOM, the meta
> model, design principles, the API standard, release evolution 8.x→14.0, adoption). **This guide
> does not re-derive the framework.** It takes the umbrella's framework as read and **deepens the
> cash-management corner of the Service Landscape** — the account domains, the payment domains,
> the cash-management domains, their service operations, their behavior qualifiers, their
> semantic APIs — and closes with a worked BIAN-aligned cash-management API design.
>
> **Verification convention (consistent with the repo series):** ✅ = verified this pass against
> primary sources (bian.org InSite Service Landscape v13/v14 pages, the official
> `github.com/bian-official/public` repository — the release 12.0/13.0/14.0 OpenAPI 3.x
> ISO20022+DDD service-domain specs — and the BIAN portal); ⚠ = flagged (inferred, single-source,
> or not re-verifiable); **unmarked** = structural/industry knowledge presented as such. **Where a
> task-expected name turned out not to exist in BIAN's published landscape, this guide says so
> plainly and names the real BIAN element instead** — nothing here is fabricated.
>
> **Audience:** Solution/enterprise architects in corporate banking (Jack Liu, Crédit Agricole
> CIB) who need the *actual* BIAN cash-management service domains — names, control records,
> operations, qualifiers, APIs — to build a BIAN-aligned cash-management platform or to evaluate
> vendor BIAN claims.

---

## Table of Contents

1. [The CMAS Overview — Cash Management and Account Services](#1-the-cmas-overview--cash-management-and-account-services)
2. [The Account Domains](#2-the-account-domains)
3. [The Payment Domains](#3-the-payment-domains)
4. [The Cash-Management Domains](#4-the-cash-management-domains)
5. [The Service Operations — the Action-Term Vocabulary](#5-the-service-operations--the-action-term-vocabulary)
6. [The Behavior Qualifiers (BQs)](#6-the-behavior-qualifiers-bqs)
7. [The Semantic APIs — REST per Domain](#7-the-semantic-apis--rest-per-domain)
8. [The Worked Example — a BIAN-Aligned Cash-Management API Design](#8-the-worked-example--a-bian-aligned-cash-management-api-design)
9. [The One-Page Summary — the Vocabulary of Cash](#9-the-one-page-summary--the-vocabulary-of-cash)
10. [Glossary](#10-glossary)
11. [References and Further Reading](#11-references-and-further-reading)

### Reading Map — how this guide connects to the series

| This guide's section | Umbrella anchor (read first) | Sibling anchors |
|---|---|---|
| §1 CMAS overview | umbrella §3 (Service Landscape hierarchy), §4 (service-domain anatomy) | [`core_banking_systems_guide.md`](core_banking_systems_guide.md) (account/payment domains in the core) |
| §2 Account domains | umbrella §4.2 (functional patterns), §6 (BOM: Account business object) | [`core_banking_systems_guide.md`](core_banking_systems_guide.md) (posting/accounting), [`treasury_alm_guide.md`](treasury_alm_guide.md) (liquidity) |
| §3 Payment domains | umbrella §5 (action terms), §9 (API standard), §11.2 (ISO 20022) | [`payments_hub_guide.md`](payments_hub_guide.md) (payments-hub patterns), [`nets_singapore_guide.md`](nets_singapore_guide.md) / [`singapore_fintech_payments_guide.md`](singapore_fintech_payments_guide.md) (rails) |
| §4 Cash-management domains | umbrella §4 (control records, BQs), §10 (release evolution) | [`treasury_alm_guide.md`](treasury_alm_guide.md), [`capital_markets_architecture_guide.md`](capital_markets_architecture_guide.md) (lightly) |
| §5 Service operations | umbrella §5 (the full action-term table) | [`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md) (service domain as bounded context) |
| §6 Behavior qualifiers | umbrella §4.3 (BQs, the fractal pattern) | [`../technology/monolith_to_microservices_guide.md`](../technology/monolith_to_microservices_guide.md) (decomposition) |
| §7 APIs | umbrella §9 (Semantic API standard, path structure, repo) | [`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md), [`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md) (ledger/persistence) |
| §8 Worked example | umbrella §7.3 (the mortgage scenario pattern), §13 (adoption) | [`credit_agricole_software_systems_guide.md`](credit_agricole_software_systems_guide.md) (the Cymbal Bank context), [`payments_hub_guide.md`](payments_hub_guide.md) |
| §9 Summary | umbrella §20 (key takeaways) | [`../management/mba_body_of_knowledge_guide.md`](../management/mba_body_of_knowledge_guide.md) (lightly) |

**The one-paragraph orientation:** the umbrella gives you the framework — MECE service domains,
control records, action terms, BQs, semantic APIs. This guide answers the *cash-management*
questions a corporate-banking architect actually asks: *which BIAN service domains run a
corporate bank's cash management? What are their real names, control records, operations and
qualifiers? What do the published OpenAPI specs look like? And what changed in release 14.0?*
The honest headline, verified below: **the "cash management" corner of BIAN is spread across the
account domains, the payment domains, and a small set of dedicated cash-management domains —
and release 14.0 (2026) reshuffled the payment domains and added a brand-new service domain
literally named `Cash Management And Account Services`.**

---

## 1. The CMAS Overview — Cash Management and Account Services

### 1.1 The name, verified

**"Cash Management and Account Services" (CMAS) is a real BIAN term, but not in the place most
people expect.** Verification against primary sources:

- **As a Service Domain — YES, since release 14.0 (2026) ✅.** The release-14.0
  ISO20022+DDD API set (`release14.0.0/apis-iso20022_ext-ddd/oas3/yamls/`) contains a service
  domain spec named **`CashManagementAndAccountServices`** whose description reads: *"This
  service domain orchestrates a cash management and accounting services facility typically used
  by corporations to support additional cash management features over and above the standard
  facilities of current and savings accounts."* Its control record is the
  `CashManagementAndAccountServicesFacility` (verified from the OpenAPI spec). This is the
  closest thing BIAN has to a single "corporate cash-management wrapper" domain — the umbrella
  product under which a corporate's balances, sweeps, payments, deposits, and charges are
  orchestrated.
- **As a Business Area in the published Service Landscape — NO ⚠.** The InSite Service
  Landscape v13 and v14 overview pages (`bian.org/servicelandscape-13-0-0/`,
  `bian.org/servicelandscape-14-0-0/`) list the top-level overview diagrams as: **Bank
  Relations, Business Development, Card Products, Channels, Corporate Banking Products,
  Corporate Finance, Lending, Payments, Product and Price, Retail Banking and Consumer,
  Wealth** (+ **Payment (New)** in v14). There is no top-level "Cash Management" business area
  in the current public landscape. The cash-management-relevant domains sit mostly under the
  **Payments** area (the **"Clearing And Settlement"** view, verified v13: Order Allocation,
  ACH Operations, Payment Order, Transaction Engine, Correspondent Bank Operations,
  Correspondent Bank Directory, Counterparty Administration, Payment Execution, Cheque
  Processing, Payment Instruction, Card Financial Settlement, Card Clearing, Card eCommerce
  Gateway, Payment Rail Operations) and under **Corporate Banking Products**.
- **As a vendor/industry grouping — YES.** "Cash Management and Account Services" is the
  conventional vendor grouping for the corporate cash-management module cluster (Temenos
  Transact's cash-management module, Oracle FLEXCUBE, Finastra Fusion — see umbrella §15 and
  [`core_banking_systems_guide.md`](core_banking_systems_guide.md)). BIAN members map these
  products onto the account + payment + cash domains catalogued in this guide.
- **The capabilities layer.** The BIAN landscape also publishes *business capabilities*
  distinct from service domains — e.g. **Liquidity Management** appears as a capability on
  bian.org ("*Ability to monitor, forecast and influence liquidity and cash positions and flows
  in order to optimize flows and ensure settlement/payment obligations can be met*" ✅, searched
  this pass). Several task-expected "domains" (Account Administration, Account Balance, Account
  Reporting, Transaction Capture…) live at this capability level or inside other domains'
  qualifiers rather than as standalone service domains — reconciled honestly in §2–§4.

### 1.2 What the CMAS corner of the landscape actually contains

The verified working set for a corporate cash-management build (release 13.0 names unless
noted; every name below is a real BIAN service domain, verified against the API repository and
the InSite landscape):

| Group | Service Domains (verified names) | Where they live in the landscape |
|---|---|---|
| **Account domains** | Current Account, Corporate Current Account, Savings Account, Term Deposit, Virtual Account, Internal Bank Account, Position Keeping, Account Reconciliation, Customer Position, Financial Accounting | Products / accounts area; Position Keeping and Account Reconciliation are the balance-and-reconcile core |
| **Payment domains** | Payment Execution, Payment Instruction, Payment Order, Direct Debit, Direct Debit Mandate, Standing Order, Cheque Processing, Disbursement, Financial Gateway, Transaction Engine, ACH Operations, Correspondent Bank Operations, Corporate Payroll Services | Payments area — the "Clearing And Settlement" view (v13) |
| **Cash-management domains** | Cash Concentration, Cash Management And Account Services (r14+), Notional Pooling (r14+), Open Item Management (r14+), Central Cash Handling, Currency Exchange | Cash management / corporate banking products |
| **r14 payment replacements** | Payment Order Initiation, Payment Rail, Payment Orchestration (portal) | Payments area — the r14 rationalization (see §3.4) |

### 1.3 The overview table — aspect / description

| Aspect | Description (verified) |
|---|---|
| **What CMAS is in BIAN terms** | Not a business area in the published landscape ⚠; a **service-domain cluster** (account + payment + cash domains) plus, since 14.0, a dedicated **service domain named Cash Management And Account Services** ✅ |
| **The r14 CMAS service domain** | CR `CashManagementAndAccountServicesFacility`; BQs `AmountBlock`, `Charge`, `DebitandCredit`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `PositivePay`, `Sweep` (verified from the r14 OpenAPI spec) — corporate cash-management features "over and above the standard facilities of current and savings accounts" |
| **The account anchor** | Corporate Current Account (CR `CorporateCurrentAccountFacility`) is the corporate transaction account; Current Account (`CurrentAccountFacility`) the retail/basic one; Virtual Account (`VirtualAccountFacility`) the sub-accounting overlay; Internal Bank Account (`InternalBankAccountFacility`) the bank's own books |
| **The position anchor** | Position Keeping (CR `FinancialPositionLog`, BQ `FinancialTransactionCapture`) — the domain that knows the balance/position; Account Reconciliation (CR `AccountReconciliationProcedure`, BQs `AccountAssessment`/`AccountResolution`) — the domain that makes internal vs external books agree |
| **The payment anchor** | r13: Payment Instruction → Payment Execution → Financial Gateway (verify §3); r14: Payment Order Initiation → Payment Orchestration → Payment Rail |
| **The cash anchor** | Cash Concentration (CR `AccountBalanceSweepingFacility`, BQ `CashTransfer`) — sweeps/notional pooling of balances; r14 adds Notional Pooling and Open Item Management |
| **Functional patterns in play** | Process-type procedure domains (Payment Execution), FULFILL-type arrangement domains (Current Account, Cash Concentration), OPERATE-type session domains (Financial Gateway, Payment Rail), TRACK-type log domains (Position Keeping), CATALOG-type directory domains (Direct Debit Mandate) — see umbrella §4.2 for the pattern taxonomy |
| **Control records in play** | `*Facility` (arrangements), `*Procedure` (processes), `*OperatingSession` (operating sessions), `*Log` (position/audit logs), `*DirectoryEntry` (catalog entries) — the CR-naming conventions are visible across all cash-management domains (§2–§4) |
| **Why it matters** | The CMAS cluster is the canonical map for a corporate-banking cash-management platform: initiation → execution → rails → positions → reconciliation → sweeping, with the r14 CMAS domain as the corporate-facing wrapper. Same vocabulary works for vendor evaluation (umbrella §12.2) |

> **Cross-ref:** the Service Landscape hierarchy (Business Area → Business Domain → Service
> Domain, the two layouts, "the Service Domain set is canonical, the grouping is a view") is
> umbrella **§3.1–3.2**; the service-domain anatomy (functional pattern, asset type, control
> record, BQs, operations, first-order connections) is umbrella **§4.1**; the 19 functional
> patterns are umbrella **§4.2**.

---

## 2. The Account Domains

### 2.1 Verification: which "account" names are real BIAN service domains

The task brief asked to verify **Account Administration, Account Balance, Account Reporting,
Account Access**. The honest verification result (against the API repository across releases
9.1 → 14.0 and the InSite landscape):

| Task-expected name | Status | What BIAN actually has |
|---|---|---|
| **Account Administration** | ❌ No such service domain (any release) | The administration of an account **is** the account: Current Account / Corporate Current Account are FULFILL-type arrangement domains whose control record *is* the account facility (CR `CurrentAccountFacility` / `CorporateCurrentAccountFacility`) — see umbrella §6.3 ("in Account Management the control record is the Account itself") |
| **Account Balance** | ❌ No such service domain | Balance is a **state**, maintained by Position Keeping (CR `FinancialPositionLog`) and exposed through the account domains' qualifiers; the `FinancialTransactionCapture` BQ on Position Keeping is where transactions update the position ✅ |
| **Account Reporting** | ❌ No such service domain | Reporting is delivered by Account Reconciliation (BQs `AccountAssessment`/`AccountResolution`), by the `Reporting` BQ on Direct Debit, by Regulatory Reporting / Financial Accounting, and by the account domains' Retrieve operations ✅ |
| **Account Access** | ❌ No such service domain | Access entitlements are a **Customer Access Entitlement** concern (the access-entitlement domain), plus channels — see umbrella §3.3 (Customer row) |
| **Current Account** | ✅ Real domain | CR `CurrentAccountFacility`; 9 BQs in r13 (`AmountBlock`, `BookingAuthorization`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `ServiceFee`, `Sweep`, `Withdrawal`); 34 paths in the r13 spec ✅ |
| **Corporate Current Account** | ✅ Real domain | CR `CorporateCurrentAccountFacility`; same BQ set as Current Account in r13; the corporate variant (multicurrency, sweeps, limits) ✅ |
| **Savings Account** | ✅ Real domain | In the landscape and API set (r14 list verified) |
| **Term Deposit / Deposit Account** | ✅ Real domain | Term Deposit in the r14 list ✅ |
| **Virtual Account** | ✅ Real domain | CR `VirtualAccountFacility`; 48 paths in r13; BQs include `AccountStatement`, `AmountBlock`, `BookingAuthorization`, `DirectDebit`, `Interest`, `IssuedDevice`, `ServiceFee`, `StandingOrder`, `Statement` — the sub-ledger overlay used for receivables/collections ✅ |
| **Internal Bank Account** | ✅ Real domain | CR `InternalBankAccountFacility`; BQ `BookingAuthorization`; the bank's own GL/control accounts ✅ |
| **Position Keeping** | ✅ Real domain | CR `FinancialPositionLog`; BQ `FinancialTransactionCapture`; the domain that *is* "account balance" as a service ✅ |
| **Account Reconciliation** | ✅ Real domain | CR `AccountReconciliationProcedure`; BQs `AccountAssessment`, `AccountResolution`; 22 paths in r13 ✅ |

**Bottom line:** the four task-expected "account" domains (Administration/Balance/Reporting/
Access) are **capability-level concerns, not service domains**. The real BIAN service-domain
decomposition splits account work differently: the account facility (Current/Corporate Current/
Virtual/Internal Bank), the position (Position Keeping), the reconciliation (Account
Reconciliation), and the access entitlement (Customer Access Entitlement). This is exactly the
kind of "the standard disagrees with your mental model" finding that makes BIAN valuable as a
shared vocabulary — and it is the reason a BIAN-aligned account layer looks different from a
classic core-banking account module (see [`core_banking_systems_guide.md`](core_banking_systems_guide.md)).

### 2.2 The account table — domain / purpose / operations

Verified from the release-13.0 ISO20022+DDD OpenAPI specs (`release13.0.0/apis-iso20022_ext-ddd/
oas3/yamls/*.yaml`) unless noted. Operation verbs are the BIAN action terms (umbrella §5).

| Domain | Control record (CR) | Purpose (from spec description / structure) | Operations (verified paths) | Key BQs |
|---|---|---|---|---|
| **Current Account** | `CurrentAccountFacility` | The basic demand-deposit account facility — the money container of the BOM (umbrella §6.2) | `Initiate`, `Update`, `Control`, `Retrieve` on the CR + BQ-scoped ops (`Deposit/Initiate`, `Withdrawal/Initiate+Update`, `ServiceFee/Initiate+Execute+Retrieve`, `Sweep/Initiate+Update+Execute+Retrieve`, `Interest/Retrieve`, `Payment/*`, `AmountBlock/*`) — 34 paths, 8 POST / 17 PUT / 9 GET | `AmountBlock`, `BookingAuthorization`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `ServiceFee`, `Sweep`, `Withdrawal` (r13); r14 swaps to `Booking`, `Charge`, `DebitandCredit` |
| **Corporate Current Account** | `CorporateCurrentAccountFacility` | The corporate transaction account — the anchor of the CMAS cluster | Same operation skeleton as Current Account (34 paths; 8/17/9) | Same 9 BQs as Current Account (r13); same r14 revision |
| **Virtual Account** | `VirtualAccountFacility` | Sub-accounting overlay — allocate one real account into many virtual accounts (receivables, collections, corporate treasury sub-ledgers) | Richest account spec in r13: CR ops `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Grant`, `Capture`, `Request`, `Retrieve` + BQ-scoped ops — 48 paths | `AccountStatement`, `AmountBlock`, `BookingAuthorization`, `DirectDebit`, `Interest`, `IssuedDevice`, `ServiceFee`, `StandingOrder`, `Statement` |
| **Internal Bank Account** | `InternalBankAccountFacility` | The bank's own internal/GL accounts (nostro/vostro control, suspense, clearing accounts) | `Initiate`, `Update`, `Control`, `Notify`, `Capture`, `Retrieve` + `BookingAuthorization/Execute` — 7 paths | `BookingAuthorization` |
| **Position Keeping** | `FinancialPositionLog` | Maintains the account position/balance; every movement updates the log | `Initiate`, `Update`, `Control`, `Retrieve` + `FinancialTransactionCapture/Update+Capture+Retrieve` — 7 paths | `FinancialTransactionCapture` |
| **Account Reconciliation** | `AccountReconciliationProcedure` | Reconciles internal records against external sources (nostro statements, scheme reports) | CR ops `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Notify`, `Request`, `Retrieve` + `AccountAssessment/*`, `AccountResolution/*` — 22 paths | `AccountAssessment`, `AccountResolution` |
| **Customer Position** | (position domain) | The customer-level aggregate position across products — umbrella §3.3 (Accounts row) | In the landscape/API set (r14 list) | — |
| **Savings Account / Term Deposit** | `SavingsAccountFacility` / `TermDeposit*` | The savings/deposit product families (r14 list verified; deep-dive out of scope here) | — | — |

**Architect's reading of the account table:**

- **The account facility is the aggregate root.** `*Facility` control records are FULFILL-pattern
  arrangements (umbrella §4.2) — the account *is* the contract between bank and customer, and
  its BQs (Deposit, Withdrawal, Payment, Sweep, Interest, ServiceFee, AmountBlock) are the
  *things that happen to* an account. This is DDD-friendly: the Facility is the aggregate, the
  BQs are the sub-aggregates (see [`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md)).
- **Balance is not an account.** BIAN separates the money container (the Facility) from the
  position (Position Keeping's `FinancialPositionLog`) — a TRACK-pattern log that
  `FinancialTransactionCapture` updates. This is the BOM's Account-vs-Transaction split
  (umbrella §6.2) made operational, and it maps neatly onto the ledger/persistence thinking in
  [`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md) (position tables vs
  transaction tables).
- **Virtual Account is the corporate workhorse.** 48 paths — the largest account spec — because
  corporate treasury overlays sub-accounting (one legal account, many virtual sub-accounts for
  subsidiaries, projects, or trade flows) on top of the real account. Cymbal Bank corporate
  banks use exactly this pattern for their corporate clients' sub-account structures.
- **The r14 BQ revision is visible here.** `BookingAuthorization` → `Booking`, `ServiceFee` →
  `Charge`, `Withdrawal`/`Deposit` → `DebitandCredit` in r14 Current/Corporate Current Account
  — the release-note "rationalisation of Service Domain definitions" (bian.org ✅) applied to
  the account qualifiers too.


---

## 3. The Payment Domains

### 3.1 Verification: which "payment" names are real BIAN service domains

The brief asked to verify **Payment Execution, Payment Order, Direct Debit, Credit Transfer,
Standing Order, Payment Messaging**. Verified against the API repository (releases 9.1 → 14.0)
and the InSite landscape:

| Task-expected name | Status | What BIAN actually has |
|---|---|---|
| **Payment Execution** | ✅ Real domain (r9.1–r13); **removed in r14** ⚠ | CR `PaymentExecutionProcedure` (v13 InSite: Functional Pattern = **Process**, Generic Artifact = Procedure, BQ type = Workstep); BQs `PaymentExecution`, `PaymentMechanism`. Description (v13 landscape): *"handles the back-end processing of a movement of funds from a debtor account to a creditor account… authorized and validated… before being instructed to Payment Execution."* In r14 the payment domains were rationalized — see §3.4 |
| **Payment Order** | ✅ Real domain (r9.1–r13); **removed in r14** ⚠ | CR `PaymentOrderProcedure`; BQs `ExecutionInitiation`, `OrderConfirmation`. The instruction-side domain (customer-facing payment order) |
| **Direct Debit** | ✅ Real domain (all releases) | CR `DirectDebitFacility`; r13 BQs `BatchDebit`, `FundsAvailableCheck`, `PaymentInitiation`, `PaymentTracking`, `Reporting`, `VerifyMandate` (the umbrella §4.3 example is this domain ✅); r14 BQs simplified to `PaymentBatch`, `Reporting` |
| **Credit Transfer** | ❌ No standalone service domain (any release) ⚠ | Credit transfers are handled by Payment Execution / Payment Instruction (r13) and by Payment Order Initiation (r14). "Credit transfer" is an **ISO 20022 message family** (pacs.008), not a BIAN service domain — BIAN maps to ISO 20022 (umbrella §11.2) rather than duplicating message-type domains |
| **Standing Order** | ✅ Real domain (all releases) | CR `StandingOrderFacility`; BQ `StandingOrderPayment`; CRUD + `Execute` on the facility (r13/r14: 11 paths) |
| **Payment Messaging** | ❌ No service domain with this name (any release) ⚠ | Messaging/connectivity is **Financial Gateway** (r13: CR `FinancialGatewayOperatingSession`, BQs `Inbound`/`Outbound`) and, in r14, **Payment Rail** (CR `PaymentRailOperatingSession`, BQs `InboundTransaction`, `OutboundTransaction`, `PaymentAccountReconciliation`, `PaymentClearingandSettlement`) — the domain that "handles the operational interface with one or more external payment services or 'payment rails'… formatting outbound messages… store and forward… extracting and [processing inbound]" (r14 landscape ✅) |
| **Payment Instruction** | ✅ Real domain (r9.1–r13); **removed in r14** ⚠ | CR `PaymentInstructionProcedure`; BQs `AgreementConfirmation`, `ComplianceCheck`, `PaymentInstruction`; **29 paths in r13** — the richest payment spec, exposing the full action-term set (Initiate, Update, Retrieve, Execute, Control, Exchange, Request, Notify) |
| **Direct Debit Mandate** | ✅ Real domain (all releases) | CR `DirectDebitMandateDirectoryEntry`; BQ `MandateRegistration`; a **Register-based** CATALOG domain — mandates are directory entries, not facilities |
| **Financial Gateway** | ✅ Real domain (r9.1–r14) | CR `FinancialGatewayOperatingSession`; BQs `Inbound`, `Outbound`; the r13-era connectivity domain (SWIFT, ACH, RTGS interfaces) |
| **Transaction Engine** | ✅ Real domain (r9.1–r14) | CR `TransactionScheduleFacility`; the scheduling/execution engine behind payment runs |
| **Cheque Processing** | ✅ Real domain | CR `ChequeProcessingOperatingSession`; BQs `ChequeExceptionHandling`, `ChequeExtraction`, `FinancialTransactionInitiation` |
| **ACH Operations / Correspondent Bank Operations / Payment Rail Operations** | ✅ Real domains (landscape v13 view "Clearing And Settlement") | The rail-specific operational domains |

**Bottom line:** of the six task-expected payment names, **Payment Execution, Direct Debit and
Standing Order are real domains**; **Payment Order is real but renamed in r14**; **Credit
Transfer and Payment Messaging are not BIAN service domains** — the former is an ISO 20022
message concept, the latter is Financial Gateway / Payment Rail. The umbrella's §3.3 payments
row ("Payment Execution, Payment Instruction, Payment Order, Direct Debit, Direct Debit
Mandate, Standing Order, Disbursement, Cheque Processing, Cheque Lock Box, ACH Fulfilment,
Financial Gateway, Correspondent Bank Operations, Transaction Engine") is the verified r13-era
list; r14 replaced the three Instruction/Order/Execution domains.

### 3.2 The payment table — domain / purpose / operations (r13-era canonical set)

Verified from `release13.0.0/apis-iso20022_ext-ddd/oas3/yamls/*.yaml`:

| Domain | Control record (CR) | Purpose | Operations (verified) | Key BQs |
|---|---|---|---|---|
| **Payment Instruction** | `PaymentInstructionProcedure` | Validates and prepares a payment instruction for execution (agreement confirmation, compliance checks, instruction assembly) | CR: `Initiate`, `Update`, `Retrieve`, `Execute`, `Control`, `Exchange`, `Request`, `Notify`; BQ-scoped `Execute`/`Exchange` on `ComplianceCheck`, `AgreementConfirmation` — 29 paths | `AgreementConfirmation`, `ComplianceCheck`, `PaymentInstruction` |
| **Payment Execution** | `PaymentExecutionProcedure` | Back-end processing of the funds movement — selects the payment mechanism, moves funds debtor → creditor (intrabank or via rails) | CR: `Initiate`, `Update`, `Retrieve`, `Control`; BQ-scoped `Retrieve` on `PaymentMechanism` — 6 paths (r13) | `PaymentExecution`, `PaymentMechanism` |
| **Payment Order** | `PaymentOrderProcedure` | The customer-facing payment order with execution-initiation and order-confirmation sub-records | CR: `Initiate`, `Update`, `Retrieve`, `Control`; BQ-scoped `Retrieve`/`Control` on `ExecutionInitiation`, `OrderConfirmation` — 6 paths | `ExecutionInitiation`, `OrderConfirmation` |
| **Direct Debit** | `DirectDebitFacility` | The direct-debit collection facility: mandate verification, funds-available checks, batch collection, tracking, reporting | CR: `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Request`, `Retrieve`; BQ-scoped `Execute` on `FundsAvailableCheck`/`PaymentInitiation`/`VerifyMandate`; `PaymentTracking` `Initiate`+`Control`+`Execute`; `Reporting` `Initiate` — 21 paths | r13: `BatchDebit`, `FundsAvailableCheck`, `PaymentInitiation`, `PaymentTracking`, `Reporting`, `VerifyMandate`; r14: `PaymentBatch`, `Reporting` |
| **Direct Debit Mandate** | `DirectDebitMandateDirectoryEntry` | The mandate registry (CATALOG pattern) — register, update, execute, exchange mandate records | `Register` (CR + `MandateRegistration`), `Update`, `Control`, `Exchange`, `Execute`, `Request`, `Retrieve` — 10 paths | `MandateRegistration` |
| **Standing Order** | `StandingOrderFacility` | The recurring-payment facility: a standing order that generates periodic payments | CR: `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Request`, `Retrieve`; BQ `StandingOrderPayment` `Initiate`+`Update`+`Exchange`+`Retrieve` — 11 paths | `StandingOrderPayment` |
| **Financial Gateway** | `FinancialGatewayOperatingSession` | The external-connectivity operating session (SWIFT/ACH/RTGS interfaces) — the r13 "payment messaging" domain | CR: `Initiate`, `Update`, `Control`, `Request`, `Retrieve`; BQ `Inbound`/`Outbound` `Initiate`+`Retrieve` — 9 paths | `Inbound`, `Outbound` |
| **Transaction Engine** | `TransactionScheduleFacility` | The transaction scheduling/execution engine | CR: `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Retrieve` — 6 paths | — |
| **Cheque Processing** | `ChequeProcessingOperatingSession` | Cheque capture, extraction, exception handling, and initiation of the financial transaction | CR: `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Request`, `Retrieve`; BQ-scoped ops on `ChequeExceptionHandling`, `ChequeExtraction`, `FinancialTransactionInitiation` — 16 paths | `ChequeExceptionHandling`, `ChequeExtraction`, `FinancialTransactionInitiation` |

### 3.3 The r13 payment chain — how the domains cooperate

The r13-era canonical flow (the shape [`payments_hub_guide.md`](payments_hub_guide.md) describes
as the BIAN-aligned payment pattern):

```
Channel/API  →  Payment Order (Initiate)      — customer places the payment order
             →  Payment Instruction (Execute) — validate: AgreementConfirmation, ComplianceCheck
             →  Payment Execution (Execute)   — select PaymentMechanism, process the funds movement
             →  Financial Gateway (Outbound)  — format + send to the rail (SWIFT/ACH/RTGS)
             →  Position Keeping (Capture)    — update debtor/creditor positions
             →  Account Reconciliation (…)    — reconcile vs rail/statement responses
```

Each hop is a *delegated service exchange* justified by the control-record-lifecycle test
(umbrella §7.3): the payment order, the validated instruction, the executed movement, the
outbound session, and the position log each own an independent lifecycle, so each gets its own
domain. The umbrella §5.3 scoping rule applies at every hop: *action term + BQ* makes each
operation unambiguous (an `Execute` on `PaymentInstruction/ComplianceCheck` is not an `Execute`
on `PaymentExecution`).

### 3.4 Release 14.0 — the payment rationalization (verified)

bian.org's Service Landscape page states it plainly ✅: *"BIAN 14.0 has focused on
rationalisation of the Service Domain definitions. Unnecessary Service Operations are
eliminated, more links to ISO20022 are provided. Some Service Domains have been renamed,
removed and added especially in the Payment Area."* What this guide verified in the
release-14.0 API set:

- **Removed/renamed (r13 → r14):** `PaymentExecution`, `PaymentInstruction`, `PaymentOrder` are
  **gone from the r14 ISO20022+DDD API set** (verified by diffing the 251-name r13 list against
  the 259-name r14 list).
- **Replaced by three domains (verified):**
  - **Payment Order Initiation** — CR `PaymentOrderInitiationTransaction`; BQs `Compliance`,
    `Confirmation`, `OrderInitiation` (r14 spec ✅). The initiation side of the old
    Payment Order + Payment Instruction.
  - **Payment Rail** — CR `PaymentRailOperatingSession`; BQs `InboundTransaction`,
    `OutboundTransaction`, `PaymentAccountReconciliation`, `PaymentClearingandSettlement`
    (r14 spec ✅). Absorbs Financial Gateway + Payment Execution's rail-facing work; the
    description (bian.org ✅): *"handles the operational interface with one or more external
    payment services or 'payment rails'… formatting outbound messages… store and forward of
    payments for clearing and settlement purposes, and extracting and [processing inbound]"* —
    36 paths in r14.
  - **Payment Orchestration** — present on the BIAN portal (portal.bian.org,
    `BIAN-14.0.0-PaymentOrchestration` ✅) but **not in the r14 oas3 API-spec set ⚠** — *"the
    service domain orchestrates the execution of the payment instruction… the coordination of
    multiple steps depending on the type of payment instruction (e.g., credit transfer, direct
    debit, request to pay…)… Invoking Payment Confirmation for internal bank policies and
    account [checks]…"* (portal ✅).
- **Direct Debit kept, simplified:** r14 BQs `PaymentBatch` + `Reporting` (the fine-grained
  VerifyMandate/FundsAvailableCheck/PaymentInitiation/PaymentTracking qualifiers were folded
  away — "unnecessary Service Operations are eliminated" ✅).
- **Standing Order, Direct Debit Mandate, Cheque Processing, Financial Gateway, Transaction
  Engine, Disbursement, ACH Operations:** retained in r14 ✅.

**Architect's reading:** the r14 payments redesign collapses the three-layer
Order→Instruction→Execution chain into an initiation/orchestration/rail split, and pushes the
bank's rail connectivity into one operating-session domain (Payment Rail). If you are designing
a payments hub today (see [`payments_hub_guide.md`](payments_hub_guide.md)), design against the
**r14 shape** (Order Initiation → Orchestration → Rail) but note the r13 specs are still
maintained on GitHub and are the richer reference for operation-level semantics.

---

## 4. The Cash-Management Domains

### 4.1 Verification: which "cash" names are real BIAN service domains

The brief asked to verify **Cash Concentration, Cash Management, Liquidity Management, Sweep
Account, Position Keeping, Reconciliation, Transaction Capture**:

| Task-expected name | Status | What BIAN actually has |
|---|---|---|
| **Cash Concentration** | ✅ Real domain (r9.1–r14) | CR `AccountBalanceSweepingFacility`; BQ `CashTransfer`; exposes `Activate`, `Execute`, `Control`, `Exchange`, `Notify`, `Request`, `Update`, `Retrieve` (r13: 14 paths; r14: 16 paths) — the sweep/zero-balance-account engine |
| **Cash Management** | ❌ No standalone domain named "Cash Management" ⚠ | The r14 **Cash Management And Account Services** domain is the wrapper (§1); "Cash Management" is a capability/vendor-module term (SAP's FSCM Cash Management, Temenos cash management — see umbrella §15) |
| **Liquidity Management** | ❌ No service domain; ✅ a BIAN **capability** | bian.org: *"Ability to monitor, forecast and influence liquidity and cash positions and flows in order to optimize flows and ensure settlement/payment obligations can be met"* — a capability, not a service domain; cross-ref [`treasury_alm_guide.md`](treasury_alm_guide.md) for the bank-side liquidity function |
| **Sweep Account** | ❌ No standalone domain ⚠ | **Sweep is a Behavior Qualifier**, not a domain: `Sweep` BQ on Current Account / Corporate Current Account / the r14 CMAS facility, and the CR of Cash Concentration is literally `AccountBalanceSweepingFacility` — sweeping is a *sub-aggregate behavior*, exactly the fractal-qualifier pattern of umbrella §4.3 |
| **Position Keeping** | ✅ Real domain (all releases) | CR `FinancialPositionLog`; BQ `FinancialTransactionCapture`; operations include `Capture` — the position/balance domain |
| **Reconciliation** | ✅ Real domain as **Account Reconciliation** | CR `AccountReconciliationProcedure`; BQs `AccountAssessment`, `AccountResolution`; the r14 Payment Rail even carries a `PaymentAccountReconciliation` BQ |
| **Transaction Capture** | ❌ No standalone domain (except **Card Transaction Capture**) ⚠ | Capturing a movement is the **`FinancialTransactionCapture` BQ on Position Keeping** (and `CardTransactionCapture` for cards); "transaction capture" is a behavior, not a domain — again the qualifier-over-domain pattern |
| **Notional Pooling** | ✅ Real domain (new in r14) | In the r14 domain list (verified) — the interest-optimization pooling facility, a classic corporate cash-management product |
| **Open Item Management** | ✅ Real domain (new in r14) | In the r14 domain list (verified) — the open-items (AP/AR aging) ledger behind cash-matching |
| **Central Cash Handling** | ✅ Real domain | Vault/cash-in-transit handling (r14 list verified) |
| **Currency Exchange** | ✅ Real domain | FX conversion for cross-currency sweeps/payments (r14 list verified) |

### 4.2 The cash table — domain / purpose / operations

Verified from the release-13.0 and release-14.0 ISO20022+DDD specs:

| Domain | Control record (CR) | Purpose | Operations (verified) | Key BQs |
|---|---|---|---|---|
| **Cash Concentration** | `AccountBalanceSweepingFacility` | Sweep balances between accounts (zero-balance accounting, target-balance sweeping) to concentrate funds and optimize interest | `Activate`, `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Notify`, `Request`, `Retrieve` (CR) + `CashTransfer` `Initiate`+`Update`+`Execute`+`Exchange`+`Notify`+`Request`+`Retrieve` (BQ) — r13: 14 paths, r14: 16 paths | `CashTransfer` |
| **Cash Management And Account Services** (r14, new) | `CashManagementAndAccountServicesFacility` | The corporate cash-management wrapper: "additional cash management features over and above the standard facilities of current and savings accounts" — Positive Pay, sweeps, debit/credit handling, charges, deposits, devices, amount blocks | CR: `Initiate`, `Update`, `Control`, `Retrieve`; BQ-scoped: `Charge` `Initiate`+`Execute`+`Retrieve`, `Sweep` `Initiate`+`Update`+`Execute`+`Retrieve`, `DebitandCredit` `Initiate`+`Update`+`Execute`+`Retrieve`, `Payment` `Initiate`+`Update`+`Execute`+`Exchange`+`Retrieve`, `Deposit` `Initiate`+`Update`+`Execute`+`Retrieve`, `AmountBlock` `Initiate`+`Update`+`Retrieve`, `IssuedDevice` `Initiate`+`Update`+`Request`+`Retrieve`, `PositivePay` `Retrieve`+`Exchange`, `Interest` `Retrieve` — 34 paths | `AmountBlock`, `Charge`, `DebitandCredit`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `PositivePay`, `Sweep` |
| **Position Keeping** | `FinancialPositionLog` | Maintain the position/balance log; capture every transaction into the position | `Initiate`, `Update`, `Control`, `Retrieve` + `FinancialTransactionCapture` `Update`+`Capture`+`Retrieve` — 7 paths | `FinancialTransactionCapture` |
| **Account Reconciliation** | `AccountReconciliationProcedure` | Reconcile internal vs external books (statements, rail reports, nostro) | `Initiate`, `Update`, `Control`, `Exchange`, `Execute`, `Notify`, `Request`, `Retrieve` + `AccountAssessment`/`AccountResolution` `Initiate`+`Update`+`Exchange`+`Execute`+`Notify`+`Request`+`Retrieve` — 22 paths | `AccountAssessment`, `AccountResolution` |
| **Notional Pooling** (r14, new) | (pooling facility) | Interest-optimization pooling across group accounts without physical movement | In the r14 landscape/API list; deep-dive not yet published at the r13 richness ⚠ | — |
| **Open Item Management** (r14, new) | (open-items ledger) | Track and match open items (invoices, unmatched receipts) for cash-matching | In the r14 list ⚠ | — |
| **Central Cash Handling** | (cash-handling session) | Branch/vault cash logistics | In the r14 list ⚠ | — |

**Architect's reading of the cash table:**

- **Sweeping is BIAN's cash-concentration primitive.** The CR name `AccountBalanceSweepingFacility`
  says it: concentration = balance sweeping, modeled as a FULFILL arrangement with `CashTransfer`
  sub-aggregates. The `Activate` operation (rare in the cash cluster) reflects that a sweep
  facility is *configured and activated* — a standing arrangement, not a one-off transaction.
- **Position Keeping + Account Reconciliation are the data-integrity spine.** Everything else
  moves money; these two keep the books honest — the position log (TRACK pattern) and the
  reconciliation procedure (PROCESS pattern). For the ledger/persistence angle see
  [`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md); for the
  reconciliation-of-positions-to-reality angle see [`treasury_alm_guide.md`](treasury_alm_guide.md).
- **The r14 CMAS domain is the corporate product wrapper.** Its BQ list (Positive Pay, Sweep,
  Payment, DebitandCredit, AmountBlock, IssuedDevice…) is a corporate cash-management product
  sheet: cheque positive pay, sweeps, payment initiation, debit/credit posting, balance blocks,
  devices. A Cymbal Bank build can anchor its corporate cash-management API on exactly this
  domain (worked example, §8).
- **"Transaction Capture" is a qualifier, not a domain** — the single most useful
  vocabulary correction in this section: BIAN models *behaviors on facilities*, so capturing a
  transaction is `PositionKeeping/FinancialTransactionCapture/Capture`, and card capture is its
  own domain only because cards have a distinct lifecycle.

---

## 5. The Service Operations — the Action-Term Vocabulary

### 5.1 The verified vocabulary (from the umbrella, confirmed in the specs)

The umbrella §5 gives the full BIAN action-term set — the controlled vocabulary of service
operations. The modern API-era taxonomy is **11 primary operations**: `Initiate`, `Register`,
`Evaluate`, `Update`, `Control`, `Exchange`, `Execute`, `Request`, `Retrieve`, `Notify`,
`Capture` (umbrella §5.5). The task brief asked to verify the **Execute / Initiate / Retrieve**
core — verified ✅ and then some: every cash-management spec in §2–§4 uses `Initiate` (create a
facility/procedure/session), `Retrieve` (read current state), and `Update` (modify); `Execute`
appears wherever an automated action is triggered (payment execution, sweep execution, direct
debit steps); `Control`, `Exchange`, `Request`, `Notify`, `Capture`, `Register`, `Activate`,
`Grant`, `Evaluate` appear selectively. The REST mapping (umbrella §5.2) is:
`Initiate`/`Register`/`Evaluate` → POST, `Update`/`Control`/`Exchange`/`Execute`/`Request`/
`Grant`/`Capture` → PUT, `Retrieve` → GET, `Notify` → event subscription (AsyncAPI).

### 5.2 The operations table — action term / meaning / REST / where it fires in the cash cluster

Verified operation occurrences are cited per domain from the r13/r14 OpenAPI specs:

| Action term | Meaning (umbrella §5.2) | REST (umbrella §5.2) | Where it fires in the cash cluster (verified) |
|---|---|---|---|
| **Initiate** | Instantiate a new control record / BQ instance — the default creation term for FULFILL/TRANSACT/PROCESS/OPERATE/MONITOR/TRACK domains | POST `/{SD}/Initiate` | **Everywhere**: `PaymentOrder/Initiate`, `PaymentExecution/Initiate`, `DirectDebit/Initiate`, `StandingOrder/Initiate`, `CurrentAccount/Initiate`, `VirtualAccount/Initiate`, `CashConcentration/Initiate`, `PositionKeeping/Initiate`, `AccountReconciliation/Initiate`, and BQ-scoped (`CashTransfer/Initiate`, `StandingOrderPayment/Initiate`, `Reporting/Initiate`) |
| **Register** | Create a directory/catalog entry (CATALOG/ENROLL domains) | POST `/{SD}/Register` | **Direct Debit Mandate**: `DirectDebitMandate/Register` + `MandateRegistration/Register` — the only Register-based domain in the cash cluster (mandates are directory entries) |
| **Execute** | Trigger an automated action (vs Request = human intervention) | PUT `/{SD}/{id}/Execute` | The payment/cash workhorse: `PaymentInstruction/Execute`, `DirectDebit` BQ steps (`FundsAvailableCheck/Execute`, `VerifyMandate/Execute`, `PaymentInitiation/Execute`), `StandingOrder/Execute`, `CashConcentration/Execute`, `CashTransfer/Execute`, `Sweep/Execute`, `Charge/Execute`, `Payment/Execute`, `DebitandCredit/Execute`, `BookingAuthorization/Execute`, `TransactionEngine/Execute` |
| **Retrieve** | Query current state — read-only (the CQRS read side) | GET `/{SD}/{id}/Retrieve` | **Every domain**; also the only operation on many BQs (`PaymentMechanism/Retrieve`, `Interest/Retrieve`, `PositivePay/Retrieve`, `OrderConfirmation/Retrieve`) |
| **Update** | Modify an existing instance; idempotent | PUT `/{SD}/{id}/Update` | **Everywhere** — facility/procedure maintenance across all account, payment, cash domains |
| **Control** | Manage the processing lifecycle (suspend/resume/terminate) | PUT `/{SD}/{id}/Control` | `PaymentInstruction/Control`, `PaymentExecution/Control`, `PaymentTracking/Control`, `StandingOrder/Control`, `CurrentAccount/Control`, `CashConcentration/Control`, `PositionKeeping/Control`, `AccountReconciliation/Control`, `PaymentRail/Control` |
| **Exchange** | Accept/verify/acknowledge — multi-party verification (correspondent/clearing/rail) | PUT `/{SD}/{id}/Exchange` | The rail-facing verb: `PaymentInstruction/Exchange` (AgreementConfirmation, ComplianceCheck), `DirectDebitMandate/Exchange`, `FinancialGateway` (Inbound/Outbound), `PaymentRail/Exchange` (InboundTransaction, OutboundTransaction, PaymentAccountReconciliation, PaymentClearingandSettlement), `VirtualAccount/Exchange` |
| **Request** | Request manual intervention / human decision (escalation) | PUT `/{SD}/{id}/Request` | Exception paths: `PaymentInstruction/Request`, `StandingOrder/Request`, `DirectDebit/Request`, `CashConcentration/Request`, `AccountReconciliation/Request`, `IssuedDevice/Request`, `ChequeProcessing/Request` |
| **Notify** | Event notification / subscription (AsyncAPI) | event subscription | `PaymentInstruction/Notify`, `CashConcentration/Notify`, `InternalBankAccount/Notify`, `AccountReconciliation/Notify`, `PaymentRail/Notify` — the event-driven seams (umbrella §14.3; [`../technology/event_stream_processing_guide.md`](../technology/event_stream_processing_guide.md)) |
| **Capture** | Record activity/event/audit information against an instance | PUT `/{SD}/{id}/Capture` | The position-update verb: `PositionKeeping/FinancialTransactionCapture/Capture`, `VirtualAccount/Capture`, `InternalBankAccount/Capture` — transaction capture into the logs |
| **Activate** | Bring a standing arrangement into service (applies to all domains per umbrella §5.4) | PUT `/{SD}/{id}/Activate` | `CashConcentration/Activate` — the sweep facility is activated once configured; also r14 CMAS-era activation semantics |
| **Grant** | Grant permission/approval | PUT `/{SD}/{id}/Grant` | `VirtualAccount/Grant` — permissioned sub-account operations |
| **Evaluate** | Establish an agreement/assessment requiring evaluation logic | POST `/{SD}/Evaluate` | Not observed in the cash cluster's core specs ⚠ (more common in ASSESS/ANALYSE domains — underwriting, credit) — the vocabulary is *available* even where the cash domains don't use it |

**The two structural lessons (verified against the specs):**

1. **The verb set is small and reusable.** Twelve action terms cover every operation in the
   ~200 paths of the cash cluster's r13 specs. A BIAN-aligned API program gets its entire
   cash-management contract from this vocabulary — no bespoke verbs.
2. **Action term + BQ = unambiguous scope** (umbrella §5.3). `Execute` on
   `CashConcentration/{id}/CashTransfer/{id}` means "run this sweep leg now"; `Execute` on
   `DirectDebit/{id}/VerifyMandate/{id}` means "verify this mandate now". The same verb, the
   same REST shape, different sub-aggregate — this is what makes BIAN APIs composable and
   vendor-swappable (umbrella §9.4).

---

## 6. The Behavior Qualifiers (BQs)

### 6.1 What BQs are (one paragraph, then the table)

Behavior Qualifiers partition the control record into sub-aggregates that give service
operations an unambiguous scope (umbrella §4.3 — "in DDD terms, the sub-aggregates"; the
fractal pattern: partitions carry the same characteristics as their parent and stay MECE). In
the OpenAPI specs each BQ appears as a path segment (`/{SD}/{cr-id}/{BQ}/{bq-id}/{Operation}`)
and as a tag (`BQ (DDD Aggregate) - <Name>`). Everything in the table below is a **verified BQ
name extracted from the published specs** — none invented.

### 6.2 The BQ table — domain / control record / BQs (verified) / notes

| Domain | Control record (CR) | BQs (verified, r13 unless noted) | Notes |
|---|---|---|---|
| Current Account | `CurrentAccountFacility` | `AmountBlock`, `BookingAuthorization`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `ServiceFee`, `Sweep`, `Withdrawal` (r13); r14: `AmountBlock`, `Booking`, `Charge`, `DebitandCredit`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `Sweep` | The classic account BQ set — deposits/withdrawals/payments in, interest/fees out, blocks and sweeps as controls; r14 renamed BookingAuthorization→Booking, ServiceFee→Charge, merged Withdrawal+Deposit into DebitandCredit |
| Corporate Current Account | `CorporateCurrentAccountFacility` | Identical to Current Account (r13 and r14) | The corporate variant shares the qualifier set — the difference is facility terms, not sub-aggregates |
| Virtual Account | `VirtualAccountFacility` | `AccountStatement`, `AmountBlock`, `BookingAuthorization`, `DirectDebit`, `Interest`, `IssuedDevice`, `ServiceFee`, `StandingOrder`, `Statement` | Note the embedded payment qualifiers (DirectDebit, StandingOrder) — sub-account services run *inside* the virtual account |
| Internal Bank Account | `InternalBankAccountFacility` | `BookingAuthorization` | Minimal — internal books need authorization, nothing else |
| Position Keeping | `FinancialPositionLog` | `FinancialTransactionCapture` | The single most important BQ in the cluster: *transaction capture* is a qualifier, not a domain (§4.1) |
| Account Reconciliation | `AccountReconciliationProcedure` | `AccountAssessment`, `AccountResolution` | Assess the difference; resolve the difference — the two-step reconciliation model |
| Payment Instruction | `PaymentInstructionProcedure` | `AgreementConfirmation`, `ComplianceCheck`, `PaymentInstruction` | Compliance is a first-class sub-aggregate — screening/limits live here |
| Payment Execution | `PaymentExecutionProcedure` | `PaymentExecution`, `PaymentMechanism` (v13 InSite also lists BQ type "Workstep"; BQs shown on the SD page: `Payment Execution Workstep`, `Payment Mechanism`) | The mechanism selection (SEPA vs SWIFT vs intrabank) is a qualifier |
| Payment Order | `PaymentOrderProcedure` | `ExecutionInitiation`, `OrderConfirmation` | Order → initiation → confirmation lifecycle |
| Direct Debit | `DirectDebitFacility` | r13: `BatchDebit`, `FundsAvailableCheck`, `PaymentInitiation`, `PaymentTracking`, `Reporting`, `VerifyMandate`; r14: `PaymentBatch`, `Reporting` | The umbrella's own §4.3 example (PaymentTracking, FundsAvailableCheck, PaymentInitiation, VerifyMandate) — verified; r14 simplified the set |
| Direct Debit Mandate | `DirectDebitMandateDirectoryEntry` | `MandateRegistration` | Single qualifier; Register-based domain |
| Standing Order | `StandingOrderFacility` | `StandingOrderPayment` | Each generated payment is a standing-order-payment sub-record |
| Cash Concentration | `AccountBalanceSweepingFacility` | `CashTransfer` | The sweep leg is the qualifier — one facility, many transfers |
| Financial Gateway | `FinancialGatewayOperatingSession` | `Inbound`, `Outbound` | Messaging split by direction (r13-era) |
| Payment Rail (r14) | `PaymentRailOperatingSession` | `InboundTransaction`, `OutboundTransaction`, `PaymentAccountReconciliation`, `PaymentClearingandSettlement` | The r14 rail domain — direction + reconciliation + clearing/settlement qualifiers |
| Payment Order Initiation (r14) | `PaymentOrderInitiationTransaction` | `Compliance`, `Confirmation`, `OrderInitiation` | The r14 initiation domain — compliance check and confirmation as qualifiers |
| Cash Management And Account Services (r14) | `CashManagementAndAccountServicesFacility` | `AmountBlock`, `Charge`, `DebitandCredit`, `Deposit`, `Interest`, `IssuedDevice`, `Payment`, `PositivePay`, `Sweep` | The corporate cash-management wrapper — Positive Pay is the corporate-specific qualifier (cheque positive pay) |
| Cheque Processing | `ChequeProcessingOperatingSession` | `ChequeExceptionHandling`, `ChequeExtraction`, `FinancialTransactionInitiation` | Capture/extraction/exception + initiation |
| Transaction Engine | `TransactionScheduleFacility` | (none in the spec) | A pure procedure domain — operations on the CR only |
| Direct Debit (r14) | `DirectDebitFacility` | `PaymentBatch`, `Reporting` | Rationalized in 14.0 |

**BQ patterns worth naming (all verified in the tables above):**

- **The account qualifier set recurs** — `AmountBlock`/`Payment`/`Sweep`/`Interest`/`Charge`
  appear on Current Account, Corporate Current Account, Virtual Account and the r14 CMAS
  facility. BIAN reuses qualifier *types* across domains (the fractal pattern of umbrella
  §4.3) — a bank's API consumers learn one account vocabulary and it works across products.
- **Directional qualifiers for sessions** — `Inbound`/`Outbound` (Financial Gateway),
  `InboundTransaction`/`OutboundTransaction` (Payment Rail): operating sessions partition by
  message direction.
- **Process-step qualifiers for procedures** — `ComplianceCheck`, `VerifyMandate`,
  `FundsAvailableCheck`, `AccountAssessment`, `AccountResolution`: PROCESS-pattern domains
  decompose into work steps (the BQ type "Workstep" confirmed on Payment Execution's InSite
  page).
- **A BQ can be a product feature** — `PositivePay` (r14 CMAS) and `Sweep` (accounts) are
  qualifiers that, in vendor products, are sold as named features. The BIAN model expresses
  features as sub-aggregates of the facility — which is precisely how a DDD implementation
  would model them ([`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md)).

---

## 7. The Semantic APIs — REST per Domain

### 7.1 The API standard, in four facts (from the umbrella §9, re-verified)

1. A **BIAN Semantic API** = the service operations of one service domain, REST-shaped
   (umbrella §9.1). The service domain is the application boundary; the operations are the
   program interfaces.
2. **Path structure** (canonical, umbrella §9.2): `/{ServiceDomain}/{cr-ref-id}/{BQ}/{bq-ref-id}/{Operation}`
   — e.g. `/DirectDebit/{id}/FundsAvailableCheck/{id}/Execute`.
3. **REST mapping**: POST for Initiate/Register/Evaluate; PUT for Update/Control/Exchange/
   Execute/Request/Grant/Capture; GET for Retrieve; AsyncAPI subscriptions for Notify.
4. **The official repository** (`github.com/bian-official/public`, Apache-2.0) publishes every
   release; the ISO20022+DDD variant is the primary one since 12.0 (umbrella §9.3). Verified
   this pass: r12 = 247 specs, r13 = 251, r14 = 259 (oas3 yamls in
   `apis-iso20022_ext-ddd/oas3/yamls/`), plus a slimmer `semantic-apis` variant and AsyncAPI 3.x
   event specs per release.

### 7.2 The API table — domain / canonical API resource / notes

Representative canonical resources and operations, verified from the published specs (r13 names
unless noted; paths abbreviated to the CR + one BQ example):

| Domain | Canonical API (path shape, verified) | Notes |
|---|---|---|
| Payment Instruction | `POST /PaymentInstruction/Initiate`; `PUT /PaymentInstruction/{id}/Execute`; `GET /PaymentInstruction/{id}/Retrieve`; `…/ComplianceCheck/{id}/Execute` | 29 paths — the richest payment API in r13; the full action-term surface |
| Payment Execution | `POST /PaymentExecution/Initiate`; `GET /PaymentExecution/{id}/PaymentMechanism/{id}/Retrieve` | The r13 execution engine; 6 paths |
| Payment Order | `POST /PaymentOrder/Initiate`; `GET /PaymentOrder/{id}/OrderConfirmation/{id}/Retrieve` | The customer-facing order; 6 paths |
| Payment Order Initiation (r14) | `POST /PaymentOrderInitiation/Initiate`; `PUT …/OrderInitiation/{id}/Exchange`; `GET …/Compliance/{id}/Retrieve`; `GET …/Confirmation/{id}/Retrieve` | r14 replacement for the initiation side; 7 paths |
| Payment Rail (r14) | `POST /PaymentRail/Initiate`; `PUT …/OutboundTransaction/{id}/Exchange`; `PUT …/PaymentClearingandSettlement/{id}/Execute`; `GET …/PaymentAccountReconciliation/{id}/Retrieve` | The r14 rail-facing session domain; 36 paths |
| Direct Debit | `POST /DirectDebit/Initiate`; `PUT /DirectDebit/{id}/PaymentTracking/Initiate`; `PUT …/FundsAvailableCheck/{id}/Execute`; `GET /DirectDebit/{id}/Retrieve` | r13: 21 paths (BQ-rich); r14: 11 paths |
| Direct Debit Mandate | `POST /DirectDebitMandate/Register`; `PUT …/MandateRegistration/{id}/Execute`; `PUT …/MandateRegistration/{id}/Exchange` | The Register-based (CATALOG) API |
| Standing Order | `POST /StandingOrder/Initiate`; `PUT …/StandingOrderPayment/Initiate`; `PUT /StandingOrder/{id}/Execute` | 11 paths |
| Financial Gateway | `POST /FinancialGateway/Initiate`; `POST …/Outbound/Initiate`; `GET …/Inbound/{id}/Retrieve` | The r13 messaging session; 9 paths |
| Current Account | `POST /CurrentAccount/Initiate`; `PUT …/Sweep/{id}/Execute`; `PUT …/ServiceFee/Initiate`; `GET …/Interest/{id}/Retrieve` | 34 paths; r14 variant uses Charge/DebitandCredit/Booking |
| Corporate Current Account | Same shape as Current Account | 34 paths |
| Virtual Account | `POST /VirtualAccount/Initiate`; `PUT /VirtualAccount/{id}/Capture`; `PUT …/StandingOrder/{id}/Capture`; `PUT …/DirectDebit/{id}/Control` | 48 paths — the largest account API |
| Internal Bank Account | `POST /InternalBankAccount/Initiate`; `PUT /InternalBankAccount/{id}/Capture`; `PUT …/BookingAuthorization/{id}/Execute` | 7 paths |
| Position Keeping | `POST /PositionKeeping/Initiate`; `PUT …/FinancialTransactionCapture/{id}/Capture`; `GET …/FinancialTransactionCapture/{id}/Retrieve` | 7 paths — balance-as-a-service |
| Account Reconciliation | `POST /AccountReconciliation/Initiate`; `POST …/AccountAssessment/Initiate`; `PUT …/AccountResolution/{id}/Execute` | 22 paths |
| Cash Concentration | `PUT /CashConcentration/{id}/Activate`; `POST …/CashTransfer/Initiate`; `PUT …/CashTransfer/{id}/Execute`; `PUT /CashConcentration/{id}/Control` | 14 paths (r13), 16 (r14) — the sweep engine API |
| Cash Management And Account Services (r14) | `POST /CashManagementAndAccountServices/Initiate`; `PUT …/Sweep/{id}/Execute`; `PUT …/PositivePay/{id}/Exchange`; `PUT …/Payment/{id}/Execute`; `GET …/Interest/{id}/Retrieve` | 34 paths — the corporate cash-management product API |
| Cheque Processing | `POST /ChequeProcessing/Initiate`; `PUT …/ChequeExceptionHandling/Initiate`; `GET …/ChequeExtraction/{id}/Retrieve` | 16 paths |
| Transaction Engine | `POST /TransactionEngine/Initiate`; `PUT /TransactionEngine/{id}/Execute`; `PUT /TransactionEngine/{id}/Control` | 6 paths |

**API-governance notes for the architect (cross-ref umbrella §9.4):**

- **The spec is the contract.** Every API above is published as OpenAPI 3.x YAML on GitHub —
  clone `release14.0.0/` and you have machine-readable contracts for 259 domains. Your API
  catalog becomes a projection of the capability map; BIAN conformance = your OpenAPI specs
  follow the path/verb/qualifier conventions. This plugs directly into the spec-driven
  practice of [`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md)
  and umbrella §9.4.
- **Choose your variant deliberately.** `apis-iso20022_ext-ddd` (ISO 20022-mapped, DDD-annotated
  — "CR (DDD Aggregate)", "BQ (DDD Aggregate)") is the primary variant since 12.0; `semantic-apis`
  is the BIAN-native BOM model. The r14 `semantic-apis` specs are slim (mostly `Initiate`) while
  the ISO20022+DDD specs carry the full operation sets — a practical reason to target the
  ISO20022+DDD variant (umbrella §9.3).
- **Server URLs in the specs point at SwaggerHub** (e.g. `virtserver.swaggerhub.com/BIAN-3/
  PaymentExecution/12.0.1` ✅) — useful for quick mock/experimentation against BIAN-shaped
  contracts.
- **Events exist.** Each release also ships AsyncAPI 3.x specs; `Notify` operations on Payment
  Instruction, Cash Concentration, Account Reconciliation and Payment Rail are the event seams
  (umbrella §14.3; [`../technology/event_stream_processing_guide.md`](../technology/event_stream_processing_guide.md)).

---

## 8. The Worked Example — a BIAN-Aligned Cash-Management API Design

### 8.1 The scenario — a Cymbal Bank corporate bank (the familiar context)

**The bank:** a Cymbal Bank wholesale/corporate banking arm (see
[`credit_agricole_software_systems_guide.md`](credit_agricole_software_systems_guide.md) §3/§10 —
the CIB estate, the Singapore APAC hub serving corporates and financial institutions). It runs a
legacy host core (accounts, GL) wrapped by a service layer, and it is replacing its
point-to-point corporate cash-management interfaces with a BIAN-aligned API platform. The
adoption posture is umbrella §13.1 **Level 2–3**: *align new interfaces to BIAN service
operations and semantics; begin componentizing around service domains* — not a full Level-4
BIAN-native rewrite.

**The client:** a regional corporate treasurer (e.g. a shipping/commodities group with a
Singapore treasury centre) who manages: a multicurrency corporate current account; subsidiary
accounts across the region; sweeping to a concentration account at end of day; EUR and USD
payments to suppliers; direct-debit collections from customers; and daily reconciliation
against bank statements.

**The design target (BIAN terms):** the corporate cash-management platform = the **Cash
Management And Account Services** wrapper (r14) over the **account cluster** (Corporate Current
Account, Virtual Account), driving the **payment chain** (Payment Order Initiation → Payment
Orchestration → Payment Rail), keeping books via **Position Keeping** and **Account
Reconciliation**, concentrating via **Cash Concentration**. All mapped to the r14/13 spec
shapes verified in §2–§7.

### 8.2 The payment-execution flow design — endpoint by endpoint

The flagship flow: the treasurer initiates a EUR supplier payment of €250,000, with an
end-of-day sweep of subsidiary balances. Every call below uses verified BIAN domain/CR/BQ/verb
names; the paths follow the umbrella §9.2 canonical structure. (r14 domain names for the
payment chain, r13-vintage names noted where the r14 spec is slimmer.)

**Step 1 — Initiate the payment order (customer-facing entry):**

```
POST /PaymentOrderInitiation/Initiate
{
  "payerAccount": "CorporateCurrentAccountFacility/…/SG-CCY-1001",
  "payee": { "name": "Supplier GmbH", "iban": "DE89…" },
  "amount": 250000.00, "currency": "EUR",
  "mechanism": "SEPA_CT",            // ISO 20022 pacs.008 semantics
  "executionDate": "2026-08-25"
}
→ 201 { "paymentOrderInitiationId": "POI-88231", "status": "Initiated" }
```

The CR is `PaymentOrderInitiationTransaction` (r14 ✅). The response returns the aggregate id —
the only id the client ever holds.

**Step 2 — Compliance and confirmation (the BQ-scoped sub-aggregates):**

```
PUT /PaymentOrderInitiation/POI-88231/Compliance/{complianceId}/Retrieve   // read screening status
GET /PaymentOrderInitiation/POI-88231/Confirmation/{confirmationId}/Retrieve
```

The r14 domain models compliance and confirmation as **BQs** (verified §6.2) — sanctions/
AML screening and the client's confirmation of terms are sub-aggregates of the initiation
transaction, not separate systems. In the r13 shape this was Payment Instruction's
`ComplianceCheck` BQ with `Execute` — the same vocabulary, one release apart (the migration
note for the team: map r13 `ComplianceCheck/Execute` → r14 `Compliance/Retrieve`-plus-orchestration).

**Step 3 — Orchestrate and execute (Payment Orchestration / Payment Rail):**

```
PUT /PaymentRail/{railId}/OutboundTransaction/{otId}/Exchange   // submit to SEPA via the rail session
   ← rail ack, status: "Accepted"
PUT /PaymentRail/{railId}/PaymentClearingandSettlement/{pcsId}/Execute  // clearing/settlement step
```

The bank's connectivity lives in the `PaymentRailOperatingSession` (r14 ✅, CR + BQs verified)
— outbound formatting, store-and-forward, clearing/settlement — replacing the old Financial
Gateway `Outbound` session (r13). In a real hub this is where
[`payments_hub_guide.md`](payments_hub_guide.md) plugs in: the rail domain is the BIAN-shaped
boundary in front of the SWIFT/SEPA/FAST adapters.

**Step 4 — Capture the movement into positions (the books stay honest):**

```
PUT /PositionKeeping/{positionId}/FinancialTransactionCapture/{ftcId}/Capture
  { "debit": "…/SG-CCY-1001", "credit": "…/EUR-NOSTRO", "amount": 250000.00, "currency": "EUR" }
GET /CorporateCurrentAccount/{ccaId}/Retrieve          // client sees the updated balance
```

`PositionKeeping` (CR `FinancialPositionLog`, BQ `FinancialTransactionCapture` ✅) is the
balance-of-record; the account facility is updated via its own `DebitandCredit`/`Payment` BQs
(r14). Note the clean separation: the *facility* is the contract, the *position log* is the
state — a pattern that maps to a transaction/position table split in the ledger
([`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md)).

**Step 5 — Reconcile (the daily close):**

```
POST /AccountReconciliation/Initiate  →  reconcile internal vs SEPA statement
POST /AccountReconciliation/{id}/AccountAssessment/Initiate   // quantify the difference
PUT  /AccountReconciliation/{id}/AccountResolution/{arId}/Execute  // resolve/adjust
```

**Step 6 — Concentrate (the end-of-day sweep):**

```
PUT  /CashConcentration/{sweepFacilityId}/Activate          // facility already configured
POST /CashConcentration/{sweepFacilityId}/CashTransfer/Initiate  // zero-balance the subsidiary
PUT  /CashConcentration/{sweepFacilityId}/CashTransfer/{ctId}/Execute
GET  /CashManagementAndAccountServices/{cmasId}/Sweep/{sweepId}/Retrieve   // client view
```

The sweep facility (CR `AccountBalanceSweepingFacility` ✅) executes the `CashTransfer` legs;
the treasurer's dashboard reads the corporate wrapper domain (`CashManagementAndAccountServices`
✅) — its `Sweep` BQ, `PositivePay` BQ (cheque control), `AmountBlock` BQ (reserve
requirements), `Payment` BQ. This is the *product* API the corporate client's treasury
workstation consumes; the platform APIs behind it are the payment/position/reconciliation
domains.

**The full sequence, in one line (BIAN vocabulary):**

```
PaymentOrderInitiation.Initiate → Compliance/Confirmation → PaymentRail.OutboundTransaction.Exchange
→ PositionKeeping.FinancialTransactionCapture.Capture → AccountReconciliation.AccountAssessment/Resolution
→ CashConcentration.CashTransfer.Execute → CashManagementAndAccountServices.Sweep.Retrieve
```

### 8.3 The lessons

1. **The vocabulary does the design work.** Every step above was named from verified BIAN
   elements — no invented domains, no bespoke verbs. The design conversation with business
   ("Positive Pay", "sweep", "reconciliation") and the API contract use the same words.
2. **Version the landscape, not just the APIs.** The same flow differs r13 → r14 (Instruction/
   Execution/Order → Initiation/Orchestration/Rail; Direct Debit BQs simplified; the CMAS
   wrapper added). Pin your target release (r14 is current, Feb–Mar 2026 ✅) and keep the r13
   specs as the semantic reference — both are on GitHub indefinitely (umbrella §9.3).
3. **BQs are where the product lives.** The features corporate treasurers buy — sweeps, positive
   pay, amount blocks, compliance checks — are **behavior qualifiers** in BIAN, i.e.
   sub-aggregates with their own lifecycles. Implement them as sub-resources with their own
   endpoints; that is what makes the API composable (umbrella §5.3, §4.3).
4. **Balance is a service, not a field.** Position Keeping's `FinancialTransactionCapture`
   gives you balance-as-a-service; the account facility stays the contract. This separation
   survives whatever ledger tech you choose underneath ([`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md)).
5. **Events are first-class.** The `Notify` operations on Payment Instruction / Cash
   Concentration / Payment Rail are your AsyncAPI seams (umbrella §14.3) — the treasury
   dashboard subscribes instead of polling (cross-ref
   [`../technology/event_stream_processing_guide.md`](../technology/event_stream_processing_guide.md)).
6. **Vendor evaluation becomes a diff.** Ask the cash-management vendor (Temenos, Oracle
   FLEXCUBE/OBMA, Finastra, SAP — umbrella §15.2) for its Service Domain coverage matrix; the
   gaps on *this* guide's tables are the gaps in your platform.
7. **Decomposition is DDD-ready.** Each domain above is a bounded context; the CR is the
   aggregate root; the BQs are sub-aggregates ([`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md),
   [`../technology/monolith_to_microservices_guide.md`](../technology/monolith_to_microservices_guide.md)) —
   but cluster 3–8 domains per deployable (umbrella §14.2 caveat), don't go one-domain-per-pod.

---

## 9. The One-Page Summary — the Vocabulary of Cash

**The map.** BIAN's cash-management corner is **not one business area** — it is a verified
cluster of service domains across accounts, payments, and cash: the account facilities
(Corporate Current Account, Current Account, Virtual Account, Internal Bank Account), the
position and reconciliation spine (Position Keeping, Account Reconciliation), the r13 payment
chain (Payment Order → Payment Instruction → Payment Execution → Financial Gateway), the r14
payment chain (Payment Order Initiation → Payment Orchestration → Payment Rail), and the
dedicated cash domains (Cash Concentration, and the r14 additions: Cash Management And Account
Services, Notional Pooling, Open Item Management).

**The corrections.** Of the names this brief expected: **Account Administration, Account
Balance, Account Reporting, Account Access, Credit Transfer, Payment Messaging, Liquidity
Management, Sweep Account, Cash Management, Transaction Capture are not BIAN service domains.**
The real decomposition: administration *is* the facility; balance *is* Position Keeping;
reporting *is* Account Reconciliation plus Retrieve; access *is* Customer Access Entitlement;
credit transfer *is* an ISO 20022 message family; messaging *is* Financial Gateway / Payment
Rail; liquidity *is* a capability; sweep *is* a BQ (and Cash Concentration's CR is literally
`AccountBalanceSweepingFacility`); "Cash Management" *is* the r14 CMAS service domain;
transaction capture *is* the `FinancialTransactionCapture` BQ on Position Keeping. Every one of
these corrections was verified against the published specs — and each one is a vocabulary gift:
say "capture a financial transaction on the position log" and every BIAN-literate counterpart
knows exactly what you mean.

**The verbs.** Twelve action terms (Initiate, Register, Evaluate, Update, Control, Exchange,
Execute, Request, Retrieve, Notify, Capture, Activate) cover every operation in the cash
cluster; REST maps them mechanically (POST/PUT/GET/events). **The qualifiers.** BQs such as
`Sweep`, `Payment`, `PositivePay`, `CashTransfer`, `ComplianceCheck`, `FinancialTransactionCapture`,
`AccountAssessment`/`AccountResolution`, `Inbound`/`Outbound` carry the actual product and
process semantics — the BQ table (§6.2) is the single most reusable artifact in this guide.
**The APIs.** 259 OpenAPI specs in r14 (247→251→259 across r12→r13→r14), ISO20022+DDD variant,
all on GitHub Apache-2.0; path shape `/{SD}/{cr-id}/{BQ}/{bq-id}/{Operation}`.

**The release.** r14.0 (Feb–Mar 2026, ✅ bian.org): payment domains rationalized
(PaymentExecution/Instruction/Order → PaymentOrderInitiation + PaymentRail + PaymentOrchestration),
Direct Debit simplified to `PaymentBatch`/`Reporting`, and the new **Cash Management And
Account Services** domain — the corporate cash-management wrapper — plus Notional Pooling and
Open Item Management.

**The final word — the vocabulary of cash.** BIAN's gift to a corporate-banking architect is
not the framework — it is the **vocabulary of cash**: a controlled, verified set of names for
the accounts, payments, positions, reconciliations, and sweeps that a corporate bank runs
every day, stable across releases, shared across vendors, and expressive enough that "execute a
cash transfer leg on the sweep facility" is a complete, unambiguous sentence. Learn the words;
the architecture follows. And where the vocabulary disagrees with your habit — where "account
balance" is really a position log, and "transaction capture" is really a qualifier — trust the
vocabulary: that disagreement is precisely where the standardization earns its keep.

---

## 10. Glossary

| Term | Definition |
|---|---|
| **BIAN** | Banking Industry Architecture Network — the open, member-owned standard for banking IT architecture (est. 2008, Frankfurt). See umbrella §1. |
| **Banking Industry Architecture Network** | The full name of BIAN; a not-for-profit association publishing the Service Landscape, the BOM, the meta model, and Semantic API specifications. |
| **Service Landscape** | BIAN's core artifact: the complete map of the bank as ~320+ MECE service domains under Business Areas → Business Domains → Service Domains (umbrella §3). |
| **Service Domain** | The elemental building block — one functional pattern applied to one asset type for its full lifecycle; a bounded context in DDD terms (umbrella §4, this guide throughout). |
| **CMAS** | Cash Management and Account Services — used here as the cluster name for the account/payment/cash service domains; also the exact name of the new r14 service domain (verified). |
| **Cash Management and Account Services** | The r14 service domain (CR `CashManagementAndAccountServicesFacility`) orchestrating corporate cash-management features over standard account facilities — verified this pass. |
| **Account Administration** | ⚠ Not a BIAN service domain — administration is embodied in the account facility domains (Current/Corporate Current/Virtual/Internal Bank Account). |
| **Account Balance** | ⚠ Not a BIAN service domain — balance is the state maintained by Position Keeping (`FinancialPositionLog`). |
| **Account Reporting** | ⚠ Not a BIAN service domain — delivered via Account Reconciliation, the `Reporting` BQ (Direct Debit), and Retrieve operations. |
| **Account Access** | ⚠ Not a BIAN service domain — access entitlements live in Customer Access Entitlement. |
| **Payment Execution** | Verified service domain (r9.1–r13): CR `PaymentExecutionProcedure`, BQs `PaymentExecution`/`PaymentMechanism`; merged in r14. |
| **Payment Order** | Verified service domain (r9.1–r13): CR `PaymentOrderProcedure`, BQs `ExecutionInitiation`/`OrderConfirmation`; superseded by Payment Order Initiation in r14. |
| **Direct Debit** | Verified service domain (all releases): CR `DirectDebitFacility`; r13 BQs include `VerifyMandate`, `FundsAvailableCheck`, `PaymentInitiation`, `PaymentTracking`, `Reporting`, `BatchDebit`; r14 BQs `PaymentBatch`/`Reporting`. |
| **Credit Transfer** | ⚠ Not a BIAN service domain — an ISO 20022 message family (pacs.008) handled by the payment chain domains. |
| **Standing Order** | Verified service domain: CR `StandingOrderFacility`, BQ `StandingOrderPayment`. |
| **Payment Messaging** | ⚠ Not a BIAN service domain — the connectivity function is Financial Gateway (r13, BQs `Inbound`/`Outbound`) and Payment Rail (r14). |
| **Cash Concentration** | Verified service domain: CR `AccountBalanceSweepingFacility`, BQ `CashTransfer` — the sweep/concentration engine. |
| **Cash Management** | ⚠ Not a standalone service domain — the r14 CMAS domain and the capability layer cover it. |
| **Liquidity Management** | ⚠ Not a service domain — a verified BIAN *capability* ("monitor, forecast and influence liquidity and cash positions…"); see [`treasury_alm_guide.md`](treasury_alm_guide.md). |
| **Sweep** | Not a domain — a verified **Behavior Qualifier** on Current/Corporate Current/CMAS facilities; Cash Concentration's CR is the `AccountBalanceSweepingFacility`. |
| **Position Keeping** | Verified service domain: CR `FinancialPositionLog`, BQ `FinancialTransactionCapture` — the balance/position-of-record domain. |
| **Reconciliation** | The function delivered by the verified Account Reconciliation domain (CR `AccountReconciliationProcedure`, BQs `AccountAssessment`/`AccountResolution`) and the r14 `PaymentAccountReconciliation` BQ on Payment Rail. |
| **Transaction Capture** | Not a standalone domain — the verified `FinancialTransactionCapture` BQ on Position Keeping (plus Card Transaction Capture for cards). |
| **Action term** | One of BIAN's controlled operation verbs (Initiate, Register, Evaluate, Update, Control, Exchange, Execute, Request, Retrieve, Notify, Capture, Activate, …) — umbrella §5. |
| **Behavior qualifier (BQ)** | A sub-aggregate partition of a service domain's control record that scopes operations — umbrella §4.3; the verified BQ table is §6.2 of this guide. |
| **BQ** | Abbreviation of Behavior Qualifier. |
| **Semantic API** | The REST-shaped (and event-shaped) API of one BIAN service domain — umbrella §9. |
| **REST** | The HTTP style BIAN APIs follow: POST for creation terms, PUT for change terms, GET for Retrieve, subscriptions for Notify — umbrella §5.2/§9.2. |
| **Business object** | A conceptual business entity in the BIAN BOM (Party, Customer, Account, Product, Agreement, Transaction…) exchanged across service boundaries — umbrella §6. |
| **Control record** | The domain's aggregate root / master record (e.g. `CorporateCurrentAccountFacility`) — umbrella §4.1/§4.3. |
| **ISO 20022** | The international standard for financial message semantics (e.g. pacs.008 credit transfer, pain.008 direct debit) that BIAN maps to — umbrella §11.2. |
| **Corporate banking** | The wholesale banking business (transaction banking, cash management, trade finance, treasury) that this guide's CMAS cluster serves. |

---

## 11. References and Further Reading

### Primary sources (verified this pass)
- **BIAN Service Landscape v13 / v14** — bian.org/servicelandscape-13-0-0/ and bian.org/servicelandscape-14-0-0/ (InSite: overview diagrams, the "Clearing And Settlement" view, Payment Execution SD page, Liquidity Management capability)
- **BIAN Service Landscape page** — bian.org/deliverables/service-landscape/ ("BIAN 14.0 has focused on rationalisation of the Service Domain definitions… Some Service Domains have been renamed, removed and added especially in the Payment Area…")
- **BIAN v14.0 Release Notes** — bian.org/wp-content/uploads/2026/02/BIAN-v14.0-Release-Notes-v1.0_-Final-Version.pdf
- **Official GitHub repository** — github.com/bian-official/public (release9.1 → release14.0.0; `apis-iso20022_ext-ddd/oas3/yamls/*.yaml` — the ISO20022+DDD service-domain API specs; `semantic-apis/` variant; AsyncAPI 3.x event specs) — all domain names, control records, BQs, and operations in this guide verified against these files
- **BIAN Portal** — portal.bian.org (e.g. `BIAN-14.0.0-PaymentRail`, `BIAN-14.0.0-PaymentOrchestration`)

### In this repository (cross-references)
- [`bian_banking_architecture_guide.md`](bian_banking_architecture_guide.md) — **the umbrella**: §3 landscape, §4 service-domain model, §5 action terms, §6 BOM, §9 API standard, §10 releases, §13 adoption, §15 vendors. Read first; this guide deepens its cash-management corner.
- [`payments_hub_guide.md`](payments_hub_guide.md) — the payments-hub architecture the BIAN payment chain plugs into (rails, ISO 20022, MAS/PSA)
- [`core_banking_systems_guide.md`](core_banking_systems_guide.md) — the account/payment domains in the core; composable banking
- [`treasury_alm_guide.md`](treasury_alm_guide.md) — the liquidity-management angle (lightly)
- [`nets_singapore_guide.md`](nets_singapore_guide.md) / [`singapore_fintech_payments_guide.md`](singapore_fintech_payments_guide.md) — the payments rails (lightly)
- [`capital_markets_architecture_guide.md`](capital_markets_architecture_guide.md) — cross-ref (lightly)
- [`credit_agricole_software_systems_guide.md`](credit_agricole_software_systems_guide.md) — the Cymbal Bank context for §8
- [`../technology/domain_driven_design_guide.md`](../technology/domain_driven_design_guide.md) — service domain as bounded context
- [`../technology/monolith_to_microservices_guide.md`](../technology/monolith_to_microservices_guide.md) — the decomposition angle
- [`../technology/event_stream_processing_guide.md`](../technology/event_stream_processing_guide.md) — the payment-event angle
- [`../technology/cockroachdb_guide.md`](../technology/cockroachdb_guide.md) — the ledger/persistence angle (lightly)
- [`../management/mba_body_of_knowledge_guide.md`](../management/mba_body_of_knowledge_guide.md) — cross-ref (lightly)

### Vendor/adoption material (flagged ⚠ where not re-verified this pass)
- Temenos Transact ↔ BIAN Service Domain mappings (BIAN member; Coreless Banking participant — umbrella §15.2)
- Oracle Banking Microservices Architecture (OBMA) BIAN mapping — see umbrella §15.2
- Finastra FusionFabric.cloud BIAN alignment; FIS; IBM; SAP (SAP community: "Architecture Decisions for Cash and Liquidity Management" — the FSCM Cash Management / Liquidity Management module view of the same capability cluster ⚠)
- Press: Finextra / The Banker BIAN-adoption coverage — cross-ref umbrella §21 ⚠ (not re-fetched this pass)

---

*Verification honesty note: every service-domain name, control record, behavior qualifier, and
operation in this guide was verified this pass against the official BIAN GitHub repository
(release 12.0/13.0/14.0 ISO20022+DDD OpenAPI specs) and/or bian.org InSite pages. Where a
task-expected name does not exist in BIAN's published landscape (Account Administration,
Account Balance, Account Reporting, Account Access, Credit Transfer, Payment Messaging,
Liquidity Management-as-domain, Sweep-as-domain, Cash Management-as-domain, Transaction
Capture-as-domain), it is flagged ⚠ and the real BIAN element is named. Nothing is fabricated.*
