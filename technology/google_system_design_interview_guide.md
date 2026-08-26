# System Design at Google: Engineering Peak for Interviews — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Professional Development / Career Series — System Design, Interview Preparation, Career Progression
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** technology/ (career-skills / system-design series)
> **Last Updated:** August 2026

---
> A comprehensive guide to the **Google system design interview** — the round that separates "can code" from "can build". It covers the Google interview landscape (the loop, the onsite, the round itself), the level expectations (L4 / L5 / L6), the classic design questions (Search, YouTube, Maps, Drive, Gmail, Spanner), a Google-flavored design framework (requirements → estimation → components → deep-dive → trade-offs), the scale math (QPS, storage, bandwidth — with worked numbers), the Google stack (GFS, Bigtable, Spanner), deep-dives of the classic questions, the interview day, a 4–6 week practice plan, and a full worked example (designing Google Search). The framing throughout is **"engineering peak"** — the idea that the interview is a performance event at which you must hit your peak state of structured, scaled, communicative thinking, not a trivia quiz.

**Audience:** software engineers, solution architects, and SREs preparing for Google (or Google-style) system design interviews; banking technologists (the Cymbal Bank context) who evaluate large-scale design skills or want to speak the FAANG design language; hiring managers designing design-round rubrics. The worked example deliberately uses **Google Search** — the canonical Google question — so the framework is exercised on the one case every interviewer knows.

**How to use this guide:**

- **Interview candidates**: read Sections 1–2 (landscape, levels) and 4–6 (framework, math, stack) first, then drill Section 7 (deep-dives) and 9 (practice plan), and finish with Section 10 (worked example) as a rehearsal script for the round.
- **Architects / hiring managers**: Sections 2 (what each level must show), 4 (the rubric-shaped framework), and 10 (what good answers sound like).
- **Everyone**: the glossary (Section 11) doubles as a design-vocabulary checklist — if a term is unfamiliar, that is a learning item, not trivia.

**Cross-references in this series** (this guide links to, rather than duplicates, the deep-dives): `distributed_rate_limiter_guide.md` (the just-added system-design deep-dive — the rate limiter is a perfect component to drop into a Google design answer), `ml_system_design_interview_guide.md` (the just-added ML design interview — same framework family, same practice-plan structure), `quantitative_developer_skillset_guide.md` (the career-skills framing for a banking technologist), `mathematics_for_ml_and_ds_study_notes.md` (the math prep for ranking/PageRank sections), `apache_seata_guide.md` (distributed transactions — a design-component example for Spanner-style questions), `event_stream_processing_guide.md` and the Kafka guides (streaming components — YouTube processing pipelines), `message_queue_data_loss_guide.md` (the just-added MQ reliability deep-dive — queue design in upload/transcoding pipelines), `cloud_providers_guide.md` and `s3_architecture_guide.md` (cloud components and object-storage design — Drive deep-dive), and the `banking/` guides (worked designs in the banking context — cross-ref lightly for the "design in my domain" variant).

**Verification notes (honesty policy):** facts in this guide were checked against primary or near-primary sources where possible (Google's own engineering publications — the GFS, Bigtable, and Spanner papers; developers.google.com's "How Search works"; Wikipedia's YouTube statistics; publicly aggregated interview reports). Where a claim is common knowledge but not cleanly verifiable — interview *frequency* statistics, exact loop composition per team, compensation-adjacent expectations, the "8.5 billion searches per day" figure (third-party reported, Google does not publish a number), and the term "engineering peak" itself (an interpretive framing, not Google terminology) — it is explicitly **flagged** rather than asserted as fact.

---

## Table of Contents

1. [The Google Interview Landscape](#1-the-google-interview-landscape)
2. [The Level Expectations](#2-the-level-expectations)
3. [The Classic Google Design Questions](#3-the-classic-google-design-questions)
4. [The Design Framework](#4-the-design-framework)
5. [The Scale Math](#5-the-scale-math)
6. [The Google Stack](#6-the-google-stack)
7. [The Question Deep-Dives](#7-the-question-deep-dives)
8. [The Interview Day](#8-the-interview-day)
9. [The Practice Plan](#9-the-practice-plan)
10. [The Worked Example: Designing Google Search](#10-the-worked-example-designing-google-search)
11. [The Summary and the Glossary](#11-the-summary-and-the-glossary)

---

## 1. The Google Interview Landscape

### 1.1 The Google Interviews: The Loop

The Google hiring process is famously team-independent: the loop is standardized and the interviewers are drawn from the broader engineering population, not the team you might join. The purpose is to keep the bar consistent — the same interviewer pool, the same rubric, the same hiring committee — so that "getting into Google" means passing a *company* bar, not charming one manager. *(Verified: the team-independent loop and hiring-committee review are consistently described in Google's own recruiting material and in aggregated interview reports.)*

The full pipeline, in order:

1. **Recruiter screen (30–45 min).** A phone conversation about background, role, level alignment, and logistics. Not technical in the coding sense, but you should be able to narrate your resume's system-design-relevant work (scale, ownership, trade-offs you made).
2. **Phone screens (1–2).** Technical, coding-heavy, 45–60 minutes each. For L5+ candidates one screen may be a light design or role-related screen. The purpose is filtering: candidates who cannot code fluently are stopped before the onsite.
3. **Onsite (4–5 rounds, virtual or in-person).** The core event — see §1.2.
4. **Hiring committee (HC).** All interview feedback — written by interviewers who each own a *signal*, not a vote on the whole candidate — is reviewed by a committee of senior engineers who did not interview you. The HC weighs the evidence against the target level (or the level the evidence supports) and issues a hire/no-hire/lean recommendation. *(Verified: the HC model is Google-specific and well documented; the exact thresholds are internal and unverified.)*
5. **Team matching.** A "yes" from the HC does not immediately mean an offer: you match with a team, often through a series of conversations (team matching calls), and the offer is finalized with that team's headcount. This is unusual among FAANG and worth knowing — the interview is *not* the last gate.

The **onsite loop** is the heart of it. A typical SWE onsite is **5 rounds × 45 minutes** (verified against 2024–2026 aggregated reports; some loops run 4 rounds, and L6 loops can run 5–6):

| Onsite round | What it tests | Typical content |
|---|---|---|
| Coding × 2 | Algorithmic fluency, data structures, code quality | LeetCode-hard-ish problems, often on a shared doc; one round may be "solve a realistic algorithmic problem from a Google product" |
| **System design × 1–2** | Architecture at scale, trade-offs, communication | "Design YouTube" / "Design Google Drive" / your own domain — the subject of this guide |
| Googleyness & Leadership (G&L) | Values fit, leadership signal, ambiguity handling | Behavioral questions; for L5+ this round also probes mentorship and influence |
| Role-related knowledge (RRK) | Depth in your discipline | Language/domain expertise, debugging scenarios, production incidents |

The exact mix depends on level: **L4** loops are coding-heavy with design often folded into a coding round or run as a lighter design session; **L5** gets a standalone system design round; **L6** typically gets **two to three** design rounds, one of which may be an ambiguous, product-flavored design with a leadership slant. *(Verified against 2026 aggregated interview-report breakdowns; the gist.github.com/techreign breakdown of 2,847 reports and tryexponent's loop description both confirm this shape. Exact round counts per team remain anecdotal.)*

### 1.2 The System Design Round

The **system design round** is a 45-minute (up to 60-minute) open-ended conversation. The interviewer gives you a one-line prompt — "design a service like Google Drive", "design a distributed cache", "design a system to serve search results" — and the rest is up to you: what to ask, what to build, where to go deep, what to trade off. *(Verified: 45 minutes is the standard length cited across Google interview accounts; some teams run 60 minutes for L6.)*

What makes Google's version distinctive:

- **Scale is the subject.** A design that works for 1,000 users is the *starting* point, not the answer. The interviewer will keep injecting scale ("now it's 100× the users", "now the index is 10× the size") to see where your architecture bends and breaks.
- **Reliability and the 9s.** Google-grade systems are expected to be available, self-healing, and failure-aware: replication, quorums, retries with backoff, graceful degradation. Interviewers probe what happens when your database dies, your region goes dark, or your job queue backs up.
- **The process is the product.** There is no single correct architecture. The interviewer scores *how you think*: do you clarify before designing? Do you estimate before choosing? Do you name trade-offs unprompted? Do you communicate continuously (think-aloud) rather than going silent to "solve"?
- **Your domain is fair game.** Google interviewers adapt prompts to your background — a candidate from banking may be asked to design a payment system, a settlement pipeline, or a fraud-scoring service. This is the "design in my domain" variant, and it is why the banking worked designs in this series matter even for a Google prep guide.
- **The SRE variant.** For Site Reliability Engineer roles the design round takes a reliability lens: designs must include SLOs, error budgets, monitoring, capacity planning, and failure modes (the older SRE designation was NALSD — Non-Abstract Large Systems Design — and the mindset persists). A "design YouTube" answer that omits monitoring and capacity math is a weaker SRE answer. *(Verified: SRE interview guides consistently describe reliability-weighted design rounds and a separate troubleshooting round.)*

The round is scored on the standard Google rubric signals — **General Cognitive Ability, Role-Related Knowledge, Leadership, Googleyness** *(verified as the publicly described rubric)* — but in practice the design round is where **cognitive ability + role-related knowledge** are measured together: can you reason from first principles about a system you have never built?

### 1.3 The Preparation Context: Roles and the Engineering-Culture Frame

**The roles.** The round exists for the roles whose daily job is architecture at scale:

- **SWE (Software Engineer)** — the generalist track. Expects end-to-end product design: APIs, data models, services, storage, caching, and the trade-offs between them. Depth expected rises with level: L4 designs a well-scoped component correctly; L5 designs the whole product with justified choices; L6 designs under ambiguity with org-wide consequences.
- **SRE (Site Reliability Engineer)** — the reliability track. Expects the same architecture skills *plus* operational fluency: SLOs, error budgets, incident response, capacity, and the "what fails and what do we do" story for every component. A design answer that ends at "and it's deployed" is incomplete; the SRE version ends at "and here is how we know it's healthy and how we recover."
- **TPM / SET (and the ML-adjacent tracks)** — lighter or specialized variants. MLE interviews include ML system design (see `ml_system_design_interview_guide.md` for the full treatment); TPM loops use a product-flavored design conversation.

**The culture frame: "engineering peak".** *(Flag: "engineering peak" is the interpretive framing of this guide, not Google terminology. Google does not publish a term for the ideal interview state.)* The frame is borrowed from sports psychology: the interview is a **peak-performance event** — a short, high-stakes window in which you must summon your best structured thinking on demand. What that means in practice:

- **Peak performance is a state, not a trait.** It is reachable through preparation (so the cognitive load is spent on the problem, not on recall), warm-up (a light design drill the morning of), and process (a checklist you can run on autopilot).
- **The state has three components:** *calm* (no panic when the interviewer adds scale — the framework catches you), *clarity* (you always know what you are doing and why), and *speed* (you make decisions, name trade-offs, and move — you do not stall waiting for the perfect answer).
- **Google's own engineering culture rewards this.** The SRE book's "Hiring, training, and maintaining" chapter, the design-review culture, and the "disagree and commit" ethos all value engineers who can think clearly under ambiguity, communicate decisions with reasons, and keep the system (and the conversation) moving.

The whole of this guide is built to get you to that peak state: the framework (§4) is your autopilot, the math (§5) is your fluency, the stack (§6) is your vocabulary, the deep-dives (§7) are your patterns, and the practice plan (§9) is your training regimen.

### 1.4 The Landscape Table

| Aspect | Description |
|---|---|
| **The loop** | Recruiter screen → 1–2 phone screens → onsite (4–5 rounds) → hiring committee → team matching. Team-independent by design. |
| **The onsite** | ~5 × 45-min rounds: 2 coding, 1–2 system design, 1 Googleyness & Leadership, 1 role-related knowledge. L6 runs 5–6 rounds with more design. |
| **The system design round** | 45–60 min, open-ended, scale-obsessed, reliability-aware; scores the *process* (clarify, estimate, design, trade-off, communicate), not the artifact. |
| **The roles** | SWE (generalist, product-scale design), SRE (reliability-weighted design, SLO/error-budget fluency), MLE/TPM variants (ML design, product design). |
| **The culture** | "Engineering peak" — an interpretive frame: the interview is a peak-performance event; calm + clarity + speed, driven by a rehearsed framework. |
| **The bar** | Company-wide, hiring-committee-reviewed, level-calibrated. You are compared against the target level's expectations (§2), not against other candidates. |
| **The resources** | Published and public: Alex Xu's *System Design Interview* (Vols 1–2) and the ByteByteGo materials (cross-ref `ml_system_design_interview_guide.md` §16, which reviews the same canon); Google's own papers (GFS, Bigtable, Spanner, MapReduce); aggregated report sites (igotanoffer, LeetCode discuss, teamblind) for question lists — treat all third-party statistics as anecdotal. |

### 1.5 The Google Engineering Culture: What It Implies for Interviews

Google's engineering culture is unusually *visible* — most of its infrastructure philosophy is published (the SRE book, the design-doc culture, the papers) — and the system design round is a cultural artifact as much as a technical exam. Four cultural facts shape what interviewers reward:

- **Design docs are the unit of design.** Google engineers write design docs (a one-pager to a 20-pager: context, goals/non-goals, alternatives, design, trade-offs) and critique them in review meetings. The interview's five-step arc is a compressed design doc: requirements (context + goals), estimation (scale), components (design), deep-dive (the interesting section), trade-offs (alternatives + consequences). Candidates who structure answers like a design doc are literally speaking the house style.
- **Failures are expected and engineered for.** The SRE book's thesis — "hope is not a strategy" — is Google's reliability posture: systems are designed for component failure, monitored with SLOs and error budgets, and improved through blameless postmortems. The interview's "what breaks, and what do we do" questions are this culture made conversational.
- **Disagree and commit.** Decisions are made by discussion and then executed by everyone — including those who disagreed. Interviewers reward candidates who can disagree *reasonably* (push back with evidence, then commit to the interviewer's direction when the evidence doesn't land) — the flexibility signal from §8.3.
- **Scale is a moral value.** Google builds — and believes in building — the largest systems in the world, and treats that as intrinsically valuable. A candidate who treats "billion users" as a checkbox rather than a design driver reads as culturally foreign. The "think at Google scale" framing (§10.3) is not optional garnish; it is the culture.

The practical consequence: the round rewards the *habits* of a good Google engineer — written-grade structure, spoken trade-offs, calm handling of failure — not the trivia of knowing Google's internals. You are being auditioned for the culture as much as for the architecture.

### 1.6 The Market Context (Flagged)

Two market-level facts shape how you should read this guide — both flagged as anecdotal, because the hiring market moves and neither Google nor the industry publishes authoritative numbers:

- **The bar is level-and-market-dependent.** Google's hiring bar has shifted over time with headcount and market conditions (hiring freezes, AI-era reallocations, return-to-office cycles); the L5 bar of 2026 is not identical to the L5 bar of 2019. What has been stable across cycles is the *structure* — the loop, the rubric, the HC — which is why this guide optimizes for structure first.
- **AI tools are changing the loop's edges.** Coding rounds now routinely assume candidates may use AI assistance in production, and some loops have adjusted (lighter trivia, more system-level reasoning); design rounds — where the artifact is a conversation, not code — have been comparatively stable. *(Flag: specifics of AI-tool policy per round are internal and change fast; the design-round stability claim is the consensus of 2024–2026 coaching commentary, not a Google statement.)*

The practical reading: prepare the durable structure (framework, math, stack, deep-dives) — it survives every market shift — and check current loop details with your recruiter when you apply, because the recruiter is the only authoritative source for *your* loop.

---

## 2. The Level Expectations

### 2.1 The Levels: L4, L5, L6

Google grades the same design round against **different expectations per level** — this is the single most important calibration fact in this guide. The same "design YouTube" answer that is a strong L4 pass is an L5 no-hire if it lacks trade-off depth, and an L6 no-hire if it lacks ambiguity handling and org-level thinking. *(Verified: level-calibrated expectations are consistently described in L4-vs-L5 comparison write-ups (e.g., finalroundai's L4/L5 breakdown) and in aggregated L6 feedback reports; the precise internal rubrics are unverified.)*

**L4 (SWE II) — "can design a well-scoped system correctly."**

- The expectations: clean architecture for a single component or a modest end-to-end system; correct data structures and API choices; awareness of scale (can estimate QPS, storage, latency) even if the numbers are rough; a working understanding of the standard building blocks (load balancers, caches, databases, queues). The L4 design round may be lighter or folded into a coding round, but where it is standalone the bar is *competence at a subsystem level*.
- What interviewers look for: you ask good clarifying questions; you produce a coherent architecture with a diagram the interviewer can follow; you identify the one component worth deepening; you know the standard trade-offs (cache vs. consistency, SQL vs. NoSQL) and can state them in one sentence each.
- The failure mode: designing without estimation, or building "everything" at surface level — a breadth-only answer with no depth anywhere.

**L5 (Senior) — "can design the whole product at scale, with justified trade-offs."**

- The expectations: end-to-end product design (the full requirements → estimation → components → deep-dive → trade-offs arc); confident, correct scale math; explicit trade-off analysis with a *decision and a reason* ("I choose X over Y because of Z, at this scale"); reliability thinking (replication, failure modes, degradation); clear leadership of the conversation — you drive, the interviewer steers.
- What interviewers look for: unprompted trade-offs; the ability to be *pushed* — when the interviewer adds 100× load or a new constraint, you react structurally, not defensively; a deep-dive that goes several layers down (e.g., for a cache: eviction policy, sharding, hot-key handling, cold-start behavior); a design that could plausibly be built by a team, not a toy.
- The failure mode: a correct-but-generic answer — the "textbook Twitter clone" delivered at any scale. L5 requires *calibrated* design: numbers that match the problem, choices that follow from those numbers.

**L6 (Staff) — "can design under ambiguity, with cross-team and operational consequences."**

- The expectations: everything at L5 *plus*: you impose structure on an intentionally vague prompt (the interviewer may give a business goal rather than a system: "we want to let users share large files"); you consider the org (interfaces other teams consume, migration paths, phasing — build v1, then v2); you handle non-functional breadth (cost, compliance, multi-region, long-term capacity); you weigh *alternatives* and can defend the chosen one against a skeptical interviewer.
- What interviewers look for: decomposition of ambiguity into decisions; a phased plan ("first a single-region MVP with these limits, then sharding, then multi-region"); cost and capacity awareness; the ability to change your own mind gracefully when presented with better information. L6 loops run 2–3 design rounds, so breadth across question types matters more than one perfect answer.
- The failure mode: designing the *solution* before defining the *problem* — L6 candidates who jump to a Spanner-and-Kafka architecture for a prompt that only needed a cron job fail the ambiguity test.

### 2.2 The Level Table

| Level | Expectations | Focus |
|---|---|---|
| **L4 (SWE II)** | Correct design of a well-scoped component or modest system; basic scale literacy (QPS, storage, latency estimates); standard building blocks; clean API and data model | Subsystem correctness; estimation basics; asking the right clarifying questions |
| **L5 (Senior)** | End-to-end product design at scale; justified trade-offs with decisions and reasons; reliability and failure modes; drives the conversation; survives interviewer push-back | Product-scale architecture; trade-off depth; scale math fluency; communication leadership |
| **L6 (Staff)** | Design under ambiguity; decomposition of vague prompts; phased plans and migration paths; cross-team interfaces; cost, compliance, multi-region breadth; defends choices under skepticism | Ambiguity handling; org-level consequences; breadth across 2–3 design rounds; operational and capacity thinking |

*Calibration note for the reader:* the descriptions above are the **publicly reported** shape of the expectations, synthesized from interview-account write-ups. Google's internal rubrics are not public, and actual expectations vary by team and by the interviewer's own level. Treat this table as a target to aim at, not a contract.

### 2.3 The Same Answer, Three Levels

The cleanest way to internalize the level table is to hear the *same design moment* delivered at three levels. The moment: the interviewer asks "how do you serve a query that matches a billion documents?"

- **L4 answer:** "We'd shard the index, and each shard has a posting list for the term; we scan the list and merge results. If it's too long we could... maybe cache popular queries." — Correct mechanics, hesitant depth, no ownership of the hard part. The candidate knows *that* sharding helps but not *why* the scan is the problem.
- **L5 answer:** "The posting list for a common term is huge, so we never scan it fully: each shard stores a block index over the postings, and the ranking model only needs the top-K candidates, so we skip-list to the best blocks, score those, and early-terminate. Hot terms get a replicated head index in memory. The trade-off is precision — we might miss a relevant-but-unusual document — which we accept because the model is trained on this serving behavior." — The decision, the mechanism, the trade-off, and the consequence, delivered without prompting.
- **L6 answer:** "Let me reframe — this is a serving-latency problem with a ranking-quality boundary. The block index and early termination keep p99 bounded, but the *policy* question is which documents we are willing to never surface; that's a product decision the ranking team owns. I'd phase it: v1 replicates head terms and accepts the recall loss, v2 moves to a two-tier index (head + tail) with separate capacity plans, and I'd put an SLO on recall coverage so the trade-off is measurable and owned." — The reframe, the org interface, the phasing, the SLO: L6 thinking.

The exercise: take any design moment from §7 (shard failure, cache stampede, conflict resolution) and write the three versions yourself. That is the fastest calibration drill in this guide.

### 2.4 The Rubric Signals in Practice

Google's four rubric signals (verified names, §1.1) map onto design-round behavior more concretely than candidates expect:

- **General cognitive ability** — measured by the *unseen*: a novel prompt, a new constraint, a scale injection. The interviewer watches how you reason from first principles when the library doesn't have a ready pattern. This is why the framework matters: it guarantees first-principles reasoning happens *aloud*.
- **Role-related knowledge** — measured by the *seen*: the classic patterns (inverted index, CDN, quorum, conflict resolution) and the Google-stack vocabulary (§6). The deep-dives in §7 are this signal's curriculum.
- **Leadership** — measured by *conversation control*: do you drive the arc, take ownership of decisions, recover from push-back without ego, and land the summary on time? The 45-minute budget (§8.2) is a leadership test as much as a time-management one.
- **Googleyness** — measured by *how you handle disagreement and ambiguity*: reasonable push-back, honest admission of uncertainty, calm under pressure, collaborative instincts ("good call — let's fix it" beats "no, my design is right").

One sentence per signal, in the interview: that is the meta-commentary the best candidates quietly provide ("this is where I'd trade recall for latency — that's the role-related trade-off; the leadership call is to timebox it"). You do not need to narrate the rubric — you need to *hit* it.

---

## 3. The Classic Google Design Questions

### 3.1 The Questions

The classic Google design question set is remarkably stable — interview-report aggregators (igotanoffer, LeetCode discuss, designgurus) and coaching sites converge on the same core list, dominated by **Google's own products**. *(Verified: "design Google Search/YouTube/Maps/Drive/Gmail" appear consistently across aggregated 2024–2026 question lists; the exact frequency per question is anecdotal and varies by report.)* The logic is straightforward: the interviewer knows the reference system deeply, so they can judge depth, and the scale of the real product is the calibration point.

**Search.** "Design Google Search" — the canonical Google question. Focus: inverted index, index sharding and replication, query processing, ranking, snippets, freshness, cache. This guide's worked example (§10) is this question end-to-end.

**YouTube.** "Design YouTube" — the classic video platform. Focus: upload path (chunked upload, transcoding pipeline, job queues), playback path (CDN, adaptive bitrate streaming), metadata store, view counters, recommendations (the ML cross-ref), and the asymmetry of read-heavy vs. write-heavy paths.

**Maps.** "Design Google Maps" — location at scale. Focus: map data as a tiled spatial dataset, tile serving and caching, routing (graph algorithms over road networks at global scale), geocoding, live traffic (streaming position data → road-speed estimation), and offline/downloadable regions. Often adapted to "design Uber/Lyft" — same spatial backbone plus matching.

**Drive.** "Design Google Drive" — cloud storage + sync. Focus: chunked/delta sync, conflict resolution, metadata database, object storage backend (cross-ref `s3_architecture_guide.md`), sharing and permissions, real-time collaboration (cross-ref the OT/CRDT discussion in this series), offline support, deduplication.

**Gmail.** "Design Gmail" — email at billion-user scale. Focus: mailbox data model (mail is write-once-read-many), inbox/threads indexing, search over mail (cross-ref the Search deep-dive), spam filtering (ML cross-ref), delivery and retry (cross-ref `message_queue_data_loss_guide.md` for queue reliability), and the "one conversation, many messages" thread model.

**Spanner.** "Design a globally distributed database like Spanner" (also phrased "design a distributed database with strong consistency across regions"). Focus: global replication, external consistency, TrueTime/clock discipline, Paxos-based consensus groups, sharding/tablets, two-phase commit across groups, and the read/write latency trade-off of synchronous replication. Cross-ref `apache_seata_guide.md` for the distributed-transaction component vocabulary.

**Honorable mentions** (verified as recurring on Google lists): design a **distributed cache**, design **TinyURL/URL shortener**, design **Google AdWords/ad auction** (a Google favorite — ranking + real-time bidding + budget pacing), design **WhatsApp/chat**, design a **news feed**, design a **web crawler**, design a **distributed rate limiter** (cross-ref `distributed_rate_limiter_guide.md` — a complete worked deep-dive already in this series), design a **key-value store**, and the **"design something from my own domain"** variant (payments, settlement, trading systems — see the `banking/` guides).

### 3.2 The Question Table

| Question | Focus | Components |
|---|---|---|
| **Search** | Inverted index; query → retrieval → ranking → snippets; freshness; cache | Crawler (background), indexer, index shards, query servers, ranking service, snippet generator, query cache |
| **YouTube** | Asymmetric read/write; media pipelines; CDN; job orchestration | Upload service, chunk storage, transcoding job queue + workers, metadata DB, CDN/edge cache, view-counter service, recommendation service |
| **Maps** | Spatial data at global scale; tile serving; graph routing | Tile generation pipeline, tile/CDN cache, routing service (graph DB + precomputed shortcuts), geocoder, live-traffic ingestion (streaming) |
| **Drive** | Sync semantics; conflict resolution; object storage; sharing | Client sync engine, chunk/delta service, metadata DB, object store, ACL/permission service, notification/websocket service, collaboration (OT/CRDT) |
| **Gmail** | Mailbox model; threads; mail search; delivery reliability; spam | SMTP gateway, delivery queue (with retry/backoff), mailbox + thread store, mail index, spam classifier, push/notification service |
| **Spanner** | Global consistency; consensus; clock discipline; sharded transactions | Tablet servers, Paxos groups, TrueTime, two-phase commit coordinator, directory-based sharding, F1-style SQL layer |
| **Your domain** | Adapts the framework to what you know deeply — the interviewer's way of testing *judgment*, not recall | Whatever the domain demands; the framework and trade-off discipline are identical |

### 3.3 How to Choose and Prep the Question Set

You cannot deeply prep twelve questions in four weeks; you can deeply prep four and pattern-match the rest. The selection heuristic:

1. **Prep the product set that matches your target.** For a generalist SWE loop: Search, YouTube, Drive, and a distributed-database question (Spanner-style) — these four cover every pattern (index/query, async media pipeline, sync/storage semantics, consistency). For SRE: replace one with a reliability-weighted design (a storage system with SLOs, or a monitoring/alerting system — a real SRE loop question).
2. **Prep two your-domain cases.** For a banking technologist (the Cymbal Bank context): "design a real-time payments pipeline" and "design an API gateway with rate limiting and quotas" — the latter is literally `distributed_rate_limiter_guide.md` extended to gateway scale, and the `banking/` guides supply the domain vocabulary. Google interviewers adapt prompts to your background (§1.2), and your-domain depth is the one place you can out-depth the interviewer.
3. **For the rest, prep the skeleton.** One page per question: clarifying questions, estimation numbers, component list, the one deep-dive you'd choose, three trade-offs. That is enough to survive any of them; the four deep preps are what you'll actually shine on.
4. **Cross-train with the ML guide.** `ml_system_design_interview_guide.md` §9 covers the ML-flavored variants of these same questions (search ranking, recommendations, feed) — if your target role has ML surface area (and increasingly Search/YouTube questions invite an ML ranking answer), read that section as the ranking deep-dive's companion.

### 3.4 The Question Bank (Bonus)

A quick-reference list of prompts reported in Google design rounds (aggregated from interview-account sites — frequency claims are anecdotal, the *existence* of each prompt type is corroborated across multiple reports):

*Search and content:* design Google Search · design YouTube · design a news feed · design a web crawler · design Google Images/visual search · design a recommendation system (cross-ref the ML guide)
*Location and transport:* design Google Maps · design Uber/Lyft · design a delivery-routing system
*Storage and sync:* design Google Drive · design Dropbox-style sync · design S3 (cross-ref `s3_architecture_guide.md`) · design a key-value store · design a distributed file system · design a blob/object store
*Communication:* design Gmail · design WhatsApp/chat · design a video-conferencing service · design a notification system · design a pub/sub system (cross-ref the Kafka guides)
*Data and consistency:* design Spanner / a globally consistent DB · design a distributed cache · design a distributed rate limiter (cross-ref `distributed_rate_limiter_guide.md`) · design a distributed queue (cross-ref `message_queue_data_loss_guide.md`) · design a transaction/ledger system (cross-ref `apache_seata_guide.md` and the `banking/` guides)
*Ads and metrics:* design Google AdWords/ad auction · design an analytics/telemetry pipeline (cross-ref `event_stream_processing_guide.md`) · design a counting/views system
*Your domain:* design a payments pipeline · design an API gateway · design a settlement-reconciliation system — the variant where your banking depth is the point

Rule of thumb: if you have built the one-pagers for Search, YouTube, Drive, Spanner, the rate limiter, and your two domain cases, every prompt above is a variation you can pattern-match in under a minute.

---

## 4. The Design Framework

### 4.1 The Adapted Framework

Google interviewers do not publish a canonical framework, but the **five-step arc** below is the consensus shape of every strong Google design answer, and it is the same family as the ML-design framework in `ml_system_design_interview_guide.md` (§3: clarify → data → features → model → training → serving → evaluation → monitoring) and the standard system-design canon (Alex Xu, ByteByteGo, grokking). The Google flavor is in *how* each step is weighted: scale math is non-optional, reliability is expected, and trade-offs are spoken aloud.

**Step 1 — Requirements (5 minutes).** Clarify before designing. Ask functional questions (what does it do, who uses it, what are the primary actions) and non-functional questions (scale, latency, availability, consistency, cost). Then *restate*: "So we're designing X for Y users with Z latency, and the critical path is..." The restatement is your contract with the interviewer — it anchors everything that follows. Google-flavored: get explicit about **consistency vs. availability** and **read vs. write ratios** early; these two dials change every later decision.

**Step 2 — Estimation (5 minutes).** Rough but *right-order-of-magnitude* numbers: QPS (average and peak), storage (with growth and replication), bandwidth, and cache/DB sizing implications. You do not need a calculator — you need round numbers and clean reasoning (all formulas in §5). Google-flavored: state peak = 5–10× average *before* the interviewer asks, and say what breaks first at 10× the estimate.

**Step 3 — Components (10 minutes).** The architecture: clients → load balancer → application services → data layer, with the cache, queue, and search/index layers where they belong. Draw it (whiteboard or shared doc), label every box, and name one sentence of responsibility per box. Google-flavored: name the **building block** (Bigtable-like store, GFS-like object store, Spanner-like transactional store — §6) rather than drawing an anonymous "database", and cross-ref the component deep-dives in this series (rate limiter at the edge, queue for async work, S3-like object store for blobs).

**Step 4 — Deep-dive (15–20 minutes).** The interviewer will pick one component — or you should propose one ("the interesting part here is the index; let me go deep on that"). This is where levels separate (§2): L4 goes one level down, L5 goes several, L6 goes until the interviewer stops you *and* handles the org implications. Google-flavored: expect the interviewer to *choose* the deep-dive — be equally ready to go deep on the data layer, the async pipeline, or the cache. If they pick something you under-designed, recover by saying "good call — that's the harder problem, let's redo it."

**Step 5 — Trade-offs and wrap-up (5 minutes).** Name 2–3 concrete trade-offs you made, the alternatives you rejected, and the conditions under which you'd flip the decision. Then summarize the design in 3 sentences. Google-flavored: trade-offs are *scored*, not optional — "we could use X, but at this scale Y is better because..." is the L5/L6 signal. End with failure modes: "if the metadata DB dies, we serve reads from cache and queue writes" — reliability awareness closes the loop.

### 4.2 The Framework Table

| Step | Activities | Google-flavored notes |
|---|---|---|
| **Requirements** | Clarify functional + non-functional; restate the contract | Pin consistency vs. availability and read:write ratio early; ask "is this a new system or a redesign?" (redesigns are common at Google — "the index is 10× too big, what do we do?") |
| **Estimation** | QPS (avg + peak), storage (+growth, +replication), bandwidth, cache/DB sizing | State peak = 5–10× average unprompted; say what breaks first at 10× — interviewers love that sentence |
| **Components** | Client → LB → services → data layer; label every box; name the building blocks | Use the Google-stack vocabulary (§6) as *building blocks*, not as namedropping; draw the async paths (queues) explicitly |
| **Deep-dive** | Go 2+ levels into the component the interviewer picks (or you propose) | Be ready for them to pick the hardest box; recover from under-design by re-scoping aloud, not by bluffing |
| **Trade-offs** | 2–3 concrete trade-offs + rejected alternatives + flip conditions; 3-sentence summary; failure modes | Trade-offs are scored; end with "what happens when X fails" — the SRE-flavored closer |

*Cross-ref:* the same arc, adapted for ML, is Section 3 of `ml_system_design_interview_guide.md` — the two guides share the clarify-first, estimate-second, communicate-continuously discipline. The rate limiter in `distributed_rate_limiter_guide.md` is a complete worked example of the *component* layer of this framework (the "what breaks first" and "trade-offs" sections are exemplary).

### 4.3 The Framework Drills

The framework is only useful if it is automatic — under pressure, you cannot *remember* a five-step process; you must *run* it. Three drills make it reflex:

1. **The 60-second run.** Pick any product (Instagram, a parking app, a bank's payment API). In 60 seconds, out loud: one-line restatement → two scale numbers → three components → the deep-dive you'd choose → one trade-off. This is the warm-up drill; do it daily until it is boring.
2. **The push-back drill.** Take a completed one-pager from your case library; have a partner (or a timer plus a list of canned pushes: "10× the users", "the metadata DB died", "now it must be multi-region", "budget is halved") inject constraints and force you to *revise aloud* — the recovery behavior (§8.3) is a skill, and it only trains under fire.
3. **The 45-minute rehearsal.** The full mock (§9.2) — framework + math + communication + time budget, end to end. Do at least three before the real day; the first one is always the worst, and that is the point.

The standard against which to judge yourself: you should be able to produce the five-step arc for a *product you have never heard of* within the first 5 minutes of hearing the prompt. If you can, the framework is doing its job, and the content (stack, deep-dives, math) has room to breathe.

---

## 5. The Scale Math

### 5.1 The Estimation: QPS, Storage, Bandwidth

Every Google design answer earns or loses points in the estimation step. The math is deliberately simple — order-of-magnitude arithmetic with round numbers — and the skill is *fluency*, not precision. Three metrics cover 90% of cases.

**QPS (queries per second).** The formula: `QPS = (DAU × actions per user per day) / 86,400 seconds`. Then peak = 5–10× average. Sanity anchors: 86,400 ≈ 10^5 seconds/day; 1 billion daily actions ≈ 11.6K QPS average; 1 million DAU doing 10 actions/day ≈ 116 QPS average. At Google scale: if the product has ~1 billion users and each does ~10 actions a day, average QPS ≈ 10^10 / 10^5 = ~100K, peak ~500K–1M.

**Storage.** The formula: `storage = records × bytes per record × retention × replication`. Growth matters — the interviewer wants to hear "and at 3× per year, in 3 years this is 27×", not just the day-one number. Anchors: 1 GB = 10^9 bytes; 1 TB = 10^12; 1 PB = 10^15. A metadata row of ~1 KB × 1 billion users = 1 TB (×3 replication = 3 TB, ×4x growth = 12 TB) — comfortably one cluster, which is *itself* a design decision (single metadata cluster vs. sharded).

**Bandwidth.** The formula: `bandwidth = QPS × bytes per request` (plus the async/ingest side: producers × production rate × payload). Anchors: 100K QPS × 10 KB = 1 GB/s ingress. Media flips everything: 1 hour of 720p video ≈ 1–2 GB; 500 hours uploaded per minute (YouTube's verified public figure) ≈ 500–1,000 GB per minute ≈ 8–17 GB/s of upload ingest *before* transcoding — which is why the upload path is designed around chunked, resumable, async processing (§7.2).

### 5.2 The Math Examples (Worked)

**Example 1 — Google Search.** *(Flag: 8.5 billion searches/day is the widely repeated third-party figure; Google does not publish search-volume numbers.)* Average QPS = 8.5 × 10^9 / 86,400 ≈ **~98K QPS**; peak (say 3× diurnal + event spikes) ≈ 300K–1M QPS. Each query fans out to the index: ~10–100 index shards touched in parallel, so the *back-end* QPS is ~10–100× the front-door number (millions of shard-query operations/sec at peak). Storage: index of **hundreds of billions of pages** (Google's own "How Search works" describes the index at this scale) × ~10 KB inverted-index entry ≈ 1–10 PB before replication — a fleet, not a database. Latency budget: ~200–400 ms end-to-end (widely reported user-experience target; not an official SLA — flag), which is why caching, precomputed popular results, and parallel fan-out exist.

**Example 2 — YouTube upload.** 500 hours/min × ~1.5 GB/hour source ≈ **~750 GB/min ≈ 12.5 GB/s** sustained ingest, × 3 replication ≈ 37 GB/s of storage write bandwidth. Transcoding multiplies this: each video generates multiple renditions (240p–4K, ~5 renditions average), so ~2,500 hours/min of *output* media to store — ~3.7 TB/min of new media → **~5 PB/day**. The design implication: you cannot do this synchronously; the upload path is an async pipeline (chunked upload → object store → job queue → transcode workers → CDN push).

**Example 3 — Drive sync.** 1 billion users × 10 files × 1 MB average = **10 PB** of user content (+ replication ×3 = 30 PB). Metadata: 10 billion file records × ~1 KB = ~10 TB — a single (sharded) metadata cluster. Sync traffic: 1 billion users × 1 sync check per hour ≈ ~280K QPS of lightweight "anything changed?" polling — which is why notification-based sync (websocket/push) replaces polling at scale.

### 5.3 The Scale Table

| Metric | Formula | Example |
|---|---|---|
| **QPS (avg)** | (DAU × actions/day) / 86,400 | 1B users × 10 actions → ~116K QPS |
| **QPS (peak)** | avg × 5–10× | Search: ~98K avg → ~300K–1M peak |
| **Storage** | records × bytes × retention × replication | Drive: 10B files × 1 KB metadata ×3 = ~30 TB metadata; 10 PB content ×3 = 30 PB |
| **Bandwidth (ingest)** | producers × rate × payload | YouTube upload: ~12.5 GB/s source, ~37 GB/s with replication |
| **Bandwidth (serving)** | QPS × bytes/response | 100K QPS × 10 KB = 1 GB/s; video streaming: 1M concurrent × 4 Mbps ≈ 500 GB/s (CDN-absorbed) |
| **Cache sizing** | QPS × latency-to-backend × cache TTL | 100K QPS × 50 ms × 60 s ≈ 300M entries ≈ 30 GB at 100 B/key |

*Math fluency tip:* practice these three formulas until they are reflex — every mock interview in §9 must include an estimation step, because fluency under the clock is what the interview day (§8) rewards.

### 5.4 The Estimation Traps

The estimation phase fails in predictable ways — memorize the traps so you don't fall into them:

- **Off-by-86,400.** Dividing by 86,400 is the one formula everyone gets wrong under pressure. Anchor: 1 billion actions/day ≈ 11.6K QPS — if your answer is off by 100×, you skipped the division or misread the DAU.
- **Forgetting peaks and growth.** Average QPS without peak, or storage without a growth multiplier, is a half-answer. The sentence "average X, peak 5–10×, storage with 3× replication and 3× growth" is worth more than any single precise number.
- **Mixing bytes and bits.** Bandwidth is quoted in bits (Mbps) by network people and bytes (MB/s) by storage people; 1 GB/s = 8 Gbps. Pick one convention, state it, stay consistent.
- **The fan-out blind spot.** Front-door QPS is not back-end QPS. Search fans out to ~50–100 shards; a cache miss fans out to storage; a video view fans out to segments. The interesting scale is *behind* the door — saying so unprompted is a senior signal.
- **Not connecting math to design.** An estimate that doesn't change a decision is wasted breath. Every number should end in a consequence: "100K QPS → we need a cache layer and a sharded index; 10 PB → a single metadata cluster won't do, we shard by user range."

### 5.5 The Latency and the 9s Math

Two more numeric dialects that Google interviewers speak fluently — and that candidates often neglect:

**The 9s.** Availability is downtime math: 99% = ~3.65 days/year down; 99.9% = ~8.76 hours/year; 99.99% = ~52.6 minutes/year; 99.999% = ~5.26 minutes/year. The interview uses these as *design dials*: "design for 99.9%" means you need replication and graceful degradation; "99.99%" means you need multi-region and a rehearsed failover; "99.999%" means you are designing a carrier-grade system and the cost conversation is mandatory. The reliability math behind them: with 3× replication and independent failures, the chance all replicas fail together is p³ — three 9s from components that individually have one 9. The sentence "we replicate 3×, which turns component-level one-9 failures into system-level three-9 availability" is a senior-flavored sentence. (Cross-ref the SRE material in this series and the error-budget framing in §1.2 for the operational side.)

**Latency percentiles.** Average latency lies; p99 is the user-visible story (the slowest 1% of requests are the ones users complain about — and at 100K QPS, p99 means ~1,000 requests/second are slow). The design consequence: the p50/p99 gap is a *system property* (tail latency — caused by stragglers, GC pauses, hot shards) and it is fixable by design: timeouts and retries with jitter, hedging requests (fire two, take the first), work-stealing, and per-shard load balancing. When the interviewer asks "what latency do you promise?", the answer format is "p50 under X, p99 under Y, and here's the component that determines the tail." *(Flag: specific Google SLO numbers are internal; the percentiles-and-tail-latency discipline is public SRE practice.)*

---

## 6. The Google Stack

### 6.1 The Stack: GFS, Bigtable, Spanner

The Google stack is the vocabulary of Google-flavored design answers. You are **not** expected to know proprietary internals — you *are* expected to know the published papers (they are public, canonical distributed-systems literature) and to use the systems as **building blocks** in your designs. *(Verified: GFS, Bigtable, and Spanner are all published papers — Ghemawat et al. 2003, Chang et al. 2006, Corbett et al. 2012 — and their design roles are well documented.)*

**GFS — the Google File System (2003 paper).** A cluster-scale distributed file system for *huge files, append-heavy workloads*. Architecture: one **master** (metadata: file names, chunk locations) + many **chunkservers** (data); files split into **64 MB chunks**, each replicated **3×** across machines (and racks); writes are append-mostly, reads are streaming. Design philosophy that matters for interviews: **failures are the norm** (chunkservers die constantly; the system is built for that), **huge files, not many small ones**, and **relaxed consistency** in exchange for simplicity and throughput. The successor (Colossus) keeps the ideas with a distributed metadata layer — worth one sentence if you want depth.

**Bigtable (2006 paper).** A **wide-column, sparse, sorted map** — the original NoSQL: `(row key, column key, timestamp) → value`, rows sorted lexicographically, sharded into **tablets** (sorted ranges of rows), each tablet served by a tablet server, built **on top of GFS** (SSTables as GFS files), with **Chubby** (the lock service) for coordination. Used for Search's web index, Google Analytics, Maps, and Gmail internally. For interviews, Bigtable is the model for: *high-throughput, low-latency, key-ordered access to huge datasets where strong cross-row transactions are not required*. The modern public equivalent is Bigtable-on-GCP (Cloud Bigtable) and the design pattern is shared with HBase/Cassandra.

**Spanner (2012 paper).** A **globally distributed, externally consistent** database — SQL semantics, synchronous replication across regions, **TrueTime** (GPS + atomic-clock synchronized time with bounded uncertainty) to order transactions, **Paxos** groups per shard (tablets, organized in **directories**), and **two-phase commit across groups** for multi-shard transactions. F1 (Google's AdWords DB) runs on Spanner. For interviews, Spanner is the model for: *strong global consistency with transactions at planetary scale, at the price of write latency* (every write pays a synchronous round-trip to a quorum, often cross-region). The public equivalent is Cloud Spanner; the design trade-off it teaches (consistency ↔ write latency ↔ availability) is interview gold.

### 6.2 The Stack Use: How to Deploy It in Design Answers

The right way to use the stack in an interview is as **a design language, not a name-drop**:

- "For the metadata, I'd use a **Bigtable-like** store: key-ordered, horizontally sharded, high write throughput — we don't need cross-row transactions here" — *then* name the trade-off you're accepting (no multi-row atomicity; eventual consistency for derived data).
- "For user content, an object store like **GFS/Colossus** (or S3 — cross-ref `s3_architecture_guide.md` for the object-store design anatomy): immutable blobs, replication by default, no per-blob transactions needed."
- "For payments-grade consistency — cross-region, transactional — a **Spanner-like** system: external consistency, but every write pays synchronous replication latency, so I'd only use it where the business needs it" — cross-ref `apache_seata_guide.md` for the distributed-transaction patterns this replaces (2PC, saga, TCC) and when a saga is the cheaper choice.
- The **honest caveat**: when you name a Google system, add one sentence on *why* it fits — interviewers probe name-drops mercilessly. If you don't know the internals, say so and pivot to the *design pattern* ("the idea of sorted, sharded tablets") which is what is actually being scored.

### 6.3 The Stack Table

| System | Role | Design use in answers |
|---|---|---|
| **GFS / Colossus** | Cluster-scale file system: 64 MB chunks, 3× replication, master + chunkservers, append-heavy | The blob/object layer: media, index files, uploads, logs. Use when you need cheap, replicated, huge-file storage with no per-file transactions |
| **Bigtable** | Wide-column sorted map on GFS; tablets sharded by key range; SSTables; Chubby for coordination | The high-throughput key-value/wide-column layer: metadata, counters, per-user state, the web index. Use when you need scale + low latency and can live without multi-row transactions |
| **Spanner** | Globally distributed SQL: TrueTime, Paxos groups, 2PC across groups, external consistency | The strongly-consistent transactional layer: payments, balances, anything with cross-region consistency requirements. Name the cost: synchronous write latency |
| **MapReduce / Dataflow** | Batch/stream processing over GFS/Bigtable data (the paper family) | The analytics/derived-data layer: offline index building, view-count aggregation, recommendation training — cross-ref `event_stream_processing_guide.md` for the streaming-era equivalent |
| **Chubby / Paxos** | Lock service / consensus primitives | Coordination: leader election, metadata agreement — name the pattern, not the product ("a consensus-based lock service") |

### 6.4 The Papers' Interview Lessons

Each Google paper is also a *design lesson* you can deploy in any interview answer, even when the system itself is irrelevant:

- **GFS (2003): failures are the norm, and simplicity beats cleverness.** The paper's most-quoted observation — component failures are the norm, not the exception — is the interview's reliability creed: design for failure, use replication, and accept relaxed consistency where the product tolerates it. The 64 MB chunk is a lesson in *bigger granularity = fewer metadata operations*: when you design a storage layer, chunk size is a deliberate lever, not an accident.
- **Bigtable (2006): the wide-column sorted map is the workhorse.** Sorted keys enable range scans and locality (the "index shard" pattern); tablets make sharding self-managing; sparse columns make schemas cheap to evolve. When an interviewer asks "how do you store per-user state at scale?", the Bigtable-shaped answer — key = user_id, sorted, sharded by range, no cross-row transactions — is the default move.
- **Spanner (2012): consistency is a product decision with a price tag.** Spanner exists because some data (AdWords money) needs cross-region atomicity; its cost is synchronous write latency and TrueTime complexity. The interview lesson is the *calibration*: use Spanner-like semantics only where the business requires it, and be able to say the price out loud (write latency scales with quorum distance; availability drops with synchronous replication). Cross-ref `apache_seata_guide.md` for the transaction-pattern alternatives (saga, TCC) when global strong consistency is overkill.
- **MapReduce (2004): batch processing over the cluster, expressed as map + reduce.** The paper that made "think in pipelines" mainstream: offline index building, log aggregation, derived data — all map+reduce-shaped jobs over GFS-like storage. Mentioning the pattern (not the product) shows you know how derived data gets built at scale.

---

## 7. The Question Deep-Dives

### 7.1 The Search Deep-Dive: The Index and the Ranking

*(Full worked walkthrough in §10 — this subsection is the component-level playbook.)*

**The index.** The heart of Search is the **inverted index**: for every term, the list of documents (postings) containing it, with positions and metadata. At the hundreds-of-billions-of-pages scale (Google's own description of its index), the inverted index is **sharded** two ways:

- **Document partitioning**: each shard holds the postings for a subset of documents, all terms — every query hits *all* shards (or a sampled subset), and results are merged. Simple, and matches "every query scans the world".
- **Term partitioning**: each shard owns a subset of terms — queries touch only the shards of their terms, but hot terms create hot shards.

The classic design answer: **document partitioning** for serving (parallel fan-out across shards, merge by rank), with **term-level hot-spot handling** (replicate hot term shards, or use a two-tier "popular terms" index). The index itself is built **offline** (crawler → indexer pipeline, MapReduce/Dataflow-style, over a GFS-like store) and **pushed** to serving shards — freshness is a pipeline-latency problem, not a serving problem.

**The ranking.** Modern ranking is a learned model over hundreds of features (Google's "How Search works" confirms ranking uses a large set of factors; the exact model is proprietary — flag). For the interview, the *story* matters more than the model: query understanding (spell-check, synonyms, query rewrite) → **retrieval** (candidate generation from the index — BM25-style scoring or embedding-based recall; cross-ref the ML guide's two-tower discussion) → **ranking** (a model scoring candidates with features: term signals, PageRank/authority, freshness, location, device, personalization) → **snippets** (the highlighted passage — a small extraction model over the top-K documents). PageRank — the original authority signal — is worth knowing as history and as a "graph-based authority" pattern: link graph → stationary distribution → per-document authority score, precomputed offline.

**Serving path:** query → spell-correct/rewrite → cache check (popular queries: LRU at the edge, cross-ref `distributed_rate_limiter_guide.md` for the cache/edge vocabulary) → parallel fan-out to index shards → merge by score → snippet extraction → results page. Latency budget ~300 ms forces aggressive parallelism and caching.

### 7.2 The YouTube Deep-Dive: The Upload and the Streaming

**The upload path (write-heavy, async by construction).** Uploads arrive as **chunks** (resumable: a dropped connection retries the last chunk, not the file — cross-ref `message_queue_data_loss_guide.md` for the reliability patterns). Chunks land in an **object store** (GFS/Colossus-like), then a **job queue** (the pattern from `distributed_rate_limiter_guide.md`'s queue discussion and `event_stream_processing_guide.md`) feeds the **transcoding pipeline**: a DAG of jobs per video — transcode to multiple renditions (240p→4K), thumbnail extraction, audio, subtitles, content fingerprinting/abuse checks. Each stage is a pool of workers; failure = retry with backoff; the queue is the durability point (a lost job is a lost video). Output renditions go back to the object store and are **pushed to CDN edges** for the popular-content preload. The key design decision: **never block the uploader** — the user gets "upload complete" when the chunks are durably stored, *not* when transcoding finishes.

**The streaming path (read-heavy, CDN by construction).** Playback uses **adaptive bitrate streaming** (DASH/HLS): the video is cut into ~2–10 s segments at multiple bitrates; the player fetches segments over HTTP and switches bitrate with network conditions. The **CDN** is the star: edge caches serve the long tail of views; popular content is proactively replicated to edges (a popularity-prediction pipeline — cross-ref the ML guide's recommendation discussion); origin storage is hit only on cache miss. Views counters are **sharded counters** (eventual consistency is fine — nobody needs an exact concurrent view count; the design answer is "partition by video, then by hash, aggregate asynchronously"). Recommendations are a separate ML system (cross-ref `ml_system_design_interview_guide.md` §9 for the two-tower design).

**Key decisions:** chunked resumable upload; async transcoding DAG with a durable queue; object store for blobs + Bigtable-like store for metadata; CDN + adaptive bitrate for playback; sharded eventual-consistency counters; and the asymmetry — *ingest is GB/s, serving is TB/s, and the CDN is what makes serving feasible*.

### 7.3 The Drive Deep-Dive: The Sync and the Storage

**The sync model.** The client is the product: a background agent watches a folder and syncs changes. **Chunking** is the foundation: files are split into fixed or content-defined chunks (content-defined chunking — CDC — finds chunk boundaries by content hash, so a small edit only re-uploads the changed chunks); the client sends a **manifest** of chunk hashes to the server; the server deduplicates (same chunk hash = same content = store once — cross-ref `s3_architecture_guide.md` for object-store dedup anatomy). **Delta sync**: only changed chunks travel. **Resumable uploads** per chunk (the reliability cross-refs again).

**Conflict resolution.** Two devices editing the same file offline → conflict. The honest design spectrum: **last-write-wins** (simple, loses edits — acceptable for many files), **versioning** (keep both versions; user resolves — the "conflict copy" pattern Drive actually uses), and **CRDTs/OT** (mergeable edits — required for real-time collaboration on the same document; cross-ref the collaboration discussion referenced in this series' Docs material). The interview answer: LWW by default, versioned conflict copies for humans, OT/CRDT only for collaborative-editing products — and *name the trade-off* (complexity vs. lost edits).

**The storage and metadata.** Content lives in an **object store** (chunked blobs, replication by default). Metadata — file tree, permissions, versions, share links — lives in a **transactional metadata store** (the file tree needs atomic renames/moves, so a Spanner-like store, or a sharded SQL layer, is the honest choice; Drive's real implementation details are proprietary — flag). **Sharing** is an ACL layer: file → ACL entries (user/group, read/write/comment), inherited from folders; share-link tokens are capability-style access (possession of the token grants access — a nice design detail interviewers like). **Notifications** (websocket/push) tell clients to re-sync instead of polling — at 1B users, polling is bandwidth suicide (§5.2 Example 3). **Offline support** = local cache of manifests + queued sync ops + conflict resolution on reconnect.

**Key decisions:** content-defined chunking + dedup; manifest-based sync; LWW + versioned conflicts (OT/CRDT for collab); object store for chunks, transactional store for metadata; ACLs + capability links for sharing; push over poll; offline queue-and-reconcile.

### 7.4 The Maps Deep-Dive: The Tiles and the Routing

**The tile model.** Maps is a *spatial* system built on a **tiled pyramid**: the world is cut into a quadtree of tiles — zoom level z has 4^z tiles (zoom 0 = 1 tile, zoom 15 ≈ 1 billion) — each tile pre-rendered (vector or raster) at its zoom level. Serving is then a *cache problem*, not a computation problem: tiles are static content, so tile serving is a CDN/edge cache with a tile-generation pipeline behind it (cross-ref the YouTube CDN discussion — same pattern, smaller objects). Popular areas (cities) are pre-generated and edge-replicated; rarely visited tiles are generated on demand and cached. The design lesson: **precompute what you can, compute on demand what you must, and let the cache absorb the long tail.**

**Routing.** The hard part. The road network is a **graph** — tens of millions of edges at global scale — and routing is shortest-path over it. The classic answer has three layers: (1) **precomputed shortcuts** (contraction hierarchies or transit-node routing: precompute shortcuts so long-distance queries traverse a sparse overlay graph — this is why routing is fast in practice); (2) **partitioned graph shards** (the graph is split by region; cross-region queries stitch precomputed boundary-to-boundary paths); (3) **live traffic as edge weights** — position pings stream in (cross-ref `event_stream_processing_guide.md`), are aggregated into per-road-segment speeds, and update edge weights; the router recomputes with the live weights, and alternative routes are scored against the same data. The trade-off: exactness vs. cost — full Dijkstra over the whole planet is the naive answer; every real system uses precomputation plus heuristics, and interviewers want to hear that progression.

**Geocoding and places.** Forward geocoding (address → coordinates) and reverse (coordinates → address) are ML-flavored matching problems over a places dataset (cross-ref the ML guide's retrieval discussion); for the interview, one sentence — "a search over the places index, same inverted-index machinery as Search §7.1" — is enough depth, and spending the saved time on routing is the better bet.

### 7.5 The Gmail Deep-Dive: The Mailbox and the Delivery

**The mailbox model.** Email is *write-once, read-many, per-recipient*: one message fans out to N mailboxes. The classic design decision is **per-mailbox message references**: store the message blob once (dedup by content hash — cross-ref the Drive dedup discussion), and give each recipient a mailbox *entry* referencing it, with per-mailbox state (read/unread, labels, archive). Threads ("one conversation") are a grouping layer: messages grouped by subject/references header, rendered as a thread, with per-mailbox thread state. Storage: hot mail in a Bigtable-like store keyed by (user_id, message_id) — sorted, sharded by user — with cold mail tiered to object storage (cross-ref `s3_architecture_guide.md` for the tiering anatomy); ~1 KB per message × hundreds of billions of messages ≈ hundreds of TB before replication — a fleet, sharded by user range.

**Delivery and reliability.** Inbound mail arrives via SMTP gateways and enters a **durable queue** — the delivery system must *never* lose a message, so the queue is the durability point (cross-ref `message_queue_data_loss_guide.md` for exactly this reliability analysis: at-least-once delivery, acks, retries with backoff, dead-letter queues, and the exactly-once illusion). Delivery workers resolve recipients, write mailbox entries, and trigger push notifications. **Search over mail** is a per-user index — small (thousands of messages per user) and trivial next to web search: the design answer is "a per-user inverted index, updated on mailbox writes, served from the same shard as the mailbox" — the *locality* insight (index next to data) is the senior detail. **Spam** is an ML classifier in the ingest path (cross-ref `ml_system_design_interview_guide.md` §9 for the classifier design); the design point is placement — classify *before* the mailbox write, with a quarantine store and a user feedback loop. The trade-off to name: delivery latency vs. durability — "acknowledge the SMTP sender only after the message is durably queued" is the decision that makes mail loss-free at the cost of a sync write.

### 7.6 The Spanner Deep-Dive: The Global Database

*(Prompt variants: "design a globally distributed database with strong consistency", "design Spanner", "design a multi-region database for a global payments system".)*

**The core tension.** You cannot simultaneously have low-latency writes, global strong consistency, and high availability (the CAP/Brewer framing, stated as a trade-off — cross-ref `apache_seata_guide.md` for the transaction-pattern family). Spanner's choice: **strong consistency and availability, at the price of write latency** — every write synchronously replicates to a quorum across regions before acknowledging. The interview answer should *open* with this tension, because it frames everything.

**The components.** (1) **Sharding**: data is partitioned into **tablets** (sorted key ranges), each tablet replicated by a **Paxos group** (one leader, N followers across regions); a **directory** is a set of co-located keys that move between groups atomically — the placement unit (hot directories can be split; related data stays together). (2) **TrueTime**: transactions are ordered using time intervals with bounded uncertainty (GPS + atomic clocks, ±~7 ms at publication time); commit timestamps respect the uncertainty so that external consistency holds — reads at a timestamp see all writes committed before it. This is the famous "time is the answer to distributed ordering" lesson. (3) **Cross-group transactions**: multi-shard transactions use **two-phase commit** across Paxos groups — the coordinator is itself a replicated group; the cost is the extra round-trips and the latency of the slowest group. (4) **The SQL layer** (F1-style): the API is relational — the whole point is that applications get transactions and SQL, not a key-value store.

**The trade-offs to name.** Write latency grows with quorum distance (single-region quorum ≈ tens of ms; cross-region ≈ 100+ ms — synchronous, unavoidable); availability is bounded by the quorum, not by any single region; read latency can be optimized with **single-region reads at a timestamp** (read-only transactions skip the sync path) — "read from the local replica at the latest safe timestamp" is the canonical optimization. When the interviewer asks "isn't this overkill for most systems?" — the right answer is yes: the payments/ledger case (cross-ref the `banking/` guides and `apache_seata_guide.md`) is where the price is worth paying; for a social feed, eventual consistency is the correct choice, and the interview scores you for *saying* that.

### 7.7 The Deep-Dive Table

| Question | Key design decisions | Components |
|---|---|---|
| **Search** | Inverted index; document-partitioned shards + hot-term replication; offline index build, push to serving; parallel fan-out + merge; caching popular queries | Crawler, indexer, index shards, query servers, ranker, snippet extractor, query cache |
| **YouTube** | Chunked resumable upload; async transcoding DAG over a durable queue; object store + Bigtable-like metadata; CDN + adaptive bitrate; sharded view counters | Upload service, object store, job queue + transcoders, metadata DB, CDN, counter shards, recommender |
| **Drive** | Content-defined chunking + dedup; manifest sync; LWW + versioned conflicts; object store for chunks, transactional DB for metadata; ACLs + capability links; push notifications | Client sync agent, chunk/manifest service, object store, metadata DB, ACL service, notification service |
| **Gmail** | Mailbox-per-user write model; threads as groupings; mail index (search cross-ref §7.1); durable delivery queue with retry; spam classifier (ML cross-ref) | SMTP gateway, delivery queue, mailbox/thread store, mail index, spam service, push service |
| **Spanner** | Global replication via Paxos groups; TrueTime ordering; 2PC across groups; directory-based sharding | Tablet servers, Paxos group managers, TrueTime, 2PC coordinator, SQL front-end |

### 7.8 The Pattern Library: The Recurring Moves

Across all six deep-dives, the same eight patterns recur — learn them once, deploy them everywhere. These are the "components as parameters" (§10.4) you'll be recombining:

1. **Shard by key, sorted by access.** The Bigtable lesson (§6.4): choose the key so that access is local — user_id for mail, video_id for views, term-or-doc for the index. The shard key *is* the design; changing it changes everything downstream.
2. **Cache the head, compute the tail.** Popularity is a power law everywhere (queries, videos, tiles, files). Cache/precompute the head (top queries, top videos, city tiles, hot files) and let the tail be computed or fetched on demand. Naming the head/tail split with a number ("10% of queries are 90% of traffic") is a senior sentence.
3. **Queue every expensive async step.** Uploads → transcoding, mail → delivery, events → analytics: if a step is slow, batchable, or retryable, it goes behind a durable queue (cross-ref `message_queue_data_loss_guide.md` for the reliability rules; `distributed_rate_limiter_guide.md` for the throttling side).
4. **Write once, read many → object store + references.** Email blobs, video renditions, file chunks: store once by content hash, reference many times. Dedup is the default, not an optimization.
5. **Eventual consistency for derived data, strong for money.** Counters, feeds, freshness, analytics: eventual. Balances, entitlements, payments: strong (Spanner-style, cross-ref `apache_seata_guide.md`). The sentence "this is derived data, so eventual is correct" is an L5 signal.
6. **Degrade gracefully, never fail hard.** Search serves fewer results; video serves lower bitrate; mail queues delivery. Every Google-flavored design ends with the degradation story (§10.1 Phase 5).
7. **Push over poll.** Notifications, sync triggers, feed updates: push (websocket/streaming) beats polling at scale by orders of magnitude (§5.2 Example 3).
8. **Build offline, serve online.** Indexes, tiles, embeddings, thumbnails: anything precomputable is built in batch pipelines and pushed; the serving path only reads. Freshness becomes a pipeline-SLA problem, not a serving problem.

The drill: for each of your ten cases, label which patterns it uses. If a case uses fewer than four, you've missed something.

---

## 8. The Interview Day

### 8.1 The Structure of the Day

A typical Google onsite is **4–5 rounds of 45 minutes** (verified shape across aggregated reports; exact order varies). A representative day: two coding rounds in the morning, lunch (usually with a Googler — still an evaluation, stay professional but human), system design in the early afternoon, then Googleyness & Leadership and role-related knowledge. Each round is independent: interviewers write their feedback blind to the other rounds (the "no signal leakage" rule — the interviewer reads only your resume, not your other rounds' feedback), and the hiring committee assembles the picture later. Practical day-shape notes: arrive early (or test your video/audio setup an hour ahead for virtual loops), bring water, and treat the 10-minute gaps as reset time — *the previous round does not exist anymore*.

*(Flag: Google has repeatedly adjusted the loop — phone screens, virtual onsites, round counts — since 2020; the 4–5-round onsite with the round mix described in §1 is the consensus 2024–2026 shape, not a contractual guarantee.)*

### 8.2 The Time Management: The 45 Minutes

The 45-minute design round is a **budget**, and the single biggest time-management error is spending 20 minutes on requirements or 25 on a diagram nobody asked for. The canonical budget:

| Phase | Time | Non-negotiable exit condition |
|---|---|---|
| Requirements + restatement | ~5 min | The interviewer has agreed to your one-sentence contract |
| Estimation | ~5 min | You've stated QPS (avg+peak), storage, and one bandwidth number |
| Components (diagram) | ~10 min | Every box is labeled with one sentence of responsibility |
| Deep-dive | ~15–20 min | The interviewer's chosen component, 2+ levels deep |
| Trade-offs + summary | ~5 min | 2–3 trade-offs, flip conditions, 3-sentence recap, failure modes |

Two clock rules: **(1) watch the interviewer's watch** — interviewers give soft signals ("we have 15 minutes left") that mean *land the plane*: pick the highest-value remaining topic and go deep, then summarize; **(2) never run out of time mid-diagram** — a finished, summarized, trade-off-stated design beats a perfect-but-unfinished one every time, because the rubric scores the communicated arc, not the artifact.

### 8.3 The Communication: The Interviewer Expectations

Google interviewers consistently describe the same expectations (verified across published interviewer-side write-ups and coaching-site summaries):

- **Think aloud.** Silence is the enemy. A 30-second silent "let me think" is acceptable; 5-minute silent problem-solving is a fail — the interviewer cannot score what they cannot hear. Narrate your reasoning, including dead ends ("I'm considering X; the problem with X is...").
- **Lead, but let them steer.** You drive the framework; the interviewer injects constraints. Treat push-back as a gift: "good point — at that scale, my choice breaks; let's fix it" is a *strong* signal (flexibility + structure), while defending your first design to the death is a *weak* one (ego + rigidity).
- **Draw and talk.** Every diagram element gets a spoken sentence; every sentence connects to a decision. The interviewer should never have to ask "what is this box?"
- **Quantify opinions.** "Fast" is weak; "p50 under 100 ms, p99 under 300 ms, and here's what the index fan-out implies for that" is strong. Numbers are the Google accent.
- **Admit uncertainty honestly.** "I don't know the exact internals of X; here's what the design pattern tells me" is scored far above bluffing. This is also this series' honesty policy — the interview rewards the same behavior.

### 8.4 The Day Table

| Round | Focus | Tips |
|---|---|---|
| **Coding 1** | Algorithms, data structures, clean code | Talk through approach before writing; test your code aloud with a small example; handle edge cases explicitly |
| **Coding 2** | Same, possibly product-flavored | Expect a larger problem; modularize; name the complexity of each step |
| **System design** | The framework end-to-end (§4) | Budget the 45 minutes (§8.2); estimate before choosing; name trade-offs unprompted; end with failure modes |
| **Googleyness & Leadership** | Ambiguity, conflict, ownership, mentorship | STAR format; choose stories that show *judgment under ambiguity*; no humble-brag clichés; be specific about your actions vs. the team's |
| **Role-related knowledge** | Depth in your discipline | For SRE: debugging scenarios, incident playbooks, SLOs. For SWE: language/runtime depth, production debugging. Connect every answer to a system you actually built |

### 8.5 The Day Before, and the Logistics

The peak-performance frame (§1.3) applies to the logistics, not just the answers:

- **The day before:** do a light 30-minute design drill (one easy case from your library, out loud) to keep the machinery warm — then stop. No new material the night before; the mistake log (§9.1) is the only allowed review. Sleep is part of the preparation; the interview is a performance event, and a tired performer is a flat performer.
- **The setup:** for virtual loops, test camera, mic, shared-doc/whiteboard access, and the interview tooling *the day before*, not the morning of. A 15-minute setup failure can consume your first round's composure. For in-person: know the building, the check-in, and arrive 30 minutes early — the buffer is for your calm, not for traffic.
- **The day-of hygiene:** water, a snack in the bag, phone on silent, no caffeine spike right before a round. Between rounds: 10 minutes of reset — stretch, water, one breath exercise — and *no* post-mortem spiraling ("I bombed that coding round"); every round is independently scored, and the hiring committee reads signals, not stories.
- **The last round matters as much as the first.** Feedback per round is written blind to the others; a strong finish (the trade-off summary, the failure modes, the clean wrap-up) is a full-strength signal, not a consolation prize.

### 8.6 The Interviewer's Script (What They Are Doing Behind the Screen)

Understanding the interviewer's job turns the round from an interrogation into a collaboration. What the interviewer is actually doing, minute by minute:

- **They own one signal, not the verdict.** Each interviewer writes feedback on their assigned dimension (per the rubric §2.4) and does *not* see the other rounds. They are not deciding your fate; they are collecting evidence for the hiring committee. The corollary: win the round on its own terms — there is no carry-over, and no recovery needed from a bad earlier round.
- **Their moves are deliberate.** Silence = "keep talking, I'm assessing your structure". "Why?" = "justify this decision — I'm scoring the reasoning, not the choice". Scale injection ("what if it's 100×?") = "show me the failure boundary of your design". Picking the deep-dive = "show me your depth on the part I know is hardest". Each move maps to a rubric signal (§2.4) — read the move, play the signal.
- **They score the arc, not the artifact.** By the end they need: requirements handled, numbers stated, components clear, depth shown, trade-offs named, summary delivered. A design that is 80% perfect but stops at "and that's the architecture" scores below a 60%-perfect design that completes the arc — because the rubric's dimensions are all *process* dimensions.
- **They are calibrated to your level.** The same interviewer adapts the bar for L4 vs. L5 vs. L6 (§2). They know what "strong for this level" looks like because they see the level's bar in every debrief. Your job is to *signal your level* early (depth of trade-offs, ownership of the arc) so the calibration lands in your favor.
- **They want to write a positive report.** A good interview is pleasant for both sides; interviewers would rather write "strong hire" than argue a no-hire in the HC. Your clarity, honesty, and pace make their job easy — and that is not manipulation; it is the same collaboration the job itself requires.

---

## 9. The Practice Plan

### 9.1 The Plan: 4–6 Weeks

The canonical prep arc is **4–6 weeks** (the same structure as the ML-design plan in `ml_system_design_interview_guide.md` §11 — the two plans are designed to be run in parallel if you are prepping both). The logic: week 1 builds the machinery (framework + math), weeks 2–3 build the library (classic questions), week 4 builds the depth (deep-dives), weeks 5–6 build the performance (mocks under interview conditions).

- **Week 1 — The machinery.** Learn the framework (§4) cold: write it out from memory daily until it is automatic. Drill the scale math (§5) until QPS/storage/bandwidth estimation is reflex (10 drills/day, timed at 3 minutes each). Read the three Google papers' abstracts + architecture sections (GFS, Bigtable, Spanner — §6) and the Alex Xu *System Design Interview* chapters for 2–3 classic systems (cross-ref the ML guide §16 for the reading list).
- **Weeks 2–3 — The library.** One classic question per day, **written**: requirements → estimation → components → deep-dive → trade-offs, on paper or a whiteboard tool, 45 minutes timed. The list: Search, YouTube, Maps, Drive, Gmail, Spanner, + rate limiter (cross-ref `distributed_rate_limiter_guide.md` — already written for you), URL shortener, chat, news feed, cache, key-value store. By the end of week 3 you should have a **case library** of 10–12 one-page designs (§9.2).
- **Week 4 — The depth.** Re-do your weakest 4 cases with *forced deep-dives*: pick the hardest component and go 3+ levels. Add the reliability lens to every case ("what fails, what's the SLO, what's the recovery"). Read 2–3 real interview reports (igotanoffer, LeetCode discuss, teamblind) for each classic question to calibrate what interviewers actually probe.
- **Weeks 5–6 — The performance.** 3–4 **mock interviews** under real conditions (45 min, timer, an interviewer who pushes back, no notes). The mocks are the only way to train the *communication* muscle (§8.3), which is half the score. After each mock: 15 minutes of structured feedback (what phase was weak? where did you go silent? which trade-off was missing?), then re-run the same case's weak phase immediately.
- **Throughout** — keep a **mistake log**: every missed formula, every skipped restatement, every forgotten trade-off gets one line. The log is your final-week study list.

### 9.2 The Methods: Mocks and the Case Library

**Mock interviews** are non-negotiable — the round is a live performance (§1.3), and performance needs rehearsal. Options, in order of value per dollar: (1) a peer-prep partner — two candidates interviewing each other, alternating interviewer/candidate roles (the interviewer role teaches you the rubric — you literally grade the design you'd want to see); (2) paid mock services (designgurus, igotanoffer, Exponent-style platforms — verify current offerings; the market moves); (3) recorded self-mocks — 45 minutes against a timer, then *watch yourself*: silence, mumbling, and skipped phases are visible in a way they never are live. *(Flag: mock-interview service quality and pricing vary widely; treat reviews as anecdotal.)*

**The case library** is your personal "index" — one page per case containing: the one-line prompt; the clarifying questions you'd ask; the estimation numbers (avg/peak QPS, storage, bandwidth); the component diagram (boxes + one-sentence responsibilities); the deep-dive you chose and the 2–3 levels beneath it; the trade-offs (decision, reason, flip condition); and the 3-sentence summary. 10–12 cases covers the classic set (§3) plus 2–3 **your-domain** cases (for Jack: a payment-settlement design, an API-gateway design — the `banking/` guides are raw material). The library is not for memorizing answers — it is for pattern-matching: a new prompt gets mapped to "this is a YouTube variant" or "this is a Drive variant" in seconds, and your prepared depth transfers.

### 9.3 The Plan Table

| Week | Focus | Activities |
|---|---|---|
| **1** | The machinery | Framework memorized cold; 10 timed math drills/day; read GFS/Bigtable/Spanner papers + 2–3 Alex Xu chapters |
| **2–3** | The library | 1 classic case/day, 45 min timed, written; build the 10–12 case one-pagers; read interview reports per question |
| **4** | The depth | Re-do 4 weakest cases with 3-level forced deep-dives; add the reliability lens to every case (SLO, failure, recovery) |
| **5–6** | The performance | 3–4 full mocks (peer/paid/recorded); structured post-mortem per mock; re-run weak phases; drill the mistake log |

### 9.4 The Two-Week Emergency Plan and the Week-Of Checklist

**If you have only two weeks** (the deadline moved up, or you're reading this late): compress the plan — Week 1 = machinery + library (framework memorized; 2 cases/day written: Search and YouTube first, then Drive and one your-domain case); Week 2 = depth + performance (2 mocks, forced deep-dives on the four cases, mistake log). Skip breadth; four good cases plus the framework beat ten shallow ones, and the framework is what carries a novel prompt.

**The week-of checklist** (the last three days):

- [ ] Framework card: the five steps + the 5/5/10/20/5 budget, written out once from memory (if you can't, re-memorize).
- [ ] Math anchors: 86,400; 1B actions/day ≈ 11.6K QPS; peak = 5–10×; storage = records × bytes × retention × replication; the three worked examples from §5.2.
- [ ] Case library: 10–12 one-pagers, browsed once — pattern-matching refresh, not memorization.
- [ ] The Google-stack one-liners: GFS/Bigtable/Spanner roles + trade-offs (§6.3 table).
- [ ] Two full 45-minute mocks with feedback (peer or recorded) — the communication muscle must be exercised, not just read about.
- [ ] Logistics: setup test, directions, the day-of plan (§8.5).
- [ ] The rest day before: light drill, early night.

*Flag:* the plan assumes a solid systems background (you have read the cross-referenced deep-dives in this series). If the basics — load balancers, caches, queues, databases — are shaky, add a week-0 that reads `cloud_providers_guide.md` and the system-design primer materials listed in `ml_system_design_interview_guide.md` §16.

---

## 10. The Worked Example: Designing Google Search

### 10.1 The Walkthrough: Requirements, Estimation, Components, Deep-Dive, Trade-offs

This is the full script for the canonical question — study it as a *template*, then rehearse it out loud. The interviewer's opening: **"Design Google Search."**

**Phase 1 — Requirements (5 min).** *You:* "Before designing, let me clarify. Is this the full product — crawling, indexing, serving — or the serving side? What's the primary user action?" *Interviewer:* "Full system, but focus on serving; assume the crawl exists." *You:* functional requirements: (a) user types a query → relevant results page with title, URL, snippet, in ~300 ms; (b) basic query understanding — spelling, synonyms; (c) freshness — new pages appear in hours, not weeks. Non-functional: scale (1B+ users, ~100K QPS avg — §5.2), availability (target 99.9%+; search is a "must always work" product — degraded results beat no results), latency p50 ~200 ms / p99 ~500 ms, consistency (eventual is fine — stale-by-minutes is invisible to users). *Restatement:* "So: a serving system that takes 100K+ QPS of short queries, returns ranked results with snippets within ~300 ms, at 99.9% availability, over an index of hundreds of billions of pages, with eventual consistency for freshness. Correct?" — the interviewer nods; the contract is set.

**Phase 2 — Estimation (5 min).** *You:* "8.5B searches/day ÷ 86,400 ≈ ~100K QPS average — that's the widely cited figure, I'll flag it as third-party — peak 3–5× that, so ~300K–500K. Each query fans out to, say, 50–100 index shards in parallel, so back-end index QPS is ~10–50M at peak. Index: hundreds of billions of pages × ~10 KB of postings per page ≈ 1–10 PB, replicated 3× → 3–30 PB. Cache: ~10% of queries are repeats; a 1 TB edge cache of top queries absorbs a huge fraction of load. Bandwidth: 500K peak QPS × ~50 KB per results page ≈ 25 GB/s egress — that's CDN/edge territory." — numbers, order of magnitude, and the fan-out insight: *the index, not the front door, is the real scale*.

**Phase 3 — Components (10 min).** The diagram, left to right: **Client** → **Edge/load balancer** (with a **rate limiter** at the edge — cross-ref `distributed_rate_limiter_guide.md` — and a **query cache**) → **Query front-end** (parses, spell-checks, rewrites; logs queries) → **Index shards** (the document-partitioned inverted index, each shard a Bigtable-like store of term → postings; plus a hot-term replicated tier) → **Ranking service** (features: query-doc signals, PageRank-style authority, freshness, location; a learned model — cross-ref `ml_system_design_interview_guide.md` for the model-serving layer) → **Snippet generator** → **Results assembler** (merge, dedupe, layout). Background: **Crawler → Indexer** (MapReduce/Dataflow-style batch pipeline over a GFS-like store) → **Index push** to serving shards. One sentence of responsibility per box, spoken while drawing.

**Phase 4 — Deep-dive (15–20 min).** The interviewer picks the index — as expected. *You go three levels:* **Level 1 — partitioning.** Document-partitioned shards (each query hits all shards, merge by rank) because the query mix is too broad for term partitioning; hot terms handled by replicating popular-term shards or a two-tier "head index" of common terms cached in memory. **Level 2 — storage format.** Per shard, a sorted map of term → postings (Bigtable-style tablets); postings compressed (delta-encoded doc IDs, varint); a **skip-list/block index** per posting list so ranking reads only the top candidates, not the full list — reading 10K of 10M postings for "the". **Level 3 — freshness and failure.** Two index generations: the base index (built offline, hours old) plus a **real-time layer** (recently crawled pages held in memory, merged at query time) — the classic "bigtable + memtable" pattern. Shard failure: replicas serve reads; a shard down = slightly fewer results, not an outage — *degradation by design*, which is the availability story. When the interviewer pushes ("what if a term appears in 1B documents?"), you answer: posting lists that long are never fully scanned — the block index + early termination + the ranking model's candidate cap make the scan O(top-K) in practice.

**Phase 5 — Trade-offs and wrap-up (5 min).** *You:* "Three trade-offs. (1) Document vs. term partitioning: I chose document for serving — uniform shard load, but every query pays a fan-out; I'd flip to term partitioning if queries clustered on few terms (then hot-shard replication becomes the cost). (2) Freshness vs. latency: the real-time layer gives minutes-level freshness at the cost of a second code path and merge complexity; if freshness were non-critical I'd drop it and serve only the batch index. (3) Availability vs. consistency: we serve stale results rather than error — search's contract is 'something relevant, fast, always'; a payments system (see the `banking/` guides) would make the opposite choice. Failure modes: index shard loss → degraded coverage, not outage; cache stampede on a breaking-news query → single-flight + rate limiting; index push corruption → versioned rollback of the shard generation. Summary: edge + cache → query front-end → fan-out to document-partitioned index shards → learned ranking → snippets, with an offline crawl→index pipeline and a real-time freshness layer, at ~100K QPS average, ~300 ms p50, 99.9% availability, degraded-by-design under failure." — *done, on time, with trade-offs spoken and failure modes named.*

### 10.2 The Interviewer Notes: The Expected Answers

What the interviewer is listening for, by level (synthesized from published interview-account write-ups and coaching rubrics — flag as synthesized, not an official rubric):

- **At L4:** the arc exists (clarify, estimate, draw, one deep-dive, some trade-offs); numbers are order-of-magnitude correct; the inverted-index concept is right; when pushed on the posting-list length, the candidate recovers or asks for guidance. Pass signal: "coherent and buildable by a competent engineer."
- **At L5:** trade-offs are *unprompted* and *reasoned*; the fan-out math is stated before the interviewer asks; the deep-dive goes to storage format and failure behavior; the candidate leads the conversation and reacts structurally to the 1B-document push (no defensiveness). Pass signal: "I'd trust this person to design a real service and defend it in review."
- **At L6:** the candidate *frames* the problem ("is this a serving problem or an end-to-end problem?") and phases the build (v1 single-region, then sharding, then multi-region); names the org interfaces (the index is a *product* other teams consume; ranking owns the model; crawler owns freshness SLAs); and when the interviewer says "make it 10×" the answer includes cost and capacity ("10× means ~300 PB of index — that's a fleet and a data-center plan, not a shard count"). Pass signal: "this person designs systems the way the company builds them."

The **common fails** (worth memorizing): designing without a restated contract (interviewer and candidate discover they built different products); skipping estimation entirely; a diagram with unlabeled boxes; a deep-dive that stops at "and then a cache makes it fast"; silence > 60 seconds; defending a broken choice under push-back.

### 10.3 The Lessons: Think at Google Scale

1. **Scale is a verb.** Every design decision is a function of the numbers: partitioning exists *because* of the fan-out math; the real-time layer exists *because* the batch pipeline takes hours; the CDN exists *because* TB/s can't cross the internet. If you cannot tie a component to a number, you cannot defend it.
2. **Degradation is a feature.** Google-scale systems fail constantly and are *designed* for it: replicas, versioned generations, graceful loss of coverage. The interview wants your failure story, not your uptime boast.
3. **The process is the score.** Requirements → estimation → components → deep-dive → trade-offs, communicated continuously, is the rubric. Depth without structure is a lost round; structure without depth is an L4 ceiling.
4. **Trade-offs are the senior signal.** Stating a decision and its rejected alternatives, with flip conditions, is the difference between L4 and L5 in the same answer.
5. **Your domain transfers.** The Search script — inverted index, fan-out, learned ranking, degradation — is the same script you'd run for a bank's search over transactions, a settlement index, or a market-data fan-out. The framework is portable; only the numbers change. (Cross-ref the `banking/` guides for domain-flavored worked designs.)

### 10.4 The Variations: When the Interviewer Changes the Prompt

The Search script adapts to the most common prompt mutations — rehearse at least these three:

- **"Design search for a bank's internal systems"** (your-domain variant): the inverted index is over transactions, counterparties, and documents — smaller by orders of magnitude (10^8 records, not 10^12), so sharding is optional and the *hard* problems shift to access control (every result must respect entitlements — ACL filtering *inside* the index, not after ranking), auditability, and joins with account data. The trade-off conversation flips from scale to security-and-governance; the framework is identical. (Cross-ref the `banking/` guides.)
- **"Design image/visual search"**: the inverted index becomes an **embedding index** — images → vectors via a model (cross-ref `ml_system_design_interview_guide.md` for the embedding pipeline), approximate nearest-neighbor search (ANN: HNSW/IVF-style indexes) replaces exact term matching, and the pipeline gains an offline embedding job plus a hybrid path (text-metadata index + vector index, fused at ranking). The scale math is the same shape; the components swap.
- **"Design search over 10B documents with p99 under 100 ms"**: the constraint tightens everything — 100 ms p99 forces the fan-out to be near-parallel, the index to be memory-resident (SSD only for the tail), and query caching to be aggressive; the answer is "the same architecture, but every component now has a latency budget and the budget is enforced" — walking the budget from edge to index and naming where each millisecond goes is the senior move.

The lesson across all three: the framework is the invariant; the components are parameters. Naming *which parameters changed and why* is what makes a variation answer feel expert rather than memorized.

### 10.5 What the Real Google Search Looks Like (Verified Facts)

Grounding the exercise in what Google actually publishes keeps your invented architecture honest:

- **Crawling → indexing → serving** is Google's own three-stage description of Search (developers.google.com, "How Search works" — verified). Crawlers discover URLs; the indexer analyzes pages and stores them in the **index** — which Google describes as spanning **hundreds of billions of pages** (verified wording); serving matches queries against the index and ranks with "hundreds of factors" including location, language, and device (verified).
- **Ranking is a learned model over many signals.** The exact model is proprietary (flag) — the public facts are the factor count and the pipeline shape (query understanding → retrieval → ranking → snippets). PageRank (the 1998 paper) is the historical authority signal and the canonical interview reference; modern ranking is ML (cross-ref `ml_system_design_interview_guide.md`).
- **The availability/latency envelope.** Google does not publish a Search SLA (flag); the widely reported user-facing targets are ~200–400 ms query latency and 99.9%+ availability — consistent with the design constraints used in §10.1, but treat the numbers as community consensus, not official.
- **Freshness is a pipeline property.** Google describes pages being added and updated as the crawler revisits them — freshness is bounded by crawl frequency, which is why the interview design puts freshness in the *index-build* path (batch + real-time layer), not in the serving path.
- **The scale figure.** 8.5 billion searches/day (~98K QPS average) is the widely repeated third-party figure; Google has never published a search-volume number (flag) — use it in interviews with the same caveat you'd use here: "the commonly cited figure is...".

---

## 11. The Summary and the Glossary

### 11.1 The One-Page Summary: Google System Design

**The loop:** recruiter screen → phone screens → onsite (4–5 × 45-min rounds: coding, system design, Googleyness & Leadership, role-related knowledge) → hiring committee → team matching. System design is a standalone round at **L5+** (1 round; 2–3 at L6), sometimes folded into coding at L4. SRE variants weight reliability (SLOs, failure modes, capacity).

**The expectations:** **L4** — correct, well-scoped component design with basic scale literacy. **L5** — end-to-end product design with justified trade-offs, fluent scale math, and conversation leadership. **L6** — design under ambiguity with phased plans, org-level interfaces, and operational breadth. The rubric signals: general cognitive ability, role-related knowledge, leadership, Googleyness — measured by *process*, not artifact.

**The framework (the autopilot):** (1) Requirements — clarify + restate the contract; pin consistency and read:write ratio. (2) Estimation — QPS (avg + peak = 5–10×), storage (+replication +growth), bandwidth; say what breaks first at 10×. (3) Components — labeled diagram, one-sentence responsibilities, Google-stack building blocks (GFS-like blobs, Bigtable-like wide-column, Spanner-like transactions). (4) Deep-dive — 2+ levels into the interviewer's pick; recover from under-design aloud. (5) Trade-offs — 2–3 decisions with reasons, rejected alternatives, flip conditions, failure modes, and a 3-sentence summary. Budget: 5/5/10/20/5 minutes.

**The canon:** classic questions — Search, YouTube, Maps, Drive, Gmail, Spanner — each a variant of the same arc; the Google stack papers (GFS 2003, Bigtable 2006, Spanner 2012) as design vocabulary; Alex Xu's *System Design Interview* and this series' deep-dives (`distributed_rate_limiter_guide.md`, `s3_architecture_guide.md`, `message_queue_data_loss_guide.md`, `ml_system_design_interview_guide.md`) as the library. Prep: 4–6 weeks — machinery, library, depth, performance — ending in 3–4 real mocks and a 10–12-case library.

**The 45-minute cheat sheet** (last read before the door):

1. Clarify → restate the contract (at least two scoping questions).
2. Estimate: QPS avg + peak · storage ×3 replication ×3 growth · one bandwidth number · "what breaks at 10×".
3. Diagram: every box labeled, one sentence each; name the building blocks (Bigtable-like store, object store, durable queue).
4. Deep-dive: their pick, 2+ levels; recover aloud if under-designed; never bluff.
5. Trade-offs: 2–3 with flip conditions · failure modes · 3-sentence summary · thank the interviewer.

### 11.2 Think at Google Scale — The Final Word

"Engineering peak" is not a Google term — it is the discipline this guide is built on: the interview is a short, high-stakes window in which you must perform your best structured thinking on demand, and performance is trainable. The framework gives you calm (the autopilot), the math gives you clarity (the numbers anchor every decision), and the mocks give you speed (the communication muscle). On the day, you are not being tested on whether you have built a billion-user system — almost nobody has. You are being tested on whether you can *think like someone who would*: clarify before designing, estimate before choosing, name the trade-off before being asked, and keep talking. Do that, and the system design round — at Google or anywhere — stops being an obstacle and becomes the round where you shine. That is the peak.

### 11.3 The Glossary

- **Google** — the company whose interview process this guide targets; also shorthand for "the Google bar" — company-wide, hiring-committee-reviewed, level-calibrated hiring standards.
- **Interview loop** — the full sequence of interview stages: screens → onsite rounds → hiring committee → team matching; "the loop" often refers to the onsite rounds collectively.
- **Onsite** — the in-person (or virtual) interview day: 4–5 × 45-minute rounds, each independently scored by an interviewer who has not seen the other rounds' feedback.
- **SWE** — Software Engineer; the generalist engineering track; the system design round is a core SWE onsite round at L5+.
- **SRE** — Site Reliability Engineer; the reliability track; design rounds weighted toward SLOs, error budgets, capacity, and failure modes; the old name for the design exercise was NALSD (Non-Abstract Large Systems Design).
- **L4 / L5 / L6** — Google's engineering levels (SWE II / Senior / Staff); the same design round is graded against different expectations per level (§2).
- **System design** — the open-ended architecture interview: design a system at scale, scoring the process (clarify, estimate, design, trade-off, communicate) rather than a single correct answer.
- **Framework** — the five-step autopilot of this guide: requirements → estimation → components → deep-dive → trade-offs.
- **Requirements** — the clarification phase: functional + non-functional questions, followed by a restated contract the interviewer agrees to.
- **Estimation** — the scale-math phase: order-of-magnitude QPS, storage, bandwidth numbers that drive design decisions.
- **QPS** — queries (requests) per second; average = (DAU × actions/day) ÷ 86,400; peak ≈ 5–10× average.
- **Storage** — bytes of data at rest: records × bytes × retention × replication; growth is part of the answer.
- **Bandwidth** — bytes per second over the wire: QPS × payload for serving; producers × rate × payload for ingest.
- **GFS** — the Google File System (2003 paper): cluster-scale file system, 64 MB chunks, 3× replication, master + chunkservers; the blob layer of the Google stack.
- **Bigtable** — Google's wide-column sorted map (2006 paper): tablets sharded by key, SSTables on GFS; the high-throughput key-value layer of the stack.
- **Spanner** — Google's globally distributed, externally consistent SQL database (2012 paper): TrueTime, Paxos groups, 2PC across groups; the strongly-consistent transactional layer.
- **Search** — "design Google Search": inverted index, sharded index, query fan-out, ranking, snippets; this guide's worked example.
- **YouTube** — "design YouTube": chunked resumable upload, async transcoding DAG, CDN + adaptive bitrate streaming, sharded counters.
- **Maps** — "design Google Maps": tiled spatial data, tile/CDN serving, global graph routing, live-traffic ingestion.
- **Drive** — "design Google Drive": content-defined chunking + dedup, manifest-based sync, conflict resolution, object store + transactional metadata, ACL sharing.
- **Gmail** — "design Gmail": mailbox model, threads, mail index, durable delivery with retry, spam classification.
- **Index** — the inverted index: term → postings list, sharded and replicated; the heart of any search design.
- **Ranking** — turning retrieved candidates into an ordered result list: query understanding → retrieval → learned scoring → snippets; PageRank as the original authority signal.
- **Streaming** — (a) media delivery (adaptive bitrate, CDN) in the YouTube deep-dive; (b) the streaming-data pattern (cross-ref `event_stream_processing_guide.md`) for ingest pipelines like traffic or clicks.
- **Sync** — the Drive client model: chunked deltas, manifest hashes, conflict resolution (LWW / versioned / OT-CRDT), push-over-poll notifications.
- **Trade-offs** — the scored phase: a decision, its reason, the rejected alternatives, and the conditions under which you would flip it.
- **Deep-dive** — the focused phase: 2+ levels into the component the interviewer (or you) selects; where L4/L5/L6 separate.
- **45 minutes** — the standard design-round length; the time budget (5/5/10/20/5) is a scoring constraint, not a suggestion.
- **Mock interview** — a rehearsed 45-minute design round with an interviewer who pushes back; the only way to train the communication muscle; post-mortem and re-run are part of the method.
- **Case library** — your personal collection of 10–12 one-page designs (classic + your-domain) used for pattern-matching new prompts on the day.
- **Engineering peak** — the interpretive frame of this guide (flagged, not Google terminology): the interview as a peak-performance event — calm + clarity + speed, reachable through framework, math fluency, and rehearsal.

### 11.4 The Claims-Status Appendix

Per this guide's honesty policy (header), the verification status of the load-bearing facts:

- **Verified (primary or near-primary):** the Google interview pipeline shape (recruiter screen → phone screens → onsite → hiring committee → team matching) and the rubric signal names (general cognitive ability, role-related knowledge, leadership, Googleyness) — from Google recruiting material and consistent interview accounts; the GFS/Bigtable/Spanner papers' architecture facts; Search's crawl → index → serve three-stage description and "hundreds of billions of pages" index scale (developers.google.com); YouTube's 500 hours/min uploads and 1B+ hours/day watch time (Wikipedia + widely repeated statistics); the SRE design-round reliability weighting and NALSD heritage (SRE interview guides).
- **Flagged (third-party or interpretive):** "8.5 billion searches/day" and derived QPS (~98K avg, ~300K–1M peak) — third-party figure, Google does not publish search volume; Search latency/availability targets (~200–400 ms, 99.9%) — community consensus, no official SLA; interview *statistics* (round counts per level, question frequencies, mock-service quality) — aggregated and anecdotal; the L4/L5/L6 expectation descriptions — synthesized from interview write-ups, internal rubrics are not public; "engineering peak" — this guide's interpretive framing, not Google terminology; the mock-interview market (services, pricing, outcomes) — moves fast, treat reviews as anecdotal.
- **Cross-verified:** the component deep-dives in §7 align with this series' other guides (`distributed_rate_limiter_guide.md`, `s3_architecture_guide.md`, `message_queue_data_loss_guide.md`, `event_stream_processing_guide.md`, `ml_system_design_interview_guide.md`) and with the public system-design canon (Alex Xu, ByteByteGo, system-design-primer).

### 11.5 The Companion Reading List

The shortest path from this guide to interview-ready, in reading order:

1. **This series' deep-dives** (each is a complete worked design you can reuse as a case-library entry): `distributed_rate_limiter_guide.md` (the rate-limiter component), `s3_architecture_guide.md` (the object-store component), `message_queue_data_loss_guide.md` (the queue-reliability component), `event_stream_processing_guide.md` + the Kafka guides (the streaming component), `apache_seata_guide.md` (the distributed-transaction component), `ml_system_design_interview_guide.md` (the ML-design sibling — framework, case studies, practice plan, prompt bank).
2. **The Google papers**, in order: GFS (2003) → Bigtable (2006) → MapReduce (2004, read alongside) → Spanner (2012). Read each for the *lesson* (§6.4), not the internals.
3. **The canon**: Alex Xu, *System Design Interview* Vols 1–2 (the 15+ classic systems); ByteByteGo's materials for the component visualizations; the system-design-primer (GitHub) for breadth — all cross-referenced in `ml_system_design_interview_guide.md` §16, which reviews the same canon.
4. **The SRE book** (Google, O'Reilly) — chapters on SLOs/error budgets and on hiring — for the reliability dialect and the culture frame (§1.5).
5. **Primary sources for this guide's facts**: developers.google.com "How Search works" (crawl → index → serve; index scale); the paper abstracts; Wikipedia's YouTube statistics page; and your recruiter, for the loop details that matter for *your* application (the only authority on your loop — §1.6).

---

*End of guide. Next in the series: a banking-domain worked design (e.g., "design a real-time payments system at Google scale") to exercise the same framework in the Cymbal Bank context — see the `banking/` guides for the domain raw material. If you spot a factual claim that should be flagged or corrected, the honesty policy in the header is the contract: verify, then patch.*
