# Business Intelligence Transformation — A Negotiation About What a Number Means

*The deep-research guide to the discipline that a "BI migration" really is: not the movement of dashboards from one platform to another, but the settling — or the deliberate non-settling — of organisational definitions. The guide covers the legacy estate and how it grows, the semantic layer and the definition-versus-calculation split, the definitional conflicts a bank actually has and who adjudicates them, self-service and its second estate, the reconciliation-bearing migration path, the destination mapped rather than restated, the organisation and the skills, the governance and lineage that make a transformation reversible, the business case and which parts of it are usually unvalidated, the regulated-institution angle from BCBS 239 and IAS 8, six failure modes, and a fully labelled Cymbal Bank worked example. Where the evidence ends, the guide says so.*

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** github.com/jackliusr/research · **Category:** Technology Series · **Date:** October 2026

**Verification posture for this guide.** Every claim that carries a number, a normative expectation, or a mechanism is labelled with its provenance in this guide's bracket convention: **[BCBS239]** for the Basel Committee's *Principles for effective risk data aggregation and risk reporting* (January 2013), **[IAS8]** for IAS 8 *Accounting Policies, Changes in Accounting Estimates and Errors*, **[STRANGLER]** for the strangler-fig pattern of incremental legacy replacement, **[PARALLEL]** for the expand/migrate/contract (parallel-change) discipline, **[DBT]** for dbt's semantic-layer and MetricFlow documentation, **[LOOKML]** for Looker's LookML documentation, **[WEB]** for a specifically named web page, and **[REPO]** for a cross-reference to another guide in this repository. This guide states explicitly and without qualification: **no usage, cost, failure-rate, ROI or savings statistic appears anywhere in it unless it is attributed to a dated source; and where a figure is used it is a worked illustration that is labelled as an illustration in the same paragraph in which it appears.** The guide repeatedly makes points that in weaker writing would be percentages — "most reports are unused," "the estate is largely dead weight" — and it makes them instead as *observations about how estates grow and how usage is (or is not) established*, because no dated primary source supports a percentage and this guide refuses to invent one. The worked example in §14 is fiction, labelled as such in the section itself and in the claims audit in §15.

### Table of Contents

1. [The Overview, the Thesis and the Decoder](#1-the-overview-the-thesis-and-the-decoder)
2. [What a BI Transformation Actually Is](#2-what-a-bi-transformation-actually-is)
3. [The Legacy Estate](#3-the-legacy-estate)
4. [The Semantic Layer and the Definitional Problem](#4-the-semantic-layer-and-the-definitional-problem)
5. [Self-Service Enablement](#5-self-service-enablement)
6. [The Migration Path](#6-the-migration-path)
7. [Reconciliation](#7-reconciliation)
8. [The Destination, Mapped Not Restated](#8-the-destination-mapped-not-restated)
9. [The Organisation and the Skills](#9-the-organisation-and-the-skills)
10. [Governance, the Catalogue and Lineage](#10-governance-the-catalogue-and-lineage)
11. [The Business Case](#11-the-business-case)
12. [The Regulated-Institution Angle](#12-the-regulated-institution-angle)
13. [The Failure Modes](#13-the-failure-modes)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
15. [The Anti-Patterns and the Claims Audit](#15-the-anti-patterns-and-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Thesis and the Decoder

A business-intelligence transformation begins as a technology programme and turns, without warning, into an argument about language. The trigger is almost always technological — a platform reaching end of support, an acquisition that doubles the estate overnight, a regulator's observation on a reconciliation, a warehouse that cannot answer a question the business has started asking daily, or a licence renewal that makes the incumbent vendor's price visible for the first time. The deliverable that the sponsor believes is being bought is a new platform. The deliverable that actually has to be produced is a set of decisions about what the organisation's words mean when a number is printed next to them.

That is the whole thesis, and it is worth stating in one line before anything else: **a BI transformation is a negotiation about what a number means, and the technology migration is the occasion for the negotiation rather than its subject.** Read the rest of this guide as an unpacking of that sentence. The platform is real and the engineering is real, but the part of the work that decides whether the transformation succeeded is definitional, and the part that decides whether it can be *un-done* is the reconciliation and lineage discipline that a migration either builds or does not.

This guide is written for the solution architect, the data-platform lead, the analytics-engineering lead, the data-governance lead, and the business-side definition owner who has to sign that a retired report was not needed. It assumes the reader has seen at least one platform migration and is suspicious of the word "complete."

### 1.1 The twelve-term decoder

The guide uses these terms in a specific, load-bearing way. Where a word is contested in the industry, the definition here is the one this guide will hold to throughout, and the provenance column names the source the definition leans on.

| Term | Definition as used in this guide | Provenance |
|---|---|---|
| **Business intelligence** | The reporting-and-analysis layer a business uses to see what it did: the reports, the dashboards, the ad-hoc queries, and the governance around them. It answers "what happened," as distinct from the predictive and prescriptive domains owned elsewhere in the series. | [REPO] `advanced_analytics_solutions_guide.md` §1; [WEB] `advanced_analytics_solutions_guide.md` §1.1 |
| **The data warehouse** | A governed, modelled, historically-integrated store of subject-oriented data built to answer the enterprise's recurring questions. It is a *contract as much as a database*: the same revenue figure returned to every consumer. | [REPO] `data/enterprise_data_platforms_guide.md` |
| **The lakehouse** | Object storage holding raw and curated data at low cost, plus warehouse-style management (ACID transactions, schema enforcement, SQL engines) over it — the lake's economics with the warehouse's discipline. | [REPO] `cloud_object_storage_lakehouse_guide.md`, `data/on_prem_lakehouse_guide.md` |
| **The semantic layer** | The repository of *logic* between the physical tables and the consumer: the declared metrics, dimensions, hierarchies and joins that compile into SQL so that one definition produces one number everywhere. | [DBT] MetricFlow semantic models; [LOOKML] LookML models and views |
| **The metric** | A named, governed quantity — "net revenue," "active customer count" — with an owner, a definition, and a calculation that implements the definition. A metric is an artefact with a change history, not a formula in a file. | [BCBS239] ¶37 (the "dictionary" of concepts) |
| **The definition vs. the calculation** | The *definition* is an organisational fact ("who is a customer"); the *calculation* is the arithmetic that implements it (the SQL and the filters). The two change for different reasons and are owned by different people. | This guide, §4.3 |
| **The report estate** | The full population of produced artefacts — reports, extracts, subscriptions, scheduled deliveries, dashboards — that the organisation depends on, including the ones nobody can explain. | [REPO] `data/data_governance_framework.md` §10 |
| **The dashboard** | A single consumable artefact, usually built in one tool over one semantic source, presenting several metrics at once. A dashboard is an *output*; the metric is the *thing governed*. | This guide, §5 |
| **Self-service** | The capability by which business users build their own views over governed data without a request to a central team. Its cost is the governance tension it introduces. | [REPO] `../management/analytics_management_guide.md` |
| **The catalogue** | The indexed inventory of data artefacts with their owners, definitions, classifications and relationships — the enterprise's "what exists and who owns it." | [REPO] `data/data_governance_framework.md` §10 |
| **Lineage** | The traced path from a reported number back through the transformations that produced it to the source record — the evidence that a number is what it claims to be. | [REPO] `data/data_lineage_tools.md` §1–§2; [BCBS239] P3, P7 |
| **The governed metric** | A metric whose definition has a named owner, a recorded version, a stated reconciliation, and a change log — the unit that makes a transformation reversible because its meaning can be recovered. | [BCBS239] ¶34, ¶37; this guide, §4.1 |

### 1.2 The boundary, declared by name

A guide that overlaps its neighbours is a guide that will contradict them. This guide declares its boundary explicitly:

- **`advanced_analytics_solutions_guide.md`** owns the analytics *stack*, the maturity model, platform comparison, real-time engines, and the banking analytics patterns. This guide does not restate them; §8 maps the transformation onto them.
- **`../management/analytics_management_guide.md`** owns the operating model, the lifecycle, team topologies and hiring. This guide touches organisation and skills only where the transformation *changes* them (§9).
- **`data/enterprise_data_platforms_guide.md`** owns the enterprise data-platform architecture as a category. This guide assumes the destination and concerns itself with the *move*.
- **The lakehouse guides** — `cloud_object_storage_lakehouse_guide.md`, `data/on_prem_lakehouse_guide.md`, `data/on_prem_lakehouse_analysis.md`, and the object-storage comparison — own the lakehouse as a destination. This guide treats the lakehouse as the *landing zone* of a transformation, not a subject in itself.

Everything else in this guide — the definitional problem, the reconciliation discipline, the reversibility argument, the business-case honesty, the failure modes — is what none of those neighbours owns.

---

## 2. What a BI Transformation Actually Is

A BI transformation is a programme of work that changes the platform on which the organisation's numbers are produced *and* the definitions by which those numbers are understood. If it does only the first, it is a migration; if it tries to do only the second, it has no vehicle. The transformation is the deliberate pairing of the two.

### 2.1 The triggers, as drivers rather than facts

Triggers are stated here as *drivers* — the reasons an organisation starts — not as facts about any particular organisation. A transformation is launched because one or more of these is true:

- **A platform reaching end of support.** The incumbent vendor has announced a date after which the current version is unsupported. The date is a fact; what the organisation *does* about it is a driver, and it is often nothing until the date is close.
- **An acquisition.** Two estates, two definitions of "customer," two revenue-recognition conventions, and a mandate to report one number. The acquisition is the trigger; the definitional collision is the work.
- **A regulator's observation.** A supervisory finding — on a reconciliation, on lineage, on the explainability of a reported figure — makes the estate's weakness visible to the board. The observation is the driver; the migration is the remedy the organisation chooses.
- **A warehouse that cannot answer a new question.** A new product, a new risk cut, or a new reporting demand arrives that the current model was never built for. The gap is the driver; the rebuild is the response.
- **A licence renewal.** The renewal quote makes the incumbent's cost visible in one number at one moment, and the organisation compares it — usually for the first time — against an alternative.

The honest treatment of these is to note that none of them *determines* the scope. The driver sets the deadline; the scope is set by what the organisation is willing to admit about its definitions.

### 2.2 The scope, as five fronts

A BI transformation runs on five fronts, and a programme that manages fewer than five will fail on one of the fronts it ignored:

1. **Platform** — the compute, storage, query engine, and orchestration that will produce the numbers. (Owned architecturally elsewhere; the *move* is this guide's concern.)
2. **Reports** — the estate of produced artefacts: what is migrated, what is rebuilt, what is retired, and on whose evidence.
3. **Definitions** — the negotiated meanings: who is a customer, what counts as revenue, when a customer is "active," what a "transaction" is. This is the front this guide is mostly about.
4. **Skills** — the semantic modelling, metric stewardship and reconciliation analysis the new estate requires and the old one did not.
5. **Governance** — the ownership, glossary, lineage and change log that make the transformation auditable and reversible.

These five are not sequential; they run in parallel and they *feed each other*. A definitional decision (§4) determines what the report inventory (§6) must migrate; a skills gap (§9) determines whether the definitional board (§9) can function; a governance gap (§10) determines whether the migration can be rolled back.

### 2.3 A programme, not a project

The framing that matters most is that a BI transformation is **a programme and not a project**. A project has a start, an end, and a definition of done. A transformation has a start, a long middle, and a steady state — and the parts of it that are irreversible sit inside a governance structure that must outlive the platform decision. **[STRANGLER]** — the strangler-fig pattern, in which a legacy system is replaced incrementally by routing functionality to a new service behind a façade, until the old system can be switched off — is the closest engineering analogue precisely because it, too, treats the migration as a *long coexistence* rather than a cutover event. The strangler pattern's discipline is that the old and new run together, that the façade routes by function rather than in one switch, and that the legacy is decommissioned only when nothing routes to it any more.

The three properties that make a transformation a programme:

- **It is multi-year.** Definitional negotiation in an organisation of any size does not complete in a quarter; the platform migration will finish before the definitions do, and the programme must be structured so that the platform finishing does not declare victory.
- **It is governance-bearing.** Every artefact it produces — a definition, a reconciliation, a retirement approval — is an audit record. The governance does not end with the platform.
- **It is reversible only in parts.** A schema decision can be reversed; a retired report that a regulator later asks for cannot be un-retired, only rebuilt. The programme must know, at each step, which parts are reversible and price the irreversible ones accordingly. §10 makes the reversibility argument concrete.

---
## 3. The Legacy Estate

The legacy BI estate is not a design. It is a sediment. It was laid down one request at a time, and understanding how it was laid down is the precondition for migrating it.

### 3.1 The warehouse as accreted layers

A data warehouse that has been in production for a decade is a set of geological strata. Each generation of modellers added a layer on top of the last rather than replacing it: the original star schemas from the first build, the conformed dimensions added when the second business line joined, the departmental marts that appeared when the central team could not keep up with demand, the staging tables that some long-departed engineer built to work around a source-system quirk and never removed, and the one "temporary" table that has been in production since it was created.

Each layer made sense when it was laid down. Layers accumulate because **adding is cheap and removing is expensive**. Adding a table costs one engineer's afternoon; removing one requires evidence that nothing depends on it — and establishing that evidence is exactly the work the organisation has never done. The estate therefore grows monotonically and shrinks almost never. This is not a failure of any individual; it is the rational response to an incentive structure in which the cost of addition is borne by one team and the cost of removal is borne by nobody until an audit.

### 3.2 Report sprawl and how an estate grows

Reports multiply by a mechanism that is easy to state and hard to stop. Trace it:

1. **Each request adds a report.** A business user needs a number cut a particular way. The fastest way to satisfy the request is a new report, not a change to an existing one — because changing an existing report risks breaking its other consumers, and the request is urgent today.
2. **Each new report becomes a subscription.** Once it exists, someone schedules it, someone receives it, and someone's routine now includes it. The artefact acquires dependents.
3. **Each subscription creates a constituency.** The report now has people who would notice if it disappeared, even if none of them reads it closely. The constituency is the reason deletion requires evidence.
4. **Deletion requires evidence, and nobody has it.** To retire a report safely, someone must be able to say "no one uses this and no one needs it." The usage record that would prove the first half is usually absent; the named owner who would sign the second half is usually unwilling to sign without the first. So the report stays.

The estate therefore grows faster than the business it describes, and the growth is not proportional to value. This is what is meant when practitioners say an estate is "largely dead weight" — **but the honest way to state it is as an observation about the mechanism above, not as a percentage.** No dated primary source in this guide's source pack measures the proportion of unused reports in a bank's estate, and this guide will not manufacture one. What can be said without a source is structural: the mechanism described produces growth without a corresponding removal mechanism, so the estate must accumulate artefacts whose usage is unestablished. "Usage is unestablished" and "usage is zero" are different claims, and only the first is supportable.

### 3.3 The reports nobody can explain

Every mature estate contains reports whose purpose has been lost. They are identifiable by a pattern:

- No one currently in the organisation built it.
- Its name refers to a project, a product, or an acronym that no longer exists.
- It is scheduled; the schedule fires; the output is delivered; nobody routes it anywhere useful.
- Asked what it is for, the report's nominal owner says "compliance" or "it's always been there."
- Attempts to trace its logic dead-end in a staging table or a stored procedure whose author has left.

The reason these reports are dangerous is not that they consume resources — that would be a small, measurable cost — it is that **they cannot be tested.** A migrated number can only be validated against the definition it implements, and if nobody can state the definition, the migration of that report cannot be verified; it can only be *reproduced*. §6 returns to this: the hardest migration item is usually the report nobody can explain, because reproduction is the only available acceptance criterion. A reproduced-but-unexplained report is a liability carried forward in new clothing.

### 3.4 The shadow spreadsheet estate

The most consequential part of the legacy estate is usually invisible in the warehouse: the spreadsheets. Somewhere between the warehouse output and the decision, a spreadsheet has been interposed — a manual adjustment, a mapping table, a set of overrides, a "final numbers" workbook assembled by an analyst who understands the business in a way the model never did.

These spreadsheets hold **the real business logic.** When the transformation seeks to move "the calculation" to a governed semantic layer, it discovers that the calculation it must move is not the one in the warehouse SQL but the one in a workbook on someone's laptop, and that this workbook encodes definitional decisions the organisation never wrote down. The shadow estate is where a transformation's definitional problem comes home to roost: you cannot migrate a definition you have never articulated, and the spreadsheet is the artefact that proves the definition was never articulated.

The shadow estate also has a governance dimension that §10 develops: these artefacts are outside the catalogue, outside lineage, and outside the change log, which means that anything depending on them is not reversible in the way the rest of the transformation aims to be. A number that a spreadsheet adjusts is a number whose lineage has a person-shaped hole in it.

### 3.5 The estate is the starting condition, not a defect

It is worth closing the section with the framing that keeps a transformation honest. The legacy estate is not evidence that the organisation was incompetent; it is the predictable result of a decade of decisions that each made sense at the time. Each added table was a request served; each added report was a question answered; each spreadsheet was a gap bridged by someone who cared enough to bridge it. The estate's problems are the *residue of previous service*, and the transformation is not a judgement on the past — it is the moment the organisation pays to reconcile a decade of accreted answers with a set of definitions it never wrote down.

The practical consequence is that a transformation should be *conservative about what it calls waste*. An artefact that looks redundant from the platform's vantage point may be load-bearing for a reason that lives in a business head, not in a table. §6.2's evidence standard exists because the estate's value is not visible from the outside, and the transformation that assumes it can see the value will retire something it has to rebuild.

---

## 4. The Semantic Layer and the Definitional Problem

This is the centre of the guide. Everything before it establishes the estate and the triggers; everything after it is the machinery of migration and governance. The claim of this section is that a BI transformation succeeds or fails on whether it *settles the definitions*, and that most transformations migrate the technology without settling them.

### 4.1 What a governed metric is

A metric — "net revenue," "active customer count," "average balance" — is treated in most estates as a formula embedded somewhere in a report definition. A **governed metric** is the same quantity treated as an artefact with five properties:

1. **A named owner.** A person, not a team, accountable for the metric's definition. Not the person who wrote the SQL — the person whose business function would be harmed if the number were wrong.
2. **A stated definition.** A sentence, in business language, that states what the metric counts and excludes. "Net revenue" defined as *X minus Y, over period Z, for customer class W.*
3. **A version.** The definition as of a date. Definitions change; a governed metric records its changes so that a historical number can be read against the definition that was in force when it was produced.
4. **A stated reconciliation.** How this metric relates to the neighbouring metrics it could be confused with — an explicit statement of why "net revenue" in Finance differs from "net revenue" in the product report, and by how much, and why the difference is expected.
5. **A change log.** The dated record of definition changes, the approval, and the party who approved. §10 and §12 make the change log the artefact that makes the transformation auditable.

The governed metric is the unit of reversibility. If the transformation migrates it correctly, the metric can be recovered from history even after the platform is gone. If it migrates the SQL without the definition, the number survives and its meaning does not.

**Why the change log is the property everyone drops first.** Of the five properties, the change log is the one organisations most readily omit, and its omission is the most damaging, because a metric's *history* is what makes a current number interpretable. A number without a change log cannot be compared across time: if "active customer" meant one thing last year and another this year, a year-on-year movement conflates a business change with a definitional one, and the organisation cannot tell which it is looking at. The change log is what separates a movement that means something happened in the business from a movement that means somebody changed a definition. **[BCBS239] ¶37**'s "dictionary" is the definitions; the change log is the dictionary's history, and without it the dictionary describes only the present.

**The owner is the property that makes the rest enforceable.** The five properties are not independent. The version, the reconciliation and the change log all depend on there being an owner who is accountable for them, because a definition with no owner has no one to version it, reconcile it, or log its changes. This is why §9 treats the business-side definition owner as a role the transformation must *create*, and why §4.5 treats the definition authority as the body that staffs it. A transformation that builds a semantic layer with precise definitions but no owners has built a library with books and no librarian: the definitions will drift, and no one will be accountable when they do.

### 4.2 What a semantic layer actually does

A semantic layer is the repository of declared business logic that sits between the physical tables and the consumer, and whose job is to make one definition produce one number everywhere it is requested. This guide defines the two dominant vocabularies as they actually appear in their documentation, without vendor-pitching either.

**[DBT] — dbt's semantic layer and MetricFlow.** dbt's semantic layer is built from *semantic models*, which "define the structure that MetricFlow uses to build the semantic graph" and which — per dbt's own documentation — map to a dbt model and declare **entities** (the keys and relationships that let measures be joined across models), **dimensions** (the attributes by which a measure can be sliced — including time dimensions with a granularity), and **measures** (the aggregatable expressions). **Metrics** are then declared on top of those measures: a *simple* metric aggregates a single measure, and a *derived* metric combines other metrics. The purpose, in dbt's phrasing, is that MetricFlow "compiles these metric definitions into clear, reusable SQL, ensuring consistent and accurate results when analyzing data by relevant attributes (dimensions)." The essential property is *compilation*: the metric is written once as a declaration, and each consumer's query is generated from it, so that no two consumers write the aggregation by hand. ([WEB] `docs.getdbt.com/reference/semantic-model-properties`; `github.com/dbt-labs/metricflow`, retrieved October 2026.)

**[LOOKML] — Looker's modelling language.** LookML is Looker's language for describing data to the tool. Its unit of physical modelling is the *view* — a declaration, over a table or a derived table, of *fields*, where *dimensions* describe attributes (a category, a date, a geography) and *measures* describe aggregations (a sum, a count, an average). A *model* file declares the connections and the *Explores* — "an existing view added to Looker's menu of Explores" — that business users query through the tool's UI. Looker generates the SQL from the LookML rather than accepting hand-written SQL from the consumer, which is the same compilation guarantee by a different route. ([WEB] `docs.cloud.google.com/looker/docs/looker-core-qs-model-data`, page last updated 9 October 2026; `docs.cloud.google.com/looker/docs/reference/param-explore-explore`.)

The two vocabularies name the same three things: a *grain* (what one row is), *dimensions* (how you cut it), and *measures* (how you aggregate it). The reason the semantic layer matters to a transformation is not that it is modern. It is that **the semantic layer is the only place in the architecture where a definition lives as a first-class artefact instead of being buried in a report definition, a stored procedure, or a spreadsheet formula.** A transformation without a semantic layer has nowhere to put the negotiated definitions, and a definition with nowhere to live is a definition that will be re-negotiated by accident every time someone writes a new query.

**What a semantic layer does not do.** Two boundaries matter for a transformation, because over-promising the layer is how a programme loses credibility:

- **It does not decide the definitions.** The semantic layer stores and compiles the definition; it does not adjudicate it. A layer full of definitions nobody owns is a machine for producing consistent numbers that are still contested. §4.5 supplies the missing half.
- **It does not correct bad grain.** A semantic model built over the wrong grain — a measure aggregated before the join that a dimension requires — will compile and will run and will be wrong, systematically, for every consumer. The grain defect is the most common defect in a semantic layer precisely because it is invisible in the compiled SQL: the query succeeds, the number is plausible, and the error is in the modelling. The transformation's mitigation is to reconcile *at the grain the metric is reported at* (§7.1), which is where grain defects surface.

**The choice, stated without a vendor preference.** Both vocabularies are described above because an architect will meet both, and because the transformation's decision is not "which product" — that belongs to the platform comparison owned elsewhere (**[REPO]** `advanced_analytics_solutions_guide.md` §5) — but "does the organisation have a place to declare a metric once?" The answer is the design question; the product is the implementation detail.

### 4.3 The definition versus the calculation

The single most useful distinction in this guide — and the one most commonly collapsed — is between the **definition** and the **calculation**.

A **definition** is an organisational fact. It answers a question that is not technical and cannot be settled by a database: *Who counts as a customer? What counts as revenue? When is a customer "active"? What is a "transaction"? Is a dormant account a customer?* These are decisions about the organisation's reality, and they belong to the business. Two reasonable people can hold two reasonable definitions; the job is not to find the technically-correct one but to *adjudicate* and *record*.

A **calculation** is the arithmetic that implements a definition. It answers a question that is entirely technical: *Which table, which filter, which join, which aggregation over which period?* Given a definition, the calculation is an engineering exercise. Given no definition, the calculation is a guess that happens to run.

The consequence for a transformation is direct:

- A **change in the calculation** (a faster join, a corrected filter, a re-pointed source) is an engineering change; it should not change the number, and reconciliation (§7) proves that it did not.
- A **change in the definition** (a customer now means something different) is a business change; it *will* change the number, and the change must be approved, dated, and — if the number was reported — treated as a restatement (§12).

Confusing the two is the origin of a specific pathology: a platform team changes what it believes is a calculation and thereby, without intending to, changes a definition, and the organisation discovers the change only when a number moves in a board pack. **The transformation must give the two different owners, different change processes, and different evidence requirements**, because a change that is safe as a calculation and a change that is safe as a definition are not the same change at all.

The two are distinguished along five dimensions, and the transformation should encode the distinction in its change process:

| Dimension | Change in the calculation | Change in the definition |
|---|---|---|
| **Nature** | Technical — how the arithmetic is performed | Organisational — what the term means |
| **Owner** | The analytics-engineering owner | The business definition owner (§4.5) |
| **Effect on the number** | None; the number must be unchanged, and reconciliation (§7) proves it | The number changes, and the change is expected |
| **Evidence required** | A reconciliation showing equivalence | A dated approval, a new definition version, and — if the number was reported — a restatement decision (§12.2) |
| **Process** | A normal engineering change | A governance event through the definitions board |

The table is the operational content of the whole section. A transformation that runs both kinds of change through one process will either bureaucratise its engineering changes or — far more commonly and far more dangerously — treat its definition changes as engineering changes and ship them without approval.

**Why the distinction survives the platform change.** The definition and the calculation are not properties of the platform; they are properties of the organisation's relationship to the number. The platform migration changes *where* the calculation runs and *how* it is expressed (a stored procedure becomes a semantic model, a hand-written SQL becomes a compiled query). It does not change *what the calculation implements* — and if it does, that is a definition change by the back door. The discipline is to make the migration change the calculation's form while holding its meaning constant, and to treat any drift in meaning as the governed event it is.

### 4.4 The definitional conflicts a bank actually has

The definitional problem is not hypothetical. A bank of any size contains, concurrently, several defensible definitions of things it believes are simple. Each of the following is a real pattern, stated as a class of conflict rather than as an allegation about any institution:

- **The same customer counted three ways by three functions.** Retail counts a customer by the relationship (any product holder). Cards counts by the account (any cardholder). Wealth counts by the portfolio (a relationship above a threshold). The three counts differ on the same date, each is correct against its own definition, and no one definition is more true than the others — they answer different questions.
- **Revenue recognised differently in different reports.** The product report recognises revenue when the fee is charged; the finance report recognises it when the performance obligation is satisfied; the management report recognises it when the cash lands. Three numbers labelled "revenue," each consistent with a different convention, appearing in the same meeting.
- **Active versus dormant.** One function treats "active" as any account that has not been formally closed; another requires a movement in the last period; a third requires a non-zero balance. The definitions produce different totals, and the difference is not an error in any of them.
- **A "transaction" that is not the same unit in payments and in accounting.** In payments, a transaction is a message — an authorisation, a clearing presentment, a settlement entry, each a separate event. In accounting, a transaction is a balanced posting. The "number of transactions" in a payments report and in a finance report count different things that share a name. Reconciling them is not arithmetic; it is translation, and it requires a stated mapping.

The important property of all four is that **the conflict is not evidence that someone is wrong.** It is evidence that the organisation has multiple definitions and has not recorded which one each report uses. A transformation that tries to force one definition by fiat on all three functions will fail, because it will be asking two functions to accept a number that makes their own reporting wrong. The transformation's job is to *surface the conflict, attach each definition to its owner, and record the reconciliation* — not to pretend the conflict does not exist.

The four conflicts share one shape, and naming the shape is what lets a transformation recognise a new instance of it:

| Contested term | Definition A (owner) | Definition B (owner) | Why the totals differ |
|---|---|---|---|
| Customer | Relationship — any product holder (retail) | Account — any cardholder (cards) | A product-holder need not hold a card, and vice versa |
| Revenue | At charge (product) | At obligation satisfied (finance) | Timing of recognition differs; the same economic event is booked in different periods |
| Active status | Account not formally closed | Movement in the last period | A live-but-untouched account is active under A, dormant under B |
| Transaction | A message (payments) | A balanced posting (accounting) | One economic event is many payment messages and one accounting entry |

The transformation's catalogue (§10.2) should record *which definition each artefact uses*, so that the shape becomes visible: the conflict is not in the data, it is in the *unrecorded assignment of definitions to artefacts*. Once the assignment is recorded, two numbers that disagree are no longer an error — they are two artefacts implementing two documented definitions, which is a state a reconciliation can manage.

### 4.5 Who adjudicates

If there is one adjudicator, the transformation should name them: a **definition authority** — a data owner for the domain (for customer, the customer-data owner; for revenue, the finance data owner) who has the standing to bind their function to a definition. In practice this is a *definitions board*: a standing body with the domain data owners, the analytics-engineering lead, and a governance representative, whose output is a dated decision per contested term. **[BCBS239] ¶34** anchors this in the regulatory discipline: roles and responsibilities "should be established as they relate to the ownership and quality of risk data," and the business owner's role includes ensuring data is "kept current and aligned with the data definitions." The definition authority is that role, made specific to BI.

**What "binding" means, and when adjudication fails.** An adjudicator is real only if a decision, once made, changes behaviour. The three failure states of adjudication are worth naming because each looks like success from the outside:

- **The board that meets and decides nothing.** Contested terms are discussed, minutes are taken, and the same terms return next quarter. The cause is almost always representation without authority: the attendees cannot bind their functions. The test is whether a decision, once taken, is implemented in the next sprint.
- **The board that decides and is overruled quietly.** A definition is adjudicated, a function continues to report its old number anyway, and no one notices because no metric's reconciliation was pointed at the decision. The cause is the absence of a *check*: the adjudication must be wired into the artefact inventory (§6.1) so that an artefact still implementing the superseded definition is flagged.
- **The board bypassed by escalation.** A function, losing an argument, escalates above the board and the decision is reversed without the board's reconciliation being updated. The remedy is not to forbid escalation — §14.3 shows escalation is sometimes the only resolution — but to require that any escalated decision lands *back in the glossary and change log* (§10.2), so the record reflects the governing definition and not the superseded one.

But the guide must be honest about the case the board cannot resolve: **sometimes there is no single answer.** When two definitions are both correct against their own question and neither function can abandon its number — because each is embedded in that function's reporting, its incentives, and in some cases its regulatory returns — a transformation **can lawfully ship a documented set of competing definitions with a stated reconciliation.** This is not a failure of the transformation; it is the correct output of a negotiation that genuinely has no single winner. What would be a failure is shipping one number that two functions believe is theirs, with no record that the other definition exists and no statement of why the numbers differ. The artefact the transformation must produce, in the contested case, is a *definition pair plus a reconciliation*: definition A, owned by function A, reading total A; definition B, owned by function B, reading total B; and a stated, owned reconciliation explaining the difference between A and B on any given date. **[BCBS239] footnote 16** is explicit that a bank "does not necessarily need to have one data model; rather, there should be robust automated reconciliation procedures where multiple models are in use." That footnote is the regulatory permission for the documented-competing-definitions outcome, and it is the single most useful sentence in the standard for a BI transformation.

### 4.6 The lineage question

A definition is only as good as the lineage that connects it to a number. If a metric's definition says "net revenue is fee income minus rebates," the lineage must show that the number labelled "net revenue" was actually computed from fee income and rebates and from nothing else. Where lineage is absent, the definition is a claim and the number is unverified against it — and the transformation has no way to prove that the migrated number equals the legacy number for the same reason. §7 and §10 develop the mechanics; the point here is that **the semantic layer and the lineage layer are two halves of one guarantee**: the semantic layer says what a number *should* be, the lineage says what it *is*, and neither is trustworthy without the other.

**The lineage break at the migration seam.** Lineage in a transformation is not merely a pre-existing graph to be consulted; it is a graph with a *seam* at the point the platform changed, and the seam is where lineage most often breaks. A number produced in the legacy estate has a lineage that ends at a stored procedure; the same number produced in the new estate has a lineage that begins at a semantic model. If the transformation does not record the *mapping between the two* — this legacy procedure became that semantic measure — then the lineage graph has a discontinuity exactly where the organisation most needs continuity, and no supervisor, auditor, or successor engineer can trace a pre-migration number through to its post-migration equivalent. Recording the migration mapping is therefore a lineage requirement, not a documentation nicety: it is what makes the two halves of the guarantee meet across the seam.

**The definition without lineage is a slogan.** It is worth stating the failure plainly, because it is common: an organisation writes a beautiful glossary — precise definitions, named owners, dated versions — and the glossary's definitions are not connected to the numbers by lineage. The result is a document that describes the world the organisation wishes it had, while the actual numbers are produced by logic nobody has traced. The glossary is then worse than useless, because it is trusted. Lineage is what keeps the glossary honest: it is the test of whether the definition is the one the number actually implements.

### 4.7 The honest statement

Everything in this section reduces to one sentence that the guide states plainly because it is the sentence the whole discipline rests on:

**A transformation which migrates the technology without settling the definitions has moved the problem rather than solved it.** The platform will be new; the numbers will still disagree; the disagreements will now be harder to see because they will be distributed across a modern tool instead of concentrated in an old one. The migration is necessary and it is not sufficient. The sufficient condition is that the definitions are surfaced, adjudicated or explicitly paired, owned, versioned, reconciled and versioned again, and that the platform is merely the place where they are stored.

**[BCBS239] ¶37** makes this a precondition in the standard's own words: "As a precondition, a bank should have a 'dictionary' of the concepts used, such that data is defined consistently across an organisation." The dictionary is the deliverable. The platform is the delivery mechanism. A transformation that ships the mechanism and skips the dictionary has done the easy half and invoiced for the whole.

**What "settling" actually requires.** The guide uses "settle" deliberately, because the word captures both of the outcomes §4.5 permits — a single adjudicated definition, or a documented pair with a reconciliation. To settle a definition is to record, in an owned and versioned artefact, either (a) the definition, or (b) the definitions and the reconciliation between them. Both are settlements; neither is a deferral. What is *not* a settlement is a working agreement that lives in a meeting's memory, a definition that exists in a report's SQL and nowhere else, or a decision that two functions will "align later." The transformation's definitional front (§2.2) is complete when every metric in the inventory (§6.1) has a settled definition — either kind — and not before.

**The cost of not settling, made concrete.** The cost is not abstract. An unsettled definition means every consumer of the metric re-derives it, which means the number differs by consumer, which means the organisation spends its meetings reconciling rather than deciding, which means the analysis that depends on the metric is never trusted enough to act on. The organisation has paid for the platform and is still paying, every month, for the definitional debt in analyst time and in decisions deferred. The transformation's definitional work is what retires that debt; the platform is merely the vehicle for retiring it.

---
## 5. Self-Service Enablement

Self-service is presented as a feature and behaves as a *bargain*. The transformation offers the business the ability to answer its own questions without waiting for the central team; in exchange, the business agrees to build over governed sources rather than around them. When the exchange is honoured, self-service is the highest-return capability a BI transformation ships. When it is not, self-service produces the next estate.

### 5.1 What it gives

Self-service buys three things, each real:

- **Throughput.** The central team's queue is the binding constraint on how many questions the organisation can answer. Self-service moves the long tail of one-off questions off that queue and onto the people who asked them, which raises the organisation's answer rate without adding central headcount.
- **Proximity to the question.** The person who has the question also has the context the central team lacks. A business user who knows that a particular product launch distorted a region's numbers can slice around it without writing a requirements document to explain why.
- **Fewer one-off requests.** The steady drip of "can you cut this a different way" requests — the requests that generate the report sprawl of §3.2 — slows when the requester can cut it themselves.

### 5.2 What it costs

The costs are structural, and the transformation must budget for them:

- **The governed-metric tension.** Self-service wants unconstrained fields — the freedom to combine, filter and derive. Governance wants fixed definitions — one number, one meaning. These pull in opposite directions, and the tension is not resolved by making one win. The resolution is architectural: expose a *governed base* (the certified metrics and dimensions) plus a *sandbox* where users may derive, with the sandbox outputs explicitly marked as ungoverned. The tension is managed by making the boundary visible, not by eliminating it.
- **The training and data-literacy question.** A self-service tool is not self-explanatory. The measure/dimension model of §4.2 requires a user to understand what a grain is, what an aggregation does, and why joining two tables at the wrong grain inflates a total. The transformation must fund the literacy, or it will ship a tool that users operate by imitation and misuse by accident.
- **The certification question.** Not every derived artefact should be equal. The organisation needs a mechanism to *certify* some datasets and metrics as governed and usable for reporting, and to leave the rest uncertified and explicitly unfit for headline use. Without certification, a derived table built by one analyst acquires the same apparent authority as a governed metric, and the governance the transformation bought is quietly undone.

### 5.3 The honest observation

**Self-service without governance produces a second estate.** When users are given a tool but not a governed source, they build what they need: derived tables, local extracts, join-and-aggregate views that encode their own definitions. The second estate is born. It is not malicious — it is the rational response to being given capability without a source of truth — and it is *the next thing to migrate*, because within a few years the derived artefacts have consumers, the consumers have dependencies, and the derived tables have become the very accretion §3 described, only this time inside the new platform. A transformation that ships self-service *before* it ships the governed metrics has scheduled its successor transformation. The sequencing is not optional; governance is the precondition for self-service, not its follow-up.

### 5.4 The self-service maturity question

Self-service is not a switch but a progression, and the transformation should be explicit about which stage it is targeting, because each stage has a different governance requirement:

- **Stage one — read the certified.** Users consume governed metrics and certified dashboards; they can filter and drill but not derive. Governance is cheap because the surface is fixed, and the value is that one number reaches everyone.
- **Stage two — derive in a governed sandbox.** Users build their own views over governed sources; the sandbox's outputs are explicitly marked ungoverned and unfit for headline reporting. Governance is moderate, and the marking is what keeps the sandbox from silently becoming the source of truth.
- **Stage three — publish back.** A user's derived artefact may be *promoted* into the governed estate through the same definition process as anything else (§4.5, §12.3). This is the stage that most readily fails, because promotion is where an ungoverned artefact tries to acquire governance by adoption rather than by adjudication. The rule: nothing enters the governed estate without a definition owner and a reconciliation, regardless of how popular it is.

A transformation that promises stage three and delivers stage one will disappoint; a transformation that promises stage one but releases the tools without the certified metrics will deliver stage three's chaos by accident.

---

## 6. The Migration Path

This section is about **reconciliation-bearing discipline and phasing**, not about tools. The tools are covered elsewhere in the series; what is covered here is the order of operations and the evidence each step must carry.

### 6.1 The report and metric inventory

Nothing can be migrated, retired, or reconciled until the organisation knows what it has. The inventory is built artefact by artefact, and for each artefact it records five things:

1. **The artefact** — its identity, its tool, its schedule, and its destination(s).
2. **Its owner** — a named person, not a team. The owner is the party who will be asked to sign a retirement, so an inventory with team-level owners is not yet an inventory.
3. **Its consumers** — who receives it, directly and through redistribution. This is the hardest field to populate, because redistribution is invisible.
4. **Its last use** — the timestamp of the most recent access, where the tool can report one. Where it cannot, this field is *unknown*, and unknown is recorded as unknown rather than as zero.
5. **Its definition source** — where the definition the artefact implements actually lives: the semantic layer, a report definition, a stored procedure, or — the common case — a spreadsheet.

The inventory is the transformation's map. Built properly, it is also the transformation's most politically exposed document, because it records who owns the artefacts nobody wants to own.

### 6.2 What to retire, and on what evidence

Retirement is where the transformation saves its money and where it most easily makes an enemy. The evidence standard must be set before retirement begins:

- **A named owner willing to sign that the artefact is not needed.** Not a team, not a committee — a person whose name is on the decision.
- **A usage record.** Access evidence, delivery evidence, or an interrogation of the consumers — something that establishes use as a fact rather than an assumption.
- **Explicitly not: "nobody complained."** The absence of complaint is not evidence of absence of use. The users of a low-frequency but high-consequence report — a quarterly regulatory extract, an annual audit support pack — will not complain during the migration window because they will not look for the report until the quarter closes. Silence is the default state until the report is needed, and a transformation that reads silence as consent will retire something it has to rebuild.

The rule the transformation should adopt: **an artefact is retired when a named owner signs and usage is established as absent; otherwise it is migrated, even if it looks unused.** The cost of migrating an unused report is small; the cost of rebuilding a retired one under regulatory duress is not.

### 6.3 The parallel-run discipline

Every migrated number runs in parallel before it is trusted. The parallel run is the mechanism (§7 details it), and the phasing question is *how long* each parallel run lasts and *what triggers* the cutover. The discipline: the parallel run ends when every difference between legacy and new is *explained*, not when the difference is small. A small unexplained difference is worse than a large explained one, because the small one will survive into production and surface at the worst moment.

### 6.4 Phasing options and their trade-offs

The transformation chooses one of three phasing families, each with a genuine downside:

| Phase family | Mechanics | Trade-off |
|---|---|---|
| **Strangler / incremental** **[STRANGLER]** | Route functionality from the legacy estate to the new estate behind a façade, domain by domain, until nothing routes to the legacy and it can be switched off. | Slow; requires a coexistence period in which both estates must be maintained, and the façade itself becomes an asset to own. Its failure mode is never finishing the strangling — the legacy survives as a "still a bit needed" system indefinitely. |
| **Big-bang** | Migrate everything and cut over on a date. | Fast and organisationally simple; the entire risk is concentrated on one date, and a defect discovered after cutover is discovered in production with no legacy to fall back to. Its failure mode is a failed cutover with no retreat path. |
| **Dual-running / expand-migrate-contract** **[PARALLEL]** | Expand (add the new, keep the old), migrate (move consumers and backfill, comparing throughout), contract (remove the old once nothing depends on it). | The most controlled of the three, and the most demanding of discipline. Its stated downside is precise: **failing to execute the contract phase leaves you worse off than before**, because you now maintain two estates forever and the old one never dies. The contract phase is the part organisations skip, because by then the migration is "done." |

The honest guidance: in a regulated institution, dual-running with an enforced contract phase is the least bad option, precisely because it is the only one whose intermediate states are all reversible (§10). The strangler is a special case of expand-migrate-contract (the façade *is* the expand); big-bang is the option to choose only when the estate is small enough that the whole thing can be reconstituted from source.

### 6.5 The hardest migration item

**The hardest migration item is usually the one nobody can explain the purpose of.** For an explained report, the migration's acceptance criterion is *equivalence*: the new number equals the old number for the same period, and the definition explains why. For an unexplained report, no definition exists to test against, so the only available acceptance criterion is *reproduction*: the new number must equal the old number, for reasons nobody can articulate. Reproduction is a strictly weaker guarantee. It proves the new artefact behaves like the old one; it does not prove either is correct, and it bakes the old one's unknown errors into the new estate.

The transformation's response is to convert unexplained artefacts into explained ones *before* migrating them: trace the logic, name the definition, find an owner. Where that cannot be done, migrate the artefact *and record it as unexplained*, so the liability is visible and carried forward rather than laundered.

### 6.6 Why a number that moves is an incident

In a regulated institution, **a number that changes without explanation is an incident.** The reason is not bureaucracy. A reported number has been relied upon: it has been sent to a counterparty, filed with a regulator, used in a board decision, or reported to the market. If the same period's number changes on re-run, then either the earlier number was wrong (and the reliance was misplaced) or the new number is wrong (and the change is a defect). Either way, the organisation's *trust* in its numbers is damaged, and trust is the asset a BI estate exists to hold. A transformation that lets numbers move silently during migration has manufactured a series of small trust failures, and in a regulated institution those failures are reportable. The discipline of §7 is what prevents the movement, or — where a movement is correct and intended — converts it from a silent drift into a governed, dated, approved event.

---

## 7. Reconciliation

Reconciliation is the conscience of the transformation. **[BCBS239] footnote 17** defines it precisely, and the definition is worth quoting in full because it contains the word that does the work: reconciliation is "the process of comparing items or outcomes **and explaining the differences**." The emphasis is this guide's, but the requirement is the standard's. Comparison is the easy half; explanation is the discipline.

### 7.1 The parallel-run discipline

For each migrated metric, the transformation runs the legacy computation and the new computation side by side for a defined period, comparing every period-end. The rules of the parallel run:

1. **Run both; do not trust one.** The parallel run is not a smoke test; it is a sustained comparison over enough periods to include the awkward ones (month-end, quarter-end, a period with a known one-off).
2. **Compare exactly.** The comparison is at the grain the metric is reported at, not at a total that hides offsetting differences.
3. **Explain every difference.** Each difference is either a *calculation* difference (a defect — fix it and re-run) or a *definition* difference (a change in meaning — govern it as a definition change, per §12). There is no third category called "acceptable."
4. **Do not "reconcile" by loosening a tolerance.** This is the cardinal sin. When a difference persists and cannot be explained, the temptation is to widen the tolerance until the difference disappears. Widening the tolerance does not explain the difference; it hides it. A tolerance that has been set to accommodate an unexplained gap is a tolerance that has been set by the wrong party for the wrong reason, and it will hide the same class of gap forever.

### 7.2 The evidence a migrated number must carry

For a number to be *reconciled* rather than merely *equal*, it must travel with four pieces of evidence:

- **The definition version** — which definition (and which version of it) this number implements.
- **The as-of timestamp** — the moment the data was as it was when the number was produced.
- **The source lineage** — the traced path from the number back to the source records.
- **The transformation that produced it** — the specific, versioned logic that turned the sources into the number.

A number without these four is a number that cannot be explained to a supervisor, audited against a definition, or reproduced after a platform change. It is, in the transformation's terms, un-reversible (§10). The four pieces of evidence are what make the number an artefact rather than an output.

### 7.3 The tolerance question

A tolerance is a statement about materiality — *how large a difference matters* — and materiality is a business decision, not a technical one. The consequence is that **the tolerance must be set by the business owner, not by the platform team.** **[BCBS239] ¶23** defines materiality in exactly these terms: data and reports may exclude information "only if it does not affect the decision-making process," judged by whether "decision-makers, in particular the board and senior management, would have been influenced by the omitted information or made a different judgment if the correct information had been known." A platform team that sets its own tolerance has made a materiality judgement it is not empowered to make, and it will set the tolerance to whatever makes the migration converge. The tolerance must be owned, dated, and justified by the party who would be harmed by a difference larger than it.

### 7.4 What happens when a number moves

When reconciliation establishes that a number has — or must — move, the response is a fixed sequence:

1. **Freeze.** Stop the old number from being produced further; prevent an unfrozen pair of numbers from circulating.
2. **Explain.** Establish and record why the number moved: a calculation correction, or a definition change.
3. **Date the change.** Record the effective date and the date the change was made — they may differ, and both matter.
4. **Approve it.** The definition owner (for a definition change) or the engineering owner (for a calculation correction), on the evidence, per the governance of §10 and §12.
5. **If it was a reported number, treat it as a restatement.** A number that was issued, filed, or reported and now changes is not a refresh; it is a correction of a prior period, and §12 explains why that is a governance event with an accounting analogue.

The sequence is not ceremony. Each step converts an ungoverned change into a governed one, which is the only kind a regulated institution can defend.

### 7.5 The reconciliation register

The discipline of §7.1–§7.4 produces an artefact that the transformation should maintain as a first-class deliverable: the **reconciliation register**. For every migrated metric it records: the metric, its definition and definition version, the legacy and new computations, the difference observed per period, the explanation of each difference (calculation or definition), the tolerance and its business owner, and the sign-off with its date.

The register is what makes the transformation's reconciliation *auditable* rather than merely *performed*. It answers, without re-derivation, the question a supervisor, an auditor, or a successor team will ask: "when this metric moved from the legacy estate to the new one, what changed, why, who approved it, and how large was the difference?" A transformation that reconciles diligently but cannot produce the register has done the work and lost the evidence. The register belongs in the governance of §10 (it is a member of the change log family) and is the concrete form of the §7.2 evidence a migrated number must carry.

One discipline about the register: it records *explained* differences and refuses to record *tolerated* ones. A register row whose "explanation" reads "within tolerance" is a row that has failed the standard of §7.1's fourth rule; it documents that a difference exists and conceals that no one can say why. The register is only as valuable as its refusal to accept that entry.

---
## 8. The Destination, Mapped Not Restated

The destination of a BI transformation is the modern data stack: a raw/landing zone (often object storage), a transformation layer (SQL and orchestration), a semantic layer (the governed metrics of §4.2), a BI/presentation layer, and a governance layer (catalogue and lineage). This guide does **not** re-derive that stack, its components, its comparisons, or its maturity model. Those are owned elsewhere, and this section maps them onto the transformation and marks them "owned elsewhere, not restated here."

### 8.1 The stack, as a map

The transformation's destination is best read as a stack with four seams, each of which the transformation crosses:

- **Raw/landing (the lake or lakehouse).** The transformation's ingestion seam: source systems land in cheap, schema-flexible storage. The lakehouse guides own the destination's shape. **[REPO]** `cloud_object_storage_lakehouse_guide.md`; `data/on_prem_lakehouse_guide.md`; `data/on_prem_lakehouse_analysis.md`; the object-storage comparison coverage.
- **Transformation (the modelled layer).** Where the calculation of §4.3 lives and is versioned. **[REPO]** `data/enterprise_data_platforms_guide.md` owns the platform category; `data/data_pipeline_guide.md` and `data/dataops_guide.md` own the pipeline discipline.
- **Semantic (the governed metrics).** The §4.2 semantic layer — the place a definition becomes a first-class artefact. **[REPO]** `technology/ensemble_logical_model_elm.md` maps the business-to-technology modelling bridge that feeds the semantic layer; **[DBT]** and **[LOOKML]** are the two vocabulary anchors of §4.2.
- **Presentation (the reports and dashboards).** The estate of §3, rebuilt. **[REPO]** `advanced_analytics_solutions_guide.md` §13 (regulatory reporting analytics) and its banking patterns.

### 8.2 What is owned elsewhere

The following are explicitly owned by neighbours and are not restated here. Each is mapped, not re-derived:

| Topic | Owned by | What this guide takes from it |
|---|---|---|
| Analytics maturity model (§2) | `advanced_analytics_solutions_guide.md` | The transformation is the movement, not the maturity level itself. |
| Technology stack for analytics (§3) | `advanced_analytics_solutions_guide.md` | The destination's components; this guide assumes them. |
| The modern data stack (§4) | `advanced_analytics_solutions_guide.md` | The destination's shape; not re-explained. |
| Platform comparison (§5) | `advanced_analytics_solutions_guide.md` | The choice is made there; the *move* is made here. |
| Real-time analytics engines (§6) | `advanced_analytics_solutions_guide.md` | The freshness dimension of a migrated metric, not re-derived. |
| Platform decision matrix (§8) | `advanced_analytics_solutions_guide.md` | Selection criteria; not repeated. |
| Banking analytics patterns (§9–§13) | `advanced_analytics_solutions_guide.md` | The use cases the migrated metrics serve. |
| Enterprise data-platform architecture | `data/enterprise_data_platforms_guide.md` | The platform category; the transformation consumes it. |
| Lakehouse destination | the lakehouse guides | The landing zone; not re-argued. |

### 8.3 What the mapping implies for the transformation

Reading the stack as a map yields one implication the transformation must internalise: **the seams are where transformations fail, not the components.** Each component can be individually sound and the transformation still fail at a seam — between raw and modelled (a landing convention that loses a key), between modelled and semantic (a definition implemented twice, once in SQL and once in the semantic layer, which is the §3 shadow-estate problem reborn), between semantic and presentation (a dashboard that joins around the semantic layer and thereby writes its own definition), or between any layer and governance (an artefact the catalogue does not know about). The transformation's job at the destination is not to build the components — that is the stack's job — but to make the seams governed.

---

## 9. The Organisation and the Skills

This section covers the organisation and the skills **only as the transformation changes them.** The operating models, the lifecycle, team topologies, and hiring are owned by **[REPO]** `../management/analytics_management_guide.md` and are not re-derived here.

### 9.1 The three functions and how the transformation reshapes them

A transformation forces a division of labour that most organisations have not made explicit:

- **The platform team.** Owns the compute, storage, orchestration, and the reliability of the estate. The transformation *shrinks* this team's definitional role: it should not be deciding what a metric means, and the anti-pattern below is precisely that it does.
- **The analytics-engineering team.** Owns the transformation and semantic layers — the modelling, the metric implementation, the reconciliation plumbing. This team *grows* in a transformation, because the semantic layer and the parallel-run harness are its new products.
- **The business-side definition owners.** Own the definitions (§4.5). The transformation *creates* this role where it did not exist, or *activates* it where the titles existed but the authority did not. The definition owner is not a data team member; they are the business party accountable for the meaning.

The transformation's organisational test: can you name, for each governed metric, a person outside the data teams who owns its definition? If not, the transformation has built a semantic layer with no one to fill it.

### 9.2 The new skills

Three skills are new or newly central, and the transformation must source them:

1. **Semantic modelling.** The ability to express a definition as a metric declaration over entities, dimensions, and measures (§4.2) — and, crucially, to *choose the grain* correctly, because most semantic-layer defects are grain defects.
2. **Metric stewardship.** The ongoing ownership of a metric's definition: reviewing change requests, dating versions, maintaining the reconciliation, and being the answerable party when the number is questioned.
3. **Reconciliation analysis.** The skill of explaining a difference rather than dismissing it — reading two computations, finding the cause of a divergence, and classifying it as calculation or definition. This is a distinct skill from either data engineering or business analysis, and it is the skill that most often goes unstaffed.

### 9.3 The anti-pattern: the transformation owned by the platform team alone

**A recurring organisational failure — and the one this guide treats as the most damaging — is the transformation owned by the platform team alone.** It is understandable — the platform team is the team that knows how to migrate — and it is fatal, because the platform team is structurally the wrong owner of the definitional front. Its incentives are uptime, throughput and decommissioning; its skills are infrastructure; and its natural bias is to treat the definitional problems as business problems to be deferred. When the platform team alone owns the transformation, the "five fronts" of §2.2 collapse to one: the platform gets migrated, the definitions get deferred, and the organisation arrives at §4.7's outcome — the problem moved, not solved. The remedy is structural: a transformation owner accountable across all five fronts, with the definition owners seated at the programme's governance, and the platform team accountable for the platform *seam* rather than the whole.

### 9.4 How the definition authority is staffed

The definition authority (§4.5) is staffed, not announced. In practice it is a standing board whose members are the domain data owners (with delegated authority to bind their function), the analytics-engineering lead (to state the implementation consequence of a definition), and a governance representative (to record and version). It meets on a cadence, it decides contested terms, and — the part that makes it real — its decisions are binding: a function that loses a definitional argument implements the change, or registers a documented competing definition and its reconciliation (§4.5). A definition authority whose decisions can be ignored is a meeting, not a body.

---

## 10. Governance, the Catalogue and Lineage

The governance of §2.2's fifth front exists for one reason in a BI transformation: **to make the transformation reversible.** Reversibility is the property that separates a transformation that can be corrected from one that has made a permanent, un-auditable commitment.

### 10.1 What "reversible" means here

The transformation is reversible when, after the platform has moved, an artefact that was migrated or retired can be *recovered and explained*: its number reproduced, its definition identified, its source lineage traced, and the decision that changed it found. Not every part of a transformation is reversible — a decommissioned source system may be gone — so the programme marks, artefact by artefact, which changes are reversible and prices the irreversible ones (which are precisely the ones that get the most governance, not the least).

### 10.2 What must be owned, catalogued and traced

Four things must be in place for reversibility:

1. **The artefact-level inventory.** The §6.1 inventory, live and maintained — not a one-off migration asset but the estate's standing ownership record. **[REPO]** `data/data_governance_framework.md` §10 owns the data-catalog and metadata-management treatment; the transformation maintains the BI artefacts inside it.
2. **The business glossary of definitions, with owners and versions.** The §4.5 definitions made concrete: term, definition text, owner, version, effective date. This is the "dictionary" **[BCBS239] ¶37** requires, applied to BI rather than risk data.
3. **Column- and report-level lineage.** The traced path §4.6 depends on, at both the column level (which source column fed which target column) and the report level (which tables and metrics produced which report). **[REPO]** `data/data_lineage_tools.md` §1–§3 owns the lineage collection approaches and tooling; the transformation's requirement is that lineage be *continuous across the migration* — a number's lineage must not break at the point the platform changed.
4. **The change log of definition changes.** The dated record of every definitional decision and its approval (§12.3). This is the artefact that makes a number's history legible: for any reported number, what definition was in force, and when it changed.

### 10.3 The governance cross-references

The disciplines above are owned elsewhere in the series and are applied, not re-derived, here:

- **[REPO]** `data/data_governance_framework.md` — its §10 (data catalog and metadata management) is the home of the catalogue discipline; its ownership model (data owner, steward, custodian) is the staffing of §9.4.
- **[REPO]** `data/data_lineage_tools.md` — the lineage collection approaches (§3), the tooling (§4–§6), and the anti-patterns (§10) that a transformation must avoid (a lineage graph nobody maintains is worse than none, because it is trusted).
- **[REPO]** `technology/ensemble_logical_model_elm.md` — the business-to-technology modelling bridge that gives the glossary its structure, so that definitions attach to entities rather than floating as prose.
- **[REPO]** `data/data_fabric_guide.md` — the federated-governance angle where the transformation's estate is not a single platform.

### 10.4 The recurrence trap

The governance must be *sustained*, and the transformation's most common governance failure is that it stops at go-live. A catalogue, a glossary, and a lineage graph are living artefacts; if they are populated for the migration and then unmaintained, they degrade into a snapshot of a past state and actively mislead — a reader trusts the snapshot and is wrong. The transformation's governance design must therefore include the *ongoing* owner of the catalogue and glossary, or it will have bought reversibility for the migration window only.

---
## 11. The Business Case

This section describes **how the business case for a BI transformation is built, and which parts of it are usually unvalidated.** It does not assert a business case, and it does not present a savings figure, because a savings figure is a claim about a specific organisation that this guide has no source for.

### 11.1 How the case is built

A transformation's business case is assembled from five benefit lines, each legitimate as an *arguable* benefit and each with a weakness:

1. **Avoided licence and hardware spend.** The cost the organisation would have paid to continue on the incumbent platform, avoided by moving. *Weakness: the avoided spend is a forecast of a renewal that has not happened; if the renewal would have been negotiated down, the avoidance shrinks.*
2. **Retire-led cost avoidance.** The cost of running the artefacts that the transformation retires — the licences, the compute, the scheduled jobs, the support. *Weakness: the saving is contingent on the retirement actually happening. A benefit line that assumes the §6.2 retirements will occur is a benefit line that must be tracked against the retirement register, not asserted at business-case time.*
3. **Analyst time not displaced.** The time freed when analysts no longer hand-assemble the numbers the semantic layer now produces. *Weakness: freed time becomes a saving only if it is redeployed. If the analysts do something else equally valuable, the organisation has gained capacity, not cash — and capacity is not the same as a saving.*
4. **Risk avoidance of unsupported platforms.** The avoided cost of a failure on an end-of-support platform. *Weakness: risk avoidance has no counterfactual. The failure that did not occur cannot be valued, and a benefit computed from a probability-weighted catastrophe is a benefit nobody can verify after the fact.*
5. **Faster change.** The reduction in the time it takes to deliver a new report or a new cut, once the semantic layer exists. *Weakness: "faster" is measurable only against a baseline of change-lead-time, and most organisations do not have the baseline.*

**[BCBS239] ¶5** is the standard's own honest statement of this class of investment, and it is worth quoting because it matches the discipline this guide is trying to enforce: "IT systems, data and reporting processes require significant investments of financial and human resources with benefits that may only be realised over the long-term."

### 11.2 What is usually unvalidated

The pattern across the five lines is not that they are false; it is that they are **unvalidated after the fact**:

- **The retire-led saving needs a retirement to actually happen.** The business case counts the saving of artefacts that the migration *plans* to retire. Unless the retirement register of §6.2 is tracked against the benefit line, the saving is a forecast that never gets reconciled to reality. The retire-led saving is the single most-tracked benefit in theory and the least-reconciled in practice.
- **The time saving needs time to be redeployed, and it is not measured.** Analyst hours freed are counted at the business-case rate and never checked against what the analysts actually did with the hours. If they did the same work more slowly, the saving did not occur; nobody checks.
- **The risk-avoidance benefit has no counterfactual.** There is no way to observe the failure that was avoided, so the benefit can never be confirmed or denied. It is real as a *reason to act* and unverifiable as a *benefit realised*.

### 11.3 The honest statement

The guide's honest statement on the business case is the sentence the section exists for:

**A savings figure nobody will measure afterwards is a permission slip rather than a plan.**

That is not a counsel to refuse the case; a transformation must be funded. It is a counsel to *label* the case's benefit lines by their evidence status: which lines are verified, which are forecast, and which are inherently unverifiable. A business case that presents all five lines with equal confidence is a business case that has hidden its uncertainty, and the hidden uncertainty resurfaces when the first benefit line fails to materialise and the programme's credibility goes with it.

### 11.4 The standard's belief, attributed as a belief

The strongest statement available in the source pack for the long-term case is not a measurement, and it must be attributed as what it is. **[BCBS239] ¶11**: "The Committee **believes** that the long-term benefits of improved risk data aggregation capabilities and risk reporting practices will outweigh the investment costs incurred by banks." This is the Basel Committee's *belief*, stated as a belief. It is authoritative as the international standard-setter's view; it is not a measurement, not a study, and not a figure. A transformation's business case may cite it as the standard-setter's judgement, and must not present it as evidence of a return.

**[REPO]** `../management/business_case_development_guide.md` owns the construction method — the financials, the options appraisal, the risk analysis, the stakeholders — and this section applies its discipline to the BI-transformation case rather than re-deriving it.

### 11.5 Making the case trackable

The remedy for the unvalidated benefit lines is not to drop them; it is to make them *trackable*, so that the programme's claims can be confirmed or corrected as the transformation proceeds. Three practices convert a business case from a permission slip into a plan:

- **Name the counterfactual for each line.** "Avoided licence spend" is only real if the renewal actually would have cost what the case says; the case should record the renewal quote it is based on, so the avoidance can be checked rather than assumed.
- **Tie the retire-led saving to the retirement register.** The saving exists only for artefacts actually retired (§6.2); the case's saving should be recomputed from the register, not from the plan.
- **Recognise the unverifiable as unverifiable.** The risk-avoidance line cannot be confirmed after the fact, and the honest case says so rather than presenting it with the same confidence as the licence line. A case that labels its lines by evidence status is a case that survives the first line's failure.

The point is not scepticism about the transformation; it is that the transformation's credibility is a programme asset, and it is spent when benefit lines are presented as certainties and then quietly fail to materialise.

---

## 12. The Regulated-Institution Angle

For a bank, the BI transformation is not only a data programme; it is a governance programme with a supervisory audience. This section links the transformation's definitions, reconciliation and lineage to the expectations that apply to a regulated institution, and it **asserts no jurisdiction's rule without a source.**

### 12.1 What a regulator's lineage expectation implies for a semantic layer

**[BCBS239]** is an international standard — the Basel Committee's *Principles for effective risk data aggregation and risk reporting* (January 2013) — and its expectations bear directly on the semantic layer of §4.2. Its provenance here is as an international standard, not as any national rule:

- **P2, ¶33 — integrated data taxonomies and metadata.** "A bank should establish integrated data taxonomies and architecture across the banking group, which includes information on the characteristics of the data (metadata), as well as use of single identifiers and/or unified naming conventions for data including legal entities, counterparties, customers and accounts." *Implication:* a semantic layer that is a taxonomy with metadata — not a pile of report logic — is the architecture the principle describes; single identifiers and unified naming are the substrate the semantic layer's entities depend on.
- **P3, ¶36(c) — reconcile to sources.** "Risk data should be reconciled with bank's sources, including accounting data where appropriate, to ensure that the risk data is accurate." *Implication:* the §7 reconciliation discipline has a regulatory analogue; the semantic layer's numbers must reconcile to sources, including accounting.
- **P3, ¶36(d) — a single authoritative source per risk type.** "A bank should strive towards a single authoritative source for risk data per each type of risk." *Implication:* the semantic layer's role is to be that single authoritative source for the *definition*, even where the physical sources are plural; and §4.5's documented-competing-definitions outcome is bounded by this — the standard asks the bank to *strive* toward one source, and permits multiple only under §4.5's reconciliation condition.
- **P7 — "Reports should be reconciled and validated."** *Implication:* a report produced from a semantic layer is not exempt from reconciliation because the definition is now governed; the report itself must be reconciled and validated.
- **¶22 — trade-offs explainable, limitations disclosed.** Trade-offs among principles "could be accepted in exceptional circumstances," but "decision-makers at banks, in particular the board and senior management, should be aware of these trade-offs and the limitations or shortcomings associated with them," and banks "should be able to explain the impact of these trade-offs on their decision-making process." *Implication:* where the transformation cannot settle a definition and ships a documented pair (§4.5), the limitation must be disclosed to the decision-makers, not buried.

**[REPO]** `../banking/risk_data_aggregation_guide.md` owns the BCBS 239 discipline in full — the fourteen principles, the governance and aggregation and reporting layers, the implementation history. This guide does not re-derive them; it applies the principle that bears on the semantic layer.

**On the deadline and on national implementation.** The standard set the G-SIB deadline at January 2016 (¶14: banks identified as G-SIBs by the FSB in November 2011 or November 2012 "must meet these Principles by January 2016"). That deadline is a fact of the international standard. **National implementation differs, and this guide asserts no national rule, no national deadline, and no national supervisory expectation.** A reader seeking the applicable national rule must consult the relevant supervisor's own published instruments.

### 12.2 Why a restatement is a governance event

The transformation's most consequential output is not a new platform; it is a *changed number*, and in a regulated institution a changed number is governed by an accounting analogy. **[IAS8]** — IAS 8 *Accounting Policies, Changes in Accounting Estimates and Errors* — draws the distinction the transformation must respect:

- **Prior period errors are corrected retrospectively by restating comparatives.** If a number was *wrong* (the definition was implemented incorrectly, or a source was misread), the correction looks backwards: the prior period's figure is restated so the comparison remains meaningful.
- **Changes in accounting estimates are applied prospectively.** If a number *changed because an estimate changed* — not because a past figure was wrong — the change looks forward and the prior period is not restated.
- **The distinction matters** because it determines whether history is rewritten. A transformation that treats a definition change as a "refresh" and silently repaints prior periods has, in effect, made a retrospective restatement without the disclosure, approval, and dating that a restatement requires.

The consequence for §7.4: when a number moves, the transformation must classify the movement — *error correction* (retrospective, restatement) or *definition/estimate change* (prospective, dated, approved) — because the classification determines the governance treatment, the disclosure, and whether prior reported numbers are restated.

### 12.3 How a definition change is approved, evidenced and dated

The transformation's change process for a definition, stated concretely:

1. **Raise** — a definition change request naming the metric, the current definition, and the proposed one.
2. **Adjudicate** — the definition authority (§4.5, §9.4) decides: adopt, reject, or adopt-as-competing (§4.5), each with a stated reconciliation.
3. **Evidence** — the decision records the discrepancy that motivated it (the reconciliation), the affected artefacts (the §6.1 inventory), and the affected reported numbers.
4. **Approve** — the definition owner signs; where the number is reported externally, the approval escalates to the function accountable for the reported number.
5. **Date and version** — the definition's new version and effective date are recorded in the glossary (§10.2), and the change log (§10.2) carries the approval.
6. **Reconcile forward** — the parallel run (§7.1) establishes the new number against the old, and the difference is explained per §7.3.

Steps 1–6 are the mechanism by which a definition change becomes an auditable event rather than a silent drift. In a regulated institution, this process *is* the transformation's regulatory value: it is what lets the organisation answer "why is this number different from last period, and who approved the difference?"

---
## 13. The Failure Modes

Six failure modes recur across BI transformations. Each is stated as a short case with its mechanism, because the mechanism is what a reader can recognise in their own programme.

### 13.1 (a) The lift-and-shift that changed nothing

**The case.** The organisation migrates its warehouse to a cloud platform, re-points the ETL, and lifts the reports across with minimal change. The platform is new; the reports look the same; the definitions are exactly what they were. **The mechanism.** The programme's success criterion was "the reports run on the new platform," and that criterion was met. But nothing in the estate's *meaning* changed: the same three definitions of "customer" exist, the same shadow spreadsheets adjust the same numbers, and the same unexplained reports are still unexplained. The transformation has paid for a platform and bought nothing but a different address. Its cost is real and its benefit is a licence reduction; its definitional debt is unchanged and now sits on a more expensive foundation.

### 13.2 (b) The migration that moved the warehouse and kept the definitions

**The case.** Closely related to (a) but more insidious: the organisation *does* engage with definitions, writes a glossary, and then migrates the warehouse while the glossary's definitions are *not implemented* in the new estate. **The mechanism.** The glossary is produced as documentation, reviewed by a committee, and filed. The new estate implements the same old logic, because implementing the glossary would have meant revisiting every report and every disagreement, and the migration deadline did not allow it. The result is a documented aspiration (the glossary) and an undocumented reality (the estate), diverging from the first day. The glossary now actively misleads: it says "customer is X" and the number says otherwise. The problem was not solved; it was packaged as a document that made the gap harder to see.

### 13.3 (c) The metric war

**The case.** Two functions each report "the same" metric — say, net revenue — and the numbers differ. Neither function can accept the other's number, because each is embedded in that function's reporting and incentives. **The mechanism.** There is no adjudicator. The definitional conflict of §4.4 is never surfaced to a body empowered to decide, so each function continues to produce its own number and to treat the other's as wrong. When a senior meeting requests "the number," two numbers arrive, and the platform team is blamed for the discrepancy — because the platform is the visible thing and the missing adjudicator is invisible. The war is not resolved; it is *localised*, and the platform team absorbs a governance failure it did not cause and cannot fix.

### 13.4 (d) The adoption cliff

**The case.** The new tool ships, the training is delivered, the licences are bought — and the users keep the old spreadsheet. **The mechanism.** The new tool requires the user to change a habit and to trust a source whose provenance they do not yet know, while the old workbook is familiar, trusted, and under their control. Adoption is not a function of tool quality; it is a function of the *switching cost* the user perceives, and a transformation that ships a tool without removing or de-authorising the old path has offered users a choice they will decline. The cliff is at the moment the spreadsheet remains available "just in case," which is the moment the second estate (§5.3) is born.

### 13.5 (e) The transformation owned by the platform team alone

**The case.** §9.3 in the field. The programme is owned by the platform team; the definitional front is deferred as "a business problem"; the definitions are never adjudicated. **The mechanism.** Ownership determines what gets attention, and the platform team's attention is structurally on the platform. The five fronts of §2.2 collapse to one, the definitions are carried forward unsettled, and the organisation arrives at the outcome §4.7 warns against — the technology migrated, the definitions not. The failure is not competence; it is ownership mismatch, and it is the failure this guide treats as the most damaging of the six, because it can cause the others.

### 13.6 (f) The report that was retired and rebuilt in a spreadsheet

**The case.** Using the evidence standard of §6.2, the transformation identifies a report as unused, retires it — and within a month, it is rebuilt by hand in a spreadsheet by a user who needed it. **The mechanism.** The usage record was absent or wrong (the report was used infrequently, not never — see §6.2's warning about "nobody complained"), the named owner signed a retirement they were not positioned to authorise, or the consumers were not the report's nominal recipients but a downstream party who received a redistribution. The rebuilt spreadsheet is now outside the catalogue, outside lineage, outside the change log — the worst of §10.1's non-reversibility, created by the retirement that was supposed to increase control. The lesson is not "retire less"; it is "establish usage as a fact before retiring, not as an absence."

---

## 14. The Cymbal Bank Worked Example

> **This section is fiction, and it is labelled illustrative.** Cymbal Bank is a fictional institution. Every figure, date, party, and event below is an invented illustration constructed to show the discipline in operation; none of it is a claim about any real bank, and none of it is sourced. Cymbal Bank is the only institution in this worked example; no other bank, and no other institution, is named or implied. The figures are worked illustrations and are labelled as such here and in the claims audit of §15, in accordance with this guide's verification posture.

Cymbal Bank is transforming a legacy reporting estate. The platform work is underway; the definitions are the hard part. Three episodes show why.

### 14.1 The definitional conflict: two defensible definitions of "active customer"

Cymbal Bank's retail and cards functions each report an "active customer" count for the same date, and the two totals differ — in the illustration, retail reports 4,200,000 and cards reports 3,750,000 for the same quarter-end. Neither is wrong:

- **Retail's definition:** an *active customer* is a customer with **any transaction in the last 90 days**. This definition answers the retail question — who is engaging with the bank — and it counts a customer who moves money through the current account but holds no other product.
- **Cards' definition:** an *active customer* is a customer with a **non-zero product balance**. This definition answers the cards question — whose card relationship is live — and it counts a customer whose card balance is non-zero even if they have not transacted in the window.

Each definition is defensible *against its own question*. Retail's is a behavioural definition; cards' is a balance definition. Each is owned by a different function; each is used in that function's performance reporting; each produces its own total for the same date. The 450,000 difference is not an error — it is the population that transacts without a card balance (counted by retail, not cards) net of the population that holds a card balance without transacting (counted by cards, not retail), and the two do not simply offset.

The definitions board (Cymbal Bank's §4.5 definition authority) adjudicates and does **not** force a single answer. It ships a documented pair: "active customer (behavioural, retail-owned)" and "active customer (balance, cards-owned)," each with its owner, its version, and a stated reconciliation — the labelled, owned arithmetic connecting the two — registered in the glossary and the change log (§10.2). This is the **§4.5 documented-competing-definitions outcome**, and it is lawful under **[BCBS239] footnote 16** (one data model is not required where robust automated reconciliation exists). The alternative — forcing retail's definition on cards — would have been a false single answer, correct for retail's question and wrong for cards', and it would have been implemented by a platform team that had no standing to make the call.

### 14.2 The reconciliation discipline applied to one migrated metric

Take one metric — *net fee revenue* — as the illustration. Cymbal Bank migrates it to the semantic layer, and the parallel run (§7.1) produces a difference:

1. **The parallel run.** For three quarters, Cymbal Bank computes net fee revenue in the legacy estate and in the new semantic layer, every period-end, at the grain the metric is reported at.
2. **The difference found.** In the illustration, the two computations differ by a small amount in each of the three quarters — and, critically, the difference *varies* by quarter (it is not a constant offset, which would point at a single mis-mapped source). A varying difference means the two computations are treating some population differently.
3. **The explanation.** Reconciliation analysis (§9.2) traces the difference to a *definitional* divergence, not a calculation defect: the legacy report excludes a class of fee rebates from net fee revenue, while the new semantic-layer definition, written from the glossary, *nets* the rebates. Neither computation is broken; they implement different definitions of the same term. This is exactly the §4.3 calculation-versus-definition distinction, found in the field: the difference is not a bug to fix but a definition to govern.
4. **The tolerance set by the business owner.** Cymbal Bank's finance-side definition owner sets the materiality tolerance (§7.3, anchored in **[BCBS239] ¶23**) — the platform team does not set it, because materiality is a business judgement. In the illustration, the owner sets the tolerance for this metric tighter than the size of the difference, which means the difference must be *explained and governed*, not merely tolerated.
5. **The sign-off and the dated definition version.** Since the divergence is definitional, Cymbal Bank governs it: the definition owner decides to adopt the net-of-rebates definition (the prospective treatment of a definition change, §12.2), the new definition version is dated and recorded in the glossary, the change log carries the approval, and the prior periods are *not* restated — because this is a definition change applied prospectively, not a correction of a prior-period error (§12.2). The migrated metric ships with its definition version, as-of timestamp, source lineage, and the transformation that produced it — the four pieces of evidence of §7.2.

The migrated number is now not merely *equal* to a computation; it is *reconciled* — compared and explained — and it carries the evidence that makes it reversible (§10.1).

### 14.3 The blocker that is political, not technical

The third episode is where the transformation nearly fails, and it fails for a reason no schema change can address.

Cymbal Bank's cards function will not surrender its "active customer" definition. The technical resolution is trivial — the definition pair of §14.1 is exactly the engineering Cymbal Bank ships — but the cards function resists even the *documented-pair* outcome, because the behavioural definition (retail's) would, if adopted for the cards function's own reporting, change the number in the cards function's performance pack. The number is used in the function's performance reporting; adopting another definition would make the function's headline look worse without any change in the function's actual business. The function's resistance is not irrational and not technical: it is a rational defence of a number on which the function is measured.

The resolution is **executive adjudication, not a schema change.** In the illustration, the definitions board cannot bind the cards function because the function's data owner will not sign, and the matter escalates to the executive committee, which rules that the documented pair stands, that the cards function reports its own (balance) definition *explicitly labelled as such*, and that no function's performance pack silently adopts another function's definition. The executive ruling is what unblocks the transformation — and it is available only because the definitions board (§9.4) was *seated at the programme's governance from the start* (§9.3), rather than convened after the conflict had hardened.

The lesson Cymbal Bank draws, and the lesson this guide draws with it: **the hardest blocker in a BI transformation is not technical, and it cannot be resolved by engineering.** It is resolved by an adjudicator with the standing to bind the parties, and the transformation's job is to make sure that adjudicator exists and is engaged *before* the conflict is a crisis.

### 14.4 The thesis, restated

Cymbal Bank's transformation is not, in the end, a story about a platform. It is a story about three negotiations: what "active customer" means (settled as a documented pair, not a single answer), what "net fee revenue" means (settled as a prospective definition change, dated and approved), and who has the standing to decide (the executive committee, engaged early because the definitions board was seated from the start). The platform migration was the occasion for all three. **A BI transformation is not a platform migration; it is a negotiation about what a number means.**

---
## 15. The Anti-Patterns and the Claims Audit

### 15.1 The rejected claims

These are claims a reader might expect to find in a guide like this, and that this guide rejects — either because no dated primary source supports them, or because they are mechanisms stated wrongly.

| Rejected claim | Why it is rejected |
|---|---|
| **"Most reports in a bank's estate are unused."** | No dated primary source in this guide's pack measures the proportion; a percentage would be invented. The guide makes the point as an *observation about how estates grow* and how usage is (or is not) established (§3.2), and distinguishes "usage unestablished" from "usage zero." |
| **"A BI transformation typically costs X and takes N months."** | No dated, generalisable source; and a month-count would be an unsourced statistic of exactly the kind this guide bans. Cost and duration are organisation-specific and the guide offers no figure. |
| **"A large fraction of BI programmes fail."** | A frequency statistic with no dated source. The guide names six *mechanisms* of failure (§13) and claims nothing about how often any of them occurs. |
| **"A modern platform makes the definitional problem go away."** | Mechanism error. The platform is where definitions are stored, not where they are decided (§4.7); a new platform with unsettled definitions moves the problem. |
| **"Reconciliation is achieved by setting a tolerance that the difference fits inside."** | Mechanism error, and the cardinal sin of §7.1. Widening a tolerance hides a difference; it does not explain it, which is what reconciliation requires ([BCBS239] footnote 17). |
| **"Reconciliation requires a single authoritative source and therefore a single definition."** | Contradicts [BCBS239] footnote 16: one data model is not required where robust automated reconciliation exists. The guide's §4.5 documented-pair outcome is built on this. |
| **"Self-service removes the need for a central data team."** | Mechanism error. Self-service without governance produces a second estate (§5.3); the central team's role shifts to governing the base, not disappears. |
| **"The migration is complete when the reports run."** | Mechanism error. Reports running on a new platform is the platform milestone, not the transformation's (§6.5); the definitions, reconciliations and retirements are the substance. |
| **"Lift-and-shift is sufficient if the platform is modern."** | Rejected as the §13.1 failure mode: platform moved, definitions untouched, meaning unchanged. |
| **"A retired report cannot be needed again."** | Rejected: the report retired and rebuilt in a spreadsheet (§13.6) is the counterexample; retirement evidence must establish usage as absent, not assume it (§6.2). |

### 15.2 The claims audit

Every sourced claim this guide makes appears below, in the repository's standard mark style. **Verdict** is the mark's outcome; **Kind** is the provenance label declared in the verification posture. "Verified" means the claim is stated in the named source and is quoted or paraphrased accurately; "Attributed belief" means the source states it as a belief or expectation, not a measurement; "Framed" means the claim is this guide's own analytical observation, offered as reasoning rather than as a sourced fact; "Illustrative" means the claim is fiction, labelled as such.

**Table 15.2a — International standard (BCBS 239, January 2013)**

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| A bank should, as a precondition, have a "dictionary" of the concepts used, such that data is defined consistently across an organisation. | Verified (verbatim) | [BCBS239] ¶37 | Jan 2013 | International standard |
| A bank should strive towards a single authoritative source for risk data per each type of risk. | Verified (verbatim) | [BCBS239] ¶36(d) | Jan 2013 | International standard |
| Banks do not necessarily need one data model; there should be robust automated reconciliation procedures where multiple models are in use. | Verified (verbatim) | [BCBS239] footnote 16 | Jan 2013 | International standard |
| The business owner's role includes ensuring data is kept current and aligned with the data definitions. | Verified (paraphrase) | [BCBS239] ¶34 | Jan 2013 | International standard |
| Reconciliation is "the process of comparing items or outcomes and explaining the differences." | Verified (verbatim, quoted) | [BCBS239] footnote 17 | Jan 2013 | International standard |
| IT systems, data and reporting processes require significant investments of financial and human resources with benefits that may only be realised over the long-term. | Verified (verbatim, quoted) | [BCBS239] ¶5 | Jan 2013 | International standard |
| The Committee *believes* the long-term benefits of improved risk data aggregation and risk reporting will outweigh the investment costs. | Attributed belief | [BCBS239] ¶11 | Jan 2013 | International standard |
| A bank should establish integrated data taxonomies and architecture across the banking group, including metadata, single identifiers and unified naming conventions. | Verified (verbatim, quoted) | [BCBS239] P2 ¶33 | Jan 2013 | International standard |
| Risk data should be reconciled with the bank's sources, including accounting data where appropriate. | Verified (verbatim, quoted) | [BCBS239] P3 ¶36(c) | Jan 2013 | International standard |
| Risk management reports should accurately and precisely convey aggregated risk data; reports should be reconciled and validated. | Verified (verbatim, quoted) | [BCBS239] P7 | Jan 2013 | International standard |
| Trade-offs among principles may be accepted in exceptional circumstances; decision-makers should be aware of the trade-offs and limitations; banks should be able to explain their impact. | Verified (paraphrase) | [BCBS239] ¶22 | Jan 2013 | International standard |
| Materiality: information may be excluded only if it would not affect the decision-making process. | Verified (paraphrase) | [BCBS239] ¶23 | Jan 2013 | International standard |
| G-SIBs identified by the FSB in November 2011 or November 2012 must meet the Principles by January 2016. | Verified (verbatim) | [BCBS239] ¶14 | Jan 2013 | International standard |

**Table 15.2b — Accounting standard (IAS 8)**

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| Prior period errors are corrected retrospectively by restating comparatives; changes in accounting estimates are applied prospectively. | Verified (paraphrase of the standard's prescriptions) | [IAS8] — IAS 8 *Accounting Policies, Changes in Accounting Estimates and Errors* (IFRS Foundation) | current standard, retrieved Oct 2026 | Accounting standard |

**Table 15.2c — Vendor documentation (semantic layer)**

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| dbt semantic models define the structure MetricFlow uses to build the semantic graph; they declare entities, dimensions and measures, with metrics declared over measures. | Verified (paraphrase) | [DBT] `docs.getdbt.com/reference/semantic-model-properties` | retrieved Oct 2026 | Vendor doc |
| MetricFlow "compiles these metric definitions into clear, reusable SQL, ensuring consistent and accurate results when analyzing data by relevant attributes (dimensions)." | Verified (quoted) | [DBT] `github.com/dbt-labs/metricflow` | retrieved Oct 2026 | Vendor doc |
| LookML declares data through model files and view files; a view defines fields (dimensions and measures); an Explore adds an existing view to Looker's menu of Explores. | Verified (paraphrase) | [LOOKML] `docs.cloud.google.com/looker/docs/looker-core-qs-model-data` | page updated 9 Oct 2026 | Vendor doc |

**Table 15.2d — Pattern write-ups**

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| The strangler-fig pattern replaces a legacy system incrementally by routing functionality to a new service behind a façade until the legacy can be switched off. | Verified (paraphrase, [STRANGLER]) | Martin Fowler, *Strangler Fig Application*, `martinfowler.com/bliki/StranglerFigApplication.html` (original post 2004; current page dated 22 August 2024); also corroborated in-repo at `technology/monolith_to_microservices_guide.md` §3.1, which dates the pattern to Fowler, 2004 | 2004; page updated 22 Aug 2024 | Pattern write-up |
| Expand-migrate-contract / parallel change runs expand → migrate → contract; the migrate phase is the transition period between old and new until all access is updated; failing to execute contract leaves two estates maintained and the supplier supporting two versions. | Verified (paraphrase, [PARALLEL]) | Danilo Sato, *Parallel Change* (also called expand and contract), `martinfowler.com/bliki/ParallelChange.html`, 13 May 2014; also in-repo `technology/zero_downtime_system_design_guide.md` §4.2 | 13 May 2014 | Pattern write-up |

**Table 15.2e — In-repo cross-references**

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| The analytics stack, maturity model, platform comparison, real-time engines and banking analytics patterns are covered in the series' advanced-analytics guide. | Verified (inventory of that guide's sections) | [REPO] `advanced_analytics_solutions_guide.md` §2–§6, §8, §9–§13 | Jul 2026 (guide's last-updated) | In-repo cross-reference |
| The data-catalog and metadata-management discipline (§10) and the ownership model are the home of the governance applied here. | Verified (inventory of that guide's sections) | [REPO] `data/data_governance_framework.md` §10 | guide in repo | In-repo cross-reference |
| Lineage collection approaches, tooling and anti-patterns are owned by the lineage-tools guide. | Verified (inventory) | [REPO] `data/data_lineage_tools.md` §1–§3, §10 | guide in repo | In-repo cross-reference |
| The operating model, lifecycle, team topologies and hiring are owned by the analytics-management guide. | Verified (inventory) | [REPO] `../management/analytics_management_guide.md` §2, §5 | guide in repo | In-repo cross-reference |
| The BCBS 239 discipline — principles, layers, implementation history — is owned by the risk-data-aggregation guide. | Verified (inventory) | [REPO] `../banking/risk_data_aggregation_guide.md` | guide in repo | In-repo cross-reference |
| The business-case construction method is owned by the business-case-development guide. | Verified (inventory) | [REPO] `../management/business_case_development_guide.md` | guide in repo | In-repo cross-reference |

**Table 15.2f — This guide's own analytical claims (framed, not sourced)**

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| Estates grow by accretion because addition is cheap and removal requires evidence nobody has. | Framed | This guide §3.1–§3.2 (observation about mechanism) | Oct 2026 | In-repo cross-reference (this guide) |
| Some reports cannot be explained, so they can be reproduced but not tested. | Framed | This guide §3.3, §6.5 | Oct 2026 | — |
| Self-service without governance produces a second estate. | Framed | This guide §5.3 | Oct 2026 | — |
| A number that changes without explanation is an incident in a regulated institution. | Framed | This guide §6.6 | Oct 2026 | — |
| A savings figure nobody will measure afterwards is a permission slip rather than a plan. | Framed | This guide §11.3 (judgement) | Oct 2026 | — |
| The Cymbal Bank episodes (active-customer pair, net-fee-revenue reconciliation, cards-function escalation) and all their figures. | Illustrative | This guide §14 (fiction, labelled) | Oct 2026 | In-repo cross-reference (this guide) |

---
## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

This section records, honestly, what this guide could not establish — and distinguishes a *tool limitation* from an *absence of material*.

| Item | Status | What would resolve it |
|---|---|---|
| **Vendor semantic-layer and BI-migration documentation beyond the two referenced pages.** | Partially verified. The dbt semantic-model/MetricFlow pages and the Looker LookML quickstart extracted cleanly, but several vendor BI-migration and semantic-layer pages are JavaScript-rendered and would not extract through the retrieval tool used here. The semantic-layer vocabulary in §4.2 rests on the two pages that *did* extract. | Extracting those pages via a browser-rendered retrieval path, or reading the vendor documentation directly. |
| **Search retrieval of the strangler-fig and expand/migrate/contract pattern write-ups.** | **Search-tool limitation; the primary pages themselves were retrieved.** The `web_search` tool returned an empty result set for multiple queries on these patterns (including "Strangler Fig pattern legacy system migration" and "expand migrate contract database refactoring"), despite the pages being live — recorded as a search-tool limitation, **not** as absence of material. The two write-ups were then retrieved directly by URL: Martin Fowler, *Strangler Fig Application* (`martinfowler.com/bliki/StranglerFigApplication.html`; original post 2004, page dated 22 August 2024) and Danilo Sato, *Parallel Change* (`martinfowler.com/bliki/ParallelChange.html`, 13 May 2014). Both are also corroborated in-repo — `technology/monolith_to_microservices_guide.md` §3.1 for the strangler fig, `technology/zero_downtime_system_design_guide.md` §4.2 for parallel change. | Nothing further is required for the two patterns; a working `web_search` would restore the search path for any future claim. |
| **A dated primary source for any usage, cost, ROI or savings statistic.** | **Not available, and therefore not used.** No such statistic appears in this guide. Every point that would conventionally be made as a percentage is made instead as an observation about mechanism (§3.2, §15.1). | A dated, sourced study measuring report usage, migration cost, or BI ROI in a comparable institution. |
| **National implementation of the BCBS 239 principles.** | Deliberately not asserted. §12.1 cites the international standard and its January 2016 G-SIB deadline ([BCBS239] ¶14) and explicitly declines to assert any national rule, national deadline, or national supervisory expectation. | The relevant supervisor's own published instruments for a named jurisdiction. |
| **Attribution precision on the assessment of how often the failure modes of §13 occur.** | Unpinned. The guide names six *mechanisms* of failure and claims no frequency for any of them; the two places that previously read as frequency claims ("most common failure") were rewritten as judgements of consequence, not counts. | A dated survey measuring the incidence of each failure mechanism. |

The guide's posture is that a limitation recorded is worth more than a statistic invented. Where the retrieval tool returned nothing, this guide says so and routes around it; it does not fill the gap with a plausible number.

### 16.2 Glossary

| Term | Meaning in this guide |
|---|---|
| **Business intelligence (BI)** | The reporting-and-analysis layer that answers "what happened" — reports, dashboards, ad-hoc queries and their governance. |
| **BI transformation** | A programme that changes both the platform on which numbers are produced and the definitions by which they are understood. |
| **Calculation** | The arithmetic that implements a definition (tables, filters, joins, aggregations). An engineering artefact. |
| **Catalogue** | The indexed inventory of data artefacts with owners, definitions, classifications and relationships. |
| **Definition** | An organisational fact — who is a customer, what counts as revenue — owned by the business, not derived from the database. |
| **Definition authority / definitions board** | The standing body of domain data owners empowered to adjudicate contested definitions. |
| **Documented competing definitions** | The lawful outcome where two definitions are both correct for their own question: a definition pair plus a stated, owned reconciliation ([BCBS239] footnote 16). |
| **Governed metric** | A metric with a named owner, a stated definition, a version, a stated reconciliation, and a change log. |
| **Lakehouse** | Object storage with warehouse-style management over it — the lake's economics with the warehouse's discipline. |
| **Lineage** | The traced path from a reported number back through the transformations to the source record. |
| **Parallel run** | Running the legacy and new computations side by side and explaining every difference before trusting the new one. |
| **Reconciliation** | "The process of comparing items or outcomes and explaining the differences" ([BCBS239] footnote 17). |
| **Report estate** | The full population of produced artefacts, including the ones nobody can explain. |
| **Restatement** | A change to a previously reported number that looks backwards, correcting a prior period ([IAS8]). |
| **Semantic layer** | The repository of declared business logic (metrics, dimensions, entities, measures) that compiles to SQL so one definition yields one number. |
| **Self-service** | The capability for business users to build views over governed data without a central-team request. |
| **Shadow spreadsheet estate** | The spreadsheets interposed between warehouse output and decision, holding the real business logic outside governance. |
| **Strangler pattern** | Incremental replacement of a legacy system by routing functionality to a new system behind a façade. |
| **Tolerance** | A materiality threshold for a difference; set by the business owner, not the platform team. |

### 16.3 Cross-References

**Repository guides cited but not restated** (the `data/` shelf is referenced as `data/x.md`; `management/` and `banking/` as `../management/x.md` and `../banking/x.md`; siblings as plain filenames):

- [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md) — the analytics stack (§3), modern data stack (§4), platform comparison (§5), real-time engines (§6), decision matrix (§8), banking analytics patterns (§9–§13), maturity model (§2).
- [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) — the lakehouse destination.
- [ensemble_logical_model_elm.md](ensemble_logical_model_elm.md) — the business-to-technology modelling bridge that structures the glossary.
- [data/enterprise_data_platforms_guide.md](data/enterprise_data_platforms_guide.md) — the enterprise data-platform architecture as a category.
- [data/on_prem_lakehouse_guide.md](data/on_prem_lakehouse_guide.md) and [data/on_prem_lakehouse_analysis.md](data/on_prem_lakehouse_analysis.md) — the on-premise lakehouse destination.
- [data/data_governance_framework.md](data/data_governance_framework.md) — §10, the data-catalog and metadata-management discipline; the ownership model.
- [data/data_lineage_tools.md](data/data_lineage_tools.md) — lineage collection approaches (§3), tooling (§4–§6), anti-patterns (§10).
- [data/data_fabric_guide.md](data/data_fabric_guide.md) — the federated-governance angle for a multi-platform estate.
- [../management/analytics_management_guide.md](../management/analytics_management_guide.md) — operating models, lifecycle, team topologies, hiring (§2, §5).
- [../management/business_case_development_guide.md](../management/business_case_development_guide.md) — the business-case construction method.
- [../banking/risk_data_aggregation_guide.md](../banking/risk_data_aggregation_guide.md) — the full BCBS 239 discipline; not re-derived here.

**Primary sources retrieved October 2026:**

- **[BCBS239]** Basel Committee on Banking Supervision, *Principles for effective risk data aggregation and risk reporting*, January 2013 (BIS; ISBN 92-9131-913-9) — ¶5, ¶11, ¶14, ¶22, ¶23, ¶33, ¶34, ¶36(c), ¶36(d), ¶37; footnotes 16 and 17; Principles 2, 3 and 7.
- **[IAS8]** IAS 8 *Accounting Policies, Changes in Accounting Estimates and Errors*, IFRS Foundation.
- **[DBT]** dbt Developer Hub, *Semantic model properties* (`docs.getdbt.com/reference/semantic-model-properties`) and *MetricFlow* (`github.com/dbt-labs/metricflow`).
- **[LOOKML]** Google Cloud Looker documentation, *Quickstart: Model your data in LookML* (`docs.cloud.google.com/looker/docs/looker-core-qs-model-data`, page updated 9 October 2026) and the *explore* parameter reference.
- **[STRANGLER]** Martin Fowler, *Strangler Fig Application*, `martinfowler.com/bliki/StranglerFigApplication.html` (original post 2004; page dated 22 August 2024). **[PARALLEL]** Danilo Sato, *Parallel Change* (also called expand and contract), `martinfowler.com/bliki/ParallelChange.html`, 13 May 2014. See §16.1 for the search-tool limitation and the in-repo corroboration.

### 16.4 Closing Summary

A BI transformation is the occasion on which an organisation is forced to say what its numbers mean, and most organisations discover in the process that they never formally said. The guide has argued this in one direction and then the other: from the estate, which grows by accretion and holds reports nobody can explain and spreadsheets that hold the real logic; through the semantic layer, which is the only place a definition can live as a first-class artefact; through the definition-versus-calculation distinction, which assigns the definition to the business and the arithmetic to engineering; through the definitional conflicts a bank actually has, and the honest permission — **[BCBS239] footnote 16** — to ship documented competing definitions with a stated reconciliation rather than force a false single answer; through self-service, whose ungoverned form is the next estate; through the migration path and the reconciliation discipline, in which the hardest item is the report nobody can explain and a number that moves without explanation is an incident; through the destination, mapped not restated; through the organisation, the governance and lineage that make the transformation reversible, the business case and the parts of it that are usually unvalidated, and the regulated-institution angle in which a restatement is a governance event and not a refresh; through six failure modes and a fully labelled Cymbal Bank worked example in which the blocker was political rather than technical; and finally through the audit in which every sourced claim is marked, every rejected claim is rejected for a stated reason, and every limitation is recorded rather than papered over.

The through-line is the thesis, and it bears restating as the guide closes. The platform will move; the platform always moves. What determines whether the organisation is better off afterwards is whether the definitions moved with it — surfaced, adjudicated or explicitly paired, owned, versioned, reconciled, and dated — because the platform is the place the numbers live and the definitions are the thing the numbers mean. When the two move together, the transformation is real and reversible. When only the platform moves, the organisation has paid for the appearance of a transformation and kept the substance of the problem, and it will meet the problem again at the next renewal.

**A BI transformation is not a platform migration; it is a negotiation about what a number means.**
