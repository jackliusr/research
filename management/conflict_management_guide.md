# Conflict Management: Turning Friction into the Working Peace — A Solution Architect's Guide

> **Author:** Jack Liu Shurui, Solution Architect — Cymbal Bank, Singapore
> **Context:** the conflict-management deep-dive of the management series — how a solution architect diagnoses, de-escalates, and resolves the conflicts that architecture work generates: requirements clashes, design disagreements, priority disputes, cross-functional friction, and vendor stand-offs. It is the dispute layer that sits on top of the people skills: it cross-references [Communication & Stakeholder Management Skills](../management/communication_stakeholder_management_skills_guide.md) (the stakeholder and influencing layer every difficult conversation runs on), [Facilitation Skills](../management/facilitation_skills_guide.md) (listening, questioning, and the mediator-vs-facilitator hat discipline), [Workshop Methodology](../management/workshop_methodology_guide.md) (the process mechanics for decision sessions), [Vendor Management](../management/vendor_management_guide.md) (the vendor-side escalation context), and the repository's decision-capture anchor [Architecture Decision Records](../technology/architecture_decision_record_guide.md) (the convention this guide uses for recording how a conflict was settled — it does *not* re-derive ADR practice).
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass had **live web access** (web_search + web_extract). Verified at primary or authoritative sources: the co-author's own company site kilmanndiagnostics.com and the Wikipedia record for the Thomas–Kilmann Conflict Mode Instrument (Kenneth W. Thomas & Ralph H. Kilmann, introduced 1974, Xicom, Tuxedo NY ✅; the five modes ✅; the assertiveness/cooperativeness axes ✅; the Blake & Mouton managerial-grid lineage ✅; 30 statement pairs ✅; "more than 10 million copies since 1974" ✅) and the APA PsycNet record (DOI 10.1037/t02326-000) ✅; the Wikipedia record of *Getting to Yes* (Fisher & Ury, 1981, Houghton Mifflin, Harvard Negotiation Project ✅; the four principles ✅; BATNA ✅; 1991/2011 editions adding Bruce Patton ✅); the PMI Global Leadership Community document reproducing Mind Tools' *Interest-Based Relational* article (explicit attribution of IBR to Fisher & Ury's 1981 *Getting to Yes*, and the six IBR steps ✅); the Internet Archive catalog record of Stone, Patton & Heen's *Difficult Conversations: How to Discuss What Matters Most* (Viking, 1999 ✅, with the Penguin 2000 paperback noted via the Sheila Heen Wikipedia record ✅); the Wikipedia records for intragroup conflict and Morton Deutsch (the APA Dictionary's task/relationship/process taxonomy ✅; Jehn 1995, *Administrative Science Quarterly* 40(2):256–282, DOI 10.2307/2393638 ✅; de Wit, Greer & Jehn 2012 meta-analysis, *Journal of Applied Psychology* 97(2):360–390, DOI 10.1037/a0024844 ✅; Deutsch's *The Resolution of Conflict: Constructive and Destructive Processes*, Yale University Press, 1973 ✅); the Internet Archive record of Aaron Lazare's *On Apology* (Oxford University Press, 2004 ✅); and the Wikipedia records for the managerial grid model (1964 ✅) and Nonviolent Communication (Rosenberg; four components ✅). Anything not verifiable is flagged ⚠ in [The Claims Audit](#12-the-claims-audit) and [What Could Not Be Verified](#13-what-could-not-be-verified). Nothing in this guide is fabricated; where the brief demanded verification and the tools could not deliver it, the guide says so plainly.
> **Last Updated:** August 2026

---

### How to Use This Guide

**What this guide is.** The conflict-management deep-dive for architects: the models (Thomas–Kilmann, Interest-Based Relational, Harvard principled negotiation, the difficult-conversations framework), the skills that make them work (listening, assertiveness, emotional regulation, reframing, apology, mediation), and the application to the specific conflicts architecture work generates — requirements conflicts, design disagreements, priority disputes, cross-functional and vendor stand-offs. The worked example in §11 is a single Cymbal Bank architecture-team conflict, end to end: diagnosis, mode selection, the conversation, the resolution, the follow-up.

**How to use it.** Read §1 first — the landscape and the taxonomy every later section assumes. §2–§5 are the four resolution models (the intellectual core). §6–§8 are the skills. §9–§10 are the architecture-work application, with the escalation paths. §11 is the worked example; §12 is the claims audit; §13 the honest gaps; §14 the glossary.

**What this guide is not.** It is not a communication or stakeholder guide (that lives in `communication_stakeholder_management_skills_guide.md`), not a facilitation-craft manual (that lives in `facilitation_skills_guide.md` — listening and mediation mechanics are cross-referenced, not re-derived), not a workshop-process manual (that lives in `workshop_methodology_guide.md`), not a vendor-management manual (that lives in `vendor_management_guide.md`), and not an ADR manual (that lives in `../technology/architecture_decision_record_guide.md` — this guide cross-references its conventions for capturing how a conflict was resolved).

**The one-sentence thesis:** *conflict is the friction that either grinds a team down or polishes its decisions — and the difference is not the conflict itself but the craft applied to it, which is learnable, modelable, and runnable like any other engineering discipline.*

---

## Table of Contents

1. [The Conflict Landscape: Types, Sources, and the Constructive–Destructive Distinction](#1-the-conflict-landscape-types-sources-and-the-constructivedestructive-distinction)
2. [The Thomas–Kilmann Model: Five Modes, Two Axes](#2-the-thomaskilmann-model-five-modes-two-axes)
3. [The Interest-Based Relational (IBR) Approach](#3-the-interest-based-relational-ibr-approach)
4. [Harvard Principled Negotiation: Fisher & Ury's *Getting to Yes*](#4-harvard-principled-negotiation-fisher--urys-getting-to-yes)
5. [The Difficult Conversations Framework: Stone, Patton & Heen](#5-the-difficult-conversations-framework-stone-patton--heen)
6. [The Skills Toolkit I: Active Listening and Assertive Communication](#6-the-skills-toolkit-i-active-listening-and-assertive-communication)
7. [The Skills Toolkit II: Emotional Regulation, Reframing, Apology and Repair](#7-the-skills-toolkit-ii-emotional-regulation-reframing-apology-and-repair)
8. [Mediation Basics and the Facilitator's Hat](#8-mediation-basics-and-the-facilitators-hat)
9. [Conflict in Architecture Work: Requirements, Design, and Priorities](#9-conflict-in-architecture-work-requirements-design-and-priorities)
10. [Cross-Functional Conflict, Vendor Conflict, and Escalation Paths](#10-cross-functional-conflict-vendor-conflict-and-escalation-paths)
11. [The Worked Example: The Cymbal Bank Payment-Hub Conflict](#11-the-worked-example-the-cymbal-bank-payment-hub-conflict)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [The Glossary](#14-the-glossary)

---

## 1. The Conflict Landscape: Types, Sources, and the Constructive–Destructive Distinction

### 1.1 What conflict actually is

A useful working definition, widely quoted in practitioner literature: interpersonal conflict is *"an expressed struggle between at least two interdependent parties who perceive incompatible goals, scarce resources, and interference from the other party in achieving their goals."* ⚠ The definition is commonly attributed to Hocker & Wilmot's *Interpersonal Conflict* textbook, but the attribution was not re-verified at a primary source this pass (see audit #19). Unpack it, because every element is load-bearing for the architect:

1. **Expressed** — a conflict that is never voiced is a tension, not a conflict; the moment it is expressed, it becomes addressable.
2. **Interdependent** — the parties need each other; in architecture work this is structural: an architect and a platform team, a product owner and an engineering lead, a bank and its core-banking vendor cannot simply walk away.
3. **Perceived incompatible goals** — the perception matters as much as the reality; two goals may be compatible in fact and still conflict in belief.
4. **Scarce resources** — budget, headcount, environment capacity, timeline; scarcity is the fuel of most workplace conflict.
5. **Interference** — each side believes the other is blocking its goals; the belief, true or not, is what turns disagreement into dispute.

Conflict is not the same as disagreement. A disagreement is a difference of view; conflict is that difference *expressed under conditions of interdependence, scarcity, and perceived interference*. This is why architecture teams — deeply interdependent, chronically short of resources, and full of strong opinions — generate conflict as a by-product of doing their job.

### 1.2 The intragroup taxonomy: task, relationship, and process conflict

The standard academic taxonomy of conflict *within* a group or team distinguishes three types. Verified this pass: the American Psychological Association's *Dictionary of Psychology* defines intragroup conflict as covering three distinct problems — **process, relationship, and task** — and the empirical literature is anchored in **Karen A. Jehn's** research programme, which developed the Intragroup Conflict Scale (ICS) to measure them ✅.

| Type | What it is | Architecture example | Typical effect |
|---|---|---|---|
| **Task conflict** | Disagreement about *what* the work is: goals, content, substance, the right answer | "The event backbone should be Kafka"; "No — the bank's volume doesn't justify the operational cost" | Potentially valuable: tested decisions, better criteria (see §1.5) |
| **Relationship conflict** | Disagreement that has become *personal*: personality clashes, mistrust, animosity | "She always overrules my designs" — said about the person, not the design | Consistently corrosive: the meta-analysis says this is the type that hurts performance ✅ |
| **Process conflict** | Disagreement about *how* the work gets done: roles, process, ownership, sequencing | "Who owns the API contract — the platform team or the domain team?" | Mixed: valuable when it fixes broken ownership, corrosive when it becomes turf war |

The canonical primary citation is **Jehn (1995)**, "A Multimethod Examination of the Benefits and Detriments of Intragroup Conflict", *Administrative Science Quarterly* 40(2): 256–282, DOI 10.2307/2393638 ✅. The follow-up field study, Jehn, Northcraft & Neale (1999), "Why Differences Make a Difference", *Administrative Science Quarterly* 44(4): 741–763 ✅, showed the same taxonomy operating in real workgroups.

Why the taxonomy matters more than it looks: **the same expressed disagreement can be task conflict or relationship conflict depending on how it is handled.** The sentence "the event backbone should be Kafka" is task content; the sentence "you always pick the shiny tool and leave us to operate it" is relationship content wearing task clothes. A huge share of conflict-management craft is keeping conflict in the task channel and out of the relationship channel — the facilitation guide's §6.3 makes exactly this move, and it is cross-referenced rather than re-derived here.

### 1.3 The sources of workplace conflict

Two complementary answers, one verified and one practitioner-common.

**The verified antecedents.** The research literature identifies a set of conditions under which intragroup conflict is more likely to arise ✅ (Wikipedia's intragroup-conflict article, citing the primary studies): low task or goal uncertainty (ambiguous mandates invite competing interpretations); increased group size (more parties, more interfaces); increased diversity of gender, age, race and background (more perspectives — and more friction); lack of information sharing (each side reasons from its own slice); and **high task interdependence** (the more your work depends on mine, the more our differences collide). Every one of these maps onto architecture work: ambiguous mandates are the default state of early requirements; architecture teams are interdependent by definition; information sharing across domain teams in a bank is chronically poor.

**The practitioner fourfold.** A common practitioner taxonomy names four recurring sources: **priorities** (which goal wins when two are both legitimate), **resources** (who gets the budget, the headcount, the environments), **styles** (how people work and communicate — the TKI modes of §2 are one formalisation), and **values** (what people believe is right, fair, or safe — the deepest kind, because it does not compromise). ⚠ This fourfold is standard training-room lore and is presented here as such, not as a verified research finding (audit #18). It is compatible with — but not identical to — the verified antecedent list above; treat the fourfold as a memory aid and the antecedent list as the evidence.

One more source deserves naming because it is the architect's special curse: **ambiguity of authority**. Two senior people, two legitimate mandates, one decision. Banks institutionalise this with matrix structures (business vs technology vs risk), and architecture reviews sit precisely at the intersection. When the source is ambiguous authority, no amount of listening resolves it — someone must decide, and the decision must be recorded (see §9.4 and the ADR conventions).

### 1.4 Constructive vs destructive conflict: the Deutsch distinction

The foundational statement of the distinction is **Morton Deutsch's** *The Resolution of Conflict: Constructive and Destructive Processes* (New Haven: Yale University Press, 1973) ✅ — Deutsch, a founding father of the conflict-resolution field (PhD under Kurt Lewin at MIT, 1948; theory of cooperation and competition first presented 1949; founder of Columbia's International Center for Cooperation and Conflict Resolution in 1986) ✅, framed the question that still organises the field: what makes the *same* conflict constructive in one group and destructive in another?

Deutsch's core insight: **cooperative processes and competitive processes are self-fulfilling.** A group that treats a conflict as a joint problem to solve (cooperative orientation) tends to generate communication, trust, and integrative solutions; a group that treats it as a contest to win (competitive orientation) tends to generate misperception, rigidity, and escalation — each dynamic reinforcing itself. His "Crude Law" formulation: *the characteristic processes and effects of a given type of relationship tend to induce that type of relationship* ✅. Conflict is not inherently good or bad; the *process* the parties adopt decides which spiral they enter.

The practical translation for architects: **you cannot choose whether conflicts occur, but you can choose — early, and repeatedly — which spiral you are feeding.** The destructive spiral's warning signs: positions harden, communication narrows, issues multiply (one disagreement becomes three), the other side is caricatured, third parties are recruited, and the original issue is forgotten. The constructive spiral's signs: the problem is separated from the person, information flows both ways, the issue list *shrinks*, options multiply, and the relationship survives the disagreement.

### 1.5 What makes conflict productive — and what the evidence says

The empirical picture is more nuanced than the cheerleading. The definitive quantitative statement is the **de Wit, Greer & Jehn (2012)** meta-analysis, "The Paradox of Intragroup Conflict", *Journal of Applied Psychology* 97(2): 360–390, DOI 10.1037/a0024844 ✅:

- **Task conflict** has a *weakly negative to neutral* average relationship with team performance — not the strong positive effect that "good conflict" folklore claims — but it turns positive in specific conditions: when it occurs in management/decision-making groups, and when it is paired with low relationship conflict ✅.
- **Relationship conflict** is consistently and substantially negative ✅.
- **Process conflict** is also generally negative, and it is the least studied of the three ✅.

So the honest, evidence-grounded version of "make conflict productive" is narrower than the slogan: **productive conflict is task conflict, kept in the task channel, in a group whose relationships are healthy enough to absorb it, resolved by criteria rather than by power.** The conditions that make task conflict pay off, assembled from the verified sources:

1. **The topic is substantive** — a real design or requirement question with consequences.
2. **The people are separated from the problem** — Fisher & Ury's first principle (§4), which is exactly what prevents task conflict from spilling into relationship conflict.
3. **Criteria precede positions** — the group agrees on *what would count as a good answer* before anyone defends an answer (the facilitation guide's §9.2 decision-session signature questions; the workshop guide's converge techniques).
4. **The disagreement is settled, not smothered** — the decision is made, the dissent is recorded, and the reasoning is captured in an ADR (§9.4) so the conflict produces an artifact rather than a scar.
5. **The relationship survives** — which is the Deutsch condition: the process was cooperative even where the content was contested.

### 1.6 The stakes, and the escalation dynamic

Why a solution architect should care about conflict management at all — three costs, in increasing order of severity:

1. **The decision cost.** Unmanaged conflict delays or distorts decisions: the ADR never gets written, or gets written by the loudest voice in the room, or gets written twice (once per faction) and the architecture forks. In the worked example (§11), two cycles of the programme were lost to an unresolved pattern debate — the decision cost, measured in schedule.
2. **The team cost.** Relationship conflict's documented harm to performance (de Wit, Greer & Jehn 2012, §1.5 ✅) is the mechanism; the human cost is disengagement and attrition — the PMI/Mind Tools IBR article names it directly: "talent may be wasted as people disengage from their work and leave" ✅. An architect who loses a design fight *and* the relationship is an architect who is interviewing elsewhere within a year.
3. **The organisational cost.** In a bank, conflict that leaks upward consumes governance bandwidth: steering committees adjudicating architecture disputes instead of steering the portfolio, risk forums re-litigating settled designs, and — worst — the conflict becoming visible to audit and supervision as a governance weakness rather than a technical debate.

The escalation dynamic worth internalising: **disagreement → position → dispute → faction → us-vs-them**. Each step narrows the available instruments — a disagreement can be settled by a conversation; a faction fight can only be ended by a decision with teeth. Deutsch's constructive/destructive spiral (§1.4) is the verified theory of this dynamic: competitive process generates the misperception and rigidity that justifies more competition ✅. The practical law that follows is the one SkillsYouNeed states plainly and this pass verified: "dealing with conflict early is usually easier, because positions are not so entrenched, others are less likely to have started to take sides, and the negative emotions are not so extreme" ✅. The takeaway for the architect: **treat conflict as an incident to be triaged early, not a condition to be endured** — the same triage discipline applied to a production incident applies to a relationship incident, and the first response is the cheapest response that can still work.

---

## 2. The Thomas–Kilmann Model: Five Modes, Two Axes

### 2.1 Origin and lineage

The **Thomas–Kilmann Conflict Mode Instrument (TKI)** is the most widely used conflict-style inventory in the world ✅. Verified at the co-author's own company site (kilmanndiagnostics.com): "Dr. Ralph H. Kilmann co-created the TKI assessment in 1974", and the instrument "has sold more than 10 million copies since 1974" ✅. The Wikipedia record, citing the instrument itself, confirms: **"In 1974, Kenneth W. Thomas and Ralph H. Kilmann introduced their Thomas–Kilmann Conflict Mode Instrument (Tuxedo NY: Xicom, 1974)"** ✅. The APA PsycNet test record (DOI 10.1037/t02326-000) confirms the five-category scheme and that the statement pairs were matched for social desirability ✅.

The intellectual lineage matters: the TKI is a descendant of the **managerial grid** of **Robert R. Blake and Jane Mouton** (model 1964; *The Managerial Grid: The Key to Leadership Excellence*, Houston: Gulf Publishing, 1964) ✅, which mapped leadership styles on two axes — concern for people and concern for production. Thomas and Kilmann took the two-axis structure and re-labelled it for conflict behaviour: **assertiveness** (the degree to which you try to satisfy your own concerns) and **cooperativeness** (the degree to which you try to satisfy the other's concerns) ✅.

### 2.2 The five modes

| Mode | Assertiveness | Cooperativeness | One-line definition |
|---|---|---|---|
| **Competing** | High | Low | Pursuing your own concerns at the other's expense |
| **Collaborating** | High | High | Working with the other to find a solution fully satisfying both |
| **Compromising** | Medium | Medium | Splitting the difference; partially satisfying both |
| **Avoiding** | Low | Low | Not pursuing either your own or the other's concerns |
| **Accommodating** | Low | High | Neglecting your own concerns to satisfy the other's |

The instrument itself is a **forced-choice questionnaire of thirty pairs of statements**; for each pair you pick the A or B item that best describes your behaviour ✅. Because every pair is matched for social desirability, there is no "nicer-sounding" option to game ✅ — the score reveals your *relative* tendency across the five modes, not a personality verdict.

### 2.3 What each mode is for, and when to use it

The TKI's own framing — confirmed by the publisher's materials and the co-author's site — is that **no mode is inherently good or bad; each is appropriate in some situations** ✅. The "when to use" guidance below is the standard interpretive reading of the model (⚠ practitioner-standard; the specific situational lists are consistent with publisher materials but were not re-verified page-by-page this pass — audit #15):

**Competing** — for the urgent, the vital, and the unpopular-but-necessary. Use when: a decision must be made fast (an incident, a regulatory deadline); the issue is vital to the organisation and you are right (or at least the accountable one); an unpopular course of action must be enforced (a security control that slows delivery). The cost: it creates losers, and the Wikipedia record's own summary notes it "can strain relationships and trust if overused" ✅. The architect's discipline: *compete on the issue, never on the person* — and compete only when you can carry the consequences of winning.

**Collaborating** — for the high-stakes and the integrative. Use when: both sets of concerns are too important to compromise; you need to merge different perspectives (exactly the architect's job in design); the relationship matters as much as the outcome; time and trust are available. It is the most expensive mode — it is not the default, it is the investment.

**Compromising** — for the time-boxed and the roughly-equal. Use when: goals are moderately important but not worth the full collaborative cost; parties of equal power are committed to mutually exclusive goals; the deadline makes collaboration impossible; a temporary settlement is acceptable. The classic architecture example: "we'll standardise on the gateway for this year and revisit the mesh at the next planning cycle" — a settlement, recorded with its revisit date.

**Avoiding** — for the trivial, the cooling-off, and the outranked. Use when: the issue is trivial or symptomatic of something else; you need time to cool down (emotional regulation, §7.1); more important issues crowd it out; you have no power and no chance; the conflict will resolve itself. The discipline: *avoiding is a choice, not a default* — chronic avoiding is how small task conflicts curdle into relationship conflicts (Deutsch's destructive spiral, §1.4).

**Accommodating** — for the relationship-preserving and the one-sided. Use when: you are wrong (or the other side is right); the issue matters more to the other side than to you; preserving harmony matters more than the point; you are building goodwill credit for a later, bigger battle. The warning: chronic accommodating is how architects become order-takers — the bank's "yes" architect whose design opinions are never heard.

The architecture-situation translation, as a quick-reference table (⚠ interpretive, per audit #15):

| Architecture situation | Mode that fits | Why |
|---|---|---|
| Production incident; regulatory deadline; security vulnerability | **Competing** | Speed and certainty beat consensus; the accountable architect decides |
| Choosing the platform/pattern for a multi-year programme | **Collaborating** | Both sets of concerns are legitimate; the decision must survive contact with both teams |
| Two equal workstreams fighting over the same release slot | **Compromising** | Split the slot, record the revisit date; a settlement beats a war |
| A debate the architect cannot win and should not join (outranked, outvoted, out-of-scope) | **Avoiding** | Conserve energy for the fights that matter; say explicitly why you are stepping back |
| A stakeholder's requirement that matters more to them than to the programme | **Accommodating** | Bank the goodwill; the credit pays for a bigger battle later |

The table's lesson is the model's own: **the situation picks the mode; the default picks the failure.**

### 2.4 Using the TKI as an architect

Three practical uses, none of which is "give everyone the test and declare their personality":

1. **Self-awareness of your default.** Most people have one or two home modes (often competing or avoiding, the two extremes). Your default is not your only option — the model's whole point is that the *situation* should select the mode, not the habit.
2. **Reading the room's mix.** A team whose members all default to competing will fight over everything; one where everyone defaults to avoiding will decide nothing. The architect who sees the mix can design the process to compensate — the workshop guide's written-first rounds, private votes, and explicit decision rules are precisely such compensations.
3. **Mode selection as a conscious act.** Before a difficult conversation, ask: *which mode does this situation call for, and which mode is my default trying to drag me into?* The gap between the two is where the craft lives. §11's worked example runs this explicitly.

### 2.5 Limits and critiques

Honest caveats, in the house style: the TKI measures *self-reported behavioural tendency in conflict situations*, not personality, not skill, and not outcome. It tells you what mode you reach for, not whether you execute it well (you can compete badly, collaborate badly, or avoid skilfully). The Wikipedia record notes the instrument's psychometric properties — moderate test-retest repeatability and internal consistency in a 1978 analysis of 86 responses — and a long-running practitioner criticism that repeated exposure lets respondents game the answers ✅. And the situational guidance of §2.3 is exactly that: guidance. The single most important correction to popular TKI lore: **collaborating is not the "best" mode** — it is the most expensive one, appropriate when the stakes justify it, and the model's designers have always said so.

---

## 3. The Interest-Based Relational (IBR) Approach

### 3.1 What it is, and where it comes from

The **Interest-Based Relational (IBR) approach** is the practitioner workhorse of workplace conflict resolution: a structured way for a manager or lead to resolve a dispute between (or with) colleagues by keeping the relationship intact while solving the underlying problem. Its intellectual parentage is unambiguous: the PMI Global Leadership Community's conflict-resolution document — reproducing Mind Tools' article on the approach — states it plainly: **"Roger Fisher and William Ury developed the IBR approach and published it in their 1981 book, *Getting to Yes*"** ✅. Practitioner sources consistently attribute the approach to the Fisher–Ury / Harvard Negotiation Project lineage, which is consistent with *Getting to Yes*'s first principle (separate the people from the problem) ✅. ⚠ The honest caveat: the *label* "Interest-Based Relational approach" as a named method appears in practitioner and training literature (PMI, Mind Tools, university conflict-resolution courses) rather than as a named chapter of *Getting to Yes* itself; the exact coinage of the label could not be pinned to a single primary source this pass (audit #7).

IBR's core claim: when conflict arises, people entrench in positions and the temperature rises; the manager's job is to keep the conversation **courteous, consensual, and focused on issues rather than individuals** ✅ — resolving the conflict "in a civil and grown-up way" ✅. The approach presupposes that the people involved can listen actively, read body language, and manage their own emotions ✅ — which is precisely why §6–§8 of this guide exist.

### 3.2 The six steps

Verified against the PMI/Mind Tools text ✅:

1. **Make sure that good relationships are a priority.** Treat the other person with respect; be courteous; discuss constructively. The relationship is the asset the whole exercise protects.
2. **Separate people from problems.** The other person is usually not "being difficult" — real and valid differences lie behind conflicting positions. Attack the problem, not the person.
3. **Listen carefully to different interests.** You understand a position only when you understand *why* it was adopted — the interest underneath it (this is Fisher & Ury's "focus on interests, not positions", §4.2).
4. **Listen first, talk second.** Hear the other side fully before defending your own position — they might say something that changes your mind.
5. **Set out the "facts".** Agree jointly on the observable facts that bear on the decision. Different needs cause people to perceive the same facts differently; the group must agree on what the problem *is* before it can solve it.
6. **Explore options together.** Be open to a third position that neither side entered with — a jointly reached solution that satisfies both sets of interests.

### 3.3 When IBR fits — and when it does not

The source is explicit that IBR is not universal ✅: "you may not be able to resolve differences in such a consensual, collaborative way if your organization is in a crisis. On these occasions, you may have to 'pull rank' as a leader and make quick decisions about disputes and conflicts" ✅. The architect's translation: IBR is the default for *design-time* conflict — the disagreement has a deadline measured in weeks, both parties will keep working together, and a jointly owned answer is worth more than a faster imposed one. It is the wrong instrument when the house is on fire: a production incident, a regulatory deadline, or a security vulnerability does not get a facilitated six-step conversation — it gets a decision, an owner, and an apology if the decision trampled someone (the §7.3 repair discipline, and the competing mode of §2.3).

---

## 4. Harvard Principled Negotiation: Fisher & Ury's *Getting to Yes*

### 4.1 The book

***Getting to Yes: Negotiating Agreement Without Giving In*** — by **Roger Fisher** and **William Ury**, first published **1981** by Houghton Mifflin; both authors were members of the **Harvard Negotiation Project** ✅. The second edition (1991) and the third (2011) added **Bruce Patton** as co-author; Patton had edited the first edition ✅. Verified facts about its reach: by 2022 Ury asserted on his website that **15 million copies** had been sold and it had been translated into **more than 35 languages** ✅ (Wikipedia, citing Ury's site). It is the single most influential book on negotiation ever written — and the intellectual foundation for both the IBR approach (§3) and the difficult-conversations framework (§5), whose authors all come from the same Harvard Negotiation Project lineage ✅.

### 4.2 The four principles of principled negotiation

The book's method — **principled negotiation** — is an alternative to positional bargaining (the haggling over stated positions that "produces unwise outcomes, is inefficient, and endangers an ongoing relationship" ✅). It rests on four principles ✅:

1. **Separate the people from the problem.** Negotiators are people first — with values, cultural backgrounds, and emotions; the relationship tends to become entangled with the problem. Address perception, emotion, and communication separately from the substance. The book's concrete moves: *put yourself in their shoes, discuss each other's perceptions, and make your proposals consistent with their values* (face-saving) ✅.
2. **Focus on interests, not positions.** A position is what someone says they want; an interest is *why* they want it. Behind opposed positions usually lie compatible interests — and the classic technique is to ask "why?" and "why not?" until the interest underneath the position is visible.
3. **Invent options for mutual gain.** Generate a wide range of possibilities before deciding — separate inventing from deciding, broaden the pie before dividing it.
4. **Insist on using objective criteria.** Agree on fair standards (market value, precedent, expert opinion, regulation) *before* arguing about outcomes, so the result rests on principle rather than on will.

### 4.3 BATNA — the negotiator's shield

The book's most famous single concept: **BATNA — the Best Alternative To a Negotiated Agreement** ✅ — "the results you can obtain without negotiating" ✅. The discipline: before entering any negotiation, know your walk-away; an agreement should never be accepted that is worse than your BATNA, and a strong BATNA improves any agreement you do accept (you can afford to say no, which is where negotiating power actually comes from) ✅. The architect's version: before the budget negotiation, the vendor renegotiation, or the priority dispute, know what you can deliver *without* the agreement — the alternative platform, the interim design, the fallback vendor. The person with the better BATNA has the leverage, and leverage is usually invisible until someone asks "and what happens if we don't agree?"

### 4.4 The three hard cases

The book devotes its final chapters to the situations where principled negotiation seems to fail ✅:

- **What if they are more powerful?** — develop your BATNA; the power balance shifts when your walk-away improves.
- **What if they won't play?** — use "negotiation jujitsu": don't attack their position, *reframe* it — ask questions, restate their position to check understanding, and direct attention back to the problem (the reframing skill of §7.2).
- **What if they use dirty tricks?** — name the trick, separately from the substance; negotiate about the rules of the game before continuing.

### 4.5 The criticisms, honestly

The Wikipedia record documents a substantial critical literature ✅: Bobette Wolski's 2012 law-review article argues interest-based negotiation is "not necessarily the ethical alternative" it is often presented as, and commentators (including former FBI hostage negotiator Chris Voss in *Never Split the Difference*, 2016) argue the method underweights power asymmetries and hardball tactics ✅. The architect's takeaway is not to discard the method but to stop treating it as a universal: principled negotiation is the right instrument when both parties can be brought to reason about interests and criteria — and the wrong one when the other side is negotiating in bad faith, holds all the power, or is not actually negotiating (see §10.3, escalation).

### 4.6 The four principles in architecture terms

The principles are abstract until they are translated. The architecture-desk version of each:

| Principle | The architecture translation | The sentence that uses it |
|---|---|---|
| Separate the people from the problem | The design is on the table; the designer is in the chair. Critique the ADR, not the author; the reviewer's identity is never the topic | "This option fails the operability criterion — not you, the option." |
| Focus on interests, not positions | "We need Kafka" (position) → "we need event delivery we can run at bank scale with governance" (interest); "we need batch" (position) → "we need a cutover the ops team can staff" (interest) | "What would the pattern have to give *your* team for you to support it?" |
| Invent options for mutual gain | The hybrid is usually dismissed too early: event backbone *with* a batch fallback for the first six months; gateway *with* an event-readiness programme in parallel | "Is there a third option that gives both teams their non-negotiables?" |
| Insist on objective criteria | The criteria are the referee: migration risk, operability, time-to-market, cost-if-wrong — agreed *before* the options are defended, and written into the ADR | "Against the criteria we agreed, this option scores — let's look at the scores, not the speakers." |

The unifying move behind all four: **the architect negotiates with criteria the way a lawyer negotiates with precedent** — the argument is never "I win", always "the agreed standard decides". That is what makes the outcome defensible in the corridor afterwards, and what makes the ADR (§9.4) write itself.

---

## 5. The Difficult Conversations Framework: Stone, Patton & Heen

### 5.1 The book

***Difficult Conversations: How to Discuss What Matters Most*** — by **Douglas Stone, Bruce Patton, and Sheila Heen** — was published by **Viking in 1999** ✅ (Internet Archive catalog record: publication date 1999, publisher "New York, N.Y.: Viking", ISBN 0670883395, with Patton and Heen as associated authors ✅; the Penguin paperback reissue followed in 2000, per the Sheila Heen Wikipedia record ✅). All three authors come from the Harvard Negotiation Project lineage — Heen is a member of the Project and co-founder of Triad Consulting Group ✅ — and the book explicitly "expands on the problem-solving approach set forth in *Getting to Yes*" ✅. Where *Getting to Yes* is about negotiation between parties, *Difficult Conversations* is about the conversations people avoid: the ones where the stakes are emotional, the identities are engaged, and the cost of saying the wrong thing is a damaged relationship.

### 5.2 The three conversations

The book's organising insight ⚠ (the structure is the book's signature and is held from the author's knowledge base; it was not re-verified against the book's own text this pass — audit #10): every difficult conversation is really *three conversations* running simultaneously:

1. **The "What Happened?" conversation** — the dispute about facts, intentions, and blame. Most people assume the disagreement is about *what happened*; the book's argument is that it is usually about the *stories* each side tells about what happened — and the two stories are usually *both* incomplete rather than one true and one false. The shift the book recommends: from "who is right / who is to blame" to a **contributions** frame — how did each of us contribute to this situation?
2. **The Feelings conversation** — what each party is feeling, and whether those feelings are legitimate to have, let alone to express. Feelings are not the noise around the problem; they are *part* of the problem, and an unresolved feelings layer will veto any intellectual agreement.
3. **The Identity conversation** — what this conversation means for each person's self-image: am I competent? am I a good person? am I worthy of respect? When a conversation threatens identity, people fight it with a ferocity out of all proportion to the topic.

The architect's translation: a design review that becomes a fight is almost never a "What Happened" fight about the design. It is a Feelings conversation (the designer feels their competence is being attacked) wearing an Identity conversation underneath (if this design is rejected, am I still a good architect?). Naming which of the three conversations is actually running is the single highest-leverage move in the whole framework.

### 5.3 From "truth vs truth" to learning

The book's central reframe: stop trying to win a contest of who-has-the-truth, and start **learning** — about the other person's story, about your own contributions, and about what a resolution would actually require ⚠ (book's core argument, flagged as not re-verified verbatim this pass — audit #10). Practical moves from the framework:

- **Sort out your own contributions first.** Before the conversation, ask: what did I do — or fail to do — that contributed to this situation? Owning your contribution is not the same as accepting blame for everything.
- **Check your assumptions about their intentions.** The book's famous distinction: the *impact* of someone's behaviour on you is fact; the *intention* you attribute to them is a story — and the story is usually unflattering. Separate impact from intent before the conversation.
- **Start from "both/and", not "either/or".** "I can hold my view *and* take yours seriously" — the "And" stance replaces the "But" stance.

### 5.4 When to use the framework

Difficult-conversations work is the *inside* of conflict management: it is what happens in the room before the models of §2–§4 can do their work. Use it when the conflict has an emotional or identity charge — a designer whose work was rejected, a vendor manager whose deal was overridden, a product owner whose roadmap was deprioritised. The framework is also the architect's self-defence: when you are the one who feels attacked, the three conversations are the mirror that shows you which conversation *you* are actually in. §11's worked example runs the framework inside a larger resolution.

### 5.5 The failure modes — how difficult conversations go wrong

Worth naming explicitly, because each failure mode maps to a specific correction:

1. **The premature solution.** One party opens with the fix ("you should have used a schema registry") before the problem has been jointly defined. The correction is IBR step 5 — agree what the problem *is* before anyone solves it — and the listening discipline of §6.1.
2. **The blame auction.** Each side's opening is a contribution-accounting of the other's sins. The correction is the contributions frame (§5.3): "how did each of us contribute?" — asked *before* the auction starts, because once the auction starts it cannot be un-started.
3. **The feeling veto.** The conversation stays strictly technical while one party's feelings are doing all the work underneath; the intellectual agreement is reached and then silently vetoed in the corridor. The correction is to name the feelings conversation explicitly (§5.2): "I notice this conversation has an emotional charge for both of us — let's say what's actually going on before we continue."
4. **The identity ambush.** The conversation turns out to be about competence or worth (the Identity conversation, §5.2) — and the party whose identity is threatened stops listening and starts defending. The correction is to protect identity *before* the substance: separate the person from the problem (§4.2) out loud, early.
5. **The performance.** The conversation is staged for an audience — a third party, an email thread, a steering committee — and every sentence is being delivered for effect rather than for resolution. The correction is structural: take it private (the §11 pre-conversation pattern), or name the audience explicitly: "we can resolve this here, or we can resolve it in front of the steering committee — which do you want?"

Each failure mode has the same shape: a three-conversations problem (What Happened / Feelings / Identity) being fought as if it were only the first one. The framework's whole point is that you can choose to fight the conversation you are actually in.

### 5.6 The unified stack: which model for which situation

The four models of §2–§5 are not competitors; they are layers of one stack. The selection table — the guide's one-page answer to "what do I reach for?":

| Situation | Primary model | Supporting skills | Output artifact |
|---|---|---|---|
| I need to know how *I* handle conflict, and what my default mode costs me | **TKI** (§2) | Self-awareness; mode-selection discipline (§2.4) | A personal mode profile; a named default to correct |
| Two colleagues are in conflict and I must resolve it without breaking the team | **IBR** (§3) | Active listening (§6.1); mediation arc (§8.2) | An agreement both own; optionally a decision note |
| I am negotiating — budget, scope, vendor terms, priority | **Principled negotiation** (§4) | BATNA discipline (§4.3); reframing (§7.2) | An agreement tested against criteria; a record |
| The conversation is emotionally or identity-charged and I am avoiding it | **Difficult conversations** (§5) | The three conversations (§5.2); NVC (§6.3); emotional regulation (§7.1) | A relationship repaired or a fight properly started |
| The conflict is an architecture decision that must stick | **All four, in sequence** | Criteria-first session (§9.2); escalation (§10.3) | **An ADR** (§9.4) |

Reading the table as a workflow: **the TKI tells you where you start; the difficult-conversations framework tells you what is actually going on in the room; IBR and principled negotiation tell you how to move; the ADR tells you how to make it stick.** An architect fluent in all four layers can take a conflict from first twitch to recorded decision without ever improvising — which is the difference between conflict management as a talent and conflict management as a craft.

---

## 6. The Skills Toolkit I: Active Listening and Assertive Communication

### 6.1 Active listening — the load-bearing skill

Every resolution model in §2–§5 assumes that at least one person in the room can actually listen: IBR step 4 ("listen first, talk second") and step 3 ("listen carefully to different interests") are listening skills wearing process clothes; *Getting to Yes* lists "not actively listening to the other party, but instead only listening to rebut" as one of the three communication problems that wreck negotiations ✅; the difficult-conversations framework is unthinkable without it. The craft of listening — the Rogers–Farson lineage, the levels of listening, paraphrasing, reflecting back — is fully derived in the sibling [Facilitation Skills](../management/facilitation_skills_guide.md) guide (§2 there), and this guide does **not** re-derive it. The architect's checklist when a conflict conversation is about to start:

- **Listen for total meaning** — content *and* feeling; the feeling channel is where the conflict actually lives (cross-ref facilitation guide §2.1).
- **Reflect back before responding** — "so what I hear you saying is…" — until the speaker confirms they were heard; this single move de-escalates more arguments than any rebuttal.
- **Resist the rebuttal reflex** — the *Getting to Yes* diagnosis: listening only to rebut is not listening at all ✅.
- **Silence is working** — do not rescue the room; the facilitation guide's §4.5 silence discipline applies verbatim to conflict conversations.

### 6.2 Assertive communication: the middle path

Assertiveness is the skill of stating your view, your needs, and your limits **clearly and firmly, without aggression and without capitulation** — the practitioner consensus is that it is "possibly the most important" skill for handling conflict (SkillsYouNeed's conflict-resolution page, verified this pass ✅). The three-way distinction is the standard frame:

| Stance | Pattern | Effect on conflict |
|---|---|---|
| **Passive** | "Whatever you think is best" | Conflict is avoided, resentment accumulates, the issue curdles (Deutsch's destructive spiral, §1.4) |
| **Aggressive** | "My way, and anyone who disagrees is wrong" | Conflict escalates; the other side digs in; relationship damage |
| **Assertive** | "Here is my view, my need, and my limit — and I hear yours" | Conflict stays in the task channel; both views stay on the table |

The workhorse verbal instruments, both standard practitioner material (⚠ — see audit #17; the SkillsYouNeed material documents the DESC pattern and "I" statements, and both are presented here as practitioner-standard rather than research-verified):

- **"I" statements** — *"When you changed the API contract without telling me, I felt blindsided, and I need a heads-up next time"* — versus the "you" version (*"You always change contracts without telling anyone"*) which lands as an attack. The "I" statement reports *your* experience and *your* need; it is much easier to hear ✅ (SkillsYouNeed).
- **The DESC script** — **D**escribe the situation factually, **E**xpress your feelings, **S**pecify what you want done, **C**onsequences (the positive ones first). A complete, non-attacking request in four moves ✅ (SkillsYouNeed).

### 6.3 Nonviolent Communication: the four components

**Nonviolent Communication (NVC)**, developed by clinical psychologist **Marshall Rosenberg** in the 1960s and 1970s, is the most systematic verbal discipline for keeping conflict out of the judgment channel ✅ (Wikipedia, verified this pass). Its four components ✅:

1. **Observation** — distinguish concrete observation from evaluation: "the API contract was changed on Tuesday without a review" (observation) vs "you keep breaking contracts" (evaluation).
2. **Feelings** — name the actual emotion: "I felt blindsided" — not the judgment dressed as a feeling: "I felt *ignored*" (which is an interpretation, not a feeling).
3. **Needs** — identify the underlying need: "I need predictability in the interface so my consumers don't break."
4. **Requests** — make a concrete, doable, present-tense request: "Would you be willing to give me 48 hours' notice before changing the contract?"

NVC's relationship to the other frameworks is direct: the four components are a *micro-implementation* of "focus on interests, not positions" (§4.2) — a position ("revert the change") is translated into an observation, a feeling, a need, and a request, which is exactly the interest underneath. ⚠ The evidence note: the Wikipedia record is candid that NVC's research base is heterogeneous — small trials and scoping reviews report improved self-reported empathy, but reviewers call for larger, more rigorous studies ✅. Use it as a discipline, not as a cure.

---

## 7. The Skills Toolkit II: Emotional Regulation, Reframing, Apology and Repair

### 7.1 Emotional regulation: the fight-or-flight override

Conflict is physiologically loaded: the body's threat response is what makes the rebuttal reflex (§6.1) so hard to resist and what makes positions harden in the room. The verified anchor points:

- *Getting to Yes* treats emotion as a first-class negotiation problem: the authors' advice is to *explore the causes of both your own and the other party's emotions*, to let the other side voice grievances, and to use apology as a symbolic gesture — techniques "to defuse anger" ✅.
- NVC's entire method is an emotional-regulation protocol: name the feeling, find the need, make a request (§6.3) ✅.

The practitioner mechanics this guide recommends, flagged ⚠ as standard practice rather than verified research (audit #16): **name it to tame it** (label the emotion internally — "I am angry because my design was dismissed" — which measurably reduces its grip); **the physiological pause** (the body's threat response peaks in seconds and takes minutes to clear — a deliberate pause, a walk, a glass of water, a postponed reply); and **the 24-hour rule for hot replies** (never send the angry message; draft it, then decide). The facilitation guide's energy discipline (§4.4 there — calmest exactly where the room is hottest) is the professional version of the same skill.

### 7.2 Reframing: the leverage move

Reframing is changing the *frame* around a statement without changing its facts — and it is the single most reusable move in conflict work. Three verified anchors:

- **Positions → interests** (Fisher & Ury): "I need the Kafka backbone" (position) → "I need event delivery that the operations team can actually run at bank scale" (interest). The frame change is what makes the conflict solvable ✅.
- **Negotiation jujitsu** (*Getting to Yes*'s answer to "what if they won't play?"): do not attack the position — *reframe it*: ask questions, restate their position to check understanding, and redirect attention to the problem ✅.
- **Face-saving**: "make your proposals consistent with their values" — the frame that lets the other side move without losing face ✅.

The architect's everyday version: when a stakeholder says "this design is over-engineered", do not defend the design — *reframe to criteria*: "what would the right amount of engineering look like for a system that must hold up under MAS audit? Let's define that, and then judge the design against it." The facilitation guide's §6.2 makes the same move ("convert the objection to a criterion") and is cross-referenced rather than re-derived.

The catalogue of reframes the architect actually uses, with the frame shift named:

| You hear (the frame you are given) | The reframe | The shift |
|---|---|---|
| "We need Kafka" / "We need batch" | "What must the pattern deliver for your team?" | Position → interest (§4.2) |
| "Your design is over-engineered" | "What would the right amount of engineering be — and how would we recognise it?" | Attack → criterion (§7.2) |
| "Risk is blocking the go-live" | "What evidence would risk need to unblock it?" | Person/mandate → condition |
| "You always pick the shiny tool" | "Which of the last three choices was wrong on the merits — and what was the criterion?" | Relationship → task (§1.2) |
| "We can't afford it" | "What would we need to believe about the cost for this to be affordable?" | Wall → testable belief |
| "I was offended by what you said" | "What did I say, and what did it mean to you?" | Intent-assumption → impact (§5.3) |

The craft note: a reframe is not a rhetorical dodge — it must be a *genuine* invitation to a better frame, offered in good faith, or it is just manipulation wearing a technique's clothes. The difference is whether you would accept the same reframe if it were aimed at you.

### 7.3 Apology and repair

Apology is the repair mechanism of relationships after conflict — and it is badly misunderstood. Two verified anchors:

- **Aaron Lazare**, the psychiatrist who wrote the definitive book on the subject — *On Apology* (Oxford University Press, **2004**) ✅ (Internet Archive catalog record verified this pass) — argued that a genuine apology is not about the apologiser's guilt but about **restoring the offended party's dignity**; a failed apology (the "I'm sorry *if* you were offended" non-apology) often does more damage than no apology at all ⚠ (the dignity thesis and the non-apology critique are the book's core arguments, held from the author's knowledge base — the book itself was not extracted this pass — audit #11).
- *Getting to Yes* endorses apology as a negotiation tool: a symbolic apology — "I'm sorry" for the *process* even when you are not wrong about the *substance* — defuses anger and costs nothing of your position ✅.

The working anatomy of a repair conversation (⚠ practitioner-standard, consistent with both anchors): (1) name what you did, specifically — no "if", no "but"; (2) acknowledge the impact on the other person; (3) state what will change; (4) do not demand forgiveness or immediate re-equality — the relationship repairs on the other person's clock. The architect's special case: **the apology for the imposed decision** — when you competed (§2.3) because the deadline demanded it, the repair is a sentence that costs nothing and buys back the relationship: "I overrode your design because the audit deadline left no room — I want to say that clearly, and I want the next cycle to include your approach properly."

---

## 8. Mediation Basics and the Facilitator's Hat

### 8.1 Facilitation, mediation, arbitration, decision — know which hat is on

The facilitation guide's §1.3 distinguishes the four hats precisely, and this guide does not re-derive them: the **facilitator** owns the process of a group producing its own outcome; the **mediator** works *between* conflicting parties toward an agreement they can both own; the **chairperson** rules on procedure with authority; the **trainer** transmits content. The architect's discipline is knowing which hat is on at any moment and saying so when you switch (§1.3 of the facilitation guide, cross-referenced).

For conflict specifically, the decision rule is: **mediate when two parties are in conflict and the session exists to settle it; facilitate when a group must decide something and conflict is incidental; arbitrate or decide when the parties cannot agree, the deadline cannot move, and someone must own the call** (the competing mode of §2.3, and the escalation paths of §10.3). The boundary failure to avoid: drifting into deciding while still claiming the mediator's neutrality — the facilitation guide's warning applies verbatim.

### 8.2 The mediation arc

The mediation process mechanics — the opening, ground rules, storytelling, issue-listing, option generation, agreement — are process craft and live in the sibling guides (the facilitation guide's difficult-situations playbook §7, and the workshop guide's technique library §4–§6, cross-referenced, not re-derived). What this guide adds is the *conflict-specific* spine, assembled from the verified frameworks of §2–§5:

| Stage | Instrument | Verified source |
|---|---|---|
| **1. Set the container** | Ground rules stated out loud: one conversation, both heard, issues not persons; the mediator's neutrality declared | Facilitation guide §1.2/§1.3 (cross-ref) |
| **2. Separate people from problems** | Each party tells their story; the mediator reflects content and feeling back; the person is never the target | Fisher & Ury principle 1 ✅; IBR step 2 ✅ |
| **3. Surface the interests** | Why-questions until the interests under the positions are visible; NVC four components as the verbal discipline | Fisher & Ury principle 2 ✅; IBR step 3 ✅ |
| **4. Agree the facts** | The group agrees what the problem *is* and which facts are relevant, jointly | IBR step 5 ✅ |
| **5. Generate options together** | Brainstorm a third position; the parties own the solution | IBR step 6 ✅; Fisher & Ury principle 3 ✅ |
| **6. Settle against criteria** | The chosen option is tested against agreed standards, and the dissent is recorded | Fisher & Ury principle 4 ✅; disagree-and-commit (facilitation guide §6.3, ⚠ unattributed) |
| **7. Capture the outcome** | The resolution is recorded as an ADR (or minutes + actions) so the conflict produces an artifact | [Architecture Decision Records](../technology/architecture_decision_record_guide.md) (cross-ref, §9.4) |

### 8.3 The mediator's stance

Two virtues, from the facilitation guide's §1.2 (cross-referenced): **neutrality** (no stake in the outcome) and **impartiality** (no favourite person). The mediator's one permitted bias is the same as the facilitator's: bias toward *participation* — more voices, more honesty, more thinking. And the professional limit: when the conflict involves the mediator personally, or the parties are so escalated that a neutral container cannot hold, the right move is to hand the dispute to someone else — a peer architect, a people manager, or HR — before the intervention makes things worse (the facilitation guide's self-knowledge discipline, and SkillsYouNeed's blunt version: "it is much better to ask for help than to step in and make matters worse" ✅).

---

## 9. Conflict in Architecture Work: Requirements, Design, and Priorities

### 9.1 Requirements conflicts: the ambiguous mandate

The architecture lifecycle's first conflict generator is the requirements phase — and the verified antecedent list of §1.3 explains exactly why: low task/goal uncertainty (ambiguous mandates invite competing interpretations) and lack of information sharing (each stakeholder reasons from their own slice) are both structural features of early requirements work ✅. The recurring shapes:

- **Two stakeholders, two legitimate requirements** — the business wants time-to-market, risk wants control, ops wants operability; all three are right, and the requirements conflict is really a priority conflict wearing requirement clothes.
- **The unstated requirement** — the stakeholder who cannot (or will not) say what they actually need; the conflict surfaces only when the delivered design misses it. The communication guide's concerns-behind-questions discipline (cross-ref [Communication & Stakeholder Management Skills](../management/communication_stakeholder_management_skills_guide.md) §2.1) is the diagnostic instrument.
- **The moving requirement** — scope changes mid-flight; the conflict is between the change's sponsor and everyone whose plan it invalidates.

The resolution discipline is the §1.5 recipe applied early: make the criteria explicit before the options, get both requirements onto the table in the stakeholders' own words (the facilitation guide's reflecting-back craft, §2.4 there), and convert each requirement into a *testable criterion* — the facilitation guide's "convert the objection to a criterion" move, applied to requirements. What the architect must *not* do is resolve requirements conflicts by private reinterpretation — the requirement that wins by quiet editing becomes the requirement that fails in audit.

### 9.2 Design disagreements: task conflict at its best and worst

Design disagreement is the architect's native habitat — and the evidence of §1.5 says it is the one kind of conflict that can genuinely improve outcomes: task conflict, kept in the task channel, in a group whose relationships can absorb it ✅. The discipline for running a design disagreement so it produces a better design rather than a scar:

1. **Separate the people from the problem** (principle 1, §4.2) — the design is on the table; the designer is in the chair. Attack the first, protect the second.
2. **Criteria before positions** — agree what a good answer must satisfy *before* anyone defends an answer (the workshop guide's §9 decision-session signature questions, cross-referenced; the facilitation guide's §9.2).
3. **Both designs get a fair hearing** — the facilitation guide's ground rule, verbatim: every option gets a fair hearing before any option gets evaluated (§11.1 there).
4. **Facts, jointly agreed** (IBR step 5) — benchmark results, volume numbers, operational cost estimates; most design wars are fought over two different sets of facts.
5. **Settle against criteria, record the dissent** — the decision is made on the agreed criteria, the dissent is recorded as a monitored risk (disagree-and-commit, facilitation guide §6.3), and the whole thing is captured as an ADR (§9.4) so the argument produces an artifact.

The failure mode to name: design disagreement descending into relationship conflict (Jehn's taxonomy, §1.2) — the moment the argument becomes "you always pick the shiny tool", the technical debate is over and the Deutsch destructive spiral (§1.4) has begun. The intervention is the same one the facilitation guide's §6.3 runs: reflect the conflict, not the content; keep it in the task channel; get back to criteria.

### 9.3 Priority disputes: who wins when both are right

Priority disputes are the most common architecture-team conflict, and the least amenable to pure reason — because both sides are usually right about *their* priority. The verified sources are the scarce-resources and perceived-incompatible-goals elements of the conflict definition (§1.1), and the practitioner fourfold's "priorities" and "resources" sources (§1.3, ⚠). The discipline:

1. **Make the trade-off explicit.** The conflict is rarely "which project is better" — it is "which project loses what, and who absorbs the loss". Naming the trade-off converts a values fight into a decision problem.
2. **Use a shared prioritisation instrument.** The workshop guide's converge techniques (MoSCoW, weighted scoring, dot-voting — cross-ref [Workshop Methodology](../management/workshop_methodology_guide.md) §6) give the dispute a neutral mechanism; the facilitation guide's private-vote discipline (public voting in front of the sponsor is a poll of the sponsor's mood, not the room's thinking) applies verbatim.
3. **Escalate to the accountable decider.** When two priorities are both legitimate and the resources are genuinely scarce, the conflict is *not resolvable by the parties* — it is a decision for the person accountable for both. The architect's job is to package the decision: options, costs, risks, and a recommendation — and to make sure the decision is recorded (ADR or steering-committee minutes). This is escalation as a *service*, not as a failure (§10.3).
4. **Protect the loser of the round.** The deprioritised initiative's sponsor needs the face-saving frame (§7.2) and a recorded revisit date — otherwise this round's priority dispute becomes next quarter's relationship conflict.

### 9.4 Recording resolutions: the ADR as the conflict's artifact

Every conflict this guide covers produces — or should produce — a decision. The repository's convention for capturing architecture decisions is the **Architecture Decision Record** (Context / Decision / Consequences), fully derived in [Architecture Decision Records](../technology/architecture_decision_record_guide.md), cross-referenced here, not re-derived. The conflict-specific guidance:

- **Write the ADR while the reasoning is fresh** — the facilitation guide's §11.5 discipline: draft the Context (the options, the criteria, the conflict), the Decision (what was chosen and *on what criteria*), and the Consequences (including the recorded dissent as a monitored risk).
- **A condition is part of the decision** — the facilitation guide's correction of its own worked example: "the ops manager's condition should have been written into the Decision line, not the Consequences — a condition is part of the decision" (§11.7 there). When a conflict is settled on a condition, the condition belongs in the Decision.
- **The ADR is what makes the conflict defensible afterwards** — the corridor conversation "why did we pick this?" is answered by the record, not by the loudest survivor. In a regulated bank, the ADR is also the audit trail: it shows the criteria, the alternatives, and the dissent — exactly what MAS-style supervision and internal audit ask for (cross-ref the ADR guide's §10 on the regulated-banking angle).

### 9.5 The architecture conflict pattern catalogue

A quick-reference of the recurring conflict patterns in architecture work, each with its diagnosis (using §1–§2) and its treatment (using §3–§8):

| Pattern | How it presents | Diagnosis | Treatment |
|---|---|---|---|
| **The pattern war** | Two senior architects, two platforms, one decision; the debate leaks upward | Task conflict + ambiguous authority (§1.2–§1.3) | Criteria-first decision session (§9.2), collaborating mode (§2.3), ADR with conditions (§9.4), sponsor as decider (§10.3) |
| **The silent veto** | Everyone agrees in the room; nothing ships; the objector never spoke | Relationship conflict in hiding; avoiding mode (§2.3) | The false-consensus check (§11.4); written-first rounds (facilitation guide §6.1, cross-ref); private pre-conversations (§11.3) |
| **The turf war** | Two teams both own the same component; ownership is contested | Process conflict (§1.2) | Name the ownership question as the real issue; decide it explicitly; record it in the ADR and the RACI (cross-ref the workshop guide's role-clarification techniques) |
| **The moving requirement** | Scope changes mid-flight; the plan's owners are blindsided | Sources: priorities + information sharing (§1.3) | Make the trade-off explicit (§9.3); change-control with the sponsor present; the "moving requirement" gets a decision, not a grudge |
| **The vendor stand-off** | The vendor's reference architecture clashes with the bank's risk posture | Technical conflict with a commercial layer (§10.2) | Criteria and facts first (IBR steps 5–6); BATNA known (§4.3); escalate to the contract and the vendor manager when it turns commercial (§10.2) |
| **The deadline fight** | Two workstreams, one release slot, both mandated | Priority dispute over scarce resources (§9.3) | Shared prioritisation instrument (§9.3); escalate packaged to the accountable decider; protect the loser of the round with a recorded revisit date |
| **The identity ambush** | A design review turns into a competence trial; the designer stops listening | The Identity conversation (§5.2) | Separate the person from the problem, out loud (§4.2); the §5.5 corrections; repair if the damage is done (§7.3) |

The catalogue's purpose is triage speed: the faster the pattern is named, the faster the right instrument is on the table — and the cheaper the resolution (the §1.6 law).

---

## 10. Cross-Functional Conflict, Vendor Conflict, and Escalation Paths

### 10.1 Cross-functional conflict: the bank's permanent friction

In a bank, architecture work is a relay between functions that do not share incentives: **business** (time-to-market, revenue), **engineering** (quality, technical debt), **operations** (stability, runnability), **risk & compliance** (control, evidence), and **security** (protection, least privilege). Conflict between functions is not a malfunction — it is the system working as designed, each function defending its mandate. The discipline:

- **Name the mandate conflict.** A risk team blocking a go-live is not being difficult; it is doing its job. The reframe (§7.2) turns "risk is blocking us" into "risk's mandate and delivery's mandate are in conflict — what criteria reconcile them?"
- **Build the shared artifact.** The ADR, the design doc, the risk register — the artifact that both functions contribute to and both sign is the container that holds cross-functional conflict without it becoming personal. This is the §1.2 taxonomy in action: keep it task conflict, make the task a shared document.
- **Bring the decider in early, not late.** Cross-functional conflicts in a bank usually terminate at a governance body — the Architecture Review Board, the change advisory board, the steering committee. The workshop guide's decision-session rule applies: the sponsor/decider must be present for decision sessions, or the session is a rehearsal (cross-ref [Workshop Methodology](../management/workshop_methodology_guide.md) §9.2).
- **Protect the relationship channel.** The facilitation guide's §6.3 disagree-and-commit convention is the exit: the dissent is recorded, the decision is committed to, and the relationship survives to fight the next issue on the merits.

### 10.2 Vendor conflicts: the commercial layer

Vendor conflict is conflict with an external party, which changes the instruments: the relationship is contractual, the power is commercial, and the resolution mechanisms are escalation ladders, service credits, and contract terms rather than facilitated conversations alone. The vendor-management deep-dive owns this territory ([Vendor Management](../management/vendor_management_guide.md), cross-referenced — this guide does not re-derive it); the conflict-specific notes:

- **Distinguish the technical dispute from the commercial dispute.** An architect's disagreement with a vendor's solution architect ("your reference architecture doesn't fit our risk posture") is a technical conflict, resolvable by criteria, facts, and design review — IBR and principled negotiation apply (§3–§4). The moment it becomes a dispute about SLA breaches, penalties, or scope pricing, it is a commercial conflict and belongs to the contract, the vendor manager, and the escalation path defined in the contract (cross-ref the vendor guide's performance and risk sections).
- **Know your BATNA before the fight** (§4.3). In vendor conflict, the BATNA is the second source, the exit clause, the in-house alternative. The vendor knows its own leverage precisely; the architect should know the bank's.
- **Never let a technical disagreement become a personal one.** The vendor's architect is a professional counterpart, not an enemy; the relationship survives the dispute if the dispute stays on the merits — the Deutsch condition (§1.4) applies across the contract boundary too.
- **Document, document, document.** The vendor conflict that ends in arbitration is won by the side whose position is documented. The ADR convention (§9.4) applies to vendor design decisions; the vendor guide's scorecards and issue logs apply to vendor performance disputes (cross-ref).

### 10.3 Escalation paths: the ladder and the protocol

Escalation is not a failure of conflict management — it is a *phase* of it. The discipline is knowing the ladder, using the rungs in order, and escalating as a service (packaged decisions) rather than as a complaint.

**The ladder, in order:**

1. **The direct conversation** — the parties themselves, using the §5 framework and the §6–§7 skills. Most conflicts end here, and should: the facilitation guide's rule that early intervention is cheaper holds for conflict generally (SkillsYouNeed's version, verified: "dealing with conflict early is usually easier, because positions are not so entrenched, others are less likely to have started to take sides, and the negative emotions are not so extreme" ✅).
2. **The manager / the lead** — when the parties cannot settle it, the conflict escalates to the person accountable for both: the architecture lead, the engineering manager. This rung uses the IBR six steps (§3.2) and the mediation arc (§8.2), wearing the manager's hat rather than the mediator's.
3. **The neutral third party** — when the manager is part of the conflict, or neutrality is compromised (§8.3), bring in a peer architect, a staff engineer, or an HR/people partner to mediate.
4. **The governance body** — for architecture decisions that will not settle: the Architecture Review Board, with the decision packaged (options, criteria, costs, risks, recommendation) and the outcome captured as an ADR (§9.4). The ARB is the escalation terminus for design conflict, exactly as the ADR guide's §9 frames it.
5. **Executive / steering committee** — for cross-functional or vendor conflicts above the ARB's authority: the packaged decision goes to the accountable executives, with the trade-offs named (§9.3).

The same ladder, as a decision table:

| Rung | Who | Instrument | Artifact |
|---|---|---|---|
| 1. Direct conversation | The parties | §5 framework + §6–§7 skills | Understanding; often nothing formal |
| 2. Manager / lead | The accountable lead | IBR six steps (§3.2), mediation arc (§8.2) | Decision note or action items |
| 3. Neutral third party | Peer architect, staff engineer, HR partner | Mediation (§8); the hats discipline (§8.1) | Agreement summary |
| 4. Governance body | Architecture Review Board | Packaged decision (options, criteria, costs, risks, recommendation) | **ADR** (§9.4) |
| 5. Executive / steering | Accountable executives | Trade-off brief; competing call if required (§2.3) | Steering minutes; decision record |

The table reads top-to-bottom as escalation, and bottom-to-top as delegation: the higher rungs *should* delegate back down whenever the lower rung can carry the decision — an executive who decides what an ARB should decide, or an ARB that decides what a conversation should decide, is not managing conflict, it is hoarding it.

**The protocol — four rules:**

- **Escalate the issue, not the person.** "I need a decision on X because the deadline moves" — never "I need you to overrule Y".
- **Escalate with a recommendation.** The decider's job is to decide, not to redo the analysis; the workshop guide's decision-session discipline applies: options, criteria, and a clear recommendation.
- **Escalate before the deadline, not after.** An escalation that arrives after the point of no return is a blame-transfer, not a decision request.
- **Record the outcome.** Whatever the rung, the resolution is captured — ADR, minutes, action item — so the conflict leaves an artifact and the ladder does not have to be re-climbed for the same issue next quarter.

The management-frame context for all of this — how escalation, delegation, and the manager's role in conflict interact — is covered in [The Manager's Path](../management/the_managers_path_guide.md) (the 1:1 and feedback disciplines, cross-ref its chapters on mentoring and the tech-lead role) and [3D Managerial Effectiveness](../management/3d_managerial_effectiveness_guide.md) (situational style selection — the managerial-grid lineage that the TKI of §2 descends from), and the consulting-side framing in [Management Consulting Skills](../management/management_consulting_skills_guide.md) (structured problem-solving applied to organisational disputes).

---

## 11. The Worked Example: The Cymbal Bank Payment-Hub Conflict

⚠ Like every worked example in this repository, this is a pedagogical construction consistent with the Cymbal Bank persona — the cast, the dates, and the ADR are illustrative, not facts about any real bank (audit #22). It is written to show every instrument of this guide running in one scenario: the diagnosis, the mode selection, the conversation, the resolution, and the follow-up.

### 11.1 The scenario and the cast

Cymbal Bank, Singapore. The payments modernisation programme must replace the legacy payment hub. Two senior architects are in open conflict over the integration pattern for the new hub:

- **Mei Lin** — senior solution architect for payments. Position: *"We standardise on the event-driven backbone — Kafka — for all payment events. Batch is a dead end; we've deferred this twice already."*
- **Ravi** — platform architect, owns the integration platform and operations. Position: *"We are not ready for streaming. Our ops runbook is built for batch and file-based integration; the last two streaming pilots bled incidents. We standardise on the gateway, and we revisit streaming next year."*

The conflict is public: it has leaked into the programme steering committee, the two architects have stopped attending each other's design reviews, and the rest of the team has started taking sides. The programme sponsor — **Grace**, head of architecture — has asked the author (the lead solution architect for the programme) to resolve it.

The stakes: the integration-pattern decision gates the whole programme, and the next regulatory reporting deadline (MAS 610-style reporting changes) is fixed and immovable — the reporting workstream needs *a* decision, not *the* decision, by end of month.

### 11.2 The diagnosis — before any conversation

The first discipline is diagnosis, using §1–§2. What kind of conflict is this, actually?

- **On the surface: task conflict** (§1.2) — a genuine technical disagreement about the integration pattern, with real facts on both sides. Mei Lin's interest is modernisation and future capability; Ravi's is stability and operability. Both are legitimate. This is the kind of conflict §1.5 says can be productive — *if* it stays in the task channel.
- **Underneath: relationship spillover** (§1.2) — the tell is the behaviour: skipped design reviews, sides being taken, the steering committee leak. The task conflict has already started curdling into relationship conflict, which the §1.5 evidence says is where the damage happens. The Deutsch spiral (§1.4) is turning cooperative process into competitive process.
- **The process layer: ambiguous authority** (§1.3) — two senior architects, two legitimate mandates, one decision. Neither can outrank the other, and both know it; that is why the conflict leaked upward instead of resolving.
- **The sources** (§1.3): *priorities* (modernisation vs stability), *resources* (ops capacity to run streaming), and *styles* (Mei Lin's competing default vs Ravi's avoiding default — see below).

The mode diagnosis, using §2.3: **Mei Lin's default is competing** — she is right about the direction and knows it, and she is winning the argument in public while losing the room. **Ravi's default is avoiding** — he has withdrawn from the shared forums rather than fight, which reads as passive obstruction and feeds Mei Lin's frustration. **The situation calls for collaborating** — the decision is high-stakes, both sets of concerns are legitimate, and the relationship must survive the decision because both architects will run the platform together for years. The only competing move on the table belongs to the *deadline*, not the pattern: the regulatory reporting workstream needs its interface decision by end of month regardless of the pattern debate — that decision will be made (competing, by the accountable sponsor) while the pattern debate gets the collaborative treatment it needs.

### 11.3 The preparation — the three conversations, run on yourself first

Before the joint session, the author runs the §5 framework on the situation:

- **The "What Happened?" conversation:** both stories are incomplete, not one true and one false. Mei Lin's story: "Ravi blocks every modernisation." Ravi's story: "Mei Lin steamrolls and leaves ops holding the bag." The contributions frame: the author's own contribution — the programme let the pattern decision drift for two cycles without a decision rule, which is what let the conflict become public. Grace's contribution: the steering committee heard the conflict before the architects did.
- **The Feelings conversation:** Mei Lin feels her expertise is being dismissed; Ravi feels his team's operational reality is being ignored. Neither feeling is illegitimate; both must be said out loud before the technical conversation can work.
- **The Identity conversation:** for Mei Lin, the pattern choice is bound up with "am I a modern architect?"; for Ravi, with "am I the guardian of stability, or the person who gets blamed when streaming fails?" The identity stakes are why the arguments are so fierce relative to the topic (§5.2).

The preparation output: a decision rule to propose (one decision, one owner, criteria first, ADR at the end), a joint session design (borrowed from the workshop guide's decision-session mechanics, cross-ref §9 there), and a private pre-conversation with each architect (the §5 one-on-one, using the listening and NVC instruments of §6).

### 11.4 The conversations

**Pre-conversation with Ravi** (the author, one-on-one, 30 minutes). The moves, in order:

> Author: "I want to understand the streaming story from your side before anything else. What happened in the two pilots, in your words?" — *listening first, talk second* (IBR step 4); the facts come out: both pilots failed on schema governance and consumer readiness, not on the broker itself.
> Author: "So the pattern you're against is *streaming without governance* — is that fair? If the event backbone came with schema governance and a consumer-readiness gate as conditions of the decision, would that change your position?" — *reframing position → interest* (§7.2), *converting the objection to a criterion* (facilitation guide §6.2, cross-ref).
> Ravi: "If the conditions were real — written into the decision, not a footnote — yes. I'm not against events. I'm against being handed a platform I can't run."

The Feelings and Identity layers surface and are acknowledged — Ravi's fear of being blamed for the next incident, his team's pride in keeping the bank stable. No argument is made against any of it. The conversation ends with the author reflecting back: "So your position is: event backbone, conditional on schema governance, validation gates, and a consumer-readiness gate, all written into the decision." Ravi agrees that this is his position, accurately restated — the first time in months someone has restated it.

**Pre-conversation with Mei Lin** (the same discipline, reversed):

> Mei Lin: "Ravi blocks everything. We'll never modernise at this rate."
> Author: "Let me check something — is it Ravi you're frustrated with, or the pattern debate going nowhere for two cycles?" — *separating person from problem* (IBR step 2; the three-conversations move: impact vs attributed intention, §5.3).
> Mei Lin: "...The debate going nowhere. Ravi's a good architect. I just can't get a hearing."
> Author: "He told me the same thing about you — that you don't hear his operations reality. What would it take for you to hear it?"

The author then tests the emerging synthesis with Mei Lin: an event backbone with the governance conditions as *part of the decision*. Her reaction is the Identity conversation in miniature — "so we're giving ops a veto over architecture?" — and the author reframes: "we're giving ops conditions we were going to need anyway. The alternative is another two cycles of this." Mei Lin agrees to the joint session.

**The joint session** (90 minutes, decision rule stated up front, per the workshop guide's §9 discipline — the author facilitates, Grace the sponsor attends as decider, the junior engineer who owns the consumer codebase is invited deliberately, per the facilitation guide's quiet-participant discipline):

1. **Opening** — the decision rule out loud: "One decision in the next 90 minutes: the integration pattern for the payment hub, with its conditions. Criteria first, options second. The decision leaves here as an ADR. Every option gets a fair hearing before any option gets evaluated."
2. **Criteria before positions** — the room produces, in its own words: migration risk, operability (can ops run it), time-to-market, and *cost if we're wrong*. The criteria go on the board in the room's words (facilitation guide §2.4, cross-ref).
3. **The option walkthrough** — Mei Lin presents the event backbone; Ravi presents the gateway; each gets a fair hearing. The storming arrives on schedule (Tuckman's stages, cross-ref facilitation guide §5): Mei Lin calls batch "a tax on every future change"; Ravi calls streaming "a pilot that has already failed twice here".
4. **The facilitator's interventions** — reflect the conflict, not the content: "Two claims on the table: batch has failed us, and streaming has failed us *as operated*. Both stay on the board." Then the criteria move: "Ravi — what would have to be true for you to support the event backbone?" — the answer is the three conditions, extracted as criteria instead of argued with. Then the quiet participant, deliberately: "Daniel, you know the consumer codebase best — are the consumers ready for events?" — his answer (the topic contracts exist, the consumers are not ready) changes the cost of the event option and is protected: "that's the strongest point in the last ten minutes — let's hold it."
5. **The decision** — the private dot-vote on the criteria-weighted options (the workshop guide's NGT mechanics, cross-ref §6 there): event backbone with conditions, four votes; gateway, one; hybrid, two. The false-consensus check is asked by the facilitator: "Silence isn't agreement here — does anyone need to speak before we commit?" Ravi's hand goes up: "I vote with the conditions — written into the decision, not the consequences." The room agrees. The dissent is converted into a monitored risk, not a footnote: the gateway option is recorded as the fallback if the conditions fail.

### 11.5 The resolution and the ADR

The ADR is drafted in the room (the facilitation guide's §11.5 discipline; the ADR conventions cross-referenced from [Architecture Decision Records](../technology/architecture_decision_record_guide.md), not re-derived):

- **Context** — the payment hub replacement; the two options; the criteria (migration risk, operability, time-to-market, cost-if-wrong); the two failed streaming pilots, diagnosed as governance failures rather than broker failures; the consumers' readiness gap.
- **Decision** — standardise on the event-driven backbone for payment events, **conditional on**: (1) schema governance for all event contracts, (2) automated validation gates, (3) a consumer-readiness gate before cutover — *the conditions in the Decision line, because a condition is part of the decision* (facilitation guide §11.7, cross-ref).
- **Consequences** — the consumers' readiness gap (owned by Daniel's team, with a date); the ops runbook investment (owned by Ravi's team); the recorded dissent — the gateway remains the documented fallback if the conditions fail, monitored at the architecture review board; the regulatory reporting workstream proceeds on the interim interface decision (the competing call, made by Grace, recorded separately).

The ownership is explicit: Mei Lin owns the ADR number and the schema-governance condition; Ravi owns the ops-readiness condition; Grace confirms the decision rule; the ARB tracks the conditions at the next review (§10.3, rung 4).

### 11.6 The follow-up and the debrief

- **The follow-up, one month later:** the schema-governance working group is live; the consumer-readiness assessment is on the ARB agenda; Ravi's team has the runbook investment in the platform roadmap. The two architects are co-presenting the ADR to the steering committee — a public signal that the relationship survived, which is the Deutsch condition (§1.4) made visible.
- **The debrief, in the author's own journal** (the facilitation guide's §10.4 discipline): what worked — the criteria-first design, the pre-conversations (the resolution was 80% done before the joint session), the conditions-in-the-Decision rule; what would be done differently — the conflict should have been diagnosed and named *before* it reached the steering committee (the §1.2 taxonomy would have caught the relationship spillover two cycles earlier), and the decision rule should have existed from the start of the programme, not from the day of the session.
- **The lesson the example is built to teach:** the same conflict, run without the craft, ends with one architect winning, one architect resigning in a year, and the platform carrying the scar; run with the craft, it ends with a better decision — the conditions genuinely improved the design — and a team that trusts the process. That is the difference the whole guide is about: *conflict is inevitable; the outcome of conflict is a choice.*

---

## 12. The Claims Audit

The house rule of this repository: never present an unverified claim as fact. This audit lists every load-bearing factual claim in the guide, with its verification status. **✅ = verified this pass at a primary or authoritative source** (URLs given); **⚠ = flagged** (single-source, not re-verified, or held from the author's knowledge base); **❌ = disputed or false as stated** (correction given in the text).

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | TKI introduced in 1974 by Kenneth W. Thomas and Ralph H. Kilmann; first published by Xicom (Tuxedo, NY) | ✅ | en.wikipedia.org/wiki/Thomas–Kilmann_Conflict_Mode_Instrument ("In 1974, Kenneth W. Thomas and Ralph H. Kilmann introduced their Thomas–Kilmann Conflict Mode Instrument (Tuxedo NY: Xicom, 1974)", citing the instrument itself); kilmanndiagnostics.com ("Dr. Ralph H. Kilmann co-created the TKI assessment in 1974") |
| 2 | TKI's five modes: competing, collaborating, compromising, avoiding, accommodating | ✅ | Wikipedia TKI article (mode definitions with axes); APA PsycNet test record DOI 10.1037/t02326-000 ("competing, collaborating, compromising, avoiding, and accommodating"); kilmanndiagnostics.com (five conflict modes listed verbatim) |
| 3 | TKI maps modes on two axes: assertiveness and cooperativeness; mode labels (competing = assertive/uncooperative, etc.) | ✅ | Wikipedia TKI article ("The TKI uses two axes... 'assertiveness' and 'cooperativeness'"; per-mode axis combinations listed) |
| 4 | TKI lineage: based on the managerial grid of Blake & Mouton (concern for people / concern for production) | ✅ | Wikipedia TKI article (development section); en.wikipedia.org/wiki/Managerial_grid_model (model, 1964; *The Managerial Grid: The Key to Leadership Excellence*, Gulf Publishing, 1964) |
| 5 | TKI instrument: 30 pairs of statements, forced choice, pairs matched for social desirability | ✅ | Wikipedia TKI article; APA PsycNet record ("statement pairs were evenly matched in terms of desirability") |
| 6 | TKI scale: "more than 10 million copies since 1974"; moderate test-retest repeatability and internal consistency per a 1978 analysis of 86 responses; overuse/gaming criticism | ✅ | kilmanndiagnostics.com homepage (10M+ since 1974); Wikipedia TKI article (1978 analysis; 1993 columnist criticism) |
| 7 | IBR approach developed by Fisher & Ury and published in *Getting to Yes* (1981) | ✅/⚠ | pmiglc.org "Conflict Resolution: Using the Interest-Based Relational Approach" (Sept 2023, reproducing Mind Tools): "Roger Fisher and William Ury developed the IBR approach and published it in their 1981 book, Getting to Yes" — the *attribution* is ✅ at multiple practitioner sources; the exact coinage of the "IBR" *label* is ⚠ (no primary academic source pinned this pass) |
| 8 | IBR's six steps (relationships first; separate people from problems; listen to interests; listen first, talk second; set out the facts; explore options together) | ✅ | pmiglc.org PDF (six steps verbatim) |
| 9 | IBR inappropriate in crisis; "pull rank" guidance | ✅ | pmiglc.org PDF ("you may not be able to resolve differences... if your organization is in a crisis. On these occasions, you may have to 'pull rank'") |
| 10 | *Getting to Yes* (1981), Roger Fisher & William Ury, Houghton Mifflin; both members of the Harvard Negotiation Project | ✅ | en.wikipedia.org/wiki/Getting_to_Yes ("best-selling 1981 non-fiction book by Roger Fisher and William Ury"; publisher Houghton Mifflin; "All of the authors were members of the Harvard Negotiation Project") |
| 11 | 1991 second edition and 2011 third edition added Bruce Patton as co-author | ✅ | Wikipedia *Getting to Yes* ("Subsequent editions in 1991 and 2011 added Bruce Patton as co-author") |
| 12 | The four principles: separate the people from the problem; focus on interests, not positions; invent options for mutual gain; insist on objective criteria | ✅ | Wikipedia *Getting to Yes* (four principles listed verbatim) |
| 13 | BATNA: Best Alternative To a Negotiated Agreement — "the results you can obtain without negotiating"; protects against bad agreements and improves accepted ones | ✅ | Wikipedia *Getting to Yes* (BATNA section, pp. 99–108 and p. 102 in the 3rd ed.; the three suggestions for developing a BATNA) |
| 14 | *Getting to Yes* reach: 15 million copies, more than 35 languages (Ury's claim, 2022) | ✅ | Wikipedia *Getting to Yes* ("As of 2022, Ury asserted on his website that 15 million copies... translated into more than 35 languages") |
| 15 | TKI per-mode situational guidance ("when to use" lists) | ⚠ | Consistent with the publisher's and co-author's materials but not re-verified page-by-page this pass; presented as standard interpretive guidance |
| 16 | Emotional-regulation mechanics (name it to tame it; physiological pause; 24-hour rule for hot replies) | ⚠ | Standard practitioner practice, unattributed; presented as such (§7.1) — the *anchors* (Getting to Yes's emotion guidance; NVC's method) are ✅ |
| 17 | "I" statements and the DESC script | ⚠/✅ | Documented as practitioner material on skillsyouneed.com (verified this pass ✅ as practitioner material); presented as practitioner-standard, not research-verified |
| 18 | Practitioner fourfold sources of conflict: priorities, resources, styles, values | ⚠ | Training-room lore, unattributed; presented as such (§1.3) — the research *antecedents* list is ✅ (see #24) |
| 19 | Interpersonal-conflict definition ("an expressed struggle between at least two interdependent parties who perceive incompatible goals, scarce resources, and interference...") | ⚠ | Quoted on skillsyouneed.com; commonly attributed to Hocker & Wilmot's *Interpersonal Conflict*, but the attribution was not re-verified at a primary source this pass |
| 20 | *Difficult Conversations: How to Discuss What Matters Most* — Stone, Patton & Heen; Viking, 1999; Penguin paperback 2000; authors of the Harvard Negotiation Project lineage | ✅ | archive.org/details/difficultconvers0000ston (publication date 1999; publisher Viking; ISBN 0670883395; associated names Patton, Bruce; Heen, Sheila); en.wikipedia.org/wiki/Sheila_Heen ("(Penguin 2000)"; "member of the Harvard Negotiation Project"; "expands on the problem-solving approach set forth in Getting to Yes") |
| 21 | The "three conversations" structure (What Happened? / Feelings / Identity) and the contributions frame | ⚠ | The book's signature structure, held from the author's knowledge base; the book's own text was not extracted this pass (§5.2–§5.3 flag it) |
| 22 | Lazare, *On Apology* (Oxford University Press, 2004); the dignity-restoration thesis and the non-apology critique | ✅/⚠ | archive.org/details/onapology00laza (2004, OUP) — the *book and year* are ✅; the *thesis details* are ⚠ (held from the author's knowledge base; book not extracted) |
| 23 | Intragroup-conflict taxonomy: task, relationship, process | ✅ | APA Dictionary of Psychology (dictionary.apa.org, "intragroup conflict": process, relationship, task), via en.wikipedia.org/wiki/Intragroup_conflict |
| 24 | Jehn (1995), "A Multimethod Examination of the Benefits and Detriments of Intragroup Conflict", *Administrative Science Quarterly* 40(2):256–282, DOI 10.2307/2393638; the Intragroup Conflict Scale (task & relationship); Jehn, Northcraft & Neale (1999) ASQ 44(4):741–763; de Wit, Greer & Jehn (2012) meta-analysis, *Journal of Applied Psychology* 97(2):360–390, DOI 10.1037/a0024844 (task conflict weakly negative/contextually positive; relationship conflict consistently negative) | ✅ | Wikipedia *Intragroup conflict* references (full citations with DOIs, verified this pass) |
| 25 | Antecedents of intragroup conflict: low task/goal uncertainty, increased group size, increased diversity, lack of information sharing, high task interdependence | ✅ | Wikipedia *Intragroup conflict* (antecedents section, citing the primary studies) |
| 26 | Deutsch: *The Resolution of Conflict: Constructive and Destructive Processes* (Yale University Press, 1973); the constructive/destructive distinction; the "Crude Law"; theory of cooperation and competition (1949); founding father of conflict resolution; ICCCR founded 1986 | ✅ | en.wikipedia.org/wiki/Morton_Deutsch (career, books, and awards sections, verified this pass) |
| 27 | NVC: developed by Marshall Rosenberg in the 1960s–70s; four components (observation, feelings, needs, requests); heterogeneous evidence base | ✅ | en.wikipedia.org/wiki/Nonviolent_Communication (history and overview sections) |
| 28 | *Getting to Yes* on apology: letting the other side voice grievances and providing an apology as a symbolic gesture to defuse anger (pp. 33–35) | ✅ | Wikipedia *Getting to Yes* (emotion section, 3rd ed. page citations) |
| 29 | "Dealing with conflict early is usually easier..." | ✅ | skillsyouneed.com conflict-resolution page ("positions are not so entrenched, others are less likely to have started to take sides, and the negative emotions are not so extreme") |
| 30 | Cymbal Bank worked example: cast, dates, votes, ADR | ⚠ | Pedagogical construction consistent with the repository's Cymbal Bank persona (§11); not facts about any real bank |

---

## 13. What Could Not Be Verified

The honesty section — the items this pass could not pin down, and exactly why:

- **The exact coinage of the "Interest-Based Relational" label.** The *attribution* of IBR to Fisher & Ury's 1981 *Getting to Yes* is consistently stated across practitioner sources (PMI Global Leadership Community / Mind Tools, Lifehack, university conflict-resolution courses), and the six steps were verified verbatim from the PMI document (audit #7–#9). But the label's origin — who first named the "Interest-Based Relational approach" as such — could not be traced to a single primary source this pass, so the guide presents the attribution with ⚠ on the label and ✅ on the lineage.
- **The verbatim structure of *Difficult Conversations*.** The book's existence, authors, publisher, and 1999 date are fully verified (audit #20). Its signature "three conversations" structure (What Happened? / Feelings / Identity) and the contributions frame are presented from the author's knowledge base (audit #21) — the book's own text was not extracted this pass, so those details are ⚠ rather than ✅. The same applies to the finer detail of Lazare's *On Apology* thesis (audit #22): the book and year are verified; the dignity argument is held from knowledge.
- **The TKI per-mode situational guidance.** The five modes, the axes, the instrument mechanics, and the 1974 origin are all ✅ (audit #1–#6). The "when to use each mode" lists (§2.3) are the standard interpretive reading of the model — consistent with the co-author's materials — but the publisher's interpretive manual was not extracted page-by-page this pass (audit #15).
- **The fourfold sources of conflict (priorities, resources, styles, values).** This is the classic training-room taxonomy (audit #18); the research-backed *antecedents* list from the intragroup-conflict literature (audit #25) is the verified counterpart, and the guide says which is which.
- **The attribution of the interpersonal-conflict definition.** The definition quoted in §1.1 is widely attributed to Hocker & Wilmot's *Interpersonal Conflict*, but the attribution came from the quoting site (SkillsYouNeed) without inline citation and was not re-verified at the source textbook this pass (audit #19).
- **The practitioner skill mechanics.** The emotional-regulation techniques of §7.1 ("name it to tame it", the physiological pause, the 24-hour rule), the DESC script, and "I" statements (audit #16–#17) are practitioner-standard material; the *anchors* they hang on (Getting to Yes's emotion guidance ✅, NVC ✅, SkillsYouNeed's documentation ✅) are verified, the mechanics themselves are flagged.
- **The worked-example details.** The Cymbal Bank payment-hub conflict — its cast, its votes, its ADR — is explicitly pedagogical (§11, audit #30). Nothing in it describes a real bank's session; Cymbal Bank is this repository's recurring fictional persona, and the ADR it produces follows the ADR guide's conventions.

---

## 14. The Glossary

| Term | Definition |
|---|---|
| **Conflict** | An expressed struggle between interdependent parties who perceive incompatible goals, scarce resources, and interference (§1.1; ⚠ attribution) |
| **Task conflict** | Disagreement about the work itself: goals, content, substance — potentially valuable when kept in the task channel (§1.2, Jehn 1995 ✅) |
| **Relationship conflict** | Disagreement that has become personal: personality, mistrust, animosity — consistently harmful (§1.2, de Wit/Greer/Jehn 2012 ✅) |
| **Process conflict** | Disagreement about how the work gets done: roles, ownership, sequencing (§1.2, APA taxonomy ✅) |
| **Constructive / destructive conflict** | Deutsch's distinction: conflict resolved through cooperative process (constructive) vs competitive process (destructive); each spiral reinforces itself (§1.4, Deutsch 1973 ✅) |
| **TKI** | Thomas–Kilmann Conflict Mode Instrument: the 1974 forced-choice inventory measuring five conflict-handling modes on the assertiveness/cooperativeness axes (§2, ✅) |
| **Competing / Collaborating / Compromising / Avoiding / Accommodating** | The five TKI modes: high assertiveness–low cooperativeness; high–high; medium–medium; low–low; low assertiveness–high cooperativeness (§2.2, ✅) |
| **IBR (Interest-Based Relational approach)** | The Fisher–Ury-derived approach to resolving workplace conflict by keeping relationships primary, separating people from problems, and exploring interests and options jointly (§3, attribution ✅/label ⚠) |
| **Principled negotiation** | Fisher & Ury's method from *Getting to Yes* (1981 ✅): decide issues on their merits via the four principles, not by haggling over positions (§4) |
| **BATNA** | Best Alternative To a Negotiated Agreement — your walk-away; the source of negotiating power (§4.3, ✅) |
| **The three conversations** | Stone, Patton & Heen's structure of every difficult conversation: What Happened?, Feelings, Identity (§5.2, ⚠) |
| **Contributions frame** | The difficult-conversations shift from blame to "how did each of us contribute?" (§5.3, ⚠) |
| **Active listening** | Rogers & Farson's receiving-plus-reflecting practice; fully derived in the facilitation guide §2 (cross-ref) |
| **Assertiveness** | Stating your view, needs, and limits clearly and firmly — without aggression or capitulation (§6.2) |
| **NVC (Nonviolent Communication)** | Rosenberg's four-component verbal discipline: observation, feelings, needs, requests (§6.3, ✅) |
| **DESC script** | Describe, Express, Specify, Consequences — the four-move assertive request (§6.2, ⚠ practitioner) |
| **Reframing** | Changing the frame around a statement without changing its facts: positions → interests, objection → criterion (§7.2) |
| **Repair conversation** | The post-conflict apology that restores dignity: name the act, acknowledge the impact, state the change, wait (§7.3; Lazare 2004 ✅, anatomy ⚠) |
| **Mediation** | Working *between* conflicting parties toward an agreement they both own; distinct from facilitation, arbitration, and chairing (§8.1) |
| **Disagree and commit** | The exit convention: record the dissent, commit to the decision, hold the dissent as a monitored risk (§8.2, ⚠ unattributed; cross-ref facilitation guide §6.3) |
| **ADR (Architecture Decision Record)** | The repository's decision-capture unit — Context / Decision / Consequences; the artifact a resolved conflict should produce; conventions live in `../technology/architecture_decision_record_guide.md` (§9.4, cross-ref) |
| **Escalation ladder** | The ordered rungs of conflict escalation: direct conversation → manager/lead → neutral third party → governance body → executive (§10.3) |
| **Architecture Review Board (ARB)** | The governance body that is the escalation terminus for design conflict, with the decision packaged and captured as an ADR (§10.3, cross-ref ADR guide §9) |

### 14.1 A Companion Reading Path

The division of labour across the library is deliberate: this guide owns the conflict models, the resolution frameworks, and the conversation craft; the siblings own the mechanics this guide deliberately does not re-derive. When a conflict lands on your desk, pull the thread that fits the moment:

| When the conflict is about... | Reach for | What it contributes |
|---|---|---|
| Listening, questioning, or the state of the room | [Facilitation Skills](../management/facilitation_skills_guide.md) | The listening stance (§2 there), the questioning toolkit, the hats, and the difficult-situations playbook |
| Stakeholders, influence, and the politics of the decision | [Communication & Stakeholder Management Skills](../management/communication_stakeholder_management_skills_guide.md) | The stakeholder and influencing layer every difficult conversation runs on |
| Designing the session that will settle the conflict | [Workshop Methodology](../management/workshop_methodology_guide.md) | The agenda design and process mechanics for the workshops that resolve disputes |
| The vendor-side commercial layer of the conflict | [Vendor Management](../management/vendor_management_guide.md) | The vendor dispute and commercial-relationship frame |
| Recording the resolution so it survives the room | [Architecture Decision Records](../technology/architecture_decision_record_guide.md) | The Context / Decision / Consequences capture conventions for the outcome |
| The manager's role in escalation, 1:1s, and delegation | [The Manager's Path](../management/the_managers_path_guide.md) | The management frame this guide's escalation paths (§10.3) assume |
| Organisational disputes that need structured analysis | [Management Consulting Skills](../management/management_consulting_skills_guide.md) | Structured problem-solving for organisation-level conflict |
| Performance, feedback, and the managerial effectiveness frame | [3D Managerial Effectiveness](../management/3d_managerial_effectiveness_guide.md) | The results–relationships–leadership lens on conflict outcomes |

The pattern across the library is the same: the models are learned, the skills are practised, and the decisions are recorded. Read this guide for the what and the why of conflict; read the sibling for the how of the particular instrument the moment calls for.

---

*End of guide. Companion guides: [Communication & Stakeholder Management Skills](../management/communication_stakeholder_management_skills_guide.md) (the stakeholder and influencing layer), [Facilitation Skills](../management/facilitation_skills_guide.md) (listening, questioning, the hats, and the difficult-situations playbook), [Workshop Methodology](../management/workshop_methodology_guide.md) (the process mechanics for the sessions that settle conflicts), [Vendor Management](../management/vendor_management_guide.md) (the vendor-side commercial layer), [Architecture Decision Records](../technology/architecture_decision_record_guide.md) (how resolutions get recorded), [The Manager's Path](../management/the_managers_path_guide.md) and [3D Managerial Effectiveness](../management/3d_managerial_effectiveness_guide.md) (the management frame), and [Management Consulting Skills](../management/management_consulting_skills_guide.md) (structured problem-solving for organisational disputes). The models are learned, the skills are practised, the decisions are recorded — and the conflict, handled well, leaves the room with nothing but a better answer and a stronger team, and that is the working peace.*
