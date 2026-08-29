# Workshop Methodology: Designing and Facilitating Effective Workshops — A Solution Architect's Guide

> **Author:** Jack Liu Shurui, Solution Architect — Cymbal Bank, Singapore
> **Context:** the workshop-craft guide of the management series — how a solution architect designs, facilitates, and follows through on workshops across the full instrument range: discovery, requirements elicitation, design, decision, planning/roadmapping, and retrospective. It sits alongside [Communication & Stakeholder Management Skills](../management/communication_stakeholder_management_skills_guide.md) (the stakeholder and influencing layer every workshop runs on), [Management Consulting Skills](../management/management_consulting_skills_guide.md) (hypothesis-driven problem solving and structured thinking — the analytical engine behind agenda design), and the repository's decision-capture anchor [Architecture Decision Records](../technology/architecture_decision_record_guide.md) (the convention this guide cross-references for capturing workshop decisions — it does *not* re-derive ADR practice).
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass had **live web access** (web_search + web_extract). Verified at primary or authoritative sources: the Design Council's own history of the Double Diamond (designcouncil.org.uk), the Design Council Wikipedia record (founding 19 December 1944 as the Council of Industrial Design), the DSDM/MoSCoW lineage (Wikipedia MoSCoW method and Dynamic Systems Development Method articles), Alex Osborn's *Applied Imagination* (1953, Open Library/Regent University), the SCAMPER and Ishikawa diagram articles (Wikipedia), the Five Whys article (Wikipedia, Sakichi Toyoda / Taiichi Ohno / Toyota Production System), the SWOT history (Wikipedia, SRI Long Range Planning Service 1965 SOFT report), the Nominal Group Technique article and its primary references (Delbecq & Van de Ven 1971, *Journal of Applied Behavioral Science*), the Pugh method article (S. Pugh, ICED 1981), Nielsen Norman Group's "Journey Mapping 101", the Pragmatic Bookshelf pages and extracted excerpts of Derby & Larsen's *Agile Retrospectives* (July 2006, including the book's own table of contents), and Open Library's record of Kerth's *Project Retrospectives* (Dorset House, 2001). Everything verified this pass is marked ✅ in the [Claims Audit](#12-the-claims-audit); anything that could not be verified is flagged ⚠ or ❌ there and in [What Could Not Be Verified](#13-what-could-not-be-verified). Nothing in this guide is fabricated; where the brief demanded verification and the tools could not deliver it, the guide says so plainly.
> **Last Updated:** August 2026

---

### How to Use This Guide

**What this guide is.** A complete, self-contained treatment of the workshop as an instrument of architecture work — when to run one, how to design one backwards from the decisions it must produce, which facilitation techniques to deploy when (with concrete mechanics, not just names), how to handle the human dynamics that make or break a session, how to run retrospectives, and how to convert workshop output into the artefacts that survive the room: minutes, action items, and decision records. The worked example in §11 is a full Cymbal Bank architecture design-workshop series, end to end.

**How to use it.** Read §1–§2 first — the instrument-selection and design mental model every later section assumes. §3–§6 are the technique library (reach for them when designing an agenda). §7–§8 are the facilitation craft, including remote/virtual. §9 covers retrospectives; §10 covers outputs and follow-through; §11 is the worked example that binds it all. §12 is the claims audit, §13 the honest gaps, §14 the glossary.

**What this guide is not.** It is not a general meeting-management or communication guide (that lives in `communication_stakeholder_management_skills_guide.md`), not an ADR manual (that lives in `../technology/architecture_decision_record_guide.md` — this guide cross-references its conventions for decision capture), not a strategy-frameworks catalogue (that lives in `strategic_management_guide.md`), and not a consulting-skills training plan (that lives in `management_consulting_skills_guide.md`).

**The one-sentence thesis:** *a workshop is a machine for turning the collective knowledge in a room into a decision, a design, or a shared understanding — and like any machine, it must be designed before it is run, and its output must be captured before the room disperses.*

---

## Table of Contents

1. [The Workshop Landscape: Types, and When a Workshop Is (and Isn't) the Right Instrument](#1-the-workshop-landscape-types-and-when-a-workshop-is-and-isnt-the-right-instrument)
2. [Designing the Workshop I: Objectives, Outcomes, and the Right Room](#2-designing-the-workshop-i-objectives-outcomes-and-the-right-room)
3. [Designing the Workshop II: Agenda, Pre-Work, Timeboxing, and Setup](#3-designing-the-workshop-ii-agenda-pre-work-timeboxing-and-setup)
4. [The Technique Library I: Diverge — Ideation and Discovery](#4-the-technique-library-i-diverge--ideation-and-discovery)
5. [The Technique Library II: Sensemaking — Synthesis and Root Cause](#5-the-technique-library-ii-sensemaking--synthesis-and-root-cause)
6. [The Technique Library III: Converge — Prioritisation and Decision](#6-the-technique-library-iii-converge--prioritisation-and-decision)
7. [The Facilitation Craft: Neutrality, Energy, and Difficult Dynamics](#7-the-facilitation-craft-neutrality-energy-and-difficult-dynamics)
8. [Remote and Virtual Facilitation](#8-remote-and-virtual-facilitation)
9. [The Retrospective Formats](#9-the-retrospective-formats)
10. [Outputs and Follow-Through: Minutes, Actions, Decisions](#10-outputs-and-follow-through-minutes-actions-decisions)
11. [The Worked Example: The Cymbal Bank Architecture Design-Workshop Series](#11-the-worked-example-the-cymbal-bank-architecture-design-workshop-series)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [The Glossary](#14-the-glossary)

---

## 1. The Workshop Landscape: Types, and When a Workshop Is (and Isn't) the Right Instrument

### 1.1 What a Workshop Actually Is

A **workshop** is a structured working session in which a defined group of people, guided by a facilitator, produces an agreed output — a decision, a design, a prioritised backlog, a shared understanding — in a bounded timebox. Three properties distinguish a workshop from a meeting:

1. **It is designed backwards from an output.** A meeting has an agenda of topics; a workshop has a target artefact. You design a workshop by asking "what must exist when people leave?", then working backwards to the activities that produce it. This is the consulting habit of answer-first design (see `management_consulting_skills_guide.md` §2): the output is the hypothesis, the agenda is the analysis.
2. **It uses structured techniques, not conversation.** Brainstorming rounds, affinity mapping, dot voting, a decision matrix — each technique is a small machine with rules, and the rules are what make the collective output better than a free-form discussion would produce. Techniques are the workshop's equivalent of code patterns.
3. **It has a facilitator distinct from the participants' content roles.** Someone must own the process, the time, and the participation balance — and that someone is usually not the most senior person in the room and not the person with the strongest opinion. (The craft of this separation is §7.)

For a solution architect, the workshop is one of the highest-leverage instruments available. Architecture is decided in rooms — design reviews, requirement sessions, backlog prioritisation, incident retrospectives — and the architect who can design and facilitate those rooms well converts their technical judgement into organisational outcomes. This guide treats that skill as a first-class professional capability, on par with diagramming and ADR writing.

### 1.2 The Six Workshop Types

Six recurring workshop types cover the architect's practice. They differ in their **primary output**, their **typical duration**, and their **dominant thinking mode** (divergent — opening up options — or convergent — closing down to a choice; the double diamond in §4.1 gives the underlying model).

| # | Workshop type | Primary output | Typical duration | Dominant mode |
|---|---|---|---|---|
| 1 | **Discovery** | Shared understanding of a problem space; user needs; current state | Half day to 2 days | Divergent |
| 2 | **Requirements elicitation** | A prioritised, agreed requirements baseline (often MoSCoW'd, §6.1) | Half day to 3 days | Both, alternating |
| 3 | **Design** | An architecture or solution design — or a shortlist of options with trade-offs | 2 hours to 3 days | Both, alternating |
| 4 | **Decision** | One or more explicit, recorded decisions with owners | 1–4 hours | Convergent |
| 5 | **Planning / roadmapping** | A sequenced plan: milestones, dependencies, owners, a roadmap | Half day to 2 days | Convergent |
| 6 | **Retrospective** | A small set of agreed improvements with owners | 1–2 hours | Convergent (with divergent input) |

These are archetypes, not silos — in practice a design workshop contains decision points and a discovery workshop feeds a requirements workshop. But naming the archetype matters because each has a different success criterion: a discovery workshop succeeds if the *problem* is understood; a decision workshop succeeds only if a *decision* is made and recorded. Designing a session without knowing which archetype it is — the most common workshop failure — produces the worst of both: lots of discussion, no artefact.

### 1.3 When Each Type Is the Right Instrument

**Discovery workshops** are right when the problem space is poorly understood and the people who understand it are in the organisation, not in documents. Typical triggers: a new initiative where the scope is fuzzy, an unfamiliar business domain, the start of a transformation programme. The output is insight and alignment — not requirements, not designs. A discovery workshop is the instrument of the first diamond (Discover/Define, §4.1): it widens and then narrows the understanding of the *problem*.

**Requirements elicitation workshops** are right when a set of stakeholders must agree on what a system must do — the classic joint application design (JAD)-style session, though the JAD brand predates the agile era. The output is a requirements baseline that the room owns: features, constraints, priorities (MoSCoW, §6.1), and acceptance criteria at whatever level of detail the phase needs. This is the instrument for the front end of a delivery lifecycle, and it is where an architect earns their keep by translating business asks into non-functional requirements (NFRs) — the requirements that workshops most often forget until they become expensive surprises.

**Design workshops** are right when a design problem has multiple plausible answers and the expertise to choose between them is distributed across roles — business, engineering, operations, risk, security. The output is a design or a design shortlist with explicit trade-offs, feeding ADRs (cross-ref `../technology/architecture_decision_record_guide.md` §6 — "when to write an ADR"). Design workshops are the second diamond (Develop/Deliver): they widen solution options, then converge to a recommendation.

**Decision workshops** are right when a specific decision is blocked — not by lack of information, but by lack of a structured forum to weigh the options and commit. Typical triggers: technology selection, build-vs-buy, a standard to adopt, a scope trade-off. The output is a recorded decision with rationale and consequences — an ADR in the making. If the decision is already effectively made and the workshop is a rubber stamp, cancel it (§1.4).

**Planning / roadmapping workshops** are right when a group must commit to a sequence: what gets built when, by whom, in what order, and what depends on what. Typical triggers: quarterly planning, release planning, technology roadmap refresh, migration sequencing. The output is a plan the participants own — which is the whole point: a plan people build themselves is a plan they defend.

**Retrospectives** are right at regular cadence — end of an iteration, a release, a phase, or a major incident — whenever the team's way of working needs deliberate inspection and adaptation (§9). The output is a small number of agreed, owned improvements. The retrospective is the workshop that pays for all the others: it is the mechanism by which the organisation learns whether its workshops and its delivery process are working at all.

### 1.4 When a Workshop Is the WRONG Instrument

The discipline of this section is knowing when *not* to spend a room of people's time. A workshop is the wrong instrument when:

- **The decision is already made.** A workshop called to "validate" a predetermined outcome is a theatre of false consensus; participants detect it within minutes and the damage to trust exceeds any benefit. If the decision is made, communicate it; if buy-in is needed, say that is the purpose and design for it honestly.
- **The information is not in the room.** Workshops synthesise what participants know. If the answer requires analysis, research, or data nobody has yet — market sizing, vendor evaluation, code archaeology — the workshop will produce opinion dressed as fact. Run the analysis first (the hypothesis-driven method of `management_consulting_skills_guide.md` §2), then workshop the interpretation.
- **The group is too large or too conflicted for one session.** A workshop is not a town hall. Beyond roughly 12–15 participants the techniques degrade (see §2.3 on the right room); beyond a certain level of political conflict, a facilitated open session will not surface the real issues — structured one-to-one stakeholder work (cross-ref `communication_stakeholder_management_skills_guide.md` §4) must come first.
- **There is no real interdependence.** If each participant's contribution is independent and the task is additive — collecting status updates, gathering individual inputs — an async board or a document does it better and cheaper. Workshops are for *interdependent* work: where the value comes from the interaction.
- **The output will not be used.** The most common organisational sin: a workshop whose outputs land in a folder and are never referenced. Before booking the room, verify there is a consumer for the output — a decision gate, a plan owner, a design authority. If nobody is waiting for the artefact, the workshop is entertainment.
- **The timing is wrong.** Too early (the problem is still being framed; participants can't yet contribute) or too late (the window for the decision has passed; the workshop is a formality). A workshop is a point-in-time instrument; its value decays sharply outside its window.

The architect's diagnostic before any workshop: *what must change as a result of this session, and is a facilitated group the mechanism that will change it?* If the answer to the second question is not clearly yes, don't run it — the cheapest workshop is the one you don't hold.

---

## 2. Designing the Workshop I: Objectives, Outcomes, and the Right Room

### 2.1 Objectives and Measurable Outcomes

Design starts with the output, not the agenda. Write the workshop objective as a **one-sentence contract**: "By the end of this session, we will have X." X must be a *thing that exists* — a decision, a shortlist, a prioritised list, a design sketch, an agreed plan — not a state of mind ("shared understanding" is a means, not an outcome; the artefact that proves the understanding is the outcome).

Then define **measurable outcomes**: how will you know, the day after, that the workshop worked? Three tests:

1. **Artefact test** — does the named artefact exist, in a form someone can use without having attended? (A photo of a whiteboard does not pass; a typed, structured capture does.)
2. **Decision test** — if the workshop was a decision workshop, is there a decision record with status, date, and owners (cross-ref `../technology/architecture_decision_record_guide.md` §3–§5 for the format conventions — this guide does not re-derive them)?
3. **Commitment test** — did participants leave with agreed actions they own, and can you name the follow-up cadence (§10)?

A useful additional discipline is the **pre-mortem**: before the workshop, ask the design team "it is two weeks later and this workshop was a failure — what happened?" The answers (wrong people, no decision authority, no pre-work, too much scope) become design constraints. The pre-mortem is a planning technique borrowed from decision science; its workshop-design application is standard practice in facilitation playbooks ⚠ (see §13).

### 2.2 The Objective Statement, Made Concrete

A well-formed objective statement for an architect's workshop looks like this:

> **Objective:** By the end of this 3-hour session, we will have selected the event backbone for the payments modernisation programme, with a recorded decision (ADR), a runner-up option documented with its trade-offs, and an agreed proof-of-concept plan.
> **Out of scope:** vendor commercial negotiation, migration sequencing, team structure.
> **Decisions needed:** (1) backbone technology; (2) POC scope; (3) who validates NFRs and by when.
> **Inputs provided in pre-work:** the option-analysis pack (§3.3), NFR draft, capacity estimates.

Note the "out of scope" line: writing down what the workshop will *not* decide is as important as the objective, because scope creep is the default failure mode of group sessions. Note also "decisions needed" as an explicit list — a decision workshop with more than three or four decision points will run out of time and energy; split it.

### 2.3 Participant Selection: Roles, Numbers, and the "Right Room"

**The three roles in any workshop:**

- **The sponsor/decider** — the person (or people) whose authority makes the output real. If a decision workshop's decider is absent, the workshop can at best produce a recommendation; say so in the objective. The sponsor need not be the most senior person in the room, but they must be the person who can say yes.
- **The participants** — the people whose knowledge, judgement, or commitment is needed. Selection rule: *everyone who must be heard, and no one who is only there to watch.* Two useful tests: (a) the **information test** — does this person hold information or perspective no one else in the room holds? (b) the **implementation test** — will this person need to act on the outcome? If neither, they are an observer; observers should be the exception, named explicitly, and seated apart from the working group so they do not dampen candour.
- **The facilitator** — the process owner, separate from content (§7.1). In an architect-run workshop the architect is usually a participant-with-content AND the facilitator; this dual role is workable for small sessions if the architect is disciplined about neutrality on their own opinions — and is the single most common reason to bring in a co-facilitator or external facilitator for high-stakes sessions (see §7.1).

**Numbers.** The practical working range for a single facilitated session is **5–12 participants**. Below five, the group loses diversity and the techniques feel forced; above twelve, the group fragments into side conversations, the quiet people disappear, and the facilitator becomes a crowd controller. For larger groups (roadshows, alignment sessions, 20+ people), use the **1-2-4-all** pattern (individual → pairs → quartets → plenary) or break into parallel breakout teams with report-backs — never run a single discussion with 25 people and call it a workshop. Timebox the report-backs ruthlessly; that is where breakout workshops die.

**The "right room"** is a real design decision, not logistics. Requirements:

- **A wall you can write on** — flip charts or a writable surface large enough for the workshop's artefact (an affinity wall for 100+ sticky notes needs 3–4 metres of wall). If the room lacks this, the workshop's capture capacity is capped before it starts.
- **Round or cabaret seating, not classroom rows** — people must see each other; the facilitator must see everyone.
- **No table fortress** — tables that separate people from the wall slow the work; sticky-note work happens standing at the wall.
- **No interruptions** — phones away, laptops closed except for the capture role, doors shut. The single biggest quality destroyer in workshops is half-present participants checking mail.
- **Natural light and a hard stop** — energy management (§7.2) is partly a room-design problem.

The virtual equivalent of the right room is the shared board (§8): the online whiteboard *is* the wall, and its setup quality determines the session's capture capacity exactly as the physical wall does.

### 2.4 Ground Rules

Ground rules are the social contract of the session, and they work best when they are few, visible, and agreed — not dictated. A starter set for an architect's workshop:

1. **One conversation at a time.** (The facilitator enforces this; it is the cheapest participation fix there is.)
2. **Defer judgment during divergence.** Ideas are captured, not critiqued, until the convergence phase — this is the rule that makes brainstorming work (§4.2).
3. **Everything captured, nothing lost.** Every idea goes on the wall or the parking lot (§7.3); nothing is dismissed by silence.
4. **Disagree with the idea, not the person.** Technical disagreements are data, not attacks.
5. **Phone-free, laptop-closed** (except the scribe).
6. **Start and end on time** — the facilitator owns the clock; the room owns the outcome.
7. **The parking lot is real** — off-topic but valuable items go there, get reviewed, and get owners (§10.3).

Post the rules visibly, and have the facilitator refer to them *by name* when enforcing ("we agreed on one conversation at a time — Anna, let's get your point in next"). Rules that are never enforced are worse than no rules: they teach the room that agreements are optional.

---

## 3. Designing the Workshop II: Agenda, Pre-Work, Timeboxing, and Setup

### 3.1 Agenda Design: The Activity Chain

An agenda is a sequence of technique-sized blocks, each with a purpose, a method, a timebox, and an output — not a list of topics. Build it as a chain that *starts* with the room's current state and *ends* with the artefact. The standard shape, whatever the workshop type, is the **session arc**:

1. **Open (10–15%)** — check-in, context, objective restated, ground rules, the artefact shown (an empty version of the target output on the wall). The room must see the destination.
2. **Diverge (25–35%)** — generate the raw material: needs, options, issues, ideas (techniques: §4).
3. **Sensemake (25–35%)** — organise and analyse the raw material: cluster it, map it, find root causes (techniques: §5).
4. **Converge (20–30%)** — prioritise, weigh, decide (techniques: §6).
5. **Close (10%)** — decisions read back, actions assigned, parking-lot review, next steps, thank-you. *Never skip the close;* the close is where the workshop becomes an artefact.

Within each block, write the agenda as **activity, not topic**: not "discuss the NFRs" but "dot-vote the NFR list (10 min) → split into three groups to draft acceptance criteria for the top six (25 min) → report back (15 min)". An agenda written as activities is an agenda that can be facilitated; an agenda written as topics is a meeting.

### 3.2 Timeboxing

Timeboxing is the discipline of fixing the time budget and letting scope flex within it — the DSDM principle that makes MoSCoW meaningful (cross-ref §6.1 and `../technology/architecture_decision_record_guide.md` for how timeboxed decisions feed ADRs). Concretely:

- **Budget total time as if it were money**, with a reserve: a 4-hour session gets ~3 hours of agenda and 1 hour of slack, because every real workshop overruns somewhere. The reserve is not waste; it is the buffer that lets you end on time.
- **Timebox every activity** and publish the times. The facilitator's job is to protect the box: "three minutes left in this round" is the most useful sentence in facilitation.
- **Design for the hard stop.** A workshop that runs late punishes the participants who have other commitments — usually the senior ones — and they will never come back. End on time even if the last item is parked (§7.3).
- **Scope flexes, time doesn't.** If a block overruns, cut scope (drop the lowest-priority agenda item), not the clock. This mirrors MoSCoW's fixed-time/flexible-scope logic exactly.

### 3.3 Pre-Work: The Inputs That Make the Room Productive

The quality of a workshop is largely determined before anyone enters the room. Pre-work serves three purposes: it *loads* the room with shared context (so the session starts at the discussion, not the briefing), it *harvests* the ideas that introverts generate better alone (§4.3), and it *sorts* the material so the session spends its time on judgement, not discovery.

Effective pre-work, sent 3–5 working days ahead with a hard deadline:

- **The objective and the agenda** — so participants can challenge the design before it costs a roomful of hours.
- **A reading pack, ruthlessly edited** — the two documents that matter, not the ten that exist. Attach a "read this, skim this, skip this" note; unguided packs are ignored.
- **The pre-work asks** — *specific, individual, timeboxed* contributions: "write your top five NFR concerns on the attached board by Wednesday; 20 minutes, no more." Async boards (§8.4) are the natural home for this.
- **The decision inputs** — for decision workshops, the option-analysis pack: each option with its trade-offs, cost/effort estimates, and risks, prepared *before* the session so the room weighs rather than researches.

The rule: **never use workshop time to do work that could have been done alone.** Every minute of pre-work that lands is a minute of workshop time converted from reading to deciding.

### 3.4 Room and Virtual Setup

The setup checklist, physical: walls papered and artefacts pre-drawn (empty affinity grid, MoSCoW quadrants, decision matrix rows); sticky notes, markers, and dots in sufficient quantity (the universal failure is running out of sticky notes mid-diverge); the agenda and ground rules posted; water and breaks scheduled; the clock visible; the scribe role assigned (a participant, not the facilitator — the facilitator must not write while facilitating; see §7.1).

The setup checklist, virtual (§8): the board pre-built with the same artefacts; participant access tested *before* the session (the first ten minutes of every unprepared virtual workshop are consumed by access problems); breakouts pre-configured; a capture document ready; camera-on expectation agreed; and a **co-facilitator or producer** for any session over six participants — one person cannot simultaneously facilitate, watch chat, manage breakouts, and troubleshoot audio.

### 3.5 The Design Review: Before You Send the Invite

Before the invite goes out, run the agenda through five questions:

1. **Output test** — does every agenda block produce something that moves the room toward the named artefact?
2. **Mode test** — do divergence and convergence alternate, or will the room spend four hours in vague discussion?
3. **People test** — is every participant needed for information or implementation, and is the decider present?
4. **Time test** — does the total fit the timebox with reserve, and is there a hard stop?
5. **Capture test** — who turns the wall into the artefact, and when (that night, not next week; §10.1)?

If any answer is no, fix the design before spending the room.

---

## 4. The Technique Library I: Diverge — Ideation and Discovery

Every technique below is given with its **when-to-use** and its **concrete mechanics** — the facilitator's runbook. The library is organised by thinking mode: this section is divergence (opening up), §5 is sensemaking, §6 is convergence.

### 4.1 Divergence/Convergence and the Double Diamond

**What it is.** The Double Diamond is the design-process model developed by the UK Design Council's Design and Innovation team — Richard Eisermann's team of Anna White, Chris Vanstone, Gill Wildman, Jennie Winhall and Jonathan Ball answered his 2003 challenge "How do we describe design process?" by codifying the common process they found across their projects into four phases — **Discover, Define, Develop, Deliver** — drawn as two diamonds, each pairing a divergent phase (widening) with a convergent phase (narrowing) ✅. Design Council began sharing it at conferences in 2004 ✅ and it entered wide circulation with the council's 2005 *Eleven Lessons: A Study of the Design Process* ✅; the model adapted the divergence-convergence idea proposed by Béla H. Bánáthy in 1996 ✅. Design Council itself was founded on 19 December 1944 as the Council of Industrial Design ✅ (sources: designcouncil.org.uk history page; en.wikipedia.org Double Diamond and Design Council articles — see audit #1–#2).

**When to use it.** As the underlying map for any workshop series that spans problem-framing and solution-finding: discovery workshops occupy the first diamond, design workshops the second. Its workshop-level use is as a *mode check*: at any moment, the room should know whether it is diverging or converging, because the two modes need opposite rules (divergence: defer judgment, go broad; convergence: critique, select, commit).

**Mechanics.**

1. Draw the two diamonds on the wall; label the four phases.
2. Place the current workshop (or the current agenda block) on the map — "we are in Discover, diverging: everything is welcome, nothing is judged."
3. When the mode flips (e.g., from Discover to Define), say so explicitly and change the rules: "we are now converging — we critique, we merge, we select."
4. Use the map as the shared language for scope control: "that question belongs in Develop; we are in Define."

The architect's use of the diamond is not decorative: the most expensive mistake in solution design is converging too early — solving the wrong problem elegantly. The diamond is the visual guard against it.

### 4.2 Brainstorming

**What it is / origin.** Brainstorming was introduced by Alex F. Osborn in his 1953 book *Applied Imagination: Principles and Procedures of Creative Thinking* — Osborn was an advertising executive and co-founder of the agency BBDO ✅ (sources: Wikipedia "Applied Imagination"; Regent University journal article; Open Library — audit #5).

**When to use it.** Early in any divergent phase, when the room must generate a broad option/idea set and the risk is *premature convergence* — the first good idea winning by default. Use it for: solution options, risk identification, feature ideas, improvement ideas in retrospectives.

**Mechanics — Osborn's rules, run strictly:**

1. **Defer judgment.** No criticism, no "we tried that", no eye-rolls — during generation. The facilitator enforces this hard; one sarcastic remark kills a brainstorm.
2. **Go for quantity.** Set a target ("40 ideas in 15 minutes"); quantity breeds quality because associations compound.
3. **Wild ideas welcome.** The absurd idea is often the bridge to the good one; record everything verbatim.
4. **Build on others' ideas** ("yes, and..."), combining and extending.

Run it as: silent individual generation first (2–5 minutes, everyone writes) → round-robin capture on the wall (one idea per sticky, facilitator or scribe writes, *no discussion*) → only then clustering and critique (which is affinity mapping, §5.1, and belongs to the sensemake phase).

**Evidence caveat:** Osborn's claim that group brainstorming outperforms individuals working alone has been contested by later experimental research (nominal groups — individuals generating independently — repeatedly matched or beat interacting groups on quantity and quality; this is precisely the finding that motivates the nominal group technique, §4.3). The practical synthesis: *generate silently and individually first, then share and combine* — the interactive part's value is association and commitment, not raw quantity ✅/⚠ (see audit #5 for the research lineage).

### 4.3 The Nominal Group Technique (NGT)

**What it is / origin.** NGT is a structured group process for problem identification and decision-making developed by André Delbecq and Andrew H. Van de Ven, first published in their 1971 article "A Group Process Model for Problem Identification and Program Planning" (*Journal of Applied Behavioral Science* 7(4)) and extended in Delbecq, Van de Ven & Gustafson's 1975 book *Group Techniques for Program Planning* ✅ (source: Wikipedia NGT article and its primary references — audit #8).

**When to use it.** When participation balance is the risk: dominant speakers present, status gradients steep, the topic controversial, or some members think better in silence. NGT is the structured answer to "the loudest person decides." Its five-stage structure is the canonical fix for the brainstorming evidence problem — it guarantees silent individual generation before any group interaction.

**Mechanics (the five stages).**

1. **Introduction** — purpose, procedure, the question written where everyone can see it.
2. **Silent generation** (~10 min) — each person writes their ideas independently; no discussion.
3. **Round-robin sharing** (15–30 min) — one idea per person per turn, recorded verbatim on the flip chart; no debate, no repeats; new ideas sparked by others are added.
4. **Discussion** (30–45 min) — clarification only: what did this idea mean, what is the context; the facilitator keeps it neutral, no judgment, no elimination.
5. **Voting and ranking** — each participant privately ranks/selects (e.g., 5 votes across the list); the tally is the group's output, computed in the room.

NGT costs more structure than free discussion but buys equal voice, a complete capture, and a defensible, auditable ranking — which is exactly what a decision workshop in a regulated bank needs.

---

## 5. The Technique Library II: Sensemaking — Synthesis and Root Cause

### 5.1 Affinity Mapping (the KJ Method)

**What it is / origin.** The affinity diagram organises large numbers of brainstormed ideas into groups by natural relationship. The term was devised by Jiro Kawakita in the 1960s — hence its alternative name, the **KJ Method** (Kawakita Jiro) ✅ (source: Wikipedia "Affinity diagram" — audit #9).

**When to use it.** Immediately after any large divergence — 40+ sticky notes of ideas, requirements, risks, or interview findings — when the room needs to find structure in the raw material. It is the standard bridge from brainstorming to analysis.

**Mechanics.**

1. Post every idea card on the wall, visible to all.
2. **Silently**, participants move cards into clusters that feel naturally related — no talking during the first pass, so the grouping reflects the data, not the loudest voice.
3. When clustering stabilises, name each cluster with a header card (the name is a claim about the cluster's essence — "data residency constraints", "onboarding friction").
4. Iterate: merge small clusters, split monsters, until the wall reads as 5–9 meaningful themes (the working-memory-friendly range).
5. The named clusters become the structure for everything downstream: the requirements baseline, the risk register, the design's concern areas.

The wall *is* the artefact at this stage; photograph it and transcribe it the same day (§10.1).

### 5.2 Journey Mapping

**What it is / origin.** A journey map is a visualisation of the process a person goes through to accomplish a goal — the actor (persona), a scenario, the journey phases, the actions/mindsets/emotions within each phase, and the opportunities that fall out (Nielsen Norman Group's canonical definition ✅ — Sarah Gibbons, "Journey Mapping 101", 2018 — audit #11). A persistent marketing claim that Walt Disney created the first customer journey map circulates widely but is **contested and unverifiable** — no primary evidence was found this pass; treat it as folklore ⚠ (§13).

**When to use it.** In discovery workshops for experience-heavy domains — onboarding, account opening, a trade lifecycle, an incident journey — when the room must see the process from the *user's* perspective and locate pain points and opportunities precisely. For an architect, journey maps are the bridge between the business's experience language and the system's touchpoint inventory: every journey phase maps to channels, systems, and data.

**Mechanics.**

1. Fix one actor and one scenario per map (a map with two personas is two maps).
2. Draw the phase lane (high-level stages: e.g., discover → apply → fund → use → support).
3. For each phase, capture: **actions** (what the actor does), **mindsets** (thoughts/questions — ideally verbatims from research), **emotions** (plotted as a line across the phases — the "emotional journey").
4. Add the **touchpoint row** (channel/system per phase) — this is the architect's row.
5. Derive **opportunities** (the bottom row): pain points, gaps, and ownership candidates. Each opportunity should be specific enough to become a requirement or a design input.

Journey maps built *in the room* are alignment devices as much as analysis: the argument about where the pain really is happens over the map, not in the abstract.

### 5.3 Personas

**What it is.** A persona is a composite, research-grounded archetype of a user group — name, goals, frustrations, context, skills — used to keep design conversations anchored to real users rather than to "the user" as an abstraction. (Personas have a long UX lineage; the term and practice were popularised through Alan Cooper's work in the 1990s ⚠ — widely cited, not re-verified against a primary source this pass; see §13.)

**When to use it.** Before or during design workshops, as the cast of characters the design must serve. In requirements workshops, personas turn "the customer wants..." into "Priya, the SME relationship manager, needs..." — which changes what gets built.

**Mechanics.**

1. Draft personas *before* the workshop from research (interviews, analytics, SME input) — a workshop is a place to *use* personas, not to invent them from stereotypes.
2. Present each persona in one page: photo/avatar, name, role, goals, pain points, context, and a quote.
3. Throughout the workshop, test every design decision against the personas: "does this work for Priya's Tuesday morning?"
4. Keep personas few (3–5) and visibly posted; a persona wall is a design workshop's conscience.

### 5.4 The Five Whys

**What it is / origin.** The five whys is an iterative interrogative technique that traces a problem's cause-and-effect chain by asking "why?" repeatedly — typically five times — to reach a root cause. It was originally developed by Sakichi Toyoda and used within Toyota Motor Corporation during the evolution of its manufacturing methodologies; Taiichi Ohno, the architect of the Toyota Production System, described it as "the basis of Toyota's scientific approach... by repeating why five times the nature of the problem as well as its solution becomes clear" ✅ (source: Wikipedia "Five whys" — audit #6; note that no single firm year is attested — do not date it).

**When to use it.** For a single, well-defined problem — an incident, a defect, a missed deadline — when the room must move past symptoms to a cause it can actually act on. Use it in incident retrospectives and root-cause sessions; pair it with the Ishikawa diagram (§5.5) when multiple causal chains may exist.

**Mechanics.**

1. State the problem precisely (the fish's head, if combined with §5.5).
2. Ask "why?" — write the answer; ask "why?" again on *that* answer; repeat.
3. Stop when the chain reaches a cause the team can act on — or when the whys stop producing new information (five is a guideline, not a law; the technique's critics note the fifth why is arbitrary ⚠).
4. Validate: the root cause, if fixed, should prevent recurrence; if fixing it doesn't, you stopped too early or went down a dead chain.

Known limitations to state in the room: different investigators reach different chains (results are not fully repeatable), and the single-chain format can miss multiple interacting causes — which is the argument for the fishbone.

### 5.5 Ishikawa / Fishbone (Cause-and-Effect) Diagram

**What it is / origin.** The Ishikawa diagram — fishbone or cause-and-effect diagram — shows potential causes of a problem grouped into major categories, with sub-branches down to root causes. It was created by Kaoru Ishikawa and popularised in the 1960s as one of the Seven Basic Tools of Quality Control (the basic concept dates to the 1920s) ✅ (source: Wikipedia "Ishikawa diagram" — audit #3).

**When to use it.** When a problem plausibly has several interacting cause families and the room must systematically exhaust them — incident retrospectives, delivery-problem diagnosis, NFR-failure analysis. It is the multi-chain complement to the five whys.

**Mechanics.**

1. Draw the spine and the head; write the problem at the head, stated with the five Ws (what, who, when, where, why) to the extent known.
2. Draw the major bones; use a standard category set or let the team set its own: manufacturing's classic 5 Ms (Manpower/Mindpower, Machine, Material, Method, Measurement); product/service variants (Product, Price, Place, Promotion, People, Process, Physical evidence, Performance; or Surroundings, Suppliers, Systems, Skills, Safety) ✅ (Wikipedia, §5.1–§5.3).
3. Brainstorm causes into the categories (sticky notes on the bones).
4. Drill down with "why is this happening?" to add sub-branches; stop when causes fall outside the team's control or influence.
5. **Look for causes appearing in more than one category** — cross-category items are the most likely true root causes (Derby & Larsen give exactly this guidance in their fishbone activity ✅ — audit #10).
6. Convert the top causes into action items — a fully branched diagram that leads to no actions is decoration.

---

## 6. The Technique Library III: Converge — Prioritisation and Decision

### 6.1 MoSCoW Prioritisation

**What it is / origin.** MoSCoW classifies requirements into Must have, Should have, Could have, and Won't have (this time). It was developed by **Dai Clegg** in **1994** for rapid application development (RAD) — Clegg was at Oracle UK — and adopted as a core practice of the Dynamic Systems Development Method (DSDM), whose consortium was founded in 1994 by a group of vendors and experts including British Airways, American Express, Oracle and Logica ✅ (sources: Wikipedia "MoSCoW method" and "Dynamic systems development method" — audit #4). DSDM fixes time, cost and quality and lets scope flex through MoSCoW — the prioritisation mechanism this guide's timeboxing (§3.2) depends on.

**When to use it.** In requirements elicitation and planning workshops, whenever scope must be agreed under a fixed timebox. It is the standard instrument for the "what are we actually building" conversation.

**Mechanics.**

1. Every candidate requirement goes on a card.
2. Define the four buckets *and their semantics* before sorting: **Must** — the release fails without it (not "nice to have"); **Should** — important, with a stated fallback if time runs out; **Could** — desirable, the first scope to cut; **Won't** (this time) — explicitly out, *recorded so it doesn't creep back silently*.
3. Sort as a room — or better, sort individually first and compare (the divergence/convergence discipline again): individual MoSCoW, then the wall, then the arguments.
4. **Guard the Musts.** The discipline's teeth are in the rule that Must is a survival category, not a wish list; a MoSCoW session that ends with 80% Musts has failed and should be re-run with a "if we cut this, what breaks?" test.
5. Record the outcome with the date and the timebox it applies to — it becomes a decision input (cross-ref ADR conventions, `../technology/architecture_decision_record_guide.md` §3–§4, for how scope decisions get recorded).

### 6.2 Dot Voting

**What it is.** The simplest convergent technique: each participant gets a fixed number of votes (dots) and places them on the options they most want to keep or prioritise.

**When to use it.** Anywhere a large list must shrink fast — after affinity mapping, before deep discussion, in retrospectives to pick the top improvement themes. It is a *triage* instrument: it identifies the shortlist the room will spend its judgement on, it does not itself decide.

**Mechanics.**

1. The list is visible (poster or wall).
2. Each person gets N dots (rule of thumb: N ≈ items/3, or a fixed 3–5; in multi-round voting, different colours per round).
3. Vote **silently and simultaneously** — no lobbying during voting; this is what makes it democratic rather than a popularity cascade.
4. Tally and discuss: the top cluster gets the room's remaining time. *Never* treat the dot count as the final decision for high-stakes calls — dots are input to a decision, not the decision itself; pair dot voting with a decision matrix (§6.3) or explicit decision-making for consequential choices.

### 6.3 The Decision Matrix (Pugh)

**What it is / origin.** The Pugh method (decision-matrix method, Pugh concept selection) is a qualitative technique for ranking multi-dimensional options against a reference: each candidate is rated better/same/worse than a baseline per criterion, and the pattern of pluses and minuses — not a summed score — drives the choice. It was invented by **Stuart Pugh**; the earliest traced publication is his 1981 paper "Concept selection: a method that works" (Proceedings of the International Conference on Engineering Design, Rome, 1981) ✅ (source: Wikipedia "Decision-matrix method (Pugh method)" and its references — audit #7).

**When to use it.** In design and decision workshops, when several options must be compared on multiple criteria and the room needs a *structured, auditable* comparison — technology selection, vendor choice, architecture alternatives. For an architect in a bank, the Pugh matrix is the natural companion to the ADR: the matrix is the workshop artefact, the ADR is the recorded decision (cross-ref `../technology/architecture_decision_record_guide.md` §6–§7).

**Mechanics.**

1. Agree the **criteria** first (the rows) — and agree them *before* anyone argues for an option, or the criteria will be rigged by the argument.
2. Choose the **baseline** (the reference candidate): the incumbent, the default, or the cheapest option — explicitly.
3. Score each candidate **+ / – / S** (same) against the baseline per criterion. This is deliberately coarse: it forces discussion of *direction* before false precision.
4. Tally pluses and minuses **without summing them** into a single number — the pattern matters (a candidate that wins on must-have criteria beats one that wins on trivia). Weights can be added for a weighted matrix when criteria importance is genuinely unequal.
5. Do a **sensitivity pass**: how much would opinions have to change to flip the ranking? If the top two are close, the decision is not the matrix's to make — surface the tie explicitly.
6. Record the matrix with the decision (it becomes the ADR's considered-options evidence).

### 6.4 RACI

**What it is / origin.** RACI (Responsible, Accountable, Consulted, Informed) is a responsibility-assignment matrix for describing stakeholder involvement in tasks or decisions. Its origin is **contested**: no single inventor is attested — the practice appears to date from around the 1950s, and the name "responsibility assignment matrix" attached in the 1970s; multiple sources describe the origin as unknown, in use since at least the 1970s–80s ✅/⚠ (sources: Wikipedia "Responsibility assignment matrix"; Perfony's history article; umbrex — audit #12). Do not repeat the common "invented by X in year Y" claims; none survived verification.

**When to use it.** In planning and decision workshops, to settle *who does what* — especially for cross-functional work where ambiguity is the risk: who is accountable for a decision, who must be consulted before it, who only needs to know. Also the standard tool for governance mapping (e.g., who owns an ADR's acceptance — cross-ref the ADR guide's decision-review gate, `../technology/architecture_decision_record_guide.md` §9.2).

**Mechanics.**

1. Rows = tasks/decisions; columns = roles (roles, not individuals — RACI on individuals does not scale).
2. Define the letters sharply before use: **R** (Responsible) — does the work; **A** (Accountable) — answers for the outcome, one A per row; **C** (Consulted) — input sought before the decision; **I** (Informed) — told after.
3. Build it as a room exercise: rows first, then assign, then *argue* — the argument is the value ("why am I Accountable for a decision I can't make?").
4. Check the failure patterns: rows with no A (decisions that will evaporate), rows with multiple As (contested ownership), columns that are all C (a stakeholder with no actual power to move anything).
5. Record it with the plan — the RACI is the follow-through map (§10).

### 6.5 SWOT

**What it is / origin.** SWOT (Strengths, Weaknesses, Opportunities, Threats) is the strategic position-snapshot: internal strengths/weaknesses, external opportunities/threats, in a 2×2. Its history is more contested than its ubiquity suggests ✅: the verifiable primary trail is Stanford Research Institute's Long Range Planning Service — the 1965 report *Formal Planning: The Staff Planner's Role at Start-Up* by Robert F. Stewart, Otis J. Benepe and Arnold Mitchell, which described the precursor acronym SOFT (Satisfactory, Opportunity, Fault, Threat). The same year, Harvard's Learned, Christensen, Andrews and Guth published *Business Policy: Text and Cases*, which Mintzberg et al. later credited as the "design school" source of "the famous notion of SWOT". A 2023 history by Puyt et al. credits SRI's LRPS as the originator and calls the Harvard-invention claim an "academic urban legend". The popular attribution to **Albert Humphrey** (who led SRI work in the 1960s) is widely repeated but was **not confirmed** in the primary trail found this pass ⚠ (source: Wikipedia "SWOT analysis", History section — audit #13).

**When to use it.** In discovery and planning workshops, as a fast shared snapshot of a position — a business line, a platform, a proposal — before strategy discussion. It is a *structuring* tool, not an analysis tool: its output is a shared, roughly-agreed picture and a list of tensions to explore.

**Mechanics.**

1. Draw the 2×2; define the frame explicitly (SWOT of *what*, over *what horizon*?).
2. Generate each quadrant separately — strengths/weaknesses (internal) first, then opportunities/threats (external); some practitioners deliberately reverse the order to stop the room leading with internal politics.
3. Keep items specific and evidence-adjacent ("no DR site in region" beats "weak resilience").
4. The value is in the **cross-quadrant reading**: S×O (where to push), W×T (what to defend), and the strategies that fall out (the TOWS matching of Weihrich, 1982 ✅/⚠ — see audit).
5. Convert the sharpest items into workshop actions — a SWOT wall with no next step is a poster.

### 6.6 SCAMPER

**What it is / origin.** SCAMPER is a structured ideation checklist — **S**ubstitute, **C**ombine, **A**djust, **M**odify (magnify/minify), **P**ut to other uses, **E**liminate, **R**everse/rearrange — developed by **Bob Eberle** and described in his 1971 book *SCAMPER: Games for Imagination Development* (a 1972 *Journal of Creative Behavior* article, "Developing Imagination Through SCAMPER", followed). Eberle built it on the checklist Alex Osborn had described in *Applied Imagination*, alongside influences from Frank E. Williams' creativity work ✅ (source: Wikipedia "SCAMPER" — audit #14).

**When to use it.** In design workshops, as a *systematic* alternative to open brainstorming when the room is stuck or when the obvious ideas have been exhausted — it forces the consideration of moves (substitute, eliminate, combine) that open brainstorming rarely volunteers. Particularly useful for improving an existing design, process, or service rather than inventing from a blank sheet.

**Mechanics.**

1. Pick the object of the exercise — a design, a process step, a requirement.
2. Walk the seven prompts one at a time, each as a short timeboxed round (5–8 minutes): "what could we *substitute*? what could we *combine*? what could we *eliminate*? what happens if we *reverse* the order?"
3. Capture every response verbatim, no judgment (divergence rules).
4. At the end, affinity-map the results (§5.1) and feed the survivors into prioritisation (§6.1–§6.3).

---

## 7. The Facilitation Craft: Neutrality, Energy, and Difficult Dynamics

The techniques of §4–§6 are the workshop's hardware; the facilitation craft is its operating system. This section covers the behaviours that separate a session that produces artefacts from a session that produces resentment.

### 7.1 Neutrality: The Facilitator's Contract

The facilitator's core contract is **process ownership without content ownership**: you own the method, the time, the participation balance, and the capture — and you do *not* own the answer. Concretely:

- **Never argue with a participant's content.** If you disagree with an idea, your disagreement is one opinion among many — and the moment you deploy facilitator authority to win a content argument, you have broken the room's trust in the process. Write your contrary view on a sticky like everyone else's, or recuse yourself from that decision point.
- **Do not write while facilitating.** The scribe role is separate (§3.4): a facilitator who is writing is not watching the room. This is non-negotiable in sessions over six people.
- **Do not answer every silence.** Silence is thinking time; the facilitator's instinct to fill it destroys the slower thinkers' contribution. Count to ten.
- **Restate, don't evaluate.** "So the proposal is to..." — neutral restatement confirms capture and comprehension without endorsing.
- **The dual-role problem.** The architect-facilitator with strong opinions (usually the case) must choose per session: facilitate (and visibly park your own views) or participate (and hand the process to a co-facilitator). For high-stakes decision workshops — technology selection, build-vs-buy — bring in a neutral facilitator or a co-facilitator who can run the process while you argue your case. The International Association of Facilitators (IAF) professionalises exactly this role; its competency framework is the reference body's articulation of the craft ⚠ (the IAF site's competency pages were not extractable this pass — see §13).

### 7.2 Energy Management

A workshop is a performance with an arc: energy rises into divergence, peaks at insight, and must be deliberately managed through convergence and close. The craft:

- **Design for the energy curve.** Heavy analytical work in the morning; generation after breaks; decisions when the room is freshest for the people who must make them; the close short and crisp. Never schedule the hardest decision for the last hour of a long day.
- **Breaks are part of the agenda.** A 90-minute work block, then 10 minutes — every time. Brains in workshops consume at meeting rate; the break is a maintenance stop, not a luxury.
- **Change the physical mode to change the mental mode.** Standing at the wall, small groups, solo writing, pairs — each mode shift resets attention. The worst energy pattern is four hours of plenary discussion.
- **Watch the room, not your notes.** Drooping postures, phones appearing, side conversations — these are the room telling you the current block is over-running or under-designed. Adjust the agenda live (that is what the time reserve is for).
- **End on a high.** The last ten minutes — decisions read back, wins named, thanks given — should feel like a landing, not a collapse. The room's memory of the workshop is disproportionately its ending.

### 7.3 The Parking Lot

The parking lot is the visible capture space for anything off-topic but too valuable to lose — the rule that lets the facilitator protect the agenda *without* dismissing people. Mechanics:

- A dedicated flip-chart page (or board section) labelled PARKING LOT, always visible.
- The rule, stated at open: "if it matters but isn't this session's objective, it goes here — and it gets reviewed before we close."
- The facilitator's script when a tangent arrives: "that's a good point and it's not this session's objective — parking it." Then *actually write it* — the capture is the respect.
- **Review the lot before close (§10.3):** each item gets a disposition — assign an owner and a follow-up, or consciously drop it (and say so). A parking lot that is never reviewed teaches the room that parking is deletion.

### 7.4 Dominant Speakers and Silent Participants

The participation problem has two sides, and both are the facilitator's job:

**Dominant speakers** — the talkers who fill every silence (often the most senior or most expert person, which makes it delicate):

- Use **structure, not confrontation**: silent individual writing (§4.2, §4.3) and round-robin capture give everyone a turn by construction — the technique does the work the facilitator would otherwise have to do personally.
- **Name the pattern neutrally**: "I want to make sure we hear everyone — let's hear from the people who haven't spoken on this one." Do not embarrass; do not let it slide.
- **Use the clock**: "we have two minutes left in this round — who hasn't spoken?" The timebox is the polite referee.
- **Deploy the rules you agreed** (§2.4): "we agreed one conversation at a time" is a group norm, not a personal attack.
- For the senior expert: privately, before or after — "the room defers to you; when you speak first, alternatives die. Can you hold your view until others have spoken?" This one conversation transforms workshops.

**Silent participants** — the quiet ones (often introverts, juniors, or people from cultures where contradicting seniors is rude):

- **Design silence in**: every technique in §4–§6 that starts with individual writing gives the quiet a voice by construction. The nominal group technique (§4.3) exists precisely for this.
- **Never cold-call to embarrass.** Invite, don't interrogate: "what did the rest of you see in that data?" after a small-group report is an invitation, not a test.
- **Harvest before plenary**: pair-share, then plenary, so the quiet person's idea enters the room through a pair, not a spotlight.
- **Watch the wall**: for some people, the sticky note is the participation — and that is legitimate participation.

### 7.5 Conflict: Technical and Political

Conflict in workshops is not the problem — *unmanaged* conflict is. The craft:

- **Separate the technical from the personal.** Technical disagreement is the workshop's raw material ("the event backbone must be Kafka because..." vs "...because the ops team can't run it"); personal attack is the line. Enforce the ground rule ("disagree with the idea, not the person") early and visibly, once — then it rarely recurs.
- **Make the disagreement explicit and structured.** Don't smooth over a real technical split: write both positions on the wall, give each its evidence, and run them through the decision matrix (§6.3). Structured comparison converts a fight into an analysis.
- **Surface the undiscussable.** The most damaging conflicts are the ones not spoken in the room — the sponsor who has already decided, the team that hates the vendor. The facilitator's job is to name the elephant carefully and neutrally ("there seems to be a view in the room that this option is already decided — is that right?"). This is where neutrality (§7.1) earns its keep.
- **Escalate by design.** If a conflict is genuinely unresolvable in the room — values, not facts — the facilitator's move is not to force consensus but to record both positions, escalate to the accountable owner (§6.4), and timebox the decision elsewhere. Forcing a fake consensus in the room poisons the follow-through.

### 7.6 The Facilitator's Kit (The Minimal Set)

A facilitator needs no more than: the agenda with timeboxes, the ground rules, sticky notes and markers (or the virtual board), the parking lot page, a dot-vote supply, and — the most important tool — the discipline to say "we are diverging now" and "we are converging now" out loud. Everything else in this guide is detail on those two sentences.

---

## 8. Remote and Virtual Facilitation

Virtual workshops are not physical workshops over a worse medium — they are a different instrument with different physics: no body language, no side conversations, no wall everyone can see, but also better capture, easier breakouts, and async participation. The craft:

### 8.1 The Board Is the Room

The shared whiteboard (Miro, Mural, FigJam, or equivalent) *is* the wall — its setup determines the session's capacity exactly as the physical room does (§2.3). Design the board before the session with the same care as the agenda:

- One **frame per agenda block** (open, diverge, sensemake, converge, close, parking lot), laid out in reading order so participants can always see where the session is.
- Pre-built artefacts: the empty MoSCoW quadrants, the decision matrix with criteria rows, the journey-map skeleton, the fishbone with bones labelled.
- A **legend** and a **navigation note** ("start here → follow the numbers"); participants lost on a 50-frame board are participants who have left the session.
- Colour discipline: one colour per person for voting, one colour family per theme — the board's visual noise is the virtual equivalent of a messy wall.
- A **producer/co-facilitator** for sessions over six: one person cannot facilitate, watch chat, run breakouts, and manage access simultaneously (§3.4).

### 8.2 The Virtual Session Arc

The arc of §3.1 survives, with adjustments:

- **Open is longer and more deliberate.** Five minutes of camera-on check-in ("one word on how you're arriving") is not small talk; it is the substitute for the physical room's social glue. State the objective, show the artefact, agree the camera rule.
- **Techniques that need silence work better virtually** — silent writing, then posting to the board, then round-robin reads. The NGT structure (§4.3) is almost native to the online board.
- **Plenary discussion is the weakest virtual mode.** Convert it to small-group breakouts (2–5 people, 10–15 minutes) with a concrete deliverable, then report-back. Breakouts are the virtual replacement for the energy of side conversations — use them aggressively.
- **Close is shorter but mandatory**: read back decisions and actions on camera, show the board's final state, name the follow-up. The virtual room disperses instantly; the close is the only glue.

### 8.3 Breakouts, Done Properly

Breakout mechanics are the highest-leverage virtual skill:

1. **Give each breakout a task, an output, and a timebox** — written on the board before the session ("Group A: dot-vote the NFR list → agree top three → one slide of rationale. 15 minutes."). Unstructured breakouts return nothing.
2. **Assign roles in each room** (a timekeeper, a reporter) — role assignment is what keeps breakout groups from being one-person shows.
3. **The facilitator visits rooms** — briefly, neutrally, to unstick, not to steer.
4. **Report-backs are structured and timeboxed**: one minute per group, only what's new, capture to the board as they speak. Nothing kills a virtual workshop like six five-minute report-backs in a row.
5. **Harvest on the board, not in chat**: the board is the shared memory; chat is where ideas go to die.

### 8.4 Async Boards: The Workshop That Never Meets

The async board extends the workshop in time: pre-work lands asynchronously (§3.3), and post-workshop follow-up continues on the same board. Use it for:

- **Pre-work harvest** — the "write your top five concerns by Wednesday" ask, collected on the board before the session.
- **The distributed retrospective** — a retro board open for 48 hours before the synchronous session, so remote and timezone-shifted participants contribute on their own schedule; the session then spends its time on clustering and decisions, not on extraction.
- **Post-workshop capture and follow-through** — the artefact lives on the board with the action items (§10); the board becomes the working document, not a photo of one.

The async board's discipline is the same as the physical wall's: everything visible, everything captured, nothing lost — and the parking lot is a real frame (§7.3).

---

## 9. The Retrospective Formats

### 9.1 The Retrospective Canon

The retrospective — the team's regular, deliberate look back at its own way of working — became a cornerstone of software practice with the agile movement (the Agile Manifesto's principle: "at regular intervals, the team reflects on how to become more effective, then tunes and adjusts its behavior accordingly" ✅ — Atlassian's retrospective guide quotes the 2001 manifesto principle; audit #15). The two books that define the format canon:

- **Norm Kerth, *Project Retrospectives: A Handbook for Team Reviews*** (Dorset House, first published **2001**) ✅ — the founding handbook, home of the **timeline** activity and of the **Prime Directive**: "Regardless of what we discover, we understand and truly believe that everyone did the best job they could, given what they knew at the time, their skills and abilities, the resources available, and the situation at hand" ✅ (sources: Open Library record; retrospectivewiki.org; funretrospectives.com — audit #16).
- **Esther Derby and Diana Larsen, *Agile Retrospectives: Making Good Teams Great*** (Pragmatic Bookshelf, **July 2006**, ISBN 9780977616640) ✅ — the activity cookbook, structured around five phases (Set the Stage, Gather Data, Generate Insights, Decide What to Do, Close the Retrospective), with the **Timeline** (5.1), **Mad Sad Glad** (5.4), **Five Whys** (6.3), **Fishbone** (6.4), **Prioritize with Dots** (6.6), **+/Delta** (8.1) and others — confirmed against the book's own table of contents ✅ (source: pragprog.com and the extracted TOC/activities PDFs — audit #17).

All four formats below follow the same shape: **gather data (diverge) → generate insights (sensemake) → decide what to do (converge) → close**, with the Prime Directive read (or better, paraphrased — the words matter less than the belief) before data gathering begins.

### 9.2 Start-Stop-Continue

**What it is.** Three columns: what the team should **start** doing, **stop** doing, and **continue** doing. It is one of the most widely used retrospective formats in practice (documented by Atlassian among others ✅); its precise origin is **not attested** in any primary source found this pass ⚠ — treat it as a folk format of agile practice, not a credited invention (audit #18).

**When it fits.** The default lightweight format: iteration retrospectives, early in a team's retrospective practice, when the team needs a fast, low-risk structure. Its three-way frame nudges balanced reflection — the "continue" column protects the things that are working, which pure problem-focused formats forget.

**Mechanics.** Three labelled columns on the wall/board → silent individual writing (2–3 per column) → round-robin posting → cluster duplicates → dot-vote the top items in each column → convert the top "start" and "stop" items into owned actions. Watch the common failure: "stop" items that are actually complaints about other teams — re-frame them as things *this* team can control.

### 9.3 The 4L: Liked, Learned, Lacked, Longed For

**What it is.** Four columns: what I **liked**, what I **learned**, what I **lacked**, and what I **longed for**. The format is widely used in agile retrospectives and is commonly attributed to the Derby & Larsen activity family, but the exact name does **not** appear in the table of contents of *Agile Retrospectives* as verified this pass ⚠ — attribute it cautiously ("the 4L is a widely used agile format") rather than to a specific book (audit #19).

**When it fits.** Teams that have outgrown start-stop-continue and need more texture: the "learned" column surfaces skills and insights; "lacked" invites resource/tooling complaints in a structured way; "longed for" opens the door to aspirations the team hasn't voiced. Good for quarterly or release retrospectives, and for teams doing their first deeper reflection.

**Mechanics.** Four columns; silent generation first; round-robin; cluster; discuss the *surprises* (the room's attention should go to what is unexpected, not what is predictable); dot-vote and convert to actions. The "longed for" column is where the boldest improvement ideas usually hide — do not let it die in the parking lot.

### 9.4 The Timeline

**What it is.** The team reconstructs the period under review as a chronological wall of event cards — memorable, meaningful, significant events, one per card — then analyses the resulting picture for patterns, energy changes, and insights. It is the signature activity of Kerth's *Project Retrospectives* (2001) ✅ and appears as the first data-gathering activity in Derby & Larsen (5.1, with colour-coding variations: feelings, events, functions, themes, swim lanes) ✅ (audit #16–#17).

**When it fits.** Release and project retrospectives, longer iterations, incident retrospectives — anywhere the span is too long for the memory of any single person and the room needs the *collective* reconstruction. Its strength is that it surfaces what different people experienced differently — the "I had no idea that happened in week three" moment that generates the deepest insights.

**Mechanics (after Derby & Larsen's runbook).** Paper the wall (or prep the board frame) → small groups (max five) write event cards, one per card → post in rough chronological order → the whole team walks the timeline, adding cards as memories surface → analyse: where were the energy highs and lows? where do patterns cluster? what do different people remember differently? → feed the insights into the decide-what-to-do phase. Colour-coding (blue = sad/mad/bad, green = satisfied/successful, yellow = cautious, etc.) layers emotional data onto the factual timeline ✅.

### 9.5 Mad-Sad-Glad

**What it is.** Three columns of emotional data: what made me **mad**, what made me **sad**, what made me **glad** during the period. It is a documented activity of Derby & Larsen's *Agile Retrospectives* (activity 5.4, "Mad Sad Glad") ✅ and is also cited in practice guides such as Atlassian's (audit #17, #20).

**When it fits.** Teams that need to surface *feelings* about the work before they can discuss the facts — common after a hard iteration, a difficult release, or an incident, or when a team's retrospective has become too clinical. The emotional frame is not softness; it is data: mad items point to process failures and boundary violations, sad items to loss and disengagement, glad items to what must be protected.

**Mechanics.** Three columns → silent writing (encourage *specific* items: "mad: the deploy failed three times on Friday" beats "mad: deploys") → round-robin → cluster and discuss — spend the discussion time proportionally on mad and sad, and *protect* the glad column from being talked over → dot-vote the top items → convert to owned actions. The facilitator's job is to keep "mad" from becoming a blame festival: the Prime Directive read beforehand is what makes this format safe.

### 9.6 Choosing the Format

| Format | Best when | Duration | Data type |
|---|---|---|---|
| Start-Stop-Continue | Lightweight iteration retro; new teams | 30–60 min | Behavioural (do/stop doing) |
| 4L | Deeper reflection; quarterly/release | 60–90 min | Mixed (feelings + learning + gaps) |
| Timeline | Long spans; releases; incidents | 90 min–half day | Chronological facts + feelings |
| Mad-Sad-Glad | Emotional recovery; hard periods | 45–75 min | Emotional |

The format is a means: the retrospective's real output is the **decide-what-to-do** phase — a small number of owned, dated, tracked improvements (§10). A retrospective that ends in feelings but no actions is a support group, not a workshop; a retrospective whose actions are never tracked is theatre. The follow-through cadence of §10.4 applies to retrospectives with extra force: the team's belief in the ritual depends on seeing last time's actions land.

---

## 10. Outputs and Follow-Through: Minutes, Actions, Decisions

The workshop's value is realised after the room disperses. This section is the follow-through discipline; it cross-references the repository's decision-capture conventions in `../technology/architecture_decision_record_guide.md` rather than re-deriving them.

### 10.1 Minutes and the Artefact

- **Capture on the day.** The wall/board becomes the artefact the same day — the facilitator or scribe transcribes, photographs (physical), or exports (virtual) within hours, while context is fresh. A workshop's capture left for a week is a workshop half-lost.
- **One page of minutes that matters.** The minutes are not a transcript; they are: the objective as stated, the decisions taken (with who and when), the actions (owner + due date), the parking-lot dispositions, and the artefact links (the board export, the matrix, the photos). Structure follows the pyramid principle (answer first — see `management_consulting_skills_guide.md` §3.3): decisions and actions at the top, detail below.
- **Distribute within 24 hours**, to participants and to the named stakeholders who need to know (§6.4's Informed column). The minutes are the workshop's contract with the organisation: what was decided, what happens next.

### 10.2 Action Items: The Discipline

- **Every action has an owner and a due date** — written in the room, read back in the close, restated in the minutes. An action without an owner is a wish; without a date it is a wish with a deadline.
- **Keep actions small and concrete**: "draft the NFR v0.2 for the backbone" beats "work on the NFRs". Small actions get done; large actions get scheduled into next quarter's workshop.
- **Track to closure.** The follow-up cadence (§10.4) exists to close actions; an action that survives three check-ins is a project that needs its own plan, not a workshop action.

### 10.3 The Parking-Lot Review

The parking lot (§7.3) must be *disposed of*, not just emptied:

- In the close, or within 48 hours: every item gets a disposition — **owner + follow-up** (a real item that was off-topic), **folded in** (absorbed into an action or decision), or **dropped consciously** (recorded as considered and dismissed, with the reason).
- The dispositions go into the minutes. The parking lot is the workshop's honesty channel: it proves the agenda was protected *and* that nothing valuable was lost. A reviewed parking lot also feeds the next workshop's agenda — many of the best future sessions start as parking-lot items.

### 10.4 Decisions and the Follow-Up Cadence

- **Decisions become records, not memories.** A decision workshop that does not produce a decision record has produced a rumour. The repository's convention — the ADR log with numbered, immutable records and explicit statuses, per `../technology/architecture_decision_record_guide.md` §1–§5 — is the house standard: the workshop produces the decision content (context, options, trade-offs from the Pugh matrix, the choice, the consequences), and the ADR is drafted from the workshop's artefact within days, then walks the status lifecycle (Proposed → Accepted via the review gate, §9.2 of the ADR guide). This guide deliberately does not re-derive the template, the statuses, or the log conventions — it points at them.
- **The follow-up cadence.** One week: actions reviewed at the team's standing meeting; decisions drafted as ADRs. Two–four weeks: a short follow-up check-in on open actions and parked items; the next workshop's agenda absorbs the residue. The cadence's purpose is to close the loop the workshop opened — the workshop's credibility is the rate at which its outputs become real.
- **The retrospective loop.** The retrospective's actions get the same cadence with extra visibility (§9.6): the team's next retrospective *starts* by reviewing the previous one's actions — that review is the ritual that makes continuous improvement believable.

---

## 11. The Worked Example: The Cymbal Bank Architecture Design-Workshop Series

This section shows the whole method on one case: a **Cymbal Bank** architecture design-workshop series for the payments modernisation programme. Cymbal Bank is the repository's recurring fictional bank persona; the dates, names, and governance frame are pedagogical constructions, consistent with the conventions used across this repo (see audit #21 and §13) — the *method* is the point. The series follows the double diamond (§4.1): discovery workshops (first diamond), then design workshops (second diamond), with decision capture feeding the ADR log per the repo's ADR conventions (`../technology/architecture_decision_record_guide.md`).

### 11.1 The Situation and the Series Design

**Situation.** Cymbal Bank's payments platform is a 15-year-old stack: a legacy message broker, point-to-point integrations, batch settlement. The programme sponsor (Head of Payments) wants a modern event backbone; the engineering leads disagree on the technology; risk and operations worry about the migration; nobody agrees on what "modern" means. The architect's diagnosis (per §1.4's wrong-instrument test): the information *is* in the room, the decision is genuinely open, and the interdependence is real — this is a workshop series, not a memo.

**Series design — one workshop per diamond phase, designed backwards from the decisions needed:**

| # | Workshop | Type (§1.2) | Output | Decisions it feeds |
|---|---|---|---|---|
| W1 | Discovery: "How payments actually works today" | Discovery | Current-state map, pain points, constraint inventory, stakeholder journey maps | Problem framing (Define) |
| W2 | Requirements: "What the backbone must do" | Requirements elicitation | MoSCoW'd requirement baseline incl. NFRs | Scope baseline |
| W3 | Design: "Backbone options" | Design | Pugh-compared option shortlist with trade-offs | ADR-0001 (backbone choice) |
| W4 | Decision: "Commit and sequence" | Decision + planning | Accepted ADR-0001, POC plan, migration sequencing, RACI | ADR-0001 (accepted), ADR-0002 (POC scope) |
| W5 | Retrospective (post-POC) | Retrospective | Improvement actions, validated assumptions | ADR-0003 (follow-on decisions) |

Each row answers §1.3's question: the instrument matches the output. W1–W2 are the first diamond (Discover/Define); W3–W4 the second (Develop/Deliver); W5 closes the loop (§9).

### 11.2 The Facilitation Plan (W3, the Design Workshop)

The plan for W3 — the highest-stakes session — shows the design discipline of §2–§3 applied completely:

- **Objective:** "By the end of this 4-hour session we will have a Pugh-compared shortlist of event-backbone options with explicit trade-offs, ready to feed an ADR — and we will *not* have decided the winner (that is W4's job)." The objective deliberately separates design (W3) from decision (W4) — the classic architect's discipline of not converging before the analysis is complete (§4.1).
- **Measurable outcomes:** (1) a completed Pugh matrix on the board with all criteria rows filled and tallied; (2) a one-page option summary per candidate (captured same-day); (3) an agreed list of open questions requiring vendor input; (4) the W4 decision agenda.
- **Participants (11):** the sponsor (decision authority, present but coached to speak last — §7.4), the two disagreeing engineering leads (the conflict is the raw material — §7.5), the platform ops lead, the risk/compliance representative, a vendor architect (observer, seated apart), the security architect, two senior engineers, the programme manager, and the architect as co-facilitator alongside an external neutral facilitator (the architect has a strong view on the technology and therefore does *not* also own the process — §7.1's dual-role rule).
- **Ground rules:** the §2.4 starter set, plus a session-specific rule: "no option is rejected in this room today; every option gets its row in the matrix."
- **Pre-work (5 days ahead):** the option-analysis pack (three candidates, each with architecture sketch, cost/effort band, ops implications, risk notes); the NFR draft from W2; the W2 MoSCoW baseline; an async-board ask: "add your top three selection criteria to the board by Wednesday" (this harvest produces the criteria rows and pre-empts criteria-rigging — §6.3).
- **Room/setup:** cabaret seating, three metres of wall papered with the pre-drawn matrix (criteria rows from the async harvest), the MoSCoW baseline, and a large parking lot frame; scribe assigned; the external facilitator owns process, the architect owns content.
- **Agenda (4 hours, ~50 min reserve):** open and objective (20') → criteria review and finalisation (20') → silent option critique: each candidate gets 15' of silent sticky-note critique by the whole room, round-robin captured (50') → matrix scoring rounds, one candidate at a time against baseline (60') → sensitivity pass: flip-test the top two (20') → open questions and vendor asks (15') → close: read-back, actions, parking-lot review (25'). Total work: 210'; the reserve absorbs the inevitable overrun.

### 11.3 Technique Selection, and Why

- **NGT-style silent critique (§4.3)** for the option evaluation: the two engineering leads are both dominant speakers *and* the people whose views matter most — silent individual critique before any plenary argument guarantees the junior engineers' concerns enter the matrix too, and converts the leads' expected clash into structured input.
- **Pugh matrix (§6.3)** as the spine of the session: it forces criteria-before-advocacy, keeps the comparison coarse-but-honest (+/–/S against baseline), and produces the auditable evidence an ADR needs. The baseline is the incumbent legacy broker — so "keep the incumbent" is a scored option, not a sacred cow.
- **MoSCoW (§6.1)** from W2 governs the matrix's must-have criteria: any candidate failing a Must row is flagged, not summed away — the matrix's known failure mode (§6.3 disadvantages) is guarded by keeping Musts outside the score.
- **Five whys / fishbone (§5.4–§5.5)** appear in W1's discovery workshops for the recurring pain points (e.g., "why do settlement breaks happen?" → root causes in batch design and missing idempotency), and their outputs become W2 requirements.
- **Journey mapping (§5.2)** in W1 for the payments journey (merchant → Cymbal Bank → clearing → beneficiary), producing the touchpoint inventory that later frames the integration architecture.
- **Dot voting (§6.2)** inside W2 to triage the 60+ candidate requirements before MoSCoW-ing the survivors; **RACI (§6.4)** in W4 to settle who owns the POC, who approves the ADR, and who must be consulted on migration sequencing.
- **SCAMPER (§6.6)** in W3's warm-up: a 15-minute forced-move round on the *incumbent* design ("what if we eliminate the batch window? what if we reverse the settlement sequence?") — deliberately before the matrix, to open the room's thinking past the three pre-cooked candidates.

### 11.4 The Decision Capture, Feeding the ADR Log

The series' decisions follow the repo's ADR conventions (`../technology/architecture_decision_record_guide.md` §3–§5 — template, statuses, numbering — are *not* re-derived here; this section shows the workshop-to-record handoff):

- **W3 produces the evidence; W4 produces the record.** The Pugh matrix, the option summaries, and the sensitivity-pass notes become the ADR's Context and considered-options material. The architect drafts **ADR-0001: Adopt Kafka as the payments event backbone** within 48 hours of W4, status *Proposed*, with the matrix as its evidence trail.
- **The review gate.** The ADR walks the repo's decision-review gate (ADR guide §9.2): the Architecture Review Board reviews it with the W4 RACI's Accountable owner named; the disagreement that W3 structured ("Kafka vs the ops team's comfort") appears in the ADR's Context as a force, and the accepted version records the consequence (ops capacity plan) — which becomes an action item with an owner and date (§10.2).
- **The supersession chain, workshop-shaped.** When the POC (W4's plan) later produces findings, the retrospective (W5) and a follow-on decision workshop generate **ADR-0002: Schema registry with Avro** and — after the migration completes — **ADR-0003: Retire the legacy message broker**, superseding ADR-0001's retention clause in part (the full chain mechanics are the ADR guide's §7 and §11 — the workshops are where the decisions get made, the log is where they get kept).
- **Every decision workshop's close includes the read-back** ("the decisions taken today: one — the backbone technology, recorded as ADR-0001 Proposed; two — POC scope; three — NFR validation owner...") — the room hears its own decision before it disperses, which is what makes the record uncontested later.

### 11.5 The Follow-Through (W4 onward)

- **Same-day capture:** the matrix, the option summaries, and the action list transcribed to the programme's documentation space; the board export linked from the minutes.
- **The 24-hour minutes:** decisions (three, with the ADR number), actions (eight, each with owner and date, e.g., "Ops lead: capacity plan for broker operations, due +2 weeks"), parking-lot dispositions (four items: two given owners, one folded into the NFR backlog, one dropped with reason).
- **The cadence (§10.4):** actions reviewed at the programme's weekly stand-up; ADR-0001 drafted within 48 hours, accepted at the ARB two weeks later; the POC plan kicked off with its RACI; the W5 retrospective scheduled for the week after POC results, its agenda seeded with the POC's open questions.
- **The retrospective loop:** W5 opens by reviewing the W4 actions — the POC validation points that were closed, the two that slipped (with five-whys on why), the ops-capacity plan that landed. Its output: three improvement actions, including the one that becomes ADR-0002's trigger.

The series is the method end to end: the right instruments in the right order (§1), designs built backwards from decisions (§2–§3), techniques matched to dynamics (§4–§6), a facilitator separated from the content (§7), the decision trail kept by the repo's ADR conventions (§10), and a follow-up cadence that closes every loop. A room that works like this is worth more to an architecture programme than any diagram the architect could draw alone.

---

## 12. The Claims Audit

The house rule of this repository: never present an unverified claim as fact. This audit lists every load-bearing factual claim in the guide, with its verification status. **✅ = verified this pass at a primary or authoritative source** (URLs given); **⚠ = flagged** (single-source, not re-verified, or held from the author's knowledge base); **❌ = disputed or false as stated** (correction given in the text).

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | Double Diamond: developed by the Design Council's Design & Innovation team (Eisermann, White, Vanstone, Wildman, Winhall, Ball) answering the 2003 "How do we describe design process?" challenge; shared at conferences from 2004; popularised via the 2005 *Eleven Lessons: A Study of the Design Process*; adapted from Bánáthy's 1996 divergence-convergence model | ✅ | designcouncil.org.uk/resources/the-double-diamond/history-of-the-double-diamond/ ("In 2004, Design Council started to share the Double Diamond"); designcouncil.org.uk framework-for-innovation page ("Launched in 2004"); en.wikipedia.org/wiki/Double_Diamond_(design_process_model) (2005 popularisation, Bánáthy 1996) |
| 2 | Design Council founded 19 December 1944 as the Council of Industrial Design (COID) by Hugh Dalton; renamed Design Council 1972; charity by royal charter 1976 | ✅ | en.wikipedia.org/wiki/Design_Council (infobox: "Founded 19 December 1944"; History section) |
| 3 | Ishikawa/fishbone: created by Kaoru Ishikawa; popularised in the 1960s; among the Seven Basic Tools of Quality Control; basic concept from the 1920s | ✅ | en.wikipedia.org/wiki/Ishikawa_diagram ("popularized in the 1960s by Kaoru Ishikawa"; "basic concept was first used in the 1920s") |
| 4 | MoSCoW: developed by Dai Clegg (Oracle UK) in 1994 for RAD; core DSDM practice; DSDM Consortium founded 1994 (BA, AmEx, Oracle, Logica et al.); DSDM first released 1994 | ✅ | en.wikipedia.org/wiki/MoSCoW_method ("developed by Dai Clegg in 1994"); en.wikipedia.org/wiki/Dynamic_systems_development_method (consortium founded 1994; "First released in 1994") |
| 5 | Brainstorming: introduced by Alex F. Osborn in *Applied Imagination: Principles and Procedures of Creative Thinking* (1953); Osborn co-founded BBDO | ✅ | en.wikipedia.org/wiki/Applied_Imagination; regent.edu journal article; openlibrary.org (all confirm 1953) |
| 6 | Five Whys: originally developed by Sakichi Toyoda, used within Toyota Motor Corporation; described by Taiichi Ohno ("the basis of Toyota's scientific approach…") | ✅ | en.wikipedia.org/wiki/Five_whys (History section; Ohno quotation) — no firm year is attested; the guide deliberately does not date it |
| 7 | Pugh method/decision matrix: invented by Stuart Pugh; earliest traced publication Pugh (1981) "Concept selection: a method that works", ICED Rome | ✅ | en.wikipedia.org/wiki/Pugh_method (reference: S. Pugh 1981, in Hubka (ed.), *Review of Design Methodology*, ICED Rome 1981) |
| 8 | NGT: developed by Delbecq & Van de Ven; first published 1971 (*Journal of Applied Behavioral Science* 7(4): 466–91); extended in Delbecq, Van de Ven & Gustafson (1975), *Group Techniques for Program Planning* | ✅ | en.wikipedia.org/wiki/Nominal_group_technique, references section (primary citations verbatim) |
| 9 | Affinity diagram/KJ Method: term devised by Jiro Kawakita in the 1960s; among the Seven Management and Planning Tools | ✅ | en.wikipedia.org/wiki/Affinity_diagram ("devised by Jiro Kawakita in the 1960s… sometimes referred to as the KJ Method") |
| 10 | Derby & Larsen's fishbone activity guidance (cross-category causes are the most likely root causes; stop when causes leave the team's control) | ✅ | media.pragprog.com/titles/dlret/Activities.pdf (book extract, Activity 6.4 Fishbone, extracted this pass) |
| 11 | Journey mapping: NN/g definition (actor, scenario, phases, actions/mindsets/emotions, opportunities) | ✅ | nngroup.com/articles/journey-mapping-101/ (Sarah Gibbons, 2018, last reviewed 2026) |
| 12 | "Walt Disney created the first customer journey map" — commonly repeated claim | ❌/⚠ | No primary evidence found this pass; treated as folklore in §5.2 and §13 — the claim is contested and unverified |
| 13 | RACI: origin contested — no designated inventor attested; practice dates to around the 1950s; "responsibility assignment matrix" name from the 1970s; in use since at least the 1970s–80s | ✅/⚠ | en.wikipedia.org/wiki/Responsibility_assignment_matrix; perfony.com/en/raci-matrix-history-of-a-managerial-invention/; umbrex.com — the *contestedness* is verified ✅; the specific decades are single-source ⚠ |
| 14 | SWOT: SRI Long Range Planning Service 1965 SOFT report (Stewart, Benepe, Mitchell); Harvard design school 1965 (Learned, Christensen, Andrews, Guth); Puyt et al. (2023) call the Harvard claim an "academic urban legend"; popular Albert Humphrey attribution not confirmed in the primary trail | ✅/⚠ | en.wikipedia.org/wiki/SWOT_analysis, History section — the SRI/Harvard/Puyt trail is ✅; the Humphrey attribution is ⚠ |
| 15 | SCAMPER: developed by Bob Eberle; described in *SCAMPER: Games for Imagination Development* (1971); roots in Osborn's *Applied Imagination* checklist; 1972 *Journal of Creative Behavior* article | ✅ | en.wikipedia.org/wiki/SCAMPER (History section; Eberle 1972 JCB citation) |
| 16 | Agile retrospective principle from the Agile Manifesto (2001): "At regular intervals, the team reflects on how to become more effective…" | ✅ | atlassian.com/agile/scrum/retrospectives (quotes the 2001 principle) |
| 17 | Kerth, *Project Retrospectives: A Handbook for Team Reviews*, Dorset House, first published 2001; home of the timeline activity and the Prime Directive | ✅ | openlibrary.org (2001, Dorset House); retrospectivewiki.org and funretrospectives.com (Prime Directive text attributed to Kerth) |
| 18 | Derby & Larsen, *Agile Retrospectives: Making Good Teams Great*, Pragmatic Bookshelf, July 2006, ISBN 9780977616640; activities incl. Timeline (5.1), Mad Sad Glad (5.4), Five Whys (6.3), Fishbone (6.4), Prioritize with Dots (6.6), +/Delta (8.1) | ✅ | pragprog.com/titles/dlret/agile-retrospectives/ ("Published: July 2006"); media.pragprog.com/titles/dlret/TOC.pdf (activity list verbatim) |
| 19 | Mad Sad Glad as a documented retrospective activity | ✅ | Derby & Larsen TOC (5.4); atlassian.com (Glad/Sad/Mad) |
| 20 | Start-Stop-Continue: standard retrospective format | ✅/⚠ | atlassian.com (Start / Stop / Continue documented) — the format's *existence* is ✅; its *origin* is unattested ⚠ |
| 21 | 4L (Liked, Learned, Lacked, Longed for): widely used format; attribution to the Derby & Larsen activity family | ⚠ | Not found in the verified *Agile Retrospectives* TOC this pass; the guide attributes it cautiously (§9.3) |
| 22 | Nominal groups (individual generation) matching/beating interacting groups on idea quantity/quality — the empirical motivation for NGT | ✅ | en.wikipedia.org/wiki/Nominal_group_technique, Effects section (1958 study; Delbecq & Van de Ven findings) |
| 23 | Weihrich's TOWS matching (1982) | ⚠ | Cited via en.wikipedia.org/wiki/SWOT_analysis reference list (Weihrich 1982 marker); not independently re-verified this pass |
| 24 | Personas popularised through Alan Cooper's work in the 1990s | ⚠ | Held from the author's knowledge base; not re-verified against a primary source this pass |
| 25 | IAF (International Association of Facilitators) as the professional body for facilitation, with a published competency framework | ⚠ | iaf-world.org is the body's site ✅; the competency pages 404'd this pass — no IAF content claims are made beyond its existence |
| 26 | "1-2-4-all" participation pattern; the pre-mortem as a planning technique | ⚠ | Standard facilitation practice (Liberating Structures family for 1-2-4-all); specific attributions not re-verified this pass |
| 27 | Cymbal Bank worked example: dates, names, programme, governance frame | ⚠ | Pedagogical constructions consistent with the repo's Cymbal Bank persona (§11.1, §13); not facts about any real bank |

---

## 13. What Could Not Be Verified

The honesty section — the items this pass could not pin down, and exactly why:

- **The Disney journey-mapping claim.** The assertion that Walt Disney invented or created the first customer journey map circulates widely in marketing and CX content, but no primary or reliable secondary evidence was found this pass (the search returned nothing usable and no canonical history attests it). The guide therefore treats it as folklore and says so (§5.2, audit #12). The verified alternative is NN/g's documented definition and practice lineage (§5.2).
- **The origin of Start-Stop-Continue.** It is verifiably a standard, widely documented retrospective format (Atlassian documents it ✅), but no primary source attesting its inventor or date was found. It is presented as a folk format of agile practice (§9.2, audit #20).
- **The 4L's precise attribution.** "Liked, Learned, Lacked, Longed For" is widely used and commonly attributed to the Derby & Larsen activity family, but the name does not appear in the verified table of contents of *Agile Retrospectives* (2006) — the book may contain it under another name, or the attribution may be loose folklore. The guide attributes it cautiously (§9.3, audit #21).
- **Albert Humphrey as the originator of SWOT.** The popular attribution could not be confirmed against the primary trail found this pass, which leads to the SRI Long Range Planning Service's 1965 SOFT report by Stewart, Benepe and Mitchell (and the parallel 1965 Harvard design-school lineage). Humphrey may well have led or influenced that work — the 2023 Puyt et al. history was not read in full — but the guide does not assert it (§6.5, audit #14).
- **RACI's precise inventor and date.** Multiple sources describe the origin as unknown, with the practice emerging around the 1950s and the "responsibility assignment matrix" name attaching in the 1970s; some of that dating is single-source. The guide asserts the *contestedness* (verified) and flags the dates (§6.4, audit #13).
- **A firm year for the five whys.** The technique is attested to Sakichi Toyoda and the Toyota Production System, with Ohno's quotation, but no reliable single year of origin was found; the guide deliberately does not date it (§5.4, audit #6).
- **The SCAMPER book-year nuance.** Wikipedia gives 1971 for *SCAMPER: Games for Imagination Development* and 1972 for the *Journal of Creative Behavior* article; some secondary sources cite the book as 1972. The guide follows Wikipedia's 1971 with the 1972 article noted (§6.6, audit #15); a physical copy check would settle it.
- **IAF content.** The International Association of Facilitators' site was reachable, but its competency-framework pages returned 404 during this pass, so nothing is claimed about IAF's framework content (§7.1, audit #25).
- **The DSDM Consortium's own site.** dsdm.org now resolves to the Agile Business Consortium, whose specific historical pages 404'd this pass; DSDM/MoSCoW facts were verified via Wikipedia's well-referenced articles instead (audit #4).
- **Personas, the pre-mortem, and 1-2-4-all attributions.** All three are presented as standard practice without a verified originator (audit #24, #26) — the guide says what they do, not who invented them.
- **The worked-example details.** The Cymbal Bank series — dates, names, programme structure, governance frame — is explicitly pedagogical (§11.1, audit #27). Nothing in §11 describes a real bank's programme; Cymbal Bank is this repository's recurring fictional persona, and the ADR mini-series it references is the ADR guide's own worked example.

---

## 14. The Glossary

| Term | Definition |
|---|---|
| **Workshop** | A structured working session in which a defined group, guided by a facilitator, produces an agreed output (decision, design, plan, shared understanding) in a bounded timebox (§1.1) |
| **Facilitator** | The process owner of a workshop: method, time, participation balance, capture — without owning the content (§7.1) |
| **Sponsor / decider** | The person whose authority makes the workshop's output real; must be present for decision workshops (§2.3) |
| **Right room** | The physical or virtual environment with the capacity the session needs: writable wall/board, round seating, no interruptions (§2.3, §8.1) |
| **Divergence / convergence** | The two thinking modes of every workshop: opening up options vs closing down to a choice; they need opposite rules (§4.1) |
| **Double Diamond** | The Design Council's four-phase design-process model — Discover, Define, Develop, Deliver — two diamonds of diverge-then-converge (§4.1) |
| **Parking lot** | The visible capture space for off-topic-but-valuable items; reviewed and disposed of before close (§7.3, §10.3) |
| **Pre-mortem** | Design-time exercise: imagine the workshop has failed and work backwards to the causes, then design them out (§2.1) |
| **Timebox** | A fixed time budget within which scope flexes — the DSDM principle behind MoSCoW and agenda design (§3.2, §6.1) |
| **MoSCoW** | Must/Should/Could/Won't prioritisation, developed by Dai Clegg (1994) for RAD and core to DSDM (§6.1) |
| **NGT (Nominal Group Technique)** | Delbecq & Van de Ven's five-stage structured group process: silent generation, round-robin, clarification, private vote (§4.3) |
| **Affinity map / KJ Method** | Kawakita's technique for clustering large idea sets into named themes by natural relationship (§5.1) |
| **Journey map** | A visualisation of a person's process toward a goal: actor, scenario, phases, actions/mindsets/emotions, opportunities (§5.2) |
| **Persona** | A composite, research-grounded archetype of a user group used to anchor design to real users (§5.3) |
| **Five whys** | The Toyota Production System's iterative why-chain to a root cause (§5.4) |
| **Ishikawa / fishbone** | Kaoru Ishikawa's cause-and-effect diagram, popularised in the 1960s; groups causes into categories (§5.5) |
| **Pugh matrix (decision matrix)** | Stuart Pugh's qualitative option-comparison: +/–/S against a baseline per criterion; patterns, not summed scores (§6.3) |
| **RACI** | Responsible/Accountable/Consulted/Informed responsibility assignment; origin contested (§6.4) |
| **SWOT** | Strengths/Weaknesses/Opportunities/Threats position snapshot; SRI LRPS 1965 SOFT lineage (§6.5) |
| **SCAMPER** | Eberle's seven-prompt ideation checklist (Substitute, Combine, Adjust, Modify, Put to other uses, Eliminate, Reverse), rooted in Osborn's checklist (§6.6) |
| **Retrospective** | The regular workshop in which a team inspects its own way of working and agrees owned improvements (§9) |
| **Prime Directive** | Kerth's retrospective opening: everyone did the best job they could, given what they knew and had at the time (§9.1) |
| **ADR (Architecture Decision Record)** | The repository's decision-capture unit — Context/Decision/Consequences, numbered, immutable except for status; conventions live in `../technology/architecture_decision_record_guide.md`, cross-referenced here, not re-derived (§10.4) |
| **Action item** | A workshop output with an owner and a due date; the atomic unit of follow-through (§10.2) |
| **Scribe** | The capture role, separate from the facilitator, who turns the wall into the artefact (§3.4) |
| **Producer / co-facilitator** | The virtual-session role handling access, chat, and breakouts so the facilitator can facilitate (§8.1) |
| **Breakout** | A small-group working session (physical or virtual) with a task, output, and timebox, reporting back to plenary (§2.3, §8.3) |
| **Async board** | A shared board that extends the workshop in time: pre-work before, follow-through after (§8.4) |

---

*End of guide. Companion guides: `communication_stakeholder_management_skills_guide.md` (stakeholders and influencing), `management_consulting_skills_guide.md` (structured problem solving), `strategic_management_guide.md` (frameworks incl. SWOT/TOWS detail), `team_lead_methodologies_guide.md` (coaching and feedback), and `../technology/architecture_decision_record_guide.md` (the decision-capture conventions this guide cross-references). The workshop is where architecture is decided; the record is how the decision survives; and the room is where both happen — the discipline of the facilitated room.*
