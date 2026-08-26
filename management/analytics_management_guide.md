# Analytics Management: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Management / Analytics — The Discipline of Organizing, Running, and Governing the Analytics Function: Strategy, Organization, Lifecycle, Governance, Team, Products, Culture, Banking  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [Foundations: The Analytics Management Discipline](#1-foundations-the-analytics-management-discipline)
2. [The Analytics Organization: Operating Models and Design](#2-the-analytics-organization-operating-models-and-design)
3. [The Analytics Lifecycle: From Problem to Action](#3-the-analytics-lifecycle-from-problem-to-action)
4. [Analytics Governance: Demand, Value, Risk, and Culture](#4-analytics-governance-demand-value-risk-and-culture)
5. [The Analytics Team: Roles, Topologies, and Talent](#5-the-analytics-team-roles-topologies-and-talent)
6. [Analytics Products and Tools](#6-analytics-products-and-tools)
7. [Analytics in Banking](#7-analytics-in-banking)
8. [Worked Example: Building an Analytics Function in a Mid-Size Bank](#8-worked-example-building-an-analytics-function-in-a-mid-size-bank)
9. [The Future: Analytics in 2026 and Beyond](#9-the-future-analytics-in-2026-and-beyond)
10. [Glossary](#10-glossary)
11. [References](#11-references)

---

## 1. Foundations: The Analytics Management Discipline

Analytics management is the discipline of **organizing, running, and governing the analytics function of an enterprise** — the people who do analytics, the process by which analytics work is requested, prioritized, delivered, and adopted, the technology that supports it, and the value it produces. It sits one level above the analytics craft itself: where a data analyst asks "what does the data say?", the analytics manager asks "is the right analyst working on the right question, with the right data, and will the answer actually change a decision?"

This guide is the management counterpart to the technical data guides in this repository. The data layer ([data_governance_guide.md](../technology/data_governance_guide.md), the data engineering guides) describes *how to manage data as an asset*; this guide describes *how to manage the analytics function that turns data into decisions and value*.

### 1.1 What Analytics Management Is

Analytics management covers four interlocking domains — the same four that any management discipline covers, applied to analytics:

| Domain | Question | Content |
|--------|----------|---------|
| **People** | Who does the work? | The analytics team: roles (data scientists, analysts, analytics engineers, translators), skills, hiring, retention, career paths, upskilling |
| **Process** | How does work flow? | The analytics lifecycle: intake, prioritization, delivery, adoption, measurement; governance and demand management |
| **Technology** | What do they work with? | The analytics stack: data platforms, BI tools, notebooks, ML platforms, and the products they produce |
| **Value** | Why does it exist? | Value cases, ROI, outcomes, and the link from analytics output to business results |

A useful shorthand: analytics management is the answer to "how do we make sure our analytics investment pays off, at scale, repeatedly?" Left unmanaged, analytics degenerates into either a **reporting factory** (lots of output, little decision impact) or a **science fair** (interesting models nobody uses). Management is what converts analytics from an activity into a function with a strategy, an operating model, and measurable value.

### 1.2 The Scope of "Analytics": The Analytics Ladder

The term "analytics" covers a spectrum of analytical capability, conventionally arranged as a ladder of increasing sophistication. The four-rung framing — **descriptive → diagnostic → predictive → prescriptive** — is the standard industry model (widely associated with Gartner's analytics maturity framing and echoed across the industry; attribution is diffuse rather than a single canonical source):

| Rung | Question answered | Typical output | Human input |
|------|-------------------|----------------|-------------|
| **Descriptive** | What happened? | Reports, dashboards, KPIs | High — human interprets |
| **Diagnostic** | Why did it happen? | Drill-downs, segmentation, root-cause analysis, correlation | High — human investigates |
| **Predictive** | What will happen? | Forecasts, propensity scores, churn/risk models | Medium — model proposes |
| **Prescriptive** | What should we do? | Recommendations, next-best-action, optimization, automation | Low — system recommends or acts |

Three things worth noting about this ladder. First, **it is also a maturity progression** — organizations typically climb it over years, and most enterprises still spend the majority of their analytics effort on the bottom two rungs. Second, **each rung builds on the one below** — you cannot predict reliably without solid descriptive foundations, and prescriptive output is only as trustworthy as the predictive models underneath it. Third, **the rungs are not "better" in the abstract** — prescriptive analytics on a broken data foundation is worse than descriptive analytics done well. The management question is not "how high can we climb?" but "which rungs create value for which decisions?"

**A note on framing verification:** the four-rung ladder is sometimes called the "analytics maturity model" or "analytics progression"; the descriptive/diagnostic/predictive/prescriptive taxonomy itself is broadly attributed to Gartner's maturity research and popularized by numerous practitioners. This guide treats it as the *standard industry framing* rather than a precisely attributed academic framework. A related informal framing is "Analytics 1.0 / 2.0 / 3.0" (BI-era reporting → big-data/data science → data-enriched products and services), popularized around Davenport's work; it is a loose historical shorthand, not a formal standard.

### 1.3 Analytics vs BI vs Data Science

Three terms are constantly conflated. They are three layers of the same value chain, distinguished by their primary output:

- **Business Intelligence (BI)** — the *reporting* layer: standardized dashboards, scorecards, and reports that describe the state of the business. Its output is *information* ("what happened, right now, across the org"). Tooling: Tableau, Power BI, Looker, Cognos, MicroStrategy (see [advanced_analytics_solutions_guide.md](../technology/advanced_analytics_solutions_guide.md) for the vendor landscape).
- **Analytics** — the *analysis* layer: ad-hoc and structured investigation that produces *insight* ("why did it happen, and what does it imply?"). Its output is understanding and recommendations, often delivered as analyses, insights, and decision support. It consumes BI foundations but goes one step further.
- **Data Science** — the *modeling* layer: building, validating, and deploying statistical and machine-learning *models* that predict or prescribe at scale. Its output is *models and algorithms*, often embedded in products or automated decisions (see the AI/ML guides under [../technology/ai_llm/](../technology/ai_llm/), e.g. the LLM and ML engineering guides).

The practical boundary is blurry and varies by organization. A helpful rule of thumb: **BI answers "what", analytics answers "why and so what", data science answers "what next" at scale and with automation.** The same person or team may span layers, but the *management* implications differ — BI is governed like an information product (data quality, access, definitions), analytics like a consulting service (demand, scoping, delivery), and data science like an engineering discipline (model lifecycle, validation, deployment, monitoring).

### 1.4 Analytics Management vs Data Management

Analytics management is frequently confused with data management/data governance. They are complementary disciplines over the same territory:

| | **Data management / governance** | **Analytics management** |
|---|---|---|
| Object of management | The **data** — quality, lineage, definitions, access, security | The **analytics function** — people, demand, delivery, value |
| Core questions | Is the data trusted, governed, available? | Are the right questions being answered, with impact? |
| Typical owner | CDO, data governance council, stewards | Head of Analytics / CDO / COE |
| Outputs | Policies, standards, data products, quality metrics | Prioritized roadmap, delivered insights, realized value |

Analytics management *consumes* the outputs of data management — a governed, trusted data foundation is the precondition for credible analytics — and in turn *informs* data management by surfacing what data matters most (see [data_governance_guide.md](../technology/data_governance_guide.md) for the data side in depth). When the two are not coordinated you get the classic failure pattern: the data team measures quality on tables nobody analyzes, while the analytics team complains the data is unusable. Cross-reference: data governance is the "what and who" for data; analytics management is the "how and why" for the analytics function itself.

### 1.5 Why It Matters

The case for managing analytics as a function, rather than letting it grow organically, rests on four arguments:

1. **Better decisions.** The core promise of analytics is evidence-based decision-making — replacing intuition and politics with data where the data genuinely knows. Managed analytics institutionalizes this; unmanaged analytics leaves it to individual heroics.
2. **Competitive advantage.** Thomas Davenport's *"Competing on Analytics"* (Harvard Business Review, January 2006 — the landmark statement of the idea) argued that when products and technology converge across an industry, **business processes become the main differentiator, and analytics competitors "wring every last drop of value" from those processes**. The "analytical competitor" — an organization whose distinctive capability is superior analytics — has a durable edge because the capability (people, processes, data, culture) is hard to copy (see [strategic_management_guide.md](strategic_management_guide.md) for the strategy layer).
3. **Analytics as a differentiator vs table stakes.** There is a spectrum: at one end analytics is a cost of doing business (regulatory reporting, basic scorecards); at the other it is the product or the core of the business model (algorithmic trading, risk-based pricing, personalization). Management decides where on that spectrum the organization chooses to play.
4. **Return on investment.** Analytics is expensive — talent, platforms, data. Without management discipline (prioritization, measurement, adoption focus), most of that spend produces artifacts rather than outcomes. The frequently-cited failure statistic — that a large share of analytics projects never reach production or adoption — is impossible to verify precisely, but every practitioner survey in the last decade has pointed the same direction: **the bottleneck is rarely the model, it is the management**.

### 1.6 Analytics Strategy

An analytics strategy is the answer to: *what will analytics do for this business, and how?* It is a subordinate strategy — it must be derived from and aligned with the business strategy (see [strategic_management_guide.md](strategic_management_guide.md)), not invented in a data-department vacuum.

A workable analytics strategy has five components:

1. **Strategic alignment** — which business objectives does analytics serve? (e.g., grow share of wallet, reduce credit losses, cut operating costs). Every prioritized use case should trace to a business objective; unaligned analytics is hobby work.
2. **Value thesis** — where is the value? (revenue growth, cost reduction, risk reduction, speed, customer experience). Not everything is measurable in P&L terms, but the thesis must be explicit.
3. **Capability model** — what must be true to deliver: data, people, platforms, governance, culture. The strategy names the *capabilities to build*, not just the use cases to run.
4. **Investment framing** — how much, over what horizon, with what funding model (central budget, business-funded, co-funded)? Analytics functions commonly fail from being funded as a cost center while expected to behave like a value center.
5. **Sequencing** — what comes first? (Typically: foundations and quick wins before moonshots.)

The most common strategic error is **strategy as a wish list** — a slide deck of "we will use AI for everything" with no priorities, no sequencing, and no accountability. A useful corrective discipline is to force the strategy to name, for each initiative: the decision it changes, the owner of that decision, and the metric that will prove value.

### 1.7 Analytics Vision

The vision is the ambition underneath the strategy: the "data-driven organization" — an enterprise where decisions at every level are routinely informed by data and analysis, not just by hierarchy and intuition. Vision statements in this space tend to cluster around a few archetypes:

- **"Data-driven decision-making"** — decisions use evidence as a default, not an exception.
- **"Analytics as a competitive weapon"** (Davenport's framing) — analytics is a deliberate differentiator.
- **"AI-enabled / data-empowered" organization** — the contemporary version, where analytics and ML are embedded in products and processes.

The vision matters less for its exact wording than for what it does: it sets the *direction* for the culture and the ambition (see §5.5 on culture), it legitimizes investment, and it gives the analytics leader a mandate. Vision without strategy is aspiration; strategy without vision is a portfolio.

### 1.8 Analytics Value Cases

The strategy and vision are realized through **value cases** — specific, prioritized opportunities where analytics will create measurable business value. A value case is the unit of management in analytics:

| Element | Description | Example |
|---------|-------------|---------|
| **Business problem** | The decision or pain point | "Deposit attrition is 12%/yr; we don't know who is at risk until they leave" |
| **Use case** | The analytics answer | Churn propensity model + retention playbook |
| **Value hypothesis** | How value materializes | X% of predicted churners retained via offers → incremental margin |
| **Metric & baseline** | How we'll know | Retention rate, offer uptake; baseline vs current |
| **Decision owner** | Who will act on it | Head of Retail Deposits |
| **Effort & cost** | What it takes | 3 analysts × 4 months, data engineering support |

**Value prioritization** is the discipline of ranking value cases — by value vs effort, strategic fit, risk, and readiness — so that the scarce analytics capacity goes to the opportunities that matter most (see §4.2 for the prioritization mechanics). A healthy analytics function runs on a **living portfolio of value cases**: some in discovery, some in delivery, some in production being measured for realized value. The portfolio — not the individual project — is the true object of analytics management.

---

## 2. The Analytics Organization: Operating Models and Design

The first structural decision in analytics management is **how the analytics function is organized**. This is the operating model question: where does the analytics talent sit, who owns the priorities, and how does work flow between the center and the business?

### 2.1 The Three Operating Models

The standard menu of analytics operating models is three archetypes (the same three used for enterprise AI/data functions generally; this is common practitioner usage rather than a single canonical source):

**1. Centralized (Center of Excellence / COE).** One central analytics team serves the whole enterprise. The team owns talent, platforms, standards, and priorities; business units submit demand.

- *Strengths*: critical mass of scarce talent, consistent standards and platforms, easier governance, career paths, economies of scale, one voice on data needs.
- *Weaknesses*: distance from business realities, prioritization by committee, "queue mentality" (business feels it is *ordering* from IT), slow response, analysts become order-takers, business buy-in is fragile.
- *Best for*: organizations starting out, with scarce talent and immature data; or highly standardized, enterprise-wide analytics (e.g., group-level risk analytics).

**2. Federated (Decentralized / Embedded).** Analysts sit *inside* business units, reporting to business leaders. Each unit runs its own analytics with its own tools and priorities.

- *Strengths*: deep business intimacy, fast response, local ownership, high adoption of results, alignment to unit incentives.
- *Weaknesses*: duplication of roles and tooling, inconsistent definitions and metrics, no critical mass, weak career paths, "shadow analytics" that undermines the single source of truth, politics-driven priorities, harder to govern.
- *Best for*: organizations where units are genuinely autonomous (diverse businesses, conglomerates) or where speed-to-insight matters more than scale.

**3. Hub-and-Spoke (Hybrid).** A central **hub** (the analytics COE) owns the platforms, standards, methods, and the scarce deep-specialist talent; **spokes** — embedded analysts or small analytics teams in the business units — own business-facing delivery and local priorities. The hub governs and enables; the spokes execute and translate.

- *Strengths*: balances scale and intimacy; the hub prevents fragmentation while the spokes prevent ivory-tower isolation; a proven "best of both" pattern; the most commonly recommended model for mid-to-large enterprises.
- *Weaknesses*: the hardest to run well — requires genuine matrix management, clear decision rights, and constant communication; can degenerate into either central control (spokes as puppets) or central irrelevance (spokes as renegades).
- *Best for*: most mid-to-large enterprises with multiple business units sharing common data platforms — which is precisely the profile of a regional or mid-size bank.

### 2.2 The Center of Excellence (COE)

The COE — also called the Analytics Center of Excellence, Analytics Competency Center, or Data & Analytics Office — is the central organizational vehicle of analytics management. Its job is not to do all the work, but to make the work excellent and reusable:

- **Standards and methods**: naming conventions, metric definitions, model development standards, tooling choices, delivery playbooks.
- **Platforms and tooling**: owning the shared analytics stack (warehouse, BI, ML platform) so units don't each buy their own.
- **Talent and capability**: career paths, training, hiring pipelines, upskilling the business in self-service.
- **Governance**: demand intake, prioritization, portfolio management, value measurement.
- **Advocacy**: evangelizing data-driven decision-making, communicating wins, building the culture.

The COE's scope varies — some are thin (platform + governance), some thick (full delivery), some are *virtual* (a community of practice over federated teams rather than a line organization). The recurring pitfall: **a COE that becomes a monopoly** — everything must pass through it, so it becomes the bottleneck it was meant to dissolve. The COE should own *how* analytics is done; the business should own *what* analytics is for.

### 2.3 Operating Model Comparison

| Dimension | Centralized / COE | Federated | Hub-and-Spoke |
|-----------|-------------------|-----------|---------------|
| **Structure** | One central team serves all units | Analytics embedded in each business unit | Central hub (standards/platforms/specialists) + embedded spokes |
| **Reporting lines** | Analysts report to the central analytics leader | Analysts report to business-unit leaders | Spoke analysts report to the business (dotted to hub); hub reports to CDO/analytics head |
| **Pros** | Critical mass; consistent standards; governance; career paths; economies of scale | Business intimacy; speed; ownership; adoption | Scale + intimacy; consistent standards with local relevance |
| **Cons** | Distance from business; queue mentality; adoption risk | Duplication; inconsistency; weak governance; no critical mass | Complexity; matrix friction; needs strong leadership |
| **Best for** | Start-ups of the function; standardized enterprise analytics | Autonomous diverse units; speed-critical niches | Most mid/large enterprises with shared platforms — incl. banks |

Most analytics functions **evolve through the models**: start centralized to build capability and standards, then federate delivery via hub-and-spoke as the business matures and trust grows. Jumping straight to full federation without a hub typically produces anarchy; staying centralized forever produces irrelevance.

### 2.4 Analytics Placement: Where Does Analytics Report?

A second structural decision is *where the analytics function sits in the org chart*. There is no universal right answer — the placement signals what the organization wants from analytics:

- **Under the CDO (Chief Data Officer)** — the most common modern placement for analytics: data and analytics form one "data organization" with joined-up governance of data assets and analytical use of them. The CDO's mandate spans data governance (see [data_governance_guide.md](../technology/data_governance_guide.md)) and analytics delivery. Risk: analytics becomes a support service to data programs and loses business teeth.
- **Under the CTO / CIO / IT** — common historically; guarantees technical plumbing. Risk: analytics is perceived as an IT cost center; business intimacy and value framing suffer. Works when the CTO is business-fluent and analytics is treated as product delivery, not infrastructure.
- **Under the CFO** — frequent in finance-led organizations, because finance owns the budgets and the data. Strengthens value discipline (ROI, prioritization) but can over-focus on finance use cases and cost metrics.
- **Under the CMO** — seen in consumer companies where customer analytics is the dominant use case. Deep business impact in marketing; risks neglecting risk/operations use cases.
- **Distributed into business units** — the federated end of the spectrum; each unit's analytics reports to its own P&L owner.
- **A standalone Chief Analytics Officer (CAO) / Head of Analytics** — the purest form: analytics as a first-class function reporting to the CEO or COO, signaling that analytics is strategic. Rare outside analytics-native firms because it needs CEO-level sponsorship to survive.

**Verification note:** the "analytics under the CDO" placement is the common pattern in large enterprises and in banks (the CDO office typically houses or coordinates data science teams), but placement varies widely by industry and company; treat the list above as observed practice, not a standard. In banks, the CDO office is usually the home for the data platform and increasingly for the analytics COE, while model risk governance sits separately in risk (see §7.2).

### 2.5 Organizational Design: The Analytics Function

Beyond the placement, three design choices shape the analytics function's day-to-day:

1. **Reporting lines and matrix.** Who formally manages analysts — the analytics leader, the business, or both? A dotted-line matrix (business owns the analyst's day-to-day; the analytics leader owns methods, quality, career) is the hub-and-spoke norm. Clear decision rights matter more than the org chart: who decides *what* gets worked on, *how* it's done, *when* it ships, and *who* evaluates the analyst.
2. **Centers vs pods.** Some functions organize analysts by *capability* (a modeling team, a reporting team); others by *domain* (a retail-analytics pod, a risk-analytics pod); the modern pattern is cross-functional **pods** — a small mixed team (analyst/data scientist/engineer) owning a domain or product end-to-end (see §5.2).
3. **Governance structure.** A functioning analytics function has a cadence: a **demand review** (intake and triage), a **prioritization forum** (business and analytics leadership agreeing the portfolio), and a **value review** (measuring realized outcomes). These forums are the organizational skeleton of governance (see §4).

### 2.6 Analytics Maturity: Models and Ladders

"Maturity" in analytics means how advanced, embedded, and value-producing the organization's analytics capability is. The two most cited framings:

**1. The analytics capability ladder (Davenport's "analytical competitor").** Davenport's *Competing on Analytics* (HBR, 2006) distinguished organizations by how strategically they use analytics — from those using analytics barely at all, through using it for decision support, to **analytical competitors** who *compete on* analytics as a core differentiator. His follow-up work (with Harris and Morison, *Analytics at Work*, 2010) introduced the **DELTA model** — five factors that predict analytics success: **D**ata, **E**nterprise (enterprise-wide integration), **L**eadership, **T**argets, **A**nalysts. DELTA remains a practical diagnostic: most analytics failures trace to weakness in leadership or enterprise integration, not to data or analysts.

**2. The analytics maturity ladder (capability progression).** The descriptive → diagnostic → predictive → prescriptive ladder of §1.2 doubles as a maturity framing: organizations mature by climbing from "what happened" to "what should we do". Related industry models (Deloitte, Gartner, various) formalize this into 4-6 stage ladders with names like "analytically impaired → analytical competitor", or stages from "basic reporting" to "AI-driven". The exact stage names differ by vendor; the *structure* is consistent: **data foundations → reporting → insight → prediction → embedded/automated decisions**.

**Verification note:** there is no single canonical "analytics maturity model" — the phrase covers a family of similar ladders (Gartner's, Deloitte's, Davenport's, vendor-specific ones). This guide uses the family, not any one vendor's version, and flags specifics where they matter. The "Analytics 1.0–3.0" framing is informal historical shorthand (see §1.2), not a formal maturity standard.

### 2.7 Analytics Maturity Assessment

Maturity assessment is the diagnostic that tells you where the function stands and where to invest next. A practical assessment scores the organization on four dimensions:

| Dimension | What it measures | Typical signs of immaturity → maturity |
|-----------|------------------|----------------------------------------|
| **Data** | Quality, availability, governance, single source of truth | Excel silos → governed, certified data products |
| **People** | Skills, roles, career paths, analytics literacy | One lone analyst → full role taxonomy + business upskilling |
| **Process** | Intake, prioritization, delivery, adoption, measurement | Ad-hoc requests → governed demand + value tracking |
| **Culture** | Whether decisions actually use data | "Data to confirm opinions" → "data to change decisions" |

Assessment outputs typically include a maturity level per dimension, a gap analysis against target, and a prioritized investment plan. The trap to avoid: running the assessment as a scoring exercise that produces a pretty chart and no decisions. Maturity assessments earn their keep only when they drive the roadmap — which is why they should be tied to the analytics strategy (§1.6) and reviewed at least annually.

---

## 3. The Analytics Lifecycle: From Problem to Action

Analytics management is, at its core, the management of a **value chain** — the journey from a business problem to a business outcome. Every analytics engagement, from a one-day ad-hoc analysis to a year-long ML program, runs the same chain.

### 3.1 The Analytics Value Chain

The canonical chain — in its most common formulation — is:

**Problem → Data → Analysis → Insight → Decision → Action**

| Stage | Question | Output | Failure mode if skipped |
|-------|----------|--------|------------------------|
| **Problem framing** | What decision are we informing? | A crisp business question + success metric | Analysts solve the wrong problem beautifully |
| **Data** | Do we have what we need? | Acquired, prepared, validated data | Garbage in, garbage out; "data work" never ends |
| **Analysis** | What does the data say? | Models, analyses, evidence | Analysis for its own sake, no decision relevance |
| **Insight** | What does it mean for the business? | A communicated, interpretable insight | Insight trapped in a notebook or slide |
| **Decision** | What will we do differently? | A decision taken (or explicitly deferred) | "Interesting, but we did nothing" |
| **Action** | Did it work? | Executed change, measured outcome | No feedback loop; value never realized |

The chain has two properties that drive everything else in this guide:

1. **Value only realizes at the far end.** An analysis that ends at "insight" has produced *cost*, not value. The last mile — decision and action — is where value materializes, and it is where most analytics fails (the "insight-to-action gap", §3.6).
2. **The chain is a loop, not a line.** Action produces new data; measurement feeds back into problem framing. Mature functions manage the loop; immature ones manage the line and wonder why value is elusive.

### 3.2 Stages in Detail

**Problem framing.** The single highest-leverage stage. A well-framed problem states: the decision to be informed, the decision owner, the timeframe, the success metric, and the constraints (data, cost, risk). Framing tools: the "five whys" to find the real question; decision-owner interviews; "what would we do differently with the answer?" as the test of relevance. Most analytics rework traces to skimped framing.

**Data.** Acquisition, preparation, and validation — the territory of data engineering ([closed_loop_data_engineering_guide.md](../technology/ai_llm/closed_loop_data_engineering_guide.md) covers the engineering loop in depth) and data governance ([data_governance_guide.md](../technology/data_governance_guide.md) covers quality and definitions). Management's job here is to *bound* the stage — data work is infinitely extensible, so the manager's discipline is "enough data for the decision, agreed up front".

**Analysis.** Descriptive and diagnostic work, or predictive/prescriptive modeling (the craft is covered in the AI/ML guides; management's job is scoping, method quality, and deadlines). The management artifact is the **analysis plan**: question, approach, data, milestones, deliverables, and the review gate.

**Insight.** Interpretation and communication: translating model output or analysis findings into a business-usable form — a narrative, a recommendation, a quantified trade-off, a visual. This is the stage where the **translator** role (§5.1) earns its keep and where communication skill ([communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)) matters as much as statistical skill.

**Decision.** The insight must land in an actual decision process: a steering committee, a budget cycle, an operating rhythm. Analytics leaders deliberately engineer this — they know *whose* decision it is, and they put the insight in front of that person at the right moment. A decision can legitimately be "no change" — but it must be explicit.

**Action.** Execution of the decision (launch the campaign, adjust the pricing, restructure the process) and **measurement** of the outcome against the success metric from framing. Measurement closes the loop and generates the attribution evidence that governance needs (§4.4).

### 3.3 The Analytics Workflow: Request to Adoption

In the running of the function, the lifecycle takes the operational form of a workflow:

**Request → Intake & Triage → Prioritization → Delivery → Review → Adoption → Measurement**

- **Request** arrives from the business (or is discovered by the function itself) — from "can I get a dashboard?" to "can you build a churn model?"
- **Intake & triage** (§4.1) captures the request, qualifies it against the framing questions, and routes it: trivial (self-service answer), standard (delivery backlog), or substantial (portfolio case).
- **Prioritization** (§4.2) decides whether it enters the portfolio and with what urgency.
- **Delivery** runs the lifecycle stages; the management artifact is the delivery review (mid-point and final), checking not just *done-ness* but *decision-usefulness*.
- **Adoption** is the hand-off to the decision owner and users — training, documentation, embedding into process, change management. Adoption is a *managed* stage, not an assumption.
- **Measurement** verifies realized value and feeds the value ledger.

### 3.4 Analytics Use Cases: A Classification

Use cases can be classified by the *role analytics plays in the decision* — a classification that maps directly onto delivery style and governance weight:

| Type | What analytics does | Examples | Delivery style |
|------|---------------------|----------|----------------|
| **Decision support** | Informs a human decision with evidence | Ad-hoc analyses, market studies, "should we enter X?" | Consulting-like: fast, insight-focused |
| **Optimization** | Finds the best configuration within constraints | Pricing, portfolio mix, resource allocation | Operations-research: models + solvers |
| **Prediction** | Forecasts an outcome | Demand forecasts, churn scores, credit risk scores | Model lifecycle: build → validate → deploy → monitor |
| **Automation** | Makes or executes decisions automatically | Fraud screening, algorithmic pricing, real-time next-best-action | Product engineering: embedded, governed like software |

The management implication: **don't run all four the same way.** Decision support needs speed and business intimacy; automation needs rigorous model governance (§7.4) and engineering discipline. A common error is treating a regulatory-grade automation model like an ad-hoc analysis — or vice versa.

### 3.5 Value Realization and the "Last Mile"

The phrase **"the last mile"** (or the "insight-to-action gap") describes the most persistent failure in analytics: the distance between producing an insight and realizing its value. Studies and practitioner surveys repeatedly find that the majority of analytics initiatives stall short of adoption — the exact figures vary by survey and are not reliably verifiable, but the *pattern* is consistent and widely acknowledged. The causes are management causes, not technical ones:

- **No decision owner** — the insight has no designated actor.
- **No decision moment** — the insight isn't wired into a process, budget, or review where a decision actually gets made.
- **Adoption treated as an afterthought** — the deliverable lands, the analyst moves on, nobody trains or champions.
- **No measurement** — even where action happens, nobody proves the value, so the next initiative loses sponsorship.
- **Cultural resistance** — "we already know our customers" (see §5.5).

The management antidotes: make **decision ownership a criterion in framing** ("who will act, and when?"), budget for **adoption and change management** as first-class project stages, assign a **business sponsor** per value case, and run a **value measurement** step that closes the loop. The analytics leader's single most important habit is asking, at every review: *"what decision does this change, and who is making it?"*

---

## 4. Analytics Governance: Demand, Value, Risk, and Culture

Governance in analytics management is the machinery that decides **what analytics work gets done, in what order, by whom, and with what accountability**. It is the answer to the eternal analytics complaint: "everyone wants everything now, we have five people, and nobody can tell us what matters." Good governance turns that chaos into a managed portfolio.

### 4.1 Demand Management: Intake and Triage

**Demand management** is the front door of the analytics function — the structured capture of everything the business wants from analytics. The mechanics:

1. **Intake channel(s)**: a single, known entry point (portal, mailbox, Jira/ServiceNow project, or the COE itself). Multiple ad-hoc channels — "just email the analyst directly" — bypass governance and quietly recreate the reporting factory.
2. **Request capture**: every request is captured with a minimum set of fields: requester, business owner, business question, decision it informs, desired date, rough value expectation.
3. **Triage**: requests are classified on intake into:
   - **Self-service** — answerable by the requester with existing dashboards/data (route to enablement, not to an analyst).
   - **Standard** — a bounded analysis or report (enter the delivery backlog).
   - **Substantial** — a value case (enter portfolio prioritization).
   - **Reject/redirect** — unclear value, no owner, or no data (return with coaching).

Triage has a dual purpose: it *filters* (protecting scarce capacity) and it *educates* (teaching requesters to frame business questions rather than place orders). A well-run intake makes demand visible — the first step to managing it. Without intake visibility, the analytics leader cannot answer the board's most basic question: "what are we actually working on?"

### 4.2 Prioritization: Value vs Effort

With demand captured, the portfolio question is: *which of these, now?* The workhorse is a **value vs effort** assessment — every candidate scored on two axes and placed on a 2×2 (quick wins / big bets / fill-ins / time sinks). The scoring can be structured with a lightweight framework:

- **Value** (business impact): revenue/margin impact, cost saving, risk reduction, strategic fit, customer experience, regulatory necessity.
- **Effort** (cost to deliver): analyst-days, data readiness, platform needs, dependency on other teams, ongoing maintenance.
- **Confidence**: how sure are we of both estimates? (This is the "C" in the well-known **RICE** prioritization framework — Reach, Impact, Confidence, Effort — borrowed from product management: score = (Reach × Impact × Confidence) / Effort. RICE is Intercom's published product-prioritization method; applying it to analytics demand is a common adaptation.)
- **Urgency/risk**: deadlines (regulatory), opportunity windows, dependencies.

The prioritization forum — a recurring meeting of business and analytics leadership — ratifies the portfolio. Its outputs: a ranked backlog, agreed criteria, and a visible record of what was *not* chosen (saying no explicitly is the point). Prioritization failure modes: everything is "top priority" (no ranking), prioritization by loudest voice, and the deadliest — **prioritizing by what is easy rather than what matters**, because analytics teams naturally drift to tractable problems.

### 4.3 The Analytics Backlog and Roadmap

The outputs of prioritization are managed as a **portfolio**: a **backlog** (the ranked, unstarted work) and a **roadmap** (the planned delivery over the next quarters — what ships when, with which value case).

| Artifact | Horizon | Content | Managed by |
|----------|---------|---------|------------|
| Backlog | Immediate | Ranked, sized work items ready for delivery | Delivery lead / COE |
| Roadmap | 2-4 quarters | Value cases with dates, owners, dependencies | Analytics leader + business sponsors |
| Strategy portfolio | 1-3 years | Capability investments (platforms, talent, data) | CDO / Head of Analytics |

A healthy roadmap is **visible, owned, and adjusted at a fixed cadence** (quarterly at minimum). Two disciplines keep it honest: (1) *capacity-based planning* — the roadmap should reflect actual analyst capacity, not aspiration; (2) *value-based review* — every quarter, shipped items are revisited for realized value (§4.4) and the learning feeds the next prioritization round. The roadmap is also the analytics leader's principal **communication instrument** — it is how the business sees what the function is doing and why (see [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)).

### 4.4 Value Measurement: Outcomes and KPIs

"What did analytics actually deliver?" — the hardest and most important governance question. The discipline of value measurement:

- **Outcomes, not outputs.** Outputs are deliverables (dashboards shipped, models built, analyses delivered). Outcomes are business results (retention up, losses down, cycle time cut). Analytics functions are routinely managed on outputs because they are easy to count; the governance question must always push to outcomes.
- **KPIs: leading vs lagging.** Lagging indicators (revenue, profit, churn rate) tell you what already happened; leading indicators (model adoption rate, % of decisions using analytics, offer uptake, dashboard usage) predict where outcomes are heading. Analytics value tracking needs both: *leading* indicators prove the analytics is being *used*; *lagging* indicators prove it *matters*. A useful rule: if usage/engagement metrics are flat, the lagging business metrics will never move — adoption is the leading indicator of value.
- **Baselines and counterfactuals.** Value claims need a baseline ("churn was 12% before the model; retention offers cut predicted-churner attrition by a third vs the control group"). Where possible, run controlled experiments or matched comparisons — this is the same rigor the function applies to its own products.
- **The value ledger.** A maintained register of value cases with: expected value, realized value, evidence, and status. Reviewed quarterly. The ledger is the single best defense of the analytics budget — it converts the function from a cost center narrative to a value narrative.

### 4.5 Analytics ROI and Attribution

**ROI cases** — "this analytics program delivered X" — live and die on **attribution**: how much of the observed business change is *caused* by the analytics, versus market conditions, other initiatives, or luck. The attribution toolkit:

- **Experiments** (A/B tests, holdout groups) — the gold standard where feasible.
- **Before/after with control** — where randomization isn't possible, compare against a control population or a synthetic counterfactual.
- **Model-based attribution** — quantify the incremental contribution of the analytics-driven decision vs the prior rule (e.g., "the model-selected portfolio outperformed the old rule by N bps").
- **Conservative reporting** — where attribution is genuinely impossible, say so. Over-claiming ROI destroys credibility faster than under-claiming.

Honest attribution is not just an accounting exercise: it is the feedback loop that tells the function *which types of use case actually pay* — and therefore where to invest next. The governance framing to insist on: "analytics contributed X" is a **claim with evidence**, not a slogan. For the broader discipline of measuring and communicating value up the chain, see [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md).

### 4.6 Ethics and Responsible Analytics

Responsible analytics is the management discipline over the *side effects* of analytics: fairness, privacy, transparency, and harm. The analytics function must govern not only whether the work is *valuable* but whether it is *right*:

- **Fairness** — models and analyses can disadvantage groups (biased credit scoring, biased targeting). Fairness is assessed and mitigated at design time, not after complaints (the deep treatment is in the AI governance guides — see [implementing-responsible-ai.md](../technology/ai_llm/implementing-responsible-ai.md)).
- **Privacy** — analytics operates on personal data; use must be proportionate, consented, and minimized, under the applicable regimes (PDPA in Singapore, GDPR in Europe) — the data-side controls are in [data_governance_guide.md](../technology/data_governance_guide.md).
- **Transparency** — decision subjects should be able to understand (in principle) why a decision was made; explainability is a governance requirement for consequential models, and a regulatory one in banking (§7.4).
- **Purpose limitation** — analytics built for one purpose can drift into another (a marketing model used for credit decisions). Governance must police the boundary.

Operationally: an **ethics checklist** in the intake/triage stage for any analytics touching people; an **escalation path** for concerns; and a standing review forum (the bank's model risk committee or data ethics council) for consequential cases. Ethics governance is not bureaucratic padding — in banking it is a regulatory expectation and a license-to-operate issue (see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md)).

### 4.7 Analytics Risk

Beyond ethics, the function must govern the *operational* risks of analytics itself:

- **Misuse** — an insight applied outside its assumptions ("the model was built on retail deposits, not SME").
- **Bias and data drift** — models silently degrade as the world changes; monitoring is a governance requirement, not an option (see the MLOps and model-monitoring material in the AI guides).
- **Compliance** — analytics touching regulated decisions (credit, fraud, pricing, AML) falls under model risk management and supervisory scrutiny (SR 11-7 in the US; MAS guidance in Singapore) — see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) for the full banking treatment.
- **Dependency risk** — decisions becoming dependent on fragile analytics (a model that silently fails, a dashboard fed by a broken pipeline).

The governance instruments: a **risk register** for analytics (like any other risk), **model governance** for consequential models (validation, approval, monitoring, retirement — §7.4), and **operational guardrails** (alerting, rollback, human-in-the-loop where consequences are high).

### 4.8 Analytics Standards: Definitions and Single Source of Truth

Analytics quality ultimately rests on **shared standards**, and the most important standard is a shared language:

- **Metric definitions** — "active customer", "NPL", "revenue" must mean one thing enterprise-wide; otherwise analytics contradicts analytics. Ownership lives in data governance ([data_governance_guide.md](../technology/data_governance_guide.md) — metric dictionaries, business glossaries, reference data).
- **Naming and conventions** — consistent naming for tables, fields, dashboards, and models; documented conventions make analytics legible and reusable.
- **Single source of truth** — one governed version of key facts (customer, product, risk data) that all analytics draws from, instead of each unit's spreadsheet reality. This is the *analytic* payoff of data governance: without it, analytics produces "multiple versions of the truth" and the function loses the argument to the CFO.
- **Delivery standards** — reproducibility (code over clicks), documentation, testing for models, review checkpoints.

The governance relationship is deliberately two-way: analytics management *consumes* data standards and *surfaces* the demand for better data — feeding the data governance roadmap with the analytics function's priorities.

### 4.9 Analytics Culture: The Data-Driven Organization

Culture is the governance layer that cannot be written down — the habits and norms about how decisions get made. The target culture is **data-driven decision-making**: evidence as the default in decisions, curiosity rewarded, and "let's look at the data" a normal response to disagreement. (The intellectual roots run to "evidence-based management" — the movement popularized by Pfeffer & Sutton's work of the same name (2006) — arguing that decisions should rest on evidence, not authority or fashion.)

The analytics leader manages culture through levers, not slogans:

1. **Model the behavior** — the leadership team must visibly change decisions based on data; nothing kills the culture faster than executives who commission analytics and then ignore it.
2. **Make data easy** — self-service access, good dashboards, low friction to "check the data" (the product layer, §6).
3. **Reward the behavior** — celebrate analysts whose work changed a decision; hold decision owners accountable for using (or explicitly rejecting) the evidence.
4. **Upskill the business** — analytics literacy for non-analysts (what a p-value is, what a model can and cannot do) is a direct culture investment (§5.4).
5. **Change management** — adoption of analytics-driven ways of working is a change program: sponsors, champions, training, and patience. The culture shift from "reports for review" to "decisions from data" typically takes years and is the difference between an analytics *function* and an analytics *culture*.

---

## 5. The Analytics Team: Roles, Topologies, and Talent

The people are the asset. The team chapter covers the role taxonomy, how teams are structured, and how talent is grown and kept.

### 5.1 Roles in the Analytics Team

The modern analytics function is built from a set of distinct roles (the taxonomy below is the common industry practice; job titles vary, but the *functions* are stable):

| Role | Core function | Typical outputs | Key skills |
|------|---------------|-----------------|------------|
| **Data Analyst** | Descriptive/diagnostic analysis and reporting | Dashboards, reports, ad-hoc analyses, insights | SQL, BI tools, statistics, business communication |
| **Data Scientist** | Predictive/prescriptive modeling | Models, experiments, recommendations | Statistics, ML, coding, experimental design, domain sense |
| **Analytics Engineer** | Data preparation, pipelines, and analytics tooling | Transformed datasets, dbt models, feature stores, tooling | SQL/dbt, data modeling, orchestration, software practices (see the data engineering guides, e.g. [closed_loop_data_engineering_guide.md](../technology/ai_llm/closed_loop_data_engineering_guide.md)) |
| **ML Engineer** | Model deployment, serving, and monitoring | Production ML services, monitoring, MLOps | Software engineering, ML platforms, CI/CD (see the AI/ML guides under [../technology/ai_llm/](../technology/ai_llm/)) |
| **Analytics Translator** | The bridge between business and analytics | Framed problems, scoped projects, interpreted insights, adoption | Business fluency + analytics literacy (see below) |
| **Data/Platform Engineer** | The data platform itself | Warehouse, pipelines, data products | Data infrastructure (see the data engineering guides) |
| **Analytics Leader (Head of Analytics)** | Function leadership | Strategy, portfolio, team, stakeholders | Leadership, prioritization, communication (see [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md)) |

**The analytics translator** — a role McKinsey has championed (its QuantumBlack practice published the influential *"Analytics translator: the new must-have role"*): translators are business-fluent people who speak both the language of the business and of analytics — they frame business problems as analytics problems, scope the work, and convert findings into decisions and action. McKinsey's estimate that US demand for translators could reach **2-4 million by 2026** is widely cited (it is a consultant forecast, not an audited statistic — treat the magnitude as directional). The recurring practical insight from McKinsey's work and industry practice: **translators are more often grown than hired** — trained from business analysts, product people, and consultants — because the role needs deep business context more than deep statistics.

**The analytics leader** role deserves special note: heads of analytics typically fail not on analytics but on leadership — portfolio discipline, stakeholder management, and the courage to say no. The leadership layer is covered in depth in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) (leading the team) and [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) (managing up/down/sideways).

### 5.2 Team Topologies

How the roles are assembled into teams mirrors the operating model (§2.1), at a finer grain:

- **Centralized team** — all roles in one function, serving the whole enterprise (matches the centralized operating model).
- **Embedded teams** — analysts physically and organizationally inside business units (matches federated).
- **Hybrid** — a central core (platform, specialists, standards) + embedded analysts (matches hub-and-spoke).
- **Pods (cross-functional analytics pods)** — the emerging team-level pattern: small, stable, mixed teams (analyst + data scientist + analytics engineer, sometimes + product/domain person) each owning a *domain or product* end-to-end — e.g., a "retail deposits analytics pod" or a "fraud analytics pod". Pods align with the product mindset (§6.3): ownership, autonomy, and accountability for outcomes rather than tasks. Pods are the analytics analogue of product squads in modern software organization; the pattern is common in analytics-native and product-led companies, and increasingly adopted in banks' data offices.

The topology choice interacts with the operating model: hub-and-spoke operating models are typically delivered by a mix of central specialist teams and domain pods. **The recurring principle: structure follows the decision.** Teams should be shaped around the decisions and domains they serve, not around the tools they use.

### 5.3 Team Skills: The Three-Legged Stool

The skills profile of a healthy analytics team spans three legs — and the *mix* matters more than any single specialist:

1. **Technical** — SQL, statistics, ML, engineering, tooling. Necessary but insufficient alone.
2. **Business** — domain knowledge (banking products, risk, customers), commercial judgment, the ability to tell a valuable question from a tractable-but-useless one.
3. **Communication** — writing, presenting, storytelling with data, stakeholder management ([communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)).

The failure pattern to design against: teams that are technically brilliant and business-deaf produce models nobody wants; teams that are business-fluent and technically shallow produce plausible nonsense. The **translator** role exists precisely to hold these legs together — and the analytics leader's job is to make sure all three legs exist on every team, even if they live in different people.

### 5.4 Analytics Talent: Hiring, Retention, Upskilling

- **Hiring.** Analytics talent is scarce and expensive. The hiring discipline: hire for the *mix* (a team of only junior analysts or only senior scientists both fail), use structured assessments that test real work (SQL/data tasks, case framing — not trivia), and screen for communication and business sense as rigorously as for technical skill. For scarce senior roles, consider "grow or rent": consulting/contract specialists for spikes, internal development for the core.
- **Retention.** Analytics professionals leave for three reasons: no career path (the "senior analyst forever" trap), no impact (their work doesn't change anything — the governance failure of §3.5), and no growth (stale tooling, no learning). The retention levers are therefore structural: dual career tracks (individual contributor vs manager), visible impact (value ledger feedback), and continuous capability investment. Compensation is table stakes; these are the differentiators.
- **Upskilling and democratization.** "Analytics upskilling" — raising the analytics literacy of the *whole* workforce — is the supply-side answer to demand pressure: every question that a trained business user can answer themselves is capacity freed for high-value work. Democratization programs: self-service training, "analytics bootcamps" for business teams, champions networks, and the self-service product layer (§6.2). Democratization is also the cultural engine of §4.9 — people who can check the data start to think with data.

---

## 6. Analytics Products and Tools

The outputs of the analytics function — dashboards, reports, models, self-service assets — are increasingly managed as **products**: things with users, adoption, and lifecycles, rather than one-off deliverables. This chapter covers the product layer and the tooling that supports it.

### 6.1 The Analytics Product Family

| Product type | What it is | Users | Governance weight |
|--------------|-----------|-------|-------------------|
| **Dashboards** | Persistent visual views of metrics (BI) | Broad — executives to ops | Light-medium: definitions, access, refresh SLAs |
| **Reports** | Structured, often scheduled deliverables | Management, regulators | Medium: definitions, review cycles, versioning |
| **Analyses / insight briefs** | One-off or recurring decision-support documents | Decision owners | Light: framing, quality review |
| **Models** | Predictive/prescriptive ML or statistical models | Systems and decision-makers | Heavy: validation, monitoring, model risk (§7.4) |
| **Self-service assets** | Certified datasets, semantic layers, curated metrics | Business users | Medium: certification, ownership, support |
| **Embedded analytics** | Analytics inside operational apps and products | End customers, staff | Heavy: engineering discipline, monitoring |

The BI vendor landscape (Tableau, Power BI, Looker, and the broader analytics platform market — including the AI-assisted entrants covered under augmented analytics, §6.7) is treated in depth in [advanced_analytics_solutions_guide.md](../technology/advanced_analytics_solutions_guide.md); the point here is the *management* of these products, whatever the vendor.

### 6.2 Self-Service Analytics: Governed Democratization

**Self-service analytics** — business users answering their own questions from governed data assets — is the standard answer to the capacity problem, and it sits on a permanent tension: too little governance and you get spreadsheet anarchy (multiple versions of the truth); too much and self-service dies (nobody can get access). The resolution is **governed self-service**:

- **Certified data layer**: curated, documented, governed datasets/metrics that users are *allowed* and *encouraged* to use directly (the "certified data products" of the data mesh world — see §9.3).
- **Guardrails, not gates**: permissions, row-level security, and definitions enforced in the platform — rather than human approval queues for every query.
- **Enablement**: training, templates, champions — the upskilling engine of §5.4.
- **Clear boundary**: what self-service can and cannot do (self-service for *questions about certified data*; analysts for *new questions, new data, or models*). The boundary must be explicit or self-service quietly collapses back into "email the analyst".

Self-service is measured like any product: adoption (active users, queries), quality (how often users need analyst rescue), and trust (do users still keep their own shadow spreadsheets — the canary for governance failure).

### 6.3 Analytics Product Management

The product mindset applied to analytics means treating the deliverables as **products with users, adoption curves, feedback loops, and lifecycle management** — and increasingly, appointing an **analytics product manager** (the role exists in larger analytics-native organizations and is spreading; title variants include "analytics product owner" and "BI product manager"). The practice:

- **Users and personas**: a dashboard's users are understood — who, what decision, how often, what would make them stop using it.
- **Adoption as the metric**: products that aren't used are failures regardless of technical quality; usage and engagement are tracked per product.
- **Iteration**: products evolve through feedback — quarterly product reviews, usage analytics, "retire or refresh" decisions. A dashboard that nobody opens should be killed or reworked, not maintained forever.
- **Product owners**: each analytics product has an owner accountable for its fitness — the human anchor of the product mindset.
- **Portfolio view**: the analytics product portfolio (all dashboards/models/self-service assets) is reviewed as a portfolio — value, usage, cost, redundancy — mirroring the value-case portfolio of §4.3.

The product mindset is the antidote to the **reporting factory**: it converts "we delivered what was asked" into "we operate assets that are used and valued."

### 6.4 Analytics Tooling

The tool landscape, managed as a coherent set (details of vendors in [advanced_analytics_solutions_guide.md](../technology/advanced_analytics_solutions_guide.md)):

- **BI and visualization**: Tableau, Power BI, Looker, ThoughtSpot, Qlik, MicroStrategy, open-source (Metabase, Superset).
- **Notebooks and analysis**: Jupyter, RStudio/Posit, Hex, Deepnote — the analyst's workbench.
- **Statistical/ML platforms**: Python/R ecosystems, Dataiku, SAS, Alteryx, and the ML platforms (Databricks, SageMaker, Vertex) covered in the AI/ML guides.
- **Data platforms**: warehouses/lakehouses (Snowflake, BigQuery, Databricks), transformation (dbt), orchestration (Airflow) — see the data engineering guides, e.g. [closed_loop_data_engineering_guide.md](../technology/ai_llm/closed_loop_data_engineering_guide.md).
- **Collaboration and governance**: catalog (Datahub, Collibra, OpenMetadata), lineage, metric stores.

Tooling management principles: **fewer platforms, more adoption** (tool sprawl is the norm and the enemy — every extra tool is a governance gap); **buy before build** for commodity capabilities (vendor selection discipline in [vendor_management_guide.md](vendor_management_guide.md)); and **fit to the user**, not the fashion (analysts need notebooks, executives need dashboards, and forcing one tool on both fails).

### 6.5 The Modern Analytics Stack

The "modern analytics stack" — the reference architecture most new analytics functions adopt — is built on cloud data platforms:

**Sources → Ingestion → Warehouse/Lakehouse → Transformation → Semantic layer → BI/Notebooks → Models/Apps**

- **Warehouse/lakehouse** (Snowflake, BigQuery, Databricks) — the single store for analytical data.
- **Transformation** (dbt and peers) — versioned, testable SQL transforms that turn raw into modeled.
- **Semantic layer / metrics store** — the governed definitions layer that makes "revenue" mean one thing (the technical home of §4.8 standards).
- **BI and notebooks** — the consumption layer.
- **Orchestration and observability** — Airflow/Dagster, lineage, data quality checks (see the data engineering guides, e.g. [closed_loop_data_engineering_guide.md](../technology/ai_llm/closed_loop_data_engineering_guide.md) and [data_governance_guide.md](../technology/data_governance_guide.md)).

The management implications of the modern stack: it *lowers the cost of self-service* (the semantic layer is what makes governed self-service possible), it *centralizes governance in code* (definitions and quality as code, not memos), and it *changes the analyst's job* (from data wrangling to analysis — the wrangling moves into the transformation layer). The stack is an enabler of the operating model, not a strategy in itself.

### 6.6 Decision Intelligence

**Decision intelligence** (DI) is the emerging discipline of making *decisions* — rather than data or models — the unit of analysis. Gartner has tracked it as an emerging category (publishing a Market Guide for Decision Intelligence Platforms and, most recently, a Magic Quadrant for Decision Intelligence Platforms — a signal it has moved from fringe to category). The core ideas:

- **Decision-centric framing**: instead of "build a churn model", ask "what is the retention decision, who makes it, with what information, and what is the decision's value?"
- **Decision modeling**: mapping decisions as assets — their inputs, rules, owners, and feedback — so they can be improved systematically rather than rediscovered each time.
- **Combining analytics, AI, and decision theory**: DI explicitly draws on decision theory and behavioral science, recognizing that analytics changes decisions only through human and organizational processes (the §3.5 problem, addressed at the decision level).

For the analytics manager, DI is less a new tool and more a **reframing** that the best analytics leaders already practice: the value chain of §3.1 is a decision chain, and managing "the decision" as the unit of value is the mature form of analytics management. Expect DI terminology (decision models, decision catalogs, decision registries) to keep migrating into analytics governance over the next few years.

### 6.7 Augmented Analytics

**Augmented analytics** is Gartner's term (coined in 2017) for the use of **machine learning and AI to assist with data preparation, insight generation, and insight explanation** inside analytics and BI platforms. Concretely:

- **Auto-insights**: the platform itself surfaces anomalies, correlations, and drivers ("deposits dropped 8% in the North region, driven by X") without the analyst scripting every view.
- **Natural-language interaction**: asking the data in plain language (the ancestor of today's GenAI chat-with-data) — Power BI Q&A, Tableau's ask features, ThoughtSpot's search, and the new LLM-based copilots.
- **Automated data preparation**: ML-assisted cleaning, joining, and feature suggestion.

Gartner's projections that augmented analytics would become a primary driver of analytics spending and that a majority of analytics tasks would be "augmented" proved directionally right — the capability is now standard in BI platforms and has been absorbed into the GenAI wave (§9.1). For management, augmented analytics matters in three ways: it *lowers the skill floor* for self-service (widening democratization), it *changes analyst work* (from building routine views to curating and verifying), and it *moves verification burden* (auto-generated insights still need human judgment — the "trust but verify" discipline of §4.7).


---

## 7. Analytics in Banking

Banks were among the first industrial-scale analytics adopters — risk management forced them to quantify early — and banking remains one of the most analytics-dense industries. This chapter covers how the general discipline lands in a bank, cross-referencing the dedicated banking and risk guides.

### 7.1 Bank Analytics Domains

Bank analytics clusters into three broad domains (with a fourth, finance/regulatory, running through all of them):

**Customer analytics** — the commercial engine:

- **Segmentation** — clustering customers into behavioral groups (by value, lifecycle, channel preference, risk profile) to shape propositions and targeting.
- **Churn / attrition analytics** — predicting customers likely to leave (deposits, cards, wealth) and the retention playbooks to keep them (the churn use case is the classic first-value banking analytics, and the worked example in Section 8 builds on it).
- **Next-best-action (NBA)** — recommending the best offer, message, or channel interaction for each customer at each touchpoint — the prescriptive rung of the ladder applied to distribution; a staple of retail-bank analytics, executed through CRM and channel systems.
- **Customer 360 / single customer view** — the data foundation underneath: a governed, joined view of the customer across products and channels (see [dbs_bank_guide.md](../banking/dbs_bank_guide.md) — DBS's "data-driven" transformation is built on customer-360 thinking — and [wealth_management_guide.md](../banking/wealth_management_guide.md) for the client-analytics angle in wealth).

**Risk analytics** — the protective and capital engine (covered in depth in [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md)):

- **Credit scoring** — application and behavioral scoring for lending decisions (the oldest and most regulated bank analytics).
- **Stress testing and capital planning** — macro/portfolio-level models used for ICAAP, CCAR-type exercises, and MAS stress testing — where model governance meets the P&L.
- **Market and liquidity risk analytics** — VaR, limits, scenario analysis; the quantitative infrastructure of the risk function.
- **Collections and recovery analytics** — prioritizing and optimizing debt collection.

**Operational analytics** — the efficiency and control engine:

- **Fraud detection** — real-time transaction monitoring models (the automation rung: models screen every transaction; see the banking guides' fraud material).
- **Reconciliation and ops analytics** — exception detection, STP-rate analysis, cost-to-serve analytics (see the payments and ops guides under [../banking/](../banking/), e.g. [payments_hub_guide.md](../banking/payments_hub_guide.md)).
- **Process and workforce analytics** — back-office optimization, branch/ATM network analytics, contact-center analytics.
- **AML analytics** — transaction monitoring, client screening, anomaly detection (regulatory-critical; see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md)).

A structural point about banks: the **risk/regulatory analytics** (models that drive credit, capital, or AML decisions) and the **commercial analytics** (marketing, customer) are governed differently and often live in different organizations — model risk governance for the former (Section 7.3), lighter governance for the latter. The analytics function spans both, which is precisely why banking analytics management is a *two-speed* discipline: speed and experimentation for customer analytics; rigor and evidence for regulated models.

### 7.2 Analytics in the Bank Organization

The bank's analytics organization typically involves four pieces (arrangement varies by bank):

1. **The CDO / data office** — owns the data platform, data governance (BCBS 239 risk-data requirements and MAS expectations are the regulatory drivers — see [data_governance_guide.md](../technology/data_governance_guide.md) and [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md)), and increasingly the analytics COE. The "analytics under the CDO" placement (Section 2.4) is the common bank pattern.
2. **Business-unit analytics teams** — embedded analysts in retail, wealth, corporate, markets, ops — the spokes.
3. **Central data science / AI teams** — scarce specialists for advanced modeling, often in a central "AI lab"/COE, serving the units (hub).
4. **Model risk management (MRM)** — the second line governing *consequential* models: validation, approval, monitoring. MRM is the defining feature of bank analytics governance (below).

The two-line structure is the key bank-specific design fact: **the model validators must be independent of the model builders** — the same analytics function cannot both build and approve its own consequential models.

### 7.3 Model Risk and SR 11-7

The regulatory cornerstone of bank analytics governance is **SR 11-7** — the U.S. Federal Reserve/OCC *"Supervisory Guidance on Model Risk Management"* (2011), which defines **model risk** as the potential for adverse consequences from decisions based on incorrect or misused model outputs, and requires banks to manage it through sound development, independent validation, and ongoing monitoring. Its principles have become the de facto global standard for model governance (MAS and other supervisors follow equivalent expectations; see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) for the full treatment). The operational translation for the analytics function:

- **Model inventory** — every consequential model documented and registered (purpose, data, method, owner, limitations).
- **Independent validation** — models validated by a function separate from development before use, and periodically after.
- **Ongoing monitoring** — performance, drift, and assumption checks on a schedule.
- **Governance forums** — model risk committees approving models and their use.
- **Documentation discipline** — the model lifecycle is an audit trail, not an afterthought.

The management takeaway: in banking, "analytics governance" (Section 4) and "model risk governance" overlap heavily — a bank's analytics roadmap must route consequential models through MRM as a first-class delivery stage, with validation time and evidence budgeted into the plan. Teams that discover MRM requirements after building the model pay double.

### 7.4 Bank Analytics Maturity: DBS and the Singapore Context

**DBS** is the region's most-cited case of analytics maturity — its decade-plus "data-driven bank" transformation (under CEO Piyush Gupta) is widely documented in industry sources: the customer-360 data foundation, embedded analytics across channels, and an explicitly AI-enabled operating model (DBS publicly describes itself in "AI-enabled bank" terms; see [dbs_bank_guide.md](../banking/dbs_bank_guide.md) for the bank-specific detail). The lessons banks typically extract from the DBS story are the standard maturity lessons: start from the customer decision, build the data foundation once, embed analytics in channels rather than in slide decks, and treat the transformation as cultural as much as technical.

**The Singapore context** adds the regulatory dimension: MAS (the Monetary Authority of Singapore) supervises bank data and analytics — its data-management expectations (and its principles on responsible AI use, e.g. the FEAT principles of Fairness, Ethics, Accountability, Transparency, published jointly with other Singapore regulators) shape how analytics is governed locally; see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) for the regulatory detail, and [standard_chartered_guide.md](../banking/standard_chartered_guide.md) for a second regional benchmark.

### 7.5 Banking Analytics Governance: Compliance and the Two Speeds

Pulling the chapter together, bank analytics governance runs at two speeds with one spine:

- **One spine**: data governance (definitions, quality, lineage — BCBS 239 disciplines) and model risk governance (SR 11-7 disciplines) underpin everything.
- **Speed one — commercial analytics** (customer/operations): governed for fairness, privacy, and conduct (PDPA, conduct risk), but fast — experimentation allowed, lighter validation.
- **Speed two — regulated analytics** (credit, capital, AML, fraud): full MRM lifecycle, independent validation, audit trails, regulatory reporting obligations.

The analytics leader in a bank must be fluent in both speeds — knowing which lane each piece of work runs in — and must keep the two from contaminating each other: a customer segmentation model is not a credit model, and must not be quietly used as one (the purpose-limitation risk of Section 4.6).

---

## 8. Worked Example: Building an Analytics Function in a Mid-Size Bank

This section walks through a realistic end-to-end case of standing up an analytics function — applying every chapter of this guide in sequence.

### 8.1 Scenario and Starting Point

**The bank**: a mid-size regional bank (say, ~200 branches, retail + SME + wealth, one country). Profitable, traditional, with decent core systems but **no analytics function to speak of**: each unit runs its own Excel reporting, a legacy warehouse feeds regulatory reporting, and there is no data science anywhere. Pressure is rising: a digital-only competitor is taking deposit share; the regulator is asking sharper questions; the board has read a report on "AI in banking" and wants an answer.

**The starting point**: the board appoints a **Chief Data Officer (CDO)** — the common entry pattern (Section 2.4, Section 7.2) — with a mandate to build data *and* analytics capability. Day one reality: the CDO has a mandate, a budget (small but real), two data engineers, zero analysts, a legacy warehouse, and 200 business stakeholders who all want everything.

### 8.2 Strategy: The Use Cases

The CDO starts from Section 1.6 discipline — strategy from business objectives, not from technology. Three candidate use-case families surface from business conversations:

1. **Deposit churn** (retail): the bank loses ~10%/yr of deposit customers; digital competitors are accelerating it. *Value: retention of deposits and relationship margin. Feasible: the data exists (products, transactions, service history).*
2. **Cross-sell / next-best-action** (retail + wealth): average product-holdings per customer is 1.6 vs 3+ for peers. *Value: share-of-wallet growth. Feasible but depends on the customer-360 build.*
3. **Early-warning credit risk** (SME): the bank's SME portfolio has rising early-stage delinquency; collections is reactive. *Value: loss reduction. Feasible: loan data exists; but this lane is regulated (SR 11-7-style expectations) — slower to production.*

The strategy: **"two quick wins, one regulated capability, all on one data foundation."** Year-one value cases: churn (flagship quick win), SME early-warning (the regulated lane, started early because its validation runway is long), and the customer-360 data foundation as the enabling investment (which also unlocks cross-sell in year two). This sequencing is deliberate: quick wins fund the narrative; the regulated lane builds the governance muscle; the foundation is the compounding asset.

### 8.3 Operating Model: Hub-and-Spoke

The CDO chooses **hub-and-spoke** (Sections 2.1-2.3):

- **The hub** (the new Analytics COE inside the CDO office): the data platform team, a small central data-science pod (the SME early-warning model), standards and governance, and the self-service enablement program.
- **The spokes**: one embedded analyst in Retail (churn and, later, NBA), one in Wealth (cross-sell), one in Risk (working with the credit team and, critically, *with the model-validation function* — independence preserved per Section 7.2).
- **Reporting lines**: spoke analysts sit in the business (day-to-day priorities) with dotted line to the hub (methods, quality, careers) — the matrix of Section 2.5.
- **Why not centralized or federated?** Centralized would reproduce the old "IT queue" the business distrusts; full federation would fragment the scarce talent and the standards the bank badly needs. Hub-and-spoke fits a mid-size bank with shared platforms and a CDO mandate.

### 8.4 Governance: Intake, Prioritization, ROI

The governance machinery (Section 4), stood up from month one:

- **Intake**: a single analytics intake channel with a lightweight request form (business question, decision, owner, value expectation). Triage rules: self-service vs standard vs substantial.
- **Prioritization forum**: monthly — the CDO chairs, business-unit heads and the spokes attend; candidates scored on value/effort/confidence (RICE-style, Section 4.2); the ranked backlog and quarterly roadmap ratified and published. The bank's leadership learns the discipline of *visible no's*.
- **Value measurement**: every value case opens with a baseline and a success metric; the CDO keeps a **value ledger** (Section 4.4): expected vs realized value, reviewed quarterly at the board. Churn's ledger entry: "baseline attrition 10.4%; target: hold attrition at or below 9.0% by Q3 via retention offers to model-flagged customers."
- **Standards**: the hub owns metric definitions and naming (with data governance — one "active customer" definition, one customer master); every dashboard must draw from the certified layer once it exists.

### 8.5 The Team: Roles and Hiring

Year-one build (Section 5):

- **Hire first**: one **senior data scientist** (leads the churn model and mentors), one **analytics engineer** (builds the foundation models/transforms on the warehouse), and the three **embedded analysts** (retail, wealth, risk) — hired for the three-legged mix (Section 5.3): each must demonstrate business framing and communication, not just SQL.
- **Grow, don't buy, the translators**: the CDO recruits the Retail and Wealth spokes partly from *business* people with analytical aptitude (the McKinsey "grow translators" lesson, Section 5.1) — one of them is a former retail product manager; this pays off immediately in adoption.
- **Upskill**: a quarterly "analytics literacy" bootcamp for business teams (what a model can/can't do, how to read a dashboard, how to frame a request) — the democratization engine (Section 5.4) that keeps the hub from drowning in small requests.

### 8.6 The 12-Month Roadmap

| Quarter | Foundations | First products | Adoption | Maturity |
|---------|-------------|----------------|----------|----------|
| **Q1** | Data audit; warehouse refresh; intake + forum live; hire core team | Baseline churn cohort analysis (diagnostic: who is leaving and why) | Executive churn deep-dive with retail leadership; retention hypotheses agreed | Hub-and-spoke operating, value ledger open |
| **Q2** | Customer-360 build starts (with data governance — master data, definitions) | Churn propensity model v1 (predictive) + retention offer playbook | Pilot retention campaign on 2 branches/segments with control group | First ROI evidence; business requests now arrive framed |
| **Q3** | Semantic layer/certified datasets for retail | Churn model v2 in production, scored monthly; SME early-warning model development starts (with validation team engaged from day one) | Retention playbook rolled out to 20 branches; self-service dashboards live for branch managers | Attrition trending down; spokes fully embedded; NBA pilot scoped |
| **Q4** | SME data mart completed | SME early-warning model enters independent validation; NBA (cross-sell) model v1 built | Validation findings drive model v2; NBA pilot with wealth | Year-one value ledger reviewed: churn retention value + early-warning losses avoided; year-two roadmap (cross-sell scale-up, AML anomaly detection) ratified |

The roadmap is deliberately **foundations → products → adoption → maturity**: each quarter's deliverables are sequenced so that data, models, governance, and business adoption land in the right order — the lifecycle discipline of Section 3 applied to the function itself.

### 8.7 A Week in the Life of the Analytics Leader

A representative week for the CDO/Head of Analytics in month eight:

- **Monday — intake review**: triages the week's requests with the hub lead; three self-service redirects (users pointed to the new dashboards), two standard analyses scheduled, one substantial request (treasury wants an FX-customer analytics pilot) — deferred to the monthly forum with a value hypothesis request.
- **Tuesday — delivery reviews**: the retail analyst presents churn model v2 results; the CDO asks the standing question (*"what decision does this change, and who is making it?"*), pins the adoption plan (branch rollout, training, control group) before approving.
- **Wednesday — stakeholders**: lunch with the retail head (the churn program sponsor) — reviewing the pilot's leading indicators (offer uptake, model hit rate) and removing a data-access blocker; then 30 minutes with the model-validation team on the SME model's validation timeline.
- **Thursday — the team**: 1:1s with the spokes (career, skills gaps), a hiring screen for a second data scientist, and an hour protecting the team from scope creep (three "quick asks" redirected to the backlog).
- **Friday — portfolio and the board**: prepares the quarterly value-ledger review — churn pilot results, early-warning progress, adoption metrics — and rehearses the two numbers that will matter: realized value to date, and the year-two ask.

The week shows the actual shape of the job: **governance cadence (intake/forum), delivery management (reviews), stakeholder management (sponsors, validators), team leadership (1:1s, hiring), and value communication (the board)** — the four domains of Section 1.1 in weekly rotation.

---

## 9. The Future: Analytics in 2026 and Beyond

The analytics management landscape is shifting on four fronts, all visible in 2026.

### 9.1 AI-Powered Analytics: GenAI and Natural-Language BI

The GenAI wave has landed inside analytics: **natural-language BI** (asking dashboards and data in plain language), **AI-generated insights** (auto-summaries of performance, automated anomaly explanations), **copilots for analysts** (code and query generation, documentation, test writing), and **AI-assisted data preparation** (the augmented-analytics lineage of Section 6.7, now LLM-powered). The management implications are significant but manageable:

- The **skill floor drops** — natural-language interfaces widen self-service (democratization accelerates), so the certified semantic layer becomes *more* important, not less: an LLM answering questions against ungoverned data is a liability engine.
- **Verification burden rises** — generated insights and code must be checked; the analyst's job shifts from production to curation and validation.
- **The analytics team's role re-centers** on judgment, framing, and adoption — the human end of the value chain of Section 3.1 gets more valuable, not less. (See the AI/LLM guides under [../technology/ai_llm/](../technology/ai_llm/) for the technology and governance detail, including [implementing-responsible-ai.md](../technology/ai_llm/implementing-responsible-ai.md).)

### 9.2 Decision Intelligence Goes Mainstream

As covered in Section 6.6, decision intelligence is moving from Gartner's emerging-category tracking into concrete platforms (the 2026 Magic Quadrant for Decision Intelligence Platforms marks the category's arrival). Expect analytics governance to absorb DI vocabulary — **decision registries, decision models, decision owners** — and the value-chain discipline of Section 3.1 to become explicit "decision engineering". The management opportunity: DI gives the analytics leader a language to connect analytics spend to decision outcomes — the strongest possible answer to "what are we getting for this?".

### 9.3 Data Products and Data Mesh

The **data product** movement — treating curated data as products with owners, SLAs, and users — and the **data mesh** organizational pattern (domain teams owning their data products, governed by federated standards) continue to reshape the data layer (see the data governance and data engineering guides: [data_governance_guide.md](../technology/data_governance_guide.md)). For analytics management: the certified data layer of Section 6.2 and the data products of Section 9.3 are converging — the analytics function's self-service foundation *is* a portfolio of data products. The management shift: analytics leaders increasingly co-own data product portfolios with the data organization, and the analytics backlog and the data product roadmap must be planned as one system.

### 9.4 Analytics Everywhere: Democratization at Scale

The long democratization trend (Sections 5.4, 6.2) reaches its logical end-state: analytics embedded in every workflow — dashboards inside operational apps, alerts in chat channels, models inside core processes (the "embedded analytics" row of Section 6.1 becomes the default). "Analytics everywhere" redefines the analytics function's job from *producing* insight to *governing and enabling* insight production across the enterprise: standards, platforms, literacy, and the value ledger — while the production of routine insight migrates to the business itself and to AI.

### 9.5 Trends Summary

| Trend | What changes | The management action |
|-------|--------------|-----------------------|
| GenAI / natural-language BI | Self-service widens; verification burden grows | Invest in the semantic layer; re-skill analysts to curation |
| Decision intelligence | Decisions become the managed unit | Adopt decision-level framing and registries in governance |
| Data products / mesh | Data layer and analytics layer merge | Co-manage the data-product and analytics portfolios |
| Analytics everywhere | Analytics embedded in all workflows | Govern through platforms and standards, not queues |

The through-line of all four: **analytics management is moving from managing a team that produces insights to governing an enterprise that produces decisions.** The discipline of this guide — strategy, organization, lifecycle, governance, team, products, culture — remains the operating system; only the execution surface changes.

---

## 10. Glossary

- **Analytics** — the analysis layer that turns data into insight: descriptive ("what happened"), diagnostic ("why"), predictive ("what will happen"), prescriptive ("what to do").
- **Analytics engineer** — builds the data preparation, transformation, and tooling that analysts and models consume (pipelines, dbt models, semantic layers).
- **Analytics management** — the discipline of organizing, running, and governing the analytics function: people, process, technology, and value.
- **Analytics maturity** — how advanced and value-producing an organization's analytics capability is (data/people/process/culture dimensions; capability ladders from reporting to embedded AI).
- **Analytics product** — a managed, user-facing analytics deliverable (dashboard, report, model, self-service dataset) run with product discipline: users, adoption, iteration.
- **Analytics product manager** — the owner accountable for an analytics product's fitness and adoption (an emerging role in analytics-native organizations).
- **Analytics translator** — the business-fluent bridge between business problems and analytics solutions (McKinsey-championed role); frames problems, scopes work, converts findings into decisions.
- **Augmented analytics** — Gartner's term (2017) for AI/ML-assisted data preparation, insight generation, and explanation inside analytics platforms.
- **Backlog** — the ranked, unstarted analytics work awaiting delivery.
- **BI (Business Intelligence)** — the reporting layer: dashboards, scorecards, and reports describing the state of the business.
- **CDO (Chief Data Officer)** — the executive accountable for data (and, commonly, analytics) as an organizational asset.
- **Center of Excellence (COE)** — the central analytics vehicle: standards, platforms, talent, governance (also "analytics COE" / "competency center").
- **Centralized (operating model)** — one central analytics team serving the enterprise.
- **Churn** — customer attrition; "churn analytics" predicts who is likely to leave and why.
- **Customer analytics** — analytics over customer behavior: segmentation, churn, next-best-action, customer 360.
- **Dashboard** — a persistent visual view of metrics for monitoring and decision support.
- **Data-driven** — the organizational norm of basing decisions on evidence rather than intuition alone; "data-driven decision-making".
- **Data mesh** — an organizational pattern where domain teams own their data products under federated governance.
- **Data product** — a curated, owned, served data asset with users and SLAs.
- **Data scientist** — builds predictive/prescriptive models and experiments.
- **Data analyst** — performs descriptive/diagnostic analysis and reporting.
- **Decision intelligence** — the emerging discipline of making decisions (not data) the unit of analysis; decision modeling, analytics, and AI combined (Gartner-tracked category).
- **Demand management** — the structured intake, triage, and routing of analytics requests.
- **Descriptive analytics** — "what happened": reporting and visualization of historical data.
- **Diagnostic analytics** — "why did it happen": drill-down, segmentation, root-cause analysis.
- **Federated (operating model)** — analytics embedded in and owned by business units.
- **Hub-and-spoke (operating model)** — central hub (standards, platforms, specialists) + embedded spokes (business-facing delivery); the common hybrid.
- **KPI (Key Performance Indicator)** — a metric tied to a strategic objective; leading (predictive of outcomes) vs lagging (outcome itself).
- **ML engineer** — deploys, serves, and monitors machine-learning models in production.
- **Model risk** — adverse consequences from decisions based on incorrect or misused model outputs (SR 11-7 definition).
- **Natural-language BI** — querying data and dashboards in plain language (now GenAI-powered).
- **Next-best-action (NBA)** — the recommended best offer/message/channel per customer at each touchpoint.
- **Operational analytics** — analytics over bank operations: fraud detection, reconciliation, process optimization.
- **Predictive analytics** — "what will happen": forecasting and propensity modeling.
- **Prescriptive analytics** — "what to do": recommendations, optimization, automated decisions.
- **Prioritization** — ranking analytics work by value vs effort (and confidence, urgency); e.g., RICE-style scoring.
- **Risk analytics** — analytics over credit, market, liquidity, and operational risk (credit scoring, stress testing).
- **Roadmap** — the planned analytics delivery over coming quarters, tied to value cases.
- **ROI (Return on Investment)** — measured value returned on analytics investment, with attribution evidence.
- **Segmentation** — dividing customers into behavioral groups for targeting and propositions.
- **Self-service** — business users answering their own questions from governed data assets (governed self-service: certified data + guardrails).
- **SR 11-7** — U.S. Federal Reserve/OCC supervisory guidance on model risk management (2011); the de facto global standard for model governance.
- **Translator** — see *Analytics translator*.

---

## 11. References

**Frameworks and concepts**

- Davenport, T. — *Competing on Analytics* (Harvard Business Review, January 2006); Davenport, Harris & Morison — *Analytics at Work* (2010, DELTA model: Data, Enterprise, Leadership, Targets, Analysts).
- McKinsey / QuantumBlack — *Analytics translator: the new must-have role* (analytics translators; US demand forecast of 2-4M by 2026).
- Gartner — augmented analytics (term coined 2017); Market Guide for Decision Intelligence Platforms (2024) and Magic Quadrant for Decision Intelligence Platforms (2026).
- Pfeffer, J. & Sutton, R. — *Evidence-Based Management* (HBR, 2006) — the intellectual roots of data-driven decision-making culture.
- Intercom — the RICE prioritization framework (Reach, Impact, Confidence, Effort) — product-management origin, adapted to analytics demand.
- Analytics maturity family: descriptive → diagnostic → predictive → prescriptive ladder (Gartner-associated framing); DELTA model (Davenport); "Analytics 1.0-3.0" informal historical shorthand.

**Regulatory and banking**

- SR 11-7 — *Supervisory Guidance on Model Risk Management* (Federal Reserve / OCC, 2011).
- MAS (Monetary Authority of Singapore) — data management and responsible-AI expectations (including the FEAT principles: Fairness, Ethics, Accountability, Transparency, published with other Singapore regulators).
- BCBS 239 — risk data aggregation and risk reporting principles (see [data_governance_guide.md](../technology/data_governance_guide.md)).

**Related guides in this repository**

- Management: [strategic_management_guide.md](strategic_management_guide.md) (strategy layer), [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) (team leadership), [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) (managing up/down/sideways), [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) (communication), [vendor_management_guide.md](vendor_management_guide.md) (tool vendor selection).
- Technology: [data_governance_guide.md](../technology/data_governance_guide.md) (the data side), [advanced_analytics_solutions_guide.md](../technology/advanced_analytics_solutions_guide.md) (analytics products/vendors), [closed_loop_data_engineering_guide.md](../technology/ai_llm/closed_loop_data_engineering_guide.md) (data engineering loop), [implementing-responsible-ai.md](../technology/ai_llm/implementing-responsible-ai.md) (responsible AI), plus the AI/LLM guides under [../technology/ai_llm/](../technology/ai_llm/).
- Banking: [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) (risk/compliance analytics, SR 11-7, MAS), [dbs_bank_guide.md](../banking/dbs_bank_guide.md) (DBS data-driven transformation), [standard_chartered_guide.md](../banking/standard_chartered_guide.md), [wealth_management_guide.md](../banking/wealth_management_guide.md) (client analytics), and the banking operations guides under [../banking/](../banking/).

---

*End of guide. Verification notes: frameworks marked "verify" in the drafting process were checked against public sources — the descriptive→prescriptive ladder (Gartner-associated industry framing), Davenport's Competing on Analytics (HBR, Jan 2006), the McKinsey analytics translator role, Gartner's augmented analytics (2017) and decision intelligence (Market Guide 2024; MQ 2026), and the centralized/federated/hub-and-spoke operating models (common practitioner usage). Items flagged in-text as "informal", "directional", or "varies by organization" (e.g., Analytics 1.0-3.0 shorthand, McKinsey's 2-4M translator demand figure, analytics maturity model stage names, CDO placement prevalence) should be read as such.*
