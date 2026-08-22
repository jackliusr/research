# IT Strategy: A Comprehensive Guide

> **A deep-dive reference on the IT-strategy discipline — alignment, operating models, governance, portfolio, sourcing, digital transformation, value, and the IT-doesn't-matter debate — with a worked example for a CACIB-style bank.**

**Author:** Jack Liu Shurui
**Domain:** Technology / Management & Leadership
**Part of:** Management & Leadership Series — the IT-side counterpart to [Strategic Management](strategic_management_guide.md)
**Companion guides:** [Business Case Development](business_case_development_guide.md) · [Vendor Management](vendor_management_guide.md) · [MBA Body of Knowledge](mba_body_of_knowledge_guide.md) · [Organizational Behavior](organizational_behavior_guide.md) · [McKinsey Approach](mckinsey_approach_guide.md) · [Management Consulting Skills](management_consulting_skills_guide.md)
**Banking context:** [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) · [Crédit Agricole Software Systems](../banking/credit_agricole_software_systems_guide.md) · [HSBC Software Systems](../banking/hsbc_software_systems_guide.md)
**Technology context:** [Data Architect Skill Gaps](../technology/data_architect_skillgaps_guide.md) · [FinOps](../technology/finops_guide.md) · [Monolith to Microservices](../technology/monolith_to_microservices_guide.md) · [Enterprise AI Gateway](../technology/enterprise_ai_gateway_guide.md)
**Last Updated:** August 2026

---

## Table of Contents

1. [IT Strategy Overview](#1-it-strategy-overview)
1. [The Alignment: Henderson–Venkatraman](#2-the-alignment-hendersonvenkatraman)
1. [The Operating Models: Weill–Ross](#3-the-operating-models-weillross)
1. [The IT Governance](#4-the-it-governance)
1. [The Portfolio: McFarlan's Strategic Grid](#5-the-portfolio-mcfarlans-strategic-grid)
1. [The Sourcing: Build, Buy, Outsource](#6-the-sourcing-build-buy-outsource)
1. [The Digital Transformation](#7-the-digital-transformation)
1. [The IT Value](#8-the-it-value)
1. [The IT-Doesn't-Matter Debate](#9-the-it-doesnt-matter-debate)
1. [The Worked Example: IT Strategy for a Bank](#10-the-worked-example-it-strategy-for-a-bank)
1. [The Summary: One Page](#11-the-summary-one-page)
1. [Glossary](#glossary)
1. [References and Further Reading](#references-and-further-reading)

---

## 1. IT Strategy Overview

### 1.1 What IT Strategy Is

**IT strategy is the plan of action that determines how an organisation's information-technology capability — its systems, data, infrastructure, applications, people, and governance — will be built, run, and used to advance the business strategy.** The discipline of IT strategy answers one question above all others, and that question *is* the definition:

> **The alignment question: how does IT contribute to the business strategy, and who decides?**

That is not a rhetorical flourish. The canonical research behind the discipline — Henderson and Venkatraman's Strategic Alignment Model (1993, verified below) — argues that the difficulty in realising value from IT investments stems *first* from the lack of alignment between business and IT strategy, and *second* from the absence of a dynamic administrative process to keep the two continuously aligned. In other words: **IT strategy is the alignment, made operational.** Everything else in this guide — operating models, governance, portfolio, sourcing, transformation, value — is machinery for producing and sustaining that alignment.

This framing deliberately mirrors its business-side sibling, [Strategic Management](strategic_management_guide.md): where strategic management asks "which game are we playing, and how do we win?", IT strategy asks "how does technology change the game, and how do we use it to win?". Where the [MBA Body of Knowledge](mba_body_of_knowledge_guide.md) treats strategy and finance as separate chapters, IT strategy is where they collide: every IT investment is a resource-allocation decision, and the [Business Case Development](business_case_development_guide.md) guide covers the investment-decision machinery that IT strategy feeds.

### 1.2 IT Strategy vs Business Strategy vs Digital Strategy

| Concept | Question it answers | Owner | Horizon |
|---|---|---|---|
| **Business strategy** | Which markets, products, and advantages? | CEO/board | 3–10 years |
| **IT strategy** | How does the IT estate (build, buy, run) serve the business strategy? | CIO/CTO | 2–5 years |
| **Digital strategy** | How do emerging technologies change the business model itself? | CEO + CDO/CIO | 2–5 years, continuously revised |
| **Digital transformation** | How do we execute the digital strategy across the organisation? | CEO-sponsored, DTO-led | programme-based |

A useful rule of thumb: business strategy says *what and why*; IT strategy says *which systems, built or bought, run how, at what cost and risk*; digital strategy says *what if technology changes the what itself*. In a bank, "we will be the leading corporate-and-investment bank in Asia" is business strategy; "we will run a unified risk platform across all desks, buy market data from Refinitiv, and outsource the data centre" is IT strategy; "we will make our client onboarding fully digital and API-first" is digital strategy shading into transformation.

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Definition** | The plan for how IT capability (systems, data, infrastructure, people, governance) will be created and used to achieve business goals; its essence is **alignment** between business strategy and IT decisions (Henderson–Venkatraman, 1993). |
| **Core question** | *The alignment question*: what is IT for, and who decides? |
| **Scope** | Everything the IT function does: strategy formulation, architecture and operating model, governance and decision rights, portfolio and investment, sourcing, delivery, operations, and value measurement. |
| **Time horizon** | 2–5 years for the strategy itself; continuously refreshed (technology moves faster than strategy cycles). |
| **Owner** | CIO/CTO in partnership with the CEO and business-line heads; governance sits with the board (via IT governance). |
| **Outputs** | The IT strategy document, the operating model, the IT investment portfolio, governance structures, sourcing strategy, value/performance scorecard. |
| **Key inputs** | Business strategy (from [Strategic Management](strategic_management_guide.md)), market/competitive context, regulatory constraints, technology trends, current-state IT assessment. |
| **Enablers** | Executive sponsorship, business–IT partnership, governance with teeth, portfolio discipline, sourcing flexibility, measurable value. |
| **Failure mode** | IT strategy as a stand-alone document that the business never reads — i.e., **misalignment**: the IT estate grows without connection to business goals, costs rise, and "IT" becomes a cost centre to be cut rather than a capability to be invested in. |
| **Discipline family** | A functional-level strategy within the corporate strategy stack (see [MBA Body of Knowledge](mba_body_of_knowledge_guide.md), strategy chapter); sibling to HR, finance, and marketing strategies, but with uniquely fast-changing technology and uniquely deep reach into every process. |

### 1.4 Why IT Strategy Is Hard

Three properties make IT strategy harder than other functional strategies:

1. **IT is everywhere.** Technology is embedded in products (trading platforms), processes (loan origination), and relationships (client APIs). There is no clean boundary around "the IT function".
2. **IT changes fast.** The strategy horizon for a technology is often shorter than the strategy horizon for the business it serves — 18-month technology cycles inside 5-year strategies.
3. **IT value is indirect.** An investment in a risk platform shows up as fewer losses and lower capital charges, not as a revenue line. Attribution is hard, which is why [IT value](#8-the-it-value) needs its own discipline.

The rest of this guide builds the discipline piece by piece. The order matters: alignment first (Section 2), because it defines the goal; then the operating model (Section 3), which is the bridge from strategy to architecture; then governance (Section 4), which makes decisions; then portfolio (Section 5), which allocates money; then sourcing (Section 6), which allocates build/buy; then transformation (Section 7) and value (Section 8); then the sceptical counterweight (Section 9); and finally a worked example (Section 10) that ties it together for a bank.

### 1.5 The IT Strategy Process

The formulation cycle is the consulting framing of [McKinsey Approach](mckinsey_approach_guide.md) and [Management Consulting Skills](management_consulting_skills_guide.md) applied to IT — current state, future state, gap, roadmap, then the machinery to make it stick:

1. **Current-state assessment** — inventory the estate (applications, infrastructure, data, skills), classify every system on the strategic grid (Section 5), and cost it honestly split into run-the-bank vs change-the-bank.
2. **Future-state definition** — from the business strategy and the SAM analysis (Section 2): what capabilities must IT provide in 3–5 years, and what must the firm be able to do that it cannot do today?
3. **Gap analysis** — what must change: new platforms, retirements, integration work, skills, sourcing shifts. The operating model (Section 3) states *how standardised and integrated* the future must be; the gap is the work.
4. **Roadmap and investment plan** — sequenced programmes, each with a business case (see [Business Case Development](business_case_development_guide.md)), prioritised through the portfolio (Section 5), with the sourcing posture (Section 6) embedded.
5. **Governance and metrics** — decision rights (Section 4) and the scorecard (Section 8) that keep the strategy alive between planning cycles.

Iterate quarterly. The strategy *document* is a snapshot; the *process* is the strategy — the "dynamic administrative process" that Henderson and Venkatraman identified as the second requirement for IT value.

### 1.6 IT Strategy vs Enterprise Architecture vs IT Operations

| Layer | Question it answers | Horizon | Owner | Framework |
|---|---|---|---|---|
| **IT strategy** | What should IT achieve for the business? | 3–5 years | CIO with the business | SAM, operating model, portfolio |
| **Enterprise architecture** | What is the target structure — business, data, application, technology? | 2–5 years | Chief Architect | TOGAF |
| **Delivery & operations** | How do we build and run it, day to day? | Weeks–months | Delivery and ops heads | Agile/DevOps, ITIL |

Strategy sets direction; EA translates it into architecture work products and standards; delivery and operations execute. The classic failure is skipping the EA layer and going straight from strategy to projects — which is how banks end up with 2,000 applications nobody chose. The [Enterprise AI Gateway](../technology/enterprise_ai_gateway_guide.md) guide shows the same layering for AI: strategy decides what AI is for, EA decides the platform structure, ops runs it.

---

## 2. The Alignment: Henderson–Venkatraman

### 2.1 The Strategic Alignment Model (SAM), 1993 — verified

The foundational model of the IT-strategy discipline is the **Strategic Alignment Model (SAM)** of **John C. Henderson and N. Venkatraman**, published in *"Strategic Alignment: Leveraging Information Technology for Transforming Organizations,"* **IBM Systems Journal, Vol. 32, No. 1, 1993** (the same authors' *"Continuous Strategic Alignment: Exploiting Information Technology Capabilities for Competitive Success"*, with Scott Oldach, appeared the same year in European Management Journal; a 1992 working paper preceded both — the canonical citation is the 1993 IBM Systems Journal paper). *Verified against multiple independent sources (IBM/IEEE, ScienceDirect, academic citations); the 1993 attribution is sound.*

SAM's thesis, stated earlier in Section 1.1: **IT delivers value only when business strategy and IT strategy are aligned — and alignment must be actively managed, not assumed.** The model has two building blocks:

- **Strategic fit** — the consistency between an *external* component (strategy: how the firm positions itself in its market) and an *internal* component (infrastructure and processes: how the firm is organised to execute).
- **Functional integration** — the consistency between the *business* domain and the *IT* domain, in both directions: business strategy/structures must be reflected in IT strategy/structures, and IT capabilities must be able to shape business strategy.

Crossing these gives **four fundamental domains of strategic choice**:

| Domain | External focus | Internal focus |
|---|---|---|
| **Business strategy** | Business scope, distinctive competencies, business governance | — |
| **IT strategy** | Technology scope, systemic competencies, IT governance | — |
| **Organizational infrastructure & processes** | — | Administrative structure, processes, skills |
| **IT infrastructure & processes** | — | IS architecture, IS processes, IS skills |

Each strategy domain carries three dimensions — **scope, competencies, governance** — so the model is a 4-domain × 3-dimension grid. The insight that made the model famous is that alignment is **not one relationship but four**, depending on which domain drives:

1. **Strategy Execution** — business strategy drives organizational infrastructure, which drives IT infrastructure. The classic top-down view: top management formulates, IS management implements.
2. **Technology Potential** — business strategy drives IT strategy, which drives IT infrastructure. Top management provides the technology vision; IS management acts as technology architect.
3. **Competitive Potential** — IT strategy drives business strategy, which drives organizational infrastructure. Emerging IT capability reshapes the business model. Top management is the business visionary; IS management is the catalyst.
4. **Service Level** — IT strategy drives IT infrastructure, which drives organizational infrastructure. Building a world-class IT organisation. Top management prioritises; IS management leads the internal business of IT.

The practical lesson most organisations miss: **alignment is not "business strategy → IT strategy" in one direction.** In the Competitive Potential and Service Level perspectives, IT drives. A bank whose IT is only ever an implementer of business strategy will never discover the client-facing products that data and APIs make possible; a bank whose IT strategy is unconstrained by business strategy will build a beautiful estate that nobody uses. Both are misalignment.

### 2.2 The Alignment Table

| Perspective | Driver domain | Logic | Top management role | IS management role | When it fits |
|---|---|---|---|---|---|
| **Strategy Execution** | Business strategy → org. infra → IT infra | "Business strategy is given; IT implements it." | Strategy formulator | Strategy implementer | Stable strategy, mature industry, IT as cost to be optimised |
| **Technology Potential** | Business strategy → IT strategy → IT infra | "Choose the IT strategy that best supports the business strategy." | Technology visionary | Technology architect | IT is a source of efficiency; business wants the best technology for a fixed strategy |
| **Competitive Potential** | IT strategy → business strategy → org. infra | "Emerging IT capability changes what the business can do." | Business visionary | Catalyst (identifies IT trends and opportunities) | Disruptive technology windows; digital-first strategy |
| **Service Level** | IT strategy → IT infra → org. infra | "Build world-class IT first; the business benefits follow." | Prioritizer (allocate scarce resources) | Business leader of the IT organisation | IT quality is the binding constraint; turnaround situations |

### 2.3 Using SAM in Practice

- **Diagnose the current state.** Which perspective does the organisation actually run on? Most banks run Strategy Execution for core banking and Competitive Potential nowhere — a diagnostic finding in itself.
- **Choose the perspective per domain, not per enterprise.** Core risk and accounting systems should run Strategy Execution (stability, control). Client-facing and data capabilities should run Competitive Potential (innovation). One model for the whole firm is itself a misalignment.
- **Treat alignment as a process, not a state.** Venkatraman, Henderson and Oldach's 1993 companion paper stressed the *administrative process* — planning, monitoring, adapting — that keeps alignment continuous. Modern practice: a standing business–IT planning cycle, an enterprise architecture function, and governance (Section 4) that re-links the four domains as strategies change.
- **Complement with maturity tools.** Jerry Luftman's SAMm (Strategic Alignment Maturity, 2000s) turns SAM into a 5-level maturity model (initial/ad hoc → optimized) across six criteria (communications, value governance, partnership, scope & architecture, skills, and the "IT sophistication" of the business). It is a practitioner extension of SAM, not part of the 1993 paper — flagged so the attribution stays clean.

Alignment is the goal; the **operating model** is the bridge that makes it concrete — how much the business processes must be standardised and integrated, which then dictates the IT architecture.

### 2.4 Why Alignment Fails

The barriers are predictable, which makes them preventable:

- **IT as cost centre.** When the business narrative is "IT is expense", every alignment conversation becomes a budget negotiation. The fix is value framing (Section 8): the scorecard, business cases, and benefit tracking that let the business see IT as investment.
- **Strategy as poster.** If the business strategy itself is vague or unlived, there is nothing to align to. Alignment cannot be manufactured inside IT; it starts with a real business strategy (see [Strategic Management](strategic_management_guide.md)).
- **Static alignment.** Alignment achieved at planning time decays as markets and technology move. Henderson and Venkatraman's own "continuous alignment" paper was about exactly this: the administrative process that re-links the four domains on a cadence.
- **Structural separation.** IT buried deep in a support function, with no CIO seat at the strategy table, guarantees misalignment regardless of documents. Alignment is an organisational-design choice first.
- **One perspective everywhere.** Running Strategy Execution for everything starves Competitive Potential (no innovation); running Competitive Potential for everything destabilises operations. The perspective must fit the domain, as the worked example (Section 10) shows.

Every one of these failures is a *process* failure, not a model failure. The model is sound; the discipline is the process — a theme that returns in Section 10.3.

---

## 3. The Operating Models: Weill–Ross

### 3.1 The Four Operating Models — verified

Peter Weill and Jeanne W. Ross (MIT Center for Information Systems Research, CISR) introduced the operating-model concept in their 2004 book *IT Governance: How Top Performers Manage IT Decision Rights for Superior Results* (Harvard Business School Press, 2004), the same research programme that produced the governance archetypes in Section 4. The companion *"IT Governance on One Page"* (MIT Sloan Working Paper No. 4517-04, 2004) summarised both. *Verified: Weill & Ross, 2004, HBS Press; the four models — diversification, coordination, unification, replication — are the canonical set.*

The operating model answers a question that must be answered **before** architecture: *"To deliver value, how standardised must our business processes be, and how integrated must our data and systems be across business units?"* Two dimensions:

- **Business process standardization** — to what degree processes are executed the same way everywhere ("one way to do it").
- **Business process integration** — to what degree data, systems, and process steps link across units ("one view of the customer/risk").

| | Low integration | High integration |
|---|---|---|
| **High standardization** | **Replication** | **Unification** |
| **Low standardization** | **Diversification** | **Coordination** |

1. **Diversification** — low standardization, low integration. Independent business units with different processes and systems; corporate adds shared services (finance, HR, common infrastructure) but units run their own business. Logic: local autonomy maximises unit performance; synergy comes from portfolio effects, not shared operations. Classic: conglomerates, diversified financial groups, insurance groups with distinct product lines.
2. **Coordination** — low standardization, high integration. Units keep their own processes but share data and systems where the customer or risk is shared — one client view, integrated product delivery. Logic: the firm wins on cross-selling and one-face-to-customer without forcing one way of working. Classic: investment banks and universal banks with shared client, CRM, and risk data across desks.
3. **Unification** — high standardization, high integration. One set of end-to-end processes and one shared data model everywhere. Logic: scale, consistency, and control dominate; the firm operates as one company. Classic: retail banks, airlines, global supply-chain firms; also the regulatory-driven target for many banks' risk and finance processes (one book, one risk model).
4. **Replication** — high standardization, low integration. Standardised, proven processes replicated into autonomous local units that run them locally. Logic: global consistency of method with local independence of execution; scale in process design, flexibility in operation. Classic: McDonald's, hotel chains, and banks' standardised branch/back-office processes rolled out per country.

### 3.2 The Models Table

| Model | Logic | Standardization | Integration | Typical industries | IT implications | Notes |
|---|---|---|---|---|---|---|
| **Diversification** | Autonomy maximises unit performance; synergy via portfolio | Low | Low | Conglomerates; diversified insurers; holding-company banks | Shared only at the infrastructure layer (network, data centre, email); unit-level systems differ freely; IT is a portfolio of separate estates | Cheapest to govern centrally; hardest to consolidate later. "Every unit does its own thing." |
| **Coordination** | Win on shared customer and shared risk without one way of working | Low | High | Investment banks, universal banks, multi-desks with shared client/risk data | Shared client, product, and risk data platforms; interfaces and data standards matter more than process uniformity; MDM and data governance are critical | The natural starting model for a global CIB (see Section 10). Integration without standardization is the hardest to achieve technically — you share data but not processes, so interfaces multiply. |
| **Unification** | One company, one process, one data model; scale and control | High | High | Retail banks, airlines, global supply chains; risk/finance target states | One ERP/core platform, one data model, centralized development; highest infrastructure standardisation; change is expensive because everything is shared | Best economies of scale and control; slowest to change; high one-time transformation cost. Regulators often push banks here for risk/finance. |
| **Replication** | Global method, local execution | High | Low | Franchises (McDonald's), hotels, banks' standardised country rollouts | Standard application templates deployed per site; local data; thin central team maintains the "golden process"; heavy use of packaged software with configuration, not customisation | Great for international expansion of a proven model; risks local divergence over time ("shadow replication") if not disciplined. |

### 3.3 Why the Operating Model Matters

- **It is the bridge from alignment to architecture.** SAM says *what* should be aligned; the operating model says *how integrated and how standardised*, which is precisely what enterprise architects need before choosing platforms. See [Enterprise AI Gateway](../technology/enterprise_ai_gateway_guide.md) and the EA layer for where this lands.
- **It precedes and constrains IT decisions.** Weill and Ross's research showed top-performing firms agree the operating model *first*, then make the five key IT decisions (Section 4.4) consistently with it. Firms that skip it end up with a de facto model — usually diversification — chosen by accident rather than design.
- **Banks usually need more than one.** A global CIB typically runs **coordination** for client-facing markets (shared client and risk data across desks), **unification** for risk, finance, and regulatory reporting (one book, one model — regulator-mandated), **replication** for standardised country back-offices, and accepts **diversification** in genuinely independent subsidiaries. The art is declaring which model governs which domain — and funding the interfaces between them.

### 3.4 Choosing the Operating Model: Assessment Questions

The choice is a business decision, not an IT decision. The questions that surface it:

- **Where does the firm's value come from — local responsiveness or global integration?** Local responsiveness points to diversification/replication; global scale and one-face-to-customer point to coordination/unification.
- **Does the customer experience depend on one view of the customer?** A CIB whose relationship managers must see the client's full franchise across desks needs coordination at minimum; a retail bank needs unification.
- **Does the business depend on cross-unit data — risk, finance, capital?** Shared risk data across desks (coordination) or one risk book (unification) is rarely optional for a bank: regulation decides.
- **How standard is the product?** Identical products delivered everywhere → replication (local execution) or unification (one global process). Bespoke products per market → diversification or coordination.
- **What does the current estate actually do?** Map today's real standardisation and integration levels and compare to what the strategy demands. The gap between de facto and desired is the transformation programme (Section 7).

Weill and Ross's research is unambiguous: firms that agree the operating model *before* making architecture and investment decisions outperform those that let the estate decide by drift. The de facto model is usually diversification — chosen by nobody, paid for by everyone.

---

## 4. The IT Governance

### 4.1 What IT Governance Is

**IT governance is the system of decision rights and accountability that encourages desirable behaviour in the use of IT.** Weill and Ross's one-line definition (2004): governance specifies *who makes IT decisions and who is accountable for them* — "the decision rights and accountability framework to encourage desirable behaviour in the use of IT." Governance is **not** management: management makes and executes decisions; governance decides who gets to decide, and how decisions are monitored. The [business case](business_case_development_guide.md) machinery runs *inside* the governance framework — governance defines who approves, the business case defines what gets approved.

The discipline has three canonical frameworks (COBIT, ITIL, TOGAF) plus the decision-rights school (Weill–Ross). They are complementary, not competing: **COBIT governs, ITIL manages services, TOGAF architects.** A bank runs all three at once.

### 4.2 COBIT — verified

**COBIT** (Control Objectives for Information and related Technology) is the IT-governance framework of **ISACA** (originally the Information Systems Audit and Control Association, founded 1969). First published **1996** as an audit-oriented control framework; it broadened into full governance with COBIT 4 (2005–07), COBIT 5 (2012, with the goals-cascade and enablers model), and the current **COBIT 2019** (governance system principles, 40 core governance/management objectives across five domains — EDM: Evaluate, Direct, Monitor; APO: Align, Plan, Organise; BAI: Build, Acquire, Implement; DSS: Deliver, Service, Support; MEA: Monitor, Evaluate, Assess). *Verified against ISACA's own history pages and framework timeline: COBIT originated in the mid-1990s (first release 1996), current version COBIT 2019.*

COBIT's signature mechanism is the **goals cascade**: stakeholder needs → enterprise goals → alignment goals → governance objectives → practices. Every IT activity is traceable to an enterprise goal. For a bank, COBIT is the natural home for the *control* question: segregation of duties, audit trails, risk appetite for IT, regulatory reporting controls — the "govern the use of IT" layer that regulators (and the [capital markets architecture](../banking/capital_markets_architecture_guide.md) of a bank) take for granted.

### 4.3 ITIL and TOGAF — verified

**ITIL** (IT Infrastructure Library) is the service-management framework: how IT services are *designed, delivered, operated, and improved*. Origins: the UK government's Central Computer and Telecommunications Agency (CCTA), which began publishing ITIL volumes in the late 1980s (v1 ~1989); v2 (2000) popularised it commercially; v3 (2007) restructured into the service lifecycle; **ITIL 4 (2019)** moved to the Service Value System and 34 practices. Current owner: **Axelos** (a joint venture created 2013 by the UK Cabinet Office and Capita). ITIL is about *operational excellence* — incident, problem, change, release, service-level management — the "run it well" half of IT strategy. *Verified: origins in UK CCTA 1980s, Axelos ownership, ITIL 4 in 2019 — standard, widely documented.*

**TOGAF** (The Open Group Architecture Framework) is the enterprise-architecture framework: how the target architecture is *developed and governed*. First published **1995** by The Open Group, derived from the US Department of Defense's TAFIM; current version TOGAF 10 (2022). Its core is the **ADM** (Architecture Development Method) — a repeatable cycle (preliminary → architecture vision → business/data/application/technology architectures → opportunities & solutions → migration planning → implementation governance → change management) — plus the enterprise continuum and content framework. TOGAF gives the "build the right target architecture" half: it operationalises the operating model (Section 3) into business, data, application, and technology architecture work products. *Verified: The Open Group, 1995, TAFIM lineage, ADM cycle — standard, widely documented.*

### 4.4 Decision Rights and the Weill–Ross Archetypes — verified

The decision-rights school, from the same 2004 MIT CISR research as the operating models, is arguably the most *usable* part of IT governance. Weill and Ross identified **five key IT decisions** every enterprise must make:

1. **IT principles** — the role of IT in the business ("how does IT contribute?").
2. **IT architecture** — integration and standardisation rules (directly derived from the operating model).
3. **IT infrastructure** — shared services and capabilities (build centrally, consume locally).
4. **Business application needs** — what applications are needed (the business demand side).
5. **IT investment and prioritisation** — where the money goes (the [portfolio](#5-the-portfolio) decision).

And **six governance archetypes** describing *who has decision rights* (verified — the canonical Weill–Ross set; see e.g. the MIT CISR working papers and their 2004 book):

| Archetype | Who decides | Typical use |
|---|---|---|
| **Business monarchy** | C-level business executives (CEO/COO) | IT principles, major investments |
| **IT monarchy** | IT executives (CIO/CTO) | Architecture, infrastructure, technical standards |
| **Feudal** | Business-unit leaders, each for their own unit | Unit application needs (the anti-pattern of the uncoordinated enterprise) |
| **Federal** | Central corporate + business units, with coordination | Principles and investments agreed jointly — the most common top performer |
| **IT duopoly** | Two-party: IT executives + one business group | Application needs, investment |
| **Anarchy** | Each individual/user decides | None intentionally — a failure state |

Research findings that still hold: top performers make the same five decisions **consistently and transparently**, they rarely use a single archetype for everything (the right archetype varies by decision and by operating model), and they couple decision rights with **three implementation mechanisms** — *decision-making structures* (committees, roles), *alignment processes* (SLAs, chargeback, portfolio reviews, architecture reviews), and *communication approaches* (announcements, scorecards, training). Governance without mechanisms is a poster on a wall.

### 4.5 The Governance Table

| Framework / school | Owner / origin | First published | Focus (what question it answers) | Key artifacts | Where it fits in a bank |
|---|---|---|---|---|---|
| **COBIT** | ISACA | 1996 (current: COBIT 2019) | "Are we governing and controlling IT properly?" — goals cascade, 40 objectives, 5 domains (EDM/APO/BAI/DSS/MEA) | Control objectives, capability assessments, audit evidence | Audit, risk, regulatory control, board-level IT oversight |
| **ITIL** | UK CCTA → Axelos | late 1980s (current: ITIL 4, 2019) | "Are we operating IT services well?" — Service Value System, 34 practices | Service catalogue, SLAs, incident/change/release practice | Service management, run-the-bank operations, vendor SLAs |
| **TOGAF** | The Open Group | 1995 (current: TOGAF 10) | "Are we architecting toward the right target?" — ADM cycle, 4 architecture domains | Architecture vision, target architectures, roadmaps | Enterprise architecture, platform strategy, migration planning |
| **Weill–Ross decision rights** | MIT CISR (Weill & Ross) | 2004 | "Who decides what about IT?" — 5 decisions × 6 archetypes | Decision-rights matrix (RACI-style), governance mechanisms | CIO operating model, investment committees, architecture boards |
| **Operating model** (Section 3) | MIT CISR (Weill & Ross) | 2004 | "How standardised and integrated must we be?" | Operating-model statement per domain | Pre-architecture alignment with the business |

### 4.6 Governance Mechanisms in Practice

Decision rights are the skeleton; mechanisms are the muscle. Weill and Ross's three categories, made concrete:

- **Decision-making structures** — who sits in which body: the group investment committee (federal: business + IT), the enterprise architecture review board (IT monarchy), programme steering committees (duopoly per programme). The calendar of these bodies *is* the governance cadence.
- **Alignment processes** — the machinery that links decisions to strategy: portfolio review cadence (Section 5), business-case gates ([Business Case Development](business_case_development_guide.md)), architecture compliance reviews (TOGAF), SLA and chargeback mechanisms (ITIL + [FinOps](../technology/finops_guide.md)), benefit tracking (Section 8).
- **Communication approaches** — governance charters, a published decision-rights matrix, scorecard reporting to the board. Governance that is not communicated is governance that does not exist.

Three practical tests for any governance design: (1) can any executive answer "who decides IT investment and priorities?" without checking? (2) does the committee calendar match the strategy cadence (quarterly reviews, not annual)? (3) are decisions recorded, with owners and dates, and followed up? Governance that fails these tests is decoration — and the first casualty of a crisis.

Governance is the *decision* layer; the **portfolio** is what those decisions allocate money to — the bridge from governance to investment.

---

## 5. The Portfolio: McFarlan's Strategic Grid

### 5.1 The Strategic Grid — verified (1984)

The **strategic grid** was proposed by **F. Warren McFarlan** (Harvard Business School) — developed with James L. McKenney in their 1983 *Harvard Business Review* article "The Information Archipelago," and set out in its canonical form in McFarlan's 1984 HBR article **"Information Technology Changes the Way You Compete" (May–June 1984)**. *Verified: the 1983 precursor and the 1984 HBR article are both widely and consistently attributed; textbooks cite "McFarlan (1984)" for the grid. This guide follows the 1984 attribution with the 1983 origin noted.*

The grid is a 2×2 that positions IT along two axes:

- **Strategic impact of existing IT applications** (the *current* portfolio) — how dependent is the firm's present operation on IT?
- **Strategic impact of IT applications under development** (the *future/planned* portfolio) — how much will IT shape the firm's future strategy?

| | Low future impact | High future impact |
|---|---|---|
| **High current impact** | **Factory** | **Strategic** |
| **Low current impact** | **Support** | **Turnaround** |

1. **Support** — low current, low future impact. IT keeps the lights on but neither operations nor strategy depend on it. Governance posture: cost control, minimal senior attention. Example: a small advisory firm's email and accounting systems.
2. **Factory** — high current, low future impact. Operations *depend* on IT today, but IT will not reshape strategy. Posture: reliability, availability, cost discipline, strong service management (ITIL territory); downtime is a business outage. Example: a bank's core account-processing systems — critical to run, not strategic to build.
3. **Turnaround** — low current, high future impact. IT is not yet load-bearing but the firm intends to make it so. Posture: innovation management, pilot governance, senior sponsorship, careful scaling — this is where new platforms (APIs, AI, cloud) start. Example: a bank's first AI/LLM initiatives or a new digital client platform.
4. **Strategic** — high current, high future impact. IT is both load-bearing today and strategy-shaping tomorrow. Posture: full senior attention, business–IT partnership, premium funding, rigorous portfolio governance. Example: a digital-only bank's entire estate.

### 5.2 The Portfolio Table

| Quadrant | Current impact | Future impact | IT posture | Management posture | Bank examples | Governance implication |
|---|---|---|---|---|---|---|
| **Support** | Low | Low | Keep cheap and reliable | Delegate to IT management; cost control | Office productivity, HR systems, expense systems | Light-touch governance; standard sourcing (buy/SaaS) |
| **Factory** | High | Low | Run with excellence: availability, SLAs, DR | Operations-critical: board-level on outages, not on features | Core banking ledger, payments (SWIFT/MEPS+), market-data feeds, settlement | ITIL discipline; change control; "run-the-bank" budget protected |
| **Turnaround** | Low | High | Nurture: pilots, MVPs, architecture runway | Senior sponsorship, funding gates, active review | AI/LLM client assistants, API banking, new digital onboarding | Innovation governance: stage-gates, separate funding, fail-fast |
| **Strategic** | High | High | Co-create with business; strategic architecture | CEO/CIO partnership; premium investment | Digital client platform at a digital bank; firm-wide risk platform post-2008 regulation | Full portfolio governance; business case discipline (see [Business Case Development](business_case_development_guide.md)) |

### 5.3 Using the Grid

- **Classify the portfolio, then govern by class.** A single governance regime for all four quadrants either starves innovation (factory rules applied to turnaround) or endangers operations (startup rules applied to factory). McFarlan's point was precisely that firms manage IT portfolios *as if all IT were the same* — and fail.
- **The grid is dynamic.** Today's turnaround is tomorrow's strategic (a bank's mobile channel) and eventually factory (payments rails). The portfolio review should explicitly move projects between quadrants over time.
- **Pair with the demand-side view.** Practitioners often combine the grid with Ward and Peppard's application-portfolio classification (strategic / high potential / key operational / support, with age/health of systems) — a post-2000 extension, flagged as such. The grid tells you *importance*; Ward–Peppard adds *condition* (legacy vs modern) to drive renewal decisions, which connects directly to [Monolith to Microservices](../technology/monolith_to_microservices_guide.md) thinking.
- **It drives sourcing posture.** Support and Factory systems are candidates for buy/outsource; Strategic systems demand in-house control. Which leads directly to Section 6.

### 5.4 Building and Running the Portfolio

The grid is a governance instrument, not an academic exercise. The operating cycle:

1. **Inventory.** Every system and initiative goes on the grid, with cost, age, and health attached (pairing with Ward–Peppard-style condition scoring gives both *importance* and *condition*).
2. **Classify jointly.** Business and IT score current and future impact *together* — the classification conversation is itself an alignment exercise (Section 2). If the business says "strategic" and IT says "factory" for the same system, that disagreement *is* the strategy conversation.
3. **Govern by class.** Funding rules, governance depth, and metrics differ per quadrant: stage-gates and fail-fast for Turnaround; premium co-investment for Strategic; reliability and cost for Factory; cheapest-viable for Support.
4. **Reclassify on a cadence.** Annual reclassification is where the strategy is tested: today's Turnaround matures into Strategic (a successful AI platform) or dies; Strategic systems mature into Factory (a client portal becomes plumbing); Factory systems get retired to shrink the Support/Factor drag.
5. **Link demand and supply.** The portfolio is the demand side; delivery capacity and sourcing (Section 6) are the supply side. A portfolio without a capacity constraint is a wish-list; the investment committee (Section 4) is where demand meets supply.

---

## 6. The Sourcing: Build, Buy, Outsource

### 6.1 The Make–Buy–Rent Decision — verified framing

Sourcing is the IT-strategy decision of *where capability comes from*: **build** it in-house (own development), **buy** it (licensed products, SaaS, market data, packaged platforms), or **outsource** it (a third party builds and/or runs it for you — managed services, cloud as utility, business-process outsourcing). The "build/buy/outsource" trichotomy is the standard framing across the practitioner and academic literature (Make-or-Buy is the classic operations/strategy question applied to IT; no single canonical "author" owns the trichotomy — it is textbook-standard framing, used here as such). The [Vendor Management](vendor_management_guide.md) guide covers what happens *after* the sourcing decision — contract, governance, relationship, exit — and [FinOps](../technology/finops_guide.md) covers the cost-governance of the cloud/buy side.

The decision rule comes straight from strategy theory: **build what differentiates you; buy or outsource what does not.** More precisely:

1. **Is the capability strategic?** (McFarlan's grid, Section 5: Strategic and Turnaround systems are candidates for in-house control; Support and Factory are candidates for buy/outsource.)
2. **Is there a market?** If an excellent product/SaaS exists and your needs are standard, buying beats building on every dimension: cost, speed, upgrade path, talent.
3. **Is the capability a source of differentiation?** If your edge is a proprietary algorithm, a unique client experience, or an un-purchasable integration (e.g. a bank's pricing engine, its client-relationship data model), you must build or at least own the core.
4. **Can you control risk externally?** Regulation, data protection, and operational resilience may force in-house operation regardless of economics — a bank cannot outsource its regulatory obligations (see [Vendor Management](vendor_management_guide.md) on third-party risk management and DORA-style oversight).

Carr's commoditisation argument (Section 9) *strengthens* the buy side: if IT infrastructure is a utility, treat it as one. What Carr's critics correctly add: the *application* of IT — data, algorithms, client relationships, process design — is not a utility, and that is precisely what you must build or co-create.

### 6.2 The Sourcing Table

| Option | Definition | Best when | Risks | Governance needs | Banking examples |
|---|---|---|---|---|---|
| **Build (in-house)** | Own development of software and platforms | Capability is differentiating; no market product fits; IP and control matter; regulatory constraints | Cost and time overruns; talent scarcity; legacy debt if unmaintained; long delivery cycles | Delivery governance (SDLC, architecture review); value tracking (see Section 8); portfolio oversight | Proprietary pricing/risk models; client portal; algorithmic execution; the "secret sauce" layers |
| **Buy (licensed product / SaaS)** | Acquire packaged software or cloud services | Capability is standard; mature market exists; speed to market; predictable cost | Vendor lock-in; configuration limits; upgrade cycles; per-license cost growth; data sovereignty | Procurement discipline; contract/licence management; integration architecture; vendor management (see [Vendor Management](vendor_management_guide.md)) | Core banking packages (e.g. Temenos-type stacks), market-data feeds (Bloomberg/Refinitiv), CRM (Salesforce), Murex/Calypso-type trading platforms, SaaS HR |
| **Outsource (managed services / cloud)** | Third party builds and/or runs capability (ITO: infrastructure, applications, BPO) | Utility capability; scale economics; scarce skills; cost variability (cloud); 24×7 operations | Loss of control and skills; hidden costs; resilience dependence; concentration risk; exit complexity | Service-level management (ITIL); third-party risk management; exit plans; FinOps for cloud cost control | Data-centre hosting, network, service desk, application maintenance, public-cloud platforms, AML/KYC processing utilities |
| **Hybrid (co-sourcing)** | Blend: vendor builds/operates, you retain architecture and accountability | You want speed/scale without losing control of direction | Blurred accountability; interface disputes; governance overhead | Clear RACI at the boundary; architecture ownership retained in-house | Banks' classic model: in-house architects + vendor-run application stacks + cloud under the hood |

### 6.3 Sourcing as a Portfolio Decision

- **It is not one decision but a portfolio of decisions.** The same bank builds its pricing engine, buys Murex/Calypso-type trading systems, and outsources data centres. The sourcing posture per system should follow its quadrant in the McFarlan grid and its place in the operating model (Section 3): unification domains need fewer, more standardised buys; coordination domains need best-of-breed buys wired together; strategic domains need build.
- **Exit strategy before entry.** Every sourcing decision should include the exit: data portability, contract break clauses, cloud repatriation cost, knowledge transfer. Banks learned this the hard way in the 2000s outsourcing wave.
- **The cloud changed the arithmetic.** Cloud turns *buy* into *rent-by-the-drink*: no capital, elastic capacity, but continuous operating cost and FinOps discipline — see [FinOps](../technology/finops_guide.md). Regulated firms add resilience and sovereignty conditions (where data lives, exit provisions, DORA-class oversight).
- **Carr's shadow.** Sourcing is where the IT-doesn't-matter debate (Section 9) becomes operational: commodity infrastructure → outsource to a utility; differentiated capability → build and protect. The debate is not academic; it is the sourcing strategy.

### 6.4 The Sourcing Lifecycle

The decision in Section 6.1 is the first step of a lifecycle, and the later steps are where value is won or lost:

1. **Sourcing decision** — the build/buy/outsource choice per the rules of 6.1, recorded with its rationale.
2. **Procurement** — RFP, evaluation, negotiation, contract: the vendor-management machinery of [Vendor Management](vendor_management_guide.md), including exit clauses *before* signature.
3. **Transition** — knowledge transfer, data migration, service stabilisation. Transitions are systematically underestimated: the double-run period, the skill drain, the day-one incidents. Banks additionally run third-party risk assessments (DORA-class and MAS outsourcing guidelines in Singapore) before transition begins.
4. **Operate and govern** — SLA management (ITIL), contract and licence management, third-party risk monitoring, cloud cost control ([FinOps](../technology/finops_guide.md)), and the relationship governance of [Vendor Management](vendor_management_guide.md).
5. **Review and exit/renew** — benchmarking against the market, exercising the exit plan or renewing deliberately. An outsourcing relationship without a review cadence is a slowly accreting dependency.

The two most common failure points are transition (under-resourced) and exit (unplanned). Both are fixed at step 1: every sourcing decision should carry an exit plan as a first-class deliverable.

---

## 7. The Digital Transformation

### 7.1 What Digital Transformation Is

**Digital transformation is the organisation-wide change in business model, operating model, and customer experience driven by digital technology** — not a set of IT projects but a *change programme* touching strategy, process, people, and technology simultaneously. It is where IT strategy meets [Organizational Behavior](organizational_behavior_guide.md): the technology is the easy 20%; the change management is the hard 80%.

### 7.2 The Digital Transformation Office (DTO) — verified with a flag

The **DTO (Digital Transformation Office)** is the organisational pattern of standing up a dedicated team — typically reporting to the CEO/CDO or CIO — chartered to drive, coordinate, and de-silo the transformation agenda: transformation strategy, portfolio and funding, cross-functional programme management, capability building, and culture change. *Verified: the DTO is real and widely described in the practitioner press (e.g. CIO/consultancy playbooks) as the governance vehicle for transformation. Flag: unlike SAM, COBIT, or the Weill–Ross models, the DTO is not a single codified framework with one canonical author — it is an evolving organisational pattern (variants: Office of the CDO, Transformation Management Office, Digital Business Office). Attribution of a specific "creator" would be fabrication, so none is given; treat the description as practitioner consensus, not canon.*

A DTO typically owns:

- **Transformation strategy** — the bridge between the digital strategy and the portfolio (which initiatives, in what order, with what business case — see [Business Case Development](business_case_development_guide.md)).
- **Operating-model change** — Section 3 models made real: standardising and integrating processes as the digital target state requires.
- **Portfolio and funding** — transformation funding ring-fenced from run-the-bank, governed with stage gates (McFarlan's Turnaround quadrant, governed properly).
- **Capability and culture** — skills, change management, communications, metrics; the organizational-behavior half of the programme.
- **Delivery assurance** — removing blockers across business and IT; the "office" sits above delivery rather than doing the delivery.

In banks, DTO-type structures became common from the mid-2010s as incumbent institutions responded to digital-only challengers (see the [Singapore digital-bank guides](../banking/gxs_bank_guide.md) and [Maribank](../banking/maribank_guide.md) for the challenger reality) and to client expectations of API, mobile, and instant services. The pattern also shows up as the *"Digital Business Office"* in Gartner's terminology (a Gartner-coined label for the same organisational answer) — flagged for attribution.

### 7.3 The Transformation Table

| Dimension | Traditional IT (run-the-bank) | Digital transformation (change-the-bank) | Banking example |
|---|---|---|---|
| **Goal** | Stability, efficiency, control | Growth, experience, new business models | Keep payments stable vs launch instant cross-border APIs |
| **Horizon** | Continuous operations, annual cycles | Programme-based, 2–5 years, then continuous | Multi-year core modernisation vs agile client onboarding |
| **Budget** | Run cost, protected and optimised | Investment, funded by business case, stage-gated | Run-the-bank 70% / change 30% split rebalanced |
| **Governance** | ITIL, change control, SLA management | Transformation office, stage gates, benefit tracking | DTO portfolio reviews |
| **Delivery** | Waterfall, controlled releases | Agile/DevOps, MVP, continuous delivery | [Monolith to Microservices](../technology/monolith_to_microservices_guide.md) modernisation squads |
| **Technology focus** | Systems of record (ledgers, risk) | Systems of engagement + intelligence (APIs, data, AI) | Client portals, AI assistants, event streaming |
| **Skills** | Engineering, operations | Product management, design, data science, change leadership | Hiring product owners and data engineers |
| **Success measure** | Uptime, SLA, cost per transaction | Revenue, NPS, time-to-market, digitisation rate | % of onboarding fully digital |

### 7.4 Transformation Pitfalls

- **Treating transformation as an IT project.** If the business does not change its processes and the operating model stays untouched, "transformation" delivers new technology on old ways of working — the classic digitised-paperwork failure. This is why the DTO reports to the CEO side, not deep inside IT.
- **Funding transformation from run-the-bank budget.** When the run budget is squeezed, transformation is the first casualty — the reason top performers ring-fence change funding and protect the factory (Section 5).
- **Ignoring the organisation.** The change-management half is covered in depth in [Organizational Behavior](organizational_behavior_guide.md); the leadership-development half in the [360 management](360_management_guide.md) and [Managing Up, Down, Sideways](managing_up_down_sideways_guide.md) guides.
- **Confusing digital with AI.** AI is a (large) ingredient, not the whole dish. The [Enterprise AI Gateway](../technology/enterprise_ai_gateway_guide.md) guide covers the platform reality of AI enablement; the [Data Architect Skill Gaps](../technology/data_architect_skillgaps_guide.md) guide covers the skills the transformation will need.

### 7.5 A Transformation Roadmap in Phases

Transformations fail in the sequencing as often as in the strategy. A defensible phase structure:

- **Phase 0 — Foundation (0–6 months).** DTO chartered with CEO sponsorship; the transformation portfolio defined (which initiatives, which business cases); the operating-model statement (Section 3) agreed; baseline scorecard metrics (Section 8) captured. Nothing is built; everything is decided.
- **Phase 1 — Quick wins (6–18 months).** Client-facing digitisation: digital onboarding, API banking, client portals — the Turnaround quadrant (Section 5) governed with stage gates. These build delivery capability and political capital.
- **Phase 2 — Core modernisation (18–48 months).** The hard part: platform consolidation, data unification (one client view, one risk book), legacy retirement. Funded by the credibility earned in Phase 1. See [Monolith to Microservices](../technology/monolith_to_microservices_guide.md) for the technical shape of this phase.
- **Phase 3 — Intelligent enterprise (48+ months).** AI/ML at scale, continuous change culture, the DTO dissolved into the line organisation — because transformation has become how the firm operates.

Sequencing rule: never start Phase 2 before Phase 1 has built the delivery capability, and never let Phase 3 start while Phase 2's data foundations are unfinished — AI on un-unified data is the most expensive way to learn the alignment lesson.

---

## 8. The IT Value

### 8.1 The Value Problem

IT value is the hardest measurement problem in the discipline, for three structural reasons: IT's benefits are **indirect** (a risk platform's value shows up as fewer losses and lower capital, not revenue), **deferred** (benefits land years after investment), and **shared** (whose P&L owns the benefit of one shared client data platform?). The [Business Case Development](business_case_development_guide.md) guide covers the *ex-ante* machinery (justifying an investment before it is made); this section covers the *ex-post* machinery — measuring and managing value after, continuously.

### 8.2 The Scorecard — verified (Kaplan & Norton, 1992)

The anchor framework is the **Balanced Scorecard** of Robert S. Kaplan and David P. Norton, introduced in *"The Balanced Scorecard — Measures That Drive Performance,"* **Harvard Business Review, January–February 1992** (developed from a 1990 multi-company research project, the Nolan Norton Institute's 1991 report; verified against HBR and HBS sources). The BSC's core move: a single financial measure set is insufficient; performance must be viewed through **four perspectives** — *financial, customer, internal process, learning & growth* — linked by cause-and-effect (learning & growth → internal process → customer → financial).

IT strategy applies the BSC by building an **IT scorecard**: the same four perspectives translated into IT-specific measures, plus IT-specific additions such as the "IT governance" and "future orientation" views in later IT-BSC adaptations (Van Grembergen and De Haes, 2000s — a practitioner adaptation, flagged as such; the 1992 Kaplan–Norton original is the canon). The purpose is not to measure IT for its own sake but to keep the alignment question (Section 1) *measurable*: every scorecard line should trace back to a business outcome.

### 8.3 The Value Table

| BSC perspective | Sample IT metrics | What it tells you | Connected discipline |
|---|---|---|---|
| **Financial** | IT cost per transaction; IT spend as % of revenue; cost per user; unit cost of a payment/loan/statement; cloud unit economics | Is IT cheap enough *per unit of output*? Efficiency and cost control | [FinOps](../technology/finops_guide.md); [Business Case Development](business_case_development_guide.md) benefit realisation |
| **Customer / stakeholder** | System availability (99.9x%); MTTR; SLA attainment; internal-customer NPS; regulator/audit findings; time-to-respond to requests | Is IT reliable and responsive for the business and regulators? (The Factory promise, Section 5) | ITIL service management; [Vendor Management](vendor_management_guide.md) SLA governance |
| **Internal process** | Time-to-market for new products; release frequency; change-failure rate; % of processes digitised; integration latency (data available T+0 vs T+1) | Is the IT *delivery machine* fast and safe? | Agile/DevOps practice; [Monolith to Microservices](../technology/monolith_to_microservices_guide.md) |
| **Learning & growth** | % of staff on modern skills; attrition of engineers; cloud/AI capability maturity; architecture obsolescence index (% systems on supported versions) | Can the organisation sustain the strategy? The future capability stock | [Data Architect Skill Gaps](../technology/data_architect_skillgaps_guide.md); [Organizational Behavior](organizational_behavior_guide.md) |

### 8.4 Measuring Value Honestly

- **Value is a ratio, not an amount.** The same €10M platform is cheap for a global bank's risk book and ruinous for a regional boutique. Anchor every measure to the operating model (Section 3): a unification bank should show *lower unit costs at the same integration level*; a coordination bank should show *higher cross-sell and one-client-view coverage*.
- **Attribution is a governance act, not a measurement act.** Someone must *own* the benefit: the business sponsor signs the business case and answers for realisation (see [Business Case Development](business_case_development_guide.md)); IT is accountable for delivery and unit economics, not for business P&L it cannot control.
- **Track both run and change.** A bank's IT value story has two halves: run-the-bank (unit cost, availability — get cheaper, stay stable) and change-the-bank (time-to-market, digitisation rate — get faster, ship value). Reporting only one half produces either cost-cutting theatre or innovation theatre.
- **Carr is the stress test.** The IT-doesn't-matter debate (Section 9) is, at bottom, a value argument: if IT is a commodity, its value is captured by customers and vendors, not shareholders. The value section is where you argue back with evidence.

### 8.5 Leading vs Lagging Indicators and Benefit Realisation

Two disciplines keep the scorecard honest:

- **Mix leading and lagging.** Lagging indicators (cost per transaction, availability) confirm the past; leading indicators (release frequency, architecture obsolescence, skills coverage) predict the future. A scorecard of only lagging indicators is a rear-view mirror — you learn the outcome after it is unchangeable.
- **Close the benefit-realisation loop.** Business-case targets (ex-ante, from [Business Case Development](business_case_development_guide.md)) must meet post-implementation review (ex-post, 90 days after go-live) and then benefit *tracking* to the P&L, with lessons fed back into the next business case. When the loop is not closed, the value discipline decays silently: business cases become approval theatre, and Section 8 becomes a reporting exercise nobody reads.

The scorecard's purpose is not measurement; it is the alignment question made quantitative. Every line should be traceable to a business outcome — if a metric cannot be traced, drop it.

---

## 9. The IT-Doesn't-Matter Debate

### 9.1 Carr's Thesis — verified (2003)

In **"IT Doesn't Matter," *Harvard Business Review*, May 2003, pp. 41–49**, Nicholas G. Carr (then HBR's editor-at-large) argued that **information technology had become a commodity infrastructure — like railroads, electricity, and the telegraph — and therefore could no longer be a source of sustainable competitive advantage.** *Verified: HBR archive confirms the May 2003 issue; Carr's own site confirms May 2003; the article's pagination 41–49 is widely cited. The 2003 attribution is rock solid.*

Carr's argument ran:

1. **Scarcity creates advantage.** A resource delivers competitive advantage only when it is scarce. When every competitor can buy the same thing, advantage disappears.
2. **IT is now ubiquitous and cheap.** By 2003, IT had passed from proprietary rarity (the mainframe era, where IT genuinely differentiated) to commodity ubiquity (standardised hardware, packaged software, internet). Its price-performance kept improving while its differentiation potential fell.
3. **Therefore the strategic opportunities are gone.** The "strategic" era of IT is over; the *risks* remain (outages, security, regulatory failure). Carr's prescriptions: **spend less** (follow the pack, don't lead), **follow, don't innovate** (wait for standards, then adopt), **focus on vulnerabilities, not opportunities** (the downside is what remains).

The article provoked one of the fiercest debates in business-technology history — 14 letters to the editor ran in HBR, and responses came from Intel's Craig Barrett (famously dismissive), Microsoft, SAP, and a decade of rebuttals. Carr expanded the thesis into the book *Does IT Matter?* (Harvard Business School Press, 2004).

### 9.2 The Counter-Arguments

The strongest rebuttals, which have aged well:

- **IT ≠ infrastructure.** Andrew McAfee (HBR, 2006, "Mastering the Three Worlds of Information Technology"): Carr conflates IT *infrastructure* (commodity) with IT *applications* and *data* (potentially differentiating). A bank's pricing models, client data, and algorithms are not commodities.
- **Innovation moved up the stack.** Even if hardware and connectivity are utilities, the value migrated to software, data, and now AI — layers Carr's 2003 argument did not anticipate. In 2003 there was no cloud, no iPhone, no LLM; the subsequent 20 years of digital winners (Amazon, the platform economy, digital banks) are a standing refutation of "the opportunities are gone."
- **Advantage is relative, not absolute.** Even a commodity can advantage a firm that *applies* it better — the same electricity that powers every factory powers the one that reorganises its processes around it. The advantage is in use, not possession.
- **The risk argument cuts both ways.** Carr said focus on vulnerabilities; the 2010s showed vulnerabilities *are* strategic (cyber attacks, data breaches destroy value) — so IT "doesn't matter" as opportunity but "matters enormously" as risk. A bank's resilience investment is not commoditised.

The balanced verdict most practitioners accept: **Carr was right about the infrastructure layer (buy it as a utility, don't build a data centre as strategy) and wrong about the application/data layer (build what differentiates — which is exactly the sourcing rule in Section 6).** The debate's permanent contribution is discipline: it forces every IT strategy to answer "what here is genuinely differentiating, and what is commodity?"

### 9.3 The Debate Table

| Position | Core claim | Supporting evidence | Weakness | Modern verdict |
|---|---|---|---|---|
| **Carr (2003)** | IT is commodity infrastructure; no sustainable advantage; spend less, follow, mind the risks | Commoditisation of hardware/software; ubiquity; price-performance curves; analogy to railroads/electricity | Conflates infrastructure with applications, data, and use; ignores that advantage is relative and in application | Right about infrastructure — buy it as a utility (Section 6); wrong about the stack above it |
| **McAfee (2006)** | Separate infrastructure, applications, data: only infrastructure is commodity | Examples of firms differentiating on proprietary data and innovative applications | Data/applications can also be copied over time | The standard refutation; underpins the build/buy rule |
| **Industry rebuttals (2003–04)** | IT still matters; vendors and CIOs defended strategic IT | Case studies of IT-driven advantage | Often self-interested (vendors defending budgets); anecdotal | Partly vindicated by the digital era, but Carr's cost discipline stuck |
| **Synthesis (this guide)** | Differentiate on data, algorithms, integration, and use; commoditise everything else | The sourcing rule (Section 6), the portfolio rule (Section 5), the value rule (Section 8) | Requires judgement about what is truly differentiating | The working consensus for two decades of IT strategy |

### 9.4 The Debate's Aftermath: Cloud, AI, and the 2020s

Two technology waves since 2003 have each re-run the debate at a different layer:

- **Cloud (2010s)** vindicated Carr at the infrastructure layer: compute became a metered utility bought by the drink, and the data centre went from asset to subscription. The differentiation did not disappear — it migrated up the stack to data, algorithms, and application. The firms that "followed" (Carr's advice) on infrastructure and "led" (his critics' advice) on data and AI won the decade.
- **AI (2020s)** is the strongest Competitive Potential signal since the 1990s — and already shows the same commoditisation curve Carr described: foundation models are becoming utility, while the advantage sits in proprietary data, fine-tuned application, and process integration. The [Enterprise AI Gateway](../technology/enterprise_ai_gateway_guide.md) guide is the architectural expression of exactly this: buy the commodity model layer, build the differentiating data and application layer.

The debate's permanent legacy is a question every IT strategy must now answer on every line: *commodity or differentiating?* This guide institutionalises that question in the portfolio (Section 5), the sourcing rule (Section 6), and the value scorecard (Section 8). Carr lost the argument about the stack's upper layers and won the argument about its base — and the strategy discipline is the richer for both outcomes.

---

## 10. The Worked Example: IT Strategy for a Bank

### 10.1 The Scenario — a CACIB-style bank

To make the discipline concrete, we design an IT strategy for a bank shaped like **Crédit Agricole Corporate and Investment Bank (CACIB)** — the familiar context: the CIB arm of a large cooperative banking group, headquartered in Paris, with a significant Asia hub in Singapore. (For the system-level reality behind this scenario, see [Crédit Agricole Software Systems](../banking/credit_agricole_software_systems_guide.md), [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md), and [HSBC Software Systems](../banking/hsbc_software_systems_guide.md); the design below is a strategy exercise, not a description of CACIB's actual programmes.)

**Business context (from the business strategy, per [Strategic Management](strategic_management_guide.md)):**

- **Business model:** corporate banking (loans, trade finance, cash management), capital markets (rates, FX, credit, structured products), securities services, and treasury. Clients are large corporates and financial institutions; the franchise is relationship-led and advisory-led, not a retail mass market.
- **Strategic imperatives:** (1) deepen the corporate-client franchise in Asia; (2) improve ROE under Basel III/IV capital and funding constraints; (3) meet tightening regulation — EU DORA, MAS requirements in Singapore, cross-border data rules; (4) defend against fintech and digital-only entrants on the transactional/API layer; (5) harness AI for pricing, risk, and client service.
- **IT estate reality (the familiar pain):** decades of acquired and home-grown systems; a patchwork of vendor platforms for trading and risk; multiple client databases across desks; heavy regulatory reporting built by hand; data centres and legacy integration layers; skills concentrated in legacy technologies; run-the-bank consuming most of the IT budget. The classic "estate problem" covered in [Monolith to Microservices](../technology/monolith_to_microservices_guide.md).

### 10.2 The Strategy Design

**Step 1 — Alignment (Section 2).** Run SAM explicitly. Diagnosis: the bank currently operates *Strategy Execution* everywhere — IT implements whatever the desks ask for; there is no Competitive Potential anywhere; the IT strategy domain is not articulated. Design choice: keep **Strategy Execution** for core banking, payments, and regulatory reporting (stability is the business requirement); adopt **Technology Potential** for the trading and risk platform strategy (choose best-of-breed technology to support the fixed business strategy); adopt **Competitive Potential** for the client-facing and data layers (API banking, client insights, AI) where emerging capability can reshape the client franchise. *The alignment table (Section 2.2) becomes the first page of the IT strategy document*, and a standing business–IT council is chartered to re-run it quarterly — the "dynamic administrative process" Henderson and Venkatraman insisted on.

**Step 2 — Operating model (Section 3).** Declare the operating model per domain, before any architecture decision:

| Domain | Operating model | Rationale |
|---|---|---|
| Client-facing (CRM, onboarding, portals, APIs) | **Coordination** | One client view across desks is the CIB differentiator; desks keep their own processes |
| Trading & risk platforms | **Coordination** (data) + **Unification** (risk/finance) | Shared market and risk data across desks; one risk book and one regulatory report — regulator-mandated unification |
| Payments, cash management, back-office | **Unification** | Scale and control; one process end-to-end |
| Country operations (Singapore, HK, NY, London...) | **Replication** | Standard templates for local regulatory and back-office needs |

**Step 3 — Governance (Section 4).** COBIT for the control layer (goals cascade from board risk appetite; the five domains as the audit map), ITIL for service operations, TOGAF's ADM for the target architecture that encodes the Step-2 operating model. Decision rights via the Weill–Ross matrix — a practical resolution for a CIB:

| Decision | Archetype chosen | Body |
|---|---|---|
| IT principles | Business monarchy (with CIO) | Group executive committee |
| IT architecture | IT monarchy | Enterprise architecture board (TOGAF-governed) |
| IT infrastructure | IT monarchy | Infrastructure & cloud council |
| Business application needs | Federal / IT duopoly | Business–IT investment committee (one per business line + group) |
| IT investment & prioritisation | Federal | Group investment committee with the CFO/CIO — feeding the business-case machinery in [Business Case Development](business_case_development_guide.md) |

**Step 4 — Portfolio (Section 5).** Classify the estate on McFarlan's grid and govern by class:

- **Strategic:** client data platform (one client view), API banking layer, AI/risk analytics — premium funding, business–IT co-creation.
- **Turnaround:** AI/LLM client assistants, new digital onboarding, cloud-native data platform — stage-gated innovation funding, fail-fast.
- **Factory:** core payments and settlement, market-data feeds, core risk engines — protect the run budget, ITIL discipline, no innovation theatre.
- **Support:** HR, expenses, office systems — cheapest viable sourcing.

**Step 5 — Sourcing (Section 6).** Build what differentiates; buy what doesn't: **build** the client data platform, pricing and risk models, API layer, and AI integration (the competitive potential); **buy** trading platforms (Murex/Calypso-class), market data (Bloomberg/Refinitiv-class), core banking where a package fits; **outsource** data centres, network, service desk, and run the commodity layer in the cloud with FinOps discipline ([FinOps](../technology/finops_guide.md)); every contract carries an exit plan and third-party risk management per [Vendor Management](vendor_management_guide.md) and DORA-class oversight.

**Step 6 — Transformation (Section 7).** Stand up a **DTO** reporting to the CEO/CIO jointly, ring-fenced change funding, and an operating-model change programme (the hard 80%): process redesign for one-client-view, product management capabilities, data engineering skills — the [Organizational Behavior](organizational_behavior_guide.md) and [Data Architect Skill Gaps](../technology/data_architect_skillgaps_guide.md) agendas made explicit. The DTO runs the Turnaround and Strategic quadrants; IT operations run the Factory.

**Step 7 — Value (Section 8).** An IT scorecard per the 1992 BSC:

| Perspective | 3-year targets (illustrative) |
|---|---|
| Financial | IT cost per transaction −25%; run budget as % of revenue −5 pts; cloud unit cost per MIPS/transaction tracked monthly |
| Customer/stakeholder | 99.99% availability on client-facing platforms; zero critical regulatory findings; internal NPS ≥ 40 |
| Internal process | Time-to-market for new product features −50%; release frequency ×4; % processes digitised 40%→80%; T+0 data coverage for major books |
| Learning & growth | 60% of engineers on modern skills; architecture obsolescence index < 10%; AI/cloud capability maturity at level 3+ |

**Step 8 — The Carr stress test (Section 9).** For each portfolio line, answer: *is this differentiating or commodity?* The estate's commodity layers (infrastructure, desktop, connectivity) are bought and outsourced — Carr's prescription adopted. The differentiating layers (client data, pricing models, AI, integration) are built and protected — Carr's critics' prescription adopted. The strategy is explicitly a synthesis.

### 10.3 The Lessons

1. **Alignment first, always.** Every subsequent decision in this design — operating model, governance, portfolio, sourcing — traces back to the SAM diagnosis. When a desk demands a bespoke system that fragments the client view, the answer comes from the alignment and operating model, not from IT opinion. The alignment question is the strategy.
2. **The operating model is the silent contract.** Most bank IT failures are operating-model failures wearing technology costumes: interfaces multiplied because nobody declared coordination; costs exploded because nobody declared unification for risk/finance. Declare it, write it down, fund the interfaces.
3. **Governance is where strategy lives or dies.** Without the Weill–Ross decision matrix, the best SAM and operating-model analysis evaporates in the first investment committee. Governance is boring until it is absent.
4. **Portfolio discipline protects both halves.** The Factory must be protected from innovation theatre and the Turnaround from factory bureaucracy. One governance regime for both is the classic failure.
5. **Sourcing is a portfolio decision, not a fashion.** "We outsource" and "we build everything" are both strategies without portfolios behind them. The build/buy line follows differentiation, not ideology — Carr's debate made operational.
6. **Transformation is an operating-model and people change.** A new client portal on the old client-data mess is a new coat of paint. The DTO exists to change processes, skills, and governance — the organizational-behavior half.
7. **Measure value or lose the argument.** An IT organisation that cannot show unit-cost, availability, time-to-market, and capability metrics will be governed by anecdote and budget cuts. The scorecard is the political instrument of the CIO.
8. **The banking context is a constraint set, not an excuse.** Regulation, legacy, and risk are real — but they are parameters the strategy optimises within, not reasons to skip the discipline. The same framework that designs a CACIB-style estate also designs a challenger bank; the answers differ, the method does not.

### 10.4 The One-Page IT Strategy Statement (illustrative)

The entire design of Section 10.2, compressed to one page — the artefact the CIO actually presents:

> **Vision:** IT is the differentiator of the client franchise and the custodian of the bank's operational integrity.
>
> **Alignment (SAM):** Strategy Execution for core banking, payments, and regulatory reporting; Technology Potential for trading and risk platforms; Competitive Potential for the client-facing and data layers.
>
> **Operating model:** Coordination for client-facing (one client view across desks); Unification for risk, finance, and payments; Replication for country operations.
>
> **Governance:** Federal investment committee (business + IT) for investment and application needs; IT monarchy for architecture and infrastructure; COBIT-governed control layer; ITIL-run operations; TOGAF-shaped target architecture.
>
> **Portfolio (McFarlan):** Protect the Factory (payments, market data, risk engines); fund the Turnaround (AI, digital onboarding) with stage gates; co-create the Strategic (client data platform, API layer).
>
> **Sourcing:** Build what differentiates (client data, pricing and risk models, AI integration); buy what is standard (trading platforms, market data); outsource the utility layer (infrastructure, cloud) with exit plans and DORA-class oversight.
>
> **Value (BSC):** Unit cost per transaction −25%; 99.99% availability on client-facing platforms; time-to-market −50%; 60% of engineers on modern skills — reported quarterly to the board.

### 10.5 What Could Go Wrong

Each failure mode is a process failure, not a model failure — the models are sound, the discipline is the process:

- **Alignment theatre** — the SAM analysis exists as slides and the quarterly re-run never happens. Fix: put the re-run on the committee calendar with teeth.
- **Operating-model drift** — coordination declared, diversification practised; interfaces multiply because nobody funded them. Fix: an annual operating-model audit against the actual estate.
- **Governance capture** — the investment committee rubber-stamps desk asks in the name of "business partnership". Fix: the decision-rights matrix, published and enforced.
- **Portfolio misclassification** — a Strategic system labelled Factory to avoid scrutiny, or a Turnaround labelled Strategic to hoard funding. Fix: joint business–IT classification, reclassified annually.
- **Sourcing lock-in** — buy decisions without exit plans become 20-year dependencies. Fix: exit plan as a first-class deliverable at signature (Section 6.4).
- **Transformation as IT project** — the DTO reports into IT, the business does not change, and the new portals run on the old client-data mess. Fix: CEO sponsorship and the operating-model change programme (Section 7).
- **Value amnesia** — the scorecard is abandoned after year one. Fix: the benefit-realisation loop closed quarterly (Section 8.5), and the scorecard tied to the committee calendar.

---

## 11. The Summary: One Page

### 11.1 The One-Page Summary

| # | Discipline element | Canonical anchor (verified) | One-line essence | Bank application |
|---|---|---|---|---|
| 1 | **IT strategy overview** | Alignment as definition (Henderson–Venkatraman, 1993) | IT strategy *is* the alignment question, made operational | What is IT for, and who decides? |
| 2 | **Alignment** | SAM, IBM Systems Journal 32(1), 1993 | Four domains, four alignment perspectives; alignment is a managed process, not a state | Strategy Execution for core; Competitive Potential for client/data |
| 3 | **Operating models** | Weill & Ross, 2004 | Standardisation × integration → diversification, coordination, unification, replication | Coordination client-facing; unification risk/finance; replication countries |
| 4 | **IT governance** | COBIT (ISACA, 1996→2019); ITIL (CCTA→Axelos); TOGAF (Open Group, 1995); Weill–Ross decision rights (2004) | COBIT governs, ITIL runs, TOGAF architects; five decisions × six archetypes | Decision-rights matrix; goals cascade; ADM target architecture |
| 5 | **Portfolio** | McFarlan strategic grid, 1984 (with McKenney 1983) | Support / Factory / Turnaround / Strategic — govern by class | Protect the factory; fund the turnaround; co-create the strategic |
| 6 | **Sourcing** | Build/buy/outsource (textbook framing) | Build what differentiates; buy/outsource what does not | Build client data & AI; buy trading & data; outsource infrastructure |
| 7 | **Digital transformation** | DTO pattern (practitioner consensus; not a single canon) | Technology 20%, change 80% | DTO + ring-fenced funding + operating-model change |
| 8 | **IT value** | Balanced Scorecard (Kaplan & Norton, HBR, 1992) | Four perspectives; value is a ratio, attribution is governance | IT scorecard: unit cost, availability, time-to-market, skills |
| 9 | **IT-doesn't-matter debate** | Carr, HBR, May 2003 | Right about infrastructure, wrong about data and application | Commoditise the commodity; protect the differentiator |

### 11.2 The Final Word: The Alignment Question

Every element of this guide — the four domains of SAM, the four operating models, the five governance decisions, the four portfolio quadrants, the build/buy/outsource rule, the DTO, the scorecard, the Carr debate — is machinery for answering the same question, which is also the definition of the discipline:

> **What is IT for, and who decides?**

- *What is IT for?* The alignment answer: to execute the business strategy where it depends on information, and to *change* the business strategy where information creates new possibilities. Both directions. A strategy that only implements will be out-innovated; a strategy that only dreams will be out-executed.
- *Who decides?* The governance answer: the decision-rights matrix — business monarchy for principles, IT monarchy for architecture, federal for investment — written down, communicated, and enforced. Alignment is not a document; it is a standing decision process.

Henderson and Venkatraman said it in 1993: value from IT comes from alignment, and alignment must be continuously administered. Carr said it in 2003 from the other side: treat the commodity as commodity. The discipline of IT strategy is what happens when you take both seriously at once. Everything else — the models, the tables, the frameworks — is scaffolding. The alignment question is the building.

---

## Glossary

| Term | Definition |
|---|---|
| **IT strategy** | The plan for how an organisation's IT capability (systems, data, infrastructure, people, governance) will be created and used to achieve business goals; its essence is alignment between business and IT strategy. |
| **Alignment** | The degree to which the IT strategy and the business strategy are mutually consistent and reinforcing — the core objective of IT strategy (Henderson–Venkatraman). |
| **Strategic Alignment Model (SAM)** | Henderson and Venkatraman's 1993 framework: four domains (business strategy, IT strategy, organizational infrastructure & processes, IT infrastructure & processes) whose alignment — via four perspectives — determines IT value. |
| **Henderson** | John C. Henderson, co-author of the Strategic Alignment Model (IBM Systems Journal, 1993); MIT/University Professor. |
| **Venkatraman** | N. Venkatraman, co-author of SAM (1993); Boston University professor, later co-founder of the Digital Business school. |
| **Operating model** | The level of business-process standardisation and integration required to deliver value — the bridge between strategy and architecture (Weill–Ross). |
| **Weill** | Peter Weill, MIT CISR senior research scientist; co-author of *IT Governance* (2004) and the operating models. |
| **Ross** | Jeanne W. Ross, MIT CISR principal research scientist; co-author of *IT Governance* (2004). |
| **Diversification** | Operating model: low standardisation, low integration — independent units sharing only common infrastructure. |
| **Coordination** | Operating model: low standardisation, high integration — units keep own processes but share data/customer/risk. |
| **Unification** | Operating model: high standardisation, high integration — one process, one data model, one company. |
| **Replication** | Operating model: high standardisation, low integration — standard processes replicated into autonomous local units. |
| **IT governance** | The decision rights and accountability framework that encourages desirable behaviour in the use of IT (Weill–Ross); who decides, and who is accountable. |
| **COBIT** | ISACA's IT-governance framework (first published 1996; COBIT 2019 current): control objectives, goals cascade, five governance/management domains. |
| **ITIL** | The IT service-management framework (UK CCTA origins, late 1980s; ITIL 4, 2019, Axelos): how IT services are designed, delivered, operated, and improved. |
| **TOGAF** | The Open Group's enterprise-architecture framework (1995, from TAFIM; TOGAF 10): the ADM cycle for developing business, data, application, and technology architecture. |
| **Decision rights** | The specification of who is authorised to make which IT decisions — the heart of IT governance (Weill–Ross). |
| **Archetype** | In Weill–Ross governance: the pattern of who decides — business monarchy, IT monarchy, feudal, federal, IT duopoly, anarchy. |
| **McFarlan** | F. Warren McFarlan (Harvard Business School), author of the strategic grid (1984, with McKenney's 1983 precursor). |
| **Strategic grid** | McFarlan's 2×2 mapping current vs future strategic impact of IT: Support, Factory, Turnaround, Strategic. |
| **Portfolio** | The set of IT initiatives and systems, classified and governed as a whole rather than one project at a time. |
| **Sourcing** | The make–buy–rent decision: build in-house, buy licensed/SaaS products, or outsource to third parties. |
| **Build** | Developing capability in-house — chosen when the capability differentiates or no market product fits. |
| **Buy** | Acquiring packaged software or SaaS — chosen for standard, well-served capabilities. |
| **Outsource** | Contracting a third party to build and/or run capability — chosen for utility capabilities and scale economics. |
| **Digital transformation** | Organisation-wide change in business model, operating model, and customer experience driven by digital technology. |
| **DTO** | Digital Transformation Office — the dedicated team (typically CEO/CDO/CIO-sponsored) that drives, coordinates, and funds the transformation agenda. |
| **IT value** | The business benefit realised from IT investment, measured ex-post; the subject of the IT scorecard. |
| **Balanced scorecard** | Kaplan and Norton's 1992 framework: performance measured across financial, customer, internal-process, and learning & growth perspectives. |
| **Carr** | Nicholas G. Carr, author of "IT Doesn't Matter" (HBR, May 2003) — the commodity-infrastructure thesis. |
| **IT-doesn't-matter** | The Carr debate: whether IT, as commodity infrastructure, can still be a source of sustainable competitive advantage. |

---

## References and Further Reading

**Canonical sources (verified in this guide):**

- Henderson, J. C., & Venkatraman, N. (1993). *Strategic Alignment: Leveraging Information Technology for Transforming Organizations.* IBM Systems Journal, 32(1), 4–16.
- Venkatraman, N., Henderson, J. C., & Oldach, S. (1993). *Continuous Strategic Alignment: Exploiting Information Technology Capabilities for Competitive Success.* European Management Journal, 11(2), 139–149.
- McFarlan, F. W. (1984). *Information Technology Changes the Way You Compete.* Harvard Business Review, May–June 1984. (With McKenney, J. L., *The Information Archipelago*, HBR 1983, as precursor.)
- Weill, P., & Ross, J. W. (2004). *IT Governance: How Top Performers Manage IT Decision Rights for Superior Results.* Harvard Business School Press.
- Weill, P., & Ross, J. W. (2004). *IT Governance on One Page.* MIT Sloan Working Paper No. 4517-04.
- Carr, N. G. (2003). *IT Doesn't Matter.* Harvard Business Review, May 2003, 81(5), 41–49.
- Carr, N. G. (2004). *Does IT Matter?* Harvard Business School Press.
- Kaplan, R. S., & Norton, D. P. (1992). *The Balanced Scorecard — Measures That Drive Performance.* Harvard Business Review, January–February 1992.
- ISACA. *COBIT Framework* (first published 1996; COBIT 5, 2012; COBIT 2019). isaca.org.
- Axelos. *ITIL* (v1 late 1980s UK CCTA; ITIL 4, 2019). axelos.com.
- The Open Group. *TOGAF* (first published 1995; TOGAF 10). opengroup.org.
- McAfee, A. (2006). *Mastering the Three Worlds of Information Technology.* Harvard Business Review, November 2006.

**Series cross-references (this repository):**

- Business strategy side: [strategic_management_guide.md](strategic_management_guide.md), [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md)
- Investment decision machinery: [business_case_development_guide.md](business_case_development_guide.md)
- Sourcing/contracts: [vendor_management_guide.md](vendor_management_guide.md)
- Change/people: [organizational_behavior_guide.md](organizational_behavior_guide.md), [360_management_guide.md](360_management_guide.md), [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md)
- Consulting framing: [mckinsey_approach_guide.md](mckinsey_approach_guide.md), [management_consulting_skills_guide.md](management_consulting_skills_guide.md)
- Technology layer: [../technology/data_architect_skillgaps_guide.md](../technology/data_architect_skillgaps_guide.md), [../technology/finops_guide.md](../technology/finops_guide.md), [../technology/monolith_to_microservices_guide.md](../technology/monolith_to_microservices_guide.md), [../technology/enterprise_ai_gateway_guide.md](../technology/enterprise_ai_gateway_guide.md)
- Banking context: [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md), [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md), [../banking/hsbc_software_systems_guide.md](../banking/hsbc_software_systems_guide.md)

---

*End of guide. The alignment question remains open — deliberately. Re-answer it every quarter.*


