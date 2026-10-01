# ServiceNow as an Agentic Surface — The Skill, the Agent Layer, the MCP Path, and the Controls Consequence

> **A guide to ServiceNow as an agentic surface. It separates the three meanings of the word "skills" — a ServiceNow platform term of art, a general agent-packaging pattern, and a human competency — and then covers the one this guide owns: the generative-AI capabilities ServiceNow itself ships and formally calls *skills*. From there it describes the agent-building and orchestration layer (**AI Agent Studio**, **AI Agentic Workflows**, the **AI Agent Orchestrator**), the Model Context Protocol path in both directions (the **MCP Server Console** as an inbound endpoint; the **MCP Client** as an outbound consumer), and the controls consequence of all of it. Every product name, version state and mechanism here is traced to ServiceNow's own documentation, its own SDK documentation, or its own press release, and dated. Names that could not be traced are recorded as unverified or omitted — never asserted. The through-line is the thesis in §1.6: connecting an agent to the system of record is an integration project; letting it write there is a controls decision.**

**Author:** Jack Liu Shurui, Solution Architect
**Role:** Solution Architect, Cymbal Bank
**Last Updated:** October 2026
**Version:** 1.0
**Repository:** github.com/jackliusr/research
**Series:** Technology Guides — AI/LLM & Operations track

**Companion guides.** This guide owns one narrow thing: what is specific to **ServiceNow as an MCP endpoint and as an agent host**. It does not re-derive the protocol or the surrounding disciplines. The Model Context Protocol itself — its discovery model, progressive disclosure, and general integration patterns — is owned by [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md), [`ai_llm/mcp_discovery_guide.md`](ai_llm/mcp_discovery_guide.md) and [`ai_llm/mcp_progressive_disclosure_guide.md`](ai_llm/mcp_progressive_disclosure_guide.md); the agent-platform selection frame by [`ai_llm/ai_agent_platform_selection_guide.md`](ai_llm/ai_agent_platform_selection_guide.md); the harness by [`ai_llm/agent_harness_engineering_guide.md`](ai_llm/agent_harness_engineering_guide.md); the enterprise architecture by [`ai_llm/enterprise_agentic_platform_architecture_guide.md`](ai_llm/enterprise_agentic_platform_architecture_guide.md); and the deliverables pattern by [`ai_llm/agentic_solution_artifacts_guide.md`](ai_llm/agentic_solution_artifacts_guide.md). ITSM/ITIL process theory belongs to [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md) and [`capacity_sizing_guide.md`](capacity_sizing_guide.md). The assistive AI-for-IT analysis — including **ServiceNow Otto** (formerly **Now Assist**) incident summarisation, resolution notes, chat summarisation, AI search and flow generation — is owned by [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) **§6.2**, and this guide *builds on* that assistive layer rather than re-deriving it. The observability/operations bridge that names ServiceNow ITOM/AIOps Event Management and ServiceNow CMDB as an integration source is [`opentext_operations_bridge_guide.md`](opentext_operations_bridge_guide.md). Agentic workflow engines are [`agentic_workflows_guide.md`](agentic_workflows_guide.md) and [`durable_ai_agent_workflows_guide.md`](durable_ai_agent_workflows_guide.md). The Copilot-Studio-side view of ServiceNow as a *foreign* integration target — a featured knowledge source plus an incident-create connector, **not** as the agent host — is [`ai_llm/copilot_studio_agent_engineering_knowledge_guide.md`](ai_llm/copilot_studio_agent_engineering_knowledge_guide.md) **§4**; a different vantage point on the same estate, cross-referenced, never duplicated. The shared-credential and zero-trust reasoning the controls thesis reuses is [`zero_trust_legacy_estate_guide.md`](zero_trust_legacy_estate_guide.md), [`zero_trust_mainframe_guide.md`](zero_trust_mainframe_guide.md) and [`zero_trust_network_architecture_guide.md`](zero_trust_network_architecture_guide.md). The regulatory and governance frame — cross-referenced, and never asserted as any jurisdiction's rule — is [`../banking/grc_in_banking_guide.md`](../banking/grc_in_banking_guide.md), [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md), [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md), [`ai_llm/ai_governance_bias_redteaming_guide.md`](ai_llm/ai_governance_bias_redteaming_guide.md), [`audit_as_code_guide.md`](audit_as_code_guide.md), [`data_governance_guide.md`](data_governance_guide.md) and [`api_governance_guide.md`](api_governance_guide.md). The *human* sense of "skills" — competency and career — lives entirely in the repo's role-analysis and management literature: [`../personal/skill_gaps_enterprise_architect_guide.md`](../personal/skill_gaps_enterprise_architect_guide.md), [`../personal/agent_runtime_engineer_skill_gaps_guide.md`](../personal/agent_runtime_engineer_skill_gaps_guide.md), [`../personal/data_architect_skillgaps_guide.md`](../personal/data_architect_skillgaps_guide.md), [`../management/facilitation_skills_guide.md`](../management/facilitation_skills_guide.md), [`../management/communication_stakeholder_management_skills_guide.md`](../management/communication_stakeholder_management_skills_guide.md) and [`../management/management_consulting_skills_guide.md`](../management/management_consulting_skills_guide.md).

**Primary sources and evidence conventions.** Facts confirmed at a primary source during this pass are marked **✅**; claims that are vendor-published marketing, single-source, or not re-verified this pass are marked **⚠**; claims contradicted at a primary source are marked **❌**. For this guide the primary source is ServiceNow's own material: product documentation on `docs.servicenow.com` (release family **Brazil**, each article stamped *Updated September 10, 2026*, retrieved and checked **2026-10-01**), ServiceNow SDK documentation at `servicenow.github.io/sdk`, and one ServiceNow newsroom press release dated **23 October 2024**. A ✅ means "confirmed at the cited primary source during this pass," not "independently re-tested." The consolidated ledger is the **Claims Audit** (§15); the honest residue — including names in wide circulation that could **not** be traced — is **What Could Not Be Verified** (§16.1).

**How to read this guide.** §1 is the disambiguation and the boundary. §2–§4 are the platform, the AI surface it ships, and the agent layer. §5 is the MCP path in both directions. §6–§9 are the controls consequence — identity, evidence, the CMDB, and the guardrail ladder. §10–§12 are use cases, integration architecture, and the operating model. §13 is the Cymbal Bank worked example. §14–§16 close with anti-patterns, the claims audit, the unverified list, the glossary and the cross-references. Each body section ends with a cross-reference line rather than re-deriving a sibling guide's content.

**Reading time:** ~50 minutes

---

### Table of Contents

1. [The Overview: Skills Three Ways, the Decoder, and the Boundary](#1-the-overview-skills-three-ways-the-decoder-and-the-boundary)
2. [The Platform in One Page: What ServiceNow Is and Is Not](#2-the-platform-in-one-page-what-servicenow-is-and-is-not)
3. [The AI Surface ServiceNow Ships: Otto, Skills, and What They Do](#3-the-ai-surface-servicenow-ships-otto-skills-and-what-they-do)
4. [The Agent Layer: AI Agent Studio, Workflows, and the Orchestrator](#4-the-agent-layer-ai-agent-studio-workflows-and-the-orchestrator)
5. [The MCP Path: Inbound Endpoint, Outbound Tools, and the Approval Lifecycle](#5-the-mcp-path-inbound-endpoint-outbound-tools-and-the-approval-lifecycle)
6. [The Identity Question: AI User, Dynamic User, and Impersonation](#6-the-identity-question-ai-user-dynamic-user-and-impersonation)
7. [The Evidence Question: Records, Approvals, and Segregation of Duties](#7-the-evidence-question-records-approvals-and-segregation-of-duties)
8. [The CMDB as the Sharpest Case: Writing What the Estate Believes It Owns](#8-the-cmdb-as-the-sharpest-case-writing-what-the-estate-believes-it-owns)
9. [The Guardrail Ladder: Seven Rungs, What Each Buys and Costs](#9-the-guardrail-ladder-seven-rungs-what-each-buys-and-costs)
10. [Use Cases That Hold Up and Those That Do Not](#10-use-cases-that-hold-up-and-those-that-do-not)
11. [Integration Architecture: Flows, Fabric, Zero Copy, and the MCP Boundary](#11-integration-architecture-flows-fabric-zero-copy-and-the-mcp-boundary)
12. [The Operating Model: Ownership, Review, and Retirement](#12-the-operating-model-ownership-review-and-retirement)
13. [Worked Example: A Cymbal Bank ServiceNow Estate](#13-worked-example-a-cymbal-bank-servicenow-estate)
14. [Anti-Patterns: Symptom, Cause, Guardrail](#14-anti-patterns-symptom-cause-guardrail)
15. [Claims Audit](#15-claims-audit)
16. [What Could Not Be Verified, Glossary, and Cross-References](#16-what-could-not-be-verified-glossary-and-cross-references)

---

## 1. The Overview: Skills Three Ways, the Decoder, and the Boundary

### 1.1 The word "skills" is a three-way homonym

The request that produced this guide was, in effect, "ServiceNow skills and MCP." That phrase is ambiguous in a way that matters, because three different literatures are hiding inside one word. Getting the sense right decides which of the repo's guides the reader should actually be holding.

**Sense (a) — the platform term of art.** ServiceNow's own documentation defines generative AI **skills** as a first-class, named product concept: "ServiceNow AI Platform products contain generative AI skills that are tailored to meet the needs of users in different workflows." A skill is a shippable, workflow-specific generative-AI capability with a name, a prompt, an access-control list, a role restriction, and one or more deployment targets. It is not an informal label for "AI features"; it is a term the vendor uses to describe a buildable, publishable, permission-bearing artifact. **This is the sense this guide primarily addresses** (§3, and §4 for how a skill becomes an agent tool). *(ServiceNow docs, Generative AI skills, /r/intelligent-experiences/now-assist-skills/now-assist-skills.html, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

**Sense (b) — agent skills as a general packaging pattern.** Independently of ServiceNow, an "agent skill" in the wider industry means a bounded capability handed to an agent: a tool definition, an MCP tool, a callable function with a schema and a description the model can read. Any capability the LLM may invoke is a form of this. ServiceNow's own "Add tool → Generative AI skill" mechanism (§3.4) is precisely where sense (a) and sense (b) touch — a platform skill wrapped as an agent tool — but the general pattern is not ServiceNow-specific and is not owned here. It is owned by [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md) and [`ai_llm/agent_harness_engineering_guide.md`](ai_llm/agent_harness_engineering_guide.md).

**Sense (c) — human professional skills.** The competency, career and capability-gap sense — "what skills does a ServiceNow architect need?" — belongs to a completely different literature: role analysis, competency frameworks, and management practice. It is not a platform concept at all and has nothing to do with the agentic surface. It lives in [`../personal/skill_gaps_enterprise_architect_guide.md`](../personal/skill_gaps_enterprise_architect_guide.md), [`../personal/agent_runtime_engineer_skill_gaps_guide.md`](../personal/agent_runtime_engineer_skill_gaps_guide.md), [`../personal/data_architect_skillgaps_guide.md`](../personal/data_architect_skillgaps_guide.md) and the management guides [`../management/facilitation_skills_guide.md`](../management/facilitation_skills_guide.md) and [`../management/management_consulting_skills_guide.md`](../management/management_consulting_skills_guide.md).

### 1.2 Why the ambiguity is not pedantry

The three senses fail in different directions. Someone who reads (a) as (b) will assume a ServiceNow skill is a generic tool and miss that it **must** carry an ACL and role restrictions — the platform enforces that at publish time (§3.5). Someone who reads (a) as (c) will go looking for a training curriculum when the actual question was an access-control design. And someone who reads (b) as (a) will assume the vendor ships a specific skill when it does not, which is the exact class of error this repo treats as its worst defect: a product claim that a reader then takes into a procurement document. The rest of this guide is written to the vendor's own vocabulary — "skill," "AI agent," "agentic workflow," "MCP server," "AI User" — so that every noun can be checked against the source cited beside it.

### 1.3 The decoder

The names in this guide are drawn from three distinct provenance classes, and the class matters as much as the name. The table below is the decoder for every major term used in the body. Full sourcing and dates are in §15.

| Name | What it is | Class | Status / source |
|---|---|---|---|
| **ServiceNow Otto** | The current name of the vendor's generative-AI product family | Vendor product, rebrand | ✅ SDK docs, `servicenow.github.io/sdk`, v4.13.0, checked 1 Oct 2026 — "'Now Assist' has been rebranded to 'ServiceNow Otto'" |
| **Now Assist** | The **former** name of the same product | Vendor product, historical | ✅ Same SDK note: "These names refer to the same product"; technical identifiers unchanged |
| **Generative AI skill** | The platform's term of art for a named, workflow-specific generative-AI capability | Vendor product concept | ✅ ServiceNow docs, Generative AI skills, Brazil, 10 Sep 2026 |
| **AI Skill Kit** | The build surface for custom skills | Vendor product | ✅ ServiceNow docs, Exploring AI Skill Kit (tree folder retains historical name `now-assist-skill-kit`) |
| **AI Agent Studio** | The unified interface to create, manage and test AI agents and agentic workflows | Vendor product | ✅ ServiceNow docs, AI Agent Studio overview, Brazil, 10 Sep 2026 |
| **AI Agent Orchestrator** | The coordinator that manages multiple AI agents across a workflow | Vendor product | ✅ ServiceNow docs, Understand AI agents, Brazil, 10 Sep 2026 |
| **Dynamic Orchestrator** | The mechanism that narrows which agents are considered in planning when many are available | Vendor product mechanism | ✅ Same page — documented at "more than the ideal 8 to 10 AI agents" |
| **AI User** | A real `sys_user` record with `identity_type = ai_agent`, selectable as an agent's `runAsUser` | Vendor platform object | ✅ ServiceNow SDK, Building AI Agents Guide |
| **Dynamic User** | The alternative run-as model: the agent inherits roles from the invoking user | Vendor platform model | ✅ ServiceNow SDK — `dataAccess.roleMap` / `roleList` |
| **MCP Server Console** | The vendor surface that exposes ServiceNow functionality to external MCP clients | Vendor product | ✅ ServiceNow docs, MCP Server Console landing, Brazil, 10 Sep 2026 |
| **MCP Client** | The vendor surface that lets ServiceNow agents consume externally hosted MCP tools | Vendor product | ✅ ServiceNow docs, MCP Client landing, Brazil, 10 Sep 2026 |
| **AI Control Tower / AI Gateway** | The governance path: MCP server intake → AI Steward approval → client registration | Vendor product | ✅ ServiceNow docs, ai-control-tower/* |
| **AI Guardian** | Runtime protection for generative AI: offensive content, prompt injection, sensitive topics | Vendor product | ✅ ServiceNow docs, AI Guardian, Brazil, 10 Sep 2026 |
| **Workflow Data Fabric** | An integrated data layer unifying business and technology data for workflows and agents | Vendor product | ✅ ServiceNow press release, 23 Oct 2024; docs family present at Brazil/2026 |
| **RaptorDB Pro** | The high-performance database named in the same release as powering Workflow Data Fabric | Vendor product | ⚠ Press release 23 Oct 2024 only — zero hits in the docs map retrieved this pass |
| **"Agent Fabric"** | A name in wide circulation that could not be traced to ServiceNow documentation | **Unverified** | ⚠ Not found in docs TOC or web search, 1 Oct 2026 — see §16.1 |
| **"Action Fabric"** | Title of a ServiceNow **Community** article linked from the MCP Server Console docs page | Community-sourced | ⚠ Not a product name in the documentation tree |
| **Community/third-party MCP servers** | Independent GitHub projects that also speak MCP to ServiceNow | Third-party, not vendor | ⚠ e.g. `jschuller/mcp-server-servicenow`, `echelon-ai-labs/servicenow-mcp` — see §5.6 |

### 1.4 What this guide covers, and what it deliberately does not

**In scope, and owned here:** the three-way "skills" disambiguation; ServiceNow's own generative-AI skill concept and its build/security lifecycle; the agent layer (studio, workflows, orchestrator, typed tool catalogue); the MCP path in both directions *as it is specific to ServiceNow*; and the controls consequence of an agent that can read and write the system of record.

**Out of scope, and named rather than re-derived:**

- **The MCP protocol itself** — transports, discovery, progressive disclosure, general integration patterns: [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md), [`ai_llm/mcp_discovery_guide.md`](ai_llm/mcp_discovery_guide.md), [`ai_llm/mcp_progressive_disclosure_guide.md`](ai_llm/mcp_progressive_disclosure_guide.md), with the selection frame in [`ai_llm/ai_agent_platform_selection_guide.md`](ai_llm/ai_agent_platform_selection_guide.md) and the architecture frame in [`ai_llm/enterprise_agentic_platform_architecture_guide.md`](ai_llm/enterprise_agentic_platform_architecture_guide.md). This guide explains only what changes when the endpoint is ServiceNow.
- **ITSM/ITIL process theory** — incident, problem, change, request, knowledge as *process*: [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md), with capacity in [`capacity_sizing_guide.md`](capacity_sizing_guide.md). This guide treats those records as *artifacts and evidence*, not as process design.
- **AIOps and AI-for-IT assistive capability** — including Otto/Now Assist incident summarisation, resolution notes, chat summarisation, AI search and flow generation: [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) **§6.2** is the assistive-layer owner. This guide cites it and covers the **agentic** surface beyond summarisation.
- **The operations bridge** — ServiceNow ITOM/AIOps Event Management and the CMDB as an observability integration source: [`opentext_operations_bridge_guide.md`](opentext_operations_bridge_guide.md).
- **Agentic workflow engines as a discipline** — [`agentic_workflows_guide.md`](agentic_workflows_guide.md), [`durable_ai_agent_workflows_guide.md`](durable_ai_agent_workflows_guide.md).
- **The foreign-integration vantage point** — ServiceNow as a knowledge source and connector target from a non-ServiceNow agent platform: [`ai_llm/copilot_studio_agent_engineering_knowledge_guide.md`](ai_llm/copilot_studio_agent_engineering_knowledge_guide.md) **§4**. Same estate, opposite direction of travel.
- **Regulatory and governance frames** — [`../banking/grc_in_banking_guide.md`](../banking/grc_in_banking_guide.md), [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md), [`audit_as_code_guide.md`](audit_as_code_guide.md), [`data_governance_guide.md`](data_governance_guide.md), [`api_governance_guide.md`](api_governance_guide.md). No jurisdiction's rule is asserted here.

A note on the vendor's availability language: the ServiceNow documentation carries its own dated availability limits for AI features — for example that not all model providers are available for customers with in-country SKUs, that some AI products and features are currently unavailable for customers in certain restricted data centres or self-hosted environments, that some are available only in some regions, and that "some AI products and skills are not available in Regulated Markets" (KB2593939). *(ServiceNow docs availability notes, Brazil, checked 1 Oct 2026 ✅)* Those are the vendor's own statements, recorded as dated availability notes. They are **not** a roadmap, and they are **not** an assertion about any jurisdiction's rules.

### 1.5 The thesis, stated once

Everything from §6 onward is developed from a single sentence, which is the reason this guide exists in a banking repository rather than a product-catalogue one:

> **Connecting an agent to the system of record is an integration project; letting it write there is a controls decision.**

The integration half is engineering: authenticate, expose a tool surface, handle failure, version it. The controls half is not: an incident record and its timeline, a change record and its approval trail, and a configuration item in the CMDB are **evidence** — the thing an auditor reads and a regulator asks for. A platform an agent can write to is a platform whose evidence has a new author. §6 establishes that the platform itself offers a first-class named non-human identity (so the shared-account problem is a builder's choice, not a platform constraint); §7 works through evidence integrity and segregation of duties; §8 takes the CMDB as the sharpest case; §9 turns all of it into a ladder of guardrails, each rung with an explicit cost. None of §6–§9 is vendor criticism — it is architecture and controls.

> Cross-reference: the protocol half is [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md); the process half is [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md); the assistive AI half is [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2.

---

## 2. The Platform in One Page: What ServiceNow Is and Is Not

### 2.1 What it is

ServiceNow is a **configurable workflow and record platform**. Work arrives, or is raised, as a record; the record has a type, a state, a timeline, fields, an owner and a set of approvals; workflows move records between states according to configuration rather than compiled code; and everything — the record, the state change, the approval, the comment — is retained on the instance as a form of history. The generative-AI and agentic layers described in §3–§5 sit *on top of* this record platform. That ordering is the whole architecture: the AI surface is a client of the record model, not a replacement for it.

### 2.2 The principal record families

The record families this guide cares about, because they are the tables an agent would read from or write to:

| Record family | What the record represents | Why an agent might touch it |
|---|---|---|
| **Incident** | A disruption or degradation as experienced, with a timeline of actions and assignments | Triage, summarisation, draft resolution notes, status updates |
| **Problem** | The underlying cause behind incidents | Drafting root-cause narratives, linking incidents |
| **Change** | A planned alteration to the estate, with an approval trail | Drafting change records, risk assessment input, scheduling |
| **Request** (and its catalogue items) | A standard ask fulfilled through a catalogue | Fulfilment steps, status messaging |
| **Configuration item (CI)** | An element of the estate and its relationships — the CMDB's unit | Creation, enrichment, de-duplication, health |
| **Knowledge article** | Reusable guidance published for humans and, increasingly, retrieved by agents | Drafting, retrieval, currency checks |
| **Case** (customer/HR service domains) | A service interaction in a non-IT or customer-facing domain | Summarisation, routing, response drafting |

A **CI** is, in the vendor's own words, "a representation of some type of infrastructure, application, software, hardware," and CIs "belong to classes" — documented example classes include Windows Server, Data Center, Internet Cable, Apache Application, and Virtual Machine. *(ServiceNow docs, CMDB CI creator AI agent, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* This is the definition the CMDB argument in §8 rests on.

### 2.3 What it is not

State the negative just as clearly, because most agentic overreach starts with a category error:

- **Not a data warehouse.** It holds operational records and their relationships. Analytical aggregation over the estate is a different system's job.
- **Not a monitoring tool.** It receives and manages events and records; it is not the telemetry source of truth. The pattern by which observability platforms feed ServiceNow ITOM/AIOps Event Management and reference its CMDB is described in [`opentext_operations_bridge_guide.md`](opentext_operations_bridge_guide.md).
- **Not a code repository.** Configuration and customisation live in the instance; software version control is elsewhere. Agent *version control* (§4.5) is a platform concept for agents, not a general source-control substitute.
- **Not a process authority.** It implements a process model — it does not define what a correct process is. The ITSM/ITIL frame is [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md). A platform will happily let you configure a bad process.

### 2.4 Why the negative list matters for agents

Each "is not" is a boundary on what an agent on this platform can legitimately be asked to do. An agent that reaches into ServiceNow to answer an analytical question is using the wrong system. An agent that reaches in to manufacture the *appearance* of process compliance — closing records to make a state look right — is worse than wrong; it is the evidence-integrity problem of §7 arriving early. The platform's value to an agent is that it holds a *governed record of what happened and who decided*; anything that erodes that specificity is a net loss even if the workflow looks faster.

> Cross-reference: the process model these records implement is [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md); the observability relationship is [`opentext_operations_bridge_guide.md`](opentext_operations_bridge_guide.md); the AI-for-IT application of these records is [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md).

## 3. The AI Surface ServiceNow Ships: Otto, Skills, and What They Do

### 3.1 The name, first, because procurement documents will contain both strings

The current name of ServiceNow's generative-AI product family is **ServiceNow Otto**. The former name is **Now Assist**. This is not two products and not a migration: ServiceNow's own SDK documentation states plainly that **"'Now Assist' has been rebranded to 'ServiceNow Otto.' These names refer to the same product. All technical identifiers (table names, field names, string literals like 'Now Assist Panel', `now_assist_deployment`) remain unchanged."** *(ServiceNow SDK docs, `servicenow.github.io/sdk`, version 4.13.0, checked 1 Oct 2026 ✅)*

The current documentation tree uses **ServiceNow Otto** throughout — "ServiceNow Otto panel," "ServiceNow Otto context menu," "ServiceNow Otto for ITSM," "Install ServiceNow Otto AI Agents" — while material from 2024 and earlier says "Now Assist." A bank's procurement, audit and architecture documents will contain **both** strings for years, because the technical identifiers did not change with the brand. The implication for anyone reconciling a vendor inventory: **the two names to reconcile are Otto and Now Assist, and they reconcile to one product.** Anyone treating them as separate platforms is constructing an inventory defect.

### 3.2 What a "skill" actually is — the sense (a) definition

The vendor's definition: **"ServiceNow AI Platform products contain generative AI skills that are tailored to meet the needs of users in different workflows."** *(ServiceNow docs, Generative AI skills, /r/intelligent-experiences/now-assist-skills/now-assist-skills.html, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

A skill, then, is a **named, workflow-specific generative-AI capability** — a shipped unit with a purpose, not a general chatbot. The documented example skills relevant to this guide's estate:

| Skill (documented name) | Domain | What it is for |
|---|---|---|
| **Configuration item (CI) summarization** | CMDB | Produces a summary of a CI |
| **Manage duplicate CIs** | CMDB | Works the duplicate-CI problem |
| **Service Graph Connector diagnosis** | CMDB | Diagnoses connector behaviour feeding the CMDB |
| **Alert analysis** | ITOM | Analyses an alert |
| **Alert investigation** | ITOM | Investigates an alert |
| **Analyze service health** | ITOM | Reasons over service health |
| **Incident summarisation** | ITSM | Summarises an incident — the assistive-layer case owned by `ai_for_it_guide.md` §6.2 |

*(All rows: ServiceNow docs, Generative AI skills, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

### 3.3 The documented behaviour of skills — domain, data and default state

Three documented properties of how skills behave on the instance, each with a control consequence:

1. **Domain scope.** Skills exist in the **global domain by default**; in **domain-separated** environments a skill only uses data in **the user's domain**. *(ServiceNow docs, Generative AI skills, Brazil, 10 Sep 2026 ✅)* For a bank running domain separation, this means a skill's reach is bounded by the invoking context — a fact worth testing rather than assuming (§12.3 "access tests").
2. **Data locality.** Data resides on the instance, and the **shared generative-AI services do not persist prompts or responses**. *(Same source ✅)* This is the vendor's own statement about its shared service; record it as a documented vendor claim and confirm it against the estate's own data-handling obligations — the guide does not assert what any regulator requires.
3. **Default-on state.** Some skills, AI agents and agentic workflows are **ON BY DEFAULT**. *(Same source ✅)* This is the single most operationally important line in §3: a capability can exist and act without a project having "deployed" it. Inventorying what is enabled is a controls task, not an optional one.

### 3.4 A skill becomes an agent tool — the bridge between sense (a) and sense (b)

In **AI Agent Studio → Create and manage → AI agents → "Add tools and information,"** the **"Add tool"** drop-down includes **Generative AI skill**. *(ServiceNow docs, Add a generative AI skill to an AI agent, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* The form captures:

- **Name**;
- **Description** — and the documentation is explicit about why it matters: *"This description is sent to the large language model (LLM)"*;
- **Select skill** — the platform skill being wrapped;
- **Inputs** — filled by the LLM at runtime **unless overridden**;
- **Execution mode: Supervised | Autonomous** — *Supervised* means human input is required during execution; *Autonomous* means none;
- **Display output**, plus processing and completion messages.

Two preconditions are documented and both are controls: the skill **must be published and activated first**, and **"the user the AI agent is running as must pass the ACL of the skill."** Role required: **`sn_aia.admin`**. *(Same source ✅)*

This is where the two senses of "skill" meet, and the meeting point is entirely a permissions question: a platform skill (sense a) becomes an invokable agent tool (sense b), and the agent's run-as identity must satisfy the skill's ACL. §6 is about that identity.

### 3.5 The skill security model — ACLs are mandatory

The vendor's rule: **"You must define an access control list (ACL) and role restrictions for all skills."** *(ServiceNow docs, Configure security controls for a skill, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* Documented options:

- **Any authenticated user** — the broadest setting;
- **Select roles** — a user needs only **one** of the selected roles, because these are **"Allow If" ACLs**.

There is a documented grandfathering behaviour worth knowing: if an existing skill has **no ACL**, execution is **not disrupted** — but on **edit and republish an ACL becomes mandatory**. *(Same source ✅)* So an estate can carry skills with no declared access control today and be forced to declare one the moment anyone touches them. That is a maintenance event with a security outcome, and it is the kind of thing an inventory should surface before it happens by accident.

### 3.6 Building custom skills — AI Skill Kit

**AI Skill Kit** is the build surface: **"Use AI Skill Kit to create custom skills when base system Otto skills don't fit your needs."** *(ServiceNow docs, Exploring AI Skill Kit, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* The documented workflow is four steps: **define the provider → build the prompt → test the prompt → deploy the skill.**

**Deploy targets** (documented): **ServiceNow Otto panel**, **ServiceNow Otto context menu**, **Virtual Agent**, **Flow Action**, **UI Action**. *(Same source ✅)* Roles: the skill **AI developer** role requires **`sn_skill_builder.admin`**; activation is performed by the **Otto admin** with the **`admin`** role. One page in the same documentation tree renders the developer role as **`snskillbuilder.admin`** instead — a spelling inconsistency the source itself carries, flagged as ⚠ and recorded in §16.1, and a reason to verify the exact role string against the estate's instance rather than trusting a single retrieved page.

The structural takeaway, and it is the sentence to carry into any design review: **a skill is fundamentally a prompt, a deployment target, and a set of security controls.** Nothing about a skill is magic; the risk is entirely in which of the three is under-specified.

### 3.7 What belongs to the assistive layer, and where its owner is

The skills above — incident summarisation, resolution notes, chat summarisation, AI search, flow generation — are the **assistive** layer: they draft, summarise, search and generate for a human who remains the actor. That analysis, including Now Assist/Otto for ITSM, is **owned by [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2** and is **not re-derived here.** This guide's contribution begins where assistive summarisation ends: at the point where the platform's own components **take action** — the agent layer of §4 and the MCP path of §5 — and at the controls consequence of that shift, which is §6 onward.

> Cross-reference: the assistive layer's product analysis is [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2; skills as a general packaging pattern are [`ai_llm/agent_harness_engineering_guide.md`](ai_llm/agent_harness_engineering_guide.md); the ITSM process context for summarisation is [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md).

---

## 4. The Agent Layer: AI Agent Studio, Workflows, and the Orchestrator

### 4.1 AI Agent Studio

**AI Agent Studio** is the vendor's unified surface: it **"enables you to create, manage, and test AI agents and agentic workflows within a unified interface."** *(ServiceNow docs, AI Agent Studio overview, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* It requires installing the **ServiceNow Otto AI Agents** plugin to enable the agentic AI experience. *(ServiceNow docs, Install ServiceNow Otto AI Agents ✅)*

Documented pages, each with a distinct controls role:

| Page | What it does | Controls relevance |
|---|---|---|
| **Overview** | Templates and ready-made workflows; recent activity; a journey checklist | Where an unowned capability can appear by default |
| **Create and manage** | Two tabs (agents, workflows); Guided Setup; duplicate/edit/delete | The build surface; duplication is how capability sprawl starts |
| **Activity** | Execution logs for agents and agentic workflows, **filterable by version** | The version-control surface and the reasoning-trace store |
| **Testing** | Manual execution tests **plus access tests** to validate security controls, and automated agentic evaluations | The only documented place access is *proved* rather than assumed |
| **Settings** | Enable **AI Guardian** (offensiveness detection, prompt-injection decision, long-term memory for agents) | Runtime protection, §9 rung 6 |
| **AI Agent Analytics dashboard** | Analytics across agents | Monitoring, not a control by itself |

There is also documented **Version control for AI agents and agentic workflows**, and an agent carries a `versionDetails[]` array whose `state` is one of `'draft' | 'committed' | 'published' | 'withdrawn'`. *(ServiceNow SDK docs, `AiAgent` API ✅)* Those four states are the retirement mechanics §12.4 uses.

### 4.2 AI Agentic Workflows and the Orchestrator

**AI Agentic Workflows** orchestrate **"multiple agents as a team"** and have a documented **Deployment Order**. *(ServiceNow SDK docs ✅)* Above them:

**AI Agent Orchestrator** — **"The AI Agent Orchestrator centrally manages multiple AI agents, coordinating their collaboration to complete complex workflows."** *(ServiceNow docs, Understand AI agents, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* Its documented roles:

- **Task coordination and execution** — correct sequencing;
- **Dynamic decision-making** — invoke AI models dynamically per scenario;
- **Seamless enterprise integration** — the docs say it **"Works with ServiceNow workflows, CMDB, and third-party APIs"**;
- **Multi-step workflow handling** — chained execution across agents;
- **Policy and governance enforcement**.

Documented mechanisms:

- a **ReAct prompt** merging reflection with planning, with a **Route** scheduling mode enabling **asynchronous parallel tool execution**;
- a **Dynamic Orchestrator** which, **"when there are more than the ideal 8 to 10 AI agents available for an agentic workflow,"** identifies which agents to consider in the planning phase;
- **User Impersonation**, and **Virtual Agent integration** for requester/fulfiller visibility.

*(All of the above: ServiceNow docs, Understand AI agents, Brazil, 10 Sep 2026 ✅)*

**Read the "policy and governance enforcement" bullet carefully.** It is documented, and it is the orchestrator's claim about *coordination* — sequencing, model selection, agent selection. It is **not** a claim that the orchestrator enforces segregation of duties, identity policy, or evidence integrity. Those remain the bank's architecture (§6–§8). The distinction between "the orchestrator enforces the plan" and "the platform enforces the control" is exactly the kind of conflation this guide exists to prevent. *(That distinction is this guide's inference, labelled as such — the documentation states the orchestration role and does not make the control claim either way.)*

### 4.3 Agent structure — the record, not the marketing

From the SDK's `AiAgent` API and the *Building AI Agents Guide*, an AI Agent record carries:

- **`name`**, **`description`**, **`agentRole`**;
- **`agentType`** — `'internal' | 'external' | 'voice' | 'aia_internal'`;
- **`recordType`** — `'template' | 'aia_internal' | 'custom' | 'promoted'`, and it must be `'custom'` for user-created agents;
- **`channel`** — `'nap'` (ServiceNow Otto Panel only) or `'nap_and_va'` (panel plus Virtual Agent);
- **`versionDetails[]`** with `state ∈ 'draft' | 'committed' | 'published' | 'withdrawn'`;
- **`triggerConfig[]`** — triggers of type `record_create`, `record_create_or_update`, `record_update`, `email`, and scheduled/daily/weekly/monthly, where **each trigger needs a mandatory trigger condition and a mandatory trigger run-as**;
- **`memoryCategories`**;
- **`parent`** (agent hierarchy);
- **`protectionPolicy`** — `'read' | 'protected'`.

*(All of the above: ServiceNow SDK docs, `AiAgent` API; Building AI Agents Guide, checked 1 Oct 2026 ✅)*

Two facts here carry outsized weight:

1. **A mandatory trigger run-as.** An event-triggered agent cannot exist without declaring who it runs as. The platform therefore *forces the question* — §6 is about how it can still be answered badly.
2. **Agent-level `executionMode`: `"copilot"` vs `"autopilot"`.** The SDK's own guidance is that `"copilot"` is the preferred mode **"when the agent creates, updates, or deletes records,"** and `"autopilot"` is for **"read-only or fully autonomous workflows."** *(ServiceNow SDK docs ✅)* This is a **second, distinct** execution mode from the skill-as-tool **Supervised | Autonomous** setting in §3.4 — and both belong in the same guardrail ladder (§9). A record that is written in "copilot" mode with skills set "Autonomous" is a mixed posture; the ladder must account for both switches, not one.

### 4.4 The typed tool catalogue — including `mcp`

An agent's tools are **typed**. Documented out-of-the-box tool types include:

`crud`, `script`, `capability`, `subflow`, `action`, `catalog`, `topic`/`topic_block`, `search_retrieval` (the retrieval-augmented-generation tool), and the OOB types `web_automation`, `knowledge_graph`, `file_upload`, `deep_research`, `desktop_automation`, and **`mcp`**. *(ServiceNow SDK docs, Building AI Agents Guide ✅)*

The presence of **`mcp` as an out-of-the-box tool type** matters for §5: an agent on this platform can carry an MCP tool **natively**, without treating MCP as an external wrapper. It also matters for hazard A — this list is *the* verified list; any tool type a reader encounters elsewhere should be checked against the instance before being written into a design.

Documented table names for the agent layer: **`sn_aia_agent`**, **`sn_aia_agent_config`**, **`sn_aia_version`**, **`sn_aia_tool`**, **`sn_aia_agent_tool_m2m`**, **`sn_aia_usecase`**, **`sn_aia_trigger_configuration`**, **`sn_aia_external_agent_configuration`**. An **`agentDescriptor`** can be `'require_caller_id'`, `'created_by_ai_agent_advisor'`, or `'created_by_build_agent'`. *(ServiceNow SDK docs ✅)* Note the third value: it names a "build agent" as a *creator* of agents, but "Build Agent" does **not** appear as a title in the documentation tree retrieved this pass — an asymmetry recorded honestly in §16.1 rather than resolved.

### 4.5 AI Agent Advisor, and what "documented" versus "inferred" means here

**AI Agent Advisor** is a verified name, supported by the SDK's `'created_by_ai_agent_advisor'` descriptor and by documented pages including *AI Agent Advisor in AI Admin Center*, *...in AI Control Tower*, *...in AI Agent Studio*, and *Components installed with AI Agent Advisor*. *(ServiceNow docs ✅)* It is the platform's advisory surface for agent creation. This guide does not expand the mechanism beyond what is documented, because the advisory engine's internals are not in the sources retrieved.

**A note on how to read this section.** Everything in §4.1–§4.4 is **documented** and cited. Two statements are **inference**, explicitly labelled: (i) that the orchestrator's governance enforcement is coordination-scoped and does not by itself constitute a segregation-of-duties control (§4.2); and (ii) the reading of the dual execution-mode switches as a design smell when set inconsistently (§4.3). Everything else — including the tool-type list, the execution-mode guidance, the trigger requirement and the descriptor values — is quoted or paraphrased from the cited primary source. Keeping that line visible is the point.

> Cross-reference: the multi-agent patterns the orchestrator embodies — planning, routing, hierarchical coordination — are [`ai_llm/hierarchical_multi_agent_frameworks_guide.md`](ai_llm/hierarchical_multi_agent_frameworks_guide.md) and [`agentic_workflows_guide.md`](agentic_workflows_guide.md); the harness-level view is [`ai_llm/agent_harness_engineering_guide.md`](ai_llm/agent_harness_engineering_guide.md); the deliverables pattern is [`ai_llm/agentic_solution_artifacts_guide.md`](ai_llm/agentic_solution_artifacts_guide.md).

---

## 5. The MCP Path: Inbound Endpoint, Outbound Tools, and the Approval Lifecycle

### 5.1 The two directions, stated precisely

"MCP and ServiceNow" is two different architectures wearing one phrase, and conflating them is the most common error in this space:

- **Inbound** — ServiceNow is the **MCP endpoint**. External MCP clients call into a ServiceNow instance to reach its functionality through vendor-provided MCP servers. The surface is the **MCP Server Console**.
- **Outbound** — ServiceNow is the **MCP host/client**. Agents running in ServiceNow consume tools hosted **externally**, published via an MCP server. The surface is the **MCP Client**.

Both are vendor-provided. Neither requires a community project. And a reader who has seen a GitHub repository named `servicenow-mcp` may be looking at something that is neither of these — §5.6 exists to prevent exactly that confusion.

### 5.2 Inbound — the MCP Server Console

The vendor's description: **"The MCP Server Console enables secure and governed access to functionality on a ServiceNow instance for AI applications with Model Context Protocol (MCP) servers. MCP servers extend ServiceNow AI Platform® functionality into any external MCP client and employee experience over the Model Context Protocol."** *(ServiceNow docs, /r/intelligent-experiences/mcp-platform-manager-landing.html, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

Documented feature set: **Model Context Protocol integration**; **configuration and management** (create and configure MCP servers through the console); **connectivity** (MCP clients call MCP servers); and **resources**. The console's own structure is documented as **Explore → Configure** (create an MCP server and configure its tools) **→ Connect** (call it from an MCP client) **→ Reference**. *(Same source ✅)*

There is a **Model Context Protocol Server listing in the ServiceNow Store**, an **MCP Server Console FAQ**, and a **"Designing your first custom ServiceNow MCP server with MCP Server Console"** article in the ServiceNow Community. *(Same source, linked as helpful resources ✅)*

The two tool-surface facts that matter most:

1. **An MCP server's tools are configured.** The console documents configuring them — the tool surface is a **deliberate artifact**, not an automatic mirror of the instance's API.
2. **A server is governed by access rules / ACLs.** Documented security controls: role **`sn_mcp_client.admin`**; you **"select which users can access this MCP server via ACLs."** The ACL options created in AI Agent Studio are **Any authenticated user** | **Users with specified roles** (the default — and, as in §3.5, these are **"Allow If"** ACLs, so a user with at least one of the roles passes) | **Public**. The documentation's own words about the last option: **"Grants access to all users, including guests who aren't signed in"** — and it says this **"should be used sparingly and only when needed."** *(ServiceNow docs, Define security controls for MCP Servers, Brazil, 10 Sep 2026 ✅)*

Authentication, as documented: **OAuth 2.1** and **API key** server options, plus **Integrating MCP server with third-party identity providers** and **Create an OAuth inbound integration for an MCP client**. *(Same source ✅)* The third-party-IdP article is the one to read first in a bank, because it is the difference between an MCP server whose callers are the bank's managed identities and one whose callers are API keys with no human behind them — a distinction §6 and §7 turn on. Documented MCP Server Console integrations include **AI skill support**, **Knowledge Graph schema support**, and **Domain separation and MCP Server Console**. *(Same source ✅)*

### 5.3 What a ServiceNow MCP endpoint should expose — and what it must not

The console lets you choose. That choice is the security design, and it should be argued from the record families (§2.2) and the **write scope**, not from convenience.

| Surface class | Example shape | Control object |
|---|---|---|
| **Read / query / summarise** | Retrieve an incident by number; query CIs in a class; summarise a CI; search knowledge | A data-disclosure object. The question is *what data may leave the instance to which client*. |
| **Draft / propose** | Compose a proposed change description; propose a resolution note for human review | A *suggestion* object. Nothing in the record changes; the human still commits. |
| **Create / update / close** | Raise an incident; update a CI; transition a change state; close a record | A **write-to-the-system-of-record** object — a different control class entirely (§7). |

The argument: **a summarise/read/query surface is a different control object from a create/update surface**, and the two should not be bundled into one MCP server just because they share a data source. Bundling them means the client that was approved to *read* is now the client that can *write*, and the ACL option "the client has access" stops distinguishing between them. The tiering above is this guide's architectural recommendation; the *mechanisms* it rests on — configured tools, ACL options, OAuth 2.1 or API key, third-party IdP — are all documented. There is no printed vendor guidance that says "tier your tool surface this way"; that is the bank's design decision, and §9's guardrail ladder is where it gets priced.

### 5.4 Outbound — the MCP Client

The vendor's description: **"The ServiceNow Model Context Protocol Client (MCP Client) allows you to access the Model Context Protocol tools hosted externally and published via an MCP Server in the ServiceNow AI Agent Studio."** *(ServiceNow docs, mcp-client-landing.html, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* It is configured in **AI Agent Studio**, and agents can add MCP servers as tools — documented as *Add an MCP server from AI Agent Studio* and *Adding an MCP Server in AI Agent Studio*. *(Same source ✅)* Recall from §4.4 that the SDK's tool catalogue carries **`'mcp'` as an out-of-the-box tool type** — so an agent can hold an MCP tool as a first-class tool, not a bolt-on.

The outbound direction is where the bank's *own* controls gap appears most sharply: an agent in ServiceNow calling an external MCP server is **egress**. The external server's identity model is not the bank's, and the data the agent sends it may be record content. That is a data-governance question, cross-referenced to [`data_governance_guide.md`](data_governance_guide.md), and an API-governance question, cross-referenced to [`api_governance_guide.md`](api_governance_guide.md) — not re-derived here.

### 5.5 The governance path — intake, AI Steward, client registration

This is the strongest structural evidence for this guide's thesis, because it shows the vendor does not treat an MCP connection as plumbing.

Documented flow *(ServiceNow docs, ai-control-tower/*, roles `sn_ai_governance.ai_steward`, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*:

**MCP server intake → AI Steward playbook approval → client registration and AI Gateway setup.**

Intake has **three documented routes**: **automatic discovery** (add from AI Agent Studio), the **MCP Catalog**, or **manual registration from AI Control Tower**. And the vendor's own words: **"The AI Steward moves the MCP server through the onboarding lifecycle"**, and **"After the MCP server is approved, register the clients that connect to it through AI Gateway."** Related documented artifacts: the **MCP server approval playbook workflow**, **AI Gateway**, **Connect to MCP servers via AI Gateway**, **Add a Global MCP client**, and the **MCP server record**. *(Same source ✅)*

Read architecturally: the vendor's own model makes connecting an MCP server a **named-role-approved, lifecycle-managed, catalogued** event, with the client registration happening *after* approval. That is precisely the split in §1.5 — the integration is plumbing, the *decision* is governance. **AI Control Tower** and **AI Risk and Compliance** provide the lifecycle-governance half; **AI Guardian** provides the runtime half (§5.7). Roles `sn_ai_governance.ai_steward` and `sn_mcp_client.admin` are the two names to look for in a bank's RACI for this.

### 5.6 Vendor-provided versus community-maintained — the error to avoid

**Verified community / third-party examples found 1 Oct 2026** (all **community-maintained, not ServiceNow products**, marked ⚠):

- **`github.com/jschuller/mcp-server-servicenow`** — a developer data plane over ServiceNow via the Table API, schema, aggregates and update sets. Its own README says it **"Runs alongside ServiceNow's native MCP Server"** — that is a **third-party claim**, marked ⚠, not vendor documentation.
- **`github.com/echelon-ai-labs/servicenow-mcp`** — a community MCP server project.
- A **`habenani-p/servicenow-mcp`** listing ("NowAIKit").
- A **`leasstatt/servicenow-mcp-ai`** listing.
- An **"Enterprise MCP Documentation"** third-party page.

*(All ⚠ — third-party listings, not traced to ServiceNow's own documentation.)*

**The single most likely error a reader will make is to treat one of these as the vendor product, or the vendor product as a community one.** Both directions of the mistake are harmful: the first writes an unmanaged third-party component into a procurement inventory as if it carried vendor support; the second dismisses the vendor's own governed surface as "just a GitHub project" and bypasses the AI Steward path in §5.5. The dividing line is simple and checkable — **if it is not in ServiceNow's own documentation, it is not ServiceNow's product**, whatever its README says.

### 5.7 Status, dated and stated carefully

The inbound MCP Server Console and the outbound MCP Client are **documented, shipped, vendor-provided product surfaces in the Brazil release family**. They are **not** community projects, and the documentation does **not** describe them as a developer preview. The documentation itself does not carry a GA/preview **label** at all — so this guide will not invent one. What can be said precisely, and is: *documented as a shipped feature of the Brazil release; the documentation does not carry a GA/preview label; last checked 1 Oct 2026.* ⚠ *on the label question specifically, ✅ on the documentation's existence and content.*

Alongside status, record the vendor's **dated availability limits** exactly as dated constraints, not as roadmap (§1.4): not all model providers available for customers with in-country SKUs; some AI products/features currently unavailable for customers in FedRAMP, NSC DOD IL5, or Australia IRAP-Protected data centres, self-hosted customers, and other restricted environments; some available only in some regions; and some AI products and skills not available in Regulated Markets (KB2593939). *(ServiceNow docs availability notes, Brazil, checked 1 Oct 2026 ✅)* These are the vendor's own words about **today**, not a promise about tomorrow, and they are not an assertion of any jurisdiction's rules. Adding a versioning note: the SDK documentation itself is versioned (v4.13.0 at the time of the Otto rebrand note), and MCP endpoint tool surfaces change with configuration — so an MCP integration's real "version" is *the configuration at a point in time*, which is an operating-model fact (§12).

> Cross-reference: the Model Context Protocol itself — discovery, transports, progressive disclosure — is [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md), [`ai_llm/mcp_discovery_guide.md`](ai_llm/mcp_discovery_guide.md) and [`ai_llm/mcp_progressive_disclosure_guide.md`](ai_llm/mcp_progressive_disclosure_guide.md); platform selection is [`ai_llm/ai_agent_platform_selection_guide.md`](ai_llm/ai_agent_platform_selection_guide.md); the opposite-direction view of ServiceNow as an external knowledge source is [`ai_llm/copilot_studio_agent_engineering_knowledge_guide.md`](ai_llm/copilot_studio_agent_engineering_knowledge_guide.md) §4; API and data governance frames are [`api_governance_guide.md`](api_governance_guide.md) and [`data_governance_guide.md`](data_governance_guide.md).

## 6. The Identity Question: AI User, Dynamic User, and Impersonation

### 6.1 The three layers where identity is decided

An agent's "identity" on ServiceNow is not one setting. It is at least four, and they answer different questions:

| Layer | Question it answers | Documented mechanism |
|---|---|---|
| **Invocation ACL** (`securityAcl`) | **Who may call this agent?** | `'Any authenticated user'` \| `'Specific role'` \| `'Public'` |
| **Run-as identity** | **Who does the agent act as?** | `runAsUser` on an agent; `runAs` on an agentic workflow |
| **Data-access roles** | **What data may it reach, and what may it do?** | `dataAccess.roleMap` (role names) / `dataAccess.roleList` (role sys_ids) |
| **Trigger run-as** | **Who does it act as when it fires on an event?** | Mandatory per-trigger run-as (§4.3) |

*(All rows: ServiceNow SDK docs, Building AI Agents Guide and `AiAgent` API, checked 1 Oct 2026 ✅)*

A design review that asks only "does it have access?" has asked one of four questions. The controls work is in specifying all four and in keeping them consistent.

### 6.2 The two documented run-as models — and why this is the most important verified material in the guide

ServiceNow documents **two** models for who an agent runs as.

**(i) AI User — a first-class, named, non-human identity.** An **AI User** is an actual `sys_user` record whose `identity_type` is **`ai_agent`**. The SDK instructs the builder: *"Query `sys_user` (encodedQuery: `identity_type=ai_agent`) to find available AI Users."* The selected user is then set as **`runAsUser`** on an agent, or **`runAs`** on an agentic workflow. The SDK's own guide asks the builder to choose explicitly: **"Which type of user should this run as: (1) Dynamic User OR (2) AI User?"** *(ServiceNow SDK docs, Building AI Agents Guide, /sdk/guides/building-ai-agents-guide, checked 1 Oct 2026 ✅)*

**(ii) Dynamic User — inherited identity.** The agent **inherits roles from the invoking user** via **`dataAccess.roleMap`** (role names) or **`dataAccess.roleList`** (role sys_ids). The SDK notes that these data-access controls are **"required when `runAsUser` is not set."** *(Same source ✅)*

The consequences are opposite in kind, and this is the pivot of the whole guide:

- Under **Dynamic User**, the agent's authority is *the human's* authority. It can do what the caller can do, no more — and the audit trail shows the human. This is a strong control posture with a real usability cost: the agent cannot do anything the caller cannot.
- Under **AI User**, the agent acts as a **named machine identity** with its own roles. This is *also* a strong control posture — the trail shows *which* agent — provided that identity is **one per agent purpose** and its roles are scoped.

Neither model produces a shared account. **A shared integration account is not one of the two documented models.** It is a builder's choice to construct a service account and hand it to every agent, or to run many agents under one `runAsUser`. That choice is not on the platform's menu; it is grafted on. This is the finding §6.5 states plainly.

### 6.3 Impersonation and delegated access — the user-facing case

The documentation describes **User Impersonation** and Virtual Agent integration (Vendor docs, Understand AI agents ✅). The user-facing case is documented as: **"The agentic workflow executes tools as the logged-in user in the ServiceNow Otto panel... After impersonation is enabled, testing an AI agent uses the instance-level impersonation."** *(ServiceNow docs, Understand AI agents, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* This is the same underlying idea as a Dynamic User in the interactive channel: the requester's own permissions bound what the agent can reach when it acts on their behalf. It is the right default for a self-service surface, and the wrong default for a back-office capability that must act when no human is present — which is exactly where the trigger run-as and the AI User model re-enter.

### 6.4 Per-agent role sets — the two sets the CMDB page documents

The CMDB CI creator agent page documents, per agent, **two distinct role sets**:

- **"Allowed user roles"** — e.g. `sn_cmdb_editor, snc_internal` — **who may access the agent** (the invocation side);
- **"Data access roles"** — e.g. `sn_cmdb_editor` — **which data the agent can reach and what actions it can take** (the authority side).

Plus per-agent flags:

- **"Allow third party to access this AI agent"** — stored on `sn_aia_agent_config` as **External discoverable**, default **off**;
- **"Allow AI specialists to access this AI agent"** — **Specialist enabled**, default **off**;
- **"Manage long-term memory"** — property `sn_aia.ltm.enable_long_term_memory`.

*(ServiceNow docs, CMDB CI creator AI agent, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

**Two role sets, not one, is the architectural fact to internalise.** The invoke question ("who may call it?") and the authority question ("what may it touch?") are separate fields on the record, and collapsing them into one role is how an estate ends up with an agent whose callers are narrow but whose reach is broad. The defaults on the two exposure flags are both **off** — a good default, and one that only holds if nobody flips it without a reason recorded.

### 6.5 The finding — shared accounts, and why capability is irrelevant

> **An agent acting under a shared integration account destroys the meaning of the audit trail regardless of how capable the agent is.**

The reasoning is not about the agent's competence. It is about what the record *says*. If ten agents — or one agent acting for ten teams — run as one service account, then:

- **Attribution collapses.** A record written by that identity says "the integration account did this." It does not say which agent, which version, which trigger, or which intent. The trail is present but uninformative.
- **Approval separation disappears.** If the same shared identity can raise a change and approve a change, then any segregation of duties the process design intended has no enforcement surface — one login does both, and the log shows one actor (§7.4).
- **Blast radius is the union of every consumer.** Least privilege is computed across all agents sharing the account, so it is least privilege for none of them.
- **Revocation is all-or-nothing.** Retiring one agent means either leaving its authority live for the others or breaking them all.

This is the same finding the repo's zero-trust guides reach about shared credentials in general — [`zero_trust_legacy_estate_guide.md`](zero_trust_legacy_estate_guide.md), [`zero_trust_mainframe_guide.md`](zero_trust_mainframe_guide.md), [`zero_trust_network_architecture_guide.md`](zero_trust_network_architecture_guide.md) — reapplied at the agent layer rather than re-derived.

**And the constructive half, which is the reason this is architecture and not vendor criticism:** the platform already offers a **named non-human identity** — the AI User, a real `sys_user` with `identity_type = ai_agent`, selectable as `runAsUser`. The named identity is not something a bank must invent; it is documented, first-class, and supported by the per-agent role model in §6.4. So the shared-account problem is a **builder's choice on this platform, not a platform constraint** — the risk is architectural (the audit trail's author collapses to one anonymous actor), not a missing feature. Fixing it is a design decision, taken at build time, costing a small amount of per-agent administration. Nobody who says "we had to run them all as one account" has been forced to; they have chosen to.

### 6.6 Guardrail facts that appear in the same SDK material

Two documented restrictions worth carrying into any agent build:

- The **`maint` role is NOT allowed** for agents, and the SDK instructs the builder to reject it.
- **`security_admin` is allowed, with an explicit warning** in the documentation about its broad privilege.

Out-of-the-box role rosters are constrained in the SDK's deployment patterns — for example, the ACL deployment pattern requires an **`$id`**, and specific-role ACLs need a **`roles` array**. *(ServiceNow SDK docs, checked 1 Oct 2026 ✅)* These are the kind of small, verifiable constraints that decide whether a build ships with the intended scoping or with a fudge.

### 6.7 The design rule

**Name the identity, or you have not deployed a controlled agent.** Every agent and every agentic workflow should be answerable, from its own record, to the four questions in §6.1 — who may call it, who it runs as, which data roles it holds, and who it runs as when trigger-fired. If the answer to "who does it run as?" is a shared account, the correct state is not "deployed, controls pending"; it is a design defect with a documented fix (AI User, one per purpose) available on the platform.

> Cross-reference: the shared-credential and zero-trust reasoning is [`zero_trust_legacy_estate_guide.md`](zero_trust_legacy_estate_guide.md), [`zero_trust_mainframe_guide.md`](zero_trust_mainframe_guide.md) and [`zero_trust_network_architecture_guide.md`](zero_trust_network_architecture_guide.md); the machine-identity pattern in agent platforms generally is [`ai_llm/ai_agent_platform_selection_guide.md`](ai_llm/ai_agent_platform_selection_guide.md); the governance frame is [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md).

---

## 7. The Evidence Question: Records, Approvals, and Segregation of Duties

### 7.1 The record is evidence

The reason the controls argument has teeth in a bank is that these records are not merely work items. An **incident record and its timeline** are what a reviewer reads to understand what happened, when it was known, and what was done. A **change record and its approval trail** are what is read to establish that a change was authorised by the right person before it was made. **Audit and regulatory review consume these records as the operative account of events** — the regulatory frame, not asserted here as any jurisdiction's rule, is cross-referenced in [`../banking/grc_in_banking_guide.md`](../banking/grc_in_banking_guide.md), [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md) and [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md).

**A platform an agent can write to is a platform whose evidence has a new author.** At the limit, a service account writing records — and in particular the incident *timeline*, where resolution notes and closure justifications live (§3.2's incident summarisation lives here in the assistive case) — means the narrative of what happened is machine-authored, and if no agent version, no trigger, no tool invocation and no reasoning trace is retained alongside the record, then the record is the *only* account, and it is unanswered.

### 7.2 The record families in scope

- **Change.** The sharpest ITSM case: a change record asserts that an alteration was planned, assessed and approved. An agent that drafts one is harmless; an agent that can *approve* one, or that can *transition* a change's state, is authoring the artefact of an authorisation.
- **Incident and its timeline.** The timeline is the evidential spine — who knew what, when, and did what. Auto-generated entries are fine *if* they are distinguishable from human ones and if the agent is named; otherwise the timeline's meaning is diluted.
- **The approval trail.** Approval records are the artefact that says "a responsible person agreed." Anything that can create an approval, or that can be *both* the requester and the approver, voids the artefact.
- **Knowledge article.** The subtle one. Knowledge articles are drafted content that other people and other agents retrieve and act on (§10.1). An article edited by an agent becomes an operational instruction with an unmarked provenance — and, via the `search_retrieval` RAG tool (§4.4) and Knowledge Graph, it can be retrieved and acted on by other agents, not only by humans.

*(Record families: ServiceNow docs, CMDB and ITSM material, Brazil, checked 1 Oct 2026 ✅; the retrieval consequence is this guide's architectural inference, labelled as such.)*

### 7.3 Draft versus commit — the real line

> The difference that matters is not "does an agent touch a record" but **"does an agent commit, or does a human commit what the agent drafted?"**

The platform documents the controls that let you sit on the right side of that line:

- **Supervised vs Autonomous** skill execution mode — Supervised means **human input is required during execution**, Autonomous means none (§3.4) ✅.
- **copilot vs autopilot** agent execution mode — `copilot` is the SDK's **preferred** mode **"when the agent creates, updates, or deletes records"**; `autopilot` is for **"read-only or fully autonomous workflows"** (§4.3) ✅.
- The **`state`** machine on a version `'draft' | 'committed' | 'published' | 'withdrawn'` (§4.3) ✅.

A drafting capability that runs `Supervised`/`copilot` and produces a proposal a human commits is a *different control object* from one that runs `Autonomous`/`autopilot` and writes the record. It is also, in the vendor's own documentation, the recommended way to configure the writing case — `copilot` **is** the documented preference for record-mutating agents. The bank's job is to keep that preference an enforced standard rather than a suggestion, and to be able to show it per agent.

### 7.4 Segregation of duties does not survive an agent that can both raise and approve

This is the crispest control statement in the guide, and it needs no vendor claim to support it — it is a property of the design:

- Segregation of duties works because **one person cannot hold both halves of a duty**. Enforce it with two humans, two logins, two credentials.
- An agent — or a set of agents sharing one identity — can trivially hold both halves. It can raise the change and approve the change; it can open the incident and close it; it can request and grant.
- The platform will not stop this by itself. The **AI Agent Orchestrator's documented "policy and governance enforcement"** is orchestrator-scoped coordination (§4.2). The enforceable artefacts are the **ACL and the role assignment** (§6.1): if the agent's identity does not hold the approver role, the separation holds.
- Therefore the control is **role-design**: the identity that raises must not be the identity that approves, and the identity that executes must not be the identity that logs the completion. This is expressible entirely in the platform's own role model — it just has to be *specified*.

### 7.5 What AI Guardian does and does not solve

**AI Guardian** is documented as **"a real-time runtime protection layer for generative AI deployments within ServiceNow"** which **"evaluates both user requests and AI responses to detect and manage offensive content, prompt injection attacks, and sensitive topics."** *(ServiceNow docs, AI Guardian, release Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)* Its documented features:

- **Offensiveness detection** — logs by default; configurable to block and return a standard error message; applies to specific generative AI skills and agentic workflows;
- **Prompt injection detection** — **"applies universally across all generative AI applications within your instance"**, configurable per instance or per skill, applying the stricter setting;
- **Sensitive topic filters** — redirects to a fallback Virtual Agent topic, available with HRSD and CSM for Virtual Agent conversational skills;
- **Logging** — detections logged automatically with request details, conversation context and user feedback; logs visible in **AI Admin Hub**; the docs advise using them to **"help establish a baseline of issues before enabling blocking"**; and **PII is anonymized before reaching the AI model, with configurable settings for data anonymization**;
- **AI Guardian analytics**, **Export AI Guardian logs**, **Create a custom guardian**, and **Enable AI Guardian for AI agents**.

**The honest boundary — and it is one of this guide's most useful contributions:** AI Guardian guards **content** — toxicity, prompt injection, sensitive topics, PII handling. It does **not** by itself solve the **identity** problem (§6), the **segregation-of-duties** problem (§7.4), or the **evidence-integrity** problem (§7.1). A jailbreak-resistant agent running under a shared account still writes unattributable records; an agent that passes every content check can still be the only approver of its own change. AI Guardian is rung 6 of the ladder (§9), not the whole ladder.

### 7.6 The two-layer governance model, and what it means for review

The documentation places AI Guardian inside a **two-layer governance model**:

- **Lifecycle governance** — **AI Control Tower** plus **AI Risk and Compliance**: pre-deployment assessment and ongoing monitoring;
- **Runtime governance** — **AI Guardian**.

*(ServiceNow docs, AI Guardian, Brazil, 10 Sep 2026 ✅)* The split is the useful part: the platform's own model expects you to assess *before* deployment and monitor *during* operation, and it provides distinct surfaces for each. §12 turns that into an operating model. Nothing about any jurisdiction's requirements is asserted here; the procedural frame is the vendor's, and the regulatory cross-reference is the repo's banking and governance guides.

> Cross-reference: the regulatory and GRC frame is [`../banking/grc_in_banking_guide.md`](../banking/grc_in_banking_guide.md), [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md), [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md), [`ai_llm/ai_governance_bias_redteaming_guide.md`](ai_llm/ai_governance_bias_redteaming_guide.md) and [`audit_as_code_guide.md`](audit_as_code_guide.md); the ITIL process design that defines the duties being separated is [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md).

---

## 8. The CMDB as the Sharpest Case: Writing What the Estate Believes It Owns

### 8.1 Why the CMDB is different from a ticket

A ticket records an **event**; a CI records a **state of the world**. That is the whole distinction. A ticket that is wrong is a wrong work item, correctable and bounded. A CI that is wrong is a wrong *belief about the estate*, and it propagates: the estate's own statement of what it owns, how it connects, and what depends on what becomes false, and every consumer of that statement inherits the falsity silently.

### 8.2 What the CMDB is, in the vendor's own words

From §2.2, repeated here because this section rests on it: **"A CI is a representation of some type of infrastructure, application, software, hardware. Those configuration items belong to classes."** Documented example classes: **Windows Server, Data Center, Internet Cable, Apache Application, and Virtual Machine.** *(ServiceNow docs, CMDB CI creator AI agent, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

### 8.3 The documented consumers — stated as uses, not as vendor features

CMDB data is consumed for **impact analysis** (what breaks if this fails), **change risk assessment** (how risky is this alteration), **dependency mapping** (what talks to what), and **licence and asset positions** (what the estate holds and is entitled to). These are the *uses of CMDB data* — the reasons an accurate CMDB matters — and they are drawn from how CMDBs are used in the discipline, not from any ServiceNow feature claim. The point is not what ServiceNow ships; the point is that **every consumer of these uses is downstream of the CI records' truth**, and a write path into the CMDB is therefore a write path into all of them.

### 8.4 The vendor ships a CMDB-writing agent, out of the box

This is not hypothetical. Documented: the **CMDB CI creator AI agent** **"Handles the creation of a configuration item (CI)";** its tools are **`Create new record for CI class`** and **`Get similar CI Classes`**; and it is used in the agentic workflow **"Create configuration item."** Its documented data-access role is **`sn_cmdb_editor`** (§6.4). *(ServiceNow docs, CMDB CI creator AI agent, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

Sibling documented CMDB AI agents: **CMDB data certification and attestation manager**, **CMDB data ownership manager**, **CMDB health metrics manager**, **CMDB life cycle manager**, **CMDB principal class manager**, and **CMDB search**. Documented CMDB skills: **Configuration item (CI) summarization**, **Manage duplicate CIs**, **Service Graph Connector diagnosis**. *(All: ServiceNow docs, Brazil, updated 10 Sep 2026, checked 1 Oct 2026 ✅)*

Read that list as the vendor's own capability statement: **the platform ships agents that create CIs, manage duplicates, own data ownership and certification, and assess CMDB health — with a documented data-access role (`sn_cmdb_editor`) that reaches them.** That is the factual basis for everything below. It is stated as architecture and as an approval-standard implication; it is **not** a criticism of the product. The product doing its documented job is exactly why the governance question is live.

### 8.5 Why write access there deserves a different approval standard

Three reasons, all structural:

1. **The failure mode is silent and long-lived.** A wrongly-created CI does not raise an alarm; it joins the estate's belief system and is consumed by impact analysis and change risk assessment until someone notices. The detection lag is the harm.
2. **The blast radius spans systems.** A CI record is not one team's work item. Its consumers include change management, incident impact analysis, and asset/licence positions — so an error propagates across functions that never see the write happen.
3. **The write is close to the estate's model of itself.** The CI is the estate's statement of what it owns. An agent that can write it can change **what the estate believes it owns** — and, by extension, what any downstream control that trusts the CMDB will conclude. That is a categorically different claim on the system than "an agent updated a ticket."

**The implication for the approval standard:** CMDB write access should sit a tier above ticket write access in any approval taxonomy — a **different approver, a different evidence requirement, and a different rollback/sufficiency story** from closing an incident. In the guardrail ladder (§9) this is rung 4 (human approval at the write boundary) applied with a *narrower* write surface (rung 2) and a *separately owned* identity (rung 3) — because the CI-write agent's `sn_cmdb_editor` data-access role is, by design, exactly the authority the approval standard must bound.

### 8.6 The two reading rules for §8

- **Do not read this as an argument against the CMDB agents.** Automation of CI creation, de-duplication and health is the documented purpose of these components, and a CMDB that is maintained by nothing is worse. The argument is that the *write* is a control decision with a higher bar, not that the write is improper.
- **Do not assume the read side is free either.** A CMDB-reading agent — `CMDB search`, `Configuration item (CI) summarization` — exposes the estate's topology to whatever client or identity invokes it, and the same §5.3 tiering argument applies: read/query is a data-disclosure object. Reading the estate's shape is not the same exposure as reading a ticket, because the shape is a map of the target.

---

## 9. The Guardrail Ladder: Seven Rungs, What Each Buys and Costs

The ladder is ordered so each rung is cheaper than the one above it and no rung is skippable. Every rung names its cost, because an uncosted guardrail is a guardrail that gets removed at the first delivery pressure.

### Rung 1 — Read before write

**Buys:** the ability to *measure* what the agent would do before it can do it; a ground truth for whether a write capability is even needed; the discovery that many "agents" are read-and-propose jobs.
**Costs:** honest instrumentation work. You must log read behaviour and review it. There is no vendor switch that proves you did this — the **Activity** execution logs (§4.1) are the raw material, and the review discipline is the bank's own.
**Vendor mechanism:** *none that enforces it.* Activity logs make it observable; the sequencing discipline is internal.

### Rung 2 — Least-privilege scoping of the tool surface

**Buys:** a capability that can only do its job. Tools are **typed and configured** — the tool catalogue is finite and enumerable (§4.4), a skill must be **published and activated** before it can be added, and each skill must carry an **ACL and role restriction** (§3.5).
**Costs:** design time per agent, and maintenance whenever the task changes. The temptation is to add the whole tool catalogue "to be safe," which inverts the control.
**Vendor mechanism:** ✅ documented — configured MCP tools (§5.2), skill ACLs and role restrictions (§3.5), per-agent **Allowed user roles** vs **Data access roles** (§6.4), and the documented prohibition on the `maint` role (§6.6).

### Rung 3 — A named identity per agent, not one shared account

**Buys:** attribution — the record shows *which* agent, and revocation stops being all-or-nothing; plus least privilege computed per agent rather than per union.
**Costs:** per-agent identity administration, and the discipline to keep one identity per purpose. It is a small, one-time build cost, and it is the rung most often skipped for exactly that reason.
**Vendor mechanism:** ✅ documented — **AI User** (`sys_user` with `identity_type = ai_agent`) as **`runAsUser`**; **Dynamic User** with `dataAccess.roleMap` / `roleList`; the mandatory **trigger run-as**; per-agent `securityAcl`; impersonation settings (§6).
**The rule:** name the identity, or you have not deployed a controlled agent (§6.7).

### Rung 4 — Human approval at the write boundary

**Buys:** the evidence-integrity property of §7.3 — a human **commits** what the agent drafted, so the record's author is a person and the segregation of duties of §7.4 is expressible.
**Costs:** latency, and the human's time. This is the rung that gets argued away first, so it needs a written gate: which record families require a human commit, and which agents may write without one (and why).
**Vendor mechanism:** ✅ documented — **skill execution mode Supervised** (human input required during execution); **agent execution mode `copilot`** (the SDK's preferred mode when an agent **"creates, updates, or deletes records"**), with `autopilot` documented for **"read-only or fully autonomous workflows"** (§3.4, §4.3). The bank's part is making `copilot`/`Supervised` the enforced default for any write surface.

### Rung 5 — Rate and blast-radius limits

**Buys:** a bound on how much damage a misconfigured or misbehaving agent can do before a human notices — the difference between one wrong CI and a hundred.
**Costs:** it is largely the bank's own control. The platform documents triggers and versioning but does not, in the sources retrieved, document a native per-agent action-rate cap; the practical pattern is to bound the *tool surface* and the *trigger conditions* (each trigger needs a **mandatory condition**, §4.3) and to add an external action counter or an approval gate for volume. Where no vendor mechanism exists, say so and own it.
**Vendor mechanism:** ⚠ partial — mandatory trigger conditions and version control are documented; a native rate limiter is **not** evidenced in this pass.

### Rung 6 — Logging that preserves the agent's reasoning trace, alongside the record

**Buys:** the ability to answer "what did the agent believe, and what did it call, when it wrote this?" — without which the record is the only account and is unattributable.
**Costs:** storage, retention policy, and the operational work of reviewing logs; plus the integration of three separate log surfaces into one reviewable story.
**Vendor mechanisms:** ✅ documented, and all of them must be **configured** — **Activity** execution logs for agents and agentic workflows, **filterable by version**; **AI Guardian** detection logs in **AI Admin Hub**, with export; per-agent-name approval attribution — the documentation states that **"Administrators can see logs with individual AI agent names as a record of who approved the agentic action in an agentic workflow."** Be honest about the last one's exact claim: it attributes the *approval of an agentic action* to the AI agent's name; it is a vendor feature that supports attribution and it is **not** a substitute for deciding, at design time, who may approve (§7.4).

### Rung 7 — An explicit named owner for each agent capability

**Buys:** the only guardrail that survives staff turnover. An owner is the person who answers "should this still exist, with this tool surface, under this identity?" — and who is accountable when the answer is wrong.
**Costs:** an operating-model commitment, reviewed periodically. Without it, capability accumulates: agents stay published after their purpose ends, because nobody's job says otherwise.
**Vendor mechanism:** ⚠ partial — the **Overview** journey checklist, **Guided Setup** and **AI Agent Analytics** give visibility, and **AI Control Tower**/AI Risk and Compliance give lifecycle surfaces (§12); but ownership is an internal RACI, and the platform will not enforce it.

### 9.8 Reading the ladder as a whole

The rungs are ordered by *cost to reverse*: rung 1 is free to change, rung 7 is an organisational commitment. The failure pattern is always the same shape — rungs are removed from the top (the cheap ones) and the expensive ones are deferred, so the short-term discipline (read first, scope the tools) is what disappears first, exactly when it is doing the most work. Rungs 3 and 4 are the load-bearing pair for this guide's thesis, because together they are what make "the agent wrote that record" a statement with a named subject and a human commit.

> Cross-reference: the agent-governance and evaluation discipline is [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md) and [`ai_llm/llm_evaluation_vs_validation_guide.md`](ai_llm/llm_evaluation_vs_validation_guide.md); the failure catalogue for agentic reasoning is [`ai_llm/llm_agents_failures_production_guide.md`](ai_llm/llm_agents_failures_production_guide.md); the sandboxing frame for tool execution is [`ai_llm/agent_sandboxing_strategies_guide.md`](ai_llm/agent_sandboxing_strategies_guide.md); the shared-credential reasoning is [`zero_trust_legacy_estate_guide.md`](zero_trust_legacy_estate_guide.md).

## 10. Use Cases That Hold Up and Those That Do Not

Each case below names the control it requires. **No productivity claim, time-saving figure, adoption statistic, cost figure, or benchmark number appears anywhere in this section or this guide** — the judgement is structural (what the capability touches and who commits), not economic.

### 10.1 Cases that hold up

| Case | What it does | Documented mechanism it rests on | The control it still requires |
|---|---|---|---|
| **Read-only summarisation and triage support** | Summarises a record or alert; suggests a category for a human to accept | Otto generative AI skills — e.g. **Configuration item (CI) summarization**, **Alert analysis**, **Alert investigation**, **Analyze service health** (§3.2) | Named identity for the agent (§6); a read/query-tiered tool surface (§5.3); the summarisation itself is assistive and owned by [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2 |
| **Retrieval over knowledge and history** | Answers "has this happened before, and what did we do?" from knowledge and past records | The **`search_retrieval`** RAG tool type (§4.4); **Knowledge Graph**; the **`knowledge_graph`** tool type; MCP Server Console **Knowledge Graph schema support** (§5.2) | Article-level access scoping; the retrieval corpus is itself an attack surface (prompt injection via retrieved content, §7.5) |
| **Drafting that a human commits** | Produces a proposed change description, resolution note, or knowledge article revision for a person to accept | **Supervised** skill execution mode and **`copilot`** agent execution mode (§3.4, §4.3) | The write boundary must be real — the human must actually review, and the committed author must be the human (§7.3) |

These hold up for one shared reason: **the human remains the author of the decision.** The agent reduces the cost of *producing the draft*; the record's evidential meaning is unchanged because a person still committed. The retrieval case holds up because it does not mutate anything — but it is not free, because a retrieval surface is a data-disclosure surface and a prompt-injection ingress.

### 10.2 Cases that do not hold up without controls

| Case | Why it fails as designed | What would have to change |
|---|---|---|
| **Auto-closing records** | Closing is a *commitment* that the incident is resolved — an evidential claim, and a claim about the estate that a retrieval or metric will later consume. An agent that closes under a shared account makes the closure unattributable (§6.5, §7.1). | A human commit on closure (Supervised/`copilot`), a named agent identity for any pre-work, and a distinguishable "agent-assisted" marker on the timeline |
| **Auto-approving changes** | An approval is the artefact that a responsible person agreed. An agent approving its own or another agent's change voids the artefact and dissolves the separation (§7.4). | Do not do it. If a standard-change class is genuinely pre-authorised, that authorisation belongs in the *change model*, granted by a human, not in an agent's runtime decision — and the agent's identity must not hold the approver role |
| **Anything that both decides and executes** | The decision and the action must not sit in one uncontrolled actor; that is how a misjudgement becomes an irreversible state change with no intervening human. | Split the capability: one component decides and proposes, a different identity (or a human) executes. Enforce with roles, not with intent |
| **An agent holding both raise and approve rights** | The purest form of the segregation failure: one login on both sides of the duty (§7.4). | Re-scope the agent's role set so it cannot hold the approver role; verify with AI Agent Studio **access tests** (§4.1, §12.3) |
| **CMDB write without a separate approval standard** | The CI is a state-of-the-world claim consumed by impact analysis and change risk assessment, with a silent propagation path (§8). | CMDB write access sits a tier above ticket write access; separate approver, separate evidence expectation (§8.5) |

### 10.3 The test to apply to any new proposal

Four questions, in order. If any answer is missing, the case is not yet a controlled capability:

1. **What does it touch?** Record family (§2.2) and read-versus-write tier (§5.3).
2. **Who does it run as?** The named identity (§6.7) — and if the answer is a shared account, stop.
3. **Who commits?** The human at the boundary (§7.3), or the written justification for why a commit is machine-made.
4. **Who owns it?** The named owner (rung 7, §9) and the review surface it will be reviewed at (§12).

The cases that fail do so at question 2 or 3, without exception. And the failure is rarely declared as a failure — it is declared as "we'll add the identity later," which is question 2 being deferred past the point where it is cheap.

> Cross-reference: the assistive summarisation/triage analysis is [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2; the retrieval-versus-agentic-search distinction is [`ai_llm/agentic_search_vs_rag_guide.md`](ai_llm/agentic_search_vs_rag_guide.md); the guarded-autonomy pattern for action-taking agents is [`ai_llm/autonomous_agents_guide.md`](ai_llm/autonomous_agents_guide.md) and [`agentic_workflows_guide.md`](agentic_workflows_guide.md).

---

## 11. Integration Architecture: Flows, Fabric, Zero Copy, and the MCP Boundary

### 11.1 Three routes into and out of the platform

An estate has three materially different ways to connect ServiceNow to everything else, and they suit different jobs:

| Route | What it is | Suits | The operational questions it raises |
|---|---|---|---|
| **Native automation and flow** | **Automation Engine** and **Flow Designer**, plus spoke-class connectors as documented generally in the integration surfaces | Deterministic, pre-designed integrations where the sequence is known and does not need a model to decide | Versioning of flows; ownership of the flow; failure handling in the flow runtime |
| **Data-layer integration** | **Workflow Data Fabric**, **Zero Copy** connectors, **Knowledge Graph** | Making the estate's data available *to* workflows and agents **without moving or duplicating it** | Data-residency and access scope; which sources are connected; who governs the graph |
| **Agent/MCP integration** | **MCP Server Console** (inbound), **MCP Client** (outbound), MCP tools on agents (§4.4) | Cases where an *agent* — not a pre-designed flow — must decide which capability to call at runtime | Authentication model; tool-surface ownership; versioning of a *configuration*, not just code; failure handling when the agent's plan is wrong |

### 11.2 The data fabric, dated and described

**Workflow Data Fabric** is, in the vendor's own words, **"an enhanced integrated data layer that unifies business and technology data across the enterprise, powering all workflows and AI agents with real-time, secure access to data from any source."** It is described as powered by the **Automation Engine** and by the newly introduced high-performance database **RaptorDB Pro**; its **Zero Copy** connectors connect to data sources **"without moving or duplicating data"**, with Zero Copy partnerships with **Databricks** and **Snowflake** named; and it is stated to include **ServiceNow Knowledge Graph**. Availability stated in the release: **"Workflow Data Fabric is available today in controlled go-to-market."** *(ServiceNow press release, 23 October 2024, newsroom.servicenow.com ✅)*

**Date it, and read it correctly.** That press release is from **23 October 2024**; the documentation tree retrieved at Brazil/2026 also carries a **Workflow Data Fabric AI agents** family (flow generation, product knowledge, product recommendation, spoke asset discovery, spoke install advisor), **Workflow Data Fabric integration with Knowledge Graph**, and a **Zero Copy Connector for ERP AI agents**. *(ServiceNow docs, Brazil, checked 1 Oct 2026 ✅)* The phrase **"controlled go-to-market"** is a *dated availability statement from 2024*, not a current status and not a roadmap — quote it as the release said it, and do not extrapolate forward. The release also carried performance figures; **none of them appear in this guide** (see the presentation rules in §1.4 and the audit in §15). And the database name **RaptorDB Pro** is verified **only** from that press release — it returned zero hits in the documentation TOC retrieved this pass (§16.1).

### 11.3 Where the boundary sits

The MCP route is not a replacement for the flow route, and treating it as one is a category error:

- **If the sequence is known**, use the flow/automation route. It is deterministic, testable, and does not ask a model to decide a decision that a design already made. Paying for a model to re-derive a fixed sequence is cost without benefit.
- **If the sequence is not known** — the capability must decide *which* action applies to *this* record — then an agent and an MCP-exposed tool surface is the appropriate route, and the controls of §6–§9 apply in full.
- **If the problem is data reach** — the agent or flow needs values it cannot get locally — the data-layer route (Workflow Data Fabric, Zero Copy, Knowledge Graph) is the answer, and it is a *governance* question (what data, from where, under whose authority) as much as an engineering one. Cross-reference [`data_governance_guide.md`](data_governance_guide.md).

The boundary, stated as a rule: **MCP is for runtime-decided capability invocation; flows are for design-time-decided sequencing; the data fabric is for reach.** A proposal that uses MCP to do what a flow already does is a proposal with an extra control surface and no extra capability — which is a bad trade in a bank.

### 11.4 The four operational questions every route raises

1. **Versions.** An agent carries `versionDetails[]` with `draft | committed | published | withdrawn` (§4.3); an MCP server's *tool surface* changes with configuration (§5.7); a flow has its own version history. Three different versioning models on one platform, which means an inventory must record **which version of which artifact is live** for each integration. The **Activity** log's filter-by-version (§4.1) is the vendor's mechanism for making this visible.
2. **Authentication.** Inbound MCP offers **OAuth 2.1** and **API key**, plus **third-party IdP** integration (§5.2). Agents have **AI User** vs **Dynamic User** plus per-agent roles (§6). A flow runs as its configured identity. Four identity mechanisms; the §6 rule — name the identity — applies to all of them.
3. **Failure handling.** What happens when the tool call fails, the agent's plan is wrong, or the external MCP server is unavailable? The platform provides logging (`Activity`, AI Guardian) but the *recovery* design — idempotency, retry bounds, dead-letter, human escalation — is the design team's. The failure-mode catalogue is [`ai_llm/llm_agents_failures_production_guide.md`](ai_llm/llm_agents_failures_production_guide.md).
4. **Who owns the tool surface?** An MCP server's tools are **configured** (§5.2) — so someone must own that configuration, and that owner must be a named role, not "the integration team." This is rung 7 of the ladder applied to the surface rather than to the agent.

### 11.5 A structural note on the fabric's governance

The data-fabric route changes *where the estate's data can be reached from*, which is exactly why the vendor runs MCP intake through the **AI Steward** and a lifecycle (§5.5) and why the governance frame points to [`data_governance_guide.md`](data_governance_guide.md) and [`api_governance_guide.md`](api_governance_guide.md). A Zero Copy connector that reaches a data platform without copying data is still a **reach**, and the control question — who may reach it, through which identity, for what purpose — is unchanged by the elegance of the mechanism. No performance claim is made or implied here; the architectural fact is only that the data stays where it is and is accessed rather than duplicated.

> Cross-reference: the flow/workflow-engine discipline is [`agentic_workflows_guide.md`](agentic_workflows_guide.md), [`durable_ai_agent_workflows_guide.md`](durable_ai_agent_workflows_guide.md) and [`temporal_workflow_guide.md`](temporal_workflow_guide.md); integration frameworks generally are [`data_integration_frameworks_guide.md`](data_integration_frameworks_guide.md); the observability bridge that feeds ITOM/AIOps Event Management is [`opentext_operations_bridge_guide.md`](opentext_operations_bridge_guide.md).

---

## 12. The Operating Model: Ownership, Review, and Retirement

### 12.1 The finding, stated first

> **An agent is a standing integration with a lifecycle, not a feature you switch on.**

Three documented facts force this conclusion. (i) Some skills, AI agents and agentic workflows are **ON BY DEFAULT** (§3.3) — so a capability's existence is not proof a project deployed it. (ii) An agent's real "version" is a *configuration state* — tool surface, identity, triggers, roles — that can drift from any documented baseline (§5.7, §11.4). (iii) The platform provides a full lifecycle surface — intake, approval, activity-by-version, testing, withdrawal — which only means something if a human operates it (§4.1, §5.5). A capability with no owner has none of the three addressed, and will therefore drift.

### 12.2 The ownership model

For every agent, agentic workflow, skill and MCP server:

| Element | Who owns it | Why |
|---|---|---|
| The **capability** (does it exist, should it?) | A named business/process owner | Rung 7 (§9); nothing else survives turnover |
| The **tool surface** | A named platform owner | Configured artifacts (§5.2, §4.4) drift silently |
| The **identity** | The platform/identity owner, jointly with the capability owner | Role sets and run-as are the enforceable control (§6) |
| The **approval** (MCP intake, and the write boundary) | The **AI Steward** role for MCP (`sn_ai_governance.ai_steward`); the process approver for record writes | The vendor's own model names the Steward (§5.5); the record-write approval is the bank's |
| The **evidence** (logs, traces) | The audit/operations function that reviews them | Logs that are never reviewed are storage cost, not control |

### 12.3 How it is reviewed

The platform gives distinct review surfaces, and they map to the two-layer governance model (§7.6):

- **Pre-deployment / lifecycle governance** — **AI Control Tower** plus **AI Risk and Compliance**: assessment before the capability goes live, and ongoing monitoring after (§7.6 ✅).
- **MCP onboarding** — the **MCP server intake → AI Steward playbook approval → client registration via AI Gateway** flow, with the **MCP server approval playbook workflow** and the **MCP server record** as the artifacts of record (§5.5 ✅).
- **Access proof** — AI Agent Studio's **Testing** page, including **access tests to validate security controls** and automated agentic evaluations (§4.1 ✅). This is the only documented place where access is *proved* rather than assumed, which makes it the natural gate; §3.3's domain-scope behaviour is exactly the kind of thing an access test should evidence.
- **Runtime** — **AI Guardian** (offensiveness, prompt injection, sensitive topics) with its logs in **AI Admin Hub** (§7.5 ✅), plus **AI Agent Analytics** (§4.1 ✅).

The honest note: **most of these surfaces must be enabled and configured.** AI Guardian's logging helps you establish a baseline **before** enabling blocking, in the documentation's own framing (§7.5) — which means review is a *programme* with a sequence, not a switch. A bank that has not configured the logs has the platform's capability and none of the control.

### 12.4 What a capability may change without a new approval

Draw the line explicitly, or the line will be drawn by whoever is under delivery pressure:

| Change | New approval needed? | Why |
|---|---|---|
| Prompt/instruction wording inside an existing, scoped skill | Owner's discretion, with the version recorded | The tool surface and identity are unchanged (rungs 2–3 intact) |
| **Adding a tool** to an agent | **Yes** — re-scope review | The tool surface *is* the risk surface (§5.3, rung 2) |
| Changing the **run-as identity** or data-access roles | **Yes** — identity review | This is the enforceable control (§6, rung 3) |
| Moving **Supervised → Autonomous** or **copilot → autopilot** | **Yes** — controls review | It crosses the commit boundary (§7.3, rung 4) |
| Adding a new **trigger** | **Yes** — including its mandatory condition and run-as (§4.3) | A trigger is a new way for the agent to fire unattended |
| Changing an **MCP server's tool surface** | **Yes** — and re-run the AI Steward path where applicable (§5.5) | The surface is the approved artifact |
| Editing a skill with **no ACL** (grandfathered, §3.5) | **Yes** — an ACL becomes mandatory on republish | The republish forces the decision; make it deliberately |

The pattern: **anything that changes the surface, the identity, or the commit boundary needs a new approval.** Anything that changes only wording inside an unchanged surface does not — and that distinction is what makes the rule liveable instead of bureaucratic.

### 12.5 How it is retired

Retirement is a first-class operation, and the platform documents the mechanics:

- **Version state `withdrawn`** on `versionDetails[]` (§4.3 ✅) — the documented way to retire a version.
- **Deactivation** of a skill or agent (the skill must be *published and activated* to be usable, §3.4 — so deactivation removes its usability).
- **Deregistering MCP clients** and addressing the **MCP server record** (§5.5) so the intake path's artifacts reflect reality.
- **Revoking the agent's identity** (§6) — the step that actually closes the door, because the identity is what carried the authority.

Retirement is the rung-7 test made concrete: a capability whose owner has left, whose purpose has ended, and which is still **published** is an unowned standing integration. The honest position is that without rung 7, retirement does not happen — the platform will happily keep an orphaned agent running, because nothing in it knows the agent is orphaned.

### 12.6 The one-line operating rule

**Treat every agent as a product with a named owner, a versioned surface, a named identity, a documented approval path and a retirement plan — because that is what it is.** The alternative framing — "we turned on a capability" — is how an estate acquires standing integrations nobody can attribute, scope, or switch off.

> Cross-reference: the AgentOps discipline for running agents is [`ai_llm/agentops_guide.md`](ai_llm/agentops_guide.md); the lifecycle and evaluation frame is [`ai_llm/llm_evaluation_vs_validation_guide.md`](ai_llm/llm_evaluation_vs_validation_guide.md); the governance frame is [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md); the audit/evidence frame is [`audit_as_code_guide.md`](audit_as_code_guide.md).

## 13. Worked Example: A Cymbal Bank ServiceNow Estate

**Cymbal Bank is the only bank persona and the only institution anywhere in this guide. Everything in §13 is explicitly illustrative and fictional — an architecture exercise, not a case study, and not a claim about any real institution, including any real ServiceNow customer.** The three scenarios below are chosen to cover the shape of the decision: one case that holds up, one middle case, one that must not proceed as designed.

### 13.1 The illustrative estate

Cymbal Bank's illustrative ServiceNow estate carries the record families of §2.2 — incident, problem, change, request, configuration item, knowledge article, case — plus the agentic layer: **AI Agent Studio** installed with the **ServiceNow Otto AI Agents** plugin, some generative AI skills **on by default** (§3.3), and a small number of MCP servers registered inbound through the **MCP Server Console** (§5.2). In this illustrative estate, an integration team has proposed three capabilities in sequence. The point of the exercise is that all three proposals *look* like the same project — "an AI agent for ServiceNow" — and are in fact three different control decisions.

### 13.2 Case (a) — read-only summarisation and retrieval (holds up)

**The proposal.** A capability that, on request, summarises an incident and its timeline for a service-desk analyst, and retrieves related knowledge articles and past incidents. It writes nothing.

**Design.** Built on Otto generative AI skills (**Incident summarisation**, §3.2 — assistive, owned by [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2) plus a **`search_retrieval`** tool and **Knowledge Graph** reach (§4.4). Exposed inbound through an MCP server whose **tool surface is configured to read/query/summarise only** (§5.3, tier 1).

**Controls applied.** Rung 1 (read-only — moot here, since the capability is read-only by construction) and rung 2 (a tool surface that cannot write, which is a *structural* guardrail, not a policy one). An MCP server whose tools do not include a create/update tool cannot be talked into writing by a convincing prompt — the capability is absent, not discouraged. Authentication via **OAuth 2.1** with the bank's identity provider (§5.2), so the calling client is a managed identity rather than an anonymous API key. **AI Guardian** prompt-injection detection enabled, because the retrieval corpus (§10.1) is an ingress (§7.5).

**Why it holds up.** It changes no record, so it raises no evidence problem (§7.1) and no CMDB problem (§8). Its residual risk is data disclosure (§5.3) and injection via retrieved content (§7.5) — both bounded by the surface and by runtime protection, and both reviewable.

**What it still needs.** A named owner (rung 7) and a defined review point (§12.3). The case that "holds up" still has to be operated.

### 13.3 Case (b) — an agent that drafts change records (the middle case)

**The proposal.** An agent that, given an approved plan, drafts a change record — description, justification, implementation steps, risk notes — for a human to review and submit. A person is the commit point.

**Design, and the exact switches that matter.** The agent's **execution mode is `copilot`**, and the drafting skill's execution mode is **Supervised** (§3.4, §4.3). Both switches are set to the documentation's own preference for **"when the agent creates, updates, or deletes records"** — this is the vendor's documented preferred configuration for this case, not a bank-specific compromise. The agent's tool surface includes read tools and a *draft* tool; it does **not** include a state-transition tool, so it cannot move the change into an approved state even if prompted (§5.3, tier 2 as distinct from tier 3). Its **run-as identity is a named AI User**, `sys_user` with `identity_type = ai_agent`, one for this purpose (§6.2) — so the drafted record's provenance names the drafting agent, and the committed change names the human.

**Where the control lives.** At the commit boundary (§7.3). The agent produces a proposal; a human reads it and submits. If the human submits without reading, the control is a formality — which is why the review expectation must be written down, and why the record should show which fields were agent-drafted. This is the honest cost of case (b): it buys evidence integrity at the price of a human review step, and the price only pays if the review is real.

**The prohibition baked into the design.** The agent's identity holds no approver role. It can draft a change; it cannot approve one, and it cannot approve its own or anyone else's. This is enforced by role assignment (§7.4), not by instruction.

### 13.4 Case (c) — the proposal that must not proceed as designed

**The proposal.** "Let the agent close resolved incidents automatically, and let it approve standard changes."

This is one proposal hiding two failures. Work it through the three questions.

**The identity question (§6).** Suppose — as such proposals usually suppose — the agent runs as the bank's existing shared integration account, the one already used by the monitoring integration. Then every incident it closes is closed by an identity that also represents every other integration on the estate. The closure is unattributable: the record says the account did it, and the account stands for a dozen things. **The agent's capability is irrelevant.** It could be the most accurate closure engine in the world; the record it produces still cannot answer "which agent, which version, which intent." And the fix is available on the platform — a named AI User per purpose — which is why the shared account is a choice, not a constraint (§6.5). *This is the state to reject, before anything else is assessed.*

**The evidence question (§7).** "Auto-close resolved incidents" is a claim about the estate — this incident is over, the underlying issue is addressed — recorded by an author who is not a person. If the closure justification was generated by the same agent that decided to close, the record has a single machine author from decision to evidence. And "approve standard changes" is worse: an approval is the artefact that a responsible human agreed (§7.2), and an agent — especially one sharing an identity with the requester side — can trivially hold both halves of the duty (§7.4). If the same identity can raise and approve, segregation of duties does not survive, no matter how the process is drawn.

**The CMDB question (§8).** The proposal as written does not touch the CMDB, so the natural instinct is that §8 does not apply. It does, indirectly: closures feed incident-to-CI impact analysis, and **change approvals feed change risk assessment** — and change risk assessment is a documented use of CMDB data (§8.3). An agent that approves changes is injecting machine judgement into a control whose *input* is the estate's model of itself. If the proposal later grows the obvious next feature — "and let it update the CI when the change completes" — it becomes a direct CMDB-write case, and then it inherits §8's higher approval standard, with the CMDB CI creator agent's **`sn_cmdb_editor`** data-access role (§8.4) as exactly the authority that must be bounded.

**The redesign that would pass.** Strip the commit from the agent: it *proposes* closures with a distinguishable agent-authored marker and a named identity, and a human commits; it *drafts* standard-change documentation, and a human approves, with the agent's identity holding no approver role; and it does not touch the CMDB. That is case (b) applied twice. The redesign is not a compromise on the agent's usefulness — the drafting and triage work is where the agent's judgement is genuinely used — it is the removal of a machine commit from an evidential step.

### 13.5 What the three cases have in common

All three were proposed as "an AI agent for ServiceNow." Only the first is a one-line approval. The second is a design with two switches set to the vendor's documented writing-case defaults and one identity named per purpose. The third fails at the identity question before its capability is even assessed, and would fail again at the evidence question if the identity were fixed. **The capability was never the differentiator; the write scope and the identity were.**

> **Connecting an agent to the system of record is an integration project; letting it write there is a controls decision.**

> Cross-reference: the Cymbal Bank posture reused across the repo's agent guides is [`ai_llm/enterprise_agentic_platform_architecture_guide.md`](ai_llm/enterprise_agentic_platform_architecture_guide.md); the assistive-layer product analysis is [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2; the zero-trust shared-credential reasoning is [`zero_trust_legacy_estate_guide.md`](zero_trust_legacy_estate_guide.md); the GRC frame is [`../banking/grc_in_banking_guide.md`](../banking/grc_in_banking_guide.md).

---

## 14. Anti-Patterns: Symptom, Cause, Guardrail

Each row is a pattern seen in estate reviews, its cause, and the rung of §9 that addresses it.

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| 1 | Every agent's records show the same integration account | One shared account retrofitted onto agents to avoid per-agent identity administration | **Rung 3** — a named **AI User** per agent purpose (§6.2, §6.5). The platform documents the named identity; the shared account is a choice |
| 2 | The MCP tool surface mirrors the platform's API one-for-one | The server was configured by exposure rather than by task; "make everything available" | **Rung 2** — configure the tool surface to the task, and tier read/draft/write as separate surfaces (§5.3) |
| 3 | Write access was granted before read behaviour was ever measured | The capability was scoped from the use case description, not from observed need | **Rung 1** — read first; use **Activity** logs (§4.1) as the ground truth for what the agent actually needed |
| 4 | One agent can both raise and approve | Roles were assigned to make the workflow *work*, without applying the segregation rule | **Rung 4** + role design — the raising identity must not hold the approver role (§7.4); verify with **access tests** (§12.3) |
| 5 | A published agent whose business owner has left | Ownership was recorded as "the platform team" or not at all; nothing retires it | **Rung 7** — a named owner per capability, plus the documented **`withdrawn`** state and deactivation as the retirement mechanics (§12.5) |
| 6 | The only evidence of what an agent did is the record it wrote | Logging was never configured; `Activity` and AI Guardian logs were left unenabled | **Rung 6** — configure and *review* the **Activity** execution logs (filterable by version) and **AI Guardian** logs in AI Admin Hub (§9, §7.5), and preserve the reasoning trace alongside the record |
| 7 | A capability was adopted because a marketing page or a community README described it | The name was taken from a blog, a job advert or a partner page rather than the platform documentation | **The naming rule** (§1.3, §5.6) — every name traced to the vendor's own documentation and dated; unverified names recorded as unverified (§16.1) |
| 8 | An agent that was "deployed" was already running before the project started | Some skills, agents and agentic workflows are **ON BY DEFAULT** (§3.3) | **Inventory first** — enumerate what is enabled and owned before building anything new |
| 9 | An agent changed what it could do, and nobody noticed | The tool surface or identity was changed as a "small adjustment" without review | **The change-control line** (§12.4) — surface, identity and commit boundary changes need a new approval |
| 10 | A skill exists with no ACL | Grandfathered skills run without an ACL until edited (§3.5) | **Inventory and act** — the republish will force an ACL anyway; make the decision deliberately and before an edit does it by accident |
| 11 | MCP was used to call a capability a flow already exposed | The MCP route was treated as a replacement for deterministic integration | **The boundary rule** (§11.3) — MCP for runtime-decided invocation; flows for design-time-decided sequencing |
| 12 | A CMDB-writing agent was approved on the same basis as a ticket-writing agent | One approval taxonomy for all writes | **A separate standard** (§8.5) — CMDB write sits a tier above ticket write: different approver, different evidence expectation |

The patterns share a shape: the capability is fine, and the **surface, identity or commit boundary** was never treated as a control object. That is why §14 exists as a list of *symptoms* — the symptom is what a review finds; the control is what it should have found instead.

---

## 15. Claims Audit

Every product name, status and mechanism used in this guide, with its source, date and quality. `✅` = confirmed at the cited primary source this pass; `⚠` = vendor-published, single-source, or not re-verified this pass; `❌` = contradicted at a primary source. Dates are the source's own dates; "checked" is when this pass retrieved it.

| # | Claim | Source | Date | Quality / notes |
|---|---|---|---|---|
| 1 | **ServiceNow Otto** is the current name; **Now Assist** is the former name of the same product; technical identifiers unchanged | ServiceNow SDK docs, `servicenow.github.io/sdk`, v4.13.0 | checked 1 Oct 2026 | ✅ Exact quote available |
| 2 | Skills are named, workflow-specific generative-AI capabilities ("tailored to meet the needs of users in different workflows") | ServiceNow docs, *Generative AI skills* | Brazil, updated 10 Sep 2026; checked 1 Oct 2026 | ✅ |
| 3 | Documented CMDB skills: **Configuration item (CI) summarization**, **Manage duplicate CIs**, **Service Graph Connector diagnosis** | ServiceNow docs, *Generative AI skills* | Brazil, 10 Sep 2026 | ✅ |
| 4 | Documented ITOM skills: **Alert analysis**, **Alert investigation**, **Analyze service health** | ServiceNow docs, *Generative AI skills* | Brazil, 10 Sep 2026 | ✅ |
| 5 | Skills are global-domain by default; in domain-separated environments a skill uses only the user's domain; data resides on the instance; shared generative-AI services do not persist prompts/responses; some skills/agents/workflows **on by default** | ServiceNow docs, *Generative AI skills* | Brazil, 10 Sep 2026 | ✅ Vendor's own statements |
| 6 | A skill can be added as an agent tool ("Add tool → Generative AI skill"); Description is sent to the LLM; Inputs filled at runtime unless overridden; **Execution mode Supervised \| Autonomous**; skill must be published and activated; run-as user must pass the skill ACL; role `sn_aia.admin` | ServiceNow docs, *Add a generative AI skill to an AI agent* | Brazil, 10 Sep 2026 | ✅ |
| 7 | Skill security: ACL and role restrictions mandatory; options Any authenticated user / Select roles; Allow-If semantics; no-ACL skills keep running but an ACL becomes mandatory on edit+republish | ServiceNow docs, *Configure security controls for a skill* | Brazil, 10 Sep 2026 | ✅ |
| 8 | **AI Skill Kit**; four-step workflow (provider → prompt → test → deploy); deploy targets Otto panel, Otto context menu, Virtual Agent, Flow Action, UI Action; AI developer role `sn_skill_builder.admin`; activation by Otto admin with `admin` | ServiceNow docs, *Exploring AI Skill Kit* (tree folder `now-assist-skill-kit`) | Brazil, 10 Sep 2026 | ✅ role string ⚠ (one page renders `snskillbuilder.admin`) |
| 9 | **AI Agent Studio** creates/manages/tests agents and agentic workflows; requires the **ServiceNow Otto AI Agents** plugin; pages Overview / Create and manage / Activity / Testing / Settings / AI Agent Analytics dashboard | ServiceNow docs, *AI Agent Studio overview*; *Install ServiceNow Otto AI Agents* | Brazil, 10 Sep 2026 | ✅ |
| 10 | **Activity** execution logs filterable by version; **Testing** includes access tests and automated evaluations; **Settings** enable **AI Guardian**; documented **Version control for AI agents and agentic workflows** | ServiceNow docs, *AI Agent Studio overview* | Brazil, 10 Sep 2026 | ✅ |
| 11 | **AI Agent Orchestrator** centrally manages multiple agents; roles: task coordination, dynamic decision-making, seamless enterprise integration ("Works with ServiceNow workflows, CMDB, and third-party APIs"), multi-step handling, policy & governance enforcement | ServiceNow docs, *Understand AI agents* | Brazil, 10 Sep 2026 | ✅ |
| 12 | **ReAct prompt** + **Route** scheduling mode (asynchronous parallel tool execution); **Dynamic Orchestrator** at "more than the ideal 8 to 10 AI agents"; **User Impersonation**; **Virtual Agent** integration | ServiceNow docs, *Understand AI agents* | Brazil, 10 Sep 2026 | ✅ |
| 13 | Impersonation: agentic workflow executes tools as the logged-in user in the Otto panel; testing uses instance-level impersonation | ServiceNow docs, *Understand AI agents* | Brazil, 10 Sep 2026 | ✅ |
| 14 | **AI User** = `sys_user` with `identity_type=ai_agent`, selected as `runAsUser`/`runAs`; **Dynamic User** inherits roles via `dataAccess.roleMap`/`roleList`; data-access controls required when `runAsUser` unset; builder asked to choose | ServiceNow SDK docs, *Building AI Agents Guide* | checked 1 Oct 2026 | ✅ Central to §6 |
| 15 | **`securityAcl`** options `'Any authenticated user'`/`'Specific role'`/`'Public'` | ServiceNow SDK docs, `AiAgent` API | checked 1 Oct 2026 | ✅ |
| 16 | Per-agent **"Allowed user roles"** (`sn_cmdb_editor, snc_internal`) vs **"Data access roles"** (`sn_cmdb_editor`); **External discoverable** / **Specialist enabled** flags on `sn_aia_agent_config`, both default off; `sn_aia.ltm.enable_long_term_memory` | ServiceNow docs, *CMDB CI creator AI agent* | Brazil, 10 Sep 2026 | ✅ |
| 17 | Agents/versions logs show individual AI agent names "as a record of who approved the agentic action in an agentic workflow" | ServiceNow docs, AI agent logging material | Brazil, 10 Sep 2026 | ✅ Exact scope of the claim is narrowly approval-of-action; see §9 rung 6 |
| 18 | `maint` role NOT allowed for agents; `security_admin` allowed with a broad-privilege warning; ACL deployment patterns constrained (`$id`; `roles` array) | ServiceNow SDK docs | checked 1 Oct 2026 | ✅ |
| 19 | `AiAgent` fields: `name`, `description`, `agentRole`, `agentType`, `recordType`, `channel`, `versionDetails[]` with `state ∈ draft/committed/published/withdrawn`, `triggerConfig[]` (mandatory condition and run-as), `memoryCategories`, `parent`, `protectionPolicy` | ServiceNow SDK docs, `AiAgent` API | checked 1 Oct 2026 | ✅ |
| 20 | Agent `executionMode` `"copilot"` preferred "when the agent creates, updates, or deletes records"; `"autopilot"` for "read-only or fully autonomous workflows" | ServiceNow SDK docs, *Building AI Agents Guide* | checked 1 Oct 2026 | ✅ Load-bearing for §7.3 |
| 21 | Tool types incl. `crud`, `script`, `capability`, `subflow`, `action`, `catalog`, `topic`/`topic_block`, `search_retrieval`, `web_automation`, `knowledge_graph`, `file_upload`, `deep_research`, `desktop_automation`, **`mcp`** | ServiceNow SDK docs, *Building AI Agents Guide* | checked 1 Oct 2026 | ✅ |
| 22 | Table names `sn_aia_agent`, `sn_aia_agent_config`, `sn_aia_version`, `sn_aia_tool`, `sn_aia_agent_tool_m2m`, `sn_aia_usecase`, `sn_aia_trigger_configuration`, `sn_aia_external_agent_configuration`; `agentDescriptor` values incl. `created_by_ai_agent_advisor`, `created_by_build_agent` | ServiceNow SDK docs | checked 1 Oct 2026 | ✅ enum value ✅; "Build Agent" as a product ⚠ (no TOC match — §16.1) |
| 23 | AI Agentic Workflows orchestrate "multiple agents as a team"; documented **Deployment Order** | ServiceNow SDK docs | checked 1 Oct 2026 | ✅ |
| 24 | **MCP Server Console** enables secure/governed access to instance functionality for MCP clients; extends ServiceNow AI Platform® functionality over MCP; features: MCP integration, configuration/management, connectivity, resources; structure Explore/Configure/Connect/Reference | ServiceNow docs, MCP Server Console landing | Brazil, 10 Sep 2026 | ✅ |
| 25 | MCP server **tools are configured**; server governed by ACLs; role `sn_mcp_client.admin`; ACL options Any authenticated user / Users with specified roles (default) / **Public** ("Grants access to all users, including guests who aren't signed in" — "should be used sparingly") | ServiceNow docs, *Define security controls for MCP Servers* | Brazil, 10 Sep 2026 | ✅ |
| 26 | Auth: **OAuth 2.1** and **API key** options; **third-party identity providers** integration; **Create an OAuth inbound integration for an MCP client**; integrations incl. AI skill support, Knowledge Graph schema support, domain separation | ServiceNow docs, MCP Server Console | Brazil, 10 Sep 2026 | ✅ |
| 27 | ServiceNow Store **Model Context Protocol Server listing**; **MCP Server Console FAQ**; Community article "Designing your first custom ServiceNow MCP server" | ServiceNow docs (linked helpful resources) | Brazil, 10 Sep 2026 | ✅ (Community article ⚠ as community content) |
| 28 | **MCP Client** accesses externally hosted MCP tools published via an MCP server, configured in AI Agent Studio; agents add MCP servers as tools | ServiceNow docs, MCP Client landing; *Add an MCP server from AI Agent Studio*; *Adding an MCP Server in AI Agent Studio* | Brazil, 10 Sep 2026 | ✅ |
| 29 | MCP governance: intake (automatic discovery / MCP Catalog / manual from AI Control Tower) → **AI Steward** playbook approval → client registration via **AI Gateway**; roles `sn_ai_governance.ai_steward`; **MCP server approval playbook workflow**, **MCP server record**, **Add a Global MCP client** | ServiceNow docs, ai-control-tower/* | Brazil, 10 Sep 2026 | ✅ Strongest structural evidence for the thesis |
| 30 | **AI Guardian** is a real-time runtime protection layer; evaluates requests and responses for offensive content, prompt injection, sensitive topics; offensiveness logs by default and can block; prompt-injection detection "applies universally"; sensitive topics redirect to a fallback topic (HRSD/CSM Virtual Agent); logs in **AI Admin Hub**; baseline before blocking; PII anonymized; analytics; export; custom guardian; enable for AI agents | ServiceNow docs, *AI Guardian* | Brazil, 10 Sep 2026 | ✅ Content-scoped only — see §7.5 |
| 31 | Two-layer governance: lifecycle (AI Control Tower + AI Risk and Compliance) and runtime (AI Guardian) | ServiceNow docs, *AI Guardian* | Brazil, 10 Sep 2026 | ✅ |
| 32 | **CMDB CI creator AI agent** creates CIs; tools `Create new record for CI class`, `Get similar CI Classes`; used in agentic workflow "Create configuration item" | ServiceNow docs, *CMDB CI creator AI agent* | Brazil, 10 Sep 2026 | ✅ |
| 33 | CI definition and example classes (Windows Server, Data Center, Internet Cable, Apache Application, Virtual Machine) | ServiceNow docs, *CMDB CI creator AI agent* | Brazil, 10 Sep 2026 | ✅ |
| 34 | Sibling CMDB AI agents: certification & attestation manager, data ownership manager, health metrics manager, life cycle manager, principal class manager, CMDB search | ServiceNow docs, CMDB AI agents family | Brazil, 10 Sep 2026 | ✅ |
| 35 | **Workflow Data Fabric** description; powered by **Automation Engine** and **RaptorDB Pro**; **Zero Copy** connectors "without moving or duplicating data" (Databricks, Snowflake); includes **Knowledge Graph**; "available today in controlled go-to-market" | ServiceNow press release, newsroom.servicenow.com | **23 Oct 2024** | ✅ press release; "controlled go-to-market" is a 2024 status — not extrapolated |
| 36 | Workflow Data Fabric AI agents family (flow generation, product knowledge, product recommendation, spoke asset discovery, spoke install advisor); WFDF + Knowledge Graph integration; Zero Copy Connector for ERP AI agents | ServiceNow docs | Brazil, checked 1 Oct 2026 | ✅ |
| 37 | Doc-tree presence of AI Control Tower, AI Risk and Compliance, AI Gateway, AI Admin Hub, AI Agent Advisor, Knowledge Graph, Virtual Agent, Assistant Designer, Flow Designer, Service Portal, Guided Setup, AI Agent Analytics | ServiceNow docs TOC | Brazil, checked 1 Oct 2026 | ✅ |
| 38 | Availability limits: not all model providers for in-country SKUs; some features unavailable in FedRAMP/NSC DOD IL5/Australia IRAP-Protected, self-hosted, restricted environments; some region-limited; some AI products and skills not available in Regulated Markets (KB2593939) | ServiceNow docs availability notes | Brazil, checked 1 Oct 2026 | ✅ dated vendor statements, not a roadmap |
| 39 | Community/third-party MCP servers: `jschuller/mcp-server-servicenow`, `echelon-ai-labs/servicenow-mcp`, `habenani-p/servicenow-mcp` ("NowAIKit"), `leasstatt/servicenow-mcp-ai`, "Enterprise MCP Documentation" page | GitHub listings / third-party page | checked 1 Oct 2026 | ⚠ third-party, not vendor products; the `jschuller` README's "Runs alongside ServiceNow's native MCP Server" is a third-party claim |
| 40 | "**Agent Fabric**" | — (not found) | checked 1 Oct 2026 | ⚠ unverified — zero TOC and web hits; see §16.1 |
| 41 | "**Action Fabric**" | ServiceNow **Community** article title, linked from the MCP Server Console docs page | checked 1 Oct 2026 | ⚠ community-sourced, not a documentation-tree product name |
| 42 | "**A2A**" | Only inside the Community article title above | checked 1 Oct 2026 | ⚠ not a documented ServiceNow product surface; no section built on it |
| 43 | "**RaptorDB**" | — (docs map: zero hits) | retrieved 1 Oct 2026 | ⚠ verified only via the 23 Oct 2024 press release, as **RaptorDB Pro** |
| 44 | "**Build Agent**" | — (docs TOC: zero title matches) | retrieved 1 Oct 2026 | ⚠ asymmetry with the SDK `agentDescriptor` value `created_by_build_agent` — reported, not resolved |

**What this table does not contain, deliberately:** no productivity claim, no time-saving figure, no adoption statistic, no cost or licence figure, no MTTR improvement, no benchmark number, no roadmap statement, no competitor comparison, and no claim of any ServiceNow capability beyond its own published documentation. The performance figures present in the 23 Oct 2024 press release (source of row 35) are **not reproduced** anywhere in this guide.

---

## 16. What Could Not Be Verified, Glossary, and Cross-References

### 16.1 What Could Not Be Verified

The honest residue. Each item states exactly what was checked, when, and what can and cannot be concluded.

- **"Agent Fabric" — NOT FOUND.** Zero occurrences in the ServiceNow documentation table of contents retrieved this pass — the full Brazil *Enable AI Experiences* TOC, 371 KB, retrieved **2026-10-01** — and zero results in a web search. **What this means:** a name in wide circulation could not be traced to ServiceNow's own documentation as of 1 Oct 2026. **What it does not mean:** this guide does **not** assert that the name exists, and does **not** assert that it does not exist in the product. The precise statement is that it was not found in the documentation TOC or in a web search on 2026-10-01, and anyone who has seen the name attributed to ServiceNow should ask for the documentation page. It is a `⚠` for a reason.
- **"Action Fabric" — community-sourced only.** It appears **only as the title of a ServiceNow *Community* article** — *"Action Fabric: MCP Server, MCP Client, and A2A explained"* — linked from the **MCP Server Console** documentation page under "Helpful resources." It is therefore referenced by the vendor's own docs but is **not a product name in the documentation tree**. Label: `⚠`, community-sourced. The same Community list also links a "Model Context Protocol Server listing in the ServiceNow Store" and a "Designing your first custom ServiceNow MCP server" article.
- **"A2A" is not a documented ServiceNow product surface.** It appears only inside that same Community article title. No section of this guide is built on A2A, and it should not be treated as a verified ServiceNow capability on the strength of this pass.
- **"Build Agent" — an unresolved asymmetry.** Zero title matches in the documentation TOC, **yet** the SDK's `agentDescriptor` enum carries the value `'created_by_build_agent'`. Both facts are reported; the guide does **not** resolve the divergence — it records that a builder-creating descriptor exists while no product page titled "Build Agent" was found on 2026-10-01.
- **"RaptorDB" — poor documentation coverage.** Zero hits in the docs map retrieved this pass. The database name **RaptorDB Pro** is verified **only** from the vendor's press release of **23 October 2024** (per §11.2). Treat the technology as press-release-sourced, not documentation-sourced.
- **The `sn_skill_builder.admin` spelling inconsistency.** The skill AI-developer role is rendered **`sn_skill_builder.admin`** on one page and **`snskillbuilder.admin`** on another in the same documentation tree (both retrieved 1 Oct 2026). The inconsistency is in the source. Verify the exact role string against the target instance before using it in a build or an access request.
- **Search-tool limitation, recorded as a limitation and not as evidence of absence.** `web_search` returned empty result sets for several ServiceNow queries during this pass. An empty search result is a property of the tool, not a demonstration that the thing does not exist — which is why the negative findings above are stated as "not found in the sources checked," never as "does not exist."
- **Docs-retrieval method, recorded so a future maintainer can reproduce it.** The ServiceNow documentation site is a JavaScript-rendered Fluid Topics single-page application whose article text is **not** retrievable by plain page scraping. The facts in this guide were retrieved this pass **through the site's own content API while driving a live browser**. A maintainer who tries to re-verify by fetching the HTML directly will get an empty shell and may wrongly conclude an article is missing — the method matters as much as the URL.
- **A note on the two AI Agent Studio execution switches.** The guide treats the skill-level **Supervised/Autonomous** mode and the agent-level **copilot/autopilot** mode as a single guardrail ladder (§4.3, §7.3, §9). Both switches are individually documented ✅; the **combination policy** — which pairs the bank should permit — is **not** vendor-published, and is stated in this guide as architecture, not as a vendor specification.

### 16.2 Glossary

- **AI Agent Advisor** — the platform's advisory surface for agent creation; documented in AI Admin Center, AI Control Tower and AI Agent Studio ✅
- **AI Agent Orchestrator** — the component that centrally manages multiple AI agents and coordinates their collaboration ✅
- **AI Agent Studio** — the unified interface to create, manage and test AI agents and agentic workflows; requires the **ServiceNow Otto AI Agents** plugin ✅
- **AI Control Tower** — the lifecycle-governance surface; the intake path for MCP servers ✅
- **AI Gateway** — the surface where MCP clients are registered after a server is approved ✅
- **AI Guardian** — the runtime protection layer for offensive content, prompt injection and sensitive topics; **content-focused** and not a substitute for identity or segregation controls ✅
- **AI Skill Kit** — the environment for building custom generative AI skills; a skill is a prompt + deployment target + security controls ✅
- **AI User** — a `sys_user` record with `identity_type = ai_agent`, settable as an agent's `runAsUser`; the platform's named non-human identity ✅
- **Assistant Designer / Flow Designer / Service Portal / Virtual Agent** — adjacent platform surfaces named in the docs tree ✅
- **CI (configuration item)** — the CMDB's unit; "a representation of some type of infrastructure, application, software, hardware" belonging to a **class** ✅
- **copilot / autopilot** — agent-level execution modes; `copilot` is the documented preference when an agent creates, updates or deletes records ✅
- **Dynamic Orchestrator** — the mechanism that narrows which agents are considered in planning when more than the documented "ideal 8 to 10 AI agents" are available ✅
- **Dynamic User** — the run-as model in which an agent inherits roles from the invoking user ✅
- **Generative AI skill** — the platform's term of art for a named, workflow-specific generative-AI capability with an ACL and role restrictions ✅
- **Knowledge Graph** — the graph representation surfaced in the docs tree and supported by the MCP Server Console ✅
- **MCP (Model Context Protocol)** — the protocol by which MCP clients and servers exchange capabilities; the protocol itself is owned by [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md) ✅
- **MCP Client** — ServiceNow's outbound surface for consuming externally hosted MCP tools ✅
- **MCP Server Console** — ServiceNow's inbound surface for exposing instance functionality to external MCP clients ✅
- **Now Assist** — the **former** name of **ServiceNow Otto**; technical identifiers unchanged ✅
- **ServiceNow Otto** — the current name of the vendor's generative-AI product family ✅
- **ServiceNow AI Platform®** — the vendor's own registered-mark phrasing for the platform ✅
- **Supervised / Autonomous** — skill-level execution modes; Supervised requires human input during execution ✅
- **versionDetails[].state** — `draft | committed | published | withdrawn`; the agent version state machine, and the retirement mechanic ✅
- **Workflow Data Fabric** — the integrated data layer for workflows and agents; press-release-sourced 23 Oct 2024, with a docs presence at Brazil ✅
- **Zero Copy** — connectors that reach data sources without moving or duplicating data ✅

### 16.3 Cross-References

| Topic | Guide | Direction |
|---|---|---|
| The MCP protocol itself — discovery, disclosure, patterns | [`ai_llm/mcp_framework_tools_guide.md`](ai_llm/mcp_framework_tools_guide.md), [`ai_llm/mcp_discovery_guide.md`](ai_llm/mcp_discovery_guide.md), [`ai_llm/mcp_progressive_disclosure_guide.md`](ai_llm/mcp_progressive_disclosure_guide.md) | Protocol owner; this guide covers the ServiceNow endpoint only |
| Agent platform selection / harness / enterprise architecture | [`ai_llm/ai_agent_platform_selection_guide.md`](ai_llm/ai_agent_platform_selection_guide.md), [`ai_llm/agent_harness_engineering_guide.md`](ai_llm/agent_harness_engineering_guide.md), [`ai_llm/enterprise_agentic_platform_architecture_guide.md`](ai_llm/enterprise_agentic_platform_architecture_guide.md), [`ai_llm/agentic_solution_artifacts_guide.md`](ai_llm/agentic_solution_artifacts_guide.md) | General agent engineering; this guide specialises it |
| ITSM/ITIL process theory and capacity | [`operational_support_frameworks_guide.md`](operational_support_frameworks_guide.md), [`capacity_sizing_guide.md`](capacity_sizing_guide.md) | Process owner; this guide treats records as evidence |
| Assistive AI-for-IT (Otto/Now Assist in ITSM) | [`ai_llm/ai_for_it_guide.md`](ai_llm/ai_for_it_guide.md) §6.2 | Assistive-layer owner; this guide builds on it |
| Operations bridge — ITOM/AIOps Event Management + CMDB as integration source | [`opentext_operations_bridge_guide.md`](opentext_operations_bridge_guide.md) | Observability integration; not re-derived |
| Agentic workflow engines | [`agentic_workflows_guide.md`](agentic_workflows_guide.md), [`durable_ai_agent_workflows_guide.md`](durable_ai_agent_workflows_guide.md), [`temporal_workflow_guide.md`](temporal_workflow_guide.md) | Engine discipline; cross-referenced |
| ServiceNow as a foreign integration target (opposite direction) | [`ai_llm/copilot_studio_agent_engineering_knowledge_guide.md`](ai_llm/copilot_studio_agent_engineering_knowledge_guide.md) §4 | Different vantage point; not duplicated |
| Shared credentials and zero trust | [`zero_trust_legacy_estate_guide.md`](zero_trust_legacy_estate_guide.md), [`zero_trust_mainframe_guide.md`](zero_trust_mainframe_guide.md), [`zero_trust_network_architecture_guide.md`](zero_trust_network_architecture_guide.md) | Reused reasoning for the agent-identity thesis |
| Governance, risk, GRC, audit, data and API governance | [`../banking/grc_in_banking_guide.md`](../banking/grc_in_banking_guide.md), [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md), [`ai_llm/ai_governance_framework_guide.md`](ai_llm/ai_governance_framework_guide.md), [`ai_llm/ai_governance_bias_redteaming_guide.md`](ai_llm/ai_governance_bias_redteaming_guide.md), [`audit_as_code_guide.md`](audit_as_code_guide.md), [`data_governance_guide.md`](data_governance_guide.md), [`api_governance_guide.md`](api_governance_guide.md) | Regulatory frame; no jurisdiction's rule asserted |
| Human professional skills (sense c) | [`../personal/skill_gaps_enterprise_architect_guide.md`](../personal/skill_gaps_enterprise_architect_guide.md), [`../personal/agent_runtime_engineer_skill_gaps_guide.md`](../personal/agent_runtime_engineer_skill_gaps_guide.md), [`../personal/data_architect_skillgaps_guide.md`](../personal/data_architect_skillgaps_guide.md), [`../management/facilitation_skills_guide.md`](../management/facilitation_skills_guide.md), [`../management/communication_stakeholder_management_skills_guide.md`](../management/communication_stakeholder_management_skills_guide.md), [`../management/management_consulting_skills_guide.md`](../management/management_consulting_skills_guide.md) | A different literature entirely |
| Agent failures and sandboxing | [`ai_llm/llm_agents_failures_production_guide.md`](ai_llm/llm_agents_failures_production_guide.md), [`ai_llm/agent_sandboxing_strategies_guide.md`](ai_llm/agent_sandboxing_strategies_guide.md) | Failure catalogue and tool-execution containment |

### 16.4 Closing

ServiceNow's own material supports a precise reading of what the platform offers an agent: a first-class, named generative-AI **skill** with a mandatory ACL; the bridge that turns a skill into an agent tool; an agent record with a typed tool catalogue, a mandatory trigger run-as, and a documented preference for `copilot` mode when it writes; a first-class non-human identity in the **AI User**; an inbound MCP endpoint with configured tools and ACLs; an outbound MCP client; and — most tellingly — a governance path that routes MCP server onboarding through a named **AI Steward** and a lifecycle. None of that is in dispute, and none of it is criticised here. It is also not the whole story, because the platform's own features — AI Guardian included — guard *content*, while the controls that decide whether an agent's record can be trusted are *identity*, *segregation* and *evidence*. Those are the bank's to design, on a platform that documents the mechanisms for doing so. The Cymbal Bank exercise in §13 is the practical form of the argument: all three proposals looked like one project, and only the write scope and the identity distinguished them. Name the identity, keep the commit boundary, and the record keeps its meaning.

**connecting an agent to the system of record is an integration project; letting it write there is a controls decision.**
