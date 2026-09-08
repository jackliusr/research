# Technology Lifecycle Management (TLM): From Plan to Retirement — A Deep-Dive Guide

*The cross-cutting deep-dive on technology lifecycle management — the discipline that turns an IT estate into a governed portfolio: every asset known, every version tracked, every end-of-support date on a register, and every retirement executed with the same rigour as the acquisition. From the five lifecycle phases to the vendor EOL/EOSL ladder, from MAS technology-risk expectations to a mainframe-retirement worked example at a Cymbal Bank.*

> **Author:** Jack Liu Shurui, Solution Architect
> **Purpose:** Research reference for the TLM discipline — the definition and lifecycle phases; the vendor support-lifecycle ladder (EOL/EOSL) and how it differs across vendors; the technology-risk and regulatory angle (MAS expectations, audit visibility); the operating model (asset inventory, lifecycle tracking, governance gates); software and hardware lifecycle practice; retirement and decommissioning; tooling (CMDB, SAM, and the contested "TPM" label); closing with a worked TLM-program example at a Cymbal Bank.
> **Repo:** https://github.com/jackliusr/research
> **Last Updated:** September 2026
> **Companion guides:** this file lives in `technology/`, so same-folder siblings link by plain filename and other folders by `../<folder>/` prefix: [IT Strategy](../management/it_strategy_guide.md) · [Vendor Management](../management/vendor_management_guide.md) (vendor lifecycle and exit) · [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) (the technology-risk angle) · [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) (MAS tech-risk notices and guidelines) · [Core Banking Systems](../banking/core_banking_systems_guide.md) and [Coreless Banking](../banking/coreless_banking_guide.md) (mainframe/legacy estate reality) · [Agent Versioning](ai_llm/agent_versioning_guide.md) (agent/software versioning) · [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) (software-supply-chain inventory in a COBOL shop) · [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md) · [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md) · [DevStack / OpenStack](devstack_openstack_guide.md) (the platform-lifecycle cluster).

**Verification-markers convention.** Facts checked against a primary source during this pass (vendor lifecycle pages, standards bodies, regulator pages) are marked ✅. Claims that are only partially verified, that differ across sources, or where practice genuinely varies are marked ⚠. Claims that could not be verified against a primary source are marked ❌ or live in the "What Could Not Be Verified" section (§14). The consolidated ledger is the Claims Audit (§13). Web checks were bounded (≤10 calls), so body markers that survived to the audit reflect what a single primary-source pass could confirm; anything else is honestly flagged.

**How this guide is organised:** §1 defines TLM and scopes the asset estate; §2 walks the five lifecycle phases and maps them onto the industry frameworks (ITIL, ISO/IEC 20000 and 19770, COBIT, NIST, Gartner-style portfolio thinking); §3 turns to the vendor side — the EOL/EOSL support ladder as published by Microsoft, Oracle, Red Hat, IBM and others; §4 connects TLM to technology risk (condensed from the ERM guide); §5 covers MAS technology-risk expectations; §6 faces the mainframe/COBOL legacy reality; §7–§10 are the practice: the TLM operating model, software lifecycle, hardware lifecycle, and retirement/decommissioning; §11 covers tooling and the contested "TPM" label; §12 is the Cymbal Bank worked example (asset inventory, EOL risk register, mainframe-retirement case study); §13 is the Claims Audit; §14 records what could not be verified; §15 is the glossary and closing. Each body section ends with a cross-reference line instead of re-deriving sibling content.

---

### Table of Contents

1. What Technology Lifecycle Management Is
2. The Lifecycle Phases: Plan, Acquire, Deploy, Operate, Retire
3. The Vendor Lifecycle: EOL, EOSL, and the Support Ladder
4. TLM as Technology Risk
5. The Regulatory Angle: MAS Technology-Risk Expectations
6. The Legacy Reality: Mainframes, COBOL, and the Long Tail
7. The TLM Operating Model: Inventory, Tracking, Governance Gates
8. The Software Lifecycle: Versions, Patches, Supply Chain
9. The Hardware Lifecycle: Procurement to Decommissioning
10. Retirement and Decommissioning Done Right
11. Tooling: CMDB, SAM, and the TPM Question
12. The Worked Example: A TLM Programme at Cymbal Bank
13. Claims Audit
14. What Could Not Be Verified
15. Glossary and Closing

---

## 1. What Technology Lifecycle Management Is

### 1.1 Definition

**Technology lifecycle management (TLM)** is the end-to-end, governance-backed discipline of shepherding every item of the technology estate — software, hardware, cloud services, firmware, licences, and the data attached to them — through its whole life: *plan → acquire → deploy → operate → retire* — so that at every moment the organisation knows what it has, what state it is in, when vendor support ends, what it costs, and what happens to it (and its data) when it goes away.

The load-bearing words are **"governance-backed"** and **"every item"**:

- **Every item** — TLM fails the moment it becomes a sample. One un-inventoried application, one "it's in someone's drawer" server, or one product on an undocumented release is exactly where an end-of-support surprise comes from. TLM is a *complete-coverage* discipline or it is not TLM.
- **Governance-backed** — inventory data alone is asset management; TLM adds the decision machinery: who approves an acquisition, who signs off a version upgrade, who accepts residual end-of-life (EOL) risk, and who is accountable when a product is retired. The governance overlay is what separates a TLM *programme* from a CMDB with good intentions.

### 1.2 What TLM is not (the boundary map)

| Neighbouring discipline | What it owns | How it relates to TLM |
|---|---|---|
| **IT Asset Management (ITAM)** | Financial and inventory control of IT assets across their life (ITIL 4 defines ITAM as a practice) | TLM uses ITAM data as its substrate; ITAM without lifecycle decisions is bookkeeping |
| **Software Asset Management (SAM)** | Software licences, entitlements, usage, compliance (ISO/IEC 19770 family) | A subset of TLM scoped to the software estate; TLM extends the same logic to hardware, cloud, firmware |
| **Configuration Management / CMDB** | Configuration items (CIs) and their relationships | The *state* model; TLM adds the *time* model — CMDBs answer "what is deployed?", TLM answers "what is deployed, and how old is it?" |
| **IT Service Management (ITSM)** | Services and their operation (incident, problem, change) | TLM supplies the lifecycle context that ITSM events (e.g. a patch that cannot be applied) refer to |
| **Technology Portfolio Management** | Investment view across the application/infrastructure portfolio (see §11 for the "TPM" naming fight) | The portfolio *view*; TLM is one of the mechanisms that keeps the portfolio's age and risk data honest |
| **Enterprise Risk Management (ERM)** | Whole-enterprise risk governance (COSO, ISO 31000) | Consumes TLM's EOL/end-of-support risk registers as an input; see §4 |

### 1.3 Why the discipline exists — the three drivers

1. **Risk and security.** Software that passes its end-of-support date receives no security fixes. Unpatched, unsupported software is a standing vulnerability. The two canonical exhibits: the 2017 WannaCry outbreak propagated through unpatched Windows SMBv1 — and Microsoft took the unusual step of issuing an emergency out-of-band security update for the *out-of-support* Windows XP in May 2017, with published security-response guidance ✅ (Microsoft Security Response Center); and the 2017 Equifax breach, traced to an unpatched Apache Struts vulnerability (CVE-2017-5638) ✅ (US House Oversight Committee report, 2018). Together they are the standing argument for why "it still runs, therefore it is fine" is not a lifecycle policy.
2. **Financial efficiency.** Unmanaged estates carry shelfware (licences paid for and unused), duplicated capabilities, premium-priced "extended support" contracts that were never negotiated because the renewal date was not on anyone's radar, and last-minute "keep the lights on" projects that cost multiples of a planned upgrade. Portfolio-level lifecycle data is what turns technology spend from a sunk cost into a forecastable budget line.
3. **Regulatory and audit exposure.** In regulated sectors — banking above all — supervisors and auditors expect the board and senior management to see technology risk, and EOL/end-of-support exposure is now an explicit item in that visibility (MAS expectations in §5, audit expectations in §4). An estate that cannot produce a complete, dated, support-status view of its own assets fails the most basic due-diligence test.

### 1.4 The estate in scope

| Class | Examples | Lifecycle particularity |
|---|---|---|
| **Commercial off-the-shelf (COTS) software** | Databases, middleware, operating systems, office suites | Lifecycle is *set by the vendor's* support policy — see §3 |
| **Custom / in-house applications** | Core banking, payments, internal tooling | Lifecycle is set by the *owner's* maintenance commitments; internal "end of life" is a governance decision, not a vendor calendar |
| **Mainframe / legacy applications** | COBOL workloads on z/OS, CICS, IMS/DB2 | Decades-old code, vendor support still purchasable, skills retiring — see §6 |
| **SaaS and cloud services** | Managed databases, PaaS runtimes, SaaS apps | No on-prem "retire"; lifecycle is about *version/feature deprecations, pricing-model changes and exit rights* — the cloud vendor's roadmap replaces the EOL calendar |
| **Infrastructure hardware** | Servers, mainframe hardware, storage, network gear | Physical wear, manufacturer EOL/EOSL for parts and firmware — see §9 |
| **Firmware / embedded** | BIOS/UEFI, disk and array firmware, network-device firmware, ILO/BMC | Often the least tracked class and the one with real security consequences (no patching path after vendor EOL) |
| **Licences, contracts, entitlements** | Software licences, support contracts, maintenance renewals | The *paper* lifecycle that drives cost — managed under SAM practice |
| **Data** | Data retained by retired applications | Outlives the application; retention/destruction obligations drive decommissioning — see §10 |

### 1.5 The core loop

TLM collapses to a closed loop that the operating model in §7 industrialises:

1. **Know** — a complete inventory with identity, version, owner, location, and support-status (what is out there).
2. **Track** — every asset's position in the lifecycle and its vendor/key dates (what is changing and when).
3. **Govern** — gates that force a decision at each transition: acquire, upgrade, extend support, tolerate risk, or retire (who decides, with what evidence).
4. **Act** — execute the decision with change control, and **retire** properly so the loop does not leak (the phase everyone skips — §10).

**Cross-references:** the strategic layer that decides *which* assets deserve investment is [IT Strategy](../management/it_strategy_guide.md); the ERM framing that TLM feeds is in [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md); the estate-inventory and supply-chain companion for the software half is [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md). This guide does not re-derive those disciplines — it consumes them.

---

## 2. The Lifecycle Phases: Plan, Acquire, Deploy, Operate, Retire

### 2.1 The canonical five-phase model

Nearly every serious treatment of technology/IT lifecycle management — whatever its vocabulary — resolves into the same five moments. This guide uses **Plan → Acquire → Deploy → Operate → Retire** as its spine:

| Phase | What happens | Typical gates and artefacts | Dominant risk if skipped |
|---|---|---|---|
| **1. Plan** | Needs analysis, options (build/buy/rent), business case, architecture fit, total cost of ownership (TCO) including *the cost of eventually retiring it*, target lifecycle and support horizon, entry into the asset register as a "planned" item | Business case; architecture review; risk assessment; budget approval | Strategic: acquiring something that duplicates, cannot be supported, or outlives its vendor |
| **2. Acquire** | Procurement, licensing, contracting (support terms, EOL commitments, exit rights), security review, registration of licence/asset records | Contract review; vendor risk assessment; purchase order; asset-record creation with support-end dates captured *at day one* | Financial/legal: unbounded support costs, no exit rights, licences nobody can reconcile |
| **3. Deploy** | Build/configure/install, integration, testing, release/change control, handover to operations with runbooks, training, baselining of the configuration item in the CMDB | Change approval; test evidence; operational-readiness review; CMDB update | Operational: assets live outside the inventory from birth, so their lifecycle clock never starts |
| **4. Operate** | Run, monitor, maintain, patch, upgrade within version, renew support, track licence compliance, capacity manage — and **watch the vendor lifecycle calendar** so renewal/upgrade decisions are made early, not in a crisis | Patch and version policy (see §8); support-renewal reviews; periodic portfolio health reviews | The classic: an asset quietly passes end of support because no process was watching the date |
| **5. Retire** | Decommission, data handling (retention/destruction), licence/support cancellation, vendor exit, disposal — and a **post-retirement check** that nothing is still calling the thing | Retirement plan and approvals; data-retention sign-off; decommissioning evidence; register update to "retired" | Zombie assets: the retired application whose data, licence and connections live on for years (§10) |

The phases are a *loop at portfolio level*, not a one-way trip per asset: one application's "retire" frees budget and attention that a different plan cycle re-invests. A bank that only ever plans, acquires, deploys and operates — never retires — is a bank whose estate ages without a release valve (see §6 on the legacy reality).

### 2.2 How the industry frameworks frame the lifecycle

No single authority publishes *the* phase list — the models below use different units of analysis (services, processes, assets, systems, products) and different verbs. The mapping is honest rather than forced, and where a model genuinely resists the five-phase frame it is flagged ⚠:

| Framework / source | Its lifecycle machinery | Map onto Plan/…/Retire | Verified? |
|---|---|---|---|
| **ITIL v3 (2007, updated 2011 — Axelos/PeopleCert)** | Five service-lifecycle stages: Service Strategy → Service Design → Service Transition → Service Operation → Continual Service Improvement | Strategy≈Plan; Design/Transition≈Acquire+Deploy; Operation≈Operate; CSI spans all; **Retire has no dedicated stage** — it hides inside Transition/CSI ⚠ | ✅ stage names and 2011 refresh are standard ITIL v3 facts |
| **ITIL 4 (2019)** | Abandoned the lifecycle for a **Service Value Chain** of six activities: Plan, Improve, Engage, Design & Transition, Obtain/Build, Deliver & Support — plus a practices catalogue (ITAM and SAM among the practices) | "Plan" survives by name; Obtain/Build≈Acquire; Deliver & Support≈Operate; **retirement is not an activity** — it is handled inside practices ⚠ | ⚠ launched in 2019 (year certain from the industry record; exact month not re-verified against Axelos this pass); activity names are standard industry knowledge |
| **ISO/IEC 20000-1:2018** | Service-management system (SMS) requirements; processes over the service lifecycle with Plan-Do-Check-Act continual improvement; not a phase model | A *process* frame, not a *phase* frame — maps onto all five phases at once ⚠ | ⚠ standard exists; exact clause numbering not re-verified this pass |
| **ISO/IEC 19770-1:2017** — *IT asset management — Part 1: IT asset management systems — Requirements* (ISO/IEC JTC 1/SC 7) | A management-system standard — ISO's abstract describes it as a discipline-specific extension of ISO 55001 — setting requirements for an ITAM system that can be applied to **all types of IT assets** and to organisations of all types and sizes, managing assets **"throughout all stages of the life cycle"** | Plan/Operate/Retire coverage flows from the life-cycle scope the ITAM system must manage; the 19770 family (Part 2: software identification/SWID tags; Part 3: usage) supplies the identification mechanics under the system umbrella | ✅ title/scope verified on iso.org (standard 68531); clause-level detail is paywalled → clause claims ⚠ |
| **COBIT 2019 (ISACA)** | Governance and management objectives grouped in domains — the BAI domain (**Build, Acquire, Implement**) and the DSS domain (**Deliver, Service, Support**) carry the lifecycle weight; APO carries strategy/planning | BAI≈Acquire+Deploy; DSS≈Operate; APO≈Plan; **retirement sits inside BAI/DSS objectives rather than as a named stage** ⚠ | ⚠ domain letters standard; objective-level mapping is this guide's reading |
| **NIST system life cycle** | NIST SP 800-64 Rev. 2 (*Security Considerations in the System Development Life Cycle*, October 2008) framed the federal SDLC — initiation, development/acquisition, implementation, operation/maintenance, disposition — but NIST **withdrew SP 800-64 Rev. 2 in May 2019** (content out of date), so the five-phase SDLC vocabulary is now *historical*; current NIST life-cycle thinking runs through the Risk Management Framework (SP 800-37) and systems-engineering guidance (SP 800-160) | Initiation≈Plan; development/acquisition≈Acquire; implementation≈Deploy; operation/maintenance≈Operate; **disposition** was the federal word for Retire (historical vocabulary) | ✅ withdrawal verified (nist.gov news item, May 2019); currency of RMF/SP 800-160 as the successors not re-verified this pass → ⚠ |
| **Gartner-style portfolio management** | Application/technology portfolio analysis classifies holdings (e.g. invest/hold/divest/retire quadrants, TCO modelling, "technical debt" scoring) rather than prescribing phases | A *state* model layered on top of any phase model — tells you which assets to push through which phase | ⚠ practitioner-style; not a single citable standard |

**Reading the table.** Three genuine differences matter in practice:

1. **Service-lifecycle vs asset-lifecycle units.** ITIL and ISO 20000 manage *services*; ISO 19770 and CMDB/ITAM practice manage *assets*. A service outlives the assets that implement it (a payments service survives three middleware upgrades), so banks must run both clocks — asset support dates *and* service capability plans — and reconcile them at portfolio reviews.
2. **Where "retire" lives.** Only the asset-centric frames (ISO 19770 "disposal", NIST "disposition") name retirement as a first-class stage ⚠. The service-centric frames absorb retirement into transition/improvement practice — which is precisely why organisations that adopt only an ITSM vocabulary systematically under-manage decommissioning.
3. **v3 lifecycle vs ITIL 4 value chain.** Any 2019+ ITSM implementation inherits ITIL 4's value-chain language, but most vendor EOL/EOSL documentation and most audit questionnaires still speak the v3/asset vocabulary ("end of life", "end of support"). A TLM programme should treat Plan/…/Retire as its internal canonical model and map vendor and framework vocabularies onto it — this guide does exactly that in §3.

### 2.3 Phase boundaries are where the failures live

The classic lifecycle failures are not failures *within* a phase; they are failures *at the seams*:

- **Acquire→Deploy seam:** assets provisioned outside the official channel (a developer's shadow server, a SaaS tool expensed on a credit card) never enter the register, so every later phase for that asset is guesswork. Shadow IT is a TLM inventory problem before it is a security problem.
- **Deploy→Operate seam:** no operational handover means nobody owns patching, so the asset ages in the dark until an incident or an audit finds it.
- **Operate→Retire seam:** the most expensive seam of all. Applications in "operate" for years past their vendor support, kept alive by premium support and heroic staff, because no one made the retire decision while there was still budget and time to do it properly. §10 and the §12 worked example are devoted to this seam.

### 2.4 The discipline's golden rule

> **Every asset has an owner, a support status, and a known date when its current support arrangement ends — and no asset is allowed to reach that date without a recorded decision (upgrade, extend, accept-risk, or retire) made in advance.**

That rule is the entire guide in one sentence. Everything else — the registers, the gates, the tooling, the audit evidence — exists to make the rule mechanically true rather than aspirationally true.

**Cross-references:** version/change mechanics that keep the Operate phase honest: [Agent Versioning](ai_llm/agent_versioning_guide.md) for the software/agent side and the platform-lifecycle cluster ([OpenShift](secure_red_hat_openshift_guide.md), [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md), [DevStack / OpenStack](devstack_openstack_guide.md)) for infrastructure platforms that themselves follow release lifecycles.

## 3. The Vendor Lifecycle: EOL, EOSL, and the Support Ladder

### 3.1 The ladder and the vocabulary problem

For any product the organisation does not own outright (and many it thinks it does), the lifecycle clock is set by the **vendor's support policy**, not by the buyer. The generic ladder below is how the industry talks about it — but the very first lesson of §3 is that **every vendor names the rungs differently**, and a TLM programme must normalise them:

| Generic rung | What it means | Vendor vocabularies |
|---|---|---|
| **General availability (GA) / launch** | Product released; support clock starts | Same term everywhere ✅ |
| **Full / mainstream support** | All fixes, security updates, feature requests, design changes | Microsoft "Mainstream Support"; Oracle "Premier Support"; Red Hat "Full Support Phase"; Cisco "within support contract" |
| **Extended / maintenance support** | Limited fixes — typically security only, paid for some vendors | Microsoft "Extended Support"; Oracle "Extended Support" (fee); Red Hat "Maintenance Support Phase"; IBM "Extended Support" offering |
| **End of (mainstream/extended) support / last support date** | Date after which the vendor no longer provides the listed support | Microsoft "end of support"; Cisco "Last Date of Support (LDOS)"; Oracle "end of Premier/Extended Support" per release |
| **End of sale** | Can no longer be purchased | Cisco "End of Sale (EOS)"; Oracle/IBM fold into lifecycle milestones |
| **End of life / retirement** | Product withdrawn from the lifecycle; no new fixes, no new sales, often no support at all | Cisco "End of Life" = the whole process ending in obsolescence; IBM "withdrawn from support"; hardware vendors often say "End of Service Life" ⚠ (term common in hardware practice; not separately verified per-vendor this pass) |
| **Survival rungs (paid)** | Post-support lifelines the vendor sells | Microsoft Extended Security Updates (ESU); Oracle Sustaining Support; Red Hat Extended Life Cycle + Long-Life Add-on; IBM Sustaining Support; Cisco support-contract renewals capped at LDOS |

**⚠ Practice varies.** Two vendors can mean opposite things by "end of life": for Cisco, EOL is a *process* ending in obsolescence (definition below); for many software vendors "EOL" means "the vendor will stop supporting this release soon", with support still running meanwhile. Microsoft's formal vocabulary is "end of support", not "end of life". The same word maps to different rungs; a TLM register should store a *normalised* field ("support end date", "post-support lifeline available?") next to the vendor's own terms, never the vendor's terms alone.

### 3.2 Microsoft: Modern vs Fixed Lifecycle (✅ verified at learn.microsoft.com)

Microsoft runs two policies side by side — which products fall under which is published per product:

- **Fixed Lifecycle Policy** — a defined support timeline set **at product launch**: a minimum of **five years of Mainstream Support**, plus an **Extended Support** period for some products. Mainstream Support includes incident support, security updates, and the ability to request non-security updates/design changes; Extended Support adds **paid support and security updates only** — no warranty claims, design changes, or new features. Beyond end of support, security updates are available **only via the Extended Security Update (ESU) program** (paid, time-boxed); self-help online content stays available for a minimum of 12 months after end of support. Service-pack policy was folded into the Fixed Lifecycle Policy in February 2020.
- **Modern Lifecycle Policy** — for products "serviced and supported continuously". Support continues while the customer (1) stays current per published servicing/system requirements, (2) is licensed, and (3) the vendor still offers support. Notification rules are explicit: **minimum 30 days' notice** when the customer must act to avoid significant degradation; **minimum 12 months' notice prior to ending support when no successor product or service is offered** (excluding free/preview); for a select subset of Azure services Microsoft may give up to **three years'** notice of end of applicable support.

**Read for TLM:** fixed-lifecycle products have hard, published dates (plan upgrades against them); modern-lifecycle products have *servicing requirements* — the risk is not a date but "am I still on a serviced release?" Windows 10 is the fixed-lifecycle exemplar of the era: end of support was reached per Microsoft's published lifecycle ⚠ (the 14 October 2025 date is widely published and the announcement is listed on Microsoft's lifecycle hub, but the individual announcement page could not be re-extracted this pass) — and the follow-on lifeline is the paid ESU programme, the pattern to expect for every late-life Microsoft product a bank still runs.

### 3.3 Oracle: Premier → Extended → Sustaining (✅ verified at oracle.com/support/lifetime-support)

Oracle's Lifetime Support Policy is explicitly staged for "database to middleware to applications":

- **Premier Support** — "comprehensive maintenance and software upgrades … for five years from the general availability (GA) date" (Oracle Database, Fusion Middleware, Oracle Applications).
- **Extended Support** — additional maintenance and upgrades **for an additional fee**, letting customers lag releases deliberately.
- **Sustaining Support** — maintenance "for as long as you use" the software: online tools, upgrade rights, and **pre-existing fixes** — but the feature table shows no new software updates, security alerts/updates, critical patch updates, or new certifications: only "pre-existing" ones. Sustaining is a lifeline, not a development stream.

Hardware/OS get their own stages: Premier Support covers hardware and integrated software (firmware); Oracle operating systems and Oracle VM carry **ten years** of maintenance from GA. The applications line also runs **Applications Unlimited** — a commitment to offer Premier Support on the continuous-innovation releases of PeopleSoft, E-Business Suite, JD Edwards EnterpriseOne, Siebel and Hyperion **through at least 2037**.

**Read for TLM:** Oracle's model turns "end of support" into a *price ladder* — after Premier ends, the decision is pay-for-Extended, drop-to-Sustaining (no new fixes), or upgrade. Sustaining's "pre-existing fixes only" is precisely the trap for security: a vulnerability found after you drop to Sustaining gets no Oracle fix at any price.

### 3.4 Red Hat: ten-year, three-phase RHEL cycle (✅ verified at access.redhat.com)

Red Hat publishes a **ten-year life cycle across three production phases — Full Support, Maintenance Support, and an Extended Life Phase — followed by optional extended support**. The errata policy distinguishes security errata (RHSA) from bug-fix errata (RHBA), with explicit severity criteria (e.g. security errata cover Critical/Important/Moderate CVEs at CVSS ≥ 7, effective 1 April 2025). From **RHEL 9**, Red Hat unified its paid extensions into the **Extended Life Cycle (ELC)** — replacing the older EUS/EEUS/E4S offerings with a consistent six-year support window for eligible minor releases — plus renewable **Long-Life (LL) Add-on** annual increments, taking errata coverage to **14 years and beyond** on eligible releases. RHEL 8 and 7 sit on the older/lifecycle-phased tracks (Red Hat keeps a "Retired Life Cycle Dates" table; RHEL 7's extended-support end was not re-verified this pass ⚠).

**Read for TLM:** RHEL is the cleanest published *phase* model of the five vendors here (Full → Maintenance → Extended Life → paid extension), and the migration from "10-year cycle" to "10+4-year cycle with paid add-ons" shows the industry drift: base support shrinks in content over time while the *paid* tail lengthens — exactly why banks must budget the tail years at acquisition time, not discover them at year nine.

### 3.5 IBM: per-product lifecycle with Extended/Sustained offerings (✅ verified at ibm.com/support/lifecycle)

IBM's support-lifecycle site states the policy plainly: it "specifies how long support will be available for IBM products, from when the product is available for purchase to when it is no longer supported", and the lifecycle site publishes **per-product, per-version dates** with a search form, lifecycle news feed, and machine-readable XML downloads ("to import into your spreadsheet program or custom data processing application" — a TLM tool's dream data source). IBM also sells post-base-support offerings for Passport Advantage software — **Advanced Support, Extended Support and Sustaining Support** — and the search table now carries an "Extended/Sustained Support ends" column.

**Read for TLM:** IBM does **not** publish one universal support length the way Microsoft (5+ years) or Red Hat (10 years) do — support windows are product- and version-specific, which is why IBM publishes dates rather than rules. For mainframe software (z/OS, CICS, DB2, IMS) the lifecycle discipline is: look the version up, download the XML, and load the dates into the register — and note that mainframe *hardware* (IBM Z, Power, storage) runs on the same lifecycle site with "Eligible Services" listings.

### 3.6 Cisco: the EOL *process* (✅ verified at cisco.com EOL policy)

Cisco's End-of-Life Policy treats EOL as a process, not a date, with published milestones:

- **EOL Notification Date** — when the end-of-sale and EOL milestones are communicated publicly; external notification of end of sale is **typically six months before End of Sale**.
- **End of Sale (EOS)** — after this date the product can no longer be purchased (last customer ship for hardware: three months after hardware EOS).
- **Last Date of Support (LDOS)** — the last date support is available under active contracts; after it, "support is no longer available".
- Cisco's definition of **End of Life**: "a process that guides the final business operations associated with the … life cycle … once completed, make a Product obsolete. Once obsolete, the Product is not sold, improved, maintained, or supported."
- Post-EOS support entitlements for active contracts are specified (e.g. one year of routine hardware failure analysis; critical-bug fixes one year from EOS plus two years for OS software / one year for application software; TAC support three years for OS software, two for application software, five for hardware; five years of hardware replacement parts). The policy covers new EOL notifications on or after **29 September 2022**.

**Read for TLM:** Cisco gives the *network-hardware* pattern — long, contract-anchored tails (five years of parts/TAC after EOS) but a hard LDOS cliff. Network gear in a bank (firewalls, switches, load balancers) is EOL-tracked by part number and contract, which is a different register shape from software versions.

### 3.7 Vendors not verified this pass ⚠

- **VMware/Broadcom** — the VMware lifecycle site could not be reached/verified within budget this pass ⚠. Practice note: VMware product lifecycle (vSphere etc.) used published "general support"/"technical guidance"/"end of general support" phases; post-acquisition (Broadcom, November 2023) support and licensing terms changed materially and are exactly the kind of *vendor-roadmap risk* a TLM register must track — verify against current Broadcom pages before relying on any pre-2024 dates.
- **Open-source and community support windows** ⚠ — OSS projects set their own "end of security support" per release train (the concept exists everywhere — e.g. community-supported vs vendor-supported builds — but there is no single policy to cite); treat community EOL as *weaker* than commercial EOL because there is often no paid lifeline at all.

### 3.8 The cross-vendor comparison

| Dimension | Microsoft | Oracle | Red Hat (RHEL) | IBM | Cisco |
|---|---|---|---|---|---|
| Policy name | Fixed / Modern Lifecycle ✅ | Lifetime Support Policy ✅ | RHEL Life Cycle ✅ | Support Lifecycle ✅ | End-of-Life Policy ✅ |
| Base support length | ≥5 yrs Mainstream (Fixed) | 5 yrs Premier from GA (DB/FMW/apps) | 10 yrs across 3 phases | Per product/version | Per product; contract-based |
| Standard extension | Extended Support (some products) | Extended Support (paid) | Maintenance → Extended Life Phase | Extended Support offering | Renewals capped at LDOS |
| Lifeline after end | ESU (paid) | Sustaining (pre-existing fixes only) | ELC + Long-Life Add-on (paid, ≥14 yrs) | Sustaining Support | None past LDOS |
| Explicit published dates | ✅ lifecycle search | ✅ per product | ✅ per release | ✅ per version (+XML) | ✅ per product |
| Notice periods | 30 days / 12 months / up to 3 yrs (Modern) ⚠-varies | None standardised ⚠ | None standardised ⚠ | None standardised ⚠ | ~6 months EOS notice |

Every cell in the "notice periods" row except Microsoft's is marked ⚠ because **only Microsoft publishes fixed customer-notice minimums**; the other vendors publish lifecycle *dates* but no comparable notice commitment — which is precisely why a TLM register cannot rely on vendors to remind you.

**Read for the whole section:** EOL is not a single moment of "product dies". It is a *ladder of declining service*, priced and timed differently per vendor, with the general shape: full support → security-only support → paid lifeline → nothing. The TLM job is to know, for every product, **which rung it is on, when the next rung arrives, what the paid lifeline would cost, and who has decided whether to pay, upgrade, or retire** — that decision record is the EOL risk register of §4 and the worked example of §12.

**Cross-references:** the risk treatment of the ladder above is [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md); platform products following release lifecycles are covered in the platform cluster ([OpenShift](secure_red_hat_openshift_guide.md), [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md), [DevStack / OpenStack](devstack_openstack_guide.md)); versioning mechanics on the software side are in [Agent Versioning](ai_llm/agent_versioning_guide.md).

---

## 4. TLM as Technology Risk

### 4.1 The risk-classification home

In the enterprise risk taxonomy, lifecycle failures are **operational/technology risk**: the risk of loss from inadequate or failed internal processes, people, and systems — the standard operational-risk definition banks encode in their Basel/ICAAP machinery (see the ERM guide's treatment, which this section condenses rather than re-derives). TLM is one of the *control families* that keeps technology risk inside appetite: it is the discipline that makes "the estate is supported, patched, and current" a measured fact rather than a hope.

### 4.2 Lifecycle failure modes and the risks they become

| Lifecycle failure | Example | Risk materialisation |
|---|---|---|
| Product past end of support, still in production | Windows 10 fleet after EOS without ESU; RHEL release past maintenance | **Security** — no new fixes; every future CVE on that product is permanent (see §1.3's WannaCry/Equifax pattern) |
| Version drift inside support | Ten different middleware minor versions, three unserviced | **Availability/recoverability** — vendor support refused ("upgrade first"), incident resolution slow, DR restore from an unsupported version |
| Silently retired vendor features | Cloud service deprecated with 12-month notice (Microsoft Modern policy), consumption still embedded | **Operational resilience** — capability vanishes on the vendor's calendar, not yours |
| Unmanaged retirement | Application decommissioned, data kept on unpatched legacy storage | **Data/privacy + security** — retained data outside retention policy, on an asset nobody patches |
| Ghost licences / shelfware | Licences renewed yearly for a retired product (the §2 "Retire" phase skipped) | **Financial** — spend with no asset behind it; also **compliance** exposure in a vendor audit |
| Shadow assets | Server or SaaS tool provisioned outside the official channel | **All of the above, unmeasurable** — the asset is off every register, so its risk is invisible until it fails |

### 4.3 The three lines and the board

TLM maps onto the standard three-lines model that the ERM guide details:

1. **First line — asset owners and operations:** run the lifecycle processes (patch, renew, upgrade); own the day-to-day "is my product supported?" question.
2. **Second line — technology risk management:** set the TLM policy and standards, maintain the EOL risk register, review risk acceptances, and report aggregate exposure (e.g. "% of servers past end of support", "applications with no support contract") to management risk committees.
3. **Third line — internal audit:** independently test that the inventory is complete, that the register is current, and that acceptances were actually signed by someone with authority. Audit is where "we think we're fine" meets "show me the date".

Board and senior management sit above all three: technology-risk appetite is a board-level artefact (the MAS TRMG makes board/senior-management ownership explicit — §5), and the classic board question is *"which of our critical systems is running on unsupported software, and what are we doing about it?"* A TLM programme exists to make that question answerable in an afternoon, with evidence.

### 4.4 The EOL risk register

The operational heart of TLM-as-risk is a **risk register of lifecycle items**, not just a list of assets. Each entry carries:

| Field | Purpose |
|---|---|
| Asset/version identification | What exactly is at risk (product, version, platform, owner) |
| Support status + dates | Current rung and the date of the next rung (vendor-sourced, §3) |
| Criticality | Is this a critical business service? (MAS-criticality language, §5) |
| Exposure | What breaks if support ends: unpatched CVEs, no vendor recourse, audit findings |
| Mitigation options + cost | Upgrade, migrate, pay the lifeline (ESU/Sustaining/ELS), accept, retire |
| Decision + owner + expiry | The recorded acceptance with a **time-box** — acceptances must expire and be re-reviewed, or "accepted risk" quietly becomes permanent risk |
| Review cadence | Quarterly for EOL-in-12-months items; annually otherwise |

The register's two invariants: (1) **no item reaches its support-end date without a decision recorded before that date** (the golden rule of §2.4 restated), and (2) **every acceptance has an expiry** — risk acceptance is a renewable decision, never a one-time pardon. §12 builds a concrete register.

### 4.5 What auditors and supervisors look for

Internal audit over the IT estate (and, in Singapore, the audit expectations that regulators attach to the TRMG and to audit-committee accountability) converges on the same evidence set:

1. A **complete, current inventory** — auditors sample backwards from the CMDB/register to the network and find the shadow assets.
2. **Patch/version compliance evidence** — supported versions deployed, patch SLAs met, exceptions approved.
3. An **EOL/EOSL register with decisions** — every out-of-support item mapped to a signed, expiring acceptance or a remediation plan.
4. **Decommissioning records** — proof that retired assets were actually removed, data was handled per retention policy, and access was cut (see §10).
5. **Board/committee visibility** — the EOL exposure appears in technology-risk reporting that reaches the board or board-level committee.

**Cross-references:** the framework language (three lines, appetite, RAG) is [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md); the regulatory expectation side is §5 and [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md); the vendor-risk overlay (third-party lifecycle, exit) is [Vendor Management](../management/vendor_management_guide.md).

---

## 5. The Regulatory Angle: MAS Technology-Risk Expectations

### 5.1 The stack for a Singapore bank

MAS regulates technology risk through a layered stack. For a bank the current architecture (verified this pass against mas.gov.sg and the repo's MAS guide, which itself records verified mas.gov.sg extracts) is:

1. **Guidelines on Risk Management Practices – Technology Risk (the TRM Guidelines, "TRMG")** — the guideline layer. Current text **published 18 January 2021** ✅ (verified on the mas.gov.sg page), revising the original 2013 TRMG; described by MAS as risk-management principles and best-practice standards guiding financial institutions in managing technology risk. Applies to a very wide perimeter — roughly 60 institution types including every class of bank, merchant banks, finance companies, insurers, CMS licensees, payment institutions and trust companies ✅ (the mas.gov.sg "applies to" list).
2. **FSM-series notices under the Financial Services and Markets Act 2022** — the binding layer since the **10 May 2024** harmonisation ✅: the sectoral TRM notices (for banks, Notice 644/644A and its siblings across sectors) were all cancelled with effect from 10 May 2024 and replaced by harmonised notices, of which the bank notice is **FSM-N05** (with FSM-N22 covering cyber hygiene and siblings for other sectors). The notices set "requirements for a high level of reliability, availability and recoverability of critical IT systems" and IT controls to protect customer information (repo-guide verified wording ✅).
3. **Incident-reporting instruments** — the Instructions on Incident Notification and Reporting to MAS and the December 2025 circular moving reportable-incident submissions onto the MAS-Tx platform (from 1 February 2026), per the repo's MAS guide ✅.

The repo's MAS guide is the authoritative deep-dive on this stack; §5 condenses only what TLM must know. **Cross-reference:** [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) §3–§4 for the notice map and the TRMG treatment, and the risk-side companion [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md).

### 5.2 What the TRMG expects that TLM serves

The 2021 TRMG revision, per the repo guide's verified reading and contemporaneous industry summaries, spans:

- **Technology risk governance and oversight** — board and senior-management ownership of technology risk, explicit risk appetite, and three-lines-of-defence operation. TLM feeds this: the EOL exposure and support-status metrics are board-reportable technology-risk indicators.
- **IT resilience** — system availability and recoverability expectations for critical systems. An unsupported OS or database is a direct resilience exposure: it cannot be patched, its failure cannot be fully remediated, and its recovery (DR from a frozen, unsupported version) is fragile. Industry summaries of the 2021 revision (PwC, Kroll — ⚠ secondary sources) note new sections on cyber security assessment and cyber surveillance/security operations; the repo guide flags the 2013-origin date as documented-but-not-directly-confirmed ⚠.
- **Cyber resilience** — security controls, detection and response; the companion cyber-hygiene notice (FSM-N22 for capital-markets entities per the repo's notice map ✅) covers securing administrative accounts, security patching, and baseline standards — patching being the TLM-adjacent control par excellence.

MAS does **not** publish a rule that literally says "thou shalt maintain an end-of-support register" ⚠ — that requirement does not exist under that name. The EOL expectation is *derived*: governance (you must know and own your technology risk), resilience (unsupported systems cannot meet availability/recoverability expectations), and cyber hygiene (you must patch; you cannot patch what the vendor no longer fixes). A bank that runs a critical system on unsupported software and cannot show a documented, owned decision about it is, in supervisory practice, carrying a technology-risk finding — which is why the §12 worked example treats the EOL register as the *evidence artefact* of MAS-facing technology-risk management.

### 5.3 The outsourcing overlay

Where lifecycle risk sits with a third party (a hosted core, a SaaS channel platform, a cloud provider), MAS's **Outsourcing Guidelines (published 11 December 2023, effective 11 December 2024)** and **Notices 658 (banks) / 1121 (merchant banks)** require lifecycle-style management of the *outsourcing arrangement itself*: materiality assessment, risk management across the arrangement's full lifecycle, notification of material outsourcing, and **exit plans**. The vendor's product lifecycle (EOL of the SaaS version you run, end of support for the on-prem product the vendor hosts) becomes third-party risk that the bank must oversee — condensed here; the full treatment is [Vendor Management](../management/vendor_management_guide.md) and the MAS guide §4.4.

### 5.4 Audit visibility expectations

The TRMG-era supervisory expectation, reinforced by the BCM Guidelines (2022) audit-cycle language and the incident-reporting circular, is that **audit covers technology risk on a cycle** and that the board-level committee sees the material findings. For TLM the practical reading: the EOL register, the patch-compliance metrics, and the decommissioning records are exactly the artefacts a MAS-supervised bank's internal audit will request when testing technology-risk governance — so they must exist, be current, and be *owned* (named accountable roles, per the accountability expectations the repo's MAS guide records as the IAC theme ⚠). §4.5 lists the evidence set.

**Cross-references:** [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) (the full instrument map — notices, TRMG, BCM, outsourcing); [Core Banking Systems](../banking/core_banking_systems_guide.md) and [Coreless Banking](../banking/coreless_banking_guide.md) for the estate those notices apply to; [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) for the three-lines and appetite machinery.

## 6. The Legacy Reality: Mainframes, COBOL, and the Long Tail

### 6.1 The estate that will not die — and why

Banking's hardest TLM problem is not the shiny new stack; it is the **core systems built between the 1970s and 1990s** that still run the bank. The repo's core-banking research summarises the reality: the oldest living cores run **COBOL on IBM mainframes (z/OS, with CICS or IMS transaction managers and DB2 or VSAM storage)**; they were built in the 1970s–1990s, have been extended continuously for decades, and still run "a startling share of the world's banking — most large tier-1 banks in North America, Europe, and Japan process their core retail banking on mainframe platforms" (cross-ref, condensed from [Core Banking Systems](../banking/core_banking_systems_guide.md) §3.2, which this guide does not re-derive). The companion [Coreless Banking](../banking/coreless_banking_guide.md) guide covers the architecture debate about what replaces them.

For TLM the mainframe estate is distinctive in five ways:

1. **The vendor clock still runs — and it is generous.** IBM publishes per-version lifecycle dates for z/OS and its subsystem software (CICS, IMS, DB2) on the IBM support-lifecycle site verified in §3.5, including machine-readable feeds. Mainframe software has historically had very long, well-published support windows — the discipline problem is not "no dates" but "dates nobody loaded into a register".
2. **The code has no vendor clock at all.** COBOL written in-house in 1985 has no end-of-support date; its "support" is the maintenance team's knowledge, the remaining COBOL skills in the market, and the organisation's willingness to keep paying for it. Internal custom code is on an *owner-set* lifecycle (the §1.4 table), which means EOL is a governance decision, not a calendar event — and governance decisions are easier to postpone forever.
3. **The risk profile is inverted.** A mainframe running a 1990s COBOL application on a current, vendor-supported z/OS version may be *lower* risk than the same logic re-hosted badly — but a mainframe on an outdated z/OS release, or an application whose only "support" is three soon-to-retire programmers, is a genuine EOL risk that no licence-renewal process will ever surface, because there is no vendor sending the renewal invoice.
4. **The skills clock is the real EOL date.** COBOL/mainframe skills are widely reported as scarce and ageing ⚠ (the "retiring mainframe workforce" is a constant of industry commentary, but a citable, current statistic was not verified this pass — treat any specific percentage as unverified ❌). TLM must track *skill availability* as a lifecycle input alongside vendor dates: when the last people who can change the code retire, the asset reaches effective end-of-life regardless of what IBM supports.
5. **Retirement is technically and politically huge.** A core-banking mainframe cannot be switched off like a server. The repo's core-banking research notes the practitioner rule that **data migration is 60–70% of a core-replacement programme's effort** ⚠ (rule of thumb from the sibling guide, not a sourced statistic), and that cutting over without a dual-run period is how banks lose money. §12 works a smaller-scale but complete mainframe-retirement case (a COBOL workload, not the whole core).

### 6.2 The long tail beyond the mainframe

The mainframe is the visible peak; beneath it sits the **long tail**: thousands of small applications, Access-database "systems", Excel-based processes, ageing client-server apps, and endpoint fleets. TLM research across industries consistently finds that a minority of applications carry the majority of business value while the long tail consumes disproportionate support effort and risk ⚠ (practitioner consensus — e.g. portfolio "technical debt" and rationalisation studies — rather than one citable statistic). For a bank the tail matters because:

- Tail applications quietly accumulate on **unsupported runtimes** (a 2010 .NET Framework version, an OS past EOS) — each one a §4 risk-register entry.
- Tail data is where **data-retention breaches** live: an old system holding customer data past policy, with no owner and no retirement plan.
- Tail rationalisation is the *cheapest* TLM win: the portfolio review that retires fifty tail apps funds the mainframe programme.

### 6.3 The architectural responses and their lifecycle shape

The sibling guides carry the detail; the TLM-relevant lifecycle shapes are:

| Response | Lifecycle shape | TLM consequence |
|---|---|---|
| **Keep + maintain** (stay on mainframe, modernise incrementally) | Vendor dates (IBM) + skills clock + annual maintenance investment decision | The honest option when the core works; needs the §4 register discipline and a board-visible "we choose to stay" record |
| **Re-platform / re-host** (COBOL to new hardware/OS, e.g. cloud) | One-time migration project then a *new* lifecycle for the re-hosted code | Cheapest migration; the COBOL logic survives, so the skills/change problem partly survives ⚠ |
| **Re-write / extract** (to Java/microservices/cloud-native) | Full new lifecycle on modern platforms (see the platform cluster) | Highest cost/risk; the §12 case study |
| **Replace with vendor package / SaaS core** | Vendor lifecycle (the §3 ladder) + outsourcing lifecycle (Notice 658/1121, §5.3) | Swaps a skills problem for a vendor-management problem — see [Core Banking Systems](../banking/core_banking_systems_guide.md) and [Vendor Management](../management/vendor_management_guide.md) |
| **Retire the capability** (product/business line ended) | Pure decommissioning (§10) | Often overlooked as an option; sometimes the *right* business answer |

**Cross-references:** [Core Banking Systems](../banking/core_banking_systems_guide.md) and [Coreless Banking](../banking/coreless_banking_guide.md) own the architecture and modernisation analysis; [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) owns the software-supply-chain inventory problem for COBOL estates; §12 executes a retirement end-to-end.

---

## 7. The TLM Operating Model: Inventory, Tracking, Governance Gates

### 7.1 The three pillars

A TLM programme rests on three pillars, each with a practice pattern that recurs across the ITAM/SAM/portfolio literature ⚠ (these are practitioner-consensus patterns, verified in shape against the ISO 19770 ITAM-system requirements and the portfolio-management sources of §11, not against one canonical textbook):

**Pillar 1 — A complete, attributed inventory.** The §1.5 "know". Every asset with: unique identifier; product/version (for software) or model/part (for hardware); owner and support contact; business service and criticality; location (data centre/cloud/endpoint); acquisition date and cost; licence/support contract reference; **vendor support status and key dates** (current rung, next rung date, lifeline availability). ISO/IEC 19770-1:2017's ITAM-system requirements are the closest thing to a standard for *what the inventory process must be able to prove* (✅ title/scope verified; see §2.2). Completeness is an *enforcement* problem: the inventory is only as good as the acquisition and provisioning controls that feed it (any asset that can enter the estate without creating an inventory record will do so — the §2.3 seam failures).

**Pillar 2 — Lifecycle tracking with a clock.** The §1.5 "track". Three clocks per asset: (a) the **vendor clock** (support dates from §3 sources — Microsoft lifecycle export, Oracle policy PDFs, Red Hat tables, IBM XML feeds, Cisco EOL notices); (b) the **internal clock** (maintenance commitments for custom code, skill availability for legacy platforms — §6.1); (c) the **contract clock** (support/maintenance/cloud-term renewal dates). The tracking process emits *alerts at defined look-aheads*: e.g. T-24 months (planning starts), T-12 months (decision required), T-6 months (decision recorded or escalation), T-0 (no asset reaches the date without a decision — the golden rule).

**Pillar 3 — Governance gates.** The §1.5 "govern". Formal decision points at each lifecycle transition, each with evidence and an accountable role:

| Gate | Trigger | Decision | Accountable |
|---|---|---|---|
| **Acquisition gate** | New asset requested (Plan→Acquire) | Approve/reject; set support horizon, exit rights, TCO incl. retirement cost | Architecture + procurement + risk |
| **Entry gate** | Asset deployed (Acquire→Deploy) | Inventory record complete and correct before production traffic | Asset owner + release mgmt |
| **Renewal gate** | Support/contract renewal approaching | Renew, renegotiate, downgrade, or begin exit | Vendor/contract owner + finance |
| **Version gate** | New vendor version or end of minor-release support | Upgrade now, schedule upgrade, or accept risk (time-boxed) | Asset owner + technology risk |
| **EOL gate** | Vendor end-of-support within look-ahead window | Upgrade, pay lifeline (ESU/Sustaining/ELS/LL), accept (signed, expiring), or **retire** | Technology risk + asset owner (+ board-level for critical) |
| **Retirement gate** | Retire decision taken | Decommission plan approved: data handling, dependency cut, evidence, register closure | Asset owner + data owner + risk + audit |

The gates are where "asset management" becomes "**lifecycle management**": each gate forces a dated, owned decision, and the set of gate records *is* the governance evidence that audit and MAS-facing reporting consume (§4.5).

### 7.2 Roles and the operating rhythm

- **Roles:** asset owners (first line), a TLM/SAM function (second line: register maintenance, reporting, gate secretariat), technology risk (policy, acceptance authority for risk items), procurement/finance (renewal and TCO data), and internal audit (independent testing). For critical systems, the EOL acceptance escalates to a senior management committee or the board-level technology committee — the MAS TRMG governance expectations of §5 make board/senior-ownership of technology risk the supervisory baseline.
- **Rhythm:** continuous data feeds (discovery, vendor feeds) with *periodic decision points*: monthly renewal/alert reviews, quarterly EOL-register reviews and risk-acceptance refreshes, annual portfolio health review (which feeds the IT-strategy planning cycle — cross-ref [IT Strategy](../management/it_strategy_guide.md)), and audit-cycle evidence refreshes. The rhythm matters more than the tooling: a register reviewed quarterly beats a perfect CMDB reviewed never.
- **Metrics that management committees actually use:** % of critical assets past support end (must be ~0 or fully risk-accepted); % of assets within 12 months of support end; licence-compliance position; applications with no named owner; technical-debt and age distributions; retirement throughput (assets retired per quarter — the metric that proves the "retire" phase is real).

### 7.3 Maturity: from register to programme

Practice patterns ⚠ (shape only): **Level 1** — ad hoc spreadsheets, discovered in crises; **Level 2** — a central register with owners and vendor dates, quarterly review; **Level 3** — automated discovery + vendor feeds, gates enforced in change/procurement process, EOL acceptances tracked with expiries; **Level 4** — lifecycle data drives planning (renewal budgets forecast years ahead, retirements scheduled, portfolio risk reported to the board monthly). Most banks operate between 2 and 3 on the run-of-the-mill estate and at 1 for the long tail — the §12 worked example builds a Level 3 programme from a Level 1 starting point.

**Cross-references:** the strategy layer that sets *what the portfolio should be* is [IT Strategy](../management/it_strategy_guide.md); the vendor-side contract machinery is [Vendor Management](../management/vendor_management_guide.md); the risk-reporting destination is [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md); the tooling options are §11.

---

## 8. The Software Lifecycle: Versions, Patches, Supply Chain

### 8.1 The software estate has three lifecycles at once

For every piece of software, TLM must hold three clocks that are *not* the same thing:

1. **The product lifecycle** — the vendor's support ladder for the product/version (§3). This is what EOL means.
2. **The deployment lifecycle** — the version *you actually run*, which lags the vendor's current version; the lag is the "version drift" that support departments measure. Patch/update discipline collapses the lag.
3. **The supply-chain lifecycle** — the *components* inside the software (libraries, frameworks, open-source packages), each with its own versions and vulnerabilities. This is the SBOM layer, and it is where modern software risk actually lives (Log4j-class events). The repo's [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) guide documents how hard this layer is for the languages that run banking cores; [Agent Versioning](ai_llm/agent_versioning_guide.md) extends the versioning discipline to AI agents and models.

The three-clock view explains the common failure: an organisation that tracks *only* the product lifecycle (renewal invoices) gets blindsided by a vulnerability in a third-party library inside a product that is itself fully supported — or by a drift between the supported product version and the unserviced minor version actually deployed (the §3.2 "stay current" requirement of Microsoft's Modern policy, and the service-pack rules of the Fixed policy, are vendor-side expressions of the same idea).

### 8.2 Version and patch management as lifecycle control

Patch management is the Operate-phase execution of lifecycle policy:

- **Patch policy** should be version-aware: "stay on vendor-supported versions; apply security patches within SLA X; major-version upgrades are lifecycle projects, not patches". The MAS cyber-hygiene notice (FSM-N22, §5) and the TRMG's cyber-resilience expectations make security patching a *regulatory* baseline for Singapore banks ✅ (notice existence and content verified in the repo MAS guide).
- **The EOL interaction:** patching a product past end of support is impossible by definition (no new fixes — §3.3's Sustaining "pre-existing fixes only" is the sharpest example). So patch compliance and EOL exposure are the same risk in two clothes: an unpatched in-support product is a *process* failure; an unpatched out-of-support product is a *lifecycle decision* failure. The register of §4.4 distinguishes them.
- **Version management mechanics** (semantic versioning, release trains, lockfiles, container-image tagging, rollback) belong to the engineering guides; TLM's contribution is the *policy and the evidence*: which versions are approved, who approves exceptions, and how version data flows into the inventory automatically rather than by survey. [Agent Versioning](ai_llm/agent_versioning_guide.md) covers the analogous problem where the "software" is an AI agent whose behaviour changes with model, prompt, and tool versions — a lifecycle register for agents needs model-version and behaviour-baseline fields, not just build numbers.

### 8.3 SBOMs and the supply-chain clock

The SBOM sibling guide is the deep treatment; TLM's condensed obligations are: (1) require an SBOM at acquisition for all third-party software (make it a procurement gate — §7.1); (2) reconcile SBOMs against vulnerability feeds on a cadence; (3) treat *component* end-of-life (an abandoned open-source library) as lifecycle risk even when the product containing it is supported; (4) for COBOL/Pro\*C/C estates, accept that SBOM generation is hard and budget the bespoke tooling the sibling guide describes. US federal and EU regulatory pushes (EO 14028/OMB M-22-18/NIST SP 800-218; the EU Cyber Resilience Act) are documented in the sibling guide ✅ — cross-ref [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md).

### 8.4 SaaS and cloud software: lifecycle without a "retire"

SaaS/cloud products invert the model: there is no on-prem asset and no vendor "end of support" in the classic sense; instead the vendor *deprecates features and versions on its own roadmap*, and the customer must track: feature/API deprecation notices, forced version upgrades (Microsoft Modern-policy products: 30-day/12-month notice minimums — §3.2), pricing-model changes, and *exit* (data export, API freeze dates). The lifecycle artefact is the **deprecation register** — every SaaS feature/API the bank depends on, with its announced end date and the bank's migration — and the exit plan the outsourcing regime already demands for material services (§5.3). ⚠ SaaS deprecation-notice norms vary by vendor and are not governed by any standard verified this pass; the Microsoft notice minimums are the only fixed rule found (✅ §3.2).

**Cross-references:** [Agent Versioning](ai_llm/agent_versioning_guide.md) (version discipline for agents/software); [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) (supply-chain inventory); [Vendor Management](../management/vendor_management_guide.md) (SaaS contract and exit lifecycle); the platform cluster for the infrastructure software that hosts all of it ([OpenShift](secure_red_hat_openshift_guide.md), [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md), [DevStack / OpenStack](devstack_openstack_guide.md)).

---

## 9. The Hardware Lifecycle: Procurement to Decommissioning

### 9.1 Hardware is a lifecycle, not a purchase

Hardware TLM tracks the physical asset through: procurement → install → warranty → **maintenance/support contracts** → manufacturer EOL (parts and firmware stop) → decommissioning → disposal. The vendor-clock mechanics of §3 apply, with hardware-specific wrinkles:

- **The manufacturer clock is about parts and firmware, not "the server"**: after manufacturer end-of-life, replacement parts and firmware updates stop. Cisco's policy (§3.6) is the clearest published model: post-EOS support entitlements (failure analysis, TAC, replacement parts for defined years) and a hard **Last Date of Support** after which "support is no longer available". IBM publishes hardware lifecycle and "Eligible Services" on the same lifecycle site as software (§3.5). Oracle's hardware Premier Support covers hardware and integrated software incl. firmware, with ten-year maintenance for Oracle OS/VM products (§3.3) ✅.
- **Third-party maintenance (TPM — here it genuinely is "third-party maintenance"!)** — a naming collision flagged honestly: in hardware practice, **TPM overwhelmingly means third-party maintenance** (post-manufacturer support by independents such as the specialists that keep old mainframes and storage alive), a different meaning again from §11's "technology portfolio management" reading. The acronym is overloaded across contexts ⚠; a TLM discussion must state which TPM it means. This guide uses "TPM" for technology/IT portfolio management in §11 (the reading the task brief asked to verify) and spells out "third-party maintenance" here in full to avoid ambiguity.
- **Firmware is the forgotten layer**: disk/array firmware, network-device firmware, BMC/iLO — each has its own version and support status, often *after* the manufacturer has EOL'd the device that hosts it. Firmware CVEs with no fix path are a pure lifecycle exposure (§1.4 table). Server, storage and network vendors publish firmware advisories and lifecycle dates ⚠ (per-vendor firmware lifecycle pages were not individually verified this pass).
- **End-of-service-life vocabulary** ⚠: hardware vendors frequently use "end of service life" (EOSL) rather than "end of life" for the date after which the manufacturer will no longer service/repair a product; the term is common in hardware practice (server/storage vendors, and in bank procurement language) but was not verified against each manufacturer's policy page this pass — treat EOSL as manufacturer-specific terminology and read the actual policy.

### 9.2 The hardware register and its decision points

The hardware register fields (§7.1) plus: serial number and warranty end; maintenance contract level and end (next-day? 4-hour? parts+labour?); manufacturer EOL/EOSL dates for the *model* and for the *firmware line*; criticality and redundancy (is there a supported spare?); location and lease/ownership. Decision points: **warranty→contract** (buy maintenance or self-insure with spares), **contract renewal** (renew, downgrade, replace), **manufacturer EOL** (replace before parts/firmware dry up, or accept the risk on non-critical gear with spares on the shelf), **decommission** (§10: data erasure is mandatory before disposal — disk/SSD destruction or certified erase, per the bank's data-protection policy; ⚠ specific erasure standards, e.g. NIST SP 800-88, exist and are widely used in practice but were not re-verified this pass).

### 9.3 Cloud and the disappearing hardware lifecycle

Cloud shifts hardware lifecycle to the provider: the customer's hardware register shrinks to *logical* assets (VMs, managed services, reserved instances, storage tiers) whose lifecycle is: provision → rightsize → (deprecation by the provider — see §8.4) → deprovision. The TLM-relevant practices: tag everything at creation (so the inventory is born complete — the §2.3 seam fix); track reserved-instance/commit expiry (the *financial* renewal clock, the cloud analogue of a maintenance contract); watch provider deprecations of VM families, OS images, and API versions; and manage *capacity retirement* (deleting the un-used test environments that quietly consume budget — the cloud version of zombie assets). ⚠ Cloud provider deprecation-notice norms vary (Microsoft publishes the fixed minimums of §3.2 for Azure; other providers' practices were not verified this pass).

**Cross-references:** the platform-lifecycle cluster for what runs on the hardware/cloud ([OpenShift](secure_red_hat_openshift_guide.md), [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md), [DevStack / OpenStack](devstack_openstack_guide.md)); vendor-exit practice in [Vendor Management](../management/vendor_management_guide.md); decommissioning in §10.

## 10. Retirement and Decommissioning Done Right

### 10.1 Why retirement is the phase everyone skips

The retire phase is where TLM programmes quietly fail. Industry post-mortems and audit findings converge on the same story ⚠ (practitioner-consensus pattern): applications are "retired" in name — the business stops using them — but the *asset* never dies: the server stays powered for "just in case", the data stays on disk, the licence keeps renewing, the interfaces stay open, and the CMDB still lists it as production. Zombie assets are not harmless; they are the §4.2 failure table in one object: unpatched (nobody owns them), full of retained customer data (privacy breach waiting), licence-expensive, and invisible to risk reporting because they are *nobody's* responsibility.

### 10.2 The decommissioning framework

A defensible retirement has seven workstreams, each with evidence:

1. **Decision and scope.** The retirement gate (§7.1): business sign-off that the capability is no longer needed (or is replaced), the asset owner is accountable, and the target date is set. Scope: which applications, which data, which interfaces, which infrastructure.
2. **Discovery and dependency mapping.** Find everything attached to the asset: upstream/downstream interfaces, batch jobs and cron entries, file transfers, database links, report consumers, monitoring hooks, security exceptions referencing it, and — the classic — the *other* application that still calls it once a month. The repo's core-banking sibling notes the practitioner rule that **data migration is 60–70% of a core-replacement programme's effort** (⚠ rule of thumb, §6.1); dependency discovery is the same truth at smaller scale — most decommissioning overruns are undiscovered dependents.
3. **Data handling.** The legal heart of retirement. For each dataset: (a) does a **retention obligation** apply (banking records, transaction data, tax/regulatory records — retention periods are set by the bank's regulatory and legal obligations; the repo's MAS guide records the record-keeping expectations that attach to the FSMA/Banking Act stack ✅-cross-ref); (b) is there a **legal hold** (litigation, investigation — holds override retention schedules); (c) what must be **archived** (to the record-keeping system or cold storage, with access controls and a defined owner) versus **destroyed** (per the retention schedule, with certification of destruction); (d) is customer data handled under the bank's privacy and secrecy obligations (banking-secrecy and data-protection duties apply to the *data*, not the dead system). Every dataset gets one of: archive-and-retain, destroy-and-certify, or transfer-to-successor — and the register records which, with who approved it.
4. **Application shutdown and infrastructure decommission.** Cut the interfaces in dependency order (dependents first), run the observation period, then shut down and **physically remove** — decommissioned infrastructure is removed from power and network, storage is erased or destroyed (certified erasure or physical destruction per the bank's policy — §9.2), cloud resources are deleted with evidence, not just "disabled".
5. **Vendor and licence exit.** Terminate support/maintenance contracts, surrender licences (or transfer them where permitted), cancel SaaS subscriptions, close vendor accounts, and *confirm in writing* that the vendor has destroyed/returned data per the contract and the outsourcing exit plan (§5.3). Contract termination is a process with notice periods — it must start *before* the shutdown, not after. Full treatment: [Vendor Management](../management/vendor_management_guide.md).
6. **Governance sign-off and records.** The retirement file: decision record, dependency map, data-handling evidence (archive manifests, destruction certificates), vendor-exit confirmations, change records, and the register update closing the asset. This file is what audit and MAS-facing review will ask for (§4.5) — without it, the retirement did not happen.
7. **Post-retirement verification.** After 3–6 months, verify the zombie didn't resurrect: no traffic, no logins, no batch runs, no licence renewals, no helpdesk tickets. The verification finding closes the loop and feeds the "retirement throughput" metric (§7.2).

### 10.3 Pitfalls

- **The observation period is skipped** — a dependent that runs quarterly surfaces six months later, and the "retired" system is hurriedly rebooted from a backup, *now* with no support contract and no owner.
- **Data is archived but unowned** — the archive needs a named owner and access control, or "archived" becomes "lost customer data with an open pipe".
- **Licences outlive the asset** — the renewal process (§7.1 renewal gate) must receive the retirement record, or the ghost licence renews for years (shelfware in its purest form).
- **"Retired" in the CMDB but alive on the network** — only infrastructure removal, with evidence, closes the risk; a status change is not a decommission.

**Cross-references:** [Vendor Management](../management/vendor_management_guide.md) (exit and contract termination); [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) (record-keeping/regulatory context); [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) (the risk-acceptance machinery); §12 executes the whole framework.

---

## 11. Tooling: CMDB, SAM, and the TPM Question

### 11.1 The tool stack, honestly labelled

TLM tooling is a market, not a standard ⚠ — every claim below about specific products is a market observation, not a verified product benchmark (product capabilities were not independently tested this pass):

| Layer | What it does | Market examples (⚠) | Lifecycle gap it leaves |
|---|---|---|---|
| **Discovery / CMDB** | Find assets and record configuration items and relationships | Enterprise CMDB suites (ServiceNow-class), discovery agents, cloud asset tools | Records *state*, not *time* — CMDBs rarely hold vendor support dates or compute EOL exposure by themselves |
| **Software Asset Management (SAM) tools** | Licence reconciliation, entitlement, usage, compliance (ISO 19770 practice) | Dedicated SAM suites and ITSM-platform SAM modules | Scope to software licences; hardware/cloud/firmware need other layers |
| **ITAM suites** | Broaden SAM to all IT assets | Platform ITAM modules | Often still a register, not a decision engine |
| **Portfolio / enterprise-architecture tools** | Application and technology portfolio maps, lifecycle/roadmap views, technical-debt scoring | Architecture/portfolio platforms | The *analysis* layer — depends on good inventory below |
| **Vendor lifecycle feeds** | The dates themselves | Microsoft lifecycle export; IBM lifecycle XML; vendor EOL notices (Cisco), policy PDFs (Oracle, Red Hat) — all verified reachable in §3 ✅ | Vendor-specific; need normalising into one register (the §3.1 vocabulary problem) |
| **Spreadsheets** | Where every TLM programme starts | — | Fine at Level 1 (§7.3); a governance liability at Level 3 |

The architecture lesson: **no single tool is TLM**. TLM is the *integration* of discovery data (what exists), vendor feeds (when support ends), contracts/licences (what is paid), and the governance register (decisions taken) — the tool stack exists to feed the §7 operating model, not to replace it. The ISO/IEC 19770-1:2017 ITAM-system requirements (§2.2) are the nearest thing to a specification for what the *process* layer must prove, independent of tooling ✅.

### 11.2 What does "TPM" actually stand for? (verified with flags)

The task behind this section was to verify the acronym in the TLM context, because "TPM" is overloaded to the point of being unusable without qualification. Findings from this pass:

1. **Technology Portfolio Management (TPM)** — the IT/TLM-relevant reading. Multiple independent practitioner and vendor sources use TPM for the discipline of governing the technology portfolio across its lifecycle — e.g. the IF4IT best-practices document defines it as "the enterprise discipline for governing technologies as strategic assets through connected inventories, explicit ownership, lifecycle and standards controls, evidence-based assessment, financial and risk transparency, and managed transition execution" ✅ (verified reachable and consistent across IF4IT and vendor knowledge hubs from Ardoq and BlueDolphin in this pass's search). **⚠ however:** this is *practitioner/vendor usage*, not an ISO/consensus standard — the phrase "TPM" is not a standards-body term verified this pass, and the vendor sources (Ardoq, BlueDolphin, pemvish) are marketing-adjacent, so the definition should be treated as convergent industry usage rather than an authoritative standard. Related terms: **IT portfolio management (ITPM)** and **application portfolio management (APM)** cover overlapping ground.
2. **It is NOT the manufacturing "Total Productive Maintenance"** — that TPM belongs to lean manufacturing (equipment effectiveness); the search record shows no IT-lifecycle source using it that way ✅ (exclusion verified by absence: every IT-context hit resolved to Technology Portfolio Management).
3. **Two more collisions a bank will meet:** **Trusted Platform Module** (the security chip — a hardware TPM on every enterprise laptop, which is itself a firmware lifecycle item, §9.1) and **third-party maintenance** (hardware support after the manufacturer's EOL — flagged in §9.1, where "TPM" genuinely means independent maintenance vendors). Context disambiguates, but documents should spell the term out.

**House rule for this guide:** when this guide says TPM it means **technology portfolio management**; hardware discussions say "third-party maintenance" in full; and the glossary (§15) lists all four meanings.

### 11.3 Tooling's honest limits

Tooling cannot fix governance gaps — the repeated lesson of the vendor and practitioner record ⚠: an estate with a perfect CMDB and no EOL gate still runs unsupported software (the decision machinery is missing), and an estate with a complete register and no discovery feed decays within a quarter (the data goes stale). The §7 operating model — inventory, clocks, gates, rhythm — is the actual system of record; tools are its sensors and dashboards.

**Cross-references:** the SAM/SBOM software-inventory mechanics are in [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md); portfolio strategy is [IT Strategy](../management/it_strategy_guide.md); the vendor/contract layer is [Vendor Management](../management/vendor_management_guide.md).

---

## 12. The Worked Example: A TLM Programme at Cymbal Bank

> **Setting and honesty note.** Cymbal Bank is the recurring persona bank of this research series: a full bank with a Singapore hub, supervised by MAS. Everything in this worked example — names of applications, counts, dates, register entries — is a **pedagogical construction** built to exercise the framework of §1–§11; none of it is drawn from any real bank's disclosures, and no vendor date cited here should be read as the real support date of any real product.

### 12.1 The trigger: an audit finding and a supervisory conversation

In early 2026 Cymbal Bank's internal audit completed a technology-risk review of the payments estate. Three findings set the programme in motion:

1. **Finding TR-2026-014 (High):** no complete inventory of production software versions existed — the CMDB covered servers but not the applications on them, and the "asset register" for several payment applications was a spreadsheet last updated in 2022.
2. **Finding TR-2026-015 (High):** at least one critical application was running on a version whose vendor support had ended, with **no recorded risk acceptance** — the §4.4 invariant violated.
3. **Finding TR-2026-016 (Medium):** audit sampled five "retired" applications and found two still powered on, one still receiving file transfers, and licence renewals continuing for all five — the §10 zombie pattern.

The audit report went to the board technology committee alongside the bank's MAS-facing technology-risk posture (the TRMG governance expectations and FSM-N05 reliability/availability/recoverability requirements of §5). The committee's question — *"which critical systems run on unsupported software, and what are we doing about it?"* — became the mandate for a TLM programme: **build the inventory, stand up the EOL register, and retire the worst offender properly** (the §12.4 case study) within 18 months.

### 12.2 Building the asset inventory

The programme ran discovery (network scanning, agent-based server discovery, mainframe dataset analysis, SaaS catalogue extraction from procurement records) and reconciled results against the CMDB, licence records, and procurement history. What surfaced (illustrative counts for a mid-size bank's payments/APAC hub):

| Estate class | Items found | Surprises found by reconciliation |
|---|---|---|
| Applications (custom + COTS) | ~900 | ~60 had no named owner; ~40 existed only in the CMDB (already dead) |
| Servers (virtual + physical) | ~2,400 | ~90 "ghost" servers — no traffic in 12 months, still patched and licensed |
| Mainframe workloads (z/OS region) | 14 applications | 2 had no IBM support entitlement recorded; 3 depended on the §12.4 workload |
| Software products/versions in use | ~700 products | 31 product-version combinations past end of support; 60 more within 24 months |
| Hardware models | ~120 models | 8 server/storage models past manufacturer EOL with firmware no longer updated |
| SaaS/cloud services | ~150 | 11 deprecation notices outstanding, none tracked in one place |
| Licences/contracts | ~1,100 | annual overspend estimated at ~8–10% from ghost licences and shelfware ⚠-illustrative |

Sample inventory rows (register fields per §7.1):

| Asset ID | Product / version | Type | Owner | Business service | Vendor support status | Support end | Next action |
|---|---|---|---|---|---|---|---|
| CB-SRV-11873 | RHEL 7.9 (older track) | OS | Payments ops | Payments gateway | Extended-life track ⚠-real dates differ | T-14 months | Upgrade to RHEL 9 or accept with expiry |
| CB-APP-0042 | In-house COBOL "STDINS" (standing instructions) on z/OS CICS | Custom mainframe | Payments ops | Wholesale standing instructions | IBM-supported version; **internal** support = 3 staff (2 retiring) | skills T-36 months | **Retire** (§12.4) |
| CB-APP-0311 | COTS reconciliation suite v2018 | COTS | Finance | Nostro reconciliation | **Past end of support** | already past | Risk acceptance + upgrade plan |
| CB-SRV-0330 | Dell-class server model X | Hardware | Infrastructure | Batch processing | Past manufacturer EOL; firmware frozen | already past | Replace at next hardware refresh |

### 12.3 The EOL risk register

From the inventory, the programme built the register (§4.4 fields). Extract, red/amber/green by proximity and criticality:

| # | Item | Rung & date | Critical? | Exposure | Decision | Owner | Review |
|---|---|---|---|---|---|---|---|
| EOL-001 | COTS reconciliation suite v2018 — past end of support | Past EOS; lifeline (paid) available | Yes (finance reporting) | Unpatched; audit repeat finding | **Upgrade to current version by Q3 2026**; interim acceptance (expires 31 Jul 2026) | Finance ops | Monthly |
| EOL-002 | RHEL 7.9 fleet (214 servers) | Maintenance phase ending T-14 months | Mixed | Patch gap post-maintenance; ELS cost if delayed | **Upgrade wave RHEL 8→9** starting Q2 2026; no acceptances beyond T-6 | Infrastructure | Quarterly |
| EOL-003 | Mainframe workload STDINS — COBOL skills clock | IBM supported; internal support 3 staff, 2 retiring ≤36 months | Yes (wholesale payments) | Capability loss + change freeze when skills leave | **Retire by Q4 2027** (§12.4); board-visible decision | Payments ops + risk | Quarterly to board committee |
| EOL-004 | 8 hardware models past manufacturer EOL | Past EOL | No (non-critical) | Firmware CVEs unfixable | **Accept** (spares on shelf, isolated segment), expires annually; replace at refresh | Infrastructure | Annually |
| EOL-005 | SaaS payment-notification API — vendor deprecation announced | Deprecation T-9 months (vendor notice) | Yes | Capability loss | **Migrate** to successor API by T-3; exit plan per outsourcing regime | Payments ops | Monthly |

Register discipline delivered two things the audit had asked for: **no critical item reaches a support end without a dated decision**, and **every acceptance expires** (EOL-001's interim acceptance, EOL-004's annual re-acceptance). The quarterly EOL report — count of critical items past support, count within 12 months, acceptance expiry calendar — became a standing board-technology-committee paper, i.e. the §4.5 board visibility the TRMG framing expects.

### 12.4 Case study: retiring the STDINS COBOL workload

**The asset.** STDINS ("standing instructions") — an in-house COBOL application on z/OS (CICS transactions, DB2 storage), built 1989, executing wholesale customers' recurring payment instructions. Functionally small; institutionally terrifying: it was the §6.1 profile exactly — vendor-supported platform, but the *code* was supported by three people, two retiring within three years.

**1. Discovery.** The §10.2 workstreams: interface inventory found 23 upstream/downstream connections — the real-time CICS transactions to the payments hub, a nightly batch to the nostro reconciliation system, file transfers to the sanctions-screening queue, monthly reports to finance, and (the classic zombie risk) a quarterly sweep used by one operations team "just in case". Data discovery: 31 years of standing-instruction records and history — customer data with retention and secrecy obligations.

**2. Risk assessment.** In the §4 register language: the platform was supported, so the *vendor* risk was amber; the real red risk was **skills and change**: a required payments-scheme change (ISO-style message changes, regulatory deadline) could not be delivered if the two senior programmers had retired, and the 2027 deadline set the runway. Also red: the quarterly "just in case" consumer meant no one could prove which dependents were real — the §10.3 pitfall, found before it bit.

**3. Remediation/extraction.** The bank chose **extraction to a Java microservices implementation on the OpenShift platform** (the platform cluster guides cover the target environment; the business decision was "replace the capability, not the hardware"). Execution pattern, following the sibling guides' migration rules of thumb (never skip dual-run; do the data work first):

- **Data work first:** standing-instruction records were profiled, cleansed, and mapped to the new data model; history was migrated to the new archive store with the retention schedule applied (7 years of executed-instruction records retained in the archive; older records scheduled for destruction with certification; the §10 data-handling matrix signed by the data owner and compliance). Data migration consumed ~60% of the programme effort — matching the sibling guide's rule of thumb ⚠ (§6.1).
- **Dual-run:** STDINS and the new service ran in parallel for a full quarterly cycle so the "just in case" consumers had to declare themselves — every dependent was migrated or cut over *before* the observation period started.
- **Regression and equivalence:** a comparison harness replayed a quarter of real instruction executions through both systems and reconciled outcomes to the cent — the COBOL logic (cut-off times, holiday calendars, currency rounding) became the test oracle.

**4. Data handling and shutdown.** After the observation period: interfaces cut in dependency order, the z/OS region resources released, DB2 data archived per the retention matrix with access controls and a named archive owner, extraneous copies destroyed with certificates of destruction logged. The batch jobs, file transfers, and the quarterly sweep were verified dead — post-retirement verification found one resurrected report feed in month two (an operations spreadsheet that still pointed at the old file share), which is exactly why §10.2 step 7 exists.

**5. Vendor exit.** The bank cancelled the IBM support entitlement for the retired workload's software stack (keeping entitlements for the z/OS platform itself, which continues to run the remaining 13 workloads), surrendered the DB2 capacity, and closed the related maintenance line items. The exit followed the contract-notice periods and was confirmed in writing; the licence/contract register was updated at the same change record as the shutdown, so the renewal gate (§7.1) would never see the ghost licence renew. (For the *outsourced* parts of the estate the same exit runs under the MAS outsourcing regime's exit-plan expectations — §5.3.)

**6. Governance sign-off.** The retirement gate record assembled: board-committee-approved retirement decision (with the skills-clock evidence), the dependency map, the data-handling matrix with compliance sign-off, the dual-run evidence, the vendor-exit confirmations, and the register closure. Internal audit sampled the file in Q1 2027 and closed finding TR-2026-015's STDINS limb with no issues — the first time a Cymbal Bank retirement had survived audit intact, because the evidence existed rather than being reconstructed afterwards.

### 12.5 Programme outcomes and lessons

Illustrative outcomes at 18 months: critical assets past support end reduced to **zero** (all remaining items under expiring acceptances or in upgrade); inventory completeness measured by discovery-reconciliation at ~97%; ghost servers reduced by ~80%; licence spend down ~7% from shelfware removal; and the STDINS retirement freed the mainframe budget line that funded the next two modernisations. The lessons the programme wrote down:

1. **The inventory is a control, not a project** — it decays from the moment discovery ends; continuous feeds and the entry gate (§7.1) keep it alive.
2. **The register's decisions matter more than its dates** — audit did not challenge the one past-EOS item; it challenged the *absence of a signed, expiring decision* about it.
3. **Retire the skills clock alongside the vendor clock** — STDINS was retired because its *people* had an end-of-life date, which no vendor invoice would ever have flagged.
4. **Data work first, dual-run always, verification after** — the three rules that kept the case study on budget and audit-clean.

**Cross-references:** the target platform for extraction work is covered by [OpenShift](secure_red_hat_openshift_guide.md) / [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md); the MAS obligations the programme answered are in [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md); the ERM machinery (appetite, acceptance, committees) is [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md).

---
## 13. Claims Audit

Every factual claim in this guide carries an in-body marker — ✅ (verified against a primary source this pass), ⚠ (partially verified, source-dependent, or practice-varies), ❌ (not verifiable) — per the marker convention in the header. This section consolidates those markers into one ledger, grouped by claim family, so a reader can see at a glance what the guide stands behind and where it is explicitly asking to be checked. The verification pass was bounded (the header records the ≤10-call web budget), so the ledger reflects one disciplined pass against the primary sources listed in §15.2, not an exhaustive re-verification of every page a vendor publishes.

### 13.1 Security-history claims (§1.3)

| Claim | Where | Verdict | Verification basis |
|---|---|---|---|
| WannaCry (2017) spread via unpatched Windows SMBv1; Microsoft issued an emergency out-of-band update for the out-of-support Windows XP in May 2017 | §1.3 | ✅ | Microsoft Security Response Center material, verified this pass |
| Equifax breach (2017) traced to unpatched Apache Struts, CVE-2017-5638 | §1.3 | ✅ | US House Oversight Committee report (2018), verified this pass |

### 13.2 Industry-framework claims (§2.2)

| Claim | Where | Verdict | Verification basis |
|---|---|---|---|
| ITIL v3's five service-lifecycle stages and the 2011 refresh | §2.2 | ✅ | Standard ITIL v3 facts; stage names stable across sources |
| ITIL 4 (2019) replaced the lifecycle with a Service Value Chain; ITAM and SAM became practices | §2.2 | ⚠ | Launch year certain from the industry record; exact month and the current practice catalogue not re-verified this pass |
| ISO/IEC 20000-1:2018 is a service-management-system requirements standard, not a phase model | §2.2 | ⚠ | Standard exists; clause numbering not re-verified this pass |
| ISO/IEC 19770-1:2017 sets ITAM-system requirements covering all types of IT assets "throughout all stages of the life cycle" | §2.2 | ✅ | Title and scope verified on iso.org (standard 68531); clause-level detail is paywalled ⚠ |
| COBIT 2019's BAI (Build, Acquire, Implement) and DSS (Deliver, Service, Support) domains carry the lifecycle weight | §2.2 | ⚠ | Domain letters standard; objective-level mapping is this guide's reading |
| NIST withdrew SP 800-64 Rev. 2 in May 2019; its SDLC vocabulary is now historical | §2.2 | ✅ | nist.gov news item (May 2019), verified this pass |
| Current NIST life-cycle thinking runs through SP 800-37 (RMF) and SP 800-160 | §2.2 | ⚠ | Successor status not re-verified this pass |

### 13.3 Vendor support-ladder claims (§3)

| Claim | Where | Verdict | Verification basis |
|---|---|---|---|
| Microsoft Fixed Lifecycle: minimum five years of Mainstream Support; Extended Support (paid, security and limited fixes) for eligible products; service-pack policy folded in February 2020 | §3.2 | ✅ | learn.microsoft.com lifecycle policy, verified this pass |
| Microsoft Modern Lifecycle: continuous servicing while the customer stays current; notice minimums of 30 days, 12 months, and up to three years for select Azure services | §3.2 | ✅ | learn.microsoft.com, verified this pass |
| Beyond end of support, security updates are available only via the paid ESU programme; self-help content stays available for at least 12 months | §3.2 | ✅ | learn.microsoft.com, verified this pass |
| Windows 10 end of support on 14 October 2025 | §3.2 | ⚠ | Widely published and listed on Microsoft's lifecycle hub; the individual announcement page was not re-extracted this pass |
| Oracle: Premier Support for about five years from GA; Extended Support for an additional fee; Sustaining Support is "pre-existing fixes only", with no new security updates | §3.3 | ✅ | oracle.com/support/lifetime-support, verified this pass |
| Oracle OS and VM products carry ten years of maintenance from GA; hardware Premier Support covers integrated software and firmware | §3.3 | ✅ | oracle.com/support/lifetime-support, verified this pass |
| Applications Unlimited keeps Premier Support on the continuous-innovation releases through at least 2037 | §3.3 | ✅ | oracle.com/support/lifetime-support, verified this pass |
| RHEL runs a ten-year life cycle across three production phases (Full, Maintenance, Extended Life) plus optional paid extensions | §3.4 | ✅ | access.redhat.com life-cycle pages, verified this pass |
| From RHEL 9, the Extended Life Cycle (ELC) unifies the paid extensions (replacing EUS/EEUS/E4S) and the Long-Life add-on reaches 14+ years | §3.4 | ✅ | access.redhat.com, verified this pass |
| Security errata (RHSA) severity criteria cover Critical/Important/Moderate CVEs at CVSS ≥ 7, effective 1 April 2025 | §3.4 | ✅ | access.redhat.com errata policy, verified this pass |
| RHEL 7 extended-support end date | §3.4 | ⚠ | Not re-verified this pass; see §14 |
| IBM publishes per-product, per-version lifecycle dates with machine-readable XML feeds, and sells Advanced/Extended/Sustaining Support for Passport Advantage software | §3.5 | ✅ | ibm.com/support/lifecycle, verified this pass |
| Cisco treats EOL as a process (notification → end of sale → Last Date of Support), typically notifying about six months before end of sale; the verified policy text covers notifications on/after 29 September 2022 | §3.6 | ✅ | cisco.com End-of-Life Policy, verified this pass |
| VMware/Broadcom post-acquisition lifecycle and licensing state | §3.7 | ❌ | Lifecycle site not reachable within this pass's budget — re-verify against current Broadcom pages before relying on pre-2024 dates |
| Microsoft is the only one of the five compared vendors that publishes fixed customer-notice minimums | §3.8 | ⚠ | Conclusion from this pass's reading of the five policy pages; absence claims are only as good as the pages read |

### 13.4 Regulatory claims (§5, §8)

| Claim | Where | Verdict | Verification basis |
|---|---|---|---|
| TRMG current text published 18 January 2021, revising the 2013 original | §5.1 | ✅ | mas.gov.sg TRM Guidelines page, verified this pass |
| TRMG applies to roughly 60 institution types, including every class of bank | §5.1 | ✅ | mas.gov.sg "applies to" list, verified this pass |
| Sectoral TRM notices, including bank Notice 644/644A, were cancelled with effect from 10 May 2024 and replaced by harmonised FSM notices — FSM-N05 for banks, FSM-N22 for cyber hygiene | §5.1 | ✅ | Repo MAS guide's verified mas.gov.sg extracts (cross-reference) |
| Reportable-incident submissions move to the MAS-Tx platform from 1 February 2026 (December 2025 circular) | §5.1 | ✅ | Repo MAS guide (cross-reference) |
| Detail of the original 2013 TRMG text | §5.2 | ⚠ | Documented in the repo guide but not directly re-confirmed this pass |
| The 2021 TRMG revision added cyber-security assessment and cyber-surveillance/security-operations content | §5.2 | ⚠ | Secondary industry summaries (PwC, Kroll); not re-extracted from the primary text this pass |
| MAS publishes no literal "end-of-support register" requirement; the EOL expectation is derived from governance, resilience and cyber-hygiene provisions | §5.2 | ⚠ | The guide's reading of the guidelines — defensible but interpretive |
| FSM-N22 cyber-hygiene scope (administrative accounts, security patching, baseline standards); MAS Outsourcing Guidelines (published 11 December 2023, effective 11 December 2024) with Notices 658 (banks) / 1121 (merchant banks) | §5.3, §8.2 | ✅ | Verified wording recorded in the repo MAS and Vendor Management guides (cross-reference) |
| SBOM regulatory context: US EO 14028 / OMB M-22-18 / NIST SP 800-218 and the EU Cyber Resilience Act | §8.3 | ✅ | Verified in the repo's SBOM sibling guide (cross-reference) |

### 13.5 Practice, tooling and worked-example claims (§6–§12)

| Claim | Where | Verdict | Verification basis |
|---|---|---|---|
| Data migration consumes roughly 60–70% of a core-replacement programme's effort | §6.1, §10.2, §12.4 | ⚠ | Practitioner rule of thumb from the sibling core-banking guide; not a sourced statistic |
| Specific mainframe/COBOL workforce-scarcity percentages | §6.1 | ❌ | No citable current statistic verified this pass; treat any specific figure as unverified |
| Long-tail pattern: a minority of applications carries most business value while the tail consumes disproportionate support effort and risk | §6.2 | ⚠ | Practitioner consensus; no single citable statistic |
| Zombie assets: "retired" systems that stay powered, licensed and connected | §10.1 | ⚠ | Practitioner and audit post-mortem consensus, not one citable study |
| ISO/IEC 19770-1:2017 is the nearest standard for what an inventory process must prove | §7.1, §11.1 | ✅ | Title/scope verified (§2.2); the "nearest standard" framing is the guide's judgement |
| TLM maturity levels 1–4, from ad-hoc registers to a governed programme | §7.3 | ⚠ | Practice pattern, shape only |
| SaaS and cloud deprecation-notice norms beyond Microsoft's published minimums | §8.4, §9.3 | ⚠ | Vary by vendor; no standard verified this pass |
| NIST SP 800-88 as the data-erasure reference in practice | §9.2 | ⚠ | Standard exists and is widely used; not re-verified this pass |
| "TPM" = Technology Portfolio Management in IT practice (IF4IT-style definition) | §11.2 | ⚠ | Consistent across the IF4IT/Ardoq/BlueDolphin sources verified this pass, but convergent industry usage, not a standards-body term |
| "TPM" is not used as "Total Productive Maintenance" in IT-lifecycle sources | §11.2 | ✅ | Verified by absence across this pass's IT-context search results |
| Cymbal Bank worked-example figures (inventory counts, register entries, dates) | §12 | ⚠ | Pedagogical constructions by design, per the §12 honesty note — not real disclosures and deliberately not verifiable |

**Reading the ledger.** The guide's centre of gravity is exactly where it should be for a reference document: the hard, citable claims — security history (§13.1), the shape of vendor policies (§13.3), and MAS instrument dates (§13.4) — verified ✅ against primary sources; the interpretive and practice-consensus material honestly marked ⚠; and exactly two ❌ rows: VMware/Broadcom's current state (site unreachable this pass, §3.7) and specific mainframe/COBOL workforce statistics (none citable, §6.1). The ⚠ rows are not weaknesses in the writing; they are the guide telling the reader precisely where to re-check before relying on it. §14 itemises those re-check points.

---
## 14. What Could Not Be Verified

The Claims Audit (§13) records *what* this pass could and could not stand behind; this section names the specific gaps so a reader knows exactly where to re-check before relying on the guide. Nothing below is guessed — each item is either flagged ❌ or ⚠ in the body, cross-referenced here for honesty. Where the marker convention, the bounded web budget (≤10 calls, per the header) or a paywall/portal boundary stopped verification, this section says so plainly.

### 14.1 The specific gaps

- **VMware/Broadcom current state (§3.7, ❌).** The VMware lifecycle site could not be reached within the pass budget. Post-acquisition (Broadcom, November 2023) support and licensing terms changed materially; any pre-2024 dates or support-phase names circulating in third-party material should be re-verified against current Broadcom pages before use.
- **Individual vendor dates as of reading time (§3.2–§3.5, ⚠).** The guide verifies vendor *policies*, not every published date: Windows 10's individual end-of-support announcement (§3.2), RHEL 7's extended-support end (§3.4) and per-product IBM dates (§3.5) were not extracted date-by-date this pass. Load dates into the register from the vendor's current page when you build it — lifecycle pages are living documents.
- **ISO clause-level text (§2.2, ⚠).** ISO/IEC 19770-1 and ISO/IEC 20000-1 titles and scope were verified on iso.org, but clause-level content is paywalled, so any clause-level claim stays ⚠.
- **MAS instrument detail (§5.1–§5.4, ⚠).** The FSM notice map (FSM-N05/FSM-N22), the December 2025 MAS-Tx circular and the outsourcing Guidelines dates are cross-referenced from the repo MAS guide's verified mas.gov.sg extracts rather than re-extracted from mas.gov.sg this pass; the original 2013 TRMG text's detail and the audit-accountability ("IAC") theme wording are recorded-but-not-directly-re-verified. MAS publishes no literal "end-of-support register" requirement — that reading is the guide's, derived from the governance, resilience and cyber-hygiene provisions (§5.2, ⚠).
- **Retention periods for specific record classes (§10.2).** The guide states the principle — retention is set by the bank's regulatory and legal obligations, with legal holds overriding schedules — but lists no specific periods; they vary by jurisdiction, instrument and entity, and must be taken from the bank's own obligations register rather than from this guide.
- **Industry statistics (§6.1–§6.2, ❌/⚠).** Mainframe/COBOL workforce-age figures (§6.1 ❌), the 60–70% data-migration rule of thumb (§6.1, §12.4 ⚠) and long-tail distribution claims (§6.2 ⚠) are practitioner consensus or sibling-guide rules of thumb, not sourced statistics; trace any specific number before reuse.
- **Cymbal Bank worked-example figures (§12, by design).** Names, counts, dates and register entries in the worked example are pedagogical constructions per the §12 honesty note. They are unverifiable by design — none are drawn from any real bank's disclosures — and must not be cited as real.
- **Firmware lifecycle pages (§9.1, ⚠).** The firmware layer's per-manufacturer EOL/EOSL pages were not individually verified this pass; the body flags the layer's risk profile and points to manufacturer policy pages rather than citing specific firmware dates.
- **Non-Microsoft notice practices (§3.8, ⚠).** The conclusion that only Microsoft publishes fixed customer-notice minimums rests on the five vendor policy pages read this pass. Absence claims are only as good as that reading; a vendor may publish notice commitments on pages this pass did not surface.
- **NIST SP 800-88 specifics (§9.2, ⚠).** Referenced as the widely used data-erasure standard without re-verifying its current revision this pass.
- **SaaS and cloud deprecation norms (§8.4, §9.3, ⚠).** Beyond Microsoft's published minimums (§3.2), no other vendor's deprecation-notice norms were verified this pass.
- **The "TPM" label (§11.2, ⚠).** No standards-body definition of TPM as Technology Portfolio Management was found this pass; the guide relies on convergent practitioner/vendor usage (IF4IT, Ardoq, BlueDolphin) and says so rather than presenting the label as authoritative.

The pattern across all of §14 is deliberate: this guide would rather mark a claim ⚠ and point the reader at the re-check than let an unverified number or a portal-only source masquerade as fact. The primary sources a reader needs for those re-checks are listed in §15.2.

---
## 15. Glossary and Closing

### 15.1 Glossary

Terms below are all used in this guide and defined at the point of use; this list is a one-line lookup index with the section where each term carries the most weight. Where the guide has flagged vocabulary as vendor-specific or overloaded (§3.1, §9.1, §11.2), the gloss says so.

- **Applications Unlimited** — Oracle's commitment to offer Premier Support on the continuous-innovation releases of PeopleSoft, E-Business Suite, JD Edwards, Siebel and Hyperion through at least 2037 (§3.3).
- **CI — configuration item** — a recordable asset or component in the CMDB, with its relationships (§1.2).
- **CMDB — configuration management database** — the *state* model of the estate (what is deployed); TLM adds the *time* model (how old it is, when support ends) (§1.2).
- **COBOL** — the language of most legacy banking cores; in-house COBOL has no vendor clock, so its "EOL" is a governance and skills decision, not a calendar event (§6.1).
- **CVE — Common Vulnerabilities and Exposures** — the standard identifier for public vulnerabilities; the currency of patch and EOL risk discussion (§1.3, §3.4).
- **Decommissioning** — the disciplined retirement of an asset: dependency cut, data handling, vendor exit and evidence — the "retire" phase executed properly (§10.2).
- **Dual-run** — running old and new systems in parallel over a full business cycle so every dependent declares itself before cutover (§12.4).
- **ELC and Long-Life (LL) add-on** — Red Hat's paid extensions from RHEL 9 (ELC replaces EUS/EEUS/E4S), taking errata coverage to 14+ years on eligible releases (§3.4).
- **EOL — end of life** — dangerously ambiguous across vendors: for Cisco a *process* ending in obsolescence; for many software vendors a looming support cut-off. Normalise it; never store it as the vendor's word alone (§3.1).
- **EOL risk register** — the operational heart of TLM-as-risk: every item at or near end of support, with criticality, exposure, mitigation options and a dated, owned, *expiring* decision (§4.4).
- **EOS — end of sale** — after this date a product can no longer be purchased (Cisco milestone; Oracle and IBM fold it into lifecycle milestones) (§3.1, §3.6).
- **EOSL — end of service life** — hardware-vendor vocabulary for the date after which a product is no longer serviced or repaired; manufacturer-specific terminology — read the actual policy (§9.1).
- **ESU — Extended Security Updates** — Microsoft's paid, time-boxed post-support lifeline; the pattern to expect for every late-life Microsoft product a bank still runs (§3.1–§3.2).
- **Fixed Lifecycle Policy** — Microsoft's dated-support model: a minimum of five years of Mainstream Support plus Extended Support for eligible products (§3.2).
- **FSM notices (FSM-N05 / FSM-N22)** — the post-May-2024 harmonised binding notices under the FSMA 2022: FSM-N05 for bank technology-risk management (replacing the sectoral TRM notices) and FSM-N22 for cyber hygiene (§5.1).
- **GA — general availability** — product launch; the vendor support clock starts (§3.1).
- **ITAM — IT asset management** — financial and inventory control of IT assets across their life (ISO/IEC 19770 practice); TLM's substrate (§1.2, §2.2).
- **LDOS — Last Date of Support** — Cisco's hard cliff: after it, "support is no longer available" (§3.6).
- **Lifecycle phases** — this guide's canonical model, Plan → Acquire → Deploy → Operate → Retire; a loop at portfolio level, not a one-way trip per asset (§2.1).
- **Mainframe (z/OS, CICS, IMS, DB2)** — IBM's big-iron platform and its stack, still running a startling share of core banking; long vendor support windows but an ageing skills base (§6.1).
- **Mainstream / Extended Support** — Microsoft's Fixed-Lifecycle rungs: full fixes versus paid, security-plus support; beyond Extended, only ESU (§3.2).
- **MAS Notice 644/644A** — the pre-harmonisation sectoral technology-risk notice for banks, cancelled with effect from 10 May 2024 and replaced by FSM-N05 (§5.1).
- **Modern Lifecycle Policy** — Microsoft's continuous-servicing model with published notice minimums: 30 days, 12 months, and up to three years for select Azure services (§3.2).
- **Premier / Extended / Sustaining Support** — Oracle's price ladder: comprehensive support for about five years from GA, a paid extension, then "pre-existing fixes only" with no new security updates (§3.3).
- **Retention schedule / legal hold** — data-keeping obligations that drive decommissioning, versus litigation or investigation freezes that override them (§10.2).
- **RHEL — Red Hat Enterprise Linux** — the ten-year, three-phase life cycle (Full → Maintenance → Extended Life) plus paid extensions; the cleanest published phase model of the vendors compared (§3.4).
- **RHSA / RHBA** — Red Hat security and bug-fix errata; security errata severity criteria cover Critical/Important/Moderate CVEs at CVSS ≥ 7 from 1 April 2025 (§3.4).
- **Risk acceptance** — a recorded, time-boxed decision to tolerate residual lifecycle risk; acceptances must expire and be re-reviewed, or "accepted risk" quietly becomes permanent risk (§4.4).
- **SAM — software asset management** — licence, entitlement and usage discipline (ISO/IEC 19770 practice); the software subset of TLM (§1.2, §11.1).
- **SBOM — software bill of materials** — the component inventory of software, and the supply-chain clock inside the software estate (§8.3; sibling guide).
- **Shadow IT** — assets provisioned outside official channels; an inventory problem before it is a security problem (§2.3).
- **Shelfware / ghost licences** — licences paid for products that are unused or already retired; the financial signature of a skipped Retire phase (§1.3, §10.3).
- **SWID tag — software identification tag** — the ISO 19770-2 machine-readable identity mechanism for installed software (§2.2).
- **TCO — total cost of ownership** — whole-life cost, including the cost of eventually retiring the asset (§2.1).
- **Three lines of defence** — first line owns and operates, second line sets policy and monitors risk, third line audits; TLM maps onto it cleanly (§4.3).
- **TLM — technology lifecycle management** — the governance-backed discipline of managing every technology asset through plan → acquire → deploy → operate → retire (§1.1).
- **TPM — four meanings, state which you mean** — technology portfolio management (this guide's §11 reading); Trusted Platform Module (the security chip, itself a firmware lifecycle item); third-party maintenance (hardware support after manufacturer EOL — §9.1); Total Productive Maintenance (lean manufacturing, not IT) (§11.2).
- **TRMG — Technology Risk Management Guidelines** — MAS's technology-risk guideline layer; current text published 18 January 2021, revising the 2013 original (§5.1).
- **Zombie asset** — a "retired"-in-name system still powered, patched, licensed and connected; the §10.1 failure pattern in one object.

### 15.2 Primary sources and further reading

The primary sources below are the ones this guide actually verified during the pass (see §13) — no URL is listed here that was not reachable or already cross-referenced in the body. Vendor lifecycle pages are living documents: re-check dates on the current page at use time (§14).

- **Microsoft lifecycle** — Fixed and Modern Lifecycle policy pages and per-product search at learn.microsoft.com/lifecycle (§3.2).
- **Oracle Lifetime Support** — policy and feature tables at oracle.com/support/lifetime-support (§3.3).
- **Red Hat Enterprise Linux Life Cycle** — lifecycle and errata policy pages at access.redhat.com (§3.4).
- **IBM Support Lifecycle** — per-product, per-version dates and machine-readable XML feeds at ibm.com/support/lifecycle (§3.5).
- **Cisco End-of-Life Policy** — the EOL-process milestones at cisco.com (§3.6).
- **MAS technology risk** — the TRM Guidelines and FSM notices at mas.gov.sg (§5.1); the repo's [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) carries the verified instrument map this guide cross-references.
- **ISO/IEC 19770-1:2017** — ITAM-system requirements, at iso.org (standard 68531) (§2.2).
- **NIST** — SP 800-64 Rev. 2 withdrawal notice (May 2019) and the current Risk Management Framework (SP 800-37) at nist.gov (§2.2); SP 800-88 for data erasure (§9.2, flagged ⚠ in §14).
- **Repo siblings** — the deeper treatments this guide deliberately does not re-derive: [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md), [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md), [Vendor Management](../management/vendor_management_guide.md), [Core Banking Systems](../banking/core_banking_systems_guide.md), [Coreless Banking](../banking/coreless_banking_guide.md), [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md), [IT Strategy](../management/it_strategy_guide.md), and the platform-lifecycle cluster listed in the header.

### 15.3 Closing

A lifecycle-management guide that ended at the worked example would have shown the machinery without the mandate. So here it is: every asset known and owned; every vendor clock, internal clock and contract clock on a register with an alert at the look-ahead; every support-end date met by a recorded, expiring, owned decision; every retirement executed with dependency evidence, data handling, vendor exit and post-retirement verification. That is the golden rule of §2.4 made mechanical, the register of §4.4 made current, and the operating model of §7 made routine — at a bank that runs mainframes, SaaS, a cloud estate and twenty years of tail applications at once.

None of it is exotic. The exotic thing would be a bank that discovered its own estate only when a vendor stopped answering the phone, or when an auditor asked which critical systems were running on unsupported software. The guide's answer to that question is deliberately unglamorous: know the estate completely, watch every clock, force every decision in advance, and close every retirement with evidence — once a quarter, every quarter, for as long as the estate exists. Every asset known, every version tracked, every end-of-support date on a register, and every retirement executed with the same rigour as the acquisition — that loop, run with governance on the whole portfolio, is the managed lifecycle.
