# AI Chatbot Architecture: The Conversation Is the State

> **Author:** Jack Liu Shurui
> **Role:** Solution Architect
> **Repository:** github.com/jackliusr/research
> **Series:** AI/LLM Engineering Guides — Agent & Platform Architecture track
> **Date:** 23 September 2026
>
> **Companion guides.** This guide owns exactly one layer of the stack: **the architecture of the conversation** — the reference layering of a conversational system, the conversation-state problem, the dialogue-management patterns, the channel layer, the voice pipeline, the human handoff, and the chatbot-versus-agent distinction. It defers and does not re-derive: **conversational retrieval** (query rewriting, question condensation, conversational RAG) → [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md) §9 "Query Transformation for Conversational RAG" and §13 "Conversational RAG — Deep Dive"; **the regulatory treatment of client-facing assistants** → [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3; **context and memory engineering mechanics** (window management, summarisation, memory stores) → [context_engineering_guide.md](context_engineering_guide.md); **the agent end of the spectrum** (goal-directed autonomy, planning, tool loops) → [autonomous_agents_guide.md](autonomous_agents_guide.md), [agent_scaffolding_guide.md](agent_scaffolding_guide.md), [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §5 and §7, [agentops_guide.md](agentops_guide.md) §6, [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §5, [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), [agents_at_scale_guide.md](agents_at_scale_guide.md) (concurrency, economics, volume failure), [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md), [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md); **guardrails and injection** → [llm_guard_models_guide.md](llm_guard_models_guide.md), [prompt_injection_guide.md](prompt_injection_guide.md); **the gateway and egress layer** → [../enterprise_ai_gateway_guide.md](../enterprise_ai_gateway_guide.md); **evaluation and testing machinery** → [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md), [rag/deepeval_guide.md](rag/deepeval_guide.md) §ConversationalTestCase; **the speech and real-time audio shelf** → [ai_speech_tutor_research.md](ai_speech_tutor_research.md), [../livekit_alternatives_guide.md](../livekit_alternatives_guide.md), [../accent_detection_guide.md](../accent_detection_guide.md).
>
> **How to read it.** §1–§3 set the thesis, draw the chatbot-versus-agent line, and lay out the reference layering. §4 is the section the guide exists for — the conversation state. §5–§6 are the two things you do about it: manage the window and choose a dialogue-management pattern. §7–§11 are the surrounding layers the conversation layer actually touches: retrieval, channels, voice, handoff, safety and disclosure. §12–§13 are testing and the regulated case. §14 is a fictional worked example; §15 is the anti-pattern catalogue; §16 carries the claims audit, the glossary and the closing. Every figure is labelled. `VERIFIED` means I read it on the primary source on the date shown. `ILLUSTRATIVE` means it is a constructed arithmetic input, not a published fact. `⚠` marks an uncertain claim; `❌` marks one I could not verify at all.

---

### Table of Contents

1. [The Overview, the Boundary and the Decoder](#1-the-overview-the-boundary-and-the-decoder)
2. [Chatbot Versus Agent: The Architectural Distinction](#2-chatbot-versus-agent-the-architectural-distinction)
3. [The Reference Layering](#3-the-reference-layering)
4. [The Conversation State](#4-the-conversation-state)
5. [The Context Window as a Constrained Resource](#5-the-context-window-as-a-constrained-resource)
6. [The Dialogue-Management Patterns](#6-the-dialogue-management-patterns)
7. [The Retrieval and Grounding Layer, in Brief](#7-the-retrieval-and-grounding-layer-in-brief)
8. [The Channel Layer](#8-the-channel-layer)
9. [The Voice Dimension](#9-the-voice-dimension)
10. [The Human Handoff](#10-the-human-handoff)
11. [Safety, Grounding and Disclosure, in Brief](#11-safety-grounding-and-disclosure-in-brief)
12. [Testing a Conversation](#12-testing-a-conversation)
13. [The Regulated-Deployment Constraints](#13-the-regulated-deployment-constraints)
14. [Worked Example: Cymbal Bank Redesigns Its Service Chatbot](#14-worked-example-cymbal-bank-redesigns-its-service-chatbot)
15. [The Anti-Patterns](#15-the-anti-patterns)
16. [The Claims Audit, What Could Not Be Verified, Glossary, Cross-References and Closing](#16-the-claims-audit-what-could-not-be-verified-glossary-cross-references-and-closing)

---

## 1. The Overview, the Boundary and the Decoder

**Thesis, in one line: an agent's state is its task, but a chatbot's state is the conversation — and the whole architecture exists to manage a state that must outlive a stateless model and a finite context window.**

That sentence is the whole guide. Everything below it is a consequence of three facts that are individually mundane and jointly awkward:

1. **The model is stateless.** Each inference call receives exactly what you send it and remembers nothing. There is no "the bot" that persists between calls — there is a prompt, a completion, and a gap.
2. **The context window is finite and expensive.** Whatever continuity exists must be carried *inside* the request, and the request has a token budget, a latency cost and a price per token. Continuity is not free; it is metered.
3. **The channel can drop.** The user closes the tab, switches from web to the mobile app, gets a phone call, replies to the same thread three days later. A conversation is not a process with a lifetime; it is an interaction with gaps, and the gaps are where architectures fail.

An agent has these same facts to deal with, but they resolve differently, because an agent's unit of meaning is a *task* and a task can be re-derived from its goal. If an agent loses context, it can re-plan. A chatbot cannot re-plan its way back to a conversation: if the bot forgets that the customer already said which account they meant, there is no goal to recover from — there is a customer who has to repeat themselves, which reads as incompetence. That asymmetry is why the conversation, not the task, is the state that the architecture must protect.

**What a chatbot is architecturally FOR.** The purpose of a conversational system is narrower than the purpose of an agent, and naming it precisely prevents a whole class of design mistakes. A conversational system exists to:

- **Maintain an exchange** across turns, with continuity that the model itself does not provide;
- **Resolve language to structured intent** so that something downstream — a rule, a lookup, an API — can act on it;
- **Advance a state machine** (explicitly or implicitly) from an unresolved request toward a resolved one;
- **Ground the answer** in something other than the model's parametric memory;
- **Hand control to a human** when the state says the system cannot finish;
- **Produce a record** of what was said, which is often the actual deliverable in a regulated setting.

Note what is *not* on that list: open-ended autonomy, multi-step planning toward a self-chosen goal, tool discovery, or self-directed iteration. Those are agent properties. A chatbot may *use* an agent internally — and increasingly does — but the product it delivers is the exchange, not the task.

**The decoder.** Vocabulary in this domain is loose, and loose vocabulary is what produces the wrong architecture — "context", "memory" and "state" get used interchangeably by people who then build three half-versions of the same thing. These are the definitions used throughout this guide.

| Term | What it means here | Why it matters architecturally |
|---|---|---|
| **Turn** | One user input and the system's response to it. The atomic unit of the exchange. | It is the unit of latency, of cost, and of testing. Everything per-turn is metered per-turn. |
| **Session** | A bounded period of continuous interaction, usually with a session identifier and a timeout. The unit the *channel* thinks in. | Sessions expire, which is exactly why they cannot be the only place conversation state lives. |
| **Conversation state** | The durable, structured record of what has been established, what remains unresolved, and what the system has committed to. Outlives any single session. | This is the guide's subject. It is the thing the model does not have and the channel will not keep. |
| **Slot** | A named field the system needs filled to complete a task — `account_id`, `amount`, `date_from`. | Slots are how conversation becomes structured; unfilled slots are how the system knows it is not done. |
| **Entity** | A value extracted from the user's language and bound to a slot or an action — a date, a currency amount, an account reference. | Entity resolution is the bridge from free text to something a deterministic system can act on. |
| **Intent** | A labelled categorisation of what the user wants — `check_balance`, `report_lost_card`, `dispute_transaction`. | The intent taxonomy is a maintained artefact, not a model output. Unowned taxonomies rot (see §15). |
| **Dialogue manager** | The component that decides what happens next: which slot to ask for, which flow to enter, whether to escalate. | It is the state machine's controller. Whether it is a rules engine or a prompt, it is the same seat. |
| **Deterministic flow** | A scripted path with explicit states and transitions, where the same input always produces the same next step. | Required wherever the outcome has legal or financial consequence. See §13. |
| **Handoff** | The transfer of the conversation to a human agent, carrying context. | It is an architectural path, not an exception handler (see §10). |
| **Channel** | The surface the conversation happens on — web widget, messaging platform, email, telephony, in-product. | The channel constrains what the conversation layer may assume: length, rich content, presence, interruption (see §8). |
| **Barge-in** | The user speaking while the system is still speaking. | Voice-only. It forces the system to be interruptible, which constrains the whole voice pipeline (see §9). |
| **Grounding** | Tying an answer to an identified source rather than the model's parametric recall. | The difference between an answer and a plausible answer. Owned in depth by the RAG shelf (see §7). |
| **Disclosure** | Telling the user they are interacting with an AI system. | In several jurisdictions this is a legal obligation, not a UX preference (see §11). |

**Boundary declaration.** This guide is the **conversation layer** and nothing else. It does not own the retrieval machinery — question condensation, query transformation, hybrid search, chunking, reranking — which belongs to [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md) §9 and §13, and to the RAG shelf generally; §7 here states only what the *conversation* requires from retrieval and then stops. It does not own the **agent** end of the spectrum — planning, tool loops, scaffolding, multi-agent topology — which belongs to [autonomous_agents_guide.md](autonomous_agents_guide.md), [agent_scaffolding_guide.md](agent_scaffolding_guide.md), [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) and [hierarchical_multi_agent_frameworks_guide.md](hierarchical_multi_agent_frameworks_guide.md); §2 here draws the line between the two and then cross-refers. It does not own **context and memory engineering mechanics** — how to summarise, what to store, how the window is assembled — which belongs to [context_engineering_guide.md](context_engineering_guide.md); §5 here states the conversational *requirements* those mechanics serve. It does not own **observability, evaluation, failure taxonomy or cost control** — [agentops_guide.md](agentops_guide.md) §6, [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7, [agents_at_scale_guide.md](agents_at_scale_guide.md). It does not own **guardrails or prompt injection** — [llm_guard_models_guide.md](llm_guard_models_guide.md), [prompt_injection_guide.md](prompt_injection_guide.md); §11 states only where they sit in the conversational path. It does not own **the regulatory treatment of client-facing assistants** — [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3; §13 states only what the *architecture* must then do. The test applied to every paragraph in this guide: *would this sentence be equally true of a document pipeline or a batch job?* If yes, it belongs somewhere else, and I have tried to leave it there.

---

## 2. Chatbot Versus Agent: The Architectural Distinction

The distinction is made once here, and used throughout without re-arguing it.

**The agent.** An agent receives a goal, decomposes it, chooses tools, executes steps, observes results, and iterates until the goal is met or it gives up. Its primary product is **task completion**. Its state is **the task**: the plan, the step history, the artefacts produced, the open sub-goals. It is judged by whether the task got done. Crucially, an agent can re-plan — which means a *degraded* agent state is recoverable, at a cost. The agent guides in this repository own that end: [autonomous_agents_guide.md](autonomous_agents_guide.md) for the architectures, [agent_scaffolding_guide.md](agent_scaffolding_guide.md) for the scaffolding, [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §5 and §7 for deployment and cost control, [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) for how they break.

**The conversational system.** A chatbot receives an *utterance*, not a goal. It resolves that utterance into something actionable, advances a conversational state, and responds. Its primary product is **the exchange itself** — in a service context, often the correct outcome is not "the task was performed" but "the user was heard, informed, and either served or handed to someone who will serve them". Its state is **the conversation**: the transcript, the resolved slots, the unresolved intents, the flags.

The distinction has a sharp, testable edge: **an agent without a goal is broken; a chatbot without a goal is normal.** A chatbot that answers "what are your branch hours?" and then waits is functioning correctly. An agent that has nothing to do is a mis-invocation. That is why the two have different success criteria, different state models and different failure modes — and why "just make it an agent" is not a design improvement to a support chatbot, it is a change of product.

**The capability spectrum.** Chatbots are not one thing. Five rungs, each with a distinct architecture:

| Rung | What it does | State model implied | Success criterion | Characteristic failure | Testability |
|---|---|---|---|---|---|
| **1. Menu-driven bot** | Presents options; maps selection to a branch. (`Press 1 for…`, or a button tree.) | None beyond "which node am I on". Pure position in a tree. | Correct routing. | Dead ends; no path for anything unlisted. | Fully deterministic; trivially testable. |
| **2. Intent-and-entity bot** | Classifies intent, extracts entities, fills slots, runs a dialogue policy over slots. | Slot state + intent distribution + dialogue policy state. | Correct intent classification, correct slot fill, correct flow completion. | Misclassification; unresolvable ambiguity; the "none of the intents" bucket. | Unit-testable per intent; the hard part is the utterance corpus, not the logic. |
| **3. Retrieval-grounded assistant** | Answers free-form questions by retrieving and grounding, with no task state. | Conversation context (for query condensation) + retrieved evidence, not slots. | Grounded, cited, non-hallucinated answers. | Retrieval miss; fluent answer from an unretrieved source. | Requires labelled query-answer sets; conversational retrieval needs multi-turn test cases. |
| **4. Tool-calling assistant** | Chooses and calls APIs to fulfil a request, within a conversation. | Slots + tool-call history + conversation state + pending confirmations. | Correct tool selection, correct arguments, safe side effects. | Wrong tool, wrong argument, duplicate side effect. | Needs trace-level assertions on tool calls, not just output text. |
| **5. Agentic assistant** | Plans across multiple steps and tools, possibly over long horizons, inside a conversation. | Full agent state *plus* conversation state — two state machines that must stay coherent. | Task completion under supervision. | Everything in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), now amplified by the fact a human is watching turn by turn. | Hardest of all: non-deterministic path, non-deterministic output, human-visible latency. |

**The architectural consequence of climbing a rung.** Four things change, and they change together:

- **The state model** grows from "position in a tree" (rung 1) to "slots and intent" (rung 2) to "conversation context" (rung 3) to "slots + tool history + pending side effects" (rung 4) to "agent state AND conversation state, reconciled" (rung 5). Rung 5 is the one that catches teams out: they add agency to a chatbot and discover they now have two independent state machines that can disagree — the agent believes it has booked the appointment, the conversation has not told the user, and the next turn is incoherent from the user's side.
- **The success criterion** shifts from a categorical ("did it route correctly?") to a graded one ("was the answer grounded and useful?") to a compound one ("did it complete the task *and* keep the user informed?"). Graded criteria need evaluators; see [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md).
- **The failure mode** moves from *dead ends* to *misclassification* to *hallucination* to *unsafe side effects* to *compounding multi-step error*. Each rung's failure is less visible and more expensive than the last.
- **The testability** degrades monotonically. Rung 1 is a unit test. Rung 5 is a simulation with a judge model and a human in the loop. Climbing a rung without spending on evaluation is the most common way to end up with a system nobody can certify.

**Which rung to choose.** Not the highest one. The rung should be the lowest that meets the requirement, because each rung adds state, latency, cost and untestability. A bank that needs "check my balance and tell me my last five transactions" does not need rung 5; it needs rung 4 with a hard deterministic boundary (§13). Choosing rung 5 for a problem rung 2 solves is not ambition, it is an unforced reliability and governance cost.

---

## 3. The Reference Layering

The layering below is the one used for the rest of this guide. Each layer gets: **what it owns**, **the interface it presents upward**, and **the repo guide that owns its deep detail**. Only the conversation layer, the channel layer and the voice pipeline are developed here; the rest are named and handed off.

| # | Layer | Owns | Interface upward | Deep detail owned by |
|---|---|---|---|---|
| 1 | **Channel** | The surface: transport, message format, session/correlation IDs, threading, presence, attachment limits. | A stream of normalised inbound messages and a way to send outbound ones. | **This guide, §8** |
| 2 | **Session & gateway** | Lifetime of a session, authentication, rate limiting, tenancy, egress control, correlation. | An authenticated, identified session handle. | [../enterprise_ai_gateway_guide.md](../enterprise_ai_gateway_guide.md) |
| 3 | **Conversation / dialogue** | The state machine: slots, intents, flows, turn policy, escalation decisions. | "Given this state and this utterance, here is the next action." | **This guide, §4–§6, §10** |
| 4 | **Retrieval & grounding** | Source selection, condensation of the conversational query, ranking, provenance. | Cited evidence passages for a given (possibly context-dependent) query. | [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md) §9, §13; [rag/advanced_rag_techniques_guide.md](rag/advanced_rag_techniques_guide.md) |
| 5 | **Model** | Inference: prompt assembly, sampling, tool-call schema, routing between models. | Text, structured output or tool calls. | [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md); [context_engineering_guide.md](context_engineering_guide.md) |
| 6 | **Tool** | Side-effecting operations: payments, bookings, account lookups. | Idempotent, audited, permissioned operations. | [autonomous_agents_guide.md](autonomous_agents_guide.md); [../durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) |
| 7 | **Memory** | What persists beyond a session: preferences, prior resolutions, relationship facts. | Read/write access to durable, user-scoped facts. | [context_engineering_guide.md](context_engineering_guide.md) |
| 8 | **Safety** | Guardrails on input and output, injection defence, PII handling, policy enforcement. | Allow / block / redact decisions on every boundary crossing. | [llm_guard_models_guide.md](llm_guard_models_guide.md); [prompt_injection_guide.md](prompt_injection_guide.md) |
| 9 | **Analytics** | Traces, transcripts, containment, escalation, cost attribution, feedback. | Dashboards, alerts, evaluation datasets. | [agentops_guide.md](agentops_guide.md) §3–§6; [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md) |

Three properties of this layering deserve to be stated explicitly, because they are where naive designs break:

**(i) The conversation layer must not assume the model.** The dialogue layer's interface is "state + utterance → next action". Whether that action is produced by a rules engine, a classifier, or a prompt is an implementation choice *inside* the layer. Architectures that let the model call the dialogue decisions directly, with no intermediary, lose the ability to enforce determinism where the law requires it (§13) and the ability to test at all (§12).

**(ii) The conversation layer must not assume the channel, but it must know it.** The conversation layer receives normalised messages, so it should not parse WhatsApp payloads. But it *must* know the channel's class, because the channel determines whether the conversation can be interrupted (§9), how long a response may be, and whether a thread is a single conversation or a mailbox (§8). A design that treats "the channel" as an invisible transport loses the asynchronous-channel case entirely.

**(iii) The layers are not tiers.** They are responsibilities that can be co-located in one process or spread across five services. A single-team deployment may implement all nine in one Python application; a bank will separate at least the safety, gateway and tool layers for governance reasons. The layering is a *responsibility* map, not a deployment topology.

---

## 4. The Conversation State

This is the section the guide exists for.

### 4.1 What the state contains, and why each element exists

The mistake is to treat the state as "the message history". The message history is *one* element, and it is the one you can most afford to compress. The elements that actually break the system when lost are the structured ones.

| Element | What it is | Why it exists | What breaks when it is lost |
|---|---|---|---|
| **Transcript** | The ordered turns: utterances, responses, timestamps, channel. | The record; the evidence; the thing a human reads on handoff; often the legal artefact. | Handoff has nothing to hand over; audit has no record. |
| **Resolved slots** | Filled fields with values, provenance and confirmation status: `account_id = "…" (confirmed)`, `amount = 250 (unconfirmed)`. | Turns language into something a deterministic system can act on. Unconfirmed values are not facts yet. | The bot re-asks for what it already has, or acts on an unconfirmed value. |
| **Resolved entities** | Extracted values not yet bound to a slot — a date, a reference number, a named product. | The user volunteers information ahead of being asked; holding it shortens the interaction. | The bot asks for information the user already gave. The single most visible chatbot failure. |
| **Unresolved intents** | What the user asked for that has not been satisfied — including a changed intent, a parked one, and an ambiguous one. | A conversation can contain more than one request, and users interleave them. | A request silently disappears; the user believes it was handled. |
| **Behavioural flags** | `escalation_requested`, `vulnerability_signal`, `disclosure_served`, `consent_captured`, `identity_verified`. | Some behaviours must happen at most once, or exactly once, regardless of what the model says. | The bot re-announces it is an AI; or worse, proceeds without a verification step that must precede an action. |
| **Commitments** | What the system has told the user it will do — "I'll send a replacement card within 5 working days". | A commitment made in turn 3 is a constraint on turn 20. | The bot contradicts itself, which is the failure users trust least. |
| **Provenance** | Which sources were used, which policy was applied, which bot version answered. | Reproducibility, evaluation, dispute resolution. | Nothing can be explained after the fact. |

The last two — commitments and provenance — are the elements most often omitted, and they are exactly the ones a regulated environment demands and a customer notices. Provenance is owned in depth by [agentops_guide.md](agentops_guide.md) §3–§6; the *requirement* is stated here because it is a state-design decision, not an observability afterthought.

### 4.2 The state's lifetime at three scales

State has three natural lifetimes, and conflating them is the source of most state bugs.

| Scale | Lifetime | Contents | Where it must live | Failure if confused with another scale |
|---|---|---|---|---|
| **Turn** | Seconds | The pending input, the in-flight intent classification, the tool call in progress. | In-process. Disposable. | Persisting per-turn scratch state pollutes the session record. |
| **Session** | Minutes to hours | Active slots, the flow position, the current task, the unexpired transcript. | Session store keyed by session ID, with a timeout. | Treating the session as the conversation loses everything on timeout (see §8). |
| **Relationship** | Weeks to years | Preferences, verified identity, prior resolutions, standing consent, known products. | Durable, user-scoped profile store, subject to retention policy. | Treating the relationship as a session means every interaction starts from zero; users read this as an amnesiac bank. |

The architectural rule that follows: **slot and flow state belong to the relationship or the session, never only to the model's context.** Anything that lives solely in the prompt dies with the request.

### 4.3 Where the state can live, and what each placement costs

| Placement | How it works | Buys | Costs |
|---|---|---|---|
| **Client** (widget, browser, app) | State serialised to the client and returned each turn. | Cheap; no server store; trivially scales. | Lost the moment the tab closes or the user switches device; tamperable; unusable for email/async channels. |
| **Server** (session/conversation store) | State keyed by conversation ID, fetched and updated per turn. | Survives channel change, device change, timeouts; auditable; the only placement compatible with asynchronous channels. | A store to operate, secure, retain and delete; a consistency question when two turns arrive at once. |
| **Model context** (in the prompt) | State *is* the prompt; nothing stored elsewhere. | No infrastructure; maximum flexibility. | Bounded by the window, expensive per turn, non-auditable, and gone when the request ends. |
| **Split** (structured state server-side, working context in the prompt) | Authoritative state in a store; the prompt is *rendered from it* each turn. | The right answer: durable, auditable, and the prompt stays small. | Requires a renderer and a discipline: the prompt is a view, never the source of truth. |

The split is the recommended default, and the reason is worth saying plainly: **if the prompt is the source of truth, the conversation has no memory — it has a scrolling transcript.** Rendering the prompt from structured state is what allows a turn to be answered with a *small* prompt (§5) that nonetheless knows everything the conversation has established.

### 4.4 The explicit state machine versus the implicit one

A conversation always has a state machine. The only question is whether it was designed or whether it emerged.

- The **explicit** state machine names its states and transitions; it is inspectable, testable, and can be reviewed by a control function. "We are in `AWAITING_OTP`; the only transitions are to `VERIFIED` or `ABANDONED`." It is the only form that a regulated process can accept (§13).
- The **implicit** state machine exists in the prompt and in the model's behaviour. It has states — "the bot currently believes it is confirming a transfer" — but nobody declared them, so nobody can enumerate the reachable set, and no test can assert on them.

The implicit machine is not inherently wrong; it is inherently *unreviewable*. Where the consequence of an incorrect transition is a wrong balance quoted, it is wrong. The practical rule: make the states explicit wherever the transition has a financial, legal or safety consequence; allow the machine to stay implicit only in the conversational surface between those points, and record what the surface did (§4.6).

### 4.5 The hard constraint

The conversation state must satisfy three requirements simultaneously, and each one rules out the obvious shortcut:

1. **Survive a stateless model.** The model contributes nothing between calls, so continuity must be reconstructed on every turn from something the model cannot forget — a store.
2. **Survive a finite context window.** The state must be *structured and compressible*, because the window cannot hold an unbounded transcript (§5). A state that can only be represented as prose is a state that will be truncated.
3. **Survive a channel that may drop.** The state must be durable across timeouts, reconnections and channel switches, and keyed by something more stable than a socket (§8).

The design that satisfies all three is: **a durable, user-scoped conversation record — structured, versioned, and rendered into a right-sized prompt on every turn.** Everything else in this guide is either how to build that, or what to do when it says the conversation cannot continue.

### 4.6 A concrete state sketch

Illustrative shape — the field set is the point, not the syntax.

```json
{
  "conversation_id": "c_8812f",          // durable, channel-independent
  "customer_ref": "u_4471",              // relationship-scoped, stable across sessions
  "channel_sessions": [                  // one conversation, many channel sessions (§8)
    {"channel": "web_widget", "session_id": "s_aa1", "opened": "2026-09-23T09:14Z", "closed": "2026-09-23T09:31Z"},
    {"channel": "voice_ivr",  "session_id": "s_bb7", "opened": "2026-09-23T10:02Z", "closed": null}
  ],
  "state": "AWAITING_OTP",               // explicit, enumerable, reviewable
  "intent": {"primary": "replace_lost_card", "confidence": 0.93, "changed_from": null},
  "slots": {
    "card_last4":      {"value": "4417", "source": "user_turn_2", "confirmed": true},
    "delivery_address":{"value": null,   "source": null,         "confirmed": false},
    "reason":          {"value": "lost", "source": "user_turn_2", "confirmed": true}
  },
  "unresolved": [{"intent": "dispute_transaction", "raised_turn": 4, "status": "parked"}],
  "entities": [{"type": "date", "value": "2026-09-20", "turn": 4, "bound_to": null}],
  "flags": {"identity_verified": true, "disclosure_served": true,
            "consent_captured": false, "escalation_requested": false},
  "commitments": [{"turn": 6, "text": "replacement card in 5 working days"}],
  "provenance": {"bot_version": "3.4.1", "policy": "card_replacement_v7",
                 "sources": ["kb/cards/replacement#3"]}
}
```

Two things to notice. First, `state` is a **named state**, not a description — because a named state is what a test can assert on and a control function can review. Second, `confirmed` is per-slot, because the difference between "the user mentioned 4417" and "the user confirmed 4417" is the difference between a look-up and a fraudulent action.

---

## 5. The Context Window as a Constrained Resource

### 5.1 What each turn adds

Every turn adds to the prompt: the user's utterance; the system's response; any retrieved evidence; any tool call and its result; and the rendered state. The evidence and tool results are the quiet problem — a single retrieval-grounded turn can add an order of magnitude more tokens than the exchange itself. A prompt that grows monotonically is a prompt that will eventually be truncated, and truncation is silent.

### 5.2 The budget question

The budget is not "how big is the window". It is: **window minus system instructions minus the rendered state minus reserved output minus retrieved evidence = what remains for history.** Stated that way, it becomes obvious that history is the *last* claimant, not the first, and that the strategies below are about allocating a residual. The mechanics of assembling that budget belong to [context_engineering_guide.md](context_engineering_guide.md); the cost curve it feeds belongs to [agents_at_scale_guide.md](agents_at_scale_guide.md) §6 and [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7.

### 5.3 The window-management strategies

| Strategy | Mechanism | Failure mode — how it shows up to the user | When it is right |
|---|---|---|---|
| **Truncation** | Drop the oldest turns when the budget is exceeded. | The bot asks for something established early: "which account?" after being told in turn 2. | Short exchanges; or when structured state independently holds the early facts. |
| **Summarisation** | Replace a span of turns with a model-written summary. | The summary drops the *decision-relevant* detail — the amount, the date, the exception — and keeps the pleasantries. | Long, low-consequence stretches with no commitments made. |
| **Rolling summary** | Maintain one running summary, updated each turn or every N turns. | Summary drift: small errors accumulate and become unrecoverable, and the original wording is gone. | Very long relationships where only gist matters. |
| **Selective retention** | Keep decision-relevant turns verbatim (commitments, confirmations, corrections) and summarise the rest. | Requires knowing what is decision-relevant, which means the state model must label turns at write time. | The recommended default. It costs you a turn-labelling step and buys you a bot that remembers the things that matter. |

The honest statement: **every window-management strategy has a failure mode that presents as the bot forgetting something it was told, and the user cannot distinguish a truncation bug from a comprehension failure.** This is why the recommended default is selective retention *plus* structured state (§4), so that the facts the conversation established are never only in the summarised text.

### 5.4 The performance consequence

A long conversation is a latency and cost problem, not only a memory problem. Prompt tokens are processed on every turn, so a conversation that renders a 12,000-token prompt on turn 20 pays for those 12,000 tokens on *every subsequent turn* — the cost of a conversation is superlinear in its length, not linear. Two consequences follow, and both are architectural: rendering a *small* prompt from structured state (§4.3) is a cost strategy, not merely a tidiness strategy; and long conversations need an explicit degradation path — an offer to continue in a fresh context, a handoff, or a summarise-and-restart — before they hit a hard limit at the worst moment. The arithmetic is [agents_at_scale_guide.md](agents_at_scale_guide.md) §6; the mechanics are [context_engineering_guide.md](context_engineering_guide.md).

---

## 6. The Dialogue-Management Patterns

There are three families and one hybrid. Each implies a different state model, fails differently, tests differently and costs differently to maintain.

### 6.1 The deterministic flow / state machine

Explicit states, explicit transitions, scripted prompts. Heritage: telephony IVR menus, and the regulated processes where the *sequence* is the compliance requirement — verify identity, state the disclosure, confirm the amount, record consent. In those processes, the ordering is not a UX choice; it is the control.

| Dimension | Assessment |
|---|---|
| **State model** | Named states + slot store. Fully enumerable. The reachable-state set can be drawn and reviewed. |
| **Failure mode** | Brittleness at the conversational surface: the user says something outside the grammar and hits a dead end. Also combinatorial maintenance as flows multiply. |
| **Testability** | The best of any pattern. Every transition is a unit test; the state space is finite and small. |
| **Maintenance cost** | High in *content*, low in *risk*: every new flow is new authored work, but changes are reviewable diffs. |

### 6.2 The intent-and-entity pipeline (classifier plus slot filling)

An NLU stage classifies intent and extracts entities; a dialogue policy decides the next action from the slots. This was the dominant architecture of the 2016–2021 chatbot wave and it still exists inside commercial platforms.

| Dimension | Assessment |
|---|---|
| **State model** | Intent distribution + slot store + dialogue policy state. Interpretable but larger than a flow: intent × slot combinations multiply. |
| **Failure mode** | Misclassification, and the long tail — the utterances that fall into the fallback intent. The fallback bucket is where the user experience lives and where it is usually thinnest. |
| **Testability** | Per-intent unit tests plus a labelled utterance corpus; the *utterance corpus* is the real maintenance burden, not the logic. |
| **Maintenance cost** | Continuous: the taxonomy drifts from what users actually say, and every taxonomy change invalidates part of the corpus. See §15 on the unowned taxonomy. |

### 6.3 The LLM-planned approach

The model receives the conversation and decides what to say and what to do next, usually with tools available. This buys flexibility and conversational quality; it costs determinism.

| Dimension | Assessment |
|---|---|
| **State model** | Implicit. The state is what is in the prompt plus whatever the model chooses to write into a tool call. It cannot be enumerated. |
| **Failure mode** | Everything in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md), plus a conversational-specific one: confidently advancing a flow the user did not agree to. |
| **Testability** | Statistical only. Evaluators, judges, sampled review — see [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md). No assertion can cover the path space. |
| **Maintenance cost** | Prompt and policy curation; the cost moves from authoring flows to continuously evaluating behaviour, which is a *standing* cost rather than a project cost. |

### 6.4 The hybrid — deterministic control, model surface

The design position this guide recommends, and the one a regulated deployment usually converges on: **the model handles language; the state machine handles consequence.** The model resolves the user's utterance into structured input (or a proposed transition); a deterministic controller validates it against the declared state machine and decides the transition; the model renders the response. The model never decides *whether* identity has been verified, *whether* a transaction may proceed, or *which* policy applies. It never has the authority to skip a step.

This is a structural boundary, not a degree of prompting. Concretely, the model's output is treated as an *untrusted proposal*: a schema-validated structured object that the controller either accepts, rejects, or asks the user to clarify. The pattern costs you a schema, a validator and a state store — and it is what makes the conversation reviewable, testable and defensible. Stated as a design position rather than an opinion: **in a regulated process, the steps with legal consequence should not depend on a model's judgement**, because a judgement cannot be tested exhaustively, cannot be version-locked as a rule, and cannot be shown to a control function.

### 6.5 Framework status — verified at source

This is the field where tools die quietly. Each entry was read at its own source on 23 September 2026. **Dormant, archived or absorbed tools are flagged and must not be chosen for new work.**

| Framework / platform | Status | Evidence (source, read 23 Sep 2026) | Marker |
|---|---|---|---|
| **Rasa Open Source** (`RasaHQ/rasa`) | **Live, community-maintained.** Public repo, default branch `3.6.x`, Apache-2.0, ~21.3k stars. Rasa's own docs state the open-source project "is maintained by the community"; the commercial direction is Rasa Pro. | GitHub `RasaHQ/rasa`; rasa.com/docs/rasa/ | `VERIFIED` |
| **Microsoft Bot Framework SDK** | **ARCHIVED — do not adopt.** The repository "was archived by the owner on Jan 5, 2026. It is now read-only." | GitHub `microsoft/botframework-sdk` | `VERIFIED` ❌ dormant |
| **Microsoft LUIS** | **RETIRED 1 October 2025** (retirement announced on Microsoft Learn; new resources blocked from 1 April 2023). Successor: Azure AI Language. | Microsoft Learn, Bot Framework availability FAQ (page last updated 02 Jan 2025) | `VERIFIED` ❌ retired |
| **Microsoft QnA Maker** | **RETIRED 31 March 2025** (new resources blocked from 1 October 2022). Successor: Azure AI Language question-answering. | Microsoft Learn, same FAQ | `VERIFIED` ❌ retired |
| **Google Dialogflow CX** | **Live, but the console is deprecated.** The Conversational Agents console "includes the features from both deprecated Dialogflow CX console and Vertex AI Agent Builder console". Documentation last updated 22 Sep 2026. | Google Cloud docs (Dialogflow CX → Conversational Agents console overview) | `VERIFIED` (console superseded) |
| **Amazon Lex V2** | **Live.** Current AWS developer guide under `docs.aws.amazon.com/lexv2`; SDK integration documented. | AWS docs, "What is Amazon Lex V2?" | `VERIFIED` |
| **IBM watsonx Assistant** | **Repositioned.** `ibm.com/products/watsonx-assistant` now presents as **IBM watsonx Orchestrate**, an agent-management platform; the assistant product is no longer branded as a standalone chatbot product on that page. Treat as absorbed into an agent-orchestration platform. | ibm.com (URL redirect observed on the page read) | `⚠` absorbed |
| **Botpress** | **Live, repositioned.** Repo description now "The open-source hub to build & deploy GPT/LLM Agents"; MIT; ~14.9k stars. | GitHub `botpress/botpress` | `VERIFIED` |

Two conclusions follow for anyone choosing a dialogue framework in 2026. First, **the classic NLU-and-dialogue framework category has largely been absorbed into agent platforms or retired** — the two most widely deployed enterprise SDKs in the 2018–2022 wave (Bot Framework, LUIS) are archived and retired respectively, and Dialogflow CX's console has been folded into Google's agent-builder surface. Second, the surviving open-source option in this category is maintained by its community with a commercial product alongside it, which is a materially different support commitment from a vendor SLA. Both facts argue for the hybrid pattern of §6.4 backed by *your own* state store, rather than betting the conversation state on a platform's session model.

---

## 7. The Retrieval and Grounding Layer, in Brief

One short section, deliberately. This is not the retrieval guide.

**What the conversation layer needs from retrieval:** for a given turn, a set of passages that (a) answer the question, (b) carry provenance, and (c) are not the model's parametric memory. Nothing more. The conversation layer does not care how they were found.

**Why conversational retrieval is different.** A single-shot question is self-contained; a conversational one is not. "And what about the joint account?" contains no retrievable content on its own — it needs the *conversation* to be turned into a query. That transformation (question condensation, coreference resolution, query rewriting into a standalone form) is the defining feature of conversational retrieval, and it belongs to the RAG shelf: see [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md) §9 "Query Transformation for Conversational RAG" and §13 "Conversational RAG — Deep Dive". The architectural point for *this* guide is only that the state (§4) must retain enough to reconstruct the referent — which is another reason unresolved entities and the last resolved topic are first-class state elements, not transcript noise. Techniques beyond query rewriting are covered in [rag/advanced_rag_techniques_guide.md](rag/advanced_rag_techniques_guide.md).

**The citation and grounding requirement.** Grounding is a conversational obligation, not just a retrieval quality metric: an ungrounded answer in a conversation is not a slightly worse answer, it is a statement the system will be held to for the rest of the session (see the `commitments` element, §4.1). Citations must therefore survive to the conversational surface where the channel allows it, and where the channel does not (voice, SMS), the grounding must exist in the record even if it cannot be shown.

**The hand-off.** Query transformation, chunking, hybrid search, reranking and their evaluation are owned by the RAG shelf: [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md), [rag/advanced_rag_techniques_guide.md](rag/advanced_rag_techniques_guide.md), [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md), [rag/rag_optimization_techniques_guide.md](rag/rag_optimization_techniques_guide.md). This section stops here.

---

## 8. The Channel Layer

### 8.1 The channel classes and what each forces

The channel is not a transport detail. It constrains the conversation layer's design, because it determines what a "turn" is and what a "conversation" is.

| Channel class | Rich responses | Practical response length | Presence | Interruption | What a "thread" is | What it forces on the state design |
|---|---|---|---|---|---|---|
| **Web / app widget** | Yes (buttons, cards, images) | Medium; scrollable, so bounded by patience not transport | Yes (live typing indicators) | Partial | One continuous session, usually closed with the tab | Session-scoped state is tempting and wrong; the widget *will* be closed mid-flow. Requires server-side state. |
| **Messaging platform** | Constrained (platform-defined message types; template rules) | Short; long text is split and reads badly | No | No | A long-lived thread with gaps of minutes to days | Async by nature; the "session" is a fiction. See §8.3. |
| **Email** | No | Long-form tolerated, but multipart replies are common | No | No | A subject/message-id chain per person | Threading and quoting are the hard part: users reply inside quoted text, strip context, or fork threads. State must be reconstructed from message metadata, not assumed. |
| **Phone / voice** | No | Very short; the user cannot re-read | Yes (silence is information) | **Yes — barge-in is mandatory** | A single call | Real-time turn-taking and an interruption model are required; see §9. |
| **In-product assistant** | Yes, and it can act on the app's own state | Short, contextual | Yes | No | The user's current task in the product | The richest channel: it knows the screen the user is on. It also raises the expectation that the assistant *acts* rather than *explains* — which pushes it toward the agent rungs of §2. |

**Richness and interruption are the two axes that matter most.** Richness determines whether the conversation layer may return a structured object (buttons, a card, a form) or must serialise everything to prose — which changes the response-generation path, not just its formatting. Interruption determines whether the system must be able to *stop* mid-turn, which is a property only voice has (§9) and which propagates all the way down to the model call.

### 8.2 Session and correlation identifiers across channels

Every channel gives you a session identifier of its own kind, and none of them is a conversation identifier.

| Identifier | Provided by | Lifetime | Primary purpose |
|---|---|---|---|
| `channel_session_id` | The channel (call ID, socket ID, thread ID) | The channel's lifetime | Transport routing and delivery |
| `conversation_id` | **You** | Until the conversation is resolved or abandoned | The unit the state (§4) hangs from |
| `customer_ref` | **You** (from authentication or profile) | The relationship | Continuity across devices, channels and channels-to-come |
| `message_id` / `correlation_id` | Channel + your gateway | One message | Deduplication, idempotency, trace correlation |

The design rule: **never let the channel's session identifier be the conversation's identity.** If it is, a dropped call ends the conversation, an email reply starts a new one, and a user who switches from the widget to the phone number is a stranger. The durable key is the customer reference plus a conversation identifier that *you* mint and the channel only references.

### 8.3 The asynchronous channel problem

This is the part that most designs get wrong, and it deserves to be stated as a hard architectural requirement.

On a messaging platform or email, a "conversation" can span **hours or days** between turns. The synchronous session model — session opens, turns arrive, session times out and everything is discarded — does not describe this at all. When you put an async channel in front of a synchronous session model, three things break: the state expires mid-conversation; the "awaiting user input" state has no runtime that can wait for it; and re-engagement (the business messaging the user, or the user returning after two days) has no entry point.

What the messaging platforms themselves make explicit is exactly this: on the WhatsApp Business Platform, free-form **service messages** may only be sent inside a **24-hour customer service window** that opens when the user messages or calls, and that resets on each new inbound message; once the window closes, only pre-approved **template messages** may be sent, and opt-in is required before templates. Undelivered messages are retried up to a default time-to-live of **30 days** (10 minutes for authentication templates) before being dropped. `VERIFIED` — Meta for Developers, "Service messages", doc updated 21 May 2026, read 23 September 2026.

That platform behaviour has direct architectural consequences, and they are requirements rather than features:

- **State must be durable and long-lived**, keyed by the customer reference, because the session will not survive the gap (§4.2, §4.5). A conversation store with a 30-minute TTL is not an async design.
- **The dialogue must be resumable from a cold start**, which means the entry point must reconstruct "where were we" from the store — not from a live process.
- **Re-engagement is a first-class flow with its own policy**, and it is *regulated by the channel*: outside the customer-service window you may only send pre-approved templates. So the re-engagement design is partly a message-template-design exercise, and the templates are a change-controlled artefact.
- **Response timing becomes part of the UX.** In a chat thread, replying instantly and replying in four hours are different products; in email, a same-day reply reads as attentive. The conversation layer must therefore be able to *defer* a turn — which means the turn, not the process, is the unit of scheduling.

A design that treats async channels as "chat with a longer timeout" will pass testing in a synchronous harness and fail on the first holiday weekend.

---

## 9. The Voice Dimension

### 9.1 The pipeline as a chain of stages, each with its own failure

| Stage | Input → output | Characteristic failure |
|---|---|---|
| **Speech recognition (ASR)** | Audio → text | Misheard words, and — worse — *confidently* misheard words. Accents, background noise, domain vocabulary. See [../accent_detection_guide.md](../accent_detection_guide.md). |
| **Understanding (NLU)** | Text → intent + entities | Compounded by ASR: a misheard digit becomes a wrong account number, extracted with high confidence. |
| **Dialogue** | State + intent → next action | The state problem of §4, now with no scrollback for the user to check. |
| **Generation** | Action + state → response text | Verbosity: text-length answers are intolerable when spoken. |
| **Synthesis (TTS)** | Text → audio | Mispronunciation of names, digits and product codes; wrong prosody turning a confirmation into a question. |

### 9.2 The latency budget — why voice is harder than text

In text, latency is a tolerance. In voice, latency is a **conversational turn-taking failure**: the user is holding the floor open, silence reads as a breakdown, and the natural human response is to repeat themselves or start over — which corrupts the input to the next ASR stage. Voice is therefore the one channel where the system must respond *within the pace of conversation*, or the conversation degrades.

An illustrative end-to-end budget makes the constraint concrete. **All figures below are `ILLUSTRATIVE`** — constructed arithmetic inputs for reasoning about the split, not published facts about any vendor or deployment:

| Stage | Illustrative budget (ms) | Note |
|---|---|---|
| ASR finalisation + endpointing | 150 | Detecting that the user has finished speaking. |
| NLU / dialogue decision | 100 | Often rules, rarely a large model call, in the deterministic path. |
| Response generation | 300 | The dominant term if a large model is generating the whole response. |
| TTS first audio | 250 | Time to first audio, not total synthesis. |
| Network / transport | 100 | Round trip, media plus control. |
| **Total to first audio** | **~900** | `ILLUSTRATIVE` |

Anyone who has designed a text agent for a multi-second "thinking" indicator and then reused that budget unchanged for voice has built the anti-pattern in §15 — *the voice bot designed to a text latency budget*. The architectural consequences: generation must be **streamed and overlapped** with synthesis (first clause spoken while the rest is generated); the dialogue decision should be kept off the slow path wherever it is deterministic (§6.4); and a filler ("let me check that") is a legitimate latency-management tool, not a UX nicety.

### 9.3 Barge-in and endpointing

**Barge-in** is the user speaking while the system is still speaking. It is required on voice and exists nowhere else, and it forces three properties: the pipeline must be **interruptible** mid-generation and mid-synthesis (so a non-streaming architecture is disqualified); the system must decide *what* the interruption means (stop and listen, or take it as an answer); and the interrupted partial output must be handled consistently in the state — was the sentence completed as a commitment or not? That last point connects directly to `commitments` in §4.1.

**Endpointing** is the inverse problem: deciding the user has finished. Too eager and you interrupt a thinking pause (a "false barge-in", indistinguishable to the user from rudeness); too patient and every turn carries dead air, which is the latency failure of §9.2 in a different guise. Endpointing is a policy with no universally correct setting, which is why it belongs in the state and configuration model rather than hard-coded.

### 9.4 Error amplification through the chain

The voice pipeline is a serial chain, so errors do not stay local: a misheard word becomes a wrong intent becomes a wrong transition becomes a wrong answer becomes a confidently spoken wrong answer. If each stage is, illustratively, 95% accurate, the chain's joint accuracy is the product — roughly 77% over five stages, `ILLUSTRATIVE` arithmetic. Two design responses follow: **put verification at the points where errors become expensive** (read back the account number, the amount, the date — a cheap deterministic confirmation of a fallible chain), and **do not add stages**. Every added model call in a serial voice chain multiplies success probabilities downward and adds latency, which is why the deterministic path of §6.1 earns its keep in voice more than anywhere else.

### 9.5 The voice-dialogue standards — status verified at source

The repository has no prior mention of these standards, so this is fresh verification. Each entry was read at the W3C or IETF source on 23 September 2026. **Stagnant standards are marked and must not be presented as live.**

| Standard | What it is | Current status | Marker |
|---|---|---|---|
| **VoiceXML 2.0** | XML language for voice dialogs (speech, DTMF, telephony, mixed initiative). | **W3C Recommendation, March 2004.** Stable, unchanged. | `VERIFIED` (legacy, stable) |
| **VoiceXML 2.1** | Minor update to 2.0; the latest *Recommendation* in the family. | **W3C Recommendation** (2.1). This is the family's terminal Recommendation. | `VERIFIED` |
| **VoiceXML 3.0** | Proposed modular successor, built on SCXML-based resources. | **STAGNANT — never reached Recommendation.** Latest published version is the *Eighth Public Working Draft of 16 December 2010*; the document itself describes itself as "very much a work in progress". | `VERIFIED` ❌ stagnant — do not design to 3.0 |
| **W3C Voice Browser Working Group** | The group that produced the family. | **OFFICIALLY CLOSED on 2015-10-12.** No successor group continues the work. This is the decisive fact: the standards are frozen because the group is. | `VERIFIED` |
| **SRGS 1.0** (Speech Recognition Grammar Specification) | Grammar format for specifying what a recogniser should listen for. | **W3C Recommendation, 16 March 2004.** Stable; still implemented by classical speech stacks, but largely bypassed by end-to-end neural ASR, which does not require hand-authored grammars for open-domain speech. | `VERIFIED` (stable; superseded in practice where ASR is neural) |
| **SISR 1.0** (Semantic Interpretation for Speech Recognition) | Annotations on grammar rules for extracting semantic results. | **W3C Recommendation** — part of the frozen Voice Browser family. | `VERIFIED` |
| **PLS 1.0** (Pronunciation Lexicon Specification) | Phonetic information for recognition and synthesis. | **W3C Recommendation** — part of the frozen Voice Browser family; still useful as a lexicon format for TTS pronunciation control. | `VERIFIED` |
| **SSML 1.1** | Markup for controlling synthetic speech (pronunciation, prosody, rate). | **W3C Recommendation, 7 September 2010.** The most *durable* of the family: it is vendor-implemented in modern TTS APIs even though the spec is unchanged. | `VERIFIED` (live in practice) |
| **SCXML 1.0** | A generic state-machine notation and execution environment. | **W3C Recommendation, 1 September 2015** — the last Recommendation published by the Voice Browser WG before it closed. Genuinely reusable *outside* voice, as a formalism for the explicit state machine of §4.4. | `VERIFIED` |
| **CCXML 1.0** | Call-control markup: call screening, transfer, whispering. | **W3C Recommendation, July 2011** — part of the frozen family. | `VERIFIED` |
| **MRCPv2** | Media Resource Control Protocol v2 — client control of speech synthesisers, recognisers and verifiers over SIP/SDP. | **IETF Standards Track, RFC 6787, November 2012.** A stable IETF RFC; the mechanism by which a voice platform drives network speech resources. | `VERIFIED` |

**The honest reading.** The classical voice-dialogue standards family is **frozen, not current**. Its working group closed in October 2015; its proposed successor (VoiceXML 3.0) died at Working Draft; the Recommendations that exist — VoiceXML 2.1, SRGS 1.0, SISR 1.0, PLS 1.0, CCXML 1.0, SSML 1.1, SCXML 1.0 — are stable documents that modern conversational voice systems largely do not build on, because modern voice pipelines are neural ASR → LLM → neural TTS rather than grammar-and-form VoiceXML applications. Three practical implications: (i) **do not design a 2026 voice agent on VoiceXML 3.0** — it is a 2010 working draft; (ii) **SSML and PLS remain useful** as the narrow, vendor-implemented interfaces for pronunciation and prosody control; and (iii) **SCXML is the one genuinely forward-looking artefact** of the family, because a state-machine notation is exactly what §4.4 and §6.1 ask for, independent of voice. The repository's speech-adjacent material is [ai_speech_tutor_research.md](ai_speech_tutor_research.md), [../livekit_alternatives_guide.md](../livekit_alternatives_guide.md) (real-time transport and voice-agent platform options) and [../accent_detection_guide.md](../accent_detection_guide.md) (the ASR error surface).

---

## 10. The Human Handoff

Escalation is not an error path. It is an architectural path — the one that decides whether the whole system is a service or a maze.

### 10.1 The trigger taxonomy

| Trigger | Signal | Why it must be explicit |
|---|---|---|
| **Low confidence** | Intent or slot confidence below a threshold; repeated clarification requests. | Confidence is a number, and the threshold is a policy decision that must be owned, not left to the model's phrasing. |
| **Explicit request** | "I want to speak to a person." | Non-negotiable, and it must be detectable in every phrasing, including the angry ones. The cost of failing to honour it exceeds every other consideration. |
| **Repeated failure** | The same intent attempted N times without resolution, or confusion detected twice. | Without a counter, a bot will loop politely forever. The counter lives in the state (§4), not in the model. |
| **Regulated / high-consequence intent** | The conversation has entered a flow whose outcome has legal or financial consequence and lies outside the bot's authority. | A declaration in the state machine, not an inference. |
| **Affective signal** | Distress, vulnerability, threats of harm, or language indicating a complaint or a formal dispute. | Both a service obligation and, in some regimes, a regulatory one. Treat as a hard trigger, and treat false positives as cheap. |

Two things are worth stating flatly. First, **the triggers belong in the state machine** — they are conditions evaluated on every turn, and their firing is recorded as a flag (`escalation_requested`, §4.1) so that it cannot be lost. Second, **a low-confidence trigger must not be the only trigger**, because a confidently wrong bot is the worst case and it never fires the confidence trigger.

### 10.2 The context-transfer requirement

This is the requirement most often missed and the one customers notice immediately. A handoff that lands the human agent on a blank screen is not a handoff; it is a restart with a new participant.

What must actually cross the boundary:

| Item | Why the human needs it |
|---|---|
| **The transcript** | To avoid asking the customer to repeat everything. This is the single largest driver of escalation satisfaction. |
| **The resolved state** | Slots and their confirmation status, so the agent does not re-verify what is already verified — or, worse, act on something unconfirmed. |
| **The unresolved intents** | Including the ones the bot parked, so nothing is silently dropped. |
| **The reason for escalation** | Which trigger fired, and when. "The customer asked three times for a person" is actionable; "escalated" is not. |
| **What the bot already told the customer** | Critically the `commitments` element. If the bot said five working days, the agent must not say ten. |
| **The provenance** | Which sources and which bot version produced the statements, so a disputed claim can be traced. |

The architectural consequence: **the handoff interface is the conversation-state record, not a summary of it.** Handing over a model-generated summary of the conversation re-introduces the summarisation failure of §5.3 at exactly the moment accuracy matters most. Hand over the structured state and render the transcript alongside it.

### 10.3 Queueing and the capacity consequence

At volume, the handoff becomes a queueing problem, and this is where chatbot programmes most often fail on paper before they fail in production. The containment rate determines the arrival rate at the human queue; the arrival rate and the handling time determine the wait; and the wait is the service level. Two consequences:

- **A handoff design that assumes infinite human capacity is not a handoff design.** If the bot's escalation rate is 25% and the bot handles a volume the human team could never have handled directly, the human queue is now larger than it was before the bot existed. The escalation rate is therefore a *capacity* input and must be measured alongside containment.
- **The bot changes the shape of the work, not just the volume.** Escalated conversations are, by construction, the harder ones — the bot already absorbed the easy ones. So the agents receiving handoffs get a *harder* mix at the same headcount, which is a workforce-planning fact, not a technical one.

The concurrency, queueing and economics machinery is owned by [agents_at_scale_guide.md](agents_at_scale_guide.md) (§3 concurrency, §10 capacity planning with an honest model) and the cost-control dimension by [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §7. This section owns only the *handoff interface*.

### 10.4 The reverse handoff and the resumption question

The reverse handoff — a human returning the conversation to the bot — is the case teams forget, and it is where the state model is tested hardest. Three questions must have answers before launch:

1. **Who owns the conversation while the human holds it?** If the bot keeps processing inbound turns, it will contradict the human. The state needs a `HUMAN_ACTIVE` state that disables bot responses rather than a config flag that someone remembers to set.
2. **What did the human do that the bot does not know?** An agent who performs an action outside the bot's systems leaves the bot's state stale. Resumption requires either a synchronisation step or an explicit rule that the bot does not resume that conversation.
3. **Is resumption even correct?** Sometimes the honest answer is that the conversation ends in the human lane. Resumption should be a designed flow, not a default.

### 10.5 The honest operating-model point

Everything above assumes a human team exists, is staffed, and has capacity. Where that assumption is false, no amount of escalation architecture creates service — it creates a queue with better logging. The operating model (staffing, hours, skills, the escalation rate the business is willing to fund) is the real constraint, and it belongs in the design document next to the state machine.

---

## 11. Safety, Grounding and Disclosure, in Brief

One short section, with named hand-offs. Each of these is a guide of its own elsewhere in this shelf.

**Where the guardrails sit in the conversational path.** Guardrails are a layer (§3, row 8) that the conversation crosses twice per turn: once on the **inbound** utterance — before it reaches the dialogue manager and before it can influence a transition — and once on the **outbound** response, before it reaches the channel. The inbound position matters more than teams expect, because in a conversational system the *user's turn is untrusted input that can attempt to change the state machine*. Injection defence and guard-model design are owned by [prompt_injection_guide.md](prompt_injection_guide.md) and [llm_guard_models_guide.md](llm_guard_models_guide.md); the architectural requirement stated here is only that the conversation layer must never treat a user utterance as an instruction to a controller, and that a guardrail block must be a modelled state (a response the conversation state records), not an out-of-band exception that loses the turn.

**The PII question in a transcript.** A conversation transcript is, almost by construction, dense with personal data — and unlike a form submission it contains *unstructured* personal data that the user volunteered without being asked. Three architectural consequences: the state must be able to redact at rest and not merely at the source; retention must be a policy attached to the record, because in a regulated setting the transcript is a *record* (§13.2); and the durable relationship memory (§4.2) must not silently accumulate PII that the transcript's retention policy has already deleted. The regulatory treatment is owned by [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3.

**The grounding requirement.** An answer in a conversation becomes a statement the system is held to for the rest of the session, which is why grounding is a safety property and not only a quality one (§7). Ungrounded conversational answers are also the input to the worst escalation class: a customer acting on something the bot invented.

**The disclosure obligation — verified at source.** Unlike most of this guide, this is not a design preference. Two instruments were verified on 23 September 2026:

| Instrument | What it requires | Status and date | Marker |
|---|---|---|---|
| **EU AI Act, Article 50(1)** (Regulation (EU) 2024/1689) | Providers must design AI systems intended to interact directly with natural persons so that those persons **are informed they are interacting with an AI system**, unless this is obvious to a reasonably well-informed, observant and circumspect person. The European Commission's guidelines clarify that this covers **chatbots, AI agents and avatars**, define four cumulative criteria (an AI system; a genuine two-way exchange; direct interaction; with natural persons), require notification **from the start of the first interaction**, in a clear and distinguishable manner and in accordance with accessibility requirements, and state that the "obvious" exception is to be interpreted **restrictively**. | **Applies from 2 August 2026.** A limited grace period applies only to AI systems placed on the market before 2 August 2026, and only for the *marking* obligation of Article 50(2), until 2 December 2026. Fines up to **€15 million or 3% of total worldwide annual turnover**. Commission FAQ last updated 24 July 2026. | `VERIFIED` |
| **California Business & Professions Code §17941** (the "Bolstering Online Transparency" Act, SB 1001, 2018) | Unlawful to use a bot to communicate or interact with a person in California **online**, with intent to mislead about its artificial identity, for the purpose of knowingly deceiving about the content in order to incentivise a purchase or sale or influence a vote. A person "shall not be liable under this section if the person discloses that it is a bot"; the disclosure must be **clear, conspicuous, and reasonably designed** to inform. | Added by Stats. 2018, Ch. 892 (SB 1001); **effective 1 January 2019, operative 1 July 2019.** In force. | `VERIFIED` |

Three architectural consequences follow directly, and they are all state-design consequences:

1. **Disclosure is a state flag, not a message.** `disclosure_served` must be recorded so the system can prove it disclosed, and so the disclosure is served exactly once at the start of the first interaction — not on every turn (which trains users to ignore it) and not never.
2. **The "obvious" exception is a legal judgement, not a UX one.** For a bank's own-branded service assistant, assuming the interaction is obvious to every reasonably circumspect customer is not a defensible default given the guideline's restrictive reading. Disclose.
3. **Disclosure is channel-specific.** A voice bot must say it; a messaging bot must say it in the first message, which for an outbound message means the only permitted vehicle is an approved template (§8.3) — so the disclosure wording must be baked into a change-controlled template.

---

## 12. Testing a Conversation

Kept short; the evaluation machinery is elsewhere.

**Why multi-turn is harder than single-shot.** A single-shot call has one input and one output: you assert on the pair. A conversation has a *state space*, an *ordering*, and a property that breaks naive testing: **a good turn can follow a bad one, and a bad turn can follow a good one.** The final response of a 12-turn conversation can be perfect while turn 5 destroyed the state, and a final-output assertion will not see it. Three further difficulties: the same conversation can be correct via different paths (so path assertions overconstrain); the state is internal, so output-only assertions cannot localise a failure; and the regression surface is the *combination* of prompt and flow, not either alone.

**What a conversational test case actually asserts.** Four distinct things, and a suite should do all four:

| Assertion class | Example | Where it lives |
|---|---|---|
| **Turn-level output** | "Given this state and utterance, the response asks for the missing delivery address." | Deterministic assertions in code — the strongest and cheapest. |
| **State transition** | "After this utterance, `state == AWAITING_OTP` and `slots.card_last4.confirmed == true`." | Assertions on the state store. This is the class most teams skip, and it is the one that catches the silent failures. |
| **Conversation-level quality** | "The conversation resolved the user's goal, stayed on topic, did not give regulated advice." | LLM-judge metrics over a multi-turn test case. |
| **Regression on a golden corpus** | "This recorded conversation's transcript and final state are unchanged (or changed in the intended way)." | A golden set re-run on every prompt, model or flow change. |

The repository's existing conversational-testing artefact is DeepEval's **`ConversationalTestCase`** — a multi-turn test case (conversation history plus final output) used by conversation and agent metrics such as conversation relevancy, task completion, role adherence, and multi-turn safety metrics. It is documented in this repository at [rag/deepeval_guide.md](rag/deepeval_guide.md) §2.4 "Conversation and Agent Metrics" and §4.2 "The Test Cases"; use it rather than inventing a multi-turn harness. Broader evaluation methodology is owned by [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md).

**The regression question for prompts and flows together.** Prompts and flows are two artefacts that jointly determine behaviour, and they change on different cadences — prompts weekly, flows quarterly. A regression suite must therefore re-run the *same* golden conversations against both artefacts and be able to attribute a failure to one of them. The practical mechanism is the state assertion above: an output change with an unchanged state points at the prompt; a state change points at the flow.

---

## 13. The Regulated-Deployment Constraints

What a bank adds, cross-referenced rather than re-derived. The full regulatory treatment of client-facing assistants is [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3; what follows is only the architecture implications.

### 13.1 The steps that must stay deterministic, and why

The design position, stated in §6.4, has a regulatory form: **in a regulated process, the steps with legal consequence should not depend on a model's judgement.** The reason is not distrust of models; it is that a control environment requires properties a model-generated judgement cannot provide:

- **Exhaustive testability.** A rule can be tested over its input space. A judgement cannot.
- **Version-locked evidence.** A reviewer must be able to say "policy v7 was in force and it required X". "The model was instructed to require X" is not the same artefact.
- **Explainability to a third party.** A regulator, an auditor or a court asks *why*. A transition table answers that question; a sampling distribution does not.
- **Non-drift.** A rule does not change when the provider updates a model.

Which steps, concretely: **identity verification and authentication; consent capture; anything that discloses regulated information or moves money; anything that creates a legal commitment; and anything whose outcome is a regulated decision** (a credit decision, a complaint outcome, a fee reversal). In the hybrid pattern these become states in the explicit machine (§4.4), with the model permitted only to route the user *into* them and to phrase what happens *after*.

### 13.2 The disclosure and record-keeping obligations

Disclosure is verified in §11. The record-keeping consequence is the part architects under-plan, and it follows from one sentence: **a transcript is a record.**

| Implication | What it forces |
|---|---|
| **Retention** | The transcript needs a retention schedule with a defined period, and the schedule must survive the conversation store's own lifecycle. Deleting conversation state on a TTL that is shorter than the retention obligation is a records failure, not a cost saving. |
| **Retrievability** | A record that cannot be *produced* on request does not satisfy a record-keeping obligation. This means the transcript must be indexed by customer and by date, and retrievable independently of whether the conversation is still "live". |
| **Completeness** | Escalated conversations, abandoned conversations and bot versions must all be recorded — including which bot version answered and which policy was applied (`provenance`, §4.1). A record of the answer without the version is not reconstructable. |
| **Immutability** | A record that can be edited after the fact is weak evidence. The conversation record should be append-only, with corrections expressed as new entries. |
| **Redaction and retention conflict** | PII must be redactable (§11) *and* the record must be complete. Resolve this by retention tiering and by redacting the copy, not the source, and by stating which is authoritative. |

### 13.3 The model-risk and change-control question

When the conversational surface is model-driven, every model change is a behaviour change, and behaviour change in a regulated process is a *change* in the change-control sense. The architectural requirements that follow:

- **Pin the model version** for regulated flows, so that behaviour is reproducible, and treat a version change as a change requiring re-approval rather than a routine upgrade.
- **Make prompt changes reviewable artefacts.** Versioned, diffable, tied to a test run. A prompt edited in a console by an operator is an uncontrolled change, regardless of the operator's intent.
- **Keep the deterministic core outside the model's change surface**, so that a model upgrade cannot alter a control. This is the strongest argument for the hybrid: it bounds the blast radius of change control to the conversational surface.
- **Evaluation as evidence.** The regression suite (§12) is not only an engineering tool; it is the change-control evidence that a prompt change did not alter a regulated behaviour. Keep the results.

The model-risk framing is developed in the agent cluster — [agentops_guide.md](agentops_guide.md) §6 for monitoring and guardrails, [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) for deployment discipline, [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §5 for the non-functional requirements — and the regulatory treatment in [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3.

### 13.4 The honest statement

**In a regulated process the binding constraint is usually the control environment, not the conversational quality.** A bank can ship a bot that scores well on every conversational metric and still fail its own change-control, record-keeping and disclosure obligations — and the second failure is the one that stops the programme. Conversely, a modest bot whose flows are explicit, whose records are complete and whose disclosures are served is a bot that can be approved. Architects who treat the control artefacts as paperwork to be produced after the design will discover that they *are* the design.

---

## 14. Worked Example: Cymbal Bank Redesigns Its Service Chatbot

**This section is entirely fictional and illustrative.** Cymbal Bank is a fictional persona. Every figure marked `ILLUSTRATIVE` is a constructed arithmetic input, not a measurement, a benchmark or a published fact. No real institution is described, and nothing here should be read as a claim about any real bank's systems.

### 14.1 The situation

Cymbal Bank's retail service chatbot handles card services, balances and transaction queries for consumer customers. It is a rung-2/3 system (§2): an intent-and-entity bot with a retrieval-grounded FAQ surface, built on a platform whose session model was designed for a web widget. The problems are the ones this guide predicts:

- Escalations arrive at human agents with no context — agents ask customers to repeat everything.
- Customers who start in the app and continue by phone start over.
- The bot's session expires in 30 minutes, so a conversation resumed the next morning is a new conversation.
- Nobody can say which flows the model is allowed to decide, and there is no register of states.

### 14.2 The channel mix and what each channel forces

| Channel | Share of volume | What it forces on the state design |
|---|---|---|
| Web / app widget | `ILLUSTRATIVE` 55% | Rich responses available, so slots can be collected as structured forms; but the widget is closed mid-flow routinely, so flow position must be server-side. |
| Messaging platform (WhatsApp) | `ILLUSTRATIVE` 30% | Async by construction; the 24-hour customer service window and the template-only rule outside it (§8.3) make re-engagement a change-controlled template exercise and make a 30-minute session TTL unusable. |
| Phone / voice (IVR + voice bot) | `ILLUSTRATIVE` 15% | No rich responses, mandatory barge-in (§9.3), and a latency budget measured in hundreds of milliseconds — so the deterministic core must carry the flow, with the model used only for phrasing. |
| Email | `ILLUSTRATIVE` <1% | Threading and quoted replies; retained as a channel but explicitly *not* given autonomous replies in phase 1. |

The decision that follows immediately: **the conversation state moves server-side and becomes channel-independent** (§4.3, split placement), keyed on `customer_ref` plus a Cymbal-minted `conversation_id`. The platform's session identifier is demoted to a routing field. The session TTL is replaced with a conversation TTL aligned to the messaging window and the record-retention policy.

### 14.3 The deterministic-vs-model split, and the one flow that stays fully deterministic

The split follows §6.4. The model does three things: it resolves the utterance into a proposed structured object (intent + entities + a proposed transition), it phrases responses, and it decides *among permitted* clarifications. The controller does the rest.

**The one flow Cymbal keeps fully deterministic: lost/stolen card replacement.** Reasons, in order of weight:

1. **Legal consequence.** Blocking a card and ordering a replacement is a regulated, financially consequential action, and the wrong card blocked is a customer harm that no conversational quality compensates for.
2. **It contains the identity-verification control.** Authentication must precede the action, and an authentication control that a model can skip (or be talked past) is not a control (§13.1).
3. **It is exercised under stress.** Customers calling about a lost card are frequently distressed and under time pressure, which is exactly when a model's judgement is least reliable and when the affective and repeated-failure escalation triggers (§10.1) are most likely to fire.
4. **It must work on voice.** It is the flow most often handled by phone, where the latency budget (§9.2) makes extra model calls expensive.
5. **It is testable as-is.** Every transition is a unit test, and the identity-verification step is independently assertable — which is the change-control evidence Cymbal needs (§13.3).

So: states `AUTH_CHALLENGE → AUTHENTICATED → CARD_ACTION_PENDING → CARD_BLOCKED → REPLACEMENT_ORDERED → COMMITMENT_STATED`, with the model permitted to route *into* `AUTH_CHALLENGE` and to phrase the confirmation *after* `REPLACEMENT_ORDERED` — and permitted nothing in between.

### 14.4 The state sketch and the window-management strategy

Cymbal's state record is the §4.6 shape, extended with the channel sessions and the disclosure flag:

| Element | Cymbal's treatment |
|---|---|
| `conversation_id` / `customer_ref` | Minted by Cymbal; never the channel's identifier (§8.2). |
| `channel_sessions[]` | Every widget session and every WhatsApp window and every call, so a channel switch continues one conversation. |
| `state` | Explicit, from the small enumerated set above; reviewed by Cymbal's change function. |
| `slots{}` with `confirmed` | Per-slot confirmation; unconfirmed values may be read back but never acted on. |
| `flags.disclosure_served` | Set on the first turn of the first interaction; asserted in tests. |
| `commitments[]` | Every statement of a timescale, so the handoff and the bot agree later. |
| `provenance` | Bot version, policy version, sources — required for the record (§13.2). |

**Window-management strategy: selective retention plus rendered state** (§5.3). Cymbal keeps verbatim: the authentication exchange, every slot confirmation and correction, every commitment, and every escalation trigger firing. It summarises: greetings, restated FAQ exchanges, and abandoned digressions. Because the facts live in structured state and the prompt is *rendered* from it, the window stays small and the summarisation cannot lose a decision-relevant fact — it can only lose small talk. The regression suite asserts on state transitions (§12), so a summarisation change that drops a commitment fails a test rather than surfacing as a customer complaint.

### 14.5 The handoff design

| Element | Cymbal's design |
|---|---|
| **Triggers** | All five (§10.1): confidence below threshold, explicit request, two failed attempts at the same intent, any intent declared regulated, and any affective signal. The last two are hard triggers and do not depend on the model's confidence. |
| **What crosses** | The full structured conversation record — transcript, slots with confirmation status, unresolved intents, the trigger that fired and when, the commitments made, and the provenance. Rendered into the agent desktop as a panel; **not** a model-written summary (§10.2). |
| **While human-active** | The state enters `HUMAN_ACTIVE`, which disables bot responses structurally rather than by a toggle an operator must remember (§10.4). |
| **Resumption** | Phase 1: the bot does not resume after a human handoff. Phase 2 introduces a designed resumption flow with a synchronisation step. The honest phase-1 position is stated rather than assumed. |
| **Capacity** | The escalation rate is monitored as a capacity input alongside containment (§10.3), and the programme explicitly states the staffing assumption it depends on. |

### 14.6 The disclosure treatment

Cymbal serves disclosure on the first turn of the first interaction, records `disclosure_served = true`, and asserts the flag in the regression suite. On the widget it is a visible, clearly-worded banner plus a first-message statement; on voice it is spoken at the start of the call; on WhatsApp, outbound re-engagement uses an approved template carrying the disclosure wording, and inbound-first conversations state it in the first free-form reply within the 24-hour window. The disclosure is a change-controlled artefact because the WhatsApp template cannot be edited without re-approval (§8.3), and because Article 50 disclosures must be clear, distinguishable and accessible (§11).

### 14.7 The cost and latency arithmetic

**All figures `ILLUSTRATIVE`** — constructed inputs, not measurements.

Assume, per conversation: `ILLUSTRATIVE` 8 turns median, `ILLUSTRATIVE` 3,500 prompt tokens and `ILLUSTRATIVE` 250 output tokens per turn, and that a rendered-from-state prompt is `ILLUSTRATIVE` 40% the size of a transcript-accumulating prompt by turn 8.

| Quantity | Transcript-accumulating | Rendered from state | Note |
|---|---|---|---|
| Prompt tokens across 8 turns (sum) | ~`ILLUSTRATIVE` 28,000 | ~`ILLUSTRATIVE` 11,200 | The gap widens with conversation length (§5.4). |
| Relative prompt-token cost | 100% | ~40% | `ILLUSTRATIVE` |
| Turn 8 prompt size | ~`ILLUSTRATIVE` 5,600 | ~`ILLUSTRATIVE` 2,240 | State rendering caps growth. |
| Added per-turn latency from state fetch + render | — | `ILLUSTRATIVE` +20–40 ms | Paid to avoid a much larger prompt (§9.2 for why this matters on voice). |

The architectural conclusion is the one §4.3 and §5.4 predict: **rendering the prompt from structured state is a cost and latency strategy, not tidiness.** On the voice channel it is also a turn-taking requirement, because every avoided prompt token is time the caller is not waiting in silence.

### 14.8 The recommendation, including what Cymbal will NOT let the model handle

Cymbal adopts: server-side channel-independent state; the deterministic `AUTH_*`/`CARD_*` flow; the hybrid pattern everywhere else; selective retention with rendered prompts; the full-record handoff with `HUMAN_ACTIVE`; disclosure as a state flag with change-controlled templates; and state-transition assertions in the regression suite.

And Cymbal decides explicitly that the model will **not**:

- **Decide whether identity has been verified.** The model may route to the challenge; it may not evaluate the result.
- **Decide whether a high-consequence action proceeds.** Card block, replacement order, fee reversal and complaint outcome are controller decisions.
- **Decide whether to escalate on a regulated-intent or affective trigger.** Those fire deterministically; the model's confidence is not a gate on them.
- **Compose the disclosure wording at runtime.** It is a fixed, reviewable, accessible artefact.
- **Summarise the conversation for the handoff.** The record transfers, not a model's summary of it.

Each of those five is a boundary that a control function can inspect, which is precisely the point: the value of the design is not that the model is good, it is that the places where the model is not trusted are *named* (§13.4).

---

## 15. The Anti-Patterns

Seven failures, each with its symptom, its cause and the guardrail that prevents it.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **The accidental state machine** | Nobody can enumerate the conversation's states; every change breaks something unrelated; QA cannot describe how to reach a state. | The dialogue was allowed to emerge in the prompt; states were never declared (§4.4). | Enumerate the states and transitions; review them; put every consequence-bearing transition in the explicit machine and assert on it in tests (§12). |
| 2 | **Summarised until it forgets the decision-relevant turn** | The bot contradicts a commitment it made earlier; a slot confirmed in turn 3 must be re-asked in turn 12. | Summarisation (§5.3) applied uniformly, including to turns that established facts. | Selective retention — keep commitments, confirmations and corrections verbatim — plus structured state so facts never live only in prose. |
| 3 | **The chatbot that cannot continue after a channel change** | A customer starts in the app, calls, and starts from zero; or the session expires overnight mid-flow. | Conversation identity was bound to the channel's session identifier, and state was session-scoped (§8.2, §8.3). | Mint a channel-independent `conversation_id` keyed to the customer; make the conversation record durable with a TTL aligned to retention, not to a widget socket. |
| 4 | **The model making a legally consequential decision because nobody drew the boundary** | A fee is reversed, an account is actioned, or a regulated statement is made by model judgement with no rule, no version and no test. | The hybrid boundary of §6.4 was never drawn; determinism was treated as a prompt instruction rather than a structural constraint. | A named list of consequence-bearing decisions the model may not make, implemented as controller states, reviewed by the control function (§13.1, §14.8). |
| 5 | **The handoff that hands over a blank screen** | The human agent asks the customer to repeat everything; the agent contradicts a commitment the bot made. | The handoff interface was a notification ("escalated") rather than the conversation record (§10.2). | Transfer the structured state plus the transcript plus the trigger plus the commitments; never a model-written summary. |
| 6 | **The voice bot designed to a text latency budget** | Callers talk over the bot, repeat themselves, or hang up; the first response arrives seconds after the caller stops speaking. | The text latency tolerance was reused unchanged on a channel where latency is a turn-taking failure (§9.2). | An explicit voice latency budget with streamed generation overlapped with synthesis, a deterministic fast path, and mandatory barge-in support. |
| 7 | **The intent taxonomy maintained by nobody** | The fallback bucket grows; users' actual phrasings are unrecognised; every taxonomy change invalidates the utterance corpus. | An intent taxonomy was created as a project deliverable with no owner and no review cadence (§6.2). | Name an owner, put the taxonomy under the same change control as the flows, review the fallback utterances on a schedule, and treat additions as product changes. |

The common thread in all seven: **each is a state-design failure presenting as a conversational quality problem.** In every case the visible symptom is what the bot *said*; the cause is what the architecture *kept*.

---

## 16. The Claims Audit, What Could Not Be Verified, Glossary, Cross-References and Closing

### 16.1 The claims audit

Every framework status, platform capability, standard and regulatory requirement asserted in this guide, with its source, the date it was read, and a quality marker. Items flagged ❌ must not be adopted; items flagged ⚠ are uncertain.

| # | Claim | Source | Read on | Marker |
|---|---|---|---|---|
| 1 | VoiceXML 3.0 is a W3C Working Draft, latest version 16 December 2010, described in-document as a work in progress; it never reached Recommendation. | W3C, `w3.org/TR/voicexml30/` | 23 Sep 2026 | `VERIFIED` ❌ **stagnant — do not design to it** |
| 2 | VoiceXML 2.0 is a W3C Recommendation (March 2004); VoiceXML 2.1 is the family's latest Recommendation. | W3C, `w3.org/Voice/` (specification list) | 23 Sep 2026 | `VERIFIED` (legacy, stable) |
| 3 | The W3C Voice Browser Working Group was **officially closed on 2015-10-12**. | W3C, `w3.org/Voice/` | 23 Sep 2026 | `VERIFIED` |
| 4 | SSML 1.1 is a W3C Recommendation dated 7 September 2010. | W3C, `w3.org/TR/speech-synthesis11/` | 23 Sep 2026 | `VERIFIED` |
| 5 | SRGS 1.0 is a W3C Recommendation dated 16 March 2004. | W3C, `w3.org/TR/speech-grammar/`; corroborated by a vendor grammar-standard reference | 23 Sep 2026 | `VERIFIED` |
| 6 | SCXML 1.0 is a W3C Recommendation dated 1 September 2015, published by the Voice Browser WG. | W3C, `w3.org/TR/scxml/` | 23 Sep 2026 | `VERIFIED` |
| 7 | CCXML 1.0 is a W3C Recommendation dated July 2011; SISR 1.0 and PLS 1.0 are W3C Recommendations in the same family. | W3C, `w3.org/Voice/`; `w3.org/TR/scxml/` references | 23 Sep 2026 | `VERIFIED` |
| 8 | MRCPv2 is IETF Standards Track, RFC 6787, November 2012. | IETF, `rfc-editor.org/rfc/rfc6787.txt` | 23 Sep 2026 | `VERIFIED` |
| 9 | The Microsoft Bot Framework SDK repository **was archived by the owner on 5 January 2026** and is read-only. | GitHub, `microsoft/botframework-sdk` | 23 Sep 2026 | `VERIFIED` ❌ **dormant — do not adopt** |
| 10 | Microsoft **LUIS was retired on 1 October 2025**; new resources blocked from 1 April 2023. Successor: Azure AI Language. | Microsoft Learn, Bot Framework availability FAQ (page last updated 2 Jan 2025) | 23 Sep 2026 | `VERIFIED` ❌ **retired** |
| 11 | Microsoft **QnA Maker was retired on 31 March 2025**; new resources blocked from 1 October 2022. | Microsoft Learn, same FAQ | 23 Sep 2026 | `VERIFIED` ❌ **retired** |
| 12 | Google's **Dialogflow CX console is deprecated**; its features are included in the **Conversational Agents console**, which also absorbs Vertex AI Agent Builder console features. Page last updated 22 Sep 2026. | Google Cloud docs, Dialogflow CX documentation | 23 Sep 2026 | `VERIFIED` (console superseded) |
| 13 | **Amazon Lex V2** is live, with a current AWS developer guide and SDK integration documentation. | AWS docs, `docs.aws.amazon.com/lexv2/latest/dg/what-is.html` | 23 Sep 2026 | `VERIFIED` |
| 14 | `ibm.com/products/watsonx-assistant` now presents as **IBM watsonx Orchestrate**, an agent-management platform. | ibm.com (page as served) | 23 Sep 2026 | `⚠` absorbed — no formal retirement notice read |
| 15 | **Botpress** is live but repositioned: its repository describes it as "the open-source hub to build & deploy GPT/LLM Agents"; MIT, ~14.9k stars. | GitHub, `botpress/botpress` | 23 Sep 2026 | `VERIFIED` |
| 16 | **Rasa Open Source** is live and maintained by the community (public repo, default branch `3.6.x`, Apache-2.0); Rasa's docs state the open-source project is community-maintained, with Rasa Pro as the commercial product. | GitHub `RasaHQ/rasa`; `rasa.com/docs/rasa/` | 23 Sep 2026 | `VERIFIED` (community-maintained) |
| 17 | The WhatsApp Business Platform enforces a **24-hour customer service window**; free-form service messages may only be sent inside it; outside it only pre-approved template messages may be sent; opt-in is required before templates; default delivery time-to-live is **30 days** (10 minutes for authentication templates). | Meta for Developers, "Service messages" (doc updated 21 May 2026) | 23 Sep 2026 | `VERIFIED` |
| 18 | **EU AI Act Article 50(1)** requires that people interacting directly with an AI system be informed of that; Commission guidelines cover **chatbots, AI agents and avatars**, require notification from the start of the first interaction, and read the "obvious" exception restrictively. **Applies from 2 August 2026**; marking grace period to 2 December 2026 for systems placed before 2 August 2026; fines up to €15 million or 3% of worldwide turnover. | `artificialintelligenceact.eu/article/50/` (Article 50 text and Article 113 commencement); European Commission, `digital-strategy.ec.europa.eu` Article 50 FAQ (last updated 24 July 2026) | 23 Sep 2026 | `VERIFIED` |
| 19 | **California B&PC §17941** (Bolstering Online Transparency Act, SB 1001): using a bot to mislead about its artificial identity to deceive about content and incentivise a purchase/sale or influence a vote is unlawful; disclosure must be clear, conspicuous and reasonably designed. Effective 1 January 2019, operative 1 July 2019. | California Legislative Information, `leginfo.legislature.ca.gov`, Business and Professions Code §17941 | 23 Sep 2026 | `VERIFIED` |
| 20 | DeepEval's **`ConversationalTestCase`** is a multi-turn test case (conversation history plus final output) used by conversation and agent metrics. | Repository: `technology/ai_llm/rag/deepeval_guide.md` §2.4 and §4.2 | 23 Sep 2026 | `VERIFIED` (in-repo) |
| 21 | The voice latency budget (~900 ms to first audio, stage split) is constructed arithmetic for reasoning, not a published target. | This guide, §9.2 | 23 Sep 2026 | `ILLUSTRATIVE` |
| 22 | The voice chain joint-accuracy example (~77% over five stages at 95% each) is constructed arithmetic. | This guide, §9.4 | 23 Sep 2026 | `ILLUSTRATIVE` |
| 23 | All Cymbal Bank figures — channel mix, turn counts, token counts, cost ratios, latency deltas — are constructed inputs. | This guide, §14 | 23 Sep 2026 | `ILLUSTRATIVE` |
| 24 | The escalation-rate illustration (25%) in §10.3 is a constructed example, not a benchmark. | This guide, §10.3 | 23 Sep 2026 | `ILLUSTRATIVE` |

### 16.2 What Could Not Be Verified

Stated plainly, because the alternative is to imply a sourcing I do not have.

1. **A published, normative end-to-end latency target for conversational voice agents.** I found no standards body or vendor document stating a required time-to-first-audio for a voice assistant. The ~900 ms budget in §9.2 is therefore `ILLUSTRATIVE` — my own arithmetic for reasoning about the split, not a target anyone has published. Treat any round number quoted to you as a design target as unsourced unless a primary document is produced.
2. **A published figure for human conversational turn-taking gaps** (the "200 ms" style claim that circulates widely). I could not verify it at source in this session, so this guide does not assert it, and §9.2 argues the requirement qualitatively instead.
3. **Rasa Open Source being in "maintenance mode".** A third-party review asserted this, and the observation that the default branch is `3.6.x` while the commercial direction is Rasa Pro is consistent with it — but I did not read Rasa's own release-and-maintenance policy page, so the *specific* maintenance-mode claim is `⚠` unverified here. What is verified is that Rasa Open Source is community-maintained and Rasa Pro is the commercial product.
4. **A formal IBM retirement notice for watsonx Assistant.** I observed only that the product URL now serves a watsonx Orchestrate page. No end-of-support or retirement announcement was read, so the "absorbed" conclusion is `⚠`, not `VERIFIED`.
5. **Retention periods for conversational records in banking.** §13.2 asserts that retention, retrievability, completeness and immutability are required; it does not assert specific periods, because I did not read the applicable instruments in this session. The regulatory treatment is owned by [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3.
6. **Non-W3C, non-IETF voice-dialogue standards** (ETSI, ITU-T, ISO). I surveyed only the W3C Voice Browser family and IETF MRCPv2. Other standards bodies may have relevant instruments that I did not examine.
7. **The current activity of the VoiceXML Forum.** W3C's page links to it; I did not read it, so its present relevance is unassessed.
8. **Containment-rate, escalation-rate and deflection benchmarks for banking chatbots.** No primary source was found. The 25% figure in §10.3 is illustrative.
9. **Whether Article 50's provider/deployer allocation applies to a specific bank's configuration.** Article 50(1) places the obligation on *providers*; the Commission's guidance note that the system/provider entity matters. Deciding which role a specific deployment holds is a legal determination, not an architectural one, and this guide does not make it.
10. **Tool limitation.** Several `web_search` queries in this session returned empty result sets (notably for framework-status and voice-standard queries) while `web_extract` against the same organisations' primary URLs succeeded. Where that happened I used `web_extract` on the primary source and named the source in §6.5 and §9.5. An empty search result is a tool limitation; it is **not** evidence that a fact is absent.

### 16.3 Glossary

| Term | Definition as used in this guide |
|---|---|
| **Agent** | A goal-directed system that plans, calls tools and iterates toward task completion. Its state is the task. See [autonomous_agents_guide.md](autonomous_agents_guide.md). |
| **Anti-pattern** | A recurring design failure whose symptom is conversational and whose cause is architectural (§15). |
| **Barge-in** | The user speaking while the system is still speaking; required on voice, meaningless on text (§9.3). |
| **Channel** | The surface a conversation happens on, which constrains response richness, length, presence and interruption (§8). |
| **Chatbot / conversational system** | A system whose primary product is the exchange itself, not a completed task (§2). |
| **Commitment** | A statement the system makes about a future action; a first-class state element because it constrains later turns (§4.1). |
| **Context window** | The finite token budget of a single model call; a constrained resource that must be allocated, not merely filled (§5). |
| **Conversation state** | The durable, structured record of what has been established, what remains unresolved, and what the system has committed to (§4). |
| **Conversational RAG** | Retrieval where the query must be derived from the conversation rather than the utterance alone. Owned by [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md) §9, §13. |
| **Deterministic flow** | A scripted path with declared states and transitions; same input, same next step. |
| **Dialogue manager** | The component deciding the next action from the state and the incoming utterance (§6). |
| **Disclosure** | Informing the user they are interacting with an AI system; a legal obligation in some jurisdictions, verified in §11. |
| **Endpointing** | Deciding the user has finished speaking (§9.3). |
| **Entity** | A value extracted from language and bound (or bindable) to a slot or action. |
| **Escalation / handoff** | Transfer of the conversation to a human, carrying context; an architectural path, not an error handler (§10). |
| **Grounding** | Tying an answer to an identified source rather than parametric recall (§7). |
| **Hybrid pattern** | Model handles language; deterministic controller handles consequence (§6.4). |
| **Intent** | A labelled categorisation of what the user wants; a maintained taxonomy, not a model output. |
| **Rung** | A position on the capability spectrum of §2, from menu-driven bot to agentic assistant. |
| **Session** | A bounded period of continuous interaction; the channel's unit, not the conversation's (§4.2, §8.2). |
| **Slot** | A named field needed to complete a task, with a value, provenance and confirmation status. |
| **Turn** | One user input and the system's response; the unit of latency, cost and testing. |

### 16.4 Cross-references

**Owned by this guide:** the conversation state (§4), the context-window requirement (§5), dialogue-management patterns (§6), the channel layer (§8), the voice dimension (§9), the human handoff (§10), and the chatbot-versus-agent distinction (§2).

**Deferred to, by name:**

- Conversational retrieval, query transformation, question condensation — [rag/query_rewriting_rag_guide.md](rag/query_rewriting_rag_guide.md) §9, §13; [rag/advanced_rag_techniques_guide.md](rag/advanced_rag_techniques_guide.md); [rag/rag_optimization_techniques_guide.md](rag/rag_optimization_techniques_guide.md).
- Regulatory treatment of client-facing assistants — [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) §7.3.
- Context and memory engineering mechanics — [context_engineering_guide.md](context_engineering_guide.md).
- Agent architectures, scaffolding, production, cost control, deployment — [autonomous_agents_guide.md](autonomous_agents_guide.md); [agent_scaffolding_guide.md](agent_scaffolding_guide.md); [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) §5, §7; [agentops_guide.md](agentops_guide.md) §3–§6; [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) §5; [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md); [agents_at_scale_guide.md](agents_at_scale_guide.md); [feedback_mechanisms_llm_agents_guide.md](feedback_mechanisms_llm_agents_guide.md); [multi_pass_llm_pipelines_guide.md](multi_pass_llm_pipelines_guide.md).
- Guardrails and prompt injection — [llm_guard_models_guide.md](llm_guard_models_guide.md); [prompt_injection_guide.md](prompt_injection_guide.md).
- The gateway and egress layer — [../enterprise_ai_gateway_guide.md](../enterprise_ai_gateway_guide.md).
- Evaluation frameworks and conversational testing — [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md); [rag/deepeval_guide.md](rag/deepeval_guide.md) (the `ConversationalTestCase`).
- Speech and real-time audio — [ai_speech_tutor_research.md](ai_speech_tutor_research.md); [../livekit_alternatives_guide.md](../livekit_alternatives_guide.md); [../accent_detection_guide.md](../accent_detection_guide.md).

### 16.5 Closing summary

A conversational system is not a smaller agent and not a wrapper around a model. It is an architecture for holding a state that neither the model nor the channel will hold for you: the model is stateless, the context window is finite and metered, and the channel can drop mid-sentence or sleep for a day.

Everything in this guide follows from taking that seriously. The layering exists to put the state somewhere durable and to keep the model out of the decisions with consequence. The context window exists as a *budget* to be allocated by selective retention, not a buffer to be filled. The dialogue-management choice is ultimately a choice about how much of the state machine is declared and reviewable. The channel layer dictates what a turn is; the voice layer makes latency a correctness property; the handoff is where the state either transfers or is destroyed; and the regulated deployment adds records, disclosures and change control on top of all of it. The anti-patterns are all one failure wearing seven costumes: the architecture kept the wrong thing.

Build the record, declare the states, draw the boundary, and hand over what you have — because an agent's state is its task, but a chatbot's state is the conversation, and the whole architecture exists to manage a state that must outlive a stateless model and a finite context window. In one line: the conversation is the state.

