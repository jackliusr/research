# Agentic GRC: Governing Agents Through the Framework They Execute — A Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Series:** Banking & Financial Technology Guides — the governance, risk and compliance cluster
> **Audience:** Solution architects, GRC platform and control-automation engineers, risk and compliance professionals, internal-audit and assurance practitioners, and AI-governance leads who are being asked to let an agent perform a governance, risk or compliance activity — and who must be able to answer for it afterwards.

*The guide for the moment a bank asks an agent to do governance work. It does not re-explain GRC, and it does not re-explain the governance layer above the models. The shared GRC machinery — the three lines, the compliance operating model, regulatory change, control design and testing, the assurance map, issue and remediation, the KRI framework, the platform layer, board reporting, the adjacent governance domains, and individual accountability — is owned by [Governance, Risk and Compliance (GRC) in Banking: The Integrated Discipline](grc_in_banking_guide.md). The instrument landscape and the AI operating model are owned by [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md). What this guide owns is exactly one subject: **what changes when the performer of a GRC activity is not a person but an agent, and what a bank must do so that the framework can still attribute an opinion to someone.***

**Series / boundary declared by name.** Across this repository of 665 Markdown guides, the phrase **"agentic GRC" appears in zero files**, and a case-insensitive search of [grc_in_banking_guide.md](grc_in_banking_guide.md) (967 lines) for `agentic`, `AI agent` and `autonomous` returns **zero hits** — the discipline's own guide contains no agent material at all. Meanwhile "agentic" appears in 161 files, and that shelf owns **building and operating** agents, not **governing them through a GRC framework**. The two disciplines are unconnected in this repository, and no guide occupies their intersection. That gap is this guide's reason to exist. The demarcation against the nearest neighbour is one sentence: **[AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) governs the *model estate* — a population of models, their risk classification, their lifecycle gates, their committee lattice; this guide governs the *agent estate* — a population of actors that take actions through tools, persist across sessions, and are delegated authority.** A model predicts. An agent acts. The governance question is not the same question.

**What this guide does NOT do.** It does not re-derive the three lines, the control vocabulary (preventive / detective / corrective), the RCSA, the assurance map, the control library, the issue lifecycle, the KRI *framework*, the GRC platform layer, or individual accountability — [grc_in_banking_guide.md](grc_in_banking_guide.md) owns all of it, and §7's independence material and §6's testing-and-assurance material are cited here rather than restated. It does not re-derive any AI instrument: **every named framework, principle or regulator publication below is either verified at its issuing body's own material during this pass, or cross-referenced to [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) §2–§3, which owns the landscape.** It does not re-explain the isolation mechanics of a sandbox ([Agent Sandboxing Strategies](../technology/ai_llm/agent_sandboxing_strategies_guide.md)), the mechanics of red-teaming ([AI Red Teaming](../technology/ai_llm/ai_red_teaming_guide.md)), the mechanics of evaluation ([LLM Evaluation Frameworks](../technology/ai_llm/llm_evaluation_frameworks_guide.md)), the mechanics of version pinning ([Agent Versioning](../technology/ai_llm/agent_versioning_guide.md)), or the mechanics of an agent platform ([Enterprise Agentic Platform Architecture](../technology/ai_llm/enterprise_agentic_platform_architecture_guide.md)). It does not re-explain the postures that tell an agent when to ask, recommend, act or wait — [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) owns those, and this guide **consumes** them. It does not re-derive the non-human-identity register discipline — [Non-Human Identity](../technology/non_human_identity_guide.md) owns it, and this guide's agent register cites it.

**Jurisdiction and sourcing constraint.** This guide describes the regulatory relationship **structurally** — what a supervisor asks about an automated performer, and why the question is different in kind — and prints **no penalty amount, no fine, no market size, no vendor share, and no adoption statistic for the agentic-GRC product category**, and no unsourced rule, deadline or threshold. A named instrument appears below **only** where this research pass verified it at the issuing body's own material with a date; where verification was not obtained, the text describes the structure and defers to the sibling that owns the instrument. The instruments verified this pass are: the IMDA/PDPC **Model AI Governance Framework for Traditional AI** (first edition 23 January 2019; second edition 21 January 2020), the **Model AI Governance Framework for Generative AI** (released 30 May 2024, nine dimensions), the **Model AI Governance Framework for Agentic AI** (launched 22 January 2026 at the World Economic Forum, four dimensions), and the MAS **FEAT Principles** (12 November 2018) and **Safeguards for Agentic Finance at Runtime (SAFR)** information paper (3 July 2026). Everything else is labelled practice description or argument. **Every institution in every worked example below is Cymbal Bank, a fictional bank; no real bank's internal arrangements are described anywhere in this guide.** No vendor is ranked, sized, compared or recommended.

**Cross-references used throughout:** [grc_in_banking_guide.md](grc_in_banking_guide.md) (the GRC machinery), [../technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) (the instrument landscape and the AI operating model), [../technology/ai_llm/agent_decision_policy_guide.md](../technology/ai_llm/agent_decision_policy_guide.md) (the four postures and the action-class table), [../technology/ai_llm/agentic_engineering_guide.md](../technology/ai_llm/agentic_engineering_guide.md) (the engineering lifecycle), [../technology/non_human_identity_guide.md](../technology/non_human_identity_guide.md) (the register and lifecycle discipline), [ai_risk_register_guide.md](ai_risk_register_guide.md) (failure modes, controls, and the evidence test), [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) and [../technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) (the compliance overlay), [../technology/ai_llm/agent_versioning_guide.md](../technology/ai_llm/agent_versioning_guide.md) and [../technology/ai_llm/agentops_guide.md](../technology/ai_llm/agentops_guide.md) (versioning and observability), [../technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md), [../technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md), [../technology/ai_llm/llm_evaluation_frameworks_guide.md](../technology/ai_llm/llm_evaluation_frameworks_guide.md) and [../technology/ai_llm/llm_evaluation_vs_validation_guide.md](../technology/ai_llm/llm_evaluation_vs_validation_guide.md) (the method guides), [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) (§5 owns the three-lines model itself), [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md), and [regtech_guide.md](regtech_guide.md).

**Status legend (used in the claims audit and throughout):** ✅ verified at a primary source during this pass · ⚠ partially verified (secondary source, or primary source confirms only part) · ❌ could not be verified.

### Reading paths

- **Solution architects and GRC platform engineers** — §4 (the register fields), §8 (what evidence must carry, the sharpest engineering section), §11 (what a GRC platform must represent), §12 (the failure modes that your data model causes).
- **Risk and compliance professionals** — §3 (the non-determinism problem restated for a control tester), §6 (the activity rulings — what an agent may and may not do), §7 (the paradox and independence), §10 (what a supervisor asks).
- **Internal audit and assurance practitioners** — §7 (attributing an opinion to a person), §8 (sampling versus re-performance), §9 (testing a control that behaves differently each time), §15 (the claims audit).
- **AI-governance leads and second-line leadership** — §1 (the boundary), §5 (the autonomy question and where the line sits), §13 (the worked example where the loop closes), §14 (the anti-patterns checklist).
- **General readers** — §1, §2, §7, §13, §16 (the thesis, the paradox, the worked example, the closing summary).

Each section stands alone; the tables are the quick reference, and the cross-references carry the reader to the sibling guide that owns the underlying detail.

---

### Table of Contents

1. [Overview, Decoder, Thesis, and the Boundary Declared by Name](#1-overview-decoder-thesis-and-the-boundary-declared-by-name)
2. [What Agentic Changes for Governance](#2-what-agentic-changes-for-governance)
3. [The GRC Problem Restated for a Non-Deterministic Actor](#3-the-grc-problem-restated-for-a-non-deterministic-actor)
4. [Governing the Agent Estate: The Agent Register](#4-governing-the-agent-estate-the-agent-register)
5. [The Autonomy Question](#5-the-autonomy-question)
6. [Agents Doing GRC: The Activity Rulings](#6-agents-doing-grc-the-activity-rulings)
7. [The Loop — The Agentic-GRC Paradox](#7-the-loop--the-agentic-grc-paradox)
8. [Evidence and the Audit Trail](#8-evidence-and-the-audit-trail)
9. [The Control Framework Adapted](#9-the-control-framework-adapted)
10. [The Regulatory View — Structural Only](#10-the-regulatory-view--structural-only)
11. [The Platform Question](#11-the-platform-question)
12. [The Failure Modes](#12-the-failure-modes)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. Overview, Decoder, Thesis, and the Boundary Declared by Name

### 1.1 The decoder

The phrase "agentic GRC" is new, so the decoder fixes the terms before anything else. Each term is used in one sense throughout this guide and no other.

| Term | What it means in this guide | What it deliberately does not mean |
|---|---|---|
| **Agentic GRC** | The practice of letting an **agent** perform, assist or support a governance, risk or compliance activity — and governing that agent **through the same framework it is executing**. | Not a product category, not a vendor segment, not a maturity label. |
| **The agent** | A software actor that **takes actions through tools**, holds state across steps, and is **delegated authority** by a principal. Full engineering definition: [Agentic Engineering](../technology/ai_llm/agentic_engineering_guide.md). | Not a model. A model predicts; an agent acts. |
| **The agent register** | The authoritative list of agents in the estate and the attributes that make each governable — owner, purpose, autonomy level, tools, data reach, model dependencies, version. Model register discipline lives in [Non-Human Identity](../technology/non_human_identity_guide.md) §3–§4; the AI risk register in [ai_risk_register_guide.md](ai_risk_register_guide.md). | Not a model inventory, not a vendor catalogue, not a prompt library. |
| **The autonomy level** | The standing authority an agent holds — which classes of action it may **ASK**, **RECOMMEND**, **ACT** on, or **WAIT** through. Defined only by [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §2 and §9.4, and **consumed** here by reference. | Not a confidence score, not a capability rating, not a model benchmark result. |
| **The guardrail** | A constraint that **prevents** an inadmissible action before it occurs — a permission boundary, a scope limit, a whitelist, an approval gate. Preventive control, in the sense of [grc_in_banking_guide.md](grc_in_banking_guide.md) §5. | Not a prompt instruction on its own; a prompt line is a request, not a wall. |
| **The tool call** | The single unit of an agent's **effect on the world** — an API invocation, a query, a write, a payment instruction, a ticket update. Each is the atomic event an audit trail must capture. | Not a "step" in a reasoning trace; a trace records thinking, a tool call records consequence. |
| **The evidence artefact** | The durable record that lets a third party **re-establish that a control operated** — what was tested, against what, by which version, with what result. The test of control: [ai_risk_register_guide.md](ai_risk_register_guide.md) §5. | Not a log file alone; a log is raw material, an artefact answers a question. |
| **The attestation** | A **named human's** statement of opinion that a control operated or an obligation was met, made with personal accountability attached. Owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §13. | Not an agent's output. An agent may **assemble** attestation evidence; it cannot **make** the attestation. |
| **The obligation** | A duty the institution owes to a regulator, a customer or a counterparty, expressed as a rule, a principle or a supervisory expectation. Regulatory-change plumbing: [grc_in_banking_guide.md](grc_in_banking_guide.md) §4. | Not a task, not a ticket, not a control. Obligations are owed; controls discharge them. |
| **The control owner** | The **named person** accountable for a control's design, operation and evidence. [grc_in_banking_guide.md](grc_in_banking_guide.md) §5–§6. | Not the agent, even when the agent is the performer. The performer and the owner are different roles. |

Two of those terms carry most of the guide's weight. **The agent register** is where an actor becomes enumerable (§4). **The autonomy level** is what a control can *assert* (§5). Everything else follows from them.

### 1.2 The thesis, stated once

**An agent that tests a control is itself a control that needs testing.** The moment an agent performs a GRC activity — testing a control, monitoring an obligation, collecting evidence, watching an indicator — it stops being a tool used *on* the framework and becomes an actor *inside* it. The independence question, the evidence question and the assurance question that apply to any control now apply to the agent. It must be registered, owned, scoped, versioned, tested, evidenced and attested like the control it operates. It cannot be exempted from the framework because it happens to be delivering the framework.

This is not a slogan and it is not universally flattering to the technology. The guide tests the thesis against the material and reports where the evidence complicates it: §3 shows why a control that is only *sampled* weakens the re-performance tradition that assurance rests on; §7 shows that the independence question has a genuinely unresolved case (an agent performing a **second-line** test while owned by the **first line**); §8 shows that the audit trail for a non-deterministic performer is **weaker** than the deterministic ideal that a control tester was once used to; and §16 records what could not be verified. The thesis survives those tests, but in a narrower form than a vendor would write it.

### 1.3 The boundary declared by name — which layer each guide owns

Six large guides sit adjacent to this one and none of them owns the intersection. The table below states the ownership, so this guide neither duplicates nor contradicts any of them.

| Layer | Owner guide (real line count) | What it owns | What this guide takes from it |
|---|---|---|---|
| **The GRC machinery** | [grc_in_banking_guide.md](grc_in_banking_guide.md) (967) §2–§16 | The three lines (§2), compliance operating model (§3), regulatory change (§4), control design (§5), testing/RCSA/assurance map (§6), third line (§7), issue/remediation (§8), KRI (§9), platform layer incl. §10.5 no-vendor constraint (§10), board reporting (§11), adjacent domains incl. §12.2 AI governance (§12), individual accountability incl. the attestation (§13) | The vocabulary and the frame. **No restatement.** §7 of this guide cites its §2/§6 independence material; §11 cites its §10. |
| **The governance layer above the models** | [../technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) (707) §2–§9 | The instrument landscape (§2), the Singapore landscape (§3), the comparison and selection method (§4), three lines adapted to AI (§5), model-risk integration and the committee lattice (§6), the AI lifecycle gates (§7), data and third-party AI governance (§8), the regulatory mapping for a bank (§9) | **Every instrument reference.** Its subject is the **model estate**; this guide's subject is the **agent estate**. |
| **The engineering discipline** | [../technology/ai_llm/agentic_engineering_guide.md](../technology/ai_llm/agentic_engineering_guide.md) (898) | Framing, architecture, build, **evaluation (§8)**, release, **operations (§10)**, retirement, craft practices, roles, maturity, open problems | The eval and operations phases as the points at which governance attaches. Cited, not restated. |
| **When an agent should ask, recommend, act or wait** | [../technology/ai_llm/agent_decision_policy_guide.md](../technology/ai_llm/agent_decision_policy_guide.md) (824) §2, §9.2, §9.4, §9.5 | The four postures (§2), the four-eyes control when one eye is an agent (§9.2), the action-class→posture table with "the class, not the confidence, is the primary key" (§9.4), what the record must show (§9.5) | **The autonomy ladder**, consumed by reference. §5 of this guide is built on it and does not restate it. |
| **The AI risk register** | [ai_risk_register_guide.md](ai_risk_register_guide.md) (816) §3–§7 | The failure-mode inventory (§3), the controls that do the work — input/output, retrieval, **tool-scope and permission design**, HITL placement, **logging that preserves evidence (§4.5)**, evaluation as a gate, versioning (§4), **evidence and the audit question (§5)**, rating and appetite (§6), three lines and ownership (§7) | The failure-mode taxonomy and the artefact test: *a claim of control without an artifact is an assertion.* |
| **The compliance overlay and the responsible-AI frames** | [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) (717), [../technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md) (706) | The banking compliance overlay for GenAI; the responsible-AI framework families | The compliance vocabulary when a GRC activity touches GenAI. |

Two further guides are load-bearing and cross-referenced rather than summarised: [../technology/non_human_identity_guide.md](../technology/non_human_identity_guide.md) (804) — an agent **is** a non-human identity, and its §3 inventory discipline ("you cannot govern what you cannot enumerate"), §4 lifecycle and the orphan, §7.2 registration-before-issuance, §7.4 segregation of duties, §7.6 the audit question, and §8 (the agent as an NHI, and the four assumptions it breaks) are the register half of this guide; and [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) (725) §5, which owns the three-lines model itself. Supporting: [../technology/ai_llm/agent_versioning_guide.md](../technology/ai_llm/agent_versioning_guide.md) (744), [../technology/ai_llm/agentops_guide.md](../technology/ai_llm/agentops_guide.md) (783), [../technology/ai_llm/ai_agent_drift_guide.md](../technology/ai_llm/ai_agent_drift_guide.md) (1624), [../technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md) (775), [../technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md) (709), [../technology/ai_llm/llm_evaluation_frameworks_guide.md](../technology/ai_llm/llm_evaluation_frameworks_guide.md) (816), [../technology/ai_llm/llm_evaluation_vs_validation_guide.md](../technology/ai_llm/llm_evaluation_vs_validation_guide.md) (687), [../technology/ai_llm/agent_harness_engineering_guide.md](../technology/ai_llm/agent_harness_engineering_guide.md) (777), [../technology/ai_llm/multi_agent_banking_guide.md](../technology/ai_llm/multi_agent_banking_guide.md) (802), [../technology/ai_llm/enterprise_agentic_platform_architecture_guide.md](../technology/ai_llm/enterprise_agentic_platform_architecture_guide.md) (833), [regtech_guide.md](regtech_guide.md) (747), and [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md) (717).

### 1.4 The three honest positions this guide holds

The genre invites two opposite fabrications. This guide commits to neither.

**Position one — not replacement.** No claim is printed here that an agent replaces the second line, the control tester, or the compliance officer, and none is printed that it lifts their productivity by a figure, **unless a dated source is attributed and examined rather than adopted**. Where a labour or productivity claim appears in the literature ([Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §5 records the interruption-economics work this field relies on), it is attributed to that guide and treated as a claim to examine, not a result to quote as this guide's own.

**Position two — not dismissal.** Reflexive scepticism is equally unearned. Agents genuinely can **find, draft, sample, monitor and assemble** at a scale and consistency no human review team matches, and a bank that refuses the tooling on principle will simply be doing worse governance more slowly.

**Position three — the discipline that survives both.** Because judgement, attestation and accountability cannot be delegated to a non-person, **an agent may find, draft, sample, monitor and assemble; it must not conclude, attest, approve or decide; and a named human remains accountable for each.** A framework that cannot attribute an opinion to a person cannot be audited. Every activity in §6 is ruled against exactly this line.

### 1.5 How this guide is organised

§2 is the foundation — what changes when the governed subject becomes an actor rather than a model. §3 restates the GRC problem for a performer that is non-deterministic. §4 builds the agent register. §5 consumes the autonomy ladder. §6 is the real subject: the activity rulings. §7 is the central section, the paradox and the loop. §8 is evidence and the audit trail. §9 adapts the control framework. §10 is the regulatory view, structural only. §11 is the platform question. §12 is the failure modes. §13 is the Cymbal Bank worked example. §14 is the anti-patterns. §15 is the claims audit. §16 records what could not be verified, holds the glossary and the cross-references, and closes.

## 2. What Agentic Changes for Governance

### 2.1 The one-line foundation

**A model predicts; an agent acts.** Every governance difference in this guide descends from that sentence. The model estate is governed as a population of *estimators* — things that emit outputs, whose risk is that the output is wrong, whose controls are accuracy and robustness, and whose lifecycle is training, validation, deployment, monitoring. The agent estate is governed as a population of *actors* — things that **do**, whose risk is that the action is unauthorised, erroneous or unaccountable, whose controls are permission, scope and evidence, and whose lifecycle is commissioning, delegation, operation, review and revocation. [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) owns the model-estate frame; this guide owns the actor frame. Confusing the two is the first failure mode a bank commits when it puts an agent into a compliance process and tries to govern it with a model card.

### 2.2 The four properties that make an agent a different governance subject

**Property one — it acts through tools.** An agent's effect on the institution leaves the model through a **tool call**: an API invocation, a database write, a payment instruction, a record update, a ticket creation. This is the decisive property. A model output that is merely *read* by a human is an input to a decision; an agent's tool call *is* a decision executed. The governance consequence is that the unit of control is no longer the output but the **permission** — what tools the agent may call, on what data, under whose authority. Tool-scope and permission design is named as a control that does the work in [ai_risk_register_guide.md](ai_risk_register_guide.md) §4.3; in the agent estate it is not one control among several, it is **the** control.

**Property two — it persists.** A model is stateless between invocations: the same input yields the same distribution of outputs, and nothing carries over. An agent holds state — memory, a task queue, a session, a plan that survives across steps and sometimes across days. Persistence is what turns a sequence of individually-defensible actions into a **path** that no single approval covered. The governance consequence is that an agent needs a lifecycle (commissioning, review, revocation) and not merely a release (version, rollback, deployment), because the thing that must be governed is what it accumulated, not just what it was built to do.

**Property three — it carries delegated authority.** An agent does not act on its own behalf. It acts under authority delegated by a principal — a business function, a control owner, an accountable executive — and that delegation is exactly what [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §9 governs: *who may commit the institution to what*. Delegated authority is what makes an agent a **control risk** rather than only a **technology risk**: an actor holding delegated authority inherits the segregation-of-duties, approval-threshold and mandate questions that a human delegate would.

**Property four — it is non-deterministic in operation.** Even with a fixed model version, an agent's sequence of tool calls can differ between runs on the same input, because sampling, retrieval and environment state vary. This property is treated on its own in §3, because it is the property that breaks the assurance tradition most directly.

### 2.3 The model-estate questions and the agent-estate questions, side by side

The point of the contrast is not that the model questions are wrong — they are right, and the sibling guide owns them. The point is that they are **not sufficient** for an actor, and a bank that only asks them has a gap it cannot see.

| Governance question | In the **model estate** ([AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md)) | In the **agent estate** (this guide) |
|---|---|---|
| What is the governed object? | A **model** — an estimator with a version and a training lineage. | An **actor** — a principal, a set of tools, a delegation and a state. |
| What is the unit of risk? | The **output** — is it accurate, fair, robust, safe? | The **action** — was it authorised, scoped, evidenced and attributable? |
| What is the primary control? | Validation, evaluation, monitoring for drift. | **Permission and scope** — what the agent may do at all. |
| What is the lifecycle? | Train → validate → deploy → monitor. | **Commission → delegate → operate → review → revoke.** |
| Who is accountable? | The model owner, for the model's fitness. | The **delegating principal** and the **control owner**, for what the agent did. |
| What does an incident look like? | A wrong output consumed downstream. | An **unauthorised or erroneous action already taken** — a state that must be unwound. |
| Which sibling owns the frame? | [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) §5–§7. | This guide, §4–§9. |

The last row matters. The two estates are not a hierarchy; they are **two governed populations that can share a register and must not share a risk model.** An agent that emits a wrong recommendation and an agent that executes a wrong payment are both "AI risk" in a taxonomy, and are entirely different problems at the control level.

### 2.4 What does *not* change

Three things are unchanged, and saying so prevents the guide from over-claiming.

- **The three lines still hold.** An agent performing a GRC activity does not create a fourth line, and does not dissolve the first. [grc_in_banking_guide.md](grc_in_banking_guide.md) §2 owns the working relationship; [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §5 owns the model itself. An agent is an actor *within* a line, not a line.
- **The control vocabulary still holds.** Preventive, detective and corrective; design versus operating effectiveness; the RCSA and the assurance map — [grc_in_banking_guide.md](grc_in_banking_guide.md) §5–§6 owns all of it. §9 of this guide adapts the vocabulary; it does not replace it.
- **Accountability still lands on a person.** The attestation is a named human's opinion ([grc_in_banking_guide.md](grc_in_banking_guide.md) §13). An agent can produce the evidence behind an attestation; it cannot become the attester.

The claim of this guide is therefore narrow and defensible: agentic governance does not need a **new** framework — it needs the existing framework **extended to a new class of performer**, with the register, the autonomy ladder and the evidence rules that extension requires.

---

## 3. The GRC Problem Restated for a Non-Deterministic Actor

### 3.1 The guarantee a control used to carry

A traditional control, tested the traditional way, carried an implicit guarantee: **if it passed twice, understanding why it passed let you predict that it passes the third time.** That is the deep unstated assumption behind re-performance, behind walkthroughs, and behind the confidence an auditor places in a sample. A deterministic control — an access rule, a reconciliation, a four-eyes approval step — is a **mechanism**, and a mechanism can be inspected to the point where its behaviour is known. [grc_in_banking_guide.md](grc_in_banking_guide.md) §6.3 records the sampling-and-evidence question for human-operated controls; it assumes a mechanism underneath.

An agent's control is not a mechanism. It is a **policy enacted by a probabilistic actor**. Pass it twice and you have observed two successful runs, not a guarantee about the third. The pass tells you something — it is not nothing — but what it tells you is about the *distribution* of runs, not about the next one. The guarantee has been replaced by a **rate**.

### 3.2 What follows from that, stated plainly

1. **Re-performance stops being available.** A human control tester can perform the control themselves and compare. An agent's control cannot be re-performed to a single expected result, because the agent will take a different path. What remains is **sampling and evaluation** — run it many times, against a designed set, and characterise the outcome distribution (§8).
2. **A pass rate is not a control conclusion.** "The agent flagged 96 of 100 seeded issues" is a *measurement*. It becomes a control conclusion only when someone sets the threshold, accepts the residual, and signs (§7).
3. **The evidence burden moves.** With a mechanism, the evidence is the control's configuration. With an actor, the evidence is the **run** — the version, the inputs, the tools called, the outcome, and the human who accepted it (§8).
4. **The residual is explicit.** A deterministic control is either operating or not. A non-deterministic one can be *operating within a stated error rate*, which means someone must state the error rate and own it. This is why §6 forces every activity to name who remains accountable: the error rate has to land on a person.

### 3.3 The trap: reading a successful evaluation as governance

The most common failure is to confuse an **evaluation** with a **governance conclusion**. An evaluation is a measurement of the performer against a designed set, produced by the engineering or assurance function ([Agentic Engineering](../technology/ai_llm/agentic_engineering_guide.md) §8; [LLM Evaluation Frameworks](../technology/ai_llm/llm_evaluation_frameworks_guide.md)). A governance conclusion is a **decision by an accountable person** that the control may be relied upon, at a stated standard, for a stated population, until a stated review date.

The trap has a signature: the evaluation is green, the tracker shows the agent live, and no one has ever written down *what the green number entitles the bank to rely on*. [ai_risk_register_guide.md](ai_risk_register_guide.md) §4.6 names the correct framing — **evaluation as a gate, not a report** — and this guide extends it: an evaluation that no one has converted into a reliance decision is a report wearing a gate's clothes.

### 3.4 What the evidence of an agent's control actually is

Three candidate answers, and only the third is defensible.

| Candidate evidence | Why it is not sufficient alone |
|---|---|
| **The agent's own log line** — "control tested, result: PASS" | Self-reported. An actor asserting its own success is [ai_risk_register_guide.md](ai_risk_register_guide.md) §5's assertion, not evidence. |
| **The evaluation run** — a benchmark score at release | Point-in-time; says nothing about the production run, the inputs, or the version in force when it mattered. |
| **The assembled run artefact** — version + inputs + tool calls + outcome + the human acceptance | The only form that lets a third party **re-establish** that the control operated. This is §8's subject, and it is what the register's fields in §4 exist to make producible. |

### 3.5 Attestation when the performer is not a person

The attestation is a named human's opinion, made with personal accountability attached ([grc_in_banking_guide.md](grc_in_banking_guide.md) §13). An agent cannot make one — not because it is untrustworthy, but because **an opinion is only auditable if a person holds it.** The question this raises is the one the rest of the guide answers: *what exactly does a person attest to when an agent did the work?*

The defensible answer, developed in §7: the person attests to the **design and the reliance decision** — that the agent's control was fit for the purpose, that its evidence was examined, that its residual error was accepted within appetite, and that its version and scope were the ones approved. The person does **not** attest to having personally performed steps the agent performed, and a framework that pretends otherwise is asking for a signature it cannot support. Getting this distinction wrong is anti-pattern 6 in §14.

## 4. Governing the Agent Estate: The Agent Register

### 4.1 The precondition, borrowed not restated

You cannot govern what you cannot enumerate. That sentence is [Non-Human Identity](../technology/non_human_identity_guide.md) §3.4's central finding, established for the whole non-human-identity estate, and §3.1–§3.3 of that guide explains why the estate grows faster than the controls that would cover it. This guide does not re-derive any of it. What this guide adds is the specific claim that an **agent** is the hardest member of that estate to enumerate, because [Non-Human Identity](../technology/non_human_identity_guide.md) §8.2 lists four assumptions an agent breaks that a service account does not — and an agent doing GRC work adds a fifth: it *acts inside the control framework it is meant to be governed by*, so a gap in the register is also a gap in the assurance map.

### 4.2 The register's real job

The register is not a spreadsheet of names. Its job is to answer, for any agent in the estate, **four questions a control owner or auditor will actually ask**:

1. **Who is accountable for this agent?** — a named person, not a team, not a vendor.
2. **What is it allowed to do, and under whose authority?** — the delegated mandate and the tool scope.
3. **What exactly was running when the thing in question happened?** — version, model pin, prompt set, tool set.
4. **What did it touch?** — the data reach that determines the blast radius of a failure.

A register that cannot answer these four on demand is a list, not a control. [Non-Human Identity](../technology/non_human_identity_guide.md) §7.2's rule — **registration before issuance** — applies here in the same form: no agent enters the estate, and no agent's authority is expanded, without a register entry that already answers the four questions.

### 4.3 The register fields

The following fields are the minimum. They are chosen so that §8's evidence requirements and §11's platform requirements are *satisfiable* — an evidence artefact can only carry version, tools and inputs if the register already recorded them as the agent's identity.

| Field | What it records | Why the register needs it | Sibling that owns the detail |
|---|---|---|---|
| **Agent ID** | A stable, unique identifier | The join key for every log, ticket and evidence artefact | [Non-Human Identity](../technology/non_human_identity_guide.md) §7 |
| **Name / purpose** | What the agent is for, in one line | The "purpose limitation" the register enforces; an agent used off-purpose is an unregistered agent | — |
| **Accountable owner** | A **named person**, with a deputy | The accountability that §7 must be able to land; the orphan is [Non-Human Identity](../technology/non_human_identity_guide.md) §4.2's failure case | [Non-Human Identity](../technology/non_human_identity_guide.md) §7.1 |
| **Business / control owner** | The function whose control the agent operates | Separates the performer from the control owner (§7) | [grc_in_banking_guide.md](grc_in_banking_guide.md) §5–§6 |
| **Autonomy level** | The admissible postures per action class | Determines what a control may **assert** (§5) | [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §9.4 |
| **Tool set** | Every tool the agent may call, with the scope of each | The preventive control; an added tool is a scope change requiring re-approval (§12) | [ai_risk_register_guide.md](ai_risk_register_guide.md) §4.3 |
| **Data reach** | Which data domains, in what form, read or write | Blast radius and confidentiality classification | [grc_in_banking_guide.md](grc_in_banking_guide.md) §12.1 |
| **Model dependencies** | Base model(s) and pinned versions; any fine-tune lineage | Non-determinism and drift are properties of this field | [Agent Versioning](../technology/ai_llm/agent_versioning_guide.md) §4 |
| **Configuration version** | Prompt set, tool manifest, policy bundle, as a version | "What was running" must be reconstructible | [Agent Versioning](../technology/ai_llm/agent_versioning_guide.md) §6 |
| **Identity / credential** | The NHI the agent authenticates as, and its privilege level | An agent is an identity; privilege is the security surface | [Non-Human Identity](../technology/non_human_identity_guide.md) §7.3 |
| **Approval date / approver** | Who authorised the current level and scope, when | The audit trail of the delegation itself | [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §9 |
| **Review date** | When autonomy, scope and continued need are re-examined | Autonomy creep is prevented by a forced re-decision | [Non-Human Identity](../technology/non_human_identity_guide.md) §7.5 |
| **Status** | Live / paused / retired, with the effective date | A paused agent still holds authority until the credential is withdrawn | [Non-Human Identity](../technology/non_human_identity_guide.md) §4.1 |
| **Control(s) operated** | Which controls this agent performs, if any | The link that makes the paradox in §7 visible | [grc_in_banking_guide.md](grc_in_banking_guide.md) §6.4 |

### 4.4 The register anti-pattern the field table prevents

The register anti-pattern is **a model inventory wearing an agent register's name**. A bank lists its models, classifies them, and calls that the AI register — and then an agent built from a listed model, with a new tool set and a new mandate, sails in underneath the classification because *the model was already registered*. The register in §4.3 prevents this structurally: the **tool set**, the **autonomy level**, the **mandate** and the **control(s) operated** are fields a model inventory has no place to hold. If your register cannot represent the difference between *model M* and *agent A that uses model M with tools T under mandate D*, it is a model inventory and it will miss every agent.

### 4.5 Ownership: the delegation has a delegator

The register's `Accountable owner` field is where the chain of delegation terminates, and it is the field that makes §7 possible. An agent operating a second-line control answers to a principal; that principal is the named human in the register; and if the register's owner field is empty or holds a team name, the accountability for whatever the agent does is, in fact, unassigned. [Non-Human Identity](../technology/non_human_identity_guide.md) §7.1 owns the ownership model for the wider NHI estate; the specific rule for a GRC-performing agent is that **the owner must be able to originate the attestation** (§3.5) — a team cannot sign, so a team cannot own.

---

## 5. The Autonomy Question

### 5.1 Consuming the ladder, not restating it

The four postures an agent may take at a decision point — **ASK** (stop and put a question to a human), **RECOMMEND** (produce a proposal; a human commits), **ACT** (effect the change on its own authority), **WAIT** (do nothing — a cost, not a neutral default) — are defined, argued and tabulated by [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §2, and mapped to action classes by its §9.4, whose rule is that **the class, not the confidence, is the primary key**, with confidence able to move an action *down* the table but never *up* it. Its §1.4 thesis is that *the posture is a choice about who carries the cost of being wrong*, and its §9.5 lists what the record must show afterwards for each posture. None of that is restated here. This section asks the **governance** question that the posture model raises and does not itself answer: **what can a control assert at each level?**

### 5.2 What a control can assert at each posture

A control is only as strong as what its performer is permitted to do. The same agent, monitoring the same population, supports a *different* assertion depending on its posture — and a bank that registers the agent without registering the posture has no idea which assertion it is entitled to make.

| Posture (per [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §2) | What the agent actually does | What a control operated at this posture **may assert** | What it **may not assert** |
|---|---|---|---|
| **ASK** | Stops and escalates a question | That a question was raised at a defined point, and that a human decided | Nothing about the decision itself — the human made it |
| **RECOMMEND** | Produces a proposal; a human commits | That the institution was presented with a proposal at a point, and a human committed | Nothing about the proposal's correctness — it was advisory |
| **ACT** | Effects the change on its own authority | That the action was taken **within a registered mandate** — an authorised action, evidenced | Nothing beyond the mandate; the assertion is scoped by the delegation, not by the agent's confidence |
| **WAIT** | Does nothing at a decision point | That a defined decision point passed without action — **a cost to justify**, not a clean result | Not that nothing needed doing; WAIT is an active choice that consumes a decision |

The table's lesson is the same one §3.2 gave in another form: **a control's assertion is bounded by the performer's authority.** An agent at ASK cannot support a claim that a control *operated independently* — its only action is to ask. An agent at RECOMMEND cannot support a claim of *detection*, because a proposal no one committed to detected nothing. Only at ACT does the agent itself carry the assertion, and then only within the mandate the register recorded. This is why the register's `Autonomy level` field and the control's `Assertion` are the same fact viewed from two sides.

### 5.3 Where the governance line sits

The governance line is **not** drawn at a fixed posture. It is drawn at the point where the assertion a posture supports stops being **re-performable and attributable**. Three rules make that concrete:

1. **The governance demand rises as the posture rises.** ASK and RECOMMEND require governance of the *question* — that it was asked at the right point, of the right person, with the right context. ACT requires governance of the *mandate* — that the action class was admissible at that level, that the tool scope permitted it, and that the evidence ties the action to the agent's registered version. WAIT requires governance of the *threshold* — that the decision to do nothing was within the agent's design and not a silent failure.
2. **A posture may not be raised by configuration alone.** Because the class is the primary key ([Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §9.4), moving an action class from RECOMMEND to ACT is a **change of mandate** — a register change, an approval, a re-attestation — not a settings toggle. An autonomy level that can be edited without an approval is an autonomy level that is not governed.
3. **The human whose approval the posture depends on must be able to exercise it.** §9.2 of the posture guide owns the four-eyes problem when one of the two eyes is an agent; its §5 owns the economics of interruption — approval fatigue, and the finding that a prompt nobody reads is not a control. The governance consequence: a RECOMMEND that is approved at a rate that makes review nominal has become an ACT with extra steps, and the register's autonomy level is now a fiction.

### 5.4 The line this guide draws

**The governance line sits where authority is delegated, not where capability ends.** An agent may be *capable* of acting; the governance question is whether the institution has *delegated* the act, to *this* agent, at *this* scope, with evidence and a named owner. Capability without delegation is an ungoverned agent. Delegation without evidence is an unprovable one. Both are failure modes in §12, and both are visible in the register — which is the whole point of building it first.

## 6. Agents Doing GRC: The Activity Rulings

### 6.1 The rule that governs every row

This is the section the guide exists for. An agent can be put to work in a bank's GRC function today, and the argument is not *whether* — it is *what each activity permits*. Six activities recur. For each, the ruling answers three questions: **what the agent MAY do**, **what it MUST NOT do**, and **WHO REMAINS ACCOUNTABLE**. The three answers are not a spectrum of enthusiasm; they are a boundary, and the boundary is the same one §1.4 stated: an agent may **find, draft, sample, monitor and assemble**; it must not **conclude, attest, approve or decide**; a **named human** remains accountable.

### 6.2 What the agent may and may not do — the activity rulings

| Activity | What the agent **MAY** do | What it **MUST NOT** do | **Who remains accountable** |
|---|---|---|---|
| **Control-testing agent** — performs or supports a control test | **Sample** a population within a defined rule; **run** the test against a designed set; **assemble** the evidence artefact; **flag** exceptions and outliers for review | **Conclude** that the control is effective; **sign** the test result; choose the population definition or materiality **on its own authority**; test a control it is registered against as owner | The **control owner** for design, and the **second-line testing lead** for the conclusion and the rating |
| **Compliance-monitoring agent** — watches activity against an obligation | **Monitor** a defined channel or population continuously; **detect** patterns that match a stated rule; **escalate** candidates with context | **Decide** that a matter is a breach; **disposition** an alert as a false positive on its own authority; **close** a case | The **compliance officer** who dispositions the alert; the **obligation owner** for the rule's interpretation |
| **Regulatory-change tracker** — watches for and interprets change | **Find** candidate publications from named sources; **summarise** a dated document; **draft** an impact note with citations; **assemble** the change log | **Assert** that a rule applies, is binding, or takes effect on a date; **classify** the change's severity as final; **omit** the human's review of the source | The **regulatory-change lead** who validates the source and issues the impact assessment |
| **Evidence collector** — gathers the artefacts behind a control or attestation | **Collect** artefacts from named systems on a schedule; **index** them against the control; **check** completeness and flag gaps | **Assert** that evidence is sufficient or that a control is evidenced; **certify** completeness to an auditor; substitute its own summary **for** the artefact | The **control owner** who accepts the evidence pack; the **attester** who relies on it |
| **Issue tracker** — manages remediation items | **Create** a ticket from a finding; **chase** an owner on a cadence; **summarise** status; **draft** the closure write-up | **Close** an issue; **downgrade** a severity; **accept** a remediation as complete without a human decision | The **issue owner** for remediation; the **second line** for validation and closure |
| **Indicator watcher** — monitors a KRI against a threshold | **Compute** an indicator from named data; **watch** a threshold; **alert** on breach with the underlying values | **Set or change** the threshold or appetite; **judge** that a breach is tolerable; **explain away** a breach in the record | The **KRI owner** who owns the threshold, and the **risk committee** that owns the appetite ([grc_in_banking_guide.md](grc_in_banking_guide.md) §9) |

### 6.3 Reading the table

Three patterns run through every row, and they are the honest generalisation of this guide.

**Pattern one — the agent's verbs are all about *production*, the human's verbs are all about *judgement*.** Every MAY column is a verb of assembly: sample, run, assemble, flag, monitor, detect, escalate, find, summarise, draft, collect, index, create, chase, compute, watch, alert. Every MUST-NOT column is a verb of decision: conclude, sign, decide, disposition, close, assert, certify, accept, approve, judge. The division is not arbitrary — it is the division between work whose *output is a candidate* and work whose *output is a position the institution holds*. Only a person can hold a position the institution is audited on.

**Pattern two — accountability never moves to the agent.** Each row names a human role, and behind that role is a named person per [grc_in_banking_guide.md](grc_in_banking_guide.md) §13. The agent changes *who does the work*; it does not change *who is answerable*. An institution that lets "the agent closed it" stand as an answer has no answer.

**Pattern three — the must-not columns are mostly about *the agent's own output*.** The sharpest prohibitions are not on what the agent does to the world but on the agent validating itself: a control-testing agent choosing its own population, an evidence collector certifying its own completeness, an issue tracker closing its own ticket. Those are the rulings that map onto §7, because they are exactly where the loop closes on itself.

### 6.4 The activities where the ruling is genuinely contested

Honesty requires naming the cases where the MAY/MUST-NOT line is not settled by argument alone, and where a bank must make — and record — a judgement call.

- **Sampling rules.** An agent *may* sample. But *who defines the sampling rule*? If the agent defines the population and the sample, it has quietly taken a design decision that belongs to the test owner. The defensible ruling: the agent **executes** a sampling rule the test owner set and registered; the rule is part of the control's design, not part of the agent's discretion. Where a bank lets the agent choose, it has moved a MUST-NOT into a MAY, and §12 names that as independence collapse.
- **Detective confidence thresholds.** An agent *may* detect and escalate, and every detection has a threshold. The threshold is a design parameter with a real cost of being wrong in each direction, so the ruling is that the threshold is owned by the compliance function, is registered, and is changed only through the change process — not tuned live by the agent or by an engineer chasing a quieter alert queue.
- **Summarising a source.** A regulatory-change tracker *may* summarise a dated document. The contested case is a summary that **drops a qualifier** — a "subject to", a "may", a commencement date — and thereby changes the obligation. The defensible ruling: the agent's summary is a *navigation aid* to the source, never a substitute for it, and the human validates against the source, not against the summary.

Those three are where the honest answer is "it depends on a decision the bank must make explicitly." The guide's contribution is to insist the decision be **visible in the register and the evidence**, rather than implicit in a config file.

---

## 7. The Loop — The Agentic-GRC Paradox

### 7.1 The paradox stated

Here is the guide's central claim, and it is uncomfortable: **the moment an agent performs a GRC activity, it becomes an actor inside the framework it is executing.** It is no longer only a tool *applied to* the control environment. It is a component *of* the control environment, and everything the framework demands of a component — an owner, a scope, a version, a test, an evidence trail, an independent review — now applies to it. The framework has acquired a subject that is simultaneously its instrument. That is the loop.

The paradox has a concrete consequence: **every question a GRC framework asks about a control, it must now also ask about the agent that performs the control.** Is it designed to do what it claims? Is it operating as designed? Is there evidence? Who is accountable? Is it independently reviewed? A bank that answers those for the *control* and not for the *performer* has a framework with a hole exactly where the automation sits.

**An agent that tests a control is itself a control that needs testing.** Not because the agent is suspect, but because it *is* a control — a mechanism the institution relies on to produce a reliable result — and the reliance is what triggers the obligation. If a bank relies on the agent's output to conclude a control is effective, then the agent's reliability is load-bearing for the conclusion, and a load-bearing mechanism is by definition something the framework must independently test. §9 applies the control-testing vocabulary to it; §8 says what the evidence must be.

### 7.2 Independence when the performer is automation

Independence is the framework's load-bearing wall: the second line tests what the first line runs, and the third line reviews both, precisely because *the performer cannot certify themselves* ([grc_in_banking_guide.md](grc_in_banking_guide.md) §2 on the three lines as a working relationship; §6.2 on second-line testing as independent testing; §7 on the third line). Agentic performance breaks the wall in two distinct places, and only one has a clean answer.

**The clean case — an agent performing a first-line activity.** The first line's operational work moves to an agent. The second line still tests it, and if the second line is a person or a function independent of the agent's business owner, independence is preserved. The agent is simply a *subject* of second-line testing, exactly as a human operator would be. This case has a clean answer, and it is: **register the agent, scope it, and test it.** [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §9.2 — *the four-eyes control when one of the two eyes is an agent* — is the relevant cross-reference for the design of the maker-checker step when the maker is an agent; this guide does not restate it.

**The contested case — an agent performing a second-line activity.** Here independence is genuinely at risk, and the risk is not that the agent is biased — it is that **the agent's owner and the agent's subject can collapse into one function.** If the same first-line team owns the control, owns the agent that tests the control, and owns the agent's findings, then no independent eye exists, because the second-line test has been performed by an actor inside the first line. This is the loop closing, and it has **no clean answer** — only honest ones. The defensible rulings available to a bank are:

| Option | What it looks like | What it costs |
|---|---|---|
| **Separate the agent's ownership from the subject's ownership** | The agent is owned by a function organisationally distinct from the business whose control it tests, with its own mandate and budget | Governance overhead; a second-line agent is a second-line resource and must be resourced as one |
| **Keep a human in the two-eyes loop for the conclusion** | The agent samples and assembles; an independent human draws the conclusion and signs (§6.2) | The conclusion is not automated, which is the point |
| **Submit the agent itself to independent testing** | The agent's reliability is tested by a party independent of both the agent's owner and the control's owner | Requires a party able to test a non-deterministic performer (§9) |

A bank may combine the options, but it **may not** leave the contested case unaddressed, because the unaddressed version is anti-pattern 4 (§14): a control tested by the agent that owns the control. The framework's answer to "who tests the tester" cannot be "the tester."

### 7.3 Why a framework that cannot attribute an opinion to a person cannot be audited

This is the sentence the whole guide turns on, so it is worth being precise about why it is true rather than just assertive.

An audit works by **attributing a position to an accountable party and testing whether the party was entitled to hold it.** An opinion — "in my view the control is effective," "I accept this residual," "I attest that the requirement was met" — is only auditable because it is *someone's* opinion, someone who can be asked to explain, correct, or be held to it. Strip the person away and three things fail at once:

1. **There is no one to ask.** An agent cannot be examined on its reasoning, cannot be cross-questioned on a judgement call, cannot be told the standard it was wrong against, and cannot be held to account in any way that means anything. An auditor's follow-up questions have no addressee.
2. **There is no entitlement to test.** An opinion held by a person falls within that person's delegation, and the audit can test *whether they were entitled to hold it*. An agent has no delegation of *judgement* — its mandate covers actions, not positions (§5.2) — so there is nothing to compare the opinion's scope against.
3. **There is no signature.** The evidence chain terminates at a name or it terminates nowhere. [grc_in_banking_guide.md](grc_in_banking_guide.md) §13 owns this for human performance; the extension to agentic performance is that the agent's output must be **consumed by a named person who adopts it as their position**, and the adoption is the auditable act.

The consequence for the loop is the guide's core prescription: **an agent may produce everything an opinion rests on, and may not be the opinion.** The moment an agent's flag is treated as the institution's position, the framework has a conclusion with no one behind it, and the audit has nowhere to land. This is not conservatism about the technology; it is a statement about what accountability *is*.

### 7.4 The loop, drawn

The paradox has a shape, and the shape is a circle that must be broken at exactly one point.

```
        ┌─────────────────────────────────────────────┐
        │  the framework (three lines, controls,       │
        │  testing, evidence, attestation)             │
        └─────────────────────────────────────────────┘
                     ▲                        │
        governs      │                        │  is executed by
                     │                        ▼
        ┌───────────────────┐        ┌───────────────────┐
        │  the agent        │◄───────│  the agent as a   │
        │  as a CONTROL     │  is     │  PERFORMER of a   │
        │  that needs       │  the    │  framework         │
        │  testing (§9)     │  SAME   │  activity (§6)     │
        └───────────────────┘  actor  └───────────────────┘
                     │
                     ▼
        ┌─────────────────────────────────────────────┐
        │  the break: a NAMED HUMAN adopts the agent's │
        │  output as their own position (§7.3).        │
        │  This is the only point at which the loop    │
        │  becomes an auditable line.                  │
        └─────────────────────────────────────────────┘
```

The circle is not a defect to be engineered away. It is the true description of a framework that executes itself through an actor, and the honest design response is to **name the break point and staff it** — a person who adopts the agent's output, is entitled to hold the resulting position, and can be audited on it. Every failure mode in §12 is a version of this break point being missing, unnamed, or delegated back to the agent.

## 8. Evidence and the Audit Trail

### 8.1 What an agent's evidence must carry

For a human-operated control, the evidence is largely the artefact: a signed review, a reconciliation, a ticket. For an agent-operated control, the artefact is not enough, because the thing the artefact claims to prove *depends on what was running and what it was given.* The evidence must therefore carry the identity of the run, not just its result. Six fields are the minimum, and they map directly onto the register fields of §4.3 — the register exists so that this evidence is producible at all.

| Field | What it records | Where it is established as the agent's identity |
|---|---|---|
| **Model and version** | The base model, its pinned version, and any fine-tune lineage in force at the run | Register `Model dependencies` ([Agent Versioning](../technology/ai_llm/agent_versioning_guide.md) §4) |
| **Prompt / configuration version** | The prompt set, tool manifest and policy bundle, as a version identifier | Register `Configuration version` ([Agent Versioning](../technology/ai_llm/agent_versioning_guide.md) §6) |
| **Tools called** | Every tool invocation, with its scope and its arguments or a reference to them | Register `Tool set` |
| **Inputs** | The data and records the agent read, and their source and as-of date | Register `Data reach` |
| **Outcome** | What the agent produced or did — a flag, a draft, an action, a no-op | Register `Control(s) operated` |
| **Human acceptance** | Who adopted the output as the institution's position, and when | Register `Accountable owner`; the break point of §7.4 |

The sixth field is the one a bank is most likely to omit, and it is the one that makes the other five matter. Without it, the five fields document an agent's activity; with it, they document a **governed decision**. [ai_risk_register_guide.md](ai_risk_register_guide.md) §4.5 owns the framing — *logging that preserves evidence* — and §5 carries the test this section applies: **a claim of control without an artifact is an assertion.** The agentic extension is that the artefact must be a *run* artefact, not a *result* statement, because the result is what the agent says and the run is what can be re-examined.

### 8.2 The sharp question: re-performance versus sampling

This is the section's hard content, and it is where the guide's thesis is most tested.

**A deterministic control can be re-performed. A non-deterministic one can only be sampled.** That single sentence reorganises assurance work, and it is worth tracing the consequences fully rather than softening them.

| Property | Deterministic control (classic) | Non-deterministic control (agentic) |
|---|---|---|
| **Can a second party repeat it?** | Yes — run the same rule on the same data, get the same answer | No — the agent may take a different path; the outcome is a draw from a distribution |
| **What validates it?** | Re-performance and inspection of the mechanism | A **sample or evaluation set**, characterised as a rate |
| **What the conclusion rests on** | The mechanism's identity — this rule, this data, this result | The **distribution's properties** — this error rate, on this population, at this version |
| **What invalidates an earlier conclusion** | A change to the rule or the data | A change to the **model, prompt, tool set or data** — any of which can move the distribution |
| **The auditor's move** | Re-perform, compare, conclude | **Design a test, run the agent against it, examine the error rate and the human acceptance, and test the design of the sample** |

The consequence for the **assurance map** ([grc_in_banking_guide.md](grc_in_banking_guide.md) §6.4) is that an agent-operated control cannot be assigned the assurance a re-performable control earns. Its assurance is *sampling-based*, which means the map must record, for that control, **the sample design and the error rate** rather than a binary "tested / effective." A map that records "effective" for a non-deterministic performer has silently claimed a re-performance that never happened.

The consequence for the **auditor** is the sharpest of all, and it is worth stating in the auditor's own terms. When an auditor cannot re-perform, three things become the whole of their leverage:

1. **Test the design of the sample.** If the sample that produced the error rate was chosen by the agent's owner, the auditor's first question is whether the sample could have produced any other answer.
2. **Test the human acceptance.** Did a person with the right entitlement actually adopt the output, or did they approve a queue? The second is a control that exists on paper only.
3. **Test the version discipline.** Can the institution prove *which* model, prompt and tool set were in force when the sampled runs executed? If not, the error rate characterises a version that may never have run.

None of those three is a re-performance, and none is a formality. They are the *substitute* for re-performance, and a bank that has not built the register and the run evidence of §8.1 has left the auditor with nothing to work with. This is where the thesis is tested and holds: the agent's evidence burden is **higher** than a human performer's, not lower, precisely because the agent cannot be re-performed.

### 8.3 Observability, versioning and the drift problem

The evidence in §8.1 is only producible if the agent's operations are observable and its versions are pinned. Two sibling guides own the mechanisms, and this section maps them rather than restating them.

- **Observability.** [AgentOps](../technology/ai_llm/agentops_guide.md) §3 owns agent observability — what to trace, at what granularity, retained how. For GRC purposes, the requirement is that the trace be **sufficient to reconstruct the six fields of §8.1 for any run the institution relied on**, for a retention period the institution sets. A trace good for debugging an agent is not automatically good for evidencing a control; the difference is retention, tamper-evidence, and the human-acceptance link.
- **Versioning.** [Agent Versioning](../technology/ai_llm/agent_versioning_guide.md) §4–§6 owns artifact and config versioning; §7 owns eval-gated promotion. For GRC purposes, the requirement is the *inverse*: not just that a version can be promoted, but that **any past run can be mapped back to the exact versions in force.** An eval-gated release that cannot be resolved to the run that produced a control conclusion has a versioning gap that lands as an evidence gap.
- **Drift.** [AI Agent Drift](../technology/ai_llm/ai_agent_drift_guide.md) owns drift detection and mitigation. For GRC purposes, the consequence is that **an evidence artefact has a shelf life set by the version's stability.** A sample taken three months ago characterises the version in force three months ago; if the model, prompt or tool set moved since, the evidence describes a performer that no longer exists. The review cadence in the register (§4.3) is what forces that re-examination.

### 8.4 The evidence summary rule

Stated once, as the rule a bank can apply: **the institution may rely on an agent's control activity only to the extent that it retains a run artefact tying the outcome to a version, a scope, and a named human acceptance.** Everything else — a dashboard, a score, an alert count — is an indicator *about* the control activity, not evidence *of* it. The two are routinely confused, and the confusion is anti-pattern 7 (§14).

---

## 9. The Control Framework Adapted

### 9.1 The vocabulary is borrowed whole

Preventive, detective and corrective controls; design effectiveness versus operating effectiveness; the RCSA and the assurance map — [grc_in_banking_guide.md](grc_in_banking_guide.md) §5–§6 owns all of it, and the definitions are not re-derived here. What this section does is place the agent's mechanisms into that vocabulary, because the placement is not obvious and it changes how each is tested.

### 9.2 Guardrail and monitor, placed in the classical frame

| Agent mechanism | Classical control type | What it controls | How it is tested |
|---|---|---|---|
| **Guardrail** — a permission boundary, scope limit, whitelist or approval gate that blocks an inadmissible action before it occurs | **Preventive** | The action's admissibility — what the agent may do at all | By **design testing** and by **attempted violation**: can the agent be induced to attempt a blocked action, and does the block hold? ([AI Red Teaming](../technology/ai_llm/ai_red_teaming_guide.md) §6 agentic attack surfaces) |
| **Monitor** — a continuous check that detects an out-of-policy condition after it occurs | **Detective** | The occurrence and pattern of a condition | By **evaluation against a seeded set**: does the monitor detect what it claims, at what rate? ([LLM Evaluation Frameworks](../technology/ai_llm/llm_evaluation_frameworks_guide.md) §3, §15) |
| **Human-in-the-loop gate** — a checkpoint where a person must approve before an action proceeds | **Preventive**, and an **approval control** in the classical sense | The action's authorisation at a class boundary | By **testing the gate's design and its exercise rate** — a gate nobody declines is not operating (§5.3) |
| **Logging and run record** | **Detective support** — it does not prevent or detect by itself | The reconstructibility of what happened | By **sampling the record against a known run** — can you reconstruct the six fields of §8.1? |
| **Evaluation as a release gate** | **Preventive** (at the release boundary) | Which versions may run at all | By **testing the gate**: can an un-evaluated version reach production? ([ai_risk_register_guide.md](ai_risk_register_guide.md) §4.6) |

The placement matters because a guardrail and a monitor fail in different ways and must be tested differently. A guardrail fails by **admitting what it should block** — a design defect, tested by attempting the blocked action. A monitor fails by **missing what it should catch** — a detection defect, tested by seeding the conditions it claims to catch. A bank that tests both with the same method tests neither properly.

### 9.3 Control testing when the control is non-deterministic

[grc_in_banking_guide.md](grc_in_banking_guide.md) §6.1–§6.3 owns RCSA, second-line testing and the sampling-and-evidence question for human-operated controls. Three adaptations apply when the performer is an agent, and each is a *substitute* for a method that no longer works rather than an addition.

1. **Evaluation sets replace walkthroughs.** A walkthrough inspects a mechanism's operation once. For a non-deterministic performer, the equivalent is running the agent against a **designed evaluation set** — a curated population with known-correct outcomes — and characterising the result as a rate. [Agentic Engineering](../technology/ai_llm/agentic_engineering_guide.md) §8 owns the evaluation phase, and [LLM Evaluation Frameworks](../technology/ai_llm/llm_evaluation_frameworks_guide.md) the methods; the GRC requirement is that the set be **owned by the test function, not by the agent's owner**, for the independence reason in §7.2.
2. **Sampling replaces re-performance.** Since the control cannot be re-performed (§8.2), the test becomes a sample whose *design* is itself testable. The population definition, the sample size, the stratum logic and the error tolerance are design decisions the test owner makes and records; the agent executes them (§6.4).
3. **Red-teaming replaces exception inspection.** A deterministic control's edge cases are enumerable. An agent's are not, so the edge-case work moves to **adversarial testing**: attempting to induce the agent to take an inadmissible action, to miss a seeded exception, or to act outside its scope. [AI Red Teaming](../technology/ai_llm/ai_red_teaming_guide.md) §6 owns the agentic attack surfaces; [../technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md) owns the containment half. The GRC requirement is that a red-team finding on an agent performing a control feeds the **control's** test record, not only the agent's engineering backlog.

The relationship between evaluation and validation is itself a distinction with its own guide — [LLM Evaluation vs Validation](../technology/ai_llm/llm_evaluation_vs_validation_guide.md) — and the GRC point is simply that a *validation* question ("is this fit for the purpose the bank relies on?") is a governance question, while an *evaluation* question ("how does it score?") is a measurement.

### 9.4 The RCSA whose subject behaves differently each time

An RCSA asks a business to assess its own risks and controls ([grc_in_banking_guide.md](grc_in_banking_guide.md) §6.1). When a control's performer is an agent, three RCSA questions become harder, and the honest answer to each is a design choice, not a formula.

- **"Is the control operating as designed?"** For an agent, the answer is a rate, not a yes. The RCSA entry must record the **error rate and its tolerance**, and must state the population the rate applies to. A blank "as designed" is a determination the assessment cannot support.
- **"What is the inherent risk of the control failing?"** For an agent, failure has an **automation signature**: it fails at scale *and consistently*, because one defect applies to every run until it is caught. A human control degrades gradually; an agent's control can fail entirely between two reviews. The RCSA must treat the agent's failure as a *step change*, not a *drift*.
- **"Who assessed it?"** The assessor must be able to interrogate the agent's evidence (§8) and must be independent of the agent's owner (§7.2). An RCSA assessed by the agent's own builders is not an assessment.

### 9.5 What does not change

The control framework's purpose is unchanged: to give the institution a defensible basis to rely on a result. The agent does not change the purpose, the three-line structure, or the requirement that a control have an owner and evidence. It changes the *methods* by which each is established, and it adds one requirement the human version did not have — the agent performing a control is *itself* a control, and §12 is the catalogue of what happens when a bank forgets that.

## 10. The Regulatory View — Structural Only

### 10.1 The constraint restated, and what it permits

This section prints **no** rule, no deadline, no threshold and no penalty figure. It names an instrument **only** where this pass verified it at the issuing body's own material with a date, and cross-references [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) §2–§3 and §9 for **every other** instrument, because that guide owns the landscape. What this section does own is the *structure of the supervisory conversation* when an automated performer is involved — the questions a supervisor asks, and why the agentic case changes the emphasis.

### 10.2 What a supervisor asks about an automated performer

The supervisory questions are recognisable, and a bank can prepare for them without any new instrument being named. They fall into five groups.

| The question | Why it is asked | What a defensible answer requires |
|---|---|---|
| **Who is accountable for this agent's actions?** | Supervisory accountability always lands on a person or an accountable function | A register owner that is a **named person** (§4.5), not a team or a vendor |
| **How do you know it is doing what you say it does?** | The supervisory concern is the gap between the agent's design and its operation | Independent testing of the agent (§9), with evidence that can be examined (§8) |
| **What are its limits, and how are they enforced?** | Autonomy and tool scope are the risk surface | A registered autonomy level and tool scope, enforced preventively (§5, §9.2) |
| **What happens when it is wrong?** | The supervisory question is not "can it fail" but "what did you build for failure" | A residual error rate owned by a person (§3.2), and a reversal path for actions taken |
| **Can you reconstruct a specific decision?** | The question that ends the conversation if the answer is no | The run artefact of §8.1, retained and resolvable to a version |

### 10.3 Where the agentic case changes the emphasis

Three shifts distinguish a supervisor's conversation about a model from one about an agent. They are structural observations, offered as argument, not as quoted rules.

1. **From the output to the action.** For a model, the supervisory concern has historically been the *output* and its consumers. For an agent, the concern is the **action already taken** — because an erroneous output can be caught downstream, while an erroneous action is a state that must be unwound (compare §2.3's incident row). This shift is why **tool scope and authorisation** are the agentic counterpart of model validation in the supervisory conversation.
2. **From the model to the mandate.** A supervisor asking about a model asks about fitness. A supervisor asking about an agent asks about **authority** — who authorised it to do this, up to what limit, and with what oversight. That is the delegated-authority axis of §2.2, and it has no analogue in the model estate.
3. **From transparency to traceability.** Transparency about a model means explaining how it works. Traceability for an agent means being able to **reconstruct what a specific run did** — the difference between a model card and a run record. The IMDA framework for agentic AI's emphasis on measures *throughout the agent lifecycle* and on controlled access to whitelisted services, described below, is the kind of structural guidance that centres traceability and access rather than model explanation.

### 10.4 The instruments verified this pass

Two instruments speak directly to the governed actor, and both were verified at the issuing body's own material during this pass. Everything else in this area is cross-referenced to [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) §2–§3.

- **IMDA, Model AI Governance Framework for Agentic AI — launched 22 January 2026** at the World Economic Forum, developed by the Infocomm Media Development Authority and announced by the Minister for Digital Development and Information. ✅ Per the launching press release, the framework is aimed at organisations deploying agentic AI and covers four dimensions: **(1)** assessing and bounding risks upfront by selecting appropriate use cases and placing limits on agents' autonomy and access to tools and data; **(2)** making humans meaningfully accountable by defining significant checkpoints requiring human approval; **(3)** implementing technical controls and processes throughout the agent lifecycle, including baseline testing and controlling access to whitelisted services; and **(4)** enabling end-user responsibility through transparency and education. The press release states that the framework **builds on the Model AI Governance Framework for AI introduced in 2020** and that "humans are ultimately accountable." *(Source: MDDI press release, 22 January 2026, and the IMDA framework PDF at `imda.gov.sg`.)*
- **MAS, Safeguards for Agentic Finance at Runtime (SAFR) — information paper published 3 July 2026.** ✅ Per MAS's own publications page, SAFR "proposes a framework for the governance of AI agents in financial services, defining how agent actions are **authorised**, how **human oversight is activated**, and what is **recorded at the point of every decision**," and "also discusses implementation considerations for institutions deploying SAFR in their agentic AI systems." The page states SAFR was **jointly developed by the financial industry under MAS's BuildFin.ai initiative**, and that reference code is available on request. *(Source: MAS Monographs/Information Papers index, published date 3 July 2026.)*

The SAFR description is close to this guide's own argument, and it is worth noting where the two agree without treating the agreement as a rule. Its three named axes — **authorisation, human oversight, recording at the point of every decision** — correspond to this guide's autonomy/delegation axis (§5), its break-point requirement (§7.4), and its run artefact (§8.1). That is a convergence of framing, not a legal conclusion, and a bank should read SAFR itself rather than this guide's summary. This guide does **not** convert SAFR into a set of obligations, because it is an information paper developed with industry and this pass did not verify that it creates any.

### 10.5 The anchors that sit behind the instruments

Two Singapore instruments verified this pass establish the *governance-of-AI* baseline that an agentic conversation inherits, and both are cross-referenced rather than applied.

- The **Model AI Governance Framework** (IMDA and the PDPC) — first edition **23 January 2019**, second edition **21 January 2020**. ✅ *(Source: AI Verify Foundation resources page, retrieved this pass.)*
- The **Model AI Governance Framework for Generative AI** — released **30 May 2024** (a proposed version was released for international views on 16 January 2024), structured around **nine dimensions** including accountability, data, trusted development and deployment, incident reporting, testing and assurance, and security. ✅ *(Source: AI Verify Foundation resources page. Note: the downloadable PDF carries a filename dated June 2024, so the exact document-revision date is marked ⚠ in §15.)*
- The **MAS FEAT Principles** (Fairness, Ethics, Accountability, Transparency) — **12 November 2018**, released with a FEAT Committee of industry partners. ✅ *(Source: MAS media release, published date 12 November 2018.)* FEAT is the baseline against which a Singapore bank's AI governance is framed; [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) §3 owns its substance.

### 10.6 The supervisory position, stated as argument

Pulling the section together: **a supervisor does not need a new instrument for an agent performing GRC work; they need the existing accountability question answered about a new class of performer.** The framework's demand is the same demand it has always made — that a named person is accountable, that the institution can show its basis for relying on a result, and that it can reconstruct what happened. The agentic case intensifies all three, because the actor is fast, non-deterministic and sets its own path. A bank that answers §10.2's five questions with evidence is answering the supervisory conversation, whether or not any instrument names the word "agent."

---

## 11. The Platform Question

### 11.1 The question

Can a GRC platform represent an agent as a **control performer**? This is not a procurement question — [grc_in_banking_guide.md](grc_in_banking_guide.md) §10.5 carries a **deliberate no-vendor constraint** for this series, and this guide honours it: no vendor is named, ranked or sized. The question is a **data-model** question, and it is the platform layer's version of the thesis.

A GRC platform's data model was built for human performers. A control has an owner, a frequency, a test, a result and a date. The performer is implied to be a person or a process, and the platform has no field for *"the performer is a non-deterministic actor that changed version three weeks ago."* The platform question is whether that field exists, and what the platform does with the autonomy level and the evidence the register holds.

### 11.2 What the platform must record — the delta over a human performer

| What a platform already holds for a human control | The agentic delta it must hold |
|---|---|
| Control owner | Owner **plus** the agent's accountable owner (a different field, §4.5) |
| Performer (a person or a process) | The **agent ID**, resolved to a register entry |
| Test result | A result **plus** the error rate, the population, and the sample design (§8.2) |
| Evidence attached | A run artefact resolvable to a **version**, with the human acceptance (§8.1) |
| Frequency | Frequency **plus** the version-in-force timeline, so a result can be dated to a version (§8.3) |
| Status: effective / not | A **characterised** status, because "effective at an error rate of *x* on population *P*" is not a binary |
| Issue linked to a control | An issue that can be **opened against the agent itself**, not only the control (§7) |

The last row is the platform's version of the paradox, and it is the acid test. **If the platform cannot open an issue against an agent-as-control, it cannot represent the loop, and the loop will therefore be invisible in the very system the bank uses to manage risk.** That is the concrete form of the platform question, and it is answerable without naming a product.

### 11.3 What the platform cannot do, and why that is the honest boundary

Two limits should be stated plainly, because both are places where a tooling claim would be overreach.

- **A platform cannot supply the attestation.** It can hold the evidence and route the artefact (§8.1); the opinion and the signature are the person's ([grc_in_banking_guide.md](grc_in_banking_guide.md) §13). A platform feature that "auto-attests" has automated the one act that requires a person, and §14 names it.
- **A platform cannot make a non-deterministic control re-performable.** The platform can store the sample design and the error rate; it cannot convert a rate into a mechanism. A tool that presents an agent's control result as a binary "effective" is making a claim on the institution's behalf that the evidence does not support.

The platform layer is therefore the place where the thesis becomes visible or is quietly dropped. [grc_in_banking_guide.md](grc_in_banking_guide.md) §10.1–§10.4 owns what a GRC platform structurally does, the data problem underneath, its failure mode, and how to distinguish the layers; §10.5 owns the no-vendor constraint. This section adds one requirement to that layer: **the platform must model the agent as a control and not only as a tool that operates controls.**

---

## 12. The Failure Modes

### 12.1 The catalogue — symptom, cause, guardrail

Each failure mode below is a way the loop of §7 closes on a bank. The symptom is what is observed, the cause is the mechanism behind it, and the guardrail is the specific control that prevents it — not a general exhortation to be careful.

| # | Failure mode | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **Passes the evaluation, fails in production** | Green evaluation at release; an incident weeks later that the evaluation never tested | The evaluation set did not cover the production population, or the version moved after release | Evaluation set owned by the test function and **refreshed on version change** (§9.3); production sampling that can contradict the release evaluation (§8.3) |
| 2 | **The agent tests the control it owns** | A control marked effective on the strength of a test run by the agent registered against that control | Owner and performer collapsed into one actor — no independent eye exists | Independence rule of §7.2: separate the agent's ownership from the subject's ownership, or keep a human in the two-eyes loop |
| 3 | **A register of models, not agents** | The estate's agents are covered because "the underlying models are registered" | The register has no field for tools, mandate or autonomy — it is a model inventory (§4.4) | The §4.3 field table; a test that asks *"can this register distinguish model M from agent A using M?"* |
| 4 | **Evidence that cannot be re-performed or sampled** | The result is recorded; the run, version and inputs are not | Logging optimised for debugging, not for evidencing a control (§8.3) | Run artefact per §8.1; retention and tamper-evidence set by the control owner, not by the engineering team |
| 5 | **Autonomy creep** | The agent gained a tool, a data source or an action class **after** it was approved | Scope changes made by configuration, outside the change and approval process (§5.3) | Register change control: tool set and autonomy level changes require approval and re-attestation; periodic tool-set reconciliation |
| 6 | **The auto-attested control** | The attestation exists, signed by the agent or by nobody, and the tracker shows the control effective | The attestation was automated, or a human signed without adopting the output as their position | §7.4's break point: a named human adopts the output; the platform holds evidence, not signatures (§11.3) |
| 7 | **The indicator read as evidence** | A dashboard, alert count or score is cited as evidence a control operated | Confusing an indicator *about* the control activity with evidence *of* it (§8.4) | The evidence test of [ai_risk_register_guide.md](ai_risk_register_guide.md) §5, applied to the agent's output |
| 8 | **The silent WAIT** | A decision point passed with no action and no record | WAIT is a posture, and an unrecorded WAIT is indistinguishable from a failure to run (§5.2) | WAIT must leave a record showing the decision point and the reason for inaction; the register records whether WAIT is admissible for that action class |

### 12.2 The three failures that are structural, not operational

Three of the eight are not mistakes in operation — they are **missing structures**, and no amount of operational vigilance fixes them.

**Failure 2 (self-testing) and failure 5 (autonomy creep)** are both *governance gaps*, not errors: nobody made a wrong decision; nobody made the decision at all. Failure 2 exists because the framework never asked *who owns the tester*. Failure 5 exists because scope changes were treated as configuration rather than as a change of mandate. Both are prevented by a structure — an ownership rule and a change-control rule — and neither is prevented by training or intent.

**Failure 3 (the model inventory masquerading as a register)** is the deepest, because it is silent. A bank with a model inventory believes it has enumerated its agents, so it will not look. The agents it cannot see are the ones with new tool sets, new mandates and new data reach — precisely the ones whose blast radius is growing. §4.4's structural test — *can the register distinguish model M from agent A using M?* — is the only reliable detector, because it asks the register a question a model inventory cannot answer.

### 12.3 What the failure modes have in common

Every failure in the catalogue is a version of the loop's break point (§7.4) being absent, unnamed, or delegated back to the agent. Failure 2 removes the human eye entirely; failure 6 removes the human position; failures 3, 4 and 7 leave the human without the material to hold a position; failures 1, 5 and 8 let a position be held on a performer that was not the performer in force. The catalogue is therefore not a list of eight independent problems. It is one problem — *the break point is not staffed* — observed from eight angles, which is exactly what the guide's thesis predicts.

## 13. The Cymbal Bank Worked Example

### 13.1 The setting — fictional and illustrative

Cymbal Bank is a fictional institution, and the scenario below is constructed to exercise the guide's argument, not to describe any real deployment. Its numbers are illustrative placeholders chosen to make the reasoning visible; they are not measurements, not benchmarks, and not claims about what any technology achieves. The scenario draws on the same Cymbal Bank persona used by the sibling guides ([grc_in_banking_guide.md](grc_in_banking_guide.md) §14, [ai_risk_register_guide.md](ai_risk_register_guide.md) §12, [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §12, [Non-Human Identity](../technology/non_human_identity_guide.md) §13).

Cymbal Bank runs a second-line compliance monitoring programme. One of its standing activities is monitoring the handling of customer complaints against a set of internal handling standards — a detective activity that has historically been performed by a small review team on a weekly sample. The team cannot look at every complaint, so the programme has always been a **sampling** programme, and the assurance it produced has always been a rate.

### 13.2 The deployment

Cymbal deploys an agent — call it **CMA-1** (Complaints Monitoring Agent) — to monitor the complaint population. CMA-1 reads the complaint record for each item, retrieves the relevant handling standard, evaluates whether the handling matched the standard, and **flags** the items it assesses as exceptions, with a short rationale and a citation. It does not close anything, does not contact a customer, and does not write to the complaint record. Its tools are read-only. Its posture, in the terms of [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §2, is **RECOMMEND**: it produces a proposal — a flag with a rationale — and a human commits to any consequence.

The register entry (§4.3) records: owner = the second-line monitoring lead (a named person); autonomy level = RECOMMEND on the flagging action class, ACT on none; tool set = read-only complaint API and the standards corpus, both whitelisted; data reach = the complaint domain, read; model dependency = a pinned base model; configuration version = a manifest bundle; status = live; review date = a fixed interval; controls operated = "complaints handling monitoring."

### 13.3 The loop closes on them — three ways

Cymbal's programme ran cleanly for a while, and then three things happened that together *are* the loop.

**First, the agent's output became load-bearing.** The review team, which had sampled a few hundred complaints a week, found that CMA-1 could read them all. Within a quarter, the team's practice had shifted: they reviewed CMA-1's flags and a spot-check of the non-flags, rather than sampling the population directly. Nothing had been decided; the practice simply moved. But the effect was that **the institution's conclusion about complaints handling now rested on CMA-1's assessments** — and a conclusion that rests on a performer makes that performer a control.

**Second, the question "who tests CMA-1?" had no owner.** Cymbal's monitoring lead owned CMA-1. The monitoring lead also owned the complaints monitoring programme — the activity CMA-1 performed. When internal audit asked who independently tested CMA-1's reliability, the honest answer was that nobody did: CMA-1's release evaluation had been run by the engineering team that built it, and its production behaviour had never been tested by a party independent of its owner. The loop had closed, and it had closed silently.

**Third, the evidence did not reconstruct a run.** When audit asked to see *how* CMA-1 had reached a particular flag, the team could produce the flag and its rationale but not the configuration version in force on the day, nor the exact standards-corpus revision CMA-1 had retrieved. The flags were stored; the runs were not. Under §8.2, the flags were indicators about the monitoring activity, not evidence of it.

### 13.4 The decision the agentic nature changed

Cymbal's second-line leadership made **one decision** that the agentic nature forced, and it is the pivot of the example.

**They refused to let CMA-1 perform the control test on the activity CMA-1 monitored.** The proposal on the table had been to close the loop neatly: let CMA-1 flag exceptions *and* let CMA-1 assess whether the handling controls were effective, since it was already reading every record. Cymbal declined, and the reason was not technical — CMA-1 was probably good enough — but structural: **an agent that monitors an activity cannot also be the party that assesses the control over that activity, because the two roles are the two eyes of the same four-eyes control, and a single actor cannot be both.** The independence the framework requires ([grc_in_banking_guide.md](grc_in_banking_guide.md) §2, §6.2, §7) cannot be supplied by the actor whose reliability is in question.

Three consequences followed from that one decision, and each shows the guide's argument doing work.

1. **A separate, human-owned assurance step was created.** The conclusion that complaints handling controls were effective would be drawn by a person independent of CMA-1's owner, using CMA-1's flags as *input* and not as *conclusion*. This is §6.2's ruling in practice: CMA-1 may sample and assemble; it must not conclude.
2. **The assurance conclusion was recorded as sampling-based, with a rate.** The item on the assurance map ([grc_in_banking_guide.md](grc_in_banking_guide.md) §6.4) did not read "effective." It read, in substance: *the monitoring control operated on the full population; CMA-1's flag reliability was characterised at a stated rate against a test-function-owned evaluation set; the human reviewer accepted the flags and adopted the conclusion; residual error was accepted within the control owner's stated tolerance.* The map carried the rate, the population and the human acceptance — because §8.2 forbids a binary on a non-deterministic performer.
3. **CMA-1 was entered as a control in its own right.** Following §7, the register linked CMA-1 to *two* roles: performer of the monitoring activity, and **subject** of an independent reliability test. An issue line was opened against CMA-1 as a control, with the monitoring lead as owner — the platform test of §11.2, satisfied because the platform could open an issue against the agent itself.

### 13.5 What the decision bought and what it cost

The decision bought **auditability**: the institution had a named human who had adopted CMA-1's output as their position, evidence that could be resolved to a version, and a rate where a binary would have been a fiction. It cost **throughput**: the independent assurance step was human work that the neat closed loop would have avoided. That trade is the honest shape of agentic GRC. The technology did not remove the human from the loop; it changed *which part* of the loop the human occupies — from performing the monitoring to owning the conclusion — and it added a new duty, the duty to test the performer.

### 13.6 The example, ended on the thesis

Cymbal's programme is now more defensible than it was and less automated than it could have been, and those two facts are the same fact. The agent did not become the control. It became a control — a component the institution relied on — and so it acquired an owner, a scope, a version, a test and an owner-adopted conclusion like every other load-bearing mechanism in the framework. **An agent that tests a control is itself a control that needs testing**, and Cymbal's refusal to let one actor be both the monitor and the assessor is the decision that made the sentence true of their programme rather than merely written in a policy.

---

## 14. The Anti-Patterns

The following are named anti-patterns, each a recognisable way a bank gets agentic GRC wrong. Each is given a name and a single sentence; the failure-mode analysis behind them is §12 and the argument is §7.

1. **The Model Inventory in a Register's Clothing.** The bank lists its models, classifies them, calls the result "the AI register," and never notices that the agents built on those models — with new tools, new mandates and new data reach — are invisible in it (§4.4, §12 failure 3).
2. **The Auto-Attestation.** The attestation is generated rather than made: a system produces the statement of opinion, and no person adopts it as their own position, leaving a signature with no signer (§7.3, §12 failure 6).
3. **Testing the Tester.** The agent that performs a control is also the party that assesses the control's effectiveness, so the two eyes of the four-eyes control are one actor wearing two hats (§7.2, §12 failure 2).
4. **The Register Without a Review Date.** The agent's autonomy and scope are approved once and never re-decided, so the tool set and the mandate drift while the register still describes the version that was approved (§5.3, §12 failure 5).
5. **Green Means Governed.** A passing evaluation is treated as a governance conclusion, and no one writes down what the green number entitles the bank to rely on (§3.3).
6. **The Attestation That Certifies the Machine.** A human signs an attestation worded as though they had personally performed steps the agent performed, forcing a signature the person cannot support and an audit line that will not hold (§3.5).
7. **The Indicator as Evidence.** A dashboard, an alert count or an exception rate is cited as evidence that a control operated, conflating an indicator about the activity with a run artefact that proves it (§8.4, §12 failure 7).
8. **Re-Performance Assumed.** A non-deterministic control is placed on the assurance map with a binary "effective," silently claiming a re-performance that the control's nature makes impossible (§8.2).
9. **Confidence as the Primary Key.** An action class is raised from RECOMMEND to ACT because the agent's confidence was high and the approval queue was long, inverting the rule that the class, not the confidence, governs the posture (§5.2).
10. **The Unreadable Gate.** A human-in-the-loop approval exists formally but is exercised nominally — the queue is approved at a rate that makes review a formality — so a RECOMMEND becomes an ACT with extra steps (§5.3).
11. **The Unregistered Agent.** An agent enters the estate or gains a tool without a register entry, so the institution is relying on a performer it cannot enumerate (§4.2).
12. **The Silent WAIT.** The agent takes no action at a decision point and leaves no record, so a legitimate posture is indistinguishable from a failure to act (§5.2, §12 failure 8).
13. **The Orphaned Performer.** The agent's owner is a team, a vendor or a build pipeline, so the accountability that must land on a person lands nowhere (§4.5).
14. **Vendor Enthusiasm.** The technology is presented as replacing the second line or the control tester, a market is sized, and an adoption statistic is quoted — all of it unsourced, and all of it out of bounds for this guide (§1.4, §15).

## 15. The Claims Audit

Every substantive claim in this guide is classified below. The status codes are: **✅ verified at a primary source during this pass · ⚠ partially verified (secondary source, or primary source confirms only part) · ❌ could not be verified.** Every source note carries a **date** and the **kind of source** (framework, regulator publication, dated press, vendor claim, repo-internal cross-reference, or this guide's own argument).

| # | Claim | Status | Source note |
|---|---|---|---|
| 1 | The phrase "agentic GRC" appears in zero of this repository's 665 Markdown guides (the count before this file), and [grc_in_banking_guide.md](grc_in_banking_guide.md) (967 lines) contains no `agentic`, `AI agent` or `autonomous` material; "agentic" appears in 161 files | ✅ | **Repo-internal cross-reference**, verified 2026-10-16 by case-insensitive content search across `/home/ubuntu/research` |
| 2 | The GRC machinery — three lines, operating model, control design, testing/RCSA/assurance map, KRI, platform layer, accountability — is owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) at §2–§16 | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 3 | The AI instrument landscape and the AI-specific operating model are owned by [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md) at §2–§9 | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 4 | The four postures (ASK / RECOMMEND / ACT / WAIT) and the action-class→posture rule "the class, not the confidence, is the primary key" are owned by [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §2 and §9.4 | ✅ | **Repo-internal cross-reference**, headings inspected 2026-10-16 |
| 5 | The AI risk register's controls (§4), including logging that preserves evidence (§4.5), and its evidence test "a claim of control without an artifact is an assertion" (§5) | ✅ | **Repo-internal cross-reference**, headings inspected 2026-10-16 |
| 6 | A registration-before-issuance rule, segregation-of-duties and audit-question material for an NHI estate are owned by [Non-Human Identity](../technology/non_human_identity_guide.md) §7.2, §7.4, §7.6, and the agent-as-NHI material by its §8 | ✅ | **Repo-internal cross-reference**, headings inspected 2026-10-16 |
| 7 | IMDA's **Model AI Governance Framework for Agentic AI** was launched **22 January 2026** at the World Economic Forum, developed by IMDA, across four dimensions, and states that humans are ultimately accountable | ✅ | **Regulator publication / dated press** — MDDI press release, 22 January 2026, and the IMDA framework PDF (`imda.gov.sg`) |
| 8 | MAS's **Safeguards for Agentic Finance at Runtime (SAFR)** information paper was published **3 July 2026**, and defines how agent actions are authorised, human oversight activated, and events recorded at the point of every decision | ✅ | **Regulator publication** — MAS Monographs/Information Papers index entry, published date 3 July 2026 |
| 9 | The IMDA/PDPC **Model AI Governance Framework** first edition 23 January 2019 and second edition 21 January 2020 | ✅ | **Framework / regulator publication** — AI Verify Foundation resources page, retrieved this pass |
| 10 | The **Model AI Governance Framework for Generative AI** was released 30 May 2024 (proposed 16 January 2024) with nine dimensions | ✅ | **Framework** — AI Verify Foundation resources page, retrieved this pass |
| 11 | The exact revision date of the MGF-for-GenAI **PDF** as distinct from its release date | ⚠ | **Framework** — the AI Verify page states release 30 May 2024, while the downloadable file is named `...-19-June-2024.pdf`; the discrepancy is unresolved (see §16) |
| 12 | Whether the MGF for Agentic AI has been **updated** after 22 January 2026, and on what date | ⚠ | **Dated press (secondary)** — a law-firm note dated June 2026 reports an IMDA update; the IMDA update factsheet URL would not extract, so no update date is printed |
| 13 | The **MAS FEAT Principles** were released 12 November 2018 with a FEAT Committee of industry partners | ✅ | **Regulator publication / dated press** — MAS media release, published date 12 November 2018 |
| 14 | The core argument that an agent performing a GRC activity becomes a control inside the framework it executes | ⚠ | **This guide's own argument** — the guide's thesis, argued at §7 and tested against its own complications (no external source is claimed for it) |
| 15 | The claim that a deterministic control can be re-performed whereas a non-deterministic one can only be sampled, and the assurance-map consequences that follow | ⚠ | **This guide's own argument** — reasoned at §8.2 from the properties of non-determinism; the reviewer must satisfy themselves of the premise |
| 16 | The activity rulings of §6 — what an agent may do, must not do, and who remains accountable, per activity | ⚠ | **This guide's own argument**, anchored on the repo-internal frames of claims 2, 4, 5 and 6; no external standard is cited as the source of the ruling |
| 17 | The register field set of §4.3 — owner, purpose, autonomy level, tools, data reach, model dependencies, version | ⚠ | **This guide's own argument**, derived from the NHI register discipline of claim 6 and the evidence needs of §8.1 |
| 18 | The failure-mode catalogue of §12 and the anti-pattern list of §14 | ⚠ | **This guide's own argument**, each traced to a mechanism argued in the guide rather than to an external source |
| 19 | The Cymbal Bank scenario of §13, including CMA-1 and every quantity in it | ⚠ | **This guide's own argument** — explicitly fictional and illustrative; Cymbal Bank is the sole institution and the sole persona, and no figure is a measurement |
| 20 | The vendor survey and human-factors figures sometimes cited in this area are carried by a sibling guide, not by this one | ✅ *(attributed, not adopted)* | **Vendor claim, attributed** — [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §5 carries them; this guide deliberately does **not** reprint the numbers, to avoid re-owning a vendor claim |
| 21 | **No labour-replacement claim and no productivity-gain claim** about agents replacing the second line, control testers or compliance officers was printed | ✅ | **This guide's own argument / omission** — recorded deliberately; see the note below |
| 22 | **No market size, vendor share or adoption statistic** for the agentic-GRC product category was printed | ✅ | **This guide's own argument / omission** — recorded deliberately; see the note below |
| 23 | **No framework, article, clause or control reference was asserted from memory** | ✅ | **This guide's own argument / omission** — every instrument reference is either verified at a primary source this pass or cross-referenced to the sibling that owns it |

**Note on rows 21–23, and why they are checklist rows.** These three rows are printed because an audit of this guide should be able to confirm what it *did not* say, not only what it did. The reason is not modesty:

- **Row 21 is omitted because the claim is not this guide's to make.** Whether an agent replaces a compliance officer is an empirical and economic question whose answer would require a dated, defensible study of a specific deployment. This pass verified no such study, so the guide rules the activity at the level of *what a control may assert* (§5.2) and *what the agent may do* (§6.2) rather than claiming a workforce outcome. Where the adjacent literature carries a figure, it is attributed to the guide that owns it and left there (row 20).
- **Row 22 is omitted because no primary source was verified for it.** A market size for a product category that barely exists as a category would be fabricated by construction, and fabricated numbers are the fastest way to discredit a governance document.
- **Row 23 is omitted because a control reference asserted from memory is exactly the error this series exists to prevent.** The guide's sourcing constraint at §1 says it plainly: no framework, rule, standard, article, clause or control reference may be asserted from memory. Every named instrument in §10 was verified at the issuing body's material during this pass, and everything else is a cross-reference to [AI Governance Frameworks](../technology/ai_llm/ai_governance_framework_guide.md).

The bulk of this table is, honestly, rows 14–19 — this guide's own argument and its repo-internal cross-references. That is the correct shape for a guide whose subject is a gap no prior source has filled: the verified list is short, the argued list is long, and the guide says which is which.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary

### 16.1 What could not be verified

Honest entries, each a thing this pass could not establish at a primary source.

1. **The revision date of the Model AI Governance Framework for Generative AI document.** The AI Verify Foundation page states the framework was *released on 30 May 2024*, but the download it links to is a PDF whose filename carries the date *19 June 2024*. This pass could not reconcile the two, so the guide prints the page's release date and marks the exact document revision ⚠ (§15 row 11).
2. **Whether the Model AI Governance Framework for Agentic AI has been updated since its launch, and when.** A secondary (law-firm) source dated June 2026 reports an IMDA update, but the IMDA factsheet URL for that update would not extract during this pass, so the guide prints only the verified 22 January 2026 launch and marks any update ⚠ (§15 row 12). The update is **not** asserted as fact.
3. **Whether SAFR creates any obligation, expectation or standard.** This pass verified SAFR's *existence, publisher, date and self-described purpose* at MAS's own page. It did **not** verify that SAFR imposes anything on an institution — it is described as an information paper jointly developed with the financial industry under the BuildFin.ai initiative. The guide therefore describes SAFR's framing and **refuses to convert it into a rule, deadline or requirement** (§10.4).
4. **Tool limitation — pages that would not extract.** The IMDA "Artificial Intelligence" landing page (`imda.gov.sg/how-we-can-help/artificial-intelligence`) and the IMDA "Updated Model AI Governance Framework for Agentic AI" factsheet both failed to extract (scrape error, engine failure), and the MAS FEAT full-document PDF URL returned a maintenance page. These failures are recorded as **tool limitations, not as absence of the material** — the same conventions the sibling [Non-Human Identity](../technology/non_human_identity_guide.md) §12 records for its own pass. The verified FEAT date was obtained from the MAS media release instead.
5. **A claim this guide deliberately refused to print.** No figure for *how much work an agent removes from a compliance function*, and no *market size, vendor share or adoption statistic* for agentic GRC, is printed anywhere in this guide. The refusal is deliberate and is recorded as rows 21–22 of §15. The nearest figures this guide's shelf carries — a vendor survey figure for permission-prompt reduction and a human-factors figure for dialog attention — are attributed to [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §5 and **not** re-owned or reprinted here.
6. **No primary source for the guide's own thesis.** The sentence *an agent that tests a control is itself a control that needs testing* is this guide's argument, not a citation. The guide states that plainly (§15 row 14) rather than dressing it as a framework principle. The convergence between that argument and SAFR's three named axes (§10.4) is noted as a convergence of framing, not as authority.

### 16.2 Glossary

| Term | Definition (as used in this guide) |
|---|---|
| **Agent** | A software actor that takes actions through tools, holds state across steps, and is delegated authority by a principal (§1.1). |
| **Agentic GRC** | Letting an agent perform, assist or support a GRC activity, and governing that agent through the framework it executes (§1.1). |
| **Agent register** | The authoritative list of agents and the attributes that make each governable (§4.3). |
| **Agent estate** | The population of agents in the institution, governed as actors — the counterpart of the model estate (§1.3). |
| **Attestation** | A named human's statement of opinion, with personal accountability attached; owned by [grc_in_banking_guide.md](grc_in_banking_guide.md) §13 (§1.1). |
| **Autonomy level** | The standing authority an agent holds, expressed as the admissible posture per action class (§5); defined by [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §9.4. |
| **Break point** | The point at which a named human adopts an agent's output as the institution's position, converting the loop into an auditable line (§7.4). |
| **Control owner** | The named person accountable for a control's design, operation and evidence — never the agent, even when the agent performs it (§1.1). |
| **Evidence artefact (run artefact)** | The durable record — version, prompt/config, tools, inputs, outcome, human acceptance — that lets a third party re-establish that the control operated (§8.1). |
| **Guardrail** | A constraint that prevents an inadmissible action before it occurs; a preventive control (§1.1, §9.2). |
| **Monitor** | A continuous check that detects an out-of-policy condition after it occurs; a detective control (§9.2). |
| **Non-determinism** | The property that an agent may take different paths on the same input, so its control can be sampled but not re-performed (§3.1, §8.2). |
| **Obligation** | A duty the institution owes, expressed as a rule, principle or expectation; discharged by controls (§1.1). |
| **Posture** | ASK, RECOMMEND, ACT or WAIT — defined and owned by [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) §2 (§5.1). |
| **Run** | A single execution of an agent, the unit the evidence of §8.1 records. |
| **Tool call** | A single unit of the agent's effect on the world — the atomic event an audit trail must capture (§1.1, §2.2). |

### 16.3 Cross-references in this series

- **The GRC machinery this guide does not restate:** [grc_in_banking_guide.md](grc_in_banking_guide.md) — §2 three lines, §3 operating model, §4 regulatory change, §5 control design, §6 testing/RCSA/assurance map, §7 third line, §8 issue/remediation, §9 KRI, §10 platform layer (incl. §10.5 no-vendor constraint), §11 board reporting, §12 adjacent domains (incl. §12.2 AI governance), §13 individual accountability, §14 worked example, §15 anti-patterns, §16 claims audit.
- **The governance layer above the models:** [../technology/ai_llm/ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) — §2 instrument landscape, §3 Singapore landscape, §5 three lines adapted to AI, §6 model-risk integration, §7 lifecycle gates, §9 regulatory mapping. **It governs the model estate; this guide governs the agent estate.**
- **The engineering discipline:** [../technology/ai_llm/agentic_engineering_guide.md](../technology/ai_llm/agentic_engineering_guide.md) — §8 evaluation, §10 operations.
- **The postures consumed by §5:** [../technology/ai_llm/agent_decision_policy_guide.md](../technology/ai_llm/agent_decision_policy_guide.md) — §2 postures, §5 economics of interruption, §9.2 four-eyes with one eye an agent, §9.4 action-class table, §9.5 what the record must show.
- **The AI risk register:** [ai_risk_register_guide.md](ai_risk_register_guide.md) — §3 failure modes, §4 controls incl. §4.5 logging, §5 evidence and the audit question, §6 rating and appetite, §7 three lines and ownership.
- **Non-human identity and the register discipline:** [../technology/non_human_identity_guide.md](../technology/non_human_identity_guide.md) — §3 inventory, §4 lifecycle and the orphan, §7 governance (incl. §7.2, §7.4, §7.6), §8 the agent as an NHI.
- **Evidence, versioning, operations and method:** [../technology/ai_llm/agent_versioning_guide.md](../technology/ai_llm/agent_versioning_guide.md), [../technology/ai_llm/agentops_guide.md](../technology/ai_llm/agentops_guide.md), [../technology/ai_llm/ai_agent_drift_guide.md](../technology/ai_llm/ai_agent_drift_guide.md), [../technology/ai_llm/agent_sandboxing_strategies_guide.md](../technology/ai_llm/agent_sandboxing_strategies_guide.md), [../technology/ai_llm/ai_red_teaming_guide.md](../technology/ai_llm/ai_red_teaming_guide.md), [../technology/ai_llm/llm_evaluation_frameworks_guide.md](../technology/ai_llm/llm_evaluation_frameworks_guide.md), [../technology/ai_llm/llm_evaluation_vs_validation_guide.md](../technology/ai_llm/llm_evaluation_vs_validation_guide.md), [../technology/ai_llm/agent_harness_engineering_guide.md](../technology/ai_llm/agent_harness_engineering_guide.md), [../technology/ai_llm/multi_agent_banking_guide.md](../technology/ai_llm/multi_agent_banking_guide.md), [../technology/ai_llm/enterprise_agentic_platform_architecture_guide.md](../technology/ai_llm/enterprise_agentic_platform_architecture_guide.md).
- **Compliance overlay, ERM and regulation:** [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md), [../technology/responsible_ai_frameworks_guide.md](../technology/responsible_ai_frameworks_guide.md), [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) (§5 owns the three-lines model itself), [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md), [regtech_guide.md](regtech_guide.md).

### 16.4 The closing summary

An agent can do real governance work. It can sample a population no review team could read, monitor a channel continuously, find a change the day it is published, collect evidence faster than a human and more consistently, and watch an indicator around the clock. None of that is in doubt, and a bank that refuses it is not being careful — it is being slower. The guide's argument is not that the work should not move; it is that **the framework must move with it**, and the framework's core requirement does not survive the move unexamined.

Three things break when the performer changes. **Certainty** breaks, because a non-deterministic control cannot be re-performed and its assurance is a sample and a rate, not a binary (§8.2). **Independence** breaks, unless the agent's ownership is separated from its subject's, because a framework whose two eyes are one actor has no second eye (§7.2). **Attribution** breaks, unless a named human adopts the agent's output as their position, because an audit can only test an opinion that someone holds (§7.3). Every failure mode in §12 and every anti-pattern in §14 is one of those three breaks, and the Cymbal Bank decision in §13 — refusing to let one actor be both the monitor and the assessor — is what holding all three at once looks like in practice.

The honest position the guide holds throughout is neither enthusiasm nor dismissal. An agent may **find, draft, sample, monitor and assemble**; it must not **conclude, attest, approve or decide**; and a **named human remains accountable** for each, because a framework that cannot attribute an opinion to a person cannot be audited. That division is not a limit on the technology. It is the description of what accountability is, applied to a new class of performer.

And the sentence that organises all of it, one last time, because the loop is the point:

**An agent that tests a control is itself a control that needs testing.**
