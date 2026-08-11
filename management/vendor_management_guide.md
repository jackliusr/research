# Vendor Management: A Comprehensive Guide

> **Author's Note:** This guide is the dedicated deep-dive on **vendor management** — the discipline of managing third-party suppliers across the full lifecycle, from strategy and selection through contracting, onboarding, performance, relationship, risk (VRM/TPRM), and offboarding — with the enterprise IT and banking-outsourcing context throughout (Crédit Agricole CIB, Singapore). It is written for solution architects, IT managers, and business managers who own or interact with vendor relationships and want the *working* model: the lifecycle, the roles, the artifacts (scorecards, risk registers, QBR agendas), and the regulatory overlay. Where a topic has a dedicated sibling guide, this guide cross-references instead of duplicating: the *make-vs-buy / sourcing strategy* lives in [strategic_management_guide.md](strategic_management_guide.md), the *negotiation and stakeholder skills* live in [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), the *regulatory side of outsourcing and third-party risk* (MAS Notice 658, DORA, BCP, sanctions/AML) lives in [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md), the *cost/TCO lens* lives in [../technology/finops_guide.md](../technology/finops_guide.md), and the *procurement-management chapter of project delivery* lives in [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md).
> *Part of the Management & Leadership Series | Author: Jack Liu Shurui — Solution Architect, Crédit Agricole CIB*
> *Related guides in this series: [strategic_management_guide.md](strategic_management_guide.md) · [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) · [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) · [360_management_guide.md](360_management_guide.md) · [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) · [grow_team_guide.md](grow_team_guide.md) · [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) · [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) · [the_managers_path_research.md](the_managers_path_research.md)*

---

## How to Use This Guide

This guide is a reference manual and a field kit. **For a quick orientation**, read Section 1 (the discipline, roles, and value) and Section 10 (vendor management in the bank/IT context). **For a specific problem** — a selection process, a contract, a struggling vendor, a risk concern — jump straight to the matching section: strategy and segmentation → Section 2, selection/sourcing → Section 3, contracting → Section 4, onboarding → Section 5, performance → Section 6, relationships → Section 7, risk → Section 8, exit → Section 9. **For the enterprise reality check**, Section 10 grounds everything in the bank/IT context (core banking vendors, cloud, the VMO, maturity). **To see it all assembled**, Section 11 walks a core banking vendor relationship end-to-end with real artifacts (scorecard, risk register excerpt, QBR agenda). Each section is self-contained; the glossary at the end defines every term in one place.

Two reading notes. First, a recurring distinction runs through the whole guide: **vendor management vs procurement**. Procurement is the *transactional front end* — sourcing, tendering, buying at the best price. Vendor management is the *relationship back end* — making sure what was bought keeps performing, stays low-risk, and stays aligned. Most of this guide is about the back end, but selection (Section 3) and contracting (Section 4) sit on the boundary and are covered from the vendor-management vantage point. Second, **risk is a cross-cutting overlay, not just a stage**: the lifecycle is drawn as strategy → selection → contracting → onboarding → performance → relationship → risk → offboarding for teaching purposes, but in a mature organisation risk management touches every stage (see Section 8). Where a claim in this guide is contested, unverifiable, or a simplification of a shifting regulatory landscape, it is flagged honestly rather than glossed over.

---

## Table of Contents

1. [The Vendor Management Discipline](#1-the-vendor-management-discipline)
2. [Vendor Strategy and Segmentation](#2-vendor-strategy-and-segmentation)
3. [Vendor Selection and Sourcing](#3-vendor-selection-and-sourcing)
4. [Contracting](#4-contracting)
5. [Onboarding and Implementation](#5-onboarding-and-implementation)
6. [Performance Management](#6-performance-management)
7. [Relationship Management](#7-relationship-management)
8. [Vendor Risk Management (VRM/TPRM)](#8-vendor-risk-management-vrmtprm)
9. [Offboarding and Exit](#9-offboarding-and-exit)
10. [Vendor Management in the Enterprise (Bank/IT)](#10-vendor-management-in-the-enterprise-bankit)
11. [Worked Example — Managing a Core Banking Vendor](#11-worked-example--managing-a-core-banking-vendor)
12. [Future Trends (2026+)](#12-future-trends-2026)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. The Vendor Management Discipline

### 1.1 What vendor management is

**Vendor management** (also called **supplier management** — the terms are interchangeable, see the glossary) is the discipline of managing an organisation's third-party suppliers across the **full lifecycle** of the relationship, with the goal of extracting maximum value at acceptable risk. It is the systematic answer to a deceptively simple question: *once you have signed a contract with an external party, how do you make sure the deal keeps delivering?*

The canonical lifecycle used throughout this guide has **eight stages**:

```
Strategy → Selection → Contracting → Onboarding → Performance → Relationship → Risk → Offboarding
```

| Stage | Core question | Owner (typical) |
|---|---|---|
| **Strategy** | Which suppliers do we need, and how important is each? | Procurement strategy / business + VMO |
| **Selection** | Who is the best supplier for this need? | Procurement (sourcing) |
| **Contracting** | What are the terms, SLAs, and exit rights? | Procurement + Legal / Contract Manager |
| **Onboarding** | How do we get the supplier live safely? | Vendor Manager + IT / InfoSec |
| **Performance** | Is the supplier delivering what was promised? | Vendor Manager / SRM |
| **Relationship** | How do we govern and grow the partnership? | Vendor Manager + Exec Sponsor |
| **Risk** | What could go wrong, and how do we contain it? | Risk / TPRM + Vendor Manager |
| **Offboarding** | How do we exit cleanly when the time comes? | Vendor Manager + Procurement / Legal |

Three honest caveats about this picture. First, **the lifecycle is not strictly sequential** — contracting terms shape selection criteria, risk assessment starts before selection (due diligence), and offboarding obligations are written in Section 4, long before Section 9. Second, **risk is a cross-cutting overlay**: in mature organisations every stage has a risk checkpoint rather than risk being one discrete stage. Third, **industry models differ in granularity** — World Commerce & Contracting (WorldCC) frames the contract lifecycle (create → manage → perform → renew/exit); ITIL frames supplier management within service management (strategy → onboarding → categorisation → performance → exit); Gartner/Forrester vendor-management lifecycles typically run 5–8 stages with risk embedded. The eight-stage model above is this guide's organising framework, not a universal standard.

### 1.2 Vendor management vs procurement

The single most common confusion in this field. The clean distinction:

| Dimension | Procurement | Vendor Management |
|---|---|---|
| **Phase** | Front end — before and at signature | Back end — after signature, for the life of the relationship |
| **Core activities** | Sourcing, RFx, tendering, negotiation, purchasing, POs | Performance monitoring, relationship governance, risk, renewal, exit |
| **Orientation** | Transactional — get the best deal | Relational — make the deal keep working |
| **Primary measures** | Savings, cost avoidance, cycle time, contract coverage | SLA compliance, scorecard, risk exposure, value delivered |
| **Time horizon** | One sourcing event to award | Years, through renewal and exit |
| **Key tools** | RFI/RFP/RFQ, e-sourcing, P2P (procure-to-pay) | Scorecards, QBRs, risk registers, contract management systems |

Procurement is sometimes split further: **sourcing** (finding and qualifying suppliers, running the competitive process) and **purchasing** (ordering, invoicing, payment — the P2P transactional tail). Vendor management is what happens *after* award. In practice the boundary is blurred: procurement teams often own the contract and the renewal, vendor management teams often feed requirements back into sourcing. The failure mode to avoid is the **"sign and forget"** gap — procurement treats award as the finish line, no one owns the relationship, and performance decays silently until renewal.

### 1.3 The roles: vendor manager vs procurement manager vs contract manager vs supplier relationship manager

Four roles get conflated. The distinctions matter for accountability:

| Role | Primary focus | Typical accountability | Typical activities |
|---|---|---|---|
| **Procurement Manager** | Sourcing and buying — the front end | Getting the right supplier at the right total cost | Market analysis, RFx, tender, negotiation, award, P2P |
| **Contract Manager** | The contract as an asset | Contract accuracy, compliance, and lifecycle (create → manage → perform → renew/exit) | Drafting, obligations tracking, amendments, renewals, contract repository |
| **Vendor Manager** | The relationship and its delivery after award | Ongoing performance, risk coordination, value delivery | Scorecards, QBRs, escalation, onboarding/offboarding, day-to-day interface |
| **Supplier Relationship Manager (SRM)** | The strategic partnership layer | Value creation with the most important suppliers | Joint roadmaps, executive engagement, innovation, performance-to-partnership lift |

In many organisations **vendor manager and SRM are the same person** at different altitudes — the SRM title usually implies the *strategic* tier of vendors (Section 2.2), while vendor manager covers the whole portfolio. **Contract manager is the distinct profession** — WorldCC's discipline of commercial and contract management — and is often a separate role in banks, sitting in Legal or a commercial function, while the vendor manager sits in IT or the business. A useful mental model: **procurement finds and buys, contract management protects the agreement, vendor management runs the relationship, SRM grows the partnership.** One person can wear several hats in small organisations — the trap is then letting the transactional hat crowd out the relational one.

### 1.4 Strategic vs transactional vendor management

Vendor management effort should scale with vendor importance. The spectrum:

- **Transactional vendor management** applies to commodity purchases — stationery, utilities, low-value SaaS. The "management" is lightweight: contract on file, auto-renewal checks, invoice verification, a scorecard exception. Over-managing these wastes money; the goal is *low-touch efficiency* (and often consolidation into fewer suppliers, Section 2.4).
- **Strategic vendor management** applies to the top tier of vendors — core banking platforms, cloud providers, major outsourcers. The relationship is treated as a **strategic partnership**: executive sponsors on both sides, **joint roadmaps** (the vendor's product roadmap aligned with the bank's strategy), innovation reviews, co-investment, and multi-year planning. The goal is *value creation*, not just risk containment.

The Kraljic matrix (next) is the classic tool for deciding which vendors get which treatment. The strategic/transactional split is also the difference between managing a *supplier* (commodity, replaceable) and managing a *partner* (interdependent, co-invested) — and the single biggest determinant of how a vendor manager spends their week.

### 1.5 The Kraljic matrix

The **Kraljic portfolio matrix** (Peter Kraljic, "Purchasing Must Become Supply Management," *Harvard Business Review*, 1983) is the foundational portfolio model for supplier management: a **2×2** that classifies purchases/suppliers by **profit impact / strategic importance** (vertical axis, low→high) against **supply market complexity / risk** (horizontal axis, low→high):

| | Low supply risk | High supply risk |
|---|---|---|
| **High profit impact** | **Leverage items** — competitive market, high value. *Strategy: exploit buying power, competitive bidding, negotiate hard.* | **Strategic items** — high value, few suppliers, hard to replace. *Strategy: partnership, long-term contracts, joint development, de-risk with second source.* |
| **Low profit impact** | **Routine (non-critical) items** — low value, easy market. *Strategy: simplify, standardise, automate, consolidate, minimise management effort.* | **Bottleneck items** — low value but supply-constrained (single source, proprietary). *Strategy: secure supply, long-term agreements, stock/backup, find substitutes.* |

Kraljic's insight: *one purchasing strategy does not fit all*. Applying partnership-level management to routine items over-spends; treating strategic items as commodity bids under-protects the business. In vendor management the matrix is used both **pre-contract** (what sourcing strategy for this category?) and **post-contract** (how much governance does this supplier warrant?). The four quadrants map cleanly onto the tiering model in Section 2.2 — strategic items ≈ tier 1, leverage ≈ tier 2, bottleneck and routine ≈ tier 3 with different treatment (bottleneck needs supply-security attention even at low spend).

### 1.6 The value of vendor management

Why invest in the discipline at all? Four value levers:

| Value lever | Mechanism | Example (bank/IT) |
|---|---|---|
| **Cost optimisation** | Renegotiation, consolidation, SLA enforcement, avoiding surprise exit costs | Re-tendering a managed-services contract at renewal; enforcing service credits on a breached SLA |
| **Risk reduction** | Due diligence, monitoring, audit, concentration management, contract protections | Catching a financially distressed cloud reseller early; fourth-party flow-downs |
| **Innovation access** | Joint roadmaps, early access to vendor features, co-development | A core banking vendor's cloud-native roadmap aligned with the bank's transformation |
| **Quality** | Performance measurement, continuous improvement, accountability | Scorecard-driven defect reduction in an outsourced application-support contract |

The discipline's return on investment is largely **avoided loss**: avoided downtime, avoided contract disputes, avoided regulatory findings, avoided failed migrations. Because these are invisible when they don't happen, vendor management is chronically under-funded until an incident makes it visible — a pattern worth naming in any business case for a VMO.

### 1.7 The Vendor Management Office (VMO)

The **Vendor Management Office (VMO)** is the central function that owns the *discipline* across the organisation: policy, process, tools, standards, and reporting for how vendors are managed. It typically:

- Defines the **vendor management framework** (lifecycle, roles, governance gates) and the **vendor policy**;
- Runs the **vendor portfolio** — segmentation, spend analysis, concentration reporting (Section 2.3);
- Owns **common tools and data** — the vendor register, contract repository, risk register, scorecard standards;
- Sets **governance cadence** — QBR standards, escalation thresholds, board/risk-committee reporting;
- Coordinates **TPRM** with Risk (Section 8) and **audit readiness**;
- Builds **capability** — training, career paths for vendor managers, communities of practice.

The VMO can be centralised (one team for the whole enterprise), federated (central standards + embedded vendor managers in business units), or virtual (a working group of vendor managers from different units). Banks and large IT organisations usually run a **federated model**: a small central VMO/TPRM office sets the framework, while IT and business units run day-to-day relationships against it. The VMO's existence is also the difference between "we have vendors" and "we manage vendors" — and it is the natural home for the maturity journey in Section 10.5.

---

## 2. Vendor Strategy and Segmentation

### 2.1 Make vs buy vs partner

Vendor strategy starts before any vendor exists: the **make vs buy decision**. Three options, plus the hybrid:

- **Build (make):** develop and run the capability in-house. Justified when the capability is *core* to competitive advantage, when the market offers nothing adequate, or when control/regulatory constraints demand it (e.g., certain risk engines, proprietary analytics).
- **Buy:** purchase the capability from the market as a product or service. Justified when the market is mature, the capability is *non-differentiating* (everyone needs it — payroll, email, general ledger), or speed/time-to-market beats build cost.
- **Partner:** co-develop or co-invest with a vendor whose roadmap you influence. Justified when the capability is important but the vendor can build it better/faster than you (e.g., a core banking platform where the vendor amortises R&D across many banks).
- **Hybrid:** buy the platform, build the customisation (the dominant pattern in banking — Temenos/Flexcube core plus deep local customisation).

The full framework — core vs non-core, transaction-cost economics, capabilities, exit costs — is covered in the strategy guide: see [strategic_management_guide.md](strategic_management_guide.md) for the make-vs-buy analysis in depth. Two vendor-management-relevant notes: (1) **the decision is never final** — every make-vs-buy decision has a built-in "switch" option (insourcing, Section 2.6) that should be re-examined at renewal; (2) **partner is not a soft option** — partnerships require governance, joint planning, and honest disagreement mechanisms, which is precisely what Sections 6–7 exist to provide.

### 2.2 Vendor segmentation and tiering

Once the vendor base exists, **segmentation** decides how much management each vendor gets. The common model is three tiers:

| Tier | Name | Typical criteria | Governance depth |
|---|---|---|---|
| **Tier 1** | Strategic / critical | High spend **and/or** high criticality (failure = major business impact), single-source dependency, customer-data access, regulatory materiality | Full: exec sponsor, joint steering committee, QBR, deep TPRM, BCP alignment, annual strategic review, succession plan |
| **Tier 2** | Important / managed | Moderate spend, important but replaceable, limited data access | Standard: named vendor manager, monthly ops + QBR-lite, risk assessment at onboarding + periodic, SLA monitoring |
| **Tier 3** | Commodity / routine | Low spend, low criticality, competitive market | Light: contract + auto-renewal control, exception-based monitoring, consolidation candidate |

Segmentation criteria in practice (combine, don't pick one):

- **Spend** — annualised spend with the vendor (and trend);
- **Criticality** — impact on business operations, customers, or regulatory obligations if the vendor fails (the dominant criterion in banking);
- **Risk** — inherent risk: data sensitivity, security posture, jurisdiction, concentration;
- **Dependency** — substitutability: how hard/expensive is replacement (Kraljic's supply-risk axis).

Two honest caveats. First, **tier names vary by institution** — some banks use "critical / high / medium / low" (the TPRM convention, Section 8.3) rather than tier 1/2/3; the regulatory world cares about *materiality* (what MAS calls "material outsourcing"), not the tier label. Second, **tiers must be re-scored periodically** — a tier 3 vendor can become tier 1 overnight if it becomes the single source for a new critical service, or if it is acquired. Tiering is a snapshot, not a verdict.

### 2.3 The vendor portfolio: spend analysis and concentration risk

**Vendor portfolio analysis** treats the vendor base as one portfolio to be shaped, not a pile of individual contracts:

- **Spend analysis** — total spend by vendor, by category, by business unit; the 80/20 rule (a handful of vendors usually absorb most spend); trend and forecast. This is the VMO's core dashboard and the entry point for every consolidation discussion.
- **Concentration risk** — the danger of **single-vendor dependency**: one vendor (or one cloud, one platform) carrying a disproportionate share of critical services. Concentration shows up as a *portfolio-level* risk that no single contract review will catch; regulators ask for it explicitly (e.g., cloud concentration in MAS's cloud guidelines and DORA's focus on critical ICT third-party providers). Portfolio metrics include the share of critical services per vendor and Herfindahl-style concentration indices (a tool borrowed from antitrust economics; not a regulatory requirement, just a useful quantification).
- **Portfolio shaping levers** — consolidation (2.4), multi-sourcing (2.5), insourcing (2.6), and deliberate "second-source" policies for the most critical categories.

### 2.4 Vendor consolidation

**Vendor consolidation** is the deliberate reduction of the vendor base: fewer vendors, each with more spend, in exchange for better commercial terms, standardised integration, and less management overhead. Classic moves: collapsing 40 point-solution SaaS vendors into 5 platform vendors; consolidating regional managed-services contracts into one global one; standardising on one cloud provider per workload class.

| Pros | Cons |
|---|---|
| Volume discounts and better pricing | **Increased concentration risk** (the exact risk Section 2.3 warns about) |
| Fewer contracts, simpler governance | Reduced competition at renewal (switching is harder) |
| Standardisation, fewer integrations | Loss of best-of-breed capability in niches |
| Less management overhead | Single points of failure if the vendor stumbles |

The discipline's answer is **consolidate, then de-risk**: consolidation for commodity categories (where concentration is harmless), but *managed* multi-sourcing for strategic categories (Section 2.5). Consolidation trends in 2026 are discussed in Section 12.

### 2.5 Multi-sourcing and second-source

**Multi-sourcing** is the deliberate use of more than one vendor for the same or adjacent services — the portfolio-level counterweight to concentration. Forms:

- **True multi-vendor:** two vendors split a category (e.g., two cloud providers, two core-adjacent platforms) — maximum resilience, maximum cost.
- **Second-source:** one primary vendor plus a qualified, contracted-but-dormant alternative — moderate cost, real leverage ("we have a credible alternative" is the strongest negotiation position in commercial life).
- **Best-of-breed:** different vendors per sub-capability, integrated — capability quality over simplicity.

Multi-sourcing costs more (duplicate licences, integration, split governance) and its benefit is **insurance**: continuity when a vendor fails, leverage at renewal, and a hedge against vendor roadmap risk (a vendor's product direction no longer being a veto over your strategy). In banking, second-sourcing is common for payments and channels, rare for core banking (where the integration cost is prohibitive — see the worked example, Section 11).

### 2.6 Insourcing

**Insourcing** (bring-back-in-house) is the strategic option of reversing an outsourcing decision: taking a capability back from a vendor into the organisation. It is the *exit strategy made real* — and the reason every outsourcing contract needs the exit clauses of Section 4 and the offboarding discipline of Section 9.

Triggers for insourcing: cost (the vendor's margin no longer justified), quality (persistent underperformance), **strategic control** (the capability became core), regulatory pressure, vendor instability, or simply the expiry of a long outsourcing deal where in-house capability has since been rebuilt. The cost reality: insourcing pays for **transition** (hiring or re-hiring, knowledge transfer, run-off of vendor staff, data and IP repatriation) and often under-estimates the *shadow costs* the vendor used to absorb (licences, tools, support coverage). It is a portfolio decision, not a grudge decision — the same analysis as make-vs-buy (2.1) applied with today's facts. For the cloud variant of this (exiting a cloud provider), see [../technology/cloud_providers_guide.md](../technology/cloud_providers_guide.md) on cloud exit and portability.

---
## 3. Vendor Selection and Sourcing

### 3.1 The sourcing process: RFI → RFP → RFQ → tender

Selection is run by procurement, but the *vendor manager* inherits its outcome, so the process matters. The standard toolkit:

| Instrument | Full name | Purpose | When |
|---|---|---|---|
| **RFI** | Request for Information | Market scan: who is out there, what can they do, capability screening | Early, when the market is unknown; shortlists candidates |
| **RFP** | Request for Proposal | Full solution proposal against a written specification: approach, solution, pricing, terms | The main event for complex/services/software buys |
| **RFQ** | Request for Quotation | Price quote against a fixed, well-defined spec | Commodities and well-specified goods |
| **Tender** | Competitive tender | Formal, often regulated competitive process (public/regulated entities); sealed bids, structured evaluation | High-value or regulated buys; banking often treats major IT procurement as a formal tender for governance/audit reasons |

The flow: **RFI** (qualify the market, cut to 3–6 candidates) → **RFP** (full proposals from the shortlist) → **evaluation** (Section 3.4) → **negotiation** (Section 4.10) → **award**. An **RFQ** is used instead of an RFP when the requirement is standard and only price differentiates. A **competitive tender** is the disciplined version of the whole flow, with documented evaluation and audit trail — mandatory in much of the public sector, and best practice in banking where procurement governance and audit expect evidence of competition. The honest caveat: *single-source* awards (no competition) happen — for extensions of existing platforms, sole vendors, or emergencies — and when they do, the justification must be documented because regulators and auditors will ask.

### 3.2 Selection criteria

What you evaluate candidates on. The standard criterion set:

| Criterion | What to check | Notes |
|---|---|---|
| **Capability** | Does the solution/service fit the requirement? Functional fit, architecture, delivery track record | Weighted highest for most buys |
| **Financial stability** | Financial statements, credit rating, going-concern health, ownership | Critical: a failed vendor is worse than no vendor |
| **References** | Verifiable customer references, especially in your industry and scale | Do reference calls, not just reference letters |
| **Security** | Security posture, certifications (ISO 27001, SOC 2), pen-test history, incident record | See Section 8; non-negotiable for data-touching vendors |
| **Compliance** | Regulatory standing, sanctions/AML screening, licences, data-protection compliance | See [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) |
| **Price / TCO** | Not just headline price — total cost of ownership (3.3) | The classic trap: cheapest bid, most expensive decade |
| **Strategic fit** | Roadmap alignment, partnership posture, willingness to commit to your terms (exit, audit, data) | The criteria that predict renewal happiness |

### 3.3 Total cost of ownership (TCO)

**TCO** is the lifecycle cost of a vendor engagement, not the sticker price: acquisition + implementation + operation + exit. A vendor whose licence is 10% cheaper but whose integration costs double, or whose exit clauses make leaving painful, is rarely cheaper. The TCO model:

```
TCO = acquisition (licences, hardware, setup fees)
    + implementation (integration, migration, customisation, training)
    + operation (annual fees, support, maintenance, infrastructure, management overhead)
    + exit (transition services, data extraction, termination fees, parallel run)
    − residual value (what you keep: IP, data, reusable integration)
```

The cost lens — unit economics, chargeback, cloud cost modelling — is covered in depth in [../technology/finops_guide.md](../technology/finops_guide.md); treat that guide as the toolkit for the *operation* line. The vendor-management-specific point: **TCO must be compared at the portfolio level, not per bid** — a "cheaper" vendor that adds two new integration points raises downstream costs, and a vendor with generous exit terms is worth a premium because it keeps the *option* of switching alive (Section 9.2).

### 3.4 Evaluation: weighted scoring, demonstrations, POC, site visits, references

Selection rigour comes from making the evaluation **structured and evidence-based**:

- **Weighted scoring matrix** — criteria (3.2) each with a weight; every shortlisted vendor scored by a cross-functional evaluation team (business, IT, security, procurement). Weights must be set *before* proposals arrive to avoid rationalising a favourite. Example (Section 11 has a full one): Capability 30%, TCO 20%, Security & Compliance 15%, Delivery/Implementation 15%, Financial stability 10%, Strategic fit 10%.
- **Demonstrations** — scripted demos against your scenarios (not the vendor's canned deck); score the demo, not the charisma.
- **Proof of concept (POC)** — a bounded trial of the actual product in your environment; essential for platforms, valuable for everything complex. Cost: POCs are expensive for everyone — bound them in time and scope.
- **Site visits** — see the vendor's delivery centre/offices, meet the actual delivery team (not the sales team); a key check for outsourcing and managed services.
- **Reference checks** — speak to 2–3 named references, with prepared questions on the *failure* modes you care about ("tell me about the worst incident in your first year").

The discipline that makes scoring honest: **evaluation criteria and weights locked before the RFP goes out; every score justified in writing; the evaluation team includes the people who will manage the vendor post-award** (the vendor manager's seat at selection pays for itself in ownership).

### 3.5 The selection decision

The decision sequence: **shortlist** (3–4 candidates from the RFI) → **deep evaluation** (3.4) → **negotiation with the top 1–2** (Section 4.10) → **award**. Governance: the award is approved at the appropriate delegation level (a bank's IT procurement committee for major deals), with the evaluation record attached. Two practices worth keeping:

- **Debrief the losers.** A short, honest feedback call protects the market's goodwill and your reputation as a buyer; it also surfaces information (did we get the decision right?).
- **Document the decision rationale.** In banking this is audit fodder — "why this vendor" must survive a regulatory or internal-audit challenge, especially for single-source awards and critical outsourcing.

### 3.6 Vendor due diligence (the KYV moment)

Before signing, the vendor is screened like a customer would be (hence **KYV — know your vendor**, the supplier-side cousin of KYC; informal terminology, but the practice is standard). Due diligence covers:

| Domain | Checks |
|---|---|
| **Financial** | Audited financials (2–3 years), credit rating, ownership structure, going-concern indicators, major litigation |
| **Legal** | Entity verification, litigation and regulatory actions, contract-capacity, IP ownership of the solution, sub-contracting practices |
| **Security** | Security questionnaire (see the shared-assessment practice in Section 8.7), certifications, breach history, incident-response capability — see the security guides in the repo for the control detail |
| **Compliance** | Sanctions screening (the vendor and its beneficial owners against OFAC/EU/UN lists), AML exposure, data-protection standing, regulatory licences relevant to the service |

The compliance lens — sanctions, AML, KYC — is covered in depth in [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md); due diligence for a *vendor* uses the same screening machinery aimed at a supplier rather than a customer. The result of due diligence is a **due diligence report** that feeds the risk assessment (Section 8.3) and is a condition precedent in the contract (the deal doesn't close until the checks clear).

---

## 4. Contracting

### 4.1 The contract stack: SOW + MSA + SLA

The contract is usually not one document but a **stack**:

| Layer | What it is | What it covers |
|---|---|---|
| **MSA** | Master Services Agreement — the *standing terms* | Definitions, warranties, liability, IP, confidentiality, data, termination, indemnities, governing law — applies to all work under it |
| **SOW** | Statement of Work — the *specific engagement* | Scope, deliverables, timeline, resources, price, acceptance criteria — one per project/service under the MSA |
| **SLA** | Service Level Agreement — the *service levels* | Measurable performance targets (availability, response, resolution), measurement, reporting, credits — often a schedule to the MSA/SOW |

Why stack them? The MSA is negotiated once and reused across many SOWs (a **framework / master vendor agreement** — see 4.11); the SOW captures what changes each engagement; the SLA captures what gets measured. Renegotiating the whole stack every engagement is the mistake the stack exists to prevent. A common variant in software: **license agreement** (perpetual or subscription) + **maintenance and support agreement** (the SLA-ish layer) + SOWs.

### 4.2 Pricing models

| Model | How it works | Risk sits with | When to use |
|---|---|---|---|
| **Fixed price** | Agreed price for agreed scope | Vendor (overrun risk) — buyer (spec risk) | Well-specified scope; classic SOW delivery |
| **Time & materials (T&M)** | Billed on hours/rates | Buyer (scope creep risk) | Evolving scope; support; staff augmentation |
| **Outcome-based** | Paid on achieved outcomes (e.g., per transaction, per resolved ticket) | Vendor (performance risk) | Services where output is measurable; aligns incentives but needs careful definition |

The discipline point: **choose the model by where the uncertainty sits**. Fixed price with an underspecified scope produces change-order warfare; T&M with no governance produces billable-hours inflation; outcome-based with fuzzy outcomes produces definitional disputes. Most serious engagements mix them — fixed-price delivery SOW, T&M support, outcome-based components where measurable.

### 4.3 Key commercial terms

Beyond price: **payment terms** (milestones vs monthly, payment windows, invoicing requirements, taxes, FX — a cross-border contract priced in USD but paid in SGD needs an FX clause), **expense treatment** (T&E pass-through, caps), **warranties** (workmanlike service, non-infringement, no malware), **change control** (how scope changes get priced and approved — the mechanism that keeps fixed-price deals sane), **acceptance criteria** (what "done" means before payment), and **audit rights** (the buyer's right to inspect the vendor's performance and books — see 8.5).

### 4.4 SLA design

SLAs are the contract's measurement engine. The classic targets (with banking-plausible numbers):

| SLA type | Definition | Typical target (banking) |
|---|---|---|
| **Availability** | % of time the service is up over a period | 99.9%+ monthly (core banking), 99.5% (non-critical) |
| **Response time** | Time to acknowledge an incident | 15 min (critical P1) to 4 business hours (P3) |
| **Resolution time** | Time to fix/restore | P1: 4–8 hours; P2: 1–2 business days; P3: 5–10 days |
| **Other** | Batch completion windows, transaction accuracy, report timeliness | Per service |

Design rules that separate good SLAs from decoration: **measure the customer experience, not the vendor's convenience** (availability measured at the user/business-service level, not the data-centre level); **define the measurement window and exclusions** (planned maintenance, force majeure — but cap the exclusions); **agree the reporting and evidence** (who measures, how, monthly SLA reports); **set targets that are demanding but achievable** (a target never breached is a target never tested — aim for breach rates that bite but don't destroy the relationship); and **link SLAs to credits** (4.6). The single most common SLA failure is measuring something that doesn't matter to the business — an SLA tree that monitors server uptime while the business cares about failed transactions.

### 4.5 Penalties and remedies

- **Service credits** — a contractual discount applied when an SLA is missed (e.g., 5% of monthly fees per 0.1% availability shortfall, capped at 20–30%). Credits are a *remedy*, not a revenue source — the intent is to compensate and incentivise, not to profit (see 6.6 for how to run them without poisoning the relationship).
- **Liquidated damages (LDs)** — pre-agreed damages for specific failures (e.g., a missed go-live date, a failed regulatory deadline), enforceable when they are a genuine pre-estimate of loss rather than a penalty.
- **Termination rights** — the ultimate remedy; for cause (breach, repeated SLA failure, insolvency) and for convenience (4.7).

The negotiation reality: vendors resist uncapped liability and open-ended LDs; buyers resist "best efforts" language. The middle ground is a **credit schedule with caps plus termination-for-persistent-failure** — the credit pays for the pain, and the termination right is the discipline behind it.

### 4.6 Termination and exit terms

Termination clauses decide how the relationship *ends* (Section 9 executes them):

- **Termination for cause** — vendor breach (SLA failure, data breach, insolvency, regulatory action). Buyer can walk with minimal notice and no penalty.
- **Termination for convenience** — either party (usually buyer) can exit without cause, with notice (90–180 days typical) and often a wind-down fee. The buyer's "for convenience" right is the single most valuable exit clause — without it, the vendor holds the relationship hostage.
- **Exit terms** — the obligations that trigger on termination: transition assistance (the vendor must help move the service for a defined period at defined rates), data return (all data in usable formats), IP return, deletion certification (8–9), and cooperation with the successor.

The discipline: **negotiate exit terms while you still have leverage** (before signature, when the vendor wants the deal). Post-signature, the vendor has no incentive to be generous. The classic negotiating line: "we're not planning to leave, but the terms under which we *could* leave are part of the price."

### 4.7 IP, confidentiality, and data

- **Intellectual property** — who owns what: the vendor's pre-existing IP stays the vendor's (licensed to you); *deliverables* created for you are either assigned to you or licensed (perpetual, irrevocable); and for software, **source code escrow** (the code held by a third party, released on vendor insolvency/failure) is the standard protection for critical proprietary platforms.
- **Confidentiality (NDA)** — mutual non-disclosure, defined term, carve-outs (regulatory disclosure); the NDA covers the *commercial* secret, the DPA covers the *personal data*.
- **Data** — the **Data Processing Agreement (DPA)** under the applicable data-protection law (PDPA in Singapore; GDPR if EU data is involved): purposes and limits of processing, sub-processor authorisation and flow-down, data residency, breach notification (timing — e.g., 72 hours under GDPR), return/deletion on termination. Data clauses are where banking vendor contracts get most of their length, and rightly so — see [../technology/data_governance_guide.md](../technology/data_governance_guide.md) for the governance side.

### 4.8 Compliance, liability, and indemnities

- **Regulatory clauses** — the vendor must comply with applicable law and *your regulatory obligations* (for banks: the outsourcing notice requirements, audit access, information to regulators, business-continuity cooperation — see Section 8.8). These clauses exist so that *your* regulator's demands flow down to *your* vendor.
- **Limitation of liability (LoL)** — the cap on the vendor's liability (typically annual fees, or a multiple for data breaches); both sides' liability for gross negligence/wilful misconduct, IP infringement, and breach of confidentiality/data clauses usually **carve out** of the cap.
- **Indemnities** — the vendor indemnifies you against third-party claims arising from its breach (IP infringement is the classic: the vendor defends you if the software infringes someone's patent).
- **Insurance** — professional indemnity, cyber, employer's liability; minimum coverage levels as a condition of the contract, with certificates on demand.

The discipline point: **liability terms are a risk-allocation decision, not a legal formality** — the vendor manager should know the cap on the contract they manage, because it sets the ceiling on what a failure can recover (and therefore how much risk the relationship carries — feed it into the risk register, Section 8.3).

### 4.9 Contract negotiation

Negotiation is a skill in its own right — the full toolkit (preparation, BATNA, positions vs interests, concession strategies, negotiation styles) lives in [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md). The vendor-management-specific framing:

- **Leverage comes from alternatives.** The single strongest position is a credible second option (Section 2.5) or a working incumbent ("we can stay with the current vendor"). Never enter negotiation without knowing your BATNA — and never *reveal* it.
- **Win-win is not softness.** The best deals align interests: the vendor wants a multi-year revenue commitment; you want investment in your account and favourable exit terms. Trade commitments for commitments ("we'll sign three years if exit terms are clean and credits are real").
- **Negotiate the whole stack, not the price.** Headline price is where vendors hold firm; the *terms* (exit, credits, audit rights, data, LoL carve-outs) are where concessions are cheapest for them and most valuable for you.
- **Leave something on the table.** A deal that humiliates the vendor's negotiator comes back as a stingy account team, aggressive change orders, and slow support. The relationship starts at signature — see Section 7.

### 4.10 Master vendor agreements and framework agreements

The **master vendor agreement** (framework agreement) is the enterprise version of the stack: one MSA with a strategic vendor (a cloud provider, a core platform vendor, a global consultancy) plus standard terms, security schedules, and DPAs pre-negotiated once, under which all business units place SOWs/orders. Benefits: consistency (one set of terms instead of 40 local variants), leverage (enterprise-wide spend negotiated once), speed (new engagements skip the legal boilerplate). Risks: business units ordering outside the framework (maverick buying), and framework terms that fit no one perfectly. The VMO (Section 1.7) typically owns the framework catalogue and the "must use the framework" policy.

---

## 5. Onboarding and Implementation

### 5.1 The onboarding process

**Vendor onboarding** is the bridge from signature to go-live — the period when the vendor goes from "contractually engaged" to "operationally live." Its purpose is to make the vendor *safe and ready*, and it is the stage where security and compliance requirements are actually enforced (a contract clause only matters once it's operationalised). A representative onboarding flow:

```
Signature → onboarding kickoff → security & compliance checks → access provisioning
→ technical integration → readiness assessment → go-live (cutover) → hypercare
```

The onboarding checklist (extend per vendor tier): contracts and POs filed; vendor entity and bank details verified (KYV, 3.6); security evidence collected and accepted (SOC 2/ISO 27001 reports, pen-test summaries); sanctions/AML re-screen where required; data-protection sign-off (DPA executed, sub-processor list); access requests raised; technical integration scheduled; support contacts and escalation matrix exchanged; SLA reporting configured; insurance certificates collected.

### 5.2 Security assessment and compliance checks

Before any systems access or data flows, the security assessment from selection (3.6) is *completed to evidence*:

- **Evidence-based sign-off** — SOC 2 Type II report (or an agreed alternative), ISO 27001 certificate, penetration-test summary (and the vendor's remediation plan), background on incident history. See Section 8.7 for the standards.
- **Architecture review** — where does the vendor's service sit in your architecture (cloud, on-prem, hybrid); what data flows where; encryption in transit and at rest.
- **Compliance checks** — regulatory standing, data-residency requirements, the sanctions/AML re-screen. For banking, onboarding a vendor that touches customer data is itself a governed event (material outsourcing notification — Section 8.8).

The discipline: **no security sign-off, no access** — the onboarding gate is where InfoSec's leverage is absolute, and vendor managers who protect that gate protect the whole portfolio.

### 5.3 Data access and least privilege

Access provisioning is the operational core of onboarding:

- **Least privilege** — the vendor gets the minimum access required for the job: scoped roles, environment limits (test vs prod), time-boxed access where possible; no standing admin accounts.
- **Identity and segregation** — vendor staff get individual identities (no shared logins), through your identity management (SSO/IDAM) where feasible; separation of duties between vendor staff and your staff for sensitive operations.
- **Vetting** — background checks and confidentiality acknowledgements for vendor staff who will handle sensitive data (per contract and policy).
- **Monitoring** — vendor access logged and reviewed; anomalous activity flagged (the SolarWinds lesson, Section 8.2, is that *trusted* access is the most dangerous access).

### 5.4 Technical integration

The integration work itself — connectivity (network, VPN/private links), authentication (SSO, certificates), APIs and file transfer, environments (dev/test/prod), monitoring and alerting integration, and the operational handover (runbooks, support rota, escalation paths, incident-management tooling). The project-management side of this (planning, resourcing, dependencies, the PMBOK procurement-management process) is covered in [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md); the vendor-management side is making sure the *operational* relationship is live: the vendor's support desk is in your escalation matrix, their monitoring sees your environment, and your teams know who to call at 3 a.m.

### 5.5 Go-live: cutover and hypercare

- **Cutover** — the switch itself: the sequenced plan (freeze, migrate, verify, switch, rollback), executed in a window, with a **rollback plan** that is tested in rehearsal, not invented on the night.
- **Hypercare** — the post-go-live support period (typically 2–4 weeks, sometimes longer for core platforms): heightened monitoring, vendor staff on standby, a daily defect/issue triage, and a defined exit from hypercare once stability criteria are met (e.g., X consecutive days without a P1/P2). Hypercare is *paid-for readiness* — the discipline is defining its exit criteria in advance, so "we're still in hypercare" doesn't become a permanent state.
- **Readiness assessment** — the formal pre-go-live check that the vendor is ready: staffing levels confirmed, runbooks delivered, SLAs measurable, BCP tested (Section 8.8), training done. Sign-off on readiness is the gate to cutover — the vendor manager's "go/no-go."

---

## 6. Performance Management

### 6.1 Performance management: from contract to behaviour

**Vendor performance management** is the machinery that turns contractual promises (SLAs, SOW commitments) into measured, managed, improving behaviour. The cycle: **define** (KPIs and SLAs, Section 4.4) → **measure** (monitoring and reporting) → **review** (scorecards and QBRs) → **act** (improvement plans, credits, renewal decisions). Without the cycle, a contract is a wish; with it, the contract becomes a *management instrument*. The performance manager's question every month is: *are we getting what we paid for, and is it improving?*

### 6.2 KPIs and SLA monitoring

- **KPIs** — the broader performance indicators (beyond SLAs): delivery timeliness, defect rates, change success rate, customer-satisfaction scores, cost trends, compliance events. KPIs tell you about the *health* of the relationship; SLAs tell you about *contractual* performance. Both feed the scorecard.
- **SLA monitoring** — automated where possible (availability and response metrics from monitoring tools), reported monthly per the contract, with breaches tracked and credited (6.6). The discipline: *verify, don't trust* — vendor-reported SLAs are verified against your own monitoring where feasible, because the vendor's measurement window is their own definition.

### 6.3 The vendor scorecard

The **vendor scorecard** is the single consolidated view of a vendor's performance — the QBR's centrepiece and the renewal decision's evidence base. The common design is a **balanced-scorecard-style** set of dimensions (borrowing the balanced-scorecard idea of looking at multiple perspectives, adapted to suppliers):

| Dimension | What it measures | Example measures |
|---|---|---|
| **Quality** | Correctness of delivery | Defect rate, error rate, rework, acceptance rate |
| **Delivery** | Timeliness and service levels | SLA compliance %, on-time delivery, response/resolution times |
| **Cost / Value** | Commercial performance | Price vs benchmark, cost trends, savings delivered, invoice accuracy |
| **Relationship** | How it feels to work with them | Responsiveness, communication quality, stakeholder satisfaction |
| **Risk / Compliance** | Exposure and adherence | Open risk findings, audit results, security incidents, compliance events |

Each dimension scores 1–5 (or 0–100), weighted per vendor tier (a tier 1 vendor's scorecard weights Risk higher than a tier 3's), rolled into a **traffic-light rating** (green ≥ 4.0, amber 3.0–3.9, red < 3.0 — thresholds defined per programme). The scorecard is *scored by the buyer* (the vendor manager and stakeholders), not self-reported by the vendor — the vendor sees it, challenges it, and acts on it, but doesn't write it. Section 11 has a full worked example.

### 6.4 Vendor-specific measures (VSMs)

**Vendor-specific measures (VSMs)** are the performance measures that are *unique to a particular vendor engagement* — the KPIs that only make sense for this contract, layered on top of the standard scorecard dimensions. Example: for a core banking vendor, a VSM might be "regulatory-change release delivered within the mandated window" or "patch deployment within X days of vendor release"; for a cloud provider, "region expansion within committed quarter." The concept is sound and widely practised under other names ("contract-specific KPIs", "engagement metrics") — but it must be flagged honestly: **"VSM" is not a standardised industry term.** In ITSM/DevOps circles **VSM overwhelmingly means *Value Stream Management*** (the Lean practice of mapping and improving value streams), and no major standards body (ISO, ITIL, WorldCC) canonises "vendor-specific measures" as a formal acronym. Use the term internally if your organisation does, but expect the ambiguity.

### 6.5 Performance reviews: the QBR

The **Quarterly Business Review (QBR)** is the flagship governance forum for tier 1–2 vendors (the full agenda and mechanics are in Section 7.2; here the *performance* content):

- **Performance review** — scorecard walk-through, SLA compliance, credits incurred, KPI trends, open issues and their ageing;
- **Delivery status** — projects, releases, changes in flight;
- **Risk review** — open risk-register items, security posture changes, audit findings, incident post-mortems;
- **Roadmap alignment** — vendor roadmap vs your strategy, upcoming changes that affect you;
- **Improvement commitments** — agreed actions from last quarter, tracked to closure;
- **Commercials** — spend vs forecast, upcoming renewals, change orders.

The QBR's discipline: **the scorecard is pre-circulated** (the meeting discusses the data, not collects it), **actions are tracked with owners and dates** (a QBR without an action log is a social call), and **seniority matches tier** (a tier 1 vendor's QBR has executives on both sides — the vendor sends people who can commit).

### 6.6 Service credits in practice

Credits (4.5) run well when they run by formula: SLA breach reported → credit calculated per the schedule → applied to the next invoice → tracked in a credits register. The relationship-smart rules: **apply credits automatically and without ceremony** (the vendor who fights every credit is signalling; the buyer who weaponises credits is destroying the relationship); **cap credits** (the schedule's cap keeps them a remedy, not a penalty); and **treat persistent breach as a management problem, not a billing problem** — when the same SLA fails every quarter, the question is *why* (root cause, improvement plan) not *how much* to invoice. Credits are the *scoreboard*; the improvement plan is the *game*.

### 6.7 Continuous improvement and vendor innovation

- **Continuous improvement** — formal improvement mechanisms: corrective action plans (CAPA) for systemic failures; agreed improvement targets in the scorecard (raise SLA compliance from 96% to 98% over two quarters); value-creation registers ("what has this vendor improved this year, and what did it save?").
- **Vendor innovation reviews** — the deliberate *future* conversation: what's new in the vendor's roadmap that could help you (a new module, a cloud capability, an AI feature), what's in your strategy the vendor could enable, and what co-development looks like. Innovation reviews are the strategic-vendor practice that separates *vendor management* (protecting the deal) from *partnership* (growing the deal) — see Section 7.4.

---

## 7. Relationship Management

### 7.1 The relationship model: sponsorship, governance, "one team"

Vendor performance is delivered by *people*, and people perform for relationships they value. The relationship model for strategic vendors rests on three structures:

- **Executive sponsorship** — a named executive sponsor on *each* side: your side (an executive who owns the relationship's success and can unblock at the top) and theirs (an account executive with authority). The sponsors meet periodically and are the escalation ceiling (7.3).
- **Governance boards** — the **Joint Steering Committee (JSC)** is the top governance forum: co-chaired by the sponsors, meeting quarterly or semi-annually, with authority over priorities, investment, disputes, and renewal. Below it: the QBR (performance, Section 6.5), the monthly ops review, and the daily/weekly operational touchpoints. The JSC is where *strategy* is governed; the QBR is where *performance* is governed — conflating them is a common maturity gap.
- **"One team"** — the partnership posture: joint planning, shared goals, vendor staff treated as team members (while *never* forgetting they are vendor staff — access, confidentiality, and conflict-of-interest boundaries stay intact). The "one team" model is what makes a vendor's best people want to work on your account.

### 7.2 Communication cadence

| Forum | Cadence | Attendees | Content |
|---|---|---|---|
| **Daily ops / standup** | Daily | Delivery leads both sides | Incidents, batch status, blockers |
| **Weekly ops review** | Weekly | Vendor manager + vendor delivery lead | SLA health, tickets, changes, risks this week |
| **Monthly review** | Monthly | Vendor manager + vendor account team | Scorecard inputs, credits, projects, financials |
| **QBR** | Quarterly | Stakeholders + vendor leadership | Scorecard, roadmap, risk, improvement (6.5) |
| **JSC / executive** | Quarterly–semi-annual | Exec sponsors | Strategy, investment, disputes, renewal |
| **Annual strategic review** | Annual | Senior leadership | Multi-year plan, innovation, relationship health |

The cadence is a *tiered* contract: tier 1 vendors run the full ladder; tier 2 runs monthly + QBR-lite; tier 3 is exception-based. The discipline is **consistency, not volume** — a skipped QBR tells the vendor exactly how unimportant they are.

### 7.3 Escalation: paths and culture

- **Escalation paths** — a written escalation matrix agreed at onboarding: for each issue type (incident, SLA breach, dispute, commercial, relationship), who escalates to whom, at what threshold, with what response-time commitment. The matrix is published to both sides — no one should be *discovering* whom to escalate to at 3 a.m. during a P1.
- **"Escalation is good" culture** — the counterintuitive norm that escalation is a *management tool*, not a complaint: escalate early, escalate with data, escalate to the person who can act. Relationships that punish escalation get silent problems instead — and silent problems become surprises (and surprises become incidents, and incidents become audit findings).
- **The ladder** — vendor manager → delivery director → account executive → exec sponsors → (rarely) contract remedies (credits, termination). The ladder should be *climbed* deliberately, not leapt — but with a clock: issues that need the top should reach the top fast.

### 7.4 Strategic vendors in practice: joint roadmaps and innovation

For tier 1 vendors, relationship management is *partnership management*:

- **Joint roadmaps** — an annual planning exercise where the vendor's product/technology roadmap and your strategy are put side by side: where they align (fund, accelerate), where they diverge (influence, or plan around), and what you will *ask* the vendor to build (with the commercial conversation that follows — roadmap influence is a legitimate negotiation currency).
- **Innovation partnerships** — co-development, early-adopter programmes, joint proof-of-concepts, advisory councils. The bank that co-develops with its core vendor gets features first and shapes the platform; the bank that waits gets the platform *as shipped*.
- **Co-investment** — the deepest form: shared funding of capability development, joint ventures on new products. Rare and powerful — and only appropriate where trust, governance, and exit terms are mature.

### 7.5 Vendor advocacy

The positive side of the relationship ledger: a vendor that performs becomes a **reference** (you recommend them, speak at their events), a **preferred vendor** (the framework-agreement status of Section 4.10), and a **partner in your own credibility** (a strong vendor's benchmark data and expertise make your internal business cases stronger). Advocacy is also a *currency*: the promise of reference status and preferred-vendor treatment is genuinely valuable to vendors and costs you little — it belongs in the negotiation and in the renewal conversation.

### 7.6 Conflict and disputes

Disputes will happen — credits, scope, data, IP. The discipline:

- **Resolve at the lowest level first** — the monthly review resolves 90% of commercial niggles if there is a working relationship (7.2).
- **Dispute resolution clauses** — the contract's escalation: negotiation → senior executives → mediation → (rarely) arbitration/litigation. The clause's purpose is to make the *process* predictable so neither side is tempted to posture.
- **Escalate to the exec** — when a dispute is stuck at working level, it goes to the sponsors (7.1), who resolve it as *business* people protecting a *business* relationship, not as lawyers defending positions. A dispute that reaches the JSC should leave it with a decision and a face-saving path for both sides.
- **The honest note** — some disputes are signals of a broken relationship rather than a broken clause; if every quarter produces a new fight, the fix is relational (Section 7) or structural (renewal, replacement — Section 9), not another credit negotiation.

---
## 8. Vendor Risk Management (VRM/TPRM)

### 8.1 VRM vs TPRM

**Vendor risk management (VRM)** is the process of identifying, assessing, monitoring, and mitigating the risks posed by a *specific vendor or group of vendors*. **Third-party risk management (TPRM)** is the broader, enterprise-wide discipline covering *all* third parties (vendors, suppliers, service providers, business partners, subcontractors) under one framework, policy, and governance. The relationship: **VRM is the vendor-shaped slice of TPRM** — a bank's TPRM programme covers vendors, but also clearing houses, correspondents, outsourcers, and cloud providers under a single risk taxonomy and reporting line. In practice the terms are used almost interchangeably for the *process*; the distinction matters for *ownership* — TPRM is an enterprise risk programme (usually owned by Operational Risk/Enterprise Risk with a central TPRM office), VRM is often run by the vendor management function. The core process is the same, and is the subject of the rest of this section:

```
Assess (tier, inherent risk, due diligence) → Mitigate (contract, controls, plan)
→ Monitor (continuous) → Review (periodic re-assessment) → Report (risk register, committees)
```

### 8.2 Risk types

| Risk type | What it is | Banking example |
|---|---|---|
| **Operational** | Failure of the vendor's delivery/service | A core-platform outage stopping account opening; batch failure breaking end-of-day |
| **Financial** | Vendor insolvency or financial distress | The vendor enters administration; support disappears mid-contract |
| **Cybersecurity** | Data breach, compromise, or **supply-chain attack** | The vendor is breached and customer data exfiltrated — or the vendor's software is weaponised to reach *you* |
| **Regulatory** | Vendor action/inaction causes regulatory breach | The vendor fails a data-residency requirement; a regulator finds the outsourcing unmanaged (MAS Notice 658, 8.8) |
| **Concentration** | Over-dependence on one vendor (2.3) | Single cloud provider or single core vendor; the vendor's failure becomes *your* systemic event |
| **Reputational** | Association with the vendor damages your standing | Vendor's conduct scandal, data breach, or sanctions exposure reflects on the bank |

The **supply-chain attack** deserves the flagship position it now holds. The **SolarWinds** attack (disclosed December 2020) is the canonical case: attackers compromised SolarWinds' build environment and injected a backdoor ("SUNBURST") into legitimate Orion software updates, which were then trusted and installed by roughly 18,000 organisations, including US government agencies and major companies — a *trusted vendor* became the attack vector. The lessons vendor managers internalised: (1) a vendor's software is an extension of your attack surface; (2) software-supply-chain integrity (build verification, signature checks, SBOMs — software bills of materials) is a vendor-requirement, not a nice-to-have; (3) vendor *access* to your environment must be monitored as closely as your own staff's (Section 5.3). The standards that grew out of this era — ISO 27036, NIST SP 800-161, and the SBOM practice — are in Section 8.7.

### 8.3 Risk assessment: tiering, inherent vs residual, the risk register

- **Tier-based assessment** — assessment depth scales with risk tier (Section 2.2). Tier 1/critical vendors get the full assessment (deep due diligence, security review, on-site or remote audit, financial review, BCP review); tier 3 vendors get a standard questionnaire. In banking TPRM the tiers are usually **critical / high / medium / low** and are heavily driven by *materiality* (regulatory definition — see 8.8), data sensitivity, and concentration.
- **Inherent vs residual risk** — the standard risk model: *inherent* risk is the risk before mitigation (what the vendor *could* do to you), *residual* risk is what remains after controls (contract terms, monitoring, BCP, insurance). The assessment scores both; the gap between them is the value of your mitigations. An unmanageable residual risk on a critical service is a *selection or exit* trigger, not an acceptance.
- **The vendor risk register** — the living inventory: per vendor, the assessed risks (type, inherent score, controls, residual score, owner, action plan, review date, status). The register is the TPRM report's raw material — risk committees, regulators, and internal audit all read it. Section 11 shows an excerpt.

### 8.4 Continuous monitoring

Risk is not a point-in-time event; the monitoring layer watches for change:

- **Financial health** — credit-rating changes, financial news, ownership changes (an acquisition changes everything — the vendor's strategy, contracts, and stability), payment-behaviour signals, distress indicators.
- **Security posture** — certification expiry (SOC 2, ISO 27001), breach disclosures, vulnerability disclosures affecting the vendor's products, security-assessment results over time.
- **Dark-web monitoring** — monitoring for leaked credentials or chatter referencing your organisation/vendor. Honest flag: this is a *signal*, widely sold by TPRM platforms (OneTrust, Recorded Future-style threat intel), but its value is debated — it rarely prevents an incident and mostly confirms what breach notifications later reveal; treat it as one input, not a control.
- **News and regulatory signals** — sanctions-list changes, litigation, regulatory actions against the vendor.
- **Event-driven triggers** — any material change (breach, insolvency filing, major incident, change of control) triggers a **re-assessment** and, for critical vendors, an immediate escalation to risk committees.

### 8.5 Vendor audits and the right to audit

The **right-to-audit** clause is the buyer's quality assurance on the vendor's claims: the right to review the vendor's performance, controls, and (for critical services) premises, at agreed notice. In practice:

- **Evidence-based assurance** — the first line: SOC 2 Type II reports, ISO 27001 certificates, pen-test summaries (Section 5.2). Most assurance happens this way — cheaper for both sides than a full audit.
- **On-site audits** — for critical/regulated services, periodic on-site (or since COVID, remote-conducted) audits of the vendor's operations, controls, and subcontractors, per the contract's audit schedule.
- **Regulatory-driven audits** — in banking, the *regulator's* right to inspect outsourced arrangements flows down to the vendor contract (the vendor must cooperate with the bank's regulators — MAS/DORA both require this); see 8.8.

The discipline: **an audit right that is never exercised atrophies** — and a vendor that resists a *reasonable* audit is a risk signal in itself. Audits feed the scorecard (Risk dimension, Section 6.3) and the risk register.

### 8.6 Fourth-party risk (sub-vendors)

The vendor's own vendors are **fourth parties** (sub-vendors, subcontractors) — and they carry your risk too. The classic failure: a vendor subcontracts a critical component to a party you never assessed, and that party's failure becomes your incident. Controls:

- **Flow-down clauses** — the contract requires the vendor to impose equivalent obligations (confidentiality, data protection, security, audit) on its sub-contractors;
- **Sub-vendor registers** — the vendor discloses material sub-contractors; the buyer reviews the list (and screens the significant ones — KYV applied one level down);
- **Consent rights** — material changes to sub-contracting (especially data processors) require buyer consent or notification (the DPA's sub-processor mechanism);
- **Fourth-party visibility in TPRM** — the TPRM register tracks material fourth parties so that concentration and single points of failure are visible across the whole chain, not just the first hop.

### 8.7 Standards and frameworks

| Standard | What it is | Use in vendor management |
|---|---|---|
| **ISO/IEC 27036** | *Cybersecurity — Supplier relationships* (ISO series): Part 1 Overview and concepts (2021), Part 2 Requirements, Part 3 Guidelines for hardware, software, and services supply chain security (2023), Part 4 Cloud services | The dedicated supplier-security standard: governs security in supplier relationships across the acquisition lifecycle; the framework for supplier-security clauses and assessments |
| **NIST SP 800-161 Rev. 1** | *Cybersecurity Supply Chain Risk Management (C-SCRM) Practices for Systems and Organizations* (May 2022; Update 1, Nov 2024) | The US government's C-SCRM practice guide: multi-level supply-chain risk assessment, SBOMs, counterfeit/component integrity — increasingly the reference for critical-infrastructure and regulated buyers |
| **ISO/IEC 27001 / SOC 2** | Information-security management certification / attestation of controls (AICPA Trust Services Criteria) | The *evidence* layer: what you ask vendors to produce (Section 5.2) to demonstrate their security posture |
| **Shared assessments (SIG / SIG-Lite)** | Standardised Information Gathering questionnaires (industry consortium) | Standardised security questionnaires to avoid every buyer re-inventing the vendor questionnaire; widely used in banking TPRM |

Honest note: the *certification* layer (27001, SOC 2) is evidence of a control *programme*, not proof of a *secure* vendor — certifications expire, scopes change, and the SolarWinds case involved a vendor with mature security. Treat certificates as the floor, not the ceiling.

### 8.8 Regulatory TPRM in banking

This is where vendor management meets the regulator — and where this guide's banking context (Crédit Agricole CIB, Singapore) bites. The full regulatory landscape (MAS, DORA, BCP, sanctions/AML) is covered in [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md); the vendor-management-relevant core, verified as of August 2026:

- **MAS Notice 658** — *Management of Outsourced Relevant Services for Banks*: issued 11 December 2023, **effective 11 December 2024**, applicable to banks in Singapore (MAS Notice 1121 covers merchant banks). It replaced the previous banks' outsourcing notice (MAS Notice 634) and operationalises the 2023 Guidelines on Outsourcing: risk-based governance of *outsourced relevant services* (with focus on **material** services), notification to MAS, customer-information handling, audit/access rights, BCP, and concentration management. Note: earlier references (including the first edition of this repo's banking guide, which cites "MAS Notice 653") predate or mislabel this regime — the current statutory outsourcing notice for banks is **MAS Notice 658**; verify notice numbers against MAS's current list before citing them in formal documents.
- **MAS Technology Risk Management (TRM)** — the statutory TRM notice/guidelines (in force since August 2023 for banks) cover technology resilience, cyber hygiene, cloud outsourcing, and incident notification; the outsourcing and TRM regimes are designed to be read together.
- **MAS TPRM evolution** — on 6 March 2026 MAS published Consultation Paper **P004-2026** proposing new **Guidelines on Third-Party Risk Management (TPRMG)** (consultation closed 20 April 2026; final guidelines pending as of mid-2026, with a transition period expected). Direction of travel: TPRM broadened beyond "outsourcing" to the full third-party lifecycle — selection, contracting, monitoring, exit — with board accountability and concentration risk front and centre.
- **DORA (EU)** — the *Digital Operational Resilience Act* (Regulation (EU) 2022/2554), which **applied from 17 January 2025** (in force 16 January 2023): mandatory ICT risk management, incident reporting, digital operational resilience testing (TLPT), and a **dedicated ICT third-party risk regime** — the register of information (a full inventory of ICT third-party arrangements at entity and consolidated level), contractual requirements (full-scope clauses: performance, access, exit, sub-contracting), and the **oversight framework for critical ICT third-party service providers (CICTTPs)**, with the European Supervisory Authorities able to oversee the providers themselves (including sanctions). For a global bank, DORA governs the EU-entity relationships even when the Singapore entity runs MAS rules — the *most demanding* applicable regime tends to set the standard for the group.
- **Critical outsourcing and BCP** — regulators require that *material/critical* outsourced services have: notification and approval touchpoints, **business continuity plans (BCP)** aligned with the vendor's own (tested jointly, recovery-time objectives defined), exit plans, and concentration analysis. BCP is treated in depth in [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md); the vendor-management job is to make BCP a *contractual, tested, living* thing rather than a slide deck (see the worked example, Section 11).

The synthesis: **regulatory TPRM is vendor management with teeth.** A bank's vendor manager who runs scorecards, risk registers, and BCP tests is simultaneously running the compliance programme — the same artifacts serve both masters.

---

## 9. Offboarding and Exit

### 9.1 The offboarding process

**Vendor offboarding** is the managed end of the relationship — the disciplined reverse of onboarding. Its goal: *leave with everything you own, nothing you don't, and no lingering exposure.* The process:

```
Exit decision → transition plan → data return & deletion → IP/asset return
→ service wind-down → access revocation → contractual close-out → post-exit review
```

| Step | What it covers |
|---|---|
| **Transition plan** | Timeline, successor/insourcing handover, parallel-run period, cutover, responsibilities (contractual transition assistance, Section 4.6) |
| **Data return** | Full extraction of your data in agreed, usable formats; verification of completeness |
| **Data deletion** | Deletion of your data from the vendor's systems (and sub-processors'), with a **deletion certificate**; retain minimum required by law/regulation (documented) |
| **IP return** | Return/destruction of your confidential material; confirmation of licence wind-down |
| **Service wind-down** | Run-off of services per contract (9.3), final invoices, final SLA reporting |
| **Access revocation** | Revoke all vendor access (systems, buildings, data) on exit day — the mirror of Section 5.3 |
| **Contractual close-out** | Final accounts, release of guarantees, confirmation of surviving clauses (confidentiality, audit, data) |
| **Post-exit review** | Lessons learned: what worked, what the exit cost, what the contract should have said (feeds Section 4) |

### 9.2 Exit management: lock-in, exit costs, data portability

The exit conversation starts at signature (Section 4.6) because that is when leverage exists:

- **Lock-in** — the state where exit is so expensive or difficult that the buyer is effectively captive: proprietary formats, missing data-export tooling, transition services priced punitively, termination-for-convenience absent. Lock-in is a *commercial* phenomenon (contractual) and a *technical* one (integration depth, data formats, customisation). The vendor manager's job is to price lock-in honestly at selection (TCO's exit line, Section 3.3) and to keep the exit terms current at every renewal.
- **Exit costs** — termination fees, transition-assistance fees, data-extraction effort, parallel-run costs, the successor's onboarding costs. A well-negotiated contract caps these; a bad contract lets the vendor monetise your departure.
- **Data portability** — the right and mechanism to get *your* data back in usable form (open formats where possible, documented APIs, standard export functions). Portability is the technical half of anti-lock-in; regulators increasingly expect it (DORA's exit-strategy requirement includes data portability for critical ICT services; MAS guidelines expect orderly exit). The cloud variant — portability and exit from a cloud provider — is covered in [../technology/cloud_providers_guide.md](../technology/cloud_providers_guide.md).

### 9.3 Vendor succession and run-off

- **Succession** — the successor path: a new vendor, an insourced team (2.6), or a merger/consolidation of the service. Succession planning for tier 1 vendors is a *standing* activity (a documented exit plan kept current, a second source qualified where feasible — Section 2.5), not a fire drill triggered by a vendor's collapse.
- **Run-off** — the period where the exiting vendor continues limited service to keep the business alive during transition: contracted run-off period (3–12 months typical), agreed run-off scope and rates, and the *knowledge transfer* that makes it effective (documentation, walkthroughs, shadowing — the vendor's people teaching your people or the successor's people).
- **Parallel run** — where continuity demands, both vendors run the service simultaneously for a period, with cutover when stability criteria are met (the mirror of go-live hypercare, Section 5.5).

### 9.4 The offboarding checklist

The checklist is the operational memory of the exit — every item verified, not assumed (a consolidated version of 9.1):

- [ ] Exit notice served per contract; exit terms invoked (transition assistance, run-off)
- [ ] Transition plan agreed with timeline, owners, and cutover criteria
- [ ] Data extracted in agreed formats and verified complete
- [ ] Data deleted from vendor and sub-processor systems; deletion certificates obtained
- [ ] Confidential material returned or destroyed; IP/licence wind-down confirmed
- [ ] Access revoked (systems, identities, physical) and verified
- [ ] Run-off scope/rates/period agreed and documented
- [ ] Knowledge transfer completed (docs, runbooks, walkthroughs)
- [ ] Final invoices reconciled; credits/claims settled; guarantees released
- [ ] Surviving obligations documented (confidentiality, audit, data retention, regulatory)
- [ ] Risk register updated; vendor records archived; post-exit review scheduled

### 9.5 The post-exit review

After the dust settles, the review: what did the exit cost versus the TCO model? What did the contract get right and wrong (exit clauses, data formats, transition terms)? What did the *relationship* teach (were early warning signs in the scorecard visible)? The outputs feed the next selection (Section 3) and the next contract (Section 4) — exits are the most expensive lessons a vendor programme ever buys, and the discipline is to *keep* the lesson.

---

## 10. Vendor Management in the Enterprise (Bank/IT)

### 10.1 The IT vendor landscape

The enterprise IT vendor portfolio is a zoo of relationship types, each with its own management flavour:

| Vendor type | Examples | Management flavour |
|---|---|---|
| **Software vendors (ISVs)** | Core banking platforms, market-data vendors, SaaS applications | Licence/subscription governance, roadmap alignment, version/upgrade discipline, escrow |
| **Cloud providers (CSPs)** | AWS, Azure, GCP | Shared-responsibility management, consumption/cost (see [../technology/finops_guide.md](../technology/finops_guide.md)), concentration, exit |
| **Hardware vendors** | Servers, network, end-user devices | Lifecycle/replacement planning, warranty/support, supply-chain risk |
| **IT outsourcing (ITO)** | Application development/maintenance, infrastructure outsourcing | Large contract governance: SLAs, T&M discipline, knowledge transfer, transition risk |
| **Business process outsourcing (BPO)** | Back-office operations, contact centres, KYC operations | Operational performance, data access, regulatory compliance, staff vetting |
| **Managed service providers (MSPs)** | Network ops, security operations (SOC), desktop support | Service-level discipline, escalation paths, tooling access |
| **Professional services** | Consultancies, system integrators | SOW governance, rate cards, IP in deliverables, bench management |

### 10.2 Software vendors and core banking

The most important vendor relationships in a bank are the **core banking platform vendors** — Temenos, Oracle (Flexcube), FIS, Finastra, SAP, Thought Machine (modern cloud cores), and regional players. The specifics of these platforms are covered in [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) and the per-vendor guides in the repo (e.g., [../banking/temenos_guide.md](../banking/temenos_guide.md)); the *vendor-management* specifics of core banking relationships:

- **Multi-decade lock-in** — core platforms are the extreme case of dependency (Section 2.3): switching costs run into the hundreds of millions and years of effort, so *exit terms, escrow, and roadmap influence* (Section 4.7, 7.4) matter more here than anywhere.
- **Upgrade discipline** — every core vendor's release cycle imposes upgrade obligations; the bank's version position determines support risk and access to new functionality. Vendor management runs the upgrade *relationship* (roadmap, certification, regression-testing collaboration) even when the project side is run by IT.
- **Regulatory dependency** — the vendor's platform must absorb regulatory change (payments rails, reporting, AML, ISO 20022); the bank's regulatory standing depends on the vendor's delivery. The "regulatory-change VSM" (Section 6.4) is real here: *the vendor's release calendar is the bank's compliance calendar.*
- **Bank-vendor power dynamics** — a global bank with many installations has roadmap leverage; a small bank on the same platform is a passenger. Honest positioning (Section 3.2, strategic fit) matters at selection: *what influence will you actually have?*

### 10.3 Cloud vendor management

Cloud is vendor management with its own physics: **shared responsibility** (the provider secures the cloud; you secure what's in it — the boundary is contractual *and* technical), **consumption economics** (cost is variable and needs the finops discipline — [../technology/finops_guide.md](../technology/finops_guide.md)), **concentration** (regulators watch cloud concentration explicitly), and **exit** (data egress, architecture portability — the cloud-exit discussion in [../technology/cloud_providers_guide.md](../technology/cloud_providers_guide.md)). The cloud-specific vendor artifacts: service credit programmes (cloud SLAs pay credits with caps), region/availability-zone commitments, the provider's security compliance reports (the shared-responsibility evidence), and the *cloud exit plan* as a standing tier-1 artifact. Cloud also introduced the **hyperscaler-as-ecosystem** dynamic: the CSP is simultaneously your vendor, your software vendors' platform, and your competitors' platform — governance must account for that entanglement (e.g., a SaaS vendor running on the same CSP you use: who is accountable for what, and where is the data actually).

### 10.4 The VMO in the enterprise and vendor management tools

The enterprise VMO (Section 1.7) runs on **vendor management software** — the GRC-adjacent tooling that holds the register, contracts, assessments, risk, and scorecards. Market examples (illustrative, not exhaustive): **ServiceNow** (vendor/vendor-risk management modules, popular in IT-centric organisations), **Archer** (GRC suite with third-party risk management, common in banks and large enterprises), plus OneTrust, ProcessUnity, Prevalent, and others in the TPRM tooling space. Tool selection considerations: where the register of truth lives, integration with procurement/P2P and contract management systems, assessment workflow (questionnaire distribution, evidence collection), monitoring feeds (financial/security signals, 8.4), and reporting to risk committees. The discipline point: **the tool is a means** — a mature process with a spreadsheet beats an immature process with a six-figure GRC platform; the common failure is buying the platform to *replace* governance instead of to *operationalise* it.

### 10.5 Vendor management maturity

Vendor management maturity follows the classic capability progression (a common model; labels vary by consultancy):

| Level | Name | Characteristics |
|---|---|---|
| **1** | **Ad-hoc** | Vendor relationships managed reactively by whoever signed the deal; no register, no policy; procurement awards and forgets |
| **2** | **Defined** | Policy and process exist on paper; register, contract repository, and basic TPRM in place; compliance inconsistent |
| **3** | **Managed** | Processes consistently executed: tiering, scorecards, QBRs, risk monitoring; VMO exists; metrics feed management |
| **4** | **Optimized** | Portfolio-level optimisation: consolidation, strategic partnerships, innovation reviews; risk is predictive; vendor data drives decisions |

The maturity journey maps to this guide's sections: Level 1–2 is Sections 3–5 (get the basics legal); Level 3 is Sections 6–8 (run the machinery); Level 4 is Sections 2, 7, and 12 (shape the portfolio, grow the partnerships, ride the trends). Most banks sit between 2 and 3; regulators and DORA-style requirements are pushing them toward 3–4.

### 10.6 The vendor manager career

The vendor manager role (Section 1.3) is a hybrid profession — part commercial, part relationship, part risk, part analyst. The working skill set:

- **Commercial**: contract literacy (Section 4), negotiation (cross-ref [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)), financial analysis (vendor financials, TCO);
- **Relational**: stakeholder management (business, IT, legal, risk, procurement — the same cross-functional matrix as any management role in the series), communication, conflict handling (Section 7.6);
- **Risk**: TPRM literacy (Section 8), security basics, regulatory awareness (MAS/DORA in banking);
- **Analytical**: scorecard construction, spend/concentration analysis, data-driven review (Sections 2.3, 6.3);
- **Leadership**: running QBRs and JSCs, executive presence, managing *up* (sponsors) and *sideways* (peers in procurement/risk) — the toolkit in [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) and [the_managers_path_research.md](the_managers_path_research.md) applies directly.

Career notes: vendor management is a recognised profession with its own bodies (e.g., WorldCC for commercial/contract management; vendor-management academies offering certification) — but honest flag: the certification landscape is fragmented and less standardised than PMP-style credentials; employers value demonstrated portfolio results over certificates. The natural career arcs: into **procurement leadership** (front-end), into **TPRM/operational risk** (the risk side), into **general management** (the relationship-and-commercial breadth), or into **strategic partnerships/SRM** (the top tier of the profession).

---

## 11. Worked Example — Managing a Core Banking Vendor

### 11.1 Scenario

A mid-size Asian bank (Singapore-licensed) runs its retail operations on a core banking platform and is **re-platforming the core** — the single largest vendor engagement in the bank's life. For the worked example we use a generic "CoreBankCo" platform (the real vendor landscape — Temenos, Oracle Flexcube, FIS, and the cloud-native challengers — is in [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)). The bank's vendor manager ("VM") owns the post-award relationship; the example walks the full lifecycle with artifacts.

### 11.2 Selection (Section 3)

The bank runs a formal competitive tender: RFI to 6 platforms → RFP to 4 shortlisted → demos and POC (core account migration of a test tenant) → weighted scoring by a cross-functional team (business, IT, security, risk, procurement):

| Criterion | Weight | Vendor A | Vendor B (CoreBankCo) | Vendor C |
|---|---|---|---|---|
| Functional fit & architecture | 30% | 4.2 | **4.5** | 3.8 |
| TCO (10-year) | 20% | 3.8 | **4.0** | 3.5 |
| Security & compliance | 15% | 4.3 | **4.4** | 4.0 |
| Delivery & implementation capability | 15% | 4.0 | **4.2** | 3.6 |
| Financial stability | 10% | 4.5 | **4.3** | 4.1 |
| Strategic fit & roadmap | 10% | 3.5 | **4.6** | 3.2 |
| **Weighted score** | 100% | 4.06 | **4.33** | 3.72 |

Due diligence (Section 3.6) clears CoreBankCo: audited financials healthy, no sanctions exposure, SOC 2 Type II and ISO 27001 current, strong regional references including two similar-sized banks. CoreBankCo wins; negotiation (Section 4.9) trades a 5-year commitment for clean exit terms, real credits, and a roadmap commitment (cloud-native module delivery in year 2).

### 11.3 Contract (Section 4)

The stack: **MSA** (standing terms) + **project SOWs** (implementation phases) + **SLA schedule**. Headline terms:

| Term | Agreed value |
|---|---|
| Availability (core service) | 99.9% monthly |
| P1 response / resolution | 15 min / 4 hours |
| P2 response / resolution | 1 hour / 1 business day |
| P3 response / resolution | 4 business hours / 5 business days |
| Service credits | 5% of monthly fees per 0.1% availability shortfall; cap 25% |
| Termination for convenience | 12 months' notice, no penalty; transition assistance 6 months at cost |
| Source code escrow | Yes — release on insolvency/cessation of support |
| Data | DPA (PDPA + GDPR where applicable), data residency in-region, deletion certification on exit |
| Liability | Cap: 12 months' fees; carve-outs: data breach, IP infringement, gross negligence |
| Audit | Right-to-audit annually, plus regulator access per MAS Notice 658 |

### 11.4 Onboarding (Section 5)

Security assessment accepts CoreBankCo's SOC 2/ISO 27001 evidence plus a targeted pen test of the integration scope; KYV re-screen complete. Data access follows least privilege: vendor staff get environment-scoped roles via SSO, no standing prod access, background checks for the 12 on-site consultants. Integration proceeds (API gateway, ISO 20022 message flows, monitoring feeds). Go-live: phased cutover (test tenant → pilot branch → full), rehearsed rollback, **4-week hypercare** with daily defect triage and exit criteria (5 consecutive days without a P1/P2). Readiness assessment signs off before each cutover wave.

### 11.5 Performance (Section 6) — the scorecard

First full quarter's scorecard (dimensions from 6.3, weights per tier 1):

| Dimension (weight) | Measure | Target | Actual | Score /5 |
|---|---|---|---|---|
| Quality (25%) | Defects per release | ≤ 5 | 3 | 4.5 |
| Delivery (25%) | SLA compliance (availability) | ≥ 99.9% | 99.94% | 4.6 |
| Delivery (25%) | P1 resolution within target | 100% | 95% | 3.5 |
| Cost/Value (15%) | Spend vs budget | ≤ 100% | 98% | 4.5 |
| Relationship (10%) | Stakeholder satisfaction | ≥ 4.0 | 4.3 | 4.3 |
| Risk/Compliance (10%) | Open risk findings | ≤ 3 | 5 | 3.0 |
| **Weighted total** | | | | **4.2 (Green)** |

One amber item (P1 resolution — one incident breached the 4-hour target, credit applied) and one red-ish (open risk findings: two pen-test findings in remediation). The QBR agenda (Section 6.5) walks the scorecard, the P1 post-mortem, the remediation plan for the findings, the year-2 roadmap commitment, and the action log — every action owned and dated.

### 11.6 Risk (Section 8) — the risk register excerpt

| Risk | Inherent | Controls | Residual | Owner | Action |
|---|---|---|---|---|---|
| Core platform outage (operational) | High | SLA credits, monitoring, BCP (RTO 4h, tested semi-annual), 99.9% design | Medium | VM + IT Ops | Next BCP test: Q3 |
| Vendor financial distress (financial) | Medium | Quarterly financial review, escrow in place | Low | VM | Monitor; trigger review at rating downgrade |
| Supply-chain compromise of platform (cyber) | High | Vendor security monitoring, SBOM review, access least-privilege, pen tests | Medium | InfoSec | Add SBOM verification to release checklist |
| Concentration — single core vendor (concentration) | High | Escrow, exit plan, documented run-off terms, roadmap influence | High* | VM + ExCo | *Accepted: core re-platforming is a portfolio decision; mitigate via exit plan currency |
| MAS Notice 658 material-outsourcing compliance (regulatory) | High | Materiality assessment filed, notification to MAS, audit/access clauses, BCP alignment | Low | Risk + VM | Annual re-materiality review; regulator site-visit prep |

The concentration line is the honest one: the bank *accepts* single-core-vendor risk because re-platforming is prohibitive (Section 10.2) — the mitigation is a living exit plan and escrow, not pretend alternatives.

### 11.7 Offboarding (Section 9) — the exit that never happened (yet)

The exit plan is a *standing artifact*: transition timeline, data-export specification (accounts, balances, transactions in vendor-neutral formats), deletion certification process, run-off scope (12 months at cost), and knowledge-transfer checklists. It is reviewed annually and at every material change (new module, acquisition of CoreBankCo). If the bank ever exits — to insource, to a successor core, or to a cloud-native platform — Section 9.1's checklist executes, and the contract's year-5 exit terms (Section 11.3) determine the price of the exit. The lesson of the worked example: **every artifact above is ordinary** — scorecard, risk register, QBR agenda, exit plan — and that ordinariness *is* the discipline. A vendor relationship run on this machinery performs; one run on goodwill alone is one acquisition, one breach, or one quarter of P1s away from crisis.

---

## 12. Future Trends (2026+)

### 12.1 AI in vendor management

- **Contract intelligence** — AI-assisted contract review (clause extraction, risk flagging, benchmark comparison against your playbook) is mainstreaming in CLM tools; the honest caveat: AI finds *patterns*, lawyers still own *judgement* — and AI-trained playbooks encode your own biases until deliberately curated.
- **AI-driven vendor analysis** — TPRM platforms are adding AI analysis of vendor data: financial signals, news/sentiment, security-posture feeds, and automated assessment scoring. Value is real for *triage* (which vendors need human attention); risk is over-reliance on opaque scoring — keep the human review for tier 1 and for anything AI flags.
- **Vendor data platforms** — the trend toward consolidated vendor master data (one record per vendor across procurement, contract, risk, finance, and operations) as the single source of truth — the data foundation that makes AI analysis possible at all.
- **AI vendors as a new category** — managing *AI/LLM vendors* (model providers, AI SaaS) is a new vendor class with its own questions: data use in training, output liability, model governance, evaluation evidence — see [../technology/enterprise_ai_gateway_guide.md](../technology/enterprise_ai_gateway_guide.md) for the enterprise-AI vendor context.

### 12.2 Supply-chain resilience

Post-COVID, post-SolarWinds, and post-geopolitical-shock, resilience is the portfolio theme: **reshoring/nearshoring** (moving critical production/service delivery closer), **multi-sourcing** for critical components (Section 2.5), inventory/dual-supply strategies, and **supply-chain transparency** (SBOMs, sub-tier visibility, Section 8.6–8.7). Regulators and customers increasingly expect resilience *evidence*, not resilience *claims*.

### 12.3 ESG in vendor selection

**ESG criteria are entering the selection matrix** (Section 3.2): sustainability (emissions, energy), labour standards, and governance/ethics scores, driven by regulation (EU CSDDD — corporate sustainability due diligence — and similar regimes) and customer expectation. Honest flag: ESG scoring is still immature and inconsistently standardised; treat vendor ESG data as directional, verify material claims, and expect the standards to consolidate.

### 12.4 Regulatory evolution

- **DORA in operation** — the 17 January 2025 application is now live reality for EU entities: the register of information, contractual clauses, and the oversight framework for critical ICT third-party providers are being exercised; expect enforcement and refinement (and non-EU banks feeling the contractual flow-down).
- **MAS TPRMG (P004-2026)** — Singapore's proposed Guidelines on Third-Party Risk Management (March 2026 consultation; final guidance pending as of mid-2026) point to TPRM as a *board-level, lifecycle-wide* discipline — the regulatory codification of everything in this guide.
- **Evolving TPRM globally** — the direction everywhere is the same: more scope (all third parties, not just "outsourcing"), more evidence (registers, assessments, testing), more concentration scrutiny, and more accountability (board/executive ownership).

### 12.5 The consolidation trend

Consolidation (Section 2.4) continues — platform vendors absorbing point solutions, hyperscalers absorbing SaaS layers, banks pruning vendor bases for manageability — but with the *maturity* caveat: consolidation into fewer, bigger vendors trades management overhead for **concentration risk**, so the 2026 pattern is "consolidate commodities, multi-source criticals, and measure concentration as a first-class portfolio metric."

### 12.6 Trends summary

| Trend | What it means for vendor managers |
|---|---|
| AI in VM | Faster triage and contract analysis; new AI-vendor risk questions |
| Supply-chain resilience | Multi-sourcing, transparency (SBOMs), sub-tier visibility as default expectations |
| ESG in selection | New criteria, new evidence demands, still-standardising |
| Regulation (DORA, MAS TPRMG) | The lifecycle discipline becomes *statutory*; registers and evidence are non-negotiable |
| Consolidation | Fewer, bigger vendors — concentration management becomes the counter-discipline |

The through-line: **vendor management is converging with risk management and with data** — the vendors of 2030 will be managed on unified data platforms, under statutory lifecycle frameworks, by professionals who are part commercial, part risk, part analyst. The fundamentals in this guide — lifecycle discipline, honest segmentation, real scorecards, living exit plans — are the part that doesn't change.

---

## 13. Glossary

| Term | Definition |
|---|---|
| **Vendor / Supplier** | An external party providing products or services; "vendor" and "supplier" are interchangeable (regional preference: vendor in the US/IT, supplier in Europe/UK and in procurement literature) |
| **Third party** | Any external party the organisation depends on; vendors are third parties, but so are partners, outsourcers, and service providers |
| **Procurement** | The front-end discipline: sourcing, tendering, negotiation, purchasing (Section 1.2) |
| **Sourcing** | The part of procurement that finds, qualifies, and selects suppliers |
| **RFI / RFP / RFQ** | Request for Information / Proposal / Quotation — the staged instruments of competitive selection (Section 3.1) |
| **Tender** | A formal competitive selection process, often regulated, with documented evaluation |
| **TCO** | Total Cost of Ownership — the lifecycle cost including exit (Section 3.3) |
| **MSA** | Master Services Agreement — the standing contract terms (Section 4.1) |
| **SOW** | Statement of Work — the specific engagement scope under an MSA (Section 4.1) |
| **SLA** | Service Level Agreement — measurable performance targets (Section 4.4) |
| **KPI** | Key Performance Indicator — a performance measure (Section 6.2) |
| **Scorecard** | The consolidated multi-dimension performance view of a vendor (Section 6.3) |
| **VSM** | Ambiguous: "vendor-specific measures" (contract-unique KPIs, Section 6.4 — informal) or, dominantly, Value Stream Management (Lean/DevOps) |
| **QBR** | Quarterly Business Review — the quarterly performance/governance forum (Section 6.5) |
| **JSC** | Joint Steering Committee — the executive governance board for strategic vendors (Section 7.1) |
| **Escalation** | The structured path for raising issues to the level that can act (Section 7.3) |
| **Kraljic matrix** | The 1983 2×2 portfolio model: profit impact × supply risk → strategic/leverage/bottleneck/routine (Section 1.5) |
| **Make-vs-buy** | The build-vs-buy-vs-partner sourcing decision (Section 2.1) |
| **Insourcing** | Bringing a capability back in-house (Section 2.6) |
| **Multi-sourcing** | Using multiple vendors for resilience/leverage (Section 2.5) |
| **Vendor consolidation** | Reducing the vendor base for better terms and lower overhead (Section 2.4) |
| **Concentration risk** | Over-dependence on a single vendor (Sections 2.3, 8.2) |
| **VRM** | Vendor Risk Management — risk process applied to vendors (Section 8.1) |
| **TPRM** | Third-Party Risk Management — the enterprise-wide risk framework (Section 8.1) |
| **Fourth party / Sub-vendor** | A vendor's own vendor/subcontractor (Section 8.6) |
| **ISO 27036** | The ISO series on cybersecurity in supplier relationships (Section 8.7) |
| **NIST SP 800-161** | NIST's Cybersecurity Supply Chain Risk Management (C-SCRM) practice guide (Section 8.7) |
| **MAS Notice 658** | Singapore's outsourcing notice for banks: Management of Outsourced Relevant Services (issued Dec 2023, effective 11 Dec 2024) (Section 8.8) |
| **DORA** | EU Digital Operational Resilience Act — applied 17 January 2025; includes the ICT third-party risk regime (Section 8.8) |
| **Critical outsourcing** | Outsourcing of material/critical functions, subject to notification and heightened oversight (Section 8.8) |
| **BCP** | Business Continuity Plan — the tested continuity arrangements for critical services (Section 8.8) |
| **Offboarding** | The managed end of a vendor relationship (Section 9) |
| **Exit management** | The discipline of leaving a vendor relationship cleanly (Section 9.2) |
| **Lock-in** | The state where exit is prohibitively expensive/difficult (Section 9.2) |
| **Data portability** | The right/mechanism to retrieve your data in usable form (Section 9.2) |
| **Run-off** | The contracted period of continued service during transition (Section 9.3) |
| **VMO** | Vendor Management Office — the central governance function (Section 1.7) |
| **MSP** | Managed Service Provider (Section 10.1) |
| **BPO** | Business Process Outsourcing (Section 10.1) |
| **ITO** | IT Outsourcing (Section 10.1) |
| **ISV** | Independent Software Vendor (Section 10.1) |
| **CSP** | Cloud Service Provider (Section 10.3) |
| **GRC** | Governance, Risk and Compliance — the tooling/capability domain (Section 10.4) |
| **ServiceNow / Archer** | Vendor/TPRM software platforms (illustrative market examples, Section 10.4) |
| **ESG** | Environmental, Social and Governance criteria (Section 12.3) |
| **SolarWinds** | The 2020 supply-chain attack — compromised software updates weaponised a trusted vendor (Section 8.2) |
| **Supply-chain attack** | An attack that compromises a product/service to reach its consumers (Section 8.2) |
| **Hypercare** | The intensified post-go-live support period (Section 5.5) |
| **Cutover** | The go-live switch itself (Section 5.5) |
| **KYV** | Know Your Vendor — due diligence of a supplier, the counterpart of KYC (Sections 3.6, 5.2; informal terminology) |

---

## 14. References and Further Reading

**Regulatory (verified August 2026):**
- MAS Notice 658 — Management of Outsourced Relevant Services for Banks (issued 11 December 2023; effective 11 December 2024); MAS Notice 1121 (merchant banks); 2023 Guidelines on Outsourcing — mas.gov.sg
- MAS Consultation Paper P004-2026 — proposed Guidelines on Third-Party Risk Management (6 March 2026; consultation closed 20 April 2026)
- Regulation (EU) 2022/2554 (DORA) — Digital Operational Resilience Act; applied 17 January 2025 (EIOPA/ESMA/EBA)

**Standards:**
- ISO/IEC 27036 series — Cybersecurity — Supplier relationships (Parts 1:2021, 2, 3:2023, 4) — iso.org
- NIST SP 800-161 Rev. 1 — Cybersecurity Supply Chain Risk Management Practices for Systems and Organizations (May 2022; Update 1, Nov 2024) — csrc.nist.gov
- ISO/IEC 27001; AICPA SOC 2 (Trust Services Criteria)

**Foundational:**
- Kraljic, P. — "Purchasing Must Become Supply Management," *Harvard Business Review*, Sept–Oct 1983
- World Commerce & Contracting (WorldCC) — commercial/contract management body of knowledge

**Repo cross-references:**
- [strategic_management_guide.md](strategic_management_guide.md) — make-vs-buy, sourcing strategy
- [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) — negotiation, stakeholder skills
- [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — MAS/DORA outsourcing risk, BCP, sanctions/AML
- [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) — core banking platforms (Temenos, Oracle, FIS)
- [../technology/finops_guide.md](../technology/finops_guide.md) — cost/TCO lens
- [../technology/cloud_providers_guide.md](../technology/cloud_providers_guide.md) — cloud vendors, shared responsibility, cloud exit
- [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md) — PMBOK procurement management
- [../technology/enterprise_ai_gateway_guide.md](../technology/enterprise_ai_gateway_guide.md) — AI vendor context
- [../technology/data_governance_guide.md](../technology/data_governance_guide.md) — data governance

*Verification notes:* MAS Notice 658 / Notice 1121 and DORA statuses verified against regulator and practitioner sources (August 2026); ISO 27036 and NIST SP 800-161 verified against ISO/NIST publications. "Vendor-specific measures (VSM)" is flagged as informal terminology (Section 6.4); the MAS TPRMG consultation status is as reported by MAS and practitioner commentary as of mid-2026. Notice numbers and regulatory statuses should be re-verified against MAS/EU sources before formal use.
