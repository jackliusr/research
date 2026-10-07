# Agent Experience (AX) — Designing for a Consumer That Never Files a Bug Report

> **Byline:** Jack Liu Shurui, Solution Architect  
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Agent Interfaces  
> **Date:** October 2026 · **Sources read:** 2026-10-07

> **Standfirst.** *Agent experience* (AX) is the practice of designing systems, interfaces, documentation and tools for a consumer that is an AI agent rather than a human end user (UX) or a human developer (DX). This guide treats AX as a **framing**, not a received discipline: the phrase is in live commercial and practitioner use as of 2025–2026, but it has no standards body, no peer-reviewed canon under that name, and at least two mutually incompatible definitions in the wild. What *is* well evidenced is a body of concrete design guidance — from Anthropic, OpenAI, Google and the Model Context Protocol specification — about how to make a tool, an API, a description or an error message usable by a model-driven caller. This guide assembles that guidance, names its source and date for every rule, and organises it around the single property that makes agent-facing design different: an agent that cannot use what you built does not tell you. It produces a plausible wrong answer instead.

> **How to read it.** Section 1 establishes the term honestly and declares the boundary against the sibling guides in this repository. Section 2 draws the line between AX, UX and DX. Section 3 states what an agent actually needs as seven design consequences. Section 4 walks the seven surfaces an agent meets. Section 5 states the design principles as rule-plus-reason, each tagged with its source. Section 6 is the disclosure framing, cross-referenced, not re-derived. Section 7 gives the measurement method — the agent audit — and its honesty caveat. Sections 8–10 are the failure modes, the security exposure, and the liability questions left open. Section 11 inverts the lens onto the agents an institution builds itself. Sections 12–13 are the scorecard and the Cymbal Bank worked example. Sections 14–16 are the anti-patterns, the claims audit, and the verification apparatus. Every design rule is attributed to its publisher and date, or labelled as this guide's own reasoning. Nothing here is presented as a benchmark figure.

> **Verification policy.** Facts marked **(verified)** were checked against a primary source — a specification, a vendor's own engineering post or product documentation, or a published paper — during writing (October 2026). Everything else carries a label: **(vendor claim)** for a statement a vendor makes about its own product or practice; **(flagged)** where a claim is repeated but not traceable to a measurement; **(unverified/flag)** where a page could not be extracted; **(negative finding)** where a searching pass established that something does *not* appear in the sources checked. No design rule in this guide is written from memory: each is either quoted from a dated primary source or explicitly marked as this guide's reasoning.

---

**Table of contents**

1. [Overview, the Term Established or Rejected, the Decoder, and the Boundary Declared](#1-overview-the-term-established-or-rejected-the-decoder-and-the-boundary-declared)
2. [Why This Differs from UX and from DX](#2-why-this-differs-from-ux-and-from-dx)
3. [What an Agent Actually Needs](#3-what-an-agent-actually-needs)
4. [The Surfaces](#4-the-surfaces)
5. [The Design Principles](#5-the-design-principles)
6. [Progressive Disclosure — the AX Framing Only](#6-progressive-disclosure--the-ax-framing-only)
7. [Measurement — How You Know an Agent Can Use It](#7-measurement--how-you-know-an-agent-can-use-it)
8. [The Failure Modes](#8-the-failure-modes)
9. [Security — the Material You Publish Is the Injection Vector](#9-security--the-material-you-publish-is-the-injection-vector)
10. [Liability and Authority — Open Questions, Not Answers](#10-liability-and-authority--open-questions-not-answers)
11. [The Inverted Discipline — the AX of Your Own Agents](#11-the-inverted-discipline--the-ax-of-your-own-agents)
12. [An AX Assessment — a Practical Scorecard](#12-an-ax-assessment--a-practical-scorecard)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit — Verified, Flagged, Rejected](#15-the-claims-audit--verified-flagged-rejected)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing Summary](#16-what-could-not-be-verified-glossary-cross-references-and-closing-summary)

---

## 1. Overview, the Term Established or Rejected, the Decoder, and the Boundary Declared

### 1.1 What this guide is

This guide is about a design surface that human factors did not name for us: the interface presented not to a person and not to a programmer who will read the manual first, but to an autonomous model-driven process that will act on whatever is *in front of it* at the moment of a call and nothing else. Call that consumer the **agent**. The question this guide asks is narrow and practical: if your API, your tool, your error message, your documentation or your authentication path is going to be used by such a consumer, what does it need to be, how do you tell whether it worked, and what happens when it does not.

It is deliberately *not* a guide to building an agent. The internals — the loop, the harness, the context budget, the skills mechanism — are owned elsewhere in this library, and §1.4 names each. This guide sits on the *other side of the call*: it is written for the people who publish the thing the agent calls.

### 1.2 The term: what "agent experience" actually means in published use

The honest starting point is that **"agent experience" is not an established discipline with a canon.** It is a marketing and practitioner term. What can be verified about it is the following.

The phrase **AX / "agent experience"** was, on the evidence available, **coined by Mathias Biilmann, CEO of Netlify, in early 2025** and popularised through a blog post titled "Introducing AX" and a Netlify landing page. Netlify's own page states, in words: *"Netlify coined the term in 2025."* **(verified as published; vendor self-attribution)** — netlify.com/agent-experience, checked October 2026. Resend's Zeno Rocha repeated the origin in a post dated **February 19, 2025**: *"A couple of weeks ago, Mathias Biilmann coined the term AX (Agent Experience)."* **(verified as published; vendor blog)** — resend.com/blog/agent-experience, 19 February 2025. Speakeasy, in a practitioner guide, writes that AX is *"a concept introduced in early 2025 by Mathias Biilmann, CEO of Netlify."* **(verified as published; vendor blog, page footer "Last updated on July 23, 2025")** — speakeasy.com/blog/agent-experience-introduction.

The **definition most widely repeated** is Netlify's own: *"Agent Experience (AX) is the holistic experience AI agents will have as the user of a product or platform."* **(vendor claim)** — netlify.com/agent-experience. That definition places the *agent* as the user. But the term is **not used consistently**. A second cluster of writing uses "AX" to mean the experience of *humans working with agents* — onboarding, trust, oversight, recovery when the AI decides on a person's behalf. That framing is a product-design framing and it is the opposite polarity: the user is human, the agent is a feature of the human's experience. **(flagged — competing definitions, no reconciliation found)** — e.g. eleken.co/blog-posts/ax-design (2026) and the community site agentexperience.ax present this human-centred reading, while Netlify and its imitators present the agent-as-user reading.

Two further facts sharpen the picture:

- **A more rigorous academic antecedent exists under a different name.** The peer-reviewed term is **agent-computer interface (ACI)**, introduced with **SWE-agent** (Yang, Jimenez, Wettig, Lieret, Yao, Narasimhan, Press), *NeurIPS 2024*, arXiv:2405.15793 (first posted **6 May 2024**; v3 **11 November 2024**). The paper's framing sentence is the cleanest statement of the discipline's premise: *"LM agents represent a new category of end users with their own needs and abilities, and would benefit from specially-built interfaces to the software they use."* **(verified — peer-reviewed; identifier resolved at the arXiv API over HTTPS, 2026-10-07)**. It also reports that interface design measurably changes agent performance. "ACI" is the term with a citation trail; "AX" is the term with a marketing budget.
- **The model vendors do not use "AX."** The platform vendors frame the same problem as *"writing tools for agents"* or designing for a non-deterministic consumer, and their guidance carries dates in 2025–2026. Anthropic's engineering post is titled *"Writing effective tools for agents — with agents"* (**11 September 2025**) and states the premise explicitly: *"instead of writing tools and MCP servers the way we'd write functions and APIs for other developers or systems, we need to design them for agents."* **(verified as published; vendor engineering post)** — anthropic.com/engineering/writing-tools-for-agents.

**Verdict for this guide.** The term "agent experience" is **live but not settled**. This guide therefore treats AX as a **framing that is useful because it forces a design question**, not as received terminology. Where a reader wants a term with an academic pedigree, the honest one is **agent-computer interface (ACI)** (SWE-agent, 2024). Where a reader wants the commercial usage, AX (Netlify, 2025) is the origin. This guide uses "AX" throughout, says so plainly, and does not manufacture a canon around it. There is **no ISO standard, no W3C note, and no peer-reviewed "AX" literature** in the sources checked **(negative finding)**.

The originator has also shipped tooling, which is itself evidence about the term's status. Netlify publishes **AXIS**, an open-source scoring framework that runs real agents against real scenarios and scores the result across four dimensions — goal achievement, service quality, environment, and agent behaviour — and frames a score explicitly as *"a baseline and a direction"* rather than a pass mark. **(verified as published; vendor tooling and product documentation — netlify.com/agent-experience and axis.run, checked October 2026)**. This guide cites AXIS only as an *instance* of the audit method of §7, not as an endorsement and not as a benchmark: it is one vendor's instrumentation of its own thesis, and its scores are neither reproduced here nor independently validated.

### 1.3 The decoder

Nine terms recur and are defined once, here, in the sense this guide uses them. Each is a design object, not a metaphor.

| Term | Definition in this guide | Why it matters |
|---|---|---|
| **Agent consumer** | A model-driven process that calls your surface to complete a task on someone's behalf. It holds no persistent memory of your documentation, cannot ask you a clarifying question, and may be one of several candidates for the caller. | It is the "user" the whole guide is written for. |
| **Surface** | Any published artefact the agent can perceive and act on: an API, a protocol tool, a command-line entry point, a documentation page, an error response, an authentication path, a structured-output schema. | Design quality is per-surface, not per-product. |
| **Tool** | A callable unit exposed to an agent, named and described in a schema, that performs a **unit of work**. Distinct from an API endpoint, which exposes a *resource operation*. | The mismatch between "tool" and "endpoint" is the most common design error (§5, §14). |
| **Description** | The natural-language text attached to a tool or parameter that the agent reads as instruction. It is **prompt text**, not documentation metadata. | It steers behaviour directly and is a security surface (§9). |
| **Affordance** | What the surface makes it *look like* it is possible to do — inferred by the agent from names, schemas and descriptions, not from visual cues. | Agents perceive affordances, not layouts; a human affordance (hover, tooltip) conveys nothing. |
| **Unit of work** | The smallest task-meaningful step an agent should take in one call (e.g. "schedule an event", not "list users then list events then create event"). | Defines what a tool should be; drives authorisation and liability (§10). |
| **Silent failure** | A failure that produces no signal to the author: no bug report, no support ticket, no error the agent surfaces. Instead the agent produces a plausible wrong answer, invents a parameter, retries, or routes around the feature. | It is the failure that defines the discipline (HAZARD 3; §8). |
| **Agent audit** | A dated experiment in which an agent is given a task on the surface with no human help, and its completion, retries and failure modes are observed and recorded. | The only honest way to know whether a surface is agent-usable (§7). |
| **Discovery path** | The route by which an agent learns a surface exists and what it does, without a human browsing or reading docs on its behalf. | If discovery requires a person, the surface is not agent-reachable (§3, §4). |

### 1.4 The thesis

**An agent is a user who never files a bug report.**

This is the load-bearing claim of the guide. Every design rule that follows is downstream of it. A human who cannot complete a task gets frustrated and complains; a developer who cannot integrate files an issue; an agent that cannot use your surface does none of these. It produces the most plausible thing it can from the material it has — a wrong call, a guessed parameter, a hallucinated field — and moves on. The feedback loop that UX and DX rely on is *broken* for AX, and the whole discipline is the work of rebuilding it deliberately.

### 1.5 The boundary declared

This guide deliberately does **not** re-derive material owned by sibling guides. It cross-references by name:

- **The context and disclosure problem** — how many tool definitions fit, deferred loading, tool search, code-execution surfaces, the skills pattern, the measurement evidence, the enterprise angle — is owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) (900 lines). This guide takes disclosure only as a *framing* for AX (§6) and does not restate its taxonomy or evidence.
- **Documentation for coding agents** — what a README, a changelog or an in-repo doc must be for an agent to work a codebase — is owned by [documentation_in_agentic_coding_age.md](../documentation_in_agentic_coding_age.md) (402 lines). This guide treats documentation as one *surface* among seven (§4.4) and does not re-derive its version-control or in-repo practices.
- **API design and governance for human consumers** — versioning, deprecation policy, contracts, the API lifecycle — is owned by [api_governance_guide.md](../api_governance_guide.md) (832 lines). This guide asks what changes when the *caller* is an agent, not what a well-governed API is.
- **The protocol itself** — MCP primitives, transports, server implementation, discovery and registration mechanics — is owned by [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) (697 lines) and [mcp_discovery_guide.md](mcp_discovery_guide.md) (640 lines). This guide cites the specification where it speaks to *tool authorship* (§5, §8) and does not restate the protocol.
- **The internals of building an agent** — context engineering, the harness, the scaffold, skills versus multi-agent systems — is owned by [context_engineering_guide.md](context_engineering_guide.md) (785), [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) (777), [agent_scaffolding_guide.md](agent_scaffolding_guide.md) (934) and [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) (819). This guide inverts that lens (§11) rather than repeating it.
- **Security and governance** — prompt injection, red-teaming, AI governance, the banking compliance frame, and the literature on how agents fail in production — is owned by [prompt_injection_guide.md](prompt_injection_guide.md) (679), [ai_red_teaming_guide.md](ai_red_teaming_guide.md) (709), [ai_governance_framework_guide.md](ai_governance_framework_guide.md) (707), [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) (717), [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) (774), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) (704) and [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md) (1099). This guide borrows their frame in §8, §9 and §10 and cites rather than re-derives.
- **The agent loop** — implementations, dispatch, safety, sub-agents — is owned by [agent_loop_implementation_guide.md](agent_loop_implementation_guide.md). §11 cites it where the AX of one's own agents is concerned.

The edge this guide owns is: **the design of the thing the agent calls, and the methods for knowing whether it can.** No sibling states that as its subject.

---

## 2. Why This Differs from UX and from DX

### 2.1 Three consumers, three feedback loops

The clean way to see the difference is by *who the consumer is and how they complain*.

| | UX | DX | AX |
|---|---|---|---|
| **Consumer** | Human end user | Human developer | Model-driven agent |
| **How it perceives** | Vision, language, layout, motion | Reads documentation and examples, then writes code | Sees only the tool list, schemas and descriptions in context at call time |
| **How it learns** | Explores, hovers, clicks, reads help | Reads docs *before* calling; recalls afterwards | Reads nothing before; infers from the surface *at the moment of the call* |
| **How it fails** | Frustration, abandonment; complains | Runtime error the developer debugs; files an issue | Silent failure: plausible wrong output, invented parameter, retry storm, route-around |
| **How you learn it failed** | Analytics, session replay, complaints | Error telemetry, issue tracker | Only if you deliberately test it — and even then it is model-dependent (§7) |

The last row is the whole reason AX is a distinct discipline rather than a subset of DX.

### 2.2 Why a visual hierarchy and a tooltip convey nothing

An agent consuming a tool surface is, almost always, consuming *text and structure*: a name, a description string, a JSON Schema, an enum, and the text of an error. It does not perceive visual hierarchy — the fact that a headline is large, that a warning is red, that a control is greyed out — because none of that survives serialisation into the model's context. It does not receive a tooltip because a tooltip is a response to a hover, and there is no cursor. It does not experience an "implicit affordance" — the convention that a link looks clickable, that a field is obviously required — because an implicit affordance is a *learned human convention*, and the agent has no shared visual culture with your designers.

The corollary is the first practical rule of AX: **anything that matters to the call must be explicit in the schema or the description, or it does not exist for the consumer.** Anthropic puts the point in the form of an instruction to the author: write the description *"how you would describe your tool to a new hire on your team… make it explicit"*, because *"specialized query formats, definitions of niche terminology, relationships between underlying resources"* are context a human author brings implicitly and an agent does not. **(vendor engineering post — Anthropic, 11 September 2025)**. OpenAI's version of the same rule is the **"intern test"**: *"Can an intern/human correctly use the function given nothing but what you gave the model? (If not, what questions do they ask you? Add the answers to the prompt.)"* **(vendor product documentation — platform.openai.com function-calling guide, checked October 2026)**.

### 2.3 The sharp distinction from DX: read-before vs see-at-call

The difference from DX is sharper than "agents are less smart." It is structural.

**A developer reads documentation before calling.** The DX contract assumes a human who will study the docs, build a mental model, write integration code, and debug against a local copy. The documentation can be long, can live on a separate site, can require a signup, can present a tutorial *before* the reference. Time-to-first-call is measured in minutes or hours, and that is acceptable because the developer amortises it across many calls and a long relationship.

**An agent sees only what is in front of it at the moment of the call.** The tool description, the parameter schema, the enum values, and the error text from the previous attempt are, effectively, the entire documentation the agent will ever consult. It does not browse to your docs site mid-task (unless a human wired a fetch tool, and even then it reads whatever page it lands on, stripped of layout). It does not remember last week's session. It will not "read the getting-started guide first." The DX surface is *read once, by a persistent human*; the AX surface is *read fresh, every turn, by an amnesiac process*.

This produces a specific and sometimes counter-intuitive consequence: **agent-facing documentation belongs in the schema, not only on the website.** Speakeasy makes the operational point with an OpenAPI example — the same endpoint description that suffices for a developer *"is not useful for agents as it doesn't provide the necessary context"*, because the agent *"reads only the endpoint descriptions"*, not the tutorial where the ordering constraint was explained. Their remedy is to push the workflow context (what to call before this, what happens if the resource already exists, what format avoids an error) *into the operation description itself*. **(vendor blog, "Last updated on July 23, 2025")**. Anthropic's remedy is the same idea at the tool layer: enrich the description and the error text with the context a new hire would need. This is why §4.4 treats "documentation" as a *surface the agent reads inline*, not as a website.

### 2.4 What AX borrows from each

AX is not a rejection of UX or DX; it is layered on top of both, and both remain the right frame for their own consumers.

- From **DX** it borrows: predictable contracts, consistent naming, explicit types, meaningful errors, the principle of least surprise.
- From **UX** it borrows: reducing cognitive load, one obvious path, discoverability, and the idea that a good interface *teaches by its shape*.
- It **adds**: the requirement that everything be machine-readable and explicit; that errors be written to *instruct the next attempt*; that responses be bounded because the consumer cannot scroll; and that the designer accept they will never receive a complaint and must instrument for failure instead.

The rest of this guide is the elaboration of those additions.

---
## 3. What an Agent Actually Needs

This section states what an agent needs from a surface as **seven design consequences**. Each is phrased as what the author must *do*, with the reason, and — where a primary source states it — an attribution. Where the requirement is this guide's own synthesis rather than a published rule, it is labelled **(guide's reasoning)**.

### 3.1 A machine-readable, self-describing surface

**Requirement.** The surface must describe itself in a form a program can parse and a model can read: a name, a purpose, a typed parameter list, allowed values, and the shape of what comes back.

**Reason.** An agent cannot infer intent from an endpoint's URL or from a rendered page. If the description is absent, vague, or rendered only as prose on a website, the agent guesses from the name — and names are ambiguous across the hundreds of tools a modern agent may hold. Anthropic reports the failure directly: *"When tools overlap in function or have a vague purpose, agents can get confused about which ones to use."* **(vendor engineering post — Anthropic, 11 September 2025)**. The MCP specification codifies the machine-readable requirement structurally: a tool definition **MUST** be a valid JSON Schema object (not `null`) for its input, may carry an `outputSchema`, and must carry a `name` and `description`. **(specification — MCP, revision 2026-07-28)**.

**Consequence for the author.** Ship a schema, not a paragraph. Every parameter you expect the agent to supply appears in the schema with a type, and every value-set appears as an enum where one exists.

### 3.2 Explicit, deterministic parameters — no implied optionality

**Requirement.** Parameters must be named unambiguously, typed strictly, and must not rely on the caller knowing a default, an ordering, or an implication.

**Reason.** A human developer infers that `user` probably wants an identifier and that omitting `status` probably means "any". An agent does not infer reliably; it fills what it can and leaves what the schema does not force. Anthropic states the naming rule plainly: *"input parameters should be unambiguously named: instead of a parameter named `user`, try a parameter named `user_id`."* **(vendor engineering post — Anthropic, 11 September 2025)**. OpenAI's guidance is to *"use enums and object structure to prevent invalid states"*, with the worked example that a `toggle_light(on: bool, off: bool)` signature *"allows for invalid calls"*. **(vendor product documentation — platform.openai.com, checked October 2026)**. Google's function-calling best practices list *"Strong Typing: Use specific types (integer, string, enum)"* and *"Naming: Use descriptive names without spaces or special characters."* **(vendor product documentation — ai.google.dev, page footer "Last updated 2026-09-23 UTC")**.

**Consequence for the author.** If a parameter is optional, say what the default is *in the description*. If two fields must not both be set, encode that as a schema constraint (an enum, a discriminated union, one object), not as a note. Implied optionality is a silent-failure generator (§8.3).

### 3.3 Errors that instruct — the error message is the prompt for the next attempt

**Requirement.** An error must tell the caller what went wrong *and what to do differently*, in language the caller can act on.

**Reason.** This is the surface's only recovery channel. A human reads a stack trace and reasons; an agent reads the error text as an instruction for its next turn. If the error is an opaque code, the agent has nothing to correct and will either retry the same call or invent a change. The MCP specification distinguishes two error classes on exactly this basis: **protocol errors** — *"issues with the request structure itself that models are less likely to be able to fix"* — and **tool execution errors** — *"actionable feedback that language models can use to self-correct and retry with adjusted parameters"* — and instructs that clients **SHOULD** provide the latter to the model *"to enable self-correction"*. **(specification — MCP, revision 2026-07-28)**. Anthropic's guidance is the same at the tool level: *"prompt-engineer your error responses to clearly communicate specific and actionable improvements, rather than opaque error codes or tracebacks."* **(vendor engineering post — Anthropic, 11 September 2025)**.

**Consequence for the author.** Write every error as if it will be read by a competent new colleague who cannot ask a follow-up question: name the offending field, the expected format, and the valid range or an example. The MCP spec's own example is worth copying in shape — *"Invalid departure date: must be in the future. Current date is 08/08/2025."* **(specification)**. That sentence contains the failure, the rule, and a datum the caller needs to fix it.

### 3.4 Idempotency, because agents retry

**Requirement.** A mutating operation must be safe to attempt more than once with the same arguments, or must carry an explicit idempotency key, and must say which it is.

**Reason.** An agent retries on timeout, on partial failure, on a transport error, and on its own uncertainty. Where a human developer reads "retry-safe" in the docs and obeys, an agent may retry a non-idempotent call and duplicate an effect. This requirement is stated most sharply by OpenAI's guidance to *"combine functions that are always called in sequence"* and to *"not make the model fill arguments you already know"* — both reduce the number of round trips, and every reduced round trip is a reduced chance of a duplicated side effect. **(vendor product documentation — platform.openai.com, checked October 2026)**. The linkage from retry behaviour to idempotency *design*, however, is **this guide's reasoning**: the primary sources describe the retry behaviour and the sequencing guidance but do not, in the pages read, state idempotency as a named rule for agent-facing tools.

**Consequence for the author.** Mark read-only and destructive tools explicitly — the MCP `annotations` object carries `readOnlyHint`, `destructiveHint` and `idempotentHint` for exactly this purpose **(specification, 2025-06-18 revision onward; the 2026-07-28 concepts page notes clients MUST treat annotations as untrusted unless the server is trusted)** — and give every write a client-supplied idempotency key so a retry collapses to one effect.

### 3.5 Bounded cost and bounded response size, because an agent cannot scroll

**Requirement.** Any response that could be large must be paginated, filtered, ranged or truncated **by default**, with sensible defaults, and the truncation must be disclosed.

**Reason.** A human scrolls, squints and skims; an agent pays for every token and its context is finite. A tool that returns everything crowds out the task. Anthropic states the rule and even gives its own number: *"implement some combination of pagination, range selection, filtering, and/or truncation with sensible default parameter values for any tool responses that could use up lots of context. For Claude Code, we restrict tool responses to 25,000 tokens by default."* **(vendor engineering post — Anthropic, 11 September 2025; the 25,000-token figure is the vendor's own product default, not a benchmark)**. OpenAI's guidance bounds the *input* side: *"Aim for fewer than 20 functions available at the start of a turn at any one time, though this is just a soft suggestion."* **(vendor product documentation — platform.openai.com, checked October 2026; the vendor labels it a soft suggestion)**. Google's guidance is *"Keep active set to 10-20 tools maximum."* **(vendor product documentation — ai.google.dev, footer 2026-09-23)**.

**Consequence for the author.** Default to the smallest useful response and let the caller ask for more (Anthropic's `response_format: concise | detailed` enum is the reference pattern). When you truncate, say so in the response — a silently truncated result is a wrong answer the caller believes.

### 3.6 Discoverability without browsing

**Requirement.** An agent must be able to *find* the surface and understand its purpose without a human navigating a portal, reading a tutorial, or pasting a URL.

**Reason.** The discovery path is part of the surface. If the only way to learn that a capability exists is a marketing page a person reads, the capability is not agent-reachable, and the agent will route around it (§8.6). The MCP specification supports discovery mechanically — `tools/list` returns the available set, `notifications/tools/list_changed` signals changes, and the set *"MUST NOT vary per-connection or as a side effect of other requests"* — so that a client can build a stable view of what exists. **(specification — MCP, 2026-07-28)**. Server-level discovery (registries, directories, how a human or platform adopts a server) is a different problem, owned by [mcp_discovery_guide.md](mcp_discovery_guide.md); the *disclosure* of which tool to load is owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md). This guide's contribution is only the design consequence: **the description is the discovery surface, because for the agent the description is the only documentation that reliably reaches it.**

**Consequence for the author.** Names should reflect natural task subdivisions so that search — whether a human-driven tool search or a model's own selection — resolves unambiguously. Anthropic's namespacing guidance (`asana_search`, `jira_search`) exists precisely to make selection possible at scale. **(vendor engineering post, 11 September 2025)**. OpenAI adds that for deferred tools *"the namespace helps the model choose what to load; the function description helps it use the loaded tool correctly."* **(vendor product documentation, checked October 2026)**.

### 3.7 No surface reachable only through a user interface

**Requirement.** Every capability you want an agent to use must be reachable by a program, not only by a person clicking through a UI.

**Reason.** This is the requirement that most often reveals that a company has no agent strategy at all: the feature works, but only behind a browser session, a click-through consent flow, a "contact sales" gate, or a captcha. Resend makes the point from the vendor side — *"Agents are simply not going to 'contact sales' or 'book a demo'. Agents should be able to get up and running in milliseconds."* **(vendor blog — resend.com, 19 February 2025)**. The corollary is that *capability parity* between the UI and the programmatic surface is a prerequisite, not a design preference.

**Consequence for the author.** Audit for functions that exist only in the UI, and for steps (interactive consent, manual approval, e-mail verification) that assume a human is present. Each is a place where an otherwise-capable agent simply cannot proceed — and, per the thesis, will not tell you.

### 3.8 The requirements, summarised

| # | Requirement | Primary source stating it | Kind / date |
|---|---|---|---|
| 3.1 | Machine-readable, self-describing surface | MCP specification; Anthropic | specification 2026-07-28; vendor post 2025-09-11 |
| 3.2 | Explicit, deterministic parameters; no implied optionality | Anthropic; OpenAI; Google | vendor posts/docs, 2025–2026 |
| 3.3 | Errors that instruct the next attempt | MCP specification; Anthropic | specification 2026-07-28; vendor post 2025-09-11 |
| 3.4 | Idempotency, because agents retry | annotation model: MCP; retry→idempotency linkage: guide's reasoning | specification; guide's reasoning |
| 3.5 | Bounded cost and response size | Anthropic; OpenAI; Google | vendor post/docs, 2025–2026 |
| 3.6 | Discoverability without browsing | MCP specification; Anthropic; OpenAI | specification/vendor, 2025–2026 |
| 3.7 | No surface reachable only through a UI | vendor blog (Resend) | vendor blog, 2025-02-19 |

---

## 4. The Surfaces

An agent meets a service through seven distinct surfaces. Each has a different failure signature, and a design that is strong on one can be weak on another. This section takes them one at a time: what the surface demands, and where it fails an agent. Where a sibling guide owns the surface, this guide cross-references rather than restates.

### 4.1 The API

**What it demands.** A stable contract, consistent naming, typed and validated inputs, meaningful status codes, and a versioned deprecation policy. For a *human* consumer, all of this is owned by [api_governance_guide.md](../api_governance_guide.md) and is not repeated here.

**What changes when the caller is an agent.** Two things. First, the *description* becomes load-bearing, because the agent reads the OpenAPI `description` in preference to any tutorial. Speakeasy's worked example shows a billing-address endpoint whose developer-facing description is correct but insufficient, and an agent-facing rewrite that adds *when* to call it, *what happens if it is called twice*, and *what format avoids an error*. **(vendor blog — speakeasy.com, "Last updated on July 23, 2025")**. Second, the *granularity* may be wrong: an API built for developers exposes resource operations, while an agent benefits from task-shaped tools (Anthropic: `search_contacts` rather than `list_contacts`). **(vendor engineering post, 11 September 2025)**.

**Where it fails an agent.** When the API is resource-oriented and the task is goal-oriented, the agent must assemble a workflow from primitives it may not order correctly. When optionality is expressed by omission rather than by schema, the agent guesses defaults. When a field is a cryptic identifier (`uuid`, `256px_image_url`) returned without a human-meaningful label, the agent's downstream calls get less reliable — Anthropic reports that resolving opaque identifiers to *"more semantically meaningful and interpretable language"* *"significantly improves Claude's precision in retrieval tasks by reducing hallucinations."* **(vendor engineering post, 11 September 2025)**.

### 4.2 The protocol tool

**What it demands.** The MCP tool contract: a valid JSON Schema input, a `name`, a `description`, optional `outputSchema`, optional `annotations`, and — from the 2026-07-28 revision — a deterministic ordering across requests so the client can cache the list and prompt caching can survive. Tool names **SHOULD** be 1–128 characters, case-sensitive, and limited to letters, digits, `_`, `-` and `.`. **(specification — MCP concepts/tools, 2026-07-28)**. The protocol mechanics — transports, capabilities, server implementation — are owned by [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md); the loading economics are owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md).

**What changes for AX.** The tool is the place where the *unit-of-work* decision is made explicit. The MCP spec's own guidance on stateful handles is a model of AX thinking: *"Because handles outlive any single connection, the server's retention policy should be stated in the creation tool's description (e.g., 'baskets expire after 24 hours of inactivity') so the model can see it when deciding to create state"* — and *"A call against an expired or unknown handle should return a tool execution error that says so, so the model can recover by creating a new one."* **(specification — MCP, 2026-07-28)**. That is the whole discipline in one paragraph: state the lifetime where the model will read it, and make expiry recoverable rather than silent.

**Where it fails an agent.** (a) A tool that merely wraps an endpoint and inherits its granularity — Anthropic names this as *"a common error we've observed"*. (b) A tool description that names the function but not its *purpose relative to sibling tools*, so selection is a coin flip among near-duplicates. (c) Annotations trusted blindly: the spec instructs that clients **MUST** treat annotations as untrusted unless the server is trusted **(specification, 2026-07-28)**. (d) A non-deterministic tool list that breaks caching silently.

### 4.3 The command line

**What it demands.** A stable, documented invocation syntax; meaningful exit codes; output that is parseable and not only pretty-printed; and non-interactive operation.

**What changes for AX.** A CLI is a *surface the agent can use directly*, and this repository's own coverage of execution surfaces treats code and shell as legitimate tool mechanisms (see [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) §5 on code-execution and tool-as-API). The AX consequences are specific: prompts that wait for a human keystroke hang the agent; spinners and progress bars written to stdout corrupt parseable output; and a tool that only prints help when run interactively does not describe itself.

**Where it fails an agent.** Commands that require confirmation (`Are you sure? [y/N]`), commands whose output is intended for a terminal and not a pipe, and commands whose failure is signalled only by a human-readable string rather than a non-zero exit code. Each is a surface the agent cannot reliably drive, and each fails silently from the author's point of view.

### 4.4 The documentation

**What it demands.** For a *human* developer, documentation is a product; for an *agent working a codebase*, its practices are owned by [documentation_in_agentic_coding_age.md](../documentation_in_agentic_coding_age.md).

**What changes for AX.** The documentation surface that matters to an agent consumer is the *inline* text: the tool description, the parameter description, the operation summary. That is the documentation the agent actually reads at call time (§2.3). A separate docs site is, for the agent, generally unreachable — it would need a fetch tool and a landing page stripped of layout. The practical consequence is that documentation has to be *duplicated into the machine-readable surface*, which Speakeasy named and solved with an `x-speakeasy-mcp` extension that carries agent-specific descriptions in the OpenAPI document while renderers like Swagger UI ignore them. **(vendor blog, 2025-07-23)**.

**Where it fails an agent.** When the important guidance lives only in a tutorial ("call A, then B, then C") and never reaches the schema. When the description names a concept the agent has no way to resolve (an internal codename, a niche term) without the definition. Anthropic's rule is the remedy: write the description for a new hire and make the implicit explicit. **(vendor engineering post, 11 September 2025)**.

### 4.5 The error taxonomy

**What it demands.** A consistent, documented set of error conditions, each with a stable machine-readable identifier and an actionable human/model-readable message.

**What changes for AX.** The taxonomy becomes a *control-flow input*. The MCP specification's split into protocol errors and tool execution errors is explicitly ordered by *recoverability* — protocol errors are the ones *"models are less likely to be able to fix"* — and clients **SHOULD** pass execution errors to the model *"to enable self-correction"*. **(specification — MCP, 2026-07-28)**. For AX, the design task is to classify every error by *what the caller should do next*: retry, fix a parameter, obtain a resource, or give up.

**Where it fails an agent.** When a validation failure returns a generic `400` with a generic message, the agent has no correction signal and will retry or hallucinate a fix. When a transient failure and a permanent failure share a code, the agent cannot know whether to retry — and a retrying agent on a non-idempotent call duplicates effects (§3.4). When an error is returned as a *successful* tool result, the silent-failure chain is complete.

### 4.6 The authentication path

**What it demands.** A way for a non-human caller to obtain and present credentials without a human completing an interactive login.

**What changes for AX.** The authentication path is the single most common place an agent is stopped cold, because most authentication was designed to prove a human is present. An agent can hold a token, an API key, or a client credential; it cannot complete a captcha, click a magic link in its own inbox, or wait two business days for onboarding approval. Resend's AX framing is blunt: agents *"should be able to get up and running in milliseconds."* **(vendor blog, 19 February 2025)**. The design consequence is that agent-facing authentication must be **programmatic, delegated, and scoped** — a credential the agent's principal can grant, with the agent's authority bounded to what the principal intends (§10).

**Where it fails an agent.** OAuth flows that require an interactive browser redirect with no device-code or client-credential alternative; consent screens that assume a human reading them; and long-lived credentials with no scoping, which push the whole problem into the liability question (§10) rather than solving it at the surface.

### 4.7 The structured-output contract

**What it demands.** A declared shape for what the surface returns, so the caller can parse it deterministically rather than guess.

**What changes for AX.** The MCP specification allows an `outputSchema` on a tool and instructs servers to validate inputs and sanitize outputs **(specification, 2026-07-28)**. Vendor guidance converges on **structured output** as the mechanism that lets a model reliably consume a result: OpenAI's and Google's function-calling surfaces both pair tool calls with schema-constrained outputs, and Google documents *"Function calling with Structured output"* as a first-class combination **(vendor product documentation — ai.google.dev, footer 2026-09-23)**.

**Where it fails an agent.** When the response format drifts from the schema, when a field is returned sometimes as a string and sometimes as an object, or when pagination is expressed as prose rather than as a cursor field — the agent's parser breaks and the next call is built on a malformed model of the data. Anthropic reports that even the *response structure* (XML, JSON, Markdown) affects performance and *"there is no one-size-fits-all solution"*, recommending the choice be made by evaluation. **(vendor engineering post, 11 September 2025)**.

### 4.8 The surfaces, compared

| Surface | What it demands | Where it most often fails an agent | Owned by |
|---|---|---|---|
| API | Stable contract, typed inputs, good status codes | Resource-shaped where the task is goal-shaped; description too thin | [api_governance_guide.md](../api_governance_guide.md) (human frame) |
| Protocol tool | Schema, name, description, annotations, deterministic order | Endpoint-wrapping; ambiguous names; broken caching | [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) |
| Command line | Stable syntax, exit codes, parseable output | Interactive prompts; human-only output; no exit code | — (this guide) |
| Documentation | Inline description that carries workflow context | Guidance stranded in a tutorial the agent never reads | [documentation_in_agentic_coding_age.md](../documentation_in_agentic_coding_age.md) (coding agents) |
| Error taxonomy | Classified, actionable, recoverability-aware errors | Opaque 400s; transient vs permanent undivided | — (this guide) |
| Authentication path | Programmatic, delegated, scoped credentials | Interactive-only flows; unscoped long-lived keys | — (this guide) |
| Structured output | Declared shape, schema-validated, paginated | Format drift; prose pagination | — (this guide) |

The through-line: **every surface fails the same way — quietly.** The authoring team sees a green deploy, a healthy dashboard, and no complaints, while the agent silently takes a different route or builds a wrong answer on a thin description. That is the subject of §8.

---
## 5. The Design Principles

Seven principles, each stated as a **rule with its reason**. Every rule carries an attribution: who published it, when, and of what kind — or the label **(guide's reasoning)** where it is this guide's synthesis rather than a quotation from a source. None is presented as a finding; a vendor's recommendation is a recommendation.

### 5.1 Name the intent, not the mechanism

**Rule.** Name a tool for the *outcome it produces*, not for the *system it touches*.

**Reason.** The agent selects by description and name against a task stated in intent. A tool named `orders_api_v2_put` tells the agent about a mechanism; `submit_refund` tells it about an outcome, which is what the task contains. OpenAI's guidance is to *"write clear and detailed function names"* and to make functions *"predictable and intuitive"* under the *"principle of least surprise."* **(vendor product documentation — platform.openai.com function-calling guide, checked October 2026)**. Anthropic's version is the `search_contacts` vs `list_contacts` contrast. **(vendor engineering post — Anthropic, 11 September 2025)**.

**Failure if ignored.** Selection ambiguity: several mechanism-named tools look equally plausible for one intent, and the agent's choice becomes arbitrary — a silent-failure class (§8.2).

### 5.2 One obvious way to do a thing

**Rule.** For any single intent, expose exactly one tool. Do not ship two callable paths to the same outcome.

**Reason.** Duplicate capability is the direct cause of near-miss selection errors. Anthropic: *"When tools overlap in function or have a vague purpose, agents can get confused about which ones to use."* **(vendor engineering post, 11 September 2025)**. The principle is the agent-facing analogue of a rule developers already accept — one obvious way — and OpenAI encodes a version of it by advising authors to *"combine functions that are always called in sequence"* and to remove parameters the model cannot usefully set. **(vendor product documentation, checked October 2026)**.

**Failure if ignored.** The agent picks the wrong twin. Because both succeed, there is no error and no complaint — only a wrong effect (§8.2).

### 5.3 A tool is a unit of work, not an endpoint

**Rule.** Design a tool around a task a competent colleague would recognise as a step, and let it perform the several API calls that step requires underneath.

**Reason.** This is the single most-repeated published rule, and it is stated by two independent vendors. Anthropic: *"A common error we've observed is tools that merely wrap existing software functionality or API endpoints"*, and tools *"can consolidate functionality, handling potentially multiple discrete operations (or API calls) under the hood"* — with the worked examples `schedule_event` (instead of `list_users` + `list_events` + `create_event`), `search_logs` (instead of `read_logs`) and `get_customer_context` (instead of `get_customer_by_id` + `list_transactions` + `list_notes`). **(vendor engineering post — Anthropic, 11 September 2025)**. OpenAI states the same rule for actions: *"if you always call mark_location() after query_location(), just move the marking logic into the query function call."* **(vendor product documentation, checked October 2026)**.

**Failure if ignored.** The agent must chain primitives in the right order from a thin description, consuming context and creating the opportunity for a wrong intermediate call. The unit-of-work decision is also where authorisation and liability attach (§10).

### 5.4 State the preconditions explicitly

**Rule.** If a call has an ordering constraint, a required prior state, or a data dependency, put that in the description — not in the tutorial.

**Reason.** Preconditions are exactly the context a human author holds implicitly and an agent does not. Speakeasy's corrected example adds *"It must be called before the cart can proceed to checkout"* and *"If the cart already has a billing address, this call will overwrite it"* to an operation whose developer-facing text said neither. **(vendor blog — speakeasy.com, 2025-07-23)**. The MCP specification's handle guidance — state the handle's retention in the *creation* tool's description — is the same rule applied to state lifetime. **(specification — MCP, 2026-07-28)**.

**Failure if ignored.** The agent calls the right tool at the wrong time, receives a business-logic error it cannot interpret, and either retries or invents a workaround (§8.3).

### 5.5 Idempotent semantics

**Rule.** Say, in the surface itself, whether a call is safe to repeat. Make reads and repeats harmless; give writes a caller-supplied idempotency key.

**Reason.** The agent will retry — on timeout, on transport error, on ambiguity. The MCP `annotations` object carries `readOnlyHint`, `destructiveHint` and `idempotentHint` precisely so a client can reason about a tool's behaviour. **(specification — MCP, from the 2025-06-18 revision; the 2026-07-28 concepts page restates annotations and instructs clients to treat them as untrusted unless trusted)**. The MCP spec's error-handling section also distinguishes transient from permanent failure, which is what a caller needs to decide *whether* to retry. **(specification, 2026-07-28)**. **The linkage of "agents retry" to "therefore make writes idempotent" is this guide's reasoning**; the sources describe the retry behaviour and the annotation vocabulary but do not, on the pages read, state idempotency as a named tool-authoring rule.

**Failure if ignored.** A duplicated payment, a double-sent notification, a duplicate row. The author sees a successful call in the logs and a duplicated business object in the database, with no idea which agent retried and why.

### 5.6 Bounded and pageable responses

**Rule.** Default to the smallest useful response; make more available on request; disclose truncation and provide a cursor for continuation.

**Reason.** An agent cannot scroll and pays per token. Anthropic: pagination, range selection, filtering and truncation *"with sensible default parameter values"*, and explicitly *"steer agents with helpful instructions"* when you truncate. **(vendor engineering post, 11 September 2025)**. Google's and OpenAI's guidance bounds the *tool-set* side, not only the response side. **(vendor product documentation, 2025–2026)**.

**Failure if ignored.** Context exhaustion (§8.5): one oversized response crowds out the task and the agent fails late, having spent the budget. The failure looks like a model failure and is an interface failure.

### 5.7 The failure of a tool surface that mirrors an API rather than a task

**Rule.** Do not generate a tool list mechanically from your API specification and call it a tool surface.

**Reason.** This is the summary failure of the previous six, and it is named by Anthropic as the common error: wrapping endpoints regardless of whether they suit an agent, producing *"too many tools or overlapping tools"* that *"distract agents from pursuing efficient strategies."* **(vendor engineering post, 11 September 2025)**. A mechanical mirror produces: mechanism names (§5.1), duplicate paths (§5.2), endpoint-sized units (§5.3), unstated preconditions (§5.4), no idempotency vocabulary (§5.5), and unbounded resource dumps (§5.6) — all at once.

**Failure if ignored.** The surface appears complete and is nearly unusable. This is the archetypal AX failure: everything works, selection is unreliable, and nobody complains.

### 5.8 The principles and their sources

| # | Principle | Source | Kind / date |
|---|---|---|---|
| 5.1 | Name the intent, not the mechanism | Anthropic; OpenAI | vendor post 2025-09-11; vendor docs, checked 2026-10 |
| 5.2 | One obvious way to do a thing | Anthropic; OpenAI | vendor post 2025-09-11; vendor docs, checked 2026-10 |
| 5.3 | A tool is a unit of work, not an endpoint | Anthropic; OpenAI | vendor post 2025-09-11; vendor docs, checked 2026-10 |
| 5.4 | State the preconditions explicitly | Speakeasy; MCP specification | vendor blog 2025-07-23; specification 2026-07-28 |
| 5.5 | Idempotent semantics | annotation vocabulary: MCP spec; retry→idempotency linkage: guide's reasoning | specification; guide's reasoning |
| 5.6 | Bounded and pageable responses | Anthropic; OpenAI; Google | vendor post/docs, 2025–2026 |
| 5.7 | Do not mirror the API; model the task | Anthropic | vendor post 2025-09-11 |

The seven principles are, in effect, the design consequences of §3 applied at the tool layer. They are the checkable items in the scorecard (§12).

---

## 6. Progressive Disclosure — the AX Framing Only

Progressive disclosure — showing a consumer only what it needs at the moment it needs it, and revealing the rest on demand — is the mechanism most often proposed for keeping a large tool surface usable by an agent. **This guide does not own that problem and does not re-derive it.** The context-and-disclosure problem — the token economics, the taxonomy of approaches, deferred loading and tool search, code-execution surfaces, the skills pattern, the measurement evidence, the trade-offs and the enterprise angle — is owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) (900 lines). Read that guide for the mechanism. This section states only the **AX framing**, which is a different question:

- **Disclosure is a UX property of the surface, restated for an agent.** For a human, progressive disclosure means not showing everything at once. For an agent, it means not *loading* everything at once, because the cost is paid in the model's context and in its selection accuracy. The AX framing is that **the disclosed surface is the interface** — what is not disclosed is, for the call at hand, not part of the interface at all.
- **The AX decision that disclosure forces is "what is the unit of disclosure."** A tool, a namespace, a skill body, or a code API are different units; choosing one is a design decision with a direct AX consequence, because the unit determines what the agent can perceive without a lookup.
- **The disclosure mechanism changes the discovery path (§3.6).** If tools are deferred behind a search, the *description* becomes even more load-bearing, because it is what the search matches and what the agent reads after loading. A deferred tool with a weak description is undiscoverable in practice.

The one cross-referenced caution this guide adds: **disclosure is a mitigation for a large surface, not a fix for a bad one.** A surface of coherent, task-shaped tools (§5.3) works with or without disclosure; a surface of 800 endpoint-shaped tools fails with or without it, and disclosure only hides the defect from the designer's dashboard. That is the point §13 makes in the Cymbal Bank example: fix the inventory first.

---

## 7. Measurement — How You Know an Agent Can Use It

### 7.1 The agent audit

The only honest measure of whether a surface is agent-usable is an **agent audit**: an experiment in which an agent is given a task on the surface **with no human help**, and its attempt is observed and recorded. This is not a survey, not an expert review, and not a self-assessment. It is the AX analogue of a usability test, with one difference that governs the whole method: **the participant is a model, and the participant's behaviour will change when the model changes.**

The method, in the shape the primary sources support:

1. **Write realistic tasks, not toy ones.** Anthropic's evaluation guidance is explicit: tasks *"should be inspired by real-world uses and be based on realistic data sources and services"* and *"might require multiple tool calls — potentially dozens."* It gives strong and weak examples, and the weak ones are single-call lookups. **(vendor engineering post — Anthropic, 11 September 2025)**.
2. **Pair each task with a verifiable outcome.** Anthropic recommends a verifier — from exact string comparison to a model judge — and warns against overly strict verifiers that reject correct answers for formatting. **(same source)**. The outcome must be checkable without a human reading the transcript, or the audit does not scale and will not be repeated.
3. **Give the agent the surface and nothing else.** No human on the call, no hint, no walkthrough. This is what makes the audit a measurement of the *surface* rather than of the interviewer.
4. **Run many times and keep the transcripts.** Anthropic recommends running the evaluation programmatically with simple agentic loops, one loop per task, and reading the raw transcripts — *"including tool calls and tool responses"* — because agents *"don't necessarily know the correct answers and strategies"* and what they omit can matter more than what they say. **(same source)**.

### 7.2 What to instrument

Anthropic names the metrics it collects, and they are the right starting set: **top-level task accuracy; total runtime of individual tool calls and of tasks; total number of tool calls; total token consumption; and tool errors.** It adds the diagnostic reading: *"Lots of redundant tool calls might suggest some rightsizing of pagination or token limit parameters is warranted; lots of tool errors for invalid parameters might suggest tools could use clearer descriptions or better examples."* **(vendor engineering post — Anthropic, 11 September 2025)**. Speakeasy's complementary instrumentation is *traceability*: tagging agent-originated requests (`X-Agent-Request: true`, `X-Agent-Name`) so agent actions can be distinguished from human actions in logs. **(vendor blog — speakeasy.com, 2025-07-23)**. The distinction matters for measurement because a success metric that pools human and agent traffic measures neither.

A workable AX instrument set:

| Instrument | What it reveals | Source |
|---|---|---|
| Task-completion rate | Whether the task is achievable at all | Anthropic, 2025-09-11 |
| Retries per task | Interface friction / idempotency exposure | Anthropic; guide's reasoning on idempotency |
| Wrong-tool / wrong-parameter rate | Description and schema quality | Anthropic, 2025-09-11 |
| Token consumption per task | Response-size and discovery cost | Anthropic, 2025-09-11 |
| Tool-error rate (by error class) | Error-taxonomy quality (§4.5) | Anthropic, 2025-09-11 |
| Route-arounds (task done without the surface) | Silent failure (§8.6) | guide's reasoning |
| Agent-vs-human request split | Whether the metric is measuring agents at all | Speakeasy, 2025-07-23 |

### 7.3 Distinguishing an interface failure from a model failure

A failed audit does not, by itself, tell you whether the *surface* or the *model* is at fault — and the two have opposite remedies. The available discriminators:

- **Change the model, keep the surface.** If the same task on the same surface succeeds on one model and fails on another, the failure is at least partly model-dependent (§7.4). If it fails on every model tried, suspect the surface.
- **Read the transcript for a *localisable* wrong step.** An agent that calls a plausible-but-wrong tool, or fills a parameter the schema described poorly, or builds on a truncated response, is exposing a surface defect. An agent that never forms a coherent plan is more likely a model-capability limit.
- **Ask whether a competent human, given only the schema and description, would succeed.** This is OpenAI's "intern test" repurposed as a diagnostic: if a capable human would also fail given only what the agent was given, the surface is under-described, not the model under-capable. **(rule origin: vendor product documentation — platform.openai.com, checked October 2026; the use as an audit discriminator is the guide's reasoning)**.
- **Check whether the failure is in perception, selection, or execution.** Anthropic's framing that agents *"might call the wrong tools, call the right tools with the wrong parameters, call too few tools, or process tool responses incorrectly"* is effectively a diagnostic taxonomy of interface failure points. **(vendor engineering post, 11 September 2025)**.

One published data point bears on the split and must be quoted carefully: a benchmark-based study reports that a large share of agent failures are *cognitive* rather than *tool-call* failures — **63.3%** in MCP-Atlas (arXiv:2602.00933). **(verified as a measured finding in one benchmark; flagged as a generalisation — it is a single benchmark's diagnostic taxonomy, not a universal constant)**. The AX reading is not "therefore interfaces don't matter"; it is "therefore an interface audit measures one slice of the failure mass and should say so."

### 7.4 The honesty caveat: an audit is a dated observation, not a certification

This is the methodological heart of the section, and the primary sources force it.

**The same test gives different results on different models.** Anthropic reports that the choice between prefix- and suffix-based namespacing has *"non-trivial effects on our tool-use evaluations"* and that *"effects vary by LLM"*, instructing authors to *"choose a naming scheme according to your own evaluations."* **(vendor engineering post, 11 September 2025)**. A surface that passes on one model is not thereby agent-usable in general; it is usable by *that model on that date*.

**The same test gives different results after model updates.** Model updates change behaviour without any change to the surface. An audit result is therefore tied to a (surface, model, version, date) tuple and expires.

**Therefore:** an agent audit is a **dated observation**. It is not a certification, not a badge, and not a guarantee. A scorecard (§12) records *"at model X version Y, on date Z, this surface completed N of M tasks"* — and says nothing about tomorrow. Writing it down as anything stronger is the same category error as treating a one-time usability study as a permanent guarantee, except with a consumer whose behaviour the vendor changes on its own schedule.

**Practical rule.** Date every audit, record the exact model and version, re-run after any model update or any change to the surface's descriptions, and treat a stale audit as no evidence at all. Any vendor offering a permanent "AX certified" mark is offering a claim this method cannot support.

The closest thing to published AX tooling points the same way. Netlify's open-source **AXIS** framework runs agents against scenarios and scores four dimensions, and it explicitly frames a score as *"a baseline and a direction"* — with the vendor's own caveat that *"most services score lower than expected on their first run"* and that *"the value is knowing where you stand and what to improve, not achieving a number."* **(vendor tooling and product documentation — netlify.com/agent-experience and axis.run, checked October 2026)**. Even the party that coined AX presents measurement as a dated direction rather than a certificate, which is consistent with the method stated here.

### 7.5 Why the instrumented channels miss silent failure

Ordinary product instrumentation is built to measure *human* success, and it misses the AX failure for structural reasons:

- **There is no complaint channel.** A human files a bug; an agent does not. The support queue is empty because the failure never becomes a support ticket.
- **The success metric is inflated by route-arounds.** An agent that achieves the user's goal by a different path registers as a success on every outcome metric — and as a failure on none — even though your surface was bypassed (§8.6). You learn your feature is unused only if you measure *adoption of the surface*, not *achievement of the goal*.
- **Telemetry measures the call, not the attempt.** A call that never happens leaves no trace. The most damaging AX failure is a capability the agent never discovered, and discovery failures are, by construction, absent from invocation logs.
- **Aggregate accuracy hides interface defects.** A dashboard that shows "94% task success" pools model capability, task difficulty and surface quality into one number that cannot be acted on. Only the transcript-level instrument set of §7.2 localises the defect.

The remedy is the audit itself plus two specific telemetry choices: instrument **surface adoption** (was the intended tool ever called, for tasks where it should have been?), and instrument **agent-vs-human traffic** separately so that a human-side success story does not mask an agent-side failure. This is the measurement expression of the thesis: because the agent will not report the failure, the author must go looking for it.

---
## 8. The Failure Modes

This is the spine of the guide. Every other section is ultimately in service of understanding this one fact: **the failure mode that defines agent-facing design is silent failure.** A human complains; a developer files an issue; an agent that cannot use your surface does not report a bug. It produces a plausible wrong answer, invents a parameter, retries until its context runs out, or quietly takes a different route. This section states the six failure modes, each with its **cause** and its **guardrail**, then explains why the ordinary feedback channels do not catch any of them. The cross-references to the repository's failure and feedback literature are deliberate: [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md) own the general taxonomy; this section owns the *interface-attributable* subset.

### 8.1 Silent failure

**Definition.** A failure that produces no signal to the surface's author. There is no error report, no ticket, no failed transaction the author can see. There is only an agent that did something, and a caller who may or may not notice the result is wrong.

**Cause.** Structural, not incidental. As §2.3 established, the agent reads only what is in front of it and has no persistent relationship with the author. The feedback loop that UX and DX depend on — the user complains, the developer files — has no participant at the agent end. The agent is not being polite; it has no channel and, even if it had one, no expectation that anyone is listening.

**What silent failure looks like on each surface type:**

| Surface | Silent-failure appearance | Why the author cannot see it |
|---|---|---|
| API | Agent calls a neighbouring endpoint that also returns 200; wrong object created | Both calls succeed; the dashboard is green |
| Protocol tool | Agent selects a near-duplicate tool; effect is plausible but wrong | No error is raised; the tool list worked |
| Command line | Agent pipes output, spinner corrupts the parse, agent proceeds on garbage | Exit code is 0; no human watched the terminal |
| Documentation | Guidance never reaches the agent; agent assumes a wrong default | No page view is registered because the agent never browsed |
| Error taxonomy | Agent receives an ambiguous 400 and retries a non-idempotent write, duplicating it | The retry "succeeded" |
| Auth path | Agent cannot obtain a credential and abandons the feature | No request is ever made; nothing to log |
| Structured output | A field's type drifts; agent builds the next call on a wrong model of the data | Parse does not throw; the wrong call may still 200 |

**Guardrail.** Assume no complaint will arrive. Instrument surface adoption, run dated agent audits (§7), and treat the *absence* of agent traffic on a surface you built for agents as the primary alarm — because it is the only signal you will get.

### 8.2 The plausible wrong call

**Definition.** The agent calls a tool that is *wrong for the task* but *plausible by its name and description*, and the call succeeds.

**Cause.** Ambiguous or overlapping tool descriptions (§5.1, §5.2). Anthropic names the mechanism: tools that overlap in function *"can get confused about which ones to use."* **(vendor engineering post — Anthropic, 11 September 2025)**. Speakeasy's refund example shows the same root: an operation description that does not state *when* to call it. **(vendor blog — speakeasy.com, 2025-07-23)**.

**Guardrail.** One obvious way per intent; names that reflect natural task subdivisions; descriptions that state purpose *relative to sibling tools*. Measure the wrong-tool rate in the audit (§7.2) — it is the metric that catches this, and it is invisible in production because both calls succeed.

### 8.3 The invented parameter

**Definition.** The agent supplies a parameter the schema did not define, or omits one the description did not flag as required, or guesses a value within an under-specified field.

**Cause.** Implied optionality and thin typing (§3.2). A schema that permits a default to be inferred invites a guess. A description that does not state a format invites a hallucinated one. Anthropic's own reported example is the model appending `2025` to a search `query` because the description did not forbid it — a hallucination that degraded results until the description was fixed. **(vendor engineering post, 11 September 2025)**.

**Guardrail.** Enums instead of free strings (OpenAI's `toggle_light(on, off)` example); explicit defaults in descriptions; `additionalProperties: false` where the input shape is closed; and error responses that name the offending parameter and its expected format so the next attempt can correct (§3.3).

### 8.4 The retry storm

**Definition.** The agent retries a failing call repeatedly — sometimes with the same parameters, sometimes with small variations — consuming its budget and, on a non-idempotent surface, duplicating effects.

**Cause.** Two combined defects: an error that does not teach the caller what to change (§3.3), and a surface that does not declare whether a call is safe to repeat (§3.4). An agent that receives an ambiguous failure has no basis for a *different* next attempt, so it repeats the old one. Anthropic's diagnostic reading is direct: *"lots of tool errors for invalid parameters might suggest tools could use clearer descriptions or better examples."* **(vendor engineering post, 11 September 2025)**.

**Guardrail.** Errors that instruct; a transient/permanent distinction (the MCP specification's protocol-vs-execution error split is ordered by recoverability **(specification, 2026-07-28)**); explicit idempotency semantics on writes; and, where the surface can enforce it, a bounded retry with a discriminating response.

### 8.5 Context exhaustion from oversized material

**Definition.** A single response — a thousand-row result, a long document, an un-truncated log — consumes so much of the agent's context that the task fails later, or the agent loses the instruction it was following.

**Cause.** An unbounded response with no pagination default (§3.5, §5.6). Anthropic's guidance to paginate, filter and truncate with sensible defaults exists precisely because the consumer cannot scroll: *"an LLM agent uses a tool that returns ALL contacts and then has to read through each one token-by-token, it's wasting its limited context space on irrelevant information."* **(vendor engineering post, 11 September 2025)**.

**Guardrail.** Small default responses with an explicit "more available" cursor; a `response_format` enum for verbosity; and disclosed truncation. Note that this failure *looks like a model failure* — the agent "got confused" — which is why §7.3's interface-vs-model discriminator matters: reading the transcript usually shows the context was filled by one oversized response the author chose to return.

### 8.6 The agent that routes around the feature entirely

**Definition.** The agent never uses your surface. It finds another path to the goal — a different tool, a workaround, a scrape, a generic search — and succeeds without you.

**Cause.** A broken discovery path (§3.6), an unreachable auth path (§4.6), or a surface that is simply harder to use than the alternative. The agent picks the easiest route to the *task*; it does not pick *your feature*. Resend's onboarding framing captures the mechanism: agents *"will pick whatever tool is easiest to perform a task."* **(vendor blog — resend.com, 19 February 2025)**.

**Guardrail.** Capability parity with the UI and the API (§3.7); programmatic, delegated authentication (§4.6); and, above all, a *discoverable* description. Detect it by measuring surface adoption, not task success — a route-around registers as a success everywhere except on the instrument you almost certainly did not build.

### 8.7 Why the usual feedback channels do not catch any of this

The reason AX needs its own discipline is that every channel a product team relies on is blind to silent failure:

- **Error reports and bug trackers** — populated by humans and developers. An agent files neither. A surface with a broken description can run for months with zero tickets.
- **Telemetry that measures human success** — session metrics, conversion, completion. These measure the *user's* outcome, and an agent that routes around your surface (or a human who succeeds anyway) registers a success. The defect is invisible by construction (§7.5).
- **Support tickets** — a human channels a problem into a ticket; an agent has no equivalent. Worse, a human working *through* an agent may not realise the agent's odd output was an interface defect rather than a model limit.
- **Model-side evaluations** — measuring the model tells you about the model. Anthropic's own method (§7.1) is an *evaluation of the tools*, run with the model as instrument, which is the point: the model is the probe, the surface is the subject.
- **Dashboards** — aggregate success rates pool model, task and surface into one number that cannot localise a defect (§7.5).

The only reliable detector is a **deliberate, dated agent audit** (§7). Everything else is a feedback loop with the complaining party removed.

---

## 9. Security — the Material You Publish Is the Injection Vector

### 9.1 The core inversion

In every other design discipline, the material you publish to help the consumer is *inert*: a human reads the docs, a developer reads the README, the text informs but does not execute. **For an agent consumer, the material you publish is read as instructions.** The tool description, the parameter text, the documentation page, and the error string all enter the model's context as prompt content, alongside the user's task and the system prompt. There is no structural boundary between "description" and "instruction," because the model has no privileged channel for the former. The consequence is the central AX security fact:

**An AX design decision is a security decision.**

Every time you decide what a description says, you are deciding what instructions reach a model — yours and every other server's. The MCP specification acknowledges the resulting trust problem directly: clients **MUST** consider tool annotations untrusted unless they come from a trusted server, and servers **MUST** sanitize outputs. **(specification — MCP tools, 2026-07-28)**. The specification's own security-considerations list requires servers to validate all inputs, implement access controls, rate-limit invocation, and sanitize outputs, and tells clients to validate results before passing them to a model. **(same source)**.

### 9.2 Where the exposure lives

- **Tool descriptions and names** are the classic surface: content in a description is read by every agent that loads the tool, so a malicious or compromised description can steer *other* tools' behaviour. This is **tool poisoning**, and it is a documented class in the repository's security literature — see [prompt_injection_guide.md](prompt_injection_guide.md) and [ai_red_teaming_guide.md](ai_red_teaming_guide.md). This guide does not re-derive the attack taxonomy; it states the AX consequence: because descriptions are the discovery surface (§3.6), you cannot simply remove them, so you must govern them as code.
- **Error text** is an injection surface with a short path to effect. Anthropic recommends prompt-engineering error responses to *instruct* the caller **(vendor engineering post, 11 September 2025)**; the security corollary is that error text is *attacker-influenced input to the model* whenever the error is derived from user-supplied data. An error that echoes untrusted input into a "helpful instruction" is an injection vector.
- **Documentation and descriptions** must be treated as **static prompt content with a supply chain**. If a description is generated, edited by a partner, or auto-derived from an API spec, its integrity is a supply-chain question. The repository's disclosure guide documents the compensating controls (hash-pinning a tool definition over name, description and schema, re-verification before execution, re-approval on change) and this guide cross-references them rather than restating ([mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md); [prompt_injection_guide.md](prompt_injection_guide.md)).
- **Tool results** are read as *content*, and content can be instructions. Server-side output sanitisation is the required control **(specification — MCP, 2026-07-28)**.

### 9.3 The design rules this implies

1. **Treat every agent-facing string as prompt text and review it as such.** A description, a name, an error template and a doc snippet are code from a security standpoint. Change control applies.
2. **Never concatenate untrusted input into instructions.** Error messages and "helpful" guidance must be constructed, not interpolated from caller data.
3. **Publish the minimum that steers correctly.** A description should carry the context the agent needs and nothing that could steer it elsewhere; verbosity in prompt-visible text is attack surface as well as token cost.
4. **Keep the human-in-the-loop control where the consumer cannot remove it.** The MCP specification instructs that there **SHOULD** always be a human in the loop with the ability to deny invocations **(specification — MCP tools, 2026-07-28)** — but §10 observes that this control is exactly the one an autonomous consumer is designed to bypass, which makes its *enforcement point* a design and liability question, not merely a UI feature.
5. **Enforce authority server-side, never by prompt instruction.** A restriction expressed in a description is enforced by instruction-following alone; the repository's security material is consistent that the only durable control is server-side authorisation.

### 9.4 What this section does not do

It does not re-derive prompt injection, tool poisoning, the OWASP MCP Top 10, or the red-teaming method. Those are owned by [prompt_injection_guide.md](prompt_injection_guide.md) (679 lines) and [ai_red_teaming_guide.md](ai_red_teaming_guide.md) (709 lines). The contribution here is the *inversion*: for AX, the security surface is the usability surface. The thing you publish to make the agent succeed is the thing an attacker uses to make it fail.

---

## 10. Liability and Authority — Open Questions, Not Answers

This section states **open architectural questions**. It asserts **no jurisdiction's rule and no legal conclusion**; where a regulatory frame is relevant, it points to the repository's compliance and governance guides for the frame. The questions here are structural, and the design choices that raise them are choices this guide's earlier sections recommend — which is precisely why they are uncomfortable.

### 10.1 Who is contractually bound when the caller is an agent?

When a person calls an API, the human or the organisation they represent is the counterparty. When an agent calls on a person's behalf, the mapping from *actor* to *principals* is not obvious: the agent's developer, the agent's operator, the platform hosting the agent, the human who delegated the task, and the institution whose surface is called all have a stake, and a single call can implicate all of them. **Open question.** The surface design cannot answer this, but it can be *made legible* to it — which is why traceability (tagging agent-originated requests and recording which agent, on whose behalf) is treated in §7.2 as a measurement instrument. "Who called" is a design artefact before it is a legal question.

### 10.2 What does a limit mean when the caller can retry?

A spending limit, a rate limit and an approval threshold all assume a caller that does not systematically repeat itself. An agent retries (§3.4, §8.4), so a limit expressed as *per-call* or *per-request* can be defeated by a caller that simply tries again — not maliciously, just mechanically. **Open question.** Does a limit bound the *intent* (this purchase, once) or the *attempts* (each call)? The surface can be designed for either, and the design determines which kind of limit is even expressible. Idempotency keys are the mechanism that makes an intent-bound limit possible; without them, a limit is attempt-bound and a retrying agent finds the gap.

### 10.3 How does a human-in-the-loop control survive a consumer that never sleeps?

The MCP specification instructs that there **SHOULD** always be a human in the loop able to deny an invocation, and that applications **SHOULD** insert confirmation prompts for operations. **(specification — MCP tools, 2026-07-28)**. But an autonomous agent's entire value proposition is that no human is present. **Open question.** If the confirmation prompt is delivered to a human who is asleep, on another timezone, or simply not watching, is the control real or nominal? The design choices — approval *before* the agent runs versus *interstitial* confirmation versus *post-hoc* review with reversal — place the human at different points and produce different liability profiles. Confirmation is not a control; *where the human stands in the flow* is the control.

### 10.4 Whose authority does an agent carry, and how is it bounded?

An agent acting for a human inherits some subset of that human's authority. **Open question.** How is that subset defined and enforced? A delegated credential carries the ceiling, but the *unit of work* (§5.3) is what the credential is scoped to — a tool that bundles five internal calls to complete a task bundles five authorities, and the reviewer approving the tool may not see that. This is the sharpest place where an AX design decision (what a tool's unit of work is) becomes a governance decision (what the agent is thereby authorised to do).

### 10.5 What does auditability mean when the decision is non-deterministic?

For a deterministic system, an audit record shows *what happened*. For an agent, the interesting question is *why* — and the reasoning is not fully recoverable from the outcome. Anthropic notes that agents are non-deterministic *"even with the same starting conditions"* and that *"LLMs don't always say what they mean."* **(vendor engineering post, 11 September 2025)**. **Open question.** What is the auditable unit for an agent action: the final effect, the full transcript including tool calls and responses (Anthropic's own recommended artefact), or the reasoning? The three differ in size, cost and fidelity, and no source read for this guide settles it.

### 10.6 The regulatory frame is elsewhere, and it is thin on this specific question

This guide asserts no rule of any jurisdiction. For the regulatory and governance frame — including the obligations that attach to AI in a regulated institution and the general governance architecture — see [ai_governance_framework_guide.md](ai_governance_framework_guide.md) (707 lines) and [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) (717 lines). On the *specific* question of agent tool permissions, delegated authority and confirmation controls, the repository's disclosure guide records a **negative finding**: no NIST control or ISO/IEC 42001 clause read for that guide requires tool-surface inventory, tool-permission scoping or agent auditability, and no SR 11-7 / OCC / PRA / ECB statement specifically addressing agentic AI tool permissions was located. **(negative finding — recorded in [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md), carried forward here)**. The honest position for this section is therefore: **the questions are real, the design choices that raise them are unavoidable, and the settled answers are not yet in the sources this guide could read.**

### 10.7 Why state questions rather than answers here

Because the design guidance in §3 and §5 is *evidence-backed* (it comes from dated vendor and specification sources) while the liability guidance is *not*. Presenting a confident answer to "who is bound when an agent acts" would be exactly the kind of unsourced assertion this repository's verification discipline exists to prevent. The questions are stated so that the designer makes a *deliberate* choice — and records it as a choice — rather than discovering after the fact that a retrying, never-sleeping consumer walked through a control built for a human.

---
## 11. The Inverted Discipline — the AX of Your Own Agents

Everything so far has asked what *your* surface must be for *someone else's* agent. This section inverts the lens and asks the mirror question: when your institution builds its own agents, the **agent experience those agents have of the tools, harness and skills you give them** determines what they can do — and the same silent-failure dynamics apply. The internals of building an agent are owned by sibling guides; this section states only the AX consequences and cross-references, rather than re-deriving.

### 11.1 Your agents are consumers of your own surfaces

The first inversion is definitional. Every tool description you write, every skill you package, and every internal API you expose to your own agents is an AX surface — and it fails silently in exactly the same way (§8). An internally-built agent that cannot use an internal tool does not file an internal ticket. It hallucinates, retries, or routes around the tool, and the team that built the tool sees nothing. The discipline of §3–§5 applies with full force *inside* the institution: your internal tool descriptions are prompt text, your internal errors are instructions, your internal responses are context budget.

### 11.2 The harness determines the consumer's constraints

The **harness** — the loop, the tool dispatch, the context assembly, the memory tiers — sets the constraints under which every downstream tool must operate. If the harness truncates tool results at a fixed token budget, every tool's response design must respect it. If the harness is KV-cache-aware and pins the tool definitions at the head of the prefix, then the *determinism* of the tool list (MCP's deterministic-ordering rule, §4.2) becomes a harness-level concern. The AX reading: **the harness is where an institution's agent-facing design policy is actually enforced**, because the harness decides what the agent can perceive. This guide does not re-derive harness architecture; see [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) (777 lines) and, for the loop itself, [agent_loop_implementation_guide.md](agent_loop_implementation_guide.md).

### 11.3 The scaffold and the skill determine what the consumer is told

The **scaffold** — the project structure, the prompt files, the tool wiring — and the **skills** — the layered instruction format — are the mechanisms by which an institution tells its own agents what to do. Anthropic's guidance to write tool descriptions *"how you would describe your tool to a new hire"* applies to internal tools word for word **(vendor engineering post, 11 September 2025)**, and the skills pattern is the internal equivalent of instruction-level disclosure. The AX consequence is that an institution's own agents inherit the quality of its internal descriptions: a vague internal tool description is a silent failure inside the org exactly as it is outside. This guide does not re-derive scaffolding or the skills pattern; see [agent_scaffolding_guide.md](agent_scaffolding_guide.md) (934 lines), [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) (819 lines) and [context_engineering_guide.md](context_engineering_guide.md) (785 lines).

### 11.4 The internal analogue of every silent failure

| External AX failure (§8) | Internal analogue |
|---|---|
| Plausible wrong call | The agent picks the wrong internal tool because two teams named theirs similarly |
| Invented parameter | The agent guesses an internal enum value because the schema did not pin it |
| Retry storm | The agent hammers an internal service whose error text did not say "permanent" |
| Context exhaustion | One internal tool dumps a full ledger extract and ends the task |
| Route-around | The agent uses a generic shell/search tool instead of the sanctioned internal one |

The last row is the governance-relevant one: an internal agent that routes around a sanctioned, governed tool and uses a generic path is, from the institution's point of view, shadow behaviour — the same phenomenon as shadow IT, at the tool layer. The repository's disclosure guide treats shadow adoption as an enterprise control problem; this guide notes only that *making the sanctioned tool the easiest path* is the AX remedy, and it is a design remedy, not a policy one.

### 11.5 The self-consistency test for institutional agents

The practical discipline an institution should adopt for its own agents is the one §7 gives for external surfaces, pointed inward: **audit your own agents against your own tools, on a schedule, on a dated model.** Anthropic's own method — build an evaluation of your tools with the model as probe and the transcripts as evidence — is an *internal* practice before it is an external one. **(vendor engineering post, 11 September 2025)**. An institution that ships agents and never audits their experience of the internal tool surface is in exactly the position §10 warns about: it has granted authority to a consumer whose failures it cannot see.

---

## 12. An AX Assessment — a Practical Scorecard

The scorecard below is a **structured assessment instrument**, not a benchmark. It has **named inputs** — each row is a thing you can observe about a surface — and **no benchmark figures**: it does not claim that a passing score predicts production behaviour, and it deliberately carries no numeric thresholds, because any threshold would be a benchmark claim this guide cannot source. Score each row **Yes / Partial / No / Not assessed**, with a note. The **Not assessed** value is deliberate and is the honest default for rows the auditor did not test.

### 12.1 Surface inventory and discovery

| # | Input (observable) | Question | Score |
|---|---|---|---|
| S1 | Machine-readable catalogue | Does every callable surface have a name, description and typed input schema? | |
| S2 | Deterministic ordering | Is the tool/capability list returned in a stable order across requests? | |
| S3 | Inline documentation | Is the workflow context (what to call first, what overrides what) in the schema, not only in a tutorial? | |
| S4 | Discovery without a human | Can a caller learn the surface exists and what it does with no human browsing? | |
| S5 | Capability parity | Is every agent-relevant capability reachable programmatically, not only through a UI? | |

### 12.2 Parameter and description quality

| # | Input (observable) | Question | Score |
|---|---|---|---|
| P1 | Unambiguous names | Are parameters named unambiguously (`user_id`, not `user`)? | |
| P2 | Explicit defaults | Does each optional parameter state its default in the description? | |
| P3 | Enum discipline | Are closed value-sets expressed as enums, not free strings? | |
| P4 | Closed input shapes | Are inputs constrained so invalid states cannot be constructed? | |
| P5 | Description sufficiency | Would a competent new colleague, given only the schema and description, succeed? (the "intern test") | |
| P6 | Sibling disambiguation | Does each description distinguish the tool from near-duplicate siblings? | |

### 12.3 Error and failure behaviour

| # | Input (observable) | Question | Score |
|---|---|---|---|
| E1 | Actionable errors | Do errors name the offending field, the expectation, and an example? | |
| E2 | Recoverability classes | Are transient and permanent failures distinguished? | |
| E3 | Idempotency declared | Does each mutating call declare whether it is safe to repeat? | |
| E4 | Idempotency enforced | Does a write accept a caller-supplied idempotency key? | |
| E5 | No error-as-success | Is every failure returned as a failure, never as a successful result? | |

### 12.4 Response shaping and cost

| # | Input (observable) | Question | Score |
|---|---|---|---|
| R1 | Bounded by default | Does every potentially large response default to a small result? | |
| R2 | Pageable / filterable | Is there a cursor or filter for more, rather than a full dump? | |
| R3 | Truncation disclosed | Does a truncated result say it was truncated? | |
| R4 | Verbosity control | Can the caller request concise versus detailed output? | |
| R5 | Stable output shape | Does the response match its declared schema on every call? | |

### 12.5 Trust, authority and safety

| # | Input (observable) | Question | Score |
|---|---|---|---|
| T1 | Server-side enforcement | Is authority enforced server-side, not by prompt instruction? | |
| T2 | Scoped credentials | Is the agent's credential scoped to the intended unit of work? | |
| T3 | Programmatic auth | Can a non-human caller obtain a credential without an interactive login? | |
| T4 | Prompt-text review | Are descriptions and error templates under change control as prompt text? | |
| T5 | Output sanitisation | Are tool results sanitised before reaching a model? | |
| T6 | Human control placed deliberately | Is the human oversight point a deliberate design choice, not an assumed prompt? | |

### 12.6 Evidence and upkeep

| # | Input (observable) | Question | Score |
|---|---|---|---|
| C1 | Dated audit exists | Is there an agent audit with a recorded model, version and date? | |
| C2 | Audit is task-realistic | Do the audit tasks reflect real multi-step work, not toy lookups? | |
| C3 | Metrics collected | Are completion, retries, wrong-call rate, tokens and error classes measured? | |
| C4 | Route-arounds instrumented | Is surface adoption measured, so a bypass is visible? | |
| C5 | Agent/human split visible | Can agent traffic be distinguished from human traffic in logs? | |
| C6 | Re-audit on change | Is the audit re-run after any model update or description change? | |

### 12.7 How to use the scorecard, and what it does not claim

- **It is a diagnostic, not a grade.** A cluster of "No" across §12.2 and §12.3 predicts selection and recovery failures; a cluster across §12.5 is a security conversation; a "Not assessed" across §12.6 means you have no idea and should say so.
- **It carries no thresholds by design.** There is no "pass mark," because no source read for this guide establishes that one combination of Yes/No predicts production behaviour. Any vendor offering a threshold is offering an unsourced benchmark (§7.4).
- **It dates.** Record the model, version and date with any completed scorecard; a stale scorecard is no evidence (§7.4).
- **It is a conversation opener with the surface's owner, not a compliance artifact.** The rows are the questions to ask; the answers are the design work.

---

## 13. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is a fictional institution used across this repository as the worked-example persona. Every tool count, token figure, latency, error rate and date in this section is **invented for illustration**, chosen to be plausible, and marked as illustrative wherever it appears. Nothing here is a measurement, a benchmark, or a vendor figure, and none of it should be cited as evidence. Real vendors, specifications and products named elsewhere in the guide are named factually; **no real bank appears as the example.** Cymbal Bank is the only bank persona used.

### 13.1 The situation

Cymbal Bank decides to expose a **payment-initiation interface to third-party agents** — a service that lets an agent acting for an approved corporate customer initiate a payment on that customer's behalf. The commercial reasoning (illustrative): corporate treasury teams increasingly delegate routine payment preparation to agents, and Cymbal wants to be the bank those agents can actually use. The interface is exposed two ways: as a REST API for platform integrations, and as a set of protocol tools for agents attached over MCP.

The initial design, built by mirroring the existing internal payment API, exposes (all figures illustrative): **14 tools** derived from the internal API's endpoint structure — `create_payment`, `get_payment`, `list_payments`, `update_payment`, `approve_payment`, `cancel_payment`, `list_accounts`, `get_account`, `get_balance`, `list_beneficiaries`, `create_beneficiary`, `get_payment_status`, `validate_payment`, and `get_exchange_rate`. The team is proud of "full coverage."

### 13.2 The design pass against §3 and §5

Working §3 (what an agent needs) and §5 (the principles) against the initial design produces a list of defects, each traceable to a rule:

| Defect observed | Rule violated | Source of the rule |
|---|---|---|
| Tools mirror the internal API's endpoints | §5.3 tool is a unit of work, not an endpoint; §5.7 do not mirror the API | Anthropic, 2025-09-11 |
| `update_payment` and `create_payment` overlap in effect | §5.2 one obvious way per intent | Anthropic, 2025-09-11 |
| `create_payment` and `validate_payment` must be called in order, but no order is stated | §5.4 explicit preconditions | Speakeasy, 2025-07-23 |
| `list_payments` returns all payments for an account, unbounded | §3.5 / §5.6 bounded and pageable responses | Anthropic, 2025-09-11 |
| A failed validation returns `400 Bad Request` with no field named | §3.3 errors that instruct | MCP spec 2026-07-28; Anthropic 2025-09-11 |
| Nothing declares whether `create_payment` is safe to repeat | §3.4 idempotency | annotation model: MCP spec; linkage: guide's reasoning |
| `get_payment_status` cannot be learned without reading the integration guide | §3.6 discoverability without browsing | MCP spec 2026-07-28 |

The redesign, following the rules (illustrative):

1. **Collapse to task units (§5.3).** The 14 endpoint-shaped tools become **5 task-shaped tools**: `prepare_payment` (validates, resolves the beneficiary, computes fees, and returns a prepared-but-unsubmitted payment — replacing `create_payment` + `validate_payment` + `get_exchange_rate`), `submit_prepared_payment` (submits a prepared payment, with a caller-supplied idempotency key), `get_payment_status` (replacing `get_payment` + `get_payment_status` + `list_payments` for the status use case), `list_accounts` (unchanged, paginated), and `find_beneficiary` (replacing `list_beneficiaries` with a search over them).
2. **State the workflow where the agent reads it (§5.4).** `prepare_payment`'s description now says a payment must be prepared before it is submitted, that preparation has no money-movement effect, and that a prepared payment expires if not submitted within a stated window (the MCP handle-lifetime pattern, §4.2).
3. **Make the write idempotent (§3.4, §5.5).** `submit_prepared_payment` takes an idempotency key; a repeat with the same key is the same payment. The tool's annotations mark it destructive and not read-only.
4. **Bound every response (§5.6).** `list_accounts` and `find_beneficiary` paginate with a small default; every response is `concise` unless `detailed` is asked for.
5. **Write errors that instruct (§3.3).** A validation failure now names the field, the rule and an example, in the form the MCP specification demonstrates.

### 13.3 The agent audit

Cymbal runs a dated agent audit (the method of §7) on the redesigned surface. Illustrative setup and result (all values invented):

| Input | Illustrative value |
|---|---|
| Audit date | (illustrative) a single day, recorded with model and version |
| Task set | 40 realistic tasks: prepare and submit a payment from a description, resolve an ambiguous beneficiary, handle a rejected payment, query status after submission |
| Models tested | 2 (the result differed between them) |
| Completion (model A) | (illustrative) most tasks completed |
| Completion (model B) | (illustrative) fewer, on the ambiguity tasks |
| Wrong-tool calls | (illustrative) concentrated on beneficiary resolution |
| Retries | (illustrative) concentrated on the first error message draft |
| Route-arounds | (illustrative) a subset of status tasks solved by re-listing accounts instead of calling `get_payment_status` |

The audit's findings, read as §7.3 requires:

- The **wrong-tool and route-around failures disappeared after the description fix**, not after a model change — evidence of an *interface* failure, not a model failure.
- The **retries disappeared after the error text was rewritten** to name the field and rule — again an interface failure.
- The **difference between the two models** on the ambiguity tasks is the honesty caveat of §7.4 in action: the audit is a dated observation on a specific model, and Cymbal records it as such rather than as a certification.

### 13.4 The injection exposure

The design pass lands on §9. The material Cymbal publishes to make the payment tools usable is *instructional text read by an agent*, and in a payment context that is not an abstraction:

- **A beneficiary is a natural-language-influenced field.** A malicious counter-party can name a beneficiary in a way that carries text; `find_beneficiary`'s results and error text are read by the agent as content that may contain instructions. The server-side sanitisation the MCP specification requires **(2026-07-28)** is not optional here.
- **The description is a steering surface for a money-movement tool.** If an attacker can influence what a description says — through a compromised integration, a partner-supplied spec, or a generated description not under review — they can influence what every agent does with the tool. Cymbal treats descriptions as prompt text under change control (§9.3), hash-pins them, and re-approves on any change (the compensating controls documented in [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md); the attack class in [prompt_injection_guide.md](prompt_injection_guide.md)).
- **The error path echoes input.** A validation error that repeats the caller's beneficiary string into a "helpful instruction" is an injection vector; Cymbal constructs error text, never interpolates untrusted data.

The exposure is not a bug to be closed once. It is a standing property of publishing instructional material to a consumer that reads everything as instruction, and it means the security review and the usability review are the same review (§9.1).

### 13.5 The authority question

The design pass lands, finally, on §10. Collapsing `create_payment` + `validate_payment` + `get_exchange_rate` into a single `prepare_payment` was the right *usability* decision (§5.3) — and it changed the *authority* profile of the tool, because one call now spans three internal capabilities that were previously three separately-authorisable operations. The open questions Cymbal cannot close:

- **Who is bound** when a third-party agent submits a payment for Cymbal's corporate customer: the customer, the agent's operator, the agent platform, or some combination? (§10.1)
- **What a limit means** when the agent retries: is the customer's payment limit bound to the *intent* (this payment, once) or to the *attempts*? The idempotency key makes an intent-bound limit *expressible*; it does not answer which the institution intends. (§10.2)
- **Where the human stands.** Cymbal's first design put an approval prompt in the flow. Then it noticed that the whole point of a third-party agent is that no human is watching — so the prompt was nominal, not real. The redesign moved the human to a *pre-authorisation* point (the customer approves the pattern of payments, not each one) and added post-hoc reversal, choosing *where the human stands* deliberately rather than assuming a prompt was a control. (§10.3)
- **What the audit trail must contain.** Cymbal records the full transcript, following Anthropic's own recommended artefact, because the reasoning is not recoverable from the effect alone. **(vendor engineering post, 11 September 2025)**. (§10.5)

Cymbal does not resolve these. It records them as *open architectural questions*, designs the surface so that the institution *can* answer them one way or the other (intent-bound limits are expressible; authority is enforced server-side; the human's position is a parameter of the design), and refuses to pretend the questions are closed. That refusal is itself part of the AX work.

### 13.6 What Cymbal landed on

Work the design pass against §3 and §5, and three things happen in order: the interface becomes usable (task units, stated preconditions, idempotent writes, bounded responses, instructive errors); the audit becomes possible (because there is now a coherent surface to test); and the security and authority questions become *visible* (because a coherent surface is one you can reason about). None of the three is a victory lap. The first is design work that must be redone after every model update, the second is a dated observation that expires (§7.4), and the third is a set of open questions the institution must keep answering.

And underneath all of it is the reason any of this was necessary, the reason a well-governed bank with excellent human-facing operations still had to do a new kind of work: the third-party agent that could not use the original 14-tool surface never sent Cymbal a bug report. It prepared the payment wrong, or listed accounts instead of checking status, or routed around the interface entirely — and Cymbal, without the audit, would never have known. It would have seen green dashboards, no tickets, and a feature that quietly went unused. Because **an agent is a user who never files a bug report.**

---
## 14. The Anti-Patterns

Each anti-pattern is stated as **symptom → cause → guardrail**. The anti-patterns are the failure modes of §8 expressed as things a design team does on purpose.

### 14.1 The endpoint mirror

- **Symptom.** The tool list is one tool per API endpoint; the list is long; agents select poorly and chain calls that should have been one step.
- **Cause.** Generating the tool surface mechanically from an OpenAPI spec. Anthropic: tools that *"merely wrap existing software functionality or API endpoints"* is *"a common error we've observed."* **(vendor engineering post, 11 September 2025)**.
- **Guardrail.** Design tools around units of work (§5.3); collapse sequences that are always called together (OpenAI's `query_location` + `mark_location` example, **(vendor product documentation, checked October 2026)**).

### 14.2 The twin tools

- **Symptom.** Two tools do nearly the same thing under similar names; agents pick either, and both succeed.
- **Cause.** Duplicate capability added rather than a tool replaced. **Cause of the failure to notice:** both calls return 200, so nothing looks broken.
- **Guardrail.** One obvious way per intent (§5.2); descriptions that state a tool's purpose relative to its siblings (§12.2 P6).

### 14.3 The empty error

- **Symptom.** A failure returns a generic code and a generic message; the agent retries the same call or invents a change.
- **Cause.** Errors written for a human developer who will read a log, not for a consumer that will use the text as its next prompt.
- **Guardrail.** Errors that name the field, the rule and an example (§3.3); the MCP specification's protocol-vs-execution error split ordered by recoverability **(2026-07-28)**.

### 14.4 The unbounded dump

- **Symptom.** One tool returns a large collection by default; the task fails after that call.
- **Cause.** A response designed for completeness rather than for context budget. The agent *cannot scroll* (§2.1).
- **Guardrail.** Small defaults, explicit "more available," cursors, verbosity control, and disclosed truncation (§5.6).

### 14.5 The hidden precondition

- **Symptom.** The agent calls a tool in the wrong order and receives an error it cannot interpret.
- **Cause.** An ordering constraint documented only in a tutorial the agent never reads. Speakeasy's billing-address example is the canonical case. **(vendor blog, 2025-07-23)**.
- **Guardrail.** Put preconditions in the description (§5.4), where the agent actually reads them.

### 14.6 The interactive gate

- **Symptom.** The agent abandons the capability entirely; the feature has no agent traffic.
- **Cause.** A capability reachable only through a UI, an interactive login, a captcha, a magic link or a "contact sales" step (§3.7, §4.6). Resend: agents *"are simply not going to 'contact sales' or 'book a demo'."* **(vendor blog, 19 February 2025)**.
- **Guardrail.** Capability parity; programmatic, delegated authentication (§12.5 T3).

### 14.7 The prompt-as-security-control

- **Symptom.** A restriction is documented in a description and trusted to hold.
- **Cause.** Confusing instruction with enforcement. A description is enforced by instruction-following alone.
- **Guardrail.** Enforce authority server-side (§9.3); hash-pin and re-approve descriptions as change-controlled prompt text (compensating controls in [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md)).

### 14.8 The permanent certification

- **Symptom.** A team treats a passed agent audit as a lasting guarantee and stops testing.
- **Cause.** The measurement error of §7.4: an audit is a dated observation on a specific model, and both the model and the surface change.
- **Guardrail.** Date every audit, record model and version, re-run on model update and on any description change (§12.6 C6).

### 14.9 The unused-but-shipped feature

- **Symptom.** A surface built for agents has no agent traffic and no reported problem.
- **Cause.** Silent failure plus measurement blindness (§8.6, §7.5): route-arounds look like successes and discovery failures leave no log.
- **Guardrail.** Instrument surface adoption, not only task success (§12.6 C4); treat absence of expected traffic as the alarm.

### 14.10 The read-the-docs assumption

- **Symptom.** The team is confident the surface is usable "because it's all documented."
- **Cause.** Applying the DX model — read-before-call — to a consumer that sees only what is in front of it at call time (§2.3).
- **Guardrail.** Inline documentation in the schema (§4.4); test with the "intern test" (§12.2 P5).

---

## 15. The Claims Audit — Verified, Flagged, Rejected

| # | Claim | Verdict | Source, quality, date |
|---|---|---|---|
| 1 | The term **AX / "agent experience"** was coined by Mathias Biilmann, CEO of Netlify, in **early 2025** | **Verified as published** | netlify.com/agent-experience ("Netlify coined the term in 2025") — vendor marketing, checked Oct 2026; corroborated by resend.com/blog/agent-experience (19 Feb 2025) and speakeasy.com/blog/agent-experience-introduction |
| 2 | AX is defined by its originator as *"the holistic experience AI agents will have as the user of a product or platform"* | **Verified as published** | Netlify page + quoted by Resend (19 Feb 2025) — vendor marketing |
| 3 | A **competing definition** uses AX for the experience of *humans working with agents* | **Flagged — competing usage, not reconciled** | eleken.co/blog-posts/ax-design (2026) and agentexperience.ax present the human-centred reading; no source reconciles the two | 
| 4 | A **peer-reviewed antecedent term exists**: "agent-computer interface (ACI)" | **Verified — peer-reviewed** | SWE-agent, arXiv:2405.15793, first posted 2024-05-06 (v3 2024-11-11), NeurIPS 2024; identifier resolved at the arXiv API, 2026-10-07 |
| 5 | SWE-agent premise: LM agents *"represent a new category of end users with their own needs and abilities, and would benefit from specially-built interfaces"* | **Verified — peer-reviewed** | arXiv:2405.15793 abstract |
| 6 | Interface design measurably changes agent performance (SWE-agent's ACI) | **Verified as published — peer-reviewed paper's own result** | arXiv:2405.15793; the specific performance deltas are the paper's, not reproduced here |
| 7 | The model vendors **do not use the term "AX"**; they frame the problem as "writing tools for agents" | **Verified as published + negative finding** | Anthropic engineering post (11 Sep 2025) and OpenAI/Google docs read in this pass contain no "AX" usage |
| 8 | Anthropic premise: *"we need to design [tools] for agents"* rather than as functions/APIs for developers | **Verified — vendor engineering post** | anthropic.com/engineering/writing-tools-for-agents, 11 Sep 2025 |
| 9 | Designing tools by *"merely wrapping existing software functionality or API endpoints"* is *"a common error"* | **Verified — vendor engineering post** | Anthropic, 11 Sep 2025 |
| 10 | Consolidate chained operations into task-shaped tools (`schedule_event`, `search_logs`, `get_customer_context`) | **Verified — vendor engineering post** | Anthropic, 11 Sep 2025 |
| 11 | Parameter naming: prefer `user_id` over `user`; describe a tool "to a new hire" | **Verified — vendor engineering post** | Anthropic, 11 Sep 2025 |
| 12 | Claude Code restricts tool responses to **25,000 tokens** by default | **Verified — vendor product default, not a benchmark** | Anthropic, 11 Sep 2025 (stated as the vendor's own default) |
| 13 | Errors should be prompt-engineered to communicate *"specific and actionable improvements, rather than opaque error codes or tracebacks"* | **Verified — vendor engineering post** | Anthropic, 11 Sep 2025 |
| 14 | Namespacing effects *"vary by LLM"*; choose a scheme by your own evaluations | **Verified — vendor engineering post** (also flagged: no independent replication) | Anthropic, 11 Sep 2025 |
| 15 | OpenAI: functions are injected into the system message; definitions count against context and are billed as input tokens | **Verified — vendor product documentation** | platform.openai.com function-calling guide, checked Oct 2026 |
| 16 | OpenAI: *"Aim for fewer than 20 functions available at the start of a turn"* (a soft suggestion); *"use enums… to prevent invalid states"*; the "intern test" | **Verified — vendor product documentation** | platform.openai.com, checked Oct 2026 |
| 17 | Google: *"Tool Selection: Keep active set to 10-20 tools maximum"*; strong typing; descriptive names | **Verified — vendor product documentation** | ai.google.dev/gemini-api/docs/function-calling, page footer "Last updated 2026-09-23 UTC" |
| 18 | MCP: tool definitions need a name, description, valid JSON-Schema input; tools/list supports pagination and caching; deterministic ordering improves prompt cache hit rates; tool names 1–128 chars | **Verified — specification** | modelcontextprotocol.io tools concepts, revision 2026-07-28 |
| 19 | MCP: distinguishes **protocol errors** (models less likely to fix) from **tool execution errors** (actionable; clients SHOULD pass to the model to enable self-correction) | **Verified — specification** | modelcontextprotocol.io, 2026-07-28 |
| 20 | MCP: clients **MUST** treat tool annotations as untrusted unless from a trusted server; servers **MUST** validate inputs, control access, rate-limit and sanitize outputs | **Verified — specification** | modelcontextprotocol.io tools, 2026-07-28 |
| 21 | MCP: stateful-handle guidance — state the handle's retention in the creation tool's description, and make expiry recoverable via an execution error | **Verified — specification** | modelcontextprotocol.io, 2026-07-28 |
| 22 | Speakeasy: OpenAPI operation descriptions sufficient for developers are *"not useful for agents"*; provide agent-specific workflow context; solve dual-audience rendering with `x-speakeasy-mcp` | **Verified as published — vendor blog** | speakeasy.com/blog/agent-experience-introduction, "Last updated on July 23, 2025" |
| 23 | Resend: agents *"should be able to get up and running in milliseconds"*; they *"will pick whatever tool is easiest"*; they will not "contact sales" | **Verified as published — vendor blog** | resend.com/blog/agent-experience, 19 Feb 2025 |
| 24 | ~**63.3%** of agent failures are cognitive rather than tool-call failures | **Verified as a measured finding in one benchmark; flagged as a generalisation** | arXiv:2602.00933 (MCP-Atlas), cited via [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md); single benchmark's taxonomy, not a universal constant |
| 25 | *"An agent is a user who never files a bug report"* is an established formulation in the literature | **Negative finding** | Searched; the exact framing appears as this guide's own thesis. No primary source uses this sentence; it is this guide's synthesis of the attested behaviour (agents do not file reports), not a quotation |
| 26 | There exists an **"AX certification"** standard, or any ISO/W3C/peer-reviewed "agent experience" body of literature | **Negative finding** | No standard, note or peer-reviewed "AX" literature located in this pass; the peer-reviewed term is "agent-computer interface" (row 4) |
| 27 | A vendor claim that a specific **percentage** of agent-tool failures is caused by bad descriptions | **Rejected — untraceable** | Repeated in practitioner marketing without a methodology, dataset or model list; do not cite. Attribute the *direction* (descriptions matter) to Anthropic, 11 Sep 2025, which states it qualitatively |
| 28 | A published, reproducible **end-to-end task-success benchmark** of an AGENT-optimised surface versus a human/developer-optimised one | **Flagged — not found** | No such benchmark located; SWE-agent studies interface design but against its own ACI, not against a UX/DX baseline |
| 29 | The date of the Anthropic post is **11 September 2025** | **Verified** | Page-stated publication date on anthropic.com/engineering/writing-tools-for-agents, read 2026-10-07 |

Every row carries an attributed source with its date and its kind. Rows 1–3, 26 establish the term's status; rows 4–6 give the peer-reviewed antecedent; rows 8–23 are the dated vendor/specification design guidance; rows 24, 28 are the measurement evidence; rows 25, 27 are the honest negatives and the rejected claim.

---

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing Summary

### 16.1 What could not be verified

Recorded as explicit negatives, because a silent omission reads as coverage. **None of these should be asserted elsewhere.**

1. **A canonical, authoritative definition of "agent experience."** The term has an originator (Netlify, 2025) and at least two incompatible definitions in circulation; no standards body, no peer-reviewed source, and no reconciliation of the competing usages was located. The peer-reviewed term is "agent-computer interface" (SWE-agent, 2024). Anyone citing "AX" as a settled discipline is overstating the evidence.
2. **Any published, reproducible measurement that an agent-optimised surface outperforms a developer-optimised surface end-to-end.** SWE-agent shows interface design changes performance against its own ACI; it does not benchmark AX against a UX/DX baseline.
3. **Any independent replication of the vendor guidance** in §5. The rules from Anthropic, OpenAI and Google are their own recommendations, published with their own examples; none is a controlled study, and none was replicated in the sources read.
4. **A per-model, per-version map of "which descriptions work."** Both Anthropic (namespacing "varies by LLM") and the audit caveat (§7.4) imply that surface usability is model-dependent, but no source read provides a cross-model comparison of the *same* surface at the *same* date.
5. **Any regulatory instrument specifically addressing agent tool permissions, delegated authority, or confirmation controls.** Carried forward from [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md), which records a **negative finding**: no NIST control, no ISO/IEC 42001 clause, and no SR 11-7 / OCC / PRA / ECB statement on agentic AI tool permissions was located. §10 is therefore stated as open questions.
6. **The exact publication date of the Netlify "Introducing AX" blog post itself.** The Netlify landing page and the Resend post (19 Feb 2025) attest the *term's* early-2025 origin, but the primary blog post's own date was not read at source in this pass; the Resend reference ("a couple of weeks ago" as of 19 Feb 2025) is the closest dating evidence.
7. **Any quantitative threshold at which a surface becomes "agent-usable."** No source read establishes one; §12 carries none by design (§7.4).
8. **A public set of "agent-experience" design standards analogous to WCAG.** The WCAG analogy is instructive but no such standard exists for AX in the sources checked.

**Research-tooling note affecting reproducibility.** The term "agent experience" was found in **exactly 1 of 651 markdown files** in this repository (grep across `/home/ubuntu/research`, October 2026) — [agent_scaffolding_guide.md](agent_scaffolding_guide.md), where it appears only as incidental prose ("embedded agent experiences"), not as a discipline. That is consistent with this guide's §1.2 finding that AX is emerging commercial terminology, not established repository coverage. The primary research for this guide relied on direct extraction of named primary URLs (Anthropic, OpenAI, Google, the MCP documentation, Netlify, Resend, Speakeasy) and on resolving the SWE-agent identifier at the arXiv API over HTTPS; the `web_search` pass returned results on this occasion but, per the standing hazard, direct URL extraction was used as the load-bearing method, and every fact above traces to a page that was actually read.

### 16.2 Glossary

**Affordance (agent)** — What a surface makes it look like is possible, inferred by an agent from names, schemas and descriptions rather than from visual cues; a human affordance such as a hover state conveys nothing to an agent.

**Agent audit** — A dated experiment in which an agent attempts a task on a surface with no human help; the surface's usability is measured by completion, retries, wrong-call rate and tokens, with the model and version recorded. A dated observation, not a certification.

**Agent-computer interface (ACI)** — The peer-reviewed name for the discipline's subject; introduced with SWE-agent (arXiv:2405.15793, 2024, NeurIPS 2024).

**Agent consumer** — A model-driven process that calls a surface to complete a task on someone's behalf; the "user" AX designs for.

**Agent experience (AX)** — The practice of designing surfaces for an agent consumer. A framing term, coined by Netlify in 2025; not a settled discipline, and used with at least two incompatible definitions.

**Description (tool/parameter)** — The natural-language text attached to a tool or parameter that an agent reads as prompt content; it is instruction, not metadata, and it is a security surface.

**Discovery path** — The route by which an agent learns a surface exists and what it does, without a human browsing on its behalf.

**DX (developer experience)** — Designing for a human developer who reads documentation before calling and debugs against a local copy.

**Idempotency key** — A caller-supplied value that makes a repeated write collapse to a single effect; the mechanism that lets an intent-bound limit survive a retrying caller.

**Progressive disclosure (AX framing)** — Showing an agent only the surface it needs at the moment it needs it, revealing the rest on demand; mechanism owned by [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md).

**Silent failure** — A failure that produces no signal to the author; the defining failure mode of AX, because an agent neither complains nor files an issue.

**Surface** — Any published artefact an agent can perceive and act on: an API, a protocol tool, a CLI entry point, a documentation page, an error response, an authentication path, a structured-output schema.

**Tool** — A callable unit exposed to an agent that performs a unit of work, distinct from an API endpoint that exposes a resource operation.

**Tool poisoning** — Steering model behaviour by injecting content through tool descriptions or outputs; a documented attack class, owned by [prompt_injection_guide.md](prompt_injection_guide.md).

**Unit of work** — The smallest task-meaningful step an agent should take in one call; it defines what a tool should be and what authority the tool carries.

**UX (user experience)** — Designing for a human end user who perceives layout, explores, and complains when blocked.

### 16.3 Cross-references

Within `technology/ai_llm/`:

- [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) — owns context and disclosure; §6 here is only the AX framing.
- [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) and [mcp_discovery_guide.md](mcp_discovery_guide.md) — own the protocol and server discovery.
- [context_engineering_guide.md](context_engineering_guide.md), [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md), [agent_scaffolding_guide.md](agent_scaffolding_guide.md), [agent_skills_vs_multi_agent_systems_guide.md](agent_skills_vs_multi_agent_systems_guide.md) — own the internals inverted in §11.
- [agent_loop_implementation_guide.md](agent_loop_implementation_guide.md) — owns the loop.
- [prompt_injection_guide.md](prompt_injection_guide.md) and [ai_red_teaming_guide.md](ai_red_teaming_guide.md) — own the security classes in §9.
- [ai_governance_framework_guide.md](ai_governance_framework_guide.md) — owns the governance frame for §10.
- [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md) — own the general failure and feedback taxonomy for §8.

Elsewhere in the repository:

- [../documentation_in_agentic_coding_age.md](../documentation_in_agentic_coding_age.md) — owns documentation for coding agents.
- [../api_governance_guide.md](../api_governance_guide.md) — owns API design and governance for human consumers.
- [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) — owns the banking compliance frame.

Primary sources read for this guide: Anthropic, *Writing effective tools for agents — with agents* (11 September 2025); OpenAI, *Function calling* product documentation (checked October 2026); Google, *Function calling with the Gemini API* (footer 2026-09-23); the Model Context Protocol *Tools* documentation and specification (revision 2026-07-28); Netlify, *Agent Experience*; Resend, *What is AX* (19 February 2025); Speakeasy, *Designing agent experience* (2025-07-23); SWE-agent, arXiv:2405.15793 (2024).

### 16.4 Closing summary

Agent experience is a young, contested label on a real and specific problem: designing the surfaces an autonomous consumer calls. Its academic antecedent is the agent-computer interface (SWE-agent, 2024); its commercial label was coined by Netlify in 2025; its practitioners are, in date order, the model vendors and the protocol specification, whose guidance this guide assembles and attributes rather than invents. What makes it a discipline and not a rebrand of DX is one property: the failure mode is silent. A human complains and a developer files an issue; an agent does neither. It produces a plausible wrong answer, invents a parameter, retries until its context runs out, or routes around the feature entirely — and the author, seeing green dashboards and no tickets, concludes the surface is fine.

The remedies this guide states are ordinary: name the intent, offer one obvious way, make a tool a unit of work, state the preconditions, make writes idempotent, bound the responses, write errors that instruct, and publish nothing an attacker can steer with. The hard parts are the ones without sourced answers: an audit is a dated observation, not a certification; the material you publish is the injection vector; and the unit of work decides what can be authorised, in a domain whose liability questions remain open. Design the surface so the institution can answer those questions deliberately — and test it, on a dated model, with no human help — because the consumer you are designing for will use whatever is in front of it and then tell no one how it went. It will not file a bug report, because an agent is a user who never files a bug report.
