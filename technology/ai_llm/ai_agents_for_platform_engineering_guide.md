# AI Agents for Platform Engineering — The Agent as the Platform Team's Colleague

> **Standfirst.** This guide is about an agent doing platform-engineering work — scaffolding a service, drafting a migration, triaging a ticket queue, reviewing infrastructure-as-code — and it is not about the platform that hosts agents. The repository already owns the inverse subject in [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md), which defines an AI platform as the internal developer platform for AI and treats agents as the workload being hosted; that guide's governance surface is the agent as a *tenant*. Here the agent is on the *other side of the counter*: a participant in the platform team's own work, judged by whether the road it paves is one other people can walk on. Every ambiguity in this subject comes from readers collapsing those two positions into one word, so §1 separates them by name, §3 separates them by test, and the rest of the guide keeps them apart. What follows is not a productivity argument. It is an argument about which platform-engineering acts an agent may perform, which it may only prepare, and which it must never be the author of.

> **Byline:** Jack Liu Shurui, Solution Architect  
> **Date:** October 2026  
> **Series:** LLM/AI Engineering & Operations · **Domain:** Platform Engineering · Agent-Assisted Delivery  
> **Sources read:** 2026-10-19

> **How to read it.** §1 defines the terms, states the thesis and declares the boundary against ten sibling guides. §2 states what platform engineering is in one page and names the three properties that make delegating platform work different from delegating code work. §3 settles the direction question with a test a reader can apply in one sentence. §4 is the activity rulings — five platform functions, each with what an agent may do, what it must not do, and which named human remains accountable. §5 is the control surface an agent must not touch. §6 reverses the direction and treats the agent as a platform capability the team ships. §7 is blast radius. §8 is metrics and why attribution fails. §9 is the tooling landscape by category, with nothing ranked. §10 is the change board. §11 is the failure modes, §12 the Cymbal Bank worked example, §13 the anti-patterns, §14 the honest state of practice. §15 is the claims audit and §16 the negatives, glossary, cross-references and closing.

> **Verification policy.** Facts marked **(verified)** were checked against the named source on 2026-10-19. Every claim in §15 carries a date and a kind: a vendor-community reference page is not a research finding, a benchmark is not a deployment result, and a regulation is not an implementation. Where this guide needed a source it did not have, the claim is marked **(flagged)** or recorded in §16 rather than filled from memory. No productivity, adoption or percentage-improvement statistic appears anywhere in this guide, by design — see §15 row 15.

---

**Table of contents**

1. [Overview, the Decoder, the Two Senses, the Thesis, and the Boundary](#1-overview-the-decoder-the-two-senses-the-thesis-and-the-boundary)
2. [What Platform Engineering Actually Is, in One Page](#2-what-platform-engineering-actually-is-in-one-page)
3. [The Two Directions and the Third Case](#3-the-two-directions-and-the-third-case)
4. [What an Agent Can Genuinely Do in a Platform Function](#4-what-an-agent-can-genuinely-do-in-a-platform-function)
5. [What It Must Not Do, and Why](#5-what-it-must-not-do-and-why)
6. [The Agent as a Platform Capability](#6-the-agent-as-a-platform-capability)
7. [The Guardrails and the Blast Radius](#7-the-guardrails-and-the-blast-radius)
8. [The Metrics Question](#8-the-metrics-question)
9. [The Tooling Landscape, Structurally](#9-the-tooling-landscape-structurally)
10. [Banking and Regulated Change Control](#10-banking-and-regulated-change-control)
11. [The Failure Modes](#11-the-failure-modes)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The State of Practice](#14-the-state-of-practice)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-cross-references-and-the-closing-summary)

---

## 1. Overview, the Decoder, the Two Senses, the Thesis, and the Boundary

### 1.1 The question this guide owns

A platform team has a backlog it cannot clear: templates to scaffold, a base image to retire, a ticket queue that refills faster than it drains, documentation that decays the week after it is written, and infrastructure-as-code pull requests that sit for days because the two people who can review them are the two people writing the templates. An agent can now read a repository, propose a diff, open a pull request, classify a ticket and draft a page. The question this guide owns is not whether it can do those things — it demonstrably can, in a narrow and bounded sense (§14) — but **which of those acts it may be the author of, and which it may only prepare for a human to author.**

That question is a platform-engineering question, not a model question. It is answered by what the platform's artefacts *are*: a template, a default, a permission, a shared pipeline. Each is a thing whose effect is not visible in the diff that changes it. An agent that can write a correct diff and cannot see the population it lands on is not a junior engineer; it is a very fast editor of a document whose readers it has never met.

### 1.2 The decoder

Seven terms recur. Each is defined in the sense this guide uses, and each is a platform object rather than a metaphor.

| Term | Definition in this guide | Why it matters here |
|---|---|---|
| **Platform engineering** | The discipline of building and running a shared substrate — tooling, workflows, defaults — so that product teams can deliver without each mastering the whole estate. Its output is a product, not a service desk. | It is the discipline being *delegated to*, so its shape determines what is delegable. |
| **The internal developer platform** | The bound-together set of tools and services a platform team builds to provide golden paths and self-service: the substrate, the portal in front of it, and the automation underneath. | The artefact an agent would be changing. Its integrity is the thing at risk in §5 and §7. |
| **Golden path** | A preconfigured, end-to-end supported route through the platform for a common task, designed to reduce cognitive load and keep work safe and compliant (platformengineering.org, community reference; the definition as recorded in [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §2.3, whose page-level citation predates this guide and whose blog URL has since moved — §16.1). | The thing a scaffolding agent extends; a second, parallel golden path is a failure mode (§11.1). |
| **Paved road** | The older metaphor for the same idea, from Google SRE culture and Evan Bottcher's 2018 essay: the standardised, supported default — the desired way of doing things made the *easiest* way, not the enforced way. | The thesis is about paving. A pothole is a defect in the road; a preference is a choice about where the road goes. |
| **Platform team** | In Team Topologies, "a grouping of other team types that provide a compelling internal product to accelerate delivery by Stream-aligned teams" (teamtopologies.com/key-concepts, authors' site, read 2026-10-19; Skelton & Pais, 2019). | The unit whose work is being delegated, and whose accountability cannot be delegated with it. |
| **Self-service** | The property that a developer can obtain a platform capability — an environment, a pipeline, a scaffold — without filing a ticket and without a platform engineer in the loop. | The measure of whether the platform is a product; also the reason a scaffolding agent's output reaches many teams at once. |
| **The agent** | A model-driven system with tools, a mandate and a loop: it reads state, decides an action, executes it, observes the result and continues. Not a code completion, not a chat window. | The actor whose permissible acts this guide rules on. |

**Why the terms are spelled out.** The three-letter acronym for the internal developer platform is avoided throughout this guide, deliberately. In this repository the same acronym is used for an identity provider — the authentication service that issues tokens — and for the internal developer platform, and both usages are current. A reader who meets the acronym cannot tell which one is meant, and in a guide about agents touching permissions the confusion is not cosmetic: an identity provider is part of the control surface §5 protects. Every occurrence here is written in full.

### 1.3 The two senses, separated

The word "agent" appears in platform-engineering conversations in three positions, and only one of them is this guide's subject. The distinction is not academic: the three have different owners, different risks and different failure diagnoses.

| Dimension | **(a) Agent as platform CUSTOMER** | **(b) Agent as platform-team COLLEAGUE** | **(c) The platform paving paths FOR agent developers** |
|---|---|---|---|
| **Who acts** | The agent, on behalf of a product team, consuming platform capabilities. | The agent, on behalf of the platform team, producing or changing platform capabilities. | The platform team, building a golden path that agent builders walk. |
| **What it acts on** | The paved road: environments, pipelines, model gateways, catalogues, data entitlements. | The road itself: templates, defaults, module interfaces, pipelines, documentation, the ticket queue. | The road's *route*: sandboxing, eval, tracing, gateway policy, agent runtime. |
| **Whose risk it is** | The consuming team's product risk, plus the platform's tenancy and quota risk. | The platform's integrity risk, inherited by every team downstream of the default. | The platform's own product risk: whether agent teams can ship at all. |
| **Owned by** | [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md), which treats the agent wave as a multiplier of the governance surface. | **This guide.** | [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §6, the prompt-to-production path. |
| **Typical failure diagnosis** | "The platform's guardrails did not contain the workload." | "The agent changed something it could not see the population of." | "The path does not reach production for agent workloads." |

**Every ambiguity in this subject comes from readers assuming (a).** When a platform leader says "we are putting an agent in the platform team", half the room hears "we are hosting an agent" and starts asking about sandboxing, entitlements and eval gates — all correct questions about (a), all answered elsewhere. The other half hears "we are delegating platform work" and asks whether the agent can be trusted with the template — the question this guide answers. Both halves are asking about agents and platforms; they are not having the same conversation.

### 1.4 The thesis

> **The platform team's product is a paved road; an agent that paves it must be able to tell a pothole from a preference.**

The claim has three parts and each is load-bearing.

**First, the platform's product is a road, not a deliverable.** A service shipped by a product team is one artefact with one owner. A template, a default or a pipeline is a *generator* of artefacts, most of which do not exist yet. The platform team's leverage and its exposure are the same fact: it writes once and a hundred teams inherit. That is why the third property of §2 — the output is a road other people walk on — is the one that makes delegation different.

**Second, the road is full of conventions that look like defects.** A Dockerfile that does not follow the house pattern may be an oversight, or it may be a vendor certification pinned to a base image, a regulatory evidence requirement attached to build provenance, or a team halfway through its own migration. From inside the repository the two cases are indistinguishable; from inside the organisation they are not. An agent that optimises against a convention it cannot see does not fail loudly — it succeeds efficiently at the wrong thing.

**Third, the distinction between a pothole and a preference is not a model-capability question.** It is a question about where the knowledge lives. Potholes are visible in the artefact: a broken link, a deprecated base, a template that no longer renders. Preferences live in the organisation: a decision taken two years ago, a certification, a freeze window, a team boundary. No amount of repository context contains them, which is why every ruling in §4 ends with a named human and every failure mode in §11 is a variant of the same error.

The thesis is tested against each activity ruling in §4, against the control surface in §5, against the metrics problem in §8, and against one concrete decision in §12. Where the evidence complicates it, the complication is stated rather than smoothed: §14 concedes that on self-contained repository work the agent's judgement is stronger than the thesis's framing might suggest, and §12 concedes that the human exception call is itself fallible.

### 1.5 The boundary declared by name

This guide does not re-derive the platform-engineering canon, the AI-platform design, the prompt-to-production path, agent construction, agent operations, sandbox mechanics, posture taxonomy, versioning, cost, or enterprise agent-platform architecture. Each is owned, and each is cited by name and section rather than restated.

| Sibling guide | What it owns | What this guide borrows, and where |
|---|---|---|
| [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) | The platform-engineering canon (§2), the platform stack (§3), self-service and the prompt-to-production path (§6), the AI-platform design. The inverse subject. | §2 cites its canon; §6 cites its §3.8 service catalogue and §6 self-service machinery. Nothing about the AI platform is re-derived here. |
| [agentic_engineering_guide.md](agentic_engineering_guide.md) | The agent construction lifecycle: specification, tools, context, evaluation, iteration. | §4 assumes an agent built to that lifecycle; it does not describe how to build one. |
| [agentops_guide.md](agentops_guide.md) | Agent operations: deployment, monitoring, incident response for agents in service. | §7 and §13 assume its operational disciplines; §10 borrows its record-keeping posture. |
| [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) | Production operation of agents: reliability, failure handling, rollout. | §14's distinction between demonstrated and promised capability. |
| [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) | Sandbox and isolation mechanics: what the agent can reach, and how that is enforced. | §7 names containment requirements and cites this guide for the mechanics. It does not re-derive them. |
| [agent_decision_policy_guide.md](agent_decision_policy_guide.md) | The ask/recommend/act/wait posture and its banking authority mapping. | §4's may / must-not / accountable columns are an application of its postures to platform activities; §10 cites its §9. |
| [agent_versioning_guide.md](agent_versioning_guide.md) | Versioning of agents, prompts and tool surfaces. | §6's requirement that a shipped agent be a catalogued, versioned entity. |
| [../finops_agents_guide.md](../finops_agents_guide.md) | The cost of agents: token spend, run economics, attribution of agent cost. | Nothing directly — the cost of running the agent is a separate accounting question owned there, and §16.3 points to it rather than summarising it. |
| [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) | Enterprise agent-platform architecture: the platform that hosts and governs agents at scale. | §3's sense (a) routing; §6's governance demands. |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | Scaffolding **agents** — generating an agent's skeleton, tools and harness. | Nothing directly; §4.1 scaffolds **services**, which is a different act on a different artefact, and the two are routinely confused in vendor material. |
| [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) | Risk controls and the evidence question: a risk without an owner and evidence is an observation, not a control. | §10's evidence pack and §12's change-board walk. |
| [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) | The regulatory obligations a bank's AI work must satisfy. | §10's framing of change control as an obligation, not a habit. |
| [../../management/high_performance_team_guide.md](../../management/high_performance_team_guide.md) | Team design, the platform-team type and its effectiveness conditions. | §2's account of the platform team as a constrained product team. |

The edge this guide owns is: **the ruling on which platform-engineering acts an agent may author, prepare or never touch, and the accountability that survives every delegation.**

### 1.6 What this guide does not argue

Three arguments are deliberately absent, and naming them prevents a misreading.

It does not argue that agents improve platform productivity. No such claim is made, no figure is offered, and §8 explains why the attribution cannot be made cleanly even in principle: the platform is a shared input, the confounders are the platform team's own roadmap, and a ticket closed by an agent may be a ticket that should not have existed. The subject is saturated with productivity claims; this guide declines to add one.

It does not rank tools, and §9 names projects only as instances of a category. A tool's suitability is a function of the estate it lands in, and an ordering by quality would be a claim about estates the author has not seen.

It does not treat "the agent" as a stable object. Capability moves; a ruling that is correct for a 2026 agent may be too permissive or too restrictive for the next one. The rulings in §4 are therefore written against **act classes and artefacts**, not against a model version, so that they survive a capability change without being re-derived — and so that a capability improvement cannot silently widen a mandate.

---

## 2. What Platform Engineering Actually Is, in One Page

The canon is not restated here. [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §2 works it properly — the internal developer platform as the bound-together substrate, Backstage as the portal pattern, golden paths and paved roads, platform-as-a-product, and the platform team as a Team Topologies type — and §3 of that guide gives the layered anatomy. What this section does is name the three properties of platform work that change what delegation means. They are not in the canon because the canon was written about humans doing the work.

### 2.1 The platform team is constrained

A platform team is small relative to the estate it serves, and its capacity is the binding constraint on everything it does. Every hour spent answering a ticket is an hour not spent on the road; every hour spent hand-holding one team's migration is an hour not spent making the migration unnecessary for the next ten. This is the economics that makes agent assistance attractive and it is also the economics that makes it dangerous: the team is under pressure, the queue is visible, and an agent that closes tickets looks like relief. The constraint is why "the agent drafts, the human reviews" is not a cost-free control — the reviewer is the scarce resource, and §5 and §13.6 are about what happens when the review is the bottleneck the agent was hired to relieve.

### 2.2 The platform team is a product team

Its users are developers, its success measure is adoption, and its discipline is product management rather than service provision. The canon is unambiguous: developers are internal customers, the platform is built on user research and continuous improvement, and a platform nobody uses is infrastructure theatre ([../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §2.4). Two consequences follow for this guide. An agent that improves the platform's *throughput* without improving what developers can self-serve has optimised the wrong variable. And an agent that makes the platform *harder to understand* — by generating volume a human must now interpret — has damaged the product, however many tickets it closed.

### 2.3 The product is a road other people walk on

This is the property that makes delegating platform work different from delegating code work, and it deserves to be stated precisely.

A product team's change is consumed where it is made: a function behaves differently, a test catches it, a reviewer sees the blast radius in the diff's own call sites. A platform team's change is consumed *later, elsewhere, by people who were not in the room*. When a template's default logging configuration changes, the diff shows one line in one repository and the effect is every service scaffolded from that template until someone changes it back. When a module's interface changes, the diff shows a signature and the effect is every caller that has not yet been written. When a pipeline's policy changes, the effect is every build that runs tomorrow.

Three consequences, and they organise the rest of the guide:

- **The reviewable artefact is not the consequential artefact.** A reviewer of a template diff is reviewing a line of configuration; the thing that will actually happen is a population of future services inheriting it. §5 is built on this gap.
- **The population is the unit of harm.** One bad service is an incident; one bad default is an incident generator. §7's blast radius is ordered by population, not by severity in the abstract.
- **Correctness is necessary and insufficient.** A change can be technically right, pass every check, and still be organisationally wrong — wrong team boundary, wrong timing, wrong exception. §11.4 and §12 are about exactly that.

The three properties compose into the thesis. A constrained team wants throughput; a product team is judged by what its users can do; and its product propagates. An agent can supply throughput on the first, can help on the second, and is structurally blind on the third unless the platform team builds it a way to see.

### 2.4 The delegation test the three properties produce

Taken together, the properties yield a test that is sharper than "is this task repetitive?". The question is not whether the work is repetitive — migrations and triage are, and so is typing. The question is **whether the work's correctness is decidable from the artefacts the agent can read.**

A scaffold's correctness is decidable: the template declares the parameters, the pipeline runs, the catalogue entry resolves, the owner accepts. A migration diff's per-repository correctness is decidable: the base image changes, the build passes, the tests run. A ticket's classification is decidable against the queue's own labels. Documentation's factual accuracy is partly decidable — the command either exists or it does not. And a review comment's usefulness is decidable by the human who reads it.

What is not decidable from those artefacts is whether the change *should* be made in this repository, at this time, for this team, given a decision taken elsewhere. That is the whole of the gap, and it is why §4's rulings give the agent the decidable part and reserve the rest. A platform team that applies this test will find that the delegable surface is larger than it expected and that the reserved surface is narrower but harder — which is the honest shape of the opportunity.

### 2.5 What the canon leaves to this guide

The canon was written to answer "how does a platform team build a platform", and it answers that well. It is silent on three questions that only arise when the team's capacity is augmented by something that is not a person, and the silence is not a defect — the canon predates the question.

**Who is accountable when the work is done by a tool?** The canon's accountability model is the team: a platform team owns the platform, and its engineers own their changes. Delegation to an agent does not change the model, but it does strain it, because the agent's contribution disappears into a diff that a human signed. §4's accountable column and §10's evidence pack are an attempt to keep the canon's model intact under that strain.

**What is the unit of review when the artefact is a generator?** The canon's review model assumes a change is reviewed at the point it is made. A template change is reviewed at the point it is made and consumed at a hundred points it is not, which is §5's whole subject.

**How does a platform team delegate without delegating its product judgement?** The canon's product discipline — discovery, roadmap, adoption — is judgement about what the road should be. An agent can execute a road; it cannot decide one, and a team that lets execution substitute for decision has stopped running a product.

None of the three is answered by better tooling, and none is answered by the canon. They are this guide's subject, and they are the reason it exists as a separate document rather than a section of the platform guide.

---

## 3. The Two Directions and the Third Case

### 3.1 The direction table

The three positions of §1.3, stated as a routing decision.

| Direction | Who acts, and for whom | The artefact it touches | Where the risk is recognised | Where it is answered |
|---|---|---|---|---|
| **(a) Customer** | The agent, for a product team or business function | Platform-provided capabilities: environments, gateways, data entitlements, runtime | With the consumer: a product incident, contained (or not) by the platform | [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §3, §4, §7 |
| **(b) Colleague** | The agent, for the platform team itself | The platform's own artefacts: templates, defaults, modules, pipelines, docs, queue | With the producer: a platform incident whose victims are other teams | **This guide**, §4–§7 |
| **(c) Subject** | The platform team, for agent developers | The golden path for agent workloads | With the platform's product: whether agent teams can ship at all | [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §6 |

### 3.2 The test a reader can apply

One question settles the direction in almost every case: **is the agent inside the platform team or outside it?**

Inside means the agent's output changes what the platform *is* — a template, a default, a permission, a shared pipeline, a catalogue entry that other people's services point at. Outside means the agent's output is a workload the platform *carries*, and the platform's job is to contain it. The test is about the artefact, not the tool: the same coding agent, with the same harness, is a customer when a product team points it at their service repository and a colleague when the platform team points it at the template repository. Nothing about the agent changed. Everything about the accountability changed.

A second test resolves the residual cases: **who inherits the effect?** If the answer is "the team that asked for it", the agent is outside the platform team. If the answer is "everyone who scaffolds from this template next quarter", it is inside.

### 3.3 What confusion costs

The two directions fail differently, and diagnosing one as the other wastes the remediation.

**An agent-as-customer failure diagnosed as a platform-governance failure.** A product team's agent does something it should not, and the review concludes the platform's guardrails were insufficient. Sometimes that is right. Often the actual finding is that the platform never expressed a policy the agent could obey — the guardrail existed as a wiki page, not as an enforced control — and the fix is a platform capability, not a stricter prompt. The failure gets logged against the wrong owner and the wrong roadmap.

**An agent-as-colleague failure diagnosed as a model-quality failure.** A platform agent "fixes" a Dockerfile that was deliberately non-standard, and the review concludes the model was not good enough. It may be. But the information the agent needed was not in the repository, and no model improvement supplies it. The remediation — an exception list, a narrower mandate, a routing rule — is organisational, and a model-quality diagnosis will not produce it. This is the most common misdiagnosis in the subject and the one §11 exists to prevent.

### 3.4 The routing

- Sense **(a)**, the agent as a workload on the platform: [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md), and for the hosting architecture, [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md).
- Sense **(c)**, the platform paving a golden path for agent developers: [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §6, and for the sandboxing and runtime mechanics, [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md).
- Sense **(b)**, the agent doing platform work: this guide, from §4.

### 3.5 One request, two directions

A single sentence of intent can belong to either direction, and the disambiguation is worth making concrete because it is where real programmes go wrong.

"This agent should keep our services on a supported base image." Read as **(a)**, it describes a workload that patches its own service repository — the product team's agent maintaining the product team's code, with the platform team supplying the base image and the pipeline. Read as **(b)**, it describes a platform capability that migrates the estate — the platform team's agent proposing changes across 120 repositories it does not own. The first is a product-team automation with a platform dependency. The second is a platform programme with a change board. They have different owners, different failure modes, different evidence requirements, and — critically — different answers to "who may merge". Confusing them is how a product team's automation gets treated as a platform mandate, or a platform migration gets waved through as "just a team's tooling".

The disambiguating question is always the same one: **whose repository is the agent writing to, and who inherits the effect?** If the write lands in repositories owned by teams other than the agent's principal, the agent is doing platform work whether or not anyone calls it that.

### 3.6 The consequence for who owns the risk

The directions differ not only in who acts but in where the risk is recognised, and that difference has a practical consequence for how a programme is governed.

In sense (a), the risk sits with the **consumer**: a product team's agent that does damage creates a product incident, and the platform's role is to have contained it. The platform's exposure is tenancy, quota and the shared blast radius of a multi-tenant control plane — which is why the AI-platform guide's governance surface is about isolation and entitlement rather than about review.

In sense (b), the risk sits with the **producer**: the platform team's agent that does damage creates a platform incident whose victims are other teams, and the platform's exposure is the integrity of its own defaults. There is no multi-tenancy problem, because the agent is not a tenant; there is an authorship problem, because the agent is writing the thing everyone else depends on.

The practical consequence is that the two directions need different controls, and importing one into the other produces theatre. A sandbox around a platform-team agent is a good idea and it does not answer §5's question, because the risk is not that the agent reaches something it should not — it is that the agent changes something it should not, with full entitlement to do so. Conversely, an approval gate in front of every product team's agent action is not platform governance; it is a bottleneck that the product team will route around.

---

## 4. What an Agent Can Genuinely Do in a Platform Function

### 4.1 The activity rulings

Five platform activities, ruled. The **may** column is what the agent can be trusted to author or execute without a human in the loop, bounded by the containment in §7. The **must not** column is not a caution; it is a prohibition, and each item is a class of act rather than an instance. The **accountable** column names a role, because "the team" is not an accountable party — nobody can be asked why.

| Activity | May | Must not | Accountable |
|---|---|---|---|
| **Scaffolding a service from a template** | Render the template with parameters; open a pull request; fill the catalogue entry; run the template's own checks; flag missing inputs | Merge; edit the template's defaults; invent a parameter the template does not declare; scaffold into a namespace or account it was not granted | **Service owner** (accepts the service and its ownership) and **platform engineer** (owns the template) |
| **Drafting a migration** (deprecated API or base image across many repositories) | Propose one diff per repository; open pull requests; group them by pattern; produce the inventory; draft the rollback note | Apply to any environment; merge; normalise a repository that does not match the pattern without an explicit exception decision; run a mass write against a shared branch | **Platform engineer** (authors the migration) with the **platform product owner** on exceptions |
| **Triaging platform tickets** | Classify; label; route to a queue; de-duplicate; summarise a thread; link to the relevant runbook; request missing information from the requester | Close as resolved; change priority beyond a declared band; reassign a ticket owned by another team; answer a question whose answer is a policy decision | **Platform engineer** (the queue's owner) |
| **Drafting and updating documentation** | Draft a page; update a runbook section from a merged change; correct a stale command; propose a diagram's text; open a docs pull request | Publish without review; delete a page; rewrite a policy statement; document a behaviour it has not observed in the artefact | **Platform product owner** (owns what the platform says about itself) |
| **Reviewing infrastructure-as-code** | Comment on a pull request; flag a policy violation with the policy cited; identify a resource with no owner tag; summarise the plan's risk surface; request changes | Approve; merge; apply; dismiss another reviewer's comment; approve its own authoring | **Platform engineer** (the reviewing human of record) |

The pattern in the table is consistent and worth naming before the subsections: **the agent may produce artefacts, and may not produce effects.** A pull request is an artefact. A merge is an effect. A comment is an artefact. An approval is an effect, because it is the thing the change-control process counts. Every "may" in the table is a proposal in a reviewable form; every "must not" is a commit.

### 4.2 Scaffolding a service from a template

Scaffolding is the activity most often proposed for agents and the one where the artefact boundary is clearest. The agent's real work is not the file generation — a template renderer already does that deterministically — but the *gap filling*: reading the requester's intent, choosing among declared parameters, noticing that the requested service name collides with an existing catalogue entry, and opening a pull request that a service owner can accept. That is genuinely useful, and it is bounded. The prohibition on editing the template's defaults is the load-bearing one: the moment the agent can adjust the default to make a particular scaffold work, it has changed the road to suit one traveller, and every future service inherits the adjustment. §5.3 is this prohibition in general form.

### 4.3 Drafting a migration

A migration across many repositories is the highest-value platform activity in this guide and the one where the agent's contribution is most real: the work is repetitive, the pattern is explicit, and the output is a set of reviewable diffs. The agent may inventory the estate, propose per-repository changes, group them by pattern, and draft the rollback note. It must not apply, and it must not decide what to do with a repository that does not match the pattern — because non-conformance is precisely the signal that the repository's history contains a reason the agent cannot read (§11.2, §12.4). The human accountable is the platform engineer who authors the migration, and the exception decision belongs to the platform product owner, who is the person entitled to decide that a convention has an exception.

### 4.4 Triaging platform tickets

Triage is classification, and classification is what current models do most reliably on bounded, well-labelled queues. The agent may label, route, de-duplicate, summarise and ask the requester for missing information. It must not close, because closure is the assertion that a human's problem is solved, and it must not answer a policy question — "can I get production access for this" is not a classification, it is a decision with an owner. The accountable human is the platform engineer who owns the queue; the queue's shape is that engineer's design, and an agent that routes badly makes the shape worse in a way that looks like volume.

### 4.5 Drafting and updating documentation

Documentation is the platform's least-maintained surface and the one where an agent's summarising ability maps most directly onto a real gap. Drafting a runbook section from a merged change, correcting a command that no longer exists, proposing text for a diagram — all are artefacts, all improve with review, and all are cheap to reject. The two prohibitions matter more than they look. Publishing without review makes the agent the platform's voice; deleting a page destroys knowledge the agent cannot know is load-bearing; and documenting a behaviour it has not observed in the artefact is how a plausible, confident, wrong runbook enters circulation. §13.1 is the anti-pattern that follows.

### 4.6 Reviewing infrastructure-as-code

Review is the activity with the strongest case for assistance and the weakest case for delegation. An agent reading a plan can do things a tired human reviewer does badly: notice that a resource has no owner tag, that a security group's ingress has widened, that a deletion is scheduled for something with dependents, that the change contradicts a policy the reviewer would have to look up. All of that is comment-shaped, and comments are artefacts. Approval is not. Approval is the act the change-control process counts, and an agent that approves has become the second eye in a four-eyes control — which §10.2 treats as a control failure rather than an efficiency. The accountable human is the platform engineer of record, and the agent may never be the reviewer of its own authoring, a rule §13.5 develops.

### 4.7 Three activities deliberately not ruled here

Three platform activities are absent from the table, and the absence is deliberate rather than an oversight.

**Incident command.** During a live incident the platform team's acts are time-critical and irreversible, and the value of an agent is in retrieval and summarisation — pulling the last three changes, the runbook, the dependency graph — not in acting. The posture question during an incident is an incident-response question, owned by operational practice and by [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) in its production form. Ruling it here would produce a rule that the situation overrides.

**Capacity and cost planning.** An agent can assemble the data and draft the analysis; the decision allocates budget and is a platform product decision. There is a real role for assistance and no role for authorship, and the economics are owned by [../finops_agents_guide.md](../finops_agents_guide.md).

**Hiring, performance and team design.** Platform work includes a managerial surface an agent should not touch at all. The team-design questions are owned by [../../management/high_performance_team_guide.md](../../management/high_performance_team_guide.md), and the boundary is worth stating only because vendor material occasionally implies otherwise.

The three absences share a property: in each, the consequential act is either taken under time pressure the agent cannot feel, or is an allocation of scarce organisational resource. Both are exactly where the preference side of the thesis dominates the pothole side.

---

## 5. What It Must Not Do, and Why

Section 4 ruled on activities. This section rules on a different object: the platform's **control surface** — the small set of artefacts whose change alters what the platform *is*, rather than what it hosts. The distinction matters because the activities in §4 can be delegated with review, and the acts below cannot be delegated with review, because review of the diff does not review the effect.

### 5.1 The control surface

| Element | What it is | Why changing it is categorically different |
|---|---|---|
| **The paved road itself** | The golden path, the template set, the module interfaces, the supported route | It defines what "supported" means; changing it redefines the platform's contract with every team, including teams that are not in the room. |
| **The platform's defaults** | The values a scaffold inherits when nobody chooses: logging, retention, network policy, resource limits, image pinning | A default is a decision made on behalf of people who will never see it. Changing it changes a population, not an instance. |
| **Permissions and guardrails** | Role definitions, namespace boundaries, admission policy, pipeline policy, credential scopes | These are the mechanisms by which every other control operates. Changing one is not a change *under* the control regime; it is a change *to* it. |
| **Approval of a platform change** | The accountable signature on a change to any of the above | It is the control itself. An approval produced by the thing being approved is not an approval. |

### 5.2 Changing the paved road

An agent may propose a change to a template, a module interface or a golden path. It may not author the decision. The reason is not that the change is technically hard — a template edit is often trivial — but that the template is a *promise about the future*. Every service scaffolded from it after the change inherits the new behaviour; every service scaffolded before does not. The estate becomes two populations with different inherited assumptions, and the platform team now maintains both whether or not it intended to.

There is a second, subtler reason. A golden path's value is that it is *one* path. Its authority comes from being the thing everyone does, which is what makes it supportable, documentable and auditable. A change that creates a fork — even a well-intentioned one, even a temporary one — spends that authority. §11.1 is the failure mode where an agent does this by accident, by building a parallel path that looks like an improvement.

### 5.3 Altering the defaults

This is the prohibition most likely to be violated by a well-meaning agent, because the violation looks like helpfulness. A scaffold fails because a default does not fit the requester's case; the agent adjusts the default and the scaffold succeeds. Nothing in the transcript looks wrong. The agent solved the problem it was given.

What it actually did was make one team's exception the estate's inheritance. The correct act is to leave the default alone and route the exception to the platform product owner, who can decide whether the default is wrong for everyone, wrong for a class of services, or right and this team is exceptional. That decision is a product decision about the road, and it belongs to the person who owns the road's shape. §12.4 is this exact ruling applied to a real-shaped decision.

### 5.4 Modifying permissions and guardrails

A guardrail change looks like a configuration line. `effect: deny` becomes `effect: audit`; a role gains one verb; an admission policy's namespace selector widens by one label. In a diff review these are indistinguishable from a logging tweak, and this is precisely the problem: the review process is calibrated to the *size* of a change, and a permission change's size is not its reach.

The reach is lateral. A widened role is a credential that more things can use; a narrowed guardrail is a class of action that no longer stops. Neither effect appears in the diff, because the diff shows the policy and the effect is the set of actors who will now pass through it. An agent must not author these changes at all, and the containment in §7 must make the attempt fail rather than rely on a reviewer noticing.

### 5.5 Approving a change to the platform

An agent may never be the approver of a platform change, including a change it drafted and including a change it did not. Approval is the act that makes the institution accountable for the change, and accountability is not transferable to a system that cannot be asked why. §10.2 develops the consequence: an agent that drafts a change generates evidence, and an agent that approves a change destroys it, because the record then shows an approval whose author cannot be examined.

### 5.6 Why "a human reviews the diff" is not sufficient

The standard mitigation — an agent proposes, a human reviews, the human merges — is necessary and it is not sufficient for the control surface. Four reasons, each independent.

**The diff does not show the population.** A template change is one line; the effect is every future scaffold. The reviewer sees the line and must reconstruct the population from memory, which is exactly the knowledge that leaves when the platform team's most senior engineer changes team.

**The effect is not proportional to the diff.** A guardrail's `deny` becoming `audit` is a word. A default's retention period changing from thirty days to seven is a digit. Review attention is allocated by apparent size, so the smallest diffs get the least scrutiny and carry the most reach.

**The reviewer's context is the reviewer's, not the institution's.** A reviewer who has been on the platform for three years knows why the legacy base image exists. A reviewer in their second week does not, and approves. The control's strength is a property of a person's tenure, which is not a property a control should have.

**Approval can be laundered by volume.** If the agent produces forty pull requests a week and thirty-eight are correct, the reviewer's prior shifts. The two that matter get the same attention as the thirty-eight that do not, and the process has converted a review into a sampling exercise without anyone deciding to.

The remedy is not a better reviewer. It is to place the control *before* the agent's act — scoped credentials, plan-only authority, branch protection that requires a human approver distinct from the author, and an append-only log — so that the review is a confirmation of a bounded proposal rather than the only thing standing between an agent and the estate. §7 is that containment, and [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) owns the mechanics of enforcing it.

### 5.7 The thesis, applied to the control surface

The control surface is where the thesis is least metaphorical. A pothole in a template is visible: the scaffold fails, the pipeline breaks, someone reports it. A *preference* embedded in the control surface is invisible by construction — a retention period chosen for a regulatory reason, a role narrowed because of an incident three years ago, a base image pinned for a vendor certification. An agent that cannot see the preference will read it as an inefficiency, and the control surface is made almost entirely of preferences that have hardened into configuration. That is why this section's answer is prohibition rather than review: the class of act is one where the agent's information is systematically insufficient, not occasionally insufficient.

---

## 6. The Agent as a Platform Capability

### 6.1 The direction reversed

Everything so far has treated the agent as a participant in the platform team's work. There is a second, legitimate arrangement: the platform team **ships an agent to developers** as one of its capabilities — a portal assistant that answers "which template should I use", a scaffolding chatbot that turns an intent into a pull request, a catalogue question-answering agent that tells a developer who owns a service and what depends on it. This is sense (b) in production: the agent is a platform artefact, and the platform team is its product owner.

The distinction from §4 is worth stating. In §4 the agent acts *on* platform artefacts on the team's behalf. Here the agent acts *for developers*, inside a capability the platform team operates, and the platform team carries the consequences of its answers. The governance question changes from "may the agent change this" to "what may this agent say and reach, and who is answerable when it is wrong".

### 6.2 What shipping an agent demands of the platform's own governance

| Demand | What it means concretely | Why the platform team cannot skip it |
|---|---|---|
| **Entitlement scoping** | The agent's access is the intersection of the requesting developer's entitlements and the agent's own; it never reads with the platform team's credentials on a developer's behalf | Otherwise the assistant is a privilege-escalation path with a friendly interface — it can show a developer a catalogue entry, a cost line or a repository they are not entitled to see |
| **The catalogue as grounding** | Ownership, dependencies and service metadata come from the catalogue, not from the model's recollection; answers cite the entry | An assistant that invents an owner is worse than no assistant, because the developer acts on it |
| **Eval before release** | A defined question set with known-correct answers, run before each change to the prompt, the model or the tools; failures block release | The platform team's own release discipline, applied to itself |
| **A catalogued entity** | The agent is registered with an owner, a version, a description of its mandate and its tool surface, exactly like any other platform service | An agent nobody owns is an agent nobody can retire, audit or fix |

The fourth demand is the one platform teams most often miss, and it is the one their own canon should have taught them. The catalogue is the ownership graph ([../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §3.8, and the Backstage software-catalogue pattern in its §2.2); an agent that answers questions about ownership while having no catalogue entry of its own is a contradiction the estate will eventually notice. Versioning of the agent itself is owned by [agent_versioning_guide.md](agent_versioning_guide.md).

### 6.3 Both sides of its own gate

The structural point of this section is uncomfortable and worth stating plainly. When the platform team ships an agent as a capability, it becomes both the **producer** of a governed artefact and the **governor** of it. The team that decides what "evaluated" means is the team whose agent is being evaluated. The team that owns the catalogue is the team whose agent needs a catalogue entry. The team that enforces entitlement scoping is the team whose agent's scoping is being checked.

That is not a reason to refuse the arrangement — platform teams have always shipped their own services through their own gates, and the alternative (an external governance function that does not understand the platform) is worse. It is a reason to make the gate *legible*: publish the eval set, publish the version, publish the owner, and let a party outside the platform team see the evidence. The failure mode when this is skipped is not dramatic; it is a capability that quietly exempts itself, and §13.4 names it.

### 6.4 What this section does not re-derive

The prompt-to-production path — sandbox, data, eval, register, serve, observe — is [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §6, and the platform's service-catalogue machinery is its §3.8. Agent construction is [agentic_engineering_guide.md](agentic_engineering_guide.md); operating it in production is [agentops_guide.md](agentops_guide.md) and [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md). This section adds one thing to that stack: the observation that when the platform team is the agent's owner, the gate has an inside, and the inside needs a window.

### 6.5 Three questions before shipping

A platform team about to ship an agent as a capability should be able to answer three questions in writing, and a "no" to any of them is a reason to wait rather than a reason to add a control.

**What does it answer from, and what happens when that source is silent?** An assistant grounded in the catalogue can say "this entity has no owner recorded", which is a useful and truthful answer. An assistant grounded in a model's recollection will supply an owner, confidently, and be wrong. The grounding decision is the capability's quality decision.

**Whose entitlements does it use?** If the answer is "the platform team's", the capability is a privilege-escalation path and no amount of prompt instruction fixes it, because the model is not the thing enforcing the boundary. The answer must be the requesting developer's entitlements intersected with the agent's own, enforced at the tool layer.

**Who retires it?** An assistant that developers rely on has become part of the platform's surface, and its retirement is a platform change with users. The owner named in the catalogue entry is the person who answers that question, and an agent without one cannot be retired, only abandoned.

### 6.6 What the platform team learns from being on both sides

There is an unexpected benefit to the arrangement, and it is worth recording because it is the strongest argument for shipping an agent internally before deploying one against the platform's own artefacts.

A platform team that has operated an agent for developers learns, at first hand, what it is like to own something whose behaviour is probabilistic, whose failures are quiet, and whose users cannot debug it. That is precisely the experience its own agent-as-colleague programme will impose on the teams downstream of the platform. A team that has run the eval set, watched a confident wrong answer reach a developer, and had to explain it, writes a better mandate for the agent that touches the template — not because it has become more cautious, but because it has acquired the vocabulary for the failure it is trying to prevent.

---

## 7. The Guardrails and the Blast Radius

### 7.1 What an agent can break

Blast radius is the right frame for platform work because the platform's artefacts propagate. The table orders action classes by the population they can reach, not by how difficult the action is to perform.

| Action class | Worst realistic outcome | Containment that must exist first |
|---|---|---|
| **Applying infrastructure-as-code to a shared account or cluster** | Account-wide or cluster-wide destruction: a state file replaced rather than updated, a resource deleted and recreated with a new identity, a network boundary removed | Plan-only authority for the agent; apply reserved to a human or a pipeline with an approval gate; state locking and backups; non-production environment first |
| **Editing a shared template** | Every service scaffolded from it inherits the defect, including services that do not exist yet; the defect surfaces months later in a team that never saw the change | Template changes routed through the platform product owner; a render test on the template itself; branch protection requiring a reviewer who is not the author |
| **Changing a default** | Silent propagation: nothing fails, and the estate's behaviour shifts for every future service; detection comes from an audit, not an alarm | Defaults treated as policy; a diff review that names the population; a changelog entry that reaches developers |
| **Touching identity or permissions** | Lateral reach: a role that more actors can assume, a boundary that fewer things respect, a credential that outlives its purpose | The agent has no write path to identity at all; permission changes are a separate change class with a separate approver |
| **Editing the CI layer** | Supply-chain reach: every build that runs the pipeline, including builds of services the agent has never seen; a compromised or misconfigured step executes with the pipeline's credentials | Pipeline definitions owned by a named engineer; required review; no agent write access to the pipeline's own credentials; provenance recorded per build |
| **Reading the catalogue** | Lower severity but real: the catalogue is a map of tenancy, ownership and dependency; a broad read entitlement leaks the organisation's structure | Read scope limited to the requesting principal's entitlements; access logged; no bulk export |
| **Running a migration batch across repositories** | Many correct changes and a few wrong ones, merged at speed, with the wrong ones indistinguishable from the right ones in the queue | Per-repository pull requests; a concurrency limit; a hold on non-conforming repositories; a human merge per repository or per approved batch |

### 7.2 The containment, stated as requirements

The right-hand column above is not a wish list; each item is a precondition, and the test of a platform's readiness is whether the agent's authority *fails closed* when the precondition is absent.

- **Non-production first.** The agent's write path reaches a development estate before it reaches anything shared, and promotion is a separate act.
- **Plan-only until a human applies.** The agent's output is a plan — reviewable, diffable, storable — and the apply is a different credential held by a different actor.
- **Scoped credentials per action class.** The agent that drafts migrations cannot read identity; the agent that triages tickets cannot write repositories. Scope is per class, not per agent.
- **No standing production write.** Production write is granted for a window, for a change, and expires.
- **Branch protection and required review.** The agent's pull request cannot be merged by the agent, and the required reviewer is not the author.
- **An append-only action log.** Every tool call, every proposal, every read of a sensitive surface, retained where the agent cannot edit it.

The mechanics — how a sandbox is built, what an isolation boundary actually enforces, where the credential boundary sits — are [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md)'s subject, and the posture vocabulary (ask, recommend, act, wait) is [agent_decision_policy_guide.md](agent_decision_policy_guide.md)'s. This section's contribution is the ordering: the containment is selected by the *population an action class can reach*, which is a platform fact, not a model fact.

### 7.3 The asymmetry that drives the ordering

Two failures of equal technical severity are not equal in a platform. A bad change to one service is an incident with a boundary: the service is known, its owner is known, the rollback is known. A bad change to a default is an incident without a boundary: the population is "everything scaffolded since", the owner is "the platform team", and the rollback requires finding every inheritor. The blast-radius table is ordered by that asymmetry, and an agent's authority should be ordered the same way — narrowest where the population is least enumerable.

### 7.4 What the containment costs

None of the containment in §7.2 is free, and a guide that presented it as free would be selling something. Three costs are real and should be budgeted.

**Latency.** Plan-only authority means a human applies, and a human applies during human hours. A migration that an agent could complete in an afternoon completes in a week of review windows. For a platform team whose constraint is capacity, this looks like the problem getting worse before it gets better — and it is, for the first change. The second change is cheaper, because the plan template, the review pattern and the approval path already exist.

**False blocks.** Scoped credentials fail closed, which means they sometimes block an action that was legitimate. An agent that cannot read identity cannot answer a legitimate question about a role; a plan-only agent cannot fix a drift that is genuinely urgent. Each false block is a request for an exception, and a programme that treats every exception as a scope widening will end with the containment removed — which is §13.2 with better intentions.

**Reviewer load.** The containment moves work from the agent to the reviewer rather than eliminating it, and §2.1's constraint is the reviewer. A team that adopts containment without reducing the review burden some other way — smaller batches, a category of change pre-approved by class, a reviewer rotation — has made its bottleneck worse and will be tempted to remove the control. This is the honest tension in the whole design: the control that makes the agent safe is a control that consumes the resource the agent was meant to free.

---

## 8. The Metrics Question

### 8.1 What could be measured

An honest measurement programme for agent-assisted platform work starts with instruments that describe the work, not the agent's contribution to it.

| Metric | What it describes | What it cannot show |
|---|---|---|
| **Ticket cycle time** | Elapsed time from ticket open to close, by class | Whether the ticket needed to exist; whether closure meant resolution |
| **Time to first environment** | How long a developer waits for a working environment from request | Whether the environment was the one they needed |
| **Template drift rate** | Share of services whose configuration diverges from the template's current defaults | Whether the divergence is debt or a deliberate exception |
| **Review turnaround** | Time a platform pull request waits for a human reviewer | Whether the review was substantive |
| **Rework rate** | Share of agent-proposed changes that were reopened, reverted or rewritten after merge | The changes that were wrong and never noticed |
| **Exception volume** | How many cases the agent routed to a human as non-conforming | Whether the routing rule was right |

The rework rate and the exception volume are the two instruments that matter most, and they matter for the same reason: they are the only two that can see the agent's *judgement* rather than its throughput. A high rework rate is a defect in the agent. A persistently near-zero exception volume is a defect in the routing rule — it means the agent is either very good or is not looking, and the second is more likely.

### 8.2 What the platform literature measures

DORA's delivery metrics are the platform discipline's shared instrument, and the framing has moved. As of the guide page updated 5 January 2026, DORA describes **five** metrics — change lead time, deployment frequency, failed deployment recovery time, change fail rate, and deployment rework rate — and the page explicitly notes the shift from mean time to repair to failed deployment recovery time, linking a history of the metrics (DORA, "DORA's software delivery performance metrics", dora.dev, guide by Nathen Harvey, page updated 2026-01-05). The commonly cited "four keys" framing is therefore dated. This matters for any agent programme that proposes to be measured on delivery metrics: the instrument set changed, and a dashboard built on the old four is measuring a superseded model.

The second piece of literature matters more, and it is a warning about the whole measurement exercise. The CNCF Platforms White Paper states that platforms "significantly impact enterprises' actual value streams, but only indirectly" (CNCF TAG App Delivery, Platforms White Paper, whitepaper, read 2026-10-19). Platform value arrives through the teams that use the platform, mediated by their own decisions, their own roadmaps and their own competence. An agent's contribution sits one layer further out: it changes the platform team's output, which changes the platform, which changes what teams can do, which changes delivery. Each link attenuates and confounds the last.

### 8.3 Why attribution fails

The attribution problem is not a data-quality problem that better instrumentation would solve. It is structural, for three reasons.

**The platform is a shared input.** Every team consumes the same platform, so a change in one team's delivery cannot be attributed to a platform change without a comparison group — and there is no comparison group, because the platform is not randomised and the teams that adopt early are not a random sample.

**The confounders are the platform team's own roadmap.** The quarters in which an agent is introduced are the quarters in which the platform team is also shipping the things the agent was hired to free capacity for. The agent's effect and the roadmap's effect are entangled by construction.

**The counterfactual is a ticket that was never filed.** Much of the value of a faster platform is work that did not have to wait — a team that self-served instead of queueing, a service that shipped a week earlier. That value has no timestamp and no record, which is why the metrics in §8.1 measure the queue rather than the absence of one.

There is a further reason to be careful, and it comes from DORA's own model rather than from the platform literature. The DORA AI Capabilities Model (dora.dev/ai, model published 2025, page read 2026-10-19) lists the capabilities that mediate AI adoption outcomes, and among them are a **quality internal platform**, **AI-accessible internal data**, strong version control practices, working in small batches, a clear and communicated AI stance, user-centric focus and healthy data ecosystems. The implication is worth stating slowly: the platform's quality is modelled as a **precondition** for AI outcomes, not as a consequence of them. An agent programme that reports platform improvement as its benefit has the causal arrow pointing the wrong way relative to the model the industry uses to reason about exactly this question.

### 8.4 What to report instead

The defensible reporting is descriptive and bounded: what the agent was permitted to do, what it produced, what humans did with it, and what the exception flow looked like. Four numbers, all auditable, none of them a productivity claim — proposals opened, proposals merged, proposals reworked, exceptions routed. A platform team that reports those four and refuses to convert them into a percentage is making a stronger claim than one that reports a percentage, because it is reporting evidence rather than an attribution.

### 8.5 The one comparison that is worth making

There is a temptation to compare the agent-assisted period with the period before it, and the comparison is almost always uninterpretable for the three reasons in §8.3. There is one comparison that is worth making, and it is internal to the work rather than to the calendar.

**Compare the agent's proposals against the platform team's own prior proposals on the same class of change.** If the team migrated a base image two years ago by hand, the rework rate of that migration and the rework rate of the agent-assisted one are comparable in a way that no before-and-after productivity number is, because the class of change, the estate and the review process are the same. The comparison is still imperfect — the estate has aged, the team has learned — but it is a like-for-like on the dimension that matters: how often a proposed change in this class had to be redone.

That is the shape of every honest measurement in this subject. It compares artefacts, not eras; it holds the class of work constant; and it reports a rate that a reviewer can check rather than a claim about the platform team's value. It will not produce a headline, which is the point.

---

## 9. The Tooling Landscape, Structurally

### 9.1 Three categories and what each does in this argument

The tooling question is usually asked as a product question. It is more usefully asked as a structural one: which layer of the platform does an agent's action land in, and what does that layer's design permit? Three categories carry almost all of the subject. Named projects appear below **as instances of a category, not as a ranking**; no product is compared with another by quality, and the absence of a name is not a judgement.

| Category | What it owns in this argument | What an agent may do in it | Instances, named as instances only |
|---|---|---|---|
| **Developer portals and service catalogues** | The front door and the ownership graph: what exists, who owns it, what it depends on, what template it came from | Read ownership and dependency data within entitlement; propose catalogue entries for new services; draft documentation attached to an entry | Backstage (open-source framework for building developer portals, created by Spotify and open-sourced in 2020, a CNCF Incubation project since March 2022) is the instance used here to make the category concrete; other portals exist and none is ranked. |
| **Infrastructure-as-code and its plan/apply split** | The mechanism that makes an agent's proposal reviewable: a plan is a description, an apply is an effect | Produce a plan; propose a diff; never apply | The major provisioning tools — declarative, stateful, plan-then-apply — share the split. The category, not the product, is the argument. |
| **The CI layer** | Where policy is enforced and where a change either passes or does not: builds, tests, admission checks, provenance | Read a pipeline's result; propose a change to a pipeline definition; never hold the pipeline's own credentials | Continuous-integration systems and the policy engines that gate them. The category is what matters; the products are interchangeable instances. |

### 9.2 The portal and the ownership graph

Backstage is a useful instance to reason with because its three documented parts map onto the three things an agent needs. The software catalogue is the **ownership graph** — it answers "who owns this, what depends on it, which template produced it". The software templates are the **scaffolding mechanism** — the documented purpose is spinning up new projects and standardising tooling, which is §4.2's activity in product form. TechDocs is **documentation as code** — docs live with the service and are built like it, which is what makes §4.5's drafting reviewable at all (Backstage documentation, "What is Backstage?", project documentation, read 2026-10-19).

The structural point is not that a portal is useful. It is that the catalogue is the only place where the *population* of a platform change is knowable. A template change's blast radius is "every entity whose catalogue entry names this template"; a default's blast radius is the set of entries with the corresponding annotation. Without a catalogue, an agent's change has no enumerable population and a reviewer has nothing to check against. With one, §7's containment becomes implementable rather than aspirational. This is also why [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §3.8 treats the catalogue as a first-class platform service rather than a portal feature.

### 9.3 The plan/apply split is the safety mechanism

Of the three categories, infrastructure-as-code is the one whose *design* — not its product — is load-bearing for this guide. The split between a plan and an apply is the closest thing the platform world has to a native "propose, then commit" primitive, and it maps onto §4's artefact/effect distinction with unusual precision:

- **A plan is a reviewable artefact.** It enumerates what would be created, changed and destroyed, before anything is. An agent can produce one without any write authority at all.
- **An apply is an effect.** It changes state, and the state it changes is often shared.
- **The boundary between them is a credential boundary**, not a convention — which is what makes it enforceable rather than advisory.

An agent architecture that respects the split gets most of §7's containment for free: plan-only authority is a real restriction, not a promise. An architecture that collapses the split — an agent holding apply rights and "reviewing" its own plan — has no containment at all, whatever the prompt says. §5.6's four reasons apply with full force there, because the reviewer is the author.

### 9.4 The CI layer is where policy stops being documentation

The third category is where a platform's rules become executable. A policy that lives in a wiki is advice; the same policy expressed as a pipeline check or an admission gate is a control. For this guide the consequence is directional: **an agent's change is only as governed as the CI layer it must pass.** If the pipeline checks that every service has an owner tag, the agent cannot merge an orphan. If it does not, no amount of instruction will stop it.

This is also the layer where the supply-chain reach of §7.1 lives, and it deserves the emphasis: an agent with write access to pipeline definitions has reach over every build that runs them, which is a larger population than any template. The category's own logic — least privilege, pinned dependencies, provenance — is exactly the containment the agent needs, and it is available in the ecosystem rather than something a team must invent.

### 9.5 What the categories do not settle

Three things are outside all three categories, and a tooling decision will not supply them.

**The exception list.** No portal, provisioning tool or CI system knows that a repository is non-standard *on purpose*. That knowledge is organisational, and §12.3 makes it an artefact the platform team maintains.

**The authority mapping.** Which human may approve which class of change is an institutional decision. Tools implement an approval; they do not decide who should hold it.

**The retirement path.** An agent that has been scaffolding, triaging and documenting for a year has accumulated a role. No category above provides for its retirement, which is why §6.5 makes "who retires it" one of the three questions a platform team answers before shipping, and why the owner field in the catalogue entry is the answer.

---

## 10. Banking and Regulated Change Control

### 10.1 The question the change board will ask

A bank's change board has one question that matters more than the rest: **who approved this, and on what evidence?** The question is not hostile and it is not about technology. It is the institution asking whether it can be answerable for a change, and the answer must name a person.

An agent-authored platform change is not outside that frame; it is inside it with one term missing by default. The change happened, the diff exists, the pipeline passed — and nobody can be asked why. A programme that cannot answer the question will either be refused at the board or, worse, be approved on the strength of a demonstration and then fail the first audit that asks for the record.

### 10.2 Four eyes with an agent as maker

The maker-checker principle is old and simple: for a consequential transaction, at least two individuals participate — one creates, another confirms. An agent can be a legitimate **maker**. It cannot be a **checker**, and it cannot be both.

Two failure shapes follow, and both are common enough to name. The first is the agent as maker and checker — an agent drafts a change and its own evaluation approves it. That is one eye wearing two hats, and it is a control failure regardless of how good the evaluation is. The second is subtler: the agent as maker, a human as checker, but the human's approval is produced by a process the agent also wrote — the pipeline it configured, the policy it edited, the checklist it drafted. The second eye is human and the *frame* it looks through is the agent's. §5.4's guardrail change is the clean example: an agent that narrows a policy and then passes the gate that policy controls has approved itself with extra steps.

The rule that follows is narrow and enforceable: **for any change to the control surface, the checker's frame must be independent of the maker.** In practice that means the approval gate's definition is not editable by the agent, the policy it enforces is versioned outside the agent's write scope, and the reviewer can see the previous policy alongside the proposed one.

### 10.3 The evidence pack

A change board that asks the right question needs an answer in a fixed shape. Six items, each an artefact rather than an assurance.

| # | Item | What it establishes | Where it comes from |
|---|---|---|---|
| 1 | **A named accountable human** | Who is answerable; the person the board can ask | The approval record, not the agent's transcript |
| 2 | **The policy version applied** | Which rules were in force when the decision was taken | The policy store's version identifier |
| 3 | **The diff** | What actually changed, in reviewable form | The pull request |
| 4 | **The plan output** | What the change would do to the estate, enumerated before it did it | The provisioning tool's plan, retained |
| 5 | **The evidence the review happened** | That a human read it, not merely that a box was ticked | Review comments, approval identity, timestamps |
| 6 | **The record of what the agent was asked and with which tools** | The mandate and the tool surface at the time of the act | The action log, append-only |

Item 6 is the one institutions most often lack, and it is the one that makes the other five interpretable. Without it, the board knows a human approved a diff but not what the human was approving *as* — a proposal from an agent with repository read and pull-request write is a different artefact from a proposal from an agent with cluster read and pipeline write, even when the diffs are identical.

### 10.4 The regulatory floor

In the European Union, Regulation (EU) 2022/2554 — the Digital Operational Resilience Act — has applied since 17 January 2025 and requires financial entities to maintain ICT risk-management and ICT change-management policies (regulation, 2022, applicable 2025). The obligation is not about agents, and this guide does not claim it addresses them. What it does is establish that change management is a documented, auditable institutional obligation rather than an engineering habit — which is the floor a change board operates above, and the reason the evidence pack is not optional in a bank.

The corresponding internal references are [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) for the controls-and-evidence discipline and [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) for the wider obligations; the authority framing — who may commit the institution to what — is [agent_decision_policy_guide.md](agent_decision_policy_guide.md) §9.

### 10.5 The asymmetry that decides the design

One observation organises this whole section, and it is worth stating as a rule:

**An agent that drafts a change is evidence-generating. An agent that approves a change is evidence-destroying.**

A drafting agent produces a diff, a plan, a rationale, a list of repositories examined, a list of exceptions routed. All of it is additive: it makes the institution's record richer, and a human's approval on top of it is better evidenced than the same approval would have been without it. An approving agent produces a signature whose author cannot be examined. The record then contains an approval and no approver, and the institution's answer to "who approved this" becomes "the system", which is not an answer a board can accept.

The asymmetry is why §4's rulings are asymmetric in the same direction, and why the boundary is drawn at the signature rather than at the diff.

### 10.6 What the board should refuse

Four requests should be refused regardless of how good the demonstration was, and the refusals are worth writing down because each will be asked for.

An agent holding apply rights in production "with guardrails". An agent approving changes in a class it also drafts. An agent whose action log is writable by the agent. And an exception process that exists only in the agent's routing logic — because an exception the platform cannot enumerate is an exception nobody can audit, which §12 makes the central design decision of the worked example.

---

## 11. The Failure Modes

Each failure is stated as **symptom → cause → guardrail**, and each is a version of the thesis failing: an agent that could not distinguish a preference from a defect, or a platform team that could not see what its agent had made.

### 11.1 The agent that paves a road twice

- **Symptom.** A second template, module or pipeline appears alongside the existing one. It is better in some respect — simpler, more current, better suited to one team's case — and it works. Six months later the estate has two routes, two support burdens and no answer to "which one should I use".
- **Cause.** The existing path had a friction the agent read as a defect and fixed by building a parallel path rather than by changing the one that exists. The agent's mandate was "make this work", and a second road made it work.
- **Guardrail.** The agent may propose changes to the existing path and may not create a competing one; template creation is a platform product decision with an owner; the catalogue records which template produced which service, so a second path is visible as a split rather than discovered as a surprise.

### 11.2 The agent that fixes a convention it cannot see

- **Symptom.** A repository that was deliberately non-standard is normalised. The change is technically correct, the build passes, and something downstream breaks — a certification lapses, an evidence chain is broken, a team's migration is derailed.
- **Cause.** The agent's information was the repository, and the reason for the divergence was not in the repository. The convention looked like drift because drift is what it usually is.
- **Guardrail.** Non-conformance is a **routing signal, not a work item** (§12.3): the agent lists it with evidence and routes it to the platform product owner. The exception list is maintained as an artefact, and the agent's mandate explicitly excludes repositories on it.

### 11.3 The agent that scaffolds a service nobody owns

- **Symptom.** Services exist that no team claims. They run, they cost money, they appear in no on-call rota, and when they break nobody is paged. The paved road has produced orphans efficiently.
- **Cause.** Scaffolding was treated as file generation. The template rendered, the pipeline passed, and the ownership question — which is a human commitment, not a field — was never asked. An agent cannot make a team own something.
- **Guardrail.** The scaffold is not complete until a named owner accepts it; the catalogue entry requires an owner before the service is considered live; the accepting human is the service owner in §4.1's table, and the pull request's approval is that acceptance.

### 11.4 The agent whose changes are correct and organisationally wrong

- **Symptom.** A large, correct migration lands during a change freeze, or across a team boundary that was being renegotiated, or the week a team is mid-release. Every diff is right; the timing and the boundary are wrong.
- **Cause.** The agent has repository context and no organisational context. Freeze windows, team boundaries and release calendars are not artefacts it reads, and correctness at the file level is the only correctness it can check.
- **Guardrail.** Volume is throttled by design (a concurrency limit, a batch approval); freeze windows are an input to the agent's mandate rather than a hope; and the platform engineer who authors the migration owns its schedule, which is a human judgement about timing that no diff review supplies.

### 11.5 The agent that becomes the only documentation

- **Symptom.** The platform is described only in the agent's summaries. Ask a new engineer how the template system works and the answer is "ask the agent". When the agent is retired, replaced or has an outage, the platform's self-description goes with it.
- **Cause.** The agent was faster than the documentation and the documentation was allowed to decay, because the summaries were good enough in the moment. The knowledge never landed in an artefact a human owns.
- **Guardrail.** The agent writes **into** the documentation, not instead of it: every summary it produces for a human is a draft destined for a page, a runbook or a decision record, reviewed and committed. The test is whether the platform can be understood with the agent switched off — if not, the agent is a single point of knowledge failure dressed as an efficiency.

### 11.6 The five in one table

| # | Failure | The preference it could not see | The artefact that fixes it |
|---|---|---|---|
| 11.1 | Paves a road twice | That the existing path is the product, and a second one divides it | The catalogue's template provenance, plus a rule that path creation is a product decision |
| 11.2 | Fixes a convention it cannot see | The reason for the divergence: a certification, an evidence chain, a migration in flight | The exception list, owned and dated |
| 11.3 | Scaffolds a service nobody owns | That ownership is a human commitment, not a rendered field | An owner requirement in the catalogue before the service is live |
| 11.4 | Correct diff, wrong organisation | The freeze window, the team boundary, the release calendar | A throttled schedule owned by the authoring engineer |
| 11.5 | Becomes the only documentation | That the platform must be describable with the agent switched off | A documentation destination every summary lands in |

### 11.7 What the five have in common

Four of the five are failures of *seeing*: a population the agent could not enumerate (11.1, 11.3), a reason it could not read (11.2), a calendar it could not consult (11.4). The fifth is a failure of *landing*: knowledge that stayed in the agent (11.5). None is a model-capability failure in the narrow sense, and none is fixed by a better model. All five are fixed by an artefact the platform team builds — a catalogue, an exception list, an ownership requirement, a schedule, a documentation destination — which is the practical form of the thesis: the agent can pave, and the platform team must supply the map.

---

## 12. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is a fictional institution used across this repository as the worked-example persona, and it is the only institution in this guide. The repository counts, the team names and the dates below are invented scenario detail, chosen to be plausible; they are not measurements, benchmarks, vendor figures or regulatory requirements, and nothing here should be cited as evidence. "In this illustration" governs every number in the section.

### 12.1 The situation

Cymbal Bank's platform team runs a base-image standard: every service container is built from a maintained image family, and the current major version is being retired. In this illustration the estate is about **120 service repositories**, owned by roughly twenty stream-aligned teams, all scaffolded from the platform's own template at various points over four years.

The platform team deploys an agent into **one** part of its work: drafting the migration. The agent inventories the estate, proposes a base-image change per repository, opens a pull request per repository, and runs each repository's build to confirm the change compiles. It does not merge, it does not apply, and it does not touch the template. The platform team reviews. The mandate is deliberately narrow — one activity from §4.3, one artefact class (pull requests), one change type (the base image line).

### 12.2 What the agent found

The inventory returns what an inventory always returns: a majority that match the house pattern and a minority that do not. In this illustration, **11 of the 120 repositories do not follow the standard Dockerfile pattern** — a different base declaration, an extra build stage, a hand-maintained layer ordering. The agent's proposal is straightforward and, on its own terms, correct: normalise the 11 to the standard pattern, then apply the version bump uniformly.

This is the moment the thesis is about, and it is worth slowing down to see why. From inside the repository, the 11 look like drift. The house pattern is documented, the template encodes it, and the divergence has no comment explaining itself in most cases. An agent optimising for "every service on the supported base image, built the standard way" has found 11 defects and proposed the obvious fix.

### 12.3 The decision the platform-engineering context changed

A platform engineer reviews the 11 and finds that **four of them are deliberate**, each for a different reason:

- **One service builds on a legacy base for a vendor-certification reason.** The certification is attached to that base and to nothing else; moving it invalidates the evidence the vendor's attestation depends on.
- **One has a regulatory evidence requirement pinned to its build provenance.** The build's layer structure is part of what the evidence chain records; normalising it would change the artefact the evidence describes.
- **Two are owned by a team that is mid-migration** to a different runtime, and normalising them now would collide with work in flight and be redone within the quarter.

The remaining seven are genuine drift, and the agent's proposal is right about them.

The decision the team takes is not "review the 11 more carefully". It is a change to the **mandate**, and it has three parts:

1. The agent's authority is narrowed to **proposing the standard change to conforming repositories** — the 109 that match the pattern. That is the work it can do correctly and at volume.
2. For the non-conforming repositories, the agent's job becomes **enumeration with evidence**: list them, state what differs, attach the build result, and **route them to the platform product owner** rather than proposing a normalisation.
3. The exception list becomes a **first-class artefact** — owned, dated, with a reason per entry and a review date — rather than a note in a review thread.

The third part is the one that makes the other two durable. An exception list that exists only in a conversation is re-discovered every quarter, and each re-discovery costs a review; an exception list that exists as an artefact is an input to the next migration, the next agent's mandate, and the next auditor's question. It also changes what the agent is *for*: not "make everything conform", but "conform the conformable and make the exceptions legible".

### 12.4 Why this is the thesis and not a compromise

It would be easy to read the decision as a retreat — the agent was going to fix 120 and was talked down to 109. That reading misses what happened.

The agent could not tell a pothole from a preference. The 109 were potholes: the standard applied, the build was the build, the change was the change. The four were preferences, and each preference was a decision the organisation had made for a reason that lived outside the repository — a certification, an evidence chain, a migration in flight. The platform team did not narrow the mandate because the agent was unreliable. It narrowed the mandate because **the mandate was the only place the distinction could be expressed.** The agent's capability was never the question; the question was whether the platform had a way to say "these four are not defects", and the answer had to be an artefact.

The decision also has a cost that should be named, because the thesis is not free. The exception list is now maintained by a human, reviewed on a cadence, and wrong whenever the organisation changes without updating it. §16 records the honest limit: this guide has no evidence on how often such lists go stale, and the failure mode when they do is that a genuine pothole is preserved as a preference. The alternative — no list, agent decides — is worse, but the list is a human artifact with human decay.

### 12.5 The change board

The migration goes to Cymbal Bank's change board as one change, not 109.

- **Who approved.** The platform engineer who authored the migration is the accountable human. The platform product owner countersigns the exception list as a separate artefact. The board's record names both, and neither is the agent.
- **What evidence was presented.** The six items of §10.3: the named approvers; the policy version of the base-image standard in force at approval; the diff, represented by the migration's canonical change plus the per-repository pull requests; the plan output for the pipeline change that accompanies the version bump; the review record — approvals per repository or per approved batch, with the reviewer's identity; and the action log of what the agent was asked, which repositories it examined, and which it routed as exceptions.
- **What the board asked.** Two questions, both anticipated by §10: whether the agent could merge (it could not — branch protection required a human approver distinct from the author), and whether the seven drift repositories had been verified as drift rather than assumed (they had, by the platform engineer, and the verification is in the review record rather than in the agent's summary).
- **What the board recorded.** A named human accountable for the migration, a named human accountable for the exceptions, the policy version, and the fact that the agent's role was proposal and enumeration. The record does not say "the agent migrated the estate". It says who did, and what the agent produced.

### 12.6 What the example shows

Four things, in order of how much they generalise.

The **inventory is the easy part and the exception is the hard part**. An agent that can enumerate 120 repositories and propose 109 changes has done most of the labour and none of the judgement, and the judgement is what the platform team is for.

The **mandate is the interface between the agent's capability and the organisation's knowledge**. Narrowing it is not a restriction on the agent; it is the mechanism by which the organisation's knowledge enters the agent's behaviour at all.

The **exception list is a platform artefact**, on the same footing as the template and the pipeline. It has an owner, a version and a review date, and it is the thing that lets the next agent be more capable without being more dangerous.

And the **change board is satisfied by evidence, not by assurance**. Nothing in the pack says the agent is trustworthy. Everything in it says what the agent did, what a human decided, and which policy was in force.

Which is the thesis, restated where it was tested: **The platform team's product is a paved road; an agent that paves it must be able to tell a pothole from a preference.**

---

## 13. The Anti-Patterns

Eight, each named for the shape of the failure rather than the technology. They are not independent: most are one of §11's failure modes adopted as a policy.

### 13.1 The agent as the platform's ghostwriter

The agent writes the runbooks, the migration notes, the architecture summaries and the change descriptions, and nobody reads them. The artefacts exist, the repository looks maintained, and the platform's actual self-description lives in the heads of two engineers. The tell is that when the agent is unavailable, the documentation is not consulted — it is waited out. The guardrail is to treat every agent-written page as a draft with a human destination, and to measure whether humans edit it rather than whether it exists.

### 13.2 The agent that owns the template

A team concludes that reviewing every template change is the bottleneck and grants the agent template write access "for small changes". The definition of small is the agent's. Within a quarter the template has accumulated changes nobody can explain, each individually reasonable, and the platform's contract with its users has been rewritten a line at a time. The anti-pattern is not the delegation; it is the absence of a population-aware review. Ownership of the template is a product decision, and product decisions do not have a small-change exemption.

### 13.3 The auto-merge pilot

A pilot in which the agent's pull requests merge automatically once checks pass, justified by a bounded scope and a rollback plan. It works, which is the problem: the pilot's success is measured on the changes it merged, and its risk is the change it merged that passed the checks because the checks did not cover the thing that broke. The guardrail is structural rather than procedural — a pilot's merge authority is a credential, and the credential is what has to be justified, not the pilot's results.

### 13.4 The eval-free rollout

The platform team ships a portal assistant or a scaffolding agent without an eval set, on the grounds that it is internal and low-risk. It is internal, and it is not low-risk: an assistant that invents an owner sends a developer to the wrong team, and a scaffolder that misreads a parameter produces a service with the wrong retention. The platform team's own release discipline — evaluate before release, version the artefact, record the result — is the minimum, and skipping it is how a platform team exempts itself from the gate it operates for everyone else.

### 13.5 The agent as the only reviewer of its own work

An agent drafts a change and a second agent, or the same agent in a different role, reviews it. The review is real in the sense that it produces comments and sometimes catches errors; it is not a review in the sense the control requires, because both eyes share a frame. The pattern is attractive because it is cheap and because the second pass genuinely improves quality. It is a defence in depth, not a second eye, and the distinction is the whole of §10.2.

### 13.6 The ticket-closing metric

The platform team reports tickets closed, and the agent closes tickets. Volume rises, the queue shortens, and the number becomes the programme's justification. What it measures is the agent's throughput against a queue whose contents nobody re-examined — including the tickets that should have been prevented by a template change, the tickets that were questions the documentation should have answered, and the tickets closed as resolved that were not. The guardrail is to pair every closure metric with an exception-volume metric and a rework metric (§8.1), so that the queue's composition is visible alongside its length.

### 13.7 The platform agent nobody can audit

The agent has been running for a year, it works, and no one can reconstruct what it did last Tuesday: the log is the harness's default retention, the proposals were merged and the branches deleted, the prompt and tool surface have changed three times with no record of when. The anti-pattern is not a failure of the agent; it is a failure to treat the agent as a platform service with the record-keeping a platform service requires. It becomes acute at exactly the moment it matters — an incident review, an audit, or a decision to widen the mandate.

### 13.8 The demo-driven mandate

A capability is demonstrated on a stage: the agent reads a repository, proposes a correct migration, and the room is impressed. The mandate is written from the demonstration, which showed what the agent can do on a repository chosen because it was a good demonstration. Nothing in the room establishes what the agent does on the repository chosen because it is awkward — the one with the certification, the one whose build is pinned, the one nobody has touched in two years. The anti-pattern is not the demonstration; it is letting the demonstration stand in for the inventory. The guardrail is procedural and cheap: before a mandate is written, the agent runs against a sample selected by someone other than the person who built the demo, and the sample includes the repositories that are expected to be exceptions.

### 13.9 What the anti-patterns share

Every one of them is a control replaced by an assurance. The ghostwriter replaces a documentation owner with a belief that the agent writes well. The template owner replaces a product decision with a belief that small changes are safe. The auto-merge pilot replaces a credential boundary with a belief that the checks cover the risk. The eval-free rollout replaces the platform's own gate with a belief that internal means low-risk. The self-review replaces a second eye with a belief that two passes are two eyes. The ticket metric replaces a queue's composition with its length. The unauditable agent replaces a record with a running system. The demo-driven mandate replaces an inventory with a sample.

In each case the assurance is plausible, and in each case it is held by the people who would have to do the work the control requires. That is why the anti-patterns are resistant to argument and amenable to artefacts: an artefact is checkable by someone who does not share the belief.

---

## 14. The State of Practice

### 14.1 What is demonstrably possible

Four capabilities are demonstrated rather than promised, and the demonstration is narrow in a way that matters.

**Per-repository task completion on benchmarks.** SWE-bench introduced 2,294 task instances drawn from real GitHub issues across 12 Python repositories, and at publication the best model resolved "only the simplest issues" (Jimenez et al., arXiv:2310.06770, ICLR 2024; preprint 2023-10-10). Capability has moved since, and the shape of the task has not: a self-contained repository, an issue that describes the desired behaviour, a test suite that decides success.

**Pull-request-shaped work.** Proposing a diff, opening a pull request, running the build, responding to review — this is the form in which agent capability is most usable in a platform team, because it is the form the platform's own controls already understand. §4's rulings are built on it deliberately.

**Documentation drafting.** Summarising a merged change into a runbook section, correcting a stale command, proposing text — bounded, reviewable, and cheap to reject.

**Triage classification.** Labelling, routing and de-duplicating a queue with stable categories and a history to learn from.

### 14.2 What is vendor promise

Three claims appear constantly in vendor material and are not demonstrated in any source read for this guide.

**Autonomous platform ownership** — an agent that maintains the platform's templates, defaults and pipelines without a human owner. No source read shows this working, and §5 argues it should not be attempted, which makes the absence of evidence convenient but not decisive.

**Self-healing infrastructure** — an agent that detects and remediates estate-level faults. Remediation is an apply, and the apply boundary is where §7's containment lives; the demonstration typically stops at the diagnosis, which is the part that was never the hard part.

**"An agent per team"** — the claim that each stream-aligned team can have its own platform agent, with the platform team supplying a runtime. This misreads the direction: an agent per team is sense (a), a workload on the platform, and it demands the tenancy, entitlement and eval machinery of an AI platform rather than the delegation rulings of this guide.

### 14.3 The structural reason for the gap

The gap is not a capability gap that a better model closes. It is a **context gap with a defined shape**: benchmark tasks are self-contained by construction — the repository, the issue and the test are the whole world — and platform work is not. The conventions that decide whether a change is correct are in a certification, a change board's minutes, a team's migration plan, a freeze calendar, a decision taken two years ago by someone who has left. None of it is in the repository, and the repository is what the agent reads.

There is a second, less discussed reason. A benchmark's success criterion is written by the benchmark's author; a platform change's success criterion is written by the organisation, often after the fact. The agent can satisfy the first and be wrong about the second without any error being detectable at the time — which is §11.4, and which is why the honest position is that agents are useful in platform work precisely where the criterion is local and machine-checkable, and unproven everywhere else.

### 14.4 What would count as evidence

It is worth saying what would move this guide's position, because "we need more evidence" is otherwise an unfalsifiable stance.

**A controlled comparison within one estate, on one change class.** Two similar cohorts of repositories, one migrated with agent-drafted proposals and one by hand, with rework, incident and time-to-complete measured for both. Imperfect, but it would establish something about the class.

**A published account of a platform agent's exception rate.** Not its success rate — its exception rate, with the exceptions categorised. That single number would tell a reader more about the agent's judgement than any benchmark, because it measures the boundary the thesis is about.

**An incident analysis where an agent-authored platform change was the cause, with the mechanism.** One well-documented case would do more for §5 and §11 than any number of demonstrations, because the failure modes in this guide are currently argued rather than observed.

**A study of reviewers.** Whether a human reviewer of agent-drafted platform changes reviews differently from a reviewer of human-drafted ones — attention, time, override rate — is the human-factors question underneath §5.6, and it has not been studied in this setting.

None of the four exists in the sources read for this guide. Until one does, the honest claim is the narrow one: the agent can produce the artefact, and whether the artefact should be trusted is a question about the platform's knowledge, not the model's capability.

---

## 15. The Claims Audit

Every claim this guide rests on, with its source, its date and its kind. A vendor-community reference page is not a research finding; a benchmark is not a deployment result; a regulation is not an implementation.

| # | Claim | Verdict | Source · kind · date |
|---|---|---|---|
| 1 | An internal developer platform is built by a platform team to build golden paths and enable self-service; it binds tools together in a way that lowers cognitive load on developers without abstracting away context and underlying technologies; developers are internal customers; baseline configurations are enforced across workflows | **Verified** | internaldeveloperplatform.org, "What is an Internal Developer Platform?" — vendor-community reference page, read 2026-10-19 |
| 2 | A platform team is "a grouping of other team types that provide a compelling internal product to accelerate delivery by Stream-aligned teams"; four fundamental team types; three interaction modes (collaboration, X-as-a-Service, facilitation) | **Verified** | teamtopologies.com/key-concepts — authors' site, read 2026-10-19; Skelton & Pais, *Team Topologies*, 2019 (book) |
| 3 | Platforms "curate and present foundational capabilities, frameworks and experiences to facilitate and accelerate the work of internal customers"; stated aims include reducing cognitive load on product teams and improving reliability by dedicating experts | **Verified** | CNCF TAG App Delivery, "Platforms White Paper" — CNCF whitepaper, read 2026-10-19 |
| 4 | Platforms "significantly impact enterprises' actual value streams, but only indirectly" | **Verified** | same CNCF whitepaper, read 2026-10-19 — load-bearing for §8 |
| 5 | Platform capability categories include infrastructure, data, messaging/event, identity/secret, security and artifact storage, with example projects per category | **Verified (used as categories, not as a ranking)** | same CNCF whitepaper, read 2026-10-19 |
| 6 | DORA describes five delivery metrics: change lead time, deployment frequency, failed deployment recovery time, change fail rate, deployment rework rate; the page notes the shift from mean time to repair and links a history of the metrics | **Verified** | DORA, "DORA's software delivery performance metrics" (Nathen Harvey) — dora.dev guide, page updated 2026-01-05 |
| 7 | The "four keys" is the current DORA framing | **Rejected** | Superseded by the five-metric page (row 6); the four-key framing is dated, not wrong when it was written |
| 8 | The capabilities mediating AI adoption outcomes include a quality internal platform, AI-accessible internal data, strong version control practices, working in small batches, a clear and communicated AI stance, user-centric focus and healthy data ecosystems | **Verified** | DORA AI Capabilities Model — dora.dev/ai, model published 2025, page read 2026-10-19 |
| 9 | Platform quality is modelled as a precondition for AI outcomes rather than a consequence of them | **Verified (as the model's implication, not a measured finding)** | same source; the implication is drawn here, not asserted by the page as a causal result |
| 10 | Backstage is "an open source framework for building developer portals", powered by a centralised software catalogue, with software templates for spinning up new projects and standardising tooling, and TechDocs (docs like code); a CNCF Incubation project after graduating from Sandbox (March 2022); created by Spotify, open-sourced 2020 | **Verified** | Backstage docs, "What is Backstage?" — project documentation, read 2026-10-19; the 2020 date is carried from [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) |
| 11 | SWE-bench comprises 2,294 task instances from real GitHub issues across 12 Python repositories; at publication the best model resolved "only the simplest issues" | **Verified (qualitative use only — no accuracy figure quoted)** | Jimenez et al., "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?", arXiv:2310.06770 — preprint 2023-10-10, ICLR 2024 |
| 12 | The paved-road framing entered platform discourse with Bottcher's 2018 essay; the desired way of doing things should be the easiest way, not the enforced way | **Verified (as cited; essay not re-extracted this session)** | Evan Bottcher, "Paved Road in the Jungle" — essay, 2018; original URL has moved (see §16) |
| 13 | A golden path is a preconfigured, paved, end-to-end supported route that reduces cognitive load and keeps work safe and compliant | **Verified (as cited from sibling)** | platformengineering.org — community/organisation reference, as recorded in [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) §2.3; the specific blog URL returned 404 on 2026-10-19, so no page-level citation is claimed here (see §16.1) |
| 14 | Regulation (EU) 2022/2554 (Digital Operational Resilience Act) applies from 17 January 2025 and requires financial entities to maintain ICT risk-management and ICT change-management policies | **Verified** | Regulation (EU) 2022/2554 — regulation, 2022, applicable 2025 (no article numbers cited) |
| 15 | Agents improve platform-engineering productivity, or raise it by a stated amount | **By design — declined** | No statistic, percentage or productivity figure is printed in this guide. The subject is saturated with vendor productivity claims; attribution is not established (§8.3), and the CNCF white paper's indirect-value point argues against a direct claim |
| 16 | A peer-reviewed study of agents performing platform-engineering work specifically exists | **Negative finding** | Searches and sources read for this guide found none; the nearest evidence is repository-task benchmarks (row 11), which are not platform work (§14.3) |
| 17 | An agent that passes a coding benchmark is ready to change a platform default | **Rejected — category error** | The benchmark's success criterion is local and machine-checkable (row 11); a default's correctness depends on a population and a convention outside the repository (§5.3, §14.3) |
| 18 | Autonomous platform ownership and self-healing infrastructure are available capabilities | **Vendor claim / Flagged** | Vendor material, no independent evidence read; §14.2 records why the demonstrations stop short of the apply boundary |

Rows 1–14 are the sources this guide leans on. Rows 15–18 are the honest negatives: one declined by design, one absence of evidence, one rejected as a category error, one flagged as promise.

### 15.1 How to read the verdicts

The verdict vocabulary is deliberately narrow, and each term means one thing.

**Verified** — checked against the named source on the date shown, and the source says what the claim says it says. **Verified (as cited)** — the source is a sibling guide or a secondary reference that this pass did not re-extract; the claim is as reliable as that guide's verification, which is a weaker position and is labelled as such. **Verified (used as categories, not as a ranking)** — the material exists and is used structurally, with no ordering or quality judgement implied. **Vendor claim** — a statement a vendor makes about its own capability, recorded as a claim and not as a finding. **Flagged** — repeated in circulation but not traceable to a measurement that survived this pass. **Negative finding** — a search pass established that the thing does not appear in the sources checked; it is evidence of absence in a defined corpus, not proof of non-existence. **Rejected** — the claim is contradicted by a source in this table, or is a category error. **By design — declined** — the claim could be made in the genre and this guide chose not to make it, with the reason stated.

Two conventions apply throughout. No accuracy, productivity, adoption or improvement figure from any benchmark or vendor is printed, because the guide's argument does not depend on one and a figure would invite a comparison the evidence cannot support. And no source is cited from memory: a claim without a date and a kind does not appear in the table.

---

## 16. What Could Not Be Verified, the Glossary, Cross-References and the Closing Summary

### 16.1 What could not be verified

Recorded as explicit negatives, because a silent omission reads as coverage. None of these should be asserted elsewhere.

1. **The specific platformengineering.org page on golden paths.** The organisation is cited for the golden-path definition on the strength of its published material, but the specific blog URL returned a 404 when checked on 2026-10-19. The definition is attributed to the community reference at organisation level, and no page-level citation is claimed.
2. **Bottcher's essay at its original location.** "Paved Road in the Jungle" (2018) is cited as an essay whose framing entered platform discourse; the original URL has moved and the text was not re-extracted this session. The attribution is to the essay and the year, not to a quoted passage.
3. **Any peer-reviewed study of agents performing platform-engineering roles.** Searches and the sources read found none. §14's separation of demonstrated from promised capability is therefore built on adjacent evidence — repository-task benchmarks, and vendor documentation about agents generally — and the generalisation to platform work is this guide's argument, not a published result.
4. **Any measurement of exception-list decay.** §12.4 asserts that an exception list maintained by humans goes stale and that a stale list can preserve a pothole as a preference. No source read quantifies how often this happens or how fast; the claim is a structural argument from the list's maintenance burden, not an empirical finding.
5. **Any independent evaluation of vendor claims about autonomous platform ownership.** The claims in §14.2 are recorded as claims. No benchmark, case study or audit read for this guide tests them, and no vendor is named as evidence.
6. **The provenance of the 2020 Backstage open-source date at primary source.** The date is carried from [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md), which verified it; the Backstage documentation page read today confirms the project's identity, its catalogue/templates/TechDocs structure and its CNCF Incubation status, not the 2020 date itself.
7. **The evidential basis of the DORA AI Capabilities Model.** The capability list and the precondition framing are verified as the model's content. The research behind the model — sample, method, effect sizes — was not audited, and no effect size is quoted anywhere in this guide.

**Research-tooling note.** Search on this host was used only to locate sources and was uneven: several queries returned results that were generic or commercial rather than substantive, which is a relevance limitation of the tooling and **not** evidence that material does not exist. The load-bearing method was direct extraction of the named primary URLs — the Internal Developer Platform reference page, the Team Topologies key-concepts page, the CNCF Platforms White Paper, the DORA metrics guide and AI capabilities model, the Backstage documentation, the SWE-bench arXiv record, and the platformengineering.org material as recorded in the sibling guide. Two pages would not extract at their current URLs (§16.1 items 1–2). Where a page would not extract, the claim is labelled rather than filled from memory.

### 16.2 Glossary

**Agent** — A model-driven system with tools, a mandate and a loop: it reads state, decides an action, executes it and observes the result. In this guide, distinguished by position (customer, colleague, subject) rather than by capability.

**Change board** — The forum in which an institution approves changes above a threshold, and the place where the question "who approved this, and on what evidence" is asked and answered.

**Exception handling** — The human activity of deciding that a convention does not apply to a particular case, and recording why. In this guide it is a first-class artefact and never an agent's judgement (§12.3).

**Four-eyes** — The maker-checker principle: for a consequential act, one party creates and another confirms. An agent may be the maker and may never be the confirming eye (§10.2).

**Golden path** — A preconfigured, end-to-end supported route through the platform for a common task, designed to reduce cognitive load and keep work safe and compliant.

**Infrastructure-as-code plan/apply** — The split between a plan (a reviewable description of what would change) and an apply (the change). The split is a credential boundary and the platform's native propose-then-commit primitive (§9.3).

**Internal developer platform** — The bound-together set of tools and services a platform team builds to provide golden paths and self-service; the substrate, the portal in front of it, and the automation underneath.

**Paved road** — The standardised, supported default path: the desired way of doing things made the easiest way rather than the enforced way.

**Platform engineering** — The discipline of building and running a shared substrate so that product teams can deliver without each mastering the whole estate; its output is a product, not a service desk.

**Platform team** — A team type whose product is an internal platform, providing capabilities that accelerate stream-aligned teams; constrained in capacity, judged by adoption, and the owner of the road other teams walk on.

**Self-service** — The property that a developer can obtain a platform capability without a ticket and without a platform engineer in the loop.

**Service catalogue** — The ownership graph of the estate: what exists, who owns it, what it depends on, and which template produced it. The only place a platform change's population is enumerable (§9.2).

**Template/default propagation** — The mechanism by which one platform change reaches many services, most of which do not exist yet; the reason platform blast radius is measured in populations rather than instances (§7.3).

### 16.3 Cross-references

| Guide | What it owns | Read it for |
|---|---|---|
| [../ai_platform_engineering_guide.md](../ai_platform_engineering_guide.md) | The platform-engineering canon (§2), the platform stack (§3), self-service and prompt-to-production (§6), the AI-platform design — the inverse subject | The canon this guide cites rather than restates; sense (a) and sense (c) |
| [agent_decision_policy_guide.md](agent_decision_policy_guide.md) | Ask/recommend/act/wait posture and the banking authority mapping | The posture vocabulary §4 applies; §9 on who may commit the institution |
| [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) | Sandbox and isolation mechanics | How §7's containment is actually enforced |
| [agentic_engineering_guide.md](agentic_engineering_guide.md) | The agent construction lifecycle | Building the agent whose acts §4 rules on |
| [agentops_guide.md](agentops_guide.md) | Agent operations | Running it, and keeping the record §10 requires |
| [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) | Production operation of agents | The reliability discipline behind §14's demonstrated/promised split |
| [agent_versioning_guide.md](agent_versioning_guide.md) | Versioning of agents, prompts and tools | §6.2's requirement that a shipped agent be a versioned entity |
| [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) | Enterprise agent-platform architecture | The hosting side of sense (a) |
| [agent_scaffolding_guide.md](agent_scaffolding_guide.md) | Scaffolding **agents** — a different act from scaffolding services (§4.2) | Not to be confused with §4.2 |
| [../finops_agents_guide.md](../finops_agents_guide.md) | The cost of agents: run economics and cost attribution | The accounting this guide does not attempt (§8 is about value attribution, not cost) |
| [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) | Controls and the evidence question | §10.3's evidence pack |
| [../../banking/ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) | The bank's AI obligations | The regulatory context around §10.4 |
| [../../management/high_performance_team_guide.md](../../management/high_performance_team_guide.md) | Team design and the platform-team type | §2's account of the constrained product team |

### 16.4 Closing summary

The subject of this guide is an agent doing platform-engineering work, and its whole difficulty is contained in one distinction: the platform team's output is not a deliverable but a **road other people walk on**. A service change is consumed where it is made. A template, a default, a permission or a pipeline is consumed later, elsewhere, by teams that were not in the room — and the population it reaches is the unit of harm, not the diff.

That fact produces the guide's rulings. An agent may **produce artefacts**: render a scaffold, propose a migration diff, open a pull request, classify a ticket, draft a runbook section, comment on infrastructure-as-code with the policy cited. It may not **produce effects**: merge, apply, close as resolved, edit a default, approve. The line is not drawn at difficulty or at model capability; it is drawn at the signature, because approval is the act the institution is answerable for and accountability cannot be delegated to something that cannot be asked why.

The control surface — the road, its defaults, its guardrails, its approvals — is where review fails as a mitigation, for four independent reasons: the diff does not show the population, the effect is not proportional to the diff, the reviewer's context is the reviewer's rather than the institution's, and volume converts review into sampling. The remedy is containment before the act: plan-only authority, scoped credentials per action class, non-production first, no standing production write, required review by someone other than the author, and an append-only log.

The worked example is the thesis made concrete. An agent inventories 120 repositories and is right about 109 of them; four of the eleven non-standard repositories are deliberately so, and those four are the whole problem. The answer is not a better agent but a narrower mandate plus an exception list that a human owns. The exception list is the mechanism by which the organisation's knowledge enters the agent's behaviour at all, and it is a platform artefact on the same footing as the template.

What this guide declines to claim is as important as what it claims. It prints no productivity figure, ranks no tool, and asserts no adoption statistic, because the platform is a shared input, the confounders are the platform team's own roadmap, and DORA's own model treats a quality internal platform as a precondition of AI outcomes rather than a consequence of them. What can be reported is descriptive: proposals opened, proposals merged, proposals reworked, exceptions routed — evidence rather than attribution.

The failure modes all reduce to one shape. An agent that paves a road needs to know which divergences are defects and which are decisions, and that knowledge is not in the repository. Four of the five failures in §11 are failures of seeing; the fifth is a failure of landing, where the knowledge stays in the agent and leaves when it does. All five are fixed by artefacts the platform team builds — a catalogue, an exception list, an ownership requirement, a schedule, a documentation destination — which is the practical form of the argument: the agent can pave, and the platform team must supply the map.

The platform team's product is a paved road; an agent that paves it must be able to tell a pothole from a preference.
