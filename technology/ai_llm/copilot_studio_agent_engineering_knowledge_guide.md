# Copilot Studio Agent Engineering — The Agent Engineer's Syllabus

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** A role-knowledge map / study guide for the Copilot Studio and Power Platform agent-engineering role in an enterprise (banking-weighted) estate.
> **Repository:** github.com/jackliusr/research — `technology/ai_llm/`
> **Primary Sources:** Microsoft Learn (`learn.microsoft.com`) — Copilot Studio, Power Automate, Power Platform ALM and administration, Microsoft Credentials; this repo's `microsoft_power_platform_guide.md`, `financial_management_systems_guide.md`, and the `technology/ai_llm/` agent cluster.
> **Last Updated:** September 2026

---

### Abstract

This guide is a **syllabus**, not a platform anatomy. It maps the knowledge, skills, tooling, pitfalls, and study path for **one role: the Copilot Studio / Power Platform agent engineer** — the enterprise agent-builder who turns business intent into governed, integrated, observable agents. It is deliberately organised around the **eight responsibilities** a hiring manager for such a role would list, and for each responsibility it states (a) the knowledge areas and concepts, (b) the depth expected, (c) the tooling, (d) the common pitfalls, (e) the public study path, and (f) the sibling guide in this repo that owns the deeper pattern. It does **not** re-derive the platform: the platform deep-dive lives in `../microsoft_power_platform_guide.md` (§8 Copilot Studio/AI Builder/connectors, §9 governance and ALM); the ERP/FMS integration deep-dive lives in `../financial_management_systems_guide.md`; and the LLM-agent engineering cluster lives beside this file.

Three framing chapters bracket the eight areas: a **consolidated knowledge map** (dependency ordering, capability matrix, certification targets, interview questions), a **banking context** (regulated-industry considerations per area, condensed), and a **Cymbal Bank worked example** (an illustrative 90-day upskilling ramp). Every factual claim about Microsoft terminology is tagged `✅` (verified this pass at `learn.microsoft.com`), `⚠` (approximate, vendor-claimed, single-source, or not re-verified this pass), `⚠-knowledge`, or `❌` (could not be verified). The claims-audit table and the **What Could Not Be Verified** section close the loop honestly. The certification landscape is in flux as of this pass — the legacy Power Platform exam family is being retired and replaced by the AI/agent-focused **AB** family — so certification claims carry heavier `⚠` tagging than the product terminology.

---

### Companion Guides

| Guide | Relationship |
|---|---|
| [`../microsoft_power_platform_guide.md`](../microsoft_power_platform_guide.md) | **Primary platform sibling.** §8 Copilot Studio / AI Builder / connectors (platform mechanics); §9 governance and ALM (environments, managed environments, environment groups, DLP policies, CoE Starter Kit, solutions, deployment pipelines, licensing); §10 banking angle; §11 Cymbal Bank governance rollout. |
| [`../financial_management_systems_guide.md`](../financial_management_systems_guide.md) | FMS/ERP deep-dive — §3 treasury connectivity, §5 ERP vendor landscape incl. SAP. Owns responsibility 3 (SAP/Dynamics/ServiceNow in a bank estate). |
| [`enterprise_ai_platforms_guide.md`](enterprise_ai_platforms_guide.md) | The enterprise AI-platform view; where Copilot Studio sits among the major platforms. |
| [`ai_agent_platform_selection_guide.md`](ai_agent_platform_selection_guide.md) | Agent-platform selection criteria — build-vs-buy and Copilot Studio vs. code-first frameworks. |
| [`agent_scaffolding_guide.md`](agent_scaffolding_guide.md) | Agent construction patterns — instructions, tools, memory; the conceptual twin of "topics, tools, knowledge". |
| [`multi_agent_banking_guide.md`](multi_agent_banking_guide.md) | Multi-agent orchestration in a bank; owns the deeper orchestration pattern (responsibility 4). |
| [`hierarchical_multi_agent_frameworks_guide.md`](hierarchical_multi_agent_frameworks_guide.md), [`hybrid_multi_agent_systems_guide.md`](hybrid_multi_agent_systems_guide.md) | Orchestration topologies — decomposition, routing, supervisor patterns. |
| [`context_engineering_guide.md`](context_engineering_guide.md) | Knowledge grounding and context assembly — the theory behind knowledge sources. |
| [`mcp_discovery_guide.md`](mcp_discovery_guide.md), [`mcp_framework_tools_guide.md`](mcp_framework_tools_guide.md) | Model Context Protocol — servers, tools, resources; the MCP extension path for agents. |
| [`agentic_solution_artifacts_guide.md`](agentic_solution_artifacts_guide.md) | The artifact model for agentic delivery — what a "solution" contains. |
| [`agent_versioning_guide.md`](agent_versioning_guide.md), [`ai_agent_drift_guide.md`](ai_agent_drift_guide.md) | Versioning and drift — the ALM and observability concerns for agents. |
| [`agentops_guide.md`](agentops_guide.md) | Agent operations — monitoring, evaluation, cost, incident response (responsibility 8). |
| [`production_ready_llm_agents_guide.md`](production_ready_llm_agents_guide.md), [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md) | Production readiness and failure modes — fallback and error-handling theory (responsibility 4). |
| [`agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md) | Harness design — the reasoning loop beneath a Copilot Studio "harness". |
| [`autonomous_agents_guide.md`](autonomous_agents_guide.md) | Autonomous/event-triggered agents. |
| [`enterprise_agentic_platform_architecture_guide.md`](enterprise_agentic_platform_architecture_guide.md) | Platform architecture for agentic estates. |
| [`agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md) | Isolation and safe execution. |
| [`../enterprise_middleware_integration_platform_guide.md`](../enterprise_middleware_integration_platform_guide.md), [`../data_integration_frameworks_guide.md`](../data_integration_frameworks_guide.md), [`../legacy_integration_patterns_guide.md`](../legacy_integration_patterns_guide.md), [`../api_governance_guide.md`](../api_governance_guide.md) | Integration siblings (responsibility 2/3). |
| [`../cybersecurity_guide.md`](../cybersecurity_guide.md), [`../security_by_design_guide.md`](../security_by_design_guide.md), [`../beyond_zero_enterprise_security_guide.md`](../beyond_zero_enterprise_security_guide.md), [`../zero_trust_network_architecture_guide.md`](../zero_trust_network_architecture_guide.md), [`../llm_development_risks_security_guide.md`](../llm_development_risks_security_guide.md), [`../openbao_vs_vault_guide.md`](../openbao_vs_vault_guide.md) | Security/identity siblings (responsibility 7). |
| [`../cloud_providers_guide.md`](../cloud_providers_guide.md), [`../ai_platform_engineering_guide.md`](../ai_platform_engineering_guide.md) | Azure/cloud siblings. |
| [`../solution_architect_vs_devops_salary_guide.md`](../solution_architect_vs_devops_salary_guide.md) | Role/career context for the platform-engineering vs. delivery split. |
| [`../../banking/ai_genai_banking_compliance_guide.md`](../../banking/ai_genai_banking_compliance_guide.md), [`../../banking/mas_regulations_guidelines_guide.md`](../../banking/mas_regulations_guidelines_guide.md), [`../../banking/operational_resilience_framework_guide.md`](../../banking/operational_resilience_framework_guide.md), [`../../banking/enterprise_risk_management_guide.md`](../../banking/enterprise_risk_management_guide.md), [`../../banking/regtech_guide.md`](../../banking/regtech_guide.md) | Banking/regulatory siblings (banking context chapter). |

---

## Table of Contents

1. [Role Framing: The Enterprise Agent-Builder](#1-role-framing-the-enterprise-agent-builder)
2. [Responsibility 1 — Build and Configure Copilot Studio Agents, Topics, Tools, and Knowledge Sources](#2-responsibility-1--build-and-configure-copilot-studio-agents-topics-tools-and-knowledge-sources)
3. [Responsibility 2 — Agent Workflows: Power Automate, HTTP Actions, and Custom APIs](#3-responsibility-2--agent-workflows-power-automate-http-actions-and-custom-apis)
4. [Responsibility 3 — Integrating SAP, Dynamics, and ServiceNow](#4-responsibility-3--integrating-sap-dynamics-and-servicenow)
5. [Responsibility 4 — Agentic Orchestration: Decomposition, Multi-Step Execution, Fallback, and Error Handling](#5-responsibility-4--agentic-orchestration-decomposition-multi-step-execution-fallback-and-error-handling)
6. [Responsibility 5 — Solutions, Dataverse Dependencies, Environment Variables, and Connection References](#6-responsibility-5--solutions-dataverse-dependencies-environment-variables-and-connection-references)
7. [Responsibility 6 — CI/CD with Power Platform Pipelines, Azure DevOps, and GitHub](#7-responsibility-6--cicd-with-power-platform-pipelines-azure-devops-and-github)
8. [Responsibility 7 — Security: Microsoft Entra ID, DLP Policies, and Secure API Authentication](#8-responsibility-7--security-microsoft-entra-id-dlp-policies-and-secure-api-authentication)
9. [Responsibility 8 — Monitoring and Observability with Azure Monitor and Application Insights](#9-responsibility-8--monitoring-and-observability-with-azure-monitor-and-application-insights)
10. [The Consolidated Knowledge Map](#10-the-consolidated-knowledge-map)
11. [Banking Context: Regulated-Industry Considerations](#11-banking-context-regulated-industry-considerations)
12. [Cymbal Bank Worked Example: The 90-Day Agent-Engineering Ramp](#12-cymbal-bank-worked-example-the-90-day-agent-engineering-ramp)
13. [Claims Audit and What Could Not Be Verified](#13-claims-audit-and-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Closing](#15-closing)

---

## 1. Role Framing: The Enterprise Agent-Builder

### What the role is

The **Copilot Studio agent engineer** is the person who takes a business process — "answer policy questions for the contact centre", "triage a service request", "reconcile a supplier statement" — and delivers a **working, governed, integrated agent** on Microsoft's low-code agent platform. The role is best understood as the **professional-maker tier** of the Power Platform: above the citizen developer (who assembles an app or a flow for their own team), below the platform engineer / administrator (who owns environments, DLP policy, capacity and the tenant-wide operating model), and overlapping with the **solution architect** on design judgement.

The canonical Microsoft terminology this role works in — verified at `learn.microsoft.com` this pass — is:

- **Agent** — "an AI assistant that handles conversations and completes tasks", following instructions, drawing on **knowledge sources**, and using **tools** to take action. ✅
- **Topics** — "a portion of a conversation you design with connected steps, questions, and conditions"; on the **standard harness**, natural language understanding matches a request to the best topic. ✅
- **Tools / actions** — functions an agent can call, including MCP-published tools. ✅
- **Knowledge sources** — content the **maker** makes available to the agent at design time so every user gets the same grounding (distinct from an end-user **attachment**). ✅
- **Workflows** and **agent flows** — automations authored in Copilot Studio; agent flows can be attached to an agent as a tool that returns results. ✅
- **Harness** — the engine that carries out the work: the **GitHub Copilot harness** (reasoning-heavy, multi-step), the **standard harness** (rule-based/structured), and the **Copilot chat harness** (extends Microsoft 365 Copilot Chat). ✅
- **Channels** — where the agent is published (Microsoft Teams, Microsoft 365 Copilot, websites, mobile apps, telephony). ✅
- **Autonomous agents** — agents given their own account and an **event trigger** so they run proactively. ✅

### Where the role sits

Three tiers, and the agent engineer is the middle one:

| Tier | Who | Owns | Failure mode if absent |
|---|---|---|---|
| **Citizen developer** | Business analyst, ops lead | Their team's app/flow; departmental automation | Shadow IT, ungoverned connectors, unmanaged sprawl |
| **Agent engineer** (this role) | Professional maker / dev | End-to-end agent delivery: topics, tools, knowledge, workflow, integration, ALM, monitoring | Prototype purgatory — demos that never reach production |
| **Platform engineering / ADM / CoE** | Admin, architect, CoE lead | Environments, DLP, capacity, managed environments, CoE Starter Kit, operating model | Ungoverned estate, no cost control, no lifecycle |

The agent engineer's distinguishing value is **integration + productionisation**: anyone can click through a starter agent; the engineer makes it call SAP, retry on a throttled API, respect a DLP boundary, deploy through a pipeline, and show up in Application Insights.

### Depth expected per area

This is the **capability target** the eight sections below develop, stated up front so the study effort is calibrated:

| Area | Foundational | Intermediate | Advanced |
|---|---|---|---|
| 1. Agents/topics/tools/knowledge | Build a working agent | Multi-topic + generative orchestration | Harness selection, MCP, autonomous agents |
| 2. Workflows / Power Automate | Trigger→action cloud flow | Expressions, HTTP action, error scope | Custom connectors, retry/compensation |
| 3. SAP/Dynamics/ServiceNow | Know the connectors exist | OData/REST integration, auth | Hybrid/on-prem gateway, idempotency |
| 4. Orchestration | Multi-step linear | Fallback + human-in-the-loop | Decomposition, planner patterns, multi-agent |
| 5. Solutions/Dataverse | Managed vs unmanaged | Environment variables, connection refs | Dependency archaeology, layering |
| 6. CI/CD | Export/import | Power Platform Pipelines | Pipelines + Azure DevOps/GitHub, `pac` |
| 7. Security | Entra ID basics | DLP policy design, OAuth | Managed identity, vault, connector policy |
| 8. Observability | Built-in Monitor | Evaluations, alerts | App Insights telemetry, SLOs, drift |

The rest of this guide is the syllabus that fills that table in.

---

## 2. Responsibility 1 — Build and Configure Copilot Studio Agents, Topics, Tools, and Knowledge Sources

### (a) Knowledge areas and concepts

**The authoring surface.** Copilot Studio is a "graphical, low-code studio for building and managing AI-powered agents and workflows" — agents, workflows, and agent flows are built in one studio. ✅ An **agent** reasons through a request and decides the next step based on instructions, knowledge, and context; you can create one by describing it in plain language, then test it before publishing. ✅

**Harness.** Every agent runs on a **harness** and the choice "affects how your agent or workflow reasons, how complex a task it can take on, what it can do out of the box, and how it's billed." ✅ Three harnesses:

- **GitHub Copilot harness** — "for reasoning-heavy, multi-step work and complex business processes." ✅ Generative/agentic behaviour: the agent interprets the request and chooses tools rather than being scripted.
- **standard harness** — "for rule-based agents and structured, repeatable conversations." ✅ Uses NLU to route to a **topic**; falls back to generative answers from connected knowledge when no topic matches. ✅
- **Copilot chat harness** — "for extending Microsoft 365 Copilot Chat with your organization's knowledge." ✅

**Topics.** A topic is a designed slice of conversation (trigger phrases, questions, conditions, messages). On the standard harness, the **NLU** matches the user's request to the best topic. ✅ **Generative orchestration** (also called generative actions / the AI-based authoring model) changes this: instead of topic-by-topic scripting, the agent plans across topics, tools, and knowledge. ✅ Microsoft's topic-analytics note confirms the split: "If your agent uses generative orchestration, use conversation outcomes and themes on the Monitor page instead [of per-topic analytics]." ✅

**Tools and actions.** Tools are the agent's hands: connector actions, flows, prompts, MCP tools, and computer-use/other action types. **MCP (Model Context Protocol)** — verified — lets an agent connect to an MCP server and use its **Resources** (file-like data), **Tools** (functions the model can call), and **Prompts** (predefined templates). ✅ Copilot Studio "currently supports MCP tools and resources" and **requires generative orchestration to be turned on** to use MCP. ✅ When you connect to a non-Microsoft product (including an external MCP server), the maker is responsible for the tools and resources accessed. ✅

**Knowledge sources.** Verified source types: **Featured** — Public websites, SharePoint, OneDrive for Business, Salesforce, ServiceNow; **Advanced** — Azure DevOps Wiki, Azure DevOps Work Items, Custom Connector, Enterprise websites; plus uploaded files (PDF, Word, Excel, PowerPoint), Confluence, Jira, Dataverse tables, **Azure AI Search**, and **Copilot connectors** (non-Microsoft data indexed for Microsoft 365 Copilot). ✅ The distinction that matters: a **knowledge source** is maker-curated design-time grounding shared by all users; an **attachment** is an end-user file brought into a single conversation. ✅ Note the environment dependency: if the Dataverse **search-and-index setting** doesn't support agent knowledge, Dataverse/Dynamics 365/uploaded-file sources are unavailable until an administrator changes it. ✅

**Autonomous agents.** An **autonomous agent** is "an agent with an event trigger"; these agents run proactively from a payload rather than waiting for a chat, and their analytics "track from when an agent receives a payload from a trigger through any actions the agent runs in response." ✅ Some agents "can be given their own account so they can work proactively on tasks and take part in shared business processes." ✅

**Channels.** Publish targets include Microsoft Teams, Microsoft 365 Copilot, websites, mobile apps, and telephony; on the telephony channel sessions time out 3 minutes after an *End Conversation* event (vs. 30 minutes of inactivity elsewhere). ✅

**Cost/consumption.** Usage-based billing applies to using, building, testing, and evaluating agents and "might consume Copilot Credits"; licensing differs by harness. ✅

### (b) Key skills and the depth expected

- **Foundational:** build an agent from a description; write clear **instructions**; author 3–5 topics with trigger phrases, questions, and conditions; attach a cloud flow as a tool; add SharePoint and a public website as knowledge.
- **Intermediate:** choose the correct harness for the job and justify it; design a generative-orchestration agent across tools + knowledge; author **adaptive/agent flows**; add **Dataverse** and **Azure AI Search** knowledge; configure **MCP** servers and tools (with generative orchestration on); build an **autonomous agent** with an event trigger; test with the test panel and publish to Teams.
- **Advanced:** design the knowledge-grounding **strategy** (which source for which question, chunking/refresh implications, table-limit constraints); select and reason about harness capabilities and billing; design agent *identity* for autonomous agents; author **custom topics that fail gracefully** and hand off to a human; understand the fallback path when Dataverse indexing is restricted.

### (c) Tooling

Copilot Studio web app (`copilotstudio.microsoft.com`) ✅; the Copilot Studio test panel ✅; the agent build canvas; Power Platform admin center for environment-level switches (Dataverse search-and-index) ✅; Dataverse; SharePoint; Azure AI Search; Copilot connectors; MCP servers (onboarded via the MCP wizard, and optionally published as a connector for cross-tenant use) ✅. For the platform anatomy of how these fit together, see `../microsoft_power_platform_guide.md` §8.

### (d) Common pitfalls

- **Topic sprawl.** Hundreds of topics with overlapping triggers, each a maintenance liability; on a generative agent the correct answer is fewer topics, better tools, and grounded knowledge.
- **Knowledge = truth fallacy.** Adding a stale SharePoint library as knowledge reproduces the stale document; grounding quality is a *content* problem, not a toggle.
- **Attachment vs. knowledge confusion.** Users expect to "just upload my PDF"; the architect must decide which is in scope and how it is governed.
- **MCP as an ungoverned back door.** MCP tools configured directly in a maker's agent can bypass connector-centric DLP — Microsoft is explicitly evolving Copilot Studio virtual connectors into dedicated governance rules and managing MCP connectors via **advanced connector policies**. ✅
- **Ignoring the harness.** Building a deterministic, compliance-exact flow on the wrong harness (or a reasoning-heavy task on the standard harness) is a design error, not a tuning problem.
- **Testing in the dark.** The **Monitor** page does *not* show activity from the Copilot Studio test panel. ✅ Don't mistake a clean test-panel session for production telemetry.

### (e) Study path

- Microsoft Learn **Copilot Studio** training and the **Copilot Studio Academy** tracks (Recruit / Operative) — ⚠ the Academy is described by a third-party source as Microsoft's hands-on learning track; treat the exact track names as ⚠ until confirmed on Learn.
- **Microsoft Certified: AI Agent Builder Associate** (exam **AB-620**) — the exam tied to agent-building; the certification is referenced from the AB-100 page, and the exam itself is a new-2026 addition. ⚠ Verify the exact exam page and skills-measured before booking.
- Hands-on labs: the Copilot Studio "core concepts: knowledge, tools, topics" lab pattern (⚠ third-party/community-published).
- For agent-construction theory, read `agent_scaffolding_guide.md` and `context_engineering_guide.md`.

### (f) Deeper pattern

Platform mechanics of agents/topics/tools/knowledge live in [`../microsoft_power_platform_guide.md`](../microsoft_power_platform_guide.md) §8. Agent construction theory lives in [`agent_scaffolding_guide.md`](agent_scaffolding_guide.md); MCP in [`mcp_discovery_guide.md`](mcp_discovery_guide.md) and [`mcp_framework_tools_guide.md`](mcp_framework_tools_guide.md); knowledge grounding in [`context_engineering_guide.md`](context_engineering_guide.md); harness reasoning in [`agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md).

---

## 3. Responsibility 2 — Agent Workflows: Power Automate, HTTP Actions, and Custom APIs

### (a) Knowledge areas and concepts

**Cloud flows vs. desktop flows.** Power Automate has two flow species. **Cloud flows** run in the service on triggers and actions (the connective tissue between agents, connectors, and systems). **Desktop flows** "broaden the existing robotic process automation (RPA) capabilities in Power Automate", are built in the Power Automate for desktop designer with prebuilt drag-and-drop actions or recording, and automate "legacy applications, such as terminal emulators, modern web and desktop applications, Excel files, and folders", interacting with the machine "by using application UI elements, images, or coordinates." ✅ The agent-building relevance: cloud flows are the default tool; desktop flows are the escape hatch for systems with no API (a fixed-income terminal, a legacy core-banking screen).

**Triggers and actions.** A cloud flow starts on a **trigger** (manual, automated, scheduled, or — the agent-relevant one — invoked from an agent/flow). The **connector** is "a proxy or a wrapper around an API"; a **connection** is "a stored authentication credential for a connector." ✅ Agents call flows as **tools**; agent flows can "run prompts, call agents, and include human review steps" and be attached to an agent "as a tool that returns results to the agent." ✅

**The HTTP action and custom connectors.** When no certified connector exists, the engineer drops to the raw HTTP action and hand-rolls the request: method, URI, headers, auth, JSON body, and parsing of the response. **Custom connectors** "allow makers to create their own connectors to integrate with external systems or services not covered by the standard set of certified connectors." ✅ Custom connectors can be **certified** (Microsoft-tested for security, reliability, and compliance) and can expose **MCP-enabled** endpoints as tools. ✅

**Expressions and transformation.** Flow **expressions** (Power Fx-adjacent functions and workflow expression language) do the data shaping between systems: parse JSON, build ODATA queries, format dates, compose payloads. The developer certification explicitly lists "Power Automate workflow expressions" and "Complex Power Fx logic" as candidate skills. ✅

**Premium boundaries.** Connectors are classed standard vs. **premium**; premium use rights are licensed separately ("Licenses granting premium use rights are required for all managed environments"), and the HTTP action and custom connectors sit behind premium/enterprise licensing. ✅ ⚠ (the exact connector-by-connector premium classification should be checked against the current connector reference before committing to a licence design.)

**Error handling in flows.** Cloud flows expose run-after configuration, **scopes** (try/catch/finally pattern), retry policies, and terminate actions; desktop flows use block-level error handling. ⚠-knowledge (well-established product behaviour, not re-verified this pass).

### (b) Key skills and the depth expected

- **Foundational:** create an automated cloud flow with a trigger and connector actions; pass parameters in from an agent and return a typed result; handle a null response without crashing.
- **Intermediate:** use expressions and `Parse JSON`; call an API with the HTTP action using OAuth; wrap risky steps in a **scope** with run-after error branches; set retry policy; author a **custom connector** (swagger/OpenAPI definition, auth scheme, test connection); connect a desktop flow for a legacy screen.
- **Advanced:** design **idempotent** actions so a retried agent call does not double-post; design **compensation** for multi-step transactional workflows; use the **Power Platform CLI** to manage flows (`pac flow`); decide flow-vs-agent-flow-vs-Azure-Logic-Apps-vs-serverless by cost, latency, and governance; design for throttling (429) and backpressure with exponential backoff.

### (c) Tooling

Power Automate cloud flows and desktop flows ✅; the Power Automate for desktop designer ✅; the **HTTP action** and **HTTP with Microsoft Entra ID** connector ⚠-knowledge; **custom connectors** ✅; the connector reference (`learn.microsoft.com/connectors`); Power Platform CLI (`pac`) ✅; Azure Logic Apps and Azure Functions (for heavier orchestration) ⚠-knowledge; the on-premises data gateway ⚠-knowledge. Agent-side authoring of workflows/agent flows is covered in §2 and in `../microsoft_power_platform_guide.md` §5 (Power Automate) and §8 (connectors).

### (d) Common pitfalls

- **Connection-per-maker.** Flows built without **connection references** bind to a personal connection; when the maker leaves, the flow breaks (see §6).
- **No retry policy.** A single transient 429/503 from SAP or ServiceNow fails the whole agent turn; default retry is often off or too shallow for premium APIs.
- **Non-idempotent writes.** The agent retries a "create" and the system gets two records. Every mutating action needs a dedupe key or a natural idempotency token.
- **HTTP action as a security hole.** Stuffing credentials into the HTTP action headers in plain text, or disabling TLS validation, is a classic audit finding; use the Entra-ID-authenticated connector, a managed identity, or a Key Vault-backed secret.
- **Desktop flows as the default.** RPA is brittle (UI coordinates change); using desktop flows where an API exists is a maintenance tax.
- **Premium surprise.** Designing on premium connectors and only then discovering the licence gap.
- **Timeout mis-sizing.** Long-running flows can exceed the synchronous agent-call window; a batch reconciliation belongs in an asynchronous pattern with a callback, not in a conversational tool.

### (e) Study path

- Microsoft Learn Power Automate paths (cloud flows, expressions, custom connectors, desktop flows/RPA). ✅ (broad Learn catalogue; exact path names vary — ⚠).
- **Microsoft Certified: Power Platform Developer Associate** — required exams **PL-400** and, from **October 16, 2026**, **AB-400** ("The updated AB-400 exam will be available beginning October 16, 2026"). ✅ PL-400 is "transitioning to AB-400 to align the exam within Microsoft's certification portfolio." ✅
- The retired **PL-500 / Power Automate RPA Developer Associate** ("This certification and the renewal assessment are retired.") ✅ covered desktop flows and RPA — useful as a *knowledge* syllabus even though it can no longer be earned.
- For the failure-handling theory, read `production_ready_llm_agents_guide.md` and `llm_agents_failures_production_guide.md`.

### (f) Deeper pattern

Power Automate mechanics in [`../microsoft_power_platform_guide.md`](../microsoft_power_platform_guide.md) §5; connector/DLP interaction in §8–§9. Integration-artifact depth in [`../enterprise_middleware_integration_platform_guide.md`](../enterprise_middleware_integration_platform_guide.md) and [`../data_integration_frameworks_guide.md`](../data_integration_frameworks_guide.md); API design in [`../api_governance_guide.md`](../api_governance_guide.md); failure theory in [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md).

---

## 4. Responsibility 3 — Integrating SAP, Dynamics, and ServiceNow

### (a) Knowledge areas and concepts

This responsibility is where "agent engineer" stops being a low-code role and starts being an integration-engineering role. The three named systems represent three integration archetypes: **SAP** (the ERP/backbone, complex and latency-sensitive), **Dynamics 365** (Microsoft's own CRM/ERP, native to Dataverse), and **ServiceNow** (the ITSM/ITSM-adjacent system of record for incidents, changes, and service catalogues).

**Dynamics 365 and Dataverse.** Dynamics 365 is the Microsoft business-application family and **Dataverse** is the data platform beneath it. This is the "native" integration path: Copilot Studio can use **Dataverse tables** as a knowledge source ✅, and Copilot Studio's own agents, flows, and solutions live in the Power Platform/Dataverse plane, so Dynamics integration is largely in-platform (tables, views, business process flows, the Dataverse connector and the Dynamics 365 connectors). ⚠-knowledge for the specific connector names. The knowledge-source page confirms that **Dynamics 365** data can be a knowledge source where the environment's Dataverse indexing supports it. ✅

**ServiceNow.** ServiceNow appears as a **Featured knowledge source** in Copilot Studio — "connect to ServiceNow to let the agent reference service management content." ✅ There is also a ServiceNow connector for actions (create incident, query CMDB) — ⚠-knowledge on exact action set.

**SAP.** SAP is the hardest of the three. The realistic paths:

- **SAP OData services** (SAP Gateway, S/4HANA OData APIs) consumed via the OData connector or a **custom connector**. ⚠-knowledge.
- **SAP Business Accelerator Hub** APIs / the SAP ERP connector family. ⚠-knowledge.
- **RFC/BAPI/IDoc** — the classic SAP interface; reaching these from Power Platform generally means an intermediate layer (SAP PO/PI, MuleSoft, Azure Logic Apps, or a custom service) rather than the connector itself. ⚠-knowledge.
- **On-premises data gateway** for on-prem SAP, or private network connectivity for RISE/GROW cloud. ⚠-knowledge.

**The universal integration concerns.** Authentication (OAuth 2.0, service-to-service identities, certificates); **idempotency** (SAP postings must not double-post); **latency** (SAP real-time OData vs. batch extracts — an agent should not synchronously query a heavy SAP report); **transactional integrity** and **compensation** (an agent that creates a service request in ServiceNow and posts a document in SAP must not leave the two half-done on failure); and **master-data reconciliation** (customer/account IDs differ across systems).

### (b) Key skills and the depth expected

- **Foundational:** know which certified connectors exist for each system; connect a Dataverse table and query it from an agent; add ServiceNow knowledge; call a simple ServiceNow incident-create action.
- **Intermediate:** read an OpenAPI/OData contract and build a **custom connector**; authenticate to a non-Microsoft API with OAuth 2.0 through the connector's security tab; shape request/response with expressions; choose a knowledge source vs. an action for a given question.
- **Advanced:** design the integration **topology** (direct connector vs. gateway vs. middleware); design idempotent SAP-facing operations; design compensation across two systems; reason about rate limits and quotas on SAP/ServiceNow APIs; decide when the answer is "put Azure API Management in front" rather than a naked connector; design the **data residency** and audit story for each hop.

### (c) Tooling

Dataverse ✅; certified connectors and **custom connectors** ✅; **ServiceNow** knowledge source ✅; SAP OData/Gateway, SAP Business Accelerator Hub, RFC/BAPI/IDoc bridges ⚠-knowledge; **on-premises data gateway** ⚠-knowledge; Azure Logic Apps, Azure API Management, Azure Service Bus/Event Grid, Azure Functions ⚠-knowledge; MCP servers for non-Microsoft systems ✅ (protocol verified; per-system availability is not a guarantee). The ERP/FMS vendor landscape and treasury connectivity are owned by `../financial_management_systems_guide.md`.

### (d) Common pitfalls

- **Treating SAP as just another REST API.** SAP OData is powerful but not a free-for-all; unconstrained agent queries can hammer the ERP and trip ABAP query limits.
- **Synchronous heavy queries.** An agent turn should not block on a multi-million-row SAP report; push to async + callback or a pre-computed table.
- **Double-posting.** No idempotency key on a posting action → duplicate journal entries on retry.
- **Ignore the gateway topology.** Forgetting that on-prem/system integration requires the on-premises data gateway (and its own high-availability/licensing governance) is a classic late-cycle blocker.
- **Cross-system identity drift.** The agent "knows" a customer by CRM ID but SAP keys on a different number; without a mapping service the integration silently mis-routes.
- **No contract versioning plan.** Vendor APIs (ServiceNow, SAP) version and deprecate; a custom connector pinned to an old schema is a time bomb.
- **Knowledge-source vs. transaction confusion.** A ServiceNow *article* answer is fine; reflecting a ServiceNow *state change* needs a governed action, not a generative answer.

### (e) Study path

- Microsoft Learn integration paths: Dataverse, custom connectors, the on-premises data gateway, Azure Logic Apps. ✅ (broad catalogue)
- SAP and ServiceNow vendor training/accreditation (SAP Learning; ServiceNow Now Learning) — ⚠ not Microsoft Learn; treat as external.
- **PL-400 / AB-400** ("Develop integrations" is an assessed PL-400 domain ✅) and **AB-100** (multi-service integration architecture ✅).
- For the banking ERP/FMS estate, read `../financial_management_systems_guide.md` §3 and §5.

### (f) Deeper pattern

FMS/ERP treasury and SAP vendor depth in [`../financial_management_systems_guide.md`](../financial_management_systems_guide.md) §3, §5. Middleware and integration-platform depth in [`../enterprise_middleware_integration_platform_guide.md`](../enterprise_middleware_integration_platform_guide.md); data movement in [`../data_integration_frameworks_guide.md`](../data_integration_frameworks_guide.md); legacy interfaces in [`../legacy_integration_patterns_guide.md`](../legacy_integration_patterns_guide.md); API governance in [`../api_governance_guide.md`](../api_governance_guide.md).

---

## 5. Responsibility 4 — Agentic Orchestration: Decomposition, Multi-Step Execution, Fallback, and Error Handling

### (a) Knowledge areas and concepts

**Generative orchestration.** The platform's core orchestration switch: with generative orchestration on, the agent reasons across instructions, topics, tools, and knowledge and chooses its own path — rather than NLU routing a single request to a single topic. MCP requires it ✅, and the analytics model changes with it (conversation outcomes/themes instead of per-topic analytics) ✅. Orchestration is therefore a *design* discipline: the engineer curates the tool set, the knowledge grounding, and the instructions that the planner reasons over.

**Task decomposition and multi-step execution.** A business task ("onboard this client") decomposes into steps (verify identity, create the record, issue the welcome pack, schedule the review). The engineer's job is to decide which steps the agent plans dynamically and which are **deterministic sub-flows** — the split between *reasoning* and *scripted automation*. Copilot Studio supports both: **workflows** give "the predictability of automation with the flexibility of AI" ✅, and **agent flows** can "run prompts, call agents, and include human review steps" ✅.

**Multi-agent orchestration.** Agents can call agents. The architect certification AB-100 explicitly assesses "skills in designing multi-agent orchestrated solutions" and expertise with "open standards and protocols, including Agent2Agent (A2A) and Model Context Protocol (MCP)". ✅ So the orchestration vocabulary — supervisor/router, delegation, tool brokerage — maps onto Copilot Studio's connected-agent and MCP patterns. The repo's multi-agent cluster owns the topology theory.

**Human-in-the-loop.** Workflows "include built-in testing and human-in-the-loop controls for enterprise-ready automation." ✅ Conversations can be transferred to a customer service representative ✅; a flow step can pause for approval. This is the control that makes an agent deployable in a regulated process.

**Fallback and error handling.** Three layers: (1) **semantic fallback** — on the standard harness, when a request matches no topic, the agent generates an answer from knowledge ✅; (2) **escalation** — hand off to a human or another agent when confidence is low; (3) **technical fallback** — flow/action error handling (retry, compensate, terminate, or degrade to a safe response). Production agent design treats all three as first-class, not as an afterthought.

**Guardrails and control.** Prompt-injection resistance, tool-scope minimisation, output constraints, and cost caps sit on top of orchestration; the AB-100 skill statement calls out "defending against prompt manipulation" and tracking "audit trails". ✅

### (b) Key skills and the depth expected

- **Foundational:** build a multi-step agent that calls two tools in sequence and returns a result; add a human-approval step to a flow; configure a system fallback message.
- **Intermediate:** decide reasoning-vs-deterministic per step; design the tool set so the planner cannot pick an unsafe action; implement escalation paths; implement retry + graceful degradation; instrument the path so each step is traceable.
- **Advanced:** design multi-agent orchestration (router/supervisor, delegation boundaries, A2A/MCP interop); design decomposition for a genuinely complex process; design loop/infinite-recursion guards and cost ceilings; design the "what happens when the ERP is down" degradation contract; design evaluation sets that exercise the fallback branches, not just the happy path.

### (c) Tooling

Copilot Studio generative orchestration ✅; topics and agent flows ✅; **connected agents**/multi-agent authoring ⚠-knowledge (multi-agent design is an assessed architect skill ✅; the exact product surface for "connected agents" should be confirmed against current Copilot Studio docs ⚠); **MCP** ✅; A2A ✅ (named in the AB-100 syllabus); human-in-the-loop approval steps ✅; Power Automate error-handling (scopes, run-after, retries) ⚠-knowledge; Copilot Studio **evaluations** ("validate quality with test sets and a shared grader library") ✅.

### (d) Common pitfalls

- **Over-orchestrating.** Agents reasoning their way through deterministic steps that should be a single transaction — slower, costlier, flakier.
- **No loop guard.** A planner that re-tries a failing tool forever burns credits and latency; cap iterations and tool calls.
- **Fallback as an afterthought.** Shipping only the happy path; the first production incident is the first fallback test.
- **Tool bloat.** Every tool added dilutes selection accuracy and widens the blast radius; tool curation is a security control, not just a UX one.
- **Silent failure.** A tool returns an error your flow swallows → the agent confabulates success. Failures must be *typed* and surfaced.
- **Unbounded human handoff.** Escalation without queue capacity or a service-desk SLA just moves the failure.
- **No cost ceiling.** An agentic loop with a premium connector can consume Copilot Credits fast; orchestration must respect a per-task budget.

### (e) Study path

- Microsoft Learn Copilot Studio authoring + orchestration paths; ⚠ exact module names vary.
- **AB-100** (Agentic AI Business Solutions Architect) explicitly assesses agentic-first design, multi-agent orchestration, A2A/MCP, and prompt-manipulation defence. ✅
- Repo study: `multi_agent_banking_guide.md`, `hierarchical_multi_agent_frameworks_guide.md`, `hybrid_multi_agent_systems_guide.md`, `autonomous_agents_guide.md`, `production_ready_llm_agents_guide.md`, `llm_agents_failures_production_guide.md`.

### (f) Deeper pattern

Orchestration patterns in [`multi_agent_banking_guide.md`](multi_agent_banking_guide.md); topologies in [`hierarchical_multi_agent_frameworks_guide.md`](hierarchical_multi_agent_frameworks_guide.md) and [`hybrid_multi_agent_systems_guide.md`](hybrid_multi_agent_systems_guide.md); autonomy in [`autonomous_agents_guide.md`](autonomous_agents_guide.md); production hardening in [`production_ready_llm_agents_guide.md`](production_ready_llm_agents_guide.md); failure modes in [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md); platform architecture in [`enterprise_agentic_platform_architecture_guide.md`](enterprise_agentic_platform_architecture_guide.md).

---

## 6. Responsibility 5 — Solutions, Dataverse Dependencies, Environment Variables, and Connection References

### (a) Knowledge areas and concepts

**Solutions as the unit of ALM.** "Solutions are the mechanism for implementing application lifecycle management (ALM) in Power Apps and Power Automate." ✅ This is the single most important mental model in the whole role: an agent, its flows, its connectors, its tables, its environment variables, and its connection references are **solution components**, and only what is *in a solution* can be transported and managed as a unit.

**Managed vs. unmanaged.** **Unmanaged** solutions are "developed" and used in development environments; exported unmanaged versions "should be checked into your source control system" and are "your source for Microsoft Power Platform assets." ✅ **Managed** solutions are "deployed" to non-development environments (test, UAT, SIT, production), are "considered a build artifact", cannot be edited directly, and cannot be exported. ✅ You "can't import a managed solution into the same environment that contains the originating unmanaged solution." ✅

**Solution components, publishers, and dependencies.** "Anything that can be included in a solution is a component." ✅ A solution can be up to **95 MB**. ✅ Every solution has a **publisher**, and its **prefix** ("a mechanism to help avoid naming collisions") should be set before metadata is created because names can't be changed later. ✅ **Dependencies** are tracked: you can't install a solution whose base solution is missing, and you can't uninstall a base while a dependent solution is installed. ✅

**Solution lifecycle.** **Create** (author/export unmanaged), **Update** (deploy changes to a parent managed solution; you can't delete components), **Upgrade** (removes unused components; supports staged upgrade), **Patch** (small hotfix layered on the parent; you can't delete components). ✅

**Environment variables.** Solution components that carry per-environment configuration (a URL, a flag, a connector endpoint) out of the artifact so the same solution behaves correctly in dev/test/prod. Pipelines explicitly deploy "connections, connection references, and environment variables." ✅ ⚠ the detailed environment-variable-vs-secret model should be confirmed against the current ALM docs page (the URL I tried returned 404 this pass).

**Connection references.** "A connection reference is a solution component that contains a reference to a connection about a specific connector." ✅ Solution-aware canvas apps and solution-aware flows "bind to a connection reference instead of directly to a connection"; at import, "a connection is provided for all the connection references so any referencing flows can be turned on automatically." ✅ Flows use connection references for *all* connectors; canvas apps only for implicitly shared (non-OAuth) connections. ✅ Ownership of a connection reference **can't be transferred** from the modern Solutions area (only via classic solution explorer). ✅

**Dataverse dependencies.** Dataverse tables, columns, relationships, and views are components; custom tables become dependencies of the agents/flows that use them. Deleting a managed solution loses "data stored in custom tables that are part of the managed solution" ✅ — a data-loss trap, not just a config rollback.

### (b) Key skills and the depth expected

- **Foundational:** work inside a solution (never the default solution); create a publisher with a sensible prefix; export unmanaged, import managed; understand why your agent "disappeared" when you changed environments.
- **Intermediate:** use **environment variables** and **connection references** to make a solution environment-portable; resolve import-time dependency errors; add all agent components (agent, flows, custom connectors, tables) to the right solution.
- **Advanced:** design the **solution segmentation** strategy (one big solution vs. base + extension solutions by publisher/dependency); read and repair **dependency graphs**; design the `Update`/`Upgrade`/`Patch` strategy against a release cadence; design the **variable/secret** boundary (what is an environment variable vs. Azure Key Vault reference); design the Dataverse table lifecycle so a rollback doesn't destroy data.

### (c) Tooling

Power Apps/Power Automate solution explorer ✅; managed vs. unmanaged solutions ✅; solution publishers and prefixes ✅; **environment variables** ✅; **connection references** ✅; Dataverse ✅; the **preferred solution** setting ✅ (named in the solution-concepts page); Power Platform CLI (`pac solution`) ⚠-knowledge; classic solution explorer for connection-reference privilege changes ✅. Platform mechanics live in `../microsoft_power_platform_guide.md` §9.

### (d) Common pitfalls

- **Working in the default solution.** Components "leak" out of any transportable artifact; the default solution can never be exported cleanly.
- **Personal connections.** No connection reference → import breaks, flows disable, and the fix requires the original maker.
- **Hard-coded config.** Environment-specific URLs/IDs baked into an agent or flow; clones fail on first import.
- **Dependency archaeology at release time.** Discovering a missing base solution on production-import night instead of in the dev loop.
- **Publisher-per-team chaos.** Multiple prefixes and cross-publisher dependencies that can't be reorganised later.
- **Managed-solution edit attempts.** Editing a managed component creates an unmanaged dependency the managed solution can't be uninstalled past.
- **Assuming solutions carry data.** "Pipelines, or solutions in general, don't contain data stored within Dataverse tables." ✅ Configuration migrates; data does not.
- **Deleting managed solutions casually.** Uninstall deletes the managed solution's custom-table data ✅ — the opposite of a safe rollback.

### (e) Study path

- Microsoft Learn ALM learning path: **"Simplify Microsoft Power Platform deployments by using pipelines"** ✅ (linked from the pipelines page); solution concepts and managed-solution modules ✅.
- **PL-400 / AB-400** — "Build Power Platform solutions" is an assessed AB-400 domain ✅ and "Configure Microsoft Dataverse" an assessed PL-400 domain ✅.
- **PL-200** was the functional-consultant exam for Dataverse/environment configuration; it is **retired (August 31, 2026)** ✅ — use it as a syllabus reference only.

### (f) Deeper pattern

Solutions/ALM/platform governance in [`../microsoft_power_platform_guide.md`](../microsoft_power_platform_guide.md) §9. Agent artifact model in [`agentic_solution_artifacts_guide.md`](agentic_solution_artifacts_guide.md); versioning in [`agent_versioning_guide.md`](agent_versioning_guide.md).

---

## 7. Responsibility 6 — CI/CD with Power Platform Pipelines, Azure DevOps, and GitHub

### (a) Knowledge areas and concepts

**Power Platform Pipelines (native, in-product).** Microsoft's first-party ALM deployment service. Verified facts: pipelines "democratize application lifecycle management (ALM)" by bringing ALM automation and CI/CD "into the service"; admins "configure automated deployment pipelines in minutes"; makers "initiate in-product deployments with a few clicks"; professional developers can extend pipelines and run them "using the Power Platform command line interface (CLI)." ✅

**The pipeline model.** A **pipelines host** environment holds the pipeline definitions; environments are linked as **development** or **target** environments; a preconfigured pipeline runs a solution through **stages** (dev → QA → production). ✅ Verified constraints:

- Deployments are **prevalidated** against the target environment — "missing dependencies and other issues are detected before deployment." ✅
- **Connections and environment variables are provided upfront and validated before deployment begins.** ✅
- The **same solution artifact** passes through stages in sequential order; the system "prevents any tampering or modification to the exported solution artifact", so "customization can't bypass QA environments or your approval processes." ✅
- **Managed solutions are automatically exported and stored** in the pipelines host for every deployment (unmanaged too, for source-control purposes). ✅
- You **cannot deploy unmanaged** solutions; you can't deploy multiple solutions at once (one deployment per solution, same pipeline reusable); the "current implementation uses a single development environment for a given solution." ✅
- **Approvals** use **delegated deployments** ✅; **pipelines extensibility** + Power Automate allows custom steps ✅.
- Licensing: the host "should be a production environment" but need not be managed; **all other pipeline environments must be managed environments**; developer environments can use the developer plan. ✅
- **Starting February 2026, Microsoft began auto-enabling managed environments for pipeline targets** not already enabled. ✅

**Pipelines vs. the ALM Accelerator.** Both are valid; pipelines are "more simplistic… set up and managed with less effort" and are "an official Microsoft Power Platform product feature", while the **ALM Accelerator** (part of the CoE Starter Kit) is "sometimes a better fit for more advanced ALM scenarios." ✅ Microsoft's guidance: "use pipelines for core deployment functionality, and when needed, extend pipelines to integrate with other CI/CD tools." ✅

**Azure DevOps / GitHub.** For the pro-dev path, the ecosystem provides **Power Platform Build Tools** for Azure DevOps and **Power Platform GitHub Actions** for GitHub, both wrapping the **Power Platform CLI (`pac`)** — solution pack/unpack, import/export, and pipeline management (`pac pipeline`). ⚠-knowledge (the specific task/action names should be confirmed against current docs; the CLI command group `pac pipeline` is verified ✅).

**Environments and environment strategy.** CI/CD presupposes a **dev → test → UAT → prod** environment topology, managed environments, environment groups for policy inheritance, and a branching strategy (typically trunk-based with solution-aware source control). Platform depth in `../microsoft_power_platform_guide.md` §9.

### (b) Key skills and the depth expected

- **Foundational:** export a managed solution and import it to test; understand what a deployment settings file contains; use a manual pipeline.
- **Intermediate:** configure a Power Platform pipeline (host, environments, stages); enable managed environments on targets; configure delegated-deployment approvals; use `pac solution` to pack/unpack and `pac pipeline` to deploy; wire a git repo for solution source control.
- **Advanced:** design the **end-to-end CI/CD topology** (native pipelines for maker-led change, Azure DevOps/GitHub for pro-dev change, and when to extend one into the other); design **build-once-deploy-many** with environment variables + connection references; design the branch/release model; design **quality gates** (solution checker, pipeline prevalidation, approval); design the **rollback** strategy (update/upgrade/patch); automate the CoE/ALM Accelerator path for complex multi-solution estates.

### (c) Tooling

Power Platform Pipelines ✅; pipelines host and stages ✅; **delegated deployments** ✅; **pipelines extensibility** ✅; **Power Platform CLI (`pac`)** ✅; **ALM Accelerator** / CoE Starter Kit ✅ (named as the advanced alternative); Azure DevOps + Power Platform Build Tools ⚠-knowledge; GitHub + Power Platform GitHub Actions ⚠-knowledge; solution checker ⚠-knowledge; source control (git) ⚠-knowledge.

> **Repo gap note.** This repository has **no dedicated CI/CD guide**; the CI/CD depth for Power Platform is owned by `../microsoft_power_platform_guide.md` §9 (ALM, deployment pipelines). Any generic pipeline/DevOps content is not duplicated here. ⚠

### (d) Common pitfalls

- **Pipelines as a source-control substitute.** Pipelines automate *deployment*; you still need a git repo for unmanaged solution source and history ("Exported unmanaged versions of your solutions should be checked into your source control system"). ✅
- **Unmanaged target environments.** Pipeline targets must be managed — an unmanaged target breaks the governance model and, from Feb 2026, is auto-converted anyway. ✅
- **Late connection binding.** Not pre-configuring connection references and environment variables → post-import manual work and disabled flows.
- **Trying to skip a stage.** It does not work by design — the artifact is locked and must pass stages sequentially. ✅
- **One-solution-at-a-time assumption.** Multiple solutions need multiple deployments; release planning must account for it.
- **Ignoring the single-dev-environment constraint.** Multi-developer teams working in isolated dev environments are not supported by the current pipeline implementation. ✅
- **Forgetting that data doesn't deploy.** A pipeline ships configuration, not Dataverse rows; data migration is a separate discipline.
- **Power BI gaps.** "Power BI Dashboards (preview) and Power BI Datasets (preview) are not currently supported in pipelines." ✅

### (e) Study path

- Microsoft Learn: **"Simplify Microsoft Power Platform deployments by using pipelines"** learning path ✅; managed-solution and ALM modules ✅.
- **PL-400 / AB-400** (create a technical design, ALM strategy) ✅; **AB-100** ("Creating a cohesive application lifecycle management (ALM) strategy for agentic-first solutions" and an environment strategy are listed responsibilities). ✅
- **PL-600** (architect-level, incl. ALM) is **retired (June 30, 2026)** ✅ — historically the architect ALM syllabus; superseded conceptually by AB-100.

### (f) Deeper pattern

ALM/governance in [`../microsoft_power_platform_guide.md`](../microsoft_power_platform_guide.md) §9. Versioning in [`agent_versioning_guide.md`](agent_versioning_guide.md); artifact model in [`agentic_solution_artifacts_guide.md`](agentic_solution_artifacts_guide.md).

---

## 8. Responsibility 7 — Security: Microsoft Entra ID, DLP Policies, and Secure API Authentication

### (a) Knowledge areas and concepts

**Microsoft Entra ID.** The identity plane (formerly Azure Active Directory). Every Power Platform connection authenticates through it; users, groups, service principals, and (increasingly) agent identities live here. Agent access to channels (Teams, websites) and to connectors is an identity question before it is a feature question. The platform's own docs reason in terms of "user, group, or service principal" identity for connection sharing. ✅ ⚠-knowledge: the specific **Entra Agent ID / agent identity** product surface for Copilot Studio agents is evolving and was **not verified this pass — see the What Could Not Be Verified section.**

**Data policies / DLP.** "Data policies are a critical aspect of maintaining data security and compliance within the Microsoft Power Platform ecosystem", acting "as guardrails to help reduce the risk of users unintentionally exposing organizational data." ✅ They govern connectors across Power Apps, Power Automate, and Copilot Studio.

Verified connector classes and controls:

- **Certified connectors** — Microsoft-tested "for security, reliability, and compliance." ✅
- **Custom connectors** — maker-built; "require careful consideration to ensure that they comply with data policies and don't compromise data security." ✅
- **Virtual connectors** — "connectors that appear in data policies for administrators to control, but they're not based on a RESTful API"; Microsoft provides several **to govern Copilot Studio**, turning off Copilot/chatbot features; they are "evolving into their own dedicated governance rules, separate from both data policies and ACP." ✅
- **MCP connectors** — expose MCP-enabled API endpoints as tools; "you can manage and restrict these connectors and their MCP servers through advanced connector policies." ✅
- **Advanced connector policies (ACP)** — a newer, more robust governance feature for real certified connectors; ACP **does not support virtual connectors** and won't. ✅
- **Connections** — saved credentials stored in the environment. ✅

**Design-time vs. runtime enforcement.** A data policy blocks the *maker* from saving an app/flow that uses a blocked connector (design-time) **and** can disable the underlying connection so a running flow fails (runtime). ✅ **Policy change propagation** is asynchronous: save → cascade to environments → resources poll → violations suspend/quarantine the resource → blocked connections disabled → running usage fails. ✅ Latency is "within an hour" typically, "up to 24 hours" in extreme cases. ✅

**Environment groups.** Used for policy/rule inheritance across environments. ✅ (Platform depth in `../microsoft_power_platform_guide.md` §9.)

**Secure API authentication.** Patterns: **OAuth 2.0** through the connector's security configuration (verified: "OAuth credentials for the SharePoint connector" is the example of a connection ✅, and "OAuth connections can only be explicitly shared with a user representing a service principal" ✅); **service principals / managed identities** for unattended flows ⚠-knowledge; **Azure Key Vault** for secrets, referenced rather than embedded ⚠-knowledge; the on-premises data gateway for on-prem APIs ⚠-knowledge. Certificate/secret rotation and least-privilege scopes are the operational disciplines.

**Agent-specific security.** Tool-scope minimisation, prompt-injection defence, data-exfiltration prevention, and audit trails. The architect cert names "securing AI models and data workflows, including detecting and resolving vulnerabilities, enforcing data residency and access controls… maintaining audit trails, and defending against prompt manipulation." ✅

### (b) Key skills and the depth expected

- **Foundational:** understand Entra ID users/groups/service principals; know that connectors are classed standard/business/premium/blocked; read a data policy.
- **Intermediate:** design a **DLP policy** (business vs. non-business connector groups) for a bank; understand design-time vs. runtime effects and propagation latency; configure OAuth in a custom connector; use environment groups for policy inheritance; reason about virtual connectors for Copilot Studio.
- **Advanced:** design the **connector governance strategy** at tenant scale (DLP + ACP + virtual-connector rules + MCP restriction); design the identity model for autonomous agents (their own account/service principal); design a secrets architecture (Key Vault, rotation, no secrets in flow definitions); design data-residency and audit controls; design prompt-injection and exfiltration guardrails; map every control to a regulatory obligation.

### (c) Tooling

Microsoft Entra ID (formerly Azure AD) ✅-knowledge; **Power Platform admin center data policies** ✅; **advanced connector policies** ✅; Copilot Studio **virtual connectors** ✅; **MCP connector policies** ✅; connections and connection references ✅; environment groups ✅; DLP for Power Automate / Power Apps / Copilot Studio ✅; Azure Key Vault ⚠-knowledge; managed identities ⚠-knowledge; the CoE Starter Kit ⚠-knowledge; Microsoft 365 audit logs and Dataverse auditing ⚠-knowledge.

### (d) Common pitfalls

- **Blocklist-only DLP.** A default-block policy with a permissive exception list quietly leaves a business-data connector exposed to a consumer connector.
- **Forgetting the runtime path.** A policy that looks correct but leaves an existing flow running on a now-blocked connector until the 24-hour propagation completes.
- **Copilot Studio governance gaps.** Treating MCP tools and virtual connectors as immune to DLP — Microsoft explicitly had to build dedicated rules and ACP support because they weren't covered by classic policy. ✅
- **Secrets in plaintext.** API keys pasted into flow actions or custom-connector headers.
- **Over-privileged service accounts.** An unattended flow running as a global admin because "it was easy."
- **Not planning agent identity.** A proactive agent with no designed identity can't be least-privileged or audited properly.
- **Ignoring injection.** Giving an agent write-tools plus untrusted knowledge sources is a data-exfiltration path.
- **Treating security as a gate, not a design.** Late-cycle DLP discovery reshapes the whole solution.

### (e) Study path

- Microsoft Learn Power Platform admin/security paths; the **Power Platform Well-Architected** security pillar ⚠ (named as expected architect knowledge in the PL-600 syllabus ✅).
- **AB-100** — significant security content (model/data-workflow security, prompt manipulation, audit trails, access controls). ✅
- Repo study: `../cybersecurity_guide.md`, `../security_by_design_guide.md`, `../beyond_zero_enterprise_security_guide.md`, `../zero_trust_network_architecture_guide.md`, `../llm_development_risks_security_guide.md`, `../openbao_vs_vault_guide.md`.

### (f) Deeper pattern

Governance/DLP/CoE in [`../microsoft_power_platform_guide.md`](../microsoft_power_platform_guide.md) §9. Security doctrine in [`../security_by_design_guide.md`](../security_by_design_guide.md), [`../zero_trust_network_architecture_guide.md`](../zero_trust_network_architecture_guide.md), and [`../llm_development_risks_security_guide.md`](../llm_development_risks_security_guide.md); secrets in [`../openbao_vs_vault_guide.md`](../openbao_vs_vault_guide.md).

---

## 9. Responsibility 8 — Monitoring and Observability with Azure Monitor and Application Insights

### (a) Knowledge areas and concepts

**Copilot Studio built-in monitoring.** The **Monitor** page in Copilot Studio "shows you comprehensive data for your agent, from an overview of key metrics to in-depth usage analytics." ✅ Verified specifics:

- Monitoring is available in **all geographies**; **Monitor data is retained up to 360 days**, while **session details and transcripts are available for the last 28 days**; timestamps are **UTC**. ✅
- The experience is tailored for **conversational agents** and **autonomous agents** (event-triggered), and a **hybrid view** appears when an agent has both. ✅
- **Active-user metrics** (DAU/MAU) are "only supported if you set your agent to require authentication under Settings > Security > Authentication." ✅
- A **Summary** card uses Copilot to generate an AI summary of engagement, sentiment, and trends; a **customer comments summary** (preview) clusters feedback. ✅
- **Conversation outcomes** — `Escalated`, `Resolved`, `Abandoned` — plus **themes**, **customer satisfaction** score and trend are the key outcome metrics. ✅
- **Topic Monitor** is "only available for agents in classic mode" and for topic-associated sessions; on generative orchestration you use conversation outcomes and themes instead. ✅
- **The Monitor page doesn't show test-panel activity.** ✅
- Analytics can be shared view-only via the **Analytics Viewer** role; transcript access additionally needs the **Bot Transcript Viewer** security role. ✅

**Evaluations.** "Evaluations — validate quality with test sets and a shared grader library before and after you publish." ✅ This is the platform's pre-deployment quality gate and the closest thing to a regression suite for an agent.

**Azure Monitor / Application Insights.** Azure Monitor is the Azure observability control plane (metrics, logs, alerts, Application Insights for app telemetry, Log Analytics). The *generic* pattern is that Power Platform and Copilot Studio health/diagnostics surface through Power Platform admin-center analytics, the Microsoft 365/Dataverse audit trail, and — where the estate is Azure-native — Azure Monitor/Application Insights for the services the agent calls (APIs, Functions, Logic Apps, the gateway), correlated by trace/operation IDs. ⚠ **the specific "Copilot Studio → Application Insights" connector/export surface was not verified this pass** (see What Could Not Be Verified); treat the agent-telemetry-to-App-Insights path as ⚠ until confirmed.

**Flow and connector analytics.** Power Automate provides flow-level run analytics (success/failure, duration) and the admin center provides connector/service health; these are the operational telemetry behind an agent's tool calls. ⚠-knowledge.

**AgentOps framing.** Beyond dashboards, observability for agents means **traces** (per-step reasoning and tool calls), **cost telemetry** (Copilot Credits per task/agent), **quality telemetry** (evaluation scores, outcome mix), and **drift monitoring** (behaviour change after a model/knowledge update). The repo's `agentops_guide.md` owns this discipline.

### (b) Key skills and the depth expected

- **Foundational:** read the Monitor page; find resolution vs. escalation rates; filter by date range; download a transcript.
- **Intermediate:** build an **evaluation set** and grade agent quality pre-release; interpret conversation outcomes/themes; set up sharing with the Analytics Viewer/Bot Transcript Viewer roles; configure Power Automate flow-failure alerts.
- **Advanced:** design the **observability architecture** (what is native Monitor vs. Azure Monitor vs. App Insights vs. Dataverse audit); design **SLOs** for the agent (containment rate, CSAT, latency, cost-per-task); design **drift detection** and re-evaluation triggers; design cost and capacity governance (Copilot Credits); design the audit and retention story against 360-day/28-day platform limits; instrument the *integration* tier (API/Logic App/Functions) in Application Insights so a failing SAP call is visible.

### (c) Tooling

Copilot Studio **Monitor** page ✅; **evaluations** ✅; **Analytics Viewer** / **Bot Transcript Viewer** roles ✅; Azure Monitor ✅-knowledge; **Application Insights** ⚠; Log Analytics ⚠-knowledge; Power Automate analytics ⚠-knowledge; Power Platform admin center analytics ⚠-knowledge; Microsoft 365 audit / Dataverse auditing ⚠-knowledge; Power BI for custom reporting ⚠-knowledge. (Copilot Studio has out-of-the-box Power BI reporting references in the pipelines/analytics docs ✅ the *reference* to Power BI reporting exists; the specific report names are ⚠.)

> **Repo gap note.** This repository has **no dedicated observability/monitoring guide**; the closest is `agentops_guide.md` (LLM-agent operations) and `../solution_architect_vs_devops_salary_guide.md` (the DevOps/AIOps role split). ⚠

### (d) Common pitfalls

- **Confusing test-panel silence with health.** The Monitor page excludes test sessions; a "clean" test proves nothing about production. ✅
- **Ignoring the authentication prerequisite.** No auth → no active-user metrics. ✅
- **Legacy topic analytics on a generative agent.** Topic Monitor works only in classic mode; on generative orchestration you must use outcomes/themes. ✅
- **Retention surprise.** Building a compliance report on session detail older than 28 days — transcripts are gone even though aggregate data lasts 360 days. ✅
- **Dashboard-only observability.** No traces, no evaluation gate, no cost attribution → you can see *that* it failed, not *why*.
- **No drift coupling.** A knowledge-source or model update ships without a re-evaluation, and quality silently decays.
- **Cost blindness.** Copilot Credits consumed by build/test/evaluate as well as run; a runaway loop is a budget incident.
- **Orphaned integration telemetry.** The agent's own monitoring is fine but the downstream API failure is invisible because no one instrumented the integration tier.

### (e) Study path

- Microsoft Learn Copilot Studio analytics/monitoring guidance ✅ (product docs); Power Platform monitoring paths ✅.
- Azure Monitor / Application Insights learning paths ⚠-knowledge (generic Azure observability).
- Repo study: `agentops_guide.md`, `ai_agent_drift_guide.md`, `production_ready_llm_agents_guide.md`.

### (f) Deeper pattern

Agent operations in [`agentops_guide.md`](agentops_guide.md); drift in [`ai_agent_drift_guide.md`](ai_agent_drift_guide.md); production readiness in [`production_ready_llm_agents_guide.md`](production_ready_llm_agents_guide.md); role context in [`../solution_architect_vs_devops_salary_guide.md`](../solution_architect_vs_devops_salary_guide.md).

---

## 10. The Consolidated Knowledge Map

### 10.1 Dependency ordering — what to learn first

The eight areas have a real prerequisite graph. Learning in dependency order avoids the classic trap of attempting ALM or observability before the agent and its integration exist.

```
Layer 0  (foundation)     Entra ID + Power Platform environment/data model basics
                          ── you cannot reason about governance or ALM without this ──
Layer 1  (build)          §1 Agents / topics / tools / knowledge
                          §5 Solutions / Dataverse / env vars / connection refs
                          ── coexists with §1: build INSIDE a solution from day one ──
Layer 2  (connect)        §2 Workflows / Power Automate / HTTP / custom APIs
                          §3 SAP / Dynamics / ServiceNow integration
                          ── §2 is the mechanism §3 rides on ──
Layer 3  (compose)        §4 Agentic orchestration / decomposition / fallback
                          ── requires §1 (tools/knowledge) + §2 (actions) to orchestrate ──
Layer 4  (govern)         §7 Security / Entra ID / DLP / secure API auth
                          ── gates everything, designed in, not bolted on ──
Layer 5  (ship)           §6 CI/CD / Pipelines / Azure DevOps / GitHub
                          ── requires §5 (solutions) + §7 (approved connectors) ──
Layer 6  (operate)        §8 Monitoring / observability / evaluations / drift
                          ── requires everything above to have something to observe ──
```

**Reading of the graph.** §1 and §5 are the true starting pair — everyone builds inside a solution, so ALM hygiene is a *day-one* habit, not an advanced topic. §2 precedes §3 (you learn the flow mechanism, then the specific ERP/ITSM systems). §4 is the first genuinely "senior" area: it presupposes fluent §1–§3. §7 is not a late chapter in the reading order but a *cross-cutting* gate: the DLP/identity constraints shape §1–§4 designs, so a smart learner reads §7 once early (to know the boundaries) and once late (to design the controls). §6 and §8 are the ship-and-operate pair and are the clearest markers of a *professional* rather than a *demo* engineer.

### 10.2 Capability matrix — what "good" looks like at each depth

| # | Area | Foundational — evidence | Intermediate — evidence | Advanced — evidence |
|---|---|---|---|---|
| 1 | Agents/topics/tools/knowledge | Ships a working agent with clear instructions and 3 topics | Designs harness choice + generative orchestration + MCP + Dataverse knowledge | Owns the grounding strategy and autonomous-agent identity |
| 2 | Workflows / Power Automate | Builds a trigger→action flow returning a typed result | Uses expressions, HTTP action, error scopes, a custom connector | Designs idempotency and compensation across systems |
| 3 | SAP/Dynamics/ServiceNow | Connects a Dataverse table / ServiceNow knowledge | Builds OAuth custom connectors to SAP OData / ServiceNow | Owns integration topology, latency, idempotency, residency |
| 4 | Orchestration | Multi-step agent with two tools | Designs reasoning-vs-deterministic split, escalation, retries | Designs multi-agent/A2A + loop, cost and injection guards |
| 5 | Solutions/Dataverse/env vars/conn refs | Builds inside a solution with a publisher | Uses env vars + connection refs for portability | Owns solution segmentation, dependency graph, upgrade/patch |
| 6 | CI/CD | Exports managed, imports to test | Configures pipelines, stages, approvals, `pac` | Owns build-once-deploy-many + DevOps/GitHub integration |
| 7 | Security/DLP/API auth | Knows Entra ID basics and connector classes | Designs DLP + OAuth custom connector auth | Owns ACP/virtual/MCP governance, vault, agent identity |
| 8 | Monitoring/observability | Reads Monitor; finds outcome mix | Builds evaluations; shares analytics roles | Owns SLOs, drift, cost telemetry, App Insights correlation |

### 10.3 Certifications worth targeting

The Power Platform certification landscape shifted materially in 2026: the legacy **PL-200 / PL-600 / PL-500** credentials were retired and an AI/agent-focused **AB** family introduced. Verified statuses this pass are below. **Read every entry as `⚠` unless it carries `✅`** — Microsoft moves these goalposts and the skill areas change between update cycles.

| Exam | Credential | Status (verified this pass) | Relevance to this role |
|---|---|---|---|
| **PL-200** Microsoft Power Platform Functional Consultant | Power Platform Functional Consultant Associate | **Retired Aug 31, 2026** ✅ | Historical Dataverse/env-config syllabus; no longer earnable |
| **PL-400** Microsoft Power Platform Developer | (part of Power Platform Developer Associate) | **Active; "transitioning to AB-400"** ✅ | The developer core: Dataverse, Power Apps, automation, integrations |
| **AB-400** Power Platform Developer Associate | Power Platform Developer Associate | **Available from Oct 16, 2026** ✅ | AI-powered business solutions on Power Platform — the PL-400 successor |
| **PL-600** Microsoft Power Platform Solution Architect | Power Platform Solution Architect Expert | **Retired June 30, 2026** ✅ | Historical architect syllabus (incl. Well-Architected, ALM) |
| **PL-500** Power Automate RPA Developer | Power Automate RPA Developer Associate | **Certification and renewal retired** ✅ | Historical desktop-flow/RPA syllabus |
| **AB-100** Agentic AI Business Solutions Architect | Agentic AI Business Solutions Architect (expert) | **Active** ✅; requires an associate cert from a published list | The flagship architect target: agentic-first design, multi-agent, A2A/MCP, security, ALM, ROI |
| **AB-620** | **Microsoft Certified: AI Agent Builder Associate** | Credential referenced from the AB-100 page ✅; the exam page did not resolve this pass ⚠ | The agent-builder target for this exact role |
| **AB-410** | **Microsoft Certified: Intelligent Applications Builder Associate** | Credential referenced from the AB-100 page ✅ | App-builder track companion to AB-620 |
| **PL-900** Power Platform Fundamentals | Power Platform Fundamentals | Third-party sources say it remains the entry point ⚠ | On-ramp |
| **AZ-900** Azure Fundamentals | Azure Fundamentals | Third-party source recommends it as a companion ⚠ | Azure footing for Monitor/App Insights/identity |

**Practical certification plan for this role:** PL-900/AZ-900 (if new) → **AB-620** (AI Agent Builder — the role's core) → **AB-400** (developer/integration depth) → **AB-100** (architect ceiling). Treat PL-400 as a still-live developer exam *only* while it exists; assume AB-400 replaces it. ⚠ Confirm each exam's current skills-measured and retirement date on `learn.microsoft.com/credentials` before booking — the AB family moved from beta to live during 2026. ⚠

### 10.4 Sample interview / knowledge-check questions per area

**§1 Agents/topics/tools/knowledge**
- *Explain the difference between a knowledge source and an attachment, and when each is appropriate.* (Expected: maker-curated shared grounding vs. per-conversation user file ✅.)
- *You have a compliance process. Standard harness or GitHub Copilot harness — and why?* (Expected: harness affects reasoning complexity, capabilities, billing ✅.)

**§2 Workflows/Power Automate/HTTP/custom APIs**
- *When do you reach for a custom connector instead of the HTTP action, and vice versa?* (Expected: reuse, governance, certification, MCP exposure vs. one-off ⚠.)
- *Design an idempotent "create service request" action.* (Expected: dedupe key, natural idempotency token, retry-safe.)

**§3 SAP/Dynamics/ServiceNow**
- *An agent must read a SAP customer balance in real time. Sketch the path and name the risks.* (Expected: OData/Gateway/custom connector; latency, throttling, gateway topology, idempotency ⚠-knowledge.)
- *How do you keep ServiceNow and SAP from ending up half-updated?* (Expected: compensation/saga, transactional boundaries.)

**§4 Orchestration**
- *How do you decide which steps an agent plans vs. which are deterministic sub-flows?* (Expected: risk, compliance, cost, latency.)
- *What are your three fallback layers, and how do you test them?* (Expected: semantic fallback ✅, escalation/handoff ✅, technical error handling; test via evaluations ✅.)

**§5 Solutions/Dataverse/env vars/conn refs**
- *Why did the flow break after a production import, and how does a connection reference prevent it?* (Expected: personal connection vs. connection reference bound at import ✅.)
- *What survives and what is lost when you uninstall a managed solution?* (Expected: all customizations removed; custom-table data lost ✅.)

**§6 CI/CD**
- *Can a maker deploy straight to production, bypassing QA, using pipelines?* (Expected: no — same artifact, sequential stages, tamper-proof ✅.)
- *Pipelines vs. ALM Accelerator — when do you choose which?* (Expected: pipelines for core/simpler; ALM Accelerator for advanced scenarios ✅.)

**§7 Security/DLP/API auth**
- *A maker adds an MCP tool that reaches a consumer cloud. What stops the data leaving?* (Expected: data policies + ACP + MCP restriction + Copilot Studio virtual-connector governance ✅.)
- *How long can a policy change take to fully enforce, and why does it matter at release time?* (Expected: typically <1h, up to 24h ✅.)

**§8 Monitoring/observability**
- *Why does your dashboard show zero active users?* (Expected: agent not set to require authentication ✅.)
- *You need 90 days of transcripts for an audit. What does the platform give you?* (Expected: 28-day session/transcript retention vs. 360-day aggregate ✅ — plan external export.)

---

## 11. Banking Context: Regulated-Industry Considerations

A bank does not deploy the same agent a retailer does. The extra constraints are regulatory (MAS, EBA, SR 11-7-style model risk), operational (resilience, outsourcing), and reputational (an agent that mis-states a fee or leaks PII). The Power Platform governance depth is owned by `../microsoft_power_platform_guide.md` §9–§11 and the AI-compliance depth by `../../banking/ai_genai_banking_compliance_guide.md`; what follows is the condensed, per-area translation.

### 11.1 Per-area regulated-industry considerations

| # | Area | Banking-specific consideration |
|---|---|---|
| 1 | Agents/topics/knowledge | Grounding content must be the *approved* version of a policy/product document; an agent answering from a stale wiki is a conduct-risk event. Agent conversations with customers may be in scope for records retention. |
| 2 | Workflows / APIs | A flow that initiates a payment or changes a customer record is a *material* automated decision path needing approval, audit, and maker-checker controls. |
| 3 | SAP/Dynamics/ServiceNow | Integration crosses trust boundaries (general ledger, customer master); data residency, idempotency, and reconciliation are audit-relevant. Cross-ref the FMS depth below. |
| 4 | Orchestration | Human-in-the-loop is not optional for regulated decisions; escalation must route to a qualified human with capacity. Auditability of *why* the agent chose a step is expected. |
| 5 | Solutions/Dataverse | Customer data in Dataverse inherits records-management and retention obligations; environment boundaries must reflect data classification. |
| 6 | CI/CD | Change management (ITIL-style) applies to agent releases; approval gates and an immutable artifact are compliance features, not niceties. |
| 7 | Security/DLP | DLP is the bank's primary exfiltration control on citizen-developed agents; a permissive policy is a board-level risk. Least privilege and separation of duties are mandatory. |
| 8 | Monitoring/observability | Model-risk governance expects ongoing monitoring, outcome tracking, and evidence for supervisors; the 28-day transcript window must be complemented by an export/retention strategy. |

### 11.2 DLP and governance for a bank (condensed)

The bank's control stack, drawn from `../microsoft_power_platform_guide.md` §9–§11:

- **Environment architecture** — separated dev/test/UAT/prod, managed environments, environment groups for policy inheritance, and a Default environment locked down (no production workloads in the personal-productivity default).
- **Connector governance** — **DLP data policies** grouping connectors into business/non-business/blocked; **advanced connector policies** for fine-grained certified-connector control; Copilot Studio **virtual connectors** and **MCP connector** restriction; the general principle that no agent may bridge a business-data connector to a consumer cloud.
- **Identity and access** — Entra ID groups for makers; least-privilege security roles on Dataverse; service principals for unattended flows; **connection references** so credentials are owned and governed, not personal.
- **Change control** — solutions-only development, managed-solution deployment, **Power Platform Pipelines** with **delegated-deployment approvals**, audit logging.
- **Operating model** — a **Center of Excellence / ADM** function, the **CoE Starter Kit** for inventory and compliance, and an intake/triage process that classifies each proposed agent by data sensitivity and autonomy. ✔ The ADM/CoE pattern is verified in the platform sibling; the CoE Starter Kit is named in the pipelines page as the ALM Accelerator's home ✅.

### 11.3 SAP, Dynamics, and ServiceNow in a bank estate (condensed)

Banks run SAP (often S/4HANA) for finance/GL, Dynamics 365 or a native CRM for client relationships, and ServiceNow for ITSM and increasingly for risk/ops workflows. The integration reality — owned in depth by `../financial_management_systems_guide.md` §3, §5 — condensed for this syllabus:

- **SAP** is reached through SAP Gateway/OData where exposed, and through an intermediate middleware or the on-premises data gateway for RFC/BAPI/IDoc-era interfaces. Real-time reads are permissible for light queries; heavy postings and reports belong in asynchronous batch with a reconciliation step. ⚠-knowledge.
- **Dynamics 365** is the *native* integration because it shares Dataverse: knowledge sources, tables, and connectors are first-party, so the engineering effort is configuration and governance rather than protocol work. ✅ (Dataverse knowledge source); ⚠-knowledge (specific Dynamics connectors).
- **ServiceNow** is a **Featured knowledge source** ✅ and an action target via connector; the governance question is whether the agent may *change* state (create/close a record) or only *read*.
- **Cross-cutting** — the bank keeps a **system-of-record map** (which system owns which attribute), an **idempotency/reconciliation convention** for postings, and a **data-classification** rule for what may cross each integration boundary. The `../../banking/operational_resilience_framework_guide.md` and `../../banking/enterprise_risk_management_guide.md` siblings frame the resilience and risk-model expectations.

---

## 12. Cymbal Bank Worked Example: The 90-Day Agent-Engineering Ramp

> **Illustrative.** Cymbal Bank is this repo's fictional persona. The plan below is a *worked example* of how a bank might upskill an agent-engineering team against this syllabus. The dates, roles, and targets are invented for illustration and are not verified facts.

### 12.1 The setup

Cymbal Bank has adopted Copilot Studio for internal agents (policy Q&A for the contact centre, an IT service-desk triage agent, and an onboarding agent that touches SAP and ServiceNow). The Center of Excellence has stood up managed environments and a baseline DLP policy — see `../microsoft_power_platform_guide.md` §11 — and now needs a four-person **agent-engineering squad** (two professional makers, one integration-leaning developer, one engineer with an ops/monitoring bent) operating at *intermediate* depth across all eight areas within a quarter.

### 12.2 The 8-area upskilling track

Each row pairs a syllabus area with a **Cymbal Bank artifact** that proves the skill. The rule: no theory without an artifact.

| # | Area | The artifact each engineer must produce | Assessment |
|---|---|---|---|
| 1 | Agents/topics/tools/knowledge | A live internal agent with instructions, ≥4 topics, generative orchestration, and two knowledge sources (SharePoint + Dataverse) | Peer demo; grounding spot-check against the approved policy doc |
| 2 | Workflows / Power Automate / APIs | A flow that returns a typed result to the agent and calls a REST API through a custom connector with OAuth | Idempotency review; failure-path walkthrough |
| 3 | SAP/Dynamics/ServiceNow | A ServiceNow read action and a Dataverse/Dynamics table read used by the agent; a documented SAP path (even if via middleware) | Integration runbook exists; system-of-record map line signed off |
| 4 | Orchestration | The onboarding agent extended with a human approval step and a tested fallback path | Evaluation set covering one happy path + two failure paths |
| 5 | Solutions/env vars/conn refs | The agent and all its components in a named solution with a publisher, env vars, and connection references | Import to a clean test environment succeeds with zero manual fixes |
| 6 | CI/CD | A Power Platform pipeline with dev → QA → prod stages and a delegated-deployment approval | Publish a non-trivial change end-to-end via pipeline only |
| 7 | Security/DLP/API auth | A DLP impact review of their agent; a custom connector using OAuth and no inline secrets | CoE security checklist passes |
| 8 | Monitoring/observability | A Monitor-based health review plus an evaluation gate wired into the release | First monthly agent health report delivered |

### 12.3 The certification plan (illustrative)

| Engineer | Primary target | Secondary | Rationale |
|---|---|---|---|
| Professional maker A | **AB-620** (AI Agent Builder Associate) ⚠ | PL-900 | The role's core agent-builder credential |
| Professional maker B | **AB-620** ⚠ | **AB-400** ⚠ | Agent building plus developer/integration depth |
| Integration developer | **AB-400** / PL-400 ⚠ | AZ-900 | Integration and pro-dev path; PL-400 still live as of this pass ✅ |
| Ops-leaning engineer | **AB-400** ⚠ | AZ-900 | Observability and automation; Azure footing |
| Squad lead (architect) | **AB-100** (Agentic AI Business Solutions Architect) ✅ | — | Needs an eligible associate cert first ✅ |

**Caveat (illustrative plan, real caution):** the AB family moved from beta to live during 2026 and the PL family retired through mid-2026; Cymbal Bank's L&D should re-verify every exam's current status on Microsoft Learn before committing budget. ⚠

### 12.4 The 90-day ramp

| Phase | Days | Focus | Exit criterion |
|---|---|---|---|
| **Ground** | 1–10 | Environment + solution hygiene; Entra ID and DLP basics (§5, §7); the Cymbal Bank system-of-record map | Everyone builds inside a solution with a publisher; can explain the DLP boundary |
| **Build** | 11–25 | Agents/topics/tools/knowledge (§1); workflows (§2) | Area-1 and Area-2 artifacts demoed |
| **Connect** | 26–40 | SAP/Dynamics/ServiceNow (§3) + custom connector + gateway topology | End-to-end read path from agent → system via connector |
| **Compose** | 41–55 | Orchestration, decomposition, fallback, human-in-the-loop (§4) | Area-4 artifact + evaluation set |
| **Govern** | 41–55 (parallel) | Security review of existing agents; connector/ACP/virtual-connector policy (§7) | Security checklist passes for all four agents |
| **Ship** | 56–70 | Pipelines, stages, approvals, `pac`, source control (§6) | One full pipeline release to prod |
| **Operate** | 71–85 | Monitor, evaluations, drift, cost-per-agent, App Insights for the integration tier (§8) | First health report + evaluation gate in the release |
| **Consolidate** | 86–90 | Retrospective; certification kickoff; backlog of next agents | Signed-off team capability matrix (§10.2) with evidence per cell |

### 12.5 What "done" looks like

At day 90, Cymbal Bank should be able to say: *we have four engineers who each ship a governed, integrated, monitored agent end-to-end without the CoE rescuing them; every release goes through a pipeline with an approval gate; every agent has a named owner, a DLP-reviewed connector footprint, and a monthly health report; and the team has a certification plan mapped to the current Microsoft credential landscape.* That is the syllabus made operational — and it is exactly the bar the eight responsibilities describe.

---

## 13. Claims Audit and What Could Not Be Verified

### 13.1 Claims-audit table

| Claim | Tag | Basis |
|---|---|---|
| Copilot Studio is a low-code studio for agents and workflows; agents draw on knowledge and tools | ✅ | `learn.microsoft.com/microsoft-copilot-studio/fundamentals-what-is-copilot-studio` |
| Three harnesses: GitHub Copilot, standard, Copilot chat | ✅ | Same page |
| Topics route via NLU on the standard harness; generative answers fallback | ✅ | Same page |
| Knowledge-source types (SharePoint, ServiceNow, Dataverse, Azure AI Search, etc.) and knowledge-vs-attachment distinction | ✅ | `.../agents-experience/knowledge-sources-overview` |
| MCP supported (tools + resources); requires generative orchestration | ✅ | `.../agent-extend-action-mcp` |
| Autonomous agent = agent with an event trigger | ✅ | `.../analytics-overview` |
| Workflows have human-in-the-loop controls; agent flows call agents | ✅ | Copilot Studio overview |
| Cloud flows vs. desktop flows (RPA) | ✅ | `learn.microsoft.com/power-automate/desktop-flows/introduction` |
| Connector = API wrapper; connection = stored credential; connection reference = solution component | ✅ | `.../maker/data-platform/create-connection-reference` |
| Custom/certified/virtual/MCP connectors and ACP; design-time vs. runtime; up-to-24h propagation | ✅ | `learn.microsoft.com/power-platform/admin/wp-data-loss-prevention` |
| Managed vs. unmanaged solutions; components; lifecycle; publisher/prefix; dependencies; 95 MB limit | ✅ | `.../alm/solution-concepts-alm` |
| Pipelines: host/stages, managed targets, delegated deployments, `pac pipeline`, sequential tamper-proof artifact, one solution per deployment, single dev environment | ✅ | `.../alm/pipelines` |
| Feb 2026 auto-enablement of managed environments for pipeline targets | ✅ | `.../alm/pipelines` |
| Monitor: 360-day data, 28-day transcripts, DAU/MAU requires authentication, outcomes/themes, hybrid view | ✅ | `.../analytics-overview` |
| Evaluations with test sets and a shared grader library | ✅ | Copilot Studio overview |
| PL-200 retired Aug 31, 2026; PL-600 retired Jun 30, 2026; PL-500 retired | ✅ | Credentials exam/cert pages |
| PL-400 active, transitioning to AB-400; AB-400 from Oct 16, 2026 | ✅ | `.../certifications/power-platform-developer-associate/`, `/exams/pl-400/` |
| AB-100 Agentic AI Business Solutions Architect (associate-cert prerequisite list incl. AB-620, AB-410) | ✅ | `.../certifications/exams/ab-100/` |
| HTTP action, managed identity, Key Vault, `pac solution`, Azure DevOps/GitHub action names | ⚠ | Not re-verified this pass |
| Entra Agent ID / agent identity product surface | ❌ | Could not verify (see below) |
| Copilot Studio → Application Insights export path | ❌ | Could not verify (see below) |
| Copilot Studio Academy track names (Recruit/Operative) | ⚠ | Single third-party source |

### 13.2 What Could Not Be Verified

The following were not confirmed against a primary Microsoft source this pass and must **not** be treated as fact until re-checked:

1. **Copilot Studio → Application Insights.** The specific mechanism for streaming Copilot Studio agent telemetry into Azure Application Insights (and the exact topology for correlating agent traces with downstream API telemetry) was not verified — candidate documentation URLs returned 404 and a web search returned empty. The *generic* Azure Monitor / Application Insights concepts are well-established; the *agent-specific export path* is unverified. ❌
2. **Microsoft Entra Agent ID / agent identity for Copilot Studio agents.** The evolving identity surface for autonomous/proactive agents was not verified this pass. Treat "agent identity" designs as ⚠-knowledge. ❌
3. **Exact Copilot Studio learning-module and learning-path names** (including the Copilot Studio Academy "Recruit"/"Operative" tracks) — a third-party source describes them; Microsoft Learn was not re-checked for the exact strings. ⚠
4. **Exact Azure DevOps / GitHub pipeline task and action names** (Power Platform Build Tools tasks; Power Platform GitHub Actions names) — not re-verified; the `pac` CLI command group `pac pipeline` and `pac solution` are the verified primitives. ⚠
5. **The AB-620 exam page itself** did not resolve at the expected URL this pass; the **Microsoft Certified: AI Agent Builder Associate** credential *is* referenced from the AB-100 page, so the credential name is verified but the exam page/skills-measured were not. ⚠
6. **The exact premium/standard connector classification** for individual connectors (e.g. the HTTP action licence boundary) — not re-verified connector-by-connector. ⚠
7. **Environment-variable-vs-secret detail** — the ALM page URL tested returned 404; the fact that pipelines deploy environment variables is verified from the pipelines page, but the secret-model specifics were not. ⚠
8. **PL-900 / AZ-900 current status** — asserted to remain active by a single third-party source; not re-verified on Learn. ⚠

No claim in this guide should be relied on for a real migration or certification decision without a fresh check on `learn.microsoft.com`, because the platform and the credential portfolio are both mid-transition as of this writing.

---

## 14. Glossary

- **Agent** — an AI assistant that handles conversations and completes tasks using instructions, knowledge sources, and tools. ✅
- **Agent flow** — a Copilot Studio-native flow that can be standalone or attached to an agent as a tool; can run prompts, call agents, and include human review. ✅
- **Agent inventory** — the administrative view of agents across the organization, used to inventory, secure, and manage them. ✅
- **Analytics Viewer / Bot Transcript Viewer** — sharing roles granting view-only Monitor access and transcript access respectively. ✅
- **Autonomous agent** — an agent with an event trigger; runs proactively from a payload. ✅
- **AB family** — the 2026 AI/agent-focused Microsoft credential family (AB-100, AB-400, AB-410, AB-620). ✅/⚠
- **Certified connector** — a connector Microsoft tests for security, reliability, and compliance. ✅
- **Connection** — a saved authentication credential for a connector, stored in the environment. ✅
- **Connection reference** — a solution component holding a reference to a connection so flows/apps don't bind to a personal connection. ✅
- **Copilot Credits** — the consumption currency for usage-based agent billing. ✅
- **CoE Starter Kit** — Microsoft's Center of Excellence toolkit; home of the **ALM Accelerator**. ✅
- **Dataverse** — Microsoft's data platform beneath Power Platform and Dynamics 365. ✅
- **Delegated deployment** — the pipelines approval mechanism for production deployments. ✅
- **DLP / data policy** — guardrails limiting which connectors can be combined or used, across Power Apps, Power Automate, and Copilot Studio. ✅
- **Environment variable** — a solution component carrying per-environment configuration. ✅
- **Generative orchestration** — the mode where the agent plans across topics, tools, and knowledge instead of topic-by-topic NLU routing. ✅
- **Harness** — the engine running an agent/workflow: GitHub Copilot, standard, or Copilot chat. ✅
- **Knowledge source** — designer-curated content shared as grounding for all users. ✅
- **Managed / unmanaged solution** — deployed build artifact vs. development source. ✅
- **Managed environment** — an environment with the governance/analytics/licensing feature set required for pipeline targets. ✅
- **MCP (Model Context Protocol)** — a protocol exposing Resources, Tools, and Prompts to an agent; supported in Copilot Studio with generative orchestration on. ✅
- **Pipelines (Power Platform Pipelines)** — native, in-product ALM deployment automation. ✅
- **Publisher / prefix** — the owner of solution components and the collision-avoiding naming prefix. ✅
- **Topic** — a designed portion of a conversation (triggers, questions, conditions). ✅
- **Tool** — a function an agent can call (connector action, flow, prompt, MCP tool). ✅
- **Virtual connector** — a non-API connector used purely as a governance on/off control; Copilot Studio features are governed this way. ✅

---

## 15. Closing

This syllabus reduces the Copilot Studio agent-engineering role to one loop: **build the agent, connect it to the systems that hold the truth, make it reason and recover, package and ship it through a governed pipeline, secure every hop, and watch it in production.** The eight responsibilities are not eight jobs — they are eight depths of the same job, and the professional marker is that none of them is a stranger. The platform will keep moving (the harnesses, the MCP surfaces, and the entire certification portfolio are all mid-transition as of this pass), but the discipline will not: an agent is only as good as its grounding, its integrations, its fallbacks, and its evidence. Learn in dependency order, prove each area with an artifact, re-verify the facts that move, and keep the eight-area map current. That is the agent engineer's syllabus.
