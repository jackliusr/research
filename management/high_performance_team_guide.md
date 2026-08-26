# High-Performance Teams: The Research and Discipline — A Comprehensive Guide

> **Author's Note:** This guide is the dedicated deep-dive on the **high-performance team (HPT) discipline** — not another list of team-building exercises, but the *research canon and engineering discipline* behind high performance: what the canon actually says (Katzenbach & Smith 1993), what the evidence actually shows (Google Project Aristotle 2015), why psychological safety is the load-bearing wall (Edmondson 1999, *The Fearless Organization* 2018), what conditions make effectiveness possible (Hackman 2002), how the classic dynamics models fit (Tuckman, Lencioni, Belbin), how to recognise and build high-performance *engineering* teams (DORA, SPACE), and how to design a team on purpose — charter, goals, feedback loops, accountability. It is written for team leads, squad leads, and architects in enterprise technology settings (with the Cymbal Bank context throughout).
> Where a model is already covered by a sibling guide, this guide **cross-references instead of re-deriving**: the *framework toolkit* (Tuckman, Lencioni, psychological-safety levers, charters, health checks) lives in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §2, the *hiring-to-scaling lifecycle and squad topology* live in [grow_team_guide.md](grow_team_guide.md) (§7 on the Spotify model, squads, tribes), and the *organisational behaviour* discipline context lives in [organizational_behavior_guide.md](organizational_behavior_guide.md).
> *Part of the Management & Leadership Series | Author: Jack Liu Shurui — Solution Architect, Cymbal Bank*
> *Related guides in this series: [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) · [grow_team_guide.md](grow_team_guide.md) · [organizational_behavior_guide.md](organizational_behavior_guide.md) · [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) · [the_managers_path_research.md](the_managers_path_research.md) · [strategic_management_guide.md](strategic_management_guide.md) · [it_strategy_guide.md](it_strategy_guide.md) · [360_management_guide.md](360_management_guide.md)*

---

## How to Use This Guide

This guide is a **discipline reference**: it tells you what high-performance teams are, what the research actually established, and how to build one deliberately. **For a quick orientation**, read Section 1 (the overview) and Section 10 (the one-page summary). **For the evidence**, Sections 2–3 are the canon and the data. **For the single most important lever**, Section 4 (psychological safety). **For the design conditions**, Section 5 (Hackman) and Section 8 (team design). **For engineering-specific teams**, Section 7. **To see the whole discipline assembled**, Section 9 walks a Cymbal Bank squad through a full transformation. Each section is self-contained; the glossary at the end defines every term in one place.

Three reading notes. First, this guide's claim to distinctiveness is **verification**: every canonical fact (authors, years, findings, definitions) has been checked against primary or reputable secondary sources, and anything that could not be verified is **flagged honestly** rather than glossed over (see the Appendix for the verification ledger). Second, the guide draws a deliberate line between the *frameworks* (models you apply in the moment — covered in team_lead_methodologies_guide.md §2) and the *discipline* (the research canon and the design practice that this guide deepens). Third, a recurring distinction: a **group** is a collection of individuals; a **team** is an interdependent unit with a shared outcome; a **high-performance team** is a team that delivers outsized results *and* stays healthy — the distinction matters because most "team problems" are actually group-design problems.

---

## Table of Contents

1. [The HPT Overview](#1-the-hpt-overview)
2. [The Research Canon](#2-the-research-canon)
3. [The Evidence Base](#3-the-evidence-base)
4. [The Psychological Safety](#4-the-psychological-safety)
5. [The Effectiveness Conditions](#5-the-effectiveness-conditions)
6. [The Team Dynamics](#6-the-team-dynamics)
7. [The Engineering Teams](#7-the-engineering-teams)
8. [The Team Design](#8-the-team-design)
9. [The Worked Example](#9-the-worked-example--a-cymbal-bank-squad-transformation)
10. [The Summary](#10-the-summary--one-page)
11. [Glossary](#11-glossary)
12. [Appendix: Verification Notes](#appendix-verification-notes)

---

## 1. The HPT Overview

### 1.1 What a high-performance team is

**The canonical definition (verified — Katzenbach & Smith, *The Wisdom of Teams*, 1993):** a team is *"a small number of people with complementary skills who are committed to a common purpose, set of performance goals, and approach for which they hold themselves mutually accountable."* A **high-performance team** is a team that operates at the top of that definition — it has all five elements working at once, *plus* a quality of interaction that makes the results more than the sum of the parts.

The definition has exactly **five load-bearing components**, and each one is a design decision, not a hope:

| Component | What it means in practice |
|---|---|
| **Small number** | Small enough to meet frequently, interact openly, and know each other's work — typically 4–9 people; Katzenbach & Smith advise splitting if a team grows beyond ~10 (they studied teams up to ~50 and found intimacy is what breaks) |
| **Complementary skills** | The mix covers three skill families: technical/functional expertise, problem-solving ability, and interpersonal skills — gaps must be *learnable*, not just hoped away |
| **Common purpose** | A meaningful, agreed reason for existing that the team itself has shaped — not a mandate handed down; high-performing teams spend serious time forging it |
| **Performance goals** | Specific, measurable, challenging goals derived from the purpose — the goals, not the purpose alone, are what convert intent into effort |
| **Mutual accountability** | Members hold *each other* (and themselves) accountable — accountability to the team, not just to the boss |

**What HPT is not:** it is not "everyone gets along" (a pleasant group is not a team); it is not a team that merely hits its numbers once (sustainability matters); and it is not a team that feels great but delivers nothing (high *performance* is the output condition). The HPT discipline is the deliberate engineering of all five components plus the interaction quality that makes them compound.

### 1.2 The HPT vs the working group vs the ordinary team

Katzenbach & Smith's **team performance curve** (verified) distinguishes five states of collective work:

- **Working group** — the default organisational unit: people share information and best practice, but each member is individually accountable for their own results. No shared work products, no mutual accountability. Fine for many purposes; not a team.
- **Pseudo-team** — a group that *calls itself* a team but where members are not actually committed to a common purpose or mutual accountability; individual performance is what's really rewarded. The worst state: it looks like teamwork and delivers worse than a working group.
- **Potential team** — a group striving for the collective goal; real commitment is emerging but performance goals and a common approach are not yet nailed down. Under-delivers relative to what it could be.
- **Real team** — a small group with complementary skills, a common purpose, performance goals, and a common approach, holding itself mutually accountable. The baseline definition above.
- **High-performance team** — a real team whose members are also *deeply committed to one another's personal growth and success*. The mutual accountability becomes a genuine felt obligation; performance outstrips any reasonable expectation.

The insight that matters for a lead: **most teams are not broken — they are simply not teams yet.** Moving a group along the curve is a *design* problem (Section 8), not a motivational one.

### 1.3 The overview table

The table below is the map of this guide: every aspect of the HPT discipline, where the evidence comes from, and where in this guide (and the sibling series) each aspect is developed.

| Aspect | Description | Evidence base | Where it lives |
|---|---|---|---|
| **Definition & discipline** | Small number, complementary skills, common purpose, performance goals, mutual accountability | Katzenbach & Smith 1993 | §2 of this guide |
| **Team dynamics frameworks** | Tuckman stages, Lencioni dysfunctions, psychological-safety levers, charter mechanics, health checks | Tuckman 1965; Lencioni 2002; Edmondson | [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §2 (cross-ref) |
| **Evidence base** | What correlates with effectiveness, ranked | Google Project Aristotle 2015 | §3 of this guide |
| **Psychological safety** | Shared belief the team is safe for interpersonal risk-taking; the #1 predictor | Edmondson 1999; *The Fearless Organization* 2018 | §4 of this guide + team_lead §2.5 |
| **Effectiveness conditions** | The designable conditions that set the stage for great performances | Hackman 2002 | §5 of this guide |
| **Team lifecycle & topology** | Hiring → scaling → squad/tribe/chapter/guild structures | Spotify model; industry practice | [grow_team_guide.md](grow_team_guide.md) (cross-ref) |
| **Engineering performance** | Delivery metrics and productivity dimensions for software teams | DORA (Accelerate State of DevOps); SPACE 2021 | §7 of this guide + ../technology/ dev-tooling cluster |
| **Team design practice** | Charter, goals, feedback loops, accountability | Synthesised from the canon | §8 of this guide |
| **Transformation in practice** | A full worked example in a bank-IT setting | This guide's synthesis | §9 of this guide |

### 1.4 Why the HPT discipline matters in the bank/IT context

In a large regulated enterprise — a Cymbal Bank-style organisation — the default organisational unit is the *working group inside a matrix*: individually accountable, hierarchy-shaped, process-heavy. The HPT discipline matters there precisely because the environment fights it: matrix reporting splits accountability (see [it_strategy_guide.md](it_strategy_guide.md) on org design), compliance and audit push documentation over collective ownership (see [organizational_behavior_guide.md](organizational_behavior_guide.md) and the banking-series cross-refs in §9), and permanent teams are rare — people are staffed to projects. Every element of the discipline (stable membership, a real charter, team-level goals, blameless retros, mutual accountability) is a *counter-structural* choice the lead must make explicitly, because the organisation will not make it for you.

### 1.5 Section summary

- A high-performance team is precisely defined: **small, complementary skills, common purpose, performance goals, mutual accountability** (Katzenbach & Smith 1993, verified).
- Most underperforming "teams" are actually groups or pseudo-teams — the fix is design, not motivation.
- The rest of this guide develops each aspect: the canon (§2), the evidence (§3), the safety (§4), the conditions (§5), the dynamics (§6), the engineering angle (§7), the design practice (§8), and a full worked example (§9).

---

## 2. The Research Canon

### 2.1 Katzenbach & Smith, *The Wisdom of Teams* (1993) — verified

**Origin (verified):** Jon R. Katzenbach and Douglas K. Smith, both McKinsey partners at the time, published *The Wisdom of Teams: Creating the High-Performance Organization* with Harvard Business School Press in **1993** (reissued 2015). The research base was a study of **50 teams in more than 30 companies** — from Motorola and Hewlett-Packard to Operation Desert Storm planning and the Girl Scouts. The book's core contribution is not motivational; it is a **working definition and a discipline**: teams are not discovered, they are *built to a specification*, and the specification is the definition in §1.1.

The three "team basics" the authors derived (verified via Warwick Business School's summary of the theory):

1. **Accountability** — mutual accountability, for the team and each individual.
2. **Commitment** — driven by a meaningful, agreed purpose plus a common approach with specific goals.
3. **Skills** — complementary technical/functional, problem-solving, and interpersonal skills.

And the six factors of an optimally performing team: small enough in number; complementary skills; a truly meaningful purpose (written down, referred to frequently); specific goals; a clear, adaptable working approach; and a genuine sense of mutual accountability — *only the team succeeds or fails, not individuals*.

### 2.2 The discipline of teams

The title's word — *Wisdom* — is a deliberate pun: the book argues there is a **discipline** of teams, in the sense of a repeatable practice, and that "teamwork" is usually over-celebrated and under-defined. The discipline has four practical moves, all still current:

1. **Forge a common purpose** — the team shapes it, the lead facilitates; a purpose the team merely receives is a directive, not a purpose.
2. **Translate purpose into specific performance goals** — goals are what drive behaviour, focus conflict onto the work, and make progress measurable.
3. **Define a common working approach** — how work will be divided, integrated, decided, and reviewed; deliberately designed, not emergent.
4. **Build mutual accountability** — through the purpose, goals, and approach; it is an *outcome* of the first three moves, not a speech the lead gives.

A fifth contribution is the **team performance curve** (§1.2) and its sharpest warning: the **pseudo-team**. When an organisation says "we are a team" but rewards individuals and never builds a common purpose, it gets a pseudo-team — which underperforms an honest working group. The discipline is also what separates a *real team* from a *high-performance team*: the HPT adds deep mutual commitment to one another's growth, and that extra quality is *earned* through the discipline, not declared.

### 2.3 Working group vs team vs HPT — the comparison

| Dimension | Working group | Real team | High-performance team |
|---|---|---|---|
| Accountability | Individual | Individual + mutual | Mutual, felt as obligation |
| Work products | Individual | Collective | Collective, exceeding expectations |
| Purpose | Assigned | Common, shaped by team | Common, deeply owned |
| Goals | Individual targets | Team performance goals | Stretch goals, self-imposed |
| Interaction quality | Cordial, informational | Collaborative, task-focused | Deeply committed to each other's growth |
| Typical result | Sum of individuals | Meets/exceeds goals | Outperforms any reasonable expectation |

### 2.4 The canon table

The research canon of the HPT discipline — the works this guide treats as load-bearing. Each entry names the work, the year, the verified core idea, and what it contributes to the discipline.

| Work | Author(s) | Year (verified) | Core idea | Contribution to the discipline |
|---|---|---|---|---|
| *The Wisdom of Teams* | Katzenbach & Smith | 1993 | The definition: small number, complementary skills, common purpose, performance goals, mutual accountability; the performance curve | The HPT **definition and discipline** — the specification for what a team is |
| "Developmental Sequence in Small Groups" | Bruce Tuckman | 1965 | Groups develop through forming–storming–norming–performing (adjourning added 1977 with Jensen) | The **developmental lens** — teams earn performance through stages; storming is necessary (see §6.1, cross-ref team_lead §2.1) |
| *Management Teams: Why They Succeed or Fail* | R. Meredith Belbin | 1981 | Nine team roles; balance, not brilliance, predicts team success | The **composition lens** — complementary skills made concrete (see §6.3) |
| *The Five Dysfunctions of a Team* | Patrick Lencioni | 2002 | Trust → conflict → commitment → accountability → results pyramid | The **failure diagnosis** — why teams break, bottom-up (see §6.2, cross-ref team_lead §2.3) |
| *Leading Teams* | J. Richard Hackman | 2002 | Five conditions set the stage for great performances: real team, compelling direction, enabling structure, supportive context, expert coaching | The **effectiveness conditions** — what leaders can actually design (see §5) |
| "Psychological Safety and Learning Behavior in Work Teams" | Amy Edmondson | 1999 | Team psychological safety enables learning behavior, which mediates performance | The **safety foundation** — the interpersonal risk condition (see §4) |
| Project Aristotle (re:Work) | Google People Analytics | 2012–2015 | Five dynamics; psychological safety is the #1 predictor of team effectiveness | The **evidence base** — a large-N confirmation of the canon (see §3) |
| Accelerate State of DevOps / *Accelerate* | DORA (Forsgren, Humble, Kim et al.) | 2018 report; 2018 book | Four delivery metrics separate elite from low performers | The **engineering measurement** (see §7) |
| "The SPACE of Developer Productivity" | Forsgren, Storey, Maddila, Zimmermann, Houck, Butler | 2021 | Productivity has five dimensions: Satisfaction, Performance, Activity, Communication & Collaboration, Efficiency & Flow | The **engineering balance** — beyond activity metrics (see §7) |

### 2.5 Canon in practice

The canon is not a reading list; it is a **diagnostic sequence**. When a team underperforms: (1) check the definition — is it a team at all, or a group/pseudo-team? (2) check the conditions — are the five Hackman conditions in place? (3) check the safety — can people speak up? (4) check the dynamics — which Tuckman stage, which Lencioni dysfunction? (5) check the engineering — are the delivery metrics healthy? Each check maps to a section of this guide. The single most common canon violation in enterprise IT: an organisation *calls* a component team a team, rewards individuals, and never builds mutual accountability — a textbook pseudo-team.

### 2.6 Misreading the canon — three common errors

The canon is frequently cited and rarely read; the three most common misreadings are worth naming so this guide's readers can avoid them:

1. **"Teams are for everyone."** Katzenbach & Smith are explicit that a *working group* is often the right answer — when interdependence is low and individual accountability suffices, forcing teamwork creates a pseudo-team that underperforms an honest group. Team formation is a decision, not a default.
2. **"The definition is the goal."** The five-component definition is the *minimum specification* for a real team, not the ceiling. The high-performance team adds the extra quality — members deeply committed to one another's personal growth — which is earned through the discipline, not achieved by writing a purpose statement.
3. **"Teams happen naturally."** The book's whole argument is the opposite: teams are *built to a discipline*. The belief that "we're all friends, we'll figure it out" is precisely how pseudo-teams form — no common purpose, no specific goals, no common approach, and mutual accountability never emerging.

The practical test of a correct reading: when a team underperforms, the canon-trained lead asks *design questions* (Is it a team? Are the conditions set? Is it safe to speak?) — not motivational ones (Are they committed? Do they care?).

---

## 3. The Evidence Base

### 3.1 Google Project Aristotle (2015) — verified

**Origin (verified):** Project Aristotle was a research program by **Google's People Analytics team**, launched in **2012**, asking one question: *what makes a team effective at Google?* It analysed **180+ teams** across **250+ attributes** — team composition, personality, tenure, interaction patterns, and more. The name references Aristotle's quote that "the whole is greater than the sum of its parts" (verified via Google's re:Work site). Findings were published on Google's **re:Work** platform from **2015**, and popularised by Charles Duhigg's New York Times Magazine article "What Google Learned From Its Quest to Build the Perfect Team" (February 2016). The teams studied ranged from 3 to 50 people (median 9).

The study's method matters as much as its findings: Google's leaders initially pushed for objective measures (lines of code, bugs fixed) and the researchers **rejected them all as flawed** — more code isn't better, more bugs fixed means more bugs created. Effectiveness was instead measured through a combination of qualitative assessments from three perspectives (executives, team leads, team members) plus quantitative output measures.

### 3.2 The five dynamics — verified

The research surfaced **five dynamics** that consistently distinguish effective teams (verified via re:Work's "Understand team effectiveness" guide):

1. **Psychological safety** — team members feel safe to take interpersonal risks: admit mistakes, ask questions, challenge ideas, propose half-baked thoughts, without fear of punishment or humiliation. *The #1 predictor.*
2. **Dependability** — team members reliably complete their work on time and to standard; you can count on each other.
3. **Structure & clarity** — roles, goals, and plans are clear; everyone knows what is expected of them and how the team's work fits together. (Google's own operating tool for this is OKRs.)
4. **Meaning** — the work is personally meaningful to the members; it matters to them, not just to the org chart.
5. **Impact** — members believe their work is making a difference; they can see the effect of their effort on users, the business, or the organisation.

**The headline finding (verified):** psychological safety was the **single most important** factor separating high-performing teams from the rest — and it was the *only* dynamic that predicted effectiveness across every team type studied.

### 3.3 What Aristotle did NOT find

Equally important is what the data ruled out. Who was *on* the team — the mix of personalities, seniority, extroversion, or individual IQ — did **not** predict team effectiveness. Two teams with identical rosters could perform completely differently depending on *how they worked together*. The team's **norms and interaction patterns** were what mattered — which is why the finding is so empowering for leads: you cannot change who is hired easily, but you *can* change how the team interacts. The famous illustration from the NYT piece: a team of high-IQ, high-ego stars underperformed a less individually-gifted team that had better safety and turn-taking norms.

### 3.4 The evidence table

| Dynamic | What it means | Observable signals (good) | Observable signals (bad) | Leader levers |
|---|---|---|---|---|
| **Psychological safety** (#1) | Safe for interpersonal risk-taking | People admit mistakes in retros; juniors challenge seniors; questions are common | Silence in meetings; blame after incidents; "that's a dumb question" culture | Model vulnerability; respond to bad news with curiosity; invite dissent explicitly (see §4) |
| **Dependability** | Commitments are kept | Sprint commitments hold; handoffs happen on time; no surprises at review | Missed deadlines rationalised; estimates meaningless; people cover for each other | Clear role expectations; visible commitments; address slippage early (see [grow_team_guide.md](grow_team_guide.md) §6) |
| **Structure & clarity** | Roles, goals, plans are clear | Everyone can state the team's goal and their role; meetings have agendas | "What are we doing and why?" confusion; duplicated work; surprise ownership gaps | Charter (§8.1); team goals (§8.2); OKRs (cross-ref team_lead §5.1) |
| **Meaning** | Work matters personally | People connect tasks to outcomes; low attrition; energy in planning | "It's just a ticket" cynicism; disengagement; people leave | Connect work to user/business impact; surface the "why" regularly (§4.4 framing) |
| **Impact** | Work makes a difference | The team can name who benefits; progress is visible; wins are celebrated | Work vanishes into the org; no feedback from users; "we never see the result" | Show the outcome; user-centred evaluation; celebrate team-level wins (cross-ref team_lead §2.3 — inattention to results) |

### 3.5 Aristotle and the canon

Project Aristotle is best read as a **large-N confirmation** of the qualitative canon that preceded it: the "safe for interpersonal risk-taking" construct is Edmondson's (1999) definition verbatim; "structure & clarity" echoes Hackman's compelling direction and enabling structure; "dependability" is mutual accountability operationalised; "meaning" and "impact" are Katzenbach & Smith's common purpose made motivational. The evidence base does not replace the canon — it ranks it. And it ranks **safety first**, which is why the next section is the deepest one in this guide.

### 3.6 How the measurement was done — and why it matters

Three details of Aristotle's method are discipline-relevant, not trivia:

- **The effectiveness measures were qualitative triangulations, not raw metrics.** Google's leaders pushed for objective measures (lines of code, bugs fixed, customer satisfaction) and the researchers demonstrated each was flawed — more code is not better, more bugs fixed means more bugs were created. Effectiveness was assessed by rating teams from *three perspectives* — executives, team leads, and team members — each of which turned out to weight different aspects. The lesson: effectiveness is a judgment call triangulated across perspectives, which is why §7's SPACE framework insists no single metric can stand in for it.
- **The teams studied ranged from 3 to 50 people, with a median of 9.** This is an independent, large-N confirmation of the "small number" component of the Katzenbach & Smith definition — interdependence and intimacy are what make a team a team.
- **Teams were distinguished from work groups by interdependence** (verified: teams "plan work, solve problems, make decisions, and review progress in service of a specific project"; work groups "are based on organizational or managerial hierarchy"). The researchers let the teams themselves identify their interdependent working relationships rather than trusting the org chart — a reminder that the org chart's "team" labels are often fiction (§1.2).

### 3.7 Section summary

- Project Aristotle (2012–2015, 180+ teams, verified) found **five dynamics**: psychological safety, dependability, structure & clarity, meaning, impact.
- **Psychological safety is the #1 predictor** — and the only dynamic that predicted effectiveness across all team types.
- Team *composition* matters far less than *interaction norms* — the team's way of working is the leverage point.
- The evidence ranks the canon: safety first, then dependability and clarity, then meaning and impact.
## 4. The Psychological Safety

### 4.1 Edmondson 1999 — verified

**Origin (verified):** Amy Edmondson (Harvard Business School) introduced the construct of **team psychological safety** in "Psychological Safety and Learning Behavior in Work Teams," *Administrative Science Quarterly* **44(2), June 1999, pp. 350–383**. The paper — which Edmondson wrote while at Harvard, and for which she thanks Richard Hackman for advice on the study design (a nice proof that the canon is a conversation, not isolated silos) — presents a model of team learning tested in a **multimethod field study of 51 work teams in a manufacturing company**. The verified definition from the abstract: team psychological safety is *"a shared belief held by members of a team that the team is safe for interpersonal risk taking."*

The study's verified findings:

- Team psychological safety is **associated with learning behavior** (asking for feedback, seeking help, discussing errors, experimenting, reflecting on process).
- **Learning behavior mediates the relationship between psychological safety and team performance** — safety produces performance *through* learning.
- Interestingly, **team efficacy was not** associated with learning behavior when psychological safety was controlled for — confidence alone does not produce learning; safety does.
- Structural factors (context support, team leader coaching) and shared beliefs together shape team outcomes — the paper's "integrative perspective" that links back to Hackman's structural conditions (§5).

A crucial subtlety from the 1999 study and its famous 1996 predecessor (Edmondson's hospital study on medication errors, "Learning from mistakes is easier said than done," 1996): **better teams reported *more* errors** — because they felt safe enough to report them. Teams that look error-free are often teams that hide errors. This is the single most counter-intuitive finding in the HPT literature and it is the key to why safety feels soft but is hard-edged: without it, the organisation cannot see its own defects, and learning stops.

### 4.2 Why safety produces performance

Psychological safety is *not* "everyone feels comfortable" (Edmondson is explicit that comfort is not the goal). It is the condition that makes **learning and candour** possible. The mechanism:

1. **Error visibility** — mistakes are reported, so the system can fix them (banks: incident post-mortems only work if people tell the truth; see the blameless-retro link in team_lead §2.5).
2. **Voice** — juniors challenge seniors, dissent happens in the room instead of the corridor, half-formed ideas get aired and improved.
3. **Experimentation** — teams try new approaches when failure is survivable and discussable.
4. **Help-seeking** — asking for help is normalised, so expertise actually flows (the 1999 study's "asks for help from others in the company" is a measured learning behavior).

Each of these is *measurable*: the 1999 paper's observer survey items include "actively reviews its own progress," "regularly takes time to figure out ways to improve its work performance," and "asks its internal customers for feedback." You can audit safety by watching whether those behaviors happen.

### 4.3 *The Fearless Organization* (2018) — verified

**Origin (verified):** Amy Edmondson's book *The Fearless Organization: Creating Psychological Safety in the Workplace for Learning, Innovation, and Growth* (Wiley, **2018**, ISBN 978-1-119-47724-2) is the definitive practical statement of the research program that began in 1999. Its verified core claims:

- Psychological safety is **the foundation of learning organisations** — without it, the "learning organization" is a slogan.
- It is a **team-level property** ("a shared belief"), not an individual personality trait and not a culture poster.
- It must be paired with **accountability and high standards** — safety is *not* the absence of standards; the goal is a team that is *both* safe *and* demanding (see §4.4).
- Leaders create it through **three levers** (verified framing from the book and Edmondson's HBS teaching): (1) **frame the work** — set up the situation as a learning problem rather than an execution problem; (2) **invite participation** — ask good questions, model not-knowing, create structured opportunities for voice; (3) **respond productively** — how you react to bad news, mistakes, and challenges teaches the team what is actually safe.

### 4.4 The safety × standards matrix

Edmondson's most useful practical model (verified as hers) is the two-by-two of **psychological safety** (low/high) × **accountability standards** (low/high):

| | Low standards | High standards |
|---|---|---|
| **Low safety** | **Apathy zone** — nobody cares, nobody speaks. The dead zone. | **Anxiety zone** — high pressure, no voice: people fear speaking up, errors hide, burnout thrives. Common in "stretch target" cultures without safety. |
| **High safety** | **Comfort zone** — everyone feels great, nothing demanding happens. The "country club" team. | **Learning zone** — the target: high standards *and* high safety. People challenge, experiment, admit errors, and hold each other to the bar. |

The practical message: **raising standards without raising safety creates the anxiety zone** — the worst of both worlds, and a trap in target-driven enterprises. The HPT discipline pairs demanding goals (§8.2) with safety-building responses (§4.5).

### 4.5 The safety table

| Myth | Reality (verified) | Evidence | Practical consequence |
|---|---|---|---|
| "Safety means being nice / no conflict" | Safety is the condition for *productive* conflict and candour | Edmondson 1999 — safety enables voice, which surfaces disagreement | Encourage debate in the room; safety makes conflict safe, not absent (cross-ref team_lead §2.3, §9) |
| "Better teams report fewer problems" | Better teams report *more* problems, because they can | 1996 hospital study; 1999 manufacturing study | Celebrate error discovery; never punish the reporter (bank post-mortems!) |
| "Safety is a personality thing" | It is a team-level shared belief, shaped by the leader's responses | 1999 definition ("shared belief held by members of a team"); re:Work on leader modeling | Treat safety as a team design variable, not a hiring filter |
| "Safety kills accountability" | Safety without standards is comfort; safety *with* standards is the learning zone | *The Fearless Organization* 2018 (§4.4 matrix) | Pair safety with demanding goals; the two reinforce |
| "You can't measure safety" | It is measured in the literature with validated scales; you can audit it with health checks | 1999 paper's scales; Google re:Work tips; team health checks (team_lead §2.6) | Run a safety-inclusive health check quarterly |

### 4.6 Safety in the bank

In a regulated bank, safety work is *structural*, because the environment generates fear: mandatory incident post-mortems, audit findings, compliance deadlines, and a risk culture that historically asks "who is accountable?" first. The lead's counter-moves (detailed in team_lead §2.5 and §9.6): blameless retros that separate *system* from *person*; responding to bad news with "what happened?" rather than "who did this?"; inviting dissent explicitly ("tell me what's wrong with this plan"); and watching your own micro-reactions — the raised eyebrow after a status update teaches more than any stated value. In a regulated bank, psychological safety is not a soft luxury; it is the only way to get *accurate* risk information out of the people who hold it.

### 4.7 The three levers in practice — scripts

Edmondson's three levers are abstract until they have language. Concrete scripts, calibrated for the bank-IT setting:

**1. Frame the work** — set the situation up as a learning problem, not a pure execution problem. Before a risky integration: *"This is new territory for all of us — the requirements are ambiguous and the rails are unforgiving. We will make mistakes; our job is to find them fast and learn from them."* The counter-frame to avoid: *"This must be perfect; failure is not an option"* — which reads as "hide your mistakes," the anxiety zone (§4.4). Framing is not spin; it must be true. If the work genuinely has no tolerance for error (a regulatory deadline), say so honestly and then make the *reporting* of near-misses the valued behaviour.

**2. Invite participation** — replace "any questions?" (answered with silence) with targeted invitations: *"Tell me what's wrong with this plan."* *"What am I missing?"* *"Who has seen something like this fail before?"* Run structured round-robins in planning so the quiet members speak before the loud ones; ask juniors first, so the senior voice does not set the ceiling. The invitation must be *specific* — generic openness reads as theatre.

**3. Respond productively** — the third lever is where safety is actually built or destroyed, because it is the *evidence* the team uses to calibrate. To bad news: *"Thank you for surfacing that — that took courage. What happened, and what do we need?"* To a wrong call: *"That was a reasonable decision with the information we had. What did we learn, and what changes?"* To a challenge of your own idea: *"Good catch — let's look at that."* Never punish the messenger, even privately; never let a team member's candour be followed by silence and a colder 1:1. Teams track the leader's *responses* far more accurately than the leader's *words* — the calibration is continuous and unforgiving.

---

## 5. The Effectiveness Conditions

### 5.1 Hackman's model — verified

**Origin (verified):** J. Richard Hackman, the Harvard psychologist who spent four decades studying work teams, published *Leading Teams: Setting the Stage for Great Performances* (Harvard Business School Press, **2002**), synthesising research on settings "ranging from orchestras to airline cockpit crews to economic analysts." His verified core claim: **team effectiveness is not produced by heroic leadership in the moment; it is produced by conditions the leader sets in advance.** The five conditions (verified via HBS Press and multiple academic summaries):

1. **A real team** — clear boundaries, stable membership, interdependence: members need one another to get the work done, and the team has a defined existence.
2. **A compelling direction** — a purpose that is clear, challenging, and consequential; it energises, orients, and engages the team.
3. **An enabling structure** — the task design, team composition, and core norms of conduct make teamwork possible: a well-designed task (whole, meaningful, autonomous, with feedback), the right mix of people, and explicit norms.
4. **A supportive organizational context** — the reward, information, education, and material-resource systems of the organisation support rather than undermine the team.
5. **Expert coaching** — available coaching in the *process* of teamwork (how the team works together), offered at the right moments, especially at the beginning, at midpoints, and at the end of a work cycle.

### 5.2 The conditions as a design checklist

Each condition is a question the lead can answer with evidence:

| Condition | The design question | Bank/IT translation |
|---|---|---|
| Real team | Is this a bounded, stable, interdependent unit — or a pool of individuals on a shared project? | A squad with stable membership and a mission (see [grow_team_guide.md](grow_team_guide.md) §7) vs a matrix "team" that changes every quarter |
| Compelling direction | Does the team know *why* it exists and is that why energising? | A mission tied to a business outcome (e.g., "reduce trade-processing breaks") vs a pile of IT tickets |
| Enabling structure | Is the task well-designed? Right people? Explicit norms? | Cross-functional squad owning end-to-end delivery; charter norms (team_lead §2.4) |
| Supportive context | Do rewards, information, resources, and education support the team? | Team-level goals in a system that rewards individual annual reviews; access to environments and data |
| Expert coaching | Is someone coaching the *process* of teamwork, not just the content? | A squad lead who runs retros and conflict conversations (team_lead §3–§9), not just a tech lead |

### 5.3 The input–process–output lineage

Hackman's conditions sit at the end of a research lineage this guide should name, because it explains *why* conditions are the right level of intervention:

- **McGrath (1964)** — the classic **input–process–output (IPO)** model of group effectiveness: inputs (member attributes, group structure, environment) → processes (interaction) → outputs (performance, satisfaction).
- **Hackman & Morris (1975)** — group performance depends on member *knowledge and skills*, *effort*, and *performance strategies* — and process losses (coordination, motivation) can destroy all three.
- **Hackman's normative model (1987)** — effectiveness criteria plus process criteria: a team is effective when (a) its output meets the standards of those who receive it, (b) the team's social processes enhance members' capability to work together in the future, and (c) members' growth and well-being are served.
- **Leading Teams (2002)** — the five conditions as the *designable inputs*: leaders cannot command processes, but they can set conditions that make good processes (and good outputs) overwhelmingly likely.

The lineage matters because it defeats two seductive errors: (1) *"just hire better people"* (composition is one input among several — and Aristotle showed interaction norms dominate); (2) *"just fix the process"* (processes are emergent; you can only shape the conditions around them). This guide's §8 (team design) is the practical translation of the IPO view.

### 5.4 The conditions table

| Condition (verified) | Definition | If missing, you see… | First move |
|---|---|---|---|
| Real team | Bounded, stable, interdependent | Churn, vague ownership, "whose job is this?", no collective identity | Fix membership and boundaries before anything else; a team that keeps changing members cannot build safety (Section 4) |
| Compelling direction | Clear, challenging, consequential purpose | Busy-ness without energy; people can't answer "why are we doing this?" | Run a purpose-forging session; write the mission; tie it to a business outcome (§8.2) |
| Enabling structure | Task design + composition + norms | Friction everywhere; blame cycles; duplicated or dropped work | Redesign the task for whole, meaningful, autonomous work; charter the norms (team_lead §2.4) |
| Supportive context | Reward/info/resource/education systems align | Team goals that contradict individual incentives; no data access; training gaps | Escalate the contradictions; make the team's rewards match its goals (cross-ref [grow_team_guide.md](grow_team_guide.md) §6) |
| Expert coaching | Process coaching at the right moments | Teams that repeat the same retro complaints; conflict cycles | Coach *how* the team works — at kickoff, midpoints, and endings; use the retro as a coaching instrument (team_lead §8.5) |

### 5.5 The three effectiveness criteria

Before the conditions, Hackman's definition of effectiveness itself (verified — carried from his 1987 normative model into *Leading Teams*): a team is effective when **all three** hold:

1. **The output meets the standards of those who receive or use it** — the client, the user, the business. Not "did we ship" but "did they get value."
2. **The team's social processes enhance members' capability to work together in the future** — the team leaves each interaction *more* able to collaborate, not less. A team that wins by burning trust is destroying its own future.
3. **Members' growth and personal well-being are served** — the experience of the team makes members better professionals and healthier people.

The three criteria explain two recurring findings in this guide: Project Aristotle's result that psychological safety is the #1 predictor (safety is what makes criterion 2 and 3 achievable), and DORA's finding that elite delivery teams have *lower* burnout (high performance without member well-being is not effectiveness — it is a burn-out in progress). Any intervention that improves the numbers while destroying criteria 2 or 3 is not a team fix; it is a team tax.

### 5.6 Section summary

- Hackman (2002, verified) reframes leadership: **you cannot command a team to greatness; you set conditions** — real team, compelling direction, enabling structure, supportive context, expert coaching.
- The conditions descend from the input–process–output tradition (McGrath 1964; Hackman & Morris 1975): design inputs, don't micromanage processes.
- Effectiveness has three criteria: output quality, future capability, and member growth — a team that burns out its members is not effective, however fast it ships.
- The conditions are the *design contract* for Section 8.

---

## 6. The Team Dynamics

### 6.1 Tuckman (1965) — verified, cross-ref

**Origin (verified):** Bruce Tuckman's *forming–storming–norming–performing* model (with **adjourning** added in 1977 with Mary Ann Jensen) is verified and treated in depth in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §2.1–2.2 — the stage table, the stage-recognition symptoms, the leader interventions, and the two classic mistakes (suppressing storming; staying in directing mode too long). This guide cross-refs rather than re-derives; the discipline-relevant points:

- **Storming is necessary, not pathological** — teams that skip or suppress it reach only *fake norming* and never perform. The lead's job in storming is containment and channelling, not peacekeeping.
- **Stages are diagnostic, not labels** — teams regress (new member, reorg, crisis) and can be in different stages on different topics.
- **The HPT link:** the performance curve (§1.2) and Tuckman's stages align — a pseudo-team is usually stuck in fake-norming; a high-performance team is in performing *with* the capability to re-enter storming productively when challenged.

### 6.2 Lencioni (2002) — verified, cross-ref

**Origin (verified):** Patrick Lencioni's *The Five Dysfunctions of a Team* (Jossey-Bass, **2002**) is verified and treated in depth in team_lead_methodologies_guide.md §2.3 (the trust → conflict → commitment → accountability → results pyramid, the counters for each level). The discipline-relevant points:

- The pyramid is a **root-cause ladder**: symptoms at the top (inattention to results) usually have causes two or three levels down (fear of conflict rooted in absence of trust).
- **The HPT link:** Lencioni's pyramid is the *failure mode* of the HPT discipline — every dysfunction is a violation of one of Katzenbach & Smith's five components (trust/safety → interpersonal risk; commitment → common purpose; accountability → mutual accountability; results → performance goals). Build the discipline and you are building down the pyramid's failure modes; diagnose the team and you are climbing it.

### 6.3 Belbin (1981) — verified

**Origin (verified):** R. Meredith Belbin's *Management Teams: Why They Succeed or Fail* (Heinemann, **1981**) reported on a decade of observational research at **Henley Management College** (begun in the 1960s) using a simulated management game. The verified finding: teams predicted to excel on intellect alone often failed — **it was balance, not brilliance, that predicted team success**. Nine behavioural clusters — **team roles** — emerged (the original 1981 set had eight; the ninth, *Specialist*, was added in later revisions, and two were renamed):

| Role (current name) | Original name (1981) | Behavioural contribution |
|---|---|---|
| **Plant** | Plant | Creative, unorthodox idea generator |
| **Resource Investigator** | Resource Investigator | Networker; brings outside ideas and contacts; enthusiasm at the start |
| **Co-ordinator** | Chairman | Clarifies goals, delegates, sees the big picture |
| **Shaper** | Shaper | Drive and energy; challenges the team; pushes through obstacles |
| **Monitor Evaluator** | Monitor-Evaluator | Logical, impartial judge of options |
| **Teamworker** | Team Worker | The "oil between the cogs"; listens, smooths conflict |
| **Implementer** | Company Worker | Turns ideas into action; reliable, disciplined delivery |
| **Completer Finisher** | Completer-Finisher | Perfectionist; double-checks; closes things out |
| **Specialist** | — (added later) | Deep expert knowledge in a narrow domain |

**Use and caution (verified):** Belbin roles are behavioural preferences, not personality types, and the inventory is a consulting instrument — independent psychometric studies have questioned its reliability (flagged honestly: the academic record is mixed, and Belbin himself argues the tool is for practice, not psychometrics). The discipline-relevant insight survives the critique: **composition is a design variable** — check for role coverage (do you have a Shaper and a Completer Finisher? a Plant and a Co-ordinator?) and for *overlaps* (three Shapers = a knife fight; no Implementer = great ideas, nothing shipped). Belbin's own (perhaps extreme) claim is that the optimum team size is about four — a useful counterweight to "the more the merrier."

### 6.4 The dynamics table

| Model (verified) | Year | What it explains | Unit of analysis | How the lead uses it |
|---|---|---|---|---|
| Tuckman stages | 1965 (+1977) | *How teams develop over time* | The team's stage | Diagnose where the team is; match intervention (structure in forming, containment in storming, codify in norming, protect in performing) — team_lead §2.1–2.2 |
| Lencioni dysfunctions | 2002 | *Why teams break* | The team's failure pyramid | Climb from symptom to root cause; fix trust and conflict before results — team_lead §2.3 |
| Belbin roles | 1981 | *How individual behaviours combine* | Individual roles in the team | Check composition coverage and overlaps when forming/backfilling a team; pair for complementary strengths |
| Project Aristotle dynamics | 2015 | *What predicts effectiveness* | Team norms | Rank the intervention priorities: safety first, then dependability, clarity, meaning, impact — §3 |
| Hackman conditions | 2002 | *What leaders can design* | The team's environment | Set the five conditions before demanding performance — §5 |
| Katzenbach & Smith discipline | 1993 | *What a team is* | The team's definition | Use the five-component definition as the specification for every new team — §2 |

### 6.5 Dynamics × discipline

The dynamics models answer *where the team is and what is breaking*; the HPT discipline answers *what the team is and how to build it*. They compose: **Tuckman** tells you the stage (and that storming is coming); **Lencioni** tells you which failure mode is active (and that it is probably trust or conflict); **Belbin** tells you whether the *mix* can perform once the dynamics are fixed; **Aristotle** tells you what to fix first; **Hackman** tells you what to design; **Katzenbach & Smith** give you the specification. A lead who uses only one of these is working with a single lens; the discipline uses all six as a stack.

### 6.6 The dynamics in a bank team

The dynamics models need one enterprise-specific gloss, because the bank environment distorts each of them. **Tuckman:** bank teams are perpetually pushed back to forming by re-staffing, reorgs, and project-based staffing — the lead must treat "we keep regressing" as a structural symptom, not a team failure, and re-run the forming rituals (charter, early wins) after every membership change (cross-ref [grow_team_guide.md](grow_team_guide.md) §4 on scaling and re-teaming). **Lencioni:** the bank's matrix makes the avoidance-of-accountability and inattention-to-results dysfunctions almost *rational* — when rewards are individual and the org chart is a matrix, why would a peer call out a peer? The lead must counter with team-level goals and published commitments (§8.2, §8.4). **Belbin:** matrix staffing also wrecks role coverage — a "team" assembled from a skills pool is a random draw of roles. The composition check (§6.3) must happen at formation, with the lead explicitly negotiating for missing coverage (a Completer Finisher matters enormously in a bank, where "done" includes audit-ready documentation). The models are not abstract in the enterprise; they are the vocabulary for what the organisation is doing to the team — and what the lead must push back on.

---

## 7. The Engineering Teams

### 7.1 DORA — verified

**Origin (verified):** The **DevOps Research and Assessment (DORA)** program — the research behind the annual *Accelerate State of DevOps* reports (published since ~2014; the four metrics were codified in the 2018 report) and the book *Accelerate: The Science of Lean Software and DevOps* (Nicole Forsgren, Jez Humble, Gene Kim; IT Revolution Press, **2018**) — studied thousands of software teams and found that **software delivery performance separates into distinct performance clusters** (elite, high, medium, low). The verified **four key metrics**:

1. **Deployment frequency** — how often the team successfully releases to production.
2. **Lead time for changes** — the time from commit to running in production.
3. **Change failure rate** — the percentage of deployments that cause failure in production.
4. **Time to restore service (MTTR)** — how long it takes to recover from a failure.

The verified findings around the metrics: **elite performers deploy on demand, have lead times of under a day, change-failure rates well under half, and restore service in under an hour** — and they achieve this *while* having lower burnout and higher stability. The metrics are correlated with each other (you cannot game one without the others moving), and the research links them to organisational performance. *Flag (honest):* the exact elite/low benchmark numbers vary by report year, and later reports (2021 onward) added **reliability** as a fifth metric; this guide gives the four canonical metrics and flags the extension rather than presenting either as a fixed eternal table.

### 7.2 Why DORA metrics are team design, not just reporting

The four metrics are **outcomes of team design**, which is why they belong in an HPT guide:

- **Deployment frequency and lead time** are functions of *autonomy* (can this team deploy without another team's approval?) and *architecture* (can it release its part independently?) — the squad/topology questions of [grow_team_guide.md](grow_team_guide.md) §7 and the quality-culture practices of the dev-tooling cluster (see §7.4).
- **Change failure rate and time to restore** are functions of *safety* — both technical (tests, CI, monitoring, rollback) and interpersonal (do engineers report incidents honestly? the Edmondson link again: teams that hide incidents have *lower* measured failure rates and *higher* real risk).
- DORA's own finding that elite teams have **lower burnout** is the engineering proof of Hackman's third effectiveness criterion (member growth and well-being).

### 7.3 SPACE (2021) — verified

**Origin (verified):** "The SPACE of Developer Productivity: There's More to It Than You Think" by Nicole Forsgren, Margaret-Anne Storey, Chandra Maddila, Tom Zimmermann, Brian Houck, and Jenna Butler (*ACM Queue*, **February 2021**, Vol. 19(1), pp. 20–48) — the same research community as DORA, responding to a real problem: **productivity is not a single number, and activity is not productivity**. The verified framework has five dimensions:

| Dimension | Question it answers |
|---|---|
| **S**atisfaction | How happy and fulfilled are developers with their work, team, tools, and culture? |
| **P**erformance | What is the *outcome* of the work — quality, speed, value delivered? |
| **A**ctivity | What are people actually doing — commits, PRs, reviews, meetings? (The *most measurable and most misleading* dimension) |
| **C**ommunication & collaboration | How do people work together — handoffs, reviews, coordination, knowledge sharing? |
| **E**fficiency & flow | How much of the work is wasted — context switching, waiting, rework, toil? |

The verified core argument: measuring **activity alone** (commits, story points, hours) is not just incomplete — it is *actively harmful*, because it rewards visible busy-ness over outcomes, discourages collaboration (which is invisible in activity counts), and punishes the careful work (design, review, refactoring) that produces performance. Teams should pick a small number of measures spanning multiple SPACE dimensions, chosen *with the team*, and treat them as diagnostic signals rather than targets.

### 7.4 The engineering table

| Instrument (verified) | What it measures | Unit | Strengths | Traps | Use in the HPT discipline |
|---|---|---|---|---|---|
| DORA deployment frequency | Release cadence | Releases/time | Objective, outcome-linked, correlates with org performance | Gaming via micro-releases; meaningless where compliance gates dominate | The delivery *outcome* signal for the squad; baseline and trend it (§8.3 feedback loops) |
| DORA lead time for changes | Commit → production | Time | Objective; reveals integration bottlenecks | Depends on architecture (monolith vs microservices) | The autonomy/architecture diagnostic — long lead time usually means handoffs and approvals |
| DORA change failure rate | Failed deployments | % | Objective; the safety-of-release signal | Under-reported where blame culture hides incidents (Section 4!) | The *technical* safety twin of psychological safety |
| DORA time to restore | Failure → recovery | Time | Objective; the resilience signal | Rare events → noisy data | Retro material: every incident is a learning loop |
| SPACE dimensions | Productivity breadth | Qualitative + quantitative mix | Fixes the activity-metric blindness; team-owned | Subjective if not anchored; easy to over-measure | The *balance* check: are we measuring outcome, not just activity? |
| Team health checks | Team state (delivery, fun, learning, etc.) | Self-rated traffic lights | Cheap, frequent, catches drift early | Anonymity required; leads must not bias | The *people* loop that DORA/SPACE cannot see (team_lead §2.6) |

### 7.5 Engineering HPT in the bank — the honest flag

In a regulated bank, the DORA metrics cannot be adopted naively: deployment frequency is constrained by **change-management, release-window, and audit requirements**, and "elite" benchmarks assume a continuous-delivery environment most banking platforms do not have. The honest adaptation: use DORA as a **relative trend signal for the team** (are we getting faster at releasing *within our constraints*? is our failure rate falling?) rather than an absolute benchmark against public data; use SPACE to keep the productivity conversation honest when activity metrics (tickets closed, story points) dominate; and treat the *quality culture* — test coverage, CI discipline, post-incident learning — as the real objective, with the dev-tooling cluster as the reference (see [../technology/shell_testing_frameworks_comparison_guide.md](../technology/shell_testing_frameworks_comparison_guide.md) and [../technology/bashcov_vs_kcov_guide.md](../technology/bashcov_vs_kcov_guide.md) for the coverage-quality angle, and [../technology/data_architect_skillgaps_guide.md](../technology/data_architect_skillgaps_guide.md) for the skill-gap view of what a squad needs to operate those tools).

### 7.6 The capability view — what moves the metrics

DORA is not only a measurement system; the research program also identifies the **capabilities** that move the four metrics — and those capabilities are team-design inputs (Hackman's enabling structure, §5.2) rather than exhortations. The verified headline capabilities include: version control with trunk-based development, continuous integration, automated testing, deployment automation, monitoring and observability, and **loosely-coupled architecture**. One finding is especially relevant to a bank reader (flagged as a DORA report finding, not re-verified here): DORA's research associates *loosened change approval* (peer review of changes rather than external change-advisory boards) with higher delivery performance *without* lower stability — a direct challenge to the centralised change-board model that caps deployment frequency in most banks. The honest synthesis for a regulated environment: the capability list is the engineering agenda (CI, tests, observability, decoupling — see the dev-tooling cluster cross-refs in §7.5), while the change-control finding is a *negotiation target* for one service at a time, proven with the squad's own failure-rate data rather than asserted.

### 7.7 Section summary

- **DORA (verified):** four metrics — deployment frequency, lead time for changes, change failure rate, time to restore service — separate elite from low delivery performers; reliability was added as a fifth metric in later reports (flagged).
- **SPACE (verified, 2021):** productivity has five dimensions (Satisfaction, Performance, Activity, Communication & Collaboration, Efficiency & Flow); activity alone is a harmful measure.
- Engineering performance is *team design*: autonomy, architecture, technical safety, and psychological safety all show up in the metrics.
- In the bank, use the metrics as constrained, team-relative trend signals — and let the quality culture be the target.
## 8. The Team Design

### 8.1 The charter

**The charter (verified as the standard practice of the canon — cross-ref [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §2.4 for the mechanics).** The charter is the team's constitution — created *by* the team *with* the lead, usually in forming, revised in norming. The minimal charter maps exactly onto the Katzenbach & Smith definition:

- **Purpose** — why this team exists, who it serves, what success looks like (1–2 sentences + a success measure). *The common purpose component.*
- **Roles** — who does what, including the lead's role, tech-lead scope, ownership of key areas. *The complementary-skills component made visible.*
- **Norms / working agreements** — the "team contract": how we behave (disagreement in the room, no blame in retro, anyone can call a decision review, focus-time windows, documented handovers). *The common approach component.*
- **Process anchors** — ceremonies, cadence, definition of done, how decisions get made. *The performance-goals component's operating system.*

The charter discipline's two hard rules: (1) a charter written and never referenced is worse than no charter — it teaches the team that agreements are fiction; (2) the lead must *enforce* it — naming violations ("we agreed disagreement happens in the room; that design was changed in the corridor") is the enforcement mechanism, not nagging.

### 8.2 The goals

**The goals (verified — Katzenbach & Smith: performance goals are the engine of the discipline; cross-ref team_lead §5 for OKR/SMART mechanics).** Purpose without goals is a poster; goals without purpose is busy-ness. The discipline's goal rules:

- **Team-level, not just individual** — goals are owned by the team; individual goals derive from them. If only individuals have goals, you have a working group with extra meetings (the pseudo-team trap, §2.2).
- **Specific and measurable** — the goals, not the purpose, drive behaviour; "improve delivery" is a hope, "reduce trade-break fix time from 4h to 90m by Q3" is a goal. SMART mechanics live in team_lead §5.4.
- **Demanding but reachable** — stretch goals energise (Katzenbach & Smith's "significant performance challenges energise teams"); impossible goals create the anxiety zone (§4.4).
- **Aligned upward and visible outward** — the team's goals must map to the programme/business outcomes (OKR alignment, team_lead §5.1–5.2), and they must be *published* — visibility is what makes mutual accountability possible.
- **Reviewed on a cadence** — quarterly goals, sprint goals, reviewed in planning and retro (team_lead §5.5–5.6). Goals that are never revisited are decoration.

### 8.3 The feedback loops

**The feedback loops (verified as the practice dimension of the canon — safety produces learning; learning needs loops; cross-ref team_lead §2.6 and §8.5).** A high-performance team is a *learning system*, and a learning system needs closed loops at four levels:

1. **The work loop** — delivery feedback: CI results, test results, DORA metrics (deployment frequency, lead time, failure rate, restore time — §7), incident reviews. Frequency: continuous to weekly. *This is the loop the engineering metrics feed.*
2. **The improvement loop** — the retro: what went well, what went wrong, what we will change; every retro must produce 1–2 owned actions, or it is a complaint session (team_lead §8.5). Frequency: every iteration (1–2 weeks).
3. **The health loop** — the team health check: anonymous traffic-light ratings on delivery, fun, learning, support, codebase health, etc. (team_lead §2.6). Frequency: quarterly, plus after any major change (new members, reorg).
4. **The safety/learning loop** — blameless post-mortems and explicit "what did we learn" rituals; the loop that only works if psychological safety exists (§4). Frequency: after every incident and every milestone.

The discipline rule for loops: **every loop must end in an action and an owner**, or the team learns that feedback is theatre — which is the fastest way to kill safety.

### 8.4 The accountability

**The accountability (verified — mutual accountability is the fifth component of the K&S definition; the Lencioni pyramid's fourth level; the Aristotle "dependability" dynamic).** Mutual accountability is the *felt* version of dependability — peers holding peers to the bar, not the boss holding individuals. Its mechanics:

- **Visible commitments** — commitments published (sprint board, goals page, team dashboard) so peers can see and reference them. Invisibility is the enemy of accountability.
- **Peer-to-peer challenge** — coached, normalised, low-stakes: "you said you'd have that by Thursday — what changed?" must be a normal sentence. Where it is absent, Lencioni's fourth dysfunction (avoidance of accountability) sets in and the lead becomes the only enforcer — which never scales.
- **Team-level consequences** — celebrate team wins and own team failures as a team; "only the team succeeds or fails, not individuals" (Katzenbach & Smith, verified).
- **The lead's job** — model accountability (own your own misses), name violations of commitments and norms, and *never* rescue the team from its own commitments unless the commitment was structurally impossible (then fix the structure — Hackman's supportive context).

### 8.5 The design table

| Design element | The question it answers | The artifact | Cadence | Canon anchor |
|---|---|---|---|---|
| **Charter** | What are we, why, who does what, how do we behave? | Charter doc + working agreements | Forged in forming; revised in norming; revisited quarterly | K&S definition; Hackman's enabling structure; team_lead §2.4 |
| **Purpose & mission** | Why do we exist and for whom? | Mission statement with success measure | Annual, with quarterly refreshes | K&S common purpose; Aristotle meaning/impact; Hackman's compelling direction |
| **Performance goals** | What specific results will we hit? | Quarterly goals + sprint goals | Quarterly/sprint | K&S performance goals; team_lead §5 (OKR/SMART) |
| **Composition** | Do we have the complementary skills and role coverage? | Role matrix; Belbin-style coverage check; hiring needs | At formation and on every change | K&S complementary skills; Belbin; grow_team §1–§2 |
| **Feedback loops** | How do we learn? | CI/DORA dashboards; retro; health check; post-mortems | Continuous/weekly/quarterly/incident-driven | Edmondson learning behavior; §7 DORA/SPACE; team_lead §2.6, §8.5 |
| **Accountability mechanics** | How do we hold each other to the bar? | Published commitments; peer-challenge norms; team scoreboard | Continuous | K&S mutual accountability; Lencioni level 4; Aristotle dependability |
| **Safety practices** | How do we keep interpersonal risk safe? | Leader modeling; blameless retros; dissent invitations | Continuous | Edmondson 1999/2018; §4 |

### 8.6 The design sequence

The build order matters (it follows the evidence hierarchy of §3): **(1) fix the container** — real team, stable membership, boundaries (Hackman condition 1); **(2) forge the charter** — purpose, roles, norms (the K&S specification); **(3) set the goals** — team-level, specific, visible; **(4) build the safety** — leader goes first, blameless loops, dissent invited; **(5) wire the feedback loops** — retros, health checks, engineering metrics; **(6) hold the accountability** — published commitments, peer challenge, team scoreboard. Skip (1) and everything else is built on sand; skip (4) and the loops in (5) produce theatre, not learning.

### 8.7 Design anti-patterns

The design practice is also defined by what it is not. The recurring anti-patterns, named so they can be spotted and stopped:

- **The charter doc graveyard.** Charter written once, filed, never referenced — worse than no charter because it teaches that agreements are fiction (team_lead §2.4). *Antidote: reference the charter in every retro; amend it when the team changes.*
- **The goal poster.** Team goals that are inspiring and unmeasurable ("be world-class"), or measurable and uninspiring (ticket counts). *Antidote: specific, team-owned, outcome-based goals reviewed on cadence (§8.2).*
- **The feedback theatre.** Retros that produce no owned actions, health checks whose results are never discussed, post-mortems that are compliance paperwork. *Antidote: every loop ends in an action and an owner (§8.3); cancel the ritual if it cannot meet that rule.*
- **The safety monologue.** A leader who talks about psychological safety but punishes its practice — the raised eyebrow, the "whose fault was that?", the colder 1:1 after a challenge. *Antidote: audit your own responses (§4.7); ask the team, anonymously, whether candour is actually safe.*
- **The metrics hammer.** Team metrics used as individual targets, or as sticks in the 1:1. *Antidote: metrics are team-level diagnostic signals reviewed in the retro, never individual performance evidence (§7.3).*
- **The forever-storming.** Design with no accountability mechanics, where conflict is encouraged but nothing is ever decided. *Antidote: pair the conflict norms with decision rules (who decides, when, via which process — team_lead §7).*

Each anti-pattern is a *violation of a design element* in §8.5 — which is the point: the design table is both the specification and the diagnostic. When a team is unhealthy, find which element is missing or faked, and rebuild it.

---

## 9. The Worked Example — A Cymbal Bank Squad Transformation

### 9.1 The scenario

**The familiar context.** The setting is a Cymbal Bank-style bank IT environment (cross-ref [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) for the systems landscape and the banking series for the domain context). The cast: **the Payments & Channels Integration Squad** in the Singapore IT hub — nine people (four developers, two QA, one DevOps/CI engineer, one business analyst, one squad lead), owning the integration layer between the bank's payment rails (Swift, FAST, GIRO) and the front-office/channel systems. Mission inherited from the programme: "deliver payment integrations." The squad has existed for two years on paper; in practice it is a **pseudo-team**: developers work individual ticket streams, QA is a shared pool, the "squad" is staffed per project, retros are skipped, incidents are blamed, and delivery has slipped three quarters running. The new squad lead — the reader of this guide — is asked to "fix it."

### 9.2 The starting state — diagnosis

Using the discipline as the diagnostic stack:

| Check | Finding | Verdict |
|---|---|---|
| K&S definition (§2) | Not a real team: no common purpose (only inherited tickets), individual accountability only, QA pooled externally | **Pseudo-team** — the worst state, worse than an honest working group |
| Tuckman (§6.1) | Stuck at forming/fake-norming: polite standups, no real agreements, conflict suppressed | Never stormed → never normed → never performs |
| Lencioni (§6.2) | Absence of trust (blame after the last incident), fear of conflict (no debate in planning), no accountability | Four of five dysfunctions active, bottom-up |
| Hackman conditions (§5) | Not a real team (membership churns); no compelling direction (ticket factory); unsupportive context (individual-only performance reviews, no deploy autonomy); no process coaching | Four of five conditions missing |
| Aristotle (§3) | Low psychological safety (silence in meetings, blame culture), low structure & clarity (no team goals), low impact (never see the outcome) | Five dynamics, almost all red |
| Engineering (§7) | Deployments gated by a central change board; release every 6–8 weeks; lead time months; failure rate unknown (incidents hidden); productivity measured in tickets closed | DORA poor; SPACE: activity-only measurement |

The diagnosis in one line: **a pseudo-team in an unsupportive context, with no safety, no goals, and no feedback loops.** Every element of the fix maps to a section of this guide.

### 9.3 The transformation design

The 90-day design follows the build order of §8.6 — container, charter, goals, safety, loops, accountability:

**Days 1–15 — Fix the container (Hackman condition 1; grow_team §7 cross-ref).** The squad lead negotiates: stable membership for 6 months (no mid-sprint re-staffing), QA embedded into the squad (a queue, not a pool — grow_team §7.2), a named mission owned by the squad: *"reduce payment integration breaks and deliver channel integrations end-to-end."* The squad gets its own backlog and its own deploy path for the integration services (with the change-board constraint acknowledged, not fought). *Design move: real team, bounded and interdependent.*

**Days 10–30 — Forge the charter (K&S specification; team_lead §2.4).** A two-session charter offsite: (1) purpose — the squad rewrites the mission in its own words and adds a success measure ("payment break-fix time under 90 minutes; two channel integrations shipped this half"); (2) roles and norms — ownership map, definition of done, working agreements ("disagreement happens in the room," "no blame in retros," "anyone can call a decision review," "deploy windows are respected"). The lead writes nothing for them; facilitates. *Design move: common purpose + common approach, team-shaped.*

**Days 20–40 — Set the goals (K&S performance goals; team_lead §5).** Quarterly goals derived from the mission, specific and measurable: (a) break-fix time 4h → 90m; (b) two integrations delivered to production; (c) deployment frequency doubled within the change-board constraints. Sprint goals each fortnight; the goals page is published to the programme office — visibility is the accountability precondition. *Design move: performance goals, team-level, visible.*

**Days 25–60 — Build the safety (Edmondson; §4).** The lead goes first: admits a past estimation failure of their own at the first retro; responds to the next incident with "what happened?" not "who did this?"; invites dissent explicitly in planning ("tell me what's wrong with this design — I'd rather hear it here than in a UAT surprise"); introduces blameless post-mortems with the system/person split; watches micro-reactions (no raised eyebrows at bad news). The first honest incident report — a payment-file mapping error that had been hidden for a month — is *celebrated* as a learning win. *Design move: safety before standards ratcheting (the §4.4 matrix — avoid the anxiety zone).*

**Days 30–60 — Wire the feedback loops (§8.3; §7).** Retros every fortnight with owned actions; a quarterly health check (anonymised) that includes a safety question; CI and test coverage tightened (the dev-tooling cluster practices — [../technology/shell_testing_frameworks_comparison_guide.md](../technology/shell_testing_frameworks_comparison_guide.md), [../technology/bashcov_vs_kcov_guide.md](../technology/bashcov_vs_kcov_guide.md)); DORA metrics started as a *team-relative trend* (deployment frequency, lead time, failure rate, restore time) — flagged honestly to the team that the bank's change controls cap absolute frequency, so the target is *improvement within constraints*, not public benchmarks; the SPACE conversation held once to agree that tickets closed is not the productivity measure — outcomes are. *Design move: a learning system with visible, owned actions.*

**Days 45–90 — Hold the accountability (§8.4).** Commitments published and referenced; peer challenge coached (the lead models it first, then stops being the only enforcer); team-level wins celebrated publicly (the first on-time integration release gets a programme-level mention); the lead protects the boundary — buffers the squad from the re-staffing requests and the ad-hoc ticket streams that destroyed the previous two years. *Design move: mutual accountability as a felt obligation, not a boss function.*

### 9.4 The transformation timeline

| Phase | Days | Design moves | What changed (observable) | Section |
|---|---|---|---|---|
| Container | 1–15 | Stable membership; QA embedded; named mission; own backlog | Team has boundaries and interdependence for the first time | §5.2, grow_team §7 |
| Charter | 10–30 | Purpose/roles/norms forged by the team | A written charter that people reference; decisions move to the room | §8.1, team_lead §2.4 |
| Goals | 20–40 | Team-level quarterly + sprint goals, published | Everyone can state the goal; planning has a target | §8.2, team_lead §5 |
| Safety | 25–60 | Leader modeling; blameless post-mortems; dissent invited | Incidents reported; juniors challenge seniors; retros are honest | §4 |
| Loops | 30–60 | Retros, health checks, DORA trends, CI tightening | Retro actions owned; metrics visible; quality culture rises | §8.3, §7 |
| Accountability | 45–90 | Published commitments; peer challenge; team scoreboard | Peers call each other; lead enforces less; delivery holds | §8.4 |

**The 90-day outcome (illustrative, honest):** break-fix time down from ~4h to ~2h and falling; the first integration shipped on time in 18 months; one incident reported openly that would previously have been hidden; the health check shows delivery and learning in green for the first time; two members, previously exit-risk, are visibly engaged. Not yet a high-performance team — the *deep* mutual commitment to each other's growth (the K&S HPT level) takes another two quarters of stable membership and compounding trust — but a real team that is now moving up the performance curve, with the discipline in place to keep going.

### 9.5 The lessons

1. **Pseudo-team first.** The single biggest unlock was making it a *real team* (container + charter) before any performance push. Everything else was built on that.
2. **Safety is the gating factor.** The metrics and the retros only started producing once the lead's responses changed. Blame culture was the reason the failure data was invisible, and invisible failure cannot be fixed (Edmondson's counter-intuitive finding, §4.1).
3. **Goals are the engine; the charter is the steering.** The goals gave the energy; the charter gave the norms that kept the energy from turning into conflict or chaos.
4. **Bank constraints are real — adapt, don't ignore.** DORA benchmarks are not directly portable into a change-board environment; the honest move is team-relative trends within constraints, plus a quality-culture target that the controls cannot touch.
5. **The lead's micro-behaviours are the programme.** Every "who did this?" undid a week of safety-building. The team calibrates to what the leader punishes, not what the leader preaches (team_lead §2.5).
6. **HPT is a compounding state, not a switch.** Ninety days buys a real team with momentum; the high-performance level — mutual commitment to each other's growth — needs stability and trust over quarters (Katzenbach & Smith's own framing; Tuckman's performing stage).

### 9.6 What would have failed — the counterfactual lessons

The transformation design is only credible if its failure modes are named. Each of these is a version of the transformation that would have failed:

- **Starting with goals instead of the container.** Pushing performance targets at a pseudo-team with churning membership and no charter would have produced the anxiety zone (§4.4): pressure without safety, blame without accountability. The build order of §8.6 is load-bearing, not cosmetic.
- **Fighting the change board head-on.** A frontal campaign to abolish central change approval would have consumed the lead's capital, lost, and taught the team that their new lead is naive. The design worked *within* the constraint and used the squad's own data (falling failure rate) as the eventual argument for a pilot exemption — slow, evidence-based, and bank-appropriate.
- **Writing the charter for the team.** A lead-authored mission statement would have been a directive in disguise — exactly the "purpose received, not shaped" failure the canon warns about (§2.2). The team had to write its own words, even at the cost of two messy sessions.
- **Postering the safety.** A "psychological safety" slide in the kickoff, followed by a blame-driven incident review in week three, would have done more damage than no safety work at all — the gap between stated values and observed responses is what teaches cynicism (§4.7).
- **Turning the metrics into individual targets.** Using DORA or SPACE to rank individuals would have recreated the activity-measurement trap (§7.3) and destroyed the mutual accountability the metrics were meant to serve. Metrics stayed at team level, framed as diagnostic signals, reviewed in the retro — never in the 1:1.
- **Skimping on anonymity in the health check.** A non-anonymous health check in a bank culture would have returned polite greens and hidden the real state; anonymity was non-negotiable (§8.3, cross-ref team_lead §2.6).

The meta-lesson: **every failure mode is a discipline violation.** The transformation worked because it followed the canon's order and rules; the counterfactuals are the canon's warnings made concrete.

---

## 10. The Summary — One Page

### 10.1 The one-page

| The HPT discipline | In one line |
|---|---|
| **Definition** (Katzenbach & Smith 1993, verified) | A small number of people with complementary skills, committed to a common purpose, performance goals, and approach, holding themselves mutually accountable |
| **The curve** | Working group → pseudo-team → potential team → real team → high-performance team; most "team problems" are groups or pseudo-teams |
| **The evidence** (Project Aristotle 2015, verified) | Five dynamics — psychological safety (#1), dependability, structure & clarity, meaning, impact; who is on the team matters less than how they interact |
| **The safety** (Edmondson 1999/2018, verified) | Shared belief the team is safe for interpersonal risk-taking; learning behavior mediates safety → performance; safety × high standards = the learning zone |
| **The conditions** (Hackman 2002, verified) | Real team, compelling direction, enabling structure, supportive context, expert coaching — design the conditions, don't command the process |
| **The dynamics** (verified) | Tuckman: stages (storming is necessary). Lencioni: dysfunctions (trust first). Belbin: roles (balance, not brilliance) |
| **The engineering** (DORA, SPACE — verified) | Four metrics: deployment frequency, lead time, change failure rate, restore time. Productivity: five SPACE dimensions — never activity alone |
| **The design** | Charter → team goals → feedback loops → mutual accountability, built in that order, on a stable real team |
| **The transformation** (§9) | Fix the container, forge the charter, set the goals, build the safety, wire the loops, hold the accountability — 90 days to a real team, quarters to HPT |

### 10.2 The final word: the team that wins

Every model in this guide converges on the same sentence. The team that wins is **small enough to know each other, skilled enough to cover each other, bound by a purpose they built and goals they own, safe enough to be wrong in front of each other, and accountable to each other rather than just to the boss** — and it is *designed* that way, deliberately, by a leader who sets the conditions and then gets out of the way. The discipline is not a personality cult and not a poster on the wall; it is a specification, an evidence ranking, and a build order. High performance is not a talent you discover in a team. It is a state you engineer — and then protect, because the organisation's default gravity will always pull a real team back toward a working group. The team that wins is the team whose lead understood that.

---

## 11. Glossary

| Term | Definition |
|---|---|
| **High-performance team (HPT)** | A real team whose members are deeply committed to a common purpose, performance goals, and one another's growth and success, delivering results that outstrip reasonable expectations (Katzenbach & Smith's top of the team performance curve) |
| **HPT (abbreviation)** | High-performance team; also used in this guide for the *discipline* — the research canon and design practice for building such teams |
| **Katzenbach** | Jon R. Katzenbach, McKinsey partner and co-author of *The Wisdom of Teams* (1993) with Douglas K. Smith |
| **Smith** | Douglas K. Smith, McKinsey partner and co-author of *The Wisdom of Teams* (1993) |
| **Wisdom of Teams** | *The Wisdom of Teams: Creating the High-Performance Organization* (HBS Press, 1993) — the foundational HPT text: the team definition, the team performance curve, the team discipline |
| **Psychological safety** | "A shared belief held by members of a team that the team is safe for interpersonal risk taking" (Edmondson 1999) — admitting mistakes, asking questions, challenging the boss, proposing half-formed ideas |
| **Edmondson** | Amy C. Edmondson, Harvard Business School professor; introduced team psychological safety (1999 paper), wrote *The Fearless Organization* (2018) |
| **Fearless Organization** | *The Fearless Organization: Creating Psychological Safety in the Workplace for Learning, Innovation, and Growth* (Wiley, 2018) — Edmondson's practical book on building safety alongside high standards |
| **Project Aristotle** | Google People Analytics research program (2012–2015) studying 180+ teams to answer "what makes a team effective?" |
| **Google** | The company whose re:Work research (Project Oxygen for managers, Project Aristotle for teams) provided the large-N evidence base |
| **Five dynamics** | The five attributes Aristotle found to predict team effectiveness: psychological safety, dependability, structure & clarity, meaning, impact — with psychological safety the #1 predictor |
| **Hackman** | J. Richard Hackman, Harvard psychologist, author of *Leading Teams* (2002); defined the conditions for team effectiveness and the three effectiveness criteria |
| **Conditions** | Hackman's five designable conditions: a real team, a compelling direction, an enabling structure, a supportive organizational context, expert coaching |
| **Tuckman** | Bruce Tuckman, who proposed forming–storming–norming–performing (1965, *Psychological Bulletin*) and, with Mary Ann Jensen, added adjourning (1977) |
| **Forming** | First Tuckman stage: polite, uncertain, dependent on the leader; roles unclear |
| **Storming** | Second stage: conflict over roles, approach, leadership; necessary friction — suppress it and the team never truly performs |
| **Norming** | Third stage: agreements form, cohesion and trust build, the team codifies how it works |
| **Performing** | Fourth stage: high autonomy, self-management, steady delivery; the leader protects the boundary |
| **Lencioni** | Patrick Lencioni, author of *The Five Dysfunctions of a Team* (2002) — the dysfunction pyramid: absence of trust, fear of conflict, lack of commitment, avoidance of accountability, inattention to results |
| **Five dysfunctions** | Lencioni's pyramid of failure modes, bottom-up: absence of trust → fear of conflict → lack of commitment → avoidance of accountability → inattention to results |
| **Belbin** | R. Meredith Belbin, author of *Management Teams: Why They Succeed or Fail* (1981); derived nine team roles from Henley Management College research |
| **Roles** | Belbin's nine behavioural team roles (Plant, Resource Investigator, Co-ordinator, Shaper, Monitor Evaluator, Teamworker, Implementer, Completer Finisher, Specialist) — balance, not brilliance, predicts team success |
| **DORA** | DevOps Research and Assessment — the research program behind the Accelerate State of DevOps reports and the four key delivery metrics |
| **Four metrics** | Deployment frequency, lead time for changes, change failure rate, and time to restore service (MTTR); reliability was added as a fifth metric in later reports |
| **SPACE** | The 2021 developer-productivity framework (Forsgren, Storey et al., *ACM Queue*): Satisfaction, Performance, Activity, Communication & Collaboration, Efficiency & Flow |
| **Charter** | The team's constitution: purpose, roles, norms/working agreements, and process anchors, created by the team with the lead |
| **Feedback loop** | A closed cycle of action → observation → learning → change; at team level: delivery metrics, retros, health checks, post-mortems — every loop must end in an owned action |
| **Accountability** | Mutual accountability: peers holding peers (and themselves) to the team's commitments and standards — the fifth component of the Katzenbach & Smith definition |
| **Squad** | A small (typically 6–10 person), cross-functional, autonomous team owning a mission end-to-end — the Spotify model's unit, the practical home of the HPT discipline |
| **Team topology** | The organisational structure through which work flows (squads, tribes, chapters, guilds; feature vs component teams) — see [grow_team_guide.md](grow_team_guide.md) §7 |

---

## Appendix: Verification Notes

Every canonical fact in this guide was checked against primary or reputable secondary sources during research (August 2026). The ledger:

- **Katzenbach & Smith, *The Wisdom of Teams*, 1993 — verified.** Publisher (Harvard Business School Press, 1993; 2015 reissue), authorship (McKinsey partners), method (50 teams, 30+ companies), the five-component definition, the six factors, and the performance curve confirmed via multiple independent summaries (Google Books, Warwick Business School, highperformanceteams.org).
- **Tuckman 1965 / 1977 — verified** (cross-ref team_lead_methodologies_guide.md §2.1, which carries the full origin). *Psychological Bulletin* 1965; adjourning with Jensen 1977.
- **Lencioni 2002 — verified** (cross-ref team_lead §2.3). Jossey-Bass 2002.
- **Belbin 1981 — verified.** *Management Teams: Why They Succeed or Fail* (Heinemann, 1981); Henley Management College research; eight original roles with two later renames and the ninth role (Specialist) added later. Flagged honestly: independent psychometric assessments of the inventory are mixed, and Belbin himself positions it as a practice tool.
- **Edmondson 1999 — verified.** *Administrative Science Quarterly* 44(2), June 1999, pp. 350–383; 51 work teams in a manufacturing company; the "shared belief…safe for interpersonal risk taking" definition; learning behavior mediates safety → performance; team efficacy not significant when safety is controlled. Verified against the paper's own abstract (MIT-hosted JSTOR copy) and SAGE/APA records.
- **Edmondson, *The Fearless Organization*, 2018 — verified.** Wiley, 2018, ISBN 978-1-119-47724-2; subtitle "Creating Psychological Safety in the Workplace for Learning, Innovation, and Growth"; the three levers (frame, invite, respond) and the safety × standards matrix are Edmondson's published framing.
- **Project Aristotle — verified.** Google People Analytics, launched 2012, 180+ teams, 250+ attributes; five dynamics and psychological-safety-first confirmed via Google's re:Work "Understand team effectiveness" guide (published 2015) and multiple secondary sources; popularised via the NYT Magazine piece (Feb 2016).
- **Hackman, *Leading Teams*, 2002 — verified.** Harvard Business School Press, 2002; the five conditions confirmed via HBS Press and academic summaries.
- **DORA four metrics — verified.** Deployment frequency, lead time for changes, change failure rate, time to restore service; codified in the Accelerate State of DevOps reports (2018 report) and the *Accelerate* book (IT Revolution, 2018). **Flagged:** exact elite/low benchmark numbers vary by report year and were not individually re-verified here; the later addition of *reliability* as a fifth metric is flagged in §7.1. **Flagged:** specific DORA *adoption* statistics could not be verified in the research budget and are not stated in this guide.
- **SPACE framework — verified.** Forsgren, Storey, Maddila, Zimmermann, Houck, Butler, "The SPACE of Developer Productivity," *ACM Queue* 19(1), February 2021, pp. 20–48; the five dimensions confirmed via the Microsoft Research publication record.
- **Worked example** — a synthesis by the author (illustrative scenario), not a case study of a real engagement; outcomes are illustrative and flagged as such in §9.4.

### Key sources consulted

Primary and authoritative secondary sources used in the verification pass (accessed August 2026):

- Katzenbach, J. R. & Smith, D. K. (1993). *The Wisdom of Teams: Creating the High-Performance Organization*. Harvard Business School Press. (Publisher record, Google Books; theory summary via Warwick Business School.)
- Tuckman, B. W. (1965). "Developmental Sequence in Small Groups." *Psychological Bulletin* 63(6): 384–399. (Cross-verified via team_lead_methodologies_guide.md §2.1.)
- Belbin, R. M. (1981). *Management Teams: Why They Succeed or Fail*. Heinemann. (Role details and history via the Belbin Team Inventory record, Wikipedia, with its academic-critique caveats.)
- Lencioni, P. (2002). *The Five Dysfunctions of a Team*. Jossey-Bass. (Cross-verified via team_lead_methodologies_guide.md §2.3.)
- Edmondson, A. (1999). "Psychological Safety and Learning Behavior in Work Teams." *Administrative Science Quarterly* 44(2): 350–383. (Paper abstract, MIT-hosted JSTOR copy.)
- Edmondson, A. (2018). *The Fearless Organization: Creating Psychological Safety in the Workplace for Learning, Innovation, and Growth*. Wiley. (Publisher record.)
- Google re:Work. "Understand team effectiveness." Project Aristotle findings, published 2015. (rework.withgoogle.com; NYT Magazine, Feb 2016.)
- Hackman, J. R. (2002). *Leading Teams: Setting the Stage for Great Performances*. Harvard Business School Press. (HBS Press and publisher records.)
- Forsgren, N., Humble, J., Kim, G. (2018). *Accelerate: The Science of Lean Software and DevOps*. IT Revolution Press; DORA, *Accelerate State of DevOps* reports (2018+); Google Cloud "Four Keys" documentation.
- Forsgren, N., Storey, M.-A., Maddila, C., Zimmermann, T., Houck, B., Butler, J. (2021). "The SPACE of Developer Productivity." *ACM Queue* 19(1): 20–48. (Microsoft Research publication record.)

*End of guide. Related: [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §2 (frameworks) · [grow_team_guide.md](grow_team_guide.md) (lifecycle + §7 topology) · [organizational_behavior_guide.md](organizational_behavior_guide.md) (OB discipline).*
