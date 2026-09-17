# OpenText Operations Bridge — The Correlated Event: An Enterprise ITOM Deep-Dive
*Operations Bridge is the enterprise ITOM platform a remarkable number of banks own without anyone currently on staff having chosen it. This guide is the orientation, the ownership chain and the currency audit for that estate: what Operations Bridge is and the problem class it solves — a single correlated event stream over a heterogeneous estate — the name lineage from HP OpenView through HPE to Micro Focus to OpenText, and what that lineage means in practice (branding churn, support and licensing continuity, documentation and part-number migration, acquisition-driven consolidation risk). It walks the product family member by member — Operations Bridge Manager, the Operations Agent, the agentless SiteScope line, the retiree Operations Bridge Reporter, Business Value Dashboard, the RTSM/Universal CMDB foundation, Operations Orchestration and the containerised delivery — with the current status of each, the evidence for it, and an explicit statement wherever status could not be established. It then works the architecture and event pipeline, the topology/CMDB dependency, the vendor-neutral engineering of the correlation problem, the AIOps claims versus what ships, the mainframe and MFT coverage a bank actually cares about, the deployment and integration reality, an honest peer-by-peer positioning, the regulated-institution angle, and a renewal assessment at the fictional Cymbal Bank. It closes with a claims audit graded by source quality and date, an explicit list of what could not be verified, and a glossary. The currency-and-roadmap evidence is the point: current release numbers with dates, which components are active, in maintenance or sunset, and which facts the vendor simply does not publish.*

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology / Operations — this is the repository's **first ITOM product guide**. It owns the ITOM *product* and the *event-management architecture* built on it: the component inventory, the ingestion-and-correlation pipeline, the topology dependency, component-by-component currency evidence, and renewal economics. It deliberately does **not** re-derive adjacent content: operational-resilience and DR *frameworks* belong to [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); the CMDB as a *data-model* subject belongs with the data-architecture content; the incident-management *process* belongs to [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md); the AIOps *category* belongs to [ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md); observability *engineering* — tracing, metrics, SLOs — belongs to [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) and [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md). **"Monitoring" in a bank is a contested term spanning all of these**, so this guide states its boundary wherever it touches one.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** OpenText/Micro Focus product documentation and release communications fetched live this pass — the OpenText Documentation Portal (`docs.microfocus.com`: Operations Bridge Manager, SiteScope, Operations Agent, Operations Orchestration and Universal Discovery/CMDB doc sets), OpenText Support release notices (`portal.microfocus.com`: OBM 24.4, Operations Bridge Express (SiteScope) 25.4, Operations Agent 12.25, OO 2023.05, and the OBR obsolescence PDF), the product page `opentext.com/products/ai-operations-management` plus its **23 December 2024** web.archive.org snapshot, the **Operations Bridge – SaaS** (`260-000148-004 | O | 01/24`) and **Full-Stack AIOps: Operations Bridge Integrations** (`263-000031-003 | O | 01/24`) brochures, the **OBR → OPTIC Reporting** data sheet (`260-000252-001 | O | 06/23`), and the vendor-authorised **OBM-EPAC 24.2** event-processing course outline; corporate investor announcements — OpenText's **2022-08-25** agreement release and **2023-01-31** closing release for Micro Focus, HPE's **2017-09-01** SEC-filed completion announcement of the spin-merge and Micro Focus's completion release; HP OpenView/OMi lineage from the OMi 10.00/10.01/10.6x documentation trail. Where a documentation site resisted extraction (they are JavaScript-heavy), the snapshot used is named and dated in the text. Sources fetched **2026-09-17**. Vendor documentation is **primary for capability, naming and version history** but **marketing for performance, scale and outcome claims** — such claims are labelled *claim, vendor-sourced*.
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — plain filenames):** [AI for IT](ai_llm/ai_for_it_guide.md) (**THE AIOps-category companion — the coinage, the vendor stacks, anomaly detection and RCA theory; §7 here is only the claims audit**), [Operational Support Frameworks](operational_support_frameworks_guide.md) (**THE ITSM-process companion — ITIL 4, the service desk, incident/problem/change, the CMDB discussion**), [Zero-Downtime System Design](zero_downtime_system_design_guide.md) (the availability/SLO companion), [OpenShift Workload Availability](openshift_workload_availability_guide.md) (the platform monitoring stack), [Technology Lifecycle Management](technology_lifecycle_management_guide.md) (the lifecycle method behind every §3 sunset verdict), [DataOps](data/dataops_guide.md) (the CMDB population-quality angle of §5), [FinOps](finops_guide.md) (the §12 renewal economics), [Control-M](control_m_guide.md) and [Control-M External Conditions](control_m_external_conditions_guide.md) (**the batch-scheduling companion — the honest alternative to OBM's batch view in §8 and §12**), [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md), [SecOps](secops_guide.md), [Kargo GitOps](kargo_gitops_guide.md), [AgentOps](ai_llm/agentops_guide.md)
> **Companion guides (banking/, prefix `../banking/`):** [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (**THE resilience cross-ref — impact tolerances and the evidence trail §11 asks about**), [Core Banking Systems](../banking/core_banking_systems_guide.md), [Payment Rails](../banking/payment_rails_guide.md), [MQ Disaster Recovery](ibm_mq_disaster_recovery_guide.md)

---

**How to use this guide.** §1 is the orientation, the problem class, the honest market position and the **name-decoder table** you need when you inherit an estate already running it. §2 is the dated ownership chain and what it means for a customer. §3 is the product family member by member with current status and evidence — **the currency section, and the most perishable; every version claim carries a date**. §4 is architecture and the event pipeline, including who watches the watchmen. §5 is the topology/CMDB dependency. §6 states the correlation problem vendor-neutrally, useful even if you never deploy this product. §7 audits the AIOps claims. §8 is mainframe, legacy and MFT coverage. §9 is deployment, integration and scale. §10 is the competitive landscape peer by peer. §11 is the regulated-institution angle, condensed and cross-referenced. §12 is the Cymbal Bank renewal assessment. §13 is the claims audit, §14 the honest gaps, §15 the glossary, §16 the cross-references, §17 closes. **Integrity convention:** ✅ = verified at a named primary source this pass; ⚠ = flagged, uncertain or single-sourced; *claim, vendor-sourced* = a vendor performance/scale/outcome claim accepted as a claim, not as fact; **rejected** = contradicted by evidence found this pass.

---

## Table of Contents
1. [The overview and the identity](#1-the-overview-and-the-identity)
2. [Corporate lineage and what it means in practice](#2-corporate-lineage-and-what-it-means-in-practice)
3. [The product family, member by member](#3-the-product-family-member-by-member)
4. [Architecture and the event pipeline](#4-architecture-and-the-event-pipeline)
5. [Topology and the CMDB dependency](#5-topology-and-the-cmdb-dependency)
6. [Event correlation and the noise problem](#6-event-correlation-and-the-noise-problem)
7. [AIOps and the analytics claims](#7-aiops-and-the-analytics-claims)
8. [Mainframe, legacy and MFT coverage](#8-mainframe-legacy-and-mft-coverage)
9. [Deployment, integration and scale reality](#9-deployment-integration-and-scale-reality)
10. [Competitive landscape and honest positioning](#10-competitive-landscape-and-honest-positioning)
11. [The regulated-institution angle](#11-the-regulated-institution-angle)
12. [The Cymbal Bank worked example](#12-the-cymbal-bank-worked-example)
13. [The claims audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-references and further reading](#16-cross-references-and-further-reading)
17. [Closing summary](#17-closing-summary)

---

## 1. The overview and the identity
### 1.1 The one-paragraph answer
Operations Bridge is an **enterprise IT operations management (ITOM) platform whose job is to turn a heterogeneous estate's telemetry into one correlated, de-duplicated event stream mapped onto services — so that an operator sees "the payments switch is degraded" rather than four thousand raw alerts.** It collects events, metrics and topology from hundreds of sources (its own agents, agentless monitors, network tools, third-party monitoring platforms, ITSM systems), normalises them, correlates them against a topology model, and presents service health to an operator console. It is not an observability database, not a CI/CD tool, and not the service desk; it sits between them.

> **The problem class:** a large regulated enterprise runs 30–300 technology domains, each with its own native monitoring tool, alert format and severity vocabulary. The native tools are individually fine and collectively unmanageable. The estate needs a *federating* layer that is **not** the source of truth for any single domain but *is* the source of truth for "what is wrong with my services right now". Operations Bridge is one implementation of that layer.

### 1.2 What the product is
| Aspect | Description |
|---|---|
| **Category** | Enterprise ITOM — event management / AIOps platform / service-health console |
| **Core artifact** | A correlated event stream mapped to business services via a topology model |
| **Primary console** | Operations Bridge Manager (OBM) — dashboards, event browser, health indicators |
| **Collection** | Operations Agent (agent-based), SiteScope (agentless), Operations Connectors and Management Packs |
| **Correlation** | Stream-based (SBEC), **topology-based (TBEC)**, machine-learning (AEC), plus dedup, suppression, enrichment |
| **Data layer (current)** | **OPTIC Data Lake** (containerised, Vertica-powered) — replaced OBR's reporting role from 2020 |
| **Topology** | **RTSM** — the model embedded in OBM, descended from the **Universal CMDB**; **Universal Discovery and CMDB** is the discovery product |
| **Automation** | Operations Orchestration, management-pack remediation, xMatters/on-call, ITSM closed-loop incident processing |
| **Delivery** | On-premises (Classic and Container Edition), SaaS, hybrid, private cloud, managed service, air-gapped AI |
| **Vendor** | OpenText since 31 Jan 2023 — previously Micro Focus (2017–2023), HPE (2015–2017), originally HP |
| **Current name** | **OpenText AI Operations Management**; `/products/operations-bridge` **301-redirects** to `/products/ai-operations-management` (archive snapshot 23 Dec 2024) ✅ |

### 1.3 The name lineage in one line
`HP OpenView` (1990s; rebranded **HP BTO Software** in 2007) → `HP Operations Manager` (**OM**, successor to **OpenView Operations / OVO**) → `HP Operations Manager i` (**OMi**, ~2011, the new correlation-and-service-model architecture) → `Operations Bridge` (the umbrella from ~2012, OMi at its core) → `HPE Operations Bridge` (2015) → `Micro Focus Operations Bridge` (spin-merge completed 1 Sep 2017) → `OpenText Operations Bridge` (acquisition closed 31 Jan 2023) → `OpenText AI Operations Management` (current framing, "Operations Bridge" retained as the alias).

**The practical consequence is not trivia.** An existing estate contains *naming strata*: a config file saying OVO, a policy named for OM, a URL saying OMi, a contract line saying Micro Focus Operations Bridge, a doc link saying `docs.microfocus.com`, and a landing page now saying "AI Operations Management". Decoder first, decisions second.

### 1.4 The name-decoder table — for an inherited estate
| Acronym | Expansion | What it is | Current-day relevance |
|---|---|---|---|
| **OVO** | OpenView Operations | The original HP agent/server event console (pre-2007) | Historical; its `opc` agent family still appears in legacy policy and agent inventories ✅ |
| **OM** | Operations Manager (HP/HPE) | The classic console that replaced OVO; distinct from OMi | Legacy agent estate; "Operations Manager for UNIX / for Windows" still named in vendor integration tables ✅ |
| **OMi** | Operations Manager i | The ~2011 new-generation console: RTSM model + policy correlation + service health. The architectural ancestor of OBM | Still the internal name in doc URL paths (`OMi/10.6x/...`) ✅ |
| **OBM** | Operations Bridge Manager | The current event and service-health console; the suite's core | **Current** — the thing you administer ✅ |
| **OA** | Operations Agent | The host agent collecting metrics/events and executing local actions | **Current** ✅ |
| **RTSM** | Run-Time Service Model | The embedded CI-and-relationship model driving service health and topology correlation | **Current but embedded** — not a separately-sold product ✅ |
| **UCMDB / UD** | Universal CMDB / Universal Discovery and CMDB | The discovery engine and CMDB product that can populate RTSM | **Current** — "Universal Discovery and CMDB **26.1**" on the vendor's trending list ✅ |
| **BVD** | Business Value Dashboard | Flexible dashboarding of business/IT KPI data | **Still referenced 2023–2024**; no EOL notice found ⚠ |
| **OBR / SHR** | Operations Bridge Reporter / Service Health Reporter | The historical Vertica-based reporting product | **OBSOLESCENT** — end of sale 1 Oct 2021 ✅ |
| **OO** | Operations Orchestration | Runbook-automation and orchestration engine | Distinct portfolio product, **current through 24.1.1** ✅ |
| **CDF** | Container Deployment Foundation | The container/Kubernetes delivery foundation of the 2019–2021 era | **Named-product status not established** ⚠ — the capability survives as "containerized operations bridge" and Container Edition |
| **SiteScope** | — (product name) | Agentless monitoring via remote protocols, no agent on the target | **Current** — renamed **Operations Bridge Express (SiteScope)**; 25.4 released 28 Oct 2025 ✅ |
| **TBEC / SBEC** | Topology-Based / Stream-Based Event Correlation | The two rule-driven correlation modes | **Current** ✅ |
| **TBEA** | Time-Based Event Automation | Scheduled/condition-based automation of event handling | **Current** ✅ |
| **AEC** | Automated Event Correlation | ML correlation operated against OPTIC DL, for Classic and Containerized OBM | **Current** ✅ (vendor training syllabus) |
| **CLIP** | Closed-Loop Incident Processing | Event↔incident creation, sync and cross-system closure | **Current** ✅ |
| **ETI** | Event Type Indicator | The type-level event classification enabling sophisticated processing | Current architectural term ✅ |
| **EPI** | Event Processing Interface | Scripting hooks into the OBM event pipeline | Current ✅ |
| **MAO** | Maintenance-Aware Operations | Scheduled-maintenance/downtime windows | Concept certain; verbatim primary definition **not captured** ⚠ |
| **OPTIC / OPTIC DL** | OpenText OPTIC platform / Data Lake | Containerised, Vertica-powered common store behind analytics and reporting | **Current** ✅ |
| **NOM / NNMi** | Network Operations Management / Network Node Manager i | The network-monitoring siblings feeding OPTIC | Separate co-owned products ✅ |
| **DCA** | Data Center Automation | The server/provisioning automation sibling | Separate product co-sold in OBR-era suites ✅ |
| **SMAX** | Service Management Automation X | OpenText's ITSM — the native event-to-incident target | Separate product ✅ |
| **ITOM** | IT Operations Management | The solution-area label for this portfolio | Category label ✅ |

### 1.5 The honest market-position statement
Operations Bridge's defensible position is **federation and consolidation, not best-of-breed depth in any single domain.** Its hard-to-replicate strength is breadth of *ingestion*: the vendor claims integrations across "200+ domains" and "1000+ out-of-the-box integrations" — *claims, vendor-sourced* and unauditable from outside — but the underlying connector and management-pack catalogue is real, unusually wide, and is why the product survives in estates that would otherwise have ripped it out. Its second strength is **on-premisability and air-gap capability** at a time when the loudest observability vendors are SaaS-first — a genuinely rare property for a regulated buyer. Its structural weakness is that the market's centre of gravity has moved: shortlists are led by SaaS observability (Dynatrace, Datadog, Splunk ITSI, New Relic) and ITSM-native event management (ServiceNow ITOM), and this product competes on consolidation rather than on the quality of its AI or its developer experience. Its second weakness is **portfolio risk from ownership churn**: four owners in eight years, a rebranded product page, a renamed agentless product, a retired reporting product, and documentation split across two domains. The honest summary: *a consolidator's tool with a real moat in heterogeneous legacy estates and a shrinking claim on greenfield ones.*

---

## 2. Corporate lineage and what it means in practice
### 2.1 The ownership chain, dated and attributed
| Date | Event | Structure / figures | Source (rated) | Status |
|---|---|---|---|---|
| 1990s | **HP OpenView** — the HP network and systems management family | Product family | Product-history literature | ✅ (secondary; exact origin date ⚠) |
| 2007 | Rebranded **HP BTO Software** within the HP Software division | Rebrand only | Same secondary source | ✅ (secondary) |
| ~2011 | **HP Operations Manager i (OMi)** enters the 10.x line: RTSM + policy correlation + service health | OMi 10.00, 10.01, 10.6x documented | HP/Micro Focus OMi documentation trail — **primary vendor docs** | ✅ |
| 1 Nov 2015 | **HP splits** into HP Inc. and Hewlett Packard Enterprise; enterprise software goes to HPE | Corporate separation | Widely reported | ✅ |
| Sep 2016 | **HPE announces the spin-merge** of its software business with Micro Focus | Announced transaction | Contemporaneous reporting | ✅ |
| **1 Sep 2017** | **Spin-merge completes** | **~US$8.8bn**; delivered ~$8.8bn to HPE and its stockholders; combined company stated at $4.4bn revenue | Micro Focus completion release (1 Sep 2017) **and** HPE's completion announcement filed with the SEC (Ex-99.1, 1 Sep 2017) — **primary corporate** | ✅ |
| 2017–2023 | Products carry the **Micro Focus** brand; versioning migrates to **year.month** (2020.08, 2022.05, 2023.05) | Naming/versioning change | Vendor release communications | ✅ |
| **25 Aug 2022** | **OpenText announces** agreement to acquire Micro Focus | Recommended all-cash offer at **532 pence per share**; enterprise value **~$6.0bn** fully diluted; total purchase price **$6.0bn** incl. cash and debt; 2.2× pro-forma TTM revenue; funded by $4.6bn new debt + $1.3bn cash + $600m revolver | OpenText press release 2022-08-25; also PR Newswire — **primary corporate** | ✅ |
| **31 Jan 2023** | **OpenText closes** the acquisition | Closing total purchase price **~$5.8bn** incl. cash and debt ("subject to final adjustments"); 2.3× TTM revenue; ~**8% workforce reduction** expected | OpenText press release 2023-01-31 — **primary corporate** | ✅ |
| 2023–2026 | Rebrand to **OpenText**; docs remain at `docs.microfocus.com`; product page redirects to the AI Operations Management URL; support portal still `portal.microfocus.com` / `my.opentext.com` | Naming/URL migration | Multiple vendor properties; archive snapshot 23 Dec 2024 | ✅ |

### 2.2 The "$5.80" claim — corrected
The brief's hypothesis for this pass was "**~US$6.0bn / US$5.80 per share**". What the primary announcements say: the **per-share offer was 532 pence** ✅; the **announcement-day enterprise/purchase value was ~$6.0 billion** fully diluted ✅; the **closing purchase price was ~$5.8 billion** inclusive of cash and debt ✅. A "$5.80" figure does exist — as the **$5.8 billion closing consideration**, not a per-share price. **A "$5.80 per share" characterisation is rejected as stated.**

### 2.3 The spin-merge structure — what the sources support
The brief asked to verify a **50.1% / 50.1%** split, which is arithmetically impossible (it sums to 100.2%) and should be treated as a transcription error. What is established: HPE Software was **spun off and immediately merged** with Micro Focus, completed **1 September 2017**, consistently described as a **~$8.8 billion** transaction ✅, which "delivered approximately $8.8 billion to HPE and its stockholders" ✅ — the correct framing being that HPE *received value* and Micro Focus *issued equity*. The commonly reported structure is that **HPE stockholders received ~50.1% of the combined company** with Micro Focus shareholders retaining the remainder ⚠ — the standard description in deal reporting, but **not re-verified at a primary corporate document this pass**.

### 2.4 What the ownership chain means for a customer
| Consequence | What actually happens | Mitigation |
|---|---|---|
| **Branding churn** | The same product answers to HP, HPE, Micro Focus and OpenText names depending on when a component was installed or documented; skews appear in inventories, contracts, CMDB CI names and runbooks | Maintain an internal alias map (§1.4 is a template); tag CIs with the *current* vendor product name |
| **Support and licensing continuity** | Support continued across both transitions, but contract paper, renewal SKUs and portal URLs changed | Verify which legal entity is named on the contract before renewing |
| **EOL risk concentrated by acquisition** | Acquisitions force portfolio rationalisation. **OBR is the worked example** — made obsolete *before* the OpenText acquisition, because OPTIC DL superseded it (§3.4) | Treat every "co-sold in a suite" component as an obsolescence candidate at each ownership change; the OBR notice is the template for how it will look |
| **Documentation and part-number churn** | Docs live at `docs.microfocus.com` (still returning OpenText chrome); the corporate product page moved to a new URL and name — `/products/operations-bridge` **301-redirects** ✅ (snapshot 23 Dec 2024). Brochures carry both legacy Micro Focus and new OpenText numbering (`260-000148-004 | O | 01/24`) | Bookmark doc-set roots, not deep links; record the doc revision and date on any evidence taken from a brochure — these PDFs are revised in place |
| **The 8% workforce reduction** | Announced with the closing as the mechanism to "balance the combined company" ✅ | When assessing roadmap confidence, ask which product teams were touched — a legitimate diligence question vendors answer in the room |
| **Integration-driven simplification** | OpenText's own SEC filings reference a **Business Optimization Plan** expanded April 2025 to complete "strategic initiatives, integration and simplification following the Micro Focus acquisition" ✅ (8-K, 2025-04-29) | Read "simplification" as portfolio-pruning risk; pair with [technology_lifecycle_management_guide.md](technology_lifecycle_management_guide.md) |

### 2.5 The documentation/URL migration map
| Era | Documentation host | Support portal | Product-page path | Doc-chrome brand |
|---|---|---|---|---|
| HP / HPE | HP support and software pages | HP Support Center | HP Software pages | HP / HPE |
| Micro Focus | `docs.microfocus.com` | `portal.microfocus.com`, `support.microfocus.com` | `microfocus.com/OpsBridge` | Micro Focus |
| OpenText (today) | **`docs.microfocus.com` still** — returning **OpenText Documentation Portal** chrome and a 2026 OpenText copyright ✅ | `portal.microfocus.com`, `my.opentext.com/support/...` ✅ | **`opentext.com/products/ai-operations-management`** ✅ | OpenText, with a standing legal note that pre-2023 material may carry HP/HPE/Micro Focus branding and that **"As of January 31, 2023, the Material is now offered by OpenText"** ✅ |

That last row is worth internalising: the vendor's own community footer states that as of 31 January 2023 the material is offered by OpenText and that HP/HPE/Micro Focus marks are historical ✅ — the vendor's own confirmation of the date this guide verified from the press releases.

---
## 3. The product family, member by member
**This section is the most perishable in the guide.** Every status claim carries the date and source it rests on, and every place status could not be established says so explicitly. The dispatcher's hypotheses for this pass were that **BVD and OBR are the most likely casualties** and that **RTSM stopped being a separately-sold product after OMi 10** — both are tested below, and one is partly rejected.

### 3.1 The family at a glance
| Component | Role | Status (as established this pass) | Evidence |
|---|---|---|---|
| **Operations Bridge Manager (OBM)** | Event and service-health console; the core | **Current** ✅ — 24.4 released 6 Nov 2024; 24.2 "Classic Edition" release blog; doc sets to 23.4/24.1/24.4 and a 25.4 correlation page | Support release notice KM000036119; `doc/386/25.4/conttbec` |
| **Operations Agent (OA)** | Host agent for metrics/events/actions | **Current** ✅ — versions 12.14/12.15/12.25/12.26 evidenced | Support notice KM000023404; `doc/Operations_Agent/12.26/Home` |
| **SiteScope** | Agentless monitoring | **Current** ✅ — **renamed "Operations Bridge Express (SiteScope)"**; 25.4 released 28 Oct 2025 | Support release notice KM000043535 |
| **Operations Bridge Reporter (OBR/SHR)** | Historical Vertica-based reporting | **OBSOLESCENT / END OF SALE** ✅ — end of sale 1 Oct 2021; last download 31 Oct 2021; extended support to **31 Dec 2026** for 10.50 | Micro Focus OBR obsolescence support notification (© 2021) |
| **Business Value Dashboard (BVD)** | Flexible business/KPI dashboarding | **Hypothesis rejected (so far)** — **no EOL found**; named as an active surface in a **June 2023** data sheet and in **Data Protector 24.4** docs ⚠ | `260-000252-001 | 06/23`; `doc/200/24.4/bvd` |
| **RTSM** | CI/relationship model embedded in OBM | **Current but embedded** ✅ — documented as living inside OMi from the 10.x era; **not** a standalone line item in modern Edition matrices | OMi 10.01 RTSM Administration; archived Dec 2024 edition matrix |
| **Universal Discovery and CMDB (UD/CMDB)** | Discovery + CMDB populating topology | **Current** ✅ — "OpenText Universal Discovery and CMDB **26.1**" | Support portal KM000044509 (listing) |
| **Operations Orchestration (OO)** | Runbook automation / orchestration | **Current** ✅ — 2023.05 and 24.1/24.1.1 documented | `doc/Operations_Orchestration/2023.05/ReleaseNotes`, `/24.1.1/ReleaseLog` |
| **Container Deployment Foundation (CDF)** | Containerised delivery foundation | **Status NOT established** ⚠ — the capability survives as "containerized operations bridge" (Premium/Ultimate) and "Container Edition" branding | Archived Dec 2024 edition matrix; "Operations Bridge CE 24.2" blog |
| **OPTIC Data Lake / platform** | Containerised common data store and analytics foundation | **Current** ✅ — the named successor to OBR's reporting role, entitled from OpsB 2020.08 | OBR obsolescence notice (© 2021) |
| **Operations Bridge Analytics** | Analytics add-on tier | **Current** ✅ as an entitlement; ⚠ feature contents unverified | Archived Dec 2024 edition matrix |

### 3.2 Operations Bridge Manager (OBM) — the console
**Role.** The vendor's own documentation describes OBM as automatically discovering and monitoring infrastructure by collecting topology and event data, using event data to resolve infrastructure or service issues, using topology data to check health and performance, and providing "a single pane of glass to detect, correlate…" ✅ (`Operations_Bridge_Manager/23.4` and `/latest` doc-set home text).

**Currency.** The most recent OBM release captured at a primary vendor source this pass is **24.4, released 6 November 2024** ✅. A **24.2 "Classic Edition"** release is documented in the vendor community, and the post splits its notes into **Classic** and **Container** products ✅. Doc sets observed: `2023.05`, `23.4`, `24.1`, `24.4`, plus a correlation page under `doc/386/25.4/` ✅.

**The version-scheme answer.** The evidence supports a **year.month → year.quarter** migration: 2023.05 and 23.4 both appear; 24.x uses the shortened `YY.Q` form; by 2025 release notices read **25.4**. So the suite releases on a **quarterly `YY.Q` cadence**, which is why you see `YY.1`, `YY.2`, `YY.4` rather than a month stamp. ⚠ The *existence and date* of the "What's New in OpenText AI Operations Management 25.4" blog (30 Oct 2025) is verified; its body could not be extracted (scraper error), so 25.4's specifics are **not** asserted here.

### 3.3 The Operations Agent (OA) and agentless collection
**Agent-based.** The Operations Agent is the host-installed collector — metrics, events, local action execution, feeding OBM. It is an entitlement in the higher editions ✅ (archived edition matrix, Dec 2024).

**Agentless.** Two distinct paths, frequently confused: (1) **SiteScope / Operations Bridge Express** — agentless monitoring via remote protocols (WMI, SSH, SNMP, JMX, HTTP, database clients), a **separate product with its own release train** ✅ (23.05, 2023.05, and Operations Bridge Express (SiteScope) 25.4 on 28 Oct 2025); (2) **Operations Connectors** — adapters pulling events/metrics/topology from *other* monitoring tools (§9.3).

**Versions established:** 12.14, 12.15, 12.25 (released; support notice) and a 12.26 doc set ✅. **Not established ⚠:** whether the agent has moved to a "2x.xx" line — the brief's hypothesis of "12.x then 2x.xx" was **neither confirmed nor refuted**. Do not put an unverified version in a business case.

### 3.4 Operations Bridge Reporter (OBR) — the confirmed casualty
**This is the cleanest sunset finding in the guide, and it is at a primary vendor source.** Micro Focus issued a formal **product obsolescence support notification** stating: *"Since summer 2020, Micro Focus has provided an alternative reporting solution built on the OPTIC Data Lake. Therefore, Micro Focus is announcing product obsolescence of Operations Bridge Reporter (OBR)"* ✅.

| Date | Programme activity |
|---|---|
| 30 Jun 2021 | Customer notification |
| **1 Oct 2021** | **End of Sale** |
| **31 Oct 2021** | **Last day of download availability** — customers told to download and retain every version they might need, because the vendor could no longer distribute them |
| 31 Dec 2022 | OBR 10.50 Committed Support ends |
| **31 Dec 2026** | **OBR 10.50 Extended Support ends** |
| 31 Dec 2030 | OBR 10.50 Self-Help Support ends |

By version: **10.50** committed 31 Dec 2022, extended **31 Dec 2026**, self-help 31 Dec 2030; **10.4x** committed 31 Dec 2022, extended **31 Dec 2024**, self-help 31 Dec 2028; **10.3x** committed 31 Mar 2022, extended **31 Mar 2024**, self-help 31 Mar 2028 ✅. The notice names the suites that previously used OBR as their reporting solution — **Operations Bridge (OpsB), Network Operations Management (NOM) and Data Center Automation (DCA)** ✅ — and states all three "now use OPTIC reporting".

**Why it was retired, in the vendor's words:** the OPTIC Data Lake allowed cross-domain data on a single report, direct access to the data lake (powered by **Vertica**) with a documented schema, the customer's choice of BI tool, Grafana-style graphing, and the customer's own machine learning on the stored data ✅ — a real architectural improvement, not just a repackaging: reporting moved from a *product* to a *platform with a schema*.

**What this means in 2026.** If OBR is still in the estate, its **extended support cliff is 31 December 2026** — inside or immediately adjacent to the planning horizon of anyone reading this. That is a hard, dated, vendor-published deadline and it belongs at the top of any renewal assessment (§12). The hypothesis that OBR was superseded by "Operations Bridge Analytics" is **partly wrong**: it was superseded by **OPTIC reporting on the OPTIC Data Lake** (the platform); Operations Bridge Analytics is a separate entitlement on the same data ✅.

### 3.5 Business Value Dashboard (BVD) — hypothesis tested and rejected
**Disposition: an explicit negative finding.** Nothing found this pass supports BVD being retired: a **June 2023** data sheet names BVD as an active surface in the OBR→OPTIC migration ("Provide Business Value Dashboard (BVD) access to OBR and OPTIC DL"; "Design new custom reports in BVD or an external BI tool") ✅; BVD is documented as an integration in the **Data Protector 24.4** doc set, described as a way to "create custom, flexible dashboards that visualize information from OMi and other sources" ✅; an **ITOM Marketplace** entry exists; and vendor collateral positions BVD as an extract-and-dashboard target ✅.

**Therefore ⚠: BVD is not established as retired.** What *is* established is that its strategic centre has moved — the forward-looking narrative routes dashboards through **OPTIC Reporting / OPTIC DL** and third-party BI (Tableau, Qlik Sense, Power BI). The defensible reading is **"existing, still-integrated, not the strategic direction"**, not "end-of-life". Put this to the vendor in writing; this guide will not assert a sunset the evidence does not support.

### 3.6 RTSM and the Universal CMDB foundation — hypothesis partly upheld
The hypothesis was that **RTSM stopped being separately sold after OMi 10**. The evidence supports a qualified version: HP's own OMi 10.01 RTSM Administration guide states *"The RTSM contains all the configuration items (CIs) and relationships created in HP Operations Manager i, whether created automatically from incoming data collector data or the DFM process, or created manually using IT Universe Manager"* ✅ — the model living *inside* the manager, always. OMi 10.00 release notes describe switching **HP Operations Agent management to the OMi server** — consolidation of previously separate functions into the new console ✅. And the modern edition matrices **do not list RTSM as a line item**; they list **"OpenText Universal Discovery and CMDB"** as the discovery/CMDB entitlement ✅.

**Verdict: upheld in substance** ✅. RTSM is not a standalone SKU in the current portfolio; it is the *embedded model* in OBM, while **Universal Discovery and CMDB** is the separately-sold discovery product that populates it. That distinction matters enormously for §5: **the model comes with the console; population of that model is a separate product and a separate project.** ⚠ The exact commercial packaging transition date was not established.

### 3.7 Operations Orchestration (OO)
OO is the runbook-automation and orchestration engine in the same portfolio — remediation workflows, complex conditional execution with rollback — and it is **current**: the documented line runs through **2023.05** (which added a "Python Executor" micro service) and **24.1 / 24.1.1** (24.1 adding beta support for editing AFL workflows in the Workflow Designer) ✅. It has a live community group with a substantial blog archive ✅. The vendor's automation narrative increasingly also points at **Automation Center** alongside OO ✅. Note the runbook-automation training module teaches installing the **Operations Agent** and creating **OO flows** for OBM ✅ — i.e. the OO-to-OBM remediation path is the documented one.

### 3.8 Container Deployment Foundation (CDF) and containerised delivery
**Established:** the vendor markets a **"containerized operations bridge"** entitlement (Premium and Ultimate, Dec 2024 matrix) ✅; a release blog exists for **"Operations Bridge CE 24.2"** where **CE = Container Edition**, splitting Classic from Container products ✅; the product page describes SaaS, hybrid, on-premises or fully air-gapped AI ✅; and the **OPTIC Data Lake is described as "a containerized common data collection and storage"** layer ✅ — containerisation is now the platform's native delivery model, not an add-on.

**NOT established ⚠:** whether **CDF still ships as a named, versioned product** in the 2025/2026 era, and what its version is — including the specific version that accompanied OBM 2020.x/2023.05, which the brief asked for and this pass did not capture. The honest reading: CDF was the *container delivery foundation* of the 2019–2021 era, and the capability has since been absorbed into the platform's container-native design and Container Edition branding. **Do not cite a CDF version number.** Ask the vendor: *"what is the current Kubernetes delivery model for OBM, and is CDF still a supported component?"*

### 3.9 What the vendor markets today, in its own framing
| Dimension | Vendor framing (as published) | Reading |
|---|---|---|
| **Product name** | "OpenText AI Operations Management (Operations Bridge)" — the archived Dec 2024 page names it exactly so | Suite name survives as alias; marketing leads with AI ✅ |
| **Category** | "autonomous AIOps platform"; "enterprise event and performance management software" | Category-marketing vocabulary — *claims* |
| **Editions (Dec 2024)** | **Express** (off-cloud), **Premium** (private cloud + managed service), **Ultimate** (adds OpsB Analytics, Automation, UD/CMDB) | Ultimate is the full-stack tier ✅ |
| **Editions (current page)** | **Express — Private cloud** and **Premium — Private or Public cloud**, with add-ons for Integrations, Event and metric analytics, GenAI incident management, Observability, Automation | The structure has moved again; "Ultimate" is not visible on the current page ⚠ |
| **Deployment** | SaaS, hybrid, on-premises, private cloud, managed service, air-gapped AI | Genuinely broad — a real differentiator for regulated buyers ✅ |
| **Current release train** | 25.4 (blog 30 Oct 2025); 24.4 and 24.2 documented; quarterly `YY.Q` | **Actively releasing** ✅ |
| **Named AI capabilities** | Anomaly detection with learned baselines; AI correlation claimed to cut event volume "by 30-95%"; a "diagnostic AI agent" with traceable reasoning and a confidence score; **Aviator** for GenAI event remediation | *Claims, vendor-sourced* — see §7 |
| **Named marketing numbers** | "94% improved service availability", "49% fewer service outages", "1000+ out-of-the-box integrations", "30+ technologies" | *Claims, vendor-sourced*, tied to customer stories and a client survey |
| **Current marketing name** | **OpenText AI Operations Management** — a 301 redirect from `/products/operations-bridge` to `/products/ai-operations-management` was confirmed in a 23 December 2024 archive snapshot ✅; community release-readiness collateral has also labelled the family **"AIOps and Observability (fka Operations Bridge)"** ⚠ (search-result snippet only — the page sits behind a WAF) | Expect **three name strata** in one estate: the release-notice product names (`Operations Bridge Manager`, `Operations Bridge Express (SiteScope)`), the corporate product-page name, and the community family label |
| **The problem class** | A single correlated event stream over a heterogeneous estate — a *federating* layer that is not the source of truth for any single domain but is the source of truth for "what is wrong with my services right now" | — |

**Cross-ref:** the edition/entitlement structure is a *commercial* artefact; for the lifecycle method used to grade "current / maintenance / sunset", see [technology_lifecycle_management_guide.md](technology_lifecycle_management_guide.md).

---

## 4. Architecture and the event pipeline
**Boundary note:** this section documents *what the product's own architecture is*. Observability theory — traces, spans, metrics cardinality, SLOs, error budgets — belongs to [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) and [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md); the process that consumes the output belongs to [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md).

### 4.1 The component topology, as documented
| Component | Documented role |
|---|---|
| **Gateway** | First processing stage — receives events from data collectors and agent infrastructure |
| **Data Processing** | Second stage — applies the pipeline steps that classify, resolve, reduce, correlate and enrich |
| **RTSM** | The embedded CI/relationship model; the source of the class model and target of CI resolution |
| **Operations Agent (OA)** | On-node collector and local action executor; forwards to the OBM server |
| **Data collectors** | The per-domain collection layer supplying event "hints" |
| **OPTIC Data Lake** | Containerised, Vertica-powered store receiving event/metric/topology streams |
| **Performance Dashboard** | The console surface exposing *event pipeline statistics* — the artefact you use when the pipeline itself is the problem |

✅ *Source: the vendor-authorised **OBM-EPAC** course outline (Operations Bridge Manager Event Processing, Automation and Correlation, delivered for OBM 24.2 by an OpenText training partner, fetched 2026-09-17), read with the OBM 23.4/24.4 doc-set URLs and the OBM 25.4 correlation page (`doc/386/25.4/conttbec`).* Training collateral is **secondary** for architecture — but it is the vendor's own syllabus naming the components the primary documentation then describes, so it is a reliable map and an unreliable source for fine specifics.

### 4.2 Ingestion paths
| Path | Mechanism | Typical bank use |
|---|---|---|
| **Operations Agent** | Installed on the host; metrics, events, local actions | Core banking Linux/UNIX/Windows servers, middleware — the highest-fidelity path |
| **SiteScope / Operations Bridge Express** | Agentless: WMI, SSH, SNMP, JMX, HTTP, database clients; **no agent on the target** | Appliances, network devices, database servers, and the "monitor it before anyone owns it" case |
| **SNMP traps** | Generic SNMP ingestion, normalised by domain packs | Network gear, storage arrays, mainframe-adjacent appliances |
| **Web services / REST** | API ingestion into OBM and OPTIC DL — "**Open Data Ingestion**" is named as part of Operations Bridge | Modern apps, SaaS sources, in-house tooling, and the escape hatch |
| **Log / stream** | Custom-data ingestion including **Kafka** | Application logs, event hubs, third-party feeds |
| **Operations Connectors** | Adapters pulling events, metrics, topology and generic output from third-party tools; in many cases **events synchronise back to the source** | ServiceNow, BMC, Dynatrace, Datadog, Splunk, Zabbix, Nagios, SolarWinds, Jira and others |
| **Management Packs** | Domain-specific collections, policies, visualisations; selected packs add **TBEC rules** | Databases, middleware, SAP, cloud, containers, virtualisation |
| **"Manager of managers"** | The deliberate pattern of leaving domain tools in place and federating them | **The realistic bank pattern** — nobody retires the network tool to adopt the console |

✅ *Sources: the **Full-Stack AIOps: Operations Bridge Integrations** brochure (`263-000031-003 | O | 01/24`) and the **Operations Bridge – SaaS** data sheet (`260-000148-004 | O | 01/24`), fetched 2026-09-17.* Both are **vendor collateral** — excellent for capability, naming and inventory, unreliable for performance claims.

### 4.3 The event lifecycle, step by step
The single most useful list in this section. If you understand only one part of Operations Bridge, understand this.

| # | Stage | What happens | Why it matters |
|---|---|---|---|
| 1 | **Ingestion** | The collector or connector hands a raw event to the gateway | Source-normalisation quality is decided here |
| 2 | **CI resolution** | The event is mapped to a **configuration item** in the RTSM using **hints** from the source | **This is where correlation is won or lost.** An unresolved event can never correlate topologically (§5) |
| 3 | **ETI resolution** | The event is mapped to an **Event Type Indicator** — type-level classification beyond text | ETIs let grouping, dedup and correlation operate on types, not strings |
| 4 | **Duplicate event suppression** | Identical repeats folded | First defence against the storm |
| 5 | **Event suppression** | Configured conditions suppressed (maintenance, known conditions, parent-child) | Must be governed — suppression is where a bank accidentally hides an outage (§11) |
| 6 | **Event storm suppression** | A dedicated feature to dampen storms rather than suppress individual events | The one people forget to tune, and the one that saves the queue during a cascade |
| 7 | **Close related events** | Related events closed together when the cause clears | Prevents stale red and phantom green |
| 8 | **Stream-based correlation (SBEC)** | Correlation over the event stream — patterns, sequences, counts in a window | No topology required; useful where the model is weak |
| 9 | **Topology-based correlation (TBEC)** | Correlation over the **relationship graph** — symptoms suppressed, cause surfaced | Strongest mode; wholly dependent on the CMDB |
| 10 | **Automated event correlation (AEC)** | ML correlation against **OPTIC DL**, documented for both **Classic** and **Containerized** OBM | The AIOps layer (§7) |
| 11 | **Time-based event automation (TBEA)** | Scheduled/condition-based automation of event handling | Where "auto-close the noisy nightly job" lives |
| 12 | **Automation, forwarding, custom actions, EPI** | Forwarding to ITSM and other systems; custom actions; **Event Processing Interface** scripting for arbitrary pipeline customisation | The integration and escape-hatch layer |
| 13 | **Health indication** | Events roll up into **Health Indicators (HIs)** and **KPIs** on the service model, computed via the **KPI Enrichment Service (KES)** and the **Multi-process Architecture Business Logic Engine (MARBLE)** | What turns an event stream into a *service* view — the operator-facing output |

✅ *Stages 1–13: the vendor's own OBM-EPAC course outline (OBM 24.2).* **Flag:** the KES and MARBLE expansions are as given in that outline; the corresponding documentation pages were not extracted this pass, so treat their internal mechanics as ⚠ and the existence and naming of the features as ✅.

### 4.4 The correlation model
| Mode | Basis | Strength | Failure mode |
|---|---|---|---|
| **SBEC** — stream-based | Event-stream patterns over time | Works with weak or absent topology; good for flapping and repeated-failure patterns | Blind to causal structure; cannot tell you *why* |
| **TBEC** — topology-based | The RTSM relationship graph; the "**Correlation Manager**" handles rule creation, cross-domain automation and manual event relation | Correctly suppresses symptoms and surfaces the true cause; the reason to own an ITOM platform at all | **Completely dependent on CI resolution and a maintained model.** Bad CMDB ⇒ silent no-op |
| **AEC** — automated/ML | Machine learning over the OPTIC Data Lake | Catches clusters nobody wrote a rule for; needs no rules | Requires data volume and quality; results must be explainable to an operator and an auditor |

✅ *Terminology and the Correlation Manager: OBM-EPAC outline (OBM 24.2) and the OBM 25.4 correlation page. **AEC** is documented in the same outline as operating with OPTIC DL, with an explicit configuration path for **Classic** and **Containerized** OBM and its own UI, architecture and troubleshooting content* — a meaningful finding: **AEC is a distinct, configurable capability, not a slide.**

### 4.5 The operator workflow and who watches the watchmen
| Surface / concern | Purpose |
|---|---|
| **Event browser** | The queue — filter, sort, acknowledge, annotate, assign, close |
| **Service health (HIs / KPIs)** | Roll-up view: which business services are degraded, and why |
| **Dashboards** | Operational and business dashboards, including the **Performance Dashboard** for platform self-visibility |
| **Topology views / IT Universe Manager** | The model itself, navigable |
| **Runbook automation / custom actions** | Invoke remediation flows (documented alongside **OO** content packs) |
| **Annotations, downtime/MAO windows, tools** | Operator notes; planned-work windows so scheduled change does not become an incident; operator-invokable admin tools with their own authorisation |
| **Server-side HA** | The deployment is explicitly split **Classic** vs **Containerised**, the containerised mode being the platform-native direction ✅; multi-node gateway/processing redundancy is standard for large estates ⚠ (exact HA topology docs not re-extracted) |
| **Pipeline self-monitoring** | The **Performance Dashboard** exposes event-pipeline statistics — the pipeline instruments itself ✅ |
| **The genuine risk** | **A monitoring platform that fails silently is worse than no monitoring platform.** Every deployment needs an out-of-band check (synthetic transaction or heartbeat) that does *not* depend on the same pipeline |

⚠ **Not established:** published RTO/RPO figures for OBM server HA, and any vendor-published maximum event throughput per OBM instance — see §9.6.

---

## 5. Topology and the CMDB dependency
### 5.1 What the RTSM is
The **Run-Time Service Model (RTSM)** is the model of **configuration items (CIs)** and their **relationships** inside OBM. HP's own OMi 10.01 RTSM Administration documentation states it plainly: *"The RTSM contains all the configuration items (CIs) and relationships created in HP Operations Manager i, whether created automatically from incoming data collector data or the DFM process, or created manually using IT Universe Manager."* ✅ Three properties follow, and each is an inherited design decision:

1. **It is populated from multiple directions** — automatically from data-collector data and the **DFM process**, and **manually** via IT Universe Manager. Manual population is a first-class path, so model quality is partly a *policy* question, not only a tooling one.
2. **It is a run-time model, not an asset register.** It is the operational model used for correlation and health — not the bank's asset inventory or service catalogue, though they overlap and must be reconciled.
3. **It organises CIs by class model** — the schema of CI types and permissible relationships. Getting the class model right is the difference between a model that correlates and one that merely stores.

### 5.2 The working vocabulary
| Concept | Meaning | Why it matters |
|---|---|---|
| **CI** | A configuration item — server, database, service, application, cluster, interface | The unit of association for an event |
| **Relationship** | A typed link (runs-on, depends-on, contains, connected-to) | The edge along which a fault propagates — the graph TBEC traverses |
| **Class model** | The schema: which CI types exist and which relationships are legal | A weak class model produces a graph too sparse to correlate over |
| **IT Universe Manager** | The RTSM's administrative/navigation UI | Where you inspect and repair the model |
| **DFM process** | The dynamic/automatic modelling process deriving relationships from collected data | The "nobody typed it in" half of the model |

### 5.3 How topology drives event-to-service correlation
> An event arrives with a hint about its CI (**CI resolution**). If resolution succeeds, the event attaches to a node in the model. When many events arrive across *topologically connected* nodes, **TBEC** infers that events on dependent nodes are symptoms and the one nearest the graph root is the cause — suppressing symptoms and promoting the cause. Health then rolls up: CI health feeds **health indicators**, which feed **KPIs** on the service objects, which is how "the payments switch" acquires a colour instead of 400 rows of red text.

Every arrow in that paragraph depends on two preconditions: **the event resolved to a CI**, and **the graph contains the relationship the fault travelled along.** Neither is automatic.

### 5.4 The honest failure mode
**Correlation is only as good as a CMDB nobody maintains.** This is the predictable failure mode of the entire product category, and the most common reason an Operations Bridge deployment underdelivers:

| Failure | Mechanism | Symptom the bank sees |
|---|---|---|
| **Unresolved CIs** | New hosts/apps stood up and never modelled; CI resolution fails and the event arrives unattached | Those events never correlate — and they are usually the *newest*, most business-critical systems |
| **Stale relationships** | A dependency re-pointed in reality but not in the model (DR flip, re-platform, load-balancer change) | TBEC suppresses the *wrong* event or nothing; operators stop trusting the rollup |
| **Sparse graph** | The model knows servers but not application-to-server and server-to-server paths | No causal path to traverse; TBEC degrades silently to no-op |
| **Class-model drift** | Two domains model the same thing as different CI classes | Cross-domain correlation never fires — precisely the case the platform was bought for |
| **The maintenance tax** | Model upkeep is nobody's measured objective; discovery is refreshed on a project, not a lifecycle | Quality decays between projects; the platform's value decays with it |
| **The trust collapse** | After two wrong suppressions, operators open the native tool instead | The platform becomes a licence cost with a dashboard nobody opens |

**The engineering conclusion:** in this product class, **topology quality is not a data-hygiene footnote — it is the product's dependency.** Budget ongoing model maintenance as part of the platform's run cost, not as a one-off discovery project. This is the same structural insight the AIOps-category guide reaches about RCA quality ([ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md) §5).

### 5.5 CI population paths — where the model comes from
| Path | Product / mechanism | Notes |
|---|---|---|
| **Agentless discovery** | **Universal Discovery and CMDB (UD/CMDB)** — the separately-sold discovery and CMDB product | The primary bulk-population path; actively releasing ("Universal Discovery and CMDB **26.1**") ✅ |
| **Agent-reported topology** | Operations Agent data feeding the model | Fast, but only sees what has an agent |
| **Data-collector data / DFM** | Automatic modelling inside OMi/OBM | Explicitly named in HP's RTSM documentation ✅ |
| **Manual population** | **IT Universe Manager** | The repair path — and the admission that automation never covers everything ✅ |
| **Management packs** | Selected packs add domain topology and TBEC rules | Deep but narrow — good per-domain, weak across domains ✅ |
| **Third-party / ITSM** | Connectors supplying topology from other tools (ServiceNow CMDB, BMC and others in the vendor inventory) | The pragmatic path where two CMDBs must coexist ✅ |
| **The CMDB discipline itself** | Ownership, CI classes, data quality, CMDB-vs-asset-register | **Cross-ref, do not re-derive:** [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md); for population-pipeline data quality, [data/dataops_guide.md](data/dataops_guide.md) |

### 5.6 The data-quality prerequisites, as a checklist
Before blaming the correlation engine, check the model:

- [ ] **Coverage** — what share of monitored nodes resolve to a CI? (The unresolved-event rate is the metric.)
- [ ] **Relationship density** — for the services that matter, does a causal path actually exist?
- [ ] **Freshness** — how old is the discovery data, and what triggers a refresh?
- [ ] **Cross-domain consistency** — does a payment gateway exist once, or three times under three class names?
- [ ] **Ownership** — who is accountable for model quality, and is it in anyone's objectives?
- [ ] **Reconciliation** — how does the RTSM reconcile against UD/CMDB, the service catalogue and the ITSM CMDB?
- [ ] **Proof** — can you demonstrate a real incident where TBEC correctly suppressed symptoms? If not, the capability is unproven in your estate regardless of the licence.

**Cross-ref:** the CMDB as a *data-model and data-quality subject* belongs with the data-architecture and service-management content; this section owns only the RTSM's role in the event pipeline.

---

## 6. Event correlation and the noise problem
**Written to be useful to a reader who will never deploy Operations Bridge.** The engineering problem is vendor-neutral; the product is one answer to it. If you take one section into a design review at any bank with any tooling, take this one.

### 6.1 The problem, stated properly
An estate generates events from dozens of independent instruments. Each is locally sensible and globally useless, for four structural reasons:

| Reason | Mechanism | Consequence |
|---|---|---|
| **Fan-out** | One failed dependency alarms every dependent component — a switch failure alarms 60 services | The *count* grows with blast radius, exactly backwards from urgency |
| **Timing skew** | Instruments poll on different intervals; the causal event may arrive *after* its symptoms | Naive time-ordering points at the wrong culprit |
| **Vocabulary divergence** | Every tool has its own severity scale, notion of "critical", and event-text grammar | Nothing can be compared or grouped without normalisation |
| **Change blindness** | A planned change, deployment or batch job produces the same signals as an unplanned fault | Planned noise drowns unplanned signal; suppression rules created to fix this hide real faults if ungoverned |

The queue is the binding constraint, and it is not a metaphor: **a human can work N events per hour.** If the estate produces 10N, the operator triages by whatever is at the top of the list — a sort order, not a priority. Every technique below exists to raise the ratio of actionable events to arriving events.

### 6.2 The four reduction techniques, and what each one buys
| Technique | What it does | What it buys | What it costs |
|---|---|---|---|
| **Deduplication** | Folds repeated identical events from the same source | Immediate, large, safe volume reduction | Almost none — usually the first and cheapest win. Fails if uniqueness is injected into the event text (timestamps, request IDs), so normalise before dedup |
| **Suppression** | Drops events matching a condition (maintenance, known-issue pattern, parent-child) | Removes planned and known noise | **The dangerous one.** An over-broad rule is a silent outage-hider; requires change control, expiry dates and an audit trail (§11) |
| **Aggregation / grouping** | Combines related events into one work item with a count | Converts 60 rows into one row with a blast radius | Loses per-event detail unless drillable — so drillability is a requirement, not a nicety |
| **Correlation (causal)** | Uses structure (topology and/or learned patterns) to identify cause and demote symptoms | The only technique that improves *quality*, not just volume — the one that makes the queue **workable** rather than merely **shorter** | Requires a model (topology) or data volume and trust (ML). Hardest to implement and to prove |

### 6.3 Symptom versus cause
> **A symptom is an event whose cause is another event already in the queue.** A cause is an event with no such event.

Every correlation technique computes that relation without a human. It is only computable with *some* structure: a graph (topology), a sequence (stream), or a learned pattern (ML). **No structure, no correlation** — the honest reason "just turn on the AI" fails: the AI needs the same substrate the rules did. Three corollaries practitioners hold as rules of thumb:

1. **Correlation quality is bounded by association quality.** If events do not resolve to a shared identity (a CI, a service, a dependency), no algorithm can relate them. §4.3 stage 2 and §5.6 are therefore *the* investment.
2. **The best suppression is a good model.** Most symptom noise disappears when the graph is right; suppression rules are compensation for a bad graph and accumulate technical debt.
3. **Correlation that cannot be explained will not be trusted.** An operator must be able to answer "why was this the cause?" — a governance requirement in a bank (§11) and a usability requirement always. Note the vendor's current material markets *traceable reasoning* and a confidence score on the AI path ✅ (*claim, vendor-sourced*, §7).

### 6.4 How Operations Bridge addresses it
| Technique | Operations Bridge mechanism | Dependency |
|---|---|---|
| Normalisation | **ETI resolution** | Source integration quality |
| Association | **CI resolution** with source hints and a resolution cache | **RTSM model quality** |
| Dedup | **Duplicate Event Suppression** | Event uniqueness discipline |
| Suppression | **Event Suppression** + downtime/MAO windows | Governance |
| Storm control | **Event Storm Suppression** | Tuning |
| Aggregation | **Close Related Events**, grouping on ETIs | ETI design |
| Stream correlation | **SBEC** | Time-window tuning |
| Topological correlation | **TBEC** + **Correlation Manager**, cross-domain automation, manual relation | **CMDB** |
| ML correlation | **AEC** against **OPTIC DL** | Data volume, model governance |
| Enrichment | Management packs supplying domain topology and TBEC rules | Pack coverage for your domains |
| Workflow | **TBEA**, custom actions, **EPI** scripting, forwarding, CLIP to ITSM | Integration design |

### 6.5 The queue-workability test
A short, reusable diagnostic — run it before buying anything, including before renewing anything.

| Question | Bad answer | Good answer |
|---|---|---|
| **Volume** | "About 40,000 events a day" | "Peak 4,000/hour; steady-state 600/hour after reduction" |
| **Signal ratio** | "Most are duplicates" | "78% removed pre-queue; of the remainder ~90% actionable" — measured, not asserted |
| **Cause latency** | "Operators work it out" | "Median time from symptom-first-arrival to cause-identified is X minutes" |
| **Topology coverage** | "We have a CMDB" | "94% of monitored nodes resolve to a CI; 70% of the top-50 services have a complete dependency path" |
| **Suppression hygiene** | "Ops can suppress" | "Every rule has an owner, a justification and an expiry; the set is reviewed quarterly" |
| **Explainability** | "The AI decided" | "The correlation reason is visible on the event and operators can override and annotate" |
| **Self-monitoring / proof** | "It's monitored" / "It saves time" | "An out-of-band synthetic check proves the alerting path works" / "Here is an incident where the tool found the cause before the bridge did" |

**Cross-ref:** the AIOps-category framing of this same problem, including the industry's own numbers for anomaly detection and RCA, lives in [ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md) §4–§5. This section states the problem rather than re-deriving the category's literature.

---
## 7. AIOps and the analytics claims
### 7.1 First, the honest category statement
**"AIOps" is a marketing category as much as a technical one.** It was coined by analyst firms and has since been applied to everything from genuine ML correlation engines to dashboards with a trend line. Two consequences matter for procurement: (1) **a vendor claiming AIOps claims category membership, not a capability level** — the claim is unfalsifiable as stated, which is why it is used; (2) **the only useful test is architectural** — *does the product compute something over data that a threshold rule could not, and is the result explainable and governable?* Everything else is positioning. The category's provenance, vendor stacks and a rollout method are [ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md)'s subject; this section audits *this product's* specific claims.

### 7.2 What is documented and shipped
| Capability | Evidence | Source quality | Status |
|---|---|---|---|
| **ML event correlation (AEC)** | Named in the vendor's own training syllabus with its own architecture, Classic and Containerized configuration paths, UI and troubleshooting content | Vendor training outline (primary-adjacent) | ✅ **Shipped** — a real, configurable feature, not a slide |
| **Topologically-aware correlation** | TBEC + stream + ML correlation described as three complementary modes | Vendor brochure (01/24) + training outline | ✅ |
| **Anomaly detection with learned baselines** | Current product page: flags creeping latency, memory leaks and unusual volume "before a threshold trips" | Vendor marketing | ✅ as a product claim; ⚠ as an outcome |
| **BPM / real user monitoring** | Entitlements in the Premium/Ultimate edition matrices | Archived product page (Dec 2024) | ✅ entitlement exists |
| **OPTIC Data Lake analytics** | Containerised, Vertica-powered store; documented schema; customer's choice of BI, Grafana or own ML on the stored data | OBR obsolescence notice (© 2021) + brochure | ✅ |
| **Operations Bridge Analytics** | A named entitlement (Ultimate tier, Dec 2024 matrix) | Archived product page | ✅ entitlement exists; ⚠ feature contents not extracted |
| **GenAI incident management / Aviator** | Current page lists "GenAI incident management" as an add-on; community blog "Unlock the power of AI with Aviator — GenAI-powered event remediation" dated 17 Nov 2025 | Vendor marketing + community | ✅ as announced/shipped; ⚠ feature depth unverified |
| **Diagnostic AI agent; automated remediation** | Current page: agentic AI reasoning across topology, events and changes with "a confidence score and traceable reasoning"; "thousands of predefined remediation tasks under policy guardrails, role-based access, approvals, and a full audit trail" | Vendor marketing | ⚠ *claims, vendor-sourced* |

### 7.3 The vendor's own numbers, labelled
Reproduced **only as claims**, with the vendor's own attribution:

| Claim | Vendor's stated basis | Our label |
|---|---|---|
| "AI-driven correlation cuts event volume by **30-95%**" | Product page assertion | ⚠ *claim, vendor-sourced*. **A range this wide is unfalsifiable** — it spans "modest" to "transformative" and is therefore evidence for neither. Demand a measured before/after on a comparable estate |
| "**94%** improved service availability" | Third-party client-survey site | ⚠ *claim, vendor-sourced*; the metric is undefined — "improved by 94%" is not a unit of availability |
| "**49%** fewer service outages" | A named telecom customer story | ⚠ *claim, vendor-sourced*, single customer, telecom not banking |
| "**1000+** out-of-the-box integrations" | Vendor marketplace link | ⚠ *claim, vendor-sourced*; the marketplace is real and large, but the count conflates connectors, packs, community and partner content |
| "**200+** technology integrations / domains" | Brochure + data sheet | ⚠ *claim, vendor-sourced* — though the underlying **inventory (§9.3) is verifiable and genuinely broad** |
| "**50+ patents**" for high-speed data analysis | Data sheet | ⚠ *claim, vendor-sourced*, unaudited |
| "Market Leader, Vendor Selection Matrix AIOps Platforms" (Research in Action) | Data sheet citation | ⚠ *claim, vendor-sourced*; analyst placement is paid-access material, not independently confirmed |
| Alarm reduction "over 70 percent" | One named telecom customer quotation | ⚠ *claim, vendor-sourced*, single customer |

**The one number worth repeating back to the vendor:** event reduction is bounded by *your* estate. Set acceptance criteria against your own measured baseline, in the contract — not against a brochure range.

### 7.4 What is not established, and the judgement
⚠ Not established: the **feature-level contents** of Operations Bridge Analytics as a distinct product; whether **AEC** requires, recommends or tolerates on-premises OPTIC DL versus cloud for data-residency constraints; any **independent (non-vendor)** measurement of event reduction or RCA accuracy for this product in a **banking** estate; and the **model-governance** story for AEC — how a bank evidences *why* the ML correlated as it did, for audit. That last question decides whether AIOps correlation is usable in a supervised firm, and the vendor addresses explainability as a feature claim rather than as a compliance artefact.

**Judgement:** the AIOps capability appears **genuine but second-mover**. It ships real ML correlation (AEC) on a real data-lake substrate (OPTIC DL), has real anomaly detection, and is attaching agentic and GenAI features (Aviator) to the same foundation. It is *not* where the market's AI depth leadership sits — that is the SaaS-observability cohort (§10) — and its differentiation remains **breadth of ingestion plus on-premise/air-gapped deployment**, not algorithmic superiority. Score it on the substrate (data lake, model, integration breadth, explainability, deployability), and treat the AI branding as the least durable part of the proposition — because if the AI is the reason, the AI is also the reason it becomes replaceable.

---

## 8. Mainframe, legacy and MFT coverage
**Central for a bank, and where vendor brochures mislead most.** A "supported" row in a support matrix can mean five different things: a first-party agent runs there; a first-party agentless probe reaches it; a partner-supplied connector covers it; a generic SNMP/REST path can be pointed at it; or it appears because a *different* product in the portfolio reaches it. This section separates those cases and is explicit where the evidence ran out.

### 8.1 The z/OS question — does an agent run there?
**Result: no evidence was found that the Operations Agent runs on z/OS.** What the vendor's own material shows instead is mainframe coverage via **partner-authored and third-party paths**:

| Evidence | What it shows | Source |
|---|---|---|
| **"IBM Mainframe zOS"** appears under **Partner Authored Connectors** | z/OS coverage is presented as a **partner-supplied connector**, not a first-party agent | Full-Stack AIOps integrations brochure (`263-000031-003 | O | 01/24`) ✅ |
| **"IBM AS/400", "IBM DB2", "IBM Mainframe"** in the *Partner* row of supported technologies | Same conclusion — partner-supplied, listed alongside IBM i (AS/400) and DB2 | Operations Bridge – SaaS data sheet (`260-000148-004 | O | 01/24`) ✅ |
| **IBM NetCool** and **IBM Tivoli (J9)** in the *Automation* row | IBM's own monitoring products are integration **peers** — the normal architecture is *forward from IBM tooling into OBM*, not replace it | Same data sheet ✅ |
| **BMC Impact Manager**, **BMC Remedy & Helix ITSM & TrueSight OM**, **CA Spectrum**, **SCOM**, **Zabbix**, **Nagios**, **Splunk**, **SolarWinds**, **Dynatrace**, **Datadog**, **New Relic** across the connector/integration tables | The platform's model is **manager-of-managers federation**: whatever already monitors the mainframe becomes a *source* | Same brochure ✅ |
| The vendor's own training syllabus documents Operations Agent installation in the runbook-automation module, with no z/OS content | The agent story in vendor training is distributed-host oriented | OBM-EPAC outline (OBM 24.2) ✅ |

**Established ✅:** the vendor's current integration inventory routes mainframe coverage through **partner-authored connectors** and **forwarding from IBM's and BMC's own monitoring products**; there is no first-party "Operations Agent for z/OS" in it. **Not established ⚠:** whether a **separate first-party z/OS collector product** existed historically under HPE/Micro Focus (several OpenView-era products were retired or divested at various points) — this pass could not confirm or refute a specific product name, version or support status, **and the guide will not invent one**; and the **current support status of the partner-authored z/OS connector** (which partner, which version, maintained or not, supported by OpenText Support or by the partner alone). Note the vendor's own legend distinguishes **Premium** (vendor-supported, extra fee), **Standard** (vendor-supported with maintenance), **Community** (community-supported only) and **Partner** (author-supported) content — **a mainframe connector in the Partner category is supported by its author.** That distinction is a real risk item and belongs as a written question at renewal.

**Practical consequence:** a bank with core banking, payments and batch on z/OS should assume **Operations Bridge's mainframe view is only ever as good as the upstream mainframe tooling it federates**, and that mainframe CIs must be modelled or z/OS events will arrive unresolved (§5.4) and never correlate. Budget the integration work; do not assume coverage.

### 8.2 Legacy platform coverage — where the vendor is genuinely broad
| Domain | Named coverage in vendor material |
|---|---|
| **Midrange / non-z** | **IBM AS/400** (IBM i) — listed, partner-supplied |
| **Databases** | Informix, **IBM DB2**, SQL Server, MongoDB, MySQL, **Oracle Database and Oracle RAC**, PostgreSQL, SAP Sybase ASE, Cassandra, Couchbase, CouchDB, MarkLogic, Riak |
| **Middleware** | **WebSphere Application Server**, **WebSphere MQ**, **WebSphere DataPower**, WebSphere IBM Bus, WebLogic, JBoss, Tomcat, Apache Web Server, ActiveMQ, Kafka, RabbitMQ, Redis, Glassfish, iPlanet, Memcached, Varnish |
| **Infrastructure** | AIX, **HP-UX**, Solaris/Zone, SUSE, RedHat, Oracle Enterprise Linux, CentOS, Debian, Ubuntu, Windows, vSphere, Hyper-V, KVM, XEN, IBM LPAR, cluster platforms |
| **Classic legacy agents** | "OpenText Operations Manager for UNIX" and "for Windows" still appear in integration tables ✅ — exactly what an inherited estate needs |
| **Enterprise apps** | SAP, SAP HANA, Exchange, SharePoint, IIS, Active Directory, Oracle Enterprise Manager, SAP Solution Manager |

✅ *Source: Operations Bridge – SaaS data sheet (`260-000148-004 | O | 01/24`) supported-technologies table.* **The whole point of this section: that table is a marketing artefact that does not distinguish first-party from connector from partner coverage.** The integration brochure's legend does distinguish them — which is why §8.1 relies on the brochure. **Never take a brochure technology row as proof a first-party agent exists.**

### 8.3 MFT and batch monitoring — the honest gap
| Question | Answer |
|---|---|
| Does Operations Bridge **natively monitor MFT/file-transfer products** (MFT platforms, SFTP gateways, SWIFT/ACH file flows)? | **No native MFT product is evidenced.** File transfer arrives via generic monitors (file-change, FTP/SSH checks in the agentless layer), via a connector from the MFT product's own tooling, or via a scheduler integration |
| Does it **own batch scheduling and the batch critical path**? | **No.** Batch scheduling is a separate discipline with dedicated products; the portfolio's story is *federation*. **Cross-ref [control_m_guide.md](control_m_guide.md) and [control_m_external_conditions_guide.md](control_m_external_conditions_guide.md) — in most banks the batch view lives there, not in the ITOM console** |
| What *does* exist that touches batch/MFT? | **Generic monitors** (file existence, file age, FTP/SFTP availability, process/job checks) ✅; **event ingestion** from whichever scheduler or MFT tool emits events ✅; **TBEA** for time-based automation (auto-close/escalate overnight job noise) ✅; **BPM** probes for end-to-end business processes in higher editions ✅ |
| The honest summary | Operations Bridge **can be made to show** MFT and batch status; it **does not own** the scheduling or the transfer. Treat batch/MFT as an integration project with named sources, and be sceptical of any claim that the ITOM platform *is* the batch monitoring solution |

### 8.4 Where collection is thin — summary
| Area | Depth | Flag |
|---|---|---|
| Distributed Linux/UNIX/Windows servers | **Deep** — first-party agent + agentless + packs | ✅ |
| Databases (incl. DB2, Oracle RAC, Informix) | **Deep** — named packs and monitors | ✅ |
| Middleware (WebSphere MQ/AS, WebLogic, Kafka, Tomcat, JBoss…) | **Deep** — named monitors and packs | ✅ |
| Network | **Separate product (NOM/NNMi)**, federated into the same console and OPTIC | ✅ |
| Cloud and containers (AWS, Azure, GCP, OpenStack, Kubernetes, Docker) | **Present** — named coverage; depth per-service varies | ✅ coverage, ⚠ depth |
| **z/OS and mainframe subsystems** | **Partner-supplied connectors and upstream forwarding** — no first-party agent evidenced | ⚠ **the key gap** |
| **MFT / file transfer** | **Generic monitors and integration only** | ⚠ |
| **Batch scheduling critical path** | **Integration only** — the scheduler's own tooling owns it | ⚠ |
| Legacy OpenText/HPE agent estates (OM for UNIX/Windows) | Still named as integration sources | ✅ |

**Cross-ref:** the payments estate whose files and batch windows this is about belongs to [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md); the middleware layer to [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md); the core systems to [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md).

---

## 9. Deployment, integration and scale reality
### 9.1 Platform and database dependencies
| Item | Finding | Flag |
|---|---|---|
| **OBM supported databases** | **Microsoft SQL Server, Oracle and PostgreSQL** — with an **embedded PostgreSQL** as the default/typical install and documented support for remote instances (including a support article specifically on upgrading an external PostgreSQL database used by OBM, and one on connecting to the embedded instance) | ✅ `Operations_Bridge_Manager/24.4/DatabaseReq`, `23.4/DatabaseReq`, the 2020.10 system-requirements page naming "SQL Server, Oracle, or PostgreSQL", and support article KM000045955 |
| **Server operating systems** | Windows and Linux server platforms; RHEL/SUSE-class Linux in containerised and classic deployments | ⚠ not re-extracted this pass — treat the specific OS matrix as a vendor-document question |
| **Containerised deployment** | **Container Edition** exists as a distinct line from **Classic** (the CE 24.2 release note splits them) ✅; a "**containerized operations bridge**" entitlement appears in Premium/Ultimate ✅ | ✅ both modes exist; ⚠ specific Kubernetes/distro matrix |
| **CDF** | **Status not established** ⚠ — the capability (container-native delivery, containerised data lake) is current, but whether CDF still ships as a named product, and at which version, was not confirmed. **Do not cite a CDF version** | ⚠ |
| **Deployment options** | **SaaS, hybrid, on-premises, private cloud, managed service**, and a **fully air-gapped AI** option | ✅ as vendor-stated |

### 9.2 The SaaS question — real, not just announced
This corrects the common assumption that the product is on-premise-only. ✅ A **"Operations Bridge – SaaS"** data sheet (`260-000148-004 | O | 01/24`) states the product "**is now available as SaaS and on-premises deployment**" and enumerates **three SaaS offerings**: **Operations Bridge Premium – SaaS** (the most comprehensive, including the 200+ integrations; OpenText experts patch and upgrade the on-premises systems); **Operations Bridge Reporting – SaaS** (out-of-the-box cross-domain reporting for visualisation in a BI tool; OpenText manages on-cloud components, the **customer** upgrades and maintains on-premise components); and **Cloud Observability – SaaS** (a standalone offering for AWS/Azure visibility that can integrate with an on-premises Operations Bridge deployment). A documented **on-prem → SaaS transition path** is offered through vendor Professional Services ✅.

⚠ **Honest caveats:** the responsibility split is **variable** — Premium SaaS includes vendor patching of on-prem components; Reporting SaaS and Cloud Observability leave on-prem upgrades to the *customer*. That asymmetry is exactly what gets lost in a renewal conversation and then discovered in an incident. Also ⚠: no evidence was found that a **fully SaaS OBM with no on-premise components** is the *default* offer — in an estate with on-prem agents the offering is hybrid in practice. For a bank, "cloud" raises data-residency and exit questions belonging with [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md).

### 9.3 Integration — the event-to-incident and automation paths
**Where the product is unusually strong, and the inventory is verifiable.** Note the vendor classifies each connector by support tier — **Premium** (vendor, extra fee), **Standard** (vendor, with maintenance), **Community** (community-supported) and **Partner** (author-supported). *That tier, not the logo, is the procurement question.*

| Integration class | Named products (from vendor material) |
|---|---|
| **ITSM / ticketing** | **ServiceNow** (AppLink and "Do IT Wise" variants), **BMC Remedy / Helix ITSM / TrueSight OM**, **Jira**, **Ivanti (Cherwell)**, **SAP Solution Manager**, **Zendesk**, **TOPdesk**, **Salesforce / RemedyForce**, **Service Anywhere**, plus OpenText's own **SMAX** and **Service Manager** ✅ |
| **Observability / monitoring peers** | Dynatrace, Datadog, AppDynamics, New Relic, Splunk Enterprise, Splunk SignalFx, SolarWinds (NPM, SAM), Zabbix, Nagios, Icinga, Zenoss, CA UIM, CA APM, CA Spectrum, Microsoft SCOM, Oracle Enterprise Manager, vRealize Operations, Nutanix, ExtraHop, IBM Netcool, IBM Tivoli (J9), BMC Impact Manager ✅ |
| **Automation / orchestration** | **Operations Orchestration (OO)** with content packs (including one for remote Operations Agent deployment) ✅; custom actions and EPI/Groovy scripting ✅; **xMatters** for on-call/paging ✅; Chef, Jenkins, HPE OneView, HPE SIM ✅ |
| **Closed-loop incident processing** | **CLIP** — creates and synchronises incidents with the service desk, instructs third-party notification systems (the vendor names **xMatters** for paging/SMS), and closes tickets across systems when resolved anywhere ✅ |
| **The event-sync nuance** | Vendor material states that in many cases "events are synchronized from OBM back to the source system" ✅ — not a one-way drain. This matters for decommissioning |
| **Ansible** | ⚠ **not named** in the inventory captured this pass. Do not assume it; the automation story is OO-centric plus custom actions/EPI |
| **Mainframe** | **IBM Mainframe zOS** — **Partner-authored connector** (§8.1) ⚠ |

### 9.4 Authentication, APIs and scale
| Item | Finding |
|---|---|
| **LDAP / SSO / Kerberos** | ⚠ **Not established this pass.** Standard for a platform of this class and almost certainly documented, but the supported mechanisms (LDAP, SAML/OIDC, Kerberos, MFA) were **not re-verified** at a primary source, and this guide will not assert them from expectation. Ask the vendor for the current authentication matrix for **both** Classic and Container Edition, in writing, and record it as dated evidence |
| **REST APIs** | Vendor material names "OBM's **monitoring automation REST web service**" ✅ and REST API use for extracting data from OBM and Cloud Optimizer ✅ |
| **Open Data Ingestion API** | Explicitly named as part of Operations Bridge — ingest data from different sources into the OPTIC Data Lake ✅ |
| **Custom development** | **Management Pack Development Kit** and **Operations Connector SDK** ✅ — the documented route to building or tailoring integrations |
| **Scripting** | **EPI** scripts, an **OBM Script Development Kit**, **Groovy Console**, Java API documentation, and custom actions with their own authorisation model ✅ |
| **Documented scale and sizing** | ⚠ **No vendor sizing guide, maximum-event-throughput figure or supported-node-count limit was located and extracted this pass.** The correct reporting is the negative: *this pass did not establish a published scale ceiling for OBM.* Vendor system-requirements pages **reference** support matrices ("see the Support Matrices for Operations Center products") ✅ — i.e. they exist behind the support-login boundary. The **Performance Dashboard** exposing pipeline statistics ✅ is the mechanism for finding *your* ceiling rather than reading someone else's. **Sizing must be established against your own measured event rate and topology size, negotiated with the vendor, and written into the contract as a supported configuration** |

### 9.5 Deployment reality checklist
- [ ] Which **edition** does the estate hold, and what is bundled versus an add-on?
- [ ] **Classic or Container Edition**, or both, and what is the target end-state?
- [ ] **Which database** is in production (embedded PostgreSQL vs external PostgreSQL vs SQL Server vs Oracle), and is it within the supported matrix at the current version?
- [ ] **Which components are actually deployed** versus licensed-but-unused? (Unused entitlements are the easiest renewal saving and the easiest migration win)
- [ ] **Is OBR still running**, and what is the plan before **31 December 2026**?
- [ ] **Is BVD load-bearing** or vestigial?
- [ ] **Which management packs and connectors are Partner- or Community-supported** rather than vendor-supported?
- [ ] **What is the SaaS/hybrid/on-prem target**, and is the responsibility split understood for the specific offering?
- [ ] **What proves the alerting path works** when the platform itself is down?
- [ ] **What is the exit plan** — data export format, schema access, realistic time to re-platform?

---

## 10. Competitive landscape and honest positioning
**Verification note:** where a product's existence and described role were confirmed at a vendor or primary source this pass the row is ✅; where the product is undoubtedly real and well-known but was **not** re-verified at a primary source during this pass, the row is ⚠ — the flag is about *this pass's evidence*, not the product's existence.

### 10.1 The shortlist an enterprise actually runs today
| Cohort | Product | Role | Flag |
|---|---|---|---|
| **ITSM-native** | **ServiceNow ITOM/AIOps Event Management** (Event Management in the Service Operations Workspace, with Metric Intelligence, Health Log Analytics, Service Reliability Management) | Event management where the ITSM platform *is* the system of record — the strongest structural competitor in a bank, because the ticket is already there | ✅ |
| | **BMC Helix Operations Management with AIOps** | AIOps-capable operations management in the Helix suite; successor positioning to TrueSight OM | ✅ |
| **Modern SaaS observability** | **Dynatrace** | Causal-AI-led, full-stack, SaaS-first — and also a *connector source* into Operations Bridge | ✅ named in vendor material |
| | **Datadog** | SaaS telemetry with an event/incident layer — also a connector source | ✅ named in vendor material |
| | **Splunk ITSI** | Correlates events across operational systems; Splunk Enterprise and SignalFx appear as connectors | ✅ named in vendor material |
| | **New Relic**, **Elastic Observability** | SaaS/OSS observability with event and alerting layers | ⚠ not re-verified this pass |
| **Legacy enterprise suites** | **IBM Cloud Pak for AIOps** | AIOps platform explicitly positioned to put AI at the core of the ITOps toolchain; heir to the Netcool/OMNIbus lineage | ✅ ibm.com/products/cloud-pak-for-aiops |
| | **IBM Netcool/OMNIbus** + z/OS coverage | The classic fault-management console, historically the mainframe-adjacent incumbent; **IBM NetCool** is also a connector into Operations Bridge | ✅ as an integration source; ⚠ current framing |
| | **BMC TrueSight OM** (legacy), **Broadcom DX** (OI / NetOps), **BMC MainView** (mainframe) | Legacy enterprise operations suites of the same generation | ⚠ **not verified this pass** — treat current names and packaging as an open item |
| **Open source** | **Zabbix**, **Nagios**, **Icinga** | OSS monitoring — *all three appear in Operations Bridge's own connector inventory*, so they are as often sources as substitutes | ✅ vendor integration material |
| | **Prometheus/Alertmanager**, **Grafana**, **OpenSearch/Elasticsearch** | The de-facto cloud-native metrics, alerting, visualisation and search stack | ⚠ not re-verified this pass (universally established); Grafana is named as a supported graphing path ✅ |

**An observation worth more than any analyst placement:** the tools most often proposed as Operations Bridge *replacements* — ServiceNow, Dynatrace, Datadog, Splunk, New Relic, Zabbix, Nagios, Icinga, SolarWinds, SCOM — are **all listed as integration sources**. That is simultaneously the product's greatest strength (it survives because it is the federation layer) and its strategic vulnerability (it sits, by design, one layer above replaceable parts).

### 10.2 The comparison, on the axes that decide a renewal
| Axis | Operations Bridge | ServiceNow ITOM | SaaS observability (Dynatrace/Datadog/Splunk ITSI) | IBM Cloud Pak for AIOps / Netcool | OSS (Zabbix/Prometheus/Grafana) | Mainframe-native (Netcool/OMNIbus, MainView) |
|---|---|---|---|---|---|---|
| **Collection & agent breadth** | **Very strong** — agent, agentless, connectors, packs across 200+ domains (*claim, vendor-sourced*) | Strong for infrastructure; strongest where ServiceNow agents/CMDB exist | Very strong for modern app/cloud; **weaker for exotic legacy** | Strong, especially IBM estate | Strong, but **you build the integration**; no vendor packs | **Strongest for z/OS** — purpose-built |
| **Correlation & topology** | Strong (SBEC + TBEC + AEC) but **CMDB-bound** | Strong and **ITSM-native** — ticket context is a real advantage | Strong for modern app topology; less so for a mainframe-heavy bank | Strong (AI-led, Netcool lineage) | **Weakest** — you implement it | Network/mainframe oriented |
| **Mainframe coverage** | **Partner-supplied, second-hand** (§8.1) ⚠ | Via integration | Via integration | **Strong** (IBM lineage) | Via integration | **Native — the reason to shortlist** |
| **On-premisability / air-gap** | **Strong — the moat.** On-prem, private cloud, air-gapped AI ✅ | Available but SaaS-oriented | **SaaS-first — weakest axis** | Strong (containerised, deployable) | **Total control** | On-prem traditional |
| **Integration (ITSM/automation)** | **Very strong and bidirectional** (CLIP, event sync back to source) ✅ | Excellent — it *is* the ITSM | Good, connector-based | Good | Good, DIY | Narrow but deep |
| **Run cost** | Licence + support + **CMDB maintenance labour nobody budgets for** | Bundled with ITSM spend, often "free" at the margin | Consumption-priced; can escalate with telemetry volume | Licence + substantial platform footprint | Lowest licence, highest labour | High licence, low integration labour (it already fits) |
| **Migration cost off it** | **High** — topology history, connector estate, operator muscle memory, suppression-rule debt | N/A | N/A | N/A | N/A | N/A |
| **Vendor concentration risk** | **Elevated** — four owners in eight years, portfolio simplification programmes | Large single-vendor concentration already | Large single-vendor concentration | Part of a broad IBM dependency | Low vendor risk, high skills risk | Similar to Operations Bridge |

### 10.3 For whom is keeping it defensible, and for whom is it not
**Defensible when:** the estate is **heterogeneous and legacy-heavy** and the integration catalogue does real work no single replacement covers without a multi-year programme; the bank genuinely needs **on-premises or air-gapped** operations management (the axis where the SaaS leaders are structurally weakest and this product genuinely strong); the **topology model is actually maintained** and TBEC is demonstrably suppressing symptoms in real incidents — in which case the accumulated model *is* the asset and replacing it discards real value; the **CMDB-to-service mapping feeds resilience work** (§11) and would have to be rebuilt either way, so a replacement cannot claim that cost as a saving; and the event-to-ticket path is already **closed-loop** with the incumbent ITSM, so the remaining gap to a full replacement is narrower than a rip-and-replace case assumes.

**Not defensible when:** the **CMDB is not maintained**, TBEC has never been demonstrated on a real incident, and the console has become a licence-and-dashboard cost while operators live in the native tools — though note the honest corollary: **a replacement fails on exactly the same precondition unless topology ownership is fixed first**; the strategic direction is **cloud-native and SaaS-first** with a small legacy tail a modern platform plus a handful of connectors could cover; **roadmap confidence is low** — tested not by the roadmap slide but by the vendor's written answers to §9.5 and §14; or the renewal is priced at parity with a full-fidelity modern platform, at which point the comparison is no longer "consolidation value vs migration cost" but "same money, older tool".

**The honest conclusion, a reasoning rather than a verdict:** Operations Bridge is a **consolidator for messy estates**, and its economics turn on one variable the vendor does not control and the buyer usually under-resources — **whether the topology model is maintained**. Everything else (the AI, the editions, the branding, the connectors) is secondary. Decide on that variable *first*, and only then on the platform. This is the same structural insight the AIOps-category guide reaches about RCA quality ([ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md) §5, §10).

---

## 11. The regulated-institution angle
**Boundary note, stated firmly:** this section does **not** re-derive the repository's resilience and risk content, and does **not** quote any specific regime's text as verified. The resilience *framework* — important business services, impact tolerances, severe-but-plausible scenarios, mapping, testing and the evidence trail supervisors ask for — belongs to [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md). The incident/change *process* belongs to [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md). The method for judging whether a component is supportable belongs to [technology_lifecycle_management_guide.md](technology_lifecycle_management_guide.md). This section owns one narrow question: **what does an ITOM platform of this class supply, and not supply, to a supervised firm?**

### 11.1 What a supervised firm needs, and what this class supplies
| Need | Supplied? | Comment |
|---|---|---|
| **Operational-resilience evidence** — demonstrable ability to detect disruption, with timestamps and an auditable trail | **Partly** | The event stream is inherently timestamped and annotations attach human decisions to machine events; a reconstructed timeline is a native output rather than an add-on |
| **Incident timeline reconstruction** — first signal, detection, correlation, human notification, escalation, resolution | **Yes** | A native output of the event stream + annotations |
| **Dependency mapping for impact tolerances** | **Only if the model is maintained** | The RTSM is *the* artefact for "what depends on what" and exactly the input an impact-tolerance analysis needs — but see §5.4 |
| **Incident-to-ticket closure** | **Yes** | CLIP closes the loop between detection and the ITSM record — the auditable join a reviewer will pull on |
| **Planned-work separation** | **Yes** | Downtime/maintenance windows (MAO) separate planned from unplanned disruption — evidence a gap was *known and controlled* |
| **Correlation explainability (rules path)** | **Yes** | Rule-based SBEC/TBEC decisions are deterministic and re-derivable — a genuinely auditable correlation story, unlike a black box |
| **Deployment control** | **Yes** | On-premise and air-gapped options satisfy data-residency and operational-independence expectations a SaaS-first alternative may not |
| **A first-class alert/action audit trail as a compliance artefact** | **No** | An *evidentiary* record of every alert, suppression decision and automated action, with actor and justification, exportable to an auditor is **not the product's framing**. It is a records-management requirement layered on top, usually built, not bought |
| **Retention policy depth** | **Unverified** | Retention is a storage/configuration question, not a compliance product. ⚠ No verified retention matrix established this pass. Verify the ceiling against the firm's own obligations — do not assume |
| **Change-control of monitoring configuration** | **No — a governance gap the bank owns** | Suppression rules, ETIs, CI edits, policy changes and correlation rules are production configuration and typically **not** subject to ITSM change management with the rigour applied to application change. This is the single most likely finding in an internal-audit review of an ITOM estate |
| **Governed suppression** | **No** | Suppression is functionally a mechanism for *deliberately not raising an alert*. Ungoverned, it is an unreviewed risk acceptance. The mitigation — owner, justification, expiry, quarterly review — is a process control, not a product feature |
| **Autonomous action accountability** | **Partly** | Automated remediation and agentic AI that "acts before issues disrupt business" (*claim, vendor-sourced*) demands a policy-guardrail, approval and reversal story the firm must design. The vendor describes guardrails, RBAC, approvals and an audit trail as features ⚠; the *accountability model* is the firm's to define |
| **AI explainability as an audit artefact** | **Open question** | A confidence score and "traceable reasoning" are operator-facing features. Whether they satisfy an auditor or model-risk reviewer is separate and harder — and decides whether ML correlation is usable in a supervised firm (§7.4) |
| **Vendor concentration / third-party risk** | **A risk in itself** | Four owners in eight years, an announced workforce reduction at closing, and an ongoing "integration and simplification" programme (per the vendor's own SEC filings) make the vendor's roadmap stability a **risk-register line**, not a procurement footnote |
| **Mainframe and MFT evidence** | **Conditionally, one level deeper** | If mainframe and MFT coverage is partner-supplied (§8), resilience evidence for those services depends on a **supplier chain one step further out** — the partner's connector, the upstream tool, and their support arrangements |

### 11.2 The supervisory framing, stated safely
The generically safe statements — those that hold without quoting any specific regime — are these: supervisors expect firms to **be able to detect and respond to disruption**; to **know their important business services and the assets they depend on**; to **test** that capability; to **evidence** it; and to **manage the risk their suppliers represent**. Every major operational-resilience regime in force across banking jurisdictions expresses some version of that shape. **What this guide will not do** is attribute a specific obligation to a specific article of a specific regime, because no supervisory text was verified at a primary source during this pass. For that, use [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) and the repo's risk guides, and verify the current text at the regulator's own site. ⚠

### 11.3 The monitoring-evidence checklist
A practical artefact that survives a supervisor conversation better than a licence inventory:

| # | Question the firm should answer with evidence |
|---|---|
| 1 | For each important business service, what monitors detect its degradation, and when did one last actually fire? |
| 2 | What is the median time from first-signal to human-acknowledgement, and is it measured? |
| 3 | Can the firm produce a *reconstructed timeline* for the last significant incident directly from the platform? |
| 4 | Which CI relationship in the model proves service A depends on component B, and who maintains it? |
| 5 | Who changed a suppression rule in the last twelve months, why, and who approved it? |
| 6 | What did the platform do autonomously in the last twelve months, and how was it reversed when wrong? |
| 7 | How long are events, annotations and closure records retained, and does that exceed the firm's obligations? |
| 8 | What proves the alerting path works **when the monitoring platform itself is down**? |
| 9 | Which monitoring components are Partner- or Community-supported, and what is the escalation path when they break? |
| 10 | What is the exit plan, and how long would re-platforming the event stream realistically take? |

---
## 12. The Cymbal Bank worked example
**All figures in this section are illustrative and fictional.** Cymbal Bank is a fictional institution used across this repository as the worked-example persona. The costs, counts and outcomes below are constructed to make a *method* concrete; they are not benchmarks, market rates, or derived from any real institution or contract. Replace every number with your own measured data before using this as a template.

### 12.1 The situation and the estate
Cymbal Bank is a mid-size universal bank. It has run Operations Bridge (in its various brand incarnations) for over a decade, and the support subscription is up for a three-year renewal with a material price increase. The platform was inherited, not chosen, by everyone currently on staff. Leadership has asked for an honest assessment: renew, renegotiate, or replace.

| Estate slice | Scale (illustrative) | Collection path in use | Coverage reality |
|---|---|---|---|
| **z/OS mainframe** — core banking, batch, CICS | 2 sysplexes | **Not a first-party agent.** Fed from IBM's tooling through a **partner-supplied connector** | **Partial.** Only what upstream tooling forwards. Batch critical-path status is *not* in the console |
| **Core banking application servers** | ~420 nodes | Operations Agent | Good — where the platform earns its keep |
| **Payments (SWIFT/RTGS/ACH) middleware** | ~60 MQ queue managers | Agent + management packs | Good for queue depth/channel status; end-to-end payment flow comes from the **BPM** entitlement, used lightly |
| **MFT / file transfer** | 4 platforms, ~2,300 daily flows | Generic file/FTP monitors only | **Thin** — "file arrived" coverage, not transfer-SLA coverage. The scheduler owns the critical path |
| **Distributed infrastructure** | ~4,500 VMs/containers | Agent + SiteScope | Good |
| **Network** | Separate NOM tool | Federated into the same console | Good |
| **Event volume** | ~38,000/day pre-reduction; ~4,100 at peak hour | — | Post-reduction queue ~640/day steady state |

### 12.2 The correlation and topology quality problem
This is where the assessment turns, and the finding is uncomfortable but common:

| Measure (illustrative) | Value | Interpretation |
|---|---|---|
| Monitored nodes resolving to a CI | **71%** | Nearly three events in ten arrive unattached and can never correlate |
| Top-50 services with a **complete** dependency path in the RTSM | **34%** | Two-thirds of the important services have no traversable causal path |
| Last full discovery refresh | **19 months ago** (a project, not a lifecycle) | The model decayed through a core banking upgrade and a DR flip |
| TBEC rules authored; demonstrated TBEC success on a real incident | 41 rules, only **9** in the last 3 years; **2 incidents, both >3 years old** | **The capability is unproven in the current estate** |
| Suppression rules active; rules with no documented owner or expiry; suppressions found masking live conditions | 118 total; **63** unowned/unexpired; **6** masking live conditions | Direct governance gap and operational risk (§11.1) |

**The honest reading:** the platform is *licensed* for topology-based correlation and is *not meaningfully delivering it*. A replacement would face the identical problem — the model, not the tool, is the deficiency. **This single finding dominates the renewal decision.**

### 12.3 The mainframe check and the integration state
Applying §8: **no first-party agent runs on z/OS**; visibility depends on IBM-side tooling forwarding into a **partner-supplied connector** ⚠ whose support status the bank **could not establish** (§12.5). **The batch critical path is not in the ITOM console** — it lives in the scheduling tool, so monitoring batch *impact* is possible while monitoring the batch *schedule* is out of scope (**cross-ref** [control_m_guide.md](control_m_guide.md)). The claim "the ITOM platform gives us mainframe visibility" therefore narrows to "it consolidates *some* mainframe events from upstream tooling". The bank decided to keep a **mainframe-native** view as the authority for z/OS and use the ITOM console as the cross-domain roll-up only — architecturally correct, and materially reducing the platform's claimed mainframe value.

On integration: the **event→incident path (CLIP) is closed-loop and working** — a genuine strength, and not a reason to leave; **events synchronising back to source** for two third-party tools creates a decommissioning dependency; **OO runbook automation is licensed for 12 flows with 4 in use** — automation value licensed but unrealised, a real recoverable benefit; **xMatters** paging works; and **Ansible is not integrated** (and not in the vendor's connector inventory, per §9.3), so if the bank standardises on Ansible that is an integration to *build*.

### 12.4 Cost of continuing versus cost of replacement (illustrative)
| Line | Continue (3-year) | Replace (3-year) |
|---|---|---|
| Subscription / licence | $2.4m | $3.1m (new platform at ~2.2× current unit cost at full telemetry) |
| Platform infrastructure | $0.5m | $0.8m |
| **Topology/CMDB remediation** | **$0.9m — required either way** | **$0.9m — required either way** |
| Migration / implementation effort | $0 | $2.6m (18-month programme, incl. retraining and connector rebuild) |
| Suppression/model debt | Carried | Debt is *rewritten*, not eliminated |
| **3-year total** | **~$3.8m** | **~$7.4m** |
| **Risk** | Roadmap and vendor concentration | Delivery risk, dual-running, mainframe re-integration, timeline slip |

**The decisive observation:** the topology remediation line is **identical in both columns**. A replacement does not escape the precondition for value; it re-buys it. The replacement case only wins if it *also* delivers capabilities the incumbent cannot — which at Cymbal came down to three: a SaaS-first operating model, developer-grade observability for the new digital channels, and mainframe-native depth.

### 12.5 Resilience evidence, the unestablished items, and the recommendation
| Evidence a supervisor conversation needs (§11.3) | Produced today? |
|---|---|
| Reconstructed incident timeline | **Yes** — natively, from the event stream and annotations |
| Dependency model for impact tolerances | **Incomplete** — 34% path coverage on top-50 services |
| Who changed a suppression rule, why, approved by whom | **No** — no audit trail; 63 rules unowned |
| What the platform did autonomously | **Thin** — automation under-used, so low exposure and low evidence either way |
| Retention verified against obligations | **Unverified** ⚠ |
| Proof the alerting path works when the platform is down | **No** — no out-of-band synthetic |
| Proven detection of an important business service disruption | **Partially** — strong for core banking and infrastructure, weak for mainframe and batch |

The platform therefore supplies roughly **half** the resilience evidence a supervised firm should want, and the missing half is mostly **governance and orchestration around it**, not product gaps. Two of the seven gaps — the suppression audit trail and out-of-band heartbeats — are cheap to close and high-value.

**What Cymbal Bank could not establish** — recorded deliberately, because these are what a renewal must convert into written answers: ⚠ whether the **partner-supplied z/OS connector** is maintained and by whom; ⚠ the **current support lifecycle dates** for the exact OBM, SiteScope and Operations Agent versions in production; ⚠ the **current status of CDF** and the supported Kubernetes delivery model; ⚠ whether **BVD** is on a support path (the bank has two live BVD dashboards a business user depends on, and no EOL notice was found either way); ⚠ the **retention ceiling** against the bank's own records obligations; ⚠ the **authentication/SSO matrix** for the current release; ⚠ any **published scale ceiling** at the versions in production; ⚠ whether the marketed **automation and GenAI features** (Aviator, the diagnostic AI agent) are available to its edition and deployment model, and at what cost.

**Recommendation: re-negotiate and retain, with conditions.** The reasoning: (1) **the value gap is topology, not tooling** — identical remediation cost in both columns makes replacement a re-buy, not a fix; (2) **the integration estate is doing real work** with a closed-loop ticket path, and replacing it means rebuilding the federation layer and re-integrating every source, risking the bidirectional event sync the source tools now depend on; (3) **the on-premise requirement is genuine and constraining** — a real differentiator (§10.2). But re-negotiate on evidence, not relationship: (a) remove unlicensed/under-used entitlements from the renewal (OO flows, unused packs); (b) obtain **written lifecycle dates for every component in production** at renewal; (c) obtain a **z/OS and batch coverage statement in writing**, naming who supports the connector; (d) require a topology-remediation plan with a named owner and a six-month target of **≥95% CI resolution**; (e) fund the two cheap resilience gaps in *this* year's budget, before signature. Then **set a decision gate at 18 months**: if CI resolution is ≥95%, TBEC has suppressed a real symptom in a real incident, and the mainframe/batch gap is knowingly accepted rather than assumed away — keep it; if the model is still at 70%, the platform's premium is buying an unrealised benefit, and the 18-month window is what makes a replacement programme affordable rather than forced.

**The transferable lesson:** the recommendation follows from **one measured number** — the CI-resolution rate — plus a governance remediation list. That is the shape an ITOM renewal assessment should take, in any bank, with any vendor.

---

## 13. The claims audit
Every claim this guide leans on, with source, source quality and date. **Product names, versions and support dates go stale fastest** — the versioned rows are the ones to re-verify first. Rows are consolidated where several claims share one source and one date.

| # | Claim | Verdict | Source | Quality | Date |
|---|---|---|---|---|---|
| 1 | HP OpenView rebranded **HP BTO Software** (2007); origin "1990s" | ✅ rebrand / ⚠ origin date approximate | Product-history literature | Secondary | 2007 / — |
| 2 | OMi 10.x is the documented ancestor line (10.00, 10.01, 10.6x doc sets) | ✅ | HP/Micro Focus OMi docs; OMi 10.00 release notes | **Primary vendor** | 2011–2013 era |
| 3 | RTSM "contains all the CIs and relationships created in OMi… automatic, DFM process, or manual via IT Universe Manager" | ✅ verbatim | HP OMi 10.01 RTSM Administration guide | **Primary vendor doc** | OMi 10.01 |
| 4 | Spin-merge **completed 1 Sep 2017**, **~$8.8bn**, delivering ~$8.8bn to HPE and its stockholders; combined revenue $4.4bn | ✅ | Micro Focus completion release 2017-09-01; **HPE completion announcement filed with the SEC** (Ex-99.1) | **Primary corporate** | 1 Sep 2017 |
| 5 | Spin-merge split "Micro Focus shareholders ~50.1% / HPE stockholders" | ⚠ **not verified** at a primary document | Deal reporting | Secondary | — |
| 6 | A "**50.1% / 50.1%**" split | **rejected** | Sums to 100.2% | — | — |
| 7 | OpenText **announced 25 Aug 2022** at **532 pence per share**, enterprise value **~$6.0bn** fully diluted; total purchase price $6.0bn incl. cash and debt | ✅ | OpenText release 2022-08-25; PR Newswire | **Primary corporate** | 25 Aug 2022 |
| 8 | OpenText **closed 31 Jan 2023**; closing price **~$5.8bn** incl. cash and debt; ~8% workforce reduction | ✅ | OpenText release 2023-01-31 | **Primary corporate** | 31 Jan 2023 |
| 9 | Acquisition price was "**$5.80 per share**" | **rejected** | Offer was **532p/share**; **$5.8bn** is the closing total | — | — |
| 10 | "As of January 31, 2023, the Material is now offered by OpenText"; HP/HPE/Micro Focus marks historical | ✅ verbatim | OpenText community site footer | **Primary vendor** | observed 2026-09-17 |
| 11 | `/products/operations-bridge` **301-redirects** to `/products/ai-operations-management`; current name is "OpenText AI Operations Management (Operations Bridge)"; community collateral also labels the family **"AIOps and Observability (fka Operations Bridge)"** ⚠ (snippet only) | ✅ | web.archive.org capture; current + archived product pages | Archived primary; **primary vendor** | **23 Dec 2024** snapshot; fetched 2026-09-17 |
| 12 | **OBM 24.4 released 6 November 2024**; doc sets for **2023.05 / 23.4 / 24.1 / 24.4** and a correlation page under **25.4** (`doc/386/25.4/conttbec`) | ✅ | Support notice KM000036119; `docs.microfocus.com` doc URLs | **Primary vendor** | 06 Nov 2024; fetched 2026-09-17 |
| 13 | **"Operations Bridge CE 24.2 (Classic Edition)"** release exists; Classic and Container products split; **"What's New in OpenText AI Operations Management 25.4"** dated **30 Oct 2025**; quarterly `YY.Q` cadence | ✅ | Vendor community release blog; product-page blog listing | **Primary vendor** (title/date only — the 25.4 body was not extractable) | 24.2 era; 30 Oct 2025 |
| 14 | **SiteScope renamed "Operations Bridge Express (SiteScope)"; 25.4 released 28 Oct 2025** | ✅ | Support notice KM000043535 | **Primary vendor** | 28 Oct 2025 |
| 15 | **OBR obsolescence**: end of sale **1 Oct 2021**; last download **31 Oct 2021**; 10.50 committed to 31 Dec 2022, **extended to 31 Dec 2026**, self-help to 31 Dec 2030; 10.4x extended to 31 Dec 2024; 10.3x extended to 31 Mar 2024; superseded by **OPTIC Data Lake (Vertica)**; affected suites OpsB, NOM, DCA | ✅ verbatim | Micro Focus OBR obsolescence support notification (PDF) | **Primary vendor** | © 2021 |
| 16 | **BVD is end-of-life / retired** | ⚠ **not established**; no EOL notice found | — | — | — |
| 17 | BVD still an **active reporting/dashboard surface**: named in the **Jun 2023** OBR→OPTIC data sheet, in **Data Protector 24.4** docs, and on the ITOM Marketplace | ✅ as "still referenced" | `260-000252-001 | 06/23`; `doc/200/24.4/bvd`; marketplace listing | **Primary vendor** | Jun 2023 / 24.4 |
| 18 | **RTSM is no longer a separately-sold SKU**; **Universal Discovery and CMDB** is the discovery/CMDB entitlement, and **UD/CMDB 26.1** exists | ✅ substance / ✅ listing | Archived Dec 2024 edition matrix; OMi RTSM docs; support-portal article KM000044509 | Archived primary + vendor docs | Dec 2024; observed 2026-09 |
| 19 | **OO** line includes **2023.05** (Python Executor) and **24.1 / 24.1.1** (AFL workflow editing beta) | ✅ | `doc/Operations_Orchestration/2023.05/ReleaseNotes`, `/24.1.1/ReleaseLog` | **Primary vendor** | 2023 / 24.1 |
| 20 | **Operations Agent** 12.14 / 12.15 / 12.25 / 12.26 exist | ✅ | Community blog; KM000023404; `doc/Operations_Agent/12.26/Home` | **Primary vendor** | observed 2026-09 |
| 21 | Operations Agent moved to a "2x.xx" line | ⚠ **no evidence either way** | — | — | — |
| 22 | **OBM databases: Microsoft SQL Server, Oracle, PostgreSQL** (embedded PostgreSQL default; remote instances documented) | ✅ | `Operations_Bridge_Manager/24.4/DatabaseReq`, `23.4/DatabaseReq`, 2020.10 system requirements, KM000045955 | **Primary vendor** | 23.4 / 24.4 |
| 23 | **Operations Bridge – SaaS exists**, with **Premium / Reporting / Cloud Observability** SaaS offerings and a documented on-prem→SaaS transition service | ✅ | SaaS data sheet `260-000148-004 | 01/24` | **Primary vendor collateral** | 01/24 |
| 24 | Event pipeline: gateway → data processing, **CI resolution** (hints + cache), **ETI resolution**, **duplicate event suppression**, **event suppression**, **event storm suppression**, **close related events**, **SBEC**, **TBEC** (Correlation Manager, cross-domain), **TBEA**, **EPI** scripting, custom actions, forwarding | ✅ | Vendor-authorised **OBM-EPAC 24.2** course outline | Secondary (vendor syllabus) | OBM 24.2 |
| 25 | Service-health internals: **Health Indicators**, **KPIs**, **KPI Enrichment Service (KES)**, **Multi-process Architecture Business Logic Engine (MARBLE)** | ✅ named / ⚠ mechanics | Same course outline | Secondary (vendor syllabus) | OBM 24.2 |
| 26 | **AEC (Automated Event Correlation)** operates against **OPTIC DL**, with its own architecture, UI, Classic and Containerized configuration and troubleshooting content | ✅ | Same course outline (Module 11) | Secondary (vendor syllabus) | OBM 24.2 |
| 27 | **TBEC** is used "to automatically identify and display the real cause of problems"; custom TBEC rules configurable via the Correlation UI | ✅ | `doc/386/25.4/conttbec`; `.../24.4/AddingCustomTbecRules` | **Primary vendor** (snippets) | 25.4 / 24.4 |
| 28 | **CLIP** creates/synchronises incidents, instructs third-party notification (xMatters), closes tickets across systems; in many cases **events sync back to the source** | ✅ | Integrations brochure `263-000031-003 | 01/24` | **Primary vendor collateral** | 01/24 |
| 29 | Integration inventory includes **ServiceNow (AppLink, Do IT Wise), BMC Remedy/Helix ITSM/TrueSight OM, BMC Impact Manager, Jira, Ivanti/Cherwell, Zendesk, TOPdesk, Salesforce, SAP Solman, Dynatrace, Datadog, AppDynamics, New Relic, Splunk (+SignalFx), SolarWinds, Zabbix, Nagios, Icinga, Zenoss, CA UIM/APM/Spectrum, SCOM, Oracle EM, vRealize Operations, Nutanix, ExtraHop, IBM Netcool, IBM Tivoli (J9), xMatters, Chef, Jenkins, HPE OneView** | ✅ | Integrations brochure; SaaS data sheet | **Primary vendor collateral** | 01/24 |
| 30 | **"IBM Mainframe zOS"** is a **Partner Authored Connector**; "IBM AS/400 / IBM DB2 / IBM Mainframe" sit in the **Partner** row | ✅ | Integrations brochure; SaaS data sheet | **Primary vendor collateral** | 01/24 |
| 31 | A **first-party Operations Agent on z/OS**, or any first-party z/OS collector product under a current name | ⚠ **no evidence found / not established either way** | — | — | — |
| 32 | Operations Bridge **natively monitors MFT platforms / owns the batch critical path** | **rejected** as stated | No native product evidenced | — | — |
| 33 | Middleware/database depth: **WebSphere AS, WebSphere MQ, WebSphere DataPower, WebLogic, JBoss, Tomcat, Kafka, ActiveMQ, RabbitMQ, Redis, Informix, DB2, Oracle RAC, SAP Sybase ASE** | ✅ as listed coverage | SaaS data sheet supported-technologies table | **Primary vendor collateral** (does not distinguish first-party from partner) | 01/24 |
| 34 | Containerised delivery is current: **"containerized operations bridge"** entitlement + **Container Edition** branding + containerised OPTIC DL. **CDF still shipping as a named, versioned product** | ✅ capability / ⚠ **CDF not established** | Archived Dec 2024 matrix; CE 24.2 blog; OBR notice | Archived primary + vendor | Dec 2024 / 2021 |
| 35 | **LDAP / SSO / Kerberos** integrations supported; **Ansible** in the vendor inventory | ⚠ **not verified / not found** | — | — | — |
| 36 | **REST APIs** exist, incl. an **OBM monitoring-automation REST web service** and an **Open Data Ingestion API** into OPTIC DL; **Management Pack Development Kit** and **Operations Connector SDK** exist | ✅ | Integrations brochure; OBR→OPTIC data sheet | **Primary vendor collateral** | 01/24 / 06/23 |
| 37 | A **published maximum event throughput or node count** for OBM; an extracted **OS/Kubernetes support matrix** | ⚠ **not established**; vendor references "Support Matrices" behind the support login | 2020.10 system-requirements pointer | **Primary vendor** (pointer only) | — |
| 38 | **"Operations Bridge Analytics"** exists as an entitlement; **Aviator** blog dated **17 Nov 2025** | ✅ entitlement & title/date / ⚠ feature contents | Archived Dec 2024 edition matrix; product-page blog listing | Archived primary; **primary vendor** | Dec 2024; 17 Nov 2025 |
| 39 | Vendor performance claims: **"AI-driven correlation cuts event volume by 30-95%"**, **"94% improved service availability"**, **"49% fewer service outages"**, **"1000+ integrations"**, **"200+ domains"**, **"50+ patents"**, alarm reduction "over 70%" at one customer, "Market Leader, Vendor Selection Matrix AIOps Platforms" | ⚠ **all *claims, vendor-sourced***; the 30–95% range is unfalsifiable, the outcome metrics are undefined and single-customer, and the analyst placement is paid-access material | Current product page; SaaS data sheet; archive | Vendor marketing | current / 01/24 |
| 40 | "Anomaly detection with learned baselines" flags creeping latency, leaks, unusual volume | ✅ product claim / ⚠ outcome | Current product page | Vendor marketing | current |
| 41 | OpenText expanded a **Business Optimization Plan** in April 2025 for integration/simplification **following the Micro Focus acquisition** | ✅ | OpenText Form 8-K, 2025-04-29 | **Primary regulatory filing** | 29 Apr 2025 |
| 42 | **ServiceNow ITOM/AIOps Event Management**, **IBM Cloud Pak for AIOps** exist as described; **BMC Helix Operations Management with AIOps** exists as described | ✅ | servicenow.com; ibm.com/products/cloud-pak-for-aiops; PeerSpot/TrustRadius | **Primary vendor** (×2); secondary | current |
| 43 | **Broadcom DX OI / DX NetOps**, **BMC MainView**, **New Relic**, **Elastic**, **Prometheus/Alertmanager** current framing | ⚠ **not verified this pass** (universally established; New Relic and Elastic appear as connector sources ✅) | Vendor collateral (connector rows) | Vendor | 01/24 |

---

## 14. What Could Not Be Verified
The honest residue. Each item is a gap in *this pass's* evidence, not a claim that the fact is false. Every one is a legitimate written question to put to the vendor at renewal.

1. **The current Operations Agent version line** — 12.14/12.15/12.25/12.26 were confirmed, but whether the agent has moved to a "2x.xx" line was **not established either way**.
2. **CDF (Container Deployment Foundation) status and version** — including the version that accompanied OBM 2020.x/2023.05. The *capability* is current; the *named product* was not confirmed.
3. **The OBM OS/Kubernetes support matrix** — specific operating systems, distributions and Kubernetes versions.
4. **Any published scale ceiling** for OBM: maximum supported event throughput, CI count, or monitored-node count.
5. **Authentication and directory integration specifics** — LDAP, SAML/OIDC SSO, Kerberos, MFA — for both Classic and Container Edition — and **published RTO/RPO and HA topology** for OBM server components.
6. **Retention limits** for events, annotations and closure records, against banking record-retention obligations.
7. **Business Value Dashboard's true lifecycle status.** No EOL notice was found and BVD is still referenced in 2023–2024 vendor material, but whether it is on an active development path, a maintenance path, or a quiet retirement was **not established**.
8. **The complete and current vendor product lifecycle / software support lifecycle page** for Operations Bridge components. Identified as the single best source for the sunset question, and **a machine-readable lifecycle page could not be extracted this pass** — the support portal is JavaScript-heavy and the consolidated listing was not captured. **This is the most consequential gap in the guide:** the sunset findings rest on individual release notices and one PDF obsolescence notification (OBR), not on the consolidated lifecycle table.
9. **The exact HPE/Micro Focus spin-merge ownership split** (commonly reported as ~50.1% to HPE stockholders, unconfirmed at a primary document) and **a precise HP OpenView first-release date**.
10. **The current status and support owner of the partner-authored z/OS connector**, and of partner-authored connectors generally — plus **whether any first-party product ever provided an Operations Agent on z/OS**, and its disposition.
11. **The MFT/batch coverage claim in any productised form** — the *absence* was inferred from the inventory rather than from an explicit vendor statement of scope.
12. **Feature-level contents of "Operations Bridge Analytics"** as a distinct product (release train, algorithms, console surfaces), and **the contents of the 25.4 release** beyond the blog title and date (the body could not be extracted).
13. **Independent, non-vendor measurement** of event reduction or RCA accuracy for this product in a **banking** estate. Every reduction figure in the vendor material is vendor-sourced or single-customer.
14. **The current (2026) edition/entitlement structure** — the archived Dec 2024 matrix (Express/Premium/Ultimate) and the current page (Express/Premium with add-ons) differ, and the current page's feature ticks did not extract cleanly — and **whether "Express" is now the licensed SiteScope name in all geographies** or a marketing tier distinct from the release-notice product name.
15. **Operations Orchestration's forward release cadence** — the line was confirmed through 24.1.1; whether OO is still actively releasing in 2025/2026 was not established.

---

## 15. Glossary
| Term | Meaning |
|---|---|
| **AEC** | Automated Event Correlation — ML-driven correlation against the OPTIC Data Lake; configurable for Classic and Containerized OBM |
| **AIOps** | Analyst-coined label for applying AI/ML to IT operations. **A marketing category as much as a technical one** — claiming it claims category membership, not a capability level |
| **Anomaly detection** | Flagging behaviour that deviates from a *learned baseline* rather than a static threshold |
| **BVD** | Business Value Dashboard — flexible dashboarding of business/IT KPI data; still referenced by the vendor, strategic centre moved to OPTIC Reporting |
| **CDF** | Container Deployment Foundation — the historical container/Kubernetes delivery foundation; current named-product status **not established** |
| **CI / class model / IT Universe Manager** | Configuration Item — a modelled element in the RTSM; the class model is the schema of CI types and legal relationships; IT Universe Manager is the RTSM's administrative UI |
| **CI resolution** | The pipeline step mapping an arriving event to a CI using source hints. **The precondition for all topology-based correlation** |
| **Classic / Container Edition** | The two current delivery modes of the Operations Bridge products; vendor release notes split them |
| **CLIP** | Closed-Loop Incident Processing — event↔incident creation, synchronisation and cross-system closure with ITSM |
| **CMDB** | Configuration Management Database — the store of CIs and relationships; here, the *data-model* subject cross-referenced rather than owned |
| **Correlation / Correlation Manager** | Inferring a causal or structural relation between events so symptoms are demoted and the cause surfaces; the Correlation Manager is the OBM surface for creating and managing correlation rules, including manual event relation |
| **Dedup / duplicate suppression; event storm suppression** | Folding repeated identical events; a dedicated mechanism to dampen event storms rather than suppress individual events |
| **DFM** | The dynamic/automatic modelling process deriving relationships from collected data |
| **EPI** | Event Processing Interface — scripting hooks into the OBM event pipeline (with an SDK and Groovy support) |
| **ETI** | Event Type Indicator — the type-level classification of an event enabling sophisticated processing |
| **Event forwarding** | Sending events onward to ITSM or other consumers; in many cases events also synchronise back to the source system |
| **Health Indicator (HI) / KPI; KES; MARBLE** | The service-health roll-up construct computed from the event stream over the model; KES is the KPI Enrichment Service and MARBLE the Multi-process Architecture Business Logic Engine |
| **ITOM** | IT Operations Management — the solution-area category this product belongs to |
| **MAO** | Maintenance-Aware Operations — scheduled-maintenance/downtime windows so planned work does not raise incidents. Widely used abbreviation; verbatim primary definition **not captured this pass** |
| **Management Pack** | Domain-specific content bundle: collections, monitoring policies, topology and TBEC rules. Classified Premium / Standard / Community / Partner by support tier |
| **Manager of managers** | The federation pattern: leave domain tools in place, consolidate their output in one console |
| **OA** | Operations Agent — the on-node agent for metrics, events and local actions |
| **OBM** | Operations Bridge Manager — the event and service-health console, the core component |
| **OBR / SHR** | Operations Bridge Reporter / Service Health Reporter — the historical Vertica-based reporting product. **Obsolete**: end of sale 1 Oct 2021; extended support to 31 Dec 2026 for 10.50 |
| **OM / OVO / OMi** | The naming strata of the ancestor products (Operations Manager; OpenView Operations; Operations Manager i) — still visible in config, policy and URL paths |
| **Open Data Ingestion / Operations Connector** | The API path for ingesting arbitrary data into the OPTIC Data Lake; the adapter class pulling events/metrics/topology from third-party tools into OBM (has an SDK) |
| **OPTIC / OPTIC DL** | The OpenText OPTIC platform and its **Data Lake** — containerised, Vertica-powered common store for events, metrics and topology; successor to OBR's reporting role |
| **RTSM** | Run-Time Service Model — the CI-and-relationship model embedded in OBM; drives service health and TBEC |
| **SBEC / TBEC / TBEA** | Stream-Based Event Correlation (over the event stream, no topology required); Topology-Based Event Correlation (over the RTSM relationship graph); Time-Based Event Automation (scheduled/condition-based automation of event handling) |
| **SiteScope** | Agentless monitoring product; now branded **Operations Bridge Express (SiteScope)** |
| **Symptom vs cause** | A symptom's cause is another event already in the queue; the cause has none |
| **UD / UCMDB** | Universal Discovery and CMDB — the discovery engine and CMDB product that populates topology |

---

## 16. Cross-references and further reading
### 16.1 Inside this repository — and the boundary restated
| Guide | Relationship |
|---|---|
| [ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md) | **The AIOps-category companion** — the coinage, vendor stacks, anomaly detection and RCA theory, a worked rollout. §7 here is only the claims audit |
| [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) | **The ITSM-process companion** — ITIL 4, the service desk, incident/problem/change, the CMDB discussion. §4.5, §9.3 and §11 cross-ref it |
| [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md), [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) | **The availability and platform-monitoring companions** — the nines, SLOs and error budgets above the event stream, and the cluster-native monitoring stack |
| [technology_lifecycle_management_guide.md](technology_lifecycle_management_guide.md) | **The lifecycle method** behind every §3 and §13 verdict |
| [control_m_guide.md](control_m_guide.md), [control_m_external_conditions_guide.md](control_m_external_conditions_guide.md) | **The batch-scheduling companion** — where the batch critical path actually lives (§8.3, §12.3) |
| [data/dataops_guide.md](data/dataops_guide.md), [finops_guide.md](finops_guide.md) | The data-pipeline-quality angle on §5's CMDB population problem; the cost-and-renewal angle of §12 |
| [secops_guide.md](secops_guide.md), [kargo_gitops_guide.md](kargo_gitops_guide.md), [mlops_lifecycle_frameworks_guide.md](mlops_lifecycle_frameworks_guide.md), [ai_llm/agentops_guide.md](ai_llm/agentops_guide.md) | The monitoring-to-SOC boundary; the GitOps change-delivery angle; AEC model governance; where agentic remediation is in scope |
| [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) | **THE resilience cross-ref** — impact tolerances, important business services, the evidence trail §11 asks about |
| [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md), [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) | The estate §12 inventories; the payments stream; the middleware §8.2 covers well |

**The boundary, declared once more.** Because **"monitoring" in a bank is a contested term**, the division of labour is explicit: **this guide** owns the ITOM *product* and the *event-management architecture*, the component currency evidence, and the renewal economics; **the resilience guides** own operational-resilience and DR *frameworks*; **the data-architecture content** owns the CMDB as a *data-model* subject; **the ITIL/service-management content** owns the incident-management *process*; **the SRE/DevOps content** owns observability *engineering* (tracing, metrics, SLOs, error budgets).

### 16.2 Primary sources used in this pass (fetched 2026-09-17)
- **`docs.microfocus.com`** — the OpenText Documentation Portal: Operations Bridge Manager (`Operations_Bridge_Manager/2023.05`, `/23.4`, `/24.4`, `/latest`; `doc/386/25.4/conttbec`), SiteScope (`2023.05`), Operations Agent (`12.26`), Operations Orchestration (`2023.05`, `24.1.1`), Universal Discovery and CMDB, and Data Protector's BVD integration page (`doc/200/24.4/bvd`). *The portal is JavaScript-heavy — several page bodies returned only navigation chrome, and pages were read via search-result snippets where the body would not extract; this is flagged wherever it affects a claim.*
- **`portal.microfocus.com`** — OpenText Support release notices: OBM 24.4 (KM000036119, 06-Nov-2024), Operations Bridge Express (SiteScope) 25.4 (KM000043535, 28-Oct-2025), Operations Agent 12.25 (KM000023404), OO 2023.05 (KM000018755), external-PostgreSQL upgrade considerations (KM000045955), and the trending listing for Universal Discovery and CMDB 26.1 (KM000044509).
- **`opentext.com`** — the product page `products/ai-operations-management`; press releases "OpenText to Acquire Micro Focus International plc" (2022-08-25) and "OpenText Buys Micro Focus" (2023-01-31); collateral: Operations Bridge – SaaS (`260-000148-004 | O | 01/24`), Full-Stack AIOps: Operations Bridge Integrations (`263-000031-003 | O | 01/24`), Evolution Journey from OBR to OPTIC Reporting (`260-000252-001 | O | 06/23`).
- **`microfocus.com`** — "Operations Bridge Reporter product obsolescence announcement / support notification" (PDF, © 2021). **`community.opentext.com`** — "What's New in Operations Bridge CE 24.2 (Classic Edition)"; the Operations Bridge and Operations Orchestration community groups; the OBR forum discontinuation notice; blog listings for "What's New in OpenText AI Operations Management 25.4" (30 Oct 2025) and "Unlock the power of AI with Aviator" (17 Nov 2025).
- **HPE / Micro Focus spin-merge completion** — HPE's completion announcement filed with the SEC (Ex-99.1, 2017-09-01) and the Micro Focus completion press release (1 September 2017).
- **web.archive.org** — the `products/operations-bridge` → `products/ai-operations-management` 301 redirect capture, snapshot **23 December 2024**, and the archived AI Operations Management page from the same date (the Express/Premium/Ultimate edition matrix).
- **HP OMi lineage** — `docs.microfocus.com/OMi/10.6x` integration guides; support-KB copies of the OMi 10.00 Release Notes and OMi 10.01 RTSM Administration guide.
- **Training collateral (secondary)** — the vendor-authorised **OBM-EPAC 24.2** course outline delivered by an OpenText training partner: the source for the event-pipeline stage list, KES/MARBLE and the AEC description.
- **Competitor verification** — `ibm.com/products/cloud-pak-for-aiops`; `servicenow.com` ITOM/AIOps Event Management material; BMC Helix Operations Management comparison listings.
- **Legal/environmental note** — as of **31 January 2023** the material referenced is offered by OpenText; HP, Hewlett Packard Enterprise/HPE and Micro Focus marks are historical and the property of their respective owners (per the vendor's own site footer, observed 2026-09-17).

### 16.3 How to keep this current
The shelf life is set by §3 and §13, not by the architecture content. Minimum viable routine: (1) re-check the vendor's **support-lifecycle listing** quarterly — the authoritative answer to the sunset question this pass could not fully close (§14 item 8); (2) re-check **release notices** for OBM, Operations Bridge Express (SiteScope), the Operations Agent and OO each quarter, recording the version **with its date**; (3) watch for the **OBR support cliff on 31 December 2026** and for any corresponding notice on **BVD**; (4) re-check the **edition/entitlement matrix** annually — it has already changed once between the Dec 2024 archive and the current page; (5) keep a **dated evidence folder** for the vendor's written answers to §12 and §14. That folder, not this guide, is what a renewal negotiation actually runs on.

---

## 17. Closing summary
1. **Operations Bridge solves a real and durable problem** — one correlated event stream over a heterogeneous estate — and it survives because the estate, not the tooling fashion, is the hard part. Its moat is **breadth of ingestion plus on-premisability and air-gap capability**, and that moat is genuine.
2. **The ownership chain is a customer issue, not trivia.** HP → HPE (2015) → Micro Focus (spin-merge completed **1 September 2017**, ~**$8.8bn**) → OpenText (announced **25 August 2022** at **532p/share**, ~$6.0bn; closed **31 January 2023** at ~**$5.8bn**). Four owners in eight years means naming strata in your estate, documentation across two domains, a product page that silently renamed itself, an announced workforce reduction, and an "integration and simplification" programme that is, in a customer's terms, **portfolio prudence risk**.
3. **The currency findings are the most perishable and most useful content.** OBM runs on a quarterly `YY.Q` cadence, with **24.4 released 6 November 2024** and a **25.4** line evidenced into October 2025. The suite has renamed itself **OpenText AI Operations Management** while keeping Operations Bridge as the alias.
4. **The sunset finding is real and dated.** **OBR is obsolete**: end of sale **1 October 2021**, and its **extended support ends 31 December 2026** for the 10.50 line Operations Bridge customers hold. Its successor is **OPTIC reporting on the OPTIC Data Lake** — not, as the brief hypothesised, the "Operations Bridge Analytics" product.
5. **One hypothesis was rejected and one partly upheld.** **BVD is not established as retired** — still referenced in June 2023 and Data Protector 24.4 vendor material, though its strategic centre has moved to OPTIC Reporting and third-party BI. **RTSM is indeed no longer a standalone SKU** — it is the model embedded in OBM, while **Universal Discovery and CMDB** is the separately-sold discovery product.
6. **Two more hypotheses fell.** **SiteScope is not sunset** — it is current, renamed **Operations Bridge Express (SiteScope)**, at **25.4 released 28 October 2025**. **A SaaS offering is not merely announced** — an Operations Bridge – SaaS data sheet with three named offerings exists and an on-prem→SaaS transition service is documented, albeit with a responsibility split a bank must read carefully.
7. **The pricing claim was corrected.** The offer was **532 pence per share**; **$5.8bn** is the closing total consideration. A "$5.80 per share" characterisation is rejected, and a "50.1%/50.1%" spin-merge split is arithmetically impossible.
8. **The mainframe finding is the one a bank must not skip.** No evidence was found of a **first-party Operations Agent on z/OS**. Mainframe coverage arrives through **partner-authored connectors** and forwarding from IBM's and BMC's own tooling — and partner content is supported by its author, not the vendor. Batch and MFT coverage is thinner still: **the platform can show MFT and batch status; it does not own the scheduler or the transfer.**
9. **The CMDB finding is the whole ballgame.** TBEC — the capability that makes this product class worth owning — is **completely dependent on CI resolution and a maintained model**. That is not a data-hygiene footnote; it is the product's precondition, and the failure mode is silent: bad model, no-op correlation, collapsing operator trust.
10. **The AIOps audit lands on "genuine but second-mover".** AEC is a real, configurable, documented capability on a real data-lake substrate; anomaly detection ships; agentic and GenAI features (Aviator, a diagnostic AI agent) are being attached. The marketing numbers — a **30–95%** event reduction, **94%** availability improvement, **49%** fewer outages, **1000+** integrations — are *vendor-sourced claims*, and a range spanning 30 to 95 percent is not evidence. `AIOps` remains **a marketing category as much as a technical one**.
11. **The competitive conclusion is a reasoning, not a verdict.** Operations Bridge is a **consolidator for messy, legacy-heavy, on-premise-constrained estates** — defensible where the topology model is genuinely maintained and on-premisability is a real requirement, and not defensible where the model has decayed into decoration and the platform's premium is buying an unrealised benefit. **What this guide could not establish matters as much as what it did:** fifteen items are listed in §14, and the most consequential is the absence of an extracted, consolidated **vendor support-lifecycle listing**. An explicit "not established" is a finding — the honest boundary of the evidence, and every one of those items is a written question for the next renewal meeting.

The enterprise that inherits this platform inherits an event stream, a model, and a decade of accumulated correlation debt. What it should buy at renewal is not more alerts, and not a new logo on the same pipeline — it is the discipline to make one queue workable again, so that an operator sees a service in trouble rather than a storm of symptoms, and so that the platform finally earns its licence by delivering **the correlated event**.

