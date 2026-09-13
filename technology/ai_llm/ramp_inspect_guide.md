# Ramp Inspect — the inspected code

> **A system case study deep-dive on Ramp Inspect, the in-house background coding agent that reportedly writes the majority of Ramp's merged pull requests — covering what it is, why Ramp declined to buy off the shelf, the Modal-sandbox architecture as publicly documented, the impact metrics and what they do not prove, an ETCLOVG layer-by-layer reading of the lessons, enterprise and banking relevance, and an illustrative scoped pilot for a regulated institution.**

**Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank  
**Series:** Agent Engineering · System Case Studies · **Domain:** AI Engineering · Fintech Agent Platforms  
**Reading time:** ~40 minutes  
**Companion guides:** [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) (the E/T/C/L/O/V/G frame this guide maps onto), [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) (the isolation spectrum Inspect's remote sandboxes sit at the end of), [coding_agents_research.md](coding_agents_research.md) (the coding-agent product landscape this guide positions against), [agentic_engineering_guide.md](agentic_engineering_guide.md) (the engineering discipline), [context_engineering_guide.md](context_engineering_guide.md) (the grounding problem), [agentops_guide.md](agentops_guide.md) (running agents in production), [ai_governance_framework_guide.md](ai_governance_framework_guide.md) (the governance frame for §6)

---

## Table of Contents

- [Sourcing convention used throughout this guide](#sourcing-convention-used-throughout-this-guide)

1. [What Ramp Inspect Is](#1-what-ramp-inspect-is)
2. [Why Ramp Built In-House](#2-why-ramp-built-in-house)
3. [The Architecture as Publicly Documented](#3-the-architecture-as-publicly-documented)
4. [Impact and Measurements](#4-impact-and-measurements)
5. [Engineering Lessons](#5-engineering-lessons)
6. [Enterprise and Banking Relevance](#6-enterprise-and-banking-relevance)
7. [Worked Example: The Cymbal Bank Internal Coding-Agent Pilot](#7-worked-example-the-cymbal-bank-internal-coding-agent-pilot)
8. [Positioning: Ramp Inspect Against the Public Alternatives](#8-positioning-ramp-inspect-against-the-public-alternatives)
9. [One-Page Summary](#9-one-page-summary)
10. [Claims Audit](#10-claims-audit)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [References](#13-references)

---

## Sourcing convention used throughout this guide

Every factual claim in this guide carries one of three markers. Read them before reading the rest of the document, because the whole point of this case study is that its headline numbers come from parties with an interest in the headline.

| Marker | Meaning |
| --- | --- |
| ✅ | **Verified at a primary source.** The figure appears in a first-party artifact — Ramp's own engineering blog, or the company's own published spec — and is stated by a named Ramp employee. |
| ⚠ | **Single-source or vendor-published.** The figure appears in exactly one place, or appears in material published by a party with a commercial interest in the case study. Modal is the sandbox vendor; The Pragmatic Engineer is largely relaying an interview arranged with Ramp's leadership. Treat as a claim, not a measurement. |
| ❌ | **Not verifiable.** No public source states this, and it is listed in [What Could Not Be Verified](#11-what-could-not-be-verified) rather than inferred. |

Anything in a block explicitly labelled **Guide's analysis** is the author's own reasoning, not a sourced finding. Those blocks are meant to be argued with.

### The sources, and what each is

| Source | Date | Authorship | Interest |
| --- | --- | --- | --- |
| [Ramp Builders Blog — "Why We Built Our Own Background Agent"](https://builders.ramp.com/post/why-we-built-our-background-agent) | 12 Jan 2026 | Ramp (Zach Bruggeman, Jason Quense, Rahul Sengottuvelu) | Semi-promotional; also a hiring artifact; republished as a build spec |
| [Modal — "How Ramp built a full context background coding agent on Modal"](https://modal.com/blog/how-ramp-built-a-full-context-background-coding-agent-on-modal) | 19 Feb 2026 | Modal (Greta Workman, Product Marketing) | **Vendor.** Modal sells the sandbox |
| [The Pragmatic Engineer — "Why Ramp built its own in-house coding agent, Inspect"](https://newsletter.pragmaticengineer.com/p/why-ramp-built-inspect) | 25 Aug 2026 | Gergely Orosz, Jessica Salmon, Ivan Klaric | Independent outlet, paid tier; largely an interview write-up with Ramp leadership |
| [Linear customer story — "The coding agent behind 75% of Ramp's merged PRs"](https://linear.app/customers/ramp) | 2026 | Linear (vendor) | **Vendor.** Linear sells the issue tracker Ramp grounds Inspect in |
| [The New Stack — "Ramp's Inspect shows closed-loop AI agents are software's future"](https://thenewstack.io/ramps-inspect-shows-closed-loop-ai-agents-are-softwares-future/) | 2026 | Commentary | Sponsored-adjacent trade press; useful for framing, not for numbers |

**The structural warning up front.** Three of the five sources above are published by companies that sell Ramp something, and the fourth is Ramp. There is no independent audit of Inspect's usage numbers, no disclosure of how the percentage is computed, and no denominator published. That is not a reason to dismiss the case study — the engineering is specific, reproducible, and unusually candid about mechanics — but it is a reason to read §4 before you quote anything from it.

---

## 1. What Ramp Inspect Is

### 1.1 One sentence

Ramp Inspect is Ramp's internal, company-built **background coding agent**: a coding agent that runs not on the engineer's laptop but in a full, sandboxed development environment in the cloud, with access to the same internal systems the engineer has, so that it can verify its own work before a human ever looks at it. ✅ ([Ramp Builders](https://builders.ramp.com/post/why-we-built-our-background-agent))

Ramp's own summary of the design principle is unusually clean:

> Inspect writes the code like any other coding agent, but closes the loop on verifying its work by having all the context and tools needed to prove it, as a Ramp engineer would. ✅ (Ramp Builders)

And the constraint they set themselves:

> Agents should have agency, and so we made sure Inspect is never limited by missing context or tools, but only by model intelligence itself. ✅ (Ramp Builders)

That second sentence is the entire thesis of the product. It is also the thesis of this guide's §5.

### 1.2 What it actually does

The Pragmatic Engineer's summary is precise: *"Imagine an AI coding agent running on remote sandboxes with access to most internal data sources, and verifying all backend and frontend changes on the remote machine."* ✅

Concretely, Ramp documents four modes of use:

- **Coding.** Engineers prompt Inspect with small and medium-sized tasks that are often one-shot passes. For larger tasks, devs use Inspect to kick-start an idea and then take over locally. ✅ (Pragmatic Engineer)
- **Bugfixing in Slack.** A reply of `@inspect fix this` in a Slack thread causes Inspect to read the entire thread context and raise a pull request with a fix. ✅ (Pragmatic Engineer; Linear repeats the "often all it takes to open a PR" framing ✅)
- **Debugging.** Inspect steps through code in debugger mode, queries a **sanitized read-only production database replica**, makes Snowflake queries, and hunts business-logic and data mismatches. ✅ (Pragmatic Engineer)
- **Using Inspect to build Inspect.** The agent is used on its own repository. ✅ (Pragmatic Engineer)

### 1.3 Who uses it — and this is the part that is genuinely unusual

Modal's framing is that Ramp wanted *"every builder at Ramp — engineers, product managers, designers"* to be able to ship code using AI. ✅ (Modal, vendor-published)

Ramp says the same thing directly: Inspect *"lets builders of all backgrounds, contribute with the tooling and setup an engineer would."* ✅ (Ramp Builders)

The concretely documented non-engineer paths:

- **Designers** prompt Inspect from a Chrome extension that lets them visually select UI elements in a running React app, using DOM and React internals rather than screenshots, to avoid burning tokens on images. ✅ (Ramp Builders)
- **Product managers** are described as empowered to *"directly add features to their product."* ⚠ (Modal — vendor framing, no PM-level usage numbers published)
- **Sessions are shareable and multiplayer.** A designer can take a dashboard rework most of the way from a Figma file and hand the session to an engineer. Any colleague can be added to any session. ✅ (Linear; Pragmatic Engineer)

The Pragmatic Engineer adds a blunt detail that matters for §4 and §6: **all Inspect sessions are public, and there are no opt-outs allowed.** ✅ (Pragmatic Engineer)

### 1.4 How it is invoked

Four first-class clients, all syncing to the same session state:

| Client | Signature capability | Source |
| --- | --- | --- |
| **Slack** | `@inspect fix this` in a thread; a fast classifier picks the target repository from the message, thread context, and channel name | ✅ Ramp Builders / Pragmatic Engineer |
| **Web** | Chat interface, hosted VS Code editor inside the sandbox, streamed desktop view, session statistics page | ✅ Ramp Builders |
| **Chrome extension** | Visual element selection in the running React app; DOM/React-tree payload instead of screenshots | ✅ Ramp Builders |
| **Pull request** | Discussion and iteration on the PR itself | ✅ Ramp Builders |

Ramp also added **voice** input to sessions. ✅ (Ramp Builders)

### 1.5 The background/session model

This is the architectural heart, and Ramp publishes it plainly.

Each session gets **its own sandboxed VM on Modal** containing everything an engineer would have locally: Vite, Postgres, Redis, Temporal, RabbitMQ, and — per Modal — *"every service an engineer would have locally."* ✅/⚠ (Ramp Builders ✅ for the VM-per-session claim; Modal ⚠ for the enumerated service list and the "every service" completeness claim)

Inside that sandbox:

- **OpenCode** runs as the coding agent harness. ✅ (Ramp Builders)
- A **hosted VS Code server** runs for manual edits. ✅
- A **web terminal** runs. ⚠ (Modal only)
- A **VNC stack with Chromium** runs, so the agent can navigate the real app in a real browser, take before-and-after screenshots, and confirm frontend changes visually. ✅ (Ramp Builders; Modal adds VNC specifically ⚠)
- The agent is wired into **GitHub, Slack, Buildkite, Sentry, Datadog, LaunchDarkly, Braintrust**, and Ramp's observability stack. ✅ (Ramp Builders for the full list)

The agent can therefore, inside one session, **make real API requests, run the test suite, read production telemetry, query feature flags, and reproduce a bug end to end** — the "closed loop" the trade press keeps calling it. ✅ (Ramp Builders; Pragmatic Engineer)

Ramp's stated reason for putting all of this *inside* one sandbox rather than beside it is latency, and it is worth quoting because it is the least vendor-flavoured statement in the whole corpus:

> Running everything inside a single sandbox means the agent has the same low-latency access to services, files, and tools that an engineer would have locally. There's no network hop between the agent and the test suite, no remote filesystem to sync. ⚠ (Modal — vendor-published, but a technical claim about Ramp's design rather than a Modal feature claim)

### 1.6 The reported scale of impact

Headline figures, with per-claim sourcing. §4 interrogates each of these; the point here is to state them.

| Figure | As stated | Source | Marker |
| --- | --- | --- | --- |
| Share of merged PRs written by Inspect | **~30%** of all PRs merged to frontend and backend repos | Ramp Builders, 12 Jan 2026 | ✅ first-party, self-published |
| Share of merged PRs written by Inspect | **over half** of all merged pull requests at Ramp | Modal, 19 Feb 2026 | ⚠ vendor |
| Share of merged PRs written by Inspect | **75%** of all merged PRs | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source, paid article |
| Share of merged PRs written by Inspect | *"three of every four PRs Ramp merges"* | Linear customer story | ⚠ vendor, restating the same number |
| Inspect writing Inspect | **over 80%** of Inspect is written by Inspect | Modal; Pragmatic Engineer | ⚠ two sources, same origin |
| Share of PRs merged into the **Inspect repo itself** that came from an Inspect session | **~90%** | Pragmatic Engineer | ⚠ single-source |
| Engineers who have contributed to the Inspect codebase | **150+** | Pragmatic Engineer | ⚠ single-source |
| Size of the Inspect team | **5.5 people** — four engineers, a director, a part-time PM | Pragmatic Engineer | ⚠ single-source |
| Total Inspect sessions | crossed **one million** in July 2026 | Pragmatic Engineer | ⚠ single-source |
| Concurrent sessions | *"hundreds of concurrent sessions"*; *"hundreds of computers they can work on simultaneously"* | Modal | ⚠ vendor |
| Agents built on the Inspect platform | **more than 200** | Pragmatic Engineer | ⚠ single-source |
| Sandbox spin-up | end-to-end start *"in a few seconds"*; *"under 5 seconds"* to a fully provisioned environment | Modal / Pragmatic Engineer | ⚠ vendor + single-source |

The trajectory, as reported, runs 30% (January 2026) → 50%+ (February 2026) → 60% → 75% (May 2026). Note now that the 60% step contradicts the 30% step; §4.3 handles the contradiction rather than smoothing it over.

### 1.7 Where it came from

Inspect was not a mandate from the top and not a greenfield project. It has a two-act history:

1. **Inspect v1 (early 2025): a Chrome extension for designers** to make minor website changes by highlighting an area and describing the edit — copy changes, button placement. ✅ (Pragmatic Engineer) It was liked but not adopted, because engineers had no reason to use it and it still required a local development environment. ✅
2. **Inspect v2 (November 2025): a remote development environment with a coding agent on top.** ✅ (Pragmatic Engineer) The pivot to centrally-configured remote environments removed the local-setup objection, and adoption followed.

The first version was built in about two weeks by three people; Modal quotes Zach Bruggeman saying the first working version "only took me a few days to get off the ground." ⚠ (Linear for the two-week framing; Modal for the quote)

That origin matters for §5: **the winning design was the second attempt, and it was the removal of setup friction — not an intelligence upgrade — that made the difference.**

---

## 2. Why Ramp Built In-House

Ramp is a company that could afford any coding agent on the market. It looked at the market and built its own. This section records the stated reasoning — quoted, attributed, and not laundered into fact.

### 2.1 The four stated reasons

The Pragmatic Engineer enumerates three, and Ramp's own post supplies a fourth. All are Ramp's reasoning as reported:

**Reason 1 — local machines cannot run enough agents in parallel.** ✅/⚠ (Pragmatic Engineer)

> "Ramp found third-party products below expectations; they liked Claude Code on day 1, but were constrained by only being able to run one or two sessions on local machines." ⚠ (Pragmatic Engineer, relaying Ramp)

Ramp's own version: *"Because Inspect sessions are fast to start and effectively free to run, you can use them without rationing local checkouts or worktrees."* ✅ (Ramp Builders)

**Reason 2 — frontend tooling, specifically for designers.** ✅ (Pragmatic Engineer)

> "The web engineering team wanted to improve their frontend tooling so designers could make small UI tweaks. There was an opportunity to use AI to automate themselves out of that loop." ⚠

**Reason 3 — the need for remote development environments as the codebase grew.** ✅ (Pragmatic Engineer)

> "As Ramp scaled, so did the complexity, and with it there was more work at the intersection of systems, like debugging backward compatibility, and broken API contracts. The solution was to create remote dev environments." ⚠

**Reason 4 — owning the tooling is strictly more powerful than renting it.** ✅ (Ramp Builders, first-party)

> "We think anyone should be able to build this. Owning the tooling lets you build something significantly more powerful than an off-the-shelf tool will ever be. **After all, it only has to work on your code.**" ✅ (Ramp Builders)

That last clause is the best sentence anyone has written about internal agents. A general-purpose coding agent has to handle every repository, every language, every build system, every deploys story — so it must generalise. An internal agent has exactly one customer and exactly one codebase. **The scope reduction is the product advantage**, not the model.

### 2.2 The enterprise-context argument

The single most repeated Ramp claim across every source is that *context is the differentiator, and context lives behind the corporate firewall*.

- Ramp: Inspect is *"grounded by Ramp's internal codebase, enterprise context, and developer tooling."* ⚠ (Modal — vendor phrasing of Ramp's position)
- Ramp: *"closes the loop on verifying its work by having all the context and tools needed to prove it, as a Ramp engineer would."* ✅
- Pragmatic Engineer: *"At present, most third-party AI harnesses cannot do these kinds of verifications 'out of the box' because they lack internal integrations with things like telemetry and feature flag systems."* ⚠
- Pragmatic Engineer: *"Inspect's core principle is that agents should have access to the same context and tools as software engineers."* ⚠

Modal's write-up makes the local-versus-background argument explicit: Ramp *"knew that a background agent that was slower or less capable than working locally would never get adopted. Performance was crucial."* ⚠ (Modal)

### 2.3 But interrogate the reasoning

**Guide's analysis.** Three-quarters of the "why in-house" story, as publicly told, is not actually about in-house versus vendor. Read the reasons again:

- "Only one or two sessions on a local machine" is an argument for **remote execution**, which GitHub Codespaces, Gitpod, Coder, Devin, and every cloud-agent vendor already sell.
- "We wanted designers to make UI tweaks" is an argument for a **Chrome extension**, which is a client, not a harness.
- "We needed remote dev environments" is an argument for **infrastructure**, which is what Modal sells and what Ramp buys.

The genuinely in-house-specific argument is the thin fourth one: the one API key, the exact log schema, the internal product-issue tracker holding six years of specs and customer requests. Ramp's principal engineer says the integration work was quick — *"90 percent of the way there in about 30 minutes"* after prompting a coding agent with the vendor's docs. ✅ (Linear, vendor-published, but a candid and self-limiting quote).

**Guide's analysis.** If the differentiator can be wired up in half an hour by an agent, then "build your own harness" is not the only path to the same capability — "buy the harness, wire your own MCP servers and tools into it" is a live alternative, and it is what most enterprises should try first (§6). What Ramp bought by going in-house was not a moat in the wiring; it was **control of the roadmap, the client surface, and the feedback loop** — the ability to make the agent's home the PR, the Slack thread, and the design tool, on Ramp's schedule. That is a real thing to buy. It is a different thing from the one usually claimed.

### 2.4 What is *not* claimed

Two arguments people assume are in the record are not:

- **Cost.** No source publishes a cost-per-session, a token spend, a Modal bill, or an ROI figure for Inspect. Ramp says sessions are *"effectively free to run"* ✅ (Ramp Builders) — which is an adoption-nudge statement about internal chargeback, not a cost disclosure. See §11.
- **Benchmark superiority.** No source claims Inspect's *model* is better. Ramp is explicit that the ceiling is model intelligence: *"never limited by missing context or tools, but only by model intelligence itself."* ✅ The claim is that Inspect removes **non-model** limits, not that it beats other harnesses on the same model.

That distinction is worth carrying into §4 and §5, and it is the reason §8's comparison table is about interface and isolation rather than about intelligence.

---

## 3. The Architecture as Publicly Documented

This section describes only what the published record supports, and flags every place the record stops. Where internals are not public, this guide says *not disclosed* rather than drawing a plausible diagram.

### 3.1 The four published layers

Ramp's own post is effectively a public build spec — the company explicitly invites readers to *"paste the link to this post into a coding agent and let it begin building."* ✅ (Ramp Builders) That is an unusual act of disclosure, and it means the architecture below is not reconstructed; it is documented.

| Layer | Component | Documented behaviour | Source |
| --- | --- | --- | --- |
| **Execution** | Modal Sandboxes | One sandboxed VM per session, full dev environment inside | ✅ Ramp Builders / Modal |
| **Agent harness** | OpenCode | Server-first open-source coding agent; TUI and desktop app are just clients | ✅ Ramp Builders |
| **API / session state** | Cloudflare Durable Objects + SQLite | One SQLite database per session; state synchronised across clients; multiplayer | ✅ Ramp Builders |
| **Streaming** | Cloudflare Agents SDK | Real-time streaming between sandbox, API, and clients; WebSocket hibernation | ✅ Ramp Builders |
| **Client** | React / Vite front end | Web client (desktop + mobile), Slack bot, Chrome extension | ✅ Ramp Builders / Pragmatic Engineer |

Modal supplies the surrounding distributed primitives, and Modal's own case study is specific about which:

| Modal primitive | Role in Inspect | Source |
| --- | --- | --- |
| **Sandboxes** | Per-session sandboxed VM | ⚠ Modal |
| **Functions on a cron** | Every 30 minutes: clone repositories, install dependencies, build fresh filesystem snapshots | ⚠ Modal / ✅ Ramp Builders |
| **Dicts** | Session locks; image metadata | ⚠ Modal |
| **Queues** | Route prompts from any client into the right session; decouple input from execution | ⚠ Modal |
| **Filesystem snapshots** | Saved as diffs from the base image; only modified files persisted | ⚠ Modal / ✅ Ramp Builders |

### 3.2 The snapshot mechanism, in detail

This is the most reproducible engineering in the case study, and both Ramp and Modal describe it identically. ✅

The loop, as Ramp documents it:

1. Ramp maintains an **image registry**, defining one image per code repository. ✅
2. **Every 30 minutes**, images are rebuilt: clone the repository, install runtime dependencies, run any initial setup and build commands. ✅
3. Initial setup deliberately runs the app and the test suite once so that later runs hit a warm cache. ✅
4. A **snapshot** of the sandbox filesystem is saved in that completed state. ✅
5. When a builder starts a session, a **new sandbox starts from the stored snapshot**. ✅
6. Because the snapshot is at most 30 minutes old, syncing with the head of the repository is *nearly instant*. ✅
7. When the agent finishes, **another snapshot is taken** and restored later if the sandbox has exited and the user sends a follow-up. ✅

Modal explains the cost property that makes this viable: filesystem snapshots are *"calculated as the difference from your base image, so only modified files are stored,"* and restoring one *"utilizes the same infrastructure we use to get fast cold starts."* ✅ (Modal docs — first-party documentation of the mechanism, not a case-study claim)

Modal's docs also give the retention numbers Ramp does not publish: filesystem and directory snapshots default to **30 days** TTL and can be configured or set to never expire; memory snapshots expire after **7 days** and cannot currently be extended. ✅ (Modal docs)

### 3.3 Three published latency optimisations

Ramp publishes these as recommendations, which means they are claims about its own implementation:

1. **Warm the sandbox while the user is still typing.** Start cloning and setup as soon as the first keystroke lands, so the environment can be ready before the prompt is submitted. Keep a **pool of warm sandboxes** for high-volume repositories, expiring and recreating them as new image builds arrive. ✅ (Ramp Builders)
2. **Let the agent read files before the sync finishes, but block writes.** In a large repository it is unlikely a prompt touches a file changed in the last 30 minutes, so research can start immediately. The block is implemented as an **OpenCode plugin listening on the `tool.execute.before` event**, which blocks write and edit calls until git sync completes. ✅ (Ramp Builders)
3. **Push everything possible into the image build.** The image build can be slow because users only ever consume the last-built image. ✅ (Ramp Builders)

The published result: *"Under 5 seconds to spin up a fully provisioned remote dev environment."* ⚠ (Pragmatic Engineer) Modal says sessions *"start working on a prompt in a few seconds"* and that Ramp was targeting local-agent parity. ⚠

**Guide's analysis.** Optimisation 2 is the more interesting of the three, because it is a **read/write asymmetry** enforced at the tool layer rather than at the model layer. It says: staleness is tolerable for reading, not for writing. That is a reusable pattern for any agent working on a moving target, and it belongs to the Verification and Tooling layers (§5), not just to performance.

### 3.4 The pull-request interface

Ramp's design here is explicit, and it contains the single most important governance decision in the whole case study.

- **Authentication uses the invoking user's GitHub token**, not an app token. Ramp states the reason bluntly: opening PRs as the app *"would allow for any user to approve their own changes. You do not want to knowingly create a vector for unreviewed code to go into the codebase."* ✅ (Ramp Builders — first-party, and a genuine engineering-ethics statement)
- **Cloning uses a GitHub App installation token**, regenerated on each clone, so the clone does not depend on which user will consume the session. ✅
- **Git identity is set explicitly** — `user.name` and `user.email` are updated at commit and push time, because git operations are not tied to a GitHub user. ✅
- **Authorship is attributed per prompt.** Every prompt that causes code changes is attributed to the person who sent it, and authorship info is passed through to the agent. ✅ (Ramp Builders)
- **The flow:** the sandbox pushes the branch and sends an event to the API with the branch name and session ID; the API then opens the pull request using the user's token; a **GitHub webhook** tracks the PR being updated, merged, or closed. ✅

So the interaction model is: **humans hand work over as a prompt, and receive it back as a pull request in the normal review channel** — not as a chat answer, not as an auto-merge. The head of Applied AI frames the platform consequence: *"the bottleneck shifts from 'can the agent write correct code' to 'how many agents can you run in parallel'."* ⚠ (Modal) Linear's version of the same observation: *"The speed has moved Ramp's bottleneck from writing code to reviewing it."* ✅ (Linear, vendor-published interview)

### 3.5 Parallelism and concurrency

| Property | Documented claim | Source | Marker |
| --- | --- | --- | --- |
| Sessions per user | *"There's no limit to how many sessions you can have running concurrently"* | Ramp Builders | ✅ first-party |
| Multiple attempts | A builder can *"kick off multiple versions of the same prompt, and just see which one lands"* | Ramp Builders | ✅ |
| Model swapping | Users *"can try different approaches or swap models without thinking twice"* | Ramp Builders | ✅ |
| Session isolation | One SQLite database per session, so *"no one session can impact another"* | Ramp Builders | ✅ |
| Scale figure | *"scale to hundreds of concurrent sessions"* | Modal / Pragmatic Engineer | ⚠ vendor + single-source |
| Child sessions | Inspect can **spawn sessions itself** — one tool starts a new session, another reads any session's status, so the parent can check in periodically. Used for cross-repository research or for splitting one large task into many small PRs. | Ramp Builders | ✅ |
| Sub-agent ceiling | *"Don't be afraid of the possibility of it spawning too many agents; frontier models are smart enough to contain themselves."* | Ramp Builders | ✅ as a quote |

That last line is Ramp asserting that no hard cap is needed. **Guide's analysis:** it is the least defensible engineering claim in the published spec, and it is the one that most needs a spend ceiling bolted on in any regulated deployment (§6). "The model is smart enough to contain itself" is a load-bearing assumption about spend, not just about behaviour.

### 3.6 Model choice

| Question | What is disclosed | Marker |
| --- | --- | --- |
| Which model(s) does Inspect use? | Inspect *"supports all frontier models"*; a builder can swap models mid-experiment | ✅ Ramp Builders |
| Why is model choice open? | OpenCode was chosen partly because it offered *"model agnosticism"* | ✅ Pragmatic Engineer |
| Is any specific model named? | Yes, one: the **Slack repository classifier** uses *"GPT 5.2 with no reasoning"* — a fast model picking the target repo from message, thread context, and channel name | ✅ Ramp Builders |
| Which model writes the majority of the code? | **Not disclosed** | ❌ |
| Is there routing logic between models by task type? | **Not disclosed** | ❌ |
| What are the token budgets or context-window strategy? | **Not disclosed** | ❌ |

**Guide's analysis.** The single named model is the one doing the *cheapest* job. That tells you Ramp treats model selection as a per-step cost/latency decision — a fast non-reasoning model for routing, frontier reasoning models for the actual work — but it does not tell you the split, and no source does.

### 3.7 What is deliberately not in the public record

Listed here in brief; §11 restates them as the formal verification boundary with the metric gaps included. None of the following appears in any source consulted:

- **Sandbox network egress policy** — what the agent can reach outbound, whether there is an allow-list.
- **Secrets management and per-session scoping** — how credentials for Sentry, Datadog, LaunchDarkly, Braintrust, Snowflake, and the production replica are injected, rotated, and revoked.
- **Data residency, retention, and deletion** for snapshots, which persist cloned source for up to 30 days by Modal default ✅ (Modal docs) but whose *config* in Ramp's case is not published.
- **The sanitisation controls on the read-only production replica.** Ramp says "sanitized read-only prod DB replica" ✅ and stops; what is masked, who owns the control, and how it is tested are unstated.
- **The context-assembly pipeline** — how codebase, docs, tickets, and telemetry are selected, ranked, truncated, and budgeted into a prompt.
- **The evaluation harness** for Inspect itself, prompt/harness change control, cost per session and total spend, rollback and kill-switch behaviour, quota enforcement on self-spawned child sessions, and any incident history.
- **The opt-in/opt-out model** for the public-by-default session setting, beyond the statement that no opt-outs are allowed ✅ (Pragmatic Engineer).
- **The production model mix** — which models write the majority of the code, and whether routing is per-task, per-user, per-repository, or per-session.

**Guide's analysis.** That is a long list, and the honest reading is: **the public record describes the shell, not the brain.** What is documented — sandboxes, snapshots, cron, queues, tokens, PR flow, clients — is the platform. What is not documented is context assembly, which is precisely the capability Ramp credits for the results. The Skeleton is public; the selection function is not. Any organisation attempting to copy this will find that the copying is the easy half.

---

## 4. Impact and Measurements

This is the section most people will quote from. It is therefore the section that most needs the sourcing discipline applied, and the section that most needs a critique attached to it.

### 4.1 Every published figure, in date order

| # | Claim | Value | Source (exact) | Marker |
| --- | --- | --- | --- | --- |
| 1 | Share of merged PRs written by Inspect | **~30%** of all PRs merged to frontend and backend repos | [Ramp Builders Blog, "Why We Built Our Own Background Agent", 12 Jan 2026](https://builders.ramp.com/post/why-we-built-our-background-agent) — first-party | ✅ self-published |
| 2 | Time to reach that level | *"It only took a couple months"* | Ramp Builders, 12 Jan 2026 | ✅ |
| 3 | Sessions were *"effectively free to run"* | qualitative | Ramp Builders, 12 Jan 2026 | ✅ |
| 4 | Share of merged PRs | **"over half"** of all merged pull requests at Ramp | [Modal case study, 19 Feb 2026](https://modal.com/blog/how-ramp-built-a-full-context-background-coding-agent-on-modal) — sandbox vendor | ⚠ vendor |
| 5 | Share of merged PRs, scoped to frontend and backend repos | *"roughly half of all merged pull requests across Ramp's frontend and backend repos are started by Inspect"* | Modal, 19 Feb 2026 | ⚠ vendor |
| 6 | Inspect writing Inspect | **over 80%** of Inspect now written by Inspect | Modal, 19 Feb 2026 | ⚠ vendor |
| 7 | Concurrency | *"hundreds of concurrent sessions"* | Modal, 19 Feb 2026 | ⚠ vendor |
| 8 | Spin-up | sessions start on a prompt *"in a few seconds"* | Modal, 19 Feb 2026 | ⚠ vendor |
| 9 | Adoption | *"Ramp didn't mandate Inspect, they let the product speak for itself."* | Modal, 19 Feb 2026 | ⚠ vendor |
| 10 | Non-engineer reach | PMs *"empowered to directly add features"*; designers get tight feedback loops | Modal, 19 Feb 2026 | ⚠ vendor, no numbers |
| 11 | Ship date | Inspect *"shipped and opened internally last November"* (= Nov 2025) | [Pragmatic Engineer, 25 Aug 2026](https://newsletter.pragmaticengineer.com/p/why-ramp-built-inspect) | ⚠ single-source |
| 12 | Share of merged PRs | **75%** of all merged PRs come from Inspect sessions | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 13 | Share of PRs at January 2026 | *"around 60% of PRs at Ramp were authored by Inspect"*, two months after v2 launch | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source, **conflicts with #1** |
| 14 | Share of PRs by May 2026 | **75%** | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 15 | Share of PRs merged into the **Inspect repo itself** from an Inspect session | **~90%** | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 16 | Spin-up, fully provisioned env | **under 5 seconds** | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 17 | Inspect team size | **5.5 people** — four engineers, a director, a part-time PM | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 18 | Contributors to the Inspect codebase | **150+ engineers** at Ramp | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 19 | Inspect writing Inspect | *"more than 80% of Inspect is written in Inspect sessions"* | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 20 | Total sessions | crossed **one million** in July 2026 | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 21 | Agents on the Inspect platform | **more than 200** | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 22 | Sessions are public and non-opt-out | *"All Inspect sessions are public and open to collaboration, with no opt-outs allowed."* | Pragmatic Engineer, 25 Aug 2026 | ⚠ single-source |
| 23 | Share of merged PRs | *"three of every four PRs Ramp merges"*; six years of specs, requests, and roadmaps in Linear used as context | [Linear customer story, 2026](https://linear.app/customers/ramp) — issue-tracker vendor | ⚠ vendor, restates #12 |
| 24 | Company size in Linear's profile | **1,600** | Linear customer story | ⚠ vendor profile field, unaudited |
| 25 | Time-to-first-integration | Linear agent-API integration *"90 percent of the way there in about 30 minutes"* | Linear customer story (quote from Zach Bruggeman) | ⚠ vendor-published interview |
| 26 | Bottleneck shift | *"moved Ramp's bottleneck from writing code to reviewing it"* | Linear customer story | ⚠ vendor |

### 4.2 Which numbers hold up, and which do not

**Guide's analysis.** Two properties separate the durable claims from the fragile ones: *first-party with a stated date* beats *vendor summary*, and *architecture* beats *usage statistics*. The strongest items are the v2 ship date (November 2025), the mechanical detail in §3, and the converging "80%+ of Inspect written by Inspect" / "~90% of Inspect-repo PRs" pair. The weakest are the single-source percentages — above all the 75% figure, which appears in exactly one paid article and is then restated by Linear, which is quoting the same story rather than corroborating it — plus every qualitative scale word ("hundreds of concurrent sessions", "one million sessions"). Full per-claim adjudication is in [§10](#10-claims-audit).

### 4.3 The contradiction the sources do not reconcile

Two published statements about the *same* calendar month disagree:

- Ramp, 12 January 2026: **~30%** of all PRs merged to frontend and backend repos are written by Inspect.
- Pragmatic Engineer, 25 August 2026: *"By January of this year, just two months after the v2 launch, around 60% of PRs at Ramp were authored by Inspect."*

Both cannot be right. There are three plausible explanations, and no published source adjudicates:

1. **Different denominators.** "All PRs merged to frontend and backend repos" (Ramp) vs "PRs at Ramp" (Pragmatic Engineer) — Ramp's phrasing is narrower and could exclude infrastructure, data, and platform repositories.
2. **Different numerators.** "Written by" (Ramp's own post) vs "authored by" vs "started by" (Modal's phrasing, #5) — a PR that Inspect started and a human finished is neither purely agent-authored nor purely human-authored, and the sources use the verbs loosely.
3. **Dates drifted in retelling.** The 60% could be a February or March figure restated as January.

**Guide's analysis.** The disagreement is the finding. It tells you that the headline metric — "share of merged PRs" — is **not defined consistently even between the company and the journalist interviewing the company**, which is exactly what you would expect of a metric designed for internal adoption dashboards rather than external audit. If you cite a Ramp number, cite the version, the date, and the source, and do not average them.

### 4.4 What "writes half the PRs" does and does not tell you

**Guide's analysis — this subsection is the guide's own critique, not a sourced finding.**

The percentage is seductive because it is a single number and it is large. It is also unusually weak in what it licenses you to conclude. Seven problems, in ascending order of how much they should worry a reader:

**(1) PRs are a unit of process, not a unit of value.** A one-line copy fix and a distributed-systems migration are both "a PR." A metric that counts them equally will move fastest on the tasks that are easiest to automate — which is the opposite of the work Ramp says it is trying to free engineers for. A rising percentage can therefore be consistent with the agent handling *more, smaller* changes while human value-per-PR rises, falls, or stays the same. The metric cannot distinguish these.

**(2) "Started by", "authored by", and "written by" are three different things.** Modal says PRs "started by Inspect"; Ramp's post says PRs "written by Inspect." A PR that Inspect opened with a broken first attempt and a human then spent a day repairing is counted identically to one that merged untouched. No source publishes the split.

**(3) Review load is redistributed, not eliminated.** Linear's own interview snippet concedes the bottleneck *moved* from writing to reviewing ✅. That means the 75% figure describes where code is *produced*, not where engineering time is *spent*. If review time per PR is flat or rising while PR count rises, total human effort can go up while "share of PRs written by AI" goes up too. **The metric is compatible with a net increase in human toil.**

**(4) The Inspect-repo statistic is self-referential.** *"~90% of PRs merged into the Inspect repo come from an Inspect session"* sounds like the strongest evidence in the corpus. It is actually the weakest kind of evidence: it is the agent's performance measured on the codebase the agent was most heavily tuned against, maintained by the team that built the agent, in the one repository where the agent's context is densest. A high number there is close to guaranteed and predicts little about a legacy mainframe-adjacent repo. This is textbook **selection on the treated**.

**(5) Survivorship and selection effects are unmeasured.** Inspect sessions that produce nothing — abandoned prompts, failed runs, prompts where the user gave up — are invisible in a merged-PR denominator. Ramp does publish a genuinely better metric that it recommends teams track — *"how many sessions result in a merged pull request"* ✅ (Ramp Builders) — but **no value for it is published anywhere.** That omission is the single most conspicuous gap in the public record, because it is the metric Ramp itself calls "the most important metric to track."

**(6) There is a denominator problem underneath the percentage.** "75% of merged PRs" says nothing about total merged PRs. If the agent tripled PR volume and humans kept output flat, the share rises to 75% with no increase in shipped value. If the agent's PRs are concentrated in a few high-frequency repositories, the org-wide number overstates penetration everywhere else.

**(7) Adoption-curve marketing is the genre.** Ramp's post recommends building a statistics page that surfaces *"how many sessions result in a merged pull request"* and shows metrics over time *"so you can inspire more growth"* ✅. That is an honest design goal, and it also means the metric is optimised for adoption storytelling. A number built to inspire growth should be read with that purpose in mind.

**What the numbers *do* support.** Three claims survive all seven objections:

- Inspect is used at high volume, by a large number of people, across many repositories, with enough genuine output that a 5.5-person team maintains it and 150+ engineers have contributed to it. ✅/⚠
- The engineering is real and specific — a public build spec, a named harness, named infrastructure, named integrations, and a governance decision (user-token PRs) that a marketing team would not invent. ✅
- Some workflows have been measurably reshaped: designers shipping UI changes and PMs shipping features are claims about *work moving across role boundaries*, which is harder to fake than a percentage and is corroborated by two independent-ish sources. ⚠

**The honest one-line summary.** *Ramp has demonstrated that a company can build a background coding agent that its engineers voluntarily use at scale. It has not published the evidence needed to conclude that this reduced total engineering effort, and the metrics it does publish cannot in principle show that.*

---

## 5. Engineering Lessons

Each lesson below is stated as: **the documented evidence**, then **the lesson**, then **the harness layer** it belongs to, cross-referenced to the sibling guide that owns that layer. Where a lesson is the guide's inference rather than a documented finding, it says so.

The layer taxonomy used here is the repo's own E/T/C/L/O/V/G frame — **E**xecution, **T**ooling, **C**ontext, **L**ifecycle, **O**bservability, **V**erification, **G**overnance — set out in [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md). Read that guide first if the layer names are unfamiliar; this section assumes it.

### 5.1 Sandbox-per-session, with snapshots rather than clones

**Evidence.** One sandboxed VM per session, containing Postgres, Redis, Temporal, RabbitMQ, Vite and every service an engineer would have locally ✅. Images rebuilt on a 30-minute cron; sessions start from a filesystem snapshot that is at most 30 minutes stale; snapshots stored as diffs from the base image so only modified files persist ✅. Result: *"under 5 seconds"* to a fully provisioned environment ⚠. One SQLite database per session, so *"no one session can impact another"* ✅.

**Lesson.** The expensive part of a coding agent is not the model call — it is **environment preparation**, and it can be moved almost entirely off the critical path by precomputing it on a schedule and restoring from a diff-based snapshot. This is the difference between a background agent that competes with a laptop's already-warm checkout and one that loses to it.

**Layer:** **E — Execution.** The sandbox is the execution substrate. Cross-ref [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md), which places remote sandboxes at the far end of the isolation spectrum (§8 there) and covers warm pools, snapshot isolation, and the escape/blast-radius trade-offs that Ramp's design assumes but does not discuss.

**Guide's inference.** Ramp never says it chose sandbox-per-session for *security*. It says it chose it for *latency and concurrency*. In a regulated setting, that ordering should be reversed: the isolation is the point, and the latency is the bonus.

### 5.2 Full-context grounding beats prompt engineering

**Evidence.** The stated design constraint is that the agent is *"never limited by missing context or tools, but only by model intelligence itself"* ✅. Inspect is wired into Sentry, Datadog, LaunchDarkly, Braintrust, GitHub, Slack, Buildkite ✅, has a sanitized read-only production replica and Snowflake access for debugging ✅, and reaches six years of specs, customer requests, and roadmaps held in Linear ✅/⚠.

**Lesson.** The leverage in an internal agent is in the **connective tissue**, not the prompt. Ramp's own account is that a third-party harness *cannot* do what Inspect does because it lacks the internal integrations ✅/⚠ — and the one integration Ramp describes building took half an hour of agent-assisted work ✅. Which means: wiring context is cheap, and the reason most agents are context-poor is a decision, not a technical barrier.

**Layer:** **C — Context.** Cross-ref [context_engineering_guide.md](context_engineering_guide.md) for retrieval, ranking, and budget management — the part Ramp does *not* publish (§3.7). Also relevant: [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) for how the integrations are likely exposed, and [prompt_injection_guide.md](prompt_injection_guide.md), because **every one of these integrations is an untrusted-input channel** and Ramp publishes nothing about treating them as such.

### 5.3 The PR-shaped interface is the adoption mechanism

**Evidence.** The interface is a pull request, opened with the invoking user's token, attributed to the person who sent each prompt, tracked by webhook, discussed in the PR ✅. Ramp explicitly refuses the app-token shortcut because it would let users approve their own changes ✅.

**Lesson.** Three things are being bought at once:

1. **Review is unchanged.** No new approval surface, no new queue, no new tool for a reviewer to learn. This is the opposite of the usual "AI outputs go somewhere novel" failure mode.
2. **Attribution is preserved.** Per-prompt authorship ✅ — the same discipline insurance and audit want, arrived at from a code-review motivation.
3. **The self-approval hole is closed at the token layer.** Not by policy, not by a UI warning. Ramp chose an authentication mechanism whose failure mode is the *safe* one. That is the single most transferable engineering decision in the case study.

**Layer:** **T — Tooling** (the PR as the agent's output tool) and **G — Governance** (attribution, self-approval prevention). Cross-ref [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) for the T/G boundary, and [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md) on treating the PR as a first-class agent artifact.

### 5.4 Self-hosting — "Inspect maintains Inspect"

**Evidence.** *"Inspect is used to build itself, and more than 80% of Inspect is written in Inspect sessions"* ⚠, with ~90% of PRs merged into the Inspect repo coming from an Inspect session ⚠.

**Lesson (evidenced).** Self-hosting is a **forcing function for tooling quality**: if your agent is the main user of your own repository, its context quality, test harness, and PR conventions have to be good enough to satisfy the agent's own loop. It is also the cheapest possible high-signal evaluation corpus.

**Lesson (guide's inference — clearly labelled).** Self-hosting is **also the metric most likely to mislead**, per §4.4(4). The right way to use it is as a *dogfooding discipline*, not as a *capability proof*. The claim "our agent writes 90% of our agent" tells you the team trusts it on their own fast-moving TypeScript codebase; it does not transfer to a bank's batch-processing COBOL estate.

**Layer:** **L — Lifecycle** (how the agent's own codebase evolves) and **V — Verification** (dogfooding as a verification strategy). Cross-ref [coding_agents_research.md](coding_agents_research.md) for how self-hosting shows up across the coding-agent field, and [agent_versioning_guide.md](agent_versioning_guide.md) for what changes when the agent's own repository moves under it.

### 5.5 Adoption dynamics: virality instead of mandate

**Evidence.** *"We didn't force anyone to use Inspect over their own tools. We built to people's needs, created virality loops through letting it work in public spaces, and let the product do the talking."* ✅ (Ramp Builders). Modal: *"Ramp didn't mandate Inspect."* ⚠. Ramp's Slack recommendation notes that a shared channel *"introduces a virality loop. As people in your organization use it, others will see it."* ✅. And sessions are public with no opt-outs ✅/⚠ (Pragmatic Engineer).

**Lesson.** Two mechanisms do the work, and they are complementary but different in kind:

- **Voluntary adoption via visibility.** The Slack bot and public sessions mean the agent is demonstrated continuously, by colleagues, in the flow of real work. Ramp describes the outcome as *"internal adoption charts have been vertical"* ✅.
- **Zero-setup as the adoption gate.** The v1 failure is the load-bearing evidence: *"People liked v1 but it wasn't adopted because engineers already knew how to go to a file and edit a single line of code, so didn't have a reason to use it, and it also required setting up a local development environment, making it too complicated for non-devs."* ✅ (Pragmatic Engineer). v2 removed the setup. Adoption followed.

**Lesson stated plainly: the constraint was friction, not capability.** V1 was liked. V1 was not used. The change between v1 and v2 was not a better model.

**Layer:** **G — Governance** (mandate vs. free choice is a governance posture, and *public-by-default with no opt-out* is a governance choice with privacy consequences). Cross-ref [ai_adoption_strategies_guide.md](ai_adoption_strategies_guide.md) for the adoption-mechanics side, and [agentops_guide.md](agentops_guide.md) for the operational surface that visibility creates.

**Guide's analysis.** "No mandate" and "no opt-out" are not in tension with each other, but they are in tension with a common misreading: this is *not* a purely libertarian adoption story. Nobody is forced to use the agent, and nobody can hide their sessions from colleagues. That combination is deliberately maximising learning and diffusion — and it is the first thing a regulated institution will have to renegotiate (§6).

### 5.6 Composability: the agent becomes a platform

**Evidence.** Inspect *"supports all frontier models, MCPs, custom tools, and skills that encode how we ship at Ramp"* ✅. More than 200 agents run on top of the Inspect platform ⚠, including ReviewBuddy (code review, built by a single engineer in a week), Oncall Assistant (incident triage across production and observability systems), Testo (browser-based QA that generates Playwright tests), Ramp Research (agentic data analyst over Looker, Snowflake, and dbt), Voice of the Customer (90 days of feedback as a Slack bot), and error automations that raise draft PRs from Sentry or Datadog alerts ✅/⚠. Inspect can spawn child sessions and read their status ✅. As the Linear profile puts it, a central team defines what agents can access and do *"so functional teams can build their own automations safely on the same rails."* ⚠ (Linear)

**Lesson.** The durable product is the **platform**, not the agent. Once sandbox creation, session state, auth, streaming, and PR plumbing exist, every additional agent is a prompt plus a tool list. A 5.5-person team supporting 200+ agents ⚠ is only possible because the first 190 agents did not need infrastructure work.

**Layer:** **L — Lifecycle** (agents as durable, versioned artifacts) and **O — Observability** (session status, usage stats as the platform's control plane). Cross-ref [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) and [agentic_engineering_guide.md](agentic_engineering_guide.md); for the multi-agent composition question, [multi_agent_banking_guide.md](multi_agent_banking_guide.md) and [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md).

### 5.7 The layers Ramp under-documents — O and V

**Evidence.** Ramp documents a **session statistics page** with org-wide usage, *"how many sessions result in a merged pull request"*, and a live *"humans prompting"* count based on users active in the last five minutes ✅ (Ramp Builders). It documents observability **integrations** (Sentry, Datadog) as context sources ✅. It documents verification behaviour (tests, telemetry, feature flags, screenshots, live previews) ✅.

**What it does not document.** No evaluation harness for the agent itself. No published merge-through rate. No regression gating for prompt or harness changes. No incident review. No trace format, no span taxonomy, no cost telemetry. See §3.7 and §11.

**Lesson.** Ramp's **Observability (O) is adoption-facing** — dashboards that inspire growth — rather than **assurance-facing** — dashboards that catch regressions. Its **Verification (V)** is *in-loop* (the agent proves its own work to itself before opening the PR) rather than *out-of-loop* (an independent check of the agent's aggregate behaviour). The first is excellent engineering; the second is the thing an auditor asks about.

**Layer:** **O — Observability** and **V — Verification.** Cross-ref [agentops_guide.md](agentops_guide.md) (tracing, cost, and evaluation telemetry), [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) (evaluation), and [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md) (out-of-loop verification tooling). This is the largest gap between the Ramp pattern and a deployable enterprise pattern, and §6 and §7 build it back in.

### 5.8 The harness-layer summary table

| Layer | Ramp Inspect's documented posture | Strength | Gap for a regulated adopter | Owning sibling guide |
| --- | --- | --- | --- | --- |
| **E — Execution** | Sandboxed VM per session on Modal; snapshot-restored; hundreds concurrent | Very strong | Egress policy, secret scoping, residency not published | [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) |
| **T — Tooling** | OpenCode harness; MCPs, custom tools, skills; hosted VS Code, terminal, VNC/Chromium | Very strong | Tool allow-lists and write-protection of prod systems not published | [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) |
| **C — Context** | Codebase + tickets (Linear) + docs + telemetry + feature flags + read-only prod replica | Very strong (claim) | Assembly, ranking, truncation, and staleness handling not published | [context_engineering_guide.md](context_engineering_guide.md) |
| **L — Lifecycle** | Sessions are durable, resumable, multi-client, multiplayer, snapshot-backed | Strong | Prompt/harness versioning and change control not published | [agent_versioning_guide.md](agent_versioning_guide.md) |
| **O — Observability** | Usage/statistics page; "humans prompting"; merge-rate framing; Sentry/Datadog as context | Adoption-facing | No assurance telemetry, no published merge-through rate | [agentops_guide.md](agentops_guide.md) |
| **V — Verification** | In-loop: tests, telemetry, feature flags, screenshots, live previews, browser driving | Strong, and ahead of most products at the time | No out-of-loop evaluation of the agent; no published regression gate | [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) |
| **G — Governance** | User-token PRs (no self-approval), per-prompt attribution, public-by-default sessions, no mandate | Strong on attribution; unusual on privacy | No data-residency, retention, or access-review evidence | [ai_governance_framework_guide.md](ai_governance_framework_guide.md) |

**Guide's analysis.** Four of seven layers are documented to a standard most vendors never reach. The three that are thin — O, V-out-of-loop, and the compliance half of G — are precisely the three that a bank's second line of defence asks about first. That is not an accident of publication; it is the natural centre of gravity for a company whose incentive is adoption.

---

## 6. Enterprise and Banking Relevance

Ramp is a fintech. It is not a bank, and it is deliberately not a regulated deposit-taker with prudential supervision, separation-of-duties requirements, and an examiner asking for evidence. That difference is not a technicality, and this section is where it gets cashed out.

**Everything in this section is the guide's analysis.** No source consulted discusses Ramp Inspect in a banking context. Where a recommendation is anchored in a Ramp claim, the claim is marked; the inference from claim to recommendation is the author's.

### 6.1 What a regulated institution should take

**Take 1 — The PR as the agent's terminal output artifact.** This is the highest-value transfer in the case study, and it is nearly free. Ramp chose to have the agent produce a reviewed pull request rather than a merge, a deploy, or a database write ✅. That choice maps directly onto a bank's existing change-management control: the change advisory board already reviews pull requests, already requires a second pair of eyes, and already produces an auditable record. **An agent that terminates in a PR inherits the control for free. An agent that terminates anywhere else needs a new control built.**

**Take 2 — Attribution at the token and prompt layer.** Ramp attributes each prompt that causes changes to the individual who sent it, and uses the *user's* GitHub token to open the PR rather than an application identity ✅. Two benefits a bank should notice: (a) the audit trail names a human, not a service account, and (b) it structurally prevents self-approval. That second property is the mechanism equivalent of a maker-checker control.

**Take 3 — Sandbox-per-session as a control, not just a performance trick.** Ramp implemented isolated, snapshot-restored, per-session environments for latency and concurrency ✅. For a bank the same architecture delivers: no agent action reaches another agent's workspace, no agent state leaks between sessions, and — critically — **the sandbox is the natural enforcement point for egress control, secret scoping, and residency.** None of those three are published by Ramp (§3.7); all three are implementable in the same substrate. Cross-ref [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) §9 on network isolation, which is the missing layer in the Ramp account.

**Take 4 — Context grounding from *your own* systems, with metadata over records.** Inspect's advantage is that it can trace a customer request through a product spec to the relevant code ✅. A bank can reproduce the shape of that without shipping customer records into a model context window: ground on **ticket metadata, schema definitions, data dictionaries, service catalogues, runbooks, and the codebase** — and never on the underlying customer data. The grounding architecture is the transferable asset; the specific data sources are not.

**Take 5 — In-loop verification before human review.** Ramp's agent runs tests, reads telemetry, queries feature flags, and visually confirms frontend changes before it opens the PR ✅. That is the single biggest lever on review load, and it is exactly what a bank's change process wants: fewer, higher-quality submissions. Cross-ref [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) for what "verification" has to mean before it can be relied upon.

**Take 6 — The platform abstraction, only after the third agent.** Ramp's 5.5-person team maintaining 200+ agents ⚠ is a real result, but it is the *reward* for having built the rails first. A bank building a review agent, an incident agent, and a test-generation agent will get value from the platform — but building the platform before the second agent is a classic way to spend a year on plumbing. Cross-ref [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md).

### 6.2 What a regulated institution should NOT take

| Ramp practice | Why it cannot transfer as-is | What to do instead |
| --- | --- | --- |
| **Public-by-default sessions with no opt-out** ✅ (Pragmatic Engineer) | Every session prompt, thread, and diff becomes visible to all staff. In a bank, prompts will contain incident details, customer identifiers, ticket text, and unreleased product plans. Openness-by-default is a data-classification decision made implicitly. | **Private-by-default with explicit share**, plus a classifier that blocks sessions whose prompts or retrieved context contain restricted data classes. Keep the multiplayer *capability*; invert the default. |
| **Sanitized read-only production DB replica in the agent's reach** ✅ (Pragmatic Engineer) | "Sanitized" is undefined publicly. A replica of production data inside an agent workspace is a residency, retention, and access-review finding waiting to happen — even read-only, even sanitized, even in a sandbox. | Ground on **schema plus a governed, masked, sample-limited extract** with a distinct data-classification label, or on a purpose-built debug dataset. Require the sanitisation to be a documented, tested control. |
| **Snapshots retaining cloned source for up to 30 days by default** ✅ (Modal docs) | The default retention is a vendor default, not a bank's retention schedule. Snapshots also contain build artefacts, dependency caches, and whatever the agent wrote into the filesystem. | Set explicit TTLs, encryption with customer-held keys where supported, and a documented deletion path — Modal supports configurable TTL and customer-supplied encryption keys ✅ (Modal docs). Then prove deletion. |
| **"Frontier models are smart enough to contain themselves"** ✅ (Ramp quoting its own spec) | This is a statement about model behaviour, not about spend or blast radius. A runaway spawn loop is a financial-control event in a bank. | **Hard caps**: max child sessions per parent, max concurrent sessions per user, max tokens per session, max wall-clock, and a budget alarm. Wrap the agent's spawn tool in a quota. |
| **"Effectively free to run"** ✅ (Ramp Builders) | Internal cost invisibility is the mechanism by which agent spend becomes a surprise. A bank's CFO function will not accept an unbudgeted, unattributed compute line. | **Per-session cost attribution** from day one, charged back to the requesting cost centre. This is also the only way to compute a real cost-per-merged-PR (§7). |
| **No published evaluation harness for the agent itself** (§3.7) | A bank must evidence that a change to a prompt, model, or tool list did not degrade behaviour. Without an eval suite, every harness change is an uncontrolled change to a production control. | Gate prompt/harness changes on an **evaluation suite with a frozen golden set**, and version both. Cross-ref [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md), [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md), [agent_versioning_guide.md](agent_versioning_guide.md). |
| **"Share of merged PRs" as the headline metric** (§4.4) | It measures process share, not value, and it is compatible with rising human review load. Reporting it to a board would be reporting an adoption curve as an efficiency result. | Report **merge-through rate, review-hours per merged PR, rework rate, and escaped-defect rate** together. See §7.5. |

### 6.3 The three questions a second line of defence will ask

**1. Data residency and training-data exposure.** Where does the code go, where does the prompt go, and is any of it used to train a model? Ramp publishes none of this (§3.7). A bank's answer must be contractual and architectural: a zero-retention or no-training agreement with every model provider in the loop, region-pinned sandboxes and snapshots, and a documented egress policy for the sandbox. The Ramp case study establishes the *capability*; it does not establish the *control*. Cross-ref [ai_governance_framework_guide.md](ai_governance_framework_guide.md) and [llm_guard_models_guide.md](llm_guard_models_guide.md).

**2. Separation of duties and evidence.** Ramp's user-token design gets you the maker-checker gap closed at the mechanism level ✅ — but a bank additionally needs: a record of *which model version and which prompt* produced each diff, an immutable log of tool calls made inside the sandbox, and a reviewed, versioned definition of what the agent is permitted to do. The Ramp account gives you the first three words of that sentence ("use the user's token") and stops. Cross-ref [agentops_guide.md](agentops_guide.md) for the trace/evidence plumbing, and [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md) for the assurance loop.

**3. Ops resilience when the agent is wrong.** Ramp publishes no incident post-mortem, no rollback description, and no statement that an Inspect PR has ever caused a production incident (§3.7, §11). A bank cannot adopt an agent into its change pipeline without an answer to: *what is the kill switch, what is the rollback, who is on the hook at 3am, and how is the agent implicated in the incident timeline?* This is the same discipline as any other change to the pipeline, and it is best borrowed from the resilience content rather than invented. Cross-ref [ai_agent_drift_guide.md](ai_agent_drift_guide.md) (drift as a resilience concern), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) (failure taxonomy), and [agentic_engineering_guide.md](agentic_engineering_guide.md) for the operating discipline.

### 6.4 The one-line verdict

**Guide's analysis.** *Ramp Inspect is a credible blueprint for the **execution, tooling, and context** layers of an internal coding agent, and a poor blueprint for the **governance, observability, and out-of-loop verification** layers — because Ramp's incentives shaped exactly which layers it documented and which it hardened. A regulated adopter should copy §3's architecture almost wholesale and rebuild §3.7's omissions from scratch.*

---

## 7. Worked Example: The Cymbal Bank Internal Coding-Agent Pilot

> **This section is explicitly illustrative.** No part of it describes a real deployment, a real vendor, a real repository, or a real metric. Cymbal Bank is a fictional institution used throughout this repo as a standing example; the numbers below are *placeholders showing what would be measured and how*, not observed results. Anyone reading this section for evidence has misread it.

### 7.1 The scope

Cymbal Bank pilots an internal background coding agent — call it **Cymbal Inspect** for the purposes of this sketch — modelled on the Ramp pattern but scoped to one bounded, low-risk surface.

| Dimension | Pilot scope | Rationale |
| --- | --- | --- |
| **Repositories** | Three internal-tooling repositories (an internal React admin console, a Go service for internal approvals, and a Python reporting job). **No** payment rails, **no** core ledger, **no** customer-facing origination. | Matches Ramp's "start where the code is yours and the blast radius is internal" trajectory — though Ramp's starting repositories were production product code, not internal tools ✅ |
| **Teams** | Two engineering squads (~24 engineers) plus three product managers and two designers | Mirrors Ramp's multi-role builder story ✅ at a size where a weekly review is possible |
| **Duration** | One quarter (13 weeks), with a hard stop and a go/no-go | Adoption stories compress; a quarter is long enough for the metrics in §7.5 to have a baseline |
| **Users** | Opt-in. Volunteer squads only. | Deliberately the inverse of Ramp's no-mandate-but-public-default posture; the pilot buys statistical cleanliness at the cost of viral spread |

**Explicitly out of scope for the pilot:** any repository that deploys to a customer-facing environment; any agent access to production data of any classification; any autonomous merge or deploy.

### 7.2 Sandbox architecture

The architecture borrows §3's shape and adds the controls §3.7 omits.

| Component | Pilot design | Borrowed from | Added |
| --- | --- | --- | --- |
| **Execution substrate** | Sandboxed VM per session, in a private, region-pinned cloud tenancy | Ramp/Modal sandbox-per-session ✅ | Region pinning; no cross-region snapshot replication |
| **Environment images** | Prebuilt per repository on a schedule; sessions start from a filesystem snapshot | The 30-minute image-rebuild + snapshot pattern ✅ | Snapshot TTL set explicitly to 7 days, not the vendor 30-day default ✅; customer-managed keys where available |
| **Agent harness** | An open-source server-first coding agent with a typed SDK and a plugin hook on tool execution | Ramp's choice of OpenCode and its reasons ✅ | Tool allow-list enforced in the plugin, deny-by-default |
| **Context** | Codebase, internal docs, ticket metadata, schema definitions, service catalogue. **No** customer records, **no** production data rows | Ramp's full-context principle ✅ | Metadata-only grounding; classification check on every retrieval |
| **Egress** | Sandbox may reach: the bank's Git host, the artifact registry, the model gateway, the internal docs API, the ticketing API. Everything else denied | Not in the public Ramp account (§3.7) | Full egress allow-list with logging |
| **Secrets** | Scoped, short-lived tokens per session; no long-lived credentials in the sandbox | Ramp's per-clone app-installation token ✅ | Per-session expiry; no production credentials ever issued |
| **Observability** | Every tool call, prompt, and diff traced to a session ID; cost attributed per session | Ramp's session statistics page ✅ | Assurance telemetry, not just adoption telemetry |
| **Verification** | Agent runs the repo's tests, linters, and type checks before opening a PR | Ramp's in-loop verification ✅ | Plus an out-of-loop nightly evaluation suite on a frozen golden task set |

### 7.3 Context scope — the part that has to be decided explicitly

The single biggest difference between the Ramp pattern and a bank pattern is not the sandbox. It is **what goes into the context window**, and it must be a documented decision with a named owner, not an emergent property of which MCP servers got connected.

**Guide's analysis.** The pilot's context policy, stated as a rule rather than as a diagram:

- **In scope:** source code and its history; internal documentation; ticket *metadata* (title, labels, links, status) rather than ticket bodies where bodies may contain personal data; database *schemas* and data dictionaries; service catalogue and dependency graph; build and CI logs; sanitised error and trace *summaries*.
- **Out of scope:** any customer record; any row-level production data; any credential; any document classified above the pilot's ceiling; any content from a system of record that has not been through the classification gate.
- **Ambiguous, needs a ruling:** incident post-mortems (often contain customer detail), support tickets (almost always contain personal data), and design files (usually safe, occasionally contain unreleased strategy). The pilot's governance forum rules on these in week 1, and the ruling is written down.

Ramp's answer to this was *connect everything a Ramp engineer can see* ✅. That is defensible for a fintech with an internal agent and a strong enough culture. It is not a defensible default for a supervised institution, and the reason is not cultural caution — it is that the bank must be able to evidence *why* each data source was in scope.

### 7.4 The PR-based interface and the governance gates

The interaction model is copied deliberately: hand work over as a prompt, get it back as a pull request.

```
Engineer / PM / designer
  → prompt (Slack, web console, or IDE-adjacent client)
    → session created in a Cymbal sandbox (region-pinned, egress-controlled)
      → agent reads context (allow-listed sources only), edits files
        → agent runs tests + linters + type checks in-sandbox
          → agent opens a PR using the REQUESTER'S identity token
            → standard change process: two reviewers, one of whom must be a code owner
              → CI, SAST/DAST, dependency scan (unchanged bank controls)
                → merge by a human
```

Five gates, and each one exists for a stated reason:

| Gate | Enforcement point | Prevents |
| --- | --- | --- |
| **G1 — Tool allow-list** | Harness plugin intercepting tool execution before dispatch | The agent reaching a system nobody approved |
| **G2 — Context classification check** | Retrieval layer, before data enters the prompt | Restricted data entering a model context window |
| **G3 — No self-approval** | PR opened with the requester's token, never a service identity (the Ramp mechanism ✅) | A single person, or the agent, approving its own change |
| **G4 — Human merge only** | Branch protection: agents cannot merge | Unreviewed code reaching a protected branch |
| **G5 — Version pinning** | Prompt, harness, and tool definitions are versioned artifacts | A silent behaviour change from an unversioned prompt edit |

**Guide's analysis.** G3 is the gate the case study hands you. G5 is the gate the case study does not mention and which a bank cannot skip: if the prompt is edited by an engineer on a Tuesday afternoon with no review, then the change-control regime governing the agent is weaker than the regime governing the code it writes. That asymmetry is the most common governance failure in internal-agent programmes, and it is invisible until the first incident.

### 7.5 The metrics that would actually be tracked

Ramp's published metrics are adoption metrics. A pilot that wants a go/no-go needs efficacy metrics, and it needs them with a baseline.

| Metric | Definition | Why it is tracked | Ramp analogue |
| --- | --- | --- | --- |
| **Merge-through rate** | Sessions that result in a merged PR ÷ sessions started | Ramp itself calls this "the most important metric to track" ✅ but publishes no value | The missing number (§4.4(5)) |
| **Cost per merged PR** | Total session compute + model spend ÷ merged agent PRs | Makes "effectively free" ✅ falsifiable | Not published ❌ |
| **Review-hours per merged PR** | Reviewer time on agent PRs vs human PRs, from the ticketing integration | Tests the "bottleneck moved to review" claim ✅ | Not published ❌ |
| **Rework rate** | Share of agent PRs receiving a substantive (non-style) review change request | Distinguishes "PR opened" from "PR useful" | Not published ❌ |
| **Escaped-defect rate** | Defects attributable to agent-authored changes that reach a production environment, per 100 merged PRs, vs the human baseline | The only metric a regulator cares about | Not published ❌ |
| **Human takeover rate** | Agent PRs where the original author rewrote the change locally before merge | Measures the "kick-start then take over" pattern Ramp describes ✅ | Not published ❌ |
| **Adoption breadth** | Distinct users who have merged at least one agent PR; split engineer / PM / designer | Tests Ramp's multi-role claim ⚠ for transferability | Ramp publishes daily unique users ⚠ |
| **Time-to-first-PR** | Median minutes from session start to PR opened | The latency property the whole snapshot design exists to buy ✅ | "Under 5 seconds" to environment ⚠ |

**Guide's analysis.** Note what happened to the headline metric: **"share of merged PRs" is not in the pilot's table.** It is deliberately excluded, because in a 24-engineer pilot it is trivially gameable (raise lots of tiny PRs with the agent) and it answers a question nobody in the go/no-go meeting is asking. Adoption share can be *reported* — it is useful context — but it cannot be the *decision* metric. Ramp's own framing explains why: it is the metric you put on a dashboard "so you can inspire more growth" ✅. Inspiring growth and justifying a control change are different jobs.

### 7.6 What would NOT be permitted in the pilot

Stated plainly, because the boundary is the deliverable:

- **No autonomous merge.** Not at any point, not even for documentation repositories.
- **No autonomous deploy.** The pipeline terminates at a merged PR. Deployment stays with existing automation and existing approvals.
- **No write access to any production system** from inside a sandbox — including read-write database credentials, feature-flag toggles in production, or message-broker publish rights. Ramp's agent *queries* feature flags ✅; querying is not toggling a bank's production flag.
- **No customer data in any prompt, retrieval, or snapshot** — including "anonymised" exports, unless the anonymisation is a tested, documented control with an owner.
- **No agent access to the payments, ledger, or origination code** for the duration of the pilot.
- **No unattended sessions against a production-adjacent repository overnight.** Ramp advertises the "kick off a session and check the PR in the morning" workflow ✅; the pilot requires that any session targeting a deployable repository is bounded by a session cap and reviewed before its branch is pushed to the protected remote.
- **No unlimited child-session spawning.** Hard caps on fan-out, concurrency, tokens, and wall-clock — explicitly rejecting *"frontier models are smart enough to contain themselves"* ✅ (Ramp Builders) as a sufficient control.
- **No public-by-default sessions.** Inverted from Ramp's posture ✅.
- **No prompt or harness change without review.** Gate G5 is not optional.
- **No model in the loop without a no-training, zero-retention agreement** and a documented data-processing footprint.

### 7.7 The go/no-go question

**Guide's analysis.** The pilot succeeds if, at week 13: merge-through rate is high enough that the sessions are not wasted effort; cost per merged PR is lower than the fully-loaded cost of the equivalent human change; review-hours per merged PR is flat or down; escaped-defect rate is at or below the human baseline; and at least one non-engineer merged a change. Any one of those missing turns the programme from "adopt" into "narrow the scope." **Note that five of those five criteria are unanswerable from Ramp's published data** — which is the point of §4.4 and the reason this section exists.

---

## 8. Positioning: Ramp Inspect Against the Public Alternatives

This section deliberately does **not** re-derive the coding-agent landscape — that is the job of [coding_agents_research.md](coding_agents_research.md), which covers the product field, the harness taxonomy, and the vendor positioning in far more depth than a single case study should. What follows is the narrow question this guide is qualified to answer: **what is Ramp Inspect's distinctive position, and which axis is it actually differentiated on?**

The Pragmatic Engineer frames the category precisely: at a select few companies, internal agents *"write most of their code"* — Inspect at Ramp, Goose at Block, Minions at Stripe, River at Shopify — which is a different thing from *"most of the industry which uses AI coding agents and harnesses like Codex, Claude Code, Cursor, OpenCode, GitHub Copilot."* ⚠ (Pragmatic Engineer)

### 8.1 The comparison

**Guide's analysis — the axis that matters.** Every product in this space can be positioned on one axis that predicts nearly everything else: **does the vendor own the harness, or does the customer?** Ramp moved a step to the right of every product category below, and that single move explains the sandbox-per-session design, the 200+ downstream agents, the designer/PM client surface, and the omission of published assurance telemetry alike.

| Class | Representative products | Where execution happens | Who owns the harness and the context wiring | Verification reach | Ramp Inspect's relationship to it |
| --- | --- | --- | --- | --- | --- |
| **Local IDE agents** | Cursor, Claude Code, Copilot in-editor ✅ named as the mainstream in the field | The developer's laptop | Vendor owns the harness; customer owns nothing but config | Whatever the developer's local setup already had | Inspect explicitly rejected this class for **concurrency limits** — *"only being able to run one or two sessions on local machines"* ⚠. Also the class Inspect must beat on latency to be adopted at all ✅ |
| **Vendor background / cloud agents** | Codex-class and Jules-class cloud agents; the GitHub Copilot coding agent ✅ named in the same field | Vendor's cloud | Vendor owns harness; customer supplies repo access and some MCP/tool config | Limited by what the vendor can be wired into — the gap Ramp points at directly ✅/⚠ | Closest architectural analogue (cloud, async, PR-shaped output) but the harness stays the vendor's, so the internal-tooling wiring and the client surface are not the customer's to change |
| **Devin-class autonomous SWE products** | Cognition Devin and successors | Vendor's cloud, with a proprietary workspace | Vendor owns harness; customer owns prompts and integrations | Vendor-defined | Same async/PR-shaped delivery; the difference is that Inspect's verification reach is *into Ramp's own* Sentry, Datadog, LaunchDarkly, and read-only prod replica ✅ — a vendor product cannot ship those integrations per-customer at parity |
| **Spec/plan-to-PR workspace class** | GitHub Copilot Workspace-class flows; issue-tracker-native agents | Vendor cloud | Vendor owns harness | Repo-scoped | Ramp's Linear integration produces a *similar* grounding effect (ticket context into code) ✅/⚠, but by wiring the tracker into its own harness rather than adopting a workspace product |
| **Open-source SWE-agent lineage** | SWE-agent, OpenHands, Aider, OpenCode and peers | Whatever you deploy it on | **You own the harness** | Whatever you build | This is the class Ramp actually built on: OpenCode is Inspect's harness ✅. Ramp's contribution is the *platform around* the open-source agent — sandboxes, snapshots, session state, clients — not the agent loop itself ✅ |
| **Ramp Inspect** | Ramp | Modal Sandboxes, one VM per session ✅ | **Ramp owns the harness, the context wiring, and every client** ✅ | Deep: tests, telemetry, feature flags, screenshots, live previews, browser driving, read-only prod replica ✅ | — |
| **Cymbal Bank pilot (§7)** | Illustrative | Private, region-pinned sandboxes in the bank's own tenancy | Bank owns harness + wiring + client defaults | Deep in-loop; adds out-of-loop evaluation (§7.2) | Same class as Inspect, one governance layer thicker |

### 8.2 The honest verdict on differentiation

**Guide's analysis.** Ramp Inspect is **not differentiated on intelligence** — Ramp says so itself, capping the agent at model capability ✅ — and it is **not differentiated on the sandbox**, since Modal sells that to anyone. Its differentiation sits in three places:

1. **Depth of internal grounding**, which is a function of being inside the company, not of the product.
2. **Breadth of the client surface** — Slack, web, VS Code in the sandbox, Chrome extension, mobile, voice ✅ — which exists because Ramp owns the harness and can therefore put a client anywhere.
3. **The platform downstream**: 200+ agents on the same rails ⚠, which is a compounding advantage no point product offers.

**The corollary that matters for anyone evaluating this as a template.** Two of those three advantages are available to any organisation willing to own its harness; only the first requires being Ramp. Which means the actionable question is not "should we copy Inspect?" but **"is the open-source-harness-plus-our-own-platform path cheaper for us than the vendor path?"** — and for most banks the honest answer starts as *vendor path first, platform only if the vendor path demonstrably cannot reach our internal systems.*

---

## 9. One-Page Summary

**What it is.** Ramp Inspect is Ramp's internal background coding agent: a coding agent that runs in a sandboxed cloud VM per session, wired into Ramp's own internal systems, and delivers its work as a pull request. It shipped internally in November 2025, after a first version that existed only as a Chrome extension for designers. ✅ (Ramp Builders; Pragmatic Engineer)

**The headline claim.** As reported, Inspect writes a majority of Ramp's merged pull requests: ~30% in January 2026 ✅ (Ramp, first-party), "over half" in February 2026 ⚠ (Modal, vendor), and 75% by May 2026 ⚠ (Pragmatic Engineer, single-source). Also reported: ~90% of PRs merged into the Inspect repository itself came from an Inspect session ⚠, 150+ engineers have contributed to the codebase ⚠, and the whole system is maintained by 5.5 people ⚠.

**The architecture, as published.** One Modal Sandbox per session containing Postgres, Redis, Temporal, RabbitMQ, Vite and the rest of the local dev environment ✅. Repository images rebuilt every 30 minutes on a cron; sessions restore from a diff-based filesystem snapshot at most 30 minutes stale ✅. OpenCode as the agent harness ✅. Cloudflare Durable Objects and per-session SQLite for session state and multiplayer ✅. Pull requests opened with the **invoking user's** GitHub token, never an app token, specifically to prevent self-approval ✅. Snapshots taken again at session end so follow-ups resume ✅.

**The stated reasoning.** Third-party harnesses could not run enough sessions in parallel on local machines; the web team wanted designer-grade frontend tooling; the organisation needed remote dev environments as it scaled; and — the only genuinely in-house argument — *"Owning the tooling lets you build something significantly more powerful than an off-the-shelf tool will ever be. After all, it only has to work on your code."* ✅

**What is genuinely instructive.** (1) Move environment preparation off the critical path with scheduled image builds and snapshots. (2) Ground the agent in internal systems or accept that it is a generic agent. (3) Make the PR the terminal artifact, opened with the user's identity — the single most transferable decision in the case study. (4) Remove setup friction, not capability: the v1→v2 change was the pivot that produced adoption. (5) Treat the harness as a platform once the rails exist.

**What the metrics will not tell you.** PR share measures process, not value; it counts one-line fixes and migrations equally; it conflates "started by", "authored by", and "written by"; it says nothing about review load, which the sources concede has *moved* rather than disappeared ⚠; the Inspect-repo statistic is self-referential; and Ramp's own preferred metric — merge-through rate — is never published ✅/❌. See §4.4.

**What a bank should take.** The PR-shaped interface with user-token attribution; sandbox-per-session as an isolation control; metadata-over-records grounding; in-loop verification before human review. **What a bank should not take:** public-by-default sessions with no opt-out; a raw production-data replica in the agent's reach; vendor-default snapshot retention; "the models will contain themselves" as a spend control; and share-of-PRs as a management metric.

**What is not public.** Egress policy, secret scoping, data residency and retention configuration, the sanitisation controls on the production replica, the context-assembly pipeline, any evaluation harness for the agent itself, cost per session, rollback behaviour, and any incident history. §11 lists these. **The shell is published; the brain is not.**

---

## 10. Claims Audit

Every substantive claim in this guide, adjudicated. "Verified" here means *the claim is accurately attributed and stated in a primary source* — **not** that the underlying number has been independently audited. No independent audit of any Ramp figure exists.

### 10.1 Verified ✅

| Claim | Source | Note |
| --- | --- | --- |
| Inspect is Ramp's internal background coding agent, shipped internally November 2025 | Ramp Builders (12 Jan 2026); Pragmatic Engineer | Both agree on the date |
| One sandboxed VM per session with a full dev environment (Postgres, Redis, Temporal, RabbitMQ, Vite) | Ramp Builders; Modal | Enumerated identically by both |
| Repository images rebuilt every 30 minutes; sessions start from a snapshot | Ramp Builders; Modal | The core latency mechanism |
| Filesystem snapshots are stored as diffs from the base image; only modified files persist | Modal docs | Vendor documentation of the mechanism, unambiguous |
| Snapshots default to 30-day TTL (filesystem/directory) and 7-day (memory); TTL is configurable | Modal docs | First-party docs |
| OpenCode is the agent harness | Ramp Builders; Pragmatic Engineer | Named explicitly by Ramp |
| PRs are opened using the invoking user's GitHub token, explicitly to prevent self-approval | Ramp Builders | First-party, with the stated rationale |
| Per-prompt authorship attribution is passed through to the agent | Ramp Builders | First-party |
| Git identity is set at commit/push because git operations are not tied to a GitHub user | Ramp Builders | First-party |
| Client surface: Slack, web, hosted VS Code, Chrome extension, PR discussion; plus voice | Ramp Builders | Enumerated by Ramp |
| Integrations: GitHub, Slack, Buildkite, Sentry, Datadog, LaunchDarkly, Braintrust | Ramp Builders | The full list is Ramp's |
| Backend verification = tests + telemetry + feature flags; frontend verification = screenshots + live previews | Ramp Builders; Pragmatic Engineer | Consistent across both |
| Debugging uses a sanitized read-only production DB replica and Snowflake queries | Pragmatic Engineer | Single source but specific |
| Inspect can spawn child sessions and read their status | Ramp Builders | First-party spec |
| ~30% of merged PRs as of 12 January 2026 | Ramp Builders | First-party, dated |
| The v1 (Chrome extension) → v2 (remote dev environment) pivot is what produced adoption | Pragmatic Engineer | Specific, and self-critical enough to be credible |

### 10.2 Flagged ⚠

| Claim | Source | Why flagged |
| --- | --- | --- |
| "Over half of all merged pull requests at Ramp" | Modal (vendor) | Vendor's headline; the number exists to sell sandboxes; the linked X post is the only citation |
| 75% of merged PRs (May 2026) | Pragmatic Engineer (single source) | Appears once; restated by Linear, which is quoting the same story, not corroborating it |
| ~90% of PRs merged into the Inspect repo come from an Inspect session | Pragmatic Engineer (single source) | Self-referential by construction (see §4.4(4)) |
| 150+ engineers have contributed to the Inspect codebase | Pragmatic Engineer (single source) | No corroboration; no definition of "contributed" |
| More than 80% of Inspect is written by Inspect | Modal + Pragmatic Engineer | Two publications, **same interview lineage** — not independent |
| "Hundreds of concurrent sessions" | Modal (vendor) | No peak, mean, or definition |
| "Under 5 seconds" to a provisioned environment | Pragmatic Engineer (single source) | Consistent with Ramp's design intent but not measured publicly |
| One million total sessions by July 2026 | Pragmatic Engineer (single source) | No definition of "session" |
| 5.5-person Inspect team (four engineers, a director, part-time PM) | Pragmatic Engineer (single source) | Specific enough to be real; still uncorroborated |
| PMs and designers ship product changes directly | Modal (vendor) | Directionally corroborated by Ramp and Linear; **no usage numbers by role published anywhere** |
| "100% of Inspect is never limited by missing context or tools" | Ramp Builders (first-party aspiration) | An aspiration, not a measurement. The published list of integrations is evidence of intent, not of completeness |
| Inspect put Ramp "months ahead of nearly all AI coding harnesses" | Pragmatic Engineer | Editorial judgement, not a measurement |
| Ramp company size ~1,600 | Linear customer profile | Vendor profile field; unaudited; conflicts with other public headcount estimates |
| Rahul Sengottuvelu's title | Modal vs Pragmatic Engineer | Modal says **Head of Applied AI**; Pragmatic Engineer says **CTO**. Unreconciled. He is a co-author of Ramp's own post either way |
| "GPT 5.2 with no reasoning" for the Slack repo classifier | Ramp Builders | First-party and specific, but the model version cannot be confirmed from outside Ramp's stack |
| Six years of specs, requests, and roadmaps in Linear as context | Linear (vendor) | Vendor framing of a genuine integration; Ramp's own post does not quantify it |

### 10.3 Rejected ❌

| Claim | Why rejected |
| --- | --- |
| "~1,000 engineers at Ramp" | **No source found states this.** See §11. Do not use |
| "Inspect writes half of Ramp's *code*" | Sources say half of **merged PRs**. Code volume and PR count are not interchangeable, and no source measures lines, complexity, or value |
| "Ramp replaced X engineers with Inspect" | No source makes a headcount claim in either direction, and no headcount reduction is reported |
| "Inspect reduces cost by N%" | No cost figures are published at all; only the qualitative *"effectively free to run"* ✅ |
| "Inspect has never caused a production incident" | Absence of reporting is not a finding. No incident data is published in either direction |
| "Inspect is model-agnostic, therefore it uses no specific model" | Ramp names GPT 5.2 for one step ✅ and says frontier models are supported ✅; the production split is undisclosed, not absent |
| Any specific cold-start millisecond figure | Only *"under 5 seconds"* ⚠ and *"a few seconds"* ⚠ are published |
| Any inferred sandbox network policy | Not published. Do not infer (see §3.7) |
| "Other companies' internal agents (Goose, Minions, River) work like Inspect" | Named in the same category by the Pragmatic Engineer ⚠ but not compared on architecture by any source. Cross-ref [coding_agents_research.md](coding_agents_research.md) instead of assuming equivalence |

---

## 11. What Could Not Be Verified

This is the list the guide promised in §3.7, restated as the honest boundary of the research. **None of the following could be verified from any source consulted, and none of it is inferred anywhere in this guide.**

**Undisclosed internals — the system**

1. **Sandbox network egress policy.** What the agent can reach outbound, whether an allow-list exists, and whether the agent can make arbitrary internet requests from inside the sandbox.
2. **Secrets management.** How credentials for Sentry, Datadog, LaunchDarkly, Braintrust, Snowflake, and the production database replica are injected, scoped, rotated, and revoked per session.
3. **Snapshot encryption, residency, and deletion.** Where snapshots live geographically, under whose keys, and what the deletion guarantee is. Modal documents customer-supplied encryption keys and configurable TTLs ✅; Ramp's configuration of either is not published.
4. **The sanitisation controls on the read-only production replica.** Ramp says "sanitized" ✅ and stops. What is masked, what is retained, who owns the control, and how it is tested are all unstated.
5. **The context-assembly pipeline.** How codebase, documentation, tickets, telemetry, and flag state are selected, ranked, deduplicated, truncated, and budgeted into a prompt. This is the single most consequential omission, because grounding is the capability Ramp credits for the results.
6. **Retrieval staleness handling beyond the 30-minute git window.** What happens when the relevant context is stale in a way git sync does not fix (stale docs, stale tickets, stale flag state).
7. **Model routing and the production model mix.** Which models write the majority of the code, how tasks are routed between them, and whether the choice is per-task, per-user, per-repository, or per-session.
8. **Token and cost budgets per session**, and whether any budget enforcement exists.
9. **Rate limiting, quotas, and spend caps** on session creation, particularly on self-spawned child sessions.
10. **Rollback and kill-switch behaviour.** How an in-flight session is terminated and how its branch is withdrawn.
11. **Whether the agent has ever caused a production incident**, and what any incident review concluded.
12. **The prompt and harness change-control process.** Whether prompt edits are reviewed, versioned, or evaluated before deployment.

**Undisclosed internals — the metrics**

13. **Merge-through rate** — sessions that result in a merged PR, as a fraction of sessions started. Ramp calls this "the most important metric to track" ✅ and publishes no value.
14. **Definition of "session"** in the one-million-sessions figure, and whether abandoned or failed sessions are counted.
15. **Definition of "merged PR"** in the headline percentage — whether the numerator is "started by", "authored by", or "written by" Inspect (§4.3).
16. **Total merged PR volume**, and therefore what the 75% is a share *of*.
17. **The exclusion set** — which repositories are in the "frontend and backend repos" denominator, and whether platform, data, infrastructure, and mobile repositories are included.
18. **Review-load data** — reviewer hours per agent PR versus human PR, which is the number that would settle whether the bottleneck merely moved.
19. **Rework and takeover rates** — how often an agent PR required substantive human rework before merge.
20. **Defect data** — escaped defects attributable to agent-authored changes.
21. **Usage split by role** — how many of the sessions, PRs, or merged changes came from PMs and designers versus engineers. Ramp asserts multi-role use ✅/⚠; no breakdown exists.
22. **Cost per merged PR**, or any total spend figure for Inspect.
23. **Ramp's engineering headcount.** The dispatcher's working figure of ~1,000 engineers **could not be confirmed or refuted**; Linear's customer profile lists a company size of 1,600 ⚠ with no definition of what that counts (all staff? US only? product and engineering?). Treat any engineers-per-Inspect ratio as uncomputable.
24. **Ramp's ARR and valuation** as of the case-study period. Third-party estimates circulate ($1B+ ARR; valuations in the $32B–$44B range across 2026 reporting, from sources such as GetLatka and fintech trade press) but they are **unaudited third-party estimates, not company-confirmed figures**, and they are not needed for any claim in this guide. Deliberately not used.

**Editorial boundary**

25. **The paywalled portion of The Pragmatic Engineer article.** The architecture and tech-stack section of that article sits behind a paid tier; the retrievable portion ends mid-section at "Inspect's core principle." Everything this guide attributes to that article comes from the readable portion (headline figures, the four use cases, adoption numbers, the sandbox-internals summary bullet, and the platform section) — the deeper stack detail was **not** readable and is not reconstructed here. The stack description in §3.1 rests on Ramp's own post and Modal's, both fully readable.

**How to use this list.** If you are citing this guide, cite §3 and §4 for what Ramp published, and cite §11 for what Ramp did not. **A "not disclosed" is a finding.** It is the difference between "we could not find the egress policy" and "there is no egress policy" — and only the first is true.

---

## 12. Glossary

| Term | Definition |
| --- | --- |
| **Background agent** | A coding agent that runs asynchronously on remote infrastructure rather than in the developer's editor, so work continues without an open session and without consuming the developer's machine. Inspect is Ramp's. |
| **Inspect** | Ramp's internal background coding agent: sandbox-per-session execution, grounded in Ramp's internal systems, delivering work as pull requests. Shipped internally November 2025. |
| **Inspect v1 / v2** | v1 was a Chrome extension for designers making visual UI edits; v2 (November 2025) was the remote-development-environment-plus-coding-agent pivot that produced adoption. |
| **Sandbox** | An isolated execution environment — here, a full-stack development VM — containing the services an engineer would run locally. One per session means no cross-session interference and a natural control point for egress, secrets, and residency. See [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md). |
| **Snapshot** | A saved copy of a sandbox's state that a later sandbox can be created from. Modal's **filesystem snapshots** are stored as diffs from the base image and restore using the same infrastructure as cold starts, which is why they are fast. Default TTL 30 days (filesystem/directory), 7 days (memory). |
| **Cold start / spin-up time** | The time from "user submits a prompt" to "a provisioned environment is working on it." Ramp's design target was parity with an already-warm local checkout; the published figure is *"under 5 seconds"*. |
| **Warm pool** | A set of pre-created sandboxes kept ready for high-volume repositories, expired and rebuilt as new environment images arrive, to shave the last seconds off spin-up. |
| **Image registry** | Ramp's per-repository definition of the environment image, rebuilt on a 30-minute cron with dependencies installed and initial builds cached. |
| **OpenCode** | The open-source, server-first coding agent used as Inspect's harness. Chosen for its typed SDK, plugin system, HTTP API, and model agnosticism — clients become thin layers on top of it. |
| **PR-shaped interface** | The design choice to make the agent's terminal artifact a pull request — opened with the requesting user's identity token, reviewed by humans, merged by humans — rather than a chat answer, a merge, or a deploy. It lets the agent's output inherit the organisation's existing change control. |
| **Multiplayer session** | A session that multiple people can prompt and collaborate in simultaneously, with per-prompt attribution. Ramp treats this as mission-critical and not present in other products. |
| **Virality loop** | Ramp's deliberate adoption mechanism: because sessions are visible and the agent works in shared spaces like Slack channels, colleagues see it working and adopt it. |
| **Harness** | The scaffolding around a model that makes it an agent: the loop, the tool protocol, the context assembly, the policies. The harness is what you own when you build in-house; the model is what you rent. |
| **Harness layers — E/T/C/L/O/V/G** | This repo's taxonomy, defined in [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) and applied to Inspect in §5.8: **E — Execution** (where actions physically run: sandboxes, containers, VMs); **T — Tooling** (what the agent can call, and the allow-lists and guards around it); **C — Context** (what the agent knows: retrieval, grounding, ranking, truncation, staleness); **L — Lifecycle** (how runs and artifacts begin, persist, resume, version, and end); **O — Observability** (traces, logs, cost, usage, adoption metrics, and — the part Ramp omits — assurance telemetry); **V — Verification** (in-loop: the agent proving its own work before handoff; out-of-loop: an independent check of aggregate behaviour); **G — Governance** (identity, attribution, approval, separation of duties, residency, retention). |
| **Merge-through rate** | Sessions that result in a merged pull request divided by sessions started. Ramp calls it the most important metric and publishes no value. The guide's proposed decision metric for a pilot (§7.5). |
| **Maker-checker** | The control principle that the person who creates a change cannot approve it. Ramp implements a mechanism-level approximation by opening PRs under the requester's token. |
| **Egress control** | Restricting what a sandbox can reach outbound. Not published by Ramp; mandatory for a regulated adopter (§6.2, §7.2). |
| **Region pinning** | Constraining sandboxes and snapshots to a specific geography to satisfy data-residency requirements. |
| **Golden set** | A frozen, versioned set of tasks with known-good outcomes, used to detect behaviour regressions when prompts, models, or tools change. Cross-ref [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md). |
| **Denominator problem** | The failure mode of "share of merged PRs": the numerator is measured, the denominator's definition is not, and total volume is undisclosed — so the percentage can rise without any change in output or value. |

---

## 13. References

**Primary sources — first-party, read in full**

1. Ramp Builders Blog. *"Why We Built Our Own Background Agent."* 12 January 2026. Zach Bruggeman, Jason Quense, Rahul Sengottuvelu. <https://builders.ramp.com/post/why-we-built-our-background-agent> — the build spec: sandbox, image registry, snapshot loop, OpenCode, API, multiplayer, clients, GitHub token design.
2. Modal. *"How Ramp built a full context background coding agent on Modal."* 19 February 2026. Greta Workman, Product Marketing. <https://modal.com/blog/how-ramp-built-a-full-context-background-coding-agent-on-modal> — **vendor-published.** Sandbox internals, in-sandbox service list, Functions/Dicts/Queues, concurrency, the >50% headline.
3. Modal Docs. *"Snapshots"* and *"Sandboxes."* <https://modal.com/docs/guide/sandbox-snapshots> · <https://modal.com/docs/guide/sandboxes> — filesystem, directory, and memory snapshots; diff-based storage; TTL defaults of 30 days and 7 days; forkability; deletion; customer-supplied encryption keys.

**Primary sources — independent journalism and vendor interviews**

5. The Pragmatic Engineer. *"Why Ramp built its own in-house coding agent, Inspect."* 25 August 2026. Gergely Orosz, Jessica Salmon, Ivan Klaric. <https://newsletter.pragmaticengineer.com/p/why-ramp-built-inspect> — **paid tier; partially readable.** Figures used here: 75% of merged PRs, ~90% of Inspect-repo PRs, 5.5-person team, 150+ contributors, one million sessions, under-5-second spin-up, public non-opt-out sessions, 200+ platform agents, the four use cases, the three reasons for building in-house. The architecture/tech-stack section is behind the paywall and was not readable — see §11 item 25.
6. Linear. *"The coding agent behind 75% of Ramp's merged PRs"* — Ramp customer story. <https://linear.app/customers/ramp> — **vendor-published.** The two-week v1 build, the Linear agent-API grounding, the "90 percent of the way there in about 30 minutes" integration quote, the review-bottleneck observation.

**Secondary sources — commentary and syndication**

7. The New Stack. *"Ramp's Inspect shows closed-loop AI agents are software's future."* <https://thenewstack.io/ramps-inspect-shows-closed-loop-ai-agents-are-softwares-future/> — trade commentary on the closed-loop framing. Used for framing only.
8. Ramp Labs (Substack). <https://ramplabs.substack.com/archive> — Ramp's AI-research publication. Consulted; **contains no dedicated Inspect post** in the archive as of this writing (posts cover Ramp Sheets self-maintenance, Claude Code in RollerCoaster Tycoon, Agent Fill, and finance-agent prompt generation). Inspect's own write-ups live on the Builders Blog and at <https://inspect.ramp.engineering/>.
9. Ramp engineering blog index. <https://ramp.com/blog> — Inspect's primary publication is the Builders Blog (item 1), which is where the engineering posts live.

**Cross-referenced sibling guides in this repository**

10. [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) — the E/T/C/L/O/V/G frame applied in §5.
11. [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) — the isolation spectrum and network-isolation patterns (§9 there) that §6.1 recommends grafting onto Ramp's architecture.
12. [coding_agents_research.md](coding_agents_research.md) — the coding-agent product landscape; §8 defers to it rather than re-deriving it.
13. [context_engineering_guide.md](context_engineering_guide.md) — retrieval and context budgeting, the undisclosed half of Inspect (§3.7, §11 item 5).
14. [agentops_guide.md](agentops_guide.md) — tracing, cost attribution, and the assurance telemetry §5.7 finds missing.
15. [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) — out-of-loop evaluation and golden sets (§7.2).
16. [ai_governance_framework_guide.md](ai_governance_framework_guide.md) — the governance frame for §6.3.
17. [agentic_engineering_guide.md](agentic_engineering_guide.md) · [agentic_solution_artifacts_guide.md](agentic_solution_artifacts_guide.md) · [agent_versioning_guide.md](agent_versioning_guide.md) · [agent_scaffolding_guide.md](agent_scaffolding_guide.md) — the engineering discipline, artifacts, versioning, and scaffolding.
18. [prompt_injection_guide.md](prompt_injection_guide.md) — why every internal integration in §5.2 is an untrusted-input channel.
19. [ai_agent_drift_guide.md](ai_agent_drift_guide.md) · [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) · [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) — resilience and failure modes for §6.3 item 3.
20. [multi_agent_banking_guide.md](multi_agent_banking_guide.md) · [hybrid_multi_agent_systems_guide.md](hybrid_multi_agent_systems_guide.md) — the platform composition pattern of §5.6.
21. [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) — how the internal integrations of §3.1 are exposed as tools.
22. [ai_adoption_strategies_guide.md](ai_adoption_strategies_guide.md) — the adoption mechanics of §5.5.
23. [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) — the platform architecture of §5.6 and §6.1 Take 6.
24. [ai_verify_toolkit_guide.md](ai_verify_toolkit_guide.md) · [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) · [llm_guard_models_guide.md](llm_guard_models_guide.md) · [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md) — out-of-loop verification, guardrails, and assurance loops for §5.7 and §6.3.

**Sourcing note for future readers.** Figures 4–10 in §4.1 rest on two articles written within a year of each other, one of which is a vendor case study and the other a paid newsletter interview. If you are reading this guide after 2026, re-check the Ramp Builders Blog first: it is the only first-party engineering artifact in the set, and it is the one most likely to have been updated or superseded.

---

**Last word.** Ramp Inspect is the clearest public demonstration that an internal background coding agent is buildable by a competent team in weeks and adoptable at scale in months — and it is simultaneously a demonstration of how little the published numbers can prove about whether that adoption was worth it. Copy the sandbox, the snapshot loop, and the user-token pull request. Do not copy the metrics, and do not mistake a well-documented shell for a documented brain. The real lesson is in the thing Ramp barely published: the grounding is the product, the friction is the adoption gate, and the review queue is where the truth about the inspected code lives — the inspected code.
