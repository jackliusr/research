# The Management Case Study: The Method, the Canon, and the Practice

> A deep-dive on the management case study in all three of its lives — as a teaching instrument (the Harvard Business School case method), as a research methodology (the Yin / Eisenhardt / Stake tradition), and as a professional practice (case anatomy, case analysis, case discussion, case writing) — with a full worked Cymbal Bank teaching case and an instructor's analysis.

**Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
**Context:** Professional Development / Management & Leadership Series
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
**Last Updated:** September 2026

> **What this guide is.** A dedicated reference on the management case study as a form — its provenance, its two distinct epistemologies (teaching versus research), its anatomy, the analytical disciplines it demands, the classroom practice that animates it, and the case-interview genre that borrows its surface and tests something different. It ends with a complete, self-contained Cymbal Bank decision-point case, exhibits and teaching note included, so the abstractions in sections 2–9 have a worked instance to land on.

> **How to use this guide.** Read §1–§2 for the method and its history; §3–§4 if you care about the *research* case study (a different animal, and the most under-taught part of this topic); §5–§6 for the anatomy of a case and the teaching note; §7–§8 for analysis and classroom practice; §9 for the consulting case-interview cross-reference; §10 as a complete worked exemplar you can teach from. The claims-audit table in §11 and the "What Could Not Be Verified" section in §12 are the verification layer — consult them before quoting anything here.

> **What this guide is not.** It does not re-derive strategy frameworks (see `management/strategic_management_guide.md`), the MBA curriculum's treatment of the case method (see `management/mba_body_of_knowledge_guide.md` §4.1), or the case-interview taxonomy (see `management/management_consulting_skills_guide.md` §7) — it summarises and cross-references those. It is not a guide to writing an internal investment *business case* for a project or decision; that is a different artefact with a different purpose and is covered in `management/business_case_development_guide.md`. It does not re-derive the Balanced Scorecard (`management/balanced_scorecard_guide.md`), the personal-MBA reading path (`management/personal_mba_companion_guide.md`), or organisational-behaviour theory (`management/organizational_behavior_guide.md`). For the engineering-interview analogue of the case-interview genre, see `technology/system_design_interview_insiders_guide.md`, `technology/ml_system_design_interview_guide.md`, and `technology/ddia_study_companion_guide.md`.

> **A note on verification.** Claims are tagged ✅ (verified at a primary or authoritative source), ⚠ (flagged — contested, practitioner-sourced, school-specific, or otherwise not authoritative), or ❌ (could not be verified). The case-study method's origin story is unusually prone to embellishment: it is retold in institutional marketing, alumni memoir, and secondary history, and the details drift. Where the historical record is genuinely contested, this guide says so rather than choosing a tidy version. Nothing here should be cited as a primary source for a historical claim — follow the links.

---

## Table of Contents

1. [The Case Study as Two Things](#1-the-case-study-as-two-things)
2. [The Teaching Case: Origin and Purpose](#2-the-teaching-case-origin-and-purpose)
3. [The Research Case Study: The Yin / Eisenhardt Tradition](#3-the-research-case-study-the-yin--eisenhardt-tradition)
4. [Teaching Case vs Research Case: A Reconciliation](#4-teaching-case-vs-research-case-a-reconciliation)
5. [The Anatomy of a Management Case](#5-the-anatomy-of-a-management-case)
6. [The Teaching Note](#6-the-teaching-note)
7. [Case Analysis: The Discipline and Its Frameworks](#7-case-analysis-the-discipline-and-its-frameworks)
8. [Leading the Case Discussion](#8-leading-the-case-discussion)
9. [The Case-Interview Genre: A Cross-Reference](#9-the-case-interview-genre-a-cross-reference)
10. [Worked Example: The Cymbal Bank Case](#10-worked-example-the-cymbal-bank-case)
11. [Claims Audit](#11-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Sources and Further Reading](#14-sources-and-further-reading)

---

## 1. The Case Study as Two Things

### 1.1 The Two Meanings, Stated Plainly

The phrase "case study" names two artefacts that share a family resemblance and almost nothing else operationally.

| Dimension | The **teaching** case | The **research** case |
|---|---|---|
| Purpose | Develop judgment in students | Produce knowledge / theory |
| Audience | Students, in a classroom | Peers, in a journal or monograph |
| Author's stance | Neutral presenter of a dilemma | Analyst with an explicit argument |
| Ending | Deliberately unresolved (a decision point) | Analytically resolved (findings) |
| Evidence standard | Plausible, disguised, sufficient to support discussion | Defensible, traceable, methodologically argued |
| Generalisation | Does not claim any | Analytic generalisation (Yin) or theory-building (Eisenhardt) |
| Primary outlet | HBS / Ivey / Darden / The Case Centre | Academic journals, books |
| Anonymity | Often disguised, composite, or fiction-lite | Usually named, subject to ethics approval |

This distinction is the single most important thing in this guide. Conflating the two produces bad teaching cases (over-argued, pre-resolved) and bad research cases (under-theorised, anecdotal). The teaching case is a *pedagogical object*; the research case is a *methodological choice*.

### 1.2 Why Both Matter to a Working Professional

For a working manager or architect:

- **The teaching case** is a thinking tool. Reading and discussing cases builds pattern recognition for ambiguous situations — which is most of what "management" actually is (⚠ framing; this is the standard HBS rationale, not a measured finding).
- **The research case** is a reasoning tool. When you are asked to justify a decision on the basis of a single rich situation ("we saw this work at one bank"), the research-case literature is what tells you whether that inference is legitimate. Mostly, it is not — and knowing *why* saves you from a category error.
- **The case interview** tests a related but distinct skill: live, individual, structured reasoning about an unseen problem.

### 1.3 How the Three Connect

```
                        ┌────────────────────────────┐
                        │     THE CASE STUDY         │
                        │     (three lives)          │
                        └─────────────┬──────────────┘
          ┌───────────────────────┬───┴────────────────────────┐
          ▼                       ▼                            ▼
   TEACHING INSTRUMENT     RESEARCH METHODOLOGY          HIRING DEVICE
   (HBS case method,       (Yin, Eisenhardt, Stake,      (consulting case
    the "case discussion")  Flyvbjerg)                    interview)
          │                       │                            │
   §2, §5–§6, §8             §3–§4                        §9
          │                       │                            │
          └───────────────┬───────┴────────────────────────────┘
                          ▼
                 WORKED INSTANCE (§10)
                 the Cymbal Bank case + teaching note
```

---

## 2. The Teaching Case: Origin and Purpose

### 2.1 The Law-School Lineage (✅ verified, with caveats)

The case method as a *pedagogy* was not invented at Harvard Business School. It was imported from the law school:

| Element | Detail | Status |
|---|---|---|
| Originator | Christopher Columbus Langdell, Dean of Harvard Law School | ✅ verified |
| Method | The "casebook" — students read appellate decisions and derive principles by discussion rather than by lecture | ✅ verified |
| Timing | Langdell's deanship began 1870; the casebook method dates from the early 1870s | ✅ verified |
| Mechanism | Langdell's premise: law is a science, and the materials of that science are the recorded cases; students should learn from the primary materials, inductively | ✅ verified |

Langdell's innovation was to put the raw materials — the decided cases — in front of students and make them do the reasoning, instead of lecturing the distilled doctrine. That logic is exactly the logic HBS later adopted.

### 2.2 The HBS Adoption (✅ verified core; ⚠ contested detail)

| Element | Detail | Status |
|---|---|---|
| Adopter | Wallace Brett Donham, second Dean of HBS (deanship 1919–1942) | ✅ verified |
| Trigger | A memorandum on the teaching of business, conventionally dated 1920, arguing that the business of business could not be taught by lecture alone | ✅ verified in substance; ⚠ exact wording and date variably reported |
| First HBS case | The **General Shoe Company** case, written in the early 1920s | ✅ verified |
| Institutionalisation | The case method became the backbone of the HBS MBA, and HBS became its global exporter | ✅ verified between the two world wars onward |

### 2.3 Why the Origin Story Is Contested (⚠ flagged)

The popular retelling of the case method's birth is embellished in several ways, and a careful reader should treat the details as folklore until checked against HBS Baker Library's own archival record:

- **The "first case" date and author.** The frequently-repeated claim that a particular person wrote the first case at a kitchen table on a particular date is **not reliably documented** and varies by telling. The General Shoe Company case is the conventionally cited first HBS case; *who* wrote it, exactly *when*, and under what title are reported inconsistently. ⚠
- **The Copeland attribution.** The attribution of the earliest case-writing to Melvin Copeland appears in later HBS-internal histories and in the "100 years" retrospective material, but it is not established to a primary-source standard in accessible web sources. ⚠
- **The strength of the law-school lineage.** That Langdell's method influenced Donham's argument is well established in substance. *How direct and how decisive* that influence was — versus a parallel response to the demand for a practical, non-lecture business education — is debated among historians of the field. The lineage is real; the causal story is not settled. ⚠
- **The "invented at HBS" claim.** The case method in business education had parallel and earlier expressions elsewhere (e.g. in medical and other professional education and in some early commerce schools). HBS popularised and systematised it; it did not conjure it from nothing. ⚠

**Practitioner takeaway.** Cite the *structure* of the story (Langdell → Donham → HBS at scale → global export) with confidence. Cite the *specific anecdotes* only with a flag.

### 2.4 HBS's Own Stated Purpose for the Method (✅ verified at hbs.edu)

HBS's Christensen Center for Teaching and Learning frames the method in its own words. Verified points:

| Claim | Status | Source / note |
|---|---|---|
| Chris Christensen described case method teaching as **"the art of managing uncertainty"** | ✅ verified | hbs.edu Christensen Center case-method page |
| The instructor is described as **"planner, host, moderator, devil's advocate, fellow-student, and judge"** | ✅ verified | same page (quoted on hbs.edu) |
| Case method classes **unfold without a detailed script**; teachers "balance planning and spontaneity" | ✅ verified | same page |
| The method depends on **student-centred discussion**, not lecture | ✅ verified | same page + leading-a-case-discussion |
| The goal is **discovery**, with instructors guiding students toward it across multiple levels of learning | ✅ verified | same page |
| The method emphasises **decision-making under ambiguity**, with no single right answer — this is the standard framing | ✅ verified in substance | HBS method pages + case-publisher guidance |

The phrase most worth internalising is Christensen's: the case method is "the art of managing uncertainty." The instructor's job is not to deliver the answer but to run a discussion in which the answer is *discovered*, contested, and revised. That is the whole pedagogy in one sentence.

### 2.5 The Canon in One Paragraph

The teaching case is a narrative artefact — typically 5–25 pages of text plus exhibits — that drops the reader into a real (or disguised) managerial situation at a decision point, with enough data to reason and not enough to be certain. The reader is asked to occupy the protagonist's seat, identify the actual problem (which is rarely the presenting problem), generate and evaluate alternatives, recommend a course of action, and defend it under questioning. The instructor's art is to orchestrate that defence collectively, without supplying the answer, so that the class builds the judgment the lecture would have merely described. Everything else in this guide elaborates one clause of that paragraph.

---

## 3. The Research Case Study: The Yin / Eisenhardt Tradition

### 3.1 Why This Section Exists

The research case study is where most of the *intellectual* content of this topic lives, and it is where the repo previously had no coverage at all. It is also where the traps are: "case study" as a research design is much more methodologically disciplined than its reputation suggests, and much more contested than its textbooks admit.

### 3.2 Yin: *Case Study Research and Applications: Design and Methods* (✅ verified)

| Element | Detail | Status |
|---|---|---|
| Author | Robert K. Yin | ✅ verified |
| Publisher | Sage | ✅ verified |
| First edition | 1984 | ✅ verified |
| Later editions | Multiple; the sixth edition (2018) re-titled *Case Study Research and Applications: Design and Methods*; a seventh edition exists in recent years | ✅ verified |
| Title history | Early editions titled *Case Study Research: Design and Methods*; the "and Applications" appears from the 6th edition onward | ✅ verified |
| Yin's core definition | A case study is an **empirical inquiry** that investigates a contemporary phenomenon in depth and within its real-world context, especially when the boundaries between phenomenon and context are not clearly evident | ✅ verified (definition reproduced across editions) |
| Question framing | Case studies suit **"how" and "why"** questions | ✅ verified |
| Design types | **Single-case** and **multiple-case** designs | ✅ verified |
| Multiple-case logic | **Replication logic** (literal and theoretical replication), explicitly *not* sampling logic | ✅ verified |
| Role of theory | Theory development before data collection; theory as the vehicle of generalisation | ✅ verified |
| Validity tactics | Construct validity, internal validity, external validity, reliability — each with named tactics | ✅ verified |
| Rival explanations | Explicitly required as a test of internal validity | ✅ verified |
| Generalisation | **Analytic generalisation** (to theory), not statistical generalisation (to a population) | ✅ verified |

Yin's most consequential move is the last row. A case study does not generalise to a population the way a survey does — you cannot reason from one bank to all banks. It generalises *to theory*: the case is one more test of a proposition, and the accumulation of such tests is how the theory earns confidence. Anyone who says "but it's only one case" has misunderstood the design; equally, anyone who says "we saw it at one bank so it's true everywhere" has committed the error Yin's framework exists to prevent.

### 3.3 Eisenhardt: Theory Building from Cases (✅ verified citation)

| Element | Detail | Status |
|---|---|---|
| Author | Kathleen M. Eisenhardt | ✅ verified |
| Title | "Building Theories from Case Study Research" | ✅ verified |
| Journal | *Academy of Management Review* | ✅ verified |
| Year / volume / pages | 1989, **14**(4): **532–550** | ✅ verified |
| Follow-up | Eisenhardt & Graebner, "Theory Building from Cases: Opportunities and Challenges", *Academy of Management Journal*, 2007, **50**(1): **25–32** | ✅ verified |
| Core contribution | A roadmap for using multiple cases to build theory — from research question, through case selection, to cross-case analysis and theory development | ✅ verified in substance |

Eisenhardt's paper is the single most-cited "how to do it" statement for case-based theory building, and it codified the move to **multiple-case** designs and **cross-case analysis**. Where Yin is a design text, Eisenhardt is a theory-building text — she is explicit that the goal is not description but the generation of testable constructs and propositions.

Eisenhardt's distinctive claims, roughly:

1. **Cases are a route from raw richness to theory**, not an end in themselves.
2. **Theory should emerge** from the data — constructs and propositions are built inductively, though the researcher brings prior theory in as a sensitising device.
3. **Multiple cases are usually preferable** to a single case for theory building; each case is a "replication" in Yin's sense.
4. **Cross-case analysis** is the engine: comparing cases surfaces constructs and relationships that a single case hides.
5. The output is a **testable** set of propositions — falsifiable, not just interesting.

### 3.4 The Methodological Debates (⚠ flagged throughout)

The research-case literature is not a consensus. The main fault lines:

| Debate | Positions | Status |
|---|---|---|
| **Yin vs Stake** | Yin is comparatively **positivist/postpositivist** (design, validity tactics, replication, analytic generalisation). Robert E. Stake is comparatively **interpretivist/constructivist** (intrinsic case study, the case as a bounded system, naturalistic generalisation, the particularity of the case). | ✅ verified as a distinction; ⚠ the labels oversimplify both authors |
| **Generalisation** | Statistical (to a population) vs analytic (to theory, Yin) vs naturalistic (Stake) vs theory-building (Eisenhardt). | ✅ verified as competing positions |
| **The small-N critique** | The charge that N=1 or N=4 is "unscientific" and cannot produce generalisable knowledge. | ✅ verified as the standard critique |
| **Flyvbjerg's rebuttal** | Bent Flyvbjerg's "Five Misunderstandings About Case-Study Research" argues the critique rests on five misconceptions (e.g. that generalisability is the only valuable knowledge, that you cannot generalise from a single case, that the case study is only useful for hypothesis generation rather than testing). | ✅ verified as published (Qualitative Inquiry, 2006, 12(2): 219–245); ⚠ the argument is itself contested |
| **Creswell's stance** | In the mixed-methods tradition, John W. Creswell treats case study as one of several *qualitative inquiry approaches*, positionable between positivistic and interpretive designs depending on the author's output. | ⚠ flagged — his placement is a synthesising view, not a settlement |
| **Eisenhardt vs the interpretivists** | Eisenhardt's "process of building theory from case study research" is criticised by interpretive scholars for being quasi-positivist — multiple cases, replication, propositions. | ✅ verified as a live debate |

**The honest position.** There is no single authoritative "correct" case-study methodology. There are competing schools, each with a coherent internal logic and each with a critique from the others. A research case study should state *which* school it follows and defend that choice — a study that mixes them without saying so is the actual methodological error, not the choice of any one school.

### 3.5 The Research Case in One Table

| Question you are asking | Suitable design | Authority |
|---|---|---|
| Why did this happen? How did this process unfold? | Single or multiple case, explanatory | Yin |
| What is this case intrinsically — what is it like to be inside it? | Intrinsic case study | Stake |
| Can I build a testable theory from several rich sites? | Multiple case, cross-case analysis | Eisenhardt |
| Can a single case *test* (not merely generate) a proposition? | Critical case / extreme case / information-oriented selection | Flyvbjerg |
| How does case study relate to surveys and experiments? | Mixed-methods triangulation | Creswell |

---

## 4. Teaching Case vs Research Case: A Reconciliation

### 4.1 The Two Are Genuinely Different Artefacts

Yin's own sixth edition sharpens the distinction: the 2018/2017 edition explicitly **"sharpens discussion of distinguishing research from non-research case studies"** — i.e. Yin himself treats the teaching case as a *non-research* case, not a weaker research case (✅ verified at the Sage product page listing the new-to-this-edition features).

| Axis | Teaching case | Research case |
|---|---|---|
| **Purpose** | Develop judgment | Produce knowledge |
| **Epistemic goal** | Practice under ambiguity | Explanation or theory |
| **Author stance** | Neutral; withhold the answer | Analytic; state the argument |
| **Ending** | Decision point, unresolved | Findings, discussion, conclusion |
| **Evidence** | Enough to reason; disguised; exhibits may be simplified | Traceable; protocol-governed; chain of evidence |
| **Method** | None in the social-science sense | Design, protocol, coding, triangulation, rival explanations |
| **Generalisability claim** | None | Analytic (Yin) or theory-building (Eisenhardt) |
| **Anonymity** | Common; disguise is standard practice | Ethics-governed; names often retained with consent |
| **Length** | 5–25 pages + exhibits | Varies wildly; often a full article or monograph |
| **Review** | Editorial / teaching-quality | Double-blind peer review |
| **Outlet** | HBS, Ivey, Darden, The Case Centre, Emerald | Journals, books, dissertations |
| **Success criterion** | Did the discussion produce learning? | Did the analysis survive scrutiny? |

### 4.2 Where They Overlap

They are not opposites, and the overlap is instructive:

- **Both are particular.** Both refuse the average and study the specific situation in its context. Yin's emphasis on the phenomenon-in-context is structurally the same instinct as the case's insistence on a real protagonist in a real situation.
- **Both demand evidence discipline.** A teaching case whose numbers do not add up is a bad teaching case; a research case whose chain of evidence is broken is invalid.
- **Both are written artefacts.** The craft of clear, structured, exhibit-supported prose is shared.
- **Both resist the "one right answer" reflex.** Yin allows rival explanations; the teaching case exists precisely because there is no single right answer (✅ verified — HBS method pages).

### 4.3 The Three Practical Differences That Matter Most

1. **The research case must defend its method; the teaching case must defend its pedagogy.** A research case that does not explain case selection, data collection, and analysis strategy is unpublishable. A teaching case that does not make for a good 80-minute discussion is unsellable — its method section is replaced by a teaching note.
2. **The research case may name; the teaching case usually disguises.** Teaching cases routinely alter names, dates, and figures to protect the firm and to sharpen the dilemma; the resulting artefact is often described as "disguised" or "composite." Research cases face ethics review and informed-consent logic instead (⚠ flagged — practice varies by institution and publisher).
3. **The research case is judged by peers; the teaching case is judged by adoption.** A research case's currency is citations. A teaching case's currency is how many schools teach it — which is why case publishers track adoption and why the HBS case catalogue is enormous.

### 4.4 The Business Case (a third thing) — a Necessary Distinction

There is a *third* artefact that uses the same words and is frequently confused with both: the internal **business case** written to justify an investment, a project, or a decision to a governance body. That is a *persuasive internal document with a recommendation the author already believes*; it is structurally the opposite of the teaching case, which is built to withhold the recommendation. It is covered in this repo at `management/business_case_development_guide.md`; do not conflate it with either the teaching or the research case.

| Artefact | Author's goal | Ends with | Audience |
|---|---|---|---|
| Teaching case | Create a learning dilemma | A decision point | Students |
| Research case | Explain / build theory | Findings | Academic peers |
| Business case (internal) | Win approval | A recommendation and a costed plan | A governance body |

---

## 5. The Anatomy of a Management Case

### 5.1 The Standard Skeleton

A well-built teaching case is a highly conventional artefact. The conventions are not arbitrary — each element does pedagogical work.

| # | Element | Function | Typical length |
|---|---|---|---|
| 1 | **Title and header** | Identifies the case, the author, the date, the setting; carries the copyright line | 5–10 lines |
| 2 | **Opening paragraph / hook** | Drops the reader into a concrete moment — often a scene, a quote, a number, a decision already half-made | 1–3 paragraphs |
| 3 | **Protagonist** | A named individual with a role, facing the decision; the reader occupies their seat | thread throughout |
| 4 | **Situation / context** | The company, industry, market, competitive position, recent history | 2–5 pages |
| 5 | **Background** | How the situation arose; organisational history; the people involved | 1–4 pages |
| 6 | **The dilemma / tension** | The substantive problem, usually with competing pressures and incomplete information | 1–3 pages |
| 7 | **The decision point** | The explicit question: *what should [protagonist] do?* — the case must stop here | final 1–2 paragraphs |
| 8 | **Exhibits** | Financial statements, market data, org charts, KPI dashboards, competitor tables, timelines | 1–8 exhibits |
| 9 | **Epilogue** *(optional, sometimes in the teaching note)* | What actually happened — withheld from the student version when it would pre-empt the discussion | short |
| 10 | **Teaching note** | Instructor-only: objectives, analysis, board plan, questions, epilogue | separate document |

### 5.2 The Opening Paragraph — the Most Diagnostically Useful Element

The opening line is where craft is most visible. The conventions:

- **Start in a moment, not in a summary.** "At 7 a.m. on a Tuesday in March, [the protagonist] stared at a spreadsheet that did not add up" beats "This case concerns a bank's retail strategy."
- **Place the protagonist.** Name, role, and the fact that a decision is owed.
- **Signpost the stakes without resolving them.** The reader should feel the pressure and not know the answer.
- **Avoid the thesis.** A case whose opening tells you the answer has already failed.

⚠ Flagged: these are craft conventions drawn from case-writing guidance and from reading the genre, not a single authoritative rule-set. Different publishers phrase the guidance differently; the substance is stable.

### 5.3 The Decision Point — the Load-Bearing Element

The single most common defect in amateur cases is a **missing or muddy decision point**. A case that merely describes a situation is a *profile*, not a case. A case must terminate at a question its protagonist actually had to answer, with:

- a **decision** that is genuinely owed (not hypothetical);
- **alternatives** that are genuinely available and genuinely in tension;
- **incomplete information**, forcing judgment rather than calculation;
- **consequences** that are material and asymmetric.

| Test | A good decision point… | A bad one… |
|---|---|---|
| Urgency | Must be decided now | "Someday, the bank should think about…" |
| Agency | Belongs to a named person | Belongs to "the industry" |
| Tension | Has real trade-offs | Has an obviously correct answer |
| Information | Is under-determined | Can be solved by arithmetic alone |
| Scope | Is narrow enough for one discussion | Spans five disconnected problems |

### 5.4 Exhibits — the Quantitative Spine

Exhibits carry the analytical load. Standard conventions:

| Exhibit type | What it carries | Note |
|---|---|---|
| Financial statements | Income statement, balance sheet, cash flow, sometimes segmental | Often abbreviated or disguised |
| Ratio / performance table | Margins, ROE, cost-income ratio, NPL ratio | Frequently the key exhibit in banking cases |
| Market / segment table | Size, growth, share, profitability by segment | Sets up the "where to play" decision |
| KPI dashboard | Operational metrics, trends over time | Common in operations and digital cases |
| Competitor table | Peer comparison on chosen metrics | Anchors relative position |
| Organisation chart | Reporting lines, decision rights | Anchors the implementation discussion |
| Timeline | Sequence of events, decisions, failures | Anchors the "how did we get here" analysis |
| Customer / survey data | Preferences, NPS, churn, willingness to pay | Anchors pricing and proposition cases |

Design rules (⚠ craft guidance, not a published standard): every exhibit must be *used* by the discussion; data should be internally consistent (a bank case whose numbers do not reconcile is instantly discredited); and figures should be rounded in ways that keep the arithmetic tractable in a classroom.

### 5.5 Types of Management Case

Cases are usually classified by the analytical demand they place on the reader:

| Type | The question it poses | Typical disciplines |
|---|---|---|
| **Decision / dilemma case** | What should the protagonist do? | Strategy, general management |
| **Evaluation / appraisal case** | Was this the right decision? Why did it work or fail? | Strategy, org behaviour |
| **Problem-diagnosis case** | What is actually going wrong here? | Operations, turnaround |
| **Illustration case** | How does this concept look in practice? | Teaching a framework |
| **Quantitative / analytical case** | What do the numbers say, and what should follow? | Finance, valuation, analytics |
| **Industry / background case** | How does this industry work? | Setting context for other cases |

### 5.6 Case-Selection Criteria

Choosing which case to teach is a design decision. The criteria that recur in publisher guidance and pedagogy sources (⚠ synthesised from guidance, not a published checklist verbatim):

| Criterion | Question to ask |
|---|---|
| **Learning fit** | Does it exercise the specific judgment I am teaching this week? |
| **Decision clarity** | Is there a real decision point? |
| **Data sufficiency** | Is there enough to analyse and defend a position? |
| **Ambiguity level** | Is it hard enough to require discussion, easy enough to allow it? |
| **Protagonist access** | Will students identify with and argue from the protagonist's seat? |
| **Context fit** | Is the setting legible to this cohort (industry, geography, regulation)? |
| **Discussion yield** | Can I imagine 60–80 minutes of productive disagreement? |
| **Currency** | Is it still true — or, if dated, does the datedness teach something? |
| **Representation** | Does the case set broaden the protagonists and settings students see? |

### 5.7 Case-Writing: The Practice

Writing a teaching case is reporting, not fiction, even when the names are changed. The recurring stages (⚠ craft consensus, not a single authority):

1. **Find the decision.** Interview, or identify from documents, a moment where a real manager owed a real call.
2. **Secure access and consent.** Agree what may be published, in what disguise, and get sign-off on the factual record.
3. **Gather the record.** Interviews, internal documents, public filings, market data, site observation.
4. **Reconstruct the situation as the protagonist saw it.** The critical discipline: the case must present what was *knowable at the time*, not what is known now. Hindsight contaminates the dilemma.
5. **Build the exhibits** and check the arithmetic.
6. **Draft the narrative** in plain, present-tense-adjacent, non-analytical prose. The case *must not* contain the analysis.
7. **Test-read it** with someone who will be a student. If they cannot find the decision, the case is not finished.
8. **Write the teaching note** (see §6).
9. **Release for teaching**, then revise based on the discussion you actually get.

The single hardest discipline is step 6: authors who know the answer leak it. A case that reads as an argument has stopped being a case.

---

## 6. The Teaching Note

### 6.1 What It Is

The teaching note is the **instructor-only companion** to a case. It is not part of the student version and is typically only distributed to verified instructors by the publisher. Its purpose is to make the case teachable by someone other than its author.

| Teaching-note element | Function |
|---|---|
| **Case synopsis** | What happens, in a paragraph, for the instructor's orientation |
| **Teaching objectives** | What students should learn; the specific judgment being developed |
| **Target audience / course placement** | Programme level, course, position in the syllabus |
| **Assignment questions** | What students are asked to prepare before class |
| **Case analysis** | The substantive expected analysis — the instructor's own worked answer |
| **Discussion questions / teaching plan** | The sequence of questions that drives the session |
| **Board plan** | What goes on the board, in what layout, in what order |
| **Timing / class architecture** | How the 60–90 minutes is allocated across segments |
| **Epilogue / what happened** | The factual outcome, withheld from students |
| **Theoretical linkages** | The concepts and readings the case connects to |
| **Update notes** | What has changed since writing; what to watch if teaching it later |

### 6.2 Why the Teaching Note Matters More Than It Looks

Three reasons:

1. **It makes the case transferable.** A case without a teaching note can only be taught by its author. The teaching note is what turns a private script into a public asset — which is precisely why publishers require one.
2. **It is where the answer lives.** The whole point of the case is that students do not have the analysis. The teaching note holds the answer the instructor must *not* give away too early.
3. **It is the record of intent.** When the case is taught five years later by someone else, the teaching note is the only surviving evidence of what the case was designed to do.

### 6.3 The Board Plan

The board plan deserves separate treatment because it is the most distinctive artefact of the case method and it is easy to do badly.

Verified HBS guidance on board use states that a board helps the instructor (✅ verified at hbs.edu):

- capture and organise student comments;
- keep the conversation focused;
- make **the "arc" of the class visible as it unfolds**;
- provide visual anchors (diagrams, tables, frameworks).

The verified process is a three-step design:

1. **Start from goals and arc** — decide what the class should achieve and how the conversation will flow (diagnosis → options → decision → lessons is the example HBS gives), and let the board make that visible. Reserve a section for "takeaways" to fill in near the end; signal a **pasture change** with a new board or clearly separated area.
2. **Sketch a simple board plan** — board layout, tables and frameworks, headings and subheadings, on one page, before class.
3. **Use the board to guide, not dominate** — write sparingly but strategically; use headings, underlines, circles and arrows to highlight, link, and compare options; let the board evolve organically within the planned frame.

A common banking-case board layout (⚠ illustrative, composed for this guide):

| LEFT COLUMN | CENTRE (core concept) | RIGHT COLUMN |
|---|---|---|
| Facts / context as stated | The central question, restated | Options, with pros and cons |
| Symptoms vs root cause (two sub-lists) | — | Decision criteria |
| Data pulled from exhibits | — | Recommendation, with risks |
| — | **Takeaways (filled at close)** | — |

---

## 7. Case Analysis: The Discipline and Its Frameworks

### 7.1 The Standard Analysis Discipline

The sequence below is the canonical structure taught for analysing a management case. It is **convergent across pedagogy sources and practitioner guides**, though no single source owns it as a published standard (✅ verified in substance as the taught sequence; ⚠ the specific wording varies by publisher and prep guide).

| Step | What you do | The trap |
|---|---|---|
| **1. Situate** | Read the case twice: once for the story, once for the data. Identify the protagonist, the decision, and the deadline. | Skimming exhibits on the first pass |
| **2. Identify the issue** | Separate the **presenting problem** from the **root cause**. Ask "what is actually going wrong here?" | Solving the symptom (the visible problem is rarely the real one) |
| **3. Analyse** | Apply frameworks *selectively* to structure the analysis; do the arithmetic; test the logic against the exhibits | Framework recitation — applying every model to every case |
| **4. Generate alternatives** | Produce 2–4 genuinely distinct options, not one option and two straw men | False choices |
| **5. Evaluate against criteria** | Score options against explicit criteria: fit with strategy, financial return, capability, risk, feasibility, timing | Criteria invented after the conclusion |
| **6. Recommend** | One clear recommendation, answering the question asked | Hedged "it depends" answers |
| **7. Implementation / action plan** | What happens next: who, what, when, how measured; sequencing and quick wins | Stopping at "should" with no "how" |
| **8. Risks and mitigations** | What could go wrong, what would trigger a change of course | Ignoring downside and second-order effects |

The **issue-identification step is the highest-leverage one**, and it is where most analyses fail. The presenting problem is what the protagonist notices; the root cause is what actually drives the numbers. A bank case whose presenting problem is "branch profitability is falling" often has a root cause elsewhere entirely — in customer mix, product pricing, or channel cannibalisation.

### 7.2 The Canonical Frameworks — Condensed, with Cross-References

This repo already carries the strategy toolkit in depth. The frameworks below are the ones most commonly applied to management cases; the *how* is covered in the sibling guides, and is not re-derived here.

| Framework | Use in a case | Repo cross-reference |
|---|---|---|
| **SWOT** | Situational summary; a framing device, not an analysis | `management/strategic_management_guide.md` |
| **Porter's Five Forces** | Industry attractiveness; the structure behind profitability | `management/strategic_management_guide.md` |
| **Porter's value chain** | Where value is created and where cost sits | `management/strategic_management_guide.md` |
| **3C (Company, Customer, Competitor)** | The universal scan for market-entry and growth cases | `management/management_consulting_skills_guide.md` §3.4, §7.3 |
| **4P (Product, Price, Place, Promotion)** | Marketing and launch cases | `management/management_consulting_skills_guide.md` §7.3 |
| **Balanced Scorecard** | Translating strategy into measures; implementation architecture | `management/balanced_scorecard_guide.md` |
| **2×2 matrices** | Forcing a two-criterion trade-off (growth vs margin, fit vs feasibility) | `management/strategic_management_guide.md` |
| **Financial / ratio analysis** | Margins, returns, liquidity, leverage, cost-income, NPLs | `banking/*.md` for banking-specific ratios |
| **Issue tree / MECE decomposition** | Structuring the analysis so nothing is missed and nothing is double-counted | `management/management_consulting_skills_guide.md` §2.3–§2.4 |
| **Hypothesis-driven problem solving** | Starting from a hypothesis and testing it | `management/management_consulting_skills_guide.md` §2 |

**The honest rule (✅ reflected in HBS method framing).** The frameworks are scaffolding for the *discussion*, not the discussion itself. A case analysis that is a tour of frameworks has answered no question. The best analyses use one or two frameworks as structure and spend their effort on the specific, messy, particular facts of *this* situation.

### 7.3 The Analysis Failure Modes

These recur in practice literature and in case-teaching commentary. ⚠ The lists below are synthesised from practitioner prep guidance (PrepLounge, CaseCoach, managementconsulted.com, *Case in Point*) and case-teaching commentary — **practitioner evidence, not scholarship**. They are reported here as practitioner consensus, flagged as such.

| Failure mode | What it looks like | Why it fails |
|---|---|---|
| **Solving the wrong problem** | A polished analysis of the presenting symptom | Effort spent on the wrong question scores zero in a case discussion |
| **Framework recitation** | Every model applied, none illuminating | Demonstrates recall, not judgment |
| **Ignoring the exhibits** | Prose analysis that contradicts the data | The numbers are the evidence; ignoring them is disqualifying |
| **Arithmetic errors / no sanity checks** | Numbers that do not survive a plausibility test | Undermines every downstream conclusion |
| **No decision** | A summary of considerations with no recommendation | The case exists to produce a decision |
| **Straw-man alternatives** | One real option, two decoys | The evaluation is theatre |
| **Recommendation without implementation** | "Enter the market" with no plan | Management cases end in action, not intention |
| **Ignoring risk** | An upside-only recommendation | Real decisions are about downside management |
| **Confusing correlation with cause** | "Sales fell when we launched the app" | The analysis does not establish causation |
| **Overfitting to one precedent** | "It worked at Bank X, so it will work here" | Context differs; this is the research-case generalisation error in miniature |
| **Hindsight bias** | Judging the protagonist by what is known now | The case presents what was knowable then |
| **Hedging** | "It depends" as the conclusion | True but useless; the case requires a call |

### 7.4 The Written Case Analysis

Where a written deliverable is required (an exam, a case memo, a prep assignment), the standard form is answer-first:

| Section | Content |
|---|---|
| **Recommendation** | One paragraph: what should be done, and the single strongest reason |
| **Situation / issue** | The problem, stated precisely, with the root cause distinguished from the symptom |
| **Analysis** | The evidence, structured; the arithmetic shown; the framework used as a spine, not a show |
| **Alternatives considered** | The real options, and why they were rejected |
| **Recommendation detail** | What exactly, how, by when, with what resources, measured how |
| **Risks and mitigations** | What could go wrong and the response |
| **Appendix** | Supporting calculations and exhibit references |

This structure is the professional descendant of the case method — the pyramid-principle memo. For the structured-writing mechanics see `management/management_consulting_skills_guide.md` §3.6–§3.7.


## 8. Leading the Case Discussion

### 8.1 The Arc of the Class (✅ verified at hbs.edu)

HBS's own guidance describes the session as **"the arc of the class"**, with four elements:

1. an engaging **opening**;
2. purposeful **pasture transitions** — the middle of class, where the instructor moves between topics (HBS notes that at HBS these segments are sometimes called **"pastures"**);
3. a thoughtful **closing**;
4. underneath all three, **timing** — pacing each segment without losing flexibility.

The verified practical sequence for building a teaching plan (hbs.edu, "Preparing for Class") is:

1. **Start with teaching objectives** — what students should know or be able to do by the end.
2. **Allocate time across pastures** — divide the class into **3–5 segments**, each with a clear purpose.
3. **Refine key questions** — write opening questions and a few essential follow-ups for each pasture.
4. **Plan the boards** — sketch what should be on the board at the end of each segment.
5. **Identify a few key students to call on** — especially for the opening and where specific experience deepens the discussion.

HBS's own caveat, worth quoting for its candour: **"You are creating a teaching plan, not a minute-by-minute script."** (✅ verified.)

⚠ **On the "80-minute" figure.** The 60–90 minute case class is standard in MBA settings and the repo's MBA guide cites the 80-minute figure, but the exact duration is a scheduling parameter, not a methodological constant. Treat "80 minutes" as the typical convention, not a verified rule.

### 8.2 The Opening and the Cold Call (✅ verified)

The opening call is one of the method's hallmarks. Verified points from hbs.edu:

| Element | Verified detail |
|---|---|
| **Cold call** | After framing the session, the instructor typically cold calls a student to open the discussion |
| **Purpose 1** | Establish and reinforce high expectations for student preparation |
| **Purpose 2** | Provide a developmental opportunity for the opening student (content mastery, critical thinking, communication) |
| **Purpose 3** | Catalyse the discussion and serve as a touch point throughout the class |
| **Use during class** | Many instructors use additional cold calls, especially to encourage deeper preparation **in the age of AI** |
| **Instructor opening** | Serves three objectives: provide context and connect; heighten engagement; shape discussion |
| **Follow-up** | A lack of follow-up can signal that the opener's response was weak or that the instructor is unwilling to challenge; follow-ups push deeper, provoke debate, and explain anomalies |

The **instructor opening** and the **opening call** are distinct: the instructor frames context, engagement, and discussion shape; then a student is called to take the first position.

### 8.3 Questioning, Listening, Responding (✅ verified)

HBS frames these as "the three essential skills of questioning, listening and responding — the backbone of discussion-based teaching":

| Skill | Verified content |
|---|---|
| **Questioning** | Plan key questions in advance (opening questions per segment; probes); draw on a repertoire in real time to start segments, deepen analysis, transition, or rescue a stuck discussion. Distinguish open vs closed; instructional framing; personalised vs abstract. The tip sheet groups questions into four categories: starting a segment, following up, transitioning, handling special challenges |
| **Listening** | Listen on four levels: content; tone and emotion; what is left unsaid; disconnects between students ("Are you two talking about the same issue?"). Teach students to listen by echoing and having them restate each other's positions |
| **Responding** | Respond strategically, not automatically: minimal responses, echoing, indirect feedback, **silence as response** |

The single most-cited line in the whole pedagogy, attributed on hbs.edu to C. Roland Christensen, *Education for Judgment*, 1991:

> "It would be hard to name a more valuable pedagogical accomplishment than the mastery of the art of asking the right question, of the right student, at the right time—and in the right way."

### 8.4 Participation and Its Management (✅ verified in structure; ⚠ on weighting)

HBS's participation guidance is explicit that participation is central:

- **Participation is central to learning and evaluation**; content is co-created by students and instructor in real time; and **in some courses, participation can be a large share of the grade**. (✅ verified at hbs.edu.)
- **Exact weighting is school- and course-specific and not authoritatively published.** ❌ Do not cite a specific percentage as an HBS rule. ⚠
- Verified practice for managing it: create an "open line" about getting in; normalise speaking without full expertise; combine high standards with visible support (celebrate small steps, work with students until something good emerges, name the growth); **plan and monitor calling patterns**; mix volunteers and cold calls; and deliberately support the **"B players"** — the large middle group of steady but less visible contributors, whose neglect can turn them disengaged.
- Tracking is real work: the guidance recommends reviewing the roster before class, tracking who speaks during class, and scanning participation notes after class.

Student performance in discussion-based courses is assessed across **class participation, individual written work (exams, reflections, papers), and group work (projects, presentations)** (✅ verified at hbs.edu).

### 8.5 Engagement Devices (✅ verified)

Beyond questioning, HBS lists these engagement instruments:

| Device | What it is | Best practice |
|---|---|---|
| **Role play** | Students act in character; instructor may take a role | Three stages: set-up, action, debrief ("How realistic was the interaction? What was surprising?") |
| **Votes / polls** | In-class votes or pre-class surveys on a specific question | Reveals the distribution of opinion and identifies whom to call on |
| **Buzz groups** | A few minutes of discussion among students | State the time and the end signal; don't linger; allow report-outs; especially useful in executive education |
| **Class guests** | Case protagonist or expert joins | Three modes: case update, Q&A with the protagonist, guest as "live case" |
| **Tempo variation** | Alternating lively debate with quiet, reflective moments | Treat **silence as a tool**, not a problem |

### 8.6 The Discussion-Leadership Literature

The canon of the discussion-teaching literature, as recommended by HBS pedagogy pages and verifiable in publication records:

| Work | Author(s) | Year / publisher | Status |
|---|---|---|---|
| **Education for Judgment: The Artistry of Discussion Leadership** | C. Roland Christensen, David A. Garvin, Ann Sweet (eds.) | 1991, Harvard Business School Press, 312 pp. | ✅ verified (bibliographic record) |
| **Teaching and the Case Method** | C. Roland Christensen with Abby J. Hansen | Harvard Business School Press | ✅ the book exists; ⚠ exact edition/date not verified for this guide |
| **The Case Study Handbook: How to Read, Discuss, and Write Persuasively About Cases** | William Ellet | Harvard Business School Press | ⚠ the title and author are well attested; **exact edition/year not verified here** |
| **Tools for Teaching** | Barbara Gross Davis | 1993 | ✅ cited by HBS guidance |
| **Choreographing a Case Class** | V. Kasturi Rangan | 1996, HBS note | ✅ cited by HBS guidance |
| **Gaining Closure / Helping Discussions to 'Catch Fire'** | Robert F. Bruner | HBS notes | ✅ cited by HBS guidance |

The Christensen/Garvin/Sweet volume is the closest thing to a canonical text: its essays — "The Discussion Teacher in Action: Questioning, Listening, and Response" (Christensen), "With Open Ears: Listening and the Art of Discussion Leading" (Leonard), "Establishing a Teaching/Learning Contract" (Hansen) — map directly onto the HBS pedagogy structure summarised above.

### 8.7 The Method's Critique (⚠ flagged)

The case method is not uncontested, and the critics are serious. The standard positions:

| Critique | Substance | Status |
|---|---|---|
| **Mintzberg** (*Managers Not MBAs*, 2004) | The classroom cannot teach management; management is learned in practice; case discussions over-reward analytical showmanship | ✅ the book and argument exist; ⚠ the argument is contested |
| **Confident-advocacy bias** | The method rewards the articulate advocate over the deep technician | ⚠ practitioner/commentary consensus, not measured |
| **Pattern-matching over first principles** | Students learn to fit situations to known cases rather than reason from fundamentals | ⚠ commentary |
| **Cannot reproduce real stakes** | No career, no capital, no consequences — so the "decision" is simulated | ⚠ commentary |
| **The defence** | The method trains the *posture* of decision-making — triage, articulation, owning a call under uncertainty — which is closer to managerial work than lecture | ⚠ the standard defence, not a finding |

Both sides are partially right, which is why leading programmes mix cases with lectures, problem sets, simulations, and field work rather than relying on any single pedagogy. The mix is the school's fingerprint. (See `management/mba_body_of_knowledge_guide.md` §4.3 for the repo's fuller treatment.)

### 8.8 Online and Asynchronous Case Teaching (✅ verified; ⚠ limited evidence)

Verified facts (HBS Business Impact Education, "5 Key Lessons from HBS's Pandemic Teaching Transformation", Lakhani and Viceira, 1 July 2021):

| Claim | Status |
|---|---|
| HBS **migrated its entire MBA programming online within a week** at the pandemic's onset | ✅ verified (institutional account) |
| HBS had **10–15 faculty already teaching the case method online**; it then had to train **200 faculty and staff** | ✅ verified |
| By September, HBS faculty were teaching **in-person and remote students synchronously** (hybrid) | ✅ verified |
| HBS's stated lessons: keep technology simple; avoid faculty isolation (collaboration via shared channels); **reduce scope** — less content fits online; there should be **no second-class hybrid experience**; build in **social learning** outside the classroom | ✅ verified as institutional lessons |
| HBS's claim that "**50 percent of the learning occurs outside of the classroom**" | ⚠ their framing, cited as an institutional estimate, not a measured figure |
| **HBX / HBS Online** was founded in **2013** with the goal of replicating the participant-centred classroom experience online | ✅ verified (HBS timeline) |
| The arrival of AI has prompted HBS to add cold calls specifically to encourage deeper preparation | ✅ verified |

⚠ **The evidence base for online case teaching is thin.** The verified sources are institutional accounts of a forced migration, not controlled studies comparing online and in-person case outcomes. Treat claims of parity between online and in-person case discussion as **institutional assertion**, not research finding. The most defensible reading: hybrid works better than fully virtual when designed to give remote participants first-class treatment; chat tools can *increase* participation from students who are quiet in the room; and less content fits in a session online than in person.


## 9. The Case-Interview Genre: A Cross-Reference

### 9.1 The Distinction, Stated in One Line

The **case interview** is not the case method. The teaching case is a *prepared discussion of a written artefact by a group*; the case interview is a *live performance test of an individual's reasoning*. They share a surface (a business situation, a decision, structured analysis) and almost nothing else operationally.

| Axis | Teaching case | Case interview |
|---|---|---|
| **Format** | Written case, prepared in advance | Spoken, unseen, live |
| **Unit of assessment** | Collective discussion; individual contribution within it | The individual, alone |
| **Duration** | 60–90 minute class | Typically 30–45 minutes |
| **Who drives** | Instructor leads discussion | Candidate leads, or interviewer leads question-by-question |
| **What is scored** | Judgment, contribution, listening, argument | Structure, quantitative reasoning, communication, composure |
| **Evidence** | Exhibits provided | Data requested incrementally |
| **Purpose** | Learning | Hiring |
| **The artefact** | A teaching note exists | A scoring rubric exists (not published) |

### 9.2 The Format (⚠ practitioner-sourced)

The consulting case interview is **standard MBB and tier-2 recruiting practice**. ⚠ Firm-specific claims — what each firm does, how it scores, whether the case is candidate-led or interviewer-led — are **reported by prep resources and are not authoritatively published by the firms**. Treat them as reported/anecdotal.

The two commonly reported formats:

| Format | Who drives | Commonly associated with | Status |
|---|---|---|---|
| **Candidate-led** | The candidate structures the problem and drives the analysis | Reported as the classic format and used by several firms | ⚠ reported |
| **Interviewer-led** | The interviewer asks a sequence of discrete questions, each scored individually | Reported as the McKinsey style | ⚠ reported |

The repo already carries the case-interview taxonomy in full. Summarised, not re-derived:

- **Case types** — market sizing, profitability, market entry, pricing, M&A/investment, operations/growth (see `management/management_consulting_skills_guide.md` §7.2).
- **Frameworks** — profitability tree, 3C, 4P, the pricing stool, the Fermi chain (see §7.3 of that guide; frameworks themselves in §3.4).
- **The canonical arc** — clarify → structure → analyse → recommend (see §7.4).
- **Practice apparatus** — *Case in Point* (Marc Cosentino), PrepLounge, CaseCoach, firm-published sample cases, the "30 cases" heuristic (see §7.5, §7.7).

### 9.3 Where the Two Genres Touch

Three genuine connections:

1. **The case interview is the case method's skills test, extracted from the discussion.** Practising cases — even purely for interviews — builds the same analytic posture the teaching case builds: structure first, evidence next, a decision at the end.
2. **MBA case practice is the training ground.** The teaching case is the rehearsal; the case interview is the performance. Many candidates find that sustained case-method coursework is the best preparation because it makes structured reasoning habitual.
3. **Both are judgement tests under uncertainty.** Neither rewards the memorised answer; both reward the defensible call made with incomplete information. That is the deepest family resemblance.

### 9.4 The Engineering-Interview Analogue

The case interview has a structural twin in software hiring: the **system-design interview** (and, for machine-learning roles, the ML system-design interview). The parallel is instructive:

| Consulting | Engineering |
|---|---|
| Case interview — clarify, structure, analyse, recommend | System design — clarify requirements, sketch architecture, reason about trade-offs, propose a design |
| Profitability tree, 3C, 4P | Load estimation, data model, component decomposition |
| Market sizing (Fermi) | Back-of-envelope capacity estimation |
| "Why is profit down?" | "Why is latency up?" |
| Recommendation with risks and next steps | Design with bottlenecks, failure modes, and scaling path |

The repo covers the engineering side in depth — see `technology/system_design_interview_insiders_guide.md`, `technology/ml_system_design_interview_guide.md`, and `technology/ddia_study_companion_guide.md`. The lesson is transferable in both directions: the interview genre, in any field, is **a live test of structured reasoning on an unseen problem**, and the preparation method (drills, feedback, deliberate practice) is the same.


## 10. Worked Example: The Cymbal Bank Case

> **This section is illustrative and exemplary.** The case below is a **constructed teaching case**, not a real HBS/Ivey/Darden case and not a description of any real institution. **Cymbal Bank** is this repository's fictional Singapore bank persona, used for all worked examples (see `banking/*.md`, e.g. `banking/trade_finance_systems_guide.md` §7 and `banking/maybank_software_systems_guide.md` §10 for its established conventions). The exhibits are invented but internally consistent. The teaching note is written the way a real one would be, so the anatomy in §5–§6 and the practice in §7–§8 have a concrete instance.

### 10.1 The Case (student version — approx. 3,000 words, 6 exhibits)

**Cymbal Bank (A): The SME Lending Decision**

*Case: CB-2026-A. Teaching note available. This case was prepared as a basis for class discussion rather than to illustrate effective or ineffective handling of an administrative situation. Names and figures are disguised.*

---

It was 6:40 on a Monday morning in March 2026, and **Priya Nair**, Head of Business Banking at Cymbal Bank, had forty million dollars to place before Friday.

The paper in front of her was the board's mandate from the February strategy offsite: arrest the decline in Business Banking profitability, and do it within the current three-year plan. The numbers had been getting worse for three years and everyone in the room knew it. What nobody agreed on was why.

Business Banking had been Cymbal's quiet franchise for a decade — a steady earner that nobody in the top team had to worry about. Cymbal was a mid-sized Singapore bank: SGD 62 billion in assets, a strong retail deposit base, a respectable wholesale book, and a business-banking division that lent to small and medium enterprises across Singapore and, increasingly, Johor and Batam. It was not a giant. It could not outspend DBS or OCBC, and it knew it.

The February board pack had landed badly. Pre-tax profit in Business Banking had fallen from SGD 60 million in FY2024 to an estimated SGD 46 million in FY2026, against a rising cost-to-income ratio. The Chief Executive had asked one question that Priya had not been able to answer crisply: *"Is this a revenue problem or a cost problem?"* The Chief Risk Officer had added a second: *"And is it the market, or is it us?"*

Priya had three proposals on her desk.

The first, from the Chief Technology Officer, was to **build a digital SME lending platform in-house**: SGD 40 million over thirty months, a straight-through digital journey from application to disbursement, fully owned and fully Cymbal's to shape. The CTO argued that owning the platform was the only way to compete with the digital banks, and that outsourcing the customer journey was a strategic surrender.

The second, from the Head of Distribution, was to **hire forty additional relationship managers** for the small and mid-market segments and go back to what had worked: relationships, credit judgment, deeper wallet share with the customers Cymbal already had. The case for it was simple — the division's best customers were the ones a good RM had grown over years.

The third, from the Head of Digital Partnerships, was to **partner with a regional fintech lender** that had a proven digital credit engine for micro-enterprises. Integration would cost SGD 12 million and take nine months; the fintech would take a revenue share on originated loans; Cymbal would keep the customers, the funding, and the balance sheet. "We are not buying a platform," the Partnerships head wrote. "We are buying three years."

Priya had a fourth option nobody had proposed: a combination. But she was not yet sure the combined case was coherent, or whether she was just avoiding a decision.

She pulled the division's numbers and started again from the customer.

*(The case continues with the division's three-year financials; a customer-segment profitability analysis; the digital and operational KPI dashboard; the competitive context in Singapore SME banking; the regulatory backdrop; and a short section on the partnership model's risks. It ends with the decision point in §10.2. Total case length: approximately 3,000 words of narrative plus the exhibits below.)*

---

### 10.2 The Decision Point

> **It is Friday. Priya Nair must recommend to the Cymbal Bank executive committee how to deploy the SGD 40 million. She must choose between: (A) build the platform in-house; (B) hire forty relationship managers; (C) partner with the fintech; or (D) a hybrid. She must also explain what the actual problem is — because she is no longer sure the three proposals are aimed at the same one.**
>
> **What is the root cause of the Business Banking profit decline, and where should the SGD 40 million go?**

---

### 10.3 Exhibit 1 — Business Banking Financials (SGD millions)

| | FY2024 (A) | FY2025 (A) | FY2026 (F) |
|---|---|---|---|
| Revenue | 210 | 218 | 222 |
| Costs | 150 | 164 | 176 |
| **Profit before tax** | **60** | **54** | **46** |
| Cost-to-income ratio | 71% | 75% | 79% |
| Loan book (SGD bn) | 8.4 | 8.8 | 9.0 |
| NPL ratio | 2.1% | 2.6% | 3.0% |
| NIM (Business Banking) | 2.15% | 2.02% | 1.94% |

### 10.4 Exhibit 2 — Segment Profitability, FY2026 (Forecast)

| Segment | Turnover band | Customers (’000) | Loan book (SGD m) | Revenue (SGD m) | Cost-to-serve / customer (SGD) | Segment ROE |
|---|---|---|---|---|---|---|
| **Micro** | < SGD 5 m | 48.0 | 1,800 | 62 | 1,450 | **4%** |
| **Small** | SGD 5–50 m | 9.0 | 4,200 | 88 | 6,800 | 11% |
| **Mid** | SGD 50–250 m | 0.9 | 3,000 | 72 | 31,000 | 15% |
| **Total / blended** | | 57.9 | 9,000 | 222 | — | 8% |

*Note: revenue sums to the FY2026 figure. Direct costs implied by the cost-to-serve column total SGD 158.7 m; the remaining SGD 17.3 m of divisional overhead is unallocated above segment level. Micro-segment direct costs alone (SGD 69.6 m) exceed its revenue (SGD 62 m).*

### 10.5 Exhibit 3 — Operational and Digital KPI Dashboard

| KPI | FY2024 | FY2025 | FY2026 (F) |
|---|---|---|---|
| Digital onboarding share | 22% | 31% | 38% |
| Average loan decision time (working days) | 12 | 9 | 8 |
| Accounts per relationship manager | 180 | 172 | 165 |
| Cost-to-serve, micro segment (SGD) | 1,290 | 1,380 | 1,450 |
| Digitally active SME customers (’000) | 14 | 21 | 28 |
| Micro-segment accounts opened via branch | 71% | 58% | 49% |

### 10.6 Exhibit 4 — Option Comparison (As Proposed)

| Option | Capex (SGD m) | Time to effect | Control | Key claimed benefit | Key risk |
|---|---|---|---|---|---|
| **A. Build in-house** | 40.0 | 30 months | Full | Digital journey owned; long-term cost base | Long build; capability risk; benefits land after FY2028 |
| **B. 40 RMs** | 40.0 (3-yr cost) | 12–18 months | Full | Relationship depth in small/mid | Does not fix micro economics; CIR worsens first |
| **C. Fintech partner** | 12.0 + rev. share | 9 months | Shared | Fast, variable-cost micro credit | Dependency; share leakage; exit terms |

### 10.7 The Teaching Note (Instructor Only — Illustrative)

**Case synopsis.** A mid-sized Singapore bank's business-banking franchise has shed SGD 14 m of PBT in two years. Three proposals compete for SGD 40 m. The apparent decision — build, hire, or partner — is the wrong first question; the real work is diagnosing which segment is destroying value.

**Teaching objectives.** (1) Separate a presenting problem ("profit is falling") from a root cause. (2) Read a segment P&L and attribute profitability correctly. (3) Evaluate build/buy/partner against segment economics, not technology fashion. (4) Recommend with implementation and risk, not just direction.

**Assignment questions.** (i) What is the actual problem? (ii) Which segment should Cymbal serve, and how? (iii) Where should the SGD 40 m go? (iv) What would make you change your mind?

**Case analysis (the expected answer).**

1. **Diagnosis.** Exhibit 2 is the case. The **micro segment loses money**: SGD 62 m of revenue against SGD 69.6 m of direct cost, a segment ROE of 4% versus 11% (small) and 15% (mid). Micro is 83% of customers by count but only 28% of revenue. The presenting problem — falling division profit — is driven by a growing micro book whose unit economics are negative and worsening (cost-to-serve micro rose from SGD 1,290 to SGD 1,450 across three years, Exhibit 3).
2. **What it is *not*.** Exhibit 1 shows NIM compression (2.15% → 1.94%) and rising NPLs (2.1% → 3.0%), but neither is the dominant driver: revenue is *flat to slightly up*, while costs rise SGD 26 m over two years. This is substantially a **cost-to-serve problem concentrated in one segment**. It is also not RM laziness: RM productivity fell only modestly (180 → 165 accounts per RM, Exhibit 3) while digital onboarding rose. The micro cost is structural — high-touch servicing of a low-balance book.
3. **Options against the diagnosis.** (A) Build fixes the *journey*, not the *economics* — and lands too late. (B) Hiring RMs improves small/mid, where the economics are already good, but adds SGD 40 m of cost into a division whose CIR is 79% and does nothing for micro. (C) Partner is the only option aimed squarely at the losing segment, at the lowest cost and the fastest time to effect — but it is a partial answer. (D) **Hybrid is the coherent answer**: partner for micro (SGD 12 m, 9 months, variable cost), redeploy and selectively hire a *smaller* RM cohort into small/mid (say 12, not 40), and hold the remaining capital for the segment the analysis actually supports.
4. **The number that decides it.** Stopping the micro bleed is worth roughly SGD 7–8 m of PBT a year at current volumes, at a variable cost — which is most of the SGD 14 m decline. No build programme can match that payback inside the plan horizon.

**Board plan.**

| LEFT | CENTRE | RIGHT |
|---|---|---|
| Symptoms: PBT 60→46; CIR 71→79%; NPL 2.1→3.0% | **Is this revenue or cost? Market or us?** | A Build / B Hire / C Partner / D Hybrid |
| Root cause: micro loses SGD 7.6 m; cost-to-serve 1,290→1,450 | → **Cost, concentrated in micro** | Criteria: payback, fit, control, risk |
| Revenue flat (222 vs 210); costs +26 | → **Diagnosis: segment economics, not market** | **Recommend D**, with milestones |
| — | **Takeaways (at close):** diagnose before you invest; segment P&L beats aggregate P&L | — |

**Timing (80-minute session).** Opening cold call (5) → what is the problem? (20) → segment analysis and the micro loss (20) → options and criteria (20) → recommendation and risks (10) → close (5).

**Epilogue (withheld from students).** In the illustrative sequence, the executive committee approves the hybrid: a nine-month partnership for micro-credit origination with a three-year exit clause, twelve (not forty) new RMs redeployed to small/mid, and the balance of the SGD 40 m held against the FY2028 plan. Micro's cost-to-serve falls below SGD 900 by FY2028; the division's CIR returns to the low seventies. ⚠ This epilogue is a **constructed outcome for teaching purposes**, not a result.

**Transferable lessons.** Aggregate profitability hides segment-level destruction. The build/buy/partner question is a *consequence* of a segment decision, never a substitute for it. And the honest version of "we need a platform" is usually "we need to stop losing money in a segment we were never structured to serve."


## 11. Claims Audit

| Claim | Status | Source / note |
|---|---|---|
| Langdell pioneered the casebook method at Harvard Law School from the early 1870s | ✅ | Standard legal-history record |
| Wallace Brett Donham directed HBS to adopt case-based teaching (deanship 1919–1942) | ✅ | HBS Baker Library case-method exhibit |
| The **first HBS case** was "The General Shoe Company", written **1921** by **Clinton P. Biddle**, MBA 1920, of the Bureau of Business Research | ✅ | library.hbs.edu, "The General Shoe Company, 1921" |
| General Shoe was a **one-page** case about workers leaving the plant 45 minutes early | ✅ | library.hbs.edu |
| The Bureau of Business Research (formed under Dean Gay) wrote the early cases at Donham's direction | ✅ | library.hbs.edu |
| HBS faculty voted to name the approach the **"case system"** on **10 May 1922** | ✅ | hbs.edu/case-method-100 |
| By **1923**, two-thirds of HBS courses were taught by the case method | ✅ | hbs.edu/case-method-100 |
| By **1939**, nearly 15,000 cases had been produced in 18 years | ✅ | hbs.edu/case-method-100 |
| HBS's case-method centennial was celebrated in **2021** | ✅ | hbs.edu/case-method-100 |
| Christensen described case teaching as "the art of managing uncertainty"; instructor as "planner, host, moderator, devil's advocate, fellow-student, and judge" | ✅ | hbs.edu, Christensen Center |
| Case classes "unfold without a detailed script" | ✅ | hbs.edu |
| HBS guidance: build a teaching plan; divide the class into **3–5 pastures**; "a teaching plan, not a minute-by-minute script" | ✅ | hbs.edu, "Preparing for Class" |
| HBS guidance describes **the "arc of the class"** (opening / pasture transitions / closing) plus timing | ✅ | hbs.edu, "Elements of a Class Discussion" |
| Cold calling has three stated objectives (preparation expectations, developmental opportunity for the opener, catalysis of discussion) | ✅ | hbs.edu, "Questioning & Discussions" |
| Participation: "in some courses, participation can be a large share of the grade" | ✅ | hbs.edu, "Participation Management" |
| Yin, *Case Study Research and Applications*, Sage; **6th edition, September 2017**, 352 pp. | ✅ | Sage product page |
| Yin's 1st edition was **1984**; the title *Case Study Research: Design and Methods* | ✅ | Standard bibliographic record |
| The 6th edition **"sharpens discussion of distinguishing research from non-research case studies"** | ✅ | Sage product page |
| Eisenhardt, "Building Theories from Case Study Research", **AMR 1989, 14(4): 532–550** | ✅ | Journal record, multiple copies |
| Flyvbjerg, "Five Misunderstandings About Case-Study Research", **Qualitative Inquiry 2006, 12(2): 219–245** | ✅ | journals.sagepub.com |
| *Education for Judgment* — Christensen, Garvin & Sweet, 1991, Harvard Business School Press, 312 pp. | ✅ | Bibliographic records (Google Books, Internet Archive) |
| HBS migrated its MBA online within a week in 2020; trained ~200 faculty; 10–15 already taught online | ✅ | hbsp.harvard.edu, Lakhani & Viceira, 2021 |
| HBX / HBS Online founded **2013** | ✅ | HBS timeline |
| The **1920 Donham memo** as the precise trigger, and its exact wording | ⚠ | ✅ variably reported; the 1921 first case is the firmer anchor |
| The "first case written at a kitchen table" and who wrote it | ⚠ | Contested/embellished origin anecdotes; not established to primary-source standard |
| The strength/directness of Langdell's causal influence on HBS | ⚠ | The lineage is real; the causal weight is debated |
| Teaching-case vs research-case anonymisation practice | ⚠ | Varies by institution and publisher |
| Participation weighting percentages at HBS | ⚠ | School- and course-specific; **not authoritatively published** |
| Case-interview format details per firm (candidate-led vs interviewer-led) | ⚠ | Reported by prep resources; not published by the firms |
| The case-analysis failure modes in §7.3 | ⚠ | Practitioner consensus (PrepLounge, CaseCoach, managementconsulted.com, *Case in Point*), not scholarship |
| "80-minute" case class | ⚠ | Typical convention, not a verified constant |
| Online vs in-person case-teaching parity | ⚠ | Institutional accounts only; no controlled evidence reviewed |


## 12. What Could Not Be Verified

- ⚠ **The precise date and exact wording of Donham's 1920 memorandum.** It is widely cited, but a primary-source copy with a date was not verified here. The commonly cited 1920 date may be conventional rather than documentary.
- ⚠ **The "first case written at a kitchen table" story and the Copeland attribution.** These appear in later retellings; the accessible primary record attributes the 1921 General Shoe case to Clinton P. Biddle.
- ⚠ **Robert Stake's own definitional text.** *The Art of Case Study Research* (Sage) was not extracted directly here; the Yin/Stake contrast is stated as a widely drawn distinction, not quoted from Stake.
- ⚠ **Creswell's exact placement of case study within qualitative inquiry approaches.** Not verified at source for this guide.
- ⚠ **Ivey Publishing's and Darden's and The Case Centre's official case-writing guidelines.** Attempts to retrieve `thecasecentre.org` and `iveypublishing.ca` guidance pages failed (server errors); the craft guidance in §5–§6 is therefore drawn from verified HBS pedagogy pages plus general case-writing conventions, **not** from those publishers' own documents.
- ⚠ **The exact required components of a teaching note per publisher.** The element list in §6.1 is a synthesis of the genre, not a published specification.
- ⚠ **The current (7th) edition details of Yin.** The sixth edition (2017) is verified; a seventh edition is referred to in circulation but was not verified here.
- ⚠ **Participation grading weights.** No authoritative published percentage was found; HBS states only that participation can be a large share in some courses.
- ⚠ **The management case-interview's history.** No verified primary account of *when* or *how* the consulting case interview originated was located; the format is well attested in practice, not in history.
- ⚠ **Quantified efficacy of the case method.** No controlled comparison of case-method and lecture-method outcomes was reviewed; claims that the method builds "judgment" are institutional rationale, not measured results.
- ⚠ **Any claim about the Cymbal Bank case in §10 being real.** It is a constructed teaching case; every exhibit is invented.


## 13. Glossary

| Term | Meaning |
|---|---|
| **Case method** | Teaching by structured discussion of a written case; the HBS signature pedagogy |
| **Case system** | The name HBS faculty adopted for the approach on 10 May 1922 |
| **Teaching case** | A written decision-point narrative used for classroom learning; unresolved by design |
| **Research case** | A case study deployed as a social-science research design (Yin, Stake, Eisenhardt) |
| **Decision point** | The explicit question on which a teaching case terminates |
| **Protagonist** | The named individual whose seat the student occupies |
| **Exhibit** | Quantitative or visual supporting material attached to a case |
| **Teaching note** | Instructor-only companion: objectives, analysis, board plan, epilogue |
| **Board plan** | The planned layout of the classroom board, aligned to the arc of the class |
| **Cold call** | Naming a student to open or advance a discussion without prior warning |
| **Opening call** | The first student question of the session, usually a cold call |
| **Pasture** | An HBS term for a segment of the class discussion |
| **Arc of the class** | The overall shape of a session: opening, transitions, closing, timing |
| **Analytic generalisation** | Generalising a case's findings to theory rather than to a population (Yin) |
| **Replication logic** | Treating multiple cases as replications rather than as a sample (Yin) |
| **Rival explanation** | A competing account that a case study must test (Yin) |
| **Cross-case analysis** | Comparing cases to surface constructs and relationships (Eisenhardt) |
| **MECE** | Mutually Exclusive, Collectively Exhaustive — the decomposition rule |
| **Business case** | An internal investment-proposal document; **not** a teaching case |
| **Case interview** | A live consulting-hiring exercise; a performance test, not a teaching artefact |
| **Cymbal Bank** | This repository's fictional Singapore bank persona, used in §10 |


## 14. Sources and Further Reading

**Primary / authoritative (verified for this guide):** HBS Christensen Center for Teaching and Learning — `hbs.edu/teaching/case-method` and its subpages (the case method in practice; teaching foundations; leading a case discussion; providing and leveraging feedback; board use; participation management; questioning and discussions; engagement; preparing for class). HBS Baker Library — "Case Method 100 Years" and "The General Shoe Company, 1921" (`library.hbs.edu`). HBS — `hbs.edu/case-method-100` (milestone timeline). Sage Publications — *Case Study Research and Applications*, 6th ed. product page. *Academy of Management Review*, 1989, 14(4). *Qualitative Inquiry*, 2006, 12(2). Harvard Business School Publishing — "5 Key Lessons from HBS's Pandemic Teaching Transformation" (Lakhani & Viceira, 2021).

**Methodology canon:** Robert K. Yin, *Case Study Research and Applications: Design and Methods* (Sage). Kathleen M. Eisenhardt, "Building Theories from Case Study Research" (*AMR*, 1989). Eisenhardt & Graebner, "Theory Building from Cases" (*AMJ*, 2007, 50(1): 25–32 — citation verified). Bent Flyvbjerg, "Five Misunderstandings About Case-Study Research" (*Qualitative Inquiry*, 2006). Robert E. Stake, *The Art of Case Study Research* (Sage). John W. Creswell, qualitative-inquiry design texts.

**Teaching canon:** Christensen, Garvin & Sweet (eds.), *Education for Judgment* (1991). William Ellet, *The Case Study Handbook* (HBS Press). Christensen & Hansen, *Teaching and the Case Method* (HBS Press).

**Practitioner sources (⚠ flagged as practitioner evidence, not scholarship):** PrepLounge, CaseCoach, managementconsulted.com, *Case in Point* (Marc Cosentino), firm-published practice cases.

**Internal cross-references:** `management/strategic_management_guide.md`; `management/mba_body_of_knowledge_guide.md` §4.1; `management/management_consulting_skills_guide.md` §3–§4, §7; `management/balanced_scorecard_guide.md`; `management/personal_mba_companion_guide.md`; `management/organizational_behavior_guide.md`; `management/business_case_development_guide.md`; `management/mckinsey_approach_guide.md`; `technology/system_design_interview_insiders_guide.md`; `technology/ml_system_design_interview_guide.md`; `technology/ddia_study_companion_guide.md`; `banking/trade_finance_systems_guide.md` §7; `banking/maybank_software_systems_guide.md` §10.

---

*The teaching case trains judgment; the research case builds knowledge; the case interview tests reasoning under observation. Three instruments, one intellectual posture — structure the ambiguity, decide on the evidence, own the call. That is why the form has outlived every attempt to replace it, and it is why it endures — the case method.*
