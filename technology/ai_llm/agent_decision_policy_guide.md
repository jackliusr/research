# Agent Decision Policy — Ask, Recommend, Act or Wait, and Who Carries the Cost of Being Wrong

> **Byline:** Jack Liu Shurui, Solution Architect  
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Agent Posture  
> **Date:** October 2026 · **Sources read:** 2026-10-07

> **Standfirst.** Given a task and a situation, should an agent *ask*, *recommend*, *act*, or *wait* — and on what policy is that decided? This guide owns that question and nothing around it. The mechanisms an agent posture is built out of — sandboxes, permission-prompt economics, human-in-the-loop maps, guardrail ladders, risk registers, drift, silent failure — are already owned by sibling guides in this repository, and every one of them is **cited here by name and not restated**. What is unowned, and therefore what this guide holds, is the *decision policy itself*: the postures available, the variables that should drive the choice, the cost of getting each choice wrong, the form a written policy takes, and — for a bank — how a posture maps onto the institution's authority structure, where the decisive question is **who is permitted to commit the institution to what**. The four postures are **the requester's framing, not an established taxonomy**; this guide treats them as an organising frame, checks them against thirty years of prior literature on mixed-initiative interaction and the levels of automation, and says plainly where the mapping is imperfect. If the evidence supports three postures or six, an honest policy says so.

> **How to read it.** Section 1 defines the four postures, the decoder terms, and the thesis, and declares the boundary against the sibling guides by name. Section 2 works the four postures the same way — what each commits, what each costs, how each fails — and states the uncomfortable part: **waiting is not the safe default**. Section 3 lists the variables that should drive the choice, each stated as a decision input with its failure mode. Section 4 develops the reversibility-and-consequence matrix and the honest middle where most real actions live. Section 5 is the economics of interruption, cross-referenced to the harness guide rather than re-derived. Section 6 is the policy as an artefact, with a fill-in skeleton. Section 7 is the failure modes of each posture. Section 8 is the literature, date-and-kinded, mapped against the four postures with the imperfections stated. Section 9 is the banking and authority section, connecting to the register and governance frames rather than decorating with them. Sections 10–11 are measurement and current platform practice. Section 12 is the Cymbal Bank worked example. Sections 13–15 are the anti-patterns, the claims audit and the honest negatives. Section 16 is the glossary, cross-references and closing summary. Every source carries who, when, and of what kind; nothing is presented as a research finding when it is a vendor's permission-mode design.

> **Verification policy.** Facts marked **(verified)** were checked against a primary source during writing (October 2026) — a peer-reviewed paper, an established reference work, or a vendor's own product documentation. Everything else carries a label: **(vendor claim)** for a statement a vendor makes about its own product; **(flagged)** where a claim is repeated but not traceable to a measurement; **(unverified-flag)** where a page could not be extracted; **(negative finding)** where a search pass established that something does *not* appear in the sources checked. Where the literature predates LLM agents — as most of it does — the date and the kind are stated so that a 1978 or 1997 result is never presented as established about a 2026 agent. No citation in this guide is written from memory.

---

**Table of contents**

1. [Overview, the Four Postures, the Decoder, the Thesis, and the Boundary Declared](#1-overview-the-four-postures-the-decoder-the-thesis-and-the-boundary-declared)
2. [The Four Postures: Ask, Recommend, Act, Wait](#2-the-four-postures-ask-recommend-act-wait)
3. [The Variables That Drive the Choice](#3-the-variables-that-drive-the-choice)
4. [Reversibility and Consequence: The Matrix and the Middle](#4-reversibility-and-consequence-the-matrix-and-the-middle)
5. [The Economics of Interruption](#5-the-economics-of-interruption)
6. [The Policy as an Artefact](#6-the-policy-as-an-artefact)
7. [The Failure Modes of Each Posture](#7-the-failure-modes-of-each-posture)
8. [The Literature, Date-and-Kinded](#8-the-literature-date-and-kinded)
9. [Banking and Authority: Who May Commit the Institution to What](#9-banking-and-authority-who-may-commit-the-institution-to-what)
10. [Measurement: What to Instrument and What It Cannot Show](#10-measurement-what-to-instrument-and-what-it-cannot-show)
11. [Current Practice: How Today's Platforms Implement Postures](#11-current-practice-how-todays-platforms-implement-postures)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit: Verified, Flagged, Rejected](#14-the-claims-audit-verified-flagged-rejected)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary, Cross-References and Closing Summary](#16-glossary-cross-references-and-closing-summary)

---

## 1. Overview, the Four Postures, the Decoder, the Thesis, and the Boundary Declared

### 1.1 The question this guide owns

An agent is given a task. It has tools, a mandate of some kind, and a situation. Before it does anything with a consequence, one question must have an answer: **ask a human, recommend to a human, act on its own, or wait?** The agent posture is the answer to that question, and the *policy* is the rule — written down, ideally, and testable — that produces the answer from the situation.

This guide owns exactly that. It is deliberately not a guide to sandboxes, to harness internals, to human-in-the-loop design patterns, to guardrail ladders, to risk registers, or to drift. Every one of those is a mechanism an agent posture is built out of, and every one is owned elsewhere in this library; §1.5 names each and states, in one line, what this guide borrows from it. The edge this guide holds is the *choice itself* — the postures, the inputs to the choice, the cost of each error, the written form, and the authority mapping.

### 1.2 The four postures defined

Four postures are available to an agent at any decision point. They are defined here once, in the sense this guide uses them, and worked in §2.

| Posture | The agent's move | The human's role | The commitment made |
|---|---|---|---|
| **ASK** | The agent stops and puts a question to a human before proceeding: which of these, whether to, what value. | Supplies the decision the agent cannot or should not make. | None until the human answers; the agent has consumed a human's attention. |
| **RECOMMEND** | The agent produces a proposal — a draft, a classification, a prepared action — and hands it on for a human to accept, edit or reject. | Decides; the output is advisory only. | None; the artefact is a draft, not an effect. |
| **ACT** | The agent performs the action itself. | None in the moment; the human is downstream (audit, reversal, exception handling). | The full effect of the action, on the user's and the institution's behalf. |
| **WAIT** | The agent does nothing on this decision point — because it is uncertain, because a precondition is unmet, because it defers to a later moment or to another actor. | May never know the decision point existed. | None — but an action that should have happened did not, and the omission is a commitment of a kind. |

These four are the organising frame. They are **not** a settled taxonomy, and §8 shows where the literature instead offers a ten-point scale of levels (Sheridan & Verplank, 1978), four automated function *types* crossed with levels (Parasuraman, Sheridan & Wickens, 2000), and a decision-theoretic account of when an automated agent should act versus defer (Horvitz, 1999). The honest position is that "ask / recommend / act / wait" is a *usable axis*, close to but not identical with the decision-and-action-selection function that the human-factors literature has studied since the 1970s. Where the mapping is imperfect, this guide says so rather than manufacturing a canon around four words.

### 1.3 The decoder

Nine terms recur. Each is a policy object, not a metaphor.

| Term | Definition in this guide | Why it matters |
|---|---|---|
| **Posture** | One of the four dispositions — ask, recommend, act, wait — an agent takes at a decision point. | The unit a policy assigns. |
| **Ask** | Soliciting a human decision the agent cannot or should not make itself, before acting. | Costs human attention; the subject of §5. |
| **Recommendation** | A proposed artefact or action handed to a human to accept, edit or reject; advisory, non-binding. | Cheap to reverse, but only if a human actually reads it (§7). |
| **Action** | The agent effecting a change in the world on its own authority. | The only posture that creates an irreversible external fact. |
| **Wait** | Not acting at a decision point — through uncertainty, an unmet precondition, or deferral. | A *cost*, not a neutral default (§2.4, §7.4). |
| **Mandate** | The grant of authority the agent carries: what it may do, for whom, within what limits. | An action is only legitimate inside its mandate (§9). |
| **Approval gate** | A point at which a human must approve before the agent proceeds, or the agent must stop. | A control whose *position in the flow* is the real design choice (§9). |
| **Escalation** | The deliberate routing of a decision upward — to a human, or to a more senior authority — because the agent's mandate does not reach it. | The mechanism a policy uses to place the human where the risk is. |
| **Four-eyes control** | The maker-checker principle: for a given transaction, at least two individuals participate — one creates, another confirms (Wikipedia, *Maker-checker*, verified 2026-10-07). | In a bank, the question of who the *two eyes* are when one is an agent (§9). |

### 1.4 The thesis

**The posture is a choice about who carries the cost of being wrong.**

This is the load-bearing claim of the guide, and it is a reframing of the posture question away from safety and toward cost allocation:

- **ASK** places the cost of a wrong decision on the human who answers — but it charges a fee, in attention, whether the answer was needed or not.
- **RECOMMEND** places the cost on the human only if the human actually reads and accepts the proposal — which, empirically, they may not (§7.2).
- **ACT** places the cost of an error on whoever bears the consequence — the user, the institution, the third party — and on the agent's principal, who authorised the act.
- **WAIT** places the cost of the *omitted* action on the user who needed it, and it is the posture whose costs are least likely ever to be recorded (§7.4).

Read this way, a posture policy is not a safety dial. It is an **allocation of error-cost** across the agent, its user, its institution and its absent third parties. That is why the same action can warrant different postures for different principals (an AI engineer's sandbox versus a bank's payment rail), and why the deciding variables in §3 are all measures of who absorbs the cost when the thing goes wrong.

### 1.5 The boundary declared by name

This guide deliberately does **not** re-derive material owned by siblings. It cross-references by name and states, in one line each, what it borrows.

- **[agent_harness_engineering_guide.md](agent_harness_engineering_guide.md)** — owns the mechanism by which a posture is enforced and the permission-prompt economics: that without a sandbox every risky action needs a human permission prompt, and that at scale this yields either **prompt fatigue** (users abandon the agent) or **reflexive approval** (defeating the safety rationale); that a sandbox shifts permission "from an *action question* to a *session configuration*"; that introducing sandboxing to Claude Code reduced permission prompts by **84%** while preserving safety (a survey-reported vendor figure); that its ETCLOVG taxonomy makes **G — Governance** a first-class layer (permission enforcement, gateway controls, audit, human-approval workflows) and owns the **H3/H4 lifecycle hooks**; and that mobile-permission research found only **17%** of users attended to dialogs and **3%** understood them. §5 borrows this framing and cites these numbers as the sibling guide's, not as this guide's findings.
- **[multi_agent_banking_guide.md](multi_agent_banking_guide.md)** — owns the **Governed-Agent Design Principle** (scope, escalation, evidence and evaluation contracts) and the **Human-in-the-Loop Map** across four banking workflows. §9 cites its escalation contract and its override-as-metric discipline; it does not restate the map.
- **[../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md)** — owns the **controls** (preventive and detective) and **the evidence question** — the test that *a risk with no owner and no evidence is an observation, not a control*. §9 and §10 borrow that frame.
- **[../servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md)** — owns the **seven-rung Guardrail Ladder** and the vendor's **Supervised/Autonomous** skill-execution and **copilot/autopilot** agent-execution switches; **rung 4 is human approval at the write boundary**. §9 and §11 cite the ladder and rung 4 rather than re-deriving them.
- **[agent_experience_guide.md](agent_experience_guide.md)** — owns **silent failure on the interface side**: that a consumer which cannot do a thing does not file a report, it produces a plausible wrong answer instead. §7.4 borrows that analogy for the silence of the wait posture.
- **[ai_agent_drift_guide.md](ai_agent_drift_guide.md)** — owns **drift**: the degradation of an agent's behaviour as the environment and the model change under it. §6 and §7 cite drift as the reason a posture policy must be reviewed on a cadence, not written once.

The edge this guide owns is: **the policy that maps a task and a situation to a posture, and the cost accounting that makes the choice defensible.** No sibling states that as its subject.

---

## 2. The Four Postures: Ask, Recommend, Act, Wait

Each posture below is worked the same way — what the agent does, what it commits to, what it costs the user, what it costs the institution, and how it fails. The failures are catalogued again, cross-posture, in §7; here they are stated as the intrinsic shadow of each posture. The section ends with the honest statement the rest of the guide depends on: **the wait posture is not the safe default**.

### 2.1 ASK

**What the agent does.** It halts at a decision point and puts a question to a human — "which of these two accounts?", "confirm this amount?", "may I proceed?" — and does not continue until answered. The question may be blocking (the task cannot proceed) or interstitial (the agent proposes to proceed unless stopped within a window).

**What it commits to.** Nothing material, in principle. The commitment is to a *human's attention*: the ask consumes some of a person's finite capacity to attend, decide and answer. That commitment is real even though no external state changes.

**What it costs the user.** The interruption. Horvitz's *Principles of Mixed-Initiative User Interfaces* (CHI '99, verified) states the discipline as a principle, not an afterthought: a system uncertain about a user's intentions "should be able to engage in an efficient dialog with the user, **considering the costs of potentially bothering a user needlessly**" (principle 5). The cost is the user's time and focus, and it is paid whether or not the answer was needed.

**What it costs the institution.** A latent cost that only materialises at volume: if asks are frequent, the human answers them badly (§5, §7.1). An institution that put an approval gate on everything has not bought safety; it has bought a rubber-stamp, and a rubber-stamp is a control that has stopped operating while still appearing in the record.

**How it fails.** By being ignored, rubber-stamped, or routed around. The ask posture's failure mode is not "the agent asked and the human said no" — that is success. Its failure is "the agent asked and the human, who answers forty of these an hour, hit *allow* without reading", or "the user disabled the prompt because it was in the way". Both are documented in the harness guide's permission-prompt economics (§5).

### 2.2 RECOMMEND

**What the agent does.** It produces an advisory artefact — a classification, a drafted reply, a prepared transaction, a ranked set of options — and hands it to a human to accept, edit or reject. In the vocabulary of the platform vendors, this is the drafting/summarising/triaging case, and it is the case the ServiceNow guardrail ladder and the multi-agent banking HITL map both treat as the safest productive posture (§1.5).

**What it commits to.** The *proposal* is committed; the *effect* is not. The distinction is the whole value of the posture, and it is fragile, because it depends on a human actually making the decision the proposal defers.

**What it costs the user.** The review effort — reading the proposal and deciding. If the proposal is good, this is cheap; if it is bad or verbose, the user pays to discover that. The recommend posture pushes the decision cost onto the human but *reduces* the cost of producing the decision, which is the trade it exists to make.

**What it costs the institution.** A governance cost: the institution must maintain the human as the real author (§9) — which means the recommendation must not be silently accepted. The ServiceNow guide's rule that the committed author must be the human and that an auto-approval voids the artefact is the institution-facing expression of this cost.

**How it fails.** By being accepted unread. A recommendation that a human rubber-stamps has become an action with a human's name on it and no human judgement behind it — *an action by another name* (§7.2). Dietvorst, Simmons & Massey (*Algorithm Aversion*, J. Exp. Psychol. Gen. 144(1):114–126, 2015, verified) and the algorithm-aversion literature describe the *opposite* pull (people reject algorithmic advice after seeing it err), which makes the unread-acceptance failure the more insidious one: the same recommendation may be over-trusted in routine cases and under-trusted after one visible error, and neither response is the review the posture assumed.

### 2.3 ACT

**What the agent does.** It performs the action itself, on its own authority, within its mandate. This is the posture that produces an external fact: a payment moved, a record written, an email sent, a system changed.

**What it commits to.** The full effect. This is the only posture of the four that creates an irreversible external *something* — or, if the action happens to be reversible, the cost and latency of reversing it. Reversibility is the primary variable in §3 precisely because it governs how much this commitment can be taken back.

**What it costs the user.** The consequence of the action if it is wrong — money, disclosure, trust, a downstream state that must be unwound. The user's exposure is the reason the mandate exists: an action inside the mandate is one the principal has already priced.

**What it costs the institution.** Exposure on the two axes the risk register cares about: the *impact* of a wrong action (blast radius, §3) and the *evidential* cost of defending it after the fact (what the record shows, §9). The institution also bears the mandate cost: every action posture implies a grant of authority that must have been deliberate and recorded.

**How it fails.** By acting wrong irreversibly, or by acting unlogged. The first is the failure everyone anticipates; the second is the one that removes the institution's ability to explain what happened. The Cymbal Bank example (§12) is built around a class where the obvious act posture is wrong for exactly the evidential reason, not the risk-magnitude reason.

### 2.4 WAIT — and the honest statement about it

**What the agent does.** It does not act on the decision point. Wait is the posture of deliberate inaction: it may be waiting on a precondition (the beneficiary is unresolved), on a human (escalating and then yielding to the queue), on a later moment (deferring to a scheduled re-evaluation), or it may simply be the residue of uncertainty — the agent neither asked, recommended, nor acted.

**What it commits to.** Technically, nothing. In practice, it commits to the *absence* of an action that the task may have required. The commitment is invisible, which is the problem.

**What it costs the user.** The action that did not happen — the fraud not flagged in time, the payment not sent, the ticket not resolved — and the derivative cost of the user's own wasted time working around an agent that did nothing and said nothing.

**What it costs the institution.** The same, at scale, plus the loss of the value the agent was deployed to create. It also costs the institution *epistemically*: because the wait leaves no failed transaction, no ticket and no complaint, its cost is systematically under-measured (§10).

**How it fails.** By silent inaction, by stale waiting (deferred so long the moment passed), and by the unrecorded opportunity cost. These are §7.4.

**The honest statement.** *Wait is not the safe default.* There is a strong pull to treat waiting as the conservative choice — "when in doubt, do nothing" — but that treatment mistakes *risk of commission* for *all risk*. Waiting transfers the cost of the decision onto the user who needed the action, and it does so without a record, because an omission that produced no error and no complaint is not a signal any instrument in §10 naturally captures. The harness guide's sibling framing of **silent failure** ([agent_experience_guide.md](agent_experience_guide.md)) is exact here: a human who cannot get what they needed complains; an agent — or an agent's omission — is silent in the same way a broken interface is silent, with no error, no report and no complaint. A policy that defaults to wait under uncertainty is not the cautious policy. It is the policy that hides its own costs.

---
## 3. The Variables That Drive the Choice

A posture policy is only as good as the inputs it reads. This section states the variables that *should* drive the choice, each as a **decision input** with the posture it pushes toward and — the part most policies omit — its **failure mode**: the way the variable is wrong, mis-measured, or gamed when it is used as a rule. Six variables are developed; they are not the only ones, but a policy that ignores any of these six is choosing a posture on incomplete grounds.

### 3.1 Reversibility, and whether an undo exists

**The input.** Can the action be taken back, and at what cost? Three states matter: *fully reversible at trivial cost* (a draft, a read), *reversible at material cost or latency* (a payment recall, a sent email, a record edit that must itself be audited), and *irreversible* (a disclosure, a regulatory filing, a settlement, a message to a customer). The variable is not "is it reversible in theory" but "does an *undo path exist that someone can actually execute*".

**What it pushes toward.** The more reversible and the cheaper the undo, the more the policy can tolerate ACT; the more irreversible, the more it should require RECOMMEND or ASK. This is the horizontal axis of §4.

**Failure mode.** *Claimed reversibility.* An action is treated as reversible because a compensating action exists on paper, while in practice the undo is slow, partial, or owned by a team that is not on the path. A refund is "reversible" until it has settled; a sent customer message is "correctable" until the customer has read it. A policy that reads reversibility off the existence of an undo rather than off the *cost and latency* of executing it will over-authorise ACT. This is also why idempotency matters to posture (not to avoid duplication, which is the harness's concern, but because a reversible action with an unreliable repeat-semantics is not reliably reversible at all).

### 3.2 Blast radius, and whether it accumulates

**The input.** How much does one wrong action touch, and does the effect *accumulate* across repeated actions? A single mis-classified support ticket has a small blast radius; a rate or limit change on a shared system has a large one. Crucially, some effects are *non-accumulating* (each error is bounded and independent) and some *accumulate* (a hundred small errors compose into one large state change — the "one wrong CI versus a hundred" distinction the ServiceNow guardrail ladder's **rung 5** owns, §1.5).

**What it pushes toward.** Large or accumulating blast radius pushes toward RECOMMEND or ASK, and toward rate/volume limits that bound the damage before a human notices. Small, non-accumulating blast radius tolerates ACT.

**Failure mode.** *Radius misjudged because it is not local.* The action looks small from the agent's vantage point — one write — while its blast radius is a downstream consumer of that write (a risk engine, a metric, a chain of approvals) that the agent cannot see. The ServiceNow guide makes this concrete for the CMDB: a configuration-item write is "a state-of-the-world claim consumed by impact analysis and change risk assessment, with a silent propagation path". The variable must be evaluated against *who consumes the effect*, not against the size of the call.

### 3.3 Sensitivity of the data, and authority of the recipient

**The input.** Two coupled questions: how sensitive is the information involved, and *who is on the other end* — what authority, entitlement or relationship does the recipient of the output hold? A draft sent to an internal approver and the same draft sent to a customer are different actions on the disclosure axis even though the data is identical.

**What it pushes toward.** High sensitivity plus an external or unverified recipient pushes hard toward RECOMMEND or ASK — and, in a bank, toward the authority question of §9 rather than the posture question, because the constraint is entitlement, not confidence. Low-sensitivity or internal-authorised reads tolerate ACT.

**Failure mode.** *Conflating sensitivity with posture.* A policy that says "sensitive data ⇒ ask a human" mis-locates the control: the requirement is that the *recipient is entitled*, which is enforced by access control, not by an approval prompt. Sensitivity is not itself a posture driver; the *combination* of sensitivity and an unauthorised-but-plausible recipient is. Treating sensitivity alone as the trigger produces prompts that a human cannot meaningfully answer ("is it OK to show this account to an agent that just fetched it?") and that therefore train reflexive approval (§5).

### 3.4 The cost of interrupting a human versus the cost of not interrupting

**The input.** The economics of the ask itself. Every ask has a price — the human's attention, latency, the risk of a reflexive answer — and every *avoided* ask has a price too — the cost of the agent acting without the human. Horvitz (CHI '99, verified) frames this as principle 4: an agent should infer "ideal action in light of costs, benefits, and uncertainties", "guiding their invocation with a consideration of the expected value of taking actions", and (principle 3) should "consider the costs and benefits of deferring action to a time when action will be less distracting".

**What it pushes toward.** When the expected cost of the wrong action is high *relative to* the cost of an interruption, ASK or RECOMMEND; when the interruption is expensive and the action's downside is small and reversible, ACT. This is the variable that makes posture a *decision-theoretic* choice rather than a rule-following one.

**Failure mode.** *Asymmetric accounting.* Teams estimate the cost of the wrong action carefully and the cost of the interruption at zero — an interruption is "just a click". The harness guide's numbers are the antidote: mobile-permission research found only **17%** of users attended to dialogs and **3%** understood them (survey-reported, cited from [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md)), which is not a small cost; it is the cost of a control that is not operating. A policy that prices only one side of the trade will systematically over-ask.

### 3.5 Frequency and repetition

**The input.** How often does this decision point recur? A once-a-quarter decision can afford a human in the loop on every instance; a thousand-times-an-hour decision cannot, and should not try. Frequency governs whether an approval is *feasible as a control* at all, because a per-instance human gate on a high-frequency action is the exact mechanism that produces fatigue (§5). Frequency also interacts with accumulation (§3.2): a high-frequency action with accumulating blast radius is the dangerous combination.

**What it pushes toward.** Low frequency supports per-instance ASK/RECOMMEND. High frequency pushes toward *class-level* pre-authorisation (§5.3) — the human approves a *pattern*, not each instance — and toward ACT inside that pre-authorised class.

**Failure mode.** *The frequency–gate mismatch.* A policy designed for a low-frequency decision (per-instance approval) is applied to a high-frequency one, and the human, unable to sustain the attention, degrades to reflexive approval. The mismatch is invisible in the gate's own logs, which record "approved" every time. It shows up only in the outcome metrics of §10 and in incidents.

### 3.6 The reliability of any confidence signal the agent carries

**The input.** Does the agent have a confidence estimate, and *can it be trusted*? This is the trap of §3 and HAZARD 3(a): it is tempting to write a policy of the form "ask when the agent is unsure", which requires the agent's self-reported certainty to be a usable input. The calibration literature says it is not, on its own:

- Kadavath et al. (*Language Models (Mostly) Know What They Know*, arXiv:2207.05221, 2022-07-11, verified) find models "reasonably calibrated on diverse multiple choice and true/false questions when they are provided in the right format", with self-evaluation via "P(True)" showing "encouraging performance, calibration, and scaling" — but the same paper's broadly-reported companion finding is that **RLHF-trained models are worse calibrated** than their base counterparts: the training that makes a model helpful can push its stated confidence away from its accuracy.
- Lin, Hilton & Evans (*Teaching Models to Express Their Uncertainty in Words*, arXiv:2205.14334, 2022-05-28, verified) show a GPT-3 model *can* express calibrated uncertainty in words, mapping "90% confidence" to a well-calibrated probability, and remaining "moderately calibrated under distribution shift".
- Xiong et al. (*Can LLMs Express Their Uncertainty?*, arXiv:2306.13063, 2023-06-22; ICLR 2024, verified) find the opposite tendency at the frontier: "LLMs, when verbalizing their confidence, tend to be **overconfident**, potentially imitating human patterns of expressing confidence", and that "none of these techniques consistently outperform others, and all investigated methods struggle in challenging tasks, such as those requiring professional knowledge".

**What it pushes toward.** If confidence is used at all, it must be *calibrated and validated for this model, this task and this date*, and it should *supplement* a structural variable (reversibility, blast radius) rather than carry the policy alone. The cleanest policies state the posture by action class (§6) and use confidence only as an escalation trigger *within* a class.

**Failure mode.** *The confidence signal as the whole policy.* A policy of "ask when unsure" asks the wrong component: a model's stated certainty is poorly calibrated and, worse, its mis-calibration is systematically *non-random* (overconfident on hard tasks — exactly the tasks where asking matters). Such a policy is most confident, and therefore most likely to ACT, precisely where the agent is least reliable. Worse still, *verbalized* confidence is manipulable by the phrasing of the ask, so a policy that keys on it is a policy the model can, accidentally, talk itself around. Confidence is an input to posture *only after* it has been shown calibrated on the deployment's own tasks — and the burden of that proof is on the deployer, not on the model.

### 3.7 The six variables, summarised

| # | Variable | Pushes toward ACT when… | Pushes toward ASK/RECOMMEND when… | The failure mode |
|---|---|---|---|---|
| 3.1 | Reversibility | Cheap, executable undo exists | Action is irreversible or undo is slow/partial | Claimed reversibility |
| 3.2 | Blast radius | Small and non-accumulating | Large or accumulating; consumers downstream | Radius misjudged as local |
| 3.3 | Data sensitivity + recipient authority | Recipient entitlement verified, low sensitivity | Sensitive AND recipient authority unproven | Sensitivity conflated with posture |
| 3.4 | Interruption economics | Interrupt cost > expected error cost | Expected error cost > interrupt cost | Interrupt cost priced at zero |
| 3.5 | Frequency | High-frequency, pre-authorised class | Low-frequency, per-instance feasible | Frequency–gate mismatch |
| 3.6 | Confidence signal reliability | (Never on its own) | Validated calibration exists for this task/model | Confidence signal as the whole policy |

---

## 4. Reversibility and Consequence: The Matrix and the Middle

### 4.1 The two axes and the four quadrants

The two variables that dominate the choice are **reversibility** (§3.1) and **consequence** — the impact if the action is wrong, which is blast radius (§3.2) compounded by sensitivity and recipient (§3.3). Plotted, they give the familiar four quadrants:

| | **Low consequence** | **High consequence** |
|---|---|---|
| **Reversible** | **ACT** — the sandbox quadrant. Cheap to undo, small blast radius: let the agent act and unwind if wrong. This is the region a sandbox exists to create. | **ACT with a bound** — act inside limits, with monitoring and a fast reversal path. High-consequence-but-reversible is the region where rate limits and rollback are the controls, not prompts. |
| **Irreversible** | **RECOMMEND** — small impact but no undo: let the agent prepare, let a human commit. The cost of a wrong irreversible act is permanent even when small. | **ASK / escalation** — irreversible *and* high consequence is the quadrant where the human must decide before the act, or the act must not happen without a mandate that explicitly reaches it. |

### 4.2 Why the quadrant is necessary and not sufficient

The matrix is a *first cut*, and it is honest only if its limits are stated. Three of them:

1. **Consequence is not one number.** Blast radius, data sensitivity and recipient authority weigh differently for different institutions and different action classes. A "high consequence" cell for a payments agent and for an internal-tools agent mean different things.
2. **Reversibility is a property of the environment, not the action type.** The same action is reversible in a test environment and irreversible in production; the policy must bind the posture to the environment as well as the action class.
3. **Posture is not the only control.** The matrix picks the *posture*; the sandbox, the rate limit, the identity and the audit trail (the harness's E and G layers, and the ServiceNow ladder's rungs 2–6) are separate controls that apply *within* a posture. ACT in the reversible/low-consequence quadrant is safe because of the sandbox and the bound, not because ACT is intrinsically safe.

### 4.3 The honest note: most real actions are neither fully reversible nor fully consequential

The clean 2×2 misleads because **most real actions fall in the middle**, on both axes, and a policy that only handles the corners will mis-handle the centre — which is where the volume is.

- **Partial reversibility.** Most actions are reversible *up to a point and at a cost*: a payment can be recalled until it settles; a record can be corrected but the correction is itself a recorded event; a message can be followed by a correction but not unsent from the recipient's memory. The policy must handle a *graded* reversibility, not a binary, and it must name the threshold at which the action crosses from recoverable to committed.
- **Partial consequence.** Most actions have an impact that is neither negligible nor catastrophic, and that depends on *accumulation* (§3.2). A single borderline AML disposition is low-consequence; the same disposition repeated across a queue is not. The policy must handle consequence that is a function of *rate and repetition*, not just of the single call.

### 4.4 The middle: what a policy does where the corners do not apply

Four rules let a policy operate in the middle rather than pretending the middle is a corner:

1. **Bind the posture to the class, and the class to the worst plausible instance in scope.** Do not assign a posture per call (the call cannot know its own blast radius reliably). Assign it per *action class*, sized to the highest-consequence member of that class, and then use rate limits to keep the class from reaching that worst case silently.
2. **Put the gate at the moment of irreversibility, not at the moment of intent.** For a partially reversible action, the human belongs at the threshold where the action becomes hard to unwind — e.g. a *prepared* payment (reversible, no money moved) versus a *submitted* payment (irreversible once settled). This is the pattern the agent-experience guide's sibling payment example uses (prepare → submit, with the write boundary at submit), and it exists precisely to place the gate where the cost changes.
3. **Make accumulation visible to the policy, not just to the dashboard.** A posture that is right for one instance may be wrong for the hundredth. The policy needs a volume signal — a rate counter, a daily cap, a class-level budget — so that ACT is ACT-*with-a-ceiling* where accumulation is the risk.
4. **Record the middle honestly.** For actions in the middle, the record must show the *actual* reversibility state at the time of the act (was it still recallable?), because that is what a reviewer or an auditor will need and it is not recoverable from the outcome alone (§9).

The middle is not a defect in the matrix. It is the honest picture of where policies are actually exercised, and a policy that cannot describe it is a policy that will be silently overridden the first time a real action is neither clearly safe nor clearly dangerous.

### 4.5 The matrix, stated as the thesis

The matrix is the thesis drawn: each cell is a statement about **who carries the cost of being wrong**. The reversible/small cell puts the cost on the *agent and its sandbox* (cheap). The irreversible/large cell puts the cost on the *institution and the user* and therefore demands that the cost be carried deliberately, by a mandate or a human — not discovered after the fact. The middle is where the cost is *shared and shifting*, which is why the middle needs the explicit gate and the explicit record, and why posture policies that only describe the corners are policies for a world that does not occur.

---

## 5. The Economics of Interruption

This section is the economics of the *ask*. The mechanism — what a permission prompt is, and why its economics fail at scale — is **owned by [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md)** and is cited here, not re-derived. What this section adds is the posture-policy reading of that mechanism: what an approval costs, why approvals stop being read, why an approval of a *class* differs from an approval of an *instance*, and what the design of the ask changes.

### 5.1 What the harness guide establishes (cited, not restated)

From the sibling, cited by name and carried forward:

- Without a sandbox, "every risky action needs a human permission prompt; at scale this yields either **prompt fatigue** (users abandon the agent) or **reflexive approval** (defeating the safety rationale)".
- A sandbox "defines a bounded region in which the agent is authorized to act freely — shifting permission from an *action question* to a *session configuration*".
- Introducing sandboxing to Claude Code reduced permission prompts by **84%** while preserving safety — a **survey-reported vendor figure**.
- Mobile-permission research found only **17%** of users attended to dialogs and **3%** understood them.
- The harness's **ETCLOVG** taxonomy makes **G — Governance** a first-class layer for permission enforcement, gateway controls, audit and human-approval workflows, and owns the **H3/H4 lifecycle hooks**.

The posture-policy consequence is direct: **an approval is a control whose effectiveness decays with the frequency of its use.** A permission prompt is not a fixed-cost safety feature; it is a per-interruption consumer of a scarce resource (human attention), and its protection depreciates as the human answers more of them.

### 5.2 What an approval costs

An approval costs four things, only the first of which is usually counted:

1. **The human's time and attention** — the direct cost, and the one teams price.
2. **Latency** — the task waits on a human, which for any agent whose value is throughput is the whole point lost.
3. **The risk of a *wrong* answer** — a human answering forty prompts an hour produces both false-allow (rubber-stamp) and false-deny (blocks a correct action). The harness guide's 17%/3% figures are the evidence that the human is not, in fact, delivering the review the control assumed.
4. **The training effect on the human** — each reflexive approval makes the *next* one more reflexive. Approval fatigue compounds; it does not reset between prompts.

### 5.3 Why approvals stop being read

An approval stops being read for structural reasons, not because the human is careless:

- **Uniformity.** If every prompt looks the same and most are benign (they are, or the agent would not be deployed), the human learns that the expected answer is "allow". That prior is rational on the evidence and wrong on the forty-first prompt.
- **Cost–benefit drift.** The cost of reading each prompt is constant; the *perceived* benefit falls as the hit rate of genuine danger falls. The rational human stops reading long before the dangerous prompt arrives.
- **No feedback.** A rubber-stamped approval that turned out fine produces no signal; a rubber-stamp that turned out badly produces an incident the human may never connect to the moment of approval. The control degrades without a feedback loop.

This is why **the number of prompts is itself a risk metric**, not a comfort: a system that asks a great deal is not a well-controlled system, it is a system whose control has been diluted to the point of theatre.

### 5.4 Approving a class once versus approving an instance every time

The corrective move is to change *what* is approved, not merely how often. Three patterns, in increasing autonomy:

| Pattern | What the human approves | When it fits | The trade |
|---|---|---|---|
| **Per-instance approval** | Each individual action, before it happens | Low-frequency, high-consequence, irreversible actions | Survives only at low volume; degrades to reflexive approval above it |
| **Pre-authorised class** | A *pattern* of actions — a class, a limit, a counterparty set, a threshold — once, in advance | High-frequency actions with bounded blast radius | Amortises the human's attention across many actions; requires the class boundary to be precise and the volume bound to hold |
| **Session-configuration approval** | The sandbox and the mandate under which the agent runs | Long tasks, whole sessions | Moves permission "from an action question to a session configuration" (harness guide); the human approves the *environment*, and the agent acts freely inside it |

The class approval is the pivot. It is how a bank with a payments agent can keep a human meaningfully in the loop without asking per payment: the customer or the operator pre-authorises a *limit and a beneficiary set*, and the agent acts inside it. This is also where the mandate question of §9 attaches — a pre-authorised class *is* a mandate, and its boundaries are what the institution is committing.

### 5.5 What the design of the ask changes

Whether an approval is read depends on how it is designed, and the design levers are policy levers:

- **The ask must carry the evidence needed to answer it.** A prompt that says "approve this payment?" trains rubber-stamping; a prompt that shows the amount, the beneficiary, the reason, and what changes if the human says no, invites an actual decision. This is the policy analogue of the tool-description discipline in the agent-experience guide.
- **Asks must be rare enough to be read.** Rarity is a design target, and it is bought by the sandbox and the class pre-authorisation, not by the human trying harder.
- **The default answer must be the safe one, and it must be honest.** A prompt whose default is "allow" trains allow; a prompt whose default is "deny" fails safe but becomes an obstacle the user routes around. The posture-policy rule is that the default must match the consequence — deny by default where the action is irreversible and consequential, allow by default where it is not.
- **The ask must be positioned at the moment of irreversibility, not at the moment of intent** (§4.4): asking before a reversible preparation step spends attention where a cheap undo exists; asking at the commit step spends it where it matters.

### 5.6 The economics, stated as the thesis

An interruption is a payment, in a currency the institution is spending down. Every ask draws on the human's finite attention; the art of the policy is to spend that attention where the cost-of-being-wrong is highest and the undo is weakest, and to buy the right to ACT elsewhere by pre-authorising classes and configuring a sandbox rather than by asking. Horvitz's decision-theoretic framing (CHI '99, verified) is the prescription, and the harness guide's prompt-fatigue and reflexive-approval findings are what happens when it is ignored.

---

## 6. The Policy as an Artefact

A posture policy that is not written down is not a policy; it is a habit distributed across an agent's prompt and its permissions, and it will drift (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md)). This section states what a written agent decision policy must contain, then gives a fill-in skeleton.

### 6.1 What a written policy must state

Six things, minimum. Each is load-bearing; a policy missing any one of them leaves the corresponding risk unmanaged.

1. **Action classes.** The enumeration of the things the agent can do, grouped into classes by *consequence and reversibility*, not by tool name. The class is the unit to which a posture is assigned (§4.4 rule 1). An agent's tool list is not a policy; the *classes* those tools fall into are.
2. **Posture per class.** For each class, one of the four postures (§2), with the specific form: *ask* (blocking/interstitial, and what the ask shows), *recommend* (what artefact is produced and who commits), *act* (within what mandate and limits), *wait* (waiting on what, until when, and what ends the wait).
3. **Limits.** The bounds inside which ACT is permitted: monetary, volumetric, rate, counterparty, environment. Limits are what keep a mis-scoped class from accumulating (§3.2) and what make a mandate concrete (§9).
4. **Escalation path.** For each class, the conditions under which the posture steps up (a limit exceeded, a confidence band crossed, a new counterparty, an unmatched record) and *to whom* — the specific role, not "a human". This is the escalation contract the multi-agent banking guide owns; the posture policy states the triggers and the target.
5. **Evidence.** What the record must show for each posture: for ACT, the input, the decision, the mandate invoked and the reversibility state at commit; for RECOMMEND, the proposal and the human's commit; for ASK, the question and the answer; for WAIT, the fact of the wait and its reason. Evidence is the register's frame (§1.5) applied per posture.
6. **Review cadence.** When the policy is re-examined, and against what evidence. The policy is a dated artefact: models change, tools change, the environment drifts, and a posture that was right at authoring may be wrong a model-version later (§8, drift guide).

### 6.2 The fill-in skeleton

The skeleton below is a *form of words*, deliberately plain, to be completed per agent. It is not a template to be adopted wholesale; it is the minimum structure a policy needs to be reviewable.

```text
AGENT DECISION POLICY — <agent name> — version <v>, dated <YYYY-MM-DD>
Owner: <named role>        Reviewed-by: <named role>        Next review: <date>

1. MANDATE
   Principal: <who the agent acts for>
   Scope: <the task and system boundary>
   Authority ceiling: <the maximum commitment the agent may make>
   Identity: <the agent's named identity; one per purpose>

2. ACTION CLASSES  (class | consequence | reversibility | posture | limits | escalation)
   <class A> | <low/high> | <reversible-costly/irreversible> | <posture> | <limits> | <trigger→role>
   <class B> | ...

3. POSTURE SPECIFICATION  (per class)
   ASK:      <blocking? what the ask shows? default answer?>
   RECOMMEND:<artefact produced? who commits? commit author must be human?>
   ACT:      <inside which limits? sandbox? rate cap? rollback path?>
   WAIT:     <waiting on what? timeout? what ends the wait? who is told?>

4. ESCALATION
   Step-up triggers: <limit exceeded / confidence band / new counterparty / ...>
   Target roles:     <role per trigger>
   Human position:   <where in the flow the human stands, and the rationale>

5. EVIDENCE  (per posture) — what the record shows afterwards
   <field list per posture>

6. FAILURE REVIEW
   Ask rate / approval-without-review rate / override rate / incident rate / wait rate (§10)
   Re-audit on: <model change | tool change | description change | cadence>
```

### 6.3 Why the artefact matters more than the words

The artefact matters because it is *checkable*. A posture policy that lives in the agent's prompt cannot be reviewed, cannot be diffed when a model changes, and cannot be shown to an auditor. The written policy is what turns a habit into a control — and per the register's own test, "a risk with no owner and no evidence is an observation, not a control". The policy names the owner (§6.2 field 1) and the evidence (§6.2 field 5); without both, the posture assignment is an observation about how the agent happens to behave today.

---
## 7. The Failure Modes of Each Posture

Each posture has a characteristic way of failing that is *not* the failure the posture was designed to prevent. The ask posture fails when the human answers without thinking; the recommend posture fails when the human does not answer at all; the act posture fails when the act is wrong or unrecorded; the wait posture fails silently, and its failure is the hardest to see. This section states them posture by posture, then draws the through-line.

### 7.1 The failure modes of ASK

- **Fatigue.** The human, facing a stream of asks, rations their attention and eventually stops reading. This is the harness guide's **prompt fatigue** (§5.1, cited), and the 17%/3% attention figures are its measurement.
- **Reflexive approval.** Distinct from fatigue and worse: the human keeps answering but the answer has become a reflex. The gate still logs "approved" every time, so from the record's point of view the control is operating perfectly — which is exactly what makes it dangerous. This is the harness guide's **reflexive approval**, and it is the reason a high ask rate is a risk metric, not a sign of control.
- **The unusable agent.** Enough asks and the agent's value proposition collapses: an agent that asks about everything is slower than doing the task manually, and the user disables it or routes around it. The failure looks like low adoption; it is actually the ask posture mis-priced (the interruption cost exceeded the error cost, §3.4).

**Guardrail.** Make the ask rare enough to be read: sandboxes and class pre-authorisation (§5.4) rather than more prompts; design the ask to carry the evidence to answer it (§5.5); instrument the *approval-without-review* rate, not just the approval rate (§10).

### 7.2 The failure modes of RECOMMEND

- **The unread recommendation — an action by another name.** If a recommendation is accepted without review, the effect is identical to ACT, but with a human's name on it and no human judgement behind it. The recommendation *thought* it had a human in the loop; it had a signature. This is the most under-detected failure of the recommend posture because it produces a *correct-looking record*: proposal, approver, timestamp.
- **Anchoring.** A recommendation is not neutral: it frames the decision. A human shown one proposed classification and asked "is this right?" reviews differently from a human asked to classify from scratch. The literature on automation bias (Skitka, Mosier & Burdick, 1999, verified; Lee & See, 2004, verified) documents the pull toward accepting an automated suggestion and discounting contradicting evidence — the errors of commission and omission the automation-bias reference work describes. A recommend posture that does not account for anchoring has outsourced the decision while believing it kept it.

**Guardrail.** The recommendation must be *genuinely reviewable*: it must show what would change if rejected, and the human's review must be measured (override rate, §10). The multi-agent banking guide's rule that a persistently *high* override rate is a defect in the agent, while a *near-zero* override rate is a defect in the reviewer, is the discipline this posture needs (cited by name).

### 7.3 The failure modes of ACT

- **The irreversible mistake.** The action happens, it is wrong, and it cannot be taken back (or can be taken back only at material cost). This is the failure the whole posture framework is built to bound — but note that the bound comes from the sandbox and the limits, not from the posture label, and the harness guide owns those mechanisms.
- **The unlogged action.** The act succeeds but the record does not show what happened, on whose authority, or against which mandate. The institution can neither explain the effect nor defend it, and the register's test fails: there is an effect with no evidence. Unlogged action is the failure that surfaces only under audit — and, like silent failure, it is invisible to any dashboard that measures task success.

**Guardrail.** Posture-appropriate evidence (§6.1 item 5); a named identity so the action is attributable (ServiceNow ladder **rung 3**, cited); logging that preserves the reasoning trace alongside the record (rung 6, cited); and a reversal path whose *executability* (not existence) has been tested.

### 7.4 The failure modes of WAIT

- **Silent inaction.** The agent waits, the action it should have taken does not happen, and no signal is produced. There is no error, no ticket and no complaint — the same silence the agent-experience guide attributes to a consumer that cannot use a surface ([agent_experience_guide.md](agent_experience_guide.md), cited). The wait posture is silent in exactly that way, and this is why it is *not* the safe default (§2.4): its failures are invisible by construction.
- **Stale waiting.** The wait had a reason — an unmet precondition, an unresolved dependency — but the reason lapsed and the agent did not re-engage. The agent is waiting on a condition that is no longer true, and nothing re-evaluates it. A wait without a timeout is a wait forever.
- **The opportunity cost never recorded.** The action the agent did not take had a value — the fraud it would have caught, the customer it would have served, the payment it would have moved. That value is never booked, because nothing records an omission. The measurement instruments of §10 must therefore include a *wait rate* and a *wait-with-reason* log, precisely because the normal instruments cannot see this failure.

**Guardrail.** Every wait must be *reified*: logged with a reason, given a timeout, and re-evaluated at the timeout; the wait rate must be instrumented; and the policy must state, per class, when waiting is the *intended* posture (a scheduled re-check, a deliberate deferral) versus when it is the residue of an unresolved uncertainty that should have escalated to ASK or RECOMMEND.

### 7.5 The through-line: every posture fails quietly

The four postures fail in different ways, but the failures share one property: **none of them is loud.** Fatigue and reflexive approval look like "approved". An unread recommendation looks like "reviewed". An irreversible act looks like "done". A wait looks like "nothing to see". In every case the record of the moment is consistent with success, and the failure is only visible in the outcome metrics and the incidents — which is why §10 instruments the *rates* (approval-without-review, override, wait) and not just the outcomes, and why the register's evidence question is the discipline that turns each of these from an invisible habit into a checkable control.

---

## 8. The Literature, Date-and-Kinded

The question "should an agent act, ask, or defer?" has a thirty-year human-factors prior life and a short LLM-era one. This section separates them, dates and kinds every source, and maps each to the four postures — stating where the mapping is imperfect rather than implying a canon around four words.

### 8.1 The decision-theoretic foundation (mixed-initiative)

**Horvitz, E., *Principles of Mixed-Initiative User Interfaces*, CHI '99 (ACM), DOI 10.1145/302979.303030.** **(verified — peer-reviewed conference paper, 1999; PDF read at erichorvitz.com/ftp/chi99horvitz.pdf, 2026-10-07.)** This is the foundational source for the economics-of-interruption reading of posture, and it is *not* a fixed enumeration. Horvitz gives **twelve principles** for coupling automated services with direct manipulation, and the ones that bear on posture are decision-theoretic:

- Principle **3**, on the timing of services: agents "should employ models of the attention of users and consider the costs and benefits of deferring action to a time when action will be less distracting".
- Principle **4**, on inferring the ideal action: automated actions "taken under uncertainty in a user's goals and attention are associated with context-dependent costs and benefits", and their value "can be enhanced by guiding their invocation with a consideration of the expected value of taking actions".
- Principle **5**, on dialog: a system uncertain about intentions "should be able to engage in an efficient dialog with the user, considering the costs of potentially bothering a user needlessly".
- Principle **7**, on poor guesses: designs should minimise "the cost of poor guesses about action and timing".

The testbed was **LookOut**, a scheduling assistant for Outlook that parsed email into proposed appointments and let the user edit the guess. Horvitz's framework *derives* when to act versus defer from expected cost and value; it does **not** enumerate four postures. This guide's ASK/RECOMMEND/ACT/WAIT are a *specialisation* of the act-versus-dialog-versus-defer decision that Horvitz frames — and the specialisation loses the decision-theoretic generality, which is why §5 keeps the cost calculus rather than hard-coding a posture per word.

### 8.2 The levels-of-automation line

**Sheridan, T.B. & Verplank, W.L., *Human and Computer Control of Undersea Teleoperators*, MIT, 1978, DTIC ADA057655.** **(verified via DTIC/NTIS listings — technical report, 1978.)** Introduces the **ten levels of automation** (from "human does everything" to "computer does everything, ignoring the human"), the scale on which "recommend" and "act" sit as *points*, not categories.

**Parasuraman, R., Sheridan, T.B. & Wickens, C.D., *A Model for Types and Levels of Human Interaction with Automation*, IEEE Trans. SMC-A 30(3):286–297, 2000, DOI 10.1109/3468.844354.** **(verified — peer-reviewed journal, 2000; full text read this pass.)** Refines the scale by crossing it with **four types of function**: information acquisition, information analysis, decision and action selection, and action implementation. Within *decision and action selection* the paper's ten-point scale is worked concretely: at a low level "several options are provided to the human, but the system has no further say in which decision is chosen"; at level 4 "the computer suggests one decision alternative, but the human retains authority for executing that alternative or choosing another one"; and at a higher level 6 "the system gives the human only a limited time for a veto before carrying out the decision choice". The paper also names the evaluative criteria that a posture policy is really trading on: primary criteria are "human performance consequences", and secondary criteria include **"automation reliability and the costs of decision/action consequences"**.

**Mapping, and its imperfections.** The four postures map onto this line as follows, and the mapping is **imperfect in both directions**:

| This guide's posture | Nearest level-of-automation reading | Where the mapping breaks |
|---|---|---|
| **RECOMMEND** | The "computer suggests one alternative, human retains authority" band (P/S/W level 4; Sheridan–Verplank mid-scale). | The literature treats "recommend" as *one point*; this guide treats it as a posture with its own failure modes (anchoring, unread acceptance, §7.2) that the level does not address. |
| **ASK** | The dialog/deferral band, and the "limited time for a veto" band (P/S/W level 6) for interstitial asks. | Horvitz's dialog principle is about resolving *the system's* uncertainty; ASK in this guide also covers *authority* asks ("may I?") that are not about the agent's uncertainty at all. |
| **ACT** | The upper levels (fully automatic, human bypassed). | The level says *how automatic*; it does not say *who carries the error cost*, which is this guide's thesis. |
| **WAIT** | The lowest effective-automation state, or "automation off". | The levels describe inaction as *human does it*; they do not model the *agent silently omitting* an action, which is the wait posture's real failure (§7.4). |

The honest conclusion: the four postures are **not** the levels of automation relabelled, and the levels are **not** richer than four postures in general. They are different decompositions — the levels measure *degree of autonomy*; the postures measure *who decides and who carries the cost* — and a serious policy needs both: a level (how automatic) and a posture (whose decision, whose cost).

### 8.3 Trust, reliance, and the human-side failure modes

A posture exists to place a human in a flow, so the posture's success depends on how humans actually behave with automation — a body of work that predates LLM agents by decades:

- **Parasuraman, R. & Riley, V., *Humans and Automation: Use, Misuse, Disuse, Abuse*, Human Factors, 1997, DOI 10.1518/001872097778543886.** **(verified — peer-reviewed journal, 1997; metadata resolved via Semantic Scholar, 2026-10-07.)** The taxonomy of human–automation interaction: *use*, *misuse* (over-reliance, failing to monitor), *disuse* (ignoring or disabling the automation) and *abuse* (automation designed without regard for the human). Misuse is the RECOMMEND posture's unread-acceptance; disuse is the ASK posture's "unusable agent"; both are named here, decades before agentic deployment.
- **Skitka, L., Mosier, K. & Burdick, M., *Does automation bias decision-making?*, Int. J. Human-Computer Studies, 1999, DOI 10.1006/ijhc.1999.0252.** **(verified — peer-reviewed journal, 1999; metadata resolved via Semantic Scholar, 2026-10-07.)** The automation-bias finding: humans over-weight automated suggestions and under-weight contradicting information. The reference work (Wikipedia, *Automation bias*, retrieved 2026-10-07) names the resulting error classes — **errors of commission** (following an automated directive against other evidence) and **errors of omission** (failing to notice the automation missing a problem) — and cites Parasuraman & Riley for the use/misuse/disuse frame. Directly relevant: an unread RECOMMEND is an error of commission; a silent WAIT is an error of omission.
- **Lee, J.D. & See, K.A., *Trust in Automation: Designing for Appropriate Reliance*, Human Factors 46(1):50–80, 2004, DOI 10.1518/hfes.46.1.50.30392.** **(verified — peer-reviewed journal, 2004; metadata resolved via Crossref, 2026-10-07 — the SAGE full text would not extract (unverified-flag on the body).)** The goal is *calibrated trust* — reliance matched to the automation's actual reliability — rather than maximal or minimal trust. The posture-policy reading: the design target is not "the human always checks" (unattainable at volume) nor "the human always trusts" (misuse), but reliance calibrated to the posture — full reliance on a sandboxed reversible ACT, real scrutiny on an irreversible one.
- **Dietvorst, B.J., Simmons, J.P. & Massey, C., *Algorithm Aversion: People Erroneously Avoid Algorithms after Seeing Them Err*, J. Exp. Psychol. Gen. 144(1):114–126, 2015, DOI 10.1037/xge0000033.** **(verified — peer-reviewed journal, 2015; metadata resolved via Crossref, 2026-10-07.)** People abandon an algorithm after seeing it err, more readily than they abandon a human after comparable errors. The reference work (Wikipedia, *Algorithm aversion*, retrieved 2026-10-07) frames it as the tendency "to reject advice or recommendations from an algorithm in situations where they would accept the same advice if it came from a human". The posture relevance: a single visible error can poison a RECOMMEND posture (the human stops accepting), which is a failure the posture's own logs will not reveal.

### 8.4 The design-guideline line

**Amershi, S., Weld, D., Vorvoreanu, M., et al., *Guidelines for Human-AI Interaction*, CHI 2019 (18 guidelines; CHI'19 Honorable Mention).** **(verified via Microsoft Research publication page — peer-reviewed, May 2019.)** Eighteen design guidelines for human–AI interaction, validated across the interaction lifecycle — including guidelines about timing, scope, and making clear *what the system can and cannot do*. The posture-policy reading: the guidelines are the human-facing design surface of a posture choice (how an ASK is presented, how a RECOMMEND is framed); they do not themselves decide *which* posture applies to an action class, which is this guide's subject.

### 8.5 The LLM-era calibration line (HAZARD 3(a))

Three papers establish that a model's self-reported confidence is not a usable posture input on its own; all are preprints and are dated accordingly. (Details in §3.6.)

| Source | Kind / date | The finding that bears on posture |
|---|---|---|
| Kadavath et al., *Language Models (Mostly) Know What They Know*, arXiv:2207.05221 | preprint (2022-07-11), verified at arXiv API | Self-eval P(True) can be well-calibrated in the right format; the widely-reported companion is that RLHF models are **worse** calibrated |
| Lin, Hilton & Evans, *Teaching Models to Express Their Uncertainty in Words*, arXiv:2205.14334 | preprint (2022-05-28), verified at arXiv API | Verbalized confidence *can* be calibrated ("90% confidence" → well-calibrated probability) and survives some distribution shift |
| Xiong et al., *Can LLMs Express Their Uncertainty?*, arXiv:2306.13063 | preprint (2023-06-22; ICLR 2024), verified at arXiv API | Frontiers LLMs verbalizing confidence tend to be **overconfident**, and no method consistently wins |

The posture conclusion (§3.6): a policy of "ask when unsure" is asking the wrong component, because stated certainty is mis-calibrated and, on the frontier evidence, mis-calibrated *toward over-confidence on hard tasks*.

### 8.6 What the literature does NOT settle

Stated plainly, because the temptation to manufacture a canon is strong:

- **The four postures are the requester's framing, not an established taxonomy.** Horvitz is decision-theoretic, not a four-way enumeration; Sheridan–Verplank and Parasuraman–Sheridan–Wickens measure *degree of autonomy*, not *who carries the cost*. There is no peer-reviewed "ask / recommend / act / wait" taxonomy located in this pass.
- **No source read maps a *posture* onto an LLM agent's action class with evidence.** The mapping in §8.2 is this guide's reasoned bridge, not a published result.
- **The human-factors results (1978–2015) are about human operators of automation, not about LLM agents.** Using them to reason about an agent posture is an *analogy* — informative because the human-side failure modes rhyme (misuse, disuse, automation bias, algorithm aversion) — and it must not be presented as an empirical finding about LLM agents.

---

## 9. Banking and Authority: Who May Commit the Institution to What

In a bank, a posture is not primarily a usability choice; it is an **authorization** choice. The decisive question is not "how confident is the agent?" but **who is permitted to commit the institution to what**, and under what mandate. This section connects the posture policy to the authority structures the governed-agent and register literatures describe, citing those frames rather than re-deriving them.

### 9.1 The question reframed: mandate before confidence

For a consumer agent, posture is a trade between interruption cost and error cost (§3.4). For an institutional agent, at least one posture is *unavailable regardless of confidence*: an action that lies outside the agent's **mandate** cannot be taken by ACT no matter how certain the agent is, because no amount of model confidence grants authority. Conversely, an action squarely inside a pre-authorised mandate *may* be ACTed even at moderate confidence, because the principal has already accepted the error cost (that is what the mandate is). The policy therefore reads the mandate first and the confidence second, and confidence only ever *tightens* the posture inside a mandate, never widens it beyond one.

### 9.2 The four-eyes control when one of the two eyes is an agent

The **maker-checker** principle (Wikipedia, *Maker-checker*, verified 2026-10-07) is a "central principle of authorization in the information systems of financial organizations": for each transaction, "there must be at least two individuals necessary for its completion. While one individual may create a transaction, the other individual should be involved in confirmation/authorization of the same." It is the DNA of banking authorization, and it assumes both eyes are human.

An agent forces the question: **is an agent one of the eyes, and is that acceptable?** The honest answer is that an agent can be a *maker* — it can create, draft, propose — but it cannot be a *checker* in the sense the principle requires, because the principle exists to place *independent human judgement* on the confirming side, and a check performed by a model on a model's output is not an independent eye. This is why the ServiceNow guardrail ladder puts a human at **rung 4 — human approval at the write boundary** (cited), and why its "cases that do not hold up" include anything that "both decides and executes" and any agent holding "both raise and approve rights". The posture-policy reading: the two-eyes principle constrains posture directly — for a maker-checker action class, ACT is unavailable to the agent and the class is RECOMMEND *committed by a human*, with the human as the second eye.

### 9.3 What a mandate or limit means when the caller can retry

A mandate expressed as a *limit* (a spend cap, a rate, a threshold) assumes a caller that does not systematically repeat itself. An agent retries. So a limit expressed per-call or per-request can be walked through by a caller that simply tries again — not maliciously, mechanically. Two design consequences:

- **The limit must bind the *intent*, not the attempt.** A mandate of "this customer may move up to X" must be expressible as one commitment, not as N calls each individually under X. (The mechanism that makes an intent-bound limit expressible is the idempotency key, but the *policy* decision — which binding the institution intends — is the bank's and cannot be defaulted into.)
- **The limit must be enforced where the retry cannot reach.** A limit enforced in the agent's prompt is enforced by instruction-following; the durable control is server-side authorization. The posture policy points at the enforcement point; the register owns the control (§1.5).

### 9.4 Which postures are available for which action classes, regardless of confidence

A posture policy for a bank should read as a table of *class → admissible postures*, in which confidence is a modifier, not the driver. The shape:

| Action class (by consequence/reversibility) | Admissible postures | Why |
|---|---|---|
| Read / query (no commitment) | ACT | No commitment; the control is entitlement, not posture (§3.3) |
| Draft / propose (commits nothing) | RECOMMEND | The institution keeps the commit with a human and keeps the author human |
| Reversible, bounded write inside a mandate | ACT within limits | The mandate pre-prices the error; the limit bounds accumulation (§3.2) |
| Irreversible, consequential (payment commit, filing, disclosure) | RECOMMEND (human commits) or ESCALATE | The maker-checker constraint (§9.2); the four-eyes rule |
| Out-of-mandate, or a new counterparty, or above ceiling | ESCALATE / STOP | Authority does not reach it; confidence is irrelevant |

The point of the table is that **the class, not the confidence, is the primary key.** Confidence can move an action *down* the table (from ACT to RECOMMEND inside a class when the agent is unsure) but never *up* it.

### 9.5 What the record must show afterwards

For each posture, the record an auditor or a supervisor would be shown (the register's evidence question, §1.5):

- **ACT:** the input, the action taken, the **named identity** that took it (ServiceNow rung 3), the mandate and limit invoked, the reversibility state at commit (§4.4), and the reasoning trace (rung 6).
- **RECOMMEND:** the proposal, the identity/role of the **human who committed**, and evidence that the commit was a human act (the multi-agent banking evidence contract, cited) — because an auto-approved recommendation is a recommend posture that has silently become an act (§7.2).
- **ASK:** the question posed, the evidence shown with it, and the human's answer.
- **WAIT:** the fact of the wait, its reason, its timeout, and what ended it — because a wait with no record is indistinguishable from an omission (§7.4).

### 9.6 The frames this section borrows (not re-derives)

- **[multi_agent_banking_guide.md](multi_agent_banking_guide.md)** — the **Governed-Agent Design Principle** (scope, escalation, evidence, evaluation contracts) and the **Human-in-the-Loop Map** across fraud triage, AML screening, compliance automation and KYC. The escalation contract there ("written before the agent is built") is the model this section's step-up triggers follow; this section does not restate the map.
- **[../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md)** — the **controls** (preventive/detective) and the **evidence** question; the posture policy is a producer of register rows, not a competing framework.
- **[../servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md)** — the **seven-rung Guardrail Ladder**; **rung 4** (human approval at the write boundary) and **rung 3** (named identity) are the authority mechanisms this section references. The vendor's **Supervised/Autonomous** and **copilot/autopilot** switches (§11) are how the ladder is configured.
- **No jurisdiction's rule is asserted here.** This section connects to the frames above; it does not claim any specific regulatory instrument governs agent postures. Where a rule would be needed, the register and compliance guides are the place to look, and their own guides record the gaps.

### 9.7 The bank's posture question, stated once

Strip away the mechanics and the bank's posture question is singular: for each action class, **which human is permitted to commit the institution to it, what mandate has the agent been given beneath that human, and what does the record show afterwards?** The four postures are the answer's vocabulary; the answer itself is an authorization decision, and it belongs to the institution, not to the agent's confidence.

---

## 10. Measurement: What to Instrument and What It Cannot Show

A posture policy is testable only if the postures are observable. This section names the instruments and — the honesty that matters — what each can and cannot show. The bias of the set is deliberate: the ordinary metrics (task success, completion) are the ones that *cannot* see posture failure (§7.5), so the instruments below measure the *posture choices*, not the outcomes alone.

| Instrument | What it measures | What it can show | What it cannot show |
|---|---|---|---|
| **Ask rate** | Asks per unit of work (per task, per hour, per class) | Whether the ask posture is being used at a volume that can be read (§5), and whether it drifts upward | Whether the asks are *useful* — a high ask rate can be correct (high-consequence class) or brittle (mis-priced interruption) |
| **Approval-without-review rate** | Share of approvals where the human's dwell time / evidence-opening suggests no review | The reflexive-approval failure directly (§7.1) | Whether a *fast* approval was reflexive or genuinely trivial — the threshold is a proxy, not ground truth |
| **Override rate** | Share of RECOMMEND posture outputs the human edits or rejects | Whether the recommend posture is producing usable proposals; a near-zero override rate signals unread acceptance (§7.2) | *Why* the override happened (bad proposal vs anchoring vs a changed situation) — needs a reason code |
| **Incident rate (by posture)** | Failures attributed, where possible, to a posture (wrong ACT, unread RECOMMEND, silent WAIT) | Whether the posture *assignment* is producing its predicted failures | Posture attribution is often only reconstructable after the fact, from the transcript |
| **Wait rate (with reason)** | Share of decision points the agent left unactioned, logged with the reason and timeout | The wait posture's volume and its failure mode (§7.4) — the one instrument that can see inaction | The *value* of the action that did not happen — the opportunity cost is not in any log |
| **Approval latency** | Time from ask to answer | Whether humans are actually engaging with asks, or batch-dismissing them | Quality of the answer |

### 10.1 The instrument that matters most — and why

The **override rate** and the **approval-without-review rate** are the two instruments a posture policy cannot do without, because they are the only ones that can see the two silent failures (unread RECOMMEND and reflexive ASK) that the outcome metrics are blind to. The multi-agent banking guide's rule applies (cited): a *high* override rate is a defect in the agent; a *persistently near-zero* override rate is a defect in the reviewer. Neither number is meaningful alone; together they locate where the human-in-the-loop has stopped being a loop.

### 10.2 What measurement cannot do

- **It cannot price the wait.** The opportunity cost of a non-action is not recorded anywhere, so the wait posture will always *look* cheaper than it is. The remedy is not a better metric — it is the policy's obligation to define, per class, when WAIT is intended and to instrument the *expected* action that a wait forgoes (§7.4).
- **It cannot attribute causality from outcomes alone.** An incident shows an effect, not the posture that produced it. Attribution requires the transcript (the reasoning trace), which is why the record (§9.5) and the metric are one system, not two.
- **It cannot certify.** These are dated observations of a policy on a model, not a permanent grade — the same discipline the agent-experience guide applies to its audits, and the drift guide to behaviour over time (both cited). A posture policy's measurements expire with the model version.

---

## 11. Current Practice: How Today's Platforms Implement Postures

Today's agent platforms implement postures in their **permission models**, and the models are the clearest published evidence of how the industry divides the question. This section records what the vendors publish, with dates, and states what that practice does **not** settle. The material here is **(vendor documentation)** — a vendor's own product design — and is never presented as a research finding.

### 11.1 Claude Code permission modes

Source: **code.claude.com/docs/en/permission-modes**, checked October 2026. **(verified — vendor documentation, October 2026.)** Claude Code sets "which actions Claude can take in a session without asking you first" via permission modes:

| Mode | What runs without asking | The posture it encodes |
|---|---|---|
| `default` (Manual) | Reads only | ASK on everything that writes |
| `acceptEdits` | Reads, file edits, common filesystem commands | ACT for a bounded class (edits), ASK elsewhere |
| `plan` | Reads, plus classifier-approved commands when auto mode is available | RECOMMEND-ish: plan before changing |
| `auto` | Everything, with background safety checks; a second model, the "classifier", reviews actions | ACT with a machine-checked gate — the human replaced by a second model |
| `dontAsk` | Reads and pre-approved tools; anything that would prompt is **denied** | Pre-authorised class; fail-closed |
| `bypassPermissions` | Everything | ACT, always (isolated containers/VMs only) |

The documentation also states that **no mode auto-approves** certain actions — tools matched by an explicit `ask` rule, connector tools an organisation set to `ask`, tools that require user interaction, `rm`/`rmdir` on a critical path, and reads outside the working directory when `blockReadsOutsideWorkingDirectories` is on — and that permission **allow/ask/deny rules** layer on top of modes, with **deny rules blocking in every mode including `bypassPermissions`**. It documents `PreToolUse`/`PermissionRequest` **hooks** for custom permission logic, and a Bash **sandbox** as a separate mechanism from modes. **(vendor documentation, October 2026.)**

### 11.2 OpenAI Codex approval and sandbox settings

Source: **developers.openai.com** Codex configuration documentation, checked October 2026. **(verified — vendor documentation, October 2026, for the config keys below; the dedicated `agent-approvals-security` page did not extract, so the finer enumeration of approval modes is an unverified-flag.)** Codex exposes:

- **`approval_policy`** — documentation shows `"on-request"` and `"never"`, with a **retired `"untrusted"`** policy named in a migration note. ("Never" means the agent runs without approval prompts.)
- **`sandbox_mode`** — documentation shows `"workspace-write"`, with named permission profiles `:read-only`, `:workspace` and `:danger-full-access`.
- **Organization-enforced constraints** via a `requirements.toml`, "for example, disallowing `approval_policy = "never"` or `sandbox_mode = "danger-full-access"`" — i.e. the institution can forbid the most permissive postures centrally.

The design is the same two-axis shape as Claude Code: an **approval** axis (when does the agent stop and ask) and a **sandbox** axis (what may it reach when it does not) — which is the harness guide's "from an action question to a session configuration" (`approval_policy` alongside `sandbox_mode` rather than one prompt per action).

### 11.3 The vendor switches that map to postures

The ServiceNow agentic-platform guide (cited) records the vendor's two execution switches, which are posture labels by another name: a skill-level **Supervised/Autonomous** mode and an agent-level **`copilot`/`autopilot`** mode, where **`copilot` is the SDK's documented preference "when the agent creates, updates, or deletes records"** and `autopilot` is for "read-only or fully autonomous workflows". In this guide's vocabulary, `copilot` is RECOMMEND-committed-by-human and `autopilot` is ACT. That the vendor's own default preference for a *writing* agent is the human-commits posture is the strongest published signal one could ask for that the industry's practical centre of gravity for consequential writes is RECOMMEND, not ACT.

### 11.4 What current practice does NOT settle

- **The modes are permission config, not policy.** Claude Code's and Codex's settings say *how automatic* the agent is; they do not say *which action class* should be in which mode, or *who carries the cost* when a class is mis-assigned. The platform gives the dial; the institution still has to write the policy around which class sits where.
- **The `auto`/classifier mode moves the human, it does not remove the question.** A second model reviewing actions is a *machine* gate; whether a second model is an acceptable second eye for a maker-checker action class (§9.2) is an authorization decision the platform does not make for the bank.
- **The fine-grained approval-mode enumeration is not fully verified here.** The dedicated Codex approvals page did not extract in this pass (unverified-flag); only the config keys read from the configuration page are verified. Named modes beyond those keys should be re-checked at source before being relied upon.
- **Practice is dated vendor documentation, not evidence.** That several vendors converge on a two-axis (approval × sandbox) model is a fact about *their* product designs as published in October 2026. It is not a research finding that the design is correct, and it is not a canonical taxonomy of four postures.

---
## 12. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is a fictional institution used across this repository as the worked-example persona. Every action class, figure, threshold and date in this section is **invented for illustration**, chosen to be plausible, and marked as illustrative. Nothing here is a measurement, a benchmark, a vendor figure or a regulatory requirement, and none of it should be cited as evidence. Real papers, vendors and products named elsewhere in the guide are named factually; **no real bank appears as the example.** Cymbal Bank is the only institution used.

### 12.1 The situation

Cymbal Bank deploys a **customer-servicing agent** (illustrative) that handles routine retail-servicing requests end to end: reading account state, drafting replies, updating customer details, managing card controls, preparing payments, and flagging cases for human handling. The agent has a named identity, a mandate scoped to servicing (not credit, not lending, not onboarding), and a tool surface whose classes are being assigned postures for the first time. The question the bank faces is exactly this guide's: which posture for which class.

### 12.2 The action classes and their postures

Working §3 (variables) and §4 (the matrix) against each class produces the table below. Consequence and reversibility are the axes; the posture is the assignment; the limit and escalation are the bounds.

| # | Action class (illustrative) | Consequence | Reversibility | Posture | Limit / escalation |
|---|---|---|---|---|---|
| C1 | Read account state for the servicing flow | Low | Fully reversible | **ACT** | Entitlement-scoped; logged |
| C2 | Draft a customer reply for a human to send | Low | Fully reversible | **RECOMMEND** (human sends) | Human is the commit author |
| C3 | Update customer contact details (address, phone) | Low-moderate | Reversible at cost | **ASK / stepped-up verification** | See §12.4 — the class where the obvious posture is wrong |
| C4 | Place a temporary card block on a fraud signal | Moderate | Reversible (unblock, with care) | **ACT within a bound** | Auto-block on a high-confidence fraud signal, bounded per day |
| C5 | Unblock a card / reverse a block | High | Irreversible in effect | **ASK + four-eyes** | Human confirms identity out-of-band; second eye (§9.2) |
| C6 | Initiate a customer-authorised payment | High | Irreversible once settled | **RECOMMEND / ACT inside a pre-authorised limit** | Customer pre-authorises beneficiary set and cap; commit at settlement |
| C7 | Freeze or close an account | High | Irreversible | **ESCALATE** | Out of mandate; human decision |

### 12.3 Applying the reversibility/consequence framing

- **C1 and C2** are the easy corners of the matrix (§4.1): C1 is reversible and low-consequence (ACT once entitlement is right), C2 commits nothing (RECOMMEND). Neither depends on confidence.
- **C4** is the reversible-but-moderate class: ACT is admissible *because* the block is reversible and the mandate pre-prices it, but only *within a limit* (a bounded number per day), because the failure mode is *accumulation* — a hundred borderline blocks compose into a customer-experience incident even though each is individually reversible (§3.2). This is the `auto`/sandbox quadrant, bounded by a rate limit.
- **C6** is the class that shows why the middle needs a gate at the moment of irreversibility (§4.4): the customer pre-authorises a *pattern* (beneficiaries and a cap — a class approval, §5.4), and the agent may ACT inside it; the human's real gate is the settlement boundary, not each step. The mandate is intent-bound, and its enforcement point is server-side (§9.3).

### 12.4 The class where the obvious posture is wrong

**C3 — updating customer contact details.** The obvious reading: it is *low-consequence* (a phone number) and *reversible* (change it back), so the obvious posture is **ACT** — let the agent do it. That reading is wrong, and the reason is worth stating precisely, because it is the same trap as §3.1's *claimed reversibility* and it generalises.

Contact-detail change is the **fraud attack surface**: it is the step an attacker who has taken over a customer session *needs*, because changing the registered phone or address is how an attacker redirects one-time codes and paper statements. The action is small and reversible *in itself* and *irreversible in consequence* — the moment the detail is changed, the next authentication can be redirected, and the loss that follows is neither small nor reversible. This is the failure mode §3.1 warns about: reversibility judged by the existence of an undo, not by the cost of the consequence the undo cannot reach.

The correct posture is therefore **not ACT**. It is **ASK with stepped-up verification** — the agent does not act on the change request; it routes it to a step that verifies the requester through a channel the current session cannot influence (out-of-band identity confirmation), and the change is committed only then. In the four-eyes terms of §9.2, C3 needs the human as a *checker* even though the action looks trivial, because the action is the *point of attack*, not the *loss*. The generalisable lesson: **the obvious posture is wrong wherever the action's consequence is realised through a consumer the agent cannot see** — here, the authentication path that the changed detail controls.

### 12.5 Mapping the four-eyes control

Two classes show where the maker-checker principle (§9.2) binds the posture:

- **C5 (unblock):** the unblock *looks* like the reversible, safe, "just undo it" action — and that is exactly why it needs four eyes. Unblocking is the step an attacker wants in the other direction (restoring access), it is irreversible in effect, and the agent cannot be the confirming eye. The maker is the agent (it may *propose* the unblock with evidence); the checker is a human, verifying identity out-of-band. Posture: **ASK + human commit**, with the human as the second eye.
- **C6 (payment):** the classic maker-checker class. The agent may *prepare* (maker); the human or the customer pre-authorisation is the confirming side, and the commit is at settlement. Posture: **RECOMMEND / bounded ACT**, with the two-eyes property preserved by keeping the confirming act human (or an intent-bound pre-authorisation that a human granted).

In both cases the agent is a legitimate **maker** and never the **checker** — the posture reflects it.

### 12.6 What the record shows afterwards

Per §9.5: C1 logs the read and the entitlement; C2 logs the proposal and the human who sent it; C3 logs that verification happened, through which channel, and the verifying role; C4 logs the block, the signal, the identity and the daily count; C5 logs the proposal, the out-of-band verification and the human commit; C6 logs the pre-authorisation invoked, the limit state, the settlement commit and the reversibility state at commit; C7 logs the escalation and the human decision. A posture policy without these records is an observation, not a control (§6.3).

### 12.7 Where Cymbal lands — and the thesis

Work the classes against §3 and §4 and the pattern is not "use the safest posture everywhere". C1 acts; C2 recommends; C4 acts *within a bound*; C5 and C6 keep the human on the confirming side; C3 — the trivial-looking one — is the class that most needed the gate, because its consequence is realised through a consumer the agent cannot see. Every assignment is a decision about **who carries the cost of being wrong**: the agent and its sandbox for C1 and C4, the human reviewer for C2 and C6, the four-eyes pair for C5, and for C3 the customer and the bank together — which is why C3's posture had to change. The bank does not choose a posture per *technology* or per *confidence*; it chooses one per *action class*, and the choice is an allocation of the cost of error. **The posture is a choice about who carries the cost of being wrong.**

---

## 13. The Anti-Patterns

Each anti-pattern is stated as **symptom → cause → guardrail**, and each is a posture failure done on purpose.

### 13.1 Ask everything, before every action

- **Symptom.** The agent asks about nearly everything; the human approves at speed; the approval log is full and the review is empty.
- **Cause.** Treating ASK as the safe default and pricing the interruption at zero (§3.4, §5.2). Prompt fatigue and reflexive approval follow (§7.1).
- **Guardrail.** Price the interruption; buy rarity with a sandbox and class pre-authorisation (§5.4); instrument the approval-without-review rate (§10).

### 13.2 Default to WAIT when uncertain

- **Symptom.** The agent defers whenever unsure; the queue of un-actioned decisions grows; no error, no ticket, no complaint.
- **Cause.** Mistaking risk-of-commission for all risk (§2.4). The wait posture's costs are unrecorded, so it looks cheap.
- **Guardrail.** Reify every wait — reason, timeout, re-evaluation (§7.4); instrument the wait rate; define per class when WAIT is intended.

### 13.3 The recommendation nobody reads

- **Symptom.** A high volume of RECOMMEND outputs accepted with a near-zero override rate; the human is a signature, not a decider.
- **Cause.** The recommend posture was adopted for its safety *appearance* without a review that actually happens (§7.2). Automation bias (§8.3) eases the slide.
- **Guardrail.** Measure the override rate and treat near-zero as a defect in the reviewer (multi-agent banking rule, cited); make the recommendation show what changes if rejected.

### 13.4 "Ask when the agent is unsure"

- **Symptom.** A policy that keys the posture to the model's stated confidence.
- **Cause.** HAZARD 3(a): stated confidence is mis-calibrated, and on the frontier evidence over-confident on hard tasks (§3.6, §8.5). The policy asks the wrong component.
- **Guardrail.** Assign posture by action class (§6.1) and use confidence only *within* a class and only after calibration is validated for this model and task.

### 13.5 The reversible action that isn't

- **Symptom.** An action authorised by ACT because "there is an undo", then found irreversible in effect.
- **Cause.** Reversibility read off the existence of a compensating action rather than off the cost/latency of executing it, or off a downstream consumer the agent cannot see (§3.1, §12.4).
- **Guardrail.** Judge reversibility by the *executable* undo and by the effect's consumers; gate at the moment of irreversibility (§4.4).

### 13.6 The class assigned once and never reviewed

- **Symptom.** A posture policy written at deployment and unexamined through a model change, a tool change and a year of drift.
- **Cause.** Treating the policy as configuration rather than a dated artefact (§6.1 item 6).
- **Guardrail.** Review cadence against evidence; re-audit on model/tool/description change; treat a stale measurement as no evidence (§10.2, drift guide).

### 13.7 Confidence as the authority

- **Symptom.** The agent acts because it is "sure", on a class outside its mandate.
- **Cause.** Confusing *certainty* with *authority* (§9.1). No amount of confidence grants a mandate.
- **Guardrail.** Read the mandate first, confidence second; enforce authority server-side; class, not confidence, is the primary key (§9.4).

### 13.8 The second model as the second eye

- **Symptom.** A maker-checker class "checked" by a model gate (the `auto`/classifier pattern, §11.1) rather than a human.
- **Cause.** The machine gate is convenient and looks like a control; but the maker-checker principle exists to place *independent human judgement* on the confirming side (§9.2).
- **Guardrail.** For classes that require four eyes, keep the confirming eye human; treat a model gate as a *defence in depth*, not the second eye.

---

## 14. The Claims Audit: Verified, Flagged, Rejected

Every claim this guide rests on, with its source, its **date**, and its **kind**. A vendor's permission-mode design is never recorded as a research finding; a 1978 or 1997 result is never recorded as established about LLM agents.

| # | Claim | Verdict | Source · kind · date |
|---|---|---|---|
| 1 | Mixed-initiative UI principles: consider the user's attention; act on expected cost/value; use dialog to resolve uncertainty; minimise the cost of poor guesses | **Verified** | Horvitz, *Principles of Mixed-Initiative User Interfaces*, CHI '99, DOI 10.1145/302979.303030 — peer-reviewed conf. paper, 1999 (PDF read 2026-10-07) |
| 2 | Horvitz's framework is decision-theoretic — it derives when to act/defer from expected cost/value; it is **not** a four-posture enumeration | **Verified** | same source; the twelve principles are cost/attention/dialog principles, not a taxonomy of four postures |
| 3 | Ten levels of automation, from human-does-all to computer-ignores-human | **Verified** | Sheridan & Verplank, *Human and Computer Control of Undersea Teleoperators*, MIT, 1978, DTIC ADA057655 — technical report, 1978 |
| 4 | Four types of automation (info acquisition, info analysis, decision & action selection, action implementation) × levels; level 4 = computer suggests, human retains authority; level 6 = limited time for a veto; secondary criteria include automation reliability and the costs of decision/action consequences | **Verified** | Parasuraman, Sheridan & Wickens, IEEE Trans. SMC-A 30(3):286–297, 2000, DOI 10.1109/3468.844354 — peer-reviewed journal, 2000 (full text read 2026-10-07) |
| 5 | Human–automation interaction has use, misuse, disuse and abuse modes | **Verified** | Parasuraman & Riley, *Humans and Automation: Use, Misuse, Disuse, Abuse*, Human Factors, 1997, DOI 10.1518/001872097778543886 — peer-reviewed journal, 1997 (metadata via Semantic Scholar 2026-10-07) |
| 6 | Automation bias: humans over-weight automated suggestions; errors of commission and omission | **Verified** | Skitka, Mosier & Burdick, *Does automation bias decision-making?*, Int. J. Human-Computer Studies, 1999, DOI 10.1006/ijhc.1999.0252 — peer-reviewed journal, 1999 (metadata via Semantic Scholar; the commission/omission framing from the reference work *Automation bias*, retrieved 2026-10-07) |
| 7 | Appropriate reliance = calibrated trust matched to the automation's actual reliability | **Verified** | Lee & See, *Trust in Automation: Designing for Appropriate Reliance*, Human Factors 46(1):50–80, 2004, DOI 10.1518/hfes.46.1.50.30392 — peer-reviewed journal, 2004 (metadata via Crossref; SAGE body did not extract) |
| 8 | People abandon an algorithm after seeing it err, more than they abandon a human | **Verified** | Dietvorst, Simmons & Massey, *Algorithm Aversion*, J. Exp. Psychol. Gen. 144(1):114–126, 2015, DOI 10.1037/xge0000033 — peer-reviewed journal, 2015 (metadata via Crossref; body not read) |
| 9 | 18 design guidelines for human–AI interaction | **Verified** | Amershi et al., *Guidelines for Human-AI Interaction*, CHI 2019 — peer-reviewed conf. paper, May 2019 (Microsoft Research publication page) |
| 10 | Models can be reasonably calibrated on self-eval in the right format; RLHF models are **worse** calibrated | **Verified** | Kadavath et al., arXiv:2207.05221 — preprint, 2022-07-11 (arXiv API 2026-10-07) |
| 11 | A model can express calibrated uncertainty in words ("90% confidence" → well-calibrated probability, survives some shift) | **Verified** | Lin, Hilton & Evans, arXiv:2205.14334 — preprint, 2022-05-28 (arXiv API 2026-10-07) |
| 12 | Frontier LLMs verbalizing confidence tend to be **overconfident**; no method consistently wins | **Verified** | Xiong et al., arXiv:2306.13063 — preprint 2023-06-22, ICLR 2024 (arXiv API 2026-10-07) |
| 13 | Maker-checker / four-eyes: each transaction needs at least two individuals — one creates, another confirms | **Verified** | Wikipedia, *Maker-checker*, retrieved 2026-10-07 — reference work (definitional) |
| 14 | Claude Code permission modes (`default`/Manual, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`), allow/ask/deny rules, hooks, sandbox, no-mode-auto-approves list | **Verified** | code.claude.com/docs/en/permission-modes — vendor documentation, checked Oct 2026 |
| 15 | Codex `approval_policy` (`on-request`/`never`, retired `untrusted`), `sandbox_mode` (`workspace-write`), profiles (`:read-only`/`:workspace`/`:danger-full-access`), org constraints via `requirements.toml` | **Verified** | developers.openai.com Codex config docs — vendor documentation, checked Oct 2026 |
| 16 | ServiceNow `copilot` is the documented preference "when the agent creates, updates, or deletes records"; the guardrail ladder's rung 4 is human approval at the write boundary | **Verified (as cited from sibling)** | [../servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md) — vendor docs, checked Oct 2026 |
| 17 | Sandboxing reduced Claude Code permission prompts by **84%**; mobile-permission research found 17% attended and 3% understood | **Verified (as cited from sibling; survey-reported vendor figure)** | [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) — survey-reported vendor figure; this guide does not re-vouch for the number |
| 18 | "Ask / recommend / act / wait" is an established taxonomy | **Negative finding** | Searched; no primary source uses a four-posture enumeration under these names. The literature offers levels-of-automation, four types, and Horvitz's decision-theoretic principles — not this taxonomy |
| 19 | A model's self-reported confidence is a usable posture input on its own | **Rejected** | Contradicted by rows 10–12 (over-confidence; RLHF degrades calibration); do not key a posture policy on stated confidence alone |
| 20 | A vendor's permission modes constitute evidence that a posture design is correct | **Rejected — category error** | Vendor docs (rows 14–16) describe product design, not measured outcomes; they are evidence of *practice*, not of *correctness* |

Every row carries a date and a kind. Rows 1–4 are the autonomy/levels prior life; rows 5–8 the human-side failure modes; rows 9–12 the LLM-era calibration and design evidence; rows 13–17 the practice and authority material; rows 18–20 the honest negatives and the rejected claims.

---

## 15. What Could Not Be Verified

Recorded as explicit negatives, because a silent omission reads as coverage. **None of these should be asserted elsewhere.**

1. **A canonical, peer-reviewed "ask / recommend / act / wait" taxonomy.** No source read for this guide enumerates four postures under these names. The nearest antecedents are the levels of automation (1978, 2000), the four types × levels (2000), and Horvitz's decision-theoretic principles (1999) — all *related but not identical* (§8). The four postures are the requester's organising frame.
2. **Any source that maps a posture onto an LLM agent's action class with evidence.** The §8.2 mapping and the §12 class table are this guide's reasoned bridges, not published results.
3. **The body of Lee & See (2004) and of Dietvorst et al. (2015), and the body of Skitka et al. (1999) and Parasuraman & Riley (1997).** Only their metadata (authors, venue, year, DOI) was resolved, via Crossref and Semantic Scholar; the full texts were not extracted (paywalled / blocked engines). Their claims are recorded at the level of the metadata and standard summaries, not quoted from the body.
4. **The full enumeration of OpenAI Codex approval modes.** The dedicated `agent-approvals-security` documentation page did not extract (returned "Not found") in this pass; only the config keys read from the configuration page are verified (§11.2). The finer mode list is an **unverified-flag**.
5. **Any independent replication or measurement of the vendor permission-mode designs** (§11). They are documented product behaviour, dated to October 2026, not measured outcomes.
6. **A per-model, per-task calibration result that would license a "confidence = posture" policy for a specific deployment.** The calibration literature establishes the *general* unreliability (rows 10–12 of §14); no result read establishes a usable threshold for any particular agent.
7. **Any regulatory instrument that specifically governs an LLM agent's posture, action classes or confirmation controls.** Carried forward from the sibling guides' negative findings ([../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md), [../servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md)); §9 therefore asserts no jurisdiction's rule.
8. **Quantitative thresholds** at which a posture assignment becomes "correct" (an ask rate that is too high, an override rate that is too low). No source read establishes one; §10 names the instruments but carries no thresholds by design.

**Research-tooling note affecting reproducibility.** `web_search` returned **empty results on every query** in this pass (a host-level tool limitation, recorded per the standing hazard, **not** an absence of material). The load-bearing method was therefore direct extraction of named primary URLs and API resolution: the Horvitz CHI '99 PDF (erichorvitz.com), the Parasuraman/Sheridan/Wickens 2000 PDF (cs.uml.edu mirror), the Claude Code permission-modes page (code.claude.com), the Codex configuration docs (developers.openai.com), the Wikipedia reference works (*Maker-checker*, *Algorithm aversion*, *Automation bias*), the arXiv API (HTTPS, browser User-Agent) for the three calibration preprints, and the Crossref and Semantic Scholar APIs for the journal metadata. Where a source would not extract, the claim is labelled rather than filled from memory.

---

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

**Act (posture)** — The agent performs the action itself on its own authority; the only posture that produces an irreversible external fact (or the cost of reversing one).

**Approval gate** — A point in the flow at which a human must approve before the agent proceeds, or the agent stops. Its *position* (before the run, interstitial, or post-hoc) is the design choice.

**Ask (posture)** — Soliciting a human decision before proceeding. Costs human attention whether or not the answer was needed.

**Class pre-authorisation** — A human approving a *pattern* of actions (a limit, a beneficiary set, a threshold) once, so the agent may act on instances inside it. The pivot of §5.4.

**Confidence signal** — The agent's own estimate of its certainty. A posture input **only** after calibration is validated for this model and task; not a usable input on its own (HAZARD 3(a), §3.6).

**Escalation** — Routing a decision upward (to a human or a more senior authority) because the agent's mandate does not reach it.

**Four-eyes control / maker-checker** — The principle that each transaction requires at least two individuals, one creating and another confirming (Wikipedia, *Maker-checker*). When one eye is an agent, the agent can be the maker, not the checker (§9.2).

**Mandate** — The grant of authority the agent carries: what it may do, for whom, within what limits. Read before confidence (§9.1).

**Posture** — One of ask, recommend, act, wait: the agent's disposition at a decision point.

**Recommend (posture)** — The agent produces an advisory artefact for a human to accept, edit or reject. Its failure is acceptance without review (§7.2).

**Reversibility** — Whether an undo exists *and can be executed*, and at what cost/latency. A primary driver of posture (§3.1), distinct from the mere existence of a compensating action.

**Wait (posture)** — Not acting at a decision point. A cost, not the safe default (§2.4), whose failures are silent (§7.4).

### 16.2 Cross-references

Within `technology/ai_llm/`:

- [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) — owns the permission-prompt economics, the sandbox argument, the 84% figure and the ETCLOVG layer G / H3–H4 hooks; §5 cites and does not restate.
- [multi_agent_banking_guide.md](multi_agent_banking_guide.md) — owns the Governed-Agent Design Principle and the Human-in-the-Loop Map; §9 cites the escalation contract and override-as-metric discipline.
- [agent_experience_guide.md](agent_experience_guide.md) — owns silent failure on the interface side; §2.4 and §7.4 borrow its analogy.
- [ai_agent_drift_guide.md](ai_agent_drift_guide.md) — owns drift; §6 and §13 cite it as the reason for a review cadence.

Elsewhere in the repository:

- [../servicenow_agentic_platform_guide.md](../servicenow_agentic_platform_guide.md) — owns the seven-rung Guardrail Ladder and the Supervised/Autonomous and copilot/autopilot switches; rung 4 is human approval at the write boundary.
- [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) — owns the controls and the evidence question; the posture policy produces register rows.

Primary sources read for this guide: Horvitz, *Principles of Mixed-Initiative User Interfaces* (CHI '99); Sheridan & Verplank (MIT, 1978); Parasuraman, Sheridan & Wickens (IEEE Trans. SMC-A, 2000); Parasuraman & Riley (Human Factors, 1997); Lee & See (Human Factors, 2004); Skitka, Mosier & Burdick (1999); Dietvorst, Simmons & Massey (J. Exp. Psychol. Gen., 2015); Amershi et al. (CHI 2019); Kadavath et al. (arXiv:2207.05221); Lin, Hilton & Evans (arXiv:2205.14334); Xiong et al. (arXiv:2306.13063); Claude Code permission-modes documentation; OpenAI Codex configuration documentation; Wikipedia (*Maker-checker*, *Algorithm aversion*, *Automation bias*).

### 16.3 Closing summary

The posture question — ask, recommend, act or wait — has a thirty-year human-factors life and a short LLM one, and this guide has tried to hand each source the credit its date and kind deserve. The four postures are a usable organising frame, not an established taxonomy; the levels of automation and Horvitz's decision-theoretic principles are related antecedents that say *how much autonomy* and *at what expected cost*, not *whose decision and whose cost*. The variables that should drive the choice are reversibility and its executable undo, blast radius and its accumulation, sensitivity and the recipient's authority, the two-sided economics of interruption, frequency, and the guarded status of any confidence signal. Waiting is not safety — it is a transfer of cost onto the user who needed the action, made silent.

In a bank the question sharpens to one thing: who is permitted to commit the institution to what. The agent can be a maker; it cannot be a checker, because a second eye must carry independent human judgement. The mandate is read before the confidence; the class, not the certainty, is the primary key; and the record afterwards must show the input, the authority invoked and the reversibility state at commit. The mechanisms that enforce all of this — sandboxes, permission economics, HITL maps, guardrail ladders, risk registers — are owned by the guides cited here; this guide owns only the choice they serve.

And the choice has one measure. Every posture displaces the cost of being wrong onto someone — the agent and its sandbox, the human reviewer, the user, the institution, the absent third party — and a policy is defensible only when that displacement is deliberate, bounded, evidenced and reviewed. Write the classes down, assign the postures, instrument the rates, and re-examine the artefact when the model changes. Because **the posture is a choice about who carries the cost of being wrong.**
