# Business Case Development: The Business-Case Discipline — A Comprehensive Guide

*A deep dive into the business case — the decision-support document at the heart of every funded initiative — covering the purpose (the definition, the decision-support role, the overview table), the frameworks (the HM Treasury Five Case Model and its peers, the framework table), the structure (the six sections, the structure table), the financials (TCO, NPV, IRR, payback, ROI, the financials table), the options appraisal (the do-nothing baseline, the options table), the risk analysis (sensitivity analysis, the risk table), the stakeholders (the stakeholder map, the stakeholders table), the worked example (a CACIB-style platform investment, the TCO–NPV worked case, the lessons), the summary (the one-page, 'the case for the case'), and the glossary.*

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** management/ — the business and consulting skills cluster
> **Audience:** architects, technologists, and working professionals who must write, defend, or challenge investment cases
> **Last Updated:** August 2026

---

## Cross-References (the repo guides this one maps into)

- [data_architect_skillgaps_guide.md](../technology/data_architect_skillgaps_guide.md) — **the gap analysis that motivated this guide**: business case & TCO modeling for data initiatives is confirmed as a real skill gap (#2 of the top-10, a standing priority), with `finops_guide.md` covering cloud cost but no dedicated business-case/TCO methodology until now. This guide is the commissioned companion.
- [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) — **the financial-modeling vocabulary**: §6 (the Quant Skills) is the DCF/NPV/WACC/CAPM chain this guide's §4 builds on; its §6.6 worked example is the arithmetic this guide's §8 extends into a full investment case.
- [cfa_program_guide.md](../banking/cfa_program_guide.md) — the finance vocabulary at analyst depth (cross-ref lightly); the same math the CFA teaches as valuation, the business case teaches as decision support.
- [mckinsey_approach_guide.md](mckinsey_approach_guide.md) and [management_consulting_skills_guide.md](management_consulting_skills_guide.md) — the consulting-structure pattern (issue trees, the MECE discipline, the "so what" test) that makes a business case board-readable.
- [vendor_management_guide.md](vendor_management_guide.md) — the TCO angle from the vendor side: contract economics, license models, exit costs, negotiation leverage.
- [finops_guide.md](../technology/finops_guide.md) — the ongoing-cost angle: unit economics, showback/chargeback, cloud cost governance — the operational half of any platform TCO.
- [strategic_management_guide.md](strategic_management_guide.md) — strategy alignment: the strategic case must trace to the firm's objectives (Porter, the value chain, the balanced scorecard).
- [risk_management_models_guide.md](../banking/risk_management_models_guide.md) — the risk-quantification angle for §6 (likelihood/impact, stress testing, the banking risk vocabulary).
- [universal_banking_model_guide.md](../banking/universal_banking_model_guide.md) — cross-ref lightly: the business lines (trade finance, payments, markets) whose P&L a banking investment case touches.
- [trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md) — **the worked-example context**: the systems landscape of the §8 trade-finance platform case.
- The study-guide genre (cross-ref lightly for the structure pattern): [grokking_system_design_companion_guide.md](../technology/grokking_system_design_companion_guide.md), [ddia_study_companion_guide.md](../technology/ddia_study_companion_guide.md), [system_design_interview_insiders_guide.md](../technology/system_design_interview_insiders_guide.md).

---

## Conventions: Verified vs Framed vs Flagged

This guide follows the repo convention of marking epistemic status inline:

- **Verified** — checked against a primary or named source this edition (gov.uk/HM Treasury publications, Gartner research notes, the standard project-management and corporate-finance references, the press).
- **(framed)** — standard practice or well-established knowledge stated at the level of the framework's own literature, not re-verified against a primary source this edition.
- **(flag)** — a claim that is commonly cited but could not be pinned to a single authoritative source this edition; stated honestly with the nuance.

**Web verification performed for this edition:** HM Treasury business-case guidance and the Green Book (gov.uk, current edition May 2024 / updated June 2026), the Five Case Model's international adoption (OECD Infrastructure Toolkit), Gartner TCO research, PMBOK-based project-initiation and stakeholder practice, the consultant business-case literature (umbrex, deckary, clarispec), and the sensitivity-analysis / do-nothing-baseline practice sources. Everything else is standard discipline knowledge marked accordingly.

---

## Table of Contents

1. [The Purpose](#1-the-purpose) — the definition, the decision-support role, the overview table
2. [The Frameworks](#2-the-frameworks) — the Five Case Model, the framework table
3. [The Structure](#3-the-structure) — the six sections, the structure table
4. [The Financials](#4-the-financials) — TCO, NPV, IRR, payback, ROI, the financials table
5. [The Options Appraisal](#5-the-options-appraisal) — the do-nothing baseline, the options table
6. [The Risk Analysis](#6-the-risk-analysis) — sensitivity analysis, the risk table
7. [The Stakeholders](#7-the-stakeholders) — the stakeholder map, the stakeholders table
8. [The Worked Example](#8-the-worked-example) — the CACIB-style platform case, the TCO–NPV worked case, the lessons
9. [The Summary](#9-the-summary) — the one-page, 'the case for the case'
10. [The Glossary](#the-glossary)

---

## 1. The Purpose

### 1.1 The Definition (verified — the decision-support)

A **business case** is a structured decision-support document that assembles, in one place, everything a decision-maker needs to approve, reject, defer, or reshape a proposed investment: the problem, the strategic context, the options considered, the quantified costs and benefits, the risks, and a clear recommendation. **(Verified —** HM Treasury's *Guidance on developing business cases for projects and programmes* defines the business case as the vehicle that supports decisions on whether and how to proceed with a project or programme, setting out the options and the evidence for the recommended one (gov.uk); the Project Management Institute's standards treat the business case as the documented economic feasibility study used to establish the validity of the benefits of a project and to authorize funding (PMBOK-based project-initiation practice, verified in substance; flagged — PMI's guidance is dispersed across its standards rather than one publication). The consulting literature agrees: a business case is "the argument for action" — the case for change, quantified.)

The word that carries the discipline is **decision support**. A business case is not a project plan, not a budget, not a sales pitch, and not a post-hoc justification of a decision already made (the failure mode to avoid at all costs — see §1.4). It is the *evidence pack* for a capital-allocation decision: it converts an intuition ("we should modernize trade finance") into a comparable, challengeable proposition ("this option creates S$6.9M of NPV over five years at a 10% hurdle rate, with the risks managed as follows, and the board should fund it now").

Three properties follow from the decision-support definition:

1. **It is comparative, not absolute.** A business case earns its keep by comparing options — including doing nothing — on the same basis. An investment that looks good in isolation often looks different next to its alternatives (the §8 worked example shows an option with the *highest* IRR that is *not* the best decision).
2. **It is decision-grade, not perfect.** The goal is a transparent, challenge-ready model that finance can own and executives can attack — not a "perfect" forecast. (The consultant practice literature says exactly this: "the goal is not to build a 'perfect' model, but to build a transparent, decision-grade economics view" — verified, umbrex.)
3. **It is an argument, not a form.** The best business cases read like a well-structured consulting argument: situation → complication → question → answer (cross-ref [mckinsey_approach_guide.md](mckinsey_approach_guide.md)). The form is scaffolding; the argument is the deliverable.

### 1.2 What It Is For (the five jobs of a business case)

| Job | What the business case does |
|-----|-----------------------------|
| **The funding gate** | It is the document that unlocks capital: no approved business case, no budget. It is the discipline that forces an organization to decide *on the evidence* rather than on sponsorship. |
| **The accountability contract** | It fixes the promised costs, benefits, and timing in writing. After approval it becomes the baseline against which the project is governed and its benefits are realized and audited (post-implementation review). |
| **The alignment device** | It forces the proposal to trace to strategy: which strategic objective does this serve, and how? (cross-ref [strategic_management_guide.md](strategic_management_guide.md)) |
| **The communication artifact** | It is how a sponsor talks to a board, an architect talks to a CFO, a program talks to an audit committee. One document, many audiences. |
| **The risk-and-options record** | It preserves the rejected options and the identified risks, so future decision-makers know what was considered and why — the institutional memory of the decision. |

### 1.3 The Timing (verified — the iterative case)

Business cases are not written once. The UK public-sector practice — and the wider discipline — develops the case in stages of increasing depth and cost-estimate confidence, each supporting a different decision gate (verified — HM Treasury guidance; the standard "three cases" ladder):

1. **Strategic Outline Case (SOC)** — the early-stage case: is there a case for change at all? Rough cost band (±40–50%), options identified, not yet appraised. Decision gate: proceed to explore options.
2. **Outline Business Case (OBC)** — the options are appraised and a preferred option emerges; cost band tightens (±20–30%). Decision gate: confirm the preferred way forward.
3. **Full Business Case (FBC)** — the detailed, contract-ready case: final costs, agreed commercial arrangements, implementation plan. Decision gate: release funds and sign contracts.

Private-sector organizations compress this into a lighter ladder — "concept → proposal → funding request" — but the principle is identical: **the case gets more precise as the decision gets more expensive to reverse.** A multi-million-dollar platform investment deserves a full case; a tool subscription deserves a one-pager. The discipline is *proportionate* (a word the Treasury guidance itself uses — verified).

### 1.4 The Overview Table

| Aspect | Description |
|--------|-------------|
| **Definition** | A structured decision-support document justifying a proposed investment or change: problem, options, costs, benefits, risks, recommendation |
| **Purpose** | To enable an evidence-based funding decision and to fix the accountability baseline for delivery and benefits realization |
| **Core audience** | The decision-maker or investment committee (the board, the CFO, the sponsor), plus finance, audit, and the delivery team |
| **Essential property** | Decision-grade, not perfect: transparent assumptions, comparable options, challenge-ready numbers |
| **Decision it enables** | Approve / reject / defer / reshape — with the reasoning recorded |
| **Relationship to other documents** | Input to the project charter/PID (PMI practice, verified in substance); the source of the benefits baseline for benefits realization; the ancestor of the post-implementation review |
| **Temporal nature** | Iterative: strategic outline → outline → full case, each more precise (verified — HM Treasury guidance); maintained throughout the delivery lifecycle (the PRINCE2 business-case theme — framed) |
| **Owner** | The sponsor (senior accountable executive), not the analyst or the PMO — the sponsor owns the argument, the analyst builds the model |
| **What it is NOT** | A project plan, a budget spreadsheet, a sales pitch, or a post-hoc justification of a decision already taken |
| **Success criterion** | A board can read the executive summary, interrogate the financials and risk sections, and make the call in one sitting |

**The one-line summary of the purpose (framed):** *a business case exists so that money moves only when the evidence says it should, and so that, once moved, the promise can be checked.*

### 1.5 The Business Case and Its Neighbours (framed — the document ecosystem)

A business case is frequently confused with the documents around it. The distinctions matter because each document has a different owner, audience, and lifecycle — and the committee will notice if the case is doing another document's job:

| Document | What it is | How it relates to the business case |
|----------|-----------|-------------------------------------|
| **Business case** | The decision-support argument: why, what, how much, which option, at what risk | The ancestor of all of the below; the funding gate |
| **Project charter / PID** | The authorization to start delivery: scope, objectives, roles | Descends from the business case; PMI practice: business case → charter (verified in substance) |
| **Benefits-realization plan** | Who owns each benefit, how it is measured, when it is reviewed | The business case's promises, operationalized; the case is its baseline |
| **Budget / financial plan** | The funded envelope: where the money sits in the books | The case's affordability answer, booked; the case explains *why*, the budget records *what* |
| **BRD / requirements spec** | What the solution must do | The case's preferred option, decomposed; the case precedes the requirements |
| **Project plan / schedule** | When delivery happens | The case's implementation section, expanded; the case precedes the plan |
| **Post-implementation review** | Did it deliver what the case promised? | The case's accountability loop closes here — the case is the yardstick |

The rule (framed): **the business case is written to answer "should we?" — everything else answers "how."** When a draft business case reads like a requirements document or a schedule, it has left its lane.

### 1.6 The Failure Modes (framed — how business cases die)

The literature and committee-room experience converge on a short list of ways cases fail. Knowing them is the first defense:

1. **The post-hoc justification** — the decision was already made; the case is written to dress it up. Committees detect this instantly and it poisons the entire document's credibility. The antidote: options the decision-maker did *not* choose must be visible and fairly treated.
2. **The one-option case** — only the preferred option appears, with no do-nothing baseline and no alternative. This is not a case; it is a request.
3. **Unquantified benefits** — "improved efficiency" without a number, an owner, or a measure. Every benefit in the case must pass the "so what, how much, who owns it" test (§3.5).
4. **The flattering metric** — the case leads with whichever metric (usually ROI or payback) makes the option look best, hiding the NPV the committee's finance people will compute anyway (§4).
5. **The frozen baseline** — do-nothing modeled as "nothing changes," inflating every option's apparent benefit (§5.1).
6. **The spreadsheet with no story** — 40 tabs, no argument; the case cannot be summarized in one page because it was never an argument (§3.2).
7. **The disappearing risk section** — risks listed but never connected to the case's numbers (no sensitivity, no bear case), so the committee assumes the worst (§6).

A useful self-test before submission (framed): *imagine the CFO's three questions — "where does this number come from?", "what happens if it's 20% wrong?", "who owns it if it is?" — and write the case so all three have visible answers.*

---

## 2. The Frameworks

### 2.1 The Five Case Model (verified — HM Treasury)

The dominant business-case framework in public-sector and much international practice is the **Five Case Model (5CM)**, developed and mandated by **HM Treasury** for UK government spending proposals and embedded in the Treasury's *Green Book* (the government's appraisal guidance) (verified — gov.uk: "This guidance sets out the step-by-step process for developing business cases for projects and programmes, based on the Five Case Model"; the Green Book page: "The Green Book uses the five-case model as outlined in the business case guidance"). It has been adopted by the Welsh Government and was recommended by the UK Office of Government Commerce (verified — Praxis Framework), spread across Commonwealth and European public sectors, and was endorsed by the G20 as an international standard for preparing infrastructure projects in 2018 (verified — OECD Infrastructure Toolkit case study).

**Timeline and version (flag):** the model's lineage runs back to HM Treasury's PFI-era appraisal practice of the late 1990s/2000s; the versions most commonly cited are the 2011 *Green Book* edition and the 2013 business-case guidance (the gov.uk record shows updated guidance "published 18 April 2013"); the guidance series was refreshed in 2018 and the current edition was published May 2024 and updated June 2026 in line with the 2025 Green Book review (verified — gov.uk publication history). **Flag:** the "2011" that appears in most citations refers to the published guidance/Green Book version, not the invention date of the model itself, which no single primary source pins down; treat "2011" as "the commonly cited version anchor."

The five cases answer five questions (verified — the Treasury guidance structure; APMG's summary is representative):

1. **The Strategic Case — *Is it needed?*** The case for change: the strategic context, the problem/opportunity, the benefits sought, and the fit with organizational objectives. It answers "why are we doing this at all?" and "what happens if we don't?"
2. **The Economic Case — *Is it the best option?*** The options appraisal: the do-nothing baseline, the options shortlist, cost–benefit analysis, and the identification of the preferred option as the one that maximizes value for money (the Green Book's net-present-social-value lens — framed).
3. **The Commercial Case — *Is it achievable commercially?*** The procurement and contracting strategy: how the option will be sourced (buy/build/partner), the deal structure, the service model, the contract and its incentives, and supplier capability.
4. **The Financial Case — *Is it affordable?*** The funding picture: total cost of ownership, capital vs operating expenditure, the affordability envelope, and the impact on the organization's financial position — the answer to "can we pay for this, and from which pocket?"
5. **The Management Case — *Can it be delivered?*** The delivery plan: governance, roles, project/programme management approach, benefits-realization arrangements, and the plan for managing the change into the business.

The genius of the model is the **interlock**: each case is a gate for the next. If the strategic case fails ("not needed"), the economic case is moot; if the economic case fails ("not the best option"), the commercial case is moot; if the commercial case fails ("not achievable"), the financial case is moot; if the financial case fails ("not affordable"), the management case is moot. It is a funnel, and the funnel is the discipline — the framework *forces* the do-nothing baseline into the analysis (see §5) and forces benefits to be owned (see §3.5).

### 2.2 The Business Case in Other Frameworks

- **PMI / PMBOK (flag):** the PMI body of knowledge treats the business case as the documented economic feasibility study that establishes the benefits' validity and authorizes the project — an input to project initiation and the project charter (verified in substance via project-management sources; flagged because PMI's business-case treatment is dispersed across the PMBOK Guide and its practice standards rather than a single canonical publication). PMP-exam practice pairs the business case with NPV/IRR/benefit-cost-ratio (BCR) and stakeholder analysis at initiation — the same vocabulary as §4 and §7.
- **PRINCE2 (framed):** the business case is one of the seven PRINCE2 themes — the *justification* for the project, created at start-up and maintained continuously: if the business case stops justifying the project, the project should stop. The closest thing to a "living document" discipline in the frameworks.
- **ITIL 4 (framed):** ITIL 4 includes the business case as a general management practice — the justification of value for service-management initiatives and continual improvement, estimating costs, benefits, and risks before committing resources. Standard framework knowledge; not re-verified against the published ITIL 4 standard this edition.
- **TOGAF (framed):** in TOGAF's Architecture Development Method (ADM), the business case is a core artifact — seeded in Phase A (Architecture Vision) and refined through the later phases as the architecture matures, so that the architecture is only implemented when its business case holds. Standard framework knowledge; the ADM phases are well documented (verified in substance by the TOGAF practice literature this edition).
- **Praxis (verified):** the Praxis Framework (the UK-based body of knowledge for project/programme management) treats the business case as a key management document and explicitly recommends the Five Case Model (verified — the Praxis Framework's complementary guidance page: the 5CM "is the approach for developing business cases recommended by HM Treasury, the Welsh Government and the UK Office of Government Commerce").
- **The consulting "case for action" (framed):** McKinsey-style change practice frames the opening of any change program as the *case for action* — the burning platform, quantified — which is the strategic case in consulting clothes (cross-ref [mckinsey_approach_guide.md](mckinsey_approach_guide.md) and [management_consulting_skills_guide.md](management_consulting_skills_guide.md)).

### 2.3 The Framework Table

| Framework | Origin | Use |
|-----------|--------|-----|
| **Five Case Model (5CM)** | HM Treasury (UK) — codified in the Green Book (2011 edition; guidance 2013/2018/2024 — flag on the exact first-publication year; G20-endorsed 2018) | The standard for UK public-sector and international infrastructure/investment cases; the five-case funnel: strategic → economic → commercial → financial → management |
| **Green Book** | HM Treasury, UK (first published 1991; major revisions 2003, 2011, 2018, 2020, 2022, 2025 review) | Appraisal and evaluation of policies, programmes, and projects; the home of the 3.5% social time preference discount rate (framed) and optimism-bias adjustment (framed) |
| **PMBOK / PMI standards** | Project Management Institute, US (first PMBOK 1987; current editions 7th ed. 2021 + process/standards family) (flag — editions not re-verified) | Project initiation: business case → project charter; NPV/IRR/BCR at the funding gate; stakeholder analysis (the power/interest grid lives here — §7) |
| **PRINCE2** | UK government (OGC, 1996; PRINCE2 2017 refresh — framed) | The business case as a continuous theme: justification maintained from start-up to closure |
| **ITIL 4** | AXELOS (2019) (framed) | Service-management practice: business case for service changes and continual improvement |
| **TOGAF** | The Open Group (TOGAF 1.0 1995; 9.2 2018; 10th ed. 2022 — framed) | Enterprise architecture: the business case artifact in the ADM, refined as the architecture matures |
| **Praxis** | Praxis Framework (UK, 2014 — framed) | Project/programme body of knowledge that explicitly endorses the Five Case Model |
| **The consulting case for action** | McKinsey-style change practice (framed; cross-ref the consulting guides) | The burning platform, quantified — the strategic case for organizational change |

**The architect's takeaway (framed):** you do not need to pick a framework — you need to know which one your *organization* uses, and map the others onto it. A CACIB-style investment committee may not say "strategic case," but it will ask "why now, why this option, can we afford it, can we deliver it, and who owns the benefits?" — which *is* the Five Case Model wearing a banker's suit. This guide's structure (§3) is framework-agnostic for exactly that reason.
---

## 3. The Structure

### 3.1 The Six Sections (verified in substance — the standard anatomy)

Business-case templates differ in naming, but the anatomy is remarkably stable across the public-sector guidance, the project-management bodies, and the consulting practice literature. The consultant guides are explicit: a business case that gets approved carries (i) a one-page executive summary, (ii) a quantified problem statement, (iii) an options analysis that includes "do nothing," (iv) financial metrics (NPV, IRR, payback, ROI) with sensitivity analysis, (v) a risk assessment, and (vi) a clear recommendation (verified — deckary's business-case template guide enumerates exactly these six elements; the same skeleton appears in the Treasury's five-case structure when compressed, framed). The six sections below are that anatomy, named for this guide:

1. **The Executive Summary** — the decision, the ask, and the evidence, on one page.
2. **The Problem (the case for change)** — the quantified pain, the strategic context, the cost of inaction.
3. **The Options** — the alternatives considered, including do-nothing, on a comparable basis.
4. **The Analysis** — the financial model (TCO/NPV/IRR/payback/ROI), the non-financials, the risk and sensitivity analysis, the stakeholder read.
5. **The Recommendation** — the preferred option, the reasoning, the conditions, the confidence level.
6. **The Implementation** — the plan, timeline, governance, and benefits realization.

The six sections are an *argument in order*: the executive summary states the conclusion, the problem establishes why now, the options show the field was searched, the analysis shows the numbers were done, the recommendation commits, and the implementation shows it can be delivered. **A business case whose sections can be shuffled without loss of meaning is not a business case — it is a dossier.** (framed)

### 3.2 The Executive Summary (verified — the one-page discipline)

The executive summary is the most important page in the document because it is the only page the decision committee is guaranteed to read. Its discipline:

- **Decision first.** Open with the ask: "We recommend funding Option 2 — the acquisition of a commercial trade-finance STP platform — at S$8.0M capex, generating S$6.9M NPV over five years at a 10% hurdle rate, with payback at 3.1 years." If the reader learns nothing else, they learn the decision.
- **One page, hard limit.** If it takes two pages, the argument is not yet clear — the compression itself is the test.
- **The five numbers that matter:** the investment, the NPV, the IRR, the payback, and the risk headline ("downside case remains positive at S$3.1M NPV").
- **No new arguments in the summary.** Everything in it must appear, evidenced, in the body. (The classic failure: the summary promises benefits the analysis does not support.)

### 3.3 The Problem (verified — the quantified case for change)

The problem section answers three questions, and the first is the one most business cases get wrong:

1. **What is the pain, quantified?** Not "the legacy platform is old," but "the legacy platform carries a S$4.8M/yr operating cost that is growing 4%/yr; manual touch costs S$2.1M/yr in rework; STP rate is 35% vs a 60%+ market benchmark, costing S$1.4M/yr in lost share." Quantification is what turns a complaint into a case (the consulting "so what" test — cross-ref [management_consulting_skills_guide.md](management_consulting_skills_guide.md)).
2. **What is the strategic context?** Which objective does this serve, and what does the market/regulator/competition say is coming? (cross-ref [strategic_management_guide.md](strategic_management_guide.md))
3. **What happens if we do nothing?** The cost of inaction — not "nothing changes" but "the run-rate worsens": volumes grow, vendor support premiums escalate, the risk of a regulatory finding or an outage rises. The answer to this question *is* the do-nothing baseline (§5.1).

### 3.4 The Options and The Analysis (verified — the heart of the case)

The options section presents the field (typically three to five options including do-nothing — §5), and the analysis section works it over: the financial model (§4), the non-financial criteria (strategic fit, feasibility, operational risk, timing), the sensitivity analysis (§6.1), and the stakeholder read (§7). The analysis section is where the case is won or lost in committee: this is the section the CFO and the risk director will attack, so it must be **transparent about assumptions** (each assumption named, sourced, and stress-tested) and **even-handed about options** (the recommended option wins on the evidence, visibly, not on advocacy).

### 3.5 The Recommendation and The Implementation (verified — commit and deliver)

The recommendation states the preferred option, the reasoning in one paragraph, and — critically — **the conditions**: what must be true for the case to hold (sign-off on the vendor contract, the migration window, the headcount plan), and the confidence level ("the case is robust to a 20% benefit shortfall"). The implementation section shows the delivery skeleton: phases and timeline, governance (who decides what, when), the delivery organization, the change-management plan, and — the part most business cases forget — **the benefits-realization plan**: who owns each benefit, how it will be measured, when it will be reviewed, and what happens if it does not materialize (the PRINCE2 business-case theme's "justification must persist" — framed). A case with no named benefits owner is a case with no benefits.

### 3.6 The Structure Table

| Section | Purpose | Notes |
|---------|---------|-------|
| **Executive summary** | State the decision, the ask, and the headline evidence | One page, hard limit; decision first; the five numbers (investment, NPV, IRR, payback, risk headline); no new arguments |
| **Problem / case for change** | Establish why now, quantified, and aligned to strategy | Quantify the pain and the cost of inaction; name the strategic objective; the do-nothing consequence is the bridge to §5 |
| **Options** | Show the field was searched, on a comparable basis | 3–5 options including do-nothing; common evaluation criteria; options the committee will propose later should already be in here |
| **Analysis** | Work the options over: financials, non-financials, risk, stakeholders | Transparent assumptions; incremental cash flows vs baseline; sensitivity analysis; the CFO's attack surface |
| **Recommendation** | Commit to a preferred option with reasoning and conditions | One-paragraph rationale; named conditions; confidence level; what would change the recommendation |
| **Implementation** | Show it can be delivered, and benefits will be realized | Timeline, governance, delivery org, change management, benefits-realization plan with named owners |
| *(Not a section — the appendix)* | The full financial model, the detailed risk register, the stakeholder plan | The body argues; the appendix evidences. Freeze the model version and reference it |

**The board-reading pattern (framed):** committees read the executive summary, then jump to the financials and the risk section, then interrogate the recommendation's conditions. The problem section matters most to the sponsor's own leadership; the options section matters most to audit. Structure for those reading patterns.

### 3.7 How Committees Actually Read a Business Case (framed)

The committee session is a different genre from the document, and the document is read accordingly. The practical reading patterns of a senior investment committee:

- **The executive summary is read twice** — once before the meeting (the decision gate: is this worth my hour?) and once during (the anchor for questions). Everything the committee will fight about should be visible in it: the ask, the NPV, the risk headline.
- **The financials are read adversarially.** The CFO's team will have pre-read the model: discount rate consistency, incremental-vs-absolute flows, double-counted benefits, the baseline's integrity. The case that survives finance's pre-read is the case that gets a hearing.
- **The risk section is read for the bear case.** Committees want the sentence that says what happens if the optimism is wrong — and whether the decision survives it (§6.3).
- **The recommendation's conditions are read as the contract.** The conditions are what the committee will hold the sponsor to at the post-implementation review; they are quoted back verbatim a year later.
- **The options section is read for omissions.** Any option a committee member can think of that is not in the case becomes the case's vulnerability (§5.2).

The design consequence (framed): write the executive summary last but *design* it first — it defines what the rest of the case must prove; write the financials as if the CFO's analyst is auditing them; write the risk section as if the decision already went wrong.

### 3.8 The Submission Checklist (framed — the case is ready when…)

| # | Check | Where |
|---|-------|-------|
| 1 | The executive summary fits one page, leads with the decision, and contains no claim the body does not evidence | §3.2 |
| 2 | The problem is quantified, and the cost of inaction is explicit | §3.3 |
| 3 | Do-nothing is modeled at run-rate as the baseline, and all options are deltas against it | §5.1 |
| 4 | Three to five options are appraised on common criteria; the recommended option wins visibly | §5 |
| 5 | NPV is the headline metric, with IRR/payback/ROI supporting; the discount rate is named and consistent | §4 |
| 6 | Sensitivity analysis identifies the critical assumptions; the bear case stays readable | §6 |
| 7 | Every benefit has a named owner and a measure; the benefits-realization plan is attached | §3.5 |
| 8 | The stakeholder map exists and the case answers the key stakeholders' questions | §7 |
| 9 | The model is frozen, versioned, and referenced; finance can audit it in an afternoon | §4.7 |
| 10 | The one-page summary writes itself from the analysis (§9.1) | §9 |

---

## 4. The Financials

### 4.1 Total Cost of Ownership — TCO (verified — Gartner)

**TCO is the comprehensive assessment of all costs of owning and operating an asset or capability across its lifecycle** — acquisition, deployment, operations, maintenance, support, training, and disposal — across enterprise boundaries and over time. **(Verified —** Gartner defines IT TCO as "a comprehensive assessment of information technology (IT) or other costs across enterprise boundaries over time. For IT, TCO includes hardware and software acquisition, management and support, communications, end-user expenses and the cost of service downtime, security, and risk" (Gartner research; the definition is reproduced across the Gartner TCO literature, e.g., *Defining Gartner Total Cost of Ownership* by Mieritz & Kirwin). **Origin (flag):** the TCO concept was popularized by the Gartner Group in 1987 as a model for evaluating the lifetime cost of personal-computer fleets (cited consistently across industry sources — the EPA's TCO modeling guidance, Stratrix, the software field guides — flag: the 1987 date is the widely cited origin, while Gartner's first authoritative research report on PC TCO dates to the mid-1990s; the *defining* research note is the Mieritz/Kirwin one above). The formal name for the intuition "the sticker price is not the price.")

For an IT investment, the TCO components are (framed, standard practice):

- **Acquisition:** license or build cost, hardware, one-time implementation and integration, data migration.
- **Operations:** hosting/cloud (compute, storage, network — the FinOps angle, cross-ref [finops_guide.md](../technology/finops_guide.md)), run-the-bank staff, batch and support operations.
- **Maintenance and support:** annual license and support fees, vendor maintenance, patching, upgrades, version management.
- **End-user costs:** training, productivity ramp, the cost of the users' time consumed by the tool (a cost almost always forgotten).
- **Disposal/exit:** decommissioning, data extraction, exit fees, the cost of switching away later (the vendor-lock-in angle, cross-ref [vendor_management_guide.md](vendor_management_guide.md)).

The TCO discipline is what makes a buy-vs-build-vs-outsource comparison honest: the buy option's license fee is visible, but its five-year support stream, integration cost, and exit cost are not — until you do the TCO. The §8 worked example shows a case where the *build* option has the lowest operating cost but the highest TCO.

### 4.2 Net Present Value — NPV (verified)

**NPV is the sum of all future cash flows, discounted back to today, minus the initial investment.** It answers: in today's money, how much value is created after covering all costs? (Verified — standard corporate-finance definition; the consultant literature states it identically: "NPV is the sum of all future cash flows discounted back to today, minus the initial investment"; cross-ref [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §6.2, which marks the NPV rule as "the single most important decision rule in the MBA finance core.")

$$NPV = \sum_{t=0}^{T} \frac{CF_t}{(1+r)^t}$$

where $CF_t$ is the net cash flow in year $t$ (negative in investment years), $r$ is the discount rate (usually the firm's cost of capital / WACC), and $T$ is the appraisal horizon.

**The decision rule (verified):** accept projects with **NPV > 0** — they create value at the discount rate; among mutually exclusive options, prefer the highest NPV. NPV is the only capital-budgeting criterion that handles scale, timing, and risk consistently — which is why it is the master decision rule and IRR, payback, and ROI are supporting metrics. (Verified in substance — the MBA guide marks NPV as unambiguous about value creation; the consultant literature: "Use IRR primarily as a supporting metric to NPV, not the sole decision point.")

**Practice rules for the business case (verified — the consultant financials chapter):**

- Use the corporate discount rate agreed with finance (often WACC — cross-ref the MBA guide's §6.3), applied **consistently across all options and scenarios**.
- Model **incremental cash flows vs the baseline** — the do-nothing case — not absolute corporate cash flows.
- Show both total NPV over the horizon and the NPV profile over time (how quickly the case goes positive).
- For riskier cases, show risk-adjusted NPV (downside scenarios) as well.

### 4.3 Internal Rate of Return — IRR (verified)

**IRR is the discount rate at which NPV = 0** — the annual return that would make the organization indifferent between investing and not. It answers: what return does this initiative earn on the money it consumes? (Verified — standard definition; the consultant literature: "IRR is the discount rate at which NPV = 0"; cross-ref the MBA guide.)

**The decision rule:** accept if IRR > the hurdle rate (the firm's required return). **The caveats (verified):** IRR is sensitive to the *timing* of cash flows (front-loaded benefits inflate IRR), it can be misleading or undefined for non-normal cash flows (multiple sign changes → multiple IRRs), and it systematically favors small quick wins over large value creation (the §8 worked example shows this exact trap: the cheapest option has the highest IRR at 50.8% but the lowest NPV). Use IRR for relative attractiveness and communication ("39% vs our 12% hurdle"), never as the sole decision point.

### 4.4 Payback (verified)

**Payback is the time it takes for cumulative net cash flow to become positive** — the moment the investment has paid for itself. (Verified — standard definition; the consultant literature: "Payback is the time it takes for cumulative net cash flow to become positive (in nominal or discounted terms — be explicit which you are using).")

- **Simple payback** uses nominal cash flows; **discounted payback** uses discounted flows and is the honest version (it respects the time value of money).
- **Why it matters:** simple to understand, executives think in payback terms, short payback reduces exposure to long-term uncertainty; often used as a threshold ("we only accept payback under 4 years").
- **Limitations (verified):** it ignores value *after* the payback point, biasing decisions against long-lived, value-accretive investments (a platform with a 3-year payback and a 10-year life gets punished for the same profile that rewards it). Show payback as a complement to NPV and IRR; if payback is long (infrastructure, major platforms), explain why that is acceptable — strategic necessity, regulatory requirement, or transformation value.

### 4.5 Return on Investment — ROI (verified in substance; flagged as the loosest metric)

**ROI is (benefit − cost) ÷ cost over a defined period** — the classic accounting-style ratio. (Verified in substance — standard usage; flagged: ROI is the least standardized metric in the set, with no single canonical formula — it is period-dependent, sensitive to what counts as "benefit" (cash savings? revenue? avoided cost? accounting profit?), and easily gamed by choosing the period and the cost base.) Use ROI as a **communication metric** ("every S$1 invested returns S$2.10 over five years") alongside NPV/IRR/payback, not as the decision metric. A business case whose headline number is ROI alone is a case that has chosen its metric to flatter its option.

### 4.6 The Discount Rate (verified in substance; flagged where noted)

The discount rate is the price of time and risk: the rate at which future cash flows are converted to present value. Its choice is the single most consequential assumption in the model (a lower rate flatters long-dated benefits; a higher rate punishes them). Practice:

- **Corporate:** the cost of capital — usually **WACC** (the blended cost of equity and after-tax debt — cross-ref [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §6.3) — or the divisional hurdle rate. A CACIB-style investment committee will have a published hurdle; use it, and name it.
- **Public sector:** the UK Green Book's social time preference rate (3.5%, falling to 3.0% for long-dated projects — framed, standard Green Book practice; the 2025-review 2026 edition retains the social time preference approach, verified by the published 2026 Green Book).
- **Banking context:** hurdle rates commonly run 10–15% for technology investments (the cost of equity is high; cross-ref the CFA guide's cost-of-capital material, [../banking/cfa_program_guide.md](../banking/cfa_program_guide.md)).
- **The rule:** agree the rate with finance *before* building the model, use it for every option and every scenario, and sensitivity-test it (the §8 case tests 8%/10%/12%).

### 4.7 The Financials Table

| Metric | Definition | Notes |
|--------|-----------|-------|
| **TCO** | Total cost of ownership: all lifecycle costs — acquisition, operations, maintenance, support, end-user, disposal (verified — Gartner) | The honesty check for buy vs build vs partner; "the sticker price is not the price" (origin: Gartner, 1987 — flag); the §8 case shows build with lowest opex but highest TCO |
| **NPV** | Net present value: Σ CFₜ/(1+r)ᵗ − initial investment (verified) | The master decision rule: accept if NPV > 0, prefer highest NPV among exclusive options; model incremental flows vs baseline; discount rate = WACC/hurdle, applied consistently |
| **IRR** | The discount rate at which NPV = 0 (verified) | Supporting metric: accept if IRR > hurdle; misleads on non-normal cash flows and favors small quick wins over large value — the §8 trap |
| **Payback** | Time until cumulative net cash flow turns positive (verified) | Simple vs discounted — be explicit; complements NPV/IRR; ignores value after payback; long payback needs an explicit justification |
| **ROI** | (Benefit − cost) ÷ cost over a defined period (verified in substance; flagged) | Communication metric, not decision metric; period- and definition-sensitive; never the headline alone |
| **Discount rate** | The rate converting future cash flows to present value: WACC/hurdle (corporate), 3.5% social time preference (UK Green Book — framed) | The most consequential assumption; agree with finance up front; apply to all options; sensitivity-test (8/10/12%) |
| *(Companions)* | BCR (benefit–cost ratio), economic NPV, cost-effectiveness (public/social settings — verified, umbrex) | Use what your environment standardizes, and explain it in plain language |

**The financial-model design discipline (framed):** the model is a *decision engine*, not a spreadsheet: named and sourced assumptions, a clearly marked inputs sheet, incremental cash flows vs the baseline, one discount rate, a frozen version referenced by the case, and sensitivity built in from day one. If finance cannot take the model over and audit it in an afternoon, it is not decision-grade yet.

### 4.8 The Financial Model, Step by Step (framed — the eight-step recipe)

Building the model is a sequence with a discipline at each step; the §8 worked case is this recipe executed:

1. **Fix the baseline first.** Model do-nothing at run-rate for the full horizon: volumes, costs, prices, growth, and the accumulating risk costs (§5.1). Lock it. Every option is a delta against this.
2. **Fix the conventions with finance.** The discount rate (WACC/hurdle), the horizon (3/5/10 years by asset type), the currency, inflation treatment, and the tax treatment — agreed *before* the model exists, so nobody moves the goalposts later.
3. **Build the cost side per option.** Capex by category (license/build, integration, migration, change management), opex by category (support, hosting, operations, end-user), with growth rates and one-off events (renewals, upgrades). This is the TCO machinery (§4.1).
4. **Build the benefit side per option, as deltas.** Operating-cost savings, revenue uplift, risk reduction — each with a named driver, a source, and an owner. No delta without a driver; no driver without a number.
5. **Time the flows.** When does each cost hit, when does each benefit land? Timing is where honesty lives: a benefit that lands in year 3 is worth far less than one in year 1, and the delay is the most expensive assumption in the model (§8.5).
6. **Compute the metrics.** NPV (the decision), IRR (the communication), payback (the liquidity check), ROI (the ratio) — from the same cash-flow stream, never from separate models (§4.2–4.5).
7. **Stress it.** One-variable sensitivity on every material assumption, the tornado ranking, and the bear/bull scenarios (§6.1). The model is finished when the critical assumptions are known and named.
8. **Freeze and reference.** Version the model, put the version number in the case, and attach the assumptions sheet. The case argues; the model evidences; both are frozen on the day the committee decides.

The recipe's one rule (framed): **the model is built for the question "which option," not for the answer "yes" — the numbers must be able to say no.** A model that cannot produce an unfavorable NPV for the recommended option is a sales tool, not a decision engine.

---

## 5. The Options Appraisal

### 5.1 The Do-Nothing Baseline (verified — the mandatory comparator)

Every honest options appraisal starts from **do-nothing**: the baseline case against which every option's incremental costs and benefits are measured. **(Verified —** the do-nothing/do-minimum base case is standard appraisal practice: the consultant practice literature explicitly requires the "do nothing" base case as the comparator in options analysis (umbrex's options chapter: how to "define the 'do nothing' base case, generate and shortlist options, define evaluation criteria"); the HM Treasury Green Book's appraisal method likewise appraises options *against the status quo counterfactual* (framed from the published 2026 Green Book: appraisal "assesses the costs, benefits and risks of different options for achieving" objectives); and the vendor end-of-life scenario literature treats do-nothing as the first option to model (gsphere's business-case series: the do-nothing option is what happens when a vendor announces end of life — a scenario the §8 worked example deliberately mirrors).)

**The two disciplines of the baseline (framed, standard practice):**

1. **Do-nothing is not "nothing changes."** It is the run-rate: volumes grow, costs escalate, vendor support premiums rise, risk accumulates, market share erodes. A baseline that holds the world still flatters every option by comparison with a straw man. In the §8 case, the do-nothing baseline costs S$27.0M over five years — *more* than any option's TCO — because the legacy platform's cost is growing 4%/yr.
2. **All options are measured as deltas vs the baseline.** The NPV of an option is the NPV of its *incremental* cash flows against do-nothing — never its absolute cash flows. This is what makes options comparable, and it is why the baseline must be locked before options are built (a baseline that moves after options are built makes the appraisal a moving target — the Model Reef practice literature calls exactly this "locked comparison rules that keep option deltas honest," framed).

### 5.2 Generating and Shortlisting Options (framed)

Good option sets are *generated broadly, then shortlisted to three to five* for full appraisal. The generation axes for an IT investment: **buy** (commercial product/platform), **build** (in-house development), **partner/outsource** (managed service, SaaS, BPO), **upgrade/extend** (the existing asset), **retire/simplify** (decommission and fold into something else), and **do-nothing**. The shortlist criteria: strategic fit, financial case, feasibility, risk, timing, and reversibility. The discipline: **the option the committee will propose in the meeting must already be in the case** — nothing kills a business case faster than a committee member inventing an option the analysis never considered.

### 5.3 The Options Table

| Option | What it is | Cost profile | Benefit profile | Typical fate |
|--------|-----------|--------------|-----------------|--------------|
| **Do nothing** | The baseline: run the current state at run-rate | No capex; rising opex and rising risk cost | None — the comparator for all deltas | Never recommended, always required — it is the yardstick |
| **Upgrade / extend** | Patch or incrementally modernize the existing asset | Low capex, fast | Limited: fixes the immediate pain, not the structural cost | High IRR, low NPV — the "cheap but shallow" option (§8's trap) |
| **Buy** | Acquire a proven commercial product and integrate | Moderate capex + license/support opex | Full benefit set, faster to realize | The usual winner when a proven market exists |
| **Build** | Develop in-house on a modern stack | High capex, lower opex | Full benefit set, delayed; control and IP | Wins only when no market product fits or the asset is strategic |
| **Partner / outsource** | Managed service or SaaS | Opex-heavy, low capex | Benefits with low internal effort | Wins on liquidity constraints; loses on control and exit cost |

**The appraisal scoring discipline (framed):** score non-financial criteria (strategic fit, feasibility, risk, timing) on a weighted scorecard *separately* from the financials, then present both — the recommendation must survive the combination, not just one column. A case that recommends the highest-NPV option while the scorecard screams "poor strategic fit" is a case that will lose in committee; the two views must be reconciled in the analysis section, visibly.

### 5.4 The Weighted Scorecard in Practice (framed — a mini worked example)

The scorecard translates judgment into a comparable table. The mechanics: criteria, weights that sum to 100%, scores per option (1–5), weighted totals. The discipline is *declaring the weights* — the committee should be able to see what the case values, and argue with it. A condensed version for the §8-style platform case:

| Criterion (weight) | Opt 1 upgrade | Opt 2 buy | Opt 3 build |
|--------------------|:---:|:---:|:---:|
| Strategic fit (25%) | 2 | 5 | 4 |
| Feasibility / delivery risk (20%) | 4 | 4 | 2 |
| Time to benefit (15%) | 4 | 5 | 2 |
| Operational risk reduction (15%) | 2 | 4 | 4 |
| Flexibility / control (10%) | 2 | 3 | 5 |
| Resource fit (10%) | 3 | 4 | 2 |
| Vendor / exit risk (5%) | 3 | 3 | 5 |
| **Weighted total** | **2.85** | **4.25** | **3.25** |

*(Arithmetic: e.g., buy = 0.25×5 + 0.20×4 + 0.15×5 + 0.15×4 + 0.10×3 + 0.10×4 + 0.05×3 = 4.25. Scores are judgment calls — the point is the visible structure, not the precise values.)*

The scorecard confirms the financials (§8.4: buy leads on NPV): the two views agree, which is exactly the reconciliation the analysis section must demonstrate. When the scorecard and the financials *disagree*, that disagreement is the most important finding in the case — it means the highest-NPV option is not the best option, and the case must resolve it explicitly rather than hide it.

---

## 6. The Risk Analysis

### 6.1 Sensitivity Analysis (verified — the model's honesty test)

**Sensitivity analysis tests how the case's outcome (NPV, IRR, payback) responds to changes in its key assumptions** — it answers "how wrong can we be and still be right?" **(Verified —** sensitivity analysis is the standard risk-testing tool of financial modeling: one-variable-at-a-time (OVAT) tables, two-variable data tables, tornado charts, and scenario analysis (base/bull/bear) are the canonical techniques across the financial-modeling and risk-publishing literature; tornado charts rank the assumptions by their impact on the outcome (riskpublishing.com, fastercapital, the DCF-modeling guides — all describe the same toolkit); the consultant business-case practice requires "sensitivity analysis" as one of the six elements of an approvable case (deckary, verified).)

**The toolkit:**

- **One-variable sensitivity:** change one assumption (benefits −20%, opex +20%), recompute NPV, record. Simple and board-friendly.
- **Tornado chart:** vary each assumption across its range, record the resulting NPV swing, and rank — the chart shows which assumptions the case *lives or dies on* (the widest bars are the assumptions to nail down before signing).
- **Scenario analysis:** combine the moves into coherent worlds — base case, bear case (benefits down, costs up, one-year delay), bull case — and report the NPV in each. The §8 case's bear case stays positive, which is the sentence that makes a committee comfortable.
- **Monte Carlo (framed):** probability distributions over assumptions, thousands of runs, a distribution of NPV. Powerful, harder to explain to a committee, and usually unnecessary until the case is large enough to warrant it.

**What the sensitivity section must conclude (framed):** identify the **critical assumptions** (the ones that move the decision), state the **break-even points** (how much benefits can fall, or costs rise, before NPV = 0 — the §8 case's break-even is its IRR, 39%), and commit to managing the critical ones (contract clauses, phasing, validation pilots).

### 6.2 Risk and Uncertainty (framed — standard risk vocabulary)

Risk is the effect of uncertainty on objectives — typically captured as **likelihood × impact**, recorded in a risk register, and managed through mitigation (cross-ref [risk_management_models_guide.md](../banking/risk_management_models_guide.md) for the banking-grade vocabulary: probability/impact scoring, stress testing, early-warning indicators). For a business case, the risk section has a specific job beyond the project's operational risk register: **it must test the case itself.** The three risk families:

- **Delivery risks** — will the project deliver on time and budget? (Vendor failure, scope creep, key-person loss, integration complexity.)
- **Benefits risks** — will the promised benefits materialize? (Adoption failure, benefit double-counting, baseline drift, benefit leakage to the baseline.)
- **External risks** — will the world cooperate? (Regulatory change, market shifts, interest-rate moves, technology disruption.)

The Green Book adds a public-sector discipline worth borrowing (framed): **optimism bias** — the documented tendency of appraisers to understate costs and overstate benefits — is adjusted for explicitly (typically +3–84% on costs depending on project type in the Green Book's tables). Private-sector cases should at least test the equivalent: assume your own forecast is optimistic and stress it (§6.1 does exactly this).

### 6.3 The Risk Table

| Risk | Likelihood | Impact | Mitigation | Residual |
|------|-----------|--------|-----------|----------|
| **Benefit shortfall** (adoption, volume, unit economics) | Medium | High | Named benefit owners, staged benefits gates, pilot before full rollout, sensitivity shows case survives −20% | Medium |
| **Cost overrun / delay** | Medium | Medium–High | Fixed-price contract elements, phased funding, contingency (10–15%), weekly vendor governance | Medium |
| **Vendor failure / lock-in** | Low–Medium | High | Exit clauses, escrow, reference-site diligence, data portability rights, second-source fallback (cross-ref [vendor_management_guide.md](vendor_management_guide.md)) | Low |
| **Regulatory change** | Medium | Medium | Regulatory engagement plan, flexible architecture, compliance as an explicit requirement, MAS/regulator liaison (cross-ref the banking risk guide) | Low–Medium |
| **Operational disruption** (migration, cutover) | Medium | Medium | Parallel run, cutover rehearsal, rollback plan, weekend-window execution | Low |
| **Baseline drift** (do-nothing world changes, benefits double-counted) | Medium | High | Locked baseline, audit trail of assumptions, finance-owned model, post-implementation review | Low |

**The risk section's one-sentence deliverable (framed):** *"Under the bear case — benefits 20% lower, costs 20% higher, delivery a year late — the recommended option still returns positive NPV of S$3.1M."* That sentence, evidenced, is what converts a proposal into an investable proposition.

### 6.4 Optimism Bias and the Green Book (framed)

The public-sector appraisal literature contributes one discipline the private sector should borrow wholesale: **optimism bias** — the empirically documented tendency of appraisers to understate costs and overstate benefits and delivery times. The UK Green Book requires appraisers to adjust cost estimates by published optimism-bias ranges (for standard project types, the adjustment bands run from single digits to tens of percent on costs, with the exact figures varying by project category and edition — framed: the mechanism is standard Green Book practice, the specific band tables belong to the current edition and are worth reading in the original). The private-sector translation is cheaper and nearly as effective: **assume the case you wrote is the optimistic one, and stress it.** The §8 worked case does exactly this — the bear case is not a gesture, it is the mechanism by which optimism bias is neutralized. (The banking-risk vocabulary for the same idea: stress testing and add-ons — cross-ref [risk_management_models_guide.md](../banking/risk_management_models_guide.md).)

### 6.5 When to Reach for Monte Carlo (framed)

One-variable sensitivity and scenario analysis carry most business cases; Monte Carlo (probability distributions over assumptions, thousands of simulated outcomes, a distribution of NPV) is the next rung up. Use it when: (i) the case is large enough that the committee demands it; (ii) the critical assumptions are genuinely uncertain *and* correlated in ways scenarios cannot capture; or (iii) the organization's finance practice is already simulation-based. Its cost is communicability — "the 80% confidence NPV is S$4.2M" is powerful but requires an audience that reads distributions. For most cases, the tornado chart plus three scenarios delivers 90% of the insight with 10% of the complexity (§6.1). The honest rule: **sophistication should follow the size of the decision, not the enthusiasm of the analyst.**

---

## 7. The Stakeholders

### 7.1 The Stakeholder Map (verified in substance; attribution flagged)

**Stakeholder analysis identifies everyone who can affect or be affected by the decision and the project, and classifies them by interest and influence so the engagement can be planned.** The canonical tool is the **power/interest grid**: stakeholders plotted by their power (influence over the outcome) against their interest (how much they care), yielding four engagement strategies (verified in substance — the power/interest grid is standard PMBOK-based stakeholder-management practice: project-initiation guides enumerate the grid and its engagement strategies among the PMP exam's core stakeholder tools; the grid's origin is commonly attributed to Johnson & Scholes's strategy text and to Eden & Ackermann's *Making Strategy* — **flag**: the attribution is the standard one in the literature but was not re-verified against the original texts this edition).

The four quadrants:

- **High power / high interest — *Manage closely:*** the sponsor, the CFO, the COO. These decide the case's fate; they get direct, frequent, face-to-face engagement.
- **High power / low interest — *Keep satisfied:*** the CIO's committee, the risk director, audit, the regulator's shadow. Engage enough to keep them supportive and prevent surprise vetoes.
- **Low power / high interest — *Keep informed:*** the operations teams whose jobs the change touches, the front office, the IT staff. The case dies in adoption if these are surprised — inform early and often.
- **Low power / low interest — *Monitor:*** the wider organization. Minimal, scheduled communication.

### 7.2 The Discipline (framed — standard practice)

- **Identify before the case is written, not after.** The stakeholder map shapes the case: the CFO's questions shape the financials, the risk director's shape the risk section, the COO's shape the implementation plan. (This is why §7 is not an afterthought section — it is a *design input* to the case, cross-ref [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md).)
- **Include the opponents and the indifferent.** The most dangerous stakeholder in an investment case is the one who is *not* in the room: the ops manager whose team will shrink, the vendor whose contract is being displaced, the peer program competing for the same budget. Name them, understand their interest, plan the engagement.
- **RACI it:** for each key stakeholder, define their role in the decision — Responsible, Accountable, Consulted, Informed — and write the communication plan to match.

### 7.3 The Stakeholders Table

| Stakeholder | Interest in the case | Power | Engagement strategy |
|-------------|----------------------|-------|---------------------|
| **Sponsor** (e.g., Head of Trade Finance / COO) | The case's success is their program; owns the argument | High | Manage closely: co-author the case, own the executive summary, front the committee |
| **CFO / finance** | Affordability, NPV, the model's integrity, the P&L impact | High | Manage closely: agree the discount rate and model conventions up front; finance owns the model |
| **CIO / CTO / architecture** | Feasibility, fit with the target architecture, run-the-bank stability | High | Keep satisfied: the architecture review is a gate; involve early, avoid surprise vetoes |
| **Risk & compliance** | Regulatory exposure, operational risk, MAS obligations | High (in banking) | Keep satisfied: put compliance in as an explicit requirement, evidence it |
| **Front office / business lines** | Service quality, turnaround, revenue impact | Medium–High | Keep informed (and manage closely if revenue-bearing): quantify the revenue uplift with them |
| **Operations teams** | Job impact, workload change, process redesign | Medium (high interest) | Keep informed: change management, training plan, early involvement in design |
| **Procurement / vendor management** | Contract, license economics, negotiation leverage | Medium | Keep informed: run the commercial process (cross-ref [vendor_management_guide.md](vendor_management_guide.md)) |
| **Internal audit** | Governance, benefits realization, baseline integrity | Medium | Keep satisfied: the case's audit trail is its defense |
| **Peer programs** | Competing budget, shared resources | Low–Medium | Monitor + coordinate: portfolio-level deconfliction |

**The stakeholder one-liner (framed):** *a business case is approved by the people whose questions it has already answered.* Read the map, write the case to the map, and present it in the map's order.

### 7.4 The Engagement Plan Mechanics (framed — standard practice)

The stakeholder table (§7.3) is the analysis; the engagement plan is the action. The mechanics, in sequence:

1. **Segment by quadrant.** From the power/interest grid, assign each stakeholder one of the four strategies (manage closely / keep satisfied / keep informed / monitor — §7.1). The quadrant assignment drives everything downstream.
2. **Define the engagement per stakeholder:** frequency (weekly / monthly / by milestone), channel (face-to-face, committee pack, email update), and the ask (approve, advise, be consulted, be informed, do nothing).
3. **Map the questions to the case.** For each high-power stakeholder, list the questions they will ask and confirm the section of the case that answers them. If a question has no home section, the case has a gap — this is the mechanism by which the stakeholder map *designs* the case (§7.2).
4. **Plan for the opponents.** For stakeholders whose interest is threatened (the displaced vendor, the team whose workload changes), plan the engagement explicitly: early notice, involvement in design, transition support. Unplanned opponents become last-minute committee ambushes.
5. **Revisit at each gate.** The map changes as the case matures (SOC → OBC → FBC): the CFO's interest rises as the numbers firm; the regulator's interest rises as the solution takes shape. The map is a living artifact, like the case itself (§1.3).

---

## 8. The Worked Example

### 8.1 The Scenario — A CACIB-Style Platform Investment (the familiar context)

*Context: a corporate-and-investment-bank-style institution (the Crédit Agricole CIB profile) running trade finance out of its Singapore hub. Trade finance is a core CIB product line: letters of credit, guarantees, documentary collections, supply-chain finance — processed on a legacy workflow platform that is approaching vendor end-of-life (cross-ref [trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md) for the systems landscape and [universal_banking_model_guide.md](../banking/universal_banking_model_guide.md) for the business lines).*

**The problem, quantified (the case for change):**

- The bank processes ~120,000 trade transactions/year through the Singapore hub; the legacy platform handles them with a **manual touch rate of ~65%** — document checking, data re-keying, and exception handling are human-intensive.
- **Current operating cost: S$4.8M/year** for the trade-finance operations stack (platform support, manual processing labor, rework, error correction), **growing ~4%/year** (vendor end-of-life support premiums + volume growth).
- **STP (straight-through processing) rate: 35%** vs a market benchmark of 60%+: slower turnaround loses deals (estimated S$1.4M/year in forgone revenue) and erodes the bank's trade-finance franchise.
- **Regulatory exposure:** manual handling risks documentary-compliance errors; MAS supervision is tightening on operational resilience and outsourcing (cross-ref the banking risk vocabulary in [risk_management_models_guide.md](../banking/risk_management_models_guide.md)).
- **The vendor has announced end-of-life:** no further development, no roadmap, escalating support premiums — the classic do-nothing trigger (mirroring the vendor-EOL scenario from the business-case literature, §5.1).

**The strategic context:** the CIB's strategy calls for digitizing trade finance (digital LCs, API integration with clients, ISO 20022 readiness). The investment serves a named strategic objective: *"operational excellence and digital trade"* — this is the alignment sentence the strategic case requires (cross-ref [strategic_management_guide.md](strategic_management_guide.md)).

**The stakeholders in the room (from §7's map):** the Head of Trade Finance (sponsor), the CFO, the CIO and the Solution Architect (the reader of this guide — the architect owns the feasibility and architecture-fit leg), Risk & Compliance (MAS exposure), the trade-ops teams (the 40 FTE whose work the platform will reshape), and Procurement.

### 8.2 The Options (the field, including do-nothing)

| Option | Description | One-time cost (S$M) | Steady-state annual opex (S$M) |
|--------|-------------|--------------------:|-------------------------------:|
| **0 — Do nothing** | Run the legacy platform at run-rate: escalating support, manual processing, rising risk. **No capex — but the baseline is not static:** costs grow 4%/yr | 0 | 4.8 → 5.8 by yr 5 (growing) |
| **1 — Upgrade the legacy platform** | Vendor's end-of-life extension: patch, add capacity, buy time. Low capex, fixes nothing structural | 2.5 | 4.2 (growing 4%/yr) |
| **2 — Buy a commercial STP platform** | Acquire a proven trade-finance STP product; integrate with core banking, Swift, and client channels; change management for ops | 8.0 | 2.6 (growing 3%/yr) |
| **3 — Build in-house** | Custom development on a modern stack: full control, full cost, longest timeline; benefits land ~1 year later | 14.0 | 1.8 (growing 3%/yr) |

**The appraisal logic (framed, from §5):** every option is a delta against Option 0. The benefits (all measured vs the baseline) are:

- **Operating-cost savings** = baseline cost − option opex (the TCO delta).
- **Revenue uplift** — STP rate 35% → 60%+: faster turnaround and digital channels recover ~S$0.8M/year of forgone business, growing with volumes.
- **Risk reduction** — avoided regulatory fines, outage costs, and reputational damage: ~S$0.5M/year (conservative, flat).

**Non-financial scoring (weighted scorecard, from §5.3):** strategic fit (buy wins — proven market product, fastest to digital trade; build wins on control but loses on timing), feasibility (buy highest; build lowest — 24-month delivery risk), operational risk (build highest — key-person and delivery risk), flexibility (build wins on IP; buy wins on upgrade path). **On the combination, Option 2 leads; the financials below must confirm it.**

### 8.3 The TCO Worked Case (the design — 5-year total cost of ownership)

| Option | Capex (S$M) | 5-yr opex (S$M) | **5-yr TCO (S$M)** | Read |
|--------|------------:|----------------:|-------------------:|------|
| **0 — Do nothing** | 0 | 27.0 | **27.0** | The most expensive option — the run-rate is a cost, not a saving |
| **1 — Upgrade** | 2.5 | 23.7 | **26.2** | Barely cheaper than do-nothing; buys time, not change |
| **2 — Buy** | 8.0 | 14.2 | **22.2** | Cheapest TCO — the license is visible, the savings are structural |
| **3 — Build** | 14.0 | 9.8 | **23.8** | Lowest *opex* of all — but the capex dominates: the sticker price is not the price (§4.1) |

*Arithmetic (illustrative, rounded): 5-yr opex for do-nothing = Σ 4.8×1.04ᵗ ≈ S$27.0M; buy = Σ 2.6×1.03ᵗ ≈ S$14.2M; build = Σ 1.8×1.03ᵗ ≈ S$9.8M; upgrade = Σ 4.2×1.04ᵗ ≈ S$23.7M.*

**The TCO lesson baked into the design:** the build option has the lowest operating cost but the *second-highest* TCO — the honest comparison is the lifecycle total, not the opex line. And the do-nothing option, which costs nothing to approve, is the most expensive to live with — the point §5.1 makes.

### 8.4 The NPV–IRR–Payback Worked Case (the design, computed)

**Method (per §4):** incremental cash flows vs the do-nothing baseline; discount rate **10%** (the bank's technology hurdle — sensitivity-tested at 8%/12%); five-year horizon; all figures S$M, rounded.

**Option 2 (Buy) — the delta cash flows vs do-nothing:**

| Year | 0 | 1 | 2 | 3 | 4 | 5 |
|------|--:|--:|--:|--:|--:|--:|
| Capex | −8.0 | — | — | — | — | — |
| Opex saving vs baseline | — | +2.3 | +2.4 | +2.6 | +2.7 | +2.8 |
| Revenue uplift | — | +0.8 | +0.9 | +0.9 | +0.9 | +1.0 |
| Risk reduction | — | +0.5 | +0.5 | +0.5 | +0.5 | +0.5 |
| **Net delta cash flow** | **−8.0** | **+3.6** | **+3.8** | **+4.0** | **+4.1** | **+4.3** |
| Discount factor @10% | 1.000 | 0.909 | 0.826 | 0.751 | 0.683 | 0.621 |
| **Present value** | −8.0 | +3.3 | +3.1 | +3.0 | +2.8 | +2.7 |

*(Arithmetic detail: year-t delta = (baseline cost 4.8×1.04ᵗ − buy opex 2.6×1.03ᵗ) + revenue uplift 0.8×1.04ᵗ + 0.5. Rounded rows: yr1 3.6, yr2 3.8, yr3 4.0, yr4 4.1, yr5 4.3; the discounted column sums to the NPV below.)*

**The headline metrics (computed):**

| Metric | Value | Decision check |
|--------|------:|----------------|
| **NPV @ 10% (5-yr)** | **+S$6.9M** | > 0 → creates value; the highest of the three options |
| **IRR** | **39%** | >> 10% hurdle → comfortably acceptable |
| **Payback (simple)** | **3.1 years** | Within the bank's 4-year threshold |
| **Payback (discounted)** | **3.5 years** | Still within threshold — honest version |
| **NPV profile** | Positive from year 2.5 (discounted) | The case goes positive quickly |

**The comparison across options (the table that decides):**

| Metric | Opt 0 do nothing | Opt 1 upgrade | **Opt 2 buy** | Opt 3 build |
|--------|-----------------:|--------------:|--------------:|------------:|
| 5-yr TCO | 27.0 | 26.2 | **22.2** | 23.8 |
| NPV @ 10% | 0 (baseline) | +3.1 | **+6.9** | +2.2 |
| IRR | — | 50.8% | **39.0%** | 15.3% |
| Payback (simple) | never | 2.7 yr | **3.1 yr** | 4.5 yr |
| Benefits land | — | yr 1 | **yr 1** | yr 2 (delayed) |
| Strategic fit | fails | partial | **full** | full (but late) |

**The decision — and the trap it illustrates (framed, from §4.3):** Option 1 (upgrade) has the *highest IRR* (50.8%) and fastest payback — by IRR logic it would win. But it creates the *least* value (+3.1 NPV) and fails the strategic case: it does not digitize trade, it postpones. **IRR favors small, fast, shallow wins; NPV measures value creation.** The recommendation is **Option 2 — buy** — highest NPV, cheapest TCO, full strategic fit, and benefits that land in year 1. Option 3 (build) is the honest runner-up on control but its NPV barely clears the hurdle (15.3% IRR vs a 10% hurdle) and its one-year delay costs ~S$3.8M of NPV — the delay lesson (§6.1) visible in the comparison itself.

### 8.5 The Risk and Sensitivity Worked Case (the design)

**Sensitivity (one-variable-at-a-time on Option 2, NPV @ 10%, S$M):**

| Assumption moved | Value tested | NPV | Swing vs base |
|------------------|-------------:|----:|--------------:|
| *(Base case)* | — | **+6.9** | — |
| Discount rate | 8% / 12% | +7.7 / +6.2 | low sensitivity — benefits are near-term |
| Benefits | −20% / +20% | +3.9 / +9.9 | **the critical assumption** — the widest tornado bar |
| Opex | +20% | +4.8 | moderate |
| Capex | +25% | +4.9 | moderate |
| Delivery delay | benefits shift +1 yr | +3.1 | high — delay destroys value |

**The bear case (scenario, from §6.1):** benefits 20% lower, costs 20% higher, delivery one year late → **NPV +S$3.1M, still positive.** The case survives its own pessimism — the one-sentence deliverable of the risk section (§6.3), evidenced.

**The risk register (top rows, from §6.3's table applied to the case):** adoption failure (ops teams resist the new workflow — mitigation: change management budgeted S$1.0M inside the capex, ops staff embedded in the design team); vendor delivery slippage (mitigation: fixed-price integration contract, phased go-live by product line); data migration errors (mitigation: parallel run, reconciliation, rollback plan); regulatory change in trade compliance (mitigation: compliance requirements as explicit acceptance criteria, MAS liaison); and the do-nothing risk the case itself retires — the legacy platform's end-of-life (the case *is* the mitigation).

### 8.6 The Stakeholders in the Case (the design, from §7)

| Stakeholder | Interest | Power | Engagement in the case |
|-------------|----------|-------|------------------------|
| Head of Trade Finance (sponsor) | The franchise, the STP ambition | High | Co-author; owns the executive summary and the committee pitch |
| CFO / finance | Affordability, the model, the S$8.0M ask | High | Agreed the 10% hurdle and the baseline rules up front; finance owns the model |
| CIO / Solution Architect | Feasibility, architecture fit, run-the-bank stability | High | The architect owns the target-architecture and integration section — *this reader's seat* |
| Risk & Compliance | MAS exposure, documentary compliance | High | Compliance built in as acceptance criteria; regulator briefed on the digitization agenda |
| Trade ops (40 FTE) | Jobs, workflow, training | Medium (high interest) | Keep informed + embedded in design: the adoption risk lives here |
| Procurement | License terms, contract, exit clauses | Medium | Runs the commercial process; exit clauses and escrow in the contract |

### 8.7 The Lessons (the worked example's takeaways)

1. **Do-nothing is the most expensive option — on TCO.** The baseline is a cost, not a free option; it must be modeled at run-rate, not frozen (§5.1).
2. **IRR lies; NPV decides.** The highest-IRR option (upgrade, 50.8%) was the worst value decision (+3.1 NPV). A board that picks on IRR picks the shallow win every time (§4.3).
3. **TCO reveals the build trap.** Lowest opex (build, 1.8/yr) ≠ lowest total cost (build TCO 23.8 vs buy 22.2). The lifecycle total is the honest number (§4.1).
4. **Delay is a cost, quantified.** A one-year slip cost ~S$3.8M of NPV — more than the capex variance. This is why the implementation plan is part of the case, not an appendix (§3.5).
5. **The bear case sentence wins committees.** "Still positive at +S$3.1M under the pessimistic scenario" is the sentence that converts a proposal into an investment (§6.3).
6. **Benefits need owners.** The revenue uplift and the risk reduction are promises until named owners, measures, and review gates exist (§3.5). The benefits-realization plan is signed at the same meeting as the budget.
7. **The case is iterative, not a document-drop.** This case would be told three times — SOC (is there a case for change? yes), OBC (which option? buy), FBC (fund the contract) — each cheaper to abandon than the last (§1.3).

### 8.8 The Case at Three Gates (the SOC → OBC → FBC ladder for this example)

The same investment, told three times with increasing precision (§1.3) — each version is what the committee sees at its gate:

**Gate 1 — Strategic Outline Case (SOC): "is there a case for change?"**
- The problem, unadorned: platform at end-of-life, 65% manual touch, costs growing 4%/yr, STP at half the market rate, MAS pressure.
- Options named, not yet appraised: do nothing / upgrade / buy / build.
- Cost band: rough — S$6–14M capex depending on route (±40–50%).
- **Decision sought:** permission to explore options and commission the OBC. Abandonment here costs almost nothing; that is the point of the gate.

**Gate 2 — Outline Business Case (OBC): "which option?"**
- The options appraised on the common basis (§8.2–8.4): TCO, NPV, IRR, payback, scorecard.
- **Buy emerges as the preferred option** — highest NPV (+6.9), cheapest TCO (22.2), full strategic fit, benefits in year 1.
- Cost band tightens: S$8.0M ± 20–30%; the 10% hurdle agreed with finance.
- **Decision sought:** confirm buy as the way forward; authorize the commercial process (vendor shortlist, RFP). Abandonment here costs the OBC effort only.

**Gate 3 — Full Business Case (FBC): "fund the contract."**
- The commercial case: the selected vendor, the contract structure, license and support terms, exit clauses and escrow (cross-ref [vendor_management_guide.md](vendor_management_guide.md)).
- The financial case: final S$8.0M capex and the 5-yr opex profile; affordability confirmed in the plan; the model frozen and versioned.
- The management case: phased go-live by product line, named benefits owners, the review gates, the change plan for the 40-FTE ops team.
- **Decision sought:** release the funds and sign. Abandonment here is expensive — which is exactly why the previous gates existed.

**The ladder's lesson (framed):** the case is a *conversation with the committee over time*, not a document delivered once. Each gate spends a little to avoid spending a lot; the discipline of the ladder is that the decision at each gate is made on the evidence the case has assembled so far — never on the hope that the next version will be more convincing.

---

## 9. The Summary — The Case for the Case

### 9.1 The One-Page Discipline

Every framework, every table, and every metric in this guide compresses into **one page that the decision-maker reads, and one minute that the sponsor speaks**:

> **The ask:** fund the acquisition of a commercial trade-finance STP platform at S$8.0M, generating S$6.9M NPV over five years at a 10% hurdle rate, 39% IRR, 3.1-year payback — and still positive at S$3.1M under the bear case.
> **Why now:** the legacy platform is at end-of-life, its cost is growing 4%/yr, our STP rate of 35% is half the market's, and MAS is watching.
> **Why this option:** do-nothing costs the most (S$27.0M TCO); upgrade is the highest-IRR trap (+3.1 NPV); build is 24 months and barely clears the hurdle. Buy is the cheapest lifecycle cost, the fastest benefits, the full strategic fit.
> **What we need from you:** approval of the funding, the named benefits owners, and the phased go-live gates.

If the body of the case does not *prove* each sentence of that page, the page is fiction. The one-page is the compression test: **the case is ready when the summary writes itself from the analysis.**

### 9.2 The Presentation (framed — running the committee session)

The document gets read; the session gets decided. The presentation discipline for the investment-committee meeting:

- **Open with the decision, close with the ask.** "We recommend funding Option 2…" is the first sentence; the specific approval requested (funds, benefits owners, gates) is the last. Everything between is evidence for that arc.
- **Three charts carry the room:** the one-page summary (§9.1), the options comparison table (§8.4), and the sensitivity/bear-case chart (§8.5). If the argument needs more than three charts, it is not yet an argument — the rest lives in the appendix, to be deployed on question.
- **Pre-answer the known attacks.** The CFO's questions (discount rate, baseline integrity, double-counting), the risk director's (the bear case), the sponsor's own critics (why not build) are all pre-empted visibly in the deck (§3.7). A question that gets answered *before* it is asked is the strongest signal of a decision-grade case.
- **Name the conditions and the owners out loud.** "This case is conditional on the vendor contract containing X, and the benefits are owned by Y, measured by Z, reviewed at gate Q" — the committee approves conditions, not hopes (§3.5).
- **The one-minute version.** The sponsor must be able to make the entire case in one minute in the corridor: the ask, why now, why this option, what's needed. If the sponsor cannot, the case is not yet clear (framed — the consulting discipline of the elevator case, cross-ref [mckinsey_approach_guide.md](mckinsey_approach_guide.md)).

### 9.3 The Final Word — 'The Case for the Case'

The business case is the discipline that makes capital allocation honest. It exists because organizations are bad at three things on their own: **comparing** (options are rarely put side by side), **discounting** (future value is intuited, not computed), and **committing** (benefits are promised but never owned). The business case is the counterweight: it forces the comparison, the discounting, and the commitment onto paper, before the money moves.

For a technologist — a Solution Architect, a platform lead, a data-architect candidate — the business case is the bridge between the two languages of the enterprise: **the architecture (what we build) and the economics (why we build it).** The architect who can write the strategic case has the sponsor's ear; the architect who can build the financial model has the CFO's respect; the architect who can run the sensitivity analysis has the risk director's trust. This guide's motivation is the standing skill-gap priority confirmed in [data_architect_skillgaps_guide.md](../technology/data_architect_skillgaps_guide.md) (#2 of the top-10): business case & TCO modeling is a real gap for the data-architecture trajectory, and this guide — with its vocabulary cross-referenced into the MBA quant-finance chapter ([mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §6) and the FinOps ongoing-cost discipline ([finops_guide.md](../technology/finops_guide.md)) — is the closing of it.

**The final word:** *the case for the case is that money should follow evidence, and evidence should be argued — not asserted. Every funded initiative in your organization passed through a business case; the people who wrote them well shaped what the organization does next. That is the skill. This guide is the discipline.*

---

## The Glossary

| Term | Definition |
|------|-----------|
| **Business case** | A structured decision-support document justifying a proposed investment or change: problem, options, costs, benefits, risks, and recommendation (§1.1) |
| **Decision support** | The function of the business case: converting an intuition into a comparable, challengeable proposition on which an evidence-based funding decision can be made (§1.1) |
| **Five Case Model** | HM Treasury's business-case framework: the strategic, economic, commercial, financial, and management cases, forming a funnel of five gates (§2.1) |
| **HM Treasury** | The UK government's finance ministry; author of the Green Book and the Five Case Model business-case guidance (verified — gov.uk) |
| **Executive summary** | The one-page opening that states the decision, the ask, and the headline evidence; the only page the committee is guaranteed to read (§3.2) |
| **Options appraisal** | The comparison of alternative ways to meet the objective — including do-nothing — on a common basis, to identify the preferred option (§5) |
| **TCO / total cost of ownership** | All lifecycle costs of an asset or capability: acquisition, operations, maintenance, support, end-user, disposal (verified — Gartner; origin 1987, flag) |
| **NPV / net present value** | The sum of future cash flows discounted to today, minus the initial investment; the master decision rule — accept if NPV > 0 (verified, §4.2) |
| **IRR / internal rate of return** | The discount rate at which NPV = 0; accept if above the hurdle rate; a supporting metric, misleading on non-normal cash flows (verified, §4.3) |
| **Payback** | The time until cumulative net cash flow turns positive; simple (nominal) or discounted; complements NPV/IRR (verified, §4.4) |
| **ROI / return on investment** | (Benefit − cost) ÷ cost over a defined period; a communication metric, period- and definition-sensitive (verified in substance; flagged, §4.5) |
| **Cost–benefit analysis** | The systematic comparison of the costs and benefits of options, in money terms where possible — the economic-case core (framed; the Green Book's method) |
| **Sensitivity analysis** | Testing how the case's outcome responds to changes in its assumptions — OVAT tables, tornado charts, scenarios; the model's honesty test (verified, §6.1) |
| **Stakeholder** | Anyone who can affect or be affected by the decision and its delivery; classified by power and interest for engagement planning (§7) |
| **Do-nothing** | The baseline option: the run-rate cost of inaction, against which all options are measured as deltas; never "nothing changes" (verified, §5.1) |
| **Baseline** | The locked counterfactual (do-nothing world) used to measure every option's incremental costs and benefits; must be fixed before options are built (§5.1) |
| **Recommendation** | The committed preferred option, its one-paragraph rationale, its conditions, and its confidence level (§3.5) |
| **Implementation plan** | The delivery skeleton: phases, timeline, governance, change management, and benefits realization (§3.5) |
| **Risk** | The effect of uncertainty on objectives, captured as likelihood × impact and managed through mitigation (§6.2) |
| **Financial model** | The decision engine behind the case: named assumptions, incremental cash flows vs baseline, one discount rate, sensitivity built in, finance-ownable (§4.7) |
| **Discount rate** | The rate converting future cash flows to present value: WACC/hurdle (corporate) or 3.5% social time preference (UK Green Book — framed); the most consequential assumption (§4.6) |

---

*End of guide. Companion material: the financial-modeling vocabulary in [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §6; the ongoing-cost discipline in [finops_guide.md](../technology/finops_guide.md); the vendor side of TCO in [vendor_management_guide.md](vendor_management_guide.md); the strategy alignment in [strategic_management_guide.md](strategic_management_guide.md); the risk quantification in [risk_management_models_guide.md](../banking/risk_management_models_guide.md); the gap this guide closes in [data_architect_skillgaps_guide.md](../technology/data_architect_skillgaps_guide.md).*
