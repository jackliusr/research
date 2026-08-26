# The CFA Program: The Chartered Financial Analyst Designation — A Comprehensive Guide

*A deep dive into the CFA Program — the CFA Institute and the charter (the body, the designation, the value), the three levels (Level I, Level II, Level III), the curriculum (the ten topics and their weights), the exam format (the 2021 computer-based testing transition), the pass rates (2024–2026, including the 2021 collapse and the 2025 rebound), the registration (the 2026 fee overhaul), the prep ecosystem (Kaplan Schweser, Mark Meldrum, AnalystPrep, 300 Hours, and friends), the designation comparison (CFA vs FRM vs CAIA), a worked 300-hour study plan for a working professional, a one-page summary built around "the charter's three gates," and a glossary.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Banking / Asset Management (banking/) — the STUDY-GUIDE pattern (see [technology/ddia_study_companion_guide.md](../technology/ddia_study_companion_guide.md) and [technology/grokking_system_design_companion_guide.md](../technology/grokking_system_design_companion_guide.md) for the precedents)
> **Audience:** Investment-banking technologists, asset-management professionals, candidates, and anyone evaluating the charter
> **Last Updated:** August 2026

**Cross-references (the repo guides this one maps into):** [asset_management_alternatives_guide.md](asset_management_alternatives_guide.md) (the AM/alternatives landscape — the only other CFA mentions in the repo: GIPS, created by CFA Institute), [singapore_private_markets_guide.md](singapore_private_markets_guide.md) and [partners_group_company_guide.md](partners_group_company_guide.md) and [partners_group_competitors_guide.md](partners_group_competitors_guide.md) (the private-markets/AM cluster — overlaps the CFA Level III Private Markets pathway and the alternatives/portfolio-management topics), [risk_management_models_guide.md](risk_management_models_guide.md) (the quant/risk overlap — quantitative methods, derivatives, risk management), [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) (the markets-stack overlap — lightly), [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) (the FRM-angle — lightly), [universal_banking_model_guide.md](universal_banking_model_guide.md) (the banking context — lightly), [../technology/spaced_repetition_apps_guide.md](../technology/spaced_repetition_apps_guide.md) (the study-technique angle — spaced repetition for the 300-hour grind), [../technology/grokking_system_design_companion_guide.md](../technology/grokking_system_design_companion_guide.md) and [../technology/ddia_study_companion_guide.md](../technology/ddia_study_companion_guide.md) and [../technology/system_design_interview_insiders_guide.md](../technology/system_design_interview_insiders_guide.md) (the study-guide pattern this guide follows).

---

## Table of Contents

1. [The Program Overview](#1-the-program-overview)
2. [The Three Levels](#2-the-three-levels)
3. [The Curriculum](#3-the-curriculum)
4. [The Exam Format](#4-the-exam-format)
5. [The Pass Rates](#5-the-pass-rates)
6. [The Registration](#6-the-registration)
7. [The Prep Ecosystem](#7-the-prep-ecosystem)
8. [The Designation Comparison](#8-the-designation-comparison)
9. [The Singapore Context](#9-the-singapore-context)
10. [The Worked Example: A Study Plan](#10-the-worked-example-a-study-plan)
11. [The Summary: The Charter's Three Gates](#11-the-summary-the-charters-three-gates)
12. [Verification and Claims Status](#12-verification-and-claims-status)
13. [Glossary](#13-glossary)

---

**How to read this guide (a note on the pattern):** this guide follows the repo's study-guide convention (see [../technology/ddia_study_companion_guide.md](../technology/ddia_study_companion_guide.md)): verified facts are marked "(verified)", interpreted analysis is marked "(framed)", and anything that could not be checked against a primary source is explicitly flagged — the full ledger is §12. Sections 1–3 answer "what is it," sections 4–7 answer "how it works" (format, rates, money, prep), section 8 answers "why this one vs the others," sections 9–10 answer "what would I actually do," and section 11 is the one-page takeaway. Jump straight to §11 if you want the thesis, then §10 for the plan.

---

## 1. The Program Overview

### 1.1 The Body: CFA Institute (verified)

The **CFA Institute** is the US-based, not-for-profit professional association that owns and administers the CFA Program and awards the CFA charter. It is the world's largest association of investment professionals, and its history is a merger story rather than a single founding:

| Milestone | Year | What happened |
|-----------|------|---------------|
| Financial Analysts Federation (FAF) / National Federation of Financial Analysts Societies (NFFAS) | 1947 | Founded in New York City as a service organization for investment professionals — this is the "founded 1947" lineage most often cited |
| Institute of Chartered Financial Analysts (ICFA) | 1962 | Created by the FAF to administer a professional credential; the earliest charterholders were "grandfathered" in on work experience alone |
| First CFA exam | 1963 | A three-examination series was established (the origin of the three levels), with a practitioner-experience requirement |
| AIMR | 1990 | The FAF and ICFA merged into the **Association for Investment Management and Research (AIMR)** to boost the credential's public profile |
| CFA Institute | 2004 | AIMR renamed itself **CFA Institute** (flag: rename year is widely documented but not re-verified against a primary source for this edition) |

Beyond the CFA Program, CFA Institute is the custodian of several standards and products that matter to the banking/asset-management world this repo covers: the **Code of Ethics and Standards of Professional Conduct** (the ethics backbone of the designation), the **Global Investment Performance Standards (GIPS)** (the required standard for compliant track records — cross-ref [asset_management_alternatives_guide.md](asset_management_alternatives_guide.md)), the **Candidate Body of Knowledge (CBOK)** (the competency map behind the curriculum), the peer-reviewed *Financial Analysts Journal* (published since 1945), the **CIPM** designation (performance measurement), and a family of certificates (Private Equity, Private Markets and Alternative Investments, sustainable investing). The curriculum itself is refreshed every year through a **practice analysis process** that pulls in working industry professionals, so the syllabus tracks practice rather than academia.

**Scale (verified):** the CFA Institute community numbers **more than 200,000 charterholders across 160+ markets worldwide**, supported by local member societies (CFA Society Singapore among them). Exams are delivered **four times a year at more than 400 test locations worldwide**, almost all run by the testing vendor Prometric.

### 1.2 The Designation: The Charter and Its Value (verified)

**CFA** stands for **Chartered Financial Analyst**. It is a postgraduate professional certification — a *charter*, not a degree — awarded by CFA Institute to candidates who satisfy four requirements, all of which must be met:

1. **Pass all three levels** of the CFA Program (Levels I, II, and III), plus complete one **Practical Skills Module (PSM)** per level before the corresponding result is released (PSMs were introduced as a requirement in 2024).
2. **Qualified work experience**: 4,000 hours accumulated over a minimum of three years in an investment-decision role (or adjacent, as determined by CFA Institute) — exams can be taken before this is satisfied, but the charter cannot be awarded without it.
3. **References**: two to three professional references (usually sponsors who are charterholders).
4. **Membership and ethics**: become a CFA Institute member (professional membership ~USD 299/year) and sign the annual commitment to uphold the Code of Ethics and Standards of Professional Conduct. Violations can bring industry sanctions, suspension of the right to use the designation, or revocation of membership.

Entry to the exam itself is easier than the charter: you need a bachelor's degree (or equivalent), or to be a final-year undergraduate whose chosen exam window is within 23 months of graduation — no finance degree required, which is why the program is famously open to career-switchers and technologists.

**Why the charter is valued (verified in substance, framed here):**

- **It is the global benchmark for investment analysis.** CFA is the most recognized credential in portfolio management and equity research worldwide, and the only one of the major finance designations with genuinely global regulatory/legal recognition (securities regulators in markets from Australia to Singapore to Saudi Arabia accept it for licensing exemptions or recognized-role status).
- **It signals generalist depth.** The ten-topic curriculum forces competence across the whole investment stack — accounting, economics, quant, every major asset class, derivatives, and portfolio management — capped by an ethics layer that no other designation weighs as heavily (15–20% at Level I).
- **It is scarce by design.** Three levels, roughly 300 hours of study each, a one-day exam per level, and pass rates in the 40s have kept the charter a genuine differentiator for six decades. Roughly 200,000 people have made it through since 1963 — against literally millions of attempts.
- **It compounds with a finance career.** Charterholders cluster in portfolio management, equity research, investment banking, wealth management, and increasingly in private markets and risk — the areas this repo's banking guides cover (cross-ref [asset_management_alternatives_guide.md](asset_management_alternatives_guide.md), [risk_management_models_guide.md](risk_management_models_guide.md)).

**The honest counterweight:** the charter is a *signal* plus a *base of knowledge* — not a license to practice, not a guarantee of skill, and not a substitute for on-the-job experience. Its ROI depends on the role: it is near-mandatory signaling for buy-side analyst/PM tracks, valuable-but-optional in investment banking and technology, and largely irrelevant to pure software roles — which is exactly why a solution architect considering it should think of the CFA as a *career optionality* purchase (see §8 for the comparison with FRM and CAIA).

### 1.3 The Overview Table

| Aspect | Description |
|--------|-------------|
| **Designation** | Chartered Financial Analyst (CFA) — a postgraduate professional charter, not a degree |
| **Granting body** | CFA Institute (US-based, not-for-profit; lineage: FAF 1947 → ICFA 1962 → AIMR 1990 → CFA Institute 2004) |
| **Program structure** | Three sequential exams — Level I, Level II, Level III — one per year minimum (max two attempts/year) |
| **Study load** | ~300 hours per level (CFA Institute's own figure for successful candidates); a full pass typically takes ~3–4 years part-time (exams alone can be done in 2 years; the 4,000-hour experience requirement sets the real pace) |
| **Curriculum** | Ten topic areas at Level I, distilled into asset-valuation topics at Level II and portfolio-management topics at Level III, refreshed annually |
| **Exam format** | Computer-based (CBT) since 2021; one exam day per level at Prometric centers; MCQ at Level I, vignette item sets at Level II, essay + item sets at Level III |
| **Pass rates** | ~40–50% per sitting by level; 10-year averages: Level I 41%, Level II 45%, Level III 52% |
| **Cost** | USD 1,140–1,490 per level (Level III slightly higher) in 2026; USD 3,520–4,570 for all three levels |
| **Community** | 200,000+ charterholders in 160+ markets |
| **Charter requirements** | Pass 3 levels + PSMs + 4,000 hours/3 years qualified experience + 2–3 references + CFA Institute membership + annual ethics commitment |
| **Value proposition** | Global benchmark credential for investment analysis, portfolio management, and ethics; strongest signaling in buy-side and research careers |

### 1.4 A Short History: From Paper to CBT (verified)

| Era | Milestone |
|-----|-----------|
| 1947–1962 | FAF founded; ICFA created to run a credential; earliest charterholders grandfathered in on experience alone |
| 1963 | First CFA exams administered (US and Canada); the three-level examination series established |
| 1963–2000s | The program goes global — by 2003 fewer than half of candidates were US/Canada-based, with Asia and Europe the growth engines |
| 1990 | AIMR merger; the credential's public profile push begins |
| 2004 | AIMR renamed CFA Institute |
| 2010s | Curriculum modernization: machine learning, big data, and fintech content enters the syllabus |
| **2021** | **Computer-based testing**: February 2021 Level I is the first CBT exam; all levels move to CBT during 2021; Level I question count drops 240 → 180 |
| 2024 | Topic-weight overhaul (first since 2021), Level II moves to 4-question item sets, Practical Skills Modules become mandatory, Level III pathways design introduced |
| 2025 | Level III specialized pathways go live (Portfolio Management / Private Markets / Private Wealth); scale-score MPS reporting (1600/2600/3600) |
| 2026 | Enrollment fee eliminated; registration fees restructured (+USD 150/level) |
| 2027 | Curriculum refresh: Level I renames (Corporate Finance / Equities / Portfolio Construction), 93 → 102 learning modules, AI/LLM quant module (flag: prep-industry-reported) |

---

## 2. The Three Levels

The three levels are deliberately *different exams*, not three volumes of the same exam. The official framing from CFA Institute: **Level I "learn and describe," Level II "analyze and evaluate," Level III "integrate and apply."** Each level builds on the previous one and the format changes materially each time — the single most under-appreciated fact about the program.

### 2.1 Level I: Learn and Describe (verified)

**Level I is a breadth test.** It covers all ten topic areas at an introductory-to-intermediate level — the tools and inputs of investing: quant basics, economics, financial statement analysis, the mechanics of every major asset class, corporate finance, portfolio theory, and ethics. The exam is 180 standalone multiple-choice questions (three options each) in two 2-hour-15-minute sessions on one day. There is no case study, no writing, and no crossover between questions: each question tests one concept. The skill being tested is *knowledge and recall* — "do you know the vocabulary and the formulas of the investment industry."

Candidates come from every background; the exam does not assume a finance degree. Because the questions are standalone and three-option, the pass bar is a matter of breadth coverage rather than depth — which is why the exam-failing failure mode is not difficulty but *volume*: 93 learning modules and ~365 learning outcome statements in the 2026 Level I curriculum is a lot of surface area to cover (see §3).

Level I runs **four times a year (February, May, August, November)** and is the gateway: you must pass it to sit Level II.

### 2.2 Level II: Analyze and Evaluate (verified)

**Level II is a depth-and-application test** built on the Level I toolkit. The exam format shifts completely to **item sets (vignettes)**: each item set is a one-to-two-page case study — a company's financials, a fund's mandate, a market scenario, with footnotes and deliberately irrelevant data — followed by **four multiple-choice questions tied to that case**. The current format: **22 item sets with 88 accompanying questions** (11 sets per session; 20 sets scored, 2 are pretest), two sessions of 2 hours 12 minutes each.

The vignette is the whole game. The case contains more information than you need — distractors test whether you can *identify* what matters before calculating. This is fundamentally different from Level I, where the question tells you exactly what to compute. At Level II you must first decide what to look for, then execute the valuation or analysis. Strong Level I performance does not automatically translate: candidates who answer standalone questions quickly routinely underestimate the reading-and-triage skill Level II demands. Topics concentrate on **asset valuation** — equity and fixed-income valuation, financial statement analysis applied to real statements, derivatives pricing, and the "Corporate Finance / Equities / Fixed Income" cluster — with Ethics still 10–15%.

Level II runs **three times a year (May, August, November)** and is widely reported as the level with the *steepest format adjustment*, if not the lowest pass rate (see §5).

### 2.3 Level III: Integrate and Apply (verified)

**Level III is the portfolio-management and judgment test** — and the only level with written answers. The curriculum drops the asset-class mechanics and centers on **asset allocation, portfolio construction, performance measurement, derivatives and risk management, ethics — plus a specialization.** Since 2025, Level III candidates choose **one of three specialized pathways**, worth 30–35% of the exam:

- **Portfolio Management** — the traditional generalist pathway (deepens asset allocation, portfolio construction, and institutional/private-wealth investing),
- **Private Markets** — private equity, private credit, real estate, infrastructure, and their place in portfolios (the most direct CFA overlap with this repo's private-markets cluster — cross-ref [singapore_private_markets_guide.md](singapore_private_markets_guide.md) and [partners_group_company_guide.md](partners_group_company_guide.md)),
- **Private Wealth** — managing concentrated, multi-generational wealth for individuals and families.

The exam is a **mixed format**: **11 item sets and 11 essay (constructed-response) sets**, split across two 2-hour-12-minute sessions — one session carries 6 item sets + 5 essay sets, the other 5 + 6. The constructed-response questions are the famous "essay" morning-style session: you read a case and write structured answers in a text box — there is no partial credit for knowing a formula you cannot articulate, and marks are allocated per point, not per paragraph, so command words ("calculate," "explain," "justify") matter. Candidates who spend their whole prep on multiple-choice and treat essay practice as secondary consistently underperform.

Level III runs **twice a year (February, August)**. Because the essay session is human-graded (with rubric-supported scoring), results take longer (6–8 weeks vs 5–7 at Levels I–II) and the registration fee is higher (see §6).

### 2.4 The Levels Table

| Level | Focus | Format | Sessions / Duration | Windows per year | Typical first-time candidates |
|-------|-------|--------|---------------------|------------------|-------------------------------|
| **Level I** | Learn and describe — breadth of the ten topics: tools, inputs, asset-class mechanics, ethics | 180 standalone multiple-choice questions (3 options each) | 2 × 135 min (2h15m), 90 questions per session, one day | 4 (Feb, May, Aug, Nov) | Anyone with a bachelor's degree (or final-year student) |
| **Level II** | Analyze and evaluate — apply the Level I toolkit to real cases; asset valuation | 22 item sets (vignettes) × 4 questions = 88 multiple-choice questions; 20 sets scored, 2 pretest | 2 × 132 min (2h12m), 11 sets per session | 3 (May, Aug, Nov) | Passed Level I |
| **Level III** | Integrate and apply — portfolio management, judgment, written articulation; choose a pathway | 11 item sets + 11 constructed-response (essay) sets; sessions split 6+5 or 5+6 | 2 × 132 min (2h12m) | 2 (Feb, Aug) | Passed Level II |
| *All levels* | Ethics is tested at every level (10–20% by level) | Computer-based (CBT) at Prometric centers; one PSM per level required before results | ~4.5 hours content time + tutorial/break/survey | Max 2 attempts per calendar year, not in consecutive windows, not within 6 months | — |

*Sources: CFA Institute exam information and Level II/III outline pages (cfainstitute.org), verified August 2026. The 2021-era Level II format (21 sets × 6 questions) and Level III format (essay + 11 sets × 6) were consolidated into the current 4-question item-set design for 2024.*

### 2.5 What the Questions Actually Look Like (framed walkthroughs)

**Level I — a standalone question tests one idea with no context:** e.g., "A bond with a modified duration of 4.2 and a price of 98.50 would be expected to decline by approximately how much if yields rise by 50 bps?" — three options, ~90 seconds, no case, no trick beyond the math. The exam's difficulty is *breadth*: any of 365 LOS can appear, so coverage beats cleverness.

**Level II — the vignette is the test:** a one-to-two-page case — say, a consumer-goods company's financial statements with footnotes, its recent acquisition, and an analyst's forecast — followed by four questions asking for the FCFF-based valuation, the effect of the acquisition on goodwill, the appropriate discount rate adjustment, and the impact of a footnote disclosure on earnings quality. The case deliberately contains redundant or misleading data; the skill is deciding which three numbers matter. Candidates who underline nothing and compute everything run out of time.

**Level III — the essay tests articulation:** a case about a pension plan's asset allocation followed by: "Calculate the minimum required return (show your work), explain one reason a liability-driven strategy would be preferable here, and justify your choice of hedge ratio." Marks are per point: the calculation must be shown, the explanation must match the *reason* asked for, and the justification must use the case facts. "Know the formula but write vaguely" is how Level III candidates fail the essay session.

### 2.6 Level I Question Distribution by Group (derived from official weights — the exact per-exam split varies within the published ranges)

| Session | Group | Topics | Approx. questions (of 180) |
|---------|-------|--------|----------------------------|
| Session 1 | Group I | Ethics | ~27–36 |
| Session 1 | Group II | Quant + Economics + FSA | ~42–57 |
| Session 2 | Group III | Corporate Issuers + Portfolio Management | ~23–37 |
| Session 2 | Group IV | Equity + Fixed Income + Derivatives + Alternatives | ~60–86 |

---

## 3. The Curriculum

### 3.1 The Ten Topics (verified)

The CFA curriculum is organized around the **Candidate Body of Knowledge (CBOK)** and updated annually. At **Level I**, the 2026 curriculum spans **ten topic areas across 93 learning modules and ~365 learning outcome statements (LOS)** — every exam question maps to a LOS, which is why the LOS list is the de facto syllabus. The ten topics, with their 2026 Level I exam weights (given as *ranges*, which has been CFA Institute's practice since 2020):

| # | Topic | 2026 Level I weight | Notes |
|---|-------|---------------------|-------|
| 1 | **Ethical and Professional Standards** | 15–20% | The heaviest single topic at Level I (≈27–36 of 180 questions). The Code of Ethics + Standards of Professional Conduct, GIPS, and the ethics adjustment (see §5) make this the highest-ROI topic in the program |
| 2 | **Financial Statement Analysis** | 11–14% | IFRS vs US GAAP, the three statements, ratios, quality of earnings. The biggest *skill* topic: it underpins equity and fixed-income valuation at Level II |
| 3 | **Equity Investments** | 11–14% | Markets, indices, industry/company analysis, DCF and multiplier valuation models |
| 4 | **Fixed Income** | 11–14% | Bond mechanics, yield/spread analysis, duration and convexity, securitization |
| 5 | **Portfolio Management** | 8–12% | MPT, the efficient frontier, CAPM, IPS construction, portfolio risk and return |
| 6 | **Alternative Investments** | 7–10% | Hedge funds, private equity, real estate, commodities, infrastructure — fee structures, strategies, due diligence (cross-ref [asset_management_alternatives_guide.md](asset_management_alternatives_guide.md)) |
| 7 | **Quantitative Methods** | 6–9% | Time value of money, probability, hypothesis testing, regression, machine learning/big data survey |
| 8 | **Economics** | 6–9% | Micro (supply/demand, market structure), macro (GDP, business cycles), currency and international economics |
| 9 | **Corporate Issuers** | 6–9% | Capital budgeting, capital structure, dividends, corporate governance, ESG considerations (renamed from "Corporate Finance" in 2024) |
| 10 | **Derivatives** | 5–8% | Forwards, futures, swaps, options; pricing basics (binomial, Black–Scholes conceptually); hedging applications |

**Topic-group structure of the Level I exam (a 2021-CBT-era change):** the 180 questions are grouped — Session 1 holds **Group I (Ethics)** and **Group II (Investment Tools: Quant + Economics + FSA)**; Session 2 holds **Group III (Corporate Issuers + Portfolio Management)** and **Group IV (Investment Assets: Equity + Fixed Income + Derivatives + Alternatives)**. Within a group, question order can be shuffled; across groups, the structure is fixed. Because weights are ranges, the exact question count per topic varies by exam version within the published bands.

### 3.2 The Curriculum Table (topic / weight / notes) — Level I 2026

| Topic | Weight (range) | Avg. weight | Notes |
|-------|----------------|-------------|-------|
| Ethics | 15–20% | ~17.5% | Highest weight; borderline-deciding (ethics adjustment); ethics is the only topic with this special treatment |
| Financial Statement Analysis | 11–14% | ~12.5% | Heaviest "tools" topic; foundational for Level II |
| Equity Investments | 11–14% | ~12.5% | Valuation models tested here and deepened at Level II |
| Fixed Income | 11–14% | ~12.5% | Duration/convexity and spread analysis are Level II must-knows |
| Portfolio Management | 8–12% | ~10% | The Level III topic seeded at Level I |
| Alternative Investments | 7–10% | ~8.5% | Rising weight since 2024 as private markets grew |
| Quantitative Methods | 6–9% | ~7.5% | Small weight, outsized difficulty for non-quant candidates |
| Economics | 6–9% | ~7.5% | Breadth topic; low difficulty, easy marks |
| Corporate Issuers | 6–9% | ~7.5% | Business-school-flavored; fastest to cover |
| Derivatives | 5–8% | ~6.5% | Smallest weight; conceptual pricing, not stochastic calculus |

### 3.3 Beyond Level I: How the Topics Evolve (verified)

- **Level II (2026 official weights)** reorganizes the same content into valuation-focused topics: Quantitative Methods 5–10%, Economics 5–10%, Financial Statement Analysis 10–15%, Corporate Finance 5–10%, Equities 10–15%, Fixed Income 10–15%, Derivatives and Risk Management 5–10%, Alternative Investments 5–10%, Portfolio Construction 10–15%, Ethics 10–15%. Note the topic *names* at Level II already use the "2027" vocabulary (Corporate Finance, Equities, Portfolio Construction, Derivatives and Risk Management).
- **Level III (2026 official weights)** is a different animal — the core is five portfolio-level topics plus the pathway: **Asset Allocation 15–20%, Portfolio Construction 15–20%, Performance Measurement 5–10%, Derivatives and Risk Management 10–15%, Ethics 10–15%, Pathways 30–35%**. Financial statement analysis and asset-class mechanics are effectively retired; everything is tested through the portfolio lens.

### 3.4 The Level II and Level III Weight Tables (2026, official)

**Level II (2026 official weights):**

| Topic | Weight |
|-------|--------|
| Quantitative Methods | 5–10% |
| Economics | 5–10% |
| Financial Statement Analysis | 10–15% |
| Corporate Finance | 5–10% |
| Equities | 10–15% |
| Fixed Income | 10–15% |
| Derivatives and Risk Management | 5–10% |
| Alternative Investments | 5–10% |
| Portfolio Construction | 10–15% |
| Ethical and Professional Standards | 10–15% |

**Level III (2026 official weights):**

| Topic | Weight |
|-------|--------|
| Asset Allocation | 15–20% |
| Portfolio Construction | 15–20% |
| Performance Measurement | 5–10% |
| Derivatives and Risk Management | 10–15% |
| Ethical and Professional Standards | 10–15% |
| Pathways (Portfolio Management / Private Markets / Private Wealth) | 30–35% |

**How the weights move across levels (framed):** the ten Level I topics are roughly equalized at Level II with a valuation tilt (FSA, Equities, Fixed Income, Portfolio Construction each at 10–15%); by Level III the curriculum has *compressed into a portfolio-shaped syllabus* — the asset classes appear only inside Asset Allocation, Portfolio Construction, and the pathway, and the exam is dominated by allocation, construction, and the chosen pathway (together ~65–75% of the paper). A candidate should read the Level III weight table as a statement about the profession: by the end, the CFA is an *allocation and judgment* credential, not an asset-class textbook.

### 3.5 Curriculum Changes: 2024, 2026, and 2027 (verified with flags)

- **2024 (verified):** the biggest curriculum/format overhaul since CBT — topic weights moved for the first time since 2021, "Corporate Finance" became "Corporate Issuers" and "Portfolio Management" became "Portfolio Management and Wealth Planning" (later "Portfolio Management"), Level II moved to 4-question item sets, and Level III introduced the pathways design (pathways themselves went live in 2025). The Practical Skills Module requirement also landed in 2024.
- **2026 (verified):** Level I topic weights unchanged from 2025; Level II and Level III outlines refreshed; the February 2026 window was the first under the 2026 curriculum.
- **2027 (verified via multiple prep-industry sources; flag: not independently confirmed against the CFA Institute outline PDFs in this edition):** three Level I topics are renamed to match the Level II vocabulary — Corporate Issuers → **Corporate Finance**, Equity Investments → **Equities**, Portfolio Management → **Portfolio Construction** — the curriculum grows from 93 to **102 learning modules** (Equities 8→12, Ethics 5→10), and Quantitative Methods gains a new module on **financial data science** (big data, machine learning, AI, and large language models in investment management). The 2027 curriculum applies from the February 2027 window; **November 2026 is the last window on the 2026 curriculum.** A candidate starting fresh in late 2026 should study the 2027 materials.

---

## 4. The Exam Format

### 4.1 The Computer-Based Transition (verified)

The CFA exam was **paper-based for 58 years** — booklets shipped worldwide, answer sheets flown in for manual grading, ethics scripts reviewed by hand. That ended in **2021**: the **February 2021 Level I exam was the first CFA exam delivered on computers**, and all levels completed the move to **computer-based testing (CBT)** during 2021 (Levels II and III from September 2021). The transition changed the program structurally:

- **Question counts dropped.** Level I went from 240 questions (2 × 120) to **180 questions (2 × 90)**, and Level II from 21 sets × 6 questions (126) to the current **22 sets × 4 (88)** design in 2024.
- **Windows replaced a single annual date.** Level I used to be one paper sitting a year; since CBT it runs four times a year at Prometric centers worldwide, and Level II/III gained multiple windows too.
- **The exam became a same-day, center-based CBT experience**: identity verification, palm-scan biometrics, lockers for everything but a passport, an approved calculator, and a clear bottle of water; on-screen questions, provided scratch paper, a software tutorial, an optional break between sessions, and an end-of-exam survey. Total content time is ~4.5 hours per level; Level I sessions are 135 minutes each, Level II and III sessions are 132 minutes each.
- **The exam is not adaptive** — every candidate in a window sees the same set of questions (in the same session structure), and results are set against a **minimum passing score (MPS)** that is established after each window via expert judgment of question difficulty plus statistical equating.

**Hardware rules that catch candidates:** only two calculators are permitted — the **Texas Instruments BA II Plus** (including Professional) and the **HP 12C** (including Platinum and Anniversary editions). No phones, smartwatches, or internet-connected devices in the testing room; arrive at least 30 minutes early.

### 4.2 The Format Table (verified)

| Aspect | Level I | Level II | Level III |
|--------|---------|----------|-----------|
| Question format | 180 standalone MCQs (3 options) | 88 MCQs in 22 item sets (vignettes), 4 per set | 11 item sets (MCQ) + 11 constructed-response/essay sets |
| Scored content | 180 (all scored) | 20 of 22 sets scored (2 pretest) | All item/essay sets scored (essay human-graded by rubric) |
| Sessions | 2 × 135 min (2h15m) | 2 × 132 min (2h12m) | 2 × 132 min (2h12m); sets split 6+5 or 5+6 per session |
| Total content time | 4h30m | 4h30m | 4h30m (+ tutorial, break, survey) |
| Skill tested | Knowledge and recall | Application and case triage | Synthesis, judgment, written articulation |
| Pathway choice | — | — | Portfolio Management / Private Markets / Private Wealth (30–35%) |
| Windows (2026) | 4 (Feb, May, Aug, Nov) | 3 (May, Aug, Nov) | 2 (Feb, Aug) |
| Results | 5–7 weeks | 5–7 weeks | 6–8 weeks |
| Special requirement | One PSM before results | One PSM before results | One PSM before results |
| Delivery | CBT at Prometric centers, all levels, since 2021 | | |

**Format traps, by level:** Level I — none structural; the trap is coverage (93 modules of surface area). Level II — the vignette's deliberately irrelevant data; the trap is reading for the question. Level III — the essay; the trap is treating constructed response as an afterthought and losing points on command words, over-writing, and missed sub-parts.

### 4.3 The Exam Day, Hour by Hour (verified)

The CBT day is standardized across levels and centers (Prometric):

| Time block | What happens |
|------------|--------------|
| ~30+ min before start | Arrive; check-in: valid international travel passport, identity verification, palm-scan biometrics, locker storage for everything not permitted |
| Tutorial | On-screen software tutorial (not scored) |
| Session 1 | Level I: 90 MCQs / 135 min. Level II: 11 item sets / 132 min. Level III: 6 item sets + 5 essay sets (or the mirror split) / 132 min |
| Optional break | Short break between sessions; water/snacks accessible (not at the desk) |
| Session 2 | Level I: 90 MCQs / 135 min. Level II: 11 item sets / 132 min. Level III: the other 5 + 6 split / 132 min |
| Survey + exit | End-of-exam survey; scratch paper collected |

**The desk:** the computer, provided scratch paper and pencil, your approved calculator, and a clear bottle of water (no labels). **The locker:** phone, smartwatch, wallet, keys, food, outerwear — any internet-connected device in the room is a disqualification risk. **The calculator:** TI BA II Plus (incl. Professional) or HP 12C (incl. Platinum/Anniversary); bring spare batteries. **Afterward:** Level I/II results in 5–7 weeks, Level III in 6–8 weeks, with a pass/fail and a score report (Level I/II reports show topic-by-topic performance; Level III candidates only get a report on failure).

---

## 5. The Pass Rates

### 5.1 The 2025 Rates (verified via CFA Institute results reports as aggregated by the prep industry; flag: per-window rates are CFA Institute-published and the table below is cross-checked across trackers)

| Window | Level I | Level II | Level III |
|--------|---------|----------|-----------|
| Feb 2025 | 45% | — | 49% |
| May 2025 | 45% | 54% | — |
| Aug 2025 | 43% | 44% | 50% |
| Nov 2025 | 43% | 42% | — |
| **2025 average** | **~44%** | **~47%** | **~50%** |
| *For context — 2024* | *44 / 46 / 44 / 43* | *59 / 47 / 39* | *49 / 48* |
| *For context — 2026 so far* | *45 (Feb) / 39 (May)* | *43 (May)* | *50 (Feb)* |
| 10-year average (CFA Institute) | 41% | 45% | 52% |

### 5.2 How to Read the Rates (verified)

- **The 2021 collapse was a real, documented event:** Level I pass rates hit **22–27%** in 2021 (July 2021: 22%, the all-time low; May 2021: 25%; August 2021: 26%; November 2021: 27%), and Level II hit **29%** (August 2021). The consensus explanations: pandemic-disrupted study conditions, the disruption of the paper→CBT transition itself, and a record influx of under-prepared first-time candidates. The rates rebounded through 2022–2024 and have normalized in the low-to-mid 40s.
- **The 2025 story is a rebound and a new results system.** 2025 pass rates were the healthiest since the CBT transition, with May 2025 Level II at **54%** — the highest Level II rate in years. 2025 also introduced **scale-score MPS reporting**: results are now reported against a scale-score MPS of **1600 (Level I), 2600 (Level II), 3600 (Level III)** instead of the old percentage-style bands; the underlying MPS is still set per window and is never published as a percentage (industry estimates of the effective bar: roughly 60–70% of correct answers — flag: educated guess, not official).
- **The ethics adjustment is real and unique:** since 1996, candidates whose overall score lands close to the MPS have their result decided by their Ethics score — strong ethics can flip a borderline FAIL to PASS and vice versa. With Ethics at 15–20% of Level I and 10–15% at Levels II–III, it is the only topic that is simultaneously the biggest, the most testable, and the tie-breaker.
- **Pass rate ≠ difficulty ranking.** Level III has the *highest* pass rate and is generally considered the hardest exam for first-time candidates — but only candidates who have already cleared Level I and II sit it, so the pool is self-selected and experienced. Level II's rate (45% 10-yr avg) reflects the brutal format shift; Level I's (41%) reflects the unfiltered pool.
- **The real-world completion rate is far lower than any single-sitting rate**: the "sitting → charter in four years" funnel loses most candidates between Level I and the experience requirement. CFA Institute's own data and industry studies consistently show that *attempts* vastly exceed *charters* — which is precisely the scarcity the designation sells.

### 5.3 The Window-by-Window Record, 2021–2026 (verified, cross-checked across trackers)

| Window | Level I | Level II | Level III |
|--------|---------|----------|-----------|
| Feb 2021 (first CBT Level I) | 44% | — | — |
| May 2021 | 25% | 40% | 42% |
| Jul 2021 | 22% | — | — |
| Aug 2021 | 26% | 29% | 39% |
| Nov 2021 | 27% | 46% | 43% |
| Feb 2022 | 36% | 44% | — |
| May 2022 | 38% | — | 49% |
| Aug 2022 | 37% | 40% | 48% |
| Nov 2022 | 36% | 44% | — |
| Feb 2023 | 38% | — | 48% |
| May 2023 | 39% | 52% | — |
| Aug 2023 | 37% | 44% | 47% |
| Nov 2023 | 35% | 44% | — |
| Feb 2024 | 44% | — | 49% |
| May 2024 | 46% | 59% | — |
| Aug 2024 | 44% | 47% | 48% |
| Nov 2024 | 43% | 39% | — |
| Feb 2025 | 45% | — | 49% |
| May 2025 | 45% | 54% | — |
| Aug 2025 | 43% | 44% | 50% |
| Nov 2025 | 43% | 42% | — |
| Feb 2026 | 45% | — | 50% |
| May 2026 | 39% | 43% | — |

**Reads:** the 2021 trough (22–29%) → 2022–2023 recovery (mid-30s to low-40s) → 2024–2025 normalization (43–59%) — with Level II's 54–59% spikes in May 2024/2025 as the standout; the "post-CBT equilibrium" since 2024 sits around 39–50% depending on level and window. Also note the window-schedule evolution embedded in the dashes: Level II gained windows over time (Feb 2022, then May/Aug/Nov from 2022), and Level III has been Feb/Aug since 2023.

---

## 6. The Registration

### 6.1 The 2026 Fee Overhaul (verified)

CFA Institute **eliminated the one-time USD 350 enrollment fee** starting with exams from February 2026 onward (the change was announced in the "Pricing changes for CFA Program" release of 18 December 2024, and enrollment for those windows opened 29 April 2025). To offset it, **registration fees rose by USD 150 per level**. The net effect is uneven — a point most coverage misses:

- **Brand-new candidates save money:** in 2025 a first registration cost USD 990 (early) or 1,290 (standard) *plus* the one-time 350 = **1,340 / 1,640**. In 2026 the same first registration costs **1,140 / 1,490** — a saving of ~USD 150–200 on your very first exam.
- **Returning candidates pay more:** the enrollment fee was always charged once per candidate, so a candidate advancing Level I → II in 2026 simply absorbs the USD 150 registration increase with no offsetting saving.
- **Level III costs more than I and II** (USD 1,240–1,590) reflecting the human-marked essay session.

### 6.2 The Registration Table (verified, USD, 2026 exams)

| Cost item | 2025 (for contrast) | 2026 | Notes |
|-----------|---------------------|------|-------|
| One-time enrollment fee | 350 (new candidates) | **Eliminated** | First charged 2025 and earlier; gone from Feb 2026 windows |
| Level I registration — early / standard | 990 / 1,290 | **1,140 / 1,490** | Early deadline ~3 months before the window; early saves 350 |
| Level II registration — early / standard | 990 / 1,290 | **1,140 / 1,490** | Same schedule as Level I |
| Level III registration — early / standard | 1,040 / 1,340 (approx., flag) | **1,240 / 1,590** | Higher than I/II due to essay grading |
| Rescheduling (within window) | 250 | **250** | Closes ~1 week before exam dates |
| Full program (all three levels) | ~4,020–4,610 incl. enrollment | **3,520–4,570** (official CFA Institute FAQ figure) | All-early = 3,520; all-standard = 4,570 |
| Professional membership (post-charter) | 299/year | 299/year | Separate from exam fees |
| Practical Skills Module | Included | Included | Completion required before results |
| Official curriculum + Learning Ecosystem + practice | Included in registration | Included | e-books; print optional extra |
| Taxes | Excluded | Excluded | Local taxes (e.g., GST) added at checkout |

**Scholarships (verified to exist; amounts flagged):** the **Access Scholarship** (financial need; reduced program fee cited at roughly USD 400 for awardees), **Student Scholarship** (full-time students at affiliated universities), **Professor Scholarship** (qualifying faculty), and **Regulator Scholarship** (employer/regulator nomination only). Since the enrollment fee is now zero, the value of every scholarship is now purely the registration discount.

**Registration mechanics that catch people (verified):**
- **Registration ≠ scheduling.** You register on cfainstitute.org, then separately book your exam appointment (date + test center) before the scheduling deadline; seats are first-come, first-served — register and schedule early.
- **Attempt limits:** max two attempts per calendar year, and not in consecutive windows or windows within six months of each other (a February attempt makes August — not May — the next possible window).
- **Fees are nonrefundable, non-deferrable, and non-transferable** for missed appointments; missing the exam forfeits the fee unless you reschedule within the window (USD 250).
- **All deadlines are 11:59 PM ET**, and fees are paid in USD (credit cards incl. Visa/MasterCard/Amex, Alipay, or invoice).
- **Budget beyond CFA Institute:** third-party prep (USD 300–800/level for Notes/QBank bundles — see §7), the approved calculator (~USD 30–60 for a BA II Plus), taxes, and travel to the nearest test center.

### 6.3 The Three-Year Cost Journey (worked, USD)

For the candidate of §10 (all-early registration, 2026–2029 windows, Singapore):

| Year | Item | Cost (USD) | Notes |
|------|------|-----------|-------|
| Y1 (2026–27) | Level I registration (early, Aug 2027 window) | 1,140 | No enrollment fee under the 2026 rules |
| Y1 | Prep (one primary source + QBank) | ~400–700 | Schweser/Meldrum-class package; official QBank included |
| Y1 | Calculator (BA II Plus) + batteries | ~50 | One-time |
| Y2 (2028) | Level II registration (early, May 2028) | 1,140 | + prep ~400–700 |
| Y3 (2029) | Level III registration (early, Feb 2029) | 1,240 | + prep ~400–700 |
| Every year | Local taxes (e.g., SG GST at checkout) | ~5–9% of fees | Flag: exact rate/timing per CFA Institute's tax policy |
| Contingency | Rescheduling (250) or one retake (1,140–1,490) | 250–1,490 | Budget for it; the median candidate fails at least once across three levels |
| **Total, happy path** | | **~4,500–5,300** | Fees 3,520 + prep 1,200–2,100 + calculator + taxes |
| **Total, with one retake** | | **~6,000–7,000** | The honest planning number |

The structural point: the *fees* (USD 3,520–4,570) are only ~60–70% of the real cash cost, and the retake probability is the biggest single swing factor — which is why registering early, studying to a mocks-based standard, and choosing windows with a retake option (e.g., August → November at Level I) are themselves money-management decisions.

---

## 7. The Prep Ecosystem

### 7.1 The Providers (verified — lineup and positioning cross-checked across the 2026 prep-comparison press)

CFA Institute gives every registered candidate the **official curriculum e-books, the Learning Ecosystem (the digital study platform), a practice-question bank, and mock exams** — the baseline. The commercial ecosystem sells *compression and practice*: condensed notes, video lectures, larger question banks, mocks, and study tools. **Nothing third-party is officially endorsed** — CFA Institute runs an endorsement program (CFA Institute Prep Provider) but candidates are free to mix and match. The big names, as of 2026:

- **Kaplan Schweser** — the incumbent market leader; famous for SchweserNotes (condensed curriculum), the QBank, Secret Sauce final-review notes, and mock exams; the reference point for every other provider's pricing; widely regarded as the safest choice for a structured candidate. (Cross-check its CAIA/FRM arms too — Schweser covers all three designations.)
- **Mark Meldrum** — the video-first challenger; a CFA-charterholder lecturer whose comprehensive video courses (~USD 399/level in the 2026 comparisons) are the value benchmark; beloved for teaching *understanding* rather than memorization; supplements with its own QBank and mocks.
- **AnalystPrep** — the low-entry-price option (~USD 349/level in the 2026 comparisons); strong question bank and study notes; popular with cost-conscious candidates.
- **UWorld** — the practice-question specialist; exam-level questions with detailed explanations and analytics; often rated best-in-class on QBank quality.
- **Wiley (Efficient Learning)** — CFA Institute's former official-prep-partner lineage; known for the "11th Hour" final-review guide and adaptive question practice.
- **300 Hours** — not a course but the industry's most-used free resource hub: pass-rate trackers, fee and format explainers, study-plan templates, and honest provider reviews; where most candidates start their research.
- **Also in the market:** Bloomberg Exam Prep, IFT (free video lectures), FinQuiz, Salt Solutions, Chalk & Board (tutoring), and regional classroom brands (e.g., Fintree/QuintEdge in India, IMS Proschool) that add live cohorts.

### 7.2 The Prep Table (provider / offering / notes)

| Provider | Offering | Notes |
|----------|----------|-------|
| **CFA Institute** (official) | Curriculum e-books, Learning Ecosystem, QBank, mock exams, PSMs | Included in registration; the only source that is definitionally aligned with the exam — the baseline every plan starts from |
| **Kaplan Schweser** | SchweserNotes, QBank, Secret Sauce, mocks, Premium packages | Market leader; ~USD 700–1,300/level depending on package (flag: indicative, varies by year/promotion); best for candidates who want a structured syllabus substitute |
| **Mark Meldrum** | Video lectures (~250+ hours), QBank, mocks | ~USD 399/level (2026 comparisons); the value-for-understanding pick; pairs well with the official QBank |
| **AnalystPrep** | Study notes, QBank, mocks | ~USD 349/level entry (2026 comparisons); the budget pick |
| **UWorld** | High-fidelity QBank with explanations/analytics | QBank-quality leader; often used to *supplement* rather than replace a course |
| **Wiley (Efficient Learning)** | 11th Hour review, adaptive question practice | Strong final-month review brand; lighter full-course footprint in 2026 |
| **300 Hours** | Free guides, pass-rate tracker, plan templates, reviews | Free; the research layer before you spend |
| **Bloomberg / IFT / FinQuiz / Salt Solutions / Chalk & Board** | Courses (Bloomberg), free videos (IFT), QBanks (FinQuiz), study tools (Salt), tutoring (Chalk & Board) | Niche options for specific needs (Bloomberg terminal users, live tutoring, alternate QBanks) |
| **Regional classroom brands** (e.g., IMS Proschool, Fintree, QuintEdge) | Live/instructor-led cohorts, study groups | Popular in India/SG markets; add accountability and exam-window discipline |

### 7.3 How the Ecosystem Actually Works (framed)

The evidence-consistent prep recipe across the industry press: **(1) one primary learning source** (official curriculum for the ambitious, SchweserNotes or Meldrum videos for compression), **(2) massive question volume** (2,000+ QBank questions per level; practice is the single strongest predictor of pass rates in every provider study — flag: provider studies are self-interested but the direction of the finding is consistent), **(3) timed mock exams** (2–3 full mocks in the final month, at the same time of day as your real session), and **(4) spaced repetition** for formula recall (cross-ref [../technology/spaced_repetition_apps_guide.md](../technology/spaced_repetition_apps_guide.md) — the SRS machinery transfers directly to CFA formulas, ratios, and ethics rules). Candidates who skip (2) and (3) — "I read everything" — are the archetypal 2021-era failure story.

### 7.4 A Prep Strategy Matrix (framed, not a recommendation)

| Candidate profile | Primary source | Practice layer | Typical spend |
|-------------------|----------------|----------------|---------------|
| Working professional, 12 h/week (the §10 case) | SchweserNotes or Mark Meldrum videos | Official QBank + one third-party QBank (UWorld/AnalystPrep) + 3 mocks | USD 400–800/level |
| Full-time student | Official curriculum (time-rich) | Official QBank + AnalystPrep | USD 350–600/level |
| Career switcher with no finance background | Meldrum videos (concept-first) + Schweser Notes | Heavy QBank (2,500+ Qs) + 3–4 mocks | USD 500–900/level |
| Strong quant / finance background | SchweserNotes only (compression) | UWorld QBank + mocks | USD 400–700/level |
| Budget-constrained | IFT free videos + official e-books | Official QBank + free mocks (CFA Institute practice pack) | USD 0–350/level |
| Repeated failure (>1 attempt) | Full course with tutoring layer (Chalk & Board / live cohort) | Diagnostic QBank + timed mocks weekly | USD 1,000+ |

**The free-resource layer everyone should use regardless:** CFA Institute's own practice questions and one free mock (included), 300 Hours' pass-rate tracker and study-plan templates, r/CFA and the official candidate forums for window-specific intel, and the CFA Institute Learning Ecosystem's progress analytics. The ecosystem's real cost curve is not the price of the course — it is the *hour-for-dollar* efficiency of compressed notes vs the official curriculum, and the accountability of live cohorts vs solo study; both are personal-fit decisions, not quality rankings.

---

## 8. The Designation Comparison

### 8.1 CFA vs FRM vs CAIA (verified in substance; figures flagged where approximate)

For a finance-adjacent professional deciding where to spend study capital, the three credentials that dominate the comparison conversation are the **CFA** (investment analysis), the **FRM** (risk management), and the **CAIA** (alternative investments). They are complements more than competitors: CFA is the generalist's charter, FRM is the risk specialist's, CAIA is the alternatives specialist's — and the industry's standard advice is to pick by *career target*, not by difficulty.

- **CFA — Chartered Financial Analyst** (CFA Institute): three levels, ~300 hours per level, generalist coverage of the entire investment stack with an ethics core. The strongest global recognition and the broadest career optionality (portfolio management, research, wealth, increasingly private markets). The long-haul commitment (~3–4 years) and the highest total cost of the three (USD 3,520–4,570 in fees alone).
- **FRM — Financial Risk Manager** (GARP — the Global Association of Risk Professionals): two parts (Part I: foundations of risk, quant, financial markets; Part II: market, credit, operational/liquidity risk, and current issues), both sittable in a single year (Part I in May, Part II in November, or both in November). Deep, technical, quant-flavored risk content — the direct credential for market/credit/operational risk desks, treasury, and risk-technology roles (cross-ref [risk_management_models_guide.md](risk_management_models_guide.md) and [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)). Faster and cheaper than the CFA (flag: ~USD 1,500–2,000 all-in, and ~90,000+ FRMs worldwide — approximate figures to verify at garp.org).
- **CAIA — Chartered Alternative Investment Analyst** (CAIA Association, founded 2002): two levels, ~200–300 hours per level, entirely about alternatives — private equity, private credit, hedge funds, real estate, commodities, infrastructure, and the due-diligence/operations layer around them. The natural add-on for anyone already working in or targeting private markets and fund structures (cross-ref [singapore_private_markets_guide.md](singapore_private_markets_guide.md) — the overlap with the CFA Level III Private Markets pathway is substantial). Cheaper and faster than the CFA (flag: ~USD 3,000–3,500 all-in; ~13,000+ members — approximate).

### 8.2 The Comparison Table

| Dimension | CFA | FRM | CAIA |
|-----------|-----|-----|------|
| **Name** | Chartered Financial Analyst | Financial Risk Manager | Chartered Alternative Investment Analyst |
| **Granting body** | CFA Institute (1947 lineage) | GARP (founded 1997) | CAIA Association (founded 2002) |
| **Focus** | Generalist investment analysis & portfolio management | Market, credit, operational, liquidity risk | Alternative investments (PE, HF, real estate, commodities) |
| **Structure** | 3 levels (I, II, III), one level per window, sequential | 2 parts (I & II), can complete in ~1 year | 2 levels (I & II), ~1–1.5 years typical |
| **Study load** | ~300 hours per level (official) | ~200–300 hours per part (flag: approximate) | ~200–300 hours per level (flag: approximate) |
| **Exam format** | CBT since 2021: MCQ → vignettes → essay+item sets | CBT, MCQ; Parts I & II both 4-hour-ish paperless exams | CBT, MCQ; two sessions per level |
| **Approx. cost, all-in** | USD 3,520–4,570 (verified, 2026 fees) | ~USD 1,500–2,000 (flag: verify at garp.org) | ~USD 3,000–3,500 (flag: verify at caia.org) |
| **Typical timeline** | 3–4 years part-time (+ 3 yrs experience for charter) | 1 year | 1–1.5 years |
| **Community size** | 200,000+ charterholders (verified) | ~90,000+ FRMs (flag: approximate) | ~13,000+ members (flag: approximate) |
| **Best for** | Portfolio management, research, wealth, generalist AM | Risk desks, treasury, risk technology, quant risk | Private markets, hedge funds, fund structuring, real estate |
| **Ethics emphasis** | 10–20% per level + ethics adjustment + annual commitment | Code of conduct; lighter exam weight | Code of ethics; lighter exam weight |
| **Typical pairing advice** | The core credential | CFA + FRM for risk-heavy AM roles | CFA + CAIA for alternatives/private-markets roles |

**The takeaway for this repo's audience** (banking technologists and AM professionals): the CFA is the one that buys *generalist optionality* across the whole investment industry; FRM is the one that speaks the language of the risk functions you already build systems for; CAIA is the one that signals depth in the private-markets cluster this repo tracks most closely. They stack — CFA + FRM and CFA + CAIA are both common, deliberate combinations. The wrong order is picking FRM or CAIA *instead of* CFA when the career target is genuinely generalist investment management.

### 8.3 The Other Comparison: CFA vs MBA (framed)

The other credential the CFA is constantly compared with is the MBA — and the comparison is instructive because they solve different problems:

| Dimension | CFA | MBA |
|-----------|-----|-----|
| What it proves | Investment analysis, valuation, ethics, portfolio management | General management, leadership, networks, career pivot |
| Format | Self-paced exam program (3 exams) | 1–2 year full-time (or part-time/EMBA) residency program |
| Cost | USD 3,520–4,570 in fees + prep (~5–7k all-in) | USD 100k+ at top schools (tens of thousands in Asia) |
| Time | ~300 h/level, parallel to work | 1–2 years out of the workforce (or long part-time) |
| Signaling | Technical competence + ethics in investing | Broader managerial/leadership signal + alumni network |
| Best for | Buy-side, research, wealth, AM careers | Pivots into management consulting, general management, IB associate tracks, entrepreneurship |

The standard industry view: **the CFA is the cheaper, more technical, more investment-specific signal; the MBA is the network-and-pivot purchase.** They are complements for AM leadership tracks (CFA for credibility, MBA for the move into management), and the choice for a technologist is usually the CFA — because it can be earned while working and it certifies exactly the domain language the front-office-adjacent roles demand (see §9).

---

## 9. The Singapore Context

### 9.1 The Market That Pays for the Charter

Singapore is one of the densest charterholder markets in Asia and one of the places where the CFA charter's signaling value is highest — for three structural reasons:

- **The asset-management hub.** Singapore is a top-tier global AM center (see [asset_management_alternatives_guide.md](asset_management_alternatives_guide.md) and [singapore_private_markets_guide.md](singapore_private_markets_guide.md) for the AUM and growth numbers), and buy-side hiring is charter-driven: portfolio management, research, and client-facing AM roles in Singapore routinely list "CFA (or progress toward)" as a requirement or strong preference.
- **The private-markets and wealth boom.** The VCC (variable capital company) regime, family-office incentives, and the private-credit/private-equity build-out have made alternatives and private wealth the growth hiring lanes — exactly the CFA Level III Private Markets and Private Wealth pathways (cross-ref [singapore_private_markets_guide.md](singapore_private_markets_guide.md), [partners_group_company_guide.md](partners_group_company_guide.md), [partners_group_competitors_guide.md](partners_group_competitors_guide.md)).
- **Regulatory recognition.** The Monetary Authority of Singapore (MAS) accepts the CFA charter toward licensing requirements for securities and fund-management roles (flag: the exact CMFAS module exemptions change periodically — confirm against the current MAS handbook before relying on any specific exemption).

**Society scale for orientation:** CFA Institute's own community pages cite ~200,000 charterholders across 160+ markets; for a sense of local density, the region's largest societies run in the thousands of members (CFA Society Hong Kong reported ~5,700 members in mid-2026 — flag: CFA Society Singapore's exact headcount was not re-verified for this edition, but it is consistently ranked among CFA Institute's largest societies globally).

### 9.2 What the Local Ecosystem Means for a Candidate

- **Local test centers:** Prometric operates testing centers in Singapore, so a Singapore-based candidate sits the same CBT exam without travel — but seats are first-come, first-served, so schedule the appointment as soon as scheduling opens (flag: confirm current center availability at scheduling time).
- **Employer sponsorship:** Singapore banks and AM firms routinely sponsor registration and prep for staff in front-office-adjacent roles; the Cymbal Bank-type technology functions this repo covers often qualify if the role touches investment or risk platforms — worth asking before paying (see §6 for the costs).
- **The technologist's angle:** as Singapore's banking-technology market consolidates around front-office alignment (cross-ref [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md)), the ability to speak investment language fluently is a real differentiator for solution architects — the charter is the standard way to *prove* that language, which is the core argument for this repo's audience.
- **The network:** CFA Society Singapore runs study groups, mocks, and the annual charter-award ceremony; for a working candidate the local society is both a prep resource and a networking shortcut into the buy-side community.

---

## 10. The Worked Example: A Study Plan

### 10.1 The Scenario (the candidate, in a familiar context)

**The candidate:** a Singapore-based solution architect at a European investment bank (Cymbal Bank's world, in other words — the audience of this repo). Mid-career, strong on systems (risk platforms, market-data plumbing, integration architecture — cross-ref [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) and [risk_management_models_guide.md](risk_management_models_guide.md)), financially literate but not a finance graduate. She has decided to sit **CFA Level I** — not to become a portfolio manager, but to buy *credibility and optionality* in a bank whose front office, private-markets franchise (cross-ref [singapore_private_markets_guide.md](singapore_private_markets_guide.md)), and asset-management arm keep hiring people who can speak the investment language. She is comfortable with math (good for Quant), has never studied accounting formally (risk for FSA), and has 6–7 hours on weekdays plus 5–6 on weekends.

**The familiar 300-hour context (verified):** CFA Institute's own guidance — repeated across the official program pages and Level II/III outlines — is that **successful candidates report studying on average "over 300 hours" per level**. The whole plan below is built to land in that band: **312 hours over 26 weeks (12 hours/week)**, a deliberately sustainable part-time load, with the two live variables being *question volume* and *consistency*, not raw hours.

**The window (verified against the 2026–27 calendar):** target **August 2027** (Level I window 15–22 August 2027; 2027 curriculum applies from the February 2027 window). Registration opens ~November 2026 — she registers **early (USD 1,140) within days of opening**, books a Prometric seat in Singapore at scheduling time, and buys the BA II Plus the same week. Exam day: a Saturday slot; results ~5–7 weeks later, well before the November 2027 window if a retake were ever needed (a February 2027 attempt would have forced the six-month rule to skip May anyway — the August window leaves the November retake option open).

### 10.2 The Schedule Design (26 weeks ≈ 312 hours)

The design logic: **weight-driven ordering** (learn the big topics first so they get the most repetition), **difficulty coupling** (pair hard Quant/FSA weeks with lighter topics), **ethics woven in from week 1** (it is 15–20% of the exam and the borderline tie-breaker — it cannot be a cram), and **the final six weeks are pure practice** (question volume + mocks + SRS review), because the exam tests retrieval, not recognition.

| Phase | Weeks | Hours | Content | Milestone |
|-------|-------|-------|---------|-----------|
| **0. Setup** | Before W1 | 4 | Register early, buy BA II Plus, choose primary source (SchweserNotes or Mark Meldrum videos + official QBank), set up an SRS deck (cross-ref [../technology/spaced_repetition_apps_guide.md](../technology/spaced_repetition_apps_guide.md)) | Registered at early fee; seat booked |
| **1. Tools** | W1–2 | 24 | Quantitative Methods (TVM, probability, hypothesis testing, regression) — the math foundation everything else leans on | 100% QM readings + 150 QBank Qs |
| **2. The Big Skill** | W3–5 | 36 | Financial Statement Analysis (the three statements, ratios, IFRS vs US GAAP, quality of earnings) — the heaviest non-ethics topic and the Level II foundation | FSA readings done; 200 Qs |
| **3. Markets** | W6 | 12 | Economics (micro + macro + currency basics) — fast, low-difficulty marks | Eco readings + 100 Qs |
| **4. The Assets** | W7–10 | 48 | Fixed Income (duration/convexity) and Equity (DCF, multipliers) — the two 11–14% blocks; then Derivatives (conceptual pricing) and Alternative Investments (PE/HF/real estate mechanics — cross-ref the repo's AM guides) | All four asset topics read; 300 Qs |
| **5. The Issuers & The Portfolio** | W11–12 | 24 | Corporate Issuers (capital budgeting, capital structure, governance) and Portfolio Management (MPT, CAPM, IPS) | Both topics + 150 Qs |
| **6. Ethics + PSM** | W13–14 | 24 | Ethics deep-dive: Code & Standards case studies, GIPS; complete the Level I **Practical Skills Module** (required before results) | PSM done; 200 ethics Qs |
| **7. Volume** | W15–18 | 48 | Full QBank sprint: 2,000+ questions across all topics, weak-topic re-reads, SRS cards daily; first 2-hour timed mini-sessions | QBank ≥ 70% overall |
| **8. Mock 1** | W19–20 | 24 | Mock exam 1 under real conditions (2×135 min, same time of day, no phone); full paper review with a mistake log | Score + gap list |
| **9. Mock 2 & 3** | W21–24 | 48 | Mock 2, targeted fixes on gaps, Mock 3, ethics re-drill, Secret Sauce-style final notes | Two more mocks; scores trending ≥ MPS estimate |
| **10. Exam week** | W25–26 | 24 | Light review only (formula sheet, ethics cards, mistake log); sleep and logistics: passport, calculator batteries, clear water bottle, test-center route, arrive 30+ minutes early | Ready |

**Weekly cadence (the 12-hour week):** Mon/Tue/Wed 90 min each (new content + SRS), Thu 60 min (QBank), Fri off, Sat 3h (deep work: one full learning module), Sun 3h (QBank + weekly mock mini-section + SRS review). The Sunday block is non-negotiable — it is the compounding engine. Formula: **312 total hours ≈ 300-hour guideline + margin.**

**The week-by-week plan (August 2027 target, 2027 curriculum):**

| Week | Focus | Outputs / milestones |
|------|-------|----------------------|
| W1–W2 | Quant Methods: TVM, probability, distributions | Formula sheet v1; SRS deck seeded; 150 Qs |
| W3–W4 | FSA: the three statements, ratio analysis | Ratio cheat-sheet; 100 Qs |
| W5 | FSA: cash flow, quality of earnings; start Ethics readings | Ethics cards started; 100 Qs |
| W6 | Economics: micro + macro + FX basics | 100 Qs; first 30-min timed quiz |
| W7–W8 | Fixed Income: mechanics, yield/spread, duration/convexity | Duration workbook; 150 Qs |
| W9–W10 | Equity: markets/indices, DCF, multipliers | Valuation model one-pagers; 150 Qs |
| W11 | Derivatives: forwards/futures/swaps/options basics | Payoff-diagram cards; 100 Qs |
| W12 | Alternative Investments: PE/HF/RE/commodities | Fee-structure summary (cross-ref repo AM guides); 100 Qs |
| W13 | Corporate Issuers: capital budgeting/structure, governance | 100 Qs; 60-min timed quiz |
| W14 | Portfolio Management: MPT, CAPM, IPS | IPS template; 100 Qs |
| W15–W16 | Ethics deep-dive + complete the Level I PSM | PSM done; 300 ethics Qs; ethics case-log |
| W17–W18 | QBank volume sprint (1,000 Qs) + weak-topic re-reads | QBank ≥ 70%; error log v1 |
| W19 | Mock 1 (full, timed) + review | Score recorded; gap list written |
| W20 | Targeted fixes on gap list + 500 Qs | Error log v2 |
| W21 | Mock 2 + review | Trend check vs MPS estimate |
| W22–W23 | Final QBank sweep + Ethics re-drill + Secret Sauce-style notes | Formula sheet v2 (one page per topic) |
| W24 | Mock 3 + light re-reads | Confidence pass |
| W25–W26 | Review only: formula sheet, ethics cards, error log; logistics | Ready: passport, calculator, route, sleep |

**Contingency design:** if Mock 1 scores below ~55% (a common heuristic for "below the MPS band"), the plan flexes W22–W23 into remediation and the November 2027 window remains as the paid retake option — the August date was chosen precisely to keep that option open (the six-month rule would have blocked a May 2027 retake after a February attempt).

### 10.3 The Lessons

1. **The 300 hours are real but they are a floor, not a plan.** CFA Institute's "over 300 hours per level" is a *reported average of successful candidates*; the plan above spends them deliberately (60% content, 40% practice) rather than just "studying for six months."
2. **Weights are the strategy.** Ethics + FSA + Equity + Fixed Income = ~50% of Level I. Candidates who allocate by weight and start the heavy topics first systematically outperform those who read the curriculum in page order.
3. **Practice beats reading, and mock exams beat practice.** The exam is a retrieval test; the candidates who fail are overwhelmingly the ones who "finished the books" and never did timed mocks. Two to three full mocks, timed, in the final six weeks, is the single most transferable lesson from the prep-industry data.
4. **Ethics is the tie-breaker, not a checkbox.** 15–20% weight, the borderline ethics adjustment, and an annual professional obligation afterward — ethics is studied in cases, not memorized.
5. **The format shift between levels is the real difficulty curve.** Level I's standalone MCQs reward breadth; Level II's vignettes reward triage; Level III's essays reward articulation. Plan for the *next* format, not the one you just passed.
6. **The calendar drives everything.** Windows (4/3/2 per year), the six-month attempt rule, early-registration savings (USD 350/window), scheduling deadlines, and the 2027 curriculum boundary (Feb 2027 onward) — a candidate who picks the window first and works backwards never fights the calendar.
7. **Budget the full cost, not the fee.** Fees are USD 3,520–4,570 across three levels, but the real envelope is fees + prep (USD 300–800/level) + calculator + taxes + retake probability. The 2026 fee overhaul helps new candidates once (~USD 150–200) and hurts returners slightly — plan for the worst-case retake, not the happy path.
8. **For a technologist, the CFA is an optionality purchase.** The charter does not certify systems skills — it certifies that you can speak the investment language fluently, which is exactly what front-office-aligned architecture roles (and private-markets platforms) increasingly demand. Study it for the language and the signaling; the [FRM](financial_risk_compliance_systems_guide.md) remains the better pure-risk fit if risk technology is the actual destination (see §8).

### 10.4 Alternative Plans (the schedule flexes, the principles don't)

**The compressed plan (~6 months, 20 h/week ≈ 480 h):** halves the calendar by running the phases in parallel — FSA and Ethics from day one, two topics per week in the middle phase, and four mocks instead of three. Only for candidates who can sustain 20 focused hours a week; the risk is burnout in weeks 10–18, which is exactly when the 2021-style under-performance shows up.

**The extended plan (~9 months, 8 h/week ≈ 312 h):** stretches the same phase structure with a full review month before the mocks. The win is resilience (sick weeks, work crunches, and travel don't break the chain); the risk is forgetting — which is precisely why the SRS deck (cross-ref [../technology/spaced_repetition_apps_guide.md](../technology/spaced_repetition_apps_guide.md)) is non-optional in the extended variant.

**The retake plan (the one nobody wants but the median candidate needs):** after a fail, the research is unambiguous — the gap is almost always *question volume under time pressure*, not missing knowledge. The standard fix is 12 weeks, 300+ hours, 4,000+ QBank questions, and two mocks *before* the content re-read even starts. Passing a retake is a practice problem, not a re-reading problem.

---

## 11. The Summary: The Charter's Three Gates

**One page. The CFA Program is three exams and one character test, and the charter is what you get for passing all of them.**

| Gate | Exam | What it proves | Format | Typical pass bar | Cost (time/money) |
|------|------|----------------|--------|------------------|-------------------|
| **Gate 1 — Knowledge** | Level I | You know the industry's vocabulary, mechanics, and ethics | 180 standalone MCQs, one day | ~41–45% | ~300 h; USD 1,140–1,490 |
| **Gate 2 — Application** | Level II | You can use that knowledge on messy real cases | 22 vignettes / 88 questions, one day | ~45–47% | ~300 h; USD 1,140–1,490 |
| **Gate 3 — Integration & Character** | Level III | You can allocate, build, and justify in writing | 11 item sets + 11 essay sets, one day | ~50–52% | ~300 h; USD 1,240–1,590 |
| **The fourth wall** | Experience + ethics | You have *done* it professionally, and can be trusted | 4,000 h / 3 yrs experience + references + membership + annual ethics commitment | — | 3+ years alongside the exams |

- **Gate 1 — Knowledge (Level I):** prove you know the whole investment industry's vocabulary and mechanics — ten topics, 180 standalone questions, one day, ~45% of candidates pass. This gate filters *breadth*: 93 learning modules, 365 learning outcome statements, ethics at 15–20%.
- **Gate 2 — Application (Level II):** prove you can *use* that knowledge on real cases — 22 vignettes with 88 questions, two sessions, ~44–47% pass. This gate filters *judgment under ambiguity*: the case has more information than you need, and finding what matters is the skill.
- **Gate 3 — Integration and Character (Level III + the fourth wall):** prove you can build portfolios, allocate capital, and *write your reasoning* — 11 item sets and 11 essay sets across two sessions, ~50% pass, plus your chosen pathway (Portfolio Management, Private Markets, or Private Wealth). Then the fourth wall: 4,000 hours of qualified work experience, references, membership, and the annual ethics commitment — the charter is a *standing* professional obligation, not a trophy.
- **The economics:** ~300 hours per level, USD 3,520–4,570 in fees, 3–4 years part-time, and roughly 200,000 people have made it through since 1963 — the scarcity is the point. The 2021 pass-rate collapse (22–27% at Level I) is the cautionary tale; the 2025 rebound (43–54% across levels) is the reminder that the bar is stable and the format is knowable.
- **The verdict for a banking technologist:** the CFA buys the language, the signaling, and the optionality to move toward the investment side — the private-markets and asset-management franchises this repo documents are exactly where the charter's stock is highest. Take it if you want that optionality; take the FRM if risk technology is the real target; take the CAIA if alternatives depth is the goal. Take none of them if the plan is pure platform engineering — but then you are reading the wrong guide.

**The final word:** the charter's three gates are knowledge, application, and judgment — and the last one is the one nobody can study for in a weekend. Pass the first two with hours and practice; pass the third with judgment, ethics, and time in the industry. That is why the CFA has been the investment profession's benchmark for six decades: it tests not just what you know, but that you have *done* something with it, and that you will be trusted with other people's money afterward.

### 11.1 Quick Answers (FAQ)

**Is the CFA worth it?** For buy-side, research, wealth, and generalist AM careers: yes, it is the standard credential. For pure technology/engineering roles: no — take it only as an optionality purchase toward front-office-adjacent work (see §8–§9).

**How long does the whole journey take?** Exams alone can be done in ~2–3 years (one level per year with the attempt rules), but the charter needs 4,000 hours of qualified work experience over a minimum of three years — the realistic end-to-end timeline is 3–5 years part-time.

**How hard is it, really?** Each level is ~300 hours of study and a one-day exam with a 40–50% pass rate. The difficulty is less the math (conceptual, not stochastic-calculus) than the *volume* — 93 learning modules at Level I — and the format shifts between levels.

**Do I need a finance degree?** No. A bachelor's degree (any field) or final-year status within 23 months of graduation is enough to sit the exam; the curriculum assumes no finance background.

**Can I work while studying?** Yes — the program is designed for working professionals; the §10 plan is exactly that (12 h/week over 26 weeks). The failure mode is not working while studying; it is studying *without practice* (no QBank volume, no mocks).

**How much does it cost, honestly?** Fees alone: USD 3,520–4,570 for all three levels at 2026 rates. With prep, calculator, taxes, and a realistic retake contingency: budget USD 6,000–7,000+ end-to-end (see §6.3).

**Which prep provider is best?** The consistent answer: the one you will actually use. Kaplan Schweser (structure), Mark Meldrum (understanding), AnalystPrep/UWorld (practice) are the leaders; the recipe that matters more than the brand is notes/videos + 2,000+ questions + 2–3 timed mocks (see §7).

**CFA, FRM, or CAIA?** Career-target decision: CFA = generalist investment management; FRM = risk; CAIA = alternatives/private markets. They stack; the mistake is substituting the specialist credential for the generalist one (see §8).

**What happens if I fail?** You can retake (max two attempts per calendar year, not in consecutive windows, not within six months). Failing costs the fee (USD 1,140–1,590) plus time; the standard retake plan is 12 weeks of question-volume-driven study, not a re-read of the books (see §10.4).

**What does the charter actually change?** Three things: the right to use the CFA designation (with the annual ethics commitment), membership in a 200,000+ global professional community, and a durable, globally recognized signal of investment competence — which is why the buy-side treats it as near-mandatory and why it survives resume screens that filter out everything else.

---

## 12. Verification and Claims Status

**Verified against primary sources (cfainstitute.org official pages, extracted August 2026):**
- Exam dates & fees page: 2026 fees (Level I/II early USD 1,140, standard USD 1,490; Level III early USD 1,240, standard USD 1,590), USD 250 rescheduling fee, USD 3,520–4,570 full-program range, 400+ test locations, four exam periods per year, two-attempts-per-year rule, "300 hours per level" figure, taxes excluded.
- Exam information page: level-by-level formats (180 MCQ L1; 22 item sets/88 questions L2 with 20 scored + 2 pretest; 11 item sets + 11 essay sets L3 with 6+5/5+6 session split), session lengths (135 min L1, 132 min L2/L3), 10-year average pass rates (41% / 45% / 52%), results timing (5–7 weeks L1/L2, 6–8 weeks L3), PSM requirement, calculators allowed (TI BA II Plus, HP 12C), passport/water/arrival rules.
- Level II outline page: 2026 Level II topic weights; vignette/item-set structure.
- Level III outline page: 2026 Level III topic weights (incl. Pathways 30–35%); three specialized pathways (Portfolio Management, Private Markets, Private Wealth); mixed item-set/essay structure.
- History: FAF 1947, ICFA 1962, first exam 1963, AIMR 1990 (CFA Institute history timeline PDF + Wikipedia cross-check); 2021 CBT transition (Feb 2021 first computer-based Level I); 200,000+ charterholders in 160+ markets (CFA Institute community pages); 2025 scale-score MPS (1600/2600/3600) and 2024–2026 pass-rate series (cross-checked across prep-industry trackers consistent with CFA Institute results reports).

**Flagged / not fully verified (be honest before quoting):**
- 2027 curriculum specifics (renames to Corporate Finance / Equities / Portfolio Construction; 102 learning modules; AI/LLM quant module) are from prep-industry reporting, not re-verified against CFA Institute's 2027 outline PDFs — treat as high-confidence but confirm before basing a study plan on them.
- The exact 2025 Level III early/standard fees (row marked "approx., flag") and 2025 enrollment-fee arithmetic rely on the CFA Institute "Pricing changes" release as reported by the prep press (18 Dec 2024) — the 2026 official figures supersede them.
- Level III pathway launch timing "since 2025" (Wikipedia + official page references); the first Private Markets pathway window specifics were not re-extracted from the primary announcement.
- The 2004 "AIMR → CFA Institute" rename year is widely documented but not re-verified against a primary source for this edition.
- Prep-provider prices (Schweser USD 700–1,300, Meldrum ~399, AnalystPrep ~349, UWorld, Wiley) are indicative 2026 comparison-press figures, not provider-verified quotes — they vary by package, promotion, and region.
- FRM and CAIA figures (GARP founding 1997, ~90,000 FRMs; CAIA founding 2002, ~13,000+ members, ~USD 3,000–3,500 cost, ~200–300 hours/level) are approximate — verify at garp.org and caia.org before making decisions on them.
- The 2021 pass-rate explanations (pandemic disruption, CBT transition, under-prepared influx) are industry consensus framing, not CFA Institute's official attribution.

**Sources consulted (August 2026):**
- cfainstitute.org — program overview, dates & fees (interactive tool), exam information page, Level II and Level III outline pages, community/employer pages ("200,000+ charterholders"), history timeline PDF.
- Wikipedia — "Chartered Financial Analyst" (history, requirements, curriculum structure, regulatory recognition) and "CFA Institute," used as cross-checks.
- Prep-industry trackers and comparisons — Soleadea (topic weights 2026/2027; pass-rate series 2016–2026; MPS scale scores; 2027 curriculum changes), Rankers Financial Academy (format explainer by a CFA charterholder), QuintEdge (2026–27 window calendar verified against the CFA Institute tool), IMS Proschool (2026 fee-change arithmetic citing the CFA Institute pricing release of 18 Dec 2024), 300 Hours, CFAReview.org, Efficient Learning, playprephq and courses4you (provider lineup/pricing for 2026).
- The repo's own banking series for the Singapore/private-markets/AM context (see the cross-reference block at the top).

---

## 13. Glossary

- **CFA (Chartered Financial Analyst):** the professional charter awarded by CFA Institute to candidates who pass three levels of exams, complete practical skills modules, accrue 4,000 hours of qualified work experience, and commit to the Code of Ethics.
- **CFA Institute:** the US-based not-for-profit association (FAF 1947 → ICFA 1962 → AIMR 1990 → CFA Institute 2004) that administers the CFA Program, publishes the curriculum, and owns GIPS, the Code of Ethics and Standards of Professional Conduct, and the CBOK.
- **Charter:** the formal designation/certificate awarded by CFA Institute — used both for the document and, by extension, the standing professional credential ("the CFA charter").
- **Charterholder:** a person who has earned and currently holds the CFA charter and is in good standing (annual ethics commitment + membership).
- **Level I:** the first CFA exam — 180 standalone multiple-choice questions, 2 × 135 minutes, ten topics, four windows a year; tests knowledge and recall.
- **Level II:** the second CFA exam — 88 multiple-choice questions across 22 vignette item sets, 2 × 132 minutes, three windows a year; tests application and case triage.
- **Level III:** the third CFA exam — 11 item sets + 11 constructed-response (essay) sets, 2 × 132 minutes, two windows a year; tests integration, judgment, and written articulation; includes a pathway choice (Portfolio Management, Private Markets, Private Wealth).
- **Curriculum:** the annually refreshed CFA study program organized into topics, learning modules, and learning outcome statements (LOS), derived from the Candidate Body of Knowledge.
- **CBT (computer-based testing):** the exam delivery mode introduced in February 2021 (Level I) and completed across all levels during 2021, replacing paper exams; delivered at Prometric test centers.
- **Computer-based testing:** see CBT.
- **Ethics (Ethical and Professional Standards):** the highest-weight topic at every level (15–20% at Level I); the Code of Ethics, Standards of Professional Conduct, and GIPS; the only topic with a borderline score-adjustment mechanism.
- **Quantitative Methods:** the Level I/II topic covering time value of money, probability, hypothesis testing, regression, and machine-learning surveys.
- **Economics:** the micro/macro/international-economics topic; small weight, fast marks.
- **Financial Statement Analysis:** the IFRS/US-GAAP financial-reporting and ratio-analysis topic; the biggest skill block and the Level II foundation.
- **Corporate Issuers:** the corporate-finance topic (capital budgeting, capital structure, dividends, governance); renamed from "Corporate Finance" in 2024 and slated to revert to "Corporate Finance" in 2027 at Level I.
- **Equity:** the equity-markets, industry/company analysis, and valuation topic.
- **Fixed Income:** the bond-mechanics, yield/spread, duration/convexity, and securitization topic.
- **Derivatives:** the forwards/futures/swaps/options topic; conceptual pricing (binomial, Black–Scholes) and hedging applications.
- **Alternative Investments:** the hedge-fund, private-equity, real-estate, commodity, and infrastructure topic.
- **Portfolio Management:** the MPT/CAPM/efficient-frontier and IPS topic; expands into the entire Level III core (asset allocation, portfolio construction, performance measurement).
- **Pass rate:** the percentage of candidates who pass a given exam window; 2025 rates were 43–54% by level/window; 10-year averages 41/45/52% for Levels I/II/III.
- **Registration:** the process (and fee) of signing up for an exam window on cfainstitute.org — distinct from scheduling the appointment at a test center; 2026 fees USD 1,140–1,590 per level depending on level and timing.
- **Kaplan Schweser:** the market-leading third-party prep provider (SchweserNotes, QBank, Secret Sauce, mocks); the reference point for prep pricing.
- **Prep provider:** any commercial course/materials vendor (Kaplan Schweser, Mark Meldrum, AnalystPrep, UWorld, Wiley, Bloomberg, etc.) offering compression and practice on top of the official curriculum; not officially endorsed by CFA Institute.
- **FRM (Financial Risk Manager):** GARP's risk-management designation — two parts, market/credit/operational/liquidity risk focus; the risk-function counterpart to the CFA.
- **CAIA (Chartered Alternative Investment Analyst):** the CAIA Association's alternatives designation — two levels covering private equity, hedge funds, real estate, commodities, and fund structures; the alternatives counterpart to the CFA.
- **300 hours:** CFA Institute's stated average study time per level reported by successful candidates; the planning unit for every credible study plan (~300–330 hours per level).
- **Candidate:** a person enrolled in the CFA Program (registered for or studying toward an exam) who has not yet earned the charter.
- *Also useful:* **MPS** (minimum passing score — the per-window passing bar; scale scores 1600/2600/3600 since 2025), **Item set / vignette** (the Level II–III case format), **PSM** (Practical Skills Module — required per level before results), **Learning Ecosystem** (CFA Institute's digital study platform), **GIPS** (Global Investment Performance Standards), **Pathway** (Level III specialization), **CBOK** (Candidate Body of Knowledge).

---

*End of guide — cross-referenced into the repo's banking/ series; sibling banking guides are linked by plain filename and technology/ guides by the ../technology/ prefix, per repo convention. Last updated August 2026.*

*Suggested next steps for the reader: (1) read §11 and §10 first; (2) verify the current window calendar and fees on cfainstitute.org/programs/cfa-program/dates-fees before registering — the interactive tool is the only authoritative fee source; (3) if Singapore-based, talk to your employer about sponsorship and reach out to CFA Society Singapore's candidate events; (4) treat every number in this guide marked "flag" as a to-verify item rather than a fact.*
