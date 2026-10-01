# Governance, Risk and Compliance (GRC) in Banking: The Integrated Discipline — A Comprehensive Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Series:** Banking & Financial Technology Guides
> **Audience:** Solution architects, risk & compliance technologists, and governance professionals at a bank — the people who must turn a governance mandate into registers, control libraries, evidence trails, remediation workflows and board packs.

*The dedicated deep-dive on governance, risk and compliance (GRC) as an integrated discipline in banking — the connective layer that makes the business's ownership of risk, the second line's assurance of it, and the third line's independent verification of it work as one system. This is deliberately **not** a second guide to risk frameworks: the frameworks, the taxonomy, the appetite machinery and the risk process live in the sibling guides, and this guide owns the operating discipline that runs them — the three lines as a working relationship, the compliance function, regulatory change, control design and testing, the assurance map, issue and remediation management, the KRI framework, the GRC platform layer, board reporting, individual accountability, and the integration boundaries with data, AI, technology and third-party governance. It ends with a worked GRC review at Cymbal Bank.*

**Series / boundary:** The risk cluster already has a framework anchor, so the division of labour is explicit. [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) owns the COSO ERM 2004/2017 and ISO 31000 2009/2018 frameworks, the risk taxonomy, risk appetite, and the risk process end-to-end, **and it owns the three-lines model itself** — this guide names the three lines as working roles but does not re-explain the model (see ERM §5, referenced by name throughout). [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) owns the Singapore regulatory content; [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md) owns Pillar 1/2, the ICAAP and the capital machinery; [Risk Management Models Guide](risk_management_models_guide.md) owns the quantitative models; [Risk Data Aggregation Guide](risk_data_aggregation_guide.md) owns BCBS 239; [AML Certifications & Exam Content Guide](aml_certifications_exam_content_guide.md) and the repo's AML/KYC material own financial-crime content; [AI & GenAI Banking Compliance Guide](ai_genai_banking_compliance_guide.md) owns the AI-compliance overlay; [RegTech Guide](regtech_guide.md) owns the regulatory-technology layer; [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) owns the compliance *systems* estate; [Operational Resilience Framework Guide](operational_resilience_framework_guide.md) owns resilience; [Data Governance Guide](../technology/data_governance_guide.md) owns data governance; [AI Governance Framework Guide](../technology/ai_llm/ai_governance_framework_guide.md) owns AI governance; [API Governance Guide](../technology/api_governance_guide.md) owns API governance; [Audit as Code Guide](../technology/audit_as_code_guide.md) owns audit tooling; and [Directly Responsible Individual Guide](../management/directly_responsible_individual_guide.md) owns the individual-accountability craft. This guide cross-references all of them rather than re-deriving their content.

**Scope note on verification:** the framework anchors used below — the IIA Three Lines Model position paper (2020), COSO's internal-control and ERM guidance, ISACA's COBIT, ISO 31000:2018, and the two BCBS guidelines that define compliance risk and the compliance function — were checked at the issuing bodies' own pages during this research pass and are cited with issuer, edition and year. Where a fact could not be fully verified at a primary source — notably the **month** of the IIA's 2020 model update, and the exact edition designation of COBIT — it is flagged **[verify]** rather than asserted. Everything else is practice-based description: no statistic, no benchmark, no threshold, and no vendor capability claim is invented anywhere in this guide, and where practice genuinely varies by institution the text says so.

**Cross-references used throughout:** enterprise_risk_management_guide.md (the frameworks, taxonomy, appetite and the three-lines model), mas_regulations_guidelines_guide.md, basel_regulatory_capital_guide.md, risk_management_models_guide.md, risk_data_aggregation_guide.md (BCBS 239), aml_certifications_exam_content_guide.md (financial crime), ai_genai_banking_compliance_guide.md, regtech_guide.md, financial_risk_compliance_systems_guide.md, operational_resilience_framework_guide.md, banks_in_singapore_guide.md, ../technology/data_governance_guide.md, ../technology/ai_llm/ai_governance_framework_guide.md, ../technology/audit_as_code_guide.md, ../technology/api_governance_guide.md, ../technology/cybersecurity_guide.md, ../management/directly_responsible_individual_guide.md, and ../management/vendor_management_guide.md.

### Reading paths

- **Solution architects and engineers** — §1 (the decoder and the boundary), §5–§6 (control design, testing, RCSA, the assurance map — the data model behind it all), §8 (the issue lifecycle as a workflow), §10 (what a GRC platform actually does), §14 (the worked review). Pair with risk_data_aggregation_guide.md for the data layer and ../technology/data_governance_guide.md for the ownership model underneath it.
- **Risk and compliance professionals** — §2–§4 (the lines as a working relationship, the compliance operating model, regulatory change), §6 (assurance mapping), §8–§9 (issues and KRIs), §11–§13 (board reporting, integration domains, accountability).
- **Governance professionals and board-support staff** — §1 (the decoder), §2 (the handoffs), §9 (the KRI discipline), §11 (what the board is accountable for receiving), §15 (the anti-patterns as a checklist).
- **General readers** — §1, §2, §14 (the Cymbal Bank review), §16 (the glossary and closing summary).

Each section stands alone: the table at the end of each is the quick reference, and the cross-references let a reader jump to the sibling guide that owns the underlying detail.

---

## Table of Contents

1. The Overview, the Decoder and the Boundary
2. The Three Lines as a Working Relationship
3. The Compliance Function and Its Operating Model
4. Regulatory Change Management
5. Control Design and the Control Framework
6. Control Testing, RCSA and the Assurance Map
7. The Third Line
8. Issue and Remediation Management
9. The KRI Framework
10. The GRC Platform Layer
11. Board and Committee Reporting
12. The Domains GRC Must Integrate With
13. Individual Accountability
14. The Cymbal Bank Worked Example
15. The Anti-Patterns
16. The Claims Audit, then What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary

---

## 1. The Overview, the Decoder and the Boundary

### 1.1 The thesis: the business owns the risk; everything else is assurance

Most banks describe governance, risk and compliance as three functions standing side by side. That description is the source of most of the trouble. GRC is better understood as **one discipline with three audiences**, resting on a single sentence:

> **The business owns the risk; everything else is assurance.**

Read the sentence carefully and the whole architecture falls out of it. The **business** — the first line: the trading desks, the lending teams, the operations units, the branch network — *owns* the risk. It takes the risk, it earns the return, and it is accountable for the control environment that keeps that risk inside appetite. That ownership is not a reporting obligation; it is a duty that cannot be delegated upward to a risk function, because a risk function that owns the business's risk has quietly become the business. Everything else in GRC — the second line's frameworks, monitoring and challenge; the third line's independent assurance; the compliance programme; the control library; the assurance map; the issue workflow; the board pack — exists to **assure** that the business is owning its risk well. Assurance is evidence and challenge pointed at someone else's accountability. It is not a substitute for it.

This guide is about the discipline that sentence describes. It is **not** a re-run of the enterprise risk management guide: the frameworks (COSO ERM 2004/2017, ISO 31000 2009/2018), the risk taxonomy, risk appetite, and the three-lines *model* itself are owned by [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md), and this guide names them rather than re-deriving them. What this guide owns is the **integrated operating layer**: the handoffs between the lines, the compliance function as a working machine, the inventory of obligations, the design and testing of controls, the map of who assures what, the lifecycle of an issue, the pruning of indicators, the platform that records all of it, and the reporting that carries it to the board.

### 1.2 The defining decoder

GRC suffers from vocabulary slippage: the same word means different things to a business head, a risk officer, an auditor, a regulator and a system vendor. The table below is the decoder used throughout this guide. Every term here is defined so that the rest of the guide can be read without ambiguity, and the definitions are deliberately operational — what the thing *is* and what artefact carries it — rather than aspirational.

| Term | Definition used in this guide | Carried by |
|---|---|---|
| **GRC** | Governance, risk and compliance — the integrated discipline by which an organisation assigns accountability for risk (governance), measures and manages it (risk), and demonstrates conformity with its obligations (compliance), as one system rather than three silos. In banking, its organising principle is *the business owns the risk; everything else is assurance*. | The operating layer this guide describes: registers, control library, assessment and issue workflows, assurance map, reporting |
| **Three lines** | The division of risk-and-control roles into three groups: the functions that own and manage risk (first), the functions that provide independent oversight, frameworks, monitoring and challenge (second), and the function that provides independent assurance to the governing body (third). The model itself — its principles, its history, its 2013 and 2020 formulations — is owned by [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §5. | The roles, mandates and reporting lines of the bank's functions |
| **First line** | The business and operational functions that take risk and run the controls: they own the risk, own the controls, and are accountable for their design and operation. | Business units, operations, technology delivery, the process owners |
| **Second line** | The independent oversight functions — risk management (credit, market, operational, model), compliance, and increasingly the data and AI governance functions — that set frameworks, monitor against them, advise, and **challenge** the first line. They do not own the business's risk. | Risk frameworks, policies, limits, monitoring, second-line testing, advice |
| **Third line** | Internal audit: independent, objective assurance to the board (via the audit committee) on whether the first and second lines are doing what the board believes they are doing. Independence is structural, not attitudinal. | The audit universe, the risk-based audit plan, audit findings and opinions |
| **Compliance function** | The second-line function responsible for identifying, assessing, advising on, monitoring and reporting the bank's compliance risk — its exposure to legal or regulatory sanction, financial loss or reputational damage from failure to comply with applicable laws, regulations, rules, standards and internal policies. BCBS defines compliance risk and sets supervisory expectations for the function's mandate, independence, resources and reporting. | The compliance programme: policy, training, advice, monitoring, testing, reporting |
| **Control** | A measure — a process step, a check, a system restriction, an approval, a reconciliation, a review — that modifies the likelihood or impact of a risk. A control is an *action or mechanism*, not a policy document, a person's good intentions, or a general statement that "risk is managed". | The control library entry, its owner, its evidence, its tests |
| **Control design** | Whether a control, as specified and built, is capable of preventing or detecting the risk it is meant to address, in the process where it actually sits. A design assessment asks: *if this control worked exactly as written, would it reduce this risk?* | The control's description, its logical placement, its design assessment |
| **Operating effectiveness** | Whether the control actually operated as designed, consistently, over the period under review — evidenced, for the instances actually tested. A control can be perfectly designed and not operate; it can be badly designed and operate impeccably. | Test results, samples, evidence, exceptions, conclusions |
| **RCSA** | Risk and control self-assessment: the first line's own structured assessment of the risks in its processes and the controls that address them, scored and recorded. It is **self**-assessment — declared by the owner, not independent. | The RCSA record: process, risk, control, score, action |
| **Control testing** | The structured examination of a control to reach a conclusion on design and/or operating effectiveness, by inspection, re-performance, observation, enquiry or data analysis, against a defined sample. Second-line testing is independent of the first line; first-line testing is not independent of itself. | The test plan, the sample, the evidence, the conclusion, the exceptions |
| **Assurance map** | The artefact that records, for each material risk or control, **which function assures it, by what method, at what depth and at what frequency** — making the coverage, the overlaps and the gaps visible on one page. | The assurance map: risk/control × assurer × method × frequency |
| **KRI** | Key risk indicator: a metric that gives an early or current signal of the level or trajectory of a risk, with a defined threshold and a defined action when the threshold is breached. A metric nobody can act on is not a KRI. | The KRI register, thresholds, the dashboard, the escalation |
| **Issue** | A logged deficiency, gap, weakness, breach or improvement requirement, with an owner, a severity, a remediation commitment and a validation step. Distinct from an incident and from a breach (see §8.2 for the exact distinction). | The issue register, the remediation plan, the validation record |
| **Regulatory inventory** | The inventory of obligations applicable to the bank: the record of what the bank must comply with, from which source, mapped to the policies, controls, systems, processes and reports that implement each obligation. | The obligations register and its mapping to the control library |
| **Attestation** | A formal, signed declaration by a named accountable person that a defined state of affairs is true as at a stated date — for example that the controls in their area operated, or that the obligations in their area are implemented. Its value is entirely a function of the evidence behind it and the accountability of the person signing. | The attestation document, its scope, its signatory, its evidence pack |
| **GRC platform** | Software that **records and orchestrates** GRC activity structurally: registers of risks, obligations, controls and issues; assessment and testing workflows; evidence storage; reporting. It records a programme; it does not create one (§10). | The platform's registers, workflows and reports |

### 1.3 What this guide owns, and what it deliberately does not

**It owns** the integrated operating discipline: the handoffs and information flows between the lines (§2); the compliance function's mandate, organisation and operating model (§3); regulatory change and the regulatory inventory (§4); control design vocabulary and the control framework (§5); control testing, RCSA and the assurance map (§6); the third line's role and its relationship to the other two (§7); issue and remediation management (§8); the KRI framework (§9); the GRC platform layer and its underlying data problem (§10); board and committee reporting (§11); the integration boundaries with data, AI, technology and third-party governance and the domains' guide references (§12); individual accountability (§13); a worked review at Cymbal Bank (§14); the anti-patterns (§15); and the claims audit (§16).

**It does not own** — and does not re-derive — the following: the COSO and ISO frameworks, the risk taxonomy, risk appetite, risk capacity/tolerance/limits, and the risk process (owned by [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md), which also owns the **three-lines model itself**, so this guide names the lines and describes how they work together but does not re-explain the model); the Singapore regulatory content ([MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)); capital, the ICAAP and the Basel framework ([Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md)); the quantitative models ([Risk Management Models Guide](risk_management_models_guide.md)); firm-wide risk data aggregation ([Risk Data Aggregation Guide](risk_data_aggregation_guide.md) — BCBS 239); financial crime and the AML/KYC programme ([AML Certifications & Exam Content Guide](aml_certifications_exam_content_guide.md) and the repo's AML material); the AI-compliance overlay ([AI & GenAI Banking Compliance Guide](ai_genai_banking_compliance_guide.md)); the regulatory-technology layer ([RegTech Guide](regtech_guide.md)); the compliance systems estate ([Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md)); resilience ([Operational Resilience Framework Guide](operational_resilience_framework_guide.md)); data governance ([Data Governance Guide](../technology/data_governance_guide.md)); AI governance ([AI Governance Framework Guide](../technology/ai_llm/ai_governance_framework_guide.md)); API governance ([API Governance Guide](../technology/api_governance_guide.md)); audit tooling ([Audit as Code Guide](../technology/audit_as_code_guide.md)); and the individual-accountability craft ([Directly Responsible Individual Guide](../management/directly_responsible_individual_guide.md)).

### 1.4 Why the boundary matters

The boundary is not filing discipline; it is the difference between a GRC programme and a knot. When the compliance function, the risk function and internal audit each maintain their own definition of "control", their own register of obligations, and their own issue log, the bank gets three half-truths and no single answer. When the control vocabulary is shared, the same control can be described once, tested once, evidenced once and reported to whoever needs it — the second line for oversight, the third line for assurance, the board for oversight of oversight. §2 makes this argument concretely; §6 shows the artefact that only exists if the boundary is respected: the assurance map.

### 1.5 The overview table

| Element | This guide's treatment | Owner of the deeper detail |
|---|---|---|
| **The framework layer** | Cross-referenced only | enterprise_risk_management_guide.md (§2–§3) |
| **The three-lines model** | Named; used as a working relationship only (§2) | enterprise_risk_management_guide.md §5 |
| **Risk taxonomy** | Cross-referenced only | enterprise_risk_management_guide.md §4; risk_management_models_guide.md |
| **Risk appetite** | Cross-referenced by name (§11.3) | enterprise_risk_management_guide.md §6 |
| **Compliance function** | Owned here (§3) | Primary: BCBS 2005; mas_regulations_guidelines_guide.md for MAS specifics |
| **Regulatory change and the regulatory inventory** | Owned here (§4) | regtech_guide.md for the technology; mas_regulations_guidelines_guide.md for content |
| **Control design, control library, testing** | Owned here (§5–§6) | financial_risk_compliance_systems_guide.md for the systems that store them |
| **Assurance map** | Owned here (§6.5) | — |
| **Issue and remediation management** | Owned here (§8) | — |
| **KRI framework** | Owned here (§9) | risk_data_aggregation_guide.md for the reporting spine |
| **GRC platform** | Owned here (§10) — structurally, not as a survey | regtech_guide.md; financial_risk_compliance_systems_guide.md |
| **Board reporting** | Owned here (§11) | enterprise_risk_management_guide.md §6–§7 |
| **Data / AI / resilience / third-party integration** | Boundary section only (§12) | Each domain's own guide, by name |
| **Individual accountability** | Structural treatment (§13) | ../management/directly_responsible_individual_guide.md |

---

## 2. The Three Lines as a Working Relationship

The three-lines model is defined in [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §5 — its rationale, its 2013 origins, its 2020 update and its principles. This section does **not** restate the model. It takes the model as given and asks the question the model itself does not answer: **what do the lines actually hand each other, in what format, on what cadence, and where does the arrangement go wrong?**

### 2.1 The handoffs

A three-lines arrangement is only real if information moves between the lines as a matter of routine. The handoffs, stated as a working contract:

| From → To | What is handed over | Typical format | Cadence |
|---|---|---|---|
| **First line → second line** | Risk identification and assessment (RCSA output); control design and changes; loss and near-miss data; limit utilisation; exceptions and breaches; new-product and change proposals | The RCSA record; the control change request; the incident or loss record; the exception report | RCSA cycle (commonly quarterly or semi-annual, varying by institution); incidents and breaches as they occur; change proposals before approval |
| **Second line → first line** | The framework and policy requirements; the control library and control expectations; monitoring results and challenge; advisory opinions; issue findings and required remediation | Policies and standards; the control library; monitoring dashboards; review memos; the issue record | Policy cycle; monitoring as per the monitoring plan; advice on request; findings on completion of reviews |
| **First line → third line** | The process and control documentation audit will examine; evidence packs; access for testing | Process narratives, the control library extract, evidence | On request during an audit engagement |
| **Second line → third line** | The assurance the second line has already performed (so audit can rely, in part, on it — subject to the auditor's own evaluation); risk assessments that inform the audit universe | Second-line testing results and methodology; risk assessments; the assurance map | Per the audit planning cycle and periodically during engagements |
| **Third line → board/audit committee** | Independent opinion; the audit plan; findings and ratings; the state of remediation | Audit plan, audit reports, the findings register, periodic reporting to the audit committee | Per the audit plan and the audit committee's calendar |
| **Third line → first and second lines** | Findings, ratings, root-cause observations, recommendations, and validated closure status | Audit reports and the findings record | On completion of each engagement; ongoing for follow-up |
| **All three → the governing body** | The integrated risk-and-compliance picture: what the business owns, what the second line assures, what the third line has verified | The board risk-and-compliance pack (§11) | Per the board and committee calendar |

The format column matters more than it looks. A handoff in the wrong format is not a handoff: if the first line hands the second line a narrative when the second line needs a structured, scored record keyed to a control identifier, the second line will rebuild the data by hand, the two versions will diverge, and the third line will find two truths. The design principle is simple: **hand over structured records keyed to a shared identifier set, not prose about them.**

### 2.2 The four named failure modes

The three-lines arrangement fails in four characteristic ways. Each is a **mechanism**, not a moral failing, and each has a specific consequence.

**(i) The second line that operates a control instead of assuring it.** The mechanism: the first line is judged unable to run a control reliably — or simply does not want to — so the second line is asked to run it in the first line's place: the risk function performs the review, the compliance function files the report, the risk function signs off the exception. It is almost always framed as a temporary pragmatic fix. **The consequence:** the second line now both operates and assures the same control. It cannot independently challenge its own work, because the challenge would be self-challenge; the business's ownership of its risk is quietly transferred to a function that has no authority to run a business process; and the third line, arriving later, finds that the assurance it was told existed was in fact a piece of operation. The mechanism is a self-inflicted independence loss, and the tell is that the control's *owner* in the control library is a second-line function while the *process* the control sits in is a first-line process.

**(ii) The third line consulted on the design it will later audit.** The mechanism: because the audit function holds deep institutional knowledge — of how controls fail, of what regulators expect, of where the last three findings landed — the first or second line invites internal audit into the design of a new control, process, system or framework, and audit contributes its expertise. It is genuinely helpful. **The consequence:** when the control is built and later audited, audit is assessing a design it helped create. This is a self-review threat: the assurance opinion is compromised before the engagement begins, independence is impaired in fact if not in appearance, and the organisation loses the one function whose whole product was an independent view. The mechanism is not malice; it is the natural pull of the request. The guardrail is structural (audit advises on *principles and risk*, not on the design of the specific control it will later opine on) and it must be enforced by the audit function itself, because the business will keep asking.

**(iii) The first line that treats risk ownership as a reporting obligation.** The mechanism: the business learns that what is measured, escalated and praised is *reporting*, not *risk outcomes*. The RCSA becomes a form to complete before the deadline; the incident log becomes a place where only already-known incidents appear; the "risk owner" field becomes a name assigned for the register rather than a person who actually steers the risk. **The consequence:** the bank accumulates a complete paper trail of ownership covering an incomplete picture of the risk. The controls are declared effective, the register is full, and the risk is unmanaged — because nobody in the business treats the register as their instrument. The tell is a first line that can describe its reporting obligations in detail and its top three risks in none.

**(iv) Duplication: three functions collect the same evidence from the same business unit in three formats.** The mechanism: each line builds its own evidence request from its own charter, in its own format, on its own cycle — the second line's control-test sample request, the third line's audit evidence request, and the first line's own compliance attestation, all reaching the same desk in the same quarter, all asking for the same approvals, reconciliations and system extracts, formatted three different ways. **The consequence:** the business unit spends its scarce time re-formatting the same facts for three consumers; the consumers each hold a partial, differently-cut version of the same evidence; and the third line discovers the inconsistency, turning the duplication into a finding of data-integrity weakness. The mechanism is structural: three charters, three formats, no shared identifier.

### 2.3 The integration argument

The four failure modes share one root: **three functions treating one risk as three objects.** The integration argument that follows is short and it is the spine of this guide:

> **One risk-and-control narrative, assessed once and reported to three audiences.**

Concretely: the first line assesses the process and its controls once, in a structured record (§6, RCSA); the second line independently tests the same controls once, against the same identifiers (§6, control testing); the third line, where it relies on that work, evaluates its quality and re-performs only what it must, and adds its own independent procedures where the assurance is thin (§7); and the same record — risk, control, owner, assessment, test result, issue, remediation status — is *cut three ways* for three consumers: the business for management, the second line for oversight and challenge, the board for assurance. The evidence is collected once (§2.2(iv) avoided); the second line assures rather than operates (§2.2(i) avoided); the third line verifies rather than reconstructs (§2.2(ii) avoided, because it is not a co-author); and the first line's ownership is expressed as a living record rather than a filing obligation (§2.2(iii) avoided).

This is the discipline the term GRC is reaching for. It is not three systems glued together; it is one body of evidence with three readers.

### 2.4 Cross-reference

For the model itself — the three lines of defence as promulgated by the Institute of Internal Auditors, the 2013 position paper, and the 2020 update that renamed the model and added the explicit governing body (board) — see [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §5. This guide assumes that model throughout and does not restate it. The supervisory expectation that the three lines operate in a bank's governance sits in BCBS's *Corporate governance principles for banks* (Guidelines, 08 July 2015), which explicitly references the risk management roles played by business units, risk management teams, and internal audit and control functions (the three lines of defence) and the importance of a sound risk culture.

### 2.5 The handoff table

| Element | The healthy pattern | The failure mode it prevents |
|---|---|---|
| **Risk ownership** | Business owns the risk and the control; the record shows a business owner | §2.2(i): second line operating the control; §2.2(iii): ownership as reporting |
| **Assurance** | Second line independently tests; third line verifies independently | §2.2(i): self-assurance by the operator |
| **Advice** | Second line advises; third line is not a designer of what it audits | §2.2(ii): self-review threat |
| **Evidence** | Collected once, keyed to shared identifiers, cut three ways | §2.2(iv): triplicate evidence requests |
| **Reporting** | One narrative, three audiences, consistent numbers | Three inconsistent versions of the same facts |
| **Challenge** | The second line's challenge is documented with the response | Assurance that is theoretical rather than real |

---

## 3. The Compliance Function and Its Operating Model

### 3.1 The mandate and the chief compliance officer

The **compliance function** is the second-line function whose subject matter is the bank's **compliance risk**: the risk of legal or regulatory sanction, financial loss or reputational damage arising from a failure to comply with applicable laws, regulations, rules, codes of conduct and standards of good practice. Its mandate has four working parts: **identify** the compliance risk in the bank's activities; **advise** the business on how to manage it; **monitor** compliance and **test** controls; and **report** — to the business, to senior management and to the board — on the state of compliance risk and on the function's own activity.

The chief compliance officer (CCO) leads the function. Structurally, the role carries the same independence question as the CRO (see [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §5.3): does the CCO have a reporting line to the board (or board committee) that cannot be severed by a business head, a seat in the senior forums where business decisions are taken, and a mandate wide enough to cover every jurisdiction and business line the bank operates in? Compliance's independence is a supervisory expectation, not a preference. The BCBS guidelines that anchor this section — *Compliance and the compliance function in banks* (Guidelines, 29 April 2005) — set supervisory expectations for the function's mandate, independence, resources and reporting, and define compliance risk as the paper's central concept (verified at the BCBS publication page).

### 3.2 The compliance risk taxonomy

Compliance risk is not one risk; it is a family, and the taxonomy at the level sources support runs along these lines:

- **Regulatory / prudential compliance risk** — failure to comply with the rules that govern the bank's licensed activity: capital and liquidity requirements, reporting obligations, governance rules, conduct-of-business requirements. This is the family that overlaps most directly with the prudential and capital machinery owned by [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md) and [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md).
- **Conduct risk** — the risk that the bank's behaviour toward customers and markets is inappropriate: mis-selling, unsuitable advice, market abuse, unfair treatment, conflicts of interest. Conduct risk is where compliance meets the reputational family and the culture agenda (see the ERM guide's treatment of risk culture).
- **Financial-crime compliance risk** — money laundering, terrorist financing, sanctions and proliferation-financing exposure. This is a distinctive sub-family with its own programme, its own systems and its own examinable certifications; the repo's dedicated treatment is [AML Certifications & Exam Content Guide](aml_certifications_exam_content_guide.md) and the associated AML/KYC material, and this guide cross-references it rather than re-deriving it.
- **Data- and privacy-compliance risk** — obligations touching personal data, bank secrecy, cross-border transfer. The governance side is [Data Governance Guide](../technology/data_governance_guide.md); the obligation side is inventoried under §4.

Where a bank draws the line between these families varies by institution, jurisdiction and business mix — the honest statement is that **practice varies**, and the important property is not the exact taxonomy but that the bank has one, uses it consistently across the register, the control library and the reporting, and can map every obligation to a family.

### 3.3 The programme components

A compliance programme is six components, and all six must exist; a programme missing any one is a programme with a known blind spot.

1. **Policy.** The written requirements — the compliance policies and standards that translate obligations into expectations the business can follow. Policy is the translation layer between the regulatory inventory (§4) and the control library (§5). Its failure mode is policy that restates the regulation in smoother language and gives the business no operational instruction.
2. **Training.** The mechanism by which the business learns what the policy requires and what it means in their role. Generic annual training is the weak form; role-specific, scenario-based training tied to the risks in that business's RCSA is the strong form. Training completion is a *compliance activity metric*, not a risk-outcome metric (§9.2).
3. **Advisory.** The function's day-to-day service: answering the business's questions, reviewing proposed products, changes and transactions, sitting in the forums where decisions are made, and giving an opinion *before* the decision rather than after. Advisory is where compliance earns the right to be listened to, and it is the component most easily crowded out by the function's reporting load.
4. **Monitoring.** Ongoing surveillance of compliance risk and control operation — review of exception and alert queues, sampling of completed processes, monitoring of regulatory developments and business changes — designed to spot deterioration before it becomes a finding.
5. **Testing.** Independent, structured examination of whether specific compliance controls operated as designed (the control-testing discipline of §6, applied to compliance controls).
6. **Reporting.** Reporting the state of compliance risk and the function's activity to the business, to senior management and to the board — the linkage into §11.

### 3.4 The advisory-versus-enforcement tension

The honest tension inside the compliance function is that it is asked to be both **adviser** and **enforcer**, and the two roles pull in opposite directions.

- A **compliance function that is adviser-only** — helpful, present, consulted, never confrontational — becomes a function that is challenged by nobody and holds no line. The business likes it, because advice can be taken or set aside. Its consequence is that there is no spine in the second line: no escalation, no refusal, no finding that the business must own, and no early warning, because a function that never says no is never told the truth about what is happening. It arrives at the failure — the fine, the enforcement action, the conduct scandal — as surprised as everyone else.
- A **compliance function that is enforcer-only** — a policing function that appears when something is wrong, issues findings, and leaves — becomes a function the business hides from. Its consequence is that the business stops inviting compliance into the room *before* the decision, stops flagging the grey-area product, stops reporting the near-miss, and starts presenting decisions already taken. The function is then reduced to examining finished facts, which is precisely the state in which the early warning it exists to provide cannot exist.

The resolution is not to pick a side; it is to be **adviser by default and enforcer by escalation** — engaged early on the business's decisions so that most issues are resolved as advice, with a documented, pre-agreed path to formal escalation (issue raised, owner assigned, escalated to the CRO/CCO and, if unresolved, to the board committee) when advice is not followed. The two failure modes above are the consequences of resolving the tension badly in *either* direction, and the mechanism in both cases is the same: **the business's relationship to compliance determines what compliance can see.** An advisory function sees almost everything and corrects little; an enforcement function corrects some things and sees little.

### 3.5 Organisation and resourcing

Compliance functions are organised in one of a small number of shapes, and the shape is a governance choice with consequences:

- **By risk family** (regulatory, conduct, financial crime, privacy) — deep subject-matter expertise, but each family needs a way to see the business's aggregate compliance position.
- **By business line** (a compliance officer embedded in each business) — close to the business and to its decisions, but closer proximity creates a proximity problem: the embedded officer is administratively near the business and must be structurally protected from it.
- **By function/process** (aligned to the processes compliance monitors) — aligned to where the controls actually sit.
- **The matrix** (family × business) — the common reality in large banks, with all the complexity that implies about who owns which decision.

Resourcing questions that recur and that the sources do not settle for you: how much of the function's capacity goes to advisory versus monitoring versus testing versus reporting; whether testing capacity is adequate to the control population (a testing plan that samples a fraction of the population must be honest about what it can and cannot conclude); and how jurisdictional coverage is staffed in a bank operating across supervisors, because a compliance function serving both a group supervisor and a host supervisor (for example a group supervisor in Europe and MAS in Singapore — see [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)) must be able to answer both. **Resourcing ratios and modelling practices vary substantially between institutions**; this guide does not assert a benchmark, and any figure quoted as a norm elsewhere should be treated with suspicion.

### 3.6 The reporting-line question

The independence question has three parts, and all three must be answered structurally rather than by good intentions: (i) **to whom does the CCO report** — with the expectation of a reporting line to the board or a board committee for the function's core mandate, alongside an administrative line to the CEO or CRO; (ii) **who can remove the CCO** — with the expectation of board involvement; and (iii) **how is the CCO remunerated** — with the expectation that the function's measures are not driven by business-line revenue. BCBS's 2005 guidelines on the compliance function state supervisory expectations on the function's independence, resources and reporting, and BCBS's *Corporate governance principles for banks* (2015) reinforce the control functions' role in the governance framework. A compliance function whose independence exists only in its charter and not in its reporting line, its removal terms and its remuneration is an independence of appearance.

### 3.7 The compliance function table

| Component | What it is | Its failure mode |
|---|---|---|
| **Mandate** | Identify, advise on, monitor, test and report compliance risk across the bank's activities | Mandate narrower than the bank's actual footprint (missing jurisdictions, businesses, or obligations) |
| **CCO** | Leader of the function; independent board access | Reporting line severable by a business head; remuneration tied to business revenue |
| **Compliance risk taxonomy** | Regulatory, conduct, financial-crime, data/privacy (bank-specific boundaries) | No single taxonomy; register, controls and reporting use different family labels |
| **Policy** | Obligation → operational expectation | Policy restates the rule; gives the business nothing to do |
| **Training** | Role-specific competence | Generic annual completion as the only measure |
| **Advisory** | Advice before the decision | Crowded out by reporting load; advice after the fact |
| **Monitoring** | Ongoing surveillance of risk and controls | Sampling too thin to detect deterioration; alerts not worked |
| **Testing** | Independent structured testing of compliance controls | Testing that is a second-line extension of first-line self-assessment |
| **Reporting** | To business, management, board | Volume reporting without a view of compliance risk level or trend |
| **Independence** | Structural: line, removal, remuneration | Independence of appearance only |

---

## 4. Regulatory Change Management

### 4.1 The lifecycle

Regulatory change management is the process by which a bank converts a change in its obligations into a change in its operations — and can show, later, that it did. The lifecycle has five stages, and the discipline is in refusing to skip any of them:

1. **Horizon scanning.** Systematic monitoring of the sources of obligation — regulators, standard-setters, legislation, listing rules, supervisory expectations, industry codes — for proposed and actual changes, including consultations, which matter long before the rule is final. The output is a *candidate list*: what might change, when, and with what apparent relevance to the bank. Scanning is a research function and it fails quietly: the change that was never scanned is a change that arrives as a surprise.
2. **Impact assessment.** For each confirmed change: does it apply to us (scope and entity and jurisdiction), what does it require, by when, and **which specific artefacts must change** — which policies, which controls, which systems, which processes, and critically **which reports**? Impact assessment is where most regulatory programmes under-deliver, because it stops at "which policy must be updated" and never reaches "which report will now be wrong", or "which control will now be testing the wrong thing". A change that requires a new data element in a regulatory return is not an impact on policy; it is an impact on the data model, the reporting pipeline and the reconciliation — a fact the data and reporting guides in this series describe from their side ([Risk Data Aggregation Guide](risk_data_aggregation_guide.md); [RegTech Guide](regtech_guide.md)).
3. **Implementation.** The programme of work that changes the policy, the control, the system, the process and the report — with owners, dates and dependencies. Implementation is where the change meets the bank's change-management and technology-delivery machinery; a regulatory change that requires a system change is a system change, and it is governed as one.
4. **Attestation.** The formal confirmation, by the accountable person, that the change has been implemented and that the affected obligations are met (see §13 for the accountability dimension and §3.3's reporting component). The attestation is only as good as the evidence behind it; an attestation with nothing behind it is a liability transfer, not a control.
5. **Tracking and closure.** The register of open regulatory changes, their status, their evidence and their closure — the artefact that lets the bank answer "are we compliant with the change that took effect last quarter?" without rebuilding the answer from emails.

### 4.2 The regulatory inventory

The **regulatory inventory** is the inventory of applicable obligations: for each obligation the bank is subject to, the record of what it requires, its source (instrument, article, and, where relevant, the supervisory expectation around it), the entity and jurisdiction it applies to, its effective date, and — the part that turns an inventory into a management instrument — **the mapping to what implements it**: the policy, the control, the system, the process, the report and the accountable owner.

An inventory without the mapping is a bibliography. An inventory with the mapping is the backbone of the entire GRC discipline: it is what lets the bank trace from an obligation forward to the controls that satisfy it, and backward from a failed control to the obligations exposed by the failure. It is also the input on which §5's control library, §6's testing plan, §8's issue severity and §11's board reporting all depend, because every one of those artefacts needs to know *what obligation is at stake*.

### 4.3 The central finding

The central finding of this section is a statement about the process, and it is a risk of the process rather than an allegation about any institution:

> **A bank cannot demonstrate compliance with an obligation it has not inventoried — and the regulatory inventory is the artefact most likely to be stale, duplicated across functions, and unowned.**

Each clause deserves to be read separately:

- **Cannot demonstrate compliance with an obligation it has not inventoried.** Compliance is a claim that must be evidenced. If the obligation is absent from the inventory, then no policy, control, system or report has been mapped to it; nothing tests it; nothing reports on it; and when the supervisor asks, the bank has no artefact with which to answer. The obligation may in fact be met — the process may happen to comply — but the bank cannot *demonstrate* it, and in a supervisory conversation, met-but-undemonstrable is a finding.
- **Most likely to be stale.** Obligations change continuously: a rule is amended, a supervisor issues a new expectation, a grace period expires, an interpretation shifts. An inventory is a snapshot, and unless it has an owner and a refresh cadence, it describes the bank's obligations as they were when the inventory was built. Staleness is invisible from inside the inventory — it looks exactly like an accurate one.
- **Most likely to be duplicated across functions.** Compliance keeps an obligation list for its programme; the risk function keeps one for its control mapping; the legal function keeps one for its advice; the business keeps one for its own procedures; the change function keeps one for its tracking. Five lists, five formats, five owners, and no single authority on what the bank is actually subject to — which is the duplication failure of §2.2(iv) in its most structural form.
- **Most likely to be unowned.** An inventory is nobody's revenue, nobody's risk, and everybody's input. It is the classic shared-artefact governance gap: each function assumes another is maintaining it, and the artefact that everything else depends on has no accountable owner. The remedy is a named accountable owner for the *inventory as an artefact* (distinct from the owners of the individual obligations), a defined refresh cadence aligned to the horizon-scanning cycle, and a single authoritative instance that the other functions consume rather than copy.

### 4.4 What good looks like, and what varies

Good regulatory change management keeps the inventory as a single owned instance at the group level, mapping each obligation to implementing artefacts and to an accountable person; runs horizon scanning on a defined cadence with a documented candidate list; requires impact assessment to reach *reports and data*, not just policies; tracks implementation to closure with evidence; and sequences attestation behind that evidence. What varies by institution and jurisdiction: the cadence of scanning, the granularity at which obligations are recorded (per instrument, per article, per requirement), whether the inventory is a bespoke system or part of a GRC platform (§10), and how the host-versus-home supervisor relationship is reflected in the entity mapping. None of that variation is a defect; the defect is having no inventory, or having several. Jurisdiction-specific obligations are not asserted here — they live in [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) and [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md) for their respective domains.

### 4.5 The regulatory change table

| Stage | Question | Artefact | Common failure |
|---|---|---|---|
| **Horizon scanning** | What is changing, and when? | The candidate list; the scanning record | Change discovered late because scanning is ad hoc |
| **Impact assessment** | Does it apply, and what exactly changes? | The impact assessment: policies, controls, systems, processes **and reports** | Assessment stops at policy and misses data, reporting and control impacts |
| **Implementation** | Who changes what, by when? | The implementation plan with owners and dates | Treated as a policy-editing exercise rather than a change programme |
| **Attestation** | Can the accountable person confirm it is done, on evidence? | The attestation and its evidence pack | Attestation signed with nothing behind it |
| **Tracking and closure** | Is it done, and can we show it? | The open-changes register | Closure asserted without evidence; the change re-opens at the next exam |
| **The inventory** | What are we subject to, and what implements it? | Obligations → policy/control/system/report/owner mapping | Stale, duplicated across functions, unowned |

---

## 5. Control Design and the Control Framework

### 5.1 The vocabulary, exactly

Most control failures in a bank's *governance* are not failures of control; they are failures of vocabulary. Two distinctions carry almost all the weight.

**Distinction one: control design versus operating effectiveness.** These are two independent questions:

- **Control design** asks: *if this control operated exactly as specified, would it prevent or detect the risk it is meant to address, in the process where it actually sits?* Design is a property of the control as written and built.
- **Operating effectiveness** asks: *did this control actually operate as designed, consistently, over the period, for the instances tested?* Effectiveness is a property of the control as run, and it is evidenced, not asserted.

Because the two are independent, there are **four combinations**, and each is a different problem with a different owner and remedy:

| Combination | What it means | What it tells you | Remedy |
|---|---|---|---|
| **Well designed + effective** | The control is capable and it ran | The control is doing its job (for the instances tested) | Sustain, monitor, keep testing |
| **Well designed + not effective** | The control is capable but it did not run as specified | A **performance** problem: the control exists, the discipline does not | Fix execution, training, monitoring; re-test; the owner is the first line |
| **Badly designed + effective** | The control runs faithfully but cannot address the risk | A **design** problem dressed as good news: perfect execution of the wrong thing | Redesign the control; operator performance is not the issue |
| **Badly designed + not effective** | Neither capable nor running | A compounding failure: the risk was never actually controlled | Redesign and re-operate; treat urgency as the highest of the four |

The third row is the one that catches banks out, because the control *passes* every execution check — the approvals are there, the reconciliation balances, the review is signed — and the risk is untouched. A control that cannot work, performed flawlessly, produces false comfort at scale. **This design-versus-effectiveness distinction is what the entire testing section (§6) depends on**: a test must state which question it answers, because a test of operating effectiveness cannot rescue a design failure, and a design assessment cannot substitute for testing whether the control ran.

**Distinction two: the control's placement.** A control can only be assessed in the process where it sits. The same nominal control — a four-eyes check, a reconciliation — is a different control depending on where in the flow it occurs and what it sees. Control descriptions that float free of a process ("the team reviews the report monthly") cannot be tested, because there is no defined point at which the control acts and no defined population of instances over which it should have acted.

### 5.2 Preventive, detective, corrective

Controls are conventionally classified by *when they act* relative to the risk event:

- **Preventive controls** act before the risk event to stop it occurring: access restrictions, segregation of duties, mandatory approvals, system validation, pre-trade limits. Their test is whether they are actually preventing (a preventive control has few visible instances precisely because it works), so preventive controls are often evidenced by configuration and population testing rather than by exception samples.
- **Detective controls** act after the event to find it: reconciliations, exception reports, exception-queue reviews, post-trade monitoring, independent reviews. Detective controls generate instances — exceptions — and are tested by examining both the population of exceptions and how they were resolved.
- **Corrective controls** act on a detected problem to fix it and prevent recurrence: remediation workflows, error-correction processes, root-cause actions, recovery procedures.

Two honest observations. First, the widely quoted preference for *preventive over detective* is a genuine design principle — a detective control has already let the event happen, and detects it only if someone acts on the detection — but it is not a rule, because some risks can only be detected, not prevented, and a bank that pretends otherwise builds controls that cannot exist. Second, the **corrector-versus-detector question** is real: many "controls" labelled corrective are in fact the *action taken after detection*, and if the action has no defined trigger, no owner and no verification, it is not a control at all — it is a hope. Corrective controls must be as specified as preventive ones: what triggers them, who performs them, within what time, verified how.

### 5.3 Manual, automated, and IT-dependent manual

The execution-type taxonomy matters because each type has a **different failure mode**, and therefore a **different testing approach** — this is one of the most practically important points in the whole control discipline:

- **Manual controls.** A person performs the control. Failure mode: human inconsistency — done sometimes, done differently by different people, done under time pressure, done without the documentation that would evidence it. Testing approach: sampling of instances, with evidence that the person actually performed the step.
- **Automated controls.** A system performs the control — a validation rule, an access restriction, an automatic reconciliation, a system-enforced limit. Failure mode: silent configuration drift — the rule was correct when configured, and a later system change, parameter change, data change or interface change altered what it does without anyone noticing. Testing approach: configuration inspection, change-management review, and where feasible population testing (test the logic against the full population rather than sampling), with re-testing after system changes.
- **IT-dependent manual controls (ITDMs).** A person performs a judgement, but on data or within a system that must itself be reliable — a review of a system-generated exception report, an analysis of a system-calculated exposure, an approval informed by a system-produced figure. Failure mode: **two possible failures in one control** — the person does not perform the review, *or* the underlying report/catalogue/interface/completeness is wrong, so the person reviews a faithful rendering of wrong information. Testing approach: both halves — test the person's performance *and* test the completeness and accuracy of the underlying report (who owns the report, is the population complete, was the logic changed).

The practical consequence: **an ITDM tested only as a manual control is untested.** The most common control-testing weakness in banks is exactly this — a review of a report is sampled, the signatures are there, the control is declared effective, and nobody ever tested whether the report contained everything it was supposed to contain.

### 5.4 The control library

The **control library** is the bank's single, structured catalogue of its controls. At minimum, each entry must carry: a unique identifier; a description precise enough to be tested (what happens, who does it, how often, over what population, evidenced how); the risk or risks it addresses; the process and sub-process it sits in; the control type (preventive/detective/corrective) and execution type (manual/automated/ITDM); the **owner** — a named first-line owner with the authority to operate it; the frequency; the evidence it produces; the systems involved; and the obligation(s) from the regulatory inventory (§4) it serves.

The control library is the load-bearing artefact of the whole discipline, and its quality is measurable by a single test: *can a competent tester, reading a library entry alone, determine how to test it?* If the answer is no, the library is a list of phrases, and everything downstream — testing, the assurance map, issue severity, board reporting — inherits the ambiguity. The library is shared by the lines: the first line operates against it, the second line tests it, the third line audits it and, where warranted, relies on the work of the second line. The systems that store it are the compliance-systems estate ([Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md)) and, at the process level, the GRC platform (§10).

### 5.5 The control vocabulary table

| Term | Question it answers | Failure mode if confused with the other |
|---|---|---|
| **Control design** | Would this control work if it worked exactly as written? | Confusing design with effectiveness lets a sound-looking control that never ran pass as assurance |
| **Operating effectiveness** | Did the control actually run, as specified, in the instances tested? | Confusing effectiveness with design lets a faithfully performed, incapable control pass as assurance |
| **Preventive** | Does it stop the event? | Treated as automatically better; built where only detection is possible |
| **Detective** | Does it find the event? | Detection recorded but never acted upon — detection without response is not control |
| **Corrective** | Does it fix and prevent recurrence? | Described as a control but given no trigger, owner or verification — an aspiration, not a control |
| **Manual** | A person performs it | Inconsistency; no evidence the step occurred |
| **Automated** | A system performs it | Silent configuration drift after changes; never re-tested |
| **IT-dependent manual** | A person acts on system data | The person's performance is tested and the underlying report's accuracy never is |

---

## 6. Control Testing, RCSA and the Assurance Map

### 6.1 The RCSA: the first line's self-assessment

The **RCSA (risk and control self-assessment)** is the first line's own structured assessment of the risks in a process and the controls addressing them: identify the process, identify the risks to its objectives, map the controls, score the inherent and residual risk, and record the actions committed. Its defining property is in its name: it is **self**-assessment, performed by the owner of the process. That makes it the single most valuable source of risk information in the bank — nobody knows the process's failure modes like the people running it — and it makes it, by construction, **not independent**. This is not a criticism of the RCSA; it is a specification of what the RCSA is for. The RCSA is the first line's own answer; the second line's testing is the independent check on that answer. Treating an RCSA as assurance is the same category error as treating a self-appraisal as a performance review.

For the underlying risk taxonomy and the risk process the RCSA feeds, cross-reference [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §4 and §7 — this guide does not restate them.

### 6.2 Second-line testing as independent testing

**Second-line control testing** is the independent examination of the first line's controls, performed by a function that does not own or operate them, to reach a conclusion on design and/or operating effectiveness. Its independence is the point; its positioning is deliberately distinct from the two things it is often confused with:

- it is **not** the RCSA — the RCSA is the owner's self-assessment; testing is someone else's examination;
- it is **not** third-line audit — second-line testing is a management function's oversight of the first line, part of the control environment and subject to audit itself, whereas the third line's assurance is independent of the second line as well (§7).

The output of second-line testing is a documented conclusion per control: the question asked (design, effectiveness, or both), the method, the sample or population, the evidence, the exceptions found, and the resulting rating — feeding the issue workflow (§8) and the assurance map (§6.4).

### 6.3 The sampling and evidence question

Two questions decide whether a test means anything.

**Population and sample.** A test reaches a conclusion over a defined population of instances (all the approvals in the period; all the reconciliations; all the exceptions raised) and either tests the population (feasible for many automated controls via data analysis) or draws a sample. The honest questions are: is the population complete and can it be shown to be? Is the sample drawn in a way that supports the conclusion being claimed? What does a "pass" on a sample actually assert — that the control operated in the sampled instances, or that it operated throughout? The gap between those two claims is where most testing rhetoric quietly overreaches.

**Evidence.** A finding of "effective" is a claim about what happened, and it must rest on evidence: the actual artefact in which the control acted — the signed approval, the exception in the queue, the system log, the reconciliation, the report. Evidence has properties that make it usable or not: it is contemporaneous (created when the control ran, not reconstructed later for the tester), it is attributable (it shows who performed the control), and it is complete for the instance it represents. Evidence reconstructed after the fact, or evidence of the control's *existence* rather than its *operation*, is the most common source of a comforting but hollow test result.

### 6.4 The assurance map

The **assurance map** is the artefact that answers, for each material risk or control: **who assures it, by what method, at what depth, and at what frequency.** It is a matrix at heart — risk/control along one axis, assurer along the other, cells containing method, scope, depth and timing — and its value is that it forces the coverage question into the open in a way that no individual function's plan can, because each function only sees its own slice.

An assurance map for a bank typically distinguishes at least: first-line monitoring and self-assessment; second-line monitoring, testing and review; third-line audit; and — where they exist and are relevant — independent external assurance (external audit, regulator examinations, third-party certifications, specialist reviews). Depth matters as much as existence: "audit covers it" means something different when the coverage is a walkthrough in a broad engagement than when it is a full substantive test of the control's operation. Frequency likewise: a control assured once three years ago is not a control assured now.

### 6.5 The finding

Assurance mapping exists to make one structural tendency visible:

> **Assurance activity overlaps in the areas everyone considers risky and leaves gaps in the areas nobody owns — and the assurance map is the only artefact that reveals which.**

The mechanism is straightforward once stated. Functions allocate their scarce assurance capacity to the risks that are salient: the ones on the board's list, the ones the last regulator asked about, the ones that had a recent loss, the ones a named executive cares about, the ones that are intellectually interesting. Salience attracts attention, and attention attracts more attention — the high-risk area ends up tested by the first line, monitored by the second line, audited by the third line, and reviewed externally, producing four overlapping views of the same control. Meanwhile the risk that is nobody's flagship — the low-glamour process, the fragmented legacy system, the control owned by a function with no reporting line to anyone who thinks about risk — receives none. It is not that anyone decided to leave a gap; it is that gaps are invisible from within each function's own plan, and gaps go to the areas with no advocate. The assurance map is the only single artefact that shows all the coverage at once and therefore the only place where an overlap and a gap become simultaneously visible. It is, consequently, the artefact most likely to be resisted, because it makes the duplication visible to the functions producing it.

### 6.6 The testing and assurance table

| Artefact | Who produces it | Independent? | What it answers |
|---|---|---|---|
| **RCSA** | First line (process owner) | No — self-assessment | What risks does the owner see, and what controls do they believe address them? |
| **First-line monitoring** | First line, in-process | No | Is the process running as designed today? |
| **Second-line testing** | Second line (risk, compliance) | Yes, of the first line | Did the controls operate as designed, on evidence? |
| **Third-line audit** | Internal audit | Yes, of both | Are the first and second lines doing what the board believes? |
| **External / specialist assurance** | External bodies | Independent of management | Defined scopes, certifications, examinations |
| **Assurance map** | The GRC/assurance-coordination function | n/a (it maps the others) | Who assures what, by what method, at what depth, how often — and where the overlaps and gaps are |

---

## 7. The Third Line

### 7.1 Internal audit independence and its reporting line

Internal audit is the third line: the function that provides **independent, objective assurance** to the governing body on the effectiveness of governance, risk management and control. Its defining property is independence, and independence in internal audit is structural before it is attitudinal. Three structural facts carry it:

- **The reporting line.** Internal audit's functional reporting runs to the board (in practice, via the audit committee): the audit committee approves the audit charter, the audit plan, the audit budget and the appointment and removal of the head of internal audit. Administratively the function sits in the organisation, but its mandate, its plan and its people are accountable to the board, not to the management it audits. This is the same independence architecture as the CRO and CCO (§3.1), applied to the assurance function.
- **Unrestricted access.** Audit must be able to obtain the records, systems, people and explanations it needs, without the management of the area under review being able to filter them.
- **No operational responsibility.** Audit must not design, operate or own the controls it audits. This is where §2.2(ii) lives: the moment audit becomes a designer of the control, its later opinion on that control is a self-review, and the bank has lost the only function whose product was an independent view. Audit may advise on *risk and principle*; it must not become the author of the specific design it will later opine on.

BCBS's *Corporate governance principles for banks* (2015) treats internal audit as one of the bank's control functions and places it within the board's governance framework, alongside the risk and compliance functions — the supervisory version of the same architecture.

### 7.2 The audit universe and the risk-based plan

Internal audit cannot examine everything every year, so it maintains an **audit universe**: the inventory of auditable entities, processes, systems and risks in the bank. From the universe it derives a **risk-based plan**: a schedule of engagements allocated according to the assessed risk of each auditable unit — its inherent risk, the strength of the control environment, the results of prior audits, the volume and nature of change, and, where relevant, regulatory attention and known industry issues. The universe must be *complete* to be useful (the unit absent from the universe is the unit that never gets audited — the audit analogue of the assurance gap in §6.5), and the plan must be *living*, refreshed as the business, its risks and its changes evolve.

The plan's coverage question is the same one the assurance map answers from the GRC side: which risks are audited, at what depth, how often, and which are left to second-line assurance or to first-line monitoring. Where the second line has tested a control well, audit may be able to place some reliance on that work — but only after evaluating the second line's methodology, independence and evidence, because reliance is a judgement audit makes and owns, not a courtesy extended to a colleague.

### 7.3 Findings and ratings

An audit engagement ends in **findings**: statements of a deficiency or risk, supported by evidence, rated on a scale (the exact scale varies by institution — commonly a small number of severity bands from a high-severity rating to a low one), with a management action and an owner. Findings are entered into the issue workflow (§8), where they are remediated and then **validated by audit** before closure — the third line, having raised the finding, is the function that confirms the remediation is real. Audit also renders periodic opinions (on a process, a business, or the overall control environment) and reports to the audit committee on the findings population: how many are open, how severe, how old, and whether the trend is improving.

### 7.4 The relationship to the second line

The second and third lines are both independent of the first, but independent of *each other* in a different way. The second line is part of management: it sets frameworks, gives advice and its work forms part of the control environment the board relies on — so it is itself an object of audit. The third line audits the second line's frameworks, testing methodology, conclusions and independence, and takes a view on whether the second line's assurance can be relied upon. The healthy pattern is complementary coverage with a clear division: the second line assures the controls continuously and deeply; the third line provides periodic, independent verification that the second line's assurance is itself sound. The unhealthy pattern is either the second line treating audit's work as its own substitute (audit is annual; controls fail continuously), or audit duplicating the second line's testing without adding independence value.

### 7.5 The third line table

| Element | What it is | The line it is independent *of* |
|---|---|---|
| **Independence** | Functional reporting to the board/audit committee; unrestricted access; no operational role | Management, including the first and second lines |
| **Audit universe** | The inventory of auditable units, processes and risks | — |
| **Risk-based plan** | Engagements allocated by assessed risk | — |
| **Findings and ratings** | Deficiencies with evidence, severity and owners | Management — findings are management's to remediate, not audit's to implement |
| **Validation** | Audit confirms remediation before closure | The remediating owner's own attestation |
| **Opinions** | Periodic conclusions on the control environment | — |
| **Relationship to the second line** | Audits the second line as part of the control environment | The second line (audit is independent of it, not subordinate to it) |
| **Tooling** | Audit-management systems, continuous auditing, data analytics on control populations | See [Audit as Code Guide](../technology/audit_as_code_guide.md) — not re-derived here |

---

## 8. Issue and Remediation Management

### 8.1 The issue lifecycle

An **issue** is a logged deficiency, gap, weakness, breach or improvement requirement with an owner and a remediation commitment. Issue and remediation management is the workflow that carries it from identification to verified closure, and the lifecycle has six stages that must not be collapsed:

1. **Raise.** The issue is logged from wherever it arises: second-line testing (§6.2), third-line audit (§7.3), monitoring, incidents, self-identified gaps, regulatory feedback, or the business itself. Logging is the moment the deficiency stops being a private observation and becomes an owned, trackable obligation.
2. **Assess.** The issue is assessed for severity, scope and root cause: what happened, what is the effect, which risk does it relate to, which obligations from the regulatory inventory (§4) are implicated, how many processes or entities are affected, and — the discipline that §8.5 makes central — **why** it happened.
3. **Assign an owner.** A single named owner accountable for the remediation, with the authority and resources to complete it, and a committed date. "The controls team" or "operations" is not an owner; a person is.
4. **Remediate.** The fix is designed and implemented: the control corrected or redesigned, the process changed, the system fixed, the population remediated (including back-book or data remediation where the failure affected historical items, which is the part frequently forgotten).
5. **Validate.** The remediation is checked by someone independent of the person who performed it: does the fix actually address the root cause, has it been effective for a sufficient period, does the evidence support the closure? (See §8.4.)
6. **Close.** The issue is closed with its validation recorded, and the closure becomes part of the evidence trail for the board and the regulator.

Ageing, the overdue population, and the proportion of issues past their committed date are among the most informative governance metrics a bank has — which is precisely why they must be reported honestly, and why resetting a date is a governance event rather than an administrative one. **A committed date changed without a documented reason is an issue that has been reclassified from a problem into a statistic.**

### 8.2 Issue vs incident vs breach, exactly

These three words are used loosely in banks, and the looseness costs clarity in exactly the conversations that matter. The exact distinction:

| Term | Definition | Relationship |
|---|---|---|
| **Incident** | A *risk event that has occurred* — something actually happened: a fraud, an outage, an error, a processing failure, an unauthorised access, a data disclosure. The incident is a fact about the past. | An incident can *generate* an issue (the deficiency it exposed) and can *constitute* a breach (if it involved a failure to meet an obligation). |
| **Breach** | A failure to meet a defined obligation — a regulatory requirement, a limit, a policy threshold, a contractual term. The breach is a statement about compliance status, not about harm: a limit can be breached with no loss, and a loss can occur with no breach. | A breach is an obligation-level fact; it may arise from an incident, from a control failure, or from neither (a limit breach caused by a market move). It generates an issue. |
| **Issue** | A logged deficiency, gap, weakness or improvement requirement, derived from an incident, a breach, a test finding, an audit finding, or a self-identified gap, with an owner and a remediation commitment. The issue is the *work item*: the thing that must be fixed. | The issue is the container into which incidents and breaches (and observations that are neither) are converted so that they can be owned and remediated. |

The relationship in one line: **an incident is something that happened; a breach is an obligation that was not met; an issue is the work that follows either.** Conflating them produces the classic reporting muddle — a board pack that counts incidents as if they were issues, misses breaches that caused no incident, and cannot tell how much of the issue population is pending remediation. A bank's GRC record should be able to move between all three: incident → the issue it generated → the breach determination → the regulatory notification (where required) → the remediation.

### 8.3 Escalation

Issue escalation runs on severity and on age. Severity-based escalation routes the most material issues immediately to the accountable executive and the relevant board committee, irrespective of when the issue is otherwise reported. Age-based escalation catches the issue that is not severe individually but has been open too long — the slow-accumulating exception that becomes an examination finding precisely because everyone has seen it for months and nobody has fixed it. The escalation path must be pre-defined and independent of the owner's preference: an owner who can choose whether to escalate has, in effect, no escalation. Materiality is bank-specific and jurisdiction-specific; the design principle is that the *rule* is defined in advance, not the *outcome*.

### 8.4 Validating remediation rather than accepting an attestation

The weakest link in most issue workflows is closure. Two closure patterns look similar and are not:

- **Closure on attestation.** The owner, or the owner's management, states that the remediation is done, and the issue is closed. The attestation may be entirely honest and still wrong: the owner believes the fix works because they built it, which is the natural bias of the person who did the work.
- **Closure on validation.** Someone independent of the remediation examines the fix — re-performs the control, secures evidence that it operated over a period, tests the population the fix was meant to cover — and reaches a conclusion. Only then is the issue closed.

Validation is the difference between a closed issue and a fixed one. It is also the discipline that keeps the audit function's and the second line's credibility intact: an issue closed on an attestation that later proves hollow is a finding about the validation process, and it is more damaging than the original issue, because it means the bank's assurance machinery reported a cure that did not exist.

### 8.5 Root-cause discipline

The centre of this section is one sentence:

> **An issue closed without a root cause recurs, and a recurring issue is the finding an auditor values most.**

The mechanism is simple and it is why root cause is not a paperwork field. An issue describes a symptom — a control that did not operate, a report that was wrong, an approval that was missing. The symptom's immediate cause ("the operator forgot", "the interface dropped a field", "the reviewer was on leave") is almost never the whole cause; behind it sit conditions — an unclear procedure, a system that invites the error, an incentive that rewards speed over accuracy, an ownership gap, a control designed by someone who did not understand the process. A remediation that fixes only the symptom leaves the conditions intact, and the conditions reproduce the symptom. The recurrence is then worse than the original issue in three ways: it proves the first remediation failed, it proves the root-cause process failed, and it demonstrates to the auditor and the regulator that the bank's remediation is cosmetic.

The practical discipline: require a root cause (not just a cause) before a remediation plan is accepted; distinguish the immediate cause from the contributing conditions; test whether the proposed action addresses the conditions and not merely the instance; and treat any recurrence of a previously closed issue as a governance event in its own right, because a recurrence is the only hard evidence that a prior closure was wrong.

### 8.6 The issue management table

| Stage | Question | Artefact | Failure mode |
|---|---|---|---|
| **Raise** | What deficiency are we recording? | The issue record | Issues resolved verbally and never logged |
| **Assess** | How severe, how wide, and **why**? | Severity rating, scope, root cause | Cause recorded, root cause skipped |
| **Assign** | Who owns the fix? | A named owner and a committed date | Ownership diffused to a team name; dates unenforced |
| **Remediate** | What is actually being changed? | The remediation plan and its implementation | Symptom fixed; historical population not remediated |
| **Validate** | Is it really fixed, independently? | The validation record and evidence | Closure on the owner's attestation |
| **Close** | Can we prove it to a regulator? | The closure record in the evidence trail | Closed without evidence; re-opened at the next review |
| **Ageing** | What is overdue, and why? | The ageing report and date-change log | Dates reset silently; overdue population hidden |

---

## 9. The KRI Framework

### 9.1 Resist the dashboard trap

A **key risk indicator (KRI)** is a metric that gives an early or current signal of the level or trajectory of a risk, with a defined threshold and a defined action when the threshold is breached. The **dashboard trap** is the failure mode the framework exists to avoid: a bank collects metrics because they are available, presents them on a dashboard because it looks like risk management, and neither the metrics nor the dashboard change a single decision. The trap is seductive because a dashboard looks like control. A screen full of charts is evidence of *monitoring activity*, not of *risk management*; the two diverge precisely when it matters.

The test of a KRI is not whether it is interesting but whether it is **actionable**: when the metric moves, is there a defined response, a defined owner, and a defined threshold that triggers it? A metric with no threshold and no consequence is a *measurement*, not an indicator, and it belongs in analysis, not on the risk dashboard.

### 9.2 KRI versus KPI

The two are constantly conflated. A **key performance indicator (KPI)** measures how well the business is achieving its objectives — volume, revenue, efficiency, customer metrics. A **KRI** measures the level or trajectory of a risk. The distinction has an operational consequence: KPIs generally want to go up (or down, for cost), while KRIs are read against *tolerance* — a KRI's "good" value is a band, and both a deterioration and an unusual improvement can be meaningful. Some metrics can serve both roles, but not simultaneously without confusion: the same number presented as a performance measure and as a risk measure will be optimised by the business as a performance measure, which is exactly the behaviour the KRI was meant to detect. When a metric is genuinely dual-purpose, the honest approach is to state both readings and to be alert to the fact that its use as a KPI will influence its value as a KRI.

A related and recurring trap: **compliance-activity metrics masquerading as risk indicators**. Training completion rates, number of policies refreshed, number of attestations collected, number of reviews completed — these measure *activity*, not risk level. They belong on a programme-reporting page, not among the indicators the board uses to judge whether the bank's risk is rising or falling.

### 9.3 Leading versus lagging

**Lagging indicators** report what has already happened: losses incurred, incidents recorded, breaches logged, audit findings raised. They are reliable, auditable, and late — by the time a loss indicator worsens, the risk has already crystallised. **Leading indicators** are intended to give warning before the event: overdue remediation, control-testing exceptions, unresolved access reviews, ageing of unreconciled items, escalation volumes, near-miss reports, staff-turnover in control-critical roles. Leading indicators are more useful and less reliable — a leading indicator is a hypothesis about what precedes the failure, and it can be wrong or can be gamed. The mature use of the two is paired: lagging indicators as the ground truth about outcomes, leading indicators as the early-warning layer whose predictive value is periodically tested against the lagging outcomes, and any leading indicator that has never once preceded anything is a candidate for pruning.

### 9.4 Thresholds and escalation design

A KRI without a threshold is an observation; a KRI with a threshold is an instrument. The design elements:

- **A defined baseline and a defined threshold**, stated in advance, tied to the risk appetite and tolerance the bank has set (cross-reference [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §6 for appetite, tolerance and limits — this guide does not re-derive it).
- **A defined action per level.** The threshold is only half the design; the other half is what happens when it is crossed — who is notified, within what time, what authority is activated, what response is required. A threshold with no action is a colour, not a control.
- **A defined owner** for the indicator and for the response.
- **A defined frequency** matched to how fast the risk can move: a risk that can move in a day needs a daily indicator, not a quarterly one; a quarterly indicator on a fast-moving risk is a lagging indicator wearing a leading indicator's label.

The recurring debate about whether to set thresholds as absolute values, trends, or both is genuinely open and varies by institution and by metric: an absolute threshold detects a bad level; a trend threshold detects deterioration from a good level, which is often the earlier and more useful signal. **No specific threshold, target or benchmark is asserted in this guide** — thresholds are bank-, risk- and metric-specific, and any published "industry norm" for a KRI threshold should be interrogated before adoption.

### 9.5 Reporting linkage and the pruning discipline

KRIs matter because they feed the reporting chain (§11): the board's view of the risk trajectory is assembled substantially from indicators, and an indicator that is wrong, stale or unowned corrupts the pack at the source. The reporting linkage also runs in the other direction — once a KRI is in the board pack, the board may ask questions about it, which is how an ill-designed indicator is discovered (nobody can answer the question, and the metric quietly disappears next quarter).

That brings the second half of the section's finding:

> **An indicator nobody can act on is decoration, and an indicator set that grows without pruning stops being read.**

The mechanism of the second clause is worth stating precisely, because it is not obvious to the people adding indicators. Each new indicator is added for a good local reason — a new risk, a regulatory request, an incident, a committee's curiosity — and each addition increases the length of the pack. The pack has a fixed reading cost, borne by the same few senior people every cycle. Past a certain length, readers triage: they read the first few, skim the rest, and rely on the commentary to tell them what matters. The indicators at the bottom of the pack, added last and least connected to a decision, are the ones that stop being read — and because nobody is reading them, nobody notices when they move. The set has grown into a place where a real signal can hide. Pruning is therefore not housekeeping; it is the mechanism that keeps the surviving indicators meaningful. The discipline: every indicator has an owner, a threshold, an action and a review date; every indicator is periodically challenged with *what decision would this change?*; and indicators that have never triggered a response are retired, with the retirement recorded, so that the set is a decision instrument rather than an archive.

### 9.6 The KRI table

| Dimension | Design question | Failure mode |
|---|---|---|
| **KRI vs KPI** | Does it measure risk, or performance? | Performance metrics presented as risk indicators; optimised for the wrong purpose |
| **Leading vs lagging** | Does it warn, or report? | Lagging indicators relabelled as leading; leading indicators never validated against outcomes |
| **Baseline and threshold** | What is normal, and what is a breach? | Threshold absent, or set with no basis; "industry norm" adopted unexamined |
| **Action per level** | Who does what when it moves? | Escalation colour with no defined response |
| **Owner** | Who owns the indicator and the response? | Indicator with no owner, refreshed by whoever notices |
| **Frequency** | How fast can the risk move? | Quarterly indicators on daily risks |
| **Reporting linkage** | Which decision does it inform? | Indicator in the pack that no reader can act on |
| **Pruning** | What decision would this change? | The set grows; the bottom of the pack stops being read |

---

## 10. The GRC Platform Layer

### 10.1 What a GRC platform structurally does

A **GRC platform** is software that records and orchestrates governance, risk and compliance activity. Distinct from the analytical systems that *measure* risk (the models in [Risk Management Models Guide](risk_management_models_guide.md)) and from the transaction systems that *process* the business, a GRC platform is a workflow-and-register system, and structurally it does five things:

1. **Registers.** It holds the bank's structured inventories: the risk register, the regulatory inventory of obligations (§4), the control library (§5), the issue register (§8), the policy register, the KRI register (§9), and the mapping between them. The value is the *mapping*: an obligation links to controls; a control links to risks, to a process and to an owner; a failed test links back to the control and forward to an issue.
2. **Control library management.** It stores each control with its attributes (§5.4), versions it, and lets the same control be referenced by testing, by the issue workflow and by reporting without being retyped.
3. **Assessment workflow.** It orchestrates the assessment cycle: issuing RCSA campaigns to process owners, tracking completion, capturing scores and evidence, routing for review and sign-off, and versioning the assessment so the bank can show what it believed at a point in time.
4. **Issue workflow.** It carries an issue through raise → assess → assign → remediate → validate → close (§8.1), enforces dates, records evidence, and produces the ageing report.
5. **Reporting.** It renders the registers and workflows into the views the organisation needs: the control-effectiveness view, the issue-ageing view, the obligation-coverage view, the assurance map (§6.4), and the board pack inputs (§11).

### 10.2 The data problem underneath

A GRC platform is **only as good as the taxonomy and the ownership model it encodes.** This is the point that decides whether the platform project succeeds, and it is entirely independent of the software chosen.

- **The taxonomy.** If the bank has not settled what a risk is, how risks are classified, what distinguishes a control from a process step, and how obligations map to controls, then the platform will faithfully record the bank's confusion at scale — thousands of entries that cannot be aggregated, because they mean different things. A platform cannot impose a taxonomy on an organisation that has not agreed one; it can only store the one it is given, or force an ill-fitting one that the business then works around.
- **The ownership model.** If the bank has not settled who owns each risk, each control, each obligation and each issue — by name, with the authority that ownership implies — then the platform's fields will be filled with proxies (a team name, a role title, a shared mailbox), and the workflow will route to nobody. The platform's escalations, reminders and ageing reports all depend on there being a real person at the end of each record.
- **The data quality.** Even with a good taxonomy and ownership, the platform inherits the quality of the data fed into it. Where the underlying registers are migrated from spreadsheets, the migration encodes the spreadsheets' inconsistencies.

The practical consequence: the taxonomy and the ownership model are **design decisions that must be made before, or at the same time as, platform selection** — not discovered through configuration. The GRC guide's siblings describe the data foundations the platform depends on: the aggregation and quality discipline of [Risk Data Aggregation Guide](risk_data_aggregation_guide.md) (BCBS 239) for firm-wide risk data, and the ownership-and-lineage discipline of [Data Governance Guide](../technology/data_governance_guide.md) for the data itself.

### 10.3 The failure mode

> **A platform does not create a GRC programme; it records one. Buying the tool before the process produces an expensive inventory of nothing.**

The mechanism is a familiar procurement sequence with a predictable end. A bank forms the view that its GRC is fragmented, and the fragmentation is visible — five registers, three evidence formats, no single view. A platform is procured to *fix* the fragmentation. The platform is installed and configured; the vendors' implementation templates supply a default data model; the business is asked to populate the registers. Two things then happen. First, the default data model does not match the bank's actual processes, so the registers fill with entries that satisfy the form and describe nothing testable. Second, nobody has defined the workflow that the platform is meant to digitise, so the platform's campaigns (RCSA, attestation, testing) are launched on top of a process that does not exist — and completion is measured instead of quality. Two years later the bank has a paid-for system containing a large volume of low-integrity records, a business that has learned the platform is a reporting burden, and the original fragmentation intact underneath, now expressed in a new format.

The inversion is the lesson: **process before platform.** Define the taxonomy, the control library, the ownership model, the assessment cycle and the issue workflow first — even in a spreadsheet, deliberately and with the business — and let the platform be the instrument that scales a working process. A working process on spreadsheets can be migrated; a broken process on a platform cannot be fixed by the platform.

### 10.4 Distinguishing the layers by name

Three different layers are constantly conflated, and the boundary between them is worth stating plainly:

- **The GRC platform layer (this section).** The workflow-and-register system: risks, obligations, controls, assessments, issues, assurance mapping (§6.4), reporting. It is the *record* of the discipline.
- **The RegTech layer** ([RegTech Guide](regtech_guide.md)). The regulatory-technology capability — the systems that help the bank meet specific regulatory obligations, including regulatory reporting, monitoring, and the technology of compliance operations. That guide owns the layer; this guide does not re-derive it.
- **The compliance-systems layer** ([Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md)). The specific platforms performing compliance *processes* — surveillance, case management, screening, reporting systems — the operations that GRC records and assures.

A GRC platform does not perform surveillance, screen customers, or produce a regulatory return. It records that those things are controlled, owned, tested, and reported. Confusing the record with the operations is the same category error as confusing a risk register with risk management.

### 10.5 On vendors: a deliberate constraint

**No vendor product is recommended, compared, or attributed any capability beyond its own published documentation in this guide.** This is not a gap to be filled by the reader's favourite vendor; it is the position the evidence supports. Vendor capability claims are marketing artefacts, the sector's product landscape changes continuously, and a guide that ranked vendors would be stale on publication. The durable content is the structural one: what a GRC platform does (above), the data problem it inherits (above), and the sequencing failure to avoid (above). Those are true of any platform, and they are what a selection process should test a candidate product against — not against a list in a guide. The warning is the value here, not a survey.

### 10.6 The GRC platform table

| Layer / element | What it is | What it is not |
|---|---|---|
| **GRC platform** | Registers, control library, assessment workflow, issue workflow, reporting | An analytical engine; a transaction system; a substitute for process design |
| **Risk register** | The structured inventory of risks with owners and assessments | A spreadsheet circulated by email |
| **Regulatory inventory** | Obligations mapped to implementing artefacts (§4) | A bibliography of regulation |
| **Control library** | The testable catalogue of controls (§5.4) | A list of phrases |
| **Assessment workflow** | RCSA campaigns, scoring, review, versioning | A form-completion exercise |
| **Issue workflow** | Raise → assess → assign → remediate → validate → close | A status list |
| **The data problem** | Taxonomy + ownership model + data quality | Something the platform can supply |
| **The sequencing rule** | Process before platform | Procurement first, process never |
| **Adjacent layers** | RegTech ([RegTech Guide](regtech_guide.md)); compliance systems ([Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md)) | Part of the GRC platform |

---

## 11. Board and Committee Reporting

### 11.1 What the board is accountable for receiving

The board (and its risk, audit and other committees) is accountable for oversight, and oversight is only possible over information that is true, material, timely and comprehensible. The board is accountable for *receiving* — and, more sharply, for being able to *act on* — a defined set of things:

- **The risk and compliance profile against appetite**: where the bank stands relative to the appetite and tolerance it has set. (Risk appetite, capacity, tolerance and limits are owned by [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §6 and are referenced by name here, not re-derived.)
- **The material issues and their remediation status**: the significant open issues, their age, their ownership and whether remediation is on track — the issue-ageing view of §8.
- **The assurance position**: what the second and third lines have assured, at what depth, and where the assurance gaps are — the assurance map (§6.4) in summary.
- **The control environment's health**: control-testing results, the trends and the concentrations.
- **The obligations position**: whether the bank is compliant with its changing obligations — the regulatory-change and inventory view (§4).
- **The escalations**: the matters referred upward by the escalation paths of §8.3 and §3.4, because an escalation path that does not reach the board is not a path.

### 11.2 The risk-and-compliance pack

The board's risk-and-compliance pack is the artefact that carries all of the above. Its design has a tension at its heart: the pack must be complete enough to support oversight and short enough to be read. The components that earn their place:

- **A cover narrative** stating, in a few paragraphs, what changed since the last pack and what the board is being asked to note, approve or decide. The narrative is where judgement is conveyed; a pack of tables with no narrative is a data dump.
- **The appetite position**, with exceptions and breaches flagged and explained.
- **The issue population**, with the material items individually and the rest in aggregate — ageing, trend, and overdue concentration.
- **The assurance summary**: coverage, gaps, and any change in the assurance picture.
- **The regulatory position**: material regulatory changes, their implementation status, and any supervisory interactions.
- **The indicators** (§9), with thresholds and actions — the surviving, actionable ones.
- **The escalations and requests**: what the board is being asked to decide.

### 11.3 The appetite linkage

The pack is not a standalone report; it is the top of a chain that runs back through the bank's appetite. The chain: the board approves the risk appetite and limits ([Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §6) → the appetite cascades to limits, KRIs and thresholds (§9.4) → the business operates against them (first line) → the second line monitors and tests (§6.2) → issues are raised and remediated (§8) → all of it is assembled into the pack. The reporting requirement that makes the chain real is **traceability**: every red or amber in the pack should be traceable to the appetite line it relates to, and every material issue to the obligation and control it implicates. When that traceability exists, the board is genuinely overseeing the risk; when it does not, the board is reading a summary that correlates with the risk by coincidence.

### 11.4 The finding

> **A pack that reports everything reports nothing.**

The mechanism is the same one as §9.5's indicator sprawl, at a larger scale. The pack's length is set by the aggregation of everything each function wants the board to see, and each function's request is individually justified. But the board's reading capacity is fixed and small — the board does not read the pack; a small number of its members read it, and they read it against everything else they must attend to. Past a certain length, the pack's function changes: it stops being the instrument by which the board oversees risk and becomes the record by which management demonstrates that it reported. The material item, buried among forty slides of routine detail, gets the same attention as the routine item, which is to say: the reader has no way to know which is which, so both are skimmed. The failure is not that the board was misled; it is that the pack gave the board no means of discrimination, and a board that cannot discriminate cannot challenge.

The remedy is editorial discipline applied *before* the pack is written, not after: a defined materiality threshold that keeps immaterial items out of the pack (they live in the appendices and the committee papers), an explicit statement of what is *not* in the pack and why, a requirement that every item have an action or a decision attached or be removed, and a comparison against the previous pack so the board can see what changed. The pack's quality is measured by what the board does with it — the questions asked, the decisions taken, the challenges recorded — and a pack that generates no questions and no challenges is either a perfect bank or an unread report.

### 11.5 The board reporting table

| Component | What the board needs from it | Failure mode |
|---|---|---|
| **Cover narrative** | What changed; what is being decided | Tables with no judgement; no ask |
| **Appetite position** | Where the bank stands against appetite | RAG colours with no thresholds or traceability |
| **Issue population** | Material issues, age, ownership, trend | Everything listed equally; no severity discrimination |
| **Assurance position** | What is assured, where the gaps are | "Assurance is in place" with no map behind it |
| **Regulatory position** | Material changes and their implementation | Change reported as commenced, never as evidenced |
| **Indicators** | Trajectory, thresholds, actions | Sprawling sets; activity metrics among risk indicators |
| **Escalations** | The matters requiring board attention | Escalation paths that stop below the board |
| **The finding** | A pack the board can act on | A pack that reports everything and therefore nothing |

---

## 12. The Domains GRC Must Integrate With

GRC does not operate in isolation. Four adjacent governance domains each own a body of work, and each must *hand* something to the integrated GRC layer rather than running its own private assurance. This section states, per domain, what the domain owns and what it hands over; the detail lives in the domain's own guide.

### 12.1 Data governance

**What the domain owns:** the governance of the bank's data as an asset — the data ownership model, the data-quality standards and remediation, the lineage and metadata, the data lifecycle, and the accountability for data definitions ([Data Governance Guide](../technology/data_governance_guide.md)). On the risk side, the firm-wide risk data aggregation and reporting discipline is owned by ([Risk Data Aggregation Guide](risk_data_aggregation_guide.md)) (BCBS 239).

**What it hands to GRC:** the ownership model and data-quality evidence that the GRC platform depends on (§10.2), the lineage that makes an obligation-to-report traceability claim provable, and the data risk assessments that feed the risk register.

### 12.2 AI governance

**What the domain owns:** the governance of the bank's AI and generative-AI estate — the model inventory and approvals, the use-case risk assessment, the controls specific to AI (bias, explainability, human oversight, monitoring, model change), and the AI-specific regulatory expectations ([AI Governance Framework Guide](../technology/ai_llm/ai_governance_framework_guide.md); for the banking compliance overlay, [AI & GenAI Banking Compliance Guide](ai_genai_banking_compliance_guide.md)).

**What it hands to GRC:** the AI use-case register and its risk ratings, so that AI risk appears in the enterprise risk view and in the board pack rather than living in a parallel AI-governance forum; the AI controls, so that they enter the control library and the testing plan; and the AI-specific obligations, so that they enter the regulatory inventory (§4).

### 12.3 Technology and operational resilience

**What the domain owns:** the resilience of the bank's important business services — impact tolerances, mapping of services to assets, severe-but-plausible scenario testing, and the technology risk and change-management disciplines ([Operational Resilience Framework Guide](operational_resilience_framework_guide.md); the security side in [Cybersecurity Guide](../technology/cybersecurity_guide.md)).

**What it hands to GRC:** the resilience assessment results and the identify-map-test evidence, so that resilience appears as an assured control set rather than a separate programme; the technology-risk register and the change-management control evidence; and the third-party dependencies that overlap with §12.4.

### 12.4 Third-party risk

**What the domain owns:** the governance of the bank's suppliers and third parties — due diligence, contracting and exit provisions, ongoing monitoring, concentration and subcontractor risk, and the domain's own assurance of critical service providers ([Vendor Management Guide](../management/vendor_management_guide.md); the resilience-of-critical-suppliers angle in [Operational Resilience Framework Guide](operational_resilience_framework_guide.md)).

**What it hands to GRC:** the supplier risk register and criticality ratings, so that third-party risk enters the enterprise risk view; the assurance obtained over critical suppliers (audit reports, certifications, on-site reviews), so it enters the assurance map (§6.4) rather than being invisible; and the third-party obligations and controls, so they enter the inventory and the control library.

### 12.5 The integration failure

The four subsections above describe a natural arc, and its failure mode is the reason this section exists:

> **A bank running each domain's governance separately duplicates the very evidence requests that GRC exists to consolidate.**

The mechanism is a replay of §2.2(iv) with four domains instead of three functions. Data governance asks the business for data-quality evidence; AI governance asks for AI use-case assessments; resilience asks for service mapping and test results; third-party risk asks for supplier assurance. Each request is justified within its own domain, each is formatted for its own domain's needs, and each arrives at a business unit that is already answering the second and third lines. The domains do not intend to duplicate; duplication is the emergent consequence of four separate governance programmes with four separate evidence taxonomies and no shared identifier set. The remedy is not to merge the domains — each needs its own expertise and its own guide — but to make them **consumers of, and contributors to, a shared record**: the same control library, the same ownership model, the same register identifiers, and the same assurance map. When a domain's assurance is entered into the common map, the overlap and the gap both become visible — and the business answers the request once.

### 12.6 The integration table

| Domain | Owns | Hands to GRC | Its guide |
|---|---|---|---|
| **Data governance** | Data ownership, quality, lineage, definitions | Ownership model; quality evidence; data risk assessments; lineage for traceability | ../technology/data_governance_guide.md; risk_data_aggregation_guide.md (BCBS 239) |
| **AI governance** | AI inventory, use-case risk, AI controls and monitoring | AI register and ratings; AI controls; AI obligations | ../technology/ai_llm/ai_governance_framework_guide.md; ai_genai_banking_compliance_guide.md |
| **Technology & operational resilience** | Important business services, impact tolerances, scenario testing, technology risk | Resilience evidence; technology-risk register; change-management evidence | operational_resilience_framework_guide.md; ../technology/cybersecurity_guide.md |
| **Third-party risk** | Supplier due diligence, contracting, ongoing monitoring, concentration | Supplier risk register; supplier assurance; third-party obligations and controls | ../management/vendor_management_guide.md |
| **The failure** | Four parallel governance programmes | — | Duplicate evidence requests; no shared identifiers; invisible overlaps |

---

## 13. Individual Accountability

### 13.1 The regime, treated structurally

A **senior-manager or responsible-person regime** is a regulatory architecture that assigns defined responsibilities to named individuals, so that when something goes wrong there is a person — not a committee, not a function, not a document — who was accountable. The jurisdiction-specific mechanics differ (which roles are covered, how responsibilities are prescribed and recorded, how fitness and propriety are assessed, how breaches are handled); the specifics vary and are not asserted here. What is constant across such regimes, and what this section treats, is the **structure**: a named individual is assigned a defined area of responsibility, undertakes defined obligations in respect of it, and is accountable for its governance.

Two design artefacts make the structure real. The **responsibility map** records which named individual holds which responsibility, with no gaps and no unassigned overlaps — the individual-accountability analogue of the assurance map's coverage question (§6.4), and subject to the same tendency to leave the unfashionable areas unassigned. The **reasonable steps** question — what a person in that role is expected to have done to discharge the responsibility — is a structural one: it asks whether the person had the information, the authority and the resources to govern the area, or whether they were nominally accountable for something they could not see or steer.

For the craft of assigning and discharging individual responsibility — how a responsibility is defined, what a reasonable-steps narrative looks like, how responsibility is evidenced — cross-reference [Directly Responsible Individual Guide](../management/directly_responsible_individual_guide.md); this guide treats the GRC-structural dimension and does not re-derive that craft.

### 13.2 The attestation

An **attestation** is a formal, signed declaration by a named accountable person that a defined state of affairs is true as at a stated date — that the controls in their area operated, that the obligations in their area are implemented, that the information they are reporting is accurate. In a GRC programme, attestations are the mechanism by which accountability is exercised as a recurring act rather than a one-off appointment: the accountable person certifies, on a cadence, the state of what they are accountable for.

The attestation's value is entirely a function of two things: the **evidence** behind it, and the **accountability** of the person signing. An attestation with no evidence is a leap of faith recorded in a system; an attestation by a person who is not genuinely accountable (because they cannot see the area, cannot direct it, or will not be answerable for it) is a signature, not a control. The recurring attestation-design failure is the **boilerplate attestation**: a standard paragraph, circulated to many signatories, covering a scope so broad that no signatory can honestly verify it. The signatory signs because signing is expected and refusing is conspicuous, and the attestation becomes a liability-transfer device — it moves the paper risk to the signatory without moving any real assurance to the organisation.

### 13.3 The constraint

> **An accountable individual must be able to DIRECT the control they are accountable for, or the accountability is a liability assignment rather than a control.**

The distinction is the whole point of the section, and it is structural rather than motivational. To *direct* a control, the accountable individual needs: **information** — the data, reports and escalations that reveal whether the control is working; **authority** — the power to change the control, its resources, its procedures and, if necessary, the people operating it; and **reach** — the control must lie within the scope of what they actually run. Where all three are present, accountability is a control: the person has both the means and the incentive to keep it working, and the regime gives the organisation a named lever to pull. Where any is absent, the regime has produced a **liability assignment**: the person carries the consequence of a failure they could not have prevented, while the actual control sits with someone who is not named. A bank can comply perfectly with the paperwork of a senior-manager regime and have no functioning individual accountability at all, if its responsibility map assigns people to areas they cannot see and cannot change.

The corollary for the GRC record: the responsibility map and the control library must be consistent, and where they are not, the discrepancy is a finding. If the accountable person for a business area is not the owner of that area's controls in the control library, one of the two is wrong, and the organisation should know which before a supervisor asks.

### 13.4 The accountability table

| Element | Purpose | Failure mode |
|---|---|---|
| **Responsibility map** | Assign every area of responsibility to a named individual; no gaps, no unassigned overlaps | Gaps and overlaps; responsibilities assigned to those who cannot direct them |
| **Reasonable steps** | Define what discharging the responsibility requires | Judged on outcomes alone; or defined so vaguely nothing is expected |
| **Attestation** | Recurring certification of the state of the accountable area | Boilerplate scope; signatures without evidence; liability transfer |
| **Information** | The accountable person can see whether the control works | Blind accountability |
| **Authority** | The accountable person can change the control and its resources | Nominal accountability; the control sits elsewhere |
| **Reach** | The control is within the scope the person runs | Accountability for another function's control |
| **Consistency with the control library** | Owner and accountable person align | Two inconsistent records; discovered only on challenge |
| **The craft** | How responsibility is defined and evidenced | See [Directly Responsible Individual Guide](../management/directly_responsible_individual_guide.md) |

---

## 14. The Cymbal Bank Worked Example

### 14.1 The scenario

**Cymbal Bank is a fictional institution used here for illustration only.** It is the sole bank persona in this guide, it is fictional from end to end, and nothing in this section should be read as a statement about any real institution, any real vendor, or any real consultancy: no real bank, vendor or consultancy is asserted to use, recommend, deploy, or have built anything described here. The figures and artefacts are pedagogical constructions.

Cymbal Bank operates a corporate and investment banking business across four lines: global markets, structured finance, trade finance, and corporate banking, with a group function and an APAC hub. It runs the three lines described in §2 and the compliance function of §3. It has a control library (§5), an RCSA cycle (§6), a second-line testing programme (§6.2), and internal audit (§7). It has a GRC platform, purchased two years ago (§10). And it has just had a **repeat control failure**: a control that was found deficient, remediated, closed — and then failed again in the same way.

The board has asked for a GRC review. The review works through five findings in order, then reaches a decision.

### 14.2 (a) The issue that was closed without a root cause, and recurred

The failure involved a reconciliation control in the trade-finance operations area: a daily reconciliation between a processing system and the general ledger. The first time, the second line identified that the reconciliation had not been performed on a number of days in the period. The control library entry named an owner, a frequency (daily) and an evidence type (sign-off). An issue was raised, assessed as medium severity, assigned to the operations team lead, remediated by sending a reminder to the reconciliation team and re-issuing the procedure, and closed three weeks later on the team lead's attestation that the reconciliation was now being performed. No independent validation was performed; no root cause was recorded beyond "operator oversight".

Eight months later, the same control failed again: the reconciliation was again not performed on a run of days. This time internal audit raised the finding, and the assessment went further. The root cause was not operator oversight. The reconciliation was a manual step at the end of a process that, during a peak period, ran past the end of the operations shift; the people accountable for the step were on a shift pattern whose handover did not include it; and the procedure said "daily" without saying by whom, when, or what to do when the day's processing ran late. The reminder and the re-issued procedure had addressed none of that. The conditions that produced the first failure were untouched, so the first failure reproduced — and the recurrence was worse than the original, because it demonstrated that the prior remediation had been cosmetic and that the issue workflow, which had permitted closure without root cause and without validation, could be relied upon to close problems without fixing them.

**The review's finding:** the issue was closed on attestation, not validation (§8.4), and without a root cause (§8.5). The recurrence is the evidence that the closure was wrong.

### 14.3 (b) Three functions, three evidence requests, one business unit

While tracing the reconciliation, the reviewers found that the same trade-finance operations unit had, in the same quarter, been asked for substantively the same evidence three times, in three formats, by three functions. The second line's control testing had requested a sample of the reconciliation sign-offs in the second line's testing template. Internal audit, in an unrelated engagement, had requested the same sign-offs in the audit evidence request format. And the first line's own compliance attestation for the quarter had required the operations unit to confirm in the attestation workbook that reconciliations were performed. The three requests arrived at the same team, in the same quarter, requesting the same underlying records, formatted three different ways.

The consequence was exactly the mechanism of §2.2(iv): the operations team spent time re-formatting the same facts three times; each of the three consumers held a partial, differently-cut version; the audit request and the second-line sample overlapped but were not reconciled; and the mismatch between what the attestation declared and what the sample showed became, in due course, a further finding about data consistency. The duplication was not caused by malice or wastefulness; it was the emergent consequence of three charters, three evidence taxonomies and no shared identifier set.

**The review's finding:** the three-lines integration of §2.3 — one narrative, assessed once, reported to three audiences — was absent in practice, and the shared identifier set that would have prevented the duplication did not exist.

### 14.4 (c) The regulatory change implemented but never inventoried

The reviewers then looked at why the reconciliation had never been strengthened after the first failure, because a change in the applicable reporting obligations — a change to a regulatory return that drew on the reconciled data — should have triggered a review of the control. They found the change had been implemented. The reporting team had updated the return, the data team had adjusted the extract, and the change had been delivered to schedule. What had **not** happened was that the change had ever been entered into the bank's regulatory inventory.

Because the obligation was not in the inventory, it had no mapping to the reconciliation control that fed the return. The impact assessment had therefore reached the data model and the reporting pipeline but not the control that produced the data's integrity. The control library entry for the reconciliation made no reference to the return, so nobody in the control-loop had a reason to revisit it when the return changed. The bank was, on the evidence, meeting the new obligation — the return was being filed correctly — but it could not *demonstrate* the chain from obligation to control to evidence, and when the reviewers asked which obligation the reconciliation served, there was no artefact that answered.

**The review's finding:** this is the central finding of §4.3, in the flesh. The obligation was implemented but not inventoried, and the regulatory inventory was found to exist in three partial versions — one in compliance, one in risk, one in the reporting team's own tracker — with no single authoritative instance and no named owner for the inventory as an artefact.

### 14.5 (d) The assurance map that revealed a control nobody assured

The reviewers then did the thing that no individual function could have done for itself: they assembled the bank's assurance activity onto a single map (§6.4) — every control, and for each, which functions assured it, by what method, at what depth, at what frequency. What the map showed was the §6.5 finding, concretely.

The trade-finance reconciliation sat at the intersection of a business area that had been the subject of three recent audits, two second-line reviews and the repeated testing generated by its failures: it was assured many times over, by multiple functions, at multiple depths. Elsewhere on the map, a cluster of controls in a back-office support process — not glamorous, not recently on the board's list, owned by a function with no direct line to the risk committees — had a single entry: the first line's own self-assessment. No second-line testing, no audit coverage, and no evidence that anyone outside the process owner had ever looked at it. The map made both facts visible at once, which is precisely what each individual function's plan could not do: the second line's plan showed its own coverage, the audit plan showed its own, and the gap between them was visible only when overlaid.

The reviewers' harder observation was that the map had never been produced before, because no function owns the whole picture, and each function's coverage looked complete within its own plan.

**The review's finding:** assurance overlapped where everyone already agreed the risk was, and left a gap where no one owned it — and the map was the only artefact that revealed it. The bank had no function responsible for maintaining it.

### 14.6 (e) The decision: process before platform

The reviewers' final observation was about the GRC platform. Cymbal Bank had purchased it two years earlier, before the review, precisely because its GRC had felt fragmented. The platform had been populated: it contained a risk register, a control library, an issue register and an attestation module. It also contained, on inspection, the four problems the review had found — expressed in the platform's own data model. The control library entry for the reconciliation still described a control that could not be tested as specified (no defined performer for the "daily" step); the issue register recorded the first issue as closed, with no root cause field populated beyond "operator oversight"; the obligation that had changed was absent from the platform's obligation module, because the module had been populated at go-live and not maintained; and nothing in the platform held the assurance map, because the design decision about who assures what had never been made and so had never been configured. The platform was an accurate, well-organised record of a GRC programme that did not work.

Cymbal Bank's decision was therefore not to buy anything. It was to do, deliberately and in order, the work the platform could only record: (1) define the taxonomy, the control-library attributes and the ownership model, and correct the control entries that were not testable; (2) establish the regulatory inventory as a single owned instance with a refresh cadence, and map the obligations to controls; (3) fix the issue workflow so that no issue may be closed without a root cause and independent validation; (4) build and maintain the assurance map as a standing artefact with a named owner; and only then (5) configure the platform to hold the corrected records and to run the corrected workflows. In short: **process before platform.**

The bank's closing judgement was the thesis of this guide, arrived at from the evidence rather than asserted: the reconciliation had failed because the operations team that was accountable for it had not been able to direct it — the procedure, the shift pattern and the handover were not theirs to change — so the accountability had been a liability assignment rather than a control (§13.3). The three functions had each assured around it without any one of them owning whether it worked. **The business owns the risk; everything else is assurance** — and at Cymbal Bank, the assurance had been impressively abundant and had not substituted for the ownership that was missing.

### 14.7 The worked example table

| Finding | The mechanism | The artefact that revealed it | The fix |
|---|---|---|---|
| **(a) Repeat failure** | Issue closed without root cause or validation; symptom fixed, conditions untouched | The recurrence itself | Root cause required; independent validation before closure |
| **(b) Triplicate evidence** | Three charters, three formats, one business unit, no shared identifiers | The reviewers' cross-function evidence trace | One narrative, assessed once, reported to three audiences; shared identifiers |
| **(c) Uninventoried change** | Impact assessment reached data and reports, not the control; no obligation mapping | Absence of any obligation-to-control link | Single owned regulatory inventory, mapped to controls, with a refresh cadence |
| **(d) Assurance gap** | Coverage clustered where salience was; the unglamorous controls unassured | The assurance map | Maintain the map as a standing artefact with a named owner |
| **(e) Decision** | Platform recorded a programme that did not work | The platform's own four empty or stale fields | Process before platform |

---

## 15. The Anti-Patterns

The eight anti-patterns below are the recurring failure modes this guide has described, restated as a checklist. The **Symptom** column is what you can observe without an investigation; the **Cause** column is the structural mechanism underneath; the **Guardrail** column is the design decision that prevents it.

| Symptom | Cause | Guardrail |
|---|---|---|
| **The second line operates the control** | A struggling first line was relieved of a control "temporarily" so the oversight function now both runs and assures it | The control library owner field must always be a first-line process owner; a second-line owner of a first-line process control is raised as a finding in its own right; the second line advises, monitors and tests, and does not operate |
| **The audit function consulted on the design it will audit** | Audit holds the institutional knowledge, so the business asks, and audit helps; the assistance compromises the later opinion | A standing independence rule: audit may advise on risk and principle but is not the author of the design it will opine on; the request is redirected, and the boundary is enforced by audit itself |
| **Three evidence requests for one control** | Each function's charter, format and cycle generate their own request to the same business unit | A shared identifier set and a single evidence store; the second and third lines consume a joint request where the evidence overlaps; duplication is tracked as a defect, not tolerated as normal |
| **The stale regulatory inventory** | An inventory is a snapshot with no owner and no refresh cadence; maintenance is nobody's job and everybody's input | A single authoritative inventory with a named owner for the artefact, a refresh cadence aligned to horizon scanning, and obligation-to-control mapping as a required attribute |
| **The issue closed without root cause** | The workflow permits closure on attestation; the root-cause field is optional; the symptom fix looks like a fix | No issue closes without a recorded root cause and independent validation; recurrence of a previously closed issue is escalated as a governance event |
| **The KRI set that grew and stopped being read** | Indicators are added, never retired; the pack lengthens past the reader's capacity; the bottom stops being read | Every indicator has an owner, threshold, action and review date; each is periodically challenged with "what decision would this change?"; indicators that never trigger a response are retired |
| **The platform bought before the process** | The tool is procured to fix visible fragmentation; the default data model and undefined workflows populate it with records that describe nothing | Process before platform: define taxonomy, control attributes, ownership model, assessment cycle and issue workflow first; select the platform against the working process, not instead of it |
| **Accountability assigned to someone who cannot direct the control** | The responsibility map assigns areas of responsibility for completeness of coverage, without matching information, authority and reach | The accountable person must be able to direct the control; the responsibility map and the control library owner must be consistent; a mismatch is a finding |

---

## 16. The Claims Audit, then What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary

### 16.1 The claims audit

Every factual claim of consequence made in this guide, with its status and its source. "Practice claim" means a description of how GRC work is done that is not defined by a single authoritative standard; "argument" means a claim of reasoning rather than fact; "illustrative" means it belongs to the Cymbal Bank example.

| Claim | Status | Source / basis |
|---|---|---|
| GRC is the integrated discipline resting on "the business owns the risk; everything else is assurance" | Argument (the guide's thesis) | Reasoning throughout §1–§15; not a quoted standard |
| The three-lines model and its update are owned by the ERM guide and not re-derived here | Cross-reference | [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) §5 |
| The IIA's three-lines update is a Position Paper dated 08 September 2020, renaming "Three Lines of Defense" to the Three Lines Model and adding an explicit governing body | **Verified** | IIA content page for *The IIA's Three Lines Model: An update of the Three Lines of Defense* (Position Paper, September 08, 2020) |
| The original three-lines formulation is the IIA's 2013 position paper *The Three Lines of Defense in Effective Risk Management and Control* | **Verified** (as cited in the ERM guide and the IIA's own referential framing) | IIA; see also the ERM guide's verification record |
| COSO *Internal Control — Integrated Framework* was issued in 1992 and refreshed in 2013 (ICIF-2013) | **Verified** | COSO guidance page (coso.org/guidance-on-ic) |
| COSO ERM comprises the 2004 *Enterprise Risk Management — Integrated Framework* and the 2017 *Enterprise Risk Management — Integrating with Strategy and Performance* | **Verified** | COSO guidance page; ERM guide §2 |
| The COSO thought paper *Leveraging COSO Across the Three Lines of Defense* is authored by Douglas J. Anderson and Gina Eubanks | **Verified** | COSO guidance page (thought papers) |
| ISACA's COBIT is the framework for the governance and management of enterprise IT | **Verified** | ISACA COBIT resource page (isaca.org/resources/cobit) |
| COBIT 2019 is the current edition | **Flagged [verify]** | The ISACA page presents COBIT 2019 throughout and references "this new release", but a definitive "current edition" designation was not extracted; see §16.2 |
| ISO 31000:2018 *Risk management — Guidelines*, Edition 2, published 2018-02, last reviewed and confirmed 2023 | **Verified** (as given in the verified-anchors set; the ISO page is the authoritative reference) | ISO's ISO 31000 page |
| BCBS *Compliance and the compliance function in banks* was published as Guidelines on 29 April 2005 and defines compliance risk and the compliance function's mandate, independence, resources and reporting | **Verified** | BCBS publication page for the 29 April 2005 guidelines |
| BCBS *Corporate governance principles for banks* was published as Guidelines on 08 July 2015 and references the three lines of defence and the risk management roles of business units, risk management teams and internal audit | **Verified** | BCBS publication page for d328 (08 July 2015) |
| The compliance function's mandate comprises identify, advise, monitor, test and report; the advisory-vs-enforcement tension has the consequences described | Practice description (mandate structure); argument (the tension's consequences) | Framed on the BCBS 2005 guidelines' concept of compliance risk; the specific component list is practice-based |
| Regulatory change runs horizon scanning → impact assessment → implementation → attestation → tracking/closure; impact assessment must reach reports and data | Practice description | Not defined by a single standard; consistent with the reporting/data discipline in risk_data_aggregation_guide.md and regtech_guide.md |
| The regulatory inventory is the artefact most likely to be stale, duplicated and unowned; a bank cannot demonstrate compliance with an obligation it has not inventoried | Argument (a risk-of-process finding) | Stated as a risk of the process, not an allegation about any institution |
| Control design and operating effectiveness are independent, yielding four combinations; design-vs-effectiveness is the basis of testing | Practice description / argument | Standard control-assessment vocabulary; no single primary standard defines the four-cell framing |
| Manual, automated and IT-dependent-manual controls have different failure modes and testing approaches | Practice description | Consistent with control-testing practice; no single primary standard |
| RCSA is the first line's self-assessment and is not independent; second-line testing is independent of the first line | Practice description | Consistent with the three-lines model; RCSA is not defined as a formal component by any single standard |
| The assurance map is the artefact that reveals assurance overlap and gaps | Argument / practice artefact | No primary standard defines the "assurance map"; it is a recognised GRC practice artefact |
| Internal audit reports functionally to the board/audit committee and must not design what it audits (self-review threat) | Practice description | Consistent with BCBS 2015's treatment of control functions and with internal-audit independence principles |
| The issue lifecycle is raise → assess → assign → remediate → validate → close | Practice description | Consistent with issue-management practice; no single primary standard |
| Incident, breach and issue are distinct: an incident is an event, a breach is an unmet obligation, an issue is the work that follows either | Argument (definitional) | Definitional clarity, consistent with compliance practice |
| An issue closed without a root cause recurs | Argument (a probabilistic/mechanistic claim) | Stated as mechanism, not as a measured statistic |
| KRI characteristics (actable, owned, threshold, action) and the pruning discipline | Practice description | No primary standard defines KRI design; the guidance is practice-based |
| The GRC platform does five structural things (registers, control library, assessment workflow, issue workflow, reporting) and inherits a taxonomy/ownership data problem | Practice description / argument | Structural description; deliberately vendor-neutral and not a product claim |
| "Process before platform": buying a tool before the process produces an inventory of nothing | Argument | Mechanism argued in §10.3 and illustrated in §14.6 |
| A pack that reports everything reports nothing | Argument | Mechanism argued in §11.4 |
| Individual accountability requires information, authority and reach, else it is a liability assignment | Argument | Structural claim; see ../management/directly_responsible_individual_guide.md for the craft |
| Cymbal Bank and all its artefacts, figures and findings | **Illustrative** | Fictional worked example; no real institution, vendor or consultancy is asserted |

### 16.2 What Could Not Be Verified

The following were **not** verified at primary sources in this research pass, and are flagged rather than asserted. They are listed so a reader can see exactly where the guide's confidence ends.

- **The month of the IIA's three-lines update.** The IIA's own content page shows the position paper as **September 08, 2020**. The sibling [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) dates the update **July 2020**, and some secondary sources cite July. This guide states the **year (2020)** with confidence and **flags the month as uncertain**. Treat "2020" as the safe form.
- **Whether the IIA 2020 position paper remains the current three-lines reference.** The IIA page itself carries a notice that the 2020 document has been **superseded** and directs readers to a newer *Statement of Position on the Three Lines Model*. That successor document's content and date were **not** verified in this pass. This guide uses the 2020 formulation because it is the one the sibling ERM guide and the wider practice literature reference; a reader relying on the model for a current compliance purpose should check the IIA's successor statement directly. (The IIA page also notes that the 2020 paper was itself updated in **September 2024** to reflect the glossary of the new Global Internal Audit Standards — the terminology moves; the model does not.)
- **The COBIT edition designation.** The ISACA page describes COBIT 2019 in detail; this guide does not assert definitively that COBIT 2019 is the *latest* edition, because a definitive current-edition statement was not extracted. See the [verify] flag in §16.1.
- **The formal status of practice artefacts.** RCSA, the assurance map, the control library, the issue lifecycle, the KRI framework and the GRC platform are **practice artefacts**: widely used and described, but not defined as named components by a single authoritative standard in the way that COSO's components or ISO 31000's principles are. The descriptions here are practice-based and deliberately not attributed to a standard that does not contain them.
- **Any statistic, benchmark, threshold or cost figure.** None is asserted anywhere in this guide. No control counts, no assurance-coverage percentages, no cost-of-compliance figures, no "banks spend X" claims, and no KRI thresholds or industry norms. Where a design question needs a number, the guide states that the number is bank-, risk- and metric-specific.
- **Jurisdiction-specific regulatory requirements.** No specific obligation, deadline, threshold or rule is asserted. Where a point depends on a jurisdiction, the guide cross-references [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) and [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md). The individual-accountability regimes' mechanics (§13.1) vary by jurisdiction and are treated structurally only.
- **Vendor capability claims.** None. No vendor product is named, recommended, compared or attributed any capability. §10.5 states this as a deliberate constraint, not an omission.
- **Tool limitation.** `web_search` on this host was unreliable during this pass; verification was performed by `web_extract` against the primary issuing-body URLs (BCBS, IIA, COSO, ISACA, ISO) rather than by search. An empty or poor search result was treated as a tool limitation, not as absence of material.

### 16.3 Glossary

| Term | Definition |
|---|---|
| **Assurance** | Independent evidence and challenge directed at someone else's accountability; in GRC, the product of the second and third lines. Not a substitute for the business's ownership of risk |
| **Assurance map** | The artefact recording, per risk or control, who assures it, by what method, at what depth and at what frequency — and therefore where the overlaps and gaps are |
| **Attestation** | A formal signed declaration by a named accountable person that a defined state is true at a stated date; its value depends on the evidence behind it and the accountability of the signer |
| **Breach** | A failure to meet a defined obligation (regulatory, limit, policy or contractual); distinct from an incident (an event) and an issue (a work item) |
| **CCO** | Chief compliance officer — leader of the compliance function; independence requires a board reporting line, board involvement in removal, and remuneration not driven by business revenue |
| **Compliance function** | The second-line function that identifies, advises on, monitors, tests and reports compliance risk |
| **Compliance risk** | The risk of legal or regulatory sanction, financial loss or reputational damage from a failure to comply with applicable laws, regulations, rules, standards and internal policies (BCBS, 2005) |
| **Control** | A measure that modifies the likelihood or impact of a risk; an action or mechanism, not a document or an intention |
| **Control design** | Whether a control, as specified and built, is capable of addressing the risk it is meant to address in the process where it sits |
| **Control library** | The bank's structured, testable catalogue of its controls, with owners, types, frequencies, evidence and obligation mappings |
| **Control testing** | Structured examination of a control to reach a conclusion on design and/or operating effectiveness, with a defined population or sample and evidence |
| **GRC** | Governance, risk and compliance — the integrated discipline by which accountability is assigned (governance), risk is measured and managed (risk), and conformity with obligations is demonstrated (compliance), as one system |
| **GRC platform** | Software that records and orchestrates GRC activity structurally (registers, control library, workflows, reporting); it records a programme, it does not create one |
| **Incident** | A risk event that has occurred — the fact about the past from which issues and breach determinations may follow |
| **Issue** | A logged deficiency, gap, weakness or improvement requirement with an owner and a remediation commitment; the work item into which incidents and breaches are converted |
| **IT-dependent manual control (ITDM)** | A control in which a person performs a judgement on system-generated data; it has two possible failure points — the person's performance and the data's accuracy |
| **KPI** | Key performance indicator — a metric of how well the business is achieving its objectives; not a risk indicator |
| **KRI** | Key risk indicator — a metric giving an early or current signal of a risk's level or trajectory, with a defined threshold, owner and action |
| **Operating effectiveness** | Whether a control actually operated as designed, consistently, over the period and for the instances tested |
| **RCSA** | Risk and control self-assessment — the first line's own structured assessment of the risks in its processes and the controls addressing them; self-assessed, hence not independent |
| **Regulatory inventory** | The inventory of applicable obligations, mapped to the policies, controls, systems, processes, reports and owners that implement them |
| **Remediation** | The fix applied to an issue; valid only when independently validated, not merely attested by the person who made the fix |
| **Root cause** | The underlying condition that produced a symptom; distinct from the immediate cause, and the basis of a remediation that does not recur |
| **Second line** | The independent oversight functions (risk, compliance, and the data/AI governance functions) that set frameworks, monitor, advise and challenge; they do not own the business's risk |
| **Senior-manager / responsible-person regime** | A regulatory architecture assigning defined responsibilities to named individuals; treated structurally here — the accountable person must be able to direct the control |
| **Third line** | Internal audit — independent assurance to the board on the first and second lines |
| **Three lines** | The division of risk-and-control roles into the functions that own risk (first), oversee it (second) and assure it (third); the model itself is owned by the ERM guide |

### 16.4 Cross-references in this series

Sibling guides in `banking/`:

- [Enterprise Risk Management: The ERM Discipline](enterprise_risk_management_guide.md) — the frameworks, taxonomy, appetite, process **and the three-lines model itself** (§5); this guide names them, that guide owns them.
- [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) — Singapore regulatory content.
- [Basel Regulatory Capital Guide](basel_regulatory_capital_guide.md) — Pillar 1/2, ICAAP, capital.
- [Risk Management Models Guide](risk_management_models_guide.md) — the quantitative models.
- [Risk Data Aggregation Guide](risk_data_aggregation_guide.md) — BCBS 239, the data-side twin of ERM and the floor under the reporting spine.
- [AML Certifications & Exam Content Guide](aml_certifications_exam_content_guide.md) — financial crime and the AML/KYC programme.
- [AI & GenAI Banking Compliance Guide](ai_genai_banking_compliance_guide.md) — the AI-compliance overlay.
- [RegTech Guide](regtech_guide.md) — the regulatory-technology layer.
- [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) — the compliance-systems estate.
- [Operational Resilience Framework Guide](operational_resilience_framework_guide.md) — important business services and resilience.
- [Banks in Singapore Guide](banks_in_singapore_guide.md) — the Singapore banking context.

Cross-directory:

- [Data Governance Guide](../technology/data_governance_guide.md) — data ownership, quality, lineage.
- [AI Governance Framework Guide](../technology/ai_llm/ai_governance_framework_guide.md) — AI governance.
- [Audit as Code Guide](../technology/audit_as_code_guide.md) — internal-audit tooling and analytics.
- [API Governance Guide](../technology/api_governance_guide.md) — API governance.
- [Cybersecurity Guide](../technology/cybersecurity_guide.md) — the security governance overlay and security's use of the three lines.
- [Directly Responsible Individual Guide](../management/directly_responsible_individual_guide.md) — the craft of individual accountability.
- [Vendor Management Guide](../management/vendor_management_guide.md) — third-party risk.

### 16.5 The closing summary

GRC in banking is one discipline with three audiences, not three functions standing side by side. Its organising sentence is the one everything else hangs from: the business owns the risk, and the entire apparatus — the second line's frameworks, monitoring and challenge; the third line's independent assurance; the compliance function's programme; the regulatory inventory; the control library; the assurance map; the issue workflow; the KRI set; the platform; the board pack; the accountability map — exists to assure that the business is owning its risk well, not to relieve it of the duty.

The recurring failure modes all reduce to a single mistake: treating assurance as if it were ownership. The second line that operates the control, the audit function consulted on the design it will later audit, the first line that reports its risk instead of owning it, the three functions collecting one piece of evidence three times — each is a bank that has confused the evidence of control with the control itself. The artefacts that fix this are unglamorous and specific: a single owned regulatory inventory mapped to controls; a control library whose entries can be tested; an assurance map that makes the overlaps and the gaps visible at once; an issue workflow that refuses to close without a root cause and independent validation; an indicator set pruned to the indicators that change decisions; a platform that records a process rather than substituting for one; a board pack built for discrimination; and an accountability map in which every named person can actually direct what they are accountable for.

Where practice varies — the cadence of scanning, the granularity of the inventory, the taxonomy's boundaries, the thresholds and their bases, the structure of the function — this guide has said so, and has asserted no statistic, no threshold and no vendor claim to paper over the variation. The frameworks that anchor the discipline are the IIA's three-lines model (2020, as referenced by the ERM guide), COSO's internal-control and ERM guidance, ISACA's COBIT for the technology-governance dimension, ISO 31000:2018, and the BCBS guidelines of 2005 and 2015 that define compliance risk and place the control functions in bank governance — and the deeper treatment of every one of them lives in the sibling guide that owns it.

The discipline, in the end, is a discipline of ownership: the ownership the business holds, and the assurance everyone else provides to make that ownership accountable. The integrated discipline is not an org chart or a platform; it is the working arrangement by which a bank can say, on evidence and to the board and the supervisor, that the risk it is running is the risk it means to run.

**the business owns the risk; everything else is assurance.**
