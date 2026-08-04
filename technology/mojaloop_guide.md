# Mojaloop: A Comprehensive Guide to the Open-Source Real-Time Payment Switch

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Payments Architecture / Financial Inclusion — Real-Time Payments, Interoperability Switches, FSPIOP API, Open Source (Mojaloop Foundation)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Related Guides:** [payments_hub_guide.md](payments_hub_guide.md) · [apache_fineract_guide.md](apache_fineract_guide.md) · [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) · [container_certificates_guide.md](container_certificates_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is Mojaloop?](#1-what-is-mojaloop)
2. [The Switch Concept](#2-the-switch-concept)
3. [Design Goals and Core Principles](#3-design-goals-and-core-principles)
4. [Mojaloop vs RTGS and Other Payment Rails](#4-mojaloop-vs-rtgs-and-other-payment-rails)
5. [The FSPIOP API: The Interoperability Standard](#5-the-fspiop-api-the-interoperability-standard)
6. [The Mojaloop Hub Architecture](#6-the-mojaloop-hub-architecture)
7. [Core Services Deep Dive](#7-core-services-deep-dive)
8. [The Mojaloop SDK (Scheme Adapter)](#8-the-mojaloop-sdk-scheme-adapter)
9. [The P2P Transfer Flow, Step by Step](#9-the-p2p-transfer-flow-step-by-step)
10. [The Transfer Lifecycle and State Machine](#10-the-transfer-lifecycle-and-state-machine)
11. [Settlement Models: DNS and RTGS Integration](#11-settlement-models-dns-and-rtgs-integration)
12. [Liquidity, Positions, and Limits](#12-liquidity-positions-and-limits)
13. [Timeouts, Failures, and Reversals](#13-timeouts-failures-and-reversals)
14. [Bulk Transfers and Government Disbursements](#14-bulk-transfers-and-government-disbursements)
15. [Fraud and Risk Management](#15-fraud-and-risk-management)
16. [Deployment and Operations](#16-deployment-and-operations)
17. [National Deployments and the Global Ecosystem](#17-national-deployments-and-the-global-ecosystem)
18. [DFSP Integration Patterns](#18-dfsp-integration-patterns)
19. [Testing Tools: TTK, Simulator, Postman](#19-testing-tools-ttk-simulator-postman)
20. [Governance, Standards, and Open-Source Process](#20-governance-standards-and-open-source-process)
21. [Mojaloop vs Commercial Alternatives](#21-mojaloop-vs-commercial-alternatives)
22. [Mojaloop in a Banking Context](#22-mojaloop-in-a-banking-context)
23. [Getting Started](#23-getting-started)
24. [Resources](#24-resources)
25. [Glossary](#25-glossary)
26. [Conclusion](#26-conclusion)

---

## 1. What Is Mojaloop?

Mojaloop is an **open-source software platform for real-time, interoperable, account-to-account (A2A) payments**, purpose-built for financial inclusion. It is a reference implementation of an *inclusive instant payment system* (IIPS) — the kind of national retail payment switch that lets a customer of any participating provider send money instantly to a customer of any *other* participating provider, regardless of whether the two providers are banks, mobile money operators, or fintechs.

The name comes from Swahili: **"moja" means "one"** — the Mojaloop Foundation renders "Mojaloop" as **"one loop"**: a single payment loop that connects every digital financial service provider (DFSP) in a country, instead of the fragmented web of bilateral links that typically exists between mobile money wallets, bank accounts, and fintech apps.

### 1.1 Origins: The Level One Project

Mojaloop was created by the **Bill & Melinda Gates Foundation's Level One Project** — an initiative (2016–2017) researching why digital financial services had not scaled for the world's unbanked and underbanked. The project's central finding was that **interoperability is the missing layer**: mobile money had proven that poor people would use digital payments (M-Pesa's success in Kenya), but wallets, bank accounts, and payment systems operated in silos. A customer could rarely send money from a mobile wallet to a bank account, or between two different mobile money operators, without cash-out and cash-in via agents.

The Level One Project concluded that the "developed world" model — a **central switch** through which all providers interoperate (like the national payment switches in Singapore, India, or the UK) — was the right architecture, but that commercial switch vendors were too expensive and too rigid for emerging markets. The answer was to **open-source the reference model**: a royalty-free, vendor-neutral blueprint plus working software that any central bank or scheme operator could adopt, adapt, and operate.

Mojaloop was released as open source in **October 2017**, and the first production-grade pilots ran in Tanzania from 2018 onward.

### 1.2 The Mojaloop Foundation

In **2021**, governance moved from the Gates Foundation to the newly formed **Mojaloop Foundation**, a charitable nonprofit hosted within the **Financial Services for the Poor** ecosystem of the Bill & Melinda Gates Foundation. The founding **Sponsor members** were:

| Member | Role |
|---|---|
| **Bill & Melinda Gates Foundation** | Founding sponsor; mission owner (financial inclusion) |
| **ModusBox** | Primary maintainer and implementation consultancy |
| **Google** | Sponsor; engineering and cloud contributions |
| **Coil** | Sponsor (interoperability/Web Monetization background) |
| **Omidyar Network** | Sponsor (philanthropic investor) |
| **The Rockefeller Foundation** | Sponsor |

The Foundation stewards the software, the FSPIOP API specification, the community, and the adoption program. The broader contributor community spans dozens of organizations — including central bank technology partners, fintechs, and systems integrators (ModusBox has historically been the lead maintainer of most core repositories, with the Gates Foundation and others funding feature work).

### 1.3 License and Governance Model

- **License:** Apache 2.0 — free to use, modify, and embed in commercial products, with no royalty obligations. This is the single most important differentiator vs. commercial switch vendors.
- **Governance:** The Mojaloop Foundation owns the trademark and the specification; the code lives in public GitHub repositories (`github.com/mojaloop/*`) with community review, a documented release process, and semantic versioning.
- **Sustainability model:** Unlike a vendor, the Foundation does not sell licenses — it is funded by sponsors, and commercial revenue accrues to the *adopters* (system integrators, hub operators, DFSPs) who build and operate deployments.

### 1.4 What Mojaloop Is — and Is Not

| Mojaloop **is** | Mojaloop **is not** |
|---|---|
| A reference-model switch (hub) that routes, clears, and settles retail payments between DFSPs | A core banking system (it does not hold customer accounts) |
| An open API standard (FSPIOP) plus a working microservices implementation | A mobile money platform or a bank's payment app |
| A settlement engine with deferred net settlement (DNS) and RTGS integration | A central bank RTGS system for large-value payments |
| Deployable software (Helm charts, Docker) that a hub operator runs | A hosted SaaS — you operate your own instance |
| Apache 2.0 open source | A blockchain / distributed ledger (despite early ILP heritage, it is a centralized switch) |

---

## 2. The Switch Concept

Mojaloop is not a hub for one institution — it is the **interbank switch** that connects *all* participating DFSPs. This is the same concept as a national payment switch:

- **MEPS+** (Singapore) — the MAS-operated interbank transfer system;
- **UPI / NPCI** (India) — the National Payments Corporation of India's unified interface;
- **FAST** (Singapore), **FedNow** (US), **NPP** (Australia) — domestic instant payment rails;
- **TIPS / RT1** (Eurosystem) — Europe's instant settlement rails.

The difference: those are **proprietary systems built by or for specific countries and vendors**, while Mojaloop is **open source and designed from day one for emerging markets** — low-value, high-volume, mobile-first, offline-tolerant, and cheap enough for a small central bank or even a regional body to operate.

### 2.1 Why a Switch Instead of Bilateral Links?

Without a switch, every DFSP must integrate with every other DFSP to achieve interoperability — N×M point-to-point connections. With a switch, each DFSP integrates **once**, to the hub:

```
WITHOUT A SWITCH (N×M)                WITH A SWITCH (N+M)
                                        ┌─────────────┐
   BankA ── BankB ── BankC              │             │── BankA
        ╲      │      ╱                 │  MOJALOOP   │── MobileMoneyCo
   MobileMo ── FintechX                 │   SWITCH    │── FintechX
                                        │   (hub)     │── BankB
                                        │             │── ...
                                        └─────────────┘
```

The switch provides the shared services that make interoperability work:

1. **Directory / alias resolution** — "who owns this phone number / national ID / email, and which DFSP serves them?"
2. **Routing** — forwarding each request to the correct destination DFSP.
3. **Clearing** — recording transfers between DFSPs as positions on the hub's books.
4. **Settlement** — netting those positions and settling them through settlement accounts (typically at the central bank).
5. **Scheme rule enforcement** — fees, limits, fraud reporting, and participant management.

### 2.2 The "Level Playing Field" Property

Because every DFSP talks the same FSPIOP API to the same hub, **no incumbent has a proprietary advantage**: a mobile money operator cannot lock out banks, a big bank cannot dictate terms to fintechs, and a new entrant can join with the same connectivity as the largest participant. This is a *governance* property as much as a technical one — the scheme rules (set by the hub operator, typically the central bank) determine fees, limits, and access; Mojaloop simply makes the technical side neutral and open.

---

## 3. Design Goals and Core Principles

### 3.1 Design Goals

| Goal | What it means in practice |
|---|---|
| **Financial inclusion** | Serve the unbanked/underbanked: low-value (often < USD 5) and high-volume transactions; works on USSD and low-end phones; agent-friendly |
| **Interoperability** | Any DFSP can connect to any other via the switch — bank ↔ mobile money ↔ fintech, using open standards |
| **Real-time** | Instant payment confirmation — the payee's account is credited within seconds, 24/7/365 |
| **Low cost** | Per-transaction economics viable for micro-payments — no license fees, commodity hardware, efficient code |
| **Universal access** | Any licensed provider can join on equal terms; multiple channels (app, USSD, agent, QR) |
| **Level playing field** | No incumbent advantage; open API; transparent scheme rules |

### 3.2 Level One Project Principles

The architecture derives from the Level One Project's published principles for inclusive payment systems:

1. **Real-time payments** — funds move and confirm immediately.
2. **Interoperability via open standards** — a single, open API (FSPIOP) that any provider can implement.
3. **Harmonized regulatory framework** — the scheme operates under clear oversight (typically the central bank) with consistent rules for all participants.
4. **Low-cost infrastructure** — open-source software, commodity infrastructure, and competitive hosting to keep per-transaction costs near zero.
5. **Accessible to all providers** — no exclusivity, no minimum-size barriers, transparent onboarding.

These principles are why Mojaloop looks like UPI or MEPS+ in *shape* but is radically different in *economics*: the marginal cost of adding a DFSP is an integration project, not a license negotiation.

---

## 4. Mojaloop vs RTGS and Other Payment Rails

A common confusion is Mojaloop vs a **central bank RTGS** (Real-Time Gross Settlement). They are complementary, not competing:

| Dimension | Central Bank RTGS | Mojaloop (retail switch) |
|---|---|---|
| **Purpose** | Large-value, interbank, time-critical settlements | Retail, low-value, high-volume person-to-person (P2P), P2M, G2P |
| **Value profile** | Millions–billions per payment | Pennies–thousands per payment |
| **Speed** | Real-time gross (per transaction, continuous) | Real-time *perception* — instant confirmation to customers |
| **Settlement** | Gross, real-time, final (each payment settles individually) | **Deferred net settlement (DNS)** — transfers net over a window; or RTGS integration for the net positions |
| **Participants** | Banks (and sometimes large NBFIs) | Any licensed DFSP: banks, mobile money operators, fintechs |
| **Access** | Usually bank-only | Universal — designed for financial inclusion |
| **Role of Mojaloop** | Mojaloop does **not** replace RTGS | Mojaloop orchestrates retail clearing; the *net settlement* can be executed **through the RTGS** via the Settlement API |

In a typical national deployment, the flow is: **customer → DFSP app → Mojaloop switch (routing + clearing + positions) → settlement window closes → central-settlement computes net positions → net obligations paid via RTGS or commercial settlement accounts at the central bank → finality**.

Mojaloop can also be configured for **real-time gross settlement** for deployments that integrate directly with an RTGS, but the reference model is DNS.

### 4.1 Mojaloop vs Other Rails (Summary)

| Rail | Interoperability model | Open source? | Financial inclusion focus? | Typical operator |
|---|---|---|---|---|
| **Mojaloop** | Open FSPIOP API, hub-and-spoke | ✅ Apache 2.0 | ✅ Primary mission | Central bank / scheme operator |
| **National IPS (MEPS+, UPI, FedNow, FAST)** | Proprietary specs, hub-and-spoke | ❌ | Varies (UPI yes; MEPS+/FedNow mainstream) | Central bank / designated operator |
| **Card networks (Visa, Mastercard)** | Proprietary, 4-party model, interchange fees | ❌ | Partial (via card programs) | Private networks |
| **RTGS** | Gross settlement at central bank | ❌ | ❌ (large-value) | Central bank |
| **Mobile money (single operator)** | Closed loop, own network | ❌ | ✅ | Telco |

---

## 5. The FSPIOP API: The Interoperability Standard

The **FSPIOP API** (Financial Service Provider Interoperability API — also called the *Mojaloop API*) is the heart of the platform: the OpenAPI-defined contract between DFSPs and the switch. Every DFSP that joins a Mojaloop-based scheme implements this API (directly or via the SDK). Because the API is open and versioned (v1.0, v1.1), a DFSP that has integrated once can in principle connect to *any* Mojaloop-based hub.

### 5.1 Core Resources

| Resource | Method(s) | Purpose |
|---|---|---|
| `/parties/{Type}/{ID}` | POST, GET | **Party lookup** — resolve an account identifier (phone number, national ID, email) to a party + the DFSP that serves it (alias resolution, the "phonebook") |
| `/quotes`, `/quotes/{ID}` | POST, PUT, GET | **Quote request/response** — compute the transfer: fees, FX, amounts, expiration; status |
| `/transfers` | POST | **Transfer execution** — prepare the transfer (reserve funds) |
| `/transfers/{ID}` | PUT, GET | Transfer status / callback |
| `/transfers/{ID}/fulfilment` | POST | **Transfer fulfilment** — commit the transfer (payee DFSP confirms credit) |
| `/transfers/{ID}/reversals` | POST | **Reversal** — undo a completed transfer |
| `/transactionRequests` | POST | **Request to pay** — payee initiates, asking the payer to authorize payment |
| `/transactions/{ID}` | GET | Transaction status queries |
| `/bulkQuotes`, `/bulkTransfers` | POST | Batch quotes and transfers (bulk API) |
| `/authorizations` | POST | Customer authorization callbacks (e.g., for third-party / OTP flows) |
| `/settlements`, `/settlementWindows` | GET, POST | Settlement window and settlement queries (Settlement API) |
| `/fx` | POST | FX conversion requests (v1.1+) |

### 5.2 The Asynchronous Request–Response Pattern

The single most important design characteristic: **the switch is asynchronous**. A DFSP sends a POST; the switch acknowledges synchronously with **HTTP 202 Accepted**; the actual result arrives later via a **callback** — the switch (or the counterparty DFSP) sends a **PUT** to the requesting DFSP's registered callback URL.

```
Payer DFSP                  Switch                     Payee DFSP
    │  POST /quotes             │                            │
    │──────────────────────────▶│  POST /quotes (forwarded)  │
    │  202 Accepted             │───────────────────────────▶│
    │                           │  PUT /quotes/{id} ◀────────│◀── quote response
    │  PUT /quotes/{id} ◀───────│                            │
    │◀──────────────────────────│                            │
```

Why asynchronous? Because a quote may require FX quotes, fee calculations, and the payee DFSP's systems — work that can take hundreds of milliseconds or more. Synchronous request/response would couple every DFSP to the slowest participant; the async pattern lets the switch orchestrate long-lived flows without holding connections open, and gives the state machine (central-event-processor) room for retries and timeouts.

Key practical implications for integrators:

- Every outbound request must be **idempotent** and carry a unique ID (quoteId, transferId).
- Every DFSP must expose **callback endpoints** (PUT handlers) and register its callback base URL with the hub.
- The **FSPIOP-Destination** header tells the hub where to route; the switch overwrites/validates it at each hop.

### 5.3 FSPIOP Headers

| Header | Purpose |
|---|---|
| `FSPIOP-Source` | The DFSP initiating the request (the source FSP ID) |
| `FSPIOP-Destination` | The DFSP the request is routed to (set by the switch) |
| `FSPIOP-Signature` | **JWS signature** over the request (headers + body) for non-repudiation |
| `FSPIOP-URI` | The URI being called (used to build the signed payload) |
| `FSPIOP-HTTP-Method` | The HTTP method (used to build the signed payload) |
| `FSPIOP-Key` | The key/thumbprint identifier of the signing certificate |

### 5.4 Security Model

- **mTLS** between every DFSP and the switch (mutual TLS — both sides present certificates).
- **JWS signatures** on every request (`FSPIOP-Signature`) — payload integrity and non-repudiation at the application layer, independent of transport security.
- **PKI / certificate management** — the hub operates a certificate authority or integrates an external PKI; DFSPs are issued signing + TLS certificates; keys are rotated per scheme policy.
- **Client credentials** — DFSPs authenticate with OAuth2 client credentials where the hub supports it (in addition to mTLS).
- **Hub-side controls** — the switch enforces participant status (active/suspended), limits, and IP allow-listing.

---

## 6. The Mojaloop Hub Architecture

Mojaloop is a **Kubernetes-native microservices architecture** — roughly 20+ services in a full deployment, organized around the "Mojaloop Hub" reference architecture.

### 6.1 Technology Stack

| Layer | Technology |
|---|---|
| **Language** | Node.js (TypeScript) — the vast majority of services; some tooling in Python/Go |
| **Messaging** | **Apache Kafka** — the asynchronous event bus between services (event-driven, ordered, replayable) |
| **Databases** | MySQL (primary per-service datastore); PostgreSQL supported for some components; Redis for caching (e.g., alias lookup caching) |
| **Runtime** | Docker containers, orchestrated by Kubernetes |
| **Deployment** | Helm charts (official), Docker Compose (dev/test), Helmfile/Kustomize for advanced config |
| **API** | API-first: every service ships an OpenAPI 3.0 spec |
| **Observability** | Prometheus metrics, Grafana dashboards, ELK stack for logs; optional Istio service mesh |
| **Security** | mTLS, JWS, PKI (cert-manager), OAuth2 client credentials |

### 6.2 Logical Architecture

```
                    ┌────────────────────────────────────────────┐
                    │            DFSPs (participants)            │
                    │  Banks │ Mobile Money │ Fintechs │ Agents  │
                    └───────┬──────────────────────────┬─────────┘
                            │  FSPIOP API (mTLS + JWS) │
                    ┌───────▼──────────────────────────▼─────────┐
                    │          API ADAPTERS (edge layer)         │
                    │   ml-api-adapter │ bulk-api-adapter        │
                    └───────┬──────────────────────────┬─────────┘
                            │        Kafka events
        ┌───────────────────┼───────────────────────────────────┐
┌───────▼────────┐  ┌───────▼────────┐  ┌───────────────────────▼──┐
│ account-lookup │  │  quoting-      │  │  central-event-processor │
│ service (ALS)  │  │  service       │  │  (orchestration/state    │
│ "phonebook"    │  │  (quoter)      │  │   machine, retries)      │
└───────┬────────┘  └───────┬────────┘  └───────────┬───────────────┘
┌───────▼───────────────────▼───────────────────────▼───────────────┐
│                    central-ledger (the heart)                    │
│   transfers │ positions │ balances │ journal entries │ limits    │
└───────┬───────────────────────────────────────────────────────────┘
┌───────▼───────────────────────────────────────────────────────────┐
│                 central-settlement (DNS engine)                  │
│   settlement windows │ net positions │ settlement batches │      │
└───────────────────────────────────────────────────────────────────┘
        │ Settlement API
┌───────▼───────────────────────────────────────────────────────────┐
│   Central bank / RTGS / commercial settlement bank (accounts)    │
└───────────────────────────────────────────────────────────────────┘
Supporting: central-router (routing), transaction-requests-service,
bulk-api-adapter (bulk), fraud, notifications, Ops portal, TTK, SDK
```

### 6.3 How Services Communicate

Services never call each other synchronously in the hot path. Instead:

1. An **API adapter** (ml-api-adapter) receives an FSPIOP request from a DFSP.
2. It validates it, publishes an event to **Kafka** (e.g., `transfer-prepare`).
3. The relevant service (e.g., central-ledger) consumes the event, does its work (e.g., position check, journal entry), and publishes the next event (e.g., `transfer-prepared`).
4. **central-event-processor** coordinates the transfer lifecycle — state machine, timeouts, retries — and adapters translate events back into FSPIOP callbacks (PUTs) to the DFSPs.

Kafka gives the platform **durability** (events survive service restarts), **ordering** (per-partition), **replayability** (reconciliation can replay a day's events), and **horizontal scalability** (consumers scale out on partitions).

---

## 7. Core Services Deep Dive

### 7.1 central-ledger — The Heart

The **central-ledger** is the core ledger engine of the switch. It records every transfer and maintains the hub's books:

- **Transfers** — the full transfer records, with states (RECEIVED_PREPARE → RESERVED → COMMITTED / ABORTED / REJECTED / EXPIRED).
- **Positions** — per-DFSP, per-currency net positions (what each DFSP owes or is owed by the scheme).
- **Balances and journal entries** — double-entry accounting on hub accounts (the hub's own books; it does *not* hold customer accounts).
- **Participant and account management** — DFSP master data, hub accounts, DFSP limits (net debit caps).
- **Liquidity checks** — validates that a DFSP has sufficient position before allowing a transfer to be reserved.

The central-ledger is where "the money actually moves" in accounting terms: when a transfer is prepared, the payer DFSP's position is debited (reserved); when fulfilled, the payee DFSP's position is credited.

### 7.2 account-lookup-service (ALS) — The Phonebook

The **account-lookup-service** resolves **Party identifiers to DFSPs** — it is the directory/alias service:

- Input: a Party identifier — `MSISDN` (phone number), `EMAIL`, `NATIONAL_ID` (national ID), `PASSPORT`, `THIRD_PARTY_LINK`, or custom party subtypes.
- Output: the Party's information and the DFSP that serves it (used to set `FSPIOP-Destination`).
- Implementation: either an **internal participant lookup** (the hub maintains the directory) or **orchestrated lookups** — the switch fans out the party lookup to DFSPs to find who serves the identifier.
- Exposes the **Participant API** (the hub operator's directory management interface) alongside the FSPIOP party endpoints.

The ALS is what makes "send to a phone number" possible without knowing the payee's bank.

### 7.3 ml-api-adapter — The DFSP-Facing Gateway

The **ml-api-adapter** is the switch's edge component:

- Exposes the **FSPIOP API (inbound)** to DFSPs — the public endpoint they POST to.
- Translates FSPIOP requests into internal Kafka events.
- Manages **callbacks (outbound)** — delivers the asynchronous PUT responses back to DFSP callback URLs.
- Handles the HTTP mechanics: 202 responses, error mapping, FSPIOP header validation, JWS verification.

There are separate adapter deployments for the standard API and the **bulk API** (bulk-api-adapter).

### 7.4 quoting-service — The Quoter

The **quoting-service** computes **quotes** for transfers:

- Determines the **transfer amount**: principal + fees + FX conversion.
- Supports **multiple quote providers** — the hub can aggregate quotes from different DFSPs/quote providers and select the best (fee/FX) for the payer.
- Assigns **quote expiration** (quotes are only valid for a short window, e.g., 60 seconds) — this protects against stale pricing.
- The quote is agreed before the transfer is prepared; the transfer must reference the quote.

### 7.5 transaction-requests-service — Request to Pay

The **transaction-requests-service** manages the **request-to-pay** flow (the P2P "can you pay me?" pattern):

1. Payee DFSP POSTs a `/transactionRequest` to the switch.
2. The switch routes it to the payer DFSP.
3. The payer DFSP presents the request to the customer for authorization (app push, USSD, OTP).
4. On authorization, the flow continues as a normal quote + transfer.

This service powers billers, merchants, and social payments ("send me money for dinner").

### 7.6 central-event-processor — The Orchestrator

The **central-event-processor (CEP)** is the event-orchestration and state-machine engine added to the hub architecture to make the transfer lifecycle robust:

- Consumes transfer events from Kafka and drives them through the **transfer state machine**.
- Coordinates multi-party flows (payer DFSP ↔ switch ↔ payee DFSP) so that no step is lost.
- Implements **retries and timeouts** — e.g., a prepare that is not fulfilled within the window triggers expiry handling; a failed callback is retried with backoff.
- Removes the "fan-out" logic from individual services so that each service stays simple and the flow logic is centralized and testable.

### 7.7 central-router — The Routing Engine

The **central-router** is the routing layer of the hub:

- **Routes transfers between DFSPs** — given the resolved destination (from the ALS party lookup), it determines the correct path and sets `FSPIOP-Destination`.
- **Alias resolution integration** — works with the ALS to look up the DFSP for an account identifier.
- **Settlement model selection** — routes transactions to the appropriate settlement model (DNS, gross, or scheme-specific).
- **Quoter selection** — selects the quoting provider/route for a quote request.
- **Participant status awareness** — does not route to suspended or unavailable DFSPs.

(Note: routing responsibility has historically been split between the adapters and the ALS; the central-router consolidates it as a distinct hub service in the evolving architecture.)

### 7.8 central-settlement — The DNS Engine

The **central-settlement** service implements **deferred net settlement**:

- Manages **settlement windows** (e.g., hourly or daily buckets of transfers).
- Closes a window on schedule and computes **net positions per DFSP** (net of obligations both ways, across all counterparties — multilateral netting).
- Produces **settlement batches** and **settlement accounts** (the hub accounts against which net positions are posted).
- Integrates with the **Settlement API** so the hub operator (or central bank) can trigger settlement, record the outcome, and reconcile.
- Supports the "hub as counterparty" model: DFSP A's net debit is matched by DFSP B's net credit through the hub's books, and the *net* amounts move through settlement accounts (typically at the central bank) — dramatically reducing the liquidity needed versus gross settlement.

### 7.9 Position and Limit Management

Built into central-ledger (and surfaced via the participant/ops APIs):

- **DFSP positions** — net position per DFSP per currency, updated on every prepare/fulfil.
- **Net debit caps** — the maximum negative position (debit) a DFSP may accumulate; the switch **rejects prepares** that would breach the cap.
- **Liquidity monitoring** — DFSPs can query their position; hub operators get alerts on breaches and near-breaches.
- **Position reset / liquidity injection** — when a DFSP funds its settlement account, the hub resets/increases its position, restoring its ability to send.

### 7.10 Bulk Service (bulk-api-adapter)

The **bulk-api-adapter** extends the FSPIOP API with `/bulkQuotes` and `/bulkTransfers`:

- **Batch payments** — a single bulk request carries many individual transfers.
- **Government disbursements** — salaries, subsidies, social cash transfers (G2P), pension payments.
- The adapter splits the bulk into individual transfers internally, processes them through the same central-ledger machinery, and aggregates results back to the requesting DFSP — with per-item status reporting.

### 7.11 Fraud Services

The Mojaloop ecosystem includes **fraud reporting and monitoring** modules (the *Mojaloop Fraud Reporting* module and related risk-management work in the community):

- **Fraud reporting API** — DFSPs report suspicious transactions to the hub.
- **Monitoring and analytics** — the hub operator can analyze traffic patterns, detect anomalies (velocity, unusual values), and share intelligence across participants.
- The fraud module is deliberately **pluggable** — the hub does not force a particular fraud engine; operators integrate their own rules engines and case management (see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) for the bank-side view).

### 7.12 Notifications and Interop Services

- **Notifications** — the switch delivers FSPIOP callbacks/notifications to DFSPs (outbound PUTs), with retry queues for failed deliveries.
- **interop services / interop APIs** — the umbrella of specifications and reference implementations that define how the switch interoperates (the FSPIOP spec itself, plus participant/onboarding APIs).
- **Operations portal** — the hub operator's admin UI (participant onboarding, limits, settlement windows, monitoring).

---

## 8. The Mojaloop SDK (Scheme Adapter)

Most DFSPs do **not** want to build a full FSPIOP implementation themselves. The **Mojaloop SDK** solves this: it provides the DFSP-side connectivity layer so the DFSP's core systems can join a Mojaloop scheme with minimal work.

### 8.1 SDK Components

| Component | Role |
|---|---|
| **SDK Scheme Adapter** (`sdk-scheme-adapter`) | The main integration component. Bridges the DFSP's core systems to the Mojaloop hub over the FSPIOP API |
| **SDK Standard Components** (`sdk-standard-components`) | Reusable libraries: FSPIOP request/response handling, JWS signing/verification, mTLS setup, Mojaloop requests client, error handling |
| **Inbound adapter (inbound server)** | **Receives requests from the switch** (party lookups, quotes, transfers, fulfilments coming *to* the DFSP) and translates them into calls the DFSP core understands |
| **Outbound adapter (outbound client)** | **Initiates requests to the switch** (party lookups, quotes, transfers, reversals) on behalf of the DFSP core |
| **Bulk adapter** | The bulk variant of the scheme adapter for `/bulkTransfers` |

### 8.2 How the SDK Sits in a DFSP

```
   Mojaloop Hub (switch)
          │  FSPIOP API (mTLS + JWS)
┌─────────▼──────────────────────┐
│   SDK Scheme Adapter (DFSP)    │
│  ┌──────────────┐ ┌──────────┐ │
│  │ inbound (srv)│ │outbound  │ │
│  └──────┬───────┘ └────┬─────┘ │
└─────────┼───────────────┼───────┘
          │  DFSP-specific API / events
┌─────────▼───────────────▼───────┐
│  DFSP core: core banking        │
│  (Fineract), mobile money       │
│  switch, Payment Hub EE, etc.   │
└─────────────────────────────────┘
```

The SDK translates between the **FSPIOP API** (REST/JSON with FSPIOP headers, async callbacks) and whatever the DFSP core expects (its own REST API, message queue, or database). The DFSP configures:

- Its **callback URLs** and **party identifier types** (MSISDN, national ID, …) it can serve;
- **Endpoints of the hub** (the switch's FSPIOP base URL);
- **Credentials**: client certs, signing keys, DFSP ID.

### 8.3 Alternative: Direct FSPIOP Integration

A DFSP can also implement the FSPIOP API **directly** (no SDK) — the spec is open, and the Testing Toolkit's conformance suites verify compliance. Direct integration is chosen when:

- The DFSP's platform already speaks REST/JSON natively and the team wants full control;
- Performance/latency requirements demand a lean stack;
- The DFSP is itself a switch or large platform (e.g., another payment system interoperating with Mojaloop).

The trade-off: the SDK ships years of hardening (timeouts, retries, signature handling, error mapping) that a bespoke implementation must rebuild and certify.

---

## 9. The P2P Transfer Flow, Step by Step

This is the canonical end-to-end flow: a customer of **Payer DFSP (A)** sends money to a customer of **Payee DFSP (B)**, identified only by phone number.

```
Customer(A)   Payer DFSP A        SWITCH (hub)          Payee DFSP B    Customer(B)
    │  ①initiate  │                    │                       │
    │────────────▶│  ②POST /parties/   │                       │
    │             │──────MSISDN/…─────▶│  ③party lookup        │
    │             │                    │──────────────────────▶│
    │             │  ⑤PUT party ◀──────│◀── ④party info + dest │
    │             │  ⑥POST /quotes     │                       │
    │             │───────────────────▶│  ⑦quote request       │
    │             │                    │──────────────────────▶│
    │             │  ⑨PUT quote ◀──────│◀── ⑧quote response    │
    │ ⑩confirm    │                    │                       │
    │◀────────────│                    │                       │
    │             │  ⑪POST /transfers  │                       │
    │             │───────────────────▶│  ⑫prepare (reserve)   │
    │             │                    │  central-ledger:      │
    │             │                    │  position + liquidity │
    │             │                    │  checks, journal      │
    │             │                    │  ⑬notify transfer     │
    │             │                    │──────────────────────▶│
    │             │                    │  (payee DFSP credits  │
    │             │                    │   the customer)       │
    │             │                    │  ⑭POST …/fulfilment   │
    │             │                    │◀──────────────────────│
    │             │  ⑮commit callback  │                       │
    │ ⑯success    │◀───────────────────│                       │
    │◀────────────│                    │                       │
```

Numbered detail (the classic 13-step description, expanded):

1. **Initiation** — The payer's customer initiates the payment in the payer DFSP's app/USSD channel (enters the payee's phone number and amount).
2. **Party lookup (resolve payee)** — Payer DFSP → switch: `POST /parties/{Type}/{ID}` (e.g., `MSISDN/+2557…`). The switch responds `202 Accepted`.
3. **Route + query** — The switch (via account-lookup-service) resolves the identifier to Payee DFSP B and forwards the party request to B (or serves it from the internal directory).
4. **Party response** — Payee DFSP B responds with the party information (name, account type, …) and its FSPIOP identity; the switch records `FSPIOP-Destination`.
5. **Party callback** — The switch sends the party info back to Payer DFSP A via callback (`PUT /parties/{Type}/{ID}`). A now knows where to send money.
6. **Quote request** — Payer DFSP A → switch: `POST /quotes` (amount, currency, payee party, fee preferences). Switch → 202.
7. **Quote routing** — The switch (quoting-service + central-router) forwards the quote to Payee DFSP B; B computes its fee/FX and returns a quote (transfer amount, fees, expiration).
8. **Quote response** — The switch returns the quote to Payer DFSP A via callback (`PUT /quotes/{quoteId}`).
9. **Customer confirmation** — Payer DFSP A presents the total (principal + fees) to the customer; the customer confirms (PIN/USSD/OTP).
10. **Transfer execution** — Payer DFSP A → switch: `POST /transfers` (transferId, quoteId, amounts, parties). Switch → 202.
11. **Prepare / reserve** — The switch (central-ledger via central-event-processor) validates: participant active, **position check** (net debit cap), **liquidity check** — then **reserves** the funds: debits Payer DFSP A's position, records journal entries, transfer state → RESERVED.
12. **Transfer notification** — The switch notifies Payee DFSP B of the incoming transfer (`PUT /transfers/{transferId}` — prepared).
13. **Fulfil** — Payee DFSP B credits the payee's account, then confirms to the switch: `POST /transfers/{transferId}/fulfilment`. The switch commits the transfer (state → COMMITTED): final journal entries, position finalization (B's position credited).
14. **Completion callbacks** — The switch sends the fulfilment/commit result to Payer DFSP A (`PUT /transfers/{transferId}/fulfilment` or status callback). A debits the customer's account (if not already) and shows success.
15. **Settlement (offline of the customer journey)** — Transfers accumulate in a settlement window; at window close, central-settlement nets positions per DFSP; net obligations are settled via settlement accounts (RTGS/commercial bank).

### 9.1 What "Instant" Means Here

The customer-visible parts (confirm → both sides notified) complete in **seconds**. The *settlement* between DFSPs is deferred (DNS) — that is exactly how instant payment systems keep per-transaction costs low: liquidity is pooled and netted rather than moved per transaction.

---

## 10. The Transfer Lifecycle and State Machine

Every transfer in Mojaloop passes through a well-defined lifecycle:

```
 quote  →  prepare (reserve)  →  fulfil (commit)  →  settle (DNS)
```

### 10.1 Transfer States (central-ledger)

| State | Meaning |
|---|---|
| `RECEIVED_PREPARE` | The prepare request was received and validated |
| `RESERVED` | Funds reserved: payer DFSP position debited, awaiting fulfilment |
| `COMMITTED` | Fulfilment received: transfer final, payee position credited |
| `ABORTED` | The prepare was rolled back (e.g., payee DFSP rejected, or position insufficient) |
| `REJECTED` | The prepare failed validation and was rejected |
| `EXPIRED` | The transfer timed out before fulfilment |

Transitions (simplified):

```
RECEIVED_PREPARE ──► RESERVED ──► COMMITTED
       │                │
       ▼                ▼
    REJECTED        ABORTED / EXPIRED
```

### 10.2 Position and Journal Entry Semantics

- **Prepare:** position of payer DFSP is debited (money "leaves" the payer's scheme position into a reserved state). Journal entry: `DEBIT payer position / CREDIT hub suspense`.
- **Fulfil:** position of payee DFSP is credited; reserved amount released. Journal entry: `DEBIT suspense / CREDIT payee position`.
- **Abort/expire:** the reserved amount is returned to the payer's position.

The double-entry journal is the hub's immutable audit trail — every transfer can be reconstructed and reconciled (a BCBS 239-style data-quality requirement for the scheme).

### 10.3 Timeouts and the Event Processor

Because the flow spans two DFSPs, the switch must handle parties that go slow or silent:

- **Quote expiry** — a quote is only valid for a short window (e.g., 30–120 s); a transfer referencing an expired quote is rejected.
- **Prepare timeout** — if the payee DFSP does not respond to a prepare within the window, the switch expires the transfer and rolls back the reservation.
- **Fulfilment timeout** — if the payee DFSP never fulfils, the transfer is expired/aborted and the payer's position is restored.
- **Retries** — callbacks are retried with exponential backoff; events are replayed from Kafka for reconciliation.

These behaviors are centralized in **central-event-processor**, which keeps the transfer state machine honest across restarts and partial failures.

---

## 11. Settlement Models: DNS and RTGS Integration

### 11.1 Deferred Net Settlement (DNS) — the Reference Model

1. Transfers accumulate during a **settlement window** (operator-configurable: e.g., hourly, daily; windows can be closed early by the operator).
2. At close, **central-settlement** computes the **net position per DFSP** across all counterparties (multilateral netting): for each DFSP, sum of what it owes minus what it is owed.
3. Net debtors fund their **settlement accounts**; net creditors receive from theirs. The hub posts the net entries (Settlement API: `POST /settlements`, settlement windows, batches).
4. **Settlement execution** happens outside Mojaloop — typically **via the central bank RTGS** or a designated commercial settlement bank — the scheme's settlement accounts are usually held at the central bank.

DNS is what makes low-value instant payments economically viable: a day of millions of micro-transactions nets down to a handful of interbank transfers.

### 11.2 Gross Settlement Option

For deployments that need immediate finality per transaction (e.g., where the central bank requires it, or the hub is directly integrated with an RTGS), Mojaloop supports a **real-time gross settlement (RTGS) integration** mode: each transfer is settled individually through the RTGS as it occurs. This is more liquidity-intensive and is the exception rather than the rule for inclusive retail schemes.

### 11.3 Settlement Account Model

- The hub maintains **hub accounts** (e.g., a multilateral settlement account per currency).
- Each DFSP has a **settlement account** (at the central bank or settlement bank).
- **Settlement batches** reconcile the hub's position ledger against the actual account movements — the operator's reconciliation process (see [payments_hub_guide.md](payments_hub_guide.md) for the DFSP-side reconciliation patterns).

---

## 12. Liquidity, Positions, and Limits

Liquidity management is the operational heart of any switch — a DFSP that cannot fund its obligations threatens the whole scheme.

### 12.1 The Hub's Role

- **Position tracking** — central-ledger tracks each DFSP's net position in real time (every prepare/fulfil moves it).
- **Net debit caps** — the scheme sets a maximum negative position per DFSP (a "net debit cap"). Prepares that would breach the cap are **rejected at the switch** — this is the primary systemic risk control.
- **Liquidity alerts** — the hub warns the operator (and the DFSP) as positions approach limits.
- **Position reset** — when a DFSP funds its settlement account (via RTGS/commercial bank), the hub increases its position, restoring sending capacity. This is the mechanism by which real money flows into the scheme.
- **Hub risk exposure** — in DNS, the hub (and therefore the scheme) bears settlement risk between window close and settlement; caps and window frequency are calibrated to contain that risk.

### 12.2 The DFSP's Side

- Monitor the **position API** (the DFSP queries its own position).
- Fund the settlement account **before** hitting the cap (real-time liquidity management).
- Use **quotes and prepare-rejections** as early signals: a rejected prepare due to cap breach is an operational emergency, not a product bug.

---

## 13. Timeouts, Failures, and Reversals

Real-world payments fail; the design goal is that they fail **safely and accountably**.

| Scenario | Handling |
|---|---|
| Payee DFSP rejects a prepare | Transfer → ABORTED; payer's reservation released; payer DFSP informed via callback |
| Quote expired before transfer | Transfer POST rejected; payer DFSP re-quotes |
| Prepare timeout (no response) | central-event-processor expires the transfer → ABORTED/EXPIRED, reservation released |
| Fulfilment lost / payee silent | Retries, then expiry; reconciliation via event replay |
| Duplicate transferId | Idempotency: the switch returns the existing state rather than double-processing |
| Completed transfer needs undoing (wrong amount, fraud) | **Reversal flow**: `POST /transfers/{id}/reversals` — the payee DFSP processes the reversal, funds move back, both parties notified |
| Reconciliation of failed transfers | The hub exposes transfer states + event history; operators run end-of-day reconciliation between DFSP records, hub records, and settlement batches |

The **invariants** (documented by the community as "Mojaloop Invariants") guarantee: money is never created or destroyed by the switch; a transfer is either fully prepared or not at all; fulfilment only happens after prepare; every state transition is recorded.

---

## 14. Bulk Transfers and Government Disbursements

A flagship use case for financial inclusion is **G2P (government-to-person)** — paying salaries, subsidies, social protection, and emergency aid to millions of citizens.

- **Bulk API** (`/bulkQuotes`, `/bulkTransfers`): one request carries thousands of individual payments; the hub processes each through central-ledger and returns per-item results.
- **Typical flows**: payroll, pension distribution, cash-transfer programs (e.g., COVID relief), agricultural subsidies, energy subsidies.
- **Why Mojaloop is well suited**: any citizen with a bank account OR mobile wallet can receive; the government integrates once (via SDK or direct API) and reaches the entire DFSP landscape; settlement is DNS so the treasury funds one net position rather than N bank accounts.

Bulk transfers share the same FSPIOP security model (mTLS + JWS), so disbursement integrity is verifiable end-to-end.

---

## 15. Fraud and Risk Management

Fraud in instant payment systems is a shared-risk problem: by the time a fraud is detected, the money has already moved and been cashed out.

- **Scheme-level (hub) controls**: participant onboarding checks, limits and caps, transaction monitoring, velocity rules, and the **fraud reporting module** where DFSPs submit and query fraud-related events.
- **DFSP-level controls**: the DFSP remains responsible for customer authentication (PIN, biometrics, device binding), AML/CFT screening, and suspicious transaction reporting (STR) to the FIU — see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md).
- **Design mitigations in Mojaloop**: party lookup before transfer (confirm the payee before sending), quote confirmation (the customer sees total cost), idempotency (no double debits), reversals with audit trail, and full event logging for forensic analysis.
- **Deployment-specific**: national schemes typically layer a **central fraud engine** (rules + AI/ML analytics — see [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md)) in front of or alongside the switch, consuming the same event stream (Kafka) that the hub produces.

---

## 16. Deployment and Operations

### 16.1 Deployment Options

| Environment | Tooling | Purpose |
|---|---|---|
| **Local dev / demo** | Docker Compose (ships with the central-ledger repo and the helm repo's compose files) | Run the switch on a laptop; explore; develop |
| **Test / staging** | Helm charts on Kubernetes (official `mojaloop/helm` umbrella charts: `mojaloop`, `mojaloop-bulk`, `mojaloop-simulator`, `mojaloop-ttk-simulators`) | Integration testing, TTK conformance, performance tests |
| **Production (national)** | Helm + Helmfile/Kustomize, hardened K8s, HA Kafka, managed DBs, PKI, monitoring | The real scheme |

### 16.2 Infrastructure Requirements

| Component | Purpose |
|---|---|
| **Kafka cluster** | The event backbone — must be HA and durable (3+ brokers, replication) |
| **MySQL / PostgreSQL** | Per-service databases (central-ledger, ALS, quoting, settlement, …) — HA with backups |
| **Redis** | Caching (e.g., alias lookup results, rate-limit counters) |
| **Object storage** | Reports, settlement files, archives |
| **PKI / cert management** | cert-manager or external CA for mTLS and JWS certificates, with rotation |
| **Monitoring** | Prometheus + Grafana (metrics/dashboards), ELK (logs), alerting (24/7) |
| **Service mesh (optional)** | Istio for mTLS at mesh level, traffic policy, observability |
| **API gateway / ingress** | Terminates DFSP-facing TLS, rate limiting, WAF |

### 16.3 Performance Characteristics

- Designed for **low-value, high-volume**: the community's performance documentation targets **thousands of TPS** at national scale on commodity hardware.
- **Async event-driven (Kafka)** keeps services decoupled and scalable — each service scales horizontally on Kubernetes (stateless workers + stateful stores).
- The hot path (transfer prepare→fulfil) is designed to be a small number of Kafka hops with minimal synchronous I/O.
- Performance testing is a standard part of deployment projects (using the TTK and load generators); the docs include a performance feature page and production-readiness self-assessment.

### 16.4 Operations Concerns

- **24/7 real-time operations** — the switch never sleeps; on-call rotations, runbooks, and DR drills are mandatory.
- **Reconciliation** — daily reconciliation between hub ledgers, DFSP records, and settlement bank statements.
- **Release management** — the community ships regular releases; production deployments pin versions, test upgrades in staging, and use the TTK regression suites.
- **DR** — multi-AZ/region deployment, Kafka replication, DB backups + restore drills, RPO/RTO defined by the scheme.
- **Production Readiness Matrix** — the Foundation publishes a self-assessment matrix for adopters moving to production.

---

## 17. National Deployments and the Global Ecosystem

### 17.1 The Flagship: Tanzania Instant Payment System (TIPS)

The most prominent production deployment is **TIPS**, operated by the **Bank of Tanzania**:

- Rooted in the **Level One Project pilot** in Tanzania (2018–2019) — the first real-world Mojaloop trial, connecting banks, mobile money operators, and other providers.
- **TIPS** went into testing in **June 2021**, with a closed group of three banks and two telecoms, and rolled out to the full financial ecosystem in **March 2024**.
- By the end of 2024, TIPS had processed **453.7 million interoperable retail transactions** (per the BoT/FSDT case study) — the reference proof that a Mojaloop-based switch can operate at national scale.
- TIPS connects banks, mobile money operators (including M-Pesa), and other licensed payment service providers — exactly the bank ↔ mobile money interoperability Mojaloop was designed for.

### 17.2 Other Reported Deployments and Pilots

| Jurisdiction | System | Status / notes |
|---|---|---|
| **Tanzania** | TIPS | ✅ **Confirmed Mojaloop-based**; production (BoT) |
| **Sierra Leone** | SaPS ("Salone Payment Switch") | Instant payment phase live since Feb 2025 (World Bank/FASTT assistance); **Mojaloop basis reported but verify** |
| **Mozambique** | SPIM (operated by SIMO — Sociedade Interbancária de Moçambique) | National instant payment system overseen by Bank of Mozambique; **technology basis reported — verify** |
| **Mexico / Myanmar / COMESA region** | Various | Level One Project materials cite deployments/pilots for financial inclusion; **reported, verify current status** |
| **Ghana** | GhIPSS Instant Pay (GIP) + Mobile Money Interoperability (MMI) | **Not Mojaloop** — GhIPSS's own switch; useful comparator in the same financial-inclusion narrative |
| **Philippines** | InstaPay (BSP) | **Not Mojaloop** — BancNet/Vocalink-built; comparator for IPS design |

> **Verification note:** The Mojaloop Foundation's site and the Level One Project publish deployment news; attribution of specific national switches to Mojaloop should be confirmed with the scheme operator or the Foundation before citing in procurement documents. TIPS (Tanzania) is the well-documented production case.

### 17.3 Sandboxes, Demos, and Labs

Beyond production schemes, Mojaloop is widely used as a **sandbox/demo platform**:

- The official **Helm deployment** includes a **demo DFSP backend** (the Mojaloop simulator) so you can run a full switch + DFSPs locally and test end-to-end.
- Central banks and regulators use Mojaloop-based **innovation labs** to prototype instant payments, QR standards, and CBDC concepts without committing to a vendor.
- **CBDC work** — Mojaloop's architecture (hub + wallets + settlement integration) has informed central bank digital currency explorations, and its interoperability concepts are frequently cited in CBDC/IPS modernization studies.

---

## 18. DFSP Integration Patterns

### 18.1 Pattern 1: SDK Scheme Adapter (Recommended)

```
DFSP core (Fineract / mobile money switch / Payment Hub EE)
        │
        ▼
SDK Scheme Adapter (inbound + outbound)
        │  FSPIOP API (mTLS + JWS)
        ▼
Mojaloop hub
```

This is the standard onboarding path for banks and fintechs. The SDK handles the protocol; the DFSP team builds thin adapters between the SDK and their core systems.

### 18.2 Pattern 2: Direct FSPIOP Integration

For DFSPs with mature integration teams or platforms that are themselves switches (or that need custom behavior the SDK doesn't expose), implement the FSPIOP API directly and pass TTK conformance.

### 18.3 The Full Open-Source DFSP Stack

The open-source digital finance stack pairs Mojaloop (the switch) with open-source DFSP components:

| Layer | Open-source option | Guide |
|---|---|---|
| Switch (hub) | **Mojaloop** | this guide |
| DFSP payment orchestration | **Mifos Payment Hub EE** | [payments_hub_guide.md](payments_hub_guide.md) |
| DFSP core banking / ledger | **Apache Fineract** | [apache_fineract_guide.md](apache_fineract_guide.md) |
| DFSP↔switch connectivity | **Mojaloop SDK (Scheme Adapter)** | §8 |

A bank deploying for financial inclusion can run: **Fineract** (accounts/loans) → **Payment Hub EE** (payment orchestration, channels, AML hooks) → **SDK Scheme Adapter** → **Mojaloop hub** (national switch). The whole path is Apache-2.0 open source.

---

## 19. Testing Tools: TTK, Simulator, Postman

### 19.1 Mojaloop Testing Toolkit (TTK)

The **Testing Toolkit** (`mojaloop/ml-testing-toolkit`) is the official testing suite:

- **Simulates DFSPs** — the TTK can act as a full DFSP against the switch (or against a DFSP under test).
- **Runs test scenarios** — pre-built test cases cover: party lookup, quotes, transfers, error cases, timeouts, reversals, bulk transfers, and negative paths.
- **Conformance testing** — the TTK's conformance suites verify that a DFSP's FSPIOP implementation (SDK or direct) is **spec-compliant** — the standard gate for DFSP onboarding.
- **Switch regression testing** — the same suites run against hub upgrades to prove no behavioral regression.
- Ships with a **web UI** and configurable test collections; CI-integratable (used in Mojaloop's own CI).

### 19.2 The Mojaloop Simulator

`mojaloop/simulator` is a **mock DFSP** that auto-responds to FSPIOP requests (parties, quotes, transfers, fulfilments) — ideal for standing up a full demo loop quickly: hub + two simulators = working interoperable payments in minutes.

### 19.3 Postman Collections

The community publishes **Postman collections** for the FSPIOP API — handy for manual exploration, smoke tests, and demos against a sandbox or local deployment.

### 19.4 Typical Onboarding Test Plan for a DFSP

1. Stand up the hub locally (Docker Compose) with simulators.
2. Point the DFSP integration (SDK or direct) at the hub.
3. Run TTK conformance suites → fix gaps.
4. Execute end-to-end scenarios (party lookup → quote → transfer → fulfil, plus negative cases), then performance/soak tests and scheme operator acceptance.

---

## 20. Governance, Standards, and Open-Source Process

### 20.1 Governance

- **Mojaloop Foundation** — owns the mission, trademark, specification, and community; funded by sponsor members; guided by a board drawn from members.
- **Scheme governance is separate** — each national deployment has its own scheme rules, operator, and oversight (usually the central bank). Mojaloop provides the technology; the scheme provides the law, rules, and trust.

### 20.2 Standards Alignment

| Standard | Relationship to Mojaloop |
|---|---|
| **FSPIOP API (v1.0, v1.1)** | The core open API spec — versioned OpenAPI definitions, backwards-compatible evolution |
| **ISO 20022** | Mojaloop's message model **aligns with ISO 20022 concepts** (parties, accounts, amounts, FX, charges) and the community documents mappings to ISO 20022 messages (e.g., pacs.008 equivalents) for settlement/reporting integration |
| **ISO 4217** | Currency codes used throughout |
| **ISO 3166** | Country codes (e.g., in party identifiers and scheme configuration) |
| **IBAN (ISO 13616)** | Not primary — Mojaloop keys on **party identifiers** (MSISDN, national ID, email) rather than IBAN, reflecting its financial-inclusion design; IBAN-style account references are supported where schemes require them |
| **Interledger Protocol (ILP)** | Mojaloop **grew from ILP concepts** (early designs used ILP-style atomic transfers and settlement), but evolved its own FSPIOP API and settlement model; ILP heritage remains visible in the design vocabulary |

### 20.3 Open-Source Process

- **Apache 2.0** licensed; public GitHub org (`github.com/mojaloop`).
- Community contribution process: issues → PRs → review (maintainers from ModusBox and the Foundation) → CI (build, unit tests, TTK conformance) → release trains with semantic versioning.
- The documentation site (docs.mojaloop.io) is versioned alongside code, and the community publishes release notes, invariants, and security engineering principles.

---

## 21. Mojaloop vs Commercial Alternatives

### 21.1 Comparison

| Dimension | Mojaloop | Commercial IPS (ACI, FIS, Vocalink, …) | Proprietary national IPS (MEPS+, FedNow, UPI) |
|---|---|---|---|
| **Open source** | ✅ Apache 2.0 | ❌ Licensed software | ❌ Proprietary |
| **Cost** | Software free; pay for hosting/integration | High license + maintenance + integration fees | Funded by the state/scheme; not purchasable |
| **Interoperability model** | Open FSPIOP API; any DFSP can join | Vendor-specific APIs; adapters at cost | Closed/controlled specs |
| **Financial inclusion focus** | ✅ Core mission | Secondary (mainstream retail) | Varies (UPI strongly inclusive; MEPS+/FedNow mainstream) |
| **Deployment model** | You operate it (K8s/Helm); community-supported | Vendor-managed or co-managed | Operated by central bank/designated operator |
| **Governance** | Open community + Foundation; scheme sets rules | Vendor contract + scheme rules | Statutory scheme |
| **Support** | Community + commercial support from ecosystem firms (ModusBox et al.) | Enterprise SLA from vendor | State-backed |
| **Proven at national scale** | TIPS (Tanzania) and pilots; younger track record | Decades of national switches | Decades |

### 21.2 When to Choose Mojaloop

- **Financial inclusion mandate** — serving unbanked/underbanked, low-value high-volume, bank ↔ mobile money ↔ fintech interoperability.
- **Emerging-market national or regional switch** — a central bank/regulator building an IPS with constrained budget.
- **Open-source preference / sovereignty** — the country (or bank) wants to own and inspect its payment infrastructure, avoid vendor lock-in, and control its roadmap.
- **Budget constraints** — no license fees; costs are integration, hosting, and operations.
- **DFSP diversity** — when the ecosystem mixes banks, mobile money operators, and fintechs that all need equal access.

### 21.3 When a Commercial System Is a Better Fit

- **Enterprise support and SLA** — the operator wants a vendor accountable under contract.
- **Proven at scale, regulatory certification** — incumbent banks and some regulators are more comfortable with certified commercial switches (e.g., for a large incumbent-led scheme).
- **Feature depth** — commercial suites ship rich ancillary features (fraud, case management, liquidity tools) out of the box, whereas Mojaloop's are thinner and integration-heavy.
- **Incumbent-bank-dominated schemes** — where participants expect vendor-grade tooling and the inclusion agenda is secondary.

Hybrid models are common: a commercial core with Mojaloop-based interoperability, or Mojaloop with commercial support/consulting from ecosystem firms.

---

## 22. Mojaloop in a Banking Context

### 22.1 How a Bank Uses Mojaloop

| Use case | What the bank does |
|---|---|
| **DFSP in a national scheme** | Connect via SDK Scheme Adapter (or direct FSPIOP) to the country's Mojaloop-based switch; receive/send instant P2P payments |
| **Financial inclusion participant** | Offer low-value accounts/wallets reachable by the whole ecosystem (not just the bank's customers) |
| **Mobile money interoperability** | Bank ↔ M-Pesa (or any wallet) transfers through the switch — the bank becomes reachable from the mobile money world |
| **Government disbursements** | Receive G2P bulk transfers (aid, subsidies, salaries) and credit customers; or originate bulk payments as an aggregator |
| **Agent banking** | Agent cash-in/cash-out flows ride the same interoperable rails; agents of any provider can serve any customer |

### 22.2 Compliance Considerations

- **Scheme-level compliance** (participant rules, scheme fees, oversight reporting) is the **switch operator's** job; the bank still owns its obligations:
  - **AML/CFT**: KYC/CDD, transaction monitoring, STR to the FIU — see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md).
  - **Sanctions screening** on parties (the switch may do scheme-level screening; the bank screens its own customers).
  - **Regulator alignment** (e.g., MAS in Singapore, or the local central bank) — payment services licensing, fast-payment scheme participation rules, and data residency.

### 22.3 Security Considerations

- **mTLS and JWS** — manage DFSP certificates and keys with proper custody and rotation (see [container_certificates_guide.md](container_certificates_guide.md)); the signing key is a critical asset.
- **HSM / vault** for private keys; separation of TLS vs signing keys; audit logging of all signed requests.
- **Callback endpoint security** — the DFSP's FSPIOP callback endpoints must verify the switch's mTLS identity and JWS signatures (never trust unauthenticated callbacks).

### 22.4 Liquidity and Operations

- **Fund the settlement account** in line with expected outbound volume; monitor positions and net debit caps in real time.
- **24/7 operations** — the switch runs around the clock; the bank's integration, reconciliation, and support must too.
- **Reconciliation** — daily matching of hub statements vs core banking postings; exception handling for failed/expired/reversed transfers.
- **DR** — the bank's DFSP connectivity must meet the scheme's availability expectations.

### 22.5 The Central Bank's Role

In a national deployment the central bank typically: **owns/oversees the scheme** (licensing, rules, fees), **runs or mandates the switch operator**, provides **settlement** (settlement accounts, RTGS integration), and sets **oversight** expectations (reporting, risk controls). Mojaloop's Settlement API is designed to plug into exactly this arrangement.

---

## 23. Getting Started

### 23.1 Quickstart: Run a Local Switch

```bash
# 1. Clone the central-ledger repo (ships a docker-compose dev environment)
git clone https://github.com/mojaloop/central-ledger.git
cd central-ledger

# 2. Start the core switch services (Kafka, MySQL, central-ledger,
#    account-lookup, ml-api-adapter, quoting, central-settlement, ...)
docker-compose up -d

# 3. Verify health
curl http://localhost:3001/health && curl http://localhost:4001/health
```

For a fuller, Kubernetes-based environment:

```bash
# Official Helm charts
git clone https://github.com/mojaloop/helm.git
cd helm
helm repo add mojaloop https://mojaloop.github.io/helm
helm install mojaloop mojaloop/mojaloop --namespace mojaloop --create-namespace
```

### 23.2 Simulate a P2P Transfer End-to-End

1. Deploy the hub (compose or Helm) **plus two simulators** (mock DFSPs) and the **Testing Toolkit**.
2. Use the TTK UI or Postman to run the standard scenario: **party lookup → quote → transfer → fulfil**.
3. Watch the flow through the TTK's logs/dashboard: the request hitting ml-api-adapter, the Kafka events, central-ledger state transitions (RESERVED → COMMITTED), and the settlement window.
4. Run the TTK **conformance suites** to see the same tests a DFSP must pass for onboarding.

### 23.3 Explore the API and Docs

- FSPIOP API spec: `docs.mojaloop.io/api/fspiop/v1.1/` (OpenAPI definitions, call-flow routing, headers, error codes).
- Postman collections from the community repo for manual calls.
- The docs' "Deploying Mojaloop" and "Production Readiness" pages for the journey from sandbox to national scheme.

---

## 24. Resources

| Resource | Where |
|---|---|
| **Project site** | [mojaloop.io](https://mojaloop.io) |
| **Documentation** | [docs.mojaloop.io](https://docs.mojaloop.io) (technical, product, API specs, deployment guides, security principles) |
| **GitHub org** | [github.com/mojaloop](https://github.com/mojaloop) (`project` repo for the roadmap; `central-ledger`, `account-lookup-service`, `ml-api-adapter`, `quoting-service`, `central-settlement`, `central-event-processor`, `sdk-scheme-adapter`, `ml-testing-toolkit`, `simulator`, `helm`, `documentation`) |
| **Mojaloop Foundation** | [mojaloop.io/foundation](https://mojaloop.io) — members, mission, news |
| **Level One Project** | [l1p.org](https://www.l1p.org) — principles, research, pilots |
| **Testing Toolkit** | [github.com/mojaloop/ml-testing-toolkit](https://github.com/mojaloop/ml-testing-toolkit) |
| **TIPS (Tanzania) case study** | BoT/FSDT case study (2025); AfricaNenda SIIPS reports |
| **Community** | Mojaloop community Slack (linked from mojaloop.io); YouTube channel (demos, "Inside the Loop" series); annual community events |

---

## 25. Glossary

| Term | Meaning |
|---|---|
| **DFSP** | Digital Financial Service Provider — any licensed participant (bank, mobile money operator, fintech, MFI) |
| **FSPIOP** | Financial Service Provider Interoperability (API) — the Mojaloop interop standard |
| **Hub / switch** | The central platform routing and clearing payments between DFSPs |
| **Party** | A customer/account identified by a Party identifier (MSISDN, national ID, email) |
| **Alias resolution** | Mapping a Party identifier to the DFSP that serves it |
| **Quote** | A priced offer to execute a transfer (amounts, fees, FX, expiration) |
| **Prepare / fulfil** | Transfer phases: 1) prepare/reserve — validate + debit payer position; 2) fulfil/commit — payee DFSP confirms credit, transfer final |
| **Position** | A DFSP's net balance with the scheme (per currency) |
| **Net debit cap** | Maximum negative position a DFSP may hold |
| **Settlement window** | Time bucket of transfers that settle together |
| **DNS / RTGS** | Deferred Net Settlement — netting transfers and settling net positions; Real-Time Gross Settlement — per-transaction finality at the central bank |
| **TTK** | Mojaloop Testing Toolkit — conformance and scenario testing |
| **SDK Scheme Adapter** | DFSP-side connector implementing FSPIOP for the DFSP core |
| **G2P / P2P / P2M** | Government-to-Person, Person-to-Person, Person-to-Merchant payment types |

---

## 26. Conclusion

Mojaloop occupies a unique position in the payments landscape: it is the only **open-source, production-proven reference implementation of a national-scale real-time interoperable payment switch**, purpose-built for financial inclusion. Its contributions are threefold:

1. **A standard (FSPIOP)** — an open, versioned API that any provider can implement, breaking the vendor lock-in that dominates national switches.
2. **A working hub** — a Kubernetes-native microservices platform (central-ledger, account-lookup, quoting, settlement, orchestration) that a central bank or scheme operator can deploy, own, and extend.
3. **A proven model** — from the Level One Project principles through the Tanzania pilot to TIPS processing hundreds of millions of transactions, it demonstrates that inclusive instant payments are achievable at national scale on open infrastructure.

For a bank, Mojaloop is rarely something you run yourself — it is the **scheme you connect to** (via the SDK or direct FSPIOP) or the **platform you help a central bank build**. Understanding its architecture, flows, and operational model is essential for any architect working on financial inclusion, instant payment modernization, or the open-source digital finance stack (Fineract + Payment Hub EE + Mojaloop).

---