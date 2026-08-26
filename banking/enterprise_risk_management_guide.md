# Enterprise Risk Management: The ERM Discipline — A Comprehensive Guide

*The dedicated deep-dive on enterprise risk management — the discipline that looks at risk the way the board does: as a whole-enterprise, strategy-linked portfolio, not a stack of siloed specialist functions. From the COSO ERM framework (2004 cube → 2017 update) to ISO 31000 (2009 → 2018), from the risk taxonomy to the three lines of defence, from risk appetite to the ICAAP, ending in a worked ERM-framework design for a Cymbal Bank.*

**Series:** Banking & Financial Technology Guides — this is the **ERM-framework anchor** of the risk cluster. The cluster's other guides own the *quant*, *systems*, *regulatory*, and *data* sides of risk; this guide owns the *discipline* — the enterprise-wide view that holds them together. The sibling division of labour: [Risk Management Models Guide](risk_management_models_guide.md) owns the mathematics (VaR/ES, PD/LGD/EAD, FRTB — the *modelling* side); [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) owns the platforms; [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md) owns Pillar 1/2 capital and the ICAAP/SREP machinery; [Risk Data Aggregation Guide](risk_data_aggregation_guide.md) owns BCBS 239; [Treasury & ALM Guide](treasury_alm_guide.md) owns liquidity; [CECL Guide](cecl_guide.md) owns credit impairment; [RegTech Guide](regtech_guide.md) owns the regulatory-technology layer; [AI & GenAI Banking Compliance Guide](ai_genai_banking_compliance_guide.md) owns the AI-risk overlay. This guide cross-references all of them instead of re-deriving their content.

**Audience:** Solution architects, risk technologists, and engineers at a global investment bank (the recurring context: Cymbal Bank, with its Singapore APAC hub) who must answer the architect's version of the ERM question: *what does the enterprise view of risk mean for the systems I build?* The answer runs from the framework documents (COSO/ISO) down to the reporting, limit, and capital feeds that make "the enterprise view" a daily artefact rather than a board-deck slogan.

**How this guide is organised:** the ERM overview (the definition, the enterprise-wide view); the COSO ERM (2004 and 2017); the ISO 31000 (2009 and 2018); the risk taxonomy (seven families, each cross-referenced to its cluster guide); the risk governance (three lines of defence, board, CRO, committees); the risk appetite (statements, tolerance, the RAG); the risk process (identification → assessment → response → monitoring → reporting); the banking ERM (ICAAP/Pillar 2, BCBS 239); the COSO-vs-ISO comparison; a worked ERM-framework design for a Cymbal Bank; a one-page summary ("the enterprise view"); the verification record; the glossary; and the series cross-references. Every section ends with a reference table.

**Scope note on verification:** framework facts below (component lists, principle lists, dates, definitions) were checked against primary or authoritative secondary sources in August 2026 — the COSO 2004 executive summary, COSO's 2017 announcement, ISO's own pages for 31000:2009/2018 and IEC 31010:2019, the FSB's November 2013 *Principles for an Effective Risk Appetite Framework*, and the IIA's 2020 Three Lines Model position paper. Where a fact could not be fully verified — the RAG "risk appetite grid" reading in §6, and a handful of dates — it is explicitly flagged **[verify]** rather than asserted. The worked example's numbers are pedagogical constructions, not figures from any bank's disclosures.

**Cross-references used throughout:** risk_management_models_guide.md (the quant side), financial_risk_compliance_systems_guide.md (the systems), basel_regulatory_capital_guide.md (capital + ICAAP/SREP), risk_data_aggregation_guide.md (BCBS 239), treasury_alm_guide.md (liquidity/ALM), cecl_guide.md (IFRS 9/CECL impairment), regtech_guide.md, ai_genai_banking_compliance_guide.md (AI risk), banks_in_singapore_guide.md (MAS expectations), credit_agricole_software_systems_guide.md (the Cymbal Bank context), ../technology/cybersecurity_guide.md (the three-lines-for-security usage), ../management/business_case_development_guide.md (the business case for risk investment), ../management/mba_body_of_knowledge_guide.md (finance vocabulary).

### Reading paths

- **Solution architects and engineers** — §1 (what "enterprise" means for architecture), §5 (the governance structures every risk system encodes), §6–7 (appetite and process as system requirements), §8 (the ICAAP/BCBS 239 data-and-reporting spine), §10 (the worked framework design). Pair with risk_data_aggregation_guide.md for the data layer.
- **Risk managers and governance professionals** — §2–3 (the two frameworks), §5 (three lines, board, CRO, committees), §6 (appetite), §9 (which framework to choose), §11 (the one-page summary).
- **Regulatory/capital teams** — §8 plus basel_regulatory_capital_guide.md §8 (Pillar 2) and risk_data_aggregation_guide.md (BCBS 239); the appetite-to-ICAAP chain is the connective tissue.
- **General readers** — §1, §4 (the taxonomy), §11 (the summary), and the glossary in §13.

Each section stands alone; the tables at the end of each section are the quick-reference, and the cross-references let readers jump between this guide and its siblings without losing context.

---

## Table of Contents

1. The ERM Overview: The Enterprise-Wide View
2. The COSO ERM: 2004 Framework and 2017 Update
3. The ISO 31000: 2009 Standard and 2018 Revision
4. The Risk Taxonomy: Seven Families, One Enterprise
5. The Risk Governance: Three Lines, Board, CRO, Committees
6. The Risk Appetite: Statements, Tolerance, and the RAG
7. The Risk Process: Identify, Assess, Respond, Monitor, Report
8. The Banking ERM: ICAAP, Pillar 2, and BCBS 239
9. The Comparison: COSO vs ISO 31000
10. The Worked Example: An ERM Framework for a Cymbal Bank
11. The Summary: One Page — "The Enterprise View"
12. Verification Notes and Sources
13. Glossary
14. Cross-References in This Series

---

## 1. The ERM Overview: The Enterprise-Wide View

### 1.1 What enterprise risk management is

**Enterprise risk management (ERM)** is the discipline of managing *all* of an organisation's risks — across business lines, legal entities, and risk types — as a single portfolio, in the service of strategy and objectives. The canonical definition comes from the Committee of Sponsoring Organizations of the Treadway Commission (COSO), whose 2004 framework *Enterprise Risk Management — Integrated Framework* defines it as:

> "Enterprise risk management is a process, effected by an entity's board of directors, management and other personnel, applied in strategy setting and across the enterprise, designed to identify potential events that may affect the entity, and manage risk to be within its risk appetite, to provide reasonable assurance regarding the achievement of entity objectives."

Every clause in that definition is load-bearing, and each one is a design requirement:

- **"a process"** — ERM is not a department or a document but an ongoing, embedded activity; it lives in workflows, not binders.
- **"effected by an entity's board of directors, management and other personnel"** — ERM is everyone's job, from the board down; the governance structure exists to make that division of labour explicit (the three lines of defence, §5).
- **"applied in strategy setting and across the enterprise"** — this is the clause that separates ERM from siloed risk management: risk is considered *when strategy is set*, and the view spans the whole enterprise, not one desk or one legal entity. The 2017 COSO update's title — *Integrating with Strategy and Performance* — is this clause elevated to the framework's organising idea.
- **"identify potential events that may affect the entity"** — both threats *and* opportunities; ERM is as much about taking the right risks as avoiding the wrong ones.
- **"manage risk to be within its risk appetite"** — the enterprise defines how much risk it wants (appetite, §6) and manages the portfolio against that target; appetite is the bridge between strategy and risk.
- **"provide reasonable assurance regarding the achievement of entity objectives"** — assurance is *reasonable*, not absolute; ERM reduces the probability and impact of failure, it does not eliminate risk.

### 1.2 The enterprise-wide view vs the siloed view

The single most important idea in the discipline is the contrast the task of this guide is built around: **the enterprise-wide view versus the siloed view of risk**.

| | The siloed view | The enterprise-wide view (ERM) |
|---|---|---|
| **Scope** | One risk type, one business line, one legal entity | The whole group: all risk types, all business lines, all entities |
| **Question** | "How big is the credit risk in this desk's book?" | "What is the aggregate risk of the enterprise, and is it within appetite?" |
| **Unit of analysis** | The exposure, the trade, the process | The risk portfolio — including correlations and concentrations *between* risk types |
| **Time horizon** | Quarterly / annual reporting cycle | Strategy horizon (multi-year) plus the reporting cycle |
| **Owner** | The specialist risk function (credit, market, op-risk) | The board, the CRO, and every line manager |
| **Failure mode** | Risks fall between the silos; aggregate exposure is invisible until a crisis reveals it | Over-bureaucratisation: the enterprise view becomes a paper exercise disconnected from decisions |

The 2007–2009 crisis is the casebook example of the siloed view's failure mode: banks managed credit risk in one function, market risk in another, liquidity in a third — and the *interaction* (a funding freeze amplifying a market shock amplifying credit losses) was nobody's job until it was the whole bank's problem. The 2023 US regional-bank failures are the same lesson in miniature: interest-rate risk in the banking book, deposit-liquidity risk, and market-value losses were managed separately; the combination killed the banks. ERM exists so that the *enterprise* question — "what is our total risk, in aggregate, under stress?" — has an owner and an answer.

### 1.3 Why ERM matters to a bank, and to its architects

For a bank, ERM is not a voluntary management fashion — it is the supervisory expectation. Basel II's Pillar 2 (see basel_regulatory_capital_guide.md §8) requires the bank to run an **ICAAP** — an internal capital adequacy assessment that must cover *all* material risks, not just the Pillar 1 ones — and the board must be able to see, challenge, and steer the firm-wide risk profile. MAS's supervisory expectations for Singapore banks (banks_in_singapore_guide.md) lean on the same machinery: risk governance, risk appetite, and board oversight are supervisory topics in their own right, not just internal-management preferences. The FSB's *Principles for an Effective Risk Appetite Framework* (November 2013, §6) made the risk-appetite statement a supervisory artefact for systemically important firms. In short: **for a bank, ERM is regulation-shaped**.

For a solution architect, the enterprise-wide view translates into concrete system requirements:

- **Aggregation.** Firm-wide risk numbers require firm-wide risk data — the BCBS 239 discipline (risk_data_aggregation_guide.md) is the data-side twin of ERM: you cannot manage risk to appetite if you cannot aggregate risk data to the group level, per business line and per legal entity, quickly and accurately.
- **Integration.** The enterprise view is assembled from the specialist views — credit PD/LGD/EAD engines, market VaR/ES engines, op-risk capital calculators, liquidity LCR/NSFR engines (all in risk_management_models_guide.md) — so ERM systems are integration systems first and calculation systems second.
- **Reporting.** Board-level risk reporting (appetite dashboards, RAG status, emerging-risk registers) is the visible product of ERM, and it runs on the reporting spine described in risk_data_aggregation_guide.md §4.
- **Governance workflow.** Every ERM artefact — the risk appetite statement, the risk taxonomy, the risk register, the limit framework — has an owner, an approval path, and a review cycle. Systems must encode those workflows, not just store the documents.

### 1.4 The lineage: how ERM became a discipline

ERM's history is a series of reactions to failure, each layer adding a requirement the previous one missed:

- **1992 — COSO's *Internal Control — Integrated Framework*.** The internal-control canon that ERM grew out of; SOX (2002) later made it the audit reference for US-listed firms. ERM 2004 is "internal control, widened to the whole risk universe".
- **2001–2004 — Enron, WorldCom, and the ERM birth.** The corporate-governance scandals (and the Sarbanes-Oxley Act) made "risk oversight" a board-level topic; COSO published *Enterprise Risk Management — Integrated Framework* in September 2004 to give boards a framework for it.
- **2008 — the Global Financial Crisis.** The siloed-view failure mode (§1.2) demonstrated at system scale: credit, market, and liquidity risks interacted fatally while each was managed separately; supervisors discovered that risk data aggregation and board-level risk visibility did not exist in usable form. This crisis produced both the FSB's RAF principles (2013) and BCBS 239 (2013).
- **2013 — FSB's *Principles for an Effective Risk Appetite Framework*.** Risk appetite stopped being a management nicety and became a supervised artefact for systemically important firms: board-approved statement, cascaded limits, monitored adherence.
- **2017 — COSO's update.** The 2004 framework's compliance drift (risk registers that changed no decisions) provoked the rewrite: five components, twenty principles, strategy integration as the thesis.
- **2018 — ISO 31000's revision.** The 2009 standard simplified: eight principles, a leadership-centred framework, value creation declared as the purpose.
- **2020s — the new overlays.** Climate risk, cyber risk, and AI risk arrived as enterprise-level concerns that route through the existing taxonomy and governance (ai_genai_banking_compliance_guide.md; ../technology/cybersecurity_guide.md) — the discipline's framework layer has been stable since 2017–2018; the risk *content* keeps changing beneath it.

The lineage's lesson: every layer of ERM exists because a previous layer failed, and the layers accumulate — modern bank ERM is the 1992 controls, the 2004 enterprise view, the 2013 appetite supervision, and the 2017–2018 framework updates, all in force at once.

### 1.5 The overview table

| Aspect | Description |
|---|---|
| **Definition** | COSO 2004: "a process, effected by an entity's board of directors, management and other personnel, applied in strategy setting and across the enterprise, designed to identify potential events that may affect the entity, and manage risk to be within its risk appetite, to provide reasonable assurance regarding the achievement of entity objectives" |
| **The enterprise-wide view** | Risk managed as a firm-wide portfolio (all risk types × all business lines × all entities), integrated with strategy-setting, with aggregate exposure assessed against stated appetite |
| **The siloed view (antithesis)** | Risk managed per specialist function; aggregate and interaction risk invisible until a crisis reveals it (the 2008 and 2023 failure modes) |
| **The frameworks** | COSO ERM (2004/2017) — governance-and-strategy-centric, dominant in the US and financial services; ISO 31000 (2009/2018) — principles-based generic guidelines, global and sector-agnostic (§2–3, §9) |
| **The risk universe** | Credit, market, operational, liquidity (the Basel four) plus strategic, reputational, compliance (§4) |
| **The governance** | Board oversight, CRO, risk committees, three lines of defence (§5) |
| **The target** | Risk appetite statements, tolerance bands, limits, RAG monitoring (§6) |
| **The process** | Identification → assessment → response → monitoring → reporting (§7) |
| **The banking expression** | ICAAP/Pillar 2 capital adequacy, BCBS 239 risk-data aggregation, RAF supervision by FSB/BCBS/MAS (§8) |
| **The quant side (sibling)** | The models that measure the portfolio — PD/LGD/EAD, VaR/ES, LDA/SMA, LCR/NSFR — live in [Risk Management Models Guide](risk_management_models_guide.md); ERM consumes their outputs rather than re-deriving them |

---

## 2. The COSO ERM: 2004 Framework and 2017 Update

### 2.1 The 2004 framework: *Enterprise Risk Management — Integrated Framework*

COSO — the Committee of Sponsoring Organizations of the Treadway Commission, a joint initiative of five US professional bodies (AICPA, IIA, FEI, IMA, and the American Accounting Association) — published *Enterprise Risk Management — Integrated Framework* in **September 2004** (verified against the COSO executive summary). It was the ERM answer to COSO's own 1992 *Internal Control — Integrated Framework*: same DNA, wider scope.

The 2004 framework's signature artefact is the **COSO cube**: three dimensions, each a lens on the same subject:

- **The four objective categories** (the top face): **Strategic** (high-level goals aligned with the mission), **Operations** (effective and efficient use of resources), **Reporting** (reliability of reporting), and **Compliance** (compliance with laws and regulations).
- **The eight components** (the front face): **Internal Environment**, **Objective Setting**, **Event Identification**, **Risk Assessment**, **Risk Response**, **Control Activities**, **Information & Communication**, and **Monitoring**.
- **The entity levels** (the side face): entity, division, business unit, subsidiary — the cube's claim that every component applies at every level.

The eight components are a process in disguise — a vertical flow from objective-setting through event identification and assessment to response, wrapped in control activities, information flows, and monitoring — which is why the 2017 update could reorganise them so cleanly. The 2004 framework also introduced the vocabulary that still anchors the discipline: *risk appetite* (the amount of risk the entity is willing to accept in pursuit of value) and the four risk responses — **avoid, reduce, share, accept** — that survive verbatim into COSO 2017 and ISO 31000's "treatment" options.

### 2.2 The 2017 update: *Enterprise Risk Management — Integrating with Strategy and Performance*

In **September 2017** (verified), COSO published the replacement framework: *Enterprise Risk Management — Integrating with Strategy and Performance* — commonly "COSO ERM 2017". The title is the thesis: the 2004 framework's critics said ERM had become a compliance exercise — risk registers that didn't influence decisions, "shelf documents" — and the 2017 revision was written to weld risk management into *strategy-setting* and *performance management*.

Three changes define the update (verified):

1. **Eight components became five.** The 2004 components were reorganised into five component clusters, each with a set of principles: **Governance & Culture**, **Strategy & Objective-Setting**, **Performance**, **Review & Revision**, and **Information, Communication & Reporting**.
2. **The cube became a helix.** The visual metaphor changed deliberately: the cube implied a static control environment; the helix (a continuous flow from mission/vision/core values through strategy to performance) implies a continuous, iterative process. The framework's structure runs: mission, vision, core values → strategy development → business-objective formulation → implementation and performance → enhanced value.
3. **Twenty principles under the five components.** Each principle is a short, auditable statement of what the organisation must do — the "how" under each "what". (The full list is in §2.3.)

The 2017 update is explicitly a *framework* in the guidance sense — no certification, no mandatory controls, no checklist to pass — but its 20 principles give it a precision that ISO 31000 (deliberately) lacks, which is why large regulated firms and financial institutions treat it as the de facto reference architecture for ERM programme design. COSO supplemented it in 2018 with a *Compendium of Examples* (verified via COSO's own site) showing day-to-day applications of each principle.

### 2.3 The five components and the 20 principles

**Governance & Culture** (principles 1–5) — the tone and structure: risk oversight starts at the board and the culture must reward honest risk conversation.

1. **Exercises board risk oversight** — the board oversees strategy and carries governance responsibilities for risk; it does not merely receive reports.
2. **Establishes operating structures** — operating structures (business lines, legal entities, reporting lines) are configured to pursue strategy and manage risk; accountability for risk is explicit.
3. **Defines desired culture** — the organisation defines the behaviours that characterise its desired risk culture — and audits itself against them.
4. **Demonstrates commitment to core values** — values hold even when they conflict with short-term performance pressure.
5. **Attracts, develops, and retains capable individuals** — human capital is aligned with strategy and risk needs.

**Strategy & Objective-Setting** (principles 6–9) — the 2017 signature: risk is integrated into strategy, not bolted on.

6. **Analyzes business context** — the effect of the external and internal environment on the risk profile is considered when strategy is set.
7. **Defines risk appetite** — risk appetite is defined in the strategy-setting process and articulated so it can be cascaded.
8. **Evaluates alternative strategies** — alternative strategies are assessed for their risk implications; strategy selection is risk-informed.
9. **Formulates business objectives** — objectives at every level align with strategy and appetite, with risk built in from the start.

**Performance** (principles 10–14) — the operational heart: identification, assessment, prioritisation, response, and the portfolio view.

10. **Identifies risk** — risks affecting strategy and objectives are identified across the enterprise — including upside risk (opportunities), not only threats.
11. **Assesses severity of risk** — severity (likelihood × impact) is assessed, with the inherent/residual distinction and reference to the control environment.
12. **Prioritizes risks** — risks are prioritised by severity and strategic importance; the prioritisation drives response and monitoring intensity.
13. **Implements risk responses** — responses are selected (avoid, reduce, share, accept) with cost–benefit logic; residual risk is checked against appetite.
14. **Develops portfolio view** — individual risks are aggregated into an enterprise portfolio view, exposing correlation and concentration that individual assessments miss — the principle that operationalises the "enterprise-wide view" of §1.

**Review & Revision** (principles 15–17) — the framework's self-correction loop.

15. **Assesses substantial change** — substantial changes (strategy shifts, acquisitions, regulation, incidents) trigger re-assessment of the risk profile.
16. **Reviews risk and performance** — the organisation reviews how well ERM is working — are risks identified, calibrated, responded to, reported?
17. **Pursues improvement in ERM** — ERM itself is improved continuously; maturity is a direction, not a state.

**Information, Communication & Reporting** (principles 18–20) — the enterprise view must *travel*.

18. **Leverages information systems** — information and technology support ERM: risk registers in systems, KRI dashboards, automated alerting — not stale spreadsheets.
19. **Communicates risk information** — risk information flows up (to the board), down (appetite to the business), and horizontally (across functions).
20. **Reports on risk, culture, and performance** — the organisation reports on risk, culture, and ERM performance to stakeholders — the board pack, the regulators, the disclosures.

### 2.4 The COSO table

| Element | 2004 framework | 2017 update |
|---|---|---|
| **Title** | *Enterprise Risk Management — Integrated Framework* | *Enterprise Risk Management — Integrating with Strategy and Performance* |
| **Published** | September 2004 | September 2017 |
| **Visual** | The cube: 4 objective categories × 8 components × entity levels | The helix: a continuous flow from mission/vision/values through strategy to performance |
| **Objective categories** | Strategic, Operations, Reporting, Compliance | Strategy and business objectives integrated with mission/vision/values; performance outcomes |
| **Components** | 8: Internal Environment, Objective Setting, Event Identification, Risk Assessment, Risk Response, Control Activities, Information & Communication, Monitoring | 5: Governance & Culture, Strategy & Objective-Setting, Performance, Review & Revision, Information, Communication & Reporting |
| **Principles** | (none — components carried the weight) | 20 principles: 5 / 4 / 5 / 3 / 3 across the five components |
| **Risk responses** | Avoid, reduce, share, accept | Same four, retained (P13) |
| **Strategy link** | "Applied in strategy setting" (in the definition) | The organising idea: risk integrated into strategy-setting and performance (P6–P9) |
| **Post-publication material** | — | 2018 *Compendium of Examples* supplement (verified via COSO) |
| **Typical use** | SOX-era internal-control and ERM programmes | ERM programme design in large/regulated firms; the de facto financial-services reference framework |

---

## 3. The ISO 31000: 2009 Standard and 2018 Revision

### 3.1 ISO 31000:2009 — *Risk management — Principles and guidelines*

ISO 31000:2009, published in **November 2009** (verified via ISO's catalogue page), was the first international *generic* risk-management standard. It was built by ISO/TC 262 using the Australian/New Zealand standard **AS/NZS 4360:2004** as the working draft (verified — the AS/NZS text states it directly), and it deliberately covers *any* organisation and *any* form of risk: "not specific to any industry or sector" (ISO's own description). Where COSO 2004 wrote for boards and internal control, ISO 31000 wrote for every organisation that faces uncertainty — which is to say, every organisation.

The 2009 edition's architecture — retained and sharpened in 2018 — has three layers:

- **Principles** — the attributes that make risk management effective (eight of them in 2018).
- **Framework** — the organisational scaffolding that embeds risk management: leadership and commitment, design, implementation, evaluation, improvement.
- **Process** — the workflow applied to any individual risk or decision: communication & consultation; scope, context & criteria; risk assessment (identification → analysis → evaluation); risk treatment; monitoring & review; recording & reporting.

A companion document, **ISO/IEC 31010:2009** (re-issued as **IEC 31010:2019**, *Risk management — Risk assessment techniques* — verified), catalogues the risk-assessment techniques (brainstorming, checklists, FMEA, HAZOP, bow-tie, Monte Carlo, Bayesian analysis, and ~30 more) and, crucially, *how to select* among them by context, data availability, and decision stakes. It is the toolbox under the process layer.

### 3.2 The 2018 revision: *Risk management — Guidelines*

ISO 31000:2018, available from mid-February 2018 (verified via the ISO/TC 262 committee page), is a revision, not a rewrite: same three-layer architecture, simpler language, sharper emphasis. The committee's own summary of the changes (verified):

- **"Creating and protecting value"** is declared the purpose of risk management — risk management exists to help the organisation achieve objectives, not as an end in itself.
- The principles were tightened to **eight** (below), with **continual improvement**, **stakeholder inclusion**, **customisation**, and **human and cultural factors** called out explicitly.
- The framework's *design* step now includes "understanding the organisation and its context" and "articulating risk management commitment"; *leadership and commitment* is the framework's centre of gravity.
- The process step "establishing the context" became "scope, context and criteria" — with **risk criteria** explicitly linked to the organisation's objectives and stakeholder expectations, and risk expressed in terms of *sources*, *consequences*, *likelihood*, and *events*.

The standard remains what its title says — **guidelines** — so it is deliberately non-prescriptive: no mandatory components, no certification scheme of its own (third-party certification bodies offer ISO 31000-aligned audits, but ISO does not certify against it), no required documents. That flexibility is its strength for general use and its weakness for regulated firms that need an auditable checklist — the comparison in §9.

### 3.3 The 2018 principles

The eight principles of ISO 31000:2018 (verified against ISO/TC 262's summary and the standard's public description):

1. **Integrated** — risk management is an integral part of all organisational activities, not a separate function.
2. **Structured and comprehensive** — a systematic, timely, and structured approach produces consistent and comparable results.
3. **Customized** — the framework and process are tailored to the organisation's external and internal context, objectives, and risk profile.
4. **Inclusive** — stakeholders are appropriately involved; their knowledge and views improve risk management.
5. **Dynamic** — risks emerge, change, and disappear; risk management anticipates, detects, and responds to change and events.
6. **Best available information** — inputs are based on historical and current information and forward-looking expectations, with limitations made explicit.
7. **Human and cultural factors** — human behaviour and culture influence every stage; they are recognised and accounted for.
8. **Continual improvement** — risk management is improved through learning and experience.

### 3.4 The framework and the process

**The framework** (2018) is the organisational layer — how risk management gets *embedded*: **leadership and commitment** at the centre (the board and top management set the tone, assign authority, and provide resources); then **integration** (risk management woven into the organisation's governance, strategy, and planning); **design** (understanding context, articulating commitment, assigning roles, establishing communication and accountability); **implementation** (making the framework operational); **evaluation** (measuring whether the framework is working); and **improvement** (adapting it as the organisation and its context change). The framework is a loop, not a ladder — evaluation feeds improvement, which feeds re-design.

**The process** (2018) is the workflow applied to any specific risk: **communication and consultation** (with stakeholders, throughout); **scope, context and criteria** (defining what is in scope, the internal/external context, and the risk criteria against which risk will be evaluated); **risk assessment** — itself three sub-steps: *risk identification* (finding risks that could affect objectives), *risk analysis* (understanding the nature, likelihood, and consequences, and the effectiveness of existing controls), *risk evaluation* (comparing the analysis against the criteria to decide which risks need treatment and their priority); **risk treatment** (selecting and implementing options — avoid, take/increase, remove the source, change likelihood, change consequences, share, retain — the ISO analogue of COSO's avoid/reduce/share/accept); **monitoring and review** (watching the risk and the process, on an ongoing basis); and **recording and reporting** (documenting the process and its outcomes for decision-makers and stakeholders). The process is a cycle too: treatment changes the risk, which is re-assessed, which may require new treatment.

### 3.5 The ISO table

| Element | ISO 31000:2009 | ISO 31000:2018 |
|---|---|---|
| **Title** | *Risk management — Principles and guidelines* | *Risk management — Guidelines* |
| **Published** | November 2009 (verified) | Mid-February 2018 (verified) |
| **Basis** | Developed from AS/NZS 4360:2004 as working draft (verified) | Revision of the 2009 edition |
| **Purpose** | "Principles and generic guidelines on risk management", any organisation, any risk | Same scope; purpose declared as "creating and protecting value" |
| **Principles** | 11 principles (2009 list, incl. value creation, integrated, systematic, tailored) | 8 principles: integrated; structured and comprehensive; customized; inclusive; dynamic; best available information; human and cultural factors; continual improvement |
| **Framework** | Mandate and commitment; design of framework; implementing; monitoring and review; continual improvement | Leadership and commitment (centre); integration; design; implementation; evaluation; improvement |
| **Process** | Communication & consultation; establishing the context; risk assessment (identification/analysis/evaluation); risk treatment; monitoring & review | Communication & consultation; scope, context & criteria; risk assessment (identification/analysis/evaluation); risk treatment; monitoring & review; recording & reporting |
| **Companion standard** | ISO/IEC 31010:2009 (risk assessment techniques) | IEC 31010:2019 (re-issued; ~30+ techniques with selection guidance) |
| **Certification** | None (guidelines) | None (guidelines) |
| **Typical use** | Generic risk management worldwide; enterprise-risk programmes in non-financial sectors; the process vocabulary that regulators and banks also borrow | Same, plus the reference vocabulary for risk processes in audit, cybersecurity (see ../technology/cybersecurity_guide.md), and operations contexts |

---

## 4. The Risk Taxonomy: Seven Families, One Enterprise

### 4.1 Why a taxonomy

Every ERM framework needs a map of the risk universe — the **risk taxonomy** — because "manage risk to within appetite" is meaningless until the risks have names, owners, and measurement homes. In a bank the taxonomy has two masters. The first is the Basel Committee, whose four pillar families — **credit, market, operational, liquidity** — are wired into the capital framework itself (each family has its own RWA treatment; see basel_regulatory_capital_guide.md §4 and risk_management_models_guide.md §2). The second is the ERM discipline, which needs a *complete* map of what could hurt the enterprise — which is why bank ERM taxonomies extend the Basel four with the strategic, reputational, and compliance families that Basel deliberately leaves outside Pillar 1. The families below are the seven used throughout this guide and the risk cluster.

### 4.2 The families

**Credit risk** — the risk of loss from a borrower's or counterparty's failure to meet its obligations. The dominant RWA driver for a commercial/CIB bank and the most model-mature family: PD/LGD/EAD models, scorecards, structural and reduced-form default models, portfolio credit models, and CVA for counterparty credit risk (the full model machinery is risk_management_models_guide.md §3). ERM cares about credit risk at the *portfolio* level: concentrations, correlation with other families, and the tail of the aggregate loss distribution.

**Market risk** — the risk of loss from adverse movements in market prices: rates, FX, equities, commodities, spreads, volatilities. Trading-book market risk is measured with VaR/expected-shortfall and regulated under FRTB (risk_management_models_guide.md §4); banking-book interest-rate risk (IRRBB) is an ALM concern (treasury_alm_guide.md). ERM cares about the *interaction*: a market shock that moves credit spreads, funding costs, and collateral values simultaneously is the classic enterprise-view scenario.

**Operational risk** — the risk of loss from inadequate or failed internal processes, people, and systems, or from external events; includes legal risk (Basel II definition; risk_management_models_guide.md §5). It is the family with the longest tail of mundane causes — fraud, outages, failed controls, third-party failures — and the family where the new technology risks (cyber, AI) land: ../technology/cybersecurity_guide.md maps the security side and ai_genai_banking_compliance_guide.md maps the AI side, both as operational-risk overlays.

**Liquidity risk** — the risk that the bank cannot meet its obligations as they fall due without unacceptable losses: funding liquidity risk (cannot raise cash) and market liquidity risk (cannot sell without moving the price). Basel III's LCR and NSFR made it quantitative (treasury_alm_guide.md is the dedicated treatment). Liquidity is the family that most brutally demonstrates the enterprise view: it is not a single desk's risk, it is the whole balance sheet's.

**Strategic risk** — the risk that the bank's strategy, business model, or key strategic assumptions prove wrong: a bet on a business line that fails, a market entry that misreads demand, a technology disruption that outdates the franchise. It is the least model-mature family — measured qualitatively through strategy reviews, scenario analysis, and board challenge — and it is the family the COSO 2017 framework is most explicitly about (strategy and objective-setting, P6–P9).

**Reputational risk** — the risk that negative stakeholder perception — customers, investors, regulators, the public — damages the franchise: loss of trust, customer attrition, funding pressure, regulatory response. It is largely unmodellable and is managed qualitatively (conduct standards, communication protocols, the "front-page test"), and it is usually a *consequence* amplifier: almost every material operational or conduct failure becomes a reputational event too.

**Compliance risk** — the risk of legal or regulatory sanction, financial loss, or reputational damage from failure to comply with laws, regulations, and internal policies: AML/CFT, sanctions, market conduct, data protection, prudential reporting. The systems landscape is financial_risk_compliance_systems_guide.md; the regulatory-technology layer is regtech_guide.md. Compliance risk is the family where "risk" and "obligation" blur — much of it is binary (you are compliant or you are not) rather than a spectrum.

### 4.3 Why seven, not four

The Basel four are the *capital-relevant* families; the ERM seven are the *enterprise-relevant* ones. The three extra families matter for three reasons. First, they are where strategy failures and trust failures live — the risks that kill firms rather than quarters. Second, they are the families supervisors explicitly expect to see in the ICAAP: Pillar 2 is defined as covering *all* material risks, including those outside Pillar 1 (basel_regulatory_capital_guide.md §8). Third, they are the families where quantification is weakest, which means the governance and appetite machinery (§5–6) must carry more of the weight — you cannot manage what you cannot measure, so you manage it with limits, indicators, and oversight instead.

### 4.4 The taxonomy table

| Risk | Focus | Cross-reference in the cluster |
|---|---|---|
| **Credit risk** | Obligor/counterparty default; the dominant RWA driver; portfolio concentrations | risk_management_models_guide.md §3 (PD/LGD/EAD, IRB); cecl_guide.md (IFRS 9/CECL provisioning); basel_regulatory_capital_guide.md §4 |
| **Market risk** | Price/rate/vol moves; trading book (VaR/ES, FRTB) and banking book (IRRBB) | risk_management_models_guide.md §4; treasury_alm_guide.md (IRRBB/ALM); basel_regulatory_capital_guide.md §4 (FRTB) |
| **Operational risk** | Failed processes, people, systems, external events; incl. legal risk; the home of cyber and AI risk | risk_management_models_guide.md §5 (SMA); ../technology/cybersecurity_guide.md; ai_genai_banking_compliance_guide.md |
| **Liquidity risk** | Inability to meet obligations as they fall due; funding and market faces | treasury_alm_guide.md (LCR/NSFR, the dedicated deep-dive); risk_management_models_guide.md §6 |
| **Strategic risk** | Strategy/business-model failure; assumptions prove wrong | This guide §2 (COSO 2017 strategy components); banks_in_singapore_guide.md (MAS strategy-and-risk expectations) |
| **Reputational risk** | Stakeholder-perception damage; the consequence amplifier | This guide §5 (governance response); conduct content in financial_risk_compliance_systems_guide.md |
| **Compliance risk** | Legal/regulatory sanction from non-compliance; AML, sanctions, conduct, data protection | financial_risk_compliance_systems_guide.md; regtech_guide.md; banks_in_singapore_guide.md (MAS) |

---

### 4.5 The overlays: the risks that route through the seven

The seven families are the trunk, but real bank risk taxonomies carry a set of **overlay risks** that route *through* the families rather than beside them — each is managed inside one of the seven with its own framework and systems:

- **Counterparty credit risk (CCR)** — the credit risk of derivatives and repo exposure, including CVA; a credit-family sub-discipline with its own capital treatment (basel_regulatory_capital_guide.md §4; risk_management_models_guide.md §3).
- **IRRBB (interest-rate risk in the banking book)** — a market/ALM overlay on the banking book, measured by EVE/NII sensitivity (treasury_alm_guide.md).
- **Concentration risk** — a credit overlay (single-name, sector, country) managed by limits; the FSB's RAF principles expect concentration explicitly in appetite.
- **Model risk** — the risk that decisions rest on models that are wrong or misused; governed under SR 11-7 and owned as its own second-line discipline (risk_management_models_guide.md §9).
- **Conduct risk** — the risk that behaviour — sales practices, market conduct, incentives — harms customers or markets; a compliance/reputational overlay with its own regulatory attention.
- **Climate risk** — physical and transition risk transmitting through credit, market, and operational families; increasingly in supervisory stress tests (cecl_guide.md touches the credit angle).
- **AI/GenAI risk** — model risk, data risk, and operational risk in the AI estate; the requirements map is ai_genai_banking_compliance_guide.md.

For the taxonomy's data model (§10.3), overlays are *tags* on family-tagged risks — a risk can be "credit + concentration + climate". The enterprise view is assembled by cutting along any of those dimensions; the seven families keep the taxonomy stable while the overlays keep it current.

---

## 5. The Risk Governance: Three Lines, Board, CRO, Committees

### 5.1 The three lines of defence (and the 2020 Three Lines Model)

The organising principle of risk governance is the **three lines of defence**, a model promulgated by the Institute of Internal Auditors (IIA). The original formulation — the IIA position paper *The Three Lines of Defense in Effective Risk Management and Control* (2013) — assigns risk accountability in three layers:

- **First line — operational management.** The business lines own and manage risk day-to-day: they take the risk, they run the controls, they are accountable for staying within limits. In a bank: the trading desks, the lending businesses, the operations functions.
- **Second line — risk management and compliance functions.** The oversight functions set frameworks, monitor risk against appetite, challenge the first line, and advise: group risk management (credit, market, operational risk functions), compliance, model risk management, and increasingly the data office. The **CRO** sits here.
- **Third line — internal audit.** Independent assurance: audit evaluates whether the first two lines are doing what the board believes they are doing. It reports to the board (via the audit committee), not to management — independence is the point.

In **July 2020** the IIA updated the model with a new position paper, *The IIA's Three Lines Model: An update of the Three Lines of Defense* (verified — September 2020 IIA publication page), renaming it the **Three Lines Model** and strengthening three things: an explicit **governing body** (the board) that sits above and outside the three lines, holding ultimate accountability and oversight; clearer language that the lines are *roles* that can be organised in different ways (not rigid departments); and the notion that all three lines, plus the governing body, exist to achieve the organisation's objectives — the six principles of the model all serve that end. In the 2020 model: the **governing body** (board) ensures accountability and oversight; **management** (first line: deliver value, second line: monitor and advise); **internal audit** (third line: independent assurance). The cybersecurity guide in this series uses the same model for security governance (../technology/cybersecurity_guide.md), and the AI-governance overlay in ai_genai_banking_compliance_guide.md slots the AI risk function into the same three lines — the model is the series' shared governance vocabulary.

### 5.2 The board

The board (in a bank, usually via a dedicated **board risk committee**) holds the top of the ERM stack. Its duties, in the COSO 2017 frame (P1: exercises board risk oversight) and in supervisory expectations alike:

- **Approve the risk appetite statement** and any material changes to it (the FSB's RAF principles make this explicit for systemically important firms, §6).
- **Oversee strategy and its risk implications** — the board approves the strategy *with* its risk profile attached, not after the fact.
- **Hold management accountable** — through the CRO's independent reporting line, the audit committee's assurance, and the remuneration committee's risk-adjusted incentives.
- **Review the risk profile** — a standing board-level risk report (RAG status vs appetite, top risks, incidents, emerging risks — the content list in §6.4 is a template).

The governance question that separates healthy from unhealthy banks is structural: does the **CRO report independently to the board** (directly or dotted-line, with a reporting line that cannot be severed by a business head), or does the CRO sit under the CFO or the business? The former is the supervisory expectation (and the FSB principles); the latter is the structural failure mode flagged in the COSO implementation literature.

### 5.3 The CRO

The **chief risk officer** is the second line's leader and the board's window into the risk profile. The role spans: owning the risk framework and the risk appetite framework; chairing or convening the risk committees; signing off new products and material risk-taking; overseeing the specialist risk functions (credit, market, operational, liquidity); and — critically in a bank — owning the ICAAP's risk assessment that feeds Pillar 2 (basel_regulatory_capital_guide.md §8). The CRO's independence (board access, removal only with board knowledge, remuneration not tied to business-line P&L) is a supervisory topic in every jurisdiction, MAS included (banks_in_singapore_guide.md).

### 5.4 The risk committees

Below the board, risk governance runs through a committee lattice. The standard bank structure:

- **Board risk committee** — board-level; appetite approval, risk-profile oversight, framework approval.
- **Group risk committee** (management-level, chaired by the CRO) — the operational apex: aggregate risk profile, appetite breaches, new-product approvals, framework changes.
- **Credit committee / market risk committee / operational risk committee** — the specialist forums where the family-level risk and limit decisions are taken; the systems behind them are the family-specific platforms in financial_risk_compliance_systems_guide.md.
- **ALCO (asset and liability committee)** — the liquidity/ALM forum (treasury_alm_guide.md); in many banks it is where balance-sheet risk meets the market-risk and funding views.
- **Model risk committee** — the SR 11-7 governance apex for models (risk_management_models_guide.md §9); the committee that approves model changes and validates challenges.
- **New-product approval / product governance committees** — the gate through which any new product, market, or platform passes a risk review before launch (the operational form of COSO P8's "evaluates alternative strategies").

The committee lattice is also where the *enterprise* view is actually taken: the group risk committee is the only forum where the credit view, the market view, the liquidity view, and the operational view are put on the same table and read as one risk profile.

### 5.5 Risk culture and conduct: the tone at the top

Governance structures work only if the culture underneath them does — the lesson of every conduct scandal and of COSO 2017's decision to make culture a component (P3–P4). **Risk culture** is the shared set of behaviours and attitudes toward risk: whether bad news travels up, whether limits are respected or gamed, whether risk-taking is rewarded net of risk or gross of it. The practical elements:

- **Tone at the top** — the board and executive committee model how risk information is treated: blame or problem-solving, challenge or deference. The COSO literature's blunt formulation: if management punishes bearers of bad news, risk accumulates silently until it is a crisis.
- **Incentive alignment** — remuneration is the culture's thermostat: risk-adjusted performance (RAROC-style) rather than raw revenue, with malus and clawback provisions; the remuneration committee's risk linkage is a supervisory topic in banking.
- **Conduct frameworks** — the explicit standards of behaviour (market conduct, customer treatment, conflict handling) with enforcement; conduct risk is the behavioural face of the compliance and reputational families (§4).
- **Speak-up and escalation channels** — whistleblowing mechanisms, no-retaliation policies, and the documented expectation that limit breaches and near-misses are reported — measured by what happens to the reporter, not the policy text.
- **Culture indicators** — audit findings, escalation rates, control-failure trends, turnover in risk roles, breach-reporting timeliness: the measurable proxies that let the board track culture (COSO P20's "reports on risk, culture, and performance").

The AI-governance overlay extends the same logic: model behaviour, bias, and hallucination risk are culture-and-conduct questions as much as technical ones (ai_genai_banking_compliance_guide.md §3). The three lines define who does what; culture determines whether they actually do it.

### 5.6 The governance table

| Layer | Role | Accountable for | Key artefacts |
|---|---|---|---|
| **Board / board risk committee** | Ultimate oversight; appetite and strategy approval | The enterprise risk profile vs appetite | Risk appetite statement (approved), board risk report |
| **CRO** | Second-line leadership; independent board window | Framework quality, aggregate risk profile, ICAAP risk assessment | Risk framework, risk appetite framework, ICAAP submission |
| **First line (business)** | Own and manage risk day-to-day | Staying within limits; control effectiveness | Risk registers, limit usage, RCSA outputs |
| **Second line (risk & compliance)** | Frameworks, monitoring, challenge, advice | Oversight quality; appetite adherence | Policies, limits, KRIs, stress results |
| **Third line (internal audit)** | Independent assurance | Honest evaluation of lines 1–2 | Audit opinions, findings, follow-up |
| **Risk committees** | Collective risk decisions at each level | Decisions within mandate and appetite | Committee minutes, limit approvals, new-product approvals |

---

## 6. The Risk Appetite: Statements, Tolerance, and the RAG

### 6.1 What risk appetite is

**Risk appetite** is the amount and type of risk an organisation is willing to pursue or retain in order to achieve its objectives. The canonical definitions:

- **ISO Guide 73:2009** (*Risk management — Vocabulary*): risk appetite is the "amount and type of risk that an organization is willing to pursue or retain"; **risk tolerance** is the "organization's or stakeholder's readiness to bear the risk after risk treatment in order to achieve its objectives".
- **The FSB** (*Principles for an Effective Risk Appetite Framework*, 18 November 2013 — verified): "A firm's risk appetite represents the aggregate level and types of risk a firm is willing to assume within its risk capacity to achieve its strategic objectives and business plan" — and it distinguishes **risk capacity** ("the maximum level of risk the firm can assume given its current level of resources") from appetite: capacity is the ceiling physics allows; appetite is the ceiling the board chooses.

For a bank the appetite concept is load-bearing: the COSO 2004 definition of ERM itself is "manage risk to be within its risk appetite" (§1), and the FSB made the **risk appetite framework (RAF)** — the written framework, statement, limits, and governance around appetite — a supervisory artefact for systemically important financial institutions: the FSB principles cover (i) an effective RAF, (ii) an effective risk appetite statement (RAS), (iii) risk limits, and (iv) the roles of the board and senior management (all verified against the FSB publication).

### 6.2 Appetite vs tolerance vs limit

The vocabulary stack, from the top:

| Term | Definition (source) | In practice |
|---|---|---|
| **Risk capacity** | The maximum risk the firm *could* assume given its resources (FSB) | The hard physical ceiling — capital, funding, franchise |
| **Risk appetite** | The risk the firm *is willing* to assume to achieve its strategy (FSB; ISO Guide 73) | Board-approved statements, per risk family and aggregate |
| **Risk tolerance** | The readiness to bear risk after treatment to achieve objectives (ISO Guide 73) | Acceptable deviation bands around appetite/objectives — the buffer inside which day-to-day fluctuation is acceptable |
| **Risk limits** | Binding operational thresholds set within appetite/tolerance | Desk limits, counterparty limits, country limits, KRI thresholds — the enforcement layer |

The relationship is nested: **capacity > appetite > tolerance > limits**. A bank can physically take more risk than it is willing to (capacity exceeds appetite — that is the point of a capital buffer); within appetite it tolerates fluctuation up to defined bands; within tolerance the day-to-day limits bind. The FSB's own framing puts the RAF as the mechanism that cascades the board-level appetite into limits at desk level — the "cascade" is the design problem §10 works through.

### 6.3 The risk appetite statement (RAS)

The **risk appetite statement** is the written expression of appetite: a board-approved document stating the aggregate level and types of risk the firm is willing to assume, expressed qualitatively (narrative on risk philosophy and culture) and quantitatively (metrics per risk family with thresholds). A bank-grade RAS typically has: a preamble on mission and strategy; aggregate statements (e.g., "the bank maintains a strong capital position, targeting a CET1 ratio above the regulatory minimum plus buffers, and never allows capital to fall below the ICAAP-stressed requirement"); per-family statements with metrics (credit: "expected loss within provisioning plan; concentration limits per the credit policy"; market: "VaR/ES within the trading-risk appetite and stress losses within the capital plan"; liquidity: "LCR above 100% plus management buffer, NSFR above 100%"); and reputational/compliance statements ("no material regulatory findings; zero tolerance for sanctions breaches"). The FSB principles require the RAS to be approved by the board, communicated across the firm, and linked to the strategic plan and capital planning — the ICAAP (§8) is where the linkage becomes quantitative.

The recurring implementation failure — flagged in the COSO literature (§2.2) and in the RIMS risk-appetite guidance — is an **aspirational but unmeasurable RAS**: "we maintain a conservative risk posture" tells a desk nothing. The discipline is: quantify by family, set thresholds, connect to limits and KRIs, and monitor.

### 6.4 The RAG: red-amber-green monitoring **[verify — flagged]**

The **RAG** is the traffic-light convention used to monitor risk against appetite in board reporting: a metric is **green** (within appetite), **amber** (approaching the tolerance boundary — watch and respond), or **red** (appetite/tolerance breached — escalation and remediation required). It is the standard presentation device of risk dashboards and the board risk report, and it appears throughout the risk cluster (risk_data_aggregation_guide.md §4 covers the reporting practices it colours). Two flags, honestly stated:

- **"RAG" as an acronym is industry convention, not a formal framework component.** No standard (COSO, ISO, FSB) defines "RAG" as a named element; it is the universal *presentation* convention for appetite monitoring. **[verify]** — if a source in your organisation uses "RAG" to mean "risk appetite grid", treat it as a house-specific artefact, not an industry-standard one.
- **The colours need definitions to be meaningful.** A RAG dashboard without calibrated thresholds is decoration: each metric needs a documented green/amber/red definition tied to the appetite and tolerance bands (§6.2), and the escalation response to red must be pre-agreed (who is told, how fast, what authority is triggered). The FSB principles require exactly this linkage — appetite → limits → monitoring → escalation — as part of the RAF.

### 6.5 The appetite table

| Element | Definition | Source (verified) | Bank example |
|---|---|---|---|
| **Risk capacity** | Maximum risk assumable given resources | FSB (2013) | Capital, funding, and franchise ceilings |
| **Risk appetite** | Amount and type of risk willing to pursue/retain | ISO Guide 73:2009; FSB (2013); COSO (2004 definition) | "CET1 ≥ regulatory minimum + CCB + management buffer at all times" |
| **Risk tolerance** | Readiness to bear risk after treatment | ISO Guide 73:2009 | "LCR ≥ 130% (100% regulatory + 30% management buffer)" |
| **Risk limit** | Binding operational threshold | FSB (2013) RAF principles | "Single-name exposure ≤ 15% of Tier 1; desk VaR ≤ €Xm" |
| **RAS** | The written, board-approved statement of appetite | FSB (2013) — board approval required | The §10 worked example's statement |
| **RAF** | The framework around appetite: statement + limits + governance + cascade | FSB (2013) | The §10 worked example's framework |
| **RAG** | Red/amber/green monitoring convention | Industry convention **[verify — see §6.4]** | Board risk dashboard status per metric |

---

## 7. The Risk Process: Identify, Assess, Respond, Monitor, Report

### 7.1 The process as the unit of work

If the framework is the *what* and governance is the *who*, the **risk process** is the *how* — the workflow applied to every risk, every decision, every day. The canonical process is ISO 31000's (2018) — communication and consultation and monitoring and review wrapping a cycle of scope/context/criteria → risk assessment (identification, analysis, evaluation) → risk treatment (ISO 31000 §3.4 of this guide) — and its five load-bearing stages map cleanly onto the five the practice literature uses and this guide follows: **identification, assessment, response, monitoring, reporting**. (The banking world runs the same loop inside its own artefacts: the ICAAP is the enterprise-scale instance of it, §8.)

### 7.2 The five stages

**Identification** — finding the risks that could affect objectives: what could happen (events), where (sources), and how (causes and consequences)? Techniques: risk registers and workshops, RCSA (risk and control self-assessment) for operational risk, horizon scanning and emerging-risk reviews, loss-event analysis, scenario identification. COSO P10 adds the crucial instruction to capture *upside* risk — opportunities — alongside threats. Identification is exhaustive-or-bust: a risk that was never identified cannot be assessed, and the siloed-view failure mode of §1 is, at bottom, an identification failure at the enterprise level.

**Assessment** — understanding and prioritising the identified risks: *analysis* (likelihood and consequence, inherent vs residual, with the control environment taken into account) and *evaluation* (comparing against the risk criteria/appetite to decide priority). The techniques catalogue is IEC 31010:2019 (§3.1) — qualitative heat maps up to quantitative models. In a bank the assessment stage is where the quant machinery of risk_management_models_guide.md plugs in: PD/LGD/EAD for credit, VaR/ES and stress for market, LDA/SMA for operational, LCR/NSFR for liquidity — ERM *consumes* those numbers as the analysis under its evaluation. The output is a prioritised risk profile: the heat map, the top-risks list, and the RAG status per family (§6.4).

**Response (treatment)** — deciding what to do: COSO's four — **avoid, reduce, share, accept** — and ISO's richer menu (avoid, take/increase to pursue an opportunity, remove the source, change likelihood, change consequences, share, retain). Selection is cost–benefit and appetite-driven: the residual risk after response must be within appetite, or the response escalates. In a bank the response layer is concrete: limits, hedges, collateral, insurance, capital (the Pillar 2 capital held *against* residual risk — basel_regulatory_capital_guide.md §8), provisioning (cecl_guide.md), and controls.

**Monitoring** — watching risks, controls, and the process itself: KRI thresholds and RAG dashboards, limit-utilisation reviews, backtesting of models, incident and near-miss tracking, and the periodic re-assessment cycle (quarterly RCSA, annual stress-test and ICAAP cycles). Monitoring is also where "assesses substantial change" (COSO P15) lives: acquisitions, new products, regulatory shifts, and incidents trigger an out-of-cycle re-assessment.

**Reporting** — putting the process's output in front of decision-makers: the board risk report (RAG status vs appetite, top risks, incidents, emerging risks), regulatory reporting (COREP, MAS returns — regtech_guide.md and financial_risk_compliance_systems_guide.md §9), and the risk disclosures. Reporting is where the enterprise view becomes a *recurring artefact* — and where BCBS 239's reporting principles (risk_data_aggregation_guide.md §4) bind: accurate, complete, timely, clear, and frequency-matched to the decision.

### 7.3 The process table

| Stage | Question | Techniques (IEC 31010) | Bank artefacts | COSO 2017 principle |
|---|---|---|---|---|
| **Identification** | What could affect our objectives? | Brainstorming, structured interviews, checklists, scenario identification, RCSA | Risk registers, RCSA, emerging-risk register, new-product reviews | P10 (identifies risk) |
| **Assessment** (analysis + evaluation) | How big, how likely, how urgent? | Heat maps, bow-tie, FMEA, fault/event trees, Monte Carlo, stress testing | Risk heat map, model outputs (PD/VaR/ES/LCR), top-risks list | P11 (severity), P12 (prioritise) |
| **Response (treatment)** | What will we do? | Cost–benefit analysis, decision analysis | Limits, hedges, controls, insurance, capital, provisioning | P13 (risk responses) |
| **Monitoring** | Is it still true? | KRI/KPI dashboards, audits, backtesting, trend analysis | KRI RAG dashboard, limit reports, model backtests, incident log | P15 (substantial change), P16 (review) |
| **Reporting** | Who needs to know, how fast? | — (the reporting discipline itself) | Board risk report, ICAAP, COREP/MAS returns, disclosures | P19 (communicate), P20 (report) |

---

### 7.4 The risk register and RCSA in practice

Two artefacts carry the process day-to-day. The **risk register** is the enterprise inventory of identified risks — each entry tagged with family, owner (first line), monitor (second line), appetite line, inherent/residual scores, controls, and action plan; it is the data model behind the enterprise view (§10.3) and lives in a system, not a spreadsheet (the spreadsheet register is the recurring failure mode in the COSO implementation literature). The **RCSA (risk and control self-assessment)** is the first line's periodic self-audit per process or function: catalogue the processes, identify the risks, map the controls, score inherent and residual, and commit to action. RCSA is operational risk's workhorse (risk_management_models_guide.md §5 uses its outputs for loss/scenario calibration) and it is where "identify" and "assess" become routine rather than annual. Both artefacts feed the monitoring stage: KRI thresholds and control-testing results update the register's scores, and material changes trigger the "substantial change" re-assessment (COSO P15).

For architects, the register and RCSA are *workflow applications*: role-based entry and approval, versioned assessments, control evidence attachments, and dashboards — the same pattern as the model-inventory workflow in risk_management_models_guide.md §9, applied to risks instead of models.

---

## 8. The Banking ERM: ICAAP, Pillar 2, and BCBS 239

### 8.1 Why bank ERM is different

Bank ERM is ERM with the volume turned up and the regulator in the room. The discipline is the same — enterprise-wide, strategy-linked, appetite-governed — but three things make the banking expression distinctive:

1. **It is capital-anchored.** The enterprise's ultimate risk capacity is its capital, and the bank's own assessment of whether capital is adequate for its risk profile — the **ICAAP** — is a supervisory deliverable under Pillar 2 (basel_regulatory_capital_guide.md §8). Bank ERM and bank capital are the same discipline seen from two ends: the risk process produces the risk profile; the ICAAP asks whether capital covers it.
2. **It is data-anchored.** The enterprise view requires enterprise data: BCBS 239 (January 2013, 14 principles) made risk-data aggregation and risk reporting a supervised capability in its own right (risk_data_aggregation_guide.md). Bank ERM without BCBS 239-grade aggregation is a board deck built on spreadsheets — and supervisors now treat that as a defect.
3. **It is supervised.** The FSB's RAF principles (2013), the BCBS's corporate-governance principles, and national supervisors (MAS in Singapore — banks_in_singapore_guide.md) all review the *framework itself*: appetite, limits, governance, and the board's engagement. A bank's ERM programme is an examinable artefact.

### 8.2 The ICAAP / Pillar 2 linkage

Under Basel II's three-pillar structure (basel_regulatory_capital_guide.md §1.3, verified facts), **Pillar 2 — supervisory review** has two halves: the bank's **ICAAP** (Internal Capital Adequacy Assessment Process — the bank's own assessment of the capital it needs for *all* its risks) and the supervisor's **SREP** (Supervisory Review and Evaluation Process — the supervisor's assessment, concluding in P2R/P2G, the Pillar 2 capital requirements and guidance). The ICAAP is the ERM process in capital form:

- It must cover **all material risks** — including the Pillar-1 families and the strategic, reputational, and compliance risks that Pillar 1 does not charge — which is exactly the ERM taxonomy of §4.
- It is **appetite-linked**: the FSB's principles require the RAF to be integrated with capital planning — the RAS thresholds (capital, liquidity, earnings) are the same numbers the ICAAP stress-tests.
- It is **stress-based**: the ICAAP runs the risk profile through adverse scenarios (the stress-testing machinery of risk_management_models_guide.md §8) and asks whether capital survives; the answer sets the internal capital target, which the RAS then expresses as the management buffer over the regulatory minimum.
- It is **board-governed**: the board approves the ICAAP and the internal capital target, and the SREP evaluates both the analysis and the governance around it.

For a solution architect the ICAAP is a *reporting and modelling programme*: scenario engines, risk aggregation, capital projection, and the board pack — the data and reporting spine of risk_data_aggregation_guide.md feeding the capital calculations of basel_regulatory_capital_guide.md §9.

### 8.3 The BCBS 239 reporting linkage

BCBS 239 (*Principles for effective risk data aggregation and risk reporting*, January 2013 — verified facts in risk_data_aggregation_guide.md) is the data-side twin of ERM: its definition of risk data aggregation is explicitly framed against "the bank's risk reporting requirements to enable the bank to measure its performance against its risk tolerance/appetite". The fourteen principles in four sections — governance/infrastructure (P1–P2), aggregation capabilities (P3–P6), reporting practices (P7–P11), supervision (P12–P14) — are the engineering specification for the enterprise view:

- **Aggregation principles** (accuracy, comprehensiveness, timeliness, adaptability) are what make "the aggregate risk of the enterprise" a computable number at group level, per business line and per legal entity — the P14 portfolio view of COSO, made operational.
- **Reporting principles** (accuracy, clarity, frequency, distribution, comprehensiveness of reporting) are what make the board risk report a reliable instrument — the RAG dashboard of §6.4 is only as good as the reporting spine under it.
- **Governance principles** are the data-office mirror of the three lines (§5): a data governance function, independent of the businesses, with board-level accountability.

The practical chain: ERM defines appetite (§6) → the risk process produces the risk profile (§7) → BCBS 239 aggregation assembles the firm-wide numbers → the ICAAP sizes capital against them (§8.2) → the board sees the whole in the RAG report. Break any link and the enterprise view degenerates into the siloed view — which is precisely what the post-2008 supervisory agenda was built to prevent.

### 8.4 The bank ERM architecture (and where the systems fit)

| ERM layer | Bank expression | Systems/guides |
|---|---|---|
| **Risk appetite** | RAS + RAF, board-approved; ICAAP-linked capital and liquidity buffers | This guide §6; basel_regulatory_capital_guide.md §8 |
| **Risk taxonomy** | Basel four + strategic/reputational/compliance; ICAAP risk inventory | This guide §4; risk_management_models_guide.md §2 |
| **Risk measurement** | PD/LGD/EAD, VaR/ES, SMA, LCR/NSFR, stress models | risk_management_models_guide.md (all families); treasury_alm_guide.md |
| **Risk governance** | Board risk committee, CRO, three lines, specialist committees | This guide §5; financial_risk_compliance_systems_guide.md; ../technology/cybersecurity_guide.md (security's three lines) |
| **Risk data aggregation** | BCBS 239 principles; lineage, reconciliation, data quality | risk_data_aggregation_guide.md (the dedicated deep-dive) |
| **Capital assessment** | ICAAP → internal capital target; SREP/P2R/P2G | basel_regulatory_capital_guide.md §8 |
| **Compliance and conduct** | Compliance risk programme, AML/sanctions, conduct frameworks | financial_risk_compliance_systems_guide.md; regtech_guide.md |
| **Emerging risks** | AI risk, cyber risk, climate risk — operational-risk overlays | ai_genai_banking_compliance_guide.md; ../technology/cybersecurity_guide.md; cecl_guide.md (climate-credit angle) |

---

### 8.5 The MAS angle: Singapore expectations

For the Singapore branch of a Cymbal Bank, the ERM framework has a second supervisor in the room. MAS's expectations for risk governance and risk management — set out in its corporate-governance guidelines, technology risk management notice, and supervisory expectations for banks (the verified specifics live in banks_in_singapore_guide.md) — converge on the same machinery this guide describes: board-level risk oversight and a board risk committee; a CRO with independence; a risk appetite framework integrated with capital planning; stress testing as a management tool; and (via MAS Notice 637 and the Basel implementation wave) the full Pillar 2/ICAAP apparatus under the finalised Basel framework (basel_regulatory_capital_guide.md §7). The practical consequence for the framework design in §10: the RAS, the governance structures, and the ICAAP must be *dual-regulator-legible* — the same documents must answer to the ECB-style group supervisor and to MAS, which is itself a design constraint on how appetite is cascaded to the Singapore legal entity and how the RAG reporting is localised.

---

## 9. The Comparison: COSO vs ISO 31000

### 9.1 The head-to-head

COSO ERM and ISO 31000 are the two dominant ERM frameworks, and the practitioner question is never "which is correct" — both are correct, both are voluntary, and both describe the same underlying discipline from different vantage points. The honest framing: **COSO is a governance-and-strategy framework written for boards of large, regulated organisations; ISO 31000 is a principles-and-process standard written for any organisation that faces uncertainty.** They converge on the substance (risk to objectives, appetite, process, monitoring) and diverge on structure, prescriptiveness, and emphasis.

The verified comparison points:

- **Origin and DNA.** COSO ERM grew out of the internal-control tradition (COSO 1992, SOX-era audit culture) and is owned by a US professional-accounting consortium; ISO 31000 grew out of the Australian/New Zealand risk-management tradition (AS/NZS 4360) and is owned by the international standards body. That ancestry shows: COSO is audit-shaped (components, principles, board oversight, documentation), ISO is management-system-shaped (principles, framework, process, continual improvement).
- **Prescriptiveness.** COSO 2017 gives five components and twenty auditable principles — a checklist-shaped structure that regulated firms can map their programmes onto; ISO 31000 gives principles and guidelines with no mandatory elements — deliberately flexible, deliberately light.
- **Strategy emphasis.** COSO 2017 is *about* strategy: its second component is strategy and objective-setting (P6–P9) and its title announces the integration. ISO 31000 treats strategy as context — risk management supports objectives, but the standard does not prescribe how strategy and risk should interlock.
- **Governance detail.** COSO specifies board risk oversight (P1), the CRO's independence in effect, and the appetite-definition duty (P7); ISO 31000's framework puts leadership and commitment at the centre but leaves the governance structure to the organisation.
- **Process.** Both describe the same loop (identify → analyse → evaluate → treat → monitor → communicate); ISO's process section is the more detailed and is the one the risk profession quotes (and that this guide used in §7); COSO's process content is distributed across the Performance component's principles.
- **Certification and audit.** Neither framework certifies; ISO 31000-aligned management systems can be audited/certified by third parties (like other ISO management standards), while COSO ERM is typically evidenced through internal-control and internal-audit review (SOX-adjacent in the US).
- **Ecosystem.** COSO sits in the US financial-services and audit ecosystem (and its internal-control sibling, COSO ICIF 2013, is the SOX reference); ISO 31000 sits in the international standards ecosystem and pairs with IEC 31010 (techniques), ISO 31022 (legal risk), ISO 31030 (travel risk) — a family of sector standards that COSO does not have.

### 9.2 The choice

The selection logic used in practice: **regulated financial institutions and US-listed firms** default to COSO ERM because its governance detail, board emphasis, and audit-friendliness match supervisory and SOX expectations (and because the cluster's banking context — ICAAP, RAF, three lines — reads naturally in COSO's vocabulary); **general organisations, and firms already inside the ISO management-system world** (ISO 9001, ISO 27001, ISO 22301) default to ISO 31000 because it slots into their existing management-system architecture; and **many firms run a hybrid** — ISO 31000's process vocabulary for the day-to-day risk process, COSO's components and principles for the governance and board layer. The worked example in §10 is effectively a hybrid: COSO-shaped governance and appetite, ISO-shaped process, bank regulation supplying the binding requirements.

### 9.3 The comparison table

| Dimension | COSO ERM (2017) | ISO 31000 (2018) |
|---|---|---|
| **Full title** | *Enterprise Risk Management — Integrating with Strategy and Performance* | *Risk management — Guidelines* |
| **Owner / origin** | COSO (US professional-accounting consortium); internal-control ancestry | ISO/TC 262; AS/NZS 4360 ancestry |
| **Published** | September 2017 (replacing 2004) | February 2018 (replacing 2009) |
| **Structure** | 5 components + 20 principles (5/4/5/3/3) | 8 principles + framework (leadership/design/implement/evaluate/improve) + process |
| **Prescriptiveness** | Component-and-principle architecture; auditable | Guidelines; no mandatory elements |
| **Strategy link** | Core thesis: risk integrated with strategy-setting (P6–P9) | Contextual: risk management supports objectives |
| **Governance emphasis** | Board risk oversight (P1), CRO, appetite definition (P7), culture (P3–P4) | Leadership and commitment at framework centre; governance structure left open |
| **Process detail** | Distributed across Performance principles (P10–P14) | Dedicated process section (scope/context/criteria → assessment → treatment → monitor → record) |
| **Appetite treatment** | Explicit principle (P7: defines risk appetite) | Risk criteria in the process; appetite vocabulary via Guide 73 |
| **Best fit** | Large/regulated firms, financial services, SOX-relevant environments | Any organisation, any sector; ISO management-system ecosystems |
| **Typical pairing** | Internal control (COSO ICIF), audit committees, ICAAP-style governance | IEC 31010 techniques, ISO 27001/9001/22301 family |
| **Certification** | None (framework); evidenced via audit | None (standard); certifiable management-system implementations |

---

## 10. The Worked Example: An ERM Framework for a Cymbal Bank

### 10.1 The scenario

The bank is a **Cymbal Bank global corporate and investment bank**: headquarters in Paris, a major APAC hub in Singapore, and a balance sheet of roughly €400bn across four businesses — global markets (rates, FX, credit, commodities), structured finance (project, asset, and acquisition finance), trade finance, and corporate banking. The group parent applies full Basel III (EU CRR3 from January 2025 — basel_regulatory_capital_guide.md §7), the Singapore branch is supervised by MAS with its own expectations (banks_in_singapore_guide.md), and the group is a G-SIB-adjacent entity subject to FSB expectations on risk appetite frameworks. The context is the familiar one from credit_agricole_software_systems_guide.md: a universal-bank-owned CIB whose risk profile is dominated by credit and market risk, with a heavy derivatives and collateral footprint and a growing technology dependency.

The board has commissioned a redesign of the ERM framework: the existing one is a 2008-era document stack — a risk policy, a set of limits, and a compliance manual — that does not meet the FSB RAF principles, does not link appetite to the ICAAP, and cannot answer the enterprise question. The mandate: **a single ERM framework covering appetite, taxonomy, governance, and process, defensible to the board, to MAS, and to the ECB (as group supervisor), and implementable in the existing systems estate.** This section designs it; the design choices are deliberately mapped to the framework content of §1–§9 and cross-referenced to the cluster's deeper guides.

### 10.2 The design: risk appetite

**The RAF.** The board risk committee approves a risk appetite statement and a RAF built on the FSB's four elements (statement, limits, roles, monitoring — §6.1). The cascade: board-level aggregate statements → per-family appetites → tolerance bands → desk-level limits.

**The RAS (abridged, pedagogical numbers).** Preamble: the bank's purpose, its risk philosophy ("a relationship bank that takes credit and market risk within defined bounds, funded conservatively, operated with a strong control environment"), and the statement that appetite is set to preserve capital and franchise through stress.

- *Aggregate — capital:* "The bank maintains CET1 at or above the regulatory minimum plus the combined buffer plus a 100bp management buffer at all times, and the ICAAP-stressed capital projection never breaches the minimums." (Cross-ref: the capital machinery of basel_regulatory_capital_guide.md §3–§8.)
- *Aggregate — earnings:* "Annual net loss under the group stress scenario does not exceed 25% of Tier 1 capital."
- *Credit:* "Expected credit loss within the annual provisioning plan; single-name concentration ≤ 15% of Tier 1; sector concentration limits per the credit policy; no appetite for lending to customers above the defined watchlist criteria." (Cross-ref: cecl_guide.md for the ECL machinery; risk_management_models_guide.md §3 for the PD/LGD/EAD inputs.)
- *Market:* "Trading VaR (ES) within the approved limit envelope; stress losses within the capital plan; no unhedged proprietary positions outside the approved mandates." (Cross-ref: risk_management_models_guide.md §4.)
- *Liquidity:* "LCR ≥ 130% and NSFR ≥ 105% at group and entity level; funding plan maintains the ALCO-approved diversification." (Cross-ref: treasury_alm_guide.md.)
- *Operational:* "Aggregate annual operational loss ≤ €Xm; zero tolerance for sanctions and AML breaches; critical-system availability ≥ 99.9%." (Cross-ref: risk_management_models_guide.md §5; the cyber and AI overlays in ../technology/cybersecurity_guide.md and ai_genai_banking_compliance_guide.md.)
- *Reputational/compliance:* "No material regulatory findings; all new products pass the conduct-and-reputation gate; front-page test applied to all material decisions." (Cross-ref: financial_risk_compliance_systems_guide.md.)

**The cascade.** Each statement decomposes: the group CET1 floor cascades to entity-level floors and business-level RWA budgets; the market ES envelope cascades to desk VaR limits and stop-losses; the liquidity statements cascade to ALCO's LCR/NSFR internal targets per entity; the operational statements cascade to KRI thresholds per function (the RAG of §6.4). Every limit in the limit inventory carries a reference back to the RAS line it implements — the traceability the FSB principles require and the limit systems in banking_limits_domain_guide.md (via the cluster's systems guide) must store.

**The RAG.** Each RAS metric gets a documented green/amber/red definition: green = within appetite; amber = inside tolerance but beyond the management trigger (watch, plan response); red = breached (escalate to the CRO and board risk committee within defined timeframes, remediation plan, and — for capital/liquidity metrics — invoke the recovery-plan triggers). The amber band is the operational heart of the framework: it is where most real risk management happens, because red should be rare and green should be normal.

### 10.3 The design: taxonomy

The risk taxonomy (§4) is adopted as the enterprise inventory, mapped to the ICAAP risk register and to the systems estate:

- The **Basel four** (credit, market, operational, liquidity) carry the Pillar 1 RWA and the quantitative appetite metrics.
- **Strategic risk** is owned by the CEO's office with the board: assessed through the strategy review cycle (COSO P6–P9) and an annual strategic-risk scenario (e.g., "the rates business loses its competitive position to electronification"; "a major client sector defaults simultaneously").
- **Reputational risk** is owned by the CRO with communications: every new product and every incident passes the reputation gate; the board sees a reputational-risk register alongside the financial one.
- **Compliance risk** is owned by the compliance function (second line) with the AML/sanctions programme at its centre; it feeds the operational-risk family for capital purposes while remaining a distinct governance family.

The taxonomy is also the **data model** for the risk register: every risk in the register is tagged with family, business line, legal entity, owner (first line), monitor (second line), appetite line, and limits. This tagging is what makes the enterprise view *computable* — the BCBS 239 aggregation (risk_data_aggregation_guide.md §3) cuts the register by exactly those dimensions.

### 10.4 The design: governance

The three lines are drawn explicitly (IIA 2020 model, §5.1): the **governing body** (board and its risk committee) owns oversight and appetite; **management** splits into the first line (the four businesses own and manage their risks) and the second line (group risk under the CRO — credit, market, operational, liquidity, model risk, data governance; compliance; the ICAAP office); **internal audit** provides the independent third-line assurance. The committee lattice (board risk committee → group risk committee under the CRO → specialist committees: credit, market-risk, ALCO, op-risk, model-risk, new-product) is chartered with mandates that reference the framework. Two structural decisions carry the design:

1. **The CRO's independence is structural, not rhetorical**: reporting line to the CEO for administration, direct access and reporting to the board risk committee, removal only with board approval — the FSB/regulatory expectation.
2. **The ICAAP is a standing programme, not an annual document**: a permanent ICAAP office (second line) runs the risk-inventory-to-capital pipeline year-round, feeding the SREP and the board's capital view (basel_regulatory_capital_guide.md §8; the data spine in risk_data_aggregation_guide.md).

### 10.5 The design: process and systems

The ISO 31000 process (§7) is adopted as the workflow standard — identification (risk register + RCSA per function, updated on substantial change), assessment (heat map + the model outputs of risk_management_models_guide.md + stress), response (the limit/hedge/control/capital menu), monitoring (KRI RAG dashboards, limit utilisation, model backtests), reporting (the board risk report, the ICAAP, regulatory returns via regtech_guide.md's reporting layer). System-wise, the framework is *encoded* rather than documented: the risk register lives in a system; the RAG dashboard is generated from the BCBS 239-aggregated data; the limit system enforces the cascade; the board pack is assembled from the same source. The framework document is the design spec; the systems are the framework.

### 10.6 The lessons

1. **Appetite is the keystone.** Every other element — taxonomy, limits, ICAAP, board reporting — hangs off the RAS. An unmeasurable RAS (§6.4's failure mode) collapses the whole design; the quantified, cascaded, RAG-monitored RAS is what makes the enterprise view operational.
2. **The taxonomy is a data model before it is a document.** The enterprise view is computed by cutting risk data by family × business × entity; if the taxonomy is not encoded in the data, the board deck is prose.
3. **Governance structure beats governance documents.** The CRO's independence and the committee mandates do more work than any framework chapter; regulators review structures, not binders.
4. **The ICAAP is ERM in capital form.** Linking appetite to the ICAAP (and both to stress testing) is what makes ERM binding on the business — risk appetites that never touch capital are advisory.
5. **BCBS 239 is the engineering prerequisite.** Without firm-wide aggregation, "the enterprise view" is a collection of siloed numbers with a cover page — the exact failure the framework exists to prevent.
6. **Frameworks are forks, not ends.** COSO-shaped governance + ISO-shaped process + regulatory requirements, assembled as a hybrid, is the realistic outcome — the "which framework" question (§9) resolves into "what does each contribute".

---

### 10.7 The implementation roadmap

The framework is designed; the delivery is a programme. A twelve-month arc, in the pattern of the cluster's other implementation guides:

- **Months 1–3 — foundation.** Board risk committee charter refreshed; CRO independence confirmed; the RAS drafted with the CFO and CRO, stress-tested against the ICAAP numbers, and approved by the board; the taxonomy and risk-register data model defined; the limit-inventory mapping (every limit → its RAS line) started.
- **Months 4–6 — the risk process live.** Risk register and RCSA rolled out business-line by business-line; KRI set defined per family with RAG thresholds; the RAG dashboard built on the BCBS 239-aggregated data (risk_data_aggregation_guide.md §4); first group risk committee review of a live dashboard.
- **Months 7–9 — the ICAAP linkage.** The ICAAP office runs the first full risk-inventory-to-capital cycle under the new framework; appetite thresholds reconciled with the internal capital target; the first SREP-ready pack assembled (basel_regulatory_capital_guide.md §8).
- **Months 10–12 — embed and audit.** Internal audit reviews the framework against the IIA Three Lines Model and the COSO 2017 principles (a self-assessment against the 20 principles is the standard instrument); findings feed the improvement cycle (COSO P17); the board signs off the first annual ERM review.

The roadmap's test of success is not documents produced but *decisions changed*: a limit breached and escalated, a new product declined on appetite grounds, a strategy option rejected on risk grounds — the behaviours that distinguish a framework from a shelf document (§2.2).

---

## 11. The Summary: One Page — "The Enterprise View"

Enterprise risk management is the discipline of seeing the whole risk picture at once: all risk types, all business lines, all entities, in the service of strategy — and governing that picture from the board down. The one-page version:

- **The definition** (COSO 2004, still canonical): ERM is a process, effected by the board, management, and personnel, applied in strategy setting and across the enterprise, to identify potential events and manage risk within risk appetite, providing reasonable assurance on objectives. The enterprise-wide view — versus the siloed view that failed in 2008 and again in 2023 — is the discipline's reason for being.
- **The frameworks**: COSO ERM (2004 cube: 4 objective categories × 8 components; 2017 update: 5 components — Governance & Culture, Strategy & Objective-Setting, Performance, Review & Revision, Information, Communication & Reporting — and 20 principles, organised around integrating risk with strategy) and ISO 31000 (2009, revised 2018: 8 principles + a framework of leadership/design/implementation/evaluation/improvement + a process of communication, context, assessment, treatment, monitoring, recording). COSO for governance and strategy in regulated firms; ISO for principles and process everywhere; hybrids in practice.
- **The taxonomy**: credit, market, operational, and liquidity (the Basel four, each with its model machinery in risk_management_models_guide.md) plus strategic, reputational, and compliance — the families Pillar 1 ignores but the enterprise cannot.
- **The governance**: three lines of defence (IIA 2013; the 2020 Three Lines Model adds the governing body explicitly) — business owns, risk-and-compliance oversees, audit assures — under a board that approves appetite and a CRO whose independence is structural.
- **The appetite**: capacity > appetite > tolerance > limits; the FSB's 2013 RAF principles made the risk appetite statement a supervised artefact; the RAG turns appetite into a dashboard.
- **The process**: identify → assess → respond → monitor → report, run continuously, wrapped in communication and review — the ISO loop that every bank artefact (RCSA, ICAAP, board pack) instantiates.
- **The banking expression**: the ICAAP links the risk profile to capital under Pillar 2 (basel_regulatory_capital_guide.md §8); BCBS 239 makes the firm-wide aggregation that the enterprise view requires a supervised capability (risk_data_aggregation_guide.md); supervisors review the framework itself.
- **The final word — the enterprise view**: siloed risk management asks "what is this desk's risk?" and gets a portfolio of blind spots; enterprise risk management asks "what is the firm's risk, in aggregate, against appetite, under stress?" and gets a number the board can govern. The frameworks, the taxonomy, the governance, the appetite, the process — every element of the discipline exists to make that second question answerable, every day, with data the supervisors can audit and decisions the business actually follows. That is the enterprise view: risk as a portfolio, governed as a whole, in the service of strategy.

---

## 12. Verification Notes and Sources

**Method.** Facts in this guide were verified against primary sources and authoritative secondary references using targeted web research conducted in August 2026. The verification ledger:

- **COSO 2004 definition and cube** — verified against the COSO executive summary of *Enterprise Risk Management — Integrated Framework* (September 2004): the definition quoted in §1.1 verbatim; the eight components and four objective categories of the cube in §2.1.
- **COSO 2017 update** — verified against COSO's own ERM page and multiple independent summaries: title *Enterprise Risk Management — Integrating with Strategy and Performance*; September 2017; five components and the 5/4/5/3/3 distribution of the 20 principles; the helix metaphor; the 2018 *Compendium of Examples* supplement. Principle titles follow the framework's official wording.
- **ISO 31000:2009 and 2018** — verified against ISO's catalogue pages (standard 43170 for 2009; standard 65694 and the ISO/TC 262 project page for 2018): November 2009 publication; mid-February 2018 availability; the AS/NZS 4360:2004 lineage; the 2018 emphasis on creating and protecting value, continual improvement, stakeholder inclusion, customisation, human and cultural factors; the eight 2018 principles; the framework and process structure. The 2009 edition's eleven-principle count is asserted from the 2009 text's known structure **[verify]** — treat the specific count as approximate if precision matters.
- **IEC 31010** — verified via the ISO catalogue (IEC 31010:2019, *Risk management — Risk assessment techniques*); the 2009 ISO/IEC 31010 edition is referenced from the standard's history.
- **IIA three lines** — verified via the IIA's own publication pages: the 2013 position paper *The Three Lines of Defense in Effective Risk Management and Control* and the September 2020 update *The IIA's Three Lines Model: An update of the Three Lines of Defense* (the IIA site dates the update position paper September 2020; some sources cite July 2020 for its release — the model is sometimes dated July 2020 **[verify]** — the September 2020 position-paper page is the authoritative reference used here).
- **FSB risk appetite** — verified via the FSB site: *Principles for an Effective Risk Appetite Framework*, 18 November 2013; the risk-capacity/risk-appetite distinction and the four RAF elements quoted in §6.1 are from the FSB text.
- **ISO Guide 73:2009 definitions** (risk appetite, risk tolerance) — asserted from the standard's well-established vocabulary; the Guide 73 definitions are widely quoted verbatim and were not re-extracted in this research pass **[verify]** — the definitions given match the standard's canonical wording.
- **RAG** — flagged as industry convention rather than a formal framework element (§6.4); no primary standard defines "RAG" as a named component.
- **Banking facts** (Basel II Pillar 2/ICAAP/SREP, BCBS 239, Basel III, MAS) — cross-referenced rather than re-derived; the verified facts live in basel_regulatory_capital_guide.md and risk_data_aggregation_guide.md, which were read for this guide.
- **Worked example** — pedagogical construction; no figures are drawn from any bank's public disclosures.

---

## 13. Glossary

| Term | Definition |
|---|---|
| **ERM** | Enterprise risk management — the discipline of managing all of an organisation's risks as a single enterprise-wide portfolio, integrated with strategy and governed to a stated risk appetite |
| **Enterprise risk management** | COSO 2004: "a process, effected by an entity's board of directors, management and other personnel, applied in strategy setting and across the enterprise, designed to identify potential events that may affect the entity, and manage risk to be within its risk appetite, to provide reasonable assurance regarding the achievement of entity objectives" |
| **COSO** | Committee of Sponsoring Organizations of the Treadway Commission — the US professional-body consortium that publishes the ERM and internal-control frameworks |
| **2004 framework** | *Enterprise Risk Management — Integrated Framework* (September 2004): the COSO cube — 4 objective categories (strategic, operations, reporting, compliance) × 8 components × entity levels; origin of the ERM definition and the avoid/reduce/share/accept responses |
| **2017 update** | *Enterprise Risk Management — Integrating with Strategy and Performance* (September 2017): the revision that replaced the 8 components with 5 components and 20 principles, and made strategy integration the framework's thesis |
| **Five components** | The COSO 2017 components: Governance & Culture; Strategy & Objective-Setting; Performance; Review & Revision; Information, Communication & Reporting |
| **20 principles** | The COSO 2017 principles under the five components: 5 + 4 + 5 + 3 + 3 (e.g., P1 exercises board risk oversight; P7 defines risk appetite; P14 develops portfolio view; P20 reports on risk, culture, and performance) |
| **ISO 31000** | The international risk-management standard: 2009 first edition (*Principles and guidelines*), 2018 revision (*Guidelines*) — principles + framework + process for managing any risk in any organisation |
| **2018 revision** | ISO 31000:2018 (February 2018): 8 principles, a leadership-centred framework (integration, design, implementation, evaluation, improvement), and the process (communication; scope/context/criteria; assessment; treatment; monitoring; recording) |
| **Principles** | ISO 31000's eight: integrated; structured and comprehensive; customized; inclusive; dynamic; best available information; human and cultural factors; continual improvement |
| **Framework** | The organisational layer that embeds risk management (ISO: leadership and commitment, design, implementation, evaluation, improvement; COSO: the five components) |
| **Process** | The workflow applied to any risk: identification → assessment (analysis/evaluation) → response/treatment → monitoring → reporting, wrapped in communication and review (ISO 31000) |
| **Risk taxonomy** | The map of the risk universe: credit, market, operational, liquidity (the Basel four) plus strategic, reputational, compliance |
| **Credit risk** | Risk of loss from obligor/counterparty failure to meet obligations (see risk_management_models_guide.md §3) |
| **Market risk** | Risk of loss from adverse price/rate/volatility movements (see risk_management_models_guide.md §4) |
| **Operational risk** | Risk of loss from failed processes, people, systems, or external events, incl. legal risk (see risk_management_models_guide.md §5) |
| **Liquidity risk** | Risk of inability to meet obligations as they fall due without unacceptable losses (see treasury_alm_guide.md) |
| **Strategic risk** | Risk that strategy or business-model assumptions prove wrong |
| **Reputational risk** | Risk of stakeholder-perception damage to the franchise |
| **Compliance risk** | Risk of legal/regulatory sanction from failure to comply with laws, regulations, and policies |
| **Three lines of defence** | The IIA model: first line (operational management owns risk), second line (risk and compliance oversee), third line (internal audit assures) |
| **IIA** | Institute of Internal Auditors — publisher of the Three Lines of Defense (2013) and the Three Lines Model (2020) |
| **Board** | The governing body that approves appetite, oversees strategy's risk implications, and holds management accountable (COSO P1) |
| **CRO** | Chief risk officer — the second line's leader; independent board access and structural independence are supervisory expectations |
| **Risk committee** | The board- or management-level forum for risk decisions; e.g., board risk committee, group risk committee, ALCO, model risk committee |
| **Risk appetite** | The amount and type of risk an organisation is willing to pursue or retain (ISO Guide 73); "the aggregate level and types of risk a firm is willing to assume within its risk capacity" (FSB) |
| **Tolerance** | The readiness to bear risk after treatment in order to achieve objectives (ISO Guide 73) — the acceptable deviation bands within appetite |
| **RAG** | Red/amber/green — the traffic-light convention for monitoring metrics against appetite thresholds; industry convention, not a formal standard element **[verify]** |
| **Identification** | The process stage that finds risks that could affect objectives (COSO P10; ISO risk identification) |
| **Assessment** | The process stage that analyses (likelihood × impact, inherent vs residual) and evaluates (priority vs criteria) risks (COSO P11–P12; ISO risk analysis/evaluation) |
| **Response** | The process stage that selects treatment: avoid, reduce, share, accept (COSO P13; ISO risk treatment) |
| **Monitoring** | The process stage that watches risks, controls, and the process itself via KRIs, limits, and review cycles (COSO P15–P16; ISO monitoring and review) |
| **Reporting** | The process stage that puts the risk picture before decision-makers and regulators (COSO P19–P20; ISO recording and reporting; BCBS 239 reporting principles) |
| **ICAAP** | Internal Capital Adequacy Assessment Process — the bank's own Pillar 2 assessment of the capital needed for all material risks (see basel_regulatory_capital_guide.md §8) |
| **Pillar 2** | The second Basel pillar — supervisory review: ICAAP (bank's assessment) + SREP (supervisor's assessment, concluding in P2R/P2G) |

---

## 14. Cross-References in This Series

| Guide | Why it matters here |
|---|---|
| [Risk Management Models Guide](risk_management_models_guide.md) | The **quant side** of the enterprise view — PD/LGD/EAD, VaR/ES, SMA, LCR/NSFR; ERM consumes these model outputs in the assessment stage (§4, §7) |
| [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) | The **systems landscape** the ERM framework must be encoded in — risk platforms, compliance systems, regulatory reporting |
| [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md) | The **capital side** — Pillar 1/2, ICAAP/SREP, P2R/P2G; the ICAAP is bank ERM in capital form (§8) |
| [Risk Data Aggregation Guide](risk_data_aggregation_guide.md) | **BCBS 239** — the data aggregation and reporting discipline that makes the enterprise view computable (§8.3) |
| [Treasury & ALM Guide](treasury_alm_guide.md) | The **liquidity family** — LCR/NSFR/IRRBB; the liquidity statements in the RAS and ALCO governance |
| [CECL Guide](cecl_guide.md) | **Credit impairment** — IFRS 9/CECL ECL as the provisioning side of credit risk |
| [RegTech Guide](regtech_guide.md) | The **regulatory-technology layer** — COREP, MAS returns, reporting automation |
| [AI & GenAI Banking Compliance Guide](ai_genai_banking_compliance_guide.md) | The **AI-risk overlay** — AI as an operational-risk family with its own governance requirements (light) |
| [Banks in Singapore Guide](banks_in_singapore_guide.md) | The **MAS expectations** — risk governance, appetite, and supervision in the Singapore context |
| [Crédit Agricole Software Systems Guide](credit_agricole_software_systems_guide.md) | The **Cymbal Bank context** behind the §10 worked example |
| [Cybersecurity Guide](../technology/cybersecurity_guide.md) | The **three-lines-for-security** usage of the same governance model (§5) |
| [Business Case Development Guide](../management/business_case_development_guide.md) | The business case for ERM investment (light) |
| [MBA Body of Knowledge Guide](../management/mba_body_of_knowledge_guide.md) | The finance vocabulary (light) |

*End of guide. Companion reading: risk_management_models_guide.md (the quant side of every family in the taxonomy), basel_regulatory_capital_guide.md §8 (the ICAAP/Pillar 2 machinery), risk_data_aggregation_guide.md (the BCBS 239 data spine), treasury_alm_guide.md (the liquidity family).*
