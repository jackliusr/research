# Talend — The Vendor, the Platform, and the Estate Decision

> **Author:** Jack Liu Shurui, Solution Architect
> **Generated:** 2026-09-14
> **Scope:** A vendor deep-dive on **Talend** — the data-integration and data-management platform now owned by **Qlik**, itself owned by Thoma Bravo: what the company is, how the platform is actually built, what the editions and the capacity-metered licensing gate, what the Qlik era has promised versus what it has shipped, and what a regulated institution should do with an existing Talend estate. Facts marked **(verified)** were confirmed against primary vendor/project sources in **September 2026**, with the source and date given inline for every ownership, version, licence and lifecycle claim, because those are the claims that go stale fastest. Items that are ambiguous, unpublished, secondary-sourced or simply unknown are marked **(flagged)** and are collected in §11.
> **Out of scope:** the *discipline* — integration patterns, CDC mechanics, data-quality theory and the vendor-selection framework — which belongs to [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md), and the scheduler landscape, which belongs to [control_m_guide.md](control_m_guide.md). This guide owns the **vendor**.

**Related guides in this series:** [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (the DI discipline: patterns, its §3 tool comparison, §6 CDC, §7 data quality, §11 tool matrix, §12 decision tree — read it for *how to choose*, not for who Talend is) · [data_governance_guide.md](data_governance_guide.md) (the governance operating model this guide's §7 sits under) · [data/data_lineage_tools.md](data/data_lineage_tools.md) (lineage tooling landscape and evidence models) · [data/data_profiling_guide.md](data/data_profiling_guide.md) (profiling practice, which Talend Studio also implements) · [data/data_fabric_guide.md](data/data_fabric_guide.md) (data-fabric architecture — the architectural pattern Talend markets its platform into) · [control_m_guide.md](control_m_guide.md) (the scheduler landscape that actually triggers most Talend estates) · [databricks_guide.md](databricks_guide.md) (the warehouse/lakehouse side of the same pipelines) · [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (the API/ESB lane Talend competes in with Routes and Data Services) · [architecture/enterprise_architecture_guide.md](architecture/enterprise_architecture_guide.md) (where a platform of this kind sits in an architecture baseline) · [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (the operational-resilience anchor behind §7 and §9) · [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) (the risk/compliance system context)

---

## Table of Contents

1. [The Overview and the Identity](#1-the-overview-and-the-identity)
2. [The Platform Architecture, Verified](#2-the-platform-architecture-verified)
3. [The Editions and the Licensing](#3-the-editions-and-the-licensing)
4. [The Cloud Platform, Verified](#4-the-cloud-platform-verified)
5. [The Qlik Era: Promised Versus Shipped](#5-the-qlik-era-promised-versus-shipped)
6. [The Competitive Position](#6-the-competitive-position)
7. [The Regulated-Enterprise and Banking Angle](#7-the-regulated-enterprise-and-banking-angle)
8. [The Estate Decision](#8-the-estate-decision)
9. [The Cymbal Bank Worked Example](#9-the-cymbal-bank-worked-example)
10. [The Claims Audit](#10-the-claims-audit)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [Cross-References and Further Reading](#13-cross-references-and-further-reading)
14. [Closing Summary](#14-closing-summary)

---

## 1. The Overview and the Identity

Talend is a **data-integration and data-management platform vendor** — the class of software that moves data between systems, transforms it on the way, and layers quality, cataloguing and stewardship on top. In the taxonomy of the discipline guide it is a *classical, design-time-authoring, code-generating* integration platform that has spent the last decade trying to become a cloud-native platform: strong on breadth of connectors and on data quality, historically weak on the operating model that a large estate needs once the number of pipelines passes the point where a human can remember them. For the pattern-level comparison against Informatica, IBM DataStage, SSIS, dbt, Fivetran, Matillion, Airbyte and Apache Hop, §6 of this guide gives the positioning table only; the *selection* framework is [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) §§3, 11 and 12 and is not rebuilt here.

What makes Talend a vendor worth a dedicated guide in 2026 is not its connector count. It is that **Talend has been owned three times in five years** — public, then private equity, then folded into a larger private-equity-backed data company — and that the ownership history is the single most useful predictor of what an existing Talend estate can expect. Each transition changed what the product *is*, what it is called, and what is safe to build on it.

### 1.1 The ownership history, laid out and dated

| Era | Event | Date | Source |
| --- | --- | --- | --- |
| 2005 | **Founded** in **France** by **Bertrand Diard** and **Fabrice Bonan** as a commercial open-source data-integration vendor | late **2005** | **(verified** — corroborated by two independent secondary sources read 2026-09-14: Computerworld's 2009 interview with Diard ("In late 2005, when Talend was founded, the data integration market was dominated by a handful of proprietary…") and Yahoo Finance's 2016-07-29 IPO report ("It was founded in 2005 by Fabrice Bonan and Bertrand Diard"). No first-party corporate-history page remained online at verification; see §11 item 1.) |
| 2006 | **Talend Open Studio for Data Integration** released as a free, open-source download — the origin of the commercial motion: give away the design tool, sell the enterprise platform, support and scale | 2006 | **(flagged)** — secondary encyclopaedic sources only; the 2006 launch date could not be confirmed at a first-party source. What *is* verified is the existence and duration of the free distribution: "Over the last 20 years, Talend has provided a free open-source version of Talend Studio" **(verified** — qlik.com/us/products/talend-open-studio, read 2026-09-14) |
| 2016-07-28 | **IPO.** Talend S.A. priced its Nasdaq IPO at **$18.00 per ADS**, 5,250,000 ADSs, ticker **TLND** | 2016-07-28 | **(verified** — "Talend Announces Pricing of Initial Public Offering", GlobeNewswire, 2016-07-28) |
| 2016-08-03 | **IPO closing**, including full exercise of the underwriters' over-allotment: **6,037,500 ADSs** at $18.00, raising approximately **$94.5 million** | 2016-08-03 | **(verified** — "Talend Announces Closing of Initial Public Offering and the Full Exercise of the Underwriters' Option", GlobeNewswire, 2016-08-03; the $94.5M figure also reported by Fortune, 2016-07-28) |
| 2016-07-29 | First-day trading opened at **$27.66**, up ~54% on the IPO price | 2016-07-29 | **(verified** — TechCrunch, 2016-07-29) |
| 2021-03-10 | **Take-private announced**: Thoma Bravo to acquire Talend | 2021-03-10 | **(verified** — thomabravo.com press release "Thoma Bravo Completes Acquisition of Talend", read 2026-09-14, which records the March 10 2021 announcement) |
| 2021-07-29 | **Tender offer completed** by Tahoe Bidco B.V., a Thoma Bravo affiliate | 2021-07-29 | **(verified** — same source; tender-offer release dated 2021-07-29) |
| 2021-08-09 | Subsequent offering period concluded | 2021-08-09 | **(verified** — same source) |
| 2021-09-02 | **Take-private completed.** All-cash transaction valuing Talend at **approximately $2.4 billion**; shareholders received **$66.00** per ordinary share/ADS, a ~**29%** premium to the 2021-03-09 close and ~**81%** premium to the trailing twelve-month VWAP. CEO at the time: **Christal Bemont** | 2021-09-02 | **(verified** — thomabravo.com press release, dated 2 September 2021, read 2026-09-14) |
| 2023-01-05 | **Qlik announces its intention to acquire Talend** from Thoma Bravo — "two Thoma Bravo-backed industry leaders". Explicitly: "Terms of the proposed transaction were not disclosed." Expected to close in H1 2023 subject to works-council consultation and regulatory approval. Talend then described as serving "over **7,250** customers" | 2023-01-05 | **(verified** — thomabravo.com / GlobeNewswire, "Qlik Intends to Acquire Talend…", dated 5 January 2023, read 2026-09-14) |
| 2023-05-16 | **Qlik acquisition closed.** "Qlik today announced it has closed its acquisition of Talend." Combined entity led by **Mike Capone**; Qlik describes the combined base as "more than **40,000** active customers in over 100 countries" | 2023-05-16 | **(verified** — qlik.com press room, "Qlik Acquires Talend…", GlobeNewswire dateline 2023-05-16, read 2026-09-14) |
| — | **Reported deal value of the Qlik transaction (~$2.4bn)** | — | **(flagged)** — the primary announcement states terms were **not disclosed**; the ~$2.4bn figure circulates only in secondary press. The *Thoma Bravo* transaction's ~$2.4bn valuation is separately and properly sourced; do not conflate the two (§10, §11 item 2) |

Two things fall out of that table and they matter more than any feature list.

**First, the deal-value folklore is real and it is wrong.** Both the 2021 take-private and the 2023 Qlik acquisition are widely reported at "~$2.4 billion", and the second one is widely reported as if the number were official. It is not: Qlik's own release says terms were not disclosed. The correct reading is that Thoma Bravo bought Talend for approximately $2.4bn in 2021 and, two years later, sold it to another portfolio company at terms its own press release declined to state.

**Second, the customer-count claims are the vendor's and they are not consistent with each other.** Thoma Bravo's September 2021 release said "over 6,500 customers"; GlobeNewswire's January 2023 release said "over 7,250"; Qlik's May 2023 release quoted 40,000+ for the *combined* entity (Qlik alone had "more than 38,000" in the January 2023 release). None of these is a like-for-like number in the sense an auditor would want, and an existing customer should not use any of them as a proxy for the population of Talend estates still running.

### 1.2 What problem class Talend actually serves

Stated at the level a solution architect needs, Talend occupies three adjacent lanes and has always been strongest in the first:

- **Batch ETL/ELT integration** — extract from sources, transform, load to warehouses/lakes, orchestrated on a schedule. This is the historical core: the Studio job, the component palette, the generated artefact, the scheduler task.
- **Data quality and governance** — profiling, rule-based validation, cleansing, masking, stewardship workflows, cataloguing and (in the Qlik era) lineage pushed into Qlik Cloud's catalog. Talend's analyst standing in data quality has for years been as strong as its standing in data integration **(verified** — Qlik/Talend market its Gartner standing in both categories together: "For ten years, Gartner® has positioned Qlik® as a Leader in its Magic Quadrant™ for Data Integration Tools" and "Leader in the 2026 Gartner® Magic Quadrant™ for Augmented Data Quality Solutions for the seventh time", talend.com/products, read 2026-09-14).
- **Application/API integration** — Routes and Data Services, API design and testing, microservice packaging. This lane exists but is **licence-gated** to the higher subscription products, and it is the lane in which the platform is least differentiated against purpose-built API gateways and iPaaS.

**Where it sits in a bank.** In a regulated estate, Talend typically shows up as the **batch integration backbone**: the thing that feeds the risk data mart, moves reference data between core systems, reconciles ledgers and runs the data-quality gate before a downstream reporting layer consumes anything. That placement — high criticality, high regulatory visibility, moderate engineering headcount — is exactly why the estate decision in §8 is not a technology preference question.

---

## 2. The Platform Architecture, Verified

Talend's architecture is easiest to hold in the head as **a design-time tool plus a code generator, a cloud-or-self-hosted control plane, and a set of execution engines that are deliberately distinct from the design tool**. Everything else — catalogs, quality modules, API tooling — hangs off that spine. The names have changed repeatedly, so this section gives the *current documented* names first and treats the older ones as fossils (§12 has the decoding table).

### 2.1 Design time versus run time

**Design time is Talend Studio.** Talend's own description is precise and worth quoting in full: "Talend provides you with a range of **subscription Talend Studio licenses** you can use to create your projects and manage data of any type or volume. Using the graphical User Interface and hundreds of pre-built components and connectors, you can design your **Jobs or Routes** with a drag-and-drop interface and **native code generation**." **(verified** — help.qlik.com Talend Studio User Guide, "What is Talend Studio?", last updated 2026-08-27, read 2026-09-14).

Three verified properties of Studio follow from its own documentation and are worth naming because they carry operational consequences:

- **It is licence-gated.** "The availability of the perspectives depends on your license in the case of a local project, or on the type of your remote project." **(verified** — same page.) A developer on a lower-tier entitlement sees a different Studio than a developer on Data Fabric.
- **It is Eclipse-lineage tooling.** The current patch documentation for Studio references an Eclipse **p2** update option (`-Dtalend.studio.p2.update`) for CI builds, a **Feature Manager** wizard for installing features, an Eclipse-style **workspace** folder, and errors surfacing in `<Studio_Home>\workspace\.metadata\.log** **(verified** — Studio cumulative patch note R2026-08, update.talend.com, read 2026-09-14). The commonly repeated statement that "Talend Studio is Eclipse-based" is therefore *supported by the tooling's own artefacts*; a current first-party sentence saying so verbatim was not located, which is why §10 records it as verified-by-tooling rather than quoted **(flagged)** for the literal phrasing.
- **It is a desktop application with a server-side project store.** Studio works in local projects or remote projects; the documentation set covers "Managing repository items", "Searching a Job in the repository", "Working with referenced projects", "Managing Job and Route versions", "Comparing Jobs", "Documenting a Job or a Route", and managing items on **different branches and tags** **(verified** — Talend Studio User Guide index, "Managing Jobs, Routes and Services", last updated 2026-08-27, read 2026-09-14). The repository is therefore a **design-time metadata store under version control**, not the runtime.

### 2.2 The Job / Component model and the code-generation question

A **Job** is the unit of integration design. It is assembled from **components** dragged from a palette, wired together, with **subJobs** and joblets providing internal structure; **Routes** and **Data Services** are the application-integration variants built from a different component set **(verified** — the Studio documentation enumerates "Activating/Deactivating a component or a subJob or a Route", the Data Integration vs Data Services split, and "Testing Jobs and Services using test cases" / "Testing Routes using test cases"; Studio User Guide, read 2026-09-14). Routes and services are available only in the higher subscription products — the documented list is **Talend ESB, Talend Data Services Platform, Talend Real-Time Big Data Platform, Talend MDM Platform, Talend Data Fabric, Talend Cloud API Services Platform, Talend Cloud Data Fabric** **(verified** — same page).

The code-generation model is the part most often mis-stated, in both directions. What the vendor documents, and what it does not:

**What is verified:** Studio produces **build artefacts you can run without Studio**. Building a Job produces a **`.jar`** by default, or a `.zip` with an external config folder and Linux/Windows start scripts when you choose that option, and the documented result is: "A .jar or .zip file for the Job is created in the defined place. **You can run it independently of Talend Studio in standalone mode.**" **(verified** — Studio User Guide, "Building a Data Service Job as a Microservice", last updated 2026-08-27, read 2026-09-14.) Jobs can also be published to an artifact repository, built as **Docker images**, and built as **Microservices** with a Prometheus metrics endpoint **(verified** — the same page plus the Studio documentation index entries "Publishing a Job as a Docker image" and "Publishing to an artifact repository", read 2026-09-14).

**What is verified about the runtime stack underneath:** since Studio **R2025-02**, artefacts are built with **Java 17** and **Camel 4**, and Talend Runtime must be at **8.0.1-R2025-02-RT or later** to run them; **Java 21 is mandatory to run Studio from R2026-06 onwards**; the Studio default Docker image was updated to `eclipse-temurin:11`; and the CI builder must be Maven 3.6.3+ **(verified** — Studio cumulative patch note R2026-08, patch `Patch_20260821_R2026-08_v1-8.0.1`, release date 2026-08-21, read 2026-09-14). "Data Services and Routes built as OSGi artefacts with R2025-02 and later are only compatible with Talend Runtime 8.0.1-R2025-02-RT and later", and Talend "highly recommend[s] for Talend Runtime users to keep Studio and Runtime versions in sync. **Using unaligned versions is at risk.**" **(verified** — same source). That last sentence is the single most operationally important line in the whole documentation set for anyone running client-managed Talend.

**What is not documented and circulates anyway:** the mechanics of the generator itself. Talend's community folklore — stated by a practitioner on Qlik's own forum — is that "Talend Data Integration studio, more than anything else, **is a code generator**… Talend combines {javajet template, xml, properties} files of the job and generates a **single Java file** for the job", with **JavaJet** templates interpreted to Java source **(flagged** — Qlik Community thread "The process of Java code generation from javajet templates of a job", posted 2016-09-29, read 2026-09-14; a community answer, not vendor documentation). Talend does not publish the generation process as product documentation, and a second community responder's view was that programmatic job generation is possible "but it is tricky and wouldn't be supported by Talend" **(flagged** — same thread, 2016-10-10).

**Why this matters to an architect, in plain terms.** Because the artefact is a compiled Java application, a Talend job is:

- **Debuggable at the Java layer** — you can attach, log, and inspect the generated code — but the *design of record* is the Studio repository object, not the Java, so a code review of the Java reviews the generator's output, not the author's intent. This is a real separation-of-duties problem and §7 returns to it.
- **Portable as a runtime, not as a design.** The `.jar` runs anywhere with a JVM; the job cannot be maintained, diffed or re-versioned without Studio and a valid licence. Exit cost is therefore concentrated in the *maintenance* path, not the *execution* path.
- **Version-coupled in ways that surface at exactly the wrong time.** Java 17 builds, Camel 4, Java 21 Studio, and the Studio/Runtime alignment warning above mean an upgrade is not a Studio upgrade; it is a coordinated change across developer desktops, CI builders, runtime hosts and any container images built from the *old* artefacts.

### 2.3 The execution engines, under their current names

| Engine | What it is | Current version/state, with date |
| --- | --- | --- |
| **Talend JobServer** | The classic client-managed execution service used with Talend Administration Center | Exists and is version-aligned with Administration Center; documentation recommends same version as TAC **(verified** — "Talend modules compatibility", help.qlik.com Customer Support Statements, last updated 2026-09-09) |
| **Talend Runtime** | The OSGi/Apache-Camel-based container that runs Routes, Data Services and Microservices; must match Studio generation version | Must be `8.0.1-R2025-02-RT` or later for artefacts built from R2025-02 **(verified** — Studio patch note R2026-08, 2026-08-21) |
| **Talend Administration Center (TAC)** | The client-managed web console: projects, users, scheduling, software-update repository for on-premises estates | Exists; "Talend Administration Center compatibility" is a documented topic and the patch instructions reference its Settings → Software Update repository **(verified** — Studio patch note R2026-08 and "Talend modules compatibility", 2026-09-09) |
| **Talend Remote Engine — Gen1** | The customer-hosted engine that runs Studio-built tasks published from Talend Cloud | **2.14** GA **January 2026**; **2.13** GA October 2023, retired January 2026, **end of life 2027-06-30** **(verified** — "Talend Remote Engine lifecycle", help.qlik.com, last updated 2026-09-09) |
| **Talend Remote Engine — Gen2** | The engine used with **Talend Cloud Pipeline Designer**, running pipelines and preparations next to the data | **R2026-03** GA March 2026; "used with Talend Cloud Pipeline Designer only for connections and datasets" in its Gen2 role **(verified** — "Talend Remote Engine lifecycle", 2026-09-09) |
| **Cloud Engine for Design** | A small engine embedded by default in Talend Management Console so users can try cloud apps immediately | "come[s] embedded by default in Talend Management Console to help users quickly get started… but it is recommended to install the secure Remote Engine Gen2 for advanced processing of data" **(verified** — Talend Remote Engine Gen2 Quick Start Guide, "Remote Engine Gen2 architecture", last updated 2026-09-09) |
| **Qlik Data Gateway — Data Movement** | Qlik-side gateway for replication/CDC when the customer hosts the movement path | Referenced as the mechanism higher tiers use for continuous/CDC replication **(verified** — Qlik Talend Cloud subscription options, help.qlik.com, last updated 2026-09-11) |

**The naming trap.** "Remote Engine Gen1" and "Remote Engine Gen2" are **not versions of the same thing**: Gen1 is the engine that runs Studio-built tasks published to Talend Cloud; Gen2 is the engine for Pipeline Designer pipelines. Their lifecycles are separate and their release trains are different (Gen1 uses `2.x`, Gen2 uses `RYYYY-MM`). Any upgrade plan that treats "the Remote Engine" as one component is already wrong.

### 2.4 The control plane and the suite modules, under current names

| Module (current documented name) | Role |
| --- | --- |
| **Talend Studio** | Design-time desktop; jobs, routes, data services; licensed per subscription tier |
| **Talend Management Console (TMC)** | Cloud control plane: "managing and running tasks and plans, administrating Studio projects, users, and user roles, and managing execution engines" **(verified** — TMC User Guide, "What is Talend Management Console?", last updated 2026-09-09). Includes Users & Security, Studio projects, Environments/spaces, Promotions, Processing (engines and run profiles), Configurations (static IP, log export), Operations and Management (schedule, execute, monitor) |
| **Talend Administration Center (TAC)** | Client-managed equivalent for on-premises estates |
| **Talend Cloud Pipeline Designer** | Low-code cloud pipeline authoring; runs on Gen2 engines |
| **Talend Cloud Data Inventory** | The dataset collection/inventory app; the shared dataset service |
| **Talend Cloud Data Preparation** | Self-service cleansing/preparation |
| **Talend Cloud Data Stewardship** | Stewardship workflows (Enterprise tier in the Qlik nomenclature) |
| **Talend Cloud API Designer / API Tester** | API design and testing (Premium tier and above) |
| **Talend Data Catalog** | The catalog product, with its **own lifecycle** separate from Talend 8 |
| **Talend Data Quality components** | In-Studio DQ components (`tDQRules`, `tSchemaComplianceCheck`, `tMatchGroup`, masking) |
| **Talend MDM** | Master data management — **retired**; see §3.3 |
| **Talend Change Data Capture** | The standalone CDC product — **retired**; see §3.3 |
| **Qlik Talend Cloud** | The cloud subscription that "combines the capabilities of Qlik Talend Data Integration and Talend Cloud" **(verified** — Qlik Talend Cloud subscription options, 2026-09-11) |

Connector and component breadth is marketed as "**more than 900 connectors and components**" in the Talend Cloud context **(verified** — TMC User Guide, 2026-09-09) and as "1,000+ pre-built connectors" in third-party profiles **(flagged** — vendor-adjacent secondary source; treat any single connector-count figure as a marketing number, not an inventory, and count your own).

---

## 3. The Editions and the Licensing

This is the section where folklore does the most damage, because three of the four things a buyer wants to know — *what the tiers gate*, *what happened to the free version*, *what it costs* — are answered by vendor sources that are scattered, and one of them (**price**) is answered by no vendor source at all. What follows is what the documentation actually says, with the gaps recorded as gaps.

### 3.1 The historic subscription product line

Before the Qlik-era consolidation, Talend sold a family of named subscription products. The one place that still enumerates part of the family authoritatively is the current Studio documentation, in the note that **Routes and services are available only in certain subscription products** **(verified** — Talend Studio User Guide, "What is Talend Studio?", last updated 2026-08-27):

| Historic subscription product (still named in current documentation) | What it gated |
| --- | --- |
| **Talend Data Fabric** | The full on-premises platform: integration + quality + governance + catalog + services |
| **Talend ESB** | The application-integration lane: Routes, services, mediation |
| **Talend Data Services Platform** | Data Services + API/service build, build-as-microservice |
| **Talend Real-Time Big Data Platform** | The Spark/Big-Data runtime lane (Spark batch and streaming jobs) |
| **Talend MDM Platform** | Master data management — **retired**, see §3.3 |
| **Talend Cloud API Services Platform** | Cloud-side API design/testing/service packaging |
| **Talend Cloud Data Fabric** | Cloud-side integration + quality + governance |

**Flagged:** the wider historical SKU list — *Talend Data Integration Platform*, *Talend Big Data Platform*, *Talend Cloud Data Integration*, *Talend Cloud Data Quality*, and the desktop/developer editions by which early customers were licensed — did **not** appear in a current first-party enumeration at verification, and this guide therefore asserts only what the list above supports **(flagged**, §11 item 3). If your contract names a SKU not in this table, the licence document you hold outranks any guide.

### 3.2 The Qlik-era editions: capacity, tiers, and what each tier gates

Under Qlik, the commercial shape changed from named products to a **capacity-metered, four-tier subscription**. The primary source is unambiguous and current:

- **Model:** "Qlik Talend Cloud subscriptions are based on a **capacity model** with three primary value meters: volume of **Data Moved**, **Job executions**, and **Job duration**." **(verified** — "Qlik Talend Cloud subscription options", Qlik Cloud Help, last updated 2026-09-11, read 2026-09-14.)
- **Tiers:** "available as a subscription option in the form of **four tiers: Starter, Standard, Premium, and Enterprise**. The higher editions provide more advanced capabilities including additional data sources and transformations, as well as governance, AI, and deployment capabilities. **All subscriptions include default analytics capabilities in Qlik Cloud with 25GB capacity.**" **(verified** — same page.)

What each tier gates, per the same source:

| Tier | Documented inclusions and exclusions |
| --- | --- |
| **Starter** | Replication from all supported SaaS applications and a **limited set of databases**, to any supported target; "a fully managed data gateway for simple replication projects is included. Qlik Talend Cloud Starter **does not support all sources or low-latency change data capture**, nor does it provide a way for clients to host and manage their own data gateway." |
| **Standard** | Real-time data movement from **all supported relational databases and SaaS applications including CDC where possible**; basic transformation (filtering/renaming, add/edit/delete columns). "This edition **does not include support for real-time SAP and mainframe sources, or advanced transformations.**" |
| **Premium** | Adds **ETL or ELT** transformation with "client managed, hybrid and multi-cloud deployments"; adds **application and API integration, including API design and testing**; adds "mostly technical data quality features, plus basic data governance capabilities", data products and marketplace. "This edition **does not include support for real-time SAP and mainframe sources.**" |
| **Enterprise** | Everything in Premium, **plus real-time data movement from SAP and mainframe sources**, plus **data stewardship**. Note: "Qlik Talend Cloud Enterprise **does not include** Qlik Cloud Analytics Premium capabilities." |
| **Premium and Enterprise only** | "data spaces, users, and roles are **automatically synchronized** between Qlik Cloud and Talend Cloud" |
| **Cross-cutting gate** | "Talend Studio **8.0.1 R2024-05 or higher** is required to use Talend Studio capabilities with a Qlik Talend Cloud **Premium Edition** or **Enterprise Edition** license" |

**(All rows verified** — "Qlik Talend Cloud subscription options", help.qlik.com, last updated 2026-09-11, read 2026-09-14.)

Two consequences worth stating plainly, because existing estates trip over them:

1. **Lineage into Qlik Cloud is a Premium/Enterprise capability.** "Data Integration Jobs published from Talend Studio to Talend Cloud can generate input and output datasets and lineage, which can be sent to Qlik Cloud", and the "Available in…" list for that task is exactly **Qlik Talend Cloud Enterprise Edition and Premium Edition** **(verified** — Studio User Guide, "Publishing datasets and lineage to Qlik Cloud", last updated 2026-08-27). An institution that downgrades to Standard to save money loses its lineage push, not merely some dashboard.
2. **Minimum scheduling intervals are a licensing feature.** Landing/replication cadence is tier-gated: without a Data Movement gateway, **Starter and Standard are 1 hour**, **Premium and Enterprise 30 minutes**; with a gateway, database sources are **continuous** for Standard and above, while SaaS sources are **15 minutes (Standard)** and **5 minutes (Premium/Enterprise)** **(verified** — "Qlik Talend Cloud subscription options", 2026-09-11). If a use case needs 5-minute SaaS freshness, that is an Enterprise-tier requirement, not an engineering choice.

### 3.3 The open-source edition's actual fate

This is the currency landmine. The widely repeated story — "Qlik killed the free version straight after buying Talend" — is wrong in its causal claim and roughly right in its outcome, and the primary source is a single vendor page that must be read rather than paraphrased:

> "Over the last 20 years, Talend has provided a free open-source version of Talend Studio… **As of January 31, 2024, the open-source version of Talend Studio was retired and is no longer hosted or updated by Qlik and Talend.** This decision stems from **diminishing community adoption and lack of contribution to the project** as organizations pivot to Qlik's commercial products. In addition, the commercial version of Talend Studio has now fulfilled its full potential and the capabilities of Qlik Talend Cloud, which includes Talend Cloud functionality, continues to grow."

**(verified** — qlik.com/us/products/talend-open-studio, read 2026-09-14.)

Read carefully, four things are true and one thing is not:

- **True:** the free, open-source **Talend Open Studio** distribution ended on **31 January 2024** and "is no longer hosted or updated by Qlik and Talend" — so there is no vendor-supported download, no patch stream, and no CVE stream for that distribution **(verified** — same page).
- **True:** the vendor's stated reasons are **community adoption and contribution**, not the acquisition. The retirement date is nine months after the Qlik close, and Qlik's own framing points at the commercial pivot rather than at consolidation.
- **True:** the **commercial** Talend Studio continues, is subscription-licensed, and is what "Talend Studio" now means in all current documentation **(verified** — Studio User Guide, 2026-08-27).
- **True:** Qlik states that "many customers [have] successfully transition[ed] to the powerful commercial version of Talend Studio as well as Qlik Talend Cloud" **(verified** — same page).
- **Not findable, and therefore flagged:** the precise legal nature of the retirement — whether the *source code* of the open-source distribution was a separate repository with a separate licence, what that licence was, and the retirement of that specific artefact — could not be confirmed at a first-party licence statement in this session **(flagged**, §11 item 4). Anyone whose estate rests on a claim like "we can always fall back to the open-source Studio" should treat the fallback as unverified until the licence text of the specific build they hold is produced.

**What remains available today, verified:** a **free trial** of the commercial products ("Try for Free" against Qlik Talend Cloud), the **Talend Cloud API Tester free edition** (a Chrome extension route is documented alongside the cloud URL) **(verified** — "Accessing Talend Cloud applications", Talend Installation and Upgrade Guide, last updated 2026-09-09), and the vendor's offer that "Qlik is ready to answer your questions and help you learn more about how you can expand your use cases" **(verified** — qlik.com Talend Open Studio page). There is **no** free, maintained, full-platform download.

### 3.4 Developer licensing and the desktop model

The desktop model is subscription-based and tier-coupled: "Talend provides you with a range of **subscription Talend Studio licenses**" **(verified** — Studio User Guide, 2026-08-27), and the perspectives a developer sees depend on the licence or on the type of remote project **(verified** — same page). The cloud-side control plane administers "roles, users, projects, engines, and **licenses**" **(verified** — Remote Engine Gen2 architecture, 2026-09-09). **Flagged:** the *counting rule* for a Studio developer licence — named user versus concurrent developer versus entitlement-by-tier, and whether CI build agents consume a licence — was not published in the documentation reviewed **(flagged**, §11 item 5). This is a first-order procurement question for any estate with a CI pipeline that builds jobs, and it must be answered in writing by Qlik Sales before a renewal is signed.

### 3.5 Runtime and licence accounting, as documented

The vendor documents the **meters** (Data Moved, Job executions, Job duration) and states that a service account owner can see "the current plan and subscription type" in **My Qlik** **(verified** — "Qlik Talend Cloud subscription options", 2026-09-11). It does **not** publish, in the documentation reviewed, how each meter is computed in edge cases — retries, aborted tasks, idle engine time, or whether a failed execution consumes execution credit **(flagged**, §11 item 6). In a capacity-metered model the *definition* of the meter is the price. Do not accept a quotation that does not define all three meters in the contract text.

### 3.6 The support lifecycle for older generational releases

This table is the most important set of dates in the guide, and it is fully sourced to Qlik's own customer support statements **(verified** — "Talend products lifecycle", "Talend Data Catalog lifecycle", "Talend Remote Engine lifecycle" and "Talend MDM End-of-Life", all at help.qlik.com, last updated 2026-09-09 / 2026-09-10, read 2026-09-14):

| Product / version | Released | Retired | End of Life |
| --- | --- | --- | --- |
| **Talend 8** | November 2021 | N/A | N/A — receives new features, bug fixes and CVE fixes through **monthly service releases** |
| **Talend 7.3** | February 2020 | May 2023 | **30 November 2024**; extended support purchasable **through December 2026** |
| **Talend 7.2** | June 2019 | January 2021 | June 2022 |
| **Talend Change Data Capture** | — | **December 2023** | **December 2025** |
| **Talend MDM Server** (in MDM Platform and on-premises Data Fabric) | retirement announced **2021-07-16** | — | **31 December 2024** |
| **Talend Data Catalog 8.1** | April 2024 | N/A | N/A (current; monthly releases for the latest version only) |
| **Talend Data Catalog 8.0** | April 2022 | April 2024 | 31 December 2024 |
| **Talend Data Catalog 7.3** | January 2020 | April 2022 | December 2022 |
| **Talend Remote Engine Gen1 2.14** | January 2026 | N/A | N/A |
| **Talend Remote Engine Gen1 2.13** | October 2023 | January 2026 | **30 June 2027** |
| **Talend Remote Engine Gen2 R2026-03** | March 2026 | N/A | N/A |

Qlik also publishes the vocabulary that governs all of the above, and it is worth quoting because it defines what a customer actually loses: "**Retired Software** is Software or Cloud Service Client that is not available for general sale. Qlik does not provide product enhancements or Service Releases for Retired Software… **End of Life Software** (also referred to as End of Support Software) is Software or Cloud Service Client that is not available for sale or support by Qlik and does not receive Service Releases or Patches. Limited Extended Support may be available for purchase at Qlik's discretion." **(verified** — "Talend product lifecycle and compatibility", help.qlik.com, last updated 2026-09-09.) The default extended-support rule is that "extended support may be available for purchase to support an end of life product for an additional **12 months** after end of support" **(verified** — "Talend products lifecycle", 2026-09-09) — which is the correct lens for the 7.3 December 2026 cliff noted in §9.

### 3.7 Price: the absence, recorded explicitly

**No list price is published anywhere in the sources this guide reviewed.** The subscription-options page directs the reader to "**Qlik Pricing** or contact Qlik Sales" for "what is included, and available in your region" **(verified** — "Qlik Talend Cloud subscription options", 2026-09-11), and the Talend product pages offer a free trial and a "Contact Us" path with no figures **(verified** — qlik.com and talend.com product pages, read 2026-09-14). **This guide therefore quotes no price, no per-unit rate and no tier uplift, and any figure you have seen elsewhere is someone else's estimate** **(flagged** for the whole pricing dimension, §11 item 5). Note also that the third-party commentary that describes the pricing as "opaque", and reports customer complaints about cost predictability, is exactly what a capacity-metered model with unpublished meter definitions produces **(flagged** — secondary analyst/community commentary, not a primary source).

---

## 4. The Cloud Platform, Verified

The cloud platform is where Qlik is putting its money, and it is also where the naming is most tangled. The clean statement, from the vendor: "**Qlik Talend Cloud combines the capabilities of Qlik Talend Data Integration and Talend Cloud**" **(verified** — "Qlik Talend Cloud subscription options", 2026-09-11). Below that line sit the components, and §2.4 already listed them; this section is about where the work actually happens.

### 4.1 What the cloud control plane does

**Talend Management Console (TMC)** is the control plane. Its own description of its scope: "This guide provides information about managing and running tasks and plans, administrating Studio projects, users, and user roles, and managing execution engines in Talend Management Console." **(verified** — TMC User Guide, 2026-09-09.) Structurally, TMC presents:

- **Users & Security** (roles, users), **Studio projects**, **Environments** with space access, **Promotions** to move designs between environments;
- **Processing** — "Manage your remote engines and clusters, or Remote Engines for Pipelines and run profiles";
- **Configurations** — "configure static IP addresses or enable log export";
- **Operations and Management** — "configure, schedule, execute, and monitor tasks and plans".

**(All verified** — TMC User Guide, "What is Talend Management Console?", last updated 2026-09-09, read 2026-09-14.)

### 4.2 Where execution actually happens

This is the question that decides a regulated deployment, and the documentation answers it precisely:

- **Studio-built artefacts:** "depending on your license, you can create executable tasks from **Jobs, Data Services, and Routes** published from Talend Studio and run them **directly in the cloud or on Remote Engines**, ensuring the security of your data." **(verified** — TMC User Guide, 2026-09-09.)
- **Pipeline Designer artefacts:** "you can create executable tasks from pipelines published from Talend Cloud Pipeline Designer and run them **directly in the application or on a Remote Engine Gen2**." **(verified** — same page.)
- **Inside the customer's network:** "Your Virtual Private Cloud includes the Remote Engine Gen2 that is used to run pipelines and preparations in a secure way. The Remote Engine Gen2 ensures a secure access to your data stored on Kafka, databases, file systems, etc. and **executes your artifacts on a local Spark engine (default)**." **(verified** — Remote Engine Gen2 architecture, last updated 2026-09-09.)
- **Health semantics you must design for:** "the Remote Engine Gen2 sends a **heartbeat message every 60 seconds**. If the IPAAS service does not receive a heartbeat for **180 seconds**, the connection between the engine and Talend Cloud is considered broken. If the engine status is updated with the warning *Not available* in Talend Management Console, restart the virtual machine on AWS or Azure." **(verified** — TMC User Guide, "Remote Engine Gen2", last updated 2026-09-09.) Three minutes of network loss to the control plane makes an engine look dead — plan the alerting around that, not around your own expectations.
- **The trial-grade engine:** the **Cloud Engine for Design** runs artefacts in the cloud and is embedded by default, but the documentation recommends "the secure Remote Engine Gen2 for advanced processing of data" **(verified** — Remote Engine Gen2 architecture, 2026-09-09). Treat the design engine as a sandbox, not a production path.

### 4.3 Regions, tenancy and data residency

Region choice is a documentation-backed fact set, not a marketing claim. Talend Cloud's AWS regions are mapped one-to-one onto Qlik Cloud regions:

| Qlik Cloud region | Talend Cloud region | Location | Region code |
| --- | --- | --- | --- |
| Asia Pacific (Tokyo) | AWS Asia Pacific | Tokyo | ap-northeast-1 |
| Asia Pacific (Mumbai) | AWS India (Mumbai) | Mumbai | ap-south-1 |
| Asia Pacific (Singapore) | AWS Singapore | Singapore | ap-southeast-1 |
| Asia Pacific (Sydney) | AWS Australia | Sydney | ap-southeast-2 |
| Europe (Frankfurt) | AWS Europe | Frankfurt | eu-central-1 |
| Europe (Ireland) | AWS Europe (Ireland) | Ireland | eu-west-1 |
| Europe (London) | AWS Europe (London) | London | eu-west-2 |
| Europe (Paris) | AWS Europe (Paris) | Paris | eu-west-3 |
| Israel (Tel Aviv) | AWS Israel (Tel Aviv) | Tel Aviv | il-central-1 |
| Middle East (UAE) | AWS Middle East (UAE) | UAE | me-central-1 |
| South America (São Paulo) | AWS Brazil (São Paulo) | São Paulo | sa-east-1 |
| US East (North Virginia) | AWS United States - East | North Virginia | us-east-1 |

**(verified** — "Accessing Talend Cloud applications", Talend Installation and Upgrade Guide, last updated 2026-09-09, read 2026-09-14.)

Three residency constraints come with that table and all three are vendor-stated **(verified** — same page and "Qlik Talend Cloud subscription options", 2026-09-11):

1. **Not every capability is in every region.** "Some Talend Cloud regions, such as **Azure United States - West**, do not have corresponding Qlik Cloud deployments and are not included in this mapping"; and "Talend Cloud capabilities are **not yet available in all regions**. Regional availability can vary by capability."
2. **Some applications are excluded from specific regions.** For example the region listing carries an explicit restriction that a given set of capabilities is "**Not available in the following Qlik Talend Cloud regions: Canada (Montréal)**".
3. **A sovereign-type option exists.** Qlik documents **Qlik Cloud Government** and points to its data-integration capabilities **(verified** — "Qlik Talend Cloud subscription options", 2026-09-11). Whether that satisfies a specific institution's regulator is a question for the institution, not for this guide.

### 4.4 Integration with the wider data stack

The vendor's own connectivity story is unusually easy to verify because it appears in the product pages and the run-time documentation simultaneously: native integrations are marketed with **Snowflake, Apache Iceberg, Databricks, Microsoft Fabric, AWS, Azure, Google Cloud, Kafka/Confluent, Oracle, SAP, Salesforce and Marketo** **(verified** — talend.com and qlik.com product pages, read 2026-09-14), and the movement pattern is "Agentless CDC from databases, SAP, mainframe, and SaaS" with "native integrations with Snowflake, AWS, Apache Iceberg, Microsoft Fabric, Databricks and more" **(verified** — qlik.com Qlik Talend Cloud page, read 2026-09-14). On the *identity* side, Talend Cloud exposes **SSO configuration** (`/oidc/ssologin` per region) and, at Premium/Enterprise, **synchronises data spaces, users and roles between Qlik Cloud and Talend Cloud** **(verified** — region URL table, 2026-09-09; subscription options, 2026-09-11).

Two specific claims are worth isolating because they are the clearest evidence of the two platforms actually being joined:

- **Lineage into Qlik Cloud:** verified in §3.2 — a Studio job run as a Talend Management Console task can generate datasets and lineage that appear in **Qlik Cloud's Catalog**, showing "the transformations backwards to the original source", gated to Premium/Enterprise **(verified** — Studio User Guide, "Publishing datasets and lineage to Qlik Cloud", 2026-08-27).
- **A Qlik-specific output component:** the Talend Studio monthly release **R2026-08** lists, among new features, "**QTDI-2175 - tQlikOutput file extension is required**" **(verified** — Studio cumulative patch note R2026-08, released 2026-08-21). A `t…` component whose target is Qlik is the most concrete possible signal that Talend Studio is being wired into Qlik's estate rather than maintained as a museum piece.

**Flagged:** a press release dated **2025-06-10** is reported to announce "Qlik Expands Integration with the Databricks Data Intelligence Platform", including streaming CDC to Unity Catalog and Iceberg optimisation via Talend. The URL and date were seen in a secondary aggregator; the release body was **not** opened at source **(flagged**, §11 item 7).

---

## 5. The Qlik Era: Promised Versus Shipped

This is the section that an existing Talend estate actually needs, and it is deliberately split into two columns: what Qlik **said**, and what is **observable** in documentation, release notes and lifecycle tables. The gap between them is the risk.

### 5.1 What Qlik publicly committed to

| Commitment | Exact words | Date/source |
| --- | --- | --- |
| Complementary, not cannibalising | "Qlik and Talend provide comprehensive and complementary capabilities with industry-leading solutions across real-time data and application integration, data governance, data quality, transformation, analytics, artificial intelligence and machine learning" | **(verified** — "Qlik Intends to Acquire Talend", 2023-01-05) |
| Cloud-independent | "The proposed combination would deliver a unique, **cloud-independent** portfolio" | **(verified** — same release) |
| Support for existing investments | "We are honored to have earned our position as a leading solution provider in multiple market categories and **remain committed to supporting our customers' existing investments**" — Mike Capone, CEO | **(verified** — qlik.com blog "Qlik \| Talend: Looking Forward", read 2026-09-14; posted at close, May 2023) |
| Open and methodology-agnostic | "Qlik will continue to remain **open to virtually any data source, target, architecture, or methodology**" | **(verified** — qlik.com press release, 2023-05-16) |
| Increased investment | "expanded product offerings, enhanced support and services, and increased investments in innovation and R&D" — Mike Capone | **(verified** — same release) |
| No forced convergence deadline | No published statement requiring or dating a migration of Studio customers to Qlik Cloud was found | **(flagged** — absence of evidence; §11 item 8) |

Note what is **not** in that list: no product end-of-life announcements, no "Talend Studio will be retired in year X", no commitment to a unified codebase by a date. Qlik's public commitments in 2023 were about *support* and *investment*, both of which are unfalsifiable in the press-release sense.

### 5.2 What is actually observable, five years on the clock

Observable means: published release notes, dated documentation, and dated lifecycle tables. All of the following are **(verified)**, with the source noted:

| Observable signal | Evidence | Date |
| --- | --- | --- |
| **Monthly Studio release train is alive** | Cumulative patch **R2026-08** (`Patch_20260821_R2026-08_v1-8.0.1`) with new features, fixed issues and **22 CVE fixes** listed across DI, DQ, BD, APPINT, Studio and TDM components | 2026-08-21 |
| **New features are still being added to the classic Studio**, not only to the cloud product | R2026-08 new features include `tDSDelete` filter criteria, SAP RFC Server queue-name configuration, JSON Schema draft auto-detection in `tSchemaComplianceCheck`, Databricks 14.x/15.x Spark 3.5 + Spark 4 row-normalisation work, API-first security management for data services, and "Lineage append or overwrite" | 2026-08-21 |
| **Documentation is actively maintained** | Talend Studio User Guide last updated **2026-08-27**; subscription/lifecycle pages last updated **2026-09-09/10/11**; the Studio doc is versioned **8.0-R2026-08** | Aug–Sep 2026 |
| **Runtime modernisation is being pushed, not frozen** | Artefacts built with Java 17 and Camel 4 since R2025-02; Java 21 mandatory for Studio from R2026-06; Remote Engine must be 2.12.7+ for at least one fix class | since 2025-02 |
| **Qlik–Talend plumbing has shipped** | `tQlikOutput` component; lineage/dataset publishing into Qlik Cloud Catalog; spaces/users/roles synchronisation; a documented "Connecting Qlik Cloud with Talend Cloud" onboarding step | 2026-08 / 2026-09 |
| **Consolidation is real and dated** | Open-source Studio retired 2024-01-31; MDM Server EOL 2024-12-31; Talend Change Data Capture retired Dec 2023 and EOL Dec 2025; Talend 7.3 EOL 2024-11-30 with extended support ending **December 2026** | 2023–2026 |
| **Analyst framing moved from "Talend" to "Qlik"** | Product pages now say "Qlik Talend Cloud®" and quote Qlik's own positions: ten years a Leader in the Gartner MQ for Data Integration Tools; seventh time a Leader in the 2026 Gartner MQ for Augmented Data Quality Solutions | read 2026-09-14 |
| **The platform is being repositioned to "agentic data engineering"** | Qlik markets data-engineering agents (Data Quality Agent, Data Product Agent, Catalog & Glossary Agent, Declarative Pipelines, Helper Agent), a **Qlik MCP Server**, and building pipelines with coding agents such as Claude Code and GitHub Copilot, with "you never lose control of the code" and human-in-the-loop framing | read 2026-09-14 (talend.com and qlik.com) |

### 5.3 The honest verdict: promised versus shipped

**Shipped, and safe to plan on** (each backed by a dated release note or documentation page above): the monthly Studio train with CVE remediation; the Java 17/Camel 4/Java 21 runtime progression; lineage and dataset publishing into Qlik Cloud; Tenanted synchronisation of spaces/users/roles for Premium and Enterprise; the retirement schedule for MDM, CDC, the open-source edition and Talend 7.3; a four-tier cloud subscription with published tier gating.

**Announced or marketed, not verifiable as shipped** **(flagged)**: the agentic data-engineering capability set, the MCP server's production maturity, and the "agent-built pipelines" workflow. These appear as product-page positioning and tour links, not as dated release-note entries of the kind that accompanies the classic Studio train. Treat them as direction, not as a delivery commitment.

**Directional, and the thing an existing estate must decide about**: Qlik is unambiguously steering the future to **Qlik Talend Cloud** — a cloud, capacity-metered, Qlik-administered product — while continuing to service and patch the **client-managed** (Studio/TAC/Runtime/JobServer) estate. Both halves are true in September 2026, and no vendor source says when the second half stops being funded. That is not a criticism of Qlik; it is the standard shape of a post-acquisition platform, and it is precisely why §8 exists.

**The one-line summary an architect can carry:** *the train is running, the consolidated products are the ones already retired, and the strategic centre of gravity is the cloud subscription — plan upgrades inside the client-managed line, but do not plan new ten-year commitments inside it without a written lifecycle answer from Qlik.*

---

## 6. The Competitive Position

This section is a **positioning table only**, as required by the scope of this guide. It does not rebuild the selection framework, does not score capabilities, and does not re-derive the integration patterns. For *how to choose*, read [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) §3 (tool comparison), §11 (tool matrix) and §12 (architecture decision tree); that file owns the discipline, this one owns what each competitor *is* in September 2026 and whether the product named still exists as described.

Every competitor in the table below was checked for existence and current positioning on **2026-09-14**; the verification note names what was actually read.

### 6.1 Classical ETL

| Product | What it is today | Verified on 2026-09-14 |
| --- | --- | --- |
| **Informatica** | Informatica Intelligent Data Management Cloud (IDMC); its data-integration product is marketed as **"Data Integration and Engineering"**, claiming ETL/ELT/replication/CDC, Spark and serverless options, CLAIRE AI, and "up to 65% lower TCO" / "up to 80% less time" marketing claims. **Material ownership change: Informatica is now part of Salesforce** | **(verified** — informatica.com/products/data-integration.html, which carries the "**Informatica from Salesforce**" lockup and a "© 2026 **Salesforce, Inc.**" footer, and lists the **2025** Gartner MQ for Data Integration Tools as its analyst anchor.) The completion date of the Salesforce transaction (~November 2025) circulates in secondary sources but was **not** confirmed at a first-party press release here **(flagged**, §11 item 9) |
| **IBM DataStage** | Still shipped; "supports extract, transform, load (ETL) and extract, load, transform (ELT) patterns", with a **remote engine** split (managed control plane, data-plane execution next to the data), an **ETL/ELT/TETL toggle**, a Python SDK, an AI pipeline assistant, and — importantly for naming — "The powerful capabilities of DataStage are **now available within watsonx.data integration**". Also offered **as a Service on AWS** | **(verified** — ibm.com/products/datastage, read 2026-09-14, including the 2025 Gartner MQ leader claim and the watsonx.data integration statement) |
| **SSIS (SQL Server Integration Services)** | Still the Microsoft on-premises/ADF-adjacent integration platform: "a platform for building enterprise-level data integration and data transformations solutions", with tasks, transformations, graphical tools, and an SSIS Catalog database to "store, run, and manage packages". Applies to SQL Server (v17 docs) **and** the SSIS Integration Runtime in Azure Data Factory | **(verified** — learn.microsoft.com/sql/integration-services/sql-server-integration-services, page metadata "Last updated on 06/23/2026", read 2026-09-14) |

### 6.2 Modern ELT / transformation

| Product | What it is today | Verified on 2026-09-14 |
| --- | --- | --- |
| **dbt** | Still "build modular, maintainable data products"; SQL-defined transformations executing in the warehouse; testing, versioning, documentation, Git-based review. **Material corporate change: "Fivetran and dbt Labs have merged to deliver the data infrastructure for agents you trust"** — announced on dbt's own site banner | **(verified** — getdbt.com/product/what-is-dbt and the site banner, read 2026-09-14. An "all-stock merger announced October 2025 at ~$600M ARR" figure appears in secondary sources and is **(flagged)**; the merger statement itself is first-party) |
| **Matillion** | The company now leads with **"Maia by Matillion"** — "the first AI Data Automation platform" — while keeping **Matillion Data Productivity Cloud + ETL** as the product its existing customers use | **(verified** — matillion.com, read 2026-09-14: "Introducing Maia by Matillion: Evolving beyond traditional ETL into AI Data Automation"; "Existing Customers — Matillion Data Productivity Cloud + ETL") |
| **Fivetran** | "Automated data movement platform"; "Continuously deliver trusted data from SaaS apps, databases, ERPs, and file connectors into your warehouse or lake"; "900+ sources and destinations"; hybrid deployment, governance and security pages; published scale numbers (e.g. "2T+ rows synced per month") | **(verified** — fivetran.com, read 2026-09-14) |

### 6.3 Open source

| Product | What it is today | Verified on 2026-09-14 |
| --- | --- | --- |
| **Airbyte** | Still open-source data integration, but repositioned: the site now leads with "**Airbyte \| The Context Layer for AI Agents \| Open-Source Data Integration**", a managed connector library, a **Context Store**, and an **MCP gateway** serving agents. Connectors, CLI/SDK/API/MCP interfaces and "it's your context… never locked in" framing are all still present | **(verified** — airbyte.com, read 2026-09-14) |
| **Apache Hop** | A genuine **Apache Software Foundation Top-Level Project** — "available under the Apache v2 license" — metadata-driven, visual, with "design once, run anywhere" across the native Hop runtime plus Apache Spark, Apache Flink and Google Dataflow via Apache Beam. Actively released: **Apache Hop 2.19 on 2026-08-16**, preceded by 2.18 (2026-06-01) and 2.18.1 (2026-06-18) | **(verified** — hop.apache.org homepage and release list, read 2026-09-14; graduation as a TLP in the final days of 2021, with the ASF press release reprinted and dated **2022-01-18**, verified at hop.apache.org/categories/Graduation/, read 2026-09-14) |

### 6.4 The one-paragraph verdict, without rebuilding the framework

Talend's defensible ground in September 2026 is **breadth plus quality plus lineage-in-one-platform**, sold to organisations that already run it: the connector surface (900+/1,000+ claimed), the in-Studio data-quality components, the catalog, and now the lineage push into Qlik Cloud. Its exposed ground is everything that has changed around it: **Informatica now sits inside Salesforce**, which changes both the commercial leverage and the roadmap risk of the classical alternative; **dbt and Fivetran are one vendor**, which collapses the "best-of-breed ELT + best-of-breed ingestion" argument into a single procurement; **Matillion and Airbyte have repointed themselves at the AI-agent data context layer** rather than at classical ETL; and **Apache Hop remains the credible open-source escape hatch**, now four minor versions into a steady release cadence. Any estate decision in §8 that does not price those four developments is pricing the 2023 market with 2026 money.

---

## 7. The Regulated-Enterprise and Banking Angle

A Talend estate in a bank is not evaluated on connectors. It is evaluated on five things a regulator, an internal auditor or a model-risk function will actually ask about: **auditability, lineage evidence, separation of duties, data residency, and exit risk**. This section states what the platform does and does not produce, using the documentation verified above, and cross-references the repo's discipline guides rather than re-deriving them.

### 7.1 What the platform produces natively

| Control question | What Talend produces | Source |
| --- | --- | --- |
| *Can we prove who changed a job and when?* | **Versioned design objects own the record**, not the generated code: "Managing Job and Route versions", "Managing items on different branches and tags", "Comparing Jobs", "Documenting a Job or a Route" are all documented Studio capabilities | **(verified** — Studio User Guide index, last updated 2026-08-27) |
| *Can we prove a job was tested before release?* | **Test cases are first-class**: "Testing Jobs and Services using test cases" and "Testing Routes using test cases" | **(verified** — same index) |
| *Can we prove a deployment was promoted, not hand-copied?* | **Environment and space model with promotions**: TMC administers "Environments" and a "Promotions" page "to promote your designs from one environment to another"; Studio projects can be pushed through CI with Maven and Talend CI Builder | **(verified** — TMC User Guide, 2026-09-09; Studio patch note R2026-08, 2026-08-21) |
| *Can we show who is allowed to run what?* | **Users, roles and spaces** are administered in TMC under Users & Security, with engine-to-space allocation; Premium/Enterprise synchronise spaces, users and roles with Qlik Cloud | **(verified** — TMC User Guide, 2026-09-09; subscription options, 2026-09-11) |
| *Can we show where data came from and went?* | **Yes, at task level**, gated to Premium/Enterprise: jobs generate datasets and lineage to **Qlik Cloud's Catalog**, showing "the transformations backwards to the original source"; lineage is "**generated at run time when the task is executed**"; there is a documented "Disable Lineage" setting and a local lineage test before publishing | **(verified** — Studio User Guide, "Publishing datasets and lineage to Qlik Cloud", 2026-08-27) |
| *Can we prove nothing was tampered with?* | Job history, task history and log export exist (`enable log export` is a documented configuration), and **static IP addresses** can be configured for engines | **(verified** — TMC User Guide, 2026-09-09) |

### 7.2 What the platform does **not** produce natively

This is the part that decides whether an audit will be a two-day exercise or a two-week one. None of these is a defect in Talend specifically; all of them are the standard gap between *tool telemetry* and *regulatory evidence*.

1. **A single, immutable, regulator-shaped audit pack.** Talend produces **design history** (repository/branches/versions/tags), **test artefacts** (test cases), **promotion records** (environments/spaces) and **execution telemetry** (task history, logs) — in *different places*, at *different levels of granularity*, with **no single artefact that joins change-approval → release → executed run → data moved** into one signed object. That join is the institution's to build; the repo's [audit_as_code_guide.md](audit_as_code_guide.md) and [data_governance_guide.md](data_governance_guide.md) are the places to design it.
2. **Field-level lineage for the whole job portfolio.** Lineage is component-scoped and explicitly bounded: "the components you used in the Job **support lineage in Qlik Cloud**" is a precondition, and the documentation warns that "as Talend Studio supports a **different list of connectors** from Qlik Cloud, **the support scope of Qlik Cloud will be limited on some operations like Profiling and Preview**" **(verified** — Studio User Guide, 2026-08-27). An estate that assumes "we have column-level lineage everywhere" without auditing which components are in the supported list will discover the gap during the audit, not before it.
3. **Approval authority.** The tool records *what ran*. It does not record *who authorised the change*; that lives in the change-management system. **[control_m_guide.md](control_m_guide.md)** owns the scheduler-side half of this problem, and the separation between "the job ran" and "the change was approved" is the join a bank must evidence explicitly.
4. **Separation of duties as an enforced property.** TMC has roles and spaces, and Studio has licence-gated perspectives — but a *developer with a studio licence and production engine access* can, absent configuration, be the same person who authors, promotes and runs. SoD here is **configuration and process**, not an inherent property of the platform. Asserting "the tool enforces SoD" is a finding waiting to happen.
5. **Data-residency guarantees beyond the region you selected.** Region mapping is real and documented (§4.3), but capability availability varies by region, some applications are excluded from specific regions, and the mapping itself notes Talend regions without a Qlik Cloud counterpart. A residency statement must therefore be written **per tenant, per capability, per region** — not once for "Talend".
6. **Immutability/tamper-evidence of logs by default.** Log export and static IP configuration exist; whether the exported logs are tamper-evident and how long they survive is a platform-operations question the vendor documentation does not settle here **(flagged** for the retention/tamper-evidence dimension).

### 7.3 Deployment constraints a regulated estate must resolve before choosing cloud

| Constraint | What must be resolved | Anchor |
| --- | --- | --- |
| **Execution locality** | Confirm whether tasks run in the Qlik/Talend cloud or on a customer-hosted Remote Engine, and whether the licence tier permits customer-hosted engines at all (Starter explicitly does not) | §3.2, §4.2 |
| **Control-plane connectivity** | The engine's 60-second heartbeat / 180-second timeout means the control plane must be reachable or engines are declared unavailable | §4.2 |
| **Residency** | Region-by-region, capability-by-capability mapping; note excluded regions | §4.3 |
| **Sovereign options** | Confirm whether a government/sovereign tenancy satisfies the institution's regulator, and whether the data-integration capabilities are present there | §4.3 |
| **Third-party runtime dependencies** | Java 17/21, Camel 4, Maven 3.6.3+, container images, and the Studio↔Runtime alignment warning — all of which are Ops-owned, not just Dev-owned | §2.2 |
| **Endpoint and firewall** | Talend Cloud apps are per-region URLs; Studio requires internet access for at least the ESB microservice build path ("Talend Studio does not provide a pure offline mode for the ESB Microservice build operation") | **(verified** — Studio User Guide, "Building a Data Service Job as a Microservice", 2026-08-27) |

Deployment-permission questions that need an operational-resilience lens — what happens to the pipeline estate during a control-plane outage, how the institution classifies the platform, what the recovery objectives are — belong to [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) and are not re-derived here.

### 7.4 Migration and exit risk: where the lock-in actually sits

The instinct is to say "Talend jobs generate Java, so exit is cheap". That is half true and the half that is false is the expensive half:

- **Execution is portable.** A built job is a `.jar` or `.zip` runnable "independently of Talend Studio in standalone mode" **(verified** — Studio User Guide, 2026-08-27). For a retirement scenario where the objective is *keep it running, stop paying*, this matters and should be costed properly.
- **Maintenance is not portable.** The design of record is a Studio repository object. Change a mapping, add a column, fix a data-quality rule — all of it requires **Studio, a subscription licence and a licensed developer**. The generated Java is a build output, not a maintainable source of truth.
- **The exit path from the platform is therefore a rewrite, not a lift.** The realistic migration surface is *re-express in the target tool* (dbt/Fivetran, DataStage on watsonx.data, Informatica-IDMC-on-Salesforce, Apache Hop, Airbyte or in-house frameworks), plus a decision about the estate that is not worth re-expressing. The discipline guide's §11 matrix and §12 decision tree cover how to make that target choice; this guide's contribution is the *cost shape*: inventory first, then per-job complexity, then a stay-versus-move economic case, then a sequencing plan in which the first wave is deliberately low-criticality.
- **Two lock-in vectors have already been exercised by the vendor, and both are instructive.** The **MDM Server** line was announced for retirement on 2021-07-16 and reached EOL 2024-12-31; **Talend Change Data Capture** was retired in December 2023 and reached EOL in December 2025; the **open-source Studio** ended 2024-01-31. In every case customers got years of notice and a documented date — and in every case the only real options were commercial migration or re-platforming. **[data/data_lineage_tools.md](data/data_lineage_tools.md)** and **[data/data_profiling_guide.md](data/data_profiling_guide.md)** are the relevant companions for lineaging and profiling capability changes on the way out.

**The architect's summary of §7:** Talend produces the raw material of audit — versions, tests, promotions, roles, run-time dataset and lineage graphs — but not the *joined, immutable, authority-linked* artefact a regulator asks for. The gap is designed, built and owned by the institution, and it is the single largest hidden cost in any Talend estate review.

---

## 8. The Estate Decision

> **Label, per the scope of this guide: everything in this section is the author's own analysis, not vendor guidance.** Nothing here is a Qlik statement, a Qlik commitment or a Qlik recommendation. It is a decision framework an architect can take into a platform review and defend.

### 8.1 The assessment: six exposures to quantify before any decision

| # | Exposure | The question to answer with evidence | Why it is first-order |
| --- | --- | --- | --- |
| 1 | **Inventory** | How many jobs, routes, data services, joblets, routines, custom Java components, contexts and shared connections exist — by version, by project, by environment? | You cannot cost a migration you have not counted; custom components and routines are where the estimates die |
| 2 | **Version / EOL exposure** | What is on Talend 8, what is on 7.3 (EOL **2024-11-30**, extended support ending **December 2026**), what is on 7.2 (EOL June 2022), what data-catalog versions, what Remote Engine generation? | 7.3 customers are inside a **cliff that closes at the end of 2026**; after that the only vendor path is upgrade or unsupported |
| 3 | **Licence exposure under the new owner** | Which tier entitles you to what, under the capacity meters (Data Moved, Job executions, Job duration), and where is your current consumption relative to the metered boundary? | Qlik-era tiers are capability gates, not just price tiers — a "saving" can silently remove lineage, API integration or CDC |
| 4 | **Retired-product exposure** | Is MDM still in the estate? Is standalone CDC? Is anything still dependent on the now-retired open-source distribution? | All three are already past EOL; these are forced-exit items, not choices |
| 5 | **Skills-market risk** | How many people know Studio, the repository model, the Runtime/JobServer topology — and can you replace them at market rates? | Specialist skills around a platform whose centre of gravity has moved to Qlik Cloud get scarcer and dearer; this is a supply-chain risk, not an HR footnote |
| 6 | **Migration cost versus stay cost** | Full economic case: licence delta + infrastructure + upgrade programme + interim-run costs, against re-platform cost (inventory × complexity × target tool) + parallel-run cost + decommissioning | The honest comparison is *total cost over the next 3–5 years*, never the next renewal price |

### 8.2 The four migration paths

| Path | What it means | Best when | Principal risk |
| --- | --- | --- | --- |
| **In-place upgrade** | 7.2/7.3 → Talend 8 (Studio + Runtime + TAC/JobServer kept aligned to the same version); then stay on the monthly service train | The estate is on 7.3 and the business case is "stay", or the 7.3 extended-support end (Dec 2026) is close | **Runtime alignment**: "using unaligned versions is at risk"; Java 17/Camel 4 artefact changes; Java 21 on the developer desktop; a large regression surface across jobs |
| **Cloud move (re-host)** | Publish Studio artefacts to Talend Cloud; run on customer-hosted **Remote Engines** (Gen1) or re-author in **Pipeline Designer** (Gen2); adopt TMC for users/spaces/promotions; enable lineage into Qlik Cloud | You want the strategic product, the operating model is acceptable, and residency is satisfied per §4.3 | Licence-tier gates (Premium/Enterprise for lineage and API work), Engine lifecycle currency, control-plane dependency, and capacity-metering behaviour under load |
| **Replatform** | Re-express pipelines in a modern target — dbt/Fivetran, Matillion/Maia, Airbyte, Apache Hop, or the classical alternatives (DataStage on watsonx.data, Informatica-in-Salesforce) | The estate is small, the business logic is simple and documented, or the direction of travel is already a lakehouse-first architecture | Rewrite cost and *silent semantic drift*; data-quality and lineage logic is the part most often lost in translation |
| **Coexistence** | Keep Talend for the regulated batch core; build everything new in the strategic target (or in Qlik Cloud); wire the two through the shared metadata/lineage layer | Almost always the correct *first* answer: it converts a forced big-bang into a sequenced programme and preserves the option value | Requires deliberate governance — a target-state architecture and a hard "no new jobs here" rule for the legacy side, or coexistence silently becomes permanent duplication |

The scheduler question interacts with all four and is deliberately not covered here: **[control_m_guide.md](control_m_guide.md)** owns it, and any migration plan that changes the job runtime without changing the trigger model has not been thought through.

### 8.3 Decision criteria, in the order they should bind

1. **EOL proximity dominates.** Anything on 7.3 with a December 2026 extended-support end has a **stated deadline**; decisions on everything else can wait, decisions on this cannot.
2. **Retired products force exits regardless of strategy.** MDM and CDC are past EOL. There is no "stay" option for them, only "how".
3. **Lineage and quality capability decide the tier, not the discount.** Because lineage-to-Qlik-Cloud and API integration are Premium/Enterprise features, a tier reduction is a *capability* reduction. Audit §3.2 before accepting any saving.
4. **Custom code is the migration cost.** A portfolio of standard `tMap`/`tDB*` jobs migrates; a portfolio full of `tJava`, `tJavaFlex`, custom routines and unusual components does not. Count these first; they are the estimate.
5. **Skills scarcity is a risk premium on "stay".** A shrinking pool of Studio specialists is a cost that rises quietly each year and shows up as an outage or a rate card.
6. **Write the exit in the contract, not in the strategy deck.** Whatever the decision, the renewal should preserve: the right to run built artefacts, a stated data-export path for the repository and lineage, and a documented schedule for the client-managed lifecycle.
7. **Prefer reversibility.** Given that the platform has changed owners twice in five years, choose the option that keeps the most options open at the lowest cost per year — usually upgrade-plus-coexist, rarely big-bang replatform.

---

## 9. The Cymbal Bank Worked Example

> **All figures in this section are ILLUSTRATIVE AND FICTIONAL.** No number below is a quotation, a benchmark, a market rate or a vendor price. They exist to show the *shape of the calculation* — the units, the categories and the sequencing — using a single bank persona, **Cymbal Bank**, which is the only institution named in this guide. Substitute the institution's real inventory and real rates and the arithmetic is the same; adopt the numbers as if they were evidence and the analysis is worthless.

### 9.1 The estate as found

**Cymbal Bank's** integration platform team supports a Talend estate built between 2016 and 2023. The review opens with an inventory sprint:

| Dimension | Illustrative finding |
| --- | --- |
| **Jobs / artefacts** | 3,180 DI jobs, 610 of them containing `tJava`/`tJavaFlex` or custom routines; 240 Routes/Data Services; 96 joblets; 41 shared routines; ~1,900 context variables across 14 contexts |
| **Functional mix** | ~60% batch integration (feeds to the risk data mart, finance GL extracts, reference-data distribution); ~20% data quality (rule-based validation and masking on the customer and payments domains); ~12% API/ESB Routes and Data Services for internal service composition; ~8% remaining MDM-dependent flows |
| **Versions** | 62% on Talend 8.0.1 with the monthly patch train current to within two releases (**verified** as the live train: R2026-08 released 2026-08-21); 29% on Talend 7.3 (**verified** EOL 2024-11-30, extended support purchasable through December 2026); 9% on 7.2 (**verified** EOL June 2022 — i.e. **already unsupported**) |
| **Retired products in the estate** | MDM Server still running two flows (**verified** EOL 2024-12-31 — a forced exit, not a choice); standalone Change Data Capture in two data-movement paths (**verified** retired December 2023, EOL December 2025); no reliance on the retired open-source Studio distribution was found |
| **Engines** | 6 Remote Engine Gen1 instances on the deployed `2.x` line (**verified** Gen1 2.14 GA January 2026; 2.13 retired January 2026 with **EOL 2027-06-30** — see §9.5 and §11 item 10 for the version-drift check that must be run on the live estate); 2 client-managed Runtime hosts; 1 TAC instance |
| **People** | 18 engineers able to author in Studio, of whom 4 are the actual repository-experienced group; 2 contractors on the 7.3 estate |

### 9.2 Exposure classification

| Exposure | Finding | Severity |
| --- | --- | --- |
| **7.2 estate (9%)** | Already past EOL; no patches, no CVE stream | **Critical** — an unsupported integration platform inside a regulated estate is an audit finding with a date on it |
| **7.3 estate (29%)** | EOL 2024-11-30, riding on purchased extended support that **ends December 2026** | **Critical and time-boxed** — this is the single item that sets the programme's deadline |
| **MDM flows (8%)** | Vendor product EOL 2024-12-31 | **Critical** — no upgrade path exists; these flows must be re-expressed or retired |
| **CDC paths** | Product retired Dec 2023, EOL Dec 2025 | **High** — confirm whether the flows are running on unsupported software or have already been replaced |
| **Custom-code concentration (610 jobs)** | ~19% of the portfolio carries custom Java/routines | **High** — dominates any replatform estimate |
| **Licence tier** | Currently on a mid-tier entitlement; the lineage-into-Qlik-Cloud capability that audit has begun asking about sits above it | **High** — a capability gap disguised as a commercial question |
| **Skills** | 4 deep repository practitioners for a 3,180-job estate | **Medium-High** — bus-factor of 4, with a market that is thinning |

### 9.3 Criticality tiering (the sequencing spine)

| Tier | Definition | Illustrative count | Treatment |
| --- | --- | --- | --- |
| **T1** | Regulated reporting, risk/finance data, anything with a regulatory deadline or a penalty attached | ~210 jobs | Upgrade first, test hardest, never part of a first-wave rewrite |
| **T2** | Payment/customer data quality, masking, reconciliation | ~640 jobs | Upgrade in wave 2 behind T1; quality rules get dedicated regression suites |
| **T3** | Internal/operational integration, non-regulatory feeds | ~1,780 jobs | Candidates for consolidation or retirement during the upgrade |
| **T4** | Low-use, duplicated, orphaned (no owner, no downstream consumer) | ~550 jobs | **Decommission** — the cheapest capacity the programme will ever find |
| **Forced-exit** | MDM-dependent and CDC-dependent flows | ~250 jobs | Separate workstream; re-express into the strategic target, no upgrade attempted |

*(Counts are illustrative and deliberately sum to roughly the §9.1 portfolio; a real inventory will not partition this cleanly, which is itself the first finding.)*

### 9.4 Options, with illustrative costs

**All costs are fictional, rounded, and expressed in the same unnamed currency unit. They are not prices and must not be reused.**

| Option | Scope | Illustrative 3-year cost | Illustrative effort | Notes |
| --- | --- | --- | --- | --- |
| **A. Stay on 7.3, extend support again** | Do nothing structural; renew extended support | 1.0× licence, plus an uplift the vendor is under no obligation to offer | Minimal | **Not viable beyond Dec 2026** on the published default; buys months, not years |
| **B. Upgrade in place to Talend 8** | 7.2+7.3 estate → 8.0.1; Studio/Runtime/JobServer aligned; decommission T4 in flight | 1.4× current licence run-rate over 3 years, plus ~5,500 engineering-days | 9–14 months, 3 waves | Preserves the estate, fixes EOL exposure, does **not** fix the capability gap or the skills concentration |
| **C. Upgrade plus cloud move** | B, plus remote-engine execution for a subset, plus TMC adoption and lineage publication | 1.9× current licence run-rate (capacity meters on a cloud tier) plus ~7,200 engineering-days | 14–20 months | Delivers lineage-into-catalog, and commits the institution to a metered model; requires the meter definitions in writing (§3.5) |
| **D. Replatform the non-core** | Keep T1 on Talend (upgraded); re-express T3 + forced-exit in the strategic target | 1.6× current licence over 3 years (shrinking Talend footprint) plus ~9,800 engineering-days | 18–30 months, parallel-run cost | The most expensive to execute and the cheapest to own from year 4; requires a target architecture decision this guide does not make |
| **E. Big-bang replatform** | Move everything | 0.7× current licence from year 2 plus ~26,000 engineering-days | 30–48 months | Rejected on risk: the T1 regulatory surface would be rewritten under deadline pressure |

**Illustrative recommendation: C, with D as the committed direction of travel.** Upgrade the whole estate to Talend 8 to close the EOL exposure, decommission T4 first because it pays for part of the programme, move a scoped subset to the cloud product to obtain the lineage capability audit has asked for, and take the *target architecture decision* for D in the same 12-month window rather than after the upgrade lands. Option B alone leaves the institution paying to arrive exactly where it started, plus eighteen months; option E bets the regulatory reporting estate on a rewrite.

### 9.5 Risk and governance gates

| Gate | Trigger | Evidence required to pass |
| --- | --- | --- |
| **G1 — Inventory completeness** | Before any cost estimate is accepted | Job count reconciled against scheduler triggers, repository inventory **and** engine task history; unowned jobs identified |
| **G2 — Live version audit** | Before wave planning | Actual deployed Studio, Runtime, JobServer, TAC and Remote Engine versions read from the estate, not from the CMDB (the §9.1 engine line is illustrative and **must** be replaced with a read of the live estate — the Gen1/Gen2 and 2.13/2.14 distinctions in §2.3 are exactly where a CMDB lies) |
| **G3 — Licence and capability confirmation** | Before the commercial decision | Written confirmation from Qlik of: tier entitlements, all three meter definitions, developer-licence counting rule, extended-support availability and price, and the client-managed lifecycle position |
| **G4 — Lineage scope confirmation** | Before audit relies on it | The supported-component list for lineage in Qlik Cloud mapped against the actual job portfolio, with the uncovered jobs named |
| **G5 — SoD and change-control design** | Before cloud cut-over | Documented separation of author, promoter and operator roles; the change-approval → deployment → execution join evidenced end-to-end once, by hand, as a proof |
| **G6 — Residency sign-off** | Before any tenant is provisioned | Region, capability and exclusion confirmation against §4.3, signed by the institution's data-protection function |
| **G7 — T1 regression evidence** | Before each wave completes | Test-case evidence per T1 job, plus reconciliation of outputs against pre-upgrade baselines |
| **G8 — Exit-option preservation** | Before renewal signature | Artefact-runnability, repository/lineage export path and lifecycle commitments recorded in the contract |

### 9.6 What Cymbal Bank cannot know yet

An honest statement, because a review that pretends to certainty is the review that gets overruled:

1. **The actual price and the actual meters.** No list price is published (§3.7) and the meter definitions are not documented in the material reviewed (§3.5). Every cost in §9.4 is a placeholder until Qlik answers G3 in writing.
2. **The developer-licence counting rule**, including whether CI build agents consume entitlements (§3.4). This can move the licence line more than a tier change can.
3. **What Qlik will do with the client-managed line after the current support window.** The monthly train is alive and observable; the commitment horizon is not published (§5.1, §5.3).
4. **Whether the agentic/MCP capability set becomes a supported production capability** or remains positioning (§5.3).
5. **The true condition of the 7.2 and 7.3 jobs** until each is regression-tested — the version number tells you the risk class, not the work.
6. **Whether the 610 custom-code jobs are re-expressible at the assumed effort**, until a sample of thirty is actually attempted.
7. **The completeness of the lineage support list against the estate's connector surface**, until G4 is executed job by job.
8. **The institution's own target architecture decision** — which is a business-strategy question that the platform review can inform but cannot settle.

---

## 10. The Claims Audit

Every ownership, version, licence and lifecycle claim this guide makes, with its classification, its source and the date the source carries. **The dates are the point**: in this subject, a fact without a date is a rumour with good grammar. Verification date for all rows: **2026-09-14**.

### 10.1 Verified claims

| # | Claim | Source | Source date |
| --- | --- | --- | --- |
| V1 | Talend was founded in **2005** in France by **Bertrand Diard** and **Fabrice Bonan** | Computerworld interview with Diard ("In late 2005, when Talend was founded…"); Yahoo Finance 2016-07-29 IPO report ("founded in 2005 by Fabrice Bonan and Bertrand Diard") | 2009-02-24 / 2016-07-29 (read 2026-09-14) |
| V2 | IPO priced at **$18.00/ADS** on **2016-07-28**, 5,250,000 ADSs, Nasdaq **TLND** | Press release, "Talend Announces Pricing of Initial Public Offering" | 2016-07-28 |
| V3 | IPO closed **2016-08-03** with full over-allotment, **6,037,500 ADSs**, raising ~**$94.5M**; first trade **$27.66** | Closing press release; Fortune IPO report; TechCrunch first-day report | 2016-08-03 / 2016-07-28 / 2016-07-29 |
| V4 | Thoma Bravo take-private **announced 2021-03-10**, tender offer completed **2021-07-29**, subsequent period ended **2021-08-09**, **completed 2021-09-02** | Thoma Bravo press release, "Thoma Bravo Completes Acquisition of Talend" | 2021-09-02 |
| V5 | Take-private terms: all-cash, **~$2.4bn** valuation, **$66.00** per share, ~**29%** premium to 2021-03-09 close; CEO **Christal Bemont** | Same release | 2021-09-02 |
| V6 | Talend customer count **6,500+** at take-private | Same release | 2021-09-02 |
| V7 | Qlik's intention to acquire announced **2023-01-05**; **terms not disclosed**; expected close H1 2023 | "Qlik Intends to Acquire Talend…" (Thoma Bravo / GlobeNewswire) | 2023-01-05 |
| V8 | Talend customer count **7,250+** at Qlik announcement; Qlik **38,000+** | Same release | 2023-01-05 |
| V9 | Qlik acquisition **closed 2023-05-16**; combined entity led by **Mike Capone**; combined **40,000+** customers in **100+** countries | Qlik press release (GlobeNewswire dateline 2023-05-16) | 2023-05-16 |
| V10 | Open-source **Talend Open Studio retired 2024-01-31**, "no longer hosted or updated"; reasons stated as community adoption/contribution | qlik.com/us/products/talend-open-studio | page read 2026-09-14 |
| V11 | Talend **8** GA **November 2021**; **7.3** GA Feb 2020, retired May 2023, **EOL 2024-11-30**, extended support through **Dec 2026**; **7.2** EOL June 2022 | "Talend products lifecycle", help.qlik.com (last updated 2026-09-09) | table read 2026-09-14 |
| V12 | **CDC** retired Dec 2023, EOL Dec 2025; **MDM Server** retirement announced 2021-07-16, EOL 2024-12-31 | Same lifecycle page; "Talend MDM End-of-Life" (last updated 2026-09-10) | read 2026-09-14 |
| V13 | **Data Catalog**: 8.1 GA April 2024; 8.0 EOL 2024-12-31; 7.3 EOL Dec 2022 | "Talend Data Catalog lifecycle" (last updated 2026-09-09) | read 2026-09-14 |
| V14 | **Remote Engine Gen1** 2.14 GA Jan 2026; 2.13 retired Jan 2026, **EOL 2027-06-30**; **Gen2** R2026-03 GA Mar 2026 | "Talend Remote Engine lifecycle" (last updated 2026-09-09) | read 2026-09-14 |
| V15 | Qlik Talend Cloud: **capacity model** with three meters (Data Moved, Job executions, Job duration); **four tiers** (Starter, Standard, Premium, Enterprise); 25GB analytics capacity included | "Qlik Talend Cloud subscription options", help.qlik.com (last updated 2026-09-11) | read 2026-09-14 |
| V16 | Tier gating: Starter excludes CDC/customer gateway; Standard excludes SAP/mainframe and advanced transformation; Premium excludes SAP/mainframe; Enterprise adds SAP/mainframe + stewardship; Premium/Enterprise sync spaces/users/roles; Studio 8.0.1 R2024-05+ required for Premium/Enterprise | Same page | 2026-09-11 |
| V17 | **Minimum scheduling intervals**: 1h (Starter/Standard without gateway), 30min (Premium/Enterprise without gateway); continuous for DB with gateway; 15min (Standard SaaS), 5min (Premium/Enterprise SaaS) | Same page | 2026-09-11 |
| V18 | Studio: "**subscription Talend Studio licenses**"; design via drag-and-drop with "**native code generation**"; perspective availability depends on licence | Talend Studio User Guide, "What is Talend Studio?" (last updated 2026-08-27) | read 2026-09-14 |
| V19 | Routes/services available only in: Talend ESB, Data Services Platform, Real-Time Big Data Platform, MDM Platform, Data Fabric, Cloud API Services Platform, Cloud Data Fabric | Same page | 2026-08-27 |
| V20 | Built jobs are `.jar`/`.zip` runnable "independently of Talend Studio in standalone mode"; Docker-image and Microservice builds documented | Studio User Guide, "Building a Data Service Job as a Microservice" (last updated 2026-08-27) | read 2026-09-14 |
| V21 | Since Studio **R2025-02** artefacts are built with **Java 17** and **Camel 4**; **Java 21** mandatory for Studio from **R2026-06**; Runtime must be `8.0.1-R2025-02-RT`+; "**using unaligned versions is at risk**"; Maven 3.6.3+ | Studio cumulative patch note **R2026-08** (`Patch_20260821_R2026-08_v1-8.0.1`), update.talend.com | release date 2026-08-21 |
| V22 | TMC scope: tasks/plans, Studio projects, users/roles, engines; Users & Security, Environments/spaces, Promotions, Processing, Configurations (static IP, log export), Operations and Management | Talend Management Console User Guide (last updated 2026-09-09) | read 2026-09-14 |
| V23 | Studio artefact execution: "run them directly in the cloud or on Remote Engines"; Pipeline Designer pipelines run "in the application or on a Remote Engine Gen2" | Same page | 2026-09-09 |
| V24 | Remote Engine Gen2 runs in the customer VPC and "executes your artifacts on a **local Spark engine (default)**"; Cloud Engine for Design is the embedded starter engine | Remote Engine Gen2 architecture (last updated 2026-09-09) | read 2026-09-14 |
| V25 | Engine heartbeat **60s**; connection considered broken after **180s** without a heartbeat | TMC User Guide, "Remote Engine Gen2" (last updated 2026-09-09) | read 2026-09-14 |
| V26 | Talend Cloud regions map to 12 Qlik Cloud regions (list in §4.3); Azure United States - West has no Qlik Cloud counterpart; some capabilities "Not available in … Canada (Montréal)"; capability availability varies by region | "Accessing Talend Cloud applications", Installation and Upgrade Guide (last updated 2026-09-09) | read 2026-09-14 |
| V27 | Lineage/datasets from Studio jobs can be sent to **Qlik Cloud Catalog**, "generated at run time", gated to **Premium/Enterprise**; lineage-collection toggle; local lineage test | Studio User Guide, "Publishing datasets and lineage to Qlik Cloud" (last updated 2026-08-27) | read 2026-09-14 |
| V28 | `tQlikOutput` appears in new features of Studio **R2026-08** | Studio cumulative patch note R2026-08 | 2026-08-21 |
| V29 | "Talend Cloud puts powerful graphical tools, and **more than 900 connectors and components**…" | TMC User Guide, "What is Talend Management Console?" (last updated 2026-09-09) | read 2026-09-14 |
| V30 | Retired vs End of Life definitions; default extended support of up to 12 months after end of support | "Talend product lifecycle and compatibility" / "Talend products lifecycle" (last updated 2026-09-09) | read 2026-09-14 |
| V31 | Qlik's stated commitments (complementary portfolio, cloud-independent, support for existing investments, open to any source/target/architecture/methodology, increased R&D) | Qlik press releases 2023-01-05 / 2023-05-16; Capone blog (May 2023) | read 2026-09-14 |
| V32 | Analyst framing: "For **ten years**, Gartner has positioned Qlik as a Leader in its Magic Quadrant for Data Integration Tools"; Leader in the **2026** Gartner MQ for **Augmented Data Quality Solutions** for the seventh time | talend.com/products | read 2026-09-14 |
| V33 | Competitors verified to exist as described: **Informatica** (now part of Salesforce, per site lockup/footer), **IBM DataStage** (capabilities "now available within watsonx.data integration"), **SSIS** (SQL Server v17 domain metadata 2026-06-23), **dbt** (Fivetran–dbt Labs merger banner), **Matillion** ("Maia by Matillion"), **Fivetran** (900+ connectors), **Airbyte** ("Context Layer for AI Agents"), **Apache Hop** (TLP; **2.19** released 2026-08-16) | The nine vendor/project sites listed in §6 | read 2026-09-14 |
| V34 | Apache Hop graduated to Apache **Top-Level Project** (final days of 2021); ASF press release dated **2022-01-18** | hop.apache.org/categories/Graduation/ | read 2026-09-14 |

### 10.2 Flagged claims (used with the caveat stated inline)

| # | Claim | Why flagged |
| --- | --- | --- |
| F1 | Founding year/founders from first-party corporate history | No vendor-maintained corporate-history page survived verification; the 2005 date rests on two independent secondary sources (V1) |
| F2 | **Talend Open Studio for Data Integration first released 2006** | Secondary encyclopaedic sources only |
| F3 | **Qlik's acquisition value of Talend** (~$2.4bn) | Qlik's announcement explicitly declined to disclose terms; the figure is secondary-press only |
| F4 | The **legal nature/licence terms of the retired open-source distribution** | Not confirmed at a first-party licence statement |
| F5 | **List price, per-unit rates, developer-licence counting rule** | No price published; the Studio developer counting rule (named vs concurrent vs tier, and CI agents) was not published in material reviewed |
| F6 | **Meter computation edge cases** (retries, aborted tasks, partial runs) | Not documented in material reviewed |
| F7 | "Qlik Expands Integration with the Databricks Data Intelligence Platform", 2025-06-10 (streaming CDC to Unity Catalog; Iceberg optimisation via Talend) | Headline and date seen in a secondary aggregator; release body not opened at source |
| F8 | Existence of any published deadline forcing Studio customers to Qlik Cloud | No such statement found; absence of evidence, not evidence of absence |
| F9 | Informatica/Salesforce transaction **completion date** (~November 2025) | Ownership confirmed by the vendor's own site footer; the completion date was not confirmed at a first-party release |
| F10 | Any specific estate's deployed engine/Studio/Runtime versions | Must be read from the live estate; CMDBs are unreliable exactly here (§9.5, G2) |
| F11 | **Log retention and tamper-evidence** of exported Talend logs | Not settled by the documentation reviewed |
| F12 | **Agentic data engineering / MCP server production readiness** | Appears as product-page positioning and tours, not as dated release-note entries |
| F13 | The literal sentence "**Talend Studio is Eclipse-based**" in a current vendor source | Supported by Eclipse-lineage artefacts in current docs (p2 update option, Feature Manager, workspace, `.metadata/.log`) but not located as a verbatim current vendor sentence |
| F14 | "JavaJet templates generate a single Java file per job" | Community-source statement on Qlik's forum (2016), not vendor documentation |
| F15 | Third-party characterisations of pricing as "opaque" / customer support complaints | Analyst/community commentary, not primary evidence |

### 10.3 Rejected claims (they circulate; they are wrong or misleading)

| Claim | Why it is rejected |
| --- | --- |
| "**Qlik shut down the open-source edition right after buying Talend**" | The retirement date is **2024-01-31**, roughly nine months after the close, and the vendor's stated reasons are community adoption and contribution — not the acquisition. The outcome (no free maintained platform) is right; the causal story is not **(verified** — qlik.com Talend Open Studio page) |
| "**Talend was acquired by Qlik for $2.4 billion**" | Terms were explicitly **not disclosed**. The $2.4bn figure belongs to the **Thoma Bravo** transaction and has been carried across by repetition **(verified** — 2023-01-05 release: "Terms of the proposed transaction were not disclosed") |
| "**Thoma Bravo completed the Talend take-private in July 2021**" | The **tender offer** completed 2021-07-29; the **acquisition** completed **2021-09-02**. Both dates are correct for different events **(verified** — Thoma Bravo release) |
| "**Talend 7.3 is still supported until the end of 2026, so there is no urgency**" | Talend 7.3's **EOL was 2024-11-30**. What runs to December 2026 is *purchased extended support* on an end-of-life product, at Qlik's discretion, with no enhancement or service-release stream **(verified** — lifecycle page) |
| "**Talend Studio is Eclipse, so it is open source and we can fork it**" | Eclipse *tooling lineage* is not an open-source licence for the product. Studio is subscription-licensed; the free open-source distribution was **retired on 2024-01-31** **(verified)** |
| "**Jobs are just Java, so migrating off Talend is a lift-and-shift**" | The `.jar` is a **build output**; the design of record is a Studio repository object requiring Studio and a licence to maintain. Execution is portable; maintenance is not (§7.4) |
| "**Lineage comes free with the platform**" | Lineage to Qlik Cloud requires **Premium or Enterprise** and only works for **supported components**; profiling/preview scope is explicitly limited where the connector lists differ **(verified** — Studio User Guide, 2026-08-27) |
| "**The Remote Engine is one component; check its version and you are done**" | **Gen1** (Studio-task execution, `2.x`) and **Gen2** (Pipeline Designer, `RYYYY-MM`) are separate products with separate lifecycles (V14) |
| "**Talend MDM and CDC are covered by extended support**" | Both are past EOL (MDM 2024-12-31; CDC December 2025). The standard extended-support window is up to 12 months **after** end of support, not indefinitely **(verified** — lifecycle pages) |
| "**A capacity-metered price is just a discount conversation**" | The meters (Data Moved, Job executions, Job duration) are **undefined in the material reviewed**; without definitions the price is not comparable to anything (§3.5, F6) |

---

## 11. What Could Not Be Verified

An explicit list of what this guide **does not know**, so that silence is never mistaken for confirmation. Each item is a task for whoever acts on this material.

1. **The founding date and founders at a first-party source.** The 2005 date and the Diard/Bonan pairing are corroborated by two independent secondary sources (V1), but no vendor-maintained corporate-history page survived verification. **Treat 2005 as well-corroborated, not first-party.**
2. **The financial terms of the Qlik acquisition.** Qlik stated that terms were not disclosed. The widely cited ~$2.4bn belongs to the 2021 Thoma Bravo transaction.
3. **The complete historic SKU list and what each old SKU entitled a customer to.** Only seven named subscription products remain enumerated in current documentation (V19); older SKU names and their entitlements were not confirmed.
4. **The legal and licence detail of the retired open-source distribution** — the repository, the licence text, and whether any artefact of it remains lawfully distributable.
5. **Commercial terms: list price, per-unit rates, and the developer/desktop licence counting rule** (named user vs concurrent developer vs tier, and whether CI build agents consume entitlements). No price is published anywhere reviewed; this must be answered in writing by Qlik. **Do not budget on an assumption here.**
6. **How the three capacity meters are computed in edge cases** — retries, aborted tasks, idle time, failed executions.
7. **The Qlik–Databricks integration announcement of 2025-06-10** — headline and date observed via a secondary aggregator; release body not read.
8. **Any published convergence deadline for client-managed Talend.** None was found; no such claim is made in this guide.
9. **The Informatica/Salesforce transaction completion date.** Ownership is confirmed by the vendor's own site; the date is not.
10. **Any specific estate's true deployed versions.** Version drift between CMDB, repository and engines is normal; only a live read settles it.
11. **Log retention periods and tamper-evidence properties** of exported Talend/TMC logs and task history.
12. **Whether the agentic data-engineering and MCP capabilities are production-supported** offerings or positioning.
13. **A current vendor sentence stating "Talend Studio is Eclipse-based"**; the Eclipse lineage is inferred from the tooling's own artefacts (p2 updates, Feature Manager, workspace/metadata layout).
14. **The JavaJet code-generation pipeline as vendor-documented behaviour**; the only source located is a 2016 community thread.
15. **Third-party pricing/support sentiment.** The "opaque pricing" and support-responsiveness commentary that circulates is secondary and is reported here only as sentiment, never as fact.
16. **Whether Qlik Cloud Government's data-integration capability set is complete** relative to the commercial tiers, and whether it satisfies any specific regulator.

---

## 12. Glossary

A working dictionary for *this* guide: the terms that carry a specific meaning in the Talend/Qlik lineage, plus the naming decoder. Generic integration vocabulary (ETL, ELT, CDC, DQ) is deliberately omitted — the discipline guide owns it.

- **Talend:** the vendor, founded 2005, IPO'd 2016, taken private by Thoma Bravo in 2021, acquired by Qlik in 2023. Also, loosely and confusingly, the product family that survives in current documentation names.
- **Talend Studio:** the design-time desktop application — drag-and-drop Jobs and Routes with **native code generation**; subscription-licensed; Eclipse-lineage tooling. What "Talend" means to a developer.
- **Job:** the unit of integration design in Studio; assembled from components, may contain subJobs; the thing that becomes a build artefact.
- **Route:** the application-integration design type (mediation, messaging, REST) available only in the higher subscription products; runs on Talend Runtime.
- **Data Service:** a service-exposing job design type (e.g. via `tRESTRequest`); buildable as a microservice or Docker image; licence-gated like Routes.
- **Component:** the palette unit (`tMap`, `tDBInput`, `tRESTRequest`, `tQlikOutput`, …). The `t` prefix is the marker of a Talend component in code, logs and documentation.
- **Joblet:** a reusable sub-design referenced by multiple jobs; part of the repository, part of the migration inventory.
- **Context (variables):** the parameterisation mechanism by which a job takes environment-specific values; the thing that makes one job serve dev/test/prod.
- **Repository:** the design-time metadata store holding jobs, routes, services, routines, contexts and versions. **The design of record** — not the runtime database.
- **Referenced project:** a second repository project whose items a job can reference; a common source of unexpected migration coupling.
- **Native code generation:** the vendor's term for Studio turning a design into executable code; the output is a `.jar` (or `.zip` with external config) runnable without Studio.
- **JavaJet:** the template technology identified in community sources as the generator behind Studio job code; **not** documented as product behaviour by the vendor.
- **Talend Administration Center (TAC):** the client-managed web console for projects, users, scheduling and software updates; the on-premises control plane.
- **Talend JobServer:** the classic client-managed execution service that TAC drives.
- **Talend Runtime:** the OSGi/Apache-Camel container that executes Routes, Data Services and Microservices; must be version-aligned with Studio.
- **Talend Management Console (TMC):** the cloud control plane — tasks, plans, projects, users, roles, engines, spaces, promotions, configurations. The Qlik-era successor in function to TAC.
- **Remote Engine (Gen1):** the customer-hosted engine that runs Studio-built tasks published to Talend Cloud; versioned `2.x`.
- **Remote Engine Gen2:** the customer-hosted engine for **Talend Cloud Pipeline Designer**; runs on a **local Spark engine by default**; versioned `RYYYY-MM`. A different product from Gen1.
- **Cloud Engine for Design:** the small engine embedded in TMC for getting started; not the production path.
- **Talend Cloud Pipeline Designer:** low-code cloud pipeline authoring; runs on Gen2 engines.
- **Talend Cloud Data Inventory / Data Preparation / Data Stewardship:** the shared dataset inventory, self-service cleansing, and stewardship workflow applications.
- **Talend Cloud API Designer / API Tester:** API design and testing; the latter has a free edition.
- **Talend Data Catalog:** the catalog product with its own lifecycle (8.1 current, GA April 2024).
- **Talend MDM:** master data management; the **MDM Server** component reached EOL **2024-12-31**.
- **Talend Change Data Capture:** the standalone CDC product; retired December 2023, EOL December 2025.
- **Talend Open Studio / the open-source edition:** the free distribution, **retired 2024-01-31**; not the same thing as the commercial Talend Studio.
- **Qlik Talend Cloud:** the Qlik-era cloud subscription combining Qlik Talend Data Integration and Talend Cloud; four tiers (Starter, Standard, Premium, Enterprise) on a capacity model.
- **Talend Data Fabric:** the on-premises platform brand, still used ("Talend Data Fabric — powered by Qlik"), covering integration + quality + governance + catalog.
- **Qlik Cloud:** Qlik's cloud platform (analytics, catalog, lineage, data products); the destination for Talend-generated datasets and lineage.
- **Qlik Data Gateway — Data Movement:** the Qlik-side gateway that enables continuous/CDC replication in the higher tiers.
- **Data Moved / Job executions / Job duration:** the three **capacity meters** on which Qlik Talend Cloud subscriptions are measured.
- **Retired (Qlik lifecycle term):** not available for general sale; no enhancements or service releases; existing keys capped at the EOL date.
- **End of Life / End of Support (Qlik lifecycle term):** not for sale or support; no service releases or patches; limited extended support at Qlik's discretion.
- **Extended support:** purchasable support beyond EOL — by default up to **12 months** after end of support; for Talend 7.3, purchasable **through December 2026**.
- **Cymbal Bank:** the illustrative (fictional) institution used for the worked example in §9. The only bank persona in this guide.

### The naming decoder, for reading old documents

| You read (in a document, runbook or contract) | What it means today |
| --- | --- |
| "Talend Open Studio" | The **retired** free distribution (ended 2024-01-31). Today's "Talend Studio" is the commercial product |
| "Talend DI" / "Talend Data Integration" | The integration lane; under Qlik it appears as **Qlik Talend Data Integration** capability inside Qlik Talend Cloud |
| "Talend ESB" / "Talend Data Services Platform" | The application/API-integration subscription products; the licence gate for Routes and Data Services |
| "Talend Big Data Platform" / "Real-Time Big Data Platform" | The Spark runtime lane for batch/streaming big-data jobs |
| "Talend Cloud Data Fabric" / "Cloud API Services Platform" | Cloud-side equivalents of the above; now folded into the Qlik Talend Cloud tiers |
| "Talend Data Fabric" | The on-premises platform brand, still current, now "powered by Qlik" |
| "Talend Cloud" | The cloud product family now delivered as **Qlik Talend Cloud** |
| "TAC" vs "TMC" | **TAC** = client-managed console; **TMC** = cloud console. Which one your estate has determines which half of §5 applies to you |
| "Remote Engine" (unqualified) | Ambiguous **and** dangerous: ask **Gen1 or Gen2** before planning anything |
| "Talend Trust Score" | Now **Qlik Talend Trust Score™** |
| "Talend Data Catalog" | Same product; **8.1** current; its lifecycle is separate from Talend 8 |

---

## 13. Cross-References and Further Reading

### 13.1 In this repository

- [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) — **the discipline this guide deliberately does not re-explain**: integration patterns, CDC mechanics (§6), data-quality theory (§7), the tool-selection framework and its §3 comparison, §11 matrix and §12 decision tree. §6 here is a positioning table; the *choice* framework lives there.
- [data_governance_guide.md](data_governance_guide.md) — the **governance operating model** under which §7's audit and lineage requirements are designed.
- [data/data_lineage_tools.md](data/data_lineage_tools.md) — the **lineage tooling landscape**; read it alongside §7.2 for what a lineage capability gap costs to close.
- [data/data_profiling_guide.md](data/data_profiling_guide.md) — **profiling practice**; Talend Studio implements profiling, so this is the discipline behind its Profiling perspective.
- [data/data_fabric_guide.md](data/data_fabric_guide.md) — the **data-fabric architecture** Talend markets its platform into; read it for the pattern, this guide for the vendor's implementation of it.
- [control_m_guide.md](control_m_guide.md) — the **scheduler landscape** that triggers most Talend estates; §8's migration paths are incomplete without it.
- [databricks_guide.md](databricks_guide.md) and [data/on_prem_lakehouse_guide.md](data/on_prem_lakehouse_guide.md) — the **lakehouse side** of the same pipelines; read them beside §4.4's list of native targets.
- [data/data_pipeline_guide.md](data/data_pipeline_guide.md) and [data/dataops_guide.md](data/dataops_guide.md) — **pipeline engineering and the operating model** that §8's migration paths must fit into, and the practice behind the "test cases, promotions, spaces" control set in §7.1.
- [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) — the **API/ESB lane** Talend competes in with Routes and Data Services.
- [architecture/enterprise_architecture_guide.md](architecture/enterprise_architecture_guide.md) — where a platform of this class sits in an architecture baseline.
- [audit_as_code_guide.md](audit_as_code_guide.md) — how the **evidence join** in §7.2 gets built rather than promised.
- [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) — the **operational-resilience anchor** behind §7.3 and §9.5.
- [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — the **risk/compliance system context** in which a bank's Talend estate usually sits.

### 13.2 Primary sources to re-check before acting

Naming, versions, licences and lifecycle dates go stale in months — **re-verify rather than re-quote**. The URLs this guide actually leans on:

- **<https://help.qlik.com/talend/en-US/customer-support-statements/Cloud/product-end-of-life-planning>** — the Talend lifecycle table (Talend 8 / 7.3 / 7.2 and the extended-support position). **Check this first; if the dates have moved, everything downstream of §3.6 needs a pass.**
- **<https://help.qlik.com/talend/en-US/customer-support-statements/Cloud/tdc-end-of-life-planning>** and **<https://help.qlik.com/talend/en-US/customer-support-statements/Cloud/talend-remote-engine-lifecyle>** — Data Catalog and Remote Engine Gen1/Gen2 lifecycles. The Engine page is where Gen1/Gen2 confusion gets resolved.
- **<https://help.qlik.com/talend/en-US/customer-support-statements/Cloud/talend-cloud-compatibility>** — which Studio and Engine versions are compatible with Talend Cloud.
- **<https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Admin/subscription-options-QTC.htm>** — the four Qlik Talend Cloud tiers, their gating, and the minimum scheduling intervals. The single most decision-relevant page in the documentation set.
- **<https://help.qlik.com/talend/en-US/installation-guide-linux/Cloud/talend-cloud-application-url>** — the region→URL mapping and the capability-availability restrictions.
- **<https://update.talend.com/Studio/8/updates/latest/PATCH_RELEASE_NOTE.html>** — the current monthly Studio patch, its release date, the Java/Camel requirements and the **CVE list**. The fastest way to date-check what Qlik is actually shipping.
- **<https://help.qlik.com/talend/en-US/studio-user-guide/8.0-R2026-08>** — the Studio user guide (doc version string itself tells you the release train you are reading).
- **<https://www.qlik.com/us/products/talend-open-studio>** — the open-source edition's retirement notice. Read it verbatim before repeating any version of the story.
- **<https://www.qlik.com/us/legal/product-terms>** — Qlik Cloud Subscriptions product description (the **legal** statement of what a tier includes, which outranks any product page).
- **<https://www.thomabravo.com/press-releases/thoma-bravo-completes-acquisition-of-talend>** and **<https://www.thomabravo.com/press-releases/qlik-intends-to-acquire-talend-delivering-comprehensive-data-integration-to-accelerate-cloud-modernization>** — the primary ownership-history dates and terms.
- **<https://www.qlik.com/us/news/company/press-room/press-releases/qlik-acquires-talend>** — the close of the Qlik acquisition.
- **<https://hop.apache.org/>** — if you are evaluating the open-source escape hatch, check the current release number; it moves.

**How to keep this guide honest:** treat the ownership lineage and the lifecycle tables as *claims with dates*. The lifecycle tables are the ones that move; the ownership history is stable. Re-read the lifecycle page first, then the Studio patch note, then the subscription-options page — if those three have shifted, §3, §5 and §8 all need a rewrite rather than an edit.

---

## 14. Closing Summary

Talend is a platform whose *technology* is more durable than its *ownership*, and the whole of this guide is a description of that tension.

- **The vendor has changed hands twice in five years, and both deals are dated and sourced.** Founded 2005 in France; Nasdaq IPO at $18.00/ADS in July 2016; taken private by Thoma Bravo for $66.00 per share in an all-cash transaction valued at approximately $2.4bn, completed 2 September 2021; acquired by Qlik from another Thoma Bravo portfolio company, announced 5 January 2023 at *undisclosed* terms and closed 16 May 2023. The frequently repeated "$2.4bn Qlik deal" is not a fact; it is the 2021 number carried across by repetition.
- **The platform's architecture is a design tool, a generator, a control plane and a set of engines — and the engines are the part that will break your upgrade plan.** Jobs are designed in Studio, compiled to Java artefacts you can run without Studio, published to Talend Cloud or a client-managed console, and executed by engines whose names, versions and lifecycles are genuinely distinct: Gen1 versus Gen2, `2.x` versus `RYYYY-MM`, Studio versus Runtime, all of them requiring alignment and one of them carrying an explicit vendor warning that misalignment is a risk.
- **The licensing moved from named products to capacity meters, and the tier gates are capability gates.** Starter, Standard, Premium and Enterprise; three meters (Data Moved, Job executions, Job duration); lineage-into-Qlik-Cloud, API integration and SAP/mainframe movement all behind Premium or Enterprise; minimum scheduling intervals themselves a tier feature. No price is published anywhere, and this guide quotes none.
- **The open-source story is remembered wrongly.** The free distribution was retired on 31 January 2024 for stated community-adoption and contribution reasons, not in the immediate aftermath of the acquisition — and what that retirement did and did not do to the licence position of existing builds is not something the vendor documentation settles.
- **The Qlik era has shipped more than the cynics allow and less than the marketing implies.** A live monthly Studio train with CVE remediation, Java 17/Camel 4/Java 21 modernisation, a `tQlikOutput` component, lineage and dataset publishing into Qlik Cloud, and synchronised tenants are all dated facts. The agentic data-engineering and MCP story is positioning. And no vendor source dates the end of the client-managed line that most regulated estates still run on.
- **For a bank, the tool produces the raw material of audit, not the audit.** Versions, tests, promotions, roles and run-time lineage graphs exist in different places at different granularities, and the joined artefact that ties change-approval to executed run to data moved is the institution's to build. Separation of duties is configuration, not a property. Residency must be evidenced per tenant, per capability, per region.
- **The estate decision is an EOL decision first and a strategy decision second.** Talend 7.3's end of life passed in November 2024 with purchased extended support closing at the end of 2026; MDM and standalone CDC are already past theirs; the open-source fallback is gone. Decide the 7.3 cliff on the published date, decide the tier on capability rather than discount, decide the target architecture in the same window rather than after the upgrade — and write the exit into the contract while you still have leverage.
- **The test you can apply this quarter:** take your ten most regulated pipelines and ask which of them can be rebuilt, from evidence you already hold, on a platform you do not currently own. If the answer is "none of them, and we cannot prove what any of them does", the platform is not the problem the next programme should be scoped around.

Talend's engineering survived the IPO, the take-private and the acquisition; what an institution cannot outsource to any of those events is the obligation to know what its pipelines do, who changed them, and what it will do when the vendor's roadmap moves again — **the integrated record.**
