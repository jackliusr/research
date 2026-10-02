# The Consolidated AI Risk Register for a Bank

*The single register no other guide in this repository provides: one uniform row per AI failure mode — mechanism, preventive control, detective control, the evidence an auditor would ask for, the named owner role, the residual risk after controls, and the guide that owns the depth. A map with owners, not a new framework. The thesis is the test: **a risk with no owner and no evidence is an observation, not a control.***

Jack Liu Shurui, Solution Architect

> **What this guide is — and where it sits.** This is the repository's **consolidated AI risk register**: the one artifact that assembles the failure modes an AI or GenAI deployment in a bank actually produces, and forces each of them to carry an owner, a control, an artifact, and a residual rating. It is deliberately **a map with owners, not a framework**. It does not re-derive the frameworks, the regulatory landscape, the model-risk discipline, or the risk taxonomy — every one of those has an owner guide in this repository, and this guide names that owner, cites the section, and moves on.
>
> - **Framework and operating-model layer** — NIST AI RMF, ISO/IEC 42001, the EU AI Act, the OECD principles, the three lines adapted to AI, model-risk integration, committee positioning — is owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) (§2–§7, and the banking regulatory mapping at §9). This guide does not restate it.
> - **Responsible-AI canon and the human-oversight design** — corporate and international RAI frameworks, the implementation playbook, the banking angle, human-in-the-loop placement — is owned by [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) (§2–§7).
> - **Model-risk discipline** — SR 11-7 lineage, independent validation, the three-lines vocabulary for models — is owned by [banking/risk_management_models_guide.md](./risk_management_models_guide.md) §9–§10.
> - **The banking AI risk taxonomy** — what the risk *families* are — is owned by [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §4.
> - **Register and tolerance mechanics, evidence and GRC operating discipline** — are owned by [banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §6 and [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §5–§6.
>
> **Verification discipline.** This guide makes almost no external factual claims: it is a register structure and a set of repository-internal map declarations. The one thing it asserts — that each failure mode is *owned in depth* by a named guide in *this* repository — was verified during this pass by inspecting each file at its absolute path and confirming its line count and section headings (the audit is in [§14](#14-the-claims-audit)). Where a mapping is a judgment call rather than a direct section match, it is flagged ⚠. Nothing here restates a jurisdiction's rule, a regulator's threshold, a bank's internal limit, or a rating scale as fact — the rating scheme in [§6](#6-rating-and-appetite) is labelled **illustrative** and the institution's own taxonomy governs.

---

### Contents

1. [The Overview, the Decoder, the Thesis and the Boundary](#1-the-overview-the-decoder-the-thesis-and-the-boundary)
2. [Why an AI Risk Register Is Not the Model-Risk Register](#2-why-an-ai-risk-register-is-not-the-model-risk-register)
3. [The Core: The Failure-Mode Inventory](#3-the-core-the-failure-mode-inventory)
4. [The Controls That Do the Work](#4-the-controls-that-do-the-work)
5. [Evidence and the Audit Question](#5-evidence-and-the-audit-question)
6. [Rating and Appetite](#6-rating-and-appetite)
7. [Three Lines and Ownership](#7-three-lines-and-ownership)
8. [Third-Party and Supply-Chain Entries](#8-third-party-and-supply-chain-entries)
9. [The Agentic Extension](#9-the-agentic-extension)
10. [The Regulatory Frame](#10-the-regulatory-frame)
11. [The Register in Operation](#11-the-register-in-operation)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified, and the Glossary](#15-what-could-not-be-verified-and-the-glossary)
16. [The Cross-References, and the Closing Summary](#16-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Decoder, the Thesis and the Boundary

### 1.1 The thesis

Most banks already have a model-risk register, an enterprise risk register, an operational-risk taxonomy, a third-party register and a control library. What they usually do **not** have is a single view of the AI deployment that says, for every way the thing can fail: *here is how it fails, here is what stops it, here is what catches it, here is the artifact that proves it, and here is the human being whose name is on it.*

That gap is what produces the sentence this guide is built to test:

> **A risk with no owner and no evidence is an observation, not a control.**

Read it as a filter. A line in a register that says *"hallucination — medium"* is an observation. The moment the same line reads *"fabricated customer-facing answer; preventive control = grounding-only answers with mandatory citation; detective control = faithfulness evaluation plus sampled review; evidence = evaluation report and monitoring baseline; owner = AI product owner; residual = medium"* it has become a control entry — because a named role is accountable for it and an artifact can be produced on request. The register exists to force that conversion, line by line.

### 1.2 The decoder: the ten words

The register is only useful if its vocabulary is fixed. These ten terms mean the same thing in every row of this guide.

| Term | What it means here |
|---|---|
| **Register** | The maintained, versioned list of AI risk *entries* for a defined scope (one system, one portfolio, or the enterprise AI estate), with the fields below. It is a living document with a cadence, not a slide. |
| **Entry** | One row: one failure mode, for one system or class of systems, with all fields populated. An entry with blank fields is incomplete by definition (see §11.3). |
| **Owner** | A *named role* (not a team, not a committee) accountable for the entry: for keeping the controls operating, the evidence current, and the residual rating honest. Ownership is a duty, not a reporting line — the ERM and GRC guides own the model ([banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §5–§6; [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §2). |
| **Rating** | The assessed severity/likelihood of the entry *before and after* controls, expressed in the institution's own scales. This guide uses only the words *high / medium / low* as placeholders and never asserts a threshold (see §6). |
| **Appetite** | The amount of a given risk the institution is willing to run, expressed as a statement the entry is tested against. Owned by the ERM guide §6. |
| **Control** | An action, configuration or process that changes the probability or impact of the failure mode. Split into **preventive** (stops it or reduces likelihood) and **detective** (finds it after the fact). A named control without an artifact is an assertion (see §5). |
| **Evidence** | The artifact an auditor or supervisor would be shown to prove the control operated: a report, a log, a config record, an approval, a test result, a decision record. |
| **Residual** | The risk that remains *after* the controls, as rated by the owner and challenged by the second line. Residual, not inherent, is what the register is for. |
| **Treatment** | The decision recorded against the entry: accept, mitigate, transfer, avoid, or (for AI systems) restrict the use case. In a register, "accept" must be an explicit decision with a rationale, never a default. |
| **Review cadence** | How often the entry is re-examined — tied to the system's risk tier and to the change cadence of the system, not to a calendar someone chose once. |

### 1.3 The boundary: who owns the depth

This guide provides one thing — the register — and refuses to duplicate its siblings. The boundary is declared by name:

| Owner guide | What it owns (do not expect it re-derived here) | How this guide uses it |
|---|---|---|
| [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) | Framework landscape (§2–§4), three lines adapted to AI (§5), model-risk integration and committees (§6), lifecycle gates and artifacts (§7), data & third-party governance (§8), **the regulatory mapping for a bank (§9)** | The register's §3 rows point here for gate and regulatory mechanics; §10 defers the entire regulatory map to §9 |
| [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) | RAI canon (§2–§3), tooling (§4), implementation (§5), banking angle (§6), worked framework (§7) | Human-oversight and automation-bias entries (§3 row FM-12, §4) defer to it |
| [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) | The LLM/GenAI security-risk taxonomy (§1–§13), defence-in-depth (§14), secure SDLC (§15), testing (§16) | Owns the mechanisms behind fabrication, injection, leakage, supply chain, excessive agency and denial of service |
| [technology/ai_llm/ai_governance_bias_redteaming_guide.md](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) | Bias types, fairness definitions, measurement, mitigation (§2–§6), red-teaming practice (§7–§9) | Owns the bias and fairness entries in depth |
| [technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md) | Attack taxonomy (§2), prompt injection (§3), jailbreaks (§4), agentic/RAG surfaces (§6), the red-team process (§8) | Owns the injection mechanism and the pre-launch red-team evidence |
| [technology/ai_llm/ai_agent_drift_guide.md](../technology/ai_llm/ai_agent_drift_guide.md) | Drift definition, types, detection, measurement, mitigation, banking context (§1–§8) | Owns the drift entry in depth |
| [technology/ai_verify_guide.md](../technology/ai_verify_guide.md) and [technology/ai_llm/ai_verify_toolkit_guide.md](../technology/ai_llm/ai_verify_toolkit_guide.md) | Verification/testing methodology and the toolkit (tests, engine, outputs) | Own the non-determinism and reproducibility entry and the testing-evidence format |
| [banking/risk_management_models_guide.md](./risk_management_models_guide.md) | Risk taxonomy (§2), model families, **model risk management and SR 11-7 (§9)**, ML in risk (§10) | The register's model-drift and bias rows defer to §9–§10 for discipline, not this guide |
| [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) | The banking AI requirements map, governance requirements (§3), **the GenAI risk taxonomy (§4)**, privacy (§5), security (§6) | Owns the requirements framing and the taxonomy this register instantiates |
| [banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) | ERM frameworks (§2–§3), risk taxonomy (§4), governance and three lines (§5), **risk appetite (§6)**, the risk process (§7) | Owns the mechanics of rating, appetite and the risk process |
| [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) | Control design (§5), control testing/RCSA/assurance mapping (§6), issue remediation (§8), KRIs (§9), board reporting (§11) | Owns the evidence regime and the operating discipline this register plugs into |
| [technology/ai_llm/rag/rag_conflict_resolution_guide.md](../technology/ai_llm/rag/rag_conflict_resolution_guide.md) | Conflict kinds, detection, the resolution ladder, authority/entitlement, temporal conflict (§2–§8) | Owns the stale/conflicting-knowledge entry in depth |
| [technology/ai_llm/autonomous_agents_guide.md](../technology/ai_llm/autonomous_agents_guide.md), [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md), [technology/ai_llm/llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md), [technology/ai_llm/agents_work_fall_apart_guide.md](../technology/ai_llm/agents_work_fall_apart_guide.md) | Agent definitions and control (§1–§6), sandboxing, production failure taxonomy, success/failure conditions | Own the agentic extension (§9) in depth |
| [technology/zero_trust_network_architecture_guide.md](../technology/zero_trust_network_architecture_guide.md) and [technology/zero_trust_legacy_estate_guide.md](../technology/zero_trust_legacy_estate_guide.md) | Zero-trust architecture and its application to legacy estates | Own the mechanism behind scoped-identity and action-boundary controls (§9) |

### 1.4 The honest statement

This guide is **a map with owners, not a framework.** It invents no models of maturity, no scoring rubric, no control catalogue and no new regulatory interpretation. Every mechanism it points at is built and explained in a guide that already exists in this repository; every control it names is a control whose mechanics the reader can go and read elsewhere. What is new here is only the **assembly and the accountability discipline**: one uniform row per failure mode, every row carrying an owner and an artifact, every "covered by" cell resolving to a guide that exists.

That honesty is load-bearing. A register that pretends to be a framework becomes a document nobody maintains, because it competes with the real frameworks for the reader's attention and loses. A register that is frankly a map with owners is cheap to maintain and hard to argue with, because its only claim is: *this failure mode is owned here, controlled like this, evidenced like that.*

### 1.5 How to read this guide

| Reader | Start with | Then |
|---|---|---|
| Risk / audit / compliance | §3 (the inventory), §5 (evidence), §6 (rating), §14 (claims) | §12 (the worked example) as the acceptance test |
| AI product owner / engineer | §3, §4 (controls), §9 (agentic), §11 (operation) | §13 (anti-patterns) as a self-check |
| Model risk / validation | §2 (why this is not the model-risk register), §3 rows FM-05/FM-06, §7 | §6 for the illustrative rating boundary |
| Architect / platform owner | §4, §8 (supply chain), §9 (agentic) | §10 (regulatory owners) and §16 (cross-references) |
| Executive / board support | §1, §6, §7, §12 | §13 (anti-patterns) as the health check |

---

## 2. Why an AI Risk Register Is Not the Model-Risk Register

### 2.1 What the bank already has (and does not need invented here)

A bank's risk estate is mature before AI arrives. It already has:

- **A model inventory and an independent validation function**, with a discipline that long predates GenAI — the SR 11-7 lineage of development soundness, independent validation, and governance. That is owned by [banking/risk_management_models_guide.md](./risk_management_models_guide.md) §9, and its extension to ML is §10 there.
- **An enterprise risk taxonomy** spanning credit, market, operational and liquidity risk, with tolerance mechanics and an enterprise register. That is owned by [banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §4 and §6.
- **Three lines of defence as a working relationship**, with the second line's framework/monitoring/challenge and the third line's independent assurance. Owned by the ERM guide §5 and operationalised in [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §2–§7.
- **A control library, an assurance map, and an evidence and issue-management regime.** Owned by the GRC guide §5–§6 and §8.
- **A banking AI requirements map and risk taxonomy for AI use cases.** Owned by [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §3–§4.
- **An AI governance operating model** — gates, committees, third-party AI governance. Owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §6–§8.

None of that is in question, and none of it is re-derived here. The question this guide answers is narrower: *given all of that, why does the AI deployment still fall through the gaps, and what does the register do about it?*

### 2.2 The failure modes are not the model's alone

A model-risk register asks: is the model sound — well designed, validated, monitored, and fit for its stated purpose? That remains necessary and it is owned above. But the failure modes that hurt an AI deployment most often arise not in the model but in **the system around the model**:

- **The retrieval corpus.** A grounded assistant can be perfectly sound as a model and still answer with an expired interest rate because the corpus held a superseded document — a *knowledge* failure, not a model-soundness failure. Owned by [technology/ai_llm/rag/rag_conflict_resolution_guide.md](../technology/ai_llm/rag/rag_conflict_resolution_guide.md).
- **The tool surface.** An agent can be a sound language model and still wire-fund the wrong account because its payment tool was over-scoped. A *permission* failure, not a model failure. Mechanism owned by [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §9–§10 and [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md).
- **The prompt and the retrieved content.** An assistant can be sound and still be steered by instructions hidden in a document it retrieved. An *injection* failure, not a model failure. Owned by [technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md) §3.
- **The human around it.** A sound model with a reviewer who rubber-stamps produces automation bias. A *process* failure. Owned by [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) §5–§6.

A register that classifies these as "model risk" mis-assigns them to the model owner, who does not control the corpus, the tool scope, the prompt, or the reviewer — the AI-specific ownership gap that §7 addresses directly.

### 2.3 Three things genuinely new

Strip away what the model-risk register already covers, and three shapes remain that are new to the AI estate:

1. **The artifact changes without a change record.** A foundation model or hosted service can change under the bank — a provider update, a default change, a silent behavioural shift — with no version bump the bank controls and no change ticket in its own system. The classical assumption that the validated artifact is frozen until *the bank* changes it does not hold. This is the drift and supply-chain pair (FM-06, FM-10), owned by [technology/ai_llm/ai_agent_drift_guide.md](../technology/ai_llm/ai_agent_drift_guide.md) and [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §8.
2. **The system acts.** A model that predicts does not move money; an agent that acts does. Autonomy introduces compounding failure, privilege accumulation, and action cascades that have no analogue in the predict-only estate. Owned by the agentic guides, assembled in §9.
3. **The third party can change the thing under you.** Provenance, training data, safety behaviour and availability are the provider's to change; the bank's control is at best contractual and architectural. Owned by the framework guide's third-party section (§8) and the supply-chain row in §3.

### 2.4 What this guide deliberately does not do

It does not re-state SR 11-7 or its successors (that is [banking/risk_management_models_guide.md](./risk_management_models_guide.md) §9); it does not re-state the risk taxonomy ([banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §4 and [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §4); it does not re-state the framework landscape or the regulatory map ([technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §2–§4, §9); it does not re-derive the three-lines model ([banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §5). It builds the one artifact none of them is: the consolidated register, with an owner and an artifact on every line.

---

## 3. The Core: The Failure-Mode Inventory

### 3.1 The uniform table

Every row below is one entry in the register. The columns are fixed: a failure mode, the mechanism by which it actually happens, a preventive control, a detective control, the evidence an auditor would ask for, the named owner role, the residual risk after controls, and the repository guide that covers the mechanism in depth. The ratings are placeholders only (see §6). The owner roles are named roles, not teams; where a second line oversees, that is noted in §7 rather than in this table.

| ID | Failure mode | Mechanism (how it actually happens) | Preventive control | Detective control | Evidence an auditor would ask for | Named owner role | Residual after controls | Covered by |
|---|---|---|---|---|---|---|---|---|
| **FM-01** | Fabricated or unsupported output | The model produces fluent content not grounded in the retrieved corpus or in any reliable source — invented figures, plausible-but-false citations, wrong product terms presented with confidence | Grounding-only answers with mandatory citation; bounded answer domains; refusal and "I don't know" policy; curated corpus; decoding constrained for factual tasks | Faithfulness/grounding evaluation; hallucination-rate monitoring against a baseline; sampled human review of answers; user-feedback triage | Evaluation report with grounding metrics; monitoring baseline and thresholds; a sample of flagged outputs with dispositions | **AI Product Owner** (first line) | Medium where customer-facing and ungrounded; reduced by grounding plus review | [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §4; [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §11 |
| **FM-02** | Prompt injection and indirect instruction injection | Instructions arrive through user input or, worse, through retrieved content — a document, email, web page, or record the system reads — and the model treats data as instruction, redirecting its behaviour | Input and output guardrails; instruction/data separation; sanitisation of retrieved content; least-privilege tool scope; allowlisted destinations | Injection-attempt telemetry and alerting; red-team regression suite run on change; anomalous tool-call monitoring | Red-team report with injection findings and fixes; guardrail configuration and version; injection-attempt telemetry and response records | **AI Security Lead** (first line security) | High where the agent reads untrusted content; medium where read-only | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §3; [technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md) §3 |
| **FM-03** | Data leakage and exfiltration | The system echoes personal or confidential data from the prompt, from memorised training data, from the retrieval corpus, or from a prior session; or an agent ships data to an external endpoint it should never reach | Data classification and redaction before the model sees the prompt; tenant/session isolation; egress and DLP controls; prohibition on sensitive data in prompts; least-privilege data access | Egress monitoring; DLP alerting; output scanners for classified patterns; session-isolation tests | DLP policy and alert log; redaction/leak test results; data-flow diagram; egress-control configuration | **Data Protection Officer / Information Security** (second line) jointly with the **Platform Owner** | Low–medium with egress controls; high without them | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §8 |
| **FM-04** | Over-permissive tool or action scope | The agent is granted broad credentials or APIs it does not need; a benign or manipulated prompt triggers a consequential action; scope creeps across sprints as features are added | Least privilege per tool; short-lived scoped credentials; action allowlists; read-only by default, write behind approval; explicit action inventory | Periodic permission/access review; tool-call audit; anomaly detection on action patterns | Tool-permission matrix; credential-scoping design; access-review records; tool-call audit log | **Agent/Platform Engineer** (first line) with **IAM** | Medium; reduced by the approval gate on irreversible actions | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §9–§10; [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md) §10 |
| **FM-05** | Uncontrolled agent autonomy and compounding actions | A multi-step loop feeds each step's small error into the next; runaway iteration, repeated retries, or a cascade of external effects with no human commit point | Step and budget caps; circuit breakers; idempotent actions; a human commit point before irreversible steps; execution in a sandbox | Run-level tracing; anomaly thresholds on step count, spend, and side effects; kill-switch telemetry | Run traces; guardrail/limit configuration; incident postmortems; step-cap and budget settings | **Product Owner for the agent** (first line) | Medium–high; falls as the reversibility of actions rises | [technology/ai_llm/llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md) §1; [technology/ai_llm/autonomous_agents_guide.md](../technology/ai_llm/autonomous_agents_guide.md) §5 |
| **FM-06** | Model drift and silent behaviour change | The provider updates the model under the bank; the input distribution shifts; prompts or retrieval change; behaviour degrades gradually with no visible event and no change ticket | Pinned model versions where offered; contractual change-notification; canary/shadow before promotion; change-record discipline | Behavioural drift monitoring; regression evaluation on a fixed cadence; output-distribution tracking | Version register with pin records; drift dashboard; regression-evaluation results; change log | **Model Owner** (first line) | Medium; higher for third-party models without version control | [technology/ai_llm/ai_agent_drift_guide.md](../technology/ai_llm/ai_agent_drift_guide.md) §3, §5 |
| **FM-07** | Bias and unfair outcomes | Training data or the retrieval corpus under-represents groups; proxy variables stand in for protected traits; feedback loops amplify disparity; language and dialect gaps degrade service unevenly | Fairness constraints at design; representative data and corpus review; bias-aware feature review; pre-launch bias audit | Disparate-impact monitoring by cohort; fairness metrics tracked over time; complaint and appeal analysis | Bias-audit report; fairness-metric definitions and results; mitigation steps taken; monitoring thresholds | **Model Owner** with **Fairness/Conduct** (second line) | Medium; fairness is managed, not solved — monitored after mitigation | [technology/ai_llm/ai_governance_bias_redteaming_guide.md](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) §2, §4 |
| **FM-08** | Non-determinism and irreproducibility | Sampling, provider-side variation, hardware and floating-point differences, and context ordering mean the same input yields different outputs — and a past decision cannot be replayed exactly | Temperature zero where determinism is required; seed control where available; full-context capture; deterministic retrieval ordering | Replay tests on sampled decisions; output-stability checks; reproduction audits | Replay log with seeds and parameters; stability-test results; reconstruction of sampled decisions | **AI Platform / MLOps Owner** (first line) | Medium; provider-side non-determinism cannot be fully controlled | [technology/ai_llm/ai_verify_toolkit_guide.md](../technology/ai_llm/ai_verify_toolkit_guide.md) §3, §7; [technology/ai_verify_guide.md](../technology/ai_verify_guide.md) §5 |
| **FM-09** | Stale or conflicting source knowledge | The retrieval corpus holds an expired document, or two documents disagree, and the system picks one silently — answering with superseded rates, terms, or procedures | Corpus freshness SLAs; authority and entitlement metadata; conflict detection at ingest; effective-dating of documents | Freshness monitoring; conflict-scan reports; spot-checking of temporal answers against point-in-time truth | Corpus governance record; freshness dashboard; conflict-resolution decisions; point-in-time test results | **Knowledge / RAG Custodian** (first line) with **Data Governance** | Medium; conflict is inherent and needs a recorded decision | [technology/ai_llm/rag/rag_conflict_resolution_guide.md](../technology/ai_llm/rag/rag_conflict_resolution_guide.md) §5, §7 |
| **FM-10** | Third-party and model-supply-chain change | The provider deprecates a model or API, changes a default, revises terms or price, or ships a behavioural change with little notice — the artifact moves without the bank's change record | Contractual notice, versioning and deprecation clauses; an exit plan; an abstraction layer over the provider; pinned versions where offered | Vendor-change monitoring; contract-review cadence; regression evaluation after any announced change; deprecation watch | Contract clauses; supplier due-diligence file; exit plan; post-change evaluation results | **Third-Party/Vendor Risk Owner** (second line) with **Procurement**; **Product Owner** for impact | Medium–high; concentration raises it further | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §7; [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §8 |
| **FM-11** | Concentration and availability dependency | Many systems depend on one provider, region, or endpoint; an outage or a rate limit halts a business process; the dependency is a single point of failure nobody drew on a diagram | Multi-provider abstraction; a documented degraded mode; capacity and rate-limit planning; regional redundancy | Availability monitoring; dependency mapping; failover testing | Dependency register; failover-test results; SLA and monitoring records; degraded-mode runbook | **AI Platform Owner** with **Operational Resilience** (second line) | Medium; fallback reduces but does not remove the dependency | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §6; [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §8 |
| **FM-12** | Human-factor risk: automation bias and review that does not review | Reviewers rubber-stamp fluent output; over-reliance replaces judgement; alert fatigue sets in; approval thresholds sit so low they force no engagement — the human control is present on paper and absent in practice | Human-in-the-loop placed at decision-relevant points; challenge prompts embedded in the workflow; sampled checks against ground truth; workload designed so review is possible | Reviewer-accuracy checks; override-rate monitoring; quality assurance on approvals; audit re-performance of sampled decisions | Oversight design record; QA results on approvals; override statistics with rationale; reviewer training records | **Business Process Owner** (first line) with **Conduct/Human Oversight** (second line) | Medium; the human control is also a risk and must be tested | [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) §5, §6 |
| **FM-13** | Output used outside its validated domain (scope creep of use) | A system validated for one use case is stretched to another; users push prompts beyond intent; a capability is marketed beyond its evidence; no re-assessment fires when the use changes | A registered use-case statement; domain enforcement at the interface; change control triggered by any use change; re-assessment on scope change | Usage monitoring against the registered purpose; prompt-pattern analysis; periodic recertification | Use-case register entries; monitoring of actual versus registered usage; re-assessment records | **AI Product Owner** (first line) with **AI Governance/Model Risk** (second line) | Medium; rises fast when a use case is stretched quietly | [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §7; [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §4 |

### 3.2 Reading a row

Take FM-02 and read it as an auditor would. The failure mode is named concretely (not "security risk"); the mechanism explains *how it actually happens* (through retrieved content, not only user input — the part teams forget); the controls split into something that reduces likelihood and something that finds it; the evidence names artifacts that either exist in a file share or do not; the owner is one role; the residual is stated honestly as high-or-medium depending on whether the agent reads untrusted content; and the last cell resolves to a guide that exists at a verified path. That is a control entry. Change the owner to "the AI team" and delete the evidence column and the same row becomes an observation.

Take FM-13 and notice the shape of a mode teams miss. Nothing in it is exotic — it is the ordinary slide from "we validated this for customer FAQs" to "let's also let it summarise complaint files." It is the mode the first-draft register in §12 omits entirely, and it is the one that quietly invalidates a validation that was, at the time, perfectly correct.

### 3.3 The owner map

The same inventory, viewed by owner, is what makes an entry routable. This table does not restate the controls; it answers "whose name is on it, who challenges it, and who can audit it" (§7 expands the lines).

| ID | Failure mode (short) | Owner role (first line) | Challenger (second line) | Auditable by (third line) |
|---|---|---|---|---|
| FM-01 | Fabrication / unsupported output | AI Product Owner | Model Risk / AI Governance | Internal Audit (AI/Model) |
| FM-02 | Prompt / indirect injection | AI Security Lead | CISO / Security oversight | Internal Audit (Cyber/AppSec) |
| FM-03 | Data leakage / exfiltration | DPO / Information Security (joint with Platform Owner) | Privacy & Data Protection | Internal Audit (Privacy/Data) |
| FM-04 | Over-permissive tool scope | Agent/Platform Engineer | Security Architecture | Internal Audit (Cyber) |
| FM-05 | Autonomy / compounding actions | Product Owner (agent) | AI Risk / Operational Risk | Internal Audit (AI/Ops) |
| FM-06 | Model drift / silent change | Model Owner | Model Risk (MRM) | Internal Audit (Model/Validation) |
| FM-07 | Bias / unfair outcomes | Model Owner | Fairness / Conduct | Internal Audit (Conduct) |
| FM-08 | Non-determinism / irreproducibility | AI Platform / MLOps Owner | Model Risk / Technology Risk | Internal Audit (Technology) |
| FM-09 | Stale / conflicting knowledge | Knowledge / RAG Custodian | Data Governance | Internal Audit (Data) |
| FM-10 | Third-party / supply-chain change | Third-Party Risk Owner | Vendor Risk / Procurement | Internal Audit (Third Party) |
| FM-11 | Concentration / availability | AI Platform Owner | Operational Resilience | Internal Audit (Resilience) |
| FM-12 | Human factors / automation bias | Business Process Owner | Conduct / Human Oversight | Internal Audit (Conduct/Ops) |
| FM-13 | Use outside validated domain | AI Product Owner | AI Governance / Model Risk | Internal Audit (AI/Model) |

The map is deliberately boring, and that is the point. Every AI failure mode resolves to a role that already exists in the bank's risk estate — there is no need to invent an "AI risk owner" species. The AI-specific difficulty is not that no role fits; it is that the *wrong* role usually gets assigned, which §7 takes up.

### 3.4 The uniform rules for the table

Four rules keep the inventory a register rather than a wish-list:

1. **No blank cells.** An entry missing an owner, a control, or an evidence artifact is marked incomplete and escalates; it does not sit in the register as if it were covered (§11.3).
2. **One owner per entry.** A committee may *oversee* an entry; a role *owns* it. "Committee-owned" means unowned.
3. **Every "covered by" cell resolves to a guide that exists.** This is verifiable and was verified (see [§14](#14-the-claims-audit)); a dangling reference is treated as a defect in the register.
4. **Residual, not inherent.** The rating recorded is what remains after the named controls — which forces the owner either to defend the controls or to admit the residual is higher than the label suggests.

---

## 4. The Controls That Do the Work

The inventory names *what* controls each failure mode; this section names the *mechanisms* — the recurring control families a bank builds, independent of any vendor. No product is endorsed, ranked, or credited here; the mechanisms are configuration, process, and design patterns that any platform can implement, and their deeper engineering is owned by the cross-referenced guides.

### 4.1 Input and output controls

These are the controls at the boundary of the model — what goes in and what comes out.

- **Input filtering and classification** — detecting disallowed content, instruction-like text, and classified data before the model sees it. Reduces FM-02 and FM-03.
- **Instruction/data separation** — the structural discipline of never letting retrieved content sit in the same privilege channel as instructions. Reduces FM-02.
- **Output validation** — schema checks, refusal checks, blocklist scans, and PII scans on the way out. Reduces FM-01 and FM-03.
- **Answer-domain bounds** — an explicit allowlist of what the system may answer; transactional or advisory topics are blocked or handed off. Reduces FM-01 and FM-13.

The point of the pair is asymmetry: input controls reduce the *likelihood* a bad instruction or datum enters; output controls reduce the *impact* if it does. An entry with only one half is half-controlled.

### 4.2 Retrieval and grounding controls

Grounding is what turns a generator into a system whose answers can be sourced. The controls that make it real:

- **Curated corpus with ownership** — every source has a custodian and a freshness expectation. Reduces FM-09.
- **Authority and entitlement metadata** — the corpus records which document wins when two disagree, and who may see what. Reduces FM-09 and FM-03; mechanism owned by [technology/ai_llm/rag/rag_conflict_resolution_guide.md](../technology/ai_llm/rag/rag_conflict_resolution_guide.md) §7.
- **Conflict detection at ingest** — flagging contradictory documents rather than silently indexing both. Reduces FM-09.
- **Mandatory citation and answer-only-from-cited-passage** — the model may not answer beyond the retrieved and cited text. Reduces FM-01.
- **Faithfulness evaluation** — measuring whether each answer is entailed by its sources, as a pre-launch gate and a monitoring metric. Reduces FM-01.

### 4.3 Tool-scope and permission design

When the system acts, the tool surface is the risk surface.

- **Least privilege per tool** — each tool carries the minimum scope for its function, not a broad credential reused across tools. Reduces FM-04.
- **Short-lived, scoped credentials** — access bounded in time and target; no standing all-powerful service account. Reduces FM-04.
- **Read-by-default, write-behind-approval** — irreversible actions require an explicit human commit. Reduces FM-04 and FM-05; the approval boundary is developed in §9.
- **Action allowlists and destination allowlists** — the agent may call only known tools and reach only known endpoints. Reduces FM-02 and FM-04.
- **Sandboxing of execution** — running tool code in an isolated environment with no ambient authority. Reduces FM-05; mechanism owned by [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md) §2, §10.

### 4.4 Human-in-the-loop placement

Human oversight is a control only where it is *placed at a decision-relevant point and made possible by the workflow*. Placing a reviewer after an irreversible action is theatre.

- **Commit points before irreversible effects** — the human approves the money move, the filing, the customer communication. Reduces FM-05.
- **Challenge prompts and ground-truth sampling** — the reviewer is prompted to disagree and is occasionally tested against known answers. Reduces FM-12; practice owned by [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) §5.
- **Escalation and handoff triggers** — the system must route to a person on defined conditions, not only on its own confidence. Reduces FM-01 and FM-12.
- **Workload that permits review** — a queue so large that review is impossible has no control in it. Reduces FM-12.

### 4.5 Logging that preserves evidence

Logging is a control family because most detective controls run on it, and because evidence is what separates a control from an assertion (§5).

- **Decision records** — for each consequential output: inputs, retrieved sources, model version, parameters, and the outcome. Supports FM-06, FM-08, and reconstruction for audit.
- **Trace/run records for agents** — step-by-step execution with tool calls, so a cascade can be reconstructed (FM-05).
- **Immutable retention** — logs written where they cannot be quietly edited, retained per the bank's records schedule (owned by [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §5–§6).
- **Telemetry that feeds detection** — injection attempts, drift, fairness, availability, and anomaly signals all keyed to the same system identity. Enables FM-02, FM-06, FM-07, FM-11.

A log nobody reads is storage, not evidence. The detective column in §3 names which log is *watched* and by whom.

### 4.6 Evaluation as a gate, not a report

An evaluation that produces a document filed once is a report. An evaluation that can *block* a deployment is a gate. The register's preventive controls assume the second kind: pre-launch evaluation with pass/fail criteria attached to the deployment decision, and regression evaluation re-run on a cadence and on change. The gate structure is owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §7; the tooling and output format by [technology/ai_llm/ai_verify_toolkit_guide.md](../technology/ai_llm/ai_verify_toolkit_guide.md) §7–§8. Evaluation types that matter to the register: grounding/faithfulness, injection resistance, fairness across cohorts, stability/reproducibility, and boundary tests for scope (FM-13).

### 4.7 Versioning discipline

Because an AI system is a bundle — model, prompt, corpus, guardrails, tools, parameters — versioning is what makes a claim of "unchanged" credible.

- **A composite system version** that records all of the above together, so a change to any one is a change to the system.
- **Pinning where the provider offers it, and a change record where it does not.**
- **Prompt and guardrail version control** with the same rigour as code; these are behaviour.
- **Corpus versioning** tied to the retrieval index, so a point-in-time answer can be reproduced (FM-08, FM-09).

The composite version is what lets FM-06's detective control mean anything: without it, "the model changed" is unfalsifiable.

### 4.8 The control-to-mode map

| Control family | Reduces primarily | Detective feeds |
|---|---|---|
| Input/output controls (§4.1) | FM-01, FM-02, FM-03, FM-13 | Output scans, refusal telemetry |
| Retrieval/grounding (§4.2) | FM-01, FM-09 | Faithfulness metrics, freshness dashboards |
| Tool-scope/permission (§4.3) | FM-02, FM-04, FM-05 | Tool-call audits, egress monitoring |
| Human-in-the-loop (§4.4) | FM-05, FM-12 | QA on approvals, override stats |
| Logging/evidence (§4.5) | (enables all detective) | All |
| Evaluation as gate (§4.6) | FM-01, FM-02, FM-05, FM-07, FM-13 | Regression results, drift |
| Versioning (§4.7) | FM-06, FM-08, FM-09, FM-10 | Change logs, replay tests |

---

## 5. Evidence and the Audit Question

### 5.1 What a reviewer actually asks

For each failure mode, the reviewer is not asking "do you have a control?" — every register says yes. The reviewer is asking for the artifact. The question and the artifact differ per mode:

| ID | The audit question | The artifact that answers it |
|---|---|---|
| FM-01 | "How do you know the answers are grounded, and how often are they not?" | Faithfulness evaluation report; hallucination-rate metric and baseline |
| FM-02 | "Show me your last red-team run and what you fixed." | Red-team report with injection findings, fixes, and re-test |
| FM-03 | "Show me the data-flow and prove classified data cannot leave." | Data-flow diagram; DLP/egress configuration; leak-test results |
| FM-04 | "List every tool the agent can call and its scope." | Tool-permission matrix; access-review record |
| FM-05 | "Show me a run trace for a case that hit the limit." | Step/budget caps config; a sampled run trace; incident file |
| FM-06 | "What changed, when, and how would you know?" | Version register; drift dashboard; regression-eval results |
| FM-07 | "Show me fairness results by cohort, before and after mitigation." | Bias-audit report; fairness metrics; mitigation decisions |
| FM-08 | "Reproduce this output from this decision." | Replay log with parameters; reconstruction of a sampled case |
| FM-09 | "Why did the system answer with an expired term?" | Corpus governance record; freshness dashboard; conflict decisions |
| FM-10 | "What did the supplier undertake to tell you, and what is your exit?" | Contract clauses; exit plan; post-change evaluation |
| FM-11 | "What happens to the business when the provider is down?" | Dependency register; failover test; degraded-mode runbook |
| FM-12 | "How do you know your reviewers are paying attention?" | QA results on approvals; override stats; reviewer-accuracy check |
| FM-13 | "What was this system validated to do, and what is it being used for?" | Registered use-case statement; usage versus purpose monitoring |

### 5.2 A claim of control without an artifact is an assertion

This is the mechanical reason the thesis holds. "We have guardrails" is an assertion; "here is the guardrail configuration, its version, and the red-team run that tested it" is a control. "We monitor drift" is an assertion; "here is the drift dashboard and the alert that fired last month" is a control. The register does not require every control to be *perfect*; it requires every control to be *demonstrable*. An entry whose evidence cell cannot be filled is, by the thesis, an observation — and it should be labelled as such rather than counted as coverage.

### 5.3 Tying the register into the existing evidence regime

The register does not invent an evidence system. Each artifact above is filed in the bank's existing regime:

- **Control library and testing** — the controls named in §3 are registered as controls and tested per the GRC guide's control-design and testing/RCSA discipline ([banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §5–§6).
- **Assurance mapping** — each entry's controls and evidence map onto the assurance map, so the third line can see what is assured and by whom (GRC guide §6).
- **Issues and remediation** — a failed control or a missing artifact becomes an issue with an owner and a due date (GRC guide §8).
- **KRIs** — the monitoring metrics (hallucination rate, drift signals, injection attempts, override rates) are candidate key risk indicators, reported per the GRC guide's KRI framework (§9).
- **Appetite and board reporting** — residual ratings roll up against appetite and to the board per the ERM and GRC reporting machinery ([banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §6–§7; GRC guide §11).

### 5.4 The honest note

**An AI register is only as good as the records behind it.** A register built from memory, populated with confident-sounding controls and no files, will survive exactly one audit. The register's value is not the table — it is the discipline of refusing to record a control until the artifact that proves it exists. Where a bank cannot produce the artifact, the honest register entry is not "controlled" but "asserted, evidence pending" — and that entry should carry an owner and a deadline, which is precisely how it becomes a real one.

---

## 6. Rating and Appetite

### 6.1 The rating scheme is illustrative — the institution's own taxonomy governs

> **This sub-section is explicitly illustrative.** The scales, bands, and labels below are a pedagogical placeholder to show *how* an AI risk is rated. They are **not** any institution's thresholds and must not be read as a standard. A bank's own risk taxonomy, rating matrix, and appetite framework govern; those mechanics are owned by [banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §6 and [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §9. No internal limit, threshold, or score appears in this guide as fact.

Rated in the institution's own scales, an AI risk entry is typically assessed on two axes — **impact** and **likelihood** — and then rated **before controls** (inherent) and **after controls** (residual). Three habits make the rating useful:

1. **Rate the residual, not the inherent, as the headline.** The register's job is to show what is left after the controls the owner actually runs.
2. **Tie the rating to the failure mode's blast radius**, not to how advanced the technology feels. An ungrounded customer-facing answer and an internal drafting error are not the same risk even if both are "hallucination."
3. **Write one line of rationale per rating.** A rating without a rationale is not an assessment; it is a mood.

An illustrative entry shape (all values placeholders): `FM-01 · impact: high (customer-facing, regulated product) · likelihood: medium (grounded + reviewed) · residual: medium · rationale: "...".`

### 6.2 What the appetite statement does

A risk appetite statement turns the rating into a decision. In the register it plays three roles: it **bounds** what residual is acceptable for a class of AI risk; it **forces escalation** when a residual breaches the bound; and it **makes "accept" a decision** rather than a default. A register whose entries are all "accepted" without ever testing them against appetite has not used the appetite at all. The appetite machinery — statements, tolerance, threshold breach and escalation — is owned by the ERM guide §6; the register simply carries the pointer and the residual that must be tested against it.

### 6.3 Why "low" with no rationale is the most common entry in a real register

Because "low" is the cheapest way to close a line. Rating an entry "low" ends the conversation: no remediation, no escalation, no follow-up. That is exactly why a real register — especially a first draft — fills with "low" labels that carry no rationale, no owner and no evidence. The label is doing the work that the assessment should have done. The fix is mechanical and it is the one §12 demonstrates: a rating is only accepted when it carries a rationale, and a "low" is re-tested whenever the entry has no artifact behind it. An unrationalised "low" is an observation wearing a rating.

### 6.4 The review cadence

Cadence should follow the risk and the change rate, not the calendar. Four triggers matter more than a fixed date:

- **Tier-based periodic review** — higher-tier systems reviewed more often, per the bank's tiering (owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §7).
- **Change-triggered review** — any change to model, prompt, corpus, tools or use case re-opens the affected entries (FM-06, FM-10, FM-13).
- **Incident-triggered review** — an incident re-rates the entry and its neighbours.
- **Appetite-breach review** — a residual above appetite escalates on the spot, not at the next cycle.

A register with a single annual review date has hidden every failure mode whose owner changed the system in between.

---

## 7. Three Lines and Ownership

### 7.1 The three lines, applied to an AI entry

This guide does not re-explain the three-lines model; that is owned by [banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) §5, with its AI adaptation in [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §5. What matters for the register is the division of labour on a single entry:

| Line | Role on an AI register entry | The register field it touches |
|---|---|---|
| **First line** | Owns the risk and the controls; runs the system day to day; keeps the evidence; proposes the rating | Owner, controls, evidence, proposed residual, treatment |
| **Second line** | Sets the framework; challenges the rating; validates the controls and evidence; owns the appetite test | Challenge of residual, appetite breach, control testing |
| **Third line** | Independently assures the whole: that the controls exist, operate, and are evidenced; that the register is complete | Assurance over entries, completeness, evidence integrity |

The first line owns; the second line assures and challenges; the third line audits. Nothing in AI changes that; what changes is *where the ownership usually lands*.

### 7.2 What the second line validates

The second line's job on an AI entry is not to run the control but to test the claim. Specifically:

- **That the residual rating is proportionate** to the failure mode and its blast radius — the challenge that stops "low" with no rationale (§6.3).
- **That the named controls are real controls**, i.e. that an artifact exists and a test was run (§5.2).
- **That the evidence is current** — the red-team run is not from two model versions ago; the drift dashboard is fed; the corpus-freshness report is dated.
- **That the appetite test was actually applied** and escalation happened on breach.
- **That the entry's owner is the right role** — that a vendor risk is not owned by the model team by default (the gap in §7.4).

The second line for AI control testing plugs into the existing control-testing and RCSA discipline rather than a parallel one ([banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) §6).

### 7.3 What the third line audits

Internal audit's AI work is a completeness-and-evidence audit over the register:

- **Completeness** — are the failure modes that should be there actually there? (The §13 anti-pattern of a register that has never had an entry retired is a completeness tell in the other direction.)
- **Ownership integrity** — does every entry have a named owner who knows they own it?
- **Control operation** — did the controls run in the period, and does the evidence prove it?
- **Evidence integrity** — are the artifacts authentic, retained, and reconcilable to decisions?
- **Register mechanics** — lifecycle followed, ratings rationalised, changes triggering reviews (§11).

Because much of the evidence is machine-readable (logs, dashboards, evaluation outputs), audit increasingly reconciles it directly; the audit-tooling craft is out of scope here and owned elsewhere in the repository.

### 7.4 The AI-specific ownership gap

Here is the failure this section exists to name: **an AI risk entry often lands on the model owner when the risk arises in the integration, the data, or the vendor.**

| If the risk arises in… | …the default (wrong) owner is | …the correct owner is |
|---|---|---|
| Retrieval corpus staleness/conflict (FM-09) | Model owner | Knowledge/RAG custodian, with Data Governance |
| Tool scope or permission (FM-04) | Model owner | Agent/platform engineer with IAM |
| Provider deprecation or change (FM-10) | Model owner | Third-party/vendor risk owner with Procurement |
| Human review that does not review (FM-12) | Model owner | Business process owner with Conduct |
| Concentration/availability (FM-11) | Model owner | Platform owner with Operational Resilience |

The pattern is always the same: the model team is the most technically proximate, so the entry defaults to them, and the actual control lives somewhere they have no authority over. The register's fix is to make the *owner* the role that can actually operate the control, and to make the *model owner* the owner only of the model-soundness modes (FM-06, FM-07 in part). The challenge duty sits with the second line, which should reject any entry whose owner cannot operate its own preventive control.

### 7.5 Escalation

An AI entry escalates when: residual breaches appetite; an evidence artifact is missing past its date; a failure mode is discovered that has no entry; or a control test fails. Escalation paths and the committee lattice are owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §6; the register only flags the trigger.

---

## 8. Third-Party and Supply-Chain Entries

### 8.1 The entries

An AI deployment adds third-party entries the classical vendor register did not have. This section lists them **structurally** — what the entry is and what it must carry. No vendor is named as risky, ranked, or credited here.

| Supply-chain entry | What makes it an AI-specific entry | Owner role |
|---|---|---|
| **Model provider** | The bank does not control the artifact's training, safety behaviour, or upgrade cadence; quality can change under the bank | Third-party risk owner |
| **Platform/service** | The runtime, API, and its defaults change; availability and rate limits bind the business process | AI platform owner |
| **Tool surface** | Third-party tools the agent calls carry their own permissions, terms, and failure modes | Agent/platform engineer |
| **Deprecation** | A model/API version can be retired on the provider's schedule, forcing migration under time pressure | Third-party risk owner with product owner |
| **Version and default change** | A silent default change alters behaviour with no bank-side change record (FM-06, FM-10) | Model owner with third-party risk owner |
| **Terms, data-use and residency** | What the provider may do with inputs; where data is processed; whether inputs train other models | DPO/legal with third-party risk owner |

The *mechanisms* for these are owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §8 and [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §7; the register records the owner, the contractual control, and the evidence.

### 8.2 Procurement questions the register expects answered

The register entry is only as strong as the procurement position behind it. Before a system is registered, the following questions are expected to have answers, each producing an artifact:

1. **Notice** — what and how much notice does the supplier give before a breaking change or deprecation?
2. **Versioning** — can we pin a version, and how long is it supported?
3. **Data handling** — are our inputs retained, used for training, or transferred across borders?
4. **Sub-processors** — who else touches the data, and are they disclosed?
5. **Evaluation access** — may we test the system ourselves and retain the results?
6. **Support and SLA** — what is committed, and what is the remedy on breach?

A "no" to any of these is not disqualifying, but it must be *recorded as a residual* in the entry, not left as a blank.

### 8.3 Exit questions

Every third-party entry carries an exit position, because the alternative is a dependency with no exit that nobody chose:

- **Substitutability** — can the function move to another provider or an in-house model, and at what effort?
- **Data portability** — can the bank's data and configuration be extracted on exit?
- **Abstraction** — is there an interface layer that makes replacement a swap rather than a rebuild?
- **Continuity** — what is the fallback for the period between deciding to exit and exiting?

The exit plan is itself a required artifact for the highest-concentration entries (FM-10, FM-11).

### 8.4 Concentration

Concentration is the supply-chain risk that does not announce itself. It appears when many systems route to the same provider, when one provider's outage stalls a business process, or when the bank's skills concentrate on one stack. The register's treatment is to make concentration visible as an entry in its own right (FM-11) with a named owner and a failover or fallback artifact, rather than to assume it is captured by the sum of individual vendor entries. The structural mitigations — abstraction, fallback, dependency mapping — are recorded against the entry; the mechanisms are owned by [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) §6 and the resilience context by the bank's operational-resilience frame.

---

## 9. The Agentic Extension

### 9.1 What changes when the system acts

A generator produces text; an agent takes actions. That difference adds failure modes with no analogue in the predict-only world, and it changes what a register entry must contain. The agentic guides own the depth — [technology/ai_llm/autonomous_agents_guide.md](../technology/ai_llm/autonomous_agents_guide.md) §1–§6, [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md), [technology/ai_llm/llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md), and [technology/ai_llm/agents_work_fall_apart_guide.md](../technology/ai_llm/agents_work_fall_apart_guide.md) §4–§5 — this section supplies only the register entries.

### 9.2 The agentic failure modes

| ID | Agentic failure mode | Mechanism | Preventive control | Detective control | Evidence | Owner |
|---|---|---|---|---|---|---|
| **AG-01** | Tool misuse | The agent calls the right tool with wrong parameters, or the wrong tool entirely, because the tool's contract was ambiguous or the prompt allowed it | Typed tool schemas; parameter validation; allowlisted tools; dry-run modes | Tool-call audit; anomaly detection on parameters | Tool schema register; call log; incident samples | Agent/platform engineer |
| **AG-02** | Privilege accumulation | Scopes are added over time until the agent holds far more authority than any single task needs | Periodic least-privilege re-baselining; scoped short-lived credentials; deny-by-default | Access reviews; permission-drift detection | Permission matrix history; access-review records | Agent/platform engineer with IAM |
| **AG-03** | Action cascade | A chain of individually-reasonable steps compounds into a large, unintended effect | Step/budget caps; circuit breakers; idempotency; reversibility preference | Run tracing; cascade anomaly thresholds; kill switch | Run traces; cap configuration; postmortems | Product owner (agent) |
| **AG-04** | Approval-boundary failure | The human commit point is placed after the irreversible action, or is bypassable, or is automated away | Commit point before irreversible effects; non-bypassable approval; explicit action inventory | Approval audit; verification that no irreversible action executed unapproved | Approval design; approval log; test evidence | Business process owner |
| **AG-05** | Identity ambiguity | The agent acts under a shared or inherited identity, so its actions cannot be attributed or revoked | Dedicated agent identity; scoped identity per task; revocation capability | Identity/attribution monitoring; orphaned-identity checks | Identity register; attribution logs; revocation records | Platform owner with IAM |

The mechanisms behind AG-02 and AG-05 — scoped identity, no ambient authority, verify-explicitly — are the zero-trust principles developed for the network and the legacy estate in [technology/zero_trust_network_architecture_guide.md](../technology/zero_trust_network_architecture_guide.md) and [technology/zero_trust_legacy_estate_guide.md](../technology/zero_trust_legacy_estate_guide.md); the register applies them at the agent's action boundary.

### 9.3 The entry must name both the agent identity and the human commit point

An agent register entry is incomplete without two fields that a model entry never needed:

1. **The agent's identity** — under what identity does it act, what can that identity reach, and how is it revoked? Without this, AG-02 and AG-05 cannot be controlled or audited.
2. **The human commit point** — which specific actions require an explicit human approval before taking effect, and where in the flow that approval sits. Without this, AG-03 and AG-04 have no anchor.

A register row that says "agent risk: medium" and names neither is the purest form of the thesis's observation: it describes a possibility without an owner or an action boundary, which is exactly the thing that cannot be controlled.

### 9.4 Why the agentic modes rarely appear in a first-draft register

First drafts are usually written by the people who built the *answer-generation* use case; the agentic modes surface later, when autonomy and tools are added. The result is a register that is complete for a chatbot and blind for an agent. The guardrail is procedural: when a system gains the ability to act, the register gains the agentic entries (AG-01…AG-05) in the same change, and the tier/review cadence tightens accordingly. This is the register-level echo of the lifecycle gates owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §7.

---

## 10. The Regulatory Frame

### 10.1 The owner of the map

The regulatory mapping for a bank's AI is **not owned here**. It is owned by [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) **§9 (The Regulatory Mapping for a Bank)** — which maps the global instruments (NIST AI RMF, ISO/IEC 42001, the EU AI Act, the OECD principles), the Singapore instruments, and the banking supervisory stack onto concrete obligations and evidence. The banking requirements map that binds each obligation to a use case is owned by [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md), and the Singapore rulebook by the repository's MAS guide. This section states only **which family of instruments an AI register answers to, and which guide owns the answer** — it asserts no jurisdiction's rule as fact.

### 10.2 The instruments the register answers to

| Instrument family | What it asks of an AI register | Owner of the depth |
|---|---|---|
| **AI risk-management frameworks** (NIST AI RMF and equivalents) | A governed, documented process: identify, measure, manage, and a record of it — the register is a natural evidence artifact | [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §2, §9 |
| **AI management systems** (ISO/IEC 42001 and equivalents) | Documented controls, a statement of applicability, and evidence that controls operate — the register sits inside the management system | Framework guide §2, §9 |
| **Hard AI regulation** (the EU AI Act and equivalents) | Risk-tiered obligations, documentation, human oversight, and — for some categories — registration and conformity evidence | Framework guide §2, §9; requirements map §2 |
| **Banking supervisory expectations** (model-risk and AI guidance) | Inventory, validation, monitoring, and governance extended to AI; the register must reconcile with the model-risk estate | [banking/risk_management_models_guide.md](./risk_management_models_guide.md) §9; [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §3–§4 |
| **Principles and conduct frameworks** (national AI principles, fairness/ethics guidance) | Fairness, accountability, transparency, and human oversight commitments evidenced per use case | [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) §2–§6 |
| **Data-protection law** | Lawful processing, automated-decision safeguards, impact assessment, and data-flow evidence | [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) §5 |

### 10.3 No rule asserted as fact

This guide states no article number, no obligation, no threshold, and no effective date as fact. Where a register entry needs a regulatory anchor, the anchor is looked up in the owning guide's mapped section, which carries its own verification discipline and claims audit. The register's only regulatory claim is structural: an entry that names a control which happens to satisfy a supervisory expectation is stronger when the entry also points at the expectation's owner-mapped section — because it shows the control was designed for a reason, not discovered by accident.

---

## 11. The Register in Operation

### 11.1 The entry lifecycle

A register is a process, and each entry moves through seven stages. Naming them matters because most broken registers fail at a stage nobody named.

| Stage | What happens | The test that it was done |
|---|---|---|
| **Raise** | A failure mode is identified — by design review, red team, incident, or a completeness check against §3 | The entry has an ID and a defined scope |
| **Rate** | Impact, likelihood, and residual are assessed with a rationale (§6) | A rationale line exists; the residual is defensible |
| **Treat** | A decision: accept, mitigate, transfer, avoid, or restrict the use case, with dates | The treatment is explicit, not a default |
| **Own** | A named role is assigned that can operate the controls | The owner is a role, not a committee |
| **Evidence** | Artifacts are attached or the entry is marked "asserted, evidence pending" | At least one artifact resolves, or a dated gap exists |
| **Review** | Re-rated on the cadence and on change triggers (§6.4) | The review date and trigger are recorded |
| **Retire** | The entry is closed because the failure mode no longer applies, with a reason | A retirement reason exists — the register can shrink |

### 11.2 What makes an entry good

An entry is good when it is **specific, owned, evidenced, and honest about residual**. Specific means the failure mode is named concretely enough that the control makes sense. Owned means one role whose authority matches the control. Evidenced means an artifact exists or a dated gap does. Honest means the residual after controls is stated as it is, not as the label the owner wishes it were. A good entry survives a hostile reading; a poor one does not survive a friendly one.

### 11.3 What makes an entry dead

An entry is dead when any of the following is true, and dead entries are the register's real problem because they look like coverage:

- **No owner** (or a committee/team as owner) — the thesis's observation.
- **No evidence** — a control claimed but never demonstrated.
- **A "low" with no rationale** — the cheapest way to close a line (§6.3).
- **A control that is not a control** — a policy statement, an intention, or a monitoring wish described as if it operated.
- **A stale review date** — the entry has not been re-tested since a system change.
- **An owner who cannot operate the control** — the §7.4 mis-assignment.
- **An entry that has never been retired** — not proof of completeness, but a sign the register has no exit and therefore no honesty about what changed.

### 11.4 The quarterly question

Every quarter, the register is asked one question that supersedes the others: **"Which entries have no owner, or no evidence, or a rating with no rationale — and what is the plan and date to close each?"** That single question surfaces most of the dead entries, forces the ownership gaps into the open, and turns the register from a document into a work queue. A register that cannot answer it has become an observation about an observation.

---

## 12. The Cymbal Bank Worked Example

### 12.1 Context

Cymbal Bank — the fictional persona used throughout this repository and the **only** institution in any example here — has deployed *Cymbal Assist*, a customer-facing GenAI assistant grounded in a curated product-knowledge corpus, with a human handoff for transactions and an internal drafting copilot alongside it. The AI governance function asks for the consolidated AI risk register the model-risk register does not provide. The following is **explicitly illustrative and pedagogical**; the entries, owners and ratings are constructed to demonstrate the register's mechanics, not to describe any real institution's register.

### 12.2 The first draft (twelve entries, mostly labels)

The first draft arrives as a twelve-row list. Most rows are labels: a failure mode and a rating, with no owner and no evidence.

| # | Label as written | Rating as written | Owner | Evidence |
|---|---|---|---|---|
| 1 | Hallucination | Medium | — | — |
| 2 | Prompt injection | Medium | — | — |
| 3 | Data leakage | Low | — | — |
| 4 | Model drift | Low | — | — |
| 5 | Vendor change / deprecation | Medium | — | — |
| 6 | Bias | Low | — | — |
| 7 | Overreliance | Medium | — | — |
| 8 | Availability | Medium | — | — |
| 9 | Cost overrun | Low | — | — |
| 10 | Reputational | Medium | — | — |
| 11 | Regulatory change | Medium | — | — |
| 12 | Staff misuse | Low | — | — |

### 12.3 What is wrong with the first draft

Three defects, in rising order of seriousness:

1. **Most rows are observations.** Rows 3, 4, 6, 9 and 12 are rated "low" with no rationale, five of twelve rows are blank-owner, and all twelve are blank-evidence. By the thesis, five of these are not control entries at all.
2. **Row 6 (Bias) is rated low for no reason.** The system is customer-facing and grounded in a corpus built from historical product material; fairness is a first-order concern for a customer-facing assistant across language and dialect communities. "Low" here is the label doing the work the assessment should have done (§6.3) — an unrationalised low on a mode that deserves an explicit rating and a fairness test.
3. **One failure mode is missing entirely: output used outside its validated domain.** Cymbal Assist was validated to answer questions about the product corpus. Nothing in the first draft captures the ordinary slide to *"let's also let it summarise complaint files"* or *"let it draft customer replies"* — the scope creep of use (FM-13) that invalidates a validation which was, at the time, correct. The first draft is complete for a grounded Q&A assistant and blind to what it becomes.

### 12.4 The repaired register (thirteen entries)

The repaired version keeps the twelve labels but converts each into a control entry — owner, control, evidence, residual — and adds the missing thirteenth (scope creep). Ratings that changed are shown with the new rationale; the bias row is the one that moves for a reason.

| # | Failure mode | Owner role | Preventive / detective control | Evidence | Residual (with rationale) |
|---|---|---|---|---|---|
| 1 | Fabrication / unsupported output | AI Product Owner | Grounding-only with mandatory citation / faithfulness evaluation + sampled review | Evaluation report; hallucination-rate baseline | Medium — customer-facing, grounded, still ungrounded failure possible |
| 2 | Prompt / indirect injection | AI Security Lead | Input+output guardrails, instruction/data separation / injection telemetry + red-team regression | Red-team report; guardrail config | High — the assistant reads a corpus that can carry injected content |
| 3 | Data leakage / exfiltration | DPO / InfoSec (joint with platform owner) | Redaction, session isolation, egress control / DLP + output scans | Data-flow diagram; DLP alert log | Low–medium — bounded by egress controls (rationale now written) |
| 4 | Model drift / silent change | Model Owner | Pinned version where offered; change notice / drift dashboard + regression cadence | Version register; drift dashboard | Medium — provider-side change is not fully controllable |
| 5 | Vendor change / deprecation | Third-Party Risk Owner | Contract notice/versioning clauses; exit plan / vendor-change watch | Contract clauses; exit plan | Medium–high — concentration raises it |
| 6 | Bias / unfair outcomes | Model Owner with Fairness/Conduct | Representative corpus; bias-aware review / cohort fairness metrics + complaint analysis | Bias-audit report; fairness metrics | **Medium — re-rated from "low": customer-facing, corpus-derived, cohorts under-served by the source material** |
| 7 | Overreliance / automation bias | Business Process Owner | Human handoff at transactions; challenge prompts / override-rate + QA on handoffs | Oversight design; QA results; override stats | Medium — the human control is itself tested |
| 8 | Concentration / availability | AI Platform Owner | Fallback/degraded mode; rate-limit planning / availability monitoring + failover tests | Dependency register; failover test | Medium — fallback reduces, does not remove, the dependency |
| 9 | Cost overrun / runaway usage | AI Platform Owner | Budget/rate caps; usage alarms / spend telemetry | Cap configuration; spend dashboard | Low — caps bound the exposure (rationale now written) |
| 10 | Reputational / conduct | Business Process Owner with Conduct | Answer-domain bounds; disclosure that the customer is talking to AI / complaint and sentiment monitoring | Disclosure design; complaint log | Medium — conduct risk lands on the assistant's behaviour |
| 11 | Regulatory change | Compliance / AI Governance (second line) | Horizon scanning; mapping to the requirements owner / periodic recertification | Change log; re-assessment records | Low–medium — cadence-bounded (rationale now written) |
| 12 | Staff misuse / unsafe prompts | AI Security Lead | Acceptable-use policy; prompt logging / misuse detection | Policy; misuse alert log | Low — bounded by logging and policy |
| 13 | **Output used outside its validated domain (scope creep)** | **AI Product Owner with AI Governance/Model Risk** | **Registered use-case statement; domain enforcement; change control on any use change / usage-vs-purpose monitoring + recertification** | **Use-case register entry; usage monitoring; re-assessment record** | **Medium — rises fast if a use case is stretched quietly; this is the entry the first draft missed** |

### 12.5 What the repair demonstrates

The repair did not add a framework. It did four mechanical things, each traceable to a section of this guide:

- It **converted labels into entries** by requiring an owner, a control and an evidence artifact for every line — the §5.2 test.
- It **re-rated the bias row** from an unrationalised "low" to a justified "medium" — the §6.3 fix.
- It **added the missing failure mode** (scope creep, FM-13) rather than accepting a register that was complete only for the system's first version — the §11.3 completeness tell.
- It **re-ran the ownership map**, moving vendor risk off the model team (row 5, FM-10) and conduct risk to the business process owner (row 10, FM-12) — the §7.4 gap.

The repaired register is not a promise that Cymbal Assist will not fail. It is the statement that every way it might fail now has a name, a controller, an artifact, and a person. A risk with no owner and no evidence is an observation, not a control.

---

## 13. The Anti-Patterns

Each anti-pattern below is written as symptom, cause, guardrail — because the symptom is what an auditor sees, the cause is what the owner must fix, and the guardrail is what the register process must enforce.

### 13.1 A register of labels with no owners

- **Symptom.** The register has many rows and almost no names; "owner" columns are empty or contain a team or committee.
- **Cause.** The register was built to *show completeness* rather than to *assign accountability*; filling rows is quicker than negotiating ownership.
- **Guardrail.** No entry is accepted without a named role that can operate the preventive control (§3.4, §7.4). A committee may oversee; a role owns.

### 13.2 A control named with no artifact behind it

- **Symptom.** The controls column reads like a policy index — "guardrails", "monitoring", "review" — and no config, report, or log can be produced.
- **Cause.** Controls were copied from a framework or a sibling register without being implemented; the register describes intent, not operation.
- **Guardrail.** The evidence cell must resolve to an artifact or be marked "asserted, evidence pending" with an owner and a date (§5.4). The claim/artifact distinction is the register's central discipline.

### 13.3 An entry owned by the model team for a vendor risk

- **Symptom.** FM-10 (deprecation, provider change) and FM-11 (concentration) are owned by "model risk" or "the AI team."
- **Cause.** Technical proximity: the model team sees the provider most often, so the row defaults to them, even though the control (contract clauses, exit plan, failover) lives with procurement, vendor risk, and the platform.
- **Guardrail.** The §7.4 ownership map. The second line rejects any entry whose owner cannot operate its own preventive control.

### 13.4 "Low" with no rationale

- **Symptom.** A majority of entries are "low," and none explains why; the low-rated set includes customer-facing and fairness modes.
- **Cause.** "Low" closes a line; writing the rationale reopens it. The label substitutes for the assessment.
- **Guardrail.** No rating is accepted without a one-line rationale (§6.1, §6.3); any "low" on an entry with no artifact is re-tested.

### 13.5 A register that has never had an entry retired

- **Symptom.** The register only grows; no entry has ever been closed; the count is presented as evidence of thoroughness.
- **Cause.** Retirement is treated as losing coverage rather than as honest housekeeping; there is no defined close-out step.
- **Guardrail.** The lifecycle includes **retire** with a recorded reason (§11.1). A register that never shrinks is not managing change; it is accumulating it.

### 13.6 An AI register bolted onto the model-risk register without the integration failure modes

- **Symptom.** The AI register is the model-risk register with "GenAI" appended: it holds model-soundness modes (drift, bias, validation) but not the modes that arise in the system around the model (retrieval staleness, tool scope, injection, human review).
- **Cause.** Ownership was granted to the model-risk function, whose register structure was designed for the model, not for the integration, the data, or the vendor (§2.2).
- **Guardrail.** The consolidated inventory (§3) is owned such that integration, data, tool, and human modes are in scope, with their correct owners — not deferred to a model register that structurally cannot hold them.

### 13.7 A register with no cadence and no change trigger

- **Symptom.** Entries have a "last reviewed" date from long ago; system changes (model, prompt, corpus, tools) do not re-open entries.
- **Cause.** Review is treated as an annual calendar event rather than as a function of the system's change rate (§6.4).
- **Guardrail.** Change-triggered and incident-triggered review are written into the lifecycle; a change to any component of the composite version re-opens the affected entries.

### 13.8 Evidence that is stale but present

- **Symptom.** The artifact exists, but it predates the model version now in production; the red-team run is from two versions ago; the fairness metrics predate the corpus refresh.
- **Cause.** "Evidence exists" was treated as the test, rather than "evidence is current to the running version."
- **Guardrail.** The composite system version (§4.7) dates every artifact; evidence older than the running version is a gap, not a control.

### 13.9 The anti-pattern summary

| # | Anti-pattern | Core guardrail | Section |
|---|---|---|---|
| 1 | Labels with no owners | Named role required to open an entry | §3.4, §7.4 |
| 2 | Control with no artifact | Evidence cell must resolve or be a dated gap | §5.4 |
| 3 | Vendor risk owned by model team | Ownership map; reject un-operable owners | §7.4 |
| 4 | "Low" with no rationale | Rationale required; low-with-no-artifact re-tested | §6.3 |
| 5 | Entry never retired | Retire step with recorded reason | §11.1 |
| 6 | Bolted onto model-risk register | Integration modes in scope with correct owners | §2.2, §3 |
| 7 | No cadence / no change trigger | Change- and incident-triggered review | §6.4 |
| 8 | Stale-but-present evidence | Composite version dates every artifact | §4.7 |

---

## 14. The Claims Audit

Status legend: ✅ verified during this pass · ⚠ flagged (verified in part, or a judgment call recorded openly) · ❌ could not be verified. This guide is a repository-internal map, so its auditable claims are mostly of the form *"guide X exists at path Y with section Z"* — verified by inspecting each file during this pass — plus a set of judgment calls about which section best anchors each failure mode. No external regulatory or statistical claim is made, by design.

| # | Claim | Status | How verified / source note |
|---|---|---|---|
| 1 | All nineteen owner guides referenced in §1.3 and §3 exist at the stated paths with the stated order of magnitude of length | ✅ | Direct file inspection during this pass; line counts confirmed (e.g. ai_agent_drift 1624; grc_in_banking 967; rag_conflict_resolution 925; llm_development_risks_security 974) |
| 2 | Regulatory mapping for a bank is owned by ai_governance_framework_guide.md §9 | ✅ | Section heading confirmed: "9. The Regulatory Mapping for a Bank" |
| 3 | Banking AI risk taxonomy is owned by ai_genai_banking_compliance_guide.md §4 | ✅ | Heading confirmed: "4. The Risk Requirements: Model Risk for AI and the GenAI Risk Taxonomy" |
| 4 | Model-risk discipline (SR 11-7) owned by risk_management_models_guide.md §9, ML in risk by §10 | ✅ | Headings confirmed: "9. Model Risk Management: SR 11-7 and the Validation Discipline"; "10. Machine Learning in Risk" |
| 5 | Risk-appetite mechanics owned by enterprise_risk_management_guide.md §6 | ✅ | Heading confirmed: "6. The Risk Appetite: Statements, Tolerance, and the RAG" |
| 6 | Three-lines model owned by enterprise_risk_management_guide.md §5 | ✅ | Heading confirmed: "5. The Risk Governance: Three Lines, Board, CRO, Committees" |
| 7 | Control design/testing and evidence regime owned by grc_in_banking_guide.md §5–§6 | ✅ | Headings confirmed: "5. Control Design and the Control Framework"; "6. Control Testing, RCSA and the Assurance Map" |
| 8 | Fabrication/unsupported output anchored to llm_development_risks_security_guide.md §11 | ⚠ | Guide and §11 exist ("Evaluating Your Security Posture"); §11 is the overreliance/assurance area, a judgment-call anchor rather than a dedicated hallucination clause — the banking requirements map §4 is the stronger primary anchor |
| 9 | Prompt/indirect injection anchored to llm_development_risks_security_guide.md §3 and ai_red_teaming_guide.md §3 | ✅ | Headings confirmed: "3. LLM01: Prompt Injection"; "3. Prompt Injection: Direct & Indirect" |
| 10 | Data leakage anchored to llm_development_risks_security_guide.md §8 | ✅ | Heading confirmed: "8. LLM06: Sensitive Information Disclosure" |
| 11 | Over-permissive tool scope anchored to llm_development_risks_security_guide.md §9–§10 and agent_sandboxing_strategies_guide.md §10 | ✅ | Headings confirmed: "9. LLM07: Insecure Plugin Design"; "10. LLM08: Excessive Agency"; "10. Tool-Execution Patterns" |
| 12 | Autonomy/compounding actions anchored to llm_agents_failures_production_guide.md §1 and autonomous_agents_guide.md §5 | ✅ | Headings confirmed: "1. The Compounding-Error Problem"; "5. Control & Safety" |
| 13 | Drift anchored to ai_agent_drift_guide.md §3, §5 | ✅ | Headings confirmed: "3. Types of AI Agent Drift"; "5. Detecting AI Agent Drift" |
| 14 | Bias/fairness anchored to ai_governance_bias_redteaming_guide.md §2, §4 | ✅ | Headings confirmed: "2. Bias in AI: Types & Sources"; "4. Bias Measurement" |
| 15 | Non-determinism/irreproducibility anchored to ai_verify_toolkit_guide.md §3, §7 and ai_verify_guide.md §5 | ⚠ | Guides and sections exist ("3. The Test Engine"; "7. Outputs and the Report"; "5. The Technical Tests"); the section-to-mode assignment is a judgment call — neither section is titled "non-determinism" |
| 16 | Stale/conflicting knowledge anchored to rag_conflict_resolution_guide.md §5, §7 | ✅ | Headings confirmed: "5. Detecting That a Conflict Exists At All"; "7. Authority and Entitlement — the Enterprise Mechanism" |
| 17 | Third-party/supply-chain change anchored to llm_development_risks_security_guide.md §7 and ai_governance_framework_guide.md §8 | ✅ | Headings confirmed: "7. LLM05: Supply Chain Vulnerabilities"; "8. Data Governance and Third-Party AI Governance" |
| 18 | Concentration/availability anchored to llm_development_risks_security_guide.md §6 and ai_governance_framework_guide.md §8 | ⚠ | §6 exists ("LLM04: Model Denial of Service"); concentration as an enterprise concept is a judgment-call extension of an availability control — the strongest named owner available in this repository |
| 19 | Human factors/automation bias anchored to responsible_ai_frameworks_guide.md §5, §6 | ⚠ | Headings confirmed: "5. The Implementation"; "6. The Banking Angle"; the human-oversight material is distributed rather than a dedicated section — anchor is section-level, not clause-level |
| 20 | Agentic guides and zero-trust guides exist and own the agentic mechanisms | ✅ | Files confirmed (autonomous_agents 924; agent_sandboxing 775; llm_agents_failures 704; agents_work_fall_apart 774; zero_trust_network 701; zero_trust_legacy 907) |
| 21 | llm_development_risks_security_guide.md contains ~22 repository references with a stray '../' prefix that resolve broken | ⚠ | Known repository link-sweep item recorded in the task brief; not re-counted line by line this pass — flagged as a known defect, not a missing guide |
| 22 | No external regulatory rule, threshold, statistic, or rating scheme is asserted as fact in this guide | ✅ | By construction: the rating scheme is labelled illustrative (§6.1); the regulatory section asserts no rule and defers to the owner (§10) |
| 23 | Cymbal Bank is the only institution and the only bank persona in this guide; all register entries are explicit pedagogical constructions | ✅ | By construction — §12.1 states it; no other institution is named |
| 24 | A real bank's AI register contents, rating bands, or thresholds | ❌ | Not public and not asserted; the register here is a structural pattern, and the institution's own taxonomy governs (see §15) |

The audit's honest summary: the *existence* claims are strong (verified by inspection); the *best-anchor* claims are judgment calls flagged ⚠ where a section was the closest available owner rather than a purpose-built one. That is the difference between a verified map and a verified fact — and the register does not need the map to be perfect, only to be honest about which cells are which.

---

## 15. What Could Not Be Verified, and the Glossary

### 15.1 What could not be verified

Honesty flags beat fabrication. The following were not confirmed during this pass and are recorded rather than glossed:

- **The external facts inside the owner guides.** This guide asserts no regulatory rule, date, threshold, or statistic, so it verifies none; the regulatory, framework, bias, drift, and security facts live in the owner guides, each with its own verification ledger and "What Could Not Be Verified" section (for example [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) §12). A reader who needs a clause-level fact must check the owner, not this map.
- **Any real institution's AI register, rating scheme, or thresholds.** These are internal and institution-specific; no bank's register contents were inspected, and none are asserted. The rating scheme in §6.1 is a placeholder and the worked example in §12 is a pedagogical construction. This is a deliberate limit, not a gap: a guide that invented a bank's thresholds would be doing exactly what the hazards forbid.
- **That each "covered by" section is the single best anchor for its failure mode.** The section-to-mode mappings flagged ⚠ in §14 are judgment calls — the closest owner in the repository rather than a purpose-built section. A dedicated hallucination clause or a dedicated concentration section does not exist by name, so the nearest defensible section is cited and flagged.
- **The exact count of broken '../'-prefixed references** in [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md). The task brief records roughly twenty-two; this pass did not re-count them individually. The defect is a known link-sweep item and is not evidence of a missing guide.
- **Anything requiring live web search.** `web_search` was unreliable on this host during this pass; where an external check would have been needed, the fallback was direct inspection of repository files (which is, for a repository-internal map, the correct primary source anyway). Empty search results reflect a tool limitation, not the absence of material.

### 15.2 Glossary

| Term | Meaning in this guide |
|---|---|
| **Agent identity** | The distinct, revocable identity under which an agent acts, so its actions are attributable and its authority can be withdrawn (§9.3). |
| **Appetite** | The amount of a given risk the institution is willing to run; the statement the residual is tested against (§6.2; mechanics owned by the ERM guide §6). |
| **Approval boundary** | The designed point beyond which an action requires an explicit human commit; the agentic register entry names it (§9.2–§9.3). |
| **Artifact** | A concrete record that proves a control operated: a report, log, config, approval, or test result (§1.2). |
| **Automation bias** | The human tendency to accept a fluent machine output without genuine review; a control that becomes a risk (§3 FM-12). |
| **Composite version** | The recorded bundle of model, prompt, corpus, guardrails, tools, and parameters that together define "the system" for change and evidence purposes (§4.7). |
| **Control (preventive / detective)** | An action that reduces the likelihood/impact of a failure mode (preventive) or finds it after the fact (detective) (§1.2). |
| **Drift** | Change in a system's behaviour after deployment, from provider updates, distribution shift, or component change; owned by the drift guide (§3 FM-06). |
| **Entry** | One register row: one failure mode for a defined scope, with all fields populated (§1.2, §11.2). |
| **Evidence** | The artifact an auditor would be shown to prove the control operated (§1.2, §5). |
| **Hallucination** | A fluent but unsupported or false output; the mechanism of FM-01. |
| **Human-in-the-loop (HITL)** | A designed human review or commit point in the workflow; a control only where it is decision-relevant and possible (§4.4). |
| **Injection (direct / indirect)** | Manipulation of a model through crafted input (direct) or through content it retrieves and reads (indirect) (§3 FM-02). |
| **Model risk register** | The register of model-soundness risks owned by the MRM function; this guide's register is deliberately not it (§2). |
| **MRM / SR 11-7** | Model risk management and its lineage of development soundness, independent validation, and governance; discipline owned by [banking/risk_management_models_guide.md](./risk_management_models_guide.md) §9. |
| **Observation** | The thesis's term for a register line with no owner and no evidence: a description of a possibility, not a control. |
| **Owner** | The named role accountable for an entry and able to operate its preventive control (§1.2, §7.4). |
| **RAG** | Retrieval-augmented generation; grounding answers in a curated corpus, which makes the corpus part of the risk surface (§4.2). |
| **Rating (inherent / residual)** | The assessed severity/likelihood before and after controls; the headline is the residual (§6.1, illustrative only). |
| **Residual** | The risk remaining after the named controls; what the register records (§1.2). |
| **Retire** | The lifecycle step that closes an entry with a recorded reason (§11.1). |
| **Register** | The maintained, versioned list of entries for a defined scope (§1.2). |
| **Second line / third line** | The challenge/oversight function and the independent assurance function; the model is owned by the ERM guide §5 (§7.1). |
| **Third-party entry** | A register row for a supply-chain exposure: provider, platform, tool surface, deprecation, version change (§8.1). |
| **Tool scope** | The authority a tool call carries; over-broad scope is FM-04 and the agentic AG-02. |
| **Treatment** | The recorded decision for an entry: accept, mitigate, transfer, avoid, or restrict the use case (§1.2, §11.1). |

---

## 16. The Cross-References, and the Closing Summary

### 16.1 The owner map at a glance

| Failure mode / topic | Owner guide (depth lives here) | Owner section |
|---|---|---|
| Fabrication / unsupported output (FM-01) | [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md); [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) | §4; §11 |
| Prompt / indirect injection (FM-02) | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md); [technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md) | §3; §3 |
| Data leakage / exfiltration (FM-03) | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md) | §8 |
| Over-permissive tool scope (FM-04) | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md); [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md) | §9–§10; §10 |
| Autonomy / compounding actions (FM-05) | [technology/ai_llm/llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md); [technology/ai_llm/autonomous_agents_guide.md](../technology/ai_llm/autonomous_agents_guide.md) | §1; §5 |
| Model drift / silent change (FM-06) | [technology/ai_llm/ai_agent_drift_guide.md](../technology/ai_llm/ai_agent_drift_guide.md) | §3, §5 |
| Bias / unfair outcomes (FM-07) | [technology/ai_llm/ai_governance_bias_redteaming_guide.md](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) | §2, §4 |
| Non-determinism / irreproducibility (FM-08) | [technology/ai_llm/ai_verify_toolkit_guide.md](../technology/ai_llm/ai_verify_toolkit_guide.md); [technology/ai_verify_guide.md](../technology/ai_verify_guide.md) | §3, §7; §5 |
| Stale / conflicting knowledge (FM-09) | [technology/ai_llm/rag/rag_conflict_resolution_guide.md](../technology/ai_llm/rag/rag_conflict_resolution_guide.md) | §5, §7 |
| Third-party / supply-chain change (FM-10) | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md); [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) | §7; §8 |
| Concentration / availability (FM-11) | [technology/llm_development_risks_security_guide.md](../technology/llm_development_risks_security_guide.md); [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) | §6; §8 |
| Human factors / automation bias (FM-12) | [technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) | §5, §6 |
| Use outside validated domain (FM-13) | [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md); [banking/ai_genai_banking_compliance_guide.md](./ai_genai_banking_compliance_guide.md) | §7; §4 |
| Rating, appetite, risk process | [banking/enterprise_risk_management_guide.md](./enterprise_risk_management_guide.md) | §6–§7 |
| Evidence regime, controls, KRIs, board reporting | [banking/grc_in_banking_guide.md](./grc_in_banking_guide.md) | §5–§6, §9, §11 |
| Model-risk discipline | [banking/risk_management_models_guide.md](./risk_management_models_guide.md) | §9–§10 |
| Framework landscape, three lines for AI, gates, regulatory map | [technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) | §2–§7, §9 |
| Agentic failure mechanics | [technology/ai_llm/autonomous_agents_guide.md](../technology/ai_llm/autonomous_agents_guide.md); [technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md); [technology/ai_llm/agents_work_fall_apart_guide.md](../technology/ai_llm/agents_work_fall_apart_guide.md) | §5; passim; §4–§5 |
| Scoped identity / action-boundary principles | [technology/zero_trust_network_architecture_guide.md](../technology/zero_trust_network_architecture_guide.md); [technology/zero_trust_legacy_estate_guide.md](../technology/zero_trust_legacy_estate_guide.md) | §4; §2 |

### 16.2 Closing summary

A bank does not lack frameworks for AI. It lacks one page where each way AI fails stands next to the control that stops it, the check that finds it, the artifact that proves it, and the person who owns it. This guide is that page, and only that page — a consolidated register assembled from work the repository already owns, refusing to re-derive the frameworks, the taxonomy, the model-risk discipline, or the regulatory map that its sibling guides own in depth.

The register's value is not the table. It is the discipline the table enforces: a failure mode is not covered until a named role can operate a control over it and produce the artifact that proves the control ran. That is the whole test, and it is the difference between a document that reassures and a document that governs. Every entry in this guide is one line of that test; every anti-pattern is a way to fail it; every worked example is the test applied. A register assembled this way will still miss things, and its honesty about what it cannot verify is the price of being trusted where it can.

Which leaves the sentence the register was built to make true, and the one it returns to:

a risk with no owner and no evidence is an observation, not a control.
