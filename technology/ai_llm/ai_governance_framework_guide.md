# AI Governance Frameworks and the Enterprise Operating Model

*A deep-dive on the AI-governance framework landscape — NIST AI RMF, ISO/IEC 42001, the EU AI Act, the OECD AI Principles, Singapore's IMDA frameworks and MAS FEAT — and the operating model that turns them into a working bank program: the three lines of defence adapted to AI, model-risk integration, committee positioning, the lifecycle governance gates, and a regulatory mapping for a bank, ending in a Cymbal Bank rollout.*

Jack Liu Shurui, Solution Architect

> **What this guide is — and where it sits.** This is the repository's umbrella guide for the **framework-and-operating-model layer** of AI governance: choosing and stacking the external frameworks, and building the internal governance machinery (lines of defence, model-risk integration, committees, lifecycle gates) that turns framework commitments into enforced practice. It deliberately does **not** re-derive what its siblings already own:
>
> - **Verified framework facts** (clause-level digests of NIST, the EU AI Act, ISO 42001, the OECD Principles, and Singapore's instruments) live in [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2–§3 — the trust-and-assessment umbrella. This guide gives each framework the *governance-relevant* essentials and the comparison logic, then moves on.
> - **Bias measurement and red-teaming practice** (the technical evidence layers) live in [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) — cross-referenced at the lifecycle gates where their outputs feed decisions.
> - **Organisational how-to** — RAI committee composition, Centre of Excellence build-out, role-by-role playbooks, policy templates, risk-assessment checklists — lives in [./implementing-responsible-ai.md](./implementing-responsible-ai.md). This guide only *positions* those organs on the three lines of defence and shows escalation paths; it does not re-run the playbook.
> - **Banking-domain overlays** — the requirements map ([../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md)), enterprise risk and the three lines ([../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md)), the MAS rulebook ([../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md)), and the SR 11-7 model-risk discipline ([../../banking/risk_management_models_guide.md](../../banking/risk_management_models_guide.md) §9) — are condensed into cross-references here.
>
> **Verification discipline.** Every dated claim was checked against a primary source during this pass (September 2026): nist.gov, iso.org, the European Commission's digital-strategy pages, oecd.ai, aiverifyfoundation.sg / imda.gov.sg, and mas.gov.sg. Where a claim rests on a secondary source or could not be re-confirmed, it is flagged ⚠ inline and again in [§11 The Claims Audit](#11-the-claims-audit) and [§12 What Could Not Be Verified](#12-what-could-not-be-verified). Note that the regulatory timeline below reflects the **AI Omnibus amendments** (in force 27 July 2026) where the European Commission's current pages state them.

---

## Contents

1. [Introduction: The Governance Layer Above the Models](#1-introduction-the-governance-layer-above-the-models)
2. [The Global Instrument Landscape: NIST, ISO, the EU AI Act, OECD](#2-the-global-instrument-landscape-nist-iso-the-eu-ai-act-oecd)
3. [The Singapore Landscape: IMDA Frameworks and MAS FEAT](#3-the-singapore-landscape-imda-frameworks-and-mas-feat)
4. [The Landscape Compared: One Table and a Selection Method](#4-the-landscape-compared-one-table-and-a-selection-method)
5. [The Operating Model: Three Lines of Defence Adapted to AI](#5-the-operating-model-three-lines-of-defence-adapted-to-ai)
6. [Model-Risk Integration and the Committee Lattice](#6-model-risk-integration-and-the-committee-lattice)
7. [The AI Lifecycle: Governance Gates and Artifacts](#7-the-ai-lifecycle-governance-gates-and-artifacts)
8. [Data Governance and Third-Party AI Governance](#8-data-governance-and-third-party-ai-governance)
9. [The Regulatory Mapping for a Bank](#9-the-regulatory-mapping-for-a-bank)
10. [The Worked Example: A Cymbal Bank AI-Governance Rollout](#10-the-worked-example-a-cymbal-bank-ai-governance-rollout)
11. [The Claims Audit](#11-the-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)

---

## 1. Introduction: The Governance Layer Above the Models

### 1.1 Why a Framework-and-Operating-Model Guide Exists

A bank does not govern AI by fine-tuning models or running red-team exercises alone — those are the *evidence layers*, and they only change behaviour when a governance layer turns their outputs into decisions with owners, deadlines, and consequences. That layer has two halves:

1. **The framework landscape** — the external instruments (laws, standards, principles, testing toolkits) a bank must read, map, and answer to. Choosing among them is not a compliance exercise; it is an architectural decision about which instrument becomes the *spine* of internal practice, which ones provide the *evidence format* regulators will recognise, and which ones only need a *crosswalk*.
2. **The operating model** — the internal machinery: who owns AI risk (three lines of defence), how AI systems enter the model-risk estate, which committees sit where and what escalates to them, and the lifecycle gates every AI system must pass with which artifacts.

This guide covers both, then maps the frameworks onto the banking regulatory stack and runs the whole design through a worked Cymbal Bank rollout (§10). It is written to be read *before* the sibling deep-dives: it is the umbrella that their overview sections gesture toward.

### 1.2 The Sibling Map: Who Owns What

| Guide | Owns | How this guide relates |
|---|---|---|
| [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) | Trust dimensions; verified framework digests (NIST/EU/ISO/OECD); Singapore angle; **assessment** methods and process | This guide assumes its verified facts and adds the **governance decision layer**: comparison, selection, operating model, gates |
| [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) | Governance overview §1; bias & fairness deep-dive; red-teaming practice | Cross-referenced at the Measure/validation and monitoring gates (§7) |
| [./implementing-responsible-ai.md](./implementing-responsible-ai.md) | Organisational playbook: RAI committee, CoE, roles, policy templates, risk-assessment how-to, roadmap | This guide **positions** those organs on the three lines (§5–§6) instead of re-deriving them |
| [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) | Banking-domain requirements map (which requirement binds which use case) | Regulatory mapping (§9) condenses and cross-references |
| [../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) | ERM, COSO/ISO 31000, three lines, risk appetite, ICAAP | The three-lines adaptation (§5) is positioned against its §5 treatment |
| [../../banking/risk_management_models_guide.md](../../banking/risk_management_models_guide.md) | Model risk and the SR 11-7 validation discipline (§9) | Model-risk integration (§6) condenses and cross-references |
| [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) | The MAS rulebook (statutes, notices, guidelines incl. FEAT §4.6) | MAS FEAT (§3.3) and the SG regulatory rows (§9) condense and cross-reference |

### 1.3 The Shape of a Governance Program

Every serious AI-governance program — whatever its chosen frameworks — assembles the same seven building blocks. The remainder of this guide maps each block to the frameworks, the operating model, and the gates:

| # | Building block | Where covered here |
|---|---|---|
| 1 | Principles and risk appetite for AI | §1.3, §5 (appetite sits with the board); OECD/principle layer (§2.4) |
| 2 | Framework selection and stacking | §2–§4 |
| 3 | Roles, committees, and escalation | §5–§6 (positioning), [implementing-responsible-ai.md](./implementing-responsible-ai.md) (how-to) |
| 4 | Inventory and use-case risk tiering | §7 (Gate 1–2) |
| 5 | Validation, bias and red-team evidence | §7 (Gate 3), [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) |
| 6 | Deployment approval and monitoring | §7 (Gate 4–5) |
| 7 | Incident response, reporting, and audit | §7 (Gate 6), §5 (third line) |

### 1.4 How to Read This Guide

Different readers need different thirds of this guide. The section map below is the fast route:

| Reader | Start with | Then | Skim |
|---|---|---|---|
| Board / CRO / risk executives | §1, §5, §9.1 | §10 (the rollout) | §2–§4 (landscape detail) |
| Model risk / validation leads | §6.1, §7.3, §7.5 | §2.1 (RMF spine) | §3 (Singapore instruments) |
| Compliance / regulatory officers | §2.2 (AI Act), §3.3 (FEAT), §9 | §7.7 (artifact ledger) | §5–§6 |
| AI product owners / engineers | §7 (the gates) | §5.5 (role positioning) | §9, §11–§12 |
| Data / privacy / vendor officers | §8 | §7.2 (inventory) | §6 |
| Architects new to the cluster | §1.2 (sibling map) → this whole guide | the sibling deep-dives the map points to | §11–§12 for the honesty flags |

One reading note: sections 2–4 (framework landscape), 8 (data/vendor), and parts of 5–6 deliberately cross-reference sibling guides instead of re-deriving them — where a sentence says "full treatment in [guide]", the detail is there, not in a gap this guide should have filled.

### 1.5 The GenAI Governance Delta

Much of this guide applies to AI of any vintage, but generative AI changed the governance problem in five concrete ways — worth naming once, up front, because they are the reason banks needed this operating model rather than just an extended model-risk policy:

1. **From discrete models to continuous systems.** A traditional credit model is versioned and frozen; a GenAI assistant is a foundation model plus prompts, retrieval, and guardrails that change weekly. Inventory entries and validation findings decay faster — hence the change-control and delta-validation rules in §7.4/§7.5 and §8.2.
2. **From prediction to generation.** Models once scored or classified; GenAI produces *content* that can be wrong (hallucination), harmful, or misleading, and each output is novel — hence hallucination/fairness monitoring and answer-domain limits rather than a single accuracy number.
3. **From owned to procured intelligence.** The most capable models are third-party and remote (§8.2): the bank's risk surface now includes a vendor's training data, safety work, and upgrade cadence — hence supplier due-diligence files and contractual flow-down.
4. **From decision support to conversational surface.** Customer trust is now exercised through an always-on natural-language channel — hence FEAT-Transparency disclosure duties, human-handoff design, and conduct risk landing squarely on the assistant's behaviour.
5. **From bounded harm to adversarial exposure.** LLMs are prompt-injectable and jailbreakable in ways classifiers were not — hence red-teaming is a *pre-launch gate* (validation, §7.3), not an optional exercise. The attack mechanics are in [../adversarial_ml_attacks_guide.md](../adversarial_ml_attacks_guide.md); the practice in [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md).

The five deltas explain every design choice in §5–§7: the three lines because systems outlive teams, the gates because content cannot be pre-approved, the committees because these decisions are novel and material, and the monitoring because a GenAI system is never "done."

---

## 2. The Global Instrument Landscape: NIST, ISO, the EU AI Act, OECD

The four global instruments operate at different layers and are complements, not rivals. A compact statement of each layer, then the per-instrument governance essentials; the clause-level verified digests live in [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2 and are not re-derived here.

- **NIST AI RMF 1.0** (2023) — the voluntary *risk-management process* layer: how to identify, measure, and manage AI risk. The operating system most enterprises actually run on.
- **ISO/IEC 42001** (2023) — the *management-system* layer: a certifiable wrapper of policy, assessments, controls, and continual improvement that the RMF process slots into.
- **EU AI Act** (2024) — the *hard-regulation* layer: binding obligations keyed to risk tier, with penalties. Reaches banks with any EU footprint, and GPAI providers everywhere.
- **OECD AI Principles** (2019, updated 2024) — the *principles* layer: the consensus bedrock most later law and frameworks trace their lineage to.

### 2.1 NIST AI RMF 1.0 — the Process Spine

**Verified facts.** The National Institute of Standards and Technology released the AI Risk Management Framework (AI RMF 1.0, NIST AI 100-1) on **26 January 2023**, developed by NIST's Information Technology Laboratory with industry and public-sector input; it is explicitly **voluntary** and designed for organisations that develop, deploy, or use AI (source: [nist.gov/itl/ai-risk-management-framework](https://www.nist.gov/itl/ai-risk-management-framework)). A Generative AI Profile, NIST-AI-600-1, followed on **26 July 2024**. As of 2026 the RMF 1.0 is being revised under the White House AI Action Plan, and NIST released a concept note (7 April 2026) for an AI RMF Profile on Trustworthy AI in Critical Infrastructure.

**The core structure.** The RMF core organises risk-management activity into **four functions — Govern, Map, Measure, Manage** — each subdivided into *categories* and *subcategories* carrying suggested actions. Govern is the cross-cutting function that infuses the other three: culture, policies, processes, roles and responsibilities, and accountability structures. Map establishes context (use cases, affected parties, applicable law, risk tolerance); Measure applies quantitative and qualitative evaluation (metrics, tests, audits); Manage prioritises and responds, then feeds back into Govern — the loop is deliberate. Profiles (current vs. target) and crosswalks to other standards make the RMF the natural *lingua franca* for a bank that must speak to many frameworks at once.

**What it gives a governance program.** A structure for the *process* half of governance — but deliberately no organisational mandate: the RMF does not tell you which committee approves what or how validation must be staffed. That is the operating model's job (§5–§6), which is why this guide treats the RMF as the spine the bank's own machinery hangs on. Where the *measurement content* of Map/Measure comes from, see the bias and red-teaming guide §2–§9.

### 2.2 The EU AI Act — the Binding Layer

**Verified facts.** The AI Act, **Regulation (EU) 2024/1689** laying down harmonised rules on artificial intelligence, entered into force on **1 August 2024** (adopted 2024) and became generally applicable on **2 August 2026**, with staged exceptions (source: [digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai), page updated August 2026). It is the first comprehensive horizontal AI law worldwide and the only instrument in this landscape that is directly binding and carries penalties.

**Four risk tiers.** The Act defines four levels: **unacceptable risk** (practices prohibited outright), **high risk** (the full obligations regime), **limited risk** (transparency duties), and **minimal risk** (no mandatory rules). The prohibited-practices list has grown with the 2026 AI Omnibus amendments: the original eight bans — harmful manipulation and deception, exploitation of vulnerabilities, social scoring, individual criminal-risk assessment, untargeted scraping for facial-recognition databases, emotion recognition in workplaces and education, biometric categorisation of protected characteristics, and real-time remote biometric identification for law enforcement — applied from **2 February 2025**, and a ninth (AI generating non-consensual sexually explicit content or child-sexual-abuse material, e.g. "nudification" apps) applies from December 2026.

**The high-risk regime.** High-risk systems must carry a risk-management system, high-quality datasets, activity logging for traceability, detailed technical documentation, clear information to deployers, **human oversight**, and robustness/cybersecurity/accuracy protections — the obligations most analogous to what a bank already does for models. The banking-relevant high-risk bucket sits in **Annex III** (access to essential services including **creditworthiness assessment**, employment, education, critical infrastructure, and more). **Phased application (Omnibus-amended):** Annex III high-risk obligations apply from **2 December 2027** (extended from the original August 2026/2027 schedule by the AI Omnibus, which entered into force 27 July 2026); high-risk systems embedded in regulated products (Annex I) apply from **2 August 2028**; GPAI-model obligations applied from **2 August 2025**; transparency rules for chatbots and AI-generated content applied from August 2026.

**Governance articles and bodies.** Enforcement is shared: the **AI Office** (the Commission's in-house authority) enforces rules on general-purpose AI models, on AI systems built by the GPAI provider itself, and on very-large online platforms; **national competent authorities** enforce everything else; the European Data Protection Supervisor covers EU institutions. The **AI Board** (member-state representatives), a Scientific Panel, and an Advisory Forum steer and advise. The AI Office holds investigative powers (requests for information, model evaluations, access requests, inspections) and sanctioning powers. **Penalties** (source: [EC enforcement page](https://digital-strategy.ec.europa.eu/en/policies/enforcement-ai-act), updated 24 August 2026): up to **€35 million or 7% of total worldwide annual turnover** (whichever is higher) for prohibited practices; up to **€15 million or 3%** for other breaches including GPAI obligations; up to €7.5 million or 1% for certain information-related failures on AI systems.

**What it means for a bank's governance program.** The Act is the only instrument here where "voluntary" is not an option: it is a *must*, and the artifacts it demands (technical documentation, logging, human-oversight records, post-market monitoring) are exactly the artifacts a NIST- or 42001-style program produces anyway. The practical governance consequence is supply-chain traceability: a bank deploying a GPAI-based assistant inherits deployer obligations it cannot delegate to the model vendor, and if the bank itself has EU entities the extraterritorial reach is direct. The banking-compliance overlay is mapped in [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §2.

### 2.3 ISO/IEC 42001 — the Certifiable Management System

**Verified facts.** **ISO/IEC 42001:2023**, *Information technology — Artificial intelligence — Management system*, is the **world's first AI management-system standard**: Edition 1, published **December 2023** (publication stage 2023-12-18) by ISO/IEC JTC 1/SC 42 (source: [iso.org/standard/81230.html](https://www.iso.org/standard/81230.html)). It specifies requirements for establishing, implementing, maintaining, and continually improving an **AI management system (AIMS)** in organisations that provide or use AI-based products or services, using the Plan-Do-Check-Act methodology — the same management-system architecture as ISO/IEC 27001, which is what makes it **certifiable**: an accredited body can certify an organisation's AIMS ⚠ (certifiability is the standard's design intent as a management-system standard; the ISO store page confirms the MSS structure, and certification practice is documented by accredited certification bodies — see §12).

**Structure and controls.** The standard follows the ISO high-level structure (context, leadership, planning, support, operation, performance evaluation, improvement), so organisations already running ISO 9001/27001 extend rather than rebuild. Its AI-specific machinery: an **AI policy**, an **AI risk assessment**, an **AI system impact assessment** (required before deployment and on significant change — AI's analogue of a privacy impact assessment), lifecycle-oriented operation, and an **Annex A control set** that secondary sources consistently report as **38 controls in nine groups** ⚠ (A.2–A.10), spanning AI policy, roles, impact assessment, the AI lifecycle, data, and supplier relationships, with a Statement of Applicability used to declare which controls apply. Companion standards extend it: ISO/IEC 42005 (AI risk assessment guidance) and ISO/IEC 42006 (requirements for bodies auditing AI management systems) ⚠ (edition dates not re-verified this pass).

**What it gives a governance program.** The *evidence wrapper*: whereas the RMF says *what to do*, 42001 says *how to run it as a managed, audited, continually-improving system* — which is precisely the language internal audit and third-party assurance speak. For a bank, a certified AIMS is the shortest path to demonstrating to supervisors and counterparties that AI governance is a real, documented management discipline rather than a set of point-in-time assessments. The deep verified digest is in [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2.3.

### 2.4 The OECD AI Principles — the Consensus Bedrock

**Verified facts.** The OECD AI Principles — the **first intergovernmental standard on AI** — were adopted as an OECD Council Recommendation in **May 2019** (initially by 42 countries) and **updated in May 2024** to address generative and general-purpose AI developments (source: [oecd.ai/en/ai-principles](https://oecd.ai/en/ai-principles) and the OECD legal instrument OECD-LEGAL-0449). There are now **47 adherents**, and the OECD's definition of an AI system and its lifecycle have been adopted into the EU AI Act and numerous national frameworks. The Principles have two halves: **five values-based principles** (inclusive growth, sustainable development and well-being; human rights and democratic values including fairness and privacy; transparency and explainability; robustness, security and safety; accountability) and **five recommendations to governments** (invest in R&D; foster an inclusive AI-enabling ecosystem; shape an enabling interoperable governance and policy environment; build human capacity and prepare for labour-market transition; international co-operation for trustworthy AI).

**What it gives a governance program.** The *principles floor and the external-communication language*: when a bank writes its AI principles statement or its public responsible-AI page, the OECD framing is the dialect regulators worldwide already speak. It is non-binding and carries no machinery, which is why it anchors §1.3's first building block (principles and appetite) but nothing operational. The OECD.AI incident monitor is a useful external source for populating risk registers (§7, Gate 2) with real failure modes.

### 2.5 What Each Instrument Demands of the Governance Structure

Read as a set of demands rather than a set of documents, the landscape prescribes surprisingly little about *structure* — which is exactly why §5–§6 (the operating model) must supply it. The table below states each instrument's structural demand in one line:

| Instrument | Explicit structural demand | Left to the organisation (supplied by §5–§6) |
|---|---|---|
| NIST AI RMF | Govern function: roles, responsibilities, policies, and accountability structures must exist and be documented; transparency commitments | Which committees, which roles, how validation is staffed |
| ISO/IEC 42001 | Leadership clause: top management must demonstrate commitment, assign AI responsibilities, and review the AIMS; competence requirements | Committee design and the line-of-defence split |
| EU AI Act | Provider/deployer duties, human oversight, and — for high-risk — a conformity-assessment trail with named responsible parties | Enterprise structure; banks map it onto existing risk governance |
| OECD AI Principles | Accountability principle: AI actors should be accountable "for the proper functioning of AI systems" per their roles | Everything operational |
| IMDA MGF (GenAI) | Accountability dimension: incentive structures so players across the lifecycle answer to end-users; incident reporting | Internal mapping of those players |
| MAS FEAT | Accountability: clear internal governance, senior-management ownership, escalation, and model-lifecycle management | The bank's own three-lines and committee map |

The pattern to notice: every instrument demands *accountability machinery* but deliberately stops short of prescribing its shape. A bank that bolts a committee onto the framework without re-drawing the three lines (§5) and the model-risk interface (§6) has satisfied the letter of none of them.

---

## 3. The Singapore Landscape: IMDA Frameworks and MAS FEAT

For a Singapore-headquartered bank, the local instruments are the most immediately relevant external expectations after internal policy. The sequence matters: **principles first (2018–2020), then testing (2022), then generative-AI specifics (2024–2026)**. Two distinct regulators are involved — IMDA/PDPC build the *economy-wide* frameworks, MAS sets the *financial-sector* expectations. The dedicated deep-dives are [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §3 and [../ai_verify_guide.md](../ai_verify_guide.md); this section is the condensed governance reading.

### 3.1 The Model AI Governance Frameworks (Traditional AI, 2019–2020)

**Verified facts.** IMDA and the PDPC released the first edition of the **Model AI Governance Framework** on **23 January 2019** (launched at Davos — the world's first such framework) and a **second edition on 21 January 2020** adding robustness, reproducibility, and usability refinements (source: [aiverifyfoundation.sg/resources/mgf-gen-ai](https://aiverifyfoundation.sg/resources/mgf-gen-ai/)). It is voluntary, organisation-focused, and organised around internal governance structures and risk-management processes for traditional AI — the ancestor of everything Singapore has published since.

### 3.2 A.I. Verify and the Model AI Governance Framework for Generative AI

**A.I. Verify (2022–present).** Singapore launched **A.I. Verify** — billed as the world's first AI governance testing framework and software toolkit — at the Asia Tech x Singapore summit in **May 2022** as a Minimum Viable Product released for international pilot and feedback (IMDA press release, May/June 2022 ✅; announced January 2022). In **June 2023** IMDA open-sourced the toolkit and stood up the **AI Verify Foundation**, a not-for-profit wholly owned subsidiary of IMDA, to steward it; the testing framework assesses an AI system against **11 internationally recognised AI governance principles** organised as *principles → outcomes → processes → evidence*, and the toolkit runs automated technical tests (fairness, explainability, robustness, performance). The dedicated history, component anatomy, and worked assessment live in [../ai_verify_guide.md](../ai_verify_guide.md); the practical point for a governance program is that AI Verify converts "we think our AI is responsible" into a self-assessed, published evidence file — the same *self-assessment plus evidence* pattern the EU AI Act uses for limited-risk systems and ISO 42001 uses for its management system.

**Status note (honest).** The brief for this guide asked whether the 2022 pilot "concluded" and whether a **July 2025 second edition** of the GenAI framework superseded it. What the primary sources show: (a) the original toolkit was open-sourced in June 2023 and the Foundation has continuously evolved it (the framework was updated on **29 May 2025** to cover generative-AI applications, and Project Moonshot provides LLM testing — per [../ai_verify_guide.md](../ai_verify_guide.md) §1); (b) a **"July 2025 second edition" of the Model AI Governance Framework for Generative AI could not be confirmed** at the IMDA/AI Verify Foundation pages reviewed ⚠ — the confirmed GenAI framework milestones are the proposed framework of 16 January 2024 and the final **Model AI Governance Framework for Generative AI released 30 May 2024**; and (c) IMDA's assurance work continued through the **Global AI Assurance Sandbox** (launched 7 July 2025, building on a pilot from February 2025) and, in this timeline, a **new Model AI Governance Framework for Agentic AI announced 22 January 2026** at the World Economic Forum (IMDA/MCI press release ⚠ — press-release level, not yet a framework text reviewed here). Treat the July-2025-edition claim as unverified; treat the 2024 and 2026 milestones as verified.

**The MGF for GenAI (30 May 2024).** The final framework expands the traditional-AI model to generative AI across **nine dimensions** (source: aiverifyfoundation.sg): (1) **accountability**, (2) **data**, (3) **trusted development and deployment**, (4) **incident reporting**, (5) **testing and assurance**, (6) **security**, (7) **content provenance**, (8) **safety and alignment R&D**, and (9) **AI for public good**. For a bank's governance program, dimensions 1, 4, and 5 are the ones with operating-model consequences: accountability maps to the three-lines ownership in §5, incident reporting to the incident gate in §7.6, and testing/assurance to the validation gate in §7.3.

### 3.3 MAS FEAT — Condensed Cross-Reference

**Verified facts.** MAS published the **Principles to Promote Fairness, Ethics, Accountability and Transparency (FEAT) in the Use of Artificial Intelligence and Data Analytics in Singapore's Financial Sector** on **12 November 2018** (source: [mas.gov.sg FEAT publication page](https://www.mas.gov.sg/publications/monographs-or-information-paper/2018/FEAT); paragraph 1.4 was updated 7 February 2019 to reference the PDPC Model AI Governance Framework). FEAT = **Fairness, Ethics, Accountability, Transparency**. It is a *guideline*, not a statute: contravention attracts no civil penalty, but — as the MAS guide in this repository stresses — how well an institution observes MAS guidelines feeds MAS's overall supervisory risk assessment, so "non-binding" is not "optional."

**Governance reading (condensed — full treatment in [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) §4.6 and [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §3.4):** FEAT's four principles are the *criteria* layer for a bank's AI governance: **Fairness** (decisions should not be biased, and should be regularly reviewed for unintended bias), **Ethics** (AIDA use should be consistent with the firm's stated values; customers should be treated fairly), **Accountability** (internal governance — senior management ownership, model lifecycle management, escalation), and **Transparency** (disclosure to customers that AI is used, and explainability of decisions). FEAT's later elaborations — MAS's 2024 information paper on model risk management for AI and the 2025 AI risk-management consultation — are covered through the banking guide's lens in [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) rather than re-derived here. MAS has also published on AI-agent governance (**SAFR — Safeguards for Agentic Finance at Runtime**, an information paper visible on its publications listing ⚠ — date and full text not reviewed in this pass), which matters for any bank moving from assistants toward autonomous agents.

### 3.4 The Evolution of Singapore's Instruments in One View

The Singapore sequence is itself a governance lesson — each instrument answered the gap the previous one left (the same layering logic as §4.2):

| Year | Instrument | Gap it answered | Governance consequence for a bank |
|---|---|---|---|
| 2018 | MAS FEAT | No financial-sector AI criteria | Four criteria to govern and evidence against |
| 2019–2020 | Model AI Governance Framework (traditional AI) | No economy-wide organisational guidance | Internal-governance and risk-management expectations for conventional AI |
| 2022 | A.I. Verify MVP + international pilot | Principles without testing | Self-assessment + published evidence became the assurance pattern |
| 2023 | AI Verify Foundation; toolkit open-sourced | Stewardship and global adoption | Testing tooling available in-house; assurance ecosystem emerging |
| 2024 | MGF for GenAI (final, 30 May) | Traditional-AI framework did not cover generative AI | Nine dimensions (accountability, data, deployment, incident reporting, testing, security, provenance, safety R&D, public good) became the GenAI checklist |
| 2025 | AI Verify framework GenAI update (29 May); Global AI Assurance Sandbox (7 July) | Testing coverage for GenAI; assurance-market building | GenAI applications testable against the 11 principles; third-party testers available |
| 2026 | Model AI Governance Framework for Agentic AI (announced 22 January ⚠) | Generative-AI guidance did not cover autonomous agents | Agentic use cases need their own governance pass before scaling |

For the worked example in §10, this table is the *provenance* of Cymbal Bank's framework charter: the charter's checklist rows trace directly to the 2024 nine dimensions and the FEAT criteria.

---

## 4. The Landscape Compared: One Table and a Selection Method

### 4.1 The Comparison Table

| Instrument (year) | Issuer / status | Scope | Binding vs. voluntary | Lifecycle coverage | Certifiability / auditability | Key artifacts a bank produces |
|---|---|---|---|---|---|---|
| **NIST AI RMF 1.0** (2023; GenAI profile 2024) | NIST (US), voluntary framework | Sociotechnical AI risk for any organisation developing or using AI | **Voluntary**; de-facto internal standard; referenced by US policy and supervisory practice | Full lifecycle: design → development → deployment → monitoring (Govern/Map/Measure/Manage) | Not certifiable; self-assessment with profiles; auditable as *process* evidence | RMF-aligned risk register, current vs. target profiles, measurement results, control/mitigation records |
| **ISO/IEC 42001** (2023) | ISO/IEC JTC 1/SC 42, international standard | AI management system (AIMS) in any organisation providing or using AI | **Voluntary** but certifiable; evidence wrapper for other obligations | Full lifecycle inside a PDCA management system; AI risk + impact assessments on change | **Certifiable** by accredited bodies (third-party AIMS certification); Statement of Applicability | AI policy, AI risk assessment, AI system impact assessment, Annex A controls evidence, audit reports |
| **EU AI Act** — Reg. (EU) 2024/1689 (2024) | European Union, regulation | All AI placed on/market in EU, incl. extraterritorial reach; GPAI models | **Binding law**; penalties up to €35M / 7% worldwide turnover (prohibited practices) | Full lifecycle for high-risk systems (pre-market conformity → post-market monitoring) | Conformity assessment + CE marking for high-risk; EU database registration; AI Office enforcement for GPAI | Technical documentation, logging records, human-oversight evidence, conformity declaration, incident reports, GPAI training-data summaries |
| **OECD AI Principles** (2019, updated 2024) | OECD Council Recommendation, 47 adherents | Values-based principles + government recommendations | **Voluntary**; foundational — EU AI Act and most frameworks trace lineage to it | Principles level, not process level | Not certifiable; no machinery | Principles statement, external-communications language, policy alignment mapping |
| **IMDA Model AI Governance Frameworks** (2019/2020 traditional; 2024 GenAI; 2026 Agentic) | IMDA / PDPC / AI Verify Foundation (Singapore) | Voluntary organisational guidance for AI, then GenAI, then agentic AI | **Voluntary**; supervisory expectation in Singapore practice | Organisation + lifecycle dimensions (9 for GenAI: accountability, data, deployment, incident reporting, testing, security, provenance, safety R&D, public good) | Not certifiable; self-assessment; feeds AI Verify assurance | Framework-aligned governance self-assessment, incident-reporting process evidence |
| **A.I. Verify** (2022; GenAI update 2025) | IMDA → AI Verify Foundation | Testing framework + open-source toolkit vs. 11 principles | **Voluntary**; self-testing with published evidence | Development/evaluation focus (fairness, explainability, robustness, performance); GenAI extension + Moonshot | Not certification; tester accreditation and assurance sandboxes emerging | AI Verify testing report (framework process checks + toolkit technical test results) |
| **MAS FEAT** (2018) | MAS (Singapore financial regulator) | AI and data analytics in financial-sector decision-making | **Voluntary guideline** with supervisory consequences; no civil penalties | Principles for governance, model lifecycle, and customer-facing disclosure | Not certifiable; assessed through supervision | FEAT-aligned governance evidence: fairness reviews, accountability map, disclosure records |

### 4.2 How a Bank Actually Selects and Stacks

Selection is not "pick the best framework" — it is "decide the spine, the wrapper, and the crosswalks," driven by three questions:

1. **Where do we operate, and what is binding there?** Any EU footprint makes the AI Act a *must* (phased obligations per §2.2). Singapore operations make MAS guidelines and IMDA frameworks the supervisory-relevant posture. US operations make the model-risk guidance lineage (SR 11-7 → revised 2026 guidance, §6) the examination lens. The binding layer is not chosen; it is mapped.
2. **What do we want the internal spine to be?** Most banks converge on **NIST AI RMF as the risk-process spine** (sector-neutral, lifecycle-wide, framework-neutral enough to crosswalk to everything else) wrapped in an **ISO/IEC 42001-style AIMS** where certification or third-party assurance is wanted, because a certified AIMS is the shortest path to "show us your governance" requests from supervisors, auditors, and counterparties.
3. **What is the local regulator watching?** For a Singapore bank, FEAT supplies the *criteria* (fairness, ethics, accountability, transparency) and the MAF/GenAI framework supplies the *dimensions* that a supervisory review expects to see evidenced; A.I. Verify supplies the testing vocabulary for the evidence.

The common skeleton underneath every instrument — *context → risk assessment → measurement → controls → monitoring → documented evidence* — is the same. A program that produces NIST-Map-style risk registers, 42001-style impact assessments, and AI-Act-style technical documentation from **one evidence program** is the pattern banks actually deploy; §10 shows it end to end. The deeper "how the frameworks compose" argument is made in [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2.6 and not repeated here.

### 4.3 The Crosswalk Sketch: Speaking Every Dialect From One System

Because the instruments share one skeleton, a single row of internal evidence can be *labelled* in every dialect. The sketch below shows how one internal control maps across the four global instruments and FEAT — the pattern a compliance team uses when building the crosswalk table that lives beside the framework charter:

| Internal control (what the bank actually does) | NIST RMF | ISO/IEC 42001 | EU AI Act (if in scope) | FEAT |
|---|---|---|---|---|
| Board-approved AI risk appetite statement | Govern (governance culture; risk-management integration) | Clause 5 (leadership) + AI policy | High-risk risk-management system obligations | Accountability |
| Use-case risk tiering at intake | Map (context, risk identification) | AI risk assessment | Risk-tier classification | Accountability / Ethics |
| AI system impact assessment before deployment | Map (impact assessment) | AI system impact assessment | Data governance + documentation for high-risk | Ethics / Fairness |
| Independent validation with bias + red-team evidence | Measure (metrics, testing) | Annex A controls (evaluation) | Conformity assessment evidence (high-risk) | Fairness |
| Human-oversight design record | Manage (controls) | Annex A (human oversight of AI) | Human-oversight obligations for high-risk | Accountability |
| Monitoring, drift and fairness telemetry | Measure/Manage (monitoring) | Clause 9 (performance evaluation) | Post-market monitoring (high-risk) | Fairness |
| Incident log with severities and RCA | Manage (incident response) | Annex A (incident management) | Serious-incident reporting | Accountability / Transparency |

The exact numbering differs by edition and instrument version — the sketch is directional, and formal artifacts should cite the specific clause/category of the version in force (the verified digests in [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2 carry that detail). Its purpose is architectural: **one evidence program, labelled five ways** — which is precisely what §10's Cymbal Bank rollout builds.

## 5. The Operating Model: Three Lines of Defence Adapted to AI

### 5.1 Why AI Forces the Question

Traditional model risk already had a home in banks: the model-risk-management discipline (SR 11-7 lineage, §6.1) assigns first-line model owners, second-line independent validation, and third-line audit. **Generative AI breaks the containment that made that arrangement tidy**: models are no longer discrete, versioned artefacts built by a modelling team — they are foundation models embedded in products, wrapped in prompt layers and RAG pipelines, continuously re-prompted by users, and often procured from third parties. The governance answer is not a new discipline bolted on the side; it is an explicit **adaptation of the three lines of defence** so that every AI system has a named owner, an independent challenger, and an auditor — with committees positioned where decisions actually get made. This is industry practice (the IIA's Three Lines Model and bank model-risk governance, cross-referenced condensed in [../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) §5) rather than any single standard's prescription — no framework in §2 mandates this exact shape, but every framework's accountability requirements imply it (FEAT's Accountability principle, the RMF's Govern function, 42001's leadership clause, the AI Act's provider/deployer duties).

### 5.2 The Three Lines, Mapped to AI

| Line | Who | What they own for AI | Typical AI-era titles |
|---|---|---|---|
| **Governing body** (above the lines) | Board and its risk committee | AI risk appetite, overall accountability, oversight of the program, approval of material AI use cases | Board risk committee, board AI sub-committee |
| **First line — risk owners** | Business units, product teams, model/AI developers and operators | Building and running AI within policy: use-case intake, risk self-assessment, data preparation, development, deployment, day-to-day monitoring, incident containment; *owns* the AI system and its outcomes | AI product owner, model owner, use-case sponsor, GenAI application lead, engineering/ML teams |
| **Second line — oversight and challenge** | Group risk (under the CRO), model risk management, compliance, the AI governance office, data office | Frameworks and standards, independent validation and challenge, risk tiering rules, monitoring of aggregate AI risk against appetite, regulatory interpretation, second-line review of incidents | Head of AI governance / AI risk officer, model risk management (validation), compliance AI lead, data governance office |
| **Third line — independent assurance** | Internal audit | Periodic, risk-based assurance that the AI governance framework is designed and operating effectively; audits of specific high-risk AI systems and of the second line itself | Internal audit (AI/technology audit specialists) |

The governing body sets appetite; the first line runs; the second line challenges and sets rules; the third line verifies the whole machine. The discipline-specific definitions of the three lines (COSO, ISO 31000, IIA 2020) are in the ERM guide §5 and are not re-derived here — this table is the AI-specific translation.

### 5.3 Where the Risk Families Touch

AI risk is not one risk family; it lands across several, and the operating model must say who leads where. The enterprise taxonomy in [../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) §4 groups risk into seven families; an AI system can touch most of them:

| Risk family | What AI adds | Primary line owning the *response* |
|---|---|---|
| **Model risk** | The AI system itself: wrong or misused outputs, drift, hallucination | First line (owner) + second line (independent validation) |
| **Operational risk** | Failures, outages, integration defects, vendor failure | First line operations + second line op-risk |
| **Conduct / reputational** | Customer harm, biased treatment, misleading disclosure (FEAT's territory) | First line business + second line compliance/conduct |
| **Legal / regulatory** | AI Act obligations, PDPA/GDPR, employment law | Second line compliance/legal |
| **Cyber / information security** | Prompt injection, data leakage, model theft, supply-chain compromise | First line security + second line cyber risk |
| **Data risk** | Training-data quality, bias in data, lineage gaps | First line data teams + second line data office |
| **Strategic** | Competitive position, innovation choices | First line strategy (with board appetite) |

The mapping point: **model risk is the anchor family** for governance purposes (most AI systems are models or model-like), which is why §6 builds the AI operating model as an extension of model-risk management rather than a parallel universe.

### 5.4 Risk Appetite for AI

An AI risk-appetite statement is the bridge between board-level appetite and use-case-level decisions. Following the ERM guide's appetite structure (§6 there), a bank would express AI appetite across a few measurable dimensions — e.g., *no AI system that makes autonomous credit decisions above a defined materiality without human review*; *customer-facing GenAI must stay within bounded, monitored answer domains*; *zero tolerance for knowingly deploying a system with unresolved high-severity bias findings*; *all AI systems inventoried within N days of first use*. Appetite statements turn the framework principles (§1.3, building block 1) into testable limits the second line monitors and committees enforce.

### 5.5 Role Positioning: Which Existing Bank Role Takes Which AI Accountability

A bank rarely hires its way to AI governance; it re-positions existing roles and adds a thin layer of specialists. The table below positions common bank roles on the three lines and states their AI accountability in one line — the *positioning* companion to the playbook's role-by-role how-to ([./implementing-responsible-ai.md](./implementing-responsible-ai.md) §2), which carries the detailed job descriptions, reporting lines, and RACI templates this table deliberately omits.

| Existing role | Line | AI accountability in one line |
|---|---|---|
| Board / board risk committee | Governing body | Sets AI risk appetite; approves the framework and board-material use cases; receives the quarterly AI-risk dashboard |
| CRO / group risk | 2nd line apex | Owns the AI risk framework; chairs or delegates the AIGC; answers to the board for aggregate AI risk |
| Chief data officer / data office | 2nd line (data risk) | Owns data lineage, quality, and PDPA handling for AI; sits on the AIGC |
| Head of model risk (MRM) | 2nd line | Extends SR 11-7 validation to AI; runs independent validation; secretary of the MRC |
| Chief compliance officer | 2nd line | Owns regulatory interpretation (FEAT, AI Act applicability, fair-dealing) and the applicability notes artifact |
| Head of AI governance / AI risk officer (new, thin) | 2nd line | Owns the inventory, tiering rules, gate process, and the framework's day-to-day operation |
| Chief information security officer | 2nd line (cyber) | Owns prompt-injection/data-leakage controls and the AI supply-chain security posture |
| AI product owner / use-case sponsor | 1st line | Owns one AI system end to end: risk self-assessment, development, deployment, monitoring, incident containment |
| GenAI application lead / ML engineers | 1st line | Build and run the system to policy; produce the model documentation and evaluation evidence |
| AI Centre of Excellence | 1st line enablement | Provides patterns, guardrails, evaluation harnesses, and skills — deliberately *not* a risk owner |
| Internal audit (AI specialists) | 3rd line | Assures the framework design and operation; audits high-risk AI systems on a risk-based plan |
| RAI ethics forum | Advisory (feeds AIGC) | Advises on values, ethics, and conduct dimensions; no enforcement authority of its own (§6.2) |

Two design rules keep this lattice honest: **no AI system without a named first-line owner** (the inventory enforces it at Gate 0), and **no second-line function that both builds standards and validates against them** — the reason independent validation sits in MRM rather than inside the Centre of Excellence, however tempting it is to co-locate them.

---

## 6. Model-Risk Integration and the Committee Lattice

### 6.1 The SR 11-7 Lineage, Extended to AI — Condensed

The anchor discipline is bank model-risk management. The US supervisory guidance **SR 11-7** (Federal Reserve, 2011, with the companion OCC Bulletin 2011-12) defined the expectations that became the global template: models are governed through **development soundness, independent validation (conceptual soundness, outcome analysis, ongoing monitoring), and documented governance** — with board/committee oversight and a model inventory. The deep treatment lives in [../../banking/risk_management_models_guide.md](../../banking/risk_management_models_guide.md) §9 and the banking-AI requirements map in [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §3.4–§4.1 (which notes the US guidance has since been revised — in this timeline superseded by revised guidance issued April 2026 ⚠ repo-internal claim, not re-verified here — see §12).

**How the lineage extends to AI/GenAI** (industry practice, condensed):

| SR 11-7 expectation | Traditional model | AI / GenAI extension |
|---|---|---|
| Inventory | Model inventory of material models | **AI system inventory/register** covering traditional models *and* GenAI applications, RAG pipelines, agentic systems; foundation models tracked as third-party components |
| Development soundness | Documented design, data, methodology | Adds prompt design, retrieval design, guardrails, evaluation data, alignment choices; foundation-model *selection* rationale |
| Independent validation | Conceptual soundness, outcome analysis, ongoing monitoring by independent validators | Adds LLM-specific evaluation (see [./llm_evaluation_vs_validation_guide.md](./llm_evaluation_vs_validation_guide.md)), bias audits and red-teaming ([./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md)), and supplier validation of the underlying model |
| Governance | Policies, roles, committees, change control | Adds AI-specific committee mandates, use-case risk tiering, human-oversight design, incident thresholds, third-party AI oversight (§8) |
| Ongoing monitoring | Performance monitoring vs. baseline | Adds drift, fairness drift, hallucination/refusal rates, prompt-injection attempts, usage and harm telemetry (§7.5) |

The continuity argument matters when talking to a model-risk committee: GenAI systems are not "not models" — they are models with a larger blast radius, less deterministic behaviour, and a supply-chain dependency, governed by the same discipline with AI-specific instrumentation.

### 6.2 The Committee Lattice — Positioning, Not Re-Derivation

The organisational playbook ([./implementing-responsible-ai.md](./implementing-responsible-ai.md) §2) explains how to charter an RAI/ethics committee, staff a Centre of Excellence, and write role descriptions — this section does not repeat that how-to. What this section adds is **where those organs sit on the three lines and what escalates to whom**, because mis-positioning is the most common failure: an AI ethics committee with no line into the risk framework approves things nobody enforces, while a model risk committee without AI vocabulary under-reviews the GenAI estate.

A workable bank lattice, top to bottom:

1. **Board and board risk committee** (governing body). Owns AI risk appetite and the annual AI-risk report; approves the AI governance framework and any use case the appetite statement designates as board-material. Where the board has a technology/innovation committee, AI oversight usually splits: innovation committees sponsor, risk committees govern.
2. **Group risk committee** (second line, under the CRO) — the ERM apex for risk decisions; the AI risk framework is its delegated framework.
3. **AI governance committee (AIGC)** — the *operational apex* for AI decisions, positioned as a second-line-chaired forum with first-line membership: use-case risk-tier approvals above a threshold, framework policy, cross-business standards, material incident review. It is *not* the RAI ethics committee in a new chair — the ethics/RAI committee (playbook §2.1) typically becomes the AIGC's values-and-principles sub-forum or an advisory body, so that ethical review feeds a decision body with enforcement reach.
4. **Model risk committee (MRC)** — the SR 11-7 governance apex for models ([../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) §5; [../../banking/risk_management_models_guide.md](../../banking/risk_management_models_guide.md) §9), extended so its charter covers AI systems in the model estate, approves validation findings and waivers, and signs off high-materiality AI deployments. The AIGC and MRC charters must state who decides what: a clean split is *MRC decides on model soundness and risk acceptability of AI-as-model; AIGC decides on use-case appropriateness, ethics/conduct dimensions, and cross-business consistency* — with joint escalation for anything that is both.
5. **First-line business governance** — business-level AI councils/use-case forums under the first line, plus the **Centre of Excellence / AI enablement function** (playbook §2) which sits as first-line enablement (or shared services) providing tooling, patterns, and skills — not as a risk owner.

**Escalation paths (the short version).** Use-case sponsor → (tiering) → second-line AI risk office → AIGC for material or novel use cases → MRC for model-soundness matters → group risk committee → board risk committee for appetite breaches or board-material use cases. Incidents escalate on a severity ladder (§7.6) that mirrors this: severity-1 AI incidents reach the AIGC chair and the CRO the same day, and the board risk committee at its next meeting.

### 6.3 RACI Sketch for the Key Decisions

| Decision | First line | Second line (AI risk/MRM) | AIGC | MRC | Board risk cttee |
|---|---|---|---|---|---|
| Register a new AI use case | R | A | I | I | — |
| Tier the use-case risk | C | R | A (above threshold) | I | — |
| Approve deployment of a material AI system | C | R (validation sign-off) | A | A | I |
| Approve an AI system with appetite breach | C | R | C | C | **A** |
| Accept a validation finding/waiver | C | R (challenge) | I | A | — |
| Declare a severity-1 incident | R (containment) | A | I (notified) | I | I (notified) |

*(R = responsible, A = accountable/approves, C = consulted, I = informed. This sketch is a starting pattern, not a template — the playbook carries the full how-to.)*

### 6.4 Escalation Scenarios: The Lattice Under Load

The cleanest way to test a committee lattice is to push three realistic scenarios through it — each shows a different escalation shape and a common failure mode to design out.

**Scenario A — a novel high-risk use case arrives (bottom-up escalation).** The trade-finance business proposes an AI that autonomously screens and flags discrepancies in shipping documents. The use-case sponsor registers it (G0) and the risk assessment (G1) lands it in Tier 3 — novel, customer-money-adjacent, autonomous judgment. Because it is above the delegated threshold, it escalates sponsor → second-line AI risk office → **AIGC**, which approves the tier and the validation scope; the MRC confirms the model-soundness treatment (it is a computer-vision-plus-LLM hybrid, squarely in the model estate); the board risk committee is informed at the quarterly dashboard. *Failure mode to design out:* a business that self-tiers to avoid the AIGC — hence tiering rules owned by the second line, not the sponsor.

**Scenario B — a validation disagreement (horizontal escalation).** Independent validation rejects a credit-decisioning model for a fairness finding; the first line disputes the finding's severity. The dispute goes to the **MRC** as the model-soundness apex: MRM presents, the first line challenges, the MRC decides (uphold / condition / waiver with compensating controls). *Failure mode to design out:* the dispute silently dying in email — the charter requires validation findings and waivers to be on a committee agenda, not exchanged bilaterally.

**Scenario C — an appetite breach (top-down escalation).** Monitoring shows a customer-facing assistant exceeding the agreed answer-domain boundary (it began quoting personalised rates — a transaction-adjacent action the appetite statement forbids without human confirmation). This is simultaneously an incident (severity ladder) and an appetite breach: first line contains (disables the capability), second line assesses, and because the appetite statement designates it, the matter goes to the **board risk committee** — which decides whether the appetite line itself needs revising or the system needs re-scoping. *Failure mode to design out:* the incident being closed as a technical bug without the appetite question ever being asked — the incident record must carry an explicit "appetite impact: yes/no" field.

## 7. The AI Lifecycle: Governance Gates and Artifacts

The operating model (§5–§6) decides *who*; the lifecycle gates decide *when and with what evidence*. A governance gate is a decision point where a named role or committee must approve progression, and every gate consumes and produces **artifacts** — because frameworks, regulators, and auditors all ultimately ask for the same thing: *the file*. The gate names below follow industry practice and map cleanly onto the RMF's four functions (Govern at every gate; Map at Gate 1–2; Measure at Gate 3 and 5; Manage at Gate 4–6).

### 7.1 The Gate Model at a Glance

| Gate | Decision | Who approves | Primary artifacts produced | RMF function |
|---|---|---|---|---|
| **G0 — Intake & inventory** | Is this in-scope AI? Enter the register | AI governance office (second line) | Model/AI inventory entry, use-case registration, owner named | Govern |
| **G1 — Use-case risk assessment** | What tier? What controls apply? | Risk tiering by second line; AIGC above threshold | Use-case risk assessment (tier, data classes, jurisdictions, blast radius), FEAT/AI-Act applicability notes | Map |
| **G2 — Development & validation** | Is it fit to seek deployment? | Independent validation (MRM) sign-off | Validation report (incl. bias audit and red-team evidence), model documentation, evaluation results | Measure |
| **G3 — Deployment approval** | May it go live, with what conditions? | AIGC/MRC per §6.3; board if appetite breach | Deployment approval record, human-oversight design, rollout plan, monitoring plan baseline | Manage |
| **G4 — Ongoing monitoring** | Is it still within appetite? | First line runs; second line oversees | Monitoring dashboards, drift/fairness/incident metrics, periodic review reports | Measure/Manage |
| **G5 — Incident & change** | How bad, who decides, what changes? | Severity ladder (§7.6) | Incident record, containment evidence, root-cause analysis, change-control record | Manage |
| **G6 — Retirement** | Is decommissioning clean? | First line + AI governance office | Decommissioning record, data disposal evidence | Govern |

### 7.2 Gate 0–1: The Inventory and the Use-Case Risk Assessment

**The AI system inventory / register** is the single most important artifact in the whole framework — every other control (validation scheduling, monitoring, audit scope, incident response, regulatory reporting) keys off it. It must cover not just "models" but GenAI applications, RAG pipelines, agentic workflows, embedded assistants, and the underlying third-party foundation models (as supplier components, §8). Practically: one row per *AI system* (not per model version), with owner, tier, status, data classes, jurisdictions, materiality, and links to every downstream artifact. Regulators expect it: SR 11-7's inventory discipline, the EU AI Act's registration/database obligations for high-risk systems, and MAS's expectation of a model inventory all converge here.

**The use-case risk assessment** is AI's intake form — deliberately lighter than validation. It answers: what does the system do, who is affected, what could go wrong, which data (including personal data) flows through it, which jurisdictions and therefore which binding instruments apply, and what tier results. Tiering follows the risk-based logic every framework shares (the AI Act's tiers, the RMF's risk-characterisation, FEAT's materiality): a sensible bank scale is **Tier 1 (minimal)** — internal productivity aids with human review; **Tier 2 (standard)** — internal decision support and bounded customer-facing tools; **Tier 3 (high)** — customer-facing decisions or actions with material harm potential (credit decisions, onboarding, trading), autonomous actions, or systems touching vulnerable populations. Tier drives the depth of validation, monitoring, and committee review. The how-to checklists for this assessment live in the playbook ([./implementing-responsible-ai.md](./implementing-responsible-ai.md) §3) and the banking compliance guide §7; this guide's point is that the assessment is the *Map* artifact that makes everything downstream proportionate.

### 7.3 Gate 2: Independent Validation — What the Evidence Must Contain

Independent validation is the second line's core value-add: someone other than the builders challenges whether the system does what is claimed, under the conditions that matter. For traditional models the SR 11-7 triad applies (conceptual soundness, outcome analysis, ongoing-monitoring readiness — [../../banking/risk_management_models_guide.md](../../banking/risk_management_models_guide.md) §9). For GenAI systems the validation file must additionally contain:

- **Bias and fairness evaluation** — metrics against the definitions and taxonomies in the bias guide ([./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) §2–§6);
- **Red-team evidence** — structured adversarial testing results with severity-rated findings ([./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) §7–§9; the attack taxonomy in [../adversarial_ml_attacks_guide.md](../adversarial_ml_attacks_guide.md));
- **LLM-specific evaluation** — task accuracy, hallucination/refusal rates, faithfulness on RAG, safety and jailbreak resistance ([./llm_evaluation_vs_validation_guide.md](./llm_evaluation_vs_validation_guide.md), [./llm_evaluation_frameworks_guide.md](./llm_evaluation_frameworks_guide.md));
- **Foundation-model due diligence** — what the vendor's own evaluations show, model card, and how the application layer compensates for known weaknesses (supplier validation, §8);
- **Human-oversight design** — where a human reviews, with what information and time budget, and how that was tested.

Every high/medium finding needs an owner and a disposition: fix before launch, mitigate with a control, or accept with a documented waiver through the MRC (§6.3). A validation report with no finding dispositions is a receipt, not a gate.

### 7.4 Gate 3: Deployment Approval

Deployment approval is the moment the governance machinery becomes irreversible, so it should be the *narrowest* gate: the approvers verify that every prior artifact exists and is current — inventory entry, tier, validation sign-off with dispositions, human-oversight design, monitoring plan with baselines, incident-response runbook, and named owners — then approve with explicit **conditions** (e.g., launch to 5% of customers first; shadow mode for two weeks; mandatory human review above a transaction threshold) and a **review date**. Conditions are the mechanism that lets a bank launch fast without lowering the bar: they are tracked by the second line and every condition must be closed or explicitly extended at the review date. Board approval is required only where the appetite statement says so (§6.3); the discipline is that the *path* is identical for every system — only the approving body scales with tier.

### 7.5 Gate 4: Ongoing Monitoring — Drift, Fairness, Incidents

Monitoring turns a one-time approval into a continuing licence. The monitoring plan (approved at Gate 3) defines, per system:

- **Performance drift** — the model's task metrics against the Gate-3 baseline (accuracy, relevance, task completion), refreshed on a defined cadence; for GenAI, answer quality, hallucination and refusal rates, and retrieval faithfulness where RAG is used;
- **Fairness drift** — the bias metrics from validation re-measured on live traffic by protected group; fairness drift is the FEAT-Fairness and bias-guide concern moving from pre-launch to post-launch ([./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) §4 has the metric definitions);
- **Usage and harm telemetry** — volumes, escalation/handoff rates to humans, user complaints, prompt-injection/jailbreak attempts, data-exfiltration signals;
- **Incident thresholds** — pre-agreed numbers that trigger action, e.g. a hallucination rate or harm-complaint rate above X for Y hours triggers a severity-2 review; any customer-facing financial harm or regulated-decision error triggers severity-1 (§7.6). Thresholds must be *tiered by the Gate-1 tier*: a Tier-3 credit-adjacent system monitors tighter than a Tier-1 copilot.

Monitoring is first-line-run, second-line-overseen, and the results feed the periodic review (quarterly per system, or per the MRC's schedule) that decides whether the deployment licence continues, is conditioned, or is suspended. The drift-detection methods themselves are covered in [../schema_evolution_data_drift_guide.md](../schema_evolution_data_drift_guide.md) and [../drift_detection_methods_guide.md](../drift_detection_methods_guide.md) — this gate is where their outputs become governance decisions.

### 7.6 Gate 5: Incident Response and Reporting

An AI incident is an event where an AI system causes or risks harm — a wrong regulated decision, a harmful or biased output that reached a customer, a data leak through the model, a prompt-injection compromise, an outage of a customer-facing AI channel. Industry practice (and the shape of the IMDA GenAI framework's incident-reporting dimension, §3.2) is a severity ladder and a runbook:

| Severity | Definition (example) | Response | Reporting |
|---|---|---|---|
| **S1** | Actual or likely material customer harm, financial loss, breach of law, or regulated-decision error | Immediate containment (disable/take down), AIGC chair + CRO same day, root-cause analysis | Board risk committee at next meeting; regulator where required (MAS incident reporting obligations apply — see the MAS guide); AI-Act serious-incident reporting where in scope |
| **S2** | Significant but contained harm or drift beyond threshold | Contain within 24h, second-line review, corrective plan | AIGC at next meeting; log to risk system |
| **S3** | Minor or potential issue, no demonstrated harm | Fix in normal change cycle | Register and trend in the incident log |

Every incident, at every severity, produces the same artifact family: **incident record → containment evidence → root-cause analysis → corrective actions with owners → lessons-learned feed-back into the framework** (a recurring failure mode should change the gate, not just the system). Incident logs are also where the *trend* — the thing supervisors and boards care about — becomes visible: three S2s in one quarter is a framework problem even if no single incident was material. The bias/red-team guide's severity-rating conventions (§8 there) should be used so that pre-launch findings and live incidents speak the same language.

### 7.7 The Artifact Ledger: One Evidence Program, Every Audience

The point of gate discipline is that every framework, supervisor, and auditor ultimately asks for the same file. The ledger below ties each artifact to the gate that produces it, the role that owns it, and the external instrument that recognises it — the practical form of §9.1's "one evidence program" argument.

| Artifact | Produced at | Owned by | Recognised by (external instrument) |
|---|---|---|---|
| AI system inventory / register entry | G0, maintained throughout | AI governance office (2nd line) | SR 11-7 inventory discipline; EU AI Act registration for high-risk; MAS model-inventory expectations |
| Use-case risk assessment (tier, data, jurisdiction, blast radius) | G1 | Use-case sponsor (1st line), tiered by 2nd line | NIST RMF Map; ISO 42001 AI risk assessment; FEAT-Accountability |
| AI system impact assessment | G1 (on change: re-run) | 2nd line AI risk with 1st line input | ISO 42001 (AI's PIA analogue); PDPA-adjacent evidence |
| Applicability notes (which instruments bite) | G1 | Compliance (2nd line) | EU AI Act tiering; FEAT; local law |
| Model documentation / system card | G2 | 1st line builders | SR 11-7 development soundness; EU AI Act technical documentation; AI Act GPAI model cards flowed down |
| Validation report with finding dispositions | G2 | Independent validation (MRM, 2nd line) | SR 11-7 validation; ISO 42001 Annex A; audit evidence |
| Bias audit and red-team evidence | G2 | Validation + specialist teams | NIST AI RMF Measure; A.I. Verify-style testing; bias/red-team guide practice |
| Human-oversight design record | G2–G3 | 1st line with 2nd line challenge | EU AI Act human-oversight requirement; FEAT-Accountability |
| Deployment approval record with conditions | G3 | AIGC / MRC per tier | Internal policy; ISO 42001 change/operation controls |
| Monitoring plan and baselines | G3 | 1st line, 2nd line approves | NIST RMF Manage; SR 11-7 ongoing monitoring |
| Monitoring dashboards and periodic review reports | G4 | 1st line runs; 2nd line oversees | SR 11-7 ongoing monitoring; FEAT-Fairness (fairness drift); AI Act post-market monitoring |
| Incident records, RCA, corrective actions | G5 | 1st line + 2nd line review | IMDA GenAI framework incident dimension; AI Act serious-incident reporting; MAS incident reporting |
| Framework review and audit reports | Annual / audit cycle | 2nd line + internal audit | ISO 42001 internal audit/management review; IIA three-lines assurance |
| Decommissioning record and data-disposal evidence | G6 | 1st line + AI governance office | ISO 42001; data-protection retention law |

One discipline follows from the ledger: **an artifact that exists but has no owner and no stated audience will not survive an audit** — every row needs both, and the second line's periodic review should check the ledger itself (artifacts current? owners still named? dispositions closed?) at least quarterly.

### 7.8 Cadence and Evidence Retention

The gates run on a cadence, and the cadence should be written down so monitoring and review do not silently lapse. A workable default rhythm for a mid-size bank (calibrate to system count and tier mix):

| Activity | Cadence | Responsible |
|---|---|---|
| Inventory reconciliation (new AI systems found in the wild get registered or decommissioned) | Monthly | AI governance office |
| Tier-review of the register (tiers still right? systems changed?) | Quarterly | AI governance office + AIGC |
| Periodic review per high-tier system (deployment licence continues?) | Quarterly | MRM + first-line owner |
| Periodic review per standard-tier system | Quarterly-to-annual by risk | MRM |
| Fairness-drift re-measurement on customer-facing systems | Monthly–quarterly by tier | First line + validation |
| AIGC meeting | Monthly–bimonthly | AIGC chair |
| Board risk committee AI dashboard | Quarterly | CRO / AI governance office |
| Framework health check (gate data: approvals on time? conditions closed? incidents trending?) | Annual, plus after any S1 | Second line, assured by internal audit |

Evidence retention follows the artifact ledger: each artifact row gets a retention period consistent with the longest applicable requirement (records-keeping rules, the AI Act's documentation-retention expectations for high-risk systems, and internal audit's need for a multi-year trail). A practical floor is *the life of the system plus a defined tail* — and retention periods themselves belong in the framework policy, not left to individual teams.

---

## 8. Data Governance and Third-Party AI Governance

Two cross-cutting domains get condensed treatment here because the repository owns their deep content; this section states only the governance-gate consequences.

### 8.1 Data Governance for AI (Condensed)

AI governance fails without data governance: the Gate-2 validation and Gate-4 fairness monitoring are only as good as the data lineage, quality, and rights behind them. The deep treatment of data governance lives in [../data_governance_guide.md](../data_governance_guide.md); privacy and PDPA/GDPR overlays in [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §5; and the playbook's data-governance section in [./implementing-responsible-ai.md](./implementing-responsible-ai.md) §7. What the gates require: **lineage** for training and evaluation data (provenance, consent basis, sensitivity class), **quality** evidence (the EU AI Act's high-risk data-governance requirements and the RMF's Map function both demand it), **no untrained PII leakage** into prompts or model context, and **retention/disposal** controls at retirement. The inventory (§7.2) should link every AI system to its data flows so a data-rights request or a breach notification can be executed in hours, not weeks.

### 8.2 Third-Party and Vendor AI Governance (Condensed)

A bank's GenAI estate is mostly *procured*: foundation models from vendors, assistants built on vendor APIs, and increasingly agents running on third-party platforms. Vendor AI governance is where the frameworks' supply-chain logic lands: the AI Act's provider/deployer split (a bank deploying a GPAI-based assistant keeps deployer obligations it cannot delegate), ISO 42001's supplier-relationship controls, SR 11-7's vendor-validation expectations for models, and MAS's outsourcing expectations. The condensed governance rules:

1. **Every foundation model is a third-party component** — tracked in the AI inventory with its own supplier due-diligence file (model card, vendor evaluations, security posture, data-handling terms, sub-processors, incident history).
2. **Contractual flow-down** — the vendor contract must pass down the obligations the bank cannot delegate: transparency about model versions and changes, evaluation access, breach notification SLAs, data-use restrictions (no training on the bank's prompts), and exit provisions.
3. **Change control on the supply chain** — a vendor model upgrade is a change to the bank's AI system: it re-triggers validation evidence (at least delta-evaluation and bias re-checks) rather than silently shipping.
4. **Outsourcing overlay** — where the vendor arrangement meets MAS's outsourcing definition, the outsourcing-notice obligations apply (see [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) and [../../management/vendor_management_guide.md](../../management/vendor_management_guide.md) §8).

Vendor AI governance is a standing agenda item for the AIGC (§6.2): concentration risk (how many critical systems sit on one foundation model?) is a governance decision, not a procurement one.

### 8.3 The Minimum Vendor Due-Diligence File

To keep supplier governance from becoming an annual slide deck, define the file once and require it per foundation model or AI vendor before any Gate-1 sign-off. A defensible minimum contents list — the procurement-side twin of the §7.7 artifact ledger:

| File section | What it must show | Governance use |
|---|---|---|
| Model card and vendor evaluations | Capabilities, intended use, known limitations; vendor's own safety/accuracy results | Foundation for the delta-evaluation scope (§7.3) |
| Data handling terms | No training on the bank's prompts/data; processing location; sub-processors; retention | PDPA/GDPR and data-governance evidence (§8.1) |
| Security posture | Vendor's security certifications, vulnerability handling, incident history | Cyber-risk overlay; TRMG-adjacent evidence |
| Change and versioning policy | Model version cadence, deprecation windows, notice periods | Change-control trigger (§8.2 rule 3) |
| Evaluation access | Ability to run the bank's bias/eval suites on new versions before adoption | Independent validation's continuing access |
| Incident and breach notification SLAs | Timelines and formats for telling the bank about incidents affecting the service | Severity ladder (§7.6) and MAS incident expectations |
| Concentration and exit provisions | Substitution options, data-return/export, contract exit | AIGC concentration-risk review and business continuity |

A vendor that refuses an item on this list is making a risk decision on the bank's behalf — and the AIGC, not procurement, should be the body that accepts that risk explicitly if the business case demands the vendor anyway.

## 9. The Regulatory Mapping for a Bank

This section maps the framework landscape onto the banking regulatory stack — condensed, because the banking-domain guides carry the depth: [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) is the requirements map (which requirement binds which use case, with what evidence), [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) is the MAS rulebook, and [../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) is the ERM frame. The mapping here is the *translation table* a governance architect uses to answer "which instrument do I answer to, with what, and how does it all connect?"

### 9.1 The Four Layers of the Banking Stack

| Layer | What sits there | Binding force | Evidence the bank must hold |
|---|---|---|---|
| **1. Hard law** | Banking, securities, payments, data-protection and AML statutes; the EU AI Act where there is an EU footprint | Binding; penalties | Statutory records; AI-Act technical documentation and registration for high-risk systems (Annex III from 2 Dec 2027, Omnibus-amended); data-protection impact assessments |
| **2. Supervisory rules and guidelines** | MAS Notices (banking 600-series, e.g. outsourcing Notice 658) and Guidelines (TRMG, BCM, Outsourcing, IAC, **FEAT**); US model-risk guidance (SR 11-7 lineage → revised 2026 guidance ⚠); EBA/OCC equivalents | Guidelines are non-binding *but* feed the supervisor's risk assessment; notices are binding | FEAT-aligned governance evidence (fairness reviews, accountability map, disclosure); model-risk validation files; outsourcing/third-party AI oversight records |
| **3. Supervisory expectations and industry practice** | MAS information papers (2024 AI model-risk paper, 2025 AI risk-management consultation ⚠ repo-internal dating), IMDA Model AI Governance Frameworks, A.I. Verify assurance | Voluntary expectations that shape supervisory review | Framework-aligned self-assessments; AI Verify-style testing reports; GenAI nine-dimension evidence |
| **4. Voluntary international instruments** | NIST AI RMF, ISO/IEC 42001, OECD AI Principles | Voluntary — but they produce exactly the evidence layers 1–3 demand | RMF risk registers and profiles; a certified AIMS (if pursued); OECD-aligned principles statement |

The architectural insight: **a bank cannot keep four separate evidence programs**. One program built on the RMF's process structure, documented in a 42001-style management system, keyed to the risk-tier language the AI Act uses, and evidenced against FEAT's four principles satisfies every layer — which is why §4.2's stacking advice is the practical heart of this guide.

### 9.2 The Use-Case × Instrument Map (Banking)

| Banking use case | Likely tier | Binding instruments that bite | Voluntary instruments that supply evidence |
|---|---|---|---|
| **Credit scoring / loan decisioning** | High (EU: Annex III creditworthiness); Tier 3 in a bank scale | EU AI Act (EU footprint); fair-lending/ECOA law (US); FEAT-Fairness; PDPA/GDPR | NIST RMF (Map/Measure), ISO 42001 impact assessment, SR 11-7 validation file |
| **Customer-facing GenAI assistant** | Standard-to-high (bounded domain); EU: limited-risk transparency duties | AI Act transparency (chatbot disclosure, Aug 2026), FEAT-Transparency, PDPA, AI-Act GPAI obligations flow down from the model provider | A.I. Verify-style testing, bias/red-team evidence (§7.3), MAF-GenAI nine-dimension self-assessment |
| **AML / fraud detection** | High (regulated decisions, false positives affect customers) | AML statutes, MAS notices; model-risk guidance | RMF + SR 11-7 validation, fairness monitoring for adverse-impact rates |
| **Internal copilots (RM support, code, HR)** | Minimal-to-standard | Employment law where HR-adjacent; AI-Act transparency if worker-facing | Tier-1 monitoring, usage telemetry, Code-of-Practice alignment |
| **Trading / market decisions** | High if autonomous | Market-conduct rules, model-risk guidance, MAS tech-risk expectations (TRMG) | Validation, human-oversight evidence, incident thresholds |

### 9.3 The Singapore Reading (Condensed)

For the bank persona in §10, the binding-and-expectation stack is: **MAS notices and guidelines** (the rulebook — [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) §7–§8 maps them onto the bank), with **FEAT** (§4.6 there) as the AI-specific criteria, MAS's AI model-risk information paper and consultation shaping the supervisory direction ([../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §2), and the **IMDA frameworks + A.I. Verify** as the economy-wide instruments the regulator expects a leading bank to evidence against. MAS has also signalled attention to agentic AI (SAFR, §3.3 ⚠). The EU AI Act bites only to the extent the bank operates in the EU or serves EU persons — but GPAI obligations follow the *model*, so even a Singapore-only deployment inherits deployer duties from its EU- or US-based model provider.

### 9.4 The Evidence Inventory: What a Singapore Bank Should Be Able to Produce

Supervisors rarely ask for "your AI governance framework" — they ask pointed questions that presuppose evidence: *show us your inventory; walk us through a high-risk use case; what happened in your last three AI incidents; how do you know the assistant is not biased?* The inventory below is the bank-side answer to §7.7's ledger — the concrete document set, organised by the question it answers. It condenses the FEAT/TRMG-aligned expectations mapped in [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) and the banking compliance guide rather than re-deriving them.

| Supervisory question | Evidence to produce | Owning line |
|---|---|---|
| "What AI do you run?" | AI system inventory/register, current and complete, with owners and tiers | 2nd line (AI governance office) |
| "How do you decide what is allowed?" | AI risk appetite statement (board-approved), framework charter, tiering rules | Governing body + 2nd line |
| "Who is accountable for this system?" | Named owner per system; FEAT-Accountability map; committee mandates | 1st + 2nd line |
| "Why should we trust the model?" | Validation reports, bias audits, red-team evidence, human-oversight design | 2nd line (MRM) |
| "How do you treat customers fairly?" | Fairness reviews and fairness-drift monitoring records; complaint handling | 1st line + 2nd line (FEAT-Fairness) |
| "Are customers told they are talking to AI?" | Disclosure records, assistant transparency page, consent records | 1st line + compliance (FEAT-Transparency, PDPA) |
| "What happens when it goes wrong?" | Incident log with severities, RCAs, corrective actions; board reporting trail | 1st + 2nd line |
| "Who watches the watchers?" | Internal-audit AI workplan and reports; second-line self-assessment | 3rd line |
| "How do you keep current?" | Framework review cycle, horizon-scanning log (§9.5), regulatory-change tracking | 2nd line |

### 9.5 The Horizon: What Changes Between 2026 and 2028

A governance framework is a living thing; the dates below (verified in §2–§3) define the near-term change agenda a bank's horizon-scanning should track:

| When | What lands | Bank action to take |
|---|---|---|
| From 2026 | EU AI Act transparency rules applied (August 2026); AI Office enforcing with member states | Chatbot disclosure and AI-content labelling live in any EU-facing channel |
| 2 December 2027 | EU AI Act high-risk (Annex III) obligations apply — including creditworthiness assessment | EU-footprint credit/AI systems complete conformity documentation; pre-work starts 2026 |
| 2 August 2028 | EU AI Act high-risk rules for Annex I regulated products | Product-safety-adjacent AI components plan conformity |
| Rolling 2026–2027 | US model-risk guidance revision bedding in ⚠; MAS AI risk-management consultation finalising ⚠ | Track and map to the SR 11-7 lineage controls (§6.1); update the evidence inventory |
| Rolling | Singapore GenAI/agentic framework work (2026 Agentic MAF ⚠ press level; SAFR ⚠) | Extend the nine-dimension self-assessment to agentic use cases before they scale |

The horizon-scanning log itself belongs in the second line's quarterly pack to the AIGC — the framework review cycle (§7.7) is the mechanism that turns each of these dates into a checklist item with an owner.

---

## 10. The Worked Example: A Cymbal Bank AI-Governance Rollout

### 10.1 The Persona and the Program

**Cymbal Bank** is the fictional Singapore-headquartered bank used for worked examples across this repository (see the persona conventions in [../../banking/mas_regulations_guidelines_guide.md](../../banking/mas_regulations_guidelines_guide.md) §7). Cymbal Bank is a Singapore full bank with a retail and wealth franchise of roughly two million customers, an SME book, and a growing digital channel. In late 2025 its board approved a GenAI program with two flagship systems, and the group CRO asked for an AI-governance framework to govern them before either reached customers:

- **Cymbal Assist** — a customer-facing GenAI assistant embedded in the mobile app and contact-centre channel: answers product and account questions from a RAG-curated knowledge base, can initiate read-only account lookups, and hands off to human agents for anything transactional. *Tier: high (customer-facing, regulated-products knowledge, PII in conversation).*
- **Cymbal Copilot** — an internal assistant for relationship managers and contact-centre staff: drafts customer responses, summarises call notes, prepares meeting briefs from CRM data. *Tier: standard (internal decision support with human review).*

The rollout below is the framework-selection, operating-model, and lifecycle-gate design from §2–§7 applied to this program. Names, dates, and figures are pedagogical constructions, not any real bank's disclosures.

### 10.2 Phase 1 — Framework Selection (Three Months)

The CRO's office ran the §4.2 selection method:

1. **Binding layer.** Cymbal Bank is Singapore-only (no EU branch), so the EU AI Act is not directly binding on its own systems — but its foundation-model vendor is a US GPAI provider subject to the Act's GPAI obligations, so the bank documents the provider/deployer flow-down (§8.2) and adopts the Act's risk-tier *vocabulary* anyway for interoperability. MAS guidelines (FEAT, TRMG, outsourcing) and notices are the binding-and-supervisory layer.
2. **Spine.** **NIST AI RMF** is adopted as the internal risk-process spine: the four functions become the headings of the bank's AI-risk procedure, and each of the two systems gets current-vs-target profiles.
3. **Wrapper.** The bank decides **not** to pursue ISO/IEC 42001 certification in year one (cost/benefit at its AI maturity), but adopts the AIMS *shape* — AI policy, AI risk assessment, AI system impact assessment, and an Annex-A-style control register — so that certification remains a clean later step if the board wants third-party assurance. The AI system impact assessment doubles as the FEAT-accountability and PDPA-adjacent evidence.
4. **Criteria and evidence.** FEAT supplies the four criteria (fairness, ethics, accountability, transparency); the IMDA MGF-for-GenAI nine dimensions supply the program-level checklist; A.I. Verify-style testing vocabulary is used for the externalised evidence the assistant's transparency page will summarise.

**Outcome:** a one-page framework charter: *"Cymbal Bank governs AI on a NIST AI RMF process spine, documented in an ISO 42001-shaped AI management system, measured against MAS FEAT and the IMDA GenAI framework, with EU AI Act obligations flowed down from model providers."*

The charter in table form — the artefact the CRO presents to the board and, later, to supervisors:

| Charter element | Cymbal Bank decision | Rationale (from §4.2) |
|---|---|---|
| Process spine | NIST AI RMF (Govern/Map/Measure/Manage as the AI-risk procedure's four headings) | Sector-neutral, lifecycle-wide, crosswalks to everything else |
| Management wrapper | ISO 42001 *shape* (AI policy, risk + impact assessments, control register) — certification deferred to year two | AIMS evidence format now; third-party assurance later, cleanly |
| Criteria layer | MAS FEAT four principles | The local supervisor's explicit criteria for AI in finance |
| Program checklist | IMDA MGF-for-GenAI nine dimensions | Supervisory-relevant, GenAI-specific coverage map |
| Testing vocabulary | A.I. Verify-style self-assessment + published evidence | Turns internal evidence into externally readable assurance |
| Regulatory flow-down | EU AI Act GPAI obligations tracked from the model vendor | Deployer duties cannot be delegated (§8.2) |

### 10.3 Phase 2 — Operating Model Build-Out (Three Months)

The §5–§6 design, made concrete:

- **First line:** the Digital Channels division owns Cymbal Assist (a named AI product owner); the Wealth & Consumer Banking division owns Cymbal Copilot (a named use-case sponsor). Engineering/ML teams sit with them. The AI Centre of Excellence (per the playbook, [./implementing-responsible-ai.md](./implementing-responsible-ai.md) §2) is stood up as shared first-line enablement: RAG patterns, guardrail tooling, evaluation harnesses.
- **Second line:** a new **AI Governance Office** inside Group Risk (Head of AI Governance reporting to the CRO) owns the framework, the inventory, and the tiering rules; **Model Risk Management** extends its SR 11-7 validation function to GenAI (with two new validation specialisms: LLM evaluation and bias audit — staff cross-trained via the bias/red-team guide); Compliance owns the FEAT/regulatory interpretation; the Data Office owns lineage and PDPA handling.
- **Committees:** the **AI Governance Committee (AIGC)** is chartered as a second-line-chaired forum (CRO's delegate in the chair; first-line business, engineering, compliance, data, and model risk as members) with delegated authority from the Group Risk Committee for use-case tiering above threshold, framework policy, and material-incident review. The **Model Risk Committee** charter is amended to state it approves AI-system validation findings and waivers. The ethics dimension is handled by folding the bank's RAI ethics forum (playbook §2.1) into the AIGC as its values-and-conduct sub-forum — ethical review now feeds a body with enforcement reach (§6.2). The **Board Risk Committee** receives the AI risk appetite statement and the quarterly AI-risk dashboard, and must approve any appetite breach.
- **Appetite:** the board approves four measurable AI appetite lines (§5.4), including *"no customer-facing GenAI answer initiates a transaction without human confirmation"* and *"zero tolerance for deploying with unresolved high-severity bias or red-team findings."*

**Outcome:** every AI system now has an owner (first line), a challenger (second line), and an auditor (third line, with an internal-audit AI workplan for year two), and every committee knows which AI decisions are theirs.

### 10.4 Phase 3 — Lifecycle Gates for Cymbal Assist (Four Months to Launch)

Walking the §7 gates for the flagship:

- **G0 — Inventory.** Cymbal Assist is registered (row: system ID, owner, vendor model + version, RAG corpus sources, PII classes, tier *high*). Its foundation model appears as a supplier component with its own due-diligence file.
- **G1 — Use-case risk assessment.** Tier **high** is confirmed (customer-facing, PII, regulated-products knowledge). Applicability notes: FEAT-Transparency (disclosure that customers are talking to AI), PDPA (conversation data), AI-Act limited-risk transparency duties as best practice. Controls required: bounded answer domain, human handoff for transactions, guardrails, monitoring plan.
- **G2 — Validation.** Independent validation runs over twelve weeks: a bias audit of answer quality across Singapore's language/dialect mix and demographic groups ([./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md) §4–§6); a red-team exercise targeting prompt injection, jailbreaks, and harmful-financial-advice elicitation (§7–§9 there); RAG faithfulness and hallucination-rate measurement against the curated corpus; and a human-oversight test (agents successfully catching the assistant's errors in handoff). Findings: one high (the assistant can be coaxed into quoting stale interest rates under a crafted persona prompt) and three mediums — all fixed or mitigated before sign-off; the MRC approves the validation report with no open highs.
- **G3 — Deployment approval.** The AIGC approves with conditions: **stage the launch to 5% of app users for four weeks**, mandatory human review on every handoff, an answer-domain blocklist for anything transactional, and a monitoring baseline (hallucination rate, handoff rate, complaint rate, injection-attempt rate) recorded for comparison. The board is informed (not required to approve — no appetite breach).
- **G4 — Monitoring.** Weekly telemetry against baseline; fairness drift re-measured monthly. Week 3 of staging: hallucination rate drifts above the pre-agreed threshold (a corpus refresh introduced stale product terms). This is a severity-2 event: the assistant is switched to read-only mode for answers in that domain, the corpus is corrected, and the incident file is closed with a root-cause note that changes the *gate* — the G2 validation checklist now requires a corpus-refresh delta check (the lessons-learned feedback loop of §7.6).
- **G5 — Incident (illustrative).** Two months after full launch, a customer complaint alleges the assistant gave incorrect advice about switching a retirement account, causing a tax misunderstanding. Review finds the answer was within the knowledge base but the base itself was outdated for one product variant — customer harm: none financial (the customer had not transacted), but a conduct near-miss. Handled as severity-2: disclosure to the customer, product-variant data corrected, the AIGC reviews the trend (a second S2 in the quarter triggers a framework review rather than a system fix), and the incident log feeds the quarterly board-risk dashboard.
- **Cymbal Copilot** follows the same gates at Tier *standard*: lighter validation (delta evaluation plus bias spot-check, no full red team in year one), AIGC approval, and quarterly monitoring — demonstrating that the *path* is identical and only depth scales.

**Outcome:** Cymbal Assist launches governed, monitored, and incident-ready; the evidence file produced along the way (inventory, tier assessment, validation report, approval conditions, monitoring records, incident files) is exactly the artifact set that a FEAT-aligned supervisory review, an ISO 42001-style audit, or an EU-AI-Act-style documentation request would ask for — one evidence program, many audiences (§9.1).

The two systems side by side, showing the identical path at different depths (the §7 principle: *path identical, depth scales with tier*):

| Gate | Cymbal Assist (Tier 3 — high) | Cymbal Copilot (Tier 2 — standard) |
|---|---|---|
| G0 Inventory | Registered, supplier component tracked | Registered |
| G1 Risk assessment | High tier confirmed; FEAT-Transparency + PDPA notes; bounded domain + handoff controls | Standard tier; internal-use notes |
| G2 Validation | 12-week program: bias audit, red team, RAG faithfulness, hallucination rates, human-oversight test; MRC sign-off, no open highs | Delta evaluation + bias spot-check; no full red team in year one |
| G3 Deployment approval | AIGC approval with conditions (5% staged launch, mandatory handoff review, domain blocklist, monitoring baseline) | AIGC approval, lighter conditions |
| G4 Monitoring | Weekly telemetry, monthly fairness drift, pre-agreed thresholds | Quarterly review cycle |
| G5 Incident | Severity ladder active; corpus staleness → S2, read-only containment, gate-level fix | Normal change cycle for drafting-quality issues |

### 10.5 What the Rollout Proves

The Cymbal Bank rollout demonstrates the thesis of this guide: the frameworks (§2–§4) did not dictate a single answer — the bank *chose* a stack. The operating model (§5–§6) turned that stack into named owners, challengers, and approvers. The lifecycle gates (§7) turned the operating model into a repeatable path with artifacts. And the regulatory mapping (§9) guaranteed that the artifacts produced along the path are the ones every layer of the banking stack recognises. Governance is not the layer that slows AI down; it is the layer that lets a bank say yes with evidence.

## 11. The Claims Audit

Status legend: ✅ verified at a primary source during this pass · ⚠ partially verified (secondary source, or primary source confirms only part) · ❌ could not be verified.

| # | Claim | Status | Source note |
|---|---|---|---|
| 1 | NIST AI RMF 1.0 released 26 January 2023; voluntary; Govern/Map/Measure/Manage functions | ✅ | nist.gov/itl/ai-risk-management-framework (page also confirms the July 2024 GenAI Profile NIST-AI-600-1 and the 2026 revision in progress) |
| 2 | ISO/IEC 42001:2023 first edition published December 2023 (2023-12); world's first AI management-system standard; AIMS defined; JTC 1/SC 42 | ✅ | iso.org/standard/81230.html |
| 3 | ISO/IEC 42001 Annex A = 38 controls in nine groups (A.2–A.10) | ⚠ | Consistent across multiple secondary sources; not stated verbatim on the ISO store page extracted |
| 4 | ISO/IEC 42001 is certifiable by accredited bodies | ⚠ | Standard's MSS design (same architecture as ISO/IEC 27001) implies certification; certification practice documented by certification bodies — not re-confirmed on iso.org this pass |
| 5 | EU AI Act = Regulation (EU) 2024/1689; in force 1 August 2024; generally applicable 2 August 2026 | ✅ | digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai (updated Aug 2026) |
| 6 | Four risk tiers: unacceptable / high / limited / minimal | ✅ | Same EC page |
| 7 | Prohibitions 1–8 effective 2 February 2025; ninth prohibition (nudification/CSAM generation) from December 2026 (added by AI Omnibus) | ✅ | Same EC page |
| 8 | High-risk Annex III obligations apply from 2 December 2027; Annex I product rules from 2 August 2028 (Omnibus-amended); GPAI obligations from 2 August 2025; transparency rules August 2026 | ✅ | Same EC page and digital-strategy.ec.europa.eu/en/policies/enforcement-ai-act (updated 24 Aug 2026). Note: sibling [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2.2 shows the pre-Omnibus schedule (Annex III: 2 Aug 2026/2027); the EC's current pages reflect the Omnibus extension — see §12 |
| 9 | AI Office enforces GPAI rules (with national authorities and EDPS); AI Board, Scientific Panel, Advisory Forum advise | ✅ | Same EC pages |
| 10 | Penalties: up to €35M / 7% worldwide turnover (prohibited practices); €15M / 3% (other breaches incl. GPAI); €7.5M / 1% (certain information failures on AI systems) | ✅ | EC enforcement page, updated 24 August 2026 |
| 11 | OECD AI Principles adopted May 2019 (42 countries), updated May 2024; 47 adherents; five values-based principles + five government recommendations | ✅ | oecd.ai/en/ai-principles and OECD-LEGAL-0449 |
| 12 | MAS FEAT published 12 November 2018; FEAT = Fairness, Ethics, Accountability, Transparency; non-binding guideline | ✅ | mas.gov.sg FEAT publication page (para 1.4 updated 7 Feb 2019) |
| 13 | IMDA Model AI Governance Framework 1st edition 23 January 2019; 2nd edition 21 January 2020 | ✅ | aiverifyfoundation.sg/resources/mgf-gen-ai |
| 14 | MGF for GenAI: proposed 16 January 2024; final released 30 May 2024; nine dimensions | ✅ | aiverifyfoundation.sg/resources/mgf-gen-ai |
| 15 | A.I. Verify MVP launched May 2022 (announced January 2022); open-sourced June 2023; AI Verify Foundation steward; 11 governance principles; GenAI framework update 29 May 2025 | ✅ / ⚠ | IMDA press materials (2022) ✅; open-source and Foundation details via sibling [../ai_verify_guide.md](../ai_verify_guide.md) (verified Aug 2026) ⚠-for-this-pass |
| 16 | A "July 2025 second edition" of the Model AI Governance Framework for Generative AI | ❌ | Could not be confirmed at the IMDA / AI Verify Foundation pages reviewed — see §12 |
| 17 | Model AI Governance Framework for Agentic AI announced 22 January 2026 (WEF) | ⚠ | IMDA/MCI press release (via search); framework text not reviewed |
| 18 | Global AI Assurance Sandbox launched 7 July 2025 (pilot from February 2025) | ✅ | OECD.AI policy dashboard entry for Singapore |
| 19 | SR 11-7 lineage: 2011 Federal Reserve guidance + OCC 2011-12; development soundness / independent validation / governance expectations | ✅ | Long-standing supervisory guidance; deep treatment cross-referenced in repo guides ([../../banking/risk_management_models_guide.md](../../banking/risk_management_models_guide.md) §9) — not re-extracted from federalreserve.gov this pass ⚠ |
| 20 | US model-risk guidance "superseded by revised guidance April 2026" | ⚠ | Repo-internal claim in [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md); not re-verified at a primary source |
| 21 | Three lines of defence, committee lattice, lifecycle gates, Cymbal rollout | ⚠ | Industry practice and pedagogical design (this guide's analysis), not external facts; ERM grounding cross-referenced to [../../banking/enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) §5 |
| 22 | SAFR (Safeguards for Agentic Finance at Runtime) MAS information paper exists | ⚠ | Referenced on MAS's FEAT publications page listing; date and text not reviewed |

## 12. What Could Not Be Verified

Honesty flags beat fabrication. Anything not confirmed at a primary source during this pass is listed here or marked ⚠ inline:

- **A "July 2025 second edition" of the Model AI Governance Framework for Generative AI** (§3.2). The brief for this guide named it; the IMDA and AI Verify Foundation pages reviewed confirm the 16 January 2024 proposed framework, the 30 May 2024 final release, and the AI Verify framework's 29 May 2025 GenAI update — but no July 2025 second edition of the MGF for GenAI. The claim may reflect a different instrument or an edition this pass could not locate; treat as unverified.
- **Whether the 2022 A.I. Verify pilot formally "concluded."** Primary materials describe the MVP's launch for international pilot and its June 2023 open-sourcing under the AI Verify Foundation; no explicit "pilot concluded / superseded" statement was found at the pages reviewed. The assurance work demonstrably continued (Global AI Assurance Sandbox, 2025).
- **ISO/IEC 42001 Annex A control count (38 / nine groups) and third-party certifiability** — confirmed only via secondary sources (§11, rows 3–4). The ISO store page confirms the MSS nature and publication facts but not the control inventory.
- **The pre-Omnibus vs. Omnibus-amended EU AI Act timeline discrepancy with a sibling.** [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md) §2.2 lists the original schedule (Annex III high-risk from 2 August 2026; Annex I products 2 August 2027); the European Commission's current pages (updated July–August 2026) state the AI Omnibus extended these to 2 December 2027 and 2 August 2028. This guide follows the Commission's current pages; the sibling's verification log is dated August 2026 and may predate the Omnibus entry into force (27 July 2026). Readers citing formal dates should check EUR-Lex for the consolidated text.
- **The exact EU AI Act adoption date** (13 June 2024 Council adoption) — stated only as "adopted 2024" here; the EC pages confirm the in-force date (1 August 2024) but the adoption-day figure was not re-extracted from EUR-Lex.
- **US model-risk guidance revision (April 2026) and its identifier** — taken from the repo's banking sibling; not re-verified at federalreserve.gov.
- **MAS information-paper dates for AI model risk (2024) and the AI risk-management consultation (2025)** — referenced through the banking sibling rather than verified at mas.gov.sg this pass.
- **SAFR (Safeguards for Agentic Finance at Runtime) publication date and content** — only its existence and one-line description were visible on MAS's publications listing.
- **IMDA press-release dates for the A.I. Verify launch** (May 2022 launch at ATxSG; the press PDF found is dated 1 June 2022) and **the Agentic-AI MAF** (announced 22 January 2026 per press release) — press-level dating only.
- **Cymbal Bank's customer count and franchise details** in §10 are pedagogical and drawn from repository conventions, not from any real bank disclosure.

## 13. Glossary

| Term | Meaning |
|---|---|
| **AIMS** | AI management system — the ISO/IEC 42001 term for an organisation's interrelated elements (policy, objectives, processes) for responsible AI development, provision, or use. |
| **AIGC** | AI governance committee — in this guide, the second-line-chaired operational apex for AI use-case and framework decisions (positioned in §6.2). |
| **AI system inventory / register** | The authoritative list of an organisation's AI systems (models, GenAI apps, RAG pipelines, agents) with owners, tiers, and artifact links; the keystone artifact of lifecycle governance (§7.2). |
| **Annex A controls** | ISO/IEC 42001's reference control set (reported as 38 controls in nine groups ⚠) for operating an AIMS. |
| **Annex III (AI Act)** | The AI Act's list of high-risk use areas, including creditworthiness assessment — the banking-relevant bucket. |
| **FEAT** | Fairness, Ethics, Accountability, Transparency — MAS's 2018 principles for AI and data analytics in Singapore's financial sector (§3.3). |
| **GPAI** | General-purpose AI — models that can perform a wide range of tasks (the AI Act's chapter governs their providers; obligations applied from 2 August 2025). |
| **MRM** | Model risk management — the discipline (SR 11-7 lineage) of governing models through development soundness, independent validation, and ongoing monitoring (§6.1). |
| **MRC** | Model risk committee — the governance apex for model-soundness and validation decisions (§6.2). |
| **NIST AI RMF** | US NIST's voluntary AI Risk Management Framework (2023): four functions — Govern, Map, Measure, Manage — organised into categories and subcategories (§2.1). |
| **RAI / responsible AI** | The practice of designing, developing, and governing AI to be ethical, lawful, fair, transparent, accountable, and safe — the umbrella term the playbook [./implementing-responsible-ai.md](./implementing-responsible-ai.md) develops. |
| **RAG** | Retrieval-augmented generation — grounding an LLM's answers in a curated corpus; a governance-relevant design choice because the corpus becomes part of the risk surface. |
| **Severity ladder** | The S1–S3 incident classification with defined responses and reporting (§7.6). |
| **SR 11-7** | US Federal Reserve supervisory guidance (2011, with OCC 2011-12) defining model-risk-management expectations; the lineage extended to AI in §6.1. |
| **Three lines of defence** | Governance model in which the board/governing body sets appetite, the first line owns risk, the second line oversees and challenges, and internal audit assures (§5). |
| **Tier (use-case risk tier)** | A bank-internal risk classification (this guide: Tier 1 minimal / 2 standard / 3 high) that scales validation, monitoring, and committee review (§7.2). |
| **AI Act / EU AI Act** | Regulation (EU) 2024/1689 — the EU's binding, risk-tiered AI law (§2.2). |
| **AI Office** | The European Commission's AI authority — enforces the GPAI rules, works with national authorities from August 2026 (§2.2). |
| **CoE** | Centre of Excellence — the AI enablement function (tooling, patterns, skills); positioned as first-line enablement, deliberately not a risk owner (§5.5). |
| **Guardrails** | Technical controls on an AI system's inputs and outputs (topic blocks, answer-domain limits, content filters, handoff triggers). |
| **Hallucination** | A model output that is fluent but factually wrong or ungrounded — a core GenAI risk measured at validation and monitoring (§7.3, §7.5). |
| **Human oversight** | The designed point where a person reviews, confirms, or overrides an AI output before it takes effect — a mandated element for high-risk systems in the AI Act and in bank practice (§7.3). |
| **Model card** | A structured disclosure document describing a model's intended use, training data, evaluations, and known limitations (from the vendor for foundation models, produced in-house for bespoke models). |
| **Prompt injection** | An attack that manipulates a model via crafted inputs to bypass its instructions or extract data — a first-order GenAI security risk (see the bias/red-team guide §7–§9 and the attack taxonomy guide). |
| **OECD AI Principles** | The 2019 (updated 2024) OECD Council Recommendation — the first intergovernmental AI standard; five values-based principles and five government recommendations (§2.4). |
| **PDPA** | Singapore's Personal Data Protection Act — the data-protection overlay on any AI handling personal data (cross-ref [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §5). |
| **Statement of Applicability (SoA)** | The ISO management-system document declaring which Annex A controls apply and why — the 42001 analogue of the 27001 SoA. |

---

*Companion reading: the trust-and-assessment umbrella [../ai_trust_assessments_guide.md](../ai_trust_assessments_guide.md); the A.I. Verify deep-dive [../ai_verify_guide.md](../ai_verify_guide.md); the bias and red-teaming guide [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md); the organisational playbook [./implementing-responsible-ai.md](./implementing-responsible-ai.md); and the banking overlays — requirements map, ERM, MAS rulebook, and model-risk discipline — in [../../banking/](../../banking/). A framework chosen, an operating model built, gates that bind every system to evidence, and a bank that can say yes with confidence — that is the shape of the governed intelligence.*
