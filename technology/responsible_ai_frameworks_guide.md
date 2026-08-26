# Responsible AI Frameworks: The Definitive Guide

> **The RAI discipline deep-dive — what responsible AI is and where it came from (the ethical-AI lineage), the corporate frameworks that invented the practice (Microsoft's 2018 principles + Responsible AI Standard, Google's AI Principles, IBM's trust-and-transparency), the international instruments (UNESCO 2021, IEEE Ethically Aligned Design), the open-source tooling that makes RAI measurable (AI Fairness 360, Fairlearn, ART, SHAP/LIME, model cards, datasheets), how enterprises actually implement RAI (governance structures, policies, procurement), the banking angle (MAS FEAT, fair lending), and a Cymbal Bank worked example of an RAI framework design — with a full glossary.**

**Author:** Jack Liu Shurui  
**Role:** Solution Architect, Cymbal Bank  
**Audience:** Solution Architects, AI/ML Platform Teams, Model Risk Management (MRM) Teams, Compliance and Audit, CTO Office  
**Version:** 1.0  
**Last Updated:** August 2026  
**Series:** AI & Data Platform Guides — this is the **dedicated Responsible-AI-frameworks deep-dive**. The regulatory/assurance umbrella (NIST AI RMF, EU AI Act, ISO 42001, OECD, the Singapore angle) lives in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) — its §2 (the frameworks) and §3 (the Singapore angle) are cross-referenced throughout rather than re-derived here. The governance/bias/red-teaming discipline lives in [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md); the SG testing deep-dive in [ai_verify_guide.md](ai_verify_guide.md); the agent-era governance angle in [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md); the enterprise gateway layer in [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md); the banking model-risk context in [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md).

---

## How to Read This Guide

- **Architects and platform teams** — start with [§1](#1-the-rai-overview) (what RAI actually is) and [§4](#4-the-tooling) (the toolchain you can install today); [§7](#7-the-worked-example-a-rai-framework-design) shows the whole thing wired together; pair with [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the platform controls.
- **Model risk / validation teams** — [§2](#2-the-corporate-frameworks) and [§4](#4-the-tooling) give you the vocabulary vendors and validators use; [§6](#6-the-banking-angle) maps RAI to FEAT and fair lending; cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) for the SR 11-7 wrapper.
- **Compliance / governance officers** — [§5](#5-the-implementation) (governance structures, policies, procurement) and the glossary are the core reading; [§3](#3-the-international) covers the global instruments regulators cite.
- **Anyone defending an AI budget in Singapore** — [§8](#8-the-summary) is the one-pager; the assurance-cost framing lives in [../management/business_case_development_guide.md](../management/business_case_development_guide.md).

**Scope note on verification:** dates, principle lists, issuers, and paper details below were checked against primary sources (Microsoft's responsible-AI pages and the Responsible AI Standard document, Google's AI Principles page, IBM's responsible-technology pages, UNESCO's Recommendation text, the IEEE EAD1e document, the GitHub repositories for AIF360/Fairlearn/ART, arXiv records for the Model Cards and Datasheets papers) in August 2026. Where a fact could not be fully verified it is explicitly flagged **[verify]** rather than asserted — the same discipline this guide recommends for RAI programs. The Appendix records the verification log.

**The companion map** (which sibling covers what, so you can stop reading this guide at the right point):

| You need… | Read |
|---|---|
| The regulatory frameworks (NIST AI RMF, EU AI Act, ISO 42001, OECD) and the SG angle | [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2–§3 |
| Bias definitions, fairness metrics, red-teaming practice | [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) |
| Running AI Verify tests / Moonshot for LLMs | [ai_verify_guide.md](ai_verify_guide.md) |
| Platform controls (gateway, guardrails, model access) | [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) |
| Agent-era governance and security | [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) |
| Bank model-risk (SR 11-7) wrapper | [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) |
| The RAI investment case | [../management/business_case_development_guide.md](../management/business_case_development_guide.md) |

---

## Table of Contents

[How to Read This Guide](#how-to-read-this-guide)
1. [The RAI Overview](#1-the-rai-overview)
   - 1.1 What Responsible AI Means (the Definition)
   - 1.2 The Ethical-AI Lineage
   - 1.3 RAI vs. the Regulatory Frameworks (Cross-Reference)
   - 1.4 The Overview Table
   - 1.5 RAI and the Adjacent Disciplines
2. [The Corporate Frameworks](#2-the-corporate-frameworks)
   - 2.1 Microsoft: The Six Principles (2018) and the Responsible AI Standard
   - 2.2 Google: The AI Principles (2018)
   - 2.3 IBM: Trust and Transparency
   - 2.4 The Advisory Layer: Salesforce and Accenture
   - 2.5 The Corporate Table
3. [The International](#3-the-international)
   - 3.1 UNESCO: The Recommendation on the Ethics of AI (2021)
   - 3.2 IEEE: Ethically Aligned Design
   - 3.3 The International Table
4. [The Tooling](#4-the-tooling)
   - 4.1 IBM AI Fairness 360 (AIF360)
   - 4.2 Microsoft Fairlearn
   - 4.3 IBM ART: The Adversarial Robustness Toolbox
   - 4.4 Interpretability: SHAP and LIME (Cross-Reference)
   - 4.5 Model Cards for Model Reporting (Mitchell et al., 2019)
   - 4.6 Datasheets for Datasets (Gebru et al., 2018)
   - 4.7 The Tooling Table
   - 4.8 Choosing and Combining the Tooling
5. [The Implementation](#5-the-implementation)
   - 5.1 The Governance Structures: Ethics Boards and RAI Committees
   - 5.2 RAI Policies and Risk Assessment
   - 5.3 Procurement
   - 5.4 Model-Inventory Integration
   - 5.5 The Implementation Table
   - 5.6 The RAI Maturity Model
6. [The Banking Angle](#6-the-banking-angle)
   - 6.1 MAS FEAT (Cross-Reference)
   - 6.2 Fair-Lending Analytics (Flagged)
   - 6.3 The Banking Table
   - 6.4 The Wider Singapore Banking Context
7. [The Worked Example: A RAI Framework Design](#7-the-worked-example-a-rai-framework-design)
   - 7.1 The Scenario: A Cymbal Bank
   - 7.2 The Principles Design
   - 7.3 The Governance Design
   - 7.4 The Tooling Design
   - 7.5 The Lessons
   - 7.6 The Rollout Plan: Twelve Months
8. [The Summary](#8-the-summary)
   - 8.1 The One-Page Summary
   - 8.2 The Final Word: Ethical by Design

[Glossary](#glossary)
[Appendix: Verification Log (August 2026)](#appendix-verification-log-august-2026)
[Primary Sources and Further Reading](#primary-sources-and-further-reading)

---

## 1. The RAI Overview

### 1.1 What Responsible AI Means (the Definition)

**Responsible AI (RAI)** is the practice of designing, developing, deploying, and operating AI systems so that they are aligned with ethical values and societal norms — fairness, safety, transparency, privacy, accountability, and human dignity — and so that the organization can *demonstrate* that alignment with evidence. It is the engineering-and-governance translation of "AI ethics": ethics supplies the values and the critique; RAI supplies the principles, processes, metrics, tooling, and governance that make those values operational in an enterprise.

Four properties of the definition matter for an architect:

- **RAI is a property of the whole lifecycle, not of the model.** Training data, feature engineering, the model itself, the deployment environment, the human processes around it (oversight, appeals, incident response), and the documentation trail are all in scope. This matches the sociotechnical framing the regulatory frameworks use — NIST AI RMF 1.0 treats AI risk as arising from the *interaction* of the technical system with its social and organizational context ([ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.1).
- **RAI is about demonstrated practice, not intent.** "We care about ethics" is not RAI. RAI programs produce artifacts: a published principle set, a risk-assessment process, bias-audit reports, model cards, datasheets, an incident register, sign-offs. This is the same evidence-currency logic as AI trust assurance ([ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §1.1).
- **RAI is plural and context-dependent.** Fairness has several mutually incompatible mathematical definitions; "transparency" means different things to an engineer (interpretability), a user (disclosure), and a regulator (documentation). A responsible program states *which* definition it is claiming and *why* — the discipline of the trust guide's dimension taxonomy (§1.2 there) applies wholesale here.
- **RAI is a risk-management discipline in corporate clothing.** The corporate frameworks in [§2](#2-the-corporate-frameworks) are, functionally, risk appetite statements plus operating procedures. That is why they slot so cleanly into the NIST/ISO/EU machinery: the trust guide's §2.6 "how the frameworks compose" argument (principles → risk process → management system → regulation) is exactly the stack an RAI program sits on top of.

The term itself gained corporate currency in **2017–2018**: Microsoft began publishing its responsible-AI work in 2017 (the 2017 book *The Future Computed*), Google and IBM published their principle sets in 2018, and "responsible AI" replaced "AI ethics" as the preferred enterprise label precisely because it connotes *operationalizable practice* rather than philosophical debate. (Verified: the 2018 corporate principle sets below; the 2017 Microsoft book is standard history. The label-history is interpretive framing, not a contested fact.)

### 1.2 The Ethical-AI Lineage

RAI is the latest layer of a decades-old conversation. The lineage, in compressed form:

- **The philosophical layer (1950s–1970s).** Machine ethics as a field begins with science-fiction provocation (Asimov's Three Laws, 1942) and early academic framing (Norbert Wiener's 1960 essay on automation and human values; "machine ethics" as a named field in the 2000s, but its questions — what values should machines optimize, who decides — are as old as cybernetics).
- **The institutional layer (1970s–2010s).** Professional codes and research ethics grew alongside computing: the ACM Code of Ethics (first adopted 1992, updated 2018), medical and research ethics boards as the template for later AI ethics committees, and early algorithmic-concern literature (computerized decision-making and due process).
- **The modern-AI trigger (2011–2017).** Deep learning's industrial arrival (ImageNet 2012; AlphaGo 2016) made AI a board-level topic. The field produced landmark statements: the **Asilomar AI Principles (2017)** — 23 principles signed by thousands of researchers; the **ACM Statement on Algorithmic Transparency and Accountability (2017)**; the **IEEE Global Initiative on Ethics of Autonomous and Intelligent Systems (launched 2016, published Ethically Aligned Design 2019 — see [§3.2](#32-ieee-ethically-aligned-design))**; the **Partnership on AI (2016)** — a cross-industry consortium founded by Amazon, Apple, DeepMind/Google, Facebook, IBM, and Microsoft, which remains a convening body for RAI practice.
- **The corporate layer (2018).** 2018 is the inflection year: Microsoft's six principles, Google's AI Principles, and IBM's trust-and-transparency statement all appeared within months of each other — prompted by a specific shock (Google's Project Maven controversy, which forced Google to publish its principles in June 2018) and by the realization that enterprises needed *their own* pre-commitments before regulators wrote theirs (the EU's High-Level Expert Group draft guidelines followed in late 2018; the OECD Principles in 2019).
- **The intergovernmental layer (2019–2021).** The OECD AI Principles (2019), the UNESCO Recommendation (2021, [§3.1](#31-unesco-the-recommendation-on-the-ethics-of-ai-2021)), and the G7 Hiroshima process turn the corporate ideas into international instruments.
- **The regulatory layer (2023–2025).** NIST AI RMF 1.0 (2023), the EU AI Act (2024), ISO/IEC 42001 (2023) — the frameworks the trust guide covers in depth (§2 there). RAI principles are now being *enforced* through law; the corporate principles of 2018 are the ancestors of the compliance obligations of 2026.

**The one-line takeaway:** ethical AI asks "what is right?"; responsible AI asks "how do we build, prove, and govern it?"; regulation asks "what must we do, or pay?" — and a bank's RAI program must answer all three in one integrated system.

**The lineage at a glance:**

| Era | Layer | Milestones |
|---|---|---|
| 1940s–1970s | Philosophical | Asimov's Three Laws (1942); Wiener's automation-and-values essay (1960); early machine-ethics questions |
| 1970s–2010s | Institutional | ACM Code of Ethics (1992, updated 2018); research-ethics boards as the template for AI committees; early algorithmic-fairness literature (credit scoring, due process) |
| 2011–2017 | Modern-AI trigger | Deep learning's industrial arrival (ImageNet 2012, AlphaGo 2016); **Partnership on AI (2016)**; **IEEE Global Initiative (2016)**; **Asilomar AI Principles (2017)**; **ACM algorithmic-transparency statement (2017)** |
| 2018 | Corporate | **Microsoft's six principles; Google's AI Principles (June 2018); IBM's trust-and-transparency (2018)**; Salesforce's Office of Ethical and Humane Use (2018) — the year RAI became an enterprise practice |
| 2019–2021 | Intergovernmental | **OECD AI Principles (2019)**; **UNESCO Recommendation (November 2021)**; G7 Hiroshima process begins (2023) |
| 2023–2025 | Regulatory | **NIST AI RMF 1.0 (2023)**; **ISO/IEC 42001 (2023)**; **EU AI Act (2024)** — the corporate principles become enforceable law |
| 2025– | Agent era | RAI extended to agents, GPAI supply chains, and system-level governance (cross-ref [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md)) |

### 1.3 RAI vs. the Regulatory Frameworks (Cross-Reference)

This guide deliberately does **not** re-derive the regulatory frameworks. The relationships to keep straight:

- **NIST AI RMF 1.0 (Govern–Map–Measure–Manage)** is the *risk-management process*; corporate RAI principles are the *values* the process operationalizes, and the RMF's seven trustworthy-AI characteristics (valid & reliable, safe, secure & resilient, accountable & transparent, explainable & interpretable, privacy-enhanced, fair) are a near-superset of the corporate principle lists in [§2](#2-the-corporate-frameworks). See [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.1.
- **The EU AI Act** turns selected RAI concerns into *hard obligations keyed to risk tier* — e.g., high-risk systems must have bias-examined, representative training data (Article 10), human oversight, and technical documentation; the corporate "transparency" principle becomes a legal duty. See trust guide §2.2.
- **ISO/IEC 42001** is the *certifiable management-system wrapper* into which an RAI program's policies, risk assessments, and impact assessments slot as evidence records. See trust guide §2.3.
- **Singapore's layer** — Model AI Governance Framework (2019/2020), AI Verify (2022), the GenAI framework (2024), and MAS FEAT (2018) — is the *local expression* of the same values; for a Singapore-based bank it is the most immediately actionable external expectation. See trust guide §3 and [ai_verify_guide.md](ai_verify_guide.md), plus [§6](#6-the-banking-angle) here.

The composition argument from trust guide §2.6 applies unchanged: **principles (OECD/corporate) → risk process (NIST) → management system (ISO 42001) → regulation (EU AI Act) → local regulator (MAS)**. This guide adds the layer underneath: the corporate principle sets (where the values language comes from) and the tooling (where the evidence gets produced).

### 1.4 The Overview Table

| Aspect | Description |
|---|---|
| **What it is** | The engineering-and-governance practice of operationalizing AI ethics: principles → processes → metrics → tooling → governance → evidence, across the full lifecycle |
| **Lineage** | Ethical AI (philosophy, Asilomar 2017, ACM 2017, IEEE 2016 initiative) → corporate RAI (Microsoft/Google/IBM 2018) → international (OECD 2019, UNESCO 2021) → regulation (NIST 2023, EU AI Act 2024, ISO 42001 2023) |
| **Corporate canon** | Microsoft's six principles (2018) + Responsible AI Standard; Google's seven AI Principles (2018); IBM's trust-and-transparency (2018, evolved) — see [§2](#2-the-corporate-frameworks) |
| **International canon** | UNESCO Recommendation on the Ethics of AI (193 member states, Nov 2021); IEEE Ethically Aligned Design (2019) — see [§3](#3-the-international) |
| **Tooling** | Fairness: AIF360, Fairlearn. Robustness: ART. Interpretability: SHAP/LIME. Documentation: model cards, datasheets — see [§4](#4-the-tooling) |
| **Implementation** | Ethics boards / RAI committees, RAI policies + risk assessment, procurement clauses, model-inventory integration — see [§5](#5-the-implementation) |
| **Banking expression** | MAS FEAT (2018); fair-lending analytics (ECOA/Reg B, EU AI Act Art. 10); MRM integration — see [§6](#6-the-banking-angle) |
| **Why it matters now** | The 2018 corporate principles are the ancestors of 2026 compliance obligations; RAI evidence produced today is the file regulators will ask for tomorrow (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2–§3) |
| **The one-sentence definition** | Responsible AI is *ethical AI made operational*: a documented, governed, tooled system for building AI that is fair, safe, transparent, private, and accountable — and for proving it |

### 1.5 RAI and the Adjacent Disciplines: Safety, Governance, Trust

RAI is frequently conflated with three adjacent disciplines. The boundaries matter because they determine which team owns what:

- **AI safety vs. RAI.** AI safety is the technical/alignment discipline focused on preventing catastrophic or harmful behavior — robustness, alignment, control, and (in its modern form) the frontier-lab safety agenda. RAI is the broader enterprise practice: safety is one of its principles (Microsoft's "reliability and safety", UNESCO's "safety and security"), but RAI also covers fairness, transparency, privacy, and accountability — concerns safety alone does not. A bank's red-team program ([ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md)) is where the two meet.
- **AI governance vs. RAI.** Governance is the *decision machinery* — committees, policies, roles, escalation, the three lines of defence. RAI is the *values-to-evidence pipeline* — principles, testing, documentation. The relationship: RAI cannot function without governance (someone must decide and sign), and governance without RAI is process without content. This guide's [§5](#5-the-implementation) treats the governance machinery of RAI; the wider enterprise-governance view lives in [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) and [api_governance_guide.md](api_governance_guide.md).
- **AI trust/assurance vs. RAI.** Trust assurance ([ai_trust_assessments_guide.md](ai_trust_assessments_guide.md)) is the *evidence-and-assessment* discipline — it asks "is this specific system trustworthy, and how do we prove it?" RAI is the *framework* discipline — it asks "what does our organization commit to, and what machinery enforces it?" The pipeline: RAI principles and tooling produce the artifacts; trust assessments consume them and add independent judgment; regulators accept the result. In the worked example ([§7](#7-the-worked-example-a-rai-framework-design)) the RAI framework is the factory and the trust assessment is the inspection.

For the agent era, add a fourth boundary: **RAI vs. agent governance.** Agents multiply the governance surface (tool permissions, autonomy levels, human-oversight design, cross-agent side effects) — the agent-specific governance angle is treated in [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) and [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md); RAI principles still apply, but the *controls* must extend to the agent's tool-use graph, not just the model's outputs.

---

## 2. The Corporate Frameworks

2018 was the year the big platform companies stopped writing about AI ethics in the abstract and published *their* operating principles. These documents matter to an enterprise architect for three reasons: (1) they are the templates nearly every later corporate RAI program copied; (2) they name the exact principle lists that vendors, regulators, and auditors will assume you mean when you say "responsible AI"; and (3) they are the public commitments that companies are held to — the Maven-driven Google principles being the canonical example of principles extracted from an organization under pressure.

### 2.1 Microsoft: The Six Principles (2018) and the Responsible AI Standard

**Verified:** Microsoft's six responsible-AI principles are **fairness, reliability and safety, privacy and security, inclusiveness, transparency, and accountability**. They were articulated in 2018 (announced by CEO Satya Nadella in February 2018 in a public essay and subsequently formalized on Microsoft's responsible-AI pages), and they remain the exact six listed on microsoft.com's responsible-AI pages today (verified August 2026: "We've distilled our commitment into six core values: Fairness; Reliability and safety; Privacy and security; Transparency; Accountability; Inclusiveness").

| Principle | Meaning (Microsoft's framing) |
|---|---|
| **Fairness** | AI systems should treat all people fairly — no systematic disadvantage of groups on protected grounds; bias is measured and mitigated |
| **Reliability and safety** | Systems should function reliably and safely under expected and unexpected conditions; failure is anticipated, detected, and mitigated |
| **Privacy and security** | Systems should be private and secure — data protection by design, and resilience against attacks (including adversarial-ML attacks) |
| **Inclusiveness** | AI should empower everyone and engage a diverse range of voices — accessibility and inclusive design are explicit engineering goals |
| **Transparency** | Systems should be understandable — people should know how and why the system works, its capabilities and limitations (model cards, documentation, explainability) |
| **Accountability** | People are accountable for AI systems; humans are in control, governance is documented, and adverse impacts are remediated |

**The Responsible AI Standard (verified).** Microsoft operationalized the principles into an internal engineering standard — the **Responsible AI Standard** — whose **version 2 was published publicly in June 2022**. It converts each principle into *goals* and *requirements* mapped to the system lifecycle (planning, design, development, deployment, operations), with a structure that aligns to the NIST AI RMF functions — Govern, Map, Measure, Manage — and it established an **Office of Responsible AI** (formed 2018) plus a senior **RAI Committee** and a **Responsible AI Review Board** to adjudicate sensitive use cases. For an architect the Standard's practical contribution is the *requirement* pattern: every AI system gets an impact assessment at the planning stage, significant-adverse-impact systems get extra oversight, and the whole thing is version-controlled like any engineering standard. Microsoft also shipped the tooling to match: Azure Machine Learning's Responsible AI dashboard, and the Fairlearn toolkit ([§4.2](#42-microsoft-fairlearn)) that grew out of Microsoft Research.

### 2.2 Google: The AI Principles (2018)

**Verified:** Google published its **AI Principles on 7 June 2018** (CEO Sundar Pichai's post), in direct response to the Project Maven controversy (employee and researcher protests over Google's work on military drone imagery analysis, which Google then declined to renew). The principles are **seven objectives** for AI applications:

1. **Be socially beneficial.** AI should benefit people and society, in domains from healthcare to security to energy; the "overall likely benefits" must "substantially exceed the foreseeable risks and downsides" (verified wording).
2. **Avoid creating or reinforcing unfair bias.** Fairness must be pursued in design, data, and algorithms, including testing for unfair bias.
3. **Be built and tested for safety.** Continuous safety testing, constrained deployment, and careful monitoring of safety and security.
4. **Be accountable to people.** Meaningful opportunity for feedback, explanations, and human agency; internal review processes.
5. **Incorporate privacy design principles.** Privacy defaults, data minimization, transparency about data use.
6. **Uphold high standards of scientific excellence.** Rigorous, reproducible, peer-reviewed research methods.
7. **Be made available for uses that accord with these principles.** Careful consideration of downstream uses, misuse potential, and provenance.

**The "applications we will not pursue" list.** Google paired the objectives with a list of prohibited applications: technologies whose principal purpose is to cause or directly facilitate injury to people (weapons), surveillance that violates internationally accepted norms, and technologies whose purpose contravenes widely accepted principles of international law and human rights. This *negative list* pattern — principles plus prohibitions plus a use-case review process (Google's **Advanced Technology Review Council**) — is the operational heart of the Google approach and is frequently copied in enterprise RAI policies.

### 2.3 IBM: Trust and Transparency

**Verified (current primary source):** IBM's responsible-AI position is expressed today as three **Principles of Trust** on ibm.com (verified August 2026):

1. **The purpose of technology is to augment human capabilities** — technology should enhance and extend human capability and potential, with humans taking on more value-adding tasks (the "AI augments, doesn't replace, humans" doctrine).
2. **Technology should be based on responsible data governance** — responsible collection, sharing, and use of data.
3. **Technology should be open and transparent** — open to multi-stakeholder feedback, with information about development and inner workings available to encourage trust.

These are the evolution of IBM's original **2018 statement** — the **"Principles for Trust and Transparency"** published in September 2018 by then-CEO Ginni Rometty: (1) the purpose of AI is to augment human intelligence; (2) data and insights belong to their creator; (3) new technology must be transparent and explainable. [verify: the September 2018 publication date is widely documented; the current three Principles of Trust above are verified from IBM's live pages.]

Around the principles IBM built the most complete *institutional* apparatus of the three: an **AI Ethics Board** (established 2018), an **AI Ethics Review process** for sensitive deployments, and — critically for this guide — the **open-source tooling stack** ([§4](#4-the-tooling)): AI Fairness 360, the Adversarial Robustness Toolbox, and (now) watsonx.governance, all anchored by an **IBM Responsible Technology Board** with an **Integrated Governance Program** and an **AI Risk Atlas** (verified from IBM's responsible-technology pages). IBM's current framing also adds "Pillars of Trustworthy AI" (transparency, fairness and human value alignment, robustness, privacy) and "Impact Dimensions" (human agency, societal well-being, environmental sustainability) — the clearest published example of a principles → pillars → dimensions → governance hierarchy.

### 2.4 The Advisory Layer: Salesforce and Accenture

Beyond the platform giants, the consulting and SaaS vendors built RAI offerings that banks encounter in procurement:

- **Salesforce** publishes **Responsible AI Principles** on its site (accountability; respect for and promotion of human rights; transparency; empowering humanity and inclusion; fostering trusted and inclusive AI ecosystems) **[verify: exact principle wording — Salesforce's published list has been revised over time; the five-theme shape above is stable across versions]**, and operationalizes them through an internal **Office of Ethical and Humane Use** (founded 2018) and an ethics review process for its Einstein AI features.
- **Accenture**, as the archetypal advisory firm, sells responsible-AI *assurance*: frameworks covering fairness, transparency, explainability, robustness, privacy, and accountability, plus ethics-by-design consulting — a reminder that for most banks the practical encounter with "responsible AI" is a **vendor questionnaire and a statement-of-work clause**, not a philosophy seminar (see [§5.3](#53-procurement)).

The pattern to note: **every credible vendor now has (a) a published principle list, (b) an internal review body, and (c) a tooling/consulting product around it.** When a vendor claims "responsible AI", the audit question is: which of the three layers is real, and which is marketing? The tooling table in [§4.7](#47-the-tooling-table) gives you the concrete checks.

### 2.5 The Corporate Table

| Company | Principles | Notes |
|---|---|---|
| **Microsoft** (2018) | Six: **fairness; reliability and safety; privacy and security; inclusiveness; transparency; accountability** (verified) | Operationalized in the **Responsible AI Standard** (v2 public June 2022; goals + requirements mapped to lifecycle and NIST RMF functions); Office of Responsible AI (2018), RAI Committee, Responsible AI Review Board; Azure ML Responsible AI dashboard; Fairlearn toolkit |
| **Google** (7 June 2018) | Seven objectives: **socially beneficial; avoid unfair bias; built and tested for safety; accountable to people; privacy design principles; scientific excellence; uses accordant with the principles** (verified) | Triggered by the Project Maven controversy; paired with a **negative list** (weapons, surveillance violating international norms) and the Advanced Technology Review Council; later governance expansion (2023–24) for generative AI |
| **IBM** (2018, evolved) | Original three: **augment human intelligence; data and insights belong to their creator; transparent and explainable**; today: three Principles of Trust (augment human capability; responsible data governance; open and transparent) + Pillars of Trustworthy AI (transparency, fairness & human value alignment, robustness, privacy) (verified current form) | AI Ethics Board (2018); Responsible Technology Board + Integrated Governance Program + AI Risk Atlas; open-source tooling (AIF360, ART, watsonx.governance) — the most tooled-up of the three |
| **Salesforce** (2018–2022) | Accountability; human rights; transparency; empowering humanity & inclusion; trusted AI ecosystems **[verify: exact wording]** | Office of Ethical and Humane Use (2018); ethics review for Einstein AI; exemplar of the SaaS-vendor RAI posture |
| *(Advisory)* **Accenture** | Fairness, transparency, explainability, robustness, privacy, accountability (assurance framing) **[verify: exact list varies by offering]** | Sells RAI assurance/consulting; the shape of most enterprise RAI procurement encounters |

> **What the corporate canon converges on:** across all five, the same five values appear — **fairness, safety/reliability, privacy/security, transparency/explainability, accountability** — with inclusiveness (Microsoft), scientific excellence (Google), and data agency (IBM) as differentiators. Any enterprise RAI principle set that covers these five is defensible against every vendor and regulator in this guide.

---
## 3. The International

Where the corporate frameworks answer "what does *our* company commit to?", the international instruments answer "what does *humanity* agree on?" — and they become the language regulators use. Two instruments matter most: the UNESCO Recommendation (the only one adopted by a near-universal body) and IEEE Ethically Aligned Design (the engineering profession's own statement, and the one that most directly feeds technical standards).

### 3.1 UNESCO: The Recommendation on the Ethics of AI (2021)

**Verified:** The **Recommendation on the Ethics of Artificial Intelligence** is the **first global standard-setting instrument on AI ethics**. It was **adopted by acclamation by UNESCO's 193 Member States on 23 November 2021** at the 41st session of the General Conference in Paris (9–24 November 2021) — making it the closest thing the world has to a universal, negotiated consensus on AI ethics. It is a *recommendation* (non-binding international instrument), but its adoption process — two years of drafting, the broadest consultation in UNESCO's history per the organization's own account — gives it unique legitimacy, and its four values and ten principles are the reference list that national strategies and corporate frameworks increasingly cite.

**The four core values** (the foundation of the Recommendation):

1. **Human rights and human dignity** — AI must respect, protect, and promote human rights and fundamental freedoms and human dignity.
2. **Living in harmony with the environment** — AI should benefit the flourishing of ecosystems and environmental sustainability.
3. **Ensuring diversity and inclusiveness** — AI must be developed and used in ways that respect and promote diversity and inclusiveness, including gender equality.
4. **Living in peaceful, just, and interconnected societies** — AI should contribute to peace, justice, and interconnectedness rather than to social fragmentation.

**The ten principles** (the operational layer, each elaborated in the Recommendation with policy actions):

| # | Principle | # | Principle |
|---|---|---|---|
| 1 | Proportionality and do no harm | 6 | Human oversight and determination |
| 2 | Safety and security | 7 | Transparency and explainability |
| 3 | Fairness and non-discrimination | 8 | Responsibility and accountability |
| 4 | Sustainability | 9 | Awareness and literacy |
| 5 | Right to privacy and data protection | 10 | Multi-stakeholder and adaptive governance and collaboration |

**Why it matters to a bank:** the ten-principle list is the *common denominator* of global AI ethics — compare it with the corporate table in [§2.5](#25-the-corporate-table) and every principle maps to something already in Microsoft/Google/IBM's lists plus two the corporates under-weight (sustainability; awareness and literacy). UNESCO also pairs the Recommendation with **Readiness Assessment Methodology (RAM)** and an implementation toolkit, and it explicitly addresses the **gender-equality and environmental** dimensions that corporate frameworks treat thinly. When a regulator, client, or vendor cites "international AI ethics", this is usually the document they mean.

**The policy-action areas.** The Recommendation's second half translates the principles into **eleven areas of policy action** that states are urged to implement: (1) ethical governance and stewardship (including impact assessment, auditing, and redress mechanisms); (2) data policy and data governance (data protection, data quality, data lifecycles); (3) development and international cooperation; (4) environment and ecosystems (AI's environmental footprint); (5) gender (explicit gender-equality measures, closing the gender gap in AI); (6) culture (AI's impact on cultural diversity and heritage); (7) education and research (AI literacy for all); (8) communication and information (information integrity, addressing AI-enabled disinformation); (9) economy and labour (workplace impacts, human-centred work); (10) health and social well-being; (11) specific applications and sectors (proportional regulation by risk). For a bank, the areas that land locally are data governance, education/literacy, and economy-and-labour — the same three the MAS/PDPC layer and the EU Act emphasize. [verify: the eleven-area list is stable in the Recommendation's text; area labels here are condensed summaries.]

### 3.2 IEEE: Ethically Aligned Design

**Verified:** **Ethically Aligned Design (EAD)** — full title *Ethically Aligned Design: A Vision for Prioritizing Human Well-being with Autonomous and Intelligent Systems* — is the flagship output of the **IEEE Global Initiative on Ethics of Autonomous and Intelligent Systems** (launched 2016; a community that grew to 2,100+ participants from industry, government, academia, and civil society). The **First Edition (EAD1e) was published on 25 March 2019** by the IEEE Standards Association (CC BY-NC 4.0), following a Version 1 (2016) and Version 2 (2017) request-for-input process. EAD is a ~300-page consensus document covering well-being, affective computing, data agency, sustainable development, embedding values into systems, policy, and law — with the "From Principles to Practice" chapter mapping high-level principles onto the book's content.

**The eight General Principles (verified verbatim from EAD1e):**

1. **Human Rights** — A/IS shall be created and operated to respect, promote, and protect internationally recognized human rights.
2. **Well-being** — A/IS creators shall adopt increased human well-being as a primary success criterion for development.
3. **Data Agency** — A/IS creators shall empower individuals with the ability to access and securely share their data, to maintain people's capacity to have control over their identity.
4. **Effectiveness** — A/IS creators and operators shall provide evidence of the effectiveness and fitness for purpose of A/IS.
5. **Transparency** — The basis of a particular A/IS decision should always be discoverable.
6. **Accountability** — A/IS shall be created and operated to provide an unambiguous rationale for all decisions made.
7. **Awareness of Misuse** — A/IS creators shall guard against all potential misuses and risks of A/IS in operation.
8. **Competence** — A/IS creators shall specify and operators shall adhere to the knowledge and skill required for safe and effective operation.

**The P7000 standards series.** EAD's distinctive legacy is that it feeds directly into **IEEE standards**: the P7000 family of approved standardization projects (referenced in EAD1e) — P7001 (transparency of autonomous systems), P7002 (data privacy process), P7003 (algorithmic bias considerations), P7009 (fail-safe design of autonomous systems), P7010 (well-being metrics for ethical AI), and others. This is the *engineering-codification* path: principles become standards, standards become contract language, contract language becomes procurement requirements — the exact chain a bank's vendor-management team should understand ([§5.3](#53-procurement)). For an architect, EAD is also notable for insisting on **well-being as a primary success criterion** — a reminder that RAI metrics need not all be statistical; some are human-outcome metrics.

### 3.3 The International Table

| Instrument | Issuer | Year | Status | Core content | Why it matters |
|---|---|---|---|---|---|
| **Recommendation on the Ethics of AI** | UNESCO | Adopted **23 November 2021** (41st General Conference, Paris) | Non-binding recommendation; **adopted by acclamation by 193 Member States** — the first global AI-ethics standard | 4 values (human rights & dignity; harmony with the environment; diversity & inclusiveness; peaceful just societies) + **10 principles** (proportionality/do-no-harm, safety & security, fairness & non-discrimination, sustainability, privacy & data protection, human oversight & determination, transparency & explainability, responsibility & accountability, awareness & literacy, multi-stakeholder/adaptive governance) | The universal consensus language; the list regulators cite; RAM readiness assessment and implementation toolkit |
| **Ethically Aligned Design (EAD1e)** | IEEE (Global Initiative on Ethics of A/IS) | First Edition **25 March 2019** (initiative launched 2016) | Consensus guidance (CC BY-NC 4.0) from 2,100+ contributors | 8 General Principles (human rights, well-being, data agency, effectiveness, transparency, accountability, awareness of misuse, competence); well-being-centric; policy/law chapters; **feeds the IEEE P7000 standards series** (P7001 transparency, P7002 privacy, P7003 bias, P7009 fail-safe, P7010 well-being) | The engineering profession's own canon; principles → standards → procurement chain; well-being as a success criterion |
| *(Lineage)* **OECD AI Principles** | OECD | May 2019, updated May 2024 | First intergovernmental standard | 5 values-based principles + 5 government recommendations | The bridge from corporate (2018) to intergovernmental (2021) — covered in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.4 |
| *(Companion)* **G7 Hiroshima AI Process** | G7 | 2023–2025 | Guiding principles + Code of Conduct | Advanced-AI developer commitments | The generative-AI-era international instrument — cross-ref [ai_verify_guide.md](ai_verify_guide.md) §8 |

> **How the international and corporate layers interact:** UNESCO and IEEE supply the *legitimacy* and the *consensus*; the corporates supply the *operating procedures*; the regulators (NIST/EU/ISO/MAS) supply the *enforcement*. A bank's RAI policy that starts from the five-value convergence (fairness, safety, privacy, transparency, accountability), cites UNESCO's ten principles for its external commitments, and runs on NIST/ISO machinery is aligned at every layer — that exact stack is built in [§7](#7-the-worked-example-a-rai-framework-design).

---

## 4. The Tooling

Principles without tooling are a press release. The RAI toolchain that emerged 2018–2020 — mostly from the same companies that published the principles — makes fairness, robustness, and transparency *measurable*. For a bank, these tools are the evidence machines: they produce the numbers that go into model cards, bias audits, and regulator-facing files (cross-ref the trust guide's §4 methods, which this section operationalizes).

### 4.1 IBM AI Fairness 360 (AIF360)

**Verified:** **AI Fairness 360 (AIF360)** is IBM Research's open-source Python toolkit for detecting and mitigating algorithmic bias — "a comprehensive set of fairness metrics for datasets and machine learning models, explanations for these metrics, and algorithms to mitigate bias in datasets and models" (GitHub `Trusted-AI/AIF360`, Apache-2.0). First released in **2018** with the companion paper *AI Fairness 360: An Extensible Toolkit for Detecting and Mitigating Algorithmic Bias* (Bellamy et al., arXiv:1810.01943, October 2018). It is hosted under the Linux Foundation's Trusted-AI umbrella alongside IBM's other RAI tools.

**What it contains:**

- **Fairness metrics (70+)**: group-fairness measures computed on a dataset or a model's predictions — statistical parity difference, **disparate impact** (the ratio version of the US "4/5ths rule"), equal opportunity difference, average odds difference, theil index, and many more; each metric ships with an explanation of what it measures.
- **Bias-mitigation algorithms** across the three intervention points:
  - *Pre-processing* (fix the data): reweighing, optimized pre-processing, disparate-impact remover, learning fair representations.
  - *In-processing* (change the learner): adversarial debiasing, prejudice remover.
  - *Post-processing* (fix the decisions): equalized-odds post-processing, calibrated equalized-odds post-processing, reject-option classification.
- **Fairness explanations and dashboards**: dataset and model explanation modules that visualize where bias enters.
- A **metric-explanation pairing** design (each metric links to mitigations that address it) — the toolkit's signature feature.

**Banking fit:** AIF360 is the standard reference for credit/bias audits of traditional ML models (scorecards, PD/LGD models) — it slots directly into the trust guide's bias-audit method (§4.3 there) and the fairness dimension (§1.2). The 4/5ths-rule-adjacent disparate-impact metric is exactly what fair-lending analytics ([§6.2](#62-fair-lending-analytics-flagged)) needs for a first-pass screen.

### 4.2 Microsoft Fairlearn

**Verified:** **Fairlearn** is an open-source Python toolkit "to help data scientists assess and improve fairness of AI systems" (fairlearn.org; GitHub `fairlearn/fairlearn`). It originated at **Microsoft Research** and was **open-sourced in 2020**, with the canonical paper *Fairlearn: A toolkit for assessing and improving fairness in AI* (Bird, Dudík, Edgar, Horn, Lutz, Milan, Sameki, Wallach, Walker — Microsoft Research publication, 2020) and a major updated treatment in the Journal of Machine Learning Research 2023 (*Fairlearn: Assessing and Improving Fairness of AI Systems*, JMLR 24(2023) 23-0389). It is now community-governed and explicitly frames fairness as **sociotechnical** — the library's metrics and mitigations are meant to be used alongside judgment about the societal context, a stance that matches the trust guide's "state your fairness criterion" discipline (§4.3 there).

**What it contains:**

- **Fairness metrics**: demographic parity difference, equalized odds difference, error-rate parity, and others, computed on user-defined sensitive features.
- **Mitigation algorithms**: the **reductions** approach (ExponentiatedGradient, GridSearch — treat fairness as a constrained-optimization problem over the model class), ThresholdOptimizer (post-processing threshold selection per group), and post-processing wrappers; regression support included.
- **A Fairness dashboard** (interactive visualization of group disparities vs. performance) — the component that made Fairlearn famous.
- Its own flagship **credit-card loan use case** (assessing/mitigating fairness in a default model across applicant groups defined by sex — verified on fairlearn.org), which makes it the most banking-ready demo of any fairness tool.

**AIF360 vs. Fairlearn in one line:** AIF360 is the broader *research* toolkit (70+ metrics, many algorithms, explanations); Fairlearn is the *practice* toolkit (clean API, dashboard, sociotechnical guidance, tight sklearn integration). Banks typically use AIF360 for deep audits and Fairlearn in the MLOps loop for continuous monitoring — both feed the same model-card evidence.

### 4.3 IBM ART: The Adversarial Robustness Toolbox

**Verified:** The **Adversarial Robustness Toolbox (ART)** is the open-source Python library for **machine learning security** — "tools that enable developers and researchers to defend and evaluate Machine Learning models and applications against the adversarial threats of Evasion, Poisoning, Extraction, and Inference" (GitHub `Trusted-AI/adversarial-robustness-toolbox`). It was **started by IBM Research (2018; the v1.0.0 paper is arXiv:1807.01069, July 2018)** and **moved to the Linux Foundation AI & Data (LF AI & Data) in July 2020** (verified on adversarial-robustness-toolbox.org), where it sits with AIF360 under the Trusted-AI umbrella.

**What it contains:**

- **Attacks** across the four NIST AI 100-2 threat classes: evasion (FGSM, PGD, Carlini-Wagner, adversarial patch...), poisoning (backdoor, label-flipping...), extraction (model stealing), and inference (membership inference, attribute inference) — the same taxonomy the trust guide's companion [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) covers in depth.
- **Defenses and detectors**: adversarial training, input sanitization/denoising, certified-robustness (randomized smoothing), and attack detectors.
- **Model support** spanning the practical stack: deep neural networks, gradient-boosted decision trees, SVMs, random forests, logistic regression, Gaussian processes, scikit-learn pipelines (verified from the v1.0.0 paper abstract).
- **Certified-robustness verification** for neural networks.

**Banking fit:** ART is the tooling behind the "secure & resilient" characteristic and the adversarial-testing method of a trust assessment — e.g., verifying that a fraud model is not silently steered by crafted features, or that an LLM-based assistant's classifier head resists evasion. It pairs with [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) (the gateway is the operational control; ART is the test machinery).

### 4.4 Interpretability: SHAP and LIME (Cross-Reference)

The interpretability layer is already treated in depth in the trust guide's §4.4 (Explainability, XAI: SHAP/LIME) — including the practical cautions about explanations being approximations. The cross-reference facts:

- **SHAP (SHapley Additive exPlanations)** — Lundberg & Lee, 2017 (arXiv:1705.07874). Game-theoretic Shapley-value attributions; the only attribution method satisfying local accuracy, missingness, and consistency axioms; model-agnostic (KernelSHAP) with fast exact versions for tree ensembles (TreeSHAP). Produces per-prediction feature contributions aggregatable to global importance.
- **LIME (Local Interpretable Model-agnostic Explanations)** — Ribeiro, Singh & Guestrin, 2016 (arXiv:1602.04938). Local surrogate models (linear fits on perturbed inputs) explaining individual predictions; fast and intuitive, but less stable than SHAP across perturbations.
- **Caveats (from the trust guide, do not skip them):** explanations approximate the model, not the model; LIME fidelity is weaker; LLM token-level attributions are noisier and less mature; and every explanation intended for regulators or customers needs human review first. In banking, SHAP is the workhorse of SR 11-7 conceptual-soundness reviews and FEAT transparency responses — see [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.4 and §6.

Both are the *explanation* companion to the *documentation* artifacts below: SHAP/LIME answer "why did this prediction happen?", model cards and datasheets answer "what is this thing, and what is it for?".

### 4.5 Model Cards for Model Reporting (Mitchell et al., 2019)

**Verified:** **Model Cards for Model Reporting** — Margaret Mitchell, Simone Wu, Andrew Zaldivar, Parker Barnes, Lucy Vasserman, Ben Hutchinson, Elena Spitzer, Inioluwa Deborah Raji, and Timnit Gebru — arXiv:1810.03993 (October 2018), **published at the 1st ACM Conference on Fairness, Accountability, and Transparency (FAT\* 2019), Proceedings of Machine Learning Research Vol. 81, 2019**. The paper proposes that every trained model ship with a **model card**: a short, structured document reporting benchmark evaluation "in a variety of conditions, such as across demographic groups, environmental situations, and intersectional groups" (verified abstract wording), covering model details, intended use, factors (the groups/environments the model was evaluated on), metrics, training data, evaluation data, ethical considerations, and caveats and recommendations. The paper positions this as "a step towards the responsible democratization of machine learning" — transparency as the precondition for accountability.

**Why the model card is the RAI artifact par excellence:** it is the single document that makes fairness numbers, robustness numbers, and intended-use boundaries *portable* — from the data-science team to MRM, from the vendor to the bank, from the bank to the regulator. The trust guide's transparency dimension (§1.2) names model/data cards as the primary transparency assessment; the EU AI Act's high-risk technical documentation and ISO 42001 records are the legal heirs of the same idea. Practical evolution: Google ships model cards for many of its models (the paper grew out of Google); Hugging Face made model cards a *platform feature* with a required template on the Hub (cross-ref [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) for the hub-governance angle); and Model Cards Toolkit exists for automation.

### 4.6 Datasheets for Datasets (Gebru et al., 2018)

**Verified:** **Datasheets for Datasets** — Timnit Gebru, Jamie Morgenstern, Briana Vecchione, Jennifer Wortman Vaughan, Hanna Wallach, Hal Daumé III, and Kate Crawford — **arXiv:1803.09010, first submitted 23 March 2018** (last revised December 2021; **published in Communications of the ACM, December 2021**). The paper's analogy: in electronics, every component ships with a datasheet describing its operating characteristics, test results, and recommended uses — so every *dataset* should too. The datasheet documents the dataset's **motivation, composition, collection process, recommended uses, and (critically) its limitations** (verified abstract), with question templates for dataset creators and consumers covering topics like collection context, preprocessing, intended and unintended uses, and sensitive attributes.

**Why datasheets matter in banking:** most model harm traces to *data* decisions, not model code — a training set that under-represents a demographic, was collected under different market conditions, or contains label leakage. A datasheet makes those facts reviewable at procurement time (vendor datasets) and at validation time (internal datasets). Together with model cards, datasheets complete the documentation pair: **datasheet documents the input, model card documents the system** — and both are named artifacts in the trust guide's audit criteria (documentation completeness, §4.1 there) and in Singapore's AI Verify evidence structure (cross-ref [ai_verify_guide.md](ai_verify_guide.md) §3.1). Datasheets-for-datasets tooling exists (e.g., IBM's DataSheet for Datasets tooling, Microsoft's dataset-documentation templates), and data-centric AI platforms increasingly enforce them.

### 4.7 The Tooling Table

| Tool | Function | Notes |
|---|---|---|
| **AI Fairness 360 (AIF360)** — IBM Research, 2018 (arXiv:1810.01943) | Bias **detection and mitigation**: 70+ fairness metrics (disparate impact, statistical parity, equal opportunity...) + pre/in/post-processing mitigation algorithms + fairness explanations | Apache-2.0; Linux Foundation Trusted-AI umbrella; the deep-audit reference for credit models; metric→mitigation pairing is its signature |
| **Fairlearn** — Microsoft Research, open-sourced 2020 (JMLR 2023) | Fairness **metrics and mitigations** with a dashboard; reductions-based algorithms (ExponentiatedGradient, GridSearch), ThresholdOptimizer; sociotechnical framing; flagship credit-card-loan use case | Community-governed; sklearn-native; best for in-loop monitoring; pairs with AIF360 (research breadth vs. practice ergonomics) |
| **ART (Adversarial Robustness Toolbox)** — IBM, 2018 (arXiv:1807.01069); LF AI & Data since July 2020 | ML **security**: attacks + defenses + detectors + certified robustness across **evasion, poisoning, extraction, inference** (the NIST AI 100-2 taxonomy) | The robustness/security test machinery; feeds the trust guide's adversarial-testing method; pairs with the enterprise AI gateway |
| **SHAP** (Lundberg & Lee, 2017) | **Local + global explainability** via Shapley values; TreeSHAP fast path; axiom-grounded | The banking workhorse for SR 11-7 conceptual-soundness and FEAT transparency; cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.4 |
| **LIME** (Ribeiro et al., 2016) | **Local surrogate** explanations; fast, intuitive | Weaker fidelity than SHAP; use for spot-checks; same cross-ref |
| **Model Cards** (Mitchell et al., FAT\* 2019) | **Documentation artifact**: benchmark evaluation across groups/conditions, intended use, factors, metrics, ethical considerations, caveats | The transparency artifact; Hugging Face made it a platform feature; ancestors of EU-Act technical documentation and ISO 42001 records |
| **Datasheets for Datasets** (Gebru et al., 2018; CACM 2021) | **Dataset documentation**: motivation, composition, collection, recommended uses, limitations | The input-side counterpart to model cards; procurement review artifact for vendor data; named in audit criteria and AI Verify evidence |

> **The tooling pattern:** fairness (AIF360/Fairlearn), robustness (ART), interpretability (SHAP/LIME), documentation (model cards/datasheets) — four tool families, one evidence file. Every one of them produces the artifacts a trust assessment (§4–§5 of the trust guide) and an MRM validation need; none of them replaces the human judgment that decides *which* fairness metric or *which* threat model applies. Tooling measures; governance decides.

### 4.8 Choosing and Combining the Tooling

Selection guidance that survives contact with a bank's reality:

- **Match the tool to the model class.** AIF360 and Fairlearn cover traditional ML (sklearn-style pipelines, gradient boosting, neural nets) natively; SHAP has a fast path for tree ensembles (TreeSHAP) that makes it cheap enough for routine validation. For LLMs, the classical bias toolkits still work on *classifier heads* and structured downstream tasks, but LLM-specific evaluation (stereotype suites, counterfactual perturbation, toxicity-by-group — see [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) §4–§6) needs the eval-benchmark stack (AI Verify's Moonshot, HELM-style harnesses) rather than AIF360 alone.
- **Fairness is two questions, not one.** *Detection* (metrics: which group difference, which criterion) is AIF360/Fairlearn's job; *mitigation* (reweighing vs. adversarial debiasing vs. threshold post-processing) changes the model, so mitigation choices must be validated like any model change — the mitigation itself goes through MRM, not just the metrics. The trust guide's "state the criterion" rule ([§4.3](ai_trust_assessments_guide.md)) applies to mitigation too: post-processing that equalizes approval rates can change calibration, and calibration-preserving mitigations can leave approval-rate disparities — you cannot have all three (Kleinberg et al. impossibility, cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) §3.5).
- **Robustness is threat-modeled, not blanket.** ART's value is that it forces the threat-model question: evasion matters for screening classifiers (attackers craft inputs), poisoning matters for models retrained on external data, extraction/inference matter for sensitive models exposed via APIs. Run the relevant classes; do not run everything on everything.
- **Documentation tools are the cheapest and most underused.** Model cards and datasheets are *templates and process*, not software — the marginal cost is a few hours per model, and they are the artifacts auditors actually read first. Automate their generation from the registry (Model Cards Toolkit, Hub templates) so they cannot be forgotten.
- **One platform, not five point tools.** In a bank, the tooling should be packaged as one "RAI toolkit" service on the MLOps platform (cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md)) with a single entry point, so that evidence lands in the model registry in one schema. The worked example's [§7.4](#74-the-tooling-design) table is the deployment shape.

---

## 5. The Implementation

The corporate frameworks ([§2](#2-the-corporate-frameworks)) supply the principles and the international instruments ([§3](#3-the-international)) supply the consensus, but an enterprise RAI program lives or dies on four implementation decisions: **who governs, what the policies say, how AI gets procured, and how AI gets inventoried.** This section covers those four; the worked example ([§7](#7-the-worked-example-a-rai-framework-design)) assembles them.

### 5.1 The Governance Structures: Ethics Boards and RAI Committees

**Verified pattern:** every serious corporate RAI program — and every serious bank program — has a layered governance structure. The common three layers, observed consistently across Microsoft (Office of Responsible AI + RAI Committee + Review Board), IBM (AI Ethics Board, later Responsible Technology Board + Integrated Governance Program), Google (Advanced Technology Review Council), Salesforce (Office of Ethical and Humane Use), and the wider industry:

1. **Board / executive layer.** The board or an executive committee owns the RAI **risk appetite**: a one-page statement of which AI uses the firm will and will not pursue. The board receives RAI status reporting (use-case inventory, incidents, material findings) — this is also what MAS FEAT's Ethics principle demands ("board-level oversight of AI use", cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.4).
2. **Center-of-excellence / program layer.** A central RAI function (an "Office of Responsible AI" analog) owns the *program*: the principle set, the standard, the policy library, the risk-assessment templates, the training, and the metrics. It staffs the review bodies and maintains the inventory.
3. **Review bodies.** An **ethics board / RAI committee** reviews use cases that trip defined triggers — high materiality, sensitive personal data, credit or employment decisions, generative AI with external-facing output, any use on the restricted list. The review produces a written decision (approve / approve-with-conditions / reject) that becomes part of the evidence file. Some organizations split a standing committee (routine triage) from a senior review board (escalations and hard cases) — the Microsoft pattern.

Three structural lessons from the failures and successes of the last five years:

- **The review body must be able to say no — and must be seen to.** An ethics board that only stamps approvals is a compliance theater risk; its decisions (including rejections) should be logged and reported to the executive layer.
- **Independence beats seniority.** The most effective committees mix second-line (risk/compliance), engineering, legal, and — for hard cases — external members (academics, civil society), so that the review is not the first line reviewing itself.
- **The structure must scale to volume.** With dozens of GenAI use cases, a single committee becomes the bottleneck; the answer is tiering — defined low-risk categories (internal, non-personal-data, low-impact) auto-approved with documented self-assessment, while triggers route to the committee. This mirrors the EU AI Act's risk-tiering logic and the trust guide's inventory-triage step (§7.2 there).

### 5.2 RAI Policies and Risk Assessment

An RAI program runs on four documents, in a strict hierarchy:

1. **The AI principles** — the one-page commitments (adopt the five-value convergence from [§2.5](#25-the-corporate-table): fairness, safety/reliability, privacy/security, transparency/explainability, accountability, plus any firm-specific values).
2. **The RAI standard / policy** — the engineering-facing requirements: what every AI system must have (impact assessment, bias testing, documentation, monitoring) keyed to risk tier. The Microsoft Responsible AI Standard (v2, June 2022) is the template for the genre: goals per principle, requirements per goal, lifecycle staging, and explicit extra oversight for significant-adverse-impact systems.
3. **The AI risk-assessment procedure** — how an AI system gets classified and assessed. The working pattern, which composes NIST-style risk practice with ISO 42001-style records (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.6): intake → classification (risk tier, EU-Act tier check, FEAT relevance, materiality) → impact assessment (the ISO 42001 AI-system impact assessment, which is AI's answer to the DPIA) → bias and robustness testing plan → sign-off → monitoring plan → incident reporting. The trust guide's §7 process (inventory → risk register → assessment → reporting) is the full-blooded version of this.
4. **The incident / escalation policy** — what counts as an AI incident, who declares it, how it is investigated, when the regulator is told (EU AI Act GPAI incident reporting, and increasingly MAS expectations, make this non-optional).

**Risk assessment in practice:** the deliverable is a **risk register** with per-use-case entries: system, tier, principle-by-principle assessment (fairness: which metric, what threshold; transparency: which disclosures; accountability: which owner), testing evidence references, and residual-risk sign-off. [verify: no single industry-standard register format exists — firms adapt NIST RMF profiles or ISO 42001 records; the *content* above is the consistent core.] A worked register row (the shape, not the numbers):

| Field | Example entry (SME credit-scoring model) |
|---|---|
| System / tier | SME Credit Score v4 — **high** (credit decision, personal data, EU-Act Annex III-adjacent, FEAT-relevant) |
| Owner / validator | Business owner: SME Lending; validation: MRM; review: RAI Committee (Feb 2026) |
| Fairness | Criterion declared: **equal opportunity** (FPR parity across age/sex groups); 80%-rule screen on approval rates; AIF360 audit report ref `AIF-2026-014` — disparate impact 0.87 (pass) |
| Safety/reliability | Golden-set accuracy 91.4%; ART evasion check (ref `ART-2026-009`) no material shift; drift monitor monthly |
| Privacy/security | PDPA/GDPR DPIA completed; no prohibited-basis data collected; gateway PII-redaction enforced |
| Transparency | Model card + datasheets in registry (refs); SHAP exhibits for adverse-action reasons; customer disclosure text approved |
| Accountability | Sign-offs: MRM head, CRO delegate; committee decision: **approve with conditions** (re-audit at 6 months) |
| Monitoring / incidents | Drift + fairness monitors live; incident log: none for this model |

### 5.3 Procurement

Most AI in a bank is **bought, not built** — foundation models, vendor ML platforms, credit-scoring services, SaaS copilots. Procurement is therefore where RAI is won or lost, and the practice has consolidated into a repeatable pattern:

- **The RAI questionnaire.** Every AI vendor assessment includes a responsible-AI section: published principles and governance structure (does the vendor have a real review body?); documentation artifacts (model cards, datasheets for the datasets they trained on or resell); testing evidence (bias audits, robustness/red-team reports); EU-Act/GPAI posture (does the vendor supply the GPAI documentation the Act requires of deployers?); and incident history.
- **The procurement clause.** Contracts now carry RAI terms: warranty that the model's training data was lawfully obtained; obligations to supply documentation and testing evidence on request; restrictions on using customer data to train shared models (the single most common bank red line); audit rights; incident-notification SLAs; and liability allocation for AI-caused harm. The IEEE P7000 standards chain ([§3.2](#32-ieee-ethically-aligned-design)) shows up here as contract language.
- **The model-supply-chain view.** For foundation models, procurement is really *supply-chain risk*: which upstream model, which training-data summary, which documentation flows down to the bank under the EU AI Act (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.2's GPAI discussion), and which of the vendor's commitments survive the bank's deployment. The gateway layer ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) is where the technical controls attach; procurement is where the contractual controls attach.
- **Vendor RAI ≠ bank RAI.** A vendor's model card does not discharge the bank's obligations for the bank's *use*: the bank's own impact assessment, bias testing on the bank's population, and monitoring remain the bank's job. Procurement buys evidence; it does not buy absolution.

### 5.4 Model-Inventory Integration

The fourth implementation decision: RAI artifacts must attach to a **model/AI inventory** that is one source of truth. The trust guide makes the point in §7.1: MRM's model inventory (SR 11-7), the platform's model registry, and the compliance team's use-case list should all reconcile to a single inventory — because without it, risk assessment runs on rumors. The RAI program adds to each inventory entry: the impact assessment, the model card, the datasheets of training/eval data, the bias and robustness test reports, the committee decision, and the monitoring plan. This is also the record base for ISO 42001 certification (if pursued) and for FEAT/EU-Act regulator-facing evidence. For the agent era, the inventory must extend to agent definitions, tool permissions, and human-oversight design — the governance angle treated in [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md).

### 5.5 The Implementation Table

| Implementation lever | What it is | Verified pattern / notes |
|---|---|---|
| **Governance structure** | Layered RAI governance: board/executive risk appetite → central RAI office/program → review bodies (ethics board, RAI committee) | Microsoft (Office of RAI + RAI Committee + Review Board), IBM (AI Ethics Board → Responsible Technology Board + IGP), Google (Advanced Technology Review Council), Salesforce (Office of Ethical and Humane Use); review bodies must have real veto power and tiered triage for volume |
| **Policies & risk assessment** | Hierarchy: AI principles → RAI standard/policy → risk-assessment procedure → incident policy; per-use-case risk register with tiering | Microsoft Responsible AI Standard v2 (June 2022) is the genre template; composes with NIST RMF (process), ISO 42001 (records, incl. AI system impact assessment), EU AI Act (obligations), FEAT (board oversight) — cross-ref trust guide §2.6, §7 |
| **Procurement** | RAI vendor questionnaire + contract clauses + model-supply-chain due diligence; bank's own assessment is never outsourced | Questionnaire: principles, governance, model cards/datasheets, test evidence, GPAI posture, incidents. Clauses: data-use restrictions, documentation rights, audit rights, incident SLAs, liability. EU-Act documentation flows down to deployers |
| **Model-inventory integration** | RAI artifacts attached to the single AI inventory (MRM + platform registry + compliance list reconcile) | The inventory entry carries: impact assessment, model card, datasheets, test reports, committee decision, monitoring plan; extends to agents in the agent era (cross-ref [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md)) |

### 5.6 The RAI Maturity Model

Frameworks are implemented in stages; the industry pattern (consistent with how the corporate programs above actually grew, and with the ISO 42001 continual-improvement structure) is four maturity levels. Useful for positioning a program in a steering-committee deck and for sequencing investment:

| Level | Name | What exists | Typical state |
|---|---|---|---|
| **L1** | **Ad hoc** | Individual champions do bias checks and documentation; no standard, no committee, no inventory integration | Common at firms where AI grew bottom-up; the pre-program state |
| **L2** | **Defined** | Published principles, a risk-assessment template, a review committee, mandatory model cards for high-tier systems; inventory exists but partial | The target for year one of a program; this is where the worked example's [§7.6](#76-the-rollout-plan-twelve-months) plan lands |
| **L3** | **Managed** | Standard keyed to tiers, tooling platform integrated with MLOps, monitoring in production, incident policy exercised, evidence file per system, annual FEAT/EU-Act readiness reporting | The target for a bank facing EU-Act high-risk obligations and MAS supervisory scrutiny |
| **L4** | **Optimized** | Continual improvement: RAI metrics tracked like KPIs, third-party assurance (ISO 42001 certification, AI Verify-style published reports, external board members), lessons-learned loops feeding the standard, agent-era governance extended | Rare; the aspirational end-state; the pattern is visible in IBM's current governance apparatus ([§2.3](#23-ibm-trust-and-transparency)) and in certified SG AI firms (cross-ref [cedars_digital_company_guide.md](cedars_digital_company_guide.md) for a real ISO-42001-certified example) |

Two governance cautions at every level: (1) **maturity is measured by evidence, not documents** — the level is what an auditor finds in the inventory, not what the policy library claims; (2) **the committee's reject rate is a health signal** — a committee that never rejects is either not seeing hard cases or not empowered, and both are failures. [verify: the four-level shape above is the author's synthesis of the corporate programs' evolution and ISO 42001's improvement clauses — no single industry standard defines these levels; treat the table as a positioning tool, not a benchmark.]

---

## 6. The Banking Angle

Banks were early adopters of RAI not because they are more virtuous than other industries but because they are more supervised. Credit decisions touch protected groups (fairness), model failures move money (safety), and regulators hold management personally accountable (accountability). Two threads matter for a Cymbal Bank-Singapore architect: the local regulator's FEAT principles, and the fair-lending analytics that make fairness measurable where it is legally consequential.

### 6.1 MAS FEAT (Cross-Reference)

**Verified — cross-ref the trust guide's §3.4, which carries the verified facts.** The **Monetary Authority of Singapore** published the **Principles to Promote Fairness, Ethics, Accountability and Transparency (FEAT) in the Use of Artificial Intelligence and Data Analytics in Singapore's Financial Sector on 12 November 2018** — non-binding but authoritative expectations for financial institutions, each principle carrying specific guidance items (per [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.4):

- **Fairness** — AIDA (AI and data analytics) decisions should be sound and fair; data should be relevant and not systematically biased against protected groups; models are regularly reviewed and validated for accuracy and bias.
- **Ethics** — AIDA should be used in line with the institution's ethical standards, with management accountability and board-level oversight.
- **Accountability** — institutions are accountable for AIDA-driven decisions: clear allocation of responsibility, auditability, and model risk management.
- **Transparency** — disclosure to consumers and timely explanation of materially AI-influenced decisions.

Two implementation notes from the trust guide worth repeating here: (1) FEAT is *voluntary-but-authoritative* — MAS expects banks to be able to *demonstrate* fair dealing, which in practice means documented bias measurement and mitigation (the exact job of the [§4](#4-the-tooling) tooling); (2) the accompanying **Veritas initiative** — the MAS-industry consortium — publishes the assessment methodologies that turn FEAT principles into checkable processes (Fairness Assessment Methodology first, 2020). For a Singapore-based bank, FEAT is the RAI report card: every corporate principle in [§2](#2-the-corporate-frameworks) has a FEAT mapping, and every artifact in [§4](#4-the-tooling) is evidence for one of the four letters.

### 6.2 Fair-Lending Analytics (Flagged)

Fairness becomes *legally* binding at the lending decision — and the analytics that demonstrate it are the sharpest test of an RAI program. Cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) §6.3, which carries the verified regulatory facts:

- **US: ECOA (1974) and Regulation B (12 CFR Part 1002)** — it is unlawful to discriminate in any aspect of a credit transaction on protected grounds; **disparate impact** (a facially neutral policy that disproportionately harms a protected class without business necessity) is prohibited even without intent — which is precisely what an 80%-rule failure evidences; adverse-action notices must state specific reasons (a black-box model needs a reason engine); and Reg B limits collection of prohibited-basis data, which complicates measurement (proxies, testing, or synthetic approaches).
- **EU: AI Act Article 10** — high-risk AI training data (which includes creditworthiness assessment under Annex III) must be representative and bias-examined.
- **Singapore: MAS FEAT Fairness** — individuals/groups must not be systematically disadvantaged unless justifiable; data and models must be regularly reviewed for accuracy and bias.

**The analytics stack (flagged — practice, not regulation):** the established fair-lending analytics pattern runs: (1) an **80%-rule / disparate-impact screen** (AIF360's disparate-impact metric, [§4.1](#41-ibm-ai-fairness-360-aif360)) as the first-pass tripwire; (2) **group-fairness metric suites** with a *declared* criterion (demographic parity vs. equal opportunity vs. calibration — the impossibility result from the governance guide §3.5 means you must choose and justify); (3) **causal/adverse-action analysis** for the reason statements; (4) **threshold and post-processing mitigation** (reject-option classification, threshold optimization — AIF360/Fairlearn) with documented business-necessity justification for any remaining disparity; (5) **monitoring** — because drift can reintroduce disparity after launch. [verify: no single regulator-mandated methodology exists; the ECOA/Reg B/Art. 10/FEAT obligations are verified, the analytics sequence is industry practice synthesized here.]

**The flagged risk:** fair lending is where RAI meets *litigation and supervision*, and three recurring failure modes are documented in industry coverage: measuring on proxies without validating the proxy (prohibited-basis data collection limits make this endemic), declaring one fairness metric while regulators and courts reason with another, and treating a one-time audit as a control (disparity is a live property). The RAI answer is the same as everywhere else: declared criteria, documented evidence, and continuous monitoring — but here the stakes are enforcement actions, not reputational noise.

### 6.3 The Banking Table

| Banking thread | What it is | RAI response (verified pattern) |
|---|---|---|
| **MAS FEAT** (12 Nov 2018) | Singapore's Fairness-Ethics-Accountability-Transparency principles for AI/data analytics in financial services; Veritas methodologies operationalize them | FEAT-mapped evidence file per use case: bias audits (fairness), board oversight (ethics), MRM integration and sign-offs (accountability), disclosures + XAI (transparency) — verified facts in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.4 |
| **Fair-lending law** (ECOA/Reg B; AI Act Art. 10; FEAT fairness) | Disparate-impact and bias obligations on credit decisions; adverse-action reason requirements | Declared fairness criterion + 80%-rule screen + group-metric suites + reason engines + threshold mitigation + continuous monitoring; ECOA/Reg B facts verified in [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) §6.3 |
| **Model risk (SR 11-7)** | Bank model governance regime — inventory, validation, documentation, independent review | RAI artifacts double as MRM evidence: model cards, bias audits, SHAP analyses, committee decisions; cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) |
| **Regtech / conduct** | The compliance layer around the above | FEAT-aligned assessment programs and AI Verify-style evidence are the compliance currency; cross-ref [../banking/regtech_guide.md](../banking/regtech_guide.md) and [ai_verify_guide.md](ai_verify_guide.md) |

> **The banking summary line:** in banking, RAI is not a values exercise — it is the discipline that turns FEAT, ECOA/Reg B, the AI Act's Article 10, and SR 11-7 into one evidence system. The corporate principles say *why*; the tools ([§4](#4-the-tooling)) produce *what*; the governance ([§5](#5-the-implementation)) assigns *who*; the worked example below shows the whole thing assembled.

### 6.4 The Wider Singapore Banking Context

For a Singapore-based architect, three further strands complete the RAI picture beyond FEAT itself:

- **MAS's supervisory posture.** MAS has been explicit that AI/ML models fall under existing model-risk expectations and that FEAT-aligned governance is expected practice — so the RAI evidence file is not optional window-dressing, it is the answer to "show us your governance" in supervisory dialogue. Cross-ref [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) for the local banking landscape and [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.6 for the wider SG context.
- **The testing layer.** Singapore's answer to "prove it" is AI Verify: voluntary, but the *credible* evidence pattern — a flagship system with an AI Verify-style report (framework process checks + toolkit technical tests + Moonshot for LLMs) is the strongest FEAT demonstration a bank can make. Cross-ref [ai_verify_guide.md](ai_verify_guide.md) for the full deep-dive and its worked assessment.
- **The ecosystem precedent.** Singapore already hosts companies whose *entire business model* is certified RAI — e.g., the ISO-42001-certified SG AI firm profiled in [cedars_digital_company_guide.md](cedars_digital_company_guide.md). That is the L4 maturity state ([§5.6](#56-the-rai-maturity-model)) made real: certification, third-party audit, published governance. A bank does not need to match a pure-play AI firm's posture, but the existence of certified peers resets the credibility bar for "we take RAI seriously" claims.

---

## 7. The Worked Example: A RAI Framework Design

This section designs a complete responsible-AI framework for a bank. The scenario deliberately reuses the familiar Cymbal Bank context of the sibling guides so that the cross-references land: same bank, same Singapore constraints, same MRM machinery — but here the deliverable is the *framework itself*, not an assessment of one system.

### 7.1 The Scenario: A Cymbal Bank

**The setting.** A European-headquartered investment bank's Singapore entity (call it the Bank) — mirroring Jack's environment at Cymbal Bank: a global wholesale bank with a Singapore hub, running trade finance, markets, cash management, and corporate lending; supervised by MAS in Singapore and by home regulators in Europe (which means the **EU AI Act applies to EU-entity systems**, and **FEAT applies to Singapore systems** — the jurisdiction sweep from the trust guide's worked example §8.1 applies). The Bank has ~60 AI systems: traditional ML (credit scoring for SMEs, trade-finance document extraction, sanctions-screening classifiers, fraud models) plus a fast-growing GenAI estate (a trade-ops copilot, a client-communication summarizer, a code assistant, agent experiments).

**The trigger.** The Group's European parent is preparing for EU AI Act high-risk obligations (Annex III, from August 2026) and the Singapore entity's MAS relationship manager has asked, in the context of the annual supervisory review, for the entity's AI governance posture to be articulated **in FEAT terms**. The CTO and the MRM head ask Jack's team to design a Group-aligned but Singapore-adapted **Responsible AI Framework** — principles, governance, tooling, evidence — that will (a) satisfy FEAT and the parent's RAI standard, (b) be ready for the EU AI Act, and (c) be lightweight enough that 60 systems and a dozen GenAI use cases can actually be governed, not just documented.

**Design constraints (from the siblings, reused deliberately):** the framework must compose with the trust guide's NIST/ISO/EU stack (§2.6 there); it must produce the evidence that AI Verify-style testing and FEAT reporting expect (cross-ref [ai_verify_guide.md](ai_verify_guide.md)); it must plug into the existing MRM inventory (SR 11-7, cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md)); and it must attach to the enterprise AI gateway for technical controls ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)).

### 7.2 The Principles Design

**The principle set.** Adopt the five-value convergence from [§2.5](#25-the-corporate-table) — the values common to Microsoft, Google, IBM, UNESCO, and FEAT — and add the two values the international layer insists on and the Bank's context makes material:

| Principle | Source lineage | What it commits the Bank to |
|---|---|---|
| **Fairness** | Microsoft; Google; IBM; UNESCO #3; **FEAT #1** | No systematic disadvantage on protected grounds; declared fairness criterion per use case; bias measured, mitigated, monitored |
| **Safety & reliability** | Microsoft; Google; UNESCO #2 | Systems function as intended under expected and unexpected conditions; GenAI output is validated and grounded; failure modes are tested (red-teaming, ART) |
| **Privacy & security** | Microsoft; Google; UNESCO #5 | Data protection by design (PDPA, GDPR), lawful data use, adversarial-ML resistance, gateway-enforced controls |
| **Transparency & explainability** | All five corporates; UNESCO #7; **FEAT #4** | Model cards + datasheets on every system; SHAP/LIME where explanations matter; customer disclosure for materially AI-influenced decisions |
| **Accountability** | All five corporates; UNESCO #8; **FEAT #3** | Named owners per system; board-level oversight (FEAT Ethics); sign-offs, audit trails, incident policy |
| **Inclusiveness & human agency** | Microsoft; IBM (augmentation); UNESCO (diversity); **FEAT Ethics** | Human oversight and determination (UNESCO #6); AI augments, humans decide; accessibility by design |
| **Sustainability & literacy** | UNESCO #4, #9 | Model-efficiency budgets (cross-ref [low_latency_genai_patterns_guide.md](low_latency_genai_patterns_guide.md) lightly); AI literacy for staff (an EU AI Act obligation and FEAT-adjacent expectation) |

**The negative list.** Copy the Google pattern ([§2.2](#22-google-the-ai-principles-2018)): a short list of prohibited uses — e.g., no AI with unrestricted autonomous financial decisions affecting customers without human review; no use of customer data for shared-model training; no surveillance applications; no social-scoring uses. The negative list is what makes the principles enforceable: it gives the review body a reject path that does not require arguing about values case-by-case. It also maps forward to the EU AI Act's prohibited-practices tier (social scoring by public authorities, manipulative techniques exploiting vulnerabilities) — the Bank's internal negative list should be *at least as strict* where it covers banking conduct, and the mapping is recorded so the EU-Act classification exercise ([ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.2) reuses the same reasoning.

**The mapping artifact.** Every principle carries a mapping to: the FEAT letter(s) it serves; the EU AI Act article(s) it anticipates (Art. 10 data/bias, Art. 14 human oversight, Art. 13 transparency, Art. 12 logging); the NIST RMF function(s) it feeds; and the trust guide's trust dimension(s) (§1.2 there). This mapping table is the framework's single most reused artifact — it is what lets one evidence file serve FEAT, MRM, ISO 42001 records, and the EU Act (the composition argument of trust guide §2.6, made real).

### 7.3 The Governance Design

**Three layers, mapped to the Bank's existing machinery:**

1. **Board / executive.** The Singapore entity's Technology & Operations Risk Committee (TORC) receives a **quarterly RAI dashboard**: use-case inventory count by tier, incidents, committee decisions, FEAT-readiness status. The AI risk appetite statement (one page: what the Bank will/won't do with AI) is approved at this level — satisfying FEAT Ethics (board oversight) and giving MRM its mandate. [verify: TORC naming is an illustrative institution; the layer's existence is the verified pattern from §5.1.]
2. **The RAI Program Office (center of excellence).** A small central team (2–3 FTE: one from MRM, one from Compliance, one architect — Jack's team supplies the architecture seat) owning: the principle set and standard, the risk-assessment templates, the tooling platform (the §7.4 stack), the training curriculum, and the inventory. It is the Singapore entity's counterpart of the Group's responsible-AI office, so the Group standard is *adopted, then Singapore-adapted* (FEAT section added) — never forked.
3. **The RAI Review Committee (the ethics board).** Chaired by the Chief Risk Officer or delegate; members: MRM, Compliance/Conduct, Legal, Data Protection Officer, an architect, and (for hard cases) an external academic. Triggers for mandatory review: credit or customer-facing decisions; personal-data-heavy use; GenAI external output; any use on the restricted list; any significant-adverse-impact classification. Tiering for volume: low-risk internal uses self-assess on a checklist (auto-approved, sampled by the program office); everything else goes to the committee, which decides **approve / approve-with-conditions / reject**, logged with reasons into the inventory entry. The committee's decisions are appealable and its rejection power is explicit — the two structural lessons from [§5.1](#51-the-governance-structures-ethics-boards-and-rai-committees).

**Policy documents (the §5.2 hierarchy, adapted):** (1) AI Principles (the §7.2 table, one page); (2) the RAI Standard — lifecycle requirements keyed to tier, modeled on the Microsoft Responsible AI Standard structure (goals → requirements → evidence), with the FEAT section and the EU-Act anticipation section; (3) the AI Risk Assessment Procedure — intake → classify (tier, EU-Act tier, FEAT relevance, materiality) → impact assessment (ISO-42001-style AI system impact assessment) → testing plan → sign-off → monitoring → incident; (4) the AI Incident Policy — definitions, declaration, investigation, regulator notification paths (EU Act GPAI; MAS where applicable), lessons-learned loop.

### 7.4 The Tooling Design

The tooling layer is a thin platform on the existing MLOps estate (cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) for the MRM side and [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the gateway side):

| Tool | Deployment in the Bank | Evidence produced |
|---|---|---|
| **AIF360** | Standalone audit container, run by MRM/validation on credit, fraud, and screening models at validation and on revalidation | Bias-audit reports: disparate impact, statistical parity, equal-opportunity differences per protected group; mitigation selection when thresholds trip |
| **Fairlearn** | Embedded in the MLOps training pipeline for continuous monitoring (thresholds on demographic-parity difference, equalized-odds difference per model) | Monitoring dashboards; drift-triggered re-audit tickets |
| **ART** | Security test suite in the CI pipeline for high-tier systems (evasion/Poisoning checks on classifiers; membership-inference on sensitive models) | Robustness test reports feeding the "secure & resilient" evidence and the gateway's protection requirements |
| **SHAP/LIME** | Validation workbench for every credit/decision model; SHAP primary, LIME spot-checks | Conceptual-soundness exhibits for SR 11-7; FEAT transparency responses; adverse-action reason source material |
| **Model cards** | Mandatory artifact in the model registry (Hugging Face Hub-style template, internal registry) — every model and every GenAI deployment ships one | The transparency artifact; feeds AI Verify process checks and EU-Act technical documentation |
| **Datasheets** | Mandatory for training/eval datasets in the data catalog; vendor datasets get datasheets at procurement | Data-provenance evidence; procurement review artifact; Article 10 data-governance support |
| **Gateway + guardrails** | The enterprise AI gateway enforces the technical controls (prompt injection defense, PII redaction, model access policy); GenAI guardrails add output filtering and grounding checks | Enforcement logs; guardrail test reports — cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) and [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) |
| **AI Verify / Moonshot** | Annual FEAT-readiness demonstration on flagship systems (voluntary, but the credible evidence pattern) | Publishable AI Verify-style reports — cross-ref [ai_verify_guide.md](ai_verify_guide.md) §5 |

**The evidence file.** Each inventory entry accumulates: impact assessment, model card, datasheets, bias-audit report, robustness report, committee decision, monitoring plan, incident log. That file is simultaneously: the FEAT answer, the SR 11-7 validation record, the ISO 42001 record (if/when certified), the EU-Act technical documentation skeleton, and the AI Verify evidence. One evidence program, five consumers — the cost-efficient pattern from the trust guide's lessons (§8.4 there).

**The GenAI-specific additions.** For the copilots and agents: grounding-and-citation checks as the practical explanation layer (token-level SHAP being too noisy — trust guide §4.4); red-team exercises per release (cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) §8); agent inventories with tool-permission and human-oversight design (cross-ref [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md)); and model-supply-chain documentation from procurement (GPAI documentation flows down under the EU Act).

### 7.5 The Lessons

Five lessons from the design exercise, in the voice of the series:

1. **The principles are the cheapest part and the least valuable without the rest.** The §7.2 table took a day to write; the governance, tooling, and evidence machinery took the program. A framework is a *system*, not a document — and the system is the part regulators and auditors actually examine.
2. **Map everything to FEAT (and the EU Act) from day one.** The single mapping table (§7.2) is what converts every future artifact into regulator-ready evidence. Building the mapping after the fact is twice the work and half as convincing.
3. **Tiering is the difference between a program and a committee-shaped bottleneck.** Auto-approval for low-risk self-assessed uses, committee review for the rest, explicit triggers — this is what makes 60 systems governable and GenAI's use-case volume survivable.
4. **Tooling is evidence, not ethics.** AIF360/Fairlearn/ART/SHAP produce numbers; the committee decides what the numbers mean, which criterion was declared, and whether residual disparity is justified by business necessity. The governance layer must be able to *override* the tooling's default thresholds — that is what accountability looks like in practice.
5. **The real deliverable is the evidence file, not the framework.** When the MAS relationship manager or the EU Act auditor asks, the Bank shows the inventory with its per-system artifacts — impact assessments, model cards, bias audits, committee decisions. That file is the framework; everything else is scaffolding. (This is the trust guide's lesson 5 in §8.4, re-applied at framework scale.)

### 7.6 The Rollout Plan: Twelve Months

Frameworks die in the gap between approval and adoption. The design above rolls out in four quarters, each with a hard deliverable:

| Phase | Months | Workstreams | Deliverable |
|---|---|---|---|
| **Q1 — Stand up** | 1–3 | Principles + negative list approved by TORC; RAI Program Office staffed (MRM + Compliance + architecture seat); committee charter and triggers; policy hierarchy drafted (principles → standard → risk assessment → incident); inventory baseline completed (all 60 systems tiered) | Approved framework + baseline inventory; committee operational |
| **Q2 — Tool** | 4–6 | RAI toolkit platform stood up (AIF360 + Fairlearn + ART + SHAP packaged as one service on MLOps); model-card and datasheet templates enforced in the registry; pilot on three high-tier systems (SME credit model, sanctions-screening classifier, trade-ops copilot); procurement questionnaire v1 issued to top-10 AI vendors | Toolkit live; three pilot evidence files complete; FEAT mapping table published |
| **Q3 — Embed** | 7–9 | All high/medium-tier systems get impact assessments and bias audits on the revalidation schedule; GenAI red-team program per release; gateway guardrail controls attached (cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)); EU-Act classification memos for all EU-entity systems; incident policy exercised in a tabletop drill | Inventory at 100% for high/medium tiers; first incident-drill report |
| **Q4 — Prove** | 10–12 | FEAT-readiness self-assessment with AI Verify-style testing on flagship systems (cross-ref [ai_verify_guide.md](ai_verify_guide.md)); quarterly RAI dashboard to TORC; MAS relationship-manager briefing pack assembled from the inventory evidence; ISO 42001 gap assessment (decision point for certification); year-two plan: monitoring expansion, agent governance, L3→L4 roadmap | Regulator-ready FEAT evidence pack; year-one RAI report |

**Resourcing reality check:** this is roughly 4–5 FTE-equivalents of incremental effort (program office 2–3, plus validation and platform time spread across MRM and the MLOps team) — the same order of magnitude as the trust guide's assurance program, and the two share most artifacts. The investment framing (RAI as risk-reduction with a defensible ROI story) is treated in [../management/business_case_development_guide.md](../management/business_case_development_guide.md); the shortcut version for a steering deck: *the EU Act and FEAT make the evidence mandatory; the framework is the cheapest way to produce it once.*

---

## 8. The Summary

### 8.1 The One-Page Summary

**Responsible AI is ethical AI made operational.** It is the discipline of turning AI-ethics values — fairness, safety, privacy, transparency, accountability, and the human-centric commitments — into principles, processes, tooling, governance, and *evidence*, across the full lifecycle. The lineage runs from philosophy (Asimov, Wiener, machine ethics) through the institutional layer (ACM, Asilomar 2017, IEEE 2016) to the corporate canon of **2018** — Microsoft's six principles and Responsible AI Standard (v2, June 2022), Google's seven AI Principles (June 2018, born of the Maven controversy, with its negative list), IBM's trust-and-transparency (augment humans; responsible data; open and transparent) — and then to the international instruments: **UNESCO's Recommendation on the Ethics of AI (adopted by acclamation by 193 member states, November 2021; four values, ten principles)** and **IEEE's Ethically Aligned Design (First Edition, March 2019; eight general principles feeding the P7000 standards series)**. The tooling makes it measurable: **AIF360** and **Fairlearn** (fairness), **ART** (adversarial robustness), **SHAP/LIME** (interpretability), **model cards** (Mitchell et al., FAT\* 2019) and **datasheets for datasets** (Gebru et al., 2018; CACM 2021) (documentation). Implementation is a known pattern: layered governance (board appetite → RAI office → review committee with real veto power), a policy hierarchy (principles → standard → risk assessment → incidents), procurement with RAI questionnaires and clauses, and one inventory that carries every artifact. In banking it lands on **MAS FEAT** (fairness, ethics, accountability, transparency — 12 Nov 2018) and fair-lending law (ECOA/Reg B, EU AI Act Art. 10): RAI is the evidence discipline that keeps supervised AI legal. The regulatory frameworks themselves — NIST AI RMF, the EU AI Act, ISO 42001, OECD — are the enforcement layer, covered in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2–§3; this guide's job was the layer underneath: where the values come from, and how they get built.

### 8.2 The Final Word: Ethical by Design

The corporate principles of 2018 were pre-commitments made in a vacuum of law; in 2026 they are the ancestors of enforceable obligations. That inversion is the strongest argument for the discipline: **the RAI framework you build now is not a compliance cost, it is the design constraint that makes AI deployable at all** — the ethical-by-design equivalent of security-by-design ([security_by_design_guide.md](security_by_design_guide.md)) and of the trust-by-evidence pattern of the trust guide. A bank that adopts the five-value principle set, stands up real governance with a committee that can say no, runs the tooling that produces the numbers, and keeps one evidence file per system is not "doing ethics" — it is engineering trust into the product, in the same way it engineers reliability into payments. The final word, then: **responsible AI is not a policy to be adopted; it is a property to be designed — and in a supervised industry, the design is the only defensible posture. Ethical by design, evidenced by default, governed by humans.**

**The RAI deploy checklist** (the whole guide in ten checks — if a program can answer all ten, it is real):

1. **Principles published** — the five-value set (+ firm-specific additions) approved at executive level, with a negative list.
2. **Governance standing** — board/executive appetite statement, a central RAI office, and a review committee with documented veto power.
3. **Policies in force** — principles → standard → risk-assessment procedure → incident policy, versioned like engineering standards.
4. **Inventory complete** — every AI system tiered in one registry that reconciles MRM, platform, and compliance lists.
5. **Evidence per system** — impact assessment, model card, datasheets, bias audit, robustness report, committee decision, monitoring plan.
6. **Tooling live** — fairness (AIF360/Fairlearn), robustness (ART), interpretability (SHAP/LIME) packaged as one platform service.
7. **Fairness declared** — a stated criterion per decision system, with thresholds and mitigation policy (you cannot have every fairness notion at once — you choose and justify).
8. **Procurement covered** — RAI questionnaire + contract clauses on every AI vendor, model-supply-chain documentation flowing down.
9. **Regulator-mapped** — the FEAT mapping table maintained, EU-Act classification memos current, AI Verify-style testing scheduled for flagships.
10. **Learning loop** — incidents investigated and fed back into the standard; the committee's decisions (including rejections) logged and reported quarterly.

---

## Glossary

- **Responsible AI** — the practice of designing, developing, deploying, and operating AI so that it aligns with ethical values (fairness, safety, privacy, transparency, accountability, human dignity) and so the organization can demonstrate that alignment with documented evidence; "ethical AI made operational".
- **RAI** — shorthand for Responsible AI; also the name of the discipline/program (a "RAI program", "RAI framework").
- **Ethical AI** — the philosophical and normative layer: what AI *should* do, whose values apply; the lineage (machine ethics, Asilomar Principles 2017, ACM 2017) from which RAI descends. Ethics supplies the values; RAI supplies the practice.
- **Microsoft (RAI)** — publisher (2018) of six responsible-AI principles: fairness, reliability and safety, privacy and security, inclusiveness, transparency, accountability.
- **Principles** — the high-level value commitments (one page) that anchor an RAI program; distinct from policies (operating requirements) and standards (engineering requirements).
- **Responsible AI Standard** — Microsoft's internal engineering standard operationalizing its six principles into goals and requirements across the lifecycle; version 2 published publicly June 2022; aligned with the NIST AI RMF functions.
- **Google (AI Principles)** — seven objectives published 7 June 2018 (socially beneficial; avoid unfair bias; safety; accountable to people; privacy design; scientific excellence; accordant uses) plus a list of applications Google will not pursue.
- **AI Principles** — Google's name for its 2018 RAI commitment; generic term for corporate principle sets.
- **IBM (trust and transparency)** — IBM's RAI posture: original 2018 principles (augment human intelligence; data and insights belong to their creator; transparent and explainable), evolved into three Principles of Trust (augment human capability; responsible data governance; open and transparent) with Pillars of Trustworthy AI and Impact Dimensions.
- **Trust** — justified confidence that a system behaves as intended; the outcome RAI produces evidence for (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §1.1).
- **Transparency** — the property that a system's existence, capabilities, limitations, data provenance, and decision logic are disclosed at the right level; an RAI principle (Microsoft #5, Google-related, UNESCO #7, FEAT #4) and a trust dimension.
- **UNESCO** — United Nations Educational, Scientific and Cultural Organization; adopted the Recommendation on the Ethics of AI in November 2021.
- **Recommendation (UNESCO)** — the first global AI-ethics standard: adopted by acclamation by 193 member states, 23 November 2021; four values + ten principles; non-binding but the consensus reference.
- **Ethics** — the normative study of right/wrong conduct; in AI, the values layer that RAI operationalizes; in FEAT, one of the four principles (alignment with institutional ethical standards, board oversight).
- **IEEE** — Institute of Electrical and Electronics Engineers; publisher (via the IEEE Standards Association) of Ethically Aligned Design.
- **Ethically Aligned Design (EAD)** — IEEE's consensus vision (First Edition, 25 March 2019) prioritizing human well-being in autonomous/intelligent systems; eight General Principles (human rights, well-being, data agency, effectiveness, transparency, accountability, awareness of misuse, competence); feeds the IEEE P7000 standards series.
- **AI Fairness 360 (AIF360)** — IBM Research's open-source fairness toolkit (2018; arXiv:1810.01943): 70+ fairness metrics and pre/in/post-processing bias-mitigation algorithms; Apache-2.0; Linux Foundation Trusted-AI umbrella.
- **Fairlearn** — Microsoft Research-originated open-source fairness toolkit (open-sourced 2020; JMLR 2023): metrics, reductions-based mitigations (ExponentiatedGradient, GridSearch), ThresholdOptimizer, dashboard; sociotechnical framing; credit-loan use case.
- **ART** — the Adversarial Robustness Toolbox: IBM's open-source ML-security library (2018; arXiv:1807.01069; LF AI & Data since July 2020) covering attacks/defenses/detectors for evasion, poisoning, extraction, and inference.
- **Adversarial robustness** — a model's resilience to adversarial manipulation (crafted inputs, poisoning, extraction, inference); measured with ART-style tooling; the "secure & resilient" trust characteristic (cross-ref [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md)).
- **SHAP** — SHapley Additive exPlanations (Lundberg & Lee, 2017): game-theoretic feature attributions satisfying local accuracy/missingness/consistency axioms; the banking workhorse for explainability (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.4).
- **LIME** — Local Interpretable Model-agnostic Explanations (Ribeiro et al., 2016): local surrogate-model explanations; fast but less faithful than SHAP.
- **Model cards** — structured model documentation (Mitchell et al., FAT\* 2019): benchmark evaluation across groups/conditions, intended use, factors, metrics, ethical considerations, caveats; the transparency artifact (Hugging Face Hub feature; EU-Act/ISO record ancestor).
- **Datasheets** — dataset documentation (Gebru et al., 2018; CACM 2021): motivation, composition, collection process, recommended uses, limitations; the input-side counterpart of model cards.
- **Gebru** — Timnit Gebru, first author of *Datasheets for Datasets* (with Morgenstern, Vecchione, Vaughan, Wallach, Daumé III, Crawford).
- **Mitchell** — Margaret Mitchell, first author of *Model Cards for Model Reporting* (with Wu, Zaldivar, Barnes, Vasserman, Hutchinson, Spitzer, Raji, Gebru).
- **Governance** — the structures and decision rights around AI: board/executive appetite, RAI office, review committees, policies, and the accountability they enforce.
- **Ethics board** — the review body (RAI committee) that adjudicates sensitive AI use cases with documented decisions; must have real veto power and tiered triage to scale.
- **Procurement** — the RAI control point for bought AI: vendor questionnaires, contract clauses (data-use restrictions, documentation/audit rights, incident SLAs, liability), model-supply-chain due diligence; procurement buys evidence, not absolution.
- **MAS** — Monetary Authority of Singapore; the financial-sector regulator; issuer of FEAT.
- **FEAT** — Fairness, Ethics, Accountability and Transparency: MAS's 2018 principles for AI/data-analytics use in Singapore financial services (12 Nov 2018), with Veritas assessment methodologies; voluntary-but-authoritative.
- **Fair lending** — the legal framework against discriminatory credit decisions: US ECOA (1974)/Regulation B (disparate impact, adverse-action reasons), EU AI Act Article 10 (representative, bias-examined training data), MAS FEAT fairness; enforced through 80%-rule screens, declared fairness criteria, reason engines, and monitoring.
- **Implementation** — the four levers that make RAI real: governance structures, policies/risk assessment, procurement, and model-inventory integration.
- **AIDA** — AI and Data Analytics: MAS FEAT's term for the systems it governs ("the use of AI and data analytics").
- **Veritas** — the MAS-industry consortium (with major Singapore banks) that develops FEAT assessment methodologies (Fairness Assessment Methodology first, 2020); FEAT's operationalization layer (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.4).
- **MRM** — Model Risk Management: the SR 11-7-based bank discipline (inventory, validation, documentation) into which RAI artifacts integrate as evidence (cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md)).
- **Model registry / inventory** — the single source of truth for AI systems; RAI artifacts (impact assessments, model cards, test reports, committee decisions) attach to each entry.
- **Negative list** — the Google-pattern list of prohibited AI uses that makes principles enforceable by giving the review body a defined reject path.

---

## Appendix: Verification Log (August 2026)

| Fact | Status | Source |
|---|---|---|
| Microsoft six principles (fairness; reliability & safety; privacy & security; inclusiveness; transparency; accountability) | **Verified** | microsoft.com/en-us/ai/responsible-ai; learn.microsoft.com (Azure ML responsible-AI doc) |
| Microsoft Responsible AI Standard v2 (public, June 2022) | **Verified** (standard document publicly circulated; date widely documented) | Microsoft "Responsible AI Standard v2 — General Requirements" (blogs.microsoft.com, June 2022); public copies |
| Google AI Principles: seven objectives, published 7 June 2018 | **Verified** | developers.google.com/ai/responsibility/principles; 9to5google.com (7 June 2018); google-research PRINCIPLES.md |
| IBM current Principles of Trust (3) + Pillars of Trustworthy AI + Impact Dimensions; Responsible Technology Board | **Verified** | ibm.com/impact/ai-ethics (August 2026) |
| IBM original 2018 trust-and-transparency principles (3) | **Verified** (widely documented; exact September 2018 date flagged) | IBM CEO statement, Sept 2018; secondary press coverage |
| UNESCO Recommendation: 193 member states, acclamation, 41st General Conference, Nov 2021; four values + ten principles | **Verified** | unesco.org/en/artificial-intelligence/recommendation-ethics; unesdoc.unesco.org (Recommendation text) |
| IEEE EAD1e: First Edition 25 March 2019; eight General Principles (verbatim) | **Verified** | standards.ieee.org EAD1e PDF (eyeballed); IEEE RAS release |
| AIF360: IBM open-source fairness toolkit; metrics + mitigations; Apache-2.0 | **Verified** | github.com/Trusted-AI/AIF360; arXiv:1810.01943 (companion paper, 2018) |
| Fairlearn: MSR-originated; open-sourced 2020; JMLR 2023 paper; credit-loan use case | **Verified** | fairlearn.org; microsoft.com/en-us/research (Fairlearn publication); JMLR 24(2023) 23-0389 |
| ART: IBM-originated; evasion/poisoning/extraction/inference; LF AI & Data since July 2020 | **Verified** | github.com/Trusted-AI/adversarial-robustness-toolbox; adversarial-robustness-toolbox.org; arXiv:1807.01069 |
| Model Cards for Model Reporting: Mitchell et al.; FAT\* 2019 (PMLR Vol. 81); arXiv:1810.03993 | **Verified** | arXiv record; dl.acm.org; research.google publication page |
| Datasheets for Datasets: Gebru et al.; arXiv:1803.09010 (23 Mar 2018); CACM December 2021 | **Verified** | arXiv record (submission history + comments) |
| SHAP (Lundberg & Lee 2017); LIME (Ribeiro et al. 2016) | **Verified** (canonical papers) | Cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.4 and its verification log |
| MAS FEAT (12 Nov 2018, four principles) | **Verified** | Cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.4 (verified there) |
| ECOA/Reg B, EU AI Act Art. 10, FEAT fairness (fair-lending facts) | **Verified** | Cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) §6.3 (verified there) |
| Salesforce Responsible AI Principles (five themes) | **[verify: exact wording]** | salesforce.com/company/responsible-ai (blocked to automated extraction at writing); themes stable across versions |
| Accenture RAI assurance pillars | **[verify: exact list varies by offering]** | Accenture responsible-AI practice materials |
| Industry RAI governance structures (boards, offices, committees); procurement questionnaire pattern | **Verified pattern** (consistent across Microsoft/IBM/Google/Salesforce primary sources; no single standard) | Company primary sources as cited in §5.1–§5.3 |

## Primary Sources and Further Reading

- Microsoft — *Responsible AI Principles and Approach* (microsoft.com/en-us/ai/principles-and-approach); *Responsible AI: Ethical policies and practices* (microsoft.com/en-us/ai/responsible-ai); *Microsoft Responsible AI Standard v2 — General Requirements* (June 2022).
- Google — *Artificial Intelligence at Google: Our Principles* (June 2018; developers.google.com/ai/responsibility/principles); 9to5google coverage of the Maven-driven publication (7 June 2018).
- IBM — *IBM's Approach to Responsible Technology* (ibm.com/impact/ai-ethics); GitHub: `Trusted-AI/AIF360`, `Trusted-AI/adversarial-robustness-toolbox`.
- Fairlearn — fairlearn.org; Bird et al., *Fairlearn: A toolkit for assessing and improving fairness in AI* (2020); Weerts et al., *Fairlearn: Assessing and Improving Fairness of AI Systems*, JMLR 24 (2023) 23-0389.
- UNESCO — *Recommendation on the Ethics of Artificial Intelligence* (adopted 23 November 2021; unesdoc.unesco.org record 038045).
- IEEE — *Ethically Aligned Design, First Edition* (IEEE Global Initiative on Ethics of Autonomous and Intelligent Systems, 25 March 2019; standards.ieee.org).
- Mitchell, M. et al., *Model Cards for Model Reporting*, FAT\* 2019 / PMLR Vol. 81 (arXiv:1810.03993).
- Gebru, T. et al., *Datasheets for Datasets* (arXiv:1803.09010, 2018; Communications of the ACM, December 2021).
- Bellamy, R. et al., *AI Fairness 360: An Extensible Toolkit for Detecting and Mitigating Algorithmic Bias* (arXiv:1810.01943, 2018); *Adversarial Robustness Toolbox v1.0.0* (arXiv:1807.01069, 2018).
- Series companions: [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) (§2 frameworks, §3 Singapore/FEAT, §4 methods), [ai_verify_guide.md](ai_verify_guide.md), [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md), [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md), [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md), [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md), [../banking/regtech_guide.md](../banking/regtech_guide.md), [../management/business_case_development_guide.md](../management/business_case_development_guide.md).

---

*End of guide — responsible AI is ethical by design, evidenced by default, and governed by humans.*
