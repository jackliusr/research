# The DRI: The Directly Responsible Individual — Naming One Owner for One Outcome

*A deep dive into the DRI — the directly responsible individual: the practice of attaching exactly one human name to an outcome, a decision, a document or an action item. The guide covers the definition and what a DRI is not (not necessarily the decider, the doer or the sole participant), the provenance of the term and the correction of the "Apple invented it" folklore, the neighbouring concepts it is routinely confused with (RACI's A and R, DACI, Amazon's single-threaded leader, the Scrum Product Owner, the incident commander, CODEOWNERS, the plain line manager), the behavioural research underneath it (diffusion of responsibility, social loafing, accountability, goal-setting) and an honest account of how far that research actually carries the practice, the mechanics of running a DRI model day to day, the anti-patterns (accountability without authority above all), the variants in the wild, and — the section that matters most to this repository — the DRI's formalisation as **individual accountability regulation** in banking: UK SM&CR, MAS IAC, Hong Kong MIC, the Australian FAR and Ireland's SEAR. It closes with a fictional Cymbal Bank worked example, a claims audit, and an explicit list of what could not be verified.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Management / Leadership (management/) — the ACCOUNTABILITY-OWNERSHIP deep-dive; the guide that **owns the definition of the DRI as a repo-standard term** and the practice that goes with it, sitting beside the delegation chapter in [the_managers_path_guide.md](the_managers_path_guide.md), the team-accountability component in [high_performing_team_first_time_manager_guide.md](high_performing_team_first_time_manager_guide.md) §12.2, the decision-rights machinery in [authority_skills_guide.md](authority_skills_guide.md) §6, and the regulatory comparison that **[../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §4.5** owns for MAS in detail.
> **Audience:** Engineering and delivery managers, programme and change leads, tech leads and architects who must make ownership legible; risk, compliance and governance professionals mapping internal ownership to regulatory accountability; and anyone in the repository who needs the canonical, sourced answer to "what is a DRI, where did the term come from, and does the research actually support it?"
> **Last Updated:** September 2026

**Cross-references (the repo guides this deep-dive maps into):** This guide is the **canonical definition** of the DRI for the repository. Two sibling guides already reference the term and now point *here* for its definition rather than restating it: [the_managers_path_guide.md](the_managers_path_guide.md) lists **DRI — directly responsible individual: the named owner of an outcome; a repo-standard term for the accountability practice the book describes as delegation with authority** in its glossary and carries the delegation chapter (§12.6); [high_performing_team_first_time_manager_guide.md](high_performing_team_first_time_manager_guide.md) §12.2 carries "the DRI habit" as **one bullet** in a five-component team-accountability list alongside the ownership map (RACI), public commitments, the accountability conversation and — that guide's own contribution — the standard-not-the-person framing. Neither of those guides defines the DRI; this one does, and they should be read as applications of it. On the regulatory side, **[../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §4.5 owns the MAS Individual Accountability and Conduct (IAC) detail** — this guide **compares** the regimes and does not re-derive the MAS treatment. Operational, risk and change content is cross-referenced where §9 needs it: [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md). Other `management/` siblings cited by plain filename: [organizational_behavior_guide.md](organizational_behavior_guide.md), [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md), [product_management_frameworks_guide.md](product_management_frameworks_guide.md), [the_first_90_days_guide.md](the_first_90_days_guide.md), [management_case_study_guide.md](management_case_study_guide.md).

**How to read this guide (the integrity convention):** ✅ = verified **this pass** against a source that was actually fetched (a regulator's own page, a primary study record, a publisher page, or the article that first documented the term); ⚠ = flagged — contested, single-sourced, commercially interested, practitioner-origin or approximate, stated in place rather than smoothed over; ⚠-knowledge = well-documented standard knowledge that was **not** re-verified against a primary source this pass; ❌ = could not verify, or sources conflict. Research citations in §4 are given at DOI level and were confirmed against the Crossref metadata record — that confirms the *bibliographic existence and details* of the paper, not that this guide's paraphrase of it is beyond dispute. The audit in §11 collects every significant claim; the §12 section lists what resisted verification honestly. No citation, date or quotation in this guide was invented; where the record is thin, the guide says so in the sentence itself.

---

## Table of Contents

*(Sub-sections are listed inline for each section; every heading below is anchor-linked.)*

1. [The Overview and the Definition — One Name for One Outcome](#1-the-overview-and-the-definition--one-name-for-one-outcome) — 1.1 The Short Answer · 1.2 What a DRI Is Not · 1.3 Why a Single Name Changes Behaviour · 1.4 The Repo-Standard Term · 1.5 The One-Page Orientation
2. [The Provenance — Where the Term Comes From](#2-the-provenance--where-the-term-comes-from) — 2.1 The Earliest Widely-Cited Documentation (2011) · 2.2 The Evidential Weight · 2.3 The Book (2012) and the Derivative Literature · 2.4 The Dated Timeline of Documented Usage · 2.5 The "Apple Invented It" Correction · 2.6 The Pre-2011 Search
3. [The Concept and Its Neighbours — The Disambiguation Table](#3-the-concept-and-its-neighbours--the-disambiguation-table) — 3.1 The Confusion, Stated Plainly · 3.2 Accountability versus Responsibility (RACI's A and R) · 3.3 RACI's Own Origin — Flagged · 3.4 DACI · 3.5 The Amazon Single-Threaded Leader · 3.6 The Scrum Product Owner · 3.7 The Incident Commander · 3.8 CODEOWNERS and the Maintainer Model · 3.9 Sponsor versus Owner versus Line Manager · 3.10 The Disambiguation Table
4. [The Research Underneath It](#4-the-research-underneath-it) — 4.1 What the Research Is Being Asked to Do · 4.2 Diffusion of Responsibility · 4.3 Social Loafing and the Ringelmann Attribution · 4.4 Accountability Research — The Two Modes · 4.5 Goal-Setting · 4.6 How Far the Evidence Actually Carries the Practice · 4.7 The Mis-Citation Corrections
5. [The Mechanics — How to Actually Run It](#5-the-mechanics--how-to-actually-run-it) — 5.1 Where the Name Goes · 5.2 The Mandate · 5.3 The Decision-Rights Question · 5.4 The Escalation Path · 5.5 Handoff and Reassignment · 5.6 The DRI in a Meeting versus in a Written Plan · 5.7 Incident DRI versus Quarter-Long Outcome DRI · 5.8 The Line Manager and the Matrix Tension · 5.9 The DRI Charter — A Constructed Template
6. [The Anti-Patterns](#6-the-anti-patterns) — 6.1 Accountability Without Authority · 6.2 The Over-Assigned DRI (the Bottleneck) · 6.3 DRI in Name Only · 6.4 The Multiple-DRI Fiction · 6.5 The DRI as a Blame Instrument · 6.6 The Absent DRI · 6.7 The Cultural Precondition Question
7. [The Variants in the Wild](#7-the-variants-in-the-wild) — 7.1 The Amazon Single-Threaded Leader · 7.2 The Design-Doc Owner · 7.3 The Incident-Command Model · 7.4 The Open-Source Maintainer and the Ownership File · 7.5 Commander's Intent — The One That Is Not a DRI
8. [The Regulatory Formalisation — Individual Accountability in Banking](#8-the-regulatory-formalisation--individual-accountability-in-banking) — 8.1 Why a Regulator Would Want a DRI · 8.2 United Kingdom — SM&CR · 8.3 Singapore — MAS IAC · 8.4 Hong Kong — MIC · 8.5 Australia — FAR and Its BEAR Predecessor · 8.6 Ireland — SEAR and the IAF · 8.7 The Comparison Table · 8.8 The Boundaries
9. [The DRI in a Regulated Institution](#9-the-dri-in-a-regulated-institution) — 9.1 The Reconciliation Problem · 9.2 The Evidence Trail · 9.3 Outsourcing and Third-Party DRIs · 9.4 The Change-Management DRI · 9.5 The Audit and Supervisory Expectation · 9.6 The Conduct Dimension the Regimes Are Really Aimed At
10. [The Cymbal Bank Worked Example](#10-the-cymbal-bank-worked-example) — 10.1 The Programme and the Decomposition · 10.2 Naming the DRIs · 10.3 The Charters and the Decision-Rights Matrix · 10.4 Mapping Internal DRIs to Regulatory Accountable Roles · 10.5 The Anti-Pattern Guardrails · 10.6 How the Model Is Measured · 10.7 What the Model Cannot Fix
11. [The Claims Audit](#11-the-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Cross-references and Further Reading](#14-cross-references-and-further-reading)
15. [Closing Summary](#15-closing-summary)

---

## 1. The Overview and the Definition — One Name for One Outcome

### 1.1 The Short Answer

A **DRI — directly responsible individual** — is **the one person whose name is attached to a single, specific outcome, decision or deliverable, and who answers for it**. The term is used in this repository as a **repo-standard term** in exactly this sense: the named owner of an outcome. Three properties define it, and all three are load-bearing:

1. **One outcome.** A DRI owns something nameable and bounded — a deliverable, a decision, a risk, a document, an incident, a released change — not a vague area of concern.
2. **One name.** A single human being, not a team, not a function, not a committee, not "the payments team". The name is written down where the work is discussed.
3. **Answerability.** The DRI is the person who will be asked, "where is this?", and who is expected to have an answer. Answerability is the whole point; without it, the name is decoration.

What a DRI is *for* is narrower than most retellings suggest, and that narrowness is the interesting part. The DRI does not create capacity, authority, or expertise. It removes a specific failure mode: the state in which everyone is vaguely aware that something matters and nobody is answerable for it. The DRI is a **search-and-retrieval mechanism for responsibility** as much as a governance device — at Apple, per the source in §2, the phrase was used as a question: *"Who's the DRI on that?"* — that is, *who do I talk to about this?* The question is as important as the answer.

### 1.2 What a DRI Is Not

The most common errors in practice all come from collapsing the DRI into neighbouring roles. The corrections:

- **Not necessarily the decider.** For many outcomes the DRI is the person who drives the work and owns the result, while a formal decision right sits elsewhere (a change board, an executive sponsor, an approving regulator). The DRI owns the *outcome*; the decider owns the *decision*. §5.3 gives the rule for when one person should hold both.
- **Not necessarily the doer.** The DRI may do none of the hands-on work. Naming the DRI for a quarter-long platform migration does not mean they write all the code or run all the tests; it means they assemble the plan, drive it, escalate when it stalls, and answer for the result. The DRI is an accountability assignment, not a work assignment.
- **Not the sole participant.** A DRI with no team is a bottleneck; a DRI *over* a team is the normal case. The practice is sometimes misread as "one person does the thing", which is the anti-pattern in §6.2, not the practice.
- **Not a second reporting line.** The DRI is not a manager. The line manager owns the person's development, performance and workload; the DRI owns an outcome. Where these two land on different humans, §5.8 explains how to keep the model from becoming a shadow org chart.
- **Not permanent.** DRIs are attached to outcomes, and outcomes end. A stale DRI roster — names against things that shipped two quarters ago — is a maintenance failure, not a design.

### 1.3 Why a Single Name Changes Behaviour

Four mechanisms, in descending order of how well the evidence supports them:

- **It removes the diffused-responsibility default.** When responsibility is shared across a group, individuals feel less personally answerable; when one person is named, the question "whose is this?" has an answer (§4.2).
- **It makes the goal specific and attributable.** Goal-setting research finds that specific, difficult goals with feedback outperform vague "do your best" — and a named owner is the mechanism by which a plan's goals become attributable to a human (§4.5).
- **It makes the coordination cost visible.** The DRI is a single entry point: instead of a search through three teams to find who knows about a system, one name is published.
- **It changes what happens in the room.** A written name converts an intention into a commitment. This last mechanism is the least researched and the most claimed — see §4.6, where the guide is explicit that **no study tests the meeting ritual**.

### 1.4 The Repo-Standard Term

This guide is the canonical definition other repository guides point *to*:

- [the_managers_path_guide.md](the_managers_path_guide.md) lists in its glossary — verbatim — **"DRI — directly responsible individual: the named owner of an outcome; a repo-standard term for the accountability practice the book describes as delegation with authority (Ch.6; §12.6)"** — and its delegation chapter (§12.6, *Delegation Is Not Dumping*) is the applied treatment of handing an outcome to a named owner with the authority to carry it. This guide provides the definition that entry assumes.
- [high_performing_team_first_time_manager_guide.md](high_performing_team_first_time_manager_guide.md) §12.2 carries **"The DRI habit. One named directly-responsible individual per outcome (cross-ref [the_managers_path_guide.md](the_managers_path_guide.md)'s delegation chapter) kills the diffusion of responsibility in handoffs."** — one bullet among five, in a section whose other four components (the RACI ownership map, public commitments, the accountability conversation about the standard not the person, and the manager as the vertical accountability) are each distinct from the DRI and each weaker or stronger for different reasons. That guide treats the DRI as **one instrument**; this guide treats it as **the subject**.

The relationship is worth stating plainly because it is a common failure of large documentation sets: two guides citing a term in opposite directions, each assuming the other defines it. Here, the definition lives once.

### 1.5 The One-Page Orientation

For the reader who needs the whole practice in ten lines:

| Question | Short answer | Detail |
|---|---|---|
| What is a DRI? | The one named person answerable for a specific outcome | §1.1 |
| Where does the term come from? | Documented in print for Apple's internal convention in a 2011 Fortune article; Apple popularised it, it did not originate the idea of single ownership | §2 |
| Is the research behind it solid? | The *principle* of named accountability is well supported; the *rituals* the practice recommends are not separately tested | §4.6 |
| Who decides, if not the DRI? | Named separately as a decision right, unless the charter grants both | §5.3 |
| What is the biggest failure mode? | Accountability without authority — naming someone answerable for an outcome they cannot resource, staff or decide | §6.1 |
| Is this just RACI? | No — RACI is a per-row assignment grid; a DRI is one name on one outcome, and RACI's "A" is the closest neighbour | §3.2 |
| Is it regulated? | In banking, the *idea* is now formalised as statutory individual accountability in the UK, Singapore, Hong Kong, Australia and Ireland | §8 |
| Is the internal DRI the regulatory accountable person? | Not always — and mapping them is the reconciliation problem | §9.1 |

---

## 2. The Provenance — Where the Term Comes From

### 2.1 The Earliest Widely-Cited Documentation (2011)

The earliest widely-cited documentation of the term **DRI — directly responsible individual** is Adam Lashinsky's article **"How Apple works: Inside the world's biggest startup"**, published by **Fortune on 25 August 2011** ✅. The article describes Apple's weekly meeting discipline and then, in a passage on accountability, writes of Apple's internal vocabulary:

> "The accountability mindset extends down the ranks. **At Apple there is never any confusion as to who is responsible for what. Internal Applespeak even has a name for it, the 'DRI,' or directly responsible individual.** Often the DRI's name will appear on an agenda for a meeting, so everybody knows who is responsible."

and continues with a quotation from an unnamed former employee:

> "'Any effective meeting at Apple will have an action list,' says a former employee. '**Next to each action item will be the DRI.**' A common phrase heard around Apple when someone is trying to learn the right contact on a project: '**Who's the DRI on that?**'"

Those three fragments — the term itself, the agenda practice, and the question — are the documented core. Everything else in circulation is either a later restatement of this passage or an unverified embellishment.

### 2.2 The Evidential Weight — What the Source Actually Is

It matters a great deal that this is a **journalist's account of an internal folk practice**, and the guide will not let that weight be upgraded by repetition:

| Property | The honest reading |
|---|---|
| Source type | **Secondary journalism** — a trade/press journalist writing about a famously secretive company, on the basis of interviews |
| Attribution | **Anonymous former employees** — "says a former employee", "a former employee who now runs…" No named on-the-record Apple source for the DRI passage itself |
| Apple's own documentation | **None presented.** Apple did not publish a DRI policy; there is no Apple corporate document, handbook page, or press release in the record that defines the term |
| Executive attribution | **No Steve Jobs quotation on the DRI is documented in this article.** The Jobs quotations in the piece concern meeting agendas and product review — a related but different subject |
| Independent corroboration | The same author expanded the material into a book (below); Apple veterans have described the practice in their own words (§2.3), which raises the *practice's* plausibility considerably while adding nothing to the *term's* documented provenance |
| What it establishes | That **Apple had an internal convention called the DRI, first documented in print in 2011**, and that Apple **popularised** the term |
| What it does not establish | That Apple invented the practice or the idea of single-named ownership — see §2.5 and §2.6 |

### 2.3 The Book (2012) and the Derivative Literature

- **Adam Lashinsky, *Inside Apple: How America's Most Admired — and Secretive — Company Really Works* (Business Plus, 2012)** ✅ — the same author expanding the same reporting; this is where the DRI material reaches book length and from which most later citations ultimately derive. Treat it as **secondary, single-authored journalism**, and note that the book and article are not independent sources — they are one source in two formats.
- **Practitioner and business-press books that repeat the DRI story** — including titles such as Carmine Gallo's *The Apple Experience* (2012) and Ken Segall's *Insanely Simple* (2012) — are **derivative literature**: they are written by people with varying degrees of first-hand Apple exposure, they cite no primary Apple document, and in most cases they are downstream of the same 2011–2012 Fortune reporting. ⚠ **Treat every such citation as secondary until proven first-hand**, and treat the fact that "many books say it" as no additional evidence whatsoever.
- **Named first-hand accounts by Apple veterans** would be the strongest available upgrade to the record. This guide's search for them is reported honestly in §12 — where one exists, it belongs in the timeline; where the record is thin, the guide says so rather than padding the section with confidence it does not have.

### 2.4 The Dated Timeline of Documented Usage

Dates, sources, and — critically — the **quality of each source**:

| Date | Source | What it documents | Quality |
|---|---|---|---|
| Pre-2011 | **No documented use of the term "DRI" located** | The search is reported in §2.6 and §12 | ❌ Not found |
| Pre-2011 | Older project-management, engineering and military practice | **Single-point-of-accountability** and "one name on the plan" as a general discipline | ⚠-knowledge — the *practice* is old; a *dated citation* for the phrase is not offered here |
| **25 Aug 2011** | Lashinsky, Fortune | Apple's "Internal Applespeak" term **DRI**, the agenda practice, and the phrase "Who's the DRI on that?" | ✅ **Secondary journalism on anonymous sourcing** — the anchor source |
| 2012 | Lashinsky, *Inside Apple* (Business Plus) | The same material at book length | ⚠ Secondary; **same author, same reporting** — not independent |
| 2012 | Practitioner books repeating the story | The DRI story in the business-press canon | ⚠ Derivative |
| 2011–present | Tech, product and engineering-management writing | DRI as a generic practice noun | ⚠ Derivative of the 2011/2012 source |
| 2016 onward | UK SM&CR and successors (§8) | **Statutory** named individual accountability — the same *idea*, arrived at independently of the term | ✅ Regulator primary sources |

The timeline's honest shape: **one documented primary-adjacent source (2011), one book by the same author (2012), and a long derivative tail.** The regulatory column is a genuine independent arrival at the same principle — and it is the reason this section is short and the regulatory section is long.

### 2.5 The "Apple Invented It" Correction

Four claims circulate that the evidence does not support. They are corrected here, explicitly, because the repository's guides are used as reference material and a confident false provenance spreads faster than a careful true one.

1. **"Apple invented the DRI."** **Rejected as stated.** The verified claim is narrower: **Apple had an internal convention called the DRI, first documented in print in 2011, and Apple popularised the term.** The practice of attaching one name to one outcome long predates Apple and appears in older project-management, engineering and military practice. The correct formulation — used throughout this guide — is **Apple-popularised, not Apple-originated, on the evidence available**.
2. **"Steve Jobs invented the DRI" / "Steve Jobs said the DRI."** **Rejected as unsupported.** No primary Jobs quotation defining or using the DRI is documented in the source that introduced the term. The Jobs quotations in that article are about **weekly review meetings and agendas**, which is why the two get conflated: the DRI is described as appearing *on* the agendas Jobs ran, which is not the same as Jobs coining the term.
3. **Disney / Walt Disney "DRI" attributions.** **Rejected as unsourced.** Circulating social-media and training-deck claims that the DRI originates with Disney management practice carry no dated documentation. Do not repeat them. Related organisational practices sometimes attributed to Disney (for instance the cast-member terminology) are separately documented facts and are not evidence for a Disney DRI.
4. **"The DRI came from a specific named document or consultant."** **Rejected as unsourced.** No such document has been produced. Where a named origin story appears, the tell is usually that it cites no page, no date and no archive.

### 2.6 The Pre-2011 Search — What Was Looked For

An honest negative result requires stating the shape of the search. The guide looked for earlier dated uses of the term in four places:

- **Books and the printed record** — for a pre-2011 use of "directly responsible individual" as a defined term of art, distinct from ordinary English usage of the words.
- **Archive and digitised-document collections** — for engineering, aerospace, NASA-adjacent or military documents using the phrase as a named role.
- **The Linux kernel `MAINTAINERS` file and its history** — the nearest analogue: a maintained, versioned file mapping subsystems to named humans. What it *demonstrates* is that **named-ownership files are an older and independently evolved practice in open source** (§7.4); what it does **not** provide is an earlier attestation of the *term*.
- **Software-engineering and management literature** — for the term used generically before Apple.

The ordinary-English words "directly responsible individual" occur in many older documents without carrying the Apple sense (a role-description phrase, not a proper noun). Distinguishing those from a *named convention* is the whole difficulty of the search, and it is why the guide reports **no earlier dated attestation located**, rather than either asserting Apple's coinage or manufacturing a predecessor. See §12.

---

## 3. The Concept and Its Neighbours — The Disambiguation Table

### 3.1 The Confusion, Stated Plainly

The DRI sits in a crowded field of ownership frameworks, and the practical damage is always the same: a team adopts two of them, uses the vocabulary interchangeably, and ends up with **two names on one outcome and no decision rule**. This section separates the neighbours, states what each one actually is, and — where the attribution is asserted without evidence — flags that too.

The one-line distinction that does most of the work: **a DRI names a person against an outcome; the other frameworks name roles against a decision or a row of work.**

### 3.2 Accountability versus Responsibility (RACI's A and R)

RACI — *Responsible, Accountable, Consulted, Informed* — is a **responsibility assignment matrix**: a grid with tasks or deliverables down one axis and roles across the other, each cell carrying one or more of the four letters ✅ *(fetched: the Wikipedia article on the responsibility assignment matrix, which carries the PMI citations)*. The two letters that matter here:

- **R — Responsible:** "Those who complete the task… There is at least one role with a participation type of *responsible*, although others can be delegated to assist in the work required." Note the plural tolerance — **R can be shared.**
- **A — Accountable** (also *approver* or *approving authority*): "The one ultimately answerable for the correct completion of the deliverable or task, ensuring the prerequisites of the task are met, and delegating the work to those *responsible*." And on the cardinality: "According to some theories of project management, there must be only one accountable stakeholder specified for each task or deliverable." ✅

So **RACI's "A" is the closest thing in the canon to a DRI** — single, answerable, sign-off authority. Three differences remain, and each one bites in practice:

1. **Scope.** A DRI attaches to *one outcome* — often the whole of a programme's worth of work. RACI's A attaches to *one row of the grid*: one task, one deliverable. A DRI is a top-level assignment; a RACI A is a per-line assignment. They compose: a DRI's outcome typically decomposes into many RACI rows whose A's may themselves be DRIs for their own sub-outcomes.
2. **Form.** RACI is a **matrix** — it can express C and I, i.e. it is about consultation and information flow as well as ownership. A DRI is a **single name in a document** and says nothing about who is consulted.
3. **Cardinality discipline.** RACI *permits* sharing on R and is only "according to some theories" single on A — a hedge the DRI practice does not make. A DRI has exactly one name by definition; anything else is not a DRI (§6.4).

### 3.3 RACI's Own Origin — Flagged

⚠ **RACI is routinely attributed to a specific origin that this pass could not verify.** The widely available tertiary summary fetched here names **no originator** at all: it cites the **Project Management Institute's *PMBOK Guide*** and an IIBA **BABOK** reference for the *definition*, and a **PMI chapter presentation** ("Role & Responsibility Charting (RACI)", Smith & Erwin) for the role descriptions — but it offers no first-use attribution and no date of coinage. Practitioner sources commonly assert that RACI came from a particular consultancy, industry, or era; those assertions appear **without a page, a date or an archive**. Treat **"RACI was invented by X"** as **unverified folklore** unless a dated primary document is produced. The honest statement is: RACI is a broadly adopted project-management convention whose first publication is not established by the sources consulted here; the PMI/PMBOK lineage is the citable one.

### 3.4 DACI

**DACI — Driver, Approver, Contributors, Informed** — is a decision-roles framework. The version fetched this pass is Atlassian's team-playbook page, which defines it as follows ✅:

- **Driver** — "the person responsible for corralling stakeholders, collating all the necessary information, determining the scope of the decision, and getting a decision made by the agreed date."
- **Approver** — "**The one person (yes: one!)** who makes the decision. This turns the approver role from a passive 'rubber stamp' into a very active 'decision maker' role." ✅
- **Contributors** — "People who have subject-area knowledge and can make recommendations – i.e., they have a voice, but not a vote."
- **Informed** — "People whose work may be affected by the decision… no vote, no voice."

Two observations, one useful and one cautionary:

- **The DACI Approver is a deliberately single-seat decision right**, and the page's own parenthetical — "yes: one!" — is the same instinct that produced the DRI. But it is a decision right, not an outcome ownership: **DACI has no role for "who answers for the result six months later."** That is exactly the gap a DRI fills, which is why the two are complements rather than substitutes.
- ⚠ **The common attribution of DACI to Intuit is not verified here.** The page fetched this pass does not attribute the framework to Intuit or to any named originator, and no Intuit primary document was located. **Treat "DACI is Intuit's framework" as practitioner lore pending a dated source.** (Also note the page's McKinsey-derived claim that DACI projects have "a 25% higher success rate" — a **commercially interested, second-hand statistic**; it is not a controlled study and should not be repeated as one.)

### 3.5 The Amazon Single-Threaded Leader

Amazon's model is documented in **Colin Bryar & Bill Carr, *Working Backwards: Insights, Stories, and Secrets from Inside Amazon* (St. Martin's Press, 2021)** ⚠-knowledge *(the book's content is standard, widely-repeated knowledge; the fetched Crossref bibliographic query returned no matching record for the title, and no Amazon primary document was fetched this pass)*. Its distinguishing components:

- **The single-threaded leader (STL)** — one leader who works on one thing, with no competing responsibilities pulling their attention apart. The "single-threaded" describes **focus** as much as accountability.
- **The separable, single-threaded team** — a dedicated team that owns a service or product end to end, able to build and operate it without depending on other teams for key decisions (Amazon's own material frames the small-team sizing rule colloquially as the "two-pizza team" — ⚠-knowledge, popular shorthand rather than a verbatim policy).
- **The written narrative** — the six-page memo and the PR/FAQ, reviewed silently at the start of meetings.

**What it shares with a DRI:** one named human, one bounded thing, owned end to end. **What it does not:** the STL is an **organisational-design decision** — it changes reporting lines, team boundaries and funding so that a person *can* be single-threaded. A DRI is often assigned *within* an existing structure, which is precisely where the accountability-without-authority failure (§6.1) comes from. Amazon's model is the more expensive, more robust version: **it buys the authority before it names the owner.**

### 3.6 The Scrum Product Owner

The **2020 Scrum Guide** is explicit and quotable ✅ *(fetched: the scrumguides.org text)*:

> "The Product Owner is accountable for maximizing the value of the product resulting from the work of the Scrum Team."
> "The Product Owner is also accountable for effective Product Backlog management…"
> "**The Product Owner is one person, not a committee.** The Product Owner may represent the needs of many stakeholders in the Product Backlog. Those wanting to change the Product Backlog can do so by trying to convince the Product Owner."

That "one person, not a committee" sentence is the cleanest formal statement in mainstream methodology of the single-owner principle — and the Scrum Guide's mechanism for backing it is notable: **it is not enough to name one person; the guide adds that "for Product Owners to succeed, the entire organization must respect their decisions."** ✅ A named owner without respected decision rights is the anti-pattern this guide treats in §6.1, stated by the methodology itself.

**Difference from a DRI:** the Product Owner is a **permanent role on a persistent team**, with a defined artefact (the Product Backlog) and defined events (refinement, review). A DRI is **per-outcome and temporary** — it expires when the outcome is delivered. Adopting Scrum does not give you a DRI model, and a DRI model does not give you a Product Owner.

### 3.7 The Incident Commander

The incident-command model is the oldest and most operationally honest of the neighbours. Two lineages:

- **Incident Command System / NIMS** — the US emergency-management structure, standardised through **FEMA's National Incident Management System** ⚠-knowledge (the URL appears as a footnote reference in Google's SRE book chapter fetched this pass, which is corroboration of the reference, not of ICS content).
- **Google's SRE Book, chapter 14, "Managing Incidents"** ✅ *(fetched)*, which names the roles the model needs: the **incident commander**, the **Ops lead** ("the operations team should be the only group modifying the system during an incident"), the **communications lead** ("the public face of the incident response task force"), and **planning**.

Three details from that chapter are worth importing wholesale into any DRI model:

1. **The default catch-all:** "The incident commander holds the high-level state about the incident. They structure the incident response task force, assigning responsibilities according to need and priority. ***De facto*, the commander holds all positions that they have not delegated.**" ✅ — accountability defaults to one person and is *delegated away*, not distributed by default.
2. **The named living document:** "The incident commander's most important responsibility is to keep a living incident document… This living doc can be messy, but must be functional." ✅ — the DRI record writ small.
3. **The explicit handoff:** "It's essential that the post of incident commander be clearly handed off at the end of the working day… the outgoing commander should be explicit in their handoff, specifically stating, 'You're now the incident commander, okay?', and should not leave the call until receiving firm acknowledgment of handoff. The handoff should be communicated to others working on the incident so that it's clear who is leading." ✅ — this is the best published statement of the **handoff protocol** any DRI model needs (§5.5).

**Difference from a DRI:** the incident commander holds a **command** right (decisions, resource direction) for a **short, high-tempo bounded period**, with a formal structure of subordinate roles. A DRI may hold no decision right at all and may run for a quarter. The two are the same *idea* at different tempos — which is why the incident-command chapter is the best short text to hand a manager who has never run a DRI model.

### 3.8 CODEOWNERS and the Maintainer Model

- **GitHub CODEOWNERS** ✅ *(fetched: docs.github.com)* — a file that "define[s] individuals or teams that are responsible for code in a repository". Code owners are automatically requested for review on pull requests touching their paths; where required reviews are enabled, a code owner's approval can be **required before merge**. Note the divergence that matters most: **GitHub's mechanism explicitly permits a *team* as the code owner** — the tool sanctions exactly the plural ownership the DRI forbids, and the docs discuss the team case at length. CODEOWNERS is therefore a **routing and gate mechanism**, not a DRI implementation; to use it as one, the file must name people, not teams.
- **The Linux kernel `MAINTAINERS` file** ⚠-knowledge — the long-lived, version-controlled mapping of subsystems to named maintainers and reviewers, with entries for status, trees and mailing lists. It demonstrates that **named-ownership registries are an independently evolved practice in open source**, older than the DRI's documentation and designed for a different problem (routing patches to the right reviewer). What it lacks relative to a DRI charter is any answerability for *an outcome*: a maintainer owns a subsystem's acceptance stream, not its success or failure in a stated goal.

### 3.9 Sponsor versus Owner versus Line Manager

Three roles that get merged and should not be:

- **Sponsor** — funds and protects the work at a level above it; unblocks politically; owns the *decision* to continue or stop. A sponsor is **not** the DRI: a sponsor's attention is divided by definition across a portfolio.
- **Owner (DRI)** — answers for the outcome, drives the plan, escalates, reports. **The DRI is not necessarily the highest-ranking person on the work.**
- **Line manager** — owns the person: development, performance, workload, career. Almost never the same human as the DRI for a cross-cutting outcome, and should not be forced to be (§5.8).

### 3.10 The Disambiguation Table

| Concept | What it actually is | Origin (and evidence quality) | How it differs from a DRI |
|---|---|---|---|
| **DRI** | One named person answerable for one outcome; documented in a plan, agenda or runbook | Term documented in Fortune, 25 Aug 2011 ✅ (secondary, anonymous sourcing); Apple-popularised, not originated | — (the reference point) |
| **RACI — R** | The role(s) who do the work | PMI/BABOK lineage ✅; origin attributed variously and **unverified** ⚠ | R can be plural; a DRI may do none of the work |
| **RACI — A** | The one ultimately answerable per task row | Same ✅; single-A stated as "according to some theories" ⚠ | Nearest neighbour — but grid-scoped, not outcome-scoped; hedged on cardinality |
| **DACI — Approver** | The one person who makes the decision | Atlassian page fetched ✅ (attributes to no one); **Intuit attribution unverified** ⚠ | A decision right, not outcome answerability; expires with the decision |
| **Amazon single-threaded leader** | One leader, one thing, no competing priorities; organisational design | Bryar & Carr, *Working Backwards* (2021) ⚠-knowledge | Buys authority and focus *before* naming the owner; heavier and more durable |
| **Scrum Product Owner** | One person accountable for maximising product value and for the Product Backlog; "one person, not a committee" ✅ | Scrum Guide 2020 ✅ | Permanent role on a persistent team; a DRI is per-outcome and expires |
| **Incident commander** | Commands a response, holds all undelegated roles, keeps the living doc, hands off explicitly ✅ | ICS/NIMS ⚠-knowledge; SRE Book ch.14 ✅ | Holds a command right for hours; a DRI may hold no decision right for a quarter |
| **CODEOWNERS** | File mapping paths to owners who must review changes ✅ | GitHub docs ✅ | Offsets review *routing*; permits teams, so not a single-owner device |
| **Linux MAINTAINERS** | Registry of subsystem maintainers/reviewers ⚠-knowledge | Open-source practice ⚠-knowledge | Owns an acceptance stream, not a stated outcome |
| **Sponsor** | Funds, protects, decides continue/stop | Standard practice ⚠-knowledge | Divided attention by construction; not answerable for delivery |
| **Line manager** | Owns the person's development, performance, workload | Standard practice ⚠-knowledge | Owns a person, not an outcome; forcing the merge creates matrix conflict (§5.8) |

---

## 4. The Research Underneath It

### 4.1 What the Research Is Being Asked to Do

The DRI's defenders usually reach for a chain that runs: *groups diffuse responsibility → people free-ride in groups → therefore name one person*. The chain is real, but it is looser than the retelling. This section lays out the actual findings, gives each citation at DOI level, and then states — in §4.6 — precisely how far they carry the practice and where they stop.

Every citation below was confirmed against the **Crossref metadata record** this pass ✅ (title, authors, journal, volume, issue, pages, year). Crossref confirms **the existence and bibliographic details of the paper**. It does not certify that any particular paraphrase of it is uncontested; where a finding is contested, the guide says so in place.

### 4.2 Diffusion of Responsibility

The founding studies of **bystander intervention** established that the presence of others reduces the likelihood that any one person acts:

1. **Darley, J. M., & Latané, B. (1968). "Bystander intervention in emergencies: Diffusion of responsibility." *Journal of Personality and Social Psychology*, 8(4, Pt. 1), 377–383. doi:10.1037/h0025589** ✅ — the founding paper and the source of the phrase **diffusion of responsibility**.
2. **Latané, B., & Darley, J. M. (1968). "Group inhibition of bystander intervention in emergencies." *Journal of Personality and Social Psychology*, 10(3), 215–221. doi:10.1037/h0026570** ✅ — the companion paper. ⚠ **Note the journal: *JPSP*, not *Science*.** The mistake is common enough that the guide treats it as a mis-citation to correct (§4.7).
3. **Latané, B., & Rodin, J. (1969). "A lady in distress: Inhibiting effects of friends and strangers on bystander intervention." *Journal of Experimental Social Psychology*, 5(2), 189–202. doi:10.1016/0022-1031(69)90046-8** ✅ — the variant that varied whether the bystanders were friends or strangers.
4. **Latané, B. (1981). "The psychology of social impact." *American Psychologist*, 36(4), 343–356. doi:10.1037/0003-066X.36.4.343** ✅ — Latané's consolidation into a formal theory: the *social impact* of a source diminishes as the number of targets rises. This is the theoretical bridge from the bystander experiments to organisations: it predicts that **as the number of people nominally on the hook rises, each one's felt obligation falls**.

The organisational translation is honest but narrow: **an unnamed group genuinely is a weaker accountability structure than a named individual.** It is not a claim about anyone's moral character, and the research is about *felt obligation in the moment*, not about long-running work ownership.

### 4.3 Social Loafing and the Ringelmann Attribution

**Social loafing** is the finding that individuals exert less effort on a collective task when their individual contribution is not identifiable:

5. **Latané, B., Williams, K., & Harkins, S. (1979). "Many hands make light the work: The causes and consequences of social loafing." *Journal of Personality and Social Psychology*, 37(6), 822–832. doi:10.1037/0022-3514.37.6.822** ✅ — the paper that gave the phenomenon its name.
6. **Karau, S. J., & Williams, K. D. (1993). "Social loafing: A meta-analytic review and theoretical integration." *Journal of Personality and Social Psychology*, 65(4), 681–706. doi:10.1037/0022-3514.65.4.681** ✅ — the consolidation. The useful, non-obvious part of the meta-analysis for a practitioner is **moderator structure**: loafing is *reduced* when individual contributions are identifiable, when the task is meaningful, when the group is cohesive, and when outcomes matter to the person. **Identifiability is the lever a DRI actually pulls.** The practice does not make people try harder by magic; it makes their contribution attributable.

**The Ringelmann attribution fix — and the dispute.** The rope-pulling experiment is routinely cited as "Ringelmann, 1913" in a journal, and routinely interpreted as a demonstration of social loafing:

7. **Kravitz, D. A., & Martin, B. (1986). "Ringelmann rediscovered: The original article." *Journal of Personality and Social Psychology*, 50(5), 936–941. doi:10.1037/0022-3514.50.5.936** ✅ — the paper that **rediscovered, translated and described** the original French study for the English-language literature.

Three corrections follow, and the guide states all three because the loose version is almost universal:

- **The 1913 study was not published in a psychology journal.** It was a **French agricultural-engineering study by Max Ringelmann**, published in *Annales de l'Institut National Agronomique* ✅ *(as characterised by Kravitz & Martin's rediscovery)*. Anyone citing "Ringelmann (1913), *Journal of …*" is citing it **second-hand** — usually via Kravitz & Martin or via a textbook that itself went through them.
- **Ringelmann's own explanation was coordination loss, not motivation loss.** His interest was the mechanics of teams pulling a rope; the reading of the data as *social loafing* — a motivational deficit — is a **later overlay**, decades after the fact.
- **His data were long mis-reported.** Kravitz & Martin's point was precisely that the widely circulated figures and interpretation did not match the original article. ⚠ The guide therefore treats "Ringelmann proved social loafing in 1913" as a **folklore compression** of a real but different study.

### 4.4 Accountability Research — The Two Modes

The most important research finding for a DRI design is not that accountability works; it is that **accountability is not one thing**:

10. **Lerner, J. S., & Tetlock, P. E. (1999). "Accounting for the effects of accountability." *Psychological Bulletin*, 125(2), 255–275. doi:10.1037/0033-2909.125.2.255** ✅ — the anchor review. Its central distinction:

| Mode | What the decision-maker knows | Typical effect |
|---|---|---|
| **Pre-decisional accountability** | Who the audience is and what they want, **before** deciding | **Conformity to the audience's known views** — the decision-maker anticipates the judgement and adjusts toward it |
| **Post-decisional accountability** | Only that they must justify the decision **afterwards**, to an audience whose views are unknown | **More self-critical, more complex, more effortful** processing — the decision-maker prepares for challenge |

This is the nuance the DRI practice glosses over, and it is the reason this guide's §5.3 insists that **the decision right and the DRI role be named separately**. If the DRI is also the decider *and* the audience's preferences are known in advance, the research predicts conformity pressure, not better judgement — a named owner who has learned what the boss wants is not the same as a named owner who reasons carefully. Post-decisional accountability is the mode a good DRI charter creates: **"you will be asked, at the end, to explain the calls you made"**, with the justification made to people who have not pre-announced their view.

11. **Tetlock, P. E. (1983). "Accountability and the perseverance of first impressions." *Social Psychology Quarterly*, 46(4), 285. doi:10.2307/3033716** ✅
12. **Tetlock, P. E. (1985). "Accountability: A social check on the fundamental attribution error." *Social Psychology Quarterly*, 48(3), 227. doi:10.2307/3033683** ✅ — both are the pre-1999 experiments showing that *who* one expects to account to changes *how* one thinks, and that accountability can reduce attribution error. Read as a pair, they are the micro-foundation for the "named owner reports to a named forum" mechanic — the mechanic works by installing an anticipated audience, not by installing a punishment.

### 4.5 Goal-Setting — The Finding That Actually Underwrites the Practice

13. **Locke, E. A., & Latham, G. P. (2002). "Building a practically useful theory of goal setting and task motivation: A 35-year odyssey." *American Psychologist*, 57(9), 705–717. doi:10.1037/0003-066X.57.9.705** ✅ — the retrospective on the goal-setting literature.

The relevant finding, and the one that genuinely underwrites the practice: **specific and difficult goals, with feedback, produce higher performance than vague "do your best" goals** — and this is one of the most thoroughly replicated lines in organisational psychology, with the paper's own title advertising the **35-year** record. The DRI is, functionally, **the attributability half of that finding applied to a plan**: a goal with a name attached is a specific goal; a goal with "the team" attached retains the vagueness that the research says costs performance. ⚠ Two honest caveats the goal-setting literature itself raises: goal specificity can **narrow attention** and encourage gaming of the measured target, and extreme goals can increase risk-taking. A DRI model inherits both risks — §6.5 covers the blame variant. Nothing in this research concerns *meeting agendas*.

### 4.6 How Far the Evidence Actually Carries the Practice

This is the section that keeps the guide honest, and it is deliberately blunt.

**What the evidence supports:**

- That **unattributed collective responsibility weakens felt individual obligation**, both in emergencies (bystander studies) and in effort tasks (social loafing, with identifiability as the moderator).
- That **accountability changes cognitive processing**, and that the *timing and audience* of accountability determine whether the change is conformity or better reasoning.
- That **specific goals with feedback outperform vague ones**, over a very long replication record.
- That **organisational single-owner models have independently converged on one named person** in incident command, in Scrum's Product Owner, and in Amazon's single-threaded leader — convergent practice, not experimental evidence.

**What the evidence does not support — and this is the part the practice never says out loud:**

- **No study tests the meeting ritual.** No experiment examined "write a person's name next to the action item in the agenda and everyone knows who owns it" — and the popular claim that the DRI *caused* Apple's execution quality is not a study at all; it is a **journalist's observation of a company that was, at the time, also the most valuable in the world for many other reasons.** Correlation with a famously successful company is not evidence of mechanism.
- **No study tests a DRI charter, a responsibility map, or a decision-rights matrix** as organisational interventions. The regulatory regimes in §8 are natural experiments in *mandating* named accountability, and their evaluations are regulatory reviews, not controlled trials.
- **The bystander findings are about momentary helping behaviour**, not about who drives a six-month migration. The transfer is by analogy, and the guide labels it as such.
- **The effect sizes and boundary conditions matter more than the headlines.** The bystander effect is *not* universal (§4.7), social loafing is reduced by several moderators beyond identifiability, and accountability can make people *worse* at a task when they are accountable to a known-preferring audience.
- **The practice's central failure mode is an organisational-design problem the psychology does not address at all**: naming a person answerable for an outcome they have no authority or resources to deliver. No amount of identifiability fixes a structural mismatch (§6.1).

**The honest formulation:** the research supports **the general principle of specific, attributed, single-owner accountability** and gives useful design constraints (prefer post-decisional justification; make contributions identifiable; keep goals specific and give feedback). It does **not** validate any particular meeting ritual, agenda format, template or tooling. Anything in circulation that claims a DRI *method* has been "proven" is overselling a sound principle.

### 4.7 The Mis-Citation Corrections

Four corrections, each of which the guide makes once and can be cited thereafter:

1. **Manning, Levine & Collins (2007) is in *American Psychologist*, not the *British Journal of Social Psychology*.** **Manning, R., Levine, M., & Collins, A. (2007). "The Kitty Genovese murder and the social psychology of helping: The parable of the 38 witnesses." *American Psychologist*, 62(6), 555–562. doi:10.1037/0003-066X.62.6.555** ✅ *(the mis-citation is surveyed as commonly made in the literature; the journal is confirmed as American Psychologist at DOI level this pass)*. **The substance is more important than the journal.** The paper's finding is that **the famous "38 witnesses stood by and did nothing" story does not survive scrutiny**: the number of witnesses who actually saw the attack was far smaller than reported, the newspaper account was inaccurate, and the story hardened into a **"parable" — a textbook morality tale manufactured *after* the studies rather than a factual prompt that preceded them**. This is the single best example of the accountability literature's own folklore problem: **the illustrative anecdote used to teach diffusion of responsibility is itself unreliable.** Use it as the cautionary centrepiece: the DRI field has the same vulnerability — a memorable story that gets repeated until nobody checks the source.
2. **Latané & Darley (1968) is *JPSP*, not *Science*.** Citation 2 above. Same trap, different journal.
3. **Ringelmann's 1913 study was rediscovered by Kravitz & Martin (1986), in French, in an agricultural-engineering publication, and over-claimed as "social loafing."** Citation 7 above. The interpretation is a later overlay and the data were mis-reported for decades.
4. **The bystander effect is danger-dependent — so the research does not say "people never help in groups."** **Fischer, P., Krueger, J. I., Greitemeyer, T., Vogrincic, C., Kastenmüller, A., Frey, D., Heene, M., Wicher, M., & Kainbacher, M. (2011). "The bystander-effect: A meta-analytic review on bystander intervention in dangerous and non-dangerous emergencies." *Psychological Bulletin*, 137(4), 517–537. doi:10.1037/a0023304** ✅ — the meta-analytic correction: the bystander effect is **strongest in non-dangerous emergencies and much weaker, or absent, in dangerous ones.** Anyone who deploys diffusion-of-responsibility research to argue that a named owner is needed *because groups are cowardly* is over-reading a genuinely nuanced body of work. The defensible version is narrower: **ambiguity and shared responsibility reduce intervention; unambiguous, individually-attributable responsibility does not.**

---

## 5. The Mechanics — How to Actually Run It

### 5.1 Where the Name Goes

The DRI only exists if it is **written somewhere the work is actually read**. Five surfaces, in descending order of durability:

| Surface | Best for | Failure mode if used alone |
|---|---|---|
| **The programme/plan document** | Quarter-long outcomes; cross-team deliverables | Nobody reads a 40-page plan; the name is never seen |
| **The action-item register** (from meetings) | Short-cycle actions; the original Apple usage | Actions multiply; the register becomes a graveyard of unattended items |
| **The runbook / operations manual** | Repeatable operational tasks, checks, failovers | Drifts stale unless reviewed on the same cadence as the procedure |
| **The responsibility map / charters** | Mapping outcomes to people and to regulatory roles (§9.1) | Becomes an org chart by other means; ages badly if not owned |
| **The change record / ticket** | A single production change, its verification and its rollback | Per-change DRIs with no aggregate view — 400 names, no accountability |

The rule that makes the surface choice easy: **put the name where the question will be asked.** If a stakeholder will ask "who owns this?" in a steering meeting, the name belongs in the plan and on the slide. If the question will be asked at 02:00 during an incident, it belongs in the runbook and on the incident document. If they will ask in a code review, it belongs in `CODEOWNERS`.

### 5.2 The Mandate — What It Should and Should Not Include

A DRI's mandate is a **short, written sentence** and it should contain four things:

1. **The outcome, stated as a result with a date** ("the new payments ledger is live in production for all retail accounts, by 31 March").
2. **The scope boundary** — what the outcome explicitly excludes, so the DRI is not quietly made answerable for adjacent work.
3. **The reporting line and cadence** — who the DRI reports to, in what forum, how often.
4. **The resources actually committed** — the named people, budget and systems the DRI can call on. **This is the clause most often omitted and the one that prevents §6.1.**

What the mandate should **not** include, and the reasons:

- **Unlimited scope.** "Own the payments domain" is not a DRI mandate; it is a job description. If the outcome cannot be finished, it is not an outcome.
- **Authority the DRI does not have.** If the mandate requires the DRI to direct a team that reports elsewhere, the mandate must say so *and* name the person who has agreed to that arrangement. Otherwise the charter is a wish.
- **Sole responsibility for the work.** The mandate names who answers, not who does everything (§1.2).
- **Permanent tenure.** Every mandate carries an end condition — delivered, cancelled, or reviewed on a named date.
- **A decision right, unless it is given explicitly.** See §5.3.

### 5.3 The Decision-Rights Question

The single most common design error is the silent assumption that the DRI decides. Three clean configurations exist, and the charter must state which one applies:

| Configuration | Who is DRI | Who decides | When to use it |
|---|---|---|---|
| **DRI-accountable, separate decider** | The person answerable for the outcome | A named approver or board (a DACI-style single Approver, a change board, an executive sponsor) | Default for regulated production change, spend, and anything with an external approval step |
| **DRI-decides** | The same person | The same person | Time-bounded, low-irreversibility work; incident command; anything where a decision delay costs more than a wrong call |
| **DRI-accountable, delegated decider** | The person answerable for the outcome | A named individual the DRI has delegated a specific class of decision to | Large outcomes where one human cannot hold every decision — the incident-commander pattern ("holds all positions they have not delegated" §3.7), inverted |

Two rules follow from the research in §4.4:

- **Name the decider separately, in writing.** Where the DRI must justify to an audience whose views are already known, *pre-decisional* accountability predicts conformity rather than judgement. Where the decision right sits with a separate named approver and the DRI must justify the calls afterwards, the DRI is in the more productive *post-decisional* mode.
- **Say what the DRI may not decide, not only what they may.** A charter that lists only permissions leaves the boundary to negotiation in the moment — and in a matrix, the negotiation is usually lost.

### 5.4 The Escalation Path

A DRI without a named escalation path is a DRI who will either stall in silence or escalate sideways. The charter names:

- **The trigger** — what condition obliges the DRI to escalate rather than absorb (a slip beyond a stated tolerance, a dependency that has not responded within an agreed window, a risk above a stated level, a decision the DRI has no right to make).
- **The route** — one named person first, with a named deputy if the first is unavailable; not a distribution list.
- **The service level** — how quickly the escalation gets a response, and what happens if it does not. An escalation path with no response commitment is a channel, not a path.
- **The no-blame rule on the escalation itself** — the DRI escalating early must not be read as failure. If escalation carries a reputational cost, the DRI will delay it, and the delay is where programmes die.

### 5.5 Handoff and Reassignment

The best published protocol for a handoff is the incident-commander handoff in Google's SRE book (§3.7) and it generalises directly:

1. **Name the incoming DRI explicitly** and get an explicit acknowledgement — the SRE formulation is "You're now the incident commander, okay?", and the outgoing person does not leave until acknowledgement is received.
2. **State what is being handed over** — the outcome, the current status, the open risks, the pending decisions, and the resources currently committed.
3. **Tell the other participants** who now owns it. Silence here is how two people end up believing they own it.
4. **Record the change** in the plan, the register or the ticket — the record, not the conversation, is the artefact of record.
5. **Keep the outgoing DRI reachable for one agreed period** (a defined number of days, not indefinitely). A handoff with no shadow period loses context; a shadow period with no end date recreates the two-owner problem.
6. **For long absences — leave, illness, sabbatical — reassign in advance**, in writing. Not doing so is how an unowned DRI is discovered only when something breaks (§6.6).

### 5.6 The DRI in a Meeting versus in a Written Plan

The two usages are different instruments and the practice is stronger when the difference is stated:

- **The meeting DRI** is a *routing device*: a name next to an action item, visible to the room. Its value is speed — no post-meeting search for who is following up. Its weakness is exactly what the source in §2 describes: it lives in an agenda that will be superseded next week.
- **The written-plan DRI** is an *accountability device*: a name against an outcome, with a mandate, a reporting line, a decision-rights statement and a date. It survives personnel churn and is the version a regulator or auditor will ask for.

The mature practice runs both, with one rule: **every meeting action that represents real work must graduate into the written plan within one cycle.** An action register that never upgrades its items into a plan is a list of intentions with names attached.

### 5.7 Incident DRI versus Quarter-Long Outcome DRI

| Dimension | Incident DRI | Outcome DRI |
|---|---|---|
| Duration | Hours | Weeks to quarters |
| Decision rights | Broad, often full command | Usually narrow; often separate decider |
| Cadence | Continuous, live document | Weekly or fortnightly reporting |
| Handoff | Explicit, shift-based, acknowledged (§5.5) | Planned, with shadow period |
| Success measure | Service restored; incident closed | Outcome delivered to stated criteria |
| Primary failure mode | Unclear leadership during the response | Accountability without authority |
| Required artefact | Live incident document | Charter + plan + responsibility map |

Treating a quarter-long outcome like an incident produces a heroic, exhausted owner and an undocumented plan. Treating an incident like an outcome produces a committee during the outage. **The charter must say which type of DRI this is.**

### 5.8 The Line Manager and the Matrix Tension

The DRI model will collide with the reporting line in any organisation that has a matrix — and a bank always has a matrix. Four rules keep the collision survivable:

1. **The DRI does not direct people; they direct the outcome.** The DRI's leverage over a person who reports elsewhere is the mandate and the escalation path, not the performance review. Say this explicitly at the start of the outcome.
2. **The line manager is informed in writing when their person is named DRI.** The mandate should name the resource commitment and note that the resource's manager has agreed to it. This single step prevents the most common form of matrix conflict: a manager discovering their person is answerable for something they were never asked to resource.
3. **Two DRIs and one manager is a workload problem, not a motivation problem.** If the same person is DRI on four outcomes, the corrective is the anti-pattern in §6.2 — the accountability is real, the capacity is not.
4. **When they conflict irreconcilably, the conflict is escalated, not absorbed.** A DRI who quietly absorbs a conflicting instruction because raising it feels political has converted an organisational problem into an invisible personal one — and the outcome is now at risk precisely where nobody can see it.

### 5.9 The DRI Charter — A Constructed Template

⚠ **This template is constructed by this guide.** It is not an industry standard, not a regulator's form, and not derived from a published template. It is offered as a starting artefact, and the guide labels it as such so that nobody mistakes it for something a supervisory authority issued. (For the *regulatory* template — APRA's accountability statement guidance — see §8.5.)

```
DRI CHARTER                                              Version 1.0 · Owner: <programme>
─────────────────────────────────────────────────────────────────────────────────────────

1. OUTCOME
   Result / success test: <a delivered result, and how we will know it is done, measurably>
   Target date: <date>   In scope: <the boundary>   Out of scope: <explicitly excluded>
2. THE DRI
   Name: <one person>   Role: <day job>   Line manager: <owner of their performance/workload>
   Time commitment: <agreed % or days>   Deputy: <named, and what they may decide while acting>
3. MANDATE
   Decides: <classes of decision this DRI may make alone>
   Does NOT decide: <classes that sit elsewhere — named approver below>
   Approver(s): <one named human per decision class, not a committee>
   Reporting: <to whom, in which forum, at what cadence>
   Resources committed: <named people, budget, systems — and who agreed them>
4. ESCALATION
   Trigger: <condition that obliges escalation>   Route: <named person, then named deputy>
   Response time: <hours/days>   Rule: early escalation is not failure
5. DEPENDENCIES: <dependency — the DRI on the other side — interface — deadline>
6. RISKS AND OPEN DECISIONS: <risk — severity — mitigation — owner — date>
7. HANDOFF
   Method: <explicit, acknowledged, recorded (§5.5)>   Shadow period: <N days>
   Record of record: <where the change to this charter is written>
8. REVIEW
   Review date: <date>   Reviewer: <named>
   End condition: delivered / cancelled / reviewed on <date>
9. ANTI-PATTERN CHECK  (signed at charter approval)
   [ ] The DRI can resource, staff and decide what this charter says they can
   [ ] No second name appears anywhere against this outcome
   [ ] The override rights of any manager above are stated, not implied
   [ ] The outcome fits in one sentence a colleague could repeat

Signed: DRI ____________  Approver ____________  Line manager ____________  Date ______
```

---

## 6. The Anti-Patterns

Each anti-pattern is given with its **symptom** (what you will observe) and its **guardrail** (the structural fix). The first is the central one; the rest are variations and consequences.

### 6.1 Accountability Without Authority

- **Symptom:** A person is named DRI for an outcome whose dependencies, budget or decisions belong to others who have not agreed to the arrangement. At every steering meeting the DRI reports a slip that is someone else's to unblock; the DRI's own escalations come back as "we'll look into it". The DRI becomes the polite narrator of a problem they cannot solve — and after two or three cycles, starts to be *blamed* for it.
- **Why it is the central failure:** It is the only anti-pattern in which the DRI mechanism makes things *worse* than no mechanism at all. Without a name, the inability to deliver is a structural fact everyone can see. With a name, it becomes one person's shortfall.
- **Guardrail:** The **charter's resource clause (§5.2 item 4) and the anti-pattern check (§5.9 item 9)**. Do not sign a charter naming a DRI without the resource commitments written down and agreed by the people who control them. If a DRI cannot get the commitments in writing, the honest conclusion is that the outcome is not resourced — escalate that fact, and name *that* as the issue rather than the person's performance on it. The Amazon comparison is instructive (§3.5): Amazon buys the authority structurally before it names the owner; a DRI model applied inside an unchanged structure must buy it contractually, at charter approval, or not at all.

### 6.2 The Over-Assigned DRI (the Bottleneck)

- **Symptom:** The same three reliable people are DRIs on eleven of the fourteen outcomes. Their calendars are wall-to-wall status updates; the work they personally own slips; the outcomes they nominally own are actually driven by whoever is nearest. The organisation reads "this person is trusted" while the load is a single point of failure.
- **Guardrail:** A **DRI load register** — a view of outcomes per named person, reviewed monthly like any other capacity metric. Set a stated ceiling (three is a common practitioner figure; treat it as a heuristic, not a finding) and treat every breach as a resourcing decision requiring a named alternative. The register is also the honest answer when a manager asks why the DRI model "isn't working": the mechanism is sound; the staffing is not.

### 6.3 DRI in Name Only

- **Symptom:** A DRI is named, and then every meeting is attended by their manager, who answers the questions first, corrects the DRI in front of the room, and reserves the right to overrule the charter's stated range. The DRI learns not to decide. Over a quarter they become a reporting channel rather than an owner.
- **Guardrail:** **Name the override explicitly and bound it.** Write into the charter (§5.9 item 3) what, if anything, a manager above may override, and require that any override be *recorded* — the same way an incident handoff is recorded. Unrecorded overrides are the mechanism by which authority drains out of a named role. If a manager cannot bring themselves to leave the room, the honest model is a "DRI with a named supervising manager", stated as such — not a fiction of delegated ownership.

### 6.4 The Multiple-DRI Fiction

- **Symptom:** Two or three names appear against one outcome, usually with the phrase "jointly accountable" or "co-DRI". Where one is senior and one is not, the junior answers for the actual work while the senior answers for nothing. Where both are peers, the outcome stalls whenever they disagree, because there is no tie-breaker and no single name to break it.
- **Why it happens:** Plural naming is almost always a way of **pleasing everyone at charter approval** — nobody has to be excluded, no manager loses face. It therefore tends to appear exactly on the outcomes that matter most, in institutions that are most conflict-averse.
- **Guardrail:** **One name, always** — and where two functions genuinely must both be represented, name one DRI per outcome and use the charter's dependency table (§5.9 item 5) to record the other. That preserves representation without breaking single-owner attribution. Where the regulator is involved, note that the regimes in §8 also require a single named individual per responsibility — plurality is not a neutral design choice; it is a compliance problem.

### 6.5 The DRI as a Blame Instrument

- **Symptom:** The DRI's name is quoted most often in post-incident reviews, in executive meetings after a slip, and in performance discussions. Nobody quotes it when the outcome succeeds. The most capable people begin declining to be named; the ones who accept are the ones least able to refuse. Escalation stops happening because escalation identifies a problem area with a name on it.
- **Why it is a real risk rather than a hypothetical one:** Because it is the same mechanism, run with the sign flipped. Goal specificity narrows attention (§4.5), and accountability to a known-preferring audience produces conformity rather than judgement (§4.4). A blame culture uses DRI naming to assign outcomes that are already known to be at risk.
- **Guardrail:** **Separate the outcome review from the performance review.** Outcomes and their DRIs are reviewed in a delivery forum against the plan's criteria; individual performance is reviewed by the line manager, on a different cadence, with the DRI record as an input and not as a verdict. Write a **standing rule into the model**: a DRI who escalates early or reports a slip honestly is treated as having done their job. And note how the practice standard in this area has moved: the repository's incident practice follows the **blameless postmortem** convention, recorded in [the_managers_path_guide.md](the_managers_path_guide.md)'s glossary — the DRI model should inherit that convention rather than fight it.

### 6.6 The Absent DRI

- **Symptom:** The named DRI is on leave, has changed role, has left the organisation, or has been redeployed — and the outcome continues under nobody's name. Status reports stop; the plan's dates quietly become fictional; the discovery happens when someone outside the team asks a question nobody can answer. The charter still names a person; the record has become archaeology.
- **Guardrail:** **Mandatory reassignment on role change, with no gap.** Make reassignment a step in the offboarding and internal-move checklists, not an afterthought: no person leaves a role with an open DRI charter without a named successor or an explicit decision to close the outcome. Run a **quarterly DRI sweep** — every open outcome, its named owner, and whether that person is still in place and still resourced. The sweep is cheap and it is the only control that catches silent ownership loss.

### 6.7 The Cultural Precondition Question

This is a real design consideration, not a stereotype, and it is worth stating carefully because it is the anti-pattern most likely to be dismissed as sensitivity rather than treated as engineering.

- **The precondition:** The DRI practice rests on a set of behaviours that are not culturally neutral — **publicly accepting a name against a result, disagreeing with a more senior person in a meeting, escalating a problem upward without being invited to, and reporting a likely failure before it becomes certain.** Where the prevailing norms reward harmony over visible ownership, or where seniority is expected to be deferred to rather than questioned, those behaviours carry a personal cost that varies by person and by context.
- **The vocabulary:** The useful frame is the Hofstede-style dimensions of **power distance** (the acceptance of inequality in the distribution of power) and the indirect-communication conventions often described as **high-context** — where what is *not* said carries meaning and a direct public naming is a heavier social act than in a low-context setting. ⚠ **These dimensions are population-level statistical tendencies, not descriptions of individuals.** Applied to a person, they are a stereotype; applied to a design decision about *where a name is written and who reads it*, they are a legitimate input. The repository's own treatment of the power-distance dimension is in [authority_skills_guide.md](authority_skills_guide.md) §11.3, and that guide's caution about national-culture reasoning applies here unchanged.
- **The specific exposure:** For a Singapore- or Asia-based institution with a mixed workforce, the practical asymmetry is that **a name spoken aloud in a meeting and a name written in a plan do not carry the same social weight.** The spoken version creates public exposure for the named person and, in a high-power-distance setting, also implicitly marks who did *not* get named. The written version creates attribution — the thing the mechanism actually needs — without requiring anyone to be publicly singled out in a room.
- **The guardrails that follow:** (i) **prefer the written record to the spoken call-out** — put the name in the plan, the register, the ticket; stop the practice of naming and shaming by implication in a meeting; (ii) **name roles and outcomes, never failures and people**, in public forums; (iii) **give the DRI a private channel to raise blockers before the public forum**, so that escalation does not require a public admission; (iv) **make the escalation path's no-blame rule explicit and enforce it visibly the first time someone uses it**; (v) **rotating the DRI assignment deliberately** so that being named is not a permanent marker of one person's fragility; and (vi) **verify the practice is working by asking the DRIs, not the managers** — whether they can resource their outcomes and whether they feel able to raise a slip early. That last question is the only reliable early-warning measure in this whole guide.

---

## 7. The Variants in the Wild

Each variant is presented with what it actually is, what it genuinely shares with the DRI, and — the more useful half — what it does not.

### 7.1 The Amazon Single-Threaded Leader

Discussed in full at §3.5. The summary position: **this is the closest large-scale industrial implementation of the DRI principle, and it is achieved by organisational design rather than by naming alone.** The STL gets one thing to focus on because the organisation is deliberately shaped so that this is possible; the separable single-threaded team gets ownership because its dependencies are cut. **Shared with the DRI:** one person, one bounded thing, end-to-end answerability. **Not shared:** the DRI is usually a name inside an existing structure, with no restructuring, no dedicated team and no release from other duties. That is precisely why §6.1 is the DRI's most common death. If you can afford Amazon's version, run Amazon's version.

### 7.2 The Design-Doc Owner

The practice, common in engineering organisations, of attaching a **named owner** to a design document, RFC, or architecture decision record: one person is responsible for the document's content, its circulation, its comment resolution and its final acceptance. It shares with the DRI exactly the mechanism that matters — **a document with one name on it does not sit unresolved** — and it composes well with the charter model: an outcome DRI typically delegates the design artefact to a design-doc owner, who becomes a RACI A for that line. **Not shared:** the design-doc owner's accountability ends at acceptance of the document, not at delivery of the thing the document describes. Confusing the two is a common and expensive error: a well-owned design does not make the implementation owned. (The repository's sibling treatment of architecture decision records is [../technology/architecture_decision_record_guide.md](../technology/architecture_decision_record_guide.md).)

### 7.3 The Incident-Command Model

Discussed in detail at §3.7, and the most directly importable of the variants because it is **written down, role-structured and operationally tested.** Its transferable components for a DRI model are the three named in §3.7: the default that the commander holds all undelegated roles, the living document, and the explicitly acknowledged handoff. **Shared with the DRI:** single named owner; a written record; a defined handoff. **Not shared:** the command right (which a DRI may not have) and the tempo (hours, not quarters). The variant that generalises best across the whole repository is the handoff protocol, because it is the answer to §6.6.

### 7.4 The Open-Source Maintainer and the Ownership File

The maintainer model — `CODEOWNERS`, the Linux kernel `MAINTAINERS` file, and the wider convention of a named release manager per release — is the **oldest continuously-operating named-ownership practice in software** (§3.8). **Shared with the DRI:** a published, version-controlled map from a thing to a named human; the name is used for routing; and the record is machine-readable enough to be enforced (a required reviewer can literally block a merge) ✅. **Not shared:** the maintainer model is scoped to **a code area's change stream**, not to a stated outcome with a date; it explicitly permits **teams** as owners (which a DRI model cannot); and it is overwhelmingly **voluntary and revolving** rather than assigned by a mandate. A useful discipline to import anyway: **put the ownership record in the repository, not in a slide deck.** Files that live where the work lives get updated; files that live in a wiki nobody owns do not.

### 7.5 Commander's Intent — The One That Is Not a DRI

This is included specifically because it is so often mistaken for a DRI variant that the correction is worth a full subsection. **Commander's intent** is the practice, from military doctrine, of communicating a **purpose and desired end state** clearly enough that subordinates can execute autonomously when the plan meets reality — and can improvise correctly in the absence of further orders. It is about **decentralised execution under a stated intent**.

- **What it genuinely shares with a DRI:** both depend on a clearly stated outcome, and both are crippled by vague goals. The goal-setting finding in §4.5 underwrites both.
- **What it does not share — and where the confusion does damage:** commander's intent **does not name an owner**. It is a *communication* device that deliberately distributes decision latitude across many people, precisely so that no single person's judgement is the bottleneck. A DRI names one. The two are complements — **state the intent, then name the DRI who will be asked how it went** — and treating commander's intent as an accountability mechanism produces the opposite of what the DRI is for: many empowered actors, no single answerable one. If a manager says "we use commander's intent instead of DRIs", they have described an environment where every person can act and nobody answers — §6.4 in doctrinal clothing.

---

## 8. The Regulatory Formalisation — Individual Accountability in Banking

### 8.1 Why a Regulator Would Want a DRI

The five regimes below were not designed by people reading management books. They are the **post-crisis regulatory answer to a specific, documented failure**: an institution fails, the supervisory authority asks who was responsible for the business line, and the answer in law is "the firm" — a legal person that cannot be admonished, cannot be disqualified from holding office, and whose accountability cannot be located. The regimes solve that by **attaching named senior individuals to defined areas of responsibility, requiring that the allocation be documented and notified, and giving the regulator powers that reach the individual**.

Read against §1, the design is unmistakably DRI-shaped: **one named human, one defined responsibility, written down**. What differs from an internal DRI practice is the consequence layer — statutory duties, regulatory approval or certification, notification obligations, and enforcement against the individual rather than the firm. This section **compares** the regimes. It **does not re-derive MAS IAC** — that detail is owned by [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §4.5, and this guide cross-references it rather than duplicating it.

### 8.2 United Kingdom — SM&CR

✅ **Verified this pass against the PRA Rulebook and Bank of England policy statements** (fca.org.uk was unreachable to the scraper; the PRA/BoE sources carry the regime).

- **Name and status:** the **Senior Managers and Certification Regime (SM&CR)**. It is embedded in the **PRA Rulebook** as separate Parts — **Allocation of Responsibilities, Senior Management Functions, Certification, Conduct Rules, Fitness and Propriety** ✅ — plus FCA rules, on a statutory footing in FSMA 2000.
- **Commencement:** the PRA Rulebook's version history carries snapshots dated **07/03/2016** for the Senior Management Functions, Allocation of Responsibilities, Certification and Conduct Rules Parts ✅ — consistent with the regime's commencement for banks on **7 March 2016**. The BoE's **PS12/26 (published 22 April 2026)** states that "the SM&CR has been in place since 2016 as a response to the serious problems that emerged during the financial crisis. It ensures that appropriate senior decision-makers are identified and that these individuals are clear as to their responsibilities and accountability." ✅
- **The named-individual mechanism:** a senior manager is **approved under section 59 of FSMA** to perform a **senior management function (SMF)** ✅ (the application and Conduct Rules provisions turn on that approval). Approval is granted by the PRA or the FCA depending on the function.
- **The responsibilities document:** the **statement of responsibilities (SoR)**. The PRA Rulebook defines it as "a statement of the affairs of a *relevant authorised person*… for which it is intended that a *person* who performs (or is subject to an application to perform) a *PRA senior management function* is (or will be) responsible" ✅, requires that an application for SMF approval "**is accompanied by a statement of responsibilities**" (Allocation of Responsibilities 2.1) ✅, requires the SoR to include the **prescribed responsibilities** applicable to the firm ✅, and requires the firm to "have, at all times, a complete set of current and up to date statements of responsibilities" ✅. **This is the DRI charter with statutory force.**
- **The certification limb:** the **Certification** Part requires firms to certify individuals performing **certification functions** ✅ — roles that can cause significant harm but which the regulators do not individually approve; the firm performs the fit-and-proper assessment. Statutory hook: **s.63E FSMA** (referenced in the Certification Part) ✅.
- **The conduct limb:** the **Conduct Rules** Part applies individual rules to persons including SMF holders, certification-function holders and designated non-executive directors ✅, with **Senior Manager Conduct Rules** addressing the disclosure of information the regulators "would reasonably expect notice" ✅. These bind individuals, not only firms.
- **Supervisory expectation:** **SS28/15 — "Strengthening individual accountability in banking"** ✅, updated as part of PS12/26 ✅.
- **Current development (2026):** the regime is being **reviewed and streamlined in two phases**. **PS12/26 (22 April 2026)** sets out the PRA's Phase 1 reforms (including changes to the "12-week rule" for covering temporary SMF vacancies, clarity on scope including the Group Entity Senior Manager function SMF7, and streamlining statements of responsibilities, regulatory references and criminal record checks) ✅, alongside the FCA's own Phase 1 policy statement ✅. **HM Treasury's consultation response**, published the same day, sets out the legislative changes for a Phase 2 — including "reducing administrative requirements set in legislation, including around **Statements of Responsibilities** and Conduct Rules processes" ✅. ⚠ **Practical implication for a DRI model: the *mechanism* of named senior responsibility is enduring; the *documentation burden* around it is actively being reduced.** Design charters to survive that simplification — keep the outcome, mandate, decision right and named human explicit; keep the ceremonial paperwork minimal.
- **Extension to the wider sector:** ⚠ **the extension of the SM&CR to FCA solo-regulated firms in December 2019 and to the wider authorised sector by the Financial Services and Markets Act 2023 was not re-verified this pass** — the PRA Rulebook's version history carries a **09/12/2019** snapshot ✅, which is consistent with that date, and the HMT/BoE material describes a regime that "covers almost all firms regulated by the FCA and the PRA" ✅, but the underlying legislation was unreachable. Treat the **specific extension dates as ⚠** and the **direction of travel (broad sector coverage) as ✅**.

### 8.3 Singapore — MAS IAC

✅ **Verified this pass against the MAS page** — and summarised here only at the level needed for comparison, because the detail belongs to [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §4.5.

- **Name and status:** the **Guidelines on Individual Accountability and Conduct (IAC)**, published **10 September 2020** ✅. It is **guidance, not law** — MAS describes it as "guidance on the five high level outcomes that financial institutions should achieve to promote the accountability of senior managers, strengthen oversight over material risk personnel, and reinforce conduct standards among all employees." ✅
- **Coverage:** banking, licensed trust companies, licensed financial advisers, insurance, capital markets services (dealing, corporate finance, credit rating, clearing, benchmark administration, fund management, exchanges, trade repositories), payments, and insurance brokers ✅ — a **sector-broad** instrument.
- **Mechanism:** five **high-level outcomes** rather than prescribed forms — the approach is outcome-based, which leaves the *instrument* (a responsibility map, a charter, a committee structure) to the institution. This is the sharpest difference from the UK/Ireland model, where the document is legislated.
- **Consequence boundary:** ⚠ **a breach of the IAC guidelines is not itself an offence.** Because it is guidance under MAS's supervisory framework, the consequence is supervisory (expectations, findings, and the reputational and licensing consequences of non-compliance) rather than a statutory penalty for breach of the guidelines. **Do not describe the IAC as creating statutory duties on individuals** — it does not, on the source fetched. Actual individual consequences in Singapore arise from other instruments (approval regimes, the Banking Act, the Securities and Futures Act, and MAS's enforcement powers) — ⚠ not verified in this pass.

### 8.4 Hong Kong — MIC

✅ **Vehicle and dates verified; operational detail flagged.**

- **Vehicle:** the HKMA's **Supervisory Policy Manual module CG-1, "Corporate Governance of Locally Incorporated Authorized Institutions"** ✅. The **current issue is dated 1 August 2025** ✅; a **CIR on the "Gazettal of revised SPM CG-1 on corporate governance"** is dated 1 August 2025 ✅; and the **version history shows CG-1 V3 archived at 6 October 2017** ✅ — the revision that introduced the Managers-In-Charge expectations. The HKMA's **CIR "Corporate Governance" of 15 December 2021** is also listed against CG-1 ✅.
- **The named-individual mechanism:** the **Managers-In-Charge (MIC)** regime — AIs designate an MIC for each of the HKMA's **core functions** ⚠, publish an **organogram or responsibility map** showing the allocation ⚠, and **notify the HKMA of MIC appointments and changes** ⚠.
- ⚠ **Explicit flag:** the **CG-1 module text itself could not be extracted this pass** — the HKMA document links resolve to the Banking Regulatory Document Repository landing pages rather than the PDF content. The **vehicle, the current issue date and the 2017 revision are verified ✅**; the **specific MIC mechanics** (the named list of core functions, the organogram requirement, the notification obligation) are stated here as **⚠-knowledge — the widely-documented description of the regime, not re-verified against the module text this pass.** Verify against CG-1 before relying on any particular detail. This flag is repeated in §12.

### 8.5 Australia — FAR and Its BEAR Predecessor

✅ **Verified this pass against APRA's FAR pages.**

- **Name and status:** the **Financial Accountability Regime (FAR)** — "a strengthened responsibility and accountability framework for entities in the banking, insurance and superannuation industries and their directors and senior executives", "designed to improve the risk and governance cultures of Australia's financial institutions" ✅. It is enacted in the **Financial Accountability Regime Act 2023** (and associated rules, including the FAR (Minister) Rules 2024 and Regulator Rules) ✅ — **legislation, not guidance.**
- **Predecessor:** the FAR "replaces the **Banking Executive Accountability Regime (BEAR), which commenced in 2018**" ✅; APRA published an information paper on BEAR implementation in December 2020 ✅.
- **Administration:** **jointly administered by APRA and ASIC** ✅, under a published Joint Administration Agreement ✅.
- **Commencement dates:** **15 March 2024** for **authorised deposit-taking institutions (ADIs) and their authorised non-operating holding companies (NOHCs)**; **15 March 2025** for **insurance entities, their licensed NOHCs, and superannuation trustees** ✅.
- **Core mechanism:** the regime operates through **accountable persons** ⚠, **accountability statements** and accountability maps lodged with the regulators ⚠, supported by APRA's published **"Accountability statement guidance and template"** ✅ and APRA Connect reporting forms ✅, with the regulators' regulatory and enforcement powers available for breaches ✅.
- **Boundary worth noting:** the FAR covers **insurance and superannuation as well as banking** ✅ — it is a *financial sector* regime, not a banking-only one, which makes it structurally closer to the FCA-solo-regulated extension than to the original SM&CR. ⚠ The **civil penalty provisions and their quantum** were not verified this pass; the sources fetched confirm the statutory basis and the regulators' enforcement powers, not the specific penalties. Do not quote penalty figures from this guide.

### 8.6 Ireland — SEAR and the IAF

✅ **Verified this pass against the Central Bank of Ireland's IAF page.**

- **The statute:** the **Central Bank (Individual Accountability Framework) Act 2023** — "signed into law on **9 March 2023** and… partially commenced on **19 April 2023**. The remaining parts of the IAF Act were commenced on **29 December 2023**. Some aspects of the IAF Act did not take effect until the Central Bank issued enabling regulations." ✅
- **Four elements** ✅:
  1. **Senior Executive Accountability Regime (SEAR)** — "requires in-scope firms to set out clearly and fully where responsibility and decision-making lie within the firm's senior management, and imposes a **legal Duty of Responsibility** on persons carrying out **Pre-Approval Controlled Function (PCF)** roles in such firms"; SEAR "will apply in respect of executive PCF roles within in-scope firms from **1 July 2024**" ✅. Supported by the **SEAR Regulations** and the Central Bank's **Guidance on the IAF** ✅.
  2. **Conduct Standards** — **Common Conduct Standards** for all individuals in Controlled Function roles in all regulated firms, and **Additional Conduct Standards** for PCF roles and CF-1 (any other role that may exercise significant influence on the firm's affairs); "applied since **29 December 2023**" ✅.
  3. **Fitness and Propriety enhancements** — "require regulated firms to **proactively certify** that individuals carrying out CF and PCF roles meet the Central Bank's standards of F&P"; the Certification Regulations "have applied since **8 January 2024**" ✅.
  4. **Administrative Sanctions Procedure (ASP) enhancements** — "the Central Bank now has the ability to take enforcement action under the ASP **directly against individuals** who have breached their personal legal obligations, including those under the Conduct Standards, **rather than only for their participation in breaches committed by a regulated firm**" ✅.
- **Why Ireland matters most for this guide:** element 4 is the **clearest statutory statement in the set** that individual accountability is enforced **against the person**, not merely through the firm — the exact transformation the DRI principle implies when it leaves the meeting room.

### 8.7 The Comparison Table

| Jurisdiction | Regime | Instrument type | Named-individual mechanism | Responsibilities document | Approval / certification | Consequence |
|---|---|---|---|---|---|---|
| **UK** | SM&CR (since 2016; two-phase reform under way in 2026) | Law + regulator rules (FSMA 2000; PRA Rulebook Parts) ✅ | **Senior Management Function** holders, approved under s.59 FSMA ✅ | **Statement of Responsibilities** required with the approval application; complete set maintained at all times; includes prescribed responsibilities ✅ | **Regulatory approval** for SMFs ✅; **firm certification** for certification functions (s.63E) ✅ | Statutory: SMF misconduct can lead to prohibition and financial penalties; individual **Conduct Rules** ✅ |
| **Singapore** | MAS IAC (2020) | **Guidance — not law** ✅ | Senior managers with clear allocation of responsibility; material risk personnel under conduct and fitness standards ✅ | Not prescribed — five outcomes leave the instrument to the firm ✅ | Existing MAS approval regimes for senior appointments; ⚠ not part of the IAC guidelines | **Supervisory** — a breach of the guidelines is not itself an offence ⚠ |
| **Hong Kong** | HKMA MIC (via SPM CG-1; current issue 1 Aug 2025; MIC expectations from CG-1 V3, 6 Oct 2017) ✅ vehicle / ⚠ detail | Supervisory policy manual module + CIRs ✅ | **Managers-In-Charge** designated for each core function ⚠ | Organogram / responsibility map ⚠ | Notification of appointments and changes to the HKMA ⚠ | Supervisory; the HKMA's powers under the Banking Ordinance ⚠ |
| **Australia** | FAR (2024/2025), replacing BEAR (2018) ✅ | **Legislation** — FAR Act 2023 + rules ✅ | **Accountable persons** ⚠ | **Accountability statements** and accountability maps lodged with the regulators ✅ / ⚠ | Registration with APRA and ASIC as accountable persons; APRA accountability statement template ✅ | Regulators' regulatory and enforcement powers ✅; ⚠ penalty detail not verified |
| **Ireland** | IAF — SEAR + Conduct Standards + F&P + ASP ✅ | **Legislation** — Central Bank (IAF) Act 2023 ✅ | **Pre-Approval Controlled Function (PCF)** holders under a **legal Duty of Responsibility** ✅ | SEAR requires firms to set out where responsibility and decision-making lie ✅ | **Pre-approval** for PCF roles; firm **certification** for CF/PCF under the Certification Regulations ✅ | **Enforcement action directly against individuals** under the ASP ✅ |

Two rows of that table are the ones a DRI designer should read twice: **Singapore's is guidance and Australia's is legislation** — the same principle, with wholly different consequence layers. An institution operating across both jurisdictions cannot run one internal ownership model and assume it satisfies both.

### 8.8 The Boundaries — What Each Regime Does Not Require

The boundaries matter as much as the requirements, because they define where an internal DRI model must do work the regulator will never do for it:

- **No regime requires a DRI for a *programme*.** The statutory regimes attach individuals to **functions, business areas, prescribed responsibilities and conduct standards** — enduring responsibilities — not to time-bounded deliverables. Mapping a programme-level DRI onto an SMF/PCF/accountable-person role is **an internal design choice the regimes leave open** (§9.1).
- **No regime prescribes the meeting ritual.** Nothing in the fetched material requires a name next to an action item. The regulatory instrument is the **written allocation and notification**, which is why the guide's §6.7 guardrail — prefer the written record to the spoken call-out — is also the compliant instinct.
- **Guidance is not law.** MAS IAC sets outcomes; its breach is not itself an offence. ⚠ Regimes differ in this respect and the difference is not cosmetic: it determines what a supervisory finding can escalate into.
- **The regimes do not resource the role.** They create duties and consequences; they do not create capacity. An institution that names an accountable person for a function nobody staffed has produced a **regulatory** version of §6.1, with a higher price.
- **The regimes are being actively reformed.** The UK's two-phase review (§8.2) is the live example: the accountability principle is stable, the documentation around it is being reduced. A DRI model should be built on the durable part.

---

## 9. The DRI in a Regulated Institution

### 9.1 The Reconciliation Problem

The internal DRI and the regulatory accountable person are **not always the same human being**, and the failure to reconcile them is the most consequential mistake a regulated institution can make with this practice:

| Case | Internal DRI | Regulatory accountable person | Risk if unreconciled |
|---|---|---|---|
| **Same person** | Named for the outcome | Named for the function/area | None — but the charter must say so, and the outcome must fall inside that person's allocated responsibilities |
| **DRI below the accountable person** | A team lead or senior IC driving the delivery | The executive for the business area | **The most common and most dangerous case.** The regulator's duty attaches to the executive; the institution's delivery risk sits with the IC. If something fails, the executive must answer for an area they delegated without a documented delegation |
| **DRI outside the allocated area** | An engineer in Technology owning a business outcome | The business head for that outcome's area | The delivery is owned by someone the supervisory map does not recognise — a gap in the evidence trail |
| **DRI is a third party** | A vendor's delivery lead | The institution's accountable person (always) | Accountability cannot be outsourced; the vendor can be *responsible*, never *accountable* (§9.3) |

The reconciliation discipline: **every internal DRI charter states which regulatory accountability it sits inside**, by naming the accountable person and the allocated responsibility it maps to. Where no such mapping exists, the outcome falls outside the institution's supervisory map — which is itself the finding worth escalating.

### 9.2 The Evidence Trail

Supervision asks for evidence, not intentions. The minimum consistent artefact set:

- **The allocation of responsibilities** (the responsibility map or statements), version-controlled, with effective dates and the approving authority.
- **The DRI charters** (§5.9), each naming the outcome, the single person, the decision rights and the resource commitments.
- **The decision records** for outcomes, with the decision, the date, the decider and the rationale — the artefact that puts the DRI in the post-decisional accountability mode the research in §4.4 favours.
- **The change records** for production changes, naming the DRI for the change, its verification and its rollback (§9.4).
- **The escalation log** — what was escalated, when, to whom, and what happened. This is the artefact that demonstrates the institution's own control framework responded, rather than letting a named individual absorb a structural failure.
- **The handoff records** (§5.5) and the **quarterly DRI sweep** output (§6.6).
- **The fit-and-proper and certification records** for the persons named, where the regime requires them.

**The rule:** if the evidence exists only in a meeting that has passed, the institution has accountability without a record. Where a regime requires notification to the regulator, the record must be contemporaneous with the appointment, not reconstructed afterwards.

### 9.3 Outsourcing and Third-Party DRIs

Third parties change the *responsibility* but never the *accountability*:

- **A vendor may hold a DRI role internally** — a named delivery lead on the vendor's side — and that is useful operationally, because it gives the institution one name to talk to for the vendor's workstream.
- **The institution's DRI for the outcome remains an employee.** No regime in §8 permits an outsourced function to move individual accountability off an in-house accountable person, and the institution's own DRI model should not either.
- **The interface must be written down**: the vendor's named DRI, their escalation route, the service levels, the evidence the institution will receive, and the **exit/handover obligations** — including the transfer of ownership records at termination, which is where third-party ownership most often goes missing.
- **The composite view matters.** Where five vendors each own a slice of one outcome, the institution's DRI owns the *integration risk* explicitly and by name — that is the part no vendor can own, and the part that most often goes unnamed.

### 9.4 The Change-Management DRI

Production change is where the DRI practice meets regulated reality most concretely, and where the smallest unit of ownership is most abused. A workable model:

- **Every production change has a named DRI for the change** — distinct from the approver (the change board, or the person who authorises it in the tooling) and distinct from the person who executes it.
- **The change DRI owns four things:** the change's readiness (test evidence, rollback plan, observability in place), its execution coordination, its verification, and the decision to roll back. **The rollback decision in particular needs a single name** — a rollback decided by consensus at 02:00 happens late.
- **The approver pattern from §5.3 applies:** in regulated change, the decider is normally separate from the DRI, and the DRI justifies the calls afterwards (post-decisional accountability).
- **Individual change DRIs aggregate into an operational owner.** Four hundred named change records do not add up to accountability for the platform; the operational DRI for the service owns the aggregate and reviews the change record stream. This is the point of contact with the repository's operational-resilience material — [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) — which treats important business services and impact tolerances, and the ownership model must be expressible in that vocabulary.
- **Where the change is outsourced or platform-provided**, §9.3 applies: the institution names the internal DRI regardless of who executes the change.

### 9.5 The Audit and Supervisory Expectation

Internal audit does not typically audit "the DRI model"; it audits the **control environment that the model is supposed to sustain** — allocation of responsibilities, change governance, incident management, third-party oversight, and the integrity of the records the institution relies on. Three practical expectations follow:

- **Consistency between the documents.** The responsibility map, the DRI charters, the change records and the HR records naming post-holders must not contradict each other. Contradiction is the finding auditors look for, because it is objective and easy to evidence.
- **Completeness of the allocation.** Gaps — responsibilities with no named person — are reportable, especially where a regime requires a complete set of statements at all times (§8.2).
- **Evidence that the model is used, not just documented.** An organisation that produces charters nobody reads and sweeps nothing finds will pass a documentation check and fail a control-effectiveness one. The escalation log and the sweep output are the two artefacts that most directly evidence use.
- **For the regulator specifically:** the risk register, the responsibility map, and the response to the last supervisory review are examined together. ⚠ The repository's consolidated treatment of MAS supervisory expectations is owned by [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §4.5 and [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); this guide cross-references rather than restates them.

### 9.6 The Conduct Dimension the Regimes Are Really Aimed At

It is worth stating what the whole apparatus is *for*, because a compliance-driven DRI model misses it. The regimes in §8 are not principally instruments of blame attribution; they are instruments for **making the organisation say what it actually does and who actually decides**, and then making that statement the thing a named human is answerable for. The declared rationales make this explicit: the SM&CR "ensures that appropriate senior decision-makers are identified and that these individuals are **clear as to their responsibilities and accountability**" ✅; MAS's IAC exists to "**reinforce conduct standards among all employees**" ✅; the FAR is "designed to improve the **risk and governance cultures** of Australia's financial institutions" ✅; Ireland's IAF couples a duty of responsibility with enforceable **conduct standards** ✅.

Clarity is the mechanism; culture is the objective. That is the same claim an internal DRI model makes in miniature — and it is why the two most common ways to fail are identical in both settings: **naming a person who cannot deliver (§6.1), and documenting a model nobody uses (§6.3).**

---

## 10. The Cymbal Bank Worked Example

⚠ **Cymbal Bank is a fictional institution and this example is illustrative.** It is constructed by this guide to show the design moves end to end; it is not a case study of any real bank, and no real institution's practice is described here. The regulatory mapping refers to real regimes (UK SM&CR, MAS IAC, HK MIC, AU FAR, IE SEAR) because those are the actual supervisory frameworks a Singapore-based institution would answer to.

### 10.1 The Programme and the Decomposition

Cymbal Bank's "Real-Time Ledger Programme" replaces overnight batch posting with continuous posting for retail accounts, across five squads and two vendors, in three releases over four quarters. The decomposition rule: **an outcome is a result with a date, not an area of concern.** Four outcomes, each with one DRI:

| Outcome | Success test | Date | DRI (single name) |
|---|---|---|---|
| O1 — Core ledger service live in production for retail | Continuous posting for 100% of retail accounts; reconciliation variance zero for 20 consecutive business days | Q2 | Payments platform architect |
| O2 — Downstream reporting adapt to continuous posting | All six regulatory and management reports regenerated with no manual post-run adjustment | Q2 | Data engineering lead |
| O3 — Two legacy batch jobs decommissioned | Batch jobs switched off; no fallback dependency invoked for 30 days | Q3 | Application owner, core banking |
| O4 — Operational readiness for the new service | Service in the operational resilience register; runbooks and incident roles exercised twice | Q3 | Operational resilience manager |

### 10.2 Naming the DRIs

The decomposition is done **openly with the managers whose people are named**, before the charters are drafted, and each prospective DRI is asked one question in writing: *"to deliver this, what must you be able to decide, and what resources must be committed?"* The answers become the charter's mandate and resource clauses. Two of the four asked for a resource commitment they did not have; one outcome was re-scoped rather than assigned with the gap intact — the §6.1 guardrail, applied at design time.

### 10.3 The Charters and the Decision-Rights Matrix

Each outcome gets the §5.9 charter, plus one decision-rights line per decision class, so that "who decides" is never inferred:

| Decision class | DRI | Approver (single name) | Consulted |
|---|---|---|---|
| Release go/no-go | O1–O4 DRIs jointly report readiness | Change board chair | Risk, resilience, internal audit (observer) |
| Rollback during a release | The release DRI | None (DRI decides; recorded) | Incident commander if declared |
| Change to the target date | None (DRI cannot self-authorise) | Programme sponsor | Executive sponsor for the business line |
| Registration of a new service in the resilience register | O4 DRI | Resilience committee chair | — |

### 10.4 Mapping Internal DRIs to Regulatory Accountable Roles

The reconciliation step from §9.1, performed explicitly on the programme's one-page map: each outcome DRI is mapped to the senior manager whose allocated responsibilities the outcome falls inside. O1→Head of Payments Technology (an approved senior manager in the Singapore entity); O2→Head of Data; O3 and O4→Head of Technology Operations. **None of the four DRIs is an accountable person**, and the map records that fact deliberately: the accountability sits with the named senior manager, the delivery ownership with the DRI, and the delegation from one to the other is written down. Under MAS IAC the mapping is an internal instrument supporting the five outcomes; for the group's UK branch and its Irish subsidiary, the same programme triggers SoR and SEAR mapping respectively, and the branch/subsidiary accountable persons are named in the local responsibility maps rather than in the programme plan.

### 10.5 The Anti-Pattern Guardrails

- **The accountability-without-authority check (§5.9 item 9) is signed at charter approval** — by the DRI, the approver and the line manager — and re-signed at every date change.
- **A DRI load register** (§6.2) shows all outcomes per person; nobody holds more than three.
- **One name per outcome**; where O3 and O4 formally touch the same team, the dependency is recorded in each charter rather than resolved by adding a co-DRI.
- **A private weekly channel** from each DRI to the programme lead for blockers, separate from the steering forum (§6.7), so escalation does not require a public admission.
- **A standing rule**: a DRI who reports a slip before the date passes is treated as having done their job.

### 10.6 How the Model Is Measured

Four measures, all cheap, all backwards-looking except the first: (i) the **DRI confidence question** — asked privately each month, "can you resource and decide what your charter says?" (§6.7); (ii) **escalation timeliness** — proportion of slips reported before the target date rather than after; (iii) **ownership continuity** — open outcomes with no gap in a named owner, from the quarterly sweep (§6.6); (iv) **charter-to-record consistency** — whether the plan, the responsibility map and the change records name the same person.

### 10.7 What the Model Cannot Fix

The model **cannot** fix structural under-resourcing (naming an owner does not create capacity); **cannot** fix an unclear strategy (a DRI cannot decide what the programme is for); **cannot** fix contradictory reporting lines (two managers giving conflicting instructions is a governance defect, not an ownership defect, and §5.8 rule 4 sends it up rather than absorbing it); and **cannot** fix a culture in which being named is itself a punishment (§6.5). In each case the honest move is to name the structural problem as the deliverable — **the DRI for "the two reporting lines conflict" is an escalation, not a person**.

---

## 11. The Claims Audit

| Claim | Status | Source | Source quality | Date |
|---|---|---|---|---|
| The term DRI is documented in print for Apple's internal convention, with an agenda practice and the question "Who's the DRI on that?" | **VERIFIED** | Lashinsky, Fortune | Secondary journalism, anonymous sourcing | 25 Aug 2011 |
| The same author expanded the material into a book | **VERIFIED** | Lashinsky, *Inside Apple* | Secondary; same author/reporting | 2012 |
| Apple invented the DRI / the practice of single-named ownership | **REJECTED** | — | Old project/engineering/military practice; no primary Apple document | — |
| Steve Jobs invented or used the term, or said it | **REJECTED** | — | No primary quotation located; the article's Jobs quotes concern agendas | — |
| Disney originated the DRI | **REJECTED** | — | No dated documentation located | — |
| An earlier (pre-2011) dated use of the term exists | **NOT FOUND** | — | Search shape stated in §2.6 | — |
| Diffusion of responsibility / bystander intervention findings | **VERIFIED (Crossref)** | Darley & Latané 1968; Latané & Darley 1968; Latané & Rodin 1969; Latané 1981 | Peer-reviewed, primary | 1968–1981 |
| Social loafing, with identifiability as a moderator | **VERIFIED (Crossref)** | Latané, Williams & Harkins 1979; Karau & Williams 1993 | Peer-reviewed, primary + meta-analysis | 1979, 1993 |
| Ringelmann's 1913 rope study was French, agricultural-engineering, rediscovered in English in 1986, and over-read as social loafing | **VERIFIED (attribution fix)** / interpretation **FLAGGED** | Kravitz & Martin 1986 | Peer-reviewed rediscovery and translation | 1986 |
| Accountability has two modes: pre-decisional conformity vs post-decisional complexity | **VERIFIED (Crossref)** | Lerner & Tetlock 1999; Tetlock 1983, 1985 | Peer-reviewed review + primary experiments | 1983–1999 |
| Specific, difficult goals with feedback outperform vague goals | **VERIFIED (Crossref)** | Locke & Latham 2002 | Peer-reviewed 35-year review | 2002 |
| The bystander effect is strongest in non-dangerous emergencies and weak/absent in dangerous ones | **VERIFIED (Crossref)** | Fischer et al. 2011 | Peer-reviewed meta-analysis | 2011 |
| Manning, Levine & Collins (2007) is in *American Psychologist*, not *BJSP*, and debunks the "38 witnesses" story | **VERIFIED** | Manning et al. 2007 | Peer-reviewed; journal confirmed at DOI | 2007 |
| RACI is the closest analogue to a DRI in mainstream PM practice; its "A" is the hedged single-owner role | **VERIFIED** | Wikipedia/RAM article carrying PMI/BABOK citations | Tertiary carrier of primary citations | accessed 2026 |
| RACI's named originator | **FLAGGED — unverified** | — | No dated primary document located | — |
| DACI's Approver is deliberately one person | **VERIFIED** | Atlassian team-playbook page | Vendor practitioner documentation | accessed 2026 |
| DACI originated at Intuit | **FLAGGED — unverified** | — | No Intuit primary document located | — |
| The Scrum Product Owner is "one person, not a committee" and accountable for the backlog | **VERIFIED** | Scrum Guide 2020 | Methodology primary text | 2020 |
| The incident commander holds all undelegated roles; keeps a living doc; hands off with explicit acknowledgement | **VERIFIED** | Google SRE Book ch.14 | Practitioner primary text | 2016 |
| Amazon's single-threaded leader is documented in *Working Backwards* | **FLAGGED — knowledge, not re-verified** | Bryar & Carr 2021 | Book; Crossref returned no record | 2021 |
| CODEOWNERS permits teams as owners and can require code-owner approval before merge | **VERIFIED** | GitHub docs | Vendor primary documentation | accessed 2026 |
| UK SM&CR: in place since 2016; SMF approval under s.59 FSMA; Statement of Responsibilities required and maintained; certification functions under s.63E; Conduct Rules bind individuals | **VERIFIED** | PRA Rulebook Parts; BoE PS12/26, PS16/15 | Regulator primary sources | 2015–2026 |
| UK SM&CR Phase 1 reforms (2026) and Phase 2 legislative changes including reduced SoR requirements | **VERIFIED** | BoE PS12/26; HMT consultation response | Regulator and government primary sources | 22 Apr 2026 |
| UK extension to FCA solo-regulated firms (Dec 2019) and to the wider sector (FSMA 2023) | **FLAGGED — not re-verified** | PRA Rulebook version dated 09/12/2019 | Indirect corroboration only | 2019 |
| Singapore MAS IAC: guidelines, published 10 Sep 2020, five high-level outcomes, broad sector coverage, breach not itself an offence | **VERIFIED** | mas.gov.sg | Regulator primary source | 10 Sep 2020 |
| Hong Kong: MIC expectations carried in SPM CG-1; current issue 1 Aug 2025; CG-1 V3 archived 6 Oct 2017 | **VERIFIED** | HKMA CG-1 landing page and version history | Regulator primary source | 2017–2025 |
| Hong Kong: MIC designated per core function, organogram published, appointments notified to the HKMA | **FLAGGED — detail not re-verified** | — | Widely-documented, module text not extracted | — |
| Australia FAR: replaces BEAR (2018); jointly administered APRA/ASIC; ADIs and NOHCs from 15 Mar 2024; insurance, licensed NOHCs and superannuation trustees from 15 Mar 2025; FAR Act 2023 | **VERIFIED** | apra.gov.au | Regulator primary source | 2024–2025 |
| Australia FAR: accountable persons, accountability statements/maps, penalty quantum | **FLAGGED — mechanism widely documented, penalty detail not verified** | APRA guidance/template | Regulator primary source (partial) | 2024 |
| Ireland IAF: IAF Act 2023 signed 9 Mar 2023, partially commenced 19 Apr 2023, remainder 29 Dec 2023; SEAR for executive PCF roles from 1 Jul 2024; Conduct Standards since 29 Dec 2023; certification since 8 Jan 2024; ASP enforcement directly against individuals | **VERIFIED** | centralbank.ie | Regulator primary source | 2023–2024 |
| A DRI ritual (a name beside an agenda action item) improves outcomes | **REJECTED — no evidence** | — | No study located; not tested | — |

---

## 12. What Could Not Be Verified

Stated plainly, in the order that matters:

1. **A pre-2011 dated attestation of "DRI" as a named convention.** The search covered books and the printed record, digitised and archive collections, engineering/military documents and the Linux `MAINTAINERS` history; **nothing earlier than the 2011 Fortune article was located.** The guide therefore reports no earlier attestation rather than asserting that the term was coined in 2011.
2. **Any primary Apple document, or a named first-hand Apple source, defining the DRI.** No Apple corporate documentation, and no Steve Jobs quotation on the DRI, was located. The provenance rests on a single journalist's account with anonymous sourcing.
3. **Any named first-hand account by an Apple veteran specifically on the DRI** (for instance a dated blog post by a former Apple employee) — **not located this pass.** Books such as Gallo's *The Apple Experience* (2012) and Segall's *Insanely Simple* (2012) are secondary and were not obtained as full text; **nothing in this guide relies on them.**
4. **The full text of the HKMA's SPM module CG-1.** The vehicle, current issue date and the 2017 revision are verified; **the module's operational MIC requirements were not extracted**, so every MIC mechanic in §8.4 is flagged ⚠. Anything asserting a specific list of HKMA core functions should be checked against CG-1 directly.
5. **The extension of the UK SM&CR to FCA solo-regulated firms (December 2019) and the FSMA 2023 extension to the wider authorised sector.** fca.org.uk and legislation.gov.uk were unreachable to the scraper this run; the PRA Rulebook's 09/12/2019 snapshot is corroborative, not probative. The specific dates are flagged ⚠.
6. **The Australian FAR's civil penalty provisions and quantum**, and the **precise form of registration of accountable persons** — APRA's pages confirm the statutory basis, the commencement dates and the guidance/template, but not the penalty detail. Flagged ⚠.
7. **RACI's originator and first publication**, and **DACI's attribution to Intuit.** Neither could be pinned to a dated primary source this pass. Both are flagged ⚠ and treated as unverified folklore.
8. **Any study testing a DRI ritual, charter, responsibility map or decision-rights matrix as an intervention.** None was located, and the guide asserts plainly (§4.6) that no such study exists on the evidence found rather than implying that one might.
9. **The Amazon single-threaded-leader claim at primary-source level.** *Working Backwards* is the standard citation; a Crossref query for the title returned no matching record and no Amazon primary document was fetched, so the model is presented as ⚠-knowledge.

---

## 13. Glossary

| Term | Meaning (as used in this guide) |
|---|---|
| **Accountable person** | The regulatory term (AU FAR; FR/UK usage varies) for an individual registered as accountable for a function or area under a statutory regime (§8.5) |
| **Accountability without authority** | The central anti-pattern: being answerable for an outcome whose resources, staff or decisions lie with someone who has not agreed to the arrangement (§6.1) |
| **Certification regime** | The limb of a regulator's model in which the *firm* certifies an individual as fit and proper for a significant-harm role, without individual regulatory approval (UK s.63E; IE Certification Regulations) (§8.2, §8.6) |
| **Charter (DRI charter)** | This guide's constructed one-page artefact naming the outcome, the single DRI, the mandate, decision rights, escalation and resources — **constructed here, not an industry standard** (§5.9) |
| **Conduct Rules / Conduct Standards** | Individual rules binding persons (not only firms) in UK SM&CR and the Irish IAF (MAS uses "conduct standards" in the IAC outcomes) (§8.2, §8.6) |
| **Diffusion of responsibility** | The bystander-intervention finding that the presence of others reduces the felt obligation of any one person to act (Darley & Latané 1968) (§4.2) |
| **DRI** | Directly responsible individual — **the single named owner of one outcome**; a repo-standard term defined by this guide (§1) |
| **Duty of Responsibility** | The legal duty imposed on PCF role-holders in in-scope firms under Ireland's SEAR (§8.6) |
| **FAR / BEAR** | Australia's Financial Accountability Regime (2024/2025) and its predecessor, the Banking Executive Accountability Regime (2018) (§8.5) |
| **IAC** | MAS's Guidelines on Individual Accountability and Conduct (10 Sep 2020) — guidance, five high-level outcomes (§8.3) |
| **IAF / SEAR** | Ireland's Individual Accountability Framework and its Senior Executive Accountability Regime (§8.6) |
| **Incident commander** | The single role commanding an incident response; holds all positions not delegated; keeps the living incident document; hands off explicitly (§3.7) |
| **MIC** | Managers-In-Charge — the HKMA's named-individual expectations, carried in SPM module CG-1 (§8.4) |
| **Post-decisional accountability** | Accountability to an audience whose views are unknown until justification, producing more self-critical processing (Lerner & Tetlock 1999) (§4.4) |
| **Pre-decisional accountability** | Accountability to a known-preferring audience before the decision, producing conformity (§4.4) |
| **SM&CR** | The UK's Senior Managers and Certification Regime — SMF approval, Statements of Responsibilities, certification, Conduct Rules (§8.2) |
| **Social loafing** | Reduced individual effort on collective tasks when individual contribution is not identifiable; reduced by identifiability and other moderators (§4.3) |
| **Single-threaded leader** | Amazon's organisational-design variant: one leader, one thing, no competing priorities (§3.5) |
| **Statement of Responsibilities (SoR)** | The statutory documents of what a named senior manager is responsible for; required with UK SMF approval and maintained at all times (§8.2) |

---

## 14. Cross-references and Further Reading

**Within `management/`:** [the_managers_path_guide.md](the_managers_path_guide.md) (glossary entry for the DRI as a repo-standard term; §12.6 delegation); [high_performing_team_first_time_manager_guide.md](high_performing_team_first_time_manager_guide.md) (§12.2, the DRI habit as one of five accountability components); [authority_skills_guide.md](authority_skills_guide.md) (§6 decision rights; §11.3 power distance); [organizational_behavior_guide.md](organizational_behavior_guide.md) (power and accountability foundations); [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) (RACI mechanics); [product_management_frameworks_guide.md](product_management_frameworks_guide.md); [the_first_90_days_guide.md](the_first_90_days_guide.md); [management_case_study_guide.md](management_case_study_guide.md).

**Within `../banking/`:** [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §4.5 — **owns the MAS IAC detail**; [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md).

**Within `../technology/`:** [../technology/architecture_decision_record_guide.md](../technology/architecture_decision_record_guide.md) (the design-doc owner variant); [../technology/api_governance_guide.md](../technology/api_governance_guide.md); [../technology/data_governance_guide.md](../technology/data_governance_guide.md).

**Primary and near-primary sources fetched this pass:** Lashinsky, Fortune, 25 Aug 2011; scrumguides.org (2020 Scrum Guide); Google SRE Book ch.14 "Managing Incidents"; docs.github.com "About code owners"; mas.gov.sg IAC guidelines page; apra.gov.au FAR pages; centralbank.ie IAF page; hkma.gov.hk CG-1 landing page and version history; prarulebook.co.uk (Allocation of Responsibilities; Senior Management Functions; Certification; Conduct Rules; SS28/15); bankofengland.co.uk PS12/26; gov.uk HMT SM&CR consultation response; Atlassian team-playbook DACI; Wikipedia responsibility-assignment-matrix (tertiary carrier). **Research records confirmed at DOI level via the Crossref API:** the thirteen citations listed in §4 — Darley & Latané 1968; Latané & Darley 1968; Latané & Rodin 1969; Latané 1981; Latané, Williams & Harkins 1979; Kravitz & Martin 1986; Karau & Williams 1993; Fischer et al. 2011; Manning, Levine & Collins 2007; Lerner & Tetlock 1999; Tetlock 1983; Tetlock 1985; Locke & Latham 2002. **Flagged and not relied upon:** Bryar & Carr, *Working Backwards* (2021); Gallo, *The Apple Experience* (2012); Segall, *Insanely Simple* (2012).

---

## 15. Closing Summary

The DRI is the smallest useful unit of organisational accountability: **one name, one outcome, written down where the work is read.** Its documented life begins in 2011, in a journalist's account of Apple's internal vocabulary — Apple-popularised, not Apple-originated, and the folklore that grew around it ("Jobs invented it", "Apple invented it") does not survive contact with the sources. The research underneath it is real and narrower than the practice claims: groups do diffuse responsibility, identifiability does reduce loafing, accountability does change how people think when the mode is post-decisional rather than pre-decisional, and specific attributed goals do beat vague ones — but no study tests the ritual, and the accountability literature's own founding anecdote, the thirty-eight witnesses, turned out to be a parable. Banking regulators arrived at the same principle independently and made it law, in five jurisdictions and three different instruments, with consequences the meeting room never has. The practice fails in recognisable ways — accountability without authority above all — and it succeeds for one unglamorous reason: when the question is asked, there is someone to ask, and one person who knew the question was coming. Every mechanism in this guide, from the charter to the sweep, exists to protect that one thing. The whole practice is, in the end, the single name.
