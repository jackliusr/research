# Governance, Regulatory, Risk, Privacy & Security Requirements for AI and GenAI Solutions in Banking: A Comprehensive Guide

*The banking-domain requirements map — what binds an AI or GenAI deployment in a bank, who says so, and what the evidence looks like. The dedicated deep-dive the cluster was missing: governance + regulatory + risk + privacy + security requirements for AI and GenAI in banking, as a single requirements matrix.*

**Series:** Banking & Financial Technology Guides — this is the **dedicated banking-domain requirements map** for AI and GenAI. The framework *details* (NIST AI RMF, EU AI Act provisions, ISO 42001, MAS FEAT, the SR 11-7 validation discipline, the OWASP LLM Top 10) are owned by sibling guides in the `technology/` AI-governance and security clusters; this guide **synthesises them through the banking lens** — which requirement binds which use case, from which source, with what evidence — and cross-references constantly instead of re-deriving.

**Audience:** Solution architects, risk technologists, compliance engineers, and AI/ML platform owners at global and wholesale banks (e.g., a Cymbal Bank Singapore franchise) who must answer the recurring question: *"what do we have to do to put this AI into production?"* The answer is never one regulation — it is a stack of binding and authoritative requirements from the EU, Singapore, the US, and the bank's own model-risk machinery, and it differs by use case.

**How this guide is organised:** an overview (what a banking AI "requirement" is, and the five families); the regulatory landscape (EU AI Act, MAS, US regulators, and the regulatory table); the governance requirements (committees, three lines of defence, AI inventory, model-risk integration); the risk requirements (model risk for AI, the GenAI risk taxonomy); the privacy requirements (GDPR Art 22, DPIA, PDPA, training-data PII); the security requirements (secure-AI development, adversarial); the use-case mapping (credit scoring / AML / chatbots / trading); a worked example (an AI-compliance gap assessment for a Cymbal Bank GenAI deployment); the ongoing obligations (monitoring, incidents, and keeping current); a one-page summary; a glossary; a verification ledger; and cross-references. Every section ends with a reference table.

**Scope note on verification:** regulatory dates, article numbers, and instrument status below were checked against primary sources (EUR-Lex/EC AI Act service desk, MAS publications via reputable counsel summaries, the Federal Reserve and OCC websites, gdpr-info.eu, the PDPC) in August 2026. Where a fact could not be fully verified — most notably the **status of the MAS AI Risk Management Guidelines consultation** and the **CFPB 1071 rule's litigation/reconsideration status** — it is explicitly flagged **[verify]** rather than asserted. The April 2026 US model-risk guidance replacement (SR 26-2 / OCC Bulletin 2026-13) is verified against the agencies' own pages.

**Cross-references used throughout:** `../technology/ai_trust_assessments_guide.md` (NIST AI RMF / EU AI Act / ISO 42001 / OECD / SG frameworks — verified facts), `../technology/responsible_ai_frameworks_guide.md` (RAI canon, corporate frameworks, MAS FEAT, tooling), `../technology/ai_verify_guide.md` (Singapore AI testing), `../technology/ai_llm/ai_governance_bias_redteaming_guide.md` (governance, bias, red-teaming), `../technology/beyond_zero_enterprise_security_guide.md` (agent-era security and agent governance), `../technology/llm_development_risks_security_guide.md` (the GenAI risk taxonomy), `../technology/security_by_design_guide.md` and `../technology/cybersecurity_guide.md` (secure development and the banking security context), `../technology/adversarial_ml_attacks_guide.md` (adversarial ML), `../technology/data_governance_guide.md` (data privacy lineage), `financial_risk_compliance_systems_guide.md` (the compliance systems landscape), `risk_management_models_guide.md` (model risk and the SR 11-7 discipline), `regtech_guide.md` (regulatory technology), `banks_in_singapore_guide.md` (MAS context, Notice 637, TRM), `basel_regulatory_capital_guide.md` / `risk_data_aggregation_guide.md` / `treasury_alm_guide.md` / `cecl_guide.md` (the regulatory trio — touched lightly), `../management/business_case_development_guide.md` (the business case for the governance spend).

### Reading paths

- **Solution architects / platform owners** — §1 (the requirement families), §3 (governance machinery your platform must encode), §6 (security), §7 (which use cases pull which requirements), §8 (the gap-assessment method). Pair with `risk_management_models_guide.md` §9–10 and `../technology/llm_development_risks_security_guide.md`.
- **Compliance / risk officers** — §2 (the regulatory landscape and the regulatory table), §4 (risk requirements), §5 (privacy), §9 (the ongoing obligations), §10 (the one-page summary).
- **Model risk / validation teams** — §3.4 (model-risk integration), §4.1 (model-risk management for AI), §8; the SR 11-7/SR 26-2 vocabulary lives in `risk_management_models_guide.md` §9.
- **Data scientists / GenAI engineers** — §4.2 (the GenAI risk taxonomy), §5.4 (training-data PII), §6 (security), §7.5 (the mapping table) — with the framework mechanics in the technology cluster.

---

## Table of Contents

1. The Overview: What the Banking-AI Requirements Are
2. The Regulatory Landscape: EU AI Act, MAS, US Regulators
3. The Governance Requirements: Committees, Three Lines, Inventory, Model-Risk Integration
4. The Risk Requirements: Model Risk for AI and the GenAI Risk Taxonomy
5. The Privacy Requirements: GDPR Article 22, DPIA, PDPA, Training-Data PII
6. The Security Requirements: Secure-AI Development and the Adversarial
7. The Use-Case Mapping: Credit Scoring, AML, Chatbots, Trading
8. The Worked Example: An AI-Compliance Gap Assessment
9. The Ongoing Obligations: Monitoring, Incidents, and Keeping Current
10. The One-Page Summary: The Final Word — the Requirements That Bind
11. The Glossary
12. The Verification Ledger
13. The Cross-References

---

## 1. The Overview: What the Banking-AI Requirements Are

### 1.1 The Definition (Verified)

**A banking AI requirement is an expectation — binding (law, regulation, supervisory letter) or authoritative-but-non-binding (guidance, principles, standards, industry practice) — that a bank must satisfy to develop, deploy, or operate an AI or GenAI system, stated by a source the bank's supervisors, auditors, or counterparties will hold it to.** The definition is deliberately two-sided, because the compliance reality is two-sided:

- **Binding requirements** attach legal consequences. The **EU AI Act** (Regulation (EU) 2024/1689) is hard law with fines up to **€35M or 7% of global turnover**; the **GDPR**'s Articles 22 and 35 are directly enforceable data-protection obligations; **ECOA/Reg B** in the US carries civil liability for adverse-action failures; Singapore's **PDPA** carries financial penalties (up to 10% of local annual turnover for the most serious breaches under the 2020 amendments).
- **Authoritative-but-non-binding requirements** attach supervisory consequences. **MAS FEAT** is voluntary-but-authoritative — MAS expects banks to *demonstrate* fair dealing, and the 2024 AI model-risk information paper and the 2025 AI risk-management consultation make the supervisory direction explicit. The US **model-risk guidance** (SR 11-7, now superseded by the April 2026 revised guidance) is guidance, not a rule — but examiners enforce it through the examination process, and it is the template supervisors worldwide apply. **NIST AI RMF** and **ISO/IEC 42001** are voluntary frameworks that have become the *de facto* internal standards because they produce exactly the evidence the binding layer demands.

**The banking specificity.** Banks are not ordinary AI deployers, for five structural reasons (each expanded in later sections):

1. **The model-risk overlay.** Almost every AI in a bank is a "model" in the regulators' sense, so the classical model-risk-management (MRM) lifecycle — inventory, independent validation, ongoing monitoring, outcomes analysis — applies on top of AI-specific law. This is the single biggest difference between banking AI compliance and general-industry AI compliance.
2. **Consumer and fair-lending protection.** Credit-related AI touches protected classes, adverse-action rights, and explainability duties that no other industry faces at the same intensity.
3. **Privacy at scale.** Banks hold the most sensitive personal data in the economy; every AI pipeline — training, inference, retrieval — is a data-protection event.
4. **Security criticality.** AI systems sit inside the bank's perimeter and increasingly *act* (agents, automated decisions); adversarial AI is not theoretical for fraudsters targeting payment systems.
5. **Supervisory density.** Multiple supervisors (MAS, ECB, PRA, Fed/OCC, local regulators) with overlapping remits — the same GenAI assistant deployed from Singapore serving EU clients can be in scope of the EU AI Act, GDPR, PDPA, and the 2026 US model-risk guidance simultaneously.

### 1.2 The Five Requirement Families

The requirements sort into five families, and the families interlock — a gap in one almost always creates a gap in another:

- **Governance** — the decision machinery: who approves AI use, who owns it, what is in the inventory, how the three lines of defence divide duties. Without governance, none of the other families can produce evidence.
- **Regulatory** — the external instruments that bind the bank: EU AI Act (including the GPAI layer), MAS expectations (FEAT, the AI model-risk information paper, the proposed AIRM guidelines), US guidance (SR 26-2/2026 MRM guidance, ECOA/Reg B), and the frameworks the bank chooses to encode them (NIST AI RMF, ISO 42001).
- **Risk** — model risk (the SR 11-7 discipline applied to AI) plus the GenAI-specific risk taxonomy (hallucination, prompt injection, data leakage, drift, excessive agency).
- **Privacy** — GDPR Article 22 automated decision-making, Article 35 DPIAs, Singapore's PDPA and the PDPC's AI advisory guidelines, and the training-data PII problem.
- **Security** — secure-AI development (secure SDLC, model supply chain, MLOps security), adversarial robustness (evasion, poisoning, extraction), and agent-era controls.

### 1.3 The Overview Table

| Aspect | Description | Primary sources | Deep-dive |
|---|---|---|---|
| **Governance** | Committees, ownership, three lines of defence, AI inventory, model-risk integration, policies and escalation | Board/regulator expectations; MAS FEAT Ethics & Accountability; SR 26-2 governance chapter; ISO 42001 Clauses 5–7 | §3 |
| **Regulatory** | EU AI Act risk tiers (incl. Annex III credit scoring and GPAI); MAS FEAT + AI MRM info paper + AIRM consultation; US MRM guidance (SR 11-7 → SR 26-2) and fair-lending law | Regulation (EU) 2024/1689; MAS publications; Fed/OCC/FDIC; ECOA/Reg B; CFPB Circulars | §2 |
| **Risk** | Model risk lifecycle for AI/ML; GenAI taxonomy: hallucination, prompt injection, data leakage, drift, excessive agency | SR 26-2 / legacy SR 11-7; OWASP LLM Top 10; MAS AI MRM info paper | §4 |
| **Privacy** | Automated decision-making rights, DPIAs, PDPA obligations, training-data PII, data protection by design | GDPR Arts. 22, 35; PDPA + PDPC AI Guidelines (2024); ISO 42001 AI system impact assessment | §5 |
| **Security** | Secure-AI SDLC, adversarial robustness, model/agent supply chain, monitoring and incident response | `security_by_design_guide.md`; `adversarial_ml_attacks_guide.md`; `llm_development_risks_security_guide.md`; EU AI Act accuracy/robustness/cybersecurity | §6 |

### 1.4 The Synthesis Framing — How This Guide Relates to the Cluster

This guide is the **banking-domain capstone** of the repo's AI-governance, security, and risk clusters — it ties them together rather than duplicating them. The division of labour, which the reader should rely on throughout:

- **The frameworks live in the technology cluster.** `../technology/ai_trust_assessments_guide.md` carries the verified facts on NIST AI RMF 1.0 (January 2023, Govern–Map–Measure–Manage), the EU AI Act's four risk tiers and timeline, ISO/IEC 42001 (December 2023, the certifiable AI management system with its AI system impact assessment), the OECD principles (2019, updated May 2024), and the Singapore framework layer (Model AI Governance Framework, AI Verify, the GenAI framework). `../technology/responsible_ai_frameworks_guide.md` carries the corporate RAI canon, the tooling (SHAP, AIF360, Fairlearn), and the full MAS FEAT treatment (§6.1 there, verified facts in trust guide §3.4). `../technology/ai_verify_guide.md` covers Singapore's AI testing.
- **The GenAI risk and security content lives in the security cluster.** `../technology/llm_development_risks_security_guide.md` owns the OWASP LLM Top 10 mapping, prompt-injection mechanics, RAG data leakage, overreliance/hallucination, and the defense-in-depth reference architecture; `../technology/adversarial_ml_attacks_guide.md` owns evasion/poisoning/extraction; `../technology/security_by_design_guide.md` and `../technology/cybersecurity_guide.md` own secure-SDLC and the banking security context; `../technology/beyond_zero_enterprise_security_guide.md` owns agent-era controls.
- **The model-risk and compliance machinery lives in the banking cluster.** `risk_management_models_guide.md` §9 owns the SR 11-7 discipline and §10 the ML-in-risk treatment; `financial_risk_compliance_systems_guide.md` owns the AML/compliance platforms; `banks_in_singapore_guide.md` owns the MAS licensing/Notice 637 context.
- **What this guide alone delivers:** the **requirements matrix** (requirement → source → banking application → evidence), the **AI-in-banking regulatory landscape** (EU AI Act Annex III 5(b), MAS's 2024/2025 AI model-risk moves, the 2026 US MRM guidance replacement, ECOA/Reg B adverse-action), the **GenAI risk taxonomy in banking terms**, the **use-case mapping** (which requirements bind credit scoring vs. AML vs. chatbots vs. trading), and the **gap-assessment worked example** — the "so what do we actually do on Monday" layer.

---

### 1.5 The Requirement-to-Evidence Pattern

Every requirement in this guide reduces to an artifact — and the artifact is what the supervisor, auditor, or validator actually reads. The pattern to internalise:

| Requirement family | Canonical evidence | Who reviews it |
|---|---|---|
| Governance | Inventory entry, committee sign-off record, RACI, risk-appetite statement | Internal audit, board, examiners |
| Regulatory | Classification record (EU-Act tier, FEAT relevance), EU database registration, technical documentation, vendor GPAI due-diligence file | Compliance, EU AI Office/notified bodies, MAS |
| Risk | Validation report (conceptual soundness, monitoring, outcomes), model card, bias and robustness test reports, KRI dashboards | MRM second line, validators, examiners |
| Privacy | DPIA, lawful-basis documentation, Art. 22 safeguards record, breach register | Data protection officer, PDPC/EDPB/DPA |
| Security | Threat model, security test reports (injection/adversarial suites), trace logs, incident playbooks | CISO/security architecture, internal audit |

Two consequences. First, the **evidence file per system** — one folder per inventory entry holding every artifact above — is the unit of compliance; when a regulator asks, the bank produces the file, not a narrative. Second, the deliverable of any gap assessment is the **evidence backlog** (what is missing), because a requirement without evidence is, in practice, unmet — the same conclusion the trust guide's worked example reaches for trust assessments (`../technology/ai_trust_assessments_guide.md` §8) and the RAI guide's for RAI frameworks (`../technology/responsible_ai_frameworks_guide.md` §7).

---

## 2. The Regulatory Landscape: EU AI Act, MAS, US Regulators

### 2.1 The EU AI Act (Regulation (EU) 2024/1689)

**Verified facts** (provision-level facts cross-checked against the EC's AI Act service desk and `../technology/ai_trust_assessments_guide.md` §2.2, which carries the full treatment):

- **Adoption and timeline.** Adopted June 2024, **in force 1 August 2024**, phased application: prohibited practices from **2 February 2025**; GPAI obligations from **2 August 2025**; high-risk (Annex III) obligations from **2 August 2026**; Annex I product-related high-risk from **2 August 2027**. Penalties up to €35M or 7% of worldwide annual turnover for prohibited practices (lesser tiers: €15M/3%, €7.5M/1%).
- **The risk-tier architecture.** Four tiers — unacceptable (prohibited), high (full obligations), limited (transparency: AI-generated content labelling, chatbot disclosure), minimal (voluntary codes). Banking AI lands overwhelmingly in *high* (credit-related) or *limited* (chatbots/transparency) with a GPAI layer underneath.

**The Annex III high-risk credit-scoring category (verified).** Annex III point 5(b) classifies as high-risk: *"AI systems intended to be used to evaluate the creditworthiness of natural persons or establish their credit score, with the exception of AI systems used for the purpose of detecting financial fraud."* Three consequences for banking:

1. **Credit scoring of natural persons is high-risk, full stop** — whether the model is a classical scorecard, gradient boosting, or a GenAI-assisted underwriting pipeline. This covers consumer and SME-ish lending where the "natural person" (including sole traders and guarantors) is scored. **Wholesale/corporate credit scoring of legal entities is *not* in Annex III 5(b)** — a point of relief for a CIB-style franchise, but the bank should still treat it as high-rigour via model risk.
2. **The fraud-detection carve-out is real and narrow.** Annex III 5(b) excludes "AI systems used for the purpose of detecting financial fraud" — AML/fraud transaction monitoring is carved out of the high-risk regime (though not out of GDPR, model risk, or the Act's other provisions). Do not stretch the carve-out to adjacent uses (e.g., using a fraud model's score to deny a loan is creditworthiness, not fraud detection).
3. **High-risk obligations (from 2 August 2026)** — the full regime the bank must stand up: a **risk management system** (Art. 9); **data governance** (Art. 10 — training/validation/testing data must be relevant, representative, and bias-managed — the direct EU hook for the fair-lending analytics in `../technology/ai_llm/ai_governance_bias_redteaming_guide.md`); **technical documentation** (Art. 11); **record-keeping/logging** (Art. 12); **transparency to deployers** (Art. 13); **human oversight** (Art. 14 — the design hook for human-in-the-loop underwriting); **accuracy, robustness and cybersecurity** (Art. 15 — the hook for adversarial testing); conformity assessment, CE marking, and EU database registration (Arts. 43, 49). Under **Article 26**, deployers (the bank) carry obligations too: use per instructions, assign human oversight, monitor for risks, and inform the provider of serious incidents.

**The GPAI obligations (verified).** The general-purpose AI chapter (Articles 51–56) regulates the foundation models underneath the bank's applications: providers must publish technical documentation, training-data summaries, and a copyright-compliance policy; models above a compute threshold (Commission presumption ~10²⁵ FLOPs) are presumed **systemic risk** and must run model evaluations, adversarial testing, incident reporting, and cybersecurity protections, under the European AI Office. **The banking consequence: trace the model supply chain.** A bank deploying a GPAI-based assistant inherits deployer documentation and oversight duties it cannot delegate away — the vendor's GPAI compliance is part of the bank's procurement and risk assessment, not a substitute for it.

### 2.2 The MAS (Singapore)

**Verified facts** (FEAT facts cross-ref `../technology/ai_trust_assessments_guide.md` §3.4 and `../technology/responsible_ai_frameworks_guide.md` §6.1, which carry the verified details):

- **FEAT principles — 12 November 2018.** MAS published the *Principles to Promote Fairness, Ethics, Accountability and Transparency (FEAT) in the Use of AI and Data Analytics in Singapore's Financial Sector*: **Fairness** (AIDA decisions sound and fair; data not systematically biased against protected groups), **Ethics** (use consistent with ethical standards, board-level oversight), **Accountability** (clear allocation of responsibility, auditability, model-risk integration), **Transparency** (disclosure to consumers and timely explanation of decisions materially influenced by AI). Non-binding but authoritative; the **Veritas initiative** (MAS-industry consortium) published the Fairness Assessment Methodology (2020) and later assessment guides that operationalise FEAT into checkable processes.
- **Information Paper on AI Model Risk Management — 5 December 2024.** MAS published an information paper following a **mid-2024 thematic review of banks' AI (including generative AI) model risk management practices**. The paper sets out supervisory observations and good practices across governance and oversight, risk identification and measurement, and controls — in effect, MAS applying the SR 11-7-style discipline to AI/GenAI at Singapore banks, with GenAI-specific expectations (validation of LLM-based systems, prompt/guardrail controls, human oversight).
- **Consultation on Guidelines on AI Risk Management — 13 November 2025 (flagged).** MAS issued a **consultation paper proposing Guidelines on AI Risk Management (AIRM Guidelines)** that would apply to *all* FIs, setting out supervisory expectations on oversight of AI, and intended to complement FEAT (and the national Model AI Governance Framework for Generative AI from 2024). **Status [verify]:** as of this writing the guidelines are proposed — the consultation closed and final guidelines were expected; confirm the finalised text and effective date before citing them as binding. The direction of travel is unambiguous: MAS is moving FEAT from principles toward supervisory expectations with governance, risk-management, and testing content.
- **The wider MAS envelope.** `banks_in_singapore_guide.md` carries the context: **MAS Notice 637** (Technology Risk Management) and the TRM guidelines govern operational resilience and technology risk — AI platforms, like everything else, must meet the resilience, change-management, and (post-2023 outage saga) operational-readiness expectations; outsourcing rules govern third-party AI vendors; and the MAS 610-series returns keep the supervisory relationship data-driven.

**The MAS angle for a Cymbal Bank-Singapore architect:** FEAT is the *report card* (bias audits for Fairness, board oversight for Ethics, MRM sign-offs for Accountability, disclosures + XAI for Transparency — each mapped to evidence in the RAI guide's tooling), the December 2024 information paper is the *expected practice*, and the November 2025 consultation is the *direction of travel* — build the AI-inventory + governance + validation machinery now, and the finalised guidelines will be a marginal add, not a rebuild.

### 2.3 The US Regulators

**The model-risk guidance: SR 11-7 → SR 26-2 / OCC Bulletin 2026-13 (verified against the agencies' own pages).** The Fed and OCC issued **SR 11-7 / OCC Bulletin 2011-12, *Supervisory Guidance on Model Risk Management*, on 4 April 2011** — the canonical definition of a model ("a quantitative method, system, or approach that applies statistical, economic, financial, or mathematical theories, techniques, and assumptions to process input data into quantitative estimates"), the inventory–validation–monitoring lifecycle, independent validation (conceptual soundness, ongoing monitoring, outcomes analysis), effective challenge, and the use test. It was extended to AI/ML in supervisory practice — most explicitly in the **SR 21-8 / OCC 2021-19 interagency statement (April 2021)** applying MRM principles to BSA/AML models — and the agencies' **March 2021 AI RFI** (Fed/OCC/FDIC/NCUA/CFPB) signalled the direction. **Then the regime was replaced:** on **17 April 2026**, the Fed, OCC, and FDIC issued the **Revised Guidance on Model Risk Management — SR Letter 26-2 and OCC Bulletin 2026-13** — which **supersedes and rescinds SR 11-7 and SR 21-8** (and the 1997 credit-scoring bulletin). The revised guidance: (1) keeps the risk-based core (tailored to model risk profile, size, and complexity — most relevant to institutions over $30B assets); (2) covers development/use, validation/monitoring, governance/controls, and vendor/third-party products; (3) **defers detailed AI/GenAI/agentic-AI expectations to a planned request for information** **[verify]** — the agencies state they will issue an RFI considering banks' use of AI, including generative and agentic AI. **The practitioner's read:** SR 11-7's vocabulary (conceptual soundness, effective challenge, outcomes analysis) is still the lingua franca of validation; the 2026 guidance confirms MRM remains the umbrella for AI; and the pending RFI is where the next AI-specific rules will come from. Cross-ref `risk_management_models_guide.md` §9 (which carries the full SR 11-7 treatment and flags the 2026 refresh) and §10 (ML in risk).

**ECOA/Reg B adverse-action explainability (verified; status flagged).** The **Equal Credit Opportunity Act** and **Regulation B** (12 CFR 1002) prohibit credit discrimination and require **adverse action notices with specific reasons** (§1002.9). The CFPB has made clear this applies to AI models: **Circular 2022-3 (May 2022)** and **Circular 2023-3 (September 2023)** address adverse-action notices for credit decisions based on complex algorithms and AI — creditors **cannot** satisfy the notice requirement with a generic statement (e.g., "credit score"), and must provide the *specific* principal reasons; using a "black box" model does not excuse the obligation, and the bureau treats the adverse-action notice as a key AI-transparency tool. The **Section 1071 small-business lending rule** (final rule published 31 March 2023, implementing Dodd-Frank §1071 data collection) extended the framework to small-business credit; its data-collection provisions have been in litigation, and the CFPB has revised elements of Regulation B subpart B (including a final reconsideration rule referenced on the CFPB's own site, May 2026) **[verify: current effective status of 1071 data-collection requirements — materially in flux]**. For a wholesale bank: the practical obligation is a **reason engine** — per-decision, human-reviewable reason codes from the model (SHAP-based adverse-action analysis is the established practice; cross-ref `../technology/responsible_ai_frameworks_guide.md` §6.2 and `../technology/ai_llm/ai_governance_bias_redteaming_guide.md` §6.3).

**Other US threads (brief).** The OCC's **Comptroller's Handbook** and fair-lending examination procedures cover AI credit models; the CFPB supervises consumer-finance AI under UDAAP and ECOA; Executive Order 14110 (October 2023) — which gestured at NIST AI RMF alignment — was revoked in January 2025, leaving the US without a federal AI law: the binding US layer is therefore *existing* law applied to AI (ECOA, fair lending, MRM guidance, privacy at state level), not new AI-specific statute.

### 2.4 The Regulatory Table

| Instrument / source | Jurisdiction | Year / status | What it requires of AI in banking | Applies to |
|---|---|---|---|---|
| **EU AI Act** (Reg. 2024/1689) | EU | Adopted 2024; in force 1 Aug 2024; Annex III high-risk obligations from 2 Aug 2026; GPAI from 2 Aug 2025 | Risk-tier classification; full high-risk regime for credit-scoring AI (Annex III 5(b)); transparency for chatbots; GPAI supply-chain duties; fines to €35M/7% | Banks deploying or using AI in the EU market (any entity serving EU persons) |
| **MAS FEAT** (2018) + Veritas | Singapore | 12 Nov 2018; authoritative, non-binding | Fairness, Ethics, Accountability, Transparency; demonstrable bias management, board oversight, explainability | All FIs in Singapore, incl. branches of foreign banks |
| **MAS AI MRM Information Paper** | Singapore | 5 Dec 2024 (thematic review mid-2024) | Supervisory expectations for AI/GenAI model risk: governance, risk identification, validation, monitoring, GenAI controls | Banks in Singapore using AI/GenAI models |
| **MAS AIRM Guidelines** | Singapore | Consultation 13 Nov 2025; final status **[verify]** | Proposed binding-style guidelines on AI risk management for all FIs, complementing FEAT | All FIs (when finalised) |
| **MAS Notice 637 / TRM** | Singapore | Ongoing | Technology risk management, operational resilience, outsourcing — the envelope AI platforms run inside | All FIs |
| **SR 11-7 / OCC 2011-12** | US | 4 Apr 2011; **superseded 17 Apr 2026** | Model inventory, independent validation, effective challenge; the MRM vocabulary | US banks (Fed/OCC-supervised) |
| **SR 26-2 / OCC Bulletin 2026-13** | US | 17 Apr 2026 — **current** | Revised MRM guidance; risk-based approach; AI/GenAI/agentic expectations deferred to planned RFI **[verify]** | US banks, most relevant >$30B assets |
| **ECOA / Reg B** | US | 1974/2011; CFPB Circulars 2022-3, 2023-3 | Adverse-action notices with specific reasons for AI credit decisions; fair lending | Any creditor, incl. foreign banks' US lending |
| **GDPR** (Arts. 22, 35) | EU | 2018 | Automated decision-making rights; mandatory DPIAs for profiling-based decisions | Banks processing EU personal data |
| **PDPA + PDPC AI Guidelines** | Singapore | 2012 (am. 2020); AI Guidelines 1 Mar 2024 | Consent/legitimate-interest basis for AI use of personal data; breach notification; AI-specific advisory guidance | All SG organisations, incl. banks |
| **NIST AI RMF 1.0** | US (voluntary) | Jan 2023 | Govern–Map–Measure–Manage risk process; the de facto internal standard | Adopted voluntarily by banks |
| **ISO/IEC 42001** | Intl (voluntary) | Dec 2023 | Certifiable AI management system; AI system impact assessment; controls | Adopted voluntarily; useful certification |


---

### 2.5 The Operating Notes for a Global Bank (EU–SG–US)

For a bank that is Singapore-headquartered or Singapore-branched with EU entities (the Cymbal Bank pattern), the requirements do not attach by headquarters — they attach by **market and data subject**. The operating rules of thumb:

- **The EU AI Act follows the deployment, not the domicile.** A high-risk credit-scoring system used for EU persons is in scope regardless of where the model was built or the server sits; the Singapore branch serving EU private-banking clients runs Annex III analysis for any AI that evaluates their creditworthiness. The compliance answer is a single classification decision per system, applied across entities — not per-entity reinvention.
- **MAS expectations follow the FI in Singapore.** FEAT, the 2024 AI MRM information paper, and (when finalised) the AIRM guidelines apply to the Singapore entity and its AI use — including systems built by group technology centres abroad but *used* in Singapore. The MAS thematic review examined banks' AI including GenAI; expect the examination questions in the next TRM/MRM review cycle.
- **The US layer follows US lending and US persons.** ECOA/Reg B binds any credit decision in the US market; the group standard that most global banks adopt is to apply adverse-action reason quality *everywhere* (it is also what FEAT Transparency and GDPR Art. 22(3) want) — one reason-engine design serving three regulators.
- **Privacy follows the data.** GDPR binds processing of EU personal data wherever it occurs (Art. 3 extraterritoriality); PDPA binds SG personal data; the practical architecture is data-region-aware AI (EU data to EU processing, SG data to SG-region processing) rather than a single global model.
- **The "one bank, many registers" reality.** The bank will face MAS (AIRM/TRM), the ECB/EBA for EU entities (with the AI Act layered on), and — if US active — the Fed/OCC with SR 26-2. The sustainable pattern is **one AI inventory with per-jurisdiction obligation flags** — the inventory entry carries the EU-Act tier, the FEAT relevance, the PDPA/GDPR data scope, and the MRM status, so each supervisor's question is answered from the same record.

---

### 2.6 The AI Act's Overlap with Existing Financial Regulation

The EU AI Act does not replace financial regulation — it layers on top of it, and most of the Act's evidence already exists in some form inside a regulated bank. The overlaps a banking compliance function should map (the systems behind each live in `financial_risk_compliance_systems_guide.md`):

- **Credit.** CRR/CRD (IRB approval, the use test, validation), the **Mortgage Credit Directive** (MCD Art. 18 — creditworthiness assessment duties for mortgage loans), the **Consumer Credit Directive** (2023/2225 — including rules on automated decision-making in consumer credit), and IFRS 9 (ECL model governance) already impose documentation, data-quality, and oversight duties on credit models. The AI Act's high-risk regime (data governance, technical documentation, logging, human oversight) maps onto these: the bank is not building new machinery, it is extending the IRB/validation machinery to AI systems.
- **Conduct and markets.** MiFID II suitability and appropriateness (if AI touches investment advice), MAR (market abuse — relevant to AI-assisted trading), and the product-governance rules map onto the Act's transparency and oversight duties; the Act's Annex III does not list trading, so conduct law — not the AI Act — is the binding layer there (see §7.4).
- **AML.** The EU AML framework (AMLD6/AMLR) and the Act's fraud-detection carve-out interact: fraud/AML AI stays outside Annex III high-risk but inside the AML framework's own obligations (and inside model risk — the US SR 21-8→SR 26-2 line, §2.3).
- **The practical upshot.** Build the AI Act evidence by *extending existing control estates* — the IRB validation file becomes the high-risk technical documentation; the trade-surveillance logs become the record-keeping; the DPIAs become the impact assessments — rather than creating parallel compliance systems. Regulators themselves (EBA's AI discussion papers, ECB's supervisory expectations) push this integration reading: AI compliance is a re-pointing of the bank's existing risk-and-compliance machinery, plus the genuinely new bits (EU database registration, CE marking, GPAI supply-chain diligence, the Annex III classification discipline).

---

### 2.7 The DORA and Prudential-Supervisor Layer (EU)

Two more EU instruments bind banking AI without being AI law:

- **DORA — the Digital Operational Resilience Act (Regulation (EU) 2022/2554), applicable from 17 January 2025.** DORA makes ICT risk management, ICT-related incident reporting, and resilience testing first-class obligations for financial entities. An AI platform is an ICT system: it is in scope of the ICT risk-management framework (Art. 5–16), its failures are reportable ICT incidents (Art. 17–23 — major incidents with the regulator-notification clock), and — for significant entities — it sits inside the digital operational resilience testing programme (Art. 24–26), including threat-led penetration testing of critical systems. Third-party AI providers are ICT third parties (Art. 28–43): the register of information, contractual requirements, and exit-strategy duties apply to a foundation-model vendor no less than a core-banking SaaS. For the security requirements in §6, DORA is the binding wrapper in the EU that MAS Notice 637/TRM is in Singapore.
- **The prudential supervisors.** The ECB (for significant institutions) and the EBA treat AI through the existing prudential lenses — model risk (the EBA's guidelines on model risk management and the IRB/ICAAP expectations), outsourcing (EBA outsourcing guidelines), and SREP risk assessment. The EBA has signalled AI as a supervisory priority (including work on AI in creditworthiness assessment and the AML area) **[verify: current EBA AI deliverables and dates]**; the direction is that AI compliance will be examined under the prudential frameworks even where the AI Act does not apply (e.g., corporate-credit AI outside Annex III).

---

## 3. The Governance Requirements: Committees, Three Lines, Inventory, Model-Risk Integration

Governance is the requirement family the other four depend on: it is how the bank *decides* which AI to allow, *owns* the decision, and *proves* it. The governance machinery below is industry practice converging on what regulators (MAS FEAT Ethics/Accountability items, the MAS 2024 AI MRM information paper's governance-and-oversight observations, the 2026 US MRM guidance's governance chapter, ISO 42001 clauses 5–7) expect to see. The full governance-and-bias treatment lives in `../technology/ai_llm/ai_governance_bias_redteaming_guide.md` §1 and `../technology/responsible_ai_frameworks_guide.md` §5; this section is the banking-shaped version.

### 3.1 The AI Committees (Verify → Practice)

**Status: industry practice, not statute — the composition below is the pattern regulators recognise.** There is no law that dictates a bank's AI committee chart, but every serious AI-governance programme (and the MAS 2024 thematic review's good practices, and ISO 42001's leadership clause) assumes a standing governance body. The standard three-tier pattern:

- **Board / executive level.** The board (or a board risk/technology committee) owns the **AI risk appetite** — a one-page statement of which AI uses the firm will and will not pursue (e.g., no fully-automated credit decisions for consumers without human review; no autonomous money movement). It receives periodic AI status reporting: use-case inventory health, incidents, material findings. This is what MAS FEAT's Ethics principle (board-level oversight) demands, and what the proposed AIRM guidelines will formalise.
- **The AI governance / model risk committee (operational).** A management-level committee (typically chaired by the CRO or Chief AI Officer, with compliance, privacy, technology, and business representation) that: approves use cases above a materiality threshold, classifies risk tiers (including the EU-Act tier check), signs off impact assessments and bias/robustness test plans, and decides champion–challenger promotions and rollbacks. The model risk committee (existing in every bank under SR 11-7 / SR 26-2) is the natural parent — AI is a model.
- **The AI council / centre of excellence (advisory).** The horizontal group that sets standards (prompt-engineering guidelines, model-card templates, guardrail requirements), maintains the tooling, and triages new use-case requests before they reach the risk committee. It is also the body that keeps the *inventory* alive.

The three tiers correspond to the three governance layers in `../technology/responsible_ai_frameworks_guide.md` §5.1 (board/executive → policy/standards → working practice), and the RAI guide's §7 worked example shows the artifacts each tier produces (risk-appetite statement, committee decision records, incident reports).

### 3.2 The Three Lines of Defence (Verify → Cross-Ref)

**Status: the bank's standard control model (pervasive in banking supervision — Basel, MAS, SR 26-2's governance expectations), mapped to AI.** The three lines divide who *does*, who *challenges*, and who *audits*:

- **First line — the business and development teams.** Own the AI system: data preparation, model development, prompt design, deployment, day-to-day monitoring, and the first response to incidents. Under SR 11-7/SR 26-2 vocabulary they are the model *owners and users*; under the EU AI Act they are the *deployer* (Art. 26) or *provider* (if the bank builds and places the system on the market) with the corresponding obligations.
- **Second line — risk management, compliance, and model risk management (MRM).** Set the policies, challenge the first line, and validate: MRM performs the independent validation (conceptual soundness, ongoing monitoring, outcomes analysis — `risk_management_models_guide.md` §9.3), compliance owns the regulatory mapping (EU AI Act tier, ECOA adverse-action, GDPR Art 22), and privacy office owns the DPIAs. Independence is the point: validators must not have built or used the model.
- **Third line — internal audit.** Independent assurance that the first two lines are actually operating: samples the inventory, re-runs a validation, checks that incidents were reported, tests whether the committee decisions have teeth. External auditors and supervisors (MAS thematic reviews, Fed/OCC examinations) sit conceptually above the three lines.

The AI-specific twist: GenAI collapses the usual boundaries (the same team writes the prompt, builds the RAG pipeline, and operates the assistant), so the three lines must be *explicitly* re-drawn for each AI system in the inventory — who is the accountable owner, who validates, who audits — and the MAS 2024 information paper's governance observations are essentially this re-drawing exercise.

### 3.3 The AI Inventory (Verify → Practice)

**Status: practice with a regulatory spine — SR 11-7/SR 26-2 require a model inventory; ISO 42001 requires records; the EU AI Act requires registration of high-risk systems in the EU database; MAS expects a use-case inventory.** The inventory is the single source of truth that reconciles what the MRM registry, the compliance use-case list, and the platform's model registry would otherwise each hold separately (the RAI guide's §5.4 point: without one inventory, risk assessment runs on rumours). A banking AI inventory entry carries:

- **Identity:** system name, owner (first line), validator (second line), inventory ID, version, deployment status, location of the model package.
- **Classification:** risk tier (EU Act tier incl. Annex III check; FEAT relevance; internal materiality tier), model family (statistical ML, GenAI/LLM, agent), decision type (advisory vs. automated vs. agentic action-taking).
- **Artifacts:** model card, data cards/datasheets, the ISO 42001-style AI system impact assessment, DPIA reference, bias and robustness test reports, committee sign-off record, monitoring plan, incident log.
- **Lifecycle state:** development → validation → approved → production → retired, with change history (a retrained model is a change event; a material change is a new inventory entry — `risk_management_models_guide.md` §10.2).

For the agent era, entries must extend to agent definitions, tool permissions, and human-oversight design — the governance angle treated in `../technology/beyond_zero_enterprise_security_guide.md`.

### 3.4 The Model-Risk Integration (Verify → Cross-Ref)

**Status: verified as the governing expectation** — SR 11-7's model definition covers AI/ML models, the 2026 revised guidance (SR 26-2 / OCC 2026-13) keeps MRM as the umbrella and adds vendor/third-party considerations, and MAS's December 2024 information paper applies the same discipline to AI and GenAI at Singapore banks. The integration rules a bank should encode:

1. **Every AI system that converts data into decisions is a model** — scorecards, ML PD/LGD models, LLM-assisted underwriting, fraud detectors, chatbots that produce advice. The SR 11-7 definition does not care whether the weights are logistic or transformer.
2. **The full lifecycle applies:** development-with-governance-by-design (data versioning, experiment tracking, a formal development report), adapted validation (conceptual soundness → architecture and feature audit; ongoing monitoring → drift/PSI/calibration; outcomes analysis → realised-vs-predicted backtesting), and controlled retraining (change management triggers re-validation scoped by materiality).
3. **GenAI strains the classical discipline in known ways** (the five documented strains in `risk_management_models_guide.md` §10: explainability, bias/fairness, overfitting/drift, adversarial robustness, regulatory acceptance), and the 2024 MAS paper adds GenAI-specific expectations: validation of LLM systems (prompt/guardrail testing, hallucination measurement), human oversight of outputs, and logging/auditability of prompt→model→output flows.
4. **Vendor models are in scope.** The 2026 US guidance explicitly covers third-party products; the EU AI Act's deployer duties and MAS outsourcing rules do the same. A foundation-model API from a vendor is a third-party model: it needs a procurement assessment, an inventory entry, and monitoring — the bank cannot outsource the accountability.

### 3.5 The Governance Table

| Governance element | What it requires | Source / expectation | Banking evidence |
|---|---|---|---|
| AI committees | Board-level risk appetite; management committee approvals; standards council | FEAT Ethics; MAS AI MRM paper (governance & oversight); ISO 42001 §5 | Risk-appetite statement; committee minutes; sign-off records |
| Three lines of defence | Owner / validator / auditor separation | SR 26-2 governance; Basel/MAS control model | RACI per system; independent validation reports; audit findings |
| AI inventory | Single reconciled register with classification and artifacts | SR 26-2 (model inventory); EU AI Act Art. 49 (registration); ISO 42001 records | Inventory entries; model cards; DPIA refs; incident log |
| Model-risk integration | AI systems through the MRM lifecycle; vendor models in scope | SR 26-2 / legacy SR 11-7; MAS 2024 info paper; EU AI Act Arts. 26, 34 | Validation reports; monitoring dashboards; vendor assessments |

---

## 4. The Risk Requirements: Model Risk for AI and the GenAI Risk Taxonomy

### 4.1 The Model-Risk Management for AI (Cross-Ref)

The risk requirement is twofold: the AI system must survive the bank's **model-risk discipline** (it is a model), and the bank must manage the **AI-specific risks** the discipline was not designed for. The full MRM treatment — SR 11-7's three validation elements, the model package, the five ML strains, champion–challenger, the override economy — lives in `risk_management_models_guide.md` §9–10 and is not re-derived here. The banking-AI reading of it:

- **ML in risk has a settled architecture pattern** (`risk_management_models_guide.md` §10): keep capital-direct engines classical (logistic scorecard → IRB), run ML as challenger/early-warning/enrichment layers, use ML outputs to *inform* rather than *determine*, and put every ML model through the full inventory–validation–monitoring lifecycle. The same pattern transfers to GenAI: LLM outputs feed *recommendations* with human sign-off, not autonomous determinations — until the validation evidence justifies more.
- **Retraining is a controlled event** — a model *change* under SR 11-7/SR 26-2, triggering re-validation scoped by materiality; the "when does a retrained model become a new model?" question is answered by change-management policy (new population/features/methodology ⇒ new inventory entry).
- **Human oversight and the override economy.** Every ML risk model ships with human-in-the-loop controls: monitored overrides, decision logs, escalation thresholds; the override rate is itself a monitoring metric. This is also the EU AI Act Art. 14 design hook and the GDPR Art. 22 safeguard hook (human intervention) — one control serving three masters.
- **MAS's AI MRM expectations (December 2024)** confirm this is the supervisory bar in Singapore: banks examined on governance and oversight, risk identification and measurement, and controls for AI and GenAI — with LLM-specific controls (guardrails, hallucination monitoring, human review) named as good practice.

### 4.2 The GenAI Risk Taxonomy (Cross-Ref)

The taxonomy below is the **banking translation** of the risk content owned by `../technology/llm_development_risks_security_guide.md` (OWASP LLM Top 10 mapping, attack mechanics, defense-in-depth architecture) — cross-ref that guide for the mechanisms and controls; here the focus is what each risk *does to a bank* and which requirement it triggers.

- **Hallucination (LLM09 — Overreliance/Misinformation).** The model fabricates fluent falsehoods: invented regulations, wrong product rates, fabricated client facts, false citations. In banking the harm is direct: a relationship manager repeating a hallucinated answer as advice is a conduct and mis-selling event; an LLM-drafted SAR or credit memo with fabricated figures is a compliance failure; automation bias means downstream pipelines inherit the hallucination. Controls: retrieval-grounded answers with citations, factual-accuracy evaluation per release (hallucination rate as a first-class metric), output validation, human review of anything client- or regulator-facing. Requirement triggers: EU AI Act accuracy/robustness (Art. 15), FEAT Transparency (explainability of materially-influenced advice), model-risk monitoring, conduct rules.
- **Prompt injection (LLM01 — direct and indirect).** Malicious instructions — direct (user prompt) or indirect (hidden in retrieved documents, emails, web pages an agent reads) — hijack the model into unintended behaviour: exfiltrating data, triggering tool calls, ignoring guardrails. For a bank the highest-severity form is **indirect injection through RAG content** (a poisoned document in the knowledge base) and **agentic injection** (an injected instruction causing a tool call — a transfer, a message, a data export). There is no complete defence, only layered reduction (input/output filtering, privilege separation, tool-permission minimisation, human confirmation gates). Requirement triggers: EU AI Act Art. 15 cybersecurity; MAS TRM/Notice 637 operational resilience; the security cluster's controls.
- **Data leakage (LLM06 — sensitive information disclosure; RAG leakage).** Training data, retrieved documents, or conversation context leaking across boundaries: an LLM echoing memorised PII from training data; a RAG system returning documents the requesting user is not entitled to (the retrieval index has no inherent authorisation concept); conversation history leaking between tenants or sessions; logs capturing prompts containing client data. For a bank this is simultaneously a **privacy breach** (GDPR/PDPA), a **confidentiality breach** (client mandates, M&A data), and a **market-abuse hazard** (inside information). Controls: permission-aware retrieval, PII/secret scanning on inputs and outputs, data minimisation, tenant isolation, log redaction. Requirement triggers: GDPR Arts. 5, 25, 32; PDPA; Art. 15 cybersecurity.
- **Model drift.** Input, concept, and behaviour drift: populations and economies move (a credit model trained on pre-recession data decays), features silently stop updating (a broken feed — the classic silent model failure), and LLM behaviour drifts with upstream updates (a vendor model change alters output distribution with no code change). Controls: PSI/CSI-style stability indices, feature-missingness monitoring, calibration tracking, and — for LLMs — pinned model versions with re-evaluation on any vendor upgrade. Requirement triggers: MRM ongoing monitoring (SR 26-2 / legacy SR 11-7; MAS 2024 paper), EU AI Act Art. 15 robustness.
- **Excessive agency (LLM06:2025 — agent era).** An agent with broad tool permissions converts every other vulnerability (injection, hallucination) into a *real-world action*: a single injected instruction in an overly autonomous agent can trigger a money transfer or data deletion. The banking rule is strict: no agent moves money or executes irreversible actions without human confirmation gates; tool permissions are minimised per agent; kill switches and full audit trails are mandatory. Cross-ref `../technology/beyond_zero_enterprise_security_guide.md` for the agent-governance treatment.
- **Bias and fairness (cross-ref the governance guide).** LLMs inherit and amplify training-data bias; in credit-adjacent GenAI (underwriting summaries, chatbot eligibility advice) this recreates the fair-lending hazards of ML scorecards — measured with the same tools (AIF360 disparate-impact screen, group metrics, declared fairness criterion) in `../technology/ai_llm/ai_governance_bias_redteaming_guide.md` §4–5. Requirement triggers: FEAT Fairness, ECOA/Reg B, EU AI Act Art. 10 data governance.

### 4.3 The Risk Table

| Risk | Banking impact | Primary controls | Requirements triggered | Cross-ref |
|---|---|---|---|---|
| Hallucination / misinformation | Fabricated advice, memos, SARs; conduct and mis-selling events | RAG grounding, citation checks, hallucination-rate evaluation, human review | EU AI Act Art. 15; FEAT Transparency; MRM monitoring | `llm_development_risks_security_guide.md` LLM09 |
| Prompt injection (direct/indirect) | Data exfiltration, rogue tool calls, guardrail bypass | Layered input/output filtering, privilege separation, tool minimisation, confirmation gates | EU AI Act Art. 15; MAS TRM | `llm_development_risks_security_guide.md` LLM01 |
| Data leakage (incl. RAG) | Privacy breach, confidentiality breach, market-abuse hazard | Permission-aware retrieval, PII/secret scanning, tenant isolation, log redaction | GDPR Arts. 5/25/32; PDPA; Art. 15 | `llm_development_risks_security_guide.md` LLM06 |
| Model drift | Decayed decisions, silent failures, stale features | PSI/CSI, calibration tracking, pinned LLM versions with re-evaluation | MRM (SR 26-2; MAS 2024); Art. 15 | `risk_management_models_guide.md` §10 |
| Excessive agency | Unauthorised money movement / irreversible actions | Human confirmation gates, minimised tool permissions, kill switches | Art. 15; agent governance | `beyond_zero_enterprise_security_guide.md` |
| Bias / unfairness | Fair-lending violations, reputational damage | 80%-rule screen, group metrics, threshold mitigation, monitoring | FEAT Fairness; ECOA/Reg B; Art. 10 | `ai_governance_bias_redteaming_guide.md` §4–5 |

---

### 4.4 The AI Risk Appetite and Limits in Practice

The governance table (§3) and the risk taxonomy (§4.2) come together in the **AI risk appetite statement** — the document the board owns and the committees enforce. A banking AI risk appetite typically contains hard lines of this kind:

- **No fully automated consumer or SME credit decisions** without meaningful human review (the Art. 22 "not solely" line, the EU AI Act Art. 14 line, the ECOA adverse-action line — one rule, three regulators).
- **No agentic money movement** — no AI-initiated transfer, payment, or irreversible instruction without a human confirmation gate and a kill switch (the excessive-agency line).
- **Client-facing GenAI outputs are grounded and reviewed** — no unverified claims about products, rates, or regulatory positions; hallucination rate below a declared threshold per release (the hallucination line).
- **Personal data in AI pipelines is minimised** — no special-category data in automated decisions; prompts to external GPAI vendors carry only what the task requires (the privacy line).
- **Every AI system is in the inventory** before production; unregistered systems are shadow AI and are switched off (the governance line).

From the appetite statement, the limits cascade into **KRIs with thresholds**: override rates (a spike signals drift), hallucination rates per system, injection-attempt counts, RAG leak incidents, model performance metrics (PSI, calibration), and incident counts. Crossing a threshold triggers a defined action — model re-validation, committee review, or suspension. This is the same machinery as classical risk limits (the limits discipline in banking_limits_domain_guide.md applied to AI), and it is what makes "risk appetite" auditable rather than aspirational. The cost of the controls is a business-case question — ../management/business_case_development_guide.md carries the cost/benefit framing for the governance spend.

---

### 4.5 The Risk-Assessment Procedure for GenAI in Banking

The taxonomy (§4.2) names the risks; the procedure scores and manages them. The five-step procedure, composed from the trust guide's §7 assessment process and the RAI guide's §5.3 risk-assessment procedure:

1. **System description and threat model.** Describe the system (model, data, retrieval, tools, autonomy level, users, data subjects) and run the taxonomy check: hallucination (always — it is the baseline GenAI risk), prompt injection (if any untrusted input reaches the model — user prompts, retrieved documents, emails, web content), data leakage (if personal/confidential data flows through prompts, RAG, or logs — nearly always in banking), drift (always — models and features decay), excessive agency (if the system can act — tools, messages, transfers).
2. **Likelihood × impact scoring.** Score each applicable risk on a banking impact scale: financial loss (direct P&L, fines), regulatory breach (which requirement family), client harm (mis-selling, unfair decisions), reputational damage (press, regulator attention). The 5×5 matrix produces a risk score per system; high scores force controls, not acceptance.
3. **Controls mapping.** Map each scored risk to its controls from the §4.3 table (grounding and review for hallucination; layered filtering and privilege separation for injection; permission-aware retrieval and PII scanning for leakage; monitoring for drift; confirmation gates and kill switches for agency) — and verify the control is actually implemented, not just listed.
4. **Residual risk and sign-off.** Subtract controls; the residual must sit inside the AI risk appetite (§4.4). Residual above appetite → committee decision (redesign, restrict scope, or decline) — never silent acceptance.
5. **Monitoring plan.** Define the KRIs per risk (hallucination rate, injection attempts, leak incidents, PSI, override rates), thresholds, and the escalation path — the standing evidence for the "ongoing" obligations in §9.

This is the same shape as the ISO 42001 AI risk assessment and the EU AI Act Art. 9 risk-management system (cross-ref `../technology/ai_trust_assessments_guide.md` §2.3, §2.2) — the bank runs one procedure and cites it to all three.

---

## 5. The Privacy Requirements: GDPR Article 22, DPIA, PDPA, Training-Data PII

Privacy is where AI compliance and data-protection law collide most literally: the same model is a *decision system* (Art. 22), a *processing operation* (Art. 35), a *training-data consumer* (PII in corpora), and a *retrieval system* (PII in RAG). The data-governance lineage treatment lives in `../technology/data_governance_guide.md`; the privacy-relevant banking facts are below.

### 5.1 The GDPR Article 22 — Automated Decision-Making (Verified)

**Verified (gdpr-info.eu; EDPB guidance WP251rev.01):** Article 22(1) grants the data subject the *"right not to be subject to a decision based solely on automated processing, including profiling, which produces legal effects concerning him or her or similarly significantly affects him or her."* The banking reading:

- **Credit decisions are the canonical Art. 22 case** — a fully automated credit refusal "significantly affects" the applicant; a fully automated eligibility decision for a product with legal effects triggers the right. **The key word is "solely":** any meaningful human involvement in the decision removes it from Art. 22(1) — but the involvement must be real (a rubber-stamp reviewer does not count; the EDPB's guidance is explicit that the human must have authority and capacity to change the decision).
- **The three exceptions (Art. 22(2)):** (a) necessary for entering into or performing a contract (the usual basis for credit decisions — the automated decision is needed to conclude the loan contract); (b) authorised by EU or Member State law with suitable safeguards; (c) based on the data subject's explicit consent.
- **Safeguards (Art. 22(3)):** for contract- or consent-based automated decisions, the controller must implement *"suitable measures to safeguard the data subject's rights and freedoms and legitimate interests"* — at minimum the right to **human intervention**, to **express views**, and to **contest the decision**. This is the GDPR hook for the human-override and appeal machinery in credit underwriting — and it mirrors the EU AI Act's Art. 14 human oversight and the ECOA adverse-action right to reasons.
- **Special categories (Art. 22(4)):** automated decisions based on special-category data (health, etc.) are prohibited unless the data subject gave explicit consent and suitable safeguards exist — rarely workable in lending, so the design rule is: keep special-category data out of the automated decision pipeline entirely.
- **Interplay with the EU AI Act:** the Act (Art. 86 right to explanation, and the high-risk regime's transparency) and Art. 22 operate in parallel — an AI credit decision must satisfy both the GDPR's decision-making safeguards and the Act's high-risk obligations. EDPB and the EU AI Office have signalled joint guidance is coming; until then, the conservative reading (treat high-risk AI credit decisions as Art. 22-relevant, provide human review + reasons + contest rights) is the safe design.

### 5.2 The Article 35 DPIA (Verified)

**Verified:** Article 35(1) requires a **Data Protection Impact Assessment** where processing *"is likely to result in a high risk to the rights and freedoms of natural persons,"* and Article 35(3)(a) makes the DPIA **mandatory** for *"a systematic and extensive evaluation of personal aspects relating to natural persons which is based on automated processing, including profiling, and on which decisions are based that produce legal effects... or similarly significantly affect"* — i.e., **Art. 22-type AI decision-making always triggers a DPIA**. Banking practice:

- **DPIA before processing, iterated.** The DPIA must be conducted before deployment and revisited on material change (a new data source, a new model, a new retention). The EDPB's DPIA guidelines (WP248rev.01) and the Article 29 Working Party's list of high-risk criteria (profiling, large scale, innovative technology, data about vulnerable data subjects, etc.) give the triggers; banks tick several boxes for most AI systems.
- **Art. 36 consultation:** where the residual risk after mitigation remains high, the supervisory authority must be consulted before processing begins — a rare but real step for high-risk AI in banking.
- **The AI-management-system parallel:** ISO/IEC 42001's **AI system impact assessment** is the AI-native sibling of the DPIA (cross-ref `../technology/ai_trust_assessments_guide.md` §2.3) — running both, or one combined assessment satisfying both frames, is the standard bank pattern (the impact assessment covers safety/fairness/security dimensions the DPIA does not; the DPIA covers the data-protection dimensions the impact assessment may not).

### 5.3 The PDPA (Singapore — Cross-Ref)

**Verified facts:** the **Personal Data Protection Act 2012** (in force 2014, with the 2020 amendments — including mandatory **data breach notification**, effective from February 2022, and enhanced financial penalties up to 10% of local annual turnover for the most serious breaches) governs personal data in Singapore, administered by the **PDPC**. The banking-relevant layer for AI:

- **The PDPC's Advisory Guidelines on Use of Personal Data in AI Recommendation and Decision Systems (1 March 2024 — verified)** clarify how the PDPA applies to AI: consent remains the default basis, with the **business improvement** and **research** exceptions relevant to model training and analytics; organisations must be able to document the appropriate basis for each AI use of personal data; and the Guidelines address data intermediaries (including third-party AI developers) and best practices — including making meaningful human review available for significant automated decisions (the PDPA does not have an Art. 22 analogue, but the Guidelines nudge the same direction).
- **Consent and notification in practice:** collecting personal data to train a model needs consent or a documented exception; using customer data for a *new* AI purpose (e.g., training a chatbot on transaction histories) is a new purpose requiring fresh notice/consent assessment; cross-border transfer rules (Part 9A) matter when foundation models are hosted overseas.
- **The MAS overlap:** for banks, MAS Notice 637 (TRM) and the MAS data-protection expectations (e.g., on outsourcing and cloud) layer on top of the PDPA — a GenAI deployment on a hyperscaler must clear both the PDPA transfer rules and MAS outsourcing rules (`banks_in_singapore_guide.md` §7 carries the context).

### 5.4 The Training-Data PII (Flagged — Practice, Not Regulation)

**Status: flagged — the mechanics below are established industry practice synthesised from the privacy and LLM-security literature; the specific regulatory treatment of LLM training data is still evolving (EU AI Act training-data summaries; litigation over web-scraped training data; PDPC guidance in its early application).** The problem: foundation models are trained on corpora that include personal data (and copyright content), and models can memorise and emit training data on request. The banking obligations that follow:

- **Know what your model was trained on.** For third-party foundation models, the vendor's training-data documentation (EU AI Act Art. 53 requires providers to publish training-data summaries) is the due-diligence input; for fine-tuning, the bank controls the data and must apply the same purpose/consent analysis as any processing.
- **Fine-tuning on bank data is a privacy event.** Fine-tuning on customer data (e.g., transaction-derived examples) requires the same lawful-basis documentation as any processing; the industry-safe pattern is **synthetic data and de-identified examples** for tuning, with re-identification risk assessment (de-identification is not anonymisation — cross-ref `../technology/data_governance_guide.md`).
- **Prompt data is personal data.** Every prompt containing client data that leaves the perimeter (vendor API) is a transfer; the practical controls are data-minimised prompts, on-prem/private deployment for client data, and log redaction (this is also the security cluster's data-leakage control — one control, two requirement families).
- **Memorisation risk in production.** RAG systems and fine-tuned models can regurgitate training PII; controls are output PII scanning and, where possible, training-data filtering — the same scanners the security cluster lists (`../technology/llm_development_risks_security_guide.md` — LLM Guard et al.).

### 5.5 The Privacy Table

| Requirement | Source | What it demands of AI in banking | Evidence |
|---|---|---|---|
| No solely-automated significant decisions without safeguards | GDPR Art. 22 | Human intervention, express-views, contest rights; document the non-"solely" human involvement | Decision logs, override records, appeals process |
| DPIA before high-risk AI processing | GDPR Art. 35 (esp. 35(3)(a)) | Assessment before deployment, iterated on change; Art. 36 consultation if residual high risk | DPIA register, risk scores, mitigation records |
| Lawful basis for AI use of personal data | PDPA + PDPC AI Guidelines (2024) | Consent or documented exception (business improvement/research); new-purpose analysis; breach notification | Basis documentation, notices, breach register |
| Training/fine-tune data hygiene | GDPR/PDPA (practice) | PII-aware training data, synthetic/de-identified tuning data, vendor training-data due diligence | Data provenance, de-identification assessments |
| Data protection by design | GDPR Art. 25; PDPA; ISO 42001 | Minimisation, isolation, PII scanning, redaction built into the AI pipeline | Design reviews, architecture decisions |

---

### 5.6 The Cross-Border AI Data Flows (EU–SG)

A GenAI deployment that sends prompts to a vendor API, or fine-tunes on data in another region, is a **cross-border data transfer** under both regimes, and the two regimes do not align:

- **GDPR Chapter V:** transfers of EU personal data outside the EEA require an adequacy decision, standard contractual clauses (SCCs), or another Art. 46 mechanism. Singapore does **not** hold an EU adequacy decision (as of this writing — the EU–Singapore adequacy process has been discussed but not concluded) **[verify: current adequacy status]**, so EU client data sent to a Singapore-hosted model or an SG-incorporated vendor needs SCCs in the vendor contract, plus a transfer impact assessment. Where the vendor is US-based, the EU–US Data Privacy Framework (and its validity saga) is the relevant mechanism to check.
- **PDPA Part 9A:** transfers of SG personal data outside Singapore require the transfer limitation obligations to be met (comparable protection via contract or certification); MAS outsourcing rules add the layer for banks — a cloud-hosted GenAI platform is an outsourcing arrangement requiring MAS-compliant contracts, business-continuity provisions, and (for material outsourcings) notification.
- **The banking-safe pattern:** keep personal data in-region where the processing happens (EU data → EU region model deployment; SG data → SG region); minimise prompt content (client names and deal terms stripped to identifiers or synthetic placeholders where possible); prefer on-prem/private deployments for client-data workloads; and document every vendor transfer in both the DPIA and the PDPA records. The same controls serve the security requirement (data leakage) — the recurring theme of this guide.

---

### 5.7 The Banking DPIA Outline

When the privacy office opens a DPIA for an AI system (mandatory under Art. 35(3)(a) for profiling-based decisions), the banking-shaped outline is:

1. **System and data flows** — what the AI does, what personal data enters (training, fine-tuning, prompts, retrieval), where it is processed (region, vendor), who has access, retention.
2. **Lawful basis** — GDPR Art. 6 basis per purpose (contract/legitimate interest/consent), the Art. 22 analysis (is any decision solely automated? which exception? which safeguards?), and the PDPA basis (consent or the business-improvement/research exceptions) where SG data is involved.
3. **Necessity and proportionality** — why the AI needs the data it uses; is the data minimised (can prompts be stripped of client names? can training use synthetic data?).
4. **Risk identification** — the privacy risks specific to AI: profiling and automated decisions, special-category data (should be excluded by design), re-identification from de-identified training data, memorisation/regurgitation, cross-border transfers (Chapter V / Part 9A), over-retention of prompts and logs, and vendor access.
5. **Measures** — minimisation, pseudonymisation, encryption in transit/at rest, access control and tenant isolation, PII scanning on outputs, log redaction, human review of significant decisions, retention limits, vendor SCCs/DPA.
6. **Residual risk and Art. 36 check** — if residual risk to data subjects remains high, the DPA must be consulted before processing; document the conclusion either way.
7. **Sign-off and triggers** — DPO approval, and the review triggers (new data source, new model, new vendor, new jurisdiction — each re-opens the DPIA, matching the ISO 42001 impact-assessment trigger on significant change).

---

## 6. The Security Requirements: Secure-AI Development and the Adversarial

Security for AI in banking is the intersection of two existing disciplines — the bank's secure-SDLC and the AI-specific threat model. The full mechanics live in the security cluster (`../technology/security_by_design_guide.md` for secure development, `../technology/cybersecurity_guide.md` §8 for the banking security context, `../technology/adversarial_ml_attacks_guide.md` for adversarial ML, `../technology/llm_development_risks_security_guide.md` for the LLM layer); this section states the *requirements* those mechanics satisfy.

### 6.1 The Secure-AI Development (Cross-Ref)

The requirement: AI systems are built, deployed, and operated with the same — and in places stricter — security rigour as the bank's core systems, plus AI-specific controls. Concretely:

- **Secure SDLC applied to AI:** threat modelling at design (cross-ref `security_by_design_guide.md` §2 — with the AI-specific threat model from the LLM-security guide: prompt injection, data leakage, excessive agency added to the standard STRIDE-style set); secure coding and dependency management (a bank's AI platform is Python/dependency-heavy — supply-chain scanning is non-negotiable); security testing in CI (SAST/DAST plus LLM-specific scanning — garak, prompt-injection suites); and a DevSecOps pipeline where model promotion and deployment follow the same gates as code (`security_by_design_guide.md` §6).
- **Model and agent supply chain:** the foundation-model API, the fine-tune base, the open-source model weights, the vector database, the orchestration framework — each is a third-party component requiring procurement security assessment, version pinning, and vulnerability tracking. MAS outsourcing rules and the 2026 US MRM guidance's vendor chapter make this a supervisory expectation, not just good practice.
- **The GenAI control architecture** (cross-ref `llm_development_risks_security_guide.md` §12 — the six-layer defense-in-depth): perimeter (access, tenant isolation), input (prompt filtering, injection detection), retrieval (permission-aware RAG), model (pinned versions, guardrails), output (validation, PII/secret scanning, citation checks), monitoring (full tracing, anomaly detection, incident response). For banking, the monitoring layer doubles as the compliance audit trail (EU AI Act record-keeping; FEAT accountability).
- **Agent-era controls** (cross-ref `beyond_zero_enterprise_security_guide.md`): identity for agents, minimised tool permissions, human confirmation gates on irreversible actions, kill switches, and full action logging — the security translation of the "no autonomous money movement" governance rule.

### 6.2 The Adversarial (Cross-Ref)

The requirement: AI models must be robust against deliberate attack — and in banking the adversaries are not hypothetical. The three canonical attack classes (full mechanics in `../technology/adversarial_ml_attacks_guide.md`):

- **Evasion** — crafting inputs that defeat the model at inference: fraudsters probing ML transaction monitors to find the feature combinations that pass (the adversarial arms race in AML), subtle perturbations to documents that an LLM reviewer misreads. Controls: adversarial robustness testing in validation, continuous monitoring for detection degradation, red-team exercises.
- **Poisoning** — contaminating training/fine-tuning data so the model learns the attacker's agenda: a poisoned fine-tune corpus that makes an underwriting assistant systematically favour certain applicants, or a poisoned RAG document that injects instructions (indirect prompt injection — the two attack families converge). Controls: data provenance and integrity checks, fine-tune data vetting, retrieval-source trust, and the input-layer filtering from the LLM-security architecture.
- **Extraction / inversion** — stealing the model or its data: API-based model stealing, training-data extraction (memorised PII), membership inference. Controls: access control and rate limiting on model endpoints, output PII scanning, and (for high-value models) watermarking.

**Why this is a *regulatory* requirement, not just a security one:** the EU AI Act Art. 15 makes robustness and cybersecurity a legal obligation for high-risk systems; SR 26-2's ongoing-monitoring expectations cover model degradation from attack; MAS's AI MRM paper and TRM rules expect resilience to adversarial inputs; and ECOA/Reg B fair-lending analysis must survive the adversarial reality that fraudsters probe credit models too. Red-teaming and adversarial testing are thus compliance evidence, not optional hardening — the governance guide's §7 red-teaming section and the trust guide's §5 treat them as assessment methods precisely because regulators count them.

### 6.3 The Security Table

| Security requirement | Banking context | Primary controls | Cross-ref |
|---|---|---|---|
| Secure-AI SDLC | AI pipelines gated like code; supply-chain scanning for ML/LLM dependencies | Threat modelling, SAST/DAST, LLM scanners (garak), DevSecOps gates | `security_by_design_guide.md`; `llm_development_risks_security_guide.md` |
| Model/agent supply chain | Vendor LLMs, weights, vector DBs, orchestration as third-party components | Procurement assessment, version pinning, vulnerability tracking, MAS outsourcing rules | `cybersecurity_guide.md` §8; SR 26-2 vendor chapter |
| Adversarial robustness | Fraudsters probe ML monitors; poisoned fine-tune data; model theft | Evasion/poisoning/extraction testing, red-teaming, endpoint controls, watermarking | `adversarial_ml_attacks_guide.md` |
| GenAI runtime controls | RAG leakage, injection via retrieved content, agent actions | Six-layer defense-in-depth: input/output filtering, permission-aware RAG, tracing, kill switches | `llm_development_risks_security_guide.md` §12 |
| Monitoring & incident response | Detection as the last line; compliance audit trail | Full tracing, anomaly detection, injection-attempt logging, incident playbooks | `cybersecurity_guide.md` §3; EU AI Act Art. 12 |


---

### 6.4 The AI Security Operating Model

Security requirements are only as real as the operating model that carries them. The banking pattern:

- **Roles.** Every AI system in the inventory has a named security owner (first line), an AI-security review in the second line (CISO/security architecture), and AI in the internal audit plan (third line). A central **AI-SOC function** monitors the AI estate: injection-attempt logs, anomalous tool calls, unexpected data egress, guardrail bypasses, cost anomalies, drift in refusal behaviour (the detection signals from ../technology/llm_development_risks_security_guide.md §17).
- **Incident playbooks.** AI incidents are security incidents first: a **prompt-injection attack** (contain: revoke tokens, isolate the RAG store, preserve logs), **data leakage** (contain: suspend the system, assess GDPR 72-hour / PDPA breach notification obligations, notify affected clients), **model poisoning** (contain: quarantine the fine-tune, re-validate, roll back), **agent runaway** (contain: kill switch, revoke tool permissions, audit the action trail). Each playbook names who declares, who investigates, and who tells which regulator — the EU AI Act's serious-incident reporting (for GPAI providers and high-risk deployers), MAS Notice 637 incident reporting, and GDPR/PDPA breach notification all run on the same trigger.
- **The continuous cadence.** Red-team exercises on a schedule (adversarial and injection suites from adversarial_ml_attacks_guide.md and the LLM-security guide), benchmark re-runs on every model or prompt change, supply-chain vulnerability alerts on the AI stack, and an annual security re-assessment per inventory entry.

---

### 6.5 The AI-Security Testing Checklist

The concrete checklist a banking AI system must pass before production (each item's mechanics in the security cluster — `../technology/security_by_design_guide.md` §5, `../technology/adversarial_ml_attacks_guide.md`, `../technology/llm_development_risks_security_guide.md` §17):

- **Supply chain** — dependency and model-weights vulnerability scan; vendor security assessment for the LLM API; version pinning verified.
- **Threat model** — the standard threat model updated with the AI-specific entries (prompt injection, data leakage via RAG, excessive agency, model DoS) and reviewed by security architecture.
- **Prompt-injection suite** — direct injection attempts, indirect injection via planted RAG documents, encoding obfuscation, multiturn grooming; pass = injections contained (no tool call, no data exfiltration, no guardrail bypass).
- **Jailbreak robustness** — roleplay/hypothetical/refusal-chaining probes; pass = refusal behaviour within tolerance.
- **PII/secret exfiltration test** — attempts to elicit memorised PII, system prompts, or secrets; output scanner catches them.
- **RAG permission-boundary test** — cross-tenant queries must return nothing the requesting user cannot access; retrieval authorisation verified end-to-end.
- **Agent controls (if acting)** — tool-permission matrix review; human confirmation gates exercised in a drill; kill switch tested; full action trail verified.
- **Output validation** — schema/format validation, citation checks for grounded answers, content filtering verified.
- **Monitoring and tracing** — prompt→model→tool→output tracing verified; injection-attempt and anomaly alerts firing on test signals.
- **Red-team report** — findings, severities, and remediation tracking to closure; the report is retained as validation evidence (Art. 15 robustness and MRM monitoring both cite it).

---

## 7. The Use-Case Mapping: Credit Scoring, AML, Chatbots, Trading

The core deliverable of a requirements map: **which requirements bind which use case.** The pattern to internalise: requirements attach by (a) what the system decides (credit = high-risk + fair lending + Art. 22; AML = carve-out but heavy model risk + privacy; chatbots = transparency + GPAI + data leakage; trading = model risk + market conduct), (b) whose data it touches (EU persons ⇒ GDPR; SG persons ⇒ PDPA), and (c) whether it acts (agents ⇒ security/oversight escalation).

### 7.1 Credit Scoring (Including GenAI-Assisted Underwriting)

**The most heavily bound use case in banking.** Requirements: **EU AI Act Annex III 5(b) high-risk** (creditworthiness of natural persons — verified; fraud detection carve-out does not apply to lending decisions) with the full regime from 2 August 2026; **ECOA/Reg B adverse action** with specific reasons (CFPB Circulars 2022-3, 2023-3) and fair-lending analytics (80%-rule screens, group metrics, reason engines — `../technology/responsible_ai_frameworks_guide.md` §6.2); **GDPR Art. 22** safeguards (human intervention, contest rights) and mandatory **DPIA** (Art. 35(3)(a)); **FEAT** fairness and transparency; full **model-risk lifecycle** (a scorecard is the archetypal SR 11-7 model — and the 2026 guidance's risk-based approach keeps it front and centre); **PDPA/PDPC** basis for any SG personal data in the model; and **security** (evasion-resistant, adversarial-tested, poisoned-data-proof). The design consequence: human-in-the-loop underwriting (Art. 22 "not solely", Art. 14 oversight), a reason engine wired to adverse-action notices, bias testing in validation, and drift monitoring in production. GenAI enters this use case as *assistants* (underwriting memos, eligibility explanations) — which keeps the credit decision on the classical model but makes the GenAI layer a high-rigour component too (hallucination control, because a fabricated eligibility statement is a conduct event).

### 7.2 AML and Fraud Detection

**The carve-out use case.** Annex III 5(b) explicitly excludes fraud detection from high-risk classification — a deliberate regulatory choice (fraud models protect consumers; over-restricting them would harm financial crime control). But the carve-out is narrow and the other requirements remain: **model risk in full** (SR 21-8/2021 and now the 2026 revised guidance explicitly covered BSA/AML models — the Fed's SR 26-2 supersedes SR 21-8 but the MRM expectations on AML models continue); **privacy in full** (transaction monitoring processes masses of personal data — DPIA, PDPA basis, retention limits); **fairness is live** (AML models can produce disparate impact; FEAT fairness applies, and the EU AI Act's carve-out does not remove the GDPR or conduct-law angles); **adversarial robustness is acute** (fraudsters actively probe detection models — evasion is the AML arms race); and **GenAI SAR drafting** carries hallucination risk (a fabricated SAR narrative is a regulatory filing failure — output validation and human review are mandatory). Notice 626/AML conduct rules from MAS and the US BSA/AML regime are the underlying conduct layer (`financial_risk_compliance_systems_guide.md` §7).

### 7.3 Chatbots and Client-Facing GenAI Assistants

**The transparency + leakage use case.** Requirements: **EU AI Act limited-risk transparency** — users must be told they are interacting with an AI (Art. 50); if the chatbot is built on a **GPAI** model, the deployer duties and supply-chain documentation apply; **GDPR/PDPA** in full (chat data is personal data; prompts containing client data are transfers; DPIAs for significant profiling); **FEAT transparency** — telling customers when decisions are materially influenced by AI; **data leakage control** is the defining risk (RAG leakage across clients, memorised PII, conversation history in logs — inside-information handling for a CIB is existential); **hallucination control** (a client-facing answer that fabricates a product rate or regulatory position is a mis-selling and conduct event — grounded responses, citations, human review of anything consequential, escalation to a human advisor); and **model risk** (the assistant is a model in the inventory with validation and monitoring — including for internal RM-facing copilots, which are the same use case with higher data sensitivity). The agent-era variant (chatbot that can *act* — draft and send, fetch, transfer) escalates straight into the excessive-agency controls of §4.2 and `../technology/beyond_zero_enterprise_security_guide.md`.

### 7.4 Trading and Markets

**The model-risk + conduct use case.** Requirements: **model risk is the centre of gravity** — pricing, VaR/ES, execution, and e-trading models are SR 11-7/SR 26-2 models with validation, backtesting, and monitoring (`risk_management_models_guide.md` §4); **market conduct and surveillance** — AI-assisted trading must not create or conceal market abuse (trade surveillance systems, cross-ref `financial_risk_compliance_systems_guide.md` §8; the EU AI Act's Annex III does *not* list trading, so no high-risk tier — but MAR/conduct rules bind); **agentic caution** — autonomous execution agents sit at the top of the excessive-agency risk scale; the industry pattern is human-in-the-loop for anything beyond pre-approved strategies; **not the credit/AI-Act high-risk path** — unless the trading AI touches creditworthiness (e.g., AI-generated credit limits for clients), in which case the Annex III analysis re-engages; **privacy is lighter but real** (no consumer personal data in most wholesale trading AI, but market data licensing, insider-information controls, and any personal data in execution analytics still trigger GDPR/PDPA); and **adversarial robustness** (market-data poisoning and model exploitation are documented concerns — a poisoned feature feed to a pricing model is a financial loss event).

### 7.5 The Mapping Table

| Use case | Primary requirements | Secondary / conditional | Notes |
|---|---|---|---|
| **Credit scoring (retail/SME)** | EU AI Act Annex III 5(b) high-risk; ECOA/Reg B adverse action; GDPR Art. 22 + DPIA; FEAT; full MRM lifecycle | PDPA basis; Art. 10 data governance; security/adversarial | Fraud-detection carve-out does NOT apply; human-in-the-loop is the design default |
| **AML / fraud detection** | MRM (BSA/AML models — SR 26-2); privacy (DPIA, PDPA); fairness (FEAT); adversarial robustness | EU AI Act carve-out (5(b) exception); MAS Notice 626; SAR-quality controls for GenAI | Carve-out narrow — adjacent uses re-engage Annex III |
| **Chatbots / RM copilots (GenAI)** | Art. 50 transparency; GPAI deployer duties; GDPR/PDPA; FEAT transparency; data-leakage controls; hallucination controls | DPIA if profiling; agent controls if acting; model-risk inventory entry | Client data in prompts = transfer; grounded answers + human escalation |
| **Trading / markets AI** | MRM (SR 26-2) for pricing/execution models; market conduct/surveillance | GDPR/PDPA if personal data; agentic-oversight if autonomous; Annex III only if credit-adjacent | No AI-Act high-risk tier for trading itself; conduct rules bind |

---

### 7.6 The Use-Case Approval Workflow

The mapping tables answer "which requirements" — the approval workflow answers "how a use case actually gets through." The standard banking intake (composed from the RAI guide's §5.3 risk-assessment procedure and the trust guide's §7 process):

1. **Intake and classification** — the use-case owner files the request; the AI council triages and classifies: EU-Act tier (Annex III check), FEAT relevance, internal materiality tier, jurisdiction and data scope. This one step determines the weight of everything after.
2. **Impact assessment** — DPIA (GDPR) and the ISO 42001-style AI system impact assessment, run together; lawful-basis documentation for the data (PDPA).
3. **Testing plan** — bias testing (fairness metrics, 80%-rule screen), robustness testing (adversarial, injection, hallucination-rate evaluation), and the validation plan (MRM scope, who validates).
4. **Committee sign-off** — the AI/model risk committee reviews the classification, assessments, and test results; approval may carry conditions (human oversight design, monitoring thresholds, no-agent constraints).
5. **Inventory entry and deployment** — the system is registered with its artifacts, deployed behind the security controls, and enters monitoring (drift, KRIs, incidents).
6. **Ongoing** — periodic re-assessment (at least annually and on material change), incident reporting, and regulator-facing evidence assembly.

---

## 8. The Worked Example: An AI-Compliance Gap Assessment

### 8.1 The Scenario (A Cymbal Bank GenAI Deployment)

The context is deliberately familiar (the same franchise profile as `risk_management_models_guide.md` §11's worked example and the trust guide's §8 assessment): **Meridian Bank** — a fictional Asian wholesale bank with a **Cymbal Bank-style Singapore franchise** (trade finance, commodity finance, structured lending to Asian corporates, treasury) and EU entities (Paris and Milan branches) through which it serves EU corporate and private-banking clients.

Two AI systems are in flight:

- **System A — "RM Copilot":** an internal GenAI assistant for relationship managers. RAG over internal product documentation, transaction policies, and deal files; built on a major vendor's foundation model (GPAI) via API; prompts include client names, deal terms, and sometimes mandates; answers draft memos and eligibility summaries; no direct action-taking (no transfers, no messaging) — but it can fetch deal files from the document store. Deployed from Singapore, available to RM teams in Singapore, Paris, and Milan.
- **System B — "SME Score":** a gradient-boosting credit-scoring pilot for SME lending (sole proprietors included — natural persons) at the Singapore branch, using alternative data (transaction behaviour, registry records); scores feed a decision memo; final approval requires an RM sign-off; an LLM component drafts the eligibility explanation and the adverse-action reasons.

The CRO asks for an **AI-compliance gap assessment** against the requirements matrix before either system scales. The method: for each requirement family, take the requirement → assess applicability → find the evidence → name the gap → assign the action.

### 8.2 The Assessment — the Requirements-Matrix Walkthrough

**Governance requirements**

| Requirement | Applies? | Evidence found | Gap | Action |
|---|---|---|---|---|
| AI inventory entry for both systems | Yes | MRM registry has classical models; A and B are not yet entries | Both systems invisible to governance | Add both to the reconciled AI inventory with classification and artifacts; define owner/validator/auditor |
| Committee approval | Yes | No AI committee yet; model risk committee exists | No AI risk-appetite statement; no sign-off record | Stand up AI governance tier under the MRM committee; approve A and B with conditions |
| Three lines of defence assigned | Yes | RM Copilot owned by Digital; no named validator | Second-line validation not assigned for GenAI | Name MRM validators for both; internal audit adds AI to the audit plan |

**Regulatory requirements**

| Requirement | Applies? | Evidence found | Gap | Action |
|---|---|---|---|---|
| EU AI Act Annex III (high-risk) | **System B — yes** (creditworthiness of natural persons, SME sole proprietors, from 2 Aug 2026); System A — limited-risk (transparency) | No classification performed | B is high-risk without the regime | Classify B as high-risk; build risk-management system, data governance (Art. 10), documentation, logging, human oversight, accuracy/robustness testing; register in EU database; assign Art. 26 deployer duties |
| EU AI Act GPAI supply chain | System A — yes (vendor foundation model) | Vendor's GPAI documentation not assessed | No evidence of vendor compliance or of inherited deployer duties | Procurement due-diligence on vendor GPAI obligations; document model lineage and training-data summaries |
| MAS FEAT | Both — yes | Bias testing for B planned, not run; no customer-facing AI disclosure for A's outputs | FEAT evidence file missing | Bias audit (fairness), board oversight (ethics), MRM sign-offs (accountability), disclosure + explanation process (transparency) |
| MAS AI MRM paper / AIRM guidelines | Both — yes (SG FIs) | No GenAI-specific validation approach | Supervisory expectations unmet | LLM validation approach: guardrail testing, hallucination evaluation, human oversight, prompt→output logging |
| SR 26-2 / MRM guidance (US) | Not directly (no US entities in scope) — note only | n/a | n/a | Monitor the planned AI RFI for group-level impact |
| ECOA/Reg B adverse action | System B — yes (SG branch lends to SG SMEs, but group has EU/US consumer exposure; Reg B binds US lending; treat as group standard) | B's LLM reason drafts not validated; no reason-code engine | Adverse-action reasons may be generic or wrong | Reason engine with SHAP-based specific reasons; human review of every notice |

**Risk requirements**

| Requirement | Applies? | Evidence found | Gap | Action |
|---|---|---|---|---|
| Model-risk lifecycle | Both — yes (B is the archetypal model; A is a model in the inventory) | B has a development report but no independent validation; A has none | Neither system through validation | Full adapted validation for B (conceptual soundness, monitoring, outcomes analysis); LLM-validation package for A (hallucination rate, guardrail tests, drift) |
| Hallucination control | System A — critical (memos to RMs); System B's LLM — high (reason drafts) | No factual-accuracy evaluation; RAG grounding not measured | Fabricated product/regulatory answers possible | Grounded RAG with citations; per-release hallucination-rate evaluation; human review of consequential outputs |
| Prompt injection / data leakage | System A — critical (fetches deal files; client data in prompts) | No permission-aware retrieval; prompts go to vendor API unfiltered | Client/inside information at risk; RAG cross-client leakage | Permission-aware retrieval, PII/secret scanning, log redaction, vendor-transfer assessment |
| Model drift | Both — yes | No monitoring dashboards | Silent decay in production | PSI/CSI + calibration monitoring for B; pinned model version + re-evaluation for A |

**Privacy requirements**

| Requirement | Applies? | Evidence found | Gap | Action |
|---|---|---|---|---|
| GDPR Art. 22 | System B — yes (credit decisions; RM sign-off may not be "meaningful" human involvement); System A — no (advisory) | Override process exists informally; no documented safeguards | Art. 22 safeguards (human intervention, express views, contest) not documented | Formalise override + appeals process; document why decisions are not "solely" automated (RM authority to change outcomes) |
| DPIA (Art. 35) | Both — yes (B: profiling-based credit decisions = mandatory; A: client data in RAG/prompts) | No DPIAs | High-risk processing unassessed | DPIA for B before scale; DPIA for A (incl. vendor transfer); Art. 36 check on residual risk |
| PDPA + PDPC AI Guidelines | Both — yes (SG data) | Consent/basis analysis not done for A's use of deal files; B's alternative data not assessed | New-purpose use of existing data | Document lawful basis per use; notification updates; breach-notification readiness |
| Training-data PII | A — vendor model (due diligence); B — alternative data | No vendor training-data documentation; B's data sources partially assessed | Memorisation and proxy-bias risk | Vendor training-data summaries; PII scanning of outputs; de-identified/synthetic data for any fine-tuning |

**Security requirements**

| Requirement | Applies? | Evidence found | Gap | Action |
|---|---|---|---|---|
| Secure-AI SDLC / supply chain | Both — yes | A deployed via vendor API without security assessment; B's ML pipeline unvetted | Vendor and dependency risk | Procurement security assessment for the LLM API; dependency scanning; DevSecOps gates for B |
| Adversarial robustness | B — high (credit fraudsters); A — medium | No red-teaming; no poisoning controls on B's alternative data | Evasion/poisoning untested | Adversarial testing in validation (evasion suite for B, injection suite for A); data-provenance checks |
| Agent-era controls | A — partially (can fetch deal files; no actions) | No tool-permission matrix; no kill switch | A could escalate into an acting agent without controls | Define tool permissions, confirmation gates on any future actions, kill switch, full action logging |

### 8.3 The Lessons

1. **Classification first.** The single highest-leverage step was classifying both systems: B is EU-AI-Act high-risk (2 August 2026 deadline), A is limited-risk + GPAI-deployer — everything else (validation depth, documentation, oversight) follows from the tier. Banks that skip classification discover it at the first exam.
2. **The inventory is the foundation.** Most gaps trace to systems being invisible to governance. A reconciled AI inventory with classification and artifacts turns the gap assessment from an audit into a checklist.
3. **One control, many requirement families.** Human-in-the-loop review serves Art. 22 safeguards, EU AI Act Art. 14 oversight, and the override economy of model risk simultaneously; PII scanning serves GDPR/PDPA and the data-leakage security control; adversarial testing serves Art. 15, SR 26-2 monitoring, and FEAT accountability. Design controls once, evidence them across families.
4. **GenAI changes the risk surface, not the discipline.** A is not "just software" and not "just a model" — it is a model (inventory, validation, monitoring) *and* a data-processing operation (DPIA, PDPA basis) *and* a supply-chain component (GPAI vendor) *and* a security perimeter extension. The requirements matrix is how those four views reconcile.
5. **Evidence is the deliverable.** Every requirement reduces to an artifact: inventory entries, DPIAs, validation reports, bias audits, reason codes, logs. The gap assessment's output is the evidence backlog — the same conclusion the trust guide's worked example reaches for trust assessments and the RAI guide's for RAI frameworks.

---

### 8.4 The Remediation Plan and the Scorecard

A gap assessment without a plan is an audit finding, not a fix. The 90-day remediation for the Meridian example:

| Window | Actions | Owners |
|---|---|---|
| Weeks 1–2 | Classify both systems (EU-Act tier, FEAT, jurisdiction); add to the AI inventory; appoint owners/validators; freeze any scale-up | CRO, AI council |
| Weeks 3–6 | DPIAs for A and B; PDPA basis documentation; vendor GPAI due diligence for A; MRM validation scoping for B | Privacy office, MRM, procurement |
| Weeks 6–10 | Bias audit for B; hallucination-rate evaluation and guardrail testing for A; adversarial/red-team testing for both; reason-engine design for B's adverse-action drafts | Validation, security, data science |
| Weeks 10–12 | Controls: permission-aware retrieval and PII scanning for A; human-oversight and appeals process for B; monitoring dashboards; committee review of the evidence file | Platform, first line, AI committee |
| Ongoing | Monthly KRI review; incident playbooks rehearsed; regulatory watch (AIRM finalisation, US RFI, AI Act delegated acts) | AI-SOC, compliance |

**The readiness scorecard** — the assessment result in one table (0 = absent, 4 = fully evidenced):

| Family | System A (RM Copilot) | System B (SME Score) | Critical path |
|---|---|---|---|
| Governance | 1 (no inventory entry, no owner) | 2 (model exists, no AI classification) | Inventory + committee |
| Regulatory | 1 (no tier classification, no vendor due diligence) | 1 (high-risk obligations unbuilt) | Annex III programme for B |
| Risk | 1 (no LLM validation) | 2 (development report, no independent validation) | Validation packages |
| Privacy | 1 (no DPIA, prompts carry client data) | 1 (no DPIA, alternative data unassessed) | DPIAs + basis documentation |
| Security | 1 (no supply-chain assessment, no RAG permissions) | 2 (pipeline unvetted) | Security assessment + red-teaming |

The honest headline: neither system is production-ready; B is the bigger liability because the high-risk regime has a fixed date (2 August 2026), and the franchise-level lesson is that the inventory and classification machinery should have existed before either pilot started.

## 9. The Ongoing Obligations: Monitoring, Incidents, and Keeping Current

Requirements are not a one-time assessment — they are continuous obligations, and the regulators say so explicitly. The three ongoing duties every banking AI system carries:

**Monitoring.** SR 26-2's ongoing monitoring (and SR 11-7's before it) requires performance and drift tracking for every model; the EU AI Act requires post-market monitoring for high-risk systems (Art. 72) and logging (Art. 12); FEAT's transparency and fairness items assume ongoing review of data and models; MAS's 2024 AI MRM paper's good practices are built on monitoring GenAI in production (hallucination rates, guardrail effectiveness, human-override volumes). The monitoring dashboard per inventory entry — model metrics, KRIs, incident counts, data-quality flags — is the standing evidence file.

**Incident response.** What counts as an AI incident in banking: a serious malfunction or harmful output (EU AI Act serious-incident reporting obligations), a data breach involving AI (GDPR 72-hour notification; PDPA mandatory breach notification), a security event (prompt injection with exfiltration, poisoning, agent misuse — Notice 637 reporting), or a model failure with material impact (MRM incident escalation). The playbooks from §6.4 apply; the principle is that the same incident is reported to multiple regulators under different rules, so the incident log is structured from day one for regulator-facing reconstruction.

**Keeping current.** The regulatory map has moved four times in the period this guide covers (SR 11-7 → SR 26-2; FEAT → MAS AI MRM paper → AIRM consultation; AI Act phased application; PDPC AI guidelines), and it will keep moving: the US agencies' planned AI RFI, the AIRM final guidelines, the AI Act's delegated acts and codes of practice, and the EU–Singapore adequacy question are the live items to watch. The institutional answer is a **regulatory-watch function** (compliance owns a living map of AI requirements by jurisdiction and use case — this guide's §2.4 and §7.5 tables are the seed), an **annual re-assessment cycle** per inventory entry, and a versioned policy library so that when a requirement changes, the affected systems and evidence are identifiable in minutes. The banks that treat AI compliance as a programme — not a project — are the ones that will be ready when the next consultation becomes final.

---

### 9.1 The Annual AI Compliance Cycle

The "keeping current" obligation (§9) becomes concrete as an annual cycle, timed to the regulatory calendar:

| Month | Activity | Output |
|---|---|---|
| Q1 | Annual classification refresh: re-run the EU-Act tier / FEAT / materiality classification on every inventory entry; new systems triaged | Updated inventory with classifications |
| Q1–Q2 | Validation and monitoring reviews due; bias/robustness re-tests on changed systems; DPIAs re-opened where data or purpose changed | Validation and DPIA updates |
| Q2 | Regulatory-watch digest: AIRM finalisation, US AI RFI, AI Act delegated acts and codes of practice, PDPC updates, DORA operational events | Requirement-map refresh (§2.4, §7.5 tables) |
| Q3 | Independent assurance: internal audit samples the AI estate; external validators re-run key assessments | Audit and validation findings |
| Q4 | Board reporting: risk-appetite attestation, incident summary, material findings, next-year plan; evidence files archived per inventory entry | Board pack, evidence archive |

The cycle is the same shape as the classical model-risk and ORSA-style cycles the bank already runs — AI compliance slots into the existing calendar rather than creating a new one, which is exactly how it survives contact with the operating rhythm of a bank.

---

## 10. The One-Page Summary — The Final Word: the Requirements That Bind

**The requirements that bind** an AI or GenAI system in banking are a stack, not a single law, and the stack is now concrete:

- **The EU AI Act binds the system's *purpose*:** credit scoring of natural persons is **Annex III 5(b) high-risk** (verified) with the full regime from **2 August 2026**; chatbots carry **Art. 50 transparency**; the foundation models underneath carry **GPAI duties** (from 2 August 2025); the fraud-detection carve-out is narrow and does not bless adjacent uses. Fines to €35M/7%.
- **MAS binds the *demonstration*:** FEAT (12 November 2018) is the report card — fairness, ethics, accountability, transparency, each mapped to evidence; the **December 2024 AI model-risk information paper** set the expected GenAI practice; the **November 2025 AIRM consultation** (status flagged) is the direction of travel; Notice 637/TRM wraps the platform.
- **The US regulators bind the *models and the credit decisions*:** SR 11-7's discipline survives as vocabulary and as substance — the April 2026 revised guidance (**SR 26-2 / OCC Bulletin 2026-13**, verified) supersedes SR 11-7 and SR 21-8, keeps the risk-based MRM umbrella, and defers AI/GenAI/agentic specifics to a planned RFI (flagged); ECOA/Reg B binds credit decisions to specific, non-generic adverse-action reasons — for AI models no less (CFPB Circulars 2022-3, 2023-3).
- **The GDPR binds the *decisions and the processing*:** Article 22 — no solely-automated significant decisions without human intervention, express-views and contest rights; Article 35 — a mandatory DPIA for exactly the profiling-based AI decisions banks make. Singapore's PDPA and the PDPC's 2024 AI advisory guidelines bind the data and the basis for using it.
- **Model risk binds *everything that computes a number a decision uses*:** inventory, independent validation, monitoring, outcomes analysis, controlled retraining — for ML and for LLMs, in Singapore as much as in the US.
- **The GenAI taxonomy binds the *implementation*:** hallucination, prompt injection, data leakage, drift, and excessive agency each map to controls and to a requirement family; security for AI is secure-SDLC plus adversarial testing plus agent-era control, and regulators count it as compliance evidence.
- **The use case decides the weight:** credit scoring carries the heaviest stack; AML rides the carve-out but keeps model risk, privacy, and adversarial robustness; chatbots carry transparency, leakage, and hallucination duties; trading carries model risk and conduct, not the AI-Act high-risk tier.
- **The governance machinery makes it real:** committees, three lines of defence, the reconciled AI inventory, and model-risk integration are what turn the stack into evidence.

The bank that classifies every AI system, enters it in one inventory, runs it through the model-risk lifecycle, maps each requirement to an artifact, and keeps the human in the loop is — whatever the regulator, whatever the year — substantially compliant. The requirements bind; the evidence frees.

---

## 11. The Glossary

- **Adverse action** — a US fair-lending term (ECOA/Reg B): the denial, termination, or unfavourable change of credit, which triggers a notice to the applicant stating the *specific* principal reasons for the decision; applies to AI credit decisions (CFPB Circulars 2022-3, 2023-3).
- **AI governance** — the decision machinery for AI: committees, ownership, policies, risk appetite, the three lines of defence, and the AI inventory; distinct from RAI (values-to-evidence), which it makes enforceable.
- **AI inventory** — the single reconciled register of every AI system: identity, classification, artifacts (model cards, DPIAs, test reports), lifecycle state; reconciles the MRM registry, compliance use-case list, and platform registry.
- **AML** — anti-money laundering: transaction monitoring, screening, and reporting (SARs) against financial crime; AI is heavily used in detection; fraud-detection AI is carved out of EU AI Act Annex III 5(b) high-risk classification.
- **Annex III** — the EU AI Act's list of high-risk application areas; point 5(b) covers creditworthiness evaluation/credit scoring of natural persons (excluding financial-fraud detection); high-risk obligations apply from 2 August 2026.
- **Article 22** — GDPR provision granting the right not to be subject to decisions based *solely* on automated processing (incl. profiling) with legal or similarly significant effects; exceptions (contract, law, consent) and mandatory safeguards (human intervention, express views, contest).
- **Chatbot** — a conversational AI interface (here: banking assistants, RM copilots); bound by EU AI Act Art. 50 transparency, GPAI deployer duties, privacy law, and data-leakage/hallucination controls.
- **Credit scoring** — evaluating creditworthiness or establishing a credit score (Annex III 5(b)); the most heavily bound banking AI use case (high-risk regime, ECOA/Reg B, Art. 22, FEAT, model risk).
- **Data leakage** — GenAI risk: training data, retrieved documents, or conversation context crossing boundaries (RAG cross-tenant leakage, memorised PII, prompts in logs); a privacy breach and confidentiality breach in banking.
- **DPIA** — Data Protection Impact Assessment (GDPR Art. 35): mandatory before high-risk processing, including systematic profiling-based decisions (Art. 35(3)(a)); banking's AI sibling is the ISO 42001 AI system impact assessment.
- **ECOA** — the US Equal Credit Opportunity Act: prohibits credit discrimination; implemented by Regulation B; the fair-lending law that AI credit models must satisfy, with adverse-action notice duties.
- **EU AI Act** — Regulation (EU) 2024/1689: the EU's risk-tiered AI law (unacceptable/high/limited/minimal), in force 1 August 2024, phased; fines to €35M/7% of global turnover.
- **FEAT** — MAS's Principles to Promote Fairness, Ethics, Accountability and Transparency in the Use of AI and Data Analytics (12 November 2018): voluntary-but-authoritative expectations for Singapore FIs; operationalised by the Veritas methodologies.
- **Gap assessment** — a structured review of a system or programme against a requirements list (here: the AI requirements matrix) to identify missing obligations, evidence, and controls, with actions to close them.
- **GDPR** — the EU General Data Protection Regulation (2016/679): the data-protection law whose Art. 22 (automated decisions) and Art. 35 (DPIA) bind AI in banking.
- **GenAI** — generative AI: models (LLMs and others) that generate new content; in banking, assistants, drafting, RAG copilots, and agentic systems — with the GenAI risk taxonomy (hallucination, injection, leakage, drift).
- **GPAI** — general-purpose AI: foundation models covered by EU AI Act Articles 51–56 (documentation, training-data summaries, copyright policy; systemic-risk duties above the compute threshold); deployers inherit supply-chain duties.
- **Hallucination** — GenAI risk: fluent fabricated output (false regulations, invented figures, wrong product terms); in banking, a conduct and mis-selling hazard; controlled by grounding, citations, evaluation, and human review (OWASP LLM09).
- **High-risk** — the EU AI Act tier for systems in Annex III areas (incl. credit scoring) or Annex I products: the full obligations regime (risk management, data governance, documentation, logging, transparency, human oversight, accuracy/robustness/cybersecurity, conformity assessment).
- **ISO 42001** — ISO/IEC 42001:2023, the first certifiable AI management system (AIMS) standard: AI policy, AI risk assessment, the AI system impact assessment, lifecycle controls; the management-system wrapper for banking AI compliance.
- **MAS** — the Monetary Authority of Singapore: prudential and conduct regulator for FIs; issuer of FEAT, the 2024 AI model-risk information paper, the proposed AIRM guidelines, Notice 637/TRM.
- **Model drift** — the decay of model performance as populations, economies, features, or upstream models move; monitored via PSI/CSI, calibration tracking, and pinned-version re-evaluation; a model-risk and robustness requirement.
- **Model risk** — the risk of adverse consequences from decisions based on models that are incorrect or misused; governed by the inventory–validation–monitoring lifecycle (SR 11-7 legacy; SR 26-2 current; MAS 2024 AI MRM paper).
- **NIST AI RMF** — the US National Institute of Standards and Technology's AI Risk Management Framework 1.0 (January 2023): the voluntary Govern–Map–Measure–Manage process that is the de facto internal AI risk standard for banks.
- **PDPA** — Singapore's Personal Data Protection Act 2012 (amended 2020): consent/basis rules, breach notification (mandatory from Feb 2022), and the PDPC's AI Advisory Guidelines (1 March 2024) for AI recommendation/decision systems.
- **PII** — personally identifiable information: data identifying or relating to an individual; the object of GDPR/PDPA obligations and a core GenAI leakage/memorisation risk in training data, RAG, and prompts.
- **Prompt injection** — GenAI attack: malicious instructions (direct or indirect via retrieved content) hijacking the model into unintended behaviour; the highest-severity class for RAG and agentic banking systems (OWASP LLM01).
- **Reg B** — Regulation B (12 CFR 1002), the CFPB's implementation of ECOA: adverse-action notice rules, including for AI/complex credit models.
- **Requirements matrix** — the mapping of requirements to sources, banking applications, and evidence — the organising device of this guide and the output of a gap assessment.
- **SR 11-7** — the 2011 Fed/OCC Supervisory Guidance on Model Risk Management: the canonical model definition and validation discipline; superseded on 17 April 2026 by SR 26-2 / OCC Bulletin 2026-13, but its vocabulary remains the MRM lingua franca.
- **Three lines of defence** — the control model: first line owns (business/dev), second line challenges (risk, compliance, MRM, privacy), third line assures (internal audit); re-drawn explicitly for each AI system.
- **Trading** — markets AI (pricing, execution, surveillance): bound by model risk and market conduct rather than the AI Act high-risk tier; agentic trading demands the strongest oversight controls.

---

## 12. The Verification Ledger

**Verified against primary sources (August 2026):**

- EU AI Act: adoption June 2024, in force 1 Aug 2024, phased application (prohibitions 2 Feb 2025; GPAI 2 Aug 2025; Annex III high-risk 2 Aug 2026; Annex I 2 Aug 2027); Annex III point 5(b) creditworthiness/credit score of natural persons = high-risk with the financial-fraud-detection exception (EC AI Act service desk; cross-ref `ai_trust_assessments_guide.md` §2.2); GPAI Articles 51–56, systemic-risk presumption ~10²⁵ FLOPs; penalties to €35M/7%.
- MAS: FEAT published 12 Nov 2018 (cross-ref trust guide §3.4); Information Paper on AI Model Risk Management published 5 Dec 2024 following a mid-2024 thematic review of banks' AI/GenAI model risk (Allen & Gledhill; regulations.ai summaries).
- US: SR 11-7 / OCC Bulletin 2011-12 issued 4 Apr 2011; SR 21-8 issued 9 Apr 2021; **Revised Guidance on Model Risk Management issued 17 Apr 2026 as SR Letter 26-2 and OCC Bulletin 2026-13, superseding/rescinding SR 11-7 and SR 21-8** (Federal Reserve and OCC official pages); the agencies plan a future RFI on AI, including generative and agentic AI (OCC bulletin text).
- GDPR: Article 22 text and exceptions (gdpr-info.eu); Article 35(3)(a) mandatory DPIA for systematic profiling-based decisions.
- ECOA/Reg B: adverse-action requirements under Reg B §1002.9; CFPB Circular 2022-3 (May 2022) and Circular 2023-3 (Sept 2023) on AI/complex credit models (CFPB monitor summaries); 1071 final rule published 31 Mar 2023.
- PDPC: Advisory Guidelines on Use of Personal Data in AI Recommendation and Decision Systems, 1 Mar 2024 (PDPC).
- Framework dates: NIST AI RMF 1.0 Jan 2023; ISO/IEC 42001 Dec 2023 (cross-ref trust guide).

**Flagged [verify] — current status to confirm before citing as settled:**

- MAS Guidelines on AI Risk Management: consultation paper of 13 Nov 2025; final guidelines' issuance date and final text.
- OCC/Fed/FDIC RFI on AI (generative/agentic) model risk: announced as planned; not yet issued as of this writing.
- CFPB Section 1071 small-business data-collection provisions: litigation status and the effect of the May 2026 reconsideration/revision rulemaking.
- Executive Order 14110 revocation (Jan 2025) and any subsequent US federal AI measures.
- Sibling-guide facts cross-referenced here carry their own verification notes (see `ai_trust_assessments_guide.md` Appendix and `risk_management_models_guide.md` §9 for the SR 11-7-era ledger).

---

## 13. The Cross-References

**Technology cluster (frameworks, RAI, security):**

- `../technology/ai_trust_assessments_guide.md` — NIST AI RMF, EU AI Act, ISO 42001, OECD, SG frameworks; the verified framework facts this guide builds on.
- `../technology/responsible_ai_frameworks_guide.md` — RAI canon, corporate frameworks, tooling, MAS FEAT §6.1, the RAI framework design worked example.
- `../technology/ai_verify_guide.md` — Singapore's AI Verify testing and assurance.
- `../technology/ai_llm/ai_governance_bias_redteaming_guide.md` — governance pillars, bias measurement/mitigation, red-teaming, fair-lending analytics.
- `../technology/beyond_zero_enterprise_security_guide.md` — agent-era security and agent governance.
- `../technology/llm_development_risks_security_guide.md` — the GenAI risk taxonomy (OWASP LLM Top 10), attack mechanics, the six-layer defense-in-depth architecture.
- `../technology/security_by_design_guide.md` — secure SDLC, threat modelling, DevSecOps.
- `../technology/cybersecurity_guide.md` — the security program and the banking security context (§8).
- `../technology/adversarial_ml_attacks_guide.md` — evasion, poisoning, extraction; adversarial robustness.
- `../technology/data_governance_guide.md` — data lineage, privacy-by-design data handling.

**Banking cluster (risk, compliance, MAS context):**

- `financial_risk_compliance_systems_guide.md` — the compliance systems landscape (AML §7, surveillance §8, regulatory reporting §9, analytics §12).
- `risk_management_models_guide.md` — model risk and the SR 11-7 discipline (§9), ML in risk (§10), the PD-model worked example (§11).
- `regtech_guide.md` — regulatory technology and the compliance-software landscape.
- `banks_in_singapore_guide.md` — MAS context, Notice 637, TRM, licensing, outsourcing rules.
- `basel_regulatory_capital_guide.md`, `risk_data_aggregation_guide.md`, `treasury_alm_guide.md`, `cecl_guide.md` — the regulatory trio and IFRS 9 provisioning the AI credit models feed.

**Management cluster:**

- `../management/business_case_development_guide.md` — the business case for the AI governance and control spend.
