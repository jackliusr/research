# NALSD — Non-Abstract Large System Design: The Concrete-Design Study Guide

*A study guide to **NALSD (Non-Abstract Large System Design)** — the design methodology behind Google SRE's systems-design interview round and, by extension, the mindset behind every serious large-system design interview. The guide covers the verified definition and the term-origin investigation (what Google's own material says, what interview-prep sources repeat, and what remains ⚠ unverifiable), the NALSD-vs-abstract/academic framing (concrete requirements, numbers, contracts, then architecture — versus textbook layered diagrams), the interview structure as practiced in prep (requirements gathering → capacity estimation → API and data-model design → high-level architecture → bottleneck deep-dive → trade-offs and wrap-up), the estimation discipline (traffic, QPS, storage, bandwidth back-of-envelope — cross-referenced to the repo's capacity guide rather than re-derived), the trade-off analysis axes, the interview formats across FAANG-style loops with honest ⚠ flags, the common candidate failure modes with NALSD remedies, condensed NALSD-style readings of the repo's own large-system guides (the trading system, the middleware platform, the posting engine — cross-refs only), a full Cymbal Bank worked example (a real-time payments and accounting service at Cymbal Bank scale — clearly marked design fiction), the shelf map against the Grokking companion, the Alex Xu Insider's Guide companion, and the Google rubric guide, a claims audit with a "What Could Not Be Verified" section, a glossary, and the closing summary. The final word of the guide: the interview round is won or lost on **the concrete design.***

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Career Skills / System Design (technology/) — the METHODOLOGY deep-dive on the system-design-interview shelf
> **Audience:** Solution architects, platform engineers, backend engineers, SREs, and system design interview candidates
> **Last Updated:** September 2026

**Cross-references (the repo's system-design-interview shelf this guide sits on):** [google_system_design_interview_guide.md](google_system_design_interview_guide.md) (the Google rubric — requirements → estimation → components → deep-dive → trade-offs; its §1.2 records the SRE-design heritage this guide investigates), [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) (the Educative 7-step framework), [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (the Alex Xu 4-step framework), [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (the theory — the "why" under the interview "how"), [ml_system_design_interview_guide.md](ml_system_design_interview_guide.md) (the ML-design sibling), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (reliability/nines), [capacity_sizing_guide.md](capacity_sizing_guide.md) (the sizing discipline — the numbers layer this guide's §4 condenses), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) and [s3_architecture_guide.md](s3_architecture_guide.md) (the component deep-dives). **The worked large systems (cross-referenced, never re-derived):** [trading_system_software_architecture_guide.md](trading_system_software_architecture_guide.md), [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), and the banking pair [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) and [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) (link convention: plain filename for `technology/` siblings, `../banking/` prefix for the banking folder).

**Integrity convention (repo standard).** Every factual claim carries one of three marks: **✅** verified this pass against a primary or authoritative source (named in the claims audit, §11); **⚠** flagged — reported, approximate, single-sourced, contested, or not pinned to an authoritative public source this pass; **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what a load balancer does, what QPS means). Where a source could not be reached or a claim could not be confirmed, it is said so plainly and moved to §11.5 "What Could Not Be Verified" — never invented. The Cymbal Bank worked example in §9 is **design fiction** built on the verified facts of the guide, per house convention.

---

## Table of Contents

1. [The Overview](#1-the-overview) — the short answer, the key-facts table, why a design-interview candidate should care, and where this guide sits on the shelf
2. [The Concept: What NALSD Is](#2-the-concept-what-nalsd-is) — the verified definition, the term-origin investigation (✅/⚠ per claim), NALSD vs the abstract/academic design approach
3. [The Method: The Interview Structure](#3-the-method-the-interview-structure) — requirements → estimation → API/data model → architecture → deep-dive → wrap-up, and what each phase must produce
4. [The Estimation Discipline](#4-the-estimation-discipline) — traffic, QPS, storage, bandwidth back-of-envelope; the numbers a candidate must carry (cross-ref `capacity_sizing_guide.md`)
5. [The Trade-off Analysis](#5-the-trade-off-analysis) — the recurring trade-off axes (consistency, read/write shape, SQL vs NoSQL, push vs pull, caching tiers) — condensed, not re-derived
6. [NALSD in the Interview: The Formats](#6-nalsd-in-the-interview-the-formats) — where this round appears across FAANG-style loops (⚠ where unverifiable), time-boxing, what interviewers score
7. [The Practice Patterns: Walkthroughs via the Repo's Large Systems](#7-the-practice-patterns-walkthroughs-via-the-repos-large-systems) — condensed NALSD readings of the trading system, the middleware platform, and the posting engine (cross-refs only)
8. [The Common Failure Modes](#8-the-common-failure-modes) — the candidate mistakes, each with its NALSD remedy
9. [The Cymbal Bank Worked Example](#9-the-cymbal-bank-worked-example) — a real-time payments and accounting service at Cymbal Bank scale, designed the NALSD way (design fiction)
10. [The Shelf Map: NALSD vs Grokking vs the Xu Insider's Guide vs the Google Rubric](#10-the-shelf-map-nalsd-vs-grokking-vs-the-xu-insiders-guide-vs-the-google-rubric) — the framework comparison and guide mapping
11. [The Claims Audit](#11-the-claims-audit) — ✅/⚠/❌ per key claim, ending with §11.5 What Could Not Be Verified
12. [The Glossary](#12-the-glossary)
13. [Cross-references and the Closing Summary](#13-cross-references-and-the-closing-summary)

---
## 1. The Overview

### 1.1 The Short Answer

**NALSD — Non-Abstract Large System Design — is the system-design methodology of Google's Site Reliability Engineering organization: design iteratively from the problem statement, gather requirements, and keep refining until the design is expressed in concrete, quantified resources — machines, disk, RAM, queries per second — that provably satisfy the requirements at real scale.** ✅ The canonical published description is Chapter 12 of the *Google SRE Workbook* ("Introducing Non-Abstract Large System Design", sre.google/workbook/non-abstract-design/ — verified this pass). ✅ Google's own materials confirm that the same style is used as an interview format: "Google SRE has developed a special interview format called 'Non-Abstract Large Systems Design' or NALSD… All candidates for SRE positions at Google participate in one NALSD interview as part of their recruiting process" (Sebastian Kirsch, "Interviewing for Systems Design Skills", SREcon18 Asia / USENIX, June 2018 — verified via research.google/pubs and usenix.org).

The defining idea is in the name: **a design is not finished while it is abstract.** A whiteboard of boxes labeled "database", "cache" and "load balancer" is the *starting point*, not the answer. The NALSD discipline iterates that diagram until every box stands on concrete numbers — how many machines, how much disk per machine, how many queries per second per replica — and lets the numbers reject or reshape the architecture. This guide is the methodology deep-dive: where the term comes from (with honest ⚠ flags where the trail goes cold), how the method maps to the interview structure practiced across interview prep, the estimation and trade-off disciplines it demands, and how to practice it using the repo's own large-system guides and a Cymbal Bank worked example.

### 1.2 The Key-Facts Table

| Field | Value | Status |
|---|---|---|
| Expansion | Non-Abstract Large System Design (sources also write "Non-Abstract Large Systems Design"; never an acronym for networking/application/Linux/system-design dimensions — those are skills, not the expansion) | ✅ / ⚠-wording |
| Owner of the term | Google SRE (Site Reliability Engineering); taught via SRE Classroom workshops; used in Google SRE interviews | ✅ |
| Canonical published description | *Google SRE Workbook* (O'Reilly, 2018), Ch. 12 "Introducing Non-Abstract Large System Design" — free at sre.google/workbook/non-abstract-design/ | ✅ |
| Interview-format documentation | Sebastian Kirsch, "Interviewing for Systems Design Skills", SREcon18 Asia, USENIX, Singapore, 8 June 2018 — one NALSD interview for every Google SRE candidate | ✅ |
| Core idea | Design is not done while it is abstract: iterate whiteboard → requirements + SLOs → concrete resource estimates (machines, disk, RAM, QPS) → a design the numbers prove | ✅ |
| The method, in brief | Two phases — (1) a basic design that works in principle, (2) scaling it up — probed by the four NALSD questions: *Is it possible? Can we do better? Is it feasible? Is it resilient?* | ✅ |
| Definition of the skill | "The ability to assess, design, and evaluate large systems" — capacity planning + component isolation + graceful degradation | ✅ |
| Interview genre | The concrete-numbers systems-design round; SRE-flavored sibling of the generic FAANG design round | ✅ / ⚠-nuance |
| Earliest public artifact found | 2018 (SRE Workbook Ch. 12; SRECon18 talk). Whether the term predates 2018 inside Google is unverifiable this pass | ⚠ |
| Usage outside Google | No evidence found this pass that Amazon/Meta/Microsoft/Netflix use the NALSD *label*; the concrete-numbers style is generic to senior design rounds | ⚠ (weak negative) |
| One-sentence summary | Requirements → numbers → contracts → architecture, iterated until the resources are concrete and the failure modes are designed for — not drawn around. | — |

### 1.3 Why a Design-Interview Candidate Should Care

Three reasons, in increasing order of importance:

1. **It is a named, documented round.** Unlike most interview folklore, NALSD is not a rumor: Google's SRE organization publishes the methodology (SRE Workbook Ch. 12) and has presented the interview format at USENIX SRECon. If you are interviewing for an SRE, infrastructure, or reliability-adjacent role — including "design in my domain" variants at banks — the concrete-numbers style is a named, checkable target rather than an ambiguous "design something" prompt.
2. **The style has leaked into every senior design round.** Interviewers who have never heard the acronym still ask the NALSD question: *"how many machines is that?"*, *"what is the write throughput per shard?"*, *"what happens when the datacenter fails?"*. The repo's own Google rubric guide and the Grokking and Alex Xu companions all encode the same requirements → estimation → architecture → trade-off shape (cross-ref §10). NALSD is that shape with the reliability lens turned all the way up.
3. **It rewards exactly what a Solution Architect already does.** The method is requirements gathering, SLO setting, capacity math, contract design, and trade-off analysis against operational cost — the working toolkit of an architect, formalized. For a banking technologist, the NALSD mindset is the difference between a diagram and a defensible design: the numbers that say whether a real-time posting service fits in the batch window, whether a middleware estate's event backbone has enough partitions, whether a payment rail's peak can be absorbed (cross-ref the worked large systems in §7 and the Cymbal Bank exercise in §9).

### 1.4 Where This Guide Sits on the Shelf

This guide is the **methodology + interview-genre layer** of the repo's system-design-interview shelf. The shelf's other layers:

- **The framework companions** teach the interview *shape*: [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) (Educative's 7-step framework), [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (Alex Xu's 4-step framework), [google_system_design_interview_guide.md](google_system_design_interview_guide.md) (the Google rubric and scale math). NALSD is the underlying *discipline* these shapes operationalize — the "why concrete?" behind the "what do I say next?".
- **The component and theory guides** supply the parts: [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md), [s3_architecture_guide.md](s3_architecture_guide.md), [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (the consistency/partitioning theory), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (reliability targets).
- **The numbers layer** is [capacity_sizing_guide.md](capacity_sizing_guide.md) — this guide's §4 condenses its discipline to the back-of-envelope form an interview demands.
- **The worked large systems** are the practice corpus: §7 reads [trading_system_software_architecture_guide.md](trading_system_software_architecture_guide.md), [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), and the banking pair [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) / [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) through the NALSD lens — condensed and cross-referenced, never re-derived.

### 1.5 How to Use This Guide

**Interview candidates:** read §2 (what NALSD is, honestly sourced) and §3 (the structure) first; drill §4 (estimation) and §5 (trade-offs) until the axes are reflex; practice the failure-mode checklist of §8 against the walkthroughs of §7; then rehearse with the §9 worked example as a script. **Architects and hiring managers:** §2 and §6 tell you what the round measures and where it appears; §7 shows the method applied to systems you already know from the shelf. **Everyone:** §11 is the honesty ledger — every claim with its ✅/⚠/❌ status — and §11.5 collects what could not be verified this pass.

### 1.6 The Two Reading Paths

This guide serves two speeds of reader, and the sections map differently for each:

| Reader | Path | Time |
|---|---|---|
| **The orientation read** — architects and managers who need to know what NALSD is and whether the round uses it | §1 → §2 (the concept and the origin, honestly flagged) → §6 (the formats) → skim §11 (the ledger) | ~20 minutes |
| **The rehearsal read** — interview candidates who must perform the method | §3–§5 (structure, estimation, trade-offs) studied once, then §8 (failure modes) used as the self-scoring checklist while practicing each §7 walkthrough, then §9 (the worked example) delivered aloud under a 45-minute timer | ~2 hours per sitting, repeated across the four §7 systems |

**The rehearsal loop that makes the method stick:** for each practice system — one of the §7 walkthroughs, then the §9 example — run the four NALSD questions (§3.8) in order, then deliver the six beats (§3.2–§3.7) aloud, then score yourself against the §8 table and the §6.5 signals. The verbal delivery is the point: silent preparation does not transfer to a round that scores audible reasoning.

---
## 2. The Concept: What NALSD Is

### 2.1 The Verified Definition

The SRE Workbook's Chapter 12 — "Introducing Non-Abstract Large System Design", by Salim Virji, James Youngman, Henry Robertson, Stephen Thorne, Dave Rensin, and Zoltan Egyed, with Richard Bondi (O'Reilly, 2018; free online at sre.google/workbook/non-abstract-design/) — is the canonical definition. Verified statements from that chapter:

> "By following an iterative style of system design and implementation, we arrive at robust and scalable designs with low operational costs. We call this style Non-Abstract Large System Design (NALSD)."

> "NALSD describes a skill critical to SRE: the ability to assess, design, and evaluate large systems. Practically, NALSD combines elements of capacity planning, component isolation, and graceful system degradation that are crucial to highly available production systems."

> "Google SREs are expected to be able to start resource planning with a basic whiteboard diagram of a system, think through the various scaling and failure domains, and focus their design into a concrete proposal for resources."

The chapter's own "Why 'Non-Abstract'?" section states the thesis: *all systems will eventually have to run on real computers in real datacenters using real networks*, so the designer must exercise "the muscle of turning a whiteboard design into concrete estimates of resources at multiple steps in the process" — otherwise it is "too tempting to create systems that don't quite translate in the real world." It also carries an important calibration note for candidates: the exercises are driven "to discrete results (e.g., number of machines)", yet "examples of sound reasoning and assumption making are more important than any final values" — NALSD is about combining "many imperfect-but-reasonable results into a better understanding of the design," not about producing one perfect number.

### 2.2 The Term-Origin Investigation

The repo's sibling Google guide (google_system_design_interview_guide.md §1.2 and §11.3) records the claim that for SRE roles "the older SRE designation was NALSD — Non-Abstract Large Systems Design — and the mindset persists." This guide was asked to verify the origin independently. What the sources actually say:

| # | Claim | Status | Source note |
|---|---|---|---|
| 1 | NALSD is a genuine Google SRE term, not an interview-prep coinage | ✅ | SRE Workbook Ch. 12 (sre.google/workbook/non-abstract-design/) uses the name and defines it; Google SRE Classroom workshops teach it (sre.google/classroom/, and the distributed-pubsub and imageserver workshop pages); a Google Cloud blog post announces "Join SRE Classroom NALSD workshops" |
| 2 | NALSD is (or was) an actual Google SRE *interview format* | ✅ | Kirsch, "Interviewing for Systems Design Skills", SRECon18 Asia / USENIX, 8 June 2018: Google SRE "has developed a special interview format called 'Non-Abstract Large Systems Design' or NALSD… All candidates for SRE positions at Google participate in one NALSD interview" (research.google/pubs and usenix.org) |
| 3 | The term was "popularized by the Google SRE Workbook" | ✅ (as published artifact) | The Workbook's Ch. 12 is the canonical public text and is cited by prep glossaries (e.g. tieroneprep.com/glossary/nalsd) as the popularizer. Whether the term existed earlier inside Google is a separate, unverified question (see #5) |
| 4 | Google still uses NALSD today (methodology and teaching) | ✅ | sre.google/classroom/ workshop pages and sre.google resources pages (e.g. the "addressing cascading failures" book-update page, which introduces NALSD as the tool for designing against cascading failure) are live and current as of this pass (September 2026) |
| 5 | The coinage predates 2018 (internal Google interview lore, "the old systems-design round") | ⚠ | No dated public source earlier than 2018 was found this pass. The repo Google guide's "older SRE designation" phrasing suggests the *round name* evolved in recruiting materials while the methodology kept the NALSD label — plausible and consistent with #4, but the exact naming history of the round is not pinned to a Google primary source. See §11.5 |
| 6 | NALSD is the same thing as the generic FAANG system-design round | ⚠ (partly refuted) | Kirsch's abstract stresses reliability/robustness, provisioning estimation, and *managing change* — an operations-weighted variant of the generic round, not the same emphasis. Prep sources (⚠ third-party) describe NALSD prompts as often starting from an *existing production system* that is broken or at a physical limit, rather than a greenfield product. Generic-round material does not mention the NALSD label |
| 7 | Other FAANGs (Amazon, Meta, Microsoft, Netflix) use the NALSD label | ⚠ (no evidence) | Two targeted searches this pass returned no results tying the term to those companies' rounds. Absence of evidence, not evidence of absence — see §11.5 |
| 8 | "NALSD" expands to something other than Non-Abstract Large System Design (e.g., networking/application/Linux acronym games) | ❌ (rejected) | Multiple independent sources explicitly reject acronym-expansion readings; the expansion above is the only documented one (e.g. the AceInterviews SRE walkthrough notes the acronym is *not* an expansion of evaluation dimensions) |

**Who uses the term (this pass):** Google's own SRE organization (Workbook, SRE Classroom, Google Cloud blog, sre.google resources) — ✅ primary; the interview-prep ecosystem that targets Google SRE roles (AceInterviews' google-sre-interview-handbook on GitHub, its dev.to/hashnode walkthroughs, tieroneprep's guide and glossary, video prep channels) — ✅ third-party, consistent with the primary sources; and this repo's own Google rubric guide — repo-internal. No source was found using the term for non-Google design rounds.

### 2.3 NALSD versus the Abstract / Academic Design Approach

The clearest way to hold the concept is the contrast. Academic and textbook design teaching (and weak interview answers) produce an **abstract design**: a layered diagram of components — load balancer, application servers, database, cache, queue — connected by arrows, correct at the level of "this is the shape such systems have". NALSD demands the **non-abstract pass** over that same diagram: what each component concretely is, how much of it is needed, and why the numbers close.

| Dimension | Abstract / academic design | NALSD (non-abstract design) |
|---|---|---|
| Starting point | The reference architecture / textbook shape | The problem statement, the users' goal, and the constraints |
| Requirements | Often implied or skipped | Explicitly gathered and restated; functional + non-functional, often as SLOs |
| Numbers | Absent or decorative ("handles millions of users") | Central: QPS, payload bytes, storage growth, bandwidth, machine counts — every layer quantified |
| "Database" means | A box labeled database | Which engine, how many shards, RAM per shard, disk IOPS, replication factor, cost |
| Failure | Mentioned as a section, if at all | A design driver: "is it resilient?" asked of every component and of the whole |
| Iteration | One diagram, presented as final | Multiple passes — possible → better → feasible → resilient — each pass reshaping the design |
| Judgment of success | Looks like the canonical picture | The numbers close and the failure modes are designed for |
| Genre fit | University courses, architecture textbooks, TOGAF-style artifacts | Production SRE work and senior system-design interviews |

The SRE Workbook's AdWords example (Ch. 12) is the canonical demonstration: it starts with requirements and SLOs (99.9% of dashboard queries complete in under one second; 99.9% of the time displayed CTR data is less than five minutes old; assumed 500,000 search queries/sec and 10,000 ad clicks/sec), tries a **one-machine** design first, computes its limits, and iterates through sharded in-memory maps and a LogJoiner pipeline until the multi-datacenter design closes — "we need approximately 4 TB of RAM in each datacenter to host the sharded ClickMap and QueryMap. If we have 64 GB of RAM per machine, we can serve the data from just 64 machines, and will use only 25% of each machine's network bandwidth." That sentence — abstract components resolved into 64 machines and 25% of a NIC — is NALSD in miniature. ✅

### 2.4 The State of the Art: NALSD as a Taught, Practiced Skill

NALSD is not a museum piece: Google SRE actively teaches it. The **SRE Classroom** workshop series (sre.google/classroom/) runs NALSD workshops with Google engineers, and Google's Cloud blog announced "Join SRE Classroom NALSD workshops" for the public. The two flagship workshop pages each state the same pair of goals (✅, extracted from the workshop pages this pass): "(1) introduce participants to the principles of non-abstract large systems design (NALSD), and (2) provide hands-on experiences with applying these principles to the design and evaluation of these systems." The distributed-pubsub workshop exercises a publish-subscribe system; the distributed-imageserver workshop exercises a content-serving system; and the imageserver page adds that Google considers "NALSD a concept fundamental to" the reliability discipline.

Two consequences for a candidate:

1. **The method is practiced against exercise systems, not just taught as theory.** Google's own workshops put participants in front of a concrete system to design and evaluate — the same muscle §7 of this guide trains by running the repo's large systems through the NALSD lens.
2. **The interview claim has a teaching tail.** Because the SRE Classroom program exists (2018 → present), the NALSD label demonstrably belongs to a living Google curriculum — which is why the prep ecosystem keeps treating it as the name of the SRE design round (⚠ third-party on the round-name specifics; see §6.2).

---
## 3. The Method: The Interview Structure

### 3.1 The Shape Shared by Every Framework

Every serious system-design interview framework on the shelf is the same six-beat structure in different clothing — and NALSD is the version with the reliability lens turned up:

| Beat | Grokking 7-step (Educative) | Xu 4-step (Insider's Guide) | Google rubric (repo Google guide) | NALSD-round practice (this guide's synthesis, §3.2–§3.7) |
|---|---|---|---|---|
| 1 | Requirements clarification | Step 1 — requirements | Requirements | Interrogate the reality: user goal, constraints, requirements as SLOs |
| 2 | Capacity estimation / back-of-envelope | Step 2 — estimation | Estimation | Feasibility math: bandwidth, IOPS, storage, QPS — *before drawing* |
| 3 | API design | — (folded into design) | — | Establish the contract: APIs, data model, SLIs/SLOs |
| 4 | Data model design | — (folded into design) | — | Data model with the numbers attached (row sizes, growth) |
| 5 | High-level design | Step 3 — high-level design | Components (architecture) | Architecture for failure: components + reliability mechanisms |
| 6 | Deep dive | Step 4 — deep dive | Deep-dive | The bottleneck deep-dive: one component, fully quantified |
| 7 | Wrap-up | — (tips) | Trade-offs | Trade-offs + wrap-up: what was chosen, what was refused, what is monitored |

*Source notes: the Grokking 7-step and Xu 4-step shapes are verified as the frameworks of those courses/books via the repo companions [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) and [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md); the Google column follows [google_system_design_interview_guide.md](google_system_design_interview_guide.md) §4. The "NALSD-round practice" column synthesizes the SRE Workbook Ch. 12 process (✅) with third-party SRE interview walkthroughs (⚠) — the ordering of beats 3–4 varies by source; the *content* of the beats is what is stable.*

### 3.2 Beat 1 — Interrogate the Reality (Requirements)

NALSD's distinctive first move is refusing to design on the first prompt. The Workbook's process "begin[s] with the problem statement, gather requirements, and iterate." In interview practice this means gathering, in order: the **user goal** (what a human or business is actually trying to do — not "write to the store"), the **fixed constraints** (budget, hardware estate, legacy code, regulatory deadlines, the speed of light), and the **non-functional requirements** stated as targets the design can be checked against.

What this beat must produce: a restated one-paragraph problem statement the interviewer agrees to, a list of functional requirements (what the system does), and 2–3 quantified non-functional targets — typically an availability number, a latency percentile, and a freshness/durability requirement. NALSD practice often states these as **SLOs up front** ("before we discuss architecture, I want to agree that our target is 99.95% availability and p99 latency under 200 ms" — third-party walkthrough phrasing, ⚠, but fully consistent with the Workbook's SLO-driven AdWords example, ✅).

### 3.3 Beat 2 — The Feasibility Math (Estimation Before Drawing)

The Workbook's scaling phase asks *"Is it feasible?"* — and its "Why Non-Abstract?" section explains that feasibility is a *numbers* question. Prep sources for the SRE round are explicit that the capacity math precedes the diagram: "if you draw a box on the whiteboard before doing the capacity math, you have already failed" (⚠ third-party handbook — strong phrasing, same thesis as the primary source). The interview version of this beat is the back-of-envelope pass: peak and average request rates, payload and log sizes, storage growth, bandwidth, and the machine count implied — the full discipline is §4. A NALSD-specific nuance: if the math refutes the requirement (a 5-petabyte synchronous replication over a 10 Gbps link cannot meet a 200 ms RPO; the transfer alone takes on the order of 46 days), the correct move is to *say so and renegotiate the requirement* — interviewers treat "this requirement is physically impossible" as a pass signal, not a failure (⚠ third-party; consistent with the Workbook's assumption-revealing philosophy, ✅).

What this beat must produce: 2–4 order-of-magnitude numbers (QPS, bytes/sec, GB stored, machines) with stated assumptions, and — critically — a sentence connecting each number to a design decision.

### 3.4 Beats 3–4 — The Contract: APIs and the Data Model

Before (or while) drawing components, the NALSD-style answer pins down the **contracts** — the interfaces the design must honor. This is the "concrete" half of non-abstract: an API signature and a data model are concrete in a way a box-and-arrow diagram is not.

- **APIs:** the operations the system exposes, with request/response shapes, idempotency keys where writes are involved, and the consistency each call promises. For the SRE variant, add the operational interfaces: health/status endpoints, metrics hooks, and the manual control surfaces (drain, kill-switch, failover) — a reliability-flavored design specifies how it is *operated*, not only how it is called.
- **Data model:** the entities, the key (and why that key — this is where sharding is born), row/record size estimates, retention, and growth rate. In the Workbook's example the data model decisions *are* the architecture: query log vs click log, keyed by `query_id`/`ad_id`, sized at ~2 KB per entry, joined and grouped to produce CTR.

What these beats must produce: named endpoints with semantics, entity sketches with keys and estimated sizes, and the retention/durability story. If a box in the later diagram has no data model behind it, the design is still abstract.

### 3.5 Beat 5 — The High-Level Architecture, Designed for Failure

The Workbook's phases frame this beat: first a **basic design** that works in principle ("is it possible?" — "can we build it without magic?"), then the **scaled design** ("is it feasible?"), and every component is then run through **"is it resilient?"** — what happens when the component fails, when a rack fails, when a datacenter fails. The SRE-leaning architecture answer therefore adds to the normal component diagram (clients → gateway → services → data stores, with cache/queue tiers where the numbers justify them):

- the **failure domains** (cells/regions) and how a bad deploy is isolated;
- the **reliability mechanisms** attached to each hop — bounded retries, exponential backoff, circuit breakers, timeouts;
- the **degradation story** — what the system deliberately stops doing under overload, in what order;
- the **observability surface** — the SLIs that will prove the SLOs (latency, error rate, saturation/queue depth) and the alerting that fires on burn rate rather than on symptoms.

What this beat must produce: a labeled diagram where every component carries its quantity (instance count or shard count and size), its failure mode, and its backpressure/degradation behavior. An architecture that cannot answer "what breaks first and what do we do?" for each of its own components is still abstract.

### 3.6 Beat 6 — The Deep Dive

Interviewers allocate the last third of the round to one bottleneck or risk, pushed until the candidate's knowledge runs out. Typical NALSD probes: the write path under peak (what actually happens per request, in order); the storage engine under the retention policy (compaction, IOPS, disk fill rate); the consistency behavior during a partition (what reads return, what writes buffer, what reconciles); the hot-shard problem (one celebrity account, one large-value payment instrument, one skewed key). The deep dive is where the candidate *stops covering breadth* and goes depth-first on the single component the interviewer names — with numbers (cross-ref the deep-dive discipline in the repo's component guides: [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) and [s3_architecture_guide.md](s3_architecture_guide.md) are worked examples of depth-first component design).

### 3.7 Beat 7 — Trade-offs and the Wrap-Up

The final beat closes the loop the candidate opened in beat 1: restate the requirements, state the 2–3 main trade-offs the design embodies (and what was *refused* — a design with no refused alternatives is a design with no reasoning), state the residual risks and the monitoring that would catch them, and stop. The trade-off axes are §5; the wrap-up is what most candidates omit and interviewers score (cross-ref the failure mode in §8.8).

### 3.8 The Four NALSD Questions as a Rehearsal Loop

The Workbook concludes by compressing the process into four questions (✅, direct from Ch. 12). A candidate can rehearse any design — including the repo's worked systems in §7 — by running it four times:

1. **Is it possible?** — Can we build it without "magic"?
2. **Can we do better?** — Is it as simple as we can reasonably make it?
3. **Is it feasible?** — Does it fit within our practical constraints (budget, time, hardware)?
4. **Is it resilient?** — Will it survive occasional but inevitable disruptions?

"While we generally cover these phases and questions in this approximate order, in practice, we bounce around between the questions and phases" — the Workbook's own description of how the interview actually feels: not a linear recitation, but an iteration that the candidate visibly drives.

### 3.9 The Time Budget and the Interviewer's Probes

Prep sources converge on roughly 45 minutes for the design round (⚠ — the *allocation* below is prep-ecosystem convention, not a published rubric). A workable budget that keeps every beat alive:

| Beat | Share of the round | What the interviewer is listening for | Typical probe when the candidate stalls |
|---|---|---|---|
| 1. Requirements (§3.2) | ~10 min | A restated problem + quantified targets before any design | "Who is this for, and what does success look like in numbers?" |
| 2. Feasibility math (§3.3) | ~5 min | An audible assumption → arithmetic → consequence chain | "How many queries per second is that, and what does each cost?" |
| 3–4. Contract: APIs + data model (§3.4) | ~10 min | Concrete interfaces, keys, sizes, retention | "What is the write key, and what breaks if two of those arrive at once?" |
| 5. Architecture (§3.5) | ~10 min | Failure domains and quantities on every box | "That component dies — what happens, and how do we know?" |
| 6. Deep dive (§3.6) | ~5–7 min | Depth on the named bottleneck, with numbers | "Go deeper on the write path — what is the per-request sequence?" |
| 7. Trade-offs + wrap-up (§3.7) | ~3–5 min | Refused alternatives, residual risks, monitoring | "What did you decide *not* to do, and why?" |

The two probes every NALSD candidate should be ready for verbatim, because they are the method's two pillars in question form: **"how many machines is that?"** (the concreteness pillar — §4) and **"what happens when it fails?"** (the resilience pillar — §3.5). If a rehearsal answer cannot survive both, it is still abstract.

---
## 4. The Estimation Discipline

*Condensed for the interview. The full discipline — Little's Law, utilization targets, per-dimension sizing, cloud rightsizing — lives in [capacity_sizing_guide.md](capacity_sizing_guide.md) (its §8 works a Cymbal Bank payments-service sizing with the actual math); the interview-scale-math treatment lives in [google_system_design_interview_guide.md](google_system_design_interview_guide.md) §5. This section is the back-of-envelope core NALSD demands: traffic → QPS, payload → bandwidth, records → storage, workload → machines.*

### 4.1 Why NALSD Is an Estimation Discipline

Every NALSD source converges on the same point: the numbers are not decoration, they are the design driver. The Workbook's "Why Non-Abstract?" section says the whole method exists to force "concrete estimates of resources at multiple steps," and its calibration note says the *reasoning* matters more than the final values — so the interview is scored on the chain (assumption → arithmetic → design consequence), not on hitting a canonical answer. ✅ Kirsch's SRECon18 abstract confirms the interview measures "estimating provisioning needs." A candidate who cannot produce a QPS number within the first ten minutes cannot demonstrate NALSD, whatever the diagram later looks like.

### 4.2 The Four Back-of-Envelope Equations a Candidate Must Own

| Quantity | Formula | Units | Example (from the Workbook's AdWords pass, ✅) |
|---|---|---|---|
| Average throughput | QPS = (users × actions per user per day) ÷ 86,400 | requests/sec | — |
| Peak throughput | peak ≈ 3–10 × average (5 × is the common interview default) | requests/sec | 500,000 searches/sec assumed; 10,000 clicks/sec |
| Bandwidth (serving) | QPS × payload bytes | bytes/sec | query log at ~2 KB per entry: 1.02 × 10⁶ entries/sec × 2 × 10³ B ≈ 2.04 GB/s ≈ 16 Gbps (Workbook's arithmetic) |
| Storage | records/sec × bytes × retention seconds × replication | bytes | full-day logs at the rates above → ~4 TB of RAM for the in-memory maps per datacenter (Workbook's result) |
| Machines | workload ÷ per-machine capacity (with headroom) | machines | 4 TB ÷ 64 GB per machine ≈ 64 machines at 25% network utilization (Workbook's result) |

**The numbers to carry (prep canon, ⚠ rule-of-thumb values — the repo's capacity guide and Google guide §5 list the same constants as working defaults, not physics):** seconds per day = 86,400; a binary KB/MB/GB (2¹⁰, 2²⁰, 2³⁰); one server ≈ 64–128 GB RAM and 1–10 Gbps NIC; one SSD ≈ 100–500 MB/s sequential (the 100 MB/s-per-disk figure is the one prep glossaries use for the arithmetic); a modest DB server ≈ low thousands of QPS for complex queries, tens of thousands for point lookups behind cache; 1 million requests/day ≈ ~12 avg QPS. The point of carrying constants is *order-of-magnitude closure*, not precision — say the assumption, compute, move on.

### 4.3 The Worked Canonical Pattern: Log Ingestion

The prep-ecosystem's favorite NALSD demonstration (⚠ third-party example, tieroneprep's NALSD glossary; the arithmetic is the point, not the source) runs exactly the §4.2 chain: **1M hosts × 10 KB/s of logs = 10 GB/s ingest.** At ~100 MB/s per disk that is at least 100 disks of sequential write throughput *before* replication; at 3× replication, ~300 disks' worth — so the naive "agents write to their local disk" design dies on the math, and the numbers — not intuition — dictate an aggregation tier and per-shard batching. This is the pattern to internalize: **rate × size = throughput; throughput ÷ per-device capacity = count; count × replication/headroom = the real requirement.**

### 4.4 The NALSD Estimation Attitude: Three Moves Interviewers Score

1. **Estimate before drawing.** The feasibility math is beat 2 of the structure (§3.3), *before* the architecture diagram — because an architecture drawn before the math has no reason to be the right shape.
2. **Follow the chain aloud.** "If we assume X users doing Y per day, that is Z QPS average, call it 5Z at peak; each record is about W bytes, so the write path must absorb …" — the audible chain is what makes reasoning visible and scoreable; silent arithmetic scores nothing.
3. **Let the number change the design.** The pass signal is the sentence "that number forces a different choice here" — the aggregation tier in the log example, the sharded in-memory maps in the AdWords example. When the numbers *confirm* a design choice instead of reshaping it, say that too; the candidate who can report "the math closed, no redesign needed" has demonstrated the discipline's purpose.

**Failure version of the same beat:** quoting "we'll use Kafka and Redis" with no throughput, no partition count, no retention size; or producing numbers and then ignoring them in the architecture. Both are §8 failure modes.

### 4.5 The Banking Calibration (cross-ref, condensed)

For the "design in my domain" variant the numbers change shape but not method: payment-rail peaks are *peak* problems — the repo's capacity guide cross-refs verified rail figures (Visa ~8,500 tps average with peaks above 65,000 tps; the NETS Gateway's per-instance throughput floor; the FAST 24/7 rail) and the batch-window reality of a core banking day (cross-ref [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) §7 for the EOD/batch process, [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) for the nines math, and the §9 worked example below for the full chain at retail-bank scale). The lesson a NALSD candidate should draw: a retail bank's *retail* transaction path is honestly a few hundred QPS — the interesting numbers are the batch windows, the end-of-day peaks, and the regulatory retention, not a billion-user DAU.

### 4.6 The Constants Card and the 40-Second Feasibility Drill

**The constants card (⚠ rule-of-thumb values — the working defaults of the prep canon, not physics; say them as assumptions):**

| Constant | Value to carry | Used for |
|---|---|---|
| Seconds per day | 86,400 | QPS from daily volumes |
| Binary units | KB = 2¹⁰, MB = 2²⁰, GB = 2³⁰ | storage/bandwidth arithmetic |
| 1M requests/day | ≈ 12 average QPS | sanity-checking traffic claims |
| Peak vs average | 3–10 × (5 × is the common default) | sizing for the peak |
| Server RAM | 64–128 GB | machine counts from memory-bound workloads |
| NIC | 1–10 Gbps | bandwidth feasibility |
| SSD sequential write | ~100–500 MB/s per disk | disk counts for ingest (the 100 MB/s figure is what the prep glossaries use) |
| DB node | ~1k–10k QPS depending on query shape | whether one node closes the math |

**The 40-second drill** (practice until it is reflex — this is the whole of beat 2 compressed): hear the prompt → state one assumption → compute one rate → compute one size → say which design consequence follows. A worked 40-second pass, aloud:

> "Assume 2 million daily active users each performing 5 read actions a day — that is 10M reads a day, about 115 average reads per second, call it 600 at a 5× peak. If a read serves 200 KB, peak serving bandwidth is 600 × 200 KB ≈ 120 MB/s — a single 1 Gbps NIC handles that with headroom, so bandwidth is not the constraint; the database read load is, and that is where I would put the cache. Cache or not is therefore a *read-QPS* decision, and at 600 peak QPS a single read replica closes the math before any distributed cache earns its place."

That is the NALSD cadence: number, consequence, next number. The drill's failure version is the candidate who says "we'll cache it" with no read QPS on the table — §8.2.

---
## 5. The Trade-off Analysis

*Condensed map of the recurring axes — cross-ref, not re-derived. The theory lives in [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (consistency models, partitioning, replication); the interview-shaped versions in the Grokking and Xu companions; the reliability targets in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md). NALSD's contribution is the *stance*: every trade-off is argued with the numbers from §4 attached, and every choice names its refused alternative.*

### 5.1 The Axes, in One Table

| Axis | The trade-off | NALSD way to argue it | Typical resolution pattern |
|---|---|---|---|
| Consistency vs availability | Strong consistency costs availability and latency during partitions (CAP); PACELC adds the latency/consistency choice even when there is no partition | Name the SLO that decides: what staleness does the requirement tolerate, in seconds? | Read-your-writes or strong for the money/ledger path; eventual for analytics/feeds; quorum (R + W > N) where both matter |
| Read-heavy vs write-heavy | Same logical feature wants different storage, cache and index shapes | Ratio reads:writes from the §4 traffic math | Read-heavy: cache tiers, read replicas, denormalized projections. Write-heavy: append-only logs, batching, partitioning by write key, async fan-out |
| SQL vs NoSQL | Transactions and joins vs scale-out and flexible schema | What does the data model require (multi-record atomicity, ad-hoc query) and at what QPS? | Ledger/money: transactional store, sharded. Feeds/counters/telemetry: NoSQL/wide-column |
| Push vs pull (delivery) | Push is low-latency but needs connection state and backpressure; pull is simpler and replayable but adds latency/polling cost | Latency SLO vs consumer count vs replay needs | Push with bounded queues + retries for real-time; pull/poll for batch and for slow consumers; an event backbone (cross-ref the Kafka material in the repo) separates both from producers |
| Cache tier placement | Cache-aside vs read-through vs write-through; invalidation complexity vs hit-rate | Which reads dominate, what staleness is legal, what is the invalidation trigger | Cache-aside for hot reads with TTL + explicit invalidation on write; keep the cache a *speedup* — the source of truth must stand alone (cross-ref [s3_architecture_guide.md](s3_architecture_guide.md) for the object-store/cache pattern) |
| Sync vs async processing | Synchronous = simple, consistent, latency-coupled; async = resilient, decoupled, but eventually consistent and harder to debug | Does the user wait on the result? What is the freshness SLO? | User-facing result: sync with timeout/retry. Everything else: async with an outbox/queue and idempotent consumers |
| Batch vs stream | Batch is simple, cheap, replayable, late; stream is fresh, complex, stateful | Freshness SLO (the Workbook's "data less than 5 minutes old" is a streaming SLO; a daily CTR report is a batch SLO) | Batch for day-end/regulatory; stream for the real-time path; often both over the same store |
| Shard key choice | A key that distributes load well vs a key that keeps related records together | Enumerate the hot keys the §4 math exposes (one account, one ad, one celebrity) | Composite/derived keys, or accept the hot shard and design for it |
| Over- vs under-engineering | Every added tier (queue, cache, microservice split) buys scalability at the cost of operational surface | Only add a component when the §4 numbers *require* it — the Workbook's "can we do better / is it as simple as we can reasonably make it" | Start with the simplest design that closes the numbers; add tiers one at a time, each justified |

### 5.2 The NALSD Stance on Trade-offs

Three habits separate a NALSD trade-off discussion from a generic one:

1. **Anchor every axis in the SLOs and the numbers from §4.** "Eventual consistency is fine here because the dashboard SLO allows five-minute-old data" (the Workbook's own AdWords framing) beats "we'll use eventual consistency because it scales."
2. **Name the refused alternative.** "I am choosing a single-writer shard per account over a distributed transaction because the cross-account rate is under 1% of traffic and the complexity is not justified" — the refusal is the reasoning.
3. **Include the operational trade-offs, not just the data ones.** Reliability-flavored design weighs deployability, debuggability, and failure blast radius: canary vs big-bang, single cell vs global, push config vs pull config. These are the axes interviewers probing "managing change" (Kirsch's abstract, ✅) listen for — cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) and the middleware estate's change-management material in [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md).

### 5.3 The Two Trade-offs a Banking Candidate Must Own

For the banking domain variant, two axes dominate every design conversation and should be rehearsed to reflex:

- **Ledger integrity vs distributed convenience.** The posting path is the one place where double-entry atomicity (DR = CR, cross-ref [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) §1.1 and §8) is non-negotiable — which constrains sharding (an account's balance lives in one transactional shard), idempotency (a retried payment must not post twice), and replication (the money path wants strong consistency or a rigorous reconciliation story). Trade-offs argued here must respect that constraint first and optimize around it.
- **Real-time rail vs batch core.** The instant-payment world (FAST-style 24/7 rails) demands a real-time accounting path, while the core banking estate still closes a day (cross-ref [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) §7). The design question is where the line sits: what posts intraday in real time, what batches, and how the two reconcile. §9's worked example designs exactly this boundary.

### 5.4 The Three-Sentence Trade-off Template

Interviewers do not need an essay per axis — they need the shape of a decision. The reusable template, three sentences:

1. **The choice:** "On the X axis I am choosing A over B."
2. **The reason, with the number:** "Because the §4 math / the SLO says … (one quantified driver)."
3. **The refusal and the cost:** "I am consciously giving up …, and here is what will tell us the choice was wrong (the monitor)."

Worked micro-example (caching, from the §4.6 drill): *(1) On the read path I am choosing a read replica over a distributed cache tier. (2) Because the peak read rate is ~600 QPS and one replica closes that math with headroom. (3) I am giving up the sub-millisecond hit latency a cache would add, and the p99-serving monitor will tell us if the replica stops closing the number.* The interviewer hears: an axis named, a number attached, a consequence owned — which is the entire NALSD trade-off skill in under thirty seconds. The failure version is the candidate who lists technologies ("we have Redis, Kafka, and a CDN") without a single refusal — §8.4 and §8.11.

---
## 6. NALSD in the Interview: The Formats

### 6.1 The Verified Core: Google SRE

The only format claims with primary-source backing are the Google SRE ones:

- **One NALSD interview per candidate.** "All candidates for SRE positions at Google participate in one NALSD interview as part of their recruiting process" — Kirsch, SRECon18 Asia, 2018 (✅, usenix.org / research.google/pubs).
- **What it measures.** "Candidates demonstrate their skills in designing for scalability, reliability and robustness, estimating provisioning needs, and managing change" — same source (✅).
- **Why Google built it.** The same talk's framing: going beyond coding and algorithm skills to a "credible approach for solving a specific problem at large scale" (✅).
- **The methodology is still current.** SRE Classroom NALSD workshops and sre.google resources using NALSD are live in 2026 (✅), and the interview-prep ecosystem continues to treat the SRE design round as the NALSD round (⚠ third-party but consistent).

### 6.2 The Round's Shape per Interview-Prep Sources (⚠ third-party)

Prep material aimed at Google SRE loops (AceInterviews' google-sre-interview-handbook and walkthroughs; tieroneprep's NALSD guide; video walkthroughs) consistently describes — and the repo can only partially confirm — the following specifics, all marked ⚠ as single- or few-source third-party claims not pinned to a Google primary document this pass:

- **The prompt is often an existing production system**, not a greenfield product: "already in production, handling massive traffic, currently broken, hitting a physical limit, or requiring a zero-downtime migration." The candidate is evaluated on reasoning "like a Production SRE," including incident command when the prompt is a failing system (stabilize user impact *before* root-cause hunting).
- **A 45-minute verbal sprint** with the interviewer injecting constraints and failures ("your fallback database just ran out of inodes — now what?").
- **The "physics first" rule:** feasibility math (bandwidth, IOPS, storage, transfer time over real links) precedes any diagram, and declaring a requirement physically impossible is a pass signal rather than a failure.
- **Leveled expectations** in the L4/L5/L6 sense: junior answers add components; senior answers add failure domains, SLOs, and operational control surfaces.

These specifics align with the primary sources' *spirit* (the Workbook's feasibility/resilience questions, Kirsch's "estimating provisioning needs and managing change") but the specific format details — existing-system prompts, the 45-minute length, the level framing — could not be verified against a Google primary source this pass. Treat them as the prep ecosystem's best reconstruction, not confirmed fact (see §11.5).

### 6.3 The Generic FAANG Landscape (⚠ where unverifiable)

Where NALSD-style rounds otherwise appear, per aggregated prep content and the repo's own Google guide (whose claims about loop composition are themselves based on aggregated reports, not Google publications):

| Company/role | Design-round reality | NALSD relevance | Status |
|---|---|---|---|
| Google SWE | 45–60 min design round(s) at L5+; product-scale prompts ("design YouTube") | Same clarify-estimate-design-trade-off shape; the *reliability weighting* of NALSD is lighter than in SRE loops | ⚠ (aggregated reports; see the repo Google guide §1) |
| Google SRE | The NALSD round (one per loop, 2018 primary) | Direct | ✅ (2018) / ⚠ (current naming) |
| Amazon | Architecture/design rounds in the loop plus the bar-raiser interviewer; leadership-principle weighting | Same concrete-numbers style in practice, but no source found this pass calling Amazon's rounds NALSD; "bar raiser" is a role in the loop, not a design-round name | ⚠ (format) / ⚠-negative (label) |
| Meta / Microsoft / Netflix | Design rounds at senior levels (product/infra flavored) | Same style, no NALSD label found | ⚠-negative |
| Banks / fintech (the Cymbal Bank context) | "Design in my domain" rounds — payments, posting, middleware | The NALSD method transfers directly; the numbers change (see §4.5) | repo-internal |

### 6.4 What Interviewers Score (synthesis)

Across the primary and third-party sources, the NALSD-flavored round is scored on five observable behaviors, in rough order of weight:

1. **Requirements discipline** — clarifying, restating, and SLO-izing before designing (scored in every source).
2. **Provisioning skill** — real estimation math with stated assumptions, letting numbers drive choices (Kirsch ✅; Workbook ✅).
3. **Reliability reasoning** — failure modes per component, degradation order, resilience to datacenter/zone loss (Workbook's "is it resilient?" ✅).
4. **Change management** — how the design is operated, rolled out, and reverted; canaries, drain, kill-switches (Kirsch's "managing change" ✅).
5. **Communication** — audible chains of reasoning, refused alternatives, and a wrap-up (prep-canon, ⚠ qualitative).

**Time-boxing.** Interview-prep sources converge on 45 minutes for the round with the rough allocation: ~10 minutes requirements/numbers, ~20 minutes architecture, ~10 minutes deep dive, ~5 minutes trade-offs/wrap-up — the same budget the repo's Google guide gives its 5-step framework. ⚠ The *allocation* is prep-ecosystem convention; no primary source publishes a minute-by-minute rubric.

### 6.5 The Scorecard Behind the Round (synthesis)

Interviewers write feedback against signals, not against a checklist — but across the primary sources' stated concerns (Kirsch: scalability, reliability, robustness, provisioning, change — ✅) and the prep ecosystem's reconstructions (⚠), the scorecard that emerges has six rows a candidate can self-grade after every mock:

| Signal | What the interviewer hears | Failing version | Passing version |
|---|---|---|---|
| Scope control | The problem is narrowed to something designable in 45 minutes | Boiling the ocean: six features, three regions, every edge case | "I am scoping to the single payment path first; the rest is future work" |
| Quantification | Real numbers with stated assumptions | "High scale", "a lot of traffic" | "~46/sec average, ~370/sec peak, assuming 2M customers × 2 payments/day" |
| SLO anchoring | Targets set before architecture | No targets, or targets invented mid-design | "99.95% availability, p99 under 1 second, exactly-once posting" |
| Physics respect | Constraints checked against real limits | Magic boxes; unbounded assumptions | "At 1.5 KB × 800/sec that is ~1.2 MB/s — bandwidth is not the constraint" |
| Failure reasoning | Every component has a failure mode and a response | "It's highly available" with no mechanism | "The store is the single point of truth, so it gets the sync replica and the recovery drill" |
| Change management | The design is operable: rollout, drain, revert | Design ends at "deployed" | "We canary the new posting rules; the kill switch drains the payment path" |

A mock that scores all six rows has demonstrated the method; a mock that scores three has a named gap to drill. The §8 failure table is this scorecard's negative image.

---
## 7. The Practice Patterns: Walkthroughs via the Repo's Large Systems

*Practice corpus. Each repo large-system guide below is read through the NALSD lens — the requirements you would gather, the 2–3 numbers that anchor it, the deep-dive an interviewer would probe — as a condensed cross-ref. Full architectures live in the named guides; nothing here re-derives them.*

### 7.1 The Trading System ([trading_system_software_architecture_guide.md](trading_system_software_architecture_guide.md))

A NALSD prompt in this domain would be "design the order path for an execution desk" or "the venue feed is late — what breaks?" The repo guide is the canonical stack: order lifecycle and FIX tags (§2), OMS/EMS split (§3), pre-trade risk and the kill switch (§4), market-data path (§5), execution architecture — algos and smart order routing (§6), venue connectivity and colocation (§7), the low-latency engineering layer (§9), and surveillance/reconstructability (§10).

- **Requirements you would gather:** which order types and instruments; the regulatory overlay (pre-trade risk checks, market-access rules — §4); whether the ask is latency, throughput, or auditability; the human goal (a trader's algo intent reaching the matching engine and the truth returning).
- **The 2–3 anchoring numbers:** the **message rate** on the order path (orders are sparse; market data is not — the feed rate is the bandwidth driver); the **latency budget** in microseconds for the hot path (colo, kernel bypass — §9); the **retention/reconstructability** requirement for surveillance (§10). Cross-ref the repo's market-data and FIX material for the actual protocol rates rather than inventing them here.
- **The deep-dive an interviewer would probe:** the single hot path — one order from risk check to venue — traced instruction by instruction, with the pre-trade risk throttle placed *before* the network egress (the kill switch, §4), and the failure question "the venue is down mid-order — what happens to the order state machine?" The NALSD lesson of this guide: the design is a *state machine boxed by a latency budget and risk controls* — every component earns its place by the microsecond it costs or saves.

### 7.2 The Middleware Integration Platform ([enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md))

A NALSD prompt: "the bank's event backbone is saturating at month-end" or "rationalize the MQ/TIBCO/webMethods estate onto the modern fabric." The repo guide is the integration-fabric map: middleware history (§2), the integration taxonomy (§3), the platform layer on Kubernetes (§5), the banking angle — payments, core interfaces, SWIFT, the trading floor (§6), and its own Cymbal Bank rationalization worked example (§7).

- **Requirements you would gather:** which integration *categories* must be served (request/reply, reliable messaging, event streaming, file, API — §3); the coexistence constraint (the mainframe boundary — §5); the migration constraint (no big-bang cutover of a live estate — §7's phased migration).
- **The 2–3 anchoring numbers:** **message volumes and sizes** per integration category (the event backbone's throughput and retention — cross-ref the repo's Kafka topic catalogue with its partition counts and tiered retention in the banking Kafka guide); the **peak-vs-average** shape (month-end, batch windows); the **consumer fan-out** (how many downstream systems read each event — the difference between a queue and a topic).
- **The deep-dive an interviewer would probe:** the delivery-semantics question on one critical link — exactly-once-ish delivery for payments events (transactional outbox, idempotent consumers — cross-ref the repo's MQ-reliability and outbox material), and the failure question "the broker loses a rack — what replays, what is ordered, what reconciles?" The NALSD lesson: an integration fabric is chosen *per category* with its numbers attached, never as one product for everything.

### 7.3 The Posting Engine ([../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) + [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md))

A NALSD prompt: "design the posting path for real-time payments" — the fullest banking-domain practice case, and the direct ancestor of §9's worked example. The posting guide covers the double-entry foundations (§1–§2), the posting lifecycle — initiate → validate → authorize → post → confirm (§3), the engine architecture — posting rules, balance engine, entry engine, GL integration (§4–§6), and the architect's view of integrity and performance (§8); the processes guide supplies the surrounding business processes, especially transaction processing (§6) and the EOD/batch process (§7).

- **Requirements you would gather:** the accounting invariants (DR = CR enforced atomically — §1.1); which postings are real-time vs memo vs batch; the audit/regulatory retention; the reversals and suspense requirements (§3, §6); the 24/7 rail constraint if the prompt is instant payments.
- **The 2–3 anchoring numbers:** **transaction rate** at the rail (cross-ref the verified rail figures in the capacity guide — average vs peak matters more than the average itself); **balance-read amplification** (every posting reads and writes the account balance plus history — the actual DB load is several operations per posting); **storage growth** of the posting/entry history under regulatory retention (cross-ref §4.5 and the §9 estimation table).
- **The deep-dive an interviewer would probe:** atomicity of the multi-entry posting — how DR = CR is guaranteed across accounts (single transactional shard per account; two-phase concerns; the *wrong-but-balanced* vs *unbalanced* failure distinction of §1.1), plus idempotency under client retries. The NALSD lesson: the ledger's integrity constraint is the *requirement the architecture must not trade away* — every other trade-off (§5.3) orbits it.

### 7.4 The Legacy-Integration Variant ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md))

The "design in my domain" prompt that starts from *what exists* rather than greenfield: integration patterns and styles for estates that cannot be replaced. Its NALSD reading: the requirements phase *is* the estate discovery (what contracts exist, what the failure domains are); the anchoring numbers are the existing system's throughput, batch windows, and interface counts; the deep-dive is one strangler-fig or anti-corruption boundary fully specified. Cross-ref lightly — it is the pattern vocabulary the middleware guide operationalizes.

### 7.5 Running These Walkthroughs as Mocks

Each §7 system becomes a mock round in four steps (the rehearsal protocol of §1.6, made concrete):

1. **Turn the guide's subject into a prompt.** "Design the order path for an execution desk" (7.1); "the event backbone is saturating at month-end — what do you do?" (7.2); "design the posting path for real-time payments" (7.3); "the core estate must expose a real-time balance API over a batch core" (7.4, the legacy variant).
2. **Deliver the six beats aloud under a 45-minute timer** (§3.2–§3.7), with the guide open only for the *requirements* phase — the numbers and architecture must come from your own §4 arithmetic, not from reading the guide back.
3. **Have the timer inject one constraint at minute 25** — a zone loss, a doubled peak, a failed batch window — and absorb it with the four questions (§3.8).
4. **Self-score against §8 and §6.5** — name the three failure rows you most resembled and re-run the same prompt next session until they clear.

The point of using the repo's own guides as the corpus: they are the *verified* architectures, so the mock's ground truth is checkable afterwards — a rehearsal on a random blog design cannot tell you whether your answer was right; a rehearsal on [trading_system_software_architecture_guide.md](trading_system_software_architecture_guide.md) can.

---
## 8. The Common Failure Modes

*Each failure mode below is reported by prep sources (⚠ third-party where noted) or follows directly from the primary sources' emphasis (✅ where it is the Workbook's or Kirsch's stated concern). Each gets its NALSD remedy.*

| # | Failure mode | What it looks like | Why it fails in NALSD | The NALSD remedy |
|---|---|---|---|---|
| 8.1 | Jumping to architecture | The candidate draws boxes in the first two minutes of "design X" | Skips the requirements and SLOs the design must be checked against (Workbook process, ✅); in the SRE round, "if you draw a box before the capacity math, you have already failed" (⚠) | Spend the first ~10 minutes on beat 1–2 (§3.2–3.3): user goal, constraints, quantified targets, feasibility math — *then* draw |
| 8.2 | No numbers | "It scales horizontally," "we'll add caching," "millions of users" — with zero arithmetic | The interview exists to measure "estimating provisioning needs" (Kirsch, ✅); abstraction is the named enemy of the method | Produce the §4.2 chain aloud before each major component choice: rate × size → throughput → machines |
| 8.3 | Ignoring physics | 5 PB synchronous replication over a WAN; infinite RAM assumptions; unbounded log retention | The "Why Non-Abstract?" thesis: systems run on real computers, real datacenters, real networks (✅); "physics vs magic" is the SRE round's fastest fail (⚠) | Check every data movement against real link/disk math; when the math fails the requirement, renegotiate the requirement aloud — a pass signal |
| 8.4 | No trade-off discussion | A single confident architecture, no alternatives refused | The design cannot be evaluated without its reasoning; the four questions include "can we do better / is it feasible" (✅) | For every major choice, state the alternative and the reason for refusal (§5.2); close with the 2–3 embodied trade-offs |
| 8.5 | Over-engineering | Kafka + Redis + microservices + a feature store for a system the numbers say needs two services | Violates "is it as simple as we can reasonably make it?" — the Workbook's own gloss on "can we do better" (✅); every added tier is added operational surface | Add components one at a time, each *required* by a number from §4; start at the simplest design that closes the math |
| 8.6 | No reliability/ops story | The design ends at "deployed"; nothing about failure, SLOs, monitoring | NALSD combines "capacity planning, component isolation, and graceful degradation" (✅); SRE answers must cover failure modes and observability | Attach to each component: failure mode, backpressure/degradation order, and the SLI that proves it healthy (§3.5) |
| 8.7 | Greenfield reflexes on an existing-system prompt | Treating a broken/live system prompt as "design me Twitter" | Prep sources say NALSD prompts often start from a *production* system that is broken or at a limit (⚠); the evaluation is operational reasoning, not product ideation | Clarify the system's current state first: what is in production, what is failing, what the blast radius is; if it is failing, stabilize before root-causing (§6.2) |
| 8.8 | No wrap-up | The candidate trails off when the interviewer stops asking | The final beat restates requirements → trade-offs → residual risks → monitoring; its absence leaves the interviewer to assemble the case | Close explicitly in the last ~5 minutes: "requirements were …, the design trades …, the residual risks are …, and we would monitor …" (§3.7) |
| 8.9 | Silent solving | Long pauses while the candidate thinks privately | Interview scoring rewards the *visible* chain: assumption → arithmetic → design consequence; silence scores nothing | Think aloud continuously; state assumptions even when unsure — "reasoning and assumption making are more important than any final values" (Workbook, ✅) |
| 8.10 | Memorized answers | Reciting a canonical "design X" from a course, unresponsive to injected constraints | The round injects scale changes, failures, and cost constraints; a memorized script cannot absorb them; prep sources explicitly warn NALSD is not generic whiteboarding (⚠) | Rehearse the *method* (four questions, §3.8) against many systems rather than memorizing one answer; let each prompt's numbers drive a fresh design |
| 8.11 | Treating the round as a trivia quiz | Dropping component names (Spanner, Kafka, Bigtable) with no sizing or trade-off | Names without numbers and failure reasoning are abstract design wearing a disguise | Every named technology carries: why here, how much, what breaks, what it cost to choose it (§5.1 rows) |
| 8.12 | Requirements capture without restatement | Asking questions, then designing against the candidate's private interpretation | The contract must be *agreed*: the Workbook begins from the problem statement and iterates against requirements (✅); a misread requirement invalidates the whole design | Restate the problem and the 2–3 quantified targets and get the interviewer's nod before drawing (§3.2) |

**The meta-pattern.** Most of these are one underlying failure: **answering with an abstract design when the round is scoring the non-abstract pass.** Run any practice answer through the four NALSD questions (§3.8) and most of the table above self-corrects — possible/better/feasible/resilient each kill a cluster of failure modes.

### 8.13 The Pass Version: One Prompt, Two Openings

Side-by-side, the same prompt — *"design a log-ingestion pipeline for 1M hosts"* (the §4.3 example) — opened two ways:

| | The failing opening (abstract) | The passing opening (non-abstract) |
|---|---|---|
| First words | "Okay, so we have agents on each host sending to Kafka, then a streaming job…" | "Before the architecture: what are we ingesting, at what rate, and how long must we keep it?" |
| Second move | Draws boxes (agents → queue → workers → store) | States assumptions: "1M hosts × 10 KB/s ≈ 10 GB/s continuous ingest; at ~100 MB/s per disk that is ~100 disks of sequential write before replication" |
| Third move | Adds more boxes (a second queue "for resilience") | Draws the consequence: "3× replication makes it ~300 disks of write capacity — so the naive design dies on disk count; the numbers dictate an aggregation tier and per-shard batching, which changes the diagram I am about to draw" |
| When probed on failure | "The queue makes it reliable" | "The aggregation tier is the first thing to saturate, so it gets the backpressure signal; the store is sized from the disk math, and if a zone dies we replay from the raw log" |
| Result | Architecture-first, number-free, one un-refusable box per minute | Numbers-first, every box justified by arithmetic, failure designed for |

The two openings are the same forty-five minutes apart in quality. The passing opening is not smarter — it is *disciplined*: it refuses to draw until the math has spoken, exactly as the Workbook's process and the SRE round's "physics before boxes" rule (⚠ third-party) demand.

---
## 9. The Cymbal Bank Worked Example

> **Design fiction notice (house convention):** Cymbal Bank is the repo's fictional Singapore-based retail bank persona. This worked example is an **illustrative design exercise** — the requirements, scale assumptions, and numbers below are stated assumptions chosen for the exercise, *not* claims about a real bank. The *method* is the verified NALSD content of §2–§5; the *domain facts* (double-entry posting, FAST-style rails, MAS-style retention expectations) cross-reference the repo's verified banking guides. All arithmetic is order-of-magnitude and shown so it can be checked.

**The prompt (as an interviewer would give it):** *"Design the real-time payments and accounting service for Cymbal Bank: customers send instant domestic credit transfers 24/7; the money must move, the books must stay balanced, and the service must survive the failure of a database, a zone, or a payment rail."*

### 9.1 The Requirements Gathered (beat 1 — interrogate the reality)

- **User goal:** a Cymbal Bank customer instructs a transfer and both parties need certainty — the sender wants the money debited exactly once and the recipient credited promptly; the bank needs the double-entry books to stay in balance (DR = CR) for every payment, always.
- **Functional requirements:** initiate payment (single/instant), check sufficient funds, post debits and credits atomically, notify both parties, support cancellation in a short window, produce an audit trail, and reconcile with the external rail and the general ledger.
- **Non-functional requirements, stated as the contract:** availability **99.95%** on the payment path (≈ 4.4 hours downtime/year — cross-ref the nines math in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md)); p99 end-to-end **< 1 second** for a domestic instant payment; **no double-posting ever** (idempotent under retries — the one invariant that outranks availability); regulatory retention of posting records; MAS-style supervisory reporting feeds.
- **Assumptions (stated aloud, per NALSD practice):** ~2M active retail customers; average ~2 outbound instant payments per customer per day → **~4M payments/day ≈ 46 avg payments/sec**; inbound credits from the rail add roughly the same volume again; **peak ≈ 8× average** at campaign/month-end moments; average payment payload ≈ 1.5 KB with metadata and audit.

### 9.2 The Estimation Table (beat 2 — feasibility math before drawing)

| Quantity | Arithmetic (all order-of-magnitude, fictional) | Result |
|---|---|---|
| Average payment rate | 4M ÷ 86,400 sec/day | ≈ 46 payments/sec |
| Peak payment rate | 8 × average | ≈ 370 payments/sec |
| Combined rail + API peak | outbound 370 + inbound ~370 | ≈ 700–800 msg/sec peak |
| DB write amplification | each payment = payment row + ≥2 entries + balance updates + outbox ≈ 6–8 writes | ≈ 250–450 writes/sec avg; ≈ 3,000/sec peak |
| Balance read amplification | funds check + postings read balances per payment (with cache, mostly point reads) | ≈ 2–4 × payment rate |
| Storage | 4M payments/day × ~1.5 KB ≈ 6 GB/day raw; ×3 for indexes/replication ≈ 18 GB/day → ~6.6 TB/year | 7-year retention ≈ **~46 TB** (archive tier; cross-ref [capacity_sizing_guide.md](capacity_sizing_guide.md) for the retention/archive discipline) |
| Bandwidth | 800 msg/sec × 1.5 KB ≈ 1.2 MB/s in, plus downstream events | trivially small — not the constraint |
| Machines (initial) | the write path fits one active-active pair of transactional DB clusters with read replicas; 4–6 stateless payment-service instances | single-digit instances **— the numbers reject over-engineering** |

**The NALSD turn:** at Cymbal Bank's honest retail scale the numbers close on a deliberately *boring* design — no Kafka-per-payment fan-out, no 1,000-shard ledger, no microservice explosion. The arithmetic that *does* bite is the amplification ratios and the retention horizon, not raw QPS. This is the Workbook's one-machine-first lesson at bank scale: complexity must be *earned* by a number, and here almost none of it is. (The scale-up story — when 8× becomes 80×, or when the bank adds a wholesale book — is the sharding trade-off of §9.6.)

### 9.3 The Contract: APIs and Data Model (beats 3–4)

```
POST /v1/payments                      GET /v1/payments/{payment_id}
  Idempotency-Key: <client-generated>    → status + events
  { debit_account, credit_account,
    amount, currency, reference }
  → 202 { payment_id, status }
```
Semantics: the `Idempotency-Key` is the *write contract* — a retried identical instruction returns the original result and posts nothing twice. Cancellation (`POST /v1/payments/{id}/cancel`) is allowed only in a pre-posting window. Events (`payment.completed`, `payment.failed`) flow to the notification and reconciliation consumers via an outbox.

Data model (core tables; sizes from §9.2):

| Table | Key | Shape | Notes |
|---|---|---|---|
| `payments` | `payment_id` (ULID); unique `idem_key` per debit account | one row per payment, status enum, ~600 B | idempotency lives here |
| `accounts` | `account_id` | current balance in minor units + version, ~200 B | the balance is derived-state; the entries are the truth |
| `entries` | `posting_id`; indexed by `account_id`, `payment_id` | one row per leg, DR/CR + amount, append-only, ~300 B | the double-entry journal — never updated, only appended + reversed |
| `outbox` | sequential | events to emit, written in the same transaction as the posting | the transactional-outbox pattern (cross-ref the repo's MQ-reliability and outbox material) |

### 9.4 The Architecture (beat 5 — designed for failure)

`API gateway → payment service (stateless × N, idempotency + funds check) → posting engine (single logical transactional store: active-active pair, sync replica + async read replica, PITR backups) → GL and reconciliation consumers (async, outbox-fed) → rail adapter (FAST-style) → notifications`. The cross-account posting runs as one local transaction in the transactional store: debit leg, credit leg, balance updates, outbox row — commit or nothing, so **DR = CR is enforced by the database, not by application hope** (cross-ref [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) §1.1 and §8).

Failure design per component: the payment service is stateless and retries with exponential backoff against the store (idempotency keys make retries safe); the store is the single point of truth, so it gets the sync replica and the backup/recovery drill; the rail adapter treats the rail as an unreliable peer — outbound is idempotent-resumable, inbound credits are reconciled daily against the rail's statement; a zone loss degrades to the sync replica with the 99.95% budget absorbing the gap; the GL/reporting consumers lag rather than fail (async, replayable from the outbox). Observability: SLIs on the payment path — p99 latency, error rate, and *posting lag* — feeding burn-rate alerts against the SLOs (cross-ref §3.5 and the repo's SLO/error-budget material).

### 9.5 The Deep Dive (beat 6): Exactly-Once Posting Under Retries

The interviewer's probe: *"the client times out and retries while the first attempt is mid-post — how do you guarantee the money moves exactly once?"* The answer chain:

1. **Idempotency first:** the `Idempotency-Key` row is inserted with a unique constraint at the start of the transaction; a concurrent retry blocks on the constraint and reads the outcome — one winner, everyone else gets the result. This makes "at-least-once" transport into "exactly-once" effect without distributed locking.
2. **Atomicity second:** the debit, credits, balance updates, and outbox row commit in one transaction; there is no window where the debit exists without the credit (the *unbalanced* failure is structurally impossible; the remaining failure class is *wrong-but-balanced*, which reconciliation and audit catch — cross-ref the posting guide's §1.1 distinction).
3. **Failure mid-flight:** if the store commits but the response is lost, the retry finds the committed payment via the idempotency key and returns it; if the transaction aborted, the retry re-runs cleanly. The only hard case is *ambiguous outcome* (commit unknown) — solved by making the idempotency lookup the single arbiter, never by guessing.
4. **The scale-up trade-off, named:** when write volume outgrows one transactional cluster, the store shards by `account_id` hash — but a cross-account payment then touches two shards, and the atomicity guarantee must be re-purchased (coordinator/saga + reconciliation) or the shard key chosen so related accounts co-locate (cross-ref the shard-key axis of §5.1 and the DDIA partitioning material). The refusal is stated: *no* sharding until the §9.2 numbers demand it, because sharding spends the bank's most valuable invariant — single-transaction DR = CR — against a problem the bank does not yet have.

### 9.6 The Trade-offs and the Wrap-Up (beat 7)

- **Strong consistency on the money path vs availability:** chosen — the SLOs and the invariant require it; the availability budget is spent on the sync replica and recovery drills, not on weakening the read-your-writes guarantee for balances.
- **Transactional store vs ledger microservices:** the honest single-store design (at ~46–370 payments/sec) beats a distributed choreography of entry-services; complexity refused until the numbers earn it.
- **Real-time posting vs batch core:** real-time entries post intraday and feed the GL continuously; the day-end batch (cross-ref [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) §7) reconciles rather than re-posts — the §5.3 boundary made explicit.
- **Residual risks:** rail downtime (mitigated by the resumable adapter and reconciliation), a balance-consistency bug escaping to production (mitigated by the append-only journal + daily reconciliation + the *wrong-but-balanced* audit net), and retention growth (the ~46 TB archive horizon, mitigated by tiering — cross-ref the capacity guide).

**The wrap-up, as the candidate should deliver it:** the requirements were instant domestic payments at ~46/sec average, ~800/sec peak, with 99.95% availability, sub-second p99, and exactly-once posting; the design is a stateless payment service in front of a transactional posting store whose atomicity *is* the accounting invariant, with an outbox feeding async consumers and reconciliation; the main trade-offs are strong consistency on the money path, a deliberately non-distributed ledger until the numbers demand sharding, and real-time posting with a reconciling day-end; the residual risks and their monitors are stated above. The numbers, not the diagram, are why this design is the right size.

---
## 10. The Shelf Map: NALSD vs Grokking vs the Xu Insider's Guide vs the Google Rubric

### 10.1 The Framework Comparison

| Dimension | NALSD (this guide) | Grokking 7-step | Alex Xu 4-step | Google rubric (repo Google guide) |
|---|---|---|---|---|
| Lineage / owner | Google SRE methodology (SRE Workbook Ch. 12, 2018) + SRE interview format (SRECon18, 2018) | Educative course (Fahim ul Haq et al., ~2018) | Alex Xu / ByteByteGo book (2020) | Google hiring rubric as reconstructed by the repo guide |
| Framework | Two phases — basic design, then scaled design — driven by four questions: possible? better? feasible? resilient? | Requirements → estimation → API design → data model → high-level → deep dive → wrap-up | Requirements → estimation → design → deep dive | Requirements → estimation → components → deep-dive → trade-offs |
| Distinctive emphasis | **Concreteness + reliability**: designs iterated into machines/disk/QPS; SLOs and failure modes are first-class; operational cost is a design input | Breadth + repetition: a component library and ~16 recycled problems, so candidates recognize the shape | Product-scale pragmatics: the case-study catalogue and the "how to say it in 45 minutes" rhythm | The level calibration (L4/L5/L6) and the scale math; "engineering peak" performance framing |
| SLO/reliability weight | Highest — the design is *checked against* SLOs and failure modes | Medium — reliability appears as components (queues, redundancy) | Medium — availability mentioned per case study | High for SRE variants; medium for SWE |
| Estimation demand | Non-negotiable and load-bearing (the feasibility pass) | Strong (step 2, back-of-envelope) | Strong (step 2) | Strong (§5 scale math) |
| Interview genre fit | SRE / infrastructure / reliability-flavored and senior design rounds | Generic FAANG system design prep, entry-to-mid | Generic FAANG system design prep, self-study | Google SWE/SRE loops specifically |
| Weakness | Underspecified for product-flavored prompts (feeds, marketplaces) — its canon is reliability-shaped | Cookie-cutter risk; light on theory (per the repo companion's critique notes) | Lighter on ops/failure than the SRE flavor | Google-specific; less portable framing |
| Repo home | **this guide** | [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) | [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) | [google_system_design_interview_guide.md](google_system_design_interview_guide.md) |

### 10.2 What Each Sibling Is For (and What This Guide Adds)

- The **Grokking companion** is the *catalogue* guide: which problems recur and how the 7-step shape walks them (its §2 has the step-by-step time budget).
- The **Xu Insider's Guide companion** is the *case-study* guide: 16 chapters of worked products and the interview-dynamics tips (its §2 is the 4-step framework).
- The **Google rubric guide** is the *level + scale* guide: the L4/L5/L6 expectations and the §5 scale math this guide's §4 condenses.
- **This guide** is the *methodology* guide: it answers the question the other three assume — *why is every one of these frameworks built on requirements, numbers, contracts, and trade-offs?* — because that is the Google SRE design discipline, documented under the name NALSD, and the concrete-thinking habit is what interviewers (whether or not they use the acronym) actually score.

### 10.3 Beat-to-Section Map for a Candidate Rehearsing With the Shelf

| If you are practicing… | Use the framework from… | Bring the numbers from… | Check the reliability lens in… | Rehearse on… |
|---|---|---|---|---|
| The generic product round | Grokking §2 or Xu §2 | [capacity_sizing_guide.md](capacity_sizing_guide.md) §8 / this guide §4 | [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) | The component deep-dives ([distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md), [s3_architecture_guide.md](s3_architecture_guide.md)) |
| The Google SWE round | [google_system_design_interview_guide.md](google_system_design_interview_guide.md) §4 | same | same, plus §10 of the Google guide | The Google guide's worked example |
| The Google SRE / NALSD round | this guide §3 + §6 | this guide §4 | this guide §3.5 + §5 | this guide §7 walkthroughs + §9 |
| The banking "design in my domain" round | any framework; this guide §3 is the best fit | this guide §4.5 + capacity guide | [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) §8 | this guide §9 + the §7.3 reading |

### 10.4 The Two-Week NALSD Practice Plan (condensed from the §1.6 loop)

The repo Google guide's 4–6 week practice plan (its §9) is the generic template; this is the NALSD-specific two-week condensation for a candidate who already knows the frameworks and needs the method:

| Day(s) | Focus | Material | Exit check |
|---|---|---|---|
| 1–2 | Concept and structure | §2, §3 (this guide); skim §6 | Can state the four NALSD questions from memory and map the six beats onto Grokking's 7 steps |
| 3–4 | Estimation reflex | §4 + [capacity_sizing_guide.md](capacity_sizing_guide.md) §8 + the Google guide §5 math | The §4.6 40-second drill, five prompts, without hesitation |
| 5–6 | Trade-off fluency | §5 + [ddia_study_companion_guide.md](ddia_study_companion_guide.md) consistency chapters | The §5.4 three-sentence template, delivered on five random axes |
| 7–9 | Walkthrough mocks | §7.1–7.3, one system per sitting, §7.5 protocol | Each mock scores ≥5 of the §6.5 rows on self-grade |
| 10 | Worked-example rehearsal | §9 delivered aloud, full six beats | The wrap-up (§9.6) delivered without notes |
| 11–12 | Failure-mode hardening | §8 as a checklist; re-run the two weakest mocks | The three named failure rows from session 7–9 no longer appear |
| 13–14 | Full mock + scorecard | One fresh prompt (invent one in your own domain), §6.5 scoring | All six scorecard rows pass; two live mocks with a peer if available |

The plan's spine is the same as the method's: **requirements → numbers → contract → architecture → failure → wrap-up**, rehearsed until the order is automatic. A candidate who completes it has not memorized answers; they have made the non-abstract pass a reflex — which is the only durable thing the round can test.

---
## 11. The Claims Audit

*Every research claim in this guide, with its status and source note. ✅ = verified this pass against a primary or authoritative source; ⚠ = flagged (single-source, third-party, approximate, contested, or not re-verifiable this pass); ❌ = refuted. Unmarked statements are domain-stable technical knowledge, not research claims. No quotes, URLs, or facts were invented; where a source could not be reached or confirmed, the claim is ⚠ or lives in §11.5.*

### 11.1 The Origin and Usage Claims

| # | Claim | Status | Source note |
|---|---|---|---|
| 1 | NALSD expands to "Non-Abstract Large System Design" (sources also write "Large Systems") | ✅ | SRE Workbook Ch. 12 title and body; Kirsch SRECon18 abstract ("Non-Abstract Large Systems Design"); prep sources use both phrasings. The singular/plural variance is a wording variant, not a different concept |
| 2 | NALSD is a Google SRE methodology: "We call this style Non-Abstract Large System Design (NALSD)" | ✅ | sre.google/workbook/non-abstract-design/ — extracted this pass (quoted in §2.1) |
| 3 | NALSD combines capacity planning, component isolation, and graceful degradation; SREs must turn whiteboard designs into concrete resource proposals | ✅ | same chapter, "What Is NALSD?" (quoted in §2.1) |
| 4 | The process is iterative: problem statement → requirements → designs of increasing sophistication; probed by four questions (possible / better / feasible / resilient) | ✅ | same chapter, "Design Process" and "Conclusion" sections |
| 5 | "Reasoning and assumption making are more important than any final values" | ✅ | same chapter, "Why 'Non-Abstract'?" section |
| 6 | NALSD is a Google SRE *interview* format and every Google SRE candidate sits one NALSD interview | ✅ | Kirsch, "Interviewing for Systems Design Skills", SRECon18 Asia, USENIX, 8 June 2018 — abstract at research.google/pubs and usenix.org (extracted this pass) |
| 7 | The interview measures design for scalability/reliability/robustness, estimating provisioning needs, and managing change | ✅ | same talk abstract |
| 8 | NALSD was "popularized by the Google SRE Workbook" | ✅ (published artifact) / ⚠ (primacy) | the Workbook Ch. 12 is the canonical public text (2018) and prep glossaries attribute popularization to it (tieroneprep.com/glossary/nalsd); whether earlier internal Google artifacts coined it is not verifiable (§11.5) |
| 9 | Google still uses NALSD (2023–2026): SRE Classroom workshops, Google Cloud blog announcement, sre.google resources pages | ✅ | sre.google/classroom/ and sre.google/classroom/distributed-pubsub/ and /imageserver/; cloud.google.com blog "Join SRE Classroom NALSD workshops"; sre.google resources "addressing cascading failures" page (all surfaced this pass; classroom pages extracted via search snippets) |
| 10 | The repo Google guide's framing that NALSD was "the older SRE designation" with "the mindset persist[ing]" | ⚠ | repo-internal claim; consistent with sources 6–9 only if read as "the *round name* evolved in recruiting materials while the methodology label persisted" — the exact naming history of the round is not pinned to a Google primary source this pass (§11.5) |
| 11 | Amazon / Meta / Microsoft / Netflix use the NALSD label for their design rounds | ⚠ (no evidence) | two targeted searches returned zero results tying the term to those companies; absence of evidence only (§11.5) |
| 12 | "NALSD" is an acronym for networking/application/Linux/system-design evaluation dimensions | ❌ | explicitly rejected by multiple independent sources (e.g. the AceInterviews SRE walkthrough); the only documented expansion is Non-Abstract Large System Design |

### 11.2 The Method and Format Claims

| # | Claim | Status | Source note |
|---|---|---|---|
| 13 | Grokking teaches a 7-step framework (requirements → estimation → API → data model → high-level → deep dive → wrap-up) | ✅ (course content, cross-ref) | verified via [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) §2 (itself verified in that guide's ledger) |
| 14 | Alex Xu's Insider's Guide teaches a 4-step framework (requirements → estimation → design → deep dive) | ✅ (book content, cross-ref) | verified via [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) §2 |
| 15 | The repo Google guide's framework is requirements → estimation → components → deep-dive → trade-offs, with scale math in its §5 | ✅ (repo cross-ref) | [google_system_design_interview_guide.md](google_system_design_interview_guide.md) §4–§5 |
| 16 | The AdWords CTR example and its numbers (500,000 queries/sec, 10,000 clicks/sec, ~2 KB log entries, ~4 TB RAM / 64 machines / 25% network per datacenter) | ✅ | SRE Workbook Ch. 12, extracted this pass (§2.3, §4.2) |
| 17 | SRE-round prep specifics: existing-production-system prompts; math-before-drawing; "requirement is physically impossible" is a pass signal; 45-minute length; levels framing | ⚠ | AceInterviews google-sre-interview-handbook (GitHub) and dev.to/hashnode walkthroughs — single-source-ish third-party, consistent in spirit with the primary sources but not pinned to a Google primary document (§11.5) |
| 18 | The log-ingestion worked pattern (1M hosts × 10 KB/s → 10 GB/s → ~100 disks → ~300 with 3× replication) | ⚠ | tieroneprep.com/glossary/nalsd — third-party example; arithmetic internally consistent, used here only as an illustration (§4.3) |
| 19 | Typical design-round length is 45–60 minutes with ~5-minute wrap-up convention | ⚠ | aggregated interview reports as recorded in the repo Google guide §1; prep-ecosystem convention, not a published rubric |
| 20 | The failure modes of §8 are the common candidate mistakes | ✅ (as reported) / ⚠ (as canon) | the *list* is consistently reported across the primary sources' concerns (Workbook: feasibility/resilience/simplicity; Kirsch: provisioning/change) and third-party handbooks; no single authoritative "failure list" exists |

### 11.3 The Sibling-Ledger Cross-References

| # | Cross-ref | Status |
|---|---|---|
| 21 | Rail peak figures (Visa ~8,500 tps avg / >65,000 tps peak; NETS Gateway per-instance floor; FAST 24/7), Kafka topic/partition/retention catalogue, nines math, Little's Law, ITIL 4 practice | ✅ (cross-referenced, not re-derived) | the repo's [capacity_sizing_guide.md](capacity_sizing_guide.md), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) and banking guides carry these in their own verified ledgers; this guide cites them only as pointers |
| 22 | Double-entry DR = CR and the wrong-but-balanced vs unbalanced distinction; the posting lifecycle; EOD/batch processes | ✅ (cross-referenced) | [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) §1.1/§3/§8; [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md) §6–§7 |
| 23 | Trading-system structure (order lifecycle, risk controls, latency engineering), middleware fabric taxonomy, legacy integration patterns | ✅ (cross-referenced) | the §7 walkthroughs condense these guides' own structures; section numbers cited match their ToCs (read this pass) |

### 11.4 What This Guide Does Not Claim

- No claim that NALSD is the *only* design method Google uses (the Workbook presents it as the SRE style for production systems; product design at Google is a different, larger practice).
- No claim about current (2026) Google recruiting materials' exact round names — see §11.5.
- No original numbers beyond the §9 design-fiction assumptions (all stated as such) and the §4/§7 arithmetic reproduced from cited sources.
- The Cymbal Bank worked example (§9) is design fiction; nothing in it is asserted as fact about any real bank or rail.

### 11.5 What Could Not Be Verified

The following could not be confirmed to a primary or authoritative source this pass, and are therefore flagged rather than asserted:

- **The coinage history of the term.** The earliest public artifacts found are both from 2018 (SRE Workbook Ch. 12; the SRECon18 talk). Whether "NALSD" was coined inside Google earlier — as interview-lore naming for the SRE systems-design round — is plausible (the repo Google guide's "older SRE designation" claim) but no dated primary source before 2018 was found. The naming history of the SRE design round in Google recruiting materials (round renamed vs methodology label persisting) is likewise unverified.
- **Current-round naming at Google.** Kirsch (2018) is the primary anchor for NALSD-as-interview-format; SRE Classroom (2023–2026) confirms NALSD-as-teaching. No Google primary document from 2024–2026 describing the live SRE interview round's name was reachable this pass; third-party 2024–2026 prep sources (AceInterviews, dev.to/hashnode walkthroughs, tieroneprep) consistently call the round NALSD. ⚠
- **Format specifics of the SRE round** (existing-system prompts, 45-minute length, leveled expectations, "math before drawing" as an explicit rule): reported by the prep ecosystem, consistent with the primary sources' spirit, but not confirmed in a Google-published rubric. ⚠
- **Usage of the NALSD label outside Google.** Two web-search queries for NALSD with Amazon/Meta/Microsoft/Netflix returned no results; the search backend was partially degraded this pass (several queries returned empty), so this is a weak negative. No claim is made that other companies never use the term. ⚠
- **The exact routing of the SRE Workbook's Ch. 12 numbers** (some intermediate arithmetic of the AdWords example was in the truncated middle of the extracted page and not re-verified figure-by-figure; the quoted results — 64 machines, ~4 TB RAM, 25% network — appear in the extracted tail and are quoted as found). ⚠
- **tieroneprep's claim wording** that NALSD is "sometimes written non-abstract large-scale system design": a wording variant found in that single glossary; other sources use "large system(s) design". ⚠

---
## 12. The Glossary

- **NALSD (Non-Abstract Large System Design)** — Google SRE's iterative system-design methodology: from problem statement and requirements, iterate designs until they are expressed in concrete, quantified resources that provably satisfy the requirements at scale. Also the name of Google SRE's systems-design interview format (Kirsch, 2018).
- **Non-abstract** — the property of a design whose components carry quantities and failure modes, not just labels; "a design is not finished while it is abstract."
- **The four NALSD questions** — Is it possible? Can we do better? Is it feasible? Is it resilient? (SRE Workbook Ch. 12).
- **Abstract design** — the textbook layered-diagram style this method opposes: correct-looking boxes without the numbers, contracts, or failure analysis that make a design real (§2.3).
- **Back-of-envelope estimate** — order-of-magnitude arithmetic from stated assumptions (rate × size → throughput → resources), the NALSD feasibility pass (§4).
- **QPS (queries/requests per second)** — the traffic unit of every estimation chain; average from (users × actions ÷ 86,400), peak as a multiple of average.
- **Capacity planning / provisioning** — estimating the resources (machines, disk, RAM, bandwidth) a design needs; the skill Kirsch's interview abstract names first.
- **SLI / SLO / error budget** — the measured indicator (SLI), the target (SLO), and the allowed failure allowance (error budget); NALSD practice states requirements as SLOs and checks each design iteration against them (cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md)).
- **Idempotency** — the property that repeating an operation has the same effect as doing it once; the mechanism (idempotency key + unique constraint) that makes exactly-once posting achievable over at-least-once transport (§9.5).
- **Double-entry posting / DR = CR** — the accounting invariant that every transaction's debits equal its credits; enforced atomically by the posting engine (cross-ref [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md)).
- **Transactional outbox** — writing events in the same transaction as the state change so the event stream cannot diverge from the truth; the async-consumer pattern used in §9.4.
- **CAP / PACELC** — the consistency-availability trade-off framing (CAP under partition; PACELC adds latency-vs-consistency when there is no partition); the theory layer is [ddia_study_companion_guide.md](ddia_study_companion_guide.md).
- **Read-heavy / write-heavy** — the workload shape that drives storage, cache, and index choices (§5.1).
- **Cache-aside / read-through / write-through** — caching placements; the cache is a speedup, never the source of truth.
- **Push vs pull** — delivery models; push for latency, pull for replayability and consumer independence (§5.1).
- **Deep dive** — the interview beat where the candidate goes depth-first into one bottleneck or risk (§3.6).
- **Wrap-up** — the closing beat: restate requirements, name embodied trade-offs and refused alternatives, state residual risks and monitoring (§3.7).
- **Design round** — the open-ended architecture interview of FAANG-style loops; "system design" for SWE, NALSD-flavored for SRE, "design in my domain" variants for specialists.
- **Bar raiser** — an Amazon loop role (an independent senior interviewer guarding the hiring bar); a *role*, not the name of a design round (⚠ — see §6.3).
- **SRE Classroom** — Google SRE's public NALSD workshop series (sre.google/classroom) — primary evidence that NALSD is a current, taught methodology.
- **Cymbal Bank** — the repo's fictional Singapore-based bank persona; the setting of §9's design-fiction worked example.
- **Design fiction** — house convention: worked examples that are clearly marked as illustrative exercises built on verified facts, never passed off as real systems (§9 notice).

---
## 13. Cross-references and the Closing Summary

### 13.1 The Shelf Map Recap

- **The framework companions:** [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) (7-step), [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md) (4-step), [google_system_design_interview_guide.md](google_system_design_interview_guide.md) (the rubric, levels, and scale math — and the sibling whose §1.2 NALSD heritage claim this guide verified independently).
- **The theory and component layers:** [ddia_study_companion_guide.md](ddia_study_companion_guide.md), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md), [ml_system_design_interview_guide.md](ml_system_design_interview_guide.md), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md), [s3_architecture_guide.md](s3_architecture_guide.md).
- **The numbers layer:** [capacity_sizing_guide.md](capacity_sizing_guide.md) — the discipline behind §4.
- **The worked large systems (condensed in §7):** [trading_system_software_architecture_guide.md](trading_system_software_architecture_guide.md), [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), [../banking/posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md), [../banking/core_banking_processes_guide.md](../banking/core_banking_processes_guide.md).

### 13.2 Primary Sources for This Guide

- Google SRE Workbook, Ch. 12 "Introducing Non-Abstract Large System Design" — https://sre.google/workbook/non-abstract-design/ (✅ extracted this pass)
- Sebastian Kirsch (Google), "Interviewing for Systems Design Skills", SRECon18 Asia, USENIX, Singapore, 8 June 2018 — https://research.google/pubs/interviewing-for-systems-design-skills/ and https://www.usenix.org/conference/srecon18asia/presentation/kirsch-interviewing (✅ extracted this pass)
- Google SRE Classroom — https://sre.google/classroom/ (workshop pages: distributed-pubsub, imageserver) (✅ surfaced this pass)
- Google Cloud blog, "Join SRE Classroom NALSD workshops" (✅ surfaced this pass)
- Third-party prep (⚠ where cited): AceInterviews google-sre-interview-handbook (github.com/AceInterviews/google-sre-interview-handbook), tieroneprep.com NALSD guide and glossary, dev.to/hashnode SRE NALSD walkthroughs

### 13.3 The Closing Summary

NALSD is the documented design discipline of Google SRE — requirements first, SLOs as the contract, feasibility math before any diagram, every component carried to concrete machines and bytes, every failure mode designed for, and every iteration checked against the four questions: possible, better, feasible, resilient. For the interview candidate it explains why every serious design framework on this shelf is built the way it is: the shape is not a convention, it is the non-abstract method, and the interviewers — at Google SRE under the NALSD name, and everywhere else under the pressure of "how many machines is that?" — are scoring the visible chain from assumption to arithmetic to architecture. For the Solution Architect it is the formalization of the working day: gather the requirement, set the target, size the estate, design the contract, argue the trade-off, design the failure. Practice it on the repo's large systems, rehearse the numbers until they are reflex, and walk into the round holding this one conviction above all: the diagram is only the beginning, the boxes earn their place only when the numbers close, and the answer — the whole answer — is the concrete design.
