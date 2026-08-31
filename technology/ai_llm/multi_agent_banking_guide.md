# Multi-Agent AI Systems for Banking — The Design, Evaluation, Deployment, and Governance Guide

**The governed agent — the multi-agent architecture for the four banking workflows (fraud triage, AML screening, compliance automation, KYC), the verified orchestration patterns, the agent frameworks, the banking guardrails, the evaluation pipelines and quality gates, the deployment lifecycle and hypercare, the production standards (SLOs, observability, audit trails), the MAS-supervision expectations, and the flagship worked example — the Cymbal Bank multi-agent deployment.**

> **Author:** Jack Liu Shurui, Solution Architect — Cymbal Bank, Singapore
> **Context:** Banking Domain / AI & LLM Engineering — the multi-agent architecture for the four regulated workflows (the fraud-triage agent, the AML-screening agent, the compliance-automation agent, the KYC agent), the orchestration patterns (orchestrator-workers, prompt chaining, the hierarchical supervisor), the agent frameworks (LangGraph, CrewAI, AutoGen, Semantic Kernel), the banking guardrails (the PDPA/GDPR data-privacy themes, the model-risk overlay, the KYC/EDD conventions, the AML-screening mechanics), the evaluation pipelines (golden datasets, LLM-as-judge, task-specific metrics), the quality gates, the explainability, the deployment lifecycle (canary, champion-challenger, kill switches), the hypercare, the production standards (SLOs, fallbacks, observability, audit trails), the MAS-supervision expectations, and the flagship worked example (the Cymbal Bank multi-agent deployment for fraud triage + AML screening + KYC)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass verified the following against primary or authoritative sources: the Anthropic engineering post "Building effective agents" (December 19, 2024 — the workflow taxonomy: prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer); the LangChain official documentation and the langchain-ai GitHub organisation (the LangGraph supervisor / hierarchical multi-agent architecture, the framework maintainer, the language support); the CrewAI official documentation and the crewAIInc GitHub repository (the maintainer, the MIT license, the Crews and Flows abstractions); the Microsoft AutoGen documentation, the Microsoft Research project page, and the microsoft/autogen repository (the maintainer, the MIT license, the Python language); the Microsoft Semantic Kernel documentation and the microsoft/semantic-kernel repository (the maintainer, the MIT license, the C#/Python/Java language support); the arXiv paper "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (arXiv:2306.05685, Zheng et al., NeurIPS 2023 Datasets and Benchmarks Track — the LLM-as-judge practice, its biases, and the ~80% human-agreement finding); and published MLOps engineering references on canary deployment for machine-learning models (the gradual rollout, the traffic splitting, the rollback triggers). Facts carried from this repository's sibling guides (verified in prior passes) are marked ✅ with a cross-reference — notably the MAS regime (Notice 658, Notice 626, TRMG, FEAT 2018, the Model AI Governance Framework) carried by the MAS guide, and the hypercare practice carried by the vendor-management guide. Facts that could not be confirmed from a primary source in this pass are flagged ⚠ and listed in "What Could Not Be Verified" — nothing unverified is presented as verified.
> **Last Updated:** August 2026
> **Companion guides (banking/, prefix `../banking/`):** [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) · [Fircosoft — the AML/Sanctions Screening Engine](../banking/fircosoft_guide.md) · [Private Banking](../banking/private_banking_guide.md) · [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) · [AI/GenAI Banking Compliance](../banking/ai_genai_banking_compliance_guide.md)
> **Companion guides (technology/ and management/, prefix `../`):** [AI Platform Engineering](../ai_platform_engineering_guide.md) · [Cybersecurity](../cybersecurity_guide.md) · [Vendor Management](../management/vendor_management_guide.md) (the hypercare discipline)
> **Companion guides (ai_llm/, same folder):** [Production-Ready LLM Agents](production_ready_llm_agents_guide.md) · [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) · [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) · [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) · [Implementing Responsible AI](implementing-responsible-ai.md) · [AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md) · [LLM Agents Failures in Production](llm_agents_failures_production_guide.md) · [Enterprise Agentic Platform Architecture](enterprise_agentic_platform_architecture_guide.md) · [AgentOps — LLM Observability](agentops_guide.md) · [Schema Evolution & Data Drift](schema_evolution_data_drift_guide.md)
> **Series:** AI/LLM Engineering Guides — the Banking & Governance track (the sibling track to the Agent & Platform track of the production-agent guides)
> **How to read this guide:** the four parts map to the four questions of the build — Part 1 (Sections 1–5) what the system is, Part 2 (Sections 6–8) how it proves itself, Part 3 (Sections 9–10) how it arrives, Part 4 (Section 11) how it behaves in production — and Section 12 is the whole thing in one deployment. The reader who wants the argument in one sitting reads Sections 1.3, 7.1, 9.2, 11.3, and 12.7; the reader who wants the verification record reads Sections 14 and 15.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Regulated-AI Context — Condensed](#11-the-regulated-ai-context--condensed)
   - 1.2 [The Four Banking Workflows](#12-the-four-banking-workflows)
   - 1.3 [The Governed-Agent Design Principle](#13-the-governed-agent-design-principle)
2. [The Agent Taxonomy — the Four Workflows](#2-the-agent-taxonomy--the-four-workflows)
   - 2.1 [The Fraud-Triage Agent](#21-the-fraud-triage-agent)
   - 2.2 [The AML-Screening Agent](#22-the-aml-screening-agent)
   - 2.3 [The Compliance-Automation Agent](#23-the-compliance-automation-agent)
   - 2.4 [The KYC Agent](#24-the-kyc-agent)
   - 2.5 [The Human-in-the-Loop Map](#25-the-human-in-the-loop-map)
3. [The Orchestration Patterns](#3-the-orchestration-patterns)
   - 3.1 [The Verified Pattern Vocabulary](#31-the-verified-pattern-vocabulary)
   - 3.2 [The Orchestrator-Workers Pattern](#32-the-orchestrator-workers-pattern)
   - 3.3 [The Pipeline (Prompt Chaining) Pattern](#33-the-pipeline-prompt-chaining-pattern)
   - 3.4 [The Hierarchical (Supervisor) Pattern](#34-the-hierarchical-supervisor-pattern)
   - 3.5 [The Routing and Parallelization Patterns](#35-the-routing-and-parallelization-patterns)
   - 3.6 [The Pattern Selection Matrix](#36-the-pattern-selection-matrix)
   - 3.7 [The Orchestration Anti-Patterns](#37-the-orchestration-anti-patterns)
4. [The Agent Frameworks](#4-the-agent-frameworks)
   - 4.1 [The Framework Comparison Table](#41-the-framework-comparison-table)
   - 4.2 [LangGraph](#42-langgraph)
   - 4.3 [CrewAI](#43-crewai)
   - 4.4 [AutoGen](#44-autogen)
   - 4.5 [Semantic Kernel](#45-semantic-kernel)
   - 4.6 [The Framework Selection for Regulated Banking](#46-the-framework-selection-for-regulated-banking)
5. [The Banking Guardrails](#5-the-banking-guardrails)
   - 5.1 [The Data-Privacy Guardrails — the PDPA/GDPR Themes](#51-the-data-privacy-guardrails--the-pdpa-gdpr-themes)
   - 5.2 [The Model-Risk Overlay](#52-the-model-risk-overlay)
   - 5.3 [The KYC/EDD Conventions](#53-the-kyc-edd-conventions)
   - 5.4 [The AML-Screening Mechanics](#54-the-aml-screening-mechanics)
   - 5.5 [The Guardrail Enforcement Points](#55-the-guardrail-enforcement-points)
6. [The Evaluation Pipelines](#6-the-evaluation-pipelines)
   - 6.1 [The Evaluation Framework](#61-the-evaluation-framework)
   - 6.2 [The Golden Datasets](#62-the-golden-datasets)
   - 6.3 [The LLM-as-Judge](#63-the-llm-as-judge)
   - 6.4 [The Task-Specific Metrics](#64-the-task-specific-metrics)
   - 6.5 [The Evaluation Tooling](#65-the-evaluation-tooling)
   - 6.6 [The Evaluation-Run Record](#66-the-evaluation-run-record)
7. [The Quality Gates](#7-the-quality-gates)
   - 7.1 [The Pre-Production Gates](#71-the-pre-production-gates)
   - 7.2 [The Regression Suites](#72-the-regression-suites)
   - 7.3 [The Drift Monitoring](#73-the-drift-monitoring)
8. [The Explainability](#8-the-explainability)
   - 8.1 [The Decision Rationale](#81-the-decision-rationale)
   - 8.2 [The XAI Techniques](#82-the-xai-techniques)
   - 8.3 [The Regulatory-Examination Fit](#83-the-regulatory-examination-fit)
9. [The Deployment Lifecycle](#9-the-deployment-lifecycle)
   - 9.1 [The Architecture-to-Production Path](#91-the-architecture-to-production-path)
   - 9.2 [The Phased Production Rollout](#92-the-phased-production-rollout)
   - 9.3 [The Rollout Gate Table](#93-the-rollout-gate-table)
   - 9.4 [The Deployment Topology](#94-the-deployment-topology)
10. [The Hypercare](#10-the-hypercare)
    - 10.1 [The Runbooks](#101-the-runbooks)
    - 10.2 [The Incident Response](#102-the-incident-response)
    - 10.3 [The Stabilization Metrics](#103-the-stabilization-metrics)
    - 10.4 [The Client-Team Knowledge Transfer](#104-the-client-team-knowledge-transfer)
11. [The Production Standards](#11-the-production-standards)
    - 11.1 [The Reliability — SLOs and Fallbacks](#111-the-reliability--slos-and-fallbacks)
    - 11.2 [The Observability](#112-the-observability)
    - 11.3 [The Audit Trails](#113-the-audit-trails)
    - 11.4 [The MAS-Supervision Expectations — Condensed](#114-the-mas-supervision-expectations--condensed)
    - 11.5 [The Error Budget and the Reliability Math](#115-the-error-budget-and-the-reliability-math)
12. [The Worked Example — the Cymbal Bank Multi-Agent Deployment](#12-the-worked-example--the-cymbal-bank-multi-agent-deployment)
    - 12.1 [The Scenario](#121-the-scenario)
    - 12.2 [The Architecture — the Orchestrator and the Three Agents](#122-the-architecture--the-orchestrator-and-the-three-agents)
    - 12.3 [The Evaluation Gates](#123-the-evaluation-gates)
    - 12.4 [The Phased Rollout](#124-the-phased-rollout)
    - 12.5 [The Hypercare](#125-the-hypercare)
    - 12.6 [The Audit Trail](#126-the-audit-trail)
    - 12.7 [The Lessons](#127-the-lessons)
    - 12.8 [The Governance Record](#128-the-governance-record)
13. [The Summary](#13-the-summary)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary](#16-the-glossary)
17. [The Closing — the Governed Agent](#17-the-closing--the-governed-agent)

---

## 1. The Overview

### 1.1 The Regulated-AI Context — Condensed

The Monetary Authority of Singapore (MAS) supervises the banks that this guide serves, and the MAS regime — carried in full by the sibling [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) guide — sets the frame within which every multi-agent system in this guide must be designed. The governing instruments, cross-referenced rather than re-derived here: ✅ **Notice 658** (the outsourcing requirements — the 2023 revision binds banks that outsource material functions, and an agent platform is squarely a material outsourced function), ✅ **Notice 626** (the technology risk management notice), ✅ the **Technology Risk Management Guidelines** (TRMG, 2013, updated 2021 — the technology risk management expectations), ✅ **FEAT** (the Fairness, Ethics, Accountability and Transparency principles, 2018 — the AI governance principles), and ✅ the **Model AI Governance Framework** (2019, updated 2020 — the model lifecycle guidance). The AI-compliance overlay — the FEAT application, the accountability structures, the procurement and vendor obligations — is carried by the sibling [AI/GenAI Banking Compliance](../banking/ai_genai_banking_compliance_guide.md) guide.

The through-line: a multi-agent banking system is not an experiment to be shipped; it is a **governed capability** — designed, evaluated, deployed, and operated inside a documented control framework, with the human accountable for every consequential decision. This guide is written for MAS-supervised banks and for equivalent regulated environments (the GDPR/PDPA privacy themes travel; the FEAT-style principles recur in most mature AI-governance regimes), with Cymbal Bank — the fictional Singapore-based bank persona used across this repository — as the only bank in the worked example.

### 1.2 The Four Banking Workflows

The guide covers the four workflows where multi-agent systems earn their keep in a bank:

| The Workflow | The Owner Function | The Core Question | The Consequence of Error |
|---|---|---|---|
| The fraud triage | The fraud operations unit | "Is this transaction or account behaviour suspicious, and how urgent is it?" | Financial loss; customer harm; regulatory reporting breaches |
| The AML screening | The financial-crime compliance unit | "Does this payment or counterparty match a sanctioned, watch-listed, or suspicious profile?" | Regulatory fines; reputational damage; MAS enforcement action |
| The compliance automation | The compliance operations unit | "Does this document, process, or decision satisfy the policy and regulatory requirements?" | Policy breaches; audit findings; supervisory censure |
| The KYC | The client onboarding and due-diligence unit | "Who is this customer, and what is the risk they bring?" | Money-laundering exposure; onboarding risk; EDD failures |

The four workflows share a shape: high-volume intake, structured data plus unstructured evidence, deterministic rule layers with probabilistic judgment on top, and a human decision-maker who must remain accountable. That shared shape is what makes a single multi-agent design language — the taxonomy, the orchestration, the evaluation, the deployment, the governance — worth writing down once.

The four workflows also share the *failure economics*: in every lane, the cost of the missed case — the fraud that passes, the match that is dismissed, the requirement that is waved through — far exceeds the cost of the extra review. That asymmetry is why the escalation contract of Section 2.5 biases toward the human review on the uncertain cases: the governed agent would rather over-escalate than under-report, and the evaluation metrics of Section 6.4 are set to make that bias explicit and measurable rather than implicit and accidental.

### 1.3 The Governed-Agent Design Principle

The governing principle of this guide is the **governed agent**: an agent whose autonomy is bounded by four contracts —

1. **The scope contract** — the agent's task boundary is explicit; anything outside it routes to a human (the Anthropic guidance is to prefer the simplest solution that works, and to add agentic complexity only when the task demands it ✅).
2. **The escalation contract** — every workflow defines, in advance, the conditions under which the agent's output is advisory only and the human's decision is required.
3. **The evidence contract** — every agent decision carries a rationale and a trace back to the inputs (the FEAT transparency principle, cross-ref the MAS guide).
4. **The evaluation contract** — no agent version reaches production without passing the quality gates of Part 2.

Every section that follows is that principle made concrete. The agent taxonomy (Section 2) defines who the agents are; the orchestration (Section 3) defines how they cooperate; the guardrails (Section 5) define what they may touch; the evaluation (Sections 6–8) defines how they prove themselves; the lifecycle (Sections 9–10) defines how they arrive; the standards (Section 11) define how they behave in production; the worked example (Section 12) shows the whole thing at Cymbal Bank.

---

## 2. The Agent Taxonomy — the Four Workflows

The four agents share the same anatomy: the mission (the bounded job), the inputs (the data the agent may read), the tools (the capabilities the agent may invoke), the outputs (the artefacts the agent produces), and the failure modes (the ways the agent can be wrong — each one measured, not assumed). The sections that follow set out the anatomy for each of the four, and Section 2.5 maps the human-in-the-loop that sits above them all.

### 2.1 The Fraud-Triage Agent

**The mission.** The fraud-triage agent is the first reader of every flagged transaction, login, card-present, or account-activity event. It triages: it classifies the event, estimates the urgency, assembles the evidence pack, and drafts the disposition recommendation for the fraud analyst. It does **not** block or release funds on its own authority — the block/release decision belongs to the human analyst (and to the deterministic fraud engine where the rule layer already owns that decision).

**The inputs.** The transaction stream; the device and session signals; the customer history; the fraud-case management queue; the reference data (merchant categories, counterparties, geo-IP context); the notes from prior analyst dispositions.

**The tools.** The case-management lookup; the transaction-graph walk; the customer-relationship view; the velocity counters; the document reader (for uploaded proof — receipts, police reports); the write-back tool that drafts the case note.

**The outputs.** The triage classification (benign / suspicious / confirmed-fraud candidate); the urgency score; the evidence pack (the why: the signals that fired, the rule hits, the lookalike cases); the recommended action (monitor / block / release / refer); and the draft case note for the analyst to approve.

**The failure modes.** The agent over-flagging (noise that buries the analyst queue) and under-flagging (missed fraud). Both are measured, not assumed: the precision/recall on the golden set is the gate, and the human override rate is the live signal.

### 2.2 The AML-Screening Agent

**The mission.** The AML-screening agent works the screening and surveillance pipeline: name and entity screening against sanctions and watch lists, transaction monitoring alert triage, and the drafting of the suspicious transaction report (STR) narrative. It is the *analysis* agent — the screening match decision and the STR filing decision remain with the compliance officer, and the screening mechanics themselves (the fuzzy-match engine, the list management) belong to the deterministic layer cross-referenced in the [Fircosoft guide](../banking/fircosoft_guide.md).

**The inputs.** The payment and account messages; the counterparty and beneficial-ownership data; the sanctions and watch-list hits; the transaction-monitoring alerts; the case history; the regulatory reporting templates.

**The tools.** The name-matching review interface; the entity-resolution lookup; the transaction-pattern explorer; the alert-context assembler; the narrative-drafting tool; the STR-template writer.

**The outputs.** The screening-hit assessment (true match / false positive / requires review, with the matching rationale); the alert triage disposition; the drafted STR narrative (with the suspicion rationale and the supporting evidence references); the case chronology.

**The failure modes.** The false-positive flood (the classic AML cost) and the false-negative miss (the regulatory nightmare). The agent's value is in compressing the false-positive workload while keeping recall at the level the compliance function requires — and every disposition is attributable to a named human.

### 2.3 The Compliance-Automation Agent

**The mission.** The compliance-automation agent handles the document-and-process compliance workload: policy attestations, regulatory filing preparation, control-evidence review, breach-report drafting, and the "does this conform?" reading of contracts, marketing materials, and client communications. It is the broadest of the four agents and the one where the judgment boundary is most often tested — the agent reads, summarises, checks, and drafts; the compliance officer approves.

**The inputs.** The policy library; the regulatory text (cross-ref the MAS guide for the instruments); the documents under review; the control-evidence repositories; the filing calendars and templates; the prior approval history.

**The tools.** The document reader and summariser; the policy-checker (the clause-by-clause conformance pass); the evidence collector; the filing-preparation helper; the breach-narrative drafter.

**The outputs.** The conformance assessments with the cited clauses; the drafted filings and attestations; the evidence packs for the control reviews; the exception flags that route to the compliance officer.

**The failure modes.** The confident-but-wrong reading of a policy clause (the hallucination risk is highest in this agent because the source documents are long and the answer is a judgment). The mitigation is the citation discipline: every assessment must quote the clause, and every draft must be reviewed before it leaves the bank.

### 2.4 The KYC Agent

**The mission.** The KYC agent supports client onboarding and periodic review: it assembles the customer due-diligence (CDD) file, extracts the identifying and beneficial-ownership data from the documents, performs the initial risk-rating pass, checks the documentation against the KYC requirements, and drafts the onboarding recommendation for the relationship manager and the compliance approver. The onboarding and enhanced due-diligence (EDD) conventions are cross-referenced from the [Private Banking guide](../banking/private_banking_guide.md).

**The inputs.** The application forms; the identification documents (passports, incorporation papers, board resolutions); the beneficial-ownership declarations; the source-of-wealth and source-of-funds evidence; the screening results; the risk-rating questionnaire answers.

**The tools.** The document-extraction reader; the entity-structure builder (the ownership chain); the screening-result consumer; the risk-rating calculator; the requirement-checklist checker; the recommendation drafter.

**The outputs.** The extracted and structured customer profile; the completed requirement checklist; the risk rating with the rationale; the recommended onboarding decision (accept / enhanced due diligence / decline) — always advisory, always approved by a human.

**The failure modes.** The extraction error on a document (wrong name, wrong date — the data-quality failure that poisons everything downstream) and the checklist false-pass (declaring a requirement satisfied when the evidence is weak). Both are gated on the golden set and sampled live by the QC function.

### 2.5 The Human-in-the-Loop Map

The human-in-the-loop (HITL) is not a single checkpoint — it is a *ladder* that differs per workflow. The table sets out where the human sits and what the agent may do alone:

| The Workflow | The Agent May Do Alone | The Human Decides | The Human's Tooling |
|---|---|---|---|
| The fraud triage | Classify, score urgency, assemble the evidence pack, draft the case note | The block/release decision; the customer contact; the police referral | The case queue; the disposition console; the override log |
| The AML screening | Assess the match, triage the alert, draft the STR narrative | The screening disposition; the STR filing; the list-escalation | The review queue; the STR approval workflow |
| The compliance automation | Read, summarise, check conformance, draft filings and attestations | The filing sign-off; the breach classification; the policy interpretation | The approval workflow; the evidence vault |
| The KYC | Extract, structure, rate, check the requirements, draft the recommendation | The onboarding accept/decline; the EDD scope; the risk-rating sign-off | The onboarding workflow; the QC sampling queue |

Two design rules govern the ladder. First, **the escalation contract is written before the agent is built**: the conditions that force a human review are enumerated in the design (the score thresholds, the confidence bands, the list matches, the new-entity cases) rather than improvised in production. Second, **the human override is instrumented**: every override is logged, and the override rate is a first-class evaluation metric — a persistently high override rate on a disposition type is a defect in the agent, not in the humans.

A third rule governs the human side of the ladder: **the queue is designed, not inherited.** The human reviewers are not an infinite resource absorbing the agent's overflow — the workload balance is a design input (the agent's automation rate is set so that the human queue stays inside the staffing plan), and the queue metrics (the backlog, the age of the oldest case, the reviewer utilisation) are monitored with the same rigour as the agent metrics. The governed agent optimises the human's time, not the human's disappearance.

---

## 3. The Orchestration Patterns

The orchestration is how the agents cooperate — and in a regulated bank, the orchestration is also an audit surface: the pattern chosen determines where the decision rationale lives, where the gates sit, and where the human enters. This section names the patterns from the verified vocabulary of Section 3.1, sets out the three patterns that anchor the banking designs (Sections 3.2–3.4), the supporting patterns (Section 3.5), the selection matrix (Section 3.6), and the anti-patterns to avoid (Section 3.7).

### 3.1 The Verified Pattern Vocabulary

The orchestration patterns in this guide use the names verified against primary sources in this pass — no pattern name is invented. The canonical taxonomy is Anthropic's "Building effective agents" engineering post (December 19, 2024) ✅, which distinguishes **workflows** (systems where the LLMs and tools are orchestrated through predefined code paths) from **agents** (systems where the LLM dynamically directs its own process and tool use), and names five workflow patterns: **prompt chaining**, **routing**, **parallelization**, **orchestrator-workers**, and **evaluator-optimizer** ✅. The LangGraph documentation and the langchain-ai organisation describe the **hierarchical (supervisor)** multi-agent architecture — "hierarchical systems are a type of multi-agent architecture where specialized agents are coordinated by a central supervisor agent" ✅. The three patterns that anchor the banking designs in this guide are the orchestrator-workers pattern, the pipeline (prompt chaining) pattern, and the hierarchical (supervisor) pattern; the routing and parallelization patterns appear as supporting patterns inside them.

| The Pattern | The Primary Source | The Banking Use |
|---|---|---|
| The orchestrator-workers | Anthropic, "Building effective agents" (2024) ✅ | The fraud-triage orchestration — the central agent decomposes the case and dispatches the specialist workers |
| The pipeline (prompt chaining) | Anthropic, "Building effective agents" (2024) — prompt chaining ✅ | The KYC document flow — extract, then structure, then rate, then check, each step gated |
| The hierarchical (supervisor) | The LangGraph documentation and the langchain-ai supervisor library ✅ | The cross-workflow command — the supervisor routes between the fraud, AML, and KYC agents |

### 3.2 The Orchestrator-Workers Pattern

In the orchestrator-workers workflow, per the Anthropic definition ✅, "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results." The orchestrator decides the subtasks at runtime — the pattern fits "complex tasks where you can't predict the subtasks needed" ✅ — which is exactly the fraud-triage shape: the orchestrator reads the incoming case, decides which workers to dispatch (the transaction-graph worker, the customer-history worker, the device-signal worker, the document reader), collects their findings, and synthesises the triage package for the analyst.

The banking discipline on the pattern: the orchestrator's *decomposition* may be dynamic, but the *worker catalogue* is fixed and governed. Every worker is a scoped agent with its own evaluation record; the orchestrator cannot invent new workers at runtime, and it cannot bypass the escalation contract. The orchestrator's synthesis is the evidence pack — the workers' findings plus the rationale — and the human decision sits after the synthesis, never inside it.

### 3.3 The Pipeline (Prompt Chaining) Pattern

The prompt chaining workflow, per the Anthropic definition ✅, "decomposes a task into a sequence of steps, where each LLM call processes the output of the previous one," with programmatic checks ("gates") on the intermediate steps — the pattern is "ideal for situations where the task can be easily and cleanly decomposed into fixed subtasks" ✅. The KYC flow is the canonical banking pipeline: the extraction step (the document reader) → the structuring step (the customer profile builder) → the rating step (the risk-rating pass) → the checking step (the requirement checklist) — with a programmatic gate after each step, and a fail-out to the human queue when a step's confidence is low.

The pipeline is the pattern of choice wherever the workflow is fixed and the regulator (or the auditor) expects to see the stages: each stage boundary is a natural checkpoint for the audit trail, and a failure at any stage is attributable to that stage's evaluation record. The AML-screening flow uses a pipeline inside the alert-triage worker for the same reason — the match assessment, the context assembly, and the narrative drafting are fixed stages with gates between them.

### 3.4 The Hierarchical (Supervisor) Pattern

The hierarchical pattern is the LangGraph-documented multi-agent architecture ✅: "specialized agents are coordinated by a central supervisor agent," with the supervisor "controlling all communication flow and task delegation, making decisions about which agent to invoke based on the current context and task requirements." The supervisor is the cross-workflow command layer: a single escalation from the fraud queue that touches the AML screening (is the counterparty sanctioned?) and the KYC profile (what is the customer's risk rating?) is routed by the supervisor to the specialist agents and their results are assembled for the human.

The banking discipline on the pattern: the hierarchy is *two levels deep and no deeper* — the supervisor over the specialist agents, never a chain of supervisors over supervisors. Depth is the enemy of the audit trail: every additional level multiplies the places where a decision rationale can be lost. The supervisor's routing decisions are themselves logged as decisions, with the same evidence discipline as the specialist agents.

### 3.5 The Routing and Parallelization Patterns

Two supporting patterns complete the vocabulary. **Routing** ✅ "classifies an input and directs it to a specialized follow-up task" — the banking use is the intake classifier that sends a case to the fraud, AML, or KYC lane (or to the human queue when the classification confidence is low). **Parallelization** ✅ runs independent subtasks concurrently, in two variations: **sectioning** (independent subtasks run in parallel — the KYC document set: the passport, the incorporation papers, and the ownership declaration extracted concurrently) and **voting** (the same task run multiple times for diverse outputs — used sparingly in banking, and only where the marginal cost is justified, because every vote is a paid LLM call and a governance object). The **evaluator-optimizer** pattern ✅ (one LLM generates while another evaluates in a loop) appears in the evaluation pipelines of Part 2 rather than in the production flows — in production, the evaluation loop is the human-and-gate structure, not an unpaid LLM critic.

### 3.6 The Pattern Selection Matrix

| The Selection Question | The Pattern | The Reason |
|---|---|---|
| Is the task a fixed sequence of well-understood stages? | The pipeline (prompt chaining) | The gates give the audit trail its stage boundaries |
| Is the task decomposition unpredictable per case? | The orchestrator-workers | The orchestrator adapts the subtasks to the case |
| Do multiple specialist workflows need one command layer? | The hierarchical (supervisor) | The supervisor routes and assembles across the lanes |
| Is the first job to sort intake into lanes? | The routing | The classifier sends each case to its lane |
| Are the subtasks independent and speed-critical? | The parallelization (sectioning) | The concurrent workers compress the latency |

The rule of thumb, from the Anthropic guidance ✅: prefer the simplest pattern that meets the requirement, and only add the orchestration complexity when the simpler form measurably fails the evaluation gates.

### 3.7 The Orchestration Anti-Patterns

The patterns are tools, and the tools have failure modes. The anti-patterns this guide warns against, drawn from the design rules of this section and the failure catalogue of the sibling [LLM Agents Failures in Production](llm_agents_failures_production_guide.md) guide:

- **The unbounded delegation.** The orchestrator dispatches workers without a governed worker catalogue — the workers multiply, the costs run, and the audit trail fragments. The fix is the fixed catalogue of Section 3.2: the decomposition is dynamic, the catalogue is not.
- **The supervisor chain.** The hierarchy deeper than two levels — the supervisor over the supervisors — multiplies the places where the decision rationale can be lost. The fix is the two-level rule of Section 3.4.
- **The gate-less pipeline.** The chained steps without the programmatic checks — the error from the first stage propagates silently through the rest. The fix is the gate discipline of Section 3.3: every stage boundary is a checkpoint with a fail-out to the human queue.
- **The orchestrator as the decision-maker.** The orchestrator synthesises and *decides* — the human is presented with a conclusion rather than an evidence pack. The fix is the escalation contract of Section 2.5: the human decision sits after the synthesis, never inside it.

The anti-patterns share one root: the orchestration complexity outrunning the governance. The Anthropic rule of thumb ✅ — start simple, and add the complexity only when the simpler form fails the evaluation — is the prevention.

---

## 4. The Agent Frameworks

### 4.1 The Framework Comparison Table

Four frameworks dominate the multi-agent conversation, and all four were verified against their primary documentation in this pass:

| The Framework | The Maintainer | The License | The Languages | The Core Abstractions |
|---|---|---|---|---|
| LangGraph | LangChain, Inc. (the langchain-ai organisation) ✅ | MIT ✅ | Python and JavaScript/TypeScript ✅ | The graph — nodes, edges, state, checkpoints; the supervisor for hierarchical systems ✅ |
| CrewAI | CrewAI Inc. (the crewAIInc organisation) ✅ | MIT ✅ | Python ✅ | The Crews (role-based collaborative agents) and the Flows (event-driven automations) ✅ |
| AutoGen | Microsoft (Microsoft Research) ✅ | MIT ✅ | Python ✅ | The conversational multi-agent framework — the agent, the chat, the group chat ✅ |
| Semantic Kernel | Microsoft ✅ | MIT ✅ | C#, Python, and Java ✅ | The kernel (the orchestrator), the plugins and functions, the memory, the agent abstractions ✅ |

Two honesty notes before the detail. First, the **adoption claims** — "the most popular", "the fastest growing", the star counts — are marketing-grade claims that this guide does not verify; where a claim of that kind appears, it is flagged ⚠ rather than asserted. Second, the framework landscape moves quickly: the facts above (maintainer, license, languages) are stable and verified, but the *feature* surface of each framework changes release to release, and a production decision should be re-validated against the current documentation at decision time.

### 4.2 LangGraph

LangGraph is the LangChain organisation's graph-based agent framework ✅ — the Python and JavaScript/TypeScript library in which the agent logic is expressed as a stateful graph of nodes and edges, with checkpoints for persistence and the built-in supervisor support for hierarchical multi-agent systems ✅. For the banking reader, the attractions are the *determinism* affordances — the graph structure makes the flow explicit, inspectable, and testable, which maps directly onto the audit-trail requirement — and the *state* model, which gives the decision rationale a natural home (the state carries the evidence through the flow). The costs are the framework's size and the LangChain ecosystem coupling: the platform team must decide whether the abstraction layer earns its keep, per the Anthropic caution that frameworks "often create extra layers of abstraction that can obscure the underlying prompts and responses, making them harder to debug" ✅.

### 4.3 CrewAI

CrewAI is the CrewAI Inc. open-source Python framework for "orchestrating role-playing, autonomous AI agents" ✅ — the role-based model in which agents with defined roles, goals, and backstories cooperate inside **Crews**, with the newer **Flows** abstraction for event-driven, precise control ✅. The role-based framing is legible to business stakeholders — "the KYC extraction specialist", "the risk-rating analyst" — which helps the client-team knowledge transfer of Section 10.4. The banking caution is the same one that applies to all the autonomy-flavoured frameworks: the role-play framing is a mental model, not a governance model — the governance still comes from the scope, escalation, evidence, and evaluation contracts of Section 1.3, not from the framework.

### 4.4 AutoGen

AutoGen is Microsoft's open-source "programming framework for agentic AI" ✅, originating from Microsoft Research with the explicit research goal of "facilitating cooperation among multiple agents to solve tasks" ✅. Its core abstraction is the *conversation*: the agents cooperate through structured multi-agent conversations, including the group-chat pattern where multiple agents converse under a manager. The banking reader should note the ecosystem flux: the framework has undergone major version transitions (the 0.4 architectural rewrite), and Microsoft has since announced the successor **Microsoft Agent Framework** direction — the AutoGen facts verified here are the stable ones (the maintainer, the license, the language), and the current-recommended Microsoft agent stack should be checked against the live documentation at decision time ⚠.

### 4.5 Semantic Kernel

Semantic Kernel is Microsoft's SDK for building AI agents and enterprise AI solutions, documented on Microsoft Learn ✅, with first-class support for C#, Python, and Java ✅ — the language story that matters to a bank with a .NET or Java core-banking estate. Its core abstractions are the **kernel** (the orchestrator that composes the pieces), the **plugins and functions** (the capability units, including the native functions that wrap the bank's own systems), the memory, and the agent abstractions ✅. For the bank whose platform team lives in C# or Java, Semantic Kernel is the framework that keeps the agent work inside the existing engineering language rather than introducing a Python island — a real consideration in a regulated environment where the platform ownership is a governance question in itself.

### 4.6 The Framework Selection for Regulated Banking

The selection criteria for a MAS-supervised bank, in order:

1. **The audit-trail fit.** The framework must expose the flow, the state, and the tool calls to the observability layer — the graph structure of LangGraph and the kernel structure of Semantic Kernel make the flow inspectable by construction; the conversational models require explicit instrumentation of every message.
2. **The evaluation fit.** The framework must let the evaluation harness (Part 2) drive the agents deterministically — replay the golden set, capture the outputs, compare the versions.
3. **The platform fit.** The language and the deployment model must match the bank's platform estate (cross-ref the [AI Platform Engineering](../ai_platform_engineering_guide.md) guide) — the Semantic Kernel language story versus the Python-ecosystem story is a platform decision, not a fashion decision.
4. **The governance fit.** The framework's autonomy defaults must be overridable — the escalation contract (Section 2.5) must be enforceable in code, and the framework must not silently retry, loop, or spend without the budget and circuit-breaker controls of Section 11.

The honest summary: the framework is a *minor* determinant of the outcome compared with the contracts of Section 1.3, the evaluation of Part 2, and the operations of Part 4. The sibling guides — [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md), [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md), and the [LLM Frameworks Comparison](llm_frameworks_comparison_guide.md) — carry the deeper framework-by-framework analysis; this guide cross-references them rather than repeating them.

---

## 5. The Banking Guardrails

### 5.1 The Data-Privacy Guardrails — the PDPA/GDPR Themes

The multi-agent system is a data-processing machine, and the privacy themes apply at every stage: the intake (what personal data may the agents read?), the processing (what may the agents retain in prompts, context, and memory?), the storage (what lands in the vector store, the cache, and the logs?), and the export (what crosses the border to a model provider?). The Singapore Personal Data Protection Act (PDPA) and the EU General Data Protection Regulation (GDPR) themes — the purpose limitation, the data minimisation, the consent and legitimate-basis questions, the cross-border transfer rules, the data-subject rights (access, correction, erasure) — are carried by the sibling [AI/GenAI Banking Compliance](../banking/ai_genai_banking_compliance_guide.md) guide and the MAS guide; the guardrails here are the engineering translations:

- **The minimisation guardrail.** The agents receive the *minimum fields* the task requires — the KYC agent sees the documents for the case, never the full customer file; the fraud-triage agent sees the transaction and the history window, not the marketing profile.
- **The redaction guardrail.** The deterministic redaction layer strips the excess identifiers before the prompt is built, and the redaction rules are versioned and tested like any other code.
- **The residency guardrail.** The model endpoints and the data stores sit in the approved jurisdictions, per the MAS outsourcing and technology-risk expectations (Notice 658 and Notice 626, cross-ref the MAS guide) and the bank's own data-residency policy — the data-flow map is an artefact of the design, not an afterthought.
- **The retention guardrail.** The prompt, cache, and log retention follows the data-retention schedule; the observability layer of Section 11.2 must not become an accidental personal-data store.

### 5.2 The Model-Risk Overlay

Every agent embeds one or more models, and the models are risk objects in their own right. The model-risk overlay comes from the sibling [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) guide: the model inventory (every model inside every agent is registered, owned, and rated), the model validation (the independent validation of the model's fitness before production), the model-change control (a prompt change, a model swap, or a tool change is a model change and goes through the change control), and the performance monitoring (the live performance against the agreed metrics). The multi-agent twist: the *system* risk is greater than the sum of the *model* risks, because the orchestration composes the failures — the orchestrator's routing error can combine two individually-fine models into a bad outcome. The model-risk overlay therefore extends to the orchestration logic itself, which is treated as a model component with its own evaluation record.

### 5.3 The KYC/EDD Conventions

The KYC agent operates inside the onboarding and enhanced due-diligence conventions carried by the sibling [Private Banking guide](../banking/private_banking_guide.md): the identity verification standards, the beneficial-ownership look-through, the source-of-wealth and source-of-funds evidence expectations, the risk-rating methodology, and the EDD triggers (the politically-exposed persons, the high-risk jurisdictions, the unusual structures). The conventions bind the agent in three ways: the **requirement checklist** the agent enforces is the convention checklist, not an invented one; the **risk-rating rationale** must cite the convention's factors; and the **EDD escalation** — any case that trips an EDD trigger goes to the human, never past them.

### 5.4 The AML-Screening Mechanics

The AML-screening agent sits on top of the deterministic screening mechanics carried by the sibling [Fircosoft guide](../banking/fircosoft_guide.md): the name-matching algorithms, the list management (the sanctions lists, the watch lists, the internal risk lists), the match-scoring, and the case workflow. The agent's boundary is explicit: it *assesses and explains* the screening hits — it does not change the match logic, the lists, or the scoring, which remain owned by the deterministic engine and its vendor governance. The agent reads the hit, assembles the context, assesses the likelihood of a true match, and drafts the disposition rationale for the compliance officer; the officer's disposition, not the agent's, is the record that matters to the examiner.

### 5.5 The Guardrail Enforcement Points

The guardrails are enforced at four points, and all four are instrumented:

| The Enforcement Point | The Guardrail | The Evidence |
|---|---|---|
| The intake | The minimisation and redaction rules | The redaction audit record per case |
| The prompt boundary | The scope contract — the tools and data the agent may reach | The tool-call log |
| The output boundary | The output filters — the PII, the toxicity, the prompt-injection checks (cross-ref the [Cybersecurity](../cybersecurity_guide.md) and [Prompt Injection](prompt_injection_guide.md) guides) | The filter verdicts in the decision log |
| The decision point | The escalation contract — the human approval where the contract requires it | The approval records in the audit trail |

The guardrails are not a layer to be added at the end; they are the four contracts of Section 1.3 made operational, and the evaluation pipelines of Part 2 test the guardrails as hard as they test the task performance.

---

## 6. The Evaluation Pipelines

### 6.1 The Evaluation Framework

The evaluation framework rests on three verified practice pillars, each answering a different question:

| The Pillar | The Question It Answers | The Verification Status |
|---|---|---|
| The golden datasets | "Does the agent produce the *right* answer on cases we already know?" | ✅ practice carried by this repository's evaluation guides (cross-ref [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) and [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md)) |
| The LLM-as-judge | "Does the agent produce a *good* answer where no single right answer exists?" | ✅ verified against the primary paper (arXiv:2306.05685) |
| The task-specific metrics | "Does the agent meet the *operational* bar — the recall, the latency, the cost, the override rate?" | ✅ practice carried by the sibling production guides |

The three pillars compose: the golden set gives the deterministic floor (the right answer), the judge gives the qualitative ceiling (the good answer), and the task metrics give the operational contract (the answer that the bank can live with). A version that passes the golden set but fails the task metrics is not shippable; a version that scores well with the judge but fails the golden set is dangerous — the golden set is the non-negotiable floor for the four regulated workflows.

### 6.2 The Golden Datasets

The golden dataset is the curated, labelled set of cases with known-correct outcomes that every agent version must replay before it ships — the practice is the LLM-side cousin of the regression test suite, and the sibling [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md) guide carries the retrieval-side practice ✅. The banking discipline on the golden set:

- **The provenance.** Every golden case carries its source (the real past case, the synthetic construct, the regulatory example), its labels (the correct disposition, the correct extraction, the correct rating), and its approval (the compliance function signs off the labels — the golden set is a compliance artefact, not an engineering artefact).
- **The stratification.** The golden set is stratified by the risk segments the system must not fail: the EDD cases, the sanctioned-name variants, the confirmed-fraud cases, the edge cases (the name collisions, the poor-quality documents, the adversarial inputs). The per-stratum scores are reported, not just the aggregate — an aggregate pass can hide a stratum failure.
- **The currency.** The golden set is versioned and grown: the confirmed cases from production (the fraud that the agent flagged, the STR that was filed) are promoted into the golden set on a cadence, and the golden set itself is reviewed for label drift — the labels must reflect the current regulatory reading, not the past one.
- **The hygiene.** The golden cases are excluded from any fine-tuning or few-shot material, or the evaluation is contaminated; the sibling [LLM Evaluation vs Validation](llm_evaluation_vs_validation_guide.md) guide carries the distinction.

### 6.3 The LLM-as-Judge

The LLM-as-judge practice — using a strong LLM to evaluate the outputs of other LLMs — was established by the paper "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (Zheng et al., arXiv:2306.05685, NeurIPS 2023 Datasets and Benchmarks Track) ✅. The verified findings that matter to a bank:

- The strong LLM judges can match human preferences well — the paper reports over **80% agreement** with human preferences, the same level as the agreement between humans ✅.
- The judges carry **documented biases** — the position bias, the verbosity bias, the self-enhancement bias, and the limited-reasoning failure — and the paper proposes mitigations ✅.

The banking translation: the judge is a *scoring instrument with known error modes*, not an oracle. The mitigation discipline follows the paper's findings — the judge's scoring rubric is explicit (the bank's own criteria: the completeness of the evidence pack, the accuracy of the cited clause, the appropriateness of the escalation), the judge outputs are themselves sampled for human review (the judge's verdicts are spot-checked by the compliance QC), and the judge is never the sole gate on the four regulated workflows — the golden set and the human review sit above it. The judge is also a model, and therefore a model-risk object: the judge version is pinned, its agreement with the human labels is tracked, and a judge upgrade is a model change (Section 5.2).

### 6.4 The Task-Specific Metrics

The task-specific metrics convert the workflow's operational contract into numbers. The table sets the canonical metric sets per workflow:

| The Workflow | The Core Metrics | The Operational Bar |
|---|---|---|
| The fraud triage | The precision and recall of the triage classification; the urgency-score calibration; the analyst override rate; the time-to-evidence-pack | The recall floor on the confirmed-fraud stratum; the override rate ceiling |
| The AML screening | The true-match recall; the false-positive rate; the STR-narrative acceptance rate; the case-completion time | The recall floor set by the compliance function; the false-positive budget |
| The compliance automation | The clause-citation accuracy; the conformance-assessment agreement with the compliance reviewers; the draft acceptance rate | The citation-accuracy floor; the reviewer-agreement target |
| The KYC | The extraction field-accuracy; the checklist false-pass rate; the risk-rating agreement; the EDD-escalation recall | The field-accuracy floor; the zero-tolerance false-pass target |

The cross-cutting metrics complete the contract: the latency (the end-to-end case time against the workflow SLA), the cost (the token and tool-call cost per case — cross-ref the [FinOps](../finops_guide.md) guide), the tool-call correctness (the fraction of tool calls that were the right tool with the right arguments), and the guardrail metrics (the redaction failures, the PII leaks, the prompt-injection successes — all must be zero).

### 6.5 The Evaluation Tooling

The evaluation tooling — the harness that runs the golden set, drives the judge, and reports the metrics — is the engineering backbone of the pipelines. The sibling guides carry the tool landscape: [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) and the RAG-side [Evaluation Tools Comparison](rag/rag_evaluation_tools_comparison_guide.md); the commercial evaluation platforms (LangSmith from the LangChain organisation, MLflow from the Linux Foundation, and the rest) are covered there and in the [LLM Frameworks Comparison](llm_frameworks_comparison_guide.md) guide — this guide cross-references rather than re-derives. The banking requirements on the tooling are the same regardless of the product: the harness must be **deterministic** (the same golden case, the same version, the same score — the seed, the temperature, and the model version are pinned), **auditable** (the eval runs are logged as records: the version under test, the dataset, the scores, the judge verdicts), and **integrated** with the CI/CD pipeline of Section 9.1 — the eval gate runs in the pipeline, not in a notebook.

### 6.6 The Evaluation-Run Record

Every evaluation run is itself a governed record — the audit trail of the evaluation, without which the gate results are assertions rather than evidence. The evaluation-run record captures:

- **The system under test** — the agent version, the prompt versions, the model versions, the framework versions, and the configuration hash.
- **The dataset** — the golden-set version, the strata included, and the label version.
- **The run parameters** — the seeds, the temperatures, the judge version, and the harness version — the determinism contract of Section 6.5.
- **The results** — the per-stratum scores, the judge verdicts, the metric values, the guardrail results, and the deltas against the baseline version.
- **The gate verdicts** — which of the five quality gates passed and which failed, with the failing evidence attached.
- **The human sign-offs** — the compliance and the model-risk approvals for the release, named and timestamped.

The evaluation-run records are retained with the audit trail (Section 11.3) — the examiner's "what was tested" question is answered by the run-record archive, and the release history *is* the run-record history.

---

## 7. The Quality Gates

### 7.1 The Pre-Production Gates

The quality gates are the enforceable checkpoints between the agent versions and the production traffic. The gate sequence, in order:

1. **The golden-set gate.** The version replays the golden set; the per-stratum scores must meet the floors (Section 6.2). A stratum failure blocks the version — no override, no "we'll watch it live".
2. **The judge gate.** The judge scores the version's outputs on the bank's rubric against the baseline version; the score must meet the target or the change must be justified with evidence (Section 6.3).
3. **The task-metric gate.** The load-tested latency, cost, and tool-call correctness must meet the operational bars (Section 6.4) — an agent that is accurate but too slow or too expensive for the workflow SLA does not ship.
4. **The guardrail gate.** The redaction, PII, injection, and scope-violation tests must pass with zero failures — the guardrail metrics are pass/fail, not graded (Section 5.5).
5. **The human gate.** The compliance and model-risk sign-offs: the model change record (Section 5.2), the golden-set label sign-off, and the named accountable owner for the release. The human gate is the FEAT accountability principle made operational (cross-ref the MAS guide).

### 7.2 The Regression Suites

The regression suite is the golden set plus the failure catalogue, run continuously: every prompt change, model swap, tool change, or framework upgrade replays the full suite, and the score deltas against the previous version are the release decision's primary evidence. The failure catalogue — the sibling [LLM Agents Failures in Production](llm_agents_failures_production_guide.md) guide carries the catalogue of how agents fail — feeds the regression suite directly: every production incident and every near-miss case is added to the suite so that the failure cannot silently return. The regression suite has a cadence: the scheduled run (nightly, on the pinned evaluation environment) and the triggered run (on every pull request that touches the agent code, prompts, or configuration). The sibling [Schema Evolution & Data Drift](schema_evolution_data_drift_guide.md) guide carries the data-side regression practice.

### 7.3 The Drift Monitoring

The evaluation does not stop at the release: the drift monitoring watches the live system for the slow decays that the gates cannot see. The monitored drift dimensions:

- **The input drift.** The production case mix moves away from the golden-set distribution (the new fraud pattern, the new customer segment) — the input-distribution statistics are tracked and the golden set is grown to cover the new shape.
- **The output drift.** The agent's disposition distribution shifts (the approval rate climbs, the escalation rate falls) — the shift is a signal to investigate, benign or not.
- **The model drift.** The underlying model's behaviour drifts as the vendor updates it — the pinned-version tracking and the shadow re-scoring of the live cases against the golden labels catch it.
- **The override drift.** The human override rate moves away from the baseline — the leading indicator that the agent's judgment and the humans' judgment have diverged (Section 2.5).

The drift monitoring is an operations capability, and it runs on the observability substrate of Section 11.2 — the sibling [AgentOps — LLM Observability](agentops_guide.md) guide carries the tooling. The escalation path is defined in advance: a drift alert triggers the investigation runbook (Section 10.1), and a drift beyond the agreed threshold triggers the kill-switch question (Section 9.2).

---

## 8. The Explainability

### 8.1 The Decision Rationale

The FEAT transparency principle (cross-ref the MAS guide) and the Model AI Governance Framework both demand that the decisions affecting customers be explainable, and the audit trail of Section 11.3 demands that the decisions be reconstructable. The multi-agent design answers both with the **decision rationale**: every consequential output carries, as a first-class artefact, the reasoning trail —

1. **The inputs cited** — the specific data fields, documents, and prior cases the agent used (the citation discipline of the compliance-automation agent, Section 2.3, is the model for all four).
2. **The reasoning steps** — the orchestration path the case took (the workers dispatched, the stages passed, the gates triggered), recovered from the trace of Section 11.2.
3. **The alternatives considered** — the near-miss dispositions and the reasons they were rejected, where the workflow demands it (the AML match assessment: why the near-match was judged a false positive).
4. **The confidence** — the agent's own confidence signal, calibrated against the golden set, so the human reader knows when to distrust the output.
5. **The human overlay** — the analyst's disposition and the override rationale, completing the record.

### 8.2 The XAI Techniques

The explainability techniques layer underneath the rationale: the deterministic attribution techniques (the feature-attribution and rule-explanation methods for the tabular and rule components — the fraud score's drivers, the risk-rating factors), the model-agnostic explanation methods for the model components, and the LLM-native techniques (the citation and quotation requirements, the structured-rationale outputs, the self-consistency checks — the same answer under perturbation). The sibling guides carry the governance and technique detail: [AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md), [Implementing Responsible AI](implementing-responsible-ai.md), and the [Responsible AI Frameworks](../responsible_ai_frameworks_guide.md) guide. The banking discipline: the explanation is *for the accountable human first* — the analyst who must sign the disposition — and the examiner second; an explanation technique that produces technically correct but operationally useless output (the analyst cannot act on it) fails the FEAT purpose.

### 8.3 The Regulatory-Examination Fit

The explainability stack exists to survive the examination. What a MAS examination of a multi-agent deployment would demand, cross-referenced from the MAS guide rather than re-derived:

- **The inventory and the accountability.** The examiner expects to find the agents inventoried, with the named owners and the documented accountability structure — the FEAT accountability principle and the IAC guidelines (cross-ref the MAS guide).
- **The evidence of evaluation.** The examiner expects the evaluation record: the golden sets, the gate results, the drift monitoring, the version history — the evidence that the system was tested before it touched customers and continues to be tested.
- **The decision reconstruction.** The examiner expects to replay a decision: take any consequential case, and reconstruct what the system saw, what it did, and who decided — the audit trail of Section 11.3 is the deliverable.
- **The incident record.** The examiner expects the incidents, the investigations, and the remediations — the incident response of Section 10.2 is a supervisory artefact as much as an operational one.
- **The human-accountability demonstration.** The examiner expects the human to be genuinely in the loop — the escalation contract, the override instrumentation, and the demonstrated cases where the human overrode the agent and the record shows it.

The examination fit is not a separate activity: it is the whole design made examinable, and the worked example of Section 12 shows the fit concretely.

---

## 9. The Deployment Lifecycle

### 9.1 The Architecture-to-Production Path

The path from the architecture to the production traffic runs through four phases — the design, the build, the test, and the rollout — and the platform practices (the CI/CD pipeline, the environment strategy, the infrastructure-as-code, the deployment automation, the release management) are carried by the sibling [AI Platform Engineering](../ai_platform_engineering_guide.md) guide, cross-referenced here rather than re-derived:

| The Phase | The Deliverable | The Gate |
|---|---|---|
| The design | The architecture decision record (the agents, the orchestration pattern, the frameworks, the guardrails, the evaluation plan) — cross-ref the [Architecture Decision Record](../architecture_decision_record_guide.md) guide | The design review with the compliance and model-risk functions |
| The build | The agent code, the prompts, the tool integrations, the evaluation harness, the instrumentation | The code review and the static checks |
| The test | The golden-set results, the judge scores, the load tests, the guardrail tests | The quality gates of Section 7.1 |
| The rollout | The phased production deployment, the hypercare, the stabilization | The rollout gates of Section 9.3 |

Two lifecycle rules for the regulated environment. First, **the environment discipline**: the development, the test, the staging, and the production environments are separated, and the production data never appears in the non-production environments without the data-governance approval — the golden sets and the synthetic data serve the test environments. Second, **the change discipline**: every production change — the code, the prompts, the model versions, the tool configurations, the framework upgrades — travels the same pipeline with the same gates; there is no "quick fix to the prompt in production", because in a regulated bank the prompt is a governed artefact (Section 5.2).

### 9.2 The Phased Production Rollout

The rollout is phased by design, and the phase names are the established practice names, verified where the sources allowed:

- **The canary deployment** ✅ — the new version serves a small, controlled subset of the traffic (or of the users, or of the case types) first, with the performance and the stability monitored before the traffic share is increased — the verified practice of the gradual rollout with the traffic splitting and the rollback triggers (the MLOps engineering references). For the banking agents the canary is a *case-type* canary first: the new fraud-triage version starts on the low-risk, high-volume case stratum, never on the EDD stratum.
- **The champion-challenger** ⚠ — the comparison of the deployed model (the champion) against the candidate (the challenger) on the live traffic, with the challenger's performance measured before the promotion. The practice is well established in the model-management literature (the champion-challenger framing is a standard model-governance concept), but a primary citation could not be confirmed in this pass — the practice is described here as used, and flagged honestly.
- **The kill switch** ⚠ — the pre-built, tested mechanism to take an agent (or the whole system) out of the production path immediately: the feature flag or the circuit breaker that stops the traffic, plus the fallback path (Section 11.1) that the traffic falls onto. The term is in common use across the deployment and the AI-safety literature, but no single primary source was confirmed in this pass — the mechanism itself is mandatory regardless of the name's provenance.

The rollout sequence that combines them: **the shadow phase** (the new version scores the live cases without affecting the outcomes — the shadow scores are compared against the champion's), then **the canary** (the small live share with the rollback triggers armed), then **the champion-challenger measurement** (the live comparison on the shared traffic), then **the graduated expansion** (the share increases through the agreed steps, each step gated), with the **kill switch** armed at every step and tested in the rehearsal before the first canary.

### 9.3 The Rollout Gate Table

| The Gate | The Criteria to Pass | The Fail Action |
|---|---|---|
| The shadow gate | The shadow scores match the champion within the tolerance on the golden-set metrics; the latency and the cost within the bars | The version returns to the test phase |
| The canary gate | The canary share stable for the agreed period; the error rate, the latency, and the override rate within the bounds; no guardrail failures | The kill switch pulls the canary; the incident runbook starts |
| The champion-challenger gate | The challenger meets or beats the champion on the agreed metrics without the regression on any stratum | The champion stays; the challenger is reworked |
| The expansion gate | Each traffic-share step holds for the agreed period within the bounds | The share freezes or rolls back |
| The full-production gate | The full share stable for the agreed period; the drift monitoring and the observability baselines established | The share rolls back to the last stable level |

The gate evidence is recorded as part of the audit trail (Section 11.3) — the examiner's question "how did this version earn the production traffic?" is answered by the rollout record.

### 9.4 The Deployment Topology

The deployment topology is the physical shape of the phased rollout — the environments and the routing layers that make the phases possible:

- **The environments.** The development, the test, the staging, and the production environments, separated per the environment discipline of Section 9.1, with the staging environment replaying the full golden set before every release.
- **The routing layer.** The traffic router that implements the shadow, the canary, and the expansion — the case-type routing rules, the traffic-share percentages, and the rollback triggers live here, and they are *configuration, not code*, so the rollout control is an operational action rather than a software release.
- **The feature-flag and circuit-breaker layer.** The kill switch (Section 9.2) is implemented as the combination of the feature flags (the per-agent on/off) and the circuit breakers (the automatic tripping on the error-rate and the latency bounds) — and the layer is exercised in the drills before the first canary.
- **The fallback routing.** The deterministic fallback path (Section 11.1) is a first-class route in the router: when the agent route is off, the traffic falls to the rule engines and the manual queues without an engineering intervention.

The topology is drawn once, reviewed at the design gate, and versioned with the system — the deployment diagram is an artefact of the audit trail, because the examiner will ask where the traffic went on the incident day.

---

## 10. The Hypercare

### 10.1 The Runbooks

Hypercare is the post-go-live support period — the discipline is carried by the sibling [Vendor Management](../management/vendor_management_guide.md) guide ✅ ("the post-go-live support period... heightened monitoring, vendor staff on standby, a daily defect/issue triage, and a defined exit from hypercare once stability criteria are met"), and the banking agent deployment follows the same shape. The runbooks are written *before* the go-live, and they cover the scenarios the agents make possible:

- **The agent-quality runbook.** The golden-set score drops on the live re-scoring, or the judge agreement drops, or the override rate climbs past the threshold — the investigation steps, the evidence to gather, the decision point for the version rollback.
- **The drift runbook.** The input, output, model, or override drift alert fires (Section 7.3) — the triage, the root-cause analysis, the golden-set growth decision.
- **The outage runbook.** The model endpoint fails, the tool integration fails, the orchestrator fails — the fallback path activation (Section 11.1), the customer-impact assessment, the regulatory-reporting check (the incident reporting expectations, cross-ref the MAS guide).
- **The security runbook.** The prompt-injection attempt, the data-leak suspicion, the anomalous tool usage — the containment, the forensic capture, the reporting (cross-ref the [Cybersecurity](../cybersecurity_guide.md) guide).
- **The kill-switch runbook.** The conditions that justify the kill switch, the person authorised to pull it, the exact steps, and the restart procedure.

### 10.2 The Incident Response

The incident response follows the bank's established severity model, with the agent-specific additions: the **decision freeze** (when the agent is implicated in an incident, its production traffic stops until the investigation completes — the kill switch is the enforcement), the **evidence preservation** (the traces, the decision logs, and the eval records are preserved for the investigation — they are audit artefacts, not just logs), and the **remediation loop** (the incident feeds the failure catalogue and the regression suite of Section 7.2, so the class of failure is tested for, not just the instance). The incident record is a supervisory artefact (Section 8.3): the timeline, the impact, the root cause, the remediation, and the prevention evidence are written for the examiner as much as for the operations team.

### 10.3 The Stabilization Metrics

Hypercare has a defined exit, and the exit criteria are the stabilization metrics agreed before the go-live (the vendor-management discipline ✅: the exit once the stability criteria are met — "X consecutive days without a P1/P2" — rather than hypercare as a permanent state). For the agent deployment, the stabilization metrics include:

- **The incident count.** Zero P1 incidents and a sustained low P2 count over the agreed window (the bank's standard: the consecutive-clean-days criterion).
- **The quality metrics.** The live re-scored golden-set performance, the override rate, and the drift signals all within the agreed bounds.
- **The operational metrics.** The latency, the error rate, the cost per case, and the tool-call success within the bars, sustained.
- **The queue metrics.** The human queues are stable — the agent is not silently shifting work onto the humans (the escalation-rate and the case-backlog signals).

The hypercare exit is a formal gate with the sign-off from the operations, the compliance, and the model-risk functions — the same accountability discipline as every other gate in this guide.

### 10.4 The Client-Team Knowledge Transfer

The knowledge transfer turns the deployment from the engineering team's project into the client team's capability — the enablement, the documentation, and the training:

- **The enablement.** The client team (the fraud analysts, the compliance officers, the KYC reviewers) learns the agents as *tools with boundaries*: what each agent does, what it may do alone, what always comes to the human, and how to read the decision rationale. The role-based framing of the frameworks (Section 4.3) helps here — the analysts adopt the agents as specialist colleagues with defined jobs.
- **The documentation.** The operating documentation — the runbooks, the escalation contract, the metric definitions, the known-limitations list — lives beside the system, versioned with it, and is written for the operator, not the engineer.
- **The training.** The initial training (the classroom and the supervised live sessions), the recurring training (the new versions and the new failure modes), and the exercised scenarios (the runbook drills, including a scheduled kill-switch drill) — the training records are part of the governance record, because the examiner will ask how the humans were prepared to exercise their accountability.

The transfer completes when the client team can run the daily operation, recognise the abnormal, and execute the runbooks — and when the engineering team's support has a defined exit, aligned with the hypercare exit (Section 10.3).

---

## 11. The Production Standards

### 11.1 The Reliability — SLOs and Fallbacks

The production agent is governed by the service-level objectives (SLOs) — the agreed, measured targets for the availability, the latency, the accuracy, and the cost — and by the fallback architecture that keeps the bank running when the agent cannot. The SLO structure for the four workflows:

| The SLO | The Target Shape | The Enforcement |
|---|---|---|
| The availability | The agent service available within the agreed error budget (the nine-based or the absolute target the bank sets) | The alerting and the error-budget tracking on the observability substrate |
| The latency | The end-to-end case time within the workflow SLA (the triage answer, the screening assessment) | The percentile tracking; the queue-time monitoring |
| The accuracy | The golden-set and the live-re-scored performance within the agreed floors | The drift monitoring of Section 7.3 |
| The cost | The cost per case within the budget (the FinOps discipline, cross-ref the [FinOps](../finops_guide.md) guide) | The spend alerts and the budget caps |

The fallback architecture is the agent's shadow: **the deterministic fallback** (the rule layer and the existing engines — the fraud rules, the screening engine of the [Fircosoft guide](../banking/fircosoft_guide.md) — take the traffic when the agent is unavailable or under suspicion), **the degraded-mode fallback** (the agent runs but the automation is reduced — the outputs are advisory-only, everything escalates to the humans), and **the manual fallback** (the pre-agreed manual process, runbooks and all). The fallback paths are tested in the rehearsal, not discovered in the incident — the kill-switch drill (Section 10.4) is the fallback's dress rehearsal.

### 11.2 The Observability

The observability substrate — the tracing, the metrics, the logging, and the LLM-specific observability — is what makes the governed agent visible. The platform themes are carried by the sibling guides ([AI Platform Engineering](../ai_platform_engineering_guide.md), [Enterprise Agentic Platform Architecture](enterprise_agentic_platform_architecture_guide.md), and [AgentOps — LLM Observability](agentops_guide.md)); the agent-specific requirements:

- **The tracing.** The end-to-end trace per case — the intake event, the orchestration path, the worker calls, the tool calls, the LLM calls, the gate verdicts, the human decision. The trace *is* the decision rationale's raw material (Section 8.1), so the trace completeness is a quality requirement, not a nicety.
- **The metrics.** The golden metrics (the latency, the error rate, the cost per case, the tool-call success), the quality metrics (the override rate, the escalation rate, the disposition distribution), and the guardrail metrics (the filter verdicts — all of Section 6.4).
- **The LLM observability.** The per-call records: the model, the version, the prompt hash, the token counts, the latency, the cost, the output hash, the judge verdicts where applied — the LLM-call record is the fine-grained layer under the case trace.
- **The logging.** The structured logs with the case identifiers and the decision identifiers, the retention per the data-retention schedule (Section 5.1), and the log hygiene — the logs are subject to the same redaction and minimisation rules as the prompts, because the logs are a personal-data store by another name.

The observability and the audit trail (Section 11.3) are distinct: the observability is the operational view (fast, queryable, retained on the operations schedule), and the audit trail is the evidentiary view (immutable, retained on the regulatory schedule). The same instrumentation feeds both; the difference is the storage treatment.

### 11.3 The Audit Trails

The audit trail is the evidentiary record — the answer to the examiner's "reconstruct this decision" question (Section 8.3). The record-keeping expectations are carried by the MAS guide (cross-ref — the record-keeping and the retention obligations under the MAS regime, including the outsourcing record-keeping expectations under Notice 658); the engineering translation:

- **The immutable decision log.** Every consequential decision — the agent dispositions, the gate verdicts, the human approvals and overrides, the kill-switch activations — is written once to the append-only store (the write-once, read-many storage: the WORM storage or the cryptographic append-only log). The immutability is verified, not assumed: the store is tested for the tamper-evidence.
- **The full reconstruction record.** For each case: the inputs as seen (the redacted view with the redaction record), the orchestration trace, the LLM-call records, the tool-call records, the outputs, the human decisions, and the version identifiers of everything involved (the agent code, the prompts, the models, the frameworks).
- **The retention.** The retention periods follow the regulatory schedule (the MAS record-keeping expectations, cross-ref the MAS guide) and the bank's data-retention policy — the audit store is retained beyond the operational logs, and the retention is enforced by the storage lifecycle, not by the good intentions.
- **The access control.** The audit store is read-only to the operations staff and the write-capable only to the ingestion path; the access to the audit store is itself logged (the audit of the audit — cross-ref the [Cybersecurity](../cybersecurity_guide.md) guide).

### 11.4 The MAS-Supervision Expectations — Condensed

The MAS-supervision expectations for the multi-agent deployment are the MAS regime applied to this system, condensed here and carried in full by the sibling guides — the [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) guide and the [AI/GenAI Banking Compliance](../banking/ai_genai_banking_compliance_guide.md) guide. The condensed expectations:

- **The technology-risk expectations** — Notice 626 and the TRMG: the system is a technology service requiring the risk management, the change management, the incident management, and the operational-resilience treatment of any critical system (cross-ref the MAS guide).
- **The outsourcing expectations** — Notice 658: the agent platform, the model providers, and the evaluation vendors are outsourcing arrangements, with the materiality assessment, the due diligence, the contractual protections, the audit access, and the notification obligations (cross-ref the MAS guide).
- **The AI-governance expectations** — FEAT 2018 and the Model AI Governance Framework: the fairness (the bias testing across the strata), the ethics, the accountability (the named owners and the human accountability), and the transparency (the explainability of Section 8), applied to the whole system (cross-ref the MAS guide and the AI-compliance guide).
- **The record-keeping expectations** — the audit trails of Section 11.3, retained per the regulatory schedule.
- **The incident-reporting expectations** — the material incidents involving the system are reportable per the MAS expectations (cross-ref the MAS guide), and the incident record of Section 10.2 is the reporting substrate.
- **The model-governance expectations** — the model inventory, the validation, and the change control of Section 5.2, consistent with the MAS supervisory expectations for the models and the AI (cross-ref the [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) guide).

The condensed reading: the examiner asks four questions — who is accountable, what was tested, what was decided, and what happened when it failed. Sections 1–11 exist to answer those four questions; the worked example of Section 12 shows the answers in a concrete deployment.

### 11.5 The Error Budget and the Reliability Math

The reliability operations complete the production standards: the error-budget discipline that enforces the SLOs of Section 11.1 in the running system. The error budget is the agreed allowance of unreliability — the difference between the SLO target and the perfect service — and it is spent by the incidents and the degradations:

- **The availability error budget.** The budget burns with the outages and the degradations; the burn rate (the budget consumed per unit of time) is the alerting signal — the fast burn alerts immediately, and the slow burn alerts before the budget exhausts. The budget exhaustion is not a surprise; it is a scheduled conversation about the reliability investment.
- **The latency error budget.** The budget burns with the slow cases — the percentile breaches consume the allowance, and the alerting follows the same burn-rate logic.
- **The accuracy error budget.** The quality analogue: the override-rate and the drift allowances (Sections 6.4 and 7.3) are budgets too — the divergence allowance between the agent and the humans, spent by the override drift, with the same consequence discipline: the budget exhausted means the agent's judgment is under suspicion, and the kill-switch question is asked.
- **The cost budget.** The FinOps cap (Section 11.1) — the spend allowance per case, with the alerts and the caps enforced by the platform.

The error-budget reviews run on the operating cadence — the weekly operations review during the hypercare, the monthly review after — and the review records join the governance record. The reliability is a managed quantity, not a hope.

---

## 12. The Worked Example — the Cymbal Bank Multi-Agent Deployment

### 12.1 The Scenario

Cymbal Bank — the Singapore-based bank persona used across this repository — operates a retail and private-banking business under MAS supervision. The bank's financial-crime operations run three high-volume workloads that share the same case shape: the **fraud triage** (the transaction-monitoring alerts and the customer-reported fraud cases), the **AML screening** (the sanctions and watch-list screening hits plus the transaction-monitoring alert queue), and the **KYC** (the onboarding and the periodic-review cases). The three workloads are operated by three separate teams with three separate queues, and the same customer can appear in all three on the same day — the fraud case on the customer's account, the screening hit on the counterparty, and the periodic-review file all arriving in the same week. Cymbal Bank's programme is the **governed multi-agent deployment**: one orchestrated system, three specialist agents, the human decision-makers preserved in every workflow, and the whole system built to the standards of Sections 1–11.

The programme's objectives, agreed at the design gate: the fraud-triage backlog compressed by reducing the false-positive workload; the AML-screening false-positive flood reduced without moving the recall floor; the KYC file-assembly time cut; and — the non-negotiable objective — every consequential decision remains attributable to a named human, and every decision is reconstructable by an examiner. The programme budget and the timeline are fixed at the design gate: nine months from the design to the full production, with the platform work on the [AI Platform Engineering](../ai_platform_engineering_guide.md) estate running in parallel.

### 12.2 The Architecture — the Orchestrator and the Three Agents

Cymbal Bank selects the **hierarchical (supervisor) pattern** (Section 3.4) for the cross-workflow command, with the **orchestrator-workers** (Section 3.2) inside the fraud lane and the **pipeline (prompt chaining)** (Section 3.3) inside the KYC lane — the pattern selection matrix of Section 3.6 applied to the case shapes. The framework choice is LangGraph (Section 4.2), on the grounds of the graph's inspectability (the audit-trail fit), the state model (the evidence carried through the flow), and the Python estate the bank's AI platform team already runs — the decision recorded in the architecture decision record at the design gate.

The deployed topology:

| The Component | The Role | The Pattern | The Human Decision |
|---|---|---|---|
| The supervisor | The cross-workflow router — reads the intake, routes to the lanes, assembles the cross-lane picture | The hierarchical (supervisor) | The supervisor's routing is logged; the escalation decisions are the humans' |
| The fraud-triage agent | The triage of the transaction-monitoring alerts and the reported cases | The orchestrator-workers (the transaction-graph worker, the customer-history worker, the device-signal worker, the document reader) | The block/release decision; the customer contact |
| The AML-screening agent | The screening-hit assessment and the alert triage, the STR-narrative drafting | The pipeline inside the alert-triage worker (assess → assemble → draft) | The screening disposition; the STR filing |
| The KYC agent | The onboarding and periodic-review file assembly, the extraction, the rating, the checklist | The pipeline (extract → structure → rate → check), with the parallelization (sectioning) on the document set | The onboarding accept/decline; the EDD scope |

The guardrails of Section 5 are wired in from the first build: the intake redaction layer, the minimisation rules (the fraud lane sees the transaction and the history window, never the marketing profile), the tool-call allow-lists, the output filters, and the escalation contract — the EDD triggers, the list matches, and the low-confidence bands all route to the humans by construction, not by the agent's discretion.

### 12.3 The Evaluation Gates

Cymbal Bank builds the evaluation estate before the agents are finished — the golden set first, the harness second, the judge third:

- **The golden set.** The financial-crime and the compliance functions assemble the stratified golden set: 2,000 labelled cases across the three lanes — the confirmed-fraud cases, the sanctioned-name variants, the EDD profiles, the false-positive classics, the poor-quality documents, the adversarial inputs — each case labelled and signed off by the compliance function (the golden set as a compliance artefact, Section 6.2). The per-stratum floors are set at the design gate: the confirmed-fraud recall floor, the sanctioned-name recall floor, the KYC field-accuracy floor, the zero-tolerance false-pass on the checklist.
- **The harness.** The evaluation harness runs in the CI/CD pipeline: every pull request replays the golden set, and the score deltas against the baseline version are the release evidence (Section 6.5).
- **The judge.** The LLM-as-judge scores the narrative and the rationale quality on Cymbal Bank's rubric — the evidence-pack completeness, the citation accuracy, the escalation appropriateness — with the judge's own verdicts sampled for the human review, and the judge version pinned as a model-risk object (Section 6.3).
- **The gates.** The five quality gates of Section 7.1 run in sequence, and the guardrail gate is pass/fail — a single redaction failure blocks the release, no exceptions.

The evaluation finds what the evaluation is for: the first KYC-agent build passes the aggregate extraction accuracy but fails the poor-quality-document stratum — the aggregate hides the stratum failure, exactly the failure mode of Section 6.2 — and the build goes back for the document-quality handling before any release discussion.

### 12.4 The Phased Rollout

The rollout follows the sequence of Section 9.2, on the case-type canary rather than the user canary:

1. **The shadow phase (weeks 1–4).** The full system scores the live cases in shadow — the fraud alerts, the screening hits, the KYC files — with the outcomes untouched. The shadow scores are compared against the champion processes (the existing rule engines and the manual workflows). The shadow gate passes: the shadow scores meet the champion within the tolerance, and the latency and the cost are within the bars.
2. **The canary (weeks 5–8).** The fraud-triage agent takes the live triage on the low-risk, high-volume case stratum — 5% of the alert volume — with the rollback triggers armed: the error-rate bound, the latency bound, the override-rate bound, the guardrail-failure trigger. The kill switch is tested in the rehearsal before the canary: a scheduled drill pulls the agent, verifies the deterministic fallback (the existing fraud rules take the traffic), and restores.
3. **The champion-challenger measurement (weeks 9–12).** The fraud lane runs the live comparison of the champion process against the agent on the shared traffic ⚠ — the agent's triage dispositions are compared against the analysts' dispositions on the same cases, and the override rate is the challenger's scorecard. The challenger earns the promotion: the recall on the confirmed-fraud stratum holds, and the false-positive workload drops by the target margin.
4. **The graduated expansion (weeks 13–20).** The traffic share steps up through the agreed levels — 25%, 50%, 100% on the fraud lane — each step held for the agreed period and gated. The AML-screening agent and the KYC agent follow the same sequence on their lanes, staggered so that the operations teams absorb one change at a time.
5. **The full production (week 21).** All three agents at full share, the drift monitoring baselines established, the observability dashboards live, the hypercare begun.

### 12.5 The Hypercare

The hypercare runs for the agreed eight weeks, with the exit criteria fixed at the go-live: fourteen consecutive days without a P1 incident, the live re-scored golden-set performance within the floors, the override rates within the bounds, and the human queues stable (Section 10.3). The runbooks — the agent-quality runbook, the drift runbook, the outage runbook, the security runbook, the kill-switch runbook — are exercised in the first hypercare week, including a second scheduled kill-switch drill.

The hypercare earns its keep in week three: the override-rate drift alert fires on the fraud lane — the analysts are overriding the agent's low-urgency dispositions at twice the baseline rate. The drift runbook executes: the investigation finds the input drift — a new merchant-category pattern in the live traffic that the golden set does not cover — and the agent's urgency calibration is off on that pattern. The remediation: the new pattern is added to the golden set, the calibration is adjusted, the regression suite replays, the fix releases through the same gates, and the override rate returns to the baseline. The incident record — the timeline, the root cause, the remediation, the prevention evidence — is written for the examiner (Section 10.2), and the case joins the failure catalogue (Section 7.2). The hypercare exits on schedule: the stabilization metrics hold, and the sign-offs come from the operations, the compliance, and the model-risk functions.

### 12.6 The Audit Trail

The audit trail is the programme's evidentiary spine (Section 11.3), and the examiner's rehearsal happens throughout, not at the end. The Cymbal Bank decision-reconstruction drill: the examiner picks a case from the AML lane — a screening hit on a corporate counterparty that the analyst disposed as a false positive. The reconstruction record assembles in minutes:

- **The inputs as seen** — the redacted payment message, the hit details, the entity-resolution view, with the redaction record showing exactly what the agent saw.
- **The orchestration trace** — the supervisor's routing decision, the AML agent's pipeline (the match assessment, the context assembly, the narrative draft), the tool calls, the LLM-call records with the model and the prompt versions.
- **The rationale** — the near-match analysis: why the name variant was judged a false positive, the cited evidence, the confidence signal.
- **The human overlay** — the analyst's disposition, the approval timestamp, the analyst's own rationale note, the override flag (none — the analyst agreed).
- **The version identifiers** — the agent code, the prompts, the models, and the framework versions in force on the case date, recoverable from the pinned release record.

The record is immutable — written once to the append-only store, retained on the regulatory schedule (cross-ref the MAS guide), and read-only to the operations staff. The examiner's four questions (Section 11.4) — who is accountable, what was tested, what was decided, what happened when it failed — are answered from the same store, without a data archaeology project.

### 12.7 The Lessons

The Cymbal Bank deployment distils into the lessons the whole guide has been building toward:

1. **The contracts precede the agents.** The scope, escalation, evidence, and evaluation contracts were written at the design gate, and every subsequent decision — the pattern selection, the framework choice, the gate design, the rollout sequence — was the contracts made concrete. Nothing in the build required inventing the governance afterwards.
2. **The golden set is a compliance artefact.** The stratification and the compliance sign-off on the labels were what made the gate failures actionable — the aggregate-pass/stratum-fail episode (Section 12.3) would have shipped without the stratification.
3. **The rollout is a measurement programme.** The shadow, the canary, the champion-challenger measurement, and the expansion gates produced the evidence that the system earned the traffic — the rollout record answered the examiner's question before it was asked.
4. **The drift alert is a feature.** The hypercare override-drift episode (Section 12.5) was the system working: the instrumentation caught the divergence, the runbook handled it, the golden set grew, and the agent improved. The governed agent is not the agent that never errs; it is the agent whose errors are visible, bounded, and corrected inside the control framework.
5. **The human is the point.** The escalation contract kept the humans accountable and the override instrumentation kept the humans' judgment in the loop — the deployment reduced the workload without reducing the accountability, which is the entire purpose of the governed agent.

### 12.8 The Governance Record

The programme's standing governance record is the answer-key to the examiner's four questions (Section 11.4), assembled from the artefacts this guide has described:

- **Who is accountable** — the model inventory (Section 5.2): every agent and every model registered with the named owners; the escalation contract with the named accountable humans; the accountability structure consistent with the IAC expectations (cross-ref the MAS guide).
- **What was tested** — the evaluation-run archive (Section 6.6): the golden-set versions, the gate results, the release history — the record that every version earned its traffic.
- **What was decided** — the audit trail (Section 12.6): the immutable decision logs, the reconstruction records, the human approvals and the overrides.
- **What happened when it failed** — the incident records (Section 10.2): the override-drift episode, the drills, the remediations, and the regression-suite growth that followed.

The governance record is not a deliverable produced at the end of the programme; it is the programme's running ledger, written continuously from the design gate to the hypercare exit and beyond — and it is the reason the Cymbal Bank deployment can answer the examiner in minutes rather than months.

---

## 13. The Summary

The guide's argument in one reading: the multi-agent banking system is a **governed capability** — the four contracts of the governed agent (the scope, the escalation, the evidence, the evaluation) applied through the four parts of this guide. **Part 1** — the architecture: the agent taxonomy for the four workflows (fraud triage, AML screening, compliance automation, KYC), the verified orchestration patterns (the orchestrator-workers, the pipeline/prompt chaining, the hierarchical supervisor, with the routing and the parallelization in support), the frameworks with the verified facts and the honest flags on the adoption claims, and the guardrails cross-referenced to the banking guides. **Part 2** — the evaluation: the three pillars (the golden datasets, the LLM-as-judge, the task-specific metrics), the five quality gates, the regression suites, the drift monitoring, the explainability, and the examination fit. **Part 3** — the lifecycle: the design-to-production path, the phased rollout (the shadow, the canary, the champion-challenger, the kill switch), the hypercare with the runbooks and the stabilization metrics, and the client-team knowledge transfer. **Part 4** — the standards: the SLOs and the fallbacks, the observability, the immutable audit trails, and the condensed MAS-supervision expectations. **The worked example** — the Cymbal Bank deployment shows the whole system in operation, from the golden set to the examiner's reconstruction drill. The through-line, restated: the framework and the models are the easy part; the governance is the product.

The mapping for the examiner's four questions (Section 11.4) — where each answer lives in this guide:

| The Examiner's Question | The Answer Lives In |
|---|---|
| Who is accountable? | The escalation contract (Section 2.5), the model-risk overlay (Section 5.2), the human gate (Section 7.1), the knowledge transfer (Section 10.4), the governance record (Section 12.8) |
| What was tested? | The evaluation pipelines (Section 6), the quality gates (Section 7), the evaluation-run records (Section 6.6), the rollout gates (Section 9.3) |
| What was decided? | The decision rationale (Section 8.1), the audit trails (Section 11.3), the decision-reconstruction drill (Section 12.6) |
| What happened when it failed? | The drift monitoring (Section 7.3), the incident response (Section 10.2), the failure catalogue (Section 7.2), the incident record (Section 12.5) |

A reader who can answer those four questions from the records this guide describes has implemented the governed agent; a reader who cannot has not yet finished the design.

---

## 14. The Claims Audit

The claims audit table records the verification status of the key facts in this guide — ✅ for the facts verified against a primary or authoritative source in this pass, ✅-carried for the facts carried from this repository's sibling guides (verified in the prior passes), and ⚠ for the facts that could not be confirmed and are listed in Section 15:

| The Claim | The Status | The Source |
|---|---|---|
| The Anthropic "Building effective agents" taxonomy — the workflows vs the agents distinction; the five workflow patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) | ✅ | The Anthropic engineering post, December 19, 2024 (extracted this pass) |
| The orchestrator-workers definition — "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results" | ✅ | The Anthropic post (extracted this pass) |
| The prompt-chaining definition and the gate concept | ✅ | The Anthropic post (extracted this pass) |
| The hierarchical (supervisor) multi-agent architecture — the specialized agents coordinated by a central supervisor | ✅ | The LangChain documentation and the langchain-ai supervisor library (searched this pass) |
| LangGraph — the maintainer (LangChain), the languages (Python, JavaScript/TypeScript), the graph/state/supervisor abstractions | ✅ | The langchain-ai organisation sources (searched this pass); the license detail cross-ref the sibling framework comparison guide |
| CrewAI — the maintainer (CrewAI Inc.), the MIT license, the Python language, the Crews and Flows abstractions | ✅ | The docs.crewai.com and the crewAIInc GitHub repository (searched this pass) |
| AutoGen — the maintainer (Microsoft / Microsoft Research), the MIT license, the Python language, the conversational multi-agent abstraction | ✅ | The microsoft/autogen repository and the Microsoft Research project page (searched this pass); the Microsoft Agent Framework successor direction ⚠ |
| Semantic Kernel — the maintainer (Microsoft), the MIT license, the C#/Python/Java languages | ✅ | The learn.microsoft.com documentation and the microsoft/semantic-kernel repository (searched this pass) |
| The framework adoption claims ("the most popular", the star counts) | ⚠ | Marketing-grade claims; not verified — flagged rather than asserted |
| The LLM-as-judge practice — the paper, the >80% human-agreement finding, the documented biases (position, verbosity, self-enhancement), the MT-Bench and Chatbot Arena benchmarks | ✅ | arXiv:2306.05685, Zheng et al. (extracted this pass) |
| The golden datasets as the evaluation practice | ✅-carried | The sibling [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) and [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md) guides |
| The canary deployment for ML models — the gradual rollout, the traffic splitting, the rollback triggers | ✅ | The MLOps engineering references (mljourney, oneuptime, stackpractices — searched this pass) |
| The champion-challenger practice | ⚠ | Not confirmed against a primary source in this pass — see Section 15 |
| The kill-switch practice | ⚠ | Not confirmed against a single primary source in this pass — see Section 15 |
| The hypercare discipline — the post-go-live support period, the exit criteria | ✅-carried | The sibling [Vendor Management](../management/vendor_management_guide.md) guide |
| MAS Notice 658 (outsourcing), Notice 626 (technology risk), TRMG, FEAT 2018, the Model AI Governance Framework, the record-keeping and the incident-reporting expectations | ✅-carried | The sibling [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) guide and the [AI/GenAI Banking Compliance](../banking/ai_genai_banking_compliance_guide.md) guide |
| The PDPA/GDPR privacy themes, the KYC/EDD conventions, the AML-screening mechanics, the model-risk overlay | ✅-carried | The sibling banking guides ([AI/GenAI Banking Compliance](../banking/ai_genai_banking_compliance_guide.md), [Private Banking](../banking/private_banking_guide.md), [Fircosoft](../banking/fircosoft_guide.md), [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md)) |
| The evaluation tooling (LangSmith, MLflow) and the framework comparison detail | ✅-carried | The sibling [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md), [LLM Frameworks Comparison](llm_frameworks_comparison_guide.md), and [RAG Evaluation Tools Comparison](rag/rag_evaluation_tools_comparison_guide.md) guides |
| The observability and the platform practices | ✅-carried | The sibling [AI Platform Engineering](../ai_platform_engineering_guide.md), [Enterprise Agentic Platform Architecture](enterprise_agentic_platform_architecture_guide.md), and [AgentOps — LLM Observability](agentops_guide.md) guides |

No claim in this guide is marked ❌ — nothing was found to be false; the honest flags are the ⚠ items above, and nothing unverified is presented as verified.

---

## 15. What Could Not Be Verified

The items that this pass could not confirm from a primary source, flagged ⚠ throughout the guide and listed here honestly:

- **The champion-challenger citation.** The practice — comparing the deployed model (the champion) against the candidate (the challenger) on the live traffic before the promotion — is described in this guide as used (Section 9.2 and the worked example), but the verification searches for a primary citation returned empty in this pass (a targeted AWS source returned a 404), and no sibling guide in this repository carries the practice. The practice itself is standard model-management vocabulary, but it is flagged ⚠ rather than asserted as verified.
- **The kill-switch citation.** The mechanism — the pre-built, tested capability to take an agent out of the production path immediately — is in common use across the deployment and the AI-safety literature, but no single primary source was confirmed in this pass. The mechanism is mandated in this guide regardless of the provenance of the name; the name itself is flagged ⚠.
- **The Microsoft Agent Framework successor direction.** The AutoGen ecosystem has moved (the 0.4 architectural rewrite, the announced successor framework direction), but the successor's precise status could not be confirmed from a primary source in this pass — the guide flags the evolution ⚠ and advises re-validation against the live documentation at decision time.
- **The framework adoption claims.** The popularity claims (the adoption, the star counts, the "fastest growing" framing) are marketing-grade and were not verified; they are excluded from the framework table by design.
- **The per-workflow metric bars and the golden-set sizes.** The concrete numbers in the worked example (the 2,000-case golden set, the rollout week counts, the share steps) are Cymbal Bank's fictional programme parameters — they are illustrative by construction, not verified facts, and they are presented as the example's parameters rather than as industry benchmarks.
- **The SLO target shapes.** The availability and the latency targets are expressed as the shapes the bank sets, not as universal numbers — no industry-standard SLO values for banking agents could be verified in this pass, and none are asserted.

---

## 16. The Glossary

| The Term | The Meaning |
|---|---|
| The agent | The LLM-driven system that dynamically directs its own process and tool use (per the Anthropic taxonomy ✅) |
| The workflow | The system where the LLMs and the tools are orchestrated through predefined code paths (per the Anthropic taxonomy ✅) |
| The governed agent | The agent bounded by the four contracts — the scope, the escalation, the evidence, and the evaluation — this guide's central principle |
| The orchestrator-workers | The workflow pattern where the central LLM dynamically decomposes the task, delegates to the worker LLMs, and synthesizes the results ✅ |
| The prompt chaining | The workflow pattern where each LLM call processes the previous call's output, with the programmatic gates on the intermediate steps ✅ |
| The hierarchical (supervisor) | The multi-agent architecture where the specialized agents are coordinated by the central supervisor agent ✅ |
| The routing | The workflow pattern that classifies the input and directs it to the specialized follow-up task ✅ |
| The parallelization | The workflow pattern that runs the independent subtasks concurrently — the sectioning and the voting variations ✅ |
| The evaluator-optimizer | The workflow pattern where one LLM generates while another evaluates in a loop ✅ |
| The golden dataset | The curated, labelled set of the cases with the known-correct outcomes that every agent version must replay ✅-carried |
| The LLM-as-judge | The practice of using a strong LLM to evaluate the outputs of other LLMs, with the documented biases ✅ |
| The quality gates | The enforceable checkpoints between the agent versions and the production traffic |
| The drift monitoring | The live watching of the input, output, model, and override drift against the baselines |
| The canary deployment | The gradual rollout where the new version serves the small, controlled traffic subset first, with the rollback triggers armed ✅ |
| The champion-challenger | The comparison of the deployed model (the champion) against the candidate (the challenger) on the live traffic ⚠ |
| The kill switch | The pre-built, tested mechanism to take the agent out of the production path immediately ⚠ |
| The hypercare | The post-go-live support period with the heightened monitoring and the defined exit criteria ✅-carried |
| The runbook | The written procedure for the operating scenarios, exercised before the go-live |
| The SLO | The service-level objective — the agreed, measured target for the availability, the latency, the accuracy, or the cost |
| The fallback | The deterministic, degraded-mode, or manual path that takes the traffic when the agent cannot |
| The LLM observability | The per-call records — the model, the version, the prompt hash, the tokens, the latency, the cost, the output hash |
| The audit trail | The immutable, retained, evidentiary record of the decisions and the reconstructions |
| The HITL | The human-in-the-loop — the ladder of the human checkpoints per workflow |
| The EDD | The enhanced due diligence — the deeper KYC applied to the higher-risk customers ✅-carried |
| The STR | The suspicious transaction report — the AML filing drafted with the agent's help and filed by the human ✅-carried |
| The FEAT | The MAS Fairness, Ethics, Accountability and Transparency principles (2018) ✅-carried |
| The TRMG | The MAS Technology Risk Management Guidelines (2013, updated 2021) ✅-carried |
| The XAI | The explainable AI — the techniques that make the model decisions interpretable |

---

## 17. The Closing — the Governed Agent

The multi-agent banking system is not the triumph of the model; it is the triumph of the governance. The agents of this guide — the fraud-triage agent, the AML-screening agent, the compliance-automation agent, the KYC agent — earn their place in the bank's operations not because they are autonomous, but because their autonomy is bounded by the four contracts: the scope contract that says what they may attempt, the escalation contract that says what always comes to the human, the evidence contract that says every decision carries its rationale, and the evaluation contract that says nothing ships without passing the gates. The patterns are verified, the frameworks are compared with the honest flags, the evaluation is measured, the rollout is phased, the production is observed, and the decisions are recorded immutably — and at the end of every thread sits the accountable human, the FEAT principle made flesh, the reason the examiner finds the record in order and the customers find the bank behaving. That is the design, the deployment, the evaluation, and the running of the governed agent.
