# Project Management Methodologies: A Comprehensive Guide

**Waterfall · V-Model · Spiral · RUP · CPM/PERT · PMBOK · PRINCE2 · Scrum · Kanban · XP · Lean · DSDM · Crystal · FDD · SAFe · LeSS · Nexus · Disciplined Agile · Hybrid — the Complete Landscape, How They Compare, and How They Combine in Practice**

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Professional Development Guides · **Topic:** Project Management Methodologies
> **Focus:** The full project-management methodology spectrum — predictive/traditional, adaptive/agile, scaled, and hybrid approaches — with selection frameworks and real-world combining patterns for banking and regulated industries (Singapore, EU, global)
> **Companion Guides:** [PMP vs SAFe vs Lean Six Sigma](pmp_safe_lss_comparison_guide.md) · [Scaled Agile Framework (SAFe) Deep-Dive](scaled_agile_framework_guide.md) · [Product Management](product_management_guide.md) · [Sales Methodology Frameworks](sales_methodology_frameworks_guide.md) · [Communication & Stakeholder Management](communication_stakeholder_management_skills_guide.md) · [People Management in the AI Age](people_management_in_ai_age_guide.md) · [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) · [Agentic Workflows](agentic_workflows_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [What This Guide Covers](#1-what-this-guide-covers)
2. [What a Project Management Methodology Is](#2-what-a-project-management-methodology-is)
3. [The Methodology Spectrum](#3-the-methodology-spectrum)
4. [Waterfall](#4-waterfall)
5. [The V-Model](#5-the-v-model)
6. [The Spiral Model](#6-the-spiral-model)
7. [RUP — Rational Unified Process](#7-rup--rational-unified-process)
8. [CPM and PERT](#8-cpm-and-pert)
9. [PMBOK — The PMI Standard](#9-pmbok--the-pmi-standard)
10. [PRINCE2](#10-prince2)
11. [The Agile Manifesto](#11-the-agile-manifesto)
12. [Scrum](#12-scrum)
13. [Kanban](#13-kanban)
14. [XP — Extreme Programming](#14-xp--extreme-programming)
15. [Lean Software Development](#15-lean-software-development)
16. [DSDM and MoSCoW](#16-dsdm-and-moscow)
17. [Crystal](#17-crystal)
18. [FDD — Feature-Driven Development](#18-fdd--feature-driven-development)
19. [SAFe — Scaled Agile Framework](#19-safe--scaled-agile-framework)
20. [LeSS, Nexus, and Scrum@Scale](#20-less-nexus-and-scrumscale)
21. [Disciplined Agile and the Spotify Model](#21-disciplined-agile-and-the-spotify-model)
22. [Hybrid and Modern Approaches](#22-hybrid-and-modern-approaches)
23. [The Comparison Framework](#23-the-comparison-framework)
24. [The Selection Framework](#24-the-selection-framework)
25. [Methodology Selection in Banking](#25-methodology-selection-in-banking)
26. [Combining Methodologies: The Pragmatic Stack](#26-combining-methodologies-the-pragmatic-stack)
27. [Anti-Patterns](#27-anti-patterns)
28. [Certifications and Training](#28-certifications-and-training)
29. [Methodology in 2026: Trends](#29-methodology-in-2026-trends)
30. [Glossary](#30-glossary)
31. [References and Further Reading](#31-references-and-further-reading)

---

## 1. What This Guide Covers

Project management methodologies are the operating systems of delivery: they define how work gets planned, sequenced, controlled, and handed over. This guide surveys the **complete landscape** — from the classic predictive methods (Waterfall, V-Model, Spiral, RUP, CPM/PERT) through the governance standards (PMBOK, PRINCE2) and the entire agile family (Scrum, Kanban, XP, Lean, DSDM, Crystal, FDD), up to the scaled frameworks (SAFe, LeSS, Nexus, Scrum@Scale, Disciplined Agile, Spotify Model) and the hybrid approaches that dominate real organizations (Water-Scrum-Fall, agile-with-governance-wrappers).

It answers five questions:

1. **What is a project management methodology?** The definition, the methodology-vs-framework-vs-standard-vs-process distinction, and the predictive↔adaptive spectrum (Sections 2–3).
2. **What are the major approaches?** Origin, structure, strengths, weaknesses, and best-fit conditions of each — predictive (Sections 4–10) and adaptive (Sections 11–18), plus scaling (Sections 19–21) and hybrid (Section 22).
3. **How do they compare?** A head-to-head comparison table and the dimensions that actually distinguish methodologies (Section 23).
4. **How do you choose?** A selection framework with decision factors, a decision tree, and a fit matrix (Section 24), with the regulated-banking reality spelled out (Section 25).
5. **How do they combine?** The pragmatic layered stack — PRINCE2 governance over Scrum delivery over Kanban support, with PMBOK as the shared knowledge base — plus the anti-patterns to avoid (Sections 26–27).

**Audience.** This guide is written primarily for **solution architects, project/program managers, and engineering leads** — people who must navigate methodology choices without becoming methodologists. It deliberately avoids sectarianism: there is no "best" methodology, only better and worse fits for a given context. Banking examples reflect the author's context at Crédit Agricole CIB (Singapore); the principles transfer to any regulated, multi-stakeholder delivery environment.

---

## 2. What a Project Management Methodology Is

### 2.1 Definition

A **project management methodology** is a structured framework of principles, processes, roles, and artifacts for delivering projects. It is a repeatable system that answers the same questions for every project: *How do we plan? How do we track progress? How do we manage change? How do we know we are done? How do we hand over?*

A methodology gives an organization three things:

- **Shared language** — the same vocabulary for scope, status, risk, and "done", so stakeholders stop renegotiating definitions on every project.
- **Repeatable behavior** — defined ceremonies, artifacts, and decision gates that make delivery predictable and coachable, independent of the individuals involved.
- **An audit trail** — a record of how decisions were made and work was controlled, which is non-negotiable in regulated industries (see Section 25).

### 2.2 Methodology vs Framework vs Standard vs Process

The words are used interchangeably in practice but mean different things — and the difference matters when you are choosing what to adopt:

| Term | Definition | Example |
|------|-----------|---------|
| **Approach** | A general philosophical stance about how work should be planned and delivered (high-level strategy, no prescribed mechanics) | Waterfall ("plan everything up front, then execute in sequence"), agile ("deliver incrementally, adapt as you learn") |
| **Method / Methodology** | A defined, complete system of practices, roles, and artifacts — prescriptive enough to follow without inventing your own | PRINCE2, Kanban, DSDM, RUP |
| **Framework** | A skeleton of roles, events, and artifacts with the *details left to the user* — you must fill in the engineering practices, definitions of done, and policies | Scrum, SAFe, LeSS, Nexus |
| **Standard / Body of Knowledge** | A codified body of knowledge describing what *good practice looks like* — descriptive, not prescriptive; you pick what applies | PMBOK (PMI), ISO 21500, the Agile Manifesto's 12 principles |
| **Process** | A specific sequence of steps for a bounded activity, often embedded inside a methodology | A change-control process, a release gate, a PRINCE2 stage boundary |

The practical rule: **PMBOK is a knowledge standard** (it tells you what project management *involves*, not what to do step-by-step), **PRINCE2 is a method** (you can run a project entirely by its book), **Scrum is a framework** (it defines roles/events/artifacts but deliberately leaves engineering practice out), **Kanban is a method** (six practices applied to an existing workflow), and **Waterfall is an approach** (a sequential philosophy with many possible implementations).

### 2.3 Why Methodology Matters

- **Predictability**: methodology is how you convert "hope" into a forecast — the planning and tracking machinery that lets a bank commit a go-live date.
- **Control**: defined gates and escalation paths are how management exerts control without micro-managing (PRINCE2's "manage by exception" is the purest expression).
- **Quality**: verification points embedded in the flow (V-Model, XP's test-first practices) are how defects are caught at the cheapest possible moment.
- **Compliance**: in banking, the *delivery process itself* is audited — the methodology is the evidence (Section 25).
- **Scaling**: a methodology is the unit of organizational learning; it is what lets 10 teams behave like one coherent delivery engine.

---

## 3. The Methodology Spectrum

### 3.1 The Predictive ↔ Adaptive Continuum

Every methodology in this guide sits somewhere on a spectrum between two ideal extremes:

- **Predictive (traditional/waterfall-style)**: requirements are fixed up front, the plan is built once and controlled, delivery happens in defined phases, and change is formally managed (because change is expensive). The goal is *conformance to plan*.
- **Adaptive (agile)**: requirements are expected to evolve, planning is continuous and just-in-time, delivery happens in small increments, and change is welcomed (because feedback is cheap). The goal is *maximizing value delivered*.

The spectrum is a continuum, not a binary:

```
Predictive ──────────────── Hybrid ──────────────── Adaptive
Waterfall   V-Model   RUP   ← tailored blends →   Scrum   Kanban   XP
PRINCE2 (neutral: fits both)        DSDM (agile with governance)        SAFe (agile, scaled)
```

### 3.2 Where Each Approach Sits

| Approach | Position | Why |
|----------|----------|-----|
| Waterfall | Fully predictive | Sequential phases, fixed requirements, stage gates |
| V-Model | Fully predictive | Waterfall structure with verification/validation paired per stage |
| Spiral | Predictive-leaning iterative | Risk-driven iterations, but heavy up-front architecture and long cycles |
| RUP | Iterative, hybrid-leaning | Time-boxed iterations *inside* a phase-gated lifecycle; workflow disciplines |
| PRINCE2 | Method-neutral | Governance structure can wrap any delivery style, from waterfall to agile |
| PMBOK | Method-neutral | Knowledge standard; PMP exams now explicitly cover predictive, agile, and hybrid |
| DSDM | Adaptive with governance | Agile principles + fixed time/cost and formal control points (MoSCoW, timeboxes) |
| Scrum | Adaptive | Empirical, sprint-based framework |
| Kanban | Adaptive (continuous) | Flow-based, no fixed iterations at all |
| XP | Adaptive | Adaptive planning + extreme engineering discipline |
| Crystal | Adaptive, tailored | Family of methods sized by team and criticality |
| FDD | Adaptive, model-driven | Iterative feature delivery over an up-front domain model |
| SAFe / LeSS / Nexus | Adaptive, scaled | Agile principles applied across many teams |

### 3.3 The Tailoring Principle

Every serious methodology — PRINCE2 explicitly, DSDM by philosophy, Crystal by design — insists that **the methodology must be tailored to its context**. PRINCE2's seventh principle is literally "tailor to suit the environment". Scrum's framework leaves engineering practice unspecified on purpose. Crystal was invented to prove that method weight should scale with team size and risk.

The practical consequence: **the methodology you actually run is always a tailored instance of a named methodology, or a blend of several.** Treat "we use Scrum" as shorthand for "we use Scrum tailored to our context", never as a reason to stop thinking.

## 4. Waterfall

### 4.1 The Classic Sequential Model

Waterfall is the archetype of predictive delivery: a linear sequence of phases, each completed and reviewed before the next begins. The canonical form (popularized via Winston Royce's 1970 paper, which ironically criticized the model he described) is:

```
Requirements → Design → Implementation → Testing → Deployment → Maintenance
```

Each phase produces an approved artifact that feeds the next — a requirements specification, a design document, code, a test report. Progress is measured by phase completion; a project is "80% done" when design is complete and implementation is underway, regardless of how much remains.

### 4.2 Phases with Gates

The defining control mechanism is the **phase gate** (stage gate, tollgate): an explicit review point between phases where stakeholders decide go/no-go based on the phase's exit criteria. A gate is not a status meeting — it is a formal commitment point. In regulated settings, gates map directly onto audit and approval checkpoints (Section 25).

| Gate | Exit criterion example |
|------|------------------------|
| Requirements → Design | Requirements specification signed off; scope baseline approved |
| Design → Implementation | Design documents approved; test strategy agreed |
| Implementation → Testing | Code complete; unit tests passing; build promoted |
| Testing → Deployment | Test sign-off; defects triaged; UAT accepted |
| Deployment → Maintenance | Go-live approved; operational handover complete |

### 4.3 Strengths and Weaknesses

**Strengths:** predictability (cost/schedule estimable once requirements are fixed); documentation (a complete record for audits, handover, and regulators); clear stages (easy to staff, easy to report); simplicity (everyone understands the model); well-suited to fixed-price/contractual delivery where scope is legally locked.

**Weaknesses:** late feedback (users see the product only near the end — the classic "we built the wrong thing correctly"); change resistance (a change mid-phase means revisiting earlier artifacts; the later the change, the more expensive); big-bang integration (all components meet for the first time in testing); long time-to-value; the illusion of certainty (fixed plans on unverified assumptions).

### 4.4 When to Use — and When Not To

**Use when:** requirements are well understood and stable; scope is fixed by contract or regulation; the deliverable is a one-shot effort (construction, infrastructure, a regulatory report); the cost of failure or rework is high; documentation and auditability are primary needs.

**Do not use when:** requirements are genuinely uncertain (new products, new markets); the user needs to see value early; the technology is unproven; competitive pressure demands fast iteration. For those, even a strict organization usually pulls in agile elements — which is exactly how Water-Scrum-Fall (Section 22) was born.

---

## 5. The V-Model

### 5.1 Verification and Validation

The V-Model (developed in Germany in the 1980s, notably for defense software) keeps Waterfall's sequential logic but **pairs every development stage with a corresponding testing stage**, so verification happens continuously rather than in one final testing phase:

```
Requirements analysis ──▶ Acceptance test design (validation: did we build the right thing?)
System design ─────────▶ System test design
Architecture design ────▶ Integration test design
Module design ──────────▶ Unit test design
Implementation ─────────▶ Code + unit testing
```

The left arm is decomposition (defining), the right arm is integration and verification (proving). Each artifact on the left defines the test on the right: the requirements spec defines acceptance tests, the system design defines system tests, and so on. Testing is not an afterthought — it is designed in parallel with the design work, which is the model's core insight.

### 5.2 Strengths and Weaknesses

**Strengths:** defect detection at the cheapest possible stage (a requirement error caught at acceptance-test design costs a fraction of one caught in UAT); explicit traceability (every requirement links to a test — exactly what auditors want); works for both software and systems/hardware engineering; clear quality ownership.

**Weaknesses:** inherits Waterfall's change resistance and late delivery; heavy documentation burden; test *design* gets done early but test *execution* still happens late, so the discovery loop remains long; can encourage checkbox testing — proving the spec was met rather than that the product works.

### 5.3 When to Use

**Use when:** safety-critical or regulated environments where verification evidence is a deliverable — medical devices (FDA), aerospace/defense (DO-178C), automotive (ISO 26262), banking compliance where each control must be traceable to a test. The V-Model is less a delivery philosophy than an **evidence-generation machine**: its traceability matrix is the standard answer to "prove this control works".

**Do not use when:** requirements are fluid, time-to-market is the priority, or the team is small enough that the documentation weight exceeds its value.

---

## 6. The Spiral Model

### 6.1 Risk-Driven Iteration (Boehm, 1986)

Barry Boehm's spiral model (IEEE Software, 1986) was the first serious challenge to Waterfall for large, high-risk software projects. Each loop of the spiral is one **risk-driven iteration** with four quadrants:

```
Quadrant 1: Determine objectives, alternatives, constraints
Quadrant 2: Evaluate alternatives; identify and resolve risks (risk analysis)
Quadrant 3: Develop and verify the next-level product (prototype / build)
Quadrant 4: Plan the next iteration (review, commit, re-enter the spiral)
```

The distinctive feature is that **risk analysis precedes commitment**: each loop starts by asking "what could kill this project?" and resolves the top risks — via prototype, simulation, benchmark, or spike — before building more. The spiral therefore looks different for every project: a low-risk project spirals quickly toward a waterfall-like sequence; a high-risk one spends many loops on prototyping and de-risking.

### 6.2 Strengths and Weaknesses

**Strengths:** explicit risk management as a first-class activity (the ancestor of modern risk-based approaches and even of "spike-first" agile); early resolution of the biggest unknowns; customer sees working prototypes from early loops; accommodates change between loops.

**Weaknesses:** can appear endless — no natural "when is it done?" without discipline; heavyweight — requires risk-analysis skill most teams lack; documentation-heavy; rarely used in its pure form today; its lessons survive mostly inside other models (incremental delivery, risk-driven planning, prototypes as de-risking tools).

### 6.3 When to Use

**Use when:** the project is large, complex, high-risk, and the failure modes are unknown — big transformation programs, first-of-a-kind systems, research-adjacent builds. If your program office asks "what's the riskiest unknown and how do we retire it first?", you are spiraling whether you call it that or not.

**Do not use when:** risk is low and requirements are clear (overkill), or the organization lacks the analytical maturity to do genuine risk analysis rather than ritual risk registers.

---

## 7. RUP — Rational Unified Process

### 7.1 An Iterative Lifecycle with Disciplines

The Rational Unified Process (Rational Software, late 1990s; rooted in the Rational Objectory Process and the work of Booch, Rumbaugh, and Jacobson) is a heavyweight iterative framework for large enterprise software. It combines two orthogonal structures:

- **Four phases** (the time dimension), each ending in a milestone:
  1. **Inception** — business case, scope, feasibility (lifecycle objectives milestone).
  2. **Elaboration** — architecture baseline, risk reduction (lifecycle architecture milestone).
  3. **Construction** — most of the coding, in iterations (initial operational capability milestone).
  4. **Transition** — beta, deployment, user training (product release milestone).
- **Six core disciplines + three supporting disciplines** (the content dimension), applied with varying intensity across phases: business modeling, requirements, analysis & design, implementation, test, deployment; plus configuration & change management, project management, environment.

Each phase runs one or more **iterations** (typically 2–6 weeks) that cut across all disciplines — a small end-to-end slice of analysis, design, code, and test. This is RUP's key move: **iterative delivery inside a phase-gated lifecycle**, with the emphasis shifting from requirements in Inception to implementation in Construction.

### 7.2 Strengths, Weaknesses, and When to Use

**Strengths:** architecture-first thinking (Elaboration exists precisely to retire architectural risk); discipline without abandoning iteration; role clarity (analyst, architect, developer, tester); strong fit for large enterprise teams and tooling ecosystems (RUP was designed around Rational tools — Rose, RequisitePro, ClearCase).

**Weaknesses:** heavy — the full process is more ceremony than most teams can sustain; documentation burden; fell out of favor as leaner agile frameworks won the mindshare; essentially dormant since the early 2010s (its DNA survives in disciplined, milestone-based iterative delivery).

**Use when:** large enterprise software, a mature team that already knows the framework, and architecture risk matters more than speed. **Do not use when:** small teams, startup speed, or any context where the ceremony cost exceeds the control value — in practice, most organizations that adopted RUP migrated to Scrum or a Scrum-with-milestones hybrid.

---

## 8. CPM and PERT

### 8.1 Scheduling Techniques, Not Methodologies

CPM and PERT are not delivery methodologies — they are **scheduling techniques** that any methodology (predictive or agile) can use to answer one question rigorously: *when will this finish, and what determines that date?*

- **CPM — Critical Path Method** (DuPont, late 1950s): models the project as a network of activities with dependencies and durations; the **critical path** is the longest dependency chain through the network — the sequence of activities where any delay directly delays the project end date. Activities not on the critical path have **float/slack** (how long they can slip without delaying the finish); managing float is how a project manager protects the critical path.
- **PERT — Program Evaluation and Review Technique** (US Navy Polaris program, 1958): designed for activities with uncertain durations. Each activity gets a three-point estimate — optimistic (O), most-likely (M), pessimistic (P) — and the expected duration is the weighted average **E = (O + 4M + P) / 6**. This converts expert uncertainty into a defensible schedule.

### 8.2 Key Concepts

| Concept | Meaning |
|---------|---------|
| Critical path | Longest dependency chain; zero-float activities that define the end date |
| Float / slack | Time an activity can slip before the project is delayed (total float vs free float) |
| Dependency types | Finish-to-start, start-to-start, finish-to-finish, start-to-finish (plus lead/lag) |
| Three-point estimate | O, M, P per activity; expected duration (O+4M+P)/6; variance drives confidence |
| Crashing & fast-tracking | Adding resources to critical activities (crash) / overlapping activities (fast-track) |

Modern tools (MS Project, Primavera P6) do the arithmetic; the *thinking* — identifying the critical chain, protecting it, and using float to absorb uncertainty — is the skill.

### 8.3 When to Use

**Use when:** schedule rigor matters and the work has real dependencies — construction, infrastructure rollouts, regulatory remediation programs, merger/integration timelines, any multi-stream program where a late dependency cascades. CPM/PERT also feeds the schedule-confidence analysis that banks run before committing go-live dates (the critical path through SIT/UAT/release is exactly where programs get stuck).

**Do not use when:** work is continuous flow (Kanban territory), dependencies are trivial, or the schedule is a fiction maintained for reporting — a detailed CPM of invented durations is worse than an honest estimate.

## 9. PMBOK — The PMI Standard

### 9.1 What PMBOK Is (and Is Not)

The **PMBOK Guide** (*A Guide to the Project Management Body of Knowledge*, Project Management Institute, first published 1996) is the world's most widely referenced project-management standard. Critically: **it is not a methodology** — it is a knowledge standard that catalogs what competent project management involves, and a *guide* to applying it. You cannot "run a project by PMBOK" the way you can by PRINCE2; you use PMBOK to make sure nothing important is missing from however you do run it. It is the basis of the **PMP** certification — see [PMP vs SAFe vs Lean Six Sigma](pmp_safe_lss_comparison_guide.md) for the certification deep-dive.

### 9.2 The Classic Structure: 10 Knowledge Areas × 5 Process Groups

Through the 6th edition (2017), PMBOK organized project management as **49 processes** mapped across two dimensions:

**10 knowledge areas** (what the project touches):

| # | Knowledge Area | Concern |
|---|----------------|---------|
| 1 | Integration | Coordinating all other areas — charter, plan, change control, close |
| 2 | Scope | Defining and controlling what is in/out of the project |
| 3 | Schedule | Sequencing, estimating, and controlling the timeline |
| 4 | Cost | Estimating, budgeting, and controlling spend |
| 5 | Quality | Standards, assurance, and control of deliverables |
| 6 | Resources | Team and physical resource planning and management |
| 7 | Communications | Planning and managing information flow to stakeholders |
| 8 | Risk | Identifying, analyzing, and responding to uncertainty |
| 9 | Procurement | Acquiring goods/services from vendors |
| 10 | Stakeholders | Engaging those who affect or are affected by the project |

**5 process groups** (when the work happens): **Initiating → Planning → Executing → Monitoring & Controlling → Closing.** Every knowledge area has processes in several groups — e.g., *plan scope management* (Planning), *validate scope* (Monitoring & Controlling), *control scope* (Monitoring & Controlling).

### 9.3 PMBOK 7th Edition (2021): Principles + Performance Domains

The 7th edition (2021) was a deliberate break from the process-based structure. Instead of 49 processes, it presents:

- **12 principles of project management** — stewardship, team culture, stakeholder engagement, value focus, systems thinking, leadership behaviors, tailoring, quality, complexity navigation, risk response optimization, adaptability/resilience, change enablement. These are *values and behaviors*, not steps.
- **8 performance domains** — stakeholders, team, development approach & lifecycle, planning, project work, delivery, measurement, uncertainty. Each domain describes the outcomes to achieve, leaving the *how* to the practitioner.
- **Tailoring** elevated to a first-class concept — the guide explicitly says the approach must be adapted to the project, and it covers predictive, agile, and hybrid lifecycles on equal footing.

The shift mirrors the industry: from *"follow the process"* to *"achieve the outcomes, using whatever approach fits"*. PMP exams (updated to the new Exam Content Outline) now test all three approaches — predictive, agile, and hybrid.

### 9.4 Strengths, Weaknesses, and When to Use

**Strengths:** comprehensive coverage (nothing is off the table — risk, procurement, stakeholders are all there); vendor-neutral and globally recognized; the lingua franca between PMs, PMOs, and auditors; 7th edition's principles/domains structure is genuinely approach-agnostic.

**Weaknesses:** knowledge ≠ method — adopting PMBOK gives you a checklist, not a process; volume (the guide is a reference work, not a manual); can encourage process-by-checklist behavior if treated as a methodology; certification-heavy culture can value the credential over the practice.

**Use when:** you need a common knowledge framework across a PMO or enterprise, are preparing for PMP, or must demonstrate that your delivery approach covers the recognized body of knowledge (a regulator-friendly property). **Do not use as** your actual operating method — pair it with a method (PRINCE2) or framework (Scrum) for the mechanics.

---

## 10. PRINCE2

### 10.1 The Governance Method

**PRINCE2** (PRojects IN Controlled Environments) originated in the UK government in 1989 (PRINCE; the "2" version arrived 1996) under the CCTA/OGC, moved to AXELOS in 2013, and is now owned and administered by **PeopleCert**. It is the world's most widely adopted *project management method* (as opposed to standard or framework) — 1M+ certified professionals, dominant in the UK, Europe, and public sector globally, and widely used in banking and insurance.

PRINCE2's claim is not speed or innovation — it is **control**: a complete, tailorable method for running any project, of any size, in any environment, with clear governance, defined roles, and management by exception. It is delivery-style-neutral: you can run PRINCE2 with waterfall phases, with Scrum inside, or with a continuous-flow team.

### 10.2 The 7 Principles

Principles are the method's ground rules — if any is ignored, it is (strictly) no longer PRINCE2:

1. **Continued business justification** — the project must remain worthwhile; the business case is re-validated at every stage boundary.
2. **Learn from experience** — lessons are captured continuously and applied, not just written at the end.
3. **Defined roles and responsibilities** — clear separation of business, user, and supplier interests.
4. **Manage by stages** — the project is planned stage-by-stage; senior management commits only to the next stage.
5. **Manage by exception** — each level of management sets tolerances (time, cost, scope, risk, quality); escalation happens only when tolerances are exceeded.
6. **Focus on products** — the project is defined by its products and their quality criteria, not by activities ("a project is a means to deliver products").
7. **Tailor to suit the environment** — scale the method to the project's size, risk, and complexity.

### 10.3 The 7 Themes and 7 Processes

**Themes** (aspects to be addressed continuously): **business case, organization, quality, plans, risk, change, progress.**

**Processes** (the project's journey):

| Process | What happens |
|---------|--------------|
| Starting Up a Project | Pre-project: appoint the executive and PM, draft the business case, assemble the project brief |
| Directing a Project | The project board's process: authorize initiation, stage plans, and closure |
| Initiating a Project | Produce the project initiation documentation (PID): business case, plans, risk approach, tolerances |
| Controlling a Stage | Day-to-day management: work packages, progress checks, exception handling |
| Managing Product Delivery | Team manager's process: accept, execute, and deliver work packages |
| Managing a Stage Boundary | End-of-stage review: verify the business case, update plans/risks, get board approval for the next stage |
| Closing a Project | Formal closure: confirm products delivered, lessons logged, benefits plan handed to operations |

### 10.4 Roles

PRINCE2's role model separates the three interests every project must represent:

- **Executive** — owns the business case; the project board's chair; ultimately accountable (business interest).
- **Senior User** — represents the users; specifies needs, ensures the product will deliver benefits (user interest).
- **Senior Supplier** — represents those building the product; ensures feasibility and resource availability (supplier interest).
- **Project Manager** — runs the project day-to-day within board-set tolerances.
- **Team Manager** — manages the team delivering work packages (optional; may be the PM in small projects).

Supporting roles: project assurance (independent oversight on behalf of the board), change authority (approves change requests within tolerance), project support (admin). This role model is why PRINCE2 scales naturally to programs: the same board/user/supplier separation repeats at program level.

### 10.5 Strengths, Weaknesses, and When to Use

**Strengths:** governance and control (the board structure and stage gates give senior management genuine oversight without meddling); product focus (quality criteria per product — audit-friendly); business-case discipline (projects are killed or continued on evidence, not sunk cost); method-neutral (wraps agile or waterfall delivery); tailorable; globally portable across sectors.

**Weaknesses:** terminology and documentation overhead (even tailored, it feels bureaucratic to agile teams); can be process-heavy for small projects if not properly tailored; the certificate industry inflates "PRINCE2-certified" without guaranteeing practice; less guidance on *engineering* than agile methods (it governs, it doesn't build).

**Use when:** governance matters more than speed — public sector, regulated industries, enterprise programs, any project where the board needs controlled visibility and the audit trail must be defensible. **Do not use when:** the culture will not tolerate the ceremony, or you need a delivery *framework* rather than a governance wrapper — PRINCE2 tells you how to control a project, not how to design, build, or test.

### 10.6 PRINCE2 Agile

**PRINCE2 Agile** (launched by AXELOS in **June 2015**; refreshed within the PRINCE2 7th Edition program from 2023 under PeopleCert) is the official hybrid: PRINCE2's governance and control, with agile delivery working inside it. Its five behaviors — transparency, collaboration, rich communication, self-organization, exploration — and its "agile in PRINCE2" techniques (sprinkling agile into processes, flexing *what* is delivered rather than *when*) make it the standard answer for organizations that need governance *and* responsiveness. It is covered in depth alongside the delivery-side hybrids in Section 22.

## 11. The Agile Manifesto

### 11.1 The Document That Changed Delivery

In February 2001, seventeen software practitioners met at Snowbird, Utah, and published the **Manifesto for Agile Software Development** — the reaction against heavyweight, document-driven processes (the RUP/waterfall orthodoxy of the 1990s). It is four values and twelve principles; everything called "agile" traces its lineage here.

**The 4 values:**

> **Individuals and interactions** over processes and tools
> **Working software** over comprehensive documentation
> **Customer collaboration** over contract negotiation
> **Responding to change** over following a plan

The right-hand items still have value — the manifesto says so explicitly. The values are about *weighting*, not replacement.

**The 12 principles** (abridged): satisfy the customer through early and continuous delivery; welcome changing requirements; deliver working software frequently (weeks, not months); business people and developers work together daily; build projects around motivated individuals; prefer face-to-face conversation; working software is the primary measure of progress; maintain a sustainable pace; attend continuously to technical excellence and good design; simplicity is essential; self-organizing teams produce the best architectures; reflect regularly and tune behavior.

### 11.2 The Agile Family Tree

The manifesto did not invent a methodology — it named a family resemblance among existing ones:

- **Scrum** (1990s, Schwaber/Sutherland) — the framework that became dominant.
- **XP** (1990s, Kent Beck) — the engineering discipline wing.
- **Kanban** (from Toyota, adapted by David Anderson from ~2004) — the flow wing.
- **DSDM** (1994, UK consortium) — the governance-aware agile project management method.
- **Crystal** (1990s, Alistair Cockburn) — the tailoring wing.
- **FDD** (1997, De Luca/Coad) — the model-driven wing.
- **Lean Software Development** (2003, Poppendiecks) — the waste-reduction philosophy.

Two things unify them: **empirical process control** (inspect and adapt, on evidence, in short loops) and **customer value as the unit of progress**. Everything else — sprints, boards, test-first, timeboxes, feature lists — is a mechanism for one of those two.

---

## 12. Scrum

### 12.1 The Dominant Agile Framework

Scrum (Schwaber & Sutherland, formalized at OOPSLA 1995; the Scrum Guide is the canonical definition, with the 2020 edition current) is the most widely used agile framework and the default answer to "how do we deliver software in a bank's IT division". It is deliberately minimal: **roles, artifacts, events** — and nothing else. Engineering practice (TDD, CI, architecture) is left to the team; Scrum only provides the empirical loop.

### 12.2 Roles

| Role | Responsibility |
|------|----------------|
| **Product Owner** | Single accountable owner of the Product Backlog — prioritizes for value, represents the customer/business, decides what gets built and what "done" means per item |
| **Scrum Master** | Serves the team and organization — removes impediments, coaches Scrum, protects the process and the team from outside interference (a facilitative-leadership role, *not* a project manager) |
| **Developers** | The cross-functional people who build the Increment — self-managing; no sub-roles or titles (no "tester" or "analyst" inside Scrum; the team owns all of it) |

Notably, there is **no project manager role in Scrum** — the work of a PM (planning, tracking, stakeholder coordination) is distributed across the Product Owner, Scrum Master, and team. In bank IT, this is usually the hardest cultural adjustment (Section 25).

### 12.3 Artifacts

- **Product Backlog** — the single ordered list of everything that might be needed; continuously refined (groomed); items have description, order, size estimate, and value.
- **Sprint Backlog** — the Product Backlog items selected for the current Sprint, plus the plan for delivering them; owned by the developers.
- **Increment** — the sum of all completed Product Backlog items at the end of a Sprint, meeting the **Definition of Done** (the team's shared quality bar — "done done", not "code written").

### 12.4 Events (The Sprint Loop)

The **Sprint** is a fixed-length timebox (typically 1–4 weeks; the Guide says one month or less) containing all other events:

```
Sprint Planning → Daily Scrum → Development → Sprint Review → Sprint Retrospective → next Sprint
```

| Event | Purpose | Timebox guidance |
|-------|---------|------------------|
| Sprint Planning | Select backlog items; define the sprint goal and plan | 2 hrs/week of sprint |
| Daily Scrum | Synchronize; inspect progress toward the sprint goal; adapt the plan | 15 min, same place/time |
| Sprint Review | Inspect the Increment with stakeholders; update the backlog; what to do next | 1 hr/week of sprint |
| Sprint Retrospective | Inspect the team's process; plan improvements | 45 min/week of sprint |

The **Definition of Done** is the contract that makes "done" meaningful across teams and releases — and in regulated delivery, the DoD is where compliance criteria (tests passed, evidence recorded, approvals obtained) get baked into every increment.

### 12.5 Strengths, Weaknesses, and When to Use

**Strengths:** simple to learn, hard to master; forces frequent feedback and early risk exposure; strong empowerment and morale; adapts to evolving requirements; the most portable framework in the industry (tooling, hiring, coaching all exist at scale).

**Weaknesses:** delivers nothing until the first sprint ends; no engineering practices prescribed (a Scrum team can still write terrible code); scaling requires extra machinery (Section 19); role ambiguity in traditional orgs (who is "the PM"?); can degrade into ritual (sprint ceremonies without empirical behavior — the "process theater" anti-pattern, Section 27).

**Use when:** product development with evolving requirements, any context where feedback cycles beat up-front certainty, and team-level (or scaled, Section 19) delivery. **Do not use when:** you cannot deliver a potentially releasable increment each sprint (huge infrastructure dependencies, hard regulatory milestones), or the organization will not accept the authority changes (a PO without real authority is a backlog clerk).

---

## 13. Kanban

### 13.1 The Pull-Based Method from Toyota

Kanban (Japanese for "signboard") originated in Toyota's just-in-time manufacturing (Taiichi Ohno) and was adapted for knowledge work by David Anderson from the mid-2000s. Where Scrum timeboxes work into sprints, **Kanban has no iterations at all**: work flows continuously through a visualized pipeline, pulled by capacity rather than pushed by schedule.

The six core practices:

1. **Visualize** — make the workflow visible on a board (columns = workflow states: To Do → In Progress → Review → Done).
2. **Limit work in progress (WIP limits)** — cap the number of items in each column; the single most powerful practice, because it exposes bottlenecks instantly (the column that fills up *is* the constraint).
3. **Manage flow** — track flow metrics (cycle time, throughput, lead time) and act on them.
4. **Make policies explicit** — definitions of ready/done, priority rules, escalation rules written down.
5. **Implement feedback loops** — regular service-delivery reviews, operations reviews, risk reviews.
6. **Improve collaboratively, evolve experimentally** — use the metrics to run experiments (the "Kaizen" tradition).

### 13.2 Strengths, Weaknesses, and When to Use

**Strengths:** minimal disruption (you start from the current process — no big-bang change); continuous delivery (work ships as soon as it's done); outstanding for mixed/urgent workloads (interruptions are absorbed as flow, not sprint-wreckers); visible bottlenecks; no estimation required (flow metrics replace estimates).

**Weaknesses:** no timebox means no forced review rhythm (weak "inspect and adapt" discipline unless policies create it); no explicit roles or planning ceremony (teams must add their own); can become a dumping ground without WIP discipline; delivery date forecasting is statistical, which some stakeholders find unsatisfying.

**Use when:** operations, support, maintenance, and any continuous-flow work — production support desks, infrastructure change pipelines, regulatory remediation queues, the "sustaining" work that Scrum sprints handle badly. **Do not use when:** you need a defined product-increment cadence with stakeholders, or the team needs the structure of ceremonies to build discipline — start with Scrum and add Kanban-style flow later if needed.

---

## 14. XP — Extreme Programming

### 14.1 Engineering Discipline as the Method

Extreme Programming (Kent Beck, late 1990s, Chrysler C3 project; book 1999) is the engineering-quality wing of agile: it takes sound practices and "extremes" them — code review becomes **pair programming**, testing becomes **test-driven development** (write the failing test first), integration becomes **continuous integration** (integrate many times a day), design becomes **simple design** (the simplest thing that works, refactored continuously).

Core practices: pair programming; TDD (red-green-refactor); continuous integration; refactoring; simple design; collective code ownership (anyone can improve any code); coding standards; on-site customer; planning game (small, prioritized releases); sustainable pace (40-hour week); metaphor (shared system vision); small releases.

### 14.2 Strengths, Weaknesses, and When to Use

**Strengths:** the best-known antidote to technical-debt death spirals (tests + refactoring keep quality high while velocity stays up); defect reduction (pairs + tests catch errors at the cheapest point); sustainable pace reduces burnout; practices are complementary — TDD makes refactoring safe, refactoring keeps simple design viable.

**Weaknesses:** demanding — pair programming and TDD require real skill and continuous discipline (they are the first practices dropped under pressure, which is precisely when they matter most); team intimacy requirements (pairing and collective ownership work poorly in heavily distributed or low-trust settings); the "customer on-site" premise fits few real organizations.

**Use when:** engineering-quality-focused teams, complex or long-lived codebases where debt is the killer risk, products where defects are expensive (trading systems, payments). **Do not use when:** the organization cannot sustain the discipline, or the team is too distributed for pairing — take XP's practices selectively (TDD + CI are table stakes everywhere today; see Section 22).

---

## 15. Lean Software Development

### 15.1 The 7 Principles (Poppendieck, 2003)

Mary and Tom Poppendieck's *Lean Software Development* (2003) translated the Toyota Production System into software. Its 7 principles:

1. **Eliminate waste** — anything not adding customer value is waste (the seven software wastes: partially done work, extra features, extra processes, task switching, waiting, motion, defects).
2. **Amplify learning** — short feedback loops, iteration, and experimentation are how you learn; treat work as an options portfolio.
3. **Decide as late as possible** — defer commitment to the last responsible moment (irreversible decisions early, reversible ones late).
4. **Deliver as fast as possible** — speed is a feature: fast delivery creates learning and options.
5. **Empower the team** — respect people; give teams purpose, mastery, and autonomy (the human-respect principle Lean shares with agile).
6. **Build integrity in** — perceived and conceptual integrity; quality is designed in, not inspected in.
7. **See the whole** — optimize the value stream end-to-end, not local silos (suboptimization is the enemy).

### 15.2 Strengths, Weaknesses, and When to Use

**Strengths:** a lens, not a box — Lean works *with* any delivery method (it is why "lean-agile" is a coherent phrase); waste analysis is a powerful diagnostic for bloated processes; value-stream thinking connects IT delivery to business outcomes; combines naturally with Kanban (its operational arm) and Six Sigma (its quality arm — see the [PMP/SAFe/LSS guide](pmp_safe_lss_comparison_guide.md)).

**Weaknesses:** abstract — no roles, events, or artifacts (it needs a delivery method to operate through); "eliminate waste" can be misread as "remove process" (removing governance in a bank is not waste elimination, it is risk creation); measurement-heavy to do properly.

**Use when:** waste-focused improvement programs, process re-engineering, and any organization wanting a value-stream view of delivery. **Do not use as** a standalone delivery method — pair Lean's principles with Scrum/Kanban mechanics.

---

## 16. DSDM and MoSCoW

### 16.1 The "Agile Project Management" Method

**DSDM** (Dynamic Systems Development Method, DSDM Consortium, founded 1994 in the UK) is the agile method with the strongest project-management and governance DNA — the natural bridge from traditional PM into agile. Its current form (DSDM Atern) is the basis of the **AgilePM** certification (APMG). It is the "agile for people who must still deliver on time and budget" method.

**The 8 principles:**

1. **Focus on the business need** — every decision traces to business value.
2. **Deliver on time** — time is fixed; scope flexes (the opposite of waterfall's fixed-scope thinking).
3. **Collaborate** — active, continuous stakeholder and user involvement.
4. **Never compromise quality** — the agreed quality level is fixed; you flex scope, not quality.
5. **Build incrementally from firm foundations** — solid up-front business and architecture understanding, then incremental delivery.
6. **Develop iteratively** — feedback loops throughout.
7. **Communicate continuously** — face-to-face, workshops, show-and-tell.
8. **Demonstrate control** — the project manager runs it with visibility, not hope.

### 16.2 MoSCoW and Timeboxing — DSDM's Famous Contributions

**MoSCoW prioritization** (coined at Oracle by Dai Clegg, 1994) is DSDM's most widely used export — arguably the most used prioritization scheme in the industry, far beyond DSDM projects:

- **M — Must have**: without it the solution is not viable (not "nice", not "desired" — truly mandatory; regulators' Musts, contractual Musts).
- **S — Should have**: important, but a workaround exists; included if time allows after Musts.
- **C — Could have**: desirable; included only if Musts and Shoulds are secure.
- **W — Won't have (this time)**: explicitly excluded — the most underused and most valuable category, because it kills scope creep by naming the no.

**Timeboxing** is DSDM's delivery unit: a fixed period (2–6 weeks) in which a *prioritized* set of work is delivered, with the MoSCoW order guaranteeing that if time runs out, what's dropped is the least valuable. Timebox + MoSCoW = "deliver on time, flex scope" made operational.

### 16.3 Strengths, Weaknesses, and When to Use

**Strengths:** the best "agile with formal governance" fit — the PM role, business-case discipline, and control points survive intact, which is exactly what banks' delivery offices demand; MoSCoW gives stakeholders a concrete negotiation language; timeboxing is schedule-honest.

**Weaknesses:** heavier than Scrum (roles and management products to maintain); the firm-foundations principle needs up-front business analysis many agile teams skip; less well-known than Scrum, so hiring and tooling are thinner.

**Use when:** agile delivery inside an organization that requires formal governance — public sector, banks, any environment where "the PM owns the delivery" is non-negotiable. AgilePM certification is the credential pair for PRINCE2 holders who want agile delivery skills (Section 28).

---

## 17. Crystal

### 17.1 The Tailoring Pioneer (Cockburn)

Alistair Cockburn (a manifesto signatory) developed the **Crystal family** in the 1990s on a simple observation: *one process does not fit all*. Crystal is a family of methods, each tuned by **team size** and **criticality** (the cost of a defect):

| Method | Team size | Typical use |
|--------|-----------|-------------|
| **Crystal Clear** | Up to ~6–8 | Small teams, low criticality — the lightest: frequent delivery, reflective improvement, osmotic communication, personal safety, easy access to expert users |
| **Crystal Yellow** | ~10–20 | Medium teams |
| **Crystal Orange** | ~20–50 | Larger teams — adds roles and coordination ceremony |
| **Crystal Red** | 50+ | Large teams — formal coordination, sub-teams, more artifacts |

Criticality scales the rigor: a defect that could kill (life-support, trading settlement) demands more verification ceremony than a defect that merely annoys. Crystal's motto — "the methodology is tailored to the project" — predates and preaches what every modern framework now claims.

### 17.2 Strengths, Weaknesses, and When to Use

**Strengths:** the intellectual foundation of tailoring (Section 3.3); honest about ceremony costs; the size/criticality grid is a genuinely useful sizing heuristic; light enough to disappear into the work.

**Weaknesses:** deliberately non-prescriptive — gives you a sizing logic, not a playbook; no commercial ecosystem (no certifications, no tooling, no hiring market); the "osmotic communication" premise assumes co-location.

**Use when:** teams wanting lightweight, tailored methods and the freedom to design their own process; as a framework for thinking about how much ceremony any project deserves. **Do not use when:** you need a named, hireable, auditable process — pick Scrum/PRINCE2 and tailor that instead.

---

## 18. FDD — Feature-Driven Development

### 18.1 Model-Driven, Feature-Sized (De Luca & Coad, 1997)

Feature-Driven Development (Jeff De Luca and Peter Coad, 1997, originating on a Singapore bank project — a pleasing fact for this guide's author) delivers software by **features**: small, client-valued functions sized for delivery in 2–10 days. It is the model-driven wing of agile: unlike Scrum's emergent design, FDD starts with a domain model.

**The 5 processes:**

1. **Develop an overall model** — a domain model built by domain experts + developers in a short modeling sprint.
2. **Build a feature list** — decompose the model into features, grouped by domain area.
3. **Plan by feature** — sequence features into iterations by dependency and business priority.
4. **Design by feature** — per feature (or feature set): design review, sequence diagrams, class updates.
5. **Build by feature** — implement, unit test, inspect, and promote the feature; every feature is a progress milestone.

Roles include the **chief programmer** (a senior architect who owns design decisions for a feature area) — an early acknowledgment that architecture needs an owner. Progress reporting is feature-count-based ("X of Y features complete"), which is both FDD's transparency strength and its risk (features done ≠ value delivered).

### 18.2 Strengths, Weaknesses, and When to Use

**Strengths:** scales to larger teams (feature areas partition work cleanly); regular, inspectable progress (feature completion is concrete); architecture gets up-front attention without waterfall weight; works when multiple teams share one domain model.

**Weaknesses:** the domain-model-first premise fights emergent requirements; chief-programmer bottleneck; feature granularity discipline is hard to maintain; a smaller community than Scrum, with thin tooling.

**Use when:** larger teams, feature-oriented portfolios, and contexts where a shared domain model is genuinely valuable (banking domains are ideal — account, trade, payment models are stable). **Do not use when:** requirements are too fluid to model first, or the team is small enough that FDD's structure is overhead.

## 19. SAFe — Scaled Agile Framework

### 19.1 The Enterprise Scaling Leader

**SAFe** (Scaled Agile Framework, Dean Leffingwell, first released 2011; current major version **SAFe 6.0**, 2023) is the most widely adopted framework for scaling agile across enterprises — used by a large share of the Fortune 100 and the default scaling answer in banking. It coordinates many agile teams into a single value-delivery engine. This section is the landscape summary; the full deep-dive — configurations, levels, roles, PI Planning, banking adoption — is in the companion guide [The Scaled Agile Framework (SAFe)](scaled_agile_framework_guide.md).

### 19.2 Configurations, Levels, and Core Mechanics

**4 configurations** (grow from simple to full): **Essential** (team + program level), **Large Solution** (adds the Solution Train for building large systems), **Portfolio** (adds portfolio-level investment governance), and **Full** (everything combined).

**3 core levels:** Team (Scrum/Kanban teams), Program (the **Agile Release Train** — an ART: 5–12 teams delivering together), and Portfolio (epics, funding, lean budgeting). Large Solution adds a fourth layer, the **Solution Train**, for systems too big for one ART.

| SAFe mechanism | What it is |
|----------------|------------|
| **Agile Release Train (ART)** | The long-lived team of agile teams (5–12) delivering a solution together |
| **Program Increment (PI)** | The ART's planning and delivery cadence — 8–12 weeks, with a **PI Planning** event at the start where all teams plan together |
| **Solution Train** | The layer above ARTs for Large Solution configurations |
| **Architecture Runway** | The kept-current architectural foundation (enablers, refactoring, infrastructure) that lets features land without rework — the architect's key contribution (Section 25) |
| **RTE / STE** | Release Train Engineer (the ART's chief servant-leader) and Solution Train Engineer (its Large-Solution counterpart) |

SAFe's critics call it "agile in name, waterfall in cadence" (PI Planning can look like a big up-front commitment); its defenders note that it is the only scaling framework that seriously addresses portfolio funding and program governance — which is precisely what banks need (see [SAFe in Banking in the deep-dive](scaled_agile_framework_guide.md)).

---

## 20. LeSS, Nexus, and Scrum@Scale

### 20.1 LeSS — Large-Scale Scrum (Larman & Vodde)

**LeSS** (Large-Scale Scrum, Craig Larman and Bas Vodde, from 2005) scales Scrum with **minimal process addition**: the principle is "start with Scrum, add as little as possible". Two frameworks: **LeSS** (up to ~8 teams, ~50 people) and **LeSS Huge** (8+ teams, hundreds of people, organized in requirement areas).

Key features: **one Product Backlog and one Product Owner** for the whole product (no per-team backlogs); every team does a Sprint at the same time (one sprint, synchronized); **one Definition of Done shared by all teams**; direct team-to-team communication and joint refinement; the PO works with teams but does not micro-manage. The scaling answer is *more communication and better design*, not more roles and artifacts. LeSS is popular in product-oriented banks that want "Scrum without the SAFe machinery" — but it demands genuinely mature Scrum teams and strong architecture, because it offers little ceremonial scaffolding to hide behind.

### 20.2 Nexus (Scrum.org)

**Nexus** (Scrum.org, 2015) scales Scrum for **3–9 teams** working on one product. Its distinctive addition is the **Nexus Integration Team** — a small group (PO + Scrum Master + selected members) responsible for ensuring the teams' work actually integrates into a usable product each sprint, and the **Nexus Sprint** (all teams on one synchronized sprint with joint planning, review, and retrospective). Where LeSS strips roles away, Nexus adds exactly one: an integration-focused team, because the observed failure mode of multi-team Scrum is *broken integration*.

### 20.3 Scrum@Scale (Sutherland)

**Scrum@Scale** (Jeff Sutherland, 2016) is the "scale-free" model: instead of a fixed hierarchy, it defines **accountability circles** — overlapping teams of teams (the Scrum-of-Scrums pattern) that each retain full Scrum accountability, with the Executive Action Team handling cross-circle issues and organizational impediments. Its selling point is that the structure can grow and shrink without re-architecture — scale-free, like the internet. It is lighter than SAFe and more flexible than LeSS, but less prescriptive, which makes adoption dependent on strong coaching.

### 20.4 Choosing Among the Scaling Options

| Framework | Team count | Philosophy | Best for |
|-----------|-----------|------------|----------|
| SAFe | 5–12 per ART, many ARTs | Prescriptive, portfolio-aware | Large enterprises needing governance + alignment (banks) |
| LeSS | Up to 8 (Huge: 8+) | Minimalism — "Scrum, nothing else" | Product orgs with mature Scrum |
| Nexus | 3–9 | One integration team over Scrum teams | Single-product, integration-risk contexts |
| Scrum@Scale | Any | Scale-free circles | Orgs wanting flexibility without a fixed framework |
| Disciplined Agile | Any | Goal-driven toolkit, hybrid | Orgs wanting to *design* their own way of working |

---

## 21. Disciplined Agile and the Spotify Model

### 21.1 Disciplined Agile (DA) — PMI's Toolkit

**Disciplined Agile** (initially the Disciplined Agile Delivery framework, Scott Ambler/Mark Lines from 2011; acquired by PMI in 2019; now the **DA toolkit**) is a hybrid, goal-driven approach: rather than prescribing one method, it gives teams a decision kit — a set of process goals, each with several proven options drawn from every major method (Scrum, Kanban, XP, Lean, SAFe, and traditional practices). Teams **choose their way of working (WoW)** per goal, document the choices, and continuously improve. "DAW" (Disciplined Agile Way of Working) is the resulting tailored process.

Its value: it legitimizes *mixing* practices, which is what most organizations secretly do anyway (Section 26), and it gives PMI-certified professionals a home for agile — the **DASM/DASSM** certifications. Its risk: without a strong coach, "choose your own way" becomes an excuse for mush — goal-driven needs disciplined decision-making.

### 21.2 The Spotify Model — An Influential Org Pattern (Not a Methodology)

The **Spotify Model** (named from a 2012 blog post by Henrik Kniberg and Anders Ivarsson) is *not* a delivery methodology — it is an **organizational pattern** for structuring agile teams:

- **Squad** — a small, cross-functional, self-organizing team with a mission (≈ Scrum team).
- **Tribe** — a group of squads in a related business area (≈ mini-division).
- **Chapter** — a horizontal group of people with the same skill across squads (QA engineers, architects) — the model's answer to "how do specialists stay connected?"
- **Guild** — a voluntary community of interest across the whole company (e.g., a "testing guild").

Spotify itself later disowned the model as a codified framework (it was a snapshot of one company's evolution, not a design). Its durable lessons: mission-oriented squads, skill-aligned chapters for specialist identity, and voluntary guilds for cross-pollination — all compatible with any delivery method underneath.

---

## 22. Hybrid and Modern Approaches

### 22.1 Water-Scrum-Fall — The Common Reality

**Water-Scrum-Fall** (the name is informal, in wide use by the mid-2010s) describes the de facto reality of most large organizations: **waterfall at the edges, Scrum in the middle**. The project goes through formal requirements and design phases, development runs as Scrum sprints, then a formal test/UAT/release phase gates the go-live.

```
Requirements (formal) → Design (formal) → [ Scrum sprints ] → SIT/UAT (formal) → Release (gated)
```

It is universally mocked and universally practiced. It is neither good nor bad per se: it is what you get when the *contracting/regulatory environment* demands up-front commitment and the *development reality* demands feedback loops. The failure mode is when the waterfall edges are so rigid that sprint feedback cannot flow back (requirements frozen too early, UAT gates too late) — then you have paid for agile's ceremony without getting agile's learning.

### 22.2 Agile-Waterfall Hybrid (Agile Inside, Governance Outside)

The regulated-industry refinement of Water-Scrum-Fall is a *deliberate* architecture: **agile delivery wrapped in a governance and compliance shell**. The shell is not an accident of history — it is the audit trail (Section 25). Characteristics:

- **Outside (governance shell):** phase gates, change control, risk/compliance reviews, evidence capture, release approvals — the language of regulators and internal audit.
- **Inside (delivery core):** Scrum/Kanban teams, sprint reviews, continuous integration, Definition of Done that includes compliance criteria.

The two connect through defined interfaces: a **stage boundary** (PRINCE2) or **PI boundary** (SAFe) where the delivery evidence is packaged and presented to the governance layer. Done well, the shell *protects* the agile core from bureaucratic drift; done badly, it suffocates it (Section 27).

### 22.3 PRINCE2 Agile — Governance + Agile Delivery

As covered in Section 10.6, **PRINCE2 Agile** is the official, method-level expression of the same idea: PRINCE2's seven processes and management products provide the governance; agile behaviors (transparency, collaboration, rich communication, self-organization, exploration) and techniques (timeboxes, MoSCoW, frequent releases) operate inside. Its distinctive guidance: **flex *what* is delivered (scope) rather than *when* (time)**, and apply agile at the right depth ("sprinkle", "sandwich", or "nest" agile into the PRINCE2 processes depending on context). For a bank's PMO, PRINCE2 Agile is often the most defensible "official" answer to *"how do we do agile but stay controlled?"* — both vocabularies are auditable.

### 22.4 The "Just Enough Process" Philosophy

Every tailoring-capable method converges on the same principle: **the right amount of ceremony for the context** — Crystal's size/criticality grid (Section 17), DSDM's firm-foundations-plus-timeboxes (Section 16), PMBOK 7's tailoring as a first-class concept (Section 9.3), PRINCE2's seventh principle (Section 10.2). The operational test is simple: *does this artifact, meeting, or approval change a decision?* If yes, keep it; if it only consumes time, cut it. The question "what is the minimum process that still gives us control, auditability, and learning?" should be asked at the start of every project, not inherited from the last one.

### 22.5 The "No-Methodology" Reality

The honest truth about methodology in most organizations: **the methodology is what you actually do, not what the poster in the delivery office says.** A 2020s enterprise typically runs an undocumented blend — Scrum ceremonies with waterfall gates, Kanban boards under a PRINCE2-looking PMO, SAFe terminology with none of SAFe's actual cadence. This is not necessarily a failure: implicit hybrid is often *working* tailoring. It becomes a failure when:

- The blend is invisible and unowned — nobody can describe the actual process, so nobody can improve or audit it.
- Different teams' blends are incompatible — no shared language across handoffs.
- The poster says "agile" while the process says "waterfall" — the identity gap erodes trust.

The remedy is not "pick one methodology and enforce it" — it is to **make the actual process explicit and deliberate** (Disciplined Agile's WoW, or a tailored PRINCE2/Scrum hybrid), so the blend is chosen rather than accidental.

## 23. The Comparison Framework

### 23.1 The Methodology Comparison Table

The full landscape at a glance — origin, type, key artifacts, roles, ceremonies, and fit. "Type" uses the spectrum from Section 3 (predictive / adaptive / hybrid / scaling / standard / technique):

| Methodology | Origin | Type | Key artifacts | Roles | Ceremonies | Best for | When NOT to use |
|-------------|--------|------|---------------|-------|-----------|----------|-----------------|
| **Waterfall** | Royce 1970 (formalized) | Predictive | Requirements spec, design docs, phase-gate reports | PM, business analyst, architects, testers | Phase gates, sign-offs | Fixed scope, known requirements, contractual | Evolving requirements, fast feedback needed |
| **V-Model** | Germany 1980s | Predictive | Traceability matrix, test designs per stage | PM, architects, test leads, QA | Stage verification/validation reviews | Safety-critical, regulated builds | Fluid requirements, small teams |
| **Spiral** | Boehm 1986 | Predictive-iterative | Risk register, prototypes, spiral plans | PM, risk analysts, architects | Risk-analysis reviews per loop | High-risk, complex, large | Low risk, clear scope |
| **RUP** | Rational 1998 | Iterative hybrid | Phase milestones, architecture baseline, iteration plans | Analyst, architect, developer, tester | Phase milestone reviews, iterations | Large enterprise software | Small teams, speed-critical |
| **PMBOK** | PMI 1996 (7th ed. 2021) | Standard (method-neutral) | Charter, WBS, risk register, project plan | PM, PMO | Process groups; per-org ceremonies | Knowledge framework, PMP prep, PMO lingua franca | As a standalone operating method |
| **PRINCE2** | UK 1989/1996 | Method (governance) | Business case, PID, stage plans, work packages | Executive, senior user, senior supplier, PM, team manager | Stage boundaries, board meetings, exception reviews | Governance-heavy contexts, public sector, enterprise | Ceremony-averse cultures |
| **Scrum** | Schwaber/Sutherland 1995 | Adaptive framework | Product backlog, sprint backlog, increment, DoD | PO, Scrum Master, developers | Sprint planning, daily scrum, review, retro | Evolving product development | No releasable increment possible |
| **Kanban** | Toyota; Anderson 2000s | Adaptive (flow) method | Kanban board, WIP limits, flow metrics | No fixed roles | Service reviews, standups, replenishment | Ops, support, continuous flow | Need defined release cadence |
| **XP** | Beck 1990s | Adaptive (engineering) | Test suite, pair programming rotation, CI pipeline | Developers, on-site customer, coach | Planning game, standup, CI integration | Engineering-quality-critical code | Distributed/low-discipline teams |
| **Lean** | Poppendieck 2003 | Philosophy | Value stream map, waste analysis, kanban | Any (works through a method) | Value-stream mapping workshops | Waste reduction, process improvement | As a standalone delivery method |
| **DSDM** | UK 1994 | Adaptive with governance | Prioritized requirements, timebox plans, MoSCoW list | PM, business ambassador, tech coordinator, team | Timebox planning, workshops, show-and-tell | Agile + formal governance | Teams wanting lightweight Scrum |
| **Crystal** | Cockburn 1990s | Adaptive (tailored family) | Tailored per size/criticality | Team-dependent | Team-dependent | Small teams wanting light methods | Need a hireable/auditable named process |
| **FDD** | De Luca/Coad 1997 | Adaptive (model-driven) | Domain model, feature list, feature plans | Chief programmer, class owners, domain experts | Feature planning/design/build per feature | Larger teams, stable domain model | Highly fluid requirements |
| **SAFe** | Leffingwell 2011 | Scaling framework | ARTs, PI objectives, backlog epics, architectural runway | RTE, STE, PO, Scrum Master, architects | PI Planning, ART sync, inspect & adapt | Multi-team enterprise delivery | Small orgs, single-team products |
| **LeSS** | Larman/Vodde 2005 | Scaling framework | One product backlog, one DoD | One PO, Scrum Masters, teams | One synchronized sprint; joint ceremonies | Mature Scrum orgs scaling up | Orgs needing portfolio governance |
| **Nexus** | Scrum.org 2015 | Scaling framework | Nexus sprint backlog, integration plan | Nexus Integration Team + Scrum teams | Nexus sprint planning/review/retro | 3–9 teams, one product | Distributed orgs, multi-product |
| **Scrum@Scale** | Sutherland 2016 | Scaling framework | Scrum-of-Scrums, accountability circles | Circle leads, Executive Action Team | Scrum-of-Scrums, circle reviews | Flexible scaling without fixed structure | Need prescriptive governance |
| **Disciplined Agile** | Ambler/Lines 2011; PMI 2019 | Hybrid toolkit | WoW document, process-goal choices | Any (role kit) | Goal-driven; team-chosen | Tailored hybrid, method mixing | No coaching capability |
| **Spotify Model** | Kniberg/Ivarsson 2012 | Org pattern | Squad/tribe/chapter/guild map | Squad lead, chapter lead | Squad rituals, guild meetups | Structuring agile orgs | As a delivery methodology |

### 23.2 The Dimensions of Comparison

Beyond the table, eight dimensions distinguish methodologies — useful both for choosing and for describing what an organization *actually* runs:

| Dimension | Spectrum | Examples |
|-----------|----------|----------|
| **Planning approach** | Upfront ↔ continuous | Waterfall (upfront) vs Scrum (rolling) vs Kanban (just-in-time) |
| **Delivery** | Phased ↔ incremental ↔ continuous | Waterfall (phased) vs Scrum (incremental sprints) vs Kanban/CD (continuous) |
| **Requirements** | Fixed ↔ evolving | Contract/regulatory (fixed) vs product discovery (evolving) |
| **Team size** | Small ↔ large | Crystal Clear (small) vs SAFe (enterprise) |
| **Change tolerance** | Low ↔ high | V-Model (low) vs Scrum (high, welcomed) |
| **Documentation weight** | Minimal ↔ heavy | Kanban (minimal) vs PRINCE2/V-Model (heavy) |
| **Customer involvement** | Periodic ↔ continuous | Waterfall (at gates) vs DSDM (workshops, continuous) |
| **Governance level** | Low ↔ high | Kanban (low) vs PRINCE2 (high) — and **regulatory fit** is the meta-dimension that decides for banks (Section 25) |

---

## 24. The Selection Framework

### 24.1 Decision Factors

Choosing a methodology is a judgment across seven factors — and the honest answer is usually a *combination* (Section 26), not a single pick:

1. **Project type** — product (evolving, iterative: Scrum) vs service/delivery (defined scope: Waterfall/PRINCE2) vs internal/platform (continuous: Kanban) vs regulated (governance shell + agile core).
2. **Requirements certainty** — can requirements be fully known up front? (Yes → predictive; No → adaptive; Partly → hybrid).
3. **Team size and experience** — small/experienced (light methods) vs large/inexperienced (more structure: Scrum-with-discipline, SAFe, PRINCE2).
4. **Organizational culture** — the methodology must fit what the organization will *actually accept*; culture beats framework choice every time (Section 22.5).
5. **Regulatory/compliance requirements** — audit trails, change management, evidence (Section 25); this factor can override all others in banking.
6. **Customer/stakeholder involvement** — available and willing to engage continuously (agile works) vs remote/periodic (predictive or gated hybrid).
7. **Risk profile** — high uncertainty/novelty → iterative + risk-driven (Spiral's lesson); low → linear is fine.
8. **Delivery cadence needed** — continuous operations (Kanban), regular increments (Scrum/DSDM), one-shot milestones (Waterfall/PRINCE2).

### 24.2 The Decision Tree

```
Is the scope fixed by contract/regulation, and are requirements well understood?
├─ YES ── Is governance/audit evidence the dominant concern?
│         ├─ YES ──▶ PRINCE2 (+ V-Model for safety-critical verification)
│         └─ NO ───▶ Waterfall (or V-Model if verification-critical)
└─ NO (requirements will evolve)
    ├─ Is it continuous operations/support work? ──▶ Kanban
    ├─ Is it one team building a product? ──────────▶ Scrum (+ XP practices for quality)
    ├─ Is it a large multi-team program? ────────────▶ SAFe / LeSS / Nexus (by team count, §20.4)
    ├─ Is it high-risk, novel, complex? ─────────────▶ Spiral-style risk-driven iteration
    ├─ Does it need formal governance anyway? ───────▶ DSDM / PRINCE2 Agile / agile-in-governance-shell
    └─ Multiple contexts in one program? ────────────▶ Tailored hybrid stack (§26)
```

### 24.3 The "Methodology Fit" Matrix

| Project characteristics | Recommended approach |
|-------------------------|----------------------|
| Regulated, fixed scope (regulatory remediation, reporting) | PRINCE2 + Waterfall/V-Model phases; agile only inside a controlled envelope |
| Product development, evolving requirements, one team | Scrum (+ XP engineering practices) |
| Continuous operations / support / maintenance | Kanban |
| Large product portfolio, many teams | SAFe (governance + alignment) or LeSS (mature Scrum org) |
| High-risk, large, complex, unknown failure modes | Spiral-style risk-driven iteration, or agile with heavy architecture runway |
| Governance-heavy, any delivery style | PRINCE2 (or PRINCE2 Agile) |
| Everything at once (typical bank program) | Layered hybrid stack — Section 26 |

---

## 25. Methodology Selection in Banking

### 25.1 Regulated Delivery: Agile Inside, Governance Outside

For a bank — and for the author's context at Crédit Agricole CIB — methodology selection is dominated by one fact: **the delivery process itself is audited.** Regulators and internal audit examine not just *what* was delivered but *how it was controlled*: change management, testing evidence, approval records, risk decisions. This is the "audit trail requirement" that shapes everything.

The consequence is the pattern already introduced in Section 22.2: **agile inside, governance outside**. The agile core maximizes delivery responsiveness; the governance shell provides the evidence trail. This is not a compromise — it is the correct design for a regulated environment, and the SDLC below is its standard shape.

### 25.2 The Banking SDLC

```
Requirements (formal, business sign-off)
   → Architecture (design authority, risk/compliance review)
   → Build (agile — Scrum sprints, CI, DoD with compliance criteria)
   → SIT (system integration testing, controlled environment)
   → UAT (user acceptance, business sign-off)
   → Release (change advisory board, go/no-go gates, evidence pack)
```

Each arrow is a gate with an evidence requirement. Note what is *not* there: no pure waterfall (requirements evolve through discovery), no pure Scrum (SIT/UAT/release cannot be sprint-sized in a controlled environment with multiple integrated systems). The gates are where the regulator's questions get answered: *who approved this change? what was tested? what evidence exists? who signed off?*

### 25.3 The MAS / Regulatory Angle

In Singapore, the MAS framework (and equivalent regulators in the EU/UK/US) anchors this design. The relevant themes — from the [Financial Risk & Compliance Systems guide](../banking/financial_risk_compliance_systems_guide.md) — include:

- **Auditability of the delivery process**: the methodology must produce a defensible record — requirements-to-test traceability (the V-Model's contribution), change records, approval trails, release evidence.
- **Change management**: production changes flow through formal change control; agile's "continuous delivery" must be wrapped in the bank's change framework (which is why banks run release trains and change advisory boards rather than ship-when-ready).
- **Risk management**: risk registers, RCSA-style controls, and escalation paths map naturally onto PRINCE2's risk theme and stage tolerances.
- **Third-party and outsourcing risk**: vendor delivery must fit the same governance envelope — a constraint that pushes procurement-heavy projects toward PRINCE2/PMBOK vocabulary.

The practical message for the architect: **in a bank, "agile transformation" almost never means "remove the gates" — it means "make the space between the gates agile and make the gates themselves faster."**

### 25.4 The Architect's Role in the Methodology

The solution architect navigates methodology rather than owning it. Three concrete roles:

- **Design authority within agile delivery** — the architect owns the technical design decisions that sprints execute; in Scrum terms, the architect works through the Product Owner (prioritizing enablers) and the team (guiding design) without becoming a gate. Architecture decisions are made *incrementally*, with enough runway for the next increments.
- **The "architecture runway" in SAFe** — SAFe gives the architect an explicit home: keeping the runway (enablers, refactoring, infrastructure, non-functional work) ahead of the feature trains so that features land without rework (Section 19.2). The architect's backlog items are first-class SAFe artifacts, planned in PI Planning like any other work.
- **The architect in governance gates** — at SIT/UAT/release gates, the architect is the technical signatory: confirming the design was followed, deviations are recorded, non-functional requirements (capacity, resilience, security) are evidenced. The architect's design documents and decisions become part of the audit trail — which is why architecture decisions should be recorded even in agile delivery (the ADR — architecture decision record — habit).

In short: the architect translates between the delivery world (sprints, backlogs, DoD) and the governance world (gates, evidence, approvals) — fluent in both vocabularies is the job.

---

## 26. Combining Methodologies: The Pragmatic Stack

### 26.1 The Layered Reality

No single methodology covers everything a modern enterprise needs. The standard solution is a **layered stack** — different methods at different altitudes, each doing what it does best:

| Layer | Typical method | What it provides |
|-------|----------------|------------------|
| Knowledge base | PMBOK | The shared vocabulary of what project management involves (risk, scope, stakeholders) |
| Governance | PRINCE2 (or PRINCE2 Agile) | Stage control, business case, roles, audit trail, management by exception |
| Delivery | Scrum (or DSDM/XP practices) | Iterative delivery, feedback, team empowerment |
| Sustaining/ops | Kanban | Continuous flow for support, defects, small changes |
| Portfolio/scaling | SAFe Portfolio (or PMO gates) | Investment decisions, alignment of many streams |

The insight: **these are not competitors — they answer different questions.** PRINCE2 answers "who decides, when, and with what evidence?"; Scrum answers "how does the team build working software?"; Kanban answers "how does ongoing work flow?"; PMBOK answers "what might we be forgetting?" The famous conflicts (agile vs waterfall) are conflicts *within* a layer, not between layers.

### 26.2 A Bank Program's Methodology Stack

Concretely, a large banking program (e.g., a payments modernization or regulatory transformation) typically runs:

```
Portfolio  → SAFe Portfolio / PMO gates (epics, funding, benefits tracking)
Program    → PRINCE2 / PMBOK governance (stages, tolerances, business case, steering committee)
Delivery   → Scrum teams (sprints, DoD incl. compliance criteria) ± XP practices
Support    → Kanban (production incidents, small enhancements, BAU)
```

Every interface is a defined handoff: the program's stage boundary receives evidence from the delivery trains; the Kanban queue absorbs the post-release tail. Done deliberately, the stack is coherent: the governance layer never dictates *how* teams build, and the delivery layer never ignores the evidence the governance layer needs.

### 26.3 Making the Blend Work

- **Name the interfaces** — where does delivery evidence become governance input? (Stage boundary, PI boundary, release gate.)
- **One vocabulary per layer** — don't force Scrum teams to speak PRINCE2 product-description language for everything, nor the board to learn story-point semantics.
- **Keep the audit trail continuous** — every layer must produce the records the next layer (or the auditor) needs.
- **Review the stack periodically** — methodology stacks drift; a quarterly "is this still the process?" review (the retrospective, escalated) keeps the blend deliberate (Section 22.5).

---

## 27. Anti-Patterns

Six recurring ways methodology goes wrong — worth naming because every organization falls into at least one:

1. **Methodology dogmatism** — "pure agile" in a regulated environment (no gates, no audit trail: a compliance failure waiting to happen) or "pure waterfall" in a discovery environment (building the wrong thing). The method is a means; the context is the law. Dogmatism is Section 25's "remove the gates" mistake in reverse.
2. **No methodology (chaos)** — every team improvising, no shared language, no repeatability: the invisible-blend failure from Section 22.5. The cost shows up in handoffs, audits, and attrition.
3. **Ceremony without value (process theater)** — sprint rituals performed with no empirical behavior, stage gates that rubber-stamp, risk registers nobody reads. This is the management-side cousin of the over-orchestration anti-pattern in [Agentic Workflows](agentic_workflows_guide.md): process that consumes time and produces no decision. The Section 22.4 test — *does this change a decision?* — is the antidote.
4. **Framework soup (mixing incompatible practices)** — Scrum sprints + Kanban WIP limits + PRINCE2 stage gates + SAFe PIs, all at once, with no one able to say which governs. Blending is good (Section 26); blending *without ownership* is how "agile transformation" dies of complexity. Pick one spine (Scrum or PRINCE2) and attach practices deliberately.
5. **Ignoring tailoring (one-size-fits-all)** — a single mandated process for a 4-person internal tool and a 200-person regulatory program. PRINCE2's seventh principle and Crystal's whole existence say: scale the ceremony to the risk.
6. **Methodology as identity** — teams fighting over labels ("we're Scrum, not Kanban!") instead of delivering. The label war is a symptom of missing outcome focus; the fix is to talk about *outcomes* (value delivered, evidence produced, defects escaped), not *names*.

## 28. Certifications and Training

### 28.1 The Certification Landscape

Certifications signal *which methodology vocabulary you speak*. The landscape, by body and role:

| Certification | Issuer | Focus | Best for |
|---------------|--------|-------|----------|
| **PMP** (Project Management Professional) | PMI | PMBOK-based; since the 2021 ECO, tests predictive + agile + hybrid | PMs in traditional/hybrid orgs; the global default credential |
| **PMI-ACP** | PMI | Agile practice (Scrum, Kanban, XP, Lean, TDD) without vendor lock | PMs moving agile-ward without joining a specific camp |
| **PRINCE2 Foundation / Practitioner** | PeopleCert (AXELOS) | The PRINCE2 method: principles, themes, processes; Practitioner = apply in a scenario | Governance-focused roles; UK/EU/public sector; PMOs |
| **PRINCE2 Agile** | PeopleCert | PRINCE2 governance + agile delivery | PMs needing both vocabularies (banks, government) |
| **CSM** (Certified ScrumMaster) | Scrum Alliance | Scrum roles/events/artifacts; 2-day course | Scrum Masters and team leads |
| **PSM I / II / III** | Scrum.org | Scrum Master knowledge, no course required, harder exams | Scrum practitioners wanting credential rigor without course cost |
| **SAFe SA / SPC / RTE** | Scaled Agile Inc. | SAFe practitioner, Program Consultant (SPC — can train others), Release Train Engineer | Scaling roles in enterprises; SPC for transformation leads — see the [PMP/SAFe/LSS guide](pmp_safe_lss_comparison_guide.md) |
| **LeSS (CLP)** | Less.works | LeSS practitioner | Scrum Masters scaling with minimal process |
| **KMP (Kanban Management Professional)** | Kanban University (David Anderson's school) | Kanban systems design + coaching | Ops leads, flow-focused managers |
| **AgilePM** | APMG International | DSDM-based agile project management | PRINCE2 holders adding agile PM; agile-with-governance contexts |
| **DASM / DASSM** | PMI (Disciplined Agile) | DA toolkit: goal-driven WoW | PMs wanting a tailored-hybrid toolkit |
| **Lean Six Sigma (Green/Black Belt)** | Various (IASSC, ASQ…) | DMAIC process improvement | Process engineers; pairs with PM for transformation — see the [PMP/SAFe/LSS guide](pmp_safe_lss_comparison_guide.md) |

### 28.2 Choosing What to Certify

The pattern most professionals land on, by career vector:

- **Traditional PM / PMO / program manager** → PMP first, then PRINCE2 Practitioner (if UK/EU/public sector) or PRINCE2 Agile (if hybrid delivery).
- **Agile team roles** → CSM or PSM (Scrum Master), PSM for rigor/cost, CSM for the course-based induction; SAFe SA if your org runs trains.
- **Scaling / transformation leads** → SAFe SPC (if SAFe is the org's spine) or Certified LeSS Practitioner (if product-minimalism is the culture).
- **Architects (this guide's audience)** → none of these are required, but the *vocabulary* matters: PMP or PRINCE2 Foundation for governance literacy, PSM/CSM for team literacy. The architect's value is fluency across layers (Section 25.4), and the certifications are the fastest fluency cheat-sheet.
- **Banks/regulated delivery** → PRINCE2 Agile + AgilePM is a strong combination: governance + agile PM, both auditable vocabularies.

The honest caveat: **certifications certify knowledge of the methodology, not delivery of outcomes.** The credential opens the door; the anti-patterns in Section 27 are where certified teams still fail. Treat certification as vocabulary acquisition, not competence.

---

## 29. Methodology in 2026: Trends

### 29.1 AI in Project Management

AI is entering every layer of the methodology stack: AI-assisted planning (generating WBS structures and sprint plans from goals), estimation (pattern-based effort prediction replacing gut-feel story points), risk identification (scanning plans and histories for risk patterns), status synthesis (drafting steering-committee packs from tickets), and retrospectives (mining sprint data for improvement signals). The "AI PM" is not (yet) a replacement — it is an *amplifier* of the PM's analytic work, and it raises a new governance question: *who audits the AI's plan?* The parallels with agentic workflow orchestration — the over-orchestration risk, the need for human-in-the-loop gates — are covered in [Agentic Workflows](agentic_workflows_guide.md). Expect 2026–2027 delivery tooling to converge planning, tracking, and AI copilots into the same platform.

### 29.2 The Delivery-Ops Convergence: Project → Product → Platform

The industry's center of gravity is shifting from **projects** (finite, PM-owned) to **products** (long-lived, PO-owned, continuously improved) to **platforms** (internal developer platforms and shared capability teams). Consequences for methodology: fewer greenfield "project" artifacts (business case, closure) and more continuous product roadmaps; funding moves from project-based to product-based (SAFe's portfolio model anticipated this); Kanban-style flow and platform teams absorb the long tail. The [Product Management guide](product_management_guide.md) covers the product side of this shift.

### 29.3 Methodology Blending Is the Norm

Hybrid is no longer a compromise — it is the default and increasingly the *named* choice: PRINCE2 Agile and Disciplined Agile legitimized what Water-Scrum-Fall did accidentally. The 2026 PMBOK-era mindset ("choose the approach that fits, tailor it, document the choice") is now the mainstream professional position; the pure-vs-pure wars are a legacy of the 2010s.

### 29.4 Remote and Distributed Delivery

Post-pandemic delivery is distributed as default, which reshapes ceremonies: async standups (written updates), recorded reviews, distributed PI Planning (multi-site, tool-mediated), and deliberate "async-first" culture. The practices that suffer most — pair programming, osmotic communication (Crystal), workshop-heavy DSDM — get tool substitutes (mob sessions, Miro workshops, pairing over shared IDEs). Methodology selection now includes a *distribution dimension* that Crystal's co-location assumptions never anticipated.

### 29.5 Outcome-Based Delivery

OKRs and value streams are displacing output metrics (story points, features shipped) as the language of progress. Consequences: Definition of Done grows outcome clauses; program governance asks "did the business outcome improve?" rather than "is the milestone hit?"; benefit tracking (PRINCE2's business case, SAFe's value-stream KPIs) becomes the norm rather than the exception. The [Product Management guide](product_management_guide.md) and OKR practice are the adjacent references.

### 29.6 Regulated-Agile Maturation

Banking/financial services have moved from "can we do agile in a bank?" (2015–2020) to "how do we run agile *with* compliance gates?" (2021–2026). The maturation markers: regulator guidance explicitly acknowledging agile delivery (with controls); internal audit teams fluent in agile evidence; SAFe/PRINCE2 Agile adoption in the largest institutions; continuous delivery inside a *controlled* release envelope. Section 25's agile-inside/governance-outside pattern is the durable outcome — expect it to keep spreading to insurance, healthcare, and public sector.

---

## 30. Glossary

| Term | Definition |
|------|------------|
| **Waterfall** | Sequential predictive model: requirements → design → implementation → testing → deployment → maintenance, with gates between phases |
| **V-Model** | Waterfall variant pairing each development stage with a corresponding test stage (verification & validation) |
| **Spiral** | Boehm's risk-driven iterative model; each loop = objectives, risk analysis, build/prototype, plan next |
| **RUP** | Rational Unified Process: 4 phases (inception, elaboration, construction, transition) × disciplines, iterative |
| **CPM** | Critical Path Method: schedule analysis via the longest dependency chain (critical path) and float |
| **PERT** | Three-point estimation (optimistic/most-likely/pessimistic; expected = (O+4M+P)/6) for uncertain durations |
| **PMBOK** | PMI's Project Management Body of Knowledge — a knowledge standard (10 knowledge areas / 5 process groups; 7th ed.: 12 principles + 8 performance domains) |
| **PRINCE2** | PRojects IN Controlled Environments — process-based governance method: 7 principles, 7 themes, 7 processes |
| **Scrum** | Agile framework: 3 roles (PO, Scrum Master, developers), 3 artifacts (product backlog, sprint backlog, increment), 5 events within the sprint |
| **Kanban** | Pull-based flow method: visualize, limit WIP, manage flow, explicit policies, feedback loops |
| **XP** | Extreme Programming: engineering practices — pair programming, TDD, continuous integration, refactoring, collective ownership |
| **Lean** | Waste-elimination philosophy (Toyota → Poppendieck): 7 principles of Lean software development |
| **DSDM** | Dynamic Systems Development Method — agile PM with governance: 8 principles, MoSCoW, timeboxing |
| **MoSCoW** | Prioritization scheme: Must / Should / Could / Won't-have |
| **Crystal** | Cockburn's family of tailored methods sized by team size and criticality |
| **FDD** | Feature-Driven Development: model-driven, 5 processes (overall model → feature list → plan/design/build by feature) |
| **SAFe** | Scaled Agile Framework: configurations (Essential/Large Solution/Portfolio/Full), ARTs, PIs, RTE |
| **LeSS** | Large-Scale Scrum: one product backlog, one PO, synchronized sprints, minimal process addition |
| **Nexus** | Scrum.org's 3–9-team scaling: adds the Nexus Integration Team and Nexus Sprint |
| **Scrum@Scale** | Sutherland's scale-free model: accountability circles, Scrum-of-Scrums |
| **Disciplined Agile** | PMI's goal-driven hybrid toolkit: choose your way of working (WoW) per goal |
| **Spotify Model** | Org pattern: squads, tribes, chapters, guilds (not a delivery methodology) |
| **Hybrid** | Deliberate blend — e.g., agile delivery under a governance shell (PRINCE2 Agile, Water-Scrum-Fall) |
| **Tailoring** | Scaling a methodology to context; a first-class principle in PRINCE2, DSDM, Crystal, PMBOK 7 |
| **Sprint** | Scrum's fixed timebox (1–4 weeks) ending in a reviewable increment |
| **Backlog** | Ordered list of work — product backlog (all work) vs sprint backlog (selected for the sprint) |
| **WIP** | Work in progress; Kanban's WIP limits cap items per workflow state to expose bottlenecks |
| **ART** | Agile Release Train — SAFe's 5–12-team delivery unit |
| **PI** | Program Increment — SAFe's 8–12-week planning/delivery cadence, opened by PI Planning |
| **Ceremony** | A recurring meeting/ritual in a methodology (sprint review, stage boundary, PI Planning) |
| **Governance** | The control layer: who decides, with what evidence, within what tolerances |
| **DoD** | Definition of Done — the team's shared quality bar for "done" |
| **Gates** | Formal go/no-go review points (phase gates, stage boundaries, release gates) |
| **Flow** | The movement of work through a system; Kanban manages flow, not iterations |

---

## 31. References and Further Reading

### 31.1 Sibling Guides in This Repository

- [PMP vs SAFe vs Lean Six Sigma: A Comprehensive Comparison Guide](pmp_safe_lss_comparison_guide.md) — certification deep-dive for the PMP/SAFe/LSS trio: exam details, cost/benefit, decision tree.
- [The Scaled Agile Framework (SAFe): A Comprehensive Guide](scaled_agile_framework_guide.md) — the full SAFe deep-dive: configurations, roles, PI Planning, banking adoption.
- [Product Management](product_management_guide.md) — the product-side view of delivery: roadmaps, discovery, OKRs.
- [Sales Methodology Frameworks](sales_methodology_frameworks_guide.md) — the companion professional-skills guide for the pre-sales side of the same bank context.
- [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) — MAS/regulatory context for the compliance angle in Section 25.
- [Agentic Workflows](agentic_workflows_guide.md) — the over-orchestration anti-pattern analog for AI-delivered work (Section 27, anti-pattern 3 — process theater).
- [Communication & Stakeholder Management](communication_stakeholder_management_skills_guide.md) — the stakeholder-engagement skills that every methodology assumes.

### 31.2 Primary Sources

- *A Guide to the Project Management Body of Knowledge (PMBOK Guide)*, PMI, 7th ed., 2021.
- *Managing Successful Projects with PRINCE2*, AXELOS/PeopleCert, 6th ed., 2017; 7th ed., 2023.
- *PRINCE2 Agile*, AXELOS, 2015.
- Royce, W. W., "Managing the Development of Large Software Systems", 1970.
- Boehm, B., "A Spiral Model of Software Development and Enhancement", IEEE Software, 1986.
- *The Scrum Guide*, Schwaber & Sutherland, 2020.
- Beck, K., *Extreme Programming Explained*, 1999.
- Poppendieck, M. & T., *Lean Software Development*, 2003.
- Cockburn, A., *Crystal Clear*, 2004.
- Anderson, D., *Kanban: Successful Evolutionary Change*, 2010.
- Kniberg, H. & Ivarsson, A., "Spotify Engineering Culture" (blog), 2012.
- Larman, C. & Vodde, B., *Scaling Lean & Agile Development* (LeSS), 2008.
- The Agile Manifesto, agilemanifesto.org, 2001.

---

*Methodology is the least interesting thing about delivery — and the most consequential. The frameworks in this guide are vocabularies for control, learning, and evidence; the actual work is choosing the right blend, tailoring it to the risk, and keeping it honest. Facts and versions as of August 2026 — verify against official sources (PMI, PeopleCert/AXELOS, Scaled Agile Inc., Scrum.org) before certification or adoption decisions.*






