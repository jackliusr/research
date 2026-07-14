# End-to-End Banking Processes: A Comprehensive Guide

> Technical deep-dive into cross-system, cross-department workflows spanning
> Front Office, Middle Office, and Back Office in wholesale/corporate banking.
>
> **Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB
> **Repository:** https://github.com/jackliusr/research
> **Last updated:** July 2026

---

## 1. What End-to-End Banking Processes Are

An **end-to-end (E2E) banking process** is a complete workflow spanning three
operational layers across multiple systems and departments.

| Layer | Role | Systems | Key Activities |
|-------|------|---------|----------------|
| **Front Office** | Client-facing revenue | CRM (Salesforce, Seismic), e-banking, Bloomberg/Reuters | Onboarding, trade execution, deal negotiation |
| **Middle Office** | Risk, compliance | Murex, Calypso, RiskMetrics, AML engines | Risk assessment, limit checks, confirmations |
| **Back Office** | Settlement, reporting | Core banking (T24, Finacle), Swift, settlement hubs | Matching, settlement, reconciliation, reporting |

No single system owns an E2E process — it lives in the orchestration and
**data flow between systems**. A payment instruction starts in FO, is validated
in MO, and settles in BO, but the client was onboarded weeks earlier through
different systems.

---

## 2. Account Opening & KYC

### Workflow

```
CRM Intake → Preliminary Screening → CDD/EDD → AML Checks →
Credit Approval → Account Setup (Core Banking) → Static Data
Propagation (Payments, Trading, Reporting) → Activation
```

### Key Systems

- **CRM/Onboarding:** Salesforce FS Cloud, Fenergo, Thomson Reuters One
- **Screening:** World-Check (LSEG), LexisNexis Bridger, Dow Jones Risk
- **Identity:** Onfido, Jumio, MyInfo (Singapore), eKYC
- **Core banking:** Temenos T24, Finacle, Mphasis Silverlake
- **Downstream:** Payments hub, trade finance, lending, treasury, reporting

### CDD/EDD Tiers (MAS Notice 626)

| Tier | Trigger | Requirements |
|------|---------|-------------|
| **Simplified CDD** | Low risk (govt, listed cos) | Basic ID verification |
| **Standard CDD** | Most corporate accounts | Identity, business nature, source of funds, UBO≥25%, org structure |
| **Enhanced DD (EDD)** | PEPs, high-risk jurisdictions, complex structures | Source of wealth, board-level approval, detailed review q6 months |

### Regulatory Touchpoints

- **MAS Notice 626** — Singapore AML/CFT requirements for banks
- **FATF Recommendations** — Intl CDD, record-keeping, STR standards
- **BSA/USA PATRIOT Act** — US requirements (dollar-clearing banks)
- **CRS/FATCA** — Automatic tax information exchange
- **GDPR / PDPA (SG)** — Data privacy constraints

### Timelines & Bottlenecks

- Standard corporate: 2–4 weeks; Complex (multi-jurisdictional): 6–12 weeks
- Top bottlenecks: Client document delays, beneficial ownership opacity,
  screening false-positive resolution, manual data entry to downstream systems

### Static Data Propagation

Once the account is live, client static data (legal name, LEI, account
numbers, tax ID) must reach every downstream system. Many banks still use
batch file transfers or re-keying — creating data inconsistency risks.

---

## 3. Payment Processing

### Full Lifecycle

```
Initiation (Client/Channel/API)
    → Validation (Format, balance, limits)
    → Enrichment (BIC, routing, FX, charges)
    → Sanctions Screening (OFAC, UN, EU, MAS)
    → AML Monitoring (Velocity, anomaly)
    → Compliance Review (if flagged)
    → Execution Engine
    → SWIFT/MEPS+/ACH/Internal Transfer
    → Confirmation & Advice
    → Reconciliation (MT940/950, camt.053/054)
    → Statementing
```

### Payment Rails & Message Formats

| Rail | Use Case | Format | Settlement |
|------|----------|--------|------------|
| **SWIFT MT** | Cross-border correspondent | MT103, MT202, MT910 | Correspondent chain |
| **SWIFT MX (ISO 20022)** | Modern cross-border | pacs.008, camt.053, camt.054 | Same, richer data |
| **SEPA (EU)** | Euro SCT/SDD | pacs.008, pacs.003 | TARGET2/STEP2 |
| **MEPS+ (SG)** | SGD RTGS high-value/urgent | ISO 20022 SCRIPS | Real-time via MAS |
| **CHIPS (US)** | Large USD clearing | Proprietary | Netting via Fed |
| **FedNow (US)** | US real-time retail | ISO 20022 | Real-time gross |
| **FPS/PromptPay (Asia)** | Domestic real-time | Various | Real-time |

### Exception Handling

- **STP failure:** Format error, missing field, invalid BIC
- **Sanctions hit:** True match or false positive — requires manual review
- **Limit/credit failure:** Insufficient balance or exceeded credit line
- **Repair/investigation:** Beneficiary bank rejects — incorrect account/name
- **MT199/n199:** Investigation messages for trace, recall, amendment

Each exception adds 1–10 days. Exceptions consume ~80% of ops effort but <5%
of volume.

### ISO 20022 Migration

As of mid-2026, SWIFT MT→MX migration is in its final phase. Impacts: richer
structured data per transaction, end-to-end tracking, dual-format capability
required during coexistence period ending November 2025.

---

## 4. Trade Finance

### Letter of Credit Lifecycle

```
1. Application — Buyer applies to issuing bank; credit check, limit drawdown
2. Issuance — Issuing bank sends LC via SWIFT MT700 to advising bank
3. Advising — Advising bank authenticates, notifies seller (beneficiary)
4. Amendment (if any) — Terms changed via MT707
5. Document Presentation — Seller ships goods, presents docs
   (bill of lading, invoice, packing list, cert of origin, insurance cert)
6. Document Checking — Issuing bank checks docs vs LC terms (UCP 600, Art 14)
   ~70% of first presentations have at least one discrepancy
7. Payment/Acceptance — If compliant: sight payment or deferred acceptance
8. Reimbursement — Issuing bank reimburses advising (MT202 cover)
```

### Key SWIFT Messages

| Message | Purpose | Direction |
|---------|---------|-----------|
| **MT700** | Issue documentary credit | Issuing→Advising |
| **MT707** | Amendment to credit | Issuing→Advising |
| **MT720** | Transfer of credit | Transferring→Beneficiary |
| **MT760** | Demand guarantee / Standby LC | Guarantor→Beneficiary |

### Bank Guarantees & Collections

- **Guarantees (MT760):** Bid bonds, performance bonds, advance payment
  guarantees, retention bonds
- **Documentary Collections:** Simpler than LCs — D/P (documents vs payment)
  or D/A (documents vs acceptance). Governed by URC 522.

### Key Systems

- Trade engines: Finastra Trade Innovation, Surecomp, Bolero, Contour
- SWIFT connectivity: Alliance Access/Gateway
- Document management: Trade document imaging systems

### Key Risks

- **Fraud:** Duplicate invoicing, phantom shipments, forged documents
- **Discrepancies:** ~70% hit rate on first presentation — delays, fees,
  rejection risk
- **Country/bank risk:** Issuing bank may repudiate in volatile jurisdictions
- **Regulatory:** Sanctions screening on all parties, dual-use goods controls

---

## 5. Corporate / Wholesale Lending

### Full Lifecycle

```
Origination & Pitch → Term Sheet & Mandate → Credit Assessment →
Documentation & Legal (Loan Agreement, Security) → Conditions Precedent
(CPs) → Facility Setup (Loan System) → Drawdown/Utilization →
Servicing (Interest, Fees, Rollovers) → Covenant Monitoring →
Repayment/Maturity → Rollover or Closure
```

### Key Systems

| Platform | Vendor | Scope |
|----------|--------|-------|
| **Loan IQ** | Finastra | Syndicated/bilateral loan mgmt, agent bank, fee schedules |
| **ACBS** | FIS | Full commercial loan lifecycle — origination to accounting |
| **CRISPR** | FIS | Legacy loan origination and credit processing |

### Syndicated Loan Mechanics

- **Allocation:** Agent distributes loan amounts across syndicate members
- **Drawdowns:** Agent collects request → notifies syndicate → funding via wire
- **Interest:** Benchmark (SOFR/SORA/EURIBOR) + margin, collected from borrower,
  distributed to syndicate
- **Fees:** Arrangement, commitment, and utilization fees — each differently
  calculated

### Covenant Monitoring

1. Borrower submits periodic financial statements
2. Ratios (debt/EBITDA, interest coverage, leverage) compared to thresholds
3. Breach → waiver request → credit committee review → acceleration or waiver

### Collateral Management

- **Valuation:** Periodic revaluation of pledged assets (real estate,
  receivables, inventory, shares)
- **Perfection:** Legal registration of security interest (charges, mortgages)
- **Monitoring:** Collateral coverage ratio tracking
- **Enforcement:** Realization on default

---

## 6. Securities & Treasury

### Trade Lifecycle

```
Order Management (OMS)
    → Execution (Exchange/MTF/Broker)
    → Trade Capture (FO)
    → Confirmation/Affirmation (Matching)
    → Clearing (CCP)
    → Settlement (DVP/FOP)
    → Custody & Asset Servicing
    → Corporate Actions Processing
    → Portfolio & Performance Reporting
```

### Front Office Systems

- **Bloomberg AIM/TOMS** — Fixed income and derivatives OMS
- **Reuters Eikon** — Equities, FX, fixed income market data + execution
- **Murex MX.3** — Cross-asset front-to-back trading and risk
- **Calypso** — Treasury, derivatives, capital markets
- **Charles River (CRD)** — Multi-asset OMS for institutional managers

### Middle Office

- Trade confirmation/affirmation (DTCC CTM, MarkitServ, Bloomberg)
- P&L, VAR, Greeks calculation
- Margin/collateral calls (IM & VM for derivatives)
- Regulatory trade reporting (EMIR, MiFID II, MAS 610)

### Clearing & Settlement

| Venue | Asset Class | Settlement | Standard |
|-------|-------------|------------|----------|
| Euroclear/Clearstream | Bonds, equities | DVP via T2S | T+2 |
| DTCC (US) | Equities, treasuries | DVP via NSCC | T+1 (since May 2024) |
| CCP (LCH, CME, SGX) | Derivatives, repos | Central clearing | T+1 to T+2 |

**DVP (Delivery vs Payment):** The fundamental model — securities delivered
only if corresponding cash payment is made. Eliminates principal risk.

### Custody & Corporate Actions

**Flow:** Announcement (MT564/seev.30) → Validation (mandatory/voluntary) →
Election (if voluntary) → Ex-date tracking → Payment (MT566/seev.40) →
Reconciliation

**Types:**
- **Mandatory:** Dividends, stock splits, par value changes
- **Voluntary:** Tender offers, rights issues, exchange offers (client elects)
- **Mandatory w/ choice:** Scrip dividend (cash or stock)

**Systems:** XSP (SmartStream), TCS BaNCS Custody, Euroclear/DTCC interfaces

---

## 7. Regulatory Reporting

### Key Regulations

| Regulation | Jurisdiction | Scope | Frequency |
|-----------|-------------|-------|-----------|
| **MAS 610** | Singapore | Derivatives trade reporting | T+1 |
| **MAS 649** | Singapore | OTC derivative risk mitigation | Ongoing |
| **BCBS 239** | Global (Basel) | Risk data aggregation & reporting | Principles-based |
| **EMIR** | EU | Derivatives reporting to ESMA TRs | T+1 |
| **MiFID II/MiFIR** | EU/EEA | Transaction reporting, transparency | Daily |
| **SFTR** | EU/EEA | Securities financing transactions | T+1 |
| **Dodd-Frank** | US | Swap reporting to SDRs | T+1 |
| **CRS/FATCA** | Global/US | Tax info exchange | Annual |

### Data Aggregation Challenges

```
Source Systems (20+) → ETL → Data Lineage Mapping → Data Quality Checks
→ Transformation → Report Generation (XML/CSV/ISO 20022) →
Submission to Regulator/TR → Reconciliation
```

**Key issues:**
- **Data lineage:** Tracing a reported field to its source is often impossible
  without heavy manual effort
- **Data quality:** Missing LEIs, incorrect UTI/UPI, stale reference data
- **Reconciliation:** Same trade reported to multiple regulators — each expects
  a slightly different view (e.g., EMIR vs MAS 610 for the same swap)
- **Timeliness:** T+1 requires hours-latency processing; batch often fails

### BCBS 239 — 14 Principles

| Category | Principles |
|----------|------------|
| **Governance & Infrastructure** | 1–3: Data governance, architecture, IT infra |
| **Risk Data Aggregation** | 4–8: Accuracy, completeness, timeliness, adaptability |
| **Risk Reporting** | 9–11: Accuracy, comprehensiveness, clarity, frequency |
| **Supervisory Review** | 12–14: Remedial actions, home-host coordination |

### Example: EMIR/MAS 610 Reporting Flow

```
Trade Captured in FO → Enriched (UTI, LEI, UPI assigned) →
Validation Rules Applied → Report Packaged (XML per regulator schema) →
Submitted to TR (DTCC GTR, REGIS-TR, UnaVista, Bloomberg) →
Reconciliation (both counterparties matched) →
Discrepancy Resolution (manual amendment if mismatched)
```

---

## 8. Process Integration Patterns

### How Processes Connect

```
                  ┌──────────────────┐
                  │ Client Onboarding│ ← feeds downstream
                  │   (KYC/Static   │
                  │    Data Hub)     │
                  └────────┬─────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
     ┌────┴────┐     ┌────┴────┐     ┌─────┴─────┐
     │ Payments│     │ Lending │     │   Trade   │
     │ SWIFT/  │     │ LoanIQ, │     │  Finance  │
     │ SEPA/   │     │ ACBS    │     │  MT700/LC │
     │ MEPS+   │     │         │     │           │
     └────┬────┘     └────┬────┘     └─────┬─────┘
          │               │                │
          └───────────────┼────────────────┘
                          │
                    ┌─────┴─────┐
                    │ Treasury  │
                    │ (Funding, │
                    │ FX, Murex)│
                    └─────┬─────┘
                          │
                    ┌─────┴─────┐
                    │Regulatory │
                    │ Reporting │
                    └───────────┘
```

### Key Integration Scenarios

- **Payment from lending:** Drawdown on loan generates payment — Loan IQ sends
  to payment engine → sanctions → SWIFT → settlement confirmation feeds back
- **Settlement funding from treasury:** Payment engine checks nostro balance in
  Murex → treasury executes FX/money market deal to fund nostro before cut-off
- **Trade finance & lending:** LC issuance may utilize credit facility — trade
  system checks available limit in Loan IQ before issuing MT700
- **Static data propagation:** Client name/address change in CRM → event
  published → reference data hub propagates to payments, trade, lending,
  treasury, reporting

### Integration Technologies

| Pattern | Tech | Use Case |
|---------|------|----------|
| Batch file | SFTP, MQ file transfer | EOD positions, statements |
| API (REST/SOAP) | RESTful APIs, Web Services | Real-time balances, limits |
| Event-driven | Kafka, IBM MQ, TIBCO | Trade events, static data changes |
| Database link | JDBC/ODBC | Reporting warehouses |
| Message queues | MQ Series, ActiveMQ, RabbitMQ | Payment instructions, confirmations |

---

## 9. Common Pain Points

### Manual Handoffs

Most common issue: processes step through System A → manual export → System B
→ manual re-key → System C. 15–40% of operations headcount dedicated to manual
data transfer.

### Data Inconsistency Across Silos

Same client has different addresses in CRM vs payments vs lending. Account
opened in core banking but not in trade finance. LEI/UBO stale after
restructuring. Root cause: no golden source for reference data.

### Batch Processing Delays

T+1 batch cycles cause: late payments queue to next day, regulatory reports
relying on batch feeds with no room for error, overnight nostro reconciliation
hides daylight issues for 24 hours.

### Exception Handling Complexity

SWIFT investigations (MT199) require manual analysis. LC discrepancies require
phone/email/fax chains. Trade settlement fails need manual recovery workflow
selection. 80% of ops effort on <5% of volume.

### Regulatory Reconciliation

Two counterparties report same trade differently. Internal systems disagree
with TR positions. MAS fined 14 FIs in 2025 for compliance breaches.

### Fragmented Vendor Landscape

A mid-tier corporate bank may run 40+ systems. Each has its own data model,
schedule, and interface protocol. Upgrading one requires testing integrations
with 15+ downstream systems.

---

## 10. Modernization Patterns

### Straight-Through Processing (STP)

**Goal:** Transaction from initiation to settlement with zero manual
intervention.

**Enablers:** Standardized formats (ISO 20022), rules-based validation (90%+
format/limit checks), automated repair, ML for complex trade STP.

**Progression:** 40–60% baseline → 65–80% after format standardization →
85–95% with ML-enhanced exception handling.

### API-Based Integration

Replacing file/MQ integrations with RESTful APIs:
- **Open Banking APIs:** Client/third-party payment initiation, balance checks
- **Internal APIs:** Decouple front-end from back-end — same API for mobile,
  web, and corporate treasury
- **API gateways** (Kong, Apigee, Gravitee): Security, rate limiting, protocol
  transformation

### Event-Driven Architecture (EDA)

Instead of polling or batch jobs, systems publish events that consumers
subscribe to:

```
Trade Executed → Event (Kafka: trades.raw)
  → Risk Service (recalculates exposure)
  → Reporting Service (generates regulatory report)
  → Confirmation Service (sends MT300)
  → Settlement Service (initiates clearing)
  → Accounting Service (books entry)
```

**Benefits:** Near real-time, loose coupling, resilient (events persist for
replay). **Challenges:** Schema evolution, exactly-once semantics, monitoring.

### Process Automation (RPA)

Fills gaps where full STP is not feasible:

| Task | RPA Approach |
|------|-------------|
| Static data propagation | Bot reads core update, logs into downstream systems |
| Sanctions false-positive resolution | Screens match details, flags or approves |
| Statement reconciliation | Compares MT950 vs core banking records |
| Trade confirmation matching | Extracts PDF confirmations, matches to FO data |

**Caveat:** RPA is a bridge, not destination — each bot is fragile. Long-term,
APIs and EDA replace RPA.

### Real-Time vs Batch Trade-Offs

| Aspect | Real-Time | Batch |
|--------|-----------|-------|
| Latency | ms–sec | Hours–T+1 |
| Throughput | Per-transaction cap | High — volume-optimized |
| Complexity | High (eventual consistency, idempotency) | Lower |
| Cost | Higher infra | Lower |
| Best for | Payments, execution, limit checks | Statements, reconciliations |

Most modern banks adopt a **hybrid model:** real-time for client-facing and
risk-critical paths, batch for non-time-sensitive BO processing, event-driven
triggers bridging the two.

### Target Architecture (Phased)

```
Phase 1 — Stabilize
  → Standardize formats (ISO 20022)
  → Centralize reference data (golden source)
  → API gateway for internal integration

Phase 2 — Automate
  → RPA for highest-volume manual processes
  → STP rules engine
  → Event bus for key process events (trade, payment, KYC)

Phase 3 — Transform
  → Microservices replacing monolithic core banking
  → Real-time regulatory reporting
  → Cloud-native (containerized, managed Kafka, serverless)
  → ML for exception prediction and auto-resolution
```

---

## Appendix: Key Systems Reference

| Domain | Major Systems |
|--------|---------------|
| Core Banking | Temenos T24, Finacle, Mphasis Silverlake, FIS Profile |
| Payments | Finastra Global PAYplus, Bottomline, FIS Payment Hub, Volante |
| Trade Finance | Finastra Trade Innovation, Surecomp, Bolero, Contour |
| Lending | Finastra Loan IQ, FIS ACBS, FIS CRISPR, TCS BaNCS |
| Treasury/Capital Markets | Murex MX.3, Calypso, Bloomberg AIM/TOMS, ThinkFolio |
| KYC/Screening | Fenergo, World-Check (LSEG), LexisNexis Bridger |
| Regulatory Reporting | AxiomSL (RegTek), Vermeg, Wolters Kluwer OneSumX, DTCC GTR |
| Risk & Analytics | Murex, RiskMetrics (MSCI), Moody's RiskFrontier |
| Settlement & Custody | Euroclear, Clearstream, DTCC, T2S, SGX-DC |
| SWIFT Connectivity | Alliance Access, Alliance Gateway, SWIFT for Corporates |

---

## Further Reading

- **MAS Notice 626** — AML/CFT for Singapore banks
- **BCBS 239** — Risk data aggregation and reporting principles
- **UCP 600** — Uniform Customs for Documentary Credits
- **ISO 20022** — Financial messaging standard
- **FATF Recommendations** — International AML/CFT standards
- **SWIFT MT & MX Message Reference Guides** — SWIFT documentation

---

*This guide is maintained as part of the research repository at
https://github.com/jackliusr/research. Contributions welcome.*
