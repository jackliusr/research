# AI Verify: The Definitive Guide to Singapore's AI Testing Framework

> **The dedicated deep-dive on A.I. Verify — Singapore's world-first AI governance testing framework and open-source toolkit (IMDA/PDPC, May 2022) — the 11 governance principles, the technical tests (fairness, explainability, robustness, transparency), the AI Verify Foundation (June 2023), Project Moonshot (May 2024), the G7 Hiroshima alignment, and a Cymbal Bank worked assessment — with a full glossary.**

**Author:** Jack Liu Shurui  
**Role:** Solution Architect, Cymbal Bank  
**Audience:** Solution Architects, AI/ML Platform Teams, Model Risk Management (MRM), Compliance and Audit, CTO Office  
**Version:** 1.0  
**Last Updated:** August 2026  
**Series:** AI & Data Platform Guides — this is the **dedicated AI Verify deep-dive**. The AI-assurance umbrella (where AI Verify appears as one framework among many) lives in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) (its §3.2 is the AI Verify section — read this guide *with* that one); the governance/bias/red-teaming treatment lives in [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md); the enterprise governance layer in [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md); the audit practice in [audit_as_code_guide.md](audit_as_code_guide.md); the banking angle in [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md).

---

## How to Read This Guide

- **Architects and platform teams** — start with [§1](#1-the-overview) (what AI Verify actually is), [§3](#3-the-components) (framework vs. toolkit vs. foundation), and [§7](#7-the-project-moonshot) (the GenAI companion); pair with [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the platform the tests will run against.
- **Model risk / validation teams** — [§4](#4-the-principles-mapping) and [§5](#5-the-technical-tests) are the substance (which principle maps to which test); [§9](#9-the-worked-example-an-ai-verify-assessment) shows the toolkit-run design; cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) for the SR 11-7 wrapper.
- **Compliance / governance officers** — [§2](#2-the-history), [§6](#6-the-ai-verify-foundation), [§8](#8-the-international), and the glossary are the core reading; the summary in [§10](#10-the-summary) is the one-pager.
- **Anyone defending an AI budget in Singapore** — [§10](#10-the-summary) plus the worked example; the cost/assurance framing lives in [../management/business_case_development_guide.md](../management/business_case_development_guide.md).

**Scope note on verification:** dates, principle names, and launch facts below were checked against primary sources (IMDA and PDPC press releases, the AI Verify Foundation site and publications, the official GitHub repositories and their API metadata, IMDA's Project Moonshot release) in August 2026. Where a fact could not be fully verified it is explicitly flagged **[verify]** rather than asserted — the same discipline this guide recommends for assessments. The verification log is in the Appendix.

---

## Table of Contents

[How to Read This Guide](#how-to-read-this-guide)
1. [The Overview](#1-the-overview)
   - 1.1 What AI Verify Is (the Definition)
   - 1.2 What "Verify" Means
   - 1.3 The Three-Layer Anatomy
   - 1.4 The Overview Table
   - 1.5 The Positioning: AI Verify vs. the Global Frameworks
2. [The History](#2-the-history)
   - 2.1 The Announcement (January 2022)
   - 2.2 The Launch (May 2022)
   - 2.3 The Evolution (2023–2026)
   - 2.4 The History Table
   - 2.5 The Adoption Coverage
3. [The Components](#3-the-components)
   - 3.1 The Testing Framework
   - 3.2 The Open-Source Toolkit
   - 3.3 The Components Table
   - 3.4 The End-to-End Workflow
4. [The Principles Mapping](#4-the-principles-mapping)
   - 4.1 The 11 Governance Principles (the Exact List)
   - 4.2 The Mapping Table
   - 4.3 Using the Mapping in Practice
5. [The Technical Tests](#5-the-technical-tests)
   - 5.1 The Test Families
   - 5.2 The Tests Table
   - 5.3 Operational Notes on Running the Tests
6. [The AI Verify Foundation](#6-the-ai-verify-foundation)
   - 6.1 The Consortium (June 2023)
   - 6.2 The Members
   - 6.3 The Foundation Table
7. [The Project Moonshot](#7-the-project-moonshot)
   - 7.1 The LLM Evaluation Toolkit (2024)
   - 7.2 The Moonshot Table
   - 7.3 Moonshot in the Evaluation-Tool Landscape
8. [The International](#8-the-international)
   - 8.1 The G7 Hiroshima Alignment
   - 8.2 The International Table
9. [The Worked Example: An AI Verify Assessment](#9-the-worked-example-an-ai-verify-assessment)
   - 9.1 The Scenario: A Cymbal Bank Model
   - 9.2 The Toolkit-Run Design
   - 9.3 The Findings
   - 9.4 The Lessons
10. [The Summary](#10-the-summary)
    - 10.1 The One-Page Summary
    - 10.2 The Final Word: The SG Testing Standard

[Glossary](#glossary)
[Appendix: Verification Log](#appendix-verification-log-august-2026)
[Primary Sources and Further Reading](#primary-sources-and-further-reading)

---

## 1. The Overview

### 1.1 What AI Verify Is (the Definition)

**AI Verify** (stylised **A.I. Verify**) is **the world's first AI governance testing framework and software toolkit** for companies that want to demonstrate responsible AI "in an objective and verifiable manner". It was announced by Singapore in January 2022 and launched in May 2022 by the **Infocomm Media Development Authority (IMDA)** and the **Personal Data Protection Commission (PDPC)** as an open-source Minimum Viable Product, and it has been stewarded by the **AI Verify Foundation** since June 2023.

The official definition has two halves that must not be conflated:

- **The AI Verify testing framework** — a structured, documented assessment method that helps companies assess an AI system's responsible implementation against **11 internationally recognised AI governance principles**. It is organised as *principles → outcomes → processes → evidence*: each principle has desired outcomes, each outcome is achieved through specified processes, and each process is validated by documentary evidence. It is delivered as a PDF/Excel-based self-assessment instrument (the "process checks").
- **The AI Verify toolkit** — the companion **open-source software** (GitHub: `aiverify-foundation/aiverify`, Apache-2.0) that automates the *technical* side of the assessment: a dockerized test harness that runs automated **technical tests** against a model (fairness, explainability, robustness, model performance) and generates a **testing report** that a company can publish.

The two work together: the framework is the *what and why* (which principles, which evidence), the toolkit is the *how* (automated tests that produce numbers). A company runs the toolkit's technical tests, completes the framework's process checks, and combines both into a single **AI Verify report** — the governance equivalent of a security scan report or an audit opinion.

**Key facts (verified):**

- **World's first** — IMDA/PDPC explicitly billed AI Verify as "the world's first AI Governance Testing Framework and Toolkit" at launch.
- **Voluntary** — AI Verify is not a law. It is a voluntary, self-testing instrument: the company runs the tests, self-assesses against the principles, and decides what to publish. It is the *evidence* that Singapore's regulatory posture expects, not a licence or a certification (though third-party accreditation now exists — see §6).
- **Model scope** — the original (2022) framework and toolkit target **traditional supervised ML models** (classification and regression). The framework was updated on **29 May 2025** to also cover **generative AI** applications, and the Foundation's **Project Moonshot** (May 2024) provides the technical testing for LLMs.
- **Governance basis** — the framework tests against 11 principles that trace to Singapore's **Model AI Governance Framework** (2019, 2nd ed. 2020) and that "jurisdictions around the world coalesce around" (PDPC's wording), consistent with the EU, OECD and G7 frameworks.
- **International by design** — launched "for international pilot and feedback", aligned to the EU, OECD, US NIST AI RMF and the G7 Hiroshima Process, with published crosswalks (§8).

> **Where this sits in the series:** [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) is the umbrella — it covers AI trust dimensions (reliability, safety, fairness, transparency, accountability), the global frameworks (NIST AI RMF, EU AI Act, ISO 42001, OECD), and the full assessment process, and it treats AI Verify as *one* element of the Singapore angle (§3.2 there). **This guide is the dedicated deep-dive**: it assumes the umbrella's vocabulary and goes deep on AI Verify itself — its history, components, principle mapping, technical tests, the Foundation, Moonshot, the international alignment, and a worked assessment. Read the umbrella first if you need the trust-dimension and framework context; read this guide when you need to *run* an AI Verify-style assessment or explain AI Verify to a stakeholder.

### 1.2 What "Verify" Means

The name is a deliberate statement of method. Singapore's earlier governance instruments (the Model AI Governance Framework, MAS FEAT) were *principles* — statements of what good AI governance looks like. AI Verify is the attempt to make such principles **operationally testable**:

- **Objective** — where a principle is quantitative (fairness, robustness, performance), the toolkit computes metrics with a defined method, so two assessors should reach the same numbers on the same model and data.
- **Verifiable** — every claim in the report is backed by an artifact: a test run, a metric, a dated document. The framework's *evidence* layer exists precisely so a reviewer (customer, regulator, auditor) can check the claim rather than take it on trust.
- **Self-declared** — verification is company-run (self-testing), which preserves commercial confidentiality of models while giving a "common basis to declare results" (a point the PDPC and legal commentary both emphasise). This is the same *self-assessment plus published evidence* pattern the EU AI Act uses for limited-risk systems, and the *documentation* pattern of ISO 42001.

In short: **AI Verify turns "we think our AI is responsible" into "here is the evidence, run it yourself"** — and the evidence file is exactly the currency that bank MRM (SR 11-7), MAS FEAT-aligned governance, and Singapore's Model Framework expect (cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) and [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.4 for FEAT).

### 1.3 The Three-Layer Anatomy

Three things are often all called "AI Verify"; keeping them separate avoids most confusion:

1. **The framework** (governance instrument) — the 11-principle assessment structure with process checks and evidence requirements; free, paper/Excel-based; updated May 2025 to cover traditional + GenAI.
2. **The toolkit** (software) — the open-source technical test harness for traditional ML models; GitHub `aiverify-foundation/aiverify`.
3. **The Foundation** (institution) — the not-for-profit consortium (June 2023, wholly owned subsidiary of IMDA) that stewards the framework and toolkit, runs **Project Moonshot** (the LLM evaluation toolkit), publishes international crosswalks, and runs the tester-accreditation and assurance-sandbox programmes.

The GenAI extension adds a fourth: **Project Moonshot** is the LLM-side counterpart of the toolkit (benchmarking + red teaming), sitting under the same Foundation and feeding the same 11-principle framework's GenAI version.

### 1.4 The Overview Table

| Aspect | Description |
|---|---|
| **What it is** | The world's first AI governance **testing framework** and open-source **toolkit** for demonstrating responsible AI objectively and verifiably (IMDA + PDPC, launched **25 May 2022**) |
| **What it does** | Self-assessment of AI systems via **process checks** (documentary evidence against governance practices) plus **technical tests** (automated metrics on the model) — combined into a publishable **testing report** |
| **Governance basis** | **11 internationally recognised AI governance principles** drawn from the Model AI Governance Framework (transparency, explainability, repeatability, safety, security, robustness, fairness, data governance, accountability, human agency and oversight, inclusive growth) — see §4 |
| **Framework structure** | Principles → Outcomes → Processes → Evidence (each principle has outcomes; each outcome has processes; each process is validated by documentary evidence) |
| **Model scope** | Traditional supervised ML models (2022 version); extended to **generative AI** with the 29 May 2025 framework update; LLM technical testing via Project Moonshot (2024) |
| **Legal status** | **Voluntary** self-testing — not a law, not a licence; a "common basis to declare results" that preserves model confidentiality |
| **Stewardship** | **AI Verify Foundation** since **7 June 2023** (global open-source consortium; wholly owned subsidiary of IMDA) |
| **GenAI companion** | **Project Moonshot** (launched **31 May 2024**) — open-source LLM evaluation toolkit: benchmarking + red teaming |
| **International posture** | Launched "for international pilot and feedback"; aligned with **EU, OECD, US NIST AI RMF, G7 Hiroshima Process**; published crosswalks (NIST Oct 2023; Hiroshima CoC + NIST GenAI profile May 2025; ISO 42001 Jul 2025) |
| **Why it matters for a bank** | Produces the objective, publishable AI-governance evidence that MAS/FEAT-aligned governance and MRM expect; the local "audit report" pattern for AI (cross-ref [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) and [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3) |

> **One-line summary for stakeholders:** AI Verify is Singapore's voluntary national testing standard for AI — the framework says *what good AI governance looks like across 11 principles*, the open-source toolkit *tests it automatically*, the report *publishes the evidence*, and the Foundation *runs the whole thing with the global community*.

### 1.5 The Positioning: AI Verify vs. the Global Frameworks

AI Verify is frequently confused with the global governance frameworks it is *aligned with*. The clean positioning (details and crosswalks in §8):

| Framework | What it is | Where AI Verify sits |
|---|---|---|
| **EU AI Act** (Reg. 2024/1689) | Binding, risk-tiered regulation with penalties | AI Verify is voluntary evidence, not a conformity regime; useful as supplementary documentation but no legal substitute |
| **NIST AI RMF 1.0** (Jan 2023) | Voluntary risk-management *process* (Govern–Map–Measure–Manage) | AI Verify is a **testing instrument** you can slot into the RMF's Measure function; the official crosswalk maps it |
| **ISO/IEC 42001** (Dec 2023) | Certifiable AI *management system* (AIMS) | AI Verify evidence feeds AIMS records (test results, documentation); the 2025 crosswalk maps clauses |
| **OECD AI Principles** (2019, updated 2024) | Intergovernmental *principles* floor | AI Verify operationalises the principles into testable outcomes/evidence |
| **G7 Hiroshima AI Process** (2023) | Guiding principles + Code of Conduct for advanced-AI developers | The 2025 crosswalk maps AI Verify to the Code of Conduct items |

**The one-line positioning:** AI Verify is not an alternative to these frameworks — it is the **testing and evidence layer underneath them**. Where they say *what* to manage or *how* to govern, AI Verify says *how to demonstrate it in numbers and documents* — and the crosswalks are the translation table. This is the same layering argument [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.6 makes for the global frameworks themselves; AI Verify slots in as the measurement machinery.

---

## 2. The History

### 2.1 The Announcement (January 2022)

**Verified:** AI Verify was **announced in January 2022 at the World Economic Forum (Davos)** by Singapore's Minister for Communications and Information, **Mrs Josephine Teo**. The announcement positioned it as the next step in Singapore's AI governance journey, which had begun with the **Model AI Governance Framework** (the world's first such framework, launched at Davos in January 2019, 2nd edition January 2020 — cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.1). The logic was: principles were in place; what was missing was a way for companies to *demonstrate* adherence **objectively and verifiably**, rather than by assertion.

The framework had been developed in consultation with companies of different sectors and scale, including (per the AI Verify Foundation's own account): **AWS, DBS Bank, Google, Meta, Microsoft, Singapore Airlines, NCS (Part of Singtel Group) / Land Transport Authority, Standard Chartered Bank, UCARE.AI, and X0PA.AI**. The banking representation (DBS, Standard Chartered) is notable: Singapore's banks were in the room from the start, which is why AI Verify's evidence pattern maps so directly onto bank AI governance today.

### 2.2 The Launch (May 2022)

**Verified:** On **25 May 2022**, **IMDA and the PDPC launched A.I. Verify** — the **world's first AI Governance Testing Framework and Toolkit** — as a **Minimum Viable Product (MVP) for international pilot and feedback**. The launch was a joint press release by the Ministry of Communications and Information, IMDA and the PDPC.

What shipped in the MVP:

- The **testing framework** — the 11-principle structure with process checks and evidence requirements (the ancestor of today's PDF/Excel framework).
- The **software toolkit** — a dockerized test harness for traditional supervised ML models that runs technical tests (explainability, fairness, robustness) and produces a report companies could publish.
- The **pilot mechanism** — companies were invited to run the MVP and feed back, in Singapore's signature "regulatory sandbox-adjacent" style: learn from real use, iterate the instrument, then formalise.

Why "world's first" matters: at the time, no other jurisdiction had published a *testable* AI governance instrument combining a principles framework with runnable technical tests. The EU AI Act was still a proposal; NIST's AI RMF 1.0 would not appear until January 2023; ISO/IEC 42001 until December 2023. Singapore's claim to be first with a *testing* framework is accurate and was the foundation of its later claim to be a reference point for international AI governance.

### 2.3 The Evolution (2023–2026)

The timeline after launch (all dates verified against primary sources unless flagged):

- **7 June 2023 — open-sourced.** The AI Verify framework and toolkit were **open-sourced on GitHub** (`aiverify-foundation/aiverify`; the repository's API metadata shows creation on 3 June 2023, made public on 7 June). Open-sourcing was the move that turned a national instrument into a global one: anyone could inspect, run, extend, and contribute tests.
- **7 June 2023 — the AI Verify Foundation.** At the **ATxAI conference (part of Asia Tech x Singapore, ATxSG)**, Minister Josephine Teo announced the **AI Verify Foundation** — a global consortium to develop AI testing tools with the open-source community (§6).
- **16 January 2024 — the GenAI governance proposal.** IMDA published the **proposed Model AI Governance Framework for Generative AI** for international views (9 dimensions), signalling that the testing framework would need a GenAI extension.
- **31 May 2024 — Project Moonshot.** At **ATxSG 2024**, Minister Teo launched **AI Verify – Project Moonshot** in open beta: one of the world's first **LLM evaluation toolkits**, bringing benchmarking and red teaming together (§7).
- **May 2024 — the GenAI framework finalised.** The final **Model AI Governance Framework for Generative AI** was released, complemented by an "AI Verify for Generative AI" testing catalogue and the GenAI evaluation sandbox (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.3).
- **29 May 2025 — the updated testing framework.** The AI Verify testing framework was **updated and re-released**: enhanced to address GenAI risks, so companies can now apply it to **both traditional and GenAI use cases**. The same date saw crosswalks published to the **NIST AI RMF Generative AI Profile** and the **G7 Hiroshima Process International Code of Conduct** (§8).
- **21 July 2025 — ISO crosswalk.** A crosswalk to **ISO/IEC 42001:2023** was published.
- **2026 — professionalisation.** The Foundation introduced an **AI Tester Accreditation Programme** ("first of its kind in Asia" — codifying what good AI testing looks like, aimed at third-party testing providers) and a **Global AI Assurance Sandbox** for testing GenAI in real-world settings. **[verify: exact programme launch dates per Foundation news; visible on the Foundation site as of August 2026]**

### 2.4 The History Table

| Year | Event | Notes |
|---|---|---|
| **2019** | Model AI Governance Framework launched at Davos (world's first); 2nd edition Jan 2020 | The principles ancestor of AI Verify; IMDA + PDPC |
| **2022 (Jan)** | **AI Verify announced** at the World Economic Forum by Minister Josephine Teo | Positioning: objective, verifiable demonstration of responsible AI |
| **2022 (25 May)** | **AI Verify launched** by IMDA + PDPC as MVP — **world's first** AI governance testing framework and toolkit | Launched "for international pilot and feedback"; developed in consultation with AWS, DBS, Google, Meta, Microsoft, SIA, NCS/LTA, Standard Chartered, UCARE.AI, X0PA.AI |
| **2023 (7 Jun)** | **Framework and toolkit open-sourced** on GitHub (`aiverify-foundation/aiverify`, Apache-2.0) | Turned a national instrument into a global, community-extensible one |
| **2023 (7 Jun)** | **AI Verify Foundation launched** at ATxAI/ATxSG | Global consortium; 7 founding premier members incl. IMDA, Google, IBM, Microsoft, Red Hat, Salesforce, Aicadium (§6) |
| **2024 (16 Jan)** | Proposed Model AI Governance Framework for Generative AI published | 9 dimensions; foreshadows the GenAI testing extension |
| **2024 (31 May)** | **Project Moonshot launched** (open beta) at ATxSG | One of the world's first LLM evaluation toolkits; benchmarking + red teaming (§7) |
| **2024 (May)** | Final Model AI Governance Framework for Generative AI | System-centric accountability, content provenance, safety alignment, red teaming |
| **2025 (29 May)** | **AI Verify testing framework updated** — now covers traditional AND generative AI | Same-day crosswalks: NIST GenAI profile, Hiroshima Process CoC |
| **2025 (21 Jul)** | ISO/IEC 42001 crosswalk published | Management-system alignment for certification-minded firms |
| **2026** | AI Tester Accreditation Programme; Global AI Assurance Sandbox | Professionalisation of third-party AI testing **[verify: exact dates]** |

**Reading the history:** AI Verify's trajectory is *principles → tests → open source → consortium → GenAI → standards alignment*. Each step widened its surface: from a Singapore-only pilot to a global open-source community, from traditional models to LLMs, from a national instrument to one with published crosswalks to NIST, ISO and the G7. For a bank, the practical consequence is that an AI Verify-style evidence file produced today is already aligned with the international standards your MRM and audit functions will recognise (cross-ref [audit_as_code_guide.md](audit_as_code_guide.md)).

### 2.5 The Adoption Coverage

**Verified (named adopters and testimonial quotes from primary sources).** The AI Verify Foundation's toolkit page carries adoption testimonials from organisations that have run the toolkit, including:

- **X0PA** — "AI Verify Toolkit provides a robust framework for assessing and improving the fairness of AI systems. It offers clear and actionable insights that organizations can use to address bias, which is essential for responsible AI deployment."
- **Dell** — "The AI Verify Toolkit provides a unified framework for organisations to implement honest, fair and equitable systems."
- **Singapore Airlines** — "AI Verify Toolkit is a great starting point for organisations as it provides a robust testing framework checklist to conduct self-assessments on AI systems. This is key in contributing to the development of international standards and industry benchmarks."

**Verified (development-consultation cohort).** The framework was developed with companies of different sectors and scale — **AWS, DBS Bank, Google, Meta, Microsoft, Singapore Airlines, NCS (Part of Singtel Group)/Land Transport Authority, Standard Chartered Bank, UCARE.AI, and X0PA.AI**. The two Singapore banks in that cohort (DBS, Standard Chartered) are significant for the banking angle: AI Verify's evidence pattern was shaped with bank input from the start (cross-ref [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md)).

**Partially verified (broader adoption claims).** Secondary coverage (e.g., The Business Times, Tech in Asia, GovInsider coverage of AI Verify and the Foundation) reports wider pilot participation — including international pilot cohorts and industry-vertical take-up — but the specific lists and counts vary by article and were **not fully verifiable against primary sources for this guide**. Treat claims like "N companies have used AI Verify" as **[verify]** unless traced to an IMDA/Foundation release. What is primary-source solid: the named testimonials above, the consultation cohort, the Foundation's international premier members, and the 2026 programmes (tester accreditation, assurance sandbox) that exist precisely because adoption reached the point where third-party testing became a market.

**The adoption pattern to note for a bank:** adoption clusters in (a) large corporates with public AI-governance commitments (cloud vendors, airlines, telecoms-adjacent NCS), (b) banks already deep in Singapore's AI-governance ecosystem (DBS, Standard Chartered), and (c) AI-native vendors (UCARE.AI, X0PA) whose products embed the testing story. The bank's own adoption case — produce the evidence, publish what policy allows, re-run on a lifecycle — follows the same playbook (see the worked example in §9).

---

## 3. The Components

### 3.1 The Testing Framework

The **AI Verify testing framework** is the governance instrument: a structured self-assessment of an AI system against the **11 internationally recognised AI governance principles**. It is designed to be usable by AI application owners/developers, internal compliance teams, and external auditors, and it is distributed free as a PDF and an Excel workbook (the "process checks" tool — `go.gov.sg/aivtf-pdf`, `go.gov.sg/aivtf-excel`).

**The four-layer structure (verified from the AI Verify Foundation):**

| Layer | What it is |
|---|---|
| **Principles** | Overarching considerations an AI application should adhere to (the 11 principles — §4) |
| **Outcomes** | For every principle, desired outcomes — technical and non-technical processes plus technical tests where applicable |
| **Processes** | Actionable steps to be carried out to achieve the desired outcomes |
| **Evidence** | The processes are validated by documentary evidence |

In practice, the framework is a **checklist with teeth**: for each principle you must (a) state which outcomes apply to your system, (b) document the processes you run, and (c) attach the evidence. If you use the official process-checks tool, it generates a **summary report showing your alignment** with the framework — usable to demonstrate responsible AI, identify improvement areas, and build stakeholder trust. The Foundation publishes a sample report so organisations can see the deliverable's shape before starting.

**Process checks vs. technical tests.** The framework splits assessment into two complementary halves:

- **Process checks** — documentary review against governance practices: risk management, data governance, transparency documentation, accountability structures, human oversight. These evidence the *organisational* principles (accountability, human agency and oversight, data governance, transparency, safety processes).
- **Technical tests** — automated, quantitative tests run on the model itself (fairness, explainability, robustness, performance) — §5. These evidence the *model-level* principles.

**The 2025 GenAI update (verified).** The version released on **29 May 2025** extends the framework to generative AI applications, so a single framework now covers both traditional and GenAI use cases. The GenAI edition keeps the same 11 principles but the outcomes/processes are adjusted for LLM characteristics (prompt design, RAG grounding, content provenance, guardrails, red teaming as evidence).

### 3.2 The Open-Source Toolkit

The **AI Verify toolkit** is the software half: an open-source test harness for **traditional ML models** (classification and regression), hosted at **GitHub `aiverify-foundation/aiverify`** (Apache-2.0; open-sourced 7 June 2023; web-application frontend in TypeScript with a Python test engine). It is aimed at "data scientists and compliance teams" to "streamline your AI testing process, and ensure the reliability and fairness of your models" (Foundation's description).

**What the toolkit does (verified from the Foundation's toolkit page):**

- **Comprehensive technical tests** — "from fairness in classification models, to robustness to image corruption", the toolkit supports a broad suite of automated technical tests for traditional ML models (the three headline families are **explainability, fairness and robustness**, per the IMDA primer, plus model performance).
- **Customizable reports** — organisations can shape reports to their needs, and combine the technical-test results with the process checks into one full report.
- **Parallelized workflow** — developers can run technical tests while compliance teams work on their checklists independently, then combine into the final report. This is a deliberate operational design: it lets the model team and the governance team work in parallel without blocking each other.
- **Extensibility** — new tests can be added centrally (e.g., the **Veritas toolkit** for the financial sector — MAS's FEAT assessment methodologies) or via plug-ins (e.g., from the Competition and Consumer Commission of Singapore, CCCS). The plug-in architecture is how sector-specific requirements get layered onto the national instrument.

**The run pattern.** The toolkit runs as a **dockerized test harness around a model**: you bring your model and a prepared dataset; the toolkit runs the selected tests, computes metrics, and emits a report. It is designed to be deployed in the developer's/user's environment (so models and data never have to leave the company — a key adoption factor for banks with strict data-residency rules). Testimonials on the Foundation site (X0PA, Dell, Singapore Airlines) emphasise exactly this: a framework for assessing fairness, a unified way to implement honest and equitable systems, and a "robust testing framework checklist to conduct self-assessments" that contributes to international standards.

### 3.3 The Components Table

| Component | What it is | Form | Licence / Status | Home |
|---|---|---|---|---|
| **AI Verify Testing Framework** | The 11-principle governance assessment: principles → outcomes → processes → evidence; process checks + documentary evidence; traditional + GenAI (since 29 May 2025) | PDF + Excel workbook (process-checks tool generates an alignment report) | Free; official instrument of IMDA / AI Verify Foundation | `go.gov.sg/aivtf-pdf`, `go.gov.sg/aivtf-excel` |
| **AI Verify Toolkit** | Open-source technical test harness for **traditional ML models**: explainability, fairness, robustness, performance tests; dockerized, parallelized workflow, extensible via plug-ins (e.g., Veritas for finance, CCCS) | Software (web UI + test engine), GitHub repo | **Apache-2.0**, open-sourced 7 June 2023 | `github.com/aiverify-foundation/aiverify` |
| **Project Moonshot** | LLM evaluation toolkit: benchmarking (100+ datasets) + manual/automated red teaming for LLM apps; web UI + CLI + library APIs; CI/CD integration; IMDA Starter Kit workflows | Software (Python 3.11), GitHub repo | **Apache-2.0**, launched 31 May 2024 (open beta) | `github.com/aiverify-foundation/moonshot` |
| **IMDA Starter Kit for LLM-based App Safety Testing** | Guided benchmark workflow (safety testing of LLM applications) implemented inside Moonshot | Guidance + tooling | Free | IMDA / Foundation sites |
| **Crosswalks** | Official mappings of the framework to NIST AI RMF (Oct 2023), NIST GenAI profile (May 2025), Hiroshima Process CoC (May 2025), ISO/IEC 42001 (Jul 2025) | PDFs | Free | Foundation site (§8) |
| **Sample report** | Dummy AI Verify report showing what an assessment output comprises | PDF | Free | Foundation site |

> **Component logic in one line:** the *framework* defines the assessment (11 principles, evidence-based); the *toolkit* automates the technical tests for traditional models; *Moonshot* does the same for LLMs; the *crosswalks* make the output recognisable to international standards; the *Foundation* governs it all.

### 3.4 The End-to-End Workflow

How the components compose in a real engagement (this is the skeleton the worked example in §9 fleshes out):

1. **Scope.** Select the AI system(s); complete the AI inventory entry (cross-ref the inventory discipline in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §7.1); decide which of the 11 principles apply and at what depth.
2. **Process checks (framework).** Work through the Excel process-checks tool: for each applicable principle, document outcomes → processes → evidence. Generate the alignment summary report.
3. **Technical tests (toolkit or Moonshot, in parallel).** Model team runs the automated tests in the dockerized environment (traditional) or the Moonshot environment (LLM); compliance team finishes the process checks independently.
4. **Combine.** Merge technical-test results with process-check evidence into one **AI Verify report** (customizable template; sample report published by the Foundation shows the expected shape).
5. **Review and publish.** Internal sign-off (model owner, MRM, compliance); publish the report (or an extract) as policy allows — self-testing means the company controls disclosure.
6. **Maintain.** Re-run on model/data change, on a scheduled cadence, and when the framework itself is updated (as in May 2025) — the report is a lifecycle artifact, not a certificate.

This workflow is deliberately the same shape as the trust-assessment process in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §7 (inventory → classify → risk register → test → decide → monitor → report) — AI Verify is the Singapore-flavoured, tool-supported implementation of that generic process.

---

## 4. The Principles Mapping

### 4.1 The 11 Governance Principles (the Exact List)

**Verified against the AI Verify Foundation's "What is AI Verify" page (primary source).** The framework assesses AI systems against **11 internationally recognised AI governance principles**, drawn from Singapore's **Model AI Governance Framework** (2nd edition, 2020) and consistent with the EU, OECD and G7 instruments. The exact list, with the Foundation's own glosses:

1. **Transparency** — ability to provide responsible disclosure to those affected by AI systems, so they can understand the outcome.
2. **Explainability** — ability to assess the factors that led to the AI system's decision, its overall behaviour, outcomes, and implications.
3. **Repeatability / Reproducibility** — the ability of a system to consistently perform its required functions under stated conditions for a specific period, and for an independent party to produce the same results given similar inputs.
4. **Safety** — AI should not result in harm to humans (particularly physical harm), and measures should be in place to mitigate harm.
5. **Security** — protection of AI systems, their data and associated infrastructure from unauthorised access, disclosure, modification, destruction or disruption (confidentiality, integrity, availability).
6. **Robustness** — the AI system should be resilient against attacks and attempts at manipulation by malicious third parties, and should still function without producing undesirable output despite unexpected input.
7. **Fairness** — AI should not result in unintended and inappropriate discrimination against individuals or groups.
8. **Data Governance** — governing data used in AI systems, including good governance practices for data quality, lineage, and compliance.
9. **Accountability** — AI systems should have organisational structures and actors accountable for the proper functioning of the systems.
10. **Human Agency and Oversight** — ability to implement appropriate oversight and control measures, with humans-in-the-loop at the appropriate juncture.
11. **Inclusive Growth, Societal and Environmental Well-being** — the potential for trustworthy AI to contribute to growth and prosperity for all — individuals, society and the planet — and to advance global development objectives.

> **Source hygiene note (important):** some secondary literature gives shortened or altered versions of this list (e.g., swapping "security" for "privacy", or "human autonomy" for "human agency and oversight"). The list above is the **Foundation's current official list** and is the one to cite in formal documents. Similarly, the acronym "MLTC" for the original test criteria appears in secondary commentary but **could not be verified against primary sources** — the primary record consistently describes the testing scope as the 11 principles (the same flag appears in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3.2).

### 4.2 The Mapping Table

How the 11 principles map to what an assessment actually produces. The mapping below combines the framework's process checks (documentary) with the toolkit's technical tests (quantitative) — the "Test / Evidence" column shows the primary mechanism for each principle:

| # | Principle | Primary test / evidence mechanism | Notes |
|---|---|---|---|
| 1 | **Transparency** | Process checks: disclosure documentation, model documentation, user-facing communication of AI use and limitations | The publishable report itself is transparency evidence; aligns with FEAT transparency and EU AI Act limited-risk disclosure duties |
| 2 | **Explainability** | **Technical test**: SHAP-based feature attribution and explainability scoring on model predictions | The flagship technical test; supports conceptual-soundness review and regulator explanations (cross-ref XAI in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.4) |
| 3 | **Repeatability / Reproducibility** | Process checks + test discipline: pinned model/data versions, deterministic test harness, documented reproduction steps | The dockerized toolkit makes repeatable runs a technical fact, not a promise |
| 4 | **Safety** | Process checks: harm assessment, mitigation measures; (GenAI: safety benchmarks, red-team evidence via Moonshot) | Physical-harm emphasis in the traditional framework; content-harm and misuse in the GenAI edition |
| 5 | **Security** | Process checks: access control, data protection, infrastructure security; robustness tests cover adversarial manipulation | Cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md) and [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the platform controls |
| 6 | **Robustness** | **Technical test**: input perturbation / corruption tests (e.g., image corruption, feature perturbation), stability metrics under unexpected input | One of the three headline toolkit test families (with explainability and fairness) |
| 7 | **Fairness** | **Technical test**: group fairness metrics on classification models (e.g., demographic parity, equal opportunity, equalized odds) sliced by protected attributes | The other headline test family; state *which* fairness criterion you claim (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.3) |
| 8 | **Data Governance** | Process checks: data quality, lineage, compliance documentation for training/test data | Cross-ref [data_governance_guide.md](data_governance_guide.md) for the enterprise practice |
| 9 | **Accountability** | Process checks: RACI, ownership, sign-off chains, incident response, escalation paths | The organisational principle; evidence is governance documentation |
| 10 | **Human Agency and Oversight** | Process checks: human-in-the-loop design, override and appeal mechanisms, oversight cadence | For a bank, the review-queue design of any copilot is the evidence |
| 11 | **Inclusive Growth, Societal and Environmental Well-being** | Process checks: inclusion and sustainability considerations, societal-impact assessment | The "public good" principle; least testable technically, most dependent on documented intent and review |

> **Mapping logic for practitioners:** principles 2, 6, 7 (explainability, robustness, fairness) are the *technical-test* principles — the toolkit produces numbers for them. Principles 1, 3, 4, 5, 8, 9, 10, 11 are *process-and-evidence* principles — the framework's process checks produce documents for them. An AI Verify report is complete only when both halves are done: **the toolkit covers the model, the framework covers the organisation.**

### 4.3 Using the Mapping in Practice

The 11-principle list is the framework's full menu, but no assessment runs all 11 at equal depth. The mapping functions as a **triage tool**: for each system type, some principles bite hardest and get the technical tests, while others get process-check evidence:

| System type | Principles that bite hardest | Technical tests likely required | Process-check emphasis |
|---|---|---|---|
| **Internal screening/prioritisation model** (e.g., the worked example's SCORE-Screen) | Fairness, Explainability, Robustness, Repeatability | Fairness (group metrics), explainability (SHAP), robustness (perturbation), performance | Accountability, Human Agency and Oversight, Data Governance |
| **GenAI copilot / assistant** (e.g., DOC-Ops) | Safety, Security, Robustness, Transparency, Human Agency | Moonshot benchmarks (bias, toxicity, hallucination) + red teaming (injection, jailbreaks) | Transparency (disclosure), Accountability, Human Agency and Oversight, Data Governance |
| **Customer-facing recommendation/pricing** | Fairness, Transparency, Data Governance | Fairness tests; explainability for adverse-outcome review | Disclosure documentation; appeal/override mechanisms |
| **Internal productivity tooling** (low stakes) | Data Governance, Accountability, Security | Performance baseline only; selected robustness | RACI; data lineage; access control |

The triage decision — which principles apply at which depth — must itself be documented (that documentation is transparency evidence). The framework's process-checks tool walks through this systematically, which is why it is the recommended starting point even before any test is run.

---

## 5. The Technical Tests

### 5.1 The Test Families

**Verified scope:** the IMDA primer describes the toolkit as providing a "guided workflow for technical testing of **Explainability, Robustness and Fairness**", and the Foundation's toolkit page confirms the breadth ("from fairness in classification models, to robustness to image corruption") plus model-performance testing. The three headline families, as they apply to traditional supervised models:

**1. Fairness tests.** Measure whether a model treats groups differently on protected attributes (where such data can lawfully be collected). The standard approach is group-fairness metrics on the test set, sliced by protected groups:

- **Demographic parity (statistical parity)** — equal positive/acceptance rates across groups.
- **Equal opportunity** — equal true-positive rates (recall) across groups.
- **Equalized odds** — equal false-positive and false-negative rates across groups.
- **Disparate-impact ratio** — the ratio of favourable outcomes between groups (the 80% rule's cousin).

The fairness criteria conflict mathematically, so an AI Verify assessment must state which criterion is claimed and why (e.g., *equal opportunity* for a screening model where recall errors, not approval-rate differences, are the harm). Cross-ref the full bias-audit treatment in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.3 and [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md).

**2. Explainability tests.** Produce human-understandable accounts of model behaviour, typically via **SHAP (SHapley Additive exPlanations)** feature attributions — how much each input feature contributed to each prediction. The toolkit computes attribution scores and summarises them (feature-importance ranking, attribution consistency), supporting conceptual-soundness review, debugging, and regulator-facing explanations. (LIME is the common companion for local surrogate explanations; the toolkit's documented flagship is SHAP-based attribution. Cross-ref XAI methods in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.4.)

**3. Robustness tests.** Measure stability and graceful degradation under input perturbation:

- **Input perturbation** — adding noise or small changes to input features and measuring prediction stability (the model should not flip outputs on trivial changes).
- **Image corruption / transformation tests** — for vision models, robustness to blur, rotation, compression, lighting changes etc. (the "robustness to image corruption" the Foundation cites).
- **Adversarial-style probes** — inputs crafted to induce misclassification; the boundary between robustness testing and adversarial-ML red teaming (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.2 and the adversarial-ML guides in the ai_llm cluster).

**4. Model performance tests.** Baseline accuracy/precision/recall/F1-type metrics on a prepared dataset — the "verifies claimed performance of AI systems" function in the primer. Performance is the sanity layer: fairness and robustness claims are only meaningful relative to a model that performs its core task.

**Beyond the toolkit — the GenAI test catalogue.** For generative AI, the technical test families extend to the LLM domain via **Project Moonshot** (benchmarking + red teaming — §7) and the 2025 framework update: safety benchmarks, bias/toxicity suites, hallucination/grounding tests, jailbreak and prompt-injection attack modules, refusal-consistency checks. The IMDA **Starter Kit for LLM-based App Safety Testing** provides the guided workflow (implemented inside Moonshot) so companies run a standardised safety-test set rather than inventing their own.

### 5.2 The Tests Table

| Test family | Principle served | What it measures | Example tests / metrics | Toolkit support |
|---|---|---|---|---|
| **Fairness** | Fairness (principle 7) | Group-level differences in model outcomes on protected attributes | Demographic parity, equal opportunity, equalized odds, disparate-impact ratio on protected-group slices | AI Verify Toolkit (traditional classification models) |
| **Explainability** | Explainability (2) | How much each feature contributed to predictions; whether behaviour can be understood | SHAP feature attributions, attribution summary/ranking, consistency checks | AI Verify Toolkit (SHAP-based) |
| **Robustness** | Robustness (6) | Stability under unexpected or maliciously perturbed input | Input perturbation, image corruption/transformation tests, adversarial-style probes, stability metrics | AI Verify Toolkit |
| **Model performance** | Repeatability (3) + overall validity | Whether the model performs its claimed task | Accuracy, precision, recall, F1, AUC on prepared datasets | AI Verify Toolkit |
| **Process checks** | Transparency (1), Safety (4), Security (5), Data Governance (8), Accountability (9), Human Agency (10), Inclusive Growth (11) | Documentary evidence of governance processes | Disclosure docs, risk mgmt, data lineage, RACI, oversight design, harm assessment | Framework (PDF/Excel), not the toolkit |
| **GenAI: benchmarks** | Safety (4), Robustness (6), Fairness (7) | LLM performance and trust/safety profile | 100+ benchmark datasets: capability, quality, trust & safety (bias, toxicity, hallucination); IMDA Starter Kit workflows | Project Moonshot |
| **GenAI: red teaming** | Security (5), Robustness (6), Safety (4) | Adversarial failure modes of LLM applications | Jailbreak suites, prompt injection, harmful-content elicitation, automated attack modules, manual red-team sessions | Project Moonshot |

> **Honest-scope note:** the *published, documented* technical tests of the traditional toolkit centre on explainability, fairness, robustness and performance; the remaining principles are evidenced by process checks rather than automated tests. Do not overclaim ("AI Verify tested my model for security") — an AI Verify report evidences security via *documented controls*, not an automated pentest. The GenAI side (Moonshot) does bring automated adversarial testing into scope for LLM applications.

### 5.3 Operational Notes on Running the Tests

Practical points that determine whether the technical-test results are usable as evidence:

- **Data preparation is the real work.** The toolkit tests a model against *your* prepared datasets. For fairness tests that means labelled test sets with protected-attribute slices (where lawful to hold them); for robustness that means perturbation definitions appropriate to the model's input domain; for explainability that means a representative sample of predictions. Budget for this: data prep typically dominates the run calendar (the worked example's Phase 1 assumes it as step 1).
- **The environment is yours.** The dockerized, in-environment design means models and data never leave the company — a decisive feature for banks bound by data-residency and confidentiality rules, and the reason "run it yourself" is a selling point, not a shortcoming.
- **Metrics need a criterion, not just a number.** A fairness metric without a stated criterion (demographic parity vs. equal opportunity vs. equalized odds) is uninterpretable; a robustness metric without a perturbation budget is unverifiable. State both before running, and record them in the report (cross-ref the bias-audit discipline in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §4.3).
- **Determinism is an evidence property.** Pinned model versions, pinned datasets and pinned test-harness versions are what make a run *repeatable* (principle 3). Treat the toolkit's docker images and Moonshot's versioned releases as part of the evidence chain — record versions in the report.
- **Tests are a floor, not a ceiling.** The published test set is the nationally recommended minimum for the covered principles; the plug-in architecture (Veritas for finance, CCCS plug-ins) and custom Moonshot recipes are how you extend it to sector-specific and use-case-specific risks. A bank should plan to extend, not just to run.
- **Reports are the product.** The customizable report template plus the sample report published by the Foundation define the deliverable's shape; keep raw outputs (JSON from Moonshot, metric tables from the toolkit) archived alongside the human-readable report — auditors want the raw layer (cross-ref [audit_as_code_guide.md](audit_as_code_guide.md) for machine-checkable evidence patterns).

---

## 6. The AI Verify Foundation

### 6.1 The Consortium (June 2023)

**Verified:** The **AI Verify Foundation** was launched on **7 June 2023** at the **ATxAI conference** (part of Asia Tech x Singapore, ATxSG), announced by Minister Josephine Teo, to "harness the collective power and contributions of the global open source community to develop AI testing tools for the responsible use of AI". On the same day the AI Verify framework and toolkit were open-sourced on GitHub — the open-sourcing and the Foundation launch were deliberately paired.

**What it is:**

- A **not-for-profit**, wholly owned subsidiary of IMDA that runs its day-to-day operations independently. (The "wholly owned subsidiary of IMDA" structure is reported by industry analysis; the Foundation describes itself as "a global open-source community that convenes AI owners, solution providers, users, and policymakers, to build trustworthy AI".)
- A **global consortium** with a **tiered membership** (premier and general members) — the mechanism by which industry co-owns and co-develops the testing instruments.
- The **steward of the AI Verify framework and toolkit**, the operator of **Project Moonshot**, the publisher of **international crosswalks**, and (from 2026) the operator of the **AI Tester Accreditation Programme** and the **Global AI Assurance Sandbox**.

**Mission in practice:** the Foundation is where the *national* instrument became a *global* one. Its work products are the evidence: the framework updates (May 2025 GenAI edition), the crosswalks (NIST, Hiroshima CoC, ISO 42001), the Moonshot launch with international design partners, the ML Commons partnership on LLM safety benchmarks, and the accreditation programme for third-party testers.

### 6.2 The Members

**Verified (founding set):** the Foundation launched with **seven pioneering premier members** — **IMDA, Aicadium (Temasek's AI Centre of Excellence), IBM, Microsoft, Google, Red Hat, and Salesforce**. The premier-member tier is the governance core; general members contribute as the wider community.

**Verified (growth):** the Foundation's site (materials dated 2024) shows the premier tier expanded to include **AWS, Dell, and Resaro** alongside the founders — nine premier members on display as of 2024. **[verify: the current full member roster and tier split as of 2026; the Foundation's members page carries the live list]**.

**What membership means in practice:** premier members co-develop tests, contribute engineering (e.g., test plug-ins, cloud deployment patterns), and act as reference adopters; general members (including banks and government agencies) pilot the instruments and feed back. The DBS/Standard Chartered participation in the original development consultations carried forward into the Foundation era — Singapore's banks are represented in the consortium's orbit, which is why the instruments fit bank governance so naturally.

### 6.3 The Foundation Table

| Aspect | Detail |
|---|---|
| **Launch** | **7 June 2023** at ATxAI (part of ATxSG), announced by Minister Josephine Teo; paired with the open-sourcing of AI Verify on GitHub |
| **Type** | Not-for-profit global consortium; wholly owned subsidiary of IMDA running day-to-day operations independently |
| **Mission** | Convene AI owners, solution providers, users, and policymakers to build trustworthy AI; develop AI testing tools with the global open-source community |
| **Founding premier members (7)** | IMDA, Aicadium (Temasek), IBM, Microsoft, Google, Red Hat, Salesforce |
| **Premier members added (by 2024)** | AWS, Dell, Resaro **[verify: current full roster]** |
| **Membership model** | Tiered: premier + general members; open community contributions to the codebases |
| **Core products** | AI Verify testing framework (updated 29 May 2025), AI Verify toolkit (traditional ML), Project Moonshot (LLM evaluation, launched 31 May 2024) |
| **Standards work** | Crosswalks: NIST AI RMF (13 Oct 2023), NIST GenAI profile (29 May 2025), Hiroshima Process CoC (29 May 2025), ISO/IEC 42001 (21 Jul 2025); partnership with ML Commons on LLM safety benchmarks |
| **2026 programmes** | AI Tester Accreditation Programme (first of its kind in Asia); Global AI Assurance Sandbox for real-world GenAI testing **[verify: exact dates]** |
| **Why it matters for a bank** | The Foundation is the credible, internationalised steward — referencing its instruments and crosswalks gives a bank's AI evidence global recognisability, not just local validity |

---

## 7. The Project Moonshot

### 7.1 The LLM Evaluation Toolkit (2024)

**Verified:** **Project Moonshot** — officially **"AI Verify – Project Moonshot"** — was launched on **31 May 2024** in open beta at **Asia Tech x Singapore (ATxSG) 2024** (ATxAI conference) by Minister Josephine Teo. IMDA's press release describes it as "a generative Artificial Intelligence testing toolkit to address LLM safety and security challenges", and independent coverage (US-ASEAN Business Council, Cybersecurity Asia) confirms the launch date, venue and open-beta status. It is described by the Foundation as **one of the world's first Large Language Model (LLM) Evaluation Toolkits**.

**What it is.** An open-source tool (GitHub `aiverify-foundation/moonshot`, **Apache-2.0**; repository created 14 December 2023; Python 3.11; in beta at version 0.7.x) that brings **benchmarking and red teaming together** in one platform, so LLM application developers and compliance teams can test and evaluate LLMs and LLM applications. It answers the two questions companies actually ask: *which foundation model best suits our goals?* and *is our application, built on that model, robust and safe?*

**What it does (verified from the Foundation's Moonshot page and the repo README):**

- **Comprehensive benchmarking.** 100+ (and growing) benchmark datasets with pre-built evaluators, across **performance** (accuracy, BLEU, etc.) and **trust & safety** (bias, toxicity, hallucination). Benchmarks include community staples used by Hugging Face leaderboards and BigBench, plus domain-specific tests. Benchmark categories: Capability, Quality, Trust & Safety.
- **Manual and automated red teaming.** Adversarial prompting to induce behaviour incongruent with the model's design; **automated attack modules** generate adversarial prompts at scale (since human-only red teaming doesn't scale), and manual red-team sessions are supported for expert probing. This is the same red-teaming discipline as [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md), industrialised as software.
- **IMDA Starter Kit integration.** Guided workflows ("cookbooks") implement **IMDA's Starter Kit for LLM-based App Safety Testing**, so a company can run the nationally recommended safety-test set out of the box.
- **Interfaces and integration.** A **Web UI** (guided workflow), an interactive **CLI**, and **library/Web APIs** for MLOps integration — including **CI/CD pipeline integration** for unsupervised test runs. Reports are HTML with interactive charts plus raw JSON for programmatic analysis.
- **Customisation.** Users select only the relevant tests for their application and can add custom datasets/recipes for use-case-specific evaluation.

**Ecosystem.** Design partners and contributors include major corporates (the Foundation's May-2024 materials show, among others, **Prudential** and **Thomson Reuters** as named design partners; other logos are unnamed in the page's alt text **[verify: full design-partner roster]**), and the Foundation is partnering **ML Commons** to develop globally aligned LLM safety benchmarks.

**Positioning vs. the AI Verify toolkit (Foundation's own framing):** "Project Moonshot covers LLMs, and AI Verify Toolkit covers traditional AI." They are the two technical-test halves of the same framework — traditional models get the toolkit, GenAI gets Moonshot, and the 29 May 2025 framework update lets one report cover both. For evaluation-method context (metrics, harness design, RAG evaluation), cross-ref [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md), [ai_llm/rag/rag_evaluation_methodology_guide.md](ai_llm/rag/rag_evaluation_methodology_guide.md), and [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) (the eval-angle contrast).

### 7.2 The Moonshot Table

| Aspect | Detail |
|---|---|
| **Launch** | **31 May 2024** at ATxSG 2024 (ATxAI), open beta, launched by Minister Josephine Teo; "AI Verify – Project Moonshot" |
| **What it is** | One of the world's first **LLM evaluation toolkits**; open-source; benchmarking + red teaming in one platform |
| **Repo / licence** | `github.com/aiverify-foundation/moonshot`; **Apache-2.0**; repo created 14 Dec 2023; Python 3.11; beta (v0.7.x) |
| **Benchmarking** | 100+ datasets with pre-built evaluators: performance (accuracy, BLEU) and trust & safety (bias, toxicity, hallucination); community benchmarks incl. BigBench, Hugging Face leaderboard staples; categories: Capability, Quality, Trust & Safety |
| **Red teaming** | Manual red-team sessions + **automated attack modules** (algorithmic/LLM-generated adversarial prompts); prompt templates and context strategies |
| **Starter Kit** | Built-in guided workflows implementing **IMDA's Starter Kit for LLM-based App Safety Testing** |
| **Interfaces** | Web UI (guided workflow), interactive CLI, library/Web APIs; CI/CD integration; HTML reports + raw JSON |
| **Customisation** | Test selection by use case; custom datasets, recipes, grading scales, connectors |
| **Ecosystem** | Design partners incl. **Prudential, Thomson Reuters** **[verify: full roster]**; **ML Commons** partnership for global LLM safety benchmarks |
| **Relationship to AI Verify** | The GenAI technical-test companion to the traditional AI Verify toolkit; feeds the 11-principle framework's GenAI edition (2025) |

### 7.3 Moonshot in the Evaluation-Tool Landscape

Moonshot enters a crowded LLM-evaluation space (RAGAS, DeepEval, HELM, lm-eval-harness and similar — see the eval guides in the ai_llm cluster). Its differentiators, which matter for a bank choosing an evaluation stack:

- **Governance-integrated by origin.** Moonshot is not a standalone metrics library; it is the technical-test arm of a governance framework. Its benchmark outputs map onto the 11-principle assessment and the IMDA Starter Kit, so the same run produces both engineering insight *and* governance evidence — no separate translation step.
- **Red teaming built in.** Benchmarking (measure how good the model is) and red teaming (probe how it fails adversarially) live in one tool, with automated attack modules that scale what would otherwise be manual red-team effort (cross-ref the red-teaming discipline in [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md)).
- **Compliance-team ergonomics.** The Web UI and guided workflows are designed for compliance teams, not just ML engineers — the parallelized "developers run tests, compliance runs checklists" pattern of the traditional toolkit carries over.
- **Open and auditable.** Apache-2.0, with datasets, evaluators and attack modules inspectable — the auditability that [audit_as_code_guide.md](audit_as_code_guide.md) argues is the precondition for using evaluation output as evidence.

None of this makes Moonshot the *only* tool a bank needs — specialised RAG evaluation (e.g., the RAGAS/DeepEval family, cross-ref [ai_llm/rag/rag_evaluation_methodology_guide.md](ai_llm/rag/rag_evaluation_methodology_guide.md)) and internal golden sets remain necessary — but it makes Moonshot the natural *governance-grade* layer: the runs that go into the AI Verify report are the ones done here.

---

## 8. The International

### 8.1 The G7 Hiroshima Alignment

**Verified:** the AI Verify testing framework is explicitly aligned with international AI governance frameworks — the Foundation lists **EU, G7, OECD and US** frameworks as the reference set, and — critically — the alignment is *published, not asserted*: the Foundation has released official **crosswalks** mapping the AI Verify framework to:

- **US NIST AI RMF** (published **13 October 2023**);
- **US NIST AI RMF — Generative AI Profile** (published **29 May 2025**);
- **Hiroshima Process International Code of Conduct for Organizations Developing Advanced AI Systems** (published **29 May 2025**);
- **ISO/IEC 42001:2023** (published **21 July 2025**).

The **G7 Hiroshima AI Process** is the G7's framework for advanced-AI governance: at its May 2023 Hiroshima Summit the G7 launched the Hiroshima AI Process, which produced the **International Guiding Principles for Organizations Developing Advanced AI Systems** and the **International Code of Conduct** (agreed October 2023) — voluntary, non-binding expectations covering risk identification, safety, security, transparency, accountability, and testing/evaluation of advanced AI systems. Singapore is not a G7 member, but it participates in the Hiroshima AI Process's wider circle and, through the AI Verify Foundation, made its testing framework **directly mappable to the Code of Conduct** — the 29 May 2025 crosswalk. **[verify: Singapore's precise endorsement/participation status in the Hiroshima AI Process per official records; the crosswalk's existence and date are primary-source verified]**

**Why the crosswalks matter more than the endorsements.** A crosswalk is a working artifact: it lets an organisation that has completed an AI Verify assessment map its evidence onto NIST AI RMF categories, ISO 42001 clauses, or the Hiroshima Code of Conduct items — one evidence program, many international consumers. That is the same "one evidence program, many frameworks" pattern [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.6 describes for the global frameworks, now made concrete for AI Verify.

**Beyond the crosswalks — the internationalisation record:**

- **International pilot and feedback from day one** — the May 2022 MVP was explicitly launched "for international pilot and feedback", and the framework's development consultation drew on companies of different sectors and scale (AWS, DBS, Google, Meta, Microsoft, Singapore Airlines, NCS/LTA, Standard Chartered, UCARE.AI, X0PA.AI). **[verify: the specific list and countries of the post-launch international pilot cohort; primary releases name the consultation group, and secondary coverage of the 2022–2023 pilot cohort varies]**
- **Open source as the internationalisation engine** — the June 2023 open-sourcing let any organisation anywhere run, audit, and extend the toolkit.
- **The Foundation's global structure** — a consortium with international premier members (Google, IBM, Microsoft, Red Hat, Salesforce, AWS, Dell) and the ML Commons partnership on LLM safety benchmarks.
- **World's-first positioning in the global discourse** — Singapore consistently frames AI Verify as "the world's first" testing framework and as a "common global reference point" (PDPC's language) for objective, verifiable responsible-AI demonstration.

**For a global bank:** the practical consequence is that an AI Verify-style assessment in Singapore maps cleanly onto the standards your EU entities (EU AI Act), US operations (NIST AI RMF) and group standards (ISO 42001) already recognise — the crosswalks are the translation layer. Cross-ref [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) for the local adoption picture and [../management/business_case_development_guide.md](../management/business_case_development_guide.md) for how to price the assurance work. The same evidence-based posture runs through Singapore's public sector — the government agencies building AI systems operate under the same "test it, document it" expectations (cross-ref [htx_ngine_guide.md](htx_ngine_guide.md) and [ica_systems_guide.md](ica_systems_guide.md) for the SG govtech angle).

### 8.2 The International Table

| Initiative | What it is | Date / status | AI Verify linkage |
|---|---|---|---|
| **International pilot & feedback** | MVP launched for companies worldwide to pilot and feed back | **25 May 2022** | The launch mechanism itself; consultation group incl. AWS, DBS, Google, Meta, Microsoft, SIA, NCS/LTA, Standard Chartered, UCARE.AI, X0PA.AI |
| **EU / OECD / US alignment** | Framework consistency with EU, OECD and US AI governance instruments | Continuous (stated by PDPC/Foundation) | Basis of the framework's "internationally recognised" claim |
| **NIST AI RMF crosswalk** | Official mapping of AI Verify to NIST AI RMF 1.0 | **13 Oct 2023** | Evidence portability to the US risk-management framework |
| **G7 Hiroshima Process CoC crosswalk** | Mapping to the Hiroshima Process International Code of Conduct for Advanced AI developers | **29 May 2025** | The G7 alignment artifact — Japan's G7 presidency and the Hiroshima AI Process (2023) are the reference |
| **NIST AI RMF GenAI profile crosswalk** | Mapping to the NIST Generative AI Profile | **29 May 2025** | GenAI-era alignment |
| **ISO/IEC 42001 crosswalk** | Mapping to the certifiable AI management-system standard | **21 Jul 2025** | For organisations running/planning an AIMS |
| **Open-source community** | Global community runs, audits, extends the toolkit | Since **7 Jun 2023** | The internationalisation engine |
| **ML Commons partnership** | Joint development of globally aligned LLM safety benchmarks | Announced with Moonshot (2024) | Moonshot's benchmark credibility |
| **AI Tester Accreditation Programme** | Codifying good AI testing; third-party testers | 2026 **[verify: exact date]** | Professionalises the international testing market |
| **Global AI Assurance Sandbox** | Real-world GenAI testing programme | 2026 **[verify: exact date]** | The next internationalisation step |

---

## 9. The Worked Example: An AI Verify Assessment

### 9.1 The Scenario: A Cymbal Bank Model

**The context.** Cymbal Bank-style trade-finance operations in Singapore run two AI systems that the group wants to demonstrate as responsibly governed (the familiar context from [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §8, whose DOC-Ops Copilot this example extends):

1. **"SCORE-Screen"** — a traditional supervised ML model (gradient-boosted classifier) that **prioritises trade-document transactions for sanctions and anti-money-laundering review**: it scores transactions and flags those needing human investigation. This is the *traditional AI* case — exactly what the **AI Verify toolkit** targets.
2. **"DOC-Ops Copilot"** — the GenAI assistant (field extraction from shipping documents, RAG summarisation, banker Q&A) from the umbrella guide. This is the *generative AI* case — evaluated with **Project Moonshot** plus the 2025 framework's GenAI process checks.

**Why run AI Verify on them.** (a) The MAS/FEAT-aligned governance posture expects *objective, verifiable* evidence rather than assertions; (b) the models touch regulated activity (sanctions screening, trade operations) where model-risk documentation is expected under SR 11-7-style MRM (cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md)); (c) the AI Verify report is a **publishable artifact** — the same evidence file can satisfy internal governance, the group's audit function, and regulator conversations; and (d) Singapore is where the instruments were built — using them is the locally credible way to demonstrate "we tested it, here is the evidence" (cross-ref [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §3).

**Scope and tiering.** Both systems are **high materiality** (screening errors and extraction errors can drive financial or regulatory harm), so the full regime applies: all relevant process checks plus the technical tests, not a lightweight pass. The assessment is a **self-assessment in the AI Verify pattern** (company-run, evidence-based), with second-line (MRM/compliance) review of the evidence pack — the same split the framework's parallelized workflow was designed for.

### 9.2 The Toolkit-Run Design

**Phase 0 — Preparation (framework first).** Complete the process-checks workbook for both systems *before* running any tests: which of the 11 principles apply, which outcomes, which processes, and which evidence documents exist (model cards, data lineage, RACI, oversight design, incident response). This produces the **documentary half** of the report and tells you which technical tests you actually need (a pure internal screening model with no customer-facing output still needs fairness and explainability tests; its "transparency" evidence is internal disclosure, not customer notices).

**Phase 1 — SCORE-Screen through the AI Verify toolkit.** Standard run pattern:

| Step | Activity | Output |
|---|---|---|
| 1 | Containerise the model (docker) + prepare labelled test datasets with protected-attribute slices (counterparty region, entity type) where lawful | Model image; dataset manifest; data-lineage note (principle 8) |
| 2 | **Model performance** baseline: accuracy, precision, recall, F1, AUC on holdout | Performance report (validity basis) |
| 3 | **Fairness tests**: group-fairness metrics sliced by counterparty region and entity type — demographic parity, equal opportunity, equalized odds, disparate-impact ratio. Stated criterion: *equal opportunity* (the screening model must not systematically miss true flags for any region — recall errors are the harm) | Fairness report with criterion rationale |
| 4 | **Explainability tests**: SHAP attributions on a sample of scored transactions; feature-importance ranking; spot-check explanations against subject-matter expectations (conceptual soundness) | Explainability report; SHAP summary plots |
| 5 | **Robustness tests**: input perturbation (small changes to amount, party-name fields, feature noise) and stability metrics; confirm no output-flip on trivial changes | Robustness report |
| 6 | Combine technical reports + process-check evidence into the **AI Verify report** (customizable report template); internal review; publish internally (and externally only what policy allows — self-testing means *you* choose) | **AI Verify report: SCORE-Screen** |

**Phase 2 — DOC-Ops Copilot through Project Moonshot.** Because the copilot embeds an LLM, the GenAI side uses Moonshot rather than the traditional toolkit:

| Step | Activity | Output |
|---|---|---|
| 1 | Stand up Moonshot (Python 3.11); connect the model endpoint and the application (RAG + guardrails) via connectors | Moonshot environment; endpoint config |
| 2 | **Benchmark run**: trust & safety suites (toxicity, bias, hallucination) + capability/quality baselines; run the **IMDA Starter Kit** workflow for the safety-test set | Benchmark scorecard (baseline) |
| 3 | **Red teaming**: automated attack modules (jailbreak families, prompt injection — including document-embedded injection, the known killer finding for document-processing copilots, cross-ref [ai_llm/prompt_injection_guide.md](ai_llm/prompt_injection_guide.md)) plus manual expert red-team sessions; log every attempt with severity | Red-team log + report |
| 4 | Custom recipes: RAG grounding evaluation on the bank's document types (field extraction accuracy, no-answer behaviour) | Custom evaluation results |
| 5 | Combine with the 2025 framework's **GenAI process checks** (system-centric accountability, content provenance, safety alignment, oversight) into the report | **AI Verify report: DOC-Ops Copilot** |

**Phase 3 — Consolidation and assurance.** Merge both reports into one **evidence pack** (the deliverable): the two AI Verify reports, the raw test outputs, the process-checks workbook, and a sign-off chain (model owner, MRM reviewer, compliance). Map the pack onto the **crosswalks** (NIST AI RMF categories, ISO 42001 clauses, Hiroshima CoC items) so the same evidence serves the group's global standards — the cost-efficient pattern from [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2.6. Schedule re-runs: quarterly benchmark re-run for the copilot, re-assessment on model/data change, annual full cycle.

**Delivery plan.** The whole assessment runs as a six-week engagement on a fixed calendar:

| Week | Activity | Output |
|---|---|---|
| 1 | Inventory + scoping; process-checks workbook kickoff; acceptance criteria | Scope memo; workbook v1 |
| 2 | Data preparation (SCORE-Screen test sets with protected-attribute slices); Moonshot setup + endpoint connectors | Dataset manifests; test environment |
| 3 | Toolkit runs: performance + fairness + explainability; Moonshot benchmarks incl. the IMDA Starter Kit workflow | Metric reports; benchmark scorecard (baseline) |
| 4 | Toolkit robustness tests; Moonshot automated red teaming + manual red-team sessions | Robustness report; red-team log |
| 5 | Process-checks completion; findings consolidation; crosswalk mapping (NIST/ISO/Hiroshima) | Workbook final; evidence pack draft |
| 6 | Review, sign-off, report publication decision | AI Verify reports (both systems); sign-off chain |

**The report and its uses.** The two AI Verify reports are the *visible* deliverable; the evidence pack is the *durable* one. Publication is a policy decision, not a technical one: the reports can be shared internally (governance committee), with MRM/audit (validation file), with the group (consistent standards), or externally (customer/regulator conversations) — the framework's self-testing design means the company decides what leaves the building. What must *not* be optional is the archived raw layer (metrics, JSON, versions), because that is what makes the report checkable later (cross-ref [audit_as_code_guide.md](audit_as_code_guide.md)).

### 9.3 The Findings

| # | Finding (severity) | Where it surfaced | Action |
|---|---|---|---|
| 1 | **Fairness gap: field-recall / flag-recall 2–3 pp lower for one counterparty region's document formats** (Medium) | Toolkit fairness tests (equal opportunity) | Region-specific training/template coverage; monitoring dashboard; ops impact review |
| 2 | **SHAP attributions inconsistent with subject-matter expectations on ~5% of scores** (Low-Med) | Toolkit explainability tests | Feature-engineering review; documentation of the anomaly; re-run |
| 3 | **Output flip on small feature perturbation in edge cases** (Low) | Toolkit robustness tests | Input-validation hardening; threshold review |
| 4 | **Document-embedded prompt injection succeeds against the copilot** (High) | Moonshot automated red teaming | Gateway decontamination layer + instruction-separation guardrail (cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)); re-test all variants |
| 5 | **Hallucination on no-answer queries 3%** (Medium-High) | Moonshot trust & safety benchmarks | Grounding prompt + mandatory citations; no-answer training; monitor |
| 6 | **Process-check gaps: data lineage doc incomplete; oversight RACI stale** (Medium) | Framework process checks | Complete lineage for screening data; refresh RACI; add to inventory |
| 7 | **Bias in copilot summarisation on stereotyping probes** (Low-Med) | Moonshot bias suite | Prompt-level mitigation; monitoring |

**Sign-off:** conditional approval after findings 4–5 closed and 1, 6 under monitored mitigation — with re-test evidence attached to each closure. The point of the AI Verify pattern is that every finding above is attached to a *dated, owned artifact*, which is what makes the pack regulator-ready.

### 9.4 The Lessons

1. **The framework and the toolkit are two halves of one assessment.** Running the toolkit without the process checks produces numbers with no governance story; running the framework without the toolkit produces documents with no numbers. The report needs both — the parallelized workflow exists precisely so model teams and compliance teams can produce them concurrently.
2. **State the fairness criterion before you run the tests.** "Equal opportunity" vs. "demographic parity" changes what the fairness report shows; writing the rationale *first* is what makes the result defensible (the same lesson as the umbrella guide's bias audits).
3. **Different models, different instruments.** Traditional supervised models → AI Verify toolkit; LLM applications → Moonshot. Using the right instrument for each is itself part of demonstrating competence to a regulator.
4. **Self-testing means you control disclosure — and that is the point.** AI Verify's design lets a bank keep models and data in-house (dockerized, in-environment runs) while still producing publishable, comparable evidence. Data-residency and confidentiality constraints are solved by design, not by exception.
5. **The crosswalks are the force multiplier.** One evidence pack mapped onto NIST, ISO 42001 and the Hiroshima CoC serves MRM, group standards and international engagements simultaneously — the same "one evidence program, many consumers" economics as [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) and [../management/business_case_development_guide.md](../management/business_case_development_guide.md).
6. **Red teaming is where GenAI findings actually live.** For the copilot, the automated attack modules (injection, jailbreaks) found the real failures; benchmarks only positioned the model. Run both, and treat the red-team log as first-class evidence (cross-ref [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md)).
7. **An AI Verify report is a snapshot, not a certificate.** It evidences the state at test time; the monitoring and re-assessment schedule is what keeps the claim true. Treat the report as a lifecycle artifact (versioned, re-run on change), not a one-time deliverable.

---

## 10. The Summary

### 10.1 The One-Page Summary

**What AI Verify is.** Singapore's world-first AI governance **testing framework and open-source toolkit** (IMDA + PDPC, announced January 2022 at Davos, launched **25 May 2022** as an MVP for international pilot and feedback) for demonstrating responsible AI "in an objective and verifiable manner".

**What it tests.** An AI system's responsible implementation against **11 internationally recognised governance principles** — transparency, explainability, repeatability/reproducibility, safety, security, robustness, fairness, data governance, accountability, human agency and oversight, inclusive growth — via **process checks** (documentary evidence) and **technical tests** (automated metrics: fairness, explainability, robustness, performance).

**What the components are.** The **framework** (PDF/Excel process checks, updated 29 May 2025 to cover traditional + GenAI), the **toolkit** (open-source technical tests for traditional ML models, GitHub `aiverify-foundation/aiverify`, Apache-2.0), and **Project Moonshot** (the LLM evaluation toolkit — benchmarking + red teaming — launched 31 May 2024 at ATxSG, Apache-2.0).

**Who runs it.** The **AI Verify Foundation** (7 June 2023) — a not-for-profit global consortium (wholly owned subsidiary of IMDA) whose premier members include IMDA, Google, IBM, Microsoft, Red Hat, Salesforce, Aicadium, AWS, Dell and Resaro — stewarding the instruments, publishing international crosswalks, and (2026) accrediting third-party AI testers.

**Where it stands internationally.** Voluntary and self-testing by design; aligned with EU, OECD and US frameworks; official crosswalks to **NIST AI RMF** (Oct 2023), the **G7 Hiroshima Process Code of Conduct** (May 2025), the **NIST GenAI profile** (May 2025) and **ISO/IEC 42001** (Jul 2025).

**What it means for a bank.** An AI Verify-style assessment produces the objective, publishable evidence that MAS/FEAT-aligned governance, MRM (SR 11-7) and group standards recognise — one evidence program, many consumers — and it does so without models or data leaving the bank. It is the Singaporean way to answer the question every stakeholder asks: *show me, don't tell me.*

**What AI Verify is not (honest limits).** It is not a law (voluntary, self-testing), not a certification in the EU-Act conformity sense, not a safety guarantee (it evidences the covered principles *at test time*), and its traditional-model technical tests do not cover every principle automatically — the process checks carry the rest. It complements rather than replaces the hard-law and management-system frameworks: where the EU AI Act, NIST AI RMF or ISO 42001 apply, they still apply — the crosswalks exist to make AI Verify evidence *portable into* those regimes, not to substitute for them (cross-ref the framework-composition analysis in [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) §2). A mature bank treats AI Verify as the *evidence engine* of its AI governance, not as the governance itself.

### 10.2 The Final Word: The SG Testing Standard

AI Verify is not a law and not a certification — and that is precisely its design. It is Singapore's attempt to make responsible AI **measurable, evidence-based, and internationally legible**: the framework says what good looks like across 11 principles, the open-source toolkit makes the measurement repeatable, the report makes the evidence publishable, the Foundation makes the whole thing a global community project, and the crosswalks make the output recognisable to every major standard in the world.

For a Singapore-based bank, that combination is the practical definition of **the SG testing standard**: the local regulator's ecosystem has built the test, opened the code, and invited the world to use it — and the bank that runs it, documents it, and re-runs it on a lifecycle is speaking the language Singapore's AI governance posture and the international standards community both understand. **AI Verify is not the last word on AI trust — it is the first word that is actually checkable.** Use it, publish the evidence, and keep the record alive: that is how the SG testing standard becomes your standard.

---

## Glossary

- **AI Verify** — Singapore's world-first AI governance testing framework and open-source toolkit (IMDA + PDPC, launched 25 May 2022) for demonstrating responsible AI objectively and verifiably; stewarded by the AI Verify Foundation since June 2023.
- **IMDA** — Infocomm Media Development Authority; Singapore's infocomm and media regulator and developer of AI Verify with the PDPC; parent of the AI Verify Foundation.
- **PDPC** — Personal Data Protection Commission; Singapore's personal-data protection authority and co-launcher of AI Verify.
- **Testing framework** — the structured assessment instrument (principles → outcomes → processes → evidence) defining what responsible-AI implementation looks like and what evidence is required; here, the AI Verify testing framework.
- **Toolkit** — the accompanying software that automates the technical tests; here, the open-source AI Verify toolkit (`aiverify-foundation/aiverify`), dockerized, producing testing reports.
- **Open source** — software whose source is publicly available for use, inspection and contribution; AI Verify's toolkit and Moonshot are Apache-2.0-licensed open source.
- **Model AI Governance Framework** — Singapore's national AI governance framework (IMDA/PDPC, Jan 2019; 2nd ed. Jan 2020): the world's first; the principles ancestor of AI Verify.
- **Governance principle** — a high-level requirement for trustworthy AI (e.g., fairness, transparency); AI Verify tests against 11 internationally recognised principles.
- **Fairness** — the principle that AI should not result in unintended and inappropriate discrimination against individuals or groups; evidenced by group-fairness metrics (demographic parity, equal opportunity, equalized odds).
- **Explainability** — the principle that the factors behind an AI system's decisions and behaviour can be assessed; evidenced by SHAP-style feature attribution tests.
- **Robustness** — the principle that the system stays stable under unexpected or adversarial input; evidenced by perturbation/corruption tests.
- **Transparency** — the principle of responsible disclosure of AI use, capabilities and limitations; evidenced by documentation and the published report.
- **Security** — protection of the AI system, its data and infrastructure from unauthorised access, modification, destruction or disruption (confidentiality, integrity, availability).
- **Safety** — the principle that AI should not harm humans, with measures to mitigate harm (including content harm and misuse in GenAI).
- **Human agency** — the principle (Human Agency and Oversight) that appropriate oversight and control exist, with humans-in-the-loop at the right junctures.
- **Accountability** — the principle that organisational structures and named actors are accountable for the proper functioning of AI systems.
- **Inclusivity** — the principle (Inclusive Growth, Societal and Environmental Well-being) that trustworthy AI contributes to broad growth and well-being for individuals, society and the planet.
- **Data quality** — a component of the Data Governance principle: good practices for data quality, lineage and compliance in AI systems.
- **AI Verify Foundation** — the not-for-profit global consortium (launched 7 June 2023; wholly owned subsidiary of IMDA) that stewards AI Verify and runs Project Moonshot.
- **Consortium** — a group of organisations collaborating on a shared goal; the Foundation's tiered membership (premier + general members) co-develops the testing instruments.
- **Project Moonshot** — the AI Verify Foundation's open-source LLM evaluation toolkit (launched 31 May 2024): benchmarking (100+ datasets) + manual/automated red teaming.
- **LLM** — large language model; the foundation-model class of generative AI that Moonshot evaluates.
- **Evaluation** — systematic measurement of an AI system's performance and trust/safety profile via benchmarks, tests and metrics.
- **G7** — the Group of Seven major economies; its Hiroshima AI Process (2023) produced international guiding principles and a Code of Conduct for advanced-AI developers.
- **Hiroshima** — the 2023 G7 summit city; shorthand for the Hiroshima AI Process and its instruments, to which AI Verify publishes an official crosswalk.
- **Pilot** — a controlled early deployment for testing and feedback; AI Verify launched (May 2022) as an MVP "for international pilot and feedback".
- **Assurance** — the process and evidence that make AI trust verifiable (assessments, test reports, sign-offs); AI Verify is an assurance instrument.
- **Model card** — a documentation artifact describing a model's intended use, limitations, performance and governance details; key transparency evidence in an AI Verify assessment.
- **Crosswalk** — an official mapping between frameworks (e.g., AI Verify ↔ NIST AI RMF, Hiroshima CoC, ISO 42001) enabling evidence portability.
- **Process checks** — the framework's documentary assessment layer (governance practices evidenced by documents), as opposed to technical tests.
- **Report (AI Verify report)** — the combined output of technical tests and process checks, publishable by the assessed company.

---

## Appendix: Verification Log (August 2026)

| Claim | Status | Source |
|---|---|---|
| AI Verify announced January 2022 at WEF (Davos) by Minister Josephine Teo | **Verified** | IMDA / MDDI press release ("Singapore Launches A.I. Verify") |
| AI Verify launched 25 May 2022 by IMDA + PDPC as world's first AI governance testing framework and toolkit; MVP for international pilot and feedback | **Verified** | IMDA/PDPC launch releases; AI Verify Foundation "background" note |
| Framework assesses against 11 internationally recognised AI governance principles | **Verified** | AI Verify Foundation, "What is AI Verify" (primary list) |
| The exact 11-principle list (transparency … inclusive growth) | **Verified** | AI Verify Foundation "What is AI Verify" page |
| Toolkit tests: explainability, fairness, robustness (+ model performance) | **Verified** | IMDA AI Verify primer (PDF); Foundation toolkit page |
| Toolkit and framework open-sourced on GitHub 7 June 2023 (repo created 3 Jun 2023, Apache-2.0) | **Verified** | GitHub API (`aiverify-foundation/aiverify`) |
| AI Verify Foundation launched 7 June 2023 at ATxAI/ATxSG | **Verified** | IMDA press release; Foundation events page |
| Seven founding premier members: IMDA, Aicadium, IBM, Microsoft, Google, Red Hat, Salesforce | **Verified** | AI Verify Foundation launch page |
| Premier tier later includes AWS, Dell, Resaro (by 2024) | **Verified** (as of 2024 materials) | AI Verify Foundation homepage (2024/05 assets) |
| Project Moonshot launched 31 May 2024 at ATxSG in open beta; benchmarking + red teaming; Apache-2.0; repo created 14 Dec 2023 | **Verified** | IMDA press release (31 May 2024); US-ASEAN/PRNewswire coverage; GitHub API |
| Moonshot: 100+ benchmark datasets, Web UI + CLI + APIs, IMDA Starter Kit integration | **Verified** | Foundation Moonshot page; repo README |
| Framework updated 29 May 2025 to cover traditional + GenAI | **Verified** | AI Verify Foundation "What is AI Verify" |
| Crosswalks: NIST AI RMF (13 Oct 2023); NIST GenAI profile (29 May 2025); Hiroshima Process CoC (29 May 2025); ISO/IEC 42001 (21 Jul 2025) | **Verified** | AI Verify Foundation crosswalks page |
| Singapore's precise participation status in the Hiroshima AI Process | **Partially verified — flagged** | Crosswalk verified; endorsement records not checked to primary source |
| Post-launch international pilot cohort (which companies/countries, 2022–2023) | **Partially verified — flagged** | Consultation group named by Foundation; cohort lists in secondary coverage vary |
| Full current Foundation member roster and tier split (2026) | **Partially verified — flagged** | Live member page not captured; 2024 premier set verified |
| "MLTC" acronym for original test criteria | **Unverified — flagged** | Not found in primary sources (same flag as the umbrella guide) |
| AI Tester Accreditation Programme and Global AI Assurance Sandbox (2026) | **Partially verified — flagged** | Foundation site mentions; exact dates not captured |

---

## Primary Sources and Further Reading

- **IMDA press release — "Singapore Launches A.I. Verify" (May 2022)** — imda.gov.sg; joint release with MDDI and PDPC (mddi.gov.sg)
- **PDPC — "Launch of AI Verify – An AI Governance Testing Framework and Toolkit"** — pdpc.gov.sg
- **IMDA press release — "Singapore Launches AI Verify Foundation" (June 2023)** — imda.gov.sg; Microsoft Singapore newsroom coverage (7 June 2023)
- **AI Verify Foundation** — aiverifyfoundation.sg: "What is AI Verify", toolkit page, Project Moonshot page, crosswalks page, sample report
- **AI Verify Testing Framework (official instrument)** — go.gov.sg/aivtf-pdf (PDF) and go.gov.sg/aivtf-excel (process-checks tool)
- **AI Verify primer (IMDA)** — file.go.gov.sg/aiverify-primer.pdf
- **IMDA press release — "Singapore launches Project Moonshot" (31 May 2024)** — imda.gov.sg; PRNewswire/US-ASEAN coverage of the ATxSG launch
- **GitHub repositories** — github.com/aiverify-foundation/aiverify; github.com/aiverify-foundation/moonshot (incl. API metadata for creation dates)
- **IMDA — Starter Kit for Safety Testing of LLM-Based Applications** — imda.gov.sg (PDF)
- **Model AI Governance Framework (2019; 2nd ed. 2020) and Model AI Governance Framework for Generative AI (2024)** — IMDA/PDPC
- **Repo companions** — [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md), [ai_llm/ai_governance_bias_redteaming_guide.md](ai_llm/ai_governance_bias_redteaming_guide.md), [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md), [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md), [audit_as_code_guide.md](audit_as_code_guide.md), [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md), [../management/business_case_development_guide.md](../management/business_case_development_guide.md)

*This guide was authored by Jack Liu Shurui, Solution Architect at Cymbal Bank. All launch dates, principle names, and technical-test facts were verified against primary sources (IMDA/PDPC releases, the AI Verify Foundation, official GitHub repositories, IMDA's Project Moonshot release) in August 2026; items that could not be fully verified are flagged **[verify]** inline and in the verification log.*

*For questions, corrections, or contributions, please open an issue or PR at: https://github.com/jackliusr/research*
