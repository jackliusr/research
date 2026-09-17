# MCP Progressive Disclosure — The Disclosed Tool

> **A 2026 deep-dive on the tool-surface problem in the Model Context Protocol (MCP) ecosystem.** MCP solved *connection* — how a client talks to a server. It did not solve the consequence of connecting many of them: **every tool definition you can reach is a tax you pay on every turn**, before the user's request is even read. This guide covers progressive disclosure in that specific sense — deferred and dynamic tool loading, tool search, code-execution ("code mode") tool surfaces, hierarchical namespacing, tool retrieval, gateway aggregation, and the skills-style layered instruction format — with the measurement evidence, the documented trade-offs, the failure modes, and the enterprise treatment.

> **Series context and boundary (read this first).** This is a *dedicated* guide to one axis of the MCP story: **how much of the tool and instruction surface is disclosed to the model, and when.** It deliberately does not re-derive material owned by four sibling guides in this repository:
>
> - **Context economics, context budgets, compression, positional effects, retrieval-fed assembly** → [context_engineering_guide.md](context_engineering_guide.md) (785 lines).
> - **The agent loop, harness architecture, KV-cache-aware context design, memory tiers** → [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) (777 lines).
> - **MCP fundamentals — primitives, architecture, transports, implementing a server, MCP security basics** → [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) (697 lines).
> - **Server discovery: the official registry, directories, marketplaces, registration mechanics, enterprise discovery** → [mcp_discovery_guide.md](mcp_discovery_guide.md) (640 lines).
>
> The crisp contrast that separates this guide from the fourth: **discovering a server is not the same as disclosing a tool.** Discovery answers *"does this server exist, what does it offer, how do I reach it, and am I allowed to attach it?"* — a **finding and adoption** problem, answered once per server, by humans and registries and platform teams. Disclosure answers *"of everything this client can now reach, which definitions and instructions enter the model's context window for this turn?"* — an **inference-time surface** problem, answered on every request, by the client runtime, for thousands of tools. A team can run a flawless server-discovery process and still blow its context window on turn one; a team can defer every tool definition and still adopt an unapproved server via shadow IT. They are different controls with different owners.
>
> **Verification policy.** Facts marked **(verified)** were checked against a primary source — the MCP specification itself, a vendor's own engineering post or product documentation, or a published paper — during writing (**September 2026 baseline**). Everything else is labelled: **(vendor claim)** for a figure a vendor publishes about its own product; **(announced / beta)** for a feature described but not shipped generally; **(unverified/flag)** where a widely-repeated figure could not be traced to a primary measurement; **(negative finding)** where a searching pass established that the thing does *not* appear in the sources checked. **No token count, percentage, latency figure, feature-availability status, or revision date in this guide is written from memory.** Where a number is a vendor's own illustration rather than an independently reproduced measurement, this guide says so in the sentence that carries it. Parameters and thresholds quoted from vendor documentation are quoted because they were read there, and they move — re-read the page before you build a policy on it.

---

## Contents

1. [Overview and the Boundary Declared](#1-overview-and-the-boundary-declared)
2. [The Problem, Quantified](#2-the-problem-quantified)
3. [The Taxonomy of Approaches](#3-the-taxonomy-of-approaches)
4. [Deferred Loading and Tool Search](#4-deferred-loading-and-tool-search)
5. [Code Execution and the Tool-as-API Pattern](#5-code-execution-and-the-tool-as-api-pattern)
6. [Instruction-Level Disclosure — The Skills Pattern](#6-instruction-level-disclosure--the-skills-pattern)
7. [The Measurement Evidence](#7-the-measurement-evidence)
8. [The Trade-offs and the Decision Framework](#8-the-trade-offs-and-the-decision-framework)
9. [Design Guidance](#9-design-guidance)
10. [The Failure Modes](#10-the-failure-modes)
11. [The Enterprise and Regulated-Institution Angle](#11-the-enterprise-and-regulated-institution-angle)
12. [Cymbal Bank Worked Example](#12-cymbal-bank-worked-example)
13. [The Claims Audit — Verified, Flagged, Rejected](#13-the-claims-audit--verified-flagged-rejected)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

---

## 1. Overview and the Boundary Declared

### 1.1 What "progressive disclosure" means here

**Progressive disclosure, in the MCP ecosystem, is the practice of not putting the whole tool and instruction surface in front of the model at once — instead exposing a small index, or a summary, or nothing at all, and materialising the full definition of a tool (or the full body of an instruction) only when the task needs it.**

The distinguishing feature of this definition is that it is a statement about **when tokens enter the context window**, not about capability, not about discovery, and not about authorisation. A tool can be fully available, correctly permissioned, statically registered, and yet be **deferred** — its schema withheld from the prompt until something asks for it.

The vocabulary in the field is loose in a way that causes real confusion, because several distinct things are all described as "progressive disclosure":

- **Tool-level disclosure** — the tool schema is withheld until selection. This is the sense this guide mostly uses.
- **Instruction-level disclosure** — a skill or procedure's body is withheld until a trigger matches. Same principle, different artifact.
- **Result-level disclosure** — a tool *result* is filtered or transformed before it enters context, so a 10,000-row spreadsheet becomes five logged rows.
- **Interaction-level disclosure** — the human-facing sense: streaming the agent's progress ("searching accounts… verifying…") so a person sees the work unfold rather than one late monolith. This is a UX pattern, not a context-budget pattern, and it is covered in the repository's agent-failure material rather than here.

The first two are the subject of this guide, because the second-order consequences — selection accuracy, cache invalidation, auditability — attach to them. The third is discussed as a companion benefit of the code-execution approach (§5), and the fourth is out of scope.

### 1.2 The one-paragraph statement of the problem

In MCP, tool definitions are delivered to the model as part of the prompt. The specification's own mechanics make this explicit: a client lists a server's tools via `tools/list`, and the definitions returned — names, descriptions, and JSON Schemas for every parameter — are presented to the model so it can choose. When a client aggregates several servers, the union of those schemas is presented on **every turn of every conversation**, whether or not the turn could ever use a GitHub pull-request tool while the user is asking about a Jira ticket. Anthropic's own engineering post describes this in one sentence: there are cases where agents "need to process hundreds of thousands of tokens before reading a request" (vendor claim, November 2025). The cost is not a one-off loading cost; it is a **per-turn, per-conversation tax on the exact resource — context — that the user's actual request needs to do its work.**

### 1.3 The decoder table

The vocabulary needed to read the rest of this guide, decoded precisely — because the words "discovery," "retrieval," and "search" are used for at least three different layers of the stack.

| Term | What it actually means | Who does it | The question it answers |
|---|---|---|---|
| **Tool definition / tool schema** | The name, description, and JSON Schema (input and, since the 2025-06-18 revision, optionally structured output) describing one callable tool | Specified by MCP; authored by the server | "What arguments does this tool take?" |
| **Tool list** | The complete set of tool definitions a client holds for a server, obtained via `tools/list` | Client pulls from server | "What can this server do?" |
| **Context bloat** | The condition where definitions, results, and history crowd the window so the residual space for the task is small | Emergent | "Why is the model performing worse on a large context?" |
| **Deferred loading** | Marking a tool so its definition is *not* placed in the prompt prefix until requested (`defer_loading: true` in the Claude Developer Platform API) | Model-provider API | "How do I keep the schema out of the prompt?" |
| **Tool search** | A model-callable tool that searches a catalogue (regex, BM25, or a custom/embedding implementation) and returns matching definitions, which are then expanded into context | Model, via a provider-hosted tool | "Which of my tools match this need?" |
| **Tool retrieval** | The same *idea* implemented more generally in an agent framework: embed or index tool descriptions, retrieve a shortlist per query, bind only those to the model | Framework / application | "How do I build my own discovery step?" |
| **Code mode / code execution over tools** | Presenting MCP servers as a code API (a filesystem of typed tool files, or a generated TypeScript client) so the agent writes code that calls them and the definitions never enter the prompt | Agent harness + sandbox | "Why put schemas in the prompt at all?" |
| **Skills / SKILL.md** | A layered instruction format: a small name+description descriptor always loaded, a body loaded on trigger, references and scripts loaded only when read | Agent platform | "How do I ship 100 procedures without paying for 100 procedures?" |
| **Hierarchical namespacing** | Encoding the server (or domain) into the tool name (`github_create_issue`, `Slack:post_message`) so a search or a group can match a whole family at once | Server/harness author | "How do I make one search hit the right family?" |
| **Gateway aggregation** | A proxy in front of many servers that presents a collapsed, curated surface (often profile-based) to the client | Platform infrastructure | "How do I stop exposing everything?" |
| **Dynamic discovery vs. static disclosure** | Two different axes: finding a *server* at runtime (discovery — `server/discover`, registries) versus varying which *tools* are offered at runtime (disclosure) | Different layers | "Are you finding a server, or disclosing a tool?" |

### 1.4 The four sibling guides, and what each owns

Stated explicitly so the boundary is enforceable in review:

- **[context_engineering_guide.md](context_engineering_guide.md) owns the discipline.** Context-window economics, context budgets, compression and summarisation, retrieval-fed context assembly, positional effects, long-running context management. When this guide says "a changing tool block costs you a cache hit," it states the *consequence for the tool surface* and points there for the general theory.
- **[agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) owns the harness.** The agent loop, loop architecture, memory tiers, KV-cache-aware context design (including the prefix-stability rules), and the harness-level mechanics of assembling a prompt. The harness guide contains **one dense paragraph** on this topic — it names just-in-time retrieval / progressive disclosure, token-efficient tool design, and the "tens of thousands of tokens consumed by schemas before the user request is read" observation, and cites Agent Skills and dynamic context assembly as named tools. That paragraph is the seed of this guide, not its treatment; this guide is where the mechanism, the evidence, and the failure modes get developed.
- **[mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) owns the protocol.** What MCP is, its primitives (tools, resources, prompts), architecture patterns, transports, server implementation, and baseline security. This guide cites the specification's *tool-list and notification mechanics* only where disclosure depends on them.
- **[mcp_discovery_guide.md](mcp_discovery_guide.md) owns server discovery.** The official MCP Registry, directories and marketplaces, registration and metadata mechanics, remote and dynamic *server* discovery, enterprise discovery processes, and discovery security. It is a different axis: it answers how a client learns a server exists. That guide scores exactly one incidental hit on the progressive-disclosure vocabulary, which is consistent with it being about a different problem.

**The contrast, crisply: discovering a server is not the same as disclosing a tool.** Discovery is an *estate* problem — which servers exist, are they approved, who owns them, how do they get connected — resolved by registry, catalogue, and platform process, mostly outside the model's context window. Disclosure is an *inference-time* problem — of the tools already reachable, which definitions enter the window on this turn — resolved by the client runtime and the model provider's API, entirely inside the context window. A mature enterprise needs both, and the failure of one does not excuse the failure of the other: an unapproved server that is never discovered still can't be poisoned, and an approved server with 200 tools still costs 200 tools' worth of schema on every turn.

---

## 2. The Problem, Quantified

This section exists because "it's a problem" is not an engineering statement. The numbers below are real, dated, and attributed — and where the attribution is only to a vendor's own illustration, that is stated in the sentence.

### 2.1 How tool definitions reach the model, and why the cost recurs

The mechanical path, per the current specification:

1. A client connects to an MCP server and calls **`tools/list`**. The server returns tool definitions: name, description, input schema, and optionally an output schema and annotations. The client (or the model provider's API, in the hosted case) then formats those definitions into the model's tool-calling interface.
2. The model provider places them in the prompt — for Claude, in the **`tools` segment, which precedes `system` and `messages` in the cache-prefix hierarchy** (documented behaviour, Claude prompt-caching docs, September 2026) — and every subsequent turn re-sends the same tool block, because the block is part of the conversation prefix and the API is stateless with respect to your conversation.

That second step is where the problem lives. There is no moment at which the tool definitions are "learned" or amortised; they are re-transmitted and (absent caching) re-billed on each turn. A 55,000-token tool block is not a 55,000-token cost, it is a 55,000-token cost × turns, and it occupies 55,000 tokens of a window that also has to hold the system prompt, the conversation history, the tool results, and the user's actual request.

### 2.2 The measured overhead — what the numbers are, and whose they are

**The single attributable per-server inventory in the field is Anthropic's five-server table** (vendor claim, `anthropic.com/engineering/advanced-tool-use`, published **24 November 2025**, and restated in the Claude Developer Platform tool-search documentation):

| Server | Tools | Vendor-reported token cost |
|---|---|---|
| GitHub | 35 tools | ~26K tokens |
| Slack | 11 tools | ~21K tokens |
| Sentry | 5 tools | ~3K tokens |
| Grafana | 5 tools | ~3K tokens |
| Splunk | 2 tools | ~2K tokens |
| **Total (five servers)** | **58 tools** | **≈55K tokens before the conversation starts** |
| Jira (cited separately) | not given | ~17K tokens |

The same post adds: "Add more servers like Jira… and you're quickly approaching 100K+ token overhead. At Anthropic, we've seen tool definitions consume **134K tokens** before optimization." The documentation restates the five-server figure as "**~55k tokens** in definitions before Claude does any work."

**Read this table for what it is.** It is a vendor's own illustration with **no published tokenizer, methodology, or measurement script**; the token counts are approximations ("~26K"), and the tool counts are not dated to a specific server version. The Claude docs repeat the same figure, which is **documentation consistency, not independent verification** — both sources are Anthropic. **No independent, reproducible, third-party measurement campaign of MCP tool-definition token cost with a published script and raw inventory was found in the research pass behind this guide** (negative finding). An enterprise must therefore **measure its own estate with its own tokenizer**; the table's value is as an order-of-magnitude anchor for how *lumpy* the distribution is (2 tools ≈ 2K tokens for Splunk; 11 Slack tools ≈ 21K — nearly twice the tokens for a third of the tools, because descriptions differ in verbosity).

### 2.3 The claim-and-reality gap on the most-repeated figures

Three widely-circulated numbers deserve explicit handling, because they are the ones a stakeholder will quote back at you:

- **"The GitHub MCP server costs 55,000 tokens."** This recurs across blog posts. It **conflicts with Anthropic's own "GitHub: 35 tools ≈ 26K tokens"** and no source reconciles the discrepancy (a plausible explanation — different tool-count configurations, given GitHub's `--toolsets` flag — is nowhere documented). **Flag: untraceable as a standalone figure.** If you must cite it, attribute it to Anthropic's November 2025 estimate or drop it.
- **"150,000 → 2,000 tokens, a 98.7% saving."** This is real, quoted correctly, and comes from Anthropic's *Code execution with MCP* post (4 November 2025). It is a **vendor illustration of one workflow**, not a benchmark. Use it as an illustration, never as a planning assumption.
- **"Token usage drops 85%."** Anthropic's tool-search post (24 November 2025) states this for the Tool Search Tool path (~77K tokens of context consumption before work begins, versus ~8.7K with search), with a diagram caption giving "191,300 tokens of context compared to 122,800" — implying a ~200K window in their internal setup. **Vendor claim, asymmetric illustration.** Note that **85%, ">85%", and 98.7% are three different figures for three related-but-different techniques**, and conflating them is a common error in secondary write-ups.

### 2.4 The second-order effects

Token cost is the visible symptom. Three others matter more in production.

**(a) Selection accuracy degrades with catalogue size — and the strongest published threshold is a vendor's.** Anthropic's documentation states flatly: "Claude's ability to pick the right tool **degrades once you exceed 30–50 available tools**," and the Claude Code Agent SDK documentation restates it as "Tool selection accuracy degrades with more than 30–50 tools loaded at once." **Vendor claim, no published methodology.** Interestingly, **the only major model API vendor publishing a hard numeric guidance is Google**: the Gemini function-calling documentation carries the best-practice line "**Tool Selection: Keep active set to 10-20 tools maximum**" (verified on the page, whose footer read "Last updated 2026-09-16 UTC" at the time of checking). The vendor's own number is an *order of magnitude below* the catalogues enterprise MCP estates actually build, which is the whole point.

**(b) The user's own request is crowded out.** This is the effect the section title of the vendor post names directly: the tokens are consumed "**before the user request is read**." It is a positional and budget effect: the request sits after a wall of schemas. Note a caution — **no study was found that isolates "lost in the middle" / needle-in-a-haystack degradation specifically for tool definitions**, as distinct from retrieved prose documents (negative finding). What *is* measured is that **tool ordering changes selection**: the BiasBusters benchmark (ICLR 2026 camera-ready) found models "disproportionately favour tools that appear earlier in the context," and that **small perturbations to tool descriptions can significantly shift choices**. Do not extrapolate haystack-recall findings to schemas; do cite position and wording effects, which are directly evidenced.

**(c) The caching interaction — the effect that makes naive disclosure actively harmful.** Cache behaviour is documented, not asserted: modifying tool definitions **invalidates the entire prompt cache** (tools, system, and messages), because tools sit at the head of the cache prefix. Anthropic's own Claude Code team documented in April 2026 that "**changing the tool set in the middle of a conversation is one of the most common ways people break prompt caching**," that they treat cache breaks as **incidents**, and that in their own history one cause was "shuffling tool order definitions non-deterministically." Their resolution is the pattern this guide returns to repeatedly: **defer, don't remove** — keep every tool present in the request in a stable order, and let deferred tools be discovered, so "adding tools dynamically through tool search does not break your cache."

Two consequences follow, and they are counter-intuitive:

- **A disclosure scheme that removes tools it doesn't think it needs is cache-hostile.** It saves a prefix's worth of tokens once and then pays full uncached input on every turn afterwards.
- **The new 2026-07-28 spec revision is directly aware of this problem at the protocol level.** Its changelog instructs that servers **SHOULD** return tools from `tools/list` in a **deterministic order** "to enable client-side caching and improve **LLM prompt cache hit rates**," and requires `ttlMs` and `cacheScope` fields on list results so that a tool list can be cached rather than re-fetched. When a *protocol specification* ships a normative hint whose stated purpose is prompt-cache hit rates, the tool block's cacheability has graduated from a vendor trick to a protocol concern.

The general theory of caching, prefix stability, and context budgeting is owned by the harness and context guides — see §16. What belongs here is the tool-surface-specific rule: **any mechanism that changes the set or the order of tool definitions between turns destroys the cheapest tokens you have.**

---

## 3. The Taxonomy of Approaches

Six families solve the disclosure problem, and they are not interchangeable: they trade different things, they fail differently, and several compose. This section is the organising frame for the rest of the guide.

| Family | What enters the context | What is withheld | Saves | Costs | Breaks when |
|---|---|---|---|---|---|
| **Deferred / dynamic tool loading (tool search)** | A search tool + a few pinned tools; then 3–5 expanded definitions per search | All deferred tool schemas | Steady-state prompt tokens; accuracy at large catalogues | A discovery round trip; the index becomes a failure surface | The search query misses the right tool, or the agent under-searches |
| **Code execution over tools (code mode)** | A filesystem or API-shaped tool surface; the agent's code and its filtered output | Essentially all schemas as prompt text | The largest share of definition tokens; also result tokens | A sandbox, a runtime, an execution-paradigm shift | The environment is unavailable; code generation fails; sandbox is too permissive |
| **Hierarchical namespacing / grouping** | A shallow index of groups; then the requested group | Tools outside the selected group | Tokens, and cognitive load on selection | Name-space discipline and enforcement | Groups are wrong, or names drift from the group |
| **Tool retrieval (embeddings / keyword index)** | A retrieved shortlist bound to the model | Everything not retrieved | Tokens; enables cross-server search | A retrieval pipeline to build and evaluate; recall becomes a dependency | Recall misses (the "tool never found" failure) |
| **Gateway aggregation** | A collapsed, curated surface from a proxy | Everything not in the profile | Tokens; centralises governance and credentials | A new infrastructure component and a coupling point | The curated surface is stale, or the gateway is a single point of failure |
| **Instruction-level disclosure (skills)** | A ~100-token descriptor per skill; then the triggered body | Bodies, references, scripts | Instruction tokens at scale | The descriptor must carry enough signal to trigger | The descriptor under-specifies (never triggers) or over-specifies (always triggers) |

### 3.1 Deferred / dynamic tool loading — the model is shown a searchable index

**Mechanism.** The full catalogue is still handed to the API (this is important and widely misunderstood — see §4.4), but individual tools are marked so their definitions are **excluded from the prompt prefix**. The model sees a search tool plus a small set of always-loaded tools. When it needs a capability it invokes the search; the runtime returns matching tools, which the API expands into full definitions appended to the conversation.

**What it saves.** The steady-state definition tokens on every turn, and — per both vendor and independent evidence — selection accuracy at catalogue sizes where the model would otherwise be choosing from hundreds of options.

**What it costs.** One extra round trip per discovery; a new failure surface (the index and the query); a loss of the model's ability to reason across the *whole* catalogue at once, because it cannot see what it has not searched for; and platform coupling to whichever provider implements the mechanism.

**Where it breaks.** When the search query is wrong or too narrow; when the agent does not realise it needs to search at all; when the index's naming and descriptions are poor enough that good queries miss.

### 3.2 Code execution over tools — definitions never enter the prompt

**Mechanism.** Rather than exposing tools as tool-call schemas, the harness presents MCP servers as a **code API**: either a filesystem tree of tool files the agent lists and reads on demand (Anthropic's formulation), or a generated TypeScript client wrapping the server's schema with doc comments (Cloudflare's), or a code-execution sandbox in which the model writes orchestration code that calls tools.

**What it saves.** The largest share of the tokens, because schema text as prompt text is largely eliminated: the agent reads only the tool files it needs, and intermediate results can be filtered *inside* the execution environment so only the reduction crosses into context.

**What it costs.** A secure execution environment with sandboxing, resource limits, and monitoring — which Anthropic itself flags as operational overhead and security consideration that direct tool calls avoid. It also shifts the agent's competence requirement from argument formatting to code generation, which is a different skill surface.

**Where it breaks.** No sandbox → no technique. Sandbox misconfiguration → the technique becomes an exploit primitive. Weak code generation → the agent fails at a task it would have handled via a well-described tool.

### 3.3 Hierarchical namespacing / grouping — a shallow index, expanded on demand

**Mechanism.** The tool surface is organised into families — by server, by domain, by verb — and either the model or the retrieval layer works at the family level first. Anthropic's own documentation recommends **namespacing tool names by service** (`github_`, `slack_`) explicitly "**so one search matches the whole group**," and its per-server deferral form (`mcp_toolset`) defers an entire server while pinning named high-use tools.

**What it saves.** Tokens, and the model's attention: a group name is a cheaper decision than 200 sibling schemas.

**What it costs.** Discipline, enforced at authoring time and maintained forever, plus a naming convention that must survive renames.

**Where it breaks.** When the grouping does not match how requests arrive (a task that spans two groups), or when a rename silently breaks the mapping.

### 3.4 Tool retrieval — the build-your-own shortlist

**Mechanism.** An application-level retrieval step over tool descriptions: embed or index the catalogue, retrieve top-k per query, bind only those tools to the model. This is the general form of which provider-hosted tool search is a managed instance; it is also the earliest form of the idea in the literature (RAG-MCP, May 2025 — see §7).

**What it saves.** The same tokens, without depending on a provider feature.

**What it costs.** You now own a retrieval system whose recall failure is an agent failure; you need evaluation for retrieval separately from evaluation for task success; and the "how many tools should the shortlist contain?" question becomes your design problem — and the published answer is that a **fixed** shortlist size demonstrably fails (§7).

**Where it breaks.** Recall miss; embedding drift; a catalogue whose descriptions are too similar to discriminate.

### 3.5 Gateway aggregation — collapse many servers into one curated surface

**Mechanism.** A proxy sits between clients and servers, owns configuration, credentials, and access control, and decides which servers are made available. Docker's MCP Gateway is the clearest documented example: it "acts as a centralized proxy between clients and servers," identifies which server handles a requested tool and starts it as a container if needed, and uses a **profile** to determine which servers are available to clients.

**What it saves.** Tokens, by shrinking the offered surface at the infrastructure layer rather than the prompt layer — and it gives a platform team a single control point for credentials, approval, and audit.

**What it costs.** A new component, its own availability obligations, and a coupling between the agent platform and the gateway vendor. Also note a maturity caveat: Docker's documentation states that **"MCP Gateway as part of Docker AI Governance is an invite-only feature."**

**Where it breaks.** When the curated profile is stale relative to reality; when the gateway becomes a single point of failure; when it aggregates *more* than the client needs and re-creates the problem one layer up. **Important boundary:** on the documented pages, gateway filtering is **server/profile-level selection, not tool-level search or retrieval** — a distinction worth holding, because conflating the two leads teams to expect a gateway to solve a tool-surface problem it does not address.

### 3.6 Instruction-level disclosure — the skills pattern

**Mechanism.** A procedure ships as a directory containing a markdown file whose YAML frontmatter carries a `name` and a `description`. At startup the agent pre-loads **only that metadata** for every installed skill. When the description matches the task, the agent reads the body; deeper reference files and scripts are read (or executed) only if needed.

**What it saves.** Instruction tokens at scale, and it does so with a much shallower engineering lift than tool search — it is a *format and a loading convention*, not new infrastructure.

**What it costs.** The descriptor must be a good retrieval key; a library of skills accumulates maintenance debt; and skills are instructions, so a bad skill is a behavioural defect, not just a wasted token.

**Where it breaks.** Under-specified descriptor → the skill never triggers. Over-broad descriptor → it triggers on everything and becomes context noise.

**How it differs from tool disclosure.** Tool disclosure withholds a *schema* the model needs to construct a call; instruction disclosure withholds a *procedure* the model needs to execute a task. The trigger mechanisms are different (search over definitions vs. description matching), the failure signatures are different (wrong tool vs. skill never loaded), and the governance story is different — a hidden tool can take an action; a hidden instruction changes behaviour. §6 develops this.

---

## 4. Deferred Loading and Tool Search

### 4.1 The mechanism as implemented

The shipped, documented implementation in the Claude Developer Platform works in these steps (verified against the API documentation, September 2026):

1. Include a **search tool** in the request's `tools` array.
2. Provide **every** tool definition in `tools`, and set `defer_loading: true` on those that should not load up front. **At least one tool must remain non-deferred.**
3. The API places only the search tool plus the non-deferred tools in the model's context.
4. The model searches; **the API runs the search server-side** and returns matching tools as `tool_reference` blocks.
5. The API expands those references into full definitions; the model then calls the discovered tools.

Two built-in search variants ship — a **regex** variant and a **BM25** variant — and custom search tools (embeddings, for example) are supported. The search covers tool **names, descriptions, argument names, and argument descriptions**.

There is also an MCP-specific form: a **`mcp_toolset`** can defer an **entire MCP server** at once while pinning specific high-use tools by name to remain loaded — for example, deferring a Google Drive toolset wholesale but keeping `search_files` loaded up front.

### 4.2 Availability status, precisely

| Surface | Status | Evidence |
|---|---|---|
| Claude Developer Platform **Tool Search Tool** | **Shipped and documented** as an API feature; announced 24 Nov 2025 as a beta capability under header `advanced-tool-use-2025-11-20` | API docs page; identifiers `tool_search_tool_regex_20251119` / `tool_search_tool_bm25_20251119` |
| **MCP connector `defer_loading`** (`mcp_toolset`) | **Beta** — the connector's compatibility table lists Claude API, Claude Platform on AWS, and Microsoft Foundry as "Beta" | MCP connector docs; beta header `mcp-client-2025-11-20` |
| **Claude Agent SDK / Claude Code tool search** | **On by default**, with documented exceptions | Agent SDK docs; `ENABLE_TOOL_SEARCH` = unset / `true` / `auto` / `auto:N` / `false` |
| **OpenAI** hosted tool search or deferred loading | **Not found** — no `defer_loading`-equivalent, no hosted tool-search tool, no documented maximum tool count on the function-calling or MCP/connectors pages read (negative finding, two pages) | OpenAI function-calling and MCP/connectors docs |
| **Google Gemini** equivalent | Remote MCP is documented as a section; **no tool-search/deferred-loading primitive found**; instead published *guidance* to keep the active set to 10–20 tools (negative finding for the primitive) | Gemini function-calling docs |
| **Cloudflare** | **Code Mode** shipped in the Cloudflare Agents SDK (see §5) — a code-execution transform rather than a search primitive | Cloudflare blog + `codemode` docs |

The Agent SDK's default-on behaviour is the most operationally significant row, because it means the technique is not an opt-in a team must design for — it is **already active** in that harness, with an `auto` mode that counts deferrable definition tokens and activates tool search when they reach a share of the model's window (documented default: **10%**). Documented fallbacks and blockers matter for platform teams: it is **not supported on Microsoft Foundry deployments hosted on Azure** and is rejected server-side; the SDK disables it when `ANTHROPIC_BASE_URL` points to a **non-first-party host**, "since most proxies don't forward `tool_reference` blocks"; and earlier models load upfront. Those are not footnotes — they are the reason a disclosure strategy must be **tested per deployment path**, not asserted from a vendor page.

### 4.3 The reported results, with their sources labelled

- **Vendor claim (Anthropic, 24 Nov 2025):** "This represents an **85% reduction in token usage**," with ~77K → ~8.7K tokens of context consumption and a diagram caption giving 191,300 vs 122,800 preserved tokens. **[Vendor claim — illustrative, no published methodology.]**
- **Vendor claim (Anthropic, 24 Nov 2025):** accuracy on "MCP evaluations" with large tool libraries improved "**Opus 4 from 49% to 74%**, and **Opus 4.5 from 79.5% to 88.1%**." No eval name, sample size, baseline definition, or error bars published. **[Vendor claim — internal testing, not reproducible from the post.]** It is *directionally consistent* with the independent paper in §7, which is worth noting but is not corroboration.
- **Vendor claim (docs):** the "30–50 tools" degradation threshold, quoted in §2.4. **[Vendor claim.]**
- **Independent:** the BoR paper's downstream validation — adaptive disclosure at **93.1% vs 87.1%** tool-choice versus always-five, widening to **76.8% vs 60.9%** where the correct tool was present but not top-ranked. **[Independently measured, arXiv:2605.24660, May 2026.]** This is the strongest independent support for the *direction* of the vendor's claim, and the strongest available caution about *magnitude*.

### 4.4 The mechanical detail that undercuts the naive cost story

The API documentation is unusually blunt about a detail that most secondary write-ups get wrong:

> "`defer_loading` controls what enters the context window, not what you send in the request… You still send every tool's full definition in the `tools` array on every request, including the deferred ones. The API needs them server-side to run the search and expand `tool_reference` blocks."

**Deferral therefore saves *model context* and *cached-prefix budget*; it does not save request payload bytes.** For a cost model this is the difference between "our egress drops by 55K tokens per turn" (false) and "our prompt prefix no longer contains the definitions, so we are not billed for them as input on every turn, and we never pop the cache" (true). It also has a governance implication: the definitions are still *transmitted* to the provider on every request, so "the tool never leaves our boundary" arguments based on deferral are wrong.

### 4.5 Documented limits and the parameters worth knowing

Precise, citable, and subject to change — re-read the page:

- **Maximum deferred tools: 10,000 per request.**
- **Search results: up to 5 by default**; the model may set `limit` from **1 to 10,000**.
- **Regex pattern length ≤ 200 characters; BM25 query length ≤ 500 characters.**
- **Metering:** tool search "isn't metered as a separate server tool… the tool definitions that search loads into context **count as input tokens** like any other tool definition." Cost therefore *moves* from every turn to the turns where a tool is loaded — it does not vanish.
- **Model support:** the compatibility table excludes Claude Opus 4.1 and earlier.
- **Caching:** deferred tools are excluded from the system-prompt prefix and are appended **inline as a `tool_reference` block in the conversation history** when discovered, so the prefix is untouched — "adding tools dynamically through tool search **does not break your cache**." Strict/grammar mode also composes: the grammar is built from the full toolset regardless of deferral.

### 4.6 Failure modes specific to deferred loading

1. **The wrong search query selects the wrong tool.** The model must translate a task into a query; the query is the new failure point. The vendor names similar tool names as the most common failure source — "especially when tools have similar names like `notification-send-user` vs. `notification-send-channel`."
2. **The loss of cross-catalogue reasoning.** A model that cannot see a tool cannot reason that it *should* use it. Deferral trades "choosing badly from everything" for "not knowing something exists." The independent evidence is blunt here: on a 3,251-tool catalogue, a fixed shortlist of 5 found **nothing** on hard queries where the correct tool ranked 6th–20th (§7).
3. **The discovery round-trip latency.** Documented qualitatively: the feature "adds a search step before tool invocation, so it delivers the best ROI when the context savings and accuracy improvements outweigh additional latency." **Note what is missing: no published measurement of the latency penalty in milliseconds, and no published net-cost crossover point in turns per session** (negative finding). Treat any latency number you see elsewhere as unsourced.
4. **Breadth chosen by the agent.** Because the agent sets `limit` itself, search breadth becomes a silent failure surface: too narrow and the tool is never found; too broad and you re-create the bloat.
5. **Deferral where deferral is pointless.** Both vendor surfaces state the technique is **less beneficial** with a small tool library (under ~10 tools), when every tool is used in every session, and when definitions are compact (documented guidance: standard calling is better "when your tool definitions are small — less than 100 tokens total"). A blanket policy mandating deferral contradicts the primary source.

### 4.7 Designing a good index

The published guidance is thin but concrete, and it is all about *naming and grouping*, not about the search algorithm:

- **Namespace tool names by service** (`github_`, `slack_`) "so one search matches the whole group."
- **Prefer names that surface the intent**, not the mechanism: the SDK docs note that a name like `search_slack_messages` surfaces for more requests than `query_slack`.
- **Give the model a category map.** Both vendor surfaces recommend adding a system-prompt section that names the available tool categories, so the model has a vocabulary for searching before it has the definitions.
- **Keep descriptions discriminative.** The measurement literature is clear that descriptions drive selection and that small perturbations shift choices (BiasBusters, ICLR 2026). In a search-driven regime, a description is not documentation — it is the retrieval key.

---

## 5. Code Execution and the Tool-as-API Pattern

### 5.1 The mechanism, in two vendor formulations

**Anthropic's formulation ("Code execution with MCP," 4 November 2025).** Instead of exposing tools as tool-call definitions, present MCP servers as a **code API**: Anthropic's example generates a **file tree of tool files** from connected servers — TypeScript files named after the tools, each exporting an async function that calls into the MCP layer. The agent **discovers tools by exploring the filesystem**: it lists `./servers/` to see which servers exist, then reads only the specific tool files it needs. Anthropic's post contains a section literally headed **"Progressive disclosure"**, reasoning that "models are great at navigating filesystems. Presenting tools as code on a filesystem allows models to read tool definitions on-demand, rather than reading them all up-front."

**Cloudflare's formulation ("Code Mode: the better way to use MCP," by Kenton Varda and Sunil Pai).** "Convert the MCP tools into a TypeScript API, and then ask an LLM to write code that calls that API." When you connect a server in code mode, the Agents SDK fetches the server's schema and converts it into a typed TypeScript API with doc comments. Cloudflare's stated rationale is a competence argument rather than a token argument: "LLMs have seen a lot of code. They have not seen a lot of 'tool calls.'" Anthropic's post explicitly credits Cloudflare and says "the core insight is the same." (*Cloudflare post's own publication date could not be established from the page* — flag; it predates 4 November 2025 only by inference from Anthropic's citation.)

Both are **shipped**: Cloudflare states it extended the Cloudflare Agents SDK with a `codemode` helper, and Anthropic documents a second, complementary mechanism in the same post — a **`search_tools` tool with a `detail level` parameter** letting the agent choose disclosure granularity (name only / name and description / full definition). **Note the discrepancy:** the graded three-level ladder is **blog-described**; the **shipped** API expansion is effectively binary (deferred or fully loaded, expanded to the full schema on discovery). Distinguish a documented pattern from a shipped primitive.

### 5.2 The token economics — what leaves the context

Three distinct savings, which is why the reported figures for this technique are the largest in the family:

1. **Definitions leave the prompt.** Tool schemas become files (or API surfaces) read on demand rather than text in the prefix. This is the source of the headline "150,000 → 2,000 tokens" vendor illustration (98.7%, Anthropic, 4 Nov 2025 — a vendor illustration, not a benchmark).
2. **Results are filtered in the execution environment.** A tool that returns 10,000 rows can be reduced to the five that matter *before* anything crosses into context, because the filtering happens in code. This is a large, often-underestimated saving; the vendor's own example is a spreadsheet of thousands of rows reduced to a handful of logged entries.
3. **Intermediate state lives in files.** Long workflows can pass data between steps through the filesystem rather than through the conversation, so a two-hour transcript does not have to flow through context twice — the vendor's illustrative example puts that second pass at "an additional 50,000 tokens."

### 5.3 The security consequences — the honest version

This is the technique with the largest security delta, and the vendor says so itself:

> "code execution introduces its own complexity. Running agent-generated code requires a **secure execution environment with appropriate sandboxing, resource limits, and monitoring**. These infrastructure requirements add operational overhead and security considerations that direct tool calls avoid."

The implications for an estate:

- **You are now running model-authored code.** That is a different risk class from model-authored JSON arguments. The blast radius of a malformed argument is a failed call; the blast radius of generated code is whatever the sandbox permits.
- **The sandbox becomes load-bearing infrastructure.** It is not a nice-to-have: without it, the technique cannot be deployed at all.
- **The filesystem is a capability surface.** If tools are presented as files, then file read/write permissions *are* tool permissions, and path traversal, symlink, and write-scope questions become agent-security questions.
- **Data residency and egress change shape.** Filtering in the environment means raw data need not enter context, which can be a *privacy* improvement — but it also means sensitive data now flows through an execution environment you must govern.

The repository's dedicated treatment of sandbox design, isolation boundaries, and execution policy lives in [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md); this guide does not re-derive it. What belongs here is the disclosure-specific point: **code mode moves the tool surface's trust boundary from "what definitions did we show the model?" to "what can this code reach?"** — and the second question is answered by infrastructure, not by a prompt.

### 5.4 Operational requirements

Minimum viable deployment: a **sandboxed runtime** with resource limits and monitoring; a **stable generation step** that produces the tool surface (file tree or typed client) from the servers' schemas and *re-generates deterministically*, since a non-deterministic surface is a cache-invalidating surface; **state persistence** for intermediates; and a **skill-ification path** — Anthropic notes that saving working code as reusable functions and adding a `SKILL.md` file turns it into a structured skill, which is the bridge to §6.

### 5.5 Maturity assessment

**Code mode is real, shipped, and vendor-attributed across two vendors — and it is not yet a commodity pattern.** Evidence for that judgement: two vendor blog posts, shipping in two SDKs (Cloudflare Agents SDK; Anthropic's documented patterns), no independent measurement of its token savings found in this research pass, and no published end-to-end evaluation comparing code mode against tool search. Its strongest advocates describe it as architecture rather than product: Cloudflare's two claimed benefits are labelled by Cloudflare as "striking" **with no numbers published in the post** (vendor claim). Anthropic's operational caveat — that the technique requires infrastructure that direct tool calls avoid — is the honest counterweight. **Adopt it where you already run a sandbox for other reasons; do not make it your first disclosure move.**

---

## 6. Instruction-Level Disclosure — The Skills Pattern

### 6.1 The layered format

The skills pattern, as documented by its originator and now published as an **open standard** (Agent Skills, announced 16 October 2025; published as an open standard on 18 December 2025 at agentskills.io), is a three-tier loading model:

| Level | What loads | When | Documented budget |
|---|---|---|---|
| **Level 1 — Metadata** | `name` and `description` from YAML frontmatter, for **every** installed skill | At startup, always, into the system prompt | **~100 tokens per skill** |
| **Level 2 — Instructions** | The `SKILL.md` body | When the task matches the description — the agent reads the file from the filesystem | **Under 5K tokens** |
| **Level 3+ — Resources and code** | Bundled reference files; executable scripts | Only when the agent chooses to read/run them | **None until accessed** — scripts run and "only their output… consumes tokens" |

The documented authoring constraints are what make it a retrieval format rather than a document format: `name` is capped at **64 characters** (lowercase, hyphens, no reserved words), `description` at **1,024 characters**, and the body is recommended to stay **under 500 lines**, with overflow split into separate files using the disclosure patterns. The documentation also states the selection scale plainly: "The `description` is critical for skill selection: Claude uses it to choose the right Skill from potentially **100+ available Skills**."

### 6.2 Why the descriptor must carry enough signal

The descriptor is the *only* artifact guaranteed to be in context, and it is simultaneously the *trigger* for everything behind it. That makes it the highest-leverage 100 tokens in the system, and it creates a specific failure asymmetry:

- **Under-specified descriptor → the skill never loads.** The failure is silent: the agent simply does not do the thing, and nothing errors. In an evaluation, this looks like a capability that "doesn't work" rather than a retrieval miss.
- **Over-broad descriptor → the skill always loads.** Now every task pays the body's tokens, and the technique has degenerated into loading everything up front with extra steps.

The practical fix is the same discipline that applies to tool descriptions (§9): name the *task and the trigger conditions*, not the technology. "Use when a PDF must be filled from structured data and then flattened" beats "PDF utilities."

### 6.3 Failure modes of a skill library

- **Skills that conflict.** Two skills whose descriptions match the same class of task will compete; the model picks one, non-deterministically across runs. Nothing in the format prevents overlap, so overlap is a review-time concern.
- **The maintenance burden of a growing library.** The per-skill metadata cost is small (~100 tokens) but it is **always present**: 200 skills is ~20,000 tokens of metadata before anything runs. Skills reduce the *marginal* cost of an instruction from its full body to its descriptor — they do not make instructions free, and a sufficiently large library re-creates a smaller version of the same problem.
- **Behavioural defects shipped as content.** A skill is an instruction; a wrong skill is a behavioural change, and the format's portability (an open standard, a zip upload, a directory on disk) means the review workflow is the only gate.
- **The vendor's own security warning** is on the record: malicious skills "may introduce vulnerabilities in the environment where they're used or direct Claude to exfiltrate data," and installations should come only from trusted sources.

### 6.4 How this differs from tool disclosure

|  | Tool disclosure | Instruction disclosure |
|---|---|---|
| Artifact withheld | A JSON Schema defining *how to call* | A procedure defining *how to work* |
| Trigger | Search over names/descriptions/arguments | Description matching against the task |
| Failure if trigger misses | The model uses the wrong tool or reports inability | The model performs the task without the procedure — silently worse |
| Blast radius if the artifact is bad | An action taken with wrong arguments | Behaviour changed across every task that triggers it |
| Governance question | Was this tool authorised, and what did it do? | What behaviour was the agent taught, and by whom? |

### 6.5 Why this transfers to enterprise platforms most readily

Of the six families, this is the one an enterprise can adopt without buying anything. It requires no provider-hosted primitive, no gateway, no sandbox, and no retrieval service — it requires a **format convention and a review workflow**. That is why it is the pattern most likely to be the first disclosure mechanism adopted inside a regulated institution: the cost of a pilot is a filesystem and a naming standard, and the governance artifact it produces (a reviewed, versioned, human-readable instruction file) is exactly what a change-control process already knows how to handle. It also composes: both vendor implementations document skills alongside MCP rather than instead of it — Anthropic's code-execution post explicitly notes that saving working code plus a `SKILL.md` creates a structured skill, and the skills documentation requires fully qualified MCP tool names (`ServerName:tool_name`) so that a skill's tool references resolve.

---

## 7. The Measurement Evidence

This section separates what has been *measured* from what has been *asserted*. The distinction is the single most useful thing in this guide, because the disclosure space is unusually polluted with unsourced percentages.

### 7.1 What has been measured — the independent literature

| Study | Venue / date | Catalogue | Measured result | Methodological caveat |
|---|---|---|---|---|
| **RAG-MCP** (Gan & Sun), arXiv:2505.03275 | preprint, **6 May 2025** | Author-simulated "MCP stress test" catalogue | Prompt tokens cut **>50%**; tool-selection accuracy **43.13% vs 13.62%** baseline | Simulated catalogue; selection accuracy, not task success; the baseline is deliberately degraded by bloat — do **not** read 3× as a production expectation |
| **"How Many Tools Should an LLM Agent See?"** (Repantis et al.), arXiv:2605.24660 | preprint, **23 May 2026** (v2 7 Jun 2026) | BFCL **370 tools**; ToolBench **3,251 tools** | BFCL: adaptive policy matches always-50 coverage (**90.3% vs 90.8%**) showing only **7 tools on average**. ToolBench: fixed shortlist of 5 has *higher aggregate coverage* (**64.7% vs 61.9%**) but finds **nothing (0%)** on hard queries where the correct tool ranked 6th–20th; the deeper-searching agent finds **16.7%**. Downstream with Claude Sonnet 4.6: **93.1% vs 87.1%** adaptive vs always-5, widening to **76.8% vs 60.9%** on medium-difficulty queries | The authors describe their RL agent as "deliberately simple, serving as a probe of the metric rather than a proposed system" — this is a **metric demonstration**, not a shipped technique. Selection benchmarks, not end-to-end task success |
| **MetaTool** (Huang et al.), arXiv:2310.03128 | preprint, **4 Oct 2023** (v6 Dec 2024) | Tool-usage awareness + selection (ToolE dataset) | Across eight popular LLMs, "the majority… still struggle to effectively select tools" | Predates the MCP era; catalogue not MCP-derived. Its recommendation — rewrite tool descriptions **per downstream model** — is the durable finding |
| **BiasBusters** (Blankenstein et al.), arXiv:2510.00307 | **ICLR 2026** camera-ready | Functionally equivalent tool sets | Across seven LLMs: models fixate on one provider and **favour tools appearing earlier in context**; **small perturbations to descriptions significantly shift choices**; filtering to a relevant subset then **sampling uniformly** reduces bias while maintaining coverage | Peer-reviewed. Benchmarks *bias*, not disclosure per se — but it directly evidences that ordering and wording are load-bearing |
| **MCP-Atlas** (Bandi et al.), arXiv:2602.00933 | preprint, **31 Jan 2026** (v3 19 May 2026); Scale leaderboard | **1,000 human-authored tasks, 36 real MCP servers, 220 tools**, 20 frontier models | Pass rates up to **82.2%** at a 0.75 claim-coverage threshold; **63.3% of diagnosed failures are cognitive rather than tool-call related**; several strong models fail *after* successful execution due to premature stopping or bad synthesis | The **real-server** upgrade the earlier studies lack — and the corrective to disclosure optimism |
| **MCP-Bench** (Wang et al.), arXiv:2508.20453 | **NeurIPS 2025 Workshop** (not main conference) | 28 server implementations; **single-, two-, and three-server** configurations | Leaderboard scores from 0.428 to 0.749 across models (snapshot, Sept 2026) | Varies server count as an axis — but only 1/2/3, so it is not a scaling curve. Live leaderboard: re-read it, don't quote this table |
| **ToolScope** (ACL 2026 Long Papers) | peer-reviewed, 2026 | Tool-use benchmarks | **8.38% to 38.6%** tool-selection improvement from **merging** overlapping tools | Abstract-level verification only in this pass. The opposite lever to disclosure: shrink the catalogue instead of hiding it |

### 7.2 The three findings that should change a decision

1. **Fixed-depth disclosure demonstrably fails.** A shortlist of 5 scored **0%** on hard queries in a 3,251-tool catalogue. Any design that hard-codes "show 5 tools" is not a safe implementation — it is a measured failure mode on hard requests. Adaptive depth matched a much wider baseline while showing far fewer tools in the 370-tool case, which is the shape of the right answer.
2. **Ordering and wording are first-class variables.** Independent, peer-reviewed work shows position bias in tool choice and sensitivity to description perturbations. This is why namespacing, generation-order determinism, and description review are not cosmetic in a disclosure regime — they are the mechanism.
3. **Tool-surface optimisation addresses a minority of the failure mass.** In the one benchmark built on **real** MCP servers with **semantically plausible distractors** — 1,000 tasks, 36 servers, 220 tools, 20 models — **63.3% of diagnosed failures were cognitive, not tool-call related**: task understanding, synthesis, parsing, stopping. If that replicates, the ceiling on what any disclosure technique can buy is roughly the tool-attributable share. The honest reading is that progressive disclosure fixes a real and measurable problem *and* is not the main reason agents fail.

### 7.3 Which results are vendor-published versus independent

- **Vendor-published:** every token-reduction headline ("85%", "98.7%", "55K tokens", "134K tokens"), the 30–50 tool degradation threshold, the Opus 4 / Opus 4.5 accuracy uplift, and the per-server token table. **All of these are single-vendor, method-unpublished claims.**
- **Independent:** the BoR adaptive-depth results and the downstream 93.1/87.1 and 76.8/60.9 comparisons; the MCP-Atlas and MCP-Bench competency benchmarks; the BiasBusters ordering/description findings; RAG-MCP's retrieval numbers; ToolScope's merging numbers.
- **Where the studies disagree or fail to align:** the vendor threshold ("30–50 tools") is an assertion; the independent work finds *task-difficulty dependence* rather than a clean count threshold — the correct framing is "depth must adapt to the query," not "there is a magic number." RAG-MCP's dramatic accuracy delta is a consequence of an artificially bloated baseline, while BoR's numbers are smaller and more credible for planning. MCP-Atlas's cognitive-failure finding cuts against the whole thesis's magnitude. **These are not contradictions to resolve but different measurements of different things** — and any summary that flattens them into one number is wrong.

### 7.4 What remains unmeasured

- **No study found combines (i) real MCP servers, (ii) systematically varied catalogue size, and (iii) end-to-end task success.** This is the central methodological gap: any claim that "X tools breaks real MCP agents" is, on the evidence available, an **extrapolation from simulated catalogues**.
- **No measurement of the search round-trip's latency in milliseconds**, and **no published crossover point** in turns per session at which deferral pays for itself.
- **No independent, reproducible third-party measurement campaign of MCP tool-definition token cost** with a published script and raw inventory.
- **No published census of tool counts across popular MCP servers.**

### 7.5 The methodological traps, and the marketing

**Traps to check in any benchmark you are handed:**

- **Simulated catalogues.** A tool catalogue assembled by the authors can be tuned; a real catalogue has the ugly overlaps and semi-descriptive names that a synthetic set lacks.
- **Selection accuracy evaluated separately from task success.** Picking the right tool from a shortlist is not finishing the task. The strongest independent study here reports selection; the real-server benchmark reports task success; **the two are never measured together at scale.**
- **Coverage versus selection.** Whether the right tool *appeared* in the shortlist and whether the model *picked* it are different metrics, and papers that report only coverage overstate the technique's effect on behaviour.
- **Internal testing as a citation.** "Internal testing showed…" with no eval name, dataset, size, or baseline is a claim, not a result.

**Marketing that should be rejected outright:** figures of the form "accuracy degrades to 85–91% at 5 tools, then 65–78% at 20+ tools" and "parameter-mismatch errors account for 60–75% of tool-calling failures at scale" circulate on commercial research-marketing pages **with no dataset, model list, or method**. No traceable primary measurement for those specific ranges was found in the research behind this guide. **Do not cite them.** Likewise, treat any figure bearing the shape of "the GitHub MCP server costs exactly N tokens" as a restatement of Anthropic's November 2025 estimate unless it publishes its own tokenizer and inventory.

---

## 8. The Trade-offs and the Decision Framework

### 8.1 The trade ledger

**Latency versus context cost.** Deferred disclosure is not free; it is a *purchase*. You spend a search round trip (and, in the tool-search case, a server-side search) to buy back prompt tokens and — at large catalogues — selection accuracy. The vendor states the qualitative trade ("the best ROI when the context savings and accuracy improvements outweigh additional latency") and publishes **no latency measurement**, so the trade must be measured locally on your own stack. For a small catalogue the trade is clearly negative: both vendor surfaces recommend loading everything up front below roughly ten tools or a hundred tokens of definitions.

**The loss of cross-catalogue reasoning.** This is the deepest cost and the one most often glossed over. A static surface lets the model *see* everything and reason about the whole space: "the user asked about invoices — I have an ERP tool, and also a mailbox search that might hold the PDF." A disclosure regime replaces seeing with searching, which means the model must **know what it doesn't know** before it can look. The measured version of this cost is the 0% result on hard queries with a fixed narrow shortlist. Guardrails against it: keep the always-loaded set to the tools that are plausible *in almost any session*; advertise categories in the system prompt; and never let the agent's search breadth be a hard constant.

**The added failure surface.** A retrieval step is a new thing that can be wrong, and its failure is *silent*: a missed tool doesn't raise an exception, it produces an agent that reports it can't do something. Every disclosure mechanism therefore needs its own evaluation layer (retrieval recall, trigger rate, search-hit quality) separate from task-success evaluation — and that evaluation must be run against the *deferred* catalogue, not the visible one.

**The caching interaction.** Restated as a rule, because it changes designs: tool definitions sit in the cached prefix, so **any change to the set or order of definitions invalidates the whole cache** — tools, system, and messages. Deferral is cache-safe because the deferred definitions are never part of the prefix and arrive as inline conversation content; removal is not. Non-deterministic ordering is a documented cause of production cache breakage. A tool-surface regeneration step that is not deterministic is therefore a cost incident waiting to happen. (General prefix-stability theory: harness guide.)

**Tooling and platform coupling.** Provider-hosted tool search is a platform feature with platform-specific exception lists: documented as unsupported on one cloud's hosted deployment, disabled behind non-first-party base URLs "since most proxies don't forward `tool_reference` blocks," and gated by model version. Building a disclosure strategy that depends on one provider's primitive buys a real capability and accepts a real portability constraint — a constraint that is *hidden until you change gateway or cloud*.

**The observability cost.** When the tool surface is static, "what could the agent do?" is answered by reading a config file. When it is dynamic, it is answered by *per-step* telemetry: which definitions were visible, which search ran, what it returned, which tool was expanded and invoked. Anything less and you cannot reconstruct the capability surface at the moment of a decision. This is the price of the *and* in "dynamic and auditable," and it is the least mature part of the tooling landscape.

### 8.2 The decision framework

This framework is the guide's own labelled analysis — a synthesis of the documented guidance and the measured evidence, not a quoted standard.

**Step 0 — Measure before choosing.** Inventory your servers and tools; tokenise the definitions with the tokenizer of the model you actually run; and measure the cache hit rate you have today. Every decision below keys off numbers you produce, because no external number is both current and yours.

**Case A — small catalogue (fewer than roughly a few dozen tools, or definitions well under ~10K tokens).** **Do not add a disclosure layer.** Both vendor doc sets say deferral is a poor fit here, and the independent work shows position/description effects matter more than hiding things. Spend the effort on tool *design* instead: fewer, more expressive tools (a documented harness-guide principle), discriminative descriptions, and names that describe intent. The honest answer for many teams in this band is: the problem is your tool descriptions, not your disclosure architecture.

**Case B — medium catalogue (tens to low hundreds of tools, one or two servers).** **Hierarchical namespacing and grouping first, code-execution results filtering if you already run a sandbox, hosted tool search second.** Namespacing is cheap, requires no infrastructure, improves both search quality and selection, and is documented as the vendor's own recommendation. Tool search becomes justified when definitions exceed roughly 10K tokens or selection problems are observed — and note that current agent SDK defaults may already be acting on this for you, which is a reason to know your harness's default rather than to assume you have no disclosure layer.

**Case C — large catalogue (hundreds to thousands of tools, multi-server).** **Deferred loading with tool search is the intended design**, composed with namespacing for index quality and a stable always-loaded core. Requirements: pin the tools that must always be visible; keep the surface deterministic so the cache survives; instrument the search step; and evaluate retrieval recall and task success separately. Budget for the discovery round trip explicitly in latency SLAs.

**Case D — multi-server enterprise estate.** **Gateway aggregation for the governance boundary plus tool search for the context boundary, with instruction-level disclosure for procedures.** The gateway answers "which servers may this team reach, with what credentials, under whose approval?" — a server-level, profile-based control. Tool search answers "which definitions enter this context window this turn?" — a tool-level, inference-time control. Instruction disclosure answers "what procedures does this agent know, without paying for all of them?" These operate at three different layers and do not substitute for each other. Very large toolset aggregations are the case the vendor explicitly names as the point of tool search (200+ tools).

**Case E — the honest answer is to fix the tool design.** If a catalogue is large because one server exposes 40 near-duplicate tools with inconsistent naming, disclosure is a way of hiding a design defect that will keep producing selection errors — just quieter ones. Tool merging (published improvements of 8.38%–38.6% in selection accuracy in peer-reviewed work) and read-only/default-deny scoping (the GitHub MCP server's documented `--toolsets`, `--tools`, `--read-only` design, whose stated purpose is to "help the LLM with tool choice and reduce the context size") attack the problem at the source. **A disclosure layer that masks a bad inventory converts a visible problem into an invisible one**, which is the worst of both worlds in a regulated estate.

**Threshold triggers, if a policy is needed:** definitions above ~10K tokens, an observed selection-accuracy problem, a multi-server deployment, or more than ~10 tools — the vendor's own stated triggers — with an explicit exemption for small, compact, always-used catalogues. The rule must be **threshold-triggered, not blanket**, precisely because the primary sources say so.

---

## 9. Design Guidance

### 9.1 Writing tool names and descriptions that survive a search step

A description serves two masters in a disclosure regime: it must be **retrievable** (match a query the model will actually generate) and it must be **discriminative** (beat its near-neighbours once retrieved). Practical rules, each traceable to a source in this guide:

- **Name the intent, not the mechanism.** `search_slack_messages` surfaces for more relevant requests than `query_slack` (vendor SDK documentation's own example).
- **Namespace by service.** `github_`, `slack_`, `jira_` — so that a single search for a service matches the whole family (explicit vendor recommendation).
- **Make near-duplicates unmistakable.** The vendor names tool pairs like `notification-send-user` vs. `notification-send-channel` as *the* most common failure source. If two tools differ by one word, the description must carry the disambiguation, not the name.
- **State when to use it and when not to.** Scope words ("use for reconciliation only; not for submission") are what prevent a retrieved tool from being called in the wrong situation.
- **Do not rely on schema richness to disambiguate.** Descriptions are what search and selection read; a beautifully typed schema that nobody matches against is inert.
- **Remember that descriptions are model-specific artifacts.** Independent work (MetaTool's recommendation) suggests rewriting descriptions for the downstream model. A description optimised for one model can be misleading for another, which makes description changes **behaviour-affecting changes** — see §9.4.

### 9.2 Grouping and namespacing as an architecture decision

Choose the grouping axis by **how requests arrive**, not by how your org chart is drawn. Three viable axes: by system (`github_`, `jira_`), by domain (`payments_`, `reference_data_`), or by verb family (`search_`, `create_`, `approve_`). The test: when the model formulates a search for a real user request, does one group name plausibly appear in the query? If not, the grouping is decorative. Document the convention in one place, and enforce it mechanically — a naming convention that lives only in a wiki drifts within a release cycle.

### 9.3 Versioning and stability of a tool surface

Treat the tool surface as a **published interface with a cache contract**:

- **Deterministic ordering.** Required in spirit by the 2026-07-28 specification's own instruction that servers SHOULD return tools in a deterministic order to improve prompt cache hit rates, and required in practice because non-deterministic ordering is a documented cause of production cache breakage.
- **Additive over mutating.** Adding a tool behind a stable ordering (when the new tool sorts at the end or the surface is stably sorted) is far cheaper than renaming or re-scoping an existing one, because a rename is both a cache event and a behaviour event.
- **Renames are migrations.** A renamed tool breaks skills that reference it by fully qualified name (`ServerName:tool_name`), breaks saved code in a code-mode surface, and breaks any pinned definition hash. Version the change, keep an alias through a deprecation window, and re-run the trigger-rate evaluation afterwards.
- **Hash the whole schema, not the description.** The documented compensating control for post-review mutation is to pin a hash over the canonical JSON of name, description, *and* input schema, and re-verify before execution.

### 9.4 Testing a disclosure regime

What to measure — in three layers, because they fail independently:

1. **Retrieval/trigger layer.** For a labelled set of representative tasks: did the search return the correct tool in the top-k (recall)? For skills: what fraction of tasks where the skill *should* have triggered actually triggered it (trigger rate), and what fraction of tasks where it shouldn't have, did (false-trigger rate)?
2. **Selection layer.** Given the returned shortlist, does the model pick the right tool? This is where published measurement exists and where regressions are cheap to detect.
3. **Task layer.** Does the end-to-end task succeed? This is the layer with almost no public benchmarks and the one your business cares about; it needs the deferred catalogue in the loop, which means a test harness that exercises the real client.

**The silent regression to hunt:** *the agent stops finding a tool it used to use.* The signature is a task-success drop with no errors, no exceptions, and no change to any tool the evaluator can see — because the failure is in the index, or in a description that was edited for clarity, or in a group rename. Detection requires a **canary suite that asserts specific task→tool outcomes** (not just task success), run on every tool-surface change, with the *deferred* surface active. If your evaluation only measures the tools that are visible, you have an evaluation blind spot by construction — the benchmark measures the visible tools only, which is exactly the failure this section exists to prevent.

### 9.5 Migration pattern for an existing catalogue

1. **Inventory and measure.** Enumerate servers, tools, and tokens; find the top 20 tools by invocation frequency. This set becomes the always-loaded core.
2. **Fix names and descriptions first.** Cheapest, highest-yield, and it improves every downstream mechanism. Do it before adding machinery.
3. **Introduce the index in shadow mode.** Run the search layer alongside the static surface and compare what *would* have been returned against what was actually used. This produces your recall baseline without changing behaviour.
4. **Defer in tiers, from the tail.** Defer the least-used servers first, keep the core pinned, and keep ordering deterministic. Each tier is a measurable step with a rollback.
5. **Re-baseline the cache.** Expect the cache-hit profile to change; instrument it, and treat unexpected cache-miss-rate movement as an incident, as the vendor does.
6. **Instrument the discovery path.** Log every search query, its results, and the expansion; without this you cannot debug a missed tool.
7. **Only then** consider code-execution surfaces for the workloads with heavy tool results, since that technique adds a sandbox to the critical path.

---

## 10. The Failure Modes

A catalogue with symptom, cause, and guardrail. Repository-wide agent-failure and drift catalogues are owned elsewhere — see [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [ai_agent_drift_guide.md](ai_agent_drift_guide.md); this table is deliberately scoped to disclosure-specific failures and does not duplicate them.

| Failure | Symptom | Cause | Guardrail |
|---|---|---|---|
| **The never-found tool** | Agent reports it cannot do something it used to do; no error anywhere | The search query missed it, or the agent never searched; measured worst case is a fixed shortlist of 5 scoring 0% on hard queries | Adaptive search breadth; canary suite asserting task→tool outcomes; log every search and its results |
| **Wrong-tool selection** | Action taken with the wrong system's tool; often near-duplicate names | Similar names (`…-user` vs `…-channel`), weak or non-discriminative descriptions, position bias | Disambiguating descriptions; namespace prefixes; description review as a change-controlled artifact |
| **Discovery round-trip latency spiral** | Task latency rises while token cost falls; agent searches repeatedly | Search breadth too narrow → miss → refine → search again; no caching of discoveries within a session | Set sensible default breadth; keep discovered tools available until compaction; budget the round trip in the SLA |
| **The stale index** | A tool exists and is reachable but never retrievable | Index built at startup and not refreshed; the protocol's own change-notification surface (`notifications/tools/list_changed`, `subscriptions/listen`) not wired up | Subscribe to list-change notifications; refresh the index on change; or use cache TTLs (`ttlMs`/`cacheScope`) deliberately |
| **Description drift after a rename** | A previously reliable trigger stops firing; skills reference a tool that "doesn't exist" | Rename or re-scope without updating skills, saved code, or pinned hashes | Fully qualified names in skills; alias windows; re-run trigger evaluation after any rename |
| **Hidden tool surface evades review** | A tool that was never seen in review is found invoked in logs | Deferred definitions are not visible at connect time; review happens at connect, execution at runtime — a documented trust gap, widened by disclosure | Hash-pin the full schema and re-verify before execution; review the *catalogue* at the index layer, not just what is loaded |
| **Evaluation blind spot** | Green dashboard, production misses | Benchmark measures the visible tools only, or measures selection without task success | Evaluate against the deferred surface; separate retrieval, selection, and task metrics |
| **Disclosure masking a bad inventory** | Fewer tokens, same or worse task success, quieter errors | 40 near-duplicate tools hidden instead of consolidated | Tool merging / consolidation; default-deny scoping; review whether the catalogue should exist in this shape |
| **Cache invalidation by disclosure** | Sudden cost spike, no functional change | Tools removed mid-session, ordering non-deterministic, a gateway stripping cache markers | Defer, don't remove; deterministic ordering; alert on cache-miss-rate changes |
| **Silent gateway misconfiguration** | Cache markers stripped, entire history billed as uncached on every turn, success returned | An intermediary that removes cache markers while reporting success | Verify cache behaviour end-to-end after any gateway change; treat a marker-stripping gateway as a defect |

---

## 11. The Enterprise and Regulated-Institution Angle

### 11.1 Why a governed platform cannot simply expose everything

An estate that connects twenty internal systems over MCP reaches a tool surface in the hundreds to low thousands. Three independent pressures make "expose it all" untenable:

- **Context cost.** Per the vendor's own table, a five-server setup is already ~55K tokens before the conversation starts, and a single additional server was cited at ~17K. At estate scale the tool block is not a line item, it is the dominant input cost of every turn.
- **Selection accuracy.** The vendor's stated degradation point is 30–50 available tools; Google's published guidance is an active set of 10–20. An estate catalogue is one to two orders of magnitude larger.
- **Privilege.** Not every user, team, or agent should be offered every tool. A tool that appears in a context window is a tool that can be invoked, and the offered surface *is* the practical permission surface from the model's point of view.

### 11.2 Permission and least-privilege implications of dynamic availability

Dynamic disclosure makes least privilege harder in one specific way and easier in another:

- **Harder:** the surface is no longer a static artifact that can be diffed against an approval. A tool can be *reachable* and *unoffered* in one session and *offered* in the next because a search matched it. Scoping therefore has to be enforced **server-side and at the gateway**, not by what the client chose to show — a point the standards material makes explicitly: system-prompt restrictions "are enforced only by the LLM's instruction-following, **not by backend access controls**," and the guidance is "Do not rely on system prompt instructions to restrict tool access."
- **Easier:** deferral is a natural place to enforce default-deny. If a catalogue is disclosed through an index you own, then "which tools may this agent ever load?" becomes a configuration question rather than a hope about prompt behaviour. Combined with a gateway profile, it gives two independent gates.

Note also the supervisory direction of travel: the Financial Stability Board's June 2026 consultation report on responsible AI adoption recommends **applying least privilege to agents and their sub-agents** and adopting **dynamic identity and access management** that grants, changes, or revokes permissions in real time rather than using the static profiles designed for human users. Read carefully: the supervisors are endorsing *dynamic permissions*, not dynamic *disclosure* — the two are frequently conflated in vendor marketing, and the distinction matters when you write a control.

### 11.3 The audit question — can you evidence the offered surface?

This is the hardest question in the guide, and the honest answer is that **no retrieved source provides a template for evidencing a dynamically-disclosed tool surface** (negative finding). The shape of the requirement is nonetheless clear from the governance material:

- The OWASP MCP Top 10 (2025, published as a beta production project, not a settled standard) names **MCP08: Lack of Audit and Telemetry** as a top-ten risk, and prescribes detailed logs of tool invocations, **context changes**, and user-agent interactions with **immutable audit trails**.
- The EU AI Act's Article 12 record-keeping obligation requires that high-risk systems "technically allow for the automatic recording of events (logs) over the lifetime of the system" (in force dates for high-risk categories under the Act's Article 113: **2 December 2027** for Annex III and **2 August 2028** for Annex I — verified on a consolidated-text mirror, and **to be quoted from the official text in any filing**). Note the scope discipline: **no article or recital of the Act explicitly mentions tool permissions, tool descriptions, agent catalogues, or MCP** (negative finding). The relevance is *indirect*: if the tool surface at time of use is not reconstructable, a log of inputs and outputs does not record what the system was *capable of* at that moment. **That is an engineering inference from the article's purpose, not text from the article** — label it as such in any filing.
- The FSB report's audit expectation is the sharpest: for agents executing financial transactions it points to "audit trails of every agent transaction" and "explainability that traces intermediate steps and reasoning paths rather than final outputs alone," while noting that decision-by-decision human review does not scale and that oversight should shift to autonomy boundaries and guardrails ("human-in-command"). Note the provenance: the FSB text itself could not be retrieved in this pass and these points come from **trade-press reporting of the consultation** (11 June 2026) — **cite the FSB document directly in any filing**.

**What a supervisor will ask, in plain terms:** *(1)* Which tools was this agent offered at 14:03 on the date in question, and can you produce that list? *(2)* Were all of them approved, by whom, and when? *(3)* How do you know a tool's definition had not changed between approval and invocation? *(4)* For a deferred tool that was loaded mid-session, what caused it to load? *(5)* If your platform cannot answer (1) and (4) from immutable logs, you do not have an auditable disclosure regime — you have an efficient one.

### 11.4 Change control for a surface that shifts at runtime

The control that answers both the audit question and the review question is the **pinned definition**:

- **Pin a cryptographic hash over the canonical JSON of the tool name, description, and input schema** at discovery/approval time, and **re-hash and compare before each execution**. A mismatch indicates post-deployment mutation — the "rug pull" the OWASP cheat sheet names — and should **fail closed**, not warn.
- **Re-prompt for consent when a tool definition changes**, and never silently accept a changed schema under an unchanged name.
- **Review at the index layer as well as the connect layer.** Because deferred definitions are not visible at connect time, the reviewable artifact must be the **catalogue** (the index), not the loaded set — otherwise review covers a subset that never includes the tool that will eventually be invoked.
- **Treat description edits as behaviour changes.** Independently measured: small perturbations to descriptions shift selection. A description change is not a documentation edit; it is a functional change requiring evaluation.

### 11.5 Cross-references, condensed

Governance frameworks, model risk management, and AI assurance workflows are owned by [ai_governance_framework_guide.md](ai_governance_framework_guide.md), [implementing-responsible-ai.md](implementing-responsible-ai.md) and [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md); agent guardrails and injection controls by [llm_guard_models_guide.md](llm_guard_models_guide.md) and [prompt_injection_guide.md](prompt_injection_guide.md); sandbox design by [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md); and the platform view of where a tool registry and gateway sit by [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md). This section stops at what disclosure specifically changes for those controls: **the surface is dynamic, so review must be catalogue-level, enforcement must be server-side, and evidence must be per-step.**

---

## 12. Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is a fictional institution used across this repository as the worked-example persona. Every tool count, token figure, cost, and date in this section is **invented for illustration**, chosen to be plausible, and is marked as illustrative wherever it appears. Nothing here is a measurement, a benchmark, or a vendor figure, and none of it should be cited as evidence. Real vendors, specifications, frameworks, and products named elsewhere in the guide are named factually; **no real bank appears as the example.**

### 12.1 The situation

Cymbal Bank's governed agent platform has connected its internal systems to agents over MCP. The estate grew by attachment rather than by design: a treasury team's payment-status server, a trade-finance document service, a customer-servicing CRM connector, a reconciliation service, a reference-data service, a market-data connector, an internal wiki search, a ticketing connector, a document-management service, a notification service, plus several vendor-hosted SaaS connectors for collaboration and project tooling. Sixteen connected servers in total.

The platform team's measured starting point (**all figures illustrative**):

| Measure | Illustrative value |
|---|---|
| Connected MCP servers | 16 |
| Total tools exposed | 812 |
| Total definition tokens (measured locally, tool-call payload) | ~96,000 tokens |
| Always-loaded core tools (used in most sessions) | 31 |
| Tokens for the core alone | ~7,400 |
| Observed tool-selection errors in weekly evaluation | 9.1% of tool calls |
| Sessions in which at least one task failed after a wrong-tool call | 6.4% |
| Cache hit rate on the tool prefix | 71% (fragile — a server reconnect resets it) |
| Cost attributable to tool definitions | 38% of input-token spend |

Two symptoms triggered the work. First, an incident in which a reconciliation agent invoked a **notification** tool instead of a **reconciliation** tool in a near-duplicate naming pair, producing a misleading alert to an operations desk. Second, a cost review showing that the tool block, not the conversation, dominated input tokens.

### 12.2 The chosen combination, and why

The team deliberately did **not** adopt one technique. The reasoning, layer by layer (illustrative):

1. **Fix the inventory first — merge and scope (Case E).** 812 tools included 140 near-duplicates across four vendor connectors contributing overlapping capabilities, and a payment server exposing both read and write paths for every entity type under similar names. Consolidation and default-deny scoping removed **310 tools** before any disclosure machinery was introduced. **This step is the reason the rest worked.** Had it been skipped, the disclosure layer would have hidden the design defect rather than fixed it.
2. **Namespace and regroup (Case B/C).** Names were rewritten to `domain_system_action` (`tf_docextract_run`, `rec_match_propose`, `pymt_status_get`), with the **write** tools separated from the read tools by verb so that a search for a status check cannot match a submission tool. Grouping followed *how requests arrive* (task families: reconciliation, payment status, document extraction, customer servicing), not the org chart.
3. **Defer in tiers (Case C).** 502 tools remained after consolidation. **31 tools** — the ones used in most sessions, identified by invocation frequency, not by seniority of opinion — were pinned as always-loaded. The remaining 471 were deferred behind tool search.
4. **Do not adopt code execution yet.** A sandbox existed for other workloads, but the platform team judged that adding model-authored code to the critical path of a payment-adjacent workflow needed its own security review, and — on the evidence — the token savings from deferral plus consolidation were sufficient. Code mode was scheduled as a **phase-two candidate for the document-extraction workflow only**, where large tool results (a 300-page credit agreement) are the dominant cost and result filtering is a genuine win.
5. **Adopt instruction-level disclosure for procedures (Case D).** Twenty-two operational procedures — reconciliation runbooks, exception-handling playbooks, document-extraction templates — were converted to skills with tight descriptors, replacing long system-prompt appendices. This was the lowest-cost, highest-governance-yield step.

### 12.3 The always-loaded index versus the on-demand surface

The design decision with the most consequences was **what stays visible**. The team's rule: *a tool is pinned only if it is plausibly relevant in a session whose task is still unknown.* Concretely (illustrative):

- **Pinned (31 tools, ~7,400 tokens):** customer/account lookup, transaction search, document search, ticketing create/read, notification send (**only** the generic channel tool, with per-channel variants deferred), and the four most-used reconciliation reads.
- **Deferred (471 tools):** everything write-adjacent beyond the narrow pinned set, everything vendor-connector-specific, every per-entity variant, and every tool whose invocation frequency was in the long tail.
- **Never offered (pre-existing):** tools belonging to systems the acting user's role cannot reach at all — enforced at the gateway profile and server-side, not by prompt instruction.

The **index** — the searchable surface — is itself a governed artifact: it is generated deterministically from the approved catalogue on a schedule, its generation is versioned, and a change to the generation logic is a change-controlled event because it can silently change what is findable.

### 12.4 Security and least-privilege redesign

- **Two independent gates.** Gateway profile decides which *servers* an agent group may reach; the platform's tool-scope policy decides which *tools* within those servers may ever be loaded, with default-deny and explicit allowlists for write tools. A tool that is not on the allowlist cannot be disclosed by a search, because it is not in the index.
- **Server-side enforcement.** Restrictions were re-implemented in the backend authorisation layer for each system, on the documented principle that prompt-level restrictions are enforced only by instruction-following.
- **Hash pinning.** Every tool definition is pinned with a SHA-256 hash over the canonical JSON of name, description, and input schema; the platform re-verifies before execution and **fails closed** on mismatch. A quiet description edit by a vendor connector now produces a blocked call and a ticket rather than a changed behaviour.
- **Consent on change.** A changed definition requires re-approval before it becomes loadable, with the diff routed to the tool's owner.
- **Cross-server shadowing reviewed at the index layer.** Because the deferred surface is not visible at connect time, the security team's review artifact is the catalogue export, and a shadowing description is checked across the *index*, not across the loaded set.

### 12.5 Audit and change control

The platform's answer to §11.3's supervisor questions is a per-step disclosure record (**design, not a shipped product**):

| Question | Artifact |
|---|---|
| Which tools were offered? | Per-turn snapshot of the pinned set (version-stamped generation ID) |
| What was searchable? | Index generation ID + catalogue version at session start |
| What did the agent search for, and what came back? | Search query, result identifiers, expansion event, with the agent's `limit` value |
| Was the invoked tool approved, and unchanged? | Approval record + hash match result at execution time |
| Which user context enabled it? | Gateway profile assignment, logged per session |

Disclosure-record logging was made **immutable and centralised**, on the standards guidance to maintain immutable audit trails of invocations and context changes. Change control treats three event classes as behaviour changes requiring evaluation: a **description edit**, an **index generation change**, and a **pinning/core-set change**. Each carries the same review path as a code change to the agent.

### 12.6 Migration, and the evaluation that proves no regression

Migration followed the pattern in §9.5: shadow-mode index first (comparing what search *would* have returned against what was actually used), consolidation before deferral, then deferral in tiers from the long tail with the core pinned and rollback available at each tier.

The evaluation design is the part the team considers its real achievement, because it is what turns "we think it got better" into evidence:

- **Three layers, measured separately:** retrieval recall (was the right tool in the returned set?), selection (did the model pick it?), and task success (did the workflow complete?).
- **A canary suite of 120 tasks asserting task→tool outcomes**, not just success — because the failure mode being hunted is *silent*: a tool that stops being found. The suite runs against the **deferred** surface, so it exercises the index, not the visible tools.
- **A shadow comparison** against the pre-migration static surface, run for four weeks.
- **Cache-miss-rate alerting** as an incident class, since a disclosure change can raise cost without breaking function.

Illustrative outcome after migration (invented, for shape only): definition tokens before the request read fall from ~96,000 to ~7,400 plus 3–5 expanded tools; wrong-tool call rate falls from 9.1% to 3.2%; the specific near-duplicate confusion is eliminated because the write/read split means submission tools are never in the same retrieved family as status tools; cost attributable to tool definitions falls from 38% to 14% of input spend; median task latency **rises** by a small amount, which the team accepted and reported rather than hid.

### 12.7 Operating cost

Recurring costs added (illustrative): the index generation and versioning job; per-step disclosure logging and its storage; the canary evaluation suite and its maintenance; the definition-hash re-verification in the call path; and the review effort for definition changes. The team's judgement is that the logging and evaluation costs — not the token savings — are the ones that will grow with the estate, because a dynamic surface requires telemetry that a static one did not.

### 12.8 What remains unproven at Cymbal

Stated honestly, because it is the point of the exercise:

- **The accuracy improvement is measured on Cymbal's own suite**, not on an independent benchmark; it is not comparable to any published figure.
- **The latency cost is local.** No public measurement of the discovery round trip exists, so the platform can only report its own number as an engineering fact about its stack.
- **The hard-query failure mode is mitigated, not solved.** Adaptive search breadth helps, but there is no published evidence that any depth policy avoids the 0%-on-hard-queries outcome across a real catalogue; Cymbal's protection is its canary suite, not a technique.
- **Code mode is unevaluated** in this estate — scheduled, not tested.
- **The audit story is Cymbal's own design.** Nothing in the standards material prescribes how to evidence a runtime-disclosed surface; the platform built that, and it will be tested by an auditor rather than by a paper.
- **The 63.3%-cognitive-failure finding was never tested against Cymbal's failures.** If a similar share of Cymbal's task failures is cognitive rather than tool-related, then a large part of the remaining error mass is untouched by any of the work in this section.

---

## 13. The Claims Audit — Verified, Flagged, Rejected

| # | Claim | Verdict | Source, quality, date |
|---|---|---|---|
| 1 | Current MCP specification revision is **2026-07-28** | **Verified** | modelcontextprotocol.io/docs/learn/versioning: "The **current** protocol version is **2026-07-28**" — specification (primary), checked Sept 2026 |
| 2 | Version IDs encode *the last date backwards-incompatible changes were made*, not the release date | **Verified** | Same page, specification (primary) |
| 3 | The 2026-07-28 revision ships **July 28, 2026**; release candidate locked **May 21, 2026** | **Verified** (RC lock date is from the RC post) | MCP blog release-candidate post + spec page existence; specification/blog (primary), 2026 |
| 4 | 2026-07-28 removes protocol sessions; `tools/list` no longer varies per connection; `server/discover` added; `subscriptions/listen` replaces subscription endpoints; experimental tasks moved to an extension | **Verified** | 2026-07-28 changelog, specification (primary), checked Sept 2026 |
| 5 | 2026-07-28 requires `ttlMs` and `cacheScope` on list/read results, and says servers **SHOULD** return tools in deterministic order "to improve LLM prompt cache hit rates"; `tools/list` supports cursor pagination | **Verified** | 2026-07-28 changelog (minor changes §3, §5), specification (primary) |
| 6 | Structured tool output and resource links in tool results arrived in **2025-06-18**; tool annotations and Streamable HTTP in **2025-03-26**; icons, tool-name guidance, JSON-Schema-2020-12 default and experimental tasks in **2025-11-25** | **Verified** (changelogs read) | Revision changelogs, specification (primary) |
| 7 | `notifications/tools/list_changed` is normative in **2026-07-28** (delivered via `subscriptions/listen` with a `toolsListChanged` opt-in) | **Verified** | 2026-07-28 tools spec + changelog |
| 8 | The **origin revision** of `notifications/tools/list_changed`, and the revision that first introduced `tools/list` cursor pagination | **Flagged — unverified** | Not established from the initial-revision pages in this pass. Do not assert an origin date |
| 9 | A published post-2026-07-28 draft revision or ship date | **Flagged — not found** | No post-2026-07-28 draft identified; the RC post describes an extensions-first forward direction |
| 10 | **Anthropic "Code execution with MCP"** — 4 Nov 2025; MCP servers presented as a filesystem of tool files; a section literally titled "Progressive disclosure"; "150,000 → 2,000 tokens, **98.7%** saving"; requires a sandbox | **Verified as published; figures are vendor claims** | anthropic.com/engineering/code-execution-with-mcp, vendor primary, 4 Nov 2025. The 98.7% is an **illustration of one workflow**, not a benchmark |
| 11 | **Anthropic advanced tool use** — 24 Nov 2025; Tool Search Tool + Programmatic Tool Calling + Tool Use Examples; beta header `advanced-tool-use-2025-11-20`; tool-search identifiers `..._20251119` | **Verified as published** | anthropic.com/engineering/advanced-tool-use, vendor primary |
| 12 | Tool search mechanism: `defer_loading: true`, search tool in `tools`, at least one non-deferred tool, API expands `tool_reference` blocks, **deferred tools are still sent in the request** | **Verified — shipped and documented** | Claude Developer Platform tool-search docs, vendor primary, checked Sept 2026 |
| 13 | Documented tool-search limits: **10,000** deferred tools max; results **up to 5 by default** with `limit` 1–10,000; regex **200 chars**; BM25 **500 chars** | **Verified — documented** | Same docs |
| 14 | Deferred tools are excluded from the prompt prefix and arrive as inline conversation content, so tool search does not break prompt caching | **Verified — documented** | Tool-search docs + prompt-caching docs, vendor primary |
| 15 | Tool search "isn't metered as a separate server tool"; loaded definitions "count as input tokens"; cost **moves** rather than disappearing | **Verified — documented** | Tool-search docs |
| 16 | **85% token reduction**; ~77K → ~8.7K tokens; "preserves 191,300 vs 122,800" | **Flagged — vendor claim, illustrative, no methodology** | advanced-tool-use post, 24 Nov 2025 |
| 17 | Accuracy uplift: **Opus 4 49% → 74%; Opus 4.5 79.5% → 88.1%** | **Flagged — vendor claim, "internal testing"**, no eval name, size, or baseline | advanced-tool-use post, 24 Nov 2025. Directionally consistent with arXiv:2605.24660; not corroborating |
| 18 | Serviceable degradation threshold: "**30–50 available tools**" | **Flagged — vendor claim**, no methodology | Claude docs (tool search) and Claude Agent SDK docs, checked Sept 2026 |
| 19 | Five-server table: GitHub 35 tools ≈26K, Slack 11 ≈21K, Sentry 5 ≈3K, Grafana 5 ≈3K, Splunk 2 ≈2K = **58 tools ≈55K tokens**; Jira ≈17K; "134K tokens before optimization" | **Verified as a vendor table; flagged as a claim** — no tokenizer, method, or script published; the docs repeat it (consistency, not independence) | advanced-tool-use post, 24 Nov 2025; Claude docs |
| 20 | Claude Agent SDK tool search is **on by default**, with `ENABLE_TOOL_SEARCH` = unset / `true` / `auto` / `auto:N` / `false`; `auto` activates when deferrable definitions reach **10%** of the window; `alwaysLoad` exempts a server; max **10,000** tools; not supported on Microsoft Foundry on Azure; disabled behind non-first-party base URLs | **Verified — documented** | code.claude.com Agent SDK tool-search docs, vendor primary, checked Sept 2026 |
| 21 | **Agent Skills** — three loading tiers (metadata ~**100 tokens/skill** always; body **<5K** on trigger; resources **zero until accessed**); `name` ≤64 chars, `description` ≤1,024 chars; body <500 lines recommended; "100+ available Skills" | **Verified — documented** (tier budgets are published guidance, not benchmarks) | Agent Skills docs; launch post 16 Oct 2025; open standard 18 Dec 2025 (agentskills.io) |
| 22 | Skills can bundle executable scripts whose code "never loads into the context window" — only output does | **Verified — documented** | Agent Skills docs |
| 23 | **Gemini**: published guidance "**Tool Selection: Keep active set to 10-20 tools maximum**" | **Verified — documented** (page footer read as "Last updated 2026-09-16 UTC") | ai.google.dev Gemini function-calling docs |
| 24 | **OpenAI**: no deferred-loading field, no hosted tool-search primitive, no documented maximum tool count | **Negative finding** (two pages read: function calling; MCP/connectors). Absence on those pages is not proof of absence platform-wide | platform.openai.com docs, checked Sept 2026 |
| 25 | **Cloudflare "Code Mode"** — tools converted to a TypeScript API via the Agents SDK `codemode` helper; authors Kenton Varda and Sunil Pai | **Verified as published; no numbers published in the post (vendor claim without figures)**; **publication date unverified** | blog.cloudflare.com/code-mode; cited by Anthropic 4 Nov 2025 |
| 26 | **Docker MCP Gateway** is a centralised proxy using a **profile** to decide which servers clients see; "as part of Docker AI Governance is an **invite-only** feature" | **Verified — documented** | docs.docker.com/ai/mcp-gateway, checked Sept 2026 |
| 27 | **GitHub MCP Server** supports `--toolsets` / `--tools` / `--read-only` scoping whose stated purpose is to "help the LLM with tool choice and **reduce the context size**" | **Verified — first-party README** | github.com/github/github-mcp-server, checked Sept 2026 |
| 28 | GitHub MCP Server's own tool count and token cost | **Negative finding** — the README publishes **no** tool count and **no** token figure. Anthropic's "35 tools / ~26K tokens" remains the only attributable number |
| 29 | **Prompt caching**: tools sit at the head of the prefix hierarchy; modifying tool definitions **invalidates the entire cache**; removing tools mid-conversation is "one of the most common ways people break prompt caching"; the pattern is **defer, don't remove** | **Verified — documented** | Claude tool-with-prompt-caching docs; Claude Code prompt-caching blog, 30 Apr 2026 (author Thariq Shihipar) |
| 30 | Claude Code treats cache breaks as **incidents**; connecting/disconnecting an MCP server is a documented cache-invalidating action; a gateway that strips cache markers while returning success causes the whole history to bill as uncached | **Verified — documented** | Claude Code prompt-caching docs + blog, Apr 2026 |
| 31 | Cache pricing multipliers ("write 25% more / read 90% less") | **Flagged — untraceable** in this pass; not verified against the primary pricing page |
| 32 | **BoR / "How Many Tools Should an LLM Agent See?"** — BFCL 370 tools: adaptive matches always-50 coverage (**90.3% vs 90.8%**) at **7 tools** average; ToolBench 3,251 tools: fixed-5 gets 0% on hard queries vs **16.7%** deeper; downstream Claude Sonnet 4.6 **93.1% vs 87.1%**, **76.8% vs 60.9%** | **Verified — independent, peer-unreviewed preprint** | arXiv:2605.24660, 23 May 2026 (v2 7 Jun 2026); abstract re-read at source. Authors describe the agent as a metric probe, not a system |
| 33 | **RAG-MCP**: prompt tokens cut >50%; selection accuracy **43.13% vs 13.62%** | **Verified as published — independent, with traps**: simulated catalogue, selection-only metric, deliberately bloated baseline | arXiv:2505.03275, 6 May 2025 |
| 34 | **MetaTool**: majority of eight LLMs struggle to select tools; recommends rewriting descriptions per downstream model | **Verified as published — independent** | arXiv:2310.03128, 4 Oct 2023 (v6 Dec 2024) |
| 35 | **BiasBusters** (ICLR 2026): models favour earlier tools; small description perturbations shift choices; uniform sampling after filtering reduces bias | **Verified — peer-reviewed** | arXiv:2510.00307, camera-ready ICLR 2026 |
| 36 | **MCP-Atlas**: 1,000 tasks / 36 real MCP servers / 220 tools / 20 models; pass rates up to **82.2%**; **63.3% of failures cognitive, not tool-call** | **Verified — independent** | arXiv:2602.00933, 31 Jan 2026 (v3 19 May 2026); Scale leaderboard |
| 37 | **MCP-Bench**: 28 servers; single/two/three-server configurations; NeurIPS **2025 Workshop** (not main conference) | **Verified as published** — leaderboard is live; scores quoted anywhere are snapshots and the model list is dated | arXiv:2508.20453; Accenture repo |
| 38 | **ToolScope** merging improves selection accuracy **8.38%–38.6%** | **Flagged — abstract-level verification only**; full text not re-derived in this pass | ACL 2026 Long Papers |
| 39 | The central methodological gap: no study combines real MCP servers + varied catalogue size + end-to-end task success | **Negative finding** (established by search) | Research pass, Sept 2026 |
| 40 | "**GitHub MCP = 55,000 tokens**" (widely repeated) | **Rejected as untraceable** — conflicts with Anthropic's own 35 tools ≈26K; no source reconciles it. Attribute to Anthropic's Nov 2025 estimate or drop |
| 41 | "Amplitude MCP ≈140K tokens"; "224,655 characters / 91 tools ≈35K tokens"; "PostgreSQL ≈35 tokens" | **Rejected — untraceable**; no method or raw data shown |
| 42 | "Accuracy degrades to 85–91% at 5 tools → 65–78% at 20+"; "parameter mismatch accounts for 60–75% of failures" | **Rejected as marketing** — commercial page, no dataset, model list, or method. Do not cite |
| 43 | **OWASP MCP Top 10** (MCP01–MCP10:2025) names privilege escalation via scope creep, tool poisoning, lack of audit and telemetry, and shadow MCP servers | **Verified — standards body, but a *beta* community project** ("Beta Release and Pilot Testing — we are here right now"); do not present as settled consensus | owasp.org MCP Top 10 project, checked Sept 2026 |
| 44 | Tool poisoning is a **connect-time vs runtime trust gap**: descriptions reviewed once at connect; responses unchecked; "the LLM sees all tool descriptions from all connected servers" | **Verified — OWASP attack page and cheat sheet**; the last assumption is precisely what disclosure breaks | OWASP MCP Tool Poisoning page + MCP Security Cheat Sheet |
| 45 | Compensating controls: SHA-256 pinning over **name + description + input schema** with re-verification before execution; re-prompt on change; central invocation logging; fail closed on integrity failure | **Verified — OWASP cheat sheet** | Same |
| 46 | MCP specification security best practices cover confused deputy, token passthrough, and scope minimization, and contain **no guidance on dynamic/deferred disclosure or its audit** | **Verified + negative finding** | modelcontextprotocol.io specification 2026-07-28 security best practices, checked Sept 2026 |
| 47 | **EU AI Act Article 12** requires automatic event logging for high-risk systems; in force **2 Dec 2027** (Annex III) / **2 Aug 2028** (Annex I) | **Verified via consolidated-text mirror** — quote the official text in any filing. **No article or recital mentions tool permissions/catalogues/MCP** (negative finding); the tool-surface reading is an **inference**, labelled as such | artificialintelligenceact.eu Art. 12 + Art. 113(c), checked Sept 2026 |
| 48 | **ISO/IEC 42001:2023** published **2023-12-18**, an AI management-system standard | **Verified as existing and dated; control-level mapping NOT verified** — the standard is paywalled and Annex A was not read. Do not assert clause numbers | iso.org/standard/42001 |
| 49 | **NIST AI RMF 1.0** (Jan 2023) and **Generative AI Profile AI 600-1** are voluntary and **contain no agentic tool-permission control**; an "agentic profile" surfaced is a **Cloud Security Alliance** document, not NIST | **Verified + negative finding** | nist.gov AI RMF pages; CSA document treated as third-party |
| 50 | **FSB consultation report, 10 June 2026**: 12 sound practices; least privilege for agents and sub-agents; **dynamic IAM** that grants/changes/revokes permissions in real time; audit trails of every agent transaction; explainability of intermediate steps; "human-in-command" oversight; comments closed 22 July, final to G20 in October; explicitly **not intended as an international standard** | **Flagged — via trade-press reporting only.** The FSB primary page 404'd and follow-up search returned empty. Cite the FSB document directly in any filing | theasianbanker.com, 11 June 2026 (secondary) |
| 51 | **IOSCO** Supervisory Toolkit for AI in Capital Markets finalised **25 May 2026**; "AI as a judge" experimentation | **Flagged — via the same trade-press source** | Same |
| 52 | If ~63% of agent failures are cognitive, disclosure optimises a minority of the failure mass | **Verified as a measured finding in one benchmark; flagged as a generalisation** — it is a single benchmark's diagnostic taxonomy, not a universal constant | arXiv:2602.00933 |

---

## 14. What Could Not Be Verified

Recorded as explicit negatives, because a silent omission reads as coverage. **None of these should be asserted elsewhere.**

1. **The origin revision of `notifications/tools/list_changed`** and of `tools/list` cursor pagination — both confirmed present in 2026-07-28, but the revision that introduced each was not established.
2. **A published post-2026-07-28 draft or next-revision identifier / ship date.** Consult the specification's draft area and the MCP blog directly if this matters.
3. **The Cloudflare "Code Mode" post's own publication date.** The page exposed no date; it predates 4 November 2025 only by inference from Anthropic's citation.
4. **Any independent, reproducible third-party measurement campaign of MCP tool-definition token cost** with a published script, tokenizer, and raw server inventory. Every headline token figure traces to one vendor's November 2025 post.
5. **A published census of tool counts across popular MCP servers.** Per-server counts exist only in Anthropic's five-server example; GitHub's own README publishes none.
6. **Any measurement of the search/discovery round trip's latency in milliseconds**, and **any published net-cost crossover point** (turns per session) at which deferred disclosure pays back.
7. **A study isolating "lost in the middle" / needle-in-a-haystack degradation specifically for tool definitions.** Positional-recall findings on retrieved prose must not be extrapolated to schemas.
8. **Any study combining real MCP servers + systematically varied catalogue size + end-to-end task success.** The central gap; every "X tools breaks agents" claim rests on simulated catalogues.
9. **A public, reproducible, third-party end-to-end task-success benchmark of progressive disclosure versus a full upfront surface.**
10. **Spec-level guidance on auditing, logging, or reviewing a runtime-disclosed tool surface** — the MCP security best-practices document does not address dynamic disclosure at all.
11. **Any template for evidencing a dynamically-disclosed tool surface** for audit or regulatory purposes. This is the enterprise deliverable that does not yet exist off the shelf.
12. **Control-level (clause) mapping from ISO/IEC 42001 Annex A** to tool permissions or agent auditability — the standard is paywalled.
13. **Any NIST control requiring tool-surface inventory, tool permission scoping, or agent auditability.** The "agentic profile" encountered is a Cloud Security Alliance document, not a NIST publication.
14. **Any SR 11-7 / OCC / PRA / ECB statement specifically addressing agentic AI tool permissions.** The June 2026 FSB consultation materially reduces but does not close this gap, and it is non-binding and self-described as not setting an international standard.
15. **The FSB consultation report's primary text.** The fsb.org URL returned 404 and a follow-up search returned an empty result set; all FSB facts in this guide are via trade-press reporting and must be re-sourced before use in a filing. Law-firm and trade summaries of it were surfaced but not read.
16. **Cache pricing multipliers** (cache-write premium / cached-read discount percentages). Not verified against the primary pricing page.
17. **The actual per-server token costs for Slack, Jira/Atlassian, Chrome DevTools, AWS, and any other widely-used server, attributable to that server's own publisher.** All published figures trace to Anthropic's estimates.
18. **Framework support in several cases** — LlamaIndex, CrewAI, AutoGen/Microsoft Agent Framework, the open-source OpenAI Agents SDK, the Vercel AI SDK, and Kong/Envoy-style MCP proxies were **not checked**; LangChain, Pydantic AI, and Semantic Kernel were checked and show **no documented tool-retrieval or deferred-loading feature** on the pages read (for Semantic Kernel, the plugins page footer read "12/10/2024", which is notably stale for this topic).
19. **Remote-MCP section contents in the Gemini documentation** (the section exists; contents not extracted) — and a **research-tooling note that affects reproducibility:** search backends returned empty result sets intermittently during this research (several queries across both passes), and the fallback that worked was direct extraction of named primary URLs. Any future refresh of this guide should budget for that failure mode rather than assume search coverage.

---

## 15. Glossary

**Agent Skills / SKILL.md** — A layered instruction format: a directory containing a markdown file with `name` and `description` frontmatter, an instruction body, and optional bundled references and scripts. Announced 16 October 2025; published as an open standard on 18 December 2025.

**Annotation (tool annotation)** — Metadata on a tool definition conveying behavioural hints (read-only, destructive, idempotent, open-world), introduced in the 2025-03-26 MCP revision.

**Cache scope** — The `cacheScope` field (`"public"` or `"private"`) required on MCP list/read results from the 2026-07-28 revision, controlling whether shared intermediaries may cache a response.

**Cache TTL (`ttlMs`)** — The freshness hint, in milliseconds, required on MCP list/read results from 2026-07-28; a client may cache a tool list for that long rather than re-fetching.

**Catalogue** — The full set of tool definitions an agent platform can reach across all connected servers. Distinguish from the *offered* surface (what enters the prompt) and the *invocable* surface (what authorisation permits).

**Code mode / code execution over tools** — Presenting MCP servers as a code API (a filesystem of typed tool files, or a generated client) so the agent writes code that calls them and tool schemas do not enter the prompt as text. Anthropic's formulation is "Code execution with MCP" (4 November 2025); Cloudflare's is "Code Mode."

**Context bloat** — The condition where definitions, results, and history crowd the window, leaving little residual space for the task.

**Deferred loading (`defer_loading`)** — The documented mechanism that excludes a tool's definition from the prompt prefix while still transmitting it in the request, so it can be discovered and expanded on demand.

**Deterministic ordering** — Returning tools in a stable order, which the 2026-07-28 MCP specification instructs servers **SHOULD** do to improve LLM prompt cache hit rates. A deterministic surface is a cache-survivable surface.

**Discovery (server)** — Finding, registering, publishing, and adopting an MCP **server**. Owned by `mcp_discovery_guide.md`. Distinct from disclosure.

**Disclosure (tool/instruction)** — Deciding which tool definitions and which instruction bodies enter the model's context window, and when. The subject of this guide.

**Function calling / tool calling** — The model-provider interface by which a model emits a structured request to invoke a named function with arguments. Tool definitions are supplied to the model through this interface.

**Gateway aggregation** — A proxy in front of many MCP servers presenting a collapsed, curated (often profile-based) surface to clients, while centralising credentials and access control.

**Hierarchical namespacing** — Encoding the owning server or domain into a tool name (`github_create_issue`, `Slack:post_message`) so searches, groupings, and references resolve unambiguously.

**Index (tool index)** — The searchable representation of a catalogue that a tool-search or retrieval step queries. Its quality determines whether disclosure works; it is itself a governed artifact.

**Instruction-level disclosure** — Withholding an instruction body (procedure, runbook, template) until a trigger matches, as opposed to withholding a tool schema. The skills pattern.

**MCP (Model Context Protocol)** — The open standard for connecting AI clients to external tools, resources, and prompts. Revision identifiers are date-stamped (`YYYY-MM-DD`) and encode the last date backwards-incompatible changes were made.

**Pinned tool / always-loaded tool** — A tool whose definition remains in the prompt prefix because it is expected to be relevant in nearly every session.

**Progressive disclosure** — In this guide's sense, disclosing the tool and instruction surface on demand rather than up front.

**Prompt cache / prefix caching** — Provider-side reuse of the unchanged prefix of a request. For Claude, the cache hierarchy is `tools` → `system` → `messages`; changing tool definitions invalidates the entire cache.

**Result-level disclosure** — Filtering or transforming a tool *result* before it enters context (e.g. reducing 10,000 rows to five) rather than withholding a definition.

**Retrieval (tool retrieval)** — An application-level shortlist step over a tool catalogue (embeddings or keyword search), of which provider-hosted tool search is a managed instance.

**Rug pull (tool)** — Post-approval mutation of a tool's description or schema so that a reviewed, trusted tool becomes malicious or misleading. The documented compensating control is hash-pinning the full schema and re-verifying before execution.

**Skill trigger** — The event by which a skill's `description` matches the current task and its body is loaded. Trigger failure is silent: the work proceeds, just without the procedure.

**Tool definition / tool schema** — The name, description, and JSON Schema describing one callable tool, as returned by `tools/list`.

**Tool poisoning** — Injection of malicious, misleading, or biased content through tool descriptions or tool outputs so as to manipulate model behaviour.

**Tool reference (`tool_reference`)** — The block type by which a matched tool is delivered into the conversation when discovered via tool search; the API expands it into a full definition inline.

**Tool search** — A model-callable tool that searches a catalogue and returns matching definitions for expansion into context.

**Tool set / toolset** — A named grouping of tools (e.g. an MCP server's whole set, or a framework's registered group) that can be enabled, deferred, or excluded as a unit.

---

## 16. Cross-References and Further Reading

### 16.1 Primary sources — the specification

- **MCP versioning (current revision, version semantics, feature states):** https://modelcontextprotocol.io/docs/learn/versioning
- **Current revision changelog (2026-07-28):** https://modelcontextprotocol.io/specification/2026-07-28/changelog
- **Tools (listing, pagination, caching, list-changed, annotations):** https://modelcontextprotocol.io/specification/2026-07-28/server/tools
- **Security best practices (confused deputy, token passthrough, scope minimization):** https://modelcontextprotocol.io/specification/2026-07-28/basic/security_best_practices
- **Feature lifecycle and deprecation policy:** https://modelcontextprotocol.io/community/feature-lifecycle
- **Release-candidate announcement and roadmap:** https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/ · https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/
- **Previous revisions:** https://modelcontextprotocol.io/specification/2025-11-25/changelog · https://modelcontextprotocol.io/specification/2025-06-18/changelog · https://modelcontextprotocol.io/specification/2025-03-26/changelog

### 16.2 Vendor engineering sources

- **Code execution with MCP (4 Nov 2025):** https://www.anthropic.com/engineering/code-execution-with-mcp
- **Advanced tool use — Tool Search Tool, Programmatic Tool Calling (24 Nov 2025):** https://www.anthropic.com/engineering/advanced-tool-use
- **Agent Skills (16 Oct 2025; open standard 18 Dec 2025):** https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills · https://agentskills.io/
- **Cloudflare "Code Mode: the better way to use MCP":** https://blog.cloudflare.com/code-mode/ · **Claude Code on prompt caching (30 Apr 2026):** https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything

### 16.3 Product documentation

- **Claude Developer Platform — tool search, prompt caching with tools, MCP connector (`mcp_toolset`, `defer_loading`):** https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool · https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-use-with-prompt-caching · https://platform.claude.com/docs/en/agents-and-tools/mcp-connector
- **Agent Skills overview (loading tiers and token budgets) and best practices:** https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview · /best-practices
- **Claude Agent SDK tool search (default-on; `ENABLE_TOOL_SEARCH`) and Claude Code prompt caching:** https://code.claude.com/docs/en/agent-sdk/tool-search · https://code.claude.com/docs/en/prompt-caching
- **Gemini function calling (10–20 tool active set) and OpenAI function calling / MCP and connectors:** https://ai.google.dev/gemini-api/docs/function-calling · https://platform.openai.com/docs/guides/function-calling · https://platform.openai.com/docs/guides/tools-connectors-mcp
- **Frameworks and gateways — LangChain MCP integration, Pydantic AI MCP client, Semantic Kernel plugins, Docker MCP Gateway, GitHub MCP Server (toolsets/scoping):** https://docs.langchain.com/oss/python/langchain/mcp · https://ai.pydantic.dev/mcp/client/ · https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/ · https://docs.docker.com/ai/mcp-gateway/ · https://github.com/github/github-mcp-server

### 16.4 Measurement literature

- **Core disclosure measurement — "How Many Tools Should an LLM Agent See? A Chance-Corrected Answer"** (arXiv:2605.24660, 23 May 2026) and **RAG-MCP** (arXiv:2505.03275, 6 May 2025)
- **Real-MCP-server competency benchmarks — MCP-Atlas** (arXiv:2602.00933, 31 Jan 2026; v3 19 May 2026; leaderboard https://labs.scale.com/leaderboard/mcp_atlas) and **MCP-Bench** (arXiv:2508.20453, NeurIPS 2025 Workshop; https://github.com/Accenture/mcp-bench)
- **Selection bias and catalogue-shape work — BiasBusters** (arXiv:2510.00307, ICLR 2026), **MetaTool** (arXiv:2310.03128, 4 Oct 2023), **ToolScope** (https://aclanthology.org/2026.acl-long.1573/, ACL 2026)

### 16.5 Standards, security, and supervisory material

- **OWASP MCP Top 10 (beta):** https://owasp.org/www-project-mcp-top-10/ · **Tool Poisoning:** https://owasp.org/www-community/attacks/MCP_Tool_Poisoning · **MCP Security Cheat Sheet:** https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
- **EU AI Act Article 12 (record-keeping):** https://artificialintelligenceact.eu/article/12/ — quote the official text in filings
- **ISO/IEC 42001:2023** (AI management system, published 2023-12-18, paywalled): https://www.iso.org/standard/42001 · **NIST AI RMF:** https://www.nist.gov/itl/ai-risk-management-framework
- **FSB, *The Financial Stability Implications of Artificial Intelligence* (14 Nov 2024):** https://www.fsb.org/2024/11/the-financial-stability-implications-of-artificial-intelligence/ — and the June 2026 responsible-adoption consultation, whose primary page was unreachable during this research

### 16.6 Repository companions (do not duplicate)

- [context_engineering_guide.md](context_engineering_guide.md) — context economics, budgets, compression, positional effects · [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) — the agent loop and KV-cache-aware context design
- [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) — MCP fundamentals, primitives, server implementation · [mcp_discovery_guide.md](mcp_discovery_guide.md) — server registries, directories, and adoption
- [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) — sandbox design for code execution · [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) · [ai_agent_drift_guide.md](ai_agent_drift_guide.md) — agent failure and drift catalogues
- [ai_governance_framework_guide.md](ai_governance_framework_guide.md) · [implementing-responsible-ai.md](implementing-responsible-ai.md) · [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md) — governance and assurance
- [llm_guard_models_guide.md](llm_guard_models_guide.md) · [prompt_injection_guide.md](prompt_injection_guide.md) — guardrails and injection · [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) — where the tool layer and gateway sit in a platform

---

## 17. Closing Summary

Progressive disclosure in the MCP ecosystem is best understood as a **budget decision about an interface**, not a feature. MCP made tools pluggable, and the cost of that pluggability is that every reachable definition is a candidate for the prompt prefix on every turn. The current specification revision, **2026-07-28**, shows the ecosystem taking the consequence seriously at the protocol level: tool lists no longer vary per connection, list results carry cache TTLs and a cache scope, servers are told to return tools in a deterministic order explicitly to improve **prompt cache hit rates**, and change notifications for tool lists flow through a proper subscription stream.

Five conclusions survived the evidence, and two cautions:

1. **The problem is real and vendor-documented, but the headline numbers are one vendor's estimates.** The most-cited per-server token table traces to a single November 2025 post with no published methodology. Measure your own estate; cite the table only as an order-of-magnitude anchor.
2. **The strongest independent evidence supports adaptive disclosure and refutes fixed depth.** Matching a much wider surface while showing a handful of tools is achievable; a hard-coded shortlist of five scored **zero** on hard queries in a 3,251-tool catalogue, while an adaptive policy found 16.7%. "Show five tools" is a measured failure mode, not a design.
3. **Caching makes the naive implementation wrong.** Removing tools mid-session invalidates the whole prefix cache. The correct pattern is **defer, don't remove**, and the mechanism is documented with that rationale.
4. **Ordering and wording are the mechanism, not the cosmetics.** A peer-reviewed benchmark shows models favour earlier tools and that small description perturbations shift choices — which is why namespacing, deterministic generation, description review, and hash pinning belong in the design rather than in a later hardening phase.
5. **Tool-surface optimisation addresses a minority of the failure mass.** In the one benchmark built on real MCP servers with plausible distractors, **63.3% of diagnosed failures were cognitive, not tool-call related**. Disclosure fixes a real cost and a real accuracy problem; it does not fix agent failure.

**Caution one:** the measurement base is thinner than the discourse suggests. No study combines real servers, varied catalogue size, and end-to-end task success; no independent measurement campaign of tool-definition token cost exists; no latency measurement of the discovery round trip has been published; and the space is polluted with unsourced percentage ranges that should be rejected outright. **Caution two:** the governance base is thinner still. The reviewable artifact in a regulated estate is the *catalogue*, not the loaded set; enforcement must be server-side and gateway-side, never prompt-side; the pinning control is a hash over the full schema, re-verified before execution; and the per-step record of *what was offered, what was searchable, what was searched, and what was loaded* is something you will have to build, because no source retrieved for this guide supplies a template for evidencing a runtime-disclosed surface.

The practical shape of the answer, in order of adoption: **fix the inventory before hiding it; namespace and consolidate; pin a small always-loaded core; defer the long tail behind an index you govern; treat descriptions and index generation as behaviour-affecting change; evaluate retrieval, selection, and task success separately against the deferred surface; and accept an honest latency cost rather than pretending the discovery round trip is free.** The tool that is never named, never described, and never loaded cannot be misused, audited, or improved — and the tool that is always loaded is always paid for.

---

> **About this guide:** Written September 2026 against specification revision **2026-07-28**. MCP and the vendor tooling around it are moving fast: specification revisions are date-stamped and the disclosure primitives are still shifting between beta and general availability. **Re-verify every parameter, threshold, and availability status against the primary pages cited in §16 before building a policy on it.** The measurement tables in §7 and the audit in §13 carry the source and date for every figure.
>
> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
>
> *Progressive disclosure is not the removal of the tool. It is the governance of the disclosed tool.*
