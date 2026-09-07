# Agent Versioning — The Versioned Agent

> **A deep-dive guide to the discipline of versioning, releasing, and rolling back AI agents: what an agent's version really is (prompt + tools + model + parameters + knowledge), how prompt-as-code and artifact registries work, and how eval-gated pipelines promote agent versions through shadow, canary, and production — written for engineering teams that must treat agents as governed, auditable artifacts.**

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank  \
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Agent Platform · MLOps/LLMOps  \
> **Date:** September 2026

> **Companion guides.** This guide is the versioning/release-management deep-dive of the agent-operations cluster. Evaluation *methodology* (metrics, datasets, judge design) lives in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and is cross-referenced, not duplicated; ongoing behavior *drift* (model updates, knowledge staleness, performance decay) in [ai_agent_drift_guide.md](ai_agent_drift_guide.md); agent *architecture and scaffolding* in [agent_scaffolding_guide.md](agent_scaffolding_guide.md); multi-agent *deployment, hypercare, and quality gates* in [multi_agent_banking_guide.md](multi_agent_banking_guide.md); MCP *tooling and tool governance* in [mcp_discovery_guide.md](mcp_discovery_guide.md) and the sibling MCP guides; *context engineering* (skills, AGENTS.md, context bundles) in [context_engineering_guide.md](context_engineering_guide.md); *observability and tracing* in [agentops_guide.md](agentops_guide.md).

> **How to read it.** Sections 1–6 define the discipline and the artifact model (what exactly is versioned: prompts, models, tools, knowledge, and the config that binds them). Sections 7–10 describe the release machine: eval-gated promotion, rollback, environments, and continuous validation. Section 11 is a full worked example at a fictional bank, including a step-by-step rollback drill. Sections 12–15 are the honesty apparatus: claims audit, unverified items, glossary, references. Where a fact could be confirmed only at a secondary source (or not at all this pass), it is flagged **⚠** or **❌** inline and collected in §13 — this guide prefers an honest gap over a fabricated citation.

---

## Table of Contents

1. [The Discipline: Versioning the Whole Agent](#1-the-discipline-versioning-the-whole-agent)
2. [Relationship to Classic Software Versioning](#2-relationship-to-classic-software-versioning)
3. [Artifact Versioning I: Prompts, Prompt-as-Code and Registries](#3-artifact-versioning-i-prompts-prompt-as-code-and-registries)
4. [Artifact Versioning II: Models, Pins and Aliases](#4-artifact-versioning-ii-models-pins-and-aliases)
5. [Artifact Versioning III: Tools, APIs and Knowledge](#5-artifact-versioning-iii-tools-apis-and-knowledge)
6. [Agent-Config-as-Code: Manifests and Bundles](#6-agent-config-as-code-manifests-and-bundles)
7. [The Release Pipeline: Eval-Gated Promotion](#7-the-release-pipeline-eval-gated-promotion)
8. [Rollback: Strategies and Drives](#8-rollback-strategies-and-drives)
9. [Environments: Dev, Stage and Prod for Agents](#9-environments-dev-stage-and-prod-for-agents)
10. [Drift, Regression and Continuous Validation](#10-drift-regression-and-continuous-validation)
11. [Banking Context and Worked Example: The Cymbal Bank Customer-Service Agent](#11-banking-context-and-worked-example-the-cymbal-bank-customer-service-agent)
12. [Claims Audit](#12-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [References and Further Reading](#15-references-and-further-reading)

---

## 1. The Discipline: Versioning the Whole Agent

### 1.1 Why an Agent Is Harder to Version Than a Service

A conventional service is versioned as *code plus configuration*, and its behavior is a deterministic function of that code. An agent is versioned as **code plus configuration plus artifacts that live outside the codebase and change without a deploy**: the system prompt, the tool schemas it may call, the model (and model version) it runs on, sampling parameters, and the knowledge it retrieves. Each of those artifacts can change behavior dramatically while the code stays byte-identical.

LangChain's documentation for its Prompt & Context Hub makes the underlying observation explicit: prompts, retrieval context, skills, and task instructions *"change more often than the application code around them,"* and are often edited by people who are not engineers ✅ (LangChain docs, Prompt & Context Hub). That asymmetry — volatile, non-code artifacts wrapped in relatively stable code — is the entire reason agent versioning is a discipline of its own rather than an afterthought of application versioning.

Because an agent is stochastic (temperature, sampling) and *tool-calling* (it mutates the outside world through APIs), two "identical" runs can differ, and a version's identity must therefore be captured as a **snapshot of everything that influences behavior**, not just a git SHA of the orchestrator.

### 1.2 The Versioned Artifacts

An agent version is a tuple over five artifact families:

| Artifact family | What is versioned | Typical storage | Changes without code deploy? |
|---|---|---|---|
| **System prompt / instructions** | System prompt text, persona, guardrails prose, few-shot exemplars, output contracts | Prompt registry, repo, Context Hub | ✅ Yes — the norm |
| **Tools** | Tool schemas, MCP server configs, function implementations, tool versions, permissions | Repo, MCP registry, tool catalog | Partially (schema/config can be runtime-resolved) |
| **Model** | Provider, model name, dated version/pin or alias, endpoint | Config, model registry | ✅ Yes — provider pushes new versions |
| **Parameters** | Temperature, top_p, max tokens, reasoning effort, stop sequences, caching flags | Config, prompt record | Yes |
| **Knowledge** | Retrieval corpus version, embedding model + collection, chunking config, RAG filters, memory | Vector DB, dataset registry | ✅ Yes — knowledge is updated continuously |

The model row deserves emphasis: **model versioning is the only family where the vendor, not you, owns the artifact**. Your agent pins or aliases to a model; the provider retires and replaces models on *its* schedule. Anthropic's model lifecycle documentation defines four states — **Active, Legacy, Deprecated, Retired** — and states plainly that *requests to retired models will fail*, while committing to at least 60 days' notice before retirement for publicly released models ✅ (Anthropic docs, Model deprecations). That single fact converts "model pin" from a convenience into a compliance obligation: an unpinned or unmonitored agent can break silently on a date the vendor chose.

### 1.3 Why Version Agents at All: The Four Drives

**Auditability.** In regulated contexts every customer-facing behavior must be reproducible as *"which exact prompt + model + tools produced this answer?"* — version IDs on traces are what make that query answerable. (Trace schema and lineage hooks are covered in [agentops_guide.md](agentops_guide.md).)

**Reproducibility.** Eval results are only meaningful against a fixed artifact set. A regression discovered in production is undiagnosable if you cannot reconstruct the exact prompt and model that produced it.

**Rollback.** The single most practical drive: when a release degrades, the fix must be *re-point a pointer*, not *redeploy code*. LangSmith's prompt management documents environments with an *ordered history of which commits were assigned to them*, and a rollback flow that re-points the environment at a previous commit ✅ (LangChain docs, Manage prompts). Rollback is cheap exactly when artifacts are versioned immutably (see §8).

**Eval-gated promotion.** Because agent changes are cheap to make and hard to predict, the only safe path to production is a gate chain where each candidate version must clear offline evals, shadow comparison, and canary traffic before full rollout (§7). This is the discipline that separates agent *experimentation* from agent *releases*.

### 1.4 The State of Conventions: Real but Unsettled ⚠

The vocabulary of agent versioning is converging — registries, commits, environment tags, promotion, rollback — and multiple platform vendors implement it (LangSmith, Braintrust, Azure/MLflow-style model registries, Google's Agent Registry). What is *not* settled:

- There is **no single semantic-versioning standard for prompt or agent bundles** (see §2 and §13). ⚠
- Terminology differs: LangSmith speaks of *commits, tags, environments*; Braintrust of prompt *versions* plus deployment *environments*; Azure of *model versions and aliases*; Google of *registry entries and skill revisions*. The concepts rhyme; the nouns do not. ⚠
- Whether the **model version** should be part of the agent's semantic version, or tracked separately as an external dependency, is a live design decision with no consensus (this guide recommends treating it as a pinned dependency with its own lifecycle — §4). ⚠
- Whether **rollback is ever truly complete for a state-changing agent** is unsettled: registry re-points are clean and instant, but undoing what the bad version already did (messages sent, records updated) requires reconciliation machinery that no vendor registry ships as part of the pointer (§8.4). ⚠
- Whether a prompt record **may legitimately sit in several environments at once** is answered differently across vendors — LangSmith explicitly permits staging+production overlap ✅, but the *semantics* of multi-environment membership (one promotion or two?) are not standardized, so cross-vendor promotion behavior is not interchangeable. ⚠

None of this means the discipline is immature — it means teams should adopt a *local* convention and enforce it mechanically, which is the subject of the rest of the guide.

### 1.5 The Versioned Artifact Inventory: Granularity in Practice

To make the §1.2 tuple concrete, a team building its inventory must decide, per artifact family, what a version record captures and at what granularity a change becomes a release event:

| Artifact family | What a version record captures | Example granularity that matters | Typical change owner |
|---|---|---|---|
| **System prompt / instructions** | Full text at commit (persona, guardrails prose, few-shot exemplars, output contracts), registry commit hash | One edited sentence is a new commit; wording-level diffs are release events, not chores | Domain experts + engineers (registry-hosted; promotion permissioned, §3.2) |
| **Tools** | Schema/definition version, MCP server version, implementation SHA, permission set | Schema change is model-visible (major-flavored bundle change); implementation-only change alters behavior with no prompt diff (§5.1) | Tool/platform team |
| **Model** | Provider, dated snapshot ID, serving platform (lifecycle can differ per platform, §4.1) | Snapshot bump or alias move is a release event that re-enters the gate chain | Vendor authors it; platform team pins and monitors it |
| **Parameters** | temperature, top_p, max tokens, reasoning effort, stop sequences, caching flags, deprecation status | A parameter deprecation can break a pinned config while the pinned model still lives (§4.3) | Engineer (config record) |
| **Knowledge** | Corpus/dataset version, embedding model + version, index name, chunking config, memory revision | Re-embedding the same corpus is a new knowledge version; memory revisions are inspectable (§5.3) | Content/knowledge team — changes continuously |

The granularity column is the operational point: **a version boundary belongs wherever a change can alter behavior without a code deploy** — if it can, it gets its own identifier in the inventory, and that identifier lands in the manifest (§6).

## 2. Relationship to Classic Software Versioning

### 2.1 What Maps Cleanly

| Classic concept | Agent analog | Maps? |
|---|---|---|
| Source control (git) | Prompt/tool/config files in repo; git SHA as the code part of a version | ✅ Cleanly |
| Artifact registry (Nexus/Artifactory) | Prompt registry, model registry, Context Hub, Agent Registry | ✅ Cleanly |
| Semantic versioning of libraries | Versioning of *stable, testable* prompt bundles | ⚠ Partially |
| Environments (dev/stage/prod) | Environment tags + promotion flows (§9) | ✅ Cleanly |
| Immutable release artifacts | Immutable prompt commits / registry entries that are never mutated in place | ✅ Cleanly (vendor-enforced) |
| CI gate chain | Eval-gated promotion (§7) | ✅ Conceptually, new machinery |
| Rollback = redeploy previous artifact | Rollback = re-point environment pointer to previous immutable version (§8) | ✅ Cleanly — *easier* than classic |
| Traffic splitting (canary) | Shadow + canary traffic for agent versions | ⚠ Possible but less standard |

The columns that map *cleanly* all share one property: they are **pointer mechanics** — moving a named reference from one immutable object to another. Registries, environments, and rollback are exactly this, and vendors have implemented them well.

### 2.2 What Does Not Map

**Semantic versioning semantics.** SemVer's contract (MAJOR = breaking change, MINOR = backward-compatible feature, PATCH = bug fix) assumes a *caller-visible API contract*. A prompt change is not an API change — it is a *behavioral* change whose blast radius is unknowable without evaluation. Two prompt edits can both be "PATCH-level" in diff size and one can silently break a workflow that the other improves. ⚠ There are community proposals to apply SemVer to prompts (see §13), but no dominant standard; treat MAJOR/MINOR/PATCH on an agent bundle as *release policy*, not as a proven compatibility contract.

**Dependency resolution.** Classic systems resolve dependencies at build time and lock them. Agents resolve their most behavior-critical "dependencies" — the model, the system prompt — **at runtime**, often by mutable alias (e.g., a `:production` tag, or a model name that the vendor re-points). The discipline of *locking* is only as strong as your resolve to pin and your monitoring of the pin (see §4 and §10).

**Deterministic verification.** You cannot "unit test" a prompt the way you test a function. The equivalent — offline evaluation on a fixed dataset — is probabilistic and dataset-dependent. Classic CI asserts; agent CI *scores*. This shifts the entire release question from "does it build?" to "does it score above threshold, and did we regress anything?" (methodology in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md)).

**Roll-forward culture.** Classic systems often prefer rolling forward (fix on main, redeploy). Agent changes are cheap enough that roll-forward is tempting, but a bad prompt/model change can be *harmful at scale within minutes* (a compliance breach or a bad financial answer), and the fix (an edit + full re-eval) is slower than re-pointing to a known-good version. Banking agents must treat **rollback as the default first response**, roll-forward as the follow-up (§8).

### 2.3 The Useful Framing: Version = Behavior Contract, Not API Contract

The pragmatic model used throughout this guide: an **agent version is a named, immutable bundle of artifacts whose behavior contract is certified by evaluation**, not by type signatures. Version numbers on the bundle track *release history*; the *compatibility question* ("is v2 safe where v1 was?") is answered by the eval gate chain, never assumed from the version delta. This is the mindset shift the rest of the guide operationalizes.

## 3. Artifact Versioning I: Prompts, Prompt-as-Code and Registries

### 3.1 Two Schools: Prompt-in-Repo vs Prompt-in-Registry

**Prompt-as-code** means the prompt's source of truth is a text file in your repository (often a `.prompt`/`.md`/templated file), reviewed and merged through normal pull requests, then *pushed* to a runtime registry at release time. Braintrust's documentation shows the pattern directly: you define a prompt in a code file (`project.prompts.create(...)` with name, slug, model, params, messages) and push it to the platform with a CLI command — `bt functions push summarizer.py` ✅ (Braintrust docs, Create prompts). GitHub review, git blame, and branch protection apply; the registry is the deployment target, not the source of truth.

**Prompt-in-registry** means the prompt lives (and is edited) in the platform UI, and the registry is the source of truth — LangSmith's stated rationale is that non-engineers (domain experts) own the context and can change behavior *without a full deploy* ✅ (LangChain docs, Prompt & Context Hub). The two schools answer the same governance question differently: *who may change prompts, and through which review mechanism?* Banking practice (this guide's §11) usually wants a hybrid: registry-hosted prompts, but promotion permissions restricted to owners, plus registry-to-repo export for audit.

### 3.2 How Registries Version Prompts: Commits, Tags, Environments

The vendor implementations converge on a small set of primitives:

- **Commit**: one immutable snapshot of a prompt. LangSmith records every saved change as a new commit in a per-prompt history you can browse, diff, compare, and revert ✅ (LangChain docs, Use the Context Hub).
- **Tag**: a mutable label pointing at exactly one commit. LangSmith commit tags let code reference `name:my-tag` instead of a commit hash, so the version in use changes without a code change; each tag references exactly one commit and can be re-pointed ✅ (LangChain docs, Manage prompts).
- **Environment**: a reserved, managed tag. LangSmith reserves `staging` and `production` as environment tags handled through a promotion flow, and promoting a commit to an environment updates the environment pointer immediately ✅ (LangChain docs, Manage prompts). The environment keeps an ordered history of the commits assigned to it — which is precisely what makes rollback a one-click re-point ✅ (same source).
- **Code-time resolution**: applications fetch prompts by environment, e.g. `client.pull_prompt("joke-generator:production")`, which resolves to whatever commit the pointer currently names ✅ (LangChain docs, Manage prompts). This is the canonical "change behavior without a deploy" mechanism.
- **Stable slug, mutable content**: Braintrust separates a prompt's **slug** (its unique code-reference name, constant across updates) from its changing content/version, and explicitly supports reverting to a previous version within the UI when the slug is kept ✅ (Braintrust docs, Create prompts and Prompt versioning cookbook).
- **Every change is a new commit — nothing is edited in place**: a saved change appends a commit to the per-prompt history, so "what changed when" is a query over append-only history rather than a reconstruction from memory ✅ (Manage prompts; Use the Context Hub).
- **Diffing is the review primitive**: comparing two commits — or the environment's current commit against a candidate — is how a reviewer sees exactly what a release would change; LangSmith's per-prompt browse/diff/compare UI and Braintrust's per-version history both expose that surface ✅ (sources above).
- **Commit messages carry the human record**: treating each registry commit like a git commit — who, why, linked ticket — is what turns "why did this wording land?" into a query later. The registries store the history; the *message discipline* is the team's. ⚠ (Message conventions are team practice, not a vendor-enforced field this pass.)
- **Promotion between environments moves a pointer, not content**: promoting a commit to `staging` or `production` leaves the commit object untouched and re-points only the environment label — which is why one commit can serve staging and production at the same time (§9.1) and why rollback is the same operation in reverse (§8.1) ✅ (Manage prompts; Use the Context Hub).

**Access control is part of versioning.** LangSmith's *prompt owners* mode restricts who may create/move tags and promote commits to environments — i.e., promotion becomes a permissioned act rather than an editor's whim ✅ (LangChain docs, Manage prompts). For a bank this is a control point, not a feature.

### 3.3 Beyond Single Prompts: Versioned Context Bundles

The current frontier moves versioning from *one prompt* to *a bundle of instructions and tools that defines a skill or a whole agent*. LangSmith's **Context Hub** models exactly this: a *context* is a versioned bundle — an **Agent** context includes an `AGENTS.md` file plus tools; a **Skill** context includes a `SKILL.md`; contexts have commit history and are promoted to `staging`/`production` environments that agent runtimes resolve by tag (e.g. `:production`) ✅ (LangChain docs, Use the Context Hub). This is versioning at the granularity where agents actually change in 2026: not "edit the system prompt" but "promote a new revision of the agent's entire instruction-and-tool bundle." Full context-engineering mechanics (skills, AGENTS.md structure, retrieval context) are covered in [context_engineering_guide.md](context_engineering_guide.md); this section only claims the versioning shape: **commit — evaluate — tag — promote — rollback**, applied to bundles.

### 3.4 Webhooks and the Prompt Pipeline

Because prompt changes are releases, registries expose commit webhooks so that a prompt commit can trigger CI/CD, sync back to a GitHub repository, or notify stakeholders ✅ (LangChain docs, Manage prompts — a documented use case is *"triggering a CI/CD pipeline when prompts are updated"*). The pipeline this enables — every prompt commit enters the eval gate chain before it can be promoted to production — is the subject of §7.

## 4. Artifact Versioning II: Models, Pins and Aliases

### 4.1 The Model Lifecycle Is a Vendor Contract

Models are the artifact family your team does **not** author — so model versioning is really *vendor-lifecycle management*. The reference vocabulary comes from Anthropic's model deprecations documentation ✅:

| Lifecycle state | Meaning (Anthropic's definition) |
|---|---|
| **Active** | Fully supported and recommended for use |
| **Legacy** | Will no longer receive updates; may be deprecated in the future |
| **Deprecated** | Still functional but not recommended; replacement named; retirement date assigned |
| **Retired** | No longer available; requests to retired models **fail** |

Verified specifics from the same primary source ✅:

- Anthropic commits to **at least 60 days' notice before retirement** of publicly released models, by email and in documentation.
- **Requests to models past the retirement date fail** — a hard break, not a soft degradation.
- Anthropic notifies customers with *active deployments* of models with upcoming retirements — but the burden of migrating before the date sits with the deployer.
- Partner-operated platforms (**Amazon Bedrock and Google Vertex AI set their own retirement schedules**), so the same model can have different lifecycle status and dates depending on where it is served. ✅ (Anthropic docs: "Partner-operated platforms ... set their own retirement schedules, so a model's lifecycle status and dates can differ.")

The operational translation: your registry's model pin must include **both** the provider's dated snapshot ID (e.g. `claude-sonnet-4-5-20250929`, the dated form that appears in Braintrust's own prompt examples ✅) **and** a monitoring hook on the lifecycle state of that ID. A rollback that re-points to a version pinned to a *retired* model will fail at runtime — pin provenance must include "still servable" checks (§8).

### 4.2 Aliases vs Pins

- **Pin (snapshot)**: an exact dated model version, e.g. `claude-sonnet-4-5-20250929`. Deterministic; the artifact you certify in evals is the artifact that serves traffic. Breaks only when the vendor retires it.
- **Alias (floating)**: a mutable name the vendor re-points, e.g. an unversioned `claude-sonnet-4-5` or an environment tag like `:production` on a prompt. Convenient for low-stakes use; for regulated agents an alias should be **resolved and frozen into the release manifest at promotion time** so the certified bundle names a concrete snapshot.

This guide's recommendation: **certify on pins, serve on pins, and treat alias moves as change events** that re-enter the eval gate chain (§7), because a silent alias bump is a silent behavior change. ⚠ The precise alias mechanics differ per provider (OpenAI's docs could not be scraped this pass — see §13), so treat provider-specific alias semantics as an integration detail to verify per vendor.

### 4.3 Parameters Age Too

Sampling parameters are part of the version, and vendors deprecate *parameters* as well as models. Anthropic's deprecations page records that `temperature`, `top_p`, and `top_k` are deprecated on newer Claude models, that setting them to non-default values on affected models returns a **400 error**, and that the Python SDK (v1.0+) removes them so passing them raises `TypeError` ✅. Two consequences for versioning: (1) a pinned config can break even when the pinned model still exists, and (2) parameter deprecation is a signal that the *prompting contract* (instructions instead of sampling knobs) is the supported behavior surface. Reasoning-model parameter surfaces are equally volatile — Braintrust documents that temperature is only exposed at certain reasoning-effort settings for newer GPT models ✅ — which argues for treating the **parameter block as an explicit, diffable part of the prompt record**, not an incidental code constant.

### 4.4 Model Registries: Azure Machine Learning Pattern

Where models are first-party (fine-tunes, embeddings, custom classifiers), a model registry supplies the version mechanics. Azure Machine Learning's documentation describes the pattern ✅:

- **Registration** stores and versions models in a workspace; the registry "helps you organize and keep track of your trained models."
- Model assets are referenced by explicit version: `azureml:<model-name>:<version>` within a workspace, and across workspaces via a registry path `azureml://registries/<registry-name>/models/<model-name>/versions/<version>`.
- **Archiving** a version hides it from default list queries while leaving it referenceable and usable — a "soft delete" that suits governance (withdraw from *new* use, keep for *existing* pinned deployments).
- Tags such as `stage="Prod"` are used to mark lifecycle stage on versions.

The same pointer discipline appears everywhere: *name + explicit version* for determinism, *tags/aliases* for mutable stage references, *archive* instead of delete for governed artifacts.

### 4.5 The Model Pin Record: What a Registry Row Must Hold

For the compliance obligation of §1.2 and the servability rule of §8.3, the model row in your registry needs more than a name:

| Field | Purpose | Example |
|---|---|---|
| Provider + snapshot ID | Deterministic identity of the certified artifact | `claude-sonnet-4-5-20250929` (Anthropic) |
| Serving platform | Lifecycle status and dates can differ per platform (§4.1) | Direct API vs Bedrock vs Vertex AI |
| Lifecycle state + source | Active / Legacy / Deprecated / Retired per the vendor page (§4.1) | Active (vendor lifecycle page, checked 2026-09) |
| Retirement date (if any) | Feeds the migration calendar and re-certification triggers (§10.3) | Notice window ≥ 60 days per vendor policy |
| Aliases that resolve to it | Which mutable names may re-point here; alias moves are change events (§4.2) | `:production` tag; `claude-sonnet-4-6` (to be frozen at promotion) |
| Parameter compatibility | Deprecation status of sampling parameters on *this* snapshot (§4.3) | temperature accepted here; manifest pins temp 0 |

The row is a monitoring object, not a filing-cabinet entry: its lifecycle field is re-checked on a schedule and on every vendor notice, and any state change fires the re-certification triggers of §10.3 — which is how a "model pin" stays a pin instead of quietly becoming a liability.

## 5. Artifact Versioning III: Tools, APIs and Knowledge

### 5.1 Tools and MCP Servers (Condensed — Cross-Ref)

Tool versioning matters because the tool surface *is* the agent's blast radius: a changed tool schema changes what the model can do, and a changed tool implementation changes what happens when it does. Three layers need version identity:

1. **Schema/definition** — the JSON schema or MCP tool definition the model sees. A schema change is a prompt-visible change and should ride the same gate chain as a prompt change.
2. **Implementation** — the function or MCP server behind the schema. This is ordinary software and versions in git/deployments like any service.
3. **Governance surface** — which tools an agent may call, at what permission level.

MCP tooling, discovery, and governance mechanics are covered in depth by [mcp_discovery_guide.md](mcp_discovery_guide.md) and the sibling MCP guides; this guide only states the versioning rule: **pin tool manifests per agent version**, and treat a tool manifest change as a minor-or-major bundle change depending on whether schemas (model-visible) or only implementations (opaque to the model, but behavior-changing) moved. ⚠ There is no widely standardized "tool manifest version" schema yet; teams typically pin by git SHA of the tool catalog plus MCP server version.

### 5.2 Registry-Level Tool and Agent Discovery: Google Cloud Agent Registry

Google Cloud's **Agent Registry** shows the emerging pattern for governing *agent-to-agent* surfaces (verified ✅):

- Registering an agent makes it a **discoverable component**; registration is either **automatic** (supported runtimes — e.g. agents deployed via the Agent Runtime SDK on Gemini Enterprise Agent Platform, GKE containers with a functional-type label, Cloud Run services with a functional-type flag — register within the same project by default) or **manual** (needed for cross-project registration, e.g. spoke agents governed through a central project's Agent Gateway).
- The registry is aligned with the **Agent2Agent (A2A) protocol**, supporting A2A spec versions 0.3 and 1.0; an agent's **Agent Card** declares its capabilities and identity, and the registry extracts A2A skills from the card for agents implementing A2A.
- **Standalone skills** can be registered as top-level `Skill` resources with version history managed via **skill revisions** — i.e., versioned, governable capability units.

The versioning reading: when agents call other agents or consume shared skills, the *callee's version and the skill revision* become part of the caller's dependency surface, exactly as a tool version does. Registry entries plus revision history are how the platform makes that dependency explicit and auditable.

### 5.3 Knowledge: The Quietly Dangerous Artifact

Knowledge (retrieval corpora, embeddings, memory) changes continuously and is the artifact most likely to degrade an agent *without anyone editing the agent*. Versioning it means snapshotting, per agent version:

- **Corpus version** — which document set was indexed (a dataset/catalog version, not just "the vector DB").
- **Index identity** — embedding model + version, collection/index name, chunking parameters; re-embedding the same corpus with a new embedding model is a new knowledge version.
- **Runtime knowledge** — the persistent memory the agent may read/write. Notably, Gemini Enterprise Agent Platform's Memory Bank exposes explicit **memory revisions** that can be inspected ✅ (Google Cloud docs, Agent Platform scale page), acknowledging that even learned/accumulated knowledge needs revision history for auditability.

Cross-reference: knowledge freshness, staleness detection, and knowledge-drift monitoring are covered in [ai_agent_drift_guide.md](ai_agent_drift_guide.md). The versioning rule here is short: **every agent release manifest should record the corpus, index, and memory-revision IDs it is certified against**, or the eval certificate is meaningless.

Concretely, the knowledge record in a release manifest names three identifiers, each pointing at an immutable object:

- **Corpus** — the dataset/catalog revision that was indexed, not the live database's current state: `product-policies-v14`. A document edited after indexing belongs to a *newer* corpus version, which is why the §11.2 table tracks `policies-v13`/`v14` as distinct rows rather than one mutable store.
- **Index** — `embeddings-v3`: embedding model + version, collection/index name, chunking config. Re-embedding the same corpus with a new embedding model produces a new index ID even when the corpus is untouched (§1.2, §1.5).
- **Runtime memory** — a memory-revision ID where the platform exposes one (Gemini Enterprise Agent Platform's inspectable memory revisions, §5.3 ✅); otherwise `off`/`none` written explicitly rather than assumed.

Reading the §6.1 manifest's `knowledge: { corpus: product-policies-v14, index: embeddings-v3, memory: off }` — or §11.2's shorthand `policies-v14, idx-v3` — is exactly this triple. Teams without a memory-revision primitive should still write the field, because "no memory" is itself a fact that a release certifies.

## 6. Agent-Config-as-Code: Manifests and Bundles

### 6.1 The Manifest: One File to Name the Whole Agent

All five artifact families only become *one version* when a manifest binds them. An agent release manifest (a YAML/JSON file in the repo, or a registry bundle) declares, for one agent version:

```yaml
# cyb-cs-agent manifest — illustrative, see §11 for the full worked example
apiVersion: agent.cyb/v1
agent: cs-agent
version: 1.5.0            # bundle version (release policy, not SemVer contract)
artifacts:
  prompt: cyb-cs-intake:production    # registry name + environment tag (resolved to a commit hash at promote time)
  model: claude-sonnet-4-5-20250929   # dated pin — never an unversioned alias for prod
  params: { temperature: 0, max_tokens: 900 }
  tools: [core-banking-mcp@2.3.1, fraud-screen-mcp@1.1.0]
  knowledge: { corpus: product-policies-v14, index: embeddings-v3, memory: off }
gates:                   # eval certificate recorded at promotion (§7)
  offline: { suite: reg-2026-09, pass: true, score: 0.97 }
  shadow:   { duration: 24h, agreement: 0.99, regressions: 0 }
  canary:   { pct: 5, duration: 24h, errors: 0 }
promoted: { env: production, at: "2026-09-07T09:00:00Z", by: platform-svc }
```

Why a manifest earns its keep: it makes the *certified bundle* a reviewable, diffable, signed object. Code review now operates on the question "what changed in this agent release?" as a single diff; the eval gate chain stamps its verdict onto the manifest; and rollback is *"promote the previous manifest,"* which re-establishes every artifact pointer at once.

A manifest diff is the unit of code review for agent releases; a reviewer checks, in order:

- **Alias resolution** — every artifact named by alias or environment tag (`:production`, `claude-sonnet-4-6`) is resolved to a concrete immutable ID in the *promoted* copy of the manifest, so the certified bundle names snapshots, not pointers-to-pointers (§4.2).
- **Certificate completeness** — the `gates` block carries suite/dataset version, per-metric scores, regression deltas, judge versions, and timestamps; a manifest whose gate block predates its artifact pins is a red flag (§7.2).
- **Knowledge presence** — corpus, index, and memory-revision IDs are present and current; a release that omits knowledge IDs certifies nothing about retrieval behavior (§5.3).
- **Tool + permission pinning** — tool versions *and* the permission sets they imply are explicit, so environment deltas in credentials cannot silently change behavior (§9.3).
- **Signed and diffable** — the released manifest is checksummed and immutable (§6.3), and the promotion record (`promoted:` block) names the actor, time, and prior pointer.

### 6.2 Registry-Native Manifest Shapes

Platforms increasingly express the manifest as a first-class registry object rather than a hand-rolled file:

- **LangSmith Context Hub Agent contexts**: an Agent context is literally a versioned bundle containing `AGENTS.md` plus its tools, committed and promoted as a unit ✅ (LangChain docs, Use the Context Hub).
- **Braintrust prompt records**: a prompt object bundles messages *plus* model *plus* parameters *plus* response format under one slug/version ✅ (Braintrust docs, Create prompts) — the prompt record is a mini-manifest for the non-code half of the call.
- **Microsoft Foundry / prompt flow**: a flow is defined declaratively (`flow.dag.yaml`) with its dependencies (`requirements.txt`) and deployed from the flow or a bulk test run as an online deployment hosted in an endpoint ✅ (Microsoft Learn, Deploy a flow). Flow-as-config, endpoint as runtime, deployment as version instance.
- **LangSmith Deployment assistants**: on the deployed runtime, an **assistant** is the configuration object (of a graph) that a client invokes, alongside threads (state) and runs (workloads) ✅ (LangChain docs, LangSmith Deployment). Assistants are the runtime's handle for "this particular configured version of the agent."

The through-line: **config objects are versioned entities with their own lifecycle**, and every major runtime now has one. The bank's job is to make its manifest *the* audit record that ties registry state, model pin, tool pins, and eval certificate together, because no single vendor object spans all five artifact families today. ⚠

### 6.3 Bundles Must Be Immutable

Immutability is the load-bearing wall. Every registry cited in §3–§6 versions by *append-only commits*: a commit, once created, is not edited; change = new commit; promotion = pointer move ✅ (LangSmith commits; Braintrust version history). Enforce the same discipline on your own manifests: store them content-addressed (or at least checksummed) in the repo, never mutate a released manifest, and make the *pointer* (environment tag / alias / promotion record) the only mutable thing. Everything in §7–§9 — gating, environments, rollback — is pointer mechanics over immutable bundles.

## 7. The Release Pipeline: Eval-Gated Promotion

### 7.1 The Gate Chain

The release pipeline promotes a candidate agent version through escalating confidence stages, each with an entry condition (gate) that must pass before promotion:

```
edit → commit → offline eval → shadow (parallel, no customer impact)
     → canary (small real traffic) → production (full traffic) → hypercare
        gate 1            gate 2                  gate 3
```

- **Gate 1 — offline eval**: the candidate (or its diff) runs against a fixed, versioned eval dataset with threshold scores and a no-regression rule against the incumbent version. This is the primary gate and the most rigorous one; methodology, dataset hygiene, judge design, and metric choice are deliberately not re-derived here — see [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md).
- **Gate 2 — shadow**: the candidate runs in parallel on real (copied) production traffic; its outputs are compared to the incumbent's *without serving customers*. Agreement metrics, cost/latency deltas, and failure-mode diffs are computed.
- **Gate 3 — canary**: a small slice of real traffic (e.g. 5%) is served by the candidate while the incumbent serves the rest; errors, escalations, and evals-on-live-traffic are watched before 100%.

Vendor support for the pieces is verifiable, though no vendor brands the whole chain identically (⚠ on the unified vocabulary; ✅ on the parts):

- LangSmith's model is a **commit → promote** flow through `staging` then `production` environment tags, with the playground as the experimentation surface ✅ (Manage prompts; Use the Context Hub).
- Braintrust's documented prompt-versioning loop is **create version → run evaluation per version (experiment_name per eval) → compare scores → revert to the best-scoring version** ✅ (Prompt versioning cookbook), and prompts are versioned/deployed *independently of code*, with environments as rollout targets ✅ (Create prompts).
- Google's Gemini Enterprise Agent Platform explicitly frames **testing and release management** as part of its managed runtime, pairing an **Evaluation Service and Example Store** for continuous quality feedback with deployment, tracing, logging, and monitoring ✅ (Vertex AI docs, Scale your agents).
- Microsoft Foundry/classic prompt flow formalizes the "**test the flow, then deploy it from the flow or from a bulk test run**" sequence into an online endpoint ✅ (Microsoft Learn, Deploy a flow).

The stages, with realistic entry criteria (a team should make these explicit numbers in its own runbook, not leave them as vibes):

| Stage | What runs | Entry criteria (examples, banking-grade) | Exit artifact |
|---|---|---|---|
| **Edit → commit** | Author changes prompt/context/tool pin; commit lands in registry | None (authoring is unblocked) | New immutable commit + optional CI webhook |
| **G1 · Offline eval** | Candidate vs fixed versioned suite (offline, no traffic) | Overall pass ≥ 0.95; per-metric ≥ 0.90; no metric > 0.02 below incumbent; refusal/safety suite = 0 failures; compliance subset signed off | Eval certificate (§7.2) |
| **G2 · Shadow** | Candidate on copied live traffic beside incumbent, 24–48 h | Agreement ≥ 0.98 with incumbent; 0 critical failure modes; cost delta ≤ 5%; p95 latency delta ≤ 10% | Shadow report + frozen incumbent IDs |
| **G3 · Canary** | 5% real traffic, 24 h, incumbent on 95% | 0 hard errors; escalation rate ≤ incumbent; live-eval score ≥ offline score − 0.02 | Canary report + promotion record |
| **Hypercare** | 100% traffic, heightened monitoring (window defined per risk) | No threshold breaches; escalations normal | Release closed; version marked known-good |

Thresholds are *comparative and absolute at once*: absolute floors catch general weakness, comparative rules (vs incumbent) catch regressions that absolute scores mask — a candidate can score 0.97 absolutely and still be 0.06 worse than an incumbent at 0.99 on the population that matters. Teams commonly under-weight the comparative rule; the incident in §11 fails precisely because of it (blended absolute score passed while a subpopulation regressed).

A complementary reading maps each stage to the **failure class it exists to catch** and the named criteria that catch it — the runbook view of the same chain:

| Stage | Primary failure classes caught | Representative gate criteria | Human checkpoint in the chain |
|---|---|---|---|
| **G1 · Offline eval** | Hallucinated policy/entitlement answers; anthropomorphic or unsafe phrasing; subpopulation regression | Eval pass rate ≥ 0.95; per-metric ≥ 0.90; refusal/safety suite = 0 failures; no metric > 0.02 below incumbent; hallucination and anthropomorphic-safety subsets scored by the judge | Compliance sign-off on regulated-answer subsets before the candidate leaves offline (the `comp-0904` pattern, §11.3) |
| **G2 · Shadow** | Silent behavior divergence on real traffic; cost/latency surprise | Agreement ≥ 0.98 with incumbent; 0 critical failure modes; cost delta ≤ 5%; p95 latency delta ≤ 10% | Engineering review of the shadow report before canary traffic is opened |
| **G3 · Canary** | Live error-rate SLO breaches; escalation drift; offline-overfit scores that do not hold on live traffic | 0 hard errors; error-rate SLO within budget; escalation rate ≤ incumbent; live-eval ≥ offline − 0.02 | Named human approver authorizes 100% after the canary report (promotion is a permissioned act, §3.2) |
| **Prod entry → hypercare** | Post-100% surprises from model/knowledge/tool drift (§10) | No threshold breaches in the heightened-monitoring window; escalations normal | Release owner closes the release and marks the version known-good |

Read top to bottom, the chain is *gates and approvals together*: every criterion is a threshold a named person consciously releases — the pipeline automates the measurement, not the judgment.

### 7.2 The Eval Certificate: What a Gate Actually Stamps

Each passing gate should produce a machine-readable verdict attached to the version — in the manifest (§6) and in the trace metadata. Minimum content: suite/dataset version, scores (per metric), regression diff vs incumbent, judge/model versions used for grading, timestamp, and the identity of the gate runner. Without the certificate, a later "was this version cleared for prod?" question is unanswerable; with it, audit is a lookup. Because LLM-as-judge grading is itself model-dependent, recording the *grader's* version is as important as recording the candidate's. (Grader design: [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md).)

Minimum certificate fields, each verifiable later as a lookup:

| Field | Records | Why audit needs it |
|---|---|---|
| Suite/dataset version | Which eval set and dataset revision ran | A score is meaningless without its distribution; dataset rot (§7.3) is diagnosed only if the set is versioned |
| Per-metric scores | Score per metric, not just blended | Blended scores hide subpopulation regressions — the §11 failure mode |
| Regression delta vs incumbent | Per-metric difference against the frozen incumbent IDs | Answers "is this *better*, or merely above floor?" |
| Judge model + version | Grader identity for every LLM-judged metric | Judge upgrades change scores; certificates must stay comparable across judge versions |
| Gate runner + timestamp | Who/what ran the gate, and when | Promotion records must be attributable (§7.1; §12 rows 3–5) |
| Candidate + incumbent artifact snapshot | The exact pins the verdict applies to | Links the certificate to one immutable bundle (§6.1) |

### 7.3 What Can Break the Chain

- **Dataset rot**: the eval set drifts from production reality; scores inflate while live quality sags. Mitigation: periodic dataset refresh from sampled production logs (Braintrust's cookbook explicitly recommends *"real customer interactions from your logs"* for the dataset ✅), plus live-trace eval in shadow/canary.
- **Incumbent movement**: if the incumbent version itself changes underneath (alias bump, knowledge refresh), the "no regression vs incumbent" comparison is apples-to-oranges. Mitigation: freeze the incumbent's artifact IDs in the comparison record.
- **Threshold gaming**: tuning prompts against the eval set. Mitigation: held-out suites and periodic renewal, as in any ML eval discipline.

## 8. Rollback: Strategies and Drives

### 8.1 Rollback = Re-Point, Not Redeploy

The entire reason §3–§6 insist on immutable, registry-versioned artifacts is that rollback then costs one operation: *move the environment pointer to the previous known-good version*. Verified vendor mechanics:

- LangSmith environments keep an **ordered history of assigned commits**, and rolling back selects a prior commit from that history; the environment pointer updates to it ✅ (Manage prompts).
- Braintrust, when a prompt slug is kept across changes, supports **reverting to a prior version in the UI**, and its cookbook walks the pattern of re-invoking the best-scoring earlier version after a worse one shipped ✅ (Prompt versioning cookbook).
- LangChain's Context Hub stores every promotion (environment labels such as `Production 7ca95573` move commit-to-commit), so the promoted-commit history is itself the rollback ledger ✅ (Use the Context Hub).

What rollback does *not* require is a code deploy — which is why agent rollback is genuinely *easier* than classic service rollback, *provided* the artifacts are versioned and the runtime resolves prompts/context by environment tag at request time.

### 8.2 The Drives That Trigger Rollback

| Drive | Signal | Response |
|---|---|---|
| **Quality regression** | Eval/live scores drop, agreement vs incumbent falls | Roll back to incumbent version |
| **Harm/compliance** | Policy-violating, unsafe, or regulated-advice outputs | **Immediate** rollback; freeze; incident |
| **Cost/latency blowout** | Per-conversation cost or p95 latency breaches budget | Roll back or re-tune params as a new candidate |
| **Tool/API breakage** | Pinned tool or model retired/breaking (requests fail) | Roll back to a version whose pins are still servable |
| **Vendor lifecycle event** | Model deprecation notice for a pinned snapshot | *Planned* migration release, not an emergency — if you monitor (§4.1, §10) |

### 8.3 Rollback Drives Against Vendor Lifecycles

A rollback target must still be *executable*: every artifact it pins must exist and be servable. Two vendor-lifecycle facts make this non-trivial ✅ (Anthropic, Model deprecations):

1. **Retired models fail** — a version pinned to a retired model cannot be rolled back to; the "known-good" version set shrinks as providers retire models.
2. **Retirement schedules differ by platform** — the same model may be servable on one platform and retired on another, so a rollback plan can be platform-specific.

Consequence: maintain **at least two servable known-good versions at all times** (current and previous), re-certify the previous version on every model migration, and treat provider retirement calendars as first-class release inputs. A rollback drill that ends by re-pointing to a retired model is a failed drill — §11 runs this scenario on purpose.

### 8.4 Dual-Write and Parallel-Run Verification

For state-changing agents (banking tools especially), rollback has a verification problem: *the bad version already did things* — sent messages, booked transactions, updated records. Two verification practices matter (⚠ both are established software/data practices adapted to agents rather than vendor-native agent features — no primary source this pass documents them as agent primitives):

- **Dual-write / parallel-run**: while a candidate is in shadow or during rollback verification, run old and new versions on the same inputs and compare side-by-side (outputs, tool calls, state deltas, cost). This is the practical engine of the shadow gate (§7.1) and of post-rollback confirmation.
- **State reconciliation**: because a rolled-back agent may have made calls the incumbent would not have, the drive is not just "re-point and forget" but "compare what the bad version did vs what the good version would do, and remediate the deltas" (e.g., retract a message, reverse a provisional hold).

The rollback *narrative* in §11.5 treats dual-run verification as an explicit phase, with the honest caveat that its depth (transaction reversal) lives in the multi-agent deployment and banking-operation siblings: [multi_agent_banking_guide.md](multi_agent_banking_guide.md).

### 8.5 Rollback Strategies Compared

Not every rollback is a registry re-point; teams should choose deliberately. The comparison table below rates each strategy on agent-specific criteria. Platform-support notes are flagged ✅ only where verified at a primary source this pass.

| Strategy | Mechanism | Speed | Customer impact during rollback | State-mutation safety | Platform support (verified) |
|---|---|---|---|---|---|
| **Revert-to-immutable (registry re-point)** | Move environment pointer to previous immutable commit/bundle | Seconds–minutes | None (pointer flip is atomic-ish; next request resolves to old version) | High — nothing new executes; but does not undo what the bad version already did | ✅ LangSmith environment rollback history; ✅ Braintrust UI revert per slug |
| **Blue-green (versioned runtime pair)** | Two full runtimes (blue = incumbent, green = candidate); traffic switch by router | Minutes | Brief cutover; infra cost ×2 | High — switch is at the router; state lives elsewhere | ⚠ Classic deployment pattern; agent-platform-native "blue-green for prompt sets" not verified this pass |
| **Canary-dial-down** | Shrink candidate traffic to 0%, then re-point | Minutes | Candidate slice affected until drained | Medium — in-flight candidate requests still complete | ⚠ Requires traffic-split substrate (§13) |
| **Shadow-swap (promote shadowed version)** | Version already shadow-tested beside incumbent becomes the new incumbent | Reuse of G2 evidence | None | High — decision pre-validated by agreement metrics | ✅ Shadow mechanics documented per-version in Braintrust cookbook evals; full swap semantics team-built ⚠ |
| **Dual-write / parallel-run + state reconciliation** | Run both versions on the same inputs, diff outputs/tool calls/state, remediate deltas | Hours (verification-bound) | None (often post-incident, on replay) | Highest for state-changing agents — the only strategy that *surfaces* what the bad version did | ⚠ Practice adaptation (§8.4); not a vendor-native primitive this pass |
| **Roll-forward (fix forward)** | Author a new candidate fixing the defect; full gate chain | Slow (eval-bound) | None (after initial rollback) | N/A — the *follow-up*, not the *response* | ✅ Supported by any registry (new commit path) |

A second axis for choosing is **what each strategy protects and what it costs** — the two numbers a runbook actually trades off (mechanics and platform support sit in the table above):

| Strategy | What it protects | Principal cost / trade-off | Choose when |
|---|---|---|---|
| **Revert-to-immutable (registry re-point)** | Customers from the degraded version; auditability of the release history | Near-zero at rollback time — the cost was paid *up front* as registry discipline (§3–§6) | Default first response for any §8.2 drive with a servable known-good target |
| **Blue-green (versioned runtime pair)** | Both populations from partial failure during cutover | ≈ ×2 runtime cost; needs a router and state that lives outside the runtime | The runtime cannot re-point prompts at request time (selection rule 4, below) — a workaround, not a replacement |
| **Shadow parallel-run** | Production from an unvalidated candidate — *before* it ever serves | ≈ ×2 compute on copied traffic; produces evidence, not protection, unless paired with a switch | The G2 stage itself, and post-rollback confirmation (§11.5, Step 5) |
| **Dual-write / parallel-run + state reconciliation** | Customers and records from the bad version's irreversible or divergent actions | Hours of verification; needs a replay harness and remediation playbooks (§8.4) | The bad version could have mutated state or sent customer-visible communications |
| **Feature-flag toggle** | Fast escape where pointer mechanics are missing; per-cohort blast-radius control | Flags must be plumbed into artifact resolution, and flag state itself needs versioning; dormant flags rot ⚠ (practice pattern — no vendor documents feature-flagging of prompt resolution as a native primitive this pass) | Teams that already resolve artifacts through flags; gradual per-cohort rollout before a full re-point |
| **Roll-forward (fix forward)** | Nothing immediately — restores service only after the full gate chain | Eval-bound latency; unsafe as the *first* response under incident pressure (§2.2) | The follow-up after a revert, or the only path when no known-good version can meet a *new* requirement |

Selection rules of thumb for banking agents: (1) **first response = revert-to-immutable** whenever a known-good servable version exists — it is the fastest safe action and §8.3's servability rule guarantees the target exists; (2) use **dual-write verification** whenever the bad version could have mutated state or sent customer-visible communications — rollback alone is then insufficient; (3) reserve **roll-forward** for defects that cannot be escaped by reverting (e.g., the known-good version itself violates a *new* regulation); (4) treat **blue-green** as an infrastructure option when the agent runtime cannot re-point prompts at request time — it is a workaround for missing pointer mechanics, not a replacement for them.

## 9. Environments: Dev, Stage and Prod for Agents

### 9.1 Environment = a Named Pointer, Not a Server Fleet

In the artifact model, an environment is primarily a **named pointer** (which version of each artifact is active there), with infrastructure as a secondary concern. Verified vendor pattern ✅:

- LangSmith: environments are reserved commit tags — `staging` and `production` — assigned through a promotion flow; a prompt may sit in staging and production simultaneously (promoting to production does *not* remove it from staging) ✅ (Manage prompts).
- Braintrust: prompts are associated with deployment environments (the TypeScript SDK takes an `environments: ["production"]` field at prompt creation) and can be rolled out to specific environments without redeploying the application ✅ (Create prompts).
- LangChain Context Hub: commits are promoted to `staging` or `production`, and agent runtimes resolve context by environment tag ✅ (Use the Context Hub).
- Microsoft Foundry/classic: flows deploy as **deployments hosted in an endpoint** — endpoint as stable address, deployment as the version instance behind it ✅ (Microsoft Learn, Deploy a flow). Azure's online-endpoint model hosts multiple deployments behind one endpoint, which is the classic substrate for staged traffic ⚠ (traffic-allocation specifics not verified in the extracted page this pass — see §13).

### 9.2 A Practical Environment Set for Agents

| Environment | Purpose | Artifact pointer | Eval expectation |
|---|---|---|---|
| **Dev** | Authoring, iteration, sandboxed tools | Latest commits, freeform tags | None formal — smoke only |
| **Stage** | Integration, compliance review, near-prod shadow | Candidate promoted to `staging` | Full offline gate passed |
| **Prod** | Customer traffic | `production` pointer | Full gate chain passed |
| **Drill/DR** | Rollback and failover practice | Previous known-good version | Re-certified servable |

The *stage* environment carries the heaviest burden for agents: it is where the eval certificate, compliance sign-off, and shadow runs attach before anything touches customers. In banking practice (see [multi_agent_banking_guide.md](multi_agent_banking_guide.md) for the deployment/hypercare layer), stage must mirror prod's *artifact pins* even if not its infrastructure scale — a stage that runs a different model alias or an older knowledge index certifies nothing about the prod candidate.

### 9.3 Environment Parity Traps

- **Pointer drift**: prod resolved to a different prompt commit than stage because promotion wasn't replicated — the classic failure when environments are separate *copies* rather than pointers to a shared registry.
- **Data skew**: shadow/canary comparisons are only meaningful if stage and prod see comparable traffic distributions; low-volume stage shadows can hide regressions that canary then finds at 5% (which is why both gates exist).
- **Key hygiene**: tool credentials, feature flags, and MCP permissions differ per environment; a version certified in stage with read-only tools can behave differently in prod with write tools. Tool permission sets belong *in* the version manifest (§6.1) precisely so environment deltas are visible at review time.

**Promotion hygiene checklist.** Beyond the parity traps, five habits keep environment pointers honest:

- **Promote by object, not by re-edit** — the artifact promoted to prod is the *identical* immutable object certified in stage (same commit hash), never a re-edited copy (§6.3).
- **Record the pointer move** — every promotion logs target commit, actor, timestamp, and prior pointer; the environment's history is the audit trail (§3.2, §8.1).
- **Resolve aliases at the door** — the *promoted* record freezes any alias or environment tag to concrete IDs, so prod never depends on a mutable name (§4.2).
- **Verify environment resolution** — a smoke request per environment confirming the runtime resolves the intended commit/tag, exactly as the drill's canary-shaped re-point does (§11.5, Step 4).
- **Re-check pins on schedule** — servability of every promoted version's pins is re-verified (§8.3) so a rollback target is never a trap.

## 10. Drift, Regression and Continuous Validation

### 10.1 Versioning Ends; Drift Begins

A released version is a snapshot, but the world keeps moving underneath it: the provider nudges or retires the model, the knowledge corpus ages, the tool backend changes behavior, the customer base shifts. The discipline of *continuous validation* watches the gap between the certified version and the live reality. This section only frames the versioning implications; detection methodology, metrics, and monitoring design live in [ai_agent_drift_guide.md](ai_agent_drift_guide.md), and trace-based observation in [agentops_guide.md](agentops_guide.md).

### 10.2 The Four Drift Sources That Versioning Must Absorb

1. **Model drift** — the provider's floating alias moves, or a deprecated pinned model approaches retirement. Versioning response: monitor the lifecycle state of every pinned model ID (Anthropic even documents a usage-audit path — export usage CSV broken down by API key and model — precisely so customers can locate deprecated-model usage ✅), and treat every model change as a new candidate that re-enters the gate chain.
2. **Knowledge drift** — corpus/index refreshes change answers while the version number stays put. Versioning response: record knowledge IDs in the manifest (§5.3) so "the version" is falsifiable.
3. **Tool/API drift** — a tool backend changes behavior without changing its schema. Versioning response: observability deltas on tool outcomes (sibling: [agentops_guide.md](agentops_guide.md)); escalate as a release event when deltas exceed tolerance.
4. **Distribution drift** — the traffic mix changes so the certified eval distribution no longer matches production. Versioning response: periodic dataset renewal from live logs (the Braintrust cookbook's recommended dataset source ✅), which re-baselines the eval contract.

### 10.3 The Continuous Validation Loop

The practical loop, consistent across the vendor materials verified this pass:

```
live traffic → traces/logs → sampled dataset refresh → periodic re-eval of the RELEASED version
        → regression found? → candidate fix → gate chain (§7) → release / rollback (§8)
        → no regression → extend certification window
```

Google's Gemini Enterprise Agent Platform frames the platform-side version of this loop: an **Example Store and Evaluation Service** create *"a continuous feedback loop and data flywheel to iteratively refine agent performance over time"* ✅ (Vertex AI docs, Scale your agents). Anthropic's operational advice for model migrations — *"test your applications with the new models well before the retirement date"* — is the manual version of the same loop ✅ (Model deprecations). Cadence recommendation: a **monthly re-certification** of every production agent version against a refreshed-but-versioned dataset, plus event-driven re-certification on any model, knowledge, or tool change. ⚠ The monthly cadence is this guide's practice recommendation, not a vendor-standard figure.

Whatever the calendar, the *event-driven* triggers are where re-certification earns its keep — each is a release event wearing a monitoring costume:

| Trigger | What to re-certify | Response if it fails |
|---|---|---|
| Calendar (monthly) | Every production agent version against a refreshed-but-versioned dataset | Candidate fix through the full gate chain (§7); interim rollback if the failure is already live (§8) |
| Model lifecycle notice (§4.1) | Every version pinning the affected snapshot | Planned migration release, not an emergency — if the monitor fired early enough (§8.2) |
| Alias move (floating model/context tag) | Versions that resolved through the moved alias | Freeze the alias into concrete IDs at promotion time (§4.2) and re-evaluate the delta as a candidate |
| Knowledge refresh (§5.3) | Versions whose manifest names the refreshed corpus/index | Treat as a candidate change: re-run G1 before the refresh can reach production |
| Tool/API change (§5.1) | Versions pinning the changed tool manifest or backend | Schema changes ride the gate chain; implementation-only changes get outcome-delta monitoring |
| Distribution shift (§10.2) | The eval contract itself (dataset distribution vs live traffic) | Renew the dataset from sampled live logs; re-baseline thresholds |

The recurring shape: re-certification is not a re-test of a static artifact — it is *the released version racing the moving world*, and every trigger above names which part of the world moved.

## 11. Banking Context and Worked Example: The Cymbal Bank Customer-Service Agent

### 11.1 Setting

**Cymbal Bank** (fictional persona bank used across this research library) runs **cyb-cs-agent**, a customer-service agent that answers balance, card, fee, and fraud questions and performs low-risk actions (statement reissue, card freeze, dispute intake). It is governed like any customer-facing banking system: every release needs an eval certificate, compliance sign-off, and an auditable promotion record. The agent is a bundle of five artifact families (§1.2), and Cymbal Bank's platform team treats *the bundle version* as the release unit. Regulatory-grade auditability means the question *"which exact prompt, model, tools, and knowledge served this customer on this date?"* must always have a single answer: the bundle version stamped on the trace.

The bank's platform is assembled from the patterns in §3–§9: a prompt/context registry with commits, tags and `staging`/`production` environment pointers; a model-pin registry with lifecycle monitoring; MCP tool servers pinned per release; a versioned knowledge index; and a manifest per bundle version. (Scaffolding details of the agent itself: [agent_scaffolding_guide.md](agent_scaffolding_guide.md). Deployment/hypercare at the platform level, including the multi-agent estate: [multi_agent_banking_guide.md](multi_agent_banking_guide.md).)

### 11.2 The Artifact Registry

Cymbal Bank's registry table for cs-agent (abbreviated; every row is an immutable object with a content hash and a promotion history):

| Bundle version | Prompt (registry commit) | Model pin | Params | Tools (pinned) | Knowledge | Status |
|---|---|---|---|---|---|---|
| 1.4.0 (incumbent) | `cyb-cs-intake` commit `c9f21e` | `claude-sonnet-4-5-20250929` | temp 0, max 900 | core-banking-mcp 2.3.1, fraud-screen-mcp 1.1.0 | policies-v13, idx-v3 | **production** · known-good |
| 1.5.0 (candidate) | `cyb-cs-intake` commit `d4a07b` | `claude-sonnet-4-5-20250929` | temp 0, max 900 | core-banking-mcp 2.3.1, fraud-screen-mcp 1.1.0 | policies-v14, idx-v3 | staged → canary (see §11.3) |
| 1.6.0 (planned) | — | `claude-sonnet-4-6` (alias; to be frozen) | temp 0 | + disputes-mcp 1.0.0 | policies-v14, idx-v3 | draft |

Notes that matter:

- **1.5.0's only artifact delta is knowledge** (`policies-v13` → `v14`, which rewords the overdraft-fee policy) **and the prompt commit** that tells the agent to prefer the new wording. No code changed — which is exactly the case the whole discipline exists for.
- The **model pin is unchanged** across 1.4.0/1.5.0, keeping the comparison clean. The planned 1.6.0 is a *model migration* (to a newer snapshot, frozen from the `claude-sonnet-4-6` alias at promotion time per §4.2) plus a new tool — a MAJOR-style bundle change.
- Registry hygiene per §8.3: **1.4.0 and 1.5.0 both pin a model that is Active** (retirement not sooner than September 29, 2026 per the vendor lifecycle page ✅ — this guide's September 2026 date context makes that pin a deliberate, monitored choice with a migration already on the calendar).

Behind the bundle rows sits per-artifact history — the same pointer discipline applied one level down. Illustrative rows from Cymbal Bank's registries (abbreviated; each object is immutable and hash-addressed, with a promotion history):

| Artifact registry | Object / ID | Version history (abbreviated) | Status |
|---|---|---|---|
| Prompt — `cyb-cs-intake` | commits `c9f21e`, `d4a07b` | `c9f21e` = v13-era wording; `d4a07b` = v14-preference wording plus the "apply as written" instruction; further commits pending for the 1.5.1 fix | `production` pointer at `c9f21e` after rollback |
| Model pin | `claude-sonnet-4-5-20250929` | One snapshot across 1.4.0/1.5.0 (clean comparison); `claude-sonnet-4-6` alias tracked, to be frozen at promotion for 1.6.0 (§4.2) | Active; lifecycle monitor armed |
| Parameters record | temp 0, max 900 | Unchanged across 1.4.0–1.5.0; no parameter-deprecation exposure on the pinned model (§4.3) | — |
| Tools | core-banking-mcp 2.3.1; fraud-screen-mcp 1.1.0 | Unchanged across 1.4.0–1.5.0; disputes-mcp 1.0.0 staged for 1.6.0 | Deployments healthy |
| Knowledge — corpus / index | policies-v13, policies-v14; idx-v3 | v14 reworded the overdraft-hardship policy into an eligibility list (the incident trigger); v13 restored with the 1.4.0 rollback; v14 retained — knowledge unchanged — for the 1.5.1 re-release | v13 live; v14 held |

These rows are what make the bundle table trustworthy: when the §11.5 drill verifies the 1.4.0 target, it is checking *these* objects one by one.

### 11.3 The Gate Chain for 1.4.0 → 1.5.0

The promotion record (stamped into the manifest, §6.1) for candidate 1.5.0:

| Gate | What ran | Threshold | Result | Evidence recorded |
|---|---|---|---|---|
| **G1 · Offline eval** | Suite `reg-2026-09`: 2,400 scenarios (policy QA, card ops, fraud, refusal safety), LLM-judge graded | pass ≥ 0.95; no metric > 0.02 below 1.4.0 | **PASS** — 0.97 overall, fee-policy subset 0.99 (1.4.0: 0.91) | certificate id `ev-1.5.0-0901` incl. judge model + dataset version |
| **G2 · Shadow (24h)** | 1.5.0 ran beside 1.4.0 on copied live traffic, ~4,100 conversations | agreement ≥ 0.98; 0 policy regressions; cost delta ≤ 5% | **PASS** — agreement 0.992; 3 disputes flagged (all correct per review) | shadow report `sh-1.5.0-0902` |
| **G3 · Canary (5%, 24h)** | 5% of real traffic to 1.5.0; 1.4.0 serves 95% | 0 hard errors; escalation rate ≤ 1.4.0's; live-eval score ≥ 0.95 | **PASS** — 0 hard errors, escalation 0.4% vs 0.5% | canary report `ca-1.5.0-0903` |
| **Compliance sign-off** | Retail-banking compliance reviews fee-policy answers vs v14 wording | approval | **APPROVED** | record `comp-0904` |

With all gates green, the platform service promoted the `cyb-cs-intake:production` pointer to commit `d4a07b` (bundle 1.5.0) on **2026-09-05**, updating the manifest's `promoted` block and emitting the promotion event into the trace pipeline. Rollback path recorded at promotion time: re-point `production` to 1.4.0's commit `c9f21e`.

### 11.4 The Incident: What Goes Wrong

Two weeks into production (mid-September 2026), the **continuous-validation loop** (§10.3) fires. Signals:

1. **Live-eval dip**: the monthly re-certification of the *released* 1.5.0 scores 0.94 overall — under the 0.95 threshold — with the fee-policy subset still strong but the *overdraft-hardship* subset collapsing to 0.71.
2. **Escalation spike**: dispute-intake escalations from customers referencing overdrafts up 3× week-over-week; three customers complain the agent "refused to discuss fee waivers" and quoted a policy line that reads as absolute.
3. **Trace forensics** (via [agentops_guide.md](agentops_guide.md)): the bad answers all trace to bundle 1.5.0's prompt commit `d4a07b` + `policies-v14`. The v14 corpus reworded the hardship policy into an *eligibility list*, and the new prompt commit told the agent to "apply the policy as written" — which the model read as *no discretion*, killing the bank's longstanding hardship-consideration path.

Root cause candidate: a **knowledge + prompt change whose offline suite undersampled the hardship subpopulation** (dataset rot — §7.3: the `reg-2026-09` suite had only 12 hardship scenarios). Classification: quality regression with customer-harm potential and a compliance flavor (the hardship path is regulator-adjacent). Per the bank's runbook (§8.2) this is an **immediate-rollback drive**: do not attempt a roll-forward edit under incident pressure.

### 11.5 The Rollback Drill: 1.5.0 → 1.4.0

The drill below is the bank's *standard* rollback runbook, executed for real during this incident and rehearsed quarterly. It deliberately exercises the §8.3 trap: the drill verifies the target version's pins are **still servable** before the pointer moves.

**Step 1 — Trigger & declare.** Monitoring page confirms both signals (score dip + escalation spike). Incident declared `INC-2026-0918` with severity SEV-1; the on-call platform engineer *freezes* the pipeline: no new promotions, no new knowledge refreshes, no alias moves. Freeze scope: cs-agent only; the rest of the estate continues.

**Step 2 — Freeze the evidence.** Before touching anything, snapshot: the current `production` pointer (1.5.0 / `d4a07b`), the bad-answer trace IDs, the re-certification run ID, and the live-eval dataset version. This is the audit record that later answers "what exactly was serving when?" — never roll back first and *then* try to reconstruct what was live.

**Step 3 — Verify the rollback target.** Check that every artifact pinned by 1.4.0 still exists and is servable: prompt commit `c9f21e` (registry — immutable, present ✅), model pin `claude-sonnet-4-5-20250929` (lifecycle: **Active**, retirement not sooner than 2026-09-29 — still servable with margin ✅), MCP tools 2.3.1/1.1.0 (deployments healthy), knowledge `policies-v13` (index intact). **The target is executable** — this check is what prevents the failed-drill failure mode of §8.3.

**Step 4 — Re-point production.** The platform service promotes `cyb-cs-intake:production` back to commit `c9f21e` (bundle 1.4.0). One pointer move; no code deploy; the environment's ordered history records the rollback as a first-class promotion event (LangSmith-style rollback history ✅). Canary-shaped re-point, not instant flip, per the bank's runbook: 5% for 15 minutes to confirm the pointer resolves and traces carry the 1.4.0 version ID, then 100%.

**Step 5 — Dual-run verification.** The incident team replays the affected traffic slice (the ~600 hardship conversations from the escalation window) through **both** 1.5.0 (frozen, for forensics) and 1.4.0 (restored), side by side: outputs, tool calls, and state deltas per conversation. Results: 1.4.0 offers hardship consideration in 94% of replay cases vs 1.5.0's 7%; 0 state deltas requiring remediation (1.5.0 had performed no irreversible writes — it only *refused*, which is why this rollback is clean). ⚠ Dual-run tooling here is Cymbal Bank's own shadow harness (§8.4), not a vendor primitive — the harness is the same one used for the G2 shadow gate.

**Step 6 — Customer & compliance remediation.** The three complainant conversations are pulled, answered correctly under 1.4.0 by a human agent, and logged against `INC-2026-0918`. Compliance confirms: no regulated disclosure was *omitted* in a way that triggers reporting (the refusals were service-level, not disclosure-level) — recorded in the postmortem.

**Step 7 — Postmortem & re-release.** Root cause: dataset rot (undersampled hardship subset) plus a prompt change that over-constrained discretion. Actions: (a) hardship subset expanded 12 → 140 scenarios with regulator-style edge cases; (b) a *discretion-preservation* rubric added to the judge; (c) the re-certification cadence now includes a stratified subpopulation report, not just a blended score; (d) 1.5.0 is fixed as **1.5.1** (prompt commit revised, knowledge unchanged) and must re-enter the full gate chain from G1 — no fast-path re-promotion of the old 1.5.0. Outcome metric for the incident: **rollback completed in 22 minutes** (trigger → 100% re-point), which the bank treats as the reference time for the quarterly drill.

**Drill lesson.** The rollback worked because the artifacts were immutable and the pointer mechanics existed *before* the incident (§3–§6), because the target's servability was verified before the move (§8.3), and because the freeze preserved the evidence needed for a clean postmortem. None of that is model quality; all of it is versioning discipline.

**Incident timeline at a glance (INC-2026-0918).** The narrative above is the runbook prose; this is the clock the postmortem quotes. Minute offsets are illustrative but constrained by the recorded outcome — full re-point 22 minutes after trigger:

| Time (relative) | Phase (drill step) | What happened | Record |
|---|---|---|---|
| T-0 | Detect (Step 1) | Monthly re-certification of *released* 1.5.0 returns 0.94 overall; overdraft-hardship subset 0.71; dispute escalations 3× week-over-week | Live-eval run ID + trace IDs of the bad answers |
| T+3 min | Declare & freeze (Step 1) | SEV-1 declared; pipeline frozen — no promotions, no knowledge refreshes, no alias moves; freeze scoped to cs-agent | `INC-2026-0918` |
| T+6 min | Freeze evidence (Step 2) | Production pointer, bad-answer traces, re-certification run, dataset version snapshotted before any change | Audit record |
| T+7 min | Verify target (Step 3) | 1.4.0's pins checked servable: commit `c9f21e` present, model Active (retirement not sooner than 2026-09-29), tools healthy, corpus `policies-v13` intact | Servability checklist |
| T+7 → 22 min | Re-point (Step 4) | Canary-shaped move: 5% of traffic to `c9f21e` for 15 minutes — pointer resolves, traces carry the 1.4.0 version ID — then 100% | Promotion events; **rollback complete in 22 min** (reference time for the quarterly drill) |
| Hours after | Dual-run verification (Step 5) | ~600 hardship conversations replayed through both 1.5.0 and 1.4.0: hardship consideration 94% vs 7%; 0 state deltas requiring remediation | Replay report |
| Same day | Remediation (Step 6) | Complainant conversations answered by humans under 1.4.0; compliance confirms no disclosure-level omission | Customer records; compliance note |
| Days after | Postmortem & re-release (Step 7) | Root cause: dataset rot + over-constrained discretion. Actions (a)–(d): hardship subset 12 → 140; discretion rubric added to the judge; stratified subpopulation reporting; 1.5.1 re-enters the chain from G1 | Postmortem + action log |

The timeline's discipline point: detection, freeze, and evidence all came *before* the fix — the incident clock started at T-0 with a monitored signal, and the first three phases changed nothing in production.

## 12. Claims Audit

| # | Claim | Verdict | Primary source |
|---|---|---|---|
| 1 | Prompts/context/task instructions change more often than application code, and may be edited by non-engineers; registry lets behavior change without a full deploy | ✅ | LangChain docs — Prompt & Context Hub (docs.langchain.com/langsmith/prompt-context-hub) |
| 2 | LangSmith prompts version by commits; each saved change creates a commit with browse/diff/revert history | ✅ | LangChain docs — Use the Context Hub |
| 3 | LangSmith reserves `staging`/`production` environment tags; promotion updates the environment pointer; environment keeps ordered commit history enabling rollback | ✅ | LangChain docs — Manage prompts |
| 4 | LangSmith commit tags reference exactly one commit; code can pull prompts by tag (e.g. `name:production`) | ✅ | LangChain docs — Manage prompts |
| 5 | LangSmith prompt-owners mode restricts who can tag/promote; prompt-commit webhooks can trigger CI/CD | ✅ | LangChain docs — Manage prompts |
| 6 | Context Hub models Agent contexts (`AGENTS.md` + tools) and Skill contexts (`SKILL.md`) as versioned bundles promoted to staging/production | ✅ | LangChain docs — Use the Context Hub |
| 7 | Braintrust prompt = slug (stable across updates) + model + params + messages; push from code via CLI; version/deploy independently of code; associate prompts with environments | ✅ | Braintrust docs — Create prompts |
| 8 | Braintrust keeps version history per slug and supports reverting to prior versions in the UI; cookbook loop = eval per version, compare, revert to best | ✅ | Braintrust docs + Prompt versioning cookbook |
| 9 | Anthropic model lifecycle: Active / Legacy / Deprecated / Retired; requests to retired models fail | ✅ | Anthropic docs — Model deprecations |
| 10 | Anthropic gives ≥ 60 days' retirement notice for publicly released models; partner platforms (Bedrock, Vertex) set their own schedules | ✅ | Anthropic docs — Model deprecations |
| 11 | Anthropic deprecated `temperature`/`top_p`/`top_k` on newer models (400 on non-default values; Python SDK v1.0+ raises `TypeError`) | ✅ | Anthropic docs — Model deprecations |
| 12 | Azure ML model registry registers and versions models; refs `azureml:<name>:<version>` and `azureml://registries/.../versions/<v>`; archiving hides from default lists but stays usable | ✅ | Microsoft Learn — Register and work with models |
| 13 | Microsoft Foundry/classic prompt flow: flow defined in `flow.dag.yaml`; deploy as online deployment hosted in an endpoint; deploy from flow or bulk test run | ✅ | Microsoft Learn — Deploy a flow |
| 14 | Gemini Enterprise Agent Platform provides managed Agent Runtime with testing/release management, Evaluation Service + Example Store continuous feedback, tracing/logging/monitoring, Memory Bank with inspectable memory revisions | ✅ | Google Cloud docs — Scale your agents (Vertex AI) |
| 15 | Google Cloud Agent Registry: automatic/manual agent registration; A2A spec support (0.3, 1.0) with Agent Cards; skills as `Skill` resources with revision-based version history | ✅ | Google Cloud docs — Agent Registry, Register agents |
| 16 | LangSmith Deployment runtime model: assistants (config) / threads (state) / runs (workloads); prompts and versioned contexts updated without redeploy | ✅ | LangChain docs — LangSmith Deployment (formerly LangGraph Platform) |
| 17 | A single SemVer-for-prompts/agent-bundles standard exists that vendors agree on | ❌ | No primary standard found this pass |
| 18 | OpenAI model-pinning/alias guidance (as of Sept 2026) | ❌ | OpenAI docs blocked scraping this pass |
| 19 | Canary/shadow/dual-write are first-class vendor-native agent primitives with standardized semantics | ⚠ | Practice adaptation; parts verified (Azure endpoints host deployments; LangSmith promote/rollback), not the full chain |
| 20 | Azure online endpoints split traffic across deployments by percentage | ⚠ | Not verified in extracted page this pass |
| 21 | Humanloop prompt versioning mechanics | ❌ | No primary page extracted this pass |
| 22 | "LangGraph Platform deployments support automatic versioning and instant rollbacks" | ⚠ | Seen in a third-party mirror of LangChain docs; not re-verified at the primary page this pass |
| 23 | Braintrust cookbook recommends real customer interactions from logs as the eval dataset source | ✅ | Braintrust docs — Prompt versioning cookbook |
| 24 | Claude model snapshots use dated IDs (e.g. `claude-sonnet-4-5-20250929`) | ✅ | Braintrust prompt example; Anthropic deprecations table |
| 25 | LangSmith documents prompt-commit webhooks; a documented use case is triggering CI/CD when prompts are updated | ✅ | LangChain docs — Manage prompts (§3.4) |
| 26 | LangSmith's playground is the experimentation surface that feeds the commit → promote flow through `staging` then `production` | ✅ | LangChain docs — Manage prompts; Use the Context Hub (§7.1) |
| 27 | Anthropic notifies customers with active deployments of models approaching retirement; the migration burden remains with the deployer | ✅ | Anthropic docs — Model deprecations (§4.1) |
| 28 | Anthropic's operational advice for model migrations: test applications with the new models well before the retirement date | ✅ | Anthropic docs — Model deprecations (§10.3) |
| 29 | Dated snapshot IDs (e.g. `claude-sonnet-4-5-20250929`) appear in Braintrust's own prompt examples, evidencing the pin syntax | ✅ | Braintrust docs — Create prompts; Anthropic deprecations table (§4.1) |
| 30 | Braintrust documents temperature as exposed only at certain reasoning-effort settings for newer GPT models | ✅ | Braintrust docs (§4.3) |
| 31 | LangSmith permits a prompt to sit in staging and production simultaneously; promoting to production does not remove it from staging | ✅ | LangChain docs — Manage prompts (§9.1) |
| 32 | Context Hub records every promotion as environment labels that move commit-to-commit, making the promoted-commit history the rollback ledger | ✅ | LangChain docs — Use the Context Hub (§8.1) |
| 33 | Gemini Enterprise Agent Platform frames Example Store + Evaluation Service as a "continuous feedback loop and data flywheel" that iteratively refines agent performance | ✅ | Google Cloud docs — Scale your agents (§10.3) |

## 13. What Could Not Be Verified

This section collects every claim this guide could not confirm at a primary source during the research pass, per the honesty rule: an explicit gap beats a fabricated citation.

- **❌ OpenAI model versioning/alias/deprecation guidance.** OpenAI's deprecations documentation page is referenced from its own docs navigation, but every scrape attempt this pass was blocked (all scraping engines failed on platform.openai.com). Specifics such as whether `gpt-*` short names are floating aliases, the notice period for model retirements, and the current pinned-snapshot syntax could therefore not be verified here. Treat §4.2's OpenAI remarks as unverified integration detail.
- **❌ A unified semantic-versioning standard for prompts or agent bundles.** Community proposals exist (semver-for-prompts style discussions), but no dominant, vendor-endorsed standard was found. This guide therefore treats MAJOR/MINOR/PATCH as release policy (§2.2) — a recommendation, not a standard.
- **❌ Humanloop prompt versioning mechanics.** Search and extraction did not return a Humanloop primary page this pass; any Humanloop-specific claims are absent from this guide by design.
- **⚠ Shadow/canary/dual-write as native vendor primitives.** The gate chain in §7 and dual-run verification in §8.4 assemble verified parts (LangSmith promote/rollback pointers ✅, Braintrust per-version evals ✅, Azure deployments-hosted-in-endpoints ✅) into a chain that no single vendor documents end-to-end with identical vocabulary. The chain is a practice synthesis; the parts are verified.
- **⚠ Azure online-endpoint traffic allocation.** The extracted Microsoft page confirms deployments are hosted in endpoints but did not (in the extracted window) document percentage traffic splitting between deployments; that capability is asserted only as the classic Azure substrate, marked unverified.
- **⚠ The "automatic versioning and instant rollbacks" deployment claim for LangGraph Platform.** Encountered in a third-party mirror of LangChain documentation (a fork's deployment page), not re-verified at the canonical page this pass. The canonical LangSmith Deployment page verified here confirms environment and assistant mechanics but not that specific phrasing.
- **⚠ Monthly re-certification cadence (§10.3)** is this guide's practice recommendation, not a figure found in any vendor standard.
- **⚠ Terminology convergence.** All vocabulary claims in §1.4 rest on the verified vendor pages cited in §12, but the *absence* of a shared standard is itself only partially verifiable (absence of evidence) — flagged rather than asserted.
- **⚠ Anthropic retirement dates cited in the worked example.** The specific "not sooner than" date for the model pin used at Cymbal Bank (September 29, 2026) is quoted from the vendor lifecycle page as of this pass; retirement dates are moving targets and the drill's real requirement is the *servability check* (§11.5, Step 3), not any specific date.
- **⚠ The dual-run harness in §11.5 (Step 5).** The replay-and-compare tooling Cymbal Bank uses for post-rollback verification is its own shadow harness (the same one that runs its G2 gate), not a vendor-shipped verification primitive — teams adopting the drill should budget to build their own replay/comparison harness rather than assume one exists in the platform.

## 14. Glossary

| Term | Definition |
|---|---|
| **Agent version / bundle version** | The named, immutable release unit binding one snapshot of each artifact family (prompt, tools, model, parameters, knowledge) plus its eval certificate (§1.2, §6). |
| **Artifact** | Any versionable input to agent behavior: prompt text, tool schema/implementation, model pin, sampling parameters, knowledge corpus/index/memory. |
| **Manifest** | The config-as-code file that declares one agent version's artifact pointers and recorded gate results (§6.1). |
| **Commit** | An immutable snapshot in a registry's append-only history (LangSmith prompt/context commits). |
| **Tag** | A mutable label referencing exactly one commit; moving a tag re-points consumers (§3.2). |
| **Environment** | A named, reserved pointer (e.g. `staging`, `production`) to the active version of an artifact; promotion moves the pointer (§9). |
| **Promotion** | The permissioned act of re-pointing an environment to a new version after gates pass (§7). |
| **Known-good version** | A previously certified, still-servable version suitable as a rollback target (§8.3). |
| **Pin** | An exact, dated artifact reference (e.g. `claude-sonnet-4-5-20250929`), as opposed to a floating alias (§4.2). |
| **Alias** | A mutable name a vendor or registry re-points over time (§4.2). |
| **Eval certificate** | The machine-readable gate record (suite, scores, regression diff, judge versions, timestamps) attached to a version (§7.2). |
| **Offline eval / shadow / canary** | The three escalating gates: fixed-dataset scoring; parallel run on copied traffic without customer impact; small real-traffic slice (§7.1). |
| **Rollback** | Re-pointing an environment to a previous known-good version after a release degrades (§8). |
| **Dual-write / parallel-run** | Running two versions on the same inputs and comparing outputs, tool calls, and state deltas (§8.4). |
| **Model lifecycle state** | Active / Legacy / Deprecated / Retired, the vendor-defined status of a model version (§4.1). |
| **Prompt-as-code** | The practice of keeping prompt source in the repo and pushing it to a runtime registry at release time (§3.1). |
| **Context bundle** | A versioned unit of instructions + tools (Agent/Skill contexts in LangSmith; AGENTS.md/SKILL.md) (§3.3). |
| **MCP** | Model Context Protocol — the standard for connecting agents to tools; MCP servers are pinned per agent version (§5.1). |
| **A2A / Agent Card** | Agent2Agent protocol and the capability-declaring card agents publish; used by Google Cloud Agent Registry for discovery (§5.2). |
| **Skill revision** | A versioned revision of a registered standalone skill in a registry (§5.2). |
| **Drift** | The gap between a certified version's behavior and live reality as the model, knowledge, tools, or traffic move (§10). |
| **Hypercare** | The heightened-monitoring window immediately after a production release (deployment layer: [multi_agent_banking_guide.md](multi_agent_banking_guide.md)). |
| **Incumbent** | The currently serving version that a candidate is compared against in gates and rollback decisions (§7.1). |
| **Roll-forward** | Authoring a new candidate that fixes a defect and running it through the full gate chain — the follow-up after a rollback, not the first response (§8.5). |
| **Servability check** | Verifying, before a rollback or promotion, that every pinned artifact (prompt commit, model lifecycle state, tool deployment, knowledge index) still exists and can serve (§8.3). |
| **Re-certification** | Re-running the eval contract against a released version on a calendar or event-driven trigger (§10.3). |
| **Incident freeze** | Halting promotions, knowledge refreshes, and alias moves for the affected agent while an incident is investigated (§11.5, Step 1). |

## 15. References and Further Reading

All URLs below were actually retrieved during this research pass (September 2026). Vendor docs pages are primary sources; where a page is a cookbook/secondary it is labeled.

**LangChain / LangSmith (primary — docs.langchain.com)**
1. Prompt & Context Hub — https://docs.langchain.com/langsmith/prompt-context-hub
2. Use the Context Hub — https://docs.langchain.com/langsmith/use-the-context-hub
3. Manage prompts (environments, tags, owners, webhooks, rollback) — https://docs.langchain.com/langsmith/manage-prompts
4. LangSmith Deployment (formerly LangGraph Platform) — https://docs.langchain.com/langgraph-platform

**Braintrust (primary — braintrust.dev docs)**
5. Create prompts (slug, model, params, environments, templating, push from code) — https://www.braintrust.dev/docs/evaluate/write-prompts
6. Prompt versioning and deployment — cookbook (version → eval → revert loop) — https://www.braintrust.dev/docs/cookbook/recipes/PromptVersioning

**Anthropic (primary)**
7. Model deprecations (lifecycle states, retirement dates, notice policy, parameter deprecations) — https://docs.anthropic.com/en/docs/about-claude/model-deprecations

**Microsoft (primary — learn.microsoft.com)**
8. Register and work with models (Azure Machine Learning model registry: versions, `azureml:` references, archive) — https://learn.microsoft.com/en-us/azure/machine-learning/how-to-manage-models
9. Deploy a flow for real-time inference (Microsoft Foundry classic prompt flow: flow.dag.yaml, online deployments in endpoints) — https://learn.microsoft.com/en-us/azure/foundry-classic/how-to/flow-deploy

**Google Cloud (primary — cloud.google.com / docs.cloud.google.com)**
10. Scale your agents (Gemini Enterprise Agent Platform: Agent Runtime, Evaluation Service, Example Store, Memory Bank revisions) — https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/deploy
11. Register agents (Agent Registry: automatic/manual registration, A2A 0.3/1.0, Agent Cards, skill revisions) — https://docs.cloud.google.com/agent-registry/register-agents

**Sibling guides in this library (cross-referenced, not duplicated)**
12. [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) — eval methodology, metrics, judges, datasets
13. [ai_agent_drift_guide.md](ai_agent_drift_guide.md) — drift detection and monitoring
14. [agent_scaffolding_guide.md](agent_scaffolding_guide.md) — agent architecture and scaffolding
15. [multi_agent_banking_guide.md](multi_agent_banking_guide.md) — multi-agent deployment, hypercare, quality gates
16. [mcp_discovery_guide.md](mcp_discovery_guide.md) — MCP tooling and governance
17. [context_engineering_guide.md](context_engineering_guide.md) — skills, AGENTS.md, context bundles
18. [agentops_guide.md](agentops_guide.md) — observability, tracing, version IDs on traces

**What each primary entry was used for in this guide.** Section pointers let a reader trace any claim back to the source that carried it:

- **Entry 1** (Prompt & Context Hub) — §1.1's central observation that prompts/context change more often than application code and are edited by non-engineers; §12 row 1.
- **Entry 2** (Use the Context Hub) — §3.2 commit history and diff/revert; §3.3 Agent/Skill context bundles; §8.1 the promotion ledger; §12 rows 2, 6, 32.
- **Entry 3** (Manage prompts) — §3.2 tags/environments/promotion/owners and code-time resolution; §3.4 webhooks; §7.1 and §9.1 staging/production semantics; §8.1 rollback-by-history; §12 rows 3–5, 25, 31.
- **Entry 4** (LangSmith Deployment) — §6.2 the assistant/thread/run runtime model; §12 row 16.
- **Entry 5** (Braintrust Create prompts) — §3.1 prompt-as-code push pattern; §4.1 dated snapshot IDs in vendor examples; §4.3 reasoning-parameter surface; §9.1 environment association; §12 rows 7, 29, 30.
- **Entry 6** (Braintrust PromptVersioning cookbook) — §7.1 eval-per-version loop; §7.3 dataset-from-logs recommendation; §8.1 revert pattern; §12 rows 8, 23.
- **Entry 7** (Anthropic Model deprecations) — §1.2 the lifecycle obligation; §4.1 lifecycle states/notice/platform schedules; §4.3 parameter deprecations; §8.3 servability; §10.3 pre-migration testing; §12 rows 9–11, 27, 28.
- **Entry 8** (Azure ML model registry) — §4.4 registration, `azureml:` references, archiving, stage tags; §12 row 12.
- **Entry 9** (Microsoft flow deployment) — §6.2 flow-as-config; §7.1 test-then-deploy; §9.1 deployments-in-endpoints; §12 row 13.
- **Entry 10** (Vertex scale-your-agents) — §5.3 inspectable memory revisions; §7.1/§10.3 Evaluation Service + Example Store loop; §12 rows 14, 33.
- **Entry 11** (Agent Registry register-agents) — §5.2 registration modes, A2A support, Agent Cards, skill revisions; §12 row 15.

**Reading order suggestion.** For a team standing this up: read §1–§2 (discipline and framing), then the platform patterns in §3–§6, then §7–§10 (the release machine), and finally re-read §11 as a template — the worked example is deliberately written to be lifted into a team's own runbook. The honest gaps in §13 mark exactly where the field is still moving fastest: provider alias mechanics, traffic-split semantics, and any would-be "SemVer for agents."

---

*Versioning is how a stochastic system earns operational trust: commit nothing mutable, certify every candidate, promote by pointer, and roll back without fear — because the artifact you can name exactly is the artifact you can govern. Ship prompts like code, models like dependencies, and knowledge like inventory, and you will always be running the versioned agent.*
