# AI Trust Assessments: The Definitive Guide

> **The AI-assurance discipline deep-dive — how to define AI trust, map it to the major frameworks (NIST AI RMF, EU AI Act, ISO 42001, OECD), apply the Singapore angle (Model Framework, AI Verify, FEAT), run model audits, red-teaming, bias audits and explainability checks, evaluate against trust benchmarks, and package the whole thing as bank-grade evidence — with a Cymbal Bank GenAI worked example.**

**Author:** Jack Liu Shurui  
**Role:** Solution Architect, Cymbal Bank  
**Audience:** Solution Architects, AI/ML Platform Teams, Model Risk Management (MRM) Teams, Compliance and Audit, CTO Office  
**Version:** 1.0  
**Last Updated:** August 2026  
**Series:** AI & Data Platform Guides — this is the **dedicated AI-trust/assurance treatment**. The enterprise AI governance and gateway landscape lives in [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md); the adversarial/security side lives in [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md); the banking model-risk deep-dive lives in [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md).

---

## How to Read This Guide

- **Architects and platform teams** — start with [§1](#1-the-trust-overview) for the dimension taxonomy, [§7](#7-the-assessment-process) for the process, and [§8](#8-the-worked-example-a-genai-trust-assessment) for the shape of a real assessment; pair with [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the platform controls the assessment will rely on.
- **Model risk / validation teams** — [§6](#6-the-banking-angle) maps everything to SR 11-7; [§4](#4-the-assessment-methods) and [§5](#5-the-evaluation) are the evidence-producing methods; pair with [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md).
- **Compliance / governance officers** — [§2](#2-the-frameworks) (the frameworks), [§3](#3-the-singapore-angle) (the local expectations), and the glossary are the core reading.
- **Anyone defending a GenAI budget** — [§9](#9-the-summary) is the one-pager; the assurance-cost framing lives in [../management/business_case_development_guide.md](../management/business_case_development_guide.md).

**Scope note on verification:** framework dates, issuers, and provision-level facts below were checked against primary sources and reputable references (NIST AI 100-1, Regulation (EU) 2024/1689, ISO/IEC 42001:2023, the OECD Council Recommendation on AI, IMDA/PDPC/MAS releases, Federal Reserve/OCC publications) in August 2026. Where a fact could not be fully verified it is explicitly flagged **[verify]** rather than asserted — the same discipline this guide recommends for assessments. The Appendix records the verification log.

---

## Table of Contents

[How to Read This Guide](#how-to-read-this-guide)
1. [The Trust Overview](#1-the-trust-overview)
   - 1.1 What "AI Trust" Means
   - 1.2 The Five Trust Dimensions
   - 1.3 The Overview Table
2. [The Frameworks](#2-the-frameworks)
   - 2.1 NIST AI RMF 1.0 (January 2023)
   - 2.2 The EU AI Act (2024)
   - 2.3 ISO/IEC 42001 (December 2023)
   - 2.4 The OECD AI Principles
   - 2.5 The Framework Table
   - 2.6 How the Frameworks Compose
3. [The Singapore Angle](#3-the-singapore-angle)
   - 3.1 Model AI Governance Framework (2019–2020)
   - 3.2 AI Verify (2022)
   - 3.3 The Generative-AI Framework (2024)
   - 3.4 MAS FEAT Principles (2018)
   - 3.5 The SG Table
   - 3.6 The Wider Singapore Context
4. [The Assessment Methods](#4-the-assessment-methods)
   - 4.1 Model Audits
   - 4.2 Red-Teaming
   - 4.3 Bias Audits
   - 4.4 Explainability (XAI: SHAP / LIME)
   - 4.5 The Methods Table
   - 4.6 Choosing and Combining Methods
5. [The Evaluation](#5-the-evaluation)
   - 5.1 Trust and Safety Benchmarks
   - 5.2 The Eval Table
   - 5.3 Building the Trust Scorecard
6. [The Banking Angle](#6-the-banking-angle)
   - 6.1 SR 11-7 / OCC 2011-12 Model Risk Management
   - 6.2 BCBS 239-Adjacent Data Governance
   - 6.3 The Banking Table
7. [The Assessment Process](#7-the-assessment-process)
   - 7.1 The AI Inventory
   - 7.2 The Risk Register
   - 7.3 The Process Table
   - 7.4 Roles, Ownership, and the Three Lines of Defence
8. [The Worked Example: A GenAI Trust Assessment](#8-the-worked-example-a-genai-trust-assessment)
   - 8.1 The Scenario: A Cymbal Bank GenAI Deployment
   - 8.2 The NIST-RMF-Mapped Assessment Design
   - 8.3 Findings, Evidence, and Actions
   - 8.4 The Lessons
9. [The Summary](#9-the-summary)
   - 9.1 The One-Page Summary
   - 9.2 The Trust Case (Final Word)

[Glossary](#glossary)

---

## 1. The Trust Overview

### 1.1 What "AI Trust" Means

**AI trust** is the justified confidence that an AI system will behave as intended across its lifecycle — that its outputs are reliable, that it will not cause harm, that it treats people fairly, that its behaviour can be understood and explained, and that someone can be held to account when it goes wrong. Trust is not a property of the model alone; it is a **sociotechnical property of the model, the data, the deployment environment, the human processes around it, and the evidence produced about all of those things**.

Three distinctions matter before anything else:

- **Trust vs. assurance.** Trust is the outcome (stakeholders are willing to rely on the system). Assurance is the *process and the evidence* that produces and sustains that outcome: documented assessments, test results, sign-offs, monitoring reports. You cannot be ordered to be trusted; you can be ordered to produce assurance, and good assurance earns trust.
- **Subjective vs. objective trust.** End-users trust based on experience; regulators and auditors trust based on *verifiable evidence*. A trust assessment serves the second kind — it produces artifacts (risk registers, test reports, audit trails) that reviewers can check.
- **Static vs. dynamic trust.** A model that passed assessment in January can drift, be re-trained, or be re-purposed by June. Trust assessment is therefore a **lifecycle discipline**, not a pre-launch gate — which is why every major framework below structures itself around the lifecycle.

In regulated contexts this is not abstract: for a bank, an AI trust assessment is what turns "we think the model is fine" into "here is the evidence the model is fine, and here is who signed it" — the exact currency of model risk management (see [§6](#6-the-banking-angle)) and of the Singapore regulator's expectations (see [§3](#3-the-singapore-angle)).

### 1.2 The Five Trust Dimensions

The assurance literature converges, with different names and groupings, on a core set of trust dimensions. The NIST AI RMF 1.0 expresses these as seven "characteristics of trustworthy AI" (valid & reliable, safe, secure & resilient, accountable & transparent, explainable & interpretable, privacy-enhanced, fair — with harmful bias managed). The OECD AI Principles compress them into five values-based principles. For a working assessment taxonomy, the five dimensions below cover the practical ground; NIST's seven map onto them cleanly (security/privacy fold under safety and transparency).

**1. Reliability (validity & robustness).** The system does what it claims, consistently, under expected and unexpected conditions. Covers accuracy, precision, consistency across inputs, graceful degradation under distribution shift, and stability over time. *Assessment:* golden-set accuracy, holdout evaluation, adversarial robustness tests, drift monitoring, stress inputs (see [§4](#4-the-assessment-methods) and [§5](#5-the-evaluation)).

**2. Safety.** The system does not cause physical, financial, reputational, or psychological harm — including harm from misuse. For generative AI this includes harmful content, hallucination-driven bad advice, and unsafe actions taken by agents. NIST splits this into *safe* and *secure & resilient*; security (see [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md)) is the precondition that safety often depends on. *Assessment:* safety benchmarks, red-teaming, harm-classification of outputs, guardrail penetration testing.

**3. Fairness.** The system does not systematically disadvantage individuals or groups on protected grounds (race, gender, age, religion, disability, and jurisdiction-specific categories), and harmful bias is detected and managed. Fairness is plural: several mathematical definitions conflict, so an assessment must state *which* fairness criterion is being claimed. *Assessment:* bias audits with group fairness metrics (see [§4.3](#43-bias-audits)).

**4. Transparency.** The system's existence, capabilities, limitations, data provenance, and decision logic are disclosed to the people who need to know — users, affected persons, regulators — at the right level of detail. For GenAI this includes model cards, data cards, disclosure that an AI is being used, and labeling of AI-generated content (a direct EU AI Act limited-risk obligation). *Assessment:* documentation reviews, model cards, explainability output (see [§4.4](#44-explainability-xai-shap--lime)), disclosure checklists.

**5. Accountability.** Ownership is clear: roles, sign-offs, escalation paths, and audit trails exist, and adverse outcomes can be traced to decisions and remediated. This is where governance (NIST's Govern) meets the model lifecycle. *Assessment:* governance-process audits — RACI, decision logs, incident response drills, internal audit findings.

### 1.3 The Overview Table

| Dimension | Meaning | Assessment |
|---|---|---|
| **Reliability** | Correct, consistent, stable outputs; graceful failure under shift | Golden-set accuracy; holdout/backtest; robustness tests; drift and quality monitoring; stress inputs |
| **Safety** | No harm to people, money, reputation — including misuse | Safety benchmarks; red-team reports; content-moderation coverage; guardrail tests; incident history |
| **Fairness** | No systematic disadvantage of protected groups; stated fairness criterion | Bias audits; group fairness metrics (demographic parity, equal opportunity, calibration); disparate-impact analysis |
| **Transparency** | Existence, limits, data provenance, and logic are disclosed appropriately | Model/data cards; explainability (SHAP/LIME); user disclosures; AI-generated-content labeling; documentation review |
| **Accountability** | Clear ownership, sign-offs, escalation, audit trails | Governance review; RACI checks; decision logs; incident-response drill; MRM inventory inclusion |
| *Cross-cutting:* **Security** (NIST) | Resilience to adversarial manipulation, exfiltration, prompt injection | Adversarial testing; red-teaming; gateway controls (see [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) |

> **Note on the dimension set:** NIST's trustworthy-AI characteristics (valid & reliable; safe; secure & resilient; accountable & transparent; explainable & interpretable; privacy-enhanced; fair) are the reference taxonomy — the five above are an assessment-oriented grouping, with security and privacy handled as cross-cutting concerns. This is a framing choice, not a change of substance.

---

## 2. The Frameworks

### 2.1 NIST AI RMF 1.0 (January 2023)

**Verified:** The National Institute of Standards and Technology published the AI Risk Management Framework (AI RMF 1.0) as NIST AI 100-1 **in January 2023**, fulfilling a directive in the National Artificial Intelligence Initiative Act of 2020 (P.L. 116-283). It is explicitly **voluntary, rights-preserving, and sociotechnical** — it treats AI risk as arising from the interaction of the technical system with the social and organizational context, not from the model alone.

**The four functions — Govern, Map, Measure, Manage.** The AI RMF's core organizes AI risk-management activity into four functions (subdivided into categories and subcategories):

- **Govern** — the cross-cutting function that "informs and is infused throughout" the other three. Culture, policies, processes, roles and responsibilities, transparency commitments, stakeholder engagement, and accountability structures. In practice: an AI governance committee, a use-case intake policy, an AI risk appetite statement, inventory ownership.
- **Map** — understand the context: what the system is used for, who is affected, what could go wrong, what legal/regulatory context applies, what the risk tolerance is. Produces the risk register and the impact analysis.
- **Measure** — assess the risks identified in Map using quantitative and qualitative methods: tests, metrics, evaluations, benchmarks, audits. This is where the assessment methods of [§4](#4-the-assessment-methods) and [§5](#5-the-evaluation) plug in.
- **Manage** — decide and act: prioritize risks, implement controls and mitigations, plan response, and monitor continuously. Feeds back into Govern (the loop is deliberate).

The framework also defines **profiles** (a "current profile" describing the organization's actual state vs. a "target profile" describing desired risk-management posture) and supplies crosswalks to other standards — a mechanism we reuse in the worked example ([§8](#8-the-worked-example-a-genai-trust-assessment)). The RMF is voluntary by design; but note that the EU AI Act's high-risk technical documentation and the US Executive Order on Safe, Secure, and Trustworthy AI (October 2023) both gesture at RMF-aligned practices, and many enterprises adopt the RMF as the *de facto* internal standard precisely because it is framework-neutral.

**The seven characteristics of trustworthy AI.** The RMF's working definition of trust is expressed as seven characteristics that the functions are meant to cultivate (AI RMF 1.0, §2):

| Characteristic | Meaning in the RMF |
|---|---|
| **Valid & reliable** | The system does what it claims; valid in the statistical sense, reliable across conditions |
| **Safe** | No unreasonable harm to people, organizations, or society |
| **Secure & resilient** | Resistant to attack and able to recover; adversarial-ML concerns |
| **Accountable & transparent** | Governance, oversight, disclosure; someone answerable for outcomes |
| **Explainable & interpretable** | Behavior can be understood by relevant audiences at appropriate depth |
| **Privacy-enhanced** | Privacy protections honored through the lifecycle |
| **Fair** | Harmful bias managed; equitable treatment |

These map to the five working dimensions of [§1](#1-the-trust-overview) (security folds under safety; privacy is a cross-cutting control) — writing that mapping down is the first thing a NIST-aligned assessment does.

**The categories under the functions.** The core is structured functions → categories → subcategories: under **Govern**, categories cover governance culture, risk-management integration, roles/resources, and transparency commitments; under **Map**, context, risk identification, and impact assessment; under **Measure**, metric selection, testing approaches, and feedback; under **Manage**, prioritization, controls, and monitoring. Each subcategory carries suggested actions — the practical checklist the worked example ([§8](#8-the-worked-example-a-genai-trust-assessment)) draws from. (The exact category/subcategory numbering is versioned in NIST AI 100-1; cite that document rather than a summary in formal artifacts.)

**Verified companion:** NIST AI 100-2 E2023, *Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations*, published **March 2024**, is the security companion — see [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) for the attack taxonomy it establishes (evasion, poisoning, extraction, inference).

### 2.2 The EU AI Act (2024)

**Verified:** Regulation (EU) 2024/1689 — the first comprehensive horizontal AI law — was adopted in **2024** and **entered into force on 1 August 2024**. It applies in stages: the risk-based obligations phase in between **February 2025 and August 2027**. Penalties scale with the tier: up to **€35M or 7% of worldwide annual turnover** for prohibited practices, €15M or 3% for most other breaches (and €7.5M or 1.5% for supplying incorrect information to authorities) — the exact figures vary by member-state adjustment, so treat them as ceilings.

**The four risk tiers:**

| Tier | Definition | Obligations (summary) |
|---|---|---|
| **Unacceptable risk** | Practices prohibited outright (e.g., social scoring by public authorities, manipulative/subliminal techniques exploiting vulnerabilities, and — with narrow law-enforcement exceptions — real-time remote biometric identification in public spaces) | **Prohibited.** Applies from 2 February 2025. |
| **High risk** | Systems in Annex III areas (biometric ID, critical infrastructure, education, **employment, access to essential services including creditworthiness assessment** — the banking-relevant bucket — law enforcement, migration, justice) or safety components of Annex I regulated products | Full regime: risk management system; data governance; technical documentation; record-keeping/logging; transparency to deployers; **human oversight**; accuracy, robustness and cybersecurity; conformity assessment; CE marking; EU database registration. High-risk obligations under Annex III apply from **2 August 2026** (Annex I products: 2 August 2027). |
| **Limited risk** | Systems with specific transparency duties (chatbots that must disclose they are AI, deepfake/AI-generated-content labeling) | Transparency duties; applies largely from 2 February 2025 with content-transparency rules phased in. |
| **Minimal risk** | Everything else | No mandatory obligations; voluntary codes of conduct encouraged. |

**The GPAI provisions.** The Act's **general-purpose AI (GPAI)** chapter (Articles 51–56) regulates the models underneath the applications: providers of GPAI models must publish technical documentation, training-data summaries, and a copyright-compliance policy; models trained above a compute threshold (the Commission's delegated act sets the initial presumption at **~10²⁵ FLOPs**) are additionally presumed to pose **systemic risk** and must run model evaluations, adversarial testing, incident reporting, and cybersecurity protections, with oversight from the new European AI Office. The GPAI obligations apply from **2 August 2025**, with codes of practice drafted through 2025 to operationalize them. For an enterprise architect the practical consequences: **trace your model supply chain** (which GPAI model, which provider, which documentation obligations flow down), because the Act reaches deployers too — a bank deploying a GPAI-based assistant under high-risk conditions inherits documentation and oversight duties it cannot delegate away.

**The timeline (verified summary):**

| Date | Milestone |
|---|---|
| 1 August 2024 | Regulation in force |
| 2 February 2025 | Prohibitions on unacceptable risk; AI-literacy obligations |
| 2 August 2025 | GPAI obligations (incl. systemic-risk models) |
| 2 August 2026 | High-risk (Annex III) obligations apply |
| 2 August 2027 | High-risk (Annex I product-safety) obligations apply |

**Flag:** the Act's exact article-level mechanics (delegated acts on the FLOP threshold, harmonised standards, member-state penalties) continued to be finalized through 2025–2026; the tier structure, in-force date, and phased timeline above are stable and verified.

### 2.3 ISO/IEC 42001 (December 2023)

**Verified:** ISO/IEC 42001:2023, *Information technology — Artificial intelligence — Management system*, was **published in December 2023** by ISO/IEC JTC 1/SC 42, and is **the world's first certifiable AI management system (AIMS) standard**. It follows the familiar ISO "Annex SL" high-level structure (clauses 4–10: context, leadership, planning, support, operation, performance evaluation, improvement), so organizations already running ISO 9001/27001 can extend rather than rebuild.

What is AI-specific in 42001: an **AI policy**, an **AI risk assessment** process, an **AI system impact assessment** (AI's answer to privacy impact assessments — required before deployment and on significant change), lifecycle-orientated planning, and a control set (Annex A) covering data quality, documentation, transparency, incident management, and third-party AI. It is **certifiable** — an accredited body can certify an organization's AIMS — which makes it attractive to banks and vendors who want third-party-verifiable assurance. Companion standards extend it: ISO/IEC 42005 (AI risk assessment guidance) and ISO/IEC 42006 (requirements for bodies auditing AI management systems) — both flagged **[verify: edition dates]** as they continue to evolve.

**How it relates to the rest:** 42001 is the *management-system* frame (processes and evidence), the EU AI Act is the *regulatory* frame (obligations and penalties), and the NIST AI RMF is the *risk-management* frame (functions and activities). They are designed to interoperate — a common compliance strategy is RMF-style risk practice, 42001-style management system, Act-driven obligations where applicable.

### 2.4 The OECD AI Principles

**Verified:** The OECD AI Principles — the **first intergovernmental standard on AI** — were adopted as a Council Recommendation in **May 2019** and **updated in May 2024** to address generative and general-purpose AI (privacy, IP, safety, and information integrity). They are non-binding but foundational: the EU AI Act and much of the global principle-and-framework wave (including Singapore's, see [§3](#3-the-singapore-angle)) trace their lineage to them.

The Principles have two parts:

- **Five values-based principles for AI actors:** (1) inclusive growth, sustainable development and well-being; (2) human-centred values and fairness; (3) transparency and explainability; (4) robustness, security and safety; (5) accountability.
- **Five recommendations to governments:** (1) invest in AI research and development; (2) foster a digital ecosystem for AI; (3) shape an enabling policy environment; (4) build human capacity and prepare for labour-market transformation; (5) international co-operation for trustworthy AI.

The 2024 update adds sharper treatment of GPAI and generative AI — content provenance, system-level safety, incident reporting — and the OECD runs the AI incidents monitor (OECD.AI), which is a useful source of real-world failure data when you are populating a risk register ([§7.2](#72-the-risk-register)).

### 2.5 The Framework Table

| Framework | Origin | Focus |
|---|---|---|
| **NIST AI RMF 1.0** (NIST AI 100-1) | USA, NIST, **January 2023** (voluntary) | Risk-management process: **Govern–Map–Measure–Manage**; sociotechnical, lifecycle-wide; profiles; trustworthy-AI characteristics |
| **EU AI Act** (Reg. 2024/1689) | European Union, **2024** (in force 1 Aug 2024; phased to Aug 2027) | Hard regulation: four risk tiers (unacceptable/high/limited/minimal), high-risk obligations, GPAI + systemic-risk provisions, penalties up to €35M/7% |
| **ISO/IEC 42001** | ISO/IEC JTC 1/SC 42, **December 2023** | Certifiable AI management system (AIMS): policy, AI risk & impact assessment, controls, continuous improvement |
| **OECD AI Principles** | OECD, adopted **May 2019**, updated **May 2024** | First intergovernmental standard: five values-based principles + five government recommendations; basis for much later law |
| *(Companion)* **NIST AI 100-2** | NIST, March 2024 | Adversarial ML taxonomy (attacks & mitigations) — feeds Measure/Manage and red-teaming |

**How to choose:** none of these is "the" framework for a bank. The pragmatic stack is: **NIST AI RMF** as the internal risk-management process; **ISO 42001** as the management-system wrapper if certification is wanted; **EU AI Act** for anything touching EU markets (which, for a global bank with EU entities, is most things); **OECD** as the principles floor and external-comms language; **Singapore's frameworks** ([§3](#3-the-singapore-angle)) where the MAS/PDPC context applies — and for Jack's environment that last one is not optional, it is the local regulator's explicit posture.
### 2.6 How the Frameworks Compose

The four frameworks are not rivals; they operate at different layers, and a mature program uses all of them:

- **Principles (OECD, 2019/2024)** — the "why": values-based commitments that shape policy language and external communication.
- **Risk process (NIST AI RMF, 2023)** — the "how": a voluntary, sector-neutral process for identifying, measuring, and managing AI risk; the operating system most enterprises actually run on.
- **Management system (ISO/IEC 42001, 2023)** — the "wrapper": a certifiable, auditable management structure (policy, risk and impact assessments, controls, continual improvement) into which the RMF process slots as the risk machinery.
- **Regulation (EU AI Act, 2024)** — the "must": hard obligations keyed to risk tier, with penalties; the RMF/42001 artifacts are what a company shows the regulator.

The common skeleton — *context → risk assessment → measurement → controls → monitoring → documented evidence* — is the same under every name. An assessment that produces NIST-Map-style risk registers, ISO-42001-style impact assessments, and EU-Act-style technical documentation from one evidence program is the pattern banks actually deploy (see the banking table in [§6.3](#63-the-banking-table)).

---

## 3. The Singapore Angle

Singapore was an early mover on AI governance and — for a Singapore-based bank — the local frameworks are the most immediately relevant external expectations after internal policy. The sequence matters: principles first (2018–2019), then testing (2022), then generative-AI specifics (2024).

### 3.1 Model AI Governance Framework (2019–2020)

**Verified:** Singapore launched the **Model AI Governance Framework in January 2019 at the World Economic Forum in Davos** — **the world's first** such framework — and released a **Second Edition in January 2020**. It was developed by the **Infocomm Media Development Authority (IMDA)** with the **Personal Data Protection Commission (PDPC)**, and is voluntary.

Its shape: two guiding principles — (1) decisions made by or with AI should be **explainable, transparent and fair**; (2) AI systems should be **human-centric** — translated into four practical sections:

1. **Internal governance structures and measures** — board/management accountability, risk frameworks, staffing and training.
2. **Determining the AI decision-making model** — when to use AI, human involvement, model selection and data choices.
3. **Operations management** — model development, testing, monitoring, incident handling.
4. **Customer relationship management** — disclosure to consumers, human intervention and appeal channels.

The Second Edition added worked examples, sector notes, and companion guidance. This framework is the ancestor of AI Verify: the testing framework operationalizes "objective and verifiable" demonstration of exactly these governance commitments.

### 3.2 AI Verify (2022)

**Verified:** **A.I. Verify — the world's first AI governance testing framework and toolkit** — was announced by Singapore's Minister for Communications and Information Josephine Teo **at the World Economic Forum in January 2022** and **launched by IMDA and the PDPC in May 2022** as an open-source Minimum Viable Product. It lets organizations demonstrate responsible AI "in an objective and verifiable manner" by combining:

- **Process checks** — documentary review against governance practices (risk management, data governance, transparency documentation).
- **Technical tests** — automated tests against **11 internationally-recognised AI governance principles** (e.g., explainability, robustness, fairness, accountability, transparency, human agency, etc.), covering supervised-learning models.

The toolkit runs as a test harness around a model (dockerized), producing a report that companies can publish — the governance equivalent of a security scan. Governance moved to the **AI Verify Foundation (established June 2023)**, with industry members (including major cloud and AI vendors); the Foundation also runs **Project Moonshot**, a generative-AI evaluation sandbox, and the 2024 work extended AI Verify's testing concepts to GenAI (see 3.3).

**Flag on "MLTC":** some secondary literature summarises the original test criteria behind AI Verify as the "MLTC" (Model AI Governance Testing framework / Model Lifecycle Testing Checklist, depending on the source). The acronym itself **could not be independently verified against primary sources** for this guide — the primary record (IMDA/PDPC releases, the AI Verify Foundation) consistently describes the framework as testing against the 11 governance principles, without using "MLTC". Treat "MLTC" as an informal label, and cite the 11-principle testing scope in any formal document. (This is exactly the kind of claim a trust assessment would flag too — provenance matters.)

### 3.3 The Generative-AI Framework (2024)

**Verified:** Singapore published a **Proposed Model AI Governance Framework for Generative AI on 16 January 2024** for international views — identifying **9 dimensions** for a trusted GenAI ecosystem (accountability, data, trusted development, incident reporting, testing and assurance, security, content provenance, safety alignment, transparency) — and released the **final Model AI Governance Framework for Generative AI in May 2024**, complemented by an AI Verify for Generative AI testing catalogue and the GenAI Evaluation Sandbox. Key moves versus the traditional-AI framework: a shift from model-centric to **system-centric** accountability (the whole deployment, not just the model), explicit treatment of **content provenance** and AI-generated-content labeling, **safety alignment and red-teaming** as expected practice, and a lifecycle emphasis on testing and incident reporting. For a bank, the practical takeaways: document the full GenAI system (prompts, RAG, guardrails, gateway), keep a model inventory, and be ready to show testing evidence — all of which this guide's process ([§7](#7-the-assessment-process)) produces.

### 3.4 MAS FEAT Principles (2018)

**Verified:** The Monetary Authority of Singapore (MAS) published the **Principles to Promote Fairness, Ethics, Accountability and Transparency (FEAT) in the Use of Artificial Intelligence and Data Analytics (AIDA) in Singapore's Financial Sector on 12 November 2018** — non-binding but authoritative expectations for financial institutions, each principle carrying specific guidance items:

- **Fairness** — AIDA decisions should be sound and fair (e.g., data used should be relevant, and not systematically biased against protected groups).
- **Ethics** — AIDA should be used in line with the institution's ethical standards, with management accountability (e.g., board-level oversight of AI use).
- **Accountability** — institutions should be accountable for AIDA-driven decisions (e.g., clear allocation of responsibility, auditability, model risk management).
- **Transparency** — disclosure to consumers and timely explanation of decisions (e.g., telling customers when decisions are materially influenced by AI).

The accompanying **Veritas initiative** — a MAS-industry consortium (with major Singapore banks) launched to give FEAT an *assessment methodology* — published the **Fairness Assessment Methodology** first (2020) and extended into ethics, accountability and transparency assessment guides thereafter. [verify: exact Veritas phase dates vary by source; the fairness methodology is consistently dated 2020.]

FEAT matters doubly for Cymbal Bank-Singapore: it is the MAS articulation of what a financial institution's AI governance must demonstrate, and its fairness/transparency items map one-to-one onto the audit and explainability methods in [§4](#4-the-assessment-methods). Cross-ref [../banking/regtech_guide.md](../banking/regtech_guide.md) for the wider regulatory-compliance landscape.

### 3.5 The SG Table

| Framework | Year | Issuer | What it is | Why it matters for a bank |
|---|---|---|---|---|
| **Model AI Governance Framework** (1st ed. **Jan 2019** at Davos; 2nd ed. **Jan 2020**) | 2019–2020 | IMDA + PDPC | **World's first** national AI governance framework; voluntary; internal governance / model / operations / customer-relations sections | The baseline "what good governance looks like" reference for AI deployments in Singapore |
| **AI Verify** | **May 2022** (announced Jan 2022 at Davos) | IMDA + PDPC; AI Verify Foundation (Jun 2023) | **World's first** AI governance testing framework + open-source toolkit; process checks + technical tests across **11 governance principles** | Produces publishable, objective AI-governance evidence — the Singaporean "audit report" pattern |
| **MGF for GenAI** (proposed **Jan 2024**, final **May 2024**) | 2024 | IMDA + AI Verify Foundation | Model AI Governance Framework for Generative AI; 9 dimensions; system-centric accountability, content provenance, safety alignment, red-teaming | Defines expectations for GenAI copilots/assistants — directly applicable to §8's scenario |
| **MAS FEAT** | **12 Nov 2018** | MAS | Non-binding principles — **Fairness, Ethics, Accountability, Transparency** — for AI/data analytics in financial services; Veritas initiative adds assessment methodologies | The local financial regulator's explicit expectations; feeds MRM and conduct frameworks |

### 3.6 The Wider Singapore Context

Three adjacent strands complete the Singapore picture for a bank:

- **National AI Strategy 2.0 (December 2023)** — Singapore's national AI plan, which treats trusted-AI governance as a pillar of AI adoption; the strategy-level frame above the frameworks in 3.1–3.4.
- **PDPA and AI data use** — the PDPC's advisory guidance on using personal data in AI recommendation and decision systems **[verify: edition dates]** sets expectations (notification, consent, accountability) that intersect with any AI consuming customer or staff data; a trust assessment must include the privacy-legitimacy check as part of data lineage.
- **AI Verify Foundation and the GenAI evaluation sandbox (Project Moonshot)** — the Foundation (June 2023) stewards AI Verify and runs GenAI evaluation work; for a bank, referencing these toolchains is a credible way to demonstrate the "objective and verifiable" evidence that MAS-style governance expects.

None of these is a law in the EU-Act sense — Singapore's approach remains framework-and-testing-led rather than prohibitions-led — which makes the *evidence* from AI Verify-style testing and FEAT-aligned assessments the practical compliance currency. For the compliance landscape more broadly, cross-ref [../banking/regtech_guide.md](../banking/regtech_guide.md).

---

## 4. The Assessment Methods

Four families of method do most of the evidence-producing work. They are complementary: audits establish *process*, red-teaming probes *adversarial failure*, bias audits quantify *group-level unfairness*, and explainability makes *decisions legible*. An assessment that uses only one family is not yet an assessment.

### 4.1 Model Audits

A **model audit** is an independent, structured examination of a model and its context against a defined standard or criteria set — the AI analogue of a financial or security audit. It spans:

- **Data lineage and quality** — provenance, licenses, PII, representativeness, label quality (cross-ref the ML-data axis in [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md)).
- **Development practice** — training methodology, leakage checks, validation discipline, reproducibility.
- **Performance and robustness evidence** — holdout results, slicing, calibration, sensitivity analysis.
- **Documentation completeness** — model card, data card, technical documentation (directly reusable for EU AI Act high-risk documentation and ISO 42001 records).
- **Governance adherence** — who approved what, change control, versioning, sign-off chain.

Audits are *criteria-based*: the audit report states the criteria (e.g., "model card exists and covers intended use, limitations, and performance slices; NIST MAP categories M-1–M-5 addressed"), the evidence examined, findings with severity, and recommendations. The audit-as-code practice — machine-checkable control definitions and evidence pipelines — is covered in [audit_as_code_guide.md](audit_as_code_guide.md), and the internal-audit angle in [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md).

### 4.2 Red-Teaming

**Red-teaming** is adversarial testing: a dedicated team (or automated suite) deliberately attempts to make the system fail — to elicit harmful output, bypass guardrails, extract data, or make wrong decisions — so that weaknesses are found *before* real adversaries (or real accidents) find them. The term comes from military/security red-teaming; in AI it has become the standard practice for safety-critical and generative systems. Cross-ref [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) for the full attack taxonomy (evasion, poisoning, extraction, inference attacks) and [ai_llm/prompt_injection_guide.md](ai_llm/prompt_injection_guide.md) for the GenAI-specific injection vector.

For LLM-based systems, red-teaming covers:

- **Jailbreaks** — prompt manipulations that override safety training ("DAN", role-play, encoded instructions).
- **Prompt injection** — instructions hidden in user content, documents, or retrieved text that hijack the system (the single most important GenAI red-team finding for RAG deployments — see §8).
- **Harmful-content elicitation** — hate, violence, sexual content, dangerous instructions.
- **Data exfiltration and privacy probes** — attempts to extract training data or other users' context (cross-ref [copilot_data_cache_guide.md](copilot_data_cache_guide.md) for the caching/privacy angle).
- **Tool/agent misuse** — for agentic systems, attempts to make the agent take unsafe actions (cross-ref [agentic_workflows_guide.md](agentic_workflows_guide.md) on agent trust).

Method notes: define a **harms taxonomy and success criteria before testing** (so "failure" is measurable); run a mix of manual expert sessions and automated suites (e.g., harm-benchmark prompts); document every attempt, outcome, and severity; and re-run after every significant model or guardrail change. NIST's AI 100-2 and the OECD/MGF-GenAI guidance both treat red-teaming as expected practice for high-stakes and generative systems.

### 4.3 Bias Audits

A **bias audit** measures whether a system treats groups differently, using **fairness metrics** computed on test data sliced by protected attributes (where such data can lawfully be collected and handled — a real constraint in practice, and one that must itself be documented).

The standard fairness criteria (they conflict; an audit must state which is claimed and why):

- **Demographic parity (statistical parity)** — acceptance/positive rate equal across groups.
- **Equal opportunity** — true-positive rates (recall) equal across groups.
- **Equalized odds** — both false-positive and false-negative rates equal across groups.
- **Predictive rate parity / calibration** — precision (or score calibration) equal across groups.

**The core fairness metrics, in formula terms** (A = protected group; ŷ = predicted positive; y = true positive; P = proportion):

| Criterion | Definition | Formula |
|---|---|---|
| Demographic parity | Positive rate equal across groups | P(ŷ=1 \| A=a) = P(ŷ=1 \| A=b) |
| Equal opportunity | Recall equal across groups | P(ŷ=1 \| y=1, A=a) = P(ŷ=1 \| y=1, A=b) |
| Equalized odds | Recall *and* false-positive rate equal | P(ŷ=1 \| y, A=a) = P(ŷ=1 \| y, A=b) for y ∈ {0,1} |
| Predictive parity | Precision equal across groups | P(y=1 \| ŷ=1, A=a) = P(y=1 \| ŷ=1, A=b) |
| Calibration | Score-level calibration equal | P(y=1 \| s, A=a) = P(y=1 \| s, A=b) for scores s |

These cannot all hold at once (except in degenerate cases) — which is *why* an audit must state its criterion. In practice, audits report per-group rates plus the **disparate impact ratio** and the **group differences** (e.g., equal-opportunity difference, EOD) rather than pass/fail on a single formula.

Commonly reported quantities: the **disparate impact ratio** (ratio of group positive rates; the US "4/5ths rule" treats < 0.8 as a red flag), and the differences between groups in demographic-parity / equal-opportunity rates (DPD, EOD). For LLMs, bias audits also include **stereotype-elicitation tests** (e.g., benchmark datasets that probe occupational or demographic stereotypes), toxicity-by-group analysis, and counterfactual input perturbation (flip a name/gender marker, measure output change). Bias-audit results feed both fairness (dimension 3) and the EU AI Act's non-discrimination expectations and FEAT's fairness principle. Where the fairness-taxonomy details matter, cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md).

### 4.4 Explainability (XAI: SHAP / LIME)

**Explainable AI (XAI)** produces human-understandable accounts of model behavior — a prerequisite for the transparency and accountability dimensions, for FEAT's transparency expectations, and for human oversight duties under the EU AI Act. Methods divide into *intrinsic* (interpretable-by-design: linear models, decision trees, rule lists) and *post-hoc* (explanations for already-trained black boxes), and into *local* (why this prediction) vs. *global* (how the model behaves overall).

The two workhorses of post-hoc local explanation:

- **SHAP (SHapley Additive exPlanations)** — Lundberg & Lee, 2017. Distributes each prediction's "credit" among features using **Shapley values from cooperative game theory**, the only attribution method satisfying a set of consistency/accuracy axioms (local accuracy, missingness, consistency). Model-agnostic (KernelSHAP) with fast exact versions for tree ensembles (TreeSHAP). Output: per-prediction feature contributions, aggregateable into global importance plots.
- **LIME (Local Interpretable Model-agnostic Explanations)** — Ribeiro, Singh & Guestrin, 2016. Approximates the black box **locally** with a simple interpretable surrogate (linear model) fit on perturbed inputs near the instance being explained. Fast and intuitive; less stable than SHAP across perturbations — a known LIME limitation.

Practical cautions to write into any assessment: explanations are **approximations of the model, not the model** (they can be unfaithful, especially LIME); SHAP's theoretical guarantees apply to the *explanation model* under specific assumptions; feature attributions for LLMs (token-level) are noisier and less mature; and explanations must be validated by human review before being shown to regulators or customers. For the banking context, SHAP-based analyses are commonly used in model validation to support SR 11-7 conceptual-soundness reviews and FEAT transparency responses (see [§6](#6-the-banking-angle)).

### 4.5 The Methods Table

| Method | Mechanics | Notes |
|---|---|---|
| **Model audit** | Structured review of data lineage, development, performance evidence, documentation, governance sign-offs, against stated criteria | Criteria-based and independent; produces findings + recommendations; feeds EU Act documentation and ISO 42001 records |
| **Red-teaming** | Adversarial probing: jailbreaks, prompt injection, harmful-content elicitation, exfiltration probes, agent-misuse tests; manual + automated suites | Define harms taxonomy first; document attempts/outcomes/severity; re-run on change; see [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) |
| **Bias audit** | Fairness metrics on protected-attribute slices: demographic parity, equal opportunity, equalized odds, calibration; disparate impact ratio; stereotype/counterfactual probes for LLMs | State the fairness criterion claimed; handle protected data lawfully; feeds FEAT fairness + EU Act non-discrimination |
| **XAI (SHAP/LIME)** | Post-hoc local attributions (Shapley values / local surrogate); global importance aggregates; human review of explanation quality | Explanations approximate the model; validate before external use; supports transparency, oversight, model validation |
| *(Support)* **Benchmark evals** | Standardized suites measuring capability/safety/truthfulness (see §5) | Cheap, comparable, but limited: benchmarks ≠ deployment context |

### 4.6 Choosing and Combining Methods

Method selection follows the risk tier, not fashion:

| Tier | Mandatory minimum evidence | When to add more |
|---|---|---|
| Low (no direct decision impact) | Benchmark scorecard; documentation; monitoring baseline | On any user-facing exposure |
| Medium (informs human decisions) | + model audit; bias audit where protected groups affected | When fairness-sensitive data present |
| High (drives consequential decisions; GenAI in regulated ops) | + red-teaming; XAI review; independent validation; incident plan | When autonomy or external input surfaces grow |

Combining matters as much as choosing: benchmarks *position* the model, audits *check the process*, red-teaming *finds the failures*, bias audits *quantify the group effects*, and XAI *makes the call legible*. A finding in one layer (e.g., a red-team injection success) should trigger re-checks in the others (does the audit's control inventory cover it? does the scorecard catch it? does the explanation surface it?) — the cross-layer loop is what turns a set of tests into an assurance program. See the worked example's evidence pack in [§8](#8-the-worked-example-a-genai-trust-assessment) for the combined shape.

---

## 5. The Evaluation

### 5.1 Trust and Safety Benchmarks

Evaluation is the quantitative backbone of Measure. Capability benchmarks are the familiar floor; **trust evaluation** adds safety, truthfulness, robustness, and bias suites on top. The honest framing: benchmarks give *comparable, reproducible, context-free* signals; they cannot substitute for deployment-specific testing (your data, your prompts, your users) — the gap between benchmark and deployment is precisely where hallucinations and injection attacks live.

The trust-relevant families (verified against public documentation):

- **Truthfulness/honesty:** **TruthfulQA** — 817 questions probing whether models reproduce human misconceptions and falsehoods; the standard "does it say false things confidently" check. **HaluEval / HaluBench**-type hallucination suites target RAG-specific fabrication.
- **Holistic / safety suites:** **HELM** (Stanford CRFM's Holistic Evaluation of Language Models) evaluates accuracy, calibration, robustness, fairness, bias, toxicity, and efficiency; its **HELM-Safety** extension bundles **5 safety benchmarks spanning 6 risk categories — violence, fraud, discrimination, sexual content, harassment, deception** — an explicitly standardized approach to LLM safety evaluation.
- **Harm-resistance:** **HarmBench** — a standardized framework for evaluating LLM responses to adversarial attacks (used in red-teaming automation); **RealToxicityPrompts** — toxicity of continuations via the Perspective API; **XSTest** — the opposite failure: over-refusal of benign prompts (a trust failure too); **SafetyBench**-class suites for Chinese-context and general safety.
- **Robustness:** **RobustBench**-style adversarial-robustness leaderboards for vision/classifier components; input-perturbation and distribution-shift tests for NLP.
- **Bias benchmarks:** **BBQ** (Bias Benchmark for QA — ambiguous-context stereotype reliance), **StereoSet/CrowS-Pairs** (stereotype association measures), **WinoBias/WinoGender** (coreference bias) — the benchmark layer under the bias audits of §4.3.

For the evaluation *engineering* — frameworks like Ragas, DeepEval, Trulens, and the eval-vs-validation distinction — cross-ref [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md), [ai_llm/rag/rag_evaluation_methodology_guide.md](ai_llm/rag/rag_evaluation_methodology_guide.md), and [ai_llm/llm_evaluation_vs_validation_guide.md](ai_llm/llm_evaluation_vs_validation_guide.md).

### 5.2 The Eval Table

| Benchmark / family | What it measures | Typical use in a trust assessment |
|---|---|---|
| **TruthfulQA** | Truthfulness; resistance to misconception-imitation | Baseline "does it lie confidently" gate for any GenAI assistant |
| **HELM / HELM-Safety** | Holistic capability + calibration + robustness + bias + toxicity; 5 safety benchmarks / 6 risk categories | Scorecard backbone; comparable cross-model trust profile |
| **HarmBench** | Resistance to standardized adversarial attacks | Automated red-team layer; quantifies guardrail strength |
| **RealToxicityPrompts** | Toxicity of continuations | Content-safety monitoring baseline |
| **XSTest** | Over-refusal of safe prompts | Usability of safety controls (under-refusal vs over-refusal balance) |
| **BBQ / StereoSet / CrowS-Pairs / WinoBias** | Stereotype reliance and coreference bias | Evidence for bias audits (§4.3) |
| **RobustBench / perturbation suites** | Adversarial robustness (classifiers); input-shift robustness | Reliability/robustness evidence for NIST Measure |
| *Capability floors* (MMLU, GSM8K, coding suites) | Knowledge/reasoning ability | Context for interpreting trust scores — a model must be capable to be useful, but capability ≠ trust |

**Design rule:** pick a small, stable **trust scorecard** (e.g., truthfulness + safety + bias + robustness) re-run on every release, and treat benchmark movement as a release gate — but pair every benchmark result with a deployment-context test (your prompts, your documents, your edge cases) before signing off.

### 5.3 Building the Trust Scorecard

A practical scorecard recipe that has held up across engagements:

1. **Fix the axes first** — pick 4–6 trust axes relevant to the deployment (e.g., truthfulness, safety/harm, robustness, bias, over-refusal, privacy-leakage) — not the model's favorite axes.
2. **Pin the benchmarks** — one benchmark per axis, frozen version, run on every release (e.g., TruthfulQA; HELM-Safety; RobustBench-style perturbation; BBQ; XSTest; an exfiltration probe set).
3. **Set thresholds with a rationale** — thresholds should trace to risk appetite (e.g., "≤1% hallucination on no-answer queries because ops staff will not second-guess confident output"), not to what the model currently achieves.
4. **Add deployment-context sets** — your golden data, your prompt variants, your edge cases; the scorecard without these is marketing.
5. **Track deltas, not absolutes** — the release-gate question is "did anything regress", with absolute thresholds as the floor.
6. **Publish it** — the scorecard is a living document stakeholders can read; it is also the fastest way to kill "but the benchmark says it's great" arguments with data.

For the evaluation-tooling layer (Ragas/DeepEval/Trulens, eval harness design), cross-ref [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md) and [ai_llm/rag/rag_evaluation_methodology_guide.md](ai_llm/rag/rag_evaluation_methodology_guide.md).

---

## 6. The Banking Angle

### 6.1 SR 11-7 / OCC 2011-12 Model Risk Management

**Verified:** **SR 11-7** (Federal Reserve Supervisory Letter, **4 April 2011**) and **OCC Bulletin 2011-12** are the joint US supervisory guidance *Supervisory Guidance on Model Risk Management* (the FDIC adopted the same guidance in 2017). It defines a **model** as a quantitative method/system applying statistical/economic/financial theory to process inputs into estimates for decision-making, and **model risk** as the potential for adverse consequences from decisions based on incorrect or misused model outputs — arising from model *error* (development/implementation) and model *misuse* (inappropriate application).

Its three key elements, which every bank-grade AI assessment must map onto:

1. **Sound development, implementation, and use** — documented development, testing, and fitness-for-purpose analysis before use.
2. **Effective validation** — independent review: **conceptual soundness**, **ongoing monitoring** (benchmarking, backtesting), and **outcomes analysis**; performed by qualified parties independent of development ("effective challenge" is the core concept).
3. **Sound governance, policies, and controls** — board/senior-management oversight, model inventory, risk tiering, documentation, change control, and audit.

The guidance is principles-based and proportionate — the rigor scales with model risk/materiality. It has become the *de facto* global standard for bank model governance, and regulators worldwide (including MAS expectations for model risk management) mirror its shape. For AI/ML the industry practice is to extend SR 11-7 principles to AI models: inventory them, tier them, validate them (with AI-appropriate techniques), and document them.

**Adapting the validation toolkit to AI models.** SR 11-7's techniques adapt to AI/ML in recognizable ways:

| SR 11-7 technique | AI/ML adaptation |
|---|---|
| Conceptual soundness review | Architecture and training-data rationale; literature basis; feature/embedding sanity; the XAI layer (SHAP/LIME) as evidence of *why* the model behaves as it does |
| Benchmarking | Cross-model comparison (the eval tables of §5); challenger models; simple baselines (rules, logistic regression) that the AI must beat |
| Backtesting | Historical replay of AI decisions vs. actual outcomes; for GenAI, golden-set extraction/summary accuracy over past documents |
| Outcomes analysis | Ongoing comparison of predicted vs. realized behavior; drift monitoring; user-feedback mining |
| Sensitivity / stress analysis | Input perturbation, distribution-shift tests, adversarial inputs, prompt-level stress |

The independent-validation function stays the same: whoever validated must not have built it, and the challenge must be *effective* — documented, probing, and empowered to block. For the governance layer in full, cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) (§9 covers model risk management). The quantitative-model deep-dive is in [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) (its §9 covers the governance layer); the systems that operationalize MRM are in [../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md).

**Flag:** US regulators have signaled updates for AI-specific model risk (the OCC issued a refreshed model-risk bulletin in early 2026 per industry coverage **[verify: bulletin number/date]**), and MAS has been increasingly explicit that AI/ML models fall under existing model-risk expectations. The SR 11-7 principles themselves are stable and verified.

### 6.2 BCBS 239-Adjacent Data Governance

Where SR 11-7 governs *models*, **BCBS 239** (Principles for Effective Risk Data Aggregation and Risk Reporting, 2013) governs the *data* under them — and any AI trust assessment rests on the same foundations: data lineage, quality controls, timeliness, and auditability. An AI system fed ungoverned data fails the trust test before any model metric is computed. Practical alignment: the AI inventory and risk register of [§7](#7-the-assessment-process) should reference the same data-asset lineage that BCBS 239 programs maintain, so that a model's data dependencies are traceable to owned, quality-controlled sources (cross-ref the ML-data axis in [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md)).

### 6.3 The Banking Table

| Banking control | What it demands | How an AI trust assessment delivers it |
|---|---|---|
| **SR 11-7 / OCC 2011-12** (2011; FDIC 2017) | Model inventory; risk tiering; development/implementation discipline; independent validation with effective challenge; ongoing monitoring; governance oversight | The assessment IS the validation artifact: documented development, independent review of tests, conceptual-soundness argument, monitoring plan; results feed the model inventory and sign-off |
| **Model risk management programs** (bank policy) | Materiality-based rigor; independent validation function; escalation | Assessment tiering by materiality; validation findings with owners and deadlines; MRM committee escalation path |
| **BCBS 239** (2013) | Risk-data aggregation, lineage, quality, auditability | Data-lineage and quality evidence in the audit; traceable to owned sources |
| **MAS FEAT** (2018) | Fairness, ethics, accountability, transparency of AIDA use | Bias audit (fairness), governance review (ethics/accountability), XAI + disclosures (transparency) — §4 methods mapped to FEAT items |
| **EU AI Act** (where applicable) | Tier classification; high-risk documentation/oversight; GPAI supply-chain duties | Use-case classification in Map; technical documentation from the audit; human-oversight design; model-supplier documentation checks |
| **ISO 42001** (if certified) | AIMS: policy, risk & impact assessments, controls, continual improvement | Assessment artifacts double as AIMS records; AI system impact assessment as standard deliverable |

---

## 7. The Assessment Process

### 7.1 The AI Inventory

Every assessment program starts with **knowing what exists**: an **AI inventory** — a register of every AI/ML system (or AI-influenced decision process) in the organization, with enough metadata to triage: owner, business area, data sources, model type, deployment status, affected populations, and whether it uses third-party/GPAI models (supply-chain visibility, per the EU Act's GPAI chapter and the gateway's model catalog — cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) and [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) for the hub/catalog angle).

The inventory is the bridge between governance and engineering: MRM's model inventory (SR 11-7), the gateway's model registry, and the compliance team's use-case list should all reconcile to one source of truth. Without it, risk assessment is performed on rumors. In practice, the inventory also becomes the input to EU AI Act high-risk classification exercises and to FEAT-style board reporting.

### 7.2 The Risk Register

The **risk register** is the Map output: one entry per use case (or per significant risk within a use case), each with:

- **Context** — system, owner, data, users, decision impact, jurisdiction(s).
- **Hazard identification** — what could go wrong (accuracy failure, bias, injection, hallucination, privacy breach, regulatory breach, misuse), informed by the inventory metadata, incident databases (e.g., the OECD AI incidents monitor), and the methods of §4.
- **Likelihood and impact** — rated on the organization's risk taxonomy (e.g., 1–5 × 1–5), producing a **tier** (low / medium / high / critical) that drives assessment depth — proportionate rigor, as SR 11-7 demands.
- **Existing controls** — guardrails, human review, monitoring, documentation, training.
- **Residual risk and actions** — what remains after controls, who owns it, by when.

The register is a living artifact: updated on material change (model swap, new data source, new prompt surface), reviewed by the AI governance committee, and audited. Controls that fail re-testing move risks back up the register.

### 7.3 The Process Table

| Step | Activity | Output | Framework anchor |
|---|---|---|---|
| 1. **Inventory** | Enumerate AI systems and metadata; reconcile with MRM/gateway registries | AI inventory | Govern (SR 11-7 inventory; gateway catalog) |
| 2. **Triage & classify** | Map to use-case context; EU Act tier check; FEAT relevance; materiality rating | Tiered use-case list; classification memo | Map (EU Act tiers; MAS expectations) |
| 3. **Risk assessment** | Hazard identification; likelihood × impact; existing controls; residual risk | Risk register entries | Map |
| 4. **Test & measure** | Benchmark scorecard; model audit; red-team; bias audit; XAI; robustness tests | Test reports + evidence pack | Measure (NIST; §4/§5 methods) |
| 5. **Evaluate & decide** | Findings vs. acceptance criteria; risk decisions; mitigations and owners | Assessment report; sign-off; acceptance/conditional-approve/reject | Measure → Manage (EU Act conformity; MRM validation) |
| 6. **Control & monitor** | Implement mitigations; drift/harm monitoring; periodic re-assessment; incident response | Controls evidence; monitoring dashboards; re-validation schedule | Manage (and back into Govern) |
| 7. **Report & audit** | Package evidence; committee review; internal audit; regulator-ready pack | Assurance file; audit trail | Govern (ISO 42001 records; FEAT transparency) |

**The golden rule of the process:** every step produces an artifact, and every artifact is dated, versioned, and owned. If it is not written down, it did not happen — trust assessments are judged on the evidence file, not on the meeting.
### 7.4 Roles, Ownership, and the Three Lines of Defence

An assessment process lives or dies on who owns what. The bank-standard mapping:

- **First line (business + platform):** use-case owners and the AI platform team build, test, and self-certify against the acceptance criteria; they own the inventory entry and the day-to-day monitoring.
- **Second line (MRM, risk, compliance):** independent validation of high/medium-tier systems; the risk register is second-line-owned; FEAT / EU-Act / ISO mapping and regulator-facing evidence are compliance-owned.
- **Third line (internal audit):** periodic audit of the assessment process itself — are inventories complete? are sign-offs real? are findings closed?

Every artifact in the process table gets an **owner and a reviewer** who are different people (effective challenge again), and the governance committee holds the escalation path: unclosed high-severity findings go to the committee, not to the backlog. This is the layer that makes "accountability" (dimension 5) a fact rather than a slide.

---

## 8. The Worked Example: A GenAI Trust Assessment

### 8.1 The Scenario: A Cymbal Bank GenAI Deployment

**The system.** A Cymbal Bank-style trade-finance deployment: **"DOC-Ops Copilot"** — an internal GenAI assistant that supports ~500 trade-operations staff in Singapore and Hong Kong. It (a) extracts structured fields (dates, amounts, parties, goods descriptions) from shipping documents (bills of lading, invoices, packing lists) for the operations team's checks, (b) summarizes transaction files and drafts answers to banker queries over them (RAG over the deal/document store), and (c) flags documents for sanctions/blocked-party review. It runs on an enterprise-hosted LLM (via the corporate AI gateway — cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the platform layer: routing, guardrails, PII redaction, audit logging) with a vector store for retrieval. It is *not* a self-executing agent: every output lands in a human review queue — an important design choice for this tier of risk.

**Why this scenario is the familiar context.** It is representative of the GenAI deployments a global bank actually ships in 2025–2026: high-volume document processing, RAG over internal documents, gateway-mediated access, human-in-the-loop, and a risk profile shaped by *accuracy of extracted/summarized information in regulated financial operations* rather than by autonomy. Real-world analogues include the Crédit Agricole group's internal GenAI assistant rollouts **[verify: specific product names/scale per press coverage]**. The assessment below is a realistic composite, not a report on any specific system.

**Why it needs a trust assessment.** Errors are consequential: a wrong extraction can delay a letter-of-credit settlement or misroute a payment; a hallucinated summary can mislead a banker; a prompt injection embedded in a received shipping document can make the copilot ignore instructions; and the data touches personal information (beneficial owners, contact details) plus sanctions-relevant content. Regulators would ask (MAS FEAT, SR 11-7, and the EU AI Act if EU counterparties are in scope) for documented evidence of governance, testing, fairness, and oversight. That is exactly what the assessment produces.

### 8.2 The NIST-RMF-Mapped Assessment Design

The assessment is structured as a NIST AI RMF lifecycle exercise (Govern–Map–Measure–Manage), with the artifacts double-counting toward ISO 42001 records, the MRM inventory (SR 11-7), and FEAT evidence. Scope and tier: **high materiality** — extraction errors can drive financial loss and the system touches sanctions screening — so the full regime applies, not a lightweight pass.

**GOVERN — governance baseline.**

- **Policy and roles:** AI use-case intake completed; business owner (Head of Trade Ops), model owner (AI Platform team), independent validation (MRM), and compliance sign-off roles assigned; RACI logged; the copilot is added to the **model inventory** as a system with an embedded LLM component (supply-chain record: which hosted model, which version, which provider documentation on file — the GPAI paperwork the EU Act would require if EU-applicable).
- **Acceptance criteria:** pre-agreed, quantitative sign-off thresholds (see Measure) so "trustworthy" is not a vibe.

**MAP — context and risk assessment.**

- **Use-case framing:** decision impact — the copilot *informs* human decisions in trade operations; it does not decide. Affected populations: staff, counterparties, and (indirectly) sanctioned-party screening outcomes.
- **Jurisdiction sweep:** Singapore (PDPA, MAS FEAT, MGF-GenAI expectations), EU AI Act (if EU entities/counterparties in scope — classification memo: not high-risk per Annex III as deployed for internal ops support, but the assessment records the reasoning), plus the bank's internal MRM policy.
- **Hazard identification (risk register excerpt):**

| Hazard | Likelihood | Impact | Tier | Key controls (existing) |
|---|---|---|---|---|
| Incorrect field extraction → ops error / settlement delay | Medium | High | **High** | Golden-set testing; human review queue; field-level confidence |
| Hallucinated summary in RAG responses | Medium | High | **High** | Grounding prompt; citations; human review; no-answer behavior |
| Prompt injection via uploaded documents | High | High | **High** | Gateway guardrails; document-vs-instruction separation; red-team testing |
| Group-level bias in prioritization/flagging | Low-Med | Medium | **Medium** | Bias audit; balanced test set; monitoring |
| Data leakage / PII exposure in responses | Low | High | **High** | PII redaction in gateway; access control; data minimization |

**MEASURE — the evidence program.** (Each item produces a dated, owned artifact.)

1. **Benchmark scorecard (release gate):** TruthfulQA + HELM-Safety + RealToxicityPrompts + XSTest on the hosted model, plus a custom extraction golden set (2,000 labeled documents: bills of lading, invoices, packing lists) measuring field-level precision/recall by field and by counterparty region. Acceptance: extraction F1 ≥ 0.97 on high-value fields (amount, date, beneficiary); hallucination rate ≤ 1% on a no-answer test set; XSTest over-refusal ≤ 5%.
2. **Model audit:** data lineage for the golden set and the RAG corpus (ownership, refresh, license/consent); leakage check (no golden-set docs in the vector store); documentation completeness (model card, data card, prompt-and-guardrail specification); reproducibility (pinned model versions, deterministic eval harness).
3. **Red-team (two-week, external + internal team):** jailbreak suites; **document-embedded prompt injection** (instructions hidden in a plausible bill of lading — the expected killer finding); data-exfiltration probes; harmful-content elicitation; refusal-consistency checks. Every attempt logged with severity. Cross-ref [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) for the attack taxonomy used to design the suite.
4. **Bias audit:** extraction accuracy sliced by counterparty country/region and document language; fairness metrics (disparate impact ratio, equal-opportunity difference) on the flag-for-review behavior; stereotype-elicitation probes on the summarization prompts. Stated criterion: *equal opportunity* — the copilot must not systematically miss fields (or over-flag) for any region — chosen because recall errors, not approval-rate differences, are the harm in this use case. That choice is documented, as §4.3 requires.
5. **Explainability:** SHAP analysis on the classification component (e.g., the field-extraction confidence model) to support conceptual-soundness review; LIME on edge cases for spot validation; for the LLM itself, citation-level grounding checks (every summary claim must trace to a retrieved document) as the practical explanation layer — token-attribution SHAP being too noisy for routine use.
6. **Robustness/monitoring baseline:** drift detection on extraction accuracy by field and region; latency/availability SLOs; prompt-and-corpus change control (every vector-store update is versioned).

**Delivery plan.** The assessment ran as a six-week engagement on a fixed calendar:

| Week | Activity | Output |
|---|---|---|
| 1 | Inventory entry, triage, classification memo, risk-register drafting | Tier = High; register v1 |
| 2 | Golden-set build + benchmark scorecard baseline | Scorecard v1 |
| 3 | Model audit (data lineage, documentation, leakage check) | Audit report draft |
| 4 | Red-team campaign (external + internal) | Red-team log + report |
| 5 | Bias audit + XAI review; findings consolidation | Bias report; explanation samples |
| 6 | Controls design, sign-off pack, committee review | Assurance file; conditional approval |

**The evidence pack (the deliverable).** Final index of artifacts archived with the assessment:

1. Classification memo (EU-Act tier reasoning, FEAT applicability)
2. Risk register (v1 → v3, showing findings moving open → mitigated)
3. Benchmark scorecard (baseline + post-fix re-runs)
4. Golden-set spec + extraction results by field and region
5. Model audit report (data lineage, leakage check, documentation gaps)
6. Red-team log (all attempts) + red-team report (severity-ranked findings)
7. Bias audit (region-sliced metrics; stated criterion: equal opportunity)
8. Explanation samples (SHAP on the confidence model; LIME spot-checks)
9. Controls specification (decontamination layer, confidence thresholds, monitoring)
10. Sign-off chain (RACI, committee minutes, conditional-approval conditions)

**MANAGE — mitigations, decision, and monitoring.**

- **Findings disposition:** each finding gets an owner, deadline, and acceptance test; high-severity findings block go-live; the injection finding (below) drove a design change before approval.
- **Controls added:** document-content decontamination (instruction-detection pass on uploaded documents before they reach the model context); field-level confidence thresholds that force manual review; region-sliced monitoring dashboards; an incident response runbook with an internal "AI incident" classification.
- **Ongoing:** quarterly re-assessment (scorecard re-run + targeted red-team re-run), triggered re-assessment on model or corpus change, annual independent audit, and board/committee reporting.

### 8.3 Findings, Evidence, and Actions

| # | Finding (severity) | Evidence | Action | Status |
|---|---|---|---|---|
| 1 | **Document-embedded prompt injection succeeds** (High) — instructions inside a benign-looking bill of lading override the extraction prompt | Red-team log, 6/12 injection variants succeeded | Decontamination layer + instruction-separation guardrail in gateway; re-test all variants | Fixed, re-tested, closed |
| 2 | Extraction F1 on high-value fields 0.93, below 0.97 gate (High) | Golden-set report | Field-specific fine-tune + confidence-threshold review routing; re-run | Fixed, closed |
| 3 | Hallucination on no-answer queries 3% (Medium-High) | No-answer test set | Grounding prompt + mandatory citation; retrain no-answer behavior; monitor | Closed (1.2% after fix) |
| 4 | Equal-opportunity gap: field recall 2.1 pp lower for one region's document formats (Medium) | Bias audit, region-sliced metrics | Region-specific document templates in golden set; monitoring; ops review of impact | Mitigated, monitored |
| 5 | RAG corpus contains two stale, unreviewed documents (Medium) | Data lineage audit | Corpus governance: owner, refresh, sign-off before ingestion | Closed |
| 6 | Over-refusal on benign queries 8% (Low-Med) | XSTest + user feedback | Refusal-tone tuning; user feedback loop | Closed |

**Sign-off:** conditional approval granted after findings 1–3 closed, with finding 4 under monitored mitigation and quarterly re-assessment as a condition. The full assurance file (scorecards, red-team log, audit report, bias analysis, SHAP/explanation samples, risk register, sign-off chain) is archived and regulator-ready — this is the deliverable, not the summary.

### 8.4 The Lessons

1. **Injection is the bank's red-team headline, not harmful content.** For a document-processing copilot, the realistic adversarial failure is *instructions hidden in trusted input*, not the model refusing to be polite. Design red-team suites around the actual input surface (cross-ref [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) and [ai_llm/prompt_injection_guide.md](ai_llm/prompt_injection_guide.md)).
2. **The gateway is the control plane.** Guardrails, PII redaction, logging, and model versioning implemented in the gateway ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) were what made several mitigations deployable in days rather than months — trust controls belong in the platform, not only in the prompt.
3. **Fairness must be defined per use case.** "Equal opportunity," not generic parity, was the right criterion for an extraction tool — and writing *why* is what makes the bias audit defensible to a regulator.
4. **Benchmarks position, context tests decide.** TruthfulQA/HELM told us the model family was sound; only the golden set, the no-answer suite, and the red-team told us *this deployment* was safe.
5. **The assessment is the MRM artifact.** Mapped to the model inventory and validation schedule, the same evidence serves SR 11-7, FEAT, ISO 42001 records, and EU-Act documentation — one evidence program, many consumers. That is the cost-efficient pattern; the assurance-cost conversation is in [../management/business_case_development_guide.md](../management/business_case_development_guide.md).
6. **Documentation is the deliverable.** The system's trustworthiness at launch mattered less than the versioned, owned evidence file that survives model swaps, team changes, and regulatory inquiries.

---

## 9. The Summary

### 9.1 The One-Page Summary

**What AI trust is:** justified, evidence-backed confidence that an AI system is reliable, safe, fair, transparent, and accountable — a sociotechnical property of model + data + deployment + people, maintained across the lifecycle.

**What the frameworks give you:** a process (NIST AI RMF's **Govern–Map–Measure–Manage**, Jan 2023), a legal floor (EU AI Act 2024: four risk tiers, high-risk obligations, GPAI provisions, phased to 2027), a certifiable management system (ISO/IEC 42001, Dec 2023), and the principles floor (OECD, 2019, updated 2024). None alone is sufficient; together they compose.

**What Singapore adds:** the world's first Model AI Governance Framework (2019, 2nd ed. 2020), the world's first AI testing framework and toolkit **AI Verify** (May 2022, 11 governance principles), the generative-AI framework (proposed Jan 2024, final May 2024), and MAS **FEAT** (Nov 2018) with the Veritas assessment methodologies — the local regulator's concrete expectations for a Singapore-based bank.

**What the methods produce:** model audits (criteria-based process evidence), red-teaming (adversarial failure discovery), bias audits (group-fairness metrics with an explicit criterion), and XAI (SHAP/LIME) for legibility — supported by benchmark evaluation (TruthfulQA, HELM-Safety, HarmBench, bias suites) that positions but never replaces deployment-context testing.

**What the process is:** inventory → classify → risk register → test → decide → monitor → report, each step emitting a dated, owned artifact, with the whole mapped onto SR 11-7 model risk management and BCBS 239 data governance for banks.

### 9.2 The Trust Case (Final Word)

The trust case is the answer to the question every stakeholder — banker, regulator, auditor, customer, and the engineer at 2 a.m. watching the drift dashboard — is really asking: **"Why should I rely on this system, and what happens if I'm wrong to?"**

A trust assessment answers it with evidence instead of assurances. It converts the five dimensions into testable claims; it maps those claims onto frameworks the institution already answers to; it produces artifacts that survive scrutiny; and it puts the failures — the injections that succeeded, the fields that were missed, the groups that were underserved — on the table where they can be fixed and monitored rather than discovered later by a regulator or a customer.

For a bank, this is not a compliance tax. It is the mechanism by which AI gets to be useful: the trade-ops copilot ships only because the assessment made its limits legible and its controls real; the next use case ships faster because the evidence template already exists. **Trust is not a property you claim — it is a record you keep.** The assessment is how you keep it, and the record is the product. That is the trust case: *we tested it, we documented it, we own the residual risk, and here is the evidence — check it yourself.*

---

## Glossary

- **AI trust** — justified, evidence-backed confidence that an AI system is reliable, safe, fair, transparent, and accountable across its lifecycle (sociotechnical: model + data + deployment + people).
- **Trust assessment** — the structured process of producing evidence about those dimensions: audits, tests, metrics, documentation, and sign-offs; the assurance discipline this guide covers.
- **NIST** — US National Institute of Standards and Technology; publisher of the AI RMF and AI 100-2.
- **AI RMF** — the NIST AI Risk Management Framework (AI 100-1, Jan 2023): a voluntary, rights-preserving, sociotechnical risk-management framework for AI.
- **Risk management framework** — a structured set of processes (identify, assess, prioritize, treat, monitor) for managing risk; the AI RMF is the AI-specific exemplar.
- **Govern** — the AI RMF's cross-cutting function: culture, policy, roles, accountability, transparency commitments.
- **Map** — the AI RMF function that establishes context and identifies risks (what could go wrong, for whom).
- **Measure** — the AI RMF function that assesses and quantifies risks via tests, metrics, and evaluations.
- **Manage** — the AI RMF function that prioritizes, mitigates, and monitors risks; feeds back into Govern.
- **EU AI Act** — Regulation (EU) 2024/1689 (in force 1 Aug 2024; phased 2025–2027): the first comprehensive horizontal AI regulation.
- **GPAI** — general-purpose AI: models with broad applicability (foundation/LLM class); subject to transparency, documentation, and (for systemic-risk models) evaluation/adversarial-testing duties from Aug 2025.
- **High risk** — the EU Act tier (Annex III areas incl. employment, creditworthiness assessment, biometrics, etc.) subject to the full conformity regime from 2026–2027.
- **Unacceptable risk** — the EU Act tier of prohibited practices (e.g., social scoring, manipulative techniques), banned from Feb 2025.
- **ISO 42001** — ISO/IEC 42001:2023 (Dec 2023): the world's first certifiable AI management system (AIMS) standard.
- **OECD** — Organisation for Economic Co-operation and Development; publisher of the first intergovernmental AI principles (May 2019, updated May 2024).
- **AI principles** — high-level values-based commitments for trustworthy AI (human-centredness, fairness, transparency, robustness, accountability); the OECD set is the canonical instance.
- **Model AI Governance Framework** — Singapore's world's-first national AI governance framework (IMDA/PDPC; Jan 2019, 2nd ed. Jan 2020).
- **AI Verify** — Singapore's world's-first AI governance testing framework and toolkit (IMDA/PDPC, launched May 2022; 11 governance principles; AI Verify Foundation from Jun 2023).
- **Generative AI** — AI that produces new content (text, images, code); the subject of Singapore's MGF for GenAI (2024) and the EU Act's GPAI provisions.
- **MAS** — Monetary Authority of Singapore; financial-sector regulator and issuer of FEAT.
- **FEAT** — Fairness, Ethics, Accountability and Transparency: MAS's 2018 principles for AI/data-analytics use in Singapore financial services.
- **Fairness** — the trust dimension that systems do not systematically disadvantage protected groups; implemented via stated fairness criteria and bias audits.
- **Ethics** — the organizational commitment that AI is used consistently with ethical standards, with management accountability (FEAT's ethics principle).
- **Accountability** — clear ownership, sign-off, escalation, and audit trails for AI decisions (a trust dimension and a FEAT principle).
- **Transparency** — disclosure of AI use, capabilities, limitations, and decision logic to the right audiences (a trust dimension, FEAT principle, and EU Act limited-risk duty).
- **Model audit** — an independent, criteria-based examination of a model and its context (data, development, documentation, governance).
- **Red-teaming** — adversarial testing that deliberately attempts to make a system fail (jailbreaks, injection, harmful content, exfiltration).
- **Bias audit** — measurement of group-level disparate treatment using fairness metrics on protected-attribute slices.
- **Fairness metric** — a quantified fairness criterion: demographic parity, equal opportunity, equalized odds, predictive parity/calibration, disparate-impact ratio.
- **XAI** — explainable AI: methods producing human-understandable accounts of model behavior.
- **Explainability** — the property that model behavior can be rendered understandable; the transparency/accountability enabler.
- **SHAP** — SHapley Additive exPlanations (Lundberg & Lee, 2017): game-theoretic feature attributions for model explanations.
- **LIME** — Local Interpretable Model-agnostic Explanations (Ribeiro et al., 2016): local surrogate-model explanations.
- **SR 11-7** — Fed Supervisory Letter 11-7 / OCC 2011-12 (April 2011): the supervisory guidance on model risk management that anchors bank model governance.
- **Model risk** — the potential for adverse consequences from decisions based on incorrect or misused model outputs (SR 11-7 definition).
- **AI inventory** — the register of all AI systems with ownership/context metadata; the foundation of assessment and MRM.
- **Risk register** — the living record of identified risks (likelihood, impact, controls, residual risk, owners).
- **Benchmark** — a standardized evaluation suite (e.g., TruthfulQA, HELM-Safety, HarmBench) giving comparable trust/capability signals.

---

## Appendix: Verification Log (August 2026)

| Claim | Status | Source |
|---|---|---|
| NIST AI RMF 1.0 published January 2023; functions Govern–Map–Measure–Manage; Govern cross-cutting | **Verified** | NIST AI 100-1 (nist.gov; nvlpubs.nist.gov) |
| EU AI Act = Regulation (EU) 2024/1689; in force 1 Aug 2024; phased Feb 2025–Aug 2027; four risk tiers | **Verified** | Multiple independent regulatory summaries |
| EU Act penalties up to €35M / 7% turnover (unacceptable practices); €15M / 3% (other breaches) | **Verified** (as ceilings) | Compliance references |
| GPAI obligations from 2 Aug 2025; ~10²⁵ FLOPs systemic-risk presumption | **Verified** (threshold per delegated act) | Regulatory summaries |
| ISO/IEC 42001 published December 2023; world's first certifiable AI management system standard | **Verified** | iso.org |
| OECD AI Principles adopted May 2019; updated May 2024 | **Verified** | oecd.org |
| SG Model AI Governance Framework launched Jan 2019 at Davos (world's first); 2nd edition Jan 2020 | **Verified** | IMDA/PDPC/EDB releases |
| AI Verify announced Jan 2022 (WEF), launched May 2022; IMDA+PDPC; 11 governance principles; open-source | **Verified** | IMDA/PDPC press releases |
| "MLTC" acronym for AI Verify's original test criteria | **Unverified — flagged** | Not found in primary sources |
| MGF for GenAI: proposed 16 Jan 2024; final May 2024; 9 dimensions | **Verified** | IMDA / AI Verify Foundation |
| MAS FEAT published 12 Nov 2018; Veritas fairness methodology dated 2020 | **Verified** (Veritas dates partially) | MAS / industry sources |
| SR 11-7 / OCC 2011-12 issued 4 Apr 2011; FDIC adopted 2017 | **Verified** | federalreserve.gov |
| HELM-Safety: 5 safety benchmarks across 6 risk categories | **Verified** | crfm.stanford.edu |
| SHAP (Lundberg & Lee, 2017); LIME (Ribeiro et al., 2016) | **Verified** | Canonical papers |
| NIST AI 100-2 adversarial-ML taxonomy published March 2024 | **Verified** | NIST |
| OCC model-risk bulletin refresh (early 2026) | **Partially verified — flagged** | Industry coverage |
| Crédit Agricole group internal GenAI assistant names/scale | **Unverified — flagged** | Press coverage varies |

---


## Primary Sources and Further Reading

- **NIST AI RMF 1.0 (AI 100-1), January 2023** — nist.gov/itl/ai-risk-management-framework
- **NIST AI 100-2 E2023 (Adversarial ML taxonomy), March 2024** — nist.gov
- **Regulation (EU) 2024/1689 (AI Act)** — EUR-Lex; official phased-timeline notices
- **ISO/IEC 42001:2023** — iso.org/standard/42001
- **OECD AI Principles (2019, updated 2024)** — oecd.org/en/topics/ai-principles.html; oecd.ai
- **Singapore Model AI Governance Framework (2019; 2nd ed. 2020)** — IMDA/PDPC
- **AI Verify (2022) and MGF for GenAI (2024)** — imda.gov.sg; aiverifyfoundation.sg
- **MAS FEAT Principles (Nov 2018) and Veritas** — mas.gov.sg
- **SR 11-7 / OCC 2011-12 (April 2011)** — federalreserve.gov/boarddocs/srletters/2011/sr1107.pdf
- **HELM / HELM-Safety** — crfm.stanford.edu/helm
- **TruthfulQA (Lin et al., 2021); SHAP (Lundberg & Lee, 2017); LIME (Ribeiro et al., 2016)** — canonical papers
- **Repo companions** — [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md), [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md), [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md), [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md), [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md), [../banking/regtech_guide.md](../banking/regtech_guide.md)


*This guide was authored by Jack Liu Shurui, Solution Architect at Cymbal Bank. Framework dates and provisions were verified against primary sources (NIST AI 100-1; Regulation (EU) 2024/1689; ISO/IEC 42001:2023; OECD Council Recommendation on AI; IMDA/PDPC/MAS releases) in August 2026; items that could not be fully verified are flagged **[verify]** inline. Series companions: [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md), [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md), [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md), [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md).*

*For questions, corrections, or contributions, please open an issue or PR at: https://github.com/jackliusr/research*
