# The Manager's Path (Camille Fournier): The Book Companion

*A comprehensive study companion to Camille Fournier's "The Manager's Path: A Guide for Tech Leaders Navigating Growth and Change" (O'Reilly Media, 2017) — the book overview and verified metadata (publisher, first edition, March 13 2017 ebook / May 2 2017 paperback, ISBNs, page count), the verified author background (Goldman Sachs → Rent the Runway → Two Sigma → JPMorgan → CoreWeave, Apache ZooKeeper committer), the six-rung career-ladder reading of the book, a chapter-by-chapter study map of all nine chapters (with the repo's research notes corrected where their chapter titles did not match the book's own text), the recurring themes (the IC-to-manager transition, the tech-lead hybrid, mentoring, 1:1s, feedback as a gift, delegation, the sandwich boss, culture), a complete worked example (a Cymbal Bank payments-platform architect walking the path from IC to manager of managers), a claims-audit ledger that flags every claim honestly, a "What Could Not Be Verified" list, and a glossary. Chapter titles and quotations were verified against the book's own text this run (via Google Books search-within-book); anything that could not be confirmed is flagged ⚠ in place and listed in §14.2.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Management / Book-Companion
> **Audience:** engineers, architects, and tech leads considering or starting the management track; new and mid-level engineering managers; managers of managers — in a bank, a product company, or any technology organization
> **Last Updated:** August 2026

**Cross-references (the repo guides this companion maps into):** [the_managers_path_research.md](the_managers_path_research.md) (the repo's research notes for this book — read them first; **note:** this guide corrects their chapter titles for Ch.4–9 after verification against the book's own text — see the claims audit §14.1), [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) (the managing-up/managing-down/managing-sideways framework — the sandwich-boss theme in depth, cross-ref Ch.6–7), [managing_inward_guide.md](managing_inward_guide.md) (the self-management layer — self-awareness, energy, burnout; cross-ref Ch.4 and Ch.9), [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) (the tech-lead toolkit — 1:1s, feedback, delegation, decision-making frameworks; cross-ref Ch.2–4), [high_performance_team_guide.md](high_performance_team_guide.md) (team dynamics and psychological safety; cross-ref Ch.5), [grow_team_guide.md](grow_team_guide.md) (hiring, onboarding, culture, scaling; cross-ref Ch.5 and Ch.9), [360_management_guide.md](360_management_guide.md) (360° feedback and 360° stakeholder management; cross-ref Ch.4), [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) (influencing without authority, difficult conversations; cross-ref Ch.4 and Ch.7), [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) (matching leadership style to development level; cross-ref Ch.4), [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) (management styles and effectiveness; cross-ref Ch.1), [time_energy_management_guide.md](time_energy_management_guide.md) (the Eisenhower matrix — the importance-vs-urgency theme of Ch.6 — and deep work), [organizational_behavior_guide.md](organizational_behavior_guide.md) (team, leadership, and organization-level behavior; cross-ref Ch.5 and Ch.8), [strategic_management_guide.md](strategic_management_guide.md) and [it_strategy_guide.md](it_strategy_guide.md) (strategy formulation for the senior-leadership chapter; cross-ref Ch.8), [balanced_scorecard_guide.md](balanced_scorecard_guide.md) (strategy measurement, with a Cymbal Bank example; cross-ref Ch.8), [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) and [personal_mba_companion_guide.md](personal_mba_companion_guide.md) (the business-vocabulary layer for the senior chapters), the banking context for the worked example — [singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md), [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md), [universal_banking_model_guide.md](../banking/universal_banking_model_guide.md), [risk_management_models_guide.md](../banking/risk_management_models_guide.md), and the companion-genre precedents [grokking_system_design_companion_guide.md](../technology/grokking_system_design_companion_guide.md) and [ddia_study_companion_guide.md](../technology/ddia_study_companion_guide.md).

---

## Table of Contents

1. [The Book Overview](#1-the-book-overview)
2. [The Manager's Path: The Career Ladder](#2-the-managers-path-the-career-ladder)
3. [Chapter 1: Management 101](#3-chapter-1-management-101)
4. [Chapter 2: Mentoring](#4-chapter-2-mentoring)
5. [Chapter 3: Tech Lead](#5-chapter-3-tech-lead)
6. [Chapter 4: Managing People](#6-chapter-4-managing-people)
7. [Chapter 5: Managing a Team](#7-chapter-5-managing-a-team)
8. [Chapter 6: Managing Multiple Teams](#8-chapter-6-managing-multiple-teams)
9. [Chapter 7: Managing Managers](#9-chapter-7-managing-managers)
10. [Chapter 8: The Big Leagues](#10-chapter-8-the-big-leagues)
11. [Chapter 9: Bootstrapping Culture](#11-chapter-9-bootstrapping-culture)
12. [The Core Themes](#12-the-core-themes)
13. [The Worked Example: The Cymbal Bank Tech-Lead-to-Manager Progression](#13-the-worked-example-the-cymbal-bank-tech-lead-to-manager-progression)
14. [The Claims Audit, the What-Could-Not-Be-Verified List, the Glossary, and the Closing](#14-the-claims-audit-the-what-could-not-be-verified-list-the-glossary-and-the-closing)

---

## 1. The Book Overview

### 1.1 The Premise

**"The Manager's Path: A Guide for Tech Leaders Navigating Growth and Change"** (O'Reilly Media, first edition, 2017) is Camille Fournier's field guide to the engineering-management career. The book's own back-cover copy states the problem it was written to solve: *"Managing people is difficult wherever you work. But in the tech industry, where management is also a technical discipline, the learning curve can be brutal—especially when there are few tools, texts, and frameworks to help you"* (verified — publisher's description, identical on the Amazon listing and the Google Books record). In this practical guide, Fournier (tech lead turned CTO) takes the reader through each stage in the journey from engineer to technical manager — from mentoring interns to working with senior staff.

The book is organized **hierarchically by career stage**: each chapter corresponds to a rung on the engineering-leadership ladder, from managing yourself (Ch.1) to bootstrapping the culture of an entire engineering function (Ch.9). The introduction says it plainly: *"The following four chapters talk about people management, team management, management of multiple teams, and managing managers"* (verified verbatim — book text, introduction). This stage-based structure lets a reader jump to the chapter most relevant to their current (or next) role, which is exactly how this companion is meant to be read: the chapter map in §3–§11 tracks the book's real structure, and §13 walks one Cymbal Bank architect through the whole ladder.

**Why it matters.** Since 2017 the book has become the most-recommended first book for new engineering managers — the "bible for new engineering managers" framing is a community reputation repeated across Reddit (r/ExperiencedDevs, r/engineeringmanagers), Hacker News, and management reading lists (⚠ qualitative reputation, not a verified fact; see §14.1). Its durability comes from covering the *full arc* — mentor, tech lead, manager, manager of managers, senior leader — in one volume, with frameworks (1:1s, 30/60/90 plans, delegation categories, skip-levels, career ladders) that are still standard practice in 2026.

### 1.2 The Verified Book Facts

| Field | Detail | Status |
|-------|--------|--------|
| Full title | The Manager's Path: A Guide for Tech Leaders Navigating Growth and Change | ✅ verified (Amazon, Google Books, Library-of-Congress-style MARC record) |
| Author | Camille Fournier | ✅ verified |
| Publisher | O'Reilly Media, Sebastopol, CA | ✅ verified (Amazon; MARC record: "Sebastopol, CA : O'Reilly Media, 2017") |
| Edition | First edition | ✅ verified (MARC record) |
| Ebook publication | March 13, 2017 | ✅ verified (Google Books record: "13 Mar 2017") |
| Paperback publication | May 2, 2017 | ✅ verified (Amazon product page: "2 May 2017") |
| Print length | 241 pages (Amazon); catalog record gives "xiv, 226 pages"; Google Books ebook record gives 244 pages | ⚠ reconciled — see §14.2 |
| Print ISBN-13 / ISBN-10 | 978-1491973899 / 1491973897 | ✅ verified (Amazon, MARC, VitalSource) |
| eBook ISBN-13 / ISBN-10 | 978-1491973844 / 1491973846 | ✅ verified (Google Books, VitalSource) |
| Formats | Trade paperback, Kindle, ebook, audiobook | ✅ verified (Amazon format list; audiobook ⚠ not re-verified this run) |
| Amazon rating | 4.6 / 5 stars, 1,352 ratings | ✅ verified (Amazon, this run) |
| Goodreads rating | ~4.2 / 5 with "30,000+ ratings" | ⚠ from the repo's research notes only, not re-verified this run |

### 1.3 The Author: Camille Fournier

Fournier is one of the most-cited voices in engineering leadership, with a career spanning a bulge-bracket bank, a startup, a quantitative hedge fund, and (currently) a cloud-AI company. Career facts below are verified against Wikipedia, Lenny's Newsletter's bio, the Rachitsky Atlas, Crunchbase, and the Google Books/O'Reilly author bios (this run); discrepancies are flagged.

| Period | Role | Organization | Status |
|--------|------|--------------|--------|
| ~2001–2005 | B.S. Computer Science | Carnegie Mellon University | ✅ Wikipedia |
| 2005 | M.S. Computer Science | University of Wisconsin–Madison | ✅ Wikipedia |
| post-2005 | Engineer, ~18 months | Microsoft | ✅ Wikipedia |
| ~2006–2011 | Engineer (credit-risk software) → VP of Technology, 6+ years | Goldman Sachs | ✅ Wikipedia |
| 2011–2015 | Director of Engineering (2011) → **CTO (2014), departed 2015** | Rent the Runway | ✅ Wikipedia (⚠ the research notes' "CTO 2013–2017" is contradicted — see §14.1) |
| ~2015–2018 | Head of Platform Engineering (per Lenny's Newsletter); Crunchbase lists "Managing Director" | Two Sigma | ⚠ role title differs across sources |
| ~2018–2023 | Managing Director (Wikipedia infobox); "Global Head of Engineering and Architecture" (Lenny's Newsletter) | JPMorgan Chase | ✅ as "Managing Director" / ⚠ exact title wording |
| 2023–present | VP Engineering, Common Services | CoreWeave | ✅ Wikipedia + LinkedIn (2023) |

**Technical and writing credentials (verified):** Apache ZooKeeper committer and PMC member (✅ Crunchbase title "ZooKeeper PMC Member, Committer @ The Apache Software Foundation"; ⚠ the ASF committer-index page could not be checked directly this run); author of *The Manager's Path* (2017); editor of *97 Things Every Engineering Manager Should Know* (O'Reilly, 2019) (✅ Wikipedia); co-author (with Ian Nowland) of *Platform Engineering: A Guide for Technical, Product, and People Leaders* (O'Reilly — ebook October 8, 2024; print November 19, 2024; 324 pages; ISBN 9781098153601) (✅ Google Books record); writer of the "Ask The CTO" column for O'Reilly Media (✅ Wikipedia); founding member of the CNCF Technical Oversight Committee and board member of ACM Queue (✅ Google Books / O'Reilly author bio). Her blog lives at camilletalk.com (the current site is a speaking reel; the older "camilletalk" blog archive is where her essays live).

### 1.4 The Book-Facts Claims Audit

| Claim | Status | Source |
|-------|--------|--------|
| "March 2017" publication | ✅ (ebook March 13, 2017) | Google Books |
| "Paperback May 2, 2017" | ✅ | Amazon |
| "~240–241 pages" | ✅ (241 per Amazon) | Amazon; ⚠ MARC says xiv+226, Google Books says 244 |
| ISBN 978-1491973899 (print) | ✅ | Amazon, MARC, VitalSource |
| ISBN 978-1491973844 (ebook) | ✅ | Google Books, VitalSource |
| CTO of Rent the Runway 2013–2017 | ❌ — Wikipedia: Director of Engineering 2011, CTO 2014, departed 2015 | Wikipedia (Business Insider 2014, Fortune 2015) |
| VP of Technology at Goldman Sachs | ✅ | Wikipedia |
| Managing Director / Head of Platform Engineering at Two Sigma | ⚠ both titles appear in different sources | Crunchbase vs Lenny's Newsletter |
| Later JPMorgan (Managing Director; Global Head of Engineering & Architecture) | ✅/⚠ | Wikipedia infobox / Lenny's Newsletter |
| Apache ZooKeeper committer | ✅ (committer + PMC member) | Crunchbase, Grokipedia |
| "Platform Engineering" 2024 | ✅ (co-author Ian Nowland; O'Reilly; Oct/Nov 2024) | Google Books |
| BS Carnegie Mellon, MS University of Wisconsin–Madison | ✅ | Wikipedia |
| Chapter titles 1–9 (see §3–§11) | ✅ verified against the book's own text this run; the research notes' titles for Ch.4–9 are ❌ | Google Books search-within-book |

---

## 2. The Manager's Path: The Career Ladder

The book's organizing idea is that the management track is a **sequence of distinct jobs**, each with its own skills, trade-offs, and failure modes — not one job with a fancier title. This section lays out the six-rung ladder this companion uses (engineer → tech lead → manager → manager of managers → director → CTO), mapped honestly onto the book's actual nine chapters. Two honest corrections up front: the book has **no chapter titled "The Engineering Director" and no chapter titled "The CTO"** — those senior rungs are covered by *The Big Leagues* (Ch.8) and *Bootstrapping Culture* (Ch.9), plus the recurring "Ask the CTO" sidebars (all verified this run — see §14.1). The ladder below therefore describes the *roles*; the §3–§11 map says where the book actually covers each.

### 2.1 Rung 0: The Engineer (and the Managee)

Before you manage anyone, you manage yourself and your relationship with your manager. The book's Ch.1 — "Management 101" — is famously also a chapter about *being managed*: know what you want, ask for it, and give your manager a break. Every later rung assumes you learned this first.

### 2.2 Rung 1: The Mentor

Mentoring is the first leadership role that requires no authority — Ch.2. It is the book's recommended "safe place to practice being a manager," and the chapter where the 1:1 habit is built.

### 2.3 Rung 2: The Tech Lead

The hybrid role — Ch.3. You keep writing code *and* represent the team, plan the work, and make technical decisions. The book calls it "not a point on the ladder, but a set of responsibilities" (⚠ quoted via secondary summary — see §14.2), which is why it sits on the path between IC and manager.

### 2.4 Rung 3: The Manager (of Individual Contributors)

The full people-management job — Ch.4 ("Managing People") and Ch.5 ("Managing a Team"). 1:1s, feedback, performance reviews, underperformers, hiring, onboarding, and team dynamics. This is where most readers of this companion are headed.

### 2.5 Rung 4: The Manager of Managers

The step from one team to several — Ch.6 ("Managing Multiple Teams") and Ch.7 ("Managing Managers"). Your direct reports become managers themselves; your job becomes delegation, standard processes, skip-levels, and scaling yourself. This is also where the **sandwich boss** squeeze (see §12) gets real.

### 2.6 Rung 5: The Director (Senior Engineering Leader)

The book covers this rung in Ch.8 ("The Big Leagues"): strategy, organizational structure, board communication, and being the face of technology. The repo's research notes called this rung "The Engineering Director" — the role exists, but that exact chapter title does not (❌ verified absent from the book text).

### 2.7 Rung 6: The CTO (and the Culture Owner)

The top rung blends technical vision, company strategy, and — per the book's final chapter, Ch.9 ("Bootstrapping Culture") — the job of setting the culture of the engineering function. There is no "The CTO" chapter (❌ verified absent); CTO-shaped questions live in the "Ask the CTO" sidebars and in Ch.8–9. The book's honest closing note is that the path is not one-way: it is common to try management, decide it is not for you, and step back to the technical track (Ch.3's closing observation, ⚠ via secondary summary).

---

## 3. Chapter 1: Management 101

**Where it sits.** The book opens here, with the reader as an engineer who must decide what to expect from a manager — and, eventually, whether to become one. The chapter opening was verified verbatim in the book text this run: *"Chapter 1. Management 101. The secret of managing is keeping…"* (snippet truncated by the search engine; the full sentence was not recovered — ⚠ see §14.2).

**Core concepts.**

- **Management is a different job, not a promotion.** The skills that made you a great engineer are not the skills that make you a great manager; your output becomes the team's output.
- **Three bad-manager patterns** the book names: benign neglect (invisible to the point of abandonment), micromanagement (control without trust), and outright abuse. Good management, by contrast, is: 1:1s, feedback and workplace guidance, and training/career-growth support.
- **The two tracks diverge early.** Managers get leverage through people; ICs get leverage through technical depth. Neither track is "higher."
- **How to be managed** (the chapter's famous other half): spend time thinking about what you want; take responsibility for asking; give your manager a break — they are human; and choose your manager wisely when you change roles.
- **The 1:1 as the foundation** — the book introduces its canonical meeting here: regular (weekly, ~30 minutes), report-driven agenda, not a status update (the framework recurs in Ch.2 and Ch.4).

**Key takeaways.**

1. Evaluate your current manager against the good/bad patterns before you decide anything about your own path.
2. If you want the management track, expect your individual output to drop toward zero — the win condition becomes the team shipping.
3. A great engineer can be a horrible manager; the two ladders test different things.

**Memorable advice.** *"A good manager helps you grow in your career, gives feedback, and is respectful"* (⚠ quoted via secondary summary — runn.io's chapter summary; consistent with the verified chapter content). The chapter's end-of-chapter assessment questions are the book's signature device: every chapter closes with reflective questions, and Ch.1's ask you to audit your own reporting relationship.

**Repo mapping.**

> - The good/bad-manager patterns and the IC-vs-manager fork connect to [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) §3 (managing up) and [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) §3–§6 (management styles and effectiveness).
> - "How to be managed" (owning your career, choosing your manager) is the inward layer of [managing_inward_guide.md](managing_inward_guide.md) §1–§2.
> - The 1:1 framework is built out in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §8 (One-on-Ones and Meetings).

---

## 4. Chapter 2: Mentoring

**Where it sits.** The first leadership role without authority. The chapter's intern section was verified verbatim this run (*"Mentoring an Intern. The first type of mentorship relationship we'll cover is the temporary employee"* — book text), and the mentor/mentee material (including the observation that the best mentoring relationships develop naturally) was confirmed at multiple points in the book text. The exact chapter heading "Mentoring" is corroborated by every independent summary (runn.io, ClickUp, the repo's research notes); the heading itself was not directly captured this run — ⚠ see §14.2.

**Core concepts.**

- **Mentoring is practice management in a safe environment** — people rarely get fired for bad mentorship (⚠ wording via secondary summary).
- **Mentoring an intern** — the book's playbook: this person knows little and may not stay; assign a real project, break it into milestones with a clear scope, pair them with a buddy, give frequent feedback, and treat the internship as an extended interview.
- **Mentoring a new hire** — a chance to see the company through fresh eyes; you explain the spoken *and* unspoken rules.
- **Technical vs. career mentoring** — the best relationships evolve naturally; forced pairings are often useless.
- **The alpha geek** — Fournier's label for the engineer who "values intelligence and technical skills above all other traits" (⚠ wording via secondary summary), believes they know best, undermines colleagues, and redoes work without warning. Alpha geeks make terrible managers unless mentorship teaches them to listen.
- **Effective 1:1s** — the full treatment appears here: weekly, ~30 minutes, a shared agenda document both parties add to during the week, report-driven, and not a status report.
- **Mentoring across difference** — acknowledge differences in background and identity rather than pretending they do not exist.
- **The mentor-vs-sponsor distinction** — ⚠ flagged: the repo's research notes place "mentor gives advice vs. sponsor opens doors" in this chapter, but an exact-phrase search for "sponsor" in the book text returned zero results this run; the distinction is standard mentoring literature but could not be verified as a named concept in this book (see §14.2).

**Key takeaways.**

1. Mentoring is the cheapest place to learn whether you enjoy developing people — take every safe opportunity.
2. Give interns a real project with milestones, not busywork; the internship is a two-way extended interview.
3. Watch for the alpha geek on your team — including in the mirror — because that pattern is the most common reason brilliant engineers fail as managers.

**Memorable advice.** The book's intern playbook (project + milestones + buddy + frequent feedback) is one of its most copy-pasted frameworks, and the 1:1 agenda-doc habit introduced here becomes the backbone of Ch.4.

**Repo mapping.**

> - The 1:1 and coaching mechanics live in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §3 (coaching and feedback frameworks) and §8 (one-on-ones).
> - The alpha-geek pattern is a personality hazard covered from the assessment angle in [personality_assessments_guide.md](personality_assessments_guide.md) §4–§5.
> - Mentoring as a development tool appears in [grow_team_guide.md](grow_team_guide.md) §5 (development) and [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) §14 (coaching and mentoring).

---

## 5. Chapter 3: Tech Lead

**Where it sits.** The hybrid role — the hardest job in the book because it asks you to keep delivering IC work *while* leading. Verified verbatim this run: *"Tech Lead. I became a tech lead many years ago. I had been promoted to senior engineer, and was working on a small team with severa…"* (book text; snippet truncated). The chapter also confirms the tech lead's 1:1 obligations — *"the tech lead is expected to manage these team members to the high management standards of RTR tech. These standards include: regular (weekly) 1-1 touchbases"* (book text, verbatim — the "RTR" reference is to Rent the Runway, Fournier's own experience).

**Core concepts.**

- **A role, not a rung** — "not a point on the ladder, but a set of responsibilities" (⚠ wording via secondary summary; the sentiment is confirmed by the verified chapter text).
- **The three hats** the book breaks the role into: systems architect / business analyst (what needs to change and be built), project planner (break work into deliverables, sequence it), and software developer / team leader (write some code, formulate challenges, delegate).
- **Keep writing code — but not too much** (⚠ wording via secondary summary). The balance problem is real: some days you are on a maker's schedule, some days on a manager's schedule.
- **The tech-lead/manager split** — two common models: the same person doing both, or a tech lead (technical direction) alongside a people manager (processes). Each has trade-offs.
- **Project management as a tech-lead skill** — break down the work, push through the unknowns, adjust the plan as you go, revisit details at the end.
- **Letting go** — the trap is trying to do 100% IC work and 100% leadership; tech leads must trust the team with code they love.
- **Recognizing when the role is breaking** — missed commitments, burnout, team distrust, degrading code quality.

**Key takeaways.**

1. The tech-lead role is where you learn that leadership without authority is possible — and that it is exhausting.
2. Protect a slice of technical work, but define success as the team's delivery, not your own line count.
3. When the role is not working, say so early; silent suffering is how tech leads burn out.

**Memorable advice.** *"It can be hard to balance the work of project management and oversight with hands-on technical delivery. Some days you're on a maker's schedule, and some days you're on a manager's schedule"* (⚠ quoted via secondary summary — runn.io). The chapter closes with the reassurance that the tracks are not one-way: trying management and stepping back to IC work is common and legitimate.

**Repo mapping.**

> - The entire tech-lead toolkit — role definition, delegation, decision-making, 1:1s — is the subject of [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §1, §6, §7, §8.
> - Project planning under uncertainty connects to [management_consulting_skills_guide.md](management_consulting_skills_guide.md) §2 (hypothesis-driven problem solving) and [business_case_development_guide.md](business_case_development_guide.md) (scope and trade-off discipline).
> - The hybrid-role pattern (technical delivery + stakeholder representation) mirrors the repo's [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) §4 (FDE vs related roles), where a technical operator also owns client-facing outcomes.

---

## 6. Chapter 4: Managing People

**Where it sits.** The core people-management chapter (the book's actual title — the repo's research notes called it "Managing Individual Contributors," a title that does not exist in the book text; ❌ verified absent). Chapter opening verified verbatim this run: *"Managing People. New engineering managers think of the job as a promotion, giving them seniority on engineering tasks and questions. This is a great approach for en…"* (book text; snippet truncated). This is the chapter the whole book builds toward: if you cannot manage individuals well, nothing else scales.

**Core concepts.**

- **Starting a new reporting relationship off right** — build trust and rapport (ask how they like feedback, what manager behaviors they hate); create a realistic 30/60/90-day plan; communicate your style and expectations explicitly; and get feedback from the new hire, taken with a grain of salt.
- **Goal setting** — connect individual goals to team and company objectives; the book's framing is consistent with OKR/SMART practice (⚠ exact framework names not re-verified in the book text this run).
- **Performance reviews** — continuous feedback (regular, real-time, positive and negative) supplemented by **360-degree feedback** (peers, reports, cross-functional partners, self-review). The book is explicit about **recency bias** — reviews over-weight recent events, so keep a running log (⚠ the "keep a running log" recommendation is from the repo research notes and runn.io's summary; the recency-bias discussion itself is confirmed in the book text).
- **Feedback: praise and criticism** — praise in public, criticism in private; direct, kind, specific; the book is against the "feedback sandwich" (praise-criticism-praise) because it muddies the message (⚠ wording via secondary summary).
- **Handling underperformers** — the emotionally hardest job in management: diagnose the root cause (skill, will, or fit), set a clear improvement plan with measurable milestones, document everything, and if there is no improvement, manage the person out with dignity — it should never be a surprise (the book's "performance improvement plan" concept is confirmed in the book's index, which lists "performance improvement plan").
- **Motivating different types** — high achievers need new challenges; struggling ICs need clarity and support; veterans need recognition and purpose.

**Key takeaways.**

1. The first 90 days of any reporting relationship set its trajectory — spend them on trust and explicit expectations.
2. Feedback given continuously is kinder than feedback saved for review season; 360-degree input is how you fight your own blind spots.
3. Underperformance is a diagnosis problem first and an administrative problem second — and the documentation is what makes a later exit humane and defensible.

**Memorable advice.** The 30/60/90-day plan and the 360-degree review are the two frameworks readers implement immediately after this chapter; both are confirmed in the book text or index.

**Repo mapping.**

> - The 360-degree review mechanics live in [360_management_guide.md](360_management_guide.md) §5 (360° feedback) and its assessment framework §12.
> - Feedback and difficult conversations are built out in [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) §9 (difficult conversations framework) and [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §3 (coaching and feedback frameworks).
> - Matching your approach to the person's development level is exactly [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) §4–§6 (styles S1–S4 matched to levels D1–D4); performance management at scale is [grow_team_guide.md](grow_team_guide.md) §6.

---

## 7. Chapter 5: Managing a Team

**Where it sits.** The step from managing individuals to managing a *team* (the book's actual title — "Managing Teams" as a chapter title does not exist in the book text; ❌ verified absent). Chapter opening verified verbatim this run: *"Managing a Team. It's a short step from managing a person or two to managing a whole team, but managing a team is more than just d…"* (book text; snippet truncated). This chapter covers hiring, onboarding, culture, conflict, and staying technical.

**Core concepts.**

- **The engineering lead must stay technical** — even if you stop writing code, you still guide technical decision making and make sure decisions "pass the technical smell test" (⚠ quoted via secondary summary — runn.io). The book is explicit that a purely administrative manager is a failure mode.
- **Team dynamics** — the forming-storming-norming-performing lifecycle shapes how the manager intervenes (⚠ the Tuckman naming is the standard label; the repo research notes associate it with this chapter; exact in-book terminology not re-verified this run).
- **Hiring and interviewing** — look for cultural *contribution* (not just "culture fit"), technical competence, communication, and growth mindset; structured interviewing to reduce bias (⚠ structured-interview framing via research notes; hiring as a chapter topic confirmed).
- **Onboarding** — structured ramp: pre-boarding, first week, first month (small wins, pairing), first 90 days to full productivity.
- **Conflict: avoider vs. tamer** — don't rely on consensus or voting; depersonalize decisions with standards; don't ignore simmering issues; don't take it out on other teams.
- **Cohesion destroyers** — the book names the **brilliant jerk** (toxic but high-output; don't hire them, don't promote them), the **noncommunicator**, and the employee who lacks respect. The "brilliant jerk" term is confirmed in the book's index and common-terms list.
- **Team culture** — culture as the (generally unspoken) rules of a community; the manager models the behaviors (the full "culture" treatment is Ch.9 — see §11).
- **Remote and distributed teams** — communication cadence, time-zone overlap, intentional cross-site collaboration (the book addressed this pre-pandemic; ⚠ depth not re-verified this run).
- **Process** — pick an agile flavor (Scrum, Kanban…) and run it well; don't cargo-cult it (⚠ framework list via research notes).

**Key takeaways.**

1. Your technical credibility is the currency of team trust — stay close enough to the work to pass the smell test.
2. Hire for contribution, onboard with a plan, and move fast on cohesion destroyers: one brilliant jerk can poison a whole team.
3. Conflict is normal; the manager's job is to make it productive and depersonalized.

**Memorable advice.** The "don't hire brilliant jerks" rule is one of the book's most-quoted lines in practice; combined with the onboarding ramp, Ch.5 is the chapter most often re-read before a hiring surge.

**Repo mapping.**

> - Hiring, onboarding, culture, and scaling are the whole subject of [grow_team_guide.md](grow_team_guide.md) §1–§4.
> - Team dynamics and psychological safety (the brilliant-jerk problem is a safety problem) are in [high_performance_team_guide.md](high_performance_team_guide.md) §4–§6, including its Cymbal Bank squad worked example §9.
> - Conflict resolution frameworks are in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §9 and [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) §8–§9.
> - The OB view of team dynamics (norms, roles, cohesion) is in [organizational_behavior_guide.md](organizational_behavior_guide.md) §4.

---

## 8. Chapter 6: Managing Multiple Teams

**Where it sits.** The first rung of "your job is now system design." Chapter opening verified verbatim this run: *"Managing Multiple Teams. Welcome to the world of multiple-team management! We're going to talk about managing multiple teams before we tal…"* (book text; snippet truncated — the "before we talk about managing managers" continuation explains why the book splits this material from Ch.7). Verified verbatim this run: *"As you navigate your new obligations, start to ask yourself: How important is the thing I'm doing? Does it seem to be important because it's urgent?"* (book text, from the importance-vs-urgency discussion — the snippet was captured with surrounding text in the book's Ch.6 pages).

**Core concepts.**

- **Importance vs. urgency** — urgent things scream, important things whisper; the book's warning is that managers of multiple teams drown in urgency unless they deliberately protect strategic time.
- **The delegation matrix** — the book's four-way split of tasks: simple-and-frequent → delegate; simple-and-infrequent → do yourself; complex-and-infrequent → do yourself (but delegate to rising leaders to train them); complex-and-frequent → delegate (project planning, system design — this is how you grow the team).
- **Delegation is a growth engine** — *"Delegation is a process that starts slow but turns into an essential element for career growth. If your teams can't operate well without you, you'll find it hard to be promoted"* (⚠ quoted via secondary summary — runn.io).
- **Scaling yourself** — you cannot attend every meeting or review every design; triage your attention ruthlessly.
- **Standard processes across teams** — on-call, incident response, reviews, promotions, technical decisions; consistency reduces cognitive load for everyone.
- **Cross-team coordination** — dependencies, shared roadmaps, preventing turf wars over ownership.
- **Warning signs of overwhelm** — you are the bottleneck; you cannot remember what each team is working on; your 1:1s are always cancelled; you are reactive all day.
- **Building the management pipeline** — identify and groom the tech leads and managers beneath you.

**Key takeaways.**

1. At this rung your output is the *system* — processes, standards, and leaders — not any single team's delivery.
2. Use the four-way delegation matrix on your own calendar every week; the complex-and-frequent quadrant is your leadership-development lever.
3. If you feel constantly urgent, you have a prioritization (or a delegation) problem, not a time problem.

**Memorable advice.** The importance-vs-urgency question ("Does it seem to be important because it's urgent?") is the chapter's most reusable tool — it is the same discipline the repo covers as the Eisenhower matrix.

**Repo mapping.**

> - The Eisenhower matrix and time-blocking discipline are in [time_energy_management_guide.md](time_energy_management_guide.md) §4 and §3 — the direct toolkit for the importance-vs-urgency theme.
> - Delegation frameworks (including authority-with-responsibility) are in [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §6 and [360_management_guide.md](360_management_guide.md) §9 (360° delegation and empowerment).
> - Scaling the org (process standardization, pipeline building) connects to [grow_team_guide.md](grow_team_guide.md) §4 (scaling) and [organizational_behavior_guide.md](organizational_behavior_guide.md) §6 (the organization level).

---

## 9. Chapter 7: Managing Managers

**Where it sits.** The sandwich rung — this companion's label, not the book's (see §12.8 for the term and §14.2 for the ⚠ phrase check): for the first time you are squeezed between the manager above you and the managers who report to you. The chapter title is verified — "Managing Managers" (✅ O'Reilly audiobook contents listing, Google Books) — and the chapter's core move is the shift from leading people to *leading people who lead people*. No chapter-opening snippet was captured verbatim this run (⚠ — see §14.2); the chapter content below is reconstructed from Page Posse's per-chapter summary (⚠ secondary) and the repo's research notes, whose "Managing Stakeholders" title for this material is ❌ not a chapter title in the book — that material is re-mapped here by theme, as the upward half of the sandwich.

**Core concepts.**

- **The sandwich, structurally** — your direct reports are managers now; you manage up (to your own manager) and down (to the managers who report to you) at the same time. Every layer of management is another layer of abstraction between you and what is actually happening on the ground (⚠ via Page Posse summary).
- **Verify the details** — do not trust the rosy summary from the layer below: check recruiting, 1:1s, project status, and postmortems yourself. The chapter's central warning is that managers of managers go blind through *distance*, not laziness (⚠ via Page Posse).
- **Skip-level meetings** — quarterly 1:1s or group lunches with your reports' reports: unfiltered signal, trust beyond a resource-level view of people, and a sanity check on your strategy. Fournier warns against the open-door policy that invites vague venting — ask people to bring a structured problem instead (⚠ via Page Posse).
- **The people-pleaser manager** — the signature dysfunction of this rung: hides problems, overpromises, burns out trying to satisfy everyone. Verifying the details is exactly what catches it (⚠ via Page Posse).
- **Upward management** — understand what your own boss cares about, adapt to their communication style, and don't bring problems without proposed solutions (⚠ via research notes / secondary summaries).
- **Communicating with your boss** — regular reporting; bad news delivered early, transparently, with a recovery plan attached (⚠ via research notes).
- **Saying no** — to stakeholders, to your boss, to your team: explain the reasoning and offer alternatives; saying no well preserves relationships instead of burning them (⚠ via research notes).
- **The eng/product relationship** — the most critical partnership in a technology organization: shared ownership, joint roadmap planning, and explicit trade-off conversations when scope and priorities collide (⚠ via research notes).
- **Organizational politics** — the book's pragmatic take (per the research notes): politics is not inherently bad; it is how decisions actually get made. Understand the power structure, build alliances, stay principled (⚠ via research notes).
- **Coaching new managers** — 1:1 structure, letting go of their old IC work, avoiding micromanagement, and training beyond the company's own curriculum (⚠ via Page Posse).
- **Debugging teams from their data** — chat, tickets, code reviews, meetings — and staying technical enough to ask informed questions. Fournier's tell for a sick team is memorably ordinary: boring meetings (⚠ via Page Posse).

**Key takeaways.**

1. Leading through other managers means seeing through other people's summaries — skip-levels and verified details are how you keep your eyes open.
2. The upward half of the sandwich is a real job: manage your boss's context, say no with reasoning, and treat organizational politics as a skill rather than a vice.
3. Watch for the people-pleaser manager under you — the problem-hider — because they will look like your best report right up until they aren't.

**Memorable advice.** No verbatim quote from this chapter was captured this run (⚠ — the Ch.7 opening sits in the truncated-snippet list in §14.2). The chapter's most reusable instruments, per the chapter summaries, are the skip-level meeting and the verify-the-details discipline — the two habits that keep a manager-of-managers honest.

**Repo mapping.**

> - The managing-up framework — the sandwich-boss theme in depth — is the whole subject of [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) §3–§5 (managing up, down, and sideways).
> - Stakeholder and influence toolkits — saying no, difficult conversations, influencing without authority — are in [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) §7–§9 and [management_consulting_skills_guide.md](management_consulting_skills_guide.md) (stakeholder analysis and influence toolkits).
> - 360° stakeholder management (the stakeholder map taken all the way around the org) is in [360_management_guide.md](360_management_guide.md) §7–§9.

---

## 10. Chapter 8: The Big Leagues

**Where it sits.** The senior-leader rung. Verified chapter title: "The Big Leagues" (✅ O'Reilly audiobook contents listing, Google Books). The repo's research notes called this rung "The Engineering Director" — the *role* exists, but that chapter title is ❌ absent from the book (see §14.1). Content below is reconstructed from Page Posse's Ch.8 summary (⚠ secondary) plus the research notes' director-level material re-mapped by theme; no verbatim Ch.8 snippet was captured this run (⚠ — see §14.2). One honest flag up front: **"the lonely middle" is this companion's label for a theme, not a verified phrase from the book** — an exact-phrase search for it in the book text returned zero results this run (⚠ see §14.1–14.2).

**Core concepts.**

- **Two senior roles, separated** — the VP of Engineering owns execution and operational excellence; the CTO shapes business strategy through the lens of technology and anticipates where the technology has to go. A CTO without real influence over the business, the book argues, isn't actually doing the job (⚠ via Page Posse).
- **Strategy and vision** — where the organization invests, what it deprioritizes, and how engineering strategy ties to business outcomes; the chapter's questions are portfolio questions, not backlog questions (⚠ via research notes).
- **Org design** — team size, reporting structures, alignment with product pillars, and knowing when to reorganize; structure is a strategic instrument at this rung (⚠ via research notes).
- **Working with senior staff** — directors often manage engineers who are more technically senior than they are (staff+ ICs): give them autonomy, a clear mission, and public recognition; earn their trust rather than asserting rank (⚠ via research notes).
- **Communicating with executives and the board** — translate technical topics for a non-technical audience; the board cares about risk, talent, velocity, and innovation (⚠ via research notes; board-level material).
- **Say everything three times** — in person, in writing, and again in a meeting; a message rarely lands the first time, and over-communication is the senior leader's core skill (⚠ via Page Posse).
- **Kill old work to make new priorities real** — a new priority is credible only when something old is explicitly stopped or postponed, not piled on top of (⚠ via Page Posse).
- **Disagree and commit** — argue hard in the room, then deliver the decision excellently and present a unified front once it is made (⚠ via Page Posse).
- **Delivering bad news at altitude** — in person, in small groups, standing fully behind the message; keep apologies brief; use small talk to make it safe for problems to surface (⚠ via Page Posse).
- **A true north** — the core principles that guide decisions when the situation runs past any playbook (⚠ via Page Posse).
- **Succession and the bench** — building leaders who can take over your role and the roles below you (⚠ via research notes).
- **The lonely middle** — ⚠ not a verified book phrase (see the flag above): the theme is that senior leaders sit between executives and the organization, translating both ways, with few peers to confide in; the research notes' director material treats peer networks, coaching, and intentional self-care as the coping toolkit.

**Key takeaways.**

1. At this rung your job is direction and communication, not decision-making heroics — the failure modes are announcing once and assuming it landed, wavering in public after a call is made, and never killing old work.
2. Separate the VP/execution role from the CTO/strategy role even when one person holds both hats.
3. The isolation of the middle is real (⚠ theme, not verified phrase) — build your peer network before you need it.

**Memorable advice.** "Say everything three times" is the chapter's most quotable rule (⚠ via Page Posse summary — not captured verbatim from the book this run); the disagree-and-commit discipline is its most load-bearing one.

**Repo mapping.**

> - Strategy formulation for the senior rung is in [strategic_management_guide.md](strategic_management_guide.md) and [it_strategy_guide.md](it_strategy_guide.md) — the direct toolkit for Ch.8's strategy material.
> - Org design, structure, and the organization level of behavior are in [organizational_behavior_guide.md](organizational_behavior_guide.md) §6–§8.
> - Scaling the organization (structure, process standardization, leadership pipeline) is [grow_team_guide.md](grow_team_guide.md) §4 (scaling).
> - Management styles and senior-leadership effectiveness are in [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) §5–§6.

---

## 11. Chapter 9: Bootstrapping Culture

**Where it sits.** The book's final chapter and the top of the ladder: the senior leader as culture owner. Verified chapter title: "Bootstrapping Culture" (✅ O'Reilly audiobook contents listing, Google Books). Content below is reconstructed from Page Posse's Ch.9 summary (⚠ secondary) and the repo's research notes; no verbatim Ch.9 snippet was captured this run (⚠ — see §14.2). The research notes' "The CTO" chapter material (CTO archetypes, board management, when to step down) is ❌ not a chapter in the book — where it genuinely fits it is folded into the senior-leader themes of Ch.8–9 and the "Ask the CTO" sidebars, and flagged ⚠ (see §14.1).

**Core concepts.**

- **Culture is how things get done without anyone thinking about it** — the chapter's anchor definition (⚠ via Page Posse; consistent with the "generally unspoken rules" framing the book uses at Ch.5 — see §7). Unspoken rules shape behavior automatically, so a culture will form whether you tend it or not; the only question is whether anyone chose it on purpose.
- **Build it deliberately** — the chapter is for anyone setting up, changing, or rescuing a team's culture; the work is explicit and unglamorous, and it is the senior leader's job to own it.
- **Explicit values** — written down and used in hiring and recognition; if your own values are not the company's, you will struggle (⚠ via Page Posse).
- **The written career ladder** — the chapter's most concrete artifact: built with senior staff, checked against external examples, aligned to compensation bands, marked with clear promotion breakpoints, and split into management and technical tracks so no one has to become a manager just to advance (⚠ via Page Posse). This is the same ladder this companion lays out in §2, and the artifact the worked example builds in §13.5.
- **Process as risk management** — not bureaucracy: add process to eliminate the critical or frequent risks, not all risk; design "half-baked" processes that still add value when followed imperfectly; and depersonalize decisions with automated gates like linters and tests (⚠ via Page Posse).
- **Cross-functional teams** — break down "us versus them," a nod to Conway's Law (the organization's structure shapes the systems people build) (⚠ via Page Posse).
- **Blameless postmortems** — examine circumstances instead of hunting for a culprit; the honest postmortem is itself a culture artifact (⚠ via Page Posse).
- **Structures as hypotheses** — treat every new structure as an experiment; you learn the most from failures. A startup can steer like a racing car; scale asks for the steadier controls of a commercial flight (⚠ via Page Posse).
- **The final turn** — the book ends by turning from managing others to managing yourself: at the top, what you do is by definition read as "good," so modeling the behavior you want *is* the culture work (⚠ via Page Posse).
- **The "CTO" themes (⚠ not a chapter)** — the research notes' CTO archetypes (platform/product/visionary/operational), board management, and knowing when to step down are senior-leader themes the book treats across Ch.8–9 and the "Ask the CTO" sidebars; they are not a standalone chapter, and none of it was verified verbatim this run (see §14.2).

**Key takeaways.**

1. Culture will be built anyway — by the loudest or earliest people — so build it on purpose with explicit artifacts: values, a written career ladder, blameless postmortems.
2. Process is risk management: target the critical and frequent risks, and keep the process light enough to survive imperfection.
3. The culture owner's main tool is their own behavior — at the top, what you do is what the culture becomes.

**Memorable advice.** The written career ladder — "no one has to become a manager just to advance" — is the chapter's most copy-pasted artifact (⚠ via Page Posse; the dual-track framing matches the book's Ch.1 IC-vs-manager fork from §3).

**Repo mapping.**

> - Culture and scaling are in [grow_team_guide.md](grow_team_guide.md) §1–§4 (culture, hiring, onboarding, scaling) — the direct toolkit for Ch.9.
> - Team culture and psychological safety (the conditions that make values real) are in [high_performance_team_guide.md](high_performance_team_guide.md) §4–§6.
> - Norms, values, and the organization level of behavior are in [organizational_behavior_guide.md](organizational_behavior_guide.md) §6–§8.
> - Values-to-strategy alignment and measurement are in [balanced_scorecard_guide.md](balanced_scorecard_guide.md) (strategy maps and value alignment, with a Cymbal Bank example).

---

## 12. The Core Themes

Ten threads run through the whole book. Each gets a short subsection with its cross-references; the worked example in §13 exercises most of them end to end.

### 12.1 The IC-to-Manager Transition

The book's central leap (Ch.1, Ch.4): management is a different job, not a promotion. Your individual output drops toward zero and your win condition becomes the team shipping. The research notes' "management leap" — letting go of "I shipped that" for "my team shipped that" — is the identity shift every later theme assumes. Cross-ref [managing_inward_guide.md](managing_inward_guide.md) §1–§2 (the self-management layer) and §2 of this guide (the ladder).

### 12.2 The Tech-Lead Hybrid Role

Ch.3's "not a point on the ladder, but a set of responsibilities" (⚠ via secondary summary): keep writing some code, plan the work, represent the team, and learn to let go of code you love. The hardest job in the book because it is two jobs at once. Cross-ref [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §1, §6–§8.

### 12.3 Mentoring as First Leadership

Ch.2: the first leadership role without authority, and the book's recommended safe place to practice management. The 1:1 habit is built here, and the alpha geek is diagnosed here. Cross-ref [grow_team_guide.md](grow_team_guide.md) §5 (development) and [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) §14 (coaching and mentoring).

### 12.4 1:1s as the Foundation

The canonical meeting, introduced in Ch.1 and refined in Ch.2 and Ch.4: weekly, ~30 minutes, report-driven, a shared agenda document both parties add to during the week, and never a status update. Nearly every chapter assumes it. Cross-ref [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §8 (one-on-ones).

### 12.5 Feedback Is a Gift

Ch.4's philosophy: direct, kind, specific; praise in public, criticism in private; continuous feedback rather than review-season dumps; and no "feedback sandwich" — the praise-criticism-praise sandwich muddies the message (⚠ wording via secondary summary). 360-degree reviews fight the manager's own blind spots. Cross-ref [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) §9 and [360_management_guide.md](360_management_guide.md) §5.

### 12.6 Delegation Is Not Dumping

The four-way delegation matrix of Ch.6 — simple-and-frequent → delegate; simple-and-infrequent → do yourself; complex-and-infrequent → do yourself (delegating to rising leaders to train them); complex-and-frequent → delegate — is the book's scaling engine: delegate outcomes with context and authority, follow up on milestones, and remember that teams that cannot operate without you keep you unpromotable (⚠ the growth quote via runn.io). Cross-ref [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) §6 and [360_management_guide.md](360_management_guide.md) §9.

### 12.7 Team Building

Hiring for contribution (not just fit), structured onboarding to full productivity by day 90, and culture modeled by the manager (Ch.5): the "don't hire brilliant jerks" rule and the cohesion-destroyer list. Cross-ref [grow_team_guide.md](grow_team_guide.md) §1–§4 and [high_performance_team_guide.md](high_performance_team_guide.md) §4–§6.

### 12.8 The Sandwich Boss

The middle-manager squeeze (Ch.6–7): managing up to your own manager and down to the managers who report to you at once — regulatory-project pressure from above, delivery pressure from below. "Sandwich boss" is this companion's term, not a verified book phrase (⚠ see §14.1); the squeeze itself is the book's Ch.7 subject. The framework in depth: [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) — and see the worked example §13.4 for the squeeze in action at Cymbal Bank.

### 12.9 Culture

From Ch.5's "generally unspoken rules" (⚠ via research notes) to Ch.9's deliberate bootstrapping: values made explicit, a written career ladder, process as risk management, blameless postmortems. Culture forms whether you tend it or not — the senior leader's job is to choose it on purpose. Cross-ref [grow_team_guide.md](grow_team_guide.md) §1–§4 and [organizational_behavior_guide.md](organizational_behavior_guide.md) §6–§8.

### 12.10 Managing Your Own Manager

The book's quiet other half, from Ch.1's "how to be managed" through Ch.7's upward management: know what you want, ask for it, give your manager a break, adapt to their style, bring solutions with problems, and say no with reasoning. Cross-ref [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) §3 (managing up).

---

## 13. The Worked Example: The Cymbal Bank Tech-Lead-to-Manager Progression

**Scenario.** Priya is a solution architect on the payments platform at **Cymbal Bank** (the repo's fictional
Singapore bank — see the banking guides in the front-matter cross-references for context: real-time payments,
MAS-style regulation, a universal banking model). Over four years she is promoted: senior engineer → tech lead of
the real-time payments squad → engineering manager of two squads → manager of managers for the payments platform
group. The progression below walks each rung through the book's framework, applying concrete beats from the
chapters — and it is honest about the book's warnings: the alpha geek on the squad (Ch.2), the tech lead who can't
let go (Ch.3), and the sandwich squeeze at the two-squad rung (Ch.6–7).

### 13.1 Rung 0–1: The engineer who mentors (Ch.1–2)

Before any promotion, Priya does Ch.1's homework: she audits her own manager against the good/bad patterns (1:1s,
feedback, career support vs. benign neglect), decides she wants the management track, and tells her manager —
taking responsibility for asking, per Ch.1's "how to be managed." She practices on the cheapest rung: mentoring the
squad's interns and new associates (Ch.2), giving each intern a real project with milestones and a buddy. Her
intern project is typical of the chapter's playbook: a small idempotent replay tool for failed payment callbacks,
scoped into weekly milestones, with a code-review buddy and feedback after every review — an extended interview in
both directions.

She also builds the 1:1 habit that will carry her whole career: a shared agenda doc with two sections ("for me",
"for you") plus a follow-ups list, both parties adding items during the week, and the meeting itself never a status
update. The squad's alpha geek — brilliant, dismissive of juniors, redoing others' work without warning — teaches
her the book's early warning: that pattern is the most common reason brilliant engineers fail as managers,
including, sometimes, in the mirror.

### 13.2 Rung 2: Tech lead of the real-time payments squad (Ch.3)

Promoted to tech lead, Priya wears the three hats: architect (the ISO-20022-style message upgrade), planner
(breaking the work into sequenced deliverables), and player-coach (writing some code, delegating the rest). The
book's warning lands immediately: she is the tech lead who can't let go — she re-reviews every pull request, and
when the callback router's retry logic needs a rewrite she quietly re-implements it herself rather than coaching
the engineer who owns it.

Her manager's intervention — "define success as the squad's delivery, not your line count" — plus an early
application of the Ch.6 delegation categories (simple-and-frequent tasks delegated first) pull her out of the trap.
She learns the maker's/manager's schedule reality the hard way: her own coding moves to the first ninety minutes
before the stand-up, design reviews are delegated to the senior engineer with Priya reviewing only the decision
record, and she protects that deep-work slot the way Ch.6 later tells her to protect strategic time.

### 13.3 Rung 3: Engineering manager (Ch.4–5)

Priya's first reporting relationships get the Ch.4 treatment: a **30/60/90-day trust-building plan** with each new
report (first 30 days: listen and learn — how do you like feedback, what manager behaviors do you hate; days 30–60:
small wins and explicit expectations; days 60–90: goals tied to the squad's and the bank's objectives), and
**weekly report-driven 1:1s on a shared agenda doc**. She adopts the **feedback-without-sandwich habit**: praise in
public, criticism in private, direct and specific, delivered continuously instead of saved for review season, with
360-degree input at review time. A typical script: "When you missed the sprint-review prep, the squad had to
re-plan live in the meeting. Next time send the numbers the day before — I'll help you build the checklist."

When one engineer's delivery slips for three sprints, she diagnoses skill vs. will vs. fit before acting, sets a
clear improvement plan with measurable milestones (two weeks to a working fix, four weeks to a stable cadence),
documents everything, and — when there is no improvement — manages the person out with dignity and no surprises. On
the team rung (Ch.5), she hires for contribution and runs a structured onboarding ramp (pre-boarding, first-week
pairings, a first-month small win, full productivity by day 90) — and, the story she tells most, declines to hire a
brilliant jerk even under delivery pressure, because the book's cohesion-destroyer list is exactly what happened to
a neighbouring squad that hired one.

### 13.4 Rung 4: The sandwich squeeze (Ch.6–7)

Now engineering manager of two squads — real-time payments and payments settlements — Priya hits the sandwich-boss
squeeze in its purest form: a MAS-style regulatory project (a settlement-reporting modernization under a Monetary
Authority of Singapore-style deadline) presses down from the head of payments platform above, while both squads'
delivery pressure presses up from below. She applies Ch.6's tools: the **delegation matrix** on the squads'
recurring work — simple-and-frequent items (on-call triage, routine releases) delegated outright;
complex-and-frequent work (the regulatory integration design) delegated to her two tech leads as their growth
assignments; and only the complex-and-infrequent decisions (the outage postmortem, the promotion case) kept on her
own plate.

Under the regulatory deadline she asks the chapter's question daily: *"How important is the thing I'm doing? Does
it seem to be important because it's urgent?"* — and uses the answer to shield the squads from
urgent-but-unimportant noise (the dashboard "quick wins" that scream loudest) while protecting the regulatory path.
Managing up (Ch.7), she learns her boss's communication style, sends bad news early with a recovery plan attached,
and says no to the third unplanned request with the book's move: "We can take the quick win or hold the regulatory
milestone — not both; which one do you want, and what do we drop?" Managing down, she runs **skip-levels** with
both squads' engineers quarterly — which is how she catches the **people-pleaser**: a settlements engineer mentions
in a skip-level that "the release keeps slipping but Ravi says it's fine," and Priya verifies the details (Ch.7's
rule) before the slippage becomes a regulatory surprise. With the business and product owners (Ch.7), she treats
the eng/product relationship as shared ownership: joint roadmap reviews, data-driven trade-off conversations, and a
reputation built on consistent delivery.

### 13.5 Rung 5: Manager of managers for the payments platform group (Ch.8–9)

Two years later Priya leads the payments platform group: five managers, ~40 engineers, and the bank's
regulatory-critical path. Ch.8's senior rung becomes her daily work: she separates the execution role from the
strategy role, presents the regulatory program to the bank's technology committee in business terms (risk, talent,
velocity — not message formats), and practices the senior-leader communication rules: say everything three times,
kill old work when a new priority lands (she formally retires the legacy batch-reporting project so the
modernization is credible), and disagree in the room then commit in public.

She designs the group's org structure around product pillars, not convenience — two squads per pillar, the
settlements squad aligned under the regulatory program — and builds a bench: her two tech leads from §13.4 are now
managers, and she grooms their successors. Ch.9's culture artifacts are her legacy project: a **written career
ladder for the payments platform group** — levels L1–L5 on two tracks (engineering and management), built with her
senior staff, checked against external examples, aligned to the bank's compensation bands with clear promotion
breakpoints; explicit values ("replayable incidents", "blame-free postmortems", "own your handoffs") used in hiring
and recognition; and blameless postmortems for the group's incidents, examined as circumstances rather than
culprits.

### 13.6 The progression at a glance

| Rung | Chapter(s) | The artifact Priya uses | The book's warning at this rung |
|------|-------------|-------------------------|---------------------------------|
| Mentor | Ch.2 | intern playbook; the 1:1 agenda-doc habit | the alpha geek |
| Tech lead | Ch.3 | the three hats; delegation starts | the tech lead who can't let go |
| Engineering manager | Ch.4–5 | 30/60/90 plan; report-driven 1:1s; feedback without the sandwich | underperformance handled too late |
| Manager of managers (squeeze) | Ch.6–7 | delegation matrix; importance-vs-urgency; skip-levels; saying no | the people-pleaser manager |
| Group lead | Ch.8–9 | say-everything-three-times; org design; career ladder; blameless postmortems | the lonely middle (⚠ theme) |

**The book's warnings, honored.** The worked example is deliberately not a straight-line success story: the alpha
geek on her squad was the reason Priya learned to listen before promoting anyone; the tech lead who couldn't let go
was herself, for a season; the sandwich squeeze cost her a 1:1 slot with every report before she rebuilt her
calendar around the delegation matrix; and the people-pleaser under her taught her that her best-looking report
could be her biggest risk. The book's honest closing note applies to her too: the path is not one-way, and staying
on it means re-deciding, every few years, that this is still the job she wants — the same re-decision Ch.3 offered
to every tech lead who ever stepped back to the IC track.

---



## 14. The Claims Audit, the What-Could-Not-Be-Verified List, the Glossary, and the Closing

This final section consolidates everything this companion claims — and everything it could
not confirm — into one honest ledger, followed by the verification notes, the glossary, and
the closing. It is the section the front matter and §2 point to: every ⚠ and ❌ flag raised
in §§1–13 is settled here.

### 14.1 The Claims Audit

The consolidated ledger for every claim this companion makes. ✅ = verified this run against a
primary source; ⚠ = plausible but supported only by secondary sources; ❌ = contradicted by
verification. Sources are named per row; the methods behind them are described in §14.3.

| Claim | Status | Source |
|-------|--------|--------|
| Publication year 2017 — ebook March 13, 2017 | ✅ | Google Books record |
| Paperback May 2, 2017 | ✅ | Amazon product page |
| Print ISBN-13 978-1491973899 (ISBN-10 1491973897) | ✅ | Amazon, MARC record, VitalSource |
| eBook ISBN-13 978-1491973844 (ISBN-10 1491973846) | ✅ | Google Books, VitalSource |
| Page count ~241 | ⚠ — Amazon 241; MARC "xiv, 226 pages"; Google Books ebook 244 (reconciled in §14.2) | Amazon / MARC / Google Books |
| Author career: BS Carnegie Mellon, MS University of Wisconsin–Madison; ~18 months at Microsoft | ✅ | Wikipedia |
| Goldman Sachs: engineer (credit-risk software) → VP of Technology (~6 years) | ✅ | Wikipedia |
| Rent the Runway: Director of Engineering (2011) → CTO (2014), departed 2015 | ✅ — corrects the research notes' "CTO 2013–2017", which is ❌ | Wikipedia (Business Insider 2014, Fortune 2015) |
| Two Sigma: "Head of Platform Engineering" vs. "Managing Director" | ⚠ — titles differ across sources | Lenny's Newsletter vs. Crunchbase |
| JPMorgan: Managing Director; "Global Head of Engineering and Architecture" | ✅/⚠ — MD confirmed; exact title wording varies | Wikipedia / Lenny's Newsletter |
| CoreWeave: VP Engineering, Common Services (2023–) | ✅ | Wikipedia + LinkedIn |
| Apache ZooKeeper committer and PMC member | ✅ | Crunchbase |
| Verified 9-chapter ToC: Introduction; Management 101; Mentoring; Tech Lead; Managing People; Managing a Team; Managing Multiple Teams; Managing Managers; The Big Leagues; Bootstrapping Culture; Conclusion | ✅ | O'Reilly audiobook contents listing (learning.oreilly.com) + Google Books search-within-book |
| "Managing Stakeholders" is a chapter title | ❌ — not in the book's ToC or text; its material was re-mapped to Ch.7 by theme in §9 | book ToC / Google Books |
| "The Engineering Director" is a chapter title | ❌ — the role is covered in Ch.8 (see §10) | book ToC / Google Books |
| "The CTO" is a chapter title | ❌ — CTO themes live in Ch.8–9 and the "Ask the CTO" sidebars (see §10–§11) | book ToC / Google Books |
| Verbatim book-text snippets: the introduction quote; the Ch.1, Ch.3, Ch.4, Ch.5, Ch.6 openings; the intern-playbook passage; the RTR 1:1 passage; the importance-vs-urgency question | ✅ | Google Books search-within-book (see §14.3) |
| Quotes sourced only from secondary summaries (runn.io, ClickUp): "A good manager helps you grow…", tech lead "not a point on the ladder", the delegation-growth quote, "people rarely get fired for bad mentorship", the feedback-sandwich wording | ⚠ | runn.io / ClickUp summaries |
| The mentor-vs-sponsor distinction appears in the book | ⚠ — exact-phrase search for "sponsor" in the book text returned **zero results** this run; the distinction is standard mentoring literature but unverified in this book (§4, §14.2) | Google Books search-within-book |
| "the lonely middle" appears in the book | ⚠ — phrase not found; the isolation-of-senior-leaders theme rests on secondary summaries (§10) | Google Books search-within-book |
| "sandwich boss" / the sandwich phrasing appears in the book | ⚠ — not found as a named term for the middle-manager squeeze; "feedback sandwich" as a rejected feedback style is reported only via secondary summaries (§6, §14.2) | Google Books search-within-book |
| Amazon rating 4.6 / 5 stars, 1,352 ratings | ✅ | Amazon (this run) |
| Goodreads ~4.2 / 30,000+ ratings | ⚠ — from the repo's research notes only, not re-verified this run | research notes |
| Sibling-guide cross-reference targets exist | ✅ — every guide linked in §§3–13 was confirmed present in the repo this run | filesystem check (this run) |
| Audiobook exists (O'Reilly, Audible) | ⚠ — not re-verified this run; O'Reilly's catalog pages block automated scraping (§14.3) | research notes |

**Reading the audit honestly.** Two rows above deserve emphasis. First, the ❌ rows are not
claims that the book's *content* is missing — the "Managing Stakeholders", "The Engineering
Director", and "The CTO" material clearly exists in the book; what is ❌ is the *chapter
titles* the research notes attached to it. This guide re-mapped that content by theme
(§6–§11) rather than discarding it, and flagged the re-mapping wherever it rested on
secondary summaries. Second, the ⚠ rows on "sponsor", "the lonely middle", and "sandwich" are
negative search results: an exact-phrase search in the book text returned zero hits, which
demonstrates absence of the *phrase* but not absence of the *concept* — the book may discuss
sponsorship, isolation, and the middle-manager squeeze in other words. The guide flags the
phrasing, not the theme.

One correction deserves its own note because it touches the author's biography: the repo's
research notes listed Camille Fournier as "CTO of Rent the Runway 2013–2017". Verification
this run (Wikipedia, cross-checked against Business Insider 2014 and Fortune 2015 coverage)
shows she was Director of Engineering from 2011, became CTO in 2014, and departed in 2015.
The old dates are ❌; the corrected row above is ✅.

### 14.2 What Could Not Be Verified

The honest list — everything this companion could not confirm directly this run, and why:

- Full verbatim text of several chapter-opening sentences: Google Books search-within-book
  returns truncated snippets, so the complete first sentences of Ch.1 ("The secret of managing
  is keeping…"), Ch.3, Ch.4, Ch.5, and Ch.6 were not recovered.
- Chapter 7, 8, and 9 opening sentences: no verbatim snippets were captured for the final
  three chapters at all; their content rests on Page Posse's per-chapter summaries and the
  repo's research notes (both secondary).
- Quotes attributed only via secondary summaries (runn.io, ClickUp, the research notes): "A
  good manager helps you grow in your career, gives feedback, and is respectful"; the tech-lead
  "not a point on the ladder, but a set of responsibilities"; "Delegation is a process that
  starts slow…"; "people rarely get fired for bad mentorship"; the "feedback sandwich"
  criticism; "Culture is the generally unspoken rules of a community." All are consistent with
  verified chapter content but were not captured from the book's own text.
- The exact chapter heading "Mentoring" was not directly captured from the book text this run;
  it is corroborated by every independent summary (runn.io, ClickUp, research notes) and by
  the verified ToC listing.
- The "sponsor" concept: an exact-phrase search for "sponsor" in the book text returned zero
  results, so the mentor-vs-sponsor distinction cannot be confirmed (or denied) as a named
  concept in the book.
- "The lonely middle" phrasing: exact-phrase search returned zero results; the
  isolation-of-senior-leaders theme rests on secondary summaries of the senior-leader material.
- The research notes' chapter titles for Ch.4–9 ("Managing Individual Contributors", "Managing
  Teams", "Managing Stakeholders", "The Engineering Director", "The CTO") do not match the
  verified ToC; their per-chapter content was re-mapped by theme in §6–§11, but the exact
  page-level attribution of any individual item could not be checked without the full text.
- Page-count reconciliation: Amazon 241, MARC "xiv, 226 pages", Google Books ebook 244 —
  three catalog sources, three numbers; the discrepancy was not resolved.
- The audiobook: existence and narrator not re-verified this run (see §14.3 on O'Reilly
  scraping).
- The CTO-archetype taxonomy (platform/product/visionary/operational CTO) and the
  board-management material: attributed by the research notes to a "The CTO" chapter that does
  not exist; whether the taxonomy appears inside Ch.8–9 or the sidebars could not be verified.
- Ch.8–9 specifics — "say everything three times", "disagree and commit", the career-ladder
  artifact, blameless postmortems — come from Page Posse's chapter summaries, not from the
  book text directly.
- The print-edition page numbers for key frameworks (30/60/90, the delegation matrix,
  skip-levels) could not be verified.
- The "Ask the CTO" sidebars: named in the research notes and the author bios; their exact
  in-book name and placement were not verified this run.
- Goodreads rating and the "bible for new engineering managers" reputation: community and
  aggregator claims from the research notes, not re-verified.
- Whether any edition differences exist: only a first edition is catalogued (MARC, Amazon), so
  no edition diff was needed — but the completeness of that absence check was not exhaustively
  verified.

### 14.3 The Verification Notes

Sources used this run:

- **O'Reilly audiobook contents listing** (learning.oreilly.com) — the authoritative 9-chapter
  ToC (Introduction; Ch.1–9; Conclusion). ✅
- **Google Books search-within-book** — the verified verbatim snippets (introduction quote;
  chapter openings for Ch.1, 3, 4, 5, 6; the intern passage; the RTR 1:1 passage; the
  importance-vs-urgency question) and the negative phrase checks ("sponsor", "the lonely
  middle", "sandwich").
- **Wikipedia** (Camille Fournier) — career dates and roles, including the Rent the Runway CTO
  2014–2015 correction.
- **Amazon** — ISBNs, publication dates, page count, and the 4.6/1,352 rating.
- **Crunchbase** — Two Sigma and JPMorgan titles; ZooKeeper PMC membership.
- **Lenny's Newsletter** — Two Sigma and JPMorgan role titles.
- **Page Posse** — per-chapter guides for Ch.7–9, used as the secondary summaries for the
  final three chapters.
- **The repo's research notes** (the_managers_path_research.md) — the base material this guide
  corrects and extends.

**Honest note on O'Reilly:** O'Reilly's catalog pages block automated scraping (product-page
requests are rejected), so the O'Reilly-side verification used the audiobook contents listing
on learning.oreilly.com rather than the catalog pages. The prior subagent's evidence log —
including the captured Google Books snippets and its reasoning — is retained in the delegation
cache at /home/ubuntu/.hermes/cache/delegation/live/deleg_76eae265/task-0.log.

### 14.4 The Glossary

**IC** — individual contributor: an engineer whose output is their own technical work, on the
non-management track (Ch.1; §2.1).

**Tech lead** — the engineer accountable for a team's technical direction who also writes some
code: architect, planner, and player-coach in one role (Ch.3; §5).

**1:1** — the weekly ~30-minute one-on-one meeting, report-driven, with a shared agenda
document both parties add to during the week; the book's foundational management practice
(Ch.1, Ch.2, Ch.4; §12.4).

**Sandwich boss** — this companion's term (⚠ not a verified book phrase) for the middle
manager squeezed between their own manager above and the managers below (Ch.6–7; §12.8;
[managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md)).

**Skip-level** — a meeting between a manager and their reports' reports, run quarterly to
gather unfiltered signal and sanity-check strategy (Ch.7; §9).

**360-degree review** — performance feedback gathered from peers, reports, cross-functional
partners, and self-review, used to fight the manager's blind spots (Ch.4; §6).

**Mentor vs. sponsor** — a mentor gives advice and guidance; a sponsor actively advocates and
opens doors. ⚠ The distinction is standard mentoring literature but was not verified as a
named concept in the book (Ch.2; §14.2).

**Alpha geek** — the engineer who values intelligence above all other traits, believes they
know best, and redoes others' work without warning; a common reason brilliant engineers fail
as managers (Ch.2; §4).

**Brilliant jerk** — the toxic-but-high-output hire the book says never to hire or promote
(Ch.5; §7).

**Delegation matrix** — the book's four-way split of tasks by simplicity and frequency:
simple-and-frequent → delegate; simple-and-infrequent → do yourself; complex-and-infrequent →
do yourself (delegating to train); complex-and-frequent → delegate (Ch.6; §8, §12.6).

**Importance vs. urgency** — the Ch.6 discipline of asking whether a thing matters or merely
screams; the book's version of the Eisenhower matrix (§8;
[time_energy_management_guide.md](time_energy_management_guide.md) §4).

**30/60/90-day plan** — the trust-building and expectation-setting plan for a new reporting
relationship or new role: first 30 days listen, days 30–60 small wins, days 60–90 goals tied
to objectives (Ch.4; §6, §13.3).

**Org design** — structuring teams (size, reporting lines, alignment to product pillars) as a
strategic instrument (Ch.8; §10).

**Career ladder** — the written artifact defining levels, promotion breakpoints, and the
management vs. technical tracks (Ch.9; §11, §13.5).

**DRI** — directly responsible individual: the named owner of an outcome; a repo-standard term
for the accountability practice the book describes as delegation with authority (Ch.6; §12.6).

**Managee** — the person being managed; the book's Ch.1 argument is that being managed well is
itself a skill (Ch.1; §3).

**Performance improvement plan (PIP)** — the documented, time-boxed improvement plan for
underperformers, with measurable milestones; confirmed in the book's index (Ch.4; §6).

**Onboarding ramp** — the structured first 90 days for a new hire: pre-boarding, first-week
pairings, a first-month small win, full productivity by day 90 (Ch.5; §7).

**Blameless postmortem** — an incident review that examines circumstances rather than hunting
for a culprit; a culture artifact (Ch.9; §11).

**Conway's Law** — the observation that an organization's structure shapes the systems its
people build; the book invokes it for cross-functional design (Ch.9; §11).

**Disagree and commit** — argue hard in the room, then deliver the decision and present a
unified front once it is made (Ch.8; §10).

**True north** — the core principles that guide decisions when situations run past any
playbook (Ch.8; §10).

**People-pleaser manager** — the manager who hides problems and overpromises to keep everyone
happy; the Ch.7 dysfunction that skip-levels catch (§9).

**Recency bias** — the review distortion that over-weights recent events; the book's argument
for continuous feedback and a running log (Ch.4; §6).

**Ask the CTO** — Fournier's recurring O'Reilly column; the "sidebars" of CTO-shaped advice the
research notes associate with the book's senior chapters (⚠ exact in-book form unverified;
§14.2).

---

This companion set out to do for its readers what the book does for its own: walk the path
from engineer to senior leader with the map in one hand and the honest warnings in the other
— the alpha geeks and brilliant jerks, the sandwich squeezes and lonely middles, the quotes
we could verify and the ones we could not. The book's own closing note is the most important
fact in this whole file: the path is not one-way, and the point is not the title at the top;
it is doing the work of the rung you are on, in the way the book teaches — deliberately, with
feedback given as a gift, with delegation that grows people, and with culture built on purpose
rather than inherited by accident. Whether you are mentoring an intern at Cymbal Bank or
presenting a regulatory program to a board, that is the manager's path.
