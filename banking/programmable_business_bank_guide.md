# The Programmable Business Bank: A Comprehensive Guide

**API-First, Programmable Business Banking — Accounts, Payments, Cards, Treasury, and Compliance as Code**

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Banking Innovation / Business Banking Architecture — Programmable Banking, Virtual Accounts, Programmable Payments, BaaS, Embedded Finance, Treasury APIs, MAS Project Orchid  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is a Programmable Business Bank?](#1-what-is-a-programmable-business-bank)
   - 1.1 [Definition](#11-definition)
   - 1.2 [The Core Concept: Banking Infrastructure as a Platform](#12-the-core-concept-banking-infrastructure-as-a-platform)
   - 1.3 [The Evolution: Portal → API-Enabled → Programmable](#13-the-evolution-portal--api-enabled--programmable)
   - 1.4 [The Drivers](#14-the-drivers)
2. [The Product Stack: Seven Capability Layers](#2-the-product-stack-seven-capability-layers)
   - 2.1 [Layer 1 — Accounts](#21-layer-1--accounts)
   - 2.2 [Layer 2 — Payments](#22-layer-2--payments)
   - 2.3 [Layer 3 — Cards](#23-layer-3--cards)
   - 2.4 [Layer 4 — Treasury / Cash Management](#24-layer-4--treasury--cash-management)
   - 2.5 [Layer 5 — Compliance-as-Code](#25-layer-5--compliance-as-code)
   - 2.6 [Layer 6 — Data / Insights](#26-layer-6--data--insights)
   - 2.7 [Layer 7 — Developer Platform](#27-layer-7--developer-platform)
   - 2.8 [The Product Stack in One View](#28-the-product-stack-in-one-view)
3. [The Virtual Account Architecture](#3-the-virtual-account-architecture)
   - 3.1 [The Master Account + Sub-Ledger Model](#31-the-master-account--sub-ledger-model)
   - 3.2 [VA Use Cases](#32-va-use-cases)
   - 3.3 [VA Ledger Design](#33-va-ledger-design)
   - 3.4 [Reconciliation Matching: Payment → VA](#34-reconciliation-matching-payment--va)
   - 3.5 [The Scale Challenge: Millions of Virtual Accounts](#35-the-scale-challenge-millions-of-virtual-accounts)
4. [Programmable Payments in Depth](#4-programmable-payments-in-depth)
   - 4.1 [The Programmable Payment Stack](#41-the-programmable-payment-stack)
   - 4.2 [Conditional / Smart Payments](#42-conditional--smart-payments)
   - 4.3 [Milestone Payments and Oracles](#43-milestone-payments-and-oracles)
   - 4.4 [Standing Instructions and Automation](#44-standing-instructions-and-automation)
   - 4.5 [MAS Project Orchid: The Programmable Payments Vision](#45-mas-project-orchid-the-programmable-payments-vision)
   - 4.6 [JPMorgan's Programmable Payments](#46-jpmorgans-programmable-payments)
   - 4.7 [The Corporate Treasury Use Case](#47-the-corporate-treasury-use-case)
5. [The Reference Architecture](#5-the-reference-architecture)
   - 5.1 [Architectural Overview](#51-architectural-overview)
   - 5.2 [The Core Ledger](#52-the-core-ledger)
   - 5.3 [The Payments Engine](#53-the-payments-engine)
   - 5.4 [The Card Issuing Platform](#54-the-card-issuing-platform)
   - 5.5 [The API Gateway](#55-the-api-gateway)
   - 5.6 [The Rules / Automation Engine](#56-the-rules--automation-engine)
   - 5.7 [The Event Backbone](#57-the-event-backbone)
   - 5.8 [The Data Platform](#58-the-data-platform)
   - 5.9 [Compliance Engines](#59-compliance-engines)
   - 5.10 [The Developer Platform](#510-the-developer-platform)
   - 5.11 [The Five P's of Programmable Banking](#511-the-five-ps-of-programmable-banking)
6. [Build vs Buy vs BaaS](#6-build-vs-buy-vs-baas)
   - 6.1 [The Three Paths (Plus Partnership)](#61-the-three-paths-plus-partnership)
   - 6.2 [Path A: Build a Full Bank](#62-path-a-build-a-full-bank)
   - 6.3 [Path B: Buy a Core, Build the Programmability Layer](#63-path-b-buy-a-core-build-the-programmability-layer)
   - 6.4 [Path C: BaaS](#64-path-c-baas)
   - 6.5 [Path D: Partnership (Bank + Fintech)](#65-path-d-partnership-bank--fintech)
   - 6.6 [The Decision Framework](#66-the-decision-framework)
7. [The Players](#7-the-players)
   - 7.1 [Business Banking Fintechs](#71-business-banking-fintechs)
   - 7.2 [BaaS Banks and Platforms](#72-baas-banks-and-platforms)
   - 7.3 [Traditional Banks' Programmable Offerings](#73-traditional-banks-programmable-offerings)
   - 7.4 [MAS Project Orchid Participants](#74-mas-project-orchid-participants)
   - 7.5 [The Comparison Table](#75-the-comparison-table)
8. [The "Why Now"](#8-the-why-now)
   - 8.1 [The Enablers](#81-the-enablers)
   - 8.2 [The Economics](#82-the-economics)
9. [The Banking Architect's View](#9-the-banking-architects-view)
   - 9.1 [The Core Design Decisions](#91-the-core-design-decisions)
   - 9.2 [Ledger Design for Virtual Accounts](#92-ledger-design-for-virtual-accounts)
   - 9.3 [The Payments Engine: Multi-Rail](#93-the-payments-engine-multi-rail)
   - 9.4 [Event-Driven Architecture](#94-event-driven-architecture)
   - 9.5 [API Design](#95-api-design)
   - 9.6 [Compliance Automation](#96-compliance-automation)
   - 9.7 [Security](#97-security)
   - 9.8 [The Risk / Control Framework](#98-the-risk--control-framework)
   - 9.9 [Integration with the Customer's Stack](#99-integration-with-the-customers-stack)
10. [Regulatory and Compliance Considerations](#10-regulatory-and-compliance-considerations)
    - 10.1 [The License Question](#101-the-license-question)
    - 10.2 [KYC / KYB Automation](#102-kyc--kyb-automation)
    - 10.3 [AML: Automated Monitoring + Human Oversight](#103-aml-automated-monitoring--human-oversight)
    - 10.4 ["Compliance as Code" Governance: Who Audits the Rules?](#104-compliance-as-code-governance-who-audits-the-rules)
    - 10.5 [Data Residency](#105-data-residency)
    - 10.6 [The MAS / Singapore Context](#106-the-mas--singapore-context)
11. [The Future: 2026 and Beyond](#11-the-future-2026-and-beyond)
    - 11.1 [Programmable Payments Mainstream: Purpose-Bound Money](#111-programmable-payments-mainstream-purpose-bound-money)
    - 11.2 [Tokenized Deposits](#112-tokenized-deposits)
    - 11.3 [The "Everything as API" Bank](#113-the-everything-as-api-bank)
    - 11.4 [AI-Driven Banking Workflows](#114-ai-driven-banking-workflows)
    - 11.5 [The Programmable Bank as Default Business Banking Model](#115-the-programmable-bank-as-default-business-banking-model)
    - 11.6 [Regulation Catching Up](#116-regulation-catching-up)
12. [Glossary](#12-glossary)
13. [Related Guides in This Repository](#13-related-guides-in-this-repository)

---

## 1. What Is a Programmable Business Bank?

### 1.1 Definition

A **programmable business bank** is a business or commercial bank — or bank-like platform — whose products are not merely *delivered* through a portal but are *programmable*: every banking primitive (accounts, payments, cards, virtual accounts, FX, treasury, compliance) is exposed as an API that the customer — a business, or the software that runs the business — can embed, automate, and compose into its own workflows.

Where a traditional business bank sells *products* (a current account, a payment, a card), a programmable business bank sells *primitives*: small, well-defined, composable operations that a developer wires together the way they would wire any other API. The bank's product is not the portal; the portal is just one client of the bank's own API surface.

Three properties distinguish a programmable bank from merely "a bank with good online banking":

1. **API-first, not portal-first.** Every capability exists as a documented, versioned, authenticated API before (or instead of) a UI. The portal, if it exists, is a thin client over the same APIs.
2. **Embeddable.** The bank's functions run inside the customer's own software — an ERP, a TMS, an invoicing tool, a marketplace, a payroll system. The customer never visits the bank; the bank visits the customer's stack.
3. **Composable and automatable.** Customers (or their software) combine primitives into workflows: "open a virtual account per invoice, sweep the balance at midnight, and pay the supplier when goods are received." The bank executes the *rules*, not just individual transactions.

This guide focuses on the *business/commercial* variant — the programmable bank as a platform for companies — as distinct from programmable *retail* banking (which shares the architecture but has different products, e.g., personal savings rules). The reference points are the modern business banking fintechs (Mercury, Brex, Ramp, Qonto, Tide, Airwallex, Aspire), the BaaS banks (ClearBank, Solaris, Griffin, Unit, Railsr), and the incumbent banks' programmable treasury and payments initiatives (J.P. Morgan's Onyx/JPM Coin, Goldman Sachs Transaction Banking, DBS Treasury Prism, Standard Chartered's SC Pay, MAS' Project Orchid).

### 1.2 The Core Concept: Banking Infrastructure as a Platform

The mental model that unifies everything in this guide:

> **A programmable business bank treats banking infrastructure as a *platform for the customer's own workflows* — "your bank in your ERP/TMS/software."**

A company does not want a bank account; it wants to *get paid, pay people, control spend, see cash, and stay compliant* — inside the systems it already runs on. The programmable bank's job is to make each of those outcomes an API call (or an automated rule) rather than a human process in a banking portal.

Concretely, this means the bank's API surface mirrors the *customer's operational vocabulary*, not the bank's internal vocabulary:

| Customer operational need | Programmable bank primitive |
|---|---|
| "Get paid by 5,000 customers without manual reconciliation" | Virtual account per customer/invoice + auto-reconciliation via payment reference |
| "Pay 300 suppliers in 12 countries, in their currency, automatically" | Batch + FX + SWIFT/instant payment APIs with payment rules |
| "Control what each employee/department can spend" | Virtual cards with merchant/amount/velocity limits, auto-issued per policy |
| "Know cash position across 20 entities at 9am every day" | Real-time balance/position APIs + cash pooling + forecasting feeds |
| "Only pay the supplier when the goods clear inspection" | Conditional payment (pay-when-condition-met) |
| "Onboard a new subsidiary's account without paperwork" | KYC/KYB-as-code: account + compliance via one API call |

The "bank in the workflow" concept is the commercial expression of this: the banking function is *embedded* in the customer's systems (see the embedded finance discussion in the [core banking guide](core_banking_systems_guide.md)). The programmable bank is the supply side of embedded finance for business customers: the infrastructure that makes embedding possible.

### 1.3 The Evolution: Portal → API-Enabled → Programmable

Business banking has evolved through three generations. The distinction matters because "API-enabled" is *not* the same as "programmable."

**Generation 1 — Traditional business banking (relationship + portal + files).**
A relationship manager is the interface; the web portal (or branch, or phone) is the instrument; files are the integration channel. Payments move via SWIFT MT messages, EBICS file uploads, host-to-host file transfers, and (for large corporates) proprietary connectivity like SWIFT for Corporates or an ERP-bank file gateway. Reconciliation is a back-office function: the bank sends MT940/MT942 statements, the customer's treasury system matches them. Batch is the natural unit of work — end-of-day processing, file-based payment runs. The relationship manager owns the customer; the customer's software is an afterthought.

**Generation 2 — API-enabled business banking (read/write APIs + cash management portals).**
Banks expose REST APIs for balances, transactions, and payment initiation — often alongside, and sometimes bolted on top of, the legacy portal. Open banking (PSD2 in Europe, SGFinDex-adjacent initiatives elsewhere) forced read/write API access to accounts; banks built developer portals and sandboxes. Corporate treasurers can now pull balances programmatically and push payments via API. But the *products* themselves are unchanged: an account is an account, a payment is a payment; the API is a faster pipe to the same products. "API-enabled" = *the old products, new pipes*.

**Generation 3 — Programmable business banking (real-time APIs, virtual accounts, embedded workflows, event-driven, conditional payments).**
The *products* are re-architected to be programmable:

- **Virtual accounts** as a first-class product — thousands of sub-ledgers behind one master account, created by API in milliseconds, each with its own reconciliation logic.
- **Real-time APIs and events** — instant payments (FAST, SEPA Instant, FedNow, UPI) exposed as APIs; webhooks/event streams pushing transaction data to the customer rather than the customer polling.
- **Programmable money movement** — conditional payments ("pay when X"), approval workflows, rules engines, standing instructions that the bank executes automatically.
- **Embedded workflows** — the bank's functions run inside the customer's ERP/TMS/accounting software; the customer's software drives the bank, not the other way around.
- **Compliance as code** — KYC/KYB, screening, and limits automated into the onboarding and payment path so that programmatic onboarding and payment volumes are operationally possible.

The table below summarizes the shift:

| Dimension | G1: Traditional | G2: API-Enabled | G3: Programmable |
|---|---|---|---|
| Interface | RM + portal + files (SWIFT/EBICS) | Portal + REST APIs | APIs + events + embedded UI; portal is optional |
| Unit of work | Batch files, end-of-day | Individual API calls | Rules, workflows, conditions |
| Accounts | Physical accounts | Physical accounts via API | Physical + unlimited virtual accounts |
| Payments | SWIFT/ACH files | API-initiated single/batch | Conditional, scheduled, rule-governed, event-confirmed |
| Data | MT940 statements, daily | API queries, near-real-time | Webhooks/event streams, enriched, real-time |
| Product model | Products | Products with API pipes | Primitives + composition |
| Customer software | Afterthought | Supported | The point |

### 1.4 The Drivers

Why now? The programmable business bank is the convergence of several forces:

- **SME and digital-native demand for automation.** Digital-native businesses (e-commerce, SaaS, marketplaces, fintechs themselves, gig-economy platforms) run on software; a bank that only offers a portal is a manual bottleneck. They demand APIs because their finance function *is* software — reconciliation, payables, and cash management are code, not spreadsheet work.
- **The API economy.** The general software industry normalized "API-first" products (Stripe, Twilio, AWS as the canonical examples). Business buyers now expect their bank to be as programmable as their payments processor. See the [spec-driven development guide](../technology/spec_driven_development_frameworks_guide.md) for the API design discipline this implies.
- **Fintech competition.** Business banking fintechs (Mercury, Brex, Ramp, Qonto, Tide, Revolut Business, Airwallex, Aspire) built programmability from day one and set the experience bar — free international transfers, instant virtual cards, automated reconciliation — that incumbent banks must match.
- **Open banking.** PSD2/PSD3 in Europe, open finance momentum in the UK, and MAS' openness in Singapore forced banks to open read/write APIs and normalized the idea that account data and payment initiation are API resources (see the open banking discussion in the [core banking guide](core_banking_systems_guide.md)).
- **Instant payments rails.** Real-time payment schemes (FAST in Singapore, SEPA Instant in Europe, FedNow/Real-Time Payments in the US, UPI in India) make *money movement itself* an event-driven, low-latency capability — the substrate that programmable payments need. See the [payments hub guide](payments_hub_guide.md) and the [Mojaloop guide](mojaloop_guide.md) for the rails and hub patterns.
- **Embedded finance.** The distribution shift from "customer comes to bank" to "bank comes to customer" (marketplaces, platforms, software ecosystems) requires banks that can be embedded — which is exactly what programmability provides.
- **Treasury digitization.** Corporate treasuries moved from spreadsheets to TMS/ERP systems; the remaining friction is bank connectivity and automation. Programmable treasury APIs (real-time positions, FX execution, pooling) close that gap.
- **Regulatory modernization.** MAS' Project Orchid explicitly advances a "programmable payments" vision for Singapore (purpose-bound money, conditional payments) — a regulator actively building the runway for programmable banking. UK and EU initiatives (open finance, digital pound explorations) point the same direction.

---

## 2. The Product Stack: Seven Capability Layers

The programmable business bank's product is best understood as seven layers. A given vendor may not have all seven (many fintechs start with payments + cards; BaaS banks with accounts + payments; incumbents with API-fied treasury), but the *target architecture* of the model is the full stack.

### 2.1 Layer 1 — Accounts

**API-created accounts.** The foundational primitive: a business can open an account (or an additional account, or a sub-entity account) via an API call, in seconds to minutes, with programmatic KYC/KYB (Layer 5) completing in the background. This replaces the branch/paper/weeks onboarding of traditional business banking.

**Virtual accounts (VA).** The defining primitive of programmable business banking: sub-accounts / ledger lines behind a master account, created instantly and in effectively unlimited number. A VA has its own identifier (a reference or URN embedded in the account number/payment reference), its own balance (a sub-ledger line), and its own reconciliation behavior — but shares the master account's physical banking relationship. Section 3 covers the VA architecture in depth. Use cases:

- **Collections:** each invoice or customer gets a VA; inbound payments to that VA auto-reconcile to the invoice.
- **Disbursements:** a VA per project, per payee, or per cost center; payments out are attributed automatically.
- **Marketplace splits:** a VA per seller; each sale is split and settled into the seller's VA instantly.
- **Treasury:** a VA per entity, per currency, per business line — a full internal structure without opening hundreds of physical accounts.

**Account programmability.** Rules attached to accounts that the bank executes automatically:

- **Auto-sweep:** sweep balances above a threshold to an interest-bearing account or to a central pool (zero-balance accounting).
- **Auto-notify:** emit a webhook/event when balance crosses a threshold, when a payment arrives, when a VA is inactive.
- **Conditional holds:** place a hold on funds (reserve for an escrow, for a dispute, for a milestone) programmatically and release it conditionally.
- **Standing rules:** recurring transfers, auto-top-up, scheduled disbursements.

The account layer is where "the bank is in your software" becomes literal: the account is a *programmable object* with state, events, and rules, not a row in a core system that a human looks up.

### 2.2 Layer 2 — Payments

**Programmable payment initiation.** All the ways a business pays — single, batch, scheduled — are API operations:

- `POST /payments` — single payment (JSON instruction: amount, currency, beneficiary, rail, reference).
- `POST /payments/batch` — hundreds/thousands of payments in one call, each with its own reference.
- Scheduled/recurring payments — a standing instruction the bank executes on a schedule.

**Payment rules.** A rules layer between "payment instruction" and "payment execution" that governs *whether* and *how* a payment goes out:

- **Approval workflows:** multi-step approval (e.g., maker-checker, amount-tiered, dual control) enforced in the payment path — the bank, not a spreadsheet, enforces segregation of duties. Approval can itself be via API (a second system approves) or via the portal.
- **Limits:** per-account, per-user, per-currency, per-counterparty velocity and amount limits — see the [banking limits domain guide](banking_limits_domain_guide.md) for the limits/controls domain model this reuses.
- **Compliance gates:** sanctions screening, AML checks, fraud scoring hooked into the payment path (Layer 5).

**Conditional payments.** The flagship programmable capability: a payment instruction that executes only when a condition is met — escrow-like release, milestone payments, pay-when-invoice-verified. Section 4 covers this in depth (including MAS Project Orchid's framing of it as "purpose-bound money").

**Rails.** The payments engine is multi-rail, exposed uniformly via API:

- **Instant:** FAST (SG), SEPA Instant (EU), FedNow/RTP (US), UPI (IN) — real-time, 24/7.
- **Batch/ACH:** SEPA Credit Transfer, ACH — for cost-sensitive, non-urgent payments.
- **Cross-border:** SWIFT (MT/ISO 20022) plus FX conversion — ideally with transparent pricing and mid-market FX (the Wise/Airwallex model).
- **Cards:** push-to-card / card payouts (disbursements to card rails).

**Payment links and request-to-pay.** The collections side of the payments layer: generate a payment link (a URL that resolves to a hosted payment page) or a request-to-pay (RTP) instruction — a formal request for payment sent to a payer's bank app — both via API. These are the modern replacements for "please transfer to this account and email us the receipt."

The payments layer is the revenue heart of the programmable bank: payment volume, FX spread, and fees are the economics (Section 8.2).

### 2.3 Layer 3 — Cards

**API-issued cards.** Physical and virtual cards issued via API:

- `POST /cards` — issue a card to an account, with a name, a limit, and a spend policy, in milliseconds.
- Virtual cards are the star: no plastic, instant issuance, designed to be created per use case and destroyed when the use case ends.

**Programmable spend controls.** Every card carries a spend policy enforced in real time at authorization:

- **Merchant controls:** allow/block by MCC (merchant category code) or specific merchant — e.g., a card that only works at AWS, or at approved travel vendors.
- **Amount controls:** per-transaction limit, per-day/week/month limit.
- **Velocity controls:** number of transactions per period, velocity windows (see [banking_limits_domain_guide.md](banking_limits_domain_guide.md) for the limits model).
- **Time controls:** valid only during business hours, or for 30 days, or for a single billing cycle.
- **Auto-destruction:** the card is closed automatically when its purpose completes (subscription paid, project closed, vendor switched) — preventing the classic "zombie card" risk.

**Per-use-case issuance patterns:**

| Use case | Pattern |
|---|---|
| Per-vendor | One card per vendor, merchant-locked to that vendor, limit = contract value |
| Per-subscription | One card per SaaS subscription, amount-locked, auto-destroyed on cancellation |
| Per-project | One card (or card pool) per project with project budget limits |
| Per-employee | Card per employee with expense-policy limits and real-time feeds |
| Per-request | On-demand virtual card for a single purchase, destroyed after settlement |

**Expense policy enforcement.** The card layer doubles as the spend-policy engine: expense policies (who can spend what, where, how much) are enforced *at the point of sale* (authorization) rather than at reimbursement time. This collapses the traditional expense-reporting loop: no out-of-policy spend happens, so there is nothing to reject later.

**Card + payment reconciliation.** Every card transaction generates a real-time feed (webhook/event, Layer 6) with rich metadata: merchant, MCC, amount, currency, FX rate, card, VA, and any custom tags supplied at issuance. The feed flows straight into accounting/ERP, closing the reconciliation loop automatically — the card's transaction *is* the accounting entry's source.

### 2.4 Layer 4 — Treasury / Cash Management

The treasury layer turns the bank into the business's *cash operating system*:

- **Real-time balances and positions.** API-accessible, event-pushed balances across all accounts, entities, currencies, and VAs — the raw material for cash visibility. Aggregation APIs report group-level positions (netting across entities/currencies).
- **Cash pooling.** Notional pooling (interest calculated on a notional net balance across accounts) and zero-balance/zero-target balancing (physical sweeps of sub-account balances into a master account) — exposed as API-managed structures and rules. See the [Nasdaq Calypso guide](nasdaq_calypso_guide.md) for the traditional treasury-management side of this domain (position keeping, cash forecasting, deal capture) that the programmable bank automates rather than replaces.
- **Sweep accounts.** Automated sweeps between operating accounts and investment/sweep vehicles, rule-driven (thresholds, schedules, target balances).
- **In-house banking.** The bank provides the infrastructure for a corporate to run its own internal bank: internal accounts (often VAs) per entity, internal payment netting, internal FX — the corporate treasury's internal bank *as a programmable product*.
- **Programmable FX.** Quote and execute FX via API: `GET /fx/quote`, `POST /fx/trades` — with mid-market pricing, limit orders, and automated hedging rules. Programmatic FX is the piece that lets a business automate multi-currency payables without a human in the loop.
- **Interest / APY products.** Programmatic savings: interest-bearing accounts, APY tiers, auto-parking of idle balances (the "sweep to yield" model popularized by Mercury/Brex).
- **Cash forecasting APIs.** Balance history, payment commitments, and receivable data exposed as feeds that the customer's forecasting models (or the bank's own forecasting service) consume — the data layer (Layer 6) feeding the treasury function.

The treasury layer is what makes the programmable bank credible for *mid-market and corporate* customers, not just startups: it is the API-fied version of the services a corporate treasury department gets from a transaction bank today.

### 2.5 Layer 5 — Compliance-as-Code

The layer that makes programmatic banking *legal*. If a bank opens accounts and moves money by API, compliance must also run by API — there is no human to re-type data into a compliance system:

- **KYC/KYB via API.** Onboarding is automated: the business's identity (ACRA/UEN in Singapore, Companies House in the UK, etc.), directors' identity and background, ownership structure, and documentation are verified through API-integrated providers (identity verification, registry lookups, sanctions databases) with a risk-based decision returned programmatically. A new subsidiary or a new entity can be onboarded in minutes, not weeks — the KYB (Know Your Business) counterpart of consumer KYC.
- **AML transaction monitoring hooks.** Every transaction (or every event on the event backbone, Section 5.7) flows through AML monitoring — rules, thresholds, typologies, anomaly detection — with alerts surfaced to a case-management system. Monitoring is *in the payment path* for scoring and *asynchronous* for investigation.
- **Sanctions screening in the payment path.** Beneficiary, counterparty, and (for cross-border) intermediary screening against sanctions lists (OFAC, EU, UN, MAS) executed synchronously (or near-synchronously) as part of payment validation — the payment API returns a compliance decision, not just a routing decision.
- **Programmatic compliance (rules, thresholds).** Compliance policies as configurable rules: per-entity risk ratings, per-product limits, transaction thresholds that trigger enhanced due diligence (EDD), reporting triggers. The [financial risk and compliance systems guide](financial_risk_compliance_systems_guide.md) covers this domain's systems architecture.
- **Reporting APIs.** Statements (ISO 20022 camt.053-style), tax documents, and regulatory reporting delivered via API and events — the compliance layer's outputs are also programmable, so the customer's own compliance stack can consume them.

The design tension running through this layer: **automation speed vs. control assurance.** Programmatic onboarding and programmatic monitoring are the only way to serve millions of small businesses at API speed, but regulators and auditors require evidence, escalation, and human accountability. Section 10 addresses the governance of "compliance as code."

### 2.6 Layer 6 — Data / Insights

The data layer is what makes all the other layers *useful inside the customer's software*:

- **Real-time transaction feeds.** Webhooks and event streams (see the [event stream processing guide](../technology/event_stream_processing_guide.md)) push every account event — payment received, payment sent, card authorized, balance changed, VA created — to the customer's endpoint. The customer subscribes; the bank emits. This replaces polling and file downloads.
- **Enriched transaction data.** Transactions carry structured metadata: merchant name and category, counterparty, invoice/URN references, FX details, custom tags. Enrichment (the Stripe/Helcim-style categorization of raw bank data) is applied in the bank so the customer receives usable data, not raw statements.
- **Accounting integrations.** One-click sync to QuickBooks, Xero, Netsuite, and other ledgers — transactions, invoices, and payments flow into the books automatically. For SMEs this *is* the reconciliation story: the bank is the bookkeeper's data source.
- **ERP/TMS integrations.** Deeper integrations for mid-market/corporate: SAP, Oracle, NetSuite ERP connectivity; TMS integrations (Kyriba, FIS Integrity, GTreasury) using the bank's APIs or file-based adapters (EBICS, SWIFT for Corporates) — the programmable bank supports both API-native and legacy-connectivity customers.
- **BI export.** Data warehouses/BI tools (Snowflake, BigQuery, Looker, Power BI) can consume account/payment/card datasets directly or via connectors — "give me all my spend data in my warehouse" as a standard feature.

The data layer is where the programmable bank competes on *integrations*, not just APIs: the winner is the bank whose data lands in the customer's tools with the least plumbing.

### 2.7 Layer 7 — Developer Platform

The developer platform is the programmable bank's *front door* — and its product discipline:

- **API documentation.** OpenAPI/Swagger-described, versioned, with examples and guides. API-first means the docs are a product surface, maintained with the rigor of a UI. See the [spec-driven development guide](../technology/spec_driven_development_frameworks_guide.md) for the spec-first workflow that keeps docs and implementation in sync.
- **SDKs.** Client libraries (TypeScript/Python/Go/Java, plus no-code/Excel) so a finance developer integrates in hours, not weeks.
- **Sandbox.** A fully functional test environment with fake money and deterministic scenarios, so integration is safe and instant. Sandbox-to-production parity is a trust feature.
- **Webhooks.** Event subscription management: endpoints, retries, signatures, replay. (The event backbone, Section 5.7, serves these.)
- **Rate limits and quotas.** Documented, fair-use limits; predictable throttling with clear error semantics.
- **API keys and security.** mTLS (mutual TLS) and OAuth2 client credentials for machine-to-machine; scoped keys (read-only, payments-only, per-account); webhook signature verification; audit logs of API usage.
- **Developer experience (DX) as a product.** The insight of the programmable bank: *the developer is the customer*. DX investment — docs quality, error messages, SDK ergonomics, changelogs, deprecation policy — is what makes a bank programmable in practice rather than in marketing.

### 2.8 The Product Stack in One View

| Layer | Primitives | Programmable features | Customer outcome |
|---|---|---|---|
| 1. Accounts | Physical + virtual accounts | API creation, VAs, rules (sweep/notify/hold) | Instant onboarding, auto-reconciliation, structure |
| 2. Payments | Single/batch/scheduled, conditional | Rules engine, approvals, limits, multi-rail, links, RTP | Automated payables/collections, controlled |
| 3. Cards | Physical + virtual cards | Spend controls, per-use-case issuance, auto-destroy | Policy enforced at POS, zero zombie spend |
| 4. Treasury | Balances, pooling, FX, interest | Sweeps, notional/zero-balance pooling, FX APIs, forecasting | Real-time cash OS, automated FX |
| 5. Compliance | KYC/KYB, AML, sanctions | Automated onboarding, in-path screening, rule-based monitoring | Compliance at API speed |
| 6. Data | Feeds, enrichment, integrations | Webhooks/events, enriched txns, accounting/ERP/BI sync | Books that reconcile themselves |
| 7. Developer platform | Docs, SDKs, sandbox | Versioned APIs, webhooks, keys, rate limits | Integration in hours |

---

## 3. The Virtual Account Architecture

Virtual accounts are the keystone of the programmable business bank — the primitive that makes programmatic collections, disbursements, and reconciliation possible at scale. This section goes deep on the model, the design, and the scale problem.

### 3.1 The Master Account + Sub-Ledger Model

A virtual account (VA) is a *logical* account — a sub-ledger line — that exists behind a *physical* master account held at the bank (and ultimately at a clearing/settlement account). The model:

```
                        ┌──────────────────────────────┐
                        │   MASTER ACCOUNT (physical)  │
                        │   e.g., GBP account at bank  │
                        │   Bank sort code + account # │
                        └──────────────┬───────────────┘
                                       │ sub-ledger (VA ledger)
        ┌───────────────┬──────────────┼──────────────┬────────────────┐
        ▼               ▼              ▼              ▼                ▼
   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐      ┌─────────┐
   │ VA-0001 │    │ VA-0002 │    │ VA-0003 │    │ VA-0004 │ ...  │ VA-9999 │
   │ Invoice │    │ Customer│    │ Seller  │    │ Project │      │ (millions)│
   │ #1234   │    │ Acme    │    │ #42     │    │ Titan   │      │         │
   └─────────┘    └─────────┘    └─────────┘    └─────────┘      └─────────┘
```

Key properties:

- **One physical account, infinite logical accounts.** The bank holds one (or a few) master accounts per currency; VAs are pure bookkeeping inside the bank. Incoming payments to a VA land in the master account and are *attributed* to the VA; outgoing payments from a VA are debited from the VA and settle from the master.
- **Each VA has an identity that travels with the payment.** The VA's "address" is typically encoded so inbound payments can be routed to it: either a dedicated sort-code + account-number range (each VA gets a unique full account number — "VA bank account numbers"), or a master account number + a payment reference (URN) that the payer must include. The former is better for payer UX (no reference required); the latter is cheaper (finite account-number space).
- **Each VA has its own balance and its own lifecycle.** Created by API, holds a balance (a sub-ledger line), can be paused/closed/archived, and can carry rules (auto-sweep, auto-notify).

The VA is thus a *ledger abstraction*: it gives the customer the *appearance and function* of many accounts — reconciliation, segregation, structure — with the *economics* of one account.

### 3.2 VA Use Cases

**Collections (each invoice/customer gets a VA — auto-reconciliation).**
The canonical use case. A business issues 1,000 invoices a month; each invoice gets a VA (or each customer gets a VA). When the customer pays, the payment arrives at the master account and is attributed to the correct VA — via the VA's account number or the reference — and the invoice is marked paid *automatically*. No one reads a bank statement and matches payments to invoices by hand. This is why "infinite VAs for reconciliation" is the defining programmable-banking pitch: reconciliation stops being a back-office job and becomes a ledger event.

**Disbursements (VA per project/payee).**
A business paying out — commissions, royalties, partner payouts, supplier payments — creates a VA per payee (or per project), funds it, and pays from it. Every payout is attributed to its VA, so spend attribution and project accounting are automatic. Combined with payment rules (approvals, limits), disbursement is controlled and auditable.

**Marketplace / escrow (VA per seller — split settlements).**
The marketplace pattern: a marketplace holds one master account; every seller gets a VA. Each sale is settled by splitting the proceeds — platform fee to the marketplace's own VA, seller amount to the seller's VA, tax to a tax VA — instantly and automatically. Funds in seller VAs can be held in escrow-style (with conditional release, Section 4.2) until delivery/completion, then released. The VA architecture *is* the settlement engine for platforms; this is the model behind Stripe-style payouts and marketplaces like Amazon/Etsy sellers' accounts.

**Treasury (VA per entity/cost center).**
A corporate runs internal accounts as VAs: one per subsidiary, per cost center, per business line, per currency. Inter-entity transfers become internal VA-to-VA ledger moves (no external payment), enabling in-house banking, notional pooling, and group cash visibility without opening dozens of physical accounts at the bank.

### 3.3 VA Ledger Design

The VA ledger must answer: *given a payment event on the master account, which VA(s) does it hit, and what are the balances afterward?* The design elements:

**Master balance + VA sub-balances (double-entry at two levels).**
Every movement posts at two levels:

1. **Master level:** the physical account's balance moves (money in/out of the bank).
2. **VA level:** the sub-ledger line moves (money attributed to a logical account).

The invariant: **the sum of all VA sub-balances + the unallocated residual = the master account balance.** The "unallocated residual" (money received that matches no VA) is itself a special ledger line — the *suspense/unknown* VA — that drives exception handling (Section 3.4).

**Allocation rules (routing logic).** When money arrives, a deterministic allocation engine decides the VA:

- **Account-number routing:** payment to VA's dedicated account number → that VA. (Requires the bank/clearing arrangement to deliver the beneficiary account number — instant payments and ISO 20022 make this reliable; legacy rails less so.)
- **Reference/URN routing:** payment to master account with a structured reference (invoice number, customer ID, payment URN) → matched to VA by reference lookup.
- **Rule-based routing:** amount/currency/counterparty-based rules can also allocate (e.g., "any GBP payment from supplier X → supplier VA").
- **Split allocation:** a single inbound payment allocated across multiple VAs (marketplace splits) — the engine produces a *distribution* rather than a single attribution.

**VA ledger data model.** See [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) for the ledger modeling patterns; the essentials:

- `virtual_account(id, master_id, currency, owner, status, limits, rules, metadata)` — the VA definition.
- `va_balance(va_id, currency, balance, available, holds)` — balances as first-class, updated by posting.
- `va_movement(va_id, master_movement_id, amount, type, reference, timestamp)` — the sub-ledger entries, linked to master movements.
- `allocation(va_id, payment_id, amount, rule_id)` — audit trail of how payments were attributed (critical for disputes and audits).

**Holds and conditional balances.** VAs support holds (reservations) distinct from available balance — escrow holds, milestone holds, dispute holds. `available = balance − holds`. Conditional payments (Section 4.2) operate on holds: release a hold → payment executes; expire a hold → funds return to available.

### 3.4 Reconciliation Matching: Payment → VA

Reconciliation is the *purpose* of VAs, so the matching engine is the core algorithm:

1. **Capture:** inbound payment arrives at master account (via instant payment, ACH, SWIFT, card settlement) with whatever reference data the rail carried.
2. **Extract:** pull candidate identifiers — beneficiary account number, remittance reference (structured or unstructured), payer identity, amount, currency.
3. **Match:** deterministic rules first (exact reference match, account-number match); then fuzzy/approximate matching (near-amount, partial reference, known-payer lookup) scored and routed to a "confidence" tier.
4. **Allocate:** on match, post to the VA and emit an event (webhook) — the customer's system sees `payment.received` with the VA and reference.
5. **Handle exceptions:** unmatched funds go to the suspense VA; the customer (or an automated rule) resolves them — reallocate, refund, or investigate. A good VA platform keeps the suspense rate under a few percent via strong reference capture and payer education (e.g., generating payment instructions with the reference pre-filled).

The matching engine is where VA platforms earn their keep: it is a hard, data-quality-intensive problem (unstructured SWIFT remittance fields, missing references, partial payments, currency conversions) and the quality of matching *is* the product's value.

### 3.5 The Scale Challenge: Millions of Virtual Accounts

The VA's promise ("infinite virtual accounts") collides with a hard reality: **ledger design at millions-of-VAs scale**. Traditional cores model accounts as rows in a general-ledger-adjacent structure with per-account balances; millions of *logical* accounts stress that model in specific ways:

- **Balance storage:** per-VA balance rows × millions of VAs; balances must be consistent with master-level movements — a *fan-out* problem: one master movement (a bulk settlement) fans out to thousands of VA postings. The ledger must support atomic, high-throughput posting with per-VA aggregation (see the ledger discussion in [core_banking_systems_guide.md](core_banking_systems_guide.md) and the data modeling patterns in [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md)).
- **Read/write amplification:** a VA platform's hot paths are VA creation (bursts of thousands), balance reads (customer dashboards polling), and postings (every payment). Caching, sharding by master/VA key, and event-sourcing (the VA state as a projection of its movement stream) are the standard answers.
- **Reference and metadata indexing:** matching inbound payments to VAs requires fast lookup by account number, reference, and custom keys — search indexes over millions of rows.
- **Lifecycle management:** dormant VAs, closed VAs (keep history, stop receiving), VA numbering schemes that never collide and are payer-friendly.
- **Statement generation:** per-VA statements at scale — generated on demand or via the event stream, not by batch jobs over the whole ledger.

The architectural consequence: **the VA ledger is usually a purpose-built ledger (event-sourced, horizontally sharded), not a bolt-on to the core.** This is why the *core-adjacent* VA platform (Section 5.2) is a distinct architectural component, and why BaaS banks like ClearBank and Griffin built their own ledgers rather than extending legacy cores.

---

## 4. Programmable Payments in Depth

### 4.1 The Programmable Payment Stack

A programmable payment is a pipeline, not a single call:

```
┌──────────────┐   ┌───────────────────────┐   ┌──────────────────┐   ┌──────────────┐   ┌────────────────┐
│ 1. INSTRUCTION│ → │ 2. RULES ENGINE       │ → │ 3. EXECUTION     │ → │ 4. CONFIRM   │ → │ 5. RECONCILE   │
│ JSON: amount, │   │ conditions: amount,   │   │ rails: instant,  │   │ webhook/event│   │ VA/URN match,  │
│ currency,     │   │ counterparty, timing, │   │ ACH, SWIFT, card │   │ settlement   │   │ accounting     │
│ beneficiary,  │   │ approvals, limits,    │   │ FX + routing     │   │ notification │   │ posting        │
│ rail, ref,    │   │ compliance gates      │   │                  │   │              │   │                │
│ condition     │   │                       │   │                  │   │              │   │                │
└──────────────┘   └───────────────────────┘   └──────────────────┘   └──────────────┘   └────────────────┘
```

1. **Payment instruction.** A JSON object fully describing the payment: amount, currency, beneficiary (account, or VA, or card), rail preference, reference(s), schedule (immediate/scheduled/recurring), and — for conditional payments — a *condition* (Section 4.2). The instruction is the API contract; everything downstream is the bank's execution.
2. **Payment rules engine.** Validates the instruction against rules: approvals (maker-checker, amount-tiered), limits (velocity, amount, counterparty — see [banking_limits_domain_guide.md](banking_limits_domain_guide.md)), compliance gates (sanctions, AML scoring, fraud), and business conditions. The engine either *passes* (execute), *blocks* (reject with reason), *defers* (wait for approval), or *holds* (wait for condition).
3. **Payment execution.** The multi-rail execution layer (Section 5.3): route to instant/ACH/SWIFT/card rail, handle FX, submit to the scheme or network, track status (submitted, accepted, settled, returned, rejected). Idempotency is mandatory — the API must tolerate retries without double-paying.
4. **Confirmation.** The result is *pushed*, not polled: webhooks/events for every status transition (submitted → accepted → settled), signed and retried. The customer's system updates its own state on the event.
5. **Reconciliation.** The settlement event flows to the VA/ledger layer: the payment is matched to its VA/URN, the accounting entry is generated, and the customer's books update. Payment *and* reconciliation are one pipeline — this is what "programmable" means operationally: the back office is eliminated, not accelerated.

### 4.2 Conditional / Smart Payments

A **conditional payment** is a payment instruction that executes only when a stated condition is satisfied: "pay supplier X SGD 50,000 *when* the goods are received and inspected," "release escrow to the seller *when* the buyer confirms delivery," "pay the contractor's milestone *when* the milestone is verified."

Mechanics (banking-rail version):

1. The instruction is created with a condition and a `hold` on funds (funds reserved in the payer's account/VA — escrow-like).
2. The payment sits in a *pending* state; the hold freezes the funds (available balance reduced).
3. A condition event arrives — from the customer's API (`POST /payments/{id}/release`), from an automated verification (the customer's system calls back when its own workflow confirms), or from an *oracle* (an external verified data source — e.g., logistics tracking confirming delivery, an inspection system passing a milestone).
4. On condition satisfied: hold released, payment executes on the chosen rail, confirmation events fire.
5. On condition expired/rejected: hold released back to available; the payment never executed.

Conditional payments are the "smart-contract payment" concept without necessarily needing DLT: the bank's rules engine *is* the contract executor. DLT versions (JPM Coin, Section 4.6; MAS Orchid, Section 4.5) do the same thing on tokenized rails. The banking-rail version has the advantage of settling in commercial bank money (no new money form, no stablecoin risk) — which is why MAS frames purpose-bound money as programmable *deposits*, not necessarily crypto.

**The trust question.** A conditional payment is only as trustworthy as the condition verification: who confirms the milestone? The customer's API (trusted, but the bank executes the customer's own rule), a third-party oracle (trusted by both parties — e.g., a logistics platform), or an escrow agent (human-in-the-loop). Programmable banks typically support all three and make the *verification source* part of the payment contract, so both payer and payee know what "condition met" means.

### 4.3 Milestone Payments and Oracles

Milestone payments are conditional payments applied to staged work: construction phases, software delivery sprints, consulting retainers, supply-chain stages. The pattern:

- Contract defines milestones; each milestone maps to a conditional payment instruction (or a VA hold).
- Each instruction's condition references a verification event: an API call from the customer, or an oracle webhook (e.g., "shipment delivered per tracking provider," "inspection passed per quality platform," "milestone approved per PM tool").
- On verification, the milestone payment executes automatically — payables automation for outcome-based contracts.

**Oracles** (in the banking-rail sense) are the verified data sources that feed condition evaluation. The oracle layer is a deliberate architecture choice: conditions should evaluate on *verifiable, auditable* signals, not on unverifiable claims. Banks (and platforms like J.P. Morgan's) integrate oracles for supply-chain data, trade documents, and IoT/asset data; the resulting conditional payments are programmable *trade finance* — payment against verified delivery, not against paper.

### 4.4 Standing Instructions and Automation

The scheduled/automation end of programmable payments:

- **Standing instructions:** recurring payments (rent, payroll, subscriptions, loan repayments) as durable instructions with their own rules and approvals — the bank executes on schedule, with events emitted each run.
- **Trigger-based automation:** "when VA balance > X, sweep to savings"; "when invoice VA receives payment, pay the supplier's corresponding payable"; "when FX rate hits target, execute hedge." The bank's rules engine *watches events* (Section 5.7) and fires actions — the programmable bank as a lightweight workflow engine for money.
- **Approval automation:** rules like "payments < SGD 10k auto-approve; > SGD 10k need CFO approval; > SGD 100k need dual approval" — encoded, enforced, and auditable in the payment path.

### 4.5 MAS Project Orchid: The Programmable Payments Vision

**Project Orchid** is the Monetary Authority of Singapore's initiative to study and pilot *programmable payments* and purpose-bound money — the regulatory articulation of exactly what this guide calls programmable payments. Key facts:

- **The vision: money as a programmable token.** MAS' framing: a digital Singapore dollar (or tokenized deposits more broadly) where *money itself* carries programmability — conditions attached to the money, not just to a payment instruction. This is **purpose-bound money (PBM)**: digital money with conditions on *how* it can be used, enforced programmatically.
- **Programmability features explored:** (1) **conditional payments** — funds that can only be spent when conditions are met (e.g., government payout that can only be spent at approved merchants, or that auto-expires); (2) **routing rules** — money that must flow through specific paths (e.g., subsidy that splits to supplier and tax automatically); (3) **automated execution** — standing logic that executes on events (e.g., payouts triggered by IoT/oracle data).
- **The Orchid whitepaper (November 2023)** set out MAS' assessment: programmable payments and purpose-bound money are *achievable and valuable*; the key design questions are where programmability lives (on the token itself vs. in the system of record), interoperability, and privacy. MAS concluded the *ledger/system-of-record* approach (programmability in the payment system, not embedded in the money token) was the more robust near-term path — an important architectural signal: you do not need tokenized money to do programmable payments.
- **The Orchid playbook (2024)** followed with a practical, technical playbook for industry: design patterns for purpose-bound money (condition models, lifecycle, interfaces), reference architectures, and lessons from the pilots — effectively MAS handing the industry a blueprint for building programmable payments.
- **Pilots and participants:** MAS ran live trials with industry — including **DBS, OCBC, UOB, Standard Chartered, CIMB, and others** — across use cases: government disbursements (programmable vouchers/credits), corporate treasury (conditional supplier payments), trade (payments against delivery), and more. The pilot outcomes informed the playbook and positioned Singapore as the most advanced jurisdiction for programmable payments (Section 10.6 for the SG context).

For a Singapore-based banking architect, Project Orchid is the *regulatory runway*: MAS is actively building the standards, sandboxes, and playbooks that make programmable business banking a licensed, supported activity.

### 4.6 JPMorgan's Programmable Payments

The incumbent-bank vanguard: **J.P. Morgan's Onyx / JPM Coin** platform:

- **JPM Coin** is a tokenized deposit (a digital representation of a JPMorgan deposit, 1:1 with USD — and later EUR) on a permissioned blockchain (Onyx's Liink network). It settles intraday, instantly, between JPMorgan clients.
- **Programmable payments on JPM Coin:** the platform supports conditional payments — payment instructions with programmable conditions (e.g., pay when invoice data is verified, split payments, schedule-driven execution) executed on-chain. J.P. Morgan frames it as "money that can be programmed to do more than one thing."
- **Why it matters:** JPMorgan is validating programmable payments *on tokenized deposit rails* — the "bank money token" path (Section 11.2) — and commercializing it for corporate clients (programmable treasury, automated disbursements, intraday liquidity management). It demonstrates the incumbent-bank strategy: keep the deposit franchise, tokenize it, program it.
- The J.P. Morgan developer portal and API strategy (transaction banking APIs for balances, payments, FX) show the same bank also pursuing the API-first path on traditional rails — the two tracks (traditional-rail APIs and tokenized-rail programmability) run in parallel.

### 4.7 The Corporate Treasury Use Case

The enterprise expression of programmable payments:

- **Programmatic FX hedging:** treasury sets a target rate; the bank's FX APIs monitor the market and execute limit orders automatically; hedges are sized and booked programmatically against forecast exposures (forecast data from Layer 6).
- **Automated disbursements:** payroll, supplier payments, and intercompany funding run as scheduled/batch instructions with rules — approvals for exceptions only. The treasury's payment factory (one entry point, many rails/countries) is the bank's payments layer.
- **Conditional supplier payments:** payments released on delivery confirmation (Section 4.3) — payables automation that reduces both fraud risk and days-payable friction, and (with banks' trade finance) enables supply-chain finance triggers: "invoice approved → payment executed or invoice financed."
- **Intraday liquidity and in-house banking:** VAs per entity, notional pooling, and sweeps keep cash working; conditional holds protect committed outflows. The treasury's cash position is a live API resource, not an end-of-day MT940.

---

## 5. The Reference Architecture

This section assembles the reference architecture of a programmable business bank — the component map a banking architect would draw. Each component is covered in depth by sibling guides; here we show how they compose into the programmable bank.

### 5.1 Architectural Overview

```
                        ┌──────────────────────────────────────────────────────┐
                        │                  DEVELOPER PLATFORM                  │
                        │   API docs · SDKs · Sandbox · API keys · Webhooks   │
                        └───────────────────────┬──────────────────────────────┘
                                                │ HTTPS / mTLS / OAuth2
                        ┌───────────────────────▼──────────────────────────────┐
                        │                     API GATEWAY                      │
                        │   authN/Z · rate limits · versioning · routing ·    │
                        │   request validation · idempotency · audit           │
                        └───────────────────────┬──────────────────────────────┘
        ┌───────────────┬───────────────┬───────┴───────┬───────────────┬───────────────┐
        ▼               ▼               ▼               ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ CORE LEDGER  │ │ PAYMENTS     │ │ CARD ISSUING │ │ RULES/AUTO   │ │ COMPLIANCE   │ │ DATA /       │
│ accounts,    │ │ ENGINE       │ │ PLATFORM     │ │ ENGINE       │ │ ENGINES      │ │ INSIGHTS     │
│ VAs, holds,  │ │ multi-rail    │ │ processor,   │ │ workflow     │ │ KYC/KYB,     │ │ warehouse,   │
│ interest,    │ │ instant/ACH/  │ │ auth, settle,│ │ orchestration│ │ AML, sanctions│ │ lakehouse,   │
│ pooling      │ │ SWIFT/card,  │ │ spend rules  │ │ conditions,  │ │ limits       │ │ BI           │
│              │ │ FX           │ │              │ │ approvals    │ │              │ │              │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │                │                │                │
       └────────────────┴────────────────┴────────────────┴────────────────┴────────────────┘
                        ┌──────────────────────────────▼──────────────────────────────┐
                        │                    EVENT BACKBONE (Kafka)                    │
                        │  account.created · payment.settled · card.authorized ·      │
                        │  va.allocated · compliance.alert · fx.quote.executed        │
                        └──────────────────────────────────────────────────────────────┘
                                        │ (feeds webhooks, data platform, AML, rules)
```

### 5.2 The Core Ledger

The system of record for accounts, VAs, holds, interest, and pooling. See [core_banking_systems_guide.md](core_banking_systems_guide.md) for the core landscape; the programmable bank's ledger requirements are specific:

- **VA sub-ledgering at scale** (Section 3.3): event-sourced, sharded ledger with master+VA double-entry; the VA platform is often *core-adjacent* (a purpose-built ledger beside a traditional core) rather than inside the core, because legacy cores cannot hold millions of logical accounts.
- **Holds as first-class**: escrow/milestone/conditional holds with expiry — not an afterthought of "freeze account."
- **Real-time posting**: instant payments mean posting is synchronous with the API response; no end-of-day batch.
- **Multi-entity, multi-currency**: group structures (holding + subsidiaries), pooling, and inter-entity ledgers.

Modern cores (Thought Machine Vault, Mambu) support product-parameter-driven account behavior and cloud deployment; older cores are typically wrapped rather than replaced (see the modernization discussion in the core guide).

### 5.3 The Payments Engine

The multi-rail execution layer — see [payments_hub_guide.md](payments_hub_guide.md) for the hub architecture and [mojaloop_guide.md](mojaloop_guide.md) for the real-time interop pattern. Requirements:

- **Rail abstraction:** one internal payment model; adapters for FAST, SEPA/SCT/Instant, CHAPS/Target2, FedNow/RTP, ACH, SWIFT (ISO 20022/MT), card rails (for payouts), and tokenized rails (JPM Coin-type) where licensed.
- **Routing:** rail selection by cost, speed, currency, and capability (e.g., instant for urgent, ACH for cheap).
- **Status lifecycle:** submitted → accepted → settled → returned → rejected, with events at every transition; idempotency keys; reconciliation of scheme reports.
- **FX:** pricing (mid + spread), execution (spot/forward/limit), and settlement of multi-currency instructions inside the engine (or adjacent FX service).

### 5.4 The Card Issuing Platform

The card layer (Section 2.3) requires a card issuing processor (e.g., a Visa/Mastercard issuing processor, or card-program platforms) integrated with the ledger and the rules engine:

- **Issuance:** physical + virtual card creation (BIN sponsorship through the processor), card art/fulfillment, virtual card instant issuance.
- **Authorization path:** the spend-control rules must be enforceable *in the authorization flow* (real-time, sub-second) — merchant/MCC/amount/velocity checks against the card policy before the network approves.
- **Settlement:** card transaction settlement posts to the ledger; chargebacks, refunds, and reversals flow back as events.

The card platform is the most "real-time-critical" component: authorization latency budget is hundreds of milliseconds, and the policy engine must be in that path (or the processor's rules must be programmatically configurable).

### 5.5 The API Gateway

The programmable bank's front door (the analog of the enterprise AI gateway for bank APIs — same concerns, different payloads):

- **Authentication/authorization:** OAuth2 client credentials, mTLS, scoped API keys; per-customer, per-key, per-scope permissions.
- **Rate limiting and quotas:** per-customer fairness, burst control, documented limits; throttling with `429` + retry semantics.
- **Versioning:** API versioning and deprecation policy (the contract with customers' software is sacred — breaking changes are breaking *their* integrations).
- **Request validation, idempotency, audit:** schema validation against OpenAPI; idempotency keys for payments; full audit logging of API usage (compliance needs it).
- **Routing and composition:** gateway routes to ledger/payments/cards/rules/compliance services; webhook delivery management (subscriptions, signatures, retries) is often gateway-adjacent.

### 5.6 The Rules / Automation Engine

The component that makes the bank *programmable* rather than just API-enabled: a workflow/orchestration engine that executes customer-defined rules and conditions. See [agentic_workflows_guide.md](../technology/agentic_workflows_guide.md) and [durable_ai_agent_workflows_guide.md](../technology/durable_ai_agent_workflows_guide.md) for the durable-workflow patterns this reuses:

- **Rules:** declarative rules on accounts/payments/cards (limits, approvals, sweeps, holds) — compiled to the enforcement points (payment path, authorization path, ledger postings).
- **Workflows:** multi-step processes (approval chains, conditional payment lifecycles, onboarding journeys) as durable state machines — the bank executes the customer's money workflows durably: retries, timeouts, compensation, audit trail.
- **Event-driven triggers:** rules subscribe to event-backbone topics and fire actions (Section 4.4).
- **Governance:** rule versioning, testing (sandbox), approval of rule changes by the bank (a rule that moves money is a control, Section 9.8).

### 5.7 The Event Backbone

The nervous system — see [event_stream_processing_guide.md](../technology/event_stream_processing_guide.md). Everything the programmable bank does emits events: `account.created`, `va.allocated`, `payment.submitted`, `payment.settled`, `card.authorized`, `balance.changed`, `compliance.alert`, `fx.trade.executed`. The backbone:

- **Powers webhooks** (customer-facing event delivery with signatures/retries — the customer's software *is* a subscriber).
- **Feeds internal consumers**: AML monitoring (every transaction), rules engine (triggers), data platform (ETL), reconciliation, BI.
- **Provides the audit trail**: the event log is the bank's own record of what happened and when.

Kafka (or a managed equivalent) is the standard backbone; ordering (per-account/VA key), exactly-once-ish delivery semantics, and replay are the design concerns.

### 5.8 The Data Platform

The warehouse/lakehouse for analytics, reporting, and insight products — see [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) for banking data modeling:

- **Ingestion** from the event backbone + ledger snapshots (CDC).
- **Enrichment:** transaction categorization, counterparty resolution, merchant metadata (Layer 6 outputs).
- **Serving:** customer-facing APIs (balance history, spend analytics, forecasting feeds), BI exports, regulatory reporting, and the bank's own risk/portfolio analytics.
- **Separation of concerns:** operational ledger (OLTP) vs. analytics (OLAP) with clean pipelines; the customer's "real-time" needs are served from operational APIs/events, while analytics serve dashboards and models.

### 5.9 Compliance Engines

See [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) for the full systems picture; the programmable bank wires them into the flow:

- **KYC/KYB engine:** onboarding automation (identity, registry, sanctions, adverse media), risk scoring, ongoing monitoring triggers.
- **Screening:** sanctions/PEPS screening in the payment path (and on customers/beneficiaries).
- **AML monitoring:** real-time scoring on the event stream + case management for alerts.
- **Limits engine:** the control limits (velocity/amount) shared with the rules engine — see [banking_limits_domain_guide.md](banking_limits_domain_guide.md).
- **Reporting:** regulatory returns, suspicious transaction reports (STRs), customer statements — API/event-delivered.

### 5.10 The Developer Platform

Section 2.7's product layer, architected: docs portal (OpenAPI), SDK generation, sandbox environment (isolated ledger + fake rails with deterministic scenarios), webhook testing tools, API-key management, and observability for customers (request logs, error rates) — the DX surface that the whole architecture exists to serve.

### 5.11 The Five P's of Programmable Banking

A useful mnemonic for the "programmable" layers — the *five P's* — and where each lives in the architecture:

| # | Programmable layer | What it means | Architecture home |
|---|---|---|---|
| 1 | **Programmable accounts** | Rules on accounts: auto-sweep, auto-notify, conditional holds, VA lifecycle | Core ledger + rules engine |
| 2 | **Programmable payments** | Conditional/automated payments, approvals, standing instructions | Payments engine + rules engine |
| 3 | **Programmable cards** | Spend controls, per-use-case issuance, auto-destruction | Card platform + policy engine |
| 4 | **Programmable compliance** | Automated KYC/KYB, in-path screening, rule-based AML | Compliance engines + event backbone |
| 5 | **Programmable treasury** | Automated cash/FX: pooling, sweeps, FX APIs, forecasting | Ledger + payments engine + data platform |

A bank is "programmable" to the degree all five P's are real (not just P2 payments APIs); the product stack in Section 2 maps one-to-one onto the five P's (layers 1–5).

---

## 6. Build vs Buy vs BaaS

How does an organization actually become a programmable business bank? Four paths, with the [core banking guide's](core_banking_systems_guide.md) build-vs-buy-vs-BaaS discussion as the foundation.

### 6.1 The Three Paths (Plus Partnership)

| Path | License | Core/ledger | Programmability layer | Typical builder |
|---|---|---|---|---|
| A: Build a full bank | Own bank license | Build or buy | Build | Incumbent banks, deep-pocketed startups (Revolut, Monzo's business arm), Griffin |
| B: Buy a core, build programmability | Own license (or partner) | Buy (Mambu, Thought Machine, Temenos) | Build | Digital banks, fintechs with license ambition |
| C: BaaS | Partner bank's license | Partner's (or your own ledger on top) | Build on BaaS APIs | Most business fintechs (Mercury, Brex, Ramp, Airwallex, Aspire, Qonto, Tide) |
| D: Partnership (bank + fintech) | Bank's license | Bank's core | Fintech's platform | Classic "banking-as-a-service" deal structures |

### 6.2 Path A: Build a Full Bank

Own the license, own the core, own everything. Griffin is the exemplar in this space: a UK bank **built from the ground up to be a BaaS bank** — its *whole* architecture is API-first programmability (its pitch: "the bank built for banking-as-a-service," with a modern core and programmability as the founding requirement, not a retrofit).

- **Pros:** total control of the product surface, margins, and roadmap; no partner dependency; can build programmability into the ledger itself (deepest VA scale, best unit economics at volume).
- **Cons:** license cost/time (a UK bank license, an MAS digital bank license — years and significant capital), core build is a decade-scale engineering effort (most "build" is really "buy a modern core + build around it"), regulatory overhead is permanent (capital, reporting, supervision).
- **When:** you are a well-funded player with a durable distribution moat, or you are building the *infrastructure* (a bank for banks) rather than a customer-facing brand.

### 6.3 Path B: Buy a Core, Build the Programmability Layer

Own the license (or the fintech status), buy a modern cloud core (Thought Machine Vault, Mambu, Temenos — see the vendor landscape in [core_banking_systems_guide.md](core_banking_systems_guide.md)), and *build the programmability layer yourself*: the VA sub-ledger, rules engine, payment orchestration, API gateway, developer platform. This is the architecture of Section 5 with the core outsourced.

- **Pros:** programmability is your differentiator and you control it fully; modern cores give product-parameterization and cloud ops; faster than building a core.
- **Cons:** you still need a license (or a BaaS partner for the license — blurring into C); the VA/ledger and rules engine are hard, capital-intensive builds; integration debt between core and the programmability layer (the classic "ledger sync" problem — the VA ledger and the core must stay consistent).
- **When:** you have license ambitions and a product thesis that the core vendors won't deliver; you expect to be acquired or to scale to millions of customers.

### 6.4 Path C: BaaS

Use a partner bank's license and infrastructure via API. The BaaS bank (ClearBank, Solaris, Griffin, Banking Circle; platforms like Unit, Railsr, OpenPayd, Payset) provides accounts (often VAs), payments rails, cards, and compliance services as APIs; the fintech builds the customer-facing product, programmability layer, and distribution on top. See the BaaS discussion in [core_banking_systems_guide.md](core_banking_systems_guide.md).

- **Pros:** fastest to market (no license, no core, no scheme memberships); the BaaS partner carries regulatory and scheme complexity; you focus on product/DX/distribution — which is where business fintechs win.
- **Cons:** thin margins (you pay the partner), dependency on partner's roadmap and stability (BaaS partner failures are a real risk class — see the UK/EU BaaS shakeouts), less control over compliance decisions (the partner owns the license and, ultimately, the risk), product ceilings (whatever the partner's APIs can't do, you can't do).
- **When:** you are a product/distribution-led fintech (the vast majority of business banking fintechs); you want to launch in new geographies fast (BaaS is the standard market-entry play).

### 6.5 Path D: Partnership (Bank + Fintech)

The classic arrangement: an incumbent bank provides license, balance sheet, and scheme access; a fintech provides the programmability layer, product, and distribution; revenue shares and risk allocations are contracted. Many "digital SME banking" offerings are structured this way. It is BaaS with a bespoke contract rather than a platform — more control, more negotiation, slower.

### 6.6 The Decision Framework

The build-vs-buy-vs-BaaS decision reduces to four axes:

| Axis | Question | Trade-off |
|---|---|---|
| **License** | Do we have/want a banking license? | License = capital, time, supervision — but full control and margin. No license = BaaS/partnership. |
| **Speed** | How fast must we launch? | BaaS: months. License + build: years. |
| **Control** | How much of the product surface (and compliance decisions) must we own? | Owning programmability = differentiation; owning everything = cost. |
| **Cost/economics** | What are our unit economics at volume? | BaaS fees compress margins at scale; owning the stack pays off at volume (and at capital cost). |

**Heuristic:** *Distribution-led → BaaS (Path C). Product-led with license ambition → buy core + build programmability (B). Infrastructure-led → build full bank (A). Incumbent with existing license → retrofit programmability on the core (A-variant) or build a greenfield digital entity.* Note that the market is converging: BaaS banks (Griffin, ClearBank) are increasingly *programmable themselves* (VAs, rules, events as their core products), so Path C customers are buying programmability from the partner rather than building it — the BaaS bank *is* the programmable bank underneath.

---

## 7. The Players

The landscape, by player type. (Fast-moving market; treat specifics as a snapshot and verify current status before relying on it.)

### 7.1 Business Banking Fintechs

The product-led disruptors — almost all BaaS-backed (Path C), competing on programmability, UX, and price:

- **Mercury** (US) — the archetype: instant onboarding, virtual accounts (up to hundreds, free), API-driven account/payment automation, no-fee domestic wires/ACH, FDIC-insured via partners. Strong with startups/VCs.
- **Brex** (US) — corporate cards + spend management (limits, controls, receipts), treasury (sweep to yield), then accounts; the "spend platform" model.
- **Ramp** (US) — expense-management-led: cards with policy controls, real-time feeds, accounting sync (NetSuite/QuickBooks/Xero); software that happens to include a bank.
- **Qonto** (FR/EU) — the European SMB bank: accounts, cards, invoicing, expense management, accounting integrations; growing via European licensing.
- **Tide** (UK) — SMB banking: accounts, cards, invoicing, expense tools, integrations; strong UK SMB base.
- **Revolut Business** (UK/EU/global) — the business arm of Revolut: accounts, multi-currency, cards, FX, payroll, APIs; bank licenses in multiple EU markets.
- **Wise Business** (global) — the FX/transfer-led model: multi-currency accounts, cheap cross-border, batch payouts, business debit cards; programmatic FX is the core competency.
- **Airwallex** (AU→global) — cross-border + virtual accounts + cards + FX as an *API-first platform*: embedded finance for platforms (marketplaces pay out via Airwallex APIs), plus a business account product.
- **Aspire** (SG/SEA) — Southeast Asia's business finance platform: accounts, cards with spend controls, expense management, FX; a strong regional reference point for the SG market.
- **Volopay** (SG/IN/SEA) — spend management: virtual/physical cards, expense policies, accounting integrations.

Common thread: **cards + spend controls + accounting sync first; accounts and payments APIs second; VAs (Mercury, Airwallex) and FX (Wise, Airwallex) as differentiators.** Programmability depth varies — from "great app + some APIs" to "API-first platform."

### 7.2 BaaS Banks and Platforms

The infrastructure layer — the banks/platforms that make Path C possible, increasingly programmable themselves:

- **ClearBank** (UK) — the UK's first clearing bank built on modern technology; provides accounts, payments (Faster Payments, CHAPS, Bacs), and **virtual accounts** via API to fintechs and banks; known for real-time, API-driven clearing.
- **Solaris** (Germany/EU) — one of Europe's first BaaS banks: full banking license, modular products (accounts, cards, lending, KYC) via API; powers many European fintechs.
- **Griffin** (UK) — the "BaaS bank built for programmability": API-first, event-driven, designed so that fintechs can build on its ledger with full programmability; the cleanest architectural expression of the programmable bank as infrastructure.
- **Unit** (US) — BaaS platform over partner banks: accounts, cards, payments, check deposits, with a strong API/DX focus; popular with US fintechs and embedded finance.
- **Railsr** (UK/EU) — BaaS: accounts, cards, payments, FX via API (formerly Railsbank); a major European BaaS player.
- **Banking Circle** (EU/global) — payments infrastructure for banks, PSPs, and marketplaces: virtual IBANs, multi-currency accounts, cross-border settlement, FX — the *payments-infrastructure* flavor of BaaS.
- **OpenPayd** (UK/EU) — virtual accounts (virtual IBANs), payments, FX for fintechs and platforms; strong on the VA-for-collections pattern.
- **Payset** (UK/EU) — business accounts + virtual IBANs + payments + card issuing via API; SME/PSP focused.

Common thread: **the BaaS bank's product *is* programmability** — VAs/virtual IBANs, real-time payments, cards, and KYC as API products. The distinction from business fintechs: the BaaS bank serves *other companies' customers*, not end businesses directly.

### 7.3 Traditional Banks' Programmable Offerings

Incumbents are retrofitting (and greenfielding) programmability:

- **J.P. Morgan** — the deepest incumbent program: **Onyx / JPM Coin** (tokenized-deposit programmable payments, Section 4.6), plus a full transaction-banking API developer portal (balances, payments, FX, trade APIs) — "JPMorgan developer" is a genuine programmable-banking surface for corporates.
- **Goldman Sachs Transaction Banking (TxB)** — built from scratch as an API-first transaction bank: accounts, payments, cash management for corporates and fintechs, with a modern core; the "bank built by an investment bank for the API era" play.
- **Standard Chartered** — **SC Ventures' SC Pay** (programmable payments pilot) and digital SME platforms; active in MAS Project Orchid pilots; the Asia/ME corridor incumbent pushing programmability.
- **DBS** — **Treasury Prism** (API-based treasury/cash management for corporates), plus participation in Orchid pilots; Singapore's flagship programmable-treasury incumbent.
- **Citibank** — **CitiConnect** APIs (account services, payments, FX, trade) over the Citi transaction banking platform — one of the broadest legacy-bank API programs.
- **HSBC** — **HSBCnet APIs** (balances, payments, FX, collections) for corporates, plus digital treasury initiatives; the Hong Kong/UK incumbent API play.

Common thread: incumbents' programmability is **two-track** — (1) API-fying the existing transaction bank (CitiConnect, HSBCnet, DBS Treasury Prism, Goldman TxB) and (2) tokenized/programmable experiments (JPM Onyx, SC Pay, Orchid participation). Depth varies enormously; the strongest (JPM, Goldman, DBS) are genuinely building programmable banks on modern cores.

### 7.4 MAS Project Orchid Participants

The Singapore programmable-payments trials: **MAS + DBS, OCBC, UOB, Standard Chartered, CIMB, and others** (plus partners like Amazon, Grab, and government agencies in various pilots), covering government disbursements (programmable credits/vouchers), corporate conditional payments, and trade payments. Outcomes fed the Orchid playbook (2024) — see Sections 4.5 and 10.6.

### 7.5 The Comparison Table

| Player | Type | Model | Programmability depth | Target | Geography | Strengths |
|---|---|---|---|---|---|---|
| Mercury | Business fintech | BaaS (partner banks) | Accounts, VAs, payments APIs, automation | Startups/SME | US | DX, free transfers, VAs |
| Brex | Business fintech | BaaS | Cards, spend controls, treasury sweep | Startups/SME | US | Spend platform, treasury |
| Ramp | Business fintech | BaaS | Cards, policies, accounting sync | SME/mid-market | US | Expense automation, integrations |
| Qonto | Business fintech | Own EU license | Accounts, cards, invoicing, integrations | SME | EU (FR/DE/IT/ES) | Full SMB suite, local licenses |
| Tide | Business fintech | BaaS/partnership | Accounts, cards, expense | SME | UK/IN | UK SMB base, ecosystem |
| Revolut Business | Fintech w/ licenses | Own licenses (EU) | Accounts, cards, FX, payroll, APIs | SME/mid-market | Global (EU core) | Multi-currency, scale |
| Wise Business | Fintech | Own licenses/partners | Multi-currency accounts, batch payouts, FX | SME/mid-market | Global | FX cost leadership, programmatic FX |
| Airwallex | Fintech/platform | Licenses + partners | VAs, cards, FX, cross-border, embedded APIs | Platforms + SME | Global (APAC strong) | Cross-border + embedded finance |
| Aspire | Business fintech | BaaS (SG) | Accounts, cards, spend controls, FX | SME | SEA (SG) | Regional SME finance suite |
| Volopay | Business fintech | BaaS | Virtual cards, expense policies | SME | SEA/IN | Spend management |
| ClearBank | BaaS bank | Own UK bank license | Accounts, VAs, real-time payments APIs | Fintechs/banks | UK | Real-time clearing, VA scale |
| Solaris | BaaS bank | Own EU license | Modular banking products via API | Fintechs | EU | Full license, modularity |
| Griffin | BaaS bank | Own UK license | API-first programmable bank, ledger, events | Fintechs | UK | Programmability-first architecture |
| Unit | BaaS platform | Partner banks (US) | Accounts, cards, payments APIs | Fintechs/embedded | US | DX, embedded finance |
| Railsr | BaaS platform | UK/EU licenses | Accounts, cards, payments, FX APIs | Fintechs | UK/EU | Card + payment breadth |
| Banking Circle | Payments infra | Licenses | Virtual IBANs, multi-currency settlement, FX | PSPs/marketplaces | EU/global | Cross-border liquidity |
| OpenPayd | BaaS/payments | Licenses | Virtual IBANs, payments, FX | Fintechs | UK/EU | VA collections pattern |
| J.P. Morgan | Incumbent | Own license | TxB APIs + Onyx/JPM Coin programmable payments | Corporate | Global | Balance sheet, tokenized deposits |
| Goldman TxB | Incumbent | Own license | API-first transaction banking | Corporate/fintech | US/global | Greenfield modern stack |
| Standard Chartered | Incumbent | Own license | SC Pay programmable payments pilot, APIs | Corporate | Asia/ME/Africa | Orchid participation, corridor strength |
| DBS | Incumbent | Own license | Treasury Prism APIs, Orchid pilots | Corporate | SG/Asia | Regional treasury programmability |
| Citi | Incumbent | Own license | CitiConnect APIs (payments, FX, trade) | Corporate | Global | Breadth of API coverage |
| HSBC | Incumbent | Own license | HSBCnet APIs, digital treasury | Corporate | Global | Global network, liquidity |

---

## 8. The "Why Now"

### 8.1 The Enablers

Programmable business banking is possible *now* because five enablers matured in the last decade:

1. **Instant payments rails.** Real-time schemes make money movement event-driven and low-latency — the substrate for programmable payments. See [mojaloop_guide.md](mojaloop_guide.md) and [payments_hub_guide.md](payments_hub_guide.md).
2. **Cloud + API infrastructure.** Cloud cores, managed event streaming (Kafka), API gateways, and DevOps make the Section 5 architecture buildable by a small team in months — impossible a decade ago.
3. **Open banking.** Regulatory opening of account data and payment initiation normalized API banking and built the talent/expectation base (see [core_banking_systems_guide.md](core_banking_systems_guide.md)).
4. **DLT/tokenization.** Tokenized deposits and programmable money experiments (JPM Coin, Orchid, BIS/central-bank work) prove the *next* money form and create the standards conversation — see [blockchain_technology_guide.md](../technology/blockchain_technology_guide.md).
5. **Regulatory openness.** MAS' Project Orchid (purpose-bound money, sandboxes, playbook), UK/EU open finance, and sandbox regimes signal regulators *want* programmable banking — de-risking the business model.

Plus the demand side: **SME digitization** — businesses run on QuickBooks/Xero/NetSuite/SAP; their finance function expects API connectivity the way it expects email. ERP/TMS connectivity demand is now a *bank-selection criterion* for mid-market companies, which is precisely what the programmable bank sells.

### 8.2 The Economics

Why the model is economically attractive (and why incumbents are worried):

- **Distribution economics.** Programmatic/embedded distribution cuts customer acquisition cost: a fintech's banking product is sold *inside* another product (a marketplace's payout flow, an ERP's payables module) or virally (developers integrate → their company becomes a customer). The revenue per SME relationship via embedded/programmatic distribution compounds without a branch or RM network.
- **Fee + FX + float.** The revenue stack: (1) **fees** — account, card, payment, subscription fees (often tiered; interchange on cards); (2) **FX** — spread on cross-border conversion (the Wise model: thin but high-volume, programmatic); (3) **float/interest** — interest margin on deposits (sweep-to-yield products turn idle balances into revenue) and on pooled balances. The programmable bank earns on *volume and automation*, not on relationship pricing — which is why unit economics favor API distribution at scale.
- **Cross-sell via data.** Real-time, enriched transaction data (Layer 6) enables credit underwriting (revenue-based lending), FX, and treasury cross-sell — the data layer is a revenue asset, not just a feature.
- **The incumbent's cost problem.** Incumbents' SME banking unit economics (branch/RM cost, manual onboarding, manual reconciliation) are structurally worse than the programmable bank's; the fintech/BaaS stack is *the* way SME banking gets profitable — which is why every major market now has a neo-business-bank cohort and why incumbents are API-fying (Section 7.3).

---

## 9. The Banking Architect's View

How a banking architect designs, evaluates, and governs a programmable business bank.

### 9.1 The Core Design Decisions

The architecture in Section 5 rests on a handful of decisions that shape everything else:

1. **Ledger design for virtual accounts** (scale) — Section 9.2.
2. **The payments engine** (multi-rail) — Section 9.3.
3. **Event-driven architecture** (everything emits events) — Section 9.4.
4. **API design** (the API is the product) — Section 9.5.
5. **Compliance automation** (programmatic controls) — Section 9.6.
6. **Security** (API security is bank security) — Section 9.7.
7. **Risk/control framework** (automated controls need human governance) — Section 9.8.

### 9.2 Ledger Design for Virtual Accounts

The single most consequential decision. Design principles:

- **Event-sourcing over CRUD.** Model VA balances as projections of a movement stream: every posting is an immutable event; balances are derived (with materialized views for reads). This gives auditability, replay, and the ability to reconstruct any VA's history — regulators love it, and reconciliation (Section 3.4) becomes a query, not a job.
- **Shard by master account / VA key.** Hot paths (VA creation, posting, balance reads) must not serialize on a single ledger. Shard the VA ledger; keep the *unallocated/suspense* handling per-shard.
- **Two-level double-entry with invariants.** Enforce `Σ VA balances + suspense = master balance` continuously; test the invariant in CI (a ledger invariant is a system property, not a report).
- **Holds as ledger state.** Conditional payments (Section 4.2) need holds: `available = balance − holds`, with hold expiry and release as first-class operations.
- **A purpose-built VA ledger, not a core bolt-on.** Plan for the VA sub-ledger to be its own service beside the core (Section 5.2), with a well-defined sync contract — the classic failure mode is the "two ledgers disagree" integration mess; the fix is a single source of truth per balance (the VA ledger owns VA balances; the core owns the master account; the bridge is a defined, reconciled pipeline).
- **Design for millions of VAs from day one** — numbering schemes, reference indexing, dormant-VA lifecycle, and statement generation (Section 3.5) are *schema decisions*, not later optimizations.

### 9.3 The Payments Engine: Multi-Rail

- **One internal model, many adapters.** Normalize rails into one payment state machine (submitted/executing/settled/returned) with rail-specific adapters; add rails (FedNow, a new scheme, a tokenized rail) as adapters, not rewrites. See [payments_hub_guide.md](payments_hub_guide.md).
- **Idempotency everywhere.** The payment API must be retry-safe: idempotency keys, exactly-once posting per key, idempotent webhook delivery. Double-pay is the cardinal sin of programmable payments.
- **Real-time + batch coexistence.** Instant rails are the showpiece, but ACH/SWIFT economics matter: the engine routes by cost/speed/capability and *communicates the rail to the customer* (different rails have different settlement certainty).
- **Status transparency.** Every transition emits an event; the customer's system and the bank's ops see the same state. Reconciliation of scheme reports (FAST/SEPA/SWIFT returns) is automated into the event stream.

### 9.4 Event-Driven Architecture

- **The event backbone is non-negotiable** (Section 5.7): webhooks for customers, AML/risk consumers, rules engine, data platform, audit. Choose Kafka (or managed streaming) with per-account/VA keyed ordering and replay capability.
- **Events are the contract.** Define the event catalog (topics, schemas, versions) as deliberately as the REST API; customers integrate against `payment.settled`, not against polling. Schema evolution (additive fields, versioning) is a compatibility discipline — see the [event stream processing guide](../technology/event_stream_processing_guide.md).
- **Outbox pattern.** Emit events transactionally with the state change (outbox table → backbone) so events never lie about state.
- **Exactly-once-ish + retries.** Webhook delivery: signed, retried with backoff, replayable from the event log.

### 9.5 API Design

- **The API is the product** — design it like one: versioned, documented (OpenAPI), stable, with deprecation policy; see [spec_driven_development_frameworks_guide.md](../technology/spec_driven_development_frameworks_guide.md) for the spec-first workflow.
- **Resource model mirrors the customer's vocabulary** (Section 1.2): accounts/VAs, payments, cards, FX quotes, webhook subscriptions — not the bank's internal product codes.
- **Granularity and idempotency:** batch endpoints for scale; idempotency keys; clear error semantics (validation vs. rules-blocked vs. compliance-blocked with *reasons* — "blocked by sanctions" is different from "exceeds limit").
- **Versioning and compatibility:** customers' ERP integrations live for years; breaking changes are migration projects for *them*. Semantic versioning + long deprecation windows + sandbox migration testing.
- **Consistency model:** document whether reads are strongly consistent (ledger) vs. eventually consistent (analytics); real-time APIs should be synchronous where money is concerned (posting) and event-based where it isn't.

### 9.6 Compliance Automation

- **Compliance in the path, not in the rearview.** Screening and limits run synchronously in the payment/card path (sub-second for cards, low-latency for payments); AML monitoring consumes the event stream asynchronously. See [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md).
- **Programmatic onboarding with risk-based depth.** Tier onboarding: low-risk businesses onboard in minutes (automated checks); higher-risk profiles trigger EDD workflows (documented, human-escalated). The API returns a decision *and* the evidence trail.
- **Limits are shared infrastructure.** The limits engine (velocity/amount) is one component consumed by payments, cards, and treasury — see [banking_limits_domain_guide.md](banking_limits_domain_guide.md) for the domain model; configure once, enforce everywhere.
- **Everything auditable.** Automated compliance generates the same audit evidence as manual compliance: decision logs, rule versions, alert histories. The compliance *data model* is a first-class design output.

### 9.7 Security

API security is bank security — the API surface *is* the attack surface (see [llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) for the general API-security discipline; the analogies transfer directly):

- **AuthN/Z:** mTLS for machine-to-machine; OAuth2 client credentials with scopes; short-lived tokens; per-customer key rotation. Scopes should be *capability-grained* (read-only vs. payments) and *resource-grained* (per-account).
- **Webhook security:** signed payloads (HMAC), replay protection (timestamps/nonces), TLS everywhere.
- **Payment-path fraud:** velocity checks, beneficiary vetting, anomaly detection on the event stream; conditional payments add a *rule-integrity* concern (who can create/alter payment rules? — treat rule changes as payment authority changes).
- **Data protection:** encryption at rest/in transit, field-level encryption for sensitive data, tokenization where possible; audit logging of API access (who did what, when — including internal access).
- **Supply chain:** BaaS dependencies (partner bank APIs) are third-party risk — contract for security posture, incident response, and data handling.

### 9.8 The Risk / Control Framework

The governance insight of programmable banking: **automated controls still need human governance.** Programmatic compliance and rules-based money movement change *how* controls execute, not the requirement that controls exist, be owned, and be auditable:

- **Rule governance:** rules that move money (approval thresholds, sweep rules, conditional releases) are *controls*: they need owners, versioning, testing in sandbox, change-approval, and audit. "Who approved this rule change?" is the governance question — see the limits governance discussion in [banking_limits_domain_guide.md](banking_limits_domain_guide.md).
- **Segregation of duties in code:** maker-checker and dual control are enforced in the rules engine, but the *rule-definition* function must itself be segregated from *rule-execution* oversight (the classic three-lines-of-defense mapping: business defines rules → risk validates → audit tests).
- **Model/rule risk:** rules are models; they drift, fail, and get gamed. Monitoring: rule effectiveness metrics (false-positive/negative on AML, limit-breach rates), periodic rule review, and kill-switches (the ability to halt a rule class instantly).
- **Human escalation paths:** automated onboarding/monitoring must route to humans for EDD, STR filing, and disputes — the programmability layer must *design the human handoff*, not assume it away.
- **Regulatory reporting:** every automated control generates reportable data (STRs, regulatory returns, audit trails); the reporting APIs (Layer 5) exist because the controls do.

### 9.9 Integration with the Customer's Stack

The programmable bank succeeds or fails on integration:

- **ERP/TMS integrations:** SAP, Oracle, NetSuite, Kyriba, FIS Integrity — via APIs, file adapters (EBICS, SWIFT for Corporates, camt statements), and hosted connectors. The bank meets the customer where its treasury already is; see the traditional treasury side in [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md).
- **Accounting software:** QuickBooks/Xero/NetSuite sync — transaction feeds, invoice matching, reconciliation-ready data (Layer 6). For SMEs this is the *product*, not a feature.
- **The "bank in the workflow" concept:** banking embedded into the customer's systems — payments initiated from the ERP's payables screen, cards issued from the expense tool, VAs created from the invoicing system, reconciliation happening in the books. This is embedded finance for business customers — see the embedded finance discussion in [core_banking_systems_guide.md](core_banking_systems_guide.md). The architect's job: make the bank's API surface *feel* like part of the customer's stack (consistent events, webhooks into their workflow engine, SDKs in their language).
- **Two integration modes, one product:** API-native (fintech/platform customers) and connectivity-native (ERP/TMS/file customers) must be served by the same underlying programmability — the file-based customer is a *translation* of the API product, not a legacy second-class citizen.

---

## 10. Regulatory and Compliance Considerations

### 10.1 The License Question

What legal wrapper does a programmable business bank need? Three options, each with different programmability ceilings:

- **Full banking license** — can take deposits, lend, issue cards, hold customer money as *deposits* (not safeguarded funds), and operate the full product stack. This is what makes a *bank* a bank: the balance sheet and the deposit franchise. Costs: capital, supervision, time (years). The programmable bank with its own license (Revolut Business, Qonto's EU licenses, ClearBank, Griffin, the SG digital banks) has the deepest product and economics.
- **E-money institution (EMI) / payment institution (PI)** — can hold customer *funds* (safeguarded, not lent), execute payments, issue cards (via a bank partner for card programs), and often operate VAs. Faster and cheaper than a license; the wrapper of many BaaS-backed fintechs. Programmability is comparable; the deposit/interest/float economics are *not* (EMI funds are safeguarded, not on-balance-sheet — no lending, no float).
- **BaaS partner** — no license at all; the partner bank holds the license and the deposits; the fintech operates on partner APIs (Path C). Simplest; the partner's compliance framework governs what you can program.

The license decision is the master constraint on everything else (Section 6.6): it determines what products you can offer (interest? lending? cards?), what capital you need, and how much of the compliance burden you own.

### 10.2 KYC / KYB Automation

Programmatic onboarding is the compliance burden of the model — and its scalability:

- **KYB (Know Your Business) as code:** verify the entity (registry lookup — ACRA in SG, Companies House in UK), its ownership structure (ultimate beneficial owners), directors, and documentation; screen against sanctions/adverse media; risk-rate. All API-driven, with evidence retained.
- **Risk-based depth:** the art is tiering — fully automated onboarding for low-risk businesses; enhanced due diligence (EDD) with human review for high-risk profiles (PEP involvement, complex structures, high-risk industries/jurisdictions). The API must be able to return "pending EDD" as a first-class state, not an error.
- **The programmatic burden is real:** API speed invites volume; volume invites regulatory scrutiny. Regulators expect the same control quality at 100,000 programmatic onboardings as at 1,000 manual ones — which is why the *evidence trail* (Section 9.6) and *rule governance* (Section 9.8) are architecture, not paperwork.
- **Ongoing monitoring:** KYB is not onboarding-only — entity changes, ownership changes, and risk-drift must re-trigger checks (events again: registry-change webhooks, periodic re-screening).

### 10.3 AML: Automated Monitoring + Human Oversight

- **Automated transaction monitoring:** the event backbone feeds monitoring engines (rules, typologies, anomaly detection) in real time; alerts route to case management. Programmatic banking generates *transaction-scale* data, so monitoring must be event-stream-native, not batch-file-native.
- **Screening in the path:** sanctions screening on beneficiaries/counterparties at payment initiation and on customers at onboarding — synchronous where the rail allows, with clear API semantics for blocked payments.
- **Human oversight remains:** automated monitoring detects; humans investigate, decide, and file (STRs). The programmable bank must design the human escalation path explicitly (Section 9.8) and demonstrate it to regulators — "the machine does the scanning, the bank does the judging."
- **The AML/UX tension:** instant payments + frictionless onboarding vs. monitoring depth. The resolution is *layered*: real-time screening for the obvious, streaming analytics for the subtle, human judgment for the consequential.

### 10.4 "Compliance as Code" Governance: Who Audits the Rules?

The novel governance question: when compliance *is* code (rules, thresholds, automated decisions), who audits it?

- **Rules are regulated artifacts.** Compliance rules (screening thresholds, monitoring parameters, onboarding criteria, limits) must be version-controlled, reviewed, and approved like any control — with change history and audit trails. The compliance team *owns* the rules; engineering implements them; audit tests them.
- **Model/rule validation:** monitoring thresholds and risk models need validation (back-testing, tuning) and periodic recalibration — the same discipline as credit models.
- **Independent testing:** internal audit must be able to *simulate* programmatic onboarding/payments in a test environment and verify controls fire as designed (the sandbox doubles as the audit lab).
- **Regulator engagement:** regulators are learning the model too; proactive engagement (MAS' sandbox approach is the model to copy) and transparent control documentation de-risk the conversation. The Orchid playbook's existence shows MAS wants *industry* to build this responsibly.

### 10.5 Data Residency

Programmable banking is data-intensive; residency rules bite:

- **Customer data residency:** accounts, transactions, and KYC data for customers in a jurisdiction generally must reside (and be processed) in that jurisdiction — SG data for SG customers, EU data under GDPR in the EU, etc. The architecture must be *regional* (per-jurisdiction data planes) even when the code is global — the same pattern as [on_prem_llm_deployment_guide.md](../technology/on_prem_llm_deployment_guide.md) describes for regulated AI data.
- **Cross-border flows:** payments cross borders by nature; the *data* (references, counterparties, enrichment) crosses with them — data-transfer agreements (EU SCCs, etc.) and transfer-impact assessments apply to the bank's own processing, not just the customer's.
- **Cloud implications:** hyperscaler regions and sovereignty offerings (EU data residency, GCC, etc.) become selection criteria; "cloud region per jurisdiction" is the default architecture for a multi-country programmable bank.
- **Third-party risk:** BaaS partners and processors hold data too; residency commitments must be contractual and verifiable (Section 9.7).

### 10.6 The MAS / Singapore Context

For the SG banking architect, the local picture:

- **Digital bank licenses:** MAS awarded digital full-bank licenses to **GXS** (Grab/Singtel) and **MariBank** (Sea/Shopee), and digital wholesale-bank licenses to **Green Link Digital Bank (GLDB)** (Greenland + Linklogis) and **ANEXT** (Ant Group) in December 2020 — **Trust** (Standard Chartered + FairPrice) is the fifth digital bank but holds a full bank license granted separately (commenced 15 Dec 2021), not a Dec-2020 digital-bank-framework award (see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) §1.4 for the verified correction) — a deliberate regulatory experiment in tech-native banking (see the digital bank discussion in [core_banking_systems_guide.md](core_banking_systems_guide.md)). These institutions are the licensed beachhead of the programmable model in SG (GXS/MariBank moving into SME/wholesale products).
- **Project Orchid** (Section 4.5): MAS' program for programmable payments and purpose-bound money — whitepaper (Nov 2023), industry playbook (2024), live pilots with DBS, OCBC, UOB, Standard Chartered, CIMB and partners. Singapore is the reference jurisdiction for "regulator building the runway for programmable money."
- **The SG business banking landscape:** SMEs are the backbone (and MAS/SG government actively digitize SME finance — from PayNow to SGFinDex-adjacent data sharing to GST digital services); the neo-business-bank cohort (Aspire, Volopay, plus GXS/MariBank SME pushes) competes with incumbents (DBS Treasury Prism, OCBC/UOB business banking APIs) on programmability.
- **The regulatory sandbox approach:** MAS' sandbox (and its fintech regulatory sandbox + the newer "sandbox express") lets programmable-payments pilots run with regulatory flexibility — the Orchid pilots ran this way. For a builder, SG offers the most *permissioned* environment to experiment with programmable banking (purpose-bound money included).
- **PayNow/FAST:** Singapore's instant payment rails (PayNow for P2P/proxy, FAST for bank-account transfers) are the real-time substrate — any SG programmable bank's payments layer sits on them.

---

## 11. The Future: 2026 and Beyond

### 11.1 Programmable Payments Mainstream: Purpose-Bound Money

The Orchid vision generalizes: programmable payments move from pilots to products — conditional disbursements (government credits, subsidies that can only be spent as intended), conditional supplier payments, programmable trade settlement. Purpose-bound money (money with attached conditions) becomes a *standard product category* in business banking, offered on both traditional rails (rules engines) and tokenized rails (tokenized deposits). The 2024 Orchid playbook gives the industry the pattern; 2026+ is the build-out.

### 11.2 Tokenized Deposits

The "bank money token" — a deposit represented on a ledger (permissioned DLT or centralized) that can be programmed and settled atomically. J.P. Morgan (JPM Coin) leads commercially; BNY, Citi, and others run tokenized-deposit pilots; BIS and central banks explore the wholesale side. For the programmable business bank, tokenized deposits are the *next money form*: conditional payments and atomic settlement become native (Section 4.6), and the five P's run on the token itself, not just on the instruction. The architectural question — programmability in the money vs. in the system of record (Orchid's key finding) — will be settled by commercial experience; the bank that supports *both* models hedges the transition.

### 11.3 The "Everything as API" Bank

The end-state of the Section 5 architecture: **every bank function is an API** — not just accounts/payments/cards, but onboarding, credit (API-driven lending decisions), trade finance, guarantees, compliance attestations, even the branch-equivalent services. The business bank becomes a *platform* in the platform-economics sense: a set of capabilities other software composes. The winners will be judged on API completeness, event richness, and DX — the developer platform (Layer 7) as the bank's primary product surface.

### 11.4 AI-Driven Banking Workflows

The convergence of programmable banking with AI agents (see [durable_ai_agent_workflows_guide.md](../technology/durable_ai_agent_workflows_guide.md) and [agentic_workflows_guide.md](../technology/agentic_workflows_guide.md)):

- **Agents initiating payments:** an AI agent (in the customer's ERP or the bank's own products) initiates, checks, and schedules payments against the payment APIs — the programmable bank is the *execution layer* for agentic finance (agents as the next "customer software").
- **Agents approving:** rule-governed approval delegation — an agent reviews and approves within encoded authority (limits, policies), with the approval itself recorded and auditable; human approval remains for out-of-policy cases (the rules engine *is* the agent's authority boundary).
- **Agentic treasury:** agents monitoring positions, executing FX hedges, optimizing sweeps — treasury as an autonomous loop over treasury APIs.
- **The bank's own AI:** the programmable bank uses agents internally (AML triage, KYC document review, anomaly investigation) — programmability applies to the bank's own operations, not just customer-facing APIs.
- **Governance implication:** agentic payments make Section 9.8's rule governance *more* important: the authority boundary for an agent is a set of rules; rule integrity *is* agent safety. Programmable banks will become the compliance substrate for agentic finance — which is a strong strategic position.

### 11.5 The Programmable Bank as Default Business Banking Model

The neo-business-bank stack (cloud core/ledger, API gateway, event backbone, VA sub-ledger, card platform, compliance engines, developer platform) becomes **table stakes** for any business banking proposition — the same way mobile banking became table stakes for retail. Incumbent SMEs banking will be API-fied (CitiConnect/HSBCnet/Treasury Prism-class programs become the minimum); new entrants will be programmable from day one; BaaS banks will be the shared infrastructure. The differentiation shifts from *having* programmability to *depth* of programmability: VA scale, event richness, conditional-payment maturity, AI-agent readiness, and compliance-as-code quality.

### 11.6 Regulation Catching Up

Regulation evolves from "allow" to "define": purpose-bound-money standards (MAS playbook → international convergence), programmable-compliance frameworks (rules-as-regulated-artifacts, Section 10.4), AI-agent accountability rules, and tokenized-deposit classification. Expect: **licensing regimes that recognize programmable banking as a distinct model** (bank-like platforms, BaaS bank categories), **cross-border PBM interoperability**, and **audit standards for automated controls**. The banks that built governance *into* their programmability (Section 9.8) will find regulation catching up to them as an advantage rather than a constraint.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **API-first** | Product design where the API is the primary product surface; UIs are clients of the API. |
| **Banking-as-a-Service (BaaS)** | Providing banking products (accounts, payments, cards) to other companies via API, usually on a partner bank's license. |
| **Business bank / commercial bank** | A bank serving businesses (SME, mid-market, corporate) rather than consumers. |
| **Compliance as code** | Compliance policies (KYC/KYB, AML, screening, limits) implemented as versioned, automated, auditable rules. |
| **Conditional payment** | A payment instruction that executes only when a stated condition is met (escrow-like release, milestone payment). |
| **Developer platform** | The product surface for integrating with the bank: docs, SDKs, sandbox, webhooks, API keys, rate limits. |
| **Embedded finance** | Financial products delivered inside non-financial products/software (the bank in the customer's workflow). |
| **Event backbone** | The message/streaming infrastructure (e.g., Kafka) carrying bank events (payment.settled, card.authorized) to internal and customer consumers. |
| **Five P's of programmable banking** | Programmable Accounts, Payments, Cards, Compliance, and Treasury — the five programmable layers. |
| **In-house banking** | A corporate running its own internal bank (internal accounts/netting/FX) using the bank's infrastructure, often on VAs. |
| **Instant payment** | A real-time, 24/7 retail payment rail (FAST, SEPA Instant, FedNow, UPI). |
| **KYB (Know Your Business)** | The business-entity analogue of KYC: verifying a company's identity, ownership, and risk profile. |
| **Limits engine** | The control component enforcing amount/velocity/merchant limits across payments, cards, and treasury. |
| **mTLS** | Mutual TLS: two-way certificate authentication for machine-to-machine API security. |
| **Multi-rail payments** | A payments engine supporting several rails (instant, ACH, SWIFT, card) behind one API. |
| **Oracle (payments)** | A verified external data source that triggers conditional payment conditions (delivery confirmation, inspection pass). |
| **Payment rule** | A condition governing payment execution (approval threshold, limit, compliance gate, business condition). |
| **Programmable payment** | A payment instruction with attached rules/conditions/automation (conditional, scheduled, rule-governed). |
| **Programmable business bank** | A business bank whose products are exposed as composable, embeddable, automatable APIs and rules. |
| **Project Orchid** | MAS' initiative (whitepaper Nov 2023, playbook 2024) studying and piloting programmable payments and purpose-bound money. |
| **Purpose-bound money (PBM)** | Digital money with attached conditions on how it may be used, enforced programmatically. |
| **Request-to-pay (RTP)** | A formal API-delivered request for payment sent to a payer's bank app. |
| **Sweep account** | An account whose balance is automatically moved (swept) per rules (e.g., to interest-bearing or pooling). |
| **Tokenized deposit** | A deposit represented on a programmable ledger (JPM Coin-style), enabling programmability and atomic settlement. |
| **Treasury APIs** | APIs for cash position, pooling, sweeps, FX, and forecasting — the treasury layer of the programmable bank. |
| **Virtual account (VA)** | A sub-ledger account behind a physical master account; infinite logical accounts for reconciliation and structure. |
| **Virtual card** | A digital card issued via API with programmable spend controls; created/destroyed per use case. |
| **Webhook** | An HTTP callback the bank calls to deliver events to the customer's endpoint (event-driven integration). |
| **Zero-balance account (ZBA)** | A sub-account whose balance is swept to a master account at day-end (physical pooling). |

---

## 13. Related Guides in This Repository

- [core_banking_systems_guide.md](core_banking_systems_guide.md) — cores, BaaS/composable banking, embedded finance, digital banks (Trust, GXS, MariBank, ANEXT), build-vs-buy-vs-BaaS.
- [payments_hub_guide.md](payments_hub_guide.md) — payment orchestration, ISO 20022, real-time payments, the payments hub architecture.
- [mojaloop_guide.md](mojaloop_guide.md) — real-time interoperable payment rails (the instant-payment substrate).
- [spec_driven_development_frameworks_guide.md](../technology/spec_driven_development_frameworks_guide.md) — OpenAPI/spec-first API design (the API discipline of the programmable bank).
- [banking_limits_domain_guide.md](banking_limits_domain_guide.md) — limits/velocity/approval domain model and governance (the rules engine's control layer).
- [event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) — Kafka, event-driven architecture, stream processing (the event backbone).
- [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) — banking ledger/data modeling (VA sub-ledger design).
- [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) — KYC/AML/screening systems (compliance engines).
- [nasdaq_calypso_guide.md](nasdaq_calypso_guide.md) — treasury management systems (the traditional treasury side the programmable bank automates).
- [blockchain_technology_guide.md](../technology/blockchain_technology_guide.md) — DLT/tokenization (tokenized deposits, programmable money rails).
- [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md) — banking microservices patterns.
- [durable_ai_agent_workflows_guide.md](../technology/durable_ai_agent_workflows_guide.md) / [agentic_workflows_guide.md](../technology/agentic_workflows_guide.md) — durable workflow orchestration (the rules/automation engine; AI-agent banking convergence).
- [llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) — API security discipline (analogies for the bank's API security).
- [on_prem_llm_deployment_guide.md](../technology/on_prem_llm_deployment_guide.md) — data residency/sovereignty patterns (regulated data planes).

---

*End of guide — The Programmable Business Bank. Feedback and corrections welcome via the repository.*
