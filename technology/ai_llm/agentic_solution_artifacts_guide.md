# The Agentic Solution's Artifacts — the artifact trail

> **A deep-dive artifact-inventory guide: every kind of artifact an agentic-AI solution consists of and produces across its lifecycle — the design-time definitions and manifests, the runtime traces, plans, tool calls, state, and deliverables, the quality evals and fixtures, and the governance logs, decision records, and compliance reports — mapped against the platform conventions that name them (OpenAI Agents SDK, Anthropic Claude Agent SDK and AGENTS.md, Google ADK, LangChain/LangSmith/LangGraph, Model Context Protocol), written for teams that must treat an agent's full artifact trail as the unit of auditability, debuggability, and reproducibility.**

> **Byline:** Jack Liu Shurui, Solution Architect  \
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Agent Platform · LLMOps  \
> **Date:** September 2026

> **Companion guides.** This guide is the cross-cutting *artifact taxonomy* of the agent-operations cluster: it inventories the artifact classes and maps where each class is owned in depth by a sibling. Versioned agent artifacts, registries, and release gates live in [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md); scaffolding and agent skeletons in [technology/ai_llm/agent_scaffolding_guide.md](technology/ai_llm/agent_scaffolding_guide.md); tool/MCP artifacts in [technology/ai_llm/mcp_discovery_guide.md](technology/ai_llm/mcp_discovery_guide.md) and [technology/ai_llm/mcp_framework_tools_guide.md](technology/ai_llm/mcp_framework_tools_guide.md); multi-agent design and production gates in [technology/ai_llm/multi_agent_banking_guide.md](technology/ai_llm/multi_agent_banking_guide.md); context, prompt, and memory artifacts in [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md); hierarchical and hybrid multi-agent decomposition in [technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md](technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md) and [technology/ai_llm/hybrid_multi_agent_systems_guide.md](technology/ai_llm/hybrid_multi_agent_systems_guide.md); runtime cache/context-layer design in [technology/ai_llm/agent_runtime_cache_design_guide.md](technology/ai_llm/agent_runtime_cache_design_guide.md); evaluation methodology in [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md) and [technology/ai_llm/llm_evaluation_vs_validation_guide.md](technology/ai_llm/llm_evaluation_vs_validation_guide.md); drift artifacts in [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md); governance and red-teaming in [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md) and [technology/ai_llm/ai_governance_bias_redteaming_guide.md](technology/ai_llm/ai_governance_bias_redteaming_guide.md).

> **How to read it.** Sections 1–2 define why an artifact inventory matters and the four-class taxonomy that organizes the whole guide. Sections 3–4 inventory design-time artifacts; 5–6 runtime artifacts; 7 quality artifacts; 8 governance artifacts. Section 9 condenses the create→version→evaluate→deploy→audit lifecycle. Section 10 is a full worked inventory for a Cymbal Bank agent. Sections 11–14 are the honesty apparatus: claims audit, unverified items, glossary, references. Every platform-specific format name was checked against a primary source this pass; anything that could not be confirmed is flagged **⚠** or **❌** inline and collected in §11–§12. Divergences between platform conventions are flagged with **⚠** at the point where they bite.

---

## Table of Contents

1. [Why an Artifact Inventory Exists](#1-why-an-artifact-inventory-exists)
2. [The Artifact Taxonomy, Four Classes and a Lifecycle](#2-the-artifact-taxonomy-four-classes-and-a-lifecycle)
3. [Design-Time Artifacts I, Agent Definitions and Manifests](#3-design-time-artifacts-i-agent-definitions-and-manifests)
4. [Design-Time Artifacts II, Prompts, Workflows, Tools, and Models](#4-design-time-artifacts-ii-prompts-workflows-tools-and-models)
5. [Runtime Artifacts I, Traces and Spans](#5-runtime-artifacts-i-traces-and-spans)
6. [Runtime Artifacts II, Plans, Tool Calls, State, and Deliverables](#6-runtime-artifacts-ii-plans-tool-calls-state-and-deliverables)
7. [Quality Artifacts, Eval Suites, Datasets, and Guardrails](#7-quality-artifacts-eval-suites-datasets-and-guardrails)
8. [Governance Artifacts, Audit Logs, Decisions, Incidents, and Compliance](#8-governance-artifacts-audit-logs-decisions-incidents-and-compliance)
9. [The Artifact Lifecycle, Condensed](#9-the-artifact-lifecycle-condensed)
10. [Worked Example, The Cymbal Bank Trade-Services Agent](#10-worked-example-the-cymbal-bank-trade-services-agent)
11. [Claims Audit](#11-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Why an Artifact Inventory Exists

### 1.1 An Agent Is Not a Program; It Is a Trail of Artifacts

A conventional service is auditable because its behavior is a deterministic function of code plus configuration, and both live in a versioned repository. An agentic solution is a stochastic, tool-calling, model-driven system whose behavior is a function of artifacts that *change independently of code*: the system prompt, tool schemas, the model and its sampling parameters, the graph or workflow that routes control, the knowledge it retrieves, and the memory it carries between turns. When something goes wrong — a bad payment instruction, a hallucinated clause, a runaway tool loop — the question "what did this agent actually do, and why?" cannot be answered by reading source code. It can only be answered by reading the **artifact trail**: the ordered, linked set of artifacts the solution both *consists of* (its definitions) and *produced* (its runs).

The artifact trail is what makes three engineering properties possible at all:

- **Auditability.** Every consequential action must resolve to a record: which agent version ran, which instructions it held, which model it called, which tool arguments it sent, which state it changed, and who approved it. Without the trail there is no answer to a regulator, an internal-audit finding, or a customer dispute — there is only a model's memory, which is to say, nothing.
- **Debuggability.** The OpenAI Agents SDK documentation states plainly that its built-in tracing collects "a comprehensive record of events during an agent run: LLM generations, tool calls, handoffs, guardrails, and even custom events" ✅ (OpenAI Agents SDK docs, Tracing). That record is the debugger's view of an agent: a run is a trace, a trace is composed of spans, and each span names the artifact type involved (generation, function, guardrail, handoff). LangSmith's evaluation documentation makes the same point as a workflow: "Add real traces to a dataset so a failure you saw once becomes a test you run every time" ✅ (LangSmith docs, Evaluation). A failure is debuggable precisely because the trace is a first-class artifact that can be replayed into the quality pipeline.
- **Reproducibility.** Google's ADK documentation frames its context machinery in exactly these terms: ADK "treat[s] context like source code — sessions, memory, tool outputs, and artifacts are assembled into a structured view," filtering irrelevant events, summarizing older turns, and lazy-loading artifacts ✅ (Google ADK docs, FAQ). The ambition of an artifact inventory is to give the agent's *context* — everything that influenced a decision — the same status as source code: named, structured, storable, and reviewable.

### 1.2 The Vocabulary Problem, and Why This Guide Exists

Here is the trap the field has built for itself: every platform that ships an agent framework invented its own name for roughly the same set of objects, and a few platforms use the same word for different objects. Google ADK calls named, versioned, binary outputs **Artifacts** ✅ (ADK docs, Artifacts); this guide uses "artifact" for *everything an agentic solution consists of and produces*, so ADK's Artifacts are one species of runtime output within a much larger inventory. OpenAI's SDK describes an agent as "an LLM configured with instructions, tools, and optional runtime behavior" ✅ (OpenAI Agents SDK docs, Agents), while Anthropic's "Building effective agents" essay reserves the word "agent" for systems where "LLMs dynamically direct their own processes and tool usage," and calls code-orchestrated LLM pipelines **workflows** ✅ (Anthropic engineering blog, Building Effective Agents). AGENTS.md is a plain-Markdown file that travels with a repository ✅ (agents.md), while ADK's Agent Config is a YAML manifest with a JSON Schema ✅ (ADK docs, Agent Config), and the OpenAI SDK's agent is a code object with no canonical on-disk form at all. A team that tries to bolt these conventions together without a shared artifact taxonomy ends up with an inventory that cannot be compared, versioned, or audited coherently.

This guide is that shared taxonomy. It maps four artifact classes across the lifecycle — design, runtime, quality, governance — names the concrete artifacts in each class, and, wherever platform conventions diverge, flags both conventions with **⚠** and names the primary source for each. It deliberately does not re-derive the deep content of any single class: each class is owned by a sibling guide, cross-referenced in one or two lines at the point of use.

### 1.3 Artifact Properties That Make the Trail Walkable

An inventory of artifacts only earns the word "trail" when its members share four properties — and every later section of this guide returns to one of them:

| Property | Meaning | Where it is grounded in this guide |
|---|---|---|
| **Identity** | Every artifact has a stable, unique ID recorded at creation: `trace_id`, `thread_id`, session ID, dataset version, experiment ID, ADR number | §5 (runs), §6.3 (state), §7.1 (quality), §8.2 (decisions) |
| **Linkage** | Artifacts carry pointers to their neighbors: a tool record points to its trace, a trace points to its definition version, a verdict points to its dataset version | §5.2, §9.2 |
| **Mutability discipline** | Some artifacts may change (definitions under review); others must not (a trace after export, an audit entry, a released version). Platforms encode this: LangSmith dataset versions are immutable views tagged `"prod"` ✅; ADK warns that state edits outside the event flow lose auditability ✅ | §7.1, §6.3, §8.1 |
| **Retention clarity** | Every artifact class has a stated retention horizon, because privacy-bearing traces, customer-derived dataset examples, and governance records cannot share one policy | §9.3 |

Two further distinctions shape every table below. **Content vs. reference:** a prompt may live in the vendor console (OpenAI platform prompt templates) while the repo holds only a reference ✅ (Agents docs) — the inventory must record *where the truth lives*. **Record vs. replay:** some artifacts are immutable records (an audit log), others are replayable state (LangGraph checkpoints enable time travel ✅) — both are needed, and they are different things (§8.3).

## 2. The Artifact Taxonomy, Four Classes and a Lifecycle

### 2.1 The Four Classes

Every artifact an agentic solution consists of or produces falls into one of four classes, distinguished by *when it exists relative to a run* and *what question it answers*:

| Class | Exists | Answers | Representative artifacts (detailed in this guide) |
|---|---|---|---|
| **Design-time (definition)** | Before any run; changes on a release cadence | What is this agent supposed to be? | Agent definitions/manifests, system prompts & instructions, workflows/graphs, tool schemas & MCP server configs, model & parameter configs |
| **Runtime (execution)** | During and after a run | What did this agent do, decide, and produce? | Traces & spans, plans & intermediate reasoning, tool-call records, state & memory, outputs & deliverables |
| **Quality (evaluation)** | In the eval loop around runs | How do we know it is good, and still good? | Eval suites, datasets/examples, fixtures & golden sets, regression baselines, guardrail configs, experiment results |
| **Governance (accountability)** | Permanently, as the record of all of the above | Who decided, what happened, who is answerable? | Audit logs, decision records (ADR-style), incident & rollback records, compliance reports |

The classes are not disjoint in storage — a trace (runtime) becomes the seed of a dataset (quality) when a failure is exported back into the eval suite, and an audit log (governance) is fed by tool-call records (runtime). What distinguishes the classes is the *question they answer* and the *lifecycle stage that owns them*, not the database they sit in. LangSmith's documentation formalizes one boundary explicitly: offline evaluation runs against curated datasets with reference outputs ("test before you ship"), while online evaluation runs against live production runs and threads ("monitor in production") ✅ (LangSmith docs, Evaluation). That is the runtime/quality seam drawn in a vendor's own terms.

### 2.2 The Lifecycle That Ties Them Together

The four classes sit on a five-stage lifecycle — **create → version → evaluate → deploy → audit** — which is the skeleton of this guide's §9 and the subject of the versioning sibling in full:

| Stage | Consumes | Produces |
|---|---|---|
| Create | Requirements, platform conventions | Agent definitions, prompts, workflows, tool/MCP configs, model configs (§3–§4) |
| Version | Definitions as they change | Versioned snapshots, manifests in registries, release candidates |
| Evaluate | Candidates + quality artifacts | Experiment results, regression baselines, release verdicts (§7) |
| Deploy | Approved candidate | Runtime artifacts: traces, plans, tool records, state, outputs (§5–§6) |
| Audit | The whole trail, continuously | Governance records: logs, decisions, incidents, compliance evidence (§8) |

Versioning, evaluation, deployment gates, and drift monitoring are each owned by siblings — [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) (registries, prompt-as-code, release gates, rollback), [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md) and [technology/ai_llm/llm_evaluation_vs_validation_guide.md](technology/ai_llm/llm_evaluation_vs_validation_guide.md) (metrics and eval methodology), [technology/ai_llm/multi_agent_banking_guide.md](technology/ai_llm/multi_agent_banking_guide.md) (production gates in a banking context), and [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md) (post-deploy behavioral drift) — and are only condensed here.

### 2.3 Divergences That Shape Everything Below

Before the inventory proper, four naming/format divergences deserve an upfront **⚠**, because they will resurface in every section:

| Divergence | Convention A | Convention B | Implication |
|---|---|---|---|
| Where the agent definition lives | **AGENTS.md** — a plain-Markdown file (no required fields, no schema) that travels with the repo and is read by coding agents; "the closest AGENTS.md to the edited file wins" ✅ (agents.md; closest-wins rule) | **ADK Agent Config** — `root_agent.yaml` with a published JSON Schema and typed fields (`name`, `model`, `instruction`, `tools`, `sub_agents`), loaded by `adk` CLI ✅ (ADK docs, Agent Config) | "The manifest" is not one format; a multi-platform estate needs a registry that can hold both Markdown instructions and schema'd YAML — see §3 |
| What a "trace" is | OpenAI SDK: a trace with `workflow_name`, `trace_id`, `group_id`; spans for agent/generation/function/guardrail/handoff ✅ (OpenAI Agents SDK docs, Tracing) | OpenTelemetry GenAI semantic conventions, now developed in a dedicated repository with span/event/metric families per provider (OpenAI, Anthropic, Bedrock, Azure) plus agent-span and MCP conventions ✅ family-level (opentelemetry.io semconv index; open-telemetry/semantic-conventions-genai repo) — ⚠ attribute-level detail unverified (§12) | Trace schemas are not interchangeable; lineage joins must map span families across vendors (§5) |
| What an "artifact" is | ADK Artifacts: named, versioned binary data (`save_artifact`/`load_artifact`) scoped to a session or user ✅ (ADK docs, Artifacts) — ⚠ narrower meaning than this guide's | This guide / common usage: artifact = any named object in the trail, text or binary | Reading ADK docs, remember its "Artifacts" are one runtime-output species, not the whole inventory |
| What an eval dataset is | LangSmith: versioned datasets of examples (inputs + optional reference outputs + metadata), evaluators, experiments ✅ (LangSmith docs) | OpenAI platform: an Evals section with getting-started, working-with-evals, graders, and best-practices guides ✅ nav-level (platform.openai.com docs nav) — ⚠ dataset format unverified (§12) | Golden sets are portable in spirit but not in file format; treat each platform's dataset as its own artifact type (§7) |

The rest of the guide fills in the detail behind each row, with the platform divergence flagged again where it changes what you store.

### 2.4 Where Each Class Lives: Storage and Ownership

The classes map onto different stores with different owners, retention profiles, and access rules — and that mapping is itself part of the design. The table below is this guide's default target architecture, with the vendor mechanisms verified above in parentheses:

| Class | Canonical stores | Typical owner | Retention posture |
|---|---|---|---|
| Design-time | Git repo (AGENTS.md, configs, graph code); platform consoles (OpenAI prompt templates ✅); registry (versioning sibling) | Engineering + prompt/context owners | Permanent, versioned |
| Runtime | Trace store (OpenAI Traces dashboard/custom processors ✅; OTel pipeline ⚠); session/state store (ADK `SessionService` ✅; LangGraph checkpointer with `thread_id` ✅); artifact store (ADK `ArtifactService`/GCS ✅) | Platform/LLMOps | Bounded by privacy/regulatory horizon (§9.3) |
| Quality | Eval platform (LangSmith datasets/evaluators/experiments ✅; Anthropic held-out golden sets ✅); dataset versions tagged `"prod"` ✅ | Quality/ML engineers | Long, as the corporate memory of "good" |
| Governance | Audit log store (tool-usage logs per MCP client obligation ✅; ADK event log ✅); decision/incident registers (ADR convention, §8.2); compliance report store (NIST-mapped evidence ✅) | Risk/compliance + platform | Longest; append-only |

Two ownership rules follow from the verified material. First, **runtime stores are chosen, not accidental**: ADK's docs are explicit that in-memory session services lose everything on restart, and that persistence is a property of the `SessionService` implementation ✅ (ADK docs, State) — the same is true of LangGraph checkpointers, which exist only when a graph is compiled with one ✅ (Checkpointers docs). An agent whose state store is an in-memory default has, by construction, no audit history. Second, **the governance layer must not depend on vendor retention**: OpenAI's tracing is unavailable under Zero Data Retention policies ✅ (Tracing docs), so the audit record must be exportable by design (custom trace processors are the documented escape hatch ✅), not assumed to exist in the vendor's cloud.

## 3. Design-Time Artifacts I, Agent Definitions and Manifests

Design-time artifacts are the *specification layer*: they exist before any run, change on a release cadence rather than a per-turn one, and are the natural unit of versioning. The versioning sibling ([technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md)) treats an agent version as a tuple over five artifact families — instructions, tools, model, parameters, knowledge — and this section and §4 inventory exactly those families as concrete artifacts, platform by platform. Scaffolding conventions (directory layout, skeleton files, boilerplate) are owned by [technology/ai_llm/agent_scaffolding_guide.md](technology/ai_llm/agent_scaffolding_guide.md) and are not repeated here.

### 3.1 Agent Definitions: Four Platforms, Four Conventions

There is no single "agent manifest" format across the industry. What exists is a spectrum from *free-form Markdown instructions that travel with code* to *schema'd YAML loaded by a runtime* to *code objects with no canonical file form*. An artifact inventory must name the convention in use, because versioning, diffing, and auditing an agent definition all depend on its format.

**AGENTS.md — the portable instructions file.** AGENTS.md is an open format "for guiding coding agents," positioned as "a README for agents": a predictable place in the repository for the context and instructions an agent needs ✅ (agents.md). It emerged from collaboration across OpenAI Codex, Amp, Google Jules, Cursor, and Factory, and is now stewarded by the Agentic AI Foundation under the Linux Foundation ✅ (agents.md, About). Verified properties of the format:

- It is standard Markdown with **no required fields** — "the agent simply parses the text you provide" ✅ (agents.md, FAQ).
- Files may be nested: "agents automatically read the nearest file in the directory tree, so the closest one takes precedence" — the OpenAI main repository alone had 88 AGENTS.md files at the time of writing ✅ (agents.md).
- Conflict resolution is positional: "the closest AGENTS.md to the edited file wins; explicit user chat prompts override everything" ✅ (agents.md, FAQ).
- Agents are expected to *execute* the programmatic checks the file lists ("The agent will attempt to execute relevant programmatic checks and fix failures before finishing the task") — which makes an AGENTS.md file a design-time artifact with runtime obligations ✅ (agents.md, FAQ).
- Common sections named by the spec: project overview, build and test commands, code style guidelines, testing instructions, security considerations ✅ (agents.md).
- Clients configure it differently per tool: Aider via `.aider.conf.yml` (`read: AGENTS.md`), Gemini CLI via `.gemini/settings.json` (`"context": { "fileName": "AGENTS.md" }`) ✅ (agents.md, FAQ).

⚠ **Divergence:** AGENTS.md is an *instruction file for coding agents in a repository*, not a deployment manifest for a production conversational agent. It coexists with — and must not be confused with — a platform's runtime agent definition. Claude Code loads project configuration from `.claude/` directories, and the Claude Agent SDK "load[s] automatically from your project's `.claude/` and from `~/.claude/`" for skills, commands, and memory ✅ (platform.claude.com, Agent SDK overview). A repo can legitimately carry both an AGENTS.md (for coding agents working on the codebase) and a runtime agent definition (for the production agent the codebase ships).

**OpenAI Agents SDK — the code-defined agent.** In the OpenAI SDK an agent is a first-class code object: "an LLM configured with instructions, tools, and optional runtime behavior such as handoffs, guardrails, and structured outputs" ✅ (OpenAI Agents SDK docs, Agents). The documented `Agent` properties, which form the de-facto agent-definition schema when serialized for review, are:

| Property | Role (verified ✅, OpenAI Agents SDK docs, Agents) |
|---|---|
| `name` | Required, human-readable agent name |
| `instructions` | System prompt, or a dynamic-instructions callback; "strongly recommended" |
| `prompt` | OpenAI platform prompt-template reference (Responses API) |
| `model`, `model_settings` | Model selection; tuning parameters such as `temperature`, `top_p`, `tool_choice` |
| `tools` | Tools the agent can call |
| `mcp_servers`, `mcp_config` | MCP servers providing MCP-backed tools; per-agent preparation config |
| `handoffs`, `handoff_description` | Delegation targets; description shown when offered as a handoff |
| `input_guardrails`, `output_guardrails` | Guardrails on first user input / final output |
| `output_type` | Structured output type |
| `hooks` | Agent-scoped lifecycle callbacks |
| `tool_use_behavior`, `reset_tool_choice` | Tool-result looping and tool-choice reset to avoid infinite loops |

The SDK also distinguishes a `SandboxAgent` variant "with manifest-defined files and sandbox-native capabilities," adding `default_manifest`, `base_instructions`, `capabilities`, and `run_as` ✅ (OpenAI Agents SDK docs, Agents). Note the vocabulary: "manifest" here means a *sandbox file manifest*, not the agent definition itself. ⚠ There is no canonical on-disk agent-definition format in the OpenAI SDK — definitions are code; teams that want file-based definitions must impose their own serialization (the versioning sibling's manifest/bundle pattern is the usual answer).

**Anthropic — Claude Agent SDK and the agent loop.** Anthropic's "Building effective agents" essay draws the architecture-level line between **workflows** ("systems where LLMs and tools are orchestrated through predefined code paths") and **agents** ("systems where LLMs dynamically direct their own processes and tool usage") ✅ (Anthropic, Building Effective Agents, Dec 2024). The Claude Agent SDK is Anthropic's framework implementation: an application "that completes a task by planning its own steps and calling tools," with the SDK providing "the same tools, agent loop, and context management that power Claude Code, programmable in Python and TypeScript" ✅ (platform.claude.com, Agent SDK overview). Verified SDK capabilities that read as definition artifacts: built-in tools (read/write/edit files, run commands, web search), hooks ("run custom code at key points in the agent lifecycle"), subagents ("spawn specialized agents for focused subtasks"), MCP connectivity, permissions ("control which tools run automatically, which need approval"), sessions ("maintain context across exchanges, resume or fork later"), and plugins that "package skills, agents, hooks, and MCP servers" ✅ (Agent SDK overview, capabilities table). ⚠ The Agent SDK docs live under "Claude Code Docs" and inherit Claude Code's `.claude/`-based configuration conventions — a different artifact layout from AGENTS.md-rooted repos or SDK-code definitions.

**Google ADK — code objects, plus an experimental YAML manifest.** ADK defines agents in code across five languages (Python, TypeScript, Go, Java, Kotlin): `Agent(name=..., model=..., instruction=..., tools=...)` ✅ (adk.dev landing; ADK docs, Agent Config). On top of that, ADK's **Agent Config** feature provides a YAML-based manifest "to build an ADK workflow without writing code": `adk create --type=config my_agent` generates a project with `root_agent.yaml` and `.env`, and the YAML is schema-annotated (`$schema: .../config_schemas/AgentConfig.json`) ✅ (ADK docs, Agent Config). Verified config keys: `name`, `model`, `description`, `instruction`, `tools` (each `name:` referencing a tool), and `sub_agents` with `config_path` references to further YAML files ✅ (ADK docs, Agent Config, examples). Running options are `adk web`, `adk run`, and `adk api_server`, and code can load the same file via `config_agent_utils.from_config(".../root_agent.yaml")` ✅. ⚠ Agent Config is explicitly experimental with known limitations: **Gemini models only**, Python/Java-only for tools, and unsupported agent types (`LangGraphAgent`, `A2aAgent`) ✅ (ADK docs, Agent Config, known limitations) — a warning against standardizing the whole estate on it today.

**Synthesis for the inventory:** an enterprise artifact inventory should record, per agent, *which* definition convention it uses (repo Markdown, platform schema'd YAML, code object, sandbox manifest), the version of that definition, and its platform binding — because the diffable, gateable unit is different in each case. The registry mechanics (hashing, promotion, rollback) are in the versioning sibling.

### 3.2 Manifests, Bundles, and the "Definition Artifact" Abstraction

Cross-platform practice converges on wrapping whatever the platform defines into a **manifest/bundle** of your own: a file that pins the definition artifact together with its model, tools, and knowledge versions — the "agent-config-as-code" pattern. The versioning sibling ([technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md)) documents manifest formats and registries; [technology/ai_llm/agent_scaffolding_guide.md](technology/ai_llm/agent_scaffolding_guide.md) documents the skeleton those manifests hang on; and [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md) documents how context bundles (skills, AGENTS.md content, few-shot material) compose into the definition. What this guide adds is only the taxonomy point: **the manifest is a governance artifact as much as a design artifact** — it is the immutable record of "what this agent was," which is exactly what an audit (§8) or a rollback (§9) needs to read.

### 3.3 Platform Definition Conventions at a Glance

The section-level comparison table that an architect actually needs when choosing (or reconciling) platforms:

| Dimension | AGENTS.md | OpenAI Agents SDK | Claude Agent SDK | Google ADK (code) | ADK Agent Config |
|---|---|---|---|---|---|
| Carrier | Repo Markdown file(s) ✅ (agents.md) | Code objects (`Agent`) ✅ | Code (Python/TypeScript) + `.claude/` config ✅ | Code objects (`Agent`/`LlmAgent`) ✅ | YAML `root_agent.yaml` + JSON Schema ✅ |
| Required fields | None ✅ | `name` ✅ | n/a (SDK program) | `name`, `model`, `instruction` in examples ✅ | `name`, `model`, `instruction` in examples ✅ |
| Schema | None (free Markdown) ✅ | Python type system | SDK types | SDK types | Published `AgentConfig.json` `$schema` ✅ |
| Instructions | Markdown sections (build/test/style/security) ✅ | `instructions` or dynamic callback; platform `prompt` templates ✅ | System prompt managed by SDK/Claude Code features ✅ | `instruction` string, `{key}` state templating ✅ | `instruction:` YAML key ✅ |
| Tools | N/A (agent may use repo tools) | `tools`, `mcp_servers` ✅ | Built-in tools, MCP, subagents ✅ | `tools=[...]` ✅ | `tools: - name:` ✅ |
| Multi-agent | Nested files per directory ✅ | `handoffs` ✅ | `subagents` ✅ | Sub-agents, teams ✅ | `sub_agents: - config_path:` ✅ |
| Guardrails | N/A | `input_guardrails`/`output_guardrails` ✅ | Permissions system ✅ | Plugins (e.g., ATR guardrail) ✅ | Via config tools ✅ |
| Status | Stewarded by Agentic AI Foundation (Linux Foundation) ✅ | GA framework | GA SDK (docs under Claude Code) ✅ | GA framework | Experimental — Gemini-only limitation ✅ (ADK docs) |
| Diffable artifact? | Yes — plain text in git ✅ | Only if team serializes definitions | Only via SDK code + config files | Only via code + optional YAML | Yes — YAML is naturally diffable ✅ |

⚠ **Reading the row that matters most:** only AGENTS.md and ADK Agent Config give you a *file you can diff in a pull request*. The code-object platforms are versioned through their source code and whatever manifest your registry imposes — which is exactly why the versioning sibling ([technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md)) treats "agent-config-as-code" as a discipline to build, not a feature platforms give you.

### 3.4 AGENTS.md in Depth: Spec Details That Matter for the Inventory

Beyond the basics in §3.1, four AGENTS.md facts shape how an estate inventories instruction files:

- **One file, many agents.** The format's value proposition is that "your agent definitions are compatible with a growing ecosystem of AI coding agents and tools"; the maintainers list adopters including GitHub Copilot (coding agent), OpenAI Codex, Cursor, Google's Jules and Gemini CLI, Windsurf, Aider, Zed, and others ✅ (agents.md). An AGENTS.md is therefore an *ecosystem-portable* instruction artifact — rare among the formats in this guide.
- **Monorepo scaling is by nesting.** "Place another AGENTS.md inside each package. Agents automatically read the nearest file in the directory tree, so the closest one takes precedence"; the OpenAI repo's 88 files are the cited example ✅ (agents.md). Inventory implication: an estate with nested AGENTS.md files must record *which file applied to which path*, since precedence is positional.
- **It is a living document.** The FAQ says to "treat AGENTS.md as living documentation," and migration guidance shows old files renamed with symlinks for backward compatibility (`mv AGENT.md AGENTS.md && ln -s AGENTS.md AGENT.md`) ✅ (agents.md) — a rare, documented *migration artifact* pattern.
- **Content is conventional, not enforced.** The spec names popular sections (project overview, build/test commands, code style, testing instructions, security considerations) but "there are no required fields" ✅ (agents.md). Inventoried content quality therefore cannot be assumed from file presence — governance must review content (the scaffolding and context-engineering siblings cover review practice).

⚠ The AGENTS.md site is the format's home, but agent-support compliance varies by tool version; treat the adopter list as directional, and verify a specific client's behavior against that client's own docs before relying on it in production.

## 4. Design-Time Artifacts II, Prompts, Workflows, Tools, and Models

Section 3 covered the agent definition itself; this section inventories the artifacts the definition *binds together*: the prompt layer, the control layer (workflows and graphs), the action layer (tools and MCP servers), and the model layer (model and parameter configuration).

### 4.1 System Prompts, Instructions, and Prompt Artifacts

The system prompt is the most volatile artifact in the whole inventory — it changes more often than code, is edited by people who are not engineers, and is frequently stored outside the repo. Deep prompt engineering, versioning, and registry mechanics are owned by [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md) and [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md); the taxonomy points are:

- **Prompt-as-code vs. prompt-as-content.** Every platform supports instructions as first-class inputs: OpenAI's `Agent.instructions` ("System prompt or dynamic instructions callback") ✅ (OpenAI Agents SDK docs, Agents); ADK's `instruction` key in both code and `root_agent.yaml` ✅ (ADK docs); AGENTS.md's free-form Markdown ✅ (agents.md). Whatever the carrier, the inventory should record the prompt as a *versioned content artifact*, not as an inline string nobody diffs.
- **Platform-managed prompt templates.** OpenAI exposes platform prompt templates that an SDK agent references via `prompt` (a "static prompt object or a function"), authored in the platform playground with variables such as `{{poem_style}}` ✅ (OpenAI Agents SDK docs, Agents, prompt templates). This is a genuinely different artifact lifecycle — the prompt lives in the vendor console, and the code holds a reference — with implications for portability and auditability that the registry pattern must absorb.
- **Dynamic and state-templated instructions.** Instructions can be computed at run time: OpenAI supports "dynamic instructions" callbacks ✅; ADK's `LlmAgent` injects session-state values into the instruction string via `{key}` templating before the LLM sees it ✅ (ADK docs, State). Such templates are design-time artifacts whose *expansion* is a runtime event — a distinction worth keeping when a regulator asks what instruction text a given run actually used.
- **Structured instruction packaging.** Anthropic's Agent Skills and Claude Code skills/commands are packaged instruction artifacts loaded from `.claude/` and `~/.claude/` ✅ (Agent SDK overview); AGENTS.md plays the same role at repo scope ✅ (agents.md). [technology/ai_llm/agent_scaffolding_guide.md](technology/ai_llm/agent_scaffolding_guide.md) and [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md) own the format details. ⚠ The Anthropic Agent Skills *file specification* (frontmatter and body conventions) was not re-verified at the primary source this pass — see §12.

The spec's own illustration of what such a file contains (trimmed from the agents.md examples — headings and instruction bullets are free-form, which is the point ✅):

```markdown
# AGENTS.md

Dev environment tips
- Use `pnpm install --filter <project_name>` to add the package to your workspace.
- Check the name field inside each package's package.json to confirm the right name.

Testing instructions
- Run `pnpm turbo run test --filter <project_name>` to run every check defined for that package.
- Add or update tests for the code you change, even if nobody asked.

PR instructions
- Title format: [<project_name>] <Title>
- Always run `pnpm lint` and `pnpm test` before committing.
```

Note what is *not* in the format: no schema, no required frontmatter, no tool declarations — just Markdown that an agent parses ✅ (agents.md). That is simultaneously its portability strength and its governance weakness: nothing machine-enforces the content, so review discipline (which agent files exist, what they promise, whether the commands they cite still work) must come from the estate, not the format.

### 4.2 Workflows and Graphs: The Control-Layer Artifacts

Where instructions say *what* the agent is, the workflow/graph says *how control flows*. Anthropic's taxonomy is the cleanest statement: workflows are "systems where LLMs and tools are orchestrated through predefined code paths," and its essay names five canonical patterns — prompt chaining, routing, parallelization, orchestrator-workers, and evaluator-optimizer — before agents proper ✅ (Anthropic, Building Effective Agents). Platform artifacts that realize this layer:

- **LangGraph graph definitions.** A LangGraph agent workflow is defined as a graph with three components — `State` ("a shared data structure that represents the current snapshot of your application"), `Nodes` ("functions that encode the logic"), and `Edges` ("functions that determine which Node to execute next") — built with `StateGraph`, then **compiled** via `.compile()` before use, with the state schema typically a `TypedDict`, dataclass, or Pydantic model ✅ (LangGraph docs, Graph API overview). Graph execution proceeds in discrete "super-steps," inspired by Google's Pregel system ✅ (LangGraph docs). Definition, compilation, and execution are three distinct artifact moments: the *graph source* is design-time; the *compiled graph* is a deployable artifact; the per-run execution state is runtime (see §6, checkpointers).
- **OpenAI handoffs and orchestration.** Multi-agent control in the OpenAI SDK is expressed through `handoffs` and `handoff_description` on the agent definition ✅ (OpenAI Agents SDK docs, Agents) and is recorded at runtime as `handoff_span`s ✅ (Tracing). The orchestration layer's own sibling is [technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md](technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md) and [technology/ai_llm/hybrid_multi_agent_systems_guide.md](technology/ai_llm/hybrid_multi_agent_systems_guide.md).
- **ADK workflows and graphs.** ADK offers template agent workflows (loop, parallel, sequential), graph-based agent workflows ("weave deterministic code with adaptive AI reasoning" — new in ADK 2.0), and routing agents ✅ (adk.dev landing and docs index). Multi-agent system design in depth lives in [technology/ai_llm/multi_agent_banking_guide.md](technology/ai_llm/multi_agent_banking_guide.md).

⚠ **Divergence:** workflow definitions are *not portable* — LangGraph graphs are code, ADK workflow templates are framework objects, and Anthropic's patterns are architectural prose. The inventory's job is to record, per solution, which pattern family and which framework realizes it, so that a control-flow change is versioned like any other definition change.

### 4.3 Tool Definitions and MCP Servers: The Action-Layer Artifacts

Tool schemas are the agent's interface to the world — Anthropic's essay argues teams should invest as much in the *agent-computer interface* (ACI) as in human UI, testing how the model uses tools and "poka-yoke"-ing arguments so mistakes are harder to make ✅ (Anthropic, Building Effective Agents). The artifact forms:

- **MCP tool definitions.** Under the Model Context Protocol (2025-06-18 spec revision verified this pass), a tool is "uniquely identified by a name and includes metadata describing its schema," with fields `name`, `title` (optional display name), `description`, `inputSchema` (JSON Schema), optional `outputSchema`, and optional `annotations` ✅ (MCP spec, server/tools). Tools are discovered via the `tools/list` request and invoked via `tools/call`; results carry `content` items (`text`, `image`, `audio`, `resource_link`, embedded `resource`) plus optional `structuredContent`, and execution failures are reported in-band with `isError: true`, while protocol errors (unknown tool, invalid arguments) use standard JSON-RPC error codes such as `-32602` ✅ (MCP spec, server/tools). The MCP tool-ecosystem and governance treatment is owned by [technology/ai_llm/mcp_discovery_guide.md](technology/ai_llm/mcp_discovery_guide.md) and [technology/ai_llm/mcp_framework_tools_guide.md](technology/ai_llm/mcp_framework_tools_guide.md) — including registry, discovery, and tool-catalog mechanics — and is not re-derived here.
- **MCP server configs (client side).** A client-side MCP connection is a configuration artifact. The canonical example is Claude Desktop's `claude_desktop_config.json` (at `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows), whose `mcpServers` map binds a friendly server name to `command`, `args`, and `env` ✅ (MCP docs, Connect to local MCP servers). The SDK equivalents are the OpenAI `mcp_servers`/`mcp_config` agent properties ✅ (OpenAI Agents SDK docs, Agents) and ADK's `McpToolset` (supported in Agent Configs) ✅ (ADK docs, Agent Config). ⚠ Spec-versions matter: MCP specs are dated (2025-03-26, 2025-06-18, 2025-11-25, 2026-07-28 exist in the docs archive ✅ (modelcontextprotocol.io llms.txt)) — an MCP server config artifact should record which spec era its client and server negotiate.
- **Direct tool schemas.** Where tools are not MCP-mediated, they are declared in framework form: OpenAI SDK `tools` on the agent (function tools with docstring-derived schemas) ✅ (Agents docs); Anthropic's tool-use docs (define-tools, handle-tool-calls, parallel tool use, strict tool use pages all present in the docs index ✅ (platform.claude.com llms.txt)). Each declaration is a schema artifact with its own version — the exact thing the versioning sibling versions as "tools."
- **Security-relevant tool metadata.** MCP tool definitions may carry `annotations` describing behavior, and the spec warns clients "MUST consider tool annotations to be untrusted unless they come from trusted servers" ✅ (MCP spec, server/tools). The security treatment — including MCP's stated client obligations (validate tool results, timeouts, log tool usage for audit) ✅ (MCP spec, server/tools, Security Considerations) — is picked up in §8 as an audit artifact, and threat-modeled in the banking/multi-agent siblings.

### 4.4 Model and Parameter Configuration

The model layer is the artifact family where the vendor, not the team, owns the artifact: teams pin or alias, providers retire. Taxonomy points, with deep coverage in [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) and [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md):

- **Model selection** is a definition property on every platform verified above: OpenAI `Agent.model` ✅; ADK `model: gemini-flash-latest` in code and `root_agent.yaml` ✅; Claude agents negotiate models through the SDK/API. ADK additionally documents a large model-adapter surface (Gemini, Claude, OpenAI, Ollama, vLLM, LiteLLM, and more, per the docs index ✅ (adk.dev llms.txt)) and model routing ✅.
- **Sampling and behavior parameters** ride on the definition: OpenAI's `model_settings` covers "model tuning parameters such as `temperature`, `top_p`, and `tool_choice`" ✅ (Agents docs); tool-choice reset behavior is itself configurable (`reset_tool_choice`) ✅. These parameter values belong *in the versioned snapshot* because two runs with identical prompts but different temperatures are different behaviors.
- **Context-window and reasoning configuration** is increasingly part of the definition: Anthropic's docs index carries dedicated pages for context windows, prompt caching, extended thinking (legacy), and preserved thinking ✅ (platform.claude.com llms.txt) — evidence that reasoning-effort and context policy are now first-class configuration surfaces. ⚠ Their exact parameter names were not re-verified this pass (see §12); the runtime-cache/context design is owned by [technology/ai_llm/agent_runtime_cache_design_guide.md](technology/ai_llm/agent_runtime_cache_design_guide.md).

### 4.5 Design-Time Review Checklist

A condensed checklist to run before a definition change is ever versioned — each item maps to a verified artifact above and a sibling owner:

1. **Definition carrier recorded?** Which convention holds the agent (AGENTS.md file, SDK code, `root_agent.yaml`, `.claude/` config) and where does it live (§3.3)?
2. **Prompt version pinned?** System prompt as a versioned content artifact — not an inline string; platform-template references recorded with the console as source of truth (§4.1, versioning sibling).
3. **Instruction expansion understood?** Which `{key}`/dynamic templates will expand at run time, and is the *expanded* text recoverable for audit (§4.1, §6)?
4. **Control flow named?** Workflow pattern family (Anthropic taxonomy) and framework artifact (LangGraph graph compiled from `StateGraph`; handoffs; ADK workflow type) recorded (§4.2).
5. **Tool layer governed?** Every tool schema (MCP `inputSchema`/`outputSchema`/`annotations`) and MCP server config (`mcpServers` binding) in a catalog; spec era noted (§4.3, MCP siblings).
6. **Model and parameters captured?** Model/alias, `model_settings` (`temperature`, `top_p`, `tool_choice`) inside the versioned snapshot, not assumed (§4.4).
7. **Eval hook defined?** Which dataset version, evaluators, and gate this definition must pass (§7, eval siblings).
8. **Definition immutable at release?** Released snapshot written to the registry so a trace can later resolve "which definition ran" (§9.2, versioning sibling).

The design-time inventory closes with the rule that animates the rest of the guide: **the definition is not the agent; the definition plus its bound artifacts is.** Every section below is about what that compound thing emits and what is recorded about it.

## 5. Runtime Artifacts I, Traces and Spans

Runtime artifacts are produced by runs and answer "what did this agent do, decide, and produce?" The trace is the spine of the runtime inventory: it is the single end-to-end record that links every other runtime artifact — each plan step, tool call, state write, and deliverable — to one auditable run.

### 5.1 Traces and Spans: The Vendor Conventions

**OpenAI Agents SDK.** The SDK's tracing model is the most completely documented one verified this pass. A **trace** "represent[s] a single end-to-end operation of a 'workflow'" and is "composed of Spans" ✅ (OpenAI Agents SDK docs, Tracing). Verified trace properties:

- `workflow_name` — "the name of the logical workflow or app" (e.g., "Code generation" or "Customer service") ✅
- `trace_id` — unique ID, "automatically generated if you don't pass one," and **must** have the format `trace_<32_alphanumeric>` ✅
- `group_id` — optional, "to link multiple traces from the same conversation," e.g., a chat thread ID ✅
- `disabled`, `metadata` — opt-out flag and free-form metadata ✅

Verified span model: spans have `started_at`/`ended_at`, a `trace_id`, a `parent_id`, and `span_data` typed per operation. Default tracing wraps each runner invocation (`task_span`), each model turn (`turn_span`), each agent run (`agent_span`), each LLM generation (`generation_span`), each function tool call (`function_span`), guardrails (`guardrail_span`), and handoffs (`handoff_span`), with audio spans (`transcription_span`, `speech_span`, `speech_group_span`) where applicable ✅ (OpenAI Agents SDK docs, Tracing). The default trace name is the literal string "Agent workflow" ✅. That span taxonomy is, in effect, the platform's own artifact inventory of a run.

Also verified: tracing is **on by default** and disable-able three ways (`OPENAI_AGENTS_DISABLE_TRACING=1`, `set_tracing_disabled(True)`, per-run `RunConfig.tracing_disabled`); it is **unavailable under Zero Data Retention (ZDR) policies** ✅ — a governance fact, not just an ops note, because it means a ZDR tenant cannot reconstruct the trace-level audit record from the vendor side; and custom trace processors can push traces to alternative destinations ✅ (all: Tracing docs). The documented ecosystem integrations (LangSmith, Langfuse, MLflow, AgentOps, Arize Phoenix, Datadog, and more) make the trace the portable unit of observability ✅ (Tracing docs, integrations list).

**OpenTelemetry GenAI semantic conventions.** OpenTelemetry's GenAI semantic conventions have moved out of the main semconv repository into a dedicated one — `open-telemetry/semantic-conventions-genai` — where markdown docs are generated from YAML definitions in a `model/` folder ✅ (opentelemetry.io GenAI semconv index; repo README). The documented surface is organized into span families per provider (OpenAI, Anthropic, AWS Bedrock, Azure AI Inference), plus general GenAI spans, agent spans, GenAI events, GenAI exceptions, GenAI metrics, and MCP conventions ✅ (opentelemetry.io semconv index). ⚠ Specific attribute names inside those families (e.g., the `gen_ai.*` registry entries) were not re-verified this pass — the taxonomy of span families is confirmed; the attribute-level detail is flagged in §11/§12. ⚠ **Divergence:** the OpenAI SDK's span vocabulary (agent/generation/function/guardrail/handoff) and OTel GenAI's (provider-modeled spans plus agent spans) are different schemas that say overlapping things; a multi-vendor estate needs an explicit mapping layer if traces are to be joined across platforms.

**Anthropic.** Anthropic's platform docs index confirms first-class session and transcript artifacts around Claude agents: Managed Agents expose session operations and session transcripts for compliance retrieval ✅ (platform.claude.com llms.txt: managed-agents/session-operations, manage-claude/compliance-sessions "Retrieve session transcripts"); a compliance API surface lists and retrieves local and remote sessions and their messages ✅ (llms.txt: api/compliance/apps/sessions/...). ⚠ A dedicated Claude Agent SDK *trace/span schema page* (analogous to OpenAI's) was not located this pass — see §12; the Claude Code Analytics API page exists in the index ✅ but its payload schema was not verified.

**Google ADK.** ADK's deployment story ties into Google Cloud Observability: deploying via Agent Runtime (Agent Platform), Cloud Run, or GKE "instantly" inherits "Cloud Trace observability" ✅ (adk.dev landing). ADK also structures every agent run as **Events** appended to a session — an event log that is simultaneously context and audit history (see §6.3) ✅ (ADK docs, State; Sessions). ⚠ ADK's own span/schema details were not re-verified beyond these surfaces (see §12).

**LangSmith.** LangSmith's tracing vocabulary — **runs** and **threads** inside tracing projects — is the substrate of its online evaluation ("evaluators run automatically on runs or threads") ✅ (LangSmith docs, Evaluation; Evaluation concepts). LangGraph's docs direct users to "trace, debug and evaluate your agents" with LangSmith ✅ (LangGraph docs, Graph API overview).

### 5.2 What a Production Trace Inventory Must Capture

Synthesis for the inventory — the minimal per-run trace record an auditable agent needs, each element verified against at least one platform above:

| Trace element | Verified platform naming | Why it is auditable |
|---|---|---|
| Run/operation ID | OpenAI `trace_id` (`trace_<32_alphanumeric>`) | The join key for everything else in this run |
| Logical workflow name | OpenAI `workflow_name` | What business function this run served |
| Conversation linkage | OpenAI `group_id`; LangSmith threads; ADK session ID | Reconstructing multi-turn context |
| Span tree | OpenAI span types; OTel GenAI span families | Agent→turn→generation→tool→guardrail→handoff structure |
| Span timestamps | OpenAI `started_at`/`ended_at` | Latency, time-of-action evidence |
| Vendor/transport | OTel GenAI per-provider families | Which provider's record to pull when the vendor is asked |

Runtime observability depth — dashboards, alerting, and cache-layer telemetry — belongs to the observability and cache siblings ([technology/ai_llm/agent_runtime_cache_design_guide.md](technology/ai_llm/agent_runtime_cache_design_guide.md) for context-cache design; [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md) for drift signals extracted from traces). This guide's contribution is only the artifact discipline: a trace that cannot be joined to its definition version (§3), its eval verdict (§7), and its governance record (§8) is a log, not a trail.

### 5.3 Span-to-Span Mapping Across Platforms

Because no two platforms share a span schema, a multi-vendor estate needs an explicit mapping. The table aligns the OpenAI SDK span taxonomy (fully verified) with the OpenTelemetry GenAI family names (verified at family level) and ADK/LangSmith equivalents (verified at surface level); cells marked ⚠ are mappings this guide asserts on the basis of naming and function, not verified schema equivalence:

| Run phase | OpenAI SDK span (✅ verified) | OTel GenAI family (✅ family exists) | ADK / LangSmith surface | Notes |
|---|---|---|---|---|
| Whole run | `task_span` (runner invocation) | GenAI/agent operation spans ⚠ | LangSmith run; ADK invocation + Events ✅ | Join key is trace/run/session ID |
| Single agent execution | `agent_span` | Agent spans ✅ | ADK agent step ⚠ | Agent boundaries differ by framework |
| LLM call | `generation_span` | Provider GenAI spans (OpenAI/Anthropic/…) ✅ | Turn in event log ⚠ | Provider span carries model/token usage per OTel ⚠ |
| Tool execution | `function_span` | Tool-call conventions ⚠ | Tool event with arguments ⚠ | MCP-spec tool records are transport-level truth ✅ |
| Guardrail | `guardrail_span` | Safety/guardrail spans ⚠ | Guardrail plugin events ⚠ | Which guardrail fired, on what |
| Handoff/delegation | `handoff_span` | Agent spans ⚠ | Sub-agent events ⚠ | Cross-agent control flow |
| Conversation grouping | `group_id` on trace ✅ | n/a | LangSmith thread ✅; ADK session ✅ | Multi-turn reconstruction |

The mapping is recorded as a configuration artifact in its own right — because the moment a trace leaves one platform for another, the schema translation is the part most likely to silently lose evidence (e.g., dropping the tool `isError` flag or the guardrail disposition). ⚠ Cells marked ⚠ should be validated against the OTel `semantic-conventions-genai` model YAML and each framework's reference implementation before production use; attribute-level detail was out of scope this pass (§12).

### 5.4 Trace Export, Retention, and the Privacy Boundary

Three verified facts govern how long traces may live and where they may go:

- **Export is a first-class capability.** OpenAI's SDK supports custom trace processors that "push traces to other destinations (as a replacement, or secondary destination)," with batching (`BatchTraceProcessor`) and explicit flush semantics for long-running workers (`flush_traces()`) ✅ (Tracing docs). An estate that needs its own audit store (ZDR, regulatory, or vendor-agnostic reasons) has a documented path.
- **Tracing can be switched off — which is itself a governance event.** Global (`OPENAI_AGENTS_DISABLE_TRACING=1`, `set_tracing_disabled(True)`) and per-run (`RunConfig.tracing_disabled`) controls exist ✅ (Tracing docs). A production system that disables tracing for a run is choosing to produce no audit record for it; that choice should be as gated as any other control change.
- **Vendor-side retention is not guaranteed.** ZDR customers have no vendor-held traces ✅ (Tracing docs); the same logic extends to any retention horizon shorter than the audit horizon. Trace export plus estate-side retention is the only robust answer — see §9.3 for the retention policy structure.

## 6. Runtime Artifacts II, Plans, Tool Calls, State, and Deliverables

### 6.1 Plans and Intermediate Reasoning

Agentic systems that "dynamically direct their own processes" (Anthropic's definition of agents proper ✅) produce intermediate reasoning artifacts before they act: candidate plans, scratchpads, and the model's own deliberation. Three verified anchors frame this artifact class:

- **Reasoning is a first-class, documented surface at Anthropic.** The platform docs index carries dedicated pages titled Thinking, Preserved thinking, and Extended thinking (legacy), alongside context-windows and prompt-caching pages ✅ (platform.claude.com llms.txt) — evidence that model reasoning is exposed, controllable, and in some configurations preserved across turns rather than discarded. ⚠ The exact wire format of those reasoning blocks was not re-verified this pass (see §12).
- **Plans are observable as spans or events.** Whatever internal form a plan takes, its *execution* is observable: OpenAI traces every model turn (`turn_span`), agent run (`agent_span`), and generation ✅, so the step-by-step trajectory of a plan is reconstructable from spans even when the plan text itself is not persisted. ADK makes intermediate action observable through Events appended to the session record ✅ (ADK docs, State).
- **Reasoning is sensitive by nature.** ⚠ Vendor-specific *retention/redaction policies for chain-of-thought text* could not be verified at a primary source this pass (see §12). Engineering practice — and the honest position of this guide — is to treat full reasoning traces as sensitive: store verdicts, citations, tool arguments, and outputs durably, but treat raw deliberation as a redactable, retention-limited artifact unless a verified policy says otherwise.

### 6.2 Tool-Call Records

Tool calls are where an agent touches the world — in a banking context, where money moves — so their records are the most legally load-bearing runtime artifacts. Verified record structure comes from the MCP specification:

- A call is a JSON-RPC exchange: `tools/call` with `name` and `arguments`; a result carrying `content` items and optional `structuredContent`; success vs. execution failure signaled by `isError`; protocol-level failures (unknown tool, invalid arguments) as JSON-RPC errors ✅ (MCP spec, server/tools).
- Content items are typed (`text`, `image` with base64 `data` and `mimeType`, `audio`, `resource_link` with `uri`/`mimeType`, embedded `resource`) ✅ (MCP spec, server/tools) — so a tool-call record can capture binary artifacts (signed documents, scanned instruments) with their MIME types intact.
- **The audit obligation is in the spec:** MCP clients SHOULD "log tool usage for audit purposes," and SHOULD validate tool results, implement timeouts, and prompt for confirmation on sensitive operations ✅ (MCP spec, server/tools, Security Considerations). A compliant client therefore produces a tool-usage audit log as a byproduct of every call.
- Framework-side, each call is a span with structured data: OpenAI's `function_span` records function tool calls ✅ (Tracing docs).

The MCP-spec record structure is worth showing in full, because it is the transport-level truth an audit will quote. These are the spec's own examples, trimmed ✅ (MCP spec, server/tools):

```json
// Request: invoke a tool
{ "jsonrpc": "2.0", "id": 2, "method": "tools/call",
  "params": { "name": "get_weather", "arguments": { "location": "New York" } } }

// Response: success
{ "jsonrpc": "2.0", "id": 2,
  "result": { "content": [ { "type": "text",
      "text": "Current weather in New York:\nTemperature: 72°F\nConditions: Partly cloudy" } ],
    "isError": false } }

// Response: tool execution error (business failure, in-band)
{ "jsonrpc": "2.0", "id": 4,
  "result": { "content": [ { "type": "text",
      "text": "Failed to fetch weather data: API rate limit exceeded" } ],
    "isError": true } }

// Protocol error: unknown tool (JSON-RPC level)
{ "jsonrpc": "2.0", "id": 3, "error": { "code": -32602,
  "message": "Unknown tool: invalid_tool_name" } }
```

For an audit record the salient distinction is *where* the failure is reported: an `isError: true` result means the tool ran and failed (a business outcome to record), while a JSON-RPC `error` means the call never executed (a protocol outcome to record). Both belong in the tool-usage audit log with the same trace linkage.

### 6.3 State, Memory, and Session Records

State is the runtime artifact class with the richest verified vendor vocabulary, and the widest naming divergence:

**ADK — Session, State, Memory, Artifacts.** ADK's model, verified in full:

- A **Session** is "a single, ongoing interaction between a user and your agent system," containing the chronological sequence of messages and actions — called **Events** — of that interaction ✅ (ADK docs, Sessions).
- **State** (`session.state`) is the session's scratchpad: serializable key-value pairs, string keys, scoped by prefix — no prefix (session-scoped), `user:` (across a user's sessions), `app:` (across all users of an app), and `temp:` (single-invocation only) ✅ (ADK docs, State).
- **Persistence is a service property:** `InMemorySessionService` loses everything on restart; `DatabaseSessionService`/`VertexAiSessionService` persist; state writes are applied through event `state_delta`s when the session service appends events ✅ (ADK docs, State).
- **Auditability is enforced by architecture.** ADK's docs warn that modifying `session.state` directly — outside a `CallbackContext`/`ToolContext` — "bypasses Event History: the change isn't recorded as an Event, losing auditability," and "will likely NOT be saved" by persistent services ✅ (ADK docs, State). The platform itself equates *state change without an event record* with *loss of auditability* — the single best one-line justification for this entire guide.
- **Memory** is the cross-session, searchable store managed by a `MemoryService`, distinct from session state ✅ (ADK docs, Sessions).
- **Artifacts** (ADK's term) are "named, versioned binary data associated either with a specific user interaction session or persistently with a user across multiple sessions," stored as `Part` objects (`inline_data` with `data` bytes and `mime_type`) and managed by an `ArtifactService` (`InMemoryArtifactService`, `GcsArtifactService`, `FileArtifactService`) with `save_artifact`/`load_artifact`/`list_artifacts`; saving under the same `filename` creates a new **version**, and `load_artifact` without a version argument returns the latest ✅ (ADK docs, Artifacts). ⚠ Terminology collision with this guide's "artifact" is flagged in §2.3.

**LangGraph — checkpoints and threads.** LangGraph checkpointers "save graph state as checkpoints at each step, organized into threads," which is what enables human-in-the-loop (interrupts), time travel ("replay prior graph executions to review and/or debug specific graph steps" and fork state to explore alternatives), conversational memory, and fault-tolerant resume ✅ (LangGraph docs, Checkpointers). A run must pass `{"configurable": {"thread_id": ...}}`; a thread's state at a point in time is a **checkpoint** (`StateSnapshot`), saved at every super-step boundary with per-node task writes for pending-write recovery ✅ (LangGraph docs, Checkpointers). For the inventory: a LangGraph deployment's durable state is a series of versioned checkpoints per thread — a runtime artifact with built-in time-travel that doubles as an incident-replay mechanism (§8).

**Anthropic — sessions and transcripts.** The Claude Agent SDK supports sessions that "maintain context across exchanges, resume or fork later" ✅ (Agent SDK overview); Managed Agents and the compliance APIs expose session transcripts and message retrieval for local and remote sessions ✅ (platform.claude.com llms.txt). Context engineering — what belongs in the window, what in memory, what in cache — is owned by [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md), with runtime cache design in [technology/ai_llm/agent_runtime_cache_design_guide.md](technology/ai_llm/agent_runtime_cache_design_guide.md).

### 6.4 Outputs and Deliverables

The terminal runtime artifact is the delivered work product: the final answer, the generated file, the executed transaction. Verified anchoring examples:

- **Structured outputs.** OpenAI agents declare `output_type` for structured output instead of plain text ✅ (Agents docs); LangGraph graphs can declare explicit output schemas distinct from input schemas ✅ (Graph API docs); MCP tools can declare `outputSchema` and return `structuredContent` ✅ (MCP spec).
- **Binary deliverables.** ADK's Artifact mechanism exists precisely to "enable tools or agents to generate binary outputs (like a PDF report or a generated image) that can be saved via `save_artifact` and later accessed" ✅ (ADK docs, Artifacts) — the canonical pattern for a bank agent that produces a PDF trade document or a CSV reconciliation.
- **Run results as artifacts.** Framework runs return result objects (OpenAI's `Runner.run` producing final output and run items ✅ (Agents docs, Results guide referenced)) and every platform ties outputs to the run/trace/session that produced them — which is the property that makes a deliverable auditable: it is reachable from its trace, which is reachable from its definition version.

The runtime inventory is complete when every plan step, tool call, state write, and deliverable carries enough linkage (trace ID, session/thread ID, event ID) to be replayed, re-evaluated, or produced in an audit. That linkage is the raw material of the quality artifacts in §7 and the governance artifacts in §8.

### 6.5 State and Memory Mechanisms Compared

The state artifact class has the richest vendor vocabulary — and the most consequential design choices. Comparison of the mechanisms verified above (with the naming divergence flagged):

| Dimension | ADK Session/State | LangGraph Checkpointer | Anthropic sessions/transcripts | OpenAI SDK (conversation) |
|---|---|---|---|---|
| Unit of state | `Session` with `session.state` (scoped keys: `user:`, `app:`, `temp:`) ✅ | Thread keyed by `thread_id`, state snapshots per super-step ✅ | Session maintained across exchanges, resume/fork ✅ | Platform conversation-state guidance exists ✅ (nav-level; ⚠ details unverified §12) |
| History record | Events appended to session (chronological); state deltas in event actions ✅ | Checkpoints (`StateSnapshot`) per super-step + per-node task writes ✅ | Session transcripts retrievable for compliance ✅ | Traces with `group_id` conversation linkage ✅ |
| Mutation discipline | State writes via contexts/event flow; direct edits "lose auditability" ✅ | State advances through graph updates; interrupts for human edits ✅ | SDK-managed | SDK-managed |
| Replay | Rewind sessions (docs page exists ✅ index-level) | Time travel: replay and fork at checkpoints ✅ | Fork later ✅ (overview) | Replay from traces ⚠ |
| Long-term memory | `MemoryService` (searchable cross-session) ✅ | Stores for durable knowledge (page exists ✅ index-level) | `.claude/` memory features ✅ (overview) | n/a this pass |
| Persistence choice | Service implementation decides (in-memory vs DB/Vertex) ✅ | Checkpointer implementation decides ✅ | Vendor-hosted or SDK-managed ✅ | Vendor/platform |

Reading the table as an architect: **session/thread identity is the spine of conversation state, and the event/checkpoint log is its audit record.** Whatever framework is chosen, the inventory should record (a) which store backs state, (b) whether that store is persistent, and (c) whether state mutations flow through the framework's audited path — because ADK's own warning generalizes: state changed outside the record is state with no evidence.

## 7. Quality Artifacts, Eval Suites, Datasets, and Guardrails

Quality artifacts answer "how do we know the agent is good, and still good?" They exist in the eval loop around runs, and the loop itself is closed: a production failure becomes a trace, the trace becomes a dataset example, the example becomes a regression test, and the test gates the next release. Methodology (which metrics, which judges, how to design evals) is owned by [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md) and [technology/ai_llm/llm_evaluation_vs_validation_guide.md](technology/ai_llm/llm_evaluation_vs_validation_guide.md); this section inventories the *artifacts* the methodology operates on.

### 7.1 Eval Suites: Datasets, Examples, Evaluators, Experiments

**LangSmith** provides the most complete verified artifact vocabulary:

- **Datasets** are "collection[s] of examples used for evaluating an application"; an **example** is "a test input, reference output pair," optionally with metadata, where reference outputs are used only by evaluators, never passed to the application ✅ (LangSmith docs, Evaluation concepts).
- **Datasets are versioned**: "every time you add, update, or delete examples in your dataset, a new version of the dataset is created," identified by timestamp, and taggable with semantic names — the docs' own example is tagging a version `"prod"` ✅ (LangSmith docs, Manage datasets). Evaluation can target a specific version via `as_of`/`asOf` ✅ (Manage datasets). A dataset version is thus a first-class, queryable artifact — the regression baseline in named form.
- **Evaluators** score outputs and come in four verified kinds: human review, code rules, LLM-as-judge, and pairwise comparison ✅ (LangSmith docs, Evaluation). Evaluators are "workspace-level resources" attachable across projects and datasets ✅ (Manage evaluators) — i.e., reusable artifacts in their own right.
- **Experiments** are the runs of an application against a dataset; results support benchmarking, unit tests, regression tests, and backtesting ✅ (LangSmith docs, Evaluation). Offline evaluation targets curated datasets ("test before you ship"); online evaluation targets live production runs/threads ("monitor in production") with reference-free checks: safety, format validation, quality heuristics, LLM-as-judge, with sampling-rate controls ✅ (Evaluation; Evaluation concepts).
- **The feedback loop is an artifact pipeline**: "Add failing production traces to your dataset, create targeted evaluators, validate fixes with offline experiments, and redeploy" ✅ (Evaluation); filtered experiment traces can be exported back into datasets ✅ (Manage datasets); and LangSmith Engine "can automatically generate ground truth dataset examples from your production traces" ✅ (Manage datasets).

**Anthropic** frames the same objects as evaluation practice: define success criteria that are specific and measurable ("Less than 0.1% of outputs out of 10,000 trials flagged for toxicity" beats "safe outputs"), test on held-out sets, and grade with code or with an LLM grader against a **golden answer** using a rubric — the docs' own grader prompt pattern instructs the judge to "Think through your reasoning in <thinking> tags, then output 'correct' or 'incorrect' in <result> tags" ✅ (platform.claude.com, Define success and build evaluations). Named metric families in the docs: F1, BLEU, perplexity, accuracy/precision/recall, latency, and rubric-based qualitative scales ✅.

**OpenAI** documents an Evals section on the platform — getting started, working with evals, graders, external models, and evaluation best practices — per the platform docs navigation ✅ (platform.openai.com docs nav). ⚠ The dataset file format behind those pages was not re-verified this pass (see §12).

**Google ADK** ships an evaluation component with dedicated pages for why-evaluate, criteria, custom metrics, environment simulation, and user simulation ✅ (adk.dev docs index). ⚠ ADK eval artifact formats were not re-verified this pass.

### 7.2 Fixtures and Golden Sets

Fixtures are the frozen, deterministic inputs an eval suite needs to be repeatable. Verified anchoring patterns:

- **Curated examples are the starting point**: LangSmith's guidance is to "start with manually curated examples" — 5–10 per critical component — defining ground truth for LLM calls, retrieval steps, tool invocations, and output formatting, and for agents "examples of correct tool selection and proper argument formatting or trajectory that the agent took" ✅ (Evaluation concepts). That is a *trajectory fixture* — a golden set that asserts not just the answer but the tool-use path to it.
- **Golden answers** are the reference artifacts LLM graders score against in Anthropic's documented eval pattern ✅ (develop-tests page).
- **Synthetic and derived fixtures**: LangSmith datasets can be built "from manually curated test cases, historical production traces, or synthetic data generation," and ground truth can be machine-generated from production traces (LangSmith Engine) ✅ (Evaluation; Manage datasets). Whatever the origin, a fixture is only as good as its provenance — the quality sibling's dataset-governance treatment applies.

### 7.3 Regression Baselines and Release Gates

Regression testing is one of LangSmith's named offline evaluation types ✅ (Evaluation); dataset versions tagged `"prod"` are the baseline mechanism ✅ (Manage datasets). The gate itself — which eval verdicts block a release, shadow, canary, and production promotion — is release-management territory owned by [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) (registries and eval-gated promotion) and [technology/ai_llm/multi_agent_banking_guide.md](technology/ai_llm/multi_agent_banking_guide.md) (production gates in banking). The taxonomy point: **a release gate is only as strong as the baseline artifact it compares against**, and baselines rot — which is why drift detection ([technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md)) must keep feeding the baseline, not just the reverse.

### 7.4 Guardrail Artifacts

Guardrails are the quality layer that runs *inside* the runtime path, so they straddle design, runtime, and quality:

- **Definition artifacts.** OpenAI agents declare `input_guardrails` (run on first user input) and `output_guardrails` (run on final output) as agent properties ✅ (Agents docs); their execution is traced in dedicated `guardrail_span`s ✅ (Tracing docs). Anthropic's docs index documents a guardrails program (mitigate jailbreaks and prompt injections, reduce hallucinations, reduce prompt leak, streaming refusals, increase output consistency) ✅ (platform.claude.com llms.txt). ADK's ecosystem includes guardrail plugins (e.g., an Agent Threat Rules guardrail plugin, per the docs index ✅ (adk.dev llms.txt)).
- **Runtime records.** Because guardrail execution is traced (OpenAI) and because MCP-mediated tools carry security annotations ✅ (MCP spec), a run's guardrail verdicts are recoverable artifacts: which guardrail fired, on what input, with what disposition.
- **Config as quality artifact.** Guardrail thresholds, allow/deny lists, and refusal phrasing are versioned config artifacts — the same registry discipline as prompts (versioning sibling), and the bias/red-teaming treatment is in [technology/ai_llm/ai_governance_bias_redteaming_guide.md](technology/ai_llm/ai_governance_bias_redteaming_guide.md).

The quality inventory closes with a loop, not a shelf: every production trace worth remembering becomes a dataset example (§7.1), every dataset version is a baseline (§7.3), and every gate verdict is itself an artifact the governance layer (§8) will want to see.

### 7.5 From Fixture to Gate: The Quality Pipeline as an Artifact Chain

The quality class is best understood as a pipeline whose stages are artifacts — a failed production run travels the chain and returns as a gate:

| Stage | Artifact produced | Verified anchoring | Gate purpose |
|---|---|---|---|
| 1. Capture | Production trace (the failure) | "Add real traces to a dataset so a failure you saw once becomes a test" ✅ (LangSmith, Evaluation); export filtered traces from experiments to datasets ✅ | Evidence that a real failure exists |
| 2. Curate | Dataset example (inputs + reference outputs + metadata) | Example structure ✅ (Evaluation concepts); 5–10 curated examples per critical component to start ✅ | Frozen, reviewable ground truth |
| 3. Version | Dataset version, tagged (`"prod"`) | Version created on every example change; taggable; addressable via `as_of` ✅ (Manage datasets) | Immutable regression baseline |
| 4. Score | Experiment with evaluator verdicts | Experiments + evaluators (human/code/LLM-judge/pairwise) ✅ (Evaluation) | Candidate quality evidence |
| 5. Gate | Release decision bound to the experiment | Eval-gated promotion discipline (versioning sibling); offline benchmarking/regression/unit/backtest types ✅ | Verdict that blocks or passes a release |
| 6. Monitor | Online-eval verdicts on live runs/threads | Online evaluation on production traces: safety, format, quality heuristics, LLM-as-judge ✅ (Evaluation) | Drift early-warning |

The loop closes when stage 6 finds a new failure and stage 1 ingests it again. Two artifacts make the loop trustworthy: the **dataset version** (so "regression" means "against a frozen set," not "against whatever the dataset is today") and the **experiment record** (so a gate verdict can be re-audited). Both are versioned, addressable artifacts on the LangSmith model ✅ — and both should be referenced, not just summarized, in the governance records of §8.

### 7.6 Dataset Shapes Compared

When datasets must move between platforms (or be recreated after a platform exit), the shapes matter. The verified structures:

| Dataset artifact | Shape (verified) | Versioning | Source |
|---|---|---|---|
| LangSmith dataset | Collection of **examples**; example = `inputs` (dict passed to the app) + optional `reference outputs` (dict used only by evaluators) + optional `metadata` (dict for filtered views) ✅ | Versioned per add/update/delete; tagged (`"prod"`); addressed via `as_of` ✅ | docs.langchain.com (Evaluation concepts; Manage datasets) |
| Anthropic golden eval set | In-code rows pairing a prompt with a **golden answer** for grader comparison — the docs' own example shape is `{ question: ..., golden_answer: ... }` rows graded against a rubric ✅ | Not a platform dataset service; version with your repo/registry ⚠ | platform.claude.com (develop-tests code samples) |
| ADK evaluation | Dedicated evaluation component (criteria, custom metrics, user/environment simulation) ✅ — artifact formats ⚠ unverified (§12) | n/v this pass | adk.dev docs index |
| OpenAI platform evals | Dedicated Evals section (getting started, working with evals, graders, best practices) ✅ — dataset file format ⚠ unverified (§12) | n/v this pass | platform.openai.com docs nav |

⚠ The portability lesson: LangSmith and Anthropic both define a "golden set" as input-plus-reference pairs, but LangSmith stores it as a versioned platform dataset while Anthropic's documented pattern is code-level rows you version yourself. Neither format imports into the other as-is — treat dataset migration as a transformation project with its own test, not a copy operation.

## 8. Governance Artifacts, Audit Logs, Decisions, Incidents, and Compliance

Governance artifacts are the permanent record of accountability: who decided, what happened, who is answerable. Where the runtime layer records *facts about runs*, the governance layer records *judgments about facts* — approvals, decisions, incidents, dispositions, compliance assertions — and must keep both linkable. Framework-level governance (AI governance programs, model risk, roles, policies) is owned by [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md), with bias and red-teaming in [technology/ai_llm/ai_governance_bias_redteaming_guide.md](technology/ai_llm/ai_governance_bias_redteaming_guide.md); this section inventories the *record artifacts* those programs produce and consume.

### 8.1 Audit Logs

An audit log is the governance artifact with the most verified vendor grounding:

- **Tool-usage logs are a protocol obligation.** The MCP specification's Security Considerations direct clients to "log tool usage for audit purposes," alongside validating tool results and prompting confirmation on sensitive operations ✅ (MCP spec, server/tools). For an MCP-mediated bank agent, every payment-related tool invocation *should* produce a durable audit entry as a matter of protocol conformance, not local policy.
- **Event-log architecture is the audit substrate.** ADK stores every agent action as an Event appended to the session and warns that state changes made outside the event flow "aren't recorded as an Event, losing auditability" ✅ (ADK docs, State). Designing state mutation to flow through event logs is what makes the audit log complete rather than best-effort. ADK's documented session services include per-session locking for concurrent writes ✅ (ADK docs, State) — an audit log is only trustworthy if it is also concurrency-safe.
- **Transcript-level records exist for compliance.** Anthropic's platform exposes compliance-grade artifacts: session transcripts for Managed Agents ("Retrieve session transcripts") and a compliance API for listing/retrieving local and remote sessions and messages ✅ (platform.claude.com llms.txt: manage-claude/compliance-sessions; api/compliance/apps/sessions/...). Where the vendor offers a transcript retrieval surface, the audit log can cite vendor-held evidence, not only internally stored spans.
- **Trace-level audit records have a platform caveat.** OpenAI's tracing is unavailable to organizations on Zero Data Retention policies ✅ (OpenAI Agents SDK docs, Tracing) — meaning a ZDR tenant must not rely on vendor-side trace retention for audit, and should export its own (custom trace processors are the documented mechanism ✅).

### 8.2 Decision Records: The ADR Pattern for Agents

Software engineering's Architecture Decision Record (ADR) pattern — a dated, numbered, immutable note stating a decision, its context, and its consequences — maps cleanly onto agent decisions, and this guide's position is that agent decisions warrant *more* records than classic ADRs, because the artifact being decided on (a prompt, a model pin, a tool permission) is more volatile than a codebase. ⚠ No vendor primary source documents an "agent ADR" format — this is an engineering practice, not a platform convention — so the honest framing is: adopt whatever ADR convention the estate already uses, and bind it to agent artifacts (prompt version, model alias, eval verdict) the way [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) binds versions to registries. Decision records that an agent estate typically needs: model adoption/retirement, temperature/reasoning-effort policy, tool approval and permission scopes, guardrail threshold changes, knowledge-corpus updates, and incident dispositions (§8.3).

### 8.3 Incident and Rollback Records

Incidents in agent systems are special because the "bad behavior" may not be a code defect — it may be drift in a model the vendor updated, a knowledge corpus that went stale, or an eval baseline that rotted. The record-keeping therefore pairs an incident record (what happened, trace IDs, which version, which eval was or wasn't passed) with a rollback record (to which version, by which gate, verified by what re-eval):

- **Incident reconstruction is a documented platform capability.** LangGraph checkpointers enable "time travel" — replaying prior executions and forking state at arbitrary checkpoints ✅ (LangGraph docs, Checkpointers) — which turns incident review from reading logs into re-running the graph. Traces are the incident's timeline (OpenAI's span tree ✅); dataset versions are the incident's baseline (LangSmith ✅).
- **Rollback mechanics belong to the versioning sibling.** Release gates, shadow/canary promotion, rollback strategies, and model-pin discipline are in [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md); the *causes* of post-deploy behavior change — model updates, knowledge staleness, performance decay — are in [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md). The governance record's job is to link an incident record to the rollback record and to the drift evidence that triggered it.
- **Human-in-the-loop approval is itself a recorded artifact.** LangGraph interrupts make human review a graph state ✅ (Checkpointers); MCP's spec puts "a human in the loop with the ability to deny tool invocations" as a SHOULD for trust and safety ✅ (MCP spec, server/tools). Approval events (who approved which tool call at which checkpoint, in which trace) are governance artifacts that the audit trail must retain.

### 8.4 Compliance Reports and External Frameworks

The compliance report is the governance artifact that faces outward — internal audit, model risk management, regulators. Two verified external frameworks anchor the report's structure:

- **NIST AI RMF.** NIST's AI Risk Management Framework (AI RMF 1.0, NIST.AI.100-1) was released January 26, 2023 as a voluntary framework "to improve the ability to incorporate trustworthiness considerations into the design, development, use, and evaluation of AI products, services, and systems"; a GenAI profile (NIST-AI-600-1) followed on July 26, 2024; and NIST announced the AI RMF 1.0 "is being revised as part of the White House AI Action Plan," with an April 7, 2026 concept note for a critical-infrastructure profile ✅ (nist.gov, AI RMF page). A compliance report that cites the RMF should name the version it maps to, because the framework itself is a moving artifact.
- **OWASP GenAI / LLM guidance.** The OWASP Top 10 for LLM Applications (whose legacy taxonomy runs LLM01–LLM10, including LLM08 **Excessive Agency** — "granting LLMs unchecked autonomy to take action" — and LLM06 sensitive-information disclosure ✅ (owasp.org project archive)) has moved to the OWASP GenAI Security Project, which published the OWASP GenAI LLM Top 10 2026 on August 4, 2026 ✅ (owasp.org project page). ⚠ The GenAI Security Project's agentic-AI-specific threat document could not be fetched at its primary site this pass (see §12); the legacy Top-10 items above are verified from the archive page.
- **MCP's own governance community.** MCP hosts community charters including a Financial Services interest group ✅ (modelcontextprotocol.io llms.txt) — an ecosystem signal that banking-specific MCP governance (tool vetting, server trust) is being standardized at the protocol level.

A compliance report for an agentic solution is only as strong as its evidence chain: it maps regulatory/framework requirements to artifact classes (design manifests for *what the agent is*, trace/tool records for *what it did*, eval verdicts for *how it was tested*, decision and incident records for *who answered for it*). The deep AI-governance program content — roles, model risk, policies — is in [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md); this guide's contribution is the inventory of records the program must retain.

### 8.5 Governance Record Register: Contents, Sensitivity, Retention

The governance class is only as good as its register discipline. Suggested contents per record type, with sensitivity and retention guidance (retention defaults are this guide's recommendation — ⚠ — not a verified platform rule; regulatory horizons from the banking siblings override them):

| Record type | Required contents (invented field sets; formats per §8.1–§8.4) | Sensitivity | Retention default (⚠) |
|---|---|---|---|
| Tool-usage audit entry | Timestamp, trace ID, tool name, arguments (or hash), result/disposition, actor (human or agent), approval event ID | High — contains business data | Regulatory horizon (per record-keeping rules) |
| Session/event record | Session ID, chronological events, state deltas, transcript pointer | High — personal/customer data | Regulatory horizon; redact on request |
| Decision record (ADR) | ID, date, decider, context, decision, artifact versions bound (prompt, model, tool, dataset), consequences | Medium — internal rationale | Permanent |
| Incident record | ID, trace IDs, definition version, eval history ("which gate did it pass?"), timeline, impact, root cause | High — post-incident sensitive | Permanent (lessons) |
| Rollback record | From-version, to-version, trigger, gate evidence, re-eval result after rollback | Medium | Permanent |
| Compliance report | Framework/regulation mapped (NIST AI RMF 1.0 / GenAI profile ✅; OWASP GenAI Top 10 ✅), artifact evidence index, sign-off | High — may be privileged | Permanent |

The invariant that makes the register auditable: every governance record **references** the runtime/quality artifacts it is about (trace IDs, dataset versions, experiment IDs) rather than paraphrasing them — paraphrase is where evidence goes to die, and the whole point of the artifact trail is that the primary record still exists to be walked.

## 9. The Artifact Lifecycle, Condensed

The four classes meet in a five-stage lifecycle — **create → version → evaluate → deploy → audit** — where each stage reads artifacts from the previous one and writes artifacts the next one consumes. The full machinery of stages 2–5 is owned by siblings; what follows is the condensed map, the stage-to-sibling handoff, and the lineage rules that make the trail continuous.

### 9.1 Stage Map

| Stage | Reads | Writes | Deep-dive owner |
|---|---|---|---|
| **Create** | Requirements, platform conventions | Definitions, prompts, workflows, tool/MCP configs, model configs, scaffolding | This guide §3–§4; [technology/ai_llm/agent_scaffolding_guide.md](technology/ai_llm/agent_scaffolding_guide.md) |
| **Version** | Definitions as they drift | Immutable version snapshots; registries; release candidates | [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |
| **Evaluate** | Candidates + quality artifacts | Experiment results, baselines, gate verdicts | This guide §7; [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md); [technology/ai_llm/llm_evaluation_vs_validation_guide.md](technology/ai_llm/llm_evaluation_vs_validation_guide.md) |
| **Deploy** | Approved candidate | Runtime artifacts: traces, plans, tool records, state, outputs | This guide §5–§6; [technology/ai_llm/multi_agent_banking_guide.md](technology/ai_llm/multi_agent_banking_guide.md) (production gates); [technology/ai_llm/agent_runtime_cache_design_guide.md](technology/ai_llm/agent_runtime_cache_design_guide.md) (context layer) |
| **Audit** | The whole trail | Decision records, incident/rollback records, compliance reports | This guide §8; [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md); [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md) |

The lifecycle is a loop with two feedback arcs that keep it alive: **quality feeds back** (a production trace becomes a dataset example, §7.1 — "a failure you saw once becomes a test you run every time" ✅ (LangSmith docs)) and **governance feeds back** (an incident disposition triggers a new decision record and a re-evaluation, §8.3).

### 9.2 Lineage: What Makes It a Trail

An inventory of artifacts is a *trail* only when every artifact carries lineage — the pointers that let an auditor walk from a compliance question back to a specific run, and from a run back to the exact definition that produced it. The lineage disciplines, each grounded in a verified mechanism above:

- **Run identity.** Traces carry unique IDs (`trace_id` in the OpenAI SDK ✅); conversations group traces (`group_id`, LangSmith threads, ADK session IDs); checkpoints organize state by `thread_id` (LangGraph ✅). Every runtime artifact should record the run/session/thread ID it belongs to.
- **Definition identity.** Every runtime record should be joinable to the *definition version* that ran — the manifest/registry discipline of the versioning sibling — because a trace without a definition version cannot answer "which prompt was live when this payment was authorized?"
- **Quality identity.** Eval verdicts reference dataset versions (tagged `"prod"`, addressed by `as_of` in LangSmith ✅) and experiment IDs, so "this release passed regression" is a checkable claim, not a memory.
- **Immutability and hashing.** ⚠ No vendor primary source verified this pass prescribes a content-hash/provenance scheme for agent artifacts (beyond ADK's built-in artifact versioning ✅ and LangSmith's dataset versioning ✅); immutable, hashed, append-only storage for the governance record remains an engineering judgment call — flagged in §12 rather than asserted as a platform convention.

### 9.3 Retention

Retention is a governance decision that the artifact classes force early: traces and transcripts are privacy-bearing (Anthropic exposes transcripts as compliance artifacts ✅); reasoning text is sensitive by default (§6.1); datasets hold customer-derived examples; ZDR tenants cannot lean on vendor trace retention (OpenAI ✅). The defensible default, stated as this guide's recommendation rather than a verified platform rule: keep the *definition, verdict, and governance* layers permanently; keep *trace/tool-call* records per the regulatory horizon that applies (banking record-keeping rules, which the banking siblings cover); and keep *raw reasoning* short-lived unless a use case justifies longer retention. ⚠ Specific retention horizons are jurisdiction- and use-case-dependent and are not asserted here. As a working matrix:

| Artifact class | Default disposition (⚠ recommendation) | Rationale anchored above |
|---|---|---|
| Design definitions, prompts, manifests | Permanent, versioned | They are "what the agent was" — needed for any later audit or rollback (§3, §9.2) |
| Eval datasets, experiments, baselines | Permanent while the agent lives | Corporate memory of "good"; regression baselines must be re-auditable (§7) |
| Traces, tool-call records, session events | Regulatory horizon (banking record-keeping rules); else bounded | Privacy-bearing (transcripts ✅); tool records are the legal load-bearing layer (§6.2) |
| Raw reasoning / chain-of-thought | Short-lived, redactable | Sensitive by default (§6.1); no verified vendor retention policy to lean on (§12) |
| Governance register (ADRs, incidents, reports) | Permanent, append-only | The accountability record itself (§8.5) |

### 9.4 The Five-Hop Drill: A Lineage Walk

A useful acceptance test for an artifact inventory is the five-hop drill: take a random production incident and walk five links, each hop requiring an actual artifact, not a memory. Using the invented Cymbal Bank identifiers from §10 as the example:

1. **Compliance question → incident record.** The Q3 compliance report references `INC-2026-0142` (a wrong-beneficiary drafting incident). Hop 1 requires the report to *cite*, not paraphrase, the incident record.
2. **Incident record → trace.** `INC-2026-0142` lists the failing run's `trace_id` and the definition version that was live. Hop 2 resolves the trace and confirms the span tree shows the drafting path.
3. **Trace → tool-call record.** The trace's `function_span`/MCP record shows `create_mt103_instruction` called with the wrong beneficiary argument, and the guardrail span shows the output guardrail did not fire (or fired late). Hop 3 exposes the actual defect.
4. **Tool record → state/checkpoint.** The case thread's checkpoints show the state where the beneficiary was set, and the session event log shows which instruction template expanded into the argument. Hop 4 identifies the root cause (stale instruction template, drifted corpus, or model update).
5. **Root cause → decision + dataset.** The trail ends at the ADR that approved the faulty template version and the dataset version whose eval verdict did not catch the case. Hop 5 yields the two artifacts that governance must now change: a new ADR and a new dataset example.

If any hop fails — a trace with no definition version, an incident record with no trace IDs, a verdict with no dataset version — the inventory is a collection, not a trail. The drill is cheap to run and brutal to fail; run it quarterly.

## 10. Worked Example, The Cymbal Bank Trade-Services Agent

Cymbal Bank (the only bank persona permitted in this research library) operates a production **trade-services agent**: a multi-agent solution that handles documentary-credit and trade-payment work for corporate clients — answering queries on outstanding letters of credit, drafting payment instructions against verified documents, checking sanction/limit conditions through bank systems, and proposing settlement entries for an operations officer to approve. Below is the full artifact inventory such a solution ships with, laid out by class. **Artifact names are invented for the example; every format claim in the 'form/provenance' column traces to a primary source cited earlier in this guide.** Formats are real; the specific files are illustrative.

### 10.1 Design-Time Artifacts (Definition Layer)

| Artifact (invented name) | Class | Form / provenance (verified) | Owned by |
|---|---|---|---|
| `TRADESVC/AGENTS.md` (repo instructions for coding agents working on the agent codebase) | Definition | Markdown, no required fields, closest-file-wins ✅ (agents.md) | [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md) |
| `root_agent.yaml` — orchestrator definition | Definition | YAML Agent Config with `$schema` JSON Schema; keys `name`, `model`, `description`, `instruction`, `tools`, `sub_agents.config_path` ✅ (ADK docs, Agent Config) | [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |
| `lc-orchestrator` (graph) — routing, sanction-check, drafting, review nodes | Definition/control | LangGraph `StateGraph` with `State` schema, `Nodes`/`Edges`, `.compile()` ✅ (LangGraph docs, Graph API) | This guide §4.2; [technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md](technology/ai_llm/hierarchical_multi_agent_frameworks_guide.md) |
| System prompt `trade-execution/v14` + state templates | Definition/prompt | Versioned prompt artifact; dynamic-instruction callbacks ✅ (OpenAI Agents docs); `{key}` state templating ✅ (ADK docs) | [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |
| `mcpServers` config: `core-banking`, `sanctions`, `documents` | Definition/MCP | Client config JSON binding server names to `command`/`args`/`env` ✅ (MCP docs, connect-local-servers); agent-level `mcp_servers`/`mcp_config` ✅ (OpenAI Agents docs) | [technology/ai_llm/mcp_discovery_guide.md](technology/ai_llm/mcp_discovery_guide.md); [technology/ai_llm/mcp_framework_tools_guide.md](technology/ai_llm/mcp_framework_tools_guide.md) |
| Tool `create_mt103_instruction` + `check_sanctions` definitions | Definition/tool | MCP tool fields `name`, `title`, `description`, `inputSchema`, `outputSchema`, `annotations` ✅ (MCP spec, server/tools) | [technology/ai_llm/mcp_framework_tools_guide.md](technology/ai_llm/mcp_framework_tools_guide.md) |
| Model & parameter pin: `gemini-flash-latest`-class alias, temperature 0, `tool_choice` policy | Definition/model | Model + `model_settings` parameters (`temperature`, `top_p`, `tool_choice`) ✅ (OpenAI Agents docs); ADK `model:` key ✅ | [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |

### 10.2 Runtime Artifacts (Execution Layer)

| Artifact | Class | Form / provenance (verified) | Owned by |
|---|---|---|---|
| Run trace per client request | Runtime/trace | Trace with `workflow_name` ("trade-payment"), `trace_id`, `group_id` (case ID) ✅ (OpenAI Tracing docs) | This guide §5; [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md) |
| Span tree: agent → turns → generations → tool calls → guardrails → handoffs | Runtime/trace | Typed spans ✅ (OpenAI Tracing docs); OTel GenAI span families ✅ (opentelemetry.io) | This guide §5 |
| Graph state history per case | Runtime/state | Checkpoints per super-step in a `thread_id` thread; time-travel replay ✅ (LangGraph Checkpointers) | This guide §6.3 |
| Session event log (case conversation) | Runtime/state | Chronological Events appended to session; state deltas recorded as events ✅ (ADK docs, State/Sessions) | [technology/ai_llm/context_engineering_guide.md](technology/ai_llm/context_engineering_guide.md) |
| Tool-call records (`check_sanctions` → verdict; `create_mt103_instruction` → draft ID) | Runtime/tool record | JSON-RPC `tools/call` + result with `content`/`structuredContent`/`isError` ✅ (MCP spec); `function_span` ✅ (OpenAI Tracing) | This guide §6.2; [technology/ai_llm/agent_scaffolding_guide.md](technology/ai_llm/agent_scaffolding_guide.md) |
| Draft payment instruction `MT103-2026-0917-001.pdf` | Runtime/deliverable | Binary artifact saved via `save_artifact` (filename-versioned; `Part` with `mime_type` `application/pdf`) ✅ (ADK docs, Artifacts) | This guide §6.4 |
| Operator approval event (HITL) | Runtime/approval | Interrupt + resume at checkpoint ✅ (LangGraph Checkpointers); human-in-the-loop SHOULD for tool calls ✅ (MCP spec) | [technology/ai_llm/multi_agent_banking_guide.md](technology/ai_llm/multi_agent_banking_guide.md) |

### 10.3 Quality Artifacts (Evaluation Layer)

| Artifact | Class | Form / provenance (verified) | Owned by |
|---|---|---|---|
| Dataset `lc-case-trajectories`, version tagged `prod` | Quality/dataset | Versioned dataset of examples (inputs + reference outputs + metadata); tag `"prod"`, address via `as_of` ✅ (LangSmith docs) | [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md) |
| Golden set: 40 sanctioned-document cases with golden drafting answers | Quality/fixture | Held-out examples graded against golden answers by rubric ✅ (Anthropic, develop-tests) | [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md) |
| Trajectory fixtures (correct tool-selection paths) | Quality/fixture | Examples asserting tool selection and argument formatting ✅ (LangSmith, Evaluation concepts) | [technology/ai_llm/llm_evaluation_vs_validation_guide.md](technology/ai_llm/llm_evaluation_vs_validation_guide.md) |
| Evaluators: schema-validator (code), sanctions-citation (LLM-as-judge), pairwise for draft quality | Quality/evaluator | Evaluator kinds: code rules, LLM-as-judge, pairwise ✅ (LangSmith docs) | [technology/ai_llm/llm_evaluation_frameworks_guide.md](technology/ai_llm/llm_evaluation_frameworks_guide.md) |
| Experiment run `v14-vs-v13-lc-corpus` verdicts | Quality/experiment | Experiments comparing versions; regression-test type ✅ (LangSmith docs) | [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |
| Guardrail config: `no-unverified-sanctions-clearance` output guardrail | Quality/guardrail | `output_guardrails` on agent ✅ (OpenAI Agents docs); traced `guardrail_span` ✅ | [technology/ai_llm/ai_governance_bias_redteaming_guide.md](technology/ai_llm/ai_governance_bias_redteaming_guide.md) |

### 10.4 Governance Artifacts (Accountability Layer)

| Artifact | Class | Form / provenance (verified) | Owned by |
|---|---|---|---|
| Tool-usage audit log (every `create_mt103_instruction` call) | Governance/audit | Client obligation to "log tool usage for audit purposes" ✅ (MCP spec, Security Considerations) | [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md) |
| ADR `ADR-0047: adopt v14 drafting prompt` | Governance/decision | ADR pattern (engineering practice, ⚠ not a platform format — §8.2) bound to prompt version + eval verdict | [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |
| Session transcripts for compliance (per case) | Governance/audit | Transcript retrieval surface ✅ (platform.claude.com, compliance-sessions) — where vendor-hosted agents are used | [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md) |
| Incident record `INC-2026-0142` (wrong beneficiary drafted) + rollback record to prompt v13 | Governance/incident | Time-travel replay of the failing run ✅ (LangGraph Checkpointers); rollback per release-gate discipline ✅ (versioning sibling) | [technology/ai_llm/ai_agent_drift_guide.md](technology/ai_llm/ai_agent_drift_guide.md); [technology/ai_llm/agent_versioning_guide.md](technology/ai_llm/agent_versioning_guide.md) |
| Compliance report `Q3-2026 trade-agent AI RMF mapping` | Governance/compliance | Maps artifact evidence to NIST AI RMF 1.0 / GenAI profile (NIST-AI-600-1) ✅ (nist.gov) | [technology/ai_llm/ai_governance_framework_guide.md](technology/ai_llm/ai_governance_framework_guide.md) |

### 10.5 The Audit Walk-Through

To make the inventory concrete, walk one case end to end. The identifiers are invented; every artifact type and format below is the verified one from the tables above.

A corporate client's trade desk asks the agent to draft a settlement instruction for invoice `AP-8831` under letter of credit `LC-7781`. What the artifact trail should contain when the case is done:

1. **Definition resolved.** The case is routed by the compiled `lc-orchestrator` graph (LangGraph `StateGraph` definition, §10.1) running definition version `trade-agent/2026-09-17/v14`, whose registry entry pins system prompt `trade-execution/v14`, the `core-banking`/`sanctions`/`documents` MCP server configs, and the model alias with `temperature 0` (versioning sibling's registry).
2. **Run recorded.** The whole case executes under one trace (`workflow_name` "trade-payment", `trace_id` `trace_<32 alphanumeric>`, `group_id` = case `CASE-2026-0917-0042`) with the verified span tree: `agent_span` → `turn_span`s → `generation_span`s → `function_span`/guardrail spans (OpenAI vocabulary) or the OTel GenAI equivalents if the estate maps to OTel (§5.3).
3. **Tools called with records.** `check_sanctions` returns a structured verdict (`structuredContent`, no `isError`); `create_mt103_instruction` returns a draft ID. Every call produces a tool-usage audit entry per the MCP client obligation (§6.2, §8.1). The output guardrail `no-unverified-sanctions-clearance` fires a `guardrail_span` confirming clearance text is present.
4. **State advanced through events.** The case thread (LangGraph `thread_id`) accrues checkpoints per super-step; the ADK-style session event log (if the estate runs the ADK variant) records each state delta. The drafting step's expanded instruction text is recoverable from the event record, so "which instructions produced this draft" is answerable (§6.3).
5. **Deliverable versioned.** The draft `MT103-2026-0917-001.pdf` is saved through the artifact service (`save_artifact`, filename-versioned, MIME `application/pdf`), and the approval event — an operations officer accepting the draft at a graph interrupt — is recorded with the approver identity and trace linkage (§6.3, §6.4, §8.3).
6. **Quality verdict attached.** Before the definition that produced this draft was promoted, experiment `v14-vs-v13-lc-corpus` scored it against dataset version `lc-case-trajectories@prod` (tagged version, addressed by `as_of`), including the 40-case golden set with rubric-graded drafting answers (§10.3). The case itself, if it later fails, becomes dataset example #41.
7. **Governance closed.** If this case is later disputed, the compliance report's artifact index points to the case trace; the trace points to the tool records and checkpoints; the checkpoints point to the definition version; the definition version points to the ADR and eval verdicts that released it. `INC-2026-0142` from §10.4 is precisely such a case — its five-hop walk is the §9.4 drill.

That is the artifact trail as it should exist in production at Cymbal Bank: not a documentation exercise, but the joined evidence structure that lets an auditor reconstruct a payment-drafting decision in an afternoon — and lets an engineer find the defect in the same afternoon.

The reading order is the whole point: start from the compliance report, follow the incident record to the trace, follow the trace to the tool-call record and the checkpoint, follow the definition version to the prompt and the dataset verdict — and at every hop the artifact is named, the format is standard, and the link is recorded. That is the artifact trail in its production form: not a diagram, but an inventory that an auditor, an engineer, and a regulator can all walk.

## 11. Claims Audit

Every platform-specific claim above was checked against a primary source during this research pass. The table records the audit trail of the audit guide: ✅ = verified directly at the cited source this pass; ⚠ = plausible, partially verified, or verified only at a secondary surface (flagged honestly); ❌ = the claim failed verification and was corrected or dropped. "This pass" refers to the September 2026 research session for this guide.

| # | Claim | Status | Source checked this pass |
|---|---|---|---|
| 1 | OpenAI Agents SDK traces: `workflow_name`, `trace_id` (`trace_<32_alphanumeric>`), `group_id`, `metadata`, `disabled` | ✅ | openai.github.io/openai-agents-python/tracing/ |
| 2 | OpenAI span types: task, turn, agent, generation, function, guardrail, handoff, transcription, speech | ✅ | Same |
| 3 | OpenAI tracing disabled under Zero Data Retention; custom trace processors supported | ✅ | Same |
| 4 | OpenAI `Agent` properties (name, instructions, model, model_settings incl. temperature/top_p/tool_choice, tools, mcp_servers, handoffs, input/output guardrails, output_type, hooks) | ✅ | openai.github.io/openai-agents-python/agents/ |
| 5 | OpenAI `SandboxAgent` with `default_manifest`, `base_instructions`, `capabilities`, `run_as` | ✅ | Same |
| 6 | OpenAI platform prompt templates referenced via `prompt` (`{{variable}}` syntax) | ✅ | Same (prompt templates section) |
| 7 | OpenAI platform Evals docs section (getting started, working with evals, graders, best practices) exists | ✅ | platform.openai.com docs navigation (fetched) |
| 8 | OpenAI Evals dataset file format details | ⚠ | Nav only; page scrape failed (see §12) |
| 9 | AGENTS.md: open format, README-for-agents framing, no required fields, nested closest-file-wins, client configs (`.aider.conf.yml`, `.gemini/settings.json`) | ✅ | agents.md |
| 10 | AGENTS.md stewarded by Agentic AI Foundation under Linux Foundation; originated from Codex/Amp/Jules/Cursor/Factory | ✅ | agents.md (About) |
| 11 | AGENTS.md vs platform manifests are interchangeable agent-definition formats | ❌ | Not supported — they are different artifact types (§3.1) |
| 12 | Anthropic "Building effective agents": workflows vs agents definitions; patterns (chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer); ACI guidance | ✅ | anthropic.com/engineering/building-effective-agents |
| 13 | Claude Agent SDK: Python/TypeScript; agent loop; hooks; subagents; MCP; permissions; sessions; skills/commands/memory from `.claude/` and `~/.claude/`; plugins | ✅ | platform.claude.com/docs/en/agent-sdk/overview |
| 14 | Anthropic docs index has Thinking, Preserved thinking, Extended thinking (legacy), context-windows, prompt-caching pages; compliance session-transcript APIs | ✅ | platform.claude.com/llms.txt |
| 15 | Anthropic eval guidance: measurable success criteria, held-out sets, golden answers, rubric grader with `<thinking>`/`<result>` tags | ✅ | platform.claude.com/docs/en/test-and-evaluate/develop-tests |
| 16 | Anthropic Agent Skills file specification (SKILL.md frontmatter/body conventions) | ⚠ | Docs index only; spec not re-fetched (see §12) |
| 17 | ADK: `Agent(name, model, instruction, tools)` code-first across 5 languages; ADK 2.0 graph workflows; Cloud Trace on Agent Runtime/Cloud Run/GKE deploys | ✅ | adk.dev (landing) |
| 18 | ADK Agent Config: `root_agent.yaml`, `$schema` AgentConfig.json, keys name/model/description/instruction/tools/sub_agents, `adk create/web/run/api_server`, experimental Gemini-only limits | ✅ | adk.dev/agents/config |
| 19 | ADK Sessions/State: Events, `session.state`, prefixes (`user:`, `app:`, `temp:`), SessionService persistence semantics, direct state writes lose auditability | ✅ | adk.dev/sessions, adk.dev/sessions/state |
| 20 | ADK Artifacts: named versioned binary data, `Part`/`inline_data`, ArtifactService implementations, `save_artifact`/`load_artifact`, latest-by-default versioning | ✅ | adk.dev/artifacts |
| 21 | ADK evaluation docs (criteria, custom metrics, user/environment simulation) exist | ✅ | adk.dev/llms.txt |
| 22 | LangSmith: datasets/examples (inputs + reference outputs + metadata), evaluators (human/code/LLM-judge/pairwise), experiments, offline vs online evaluation | ✅ | docs.langchain.com/langsmith/evaluation; evaluation-concepts |
| 23 | LangSmith: datasets versioned per change, taggable (`"prod"`), evaluable via `as_of`; traces exportable to datasets; Engine auto-generates ground truth | ✅ | docs.langchain.com/langsmith/manage-datasets |
| 24 | LangGraph: StateGraph, State/Nodes/Edges, compile, super-steps (Pregel); checkpointers with threads/`thread_id`, StateSnapshot, time travel, interrupts | ✅ | docs.langchain.com/oss/python/langgraph/graph-api; checkpointers |
| 25 | MCP (2025-06-18): tool fields name/title/description/inputSchema/outputSchema/annotations; tools/list, tools/call; content types; `isError`; `-32602` | ✅ | modelcontextprotocol.io/specification/2025-06-18/server/tools |
| 26 | MCP: clients SHOULD log tool usage for audit; human-in-the-loop SHOULD; annotations untrusted unless from trusted servers | ✅ | Same |
| 27 | MCP client config: `claude_desktop_config.json` paths and `mcpServers` map (command/args/env) | ✅ | modelcontextprotocol.io docs, connect-local-servers |
| 28 | MCP dated spec revisions (2025-03-26 through 2026-07-28) and Financial Services interest group charter | ✅ | modelcontextprotocol.io/llms.txt |
| 29 | OpenTelemetry GenAI semconv moved to open-telemetry/semantic-conventions-genai; per-provider span families + agent spans, events, metrics, MCP conventions | ✅ | opentelemetry.io semconv index; repo page |
| 30 | Specific `gen_ai.*` attribute names and values in OTel GenAI conventions | ⚠ | Index-level only (see §12) |
| 31 | NIST AI RMF 1.0 (Jan 26 2023), GenAI Profile NIST-AI-600-1 (Jul 26 2024), revision under White House AI Action Plan, 2026 concept note | ✅ | nist.gov/itl/ai-risk-management-framework |
| 32 | OWASP legacy Top 10 items incl. LLM06/LLM08; project moved to OWASP GenAI Security Project; GenAI LLM Top 10 2026 (Aug 4 2026) | ✅ | owasp.org project archive page |
| 33 | OWASP Agentic-AI-specific threats-and-mitigations document content | ⚠ | Primary site scrape failed (see §12) |
| 34 | ADR pattern is a documented platform convention for agent decisions | ❌ | No primary source found; it is engineering practice (§8.2) |

## 12. What Could Not Be Verified

Research honesty, collected in one place. These items are **not** asserted as facts in this guide — they are either verified at index/name level only, or could not be fetched during the September 2026 research pass:

1. **OTel GenAI attribute-level detail.** The existence and organization of the OpenTelemetry GenAI semantic conventions (provider span families, agent spans, events, metrics, MCP conventions) is verified, but the specific attribute names inside those families (the `gen_ai.*` registry) were not re-fetched. Treat attribute names as unverified until checked against the `semantic-conventions-genai` repo's `model/` YAML sources.
2. **Anthropic Agent Skills file spec.** Anthropic's docs index confirms Agent Skills exist (overview, quickstart, best practices, API usage pages), but the underlying file format specification was not re-fetched this pass; any frontmatter/body convention claims are deliberately omitted.
3. **Anthropic trace/span schema.** A Claude Agent SDK tracing page with an explicit span schema (analogous to OpenAI's) was not located; only transcript/session surfaces and the Claude Code Analytics API (name-level) were verified.
4. **Reasoning-block wire formats and retention policies.** Anthropic's docs index confirms Thinking / Preserved thinking / Extended thinking (legacy) pages exist; their wire formats and any chain-of-thought retention or redaction policies were not verified. No vendor CoT-retention policy is asserted anywhere in this guide.
5. **OpenAI Evals dataset format.** The platform's Evals documentation section is confirmed to exist from its docs navigation; the dataset file format behind it could not be scraped this pass.
6. **ADK event and eval internals.** ADK's Events architecture is verified at the state/session level; the detailed event schema and ADK evaluation artifact formats (criteria/custom metrics pages) were verified only at docs-index level.
7. **OWASP agentic-AI threat catalog.** The legacy OWASP LLM Top 10 (LLM01–LLM10, including Excessive Agency) and the GenAI LLM Top 10 2026 publication are verified from the OWASP project page; the GenAI Security Project's agentic-AI-specific threats-and-mitigations document could not be fetched at its primary site (scraping failed), so its content is not summarized here.
8. **Hash/provenance conventions.** No primary source verified this pass prescribes a content-hashing or artifact-provenance scheme (beyond ADK filename-versioned artifacts and LangSmith dataset versioning); the immutability/hashing guidance in §9.2 is labeled engineering judgment, not platform convention.
9. **General web search availability.** The web-search backend was rate-limited/empty for most of this pass; all verification was done via direct extraction of primary documentation URLs, which is why the citation list (§14) is primary-source-only.

## 13. Glossary

| Term | Meaning (as used in this guide) |
|---|---|
| **Artifact** | Any named object an agentic solution consists of or produces, across the four classes (design, runtime, quality, governance). ⚠ In ADK docs, "Artifacts" specifically means named, versioned binary data (§2.3, §6.3). |
| **Artifact trail** | The linked, ordered set of artifacts of a solution, joinable by trace/session/definition IDs, forming the unit of auditability, debuggability, and reproducibility. |
| **Design-time artifact** | Definition-layer artifact that exists before runs: agent definitions, prompts, workflows, tool/MCP configs, model configs. |
| **Runtime artifact** | Execution-layer artifact produced by runs: traces/spans, plans, tool-call records, state/memory, outputs. |
| **Quality artifact** | Evaluation-layer artifact: datasets, examples, evaluators, experiments, fixtures, golden sets, baselines, guardrail configs. |
| **Governance artifact** | Accountability-layer record: audit logs, decision records, incident/rollback records, compliance reports. |
| **Trace / span** | End-to-end record of a run (trace) composed of timed operation records (spans); OpenAI SDK vocabulary ✅. |
| **Run / thread / session** | LangSmith's traced execution unit and conversation grouping ✅; ADK's conversation record with Events ✅. |
| **Checkpoint** | LangGraph's snapshot of graph state at a super-step, keyed by `thread_id` ✅. |
| **Dataset / example** | LangSmith evaluation unit: a collection of examples, each an input/reference-output pair, versioned per change ✅. |
| **Experiment** | Running an application against a dataset to produce scored results ✅. |
| **Golden answer / golden set** | Reference artifact an LLM grader scores outputs against (Anthropic eval pattern ✅). |
| **Evaluator** | Scoring logic: human review, code rules, LLM-as-judge, pairwise ✅. |
| **Guardrail** | Input/output check attached to an agent (`input_guardrails`/`output_guardrails` ✅), traced as its own span ✅. |
| **Handoff** | Delegation of a conversation to a specialist agent (OpenAI SDK ✅), traced as `handoff_span`. |
| **MCP server config** | Client-side binding of a server name to command/args/env (e.g., `claude_desktop_config.json` `mcpServers`) ✅. |
| **Agent Config** | ADK's YAML agent manifest (`root_agent.yaml`, schema'd) — experimental ✅. |
| **AGENTS.md** | Open-format Markdown instruction file for coding agents; no required fields; closest-file-wins ✅. |
| **Manifest / bundle** | Estate-defined wrapper pinning a definition to its model/tool/knowledge versions (versioning sibling's pattern). |
| **ADR** | Architecture Decision Record; engineering-practice pattern for recording agent decisions (⚠ not a platform format). |
| **Time travel** | LangGraph checkpoint-based replay/fork of prior execution for review or debugging ✅. |
| **ZDR** | Zero Data Retention; OpenAI policy under which SDK tracing is unavailable ✅. |
| **GenAI semantic conventions** | OpenTelemetry's GenAI span/event/metric attribute families, developed in the semantic-conventions-genai repo ✅. |

## 14. References and Further Reading

Primary sources fetched during this pass (all ✅ unless noted in §11–§12):

- OpenAI Agents SDK docs — Tracing: openai.github.io/openai-agents-python/tracing/; Agents: openai.github.io/openai-agents-python/agents/
- OpenAI platform docs — Evals navigation: platform.openai.com/api/docs (docs/guides/evals)
- Anthropic — Building Effective Agents: anthropic.com/engineering/building-effective-agents; Agent SDK overview: platform.claude.com/docs/en/agent-sdk/overview; Define success and build evaluations: platform.claude.com/docs/en/test-and-evaluate/develop-tests; docs index: platform.claude.com/llms.txt
- Google ADK docs — landing and FAQ: adk.dev; Agent Config: adk.dev/agents/config; Artifacts: adk.dev/artifacts; Sessions: adk.dev/sessions; State: adk.dev/sessions/state; docs index: adk.dev/llms.txt
- LangChain/LangSmith/LangGraph — Evaluation: docs.langchain.com/langsmith/evaluation; Manage datasets: docs.langchain.com/langsmith/manage-datasets; Evaluation concepts: docs.langchain.com/langsmith/evaluation-concepts; LangGraph Graph API: docs.langchain.com/oss/python/langgraph/graph-api; Checkpointers: docs.langchain.com/oss/python/langgraph/checkpointers
- Model Context Protocol — spec, server/tools (2025-06-18): modelcontextprotocol.io/specification/2025-06-18/server/tools; connect-local-servers: modelcontextprotocol.io/docs/2026-07-28/develop/connect-local-servers; index: modelcontextprotocol.io/llms.txt
- OpenTelemetry — GenAI semantic conventions index: opentelemetry.io/docs/specs/semconv/gen-ai/; repo: github.com/open-telemetry/semantic-conventions-genai
- AGENTS.md — agents.md
- NIST — AI Risk Management Framework: nist.gov/itl/ai-risk-management-framework
- OWASP — Top 10 for LLM Applications project archive: owasp.org/www-project-top-10-for-large-language-model-applications/; GenAI Security Project: genai.owasp.org (site unreachable to scraping this pass)

Sibling deep-dives in the same repo carry the per-class depth this guide deliberately compresses; the cross-references are inline throughout. Every inventory in this guide is meant to be walked end to end — from the compliance report to the run, from the run to the definition, from the definition back to the evidence — because that walk, and only that walk, is the artifact trail.
