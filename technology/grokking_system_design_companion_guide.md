# Grokking the System Design Interview (Educative): The Study Companion

*A comprehensive study companion to the Educative course "Grokking the System Design Interview" — the course overview and verified metadata (platform, author, format, lineage), the 7-step framework (requirements, estimation, API design, data model, high-level design, deep dive, wrap-up) and its time budget, the catalogue of ~16 design problems (URL shortener, Pastebin, Instagram, Tinder, WhatsApp, Netflix, Twitter, Uber, Dropbox, and the rest), deep-dives of the most-recycled problems (rate limiter, WhatsApp/chat, Uber/geo), the comparison against Alex Xu's "System Design Interview – An Insider's Guide", the full chapter-to-repo-guide mapping, the interview value for the Google / ML / DDIA preparation stack, a complete worked example (WhatsApp, the full 7-step design), a one-page summary, a verification-and-claims section that flags every unverifiable claim honestly, and a glossary.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Career Skills / System Design (technology/) — the BOOK/COURSE-COMPANION pattern (see [management/the_managers_path_research.md](../management/the_managers_path_research.md) for the precedent)
> **Audience:** Solution architects, platform engineers, backend engineers, system design interview candidates
> **Last Updated:** August 2026

**Cross-references (the repo guides this companion maps into):** [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (the rate limiter — Grokking's rate-limiter chapter — THE worked deep-dive), [google_system_design_interview_guide.md](google_system_design_interview_guide.md) (the Google interview — the framework family this course feeds), [ml_system_design_interview_guide.md](ml_system_design_interview_guide.md) (the ML design interview — same clarify-first structure), [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (the theory companion — the "why" under Grokking's "how"), [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (the Alex-Xu practice companion — the closest sibling; the case-study overlap is mapped in §5), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (MQ reliability — the notification/feed/chat designs), [s3_architecture_guide.md](s3_architecture_guide.md) (object storage — the Dropbox/Instagram/Netflix designs), [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (streaming — the Twitter/feed fan-out), [late_arriving_data_guide.md](late_arriving_data_guide.md) (watermarks and out-of-order data — chat/feed ordering, cross-ref lightly), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (integration — the "design in my domain" variant, cross-ref lightly), [apache_seata_guide.md](apache_seata_guide.md) (distributed transactions — the Ticketmaster/payment-flavored designs), [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable workflows — delivery guarantees and trip/booking lifecycles), [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) (search — the typeahead/Twitter-search designs), [cloud_providers_guide.md](cloud_providers_guide.md) (the component tour — Grokking's fundamentals section), [oracle_sharding_guide.md](oracle_sharding_guide.md) + [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) (partitioning and data models — the URL-shortener/message-store designs), [livekit_alternatives_guide.md](livekit_alternatives_guide.md) (real-time transport — the WhatsApp/chat design), and the [banking/](../banking/) guides (worked designs in the banking domain — cross-ref lightly for the "design in my domain" variant).

---

## Table of Contents

1. [The Course Overview](#1-the-course-overview)
2. [The Framework: The 7 Steps](#2-the-framework-the-7-steps)
3. [The Design Problems: The Catalogue](#3-the-design-problems-the-catalogue)
4. [The Problem Deep-Dives](#4-the-problem-deep-dives)
5. [The Comparison: Grokking vs the Xu Insider's Guide](#5-the-comparison-grokking-vs-the-xu-insiders-guide)
6. [The Repo Mapping: Chapter-to-Guide](#6-the-repo-mapping-chapter-to-guide)
7. [The Interview Value](#7-the-interview-value)
8. [The Worked Example: WhatsApp, the Full 7-Step Design](#8-the-worked-example-whatsapp-the-full-7-step-design)
9. [The Summary: The Course in One Page](#9-the-summary-the-course-in-one-page)
10. [Verification and Claims-Status](#10-verification-and-claims-status)
11. [Glossary](#11-glossary)

---

## 1. The Course Overview

### 1.1 What the Course Is

**"Grokking the System Design Interview"** is the interactive course on the **Educative** platform that popularized online system design interview preparation. It is the course that started the genre: before Grokking, system design interview prep meant scattered blog posts and a few books; after Grokking, "grokking" became a verb in interview-prep marketing, and every competitor course on the market today is a variation on its template — a framework, a component library, a set of worked problems.

The course's thesis is the same one Alex Xu's book would later popularize in book form: **system design questions look open-ended and scary, but they are actually scored against a recognizable shape** — a repeatable framework applied to a small set of recurring systems. The course teaches that shape: first the building blocks of distributed systems (load balancers, caching, sharding, queues, CDNs), then a **7-step framework** for attacking any question, then a catalogue of **~16 real design problems** worked end-to-end (URL shortener, Pastebin, Instagram, Tinder, WhatsApp, Netflix, Twitter, Uber, Dropbox, and the rest — see §3).

Unlike a book, the course is **interactive**: lessons render in the browser with click-through diagrams, worked examples, quizzes, and (in the original edition) inline code sketches. You read and "do" in the same window — which is why the course, rather than any book, became the default first stop for engineers preparing for FAANG design rounds.

### 1.2 The Verified Metadata (and the Lineage Flags)

| Field | Value | Status |
|-------|-------|--------|
| Course title | Grokking the System Design Interview | Verified |
| Platform | Educative (educative.io) — interactive course platform | Verified |
| Format | Interactive course: in-browser lessons, diagrams, quizzes, worked examples | Verified |
| Author | Fahim ul Haq (Educative co-founder & CEO) — see §1.3 for the lineage nuance | Verified (flagged) |
| Launch | ~2018 — commonly cited as the course's release year | **Flagged — commonly cited, not officially confirmed** |
| Structure | System-design fundamentals + the 7-step framework + ~16 design problems + wrap-up/tips | Verified (count flagged) |
| Size (as sold today) | Design Gurus edition: 83 lessons, videos, diagrams, quizzes, certificate | Verified (Design Gurus listing) |
| Related course | Grokking the Advanced System Design Interview (Dynamo-style KV store, distributed messaging/streaming, distributed file storage) | Verified |
| Reception | Routinely called the #1 system design interview course; "the course that started the genre" | **Flagged — qualitative** |
| Critiques | "Cookie-cutter" answers, memorization risk, dated original edition, light on theory | **Flagged — qualitative** |

### 1.3 The Authors and the Messy Lineage (flagged honestly)

The authorship of "Grokking the System Design Interview" needs a careful flag, because the course's history is messier than a single byline:

- **Fahim ul Haq** — co-founder and CEO of **Educative** — is the author the course is associated with. He is a software engineer (background at Meta and Microsoft, per his Educative bio) and wrote/led the original Educative edition of the course. The *modern* successor that Educative now serves at the original URL — **Grokking Modern System Design Interview for Engineers & Managers** — is authored by Educative's in-house team and led by Fahim ul Haq (per Educative's own course-comparison content).
- **The lineage wrinkle (flagged):** the *original* Grokking course content is now also sold at **Design Gurus** (designgurus.io) as "Grokking System Design Interview — the official DesignGurus course by the original creators," credited to **Arslan Ahmad and the original Design Gurus team** (described as ex-hiring managers from Meta, Microsoft, Google, and Amazon). Design Gurus' own marketing frames its edition as the original, and Educative's as the rebuilt "Modern" successor.
- What this means in practice: **there is no single canonical edition.** An engineer who "took Grokking" in 2018 took the Educative original; one who takes it in 2026 is either on Design Gurus' preserved original or Educative's Modern rebuild (204 lessons, 140 quizzes, 8 mock interviews, ~26 hours, taught around a "RESHADED" method, per third-party course reviews). All editions share the DNA: fundamentals → framework → worked problems.

The honest statement for this companion: **the course's framework and problem catalogue (the subject of this guide) are stable across editions; the platform, lesson counts, and byline differ by era.** Every count in this guide is flagged accordingly.

### 1.4 The Format: The Interactive Course (verified)

The format is what made the course distinctive, and it is verified from Educative's own course page copy and third-party reviews:

- **In-browser lessons** — no video to fast-forward; text + diagrams that render as you scroll. The course "breaks complex topics into clear, step-by-step lessons" (Educative's own description).
- **Click-through diagrams** — architecture diagrams build up incrementally, which models exactly how an interviewer wants to see a design assembled (shape first, detail after).
- **Quizzes** — per-section checks that force recall of the framework steps and the trade-offs, not just passive reading.
- **Worked problems** — each design problem is walked through with the same 7-step skeleton, so by problem #10 the skeleton is automatic.
- **The "chapters" count (flagged):** the course is commonly described as **~16 chapters / ~16 design problems** (the design-problem section is the headline), on top of a fundamentals section. Editions differ: the Design Gurus edition lists **83 lessons** total (fundamentals + problems + tips); third-party guides cite 16-19 problems depending on edition; the Modern rebuild restructured everything into 204 lessons. **"~16 chapters" is the classic description and is flagged as edition-dependent.**

### 1.5 The Course Structure (verified shape)

The course is organized in four movements, and this shape is stable across editions:

1. **System design fundamentals (the building blocks).** The component tour every problem reuses: load balancing, caching, partitioning/sharding, indexes, proxies, queues, redundancy, SQL vs NoSQL, CAP theorem, consistent hashing, and the "scale from zero" ladder. The repo's [cloud_providers_guide.md](cloud_providers_guide.md) and [ddia_study_companion_guide.md](ddia_study_companion_guide.md) are the deep-dives of this section.
2. **The 7-step framework.** The course's center of gravity — the repeatable interview protocol (see §2).
3. **The design problems (~16).** Each problem runs the 7 steps end-to-end on a named system (see §3).
4. **Wrap-up and interview tips.** How to handle the unfamiliar question, what interviewers score, and (in the Modern edition) mock interviews.

### 1.6 The Audience

The course targets **software engineers preparing for system design interviews** — the FAANG/MAANG loop, typically the 1-6 year experience band where the design round gates leveling (see [google_system_design_interview_guide.md](google_system_design_interview_guide.md) for how the round gates L4/L5/L6). It is also the standard first read for: engineers switching into backend/platform roles who need distributed-systems vocabulary fast; architects who want a shared design language for reviews; and hiring managers who want the rubric-shaped view of a good answer.

For a **banking technologist** (the Cymbal Bank context), the value is the same double-payoff the other companions describe: the 7-step skeleton is a design-document skeleton, and the worked problems are the same shapes as banking delivery pipelines — WhatsApp is trader chat, the notification system is market-data alerts, Dropbox is the document vault, Ticketmaster is seat/booking inventory (the [banking/](../banking/) guides show the domain side).

### 1.7 Reception and Critiques (flagged as qualitative)

**The popularity is not in dispute.** The course is routinely called the #1 system design interview course ("Grokking System Design Interview is the official DesignGurus course by the original creators… Learn system design fundamentals, trade-offs, and real FAANG-style design problems," per Design Gurus; "the flagship course that popularized system design prep online," per third-party reviews). It appears on every "best system design courses" list and is the most-recommended first resource on r/ExperiencedDevs and Hacker News threads. **Specific sales or enrollment numbers are not published — any "N students" figure you see is a guess and is flagged as unverifiable.**

**The critiques are equally well-known — and this companion takes them seriously, because the same critiques apply to every Grokking-style resource (including the Xu book, see §5):**

- **"Cookie-cutter" answers.** The 7-step framework is so teachable that interviewers now see candidates recite the same skeleton for every question without adapting it. The critique sticks to graduates who treat the template as the answer rather than as a conversation protocol.
- **Memorization risk.** ~16 worked problems are seductively memorizable; the honest use is to internalize the *component library* (the same five components appear in every problem) rather than memorize sixteen diagrams.
- **Shallow on fundamentals.** Compared to [ddia_study_companion_guide.md](ddia_study_companion_guide.md), the course hand-waves the hard theory — consistency models get a table, consensus gets a paragraph. Interviewers who probe depth will catch the gap.
- **Dated original edition.** The 2018-era content predates modern data platforms; some scale numbers are interview-shaped approximations rather than 2026 facts.

The balanced reading, consistent with the repo's other companions: **Grokking is the best-known *shape* of the answer; DDIA is the depth under it; the repo's deep-dives are the working components in between** (see §6 and §7).

### 1.8 The Overview Table

| Aspect | Description |
|--------|-------------|
| What it is | The interactive Educative course that started the system-design-interview-prep genre (~2018) |
| Platform / format | Educative (educative.io); in-browser interactive lessons, diagrams, quizzes |
| Author / lineage | Fahim ul Haq (Educative co-founder & CEO); original content now also sold at Design Gurus; Educative's URL now serves the "Modern" rebuild — flagged (§1.3) |
| Structure | Fundamentals (building blocks) + the 7-step framework + ~16 design problems + wrap-up tips |
| Core deliverable | The 7-step framework (requirements → estimation → API → data model → high-level → deep dive → wrap-up) + a reusable component library |
| Problems | URL shortener, Pastebin, Instagram, Tinder, WhatsApp, Netflix, Twitter, Uber, Dropbox + ~7 more (web crawler, typeahead, news feed, Yelp/proximity, Ticketmaster, Twitter search, rate limiter, notification) — count flagged (§3) |
| Related course | Grokking the Advanced System Design Interview (KV store, messaging/streaming, file storage) — verified |
| Audience | FAANG-bound engineers 1-6 YOE; architects; hiring managers; banking technologists wanting the design-round language |
| Reputation | De facto standard first course; critiques: formulaic answers, light theory, dated original edition (flagged) |
| Companion in this repo | The course companion alongside [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (the Xu book companion — the closest sibling), [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (theory), [google_system_design_interview_guide.md](google_system_design_interview_guide.md) and [ml_system_design_interview_guide.md](ml_system_design_interview_guide.md) (the interviews) |

### 1.9 How to Take the Course (three passes)

The course rewards three passes, each with a different goal — the same reading discipline the Xu companion prescribes for the book:

- **Pass 1 — the framework (two evenings):** do the fundamentals section quickly (skim the component tour — [cloud_providers_guide.md](cloud_providers_guide.md) has the depth) and then the framework lessons *completely*. Goal: the 7 steps and their time budgets must be automatic before anything else. Exit test: can you name the 7 steps, their time budgets, and the ~7 recurring components without looking?
- **Pass 2 — the problems (a week):** work the first two problems (URL shortener, Pastebin) end-to-end *with* the framework open, then the high-value set (WhatsApp, Uber, Twitter, Dropbox, rate limiter) — these are the most-recycled interview questions and the ones with the richest repo deep-dives (§4). Skim the rest. Goal: for each problem, be able to state the question, the crux, and the one-line answer (the §3.3 table is the cheat sheet).
- **Pass 3 — the polish (the week before interviews):** redo the framework and wrap-up lessons; then rehearse — deliver the §8 worked example out loud, against a timer, with a whiteboard. Goal: the framework must survive pressure. Exit test: you can run "clarify → estimate → API → data model → shape → deepen → wrap" on a question you have never seen.

**The pairing rule:** every problem lesson should be paired with its repo guide (the §6.1 table is the pairing). The course gives the shape; the repo guide gives the depth an interviewer's follow-up questions will probe. Taking the course without the deep-dives leaves you with answers that work for the first question and collapse under the second — the exact failure the "cookie-cutter" critique describes (§1.7).

### 1.10 The Genre Timeline (the course's place in history)

The course is best understood as a point on a timeline — the repo's companions all sit on this line, and knowing where each resource sits explains what it's good for:

| Era | What happened | The repo's lens |
|-----|---------------|-----------------|
| ~2015-2017 | **The pre-Grokking era.** System design prep meant scattered blog posts (High Scalability write-ups, conference talks) and word-of-mouth question lists. Candidates walked in cold; interviewers scored on vibes because there was no shared vocabulary. | The theory that would anchor the genre already existed — DDIA (2017) — but nobody had connected it to the interview. |
| ~2018 | **Grokking launches on Educative** (flagged — the year is commonly cited, not officially confirmed). The course bundles fundamentals + a 7-step framework + ~16 worked problems into an interactive format. "Grokking" becomes the genre name. | This course — the subject of this companion. |
| 2019-2020 | **The genre industrializes.** Clones multiply; the original content migrates to Design Gurus (Arslan Ahmad and the original team); **Alex Xu self-publishes "System Design Interview – An Insider's Guide" (2020)** — the book form of the same idea, with the best-written interview-dynamics advice in the genre. | The Xu companion: [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md). The framework arms race begins — 4 steps vs 7 steps (§5). |
| 2021-2023 | **The ecosystem matures.** ByteByteGo's animated courses, "Grokking the Advanced System Design Interview" (Dynamo KV store, messaging/streaming, file storage), mock-interview platforms (Hello Interview, Exponent), DDIA rising as the depth counterweight. | The repo's component deep-dives date from this era's questions: MQ reliability, S3 architecture, streaming, workflows. |
| 2024-2026 | **The framework is table stakes.** Educative rebuilds the course as "Grokking Modern System Design Interview" (204 lessons, 140 quizzes, 8 mock interviews, the "RESHADED" method); AI-assisted prep appears; every candidate knows the skeleton, so interviews differentiate on *depth* — trade-offs, follow-up questions, domain adaptation. | The repo's companions and deep-dives are the 2026 "depth layer": shape from the course, depth from the guides (§6, §7). |

**The takeaway:** the course's historical role — popularizer, genre-starter — is why it's still the recommended first resource; its 2018-era content is why it's no longer sufficient alone. That is exactly the gap the repo's deep-dive library fills.

---

## 2. The Framework: The 7 Steps

### 2.1 The Framework (verified)

The course's center of gravity is its **7-step framework** for any system design question. The steps, verified against Design Gurus' own published "System Design Interview Guide – 7 Steps to Ace It" (the original creators' blog, which lists Steps 1-5 explicitly, and their "Mastering System Design Interview in 7 Steps" PDF, which confirms Steps 5-7):

1. **Step 1 — Requirements Clarifications.** Establish *functional* requirements (the features — what the system must do) and *non-functional* requirements (scale, latency, availability, consistency). The course's rule: ask until you can restate the problem, and write the agreed scope down. Do not design before this is settled.
2. **Step 2 — Back-of-the-Envelope Estimation.** Rough numbers: DAU, QPS (peak with a multiplier), storage per year, bandwidth. The point is the *assumptions*, stated out loud, not the arithmetic precision. (The Google guide's scale-math section has the worked numbers.)
3. **Step 3 — System Interface Definition (API design).** Define the APIs the system exposes before drawing the boxes — this forces the requirements to be concrete and catches missing functionality. E.g., URL shortener: `createURL(api_dev_key, original_url, custom_alias?, user_name?, expire_date?) → short_url` and `deleteURL(api_dev_key, url_key)`.
4. **Step 4 — Defining the Data Model.** Entities, relationships, and storage choices: which tables/collections, which storage system (SQL vs NoSQL), how it will be partitioned. The course's point: the data model exposes the read/write pattern, which drives almost everything downstream.
5. **Step 5 — High-Level Design.** The box-and-arrow architecture: clients → load balancer → application servers → caches, databases, CDN, queues as needed. Draw the shape, walk the interviewer through the main flows, get buy-in *before* adding detail.
6. **Step 6 — Deep Dive (Detailed Design).** One or two components in depth — the interviewer's chosen crux: the rate-limiter algorithm, the feed fan-out strategy, the chat delivery semantics. This is where most of the points live (see §2.2) and where the problem chapters earn their keep.
7. **Step 7 — Identifying and Resolving Bottlenecks (Wrap-up).** Name the bottlenecks in the design, propose how to resolve them (and what you'd do with more time), and state the trade-offs you made. A crisp, honest wrap-up is what the interviewer writes down.

**The framework's deeper point** — identical to the Xu book's 4-step framework (see §5): the interview is a *collaboration*, not a monologue. The steps are choreography that demonstrates the signals interviewers score — clarify (ambiguity handling), estimate (assumption-making), design (shape), deepen (expertise), wrap (ownership). Grokking's 7 steps are simply a more granular version of the same protocol: Xu compresses steps 3-4 (API, data model) into his "high-level design" step; Grokking makes them explicit so candidates don't skip them.

### 2.2 Framework Use: The Time Allocation (verified shape, flagged minutes)

The course frames the round as **45-60 minutes** and prescribes a budget across the steps. The verified shape (Design Gurus' own interview guide): **the first 5-10 minutes are requirements; the next 25-35 minutes are the design and deep-dives; the last 5 minutes are wrap-up.** The step-level breakdown below is the commonly-taught allocation from the Grokking family (Educative's blog and third-party guides cite the same ratios); **the individual minute values are flagged as commonly-cited approximations, not a verbatim course table** — real interviews flex them, and the *ratios* matter more than the minutes:

| Step | Commonly-cited budget (45-min round) | What the time buys |
|------|--------------------------------------|--------------------|
| 1. Requirements | 8-10 min | The scope that prevents designing the wrong product |
| 2. Estimation | 3-5 min | Assumptions + the numbers that justify every later choice |
| 3. API design | 3-5 min | Concrete interfaces that pin the requirements down |
| 4. Data model | 3-5 min | Entities + storage + partitioning direction |
| 5. High-level design | 10-15 min | The agreed shape — boxes and arrows, main flows |
| 6. Deep dive | 10-20 min | The crux components — where the points are won |
| 7. Wrap-up | 3-5 min | Bottlenecks, trade-offs, next steps |

(The Google variant compresses Steps 1-4 toward 15 minutes total and expands Step 6, per [google_system_design_interview_guide.md](google_system_design_interview_guide.md); a 60-minute round expands Step 6 further.)

### 2.3 How to Use the Framework (the "framework use")

- **Never skip Steps 1-4 to sprint to the diagram.** The course's most-repeated lesson — and the most common interview failure — is drawing boxes before the scope is settled. The API and data-model steps exist precisely to force concreteness.
- **Do the estimation explicitly but quickly.** State the assumptions ("let's assume 10M DAU, 10:1 read:write — correct me if that's wrong") and move on. The assumption is the gradeable artifact, not the number.
- **Let the interviewer choose the deep dive.** Offer the menu ("I'd go deeper on delivery semantics or presence — which matters more to you?") and follow their lead. Deep-diving the component *you* like instead of the one they care about is a classic failure.
- **Wrap up with trade-offs, not just a recap.** Naming what you deliberately didn't do is a senior signal.
- **The meta-rule:** the framework is a *conversation protocol*, not a script. Candidates who treat it as a script produce the cookie-cutter answers the critiques complain about (§1.7); candidates who treat it as a protocol — scope, size, interface, data, shape, deepen, wrap, with every trade-off spoken — produce the signal interviewers actually score.

### 2.4 The Framework Table

| Step | Activities | Notes (what the course stresses) |
|------|-----------|----------------------------------|
| 1. Requirements | Ask about features (functional) and scale/latency/availability (non-functional); restate the problem; write the agreed scope down | The most common failure is skipping this — "90% of the points are won or lost in the first 10 minutes" (commonly-cited paraphrase, flagged as interpretive) |
| 2. Estimation | DAU → QPS (peak multiplier), storage/year, bandwidth; state assumptions out loud | Numbers must be defensible, not precise; the assumption is the gradeable artifact |
| 3. API design | Define the REST/API surface (endpoints, params, returns) before drawing boxes | Catches missing requirements; pins the data model to real read/write patterns |
| 4. Data model | Entities, relationships, SQL vs NoSQL, partitioning key | Read-heavy vs write-heavy choice here drives cache/sharding decisions later |
| 5. High-level design | Box-and-arrow architecture; walk main flows; get buy-in on the shape | Burying the interviewer in detail before shape is agreed is the classic failure |
| 6. Deep dive | One or two components in depth, chosen with the interviewer; justify with estimation | Where most of the points live; each problem chapter models what a deep dive looks like |
| 7. Wrap-up | Recap, bottlenecks, trade-offs, next steps if given more time | A crisp recap is what the interviewer writes down — never end abruptly |

**The framework table's takeaway:** the 7 steps are a *time-budgeted collaboration protocol*. The course's answer to "what does a good system design interview look like?" is not a design — it is a design *conversation* that follows this shape.

### 2.5 The Framework Mistakes (and the course's guardrails)

The course's framework is famous enough that interviewers now recognize *misuses* of it instantly. The six classic failure modes, and how the course itself guards against each:

| Failure mode | What it looks like | The course's guardrail |
|--------------|--------------------|------------------------|
| Requirements as a checklist | Rattling off "users? QPS? latency?" without listening to the answers | Step 1's purpose is *understanding*, not interrogation — the questions must adapt to the answers |
| Estimation as performance art | Confident numbers with no stated assumptions | The course always states assumptions first ("let's assume 100M new URLs/day — correct me if that's wrong") and invites correction — the assumption is the gradeable artifact, not the number |
| API/data-model skipping | Jumping from requirements straight to the diagram — the failure Grokking's extra granularity exists to prevent | Steps 3-4 are *explicit* steps precisely so they can't be silently dropped (candidates trained only on 4-step frameworks are the usual offenders) |
| Shape-skipping | Drawing the deep-dive diagram before the interviewer agreed the high-level shape | Step 5 exists to get explicit buy-in on the boxes before adding detail |
| Deep-dive by preference | Going deep on the component *you* like, not the one the interviewer cares about | The course's rule: let the interviewer steer the deep dive — offer the menu, take direction |
| Monologue | Narrating a memorized design without check-ins | The wrap-up and check-in habits ("does that match what you had in mind?") keep it a conversation |

The meta-guardrail — identical to the Xu companion's §2.5, because it's the same genre lesson: **the framework is a conversation protocol, not a script.** Candidates who treat it as a script produce the "cookie-cutter" answers the critiques complain about (§1.7); candidates who treat it as a protocol — scope, size, interface, data, shape, deepen, wrap, with every trade-off spoken — produce the signal the interviewer is actually scoring.

---

## 3. The Design Problems: The Catalogue

### 3.1 The ~16 Problems (verified core, flagged count)

The design-problem section is the course's headline. **Nine problems are verified beyond doubt** — they are the course's canonical set, cited in every edition, every review, and every Grokking-derived study list:

1. **Designing a URL Shortening service like TinyURL**
2. **Designing Pastebin**
3. **Designing Instagram**
4. **Designing Tinder**
5. **Designing WhatsApp**
6. **Designing Netflix**
7. **Designing Twitter**
8. **Designing Uber**
9. **Designing Dropbox**

Beyond these nine, the course's problem catalogue is commonly cited to include (order varies by edition):

10. **Designing a Web Crawler like Google**
11. **Designing Typeahead Suggestion (search autocomplete)**
12. **Designing a News Feed system**
13. **Designing Yelp / a proximity server**
14. **Designing Ticketmaster / BookMyShow (event booking)**
15. **Designing Twitter Search**
16. **Designing a Rate Limiter**
17. **Designing a Notification system**

**The count is flagged honestly:** summaries variously describe the course as having **16, 17, 18, or 19 design problems** (the "definitive guide" site counts 18, "from a URL shortener to Uber and Ticketmaster"; the classic Educative-era description is ~16; the Design Gurus edition's 83 lessons include the same core). What is stable across all editions: **the nine verified problems above, plus the web crawler / typeahead / news feed / Yelp / Ticketmaster / Twitter-search / rate limiter / notification cluster.** Treat any exact total as edition-dependent and do not quote one in an interview context where precision matters.

### 3.2 The Problem Pattern (what all ~16 share)

Every problem lesson runs the same template — the 7-step framework made concrete:

1. **Requirements** — a short functional list and a non-functional paragraph (scale, latency, availability). Examples: URL shortener → "100M new URLs/day" class numbers; WhatsApp → "billions of messages per day."
2. **Estimation** — QPS, storage, bandwidth, always with assumptions stated.
3. **API design** — the endpoints that pin the requirements down.
4. **Data model** — entities, storage, partitioning direction.
5. **High-level design** — the canonical diagram for that system.
6. **Deep dive** — the problem's real payload: the two or three hard decisions unique to it.
7. **Wrap-up** — bottlenecks and trade-offs.

The pedagogical trick: because every problem runs the same skeleton, by problem #5 the skeleton is automatic, and by problem #10 the *component library* (load balancer, cache, queue, sharded store, CDN) is what varies — not the approach. That is the course's real deliverable.

### 3.3 The Problem Table (problem, key design, repo mapping)

| # | Problem | Key design move (the one-liner) | Repo mapping (deep-dive guide) |
|---|---------|--------------------------------|-------------------------------|
| 1 | URL Shortener | Key-to-ID mapping (base62 hash/snowflake); read-heavy cache; 301 vs 302; shard by key | [oracle_sharding_guide.md](oracle_sharding_guide.md) + [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) (partitioning and data models); [cloud_providers_guide.md](cloud_providers_guide.md) (components) |
| 2 | Pastebin | Same key-to-ID shape, plus content storage and expiry; read-heavy with a cache tier | [s3_architecture_guide.md](s3_architecture_guide.md) (content storage) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (async processing) |
| 3 | Instagram | Photo upload → object storage + CDN; async post-processing (thumbnails); feed by time + rank | [s3_architecture_guide.md](s3_architecture_guide.md) (photo storage) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (async thumbnail pipeline) |
| 4 | Tinder | Profile cards + swipe flow; geo-indexed discovery (lat/long bucketing); match notification | Geo design — see the Uber row (no dedicated repo geo guide; flagged); [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (match/notification events) |
| 5 | WhatsApp | Chat delivery semantics (at-least-once + dedupe), presence, group fan-out; WebSocket transport | [livekit_alternatives_guide.md](livekit_alternatives_guide.md) (real-time transport), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (delivery), [temporal_workflow_guide.md](temporal_workflow_guide.md) (delivery workflows); THE worked example in §8 |
| 6 | Netflix | Video → object storage + CDN tiering; adaptive bitrate; catalog metadata vs media planes | [s3_architecture_guide.md](s3_architecture_guide.md) (media storage), [cloud_providers_guide.md](cloud_providers_guide.md) (CDN), [event_stream_processing_guide.md](event_stream_processing_guide.md) (viewing analytics) |
| 7 | Twitter | Feed fan-out: push (on-write) for normal users, pull (on-read) for celebrities; timeline cache | [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (fan-out), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (notifications), [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) (search) |
| 8 | Uber | Geo-index (quadtree/grid) for drivers; ride-matching as an optimization; trip state machine | Geo — flagged (no dedicated repo geo guide; nearest: the Maps case in [google_system_design_interview_guide.md](google_system_design_interview_guide.md)); [event_stream_processing_guide.md](event_stream_processing_guide.md) (location stream), [temporal_workflow_guide.md](temporal_workflow_guide.md) (trip lifecycle), [late_arriving_data_guide.md](late_arriving_data_guide.md) (out-of-order location pings — light) |
| 9 | Dropbox | Chunked sync (delta), conflict resolution, metadata DB + object storage, notification for invalidation | [s3_architecture_guide.md](s3_architecture_guide.md) (object storage + sync) + [late_arriving_data_guide.md](late_arriving_data_guide.md) (out-of-order/conflict — light) |
| 10 | Web Crawler | URL frontier as a queue; politeness (per-domain rate); dedupe via checksums; distributed workers | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (frontier queue) + [temporal_workflow_guide.md](temporal_workflow_guide.md) (scheduling — light) |
| 11 | Typeahead Suggestion | Trie (or prefix-index) + top-K cache; precompute suggestions offline; fallback on miss | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) (search/prefix indexing) |
| 12 | News Feed | Same fan-out shape as Twitter; push vs pull by audience size; rank + cache timeline | [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (fan-out) |
| 13 | Yelp / Proximity | Geo-hashing or quadtree for nearby queries; read-heavy, cache tiles | Geo — flagged (same note as the Uber row) |
| 14 | Ticketmaster | Seat inventory as the crux: hold + expire seats atomically; queue for flash sales | [apache_seata_guide.md](apache_seata_guide.md) (distributed transactions — the hold/commit pattern) + [temporal_workflow_guide.md](temporal_workflow_guide.md) (booking lifecycle); banking/ booking-adjacent guides (light) |
| 15 | Twitter Search | Inverted index + real-time ingest pipeline; shard by time; top-K queries | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) (indexing) + [event_stream_processing_guide.md](event_stream_processing_guide.md) (ingest) |
| 16 | Rate Limiter | Token bucket / sliding window in Redis+Lua at the API gateway; client vs server rules | [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) — THE full worked deep-dive (§4.1) |
| 17 | Notification System | Fan-out via MQ; delivery channels (push/SMS/email); retry + dedupe; preference filtering | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) + [temporal_workflow_guide.md](temporal_workflow_guide.md) (retry workflows) |

**Two flags on the table:** (1) the geo problems (Tinder, Uber, Yelp) have **no dedicated geo deep-dive in the repo yet** — the nearest material is the Maps case in the Google interview guide and the location-stream material in the streaming guides; treat those rows as component-level mappings. (2) The Ticketmaster row maps by *analogy* (banking booking/inventory systems), not by an identical repo design — flagged as analogical.

### 3.4 The Problem Clusters (the real structure under the ~16)

The course's deeper lesson is that the ~16 problems are not sixteen independent designs — they are **seven clusters**, each with one crux family and one recurring component stack. Learn the clusters, and any new problem is just a cluster you already know wearing a different product name:

| Cluster | Problems | The crux family | The component stack |
|---------|----------|-----------------|---------------------|
| A. Read-heavy key-to-ID | URL shortener, Pastebin | Key generation, cache hit ratio, expiry | Sharded key store + cache ([oracle_sharding_guide.md](oracle_sharding_guide.md), [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md)) |
| B. Content / media | Instagram, Netflix, Dropbox | Object storage, CDN tiering, async post-processing | S3-style storage + queues ([s3_architecture_guide.md](s3_architecture_guide.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)) |
| C. Social / feed | Twitter, news feed, Instagram feed | Fan-out on write vs on read, timeline caching | Streaming + queues ([event_stream_processing_guide.md](event_stream_processing_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md)) |
| D. Real-time messaging | WhatsApp, notification system | Delivery semantics, presence, push vs pull | Sockets + MQ + workflows ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [livekit_alternatives_guide.md](livekit_alternatives_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md)) |
| E. Geo | Uber, Yelp, Tinder | Geo index, proximity queries, matching | Geo index + location stream (geo — flagged as a repo gap) |
| F. Search | Typeahead, Twitter search, web crawler | Indexing, top-K, ingest pipelines | Index/search stores ([ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md)) |
| G. Booking / inventory | Ticketmaster | Atomic holds, expiry, state machines | Transactions + workflows ([apache_seata_guide.md](apache_seata_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md)) |

**The cluster table's lesson:** the same five components (load balancer, cache, queue, sharded store, CDN) appear in every cluster; what varies is the crux. An interviewer who asks "design a parking-lot app" is handing you cluster E (geo) + cluster G (inventory); "design a comments service" is cluster C (fan-out) wearing a new name. This is the anti-memorization reading of the course (§1.7) — and the reason the §3.3 table's "key design move" column is the study cheat sheet.

---

## 4. The Problem Deep-Dives

Three problems deserve more than a table row: the **rate limiter** (because the repo owns a full deep-dive), **WhatsApp** (the chat design — the most-recycled messaging question), and **Uber** (the geo design — the one the course is most famous for teaching well).

### 4.1 The Rate Limiter (cross-ref: [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md))

Grokking's rate-limiter problem is the course's most-copied chapter — every competitor course has one now, and the repo has **the** worked deep-dive in [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (algorithms, the distributed-consistency problem, the Redis architecture with INCR/Lua/Cluster, key design decisions, performance, and a worked bank-API-gateway example).

**The Grokking shape of the problem:** rate limiting as an interview question is deliberately *narrow* — the crux is entirely in the deep-dive. The 7 steps compress to: requirements (e.g., "limit to 100 calls/sec per client, reject or queue beyond that"), estimation (how many rules, how many requests/sec the limiter must sustain), API/data (the rules table: client_id → limit window), high-level (limiter sits in front of the API servers — a gateway or middleware), and then **the deep dive is the whole interview**: which algorithm (token bucket vs leaky bucket vs fixed window vs sliding window/log), where the state lives (in-memory vs Redis), how you make it distributed and consistent (Redis INCR + Lua for atomicity), and what you do at the edge (client-side limits, CDN-level).

**What the repo guide adds:** the course teaches the algorithm zoo and the Redis pattern; the repo guide adds the distributed-consistency analysis (why naive Redis read-then-write races under load), the Lua-scripting fix, cluster partitioning of the counter space, and a production-shaped worked example. **Reading path:** Grokking's chapter for the shape → [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) for the depth an interviewer's second and third questions will probe. The Xu companion maps its Ch. 4 to the same guide — the rate limiter is the one problem where all three resources (Grokking, Xu, repo) converge on a single deep-dive.

### 4.2 WhatsApp: The Chat Design (verified shape)

Grokking's WhatsApp problem is the canonical **chat/messaging design** — and the single most-recycled messaging interview question (the Xu book's Ch. 12 "Design a Chat System" is the same problem in book form; the repo's [livekit_alternatives_guide.md](livekit_alternatives_guide.md) covers the real-time transport layer).

**The Grokking shape of the problem:**

- **Requirements:** 1:1 and group chat (group size matters — small groups ~10s, large groups ~1000s), message delivery guarantees (at-least-once), read receipts / last-seen presence, media sharing (photos/videos), push notifications for offline users, optional end-to-end encryption. Scale: billions of messages/day (the course's numbers are interview-shaped; flagged).
- **Estimation:** messages per day → peak QPS; storage per message (text is tiny, media dominates); connection count for presence (long-lived WebSocket connections per online user).
- **API design:** `sendMessage(userId, chatId, content)`, `getMessages(chatId, cursor, limit)` (cursor pagination), `createGroup(...)`, presence/typing events as push messages over the socket.
- **Data model:** users, chats (conversations), chat_members, messages (message_id, chat_id, sender_id, content, timestamp — the hot table, sharded by chat_id), message_status (delivered/read per member). Media metadata in the DB, media bytes in object storage.
- **High-level design:** clients → load balancer → chat service (WebSocket gateway / long-polling) → message store (sharded DB) → message queue → push notification service; presence service; media service → object storage → CDN.
- **Deep dive — the four cruxes:** (1) **delivery semantics** — push (WebSocket) vs pull (long-polling); at-least-once with per-message IDs + client-side dedupe; ordering via per-chat sequence numbers; (2) **presence/last-seen** — heartbeat-based online detection, last-seen persistence, fan-out of presence changes to friends; (3) **group chat** — fan-out on write (send once, duplicate per member via queue) for small groups vs fan-out on read (store once, pull per member) for large groups — the exact trade-off that defines the design; (4) **media** — upload to object storage, thumbnail generation async, CDN delivery, short-lived URLs.
- **Wrap-up:** hot chats (sharding by chat_id, sequence-number coordination), presence fan-out cost, offline delivery (store-and-forward), encryption's impact on server-side features (search, thumbnails).

**What the repo guides add:** [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) for the delivery-guarantee mechanics (at-least-once, dedupe, retries — the difference between "the message arrived" and "the message arrived exactly once"), [livekit_alternatives_guide.md](livekit_alternatives_guide.md) for the real-time transport layer the course hand-waves, [temporal_workflow_guide.md](temporal_workflow_guide.md) for durable delivery workflows, and [late_arriving_data_guide.md](late_arriving_data_guide.md) (light) for out-of-order messages and watermark-style sequencing. **§8 is this problem worked end-to-end as the full 7-step example.**

### 4.3 Uber: The Geo Design (verified shape)

Grokking's Uber problem is the canonical **geospatial design** — and the course's most distinctive contribution to the genre (the Xu book would later clone it as "Design a Ride-Sharing Service" in Volume 2's proximity chapters; the repo has no dedicated geo guide yet — flagged).

**The Grokking shape of the problem:**

- **Requirements:** driver location reporting, passenger ride request → match with nearby drivers, ride lifecycle (request → match → pickup → trip → payment → complete), ETA, surge pricing (often scoped out), offline drivers. Scale: millions of drivers/riders, location pings every few seconds.
- **Estimation:** pings per second (drivers × ping rate — this dominates); request QPS; matching scope (drivers within radius); storage of location history.
- **API design:** `updateDriverLocation(driverId, lat, lng)`, `requestRide(riderId, pickup, dropoff)`, `matchRide(requestId, driverId)`, `getRideStatus(rideId)`; all over WebSocket for location streaming.
- **Data model:** drivers (id, location, status, rating), riders, rides (request → trip state machine), location history (append-only time series), trip_events.
- **High-level design:** mobile clients → LB → location ingest service (WebSocket) → geo index (quadtree / geohash grid in memory) + location history store → ride-matching service → ride state machine (durable) → payment/notification via queue. The **geo index** is the star: a dynamic quadtree or geohash grid keeps "drivers within 5km" queries fast without scanning the planet.
- **Deep dive — the cruxes:** (1) **the geo index** — quadtree with grid-cell partitioning vs geohash; how drivers moving between cells are handled; keeping the index in memory and sharding it by region; (2) **ride matching as an optimization** — nearest-driver with constraints (rating, ETA), not just "closest euclidean point"; (3) **the location stream** — high-frequency pings, out-of-order/duplicate handling, sampling for the history store; (4) **the trip state machine** — durable, exactly-once transition handling (a booking lifecycle, same family as Ticketmaster and banking workflows).
- **Wrap-up:** hot regions (cell splitting), index consistency (a driver in two cells transiently), location privacy (scoped data), offline drivers.

**What the repo guides add:** the geo-index crux has **no dedicated repo deep-dive (flagged)** — nearest material: the Maps case in [google_system_design_interview_guide.md](google_system_design_interview_guide.md); the location stream maps to [event_stream_processing_guide.md](event_stream_processing_guide.md) (high-frequency ingest) and [late_arriving_data_guide.md](late_arriving_data_guide.md) (out-of-order pings, light); the ride lifecycle maps to [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable state machines); the payment tail maps (lightly) to the [banking/](../banking/) guides.

### 4.4 The Deep-Dive Table

| Problem | The crux (what the deep dive must nail) | Grokking's answer in one line | Repo deep-dive |
|---------|----------------------------------------|-------------------------------|----------------|
| Rate Limiter | Algorithm choice + distributed state | Token bucket / sliding window in Redis+Lua at the gateway | [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) — THE full deep-dive |
| WhatsApp (chat) | Delivery semantics, presence, group fan-out | WebSocket push + at-least-once + dedupe; fan-out on write for small / on read for large groups | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [livekit_alternatives_guide.md](livekit_alternatives_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md) |
| Uber (geo) | Geo index + ride matching + trip state machine | Quadtree/geohash in-memory index; nearest-with-constraints matching; durable trip lifecycle | Geo flagged (no dedicated guide); [event_stream_processing_guide.md](event_stream_processing_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md) |

---

## 5. The Comparison: Grokking vs the Xu Insider's Guide

### 5.1 Why They Get Compared

[system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) is this repo's companion to **Alex Xu's "System Design Interview – An Insider's Guide"** (2020, ByteByteGo) — the two resources are constantly compared because they occupy the same niche, teach the same problems, and are the two most-recommended system-design-prep resources in the world. **The relationship in one sentence: Grokking is the interactive course that started the genre (~2018); Xu is the book that systematized the genre (2020) — same DNA, different format, slightly different framework granularity.** The Xu companion's §3.1 even records the misattribution this overlap causes: Pastebin is famously a *Grokking* problem, and it gets attributed to Xu's book so often that the Xu companion has to correct it.

### 5.2 The Framework Difference (verified)

| Aspect | Grokking (Educative/Design Gurus) | Xu Insider's Guide (Volume 1) |
|--------|-----------------------------------|-------------------------------|
| Format | Interactive course (~2018) | Book (2020, ISBN 979-8664653403, ~320 pp) |
| Framework | **7 steps** — requirements, estimation, API design, data model, high-level design, deep dive, wrap-up | **4 steps** — requirements/scope, high-level design + buy-in, deep dive, wrap-up |
| Framework granularity | API design and data model are *explicit* steps (can't be skipped) | API/data-model concerns are folded into Steps 1-2 (interviewer-led) |
| Problem catalogue | ~16 problems (verified core: URL shortener → Dropbox; see §3.1) | 12 case studies (rate limiter → Google Drive; consistent hashing + KV store + unique ID as technique chapters) |
| Overlap | Both teach: rate limiter, URL shortener, chat/WhatsApp, news feed/Twitter, Dropbox/Drive, video (Netflix vs YouTube) | Same set — the overlap is ~60% of each catalogue |
| Unique to each | Tinder, Uber, Yelp, Ticketmaster, web crawler, typeahead, Twitter search, notification, Pastebin, Instagram | Consistent hashing, KV store, unique ID generator, notification system, Google Drive |
| Extras | Fundamentals section (building blocks); quizzes; Modern edition adds mock interviews | Ch. 1-2 (scale + estimation), Ch. 16 "Lessons Learned" (interview dynamics) |
| Theory depth | Shallow — hand-waves consistency/consensus | Also shallow — a step above Grokking on estimation math, still far below DDIA |

### 5.3 The Practical Verdict (the repo's position)

- **Both, in order: Grokking first, Xu second.** Grokking's interactive format and its explicit API/data-model steps make it the better *first* exposure for a beginner — you see the skeleton applied over and over. Xu's book is the better *polish* resource: its Ch. 3 tips and Ch. 16 "Lessons Learned" are the best-written interview-dynamics advice in the genre, and its estimation chapter is stronger.
- **If you only have time for one catalogue, they overlap enough that it doesn't matter which problems you drill — the component library is the same.** What differs is the framework granularity: candidates who learned Grokking's 7 steps tend to produce more complete early answers (they remember the API and data-model steps); candidates who learned Xu's 4 steps tend to be more fluid conversationalists (fewer steps to recite mechanically). The ideal is the hybrid: Grokking's completeness + Xu's conversational polish.
- **Both share the same critiques** (cookie-cutter answers, light theory, memorization risk) and both are fixed the same way: pair them with [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (depth) and the repo's component deep-dives (working detail). This companion and the Xu companion are siblings; the §6 mapping table and the Xu companion's §6 mapping table cross-reference each other's problems.

### 5.4 One Problem, Two Frameworks (the URL shortener, side by side)

The clearest way to feel the 7-step vs 4-step difference is one problem through both. Take the URL shortener — the genre's entry problem:

| Phase | Grokking (7 steps) | Xu (4 steps) |
|-------|--------------------|--------------|
| Scope | Step 1: requirements — features (shorten, redirect, expiry, analytics?) + non-functional (100M new URLs/day, 10:1 read:write) | Step 1: same scope questions — the frameworks agree entirely here |
| Sizing | Step 2: estimation — write QPS, read QPS, storage/year, key space | Step 1.5: same math, inside the same step |
| Interface | Step 3: API — `createURL(...)`, `deleteURL(...)`, `GET /{key}` — *explicit step* | Folded into the design conversation — the candidate is expected to raise the API surface unprompted |
| Data | Step 4: data model — key store, shard by key, cache tier — *explicit step* | Folded into the high-level design — schema sketched with the diagram |
| Shape | Step 5: high-level design — LB → app servers → cache → key store | Step 2: high-level design + buy-in — same diagram |
| Depth | Step 6: deep dive — base62 vs hash keys, 301 vs 302, cache hit ratio, key collision, expiry sweep | Step 3: deep dive — the *same* cruxes; Xu's case study chapter leads the interviewer to them |
| Close | Step 7: wrap-up — bottlenecks (hot keys, cache stampede), trade-offs | Step 4: wrap-up — same content |

**What the side-by-side shows:** both frameworks converge on the same cruxes and the same diagram; the difference is *where the safety nets live*. Grokking's explicit API and data-model steps force the candidate to touch the interface and schema even under pressure — a candidate who skipped Step 4 would never have said "shard by key, not by user" before drawing. Xu's compression rewards conversational fluency but trusts the candidate to volunteer the same material. For a candidate who freezes under pressure, the 7-step granularity is the safer skeleton; for a candidate who rambles, the 4-step compression is the cleaner shape. **The hybrid — Grokking's checklist internalized, Xu's conversational delivery — is the genre's best answer.**

---

## 6. The Repo Mapping: Chapter-to-Guide

### 6.1 The Chapter-to-Guide Table (the cross-refs)

This is the heart of the companion: every Grokking chapter/problem mapped to the repo guide that deep-dives it. The course gives the shape; these guides give the depth an interviewer's follow-up questions will probe.

| Grokking section / problem | Repo guide (the deep-dive) | What the guide adds |
|---------------------------|---------------------------|---------------------|
| Fundamentals (building blocks) | [cloud_providers_guide.md](cloud_providers_guide.md) | The component tour in production shape (LB, cache, CDN, storage tiers) |
| Fundamentals (theory) | [ddia_study_companion_guide.md](ddia_study_companion_guide.md) | The "why" — consistency, replication, partitioning, transactions (DDIA Ch. 1-9) |
| Fundamentals (partitioning) | [oracle_sharding_guide.md](oracle_sharding_guide.md) + [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) | Sharding in practice; NoSQL data-modeling trade-offs |
| Rate Limiter | [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) | THE full worked deep-dive: algorithms, Redis+Lua, distributed consistency, bank gateway example |
| URL Shortener | [oracle_sharding_guide.md](oracle_sharding_guide.md) + [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) | Key distribution and data-model choices for the key-to-ID store |
| Pastebin | [s3_architecture_guide.md](s3_architecture_guide.md) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) | Content storage, expiry, async processing |
| Instagram | [s3_architecture_guide.md](s3_architecture_guide.md) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) | Photo/media storage; async thumbnail pipelines; queue reliability |
| Tinder | Geo — flagged (no dedicated guide) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) | Match/notification events; geo rows need a future geo guide |
| WhatsApp | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [livekit_alternatives_guide.md](livekit_alternatives_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md) | Delivery guarantees, real-time transport, durable delivery workflows (§4.2, §8) |
| Netflix | [s3_architecture_guide.md](s3_architecture_guide.md) + [cloud_providers_guide.md](cloud_providers_guide.md) + [event_stream_processing_guide.md](event_stream_processing_guide.md) | Media storage/CDN; viewing-analytics streams |
| Twitter | [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) | Fan-out mechanics, streaming brokers, notification tail |
| Uber | Geo — flagged + [event_stream_processing_guide.md](event_stream_processing_guide.md) + [temporal_workflow_guide.md](temporal_workflow_guide.md) + [late_arriving_data_guide.md](late_arriving_data_guide.md) (light) | Location stream, trip state machine, out-of-order pings (§4.3) |
| Dropbox | [s3_architecture_guide.md](s3_architecture_guide.md) + [late_arriving_data_guide.md](late_arriving_data_guide.md) (light) | Object storage, sync, conflict/out-of-order handling |
| Web Crawler | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) + [temporal_workflow_guide.md](temporal_workflow_guide.md) (light) | Frontier queue reliability; scheduling |
| Typeahead Suggestion | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) | Prefix/search indexing for the top-K cache |
| News Feed | [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) | The fan-out deep-dive (push vs pull, timeline cache) |
| Yelp / Proximity | Geo — flagged (same as Uber row) | Needs a future geo guide |
| Ticketmaster | [apache_seata_guide.md](apache_seata_guide.md) + [temporal_workflow_guide.md](temporal_workflow_guide.md) | Seat hold/commit as distributed transactions; booking workflows (analogical — flagged) |
| Twitter Search | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) + [event_stream_processing_guide.md](event_stream_processing_guide.md) | Indexing + real-time ingest |
| Notification System | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) + [temporal_workflow_guide.md](temporal_workflow_guide.md) | Delivery, retry, dedupe, channel fan-out |
| Grokking Advanced (KV store) | [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) + [ddia_study_companion_guide.md](ddia_study_companion_guide.md) | Dynamo-style AP design; replication/partitioning theory |
| Grokking Advanced (messaging/streaming) | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) + [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) | The complete MQ/streaming stack |
| Grokking Advanced (distributed file storage) | [s3_architecture_guide.md](s3_architecture_guide.md) | Object-storage architecture |
| "Design in my domain" variant | [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) + the [banking/](../banking/) guides | Applying the framework to the bank's systems (lightly) |

### 6.2 The Suggested Reading Path

The repo's system-design series, in the order a candidate should consume it:

1. **[cloud_providers_guide.md](cloud_providers_guide.md)** — the component vocabulary (Grokking's fundamentals, in production shape). One sitting.
2. **This companion, §2** — the 7-step framework and its time budget. The skeleton.
3. **The two companions for theory and practice:** [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (the "why" — read alongside, chapter by chapter) and [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (the Xu book — same problems, book format, best interview-dynamics advice).
4. **The four deep-dives that carry the most interview weight:** [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (THE worked deep-dive), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (delivery guarantees — appears in half the designs), [s3_architecture_guide.md](s3_architecture_guide.md) (storage — appears in the other half), [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (the fan-out designs).
5. **[google_system_design_interview_guide.md](google_system_design_interview_guide.md)** and **[ml_system_design_interview_guide.md](ml_system_design_interview_guide.md)** — the interview-shaped versions of everything above (§7).
6. **The worked example in §8** — rehearse it out loud, timed, twice.
7. The optional tail: [temporal_workflow_guide.md](temporal_workflow_guide.md), [apache_seata_guide.md](apache_seata_guide.md), [late_arriving_data_guide.md](late_arriving_data_guide.md), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md), [livekit_alternatives_guide.md](livekit_alternatives_guide.md), and the [banking/](../banking/) guides — the "second question" depth and the "design in my domain" variant.

### 6.3 The Four-Week Study Plan (course + repo, week by week)

A concrete four-week plan combining the course with the repo's guides — the practical version of the three passes in §1.9:

| Week | Focus | Resources | Exit test |
|------|-------|-----------|-----------|
| 1 | Framework + fundamentals | Course fundamentals + framework sections; this guide §1-2; [cloud_providers_guide.md](cloud_providers_guide.md); [ddia_study_companion_guide.md](ddia_study_companion_guide.md) Ch. 1-2 | Name the 7 steps, their time budgets, and the ~7 recurring components without looking |
| 2 | Clusters A-B (key-to-ID, media) | URL shortener, Pastebin, Instagram, Dropbox lessons; [oracle_sharding_guide.md](oracle_sharding_guide.md), [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md), [s3_architecture_guide.md](s3_architecture_guide.md) | Rebuild the URL-shortener diagram from memory, sharding key included |
| 3 | Clusters C-D (feed, real-time) | Twitter, news feed, WhatsApp, notification, rate limiter lessons; [event_stream_processing_guide.md](event_stream_processing_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md) | Explain push vs pull fan-out and at-least-once delivery to a colleague in 5 minutes |
| 4 | Clusters E-G + the interviews | Uber, Yelp, typeahead, Ticketmaster lessons; [google_system_design_interview_guide.md](google_system_design_interview_guide.md), [ml_system_design_interview_guide.md](ml_system_design_interview_guide.md); rehearse §8 twice, timed | Deliver the §8 WhatsApp walkthrough in 45 minutes, on a whiteboard, without notes |

Daily habit through all four weeks: **one 30-minute whiteboard drill** — pick a problem at random, run the 7 steps on paper, then check the course's walkthrough for the crux you missed. The redo is where the learning happens.

---

## 7. The Interview Value

### 7.1 What the Course Is Worth in an Actual Interview

The course's interview value is real but bounded — the honest framing the repo's other companions share: **Grokking teaches the shape of a passing answer; it does not teach the depth under it.** What it buys you, verified by the framework alignment in the repo's interview guides:

- **The Google interview** ([google_system_design_interview_guide.md](google_system_design_interview_guide.md)): Google's rubric — requirements → estimation → components → deep-dive → trade-offs — is the same family as Grokking's 7 steps. A Grokking graduate walks in knowing the choreography (clarify, size, shape, deepen, wrap) and the level expectations (L4/L5/L6 gating). What the course does *not* give you is Google-scale math fluency (the Google guide's scale-math section) and the Google-stack specifics (GFS/Bigtable/Spanner) — those come from the repo guide.
- **The ML system design interview** ([ml_system_design_interview_guide.md](ml_system_design_interview_guide.md)): MLSD interviews (Chip Huyen 2022 / Xu-Aminian 2023 frameworks) share the clarify-first structure but replace the API/data-model/deep-dive steps with data → features → model → training → serving → evaluation → monitoring. Grokking gives you the *interview rhythm*; the ML guide gives you the ML-specific steps and case studies (recommendation, ranking, fraud).
- **The theory interview** ([ddia_study_companion_guide.md](ddia_study_companion_guide.md)): DDIA is the depth layer. Interviewers who probe "what does linearizable actually buy you here?" or "why quorum reads?" will catch the Grokking-level answer. The pairing rule: Grokking for the skeleton, DDIA for the follow-up.
- **The banking variant**: the same 7 steps are a design-document skeleton at Cymbal Bank, and the worked problems are the same shapes as banking systems (chat → trader chat, notification → alerts, Dropbox → document vault, Ticketmaster → booking inventory) — see the [banking/](../banking/) guides.

### 7.2 The Interview Table

| Interview / context | What Grokking contributes | What you must add (repo guide) |
|---------------------|---------------------------|--------------------------------|
| Google system design round (L4-L6) | The 7-step choreography; the ~16 problems as rehearsal material | [google_system_design_interview_guide.md](google_system_design_interview_guide.md) — scale math, level expectations, Google stack |
| ML system design round | The clarify-first rhythm; interview communication habits | [ml_system_design_interview_guide.md](ml_system_design_interview_guide.md) — data/model/serving steps, ML case studies |
| Depth probing (any round) | The vocabulary to state trade-offs | [ddia_study_companion_guide.md](ddia_study_companion_guide.md) — consistency, replication, transactions |
| Component follow-ups (the "second question") | The problem-level answers | [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [s3_architecture_guide.md](s3_architecture_guide.md), [event_stream_processing_guide.md](event_stream_processing_guide.md) |
| Practice / rehearsal | The §8 worked example, timed | The Xu companion's worked example ([system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) §8) — same problem, second pass |
| "Design in my domain" (banking) | The framework applied to unfamiliar systems | [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) + the [banking/](../banking/) guides |

### 7.3 The Rehearsal Loop (how to practice with the course)

The course gives you the walkthroughs; the practice method that converts them into interview skill is the **rehearsal loop** — the same discipline the genre's best guides converge on:

1. **Blind attempt (45 minutes).** Pick a problem the course covers but *don't look at its walkthrough*. Run the 7 steps on a whiteboard against a timer. Narrate out loud — the narration is the skill being trained, not the diagram.
2. **Compare (15 minutes).** Open the course's walkthrough. Mark where your design diverged: the crux you missed, the step you skipped (most people skip Step 4, the data model — the course made it explicit for a reason), the trade-off you failed to name.
3. **Redo (30 minutes).** Redo the same problem immediately, incorporating the gap. **The redo is where the learning happens** — the first pass builds the memory trace, the redo builds the habit.
4. **Calibrate with §8.** Deliver the WhatsApp worked example from this companion as the control run: if you can hold the shape of §8 under a timer, you can hold it under an interviewer.

Do this for one problem per day in weeks 2-4 of the §6.3 plan. Three problems in — not before — start timing yourself strictly; the first few attempts are about completeness, not speed. The exit criterion for the whole loop: you can run "clarify → estimate → API → data model → shape → deepen → wrap" on a question the course never covered (try "design a parking-lot app" — clusters E + G from §3.4) and still name two trade-offs in the wrap-up.

---

## 8. The Worked Example: WhatsApp, the Full 7-Step Design

This is the course's method applied end-to-end: the full 7-step design of **WhatsApp-style chat** — the problem worked in §4.2, now delivered the way an interview answer should be delivered. Rehearse it out loud, against a timer (45 minutes), with a whiteboard. *(Scale numbers are interview-shaped approximations in the Grokking style — flagged; the design decisions, not the numbers, are the gradeable content.)*

### Step 1 — Requirements Clarifications (8-10 minutes)

**Functional requirements:**
- 1:1 messaging and group messaging (groups up to a few hundred members for the core design).
- Delivery guarantees: at-least-once delivery, with read receipts (delivered/read states).
- Presence: online/offline status and last-seen; typing indicators (nice-to-have, explicit).
- Media sharing: photos, videos, documents; thumbnails for images.
- Push notifications for offline recipients.
- Out of scope (stated explicitly): end-to-end encryption internals (assume a client-side encryption layer exists — the server sees ciphertext), multi-device sync, voice/video calls, broadcast lists.

**Non-functional requirements:**
- Scale: assume **~1B users, ~100-200M DAU, ~50-100B messages per day** (interview-shaped; correct me if that's off).
- Availability over consistency for message *delivery* (a message must not be lost); strong ordering *per chat*.
- Latency: message send-to-deliver under ~1s online; presence updates near-real-time.
- Consistency: messages must be ordered per conversation; cross-device consistency is out of scope.

### Step 2 — Back-of-the-Envelope Estimation (3-5 minutes)

- 100B messages/day ÷ 86,400s ≈ **~1.2M messages/sec average**; peak ~2-3x → **~3M msg/sec**.
- Text: ~100 bytes avg → ~10 TB/day → ~3.6 PB/year (before replication). Media dominates: assume 10% of messages carry media, ~1MB avg → ~10 PB/year → object storage, not the DB.
- Connections: 150M DAU, ~20% online at peak → ~30M concurrent WebSocket connections — the gateway tier must be connection-dense (this number drives the presence design).
- Storage growth: messages store ~30 days hot, then archive (course convention).

### Step 3 — API Design (3-5 minutes)

```
POST   /v1/messages            {chatId, senderId, content, mediaId?}  → {messageId, seq}
GET    /v1/chats/{chatId}/messages?cursor={seq}&limit=50              → messages[]
POST   /v1/chats               {name, memberIds[]}                    → {chatId}
WS     /v1/ws                  (auth handshake → push channel for: message, delivery, presence, typing)
POST   /v1/presence            {userId, status: online|offline, lastSeen}
POST   /v1/media               (multipart upload)                     → {mediaId, uploadUrl}
```

The WebSocket is the primary interface for delivery events; the REST surface covers history, groups, and media. (This is the step the course makes explicit — and the step candidates trained only on the Xu 4-step framework often compress.)

### Step 4 — Data Model (3-5 minutes)

- **users** (id, phone, name, avatar, last_seen) — SQL, rarely hot.
- **chats** (chat_id, type: 1:1|group, name, created_at).
- **chat_members** (chat_id, user_id, joined_at) — the join table that makes group fan-out enumerable.
- **messages** — the hot table: (message_id, chat_id, sender_id, content, media_id, seq, created_at). **Shard by chat_id** (or by hash of chat_id) — all reads/writes for a conversation land on one shard, which is what makes per-chat ordering possible. Columnar/NoSQL (Cassandra-style) or MySQL with sharding both work; the course's point is the *partitioning key decision*: chat_id, not user_id, because ordering is per-chat. **seq** is the per-chat sequence number assigned by the shard (the ordering crux).
- **message_status** (message_id, user_id, status: delivered|read) — one row per recipient; the read-receipt fan-out.
- **media_metadata** (media_id, chat_id, uploader, size, type, storage_key) — bytes live in object storage.

### Step 5 — High-Level Design (10-15 minutes)

```
Clients ──► Load Balancer ──► Chat Gateway (WebSocket, connection-dense)
                                 │
              ┌──────────────────┼───────────────────────┐
              ▼                  ▼                       ▼
        Presence Service    Chat Service          Media Service
        (heartbeat,          (send/receive,        (upload → S3-style
         last-seen)          seq assignment)        object storage + CDN)
              │                  │                       │
              ▼                  ▼                       ▼
        Presence store      Message store         Object storage (s3_architecture_guide)
        (Redis, TTL)        (sharded by chat_id)  + thumbnail worker via queue
                                 │
                                 ▼
                          Message Queue ──► Push Notification Service ──► APNs/FCM
```

Main flows: (1) **send** — client → gateway → chat service → assign seq on the chat's shard → persist → enqueue delivery → gateway pushes to online members via their sockets; (2) **receive online** — pushed over the WebSocket; (3) **receive offline** — message persisted, push notification sent, history served by `GET /messages` on reconnect; (4) **presence** — heartbeat every ~30s to the presence service, TTL expiry marks offline, last-seen persisted.

### Step 6 — Deep Dive (15-20 minutes) — the four cruxes

**(a) Delivery semantics (the core of the design).** At-least-once, not exactly-once: the server persists the message with its seq *before* enqueueing delivery; a client that receives a message it already has (by message_id + seq) dedupes. Ordering: the chat shard assigns seq monotonically per chat; clients render in seq order; gaps mean "fetch from history" — this is where [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) earns its place (the guide's at-least-once mechanics, dedupe keys, and retry semantics are exactly this design's tail). Why not exactly-once end-to-end? It needs server-side dedupe state per message and still breaks at the client; at-least-once + client dedupe is the industry shape.

**(b) Presence / last-seen.** Heartbeat-based: online = heartbeat within TTL (30-60s) in the presence store (Redis with TTL); last-seen = the last heartbeat timestamp, persisted on expiry. Fan-out: presence changes broadcast to the user's friends' sockets *at most once per state change* (debounced), or pulled on chat-open to avoid the broadcast storm at logon (the "everyone's friends just came online" problem). The course's lesson: presence is a *fan-out* design in disguise — same trade-off family as the news feed.

**(c) Group chat — fan-out on write vs on read.** Small groups (≤ ~100): **fan-out on write** — one send, the delivery service enqueues one copy per member (via the MQ), members get it on their sockets. Large groups: **fan-out on read** — store once, members pull history; the notification is a lightweight "new message" ping, not a full copy. The trade-off: write fan-out costs N deliveries per message (fine for small groups, wasteful for 1000-member groups) but gives push latency; read fan-out saves delivery volume but members must poll/refresh. The course's canonical answer: hybrid — write fan-out for small, read fan-out (or on-demand) for large, threshold by group size.

**(d) Media.** Upload → object storage; thumbnail generation as an async worker behind the queue (the Instagram pipeline, same shape); short-lived signed URLs for download; CDN in front for hot media; media metadata in the DB with an expiry sweep. Offline members get the notification with the mediaId, fetch on reconnect.

### Step 7 — Wrap-Up (3-5 minutes)

- **Bottlenecks:** (1) hot chats — a celebrity group shard is a single-writer hot spot; mitigation: split large groups to read fan-out, or shard messages by (chat_id, time-bucket) with per-bucket seq coordination; (2) presence fan-out at logon — mitigate with pull-on-open for last-seen and debounced broadcasts; (3) gateway connection density — the 30M-connection tier needs sticky routing and connection draining; (4) read-receipt fan-out — per-message status rows blow up for groups; mitigate with aggregated receipts ("read by 5/7").
- **Trade-offs made:** at-least-once vs exactly-once (accepted duplicates, bought reliability); push vs pull delivery (bought latency, paid in connection cost); write vs read fan-out (bought by group size); ordering per-chat rather than global (global ordering is not a product requirement).
- **With more time:** multi-device sync (last-read watermark per device), end-to-end encryption's impact on media thumbnails and search, message retention/archival, and the broadcast-list variant.

### The Lessons (what this walkthrough teaches)

1. **The 7 steps are a time budget, not a script** — the walkthrough spends ~20 of 45 minutes in Step 6, exactly where the points are.
2. **The API and data-model steps do the design work early** — the chat_id sharding decision in Step 4 is what makes Step 6's ordering answer possible.
3. **Every "product" question reduces to a small set of recurring cruxes** — delivery semantics, fan-out, presence — and each crux has a repo deep-dive ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [event_stream_processing_guide.md](event_stream_processing_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md)).
4. **Naming trade-offs beats adding components** — the Step 7 list is the senior signal; the course's wrap-up step exists to force it.
5. **Rehearsal transfers** — delivering this walkthrough out loud, timed, twice, is what makes the unfamiliar question survivable (the Pass-3 discipline from §1.9).

### 8.2 The Follow-Up Drill (what an interviewer's second question sounds like)

The worked example is the *first* answer; interviews are won on the **second** question. Here is the follow-up drill — the six probes this design most reliably attracts, with the answers the repo's deep-dives support:

| Interviewer's probe | The answer that holds | The repo backing |
|---------------------|-----------------------|------------------|
| "Why at-least-once and not exactly-once?" | Exactly-once needs server-side dedupe state and transaction boundaries across socket + store — expensive and still fragile at the client. At-least-once + per-message ID + client dedupe is the industry shape; rare duplicates are the accepted cost. | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) — the delivery-guarantee mechanics |
| "What happens if a chat shard dies?" | Replicate per shard (quorum reads/writes); failover promotes a replica; ordering resumes from the last durable seq — expect a brief gap, and surface it as a watermark so clients know history is incomplete until caught up. | [late_arriving_data_guide.md](late_arriving_data_guide.md) — watermarks and gap handling (light); [ddia_study_companion_guide.md](ddia_study_companion_guide.md) — replication |
| "A celebrity group with 1M members — now what?" | Flip to read fan-out: store once, members pull on notification ping; time-bucket the shard; aggregate read receipts ("read by 12,401") instead of per-member status rows. The threshold decision is *the* group-chat trade-off. | [event_stream_processing_guide.md](event_stream_processing_guide.md) — fan-out at scale |
| "It's 8am — 10M users just logged on. Presence?" | Heartbeats with TTL in Redis; debounce presence broadcasts (at most one state change per friend, plus pull-on-open for last-seen); region-shard the presence service. Presence is a fan-out design in disguise. | [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) — the Redis-at-scale patterns (same infra family) |
| "How do media uploads avoid blocking the chat?" | Async end-to-end: client uploads to a signed object-storage URL; the message carries only the mediaId; thumbnails/post-processing run as queue workers; CDN fronts hot media. Chat never waits on bytes. | [s3_architecture_guide.md](s3_architecture_guide.md) + [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) |
| "You scoped out encryption — what would it actually break?" | Server-side search, thumbnail generation, spam detection, and any content processing — the server only sees ciphertext. State the E2E assumption at requirements time and name what it costs; the design survives because delivery and ordering are metadata-level. | [ddia_study_companion_guide.md](ddia_study_companion_guide.md) — the honesty of naming trade-offs |

The drill's purpose: each probe is a *second question* that the 7-step first answer must survive — and each maps to a repo deep-dive. If you can answer these six after delivering the §8 walkthrough, you have passed from "knows the course" to "can hold the design" — which is what the interview is actually scoring.

---

## 9. The Summary: The Course in One Page

**What it is.** "Grokking the System Design Interview" is the interactive Educative course (authored by Fahim ul Haq, Educative's co-founder; ~2018 — flagged) that started the system-design-interview-prep genre. It teaches that open-ended design questions are scored against a recognizable shape: a **7-step framework** (requirements → estimation → API design → data model → high-level design → deep dive → wrap-up) applied to a catalogue of **~16 design problems** (URL shortener, Pastebin, Instagram, Tinder, WhatsApp, Netflix, Twitter, Uber, Dropbox, and the rest).

**The framework.** 7 steps, ~45-60 minutes: 5-10 minutes of requirements, 25-35 minutes of design + deep dive, 5 minutes of wrap-up (verified shape; step-level minutes flagged). The steps are a *collaboration protocol*, not a script — clarify (ambiguity), estimate (assumptions), interface + data (concreteness), shape (buy-in), deepen (expertise), wrap (trade-offs).

**The problems.** ~16 worked designs (count flagged by edition; nine verified: URL shortener through Dropbox). Every problem runs the same skeleton, so the real deliverable is the **component library** — load balancer, cache, queue, sharded store, CDN — not sixteen memorized diagrams.

**The comparison.** Grokking (course, 7 steps, ~2018) vs Alex Xu's Insider's Guide (book, 4 steps, 2020): same DNA, same ~60% problem overlap, different granularity — Grokking makes API design and data model explicit steps; Xu is the stronger book for interview dynamics and estimation math. Take both: Grokking first, Xu for polish.

**The repo pairing.** Every problem maps to a repo deep-dive: the rate limiter → [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (THE worked deep-dive); chat → [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) + [livekit_alternatives_guide.md](livekit_alternatives_guide.md); media/storage → [s3_architecture_guide.md](s3_architecture_guide.md); fan-out → [event_stream_processing_guide.md](event_stream_processing_guide.md) + [kafka_alternatives_guide.md](kafka_alternatives_guide.md); theory → [ddia_study_companion_guide.md](ddia_study_companion_guide.md). The course gives the shape; the guides give the depth.

**The interview value.** Grokking gives you the choreography for Google-style rounds (same framework family as [google_system_design_interview_guide.md](google_system_design_interview_guide.md)), the rhythm for ML rounds ([ml_system_design_interview_guide.md](ml_system_design_interview_guide.md)), and — for the banking technologist — a design-document skeleton whose worked problems are the same shapes as the bank's own systems.

**The final word.** Grokking is **the course that started the genre** — every competitor is a variation on its template, every critique of it (cookie-cutter, memorization-prone, light on theory) is a critique of the genre it created. Used honestly — as a framework to internalize, a component library to learn, and a set of shapes to rehearse — it remains the best single first resource in system design interview prep. Read it with the repo's deep-dives underneath, and you have the shape *and* the depth.

### 9.1 The Companion Quick-Reference (the whole course on one card)

The day-before-interview card — everything from the course that must be automatic:

| The 7 steps | Time (45-min round) | The one-line rule |
|-------------|---------------------|-------------------|
| 1. Requirements | 8-10 min | Ask, restate, write down — never design before scope is agreed |
| 2. Estimation | 3-5 min | State assumptions out loud; the number matters less than the assumption |
| 3. API design | 3-5 min | Define the endpoints — it catches missing requirements |
| 4. Data model | 3-5 min | Entities + storage + the partitioning key (it drives everything downstream) |
| 5. High-level design | 10-15 min | Boxes and arrows; get buy-in on the shape before detail |
| 6. Deep dive | 15-20 min | One or two cruxes, chosen with the interviewer — where the points are |
| 7. Wrap-up | 3-5 min | Bottlenecks, trade-offs, next steps — never end abruptly |

| The 7 clusters (the ~16 problems) | The crux to name |
|------------------------------------|------------------|
| A. Key-to-ID (URL shortener, Pastebin) | Key generation, cache hit ratio, expiry |
| B. Media (Instagram, Netflix, Dropbox) | Object storage + CDN + async post-processing |
| C. Feed (Twitter, news feed) | Fan-out on write vs on read |
| D. Real-time (WhatsApp, notification) | Delivery semantics, presence, push vs pull |
| E. Geo (Uber, Yelp, Tinder) | Geo index, proximity, matching |
| F. Search (typeahead, Twitter search, crawler) | Indexing, top-K, ingest |
| G. Booking (Ticketmaster) | Atomic holds, expiry, state machines |

| The 5 recurring components | Where they appear |
|---------------------------|-------------------|
| Load balancer | Every design |
| Cache | Read-heavy designs (A, C) |
| Queue | Async everything (B, D, G) |
| Sharded store | The hot table in every design |
| CDN | Media and global read paths (B) |

Plus the three habits that separate passing from failing: **state your assumptions, let the interviewer choose the deep dive, and name the trade-offs in the wrap-up.** Everything else is the repo's deep-dives underneath.

---

## 10. Verification and Claims-Status

This companion follows the repo's honesty convention: every claim is marked **verified** (confirmed against a source during research) or **flagged** (commonly cited but not officially confirmed, or qualitative).

### Verified

- **The course exists on Educative** — "Grokking the System Design Interview" (educative.io/courses/grokking-the-system-design-interview); Educative is an interactive course platform.
- **The author association** — Fahim ul Haq is Educative's co-founder & CEO; the course (and its "Modern" successor) is associated with him. The original content is also sold at Design Gurus as "the official DesignGurus course by the original creators" (Arslan Ahmad / original Design Gurus team). The lineage is flagged in §1.3 as edition-dependent.
- **The 7-step framework** — requirements clarifications, back-of-the-envelope estimation, system interface definition (API design), defining data model, high-level design, detailed design deep-dive, identifying and resolving bottlenecks (wrap-up) — confirmed against Design Gurus' published "7 Steps to Ace It" blog and their "Mastering System Design Interview in 7 Steps" PDF.
- **The time-allocation shape** — 45-60 minute rounds; first 5-10 min requirements, next 25-35 min design and deep-dives, last 5 min wrap-up (Design Gurus' interview guide). Step-level minute values are commonly-cited approximations (flagged).
- **The nine core problems** — URL shortener (TinyURL), Pastebin, Instagram, Tinder, WhatsApp, Netflix, Twitter, Uber, Dropbox — the canonical Grokking set, cited consistently across editions and reviews.
- **The related course** — "Grokking the Advanced System Design Interview" exists (Educative and Design Gurus), covering Dynamo-style key-value stores, distributed messaging/streaming, and distributed file storage.
- **The Modern successor** — Educative now serves "Grokking Modern System Design Interview for Engineers & Managers" at the original URL (204 lessons, 140 quizzes, 8 mock interviews, ~26 hours, "RESHADED" method per third-party reviews; led by Fahim ul Haq per Educative's comparison content).
- **The repo guides referenced in §6** — all exist in technology/ as of August 2026 (verified by direct inspection).
- **The Pastebin misattribution note** — Pastebin is a Grokking (and ByteByteGo-course) problem, frequently misattributed to Xu's book; the Xu companion ([system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) §3.1) records the same correction.

### Flagged (honestly unverifiable / qualitative)

- **Launch year 2018** — commonly cited as the course's release year; no official confirmation found. Treat as "~2018."
- **The problem count** — "~16 chapters/problems" is the classic description, but editions vary (16-19 cited; the Design Gurus edition lists 83 lessons total; the Modern rebuild restructured to 204 lessons). The nine core problems are verified; the total count is edition-dependent.
- **Reception and popularity** — "the #1 course" claims come from the creators' own marketing and from qualitative reviews; no published enrollment or sales figures exist. Any specific student count is unverifiable.
- **The critiques** (cookie-cutter answers, memorization risk, light theory, dated original edition) — qualitative, but consistent across reviews and echoed in the repo's other companions.
- **Scale numbers in the §8 worked example** (1B users, 50-100B messages/day, 30M concurrent connections) — interview-shaped approximations in the Grokking style, not audited WhatsApp statistics.
- **The "90% of points" paraphrase** (§2.4) — a commonly-cited paraphrase of the course's emphasis; interpretive, flagged.
- **The geo gap** — the repo has no dedicated geospatial design guide yet (the Uber/Yelp/Tinder rows map to component-level material and flag a future gap).

---

## 11. Glossary

- **Grokking** — The course's title verb (from Heinlein's *Stranger in a Strange Land*: to understand so deeply it becomes part of you). In interview-prep culture, "grokking" became the genre name: the course that started the system-design-prep genre, and the template every competitor copied.
- **Educative** — The interactive learning platform (educative.io) that hosts the course; founded by Fahim ul Haq. Courses render in the browser — lessons, diagrams, quizzes — no video required.
- **System design interview** — The open-ended interview round (usually 45-60 minutes) where a candidate designs the architecture of a software system (a chat app, a news feed, a ride-hailing service) and talks through it with an interviewer; scored on structure, trade-offs, and collaboration, not on a single correct answer.
- **Framework** — The repeatable step sequence the course teaches for attacking any system design question; the course's 7-step framework is its core deliverable.
- **7-step** — The course's framework: requirements, estimation, API design, data model, high-level design, deep dive, wrap-up (Step 7 is "identifying and resolving bottlenecks").
- **Requirements** — Step 1: clarifying functional features and non-functional targets (scale, latency, availability) before designing; the most-skipped and most-scored step.
- **Estimation** — Step 2: back-of-the-envelope math (DAU → QPS, storage/year, bandwidth) with assumptions stated out loud.
- **API design** — Step 3: defining the system's interface (endpoints, parameters, returns) to pin requirements down and catch missing functionality.
- **Data model** — Step 4: entities, relationships, storage choice (SQL vs NoSQL), and the partitioning key — the step that exposes read/write patterns.
- **High-level design** — Step 5: the box-and-arrow architecture (clients, LB, services, caches, stores, queues, CDN) agreed with the interviewer before detail is added.
- **Deep dive** — Step 6: one or two components examined in depth — the crux decisions (rate-limiter algorithm, fan-out strategy, delivery semantics) where most points are won.
- **Wrap up** — Step 7: naming bottlenecks, trade-offs, and next steps; the crisp recap that becomes the interviewer's notes.
- **URL shortener** — The classic read-heavy key-to-ID design problem (TinyURL): hash/base62 keys, cache-heavy reads, 301 vs 302 redirects, sharded key store.
- **Pastebin** — The text-storage design problem (key-to-ID shape plus content storage and expiry); famously a Grokking problem, frequently misattributed to Xu's book.
- **Instagram** — The photo-sharing design problem: object-storage uploads, CDN delivery, async thumbnail pipelines, feed by time/rank.
- **Tinder** — The dating-app design problem: profile cards, swipe flow, geo-indexed discovery, match notifications (geo flagged — no dedicated repo guide).
- **WhatsApp** — The chat/messaging design problem: WebSocket transport, at-least-once delivery with dedupe, per-chat ordering, presence, group fan-out; the §8 worked example.
- **Netflix** — The video-streaming design problem: media vs metadata planes, object storage + CDN tiering, adaptive bitrate.
- **Twitter** — The social-feed design problem: fan-out on write vs on read, timeline caching, notification tail.
- **Uber** — The ride-hailing/geo design problem: driver-location ingest, quadtree/geohash geo index, ride matching, durable trip state machine.
- **Dropbox** — The file-sync design problem: chunked sync, delta updates, conflict resolution, metadata DB + object storage.
- **Rate limiter** — The API-protection design problem: token bucket / sliding window algorithms, Redis+Lua distributed state at the gateway; the repo's THE worked deep-dive ([distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md)).
- **Chat** — The messaging design family (WhatsApp, Messenger, trader chat): delivery semantics, presence, group fan-out — §4.2 and §8.
- **Geo** — The geospatial design family (Uber, Yelp, Tinder): geo indexes, proximity queries, location streams — flagged as a repo gap awaiting a dedicated guide.
- **Fahim ul Haq** — Co-founder and CEO of Educative; the author the course is associated with (the Modern successor is led by him; the original content also lives at Design Gurus — see §1.3).
- **Interactive course** — The course format: in-browser lessons, click-through diagrams, quizzes, worked examples — the format that distinguished Grokking from books and video courses.
- **Companion** — The repo's guide pattern (see [management/the_managers_path_research.md](../management/the_managers_path_research.md)): a study companion that walks through a book or course, distills its framework and material, and maps it to the repo's deep-dives — this guide is the course companion for Grokking the System Design Interview, sibling to [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (Xu) and [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (DDIA).

---

*End of the Grokking the System Design Interview study companion. The course started the genre; the repo's deep-dives keep the depth honest. — Jack Liu Shurui, August 2026*
