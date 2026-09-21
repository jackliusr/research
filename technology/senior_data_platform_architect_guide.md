# Senior Data Platform Architect — the platform is the product

> **For:** Jack Liu Shurui — Solution Architect (GenAI), Cymbal Bank, Singapore
> **Analysis Date:** September 2026
> **Source Material:** github.com/jackliusr/research — the personalized role-analysis series (banking/, technology/, management/, personal/) plus primary external sources listed in §15
> **Target Role:** Senior Data Platform Architect — the role that owns a data platform **as a product**, not the pipelines and not the models
> **Predecessors in this series:** data_architect_skillgaps_guide.md ; architecture/skill_gaps_enterprise_architect_guide.md

**Contents**

1. How this guide relates to the repository, the target role and the decoder
2. What the role owns, and what it does not
3. The skill framework for this role — technical competencies, cross-referenced
4. The architectural competencies — the part that is not tooling
5. The organisational competencies
6. What "senior" actually means here
7. The market and the ladder
8. The routes in and the transition
9. The interview and the assessment
10. The failure modes of the role
11. The regulated-enterprise and banking angle
12. The AI-era effect on the role, evidence-graded
13. The Cymbal Bank worked example (fictional, illustrative)
14. One-page summary and action list
15. The claims audit
16. What Could Not Be Verified, the glossary, the cross-references and the closing

---

## 1. How this guide relates to the repository, the target role and the decoder

### 1.1 The series convention, and what this guide does that its predecessors do not

This is the third guide in the repository's personalized role-analysis series. It follows the convention set by `data_architect_skillgaps_guide.md` (715 lines, August 2026) and `architecture/skill_gaps_enterprise_architect_guide.md` (316 lines, July 2026): a blockquote header addressed to the subject, a numbered table of contents, an explicit statement of source material, a glossary, a verification record, and a closing note that says where the repository is silent. It is written to be read alongside them, not instead of them.

Two things here are genuinely new:

**(i) A DIFFERENT TARGET ROLE.** The first predecessor analyses the transition Solution Architect → **Data Architect**: a role defined by data modelling, data architecture patterns, integration and governance, assessed across ten axes in its §1. The second analyses Solution Architect → **Enterprise Architect**: a role defined by framework literacy (TOGAF, BIAN, capability engineering) and enterprise-wide influence. This guide analyses a third and distinct role: the **data platform architect**, who owns neither the enterprise's data models (the data architect's object) nor the enterprise's target-state architecture (the enterprise architect's object), but the **platform** — the shared, self-service system of ingestion, storage, compute, orchestration, governance hooks, access control and developer experience on which everyone else builds. That distinction is the whole point of §1.3 and the decoder in §1.4.

**(ii) THE SENIORITY DIMENSION.** Neither predecessor treats seniority as a variable. The first scores the *axes of the role* on an L1–L5 proficiency rubric (§1.11 of that guide) and estimates readiness against a target profile; the second rates *evidence depth* per skill area ("Advanced", "Solid", "Intermediate") in its §2.1. Both answer "how much do you know relative to the role", and both collapse their answer into a single percentage — 70–75% and 65–70% respectively. Neither answers "what actually changes between a mid-level and a senior practitioner in this role, and what does not". That question is §6 of this guide, and it drives §9 (what distinguishes a senior answer in an interview) and §13 (which candidate Cymbal Bank should appoint). A readiness percentage with no seniority model is a number without a scale; this guide supplies the scale.

Everything else is deliberately built on the predecessors rather than re-derived. Where the first guide's assessment axes apply, this guide names them and points at them. Where the second guide's banking-domain gap list applies to a platform architect, this guide names it and adds only what the platform perspective changes. No new heatmap, no new top-10 gap ranking, no new learning plan: those artefacts exist, they are dated, and duplicating them would produce two inconsistent versions of the same assessment.

### 1.2 The one-paragraph definition

**A Senior Data Platform Architect owns the architecture and the evolution of a shared data platform considered as a product with internal customers.** They decide what capabilities the platform offers, in what layers, at what interfaces, with what guarantees, at what cost, under what access model, and how all of that changes over years without breaking the consumers who depend on it. They are accountable for the platform being *usable, affordable, reliable and extensible*, not for the business meaning of the data flowing through it. Practically: they own the ingestion, storage, compute, orchestration, cataloguing, access-control and developer-experience layers **as one coherent system**, they own the contracts the platform publishes to its consumers, and they own the consequences — including cost and outage consequences — of the choices they make. What they do not own is the business data model, the analytics products, the machine-learning models, or the source systems that feed them.

The role is distinct because of one organising idea, and it is worth stating plainly because it is what separates a platform architect from an architect who happens to work on data systems: **the platform's consumers are the platform's customers, and the platform has to earn their use.** A pipeline architect can succeed by delivering pipelines that work. A platform architect fails if the pipelines they enable are technically excellent and nobody uses the platform. That is the difference between owning an output and owning a product.

### 1.3 The distinction that gives the role its identity

Three claims define the boundary, and each one is stated against a role the market conflates with this one.

1. **Not the pipelines.** The data platform architect designs the *conditions* under which pipeline authors work — the execution model, the storage layers, the table format, the orchestration primitives, the schema-evolution policy, the secrets and access model, the backfill and replay semantics, the observability primitives — and then delegates pipeline implementation to data engineering teams. The resource the architect manages is *other people's ability to ship data reliably*. Where a specific pipeline is the only consumer of a capability, the capability is not yet a platform capability; it is that team's application code, and it belongs in their repo.
2. **Not the models (either kind).** Business data models, canonical models, dimensional models, Data Vault hubs and satellites: the object of the data architect, assessed at length in the predecessor's §1.1. Machine-learning models, feature definitions and evaluation data: the object of ML engineering and model risk management. The platform architect provides the substrate — the training-data pipeline, the feature-store storage, the lineage hooks the model-risk function needs — and does not own what is expressed on top of it.
3. **Not the enterprise target state.** The enterprise architect owns the multi-year target architecture and the standards portfolio across all domains. The data platform architect owns one domain (data) and one artefact class (the platform). The relationship is the classic one: the enterprise architect sets constraints — residency, technology standards, risk appetite — and the data platform architect produces a working artefact that satisfies them. The repository's `architecture/why_tech_bypasses_togaf_guide.md` addresses precisely the failure mode of this relationship — reference architectures that look authoritative and are quietly routed around — and it is required reading for this section. Which direction the relationship runs in practice depends entirely on whether the platform architect ships something people want; that is §10(ii) and §10(iii).

### 1.4 The decoder: eight titles the market conflates

Method note: the distinctions below are drawn from primary role definitions and published job descriptions, not from assumption. The sources are named in the "evidence in this guide's sources" column, and the whole set is audited in §15. The single most important honesty note in this guide sits directly beneath the table.

| Title | Primary object of ownership | Typical deliverable | Whose decisions it constrains | Failure mode specific to it | Where it genuinely overlaps |
|---|---|---|---|---|---|
| **Data platform architect** | The platform as a coherent product (layers, interfaces, guarantees, cost, access model) | Platform capability roadmap; working reference implementation; interface contracts; NFR budget | Data engineers, analytics engineers, data scientists, application teams consuming the platform | Design delivered as a document; platform built for its builders rather than its consumers (§10 i, iii) | With platform engineering (who builds it), data architect (storage/model boundary), enterprise architect (standards) |
| **Data architect** | The organisation's data landscape and models — "sets the vision for the organisation's use of data, through data design" (UK Government Digital and Data Profession Capability Framework, data architect role, published 7 Jan 2020, updated 29 May 2026) | Data models, data dictionaries, data standards, model-to-physical mapping | Model implementations; source-system owners; downstream warehouses | The model that is correct and unadopted; governance by document | With data platform architect on the storage layer and the semantic/contract boundary |
| **Platform engineer** | Build and run of platform components | Working services, golden paths, templates, IaC, on-call | How application/data teams deploy and operate | Over-abstraction; a platform that solves the builders' problems | With data platform architect: in small organisations the two are one job |
| **Data engineer** | Pipelines and data movement | Reliable pipelines, landed and transformed datasets, SLAs | Data availability and freshness for consumers | Pipeline sprawl; undocumented lineage; per-team reinvention of the same ingestion | With analytics engineer at the transformation boundary; with platform architect at the ingestion framework |
| **Analytics engineer** | Modelled, tested, documented, analytics-ready datasets and metrics | Transformation models, metric definitions, tests, docs | How business users query and interpret data | Metric divergence; semantic drift between teams | With data architect on modelling standards; with platform on the transformation runtime |
| **Enterprise architect** | Enterprise-wide target architecture, standards and roadmaps — "connects an organisation's business mission, methodology and processes to its IT strategy" (The Open Group, Open Certified Architect programme description) | Reference architectures, standards, capability roadmaps | Everyone, in principle | Reference architecture nobody follows |
| **Solutions architect** | The technical design of a bounded solution — "designing, describing, and managing the solution engineering in relation to specific business problems" (The Open Group, same source) | Solution designs, ADRs, integration designs | One programme or system | Point solutions that do not compose into a platform |
| **Data product owner** | The outcomes and consumer value of one data product | Prioritised backlog, SLAs, contracts, consumer map | One product team | A product with no platform underneath it; SLAs nobody can enforce |

**Evidence for the distinctions, from primary sources.** The UK government's published framework separates the data architect explicitly from the technical architect, requiring the senior data architect to "work with technical architects to make sure that an organisation's systems are designed in accordance with the appropriate data architecture" — a division of labour, not a synonym. Accenture's published requisition titled **Data Platform Architect** (requisition ATCI-5701048-S2061816, Pune, 5–10 years' experience band, accessed September 2026) defines the role as "Architects the data platform blueprint and implements the design, encompassing the relevant data platform components" and states that the incumbent "collaborates with the Integration Architects and Data Architects to ensure cohesive integration between systems and data models" — one posting, three architect titles, each with a different object. That is the clearest primary evidence available that the market does separate these roles in writing.

**The honest note on title inflation.** Title inflation here is endemic, and the same Accenture posting is the evidence. Its *title* is Data Platform Architect; its *body* describes a hands-on engineer working "on moderately complex platform components under guidance from senior architects and engineers", with a hard floor of "Minimum 2 years of experience coding, building, monitoring or troubleshooting AI/ML infrastructure, data platforms, model deployment pipelines or cloud/platform engineering solutions". A genuinely senior platform architect would be setting the design the role is described as implementing. Meanwhile, a widely circulated "Senior Data Platform Engineer role blueprint" (a content-site role template, not an employer, therefore not usable as evidence of what anyone hires for) describes scope indistinguishable from what other firms call a data platform architect. The conclusion is not that the titles are meaningless; it is that **the titles are not portable and the work is**. Where the titles are genuinely interchangeable: (a) inside single-platform organisations with one data platform team, where data platform architect, senior data platform engineer and lead data engineer are frequently the same person with three interchangeable labels; (b) inside consultancies and system integrators, where architecture titles are attached to the engagement's project role rather than to a permanent scope, exactly as in the Accenture example; (c) at mid-size firms where a "data architect" is expected to also run platform engineering, own cost, and maintain the ingestion framework. Where they are not interchangeable: in any organisation large enough to have separate data modelling, platform, and analytics-engineering functions, and in regulated enterprises where the architect of record carries an audit and sign-off obligation that a platform engineer does not.

---

## 2. What the role owns, and what it does not

### 2.1 The ownership map

| Owned | What "owning" means concretely | Artefact that proves ownership | Whose work it constrains |
|---|---|---|---|
| **The platform's architecture and its evolution** | A stated target platform architecture, a capability roadmap, and an explicit position on what the platform will *not* do this year | Target platform architecture; capability roadmap; ADRs | Every team building on or inside the platform |
| **Ingestion, storage, compute and serving layers as one system** | Coherent layer design with defined boundaries and interfaces; decisions about table formats, partitioning, compaction, tiering, engine topology | Layer diagram with interfaces; storage-layout standards; engine selection records | Data engineers implementing ingestion and transformation |
| **The interfaces and contracts the platform offers** | Versioned, documented, testable interfaces: ingestion APIs, catalog APIs, data contracts, event schemas, access-control request patterns | Contract specifications and their versions; deprecation policy | Producers and consumers on both sides of every contract |
| **Reliability and performance of the platform as a service** | Published SLOs for platform capabilities; capacity plan; failure domains and blast-radius design; incident ownership for platform-caused outages | SLOs; capacity plan; runbooks; post-incident reviews | Platform engineering; every consumer relying on an SLO |
| **Cost** | A cost model per capability, unit-economics visibility for consumers, and defended trade-offs between cost and performance | Cost-per-capability model; consumption reporting; cost guardrails | Consumers' query and storage patterns; finance and procurement |
| **Security and the access model** | The platform's authentication, authorisation, masking, and audit model — who can see what, by what mechanism, provable after the fact | Access model design; classification-to-control mapping; audit evidence | Every consumer's ability to use data; the security function's control set |
| **The developer and analyst experience** | Time-to-first-value for a new consumer; self-service paths; documentation; template quality | Onboarding path; golden paths; template library; satisfaction measure | Data engineers, analytics engineers, analysts, data scientists |
| **The platform's own observability** | Platform-level telemetry distinct from per-pipeline telemetry: saturation, queueing, error budgets, cost anomalies | Platform dashboards; alert definitions; SLO reporting | Platform engineering; incident response |

### 2.2 What the role does not own

- **The business data model.** Entity definitions, canonical customer/product/account models, the meaning of a "transaction" in the business sense: the data architect's and the business's object. The platform architect owns the *mechanism* by which models are stored, versioned and evolved — not their content. The exception, stated honestly: where no data architect exists, the platform architect will be handed this, and should negotiate it as scope creep rather than absorb it silently (§2.4).
- **Analytics products, dashboards, metrics definitions, reporting.** The analytics engineering and BI functions' object. The platform architect owns the runtime those products execute on and the metric-storage layer, not the metrics.
- **Machine-learning models, feature definitions, model validation.** The ML and model-risk functions' object. The platform provides training-data pipelines, feature storage, lineage, and the serving substrate — and is accountable for the platform's share of the model-risk evidence, not for the model.
- **Source systems.** Core banking, CRM, ERP, trading systems, the ledger. The platform architect specifies what the platform needs from them (extraction interfaces, change-data-capture availability, schema-change notice periods) and negotiates it — and does not own the source. This is one of the most reliable sources of friction in the role: the platform's quality ceiling is set by sources it does not control.
- **The data governance operating model.** Policy, ownership assignment, stewardship, classification standards: the data governance function's object, covered by `data_governance_guide.md` and `data/data_governance_framework.md`. The platform architect owns the *enforcement mechanism* — where policy becomes a technical control rather than a document.

### 2.3 The boundary is political as much as technical

Every line in §2.1 and §2.2 is drawn somewhere, and the drawing is contested. Three patterns recur:

1. **Scope creep inward.** Source systems are slow to change, so the platform absorbs the work: reconciliation logic that should live in the source, business rules that should live in a model, report-specific aggregations that should live in an analytics product. Each individual absorption is defensible and each one, accumulated, converts the platform into the enterprise's undocumented application server. The guardrail is a written boundary with a named owner on the other side, revisited on a schedule, not per crisis.
2. **Ownership claimed without authority.** The architect is "accountable" for platform cost while procurement, cloud contracts and licence renewals sit elsewhere; or accountable for reliability while headcount and on-call sit with an engineering manager who reports into a different chain. Accountability without control is the most common structural defect in this role, and identifying it during the hiring process is a senior skill (§9).
3. **The vacuum fill.** Where no data architect exists, the platform architect is the only person who understands the storage layer, and the modelling work arrives by default. That can be the right outcome — but it should be a negotiated, titled, resourced change, not a silent drift.

Good practitioners negotiate this boundary by conceding the *technical* argument and winning the *decision-rights* argument: they accept that the platform will do the work for now, and trade that for a documented decision right, a named counterpart, or budget. They also make the boundary visible in the platform's own artefacts — an ADR that records "aggregation logic for report X is deliberately implemented in the analytics layer, not the platform" is worth more than a boundary slide, because it survives the architect's departure.

---

## 3. The skill framework for this role — technical competencies, cross-referenced

This section is an index, not a curriculum. The repository already contains the substance for every row below; the work here is naming **what the role needs from each competency** and **where the substance lives**, so the reader's effort goes into the delta rather than the overlap. Where a competency has no dedicated guide in this repository, this guide says so and cites the finding rather than implying coverage that does not exist.

| Competency | What the platform architect needs from it | Where the substance lives in this repository |
|---|---|---|
| **Distributed-systems fundamentals** | The frame for every other decision: consistency models, CAP trade-offs, replication, partition tolerance, idempotency, backpressure, failure domains, quorum and consensus reasoning. Without this, engine selection is preference | `distributed_systems_engineering_guide.md` |
| **Storage and query engines** | Engine topology and selection reasoning: MPP vs. columnar vs. row-store vs. search vs. vector; concurrency and workload isolation; partitioning and clustering; compaction and vacuum; the join and aggregation cost model | `data/mpp_databases_guide.md`; `data/enterprise_data_platforms_guide.md`; `databricks_guide.md`; `clickhouse_guide.md`; `data/on_prem_lakehouse_guide.md`; `data/on_prem_lakehouse_analysis.md` |
| **The search tier** | When a data platform needs a separate search/retrieval tier rather than forcing warehouse queries, and how to size it. The repository's capacity-planning treatment is the part most architects lack | `opensearch_capacity_planning_guide.md`; `data/elasticsearch_data_modeling_schema_design.md` |
| **Streaming and messaging** | Stream-vs-batch boundary placement; event time, watermarks, late data; state and exactly-once reasoning; schema evolution on streams; the operational layer (offsets, lag, rebalancing, DLQs) | `event_stream_processing_guide.md`; `apache_flink_guide.md`; `kafka_alternatives_guide.md`; `kafka_virtualization_guide.md`; `message-queue` and CDC material inside `data_integration_frameworks_guide.md` |
| **Table formats and catalogs** | Open table format choice (Delta / Iceberg / Hudi / Paimon) and its interlock with the catalog, the engine set, and the migration path. This is a platform-architect decision, not a team-level one, because reversing it is expensive | `data/delta_lake_vs_iceberg.md`; `data/paimon_iceberg_delta_comparison.md`; `data/on_prem_lakehouse_guide.md` |
| **Orchestration and pipeline tooling** | The execution *model* the platform offers pipeline authors, not one tool: scheduling semantics, dependencies across systems, retry and backfill semantics, versioning of pipelines as artefacts, and the operational contract with pipeline owners | `apache_airflow_guide.md`; `data/data_pipeline_guide.md`; `data/data_pipeline_versioning.md`; `data/dataops_guide.md`; `data/backfill_data_engineering.md` |
| **Data correctness mechanics** | Duplicate-key handling, late-arriving data, idempotent writes, slowly-changing dimensions as a platform concern (the platform must make the *correct* pattern the easy one) | `data/handling_duplicate_keys_data_warehousing.md`; `data/types_of_dimensions_data_warehousing.md`; `data/crm_data_warehouse_modelling.md`; `data/data_vault_2_modeling.md` (for the modelling patterns the platform must support) |
| **Schema evolution and drift** | Compatibility policy as a platform guarantee: what the platform promises producers and consumers when schemas change, and how it detects drift before consumers do | `schema_evolution_data_drift_guide.md` |
| **Integration and interfaces** | The interface inventory the platform publishes, and the ingestion patterns it standardises (batch file, API, CDC, event) so that consumers stop inventing their own | `data_integration_frameworks_guide.md`; `api_governance_guide.md` |
| **Governance, lineage, quality and profiling** | Turning policy into platform mechanism: lineage capture as a platform feature, profiling as a platform service, catalog integration as a platform interface | `data_governance_guide.md`; `data/data_governance_framework.md`; `data/data_lineage_tools.md`; `data/data_profiling_guide.md` |
| **Platform engineering and Kubernetes** | The runtime the platform runs on: container platform selection, multi-tenancy, operator patterns, admission policy, GitOps, the boundary between infrastructure and data platform teams | `charmed_kubernetes_vs_openshift_guide.md`; `ai_platform_engineering_guide.md` |
| **Cost and capacity** | The unit economics of the platform, the capacity model behind the SLO, and the ability to defend both to finance | `finops_guide.md`; `capacity_sizing_guide.md`; `opensearch_capacity_planning_guide.md` |
| **Architecture decision practice** | Recording and defending expensive-to-reverse decisions | `architecture_decision_record_guide.md` |
| **Cloud, residency and sovereignty** | Region and residency constraints as an input to topology; hybrid and multi-cloud data placement | `data/enterprise_data_platforms_guide.md`; `data/data_compliance_frameworks.md`; `data/china_data_governance_frameworks.md`; `data/alibaba_data_platform.md` |

**Three honest gaps, carried forward from the predecessor.** The first predecessor's scan (§0 of `data_architect_skillgaps_guide.md`) established that this repository has **no dedicated guide for data contracts**, **no dedicated guide for data quality / observability / cataloguing platforms**, and **no dedicated guide for managed cloud data warehouses** (Snowflake / BigQuery / Redshift / Synapse appear in comparisons but own no guide). All three matter disproportionately to a platform architect: contracts *are* the platform's product surface; quality and observability are what makes a platform a service rather than a set of services; and the managed-cloud warehouse is the layer the platform will most likely be built on. The predecessor proposed commission filenames for exactly these (`data_quality_observability_guide.md`, `cloud_data_warehouse_platforms_guide.md`) and they are not in the tree. Reproducing that finding honestly is worth more here than a pretence of coverage — a platform architect who cannot name what their own repository lacks will not name it in an architecture review either.

**What this competency index is not.** It is not a claim that the platform architect needs expert depth in every row. The role needs *design-level* competence across all rows and *expert* depth in the rows that carry its expensive-to-reverse decisions: storage layout and table format, engine topology, the contract surface, and cost. Distributing attention evenly across the table is itself a failure mode (§10(v)).

---

## 4. The architectural competencies — the part that is not tooling

### 4.1 Trade-off reasoning under incomplete information

Every consequential platform decision is made with partial evidence: benchmarks that do not match your workload, vendor claims, projections from one team's experience. The competence is not "choosing correctly"; it is *making the reasoning auditable* — stating the assumptions, naming the evidence class, defining what would change the decision, and recording all four. The repository's interview guides build this muscle under time pressure (`system_design_interview_insiders_guide.md`, `google_system_design_interview_guide.md`, `nalsd_system_design_guide.md`, `grokking_system_design_companion_guide.md`); the platform architect needs the same skill under *years-long* time pressure, where the cost of a wrong assumption compounds instead of ending the interview.

### 4.2 Designing for non-functional requirements

Platforms are judged on non-functional properties, and the platform architect is the person who converts vague adjectives into numbers someone can be held to: freshness in minutes, availability with an error budget, cost per terabyte per month, access latency at the 95th percentile, recovery time and recovery point per capability. The discipline is to attach each number to a capability, a consumer, and a measurement, and to publish the ones the platform does not yet meet. An unmeasured NFR is a marketing claim; a published unmet NFR is a roadmap.

### 4.3 Interface and contract discipline

The platform's product surface is its interfaces. Contract discipline means: versioned interfaces; explicit compatibility promises; a deprecation policy with dates; a mechanism that detects a consumer breaking a contract (not a policy asking them not to); and a rule that the contract is stated in the consumer's terms rather than the platform team's internals. The repository has strong material on the *semantics* of contract-like guarantees — `schema_evolution_data_drift_guide.md`, `data/data_pipeline_versioning.md`, `api_governance_guide.md`, and the CDC and idempotency sections of `data_integration_frameworks_guide.md` — but **no dedicated data-contract guide**, as the predecessor's §0 records. Treat the contract material that exists as the substrate and the missing guide as a known hole.

### 4.4 Designing for evolution rather than replacement

The most valuable architectural property of a platform is not scalability; it is *changeability without a rebuild*. A platform that must be replaced every four years is a platform with execution risk on a schedule. Concretely, designing for evolution means: keeping the storage layout and the query engine separable; keeping the contract surface independent of the implementation behind it; preferring additive versioning over in-place mutation; deciding which layers are allowed to be thin and replaceable and which are load-bearing for a decade; and writing the migration path into the design at the moment of the design, not at the moment of the migration. The repository's `data/on_prem_lakehouse_guide.md` and the table-format comparisons are where the concrete forms of this argument live for the storage layer.

### 4.5 The reference architecture versus the working architecture

This is the competence that most clearly separates a platform architect from an architect of documents. A reference architecture is a statement of intent; a working architecture is the thing people actually use, and the two diverge wherever the reference architecture requires effort the consumer organisation has no incentive to spend. The repository addresses this catastrophe-by-default directly in `architecture/why_tech_bypasses_togaf_guide.md`, which is required reading here: it explains why technically sound architectures are routed around, and therefore what a platform architect must do differently — ship the working path, make the compliant path the cheapest path, and measure adoption rather than conformance.

### 4.6 The decision record, and the decisions that are expensive to reverse

`architecture_decision_record_guide.md` provides the form. The platform-specific discipline is to sort decisions by *reversal cost* and to spend the architect's scarce political capital only on the expensive end:

- **Very expensive to reverse:** the storage layout and file/table format; the catalog and metadata store; the identity and access model; the residency topology; the event-bus and its schema registry; the tenancy model.
- **Moderately expensive:** the orchestration execution model; the transformation runtime; the search tier's existence; the cost-allocation model.
- **Cheap:** engine versions, dashboard tooling, individual pipeline frameworks, library choices.

A platform architect who treats the cheap tier as a decision to win is burning credibility; one who never decides the expensive tier has not done the job. The concrete instrument is to require, for every expensive-tier decision, that the ADR state the migration cost of being wrong — a number that turns a design argument into a cost argument and often ends it.

### 4.7 Which of these are teachable, and which are earned

Teach-able, and genuinely taught by material in this repository: the trade-off vocabulary (§4.1), NFR specification (§4.2), contract and ADR mechanics (§4.3, §4.6). Earned, and not transferable by reading: the calibration of *how much* evidence is enough to decide (§4.1) — this comes from having been wrong at scale; the judgement of which layers are allowed to be thin (§4.4), which comes from having owned a platform through a technology generation change; and the political instinct in §4.5, which comes from having watched a technically correct design be ignored. The practical implication for a reader preparing for this role is that reading closes roughly half the gap, and the other half is only obtainable by owning something in production, with the cost and outage consequences attached. That is the argument for the transition route in §8 rather than the study route.

---

## 5. The organisational competencies

Technical competence gets a platform architect hired; organisational competence decides whether the platform exists in three years. This guide does not re-teach the material — the repository's management shelf already covers it at length, and two guides are load-bearing here.

**Influence without authority.** The platform architect's decisions constrain people who do not report to them and who can choose, quietly, not to adopt. That is the definition of an influence problem, and `../management/authority_skills_guide.md` is the repository's treatment of it. What is specific to this role: the architect's influence is *manufactured by the platform's usefulness* in a way a manager's is not. A manager's authority survives a bad decision; a platform architect's authority does not survive an unusable platform, because the escape route (build your own) is always available and cheap in the short run. This is why §10(iii) is the most dangerous failure mode in the role.

**Stakeholder management across five constituencies with incompatible incentives.** The platform architect is accountable to: data engineering (wants primitives that do not get in the way), analytics and data science (want speed and self-service), security (wants control and evidence), finance (wants unit costs and predictability), and source-system owners (want no change and no new obligations). Each constituency can block the platform for a different reason, and their requests are frequently contradictory — security's audit-log requirement and analytics' latency requirement will conflict, and the architect has to resolve the conflict rather than escalate it. `../management/communication_stakeholder_management_skills_guide.md` is the repository's material on the mechanics; the platform-specific addition is the *five-way* nature of the map and the fact that three of the five hold vetoes rather than votes.

**The vendor and procurement relationship.** Platform architects in practice spend a significant share of their time with vendors: hyperscaler account teams, warehouse and lakehouse vendors, catalog and observability vendors, and systems integrators. The specific competences are: distinguishing a benchmark from a reference customer; refusing to accept a vendor's own workload as evidence for your workload; structuring pilot agreements with defined exit costs and data-extraction guarantees; and understanding the commercial shape of the contracts they influence (commitment drawdown, egress, per-seat versus per-compute pricing). `../management/vendor_management_guide.md` is the repository's treatment and should be read before any platform-relevant procurement, with `../management/business_case_development_guide.md` for the funding argument. The platform-specific failure here is a purchase optimised for the architect's strategic narrative rather than the platform's unit economics — §10(vi) and §10(v).

**Running an architecture review that people follow rather than evade.** The review's purpose is to make the expensive-to-reverse decisions once, publicly, with the trade-offs visible. A review that people evade has usually failed in one of three ways: it reviews artefacts instead of decisions (so it becomes a document-grading exercise); it has no decision rights (so it is advisory, and advisory review is always evaded under deadline pressure); or it approves designs it has not costed (so the rejection feels arbitrary). The design that works is narrow and consequential: review only the expensive tier from §4.6, require the reversal cost in the ADR, publish the decisions, and let everything else pass. `architecture/why_tech_bypasses_togaf_guide.md` documents the evasion dynamics in detail and is the honest starting point for anyone redesigning a review board.

**The platform's internal product relationship.** This is the competence that distinguishes the role from infrastructure leadership. Treating internal consumers as customers means: an intake process that reasonably resembles a product backlog; a published service catalogue with guarantees; a deprecation policy consumers can plan against; deliberate attention to time-to-first-value for a new consumer; and a feedback mechanism that changes the roadmap. It also means accepting that a capability nobody adopts is a failed feature, not an under-appreciated one. The primary-source definitions in §1 support this framing directly: the practitioner definition of a digital platform is "a foundation of self-service APIs, tools, services, knowledge and support which are arranged as a compelling internal product" (Evan Bottcher, martinfowler.com, 5 March 2018), and the CNCF's vendor-neutral white paper defines a platform as "an integrated collection of capabilities defined and presented according to the needs of the platform's users" while insisting that platform teams "should not always implement them themselves" and should be "the thinnest reasonable layer" over backing implementations (CNCF Platforms White Paper, TAG App Delivery). The phrase "thinnest reasonable layer" is a governance instrument: it is the sentence that stops a platform team from absorbing everything.

---

## 6. What "senior" actually means here

This section answers the question neither predecessor asks. It is the dimension on which §9 (interview), §13 (the worked example) and the reader's own self-assessment depend, so it is worth getting right rather than gesturing at "more experience".

### 6.1 The one published seniority ladder that exists for this family of roles

There is **no vendor-neutral published levelling framework for "data platform architect"** that this research could verify. There *is* a published, government-owned ladder for the adjacent data-architect role, and it is instructive precisely because it is an official publication rather than an employer's internal opinion. The UK Government Digital and Data Profession Capability Framework (role page "Data architect", first published 7 January 2020, last updated 29 May 2026) defines three levels and, for each, a role summary plus the required proficiency level for eleven named skills on a four-step scale (awareness → working → practitioner → expert). Two things matter about it:

- **The ladder is expressed in scope of ownership, not in knowledge depth.** The entry level "designs and builds data models to fulfil the strategic data needs of the organisation, **as defined by** chief data architects". The senior level "**delivers the vision** for the organisation **as set by** the chief data architect" and "provide[s] oversight and advice to other data architects who are designing and producing data artefacts". The chief level "**sets the vision**", "set[s] the standards and ways of working for the data architecture community", and is "responsible for ensuring that the organisation's systems are designed in accordance with the enterprise data architecture". The progression is from *implementing a vision someone else set*, to *delivering it and governing other architects*, to *setting it and owning the standards*.
- **The skill levels rise with it, mechanically.** Across the three levels the same eleven skills move from working-level at the first level (strategic awareness only, "support strategic planning in an administrative capacity") to practitioner-level at the senior level ("work across multiple subject areas, or a single large or complicated subject area", "design data architecture that deals with problems spanning different business areas") to expert-level at the chief level ("design data architecture that deals with problems across the enterprise"). Communicating between the technical and non-technical rises working → practitioner → practitioner; data governance rises working → practitioner → expert.

The mapping to the platform architect role is direct, with one substitution: where this ladder measures ownership of a *data vision*, the platform architect's ladder measures ownership of a *platform*. Both ladders move along the same axis.

### 6.2 The six dimensions that actually change between mid and senior in this role

1. **Size of the problem owned — from a system to a set of systems and their economics.** A mid-level practitioner designs and delivers a platform capability: an ingestion framework, a table-format migration, a compute tenancy model. A senior architect owns the platform's coherence: which capabilities exist, which do not, what they cost together, and how the whole set evolves. The tell is whether the person's design conversations are bounded by a component or by the platform's total cost and risk profile.
2. **Ambiguity tolerated — from "solve the stated problem" to "decide whether this is the problem".** Mid-level work usually arrives specified: the source is X, the target is Y, the latency budget is Z. Senior work frequently arrives as "the risk function says reporting is too slow", and the senior contribution is partly diagnosis. A senior architect is expected to spend time finding out that the real constraint is a source-system batch window, not the platform's query engine — and to be comfortable stating that the platform change requested will not solve the problem.
3. **Consequence carried — from delivery consequences to economic and regulatory consequences.** Mid-level failure is a late project. Senior failure is a multi-year cost commitment, an audit finding, an outage with a regulatory-reporting consequence, or a migration everyone must now pay for. The observable marker is whether the person has written or been the named owner of a document that constrained spending or obliged other teams.
4. **Standard of artefacts — from working artefacts to defensible artefacts.** A mid-level design needs to work. A senior design needs to survive an architecture review, a security assessment, a procurement challenge, an audit query two years later, and a new joiner's questions — which in practice means it must carry assumptions, evidence class, alternatives rejected, and reversal cost (§4.6). Senior-ness here is measurable: how long does a competent outsider need to reconstruct why this decision was made?
5. **The ability to say no — from arguing to deciding.** Mid-level practitioners argue for their preferred design. Senior architects are expected to kill work: reject a requested capability, refuse a workload the platform should not carry, decline a vendor, stop a migration, and state the boundary in §2.3 without being asked. The marker is a decision that was unpopular and was not reversed. This is where the phrase "senior" is doing real work and where the Accenture-style title inflation in §1.4 is exposed: a requisition asking for two years of coding experience is not asking for the ability to say no.
6. **From designing systems to shaping the conditions in which systems get designed.** The end state of the ladder is not "better designs". It is that *other people's designs get better without the architect in the room*: the platform makes the correct pattern the cheapest one; the golden path exists and is used; the default is secure; the expensive decision has a template; the review board has clear decision rights. This is the DDaT chief-level pattern applied to a platform ("sets the standards and ways of working", "sets the vision"), and it is the single clearest signal of seniority available in an interview.

### 6.3 What is NOT different — stated honestly

- **Titles inflate and the work sometimes does not.** The Accenture requisition in §1.4 is a published example of a "Data Platform Architect" title attached to a role description that sits under "guidance from senior architects". The converse also exists: practitioners with unremarkable titles doing unambiguously senior work. Title is therefore not a reliable seniority signal in either direction, and neither is years of experience — the same requisition states a 5–10 year band while setting a two-year technical floor, which shows how loosely the band is used.
- **The craft does not change.** Senior architects still read schemas, still look at query plans, still debug a failing job when the stakes justify it. Roles that "graduate" away from the technical substance have not become more senior; they have become a different job (architecture governance, or management), which is a legitimate path but not this ladder.
- **The reading does not stop, and the repository does not stop being relevant.** The seniority delta is not "needs less knowledge". It is that knowledge is no longer the binding constraint.
- **Seniority does not reduce on-call or ownership of consequence.** In most organisations the senior platform architect is in the incident and cost-escalation path precisely *because* the consequences are theirs. Anyone expecting seniority to mean distance from production has the direction of travel backwards.

### 6.4 A practical instrument: eight questions that separate mid from senior

Use these in a hiring loop (§9) or on yourself. A mid-level candidate answers the first four competently and gives generic answers to the rest; a senior candidate's answers to 5–8 contain names, numbers and a decision that cost something.

1. Describe the biggest platform decision you have made. What was the reversal cost, and who pays it if it is wrong?
2. What platform capability did you kill, and what did you say to the person who wanted it?
3. Give an example of a requirement you pushed back on as misconceived. What did you do instead?
4. How do you know the platform is being used? What does your adoption evidence look like?
5. Where is the boundary between your platform and the business data model, and who owns the other side of it?
6. Which of your platform's published guarantees are currently unmet, and who knows?
7. What is your cost model per capability, and which capability is subsidised by which?
8. Which part of the platform is designed to be *replaced* in the next five years, and what makes that affordable?

**Observable artefacts that carry the same signal without an interview:** a platform capability roadmap with named non-goals; ADRs that state reversal cost; published SLOs alongside published misses; a deprecation policy with dated commitments that were honoured; an interface catalogue with version history; a cost-per-capability model; a post-incident review the architect wrote about their own platform. Two of these are worth more than any certification, because none of them can be produced without having actually owned a platform.

---

## 7. The market and the ladder

### 7.1 Where the role exists

The title is used across five contexts, and the work differs enough between them that they should be treated as different markets:

| Context | What the platform architect actually does there | Notes |
|---|---|---|
| **Cloud-native product and platform companies** | Builds the internal data platform as a product for dozens to hundreds of internal teams; heavy emphasis on self-service, golden paths and developer experience | Closest to the CNCF white-paper definition; often titled platform engineering lead / staff engineer rather than architect |
| **Regulated financial services** | Builds the platform under BCBS 239-style obligations, residency constraints, audit and access-control requirements; strong governance interlock | See §11. The architect-of-record obligations are real here, which is where the title has teeth |
| **Large enterprise / non-tech (telco, retail, healthcare, energy, public sector)** | Consolidates a historically fragmented data estate onto a governed platform; the work is often migration and de-duplication rather than greenfield design | Public-sector variants exist under the Government Digital and Data framework, whose published roles include data architect, data engineer, analytics engineer and platform engineer rather than a single composite role |
| **Consultancies and system integrators** | Architecture title attached to an engagement's project role, frequently hands-on delivery | The Accenture requisition in §1.4 is the example: the title is an engagement role, the body is delivery work |
| **Vendors and cloud providers** | Customer-facing or reference architecture work for the vendor's own stack | Attractive for exposure to many estates; the architectural judgement is bounded by the vendor's product set, which is a real limitation for a senior hire |

### 7.2 The ladder: what this role sits between

- **From:** data engineer / senior data engineer; platform engineer or DevOps/SRE practitioner; analytics engineer; solution architect with data-heavy engagements; DBA or infrastructure engineering with data-platform exposure; consulting delivery.
- **To:** head of data platform / platform engineering manager (management track); principal or chief architect (individual-contributor track); data architecture leadership or chief data architect (DDaT-style ladder, §6.1); platform product management (the product relationship in §5 formalised); independent consulting or practice-building.

The ladder is genuinely bidirectional at the top: the same seniority that makes someone a staff-level architect makes them credible as a platform product manager, because both roles are about the platform's consumers. That optionality is a real, under-discussed advantage of this role over the data-architect role.

### 7.3 The demand picture, with sources labelled

**Verified:** the role title is in active use by named employers and the published requisitions display architect-level titles for delivery-level scope (Accenture requisition ATCI-5701048-S2061816, accessed September 2026 — *published employer job description*). **Verified:** an official, maintained role definition exists for the adjacent data-architect role family and is refreshed regularly (UK Government Digital and Data Profession Capability Framework, published 7 January 2020, updated 29 May 2026 — *government role definition*). **Not verified:** any vendor-neutral index of demand for this exact title. No such index was found in this research; searches for one returned job-board aggregators and content sites with no stated methodology. Claims of "growing demand" for this role should therefore be treated as **unverified projection**, not evidence — including the version of that claim this guide could have made most easily.

### 7.4 Compensation: the honest result

**This guide prints no compensation figure for any geography, because none of the candidate sources survived the standard the brief set (named source + date + geography + methodology).** The reasoning, source by source:

1. **Recruitment-firm salary guides** — the most common source in this space, and a marketing document in form. Robert Walters' Singapore salary survey is published as a gated download behind a form ("Salary Survey Guide Singapore 2026", robertwalters.com.sg, marketing page, no figure published on the page itself). A recruitment guide's figures are not measurements and, in this case, are not even publicly printed. **No figure usable.**
2. **Crowd-sourced compensation sites** — no stated methodology and, in the one sample retrieved, internally implausible: a Glassdoor page for "Data Architect" in Singapore returned "SGD 11,750 per year or SGD 6 per hour", which is self-evidently a monthly figure rendered as an annual one, alongside a "90% lower than the national average" comparison that cannot be true for the role. Whatever the underlying data, a figure that fails arithmetic sanity should not be reproduced. **Rejected.**
3. **Aggregator and template sites** — undated, unsourced ranges for adjacent titles (for instance a "$135K–$195K" band published for "Cloud Data Architect" on a job-description content site with no date, no geography statement and no methodology). These are content-farm artefacts. **Rejected** and recorded in the §15 audit as an example of what was refused.
4. **Government-statistics-derived publications** — the closest thing to a defensible source: a Straits Times "Singapore Salary Guide 2025" landing page explicitly attributes its salary results to the Ministry of Manpower's Occupational Wages 2024 data and the Labour Force in Singapore 2024 report (stated methodology, stated geography, stated vintage). It does not, at the level this research could verify, publish a figure for this role or its exact-title neighbours, and the guide's title vocabulary (occupational classifications) does not map cleanly onto "data platform architect". **No figure that can be attributed to this role.**

**The finding is the absence.** A title that is widely used in requisitions has no published, methodologically defensible, geographically scoped compensation data that this research could verify — and printing a recruitment guide's number as if it were a measurement would have produced a more confident guide and a less accurate one. This follows the convention already set by `data_architect_skillgaps_guide.md`, which likewise printed no compensation figure. For **Singapore and Asia** specifically — the geography most relevant to this series — the answer is that no defensible figure was found, and the same is true of every other geography examined: **no figure is printed for Singapore, Asia, the UK, the US, or India.** Where a practitioner needs a number, the honest route is to obtain dated figures from actual offers and published pay-transparency disclosures in the specific jurisdiction, which is a primary source this guide cannot substitute for.

---

## 8. The routes in and the transition

### 8.1 The routes, and the conditions under which each works

| Route in | What it requires | Typical blockers | When it works |
|---|---|---|---|
| **Data engineering → platform** | Evidence of having built shared primitives (framework, template, contract, ingestion pattern) rather than pipelines only; cost and SLO exposure | Career identity as a pipeline author; no experience of internal consumers; rarely has owned a budget or an SLO | The most common route. Works when the practitioner deliberately moves from "my pipelines" to "the thing other people's pipelines run on" |
| **Platform engineering / DevOps / SRE → data platform** | Data-domain literacy: storage layout, table formats, query engines, lineage, governance interlock | Understands runtime and reliability deeply, weak on data semantics and the governance/policy layer; may underestimate how different data workloads are from stateless services | Works well where the organisation's platform is already infrastructure-heavy and engineering-led. The fast fix is the modelling and governance material in `data/data_vault_2_modeling.md` and `data_governance_guide.md` |
| **Analytics engineering / BI → platform** | Systems depth beneath the transformation layer; the operating discipline of an on-call platform | Knows the consumer side intimately (a real advantage) but may lack engine internals and multi-team ownership | Works where the platform's main product is the semantic and transformation layer |
| **Solution architect → platform architect** | Conversion of programme-by-programme design experience into a persistent, multi-consumer artefact; the seniority delta in §6.2 | Solution work is bounded by a delivery; platform work is not. The blocker is not knowledge, it is the absence of an owned, long-lived, multi-tenant artefact | This is the reader's most plausible route and the one this guide is written for. The predecessor `data_architect_skillgaps_guide.md` supplies the data-domain frame; `architecture/skill_gaps_enterprise_architect_guide.md` supplies the enterprise-influence frame |
| **Consulting / system integration → in-house platform** | Trading breadth for depth: one platform, owned for years, with consequences | Adjustment to being accountable after the engagement ends | Works when the consultancy experience included implementation ownership, not only design |
| **DBA / infrastructure → platform** | Distributed-systems and table-format breadth; the interface/contract layer | Historically narrow engine focus; but engine internals literacy is a genuine and scarce asset | Works in on-prem or hybrid estates where engine behaviour is the binding constraint |

### 8.2 What the market actually rewards

Three honest observations, based on the sources in §15 rather than on encouragement:

1. **The market rewards evidence of an owned, multi-consumer artefact far more than it rewards certification.** The credentials that exist for architecture are real but adjacent: The Open Group's Open Certified Architect programme certifies Business, Digital, Enterprise and Solution Architects — **four specialisations, none of them data or data platform** (The Open Group, Open CA programme pages, accessed September 2026). That is a finding, not a gap in the research: **there is no widely recognised credential for the data platform architect role**, and anyone claiming otherwise should be asked for the body that issues it. TOGAF and DAMA-DMBOK/CDMP are adjacent and genuinely useful (the predecessor's §1.12 maps the latter), but neither certifies this role.
2. **The transitions that are common are the ones with a shared artefact class.** Data engineering → platform works because the artefact class (ingestion, storage, execution) is the same. Analytics → platform works where the platform's product is the transformation layer. The rare transitions are those crossing from design-only work into ownership: solution architect → platform architect is *possible* but rarer than the title counts suggest, because the market reads "architect" as seniority rather than as scope, and will probe for the scope.
3. **The transition is won on consequence, not on knowledge.** Everything in §3 and §4.1–§4.6 can be read. §6.2's six dimensions cannot be read into existence, because they are all evidenced by things the reader must have done. This is why the practical transition plan is not a study plan: it is a request for a small piece of platform ownership with real users, plus the reading needed to be credible in it.

### 8.3 The mechanics, and where the predecessors apply

- **For the search method** — how to find and approach the roles rather than what to put in them — use `../management/reverse_job_search_guide.md`. It owns the method; this guide deliberately does not restate it.
- **For the first months in the role** — the platform-specific version of onboarding is to inventory the platform's consumers, its published guarantees, and its published misses in the first thirty days, because those three facts determine credibility — see `../management/the_first_90_days_guide.md` for the general structure.
- **For the domain gap analysis** — `data_architect_skillgaps_guide.md` §3–§5 (gap analysis, ranked list, learning plan) and its §9.3 commission list. Its findings about managed cloud data warehouses, data contracts, and quality/observability (see §3 of this guide) are the platform architect's gaps too, and re-deriving them here would produce a competing ranking rather than a useful one.
- **For the enterprise-influence gap** — `architecture/skill_gaps_enterprise_architect_guide.md` §3, whose banking-domain depth gaps (risk models, treasury, capital markets) apply unchanged to a platform architect in a regulated firm. Its readiness estimate (65–70%) and the data guide's (70–75%) are both readiness-against-profile numbers; the seniority model in §6 of this guide is what should be used to decide *which level of role* those percentages entitle the reader to apply for.

---

## 9. The interview and the assessment

### 9.1 The shape of the loop

The loop for this role is usually five to seven stages, and each stage tests a different half of the job. The repository's interview shelf already owns the *technique* for the design stages — `system_design_interview_insiders_guide.md`, `google_system_design_interview_guide.md`, `nalsd_system_design_guide.md`, `grokking_system_design_companion_guide.md`, and `ml_system_design_interview_guide.md` for the AI-shaped variants. Read those for how to run a design conversation, structure requirements, and manage time. This section covers only what is *specific to this role*.

| Stage | What it is testing | The platform-specific twist |
|---|---|---|
| Recruiter screen | Title and band calibration | Expect the band to be quoted in years and the scope not at all. Ask what the platform's consumers are; recruiters rarely know, which tells you a lot |
| Hiring manager | Whether you have owned a platform or only designed one | The first question behind the first question is "how many consumers, and who decided not to use it?" |
| Platform design / architecture case | Trade-off reasoning under incomplete information (§4.1), NFR specification (§4.2), interface design (§4.3) | The prompt is usually "design the data platform for X", not "design pipeline for X". Strong candidates immediately ask who the consumers are and what the second-year cost profile is |
| Technical depth probe | Whether the trade-off reasoning is grounded | Expect to go two levels below the technology name — not "Iceberg or Delta" but "what does that mean for your compaction job, your catalog, and your ability to change engine in eighteen months" |
| Stakeholder / behavioural | §5 and §6.2 — influence, conflict, saying no | The scenario is usually cross-functional with a veto-holder (security, finance, a source-system owner), and the scoring looks for a *decision*, not a facilitation |
| Architecture review or written exercise | Standard of artefacts (§6.2 dimension 4) | Often an ADR or a review response. The scoring looks for assumptions, evidence class, rejected alternatives and reversal cost |
| Principal / peer review | Whether senior practitioners would work with you | Frequently where the "owns no consequence" pattern (§10 vii) is caught |

### 9.2 What a strong answer looks like in the design exercise

The prompt is almost always under-specified, and that is the assessment. A strong answer, in order:

1. **Establishes the consumers and their jobs first.** Who will use this platform, for what, and what are they doing today instead. This is the platform-as-product competency and it is usually the first discriminator.
2. **Converts the requirements into a small set of numbers.** Freshness, availability/error budget, cost ceiling, access-control obligations, recovery objectives — each attached to a capability and a consumer (§4.2). Naming the numbers that are *not* yet known, and how they would be found, is better than inventing all of them.
3. **Draws the layer boundaries and the interfaces between them**, treating each interface as a published contract with a version and a compatibility promise (§4.3).
4. **States what the platform will not do.** The non-goals are the strongest signal of seniority in the whole exercise, because they demonstrate the ability to say no in advance rather than after the fact (§6.2 dimension 5).
5. **Identifies the expensive-to-reverse decisions and their reversal costs** (§4.6): storage layout, table format, catalog, identity/access model, tenancy, residency topology. It is acceptable to *defer* one of these deliberately, with a date and a criterion; it is not acceptable to be unaware of them.
6. **Closes the evolution loop.** What changes at year one, year three, and what is designed to be replaced rather than replaced by force (§4.4).
7. **Handles the organisational answer.** Who will build it, which existing team's scope this collides with, and who has the right to say no. Candidates who design a platform with no adoption path routinely fail the stakeholder stage even when the design is good — this is precisely the failure `architecture/why_tech_bypasses_togaf_guide.md` describes.

**A mid-level answer** covers points 1 and 3 well, treats the layers as technologies rather than boundaries, produces a design that would work if adopted, and does not discuss what happens to the teams who have already built their own version of it. **A senior answer** is recognisable by points 4, 5, 6 and 7 — non-goals, reversal costs, an evolution thesis, and an adoption argument — and by the fact that the numbers in point 2 have owners attached.

### 9.3 The trade-off conversation

Expect to be handed a conflict with no correct answer and asked to choose: latency against cost; self-service against control; one platform for everyone against two fit-for-purpose stacks; buying the capability against building it. The scoring is not for the choice. It is for whether the candidate (a) names the criterion that decides it, (b) states the evidence class behind each input, (c) names what would reverse the decision, and (d) states who bears the cost of being wrong. A candidate who asks for a missing constraint — the size of the data, the regulatory obligation, the budget owner — scores higher than one who resolves the conflict confidently and unilaterally.

### 9.4 The stakeholder scenario, and the depth probe

The stakeholder scenario usually hands the candidate a peer function with a veto: security refuses the proposed access pattern; finance refuses the unit-economics model; a source-system owner refuses to expose change data capture; an analytics team refuses to leave their own stack. What is being scored: whether the candidate proposes a *decision* with a decision-right rather than a consensus process, whether they have a fallback that does not require the veto-holder's consent, and whether they can concede the right thing (usually a delay or a narrower scope) without conceding the boundary (§2.3).

The depth probe is the mechanism that catches over-claimed seniority: two or three levels of "why", requested in the candidate's own terms. On an ingestion framework: how do you know a hidden producer repeated a batch? On a table format: what happens to the compaction job when the write pattern changes? On a cost model: which team is currently subsidised and who has been told? On access control: show me how you would prove, two years later, who could see this column on this date. Grounded answers to questions like these are extremely difficult to fabricate, which is why the stage exists.

### 9.5 The portfolio question

What matters as work samples for this role, in decreasing order of evidentiary value: (1) a platform capability roadmap with named non-goals (carefully redacted); (2) an architecture decision record for an expensive-to-reverse decision, with reversal cost stated; (3) published SLOs alongside published misses; (4) a deprecation or migration plan with dated commitments that were met; (5) an interface/contract catalogue showing version history; (6) a cost model per capability; (7) a post-incident review the candidate wrote about a failure in their own platform. Code, dashboards and certificates are weak substitutes for all seven, because they do not evidence ownership of consequences. Where confidentiality prevents sharing artefacts, the same signal is obtainable in an interview by describing them in enough structural detail to be probed (§9.4).

### 9.6 What is NOT assessed despite mattering

Stated honestly, because the gap between what is assessed and what determines success is unusually wide here: **the ability to say no** is barely assessed and is central to §6.2. **The political work in §2.3** — negotiating a boundary, filling a vacuum deliberately rather than by drift — is almost never assessed and determines whether the platform survives. **Actual adoption** is only assessed if the candidate volunteers it; interviewers rarely ask for it directly, and candidates rarely volunteer the number who declined. **Cost accountability** is assessed only in organisations sophisticated enough to have a unit-economics practice. **Vendor-relationship competence** (§5) is usually untested. And **whether the candidate has ever been wrong at scale, and what they did about it** — the single experience that produces §4.7's earned competencies — is assessed only by good interviewers. A candidate who prepares for all six of these questions without being asked them will outperform one who prepares only for the design exercise.

---

## 10. The failure modes of the role

Seven anti-patterns, written as things to recognise **in oneself**. Each has a symptom that is observable early, a cause that is usually structural, and a guardrail that is within the architect's own power.

| # | Anti-pattern | Symptom (observable early) | Cause | Guardrail |
|---|---|---|---|---|
| i | **Architecture delivered as documents, never as a platform** | Excellent design artefacts; no working implementation; every capability "in progress"; roadmaps that never close an item | The role was scoped as governance rather than delivery, or the architect has no build capacity and no authority to obtain it | Refuse to own a capability without a delivery path and a named builder. Make "shipped, adopted, measured" the definition of done, and publish a roadmap where each item has a consumer |
| ii | **The reference architecture nobody uses** | High conformance in review; divergence in production; teams ask for exemptions as a formality; the architecture exists in a repository nobody reads | The reference architecture is technically sound and requires effort the consumer has no incentive to spend; it is governed rather than offered | Ship the compliant path as the cheapest path (template, golden path, default). Measure adoption, not conformance. This is exactly the dynamic documented in `architecture/why_tech_bypasses_togaf_guide.md` |
| iii | **The platform built for its builders rather than its consumers** | Very elegant internal abstractions; long onboarding time; consumers writing workarounds; satisfaction not measured or not asked | The platform team optimises for its own elegance and roadmap; no consumer feedback loop exists | Fix time-to-first-value as a tracked metric; harvest proven solutions from consumers rather than designing for them; enforce the "thinnest reasonable layer" discipline from the CNCF white paper (§5); run the consumer experience end-to-end as a new joiner |
| iv | **The governance regime that drives consumers around the platform rather than onto it** | Shadow platforms; data copied out; teams keeping local storage; exemptions granted frequently; a growing "interim" estate | Controls were imposed as friction at the point of use rather than embedded as defaults; policy became the platform's output instead of its property | Embed policy in the default path (masking on by default, classification applied automatically, lineage captured without asking). Where a control must be visible, cost it and publish the trade-off. Track the shadow estate as an adoption metric |
| v | **Cost-blind design** | No unit economics; nobody can say which capability is expensive; cost discovered at invoice; performance decisions made without a cost counterpart | Cost is owned by finance rather than by the platform; consumption is not attributed to a consumer | Build the cost model per capability before the next large design; publish consumption to consumers; make cost per unit of work a first-class NFR (§4.2) — see `finops_guide.md` and `capacity_sizing_guide.md` |
| vi | **The technology choice made for the architect's CV** | Adoption of a technology without a workload that needs it; the rationale references industry trend rather than the platform's constraints; the migration is announced before the reversal cost is stated | Incentives that reward visible novelty; no obligation to state the reversal cost | Require an ADR with reversal cost, an alternatives-rejected section and a named workload for every expensive-tier decision (§4.6). Make "which of our constraints does this solve" a standing review question |
| vii | **The architect who owns no consequence** | Designs without SLOs; no incident path; no cost accountability; reviews that approve without costing; "that is an operations problem" | Role scoped as advisory, or authority granted without accountability (the mirror of §2.3's accountability without control) | Attach every architect-owned decision to a consequence: an SLO, a cost line, an on-call path, or an audit obligation. If the organisation will not grant any of these, the role is not the role (§6.2 dimension 3) and the correct response is to negotiate it, not to accept it silently |

A note on the pattern behind the patterns: (i), (ii) and (iii) all fail through the same mechanism — the platform produces what its builders can produce rather than what its consumers will use. (iv) and (v) fail through the same mechanism — a cost or a control is imposed somewhere other than the point of use, so the consumer routes around it. (vi) and (vii) fail through incentive design. That is not coincidence; a platform is a product, and all seven are product failures wearing architecture clothes.

---

## 11. The regulated-enterprise and banking angle

This section establishes what changes when the platform sits inside a regulated financial institution, using primary material. It does not name an institution.

### 11.1 BCBS 239, established from the standard itself

The following is taken from the standard's own text and annexes (Basel Committee on Banking Supervision, *Principles for effective risk data aggregation and risk reporting*, January 2013, bis.org — read directly from the published PDF for this guide, not from a secondary summary).

**What it is.** Fourteen principles, eleven addressed to banks and three to supervisors, grouped into four sections, plus a definition, scope statement and two annexes (a glossary of terms used in the document and a summary of the principles). It was issued in January 2013, following a June 2012 consultative version. Its stated motivation is that the 2007 crisis revealed banks' information technology and data architectures "were inadequate to support the broad management of financial risks", and that many banks lacked the ability to aggregate risk exposures and identify concentrations "quickly and accurately at the bank group level, across business lines and between legal entities".

**The principles, in the standard's own grouping:**

| Section (as the standard groups them) | Principles |
|---|---|
| I. Overarching governance and infrastructure | **P1 Governance**; **P2 Data architecture and IT infrastructure** |
| II. Risk data aggregation capabilities | **P3 Accuracy and Integrity**; **P4 Completeness**; **P5 Timeliness**; **P6 Adaptability** |
| III. Risk reporting practices | **P7 Accuracy**; **P8 Comprehensiveness**; **P9 Clarity and usefulness**; **P10 Frequency**; **P11 Distribution** |
| IV. Supervisory review, tools and cooperation | **P12 Review**; **P13 Remedial actions and supervisory measures**; **P14 Home/host cooperation** |

*Correction worth recording:* the first predecessor guide in this series describes the sections as "governance P1–P5, aggregation capabilities P3–P6, reporting practices P7–P11, supervisory P12–P14". The standard's own section I covers **P1–P2 only** (Governance; Data architecture and IT infrastructure); P3–P6 (accuracy/integrity, completeness, timeliness, adaptability) are section II. The count of fourteen, the four sections, the January 2013 date, the 1 January 2016 G-SIB deadline and the P12–P14 supervisory set are all confirmed against the primary text; the P1–P5 governance grouping is not, and this guide uses the standard's grouping.

**What it obliges, and why it is a platform mandate — the specific clauses the role should know:**

- **P2 is literally a platform clause.** A bank "should design, build and maintain data architecture and IT infrastructure which fully supports its risk data aggregation capabilities and risk reporting practices **not only in normal times but also during times of stress or crisis**". The standard makes risk data aggregation and reporting a subject of business continuity planning and business impact analysis. In other words: the capability must be designed for the crisis, not merely available during it — which is an architectural requirement about failure domains, degradation modes and recovery objectives, not a reporting requirement.
- **P1 makes roles and service levels explicit.** Governance must "include agreed service level standards for both outsourced and in-house risk data-related processes", and the framework must be fully documented, subject to independent validation by "staff with specific IT, data and reporting expertise", integrated into the second line of defence and kept separate from audit, considered in acquisitions and divestitures, and "unaffected by the bank's group structure". That last clause is a topology constraint: aggregation capability must be independent of legal and geographic organisation. This is the regulatory statement of the boundary this guide's §2.3 calls political.
- **P2 also fixes the identifier problem.** Integrated data taxonomies and architecture across the group, including metadata, "as well as use of single identifiers and/or unified naming conventions for data including legal entities, counterparties, customers and accounts". The standard explicitly permits more than one data model, provided there are "robust automated reconciliation procedures where multiple models are in use" — a sentence every platform architect should read twice, because it is the standard granting architectural freedom in exchange for automated reconciliation.
- **P3 pushes work from humans to the platform.** Data should be aggregated "on a largely automated basis so as to minimise the probability of errors"; controls surrounding risk data should be "as robust as those applicable to accounting data"; manual processes and desktop applications require effective mitigants; risk data should be reconciled with sources including accounting data; and a bank "should strive towards a single authoritative source for risk data per each type of risk". Documentation must explain the appropriateness of any manual workaround and the actions proposed to reduce its impact. Automation, reconciliation and workaround-elimination are platform features; this principle is where a platform programme gets its mandate.
- **P6 mandates ad hoc capability, which has architectural consequences.** A bank must be able to meet a broad range of on-demand and ad hoc reporting requests, including during stress and in response to supervisory queries. Ad hoc capability is a design property (a query and modelling surface, self-service access, elasticity) rather than a report.
- **The standard bounds its own trade-offs.** Principles must be met simultaneously; trade-offs are acceptable only in exceptional circumstances and "there should be no trade-offs that materially impact risk management decisions"; boards and senior management must be aware of the trade-offs and their limitations, and banks must have policies and processes for applying them, able to explain the impact qualitatively and where possible quantitatively. **Materiality** is defined so that data and reports may exclude information only if it would not have influenced decision-makers — and applying it requires considering more than the number or size of excluded exposures. **Expert judgement** may be used on incomplete data "only on an exception basis", and where used must be documented and transparent enough for independent review.

**Scope, which is narrower than the market often assumes.** The principles are "initially addressed to SIBs" (systemically important banks) and apply at both group and solo level; national supervisors may extend them, proportionately, to a wider range of banks. G-SIBs designated by the FSB in November 2011 or November 2012 were to meet them by **January 2016**, and G-SIBs designated in later annual updates within three years of designation; D-SIBs are strongly suggested for application three years after designation. The principles apply to a bank's risk management data and to "all key internal risk management models", including Pillar 1 regulatory capital models, Pillar 2 capital models and other key risk management models such as value-at-risk. Critically for a platform architect: **all the principles apply to processes outsourced to third parties**, which means a cloud or vendor-hosted platform does not move the obligation.

**Coming into force, and what has followed.** The standard itself provides that the Basel Committee would track G-SIB progress through its Standards Implementation Group from 2013 and share findings with the FSB at least annually from the end of 2013. The Basel Committee continues to publish implementation progress reports on these principles (a November 2023 report on adopting BCBS 239 is published at bis.org; this guide did not read its findings and makes no claim about them). In Singapore, the Monetary Authority of Singapore published an information paper, *Data Governance and Management Practices*, on **29 May 2024**, setting out supervisory expectations for banks and finance companies based on BCBS 239 as well as observations from thematic inspections of domestic systemically important banks conducted in 2022 and 2023, covering board and senior management oversight, data management organisation, data quality management and controls, data issue identification and escalation, and observations relating to BCBS 239 (MAS information paper, 29 May 2024, mas.gov.sg; the date, scope and themes are corroborated by a law-firm client note of 25 June 2024). This repository's own `../banking/risk_data_aggregation_guide.md` carries the deeper verified treatment, including the ECB's May 2018 thematic review finding that none of 25 significant institutions had fully implemented the principles and the Pillar 2 / SREP consequences; that treatment is not re-derived here, and its core facts should be read there.

### 11.2 What else changes in a regulated enterprise

Beyond risk data aggregation, five constraints re-shape the platform's architecture. Each is a design input the platform architect must obtain before designing, not after:

1. **Residency and sovereignty.** Where data may physically reside, who may access it, and under whose law. This drives region topology, replication, key management, cross-border transfer mechanisms and sometimes the decision to run a capability on-premises or in-country. The repository's `data/data_compliance_frameworks.md`, `data/china_data_governance_frameworks.md` and `data/alibaba_data_platform.md` are the regional treatments; `data/enterprise_data_platforms_guide.md` covers the platform-selection interaction. The practical consequence for the platform architect: residency can eliminate the cheapest architectural option, and the elimination must be discovered in the design phase rather than in a legal review of the build.
2. **Access control and audit obligations.** Regulated data platforms must be able to prove, after the fact, who could see what and who did see what, on a date in the past. That converts access control from a feature into an evidence-generation requirement, and it constrains the retrospective migrations that are otherwise a cheap fix: today's correct access model does not reconstruct yesterday's state unless the platform captured it. Auditability is a design constraint on the metadata and access layers, not a reporting layer bolted on top.
3. **Lineage and data-quality obligations that follow from BCBS 239.** P3's reconciliation and single-authoritative-source requirements, P1's documentation and independent validation requirements, and the workaround-documentation requirement together make lineage and quality *evidence* rather than internal quality metrics. `data/data_lineage_tools.md`, `data/data_profiling_guide.md`, `data_governance_guide.md` and `data/data_governance_framework.md` cover the capability; the regulated twist is that the evidence must be producible to a supervisor, which is a higher bar than being visible in a dashboard.
4. **The model-risk interface.** Where the platform feeds regulated models — and per the standard's own scope statement, that includes Pillar 1 capital models, Pillar 2 capital models and value-at-risk — the platform's data supply chain inherits obligations about completeness, timeliness and adaptability, and about the documentation of any manual intervention. The platform architect's part is bounded: they own the substrate, the lineage and the guarantees, and are accountable for the platform's share of the model-risk evidence. The model itself is not theirs. The repository's `../banking/risk_management_models_guide.md` and `../banking/financial_risk_compliance_systems_guide.md` own the model and compliance substance.
5. **Operational resilience expectations on the platform as a dependency.** Once the platform is a dependency of regulated reporting and risk processes, its outage is a resilience event rather than a service incident: impact tolerances, recovery objectives, third-party dependency management and scenario testing all attach to it. `../banking/operational_resilience_framework_guide.md` owns the framework. Two design consequences: single points of failure that are tolerable in a commercial platform become findings; and the platform's dependencies (a cloud region, a vendor's control plane, a third-party catalog) must themselves be assessed, since the standard applies the principles to outsourced processes.

The net effect in a regulated institution is that the platform architect's design space is smaller and their evidence obligations are larger. The compensation is real: their decisions are consequential in a way that is measurable, which is exactly the §10(vii) guardrail — regulated firms can supply the consequence that makes the role senior.

---

## 12. The AI-era effect on the role, evidence-graded

Every claim below carries its source type. The rule applied throughout: **documented evidence is separated from survey evidence, survey evidence from vendor marketing, and all of it from projection** — and where the honest answer is "this is my projection", it says so.

### 12.1 What is documented

- **AI assistance can slow experienced practitioners down on realistic work, in at least one rigorous experiment.** METR ran a randomised controlled trial with 16 experienced open-source developers working on 246 real issues from large repositories; issues were randomly assigned to allow or disallow AI tooling, developers recorded their screens and self-reported implementation time. The finding was a **19% increase in completion time when AI was allowed**, against developers' own forecast of a 24% speed-up and a post-hoc belief of a 20% speed-up. [Source type: **peer-reviewable experimental study** — METR, July 2025, arXiv 2507.09089.] The authors themselves state the limits: their developers and repositories are not representative of most software work, and the result does not show that AI fails to help elsewhere. **They also state that these results are out of date**: METR published a follow-up as of early 2026 and a 2026 survey of 349 technical workers reporting a median 1.4–2× self-reported change in the value of their work, which METR cautions is self-reported and likely overstated in magnitude. [Source type for the follow-up: **documented experimental update** (METR, 24 February 2026) and **self-reported survey with stated sample size** (METR, 11 May 2026).] The honest reading for this guide is not "AI does not work" — it is that **AI's productivity effect in real engineering work is measured, contested, changing on a timescale shorter than a platform programme, and therefore not a sound basis for a platform design decision.**
- **AI adoption is associated with worse delivery stability even as individual productivity rises, and platform engineering shows the same pattern.** The DORA research programme's 2024 *Accelerate State of DevOps* report states that AI adoption "significantly increases individual productivity, flow, and job satisfaction" while "negatively impact[ing] software delivery stability and throughput", and separately that using an internal developer platform "improves individual productivity, team performance, and overall organizational performance" while it "can also lead to decreased change stability and throughput, requiring careful implementation focused on developer independence". [Source type: **industry survey research with a stated methodology, published by Google Cloud with named sponsors** — a vendor-sponsored but methodologically disclosed source, which is why it is cited with the sponsorship stated. I read the published findings summary, not the full report PDF.] The second finding is directly relevant to this role and cuts against the natural enthusiasm of a platform team: a platform can improve the experience and degrade the delivery system it supports.
- **There is no dedicated guide in this repository on AI-era data platform operations.** No guide here covers LLM-assisted pipeline authoring, agentic operation of a data platform, natural-language access layers over data platforms, or AI-generated governance artefacts as *operational* practices. The repository has deep adjacent material (`ai_platform_engineering_guide.md` for the AI platform as a system, the `ai_llm/` and `ai_llm/rag/` shelves for the model and retrieval layers), but the intersection — AI as an operating layer *of a data platform* — is not covered. [Source type: **repository scan evidence**; recorded here as a gap rather than papered over.]

### 12.2 What is vendor marketing

Vendor product pages and release announcements assert that AI features reduce the effort of ingestion, transformation, cataloguing and query. These are claims by parties with a commercial interest, they are frequently undated, and their benchmarks use the vendor's own workloads. [Source type: **vendor marketing**.] A related and genuinely useful signal sits one level removed: published requisitions asking for platform architects now routinely list AI-adjacent platform features as expected skills — the requisition examined in §1.4 lists Snowpark, vector search, retrieval pipelines and model-enablement patterns among "good to have" skills, and platform credentials among its "good to have" list, while its body describes operating the data infrastructure for AI workloads. [Source type: **employer job description**.] A requisition shows what an employer *wants*; it is not evidence that the capability works as advertised, and it is not evidence about the role's future shape.

### 12.3 Displaced, augmented, untouched — labelled as projection

The following is **my projection**, reasoned from the evidence in §12.1 and §11.1 and from the definition of the role in §1. It is not drawn from a study, and it should be read as an argument rather than a finding.

**Plausibly augmented by AI tooling (the platform architect gets leverage, the accountability stays):** drafting and reviewing pipeline code and infrastructure-as-code; generating first-draft documentation including ADRs and runbooks; cataloguing and metadata enrichment; lineage inference across heterogeneous sources; cost and capacity analysis; test generation; first-pass troubleshooting triage. Each of these is a task with a large, cheap-to-check output surface, which is where assistance has been most evidently useful.

**Plausibly partially displaced:** boilerplate ingestion and transformation implementation; the routine portions of schema mapping; some proportion of platform support rotation triage. The mechanism is not that the architect is replaced; it is that the *volume of implementation work per architect falls*, which in most organisations will show up as a team-size and skills-mix change rather than as a change in the architect's job description.

**Plausibly untouched — and this is the important list for this role:**

- **Accountability for a multi-year cost commitment.** Someone must sign the commitment and defend it in year three. That is a consequence, and consequences are not generated.
- **The political boundary negotiation of §2.3** — deciding what the platform will not absorb, and winning that argument against a stakeholder with a veto.
- **The decision not to build a capability** (§6.2 dimension 5). AI tooling lowers the cost of building, which by itself makes *not* building harder, not easier: when construction is cheap, the pressure to construct everything rises.
- **The calibration of evidence sufficiency** (§4.7) — how much evidence is enough to make an expensive, hard-to-reverse decision.
- **Audit, regulatory and model-risk sign-off.** The accountability in §11.2 attaches to a named human owner, and the standard's own text requires independent validation by staff with specific expertise.
- **Owning an outage's consequence** — including the regulated consequence where the platform is a dependency (§11.2.5).

**The AI-era risk this role should be worried about, stated as my projection:** if AI tooling lowers the cost of producing pipelines, models and analytics, the *demand* on the platform rises faster than the platform's capacity, and the platform's scarce resource shifts from build capacity to **governance bandwidth and cost control**. That makes §10(v) (cost-blind design) and §10(iv) (governance that drives consumers around the platform) more dangerous than they were, not less. The counter-measure is the discipline in §5 — the CNCF white paper's "thinnest reasonable layer" — and a contract surface strong enough that consumption can be admitted without a review meeting per consumer. If that is right, the AI era makes the *product* framing of this role more valuable, not less, and it makes the "architect who owns no consequence" (§10 vii) the profile most likely to be squeezed.

**What the evidence does not support, in either direction.** There is no verified evidence in this research that AI tooling has reduced headcount for this role, and there is no verified evidence that it has increased it. There is no verified source for a "data platform architect demand" trend. Anyone asserting either direction confidently is selling something, and in the absence of evidence the correct posture for a practitioner is the one this section has taken: change the cost of *tasks*, keep the accountability attached to humans, and re-examine the question on the same schedule as the technology.


---

## 13. The Cymbal Bank worked example (fictional, illustrative)

**Everything in this section is fictional and illustrative.** Cymbal Bank is a persona used throughout this repository, not a real institution. The two candidates are invented composites, not descriptions of any person. The figures in §13.7 are **illustrative arithmetic invented to demonstrate a method of reasoning** — they are not market data, not benchmarks, and not compensation figures from any source, and must never be read or quoted as such (§7.4 established why no sourced figure exists). The section demonstrates one thing only: how the framework in §3 and the seniority standard in §6 convert a hiring problem into a decision with a stated basis.

### 13.1 The scenario

Cymbal Bank is filling a **Senior Data Platform Architect** position. The platform in question serves four internal populations: the data engineering function (batch and stream pipelines), the analytics function (warehouses, dashboards, semantic layer), the data science and model-risk functions (training data, feature storage, model-serving substrate), and the regulatory-reporting function — the last of which gives the platform a BCBS 239-style evidence obligation (§11.1) and therefore consequence.

Two internal candidates have applied. There is a third option the hiring panel keeps postponing: hiring externally.

### 13.2 The requirement, as it should be specified

The most common defect in a requisition for this role is that it lists technologies where it should state ownership. Specified properly, the requirement reads as follows.

| # | Requirement | Evidence sought | Seniority marker (§6.2) |
|---|---|---|---|
| R1 | Owns the platform's architecture and its multi-year evolution as one coherent system | A platform capability roadmap with named non-goals; ADRs for the expensive tier | Dimension 6 — shapes the conditions others design in |
| R2 | Owns the interfaces and contracts the platform publishes | An interface catalogue with version history; a deprecation policy with honoured dates | Dimension 4 — artefact survives an outsider |
| R3 | Owns reliability and performance as a service, with published objectives | Published SLOs *and* published misses | Dimension 3 — has carried consequence |
| R4 | Owns the platform's cost model and can defend unit economics to finance | Cost-per-capability model; consumption reporting | Dimension 3 — has constrained spending |
| R5 | Owns the access model and its audit evidence | Classification-to-control mapping; a worked proof of historical access state | Dimension 3 — regulatory consequence |
| R6 | Owns the producer/consumer experience end to end | Time-to-first-value evidence; the consumer who declined and why | Dimension 5 — accepted an unpopular decision |
| R7 | Negotiates and holds the boundary against data modelling, analytics and source systems | A written boundary with a named owner on the other side (§2.3) | Dimension 1 — platform, not component |
| R8 | Works with, not around, the governance and model-risk functions | Lineage and quality evidence a supervisor could be shown | Dimension 6 — makes the compliant path the cheap path |

Read against §11.1, R5 and R8 are not optional extras in this institution: the standard's P3 requires aggregation "on a largely automated basis", P1 requires documented service-level standards and independent validation, and P2 requires the architecture to hold "not only in normal times but also during times of stress or crisis". A candidate who cannot connect their design choices to those obligations is under-qualified for *this* platform regardless of technical depth.

### 13.3 The two candidates against the framework

| Requirement | Candidate A — "the Platform Pod engineer" | Candidate B — "the modelling-track lead" |
|---|---|---|
| R1 Platform ownership | **Strong.** Built the ingestion framework now used by fourteen teams; led the table-format migration. Owns a platform artefact, though bounded to the ingestion and storage layers | **Partial.** Designs at estate level across domains, but each design is delivered by a programme and then handed over; no artefact is permanently theirs |
| R2 Contracts | **Medium.** Defined the framework's producer interface; has not run a platform-wide contract surface or a deprecation cycle with external consumers | **Strong on paper.** Writes interface and integration designs fluently; has produced contract specifications inside programmes |
| R3 Reliability as a service | **Weak.** Owns pipelines, not SLOs; is in the escalation path for their own components, not for the platform as a service | **Weak.** No production on-call ownership of any shared capability |
| R4 Cost | **Medium.** Has instrumented ingestion cost per team and argued one budget case | **Weak.** Has written business cases but never owned a consumption model |
| R5 Access and audit | **Weak.** Has implemented controls to a design owned by the security function | **Medium.** Comfortable with the control vocabulary; no evidence obligation of their own |
| R6 Consumer experience | **Medium-strong.** Ran the onboarding path for new teams onto the ingestion framework — real internal-product work | **Weak.** Delivery-facing rather than consumer-facing; has not owned an adoption outcome |
| R7 Boundary negotiation | **Medium.** Has de facto absorbed business rules into ingestion because the alternative was a delay — drift rather than negotiation (§2.3.3) | **Strong.** Naturally operates at the boundary and is used to arguing it with peer functions — but from a position with no owned consequences |
| R8 Governance interlock | **Medium.** Implemented lineage capture; has not produced evidence for an auditor or a supervisor | **Strong.** Understands regulatory obligations and speaks to risk functions comfortably |

**Against §6.2's six dimensions**, the summary is uncomfortable for both: Candidate A is senior on dimension 1 (size of problem owned) and dimension 4 (standard of artefacts) but not on dimension 3 (consequence carried), which is the load-bearing dimension in a regulated firm. Candidate B is senior on dimension 2 (ambiguity) and dimension 6 (shapes how others design) but has never carried dimension 3 either, and their dimension 1 is bounded by delivery rather than by ownership.

### 13.4 The gaps that matter, and the gaps that do not

**Gaps that matter for this role at this institution:**

1. **Consequence ownership (both candidates).** Neither has been the named owner of a spending commitment, a published SLO, or an audit obligation. R3, R4, R5 and R8 all fail on this single missing dimension rather than on knowledge — which is exactly §8.2's finding that the transition is won on consequence, not knowledge.
2. **The contract surface as a product (Candidate A).** They have built a framework; they have not run a product with versioned contracts and an external deprecation obligation (R2).
3. **Ownership unbounded by delivery (Candidate B).** Every artefact is a programme artefact. This is the classic solution-architect pattern from §8.1, and it is the reason that route is common on paper and rare in effect.

**Gaps that do not matter here, and should not drive the decision:**

- **Depth in machine learning platforms.** Neither candidate has it; §12 shows the AI-era change to this role is measured and contested, and the platform's ML consumer needs a substrate, not a specialist.
- **Certification.** §15 records the credentials and what they cover; no verified credential for this role exists, so its absence is not evidence against either candidate.
- **Vendor-specific fluency.** Both have it for parts of the stack. It is a cheap gap — it closes in weeks — and the framework in §3 is deliberately expressed in capabilities rather than products for this reason.
- **Streaming depth.** Material for the platform's roadmap but not the binding constraint on the hiring decision; both can reach design-level competence from the repository's streaming material (§3).

This ranking is the practical use of §3 and §6 together: the competency index says what the role needs, and the seniority model says which shortfalls are *closable by reading* and which are *only closable by ownership*. A hiring decision that weights them equally will pick the more fluent candidate and install the same failure mode the role is prone to (§10 vii).

### 13.5 The development plan for the candidate not selected

A hiring decision that discards a strong internal candidate without a plan is a retention failure. The plan below assumes Candidate B is not appointed and is deliberately built from §6.2 rather than from a course catalogue.

| Horizon | Action | Which §6.2 dimension it closes | How completion is observable |
|---|---|---|---|
| 0–1 month | Be named as the accountable owner of one platform capability end to end — including its cost line | Dimension 3 (consequence) | Their name is on the cost line and the escalation path |
| 1–3 months | Publish one SLO for that capability, with its first miss reported openly | Dimension 3, dimension 4 | A published miss exists |
| 3–6 months | Run one consumer intake and one refusal — decline a capability, in writing, with the reason | Dimension 5 (say no) | A written refusal that survived |
| 3–6 months | Write three ADRs on the platform's expensive tier, each stating reversal cost (§4.6) | Dimension 4, dimension 1 | ADRs that an outsider can act on |
| 6–9 months | Own the boundary negotiation against the source-system owners (§2.3), with a written boundary and a named counterpart | Dimension 1, dimension 6 | A signed boundary, not a slide |
| 9–12 months | Produce platform evidence for an internal audit or model-risk review | Dimension 3 | Evidence accepted by the second line |
| 9–12 months | Set one standard that other teams adopt without them in the room | Dimension 6 (the terminal marker) | Adoption without enforcement |

### 13.6 The assessment design

If the panel runs a loop, it should be designed to test §6, not to test knowledge. The repository's interview-technique guides (`system_design_interview_insiders_guide.md`, `nalsd_system_design_guide.md`) own *how* to run the design stages; the design here is what to *score*.

| Stage | Instrument | Scored against |
|---|---|---|
| Written exercise | An ADR for one expensive-to-reverse decision from the platform's real backlog, with alternatives rejected and reversal cost stated | §6.2 dimension 4; §4.6 |
| Design case | "Design the platform's second-generation ingestion and storage layer for the four consumer populations" — under-specified on purpose | §9.2 points 1–7 |
| Trade-off conversation | Handed a genuine conflict from the platform's backlog with no correct answer: cost against freshness, one platform against two | §9.3 |
| Stakeholder scenario | A source-system owner refuses change-data-capture; a consumer refuses to leave their own stack | §9.4; §2.3 |
| Depth probe | Two to three levels of "why" on their own written artefacts | §9.4; §4.2 |
| Panel review | Two practising senior architects ask whether they would hand over their platform | §10 vii |

The eight questions in §6.4 are the scoring substrate. A candidate answers the first four competently and the last four with names, numbers and a decision that cost something — or they do not, and the panel has its answer.

### 13.7 The cost comparison — illustrative arithmetic, not market data

**The figures in this table are invented.** They exist only to show how the comparison should be structured; they are not drawn from any source, they do not describe any market, and they must not be quoted as evidence about compensation anywhere. Assume a nominal unit of account.

| Cost element | Promote Candidate A | Promote Candidate B | Hire externally |
|---|---|---|---|
| Onboarding to the estate | Low — already knows sources, consumers, history | Low — already knows the estate | High — 3–6 months to comparable context |
| Ramp to consequence ownership | Moderate — needs R3, R4, R5 exposure | High — needs a platform artefact and consequence | Moderate — if hired from a comparable platform |
| Cost of the unfilled period | None | None | Vacancy cost from month 0 |
| Residual risk of the wrong appointment | Medium — capable, unfamiliar with the product surface | Higher — the §8.1 delivery-to-ownership transition is the known failure mode | Medium — external candidates are assessed on artefacts, which is the right test |
| Retention effect on the internal pool | Positive if A is appointed; negative if A is passed over without a plan (§13.5) | Negative for B | Negative for both |
| Acquisition cost | Internal — no premium | Internal — no premium | Includes a recruitment premium; **no figure is printed, because §7.4 established that none could be sourced** |

The structure is the deliverable. The only genuinely decision-relevant line is the last: an external hire is the only option whose cost includes a premium that cannot be quantified from any verified source, which is precisely why the comparison must be argued on the risk row as well as the cost rows.

### 13.8 The recommendation, and the honest case against it

**Recommendation: appoint Candidate A, with a structured consequence-ownership mandate in the first two quarters, and hold the appointment at "Senior Data Platform Architect (designate)" until §13.5's first three rows are closed.**

The reasoning:

1. **A already owns a multi-consumer platform artefact.** §8.2's evidence is that the market rewards this above everything else, and it is the one requirement neither external candidates nor Candidate B has yet demonstrated. The ingestion framework used by fourteen teams is exactly the artefact class the role is about.
2. **A's gaps are the closable kind.** R3, R4 and R5 are consequence exposures, not knowledge deficits, and §13.5's first three rows are achievable inside two quarters. B's gap — ownership unbounded by delivery — has historically taken a role change to close, not a development plan (§8.2 finding 2).
3. **A's boundary drift is correctable; the absence of appetite for ownership is not.** A absorbed business rules into ingestion because it was easier than negotiating (§13.4). That is a behaviour, and it is fixed by giving them the negotiation and watching what they do with it.

**The honest case for hiring externally instead:**

- **Neither internal candidate has carried the consequence the institution's obligations require.** In a firm where P2 of the standard requires the architecture to hold "during times of stress or crisis" and P1 requires independent validation, an architect who has never produced evidence for a second line or owned a spending commitment is a real risk, and the two quarters of mandate in §13.5 are the institution's *assumption* that this can be learned on the job.
- **External hires are assessed on artefacts, which is the correct test.** An external candidate arrives having already produced a roadmap, ADRs, SLOs and a cost model elsewhere — the §9.5 portfolio is what the loop actually scores, and an external candidate can present it on day one while an internal candidate presents a promise.
- **The internal appointment has a political cost.** Appointing A over B converts B's current relationship with the platform into a stake in its failure, and the panel should assume that the §13.5 plan for B is not a courtesy but a requirement for the appointment to work.
- **The honest counterweight:** the institution has no verified external market for this title (§7.4), so it cannot know what an external hire costs or how long the search takes. That uncertainty is an argument for testing the internal option with a bounded mandate rather than for either a quick appointment or an open-ended search.

---

## 14. One-page summary and action list

### 14.1 The role in one page

**The Senior Data Platform Architect owns a data platform as a product.** The object is the platform: its architecture and its evolution, its ingestion/storage/compute/serving layers as one coherent system, the interfaces and contracts it publishes, its reliability and performance as a service, its cost, its access model, and its consumers' experience of it. Not the business data model (the data architect's), not the analytics products (analytics engineering's), not the ML models (data science and model risk's), not the source systems (their owners'). The organising idea is that the platform's consumers are its customers and it has to earn their use — which is why the role's characteristic failures are product failures wearing architecture clothes (§10).

**What it takes.** Design-level competence across the technical index in §3, expert depth in the expensive-to-reverse rows; the architectural competencies in §4, of which trade-off reasoning and contract discipline are teachable and the calibration of evidence sufficiency is earned; and the organisational competencies in §5, which the repository's management shelf already covers and this guide merely points at.

**What "senior" means** (§6): not more knowledge — a bigger problem owned, more ambiguity tolerated, consequence actually carried, artefacts that survive outsiders, a demonstrated ability to say no, and the shift from designing systems to shaping the conditions in which systems get designed. Title is not a reliable signal in either direction.

**The market** (§7): the title is in active use across five distinct contexts, from cloud-native product firms to consultancies where the title is an engagement role. **No compensation figure could be printed for any geography**, including Singapore and Asia, because no source survived the standard in §7.4. **No verified credential exists for this role** (§8.2): The Open Group's Open CA certifies four specialisations, none of them data.

**The routes in** (§8): most commonly from data engineering, platform engineering or analytics engineering; possible from solution architecture but rarer than the title counts suggest, and won on consequence rather than knowledge.

**In a regulated firm** (§11): the standard's P2 is literally a platform clause — design for stress and crisis, not merely normal times — P1 requires documented service-level standards and independent validation, P3 pushes aggregation to "a largely automated basis" and reconciliation against sources, and every principle applies to outsourced processes. Residency, historical access evidence and operational resilience attach to the platform as consequences (§11.2).

**The AI era** (§12): measured effects are contested, they change on a shorter timescale than a platform programme, and they change the cost of tasks rather than the location of accountability. The parts of the role AI plausibly does not touch are the parts that carry consequence.

**The failure to guard against** (§10): building for the builders rather than the consumers, delivering architecture as documents, and owning a decision with no consequence attached. All three are survivable individually and fatal in combination.

### 14.2 The action list

1. **Take the decoder table (§1.4) to your own role.** Write down which of the eight titles you actually do, which of the seven ownership rows in §2.1 you actually own, and who owns the other side of each boundary. Most practitioners discover they own two of the seven and are called something else.
2. **Score yourself against the six dimensions in §6.2, not against a skills list.** If your evidence for dimension 3 (consequence carried) is empty, that — not a technology gap — is the binding constraint on a senior appointment.
3. **Answer §6.4's eight questions in writing.** Questions 5 to 8 are the ones that separate the levels; if you have no answer to question 7 (cost per capability) or question 8 (what is designed to be replaced), start there.
4. **Build one artefact from §9.5 this quarter.** An ADR for an expensive-to-reverse decision with a reversal cost, or a published SLO alongside its first miss. One is worth more than a certification, because it cannot be produced without ownership.
5. **Close one gap by ownership, not by reading.** Request the smallest platform capability you can get with real consumers and a real consequence attached. §8.2's finding is that this is the only mechanism that closes the seniority delta.
6. **For a regulated platform, get the obliging clauses before designing.** Read §11.1 and the repository's `../banking/risk_data_aggregation_guide.md`; obtain the residency, access-evidence and resilience constraints as design inputs, not as review findings.
7. **If you are hiring for this role: specify ownership, not technology** (§13.2), score the loop against §6 (§13.6), and treat a candidate with no consequence exposure as a bounded-risk appointment with a written mandate rather than as a finished senior architect.

### 14.3 Running this role analysis again (repeatable protocol)

The role moves faster than the framework. Re-run this analysis annually, or whenever the repository's shelf structure changes, in six steps mirroring the predecessor's protocol:

1. **Re-verify the title boundary.** Re-read the role definitions named in §1.4 and §15. The primary sources are dated; if a newer government or professional-body role definition exists, it supersedes the cited vintage and §1.4 should be rebuilt from it.
2. **Re-run the competency index.** `ls technology/data/ technology/architecture/ technology/ | grep -iE "contract|quality|observab|catalog|snowflake|warehouse"` — if a dedicated data-contract or observability guide now exists, the three honest gaps in §3 have closed and the index should say so.
3. **Re-test the compensation finding (§7.4).** The finding is an absence. It should be re-tested, not assumed: if a named source with a date, geography and methodology now publishes a figure for this title, print it with those attributes; if the only new material is another recruitment guide, keep the absence.
4. **Re-verify the standard (§11.1).** Check bis.org for a superseding or updated publication and re-read the section groupings from the current text rather than from this guide's citation.
5. **Re-grade the AI-era claims (§12.1).** The METR experiments state that their own results are out of date on a short cycle; the correct discipline is to re-read the current version rather than to accumulate stale findings.
6. **Re-score the seniority standard (§6.2).** Ask the same eight questions of the same people and compare the answers, not the CVs. The ladder moves on evidence, and evidence is the one input that cannot be inferred from a title.

**Stopping rule:** the analysis is current when §1.4's decoder still agrees with the primary sources, §7.4's absence has been re-tested rather than assumed, §11.1's clause citations still match the standard, and §12.1's citation dates are within their own stated currency windows. Anything older than that is a stale assessment wearing a confident voice.

---

## 15. The claims audit

Rows are the guide's load-bearing claims. **Status:** VERIFIED = established from the named primary source in this pass; FLAGGED = reported with an explicit limitation, uncertainty or source-type caveat; REJECTED = a candidate claim this guide declined to make. **Methodology** states what the source is and how it was read, because in this space the method is more informative than the figure.

| Claim | Status | Source | Date | Methodology | Quality note |
|---|---|---|---|---|---|
| The data architect role is defined, with three levels and eleven named skills, in an official capability framework | VERIFIED | UK Government Digital and Data Profession Capability Framework, "Data architect" role page | First published 7 Jan 2020; last updated 29 May 2026 | Government role definition, read at the level of role summaries and proficiency expectations | The strongest primary role definition found for the adjacent family; it covers data architect, not data platform architect — used as evidence of the *ladder shape*, not as a definition of this role |
| An employer publishes a requisition titled "Data Platform Architect" whose body describes delivery-level work under guidance, with a 5–10 year band and a 2-year technical floor | VERIFIED | Published employer requisition (identifier and location recorded in §1.4) | Accessed Sept 2026 | Employer's own published requisition, read directly | Single employer: evidence that the title is used and inflated, not evidence of the role's scope across the market |
| Three architect titles with three different objects appear in one posting | VERIFIED | Same requisition (§1.4) | Accessed Sept 2026 | Direct reading of a single requisition's role description | Same limitation as above — one document, strong because it is self-contained |
| A vendor-neutral definition of a platform as an "internal product" and of platform teams as "the thinnest reasonable layer" | VERIFIED | CNCF Platforms White Paper (TAG App Delivery); Evan Bottcher, martinfowler.com | White paper undated in the retrieved material; Bottcher 5 Mar 2018 | Practitioner/vendor-neutral body publication and a dated practitioner definition, read directly | The CNCF document's undated status is itself flagged: an undated body publication should be re-checked for a current version |
| The Open Group certifies Business, Digital, Enterprise and Solution Architects — no data specialisation | VERIFIED | The Open Group, Open Certified Architect programme description | Accessed Sept 2026 | Professional body's own programme pages | Read at programme level: the four specialisations are named; no data/data-platform specialisation is listed |
| **No widely recognised credential exists for the data platform architect role** | VERIFIED as a finding | The Open Group programme scope above; absence of any other issuing body in this research | Sept 2026 | Negative finding from the one relevant professional body read | A negative finding: a credential could exist that this research did not surface. Stated as "no credential found", not "no credential exists" |
| BCBS 239 comprises fourteen principles in four sections, eleven for banks and three for supervisors; Section I = P1–P2; Sections II/III/IV = P3–P6 / P7–P11 / P12–P14 | VERIFIED | BCBS, *Principles for effective risk data aggregation and risk reporting*, bis.org — **read directly from the published PDF** | Published 9 January 2013 (following a 26 June 2012 consultation) | Primary standard text; the annex 2 summary of the principles was read line by line | Confirms the guide's correction to the predecessor's "P1–P5 governance" grouping. ISBN recorded in the standard (92-9131-913-9 print; 92-9197-913-9 online) |
| BCBS 239 P2 obliges architecture and IT infrastructure that support aggregation "not only in normal times but also during times of stress or crisis" | VERIFIED | Same standard, Principle 2 text | 9 Jan 2013 | Direct quotation from the standard | Quoted from the primary text, not from a summary |
| BCBS 239 applies to risk management data and to all key internal risk management models, including Pillar 1, Pillar 2 and value-at-risk, and to processes outsourced to third parties | VERIFIED | Same standard, paragraphs 16–19 | 9 Jan 2013 | Direct reading of the scope paragraphs | Paragraphs, not inference |
| BCBS 239 is initially addressed to SIBs at group and solo level; G-SIBs designated Nov 2011/Nov 2012 by January 2016; D-SIBs strongly suggested three years after designation; supervisors may extend proportionately | VERIFIED | Same standard, paragraphs 13–15 | 9 Jan 2013 | Direct reading of the scope and timeline paragraphs | The "initially addressed to SIBs" wording is the standard's own; the guide does not overstate its breadth |
| MAS published *Data Governance and Management Practices* on 29 May 2024, setting out supervisory expectations based on BCBS 239 and thematic inspections of D-SIBs in 2022–23 | VERIFIED (date and themes) | Monetary Authority of Singapore information paper, mas.gov.sg | 29 May 2024 | Primary regulator publication, section headings read directly; date and scope corroborated by a law-firm client note of 25 June 2024 | The guide relies on the thematic headings and the date; it does not quantify the inspections' findings |
| The ECB's May 2018 thematic review found none of 25 significant institutions had fully implemented the principles | FLAGGED — delegated | Repo's `../banking/risk_data_aggregation_guide.md` §8 (itself citing the ECB report) | ECB report May 2018; repo guide 2026 | Cross-reference to the repository's existing verified treatment | Not re-verified against the ECB source in this pass, only against the repository's treatment; the guide attributes it to the repository and says so (§11.1) |
| METR's randomised trial found a 19% increase in completion time for experienced developers when AI was allowed, against a 24% forecast speed-up | VERIFIED with caveats | METR randomised controlled trial, arXiv 2507.09089 | July 2025 | Experimental study: 16 developers, 246 real issues, randomised allow/disallow, screen recording, self-reported time | The authors state the sample is unrepresentative of most software work and that the result does not show AI fails elsewhere; the guide reproduces those caveats |
| METR's later material reports a median self-reported change of 1.4–2× in the value of technical workers' work, and METR cautions this is self-reported and likely overstated | FLAGGED | METR follow-up (documented experimental update and a survey of 349 technical workers) | 24 Feb 2026 and 11 May 2026 | Self-reported survey with stated sample size; the issuing body itself cautions on magnitude | Cited as self-reported survey evidence, explicitly not as measurement |
| DORA 2024 reports AI adoption significantly increases individual productivity, flow and job satisfaction while negatively impacting delivery stability and throughput; and that internal developer platforms improve productivity and performance while potentially decreasing change stability | FLAGGED — vendor-sponsored research | DORA / *Accelerate State of DevOps 2024*, published by Google Cloud with named sponsors | 2024 | Industry survey research with a stated methodology, published under a sponsor | **Cited with the sponsorship stated**, and read at the level of the published findings summary rather than the full report |
| The repository contains no guide on AI-era data-platform operations, and no dedicated guides on data contracts, data quality/observability/cataloguing platforms, or managed cloud data warehouses | VERIFIED | Repository scan (`technology/`, `technology/data/`, `technology/architecture/`) plus the predecessor's §0 | Sept 2026 | Direct file-listing scan; corroborated by the predecessor guide's own gap analysis | A negative finding about this repository, recorded as a gap rather than papered over |
| The phrase "data platform architect" appears in no file in this repository before this guide | VERIFIED | Repository-wide case-insensitive grep over `*.md` | Sept 2026 | Mechanical absence check (0 files) | Stated as an absence in the tree, not as a claim about the world |

### 15.1 The highest-risk classes, audited separately

**Compensation.** Every candidate figure was either gated, arithmetically implausible, unsourced or undated, and **no figure is printed anywhere in this guide**. The rejections are recorded in §7.4: a recruitment firm's survey behind a gated form (a marketing document, no figure visible); a crowd-sourced page returning an annually-rendered monthly figure that fails arithmetic sanity; an undated "Cloud Data Architect" band on a job-description content site; and a government-statistics-derived newspaper salary guide whose occupational vocabulary does not map onto this title. **Status of the class: no source met the standard; the finding is the absence.**

**Market demand.** No vendor-neutral index of demand for this exact title was found. Searches returned job-board aggregators and content sites with no stated methodology. Any "growing demand" claim is therefore marked unverified projection (§7.3). **Status: flagged; no demand figure asserted.**

**Title definitions.** Three primary sources carry the decoder (§1.4): the UK government capability framework, the employer requisition, and The Open Group's programme descriptions. The universal limitation is that a definition from one organisation is evidence of *that organisation's* usage, so the decoder is built from several and the interchangeability caveat (§1.4) is part of the finding rather than a footnote. **Status: verified for the cited sources; generalisation explicitly not claimed.**

**Credentials.** The only credential claim is a negative finding supported by The Open Group's programme scope. No certification for this role is asserted to exist, and no course, badge or framework is invented. **Status: verified as a finding, with the research-boundary caveat stated.**

**Regulatory provisions.** All BCBS 239 content was read from the standard's own text and its annex 2 summary, with the section groupings verified rather than inherited. The MAS paper's date, scope and themes were read from the regulator's page. The ECB thematic-review figure is delegated to the repository's existing treatment and labelled as such. **Status: verified from primary text, except the delegated ECB figure, which is flagged.**

---

## 16. What Could Not Be Verified, the glossary, the cross-references and the closing

### 16.1 What Could Not Be Verified

Stated plainly, because a guide that hides these is less useful than one that lists them:

1. **Compensation, for any geography.** No source met the standard of named source + date + geography + methodology. Singapore and Asia — the geographies most relevant to this series — are included in the absence, and no figure is printed for any market rather than a bad figure for one (§7.4). This is the guide's largest single gap and it is a gap by decision, not by oversight.
2. **A market-demand index for this exact title.** None found; job-board aggregators and content sites carried no stated methodology (§7.3).
3. **A vendor-neutral published levelling framework for this role.** None found. The UK framework's ladder for the adjacent data-architect role was used as the nearest official instrument, with the substitution stated (§6.1).
4. **A credential for this role.** None found (§8.2 and §15.1) — a finding, not an omission, with the caveat that a credential may exist that this research did not surface.
5. **The current text of the CNCF Platforms White Paper as a dated document.** The retrieved material was undated; the definition was therefore used as a body publication with the date flagged (§15).
6. **The ECB thematic review's findings at first hand.** Attributed to the repository's existing treatment rather than re-verified against the ECB publication (§11.1, §15).
7. **Any published evidence on how AI tooling changes data-platform work specifically, as distinct from software engineering generally.** The studies found measure software engineering; the platform-specific intersection — assisted pipeline authoring, natural-language access, agentic platform operation — is covered in this repository by no guide and in the research by no measured study (§12.1). The displaced/augmented/untouched split is labelled as projection for exactly this reason.
8. **Whether "data platform architect" is a stable title or a transient one.** No source in this research addresses title stability. The decoder in §1.4 describes current usage; it does not forecast.
9. **Tool limitation, recorded as such:** web_search returned empty result sets for several queries in this run, including queries on this title's demand and on the standard's section structure. Failures of this kind are a limitation of the search path, **not evidence of absence** — where a search came back empty, the claim was pursued through web_extract on primary URLs (bis.org, mas.gov.sg, the professional bodies, the employer requisition) and, where that also produced nothing, no claim was made.
10. **The internal structure of the two Cymbal Bank candidates and the §13.7 figures.** Fictional by construction and labelled as such throughout §13.

### 16.2 Glossary

| Term | Meaning in this guide |
|---|---|
| **Platform as a product** | The organising idea of the role: the platform has internal customers, and it must earn their use rather than assume it (§1.2) |
| **Platform capability** | One discrete thing the platform offers its consumers — ingestion, a storage layer, a catalog, a query surface — with its own guarantee and its own cost (§2.1) |
| **Consumer** | Any team or person building on the platform: data engineers, analytics engineers, analysts, data scientists, application teams, risk and reporting functions (§2.1) |
| **Contract surface** | The set of versioned interfaces the platform publishes, with compatibility promises and a deprecation policy (§4.3) |
| **NFR (non-functional requirement)** | Freshness, availability, latency, cost, recovery objectives, access evidence — the properties platforms are actually judged on, each attached to a capability and a consumer (§4.2) |
| **Reversal cost** | The cost of undoing an architectural decision; the sorting criterion for how much political capital a decision deserves (§4.6) |
| **Reference architecture vs working architecture** | A statement of intent versus the thing people use. Where they diverge, the reference architecture is routed around (§4.5) |
| **Expensive tier** | The class of decisions that are very expensive to reverse — storage layout and table format, catalog, identity and access model, residency topology, event bus, tenancy (§4.6) |
| **Thinnest reasonable layer** | The CNCF white paper's discipline for platform teams: present capabilities without implementing everything yourself (§5) |
| **Platform SLO** | A published reliability or performance objective for a platform capability, with its misses published too (§2.1, §6.2) |
| **BCBS 239** | Basel Committee standard 239, *Principles for effective risk data aggregation and risk reporting*, January 2013: fourteen principles in four sections, eleven for banks and three for supervisors (§11.1) |
| **D-SIB / G-SIB** | Domestic / global systemically important bank — the designations that set the standard's application and its deadlines (§11.1) |
| **Pillar 2 / SREP** | The supervisory review process through which data-aggregation deficiencies can become capital consequences (§11.1, delegated to the repository's treatment) |
| **Evidence class** | The explicit labelling of whether a claim is measured, surveyed, self-reported, vendor-sourced or projected — used throughout §12 to keep them apart |
| **Title inflation** | The same title covering wildly different scope across firms; the guide's position is that titles are not portable and the work is (§1.4) |

### 16.3 Cross-references

**Within this series (the two predecessors, cited by name rather than re-derived):**

- `data_architect_skillgaps_guide.md` — the first personalised role analysis. Its ten assessment axes (§1), digital-profile heatmap (§2), ranked gap list (§4) and learning plan (§5) are the machinery this guide deliberately does not rebuild. Where this guide names the data architect's objects (§1.3, §2.2) or carries forward its three honest gaps (§3), it points there.
- `architecture/skill_gaps_enterprise_architect_guide.md` — the second. Its banking-domain gap analysis (§3) applies unchanged to a platform architect in a regulated firm (§8.3), and its enterprise-influence frame is the other half of the boundary in §1.3.

**The shelves this guide does not re-derive:**

- **Data technologies** — `data/enterprise_data_platforms_guide.md`, `data/mpp_databases_guide.md`, `data/data_fabric_guide.md`, `data/on_prem_lakehouse_guide.md`, `data/delta_lake_vs_iceberg.md`, `data/paimon_iceberg_delta_comparison.md`, `databricks_guide.md`, `clickhouse_guide.md`, `data/elasticsearch_data_modeling_schema_design.md`, `data/alibaba_data_platform.md` — indexed in §3, not taught here.
- **Architecture frameworks and the EA discipline** — `architecture/enterprise_architecture_guide.md`, `architecture/togaf_guide.md`, `architecture/capability_engineering_guide.md`, `architecture/govtech_architecture_principles.md`, and `architecture/why_tech_bypasses_togaf_guide.md`, which is cited by name in §4.5 and §5 as the repository's treatment of reference architectures that get routed around.
- **Distributed systems and streaming** — `distributed_systems_engineering_guide.md`, `event_stream_processing_guide.md`, `apache_flink_guide.md`, `kafka_alternatives_guide.md`, `kafka_virtualization_guide.md`.
- **Governance, lineage, quality** — `data_governance_guide.md`, `data/data_governance_framework.md`, `data/data_lineage_tools.md`, `data/data_profiling_guide.md`, `data/data_compliance_frameworks.md`, `data/china_data_governance_frameworks.md`.
- **Platform engineering, cost and capacity** — `charmed_kubernetes_vs_openshift_guide.md`, `ai_platform_engineering_guide.md`, `finops_guide.md`, `capacity_sizing_guide.md`, `opensearch_capacity_planning_guide.md`.
- **Orchestration and pipeline operations** — `apache_airflow_guide.md`, `data/data_pipeline_guide.md`, `data/data_pipeline_versioning.md`, `data/dataops_guide.md`, `data/backfill_data_engineering.md`, `schema_evolution_data_drift_guide.md`.
- **Decisions and interfaces** — `architecture_decision_record_guide.md`, `api_governance_guide.md`, `data_integration_frameworks_guide.md`.
- **Interview technique** — `system_design_interview_insiders_guide.md`, `google_system_design_interview_guide.md`, `nalsd_system_design_guide.md`, `grokking_system_design_companion_guide.md`, `ml_system_design_interview_guide.md` — cited in §9 as the technique this guide does not re-derive.
- **The management shelf** — `../management/vendor_management_guide.md`, `../management/communication_stakeholder_management_skills_guide.md`, `../management/authority_skills_guide.md`, `../management/business_case_development_guide.md`, `../management/reverse_job_search_guide.md` (the search *method*), `../management/the_first_90_days_guide.md` (the onboarding *method*).
- **The banking and regulatory shelf** — `../banking/risk_data_aggregation_guide.md` (the repository's deep treatment of the standard, cited in §11.1), `../banking/operational_resilience_framework_guide.md`, `../banking/financial_risk_compliance_systems_guide.md`, `../banking/risk_management_models_guide.md`.

### 16.4 Closing

This guide set out to do two things the two predecessors do not: to analyse a *different* target role — the data platform architect, whose object is the platform and not the enterprise's data models or its target-state architecture — and to supply the *seniority* dimension that a readiness percentage needs in order to mean anything. Neither predecessor asked what changes between mid and senior; §6 is this guide's answer, and it is the standard against which §9's interview, §13's worked example and the reader's own self-assessment are scored.

The other thing this guide has tried to do is stay honest where the evidence runs out. No compensation figure is printed, for any market, because none could be sourced to the standard the role deserves — and a guide that prints a recruitment firm's number as if it were a measurement is a guide that has already failed at the one thing a platform architect must be able to do, which is to say what is actually known. The same discipline runs through §12, where every AI-era claim carries its source type, and through §15, where the load-bearing claims are audited one by one with their dates and their limitations attached.

What remains is the reader's to close, and §14.2 is deliberately unsentimental about it: the technical content can be read, the architectural vocabulary can be learned, and the repository already holds the substance for both — but the six dimensions in §6.2 are evidenced only by things a person has actually done. The route in §8 that matters is therefore not a study plan; it is the smallest available piece of platform ownership with real consumers and real consequences attached. Everything in this guide is downstream of that, and the reason is the one organising idea the role cannot be practised without: nobody has to use your platform, and the consumers you were built for will tell you the truth faster than any architecture review ever will. Own the platform as a product, and the architecture has a reason to exist. **The platform is the product.**
