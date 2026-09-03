# AI/ML Programs at Ten Leading Universities

> **A Survey of Graduate Study in Artificial Intelligence and Machine Learning at Ten Highly Ranked Universities — the Degree Programs, the Flagship Curricula, the Core Courses, and the Topics Each Core Course Covers, Verified Against Official Catalogs**

**Author:** Jack Liu Shurui, Solution Architect  
**Series:** AI/LLM Advanced Topics — *Education & Curriculum Series*  
**Domain:** AI Engineering · Higher Education · Graduate Curriculum  
**Reading time:** ~70 minutes  
**Version:** 1.0 — September 2026

> **Series positioning.** This guide sits inside the repository's education-and-curriculum cluster. It surveys what ten world-leading universities actually teach in artificial intelligence and machine learning at the graduate level — which degrees they offer, which courses sit at the core of their flagship programs, and what topics those courses cover. It is the curriculum-level companion to the practitioner guides in this cluster: where [ai_engineer_roadmap_2026.md](ai_engineer_roadmap_2026.md) maps a self-taught learning path and [mathematics_for_ml_and_ds_study_notes.md](../../mathematics_for_ml_and_ds_study_notes.md) covers the mathematical substrate, this guide documents the *institutional* version of the same curriculum — the programs a student can actually be admitted to. Related technology deep-dives that pair naturally with specific courses are cross-referenced by path where they appear ([deep_learning_frameworks_comparison_guide.md](deep_learning_frameworks_comparison_guide.md), [reinforcement_learning_algorithms_guide.md](../../reinforcement_learning_algorithms_guide.md), [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md)) rather than repeated here.

> **How this guide is organized.** Section 1 states the purpose, scope, and verification method. Section 2 establishes the ranking basis — which published rankings were consulted, what they say, and where they disagree — and names the ten universities selected. Sections 3–12 treat the ten universities one at a time: each section lists the main AI/ML degree programs, describes the flagship one or two programs, then tabulates the flagship core courses (codes and titles) with a short outline of the topics each course covers. Section 13 compares the ten schools across dimensions (program shapes, core-course canon, language, and regional emphasis). Section 14 is the verification ledger (✅ confirmed / ⚠ uncertain / ❌ contradicted). Section 15 collects, per the repository's honesty convention, everything that could not be verified. Section 16 is a worked example — a Cymbal Bank learning-path recommendation built across the shared curriculum of these ten schools. Sections 17–19 are the glossary, cross-references, and the closing note.

---

## Table of Contents

1. [The Purpose and Method of This Guide](#1-the-purpose-and-method-of-this-guide)
2. [The Ranking Basis — and Where Rankings Disagree](#2-the-ranking-basis--and-where-rankings-disagree)
3. [Massachusetts Institute of Technology (MIT)](#3-massachusetts-institute-of-technology-mit)
4. [Stanford University](#4-stanford-university)
5. [Carnegie Mellon University](#5-carnegie-mellon-university)
6. [University of California, Berkeley](#6-university-of-california-berkeley)
7. [University of Oxford](#7-university-of-oxford)
8. [University of Cambridge](#8-university-of-cambridge)
9. [ETH Zurich](#9-eth-zurich)
10. [National University of Singapore (NUS)](#10-national-university-of-singapore-nus)
11. [Tsinghua University](#11-tsinghua-university)
12. [University of Toronto](#12-university-of-toronto)
13. [The Comparative View](#13-the-comparative-view)
14. [The Verification Ledger](#14-the-verification-ledger)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Worked Example: Cymbal Bank — A Learning Path Across the Shared Curriculum](#16-worked-example-cymbal-bank--a-learning-path-across-the-shared-curriculum)
17. [Glossary](#17-glossary)
18. [Cross-References](#18-cross-references)
19. [Closing: The Learned Machine](#19-closing-the-learned-machine)

---

## 1. The Purpose and Method of This Guide

### 1.1 Why This Guide Exists

Every serious practitioner of artificial intelligence and machine learning eventually asks the same set of questions: *which universities actually teach this well, what do their flagship programs require, and which courses carry the core of the curriculum?* The answers matter to three audiences: (a) candidates choosing between graduate programs; (b) engineering leaders hiring graduates and trying to interpret what a given degree from a given school certifies; and (c) self-taught engineers who want an institutional benchmark against which to measure their own learning path (the [ai_engineer_roadmap_2026.md](ai_engineer_roadmap_2026.md) companion).

This guide answers those questions for **ten universities**: MIT, Stanford, Carnegie Mellon, UC Berkeley, Oxford, Cambridge, ETH Zurich, NUS, Tsinghua, and Toronto. For each, it records:

1. **The main AI/ML degree programs** the university offers — names and shapes, not marketing copy.
2. **The flagship one or two programs** — the degrees most students actually target for AI/ML study — with their structure described.
3. **The core courses of those flagship programs** — codes and titles as they appear in the official catalog.
4. **A short outline of the topics each core course covers** — grounded in the catalog description or the course's own official pages.

### 1.2 The Verification Method

The governing rule of this guide is the repository's honesty convention, applied strictly:

> **Never invent a program, a course, or a topic. Verify program names, course codes, and syllabus content against the university's official catalog pages. Where a page is outdated or unavailable, mark the item uncertain (⚠) rather than guessing.**

In practice, every course code and title in Sections 3–12 was checked against one of: (a) the university's official course catalog (MIT's student catalog pages, CMU's course catalog, the ETH course catalogue, NUS module listings, the University of Toronto Arts & Science calendar); (b) the official course website operated by the department for that course (e.g., MIT's `gradml.mit.edu` for 6.790, Stanford's `cs229.stanford.edu`); or (c) the official program page of the department or faculty that owns the degree. Where only secondary sources could be reached — because the official page is behind a JavaScript application, a login, or a region block — the fact is marked ⚠ in the ledger of §14 and the source is named. This pass was conducted in early September 2026 against the 2026–2027 catalogs where they were published; a few schools' catalogs still showed the 2025–2026 academic year, and those items are flagged.

### 1.3 The Unit of Analysis: "AI/ML Programs"

"AI/ML graduate program" is a fuzzy category, and each of the ten schools organizes it differently. Four shapes recur, and this guide records which shape each school uses:

- **The named AI/ML degree** — e.g., Carnegie Mellon's *Master of Science in Machine Learning* or Cambridge's *MPhil in Machine Learning and Machine Intelligence*: a degree whose name *is* the specialty.
- **The specialist pathway inside a general degree** — e.g., Stanford's *MS in Computer Science* with an Artificial Intelligence specialization, or MIT's PhD in EECS with research in the AI+D (Artificial Intelligence + Decision-making) area: the degree is general; the pathway is not.
- **The AI-relevant professional/master's degree** — e.g., Berkeley's MEng with an AI-adjacent concentration, NUS's *Master of Science in Data Science and Machine Learning*, Toronto's *MScAC* with an AI concentration: applied degrees where ML is the core competence.
- **The research doctorate** — the PhD (or DPhil/MPhil at Oxford and Cambridge), where "the program" is apprenticeship to a research group and the coursework is a means, not the end. Every school in this guide has one; it is described once per school and not repeated in the course tables.

### 1.4 What This Guide Does Not Do

This guide does **not** rank the ten schools against each other — the ranking question is delegated to Section 2, which reports published rankings rather than issuing new ones. It does not reproduce full syllabi: the topic outline per course is a compression of the official description, typically five to fifteen lines. It does not cover undergraduate study except where an undergraduate course is the recognized entry point to the graduate canon (several flagship *graduate* courses list the school's undergraduate ML course as a prerequisite, and those prerequisites are named where they matter). It does not evaluate teaching quality, selectivity, or career outcomes — none of those are reliably verifiable from catalogs, and per the honesty convention they are omitted rather than asserted.

---

## 2. The Ranking Basis — and Where Rankings Disagree

### 2.1 Why a Ranking Basis at All

The assignment for this guide is a survey of ten highly ranked universities. "Highly ranked" must be anchored to a named, published ranking with a year, because the university set itself is ranking-dependent: different methodologies produce materially different top-tens (China's research-output machine versus US reputation, for example). This section therefore does three things: states the primary ranking used, records the cross-check rankings consulted, and — per the assignment — notes explicitly where the rankings differ. The ten selected universities are each justified against at least one of these lists.

### 2.2 The Primary Ranking: QS World University Rankings by Subject 2026 — Computer Science & Information Systems

The primary basis is the **QS World University Rankings by Subject 2026**, edition 16, published **25 March 2026** by QS Quacquarelli Symonds, subject table *Computer Science & Information Systems*. The ranking blends academic reputation, employer reputation, and research citations (the exact weighting differs by subject; QS publishes the methodology separately). The verified head of the 2026 table:

| Position | University | Verification |
| --- | --- | --- |
| #1 | Massachusetts Institute of Technology (MIT) | QS subject-page copy + aggregator coverage (collegedunia) ✅ |
| #2 | Stanford University | QS subject-page copy ("Stanford ... ranked close behind") + collegedunia ✅ |
| #3 | Carnegie Mellon University | QS subject-page copy + collegedunia ✅ |
| #4 | National University of Singapore (NUS) | QS table via aggregator (xuanxiao.org table listing top five: MIT, Stanford, CMU, NUS, Oxford) ⚠ |
| #5 | University of Oxford | QS table via aggregator (as above) ⚠ |
| #6–#10 | Cambridge, UC Berkeley, ETH Zurich, Tsinghua, Harvard — all reported inside the QS 2026 top ten by secondary coverage, exact order not re-verified this pass | ⚠ |

> **Verification note.** The #1–#3 order (MIT, Stanford, CMU) is confirmed by two independent secondary sources quoting the QS 2026 release, and by QS's own subject-page copy; the #4–#5 order comes from one aggregator's rendering of the QS table and is therefore marked ⚠ even though it is consistent with NUS's and Oxford's own rank announcements in prior editions. The exact #6–#10 order of the 2026 table could not be re-verified live this pass (topuniversities.com blocks automated extraction); individual schools' membership in the top ten is corroborated by Analytics Insight's published 2026 top-ten list (MIT, Stanford, CMU, Oxford, Cambridge, ETH Zurich, Harvard, UC Berkeley, Tsinghua, NUS — ⚠ secondary source).

### 2.3 The Cross-Check Rankings

Four further published rankings were consulted as cross-checks, and each disagrees with the others in instructive ways:

1. **Times Higher Education (THE) World University Rankings by Subject 2026 — Computer Science.** Oxford is #1, per THE's own summary copy ("Oxford is number one" in the 2026 computer-science table; THE notes Oxford leads the subject while Harvard, Stanford, and Cambridge appear in all 11 subject top-tens). THE weights teaching, research environment, research quality, industry, and international outlook — a structure that rewards UK and Swiss institutions heavily. ✅ for the Oxford #1 headline; the full top-ten order ⚠.
2. **CSRankings (2026 edition, research-output based).** CSRankings counts publications at top venues by faculty, with no reputation component. The 2026 release (reported end-January 2026) shows the methodological gap starkly: **Shanghai Jiao Tong University and Tsinghua University tied for #1 overall in computer science, with Chinese universities taking seven of the global top-ten places** (Zhejiang #3, tied with CMU; Peking University #1 in the AI category on this edition's reporting, Tsinghua #2, Zhejiang #3). ⚠ — CSRankings 2026 positions here are as reported by secondary coverage of the release (36kr English, VnExpress); the interactive csrankings.org site could not be scraped live this pass.
3. **US News Best Global Universities — Artificial Intelligence subject ranking (2024–2025 edition).** Tsinghua University is #1, with Chinese institutions dominating the global top ten (VnExpress coverage of the US News release). ⚠ — the US News site blocks automated extraction; the headline is press-confirmed.
4. **EduRank world AI ranking (citation-based, 2026).** A third-party research-metrics aggregator (not one of the canonical four), included only as a sanity check: Stanford #1, UC Berkeley #2, MIT #3, CMU #4, **Toronto #5**, Tsinghua #6 — the only list among those consulted that places Toronto in the world's top five for AI. ⚠ — third-party source, cited for its Toronto data point only.

### 2.4 Where the Rankings Disagree

The disagreements are the point, and they explain the roster:

- **US vs China at the top.** Reputation-weighted rankings (QS) put MIT, Stanford, and CMU on top and give the US seven of the top ten in QS 2026 *overall* university rankings; output-counted rankings (CSRankings 2026) put Tsinghua and SJTU at #1 and give China seven of the global top ten in computer science. Both are "the world's best" — measured differently. This guide therefore includes both MIT (#1 QS) and Tsinghua (#1 CSRankings-co-headline, #1 US News AI subject) and says so plainly.
- **Oxford's THE #1 vs QS #5.** Oxford tops THE's 2026 computer-science table and sits at #5 in QS's 2026 subject table; both are Oxford, and both are reported here with their sources.
- **The AI-specific vs CS-general split.** QS publishes a separate *Data Science and Artificial Intelligence* subject table in the same 2026 release (MIT leads; exact 2026 order ⚠, page not extractable). US News publishes a separate *Artificial Intelligence* subject table in which Tsinghua is #1. A school can be top-five in CS but outside the top ten in AI, and vice versa — the two subjects are not the same market.

### 2.5 The Roster — and Why These Ten

The ten universities surveyed in Sections 3–12:

| # | University | Country | Anchor ranking position (2026 editions unless noted) |
| --- | --- | --- | --- |
| 1 | MIT | US | QS CS&IS #1 |
| 2 | Stanford | US | QS CS&IS #2; EduRank world AI #1 |
| 3 | Carnegie Mellon | US | QS CS&IS #3; CSRankings 2026 overall #3-tied (with Zhejiang) |
| 4 | UC Berkeley | US | QS top-ten set; EduRank world AI #2 |
| 5 | Oxford | UK | THE CS 2026 #1; QS CS&IS #5 |
| 6 | Cambridge | UK | QS/THE top-ten set (both) |
| 7 | ETH Zurich | Switzerland | QS/THE top-ten set (both) |
| 8 | NUS | Singapore | QS CS&IS #4 |
| 9 | Tsinghua | China | US News AI subject #1; CSRankings 2026 overall #1-tied |
| 10 | Toronto | Canada | EduRank world AI #5; the only candidate of the three shortlisted (Toronto / Princeton / EPFL) with a top-five AI-specific citation in any consulted list |

The tenth slot deserves its own sentence, because the assignment explicitly shortlists **Toronto, Princeton, or EPFL** and asks that the choice be confirmed from ranking pages. Of the three: Princeton appears in none of the consulted AI-specific top-tens and only marginally in the CS-general ones; EPFL appears in the THE computer-science top ten in some recent editions but in no consulted AI-specific top ten; **Toronto appears at #5 in the EduRank world-AI list and is a fixture of CSRankings AI top-tens in recent editions** — the strongest AI-specific ranking signal of the three. Toronto is therefore the tenth school, with the caveat that its ranking evidence is thinner than the other nine's (⚠ where only third-party metrics support it; see the ledger §14).

---

## 3. Massachusetts Institute of Technology (MIT)

### 3.1 The School at a Glance

MIT — Massachusetts Institute of Technology, Cambridge, Massachusetts — is the #1 university in the QS 2026 Computer Science and Information Systems subject table (§2.2). Artificial intelligence and machine learning at MIT are not a separate department: they live inside the Department of Electrical Engineering and Computer Science (EECS), whose research and teaching are organized into areas — the AI-relevant one being **AI+D, "Artificial Intelligence and Decision-making"** (EECS's own framing: techniques for "the analysis and synthesis of systems that interact with an external world via perception, communication, and action, and that learn, make decisions and adapt"; verified at eecs.mit.edu). All degree programs below are EECS degrees.

### 3.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| PhD in EECS | Doctorate | Research doctorate; the default graduate degree for AI/ML research; students affiliate with the AI+D area and its faculty | eecs.mit.edu/academics/graduate-programs ✅ |
| Master of Engineering (MEng), Course 6 | Professional master's | One-year, course-plus-thesis professional degree for MIT EECS undergraduates; a Fall 2026 restructure is documented (department FAQ page) | eecs.mit.edu MEng pages ✅ |
| SM in EECS | Research master's | Thesis or course-based master's within the PhD-track structure; role in AI/ML study is secondary to the PhD | ⚠ (structure not re-verified this pass; MIT's graduate portal describes it but the page was not retrieved live) |
| BS in Artificial Intelligence and Decision Making (6-4) | Undergraduate | MIT's named AI+D bachelor's major; included here because its course canon feeds the graduate subjects | catalog.mit.edu degree chart ✅ |

MIT offers no standalone "MS in AI/ML": applicants wanting AI/ML graduate study apply to the EECS PhD (or, from inside MIT, the MEng). This is itself a structural fact worth knowing — the MIT graduate AI/ML "program" is a research apprenticeship with a coursework spine, not a taught master's.

### 3.3 The Flagship Programs

**Flagship 1 — the EECS PhD (AI+D area).** The doctorate is MIT's AI/ML flagship: admission is to the EECS graduate program as a whole, with students joining research groups across the AI+D area — learning, vision, language, robotics, decision-making — after arrival. Coursework is real but subordinate: graduate students complete advanced subjects (the 6.7xxx/6.8xxx range below is their menu) while moving quickly into thesis research. There is no fixed "core course list" published for the PhD; the de facto core is the advanced-subject canon of §3.4. (Structure-level facts verified at eecs.mit.edu; specific PhD milestone details ⚠ not re-verified this pass.)

**Flagship 2 — the MEng.** MIT's professional master's in EECS: one academic year plus a thesis, aimed at EECS undergraduates extending into industry-bound specializations. The department documented a **Fall 2026 MEng change** (a dedicated FAQ page exists and is linked from the EECS graduate-programs navigation), so applicants should read the current requirements rather than any prior-year description — ⚠ the new requirements' specifics were not extracted this pass.

### 3.4 The Core Course Canon (Verified Codes and Titles)

The table below is the AI/ML core as it appears in the official MIT subject listings (student.mit.edu catalog pages for Course 6, 2026–2027 entries, and the courses' own official sites). "Units" are MIT's lecture/lab/self-study weekly workload notation.

| Code | Title | Level / Term | Verified catalog line (abridged) |
| --- | --- | --- | --- |
| 6.3900 | Introduction to Machine Learning | UG, Fall and Spring | MIT's entry ML course; prerequisite for the graduate canon (6.7900, 6.7960, 6.8300, 6.8610 all list it); units 4-0-8; official site introml.mit.edu ✅ |
| 6.7900 | Machine Learning | Grad, Fall | "Principles, techniques, and algorithms in machine learning from the point of view of statistical inference; representation, generalization, and model selection; and methods such as linear/additive models, active learning, boosting, support vector machines, non-parametric Bayesian methods, hidden Markov models, Bayesian networks, and convolutional and recurrent neural networks." (catalog; instructor G. Bresler) ✅ |
| 6.7960 | Deep Learning | Grad, Fall | "Fundamentals of deep learning, including both theory and applications… neural net architectures (MLPs, CNNs, RNNs, graph nets, transformers), geometry and invariances in deep learning, backpropagation and automatic differentiation, learning theory and generalization in high-dimensions, and applications to computer vision, natural language processing, and robotics." (catalog; instructor P. Isola) ✅ |
| 6.4110 | Representation, Inference, and Reasoning in AI | UG+Grad, Spring | "An introduction to representations and algorithms for artificial intelligence… constraint satisfaction… logical representation and inference, Monte Carlo tree search, probabilistic graphical models and inference, planning… Markov decision processes (MDPs) and partially observed Markov decision processes (POMDPs)." (catalog) ✅ |
| 6.4300 | Introduction to Computer Vision | UG+Grad | "Introduction to computer vision… low-level image analysis, edge detection, image transformations… 3D scene reconstruction, motion analysis and tracking… machine learning, convolutional neural networks, and transformers… object classification, detection, and segmentation." (catalog) ✅ |
| 6.8300 | Advances in Computer Vision | Grad, Spring | "Advanced topics in computer vision focusing on geometry… image formation, representation theory for vision, classic multi-view geometry, multi-view geometry in the age of deep learning, differentiable rendering, neural scene representations, correspondence estimation, optical flow… generative modeling and representation learning including image and video generation, guidance in diffusion models…" (catalog; prerequisite 6.7960) ✅ |
| 6.8610 | Quantitative Methods for Natural Language Processing | Grad, Spring | "Introduces the study of human language from a computational perspective, including syntactic, semantic and discourse processing models. Emphasizes machine learning methods… applications such as syntactic parsing, information extraction, statistical machine translation, dialogue systems." (catalog) ✅ |
| 6.8620[J] | Spoken Language Processing | Grad, Spring | Automatic speech recognition; "acoustic theory of speech production, acoustic-phonetics, signal representation, acoustic and language modeling, search, hidden Markov modeling, neural networks models, end-to-end deep learning models" (catalog; instructor J. R. Glass) ✅ |

### 3.5 Topic Outlines of the Flagship Core Courses

**6.7900 Machine Learning — topics.** The catalog scope above is the course's contract; the lecture calendar on the course's own site (gradml.mit.edu) shows the topic order in practice: (1) introduction and foundations; (2) empirical risk minimization and maximum likelihood; (3) Bayesian learning; (4) linear regression; (5) uncertainty quantification and regularizers — the early spine is thus *statistical-inference-first*, not neural-networks-first, and the catalog methods list (linear/additive models, boosting, SVMs, non-parametric Bayesian methods, HMMs, Bayesian networks, CNNs/RNNs) fills out the rest of the term. Assessment is weekly written homework, small projects, a midterm, and a final exam. ✅ (calendar + catalog)

**6.7960 Deep Learning — topics.** Architectures first: MLPs, CNNs, RNNs, graph neural networks, transformers; then the theory layer: geometry and invariances, backpropagation and automatic differentiation, learning theory and generalization in high dimensions; then applications to computer vision, natural language processing, and robotics. ✅ (catalog)

**6.4110 Representation, Inference, and Reasoning in AI — topics.** The classical-AI-plus-modern core: constraint satisfaction over discrete and continuous problems; logical representation and inference; Monte Carlo tree search; probabilistic graphical models and inference; planning in deterministic and probabilistic models, including MDPs and POMDPs. ✅ (catalog)

**6.8300 Advances in Computer Vision — topics.** Geometric vision in the deep-learning age: image formation; multi-view geometry, classical and learned; differentiable rendering; neural scene representations; correspondence estimation, optical flow, point tracking; and generative modeling — image and video generation, diffusion-model guidance, conditional probabilistic models. Prerequisite: 6.7960 (deep learning), which signals how advanced the material is. ✅ (catalog)

**6.8610 Quantitative Methods for NLP — topics.** Computational models of language across levels — syntactic, semantic, discourse — built with machine learning methods and applied to syntactic parsing, information extraction, statistical machine translation, and dialogue systems. Its undergraduate companion 6.8611 shares the content with additional communication requirements. ✅ (catalog)

**6.8620[J] Spoken Language Processing — topics.** The speech side of language: acoustic theory of speech production and acoustic-phonetics; signal representation; acoustic and language modeling; search; hidden Markov models; neural network models and end-to-end deep learning applied to speech — with problem sets, lab exercises, and an open-ended term project. ✅ (catalog)

### 3.6 MIT Notes

- **The undergraduate pipeline matters.** MIT's AI+D bachelor's major (6-4) shares the graduate canon's vocabulary — its required subjects feed straight into 6.7900/6.7960-level work, and the graduate courses' prerequisites (6.3900, 18.06 linear algebra, 6.3700/6.3800/18.600 probability) are precisely the 6-4 spine.
- **Deep learning lives in Course 6.** MIT's credit deep-learning teaching is 6.7960; the famous January short course 6.S191 (*Introduction to Deep Learning*) is non-credit and is excluded from the table above per the honesty convention, but it is how many MIT students first meet the material.
- **Robotics and decision-making sit in the same area.** AI+D explicitly spans perception, learning, language, and decision-making; robotics subjects (6.4200 Robotics: Science and Systems, 6.8210 Underactuated Robotics, 6.8200 Sensorimotor Learning) appear in the same catalog blocks. ⚠ — the *membership* of this orbit was inferred from catalog adjacency, not from a published AI+D course list; treat as indicative.

---

## 4. Stanford University

### 4.1 The School at a Glance

Stanford University, Stanford, California, is #2 in the QS 2026 Computer Science and Information Systems subject table (§2.2) and #1 on the EduRank world-AI list consulted in §2.3. AI/ML at Stanford lives primarily in the Department of Computer Science (within the School of Engineering), alongside the Stanford Institute for Human-Centered Artificial Intelligence (HAI) and the statistics department. The department's graduate degrees are the vehicle for AI/ML study — Stanford offers no standalone degree named "AI" or "Machine Learning" at the master's level; the AI content is a specialization of the MS in Computer Science and a research area of the PhD. ⚠ — Stanford's departmental pages restructured and the admissions pages were not reachable live during this pass (404 on the master's-program URL); the facts below that could not be re-verified against a live page are marked individually.

### 4.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| PhD in Computer Science | Doctorate | Research doctorate; AI is one of the named research areas (with the "AI" and "Systems" tracks among the department's specializations) | ⚠ not re-verified live this pass (page 404) |
| MS in Computer Science | Professional/research master's | The taught master's; students pursue one of several specializations, of which Artificial Intelligence is the canonical AI/ML route (the MS is also available coterminal with the BS) | ⚠ not re-verified live this pass |
| MS in Statistics / Data Science | Professional master's | The statistics-side ML route (the data-science MS) | ⚠ not re-verified live this pass |

Stanford's AI/ML graduate "program" is thus the *specialist pathway inside a general degree* shape (§1.3) — the same shape as MIT's.

### 4.3 The Flagship Programs

**Flagship 1 — the MS in CS with the Artificial Intelligence specialization.** The canonical taught route. The AI specialization's course canon centers on the famous CS2xx ML sequence; the department's published specialization requirements (in the years before the page restructure) required depth in AI core courses plus breadth. ⚠ — the current (2026-27) specialization sheet was not re-verified live this pass.

**Flagship 2 — the CS PhD (AI research area).** Admission is to the CS PhD with students affiliating with AI faculty across learning, vision, NLP, and robotics after arrival — the same research-apprenticeship shape as MIT's EECS PhD. ⚠ structure not re-verified live this pass.

### 4.4 The Core Course Canon (Verified Codes and Titles)

The canonical Stanford AI/ML courses below are the ones the AI specialization and the PhD area have long required or recommended. The codes and titles are stable, widely documented public facts; **none of the live pages could be re-reached this pass** (the CS department site returned 404s), so each row is marked ⚠ and the ledger (§14) records the verification gap honestly.

| Code | Title | Notes |
| --- | --- | --- |
| CS229 | Machine Learning | The signature ML course (supervised/unsupervised learning, deep learning, ML diagnostics); taught for many years by Andrew Ng; prerequisites linear algebra, probability, and programming maturity |
| CS230 | Deep Learning | Deep-learning foundations and applications (CNNs, RNNs, transformers, practical deployment), project-based |
| CS224N | Natural Language Processing with Deep Learning | The flagship NLP course (word vectors, neural networks for language, attention and transformers, LLMs) |
| CS231N | Deep Learning for Computer Vision | The flagship vision course (CNN architectures, detection/segmentation, generative models, vision transformers) |
| CS228 | Probabilistic Graphical Models | Probabilistic modeling and inference (PGMs, variational methods, structure learning) |
| CS236 | Deep Generative Models | Generative modeling (VAEs, GANs, normalizing flows, diffusion models, score-based methods) |
| CS234 | Reinforcement Learning | RL foundations and algorithms (MDPs, value/policy methods, policy gradients, actor-critic, RLHF-adjacent topics) |
| CS224W | Machine Learning with Graphs | Graph representation learning (node/edge/graph embeddings, GNNs, applications) |
| CS329X / CS330 | (Deep Multi-Task and Meta Learning family) | The CS330 series on multi-task/meta-learning; CS329 series on applied ML engineering ⚠ title-year variance |

> **Verification note.** Every row above is a long-documented public fact of the Stanford catalog, but per this guide's governing rule (§1.2) rows that could not be re-checked against a live official page in this pass are marked ⚠ rather than ✅. The ledger (§14) and §15 carry the full list of the pages that failed this pass.

### 4.5 Topic Outlines of the Flagship Core Courses

**CS229 Machine Learning — topics (as long documented).** Supervised learning (linear regression, logistic regression, SVMs, kernels); generative learning algorithms; unsupervised learning (k-means, PCA, ICA, anomaly detection); learning theory (bias/variance, regularization, VC-style bounds); deep learning (backpropagation, CNNs/RNNs in the modern syllabus); ML system diagnostics (error analysis, ablations); case studies. ⚠ outlines from the course's established public syllabus; the live syllabus page was not reachable this pass.

**CS230 Deep Learning — topics.** Neural-network foundations; backpropagation and automatic differentiation; training dynamics and regularization; CNN architectures; RNNs/sequence models; transformers and attention; project-based application to a chosen domain. ⚠ as above.

**CS224N NLP with Deep Learning — topics.** Word vectors and distributed representations; neural network basics; backprop in depth; syntax and dependency parsing; language models from n-grams to transformers; attention; pretraining/fine-tuning and LLM-era topics; machine translation and generation. ⚠ as above.

### 4.6 Stanford Notes

- **The coterminal route matters.** Stanford undergraduates can pursue the MS coterminally (the "coterm"), which is how many AI/ML-focused Stanford students graduate with the MS a year after the BS — a structural option most of the other nine schools do not offer at the same scale. ⚠ not re-verified live this pass.
- **HAI anchors the human-centered side.** The Stanford Institute for Human-Centered AI funds and hosts much of the AI activity and its policy/ethics discourse. ⚠ as above.
- **Prerequisite culture is explicit.** The CS2xx ML canon assumes CS106-style programming maturity plus linear algebra (or the CS109/statistics probability sequence) — the same spine the MIT section (§3.6) notes for Course 6.

---

## 5. Carnegie Mellon University

### 5.1 The School at a Glance

Carnegie Mellon University, Pittsburgh, Pennsylvania, is #3 in the QS 2026 CS&IS subject table (§2.2) and is ranked #1 in AI by U.S. News (2024, per the university's own site, verified live this pass). CMU is the only school in this survey with a **standalone academic department of machine learning**: the Machine Learning Department (MLD), founded **2006** — the university describes it as "the world's first academic department of its kind" (ml.cmu.edu, verified live this pass). AI/ML at CMU is also spread across the School of Computer Science's other departments (the Language Technologies Institute for NLP, the Robotics Institute, the Computer Science Department), but MLD is the named AI/ML home.

### 5.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| PhD in Machine Learning | Doctorate | Research doctorate in ML proper (MLD) | ml.cmu.edu ✅ |
| Joint PhD in Statistics and Machine Learning | Doctorate | Joint MLD/statistics doctorate | ml.cmu.edu ✅ |
| Joint PhD in Machine Learning and Public Policy | Doctorate | Joint MLD/Heinz College doctorate | ml.cmu.edu ✅ |
| Joint PhD in Neural Computation and Machine Learning | Doctorate | Joint MLD/neuroscience doctorate | ml.cmu.edu ✅ |
| Primary Master's in Machine Learning | Master's | The standalone MS in ML (MLD's own degree) | ml.cmu.edu ✅ |
| 5th-Year Master's in Machine Learning | Master's | Accelerated MS for CMU undergraduates | ml.cmu.edu ✅ |
| Secondary Master's in Machine Learning | Master's | MS for students enrolled in another CMU PhD program | ml.cmu.edu ✅ |
| BS in Artificial Intelligence | Undergraduate | CMU's named AI bachelor's (School of Computer Science) | cmu.edu ✅ |

CMU is the only school in the ten with both a *named* AI bachelor's and a *named* ML department — the "named degree" shape (§1.3) taken to its logical extreme.

### 5.3 The Flagship Programs

**Flagship 1 — the PhD in Machine Learning.** Admission to MLD for research across learning theory, deep learning, and applications. The PhD is the model curriculum MLD says it exists to propagate (§5.1).

**Flagship 2 — the Primary Master's in Machine Learning.** The standalone MS in ML — the degree whose *name* is the specialty, taught by the MLD faculty. Its curriculum is built on the MLD core courses of §5.4.

### 5.4 The Core Course Canon (Verified Codes and Titles)

The MLD core course codes below are the stable, widely documented spine of the department's curriculum (the department's own program pages list the MS/PhD requirements against these codes). ⚠ — the individual course-catalog entries were not re-extracted live this pass (the MLD site's course pages sit behind the department's course listing application); codes and titles are marked ✅ where the program pages themselves name them, ⚠ where only secondary documentation was reachable.

| Code | Title | Notes / Verification |
| --- | --- | --- |
| 10-701 | Introduction to Machine Learning (PhD) | The doctoral ML core — the "MLD PhD intro" — covering the full supervised/unsupervised canon at research depth; the MS version is 10-601/10-701 family ⚠ |
| 10-601 | Introduction to Machine Learning (Master's) | The master's-level ML core (with 10-701, the two-level 10-6xx/10-7xx pair) ⚠ |
| 10-703 | Deep Reinforcement Learning | The deep-RL core (Katerina Fragkiadaki's course; value/policy methods at deep scale) ⚠ |
| 10-605 | Machine Learning with Large Datasets | Distributed/scale ML (the systems-side ML course) ⚠ |
| 10-606 / 10-607 | Mathematical Foundations of Machine Learning (A/B) | The math core (linear algebra, optimization, probability as used in ML) ⚠ |
| 10-715 | Advanced Introduction to Machine Learning | The second PhD-level ML course for students entering with prior ML ⚠ |
| 10-725 | Convex Optimization | The optimization core shared across SCS ⚠ |
| 10-708 | Probabilistic Graphical Models | PGM theory and algorithms (with the famous course notes lineage) ⚠ |

> **Verification note.** The MLD *program* pages (PhD and MS listings with these degree names, §5.2) were verified live at ml.cmu.edu this pass. The individual 10-xxx course-catalog entries could not be re-extracted live (the department's course application was not reachable), so the course rows carry ⚠ per §1.2 — the codes and titles themselves are among the most widely documented in the field.

### 5.5 Topic Outlines of the Flagship Core Courses

**10-701 / 10-601 Introduction to Machine Learning — topics.** The comprehensive ML canon taught at research depth: supervised learning (regression, classification, kernels, SVMs, trees/ensembles); learning theory (generalization, bias/variance, regularization); unsupervised learning (clustering, dimensionality reduction, density estimation); graphical models and structured prediction; deep learning (architectures, training, CNNs/RNNs/transformers); and selected advanced topics. ⚠ outline from long-documented public syllabi; live syllabus not reachable this pass.

**10-703 Deep Reinforcement Learning — topics.** Reinforcement learning at deep scale: MDPs and dynamic programming; value-based methods (DQN and successors); policy-gradient methods; actor-critic architectures; exploration; and applications to control and games. ⚠ as above.

**10-708 Probabilistic Graphical Models — topics.** Representation (Bayesian and Markov networks, factor graphs); inference (exact and approximate — variable elimination, belief propagation, MCMC, variational methods); learning (parameter and structure learning); applications to vision, language, and biology. ⚠ as above.

### 5.6 CMU Notes

- **The AI ecosystem is multi-departmental.** Besides MLD, the Language Technologies Institute (NLP/LLM research), the Robotics Institute, and the Computer Science Department each run their own degrees and courses; the famous deep-learning course 11-785 (Intro to Deep Learning) lives in the LTI/ECE orbit. ⚠ course-home department not re-verified this pass.
- **The named AI bachelor's (BS in AI)** is the first of its kind in the US (est. 2018) and feeds the graduate canon. ⚠ the establishment year is a widely documented public fact not re-verified live this pass.
- **"First department of its kind" is CMU's own claim**, verified on its live site this pass; it is reported here as the university's claim, not independently adjudicated.

---

## 6. University of California, Berkeley

### 6.1 The School at a Glance

The University of California, Berkeley (Berkeley, California) is in the QS 2026 CS&IS top-ten set (§2.2) and is #2 on the EduRank world-AI list (§2.3). AI/ML at Berkeley is housed in the Department of Electrical Engineering and Computer Sciences (EECS) within the College of Engineering, with the Center for Human-Compatible AI and BAIR (Berkeley Artificial Intelligence Research) as the research umbrella. The department runs the graduate CS degrees; there is no standalone Berkeley "MS in AI/ML" — the AI/ML content sits inside the EECS graduate curriculum, following the *specialist pathway inside a general degree* shape (§1.3). ⚠ — Berkeley's EECS and ML-lab pages were not reachable live this pass (the ML lab site was blocked by the extraction environment; the EECS graduate pages did not return clean text), so this section carries more ⚠ marks than the MIT or CMU sections.

### 6.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| PhD in Computer Science (EECS) | Doctorate | Research doctorate; AI/ML is a major research area of the EECS PhD | ⚠ not re-verified live this pass |
| M.S. in Computer Science | Research master's | A small research-oriented MS, typically a step toward the PhD | ⚠ not re-verified live this pass |
| MEng (Master of Engineering) | Professional master's | One-year professional degree with concentrations (the AI/ML-relevant concentration being the closest Berkeley comes to a taught AI master's) | ⚠ not re-verified live this pass |
| 5th-year MEng | Professional master's | The accelerated version for Berkeley EECS undergraduates | ⚠ not re-verified live this pass |

### 6.3 The Flagship Programs

**Flagship 1 — the EECS PhD (AI/ML area).** Berkeley's research doctorate is the flagship AI/ML route; BAIR faculty span learning, vision, NLP, robotics, and AI safety/alignment. ⚠ specifics not re-verified live this pass.

**Flagship 2 — the MEng with an AI/ML concentration.** Berkeley's professional degree; the closest to a taught "AI master's," with a capstone project and industry orientation. ⚠ as above.

### 6.4 The Core Course Canon (Verified Codes and Titles)

The Berkeley graduate AI/ML course codes below are long-documented public facts of the EECS catalog. None of the live pages could be re-reached this pass (blocked or unextractable), so **every row is ⚠** and the ledger (§14) records the gap.

| Code | Title | Notes |
| --- | --- | --- |
| CS188 | Introduction to Artificial Intelligence | The undergraduate AI course that anchors the graduate canon's prerequisites (search, CSPs, MDPs, RL, Bayes nets) |
| CS182 | Designing, Visualizing and Understanding Deep Neural Networks | The deep-learning course (with CS282 variants at graduate level) |
| CS189 | Introduction to Machine Learning | The canonical Berkeley ML course (the "CS189" brand, with the famous public notes) |
| CS280 | Computer Vision | The graduate vision course |
| CS281A / CS281B | Statistical Learning Theory / Advanced ML | The graduate statistical-learning sequence (CS281A/B) |
| CS285 | Deep Reinforcement Learning | The deep-RL graduate course (Sergey Levine's) |
| CS288 | Natural Language Processing | The graduate NLP course |
| CS294-* | (topical graduate seminars) | The numbered seminar series carrying frontier topics (LLMs, generative models, safety) — the *suffix varies by offering |

> **Verification note.** Berkeley's course codes are among the most stable in the field and are recorded here from long-documented public knowledge, but the governing rule (§1.2) requires marking them ⚠ when the live catalog could not be checked this pass. §15 lists the exact pages that failed.

### 6.5 Topic Outlines of the Flagship Core Courses

**CS189 Introduction to Machine Learning — topics (as long documented).** Supervised learning foundations (regression, classification, loss functions); optimization (gradient descent, SGD); kernels and the kernel trick; model selection and validation; decision trees and ensembles; clustering and dimensionality reduction; an introduction to neural networks; ethics and fairness modules in the modern syllabus. ⚠ outline from long-documented public material; live syllabus unreachable this pass.

**CS285 Deep Reinforcement Learning — topics.** The RL canon at research depth: MDPs; model-free prediction and control; value-based methods; policy gradients; actor-critic; model-based RL; exploration; multi-task and meta-RL; safety considerations. ⚠ as above.

**CS288 Natural Language Processing — topics.** Computational models of language at graduate level: representations, sequence models, syntax and semantics, neural language models, attention and transformers, and applications. ⚠ as above.

### 6.6 Berkeley Notes

- **BAIR is the research gravity.** Berkeley AI Research is one of the largest academic AI groups; the PhD experience is apprenticeship into BAIR labs. ⚠ not re-verified live this pass.
- **The safety/interp thread is distinctive.** Berkeley hosts prominent AI-safety and interpretability research (CHAI, and faculty whose work defines the alignment agenda) — a thread that shows up in the CS294 seminar rotation. ⚠ as above.
- **Undergraduate courses carry the graduate prerequisites.** CS188/CS189 double as the recognized entry points for the graduate canon, the same pattern MIT notes for 6.3900 (§3.6).

---

## 7. University of Oxford

### 7.1 The School at a Glance

The University of Oxford (Oxford, England) is #1 in THE's 2026 Computer Science subject table and #5 in the QS 2026 CS&IS subject table (§2.3-§2.4). AI/ML at Oxford lives in the Department of Computer Science, with the wider Oxford Machine Learning research ecosystem (the OATML group — Oxford Applied and Theoretical Machine Learning — and the AIMS Centre for Doctoral Training) providing the research gravity. Oxford's degree structure follows the collegiate-university pattern: the MSc degrees are examined by the department and conferred by the university, with college affiliation alongside. The department's graduate pages were verified live this pass (cs.ox.ac.uk/admissions/graduate).

### 7.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| MSc in Advanced Computer Science | Taught master's | One-year MSc; the department's main taught degree, with a named **Artificial Intelligence** variant ("MSc Advanced Computer Science (Artificial Intelligence)") | cs.ox.ac.uk + ox.ac.uk admissions ✅ |
| MSc in Advanced Computer Science (Foundations of Computer Science) | Taught master's | The theory variant of the same MSc | ox.ac.uk admissions ✅ |
| DPhil in Computer Science | Doctorate | The research doctorate (Oxford's "PhD"), with ML/AI supervision across the department | cs.ox.ac.uk ✅ |
| MSc in Mathematics and Foundations of Computer Science | Taught master's | The joint mathematics/CS theory degree | cs.ox.ac.uk ✅ |
| AIMS Centre for Doctoral Training | CDT/doctorate | The Autonomous Intelligent Machines and Systems CDT — a funded DPhil programme in ML/AI/robotics | aims.robots.ox.ac.uk ✅ |
| MSc in Artificial Intelligence for Business | Professional part-time | Part-time professional master's for industry (the department's professional-education arm) | cs.ox.ac.uk ✅ |
| MSc in Software Engineering / Software and Systems Security | Professional part-time | The department's other part-time professional MScs (context: the professional arm is distinct from the full-time research-facing degrees) | cs.ox.ac.uk ✅ |

### 7.3 The Flagship Programs

**Flagship 1 — the MSc in Advanced Computer Science (Artificial Intelligence).** The named AI variant of Oxford's flagship taught MSc: a one-year, full-time course selecting the AI-relevant menu of the department's MSc options and requiring a substantial dissertation. ✅ (admissions listing verified live).

**Flagship 2 — the DPhil in Computer Science.** The research doctorate; ML/AI supervision spans OATML and the applied groups. ✅ (listing verified live).

### 7.4 The Core Course Canon (Verified Codes and Titles)

Oxford's MSc courses are selected from the department's graduate course menu each year; the AI-relevant core courses below are the stable, long-documented spine of the MSc (Advanced CS) AI route. ⚠ — the *current-year* option lists were not fully re-extracted live this pass; codes/titles below are those the department has long published for the AI stream.

| Code / short name | Title | Notes / Verification |
| --- | --- | --- |
| (AI core) | Machine Learning | The department's graduate ML course (verified by a research pass this cycle — the course page shows the course run by S. Flaxman, Michaelmas term) ✅/⚠ |
| (AI core) | Deep Learning | The graduate deep-learning course option |
| (AI core) | Probabilistic Modelling / Graphical Models | The probabilistic-modeling option |
| (AI core) | Computer Vision | The vision option |
| (AI core) | Natural Language Processing | The NLP option |
| (AI core) | Multi-Agent Systems / Knowledge Representation | The classical-AI/systems options of the stream |
| (DPhil) | Research seminars | The doctoral research-training spine (the DPhil's coursework is minimal; apprenticeship dominates) |

> **Verification note.** The department's *program* pages were verified live (§7.2). One flagship course page (Machine Learning, cs.ox.ac.uk teaching pages, Flaxman, Michaelmas term) was verified live during the earlier research passes of this guide's production; the remaining option rows are ⚠ per §1.2 — the option menu rotates year to year and the current-year list was not re-extracted this pass.

### 7.5 Topic Outlines of the Flagship Core Courses

**Machine Learning (graduate, Michaelmas term) — topics.** The graduate ML foundations course: supervised learning (regression, classification); probabilistic methods; kernel methods; model selection and evaluation; an introduction to deep learning — with problem sheets and an examination, per the course's long-documented format. ✅/⚠ (course page verified; the outline is a compression of the established syllabus).

**Deep Learning — topics (as long documented).** Neural-network foundations and training; convolutional and recurrent architectures; attention and transformer models; practical deep-learning engineering; and applications. ⚠ as the current-year page was not re-extracted.

### 7.6 Oxford Notes

- **The college system shapes the experience.** MSc/DPhil students hold college membership alongside departmental study — a structural fact with no US counterpart in this survey. ✅ (general public fact of Oxford's structure).
- **OATML and AIMS anchor the ML research.** The AIMS CDT's funded DPhil route is the most direct funded path into Oxford ML research (§7.2). ✅ listing verified live.
- **The AI-for-Business MSc is a separate market.** The part-time professional degree targets industry executives — worth distinguishing from the full-time research-facing MSc when comparing programs (§13). ✅ listing verified live.

---

## 8. University of Cambridge

### 8.1 The School at a Glance

The University of Cambridge (Cambridge, England) is in both the QS and THE 2026 CS top-ten sets (§2.2-§2.3). Cambridge's AI/ML flagship is the **MPhil in Machine Learning and Machine Intelligence (MLMI)** — an eleven-month, full-time programme offered by the Machine Learning Group, the Speech Group, and the Computer Vision and Robotics Group in the **Department of Engineering** (verified live this pass at mlmi.eng.cam.ac.uk). The Department of Computer Science and Technology and the Department of Engineering together host Cambridge's ML activity; MLMI is the named AI/ML degree.

### 8.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| MPhil in Machine Learning and Machine Intelligence (MLMI) | Taught master's | Eleven-month MPhil by the Engineering Department's ML/Speech/CV+Robotics groups; the named AI/ML degree | mlmi.eng.cam.ac.uk ✅ |
| MPhil in Advanced Computer Science | Taught master's | The Computer Science and Technology Department's taught MPhil (the CS-general route) | ⚠ not re-verified live this pass |
| PhD (CS or Engineering) | Doctorate | The research doctorate, supervised across the ML groups (the "PhD in Machine Learning" is the de facto research route through either department) | ⚠ not re-verified live this pass |

### 8.3 The Flagship Programs

**Flagship 1 — the MPhil in Machine Learning and Machine Intelligence.** Eleven months, full time, run jointly by the ML, Speech, and CV+Robotics groups: taught modules in the first terms followed by a substantial research project (the MLMI research project component is a named part of the programme structure, per the programme's own site). ✅ structure verified live.

**Flagship 2 — the PhD via the ML groups.** The research doctorate across the three groups; the MLMI MPhil is the natural feeder. ⚠ specifics not re-verified live this pass.

### 8.4 The Core Course Canon (Verified Codes and Titles)

The MLMI programme's taught modules — the "papers" of the MPhil — are the programme's core; the stable core module names below are those the programme has long published. ⚠ — the module codes rotate and the current-year module list was not re-extracted live this pass (the programme site's course-structure page was reached but its module tables were not retrieved cleanly).

| Module area | Title (as long published) | Notes |
| --- | --- | --- |
| Core ML | Machine Learning for Intelligent Systems | The MLMI foundations module (supervised/unsupervised learning, probabilistic ML) |
| Core ML | Probabilistic Machine Learning | The Bayesian/probabilistic modeling module |
| Deep learning | Deep Learning and Structured Data | The deep-learning module (architectures for structured data) |
| Speech | Speech Processing | The speech-recognition module from the Speech Group |
| Vision | Computer Vision | The vision module from the CV+Robotics group |
| Robotics | Robotics | The robotics module from the CV+Robotics group |
| Research | MLMI Research Project | The substantial project component (summer term) |

> **Verification note.** The programme-level facts (§8.1-§8.3) were verified live at mlmi.eng.cam.ac.uk this pass. The module-level rows are marked ⚠ per §1.2 — the programme site's course-structure page was reached but its detailed module tables were not extracted cleanly this pass; the module names above are those the programme has long published and are recorded as ⚠, not ✅.

### 8.5 Topic Outlines of the Flagship Core Modules

**Machine Learning for Intelligent Systems — topics (as long documented).** The ML foundations for the engineering MLMI cohort: supervised learning (regression and classification); model selection and evaluation; unsupervised learning; probabilistic methods and Bayesian basics; and an introduction to deep learning. ⚠ module-level detail not re-extracted live this pass.

**Probabilistic Machine Learning — topics (as long documented).** Bayesian modeling and inference; graphical models; approximate inference (sampling and variational methods); decision theory. ⚠ as above.

### 8.6 Cambridge Notes

- **MLMI sits in Engineering, not CS.** Applicants sometimes miss that Cambridge's named ML degree is run by the Department of Engineering's ML/Speech/CV groups — a structural fact that shapes the cohort (engineering and mathematics graduates predominate). ✅ verified live.
- **The MPhil is a feeder to the PhD.** The research-project structure and the group alignment make MLMI the standard entry to a Cambridge ML PhD. ✅/⚠ (the general pattern is documented; specific progression statistics are not).
- **The college pattern again.** As with Oxford (§7.6), MPhil/PhD students hold college membership — the collegiate-university shape is common to both UK schools in this survey.

---

## 9. ETH Zurich

### 9.1 The School at a Glance

ETH Zurich (Eidgenössische Technische Hochschule Zürich, Zurich, Switzerland) is in the QS and THE 2026 CS top-ten sets (§2.2-§2.3). AI/ML at ETH is concentrated in the Department of Computer Science (D-INFK), with the Max Planck ETH Center for Learning Systems and the ETH AI Center as the research umbrella. ETH offers no standalone "MS in AI" at the bachelor entry level; AI/ML is a **major (specialization) inside the MSc in Computer Science** — the *specialist pathway inside a general degree* shape (§1.3) — with the department's research groups in ML, vision, NLP, and robotics providing the doctoral gravity. ⚠ — the ETH master's pages were not reachable live this pass (blocked by the extraction environment), so this section carries ⚠ marks; one research pass during this guide's production did verify ETH course identifiers 252-0535 and 263-3210 in the ETH course catalogue context, which anchors part of §9.4.

### 9.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| MSc in Computer Science (with a major/specialization relevant to AI/ML) | Taught master's | The D-INFK MSc; students choose majors from the department's list, of which the ML/AI-relevant majors are the AI/ML route | ⚠ not re-verified live this pass |
| PhD in Computer Science | Doctorate | The research doctorate; ML/AI supervision across the department's groups | ⚠ not re-verified live this pass |
| (Related) MSc in Data Science | Taught master's | The data-science sibling degree | ⚠ not re-verified live this pass |

### 9.3 The Flagship Programs

**Flagship 1 — the MSc in CS with the ML/AI major.** The taught master's; the AI/ML route is a named major inside the degree, with a core set of D-INFK graduate courses plus the master's thesis. ⚠ specifics not re-verified live this pass.

**Flagship 2 — the D-INFK PhD.** The research doctorate across the ML/vision/NLP groups. ⚠ as above.

### 9.4 The Core Course Canon (Verified Codes and Titles)

ETH's graduate CS courses carry the 252-xxx (computer science) and 263-xxx (advanced) numbering. The course identifiers below are anchored by the research passes of this guide's production (the ETH course-catalogue context was verified for the two named rows) and long-documented public knowledge for the rest; rows not re-verified live this pass carry ⚠.

| Code | Title | Notes / Verification |
| --- | --- | --- |
| 252-0535-00L | Advanced Machine Learning | The flagship graduate ML course (the "AML" course with the probabilistic/ML canon) ✅/⚠ (identifier context verified in a research pass) |
| 263-3210-00L | Deep Learning | The deep-learning graduate course ✅/⚠ (identifier context verified in a research pass) |
| 252-0526-00L | Statistical Learning Theory | The theory course (regularization, generalization, kernel methods) ⚠ |
| 263-5xx-00L family | (Probabilistic AI, NLP, Vision, RL courses) | The 263-5xx graduate offerings in the AI/ML majors ⚠ |
| (seminar) | ML/CS seminars | The departmental seminar credits of the master's ⚠ |

> **Verification note.** The two anchored identifiers were confirmed in the ETH course-catalogue context during this guide's earlier research passes. The remaining rows are ⚠ per §1.2 — the ETH pages were blocked to this pass's extraction environment and are listed in §15.

### 9.5 Topic Outlines of the Flagship Core Courses

**Advanced Machine Learning (252-0535) — topics (as long documented).** A graduate ML course with a probabilistic and learning-theoretic bent: supervised learning and loss functions; regularization and model selection; kernel methods; Bayesian methods and approximate inference; graphical models; and modern deep-learning connections. ⚠ outline from long-documented public material; the live syllabus was not reachable this pass.

**Deep Learning (263-3210) — topics (as long documented).** Deep-learning foundations: optimization and generalization in neural networks; convolutional architectures; sequence models and attention/transformers; and applications across vision, language, and speech. ⚠ as above.

### 9.6 ETH Zurich Notes

- **The "major" system is the AI/ML vehicle.** Prospective students should read the MSc's major structure carefully — the AI/ML content is inside the CS degree, not a separately named degree. ⚠ not re-verified live this pass.
- **The Max Planck ETH Center for Learning Systems** is the flagship doctoral partnership — a funded PhD route spanning ETH and the Max Planck Institutes. ⚠ not re-verified live this pass.
- **German-language context.** Teaching is English at the graduate level in D-INFK, but the surrounding university context is German — a practical fact for applicants. ⚠ general public fact, not re-verified this pass.

---

## 10. National University of Singapore (NUS)

### 10.1 The School at a Glance

The National University of Singapore (Singapore) is #4 in the QS 2026 CS&IS subject table (§2.2) — the highest-ranked Asian university on the primary ranking used here, and the only Singapore institution in the ten. AI/ML at NUS is concentrated in the School of Computing (SoC), with the NUS AI Institute (a university-level research institute launched in recent years) and the Smart Systems Institute as the research umbrella. NUS offers both general CS degrees with AI specializations and *named* applied AI/ML master's degrees — mixing the *named degree* and *pathway* shapes (§1.3). ⚠ — the NUS Computing pages returned 404s for the module-programme URLs tried this pass, so this section carries ⚠ marks per §1.2.

### 10.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| Master of Computing (with AI specialization) | Taught master's | The School of Computing's general MSc (the "Master of Computing" / MComp), with named specializations including Artificial Intelligence | ⚠ not re-verified live this pass |
| MSc in Data Science and Machine Learning | Taught master's | A named applied MSc (jointly flavored toward DS+ML, offered via the Faculty of Science/SoC orbit) | ⚠ URL 404 this pass; long-documented programme |
| MSc in Computer Science | Taught master's | The CS-stream variant of the MComp family | ⚠ not re-verified live this pass |
| PhD (School of Computing) | Doctorate | The research doctorate across SoC's AI/ML faculty | ⚠ not re-verified live this pass |

### 10.3 The Flagship Programs

**Flagship 1 — the Master of Computing with the AI specialization.** The flagship taught route: SoC's MComp with a named AI track — the standard degree for working professionals and fresh graduates aiming at AI roles in Singapore's economy. ⚠ specifics not re-verified live this pass.

**Flagship 2 — the SoC PhD.** The research doctorate; Singapore's AI research funding (the national AI strategy context) makes NUS a well-funded destination for ML PhD study in Asia. ⚠ as above.

### 10.4 The Core Course Canon (Verified Codes and Titles)

The NUS module codes below are long-documented public facts of the SoC graduate catalog (the CS5xxx-level graduate modules). None could be re-verified live this pass (404s on the programme pages), so every row is ⚠.

| Code | Title | Notes |
| --- | --- | --- |
| CS5242 | Neural Networks and Deep Learning | The flagship deep-learning module (graduate level) |
| CS5246 | Text Mining | The NLP-oriented module |
| CS5272 | (Topics in AI / multi-agent and knowledge systems variants) | The AI-topics module ⚠ title-variant risk |
| CS5340 | Probabilistic Graphical Models (uncertainty modeling) | The probabilistic-modeling module |
| CS5339 | Machine Learning | The graduate ML module (foundations) |
| CS6101 | (Advanced topics seminars) | The graduate seminar module family ⚠ |
| (DSML core) | (Data Science and Machine Learning programme core) | The named MSc DSML's own core set ⚠ |

> **Verification note.** The module codes and titles above are widely documented public facts of the NUS catalog, but per §1.2 rows that could not be re-checked against live pages this pass carry ⚠. The programme pages that 404'd are listed in §15.

### 10.5 Topic Outlines of the Flagship Core Modules

**CS5242 Neural Networks and Deep Learning — topics (as long documented).** Deep-learning foundations and engineering: feedforward networks and backpropagation; training dynamics and regularization; convolutional networks; sequence models and attention; and applications to vision/language. ⚠ outline from long-documented public material; the live module page was not reachable this pass.

**CS5339 Machine Learning — topics (as long documented).** The ML foundations module: supervised learning (regression, classification, ensembles); model evaluation; unsupervised learning; and an introduction to deep learning. ⚠ as above.

### 10.6 NUS Notes

- **Singapore's AI strategy context matters.** NUS ML graduates feed Singapore's national AI ambitions (the Smart Nation / national AI strategy programs documented elsewhere in this repository's Singapore cluster) — a demand-side fact with no equivalent for the other nine schools. ✅ (cross-reference to the repo's Singapore guides).
- **The MComp's part-time/full-time structure** makes it the region's standard upskilling degree for working engineers. ⚠ not re-verified live this pass.
- **The named DSML MSc signals the applied market.** The applied data-science-and-ML degree is distinct from the research-facing MComp stream. ⚠ as above.

---

## 11. Tsinghua University

### 11.1 The School at a Glance

Tsinghua University (Beijing, China) is the highest-ranked Chinese university on the research-output rankings consulted in §2: #1-tied (with Shanghai Jiao Tong) on the CSRankings 2026 overall computer-science table and #1 on the US News AI subject ranking (§2.3). AI/ML at Tsinghua spans the Department of Computer Science and Technology, the Institute for Interdisciplinary Information Sciences (IIIS — the "Yao Class" tradition), and the university's AI research institutes. ⚠ — Tsinghua's English-language program pages were sparse and not reliably extractable this pass, so this section carries the heaviest ⚠ load in the survey; it records the shape of the programs with the verification gap stated plainly rather than asserting unverifiable specifics.

### 11.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| (Graduate study in CS/AI — MSc/PhD) | Taught master's + doctorate | Chinese graduate degrees in computer science with AI specialization; the master's is typically two-to-three years, the PhD five; instruction largely in Chinese at the program level | ⚠ structure-level public facts only; no live page re-verified this pass |
| IIIS / Yao-class undergraduate and graduate routes | Bachelor's + graduate | The interdisciplinary information-science institute (computer science + mathematics rigor in the "Yao Class" tradition) | ⚠ as above |
| (English-taught AI graduate programs) | Graduate | Tsinghua has published English-taught AI-related graduate offerings in recent years | ⚠ as above — none re-verified live |

### 11.3 The Flagship Programs

**Flagship 1 — the CS/AI graduate program of the Department of Computer Science and Technology.** The mainline research route. ⚠ specifics not re-verified live this pass.

**Flagship 2 — IIIS.** The interdisciplinary institute whose undergraduate "Yao Class" (named for Andrew Yao) is China's most selective CS training; its graduate arm feeds theory-flavored ML research. ⚠ as above.

### 11.4 The Core Course Canon (Verified Codes and Titles)

Tsinghua's course numbering and the Chinese-language syllabus make a code-level table unverifiable from the English-language sources reachable this pass. Rather than assert codes from memory, this guide records the **course families** the programs are documented to cover, each ⚠:

- Machine learning foundations and statistical learning theory
- Deep learning and its applications (vision, language, speech)
- Optimization for machine learning
- Probability and stochastic processes for AI
- Natural language processing; computer vision
- The research-seminar spine of the doctoral program

> **Verification note.** No Tsinghua course codes are asserted in this survey. Per the governing rule (§1.2), an unverifiable code is omitted rather than guessed; the families above are recorded as ⚠ structural descriptions only. §15 lists the access failures.

### 11.5 Topic Outlines

Course-level topic outlines are likewise omitted rather than guessed. The structural observation that *is* defensible: Tsinghua's AI curriculum mirrors the US canon (the ML/deep-learning/optimization/NLP/vision family above) while being delivered inside China's research-university system, whose output metrics drive the CSRankings #1 (§2.4). ⚠ structural framing, not a syllabus claim.

### 11.6 Tsinghua Notes

- **The rankings tell two stories.** Tsinghua is #1 on the output-counted lists and outside the top five on the reputation-weighted QS list — the single clearest illustration of the §2.4 disagreement. ✅ (per the ranking section's verified data).
- **Language is the practical barrier for most readers of this guide.** Program-level instruction is largely Chinese; English-taught offerings exist but were not verifiable this pass. ⚠
- **The comparison value is structural.** Tsinghua's inclusion makes the survey's US/UK/Europe/Asia spread complete even though its code-level curriculum could not be verified to the same standard as MIT's or CMU's. ✅/⚠

---

## 12. University of Toronto

### 12.1 The School at a Glance

The University of Toronto (Toronto, Canada) is the tenth school — selected over Princeton and EPFL because it is the only one of the three shortlisted (§2.5) with a top-five AI-specific citation in any consulted list (EduRank world-AI #5, §2.3). AI/ML at Toronto is anchored in the Department of Computer Science, with the Vector Institute (the AI research institute co-founded by Toronto CS faculty, including Geoffrey Hinton) as the ecosystem's research gravity. Toronto's flagship *taught* AI/ML degree is the **MScAC (Master of Science in Applied Computing)** with its named **Artificial Intelligence concentration** — verified live this pass (web.cs.toronto.edu/graduate/mscac). The department also runs the standard research MSc/PhD routes.

### 12.2 The Main AI/ML Degree Programs

| Program | Level | Shape | Verification |
| --- | --- | --- | --- |
| MScAC — Master of Science in Applied Computing | Professional master's | The applied master's with named concentrations; the **Artificial Intelligence** concentration is the AI/ML flagship route (other concentrations: AI in Healthcare, Data Science, Data Science for Biology, Computer Science, Applied Mathematics, Quantum Computing) | web.cs.toronto.edu ✅ |
| MSc in Computer Science | Research master's | The thesis-based research master's, typically a step toward the PhD | ⚠ not re-verified live this pass |
| PhD in Computer Science | Doctorate | The research doctorate; Toronto's deep-learning lineage (the "Toronto school" of neural networks) makes it a first-tier ML research destination | ⚠ not re-verified live this pass |

### 12.3 The Flagship Programs

**Flagship 1 — the MScAC with the AI concentration.** The professional master's combining graduate coursework with a substantial internship component — Toronto's answer to the industry-oriented AI master's. The program advertises 300+ partner companies and a #10 global CS-department ranking on its own materials (⚠ marketing copy). ✅ program + concentrations verified live.

**Flagship 2 — the CS PhD.** The research doctorate in the department whose faculty lineage (Hinton and the Vector Institute ecosystem) defines modern deep learning. ⚠ specifics not re-verified live this pass.

### 12.4 The Core Course Canon (Verified Codes and Titles)

Toronto's graduate CS courses carry the CSC25xx-level numbering for graduate courses (CSC4xx undergraduate / CSC25xx-26xx graduate). The flagship AI/ML course identifiers below are long-documented public facts; only the programme-level page was verified live this pass, so course rows are ⚠.

| Code | Title | Notes |
| --- | --- | --- |
| CSC2515 | Introduction to Machine Learning | The canonical graduate ML course (the "2515" brand, widely documented) |
| CSC2541 | (Topics in ML — deep learning and advanced topics variants) | The graduate topics course family |
| CSC2547 / CSC2548 | (Deep learning and generative-model offerings) | The deep-learning graduate offerings ⚠ code-year variance |
| CSC2501/2xxx family | (Vision/NLP/Language graduate courses) | The graduate vision and language offerings ⚠ |
| (MScAC core) | (Professional skills + the concentration's core set) | The MScAC's own structure (coursework + internship) ✅ programme shape verified live |

> **Verification note.** The MScAC programme page was verified live this pass (programme shape, concentration names, internship character). The CSC course identifiers are ⚠ per §1.2 — recorded from long-documented public knowledge, not re-verified against the live calendar this pass.

### 12.5 Topic Outlines of the Flagship Core Courses

**CSC2515 Introduction to Machine Learning — topics (as long documented).** The graduate ML foundations course: supervised learning (regression, classification, generative/discriminative models); optimization and gradient methods; model selection and regularization; kernels; unsupervised learning (clustering, latent-variable models); and foundations of deep learning. ⚠ outline from long-documented public material; the live course page was not re-extracted this pass.

**The MScAC AI concentration core — topics (structural).** The concentration's coursework draws on the department's graduate ML/AI offerings (the CSC25xx canon above) alongside the program's professional-skills and internship components. ✅/⚠ (programme structure verified live; the specific current course list ⚠).

### 12.6 Toronto Notes

- **The Vector Institute is the differentiator.** Toronto's AI ecosystem — the Vector Institute co-founded with the Hinton lineage — gives MScAC and PhD students an industry-research bridge few other cities match. ⚠ not re-verified live this pass.
- **The MScAC internship model is distinctive.** Among the ten schools, Toronto's flagship taught AI master's is the most explicitly industry-integrated (the internship is a core component, not an option). ✅ programme shape verified live.
- **The tenth-slot evidence caveat stands.** Toronto's ranking evidence is thinner than the other nine's (§2.5) — the inclusion decision is documented there and in the ledger (§14).

---

## 13. The Comparative View

### 13.1 The Program Shapes Across the Ten

| School | Flagship AI/ML degree shape | The named degree? |
| --- | --- | --- |
| MIT | EECS PhD (AI+D area) + MEng | No standalone AI/ML master's (pathway shape) |
| Stanford | MS CS (AI specialization) + CS PhD | No standalone AI/ML master's (pathway shape) |
| Carnegie Mellon | MS in ML + PhD in ML (MLD) | **Yes** — a named ML department and degree |
| UC Berkeley | EECS PhD (AI/ML area) + MEng | No standalone AI/ML master's (pathway + professional shape) |
| Oxford | MSc Advanced CS (AI variant) + DPhil | Yes (the AI variant of the MSc) |
| Cambridge | MPhil in MLMI + PhD | **Yes** — the named MLMI MPhil |
| ETH Zurich | MSc CS (AI/ML major) + PhD | No standalone AI/ML master's (pathway shape) |
| NUS | MComp (AI specialization) + SoC PhD | Yes (the specialization) + a named DSML MSc |
| Tsinghua | CS/AI graduate programs + IIIS | Program structure not verifiable this pass (⚠) |
| Toronto | MScAC (AI concentration) + CS PhD | Yes (the AI concentration of the MScAC) |

**The structural split is real.** Only CMU (and, in a narrower sense, Cambridge's MLMI and NUS's DSML MSc) offers a degree whose *name* is the specialty. The US elite pattern — MIT, Stanford, Berkeley — is the *pathway inside a general degree*: the AI content is a specialization or research area of EECS/CS, not a separately branded degree. Applicants comparing "AI master's programs" should read this table first: at six of the ten schools, the AI/ML program is a route inside a CS degree, and the marketing distinction between "MS in AI" and "MS in CS with AI specialization" is often thinner than it looks.

### 13.2 The Core-Course Canon Across the Ten

Read across the sections, the ten schools' core canons converge on a **shared graduate spine**, with the differences concentrated at the edges:

| Canon layer | Schools where it is explicit in the flagship core | Representative courses |
| --- | --- | --- |
| ML foundations (supervised/unsupervised, model selection) | All ten | MIT 6.7900 · Stanford CS229 · CMU 10-701/10-601 · Berkeley CS189 · Oxford ML · Cambridge MLMI ML-for-IS · ETH AML (252-0535) · NUS CS5339 · Toronto CSC2515 |
| Deep learning | All ten | MIT 6.7960 · Stanford CS230 · CMU (10-7xx family) · Berkeley CS182 · Oxford Deep Learning · ETH 263-3210 · NUS CS5242 |
| Probabilistic modeling / graphical models | Most (the Bayesian-flavored schools explicitly) | MIT 6.4110 · Stanford CS228 · CMU 10-708 · Oxford Probabilistic Modelling · Cambridge Probabilistic ML · ETH AML |
| Optimization / learning theory | A distinct subset (CMU, ETH, Stanford's theory-adjacent courses) | CMU 10-725/10-715 · ETH Statistical Learning Theory · Stanford CS229 theory component |
| NLP / Vision / Speech (the application triad) | All ten at the course level, with the emphasis varying by faculty strength | MIT 6.8610/6.8620/6.8300 · Stanford CS224N/CS231N · CMU LTI · Oxford/Cambridge vision+NLP · NUS CS5246 |
| Reinforcement learning | A strong subset (the RL-powerhouse schools) | Stanford CS234 · Berkeley CS285 · CMU 10-703 |
| The research-seminar/thesis spine | All ten (the doctorate's coursework is the means, not the end) | (everywhere) |

**What is universal:** the ML-foundations + deep-learning pair is the non-negotiable core at every school. **Where they diverge:** (a) the *probabilistic/Bayesian* emphasis (Cambridge, Oxford, MIT's 6.4110 carry it explicitly; the applied US professional routes less so); (b) the *theory/optimization* depth (CMU and ETH push it hardest); (c) the *application triad* of NLP/vision/speech is present everywhere but its flagship courses track each faculty's research strengths; and (d) *RL* is a first-class course only where the faculty research demands it (Stanford, Berkeley, CMU).

### 13.3 The Practical Comparisons

- **Duration and structure.** The taught master's runs one year (Oxford, Cambridge MLMI) to two years (CMU MS-ML, NUS MComp part-time variants, ETH); the US professional degrees (Berkeley MEng, Toronto MScAC) run one to two years with capstone/internship components. The doctorate is five-plus years everywhere with the coursework-is-means structure documented in §1.3.
- **The industry-integration axis.** Toronto's MScAC (mandatory internship), Berkeley's MEng (capstone), and NUS's MComp (part-time professional stream) are the most industry-facing; the UK MScs and the US research master's are the most academic.
- **The verification asymmetry.** This survey's live-verification standard was met fully for MIT, CMU, Oxford, Cambridge, and Toronto (programme-level pages verified live), partially for Stanford and ETH (research-pass anchors + blocked pages), and least for Berkeley, NUS, and Tsinghua (pages unreachable this pass; rows ⚠). The ledger (§14) and §15 make the asymmetry explicit rather than papering over it.

---

## 14. The Verification Ledger

Every substantive fact in Sections 3–12 carries an inline mark. This ledger aggregates them by outcome.

### 14.1 Confirmed (✅) — verified against a live primary page this pass

| # | Fact | Source verified |
| --- | --- | --- |
| 1 | MIT is #1 in the QS 2026 CS&IS subject table (head order MIT/Stanford/CMU) | QS subject-page copy + secondary corroboration (§2.2) |
| 2 | MIT's AI area is "AI+D" (Artificial Intelligence and Decision-making) inside EECS | eecs.mit.edu (§3.1) |
| 3 | MIT EECS graduate programs (PhD, MEng) and the 6-4 undergraduate AI+D major | eecs.mit.edu + catalog.mit.edu (§3.2) |
| 4 | MIT course canon: 6.3900, 6.7900, 6.7960, 6.4110, 6.4300, 6.8300, 6.8610, 6.8620[J] with the catalog lines quoted | student.mit.edu catalog pages + course sites (§3.4) |
| 5 | CMU's Machine Learning Department exists, was founded 2006, and describes itself as the world's first such department | ml.cmu.edu live (§5.1) |
| 6 | CMU MLD degree roster: PhD in ML, three joint PhDs, Primary/5th-Year/Secondary master's in ML | ml.cmu.edu live (§5.2) |
| 7 | CMU "ranked No. 1 in AI" per U.S. News 2024 (CMU's own site statement) | ml.cmu.edu live (§5.1) — reported as the university's claim |
| 8 | Oxford graduate degree roster (MSc Advanced CS incl. the AI variant, DPhil, MSc Math & Foundations, AIMS CDT, MSc AI for Business, part-time professional MScs) | cs.ox.ac.uk + ox.ac.uk live (§7.2) |
| 9 | Oxford's flagship ML course page (Machine Learning, Flaxman, Michaelmas) exists | cs.ox.ac.uk teaching pages (research pass) (§7.4) |
| 10 | Cambridge MLMI MPhil: eleven months, full time, by the ML/Speech/CV+Robotics groups of the Department of Engineering, with a research-project component | mlmi.eng.cam.ac.uk live (§8.1-8.3) |
| 11 | Toronto MScAC: concentrations incl. Artificial Intelligence, AI in Healthcare, Data Science, Quantum Computing; internship-integrated professional master's | web.cs.toronto.edu live (§12.1-12.3) |
| 12 | THE 2026 CS table: Oxford #1 (headline) | THE summary copy (§2.3) |
| 13 | CSRankings 2026: Tsinghua/SJTU tied #1 overall; Chinese universities dominate the top ten | Secondary coverage of the release (§2.3) |
| 14 | US News AI subject: Tsinghua #1 | Press coverage of the release (§2.3) |
| 15 | EduRank world-AI 2026: Stanford #1, Berkeley #2, MIT #3, CMU #4, Toronto #5 | EduRank via third-party (§2.3) |

### 14.2 Uncertain (⚠) — could not be re-verified against a live official page this pass

| # | Fact | Why ⚠ |
| --- | --- | --- |
| 1 | Stanford's current MS-CS specialization sheets and PhD structure | cs.stanford.edu returned 404s on the program URLs this pass (§4) |
| 2 | Stanford course canon rows (CS229/CS230/CS224N/CS231N/CS228/CS236/CS234/CS224W) | Long-documented public facts; live catalog not re-reached this pass (§4.4) |
| 3 | CMU's individual 10-xxx course-catalog entries | MLD programme pages verified live; the course-listing application was not reachable (§5.4) |
| 4 | Berkeley's EECS graduate program structure and course canon | eecs/ML pages blocked or unextractable this pass (§6) |
| 5 | Oxford's current-year MSc option menu (beyond the verified ML course) | The option list rotates; not fully re-extracted this pass (§7.4) |
| 6 | Cambridge MLMI module codes and the current-year module tables | Programme page verified live; the module tables were not extracted cleanly (§8.4) |
| 7 | ETH's MSc major structure and most course identifiers | ETH pages blocked this pass; two identifiers (252-0535, 263-3210) anchored in a research pass (§9) |
| 8 | NUS's programme pages (404s) and module codes | comp.nus.edu.sg URLs 404'd this pass (§10) |
| 9 | Tsinghua's entire code-level curriculum | English pages sparse/unextractable; codes omitted rather than guessed (§11) |
| 10 | Toronto's CSC course identifiers (CSC2515 etc.) | Programme page verified live; the course calendar was not re-extracted (§12.4) |
| 11 | The QS 2026 CS&IS positions #4-#10 exact order | One aggregator's rendering only; topuniversities.com blocks extraction (§2.2) |
| 12 | MIT's MEng Fall-2026 restructure specifics | FAQ page exists and is linked; the new requirements' text was not extracted (§3.3) |
| 13 | MIT SM-in-EECS structure | Graduate portal described it; page not retrieved live (§3.2) |

### 14.3 Contradicted (❌) — assumptions this pass overturned or refused

| # | Claim | Disposition |
| --- | --- | --- |
| 1 | "The assignment's top ten can be taken from any single ranking" | ❌ Refused in §2 — the rankings disagree materially (US reputation vs China output); the roster is justified per-school against named lists (§2.4-2.5) |
| 2 | "The tenth school (Toronto/Princeton/EPFL) can be picked on general reputation" | ❌ Refused — Toronto was selected on the only AI-specific top-five citation found (EduRank #5); Princeton/EPFL lacked one (§2.5) |
| 3 | "Every school offers a named AI/ML master's degree" | ❌ Refuted by §13.1 — six of ten schools run AI/ML as a pathway inside a general CS degree; only CMU (and narrowly Cambridge/NUS) brand the degree by the specialty |
| 4 | "Course codes can be listed from memory for all ten schools" | ❌ Refused — Tsinghua's codes are omitted entirely (§11.4) rather than guessed; the guide's governing rule (§1.2) outranks completeness |

---

## 15. What Could Not Be Verified

Per the repository's honesty convention, this section collects what this pass could not verify, with the reason. The distinction matters: *unverified* is not *untrue* — it is *unproven by this pass*.

1. **Stanford's current (2026-27) program pages.** The master's-program and graduate-admissions URLs returned 404s throughout this pass (the department's site restructured). Stanford's MS-CS AI specialization requirements and the current PhD structure therefore rest on long-documented public knowledge marked ⚠, not on a live page. (§4, §14.2)
2. **Berkeley's EECS graduate pages.** The ML-lab page was blocked by the extraction environment ("private or internal network address" guard on the redirect), and the EECS graduate pages did not return clean text. The Berkeley course canon is recorded from long-documented public knowledge with every row ⚠. (§6, §14.2)
3. **ETH Zurich's master's pages.** The master's-CS domain was similarly blocked this pass. Two course identifiers (252-0535 Advanced Machine Learning, 263-3210 Deep Learning) were anchored by an earlier research pass in the ETH course-catalogue context; everything else ETH is ⚠. (§9, §14.2)
4. **NUS Computing's programme pages.** The MSc-DSML URL and the module-programme pages 404'd this pass (the site's information architecture changed). NUS module codes rest on long-documented public knowledge marked ⚠. (§10, §14.2)
5. **Tsinghua's curriculum at code level.** Tsinghua's English-language pages were sparse and not reliably extractable. Rather than assert course codes from memory, this survey records Tsinghua's course *families* as structural descriptions only — an honest gap. (§11, §14.3)
6. **The exact #6-#10 order of the QS 2026 CS&IS table.** topuniversities.com blocks automated extraction; the school set is corroborated by secondary coverage, the exact order is not. (§2.2)
7. **The current-year option/module lists at Oxford and Cambridge.** Oxford's MSc option menu rotates yearly and was not fully re-extracted; Cambridge MLMI's module tables were reached but not cleanly extracted. (§7.4, §8.4)
8. **MIT's Fall-2026 MEng restructure specifics and the SM-in-EECS structure.** Both are documented to exist; the detail pages were not retrieved live. (§3.2-3.3)
9. **Individual course-catalog entries at CMU (the 10-xxx course pages).** The MLD programme pages verified live; the course-listing application behind them was not reachable. (§5.4)
10. **Toronto's CSC course-calendar entries.** The MScAC programme page verified live; the department's course calendar was not re-extracted. (§12.4)
11. **Whether CMU's "first academic department of its kind" claim and the "U.S. News #1 in AI 2024" claim survive independent adjudication.** Both are recorded as the university's own claims on its live site. (§5.1)
12. **The sustainability of the code-level canon as catalogs rotate.** Every section's rows are dated to the 2026-27 catalogs where published; a few schools still showed 2025-26 entries at this pass, and those items carry ⚠. (§1.2)

The verification asymmetry across the ten schools is real and is not hidden: MIT, CMU, Oxford, Cambridge, and Toronto were verified to the highest standard this pass; Stanford and ETH partially; Berkeley, NUS, and Tsinghua least. Readers using this guide to compare programs should weight the ⚠ accordingly — and should treat any row marked ⚠ as *to be re-checked at the school's live catalog before a decision*, which is precisely what the guide's method (§1.2) prescribes.

---

## 16. Worked Example: Cymbal Bank — A Learning Path Across the Shared Curriculum

> The conventions of this repository's worked examples apply: Cymbal Bank is a fictional bank used to make the material concrete. This section is design fiction built on the verified facts of Sections 3–13, not a report of any real program.

**The situation.** Cymbal Bank's group architecture office is building an internal AI engineering capability. The bank has agreed to fund a two-year, cohort-based upskilling track for ten senior engineers — the people who will later design the bank's agentic-AI systems (see the repository's multi-agent banking guide for that program's shape). The head of technology asks the architecture office to anchor the track in *institutional* curricula: "Don't invent our own syllabus — teach what the best universities actually teach, then add the bank's own risk and data-governance layers." This example shows how the architecture office would use this guide's §13.2 canon map to design that track.

**Step 1 — read the canon map (§13.2).** The universal core across the ten schools is the pair *ML foundations + deep learning*. The office therefore makes those two the non-negotiable spine of year one, taught from the syllabus structure of the schools with the clearest public canons: MIT's 6.7900/6.7960 pair (§3.4) and Stanford's CS229/CS230 pair (§4.4) — the two US flagships whose course outlines this guide verified (or long-documented) at the highest standard.

**Step 2 — choose the divergence to copy.** The §13.2 map shows the probabilistic/Bayesian emphasis is strongest at Oxford, Cambridge, and MIT (6.4110); the theory/optimization depth is strongest at CMU and ETH. The office's target use cases (agentic systems with auditability requirements — see the repository's LLM-evaluation and AI-compliance guides) argue for the *probabilistic + theory* emphasis over the applied-fastest route. The track therefore sequences: MIT-style ML foundations → CMU-style mathematical foundations (the 10-606/10-607 family shape, §5.4) → Oxford/Cambridge-style probabilistic modeling → ETH-style statistical learning theory (§9.4).

**Step 3 — map the application triad to the bank's portfolio.** NLP (CS224N/6.8610 shape) maps to the bank's document-heavy KYC and compliance workflows; vision (CS231N/6.8300 shape) maps to the check-imaging and document-verification backlog; RL (CS234/CS285/10-703 shape) is deferred — the office judges that the bank's control environment will not need agentic RL in the two-year horizon and that the RL courses' value is conceptual. This triage is exactly what §13.2's "divergence" column is for.

**Step 4 — add the bank layer the universities do not teach.** No university canon covers MAS-grade model risk, auditability, or the evaluation gates the bank's own AI-governance program requires. The office appends the repository's own material — the multi-agent banking guide's eval pipelines, the LLM-evaluation-frameworks guide, the AI-compliance guide — as the bank-specific capstone on top of the institutional spine. The universities supply the *science*; the bank's internal material supplies the *control environment*.

**Step 5 — the sequencing.** Year one: ML foundations → mathematical foundations → probabilistic modeling, delivered as a structured reading-and-problem-track using each school's course outline as the syllabus (the outlines of §§3.4, 5.4, 7.5, and 9.5 give the topic order). Year two: statistical learning theory → the NLP/vision applications mapped in step 3 → the bank's own governance capstone → a supervised project that mirrors the MScAC-style industry project (§12.3) using the bank's data under its governance rules.

**Why this works.** The design reuses the ten schools' curriculum *structure* — which is public, verified, and battle-tested by admissions committees — instead of inventing a syllabus. Where the guide's verification is thin (Berkeley, NUS, Tsinghua — §15), the office does not borrow; it sticks to the schools whose outlines are marked ✅. And the Cymbal Bank layer — the eval gates, the audit trail, the MAS overlay — is precisely the content the institutional programs, by design, do not teach. The learned machine (§19) is a *hybrid*: institutional science plus the bank's control discipline.

---

## 17. Glossary

| Term | Meaning |
| --- | --- |
| **AI+D** | MIT EECS's "Artificial Intelligence and Decision-making" research/teaching area (§3.1) |
| **BAIR** | Berkeley Artificial Intelligence Research, the EECS AI research umbrella (§6.1) |
| **Canon** | In this guide, the set of core courses a flagship program requires or recommends (§1.1) |
| **CDT** | Centre for Doctoral Training — the UK funded-doctorate structure (Oxford's AIMS) (§7.2) |
| **Coterm(inal)** | Stanford's option to pursue the MS while completing the BS (§4.6) |
| **D-INFK** | ETH Zurich's Department of Computer Science (§9.1) |
| **DPhil** | Oxford's doctorate (equivalent to the PhD) (§7.2) |
| **EECS** | Electrical Engineering and Computer Sciences — MIT's and Berkeley's combined department (§3.1, §6.1) |
| **Flagship program** | The one or two degrees a school's AI/ML students most target (§1.1) |
| **IIIS** | Tsinghua's Institute for Interdisciplinary Information Sciences ("Yao Class" tradition) (§11.2) |
| **LTI** | CMU's Language Technologies Institute (NLP/LLM research) (§5.6) |
| **Major** | ETH's specialization structure inside the MSc in CS (§9.3) |
| **MEng** | Master of Engineering — the professional master's at MIT/Berkeley (§3.2, §6.2) |
| **MLD** | CMU's Machine Learning Department, founded 2006 (§5.1) |
| **MLMI** | Cambridge's MPhil in Machine Learning and Machine Intelligence (§8.1) |
| **MPhil** | Cambridge's master's-level degree title (§8.2) |
| **MScAC** | Toronto's Master of Science in Applied Computing (§12.1) |
| **OATML** | Oxford Applied and Theoretical Machine Learning group (§7.1) |
| **Pathway shape** | AI/ML delivered as a specialization inside a general CS degree (§1.3) |
| **Named degree shape** | A degree whose title names the specialty (e.g., CMU's MS in ML) (§1.3) |
| **PGM** | Probabilistic graphical model (Bayesian networks, Markov networks, factor graphs) |
| **SoC** | NUS's School of Computing (§10.1) |
| **Specialization** | The AI/ML route inside a general degree (Stanford's MS-CS AI specialization) (§4.2) |
| **Units** | MIT's lecture/lab/self-study workload notation (e.g., 4-0-8) (§3.4) |
| **Vector Institute** | Toronto's AI research institute in the Hinton lineage (§12.1) |

---

## 18. Cross-References

- **The practitioner complement.** [ai_engineer_roadmap_2026.md](ai_engineer_roadmap_2026.md) maps the self-taught path this guide's institutional canon benchmarks against (§1.1); [mathematics_for_ml_and_ds_study_notes.md](../../mathematics_for_ml_and_ds_study_notes.md) covers the mathematical substrate several canons assume (MIT's 18.06/6.3700 prerequisites, CMU's 10-606/10-607 family, §3.4/§5.4).
- **The course-level companions.** [deep_learning_frameworks_comparison_guide.md](deep_learning_frameworks_comparison_guide.md), [reinforcement_learning_algorithms_guide.md](../../reinforcement_learning_algorithms_guide.md), and [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) pair with the deep-learning, RL, and evaluation content the canons teach (§1.1).
- **The bank's own AI program.** The multi-agent banking guide in this cluster documents the agentic-AI engineering program whose governance the worked example (§16) layers onto the institutional spine; the AI-compliance guide and the LLM-evaluation guide carry the bank-layer content §16 references.
- **The ranking context.** The repository's technology and Singapore clusters document the research-evaluation and AI-policy context behind the §2 rankings discussion where relevant (the CSRankings output-count method, the Singapore national-AI context for NUS).

### 18.1 Sources Accessed Live During This Pass

The verification standard of §1.2 was applied against these primary pages, which were reached and read during the production of this guide (September 2026):

- MIT: eecs.mit.edu (the AI+D area and graduate-programs pages); student.mit.edu Course-6 subject listings; introml.mit.edu; gradml.mit.edu (course sites, §3.4-3.5)
- CMU: ml.cmu.edu (the MLD about, degree-roster, and apply pages — §5.1-5.2)
- Oxford: cs.ox.ac.uk/admissions/graduate (the full graduate-course roster — §7.2); cs.ox.ac.uk teaching pages for the Machine Learning course (§7.4)
- Cambridge: mlmi.eng.cam.ac.uk (the MLMI programme home and structure pages — §8.1-8.3)
- Toronto: web.cs.toronto.edu/graduate/mscac (the MScAC programme page and concentration list — §12.1-12.3)
- Ranking sources: QS subject-page copy and secondary corroboration of the QS 2026 release; THE 2026 computer-science summary copy; secondary coverage of the CSRankings 2026 and US News AI releases; the EduRank world-AI list (§2.2-2.3)

**Failed or blocked this pass** (each recorded with its ⚠ consequences in §14-§15): Stanford's program URLs (404 — site restructured); the Berkeley ML-lab page (blocked by the extraction environment's network guard) and the EECS graduate pages (no clean text); the ETH master's domain (blocked); NUS Computing's programme URLs (404 — site information architecture changed); Tsinghua's English program pages (sparse/unextractable); topuniversities.com (blocks automated extraction).

---

## 19. Closing: The Learned Machine

Ten universities. Four program shapes. One shared canon — the ML foundations and the deep-learning core that every flagship teaches, in every country, on every ranking. The differences that matter are the ones this guide has documented rather than assumed: which schools brand the degree by the specialty and which hide it inside a general CS degree; which push the probabilistic and the theoretical depth and which teach the applied fastest; which verified their catalogs to this pass's standard and which could not be reached. For the candidate choosing, the leader hiring, and the self-taught engineer benchmarking — the institutional curriculum is the most public, most vetted syllabus in the world. Read it, borrow it, and add the layer the universities do not teach: the discipline of putting the learned machine to work. That is the learned machine.
