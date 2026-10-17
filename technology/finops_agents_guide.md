# FinOps Agents: The Optimiser as a Cost — A Comprehensive Guide

> **Author:** Jack Liu Shurui, Solution Architect  
> **Repository:** github.com/jackliusr/research · **Category:** Technology / AI Engineering Series · **Date:** October 2026

> **Companion guides (three owners, declared):** the FinOps discipline itself — principles, lifecycle, allocation, showback and chargeback, banking, metrics — is owned by [finops_guide.md](finops_guide.md). The cost *of* running agents — caps, budget layers, metering, routing, caching — is owned by [agentops_guide.md](ai_llm/agentops_guide.md) §6.2 and [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7. This guide owns the third thing, the one neither of them covers: **the agent as a FinOps actor** — an agent that finds waste, explains it, proposes a change, and sometimes executes it, and that must therefore be costed, attributed and governed like any other line in the budget it optimises.

**Contents**

1. [Overview, Decoder, Thesis and Boundary](#1-overview-decoder-thesis-and-boundary)
2. [The Two Directions and Where the Line Sits](#2-the-two-directions-and-where-the-line-sits)
3. [What an Agent Does That ML Cannot](#3-what-an-agent-does-that-ml-cannot)
4. [The FinOps Activities an Agent Can Perform](#4-the-finops-activities-an-agent-can-perform)
5. [The Loop That Closes](#5-the-loop-that-closes)
6. [The Cost of Agents, Cross-Referenced](#6-the-cost-of-agents-cross-referenced)
7. [Unit Economics of an Agent](#7-unit-economics-of-an-agent)
8. [Telemetry and Attribution](#8-telemetry-and-attribution)
9. [Financial Guardrails for an Acting Agent](#9-financial-guardrails-for-an-acting-agent)
10. [The Regulated and Banking Angle](#10-the-regulated-and-banking-angle)
11. [Failure Modes](#11-failure-modes)
12. [Cymbal Bank Worked Example](#12-cymbal-bank-worked-example)
13. [Anti-Patterns](#13-anti-patterns)
14. [State of Practice](#14-state-of-practice)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, Glossary and Closing](#16-what-could-not-be-verified-glossary-and-closing)

---

## 1. Overview, Decoder, Thesis and Boundary

**The subject is narrow and it is not the one the industry talks about.** Almost all writing on "AI and cloud cost" is about the bill a model generates: GPU hours, tokens, inference endpoints. That is direction A, the **cost of agents**, and it is well covered. This guide is about direction B: an agent that *does* FinOps — that compiles the showback, triages the anomaly, drafts the rightsizing change, corrects the tag, argues for the commitment purchase. Direction B changes the accounting, because the thing doing the optimising now has a bill of its own, produces claims that must be audited, and takes actions that must be attributable to a human owner.

**Thesis.** *An agent that optimises cost is itself a cost to be optimised.* Four consequences follow, and the whole guide is their elaboration:

1. **Its own token and compute spend must appear in the showback it compiles.** A report that excludes the reporter is not conservative, it is wrong — a lie of omission with a signature on it.
2. **Its optimisation must demonstrably save more than it costs**, including the human review time it creates and the cost of its errors. "Demonstrably" means measured against the bill, not asserted in a slide.
3. **Its automated action must be attributable to a business unit.** An actor with no cost centre is an actor nobody can charge, and therefore nobody can govern.
4. **A cost-reducing agent that cannot be costed has not closed the loop.** It has moved work, not removed cost, and it has made the ledger less trustworthy in the process.

**Decoder.** The vocabulary this guide uses, defined once so the rest can be terse.

| Term | Working definition | Why it matters here |
|---|---|---|
| **FinOps** | The operating discipline that makes technology spend legible and actionable across engineering, finance and product — Inform → Optimize → Operate | The practice the agent joins, not a tool the agent replaces |
| **Cost per task** | All metered spend for one completed unit of agent work: model calls, tool fees, retries, plus the human review time | The unit that lets an agent be compared to the saving it claims |
| **Token spend** | Input, output and cached-input tokens valued at the price in force at call time | The raw material of an agent's own bill |
| **Budget cap** | A hard ceiling enforced by a runtime counter before each step, not an alert after the fact | The difference between a limit and a wish |
| **Attribution** | The mapping from a cost record to an owner: agent, task class, cost centre, product line, legal entity | Without it, nothing downstream — no showback, no chargeback, no accountability |
| **Showback** | Reporting cost to the teams that caused it, without moving budget | The minimum viable accountability, and the agent's main output |
| **Chargeback** | Billing actual consumption against a team's budget | Requires attribution mature enough to survive a dispute |
| **Unit economics** | Cost per business unit of output — per transaction, per claim, per report | The only framing in which a cost-saving agent can be judged |
| **Rightsizing** | Matching provisioned capacity to measured need | The canonical cost action, and the canonical way to cause an outage |
| **Cost anomaly** | A spend series departing from its own baseline in a way that has a cause worth finding | The agent's richest input, and its easiest place to hallucinate |
| **Commitment (RI / SP / CUD)** | A term purchase — reserved instance, savings plan, committed use discount — traded for a rate | A financial instrument; buying one is a treasury decision, not a housekeeping one |

**Boundary, declared by name.** This guide does not re-derive the discipline. Principles, the Inform–Optimize–Operate lifecycle, Crawl–Walk–Run maturity, personas, pricing and commitments, tagging and the showback/chargeback ladder, optimisation levers, Kubernetes and multi-cloud, the banking overlay, metrics and certification all live in [finops_guide.md](finops_guide.md); where this guide needs them it names the section and moves on. **The cost-of-agents direction is owned elsewhere**: per-task dollar caps as a financial circuit breaker, budget layering (per-task hard, per-agent daily hard, per-team monthly soft, org ceiling), gateway metering and cost-per-task derivation, and the caching/routing levers are [agentops_guide.md](ai_llm/agentops_guide.md) §6.2 and §6.3, with the operational detail in [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.1–7.5. **This guide owns the agent-as-actor direction**: which FinOps activities an agent may occupy, where the line sits per activity, how its own spend enters the report it produces, how its claimed savings are verified, and who remains accountable when it acts.

**What this guide deliberately does not contain.** No cloud-waste percentage, no savings percentage, no idle-resource percentage, no market size, no vendor savings claim, no tool ranking. The reason is in §15: for the specific question "what does an agentic FinOps actor return, net of its own cost," no dated primary source publishes a figure that is not either a vendor's marketing or a single practitioner's anecdote. Arithmetic appears only in the labelled illustrative threads (§7.2, §8.4, §12), which are fictional and computed; and figures from the practitioner literature appear only where the Foundation published them (§14), each attributed with its date, what it measured, and the caveat that it is self-reported.

---

## 2. The Two Directions and Where the Line Sits

**The two directions are easy to confuse because they share a vocabulary.** Both talk about "agent cost," "cost per task," "budgets." They answer different questions and they fail in different places.

| | **(A) The cost of agents** | **(B) The agent as a FinOps actor** |
|---|---|---|
| **Question asked** | What does it cost to run this agent, and how do I stop it running away? | What may this agent do to the cost base, on whose authority, and did it pay? |
| **Owner** | [agentops_guide.md](ai_llm/agentops_guide.md) §6.2–6.3; [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7 | This guide |
| **Unit of analysis** | The task, the agent, the tenant | The decision, the action, the saving claim |
| **What it ends in** | A cap that fires, a metered cost-per-task line, a chargeback to the tenant | An attributable change to the cost base, with a named human accountable and a verified delta |
| **What it cannot do** | Say whether the agent was worth running | Cost the agent that produced the saving |
| **Failure mode** | Cost bomb; runaway loop; unattributed spend | Confident wrong explanation; unauthorised action; savings that never materialise |
| **Maturity signal** | Caps are enforced, not documented | The agent's own line item sits inside the report it generated |

**Why direction B is new, stated precisely.** The FinOps guide's AI material (§17) is *machine learning*, and it ends where ML ends: detection, forecasting, recommendation. ML anomaly detection learns a baseline; predictive forecasting adds seasonality; a recommender ranks placement options across cost, performance and carbon. All three hand a human a dashboard and stop. That is genuinely useful and it is not the same product as an agent that then opens the ticket, finds the owner from login and tag evidence, drafts the change, and either executes it or escalates with an argument. The same section's "FinOps for AI/ML" paragraph is about the *cost of AI workloads* — GPU hours, spot for training, inference cost per endpoint, time-slicing — which is direction A wearing a different hat. Neither paragraph describes an actor.

**The line, in one sentence.** Direction A asks *how much may the agent spend*; direction B asks *what may the agent do with what it finds, and who answers for it*. An agent can be perfectly capped (A solved) and still be an ungoverned actor (B unsolved), and the reverse: an agent can have immaculate approval workflows and still be a cost bomb. The two controls do not substitute for each other, which is why they live in different documents.

**An honest complication.** The boundary is not clean at the margin. An agent's own spend is trivially a cost in the base it optimises — which means direction B's subject matter *contains* direction A's output as one line item. §5 and §6 handle this by cross-reference rather than re-derivation, but the reader should notice that the FinOps report and the agent-cost report are the same report at different granularities. That is the whole reason to insist the agent's spend appears in its own showback: if the two reports are separate artefacts maintained by different teams, one of them will be wrong and it will be the one that flatters the programme.

### 2.1 Why the Confusion Persists

**Three forces keep the two directions fused in practice, and each produces a predictable mistake.**

**Vendor framing.** "AI-powered FinOps" is sold as one thing, usually a chat pane over a cost dashboard with a recommendation engine behind it. The buyer cannot tell whether they have purchased a better report (direction A's subject matter applied to a new workload) or an actor (direction B). The question that separates them is not on any datasheet: *what does it do when nobody is looking at the screen?*

**Org chart.** The team that owns the agent runtime and the team that owns the FinOps practice are usually different, and the agent sits between them. The platform team measures tokens and latency; the FinOps team measures savings. Neither owns the net, so the net is never computed — and a programme with no net number is governed by whoever tells the better story in the quarterly review.

**Genuine overlap in the artefact.** The same agent often does both jobs: a cost agent's own cost is direction A's subject applied to a direction B actor, and the report it emits is the join point. That overlap is not confusion, it is the thesis — but it means an organisation cannot solve direction B by hiring a FinOps person who has never seen a metering schema, or direction A by hiring a platform engineer who has never closed a month.

| If the problem you actually have is… | Build this first | Because |
|---|---|---|
| Runaway spend on an agent, no visibility per task | Direction A: caps, metering, cost per task | An actor you cannot bound cannot be trusted with authority, and the metering is the prerequisite for everything in this guide |
| Findings that never become changes | Direction B: the last mile, ownership resolution, verification | More analysis capacity will not fix a loop that has no closing step |
| A programme with no net number | Both, joined at §8's decision ID | Without the join, the two halves will each report success while the ledger stays flat |

---

## 3. What an Agent Does That ML Cannot

**Five separable steps, not one capability.** The useful decomposition of "agentic FinOps" is not autonomy levels but *which of five steps the agent occupies* for a given activity. The steps are separable in design even when one system performs all five, and keeping them separate is what makes the rulings in §4 possible.

| Step | What it produces | If the agent does it badly | Typical ML-era equivalent |
|---|---|---|---|
| **1. Finding** | A candidate: this resource, this account, this spend line, this deviation | Noise; a queue nobody works; alert fatigue that kills the programme | Threshold alerts, baseline deviation, recommender output |
| **2. Explanation** | The causal story: what changed, why, what it costs, what would fix it | A fluent wrong answer that authorises the wrong action | Almost none — a human wrote the narrative |
| **3. Proposed action** | A specific change with an estimated delta and a risk note | Plausible change with an unmodelled side effect | A recommendation with a size and a price |
| **4. Execution** | The change, made through an API, a pull request, or a ticket | An outage, a destroyed resource, an unauthorised commitment | Hard-coded auto-remediation rules |
| **5. Approval** | The authority to proceed, recorded | Diffused accountability; nobody said yes | Change board, ticket approval |

**The explanation step carries most of the value and most of the risk.** Finding is cheap now — every cloud has a recommender, and a threshold query is a line of SQL. Execution is a permission problem. The scarce step is explanation: the causal chain from "spend on this account rose 38% on the 14th" to "a batch job's retry loop is re-reading a cold partition after a schema change, owned by the settlements team, fixable by a partition predicate, worth roughly this much a month." That chain is what converts a signal into a decision, and it is why an agent with tool access beats a dashboard: it can query the billing export, the deployment history, the tag store and the ticket tracker in one pass.

It is also where the risk concentrates, because **an explanation is the artefact that authorises action**. A wrong number is caught by a spreadsheet. A wrong *story* is not: it is internally coherent, it cites real resource IDs, and it ends in a recommendation whose plausibility comes from the narrative rather than the evidence. The Foundation's own practitioners name this directly — model inaccuracies, "slop," make verification a required step in any agentic workflow (Morley, 27 March 2026). The design consequence: the explanation must be *checkable*, meaning every causal claim carries the query or record that supports it, and the reviewer's job is to check citations, not prose.

**The last mile is the actual problem FinOps has.** A recommendation that never reaches the owner is a cost with no benefit — the analysis was paid for and the saving was not. This is the oldest failure in the discipline: dashboards full of findings, a backlog nobody owns, an "action rate" measured in single digits. Agents change the economics of the last mile because steps 1–3 are now nearly free per candidate and step 4 can be a pull request rather than a meeting. That is the real claim, and it is a claim about *volume of completed loops*, not about intelligence.

**What actually changes technically, versus what is marketing.**

- **Real:** tool use against read APIs (billing export, CMDB, tag store, ticketing, version control); persistent state across a multi-step investigation; iteration — retrying a query that returned empty, following a reference the first pass missed; and **write access**, which is the only genuinely new risk surface and the only step that can destroy value in one call.
- **Marketing:** "autonomous" usually means a scheduled job with a model in the loop; "self-optimising" usually means threshold rules with an LLM writing the summary; "multi-agent" usually means prompt-specialised tools behind a router. The Foundation's description of an orchestrator delegating to "consultant agents" — a governance engine, an anomaly detector, a tagging expert — is exactly this pattern, and it is a reasonable architecture described in inflated language (Morley, 27 March 2026).
- **The tell:** ask which of the five steps the system occupies and what happens when step 2 is wrong. A system that cannot answer the second question has not been designed, it has been demoed.

**One thing ML does better, and the guide should say so.** For the finding step, a learned baseline over a stable series is more reliable and far cheaper than an agent reasoning about the same series in natural language. The agent's advantage is not detection accuracy; it is that detection is the only step where a cheap statistical model is sufficient, and the four steps after it are where the labour actually sits.

### 3.1 What a Checkable Explanation Looks Like

**The design problem is not "make the model more accurate"; it is "make the output auditable by a human in under a minute."** Four properties do most of the work:

1. **One claim per line, each with a pointer.** "Spend on account `payments-prod` rose 38% on the 14th" carries the query that produced it. "A retry loop is re-reading a cold partition" carries the deploy record and the log sample. A reader can check a citation; a reader cannot check a paragraph.
2. **A stated falsifier.** Each leading hypothesis carries the observation that would rule it out — "if the retry loop were the cause, the call count on the partition would show a step change; it does not." This is the single most useful field in the output, because it converts the reviewer's task from "do I believe this" to "is this test right," and the second question is answerable.
3. **Ranked alternatives, not a verdict.** The agent should present two or three candidate causes with their evidence and their confidence, because the failure mode is not a missing answer — it is a plausible answer that crowds out the right one. A single-cause output invites acceptance.
4. **A separation between observation and inference, marked in the artefact.** The observation is checkable; the inference is arguable. Mixing them in fluent prose is exactly how "slop" gets past a reviewer who is skimming, which is how reviewers read at volume.

**The reviewer's checklist follows from those four**, and it should be short enough to actually be used: does every claim have a pointer; is the falsifier stated; is there more than one candidate where the evidence is ambiguous; and does the recommended action's size match the confidence. A practice that adopts those four questions has done more for agent reliability than any model upgrade, because it changes what the system must produce rather than how well it produces it.

---
## 4. The FinOps Activities an Agent Can Perform

**One test generates every ruling in this section.** Score each proposed action on two axes: **reversibility** (can the previous state be restored in minutes, with no data loss?) and **financial commitment** (does the action create, destroy or lock in money beyond this billing period?). The quadrant dictates the authority:

| | **No commitment** | **Creates or locks in commitment** |
|---|---|---|
| **Reversible** | **Execute within policy.** Tag correction, stop-and-start of non-production on a schedule, ticket creation, report generation. The agent may act; the policy is pre-approved; the action is logged and sampled. | **Prepare only.** Commitment purchase (RI/SP/CUD), term contracts, reserved capacity. The agent may model scenarios and draft the case; a human transacts. |
| **Irreversible** | **Propose, never execute.** Terminate a resource, delete a volume or snapshot, drop a database, remove a retention policy. Cheap-looking and unrecoverable. | **Propose, never execute, and escalate beyond the service owner.** Production capacity reduction with a term commitment attached, account closure, licence surrender. |

**Why this test and not "confidence".** Confidence is the wrong gate because it is self-reported by the system being gated, and it degrades exactly when the situation is novel — which is when the action is most dangerous. Reversibility and commitment are properties of the *action*, verifiable by a human in advance, independent of how sure the model claims to be. A system that is 99% confident should still not terminate a production database; a system that is 70% confident may correct a tag, because a wrong tag costs a re-run, not a recovery.

### 4.1 Master Ruling Table

| Activity | Agent MAY | Agent MUST NOT | Accountable human | Why the line sits there |
|---|---|---|---|---|
| **Cost-anomaly triage** | Detect, investigate across billing/deploy/ticket systems, explain with citations, classify (expected change vs waste vs billing artefact), open and assign a ticket, draft the outreach message | Close an anomaly as "expected" without a citation; suppress or silence an alert class; adjust thresholds to reduce its own queue; declare a root cause it cannot evidence | FinOps analyst owns the triage queue; the resource owner owns the fix | Triage is reversible and commits nothing. But "expected" is a judgement that removes a control, so it must carry evidence a reviewer can check |
| **Rightsizing recommendation** | Compute utilisation over a stated window, select a target size, estimate the delta, draft the change record and rollback plan, flag SLA risk, open the change | Execute on production; resize anything in a regulated or stateful tier without the change process; use a utilisation window it chose itself without stating it | Service owner for the SLA; platform owner for the change | A resize is reversible in principle and not in practice — the outage happens inside the reversal window. Capacity is an availability decision |
| **Tag hygiene** | Detect missing/malformed tags, infer the likely owner from evidence, propose values with confidence, apply corrections within pre-approved rules (allow-listed keys, non-financial values), open fixes for the rest | Change a tag that is an input to a chargeback, a regulatory allocation or a legal-entity split without the allocation owner's approval; invent a cost-centre value not in the master list; mass-rewrite history | Allocation owner (finance) for anything that moves money between units; platform owner for the taxonomy | Tag correction is the one activity where execution is genuinely cheap and reversible. It is also the one where a "housekeeping" change silently restates a prior period's P&L |
| **Showback assembly** | Ingest budgets/forecasts, join to actuals, compute variances, draft the commentary, produce the pack, distribute it | Adjust a number to make a narrative work; exclude its own cost; publish a variance explanation it has not tied to a source record; present a forecast as an actual | FinOps lead signs the pack; each product owner owns their line | Assembly is reversible and commits nothing, and it is the highest-volume manual labour in most practices. The risk is not the arithmetic, it is editorial |
| **Commitment analysis** | Model coverage scenarios, compute break-even utilisation, compare term lengths and flexibility, quantify the risk of over-commitment, draft the business case | Purchase, exchange, or cancel any commitment; recommend a coverage level without stating the demand assumption and its variance; net a commitment saving against a rightsizing saving | Treasury/procurement for the purchase; CFO delegate for the term and the amount | A commitment is a financial instrument with a term. Getting it wrong locks in spend for a year; no agent may hold that pen |
| **Idle-resource identification** | Find candidates (no traffic, no attachments, no logins, stale snapshots), gather the ownership evidence, quantify the run-rate, propose stop/delete with a recovery plan, schedule stops within policy for non-production | Terminate or delete anything; stop a production resource; act on a resource whose owner is unresolved; treat "no traffic" as "no function" for a batch or disaster-recovery asset | Resource owner for deletion; platform owner for scheduled stops in non-production | Deletion is the least reversible action in the estate and the one with the worst asymmetry: saving is small and recurring, loss is total and immediate |

### 4.2 Cost-Anomaly Triage

**What good looks like.** The agent's output is not "spend is up." It is a decision record: the series, the change point, the correlated events (deploy, config change, traffic shift, price change, currency), the candidate causes ranked with the evidence for each, the owner resolved from tag/login/repo evidence, and the smallest action that would confirm or refute the leading cause. The reviewer's job becomes adjudication between ranked hypotheses, which is a task a human does well and quickly.

**The specific prohibition that matters.** The agent must not be able to mark an anomaly "expected" without a citation, because that is the action that *removes* a control. In practice this means the classification field is a closed enum with a mandatory evidence pointer, and "expected" requires either a matching change record or a matching forecast band. Without that constraint, an agent optimising for a clean queue will explain anomalies away — fluently, plausibly, and at scale. That is failure mode (b) in §11 and it is the single most likely way this technology makes a FinOps practice *worse*.

### 4.3 Rightsizing Recommendation

**The agent may hold the analysis and the draft; the service owner holds the change.** The utilisation window, the percentile, the headroom assumption and the excluded periods must be *stated in the output*, not buried in the tool call, because those four choices determine the recommendation and they are exactly what a challenged recommendation will be challenged on. A recommendation that says "reduce to 4 vCPU" without saying "over 21 days, p95 CPU 18%, p95 memory 41%, excluding the month-end batch window" is not a recommendation, it is an assertion.

**Where the SLA risk lives.** The agent can flag it; it cannot price it. The service owner knows that the month-end batch peaks at 6× the p95 and that the failover pair doubles the load during a zone event — knowledge that is not in the metrics the agent reads. This is why production rightsizing stays a human decision even at high agent maturity, and why the decision is a *capacity* decision wearing a cost costume. §10 turns this into the operational-risk argument.

### 4.4 Tag Hygiene

**The one activity where execution should be delegated — narrowly.** Tag correction is reversible, commits no money, and its volume is the reason it never gets done. The delegation must be bounded by four rules: allow-listed keys only; values validated against the cost-centre master list (no free-text invention); no change to any tag declared an allocation input; and a full before/after record joined to the resource's history so the change can be reconstructed.

**The trap is that tags are not housekeeping.** A tag feeds the allocation model, which feeds showback, which feeds a product's P&L, and in a regulated entity it may feed a cost return. An agent that "improves" tag coverage by inferring owners can therefore restate a prior period's allocation without anyone deciding to. The ruling: inference may *propose*, and may *apply* only where the value is a non-financial attribute; where the value moves money, the allocation owner approves the batch as a single decision with a stated prior-period impact.

### 4.5 Showback Assembly

**The highest-value, lowest-glory activity, and the one where the thesis bites hardest.** Assembly is where the agent saves the most analyst hours and where its own omission would be most damaging, because the report is the artefact everyone else trusts. The rule from §5 applies with no exceptions: the agent's own metered spend — model calls, tool fees, platform allocation, and the human review hours it consumes — is a line in the pack, attributed to whoever owns the agent, and visible to the readers of the pack.

**Editorial risk is the real risk.** A model writing variance commentary will produce a fluent explanation for a variance it has not traced. The constraint: every sentence in the commentary carries a record pointer, and any variance above the materiality threshold must resolve to either a tagged change record or an open anomaly. Commentary that cannot be traced is deleted, not softened.

### 4.6 Commitment Analysis

**Analysis yes, transaction never.** The agent's comparative advantage is scenario volume: coverage levels against demand distributions, term lengths against expected workload life, flexibility premiums against the probability of change. This is genuine analytical work that practices do badly because it is tedious.

**Two prohibitions, both about honesty.** First, the recommendation must carry its demand assumption and the variance around it — a coverage recommendation without a stated demand curve is a sales pitch. Second, the saving must not be netted against other levers in the same claim: a commitment that covers capacity the rightsizing programme is about to remove produces a paper saving and a real over-commitment. That is failure mode (g) in §11, and it is common precisely because the two analyses usually sit with different teams.

### 4.7 Idle-Resource Identification

**The finding is cheap; the deletion is the entire risk.** "No traffic in 30 days" is evidence of idleness for a stateless service and evidence of nothing for a batch job, a disaster-recovery standby, a compliance archive, or a resource behind a seasonal product. The agent must therefore output an *idleness case* with the evidence class stated, not an idleness verdict.

**The narrow execution the ruling allows:** scheduled stop (not terminate) of non-production resources that carry an explicit non-production tag and an owner who has been notified, within a policy window, with a documented restart path. Everything else — production stops, any termination, any deletion of a volume, snapshot or backup — is a proposal to the owner. The asymmetry justifies the conservatism: a stopped non-production instance costs a restart; a deleted snapshot costs the recovery point, and sometimes the business.

### 4.8 The Activity That Is Not on This List

**Forecasting and budget-setting.** It is absent from the ruling table deliberately, because it is the one activity where the agent has a conflict of interest that no amount of accuracy resolves: the agent's own cost is a line in the budget it would be helping to set. The rulings that follow are therefore narrower than the activity's importance suggests.

- **Forecast construction — may.** Ingest history, model seasonality, produce a range rather than a point, state the assumptions, and flag the difference between its forecast and the submitted plan.
- **Forecast submission — may not.** A submitted forecast is a commitment that other people are measured against. It is signed by the finance owner, and the agent's output is an input to that signature, clearly labelled as a model output with its assumption set.
- **Budget-setting for its own cost centre — may not, under any configuration.** The agent may report its run-rate and its cost per task; it may not propose its own budget, and it may not be the system that computes whether the proposed budget is adequate. This is the §5.4 separation applied one step earlier in the cycle, and it is the same failure as the agent that owns its own budget (§13) — just quieter, because a budget proposal looks like analysis rather than authority.
- **Variance-to-forecast commentary — may**, under §4.5's editorial constraint: every sentence traceable, and any material variance resolved to a change record or an open anomaly.

**Why the line sits there.** Budget-setting is where authority is constituted. An actor that influences the number it will later be judged against has, in effect, set its own performance bar, and the audit trail will show a clean process that produced a compromised result.

---

## 5. The Loop That Closes

**FinOps is a loop, and an agent inside FinOps is a second loop nested in the first.** The discipline's own cycle is Inform → Optimize → Operate. The agent adds a cycle of its own, and the two must be joined at a single point: the report.

### 5.1 The Cycle

1. **Compile.** The agent assembles the cost picture: actuals, budgets, variances, findings, proposed actions, and — this is the step everyone omits — **its own metered spend for the period**, as a line, attributed to its owner.
2. **Act.** Within the §4 rulings, the agent executes what policy permits (tag corrections, non-production stops, tickets, packs) and escalates what it does not.
3. **Measure own spend.** The agent's spend is metered per call and per task by the gateway (a capability owned by [agentops_guide.md](ai_llm/agentops_guide.md) §6.2), then aggregated to the period and to the task classes it actually ran.
4. **Net.** Verified saving minus agent run cost minus human review cost minus error cost. Verified, not claimed (§5.3).
5. **Decide.** Keep, narrow, re-scope, or retire the agent — and set next period's authority. This decision is taken by someone other than the agent (§5.4).
6. **Record.** The decision, the authority granted, and the net number go into the next pack, so the next cycle starts from an audited position rather than a fresh assertion.

### 5.2 The Report Must Contain the Reporter

**A showback that excludes the agent is a lie of omission, and the omission is systematic.** Three mechanisms produce it. The agent's spend is billed on a model-provider invoice or an internal AI platform chargeback, while the report is built from cloud billing exports — different systems, different schemas, different owners, so the join never happens. The agent's spend is small relative to the estate, so omitting it feels immaterial — until the agent is doing the work of six analysts and the saving it claims is partly the analysts' unspent salaries. And the team that owns the agent is usually the team that owns the report, which is the oldest conflict of interest in management reporting.

**The rule, stated as a constraint on the artefact:** if a reader of the pack cannot see a line for the agent, the pack is not publishable. Not "should include" — the report fails its own control. The practical form is a mandatory line item in the report template, with a non-zero value or an explicit zero with a reason, exactly as a zero-spend cost centre is treated in a financial close.

**And the second-order version, which is where programmes actually fail.** The saving the agent claims is often *labour*: analyst hours not spent assembling the pack, investigating tickets, writing variance commentary. That labour is a real cost only if it is released or redeployed — if the analyst now spends the same hours reviewing the agent's output, the saving is a transfer, not a reduction. §7's arithmetic and §12's worked example both carry human review as a cost line for this reason. The agent that removes three hours of assembly and creates two hours of review has saved one hour, and reporting three is savings theatre.

### 5.3 The Arithmetic of "Saves More Than It Costs"

**State it as a threshold, not an adjective:**

```
net saving = verified saving − agent run cost − human review cost − error cost
require: net saving > 0, measured over a period that matches the saving's realisation lag
```

Each term has a definition that prevents the usual cheating:

- **Verified saving** — the billed delta, not the size of the recommendation. Verification needs three things: the cost line is identified before the change; the post-change billing shows the reduction; no offsetting new spend appeared in the same period. A resize that saves on compute and triggers autoscaling to add instances has saved nothing.
- **Agent run cost** — metered model, tool and platform spend for the tasks in the period, including the ones that produced nothing. Failed tasks are part of the cost of the successful ones.
- **Human review cost** — reviewer hours × loaded rate. Not optional, and not "already sunk," unless the reviewer's time was genuinely freed.
- **Error cost** — the expected cost of the agent's mistakes: rework, misallocation corrections, incident cost from a bad action, and the review effort that follows a bad explanation. This is the term that gets omitted because it is hard to attribute, and it is the term that decides whether the programme is real.

**The realisation lag is where the arithmetic goes wrong.** A rightsizing saving shows up next billing cycle; a commitment saving is spread across the term; an idle-resource deletion shows up immediately; a tag-correction "saving" may not be a saving at all, only a reallocation. Netting a twelve-month commitment benefit against a one-month agent cost produces a spectacular and meaningless number. The window must be the saving's own window, or the comparison must be stated as a run-rate with the term risk disclosed.

**The threshold is not zero in practice, and saying so is part of the honesty.** A net saving of a few percent of the agent's cost is noise; the review cost alone will swamp it in most months. A programme needs a stated margin — the amount by which the net must clear zero before the agent is judged worth running — and that margin is a risk appetite decision, not a technical one.

### 5.4 The Recursion, Stated Honestly

**The agent that decides whether the agent is worth running should not be the same agent.** The reason is not mistrust of models in general; it is that step 5 is a decision about the agent's own budget, authority and continued existence, and a system asked to evaluate itself has an objective that is not the institution's. Three acceptable arrangements: a human (the FinOps lead or the budget owner) makes the call from the agent's pack; a *separate* evaluator — different agent, different owner, different data access, ideally a different model family — produces the assessment that a human signs; or an independent function (internal audit, finance business partner) owns the net calculation with read-only access to the metered records.

**What is not acceptable** is the agent producing its own verified-savings figure with no independent path to the billing data, and the same agent deciding the figure clears the threshold. That is a control that certifies itself, and in a regulated entity (§10) it is the first thing an auditor will find.

**The recursion terminates in a human, by design.** Not because the human is smarter, but because the decision requires someone who bears the consequence. That is the same argument that governs §4's rulings and it is the load-bearing claim of this guide.

---

## 6. The Cost of Agents, Cross-Referenced

**This section is deliberately short, because the material is owned elsewhere and re-deriving it would produce a worse copy.** The full cost-control stack for agents — per-task token and dollar caps enforced by runtime counters checked before each step, with stop-save-state-log-escalate on exhaustion; budget layering from per-task hard cap through per-agent daily cap and per-team monthly soft budget to an org ceiling; gateway metering of tokens in/out, model, price, agent ID, tenant/team ID and trace ID, with cost per task *derived* from metered records rather than estimated; and the levers of prompt caching, response caching, model routing and escalate-on-failure — is [agentops_guide.md](ai_llm/agentops_guide.md) §6.2 and §6.3, with the implementation detail and the cost-controls table in [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.1–7.5.

| Question | Where it is answered | What it gives this guide |
|---|---|---|
| How much may one task spend before it stops? | [agentops_guide.md](ai_llm/agentops_guide.md) §6.2 (per-task caps); [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.1 | The denominator of the net calculation: the agent's run cost is bounded and metered |
| How is spend attributed to an agent and a tenant? | [agentops_guide.md](ai_llm/agentops_guide.md) §6.2 (gateway metering) | The per-call records that §8 extends with a cost-centre and a decision ID |
| How is cost per task measured and monitored? | [agentops_guide.md](ai_llm/agentops_guide.md) §6.3 | The unit metric that §7 turns into a unit-economics argument |
| Which levers reduce agent spend? | [agentops_guide.md](ai_llm/agentops_guide.md) §6.2; [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.2–7.3 | The levers that make the agent's own cost small enough for the net to clear |

**The one thing that material does not answer.** Agent-cost attribution as designed for chargeback stops at the tenant/team: the LOB pays for its agents, the platform team publishes unit costs, the metered records carry an agent ID and a tenant ID. That is sufficient to send an invoice. It is not sufficient to put the agent into the same showback it produces, for three reasons.

1. **The mapping is missing.** The FinOps report is organised by cost centre, product line and legal entity — the dimensions in the allocation model (see [finops_guide.md](finops_guide.md) §7). The agent's metered records carry a tenant, which is an AI-platform construct, not a financial one. Someone must map tenant → cost centre → product line, and that mapping must be maintained as teams reorganise, or the line item will land in a shared pool and be allocated by usage proportion — which is how costs become invisible.
2. **The ledger is missing.** The agent's spend is on a model-provider or platform invoice, valued at list price, in a different currency of record, with a different billing period close. The resources it optimises are in the cloud billing export, normalised (increasingly) to an open specification — see the FOCUS column library at focus.finops.org, which exists precisely so that heterogeneous billing datasets land in one schema. Until the agent's own spend is emitted as cost and usage records in that same schema, granularity and period, the "one report" is a spreadsheet someone maintains by hand, and it will drift.
3. **The actor has no manager.** A human cost centre has an owner because someone hired them. An agent has an owner only because someone was assigned. Default that assignment to the platform team and the cost sits in the pool that everyone pays for and nobody manages — the classic shared-cost failure the discipline already knows how to avoid, applied to a new kind of actor.

**The design output of this section, in one line:** the agent's spend must be emitted as first-class cost and usage records carrying (cost centre, product line, legal entity, decision ID) in the same store as the cloud bill, so the showback query that produces the pack also produces the agent's own line. §8 specifies the fields.

### 6.1 The Two-Ledger Problem, Concretely

**Four specific mismatches make the join fail, and each has a different owner to convince.**

1. **Different close.** The cloud bill closes on one date, the model-provider invoice on another, the internal platform charge on a third. Netting across them produces a number that moves when nothing changed — and a net that moves for accounting reasons will be dismissed as unreliable within two quarters, after which nobody computes it.
2. **Different price basis.** The cloud bill reflects negotiated rates and commitments; the agent's spend sits at list price on a vendor invoice or at an internal transfer price that includes a margin. Comparing a saving at negotiated rates to a cost at list rates overstates the programme, and comparing a committed-rate saving to an uncommitted cost understates it. One basis must be chosen and stated — the finance ledger's — and both sides converted to it.
3. **Different granularity.** The bill arrives per resource per hour; the agent's spend arrives per call. Cost per task is the only grain both can produce, which is the practical argument for it beyond the business-facing one.
4. **Different owner.** The bill is owned by the FinOps practice; the metered records are owned by the platform team; the invoice is owned by procurement. Three owners, no single number — and the fix is organisational before it is technical: one named owner for the joined report, with read access to all three sources and the authority to publish.

**None of these is hard to fix technically, and all of them are hard to fix politically**, because each fix requires someone to accept a number that may be less flattering than the one they currently report. That is the real reason the agent's spend rarely appears in the showback: not schema difficulty, but the fact that the line has no friends.

---
## 7. Unit Economics of an Agent

**Cost per task is the denominator; value per task is the numerator; for a cost agent the numerator is unusually honest.** Most agents have a value story that resists measurement — better answers, faster resolution, happier users — which is why most agent business cases end in a survey. A cost agent's value is denominated in the same currency as its cost, and that is a strategic advantage the discipline has not yet exploited: it can be put on its own P&L, with a real break-even, and defended or killed on arithmetic.

### 7.1 The Two Per-Task Numbers

| | **Cost per task** | **Value per task** |
|---|---|---|
| Built from | Metered model calls + tool fees + retries + platform share + reviewer time | Verified saving attributable to the tasks completed, over the saving's own window |
| Source of truth | Gateway records (derived, not estimated) | Billing delta (measured, not claimed) |
| Volatility | Stable, forecastable | Declines as easy findings are exhausted — the important dynamic |
| What it decides | Whether the agent is affordable | Whether the agent is worth affording |
| Failure mode | Under-counting review time | Counting deadweight and double-counting |

**The decline is the thing to model.** An idle-resource agent's value per task falls over time because the obvious waste is found first; by month nine it is prospecting in the tail, where each candidate costs more to investigate and pays less. A unit-economics model with a flat value per task will therefore over-predict forever after month three, and the quarter where the agent stops paying is a predictable event rather than a surprise. §12 shows what that month looks like and what the right response is.

### 7.2 The Arithmetic, With Illustrative Numbers

*The numbers below are illustrative and belong to the fictional case in §12; they are arithmetic, not a finding about any real estate.*

Take the anomaly-triage workload alone. Per month:

| Line | Derivation | Value |
|---|---|---|
| Tasks run | 1,800 triage tasks | 1,800 |
| Run cost per task | metered model + tool spend | $0.62 |
| **Total run cost** | 1,800 × $0.62 | **$1,116** |
| Platform share | this workload's allocation of gateway, storage, orchestration | **$160** |
| Escalations reviewed | 320 of 1,800 proposals went to a human | 320 |
| Review time per escalation | 9 minutes | 2,880 min = 48 h |
| Reviewer loaded rate | analyst at $78/h | $78/h |
| **Total review cost** | 48 h × $78 | **$3,744** |
| **Total cost of the workload** | 1,116 + 160 + 3,744 | **$5,020** |
| **Break-even value per task** | 5,020 ÷ 1,800 | **$2.79** |
| Verified saving in the month | billed delta from remediated findings | $9,300 |
| **Value per task** | 9,300 ÷ 1,800 | **$5.17** |
| **Net** | 9,300 − 5,020 | **$4,280** |

Three things to notice, because they generalise:

1. **Review cost is 75% of the workload's cost.** The model is $1,116; the humans checking it are $3,744. Any business case built on token spend alone is off by a factor of four, and this ratio is typical of a human-in-the-loop design rather than an anomaly.
2. **The break-even is $2.79 per task, not $0.62.** A team that budgets the token cost and calls that the cost of the agent will approve agents that cannot pay.
3. **The net is positive but thin in structure, not in size.** $4,280/month net against a $5,020 cost means the programme keeps 46% of the value it verifies and spends 54% of it finding and checking — healthy, but it rests entirely on 320 escalations producing $9,300 of verified remediation. If half the findings were deadweight, the agent would be at break-even.

### 7.3 The Break-Even Relation

For a single workload, with all terms per period:

```
verified_saving(tasks)  >  tasks × run_cost_per_task + platform + review_cost + error_cost
```

Rearranged, the two questions that matter operationally:

- **Minimum yield per task.** With fixed costs (platform, reviewer capacity) and a variable run cost, the required saving per task rises as volume falls — the agent must clear a higher bar on a quiet month, which is why shrinking the task volume without shrinking the review capacity is the most common way a working agent turns into a losing one.
- **Maximum review rate.** If the escalation rate rises, review cost rises linearly and the net falls; an agent that improves its recall by escalating more can destroy its own economics. This is the concrete reason to tune for *precision at the escalate threshold*, not for recall.

### 7.4 The Two Ways a Cost Agent Lies About Value

**Deadweight.** The saving would have happened anyway. The resource was already in a decommission queue, the team was already migrating, the contract was already being renegotiated. The test is a control: the same measure on the population the agent did *not* touch, or a pre-period trend that already showed the decline. If the cost line was falling before the agent found it, the agent did not cause the fall — it annotated it. This is not a hypothetical; it is the default outcome when agents are deployed into estates that already have an active optimisation programme, because the agent finds the same backlog the humans were working.

**Double counting.** The same dollar claimed twice, usually by different teams who each believe they own the lever. The three common pairs: a rightsizing saving and a commitment coverage saving on the same capacity (the commitment covers instances the resize was about to shrink); an idle-resource deletion and a tag-correction "recovery" of the same spend; an agent's triage saving and the platform team's own optimisation programme reporting the same remediation. The control is a **savings ledger keyed by cost line and period**, where a line can be claimed once, by one owner, with a pointer to the decision that caused the change. §8's decision ID is what makes that ledger mechanically possible instead of politically negotiated.

**A third, subtler one worth naming.** Counting a *rate* improvement as a saving. Unit cost falling from one level to another is not a saving if volume rose to consume it — the bill is the same and the business is bigger. That may be a good outcome, but it is a productivity story, not a cost reduction, and the two should not share a number.

---

## 8. Telemetry and Attribution

**An agent's spend is attributable only if someone designed the fields before the first call.** Retrofitting attribution onto an agent estate is the same problem as retrofitting tagging onto a cloud estate, with the same outcome: a permanent, low-grade, politically contested approximation. The fields below are the minimum for the loop in §5 to be computable rather than argued.

### 8.1 The Record

| Field | Why it must exist | What breaks without it |
|---|---|---|
| `call_id` | Unique per model or tool call | Duplicate metering; unreconcilable totals |
| `agent_id`, `agent_version` | Which system, which build | Cannot attribute a regression, or a saving, to a release |
| `task_id`, `task_class` | Groups calls into one unit of work | No cost per task; only cost per call, which nobody manages |
| `tenant_id` | The platform's own scoping | Chargeback to the wrong consumer |
| `cost_centre`, `product_line`, `legal_entity` | The financial dimensions of the allocation model | The agent's spend lands in a shared pool (§6) |
| `model`, `tokens_in`, `tokens_out`, `tokens_cached`, `unit_price`, `price_version` | The valuation basis | Cost drifts silently when prices change and the report keeps the old rate |
| `tool_calls`, `tool_fees` | Non-model spend | Understates cost for tool-heavy agents |
| `period` | Billing period, aligned to the cloud bill's close | The join fails at the period boundary and someone hand-maps it |
| `trace_id` | End-to-end trace across orchestrator and sub-agents | Cannot reconstruct why a task cost what it cost |
| **`decision_id`** | One per agent decision that produced an external action or a saving claim | **No link between spend and outcome — the loop cannot close** |
| `action_id`, `approval_id` | The action taken and the human who approved it | The audit trail in §10 does not exist |
| `saving_claim_id` | The claim in the savings ledger | Double counting is undetectable |

**`decision_id` is the load-bearing field and the one nobody has.** It is the junction between the agent's cost and the agent's effect: the same ID appears on the metered calls that produced the decision, on the action record, on the approval, and on the saving claim that the finance team later verifies. With it, "what did this agent cost, and what did it return" is a join. Without it, it is a workshop.

### 8.2 The Actor With No Manager

**An automated actor has no team, no manager and no cost centre by default** — and every default that gets invented to fill the gap degrades the report.

- **Default 1: put it in the platform pool.** The agent's spend joins shared infrastructure and is allocated by usage proportion. Consequence: the cost is real, visible to nobody, and grows without a challenger. This is the same failure the discipline already recognises for shared services, applied to a new actor.
- **Default 2: bill it to the team that built it.** The consequence is worse than it looks: the team that benefits from the agent's findings is not the team that pays for it, so the agent looks like a cost centre with no revenue and gets cut in the first efficiency round — even when it is the most profitable thing in the estate.
- **Default 3: no attribution at all.** The spend sits on an AI platform invoice that finance treats as a technology line. The agent's savings are claimed in the FinOps pack; its costs are in a different pack; the two are never netted, and the programme is judged on the half of the ledger that flatters it.

**The ruling:** the agent gets a **real cost centre with a named accountable human**, and its cost is allocated to the product lines it serves by a stated key (findings attributable, tasks run on their behalf, or a fixed split reviewed annually). The named human is the same person who signs the §5.4 keep-or-retire decision — because paying for something and judging whether it is worth paying for should be the same job.

### 8.3 Join-Key Discipline

Four rules, each of which fails loudly in practice if skipped:

1. **One period definition.** The agent's spend and the cloud bill must close on the same calendar. If the bill closes on the 3rd and the model invoice on the 1st, the net is wrong every month by a few days of spend — small, systematic, and enough to make the number unauditable.
2. **One currency of record, one rate source.** Convert at the rate the finance ledger uses, with `price_version` recorded, so a re-statement can be recomputed rather than renegotiated.
3. **Schema convergence wherever possible.** Emit the agent's cost and usage records in the same normalised schema as the cloud bill — the direction the open cost-and-usage specification exists to serve (focus.finops.org, FOCUS 1.4). The point is not the schema's elegance; it is that one query can then produce a report containing both, and a report produced by one query is a report that cannot quietly omit its author.
4. **The savings ledger joins on the same keys.** `decision_id`, `cost_centre`, `period`. A savings ledger with free-text resource descriptions and a "month" column is the artefact that makes double counting inevitable.

**The test of the design.** Ask for one number — "what did the cost agent cost Cymbal Bank's payments product line last month, and what verified saving did it produce for that line" — and see whether it is a query or a project. If it is a project, the telemetry is not designed; it is accumulated.

### 8.4 One Record, End to End

**The whole design is easier to hold as a single trace than as a field list.** A tag-correction decision, from spend to saving:

```
decision_id      dec-2026-10-14-000871
agent_id         ledger@2.3.1
task_class       tag.correction.batch
tenant_id        finops-automation
cost_centre      CC-4410 (FinOps Automation)   product_line  payments
legal_entity     SG-ENTITY-1                    period  2026-10
trace_id         tr-9f21...
-- metered calls that produced the decision
calls            7   tokens_in 41,208  tokens_out 3,114  cached 28,900
model_cost       $0.19        tool_fees  $0.00        task_cost  $0.19
-- the decision and the action
proposal         set cost_center=CC-3172 on 14 resources (owner evidence: repo + login)
confidence       high          estimate  $0 reallocated; allocation-input tag: NO
action_id        act-2026-10-14-00612    executed under policy TAG-POL-3
approval_id      n/a (within delegated authority; policy owner: allocation owner, CC-3172)
rollback         prior values retained, 90-day lineage
-- the claim, verified later by finance
saving_claim_id  sc-2026-10-0412   status  verified (reallocation, not a reduction)
verified_by      finance business partner, 2026-11-04
```

**Four things this record does that a log does not.** It makes the task's cost ($0.19) attributable to a cost centre that a human owns; it records that the action was one of the class where execution is delegated, and names the policy that delegated it; it records that the claim is a *reallocation* rather than a saving, which prevents it from being added to the savings total; and it retains the prior values as lineage, which is what a restatement question will ask for (§10.2). The record is boring, and boring is the point: an auditor can read it without a specialist.

---

## 9. Financial Guardrails for an Acting Agent

**The perimeter controls are owned elsewhere and are assumed here.** Per-task token and dollar caps with runtime counters checked before each step, the hard-stop-save-log-escalate sequence, budget layering from task to org, and the anomaly detection on the spend series are [agentops_guide.md](ai_llm/agentops_guide.md) §6.2–6.3 and [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.1. Those answer *how much may the agent spend*. The controls below answer a different question: **how much financial effect may the agent cause**, which no cap on its own token spend constrains. An agent can spend $3 of tokens and cancel a $40,000 commitment.

### 9.1 The Five Controls Specific to Acting on Cost

| Control | Mechanism | What it prevents |
|---|---|---|
| **Spend authority per action** | A ceiling per action class: tag correction ≤ $0 financial effect; non-prod stop ≤ a stated monthly run-rate; anything above → propose only | A single action with a large financial effect taken under a policy written for small ones |
| **Cumulative authority per period** | A per-period budget of *authority*, not money: no more than N actions or $X of monthly run-rate changed per week without review | A thousand small compliant actions summing to a large unauthorised change |
| **Mandatory pre-action cost estimation** | Before any write, the agent must emit: the estimated monthly delta, its basis, its confidence, and the rollback path | Actions taken on a finding rather than on a forecast; unlogged side effects |
| **Kill switch owned by a human** | A tested path that halts the fleet, not just one task, owned by a named role with the authority to use it unilaterally | An agent looping through permitted actions at scale; a bad release acting fast |
| **No self-approval, no self-raise** | The agent cannot approve its own proposal, raise its own caps, change its own policy, or edit the savings ledger | The control that certifies itself (§5.4), and the agent that owns its own budget (§13) |

### 9.2 Pre-Action Cost Estimation Is the Core Discipline

**The estimate is not there to be accurate; it is there to be reviewable.** An agent that writes "estimated monthly delta: −$1,240, basis: 21-day p95 utilisation on 6 instances at on-demand rate, confidence: medium, rollback: previous instance type retained for 7 days" has produced an artefact a service owner can disagree with in ten seconds. An agent that writes "rightsize to save money" has produced nothing, and the difference is not model capability — it is a schema requirement in the action tool.

**The schema should make the estimate mandatory at the tool boundary**, not in the prompt. If the write tool rejects a call without a delta estimate, basis and rollback path, the control holds even when the model is having a bad day. This is the same lesson as parameter validation in the tool layer ([agentops_guide.md](ai_llm/agentops_guide.md) §6.4): controls that live in prompts are suggestions; controls that live in the tool interface are controls.

### 9.3 Shadow Mode Before Authority

**Every action class earns its authority in shadow mode first.** The agent produces what it *would* have done, with the estimate and the rollback, and a human compares that to what the human did. Two metrics decide promotion: agreement rate on the action, and — more important — the rate at which the agent's estimate was materially wrong. A class that agrees 95% of the time but mis-estimates the delta by a factor of three is not ready, because the estimate is what the authority is granted against.

**Promotion should be per action class, not per agent.** An agent that has earned execution authority for tag corrections has earned nothing for production rightsizing; bundling the two into one go-live is how programmes acquire authority they did not earn. And demotion must be as easy as promotion: two material estimation errors in a class within a quarter should revoke execution for that class automatically, without a committee.

### 9.4 What the Kill Switch Actually Has To Do

**Halting a task is not halting the agent.** Three distinct stops, all needed: stop the individual task (the runtime's budget stop, already owned); **pause the agent** (no new tasks, in-flight tasks finish or abort per policy); and **revoke the authority** (the agent keeps running read-only, but its write credentials are withdrawn). The third is the one that matters in an incident, because it lets the agent keep producing evidence while it cannot act — and it is the one most systems lack, because write authority is usually granted at deployment and never revisited.

**Ownership and testing.** The switch belongs to a named human role with unilateral authority — in a bank, typically the platform owner or the FinOps lead with a delegated authority documented in the policy (§10). It must be tested on a schedule; an untested kill switch is a documented intention. And it must be usable by someone who is not the agent's builder, at 3 a.m., without a change request.

### 9.5 Authority by Maturity

**The discipline already has a maturity ladder — Crawl, Walk, Run ([finops_guide.md](finops_guide.md) §4) — and agent authority should climb the same ladder rather than a parallel one.** The mapping below is the practical answer to "how much should we let it do," and it assumes the §9.1 controls exist at every level.

| Action class | Crawl | Walk | Run |
|---|---|---|---|
| **Report assembly and commentary** | Draft only; human rewrites | Draft published with review; human signs | Published directly; sampled review |
| **Anomaly triage and ticket creation** | Agent suggests, analyst raises the ticket | Agent raises and assigns; analyst reviews the queue daily | Agent runs the queue; analyst reviews exceptions and audits a sample |
| **Owner resolution** | Human resolves ownership | Agent proposes with evidence; human confirms | Agent's high-confidence resolution stands; the rest route to a human queue |
| **Tag correction** | Off | Non-financial keys, allow-listed, human-approved batches | Within policy: allow-listed keys, master-list values, batch cap, allocation-input exclusion |
| **Non-production scheduled stop** | Off | Notified, documented, reversible on request | Within policy, with an owner-notification window and an override path |
| **Production rightsizing** | Off | Off | Off — executed by humans under the change process, with the agent's analysis as input |
| **Commitment purchase** | Off | Off | Off — treasury decision, agent analysis as input |

**Two properties of this table matter more than its contents.** First, the "Off" column on the last two rows is off at *every* maturity level, because maturity is a property of the operating model and the line there is drawn by the nature of the action, not by how good the agent has become. Second, maturity is granted per row: a practice that is at Run on report assembly and Crawl on tag correction is normal and healthy, and a proposal to go to Run across the board is a proposal to skip the evidence.

---

## 10. The Regulated and Banking Angle

**The banking FinOps material is owned by [finops_guide.md](finops_guide.md) §11** — BCBS 239 cost attribution to business lines and legal entities, the audit trail and retention expectations, chargeback between legal entities under transfer-pricing rules, the multi-account structure per regulatory boundary, mandatory tagging including `legal_entity` and `compliance_regime`, and the approval thresholds for resource and commitment spend. None of that is restated here. This section adds only what changes when the actor is automated.

### 10.1 The Cost Decision as a Controlled Decision

**The question a supervisor, an auditor or a finance business partner asks is not "was the saving real."** It is a fixed sequence, and an automated action has to survive all of it:

| Question | Acceptable answer | Unacceptable answer |
|---|---|---|
| Who approved this? | A named role, under a documented delegation, recorded on the action | "The agent did." "The system." "It was automated." |
| On what basis? | The evidence record: the utilisation window, the change record, the estimate and its confidence | The recommendation text |
| With what evidence, retained where? | Immutable record joined to the decision ID, retained per the bank's policy | A log line that rotates in 30 days |
| Is it reversible, and how? | The rollback path, tested, with the recovery point stated | "It should come back up" |
| What was the customer/SLA exposure? | The service owner's risk acceptance | Nothing — the question was not asked |

**"The agent did it" is not an answer because accountability cannot be delegated to software.** The institution can delegate *authority* to a system; it cannot delegate *answerability*. The only coherent form of an automated approval is: a named human or committee approved the **policy** under which this class of action may be taken without case-by-case approval, and a named human remains answerable for the outcome. That is exactly how straight-through processing works in payments, and cost actions should inherit the same governance grammar rather than invent a lighter one because the amounts look smaller.

### 10.2 Cost Data Quality as a BCBS 239-Adjacent Concern

**The discipline's banking overlay already treats cost attribution as a risk-data problem; automation raises the stakes in a specific way.** BCBS 239 is about risk data aggregation — accuracy, completeness, timeliness, and the ability to produce a defensible number under stress. A cost allocation that feeds a regulatory return or an entity-level P&L is the same class of artefact: it must be reconstructible, and its lineage must survive.

**The new failure is silent restatement.** An agent correcting tags "for hygiene" changes the basis of an allocation, which changes a prior period's cost by entity, which — if that allocation feeds a return or a transfer-pricing calculation — is a restatement nobody decided to make. The controls: tags declared as allocation inputs are outside the agent's write authority (§4.4); any batch that changes an allocation input requires the allocation owner's approval with a stated prior-period impact; and the before/after state is retained as lineage, not just as a log.

### 10.3 Model Risk and Change Management

**If the agent's output drives a financial decision, it is a model in the institution's model-risk sense, whatever the technology.** That brings the standard obligations: inventory it; assign an owner; validate it before use and periodically after; monitor it in production for drift and for the failure modes in §11; and control changes to it — including changes to its prompt, its tools, its policy and the model behind it. The uncomfortable consequence: a silent model upgrade by a provider is a model change, and if the bank cannot detect it, it cannot claim change control. The practical mitigation is a pinned version, a canary task class, and a golden set whose cost and quality results are compared across versions — the same machinery [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.4 and [agentops_guide.md](ai_llm/agentops_guide.md) §6.3 already require for cost-per-task regressions.

**An automated cost action is a change to a production system and belongs in the change record.** Not a parallel "automated changes" log — the same record, with the same fields, so that the change board's view of the estate is complete. An agent that resizes production outside the change process has created a class of change the institution cannot see, which is a control failure independent of whether the resize was correct.

### 10.4 When a Cost Action Becomes an Operational-Risk Action

**This is the argument that should end the "just automate the obvious ones" conversation.** A resize is a capacity decision; a capacity decision is an availability decision; an availability decision in a payment path is a conduct question with a regulatory tail. The arithmetic is brutally asymmetric: a modest monthly compute saving against an outage that breaches an SLA, triggers customer redress, and produces a supervisory conversation. No agent's expected-value calculation includes that tail, which is why the decision cannot be delegated to one.

**The practical ruling for a regulated entity:** the agent may execute only where the action's worst credible outcome is a re-run. Tag corrections, ticket creation, report generation, non-production scheduled stops — yes. Production capacity, deletion, commitments, anything touching a regulated workload or a legal-entity boundary — proposal, with the service owner's risk acceptance recorded. The line is not drawn at the size of the saving; it is drawn at the size of the worst case.

### 10.5 What the Second Line Will Ask For

**The artefacts that make an automated cost action defensible are the same six every time, and they are cheaper to build before the first action than after the first incident.** A risk function reviewing this programme will ask for them in roughly this order.

| Artefact | What it must contain | Owner |
|---|---|---|
| **Delegation policy** | The classes of action the agent may take without case approval, the ceilings, the named role that delegated, and the review date | FinOps lead, countersigned by the risk function |
| **Authority register** | Every action class, its quadrant (§4), its ceiling, its promotion evidence and its expiry | Platform owner |
| **Model inventory entry** | The agent as a model: purpose, owner, validation, monitoring, change control, version pinning | Model risk |
| **Decision record sample** | The §8.4 record for every executed action, retained per policy, with lineage | Platform owner |
| **Rollback evidence** | Proof that the rollback path was tested, not merely documented, per action class | Service owner |
| **Net reconciliation** | The §5.3 net, independently computed, with the review and error terms visible | Finance business partner |

**The one that is usually missing is the fifth.** Rollback is documented in every design and tested in almost none, and the moment it is needed is the moment nobody has time to test it. A resize with an untested rollback is an irreversible action wearing a reversible action's paperwork.

---

## 11. Failure Modes

**Every row below has been observed in some form in the practitioner material or follows directly from the design in §4–§9.** None of them is exotic; the exotic failure is the one where the model is simply wrong about a number, and that one is caught by review.

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| **a** | **The agent costs more than it saves.** Net negative for two consecutive months; review hours per task rising while findings fall | Value per task declining as easy waste is exhausted (§7.1); review cost never counted in the case; escalation rate creeping up | Break-even stated per workload before launch; the §5.3 net computed independently each period; a stated margin the net must clear; a quarterly keep-or-retire decision with real authority to retire |
| **b** | **The agent explains anomalies away instead of finding their cause.** "Expected" classifications rise, remediation falls, spend is flat | The agent optimising for a clean queue; no evidence required to close; suppression inside its authority | Closed enum for classification with a mandatory evidence pointer; "expected" requires a matching change record or forecast band; suppression and threshold changes outside the agent's authority; sampled audit of every "expected" |
| **c** | **Auto-resize breaks a service level.** Incident within the change window; latency; autoscale thrash; a failover that no longer fits | Utilisation window that excluded the peak; capacity treated as a cost variable rather than an availability one; no service-owner sign-off | Production outside execution authority (§4.3); stated window and percentile in the output; change record with rollback and recovery point; shadow-mode promotion per action class |
| **d** | **Unattributed agent spend.** An AI platform invoice nobody owns; the savings in one pack, the costs in another | No cost centre for the actor (§8.2); different schemas and period closes; the report's owner is the agent's owner | A real cost centre with a named human; cost and usage records emitted in the bill's normalised schema; the one-query test in §8.3 as an acceptance criterion |
| **e** | **The cost agent's own spend is invisible in the report it produces** | Template has no line for it; the amount feels immaterial; the conflict of interest is structural, not malicious | Mandatory template line with a non-zero value or an explicit zero and reason; netting performed by someone who is not the agent's owner; the §5.4 separation |
| **f** | **The agent optimises the metric, not the cost.** Unit cost falls, the bill does not; coverage percentage rises while commitment waste rises | A proxy metric that can be satisfied without the outcome; volume absorbing the rate improvement | Verification against the bill, per cost line, in the same period; proxy metrics allowed as leading indicators but never as the savings claim |
| **g** | **The same saving counted twice with a commitment purchase.** The rightsizing and the coverage analyses both claim the same capacity | Two teams, two analyses, no shared ledger; commitment modelled against current demand rather than post-change demand | A savings ledger keyed by cost line and period with one claim per line; commitment analysis must state which pending changes it assumes; decision ID on both claims |
| **h** | **A recommendation loop with no owner.** Ticket queue grows, action rate low, findings age past their relevance | Ownership resolution inferred but never verified; success measured by findings produced, not loops closed | Owner-confidence threshold before a ticket is raised, otherwise route to a named human queue; measure closed-loop rate and aged-finding count, not findings volume |
| **i** | **Authority creep.** Action classes appear in the policy register that nobody approved; write scope has widened | Authority granted at deployment and never reviewed; convenience changes shipped as configuration | A per-class authority register with an expiry; changes to it are a governed change; automatic demotion of a class after two material estimation errors in a quarter |
| **j** | **An action nobody requested, traced to text in a resource name, tag or ticket description** | Untrusted content treated as instruction by an agent with write access | Tool outputs and resource metadata are data, never instructions; write tools accept structured parameters only; no free-text-to-action path ([agentops_guide.md](ai_llm/agentops_guide.md) §6.4) |

**The two that will actually happen first are (b) and (e)**, and they share a property: neither looks like a failure. (b) produces a quieter queue and a cleaner report, which reads as maturity. (e) produces a report that credits the programme with savings and omits its cost, which reads as a strong business case. Both are the system working as designed against the institution's interest, and neither is detectable without a control that assumes it.

**A note on (a), because it is the one people expect and the one that is least often the real problem.** Agents rarely cost more than they save in the first quarter — the easy findings are plentiful and the novelty budget is real. The failure arrives in month seven to twelve, when the tail is being prospected, the review rate has crept up, and the case was written on month two's unit economics. Design for the tail from the start: a materiality floor below which a finding does not enter the queue, and a re-test of the case on the *current* population rather than the launch population.

### 11.1 Leading Indicators

**Every failure above is visible before it is expensive, in a series nobody plots.** These are the four worth a dashboard, because each turns early and each is cheap to compute from the §8 records.

| Indicator | What it shows | Turns early when |
|---|---|---|
| **Verified saving per task, rolling 3 months** | The value curve, against §7's break-even | Value is decaying faster than cost — failure (a) is forming |
| **Escalation rate and review minutes per escalation** | The review burden, and whether findings are getting harder | Both rising together means the tail has arrived |
| **"Expected" classification share, and evidence-completeness on audit sample** | Whether explanations are being generated or asserted | The share rises while remediation falls — failure (b) |
| **Estimation error on executed actions (estimate vs billed)** | Whether the pre-action estimate is trustworthy | Two material misses in a class — the signal to demote authority (§9.3) |

**What none of these measure, deliberately.** Findings produced, tasks run, tickets opened, messages sent. Those are the activity metrics that §13.1 argues against, and putting them on the same dashboard as the four above is how an activity metric gets promoted to a success metric in the second quarter.

---

## 12. Cymbal Bank Worked Example

> **Illustrative and fictional.** Cymbal Bank, its product lines and every number in this section are invented to demonstrate the method. The arithmetic is computed and internally consistent; it is not a finding about any real institution, and no figure here should be quoted as evidence of anything.

**The setting.** Cymbal Bank's FinOps practice sits in a hybrid model — central governance, federated execution — with allocation by cost centre, product line and legal entity, and showback established for fourteen product lines before chargeback. A platform team runs the agent runtime; the FinOps lead owns the practice and the pack. In month 1 they deploy one agent, **Ledger**, onto three activities chosen from §4 to sit in three different authority quadrants: **cost-anomaly triage** (reversible, no commitment — execute within policy for tickets and outreach), **tag hygiene** (reversible, no commitment — apply within narrow rules), and **showback assembly** (reversible, no commitment — generate and draft). They deliberately do *not* deploy rightsizing execution or commitment analysis, both of which stay in the analysis-and-propose quadrant.

### 12.1 Month 4 — The Loop Closed

| Line | Derivation | Amount |
|---|---|---|
| Anomaly-triage tasks | 1,800 tasks | 1,800 |
| Tag-proposal tasks | 900 tasks | 900 |
| Showback packs | 30 packs (14 product lines, weekly and monthly) | 30 |
| **Run cost** | 1,800 × $0.62 + 900 × $0.35 + 30 × $4.10 | **$1,554** |
| Platform charge | allocated gateway, storage, orchestration | **$260** |
| Human review | 50.5 h (320 escalated proposals at 9 min, plus 30 pack sign-offs) × $78/h | **$3,939** |
| Error cost | none material this month | **$0** |
| **Total cost** | 1,554 + 260 + 3,939 | **$5,753** |
| Verified saving — anomaly remediation | billed delta on 41 remediated findings | **$9,300** |
| Verified saving — tag-driven recovery | idle spend found through 612 corrected resources | **$1,450** |
| Released analyst time | 22 h × $78 (assembly and commentary, genuinely redeployed) | **$1,716** |
| **Total verified saving** | 9,300 + 1,450 + 1,716 | **$12,466** |
| **Net** | 12,466 − 5,753 | **$6,713** |
| Cost and value per task | 5,753 ÷ 2,730 and 12,466 ÷ 2,730 | **$2.11 / $4.57** |

**What the pack shows.** The showback Cymbal Bank publishes in month 4 contains, above the product-line table, a line reading `FinOps agent (Ledger) — cost $5,753 — verified saving $12,466 — net $6,713`, attributed to a cost centre owned by the FinOps lead and allocated to the product lines by findings attributable. The line is not a footnote. It is the first thing the CFO's office looks at, because it is the only line in the pack that has both a cost and a return on the same row.

**Three honest notes on the numbers.** The review cost is 68% of the total — the agent is a review-generation machine with a cheap engine. The $1,716 of released analyst time is counted only because the two analysts were redeployed to a forecasting project and the headcount was not replaced; had they spent the hours reviewing Ledger's output, that line would have been zero, and some of those hours are already consumed by the 50.5 review hours. And the $9,300 is verified against billing, not against the acceptance of the recommendation — 41 findings, each with a billed delta.

### 12.2 Month 7 — The Month That Was Not Clearly Positive

| Line | Month 7 | Note |
|---|---|---|
| Run cost | $1,680 | task volume up; more re-investigation per finding |
| Platform charge | $260 | flat |
| Human review | $4,056 | 52 h — escalation rate rose as findings got harder |
| Error cost | $2,900 | one tag batch misallocated a product line's spend; re-run plus a re-allocation exercise, plus one suppressed alert that had to be reconstructed |
| **Total cost** | **$8,896** | |
| Verified saving — anomalies | $5,100 | the easy remediations are done; the tail pays less |
| Verified saving — tags | $610 | most correctable resources already corrected |
| Released analyst time | $1,716 | unchanged |
| **Total verified saving** | **$7,426** | |
| **Net** | **−$1,470** | |

**What they did, in order.** First, they did not adjust the savings definition — the failure mode §11(f) describes is exactly what a team does when a number turns negative, and the FinOps lead had pre-committed to the definition in the month-4 pack. Second, they set a **materiality floor**: an anomaly must carry an estimated monthly value above a stated threshold to enter the queue, which cut triage volume by a third and review hours by half. Third, they narrowed tag execution to high-confidence inferences only, added the allocation-input exclusion that should have been there from month 1 (the error came from a tag that fed a product-line allocation), and moved tag review to a sample rather than a full read. Fourth, they moved triage to two-tier routing — a cheap model for classification, escalation to the stronger model only on ambiguous cases.

**Month 8, after the changes:**

| Line | Month 8 |
|---|---|
| Tasks | 1,200 triage + 300 tag + 30 packs = 1,530 |
| Run cost | 1,200 × $0.58 + 300 × $0.35 + 30 × $4.10 = **$924** |
| Platform + review | $260 + (30 h × $78) = **$2,600** |
| Total cost | **$3,524** |
| Verified saving | 6,400 + 520 + 1,716 = **$8,636** |
| **Net** | **$5,112** |
| Cost and value per task | **$2.30 / $5.64** |

**The lesson is not "it recovered."** The net recovered and the absolute verified saving did not: $8,636 against month 4's $12,466. Ledger is doing less, more cheaply, because there is less easy waste to find. The right conclusion is not that the agent failed or that it succeeded, but that its value has a shape — front-loaded — and the quarterly review exists to notice the shape rather than to defend the launch case. Cymbal Bank kept Ledger, cut its authority on tags to the narrowest class, and moved its keep-or-retire decision to a quarterly cadence with an independent netting by a finance business partner who does not report to the FinOps lead.

### 12.3 The One Decision the Agentic Nature Changed

**The decision: tag corrections moved from "recommend to the owner" to "execute within policy." Rightsizing stayed human. Both decisions were taken in the same review, and the reasoning differs from the ML-era reasoning in a way worth stating precisely.**

In the ML era, Cymbal Bank already had auto-remediation: a rule that applied a default cost-centre tag when the tag was absent, and stopped non-production instances on a schedule. The governance question then was **"is the rule right?"** — a static artefact, reviewable once, changed rarely, and validated by inspecting the rule. That question has a comfortable answer, so the automation was approved and it worked.

With Ledger, the artefact under governance is not a rule but a **generator of per-case inferences**. The question becomes **"is the space of possible outputs bounded, and can we prove it for the case in front of us?"** That is a different kind of question, and it forces a different control design: allow-listed keys, values validated against the cost-centre master list, a confidence threshold, a batch cap, an exclusion for any tag that is an allocation input, and lineage retained for every change. Once those bounds existed, execution was *safer* than the ML-era rule — because a deterministic rule with a stale mapping silently mis-tags forever, while the agent's inferences are individually evidenced and individually reversible.

**Why rightsizing went the other way.** The agent's recommendation accuracy on production rightsizing was at least as good as the capacity team's — measured on the shadow-mode agreement rate, it matched on the majority of cases. Under the ML-era logic, that would have justified automation: the recommendation is accurate, so let it act. The agentic framing refuses that inference, because accuracy was never the binding constraint. The binding constraint is that the *worst credible outcome* of a wrong resize is an availability incident in a payment path, and the agent's estimate does not and cannot include that tail (§10.4). The trust question moved from "is the model accurate enough" to "who is answerable when it is wrong, and what is the blast radius" — and that question has a different answer for a tag than for a production resize, no matter how good the model gets.

**That is the generalisable shift.** ML-era automation asked whether the rule was correct and, if so, delegated. Agentic automation asks what the action can destroy and who answers for it, and delegates only within the answer. Accuracy is a precondition, not a justification.

**The thesis, restated on this case.** Ledger's own $5,753 sat inside the pack it compiled, attributed to a named owner, netted against verified savings by someone who did not build it. In month 7 the net was negative and the number said so, because the report included the review hours and the error cost — the two lines a programme omits when it is defending itself. An agent that optimises cost is itself a cost to be optimised, and Cymbal Bank could only optimise it because it could see it.

---

## 13. Anti-Patterns

**Twelve ways this goes wrong, each with the corrective in a clause.** They are ordered roughly by how early they appear.

1. **The Invisible Optimiser.** The agent compiles the showback and its own spend is not in it — the report is a lie of omission and the business case is a forgery with good intentions. *Fix:* a mandatory agent line in the template, explicit zero or real number.
2. **The Confident Explainer.** A fluent causal narrative with no citations, which authorises an action because it reads well. *Fix:* every causal claim carries a record pointer; reviewers check citations, not prose.
3. **Auto-resize as a Policy.** Capacity decisions delegated by writing a policy once and never revisiting it, on the theory that the policy was the hard part. *Fix:* per-action-class authority with an expiry and shadow-mode promotion.
4. **Savings Theatre.** Recommendations accepted counted as savings, dashboards celebrating findings, a "savings pipeline" that never touches a bill. *Fix:* savings verified against billing, per line, in the period.
5. **The Agent That Owns Its Own Budget.** The agent proposes its own spend, measures its own return, and decides it clears the bar — a control that certifies itself. *Fix:* the §5.4 separation; independent netting.
6. **The Queue With No Owner.** Success measured by findings produced, so the agent industrialises a backlog instead of closing loops. *Fix:* closed-loop rate and aged-finding count as the metrics.
7. **Lift-and-Shift the Dashboard.** A chat interface bolted onto the existing cost report — natural language over the same passive artefact, no investigation, no action, no loop. *Fix:* ask which of the five steps it occupies; if only step 1, it is a UI, not an agent.
8. **The Forty-Agent Org Chart.** An orchestrator and a dozen "consultant agents" for a workload one agent and three tools could carry. *Fix:* specialists behind a router only where the tool surface or the permission boundary genuinely differs.
9. **Trust the Model, Skip the Ledger.** No decision ID, savings reconstructed from memory and screenshots, double counting discovered at the annual audit. *Fix:* the §8 record, and a ledger keyed by cost line and period.
10. **The Permanent Pilot.** Shadow mode forever: no authority, no saving, a demo that has been running for eighteen months and appears in every deck. *Fix:* a promotion date per action class, and a retirement date if it is not met.
11. **Costing the Tokens, Not the Review.** A business case built on model spend, approved because it looks trivially cheap, then quietly sunk by reviewer hours. *Fix:* the §5.3 net, with review and error cost as first-class terms.
12. **The Greenfield Assumption.** The agent deployed into an estate with no tagging discipline and no allocation model, where it automates the production of noise and calls the volume productivity. *Fix:* the discipline first — the agent amplifies whatever allocation quality already exists, in both directions.

### 13.1 The Pattern Behind the Patterns

**The agent is judged on activity — tasks run, findings produced, messages sent — because activity is measurable on day one and savings are measurable in month three.** Every anti-pattern above is a version of that substitution: the Invisible Optimiser measures cost avoided by not counting it, Savings Theatre measures recommendations, the Queue With No Owner measures findings, the Permanent Pilot measures demos. None of these is dishonest at the level of the individual metric; each is a proxy chosen because the real measure reports late, and each becomes a lie once it is the metric the programme is managed on.

**Two corrections, both structural rather than cultural.** First, **the unit of account becomes the closed loop**: a finding that reached an owner, produced an action, and showed a billed delta, counted once, in the period the delta appeared. Closed loops per period is a small number, it is slow to accumulate, and it cannot be inflated by producing more findings — which is exactly why it is the right number. Second, **the savings ledger carries a status per line and a claim owner**, so that "pipeline" and "verified" are different columns and a pipeline number can never be presented as a result. The discipline already distinguishes showback from chargeback on a maturity ladder; the same instinct applies here: a claimed saving and a verified saving are different instruments, and conflating them is how a programme loses the finance function's trust in year two.

**The test to apply to any FinOps agent proposal, in one question:** *what does the report show in the month the agent is wrong?* If the answer is "nothing changes," the programme has no feedback loop and the anti-patterns above are already present, whatever the architecture diagram says.

---

## 14. State of Practice

**The dated primary source is a Foundation publication, and it is a synthesis of practitioner conversations rather than a measurement exercise.** That distinction governs how much weight each claim can carry, so this section separates what practitioners reported doing from what the market promises.

**What practitioners report doing now** (Morley, "AI for FinOps: Agentic Use Cases in FinOps," published by the Foundation, 27 March 2026 — practitioner interviews and community synthesis):

| Capability | What the source actually reports | Maturity signal |
|---|---|---|
| **Report assembly and financial reconciliation** | Agents ingest budgets and forecasts, compare to actuals, flag deviations above thresholds, and generate delta reports across providers without a human opening a spreadsheet | Described as a recurring theme across maturing practices — the most widely adopted cluster |
| **Agentic waste discovery** | The agent investigates the resource, resolves tags and IDs, identifies the owner, and opens and assigns a ticket — orchestrator plus specialist "consultant" agents in the more sophisticated implementations | Multiple practitioners; one team reported initial investigation time per ticket falling from about 15 minutes to "essentially zero" — a single team's self-report, unaudited |
| **Proactive guardrails in the pull request** | Background agents check configuration against cost and policy guardrails before deployment — shift-left at the point of decision | Reported as an established pattern, extending the CI cost-gate idea the discipline already has |
| **Contextual resource labelling** | Agents infer the purpose of unlabelled resources from when they run and what they connect to, to improve showback and accountability | Described as exploratory |
| **Personalised outreach** | Agents identify the last person to interact with a service and send a tailored message; one practitioner reported a 40–50% action rate on those messages | One practitioner's self-reported figure, not a finding |
| **Security-score pull requests** | Agents open PRs addressing cloud security findings, with the visible score as the leadership metric | Reported anecdotally; the source itself notes hours saved are hard to quantify |

**What the same source says about the limit.** Most organisations are not yet comfortable letting agents make production changes or delete resources without human approval; agents currently act as co-designers and companions rather than autonomous decision-makers; and model inaccuracy — "slop" — makes verification a required step in any agentic workflow. That is the trust gap, stated by the Foundation rather than by a sceptic, and it is the empirical basis for §4's rulings.

**What is vendor promise or unproven**, and should be treated as such in any business case:

| Claim | Status |
|---|---|
| Autonomous production change with no human in the loop | Not demonstrated in the primary source; explicitly outside current comfort. Every vendor demo of this is a demo |
| Autonomous deletion of resources | Same, with a worse downside; the reversibility argument in §4 does not weaken with model quality |
| Autonomous commitment purchase | Not a technical problem at all — it is an authority problem, and no accuracy figure changes it |
| Self-directed budget ownership by the agent | No serious proposal exists outside marketing; it fails the §5.4 separation on its face |
| A single "FinOps agent" that covers the whole lifecycle end to end | The observed pattern is narrow agents plus specialists behind a router; the generalist claim is a packaging claim |

**Two structural observations from the source that outlive any specific tool.** First, the **new unit economics**: "cost per thought" and "token use budgeted" per project, with the API cost of running the agents becoming as important as the cost of the compute they optimise — which is this guide's thesis arriving from the practitioner side. Second, the **Innovation Value Paradox**: demanding a positive NPV from every experiment strangles the experimentation that finds value, and the reported response is shorter funding cycles with more frequent review rather than heavier gates. For a FinOps agent programme that is directly applicable: fund it in quarters, review the net quarterly, and do not require it to justify itself on a three-year NPV it cannot honestly produce.

**And the role shift, which is the point of the whole exercise.** Practitioners describe moving from "doing the work" to "orchestrating the workers," with the path from agent-assisted to agent-executed, and the practitioner's value migrating to defining the guardrails and objectives the agents operate within. That is the same conclusion §4 reaches from the control side: the human's job becomes the boundary, not the task.

### 14.1 What to Ask a Vendor

**Five questions separate a FinOps agent from a cost dashboard with a language model in front of it, and none of them is about model quality.** They are worth asking in this order, because each one is harder to answer dishonestly than the last.

| Question | A serious answer sounds like | A marketing answer sounds like |
|---|---|---|
| Which of the five steps does it occupy, and where does it stop? | "It finds, explains and drafts; it opens tickets; it does not execute on production." | "It's fully autonomous end to end." |
| What does it do when its explanation is wrong? | A description of the falsifier field, the review step, the estimation-error metric | "Our model is highly accurate." |
| How does its own spend appear in the report it produces? | "Metered per call, allocated to your cost centres, on its own line." | "Pricing is per seat" or per task, with no mention of the report |
| What is the audit record for an action it takes? | Decision ID, evidence pointer, approval, rollback path, retention | A screenshot of an activity log |
| What is the net, and how would we verify it against our bill? | An offer to run a shadow-mode period with the net defined before launch | A savings percentage from a reference customer |

**The tell is the fourth question.** An agent that can act has an audit record or it is a liability; a vendor that cannot describe the record in the first meeting has not built one. The second tell is the fifth: anyone offering a savings figure before seeing the estate is offering a number about their other customers, and the discipline this guide sits inside has spent a decade learning not to buy those.

---

## 15. The Claims Audit

**Every substantive claim in this guide, with its status, its source and date, and what kind of evidence it is.** ✅ verified against a primary or dated source · ⚠ partly verified, time-varying, or illustrative · ❌ could not verify.

| Claim | Status | Source and date | Kind |
|---|---|---|---|
| Agents are being used for report assembly, waste discovery with ticket creation, PR-time cost guardrails, contextual labelling and outreach in FinOps practice | ✅ | Morley, "AI for FinOps: Agentic Use Cases in FinOps," FinOps Foundation, 27 Mar 2026 (indexed 3 Apr 2026) | Qualitative synthesis of practitioner interviews; no sample or method published |
| The orchestrator-plus-specialist-"consultant-agent" pattern is used in more sophisticated implementations | ✅ | Same, 27 Mar 2026 | Architecture description, not a benchmark |
| A trust gap exists: most organisations are not comfortable with autonomous production change or resource deletion without human approval; agents act as co-designers, not autonomous decision-makers | ✅ | Same, 27 Mar 2026 | Practitioner consensus reported by the Foundation |
| Model inaccuracy ("slop") makes verification a required step in agentic workflows | ✅ | Same, 27 Mar 2026 | Practitioner consensus |
| "Cost per thought" and "token use budgeted" are emerging unit metrics, and the API cost of running agents is becoming as important as the compute they optimise | ✅ | Same, 27 Mar 2026 | Conceptual claim; no figure attached |
| FinOps for AI is the top forward-looking priority for FinOps teams | ✅ | State of FinOps 2026, as stated in the Foundation's 27 Mar 2026 article | Survey, self-reported; the underlying instrument was not inspected for this guide |
| 98% of FinOps practices manage AI spend | ⚠ | Same, 27 Mar 2026 | Self-reported survey figure; sample and methodology not published in the article — directional only |
| One team reduced initial investigation time per ticket from ~15 minutes to "essentially zero" | ⚠ | Same, 27 Mar 2026 | Single team, self-reported, unaudited; not a population estimate |
| One practitioner reported a 40–50% action rate on agent-authored Slack outreach | ⚠ | Same, 27 Mar 2026 | One practitioner's self-report; explicitly not a finding |
| The Innovation Value Paradox: NPV demands on every experiment suppress experimentation; the reported response is shorter funding cycles with more frequent review | ✅ | Same, 27 Mar 2026 | Practitioner observation reported by the Foundation |
| The practitioner role is shifting from doing the work to orchestrating it, with value migrating to defining guardrails and objectives | ✅ | Same, 27 Mar 2026 | The author's thesis, grounded in practitioner conversations |
| The FinOps Framework is organised around Principles, Personas, Phases, Maturity, Domains, Capabilities, Scopes and Technology Categories, with a March 2026 poster revision | ✅ | finops.org/framework, accessed Oct 2026; poster dated March 2026 | Primary organisational source |
| An open specification normalises billing and usage data across AI, cloud, SaaS and data-centre vendors, current version 1.4 | ✅ | focus.finops.org, accessed Oct 2026 ("Introducing FOCUS 1.4") | Primary specification source |
| The cost-of-agents control stack (per-task caps, budget layering, gateway metering, caching and routing levers, cost per task as the unit metric) | ✅ | [agentops_guide.md](ai_llm/agentops_guide.md) §6.2–6.3; [production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) §7.1–7.5 | Internal cross-reference, read and verified against both owner sections |
| The FinOps guide's AI material is ML-based (detect, forecast, recommend) and its "FinOps for AI/ML" paragraph concerns GPU/inference cost rather than an agent actor | ✅ | [finops_guide.md](finops_guide.md) §17, read directly | Internal cross-reference; the section's four paragraphs checked |
| The five-step decomposition, the reversibility × commitment test, the §5.3 net formula and the §8 record schema | ⚠ | This guide, Oct 2026 | Design framework — a reasoned construction, not an empirical finding; no study validates the specific quadrant boundaries |
| The failure modes in §11 are reasoned from the design and from practitioner-reported problems | ⚠ | This guide §11, Oct 2026 | Design reasoning plus the reported "slop"/verification problem; no incident dataset was located |
| Cymbal Bank's month 4, 7 and 8 figures | ⚠ | This guide §12, Oct 2026 | Fictional and illustrative; arithmetic computed and checked for internal consistency, not a measurement of any real estate |
| **No cloud-waste, savings, idle-resource, market-size or tool-ranking percentage is printed anywhere in this guide** | ❌ | — | **Deliberate omission.** No dated primary source publishing such a figure *for an agentic FinOps actor, net of its own cost* was located; vendor savings claims are marketing, single-practitioner anecdotes are not population estimates, and a percentage printed here would be repeated elsewhere without its caveats. The only numbers printed are the survey statements above (attributed, with method) and the labelled illustrative arithmetic in §7.2, §8.4 and §12 |

**Verification method.** The FinOps Foundation article was read in full from a cached copy of the published page (27 March 2026, indexed 3 April 2026). The Framework and FOCUS pages were retrieved live in October 2026 and their structural claims checked against the page text. The three owner sections were read in the repository rather than summarised from memory. No figure in this guide originates from a vendor page, a consultancy report or a secondary summary, and none was inferred from a chart.

---

## 16. What Could Not Be Verified, Glossary and Closing

### What Could Not Be Verified

**Six honest gaps, each of which limits a specific claim above.**

1. **No published, methodologically disclosed measurement of a FinOps agent's net return.** The Foundation's article is a synthesis of practitioner conversations; it reports what people are building and what one team observed, not a measured population outcome. The State of FinOps survey figure it cites is self-reported, and its instrument was not available for inspection. **Consequence:** this guide can defend the *design* of the loop in §5 and cannot defend any expected value from it. Anyone who quotes a return figure for an agentic FinOps programme is quoting a vendor or extrapolating from an anecdote.
2. **The cost of human review of agent output is asserted, not sourced.** The claim that review is typically the dominant cost term (§7.2, §12) rests on the arithmetic of the illustrated case and on the general shape of human-in-the-loop systems. No study of review rates for FinOps agents was located, and review rates will vary with task class, reviewer seniority and how good the explanations are. **Consequence:** the ratio in §12 should be treated as a warning to measure it, not as a benchmark to plan against.
3. **Error rates and their cost are unquantified.** §5.3 requires an error-cost term; no source quantifies how often an agent mis-estimates a cost delta, misattributes an owner, or classifies an anomaly as expected without cause. The failure modes in §11 are reasoned from design and from the practitioner-reported "slop" problem, not from incident data. **Consequence:** the error term must be estimated locally from a shadow-mode period, and the first quarter's number will be wrong.
4. **Whether agent-authored outreach actually changes behaviour at scale.** The single 40–50% action-rate report is one practitioner, self-reported, with no denominator disclosed (messages sent? unique recipients? repeat messages?). It is cited here as an anecdote and nothing in the argument depends on it.
5. **The regulatory treatment of automated cost actions specifically.** §10's argument is built by analogy from established expectations — BCBS 239 risk-data principles, model-risk frameworks, change management, operational-risk and conduct considerations — and from the banking overlay in [finops_guide.md](finops_guide.md) §11. No supervisor has published guidance on an agent taking cost actions, and no enforcement or supervisory case was located. **Consequence:** §10 describes a defensible governance posture, not a compliance requirement, and a bank should treat it as a design argument to be validated by its own second line.
6. **The specific quadrant boundaries in §4.** The reversibility × commitment test is this guide's construction. Reasonable practitioners will draw the line differently — some will allow agent-executed production stops outside business hours, some will not allow agent-executed tag writes at all. The test is offered as a defensible method with a stated rationale, not as a standard, and the per-activity rulings are arguments to be overruled by an institution's own risk appetite.

### Glossary

| Term | Definition |
|---|---|
| **Agentic FinOps** | The use of agents that occupy two or more of the five steps — finding, explanation, proposed action, execution, approval — within a FinOps activity, as distinct from ML that ends at detection or recommendation |
| **Attribution** | The mapping of a cost record to an owner dimension: agent, task class, cost centre, product line, legal entity |
| **Break-even value per task** | Total cost of a workload (run, platform, review, error) divided by tasks completed; the verified saving per task must exceed it |
| **Chargeback** | Billing actual consumption against a team's budget, as opposed to showback |
| **Commitment** | A term purchase — reserved instance, savings plan, committed use discount — exchanging flexibility for rate |
| **Cost centre mapping** | The maintained join from an AI-platform tenant to the financial dimensions used by the allocation model; the thing that fails silently on reorganisation |
| **Cost per task** | All metered spend plus human review for one completed unit of agent work |
| **Deadweight saving** | A saving claimed by the agent that would have occurred without it |
| **Decision ID** | The identifier joining an agent's metered calls, its action, its approval and its saving claim — the junction that makes the loop computable |
| **Materiality floor** | The estimated value below which a finding does not enter the queue; the control that keeps the tail from consuming the head |
| **Net saving** | Verified saving − agent run cost − human review cost − error cost, measured over the saving's own realisation window |
| **Rightsizing** | Matching provisioned capacity to measured need; a capacity decision with cost consequences, not a cost decision |
| **Savings ledger** | The record of verified savings keyed by cost line and period, with one claim per line, that makes double counting detectable |
| **Shadow mode** | Running an agent's proposed actions without executing them, to measure agreement and estimate accuracy before granting authority |
| **Showback** | Reporting cost to the teams that caused it without moving budget |
| **Unit economics** | Cost and value per business unit of output — per transaction, per report, per task |
| **Kill switch (authority revocation)** | The tested path that withdraws an agent's write credentials while leaving it running read-only; distinct from stopping a task |

### Cross-References

| Guide | What it owns | Where this guide uses it |
|---|---|---|
| [finops_guide.md](finops_guide.md) | The discipline: principles, lifecycle, maturity, personas, pricing, allocation, showback/chargeback, optimisation levers, Kubernetes, multi-cloud, banking, tools, metrics | §1 boundary, §4 allocation and tag constraints, §6 cost-centre mapping, §10 banking overlay (§11 of that guide) |
| [ai_llm/agentops_guide.md](ai_llm/agentops_guide.md) | Cost of agents: per-task caps as a financial circuit breaker, budget layering, gateway metering, cost per task, anomaly detection, tool permissioning | §5.1 step 3, §6 in full, §9 perimeter controls, §11(j) |
| [ai_llm/production_ready_llm_agents_guide.md](ai_llm/production_ready_llm_agents_guide.md) | Operational cost control: token budgets, caching, routing, cost observability, the cost-controls table | §6 in full, §9, §10.3 |
| [ai_llm/enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md) | Platform-level enforcement of budgets, routing and tenancy | §6 (the platform that meters the actor) |
| [../management/business_case_development_guide.md](../management/business_case_development_guide.md) | The investment case: TCO, NPV, funding cycles | §5.3 (a net-positive loop is the case), §14 (shorter funding cycles) |
| [../banking/treasury_alm_guide.md](../banking/treasury_alm_guide.md) | The banking cost and capital context the commitment decision sits inside | §4.6, §10 |

### Closing Summary

**Five rulings carry the guide.** The agent may occupy the finding, the explanation and the proposal on any activity; execution only where the action is reversible and commits nothing; approval of anything irreversible, production-affecting or commitment-creating stays with a named human. The agent's own metered spend is a mandatory line in the report it produces, attributed to a real cost centre with a named owner. The net is verified saving minus run cost minus review cost minus error cost, measured over the saving's own window, and the term that decides the programme is the one everyone omits — error cost. The keep-or-retire decision is taken by someone who is not the agent, because a control that certifies itself is not a control. And the human's job moves from doing the work to defining the boundary the work happens inside — which is a promotion, not a demotion, and also the only version of this that a regulator, an auditor and a CFO will all accept.

**The gap this guide fills is narrow and real.** The discipline's AI material detects, forecasts and recommends, and ends at a dashboard. An agent can carry the recommendation through the last mile to a verified change — and the moment it does, it acquires a bill, a set of claims and a set of obligations that the discipline's existing material does not cover. Its spend must be in the ledger, its claims must survive an audit, and its actions must have an owner. Those are not AI problems. They are FinOps problems, which is why they belong here.

**An agent that optimises cost is itself a cost to be optimised.**
