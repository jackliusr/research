# Team Lead Methodologies and Frameworks: A Comprehensive Guide

> **Author's Note:** This guide is the dedicated deep-dive on the *practical toolkit* of models and frameworks a team lead uses day-to-day — coaching, feedback, team dynamics, motivation, goal-setting, delegation, decision-making, 1:1s, meetings, and conflict — plus the role distinctions between team lead, line manager, tech lead, Scrum Master, and engineering manager. It is written for first-line leaders in enterprise technology settings (with a Crédit Agricole CIB banking context throughout) who want the *working* models, not just the names. Where a model overlaps with a sibling guide, this guide cross-references instead of duplicating: leadership *styles* live in [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md), the IC→manager *transition* lives in [../technology/first_time_manager_guide.md](../technology/first_time_manager_guide.md), and the *project-side* frameworks live in [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md).
> *Part of the Management & Leadership Series | Author: Jack Liu Shurui — Solution Architect, Crédit Agricole CIB*
> *Related guides in this series: [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) · [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) · [360_management_guide.md](360_management_guide.md) · [the_managers_path_research.md](the_managers_path_research.md) · [strategic_management_guide.md](strategic_management_guide.md) · [grow_team_guide.md](grow_team_guide.md) · [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) · [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)*

---

## How to Use This Guide

This guide is a reference manual and a field kit. **For a quick orientation**, read Section 1 (the role) and Section 11 (which framework when). **For a specific problem** — a conflict, a demotivated engineer, a fuzzy decision — jump straight to the matching section: coaching → Section 3, motivation → Section 4, goals → Section 5, delegation → Section 6, decisions → Section 7, 1:1s and meetings → Section 8, conflict → Section 9. **For the enterprise reality check**, Section 10 grounds every framework in the bank/IT context (delivery, matrix, compliance, audit). **To see it all assembled**, Section 12 walks through a new team lead's first quarter end-to-end. Each section is self-contained; the glossary at the end defines every term in one place.

Two reading notes. First, a recurring distinction runs through the whole guide: **methodology vs framework**. A *methodology* is a complete process system (Scrum, Kanban, PRINCE2 — see [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md)); a *framework* is a single model you apply to one situation (GROW for a coaching conversation, SBI for feedback). Team leading is mostly **framework work** — small models, applied deliberately, in the moment. Second, every framework in this guide has a *verified origin* where possible; where an origin is contested or unverifiable, that is flagged honestly rather than glossed over.

---

## Table of Contents

1. [The Team Lead Role](#1-the-team-lead-role)
2. [Team Dynamics Frameworks](#2-team-dynamics-frameworks)
3. [Coaching and Feedback Frameworks](#3-coaching-and-feedback-frameworks)
4. [Motivation Frameworks](#4-motivation-frameworks)
5. [Goal-Setting Frameworks](#5-goal-setting-frameworks)
6. [Delegation Frameworks](#6-delegation-frameworks)
7. [Decision-Making Frameworks](#7-decision-making-frameworks)
8. [One-on-Ones and Meetings](#8-one-on-ones-and-meetings)
9. [Conflict and Difficult Conversations](#9-conflict-and-difficult-conversations)
10. [The Team Lead in the Enterprise](#10-the-team-lead-in-the-enterprise)
11. [Framework Selection](#11-framework-selection)
12. [Worked Example — A New Team Lead's First Quarter](#12-worked-example--a-new-team-leads-first-quarter)
13. [Glossary](#13-glossary)

---

## 1. The Team Lead Role

### 1.1 What a team lead is

The **team lead** is the **first-line leader**: the person one level above individual contributors, accountable for a small team's *delivery* and *people* simultaneously. In a typical IT organisation the team lead owns a squad of 4–10 engineers/analysts, carries delivery accountability for their stream of work, and usually still does hands-on technical work — which is why the role is so often described as **player-coach**. The team lead is the *first* leadership rung: the role where you stop being judged primarily on your own output and start being judged on the output of the team.

The transition into this role is itself a research topic with its own guide: see [../technology/first_time_manager_guide.md](../technology/first_time_manager_guide.md) for the IC→manager transition in depth (identity shift, letting go of "I can do it faster myself," the new accountability for other people's work). The one-line summary: **your job changes from doing the work to creating the conditions in which the team does the work.**

### 1.2 Team lead vs manager vs Scrum Master

The three "first-line-ish" roles are frequently conflated, and in small organisations one person may wear two hats. The clean distinction is about *focus*:

| Role | Primary focus | Typical accountability | Typical activities |
|---|---|---|---|
| **Team Lead** | Technical/people hybrid — the *player-coach* | Delivery of the team's stream + day-to-day leadership of the people in it | Design/review, delegation, coaching, unblocking, 1:1s, delivery planning |
| **Line Manager** | People/org focus | Employment relationship: hiring, performance, comp, career, exits | Performance reviews, salary/band decisions, career conversations, HR processes |
| **Scrum Master** | Process focus — the agile team facilitator | Health of the agile process: sprint mechanics, impediment removal, team self-management | Facilitation of standup/planning/retro, protecting the team, coaching the process |

Key relationships: in many banks, the **team lead and line manager are the same person** (the lead runs 1:1s, does the performance reviews, and facilitates or co-facilitates Scrum ceremonies). In larger matrix structures they are **split**: a Scrum Master (process) and a separate people manager (org) with the tech lead supplying technical direction. The Scrum Master's role is defined in the Scrum Guide (a process role, not a management rank); the team lead role is defined by the organisation. When one person holds both, the trap is letting process-facilitation crowd out people-leadership — the ceremonies get run and the humans get neglected.

### 1.3 The four responsibility domains

Every team lead responsibility falls into one of four domains. A healthy lead is strong in all four; a struggling lead usually has one glaring gap.

| Domain | What it includes | Failure mode when neglected |
|---|---|---|
| **Delivery** | Feature delivery, releases, quality, deadlines, dependencies, unblocking | Team is happy but ships nothing |
| **People** | 1:1s, coaching, feedback, career growth, motivation, conflict | Work ships but people burn out and leave |
| **Process** | Agile ceremonies, ways of working, estimation, retros, continuous improvement | Chaos or, conversely, process for its own sake |
| **Stakeholder** | Upward/sideways communication, expectations, status, negotiation for resources | Team delivers the wrong thing, or great work goes unnoticed |

The stakeholder domain is the one new leads under-invest in — see [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) for the full treatment. In a bank, delivery is non-negotiable (regulatory commitments, release windows), but the leads who get promoted are the ones who also manage the stakeholder domain well.

### 1.4 Role models: player-coach, servant-leader, leader-as-coach

Three classic role models give the team lead a self-image. They are complementary, not competing:

- **Player-coach.** The lead still codes/reviews/designs *and* leads. The value: credibility, context, empathy with the work. The danger: playing instead of coaching — doing the work yourself because it's easier than developing someone else to do it. The rule of thumb: **you should be the last resort for the work, not the first.**
- **Servant-leader.** Robert K. Greenleaf coined the term in his 1970 essay *The Servant as Leader* (verified: Greenleaf, 1970; the idea — leadership whose first instinct is to serve the needs of the team, so that people "grow healthier, wiser, freer, more autonomous" — long predates the 2010s "leader-as-coach" fashion). In team-lead terms: remove impediments, provide context and resources, protect the team from noise, then get out of the way.
- **Leader-as-coach.** The coaching leadership style — asking questions that develop people rather than telling them what to do. This is the behavioural style dimension; the *styles* themselves (directing, coaching, supporting, delegating) are covered in depth in [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md). The team lead uses the *coaching toolkit* (Section 3) as the default mode and escalates to directing only when the situation demands it (safety, crisis, or a genuinely new person).

### 1.5 Team lead vs tech lead

| | **Tech lead** | **Team lead** |
|---|---|---|
| Primary question | *What* are we building and *how*? | *Who* is building it, *how well*, and *how do they feel*? |
| Focus | Technical direction: architecture, design, standards, code quality | People and delivery: dynamics, coaching, priorities, stakeholders |
| Accountability | Technical outcomes | Team outcomes (which include technical ones) |
| Time split | Mostly technical, some leadership | More leadership, some technical |

In many orgs the two roles are one person (especially for teams under ~6 people). Where they are separate, the tech lead reports *into* or *alongside* the team lead; the team lead owns the tech lead's development as a person. For the tech-lead career track itself (architecture, the "staff+" path), see the architect career content referenced in Section 10.5.

### 1.6 The banking/IT team lead context

In a bank like Crédit Agricole CIB, the team lead operates in a *constrained enterprise context* that shapes how every framework in this guide is applied:

- **Delivery under compliance.** Releases ride regulatory and risk deadlines (Basel/Capital Markets requirements, audit findings with committed dates, production changes in controlled windows). "Fail fast" is a dev-team slogan; in the bank it is *"fail fast in non-production"* — and the lead is the one holding that line.
- **The matrix.** Most teams sit in a matrix: a functional line (the lead's reporting chain) and project/programme streams (dotted lines to programme managers). Authority is often *influence without control* — see Section 10.2 and [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md).
- **Compliance and audit are stakeholders.** The lead must treat risk, compliance, and audit as first-class stakeholders (Section 10.4), not as obstacles.
- **Process weight.** Banks run heavyweight processes (change management, incident management, SDLC gates). The lead's process job is to make the *required* process invisible to the team where possible and meaningful where it must exist.

### 1.7 Section summary

The team lead is the player-coach at the first leadership rung: accountable for delivery, people, process, and stakeholders, guided by servant-leader and leader-as-coach role models, distinct from the line manager (org focus), the Scrum Master (process focus), and the tech lead (technical direction). Everything that follows is the toolkit this role runs on.

---

## 2. Team Dynamics Frameworks

### 2.1 The Tuckman model (1965, adjourning 1977)

**Origin (verified):** Bruce Tuckman, an American psychologist, proposed the *forming–storming–norming–performing* model in 1965, based on a review of 50+ studies of group development ("Developmental Sequence in Small Groups," *Psychological Bulletin*, 1965). In 1977, with Mary Ann Jensen, he added a fifth stage, **adjourning** (sometimes called *mourning*), covering what happens when the team disbands ("Stages of Small-Group Development Revisited," 1977).

The stages:

| Stage | What the team looks like | Team needs from the leader |
|---|---|---|
| **Forming** | Polite, uncertain, dependent on the leader; roles unclear; people test the waters | Direction, structure, clear purpose, introductions, low-stakes wins |
| **Storming** | Conflict over roles, approach, and leadership; subgroups form; friction is normal and *necessary* | Containment, mediation, clarity on decisions and norms; do not suppress conflict — channel it |
| **Norming** | Agreements form: how we work, who does what; cohesion and trust build | Reinforcement of norms, delegation, shared leadership |
| **Performing** | High autonomy; the team self-manages; the leader is nearly redundant for day-to-day work | Protection from outside noise, resources, strategic context; get out of the way |
| **Adjourning** | Disbanding: project ends, people move on; can involve grief, loss of identity | Recognition, closure rituals, transition support, honest offboarding |

### 2.2 Tuckman in practice

**Stage recognition.** The model is a *diagnostic*, not a label to slap on a team. Symptoms to look for: forming — meetings where only the lead talks, everyone waits for direction; storming — raised voices in retro, complaints about "the way we work," cliques; norming — jokes, shorthand, disagreements resolved without the lead; performing — the lead is cc'd rather than required, delivery is steady. A team can *regress* (a new member, a reorg, a crisis pushes a performing team back to forming/storming) and can be in different stages on different topics.

**Leader's role per stage (interventions):**

- **Forming:** set the container. Team charter (2.4), clear goals, defined roles, an early win. Use a directive style (see [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) — S1/Directing).
- **Storming:** *do not panic*. This is the stage where teams either learn to fight productively or fracture. Surface the conflict explicitly ("I notice we disagree on X — let's settle it properly"), use the conflict toolkit (Section 9), and enforce process (who decides, how we argue). The single biggest storming failure is the lead who suppresses disagreement to keep the peace — that pushes the team into *fake norming*, which never reaches performing.
- **Norming:** codify what works. Write the working agreements down, delegate more, let the team own its process.
- **Performing:** the lead's job is protecting the boundary — buffer the team from organisational noise, bring strategic context, and manage stakeholder expectations so the team can focus.
- **Adjourning:** plan closure deliberately — a final retro, recognition, knowledge transfer, and honest 1:1s about next steps.

**The two most common Tuckman mistakes:** (1) reading storming as "this team is broken" and intervening to stop it, instead of containing and channelling it; (2) staying in directing mode through norming/performing and preventing the team from ever self-managing.

### 2.3 Lencioni: The Five Dysfunctions of a Team (2002)

**Origin (verified):** Patrick Lencioni's *The Five Dysfunctions of a Team* (Jossey-Bass, 2002) — a business fable that became one of the most widely used team-dynamics models. The five dysfunctions form a **pyramid**; each level undermines the one above:

```
        Inattention to Results          ← the outcome
        Avoidance of Accountability     ← nobody holds anyone to account
        Lack of Commitment              ← no real buy-in to decisions
        Fear of Conflict                ← no honest debate of ideas
        Absence of Trust                ← the foundation
```

- **Absence of trust** — team members won't be vulnerable with each other (admitting mistakes, asking for help). *Counter: personal histories, vulnerability exercises, the leader going first.*
- **Fear of conflict** — people hold back disagreement to avoid tension; decisions get made in corridors, not in the room. *Counter: normalise debate, "conflict norms" (2.4), the leader modelling productive disagreement.*
- **Lack of commitment** — without real debate, people don't buy in; they comply publicly and doubt privately. *Counter: clarify decisions, "disagree and commit" (Section 7.4), no silent after-the-meeting ambushes.*
- **Avoidance of accountability** — peers won't call each other out on missed commitments or slipping standards. *Counter: publish commitments, make them visible, coach peer-to-peer feedback.*
- **Inattention to results** — individual ego/team status trumps collective outcomes. *Counter: team-level goals (Section 5), shared scoreboard, celebrate team wins.*

The model's practical payoff for a team lead is diagnostic: when a team *looks* like it lacks commitment or results, the root cause is usually two or three levels down. Fix trust and conflict first; the upper dysfunctions often resolve themselves.

### 2.4 The team charter and working agreements

The **team charter** is the team's constitution, created *by the team with the lead* (usually in forming, revised in norming). A minimal charter contains:

- **Purpose:** why this team exists, who it serves, what success looks like (1–2 sentences + a success measure).
- **Roles:** who does what — including the lead's role, the tech lead's scope, ownership of key areas.
- **Norms / working agreements:** the "team contract" — how we behave. Examples: "disagreement is expected and must be voiced in the room," "no blame in retro," "anyone can call a decision-review," "we respect focus time 10:00–16:00," "on-call handover is documented."
- **Process anchors:** ceremonies, cadence, definition of done, how decisions get made (Section 7).

Working agreements are only worth anything if they are *enforced* — the lead's job is to notice violations and name them ("we agreed disagreements happen in the room — that design was changed in the corridor last week; let's redo that properly"). A charter that is written and never referenced is worse than no charter, because it teaches the team that agreements are fiction.

### 2.5 Psychological safety

**Origin (verified):** Amy Edmondson, Harvard professor, introduced **team psychological safety** in her 1999 paper "Psychological Safety and Learning Behavior in Work Teams" (*Administrative Science Quarterly* 44(4): 350–383) — defined as *a shared belief that the team is safe for interpersonal risk-taking*: admitting mistakes, asking "dumb" questions, challenging the boss, proposing half-baked ideas. Google's **Project Aristotle** (2012–2015, studying 180+ teams) later found psychological safety to be the *single most important* predictor of team effectiveness — a widely-cited finding that made the concept mainstream.

For the team lead, psychological safety is not "everyone feels comfortable" — it is *what makes learning and candour possible*. Practical levers:

- **Model vulnerability:** the lead admits errors first ("that estimate was mine and it was wrong").
- **Respond to bad news with curiosity, not blame:** "what happened?" beats "who did this?" — especially in a bank where incident post-mortems are mandatory; the lead sets the tone for blameless retros.
- **Invite dissent explicitly:** "tell me what's wrong with this plan" gets better results than "any questions?"
- **Watch your own reactions:** the lead's raised eyebrow after a status update teaches more than any stated value. Teams calibrate to what the leader *punishes*, not what the leader *preaches*.

### 2.6 Team health checks and team assessment

**The Spotify health-check model (verified as Spotify's; origin is Spotify's internal practice popularised via Henrik Kniberg's "Spotify Engineering Culture" videos/blog posts, ~2013–2014).** Spotify's squads rate themselves periodically on a set of dimensions (delivery, fun, learning, support, health of codebase, etc.) using a traffic-light scale: *"We're great at this" / "We're good but could improve" / "We're not happy here."* Each dimension gets a colour and a one-line comment; the team then picks the two or three worst items to work on next quarter.

**Running a health check:**

1. Pick 8–12 dimensions relevant to your team (Spotify's original set is a good starting point; a bank team might add *compliance burden* and *stakeholder clarity*).
2. Every team member rates anonymously (or votes with dots) — *anonymity matters*, or the lead's presence biases the results.
3. Discuss the spread, not just the average — a dimension with half "great" and half "unhappy" is a story.
4. Turn it into action: max two or three improvement items per quarter, owned by the team, tracked in the retro.

The health check is a *team assessment* instrument — the "state of the team" snapshot. It pairs naturally with Tuckman (stage diagnosis), Lencioni (dysfunction diagnosis), and the 360° practices in [360_management_guide.md](360_management_guide.md). Cadence: quarterly, plus an ad-hoc check after any significant change (new members, reorg, project kick-off).

---

## 3. Coaching and Feedback Frameworks

### 3.1 The GROW model

**Origin (verified):** GROW was developed in the UK in the **1980s** by **Graham Alexander, Alan Fine, and Sir John Whitmore** (co-creators — attributions vary; Whitmore himself credited Max Landsberg with coining the acronym). Whitmore popularised it in *Coaching for Performance*, first published **1992**, which became the foundational text of modern business coaching.

GROW structures a coaching conversation in four phases:

| Phase | Purpose | The coach's core question |
|---|---|---|
| **G — Goal** | What does the coachee want? (session goal + longer-term goal) | "What do you want to achieve from this conversation?" |
| **R — Reality** | What is actually happening now? Facts, not interpretations | "What is the current situation? What have you tried?" |
| **O — Options** | What could the coachee do? Generate, don't judge | "What options do you have? What else? Who could help?" |
| **W — Will / Wrap-up** | What will the coachee *actually* do? Commitment + support | "What will you do, by when? How will I know? What support do you need?" |

### 3.2 GROW in practice — the question bank

GROW is a *questioning* framework; its power is in the questions. A practical bank of questions per phase:

**Goal questions:** "What would a good outcome of this conversation be?" · "What do you want to be different in three months?" · "If you could wave a wand, what would change?" · "What will achieving this give you?"

**Reality questions:** "What is happening right now, factually?" · "What have you already tried?" · "What's stopping you?" · "Who is affected and how?" · "What did your last similar attempt teach you?" · "On a scale of 1–10, how committed are you to fixing this — and why not lower?" (the *scaling question* is the classic reality probe)

**Options questions:** "What are all the ways you could approach this?" · "What would you do if you had no constraints?" · "What would someone you admire do?" · "What's the worst option — and what's good about it?" · "If you asked X for help, what might happen?"

**Will questions:** "Which option will you choose?" · "What exactly will you do, and by when?" · "What could get in the way, and how will you handle it?" · "How will you know it's worked?" · "Do you want me to check in on this?"

**Mechanics that make or break a GROW session:** hold silence (the coachee's thinking is the product); one question at a time; resist the urge to answer your own question; end with a *written* commitment; take the coachee's agenda, not yours. A 30-minute GROW session typically spends ~5 min on Goal, ~10 on Reality, ~10 on Options, ~5 on Will.

### 3.3 Coaching vs mentoring

| | **Coaching** | **Mentoring** |
|---|---|---|
| Direction of answers | Solutions come *from the coachee* (coach asks) | Guidance comes *from the mentor* (mentor advises) |
| Stance | Questioning, Socratic, non-judgmental | Experience-sharing, role-modelling, opinion-giving |
| Domain | Performance on a specific goal/skill | Career/life breadth, organisational navigation |
| Relationship | Often the manager or a peer, time-boxed, goal-focused | Usually a senior person, longer-term, voluntary |
| Best for | "I know what to do but I'm stuck" | "I've never been here before — how does this work?" |

The team lead needs *both*: coach direct reports on performance and growth (GROW), and either mentor juniors directly or connect them to mentors elsewhere in the bank (an important stakeholder/network task — see [grow_team_guide.md](grow_team_guide.md)). The most common error is **mentoring when coaching was needed** — giving advice when the person already has the answer, which creates dependence and robs them of ownership. Ask first: "Do you want me to coach you through this, or give you my view?" (This question is itself a powerful moment of autonomy — see Section 4.)

### 3.4 The coaching stance: ask vs tell

**Ask vs tell.** Telling is faster and feels efficient; asking develops. The default stance for a team lead is *ask* — with two exceptions where *tell* is correct: (1) safety/compliance/regulatory constraints ("you must not do X in production — no discussion"), and (2) genuinely new situations where the person has no basis to answer (a first-week graduate facing a first incident). The Socratic method — questions that lead the other person to their own insight — is the underlying technique; **powerful questions** are open ("what," "how," "what if") rather than closed ("did you," "are you"), and they create thinking rather than extracting information.

**The coaching culture (leader-as-coach).** When the *whole leadership layer* of an organisation coaches rather than tells, three things happen: problems get solved closer to the work, people develop faster, and the "manager as bottleneck" pattern disappears. The team lead's part: model ask-first behaviour in 1:1s and retros, coach *your* tech leads and seniors to coach *their* people, and resist the organisation's pull toward tell-culture (which in a bank manifests as "I need to know everything to sign off"). Coaching culture is built one conversation at a time.

### 3.5 The SBI feedback model

**Origin (verified):** The **SBI** model — **Situation, Behavior, Impact** — was developed by the **Center for Creative Leadership (CCL)**, the US leadership-research institute, and is one of the most widely taught feedback frameworks in corporate leadership programmes.

The structure:

1. **Situation** — *where and when.* Anchor the feedback to a concrete moment: "In Tuesday's 11:00 status meeting with the front-office stakeholders…"
2. **Behavior** — *the observable, factual behavior*, with no interpretation or judgment: "you interrupted the business analyst twice while she was presenting the reconciliation approach…" (not "you were rude/aggressive" — that's an interpretation).
3. **Impact** — *the effect on you, the team, or the outcome:* "…which derailed her explanation, and I noticed two stakeholders stopped following the thread."

**Example (corrective):** "In Tuesday's status meeting [S], when the BA was explaining the reconciliation changes, you interrupted her twice to correct a detail [B]. The impact was that the room stopped following the main thread, and she went quiet for the rest of the presentation [I]."

**Example (positive):** "In Friday's release [S], you caught the margin-call calculation error before it reached the UAT environment and flagged it to the whole stream [B]. The impact was that we avoided a failed release and the team now double-checks that path [I]."

**Why it works:** SBI removes the two things that make feedback unhearable — *vagueness* ("you need to communicate better") and *judgment* ("you're disrespectful"). Behavior is what the person can actually change. CCL's guidance: state the *impact* including your own reaction honestly ("I felt frustrated" is legitimate impact data), then check the other person's perspective — the goal is closing the gap between *intent* and *impact* (a person who meant "I'm being rigorous" may have landed as "I'm shutting you down").

### 3.6 Radical Candor

**Origin (verified):** **Kim Scott**, formerly of Google and Apple (and before that a manager at Juice Software and others), published *Radical Candor: Be a Kick-Ass Boss Without Losing Your Humanity* in **2017** (St. Martin's Press). The model has since become a standard feedback vocabulary in tech.

Radical Candor is feedback positioned on two axes: **care personally** (you genuinely give a damn about the person) × **challenge directly** (you say the hard thing anyway). The four quadrants:

| | **Challenge directly: low** | **Challenge directly: high** |
|---|---|---|
| **Care personally: high** | **Ruinous Empathy** — "I care too much to hurt you," so you say nothing or soften it; the person never learns, and the team pays later | **Radical Candor** — caring + direct: "I'm telling you this *because* I believe in you" |
| **Care personally: low** | **Manipulative Insincerity** — neither care nor challenge; passive-aggression, performance, politicking ("I'm sure you'll figure it out") | **Obnoxious Aggression** — "brutal honesty" without the caring; criticism that lands as attack (Scott originally called this quadrant the "asshole quadrant") |

Practical guidance: the *default failure mode* for most new leads is **ruinous empathy** — especially in feedback-averse cultures (and enterprise banking can be one). The fix is not to stop caring but to add directness. Conversely, the *default failure mode* for high-pressure environments is obnoxious aggression dressed up as "I'm just direct." Radical Candor requires *both* axes: challenge in private where possible, praise in public; check that your "caring" is actually *felt* by the recipient (Scott's test: would the person say you care about them personally?).

### 3.7 Other feedback models: COIN, and the comparison

**COIN** — **Context, Observation, Impact, Next** — is a four-step variant used in some corporate leadership training. *Context* (the situation, like SBI's S), *Observation* (the behavior, like B), *Impact* (like I), and *Next* (the forward step: "what will you do differently / let's agree on the path forward"). Note on origin: **COIN has no single verifiable canonical source** — it appears in various training materials with slightly different expansions (some use "Consequence," some "Next steps"); unlike SBI (CCL) and Radical Candor (Scott), it is best treated as a training-house variant, not a documented model with an attributable author. Use it if your organisation teaches it; know that its value is identical to SBI's plus an explicit action step.

| | **SBI (CCL)** | **COIN** | **Radical Candor (Scott)** |
|---|---|---|---|
| Structure | Situation → Behavior → Impact | Context → Observation → Impact → Next | Two axes: care personally × challenge directly |
| Strength | Clean, minimal, highly teachable | Adds explicit forward action ("Next") | Names the *relational stance*; works for culture, not just one conversation |
| Best for | One-off corrective/praise feedback | Feedback that must end in a commitment | Ongoing feedback culture, coaching your own style |
| Origin | Verified: CCL | No canonical verified origin | Verified: Kim Scott, 2017 |

### 3.8 Feedback cadence

**Real-time beats batched.** Feedback should be given close to the event — weekly at worst, same-day at best. Batched feedback (the annual review as the *first* time someone hears a problem) is the single most destructive feedback pattern in corporate life; a bank's annual review cycle makes this worse, so the lead must decouple feedback from the review calendar entirely. The performance review should be a *summary* of conversations already had, never a surprise. (This is the "no surprises" principle, revisited in Section 9.6.)

**The feedback sandwich — a myth to retire.** The "sandwich" (praise → criticism → praise) is still taught in some organisations and is widely criticised by feedback practitioners: recipients learn to discount the praise as padding, the criticism arrives wrapped in manipulation, and it teaches people to brace. The evidence-based replacement is **direct, specific, timely feedback** — SBI for the critique, SBI for the praise, no wrapping. If the relationship is healthy, you don't need bread around the meat.

**Praise: specific, public, sincere.** Specific praise ("the way you structured the incident timeline made the post-mortem 20 minutes shorter — that's exactly the clarity we needed") outperforms generic praise ("good job") because it tells the person *what to repeat*. Praise in public where the person is comfortable with it; critique in private. And calibrate to the person — some people find public praise uncomfortable; the *caring* axis of Radical Candor means matching the delivery to the human in front of you.

---

## 4. Motivation Frameworks

### 4.1 Self-Determination Theory (SDT)

**Origin (verified):** **Self-Determination Theory** was developed by psychologists **Edward Deci and Richard Ryan**, building on Deci's 1970s intrinsic-motivation experiments; their foundational book *Intrinsic Motivation and Self-Determination in Human Behavior* appeared in **1985**, and the theory has been refined since (the landmark 2000 *American Psychologist* paper "Self-Determination Theory and the Facilitation of Intrinsic Motivation, Social Development, and Well-Being" is the standard citation).

SDT's core claim: humans have three **basic psychological needs**, and motivation quality (not just quantity) depends on how well the work context satisfies them:

| Need | Meaning | Satisfied when… | Team-lead translation |
|---|---|---|---|
| **Autonomy** | Acting from genuine choice, not control | People have latitude over *how* they do their work and input into *what* | Delegation with real authority (Section 6), options rather than orders, "you own this" |
| **Competence** | Feeling effective and growing | People get stretch work they can succeed at, plus feedback that shows progress | Right-sized challenges, skill development, visible progress, SBI feedback (Section 3) |
| **Relatedness** | Feeling connected and belonging | People feel cared for and connected to teammates and purpose | 1:1s that go beyond status, team rituals, psychological safety (Section 2.5) |

**Intrinsic vs extrinsic.** *Intrinsic motivation* — doing something because the work itself is interesting or meaningful — produces better engagement, persistence, and creativity than *extrinsic motivation* (rewards, punishment, pressure). Crucially, SDT's research shows that **extrinsic controls can crowd out intrinsic motivation**: when you attach a bonus to a task people found intrinsically interesting, or when you micromanage how it gets done, you can *reduce* motivation. This is the theoretical foundation for why "the money is fine but the autonomy is gone" is such a common resignation reason.

### 4.2 SDT in the team — the three levers

- **Autonomy (choice).** Give people choice of *how* (tools, approach, sequence), and where possible influence over *what* (let the team shape its backlog — planning is a motivation tool, not just a scheduling one). Avoid the bank's natural gravity toward sign-off culture: every approval layer between a person and their work is an autonomy tax.
- **Competence (mastery).** Ensure everyone has a growth edge — work that is hard enough to stretch but achievable. Watch for the two failure modes: *under-challenged* (a senior doing grunt work for months — the fastest quiet demotivator) and *over-challenged* (a junior drowning without support). Pair stretch with feedback and recognition of progress.
- **Relatedness (belonging).** Build team identity: shared rituals, real 1:1s, credit given publicly and *for the person's own work* (leads who habitually present team work as their own destroy relatedness fast). In a distributed bank team, relatedness requires deliberate effort — virtual coffees, cameras-on norms, in-person offsites when possible.

### 4.3 Herzberg's two-factor theory

**Origin (verified):** **Frederick Herzberg**, with Bernard Mausner and Barbara Bloch Snyderman, published *The Motivation to Work* in **1959**, based on interviews with ~200 Pittsburgh accountants and engineers. The **two-factor (motivation-hygiene) theory** is one of the most cited management theories of the 20th century (and one of the most debated — its methodology has been criticised; treat it as a useful heuristic, not a law).

Herzberg's finding: **satisfaction and dissatisfaction are not opposites on one scale** — they are driven by different factors.

- **Hygiene factors** (salary, working conditions, job security, company policy, relationships with colleagues/boss): when *adequate*, they prevent dissatisfaction — but they do **not** create motivation. When *inadequate*, they demotivate. Pay is a hygiene factor: once people are paid enough, more money stops motivating and only *unfair* money demotivates.
- **Motivators** (recognition, achievement, growth, responsibility, the work itself): when present, they *create* satisfaction and motivation. They are the factors that make people go the extra mile.

**For the team lead:** the theory's practical lesson is resource allocation. You cannot *motivate* with hygiene — competitive salary, nice office, and good conditions only buy the *right to motivate*. Motivation comes from the motivators: recognition (specific praise, Section 3.8), achievement (visible wins), growth (stretch and development), responsibility (delegation with authority, Section 6), and meaningful work. A team lead controls almost none of the hygiene factors but almost all of the motivators — which is exactly why the role matters so much to retention.

### 4.4 Motivation in practice: purpose, mastery, autonomy

**Origin (verified):** **Daniel Pink's *Drive* (2009)** synthesised the research (SDT prominently) into the popular formula for intrinsic motivation: **Purpose, Mastery, Autonomy** — people are most motivated by work that serves a meaningful purpose, offers the chance to get better at something (mastery), and is self-directed (autonomy). Pink's contribution is packaging the research for practitioners; his "20% time" anecdotes and the "if-then rewards backfire" claims are the popularised versions of the SDT crowding-out finding.

**The purpose lever in banking.** Bank IT teams can struggle with purpose ("we're maintaining a 20-year-old core system"). The lead's job is to connect the work to its meaning: the reconciliation fix that prevents a client from losing money; the migration that lets the bank meet a regulatory deadline; the tooling that saves the ops team 1,000 hours a year. Purpose framing is not spin — it is surfacing the real impact that the daily grind obscures.

### 4.5 Motivation pitfalls: the demotivators

The fastest way to destroy motivation is not to fail at motivating — it is to actively demotivate. The classics:

- **Micromanagement** — the single most cited motivation killer (and the mirror image of the delegation failures in Section 6). It signals "I don't trust you," destroys autonomy, and teaches people to stop thinking.
- **Unfairness** — perceived inequity (uneven workload, credit taken, rules applied selectively) is a powerful demotivator; fairness perceptions are *comparative* (people compare themselves to teammates), so the lead must be visibly even-handed.
- **Moving goalposts** — changing priorities so often that effort feels wasted; the antidote is visible goal-setting (Section 5) and honest communication when priorities *must* shift.
- **Ignoring input** — asking for opinions and never acting on them teaches people to stop offering them (a quiet killer of psychological safety too).
- **Hero-worship of overwork** — rewarding the person who burns out first sends the message that sustainable work is for the uncommitted.

### 4.6 Engagement and disengagement

**Employee engagement** is the practical, survey-measured cousin of the motivation research: engaged employees are psychologically invested in their work and organisation, and engagement correlates with retention, quality, and safety outcomes (the Gallup Q12 survey is the best-known instrument). The team lead's engagement levers are the ones already covered — clarity of expectations, feedback, recognition, growth, connection, purpose.

**Disengagement and "quiet quitting."** The term **quiet quitting** became mainstream in **2022** (popularised by a viral TikTok and then by Gallup data showing ~50% of US workers "not engaged" — note: the term's popularisation is verifiable to 2022; the underlying phenomenon of disengaged-but-present employees is as old as work itself). Quiet quitting is *not* quitting the job — it is quitting the *discretionary effort*: doing exactly the job description, no more, no less. The lead's correct response is not to shame the behaviour ("people here go above and beyond") but to diagnose *why* the person stopped investing: usually one of the demotivators in 4.5, a fairness perception, a broken manager relationship, or a genuine work-life rebalancing after burnout. Address the cause; the discretionary effort follows.

---

## 5. Goal-Setting Frameworks

### 5.1 OKRs: origin and mechanics

**Origin (verified):** **Objectives and Key Results** is generally attributed to **Andy Grove**, who introduced it at **Intel** in the **1970s** (originally as "iMBOs" — Intel Management By Objectives) and documented it in his 1983 book ***High Output Management***. **John Doerr** learned OKRs at Intel in the 1970s, took them to Google in **1999**, and popularised them globally in ***Measure What Matters*** (**2018**). (Doerr's book is the source of the famous "OKRs are the what and the how" framing and of the anecdote that Grove taught OKRs in a 1975 Intel course Doerr attended.)

**The mechanics:**

- **Objective** — a *qualitative*, inspiring statement of direction: *"Make the payment-hub migration the smoothest change the bank has ever run."* It is not a number; it is a direction with a heartbeat.
- **Key Results** — *quantitative*, measurable outcomes that prove the objective: *"Reduce manual reconciliation effort by 40%," "Zero data-loss incidents in the first month," "All 12 trading desks trained by end of quarter."* Typically **3–5 KRs per objective**; each KR must have a number and a date.
- **Cadence** — quarterly objectives, reviewed monthly (or biweekly); the annual cycle is too slow for most delivery teams, weekly is too fast for outcomes.
- **Stretch** — Grove/Doerr's OKRs are meant to be *aspirational*: a "committed" OKR is 100% expected; a "stretch" OKR is 70% success being a good quarter. Scoring is honest ("we hit 0.6 on this KR — here's why") and feeds the next quarter's planning.

**Example (banking team):**

> **Objective:** Make trade-confirmation exceptions visible and near-zero.
> **KR1:** Reduce confirmation exceptions older than T+1 from 120/day to < 20/day.
> **KR2:** Alerting for exception queues live in production by end of Month 1.
> **KR3:** 90% of exception owners trained on the new triage process.
> **KR4:** Post-migration audit review passed with no high-severity findings.

### 5.2 OKR vs KPI

| | **OKR** | **KPI** |
|---|---|---|
| Nature | Change — where we're going | Health — how we're doing |
| Answer | "What do we need to *become* this quarter?" | "Are we *staying* OK?" |
| Target | Stretch, aspirational, often missed partly | Baseline, must-hold |
| Cadence | Quarterly, reset each cycle | Continuous, monitored |
| Failure meaning | "We aimed high and learned" (if honest) | "Something is broken, fix it now" |

The practical rule: **KPIs are the guardrails; OKRs are the steering.** A delivery team needs both — KPIs (SLA compliance, defect rate, environment stability, on-call load) protect the present while OKRs move the team into the future. The classic mistake is converting a KPI into an OKR ("reduce incidents by 5%") — that's not a direction, that's a monitor with a target. Conversely, teams that run OKRs without KPIs discover the steering wheel was attached to nothing (they made the migration "amazing" while production quietly degraded).

### 5.3 OKR pitfalls

- **Cascading problems.** Top-down OKR cascade ("here are your KRs, do them") reproduces command-and-control with new vocabulary, destroys the autonomy that motivates (Section 4), and produces KRs nobody believes in. The fix: *negotiated alignment* — the team proposes its OKRs against the org's objectives; the lead's job is to make the org's intent legible, not to dictate the team's numbers.
- **OKR theater.** Teams that game the system (easy KRs to look good, KR-juggling to always land 1.0, or writing OKRs that are never read again after the planning meeting) turn OKRs into paperwork. The antidote is honest scoring and *actual use*: if the OKRs don't shape what the team works on, they are decor.
- **Too many.** One objective per team per quarter (two at most) is the norm; three objectives with 15 KRs is a wishlist, not a focus device. OKRs exist to create *focus* — the word "no" is their main product.

### 5.4 SMART goals

**Origin (verified in usage, contested in attribution):** **SMART** goals — **Specific, Measurable, Achievable, Relevant, Time-bound** — are universally attributed to **George T. Doran**, who published "There's a S.M.A.R.T. way to write management's goals and objectives" in *Management Review* (**1981**). (Note: the acronym's expansion has varied over the decades — some use A = Attainable/Assignable/Agreed, R = Realistic/Results-based, T = Timely/Trackable; Doran's original was Specific, Measurable, Assignable, Realistic, Time-related. The S-M-A-R-T spelling is verifiable to Doran 1981; the "Achievable/Relevant" variant is the modern standard.)

SMART is a *quality filter* for individual and short-horizon goals: "improve code review speed" fails; "reduce median review turnaround from 3 days to 1 day by end of next quarter" passes. The A and R letters matter most in practice — *Achievable* keeps goals from being fantasy (contrast with OKR stretch), and *Relevant* forces the "why does this matter" test.

**SMART vs OKR.** They are not rivals; they operate at different levels. OKRs express *direction and outcomes* (qualitative objective + quantitative KRs, quarterly, stretch). SMART disciplines *individual commitments and short tasks* (a specific task, a personal development goal, a sprint-level target). A healthy stack: org/team OKRs (quarterly) → individual SMART goals derived from them (reviewed in 1:1s) → sprint goals (Section 5.5). Using SMART for a team's quarterly direction produces timid, box-ticking goals; using OKR for an individual's two-week commitment produces ceremony.

### 5.5 Goals in the team: sprint goals, quarterly goals, alignment

- **Sprint goals.** Each sprint/iteration should have one crisp goal ("Get the reconciliation dashboard to UAT-ready") that gives the sprint a spine — see [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md) for the agile mechanics. A sprint without a goal is just two weeks of tasks.
- **Quarterly team goals.** The team's OKRs (or, in a non-OKR org, its quarterly commitments) are set in a planning session with the team *before* they are presented upward — the lead negotiates upward, the team owns the how.
- **Alignment: team goals → org goals.** The lead's alignment job is to translate org strategy into team meaning: "the division is under pressure on data-quality audit findings; here's what that means for our quarter" — and to protect the team from conflicting org signals (two sponsors with contradictory priorities is a classic matrix problem; see Section 10.2 and [strategic_management_guide.md](strategic_management_guide.md)).

### 5.6 Goals practice: 1:1 goal-setting and progress reviews

Goals live or die in the *review rhythm*, not the planning meeting:

- **Goal-setting 1:1** — once per quarter (or at year start): co-create the person's SMART goals from team OKRs, and their personal development goals (often *not* the same thing — one is delivery, one is growth).
- **Progress reviews** — monthly in the 1:1: "where are you against your goals; what's blocking; what changed?" This is a *coaching* conversation (Section 3), not an audit.
- **The annual review** — should be a *summary of the year's already-happened conversations* (the no-surprises principle, Sections 3.8 and 9.6). If the annual review contains new information, the goal-setting system has failed.

---

## 6. Delegation Frameworks

### 6.1 Delegation: the leader's core skill

Delegation is not a task-distribution tactic; it is **the mechanism by which the team grows** — "delegate to grow," not "delegate to offload." Every task you keep is a growth opportunity you took away from someone else. Three truths every lead must internalise:

1. **Delegation is development.** The point of handing a task to a junior is not that they do it as well as you — it's that they *become* someone who can.
2. **You are the bottleneck you complain about.** If the team's throughput depends on your review/approval/decision on everything, you haven't delegated; you've distributed the typing.
3. **Delegation is not abandonment.** It is handing over *responsibility* with *authority*, *context*, and *checkpoints* — three things that make the difference between delegation and dumping.

### 6.2 The delegation ladder

**Note on origin:** there is no single canonical "delegation ladder" — the idea of graduated delegation levels appears in many management traditions (related to situational leadership, to "empowerment" literature, and to consulting models like Appelo's *Delegation Poker*). The common five-level ladder below is a synthesis of the standard versions found in management practice; treat the *levels* as the verified content and the specific five-step wording as a common convention rather than a single attributed model.

| Level | Name | What it means | When to use |
|---|---|---|---|
| **L1** | **Do as told** | I specify the task and the method; you execute. | New juniors; safety/compliance-critical steps; first-time tasks |
| **L2** | **Research and recommend** | Investigate, come back with a recommendation; I decide. | Building judgement; medium-risk decisions; "learn this area for me" |
| **L3** | **Recommend and act** | Decide and act on your recommendation unless I object within a window. | Capable team members; routine decisions with reversibility |
| **L4** | **Act and report** | Act; tell me what you did and the outcome. | Experienced people on known ground; low-stakes, high-volume work |
| **L5** | **Full autonomy** | You own it end-to-end; I only want to hear about exceptions and strategy. | Seniors/staff on their core competency; the goal state for any delegated area |

The ladder's two uses: (1) *diagnose* — for each recurring task, ask "what level am I actually delegating, and what level is this person ready for?"; (2) *communicate* — naming the level in the handoff ("this is a Level 3 for you — act unless you hear otherwise by Thursday") removes the single biggest delegation ambiguity: *how much authority do I actually have?*

### 6.3 Situational delegation

Delegation level should follow the person's **competence and commitment** on the *specific task* — which is exactly the SLII logic. See [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) for the full model; the delegation summary:

- **Low competence, low commitment** (new, unsure) → L1–L2: direct, structure, teach, and build confidence with small wins.
- **Low competence, high commitment** (enthusiastic beginner) → L2: high direction, high support — coach the *how* while feeding the enthusiasm.
- **High competence, low commitment** (capable but demotivated/bored) → L3–L4: less direction, more support — the *delegation itself* (real responsibility) is often the motivation fix.
- **High competence, high commitment** → L5: full autonomy, keep the feedback loops on.

The cardinal situational error: delegating to *competence* while ignoring *commitment* (giving L5 autonomy to a capable but disengaged person is how things quietly rot), or delegating at the person's *general* level rather than their level *on this task* (a staff engineer is L5 on platform work and L1 on their first stakeholder negotiation).

### 6.4 Delegation mistakes

- **Micromanagement** — delegating the task but retaining control of every step (checking in hourly, demanding pre-approval of each decision). It communicates distrust and collapses the ladder to L1 regardless of what you said. Micromanagement usually comes from the lead's anxiety, not the person's capability — fix the anxiety, or the person leaves.
- **Dumping** — handing over the task without context ("why this matters"), authority ("what you can decide"), or support ("who to ask"). Dumping sets people up to fail and then confirms the lead's belief that "I have to do it myself." The antidote is the delegation conversation (6.5).
- **Unclear authority** — "you handle it" with no agreed level, so the person either freezes (afraid to overstep) or improvises (and oversteps). Always name the level and the checkpoints.
- **Reverse delegation** — see 6.6.
- **Delegating only the work you dislike** — delegating the interesting work too is how you develop people; delegating only drudgery teaches the team that "promotion means more chores."

### 6.5 The delegation conversation

A good handoff is a 10-minute conversation with four parts — **what / why / how-much-autonomy / checkpoints**:

1. **What** — the outcome, in measurable terms: "Own the environment refresh for the UAT region: environments upgraded, verified, documented."
2. **Why** — the context that makes judgement possible: "The refresh unblocks the September release train; if it slips, the whole stream slips. This is also the first time you'd own a full environment change — I want you to have this on your CV."
3. **How much autonomy** — name the ladder level explicitly: "This is Level 3 — you decide and act; if you need to change the cutover date, that's a Level 2: bring it to me first."
4. **Checkpoints** — the trust-but-verify rhythm: "Quick check Tuesday and Friday, 15 minutes, unless something's on fire — then you call me, not the other way round. Escalation path if stuck: me, then the platform team."

**Trust but verify.** Checkpoints are not micromanagement; they are *agreed transparency*. The frequency should follow risk: daily check-ins on a production cutover, a Friday note on a routine task. The person reports *outcomes and exceptions*, not steps — if the checkpoint becomes a step-by-step status report, the delegation has silently collapsed back to L1.

### 6.6 Reverse delegation (upward delegation)

**Reverse delegation** (also called *upward delegation*) is the pattern where a task delegated to the team comes *back* to the lead — "can you just check this?," "I need your approval to proceed," "what would you do here?" — often one decision at a time, until the lead is again doing everyone's work. It is usually *learned*: people reverse-delegate because past behaviour taught them the lead will take it back, decide faster, or punish mistakes.

**How to break it:**

- **Notice the pattern:** count how many times per week a question comes back that the person *could* have answered. If it's several, you have a reverse-delegation loop.
- **Redirect with the ladder:** "You've got the context on this — it's a Level 3. What's your recommendation? Go with it unless I flag something by EOD."
- **Refuse the rescue:** when someone brings you a problem, your default response is not the answer — it is "what do you think?" (the coaching stance, Section 3.4). Only escalate to telling when it's genuinely their first time or genuinely critical.
- **Reward initiative:** when someone decides and acts, acknowledge it ("good call on the rollback decision") — otherwise the team learns that initiative is punished and safety is in the lead's inbox.
- **Watch your own behaviour:** reverse delegation often starts with the lead's impatience ("I'll just do it, faster"). Every time you take a task back, you train the team to hand it up.

---

## 7. Decision-Making Frameworks

### 7.1 DACI

**Origin (verified):** **DACI** — **Driver, Approver, Contributor, Informed** — was **developed at Intuit in the 1980s** as a decision-focused variant of the RACI matrix (multiple independent sources confirm the Intuit origin; specific authorship within Intuit is not consistently documented). It is now widely used in product and tech organisations.

The four roles:

| Role | Meaning | Count | Notes |
|---|---|---|---|
| **Driver** | Runs the decision: gathers input, builds the proposal, drives it through | 1 | The engine, not the owner — has no final say |
| **Approver** | The single person with authority to say yes/no | 1 | Breaks ties; accountable for the decision's outcome |
| **Contributor** | Provides input and expertise; can be consulted | Several | Their views matter; they do not decide |
| **Informed** | Needs to know the outcome (not consulted) | Many | One-way communication after the decision |

**Why DACI works:** it assigns *decision rights* explicitly, which kills three pathologies at once — the meeting that decides nothing (no Approver), the decision made without the people who must implement it (no Contributors), and the surprise announcement (no Informed). In practice: document the DACI on any significant decision ("decision memo: who is D/A/C/I"), circulate it, and the debate gets faster because everyone knows their role in it.

### 7.2 RACI

**Origin note:** **RACI** — **Responsible, Accountable, Consulted, Informed** — is the older, task-oriented cousin of DACI. Its precise origin is **not verifiable to a single source** (it emerged from 1950s–1970s responsibility-charting practice; some trace the lineage to a 1979 *Management Review* article on "responsibility charting" by R. W. White; it was popularised through project-management bodies like PMI/PRINCE2/ITIL rather than a named author). Flagged honestly: the acronym is standard, the attribution is not.

| Role | Meaning | Count |
|---|---|---|
| **Responsible (R)** | Does the work | 1+ (usually one; "R" does the doing) |
| **Accountable (A)** | Answers for the outcome; the "A" can only be one person | 1 |
| **Consulted (C)** | Two-way: gives input before the work/decision | Several |
| **Informed (I)** | One-way: told after the fact | Many |

RACI is task/work-oriented ("who does what on this deliverable") whereas DACI is decision-oriented ("who decides this"). In practice many organisations use them interchangeably or in combination — RACI for the work breakdown, DACI for the decision log. The eternal RACI discipline: exactly one **A** per row, and if you can't fill a row's R, the task has no owner (a favourite of auditors, incidentally).

### 7.3 RAPID

**Origin (verified):** **RAPID** — **Recommend, Agree, Perform, Input, Decide** — was developed by **Bain and Company** and introduced to a wide audience in the **2006 Harvard Business Review** article **"Who Has the D? How Clear Decision Roles Enhance Organizational Performance"** by Paul Rogers and Marcia Blenko (the "D" being the Decide role — the famous punchline "if you don't have a D, you don't have a decision").

| Role | Meaning | Notes |
|---|---|---|
| **Recommend** | Develops the proposal: analysis, options, the actual work of the decision | Often 80% of the effort |
| **Agree** | Must sign off before the decision proceeds — a *veto* role, deliberately narrow | Typically legal/compliance/risk in a bank — and precisely *because* they veto, they must be engaged early |
| **Perform** | Executes once decided (may differ from the recommender) | Named so that decisions don't die after the meeting |
| **Input** | Advises; their views are sought but they have no veto | The difference between Input and Agree is the crux of RAPID |
| **Decide** | The one accountable person who makes the call | One D per decision, always |

RAPID's distinctive contribution: it makes **veto rights explicit** (Agree) and separates *who recommends* from *who decides*. In a bank, RAPID maps naturally onto governance: the D is the accountable owner (e.g. the programme sponsor), the Agree are risk/compliance/security, and the discipline of *naming the D before the debate* is itself the value.

### 7.4 DACI vs RACI vs RAPID — comparison

| | **DACI** | **RACI** | **RAPID** |
|---|---|---|---|
| Origin | Intuit, 1980s (verified) | Not verifiable to a single source | Bain, 2006 HBR (verified) |
| Unit of analysis | Decisions | Tasks/deliverables | Decisions |
| Core innovation | Driver (facilitator) + single Approver | Single Accountable ("one A per row") | Explicit veto (Agree) + named Decider + Performer |
| Best for | Product/tech decisions among a small team | Work breakdowns, process mapping, audit evidence | Big, cross-functional, governance-heavy decisions |
| Weakness | Roles can drift without a decision log | Confuses with DACI; no decision mechanism | Heavyweight for routine calls; Agree role can stall if engaged late |
| Bank context | Feature/epic decisions within a squad | SDLC task ownership, audit documentation | Change governance, risk sign-offs, architecture decisions |

**The pragmatic pattern used by strong leads:** RACI for the work (who does what, one A per deliverable), DACI or RAPID for the decisions (one D per decision, vetoes named), and — regardless of which alphabet — a **decision log** (what was decided, by whom, when, and who was informed) so that "who decided this?" has a one-line answer. In an audit-heavy environment the decision log is your evidence trail.

### 7.5 Decision types and the Vroom-Yetton model

**Decision types** — the four common modes a lead can choose from:

| Mode | Meaning | Use when |
|---|---|---|
| **Autocratic** | I decide alone (with whatever input I choose) | Crisis, trivial calls, compliance-mandated |
| **Consultative** | I get input from the team, then I decide | Most team decisions: input matters, speed and accountability matter more |
| **Consensus** | Everyone must agree before we proceed | High-stakes, high-commitment-needed decisions where buy-in is the product |
| **Democratic** | Vote/majority | Low-stakes preference calls (meeting times, tooling taste) |

**Origin (verified):** **Vroom-Yetton** (Victor Vroom and Philip Yetton, *Leadership and Decision Making*, University of Pittsburgh Press, **1973**; later extended with Arthur Jago) is the normative model that tells you *which mode to pick given the situation* — a decision tree based on seven questions: *Does the decision require quality? Do I have enough information? Is the problem structured? Would subordinate acceptance matter for implementation? If I decided alone, would they accept it? Do they share the org's goals? Is conflict among them likely?* Roughly: *quality* pushes toward more input; *acceptance-need* pushes toward participation; *shared goals* push toward delegation. The model's verified value is the *questions*, which are worth memorising as a checklist even if you never use the formal tree.

The modern, lightweight version of the same idea is the **"consult, then decide" default**: for most team decisions the lead consults widely and decides crisply — the *speed of the decision* and *clarity of who decided* are part of the decision's quality. Consensus is reserved for the few decisions where it is genuinely the right tool (team norms, definition of done, how we run our retros), because consensus is expensive and its absence of dissent is often just exhaustion.

### 7.6 Disagree and commit

**Origin (verified as an Amazon principle):** **"Disagree and commit"** is a formal Amazon Leadership Principle; Jeff Bezos described it in his **2016 shareholder letter** (explaining that after real debate, once a decision is made — even against your recommendation — you commit wholeheartedly, with no sabotage and no "I told you so"). The practice is older than the principle (it descends from general management discipline), but the Amazon codification is the verifiable anchor.

**The two-stage discipline:**

1. **Disagree hard, early, in the room.** Before the decision, dissent is *obligation*: if you see a problem, you owe the team your view. This is the conflict that Lencioni's "fear of conflict" dysfunction suppresses (Section 2.3) and that psychological safety enables (Section 2.5).
2. **Commit fully after.** Once the decision is made (by the D/Approver), the debate is over — in public and in private. No passive resistance, no "that was never my idea," no quiet re-litigation in corridors.

For the team lead, disagree-and-commit is the *behavioural contract* of the decision frameworks: DACI/RAPID tell you who decides; disagree-and-commit tells everyone how to behave before and after. A team that masters it can argue ferociously in the room and work seamlessly afterwards — the signature of a high-performing team.

### 7.7 Decision-making in the team: proposal-driven decisions and the authority matrix

**Proposal-driven decisions.** The strongest pattern for team-level decisions: *whoever wants a decision to happen writes a one-page proposal* (problem, options, recommendation, DACI roles, timeline). The team lead's role shifts from "decider" to "proposal editor and approver." Benefits: the thinking happens before the meeting (meetings become decision points, not thinking time), the quiet people get a vehicle (a proposal can be read and commented on async), and the decision log writes itself. This is the same discipline as a design doc / ADR — see [../technology/product_management_guide.md](../technology/product_management_guide.md) for the artefact side.

**The authority matrix ("who decides").** Every organisation has an informal map of who can actually decide what — usually *not* the org chart. The lead's job: (1) *know* the map (who approves budget, who owns a system, who must be informed before a release — see [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) for stakeholder mapping); (2) *make it legible* to the team (publish "decisions we can make vs decisions that need X"); (3) *push the boundary upward* where the team is ready — the lead who grows the team's authority grows the team. And in the matrix (Section 10.2), the authority matrix is usually *negotiated*, not granted — two sponsors with conflicting priorities means the lead must force the decision up to the shared boss rather than absorbing the conflict into the team.

---

## 8. One-on-Ones and Meetings

### 8.1 The 1:1 — the leader's most important meeting

The **one-on-one (1:1)** is routinely called the single most important meeting a leader has. **Origin (verified):** the *Manager Tools* podcast — **Mark Horstman and Mike Auzenne**, whose "Manager Tools" (launched 2005, one of the most-downloaded management podcasts ever) made the **weekly, 30-minute, employee-agenda 1:1** the canonical prescription: every direct report, every week, 30 minutes, and *the employee's agenda, not the manager's*. The podcast's slogan is that the 1:1 is "the single most important thing you can do for your career" (manager's career — because it is where retention, development, and performance are actually managed). The same prescription appears independently in most management canon, including Andy Grove (Section 8.3), so the *practice* is universal even though the specific "30 minutes, weekly" formula is the Manager Tools signature.

**Why it matters:** the 1:1 is the only recurring meeting where the *person* is the agenda. Everything else (status, delivery, process) has other venues; the 1:1 is where you learn what is actually going on — the thing that will make the person quit, the skill they want to build, the conflict they won't raise in a group. It is also the venue for most coaching (Section 3), feedback (Section 3.5), and goal review (Section 5.6).

### 8.2 1:1 formats — the four modes

A healthy 1:1 rotates through four modes (Manager Tools' version is "status, coaching, career, feedback" — the exact framing varies by source but the four needs are constant):

| Mode | Focus | Typical trigger / share of the hour |
|---|---|---|
| **Status** | What's happening, what's blocked, what's next | The smallest share (status belongs in the standup); use it only to spot problems |
| **Coaching** | A problem the person owns; GROW conversation (Section 3.1) | 30–50% — the default mode |
| **Career** | Where they're going, what they want, growth plan | Weekly touch + quarterly deep-dive |
| **Feedback** | Giving and receiving feedback (Section 3.5–3.8) | As needed, in the moment, not saved up |

**The employee's agenda.** The 1:1 belongs to the employee. The lead's job is to make that real: send the invitation with *"agenda: yours — bring anything you want to talk about"*, resist the urge to fill the time with your own updates, and if the person comes with nothing, *ask better questions* (8.3) rather than defaulting to a status dump. People who come unprepared are usually not disengaged — they've been trained by leads who always took over the meeting.

### 8.3 1:1 questions

**Origin (verified):** **Andy Grove's *High Output Management* (1983)** — the same book that documented OKRs — contains the canonical 1:1 guidance: the 1:1 is **the subordinate's meeting**, and the manager should ask open questions, take notes, and let the subordinate set the pace. Grove's signature question for opening a 1:1 is **"What's on your mind?"** — open, non-leading, and impossible to answer with a status report.

A question bank for the four modes:

- **Openers:** "What's on your mind?" · "What's been the best and worst part of your week?" · "What should we talk about today?"
- **Status with depth:** "What's stuck that I should know about?" · "What are you worried about that isn't in the tracker?" · "What did you learn this week?"
- **Coaching:** the GROW bank (Section 3.2) — "What would you like to be different?" · "What have you tried?" · "What's one step you could take this week?"
- **Career:** "Where do you want to be in two years?" · "What work makes you lose track of time?" · "What skill would you like to grow next — and what would you use it for?" · "What would you like to stop doing?"
- **Feedback/health:** "What should I do more of, less of, or stop doing?" · "What's one thing the team does that frustrates you?" · "On a scale of 1–10, how's your energy at work — and what would move it up a point?"

The discipline: *ask, then listen*. The 1:1's value is proportional to how much the employee talks. A useful self-check: in the average 1:1, who talks more? If the answer is "me," the meeting is a status meeting wearing a 1:1 costume.

### 8.4 1:1 cadence: weekly vs biweekly, and skip-levels

- **Weekly vs biweekly.** Manager Tools prescribes weekly 30 minutes for every direct report, and the logic is retention: issues are caught while small (a person doesn't go from happy to resigning in a month; they go in small steps over months — weekly 1:1s catch the steps). Biweekly (or monthly) is defensible for very large teams and for seniors who genuinely prefer it, but the default should be weekly, and *consistency* matters more than length: a 30-minute weekly beats a skipped 60-minute. Never cancel a 1:1 for your own convenience — that is the fastest way to teach the team the meeting is about you.
- **Skip-level 1:1.** A **skip-level** is a 1:1 between a manager's manager and the people two levels down (e.g. the department head meets the leads' reports), done periodically (quarterly is common) and *transparently* — the intermediate manager knows it happens and ideally hears a summary of themes (not verbatim content). Purpose: the anti-silo and anti-filter — it lets leadership hear ground truth that middle layers soften, and it gives juniors a sense of the org beyond their team. The team lead's role: facilitate the skip-level (schedule it, brief the boss on context, never punish what surfaces), and treat the themes as data, not as a betrayal of the chain of command.

### 8.5 Team meetings: standup, planning, retro

The recurring team meetings are the agile delivery machinery; the mechanics (ceremonies, roles, cadences) are covered in [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md). The team-lead-specific angles:

- **Standup (daily).** The daily synchronization: what did I do, what's next, what's blocked. Team-lead discipline: *the standup is for the team, not for the lead's status report* — the lead speaks last, keeps it short, and takes blockers off-line ("let's take that to a side conversation after"). In a bank with distributed teams, keep it on a fixed time and camera-on-if-comfortable; the standup is also a daily *tone check* (who's quiet, who's carrying everything — see Section 2.1 storming/performing symptoms).
- **Planning (iteration/quarter).** Where goals meet backlog (Section 5.5): the team commits to a sprint/quarter goal, negotiates scope, and the lead's job is to protect realism from stakeholder optimism. Planning is also an *autonomy* moment (Section 4.2) — let the team own estimation and commitment.
- **Retro (retrospective, end of iteration).** The improvement ceremony: what went well, what went wrong, what we'll change. Team-lead discipline: retro is *blameless* (psychological safety, Section 2.5 — in a bank, the incident post-mortem culture can leak blame into retros; the lead's job is to quarantine it), *action-producing* (every retro ends with 1–3 owned, dated actions), and *followed up* (an ignored retro teaches the team that improvement is theater — the same lesson as OKR theater, Section 5.3).

### 8.6 Meeting design: agenda, outcomes, no-meeting days

Most meeting pain is design pain, not people pain. The four design rules:

1. **Every meeting has an outcome.** State it in the invite: "Outcome: decide on the environment strategy (DACI: D = platform lead)". If you can't write an outcome, the meeting shouldn't exist — replace it with a doc (8.7).
2. **Agenda before, not during.** A shared agenda item list, populated before the meeting; the meeting starts with the agenda on screen and ends on time. The "last 5 minutes = parking lot for new topics" rule keeps scope creep out.
3. **Right people, right size.** The invite list is a decision (Section 7.7): contributors who must be in the room, informed people who get the notes. Fewer attendees, shorter meetings, better decisions — and more people free to do work.
4. **No-meeting days.** A recurring "focus block" (e.g. Thursday afternoons, or a full no-meeting day) protects deep work. In a bank, the meeting calendar will fight you (governance forums, committees, cross-team syncs); the lead's job is to defend the team's deep-work windows with the same seriousness as a release date.

### 8.7 Meeting hygiene: async-first and documentation

- **Async-first.** Default to asynchronous communication: a written proposal, a doc with comments, a recorded demo, a decision log entry — then use the meeting only for what sync genuinely adds (debate, decision, alignment). For a team in different time zones (common in a global bank), async-first is not a preference, it's a necessity. The test: *would the decision be better, worse, or the same without the meeting?* If the same, cancel it.
- **Documentation.** Every significant meeting gets a note with *decisions, owners, and dates* — not a transcript. The team lead models this (notes published within the hour) and the team follows. Documentation is what makes the decision log (Section 7.4), the no-surprises culture (Section 9.6), and audit readiness (Section 10.4) possible; an undocumented decision is, to the rest of the org, a rumour.

---

## 9. Conflict and Difficult Conversations

### 9.1 Healthy vs unhealthy conflict

Conflict is not the problem — *unmanaged* conflict is. The research distinction that matters:

- **Task conflict** — disagreement about *what* and *how* (approach, design, priority, interpretation). Healthy in moderation: it produces better decisions (Lencioni's point in 2.3 — without debate, commitment is fake) and is a sign of psychological safety (Section 2.5). The lead should *invite* task conflict and keep it task-focused.
- **Relationship conflict** — personal friction: distrust, disrespect, personality clashes. Nearly always corrosive; it hijacks attention, kills information flow, and is what people actually mean by "there's conflict in the team."

The lead's diagnostic: when a team fights about *ideas*, that's fuel; when it fights about *people*, that's fire. The management of the first is debate discipline (norms, disagree-and-commit, Section 7.6); the management of the second is direct intervention (below). The warning sign of the transition: arguments start referencing the person ("you always…", "that's typical of…") instead of the work.

### 9.2 The Thomas-Kilmann model (TKI)

**Origin (verified):** The **Thomas-Kilmann Conflict Mode Instrument (TKI)** was developed by **Kenneth W. Thomas and Ralph H. Kilmann in 1974**, based on the earlier managerial-grid work of Blake and Mouton. It remains the standard conflict-styles model.

Five modes, defined by two axes — *assertiveness* (pursuing your own concerns) and *cooperativeness* (pursuing the other's concerns):

| Mode | Assertiveness / Cooperativeness | When it's the *right* choice |
|---|---|---|
| **Competing** | High / Low — I win | Crises, compliance, unpopular-but-necessary calls (a bank lead *must* compete on audit deadlines) |
| **Collaborating** | High / High — we find the solution that serves both | High-stakes issues where both sides' concerns genuinely matter; the expensive, highest-quality mode |
| **Compromising** | Medium / Medium — split the difference | Time pressure, equal power, moderately important issues; acceptable when perfect is impossible |
| **Avoiding** | Low / Low — withdraw | Trivial issues, cooling-off moments, when the other party is too heated to hear |
| **Accommodating** | Low / High — I yield | When the issue matters more to them, when you're wrong, or when preserving the relationship is the point |

The TKI's practical lesson is **flexibility, not a favourite mode**: each mode is a *tool*, and the skilled lead (and the skilled team) switches modes deliberately. The common failure patterns: a lead who *always* competes (teaches the team to avoid raising anything), a lead who *always* accommodates (accumulates resentment and teaches the team that conflict doesn't resolve), and a team stuck in *avoiding* (the Lencioni fear-of-conflict dysfunction). In the team, the TKI is also a *diagnostic vocabulary* — naming modes ("we're both competing here; do we need this to be collaborating?") de-escalates fast.

### 9.3 Crucial conversations

**Origin (verified):** *Crucial Conversations: Tools for Talking When Stakes Are High* — **Kerry Patterson, Joseph Grenny, Ron McMillan, and Al Switzler, 2002** (with later editions) — is the standard practical toolkit for high-stakes, high-emotion, opposing-opinion conversations (exactly the ones a team lead has daily).

The core mechanics:

- **Start with heart.** Before the conversation, get clear on *what you actually want* (for yourself, for the other person, for the relationship) — and what you *don't* want. If you can't answer, don't start the conversation.
- **Make it safe.** People go silent or violent when they feel unsafe. Safety is restored by *mutual purpose* ("we both want this release to succeed") and *mutual respect* ("I respect you even though we disagree"). When the other person shuts down or attacks, the first move is not to argue the content — it's to restore safety.
- **Master your stories.** Between what someone *does* and what you *feel* lies the *story you tell yourself* about their intent ("they're undermining me" vs "they're under pressure too"). Own the story, separate fact from interpretation (the same discipline as SBI's Behavior step, Section 3.5).
- **STATE your path:** Share your facts (start with the least controversial, most observable facts), Tell your story (your interpretation, owned as such — "I'm worried that…"), Ask for their path (invite their view), Talk tentatively (your conclusion is a hypothesis, not a verdict), Encourage testing (make it safe to disagree with you).
- **Explore their path.** When they're emotional, *listen*: ask, mirror ("you seem frustrated"), paraphrase, prime ("I'm guessing you're thinking…") — then agree where you agree, build where you differ, compare (not attack) where you conflict.

### 9.4 Difficult conversations

**Origin (verified):** *Difficult Conversations: How to Discuss What Matters Most* — **Douglas Stone, Bruce Patton, and Sheila Heen, 1999** (Viking; the authors are from the Harvard Negotiation Project — the same lineage as *Getting to Yes*). The book's core claim: every difficult conversation is actually **three conversations in one**:

1. **The "what happened" conversation** — disagreement about facts and, deeper, about *who's right, who meant what, and who's to blame*. The reframe: drop "who's right" (both perspectives are valid data), separate *impact* from *intent* ("I know you didn't intend X; the impact was still Y"), and drop the blame frame for the *contribution* frame (how did each side's actions contribute to the situation?).
2. **The feelings conversation** — the emotions underneath, which are not the enemy but the *content*; unexpressed feelings leak into the conversation anyway (tone, body language, passive-aggression). Naming them ("I'm frustrated, and I suspect you're frustrated too") disarms them.
3. **The identity conversation** — what this conversation says about *who you are* ("am I competent? am I a good person?"). Identity threat is why otherwise-rational people go rigid; the fix is to notice it in yourself and keep the conversation about behaviour, not character.

The practical lead application: before a hard conversation, run the three-part prep — *what's my story about what happened (and how else could it be read)? What am I feeling, and what are they likely feeling? What am I defending about myself?* Then enter with curiosity and a learning stance. The book's famous closing advice applies to every conflict a lead faces: **"The goal is not to win the conversation; it's to get the conversation right."**

### 9.5 Performance conversations: underperformance, PIP, managing out

Performance problems are the hardest conflict because they combine identity threat (9.4), power imbalance, and real organisational stakes. The disciplined sequence:

1. **Diagnose before you label.** Underperformance has a small number of root causes: *skill* (can't do it — train), *will* (won't do it — motivation, Section 4), *environment* (can't do it *here* — tooling, blockers, unclear goals), or *mismatch* (wrong role for this person). Most early-career leads mislabel will-problems as skill-problems or vice versa. Ask: *would this person do this well if their life depended on it?* If yes → will/motivation; if no → skill/support.
2. **Coach for improvement first (SBI + GROW).** Give clear, specific feedback (SBI, Section 3.5), agree the expected standard and the timeline (SMART, Section 5.4), and coach the path (GROW, Section 3.1). The person must *know* there is a problem and what "fixed" looks like — in writing where the stakes are high. This is the no-surprises principle in action: no one should ever be surprised by a PIP.
3. **The PIP (performance improvement plan).** The **Performance Improvement Plan** is the formal, documented step: specific gaps, measurable targets, review checkpoints, support offered, and consequences if targets are missed. Note on usage: PIPs are *misused* in many organisations as disguised exits ("manage them out"), which poisons trust in the instrument. Used honestly — as a structured, supportive, time-boxed attempt to turn performance around — the PIP is a fair and useful tool. Used dishonestly, it is a betrayal of the no-surprises culture.
4. **Managing out.** If the PIP fails (or the behaviour is egregious — fraud, harassment, gross misconduct, which in a bank are instant and non-negotiable), the lead's job becomes *managing the exit humanely and compliantly*: with HR and legal (in a bank, always), with respect for the person's dignity, and with the team handled afterwards (the team will watch *how* the exit happens and draw conclusions about the organisation's fairness — Section 4.5). "Managing out" is the final act of the performance process, not an alternative to it: done right, it is a *candidate* for termination handled with honesty; done wrong, it is termination by ambush.

### 9.6 Conflict in practice: address it early, no-surprises

- **Address it early.** Every conflict, left alone, escalates and calcifies: a snide remark becomes a grudge, a grudge becomes a faction, a faction becomes a resignation. The lead's rule: *intervene at the first sign of relationship conflict* — a private conversation with each party (9.3/9.4 skills), then a joint one if needed. Early intervention costs 30 minutes; late intervention costs a team. The counter-instinct to suppress ("let's not make a fuss") is how teams learn that conflict is dangerous — exactly the lesson that produces the Lencioni dysfunctions.
- **Escalation.** Escalate deliberately, not emotionally: when two team members can't resolve between themselves (with your coaching), when a conflict crosses team boundaries (two leads need the shared manager), or when it involves behaviour HR must know about. Escalation is not failure — it is the correct use of the organisation's structure, and the lead who escalates well (with a written summary and a recommendation) models the RAPID discipline (Section 7.3).
- **No-surprises culture.** The through-line of this entire section: *no one should ever be surprised by feedback, a PIP, a restructure, or a decision that affects them*. Surprises destroy trust; trust is the foundation of everything else (Section 2.5). The no-surprises disciplines: feedback in real time (Section 3.8), goals reviewed monthly (Section 5.6), decisions communicated with their rationale (Section 7.6), and bad news delivered *early, in person (or on video), with the reasoning and the next steps* — never by email, never in a group setting, never after it's already leaked.

---

## 10. The Team Lead in the Enterprise

### 10.1 Delivery in the bank/enterprise

In a bank, delivery is *governed* delivery: release trains with fixed windows, change advisory boards, incident and problem management, SDLC gates, and audit scrutiny. The team lead's delivery job is to run the project-side machinery competently while defending the team from its weight. Cross-reference: the full project-management frameworks (waterfall/agile/hybrid, governance, RAID, etc.) live in [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md); the org-level view lives in [strategic_management_guide.md](strategic_management_guide.md). The team-lead-specific disciplines:

- **Commitments are contracts.** In a bank, a committed date reaches regulators and clients; the lead must be ruthless about what the team commits to and honest early about slippage (no-surprises, Section 9.6, applied to *upward* communication).
- **Releases are events.** The lead owns release readiness: what's in the release, who signs off, what the rollback is, who's on call. Release discipline is where compliance and engineering meet, and where the lead's credibility with both sides is earned.
- **Quality is non-negotiable.** In a bank, a production defect isn't a bug — it's a potential regulatory incident. The lead holds the line on testing, controls, and evidence even under schedule pressure (this is the *competing* mode of the TKI, Section 9.2, done correctly: "no, we will not skip the controls sign-off").

### 10.2 The matrix organization

Most bank IT teams live in a **matrix**: a *functional* line (the lead's reporting chain — competence, people, career) crossed with *project/programme* streams (dotted-line authority from programme managers, product owners, or the COO office). Consequences the lead must manage:

- **Two bosses, one person.** Team members answer to the functional lead *and* the stream's programme manager; priorities can genuinely conflict. The lead's job is to negotiate those conflicts at *their* level (with the programme manager, or by forcing the decision up to the shared boss) rather than passing the conflict down to the team — see [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) for the full playbook.
- **Influence without authority.** In the matrix, much of the lead's work is lateral: getting platform teams, security, and ops to cooperate without formal authority over them. The toolkit is stakeholder management (10.3), relationship investment, and making *your* requests easy to say yes to (clear asks, early notice, shared credit).
- **Clarity is the antidote.** The matrix's pain comes from ambiguity (who decides what — Section 7.7's authority matrix again). The lead reduces ambiguity by publishing decision rights, escalating priority conflicts fast, and never letting the team absorb two contradictory directives silently.

### 10.3 Stakeholder management

The team lead's stakeholder set: the line manager / head of department, the programme or product stakeholders, the front-office/business users (in a bank, often traders, relationship managers, or ops), risk/compliance/audit (10.4), peer leads, and the team itself (the most important stakeholders of all). The full stakeholder-management discipline — mapping, power/interest analysis, communication planning, expectation management — is in [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md). The team-lead summary:

- **Map them.** Who cares about the team's work, who can hurt/help it, who needs what frequency of what information? (Power-interest grid: manage closely / keep satisfied / keep informed / monitor.)
- **Over-communicate deliberately.** The lead's default error is under-communicating upward. A weekly one-page status (what shipped, what's next, what's at risk, what the team needs) is the cheapest stakeholder insurance there is — it turns "surprise" into "context."
- **Manage expectations, not just delivery.** In a bank, stakeholder expectations (scope, dates, what "done" means) are negotiated artefacts, and the lead is the negotiator. Under-promise on the uncertain, over-deliver on the certain, and say "we'll know by Thursday" instead of "no problem."

### 10.4 Compliance, risk, audit

In a bank, risk and compliance are not overhead — they are *the business*; the lead's regulatory literacy is part of their leadership. Practical team-lead moves:

- **Treat compliance as a stakeholder** (10.3), not an obstacle: invite risk/compliance to planning early, give them the context they need to say yes, and their sign-offs get faster.
- **Build evidence into the workflow.** Audit readiness is a by-product of good hygiene: decision logs (Section 7.4), retro actions tracked, change records complete, documentation current (Section 8.7). If the team's normal process produces audit-ready evidence, an audit is a presentation, not a firefight.
- **Know the red lines.** Some things are non-negotiable regardless of delivery pressure: segregation of duties, change approval, data protection, and — in the AI era — model-governance and data-usage rules (see [../technology/people_management_in_ai_age_guide.md](../technology/people_management_in_ai_age_guide.md) for the emerging governance questions). The lead who defends these *with the team on their side* (explaining the why) converts compliance from a constraint into a source of trust.
- **Incidents are leadership moments.** When something breaks in production, the team looks at the lead: blame-and-hunt (which the bank's incident process may push toward) or blameless learning (which builds the psychological safety of Section 2.5). The lead sets the tone of the post-mortem, and that tone determines whether the team reports near-misses or hides them.

### 10.5 The agile team lead: Scrum Master vs team lead vs agile coach

The agile framing (Scrum/Kanban) is the delivery operating system of most bank IT teams — mechanics in [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md). The role questions:

- **Scrum Master vs team lead.** The Scrum Master is a *process* role defined by the Scrum Guide: facilitator of ceremonies, guardian of the process, remover of impediments, coach of self-management. The team lead is an *organisational* role: accountable for delivery and people. Where separate, they partner (the SM protects process and the lead owns outcomes); where combined (common in small squads), the lead must not let the *facilitation* half crowd out the *people* half — running a great retro is not the same as coaching a struggling engineer.
- **The agile coach.** The agile coach is a *change* role: a specialist (internal or external) who helps teams and organisations adopt and deepen agile practice. The team lead is not an agile coach, but uses coaching *skills* (Section 3) and treats the coach as an ally — and in a bank's transformation programmes, the lead is often the person who makes the coach's work stick (or die) inside the team.
- **Agile leadership.** Agile is a leadership philosophy as much as a process: self-managing teams, inspect-and-adapt, servant-leadership (Section 1.4), empirical process control. The lead who runs agile ceremonies without agile *leadership* gets the ritual without the benefit.

### 10.6 The engineering manager (EM)

**Engineering manager** is the modern industry title for the people-and-delivery management role — effectively *team lead + line manager + delivery owner* at the engineering-team level (the term is standard industry usage; its spread tracks the tech-industry management track of the 2010s, popularised by orgs like Google/Facebook and by management literature such as Camille Fournier's *The Manager's Path* — see [the_managers_path_research.md](the_managers_path_research.md)). The EM role combines what this guide splits into team lead (delivery + people day-to-day) and line manager (hiring, comp, performance process). In a bank, the EM title is less common (banks favour "team lead" / "IT manager" / "delivery manager"), but the *function* is universal. If you hold the combined role, the practical consequences are: your 1:1 load doubles (people + delivery conversations), your calendar is the scarce resource (Section 8.6/8.7 hygiene becomes survival), and your feedback and performance-process duties are non-delegable.

### 10.7 The staff+ / tech lead track

For the lead whose strength runs *technical* rather than people-managerial, the growth path is the **staff+ track** — staff engineer, principal, architect: deep technical direction, org-wide influence, mentoring of tech leads. The team lead sits at the fork: people-management track (lead → manager → head) or technical track (tech lead → staff → principal/architect). See the architect-career and staff+ track guides in the technology library (e.g. the architecture-career material referenced there) for the technical track in depth; for this guide, the relevant points are: (1) the tracks are *parallel and equal*, not a ladder where management is the prize; (2) a team lead can deliberately hand technical direction to a staff engineer and focus on people/delivery — that is a healthy split, not a demotion; and (3) the team lead's job includes *sponsoring* the staff+ candidate on their team — visibility, stretch assignments, and advocacy upward (see [grow_team_guide.md](grow_team_guide.md)).

---

## 11. Framework Selection

### 11.1 Which framework when

The one-paragraph map — the answer to "which tool for this moment?":

| Situation | Framework(s) | Where |
|---|---|---|
| A person is stuck on a problem they own | GROW (coaching) | Section 3.1 |
| Someone needs to hear something hard | SBI (feedback) — then Radical Candor as the stance | Sections 3.5–3.6 |
| The team feels off and I can't say why | Tuckman (stage) → Lencioni (dysfunction) → health check (data) | Sections 2.1–2.6 |
| People are disengaged | SDT diagnosis (autonomy/competence/relatedness) + Herzberg (hygiene vs motivators) | Section 4 |
| We need direction for the quarter | OKR (team) + SMART (individual goals from it) | Section 5 |
| I'm doing everyone's work | Delegation ladder + the delegation conversation | Section 6 |
| A decision keeps not getting made | DACI/RAPID (roles) + proposal-driven decision | Section 7 |
| Two people are at each other | TKI (modes) + crucial/difficult-conversations skills | Section 9 |
| The team is in ceremony-without-meaning | Retro discipline + meeting design (outcomes, async-first) | Sections 8.5–8.7 |
| I don't know my people | 1:1 (weekly, employee's agenda) | Section 8.1 |

### 11.2 The minimal framework stack

If you adopt *only* seven frameworks, the set that covers the most of the job (each is cheap to learn and compound in combination):

1. **The weekly 1:1** — the container for everything else (Section 8.1).
2. **GROW** — the default conversation shape for coaching and problem-solving (Section 3.1).
3. **SBI** — the default shape for all feedback, positive and corrective (Section 3.5).
4. **OKR** — the team's direction and the reason the work matters (Section 5.1).
5. **DACI** — clarity on who decides what, killing decision-avoidance (Section 7.1).
6. **Tuckman** — the team-stage lens that tells you which interventions to run (Section 2.1).
7. **TKI** — the conflict vocabulary that de-escalates and guides your own mode choice (Section 9.2).

Everything else in this guide is a refinement of these seven. Master the stack, and the rest (RAPID vs DACI, COIN vs SBI, skip-levels, PIPs) are variations you can pick up as needed.

### 11.3 Methodology vs framework

The distinction that keeps the toolkit coherent: **a methodology is a complete process system** — Scrum, Kanban, SAFe, PRINCE2 — that prescribes roles, ceremonies, and workflow end-to-end (see [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md)). **A framework (as used in this guide) is a single model for one kind of situation** — GROW is a conversation shape, not a way of running the whole team. Consequences:

- Methodologies *replace* each other (you run Scrum *or* Kanban *or* a hybrid); frameworks *complement* each other (GROW and SBI and DACI all live in the same week).
- Methodologies are adopted by the organisation; frameworks are adopted by *you* — a team lead can install SBI tomorrow without changing the SDLC.
- The failure mode this distinction prevents: treating frameworks as if they were methodologies (running a whole team "by GROW" is nonsense) and treating methodologies as if they were frameworks (cherry-picking Scrum ceremonies without the process system is how "agile" becomes chaos).

### 11.4 Pragmatic team leading: use what works

Anti-dogma rules for the toolkit:

- **The framework serves the person, not the reverse.** If SBI feels bureaucratic in a given moment, give the feedback *well* (specific, timely, kind) — the framework is training wheels, not a religion. If GROW stalls (the person genuinely has no idea), tell them what you'd do — coaching is the default, not the only gear (Section 3.4).
- **Context beats orthodoxy.** A bank's compliance reality overrides textbook agility; a crisis overrides the meeting design rules; a person's culture and personality override the standard delivery of any model. The mark of mastery is knowing *when* to bend the model.
- **Names are optional.** The team doesn't need to know you're "doing SBI" or "running a TKI." They need feedback that lands, decisions that get made, and a lead who is consistent. Use the vocabulary with *yourself* and with peer leads; use the *behaviour* with the team.
- **Measure the outcome.** Every framework is a hypothesis: if the team's performance, health, and retention improve, the toolkit works; if not, adjust. The frameworks are the how — the *outcomes* (delivery, people, process, stakeholders — Section 1.3) are the why.

### 11.5 Team lead maturity: the development path

The team lead role is itself a development arc, and the guides exist for each rung:

- **First 90 days** — the transition itself: [../technology/first_time_manager_guide.md](../technology/first_time_manager_guide.md) (IC→leader identity shift) and this guide's Section 12 (the first quarter, worked).
- **First year** — build the habits: 1:1 rhythm, feedback fluency, delegation, the minimal stack (11.2). The manager-as-coach stance deepens via [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) (styles) and this guide's coaching section.
- **Years 1–3** — broaden: effectiveness in all directions ([3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md)), managing up/down/sideways ([managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md)), stakeholder craft ([communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)), and team building ([grow_team_guide.md](grow_team_guide.md)).
- **Years 3+** — strategic: [strategic_management_guide.md](strategic_management_guide.md) (org-level), [360_management_guide.md](360_management_guide.md) (holistic leadership), and the choice between the people-management track and the staff+/technical track (Section 10.7).

Maturity is also visible in the *behavioural* shifts: from doing → delegating → developing → designing (the classic manager progression); from telling → asking → facilitating → sponsoring; from solving the team's problems → teaching the team to solve them → creating conditions where they rarely arise.

---

## 12. Worked Example — A New Team Lead's First Quarter

### 12.1 The setup

Mei, a senior engineer, has just been made team lead of a 6-person payments-integration squad at a bank: three engineers (one senior, two mid), one QA, one business analyst, plus a part-time Scrum Master shared with another squad. Her line manager wants two things this quarter: the team's committed deliveries (the September release train) and "a team that can run itself by year end." The team is polite, quiet in retros, and heavily dependent on the previous lead's decisions. This is a classic *forming* team with a storming risk — and Mei's quarter is the walkthrough of everything in this guide.

### 12.2 Week 1: 1:1s, charter, and the forming stage

**Day 1–2 — meet the team as *their* lead.** Mei holds a 30-minute 1:1 with each person (Section 8.1) before touching any delivery detail. She uses the openers: "What's on your mind?" (Section 8.3) — and learns: one engineer is quietly furious about being skipped for a training opportunity; the BA feels excluded from technical decisions; the senior engineer is relieved someone is finally leading. Two things Mei already knows: her 1:1s are working, and the team's real map is not the org chart.

**Day 3 — the team charter (Section 2.4).** A 90-minute charter session: purpose ("we integrate payments partners into the hub — every integration a client touches"), roles, and *working agreements* the team proposes: disagreements get voiced in the room; retro is blameless; no surprise releases. Mei adds one norm herself: "when we disagree, we decide by the DACI — one approver, decided fast" (Section 7). The team is still in **forming** (polite, watching her) — Tuckman diagnosis (Section 2.1) says: give direction, structure, and an early win.

**Day 4–5 — the early win and the first delegation.** Mei deliberately picks a small, visible deliverable (a monitoring dashboard the team had been asking for) and *delegates* it end-to-end to the junior engineer with the delegation conversation: what/why/level 4/checkpoints (Section 6.5). The dashboard ships in two weeks; the junior gets credit in the team channel (specific praise, Section 3.8); the team starts believing Mei delivers.

### 12.3 Month 1: goals, delegation, and the first hard feedback

**Week 2–3 — OKR alignment (Section 5).** Mei's line manager hands down the division objectives; Mei translates them into a *team proposal* before the planning session: Objective "make partner onboarding boring" with KRs (integration time from 6 weeks to 3, zero data-loss incidents, onboarding runbook published). The team pushes back on the 3-week KR — Mei negotiates to 4 with a stretch note, then *commits the team only to what they committed to* (Section 5.5). Individual goals follow in the 1:1s (SMART, Section 5.4).

**The delegation audit.** Mei lists her own recurring tasks and runs the ladder (Section 6.2): environment changes → L3 to the senior engineer; test-automation backlog → L4 to the QA; the weekly stakeholder status → *she* keeps (it's her stakeholder relationship, Section 10.3); release sign-offs → L2 to the senior, building toward L3. She notices the pattern from Section 6.6 starting: engineers bring her questions they could answer — she redirects with "what do you think?" and rewards the first initiative with public credit.

**Week 4 — the first SBI feedback (Section 3.5).** The senior engineer's code review comments have turned sarcastic ("did anyone test this?") and the mid-level engineer has gone quiet in reviews. Mei runs SBI in private: *Situation* — "in Tuesday's review of the reconciliation fix…"; *Behavior* — "your comment read as a personal jab rather than a technical point — you wrote 'did anyone test this' instead of naming the missing test"; *Impact* — "the reviewer went quiet and I'm worried the review culture is becoming about avoiding you." The senior is defensive at first; Mei holds the crucial-conversations skills (Section 9.3): safety first, facts before story, tentative delivery. They agree a norm: review comments reference the code, not the coder. No surprises: the *first* time he hears this is not in the annual review.

### 12.4 The quarter: health, conflict, retros, coaching

**Month 2 — storming arrives (Tuckman, Section 2.2).** The integration work gets hard and the team starts fighting: the BA and the senior engineer clash over a partner's requirements interpretation; two engineers disagree on the retry-strategy design. Mei *does not suppress it* — she names it in the retro ("we're disagreeing more — that's the team growing up, let's disagree well") and runs the TKI conversation (Section 9.2): the BA-senior clash is resolved by *collaborating* (both sides' concerns turned into a requirements-decision process with one approver); the design dispute gets a DACI (the senior is the D, the other engineer the driver of the proposal, everyone else contributor/informed — Section 7.1) and a disagree-and-commit pact (Section 7.6). The team emerges with its first real *norms* — and Mei notices the storming was the price of the trust forming underneath.

**Month 2 — the health check (Section 2.6).** Mei runs the Spotify-style health check (anonymous): delivery "good," fun "mixed," learning "unhappy." The discussion surfaces: no one has time to learn because the incident queue eats the week. One improvement action for the quarter: the QA owns an incident-triage rota, the team commits to root-cause fixes for the top three recurring incidents — tracked in the retro (Section 8.5). The health check gives Mei her quarter's people-work item.

**Month 3 — GROW coaching (Section 3.1).** The junior engineer wants to grow toward mid-level but is stuck. Mei runs a GROW session: *Goal* — "own a partner integration end-to-end by year end"; *Reality* — she's only ever done components, has never run the client-facing part; *Options* — shadow the senior on the next integration, take the integration-runbook task, ask to lead the next onboarding kickoff; *Will* — she commits to leading the kickoff in three weeks, Mei agrees to check in. This is also the *autonomy* lever of SDT (Section 4.1) in action — the goal is the junior's, not Mei's.

### 12.5 The quarter review: outcomes and learnings

**The outcomes:**

- **Delivery:** all committed September-release items shipped (the OKR KR "zero data-loss incidents" held — the team's controls discipline from Section 10.1 did its job). One KR (integration time 4 weeks) landed at 4.5 — scored honestly in the review, feeding next quarter's plan (Section 5.3).
- **People:** zero attrition; the junior led her first kickoff (GROW payoff); the review culture is measurable calmer; the incident queue is down a third (health-check action).
- **Team:** the team runs planning with Mei mostly listening; retros produce actions that get done; the team now argues *in the room* and commits afterwards — the Lencioni pyramid (Section 2.3) has visibly unblocked: trust (blameless retro holds), conflict (debates happen), commitment (decisions stick), accountability (peers call out missed commitments), results (the quarter's scoreboard is shared).
- **Mei's own growth:** 1:1s are her anchor; she delegates at named levels; her first PIP-adjacent conversation (a QA performance wobble) was handled with SBI + coaching *before* it needed formal steps (Section 9.5); her upward communication (weekly one-pager, Section 10.3) has her line manager asking *her* for input on the next quarter's scope.

**The learnings (Mei's own retrospect):** (1) the 1:1s came first and everything else built on them; (2) naming the Tuckman stage out loud ("we're storming — good") was the cheapest de-escalation tool she used; (3) her instinct to fix the design dispute herself would have been the wrong move — the DACI plus disagree-and-commit was the actual fix; (4) she underestimated how much *stakeholder* work the role is — the weekly one-pager became non-negotiable.

### 12.6 The quarter at a glance — timeline

| When | What | Frameworks in play |
|---|---|---|
| Week 1 | 1:1s with everyone; team charter; early-win dashboard delegated | 1:1 (8.1), charter (2.4), Tuckman forming (2.1), delegation L4 (6.2) |
| Weeks 2–3 | Team OKRs negotiated; individual SMART goals; delegation audit | OKR (5.1), SMART (5.4), delegation ladder (6.2) |
| Week 4 | First corrective feedback to senior engineer | SBI (3.5), crucial conversations (9.3) |
| Month 2 | Storming surfaces; BA/senior conflict; design dispute | Tuckman storming (2.2), TKI (9.2), DACI (7.1), disagree-and-commit (7.6) |
| Month 2 | Team health check; incident improvement action | Spotify health check (2.6), retro discipline (8.5) |
| Month 3 | Junior's growth coaching; kickoff ownership | GROW (3.1), SDT autonomy (4.1) |
| End of quarter | Quarterly review; honest OKR scoring; next-quarter plan | OKR review (5.3), no-surprises (9.6), 1:1 (8.1) |

---

## 13. Glossary

**Team lead** — the first-line leader: accountable for a small team's delivery and people; the player-coach of the org chart (Section 1.1).

**Tech lead** — the engineer accountable for technical direction: architecture, design, standards, code quality (Section 1.5).

**Line manager** — the people/org role: hiring, performance, comp, career, exits; often combined with team lead (Section 1.2).

**Scrum Master** — the agile process role defined by the Scrum Guide: ceremony facilitator, process guardian, impediment remover (Section 10.5).

**Player-coach** — the team-lead archetype: does hands-on work *and* leads; must resist doing the work instead of developing people (Section 1.4).

**Servant-leader** — Greenleaf's 1970 leadership philosophy: serve the team's needs first, so people grow and become autonomous (Section 1.4).

**Tuckman** — Bruce Tuckman's 1965 model of group development: forming–storming–norming–performing, with adjourning added in 1977 with Mary Ann Jensen (Section 2.1).

**Forming** — first stage: polite, dependent, direction-seeking; the leader sets structure (Section 2.1).

**Storming** — second stage: conflict over roles and approach; necessary and normal; the leader contains and channels it (Section 2.1).

**Norming** — third stage: agreements, cohesion, shared ways of working (Section 2.1).

**Performing** — fourth stage: high autonomy, self-management; the leader protects the boundary (Section 2.1).

**Adjourning** — fifth stage (1977): disbanding; the leader plans closure and recognition (Section 2.1).

**Lencioni** — Patrick Lencioni, *The Five Dysfunctions of a Team* (2002): the trust→conflict→commitment→accountability→results pyramid (Section 2.3).

**Five dysfunctions** — absence of trust, fear of conflict, lack of commitment, avoidance of accountability, inattention to results (Section 2.3).

**Psychological safety** — Edmondson's 1999 concept: shared belief that the team is safe for interpersonal risk-taking; Google's Project Aristotle's top predictor of team effectiveness (Section 2.5).

**Edmondson** — Amy Edmondson, Harvard professor, author of the 1999 *Administrative Science Quarterly* paper that founded team psychological safety research (Section 2.5).

**Team charter** — the team's constitution: purpose, roles, norms, working agreements, process anchors; created by the team with the lead (Section 2.4).

**Working agreement** — a norm in the team contract ("disagreements get voiced in the room"); only meaningful if enforced (Section 2.4).

**GROW** — the coaching conversation model (Goal, Reality, Options, Will), developed in the 1980s by Whitmore, Alexander, and Fine; popularised in *Coaching for Performance* (1992) (Section 3.1).

**Coaching** — drawing solutions out of the person through questioning (Section 3.3).

**Mentoring** — sharing experience and advice; the mentor guides, the mentee absorbs (Section 3.3).

**SBI** — Situation–Behavior–Impact feedback model, developed by the Center for Creative Leadership (Section 3.5).

**Radical Candor** — Kim Scott's 2017 feedback framework: care personally × challenge directly; the four quadrants (Section 3.6).

**Ruinous empathy** — caring without challenging: saying nothing to avoid hurting, at the person's expense (Section 3.6).

**COIN** — Context–Observation–Impact–Next feedback variant; no single verifiable origin; training-house model (Section 3.7).

**Feedback sandwich** — praise→criticism→praise; a widely criticised myth; direct, specific, timely feedback replaces it (Section 3.8).

**SDT** — Self-Determination Theory (Deci & Ryan, 1985): intrinsic motivation depends on the needs for autonomy, competence, and relatedness (Section 4.1).

**Autonomy** — the need to act from genuine choice; the first SDT need (Section 4.1).

**Competence** — the need to feel effective and growing; the second SDT need (Section 4.1).

**Relatedness** — the need to belong and connect; the third SDT need (Section 4.1).

**Herzberg** — Frederick Herzberg's 1959 two-factor theory: hygiene factors prevent dissatisfaction; motivators create satisfaction (Section 4.3).

**Hygiene** — the dissatisfaction-preventing factors: pay, conditions, policy; they cannot motivate (Section 4.3).

**Motivator** — the satisfaction-creating factors: recognition, achievement, growth, responsibility, the work itself (Section 4.3).

**Intrinsic** — motivation from the work itself (interest, meaning); the higher-quality motivation (Section 4.1).

**Extrinsic** — motivation from external controls (rewards, pressure); can crowd out intrinsic motivation (Section 4.1).

**OKR** — Objectives and Key Results; Andy Grove at Intel (1970s, *High Output Management* 1983), popularised by John Doerr (*Measure What Matters*, 2018): qualitative objective + 3–5 quantitative KRs, quarterly (Section 5.1).

**KPI** — Key Performance Indicator: the health monitor of the present; complements OKRs' change-direction (Section 5.2).

**SMART** — Specific, Measurable, Achievable, Relevant, Time-bound; George T. Doran's 1981 goal-quality filter (Section 5.4).

**Delegation** — handing over responsibility with authority, context, and checkpoints; the mechanism of team growth (Section 6.1).

**Delegation ladder** — graduated authority levels (do-as-told → research-and-recommend → recommend-and-act → act-and-report → full autonomy); a common synthesis, no single canonical source (Section 6.2).

**Micromanagement** — retaining control of every step after "delegating"; the top demotivator (Sections 4.5, 6.4).

**Reverse delegation** — tasks flowing back up to the lead; broken by redirecting with the ladder and rewarding initiative (Section 6.6).

**DACI** — Driver–Approver–Contributor–Informed; decision-role model developed at Intuit in the 1980s (Section 7.1).

**RACI** — Responsible–Accountable–Consulted–Informed; the task-ownership matrix; origin not verifiable to a single source (Section 7.2).

**RAPID** — Recommend–Agree–Perform–Input–Decide; Bain's 2006 decision model from "Who Has the D?" (Section 7.3).

**Vroom-Yetton** — the 1973 normative model (Vroom & Yetton) for choosing a decision mode (autocratic/consultative/consensus/democratic) from situation questions (Section 7.5).

**Disagree and commit** — Amazon's leadership principle: dissent hard in the room, commit fully afterwards (Section 7.6).

**1:1** — the weekly one-on-one: the leader's most important meeting; employee's agenda; 30 minutes weekly is the Manager Tools (Horstman & Auzenne) prescription; Grove's "what's on your mind?" is the canonical opener (Sections 8.1–8.3).

**Skip-level** — a periodic 1:1 between a manager's manager and the people two levels down; the anti-silo, anti-filter instrument (Section 8.4).

**Standup** — the daily team sync; for the team, not the lead's status report (Section 8.5).

**Retro** — the retrospective: blameless improvement ceremony that must produce followed-up actions (Section 8.5).

**Thomas-Kilmann / TKI** — the 1974 conflict-modes instrument (Thomas & Kilmann): competing, collaborating, compromising, avoiding, accommodating (Section 9.2).

**Crucial conversations** — Patterson, Grenny, McMillan & Switzler (2002): the toolkit for high-stakes conversations — safety, stories, STATE, listening (Section 9.3).

**Difficult conversations** — Stone, Patton & Heen (1999, Harvard Negotiation Project): every hard conversation is really three — what-happened, feelings, identity (Section 9.4).

**PIP** — Performance Improvement Plan: the formal, documented, time-boxed improvement step; honest tool or trust-destroying weapon depending on use (Section 9.5).

**Matrix organization** — the structure crossing functional lines with project/programme streams; dotted lines, two bosses, influence-without-authority (Section 10.2).

**Engineering manager** — the modern title for team lead + line manager + delivery owner; see Fournier's *The Manager's Path* (Section 10.6).

**Agile coach** — the change specialist who helps teams and orgs deepen agile practice; distinct from team lead and Scrum Master (Section 10.5).

**Quiet quitting** — the 2022 term for disengagement: doing the job, quitting the discretionary effort; a symptom to diagnose, not a behaviour to shame (Section 4.6).

---

*End of guide. Companion guides in the Management & Leadership Series: [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) · [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) · [360_management_guide.md](360_management_guide.md) · [the_managers_path_research.md](the_managers_path_research.md) · [strategic_management_guide.md](strategic_management_guide.md) · [grow_team_guide.md](grow_team_guide.md) · [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) · [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md). Technology-library cross-references: [../technology/first_time_manager_guide.md](../technology/first_time_manager_guide.md) · [../technology/project_management_methodologies_guide.md](../technology/project_management_methodologies_guide.md) · [../technology/product_management_guide.md](../technology/product_management_guide.md) · [../technology/people_management_in_ai_age_guide.md](../technology/people_management_in_ai_age_guide.md).*
