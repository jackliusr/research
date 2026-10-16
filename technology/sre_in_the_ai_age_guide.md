# SRE in the Age of AI: What Can Be Measured, Who Is On Call, What Counts as a Control - A Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Series:** Technology Guides — the operations, reliability and AI-engineering cluster
> **Audience:** Solution architects, SRE and platform engineers, incident and problem managers, AI-platform owners, and the engineering leadership who must sign a reliability target for a service backed by a model.

*The guide for the moment a reliability target has to be attached to a service whose behaviour is not a function of its inputs. It does not re-explain the SRE canon, and it does not re-explain AI in IT operations. Six siblings own the components: the canon itself — toil, error budgets, the freeze mechanics, ITIL, the on-call rotation and the escalation matrix — is owned by [Operational Support Frameworks](operational_support_frameworks_guide.md), chiefly its §6 "The SRE Overlay"; AI in IT operations — AIOps, anomaly detection, alert-noise reduction, root-cause analysis, auto-remediation — is owned by [AI for IT](ai_llm/ai_for_it_guide.md); the operation of AI agents in production is owned by [AgentOps](ai_llm/agentops_guide.md); the SLI/SLO/error-budget derivation, "100 is the wrong target" and the deployment strategies are owned by [Zero-Downtime System Design](zero_downtime_system_design_guide.md); failure injection, steady-state hypotheses and game days are owned by [Chaos Engineering](chaos_engineering_guide.md); and the ML lifecycle, the drift taxonomy and the retraining loop are owned by [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md). What this guide owns is exactly one subject: **the SRE canon read forward into a world where AI is at once a tool inside the operating model and a workload with its own reliability problems — the reconciliation, not the components.***

**Series / boundary declared by name.** Across this repository the reliability vocabulary is already everywhere: `\bSRE\b` appears in **41 files**, `observability` in **185**, `runbook` in **133**, `on-call|incident response` in **230**, `error budget|SLO|SLI` in **49**, `AIOps` in **7**, `toil` in **8** and `site reliability` in **9** — all counts verified by the parent session on **2026-10-16** — and yet **no `sre_guide.md` exists**. The neighbourhood is the most heavily claimed in the repository, which means the only real failure mode for a new guide in it is *duplication*: restating what a sibling already carries under a different heading and calling it a contribution. This guide therefore treats six adjacent guides as owners and points at them by name rather than re-deriving their material. The demarcation is one sentence: **[Operational Support Frameworks](operational_support_frameworks_guide.md) §6 owns the canon, [AI for IT](ai_llm/ai_for_it_guide.md) owns AI *in* operations, and this guide owns the question that neither asks — what happens to the canon's core objects when the system being operated is itself a model.** The canon asks how a human service is kept reliable; the AIOps guide asks how AI makes that easier; this guide asks what of the canon survives when the *thing* being kept reliable is non-deterministic.

**What this guide does NOT do.** It does not restate the toil definition, the six attributes of toil, the fifty-percent operational-work rule, the on-call cap, the error-budget arithmetic, the launch-freeze mechanics, the engagement models or ITIL — [Operational Support Frameworks](operational_support_frameworks_guide.md) §4.3–§4.4 and §6.2–§6.4 own all of it, and the overlap guide's §6.3 already quotes the freeze sentence, the postmortem-trigger threshold, the outages statistic and the "1,000 errors per million requests" arithmetic. It does not explain anomaly detection, alert-noise reduction, root-cause analysis or auto-remediation mechanics — [AI for IT](ai_llm/ai_for_it_guide.md) §4, §5 and §8 own them. It does not re-derive the SLI/SLO derivation or "100 is the wrong target" — [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3 owns them. It does not re-derive a drift taxonomy ([MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md)) or failure-injection mechanics ([Chaos Engineering](chaos_engineering_guide.md)). It does not re-explain the agent SLI set or the agent-on-call material — [AgentOps](ai_llm/agentops_guide.md) §3.5 and §5.4 own those, and this guide **cites the honest flag in §5.4** rather than overclaiming a codified practice.

**Jurisdiction and sourcing constraint.** This guide describes AI-era reliability **structurally** — what a threshold, a budget, a roster and a postmortem each become when the measured system is non-deterministic — and prints **no mean-time-to-detect figure, no mean-time-to-repair figure, no benchmarking figure, no market size, no vendor ranking and no vendor market share**. No framework, metric figure, provider behaviour, article or clause is asserted from memory. A named source appears below **only** where the dated primary source was given to this pass (four `sre.google` pages, listed in §15) or where the claim is cross-referenced to the sibling guide that owns it. Everything else is described as structure or as **this guide's own argument**, labelled as such. **Every institution in the worked example below is Cymbal Bank, a fictional bank; no real bank's internal arrangements are described anywhere in this guide.** No vendor is named, ranked, sized or compared.

**Cross-references used throughout:** [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) (the canon: §4.3–§4.4 on-call and escalation, §6.1–§6.4 the SRE overlay), [ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md) (AIOps, anomaly detection, RCA, auto-remediation, and §11 the regulated estate), [ai_llm/agentops_guide.md](ai_llm/agentops_guide.md) (agent observability, §3.5 the agent SLI set, §5.4 the on-call flag), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (§2.3 the availability derivation, §3 deployment strategies, §7.3 game days), [chaos_engineering_guide.md](chaos_engineering_guide.md) (failure injection, steady-state hypotheses), [mlops_lifecycle_frameworks_guide.md](mlops_lifecycle_frameworks_guide.md) (the ML lifecycle and drift taxonomy), and, cross-referenced only, [ai_llm/ai_governance_framework_guide.md](ai_llm/ai_governance_framework_guide.md), [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md), [constrained_decoding_frameworks_guide.md](constrained_decoding_frameworks_guide.md), [../banking/](../banking/), [../management/grow_team_guide.md](../management/grow_team_guide.md), [../management/high_performance_team_guide.md](../management/high_performance_team_guide.md), [../management/team_lead_methodologies_guide.md](../management/team_lead_methodologies_guide.md), [../management/authority_skills_guide.md](../management/authority_skills_guide.md) and [../personal/agent_runtime_engineer_skill_gaps_guide.md](../personal/agent_runtime_engineer_skill_gaps_guide.md).

**Status legend (used in the claims audit and throughout):** ✅ verified at a primary source this pass · ⚠ partially verified (secondary source, or primary source confirms only part) · ❌ could not be verified.

### Table of Contents

1. [Overview, Decoder, Thesis, and the Seam Declared by Name](#1-overview-decoder-thesis-and-the-seam-declared-by-name)
2. [The Canon in One Page, Cross-Referenced](#2-the-canon-in-one-page-cross-referenced)
3. [What AI Changes About Measurement](#3-what-ai-changes-about-measurement)
4. [What AI Changes About the Error Budget](#4-what-ai-changes-about-the-error-budget)
5. [What AI Changes About Toil](#5-what-ai-changes-about-toil)
6. [What AI Changes About On-Call](#6-what-ai-changes-about-on-call)
7. [What AI Changes About the Postmortem](#7-what-ai-changes-about-the-postmortem)
8. [SRE for AI Systems - The Other Direction](#8-sre-for-ai-systems---the-other-direction)
9. [The Reliability of a Non-Deterministic Component](#9-the-reliability-of-a-non-deterministic-component)
10. [The Role and the Skills](#10-the-role-and-the-skills)
11. [The Regulated Estate](#11-the-regulated-estate)
12. [The Failure Modes](#12-the-failure-modes)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

### Reading paths

- **SRE and platform engineers** — §3 (why an eval is not an availability figure), §4 (what a burn rate assumes), §8 (the model provider as a dependency), §9 (the statistical threshold and variance reduction), §12 (the failure modes your dashboards cause).
- **Incident and problem managers** — §6 (the agent-assisted responder versus the agent as responder), §7 (blamelessness when the actor was a model), §13 (the worked decision), §14 (the anti-patterns).
- **Solution architects and AI-platform owners** — §1 (the seam), §3 and §8 (measurement and the target), §11 (the reliability artefacts inside the control set), §15 (the claims audit).
- **Engineering leadership who must sign a target** — §1 (the seam), §4 (what a budget on a model-backed service can and cannot mean), §10 (what changes for the team and what does not), §16.4 (the closing summary).
- **General readers** — §1, §2, §9, §13, §16 (the thesis, the canon, the central argument, the example, the close).

Each section stands alone; the tables are the quick reference, and every cross-reference carries the reader to the sibling guide that owns the underlying detail.

---

## 1. Overview, Decoder, Thesis, and the Seam Declared by Name

### 1.1 The short answer

Site reliability engineering is a way of running a service: you write down what the user is promised, you measure it as an indicator, you set a target, you treat the gap between the target and perfection as a budget you are allowed to spend, you cap the operational work so that engineering can scale the service instead of labour, and when the target is missed you write a blameless account of what happened. None of that machinery is about deterministic software in particular. It is a discipline for *deciding how much failure is acceptable and for keeping the decision honest*.

AI, read forward into that discipline, changes three of its inputs and none of its ends. It changes **what can be measured** — because the unit being measured may be a sampled quality statistic rather than an event count, and a sampled statistic carries a sampling error that a deterministic indicator does not. It changes **who is on call** — because the first responder may be an agent, and because the service may fail silently in a provider's model rather than loudly in your own code. It changes **what counts as a control** — because a guardrail that blocks a valid request is itself a reliability event, and because a provider's model change is a change over which you have no change control. What AI does *not* change is what reliability means: a reliability target is still a promise to a user about a distribution of outcomes, and the discipline of promising no more than you can observe and no less than the user needs is exactly as hard, and exactly as necessary, as it was before.

The short answer, then, is this. The canon is not obsolete and the AI-native operating model is not a replacement for it; the canon is a measurement-and-accountability framework, and when the measured system becomes non-deterministic the framework must be *re-grounded on what can actually be observed*, not abandoned. This guide is that re-grounding, done once and honestly.

### 1.2 The decoder

The vocabulary below is used in one sense throughout the guide and no other. The third column is the important one — it names the guide that owns the detail, so that this guide can point rather than restate.

| Term | What it means in this guide | Owner of the detail |
|---|---|---|
| **SRE** | The discipline of running a service to an explicit reliability target, treating the shortfall from perfection as a budgeted resource and operational work as a cost to be capped. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §6 |
| **SLI** | A service level *indicator*: the thing you actually measure, expressed as a number over a window. | [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3 |
| **SLO** | A service level *objective*: the target band you set for an indicator. A target is a promise about a distribution, not a guarantee about any single request. | [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3 |
| **SLA** | The contractual version of an SLO, with a consequence attached; the customer-facing commitment, typically looser than the internal target. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §4 |
| **Error budget** | The tolerable amount of the indicator's bad outcome, derived from the gap between the SLO and perfection; spending it is permitted, exhausting it triggers the policy. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.3 |
| **Burn rate** | The speed at which the error budget is being consumed; a rate over a window, used to alert before the budget is gone. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.3 |
| **Toil** | Operational work that is manual, repetitive, automatable, tactical, devoid of enduring value and, in a sense to be argued, scales linearly with service size. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.2 |
| **Postmortem** | A written, blameless account of an incident: the contributing causes, the timeline, and the prevention actions, with the *system* rather than a person as the object of repair. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.1 |
| **On-call** | The standing arrangement by which an engineer (or, newly, a responder agent) is responsible for responding to production events in a rotation. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §4.3–§4.4 |
| **Engagement model** | How SRE relates to a product team — consulting, embedded, or operating the service itself. | [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.4 |
| **AIOps** | The analyst-and-vendor category for applying AI to IT operations — telemetry, detection, diagnosis, remediation. Named here only to demarcate it. | [AI for IT](ai_llm/ai_for_it_guide.md) §1 |
| **Agent** | A software actor that takes actions through tools, holds state across steps, and is delegated authority. | [AgentOps](ai_llm/agentops_guide.md) §1 |
| **Eval** | A test that scores a model or agent output against a reference or a rubric; a *sampled* measurement over a population, not a count over an event stream. | [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md) |
| **Drift** | A change over time in the relationship between inputs, the model, and the world, such that behaviour that was correct becomes incorrect without any code change. | [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) |
| **Guardrail** | A constraint that stops an inadmissible output or action before it occurs; a preventive control, and therefore itself a possible cause of blocked valid work. | [AgentOps](ai_llm/agentops_guide.md) §6 |
| **Non-determinism** | The property that the same input may produce different outputs on different runs — the property that makes a model-backed indicator an estimate rather than a count. | [AgentOps](ai_llm/agentops_guide.md) §3 |

### 1.3 The thesis, in one sentence

**AI does not change what reliability is; it changes what can be measured, who is on call, and what counts as a control.**

The guide is built to defend that sentence and to test it honestly wherever the evidence complicates it. The three clauses are each argued in turn (§3 and §4 for measurement, §5 and §6 for on-call, §7 through §11 for controls), and where a clause does not hold — where a quality dimension is genuinely too expensive to measure continuously, where on-call load rises before it falls, where a claim cannot be supported at a primary source — the guide says so rather than smoothing it over.

### 1.4 The seam declared by name

Six siblings own parts of this subject. The table states, for each, what it owns in one clause and what this guide does instead. The row is the boundary; the *elsewhere* column is the guide's whole method.

| Owner (sibling guide) | What it owns, in one clause | What this guide does instead |
|---|---|---|
| [Operational Support Frameworks](operational_support_frameworks_guide.md) §6 | The SRE canon itself: what SRE adds, the toil definition and its attributes, the error-budget machinery and the launch-freeze policy, the overlay table, ITIL, and the on-call rotation and escalation matrix (§4.3–§4.4). | Points at it for every one of those objects; re-derives none of them (§2). |
| [AI for IT](ai_llm/ai_for_it_guide.md) | AI *in* IT operations: the AIOps category, telemetry AI, anomaly detection and alert-noise reduction, root-cause analysis, ITSM AI, runbook automation, auto-remediation under guarded autonomy, and the regulated-estate control set (§11). | Points at it wherever a mechanism is needed (§6, §12); argues only the *consequences for the canon*, never the mechanism. |
| [AgentOps](ai_llm/agentops_guide.md) | Operations *of* AI agents in production: agent observability, the agent SLI set (§3.5), production evals, agent alerting, and the honest flag that agent on-call is consolidating but not codified (§5.4). | Cites the §5.4 flag and holds the same line; does not restate the agent SLI set (§3.7, §6). |
| [Zero-Downtime System Design](zero_downtime_system_design_guide.md) | The SLI/SLO/error-budget derivation, "100 is the wrong target", the availability arithmetic, the deployment strategies, and game days. | Points at it for the derivation; argues only what changes when the indicator is a sample (§3, §9). |
| [Chaos Engineering](chaos_engineering_guide.md) | Failure injection, steady-state hypotheses, the experiment design, the safety guardrails. | Points at it for canarying and injection design (§9); never re-derives an experiment method. |
| [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) | The ML lifecycle, the drift taxonomy, retraining loops, and model deployment. | Points at it for the drift taxonomy (§8); treats drift only as a *threshold* question, not a taxonomy. |

The sentence that follows from the table and organises the whole guide: **this guide owns the reconciliation and nothing else.** Where a subject belongs to a sibling, the guide writes one clause and a link. If more than a third of this guide were restatement, it would have failed no matter how well written, and §15 audits the count honestly.

### 1.5 What this guide does not do

It does not re-explain the canon, and it does not re-explain AIOps. It does not print a metric figure, a benchmark, a market size or a vendor comparison. It does not claim a labour-market, salary or headcount outcome, and it does not claim that any part of AI-era reliability practice is "codified" where the sibling guides honestly say it is consolidating. It does not describe a real institution; Cymbal Bank is fictional and is the only bank in the guide. And it does not pretend the canon's objects are obsolete: the guide's position throughout is that the objects are *re-grounded*, not discarded.

### 1.6 How to read it

Read §2 first if the canon is unfamiliar — it is one page, fully cross-referenced, and it establishes that nothing in it is re-derived here. Then read the three "what AI changes" sections that match your stake: measurement (§3–§4), on-call (§5–§6), controls and postmortem (§7–§11). §9 is the guide's central argument and the one section a sceptical engineer should read in full. §13 is where one decision is worked end to end, and §15 is where every claim is classified, including what this guide deliberately refused to print.

---

## 2. The Canon in One Page, Cross-Referenced

The canon is the discipline's own literature, and this guide treats it as the thing being read forward rather than the thing being introduced. What follows is one paragraph per core object, each ending with the pointer to the guide that owns it. **None of this is re-derived here.**

**The SLI and the SLO.** An SLI is the thing you measure; an SLO is the target you set for it. The derivation — why an indicator is defined on the user's experience, why the target is a band rather than a point, and why "100 is the wrong target" — is owned by [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3, and this guide does not repeat it. What this guide adds is the question the derivation does not confront: what an indicator *is* when the underlying behaviour is non-deterministic (§3.1, §9.2).

**The error budget.** The budget is the tolerable shortfall from perfection, and its machinery — the launch freeze, the postmortem trigger, the arithmetic that turns a target into a tolerable error count — is owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.3, which already quotes the Workbook's example policy sentence for sentence. This guide does not reprint any of that. What it adds is the observation that the budget's accounting assumes errors are *events* and that the service is *yours* — both assumptions that a model-backed service bends (§4).

**Toil.** Toil is operational work defined by a set of attributes — the six attributes and the operational-work cap are quoted from the source and owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.2, and are not restated here. What this guide adds is the honest half: which toil AI can absorb, and what new toil it creates (§5).

**The postmortem.** The blameless postmortem — its culture, its template, its role in prevention — is the canon at its most humane, and [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.1 owns it. The primary source's exact definition of blamelessness is quoted in §7 of this guide only because this guide needs it to argue a new problem, not to introduce the practice. What this guide adds is what blamelessness means when the actor was a model (§7.2).

**On-call.** The rotation, the escalation matrix, the paging discipline and the engineering-work-as-scaling principle are owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §4.3–§4.4, which derives from the SRE book's on-call chapter. What this guide adds is the agent as a participant in the rotation — as assisted responder, and as a responder whose accountability cannot attach (§6).

**Why the canon is the frame, not the legacy.** A reader might ask whether a discipline built for deterministic services should be rebuilt for a non-deterministic world. The answer this guide gives is no, and the reason is that the canon's objects are *measurement and accountability* objects rather than *software* objects. "What did we promise, how do we know, and what do we do when the measurement says we missed" is a question that does not assume the measured system is deterministic — it assumes only that the promise is observable. The canon's *derivations* (how an indicator becomes a budget, why perfection is the wrong target) are reusable as argued elsewhere; its *apparatus* (the objective, the budget, the postmortem, the roster) is the apparatus of any service that makes a promise, deterministic or not. What a non-deterministic component changes is how hard it is to answer "how do we know" — which is the whole of §3 and §9 — not whether the question is the right one. That is why this guide adapts the canon rather than replacing it, and why the six owners above are owners of *components* rather than of a superseded paradigm.

**The canon's objects, and what each becomes.** For the reader who wants the guide's whole argument in one glance before reading it in full, the transformation can be stated object by object. The table is a map, not a summary — each cell is argued in the section named:

| Canon object | What it becomes for a model-backed service | Argued in |
|---|---|---|
| The indicator | A sample with a declared uncertainty, or an exact event count, but never a proxy dressed as either | §3 |
| The target | A band whose width is set by the sampling error, not by convention | §3.3, §9.2 |
| The ceiling | Discovered by evaluation, not assumed to be perfection | §4.4 |
| The budget | A second ledger for quality, because silent drift spends a budget with no entries | §4.3 |
| The freeze | A lever that reaches your releases *and* the provider's changes, or it reaches half the change | §4.2 |
| Toil | The same attributes applied to a new task set, with a net position that may be neutral | §5 |
| The roster | A chain ending at an accountable human, with an agent anywhere but the end | §6 |
| The postmortem | A review of the decision set around the model, since the model cannot be blamed | §7.2 |
| The control set | The reliability artefacts themselves, extended to reach the model | §11 |

**The seam, restated in one line.** Everything the canon says about *how* to run a service to a target is elsewhere; everything this guide says is about *what those objects become* when the service is model-backed. That division is the reason the guide can be short per section and still complete as an argument.

---
## 3. What AI Changes About Measurement

The canon's first object is the indicator, and the AI-era problem is not that indicators are new — it is that the *kind* of number an indicator can be changes when the measured system is non-deterministic. This section argues the change from the ground up and refuses to import any figure from elsewhere. Where it needs the canon's derivation, it points at [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3 rather than restating it.

### 3.1 The non-determinism problem for an SLI

A deterministic service has a property that SRE quietly depends on: the same input produces the same observable behaviour, so a request either succeeded or failed and the indicator is a **count of events**. Availability is then a ratio of counted successes to counted attempts, over a window. The ratio is exact because the events are exact. This is what makes the request-success-rate framing usable, and it is what the canon's own risk chapter takes for granted when it says availability is "calculated based on the proportion of system uptime" or, more usefully, as a request success rate over all requests.

A model-backed service breaks the assumption at its root. The same input may produce different outputs on different runs (the property called non-determinism in §1.2), and *whether an output is a success* is itself a judgement rather than a fact the request carries. A response can be fluent, on-topic, format-valid, and wrong. There is no exception thrown, no status code set, no failed event to count. The very unit that made the ratio exact — the discrete, adjudicable event — has to be rebuilt as a *score over a sample*. That reconstruction is the whole measurement problem of the AI era, and everything in this section is a consequence of it.

A useful way to hold the difference: a deterministic indicator is *reported*, and a sampled indicator is *estimated*. Reporting is transcription — the events happened, count them — while estimation is inference — events were sampled, so infer the population's rate. The canon's whole instrumentation habit was built for transcription, and the AI-era skill is knowing which of the two a given number is.

### 3.2 An eval score is not an availability figure

The temptation is to reach for the nearest number: an eval score. Reduce the quality rubric to a score, run it over a test set, publish the number, and declare it the SLI. This is the single most common measurement error in the field, and it fails on a definitional point before it fails on any statistical one.

**An eval scores a sampled population; an availability figure counts an event stream.** The eval's denominator is a fixed or curated set of cases someone chose; the availability figure's denominator is the population of requests real users actually made in the window. Those two populations are almost never the same population, and they can diverge arbitrarily: the eval can be green while the live traffic contains a class the eval never sampled, and it can be red while the live traffic is fine because the eval set over-weights an edge case. An eval score answers "how does the model do on the cases we thought to test?"; an availability figure answers "what fraction of the real work the service was asked to do did it do acceptably?" Only the second is an SLI in the canon's sense, and the first is an input to the second — a design-time and release-time instrument, not the production indicator. (How to build and govern the eval set is owned by [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md); this guide only insists on the role split.)

The role split has a corollary that teams find counter-intuitive: **improving the eval and improving the service are different activities with different success measures.** A team can push the eval score up by tuning to the eval set while the live service gets no better — the classic overfitting-to-the-instrument failure — or can improve the live service in a way the eval never registers because the eval never sampled that class. Keeping the two instruments distinct, and reporting them separately, is what prevents the eval from quietly becoming the target and the target from quietly becoming the eval.

The practical consequence is that a model-backed service usually needs *two* numbers, not one: a **sampled quality indicator** on live or shadowed traffic, and an **event-count indicator** for the mechanical properties (availability of the endpoint, tool-call failure rate, timeout rate) that remain countable. The canon's discipline applies to both; the difference is that the second is exact and the first must carry its sampling error.

There is a taxonomy worth naming, then: an **input** indicator (what the model consumed), a **behavioural** indicator (what it emitted, checkable by construction — format, schema, refusal), and an **outcome** indicator (whether the user's task succeeded). Only the third is a reliability indicator in the canon's sense; the first two are diagnostic. A service that reports only input and behavioural indicators is instrumented for debugging, not for reliability, and its green dashboard is a debugging dashboard (§8.6).

### 3.3 A success rate is a sampled statistic, and carries a confidence interval

When the quality indicator is computed from a sample rather than the whole population, the number that comes out is an **estimate**, and an estimate has a distribution. This is not a modelling nicety; it is the arithmetic of sampling, and it has a blunt operational consequence.

Suppose a service's quality SLI is computed by scoring a sample of live interactions against a rubric. The reported value is not the true rate; it is the true rate plus or minus a sampling error that shrinks as the sample grows and widens as the rubric's judgements disagree. Now place the SLO threshold next to that estimate. Two cases exhaust the possibilities:

- If the distance from the observed value to the threshold is **larger** than the sampling error, the comparison is meaningful — the service is clearly above or clearly below, and the threshold does work.
- If the distance is **smaller** than the sampling error, the comparison is noise. The service will cross the threshold up and down on re-sampling alone, with no change in the real quality of service. A threshold that sits inside the sampling error is not a threshold. It is a coin.

This is the sharpest single statement this guide makes about measurement, and it should be read as a design rule: **before you can set an SLO on a sampled statistic, you must know the statistic's sampling error, and the threshold must sit far enough from the plausible operating range that sampling noise cannot move the verdict.** Setting a threshold without knowing the error is not setting a target; it is naming a number. [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3 derives why a target is a band — that reasoning still holds — but the band's *width* for a sampled indicator is a function of the sampling error, and the canon's deterministic derivation does not supply that width.

### 3.4 The eval set drifts, so the indicator's definition moves under the measurement

There is a second-order problem, and it is easy to miss because it looks like model drift. The eval set that defines "quality" is itself a human artefact anchored in a world. Prompts change, user behaviour changes, the product changes, the incoming distribution shifts. As the live world moves away from the world the eval set was built in, the eval measures a **narrower and narrower slice** of the present. The score does not fall because the model got worse; it falls — or rises — because the *measuring instrument* is answering a question about the past.

This is **measurement-system drift**, and it is distinct from model drift. Model drift is the world moving away from the model ([MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) owns the taxonomy). Measurement-system drift is the *indicator's own definition* moving away from the world. The two are confounded in a single falling number, and separating them is a new diagnostic task the canon never had to name: a model-backed service must periodically ask not only "is the model still good?" but "does this eval still measure the thing the user cares about, and when was that last confirmed?" A rotating or re-validated eval set is not housekeeping; it is what keeps the SLI's *definition* honest, and an SLI with a stale, un-validated definition is a measurement of a service that may no longer exist.

Note the asymmetry that makes this dangerous. Model drift usually shows up as a *falling* quality number, which attracts attention; measurement-system drift can show up as a *rising* one, because the instrument has narrowed onto the cases the system already handles well. An improving indicator is therefore not, by itself, evidence of an improving service — it may be evidence of a shrinking instrument — and a team that celebrates the rise without asking when the instrument was last validated has confused the two.

### 3.5 The unit of the indicator: per-request, per-session, per-task

A deterministic indicator has an obvious denominator: the request. But a model-backed interaction is frequently not a single request, and choosing the unit changes the number.

- **Per request.** Cheapest to instrument, closest to the canon's request-success framing. But a request in a multi-step interaction can succeed locally while the interaction fails globally — each step was fine and the task still did not get done.
- **Per session.** Groups the interaction, which is what a user actually experiences. Harder to define (when does a session end?) and harder to score, because the session must be adjudicated as a whole.
- **Per task.** The cleanest from the user's point of view — did the job get done? — and the most expensive to measure, because task success often requires knowing the outcome in the world (was the transfer the user wanted actually made correctly?), not merely the text the model emitted.

The guidance is not "pick per-task always" — that would be unaffordable for many services. It is that the unit is a **deliberate choice with a stated cost**, and that a quality indicator silently computed per-request while the user experiences the service per-task is measuring the wrong thing. The canon's own caution that not all requests are equal, and that failing a sign-up is different from failing a background poll, is the deterministic version of this point; the probabilistic version is that the *unit itself* is now a design decision rather than a given.

The difference between a deterministic and a sampled indicator is worth tabulating, because most AI-era measurement errors are misplacements of one into the other's slot:

| Property | Deterministic SLI | Sampled quality SLI |
|---|---|---|
| Unit | One event | One scored case |
| Denominator | All real events in the window | The sampled population |
| Nature of the number | An exact ratio | An estimate with a sampling error |
| Sensitivity to traffic mix | Direct — it *is* the traffic | Indirect — via the sample's coverage |
| How its failure appears | Loudly (an error event) | Silently (nothing to count; §4.3) |
| Threshold setting | Compare to a stable rate | Compare only if the margin exceeds the error (§3.3) |
| What must be versioned | The event schema | The eval set, the rubric and the judge (§3.4) |
| Cheapest honest use | The production indicator | A release-time instrument, or a sampled production indicator (§3.2) |

### 3.6 Reference-free evaluation and the imported judge

When there is no gold answer — generative output has no single correct string — teams evaluate without a reference, often by importing a **judge model** to score the output against a rubric. This is a reasonable engineering move and, for measurement purposes, it has a consequence that must be stated plainly: **the judge's own error becomes part of the measurement.** The indicator is now a composition of the measured system's behaviour and the judge's behaviour, and the judge is itself non-deterministic, itself possibly drifting, and itself capable of the failure modes it is meant to detect. A quality SLI built on an imported judge is not wrong, but it is *two* uncertain layers deep, and a guide that pretended otherwise would be lying about the precision of the number. (The methods, and the calibration question, are owned by [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md); this guide's point is only that the judge's error belongs in the indicator's declared uncertainty, not hidden beneath it.)

The practical test a team should apply to a judge-based indicator: can you state the judge's agreement with human adjudication, on your own cases, with a date? If not, the indicator's uncertainty is unknown and the threshold cannot be checked against §9.2's condition (a). A judge is a measuring instrument, and no instrument is trusted without a calibration record.

### 3.7 What a defensible AI-era SLI looks like

Pulling the section together, a defensible indicator for a model-backed service has four properties, and a service missing two or more of them does not yet have an SLI — it has a dashboard.

1. **It is defined on the service's contract with the user.** The indicator names the outcome the user was promised, not an internal proxy that happens to be easy to log. This is the canon's rule, unchanged.
2. **It is computed from events wherever possible.** Every quality claim that can be reduced to a countable event (endpoint availability, tool failure, timeout, format violation, guardrail block) should be, because those numbers are exact. Sampling is reserved for the dimension that genuinely cannot be counted.
3. **Any sampled quality dimension declares its sampling error.** The number is published with its uncertainty and its sample size, so that a reader can tell whether the threshold comparison is meaningful (§3.3).
4. **The instrument is itself versioned and validated.** The eval set, the rubric and the judge are artefacts with versions and review dates, so that a change in the indicator is distinguishable from a change in the service (§3.4, §3.6).

A service with all four can carry an honest SLO on a quality indicator. A service with only the first has a target but no trustworthy measurement, which is the failure mode §12 calls a dashboard that measures nothing.

### 3.8 The honest complication: what this guide cannot support

Two honest limitations close the section, because a guide about measurement that hid its own unmeasurables would defeat itself.

- **Some quality dimensions are too expensive to measure continuously.** Full human adjudication of live quality is, for many services, simply not affordable at the traffic level. The consequence is that the *continuous* indicator is often a cheap proxy (sampled judge scores, guardrail-block rates, downstream-correction rates), while the *definitive* number comes from periodic deep evaluation. The gap between the proxy and the truth is real and must be declared, not smoothed. This guide cannot tell you a proxy is faithful; only your own validation study can.
- **No primary source establishes an accepted method for setting an SLO threshold over a sampled eval statistic.** The canon's threshold derivation is for countable indicators; the sampling-error reasoning in §3.3 is *this guide's own argument*, and the reviewer must satisfy themselves of the premise rather than take a standard off the shelf. This is recorded as a limitation in §16.1, not as a settled practice.

There is a third honest limitation that follows from the first two: **a fully instrumented quality indicator can still be measuring the wrong quality.** §3.7 requires the indicator to be defined on the user's contract, but the contract itself is frequently ambiguous — a customer support interaction can be "resolved" from the queue's point of view and "not resolved" from the customer's, and the rubric will have to pick one. Every quality indicator embeds such a choice, and the choice should be written down as part of the instrument's definition (§3.7 property 4), so that a future disagreement about whether the service is healthy starts from a shared statement of what was being measured rather than from two teams' intuitions.

---

## 4. What AI Changes About the Error Budget

The error budget is the canon's most elegant object: it converts a reliability target into a permit to take risk, and it makes the permit legible to the whole organisation. The AI era bends two of its load-bearing assumptions and adds a silent failure mode the accounting was never built to see. This section argues the bend; it does not re-derive the budget, whose machinery [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.3 owns and already quotes from the Workbook's example policy.

### 4.1 Two assumptions the budget quietly makes

An error budget assumes two things that a deterministic service satisfies without comment:

- **(i) The service is yours.** You control what changes it, so a change is something you did and can sequence, freeze or roll back. The budget's enforcement mechanism — halting changes when the budget is spent — is only meaningful if you are the one making the changes.
- **(ii) A user experiences failure, and failure is an event.** The budget is spent by errors that were *observed* and attributed: a request that failed, a minute of downtime, a batch that errored. The ledger has entries because the failures are events.

A model-backed service strains both. Assumption (i) is strained by a dependency you do not control (§4.2); assumption (ii) is strained by a class of failure that produces no event at all (§4.3). Neither assumption is *false* for every model-backed service — a self-hosted model with a pinned version and a countable endpoint is closer to the deterministic case — but for the common case of a service backed by a hosted model, both need to be re-examined before the budget's arithmetic can be trusted.

A useful test for whether a given model-backed service sits closer to the deterministic case or the far case: ask whether you can *freeze the change* that would alter its behaviour. If the model version is pinned and you control the deployment, the service behaves like any other versioned dependency and the canon's budget applies nearly unchanged. If the model can change under you, the service is in the far case and §4.2 becomes the central design problem. Most real services sit between the two, and naming where each one sits is the first act of budgeting it honestly.

The two assumptions can be laid out as a ledger of what the budget presumes and what a model-backed service supplies. Where the right column reads "no", the budget does not fail — it simply requires an extension, which is what the rest of this section supplies:

| The budget assumes… | A deterministic service | A model-backed service |
|---|---|---|
| The service is yours to change | True — you deploy | Only partly — the provider can change it (§4.2) |
| The freeze lever reaches the change | True | False for a provider-side change (§4.2) |
| Failures are events | True | False for silent quality drift (§4.3) |
| The reference point is attainable | A ceiling reached by engineering | A ceiling estimated by evaluation (§4.4) |
| The policy expires on a calendar | Review on a date | Review on a date *and* on model change (§4.5) |

### 4.2 Provider-side model change: a data change you neither made nor were told about

The Workbook's example error-budget policy is, in its own words, scoped to *releases*, and releases carry a duration and a plan; the freeze covers releases because a release is a thing you can hold. This guide's first new observation about the canon is that the canon already recognises **data changes** as releases: the policy's freeze sentence covers "all changes and releases", with releases scoped to "(including data changes)". So the canon was never blind to change that is not code. It is the *provenance* of the change that has newly escaped control.

A hosted model can change under a service with no release on the service's side at all. The provider updates the model, or the routing changes, or a safety layer is adjusted — and the service's behaviour changes without a single artefact in the service's own change log. Read against the policy's own scope, this is a **data change you neither made nor were told about**: it falls inside the class the canon calls a release, but it arrives with none of a release's companions — no plan, no cadence, no freeze window you set, no rollback you can perform. The policy's machinery assumes that if you want to halt change, you halt your own releases. Here, halting your releases does not halt the change.

The consequence for the budget is structural, and this guide states it as its own argument: **an error budget whose enforcement lever is "freeze my changes" cannot govern a change that is not mine.** For a service on a hosted model the policy needs a second lever — a pinned or versioned model contract, a canary on the provider's change, or a declared expectation about notification — and the honest position is that the availability of that lever is a *provider-dependency question*, not a canon question. Whether any provider's notification practice makes this lever reliable is **not verified this pass** (§16.1); the guide reasons about the structure and refuses to assert a provider's behaviour.

### 4.3 Silent quality drift produces no error, so the ledger has no entry

The second strain is quieter and worse. The budget spends errors that were *observed*. Silent quality drift — the service becoming gradually less correct while every request still succeeds — produces **no failed request to count**. The endpoint is up, the responses are well-formed, the tool calls return, and the indicator-of-record (availability) is pristine. The quality SLI of §3 may register the decay, but only if it is sampled, and only after the drift has moved far enough to clear the sampling error (§3.3).

This exposes a gap the canon's accounting was never built to see: **a burn rate assumes errors are events.** A burn rate is a *rate of error accrual*, computed by dividing accrued bad events by the budget. If the bad outcome produces no event, there is nothing to accrue, and the budget's accounting reports a healthy service that is quietly becoming useless. The failure is not that the budget is wrong; it is that the budget's *input* — the observed error — is absent, and no accounting can spend a budget it cannot measure. This is the deepest connection this guide draws between §3 and §4: **the measurability problem of §3 is the reason the error budget of §4 can fail silently.** Solve the measurement and the budget works; leave it unsolved and the budget's green light is a statement about your instrumentation, not about your service.

The design implication, stated as this guide's own argument, is that a model-backed service needs a **quality ledger alongside the error ledger** — a record of sampled quality over time, with its uncertainty, whose deterioration is treated as budget consumption even when no request failed. Where the canon's ledger records counted errors, the quality ledger records estimated quality, and the budget policy must say what deterioration in the second triggers the same response the canon triggers for the first.

This is also why the drift-or-regression triage (§5.3) is a reliability task rather than a modelling curiosity: the quality ledger's deterioration is a symptom with at least three possible causes, and the budget's response — a review, a freeze, a retraining decision — depends on which cause it is. A budget that fires on the symptom without the diagnosis will freeze the wrong thing.

### 4.4 What "100" means for a model-backed service

The canon already contains the argument that perfection is the wrong target, and this guide leans on it rather than re-making it. The SRE book's *Embracing Risk* (ch.3) puts it exactly: rather than maximising uptime, SRE "seeks to balance the risk of unavailability with the goals of rapid innovation and efficient service operations"; and users "typically don't notice the difference between high reliability and extreme reliability in a service, because the user experience is dominated by less reliable components like the cellular network or the device they are working with" — the book's own illustration being that "a user on a 99% reliable smartphone cannot tell the difference between 99.99% and 99.999% service reliability". That passage is the anchor this guide uses, quoted from the 2016 edition (an early version appeared in *;login:* August 2015).

The extension to a model-backed service is this guide's argument: **the budget must be set against an achievable ceiling estimated by evaluation, not against 100.** A model-backed service has a quality ceiling that is a property of the model, the task and the guardrails together, and that ceiling is often well below the intuitive ideal — some fraction of hard cases will be answered imperfectly no matter how much engineering is applied, and pushing for more reliability than the ceiling permits is exactly the "extreme reliability is worse for the service" error the book names. Setting the SLO against 100 would therefore set a target the service cannot meet for reasons that have nothing to do with operational excellence, guaranteeing a permanent breach and destroying the budget's usefulness as a signal. The correct reference point is the *measured achievable ceiling*: run the evaluation, find the best the current system does on the task, and set the target below that with margin — the same "band, not a point" discipline the canon applies to deterministic services, applied to a ceiling the evaluation has to discover.

A caution about the ceiling: it is not a single number but a function of the task mix. The same model scores differently on an easy mix and a hard one, so a ceiling estimated on today's distribution ages with that distribution. This is condition (b) of §9.2 reappearing at the level of the target itself, and it is why the ceiling should be re-estimated whenever the population shifts, not fixed once.

### 4.5 Revisit on model change, not only on service change

The final observation about the canon is one the canon hands over almost by accident: its own example policy is a **reviewable artefact with an expiry**. The Workbook's Example Error Budget Policy (Appendix B, authored by Steven Thurgood) carries a document metadata block with a *Date* of 2018-02-19, an *Approval Date* of 2018-02-20, and a **Revisit Date** of 2019-02-01. The lesson is not the specific dates; it is that the example policy contemplates its own review on a schedule. An error-budget policy is a decision that ages, and the example says so by carrying a date on which it should be looked at again.

For a model-backed service this guide extends the trigger: **the policy must be revisited on model change, not only on service change.** If the achievable ceiling of §4.4 moves when the model moves — and it does — then a policy calibrated to the old ceiling is calibrated to a service that no longer exists, and its thresholds, freezes and quality-ledger conditions all need re-justification against the new reality. The revisit date becomes a *change-triggered* obligation as well as a calendar one: any provider-side model change is, by §4.2, a change to the service, and therefore a legitimate occasion to re-open the policy. The canon's own artefact already encodes the principle; the AI era simply multiplies the occasions on which it fires.

The reverse also holds: a policy revisited only on a calendar can drift stale, because a model-backed service's behaviour can move without any event on the calendar. The two triggers — scheduled and change-driven — are not alternatives; a policy that carries only one of them will either age quietly or be rediscovered only after an incident.

---
## 5. What AI Changes About Toil

Toil is where the AI era's promise is loudest and its honesty is thinnest. The claim is that AI absorbs operational work, which would be a direct attack on the canon's central cost — the operational load that scales with the service. This section asks which part of the claim is true, which part is wishful, and what new toil the answer creates. It does not restate the definition of toil; the six attributes, the operational-work cap and the support-desk mapping are owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.2, quoted there from the source, and are not re-listed here.

### 5.1 A pointer, not a re-reading

The canon's definition of toil runs on a set of attributes and a cap on how much of an engineer's time operational work may consume. The attributes are the test; the cap is the discipline that keeps a team engineering rather than merely operating. Both are owned by the sibling guide, and this guide uses them as a *given* — a yardstick to be applied to a new class of work rather than a definition to be explained again. The method of this section is therefore simple: take the canon's own test and point it at the tasks AI adds, and report what the test says, including where the answer is unfavourable to the optimistic case.

One framing point before the test is applied. The canon weighs toil because it competes with engineering for the same person-hours; AI changes not the competition but the *composition* of the work. The question this section answers is therefore not "is there less work?" but "is the work that remains the kind that earns a person's attention?" — which is the canon's own criterion, and the one the optimistic case usually skips.

### 5.2 Which toil AI can genuinely absorb

The canon's attributes describe work that is repetitive, largely manual, automatable, and lacking enduring value. Held against that test, a real slice of operational work falls to AI-assisted handling, and the slice is identifiable rather than vague:

- **Triage against a known-failure library.** When an alert matches a failure the team has seen, triaged and written down, the first several minutes of human response are largely mechanical — pull the context, match the signature, retrieve the relevant runbook. That is repetitive, automatable work with no enduring value in the act itself. An agent that drafts the timeline, proposes the known hypothesis and surfaces the runbook is absorbing genuine toil (§6.1 develops the boundary between drafting and acting).
- **Change and version bookkeeping.** Correlating a deploy, a version bump or a provider change against the incidents that follow is repetitive correlation work — the kind of thing the canon would call out as toil and expect to be automated.
- **Evidence assembly.** Gathering the log excerpt, the relevant trace and the metric snapshot for an incident record is mechanical collection, well suited to automation, and directly feeds the postmortem of §7.

These are honest wins. They reduce the *volume* of manual steps, and the canon's attribute test endorses automating them.

The list is deliberately conservative: each item is work that a *known* situation generates and that a decision procedure can therefore handle. The canon's attribute test does not endorse automating judgement about a *novel* failure, because that judgement is neither repetitive nor devoid of enduring value — it is exactly the human contribution that makes the team worth having. The discipline AI-era teams most often lose is this boundary: automating the known, and then trusting the automation on the unknown, where the traits that made automation safe (repetition, a known answer) are absent.

### 5.3 The new toil AI creates

The optimistic case counts only the wins and omits the work on the other side of the ledger. Applying the canon's attributes honestly to the AI-era operating model reveals a set of tasks that did not exist before and that fit the attributes of toil at least as well as the work they replace:

- **Prompt and configuration maintenance.** Recurring, manual, and repetitive — edited to accommodate a model change, a new guardrail, a new edge case — with little enduring value in the edit itself, since the prompt is a moving part rather than a permanent artefact.
- **Eval-set curation and labelling.** Building, maintaining and re-validating the eval set of §3.4, including relabelling as the world moves. Labour-intensive, recurring whenever the measuring instrument ages out.
- **Guardrail-exception review.** Every guardrail that blocks a request generates a queue: was this block correct, or did it block valid work? Reviewing that queue is repetitive work created directly by the guards meant to reduce incidents. (§8.3 argues why the blocked-valid-request case is itself a reliability event.)
- **Model-version change review.** Each provider-side change (§4.2) creates a review obligation — re-run the evals, re-check the thresholds, re-justify the policy — and it recurs on the provider's cadence, not yours.
- **The new triage question: drift or regression?** This is genuinely new work. When a model-backed service degrades, the first diagnostic question is whether the model drifted, the eval drifted (§3.4), or the code regressed — three different owners, three different fixes. That question did not exist when the same input always produced the same output.

Each of these is repetitive, manual and largely automatable, and each has low enduring value in the doing. By the canon's own test, they are toil — **new** toil, created by the AI operating model as a side effect of the work it absorbs.

Laid out against the canon's attributes, the two columns — work absorbed and work created — make the net position inspectable rather than rhetorical:

| Task | Repetitive? | Automatable? | Enduring value? | Verdict |
|---|---|---|---|---|
| Triage against a known-failure library | Yes | Largely | Low in the act | Toil AI absorbs (§5.2) |
| Change-and-version correlation | Yes | Yes | Low | Toil AI absorbs (§5.2) |
| Evidence assembly | Yes | Yes | Low in the act | Toil AI absorbs (§5.2) |
| Prompt maintenance | Yes | Partly | Often none | New toil (§5.3–§5.4) |
| Eval-set curation and labelling | Yes | Partly | Some — in the artefact | New toil (§5.3) |
| Guardrail-exception review | Yes | Partly | Adds a safety record | New toil (§5.3) |
| Model-version change review | Recurs on the provider's cadence | Partly | Necessary, not permanent | New toil (§5.3) |
| Drift-or-regression triage | At each degradation | Hard to automate | The diagnosis *is* the value | New work (§5.3) |

### 5.4 Applying the attributes to prompt maintenance, deliberately

Prompt maintenance deserves the deliberate confrontation, because it is the task the industry most often miscategorises as permanent engineering rather than toil. Point the canon's test at it and refuse to soften the answer:

- **Is it manual and repetitive?** Yes — prompts are re-edited repeatedly in response to model changes and edge cases, often by hand.
- **Is it automatable?** Partly. Templating and systematic prompt management help, but the *judgement* of what to change in response to a model's new behaviour is not fully automatable today, which means it remains partly manual even after tooling.
- **Does it have enduring value?** This is the crux, and the honest answer is often *no* — a prompt edit that accommodates a provider's change rarely leaves behind a permanent improvement to the system; it restores equilibrium and is then itself subject to the next change. The work is real and continuous, and it is largely maintenance rather than construction.
- **Does it scale as the service grows?** Prompts do not scale linearly with request volume the way, say, ticket-triage does — which is the one point in toil's favour — but the *number of prompts and variants* tends to grow with the product surface, so the maintenance burden grows with the estate even if not with traffic.

The honest verdict: prompt maintenance is, in significant part, toil under the canon's own definition, and a team that never counts it is under-reporting its operational load and over-reporting its engineering capacity. That verdict is unpleasant for the AI-native narrative, which is exactly why the canon's test is worth applying rather than skipping. (Whether this toil should be permanently staffed or automated away is a team-design question the management guides own; this guide states the accounting, not the organisational remedy.)

### 5.5 The structural point: the cost curve changes, the shape does not

The section closes on the claim this guide is willing to defend. AI-assisted operations change the **cost curve** of toil — the amount of human minutes per unit of operational work falls, and the class of work that reaches a human shrinks toward the genuinely novel. But they do not change its **shape**: toil remains work that is manual, repetitive, automatable in part, and regenerative. Absorb one class of toil and the system presents the next — the guardrail-exception queue, the model-change review, the eval re-validation. The canon's insight was never that toil could be eliminated; it was that a team must *cap and measure* it, and that insight is more, not less, important when the work has fragmented into many small recurring tasks. AI moves the frontier of toil; it does not remove the need to know where the frontier is.

The practical discipline that follows is the canon's: measure the operational load, including the new toil, against a real denominator, and cap it. A team that does not count the eval maintenance, the guardrail queue and the model-change review will believe its toil has fallen when it has merely fragmented into tasks too small to name — and unnamed toil is toil that will not be capped.

---

## 6. What AI Changes About On-Call

On-call is where the canon's cost is felt most directly, and where the AI era's changes are simultaneously the most attractive and the most dangerous. Two distinct pictures must be kept apart: an agent that *assists* a responding human and an agent that *is* the responder. Conflating them produces the field's most common error, which is to speak of an "AI SRE" that does both. The rotation, the escalation matrix and the paging discipline are owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §4.3–§4.4; this section argues only what changes when an agent enters the picture.

### 6.1 The agent-assisted responder versus the agent as responder

The **agent-assisted responder** drafts and retrieves while a human decides. Faced with an alert, the agent assembles the timeline, proposes a hypothesis from the known-failure library, surfaces the relevant runbook, and prepares the incident record — the absorbing-toil role of §5.2. The human still decides, still acts, and still owns the outcome. This role is where the value is clearest and the accountability is intact, and it is the role most near-term operating models can actually adopt without re-opening the accountability question.

The **agent as responder** acts. It receives the alert, forms a diagnosis, executes a remediation — potentially of the kind [AI for IT](ai_llm/ai_for_it_guide.md) §8 describes as guarded autonomy — and reports back. The value proposition is obvious: it never sleeps, never fatigues, and responds in the time it takes to run the checklist. The accountability proposition is the problem. **An agent cannot be accountable.** The canon's on-call exists because a service needs an owner who is answerable — who carries the pager, who is woken, who owns the consequence. A responder that cannot be answerable cannot be the *last* escalation point; it can only be a step within a chain that terminates at a human who is. The design question is therefore not "can the agent respond?" but "at what point in the chain does a human become accountable, and how is that point enforced?" — a question the canon answers for purely human rotations and that an automated step re-opens.

The clean way to hold the distinction is a two-question test applied to any proposed agent role: *does a human still make the decision?* and *does a human still own the consequence?* The agent-assisted responder answers yes to both and can be adopted at once. The agent-as-responder answers no to both until a human is inserted back into the chain, at which point it becomes assisted again — differently shaped, but assisted. This is why the honest framing is not "autonomous responder" versus "human responder" but a *chain* whose terminal node must be accountable, and the engineering question is where along that chain the terminal node sits.

The roles can be laid out as a chain, and the table's last column is the one that decides what a team may adopt today:

| Role | Decides? | Acts? | Accountable? | Status |
|---|---|---|---|---|
| Agent-assisted responder | Human | Human | Human | Adoptable now (§6.1) |
| Agent acts on a human's approval | Human | Agent, on approval | Human | Piloted (§6.4) |
| Agent as responder (guarded autonomy) | Agent proposes | Agent | Must terminate at a human | Not codified (§6.4) |

### 6.2 Escalation when the first responder is automated

If the first responder is an agent, the escalation topology changes, and the canon's escalation matrix — who pages whom, when, and why — acquires a new node and a new failure mode.

The new node is the **hand-off to a human**, and its design must state three things the canon leaves implicit for human rotations: *when* the agent must escalate (what evidence or confidence condition compels a human), *how* the human receives the agent's state (the timeline, the hypothesis, what was already tried), and *who owns the outcome* once the hand-off occurs (unambiguously the human). The canon's rule that a person's pager carries an owner is preserved, but the route to that person now runs through a system.

The new failure mode is subtle and worth naming: **the agent can page a human with a confident, wrong diagnosis**, and the human must refute it under pressure. A human paged at night with "I believe this is the database connection pool, here is my reasoning" is now anchored on that hypothesis, and disproving a plausible-but-wrong lead costs more attention than diagnosing from scratch would have. This is a *new cost* — not a reduction in on-call load, but a shift in its character, from "what is happening?" to "why is the machine's answer wrong?" An organisation that adopts an agent responder and counts only the tickets it resolves will miss this cost entirely.

There is a second new cost beyond the confident-wrong-diagnosis problem: **the hand-off may transmit the agent's framing, not just its facts.** A human receiving a timeline, a hypothesis and a list of already-tried steps inherits the agent's theory of the incident, and if that theory is subtly wrong the human's first job is to notice the frame rather than to analyse the incident. The canon's incident-command disciplines — a shared, maintained timeline, an explicit current hypothesis, a separate scribe — exist precisely to make a team's framing inspectable, and an automated first responder must expose its framing in the same inspectable form, or it hands the human a conclusion dressed as evidence.

### 6.3 On-call load may rise before it falls

The temptation is to promise a monotonic reduction in on-call burden. The honest position is that load may **rise** in the transition and only fall later, if it falls at all. The reasons are structural, not pessimism: the new toil of §5.3 (guardrail-exception queues, model-change reviews) lands *on the on-call shift* as new alert classes; the new hand-off creates new interruption patterns (§6.2); and the new question of §5.3 — drift or regression? — is a diagnostic the responder must now run before the known playbooks apply. An agent that absorbs the old alerts while generating new alert classes can leave a team busier, not less busy, during the adoption period, and a plan that assumes otherwise will under-provision the very shift it is trying to relieve.

There is a governance consequence: an on-call load that rises during adoption is a reason to *slow the rollout*, not a reason to declare success. The canon's engagement models already prescribe that operational load be a first-class constraint on how a service is run; an agent adoption that raises load without acknowledging it is running against that constraint, however much it improves any single metric.

### 6.4 The honest line: not yet a codified practice

This is the point at which the guide must hold a line it cannot cross, and it does so by deferring to the sibling that owns the material. [AgentOps](ai_llm/agentops_guide.md) §5.4 — "On-Call: The Agent On-Call" — honestly flags that **agent on-call and "LLM SRE" are consolidating but not yet a codified practice.** This guide adopts the same position and refuses to overclaim. There is no settled framework for an agent in the rotation, no agreed escalation semantics for an automated first responder, and no codified accountability model for an agent that acts before a human is aware. What exists is a set of emerging patterns and a clear direction of travel, and a guide that presented those patterns as a codified practice would be inventing a standard the field has not written.

The consequence for a practitioner is practical rather than philosophical: an agent in the on-call chain today must be treated as a *pilot with a human-in-the-loop*, instrumented so the human hand-off is observable, and reviewed against the same criterion the canon applies to any operational change — does it reduce the work a person must do without hiding a class of failure?

A useful way to hold the line without overclaiming: treat the agent in the rotation as an *experiment with a stated hypothesis* — that it reduces the human work of a defined alert class without hiding a failure — and require the experiment to be measured against that hypothesis before the agent is granted more scope. That is not a codified practice, but it is the disciplined way to adopt something the field has not yet codified. §6 answers that AI changes *who is on call*; it does not yet answer *how an agent is held on call*, because the field has not answered it either.

---

## 7. What AI Changes About the Postmortem

The postmortem is the canon's memory: the mechanism by which a service converts a failure into a permanent improvement and a blameless record. The AI era improves its raw material and complicates its central concept at the same time. This section argues both, using the primary source's own definition of blamelessness because the argument needs it, not to introduce the practice — the culture and template remain owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.1.

### 7.1 Timeline reconstruction: the trace exists, and the ambition is old

The first change is unambiguously positive. An AI-backed service produces a trace: the prompt, the retrieval, the model call, the tool calls, the outputs. Where the canon's postmortem writer once reconstructed a timeline from logs and memory, the agent-executed service leaves a *machine-readable* record of what the system saw and did, at a granularity no human-drafted timeline could match. Reconstruction is materially easier where the instrumentation is present.

It is worth placing this against the canon rather than treating it as new, because the primary source shows the ambition is old. The SRE book's *Postmortem Culture* (ch.15, 2016) lists, as future work in the postmortem-tooling domain, **"machine learning to help predict our weaknesses, facilitate real-time incident investigation, and reduce duplicate incidents."** The 2016 text already wanted machine assistance to investigate incidents in real time; the AI era is realising an ambition a decade old, not inventing a product category. The honest framing is therefore continuity, not novelty: the trace is better instrumentation for an old goal, and any vendor narrative that presents machine-assisted incident investigation as a new idea is contradicted by the canon's own 2016 sentence.

The trace's value is bounded, though, by a property the canon's human timeline always had and a machine trace can lose: *narrative*. A trace records what happened, in order; it does not, by itself, distinguish the signal from the noise, the load-bearing step from the incidental one, or the decision from the accident. Reconstruction from a trace is not less work than reconstruction from memory — it is different work, shifting the effort from assembling facts to *interpreting* an abundance of them. The 2016 ambition to "facilitate real-time incident investigation" is realised as a firehose of machine truth, and the postmortem writer's skill moves from finding the evidence to reading it.

### 7.2 Blamelessness when the actor was a model

The hard question is not the timeline; it is what blamelessness *means* when the actor was a model. The canon's definition is precise. From the SRE book, ch.15: "For a postmortem to be truly blameless, it must focus on identifying the contributing causes of the incident without indicting any individual or team for bad or inappropriate behavior." And, on the object of repair: "You can't 'fix' people, but you can fix systems and processes."

Reason from that definition to the model case, carefully. Blame is a predicate of persons; it attaches to an agent who *could have done otherwise and chose not to*. A model is not such an agent — it does not have intentions, cannot be punished, and cannot be reformed. So **blame cannot attach to a model**, and the definition's instruction to avoid indictment is trivially satisfied and uselessly satisfied: there is no one to indict. The interesting question is not "is the model blameworthy?" (it is not, and that was never the point) but "**what is the object of review now?**"

The canon supplies the answer by analogy to its own principle. The book's rule is that when a "mistake" occurs, prevention works by "investigating the systematic reasons why an individual or team had incomplete or incorrect information." The model had no information and made no choices; but the *system around the model* did have information and did make choices, and those choices are the new object of review. Concretely, the object of review becomes the **decision set around the model**: who set the guardrail threshold, who approved the version, what the control expected when it passed, what information the human reviewer had, and whether the system's design gave the model a task it could not be trusted to perform. This is exactly the canon's method — "focus on the systematic reasons" — applied one layer out, because the proximate actor is not a moral agent and the causal chain therefore runs through the humans who built the envelope.

### 7.3 "The model did it" as the new blame-avoidance

The danger is that this correct observation becomes a new way to avoid the very analysis it enables. Watch for the swap: replacing "the person did it" (which the canon forbids) with "**the model did it**" (which reads as blameless because there is no person to blame) as a way of *ending* the inquiry rather than *redirecting* it. A postmortem that concludes "the model produced an incorrect output and the user acted on it" has named the proximate cause and hidden the human causal chain — the person who chose the model for that task, the person who set the threshold, the person who approved the prompt, the person who decided not to require confirmation. "The model did it" is not a blameless conclusion; it is a blame-*avoidance*, and it is worse than traditional blame because it is harder to see: no individual is named, so the record looks clean while the accountability has evaporated.

The corrective follows directly from the canon's own principle: because a model cannot be blamed, the analysis must move *further into the human system*, not stop at the model. A postmortem for a model-involved incident should be able to point to a person for each consequential decision in the envelope — the same standard the canon applies when it refuses to indict an individual but insists on identifying the contributing causes.

Stated as a table, the move is clearer: the question is not whether the model is blameworthy (it cannot be) but what the review's object becomes once blame is off the table.

| Actor | Can blame attach? | So the object of review becomes… |
|---|---|---|
| A person | Yes — they could have chosen otherwise | The person's decision (the canon forbids indicting them) |
| A model | No — it has no intentions and cannot be reformed | The decision set around it: thresholds, version approvals, expectations |
| An agent that acted | No — the same reason | The humans who scoped, bounded and approved its action |
| "The model did it" | — | Nothing further, which is why it is a blame-avoidance, not a finding |

Where no person can be named for a consequential decision, the finding is not "no one is responsible" but "**the decision was not owned**", which is itself a system defect worth fixing.

One practical safeguard against the swap: require every model-involved postmortem to name a person for each consequential decision in the envelope, or record explicitly that the decision was unowned. A postmortem that can do neither has not finished the analysis, it has stopped it.

### 7.4 The second story: postmortem quality is a logging requirement

The final point is a consequence of §7.1 that is easy to miss and impossible to fix after the fact. If "what the model saw" cannot be reconstructed, then "what the model did and why" cannot be analysed either — and the ability to reconstruct it is decided *before* the incident, by what the system logs. **Postmortem quality is therefore a logging requirement set at design time, not a writing skill applied after.** A service that does not capture the prompt, the retrieved context, the model version and the tool inputs leaves a postmortem that can only say "the model produced a bad output" — the blame-avoidance conclusion of §7.3, arrived at not by evasion but by omission.

The design rule is the mirror of the trace's value: instrument for the postmortem you will be forced to write. The canon's postmortem discipline assumed the evidence existed and that the writer's job was to analyse it; the AI era makes the existence of the evidence a design decision, and the design decision — what to log, at what fidelity, retained for how long — is itself a control (§8.4, §11) rather than an operational afterthought.

There is one more consequence of the blamelessness argument worth stating plainly, because it changes how a team designs incidents rather than how it writes them up. If blame cannot attach to a model and the review must therefore target the decision set, then the **decisions must be reconstructable as decisions** — not merely as events. A trace that records that a guardrail threshold was set to a value does not record *who set it, on what reasoning, and with what expectation*, and those are exactly the objects the postmortem must review. The logging requirement of §7.4 is therefore two requirements: capture what the model saw, and capture the human decisions around it. A service that logs the first and not the second produces a postmortem that can describe the model's behaviour in detail and still have nothing to fix.

---
## 8. SRE for AI Systems - The Other Direction

The guide has so far read AI *into* the canon. This section turns and reads the canon *onto* AI — treating an AI-backed service as an ordinary service with an owner, an indicator and a target, and asking what the canon's ordinary machinery says about it. The point of the reversal is that an AI service is not a special class that escapes the discipline; it is a service whose indicators are harder and whose dependencies are stranger, and the canon applies with its usual demand: name the target, measure the indicator, own the dependency. This is the section where "the other direction" of the title is argued.

### 8.1 An AI service is a service with an owner and a target

The first act of the canon is to establish that something is a *service*: it has a user, it makes a promise, and it has an owner who is answerable for the promise. An AI-backed capability is frequently operated as though it were exempt — a model endpoint, a feature flag, a capability that "just works" — with no named owner, no stated target, and no indicator. That is not an AI problem; it is the pre-canon problem, and the canon's answer applies unchanged: **name the owner, name the promised outcome, and attach a target** (§1.2 for the vocabulary; [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3 for the derivation). The discipline that a service without an owner is unowned and a service without a target is unmeasured is exactly as true for a model-backed capability as for a database. SRE-for-AI begins by declining to grant the capability an exemption.

The test of ownership is operational: can you point to the artefact that says what the service promises, and to the person whose name is against it? If the answer to either is no, the capability is not yet a service in the canon's sense, and no amount of model capability substitutes for the missing ownership — a theme this series' governance guides make in parallel.

### 8.2 Evals as indicators — not a separate science

The second act is to place evaluation where it belongs: as an **indicator family**, not a separate discipline standing beside reliability. The relationship is simpler than the literature often implies. An eval is a measurement of the service against a rubric; an SLI is a measurement of the service against its promise. When the eval is computed on the traffic the service actually serves, and defined on the outcome the user actually cares about, **the eval is an SLI** — and the whole of §3's reasoning about sampling, confidence and instrument validation applies to it. When the eval is computed on a curated test set at release time, it is a release-time instrument, not the production indicator (§3.2). The canon does not need a new science for evals; it needs evals to be *placed* correctly in the indicator taxonomy — release-time instrument versus production indicator — and the confusion between the two is one of §12's failure modes. (The method of building evals is owned by [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md); this section owns only the *placement*.)

### 8.3 Drift as a threshold question

Drift is where the AI-specific literature is richest and where this guide must point rather than re-derive. The taxonomy — data drift, concept drift, the various statistical tests, the retraining triggers — is owned by [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md), and this guide does not restate it. What this guide adds is the *reliability* framing, which the taxonomy does not supply: **drift is a threshold question, not a taxonomy question.** For an SRE, the taxonomy matters only insofar as it answers four questions the canon would ask of any changing quantity:

1. **What changed?** — the model, the data, the eval instrument (§3.4), or the world.
2. **How much?** — the magnitude of the shift, expressed on the same scale as the indicator.
3. **Over what window?** — the observation period over which the change is judged real rather than noise.
4. **Before it consumes budget?** — the point at which the change must be treated as budget consumption (§4.3) and trigger the policy.

Drift that cannot be expressed as a change in the indicator, over a window, with a trigger, cannot be acted on by the reliability process — it is a research observation, not an operational signal. The value of the ML lifecycle guide's taxonomy is that it tells you *what* to watch; the value of the canon is that it tells you *what to do* when what you watch moves — and the two meet at the threshold.

### 8.4 A guardrail as a control — and a possible reliability failure

A guardrail is a preventive control: it stops an inadmissible output or action before it occurs. Read through the canon, a preventive control has a cost structure that AI-native teams routinely ignore. **A guardrail that blocks a valid request is a reliability failure**, not a safety success. It is a service failing to do the work the user was promised, for a reason internal to the service. Every guardrail therefore sits at a trade-off the canon would recognise as a reliability trade-off: too permissive and inadmissible outputs reach users (a quality-safety failure), too restrictive and valid work is blocked (a reliability failure). Both failure modes are measured against the same service promise, and a team that counts only the first is running a service whose real reliability is worse than it reports.

This is the observation that makes the guardrail's *exception queue* a reliability artefact rather than a safety artefact (§5.3). Each blocked request is a candidate reliability event, and the design question is whether the guardrail has a calibrated, measured false-positive rate whose deterioration would be visible in the indicator set. A guardrail with no measured false-positive rate is a control with no error budget of its own — which is to say, a control whose reliability impact is unmeasured and therefore unmanaged. (The design of guardrails is owned by [AgentOps](ai_llm/agentops_guide.md) §6; this section owns only the reliability accounting.)

### 8.5 The model provider as a dependency

This is the section's sharpest point, and it is where the canon's dependency discipline meets a dependency of an unfamiliar kind.

**A provider's uptime is not your SLO.** Your SLO is a promise about *your service's* contract with *your* user. If the provider is down and your service consequently fails, that failure is still yours in the eyes of your user and your target — the canon has never allowed a dependency's weakness to excuse the service's promise. So a model-backed service must decide, honestly, how much of a provider degradation it can absorb (a degraded mode, a cached path, a fallback) before the user sees a failure — and that decision is part of the SLO's meaning, not a footnote.

**You cannot fix their model.** The dependency's core is not merely unreachable when it fails; it is *unmodifiable* when it misbehaves. A database you can patch; a hosted model you cannot. This makes the provider dependency unusual: you own the promise while controlling only the envelope around the dependency, not the dependency itself.

**A provider outage and a provider change are different reliability events.** An outage is loud: the endpoint errors, the requests fail, and the canon's machinery (availability indicator, burn rate) sees it. A change is silent: the endpoint is up, the requests succeed, and the *quality* moves (§4.2, §4.3). The two require different instrumentation and different responses, and an organisation that instruments only for the outage will miss the change.

**A second provider is a reliability decision, not just a procurement one.** Routing to a second model diversifies the outage risk — and imports a *second* drift surface, a second set of behaviours to evaluate, a second envelope to validate, and a new consistency problem (the two providers will not behave identically, so the service's quality becomes a function of which provider answered). The decision is legitimate, but its reliability cost is concrete and must be counted: redundancy against outages purchased with additional drift surface. There is no free diversification.

A practical consequence: the provider dependency should be represented *inside* the service's failure model, not outside it. The canon asks what the service does when a dependency degrades; for a model provider the answers are categorically different from a typical dependency because the degradation can be partial and unobservable (quality, not availability), unfixable (you cannot patch it), and uncaused by you (a change you did not make). Representing the provider's behaviour *as a dependency's behaviour* — with a stated degraded mode, a stated fallback, and a stated signal — is what turns an uncomfortable fact ("we depend on a black box") into an engineered decision.

The contrast that makes the point sharpest is between a dependency a reliability engineer is used to and a model provider:

| Property | A typical dependency (a database, say) | A hosted model |
|---|---|---|
| Failure mode | Loud — errors, timeouts | Loud *and* silent — availability and quality |
| Fixable by you? | Yes — patch, scale, reconfigure | No (§8.5) |
| Change provenance | Your release | Possibly the provider's (§4.2) |
| Can you freeze it? | Yes, by stopping releases | Not by stopping your releases (§4.2) |
| Instrument | Availability indicator | Availability indicator *and* a sampled quality indicator (§3) |
| Redundancy cost | A replica, a failover | A second drift surface (§8.5) |

### 8.6 The green-dashboard-broken-service gap

The section closes on the failure the canon's instrumentation is least equipped to catch, and which the AI era makes routine: **the endpoint can be perfectly available while the service is useless.** The availability dashboard is green — no errors, low latency, healthy endpoints — while the model is returning fluent but wrong answers, or the retrieval is returning stale context, or the guardrail is blocking the common case. Every countable indicator the deterministic canon relies on is healthy, and the service is failing the user. This is the operational face of §4.3's silent drift, and it is why the quality ledger of §4.3 and the sampled indicator of §3 are not optional extras: without a quality indicator, a model-backed service has no instrument capable of showing this failure at all. The green dashboard is not lying; it is answering a question — is the endpoint up? — that is no longer the question that matters, which is the precise sense in which AI changes *what can be measured*.

The lesson generalises beyond availability: whenever the countable instrumentation is healthy and the service is failing, the instrument set is answering a question adjacent to the one that matters. The AI era's characteristic version is quality, but the pattern — a green proxy for a broken outcome — predates AI and will outlast the current model generation; what AI adds is a proxy that can be green while the outcome is wrong in ways only a sampled indicator can see.

---

## 9. The Reliability of a Non-Deterministic Component

This is the guide's central argument, and it is addressed to the engineer who suspects that the whole business is unfalsifiable. The claim to be defended is narrow and testable: a model-backed service can carry a reliability target, but only under conditions that must be stated and, where they fail, named. The section does not re-derive canarying ([Zero-Downtime System Design](zero_downtime_system_design_guide.md) §3, [Chaos Engineering](chaos_engineering_guide.md)) and does not re-derive variance-reduction mechanics ([Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md), cross-referenced).

### 9.1 Variance and what it does to a threshold

A non-deterministic component has a variance: run it twice and you get different outputs. Place a threshold near its typical output and the variance alone will cross the threshold on repeated observations, so the "breach" is caused by the component's inherent spread rather than by any change in the world. The canon's deterministic threshold assumed a stable quantity whose variation was signal; a model-backed threshold has to separate the variation that is *noise inherent to the component* from the variation that is *signal about the service*. This is the same sampling-error point of §3.3, expressed as a property of the component, and it is the reason a threshold that "works" in a demo — where one run looked fine — can be useless in production, where the tenth run looks different for no reason related to the user.

A concrete way to feel the problem: a deterministic threshold set at a value never seen in normal operation is safe against noise by construction; a model-backed threshold is frequently set near the *middle* of the observed range, where the density of outcomes is highest and the noise is therefore most likely to cross it. The instinct that pulls thresholds toward the middle of what a service "usually does" is exactly the instinct that makes them unusable for a non-deterministic component.

### 9.2 The central argument: an SLO is a promise about a distribution

State the canon's object precisely. **An SLO is a promise about a distribution, not about any individual outcome.** No reliability target promises that this specific request will succeed; it promises that the *fraction* of requests that succeed will stay within a band. This is true even for deterministic services — a target is a rate, not a guarantee — and it is the same shape of promise for a model-backed service. The difference is in *how the rate is estimated*.

For a deterministic service the indicator is a **deterministic function of events**: count the attempts, count the successes, divide, and the resulting rate is exact. For a model-backed service the indicator is an **estimate from a sample**: score a sample of outputs, and the resulting rate is the true rate plus or minus a sampling error. Both are estimates of a distributional property, but only one is exact. This is the whole of the argument, and its consequence is that a statistical threshold can carry an SLO only if three conditions hold:

**(a) The sampling error is smaller than the distance from the observed value to the threshold.** If it is not, the comparison is noise (§3.3). This is the condition that forces a team to know its sampling error before it sets its target.

**(b) The population is stable.** The rate is a property of a population, and the rate only means what it says if the population — the live distribution of inputs — is the one the indicator was defined on. When the population shifts (§3.4, §8.3), the estimate is of a moving target, and the SLO's promise is about a population that no longer exists.

**(c) The failure measured is the failure the user experiences.** The indicator must be defined on the user's contract (§3.7). An indicator that measures a proxy the user never experiences can be perfectly healthy while the user is unhappy (§8.6), and no amount of statistical rigour rescues an indicator defined on the wrong thing.

Where all three hold, a statistical threshold carries the SLO honestly, and the AI-era service is as governable as any other. Where any one fails, the threshold is decorative — and the honest engineering response is not to set it anyway but to fix the condition (grow the sample, stabilise the population, redefine the indicator). This is the answer to the sceptic: the discipline is not unfalsifiable, it is *conditional*, and the conditions are checkable.

| Condition | Holds when… | Fails when… | The fix |
|---|---|---|---|
| (a) Margin vs error | The threshold sits beyond the sampling error | It sits inside it (§13.3) | Enlarge the sample, or widen the band |
| (b) Population stability | The live distribution matches the instrument's | The world moved under the instrument (§3.4) | Re-validate the instrument (§4.5) |
| (c) Failure = the user's failure | The indicator is defined on the contract | It measures a proxy the user never sees (§8.6) | Redefine on the user outcome |

### 9.3 Canarying a model

The canon's canary — rolled out wide, watched closely, rolled back on signal — is the standard way a change is made safe, and the AI era needs it for model changes as much as for code changes. But the canary presumes a signal, and for a model change the signal is the quality indicator of §3, not the availability indicator. A model canary therefore watches a *different instrument* than a code canary, and the instrument is the sampled, uncertain one — which means the canary's decision rule must account for the sampling error, or it will roll forward on noise and roll back on noise. The mechanics of canary deployment (traffic shifting, blast-radius control, the rollback trigger) are owned by [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §3 and [Chaos Engineering](chaos_engineering_guide.md); this guide's contribution is only the observation that a model canary's *decision criterion* is a statistical one and needs the same rigour as §9.2's three conditions. A canary that compares two sampled quality estimates without accounting for their uncertainty is comparing two noisy numbers and calling the difference a result.

### 9.4 Variance reduction as a reliability lever

Here is the constructive consequence of the whole argument, and it reframes a set of existing techniques as *reliability engineering*. If variance is what makes thresholds unreliable (§9.1), then **reducing variance is a reliability lever** — it widens the margin between the observed value and the threshold, which is exactly condition (a) of §9.2. The techniques are not new, and this guide points at them rather than re-deriving them: constraining the decoding so the output space is narrower ([Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md)), grounding the generation in retrieved evidence so the output is anchored to a source, and routing deterministic sub-tasks down deterministic tool paths rather than through the model at all ([AgentOps](ai_llm/agentops_guide.md) §7 on tooling). Each of these reduces the component's variance, and each therefore makes the threshold more reliable — not by making the model "better" in some general sense, but by making the indicator a more trustworthy estimate. Reliability engineering for a non-deterministic component is, to a large degree, variance engineering, and the leverage it buys is measured in the margin §9.2's condition (a) requires.

It is worth naming what variance reduction is *not*. It is not model improvement in the general sense, and it does not necessarily raise the ceiling of §4.4 — a more constrained or more grounded system is not automatically a more capable one. Its specific reliability value is narrower and more certain: it moves the observed value away from the threshold, which is condition (a), and it makes the failure modes more predictable, which is what allows a threshold to be set at all. A team that conflates "made the indicator trustworthy" with "made the service better" will over-claim on the second while doing real work on the first.

### 9.5 The honest limits

Three limits close the argument honestly.

- **Variance can be reduced, not eliminated.** Some residual non-determinism is intrinsic to the component, so the margin of condition (a) is always finite and always has to be known rather than assumed away.
- **Condition (b) can fail invisibly.** A population can shift (§3.4, §8.3) while every indicator remains green, because the indicator is still reporting a rate — of the wrong population. The discipline of §4.5's revisit trigger is what catches this in practice, and it is a discipline, not an automatic check.
- **This guide's central argument is its own.** §9.2's three conditions are not quoted from a standard; they are reasoned from the properties of sampling and the canon's definition of an SLO, and the reviewer must satisfy themselves of the reasoning (§15, §16.1). No primary source this pass codifies a method for setting an SLO threshold over a sampled statistic, and the guide will not pretend otherwise.

---

## 10. The Role and the Skills

The AI era changes the SRE practitioner's day, but less than the discourse suggests, and the useful contribution of this section is the separation of what changes from what stubbornly does not. Career, development and team-shape material is owned by the management and personal guides — [../management/grow_team_guide.md](../management/grow_team_guide.md), [../management/high_performance_team_guide.md](../management/high_performance_team_guide.md), [../management/team_lead_methodologies_guide.md](../management/team_lead_methodologies_guide.md) and [../personal/agent_runtime_engineer_skill_gaps_guide.md](../personal/agent_runtime_engineer_skill_gaps_guide.md) — and this section points at them rather than re-deriving a career framework. It claims **no labour-market, salary or headcount outcome**.

### 10.1 What changes for the practitioner

Four changes are real and specific, and each is a competency rather than a tool:

- **Reading a distribution rather than a status code.** The most concrete shift. A deterministic engineer reads an error code and acts; an AI-era engineer reads a distribution — a mean, a spread, a sample size, a confidence interval — and decides whether a change is signal. The skill is the statistical literacy §3 and §9 demand, applied within an incident.
- **Owning an eval as an engineering artefact.** The eval set, the rubric and the judge (§3.4, §3.6) are now production artefacts with versions, review dates and owners, on a par with the code and the config. Owning one is a new responsibility, not a testing chore.
- **Reasoning about a dependency you do not control.** The provider dependency of §8.5 requires a mode of judgement the canon did not emphasise: designing an envelope around a black box whose behaviour can change silently, and planning for both the loud failure and the quiet one.
- **Knowing what a model must not be trusted to do.** The boundary between "the model can do this" and "this action must not route through the model" is a judgement the engineer now owns (§7.2, §8.4). It is the operational face of the guardrail decision, and it is not answerable from technical performance alone.

### 10.2 What does not change

The changes above are additions; they do not retire the canon's core competencies, which are more valuable, not less, when the measured system is stranger. **Making failure modes legible** — the ability to turn a vague degradation into a specific, named, measurable failure — is the foundational skill, and a non-deterministic component makes it harder, not unnecessary. **Capping operational work** (§5) remains the discipline that keeps a team engineering when the work has fragmented into many small recurring tasks. **The SLO discipline** — writing down the promise, measuring it, setting a target, and treating the shortfall as a budget — is the framework §3 through §9 spend their effort adapting precisely because it is worth adapting. The practitioner who had these competencies before the AI era is not obsolete; they are the practitioner most able to make the AI era legible, because the *hard* part has always been knowing what is worth measuring.

It is worth saying why the unchanged competencies are emphasised so heavily: a team that adopts the new competencies while abandoning the old ones becomes fluent in uncertainty and incapable of holding a service to a target — a worse position than either the old discipline alone or the new. The new skills extend the old; they do not replace them, and the failure mode of treating them as a replacement is quietly common during transitions.

### 10.3 The new competencies, in one list

The additions, stated plainly as things an SRE team's development plan should now contain: statistical reasoning about samples and uncertainty; eval design and stewardship as an owned artefact; dependency judgement for black-box components; uncertainty communication — the ability to tell a stakeholder what a sampled number does and does not mean; and the boundary judgement about what must not be automated. None of these is exotic, and none replaces the canon; they are the canon's competencies extended to a component that answers with a distribution rather than a status.

One competency deserves emphasis because it is rarely named: **uncertainty communication**. An SRE in this era must be able to tell a product owner what a sampled number means and does not mean — that a quality figure of 0.94 with an error of several points cannot underwrite a target one point away, and that a green endpoint indicator says nothing about whether the service is useful. The canon always required an SRE to explain a target to stakeholders; this era requires them to explain the *limits of a measurement*, which is a harder and less practised skill.

The competency shift is legible as a small table, which also serves as the checklist a team's development plan can adopt directly:

| Competency | Deterministic-era form | AI-era form |
|---|---|---|
| Reading a signal | A status code | A distribution (§10.1) |
| Owning a test artefact | The test suite | The eval set, rubric and judge (§10.1) |
| Dependency reasoning | A service you can patch | A black box you cannot (§8.5) |
| Boundary judgement | What is worth automating | What must not route through the model (§10.1) |
| Communication | Explaining a target | Explaining a measurement's limits (§10.3) |
| The unchanged core | Make failure legible; cap toil; hold an SLO | Exactly the same (§10.2) |

### 10.4 Where the career material lives

The organisational consequences — how to grow the team, how to shape seniority, how to lead through the transition — are owned elsewhere and are pointed at, not restated: [../management/grow_team_guide.md](../management/grow_team_guide.md), [../management/high_performance_team_guide.md](../management/high_performance_team_guide.md), [../management/team_lead_methodologies_guide.md](../management/team_lead_methodologies_guide.md), [../management/authority_skills_guide.md](../management/authority_skills_guide.md) and [../personal/agent_runtime_engineer_skill_gaps_guide.md](../personal/agent_runtime_engineer_skill_gaps_guide.md). This guide's contribution is confined to *which* competencies the reliability discipline newly requires (§10.1, §10.3); how a team develops and organises around them is the management guides' subject, and no outcome figure about the labour market is offered here.

---

## 11. The Regulated Estate

The governed, audited institution is where the guide's argument lands hardest, and the material there is largely owned. This section adds one clause plus the links, and then the one thing the canon's machinery contributes that the governance literature does not usually name.

### 11.1 The control set — one clause and the links

The controls a regulated institution needs around AI actions — change control for AI-driven changes, the audit trail that survives the "explain this incident" test, the kill switch, the data-protection constraints, and the treatment of the model provider as a supervised dependency — are owned by [AI for IT](ai_llm/ai_for_it_guide.md) §11 and, at the governance layer, by [AI Governance Frameworks](ai_llm/ai_governance_framework_guide.md). This guide does not restate the control set; it *consumes* it. Where a reliability artefact below is said to be a control, the control vocabulary and the audit expectation behind it are the sibling's.

The demarcation is simple: this guide consumes the sibling's control vocabulary and adds only the reading of the canon's own artefacts as controls. Where §11.2 says an SLO is a control artefact, the control's design, testing and evidence expectations are the sibling's; what is new here is the observation that the SRE artefacts already satisfy the shape of those expectations.

### 11.2 The one addition: the reliability machinery is itself evidence

What this guide adds is a reframing, and it is the section's whole contribution: **the SRE machinery is itself evidence, and it lands inside the control set rather than beside it.**

- An **SLO** is not only a target; it is a *control artefact* — a written statement of what the service promises, which an auditor can read and test against.
- An **error budget** is not only a spending permit; it is a written **risk-appetite statement** — the institution's own declaration of how much unreliability it will tolerate before it acts, which is exactly the kind of formalised appetite a supervisor expects to find.
- A **postmortem** is not only a learning document; it is a **record** — a dated, attributed account of what happened and what was fixed, which is what an audit trail of incidents actually is.
- The **on-call rotation** is not only a staffing arrangement; it answers the audit question "**who was on call, and what did the control do?**" with a name and a chain.

The consequence is that an institution that already runs the canon does not need to build a parallel AI-governance evidence stack from nothing — it needs to extend the artefacts it already produces (§3.7's versioned instrument, §4.5's revisit trigger, §7.4's logging requirement, §8.4's guardrail error budget) so that they cover the model-backed service. The reliability artefacts are the evidence the regulated estate will be asked for; the AI era's demand is that they be extended, not replaced.

This reframing has a practical edge for the audit conversation. When a supervisor asks how the institution governs a model-backed service, the answer is not a bespoke AI register assembled for the question — it is the SLO record, the budget and its policy, the postmortem archive and the on-call roster, extended to cover the model-backed service. The reliability artefacts are already the shape that answer takes; the work of compliance is to make sure each artefact's scope reaches the model, which is why the extensions in §3.7, §4.5, §7.4 and §8.4 are framed as *scoping* changes to existing artefacts rather than as new controls bolted alongside them.

Each artefact can be read twice — as an operational instrument and as a control the audit will recognise:

| Reliability artefact | Control reading | The audit question it answers |
|---|---|---|
| The SLO | A control artefact | What did the service promise to the user? |
| The error budget and its policy | A written risk-appetite statement | How much unreliability will we tolerate, and what happens when we exceed it? |
| The postmortem | A dated record | What happened, what was changed, who owns the fix? |
| The on-call roster | Attribution | Who was on call, and what did the control do? |

---
## 12. The Failure Modes

The failure modes below are the ways an AI-era reliability practice goes wrong. Each has a symptom, a cause, and — crucially — a **guardrail that is a concrete artefact or gate**, not an exhortation. The first five are the five named in the guide's design; the sixth and seventh are the ones the argument of §3–§11 makes it dishonest to omit.

| # | Failure mode | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **A dashboard that measures nothing** | Availability is green; users report the service is useless; no indicator moves | Only countable, endpoint-level indicators are instrumented; no quality indicator exists (§8.6) | A **published quality indicator** on live or shadowed traffic, with its sample size and uncertainty (§3.7), owned by a named person |
| 2 | **An AI-assisted on-call that hides an incident** | Alerts stop firing; the incident "handled itself"; no postmortem, no learning | Triage suppresses the alert as a known pattern, so the incident becomes a thing the system absorbed and the humans never saw (§6.2) | A **suppression register**: every auto-suppressed alert logged with its rule and reviewed periodically; a rule that no suppression is silent (an artefact per suppression) |
| 3 | **Eval-set rot** | The indicator *improves* as the world moves on; input distribution has drifted under the fixed eval | The eval set ages out of distribution, so the measurement drifts with the world and no one sees the crossing (§3.4) | A **dated, versioned eval instrument** with a mandatory re-validation gate at a fixed interval and on any population shift (§4.5) |
| 4 | **Auto-remediation that widens the blast radius** | An automated fix turns one degraded component into several failed ones | Automation acts on a hypothesis without bounded scope; the guarded-autonomy ladder is skipped (mechanism owned by [AI for IT](ai_llm/ai_for_it_guide.md) §8) | A **blast-radius bound** on every automated action, with a mandatory human approval gate above a declared scope threshold |
| 5 | **An SLO copied from a deterministic service onto a probabilistic one** | The service is permanently "in breach", or permanently "fine", for reasons unrelated to users | The threshold sits *inside* the sampling error, so re-sampling crosses it either way (§3.3, §9.2) | A **threshold-margin gate**: no sampled indicator may carry an SLO unless its sampling error is measured and smaller than the threshold margin, evidenced in the SLO's own record |
| 6 | **The silent quality ledger that does not exist** | Error budget reads healthy; the service decays to uselessness; the freeze never fires | Quality drift produces no failed event, so the burn rate has nothing to accrue (§4.3) | A **timer-based freeze gate**: an automatic change-freeze review on the provider's release cadence, independent of whether any error was observed (§4.5) |
| 7 | **The uncaptured trace** | A model-involved incident cannot be analysed beyond "the model produced a bad output" | "What the model saw" was never logged, so it cannot be reconstructed after the fact (§7.4) | A **logging-requirement gate at design review**: a service may not enter production unless its prompt, context, model version and tool inputs are captured at postmortem fidelity |

Each guardrail in the table is deliberately an artefact a reviewer can point to — a register, a dated instrument, a margin record, a gate in a pipeline — because a guardrail that is a sentiment rather than an artefact cannot be audited, and this section exists to be audited.

Two observations about the set. First, the modes are not independent: failure 1 (a dashboard that measures nothing) makes failures 3, 5 and 6 invisible, because each of them is a failure of measurement that only a working quality indicator would surface — so the guardrail against failure 1 is the precondition for detecting the rest. Second, every guardrail is a *mechanism* rather than a hope: a register, a dated artefact, a gate in a pipeline. The reason is the canon's own — a reliability practice is only as good as the artefacts it leaves behind, and a failure mode without a concrete guardrail is a failure mode that will recur.

---

## 13. The Cymbal Bank Worked Example

**This section is explicitly fictional and illustrative.** Cymbal Bank is the only institution in this guide and the only bank persona in any worked example; no real bank's internal arrangements are described here. Every quantity below is **illustrative** — a number chosen to make the decision legible, not a measurement, a benchmark, or a claim about any real system. The example works one decision through the difference AI makes, end to end, and ends on the thesis sentence.

### 13.1 The service and the decision

Cymbal Bank runs SRE for an AI-backed service it calls the **resolution assistant** (illustrative): a model-backed capability that resolves routine customer requests end to end, using internal tools under guarded autonomy. The service has a countable endpoint, a tool-call layer, and a quality dimension — did the assistant actually resolve the customer's request correctly? — that can only be measured by evaluating a sample of live interactions against a rubric. The decision before the team: **whether a quality indicator derived from that sampled evaluation can honestly carry the service's SLO, and if so, how.**

### 13.2 The evidence available

The team has three pieces of evidence (all illustrative):

1. **The sampled quality estimate.** At the current sample size and rubric agreement, the quality indicator reads **0.94, with a sampling error of roughly ±0.03** across the observation window. The team proposes an SLO threshold of **0.95** on that indicator.
2. **The countable indicators.** Endpoint availability, tool-call failure rate and timeout rate are all computed exactly from events and are healthy.
3. **The instrument history.** The eval rubric was last re-validated two quarters ago; the input distribution has since shifted (a new product bundled into the assistant's scope).

The list is illustrative and deliberately small; a real service would carry more evidence, but the decision turns entirely on the three items above, and adding more would only obscure which of them is decisive.

### 13.3 The analysis

Apply §9.2's conditions to the proposed threshold, honestly.

- **Condition (a), sampling error versus threshold margin.** The proposed threshold is 0.95 and the observed value is 0.94 — a margin of 0.01. The sampling error is ±0.03. **The threshold sits inside the sampling error**, by a wide factor. Condition (a) fails decisively: the service would cross 0.95 and drop back below it on re-sampling alone, with no change in real quality. Setting this threshold would produce a permanent, meaningless breach-or-fine churn (§3.3, §12 failure 5).
- **Condition (b), population stability.** The input distribution shifted when the new product was added, and the eval instrument was last validated before that shift. So the estimate is of a partly obsolete population, and condition (b) is also in doubt until the instrument is re-validated (§3.4).
- **Condition (c), the measured failure equals the user's failure.** The rubric measures resolution correctness, which *is* the user's contract — so condition (c) holds, and this is the one condition the proposal satisfies.

The analysis is therefore clean and unfavourable: the proposed SLO threshold is not a threshold. Two of the three conditions fail, and one of them (the margin) fails by a factor of three relative to the sampling error.

The analysis also shows why the team's instinct was not foolish. 0.95 is a familiar-looking service target, and the endpoint indicators are healthy, so the proposal *reads* like a reasonable SLO. The work of §9 is precisely to catch the proposal that looks reasonable and is arithmetically meaningless: the failure is invisible in the target's plausibility and visible only in the ratio of the margin to the error. That is the difference the AI era makes to SLO-setting — the same act, one more number to check before the target means anything.

The analysis makes the team's later commitment concrete as well. "Set the target below the evaluated ceiling once the margin clears the sampling error" is not a vague aspiration; it is a computable condition. The team can state, today, roughly how large the sample must grow for the error to fall below a proposed margin — and therefore roughly when the quality SLO becomes settable. The decision to *wait* is thus itself a plan with a trigger, not an indefinite deferral, which is what distinguishes the honest position from merely declining to measure.

A last observation about the example, because it is the whole point of the section. Cymbal's decision was not to *avoid* the SLO; it was to refuse to attach a promise to a number that could not carry it, and to keep measuring while the number grew strong enough to. That is the canon's own habit — a service is held to what can be observed, and a target beyond observation is a fiction — applied to an indicator whose precision must first be established. The AI era does not ask the SRE to give up the target; it asks them to earn it by measuring the measurement first.

### 13.4 The decision taken

The team decides **not** to set a hard SLO on the sampled quality indicator at the current sample size, and instead to do three things:

1. **Set the SLO on the countable indicator.** The service's SLO is attached to what can be measured exactly — the endpoint, tool-call and timeout indicators — which is a real and honest target even though it is not the quality target the team wanted.
2. **Maintain a quality ledger with declared uncertainty.** The sampled quality estimate is tracked over time with its sample size and sampling error published alongside it (§3.7, §4.3), and it is a *review trigger*, not a breach threshold — deterioration in the ledger opens a review, it does not fire a freeze.
3. **Treat the target as a ceiling to be discovered, not 100.** The team commits to estimating the achievable ceiling by evaluation and setting a quality target below it later, with margin, once the sample is large enough that the margin clears the sampling error (§4.4).

The decision is not "we cannot measure quality". It is "we will not pretend a number carries a promise the number cannot support" — which is the canon's own discipline, applied to an indicator whose precision the team has measured rather than assumed.

Laid out as a decision record, the example's shape is ordinary — the AI era did not require an exotic method, only an extra number checked before the target was set:

| Element | What the team did | Where it is argued |
|---|---|---|
| The proposal | A 0.95 quality SLO on a sampled indicator | §13.2–§13.3 |
| The finding | The threshold sits inside the sampling error | §9.2 condition (a) |
| The decision | No hard quality SLO yet; the SLO goes on the countable indicator; a quality ledger as a review trigger | §13.4 |
| The guardrail | A threshold-margin gate in the reliability review | §13.5 |
| The known gap | Proxy fidelity, ceiling stability, masked quality, correlated failure | §13.6 |

### 13.5 The guardrail adopted

The team adopts a concrete guardrail: a **threshold-margin gate** in the reliability review. No sampled indicator may be attached to an SLO in Cymbal's service catalogue unless the SLO's own record contains (i) the recorded sampling error, (ii) the declared threshold margin, and (iii) evidence that the margin exceeds the error. The gate is an artefact with a named owner, and it also carries the re-validation obligation: the eval instrument must pass a re-validation review on a fixed interval and on any population shift, so that condition (b) is checked rather than assumed (§4.5, §12 failure 3).

### 13.6 What the team accepts it does not know

The decision is honest about its own limits, and the team records four things it does not know:

- **Whether the rubric's judgement matches the user's judgement.** The instrument is a proxy for "did the customer's request get resolved", and no study yet confirms the proxy tracks the truth (§3.8). Until it does, the ledger measures the rubric, not the customer.
- **Whether the achievable ceiling is stable.** The ceiling of §13.4 is estimated from a system that includes a hosted model; a provider change (§4.2) can move the ceiling, and the estimate ages.
- **Whether the countable SLO is masking a quality problem.** Because the hard target is on the endpoint, the service could satisfy its SLO while the quality ledger decays — the green-dashboard-broken-service gap (§8.6) is narrowed by the ledger but not closed.
- **What the correlated failure looks like.** The ledger and the endpoint indicators are assumed independent; a single cause that degrades both at once is not yet instrumented.

The team's position is the guide's position: it has made the service as governable as the evidence allows, and it has written down the gap between what it can promise and what it can observe, rather than closing that gap with a number it cannot support.

AI does not change what reliability is; it changes what can be measured, who is on call, and what counts as a control.

---

## 14. The Anti-Patterns

A checklist, each line an anti-pattern with a one-clause correction. These are the moves that look like progress and are not. The list is deliberately short: an anti-pattern list that runs to thirty items does not function as a checklist, and these twelve are the ones whose *correction* changes an artefact or a gate — which is the only kind of correction that survives an audit.

1. **The green dashboard.** Endpoint up, quality unknown. — *Instrument the quality dimension, or state plainly that the service has no quality indicator.*
2. **The eval score as an SLI.** A release-time instrument published as the production indicator. — *Place the eval correctly: release-time instrument versus production indicator (§8.2).*
3. **The threshold inside the noise.** An SLO whose margin is smaller than the sampling error. — *Measure the error before setting the target, or the target is a coin (§3.3, §9.2).*
4. **The immortal eval set.** An instrument never re-validated against the live population. — *Date it, version it, and re-validate on a schedule and on every population shift (§3.4).*
5. **The budget that cannot see silence.** A burn rate that assumes every failure is an event. — *Add the quality ledger and the timer-based freeze gate (§4.3, §12 failure 6).*
6. **The freeze that freezes nothing.** A change-halt lever pointed at your releases while the change comes from the provider. — *Add a second lever: a pinned model contract or a canary on provider change (§4.2).*
7. **The un-measured guardrail.** A guardrail with no false-positive rate, blocking valid work invisibly. — *Give every guardrail its own error budget and a reviewed exception queue (§8.4).*
8. **The invisible hand-off.** An agent responder that pages a human with no reviewable trace of what it did. — *Log every suppression and every hand-off as an artefact (§12 failure 2).*
9. **"The model did it."** A postmortem that names the model and stops. — *Push the analysis further into the human system, to a person per consequential decision (§7.3).*
10. **The un-costed second provider.** Diversifying outage risk by importing a second drift surface without counting it. — *Treat a second provider as a reliability decision with a drift cost (§8.5).*
11. **The uncaptured trace.** A model-involved incident that cannot be analysed beyond the output. — *Make postmortem-fidelity logging a design-review gate (§7.4, §12 failure 7).*
12. **The deterministic SLO ported wholesale.** A target designed for a countable service dropped onto a sampled one. — *Re-derive the target's margin from the indicator's uncertainty, or do not set it (§12 failure 5).*

---
## 15. The Claims Audit

Every substantive claim in this guide is classified below. The status codes are: **✅ verified at a primary source this pass · ⚠ partially verified (secondary source, or primary source confirms only part) · ❌ could not be verified.** Every source note carries a **date** and the **kind of source** — framework, dated primary source, repo-internal cross-reference, this guide's own argument, or omission. The table is ordered so the repo-internal and dated-primary rows come first and the argued rows follow, because a reader should see at once how small the verified core is and how large the argument is — that ratio is the honest signal of a gap-filling guide.

Two rows deserve a note for how they were decided. Row 12 (the "not yet codified" flag) is marked ✅ *attributed, not adopted*: this guide does not itself verify that agent on-call is uncodified — it cites the sibling that says so and adopts the same line, which is a different act from asserting the claim. Row 21 (the Cymbal quantities) is marked ⚠ because every figure in it is illustrative; a reader who mistook 0.94 for a measurement would have misread the section, and the status column exists precisely to prevent that misreading.

| # | Claim | Status | Source note |
|---|---|---|---|
| 1 | The reliability vocabulary is heavily claimed in this repository — `\bSRE\b` in 41 files, `observability` in 185, `runbook` in 133, `on-call|incident response` in 230, `error budget|SLO|SLI` in 49, `AIOps` in 7, `toil` in 8, `site reliability` in 9 — and no `sre_guide.md` exists | ✅ | **Repo-internal cross-reference**, counts verified by the parent session on **2026-10-16** |
| 2 | The SRE canon (what SRE adds, the toil definition, the error-budget machinery and freeze policy, the overlay table, ITIL, the on-call rotation and escalation matrix) is owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §4.3–§4.4 and §6 | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 3 | AI in IT operations (AIOps, telemetry AI, anomaly detection, alert-noise reduction, RCA, ITSM AI, runbook automation, auto-remediation, the regulated control set) is owned by [AI for IT](ai_llm/ai_for_it_guide.md) §1–§13 | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 4 | Operations of AI agents, the agent SLI set, and the honest flag that agent on-call is consolidating but not codified are owned by [AgentOps](ai_llm/agentops_guide.md) §3.5 and §5.4 | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 5 | The SLI/SLO/error-budget derivation, "100 is the wrong target", the availability arithmetic, the deployment strategies and game days are owned by [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3, §3, §7.3 | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 6 | Failure injection, steady-state hypotheses and the experiment method are owned by [Chaos Engineering](chaos_engineering_guide.md) | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 7 | The ML lifecycle, the drift taxonomy, retraining loops and model deployment are owned by [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) | ✅ | **Repo-internal cross-reference**, section headings inspected 2026-10-16 |
| 8 | SRE "seeks to balance the risk of unavailability with the goals of rapid innovation and efficient service operations"; for the risk metric "we focus on _unplanned downtime_"; and "the user experience is dominated by less reliable components like the cellular network or the device they are working with" (with the book's own "99% reliable smartphone" and "99.99% … 99.999%" illustration) | ✅ | **Dated primary source** — SRE book ch.3 *Embracing Risk*, O'Reilly **2016** (Beyer/Jones/Petoff/Murphy); an early version appeared in *;login:* **August 2015**, vol. 40 no. 4 |
| 9 | "For a postmortem to be truly blameless, it must focus on identifying the contributing causes of the incident without indicting any individual or team for bad or inappropriate behavior"; "You can't 'fix' people, but you can fix systems and processes"; and the **2016** future-work sentence "machine learning to help predict our weaknesses, facilitate real-time incident investigation, and reduce duplicate incidents" | ✅ | **Dated primary source** — SRE book ch.15 *Postmortem Culture*, O'Reilly **2016** |
| 10 | The Example Error Budget Policy is authored by Steven Thurgood with metadata Date **2018-02-19**, Approval Date **2018-02-20**, **Revisit Date 2019-02-01**; its Service Overview states "New releases of the backend code are pushed daily"; its freeze scope covers "all changes and releases" parenthetically "(including data changes)"; and it states "This policy is not intended to serve as a punishment for missing SLOs" | ✅ | **Dated primary source** — SRE Workbook Appendix B, *Example Error Budget Policy*, metadata dated **2018-02-19** with a **2019-02-01** revisit date (only the "(including data changes)" scope, the revisit date and the daily-cadence point are newly used here; the freeze sentence, the postmortem trigger, the outages statistic and the error arithmetic are left to §6.3 of the sibling guide) |
| 11 | Google's on-call chapter is offered as "a guideline for all SRE teams in Google", "might not be immediately applicable to all contexts", and uses "engineering work as the primary means to scale production responsibilities" | ✅ | **Dated primary source** — SRE book ch.11 *Being On-Call*, O'Reilly **2016**; earlier version *;login:* **October 2015**, vol. 40 no. 5 |
| 12 | Agent on-call / "LLM SRE" is consolidating but **not yet a codified practice** | ✅ *(attributed, not adopted)* | **Repo-internal cross-reference** — the honest flag is [AgentOps](ai_llm/agentops_guide.md) §5.4; this guide adopts the same line and overclaims nothing |
| 13 | The measurement argument — an eval score is not an availability figure; a sampled success rate carries a confidence interval and a threshold inside that error is not a threshold; the eval instrument itself drifts; the unit of the indicator is a design choice; a reference-free judge imports its own error | ⚠ | **This guide's own argument** — reasoned at §3 from the properties of sampling; no external standard is cited as its source |
| 14 | The error-budget argument — a provider-side model change is a data change you neither made nor were told about; silent quality drift produces no error and no ledger entry; the budget must be set against an evaluated ceiling, not 100; the policy must be revisited on model change | ⚠ | **This guide's own argument** — reasoned at §4, anchored on the ch.3 and Appendix B quotes (rows 8, 10); the provider-behaviour premise is structural, not sourced (see §16.1) |
| 15 | The toil argument — AI absorbs some operational work and creates new toil (prompt and eval maintenance, guardrail-exception review, model-change review, the drift-or-regression triage); prompt maintenance is substantially toil under the canon's own attributes; AI changes the cost curve, not the shape | ⚠ | **This guide's own argument** — reasoned at §5 by applying the attributes owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.2; the attributes are not restated |
| 16 | The on-call argument — the agent-assisted responder versus the agent as responder; an agent cannot be accountable; the new cost of refuting a confident wrong diagnosis; load may rise before it falls | ⚠ | **This guide's own argument** — reasoned at §6; the "not yet codified" line is attributed (row 12), not asserted |
| 17 | The postmortem argument — the trace improves reconstruction; blame cannot attach to a model so the object of review becomes the decision set around it; "the model did it" is a new blame-avoidance; postmortem quality is a logging requirement set before the incident | ⚠ | **This guide's own argument** — reasoned at §7, anchored on the ch.15 quotes (row 9) |
| 18 | The SRE-for-AI argument — an AI service is a service with an owner and a target; evals are an indicator family; drift is a threshold question; a guardrail that blocks valid work is a reliability failure; the provider is a dependency whose uptime is not your SLO, with outage ≠ change and a second provider as a drift cost; the green-dashboard-broken-service gap | ⚠ | **This guide's own argument** — reasoned at §8; the drift taxonomy is deferred to [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md) rather than re-derived |
| 19 | The central argument — an SLO is a promise about a distribution; a statistical threshold carries it only if (a) the sampling error is smaller than the threshold margin, (b) the population is stable, and (c) the failure measured is the failure the user experiences; variance reduction is a reliability lever | ⚠ | **This guide's own argument** — reasoned at §9 from the properties of sampling and the canon's definition of an SLO; no primary source codifies a method (see §16.1) |
| 20 | The failure-mode table of §12 and the anti-pattern list of §14, including every guardrail, artefact and gate proposed in them | ⚠ | **This guide's own argument** — each traced to a mechanism argued in the guide rather than to an external source |
| 21 | The Cymbal Bank scenario of §13 — the resolution assistant, the 0.94 estimate, the ±0.03 sampling error, the 0.95 proposed threshold, and every other quantity | ⚠ | **This guide's own argument** — explicitly fictional and **illustrative**; Cymbal Bank is the sole institution and the sole persona; no figure is a measurement |
| 22 | **No mean-time-to-detect figure, no mean-time-to-repair figure, no benchmarking figure and no market/adoption statistic** was printed anywhere in this guide | ✅ | **Omission** — recorded deliberately; see the note below. The sibling [AI for IT](ai_llm/ai_for_it_guide.md) §9 owns the published-numbers material, and this guide does not re-own it |
| 23 | **No vendor was ranked, sized or compared**, and no vendor market share appears | ✅ | **Omission** — recorded deliberately; see the note below |
| 24 | **No framework, article, clause, metric figure or provider behaviour was asserted from memory** — every named source is either one of the four dated primary sources above or a repo-internal cross-reference | ✅ | **Omission / method** — every primary-source row is one of rows 8–11; provider-side behaviour is reasoned structurally (row 14) because no primary source was verified for it |

**Note on rows 22–24, and why they are checklist rows.** These rows are printed because an audit should be able to confirm what this guide *did not* say, and the refusals are not modesty:

- **Row 22 is omitted because the figures are owned elsewhere and would be re-owned here.** The published mean-time-to-detect / mean-time-to-repair material belongs to [AI for IT](ai_llm/ai_for_it_guide.md) §9; reprinting it would duplicate the sibling, which is the one real failure mode this guide is written to avoid. The guide's subject is *what the canon's objects become*, not the size of any number.
- **Row 23 is omitted because a ranking or a share would be a vendor comparison this guide has no primary source for, and a fabricated comparison is the fastest way to discredit a reliability document.**
- **Row 24 is omitted because a reference asserted from memory is the exact error the series exists to prevent.** The guide's sourcing constraint (§1) states it: a source appears only where the dated primary source was given to this pass or where the claim is cross-referenced to the sibling that owns it.

The honest shape of this table is that rows 13–21 — the guide's own argument — are the bulk, and rows 2–12 are repo-internal cross-references and four dated primary sources. That is the correct shape for a guide whose subject is a reconciliation no prior source has written: the verified list is short, the argued list is long, and the guide says which is which.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References, and the Closing Summary

### 16.1 What could not be verified

Honest entries; each is a thing this pass could not establish at a primary source, or a thing the guide refused to print.

1. **No primary source was verified this pass for any provider-side model-change notification practice.** The argument of §4.2 — that the canon's freeze lever cannot govern a change that is not yours — depends on the *structure* of a dependency you do not control, not on a verified behaviour of any provider. No provider's notification, deprecation or versioning policy was verified this pass, so the guide reasons structurally and asserts no provider's conduct. Any statement about "providers notify X days ahead" would be fabricated by construction and is not made.
2. **No primary source establishes an accepted method for setting an SLO threshold over a sampled eval statistic.** The three conditions of §9.2 and the sampling-error rule of §3.3 are this guide's own reasoning, not a standard. The canon's threshold derivation (owned by [Zero-Downtime System Design](zero_downtime_system_design_guide.md) §2.3) is for countable indicators; no source verified this pass extends it to a sampled one. The reviewer must satisfy themselves of the reasoning.
3. **Whether any published error-budget policy contemplates a provider-side model change.** This pass verified one dated exemplar (the Workbook's Example Error Budget Policy, row 10) and found in it the "data changes" scope and the revisit date that §4.2 and §4.5 use — but whether *any* published policy explicitly contemplates a change to a hosted model made by a third party was not verified, and the guide does not claim it. The extension in §4.2 and §4.5 is this guide's argument built on what the exemplar already contains.
4. **Tool/method limitation — verification is inherited, not re-performed.** The four `sre.google` primary sources used above were available to this pass as cached, complete extractions of the pages produced by the parent pass; no live `web_extract` or `web_search` was run this pass, so the guide records that its primary-source verification is *inherited from that pass* rather than independently re-performed here. This is a limitation on this guide's own verification, recorded rather than concealed.
5. **A claim this guide deliberately refused to print.** No mean-time-to-detect figure, no mean-time-to-repair figure, no benchmarking figure, no market size, no adoption statistic and no vendor ranking or share is printed anywhere in this guide. The refusal is deliberate and is recorded as rows 22–23 of §15; the nearest such figures live in [AI for IT](ai_llm/ai_for_it_guide.md) §9 and are *not* re-owned here.
6. **Whether a provider's contract, on its own terms, obliges it to notify a customer of a model change in advance, and on what cadence.** §4.2 reasons that a provider-side change behaves like a data change you neither made nor were told about; the guide does **not** claim that any provider contracts otherwise, and it prints no notification window, no deprecation period and no service-level term, because no such term was verified this pass. The structural point stands on its own: if you cannot freeze the change, your freeze policy governs half the change.

### 16.2 Glossary

| Term | Definition (as used in this guide) |
|---|---|
| **Agent** | A software actor that takes actions through tools, holds state across steps, and is delegated authority (§1.2, §6). |
| **Agent-assisted responder** | An agent that drafts, retrieves and proposes while a human decides and owns the outcome (§6.1). |
| **Agent as responder** | An agent that acts on production events; it cannot be accountable, so it can only be a step within a chain that terminates at a human (§6.1). |
| **Burn rate** | The speed at which the error budget is consumed; assumes errors are events, which is exactly what silent quality drift denies (§4.3). |
| **Canary (model)** | A limited rollout of a model change watched on the *quality* indicator, whose decision rule must account for sampling error (§9.3). |
| **Confidence interval** | The range within which a sampled statistic plausibly lies; its width is the sampling error, and it is the reason a threshold inside it is not a threshold (§3.3). |
| **Drift** | A change over time in the relationship between inputs, model and world; owned as a taxonomy by [MLOps Lifecycle Frameworks](mlops_lifecycle_frameworks_guide.md); used here only as a *threshold* question (§8.3). |
| **Error budget** | The tolerable shortfall from perfection; machinery owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.3 (§4). |
| **Eval** | A measurement of model or agent output against a reference or rubric; a release-time instrument or a production indicator depending on its population (§3.2, §8.2). |
| **Guardrail** | A preventive control that stops an inadmissible output or action; a guardrail that blocks valid work is itself a reliability failure (§8.4). |
| **Measurement-system drift** | The eval instrument's own definition moving away from the world, distinct from model drift (§3.4). |
| **Non-determinism** | The property that the same input may produce different outputs across runs, making a model-backed indicator an estimate from a sample (§3.1, §9). |
| **On-call** | The standing arrangement of responsibility for production events; rotation and escalation owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §4.3–§4.4 (§6). |
| **Quality ledger** | A record of sampled quality over time, with declared uncertainty, treated as budget consumption even when no request failed (§4.3). |
| **Sampling error** | The uncertainty in a statistic computed from a sample; the quantity a threshold must exceed to be meaningful (§3.3, §9.2). |
| **Service level indicator (SLI)** | The thing actually measured, as a number over a window; exact for event counts, an estimate for samples (§1.2, §3). |
| **Service level objective (SLO)** | The target band set for an indicator — a promise about a distribution, not about any single request (§9.2). |
| **Silent quality drift** | Quality decay that produces no failed event, so the error budget's accounting sees nothing (§4.3). |
| **Threshold margin** | The distance between an observed value and the SLO threshold; must exceed the sampling error for the comparison to mean anything (§9.2, §13.5). |
| **Toil** | Operational work that is manual, repetitive, automatable, tactical and devoid of enduring value; definition owned by [Operational Support Frameworks](operational_support_frameworks_guide.md) §6.2 (§5). |
| **Trace** | The machine-readable record of what a model-backed service saw and did; the raw material of §7 reconstruction, whose existence is a design decision (§7.1, §7.4). |
| **Variance reduction** | Lowering a component's spread so the indicator becomes a more trustworthy estimate; a reliability lever (§9.4). |

### 16.3 Cross-references in this series

- **The canon this guide reads forward:** [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md) — §4.3–§4.4 on-call, rotation and escalation; §6.1 the SRE overlay and the postmortem; §6.2 toil and its attributes; §6.3 error budgets and the freeze policy; §6.4 the engagement models; and ITIL at §2.
- **AI in operations, pointed at rather than restated:** [ai_llm/ai_for_it_guide.md](ai_llm/ai_for_it_guide.md) — §1 AIOps, §4 anomaly detection and alert noise, §5 root-cause analysis, §8 auto-remediation under guarded autonomy, §9 the published-numbers material (owned there, not reprinted here), §11 the regulated control set.
- **Operations of AI agents:** [ai_llm/agentops_guide.md](ai_llm/agentops_guide.md) — §3.5 the agent SLI set, §5.2 alerting, §5.4 the honest "not yet codified" on-call flag, §6 guardrails and cost.
- **The reliability derivation and deployment:** [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — §2.3 the SLI/SLO derivation and "100 is the wrong target", §3 deployment strategies, §7.3 game days.
- **Failure injection:** [chaos_engineering_guide.md](chaos_engineering_guide.md) — the experiment design, steady-state hypotheses and safety guardrails.
- **The ML lifecycle and drift:** [mlops_lifecycle_frameworks_guide.md](mlops_lifecycle_frameworks_guide.md) — the lifecycle stages, the drift taxonomy and the retraining loop.
- **Adjacent, cross-referenced only:** [ai_llm/ai_governance_framework_guide.md](ai_llm/ai_governance_framework_guide.md), [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md), [ai_llm/llm_evaluation_vs_validation_guide.md](ai_llm/llm_evaluation_vs_validation_guide.md), [constrained_decoding_frameworks_guide.md](constrained_decoding_frameworks_guide.md), [../banking/agentic_grc_guide.md](../banking/agentic_grc_guide.md) and the [../banking/](../banking/) guides, [../management/grow_team_guide.md](../management/grow_team_guide.md), [../management/high_performance_team_guide.md](../management/high_performance_team_guide.md), [../management/team_lead_methodologies_guide.md](../management/team_lead_methodologies_guide.md), [../management/authority_skills_guide.md](../management/authority_skills_guide.md) and [../personal/agent_runtime_engineer_skill_gaps_guide.md](../personal/agent_runtime_engineer_skill_gaps_guide.md).

### 16.4 The closing summary

The canon is not obsolete, and the AI-native operating model is not its replacement. The canon is a discipline for deciding how much failure is acceptable and for keeping the decision honest, and that discipline is exactly what a model-backed service needs more of, not less. What the AI era changes is the *material* the discipline works on. The indicator can no longer be a count of adjudicable events, so it becomes a sample that carries an uncertainty, and a threshold that ignores that uncertainty is a coin rather than a target (§3, §9). The budget can no longer assume that every failure is an event, because quality can decay silently while every request succeeds, so a quality ledger has to stand beside the error ledger or the budget's green light reports on the instrumentation rather than the service (§4). The freeze can no longer point only at your own releases, because the change may come from a provider with no release on your side (§4.2). The roster can gain an agent as assistant, and — uncomfortably — as responder, and the field has not yet codified how an agent is held on call, so the honest position is the sibling guides' own: consolidating, not settled (§6.4). The postmortem has to find a new object of review, because blame cannot attach to a model and "the model did it" is a blame-avoidance rather than a conclusion (§7). And the guardrail has to be counted as a control that can *fail*, because a guardrail that blocks a valid request is a reliability event like any other (§8.4).

Three clauses organise all of it, and they are the guide's whole claim. **What can be measured** changes, because a sampled quality estimate and an exact event count are different kinds of number, and only one of them was the thing the canon assumed. **Who is on call** changes, because an agent can draft, retrieve and even act, but cannot be accountable, so a human must still terminate the chain and own the outcome. **What counts as a control** changes, because a guardrail's false positives, a provider's silent change and a logging decision made before an incident are all controls now, and the reliability artefacts — the SLO, the error budget, the postmortem, the roster — are themselves the evidence the regulated estate will ask for (§11). None of this retires the canon's competencies; it extends them. The practitioner who could make a failure legible, cap the operational work and hold an SLO to a target is the practitioner most able to make this era legible, because the hard part has always been knowing what is worth measuring — and what is worth measuring is precisely what a non-deterministic component makes uncertain.

A note on scope, so the close does not overstate. This guide has adapted the canon's objects to a model-backed service; it has not surveyed the field, sized a market or measured an outcome, and it has deliberately not printed the figures a survey would use. Its claims are about *structure* — what a threshold, a budget, a roster and a postmortem each become — and a structural claim is falsifiable by the reader: set a threshold inside your sampling error and watch it cross on noise; point your freeze at your own releases and leave the provider's changes ungoverned. The test of the guide is not whether its figures agree with a survey but whether its structural claims match what happens when a real service is operated this way.

The guide has kept three promises about method. It has pointed at the six owners rather than restating them, so that its argument is the reconciliation and not the components (§1.4, §15). It has printed no figure it could not source and no vendor comparison at all (§15 rows 22–23). And it has named what it could not verify rather than smoothing it over (§16.1). Those are not disclaimers; they are the discipline the guide is asking of the practice it describes.

The change, in one table, on the guide's own terms:

| Dimension | What AI changes | What it does not change |
|---|---|---|
| Measurement | A sampled quality estimate joins the exact event count | The demand to measure what the user experiences |
| The target | Its width is set by the sampling error | That a target is a band, not a point |
| On-call | An agent can assist, and can act, but not be accountable | That a human must terminate the chain |
| The budget | A quality ledger stands beside the error ledger | That the budget is spent by what has been observed |
| The postmortem | The review targets the decision set around the model | That the account must be blameless |
| Controls | Guardrail false positives and provider changes are controls | That the artefacts are the evidence |

And, to be plain about the boundary one last time: everything above is this guide's reconciliation of material owned elsewhere. The canon is the sibling's, the AIOps mechanisms are the sibling's, the agent observability is the sibling's, the drift taxonomy is the sibling's. What is here is only the question of what the canon's objects become — and the answer, stated once more, is that they become themselves, measured honestly, with one more number to check and one more actor in the chain.

AI does not change what reliability is; it changes what can be measured, who is on call, and what counts as a control.
