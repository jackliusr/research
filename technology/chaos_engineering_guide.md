# Chaos Engineering: The Steady State — A Comprehensive Guide

**The Discipline Deep-Dive — the Canon and Its Origins (the Netflix Simian Army, Dated and Verified), the Principles and Their Critique, the Experiment Design Craft, the Failure Taxonomy, the Tooling Landscape with Live Maintenance Status, the Observability Prerequisite, the Safety Guardrails, the Practice and Its Culture, the Anti-Patterns, the Regulated-Institution Angle (DORA/TLPT, PRA/FCA, CBEST, MAS, BCBS), a Cymbal Bank Worked Programme, the Claims Audit, and What Could Not Be Verified**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Series:** Technology / Reliability & Resilience — the discipline of *deliberately breaking things to find out what breaks on its own*.
> **Audience:** SREs and platform engineers who will run the experiments; engineering managers and resilience leads who must sponsor, scope and defend them; and — the audience this guide is really written for — the architect in a regulated institution who has to reconcile "inject real failure into production" with a supervisor who wants a documented, evidenced, controlled test.
> **Context:** This guide OWNS chaos engineering as a *discipline* — its canon, its craft, its tooling, its culture, and its collision with regulated financial services. It deliberately does NOT re-derive the availability mathematics, the SLO/error-budget machinery, the deployment strategies (rolling/blue-green/canary/feature flags) or the resilience patterns (circuit breakers, bulkheads, retries, idempotency, graceful degradation) — [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) owns those. **The boundary is explicit and important:** that sibling guide's §7 (lines 420–461, roughly 44 lines across five subsections — §7.1 The Chaos Monkey, §7.2 The Principles of Chaos, §7.3 GameDays, §7.4 The Chaos Table, §7.5 Building the Practice) contains a *compact summary* of chaos engineering inside an availability guide. This is the treatment. This guide expands it — and in one place **corrects** it.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — plain filenames):** [Zero-Downtime System Design](zero_downtime_system_design_guide.md) (**THE parent companion — §7 is the summary this guide expands; §2 owns the availability math, §3 the deployment strategies, §8 the resilience patterns, §9 the 24-7 payments switch**), [OpenShift Workload Availability](openshift_workload_availability_guide.md) (**THE platform companion — §28 owns platform-specific chaos on Kubernetes/OpenShift (Chaos Mesh, LitmusChaos, pod failure, PDB/anti-affinity); §29 owns DR drills**), [Enterprise Middleware Integration Platform](enterprise_middleware_integration_platform_guide.md) (the middleware estate a bank's first experiments will target), [IBM MQ Disaster Recovery](ibm_mq_disaster_recovery_guide.md) (the messaging failover drill — a different tool from a chaos experiment), and [Billion-User System Architecture](architecture/billion_user_system_arch.md) (**§5.3 owns Netflix at cloud scale — cross-ref rather than re-derive**).
> **Companion guides (banking/, prefix `../banking/`):** [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (**THE regulatory companion — §2 the regulatory landscape (MAS, PRA/FCA, DORA, BCBS), §4–5 BIA and impact tolerances, §6 severe-but-plausible scenarios, §8 disaster recovery, §11 the testing and assurance framework; this guide cross-refs it for the compliance machinery rather than re-deriving it**), [Payment Rails](../banking/payment_rails_guide.md), [Payments Hub](../banking/payments_hub_guide.md), [NETS Singapore](../banking/nets_singapore_guide.md), [Core Banking Systems](../banking/core_banking_systems_guide.md), [IBPS Payment Connect](../banking/ibps_payment_connect_guide.md).
> **Primary Sources for this pass:** the Principles of Chaos document at [principlesofchaos.org](https://principlesofchaos.org/) (extracted live, 2026-09-17); the original Netflix Simian Army post (Netflix Tech Blog, 19 July 2011 — extracted live via an archived mirror of the original `techblog.netflix.com/2011/07/netflix-simian-army.html` text); the O'Reilly publication record for *Chaos Engineering: System Resiliency in Practice*; the live repository pages of Chaos Monkey, Chaos Mesh, LitmusChaos and Toxiproxy (all extracted 2026-09-17); DORA Regulation (EU) 2022/2554 (Article 26 full text); the EBA's own single-rulebook page for the joint TLPT Regulatory Technical Standards (confirming in-force status and application date, linking to OJ CELEX 32025R1190); the Bank of England's CBEST programme pages; the ABS (Association of Banks in Singapore) Adversarial Attack Simulation Exercises guidelines.
> **Scope note on verification:** This pass had **live web access** through a self-hosted Firecrawl backend, and used **web_search** for discovery and **web_extract** for primary-document retrieval (the extracted pages are the basis of every ✅ in this guide). Where extraction failed — the `netflixtechblog.com` host refused scraping — I fell back to **direct primary URLs and an archived mirror of the original post text**, and I say so explicitly in §2 rather than pretending the primary host answered. Where **web_search returned zero results** (two queries on chaos-engineering-caused incidents and on AWS/Azure managed-service naming all returned empty result sets), I record that as an empty search, not as a negative fact — an empty search is not evidence of absence, and §8 treats it as such.
> **Integrity convention:** ✅ = verified this pass against the cited source; ⚠ = flagged, partial, or dependent on a non-primary source; ❌ = rejected (asserted somewhere, found false or unsupported). Used in every table that makes a factual claim.
> **Illustrative figures:** Every number attributed to **Cymbal Bank** (the only bank persona in this repository) is **explicitly fictional and illustrative** — a worked shape for a programme, not a benchmark, not a survey result, and not a claim about any real institution.

**How this guide is organised.** §1 draws the boundary — what chaos engineering is and, just as importantly, what it is *not* — and gives the decoder table. §2 is the canon and the origins: the 2011 Simian Army post and exactly which simians it named, the Principles of Chaos as they *currently* read, and the book lineage. §3 takes those principles one at a time and argues with them. §4 is the craft — the experiment design, with a full worked template. §5 is the failure taxonomy — the faults worth injecting, what each proves, and what can inject it. §6 is the tooling landscape with **live maintenance status and the date of the finding for every entry**. §7 is the observability prerequisite. §8 is safety and the guardrails, including the pre-flight checklist and the paradox that an experiment can *be* the incident. §9 is the practice and its culture. §10 is the regulated-institution angle — the most valuable section, and the one where every status is dated. §11 is the banking engineering reality, bounded and cross-referenced. §12 is the Cymbal Bank worked programme. §13 is the anti-patterns. §14 is the claims audit. §15 is what could not be verified. §16 is the glossary, §17 the cross-references, §18 the closing.
**Source discipline (applied throughout, and you should apply it too):** this field is unusually full of **vendor content marketing dressed as methodology**. Every claim in this guide is tagged as one of three things — a **PRIMARY DOCUMENT** (a specification, a regulation, a repository, a publication record), a **VENDOR'S CLAIM ABOUT ITS OWN TOOL** (a product page or a vendor blog: treat as marketing until independently corroborated), or a **THIRD-PARTY SUMMARY** (a consultant's explainer, a news article, a directory listing — useful for leads, not for facts). Where the distinction matters, §14's claims audit names which is which.

## Table of Contents

1. [The Overview, the Boundary and the Decoder](#1-the-overview-the-boundary-and-the-decoder) — 1.1 the one-paragraph definition · 1.2 what chaos engineering is not (the four negative definitions) · 1.3 the boundary against the sibling guides · 1.4 the decoder (glossary in advance)
2. [The Canon and the Origins](#2-the-canon-and-the-origins) — 2.1 the Simian Army, dated · 2.2 the roll-call (what was actually announced) · 2.3 the Principles of Chaos and a correction to the sibling guide · 2.4 the book lineage · 2.5 from internal culture to market
3. [The Principles, Examined](#3-the-principles-examined) — 3.1 the five, one at a time · 3.2 run in production vs regulation · 3.3 automate continuously · 3.4 the load-bearing assumption (do you have a steady state?)
4. [The Experiment Design](#4-the-experiment-design) — 4.1 the nine steps · 4.2 the steady state and its tolerance · 4.3 abort conditions and the kill switch · 4.4 the worked design (adaptable template)
5. [The Failure Taxonomy](#5-the-failure-taxonomy) — 17 fault classes, each with mechanism, blast radius, what it proves, and injectors
6. [The Tooling Landscape](#6-the-tooling-landscape) — 6.1 the dead-project rule · 6.2 commercial platforms · 6.3 foundation and open-source · 6.4 cloud providers · 6.5 proxy/process-level · 6.6 in-house · 6.7 the verdict table
7. [The Observability Prerequisite](#7-the-observability-prerequisite) — the steady-state signal, the three signal types, the false-positive problem, the control room
8. [Safety and the Guardrails](#8-safety-and-the-guardrails) — 8.1 blast-radius control · 8.2 the kill switch as a mechanism · 8.3 stakeholders, change management, environment · 8.4 the paradox (when the experiment is the incident) · 8.5 the pre-flight checklist
9. [The Practice and the Culture](#9-the-practice-and-the-culture) — 9.1 the game day · 9.2 champions and the blameless prerequisite · 9.3 maturity progression · 9.4 the honest metrics
10. [The Regulated-Institution Angle](#10-the-regulated-institution-angle) — 10.1 DORA and the EU TLPT regime · 10.2 the UK and CBEST · 10.3 Singapore and AASE · 10.4 the BCBS material · 10.5 the structural tension · 10.6 the resolutions · 10.7 the evidence question · 10.8 the third-party and cloud dimension
11. [The Banking Engineering Reality](#11-the-banking-engineering-reality) — the estate, the safe start, the calendar, the DR relationship, the honest ceiling
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example) — 12.1 scoping by criticality · 12.2 the first three experiments in full · 12.3 environment, timing, tooling, notification · 12.4 findings, remediation, regulator-facing evidence · 12.5 cost, risk register, and what it still cannot prove
13. [The Anti-Patterns](#13-the-anti-patterns) — nine, each with symptom, cause and guardrail
14. [The Claims Audit](#14-the-claims-audit) — verified / flagged / rejected, with source quality and date
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary](#16-glossary)
17. [Cross-References and Further Reading](#17-cross-references-and-further-reading)
18. [Closing Summary](#18-closing-summary)

## 1. The Overview, the Boundary and the Decoder

### 1.1 The One-Paragraph Definition

Chaos engineering is the discipline of **experimenting on a system in order to build confidence in the system's capability to withstand turbulent conditions in production**. That wording is not mine — it is the opening sentence of the Principles of Chaos document at [principlesofchaos.org](https://principlesofchaos.org/), which I extracted live on **17 September 2026** ✅ and which carries the line "Last Update: 2019 March" ✅ at the head of the page. Three words in that sentence do the real work, and most bad practice is the result of dropping one of them:

- **experimenting** — there is a hypothesis, a control, and a search for disconfirming evidence. A fault injected without a hypothesis is not an experiment; it is an outage you scheduled.
- **confidence** — the output of the practice is *knowledge*, expressed as increased confidence in a resilience property. The output is not "we ran chaos". If no belief changed, nothing was learned.
- **in production** — the discipline's most distinctive and most contested claim (§3.2). The principles doc argues it directly: "Systems behave differently depending on environment and traffic patterns… sampling real traffic is the only way to reliably capture the request path" ✅.

Everything else in this guide is downstream of those three words. When you are deciding whether some proposed activity is chaos engineering, asking which of the three it satisfies is usually enough to answer.

### 1.2 What Chaos Engineering Is Not — the Four Negative Definitions

Definitions by negation matter more here than definitions by assertion, because chaos engineering has an unusually large number of look-alikes that people label with its name. These four are the ones I encounter most in banking engineering reviews.

| It is not… | Why not | What it actually is | The tell |
| --- | --- | --- | --- |
| **Random breakage** | An experiment has a hypothesis, a bounded blast radius and an abort condition. Randomness without those is just a fault you chose not to control. Chaos Monkey's own README describes terminating instances — but the whole point is that it runs "in the middle of a business day, in a carefully monitored environment with engineers standing by" (Simian Army post, 19 July 2011 ✅). The monitoring and the standing-by are not decoration. | Controlled fault injection under observation. | If nobody can state the hypothesis out loud, it is not an experiment. |
| **Conventional QA testing** | QA asks "does the system do what the specification says?" Chaos asks "what does the system do when a real-world condition it was never specified against occurs?" A passing regression suite tells you the known paths work. | A search for *unknown* systemic weakness, in the interactions between components that individually behave correctly. | The principles doc: "Even when all of the individual services in a distributed system are functioning properly, the interactions between those services can cause unpredictable outcomes" ✅. |
| **A DR drill** | A DR drill proves you can *recover* — it is a planned failover to a standby, usually to a target RTO/RPO, usually in a maintenance window, usually with a known-good path back. It is a plan executed. | A chaos experiment proves you can *survive* — it removes a capability without failover, and asks whether the remainder absorbed the loss. Note that the Simian Army post frames it exactly this way: "just designing a fault tolerant architecture is not enough. We have to constantly test our ability to actually survive these 'once in a blue moon' failures" ✅. | A DR drill with a *hypothesis you expect to fail* starts to resemble an experiment; a chaos experiment with a *planned failover* is a DR drill wearing the wrong badge. |
| **Fault-tolerance DESIGN** | Design is the mechanism (the circuit breaker, the bulkhead, the retry budget, the replica). Chaos engineering is the *verification* that the mechanism works under the conditions it was designed for. | Adversarial verification of designed resilience. | [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §8 owns the patterns; this guide owns the proof. |

Frequently confused as well, and worth naming: **load testing** (generates demand to find capacity limits — a different axis), **penetration testing** (an adversary with intent, exploiting vulnerabilities — §10 shows how the regulated world merges this with resilience testing into TLPT), and **fuzzing** (malformed input to find parsing/crash bugs — input-space, not fault-space).

### 1.3 The Boundary Against the Sibling Guides

This repository has already covered the surrounding ground, and re-deriving it would produce a worse guide and a longer one. The boundaries are:

| Sibling guide | What it owns | What this guide does |
| --- | --- | --- |
| [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §7 (lines 420–461) | A **compact summary** of chaos engineering inside an availability guide — the Chaos Monkey, the Principles of Chaos, GameDays, a chaos table, and the blast-radius ramp. | **Expands it into the full treatment** — and corrects one factual error in it (§2.3: the sibling states four principles and omits 'Minimize Blast Radius' from the numbered list; the current canon has **five** ✅). |
| [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2–§3, §8–§9 | The availability math (the nines, MTBF/MTTR), the SLO/SLA/error-budget machinery, the deployment strategies, the resilience patterns, the 24-7 payments switch. | **Cross-references, never re-derives.** §7 of this guide depends on their error-budget definitions; §11 depends on their payments reality. |
| [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) | The **regulatory machinery**: §2 the regulatory landscape, §4–5 business impact analysis and impact tolerances, §6 severe-but-plausible scenarios, §8 disaster recovery, §11 testing and assurance. | **Cross-references for the compliance framework**, and in §10 goes to the **primary instruments** (the DORA article text, the EBA RTS page, the BoE CBEST pages, the ABS guidelines) to verify the provisions that bear specifically on *testing*. |
| [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §28–§29 | Platform-specific chaos on Kubernetes/OpenShift (Chaos Mesh, LitmusChaos, pod failure, PDB/anti-affinity) and DR drills. | **Cross-references for platform mechanics.** §6 covers those tools as a *market* (maintenance status, licensing, ownership) rather than as an installation runbook. |
| [architecture/billion_user_system_arch.md](architecture/billion_user_system_arch.md) §5.3 | Netflix at cloud scale. | **Cross-references.** §2 uses the Simian Army's *own 2011 announcement* as the origin document. |

### 1.4 The Decoder — Glossary in Advance

Every term here is expanded in §16; this table exists so the rest of the guide is readable without flipping.

| Term | Working definition |
| --- | --- |
| **Steady state** | A measurable output of the system that indicates normal behaviour — throughput, error rate, latency percentiles — used as the baseline against which an experiment's effect is judged. Express as a *band*, not a point. |
| **Hypothesis** | A falsifiable statement that the steady state will be *maintained* in both a control and an experimental group while a variable is introduced. |
| **Blast radius** | The set of users, transactions, services and downstream systems that can be *affected* if the experiment goes wrong. Bounded by scope (percentage of instances / of traffic), by environment, and by duration. |
| **Game day** | A scheduled, facilitated exercise in which one or more experiments are run with defined roles, a control room and a written record. |
| **Fault injection** | The mechanical act of introducing a variable — terminating an instance, adding latency, corrupting a cache. Fault injection is the *means*; an experiment is the *method*. |
| **Abort condition** | A pre-declared, machine-checkable threshold that halts the experiment. Declared *before* the experiment, not improvised during it. |
| **Kill switch** | The tested mechanism by which any participant can stop the experiment immediately — and which restores the system to its pre-experiment state. |
| **Control group / experimental group** | The comparison the method rests on: identical traffic, identical configuration, one differing variable. |
| **TLPT** | Threat-Led Penetration Testing — the DORA-mandated advanced test, defined in Regulation (EU) 2022/2554 Article 26 ✅. |
| **CBEST** | The Bank of England's threat-intelligence-led assessment framework ✅. |
| **AASE** | Adversarial Attack Simulation Exercises — the Association of Banks in Singapore's "red teaming" guidelines ✅. |

## 2. The Canon and the Origins

### 2.1 The Origin — the Netflix Simian Army, Dated

The origin document is a Netflix Tech Blog post titled **"The Netflix Simian Army"**, bylined **Yury Izrailevsky, Director of Cloud & Systems Infrastructure, and Ariel Tseitlin, Director of Cloud Solutions**, and dated **19 July 2011** ✅. The canonical URL cited in the original source line is `http://techblog.netflix.com/2011/07/netflix-simian-army.html`.

**A note on how I got it, because it matters for source discipline.** My first extraction attempt against the modern `netflixtechblog.com` re-host **failed** — the Firecrawl backend returned an Internal Server Error, with all scraping engines failing on that host ✅ (a retrieval failure, recorded here rather than papered over). My second attempt succeeded against an **archived mirror that reproduces the original post text verbatim and cites the original source URL** — [chris.improbable.org/2011/7/19/the-netflix-simian-army/](https://chris.improbable.org/2011/7/19/the-netflix-simian-army/) ✅ — which also independently corroborates the **19 July 2011** date in its own URL slug ✅. So: the date and the authorship are corroborated by **two sources, one of which is a third-party mirror explicitly reproducing the primary**, and the primary host is currently unreachable by automated extraction. That is a ✅ with a stated caveat, not a clean primary hit.

The passage that founded the discipline is worth reading exactly as written, because the practitioners who cite it usually paraphrase it:

> "Imagine getting a flat tire. Even if you have a spare tire in your trunk, do you know if it is inflated? Do you have the tools to change it? And, most importantly, do you remember how to do it right? One way to make sure you can deal with a flat tire on the freeway, in the rain, in the middle of the night is to poke a hole in your tire once a week in your driveway on a Sunday afternoon and go through the drill of replacing it. This is expensive and time-consuming in the real world, but can be (almost) free and automated in the cloud." ✅

Note what the metaphor actually claims: the value is in **rehearsing a recovery you have already designed**, and the cloud's contribution is that the rehearsal became cheap. It is a maintenance argument, not a "break things for science" argument. The post then states the design-versus-verification distinction directly — "just designing a fault tolerant architecture is not enough. We have to constantly test our ability to actually survive these 'once in a blue moon' failures" ✅ — and, crucially for §8, it places the tool **in a controlled frame from day one**: Chaos Monkey runs "in the middle of a business day, in a carefully monitored environment with engineers standing by to address any problems" ✅.

### 2.2 The Simian Army Roll-Call — What Was Actually Announced

This list is routinely inflated in secondary writing. Here is what the **2011 post itself** named, in the order the post names them, with what each actually does **in the post's own words** ✅:

| Simian | What the 2011 post says it does | Class of fault |
| --- | --- | --- |
| **Chaos Monkey** | "randomly disables our production instances to make sure we can survive this common type of failure without any customer impact" | Instance termination |
| **Latency Monkey** | "induces artificial delays in our RESTful client-server communication layer to simulate service degradation and measures if upstream services respond appropriately"; large delays simulate "a node or even an entire service downtime" without physically taking it down | Network / latency |
| **Conformity Monkey** | "finds instances that don't adhere to best-practices and shuts them down" — e.g. instances not in an auto-scaling group | Configuration conformance |
| **Doctor Monkey** | "taps into health checks that run on each instance as well as monitors other external signs of health (e.g. CPU load) to detect unhealthy instances", removes them from service, then terminates them | Health detection (not injection) |
| **Janitor Monkey** | "ensures that our cloud environment is running free of clutter and waste. It searches for unused resources and disposes of them" | Resource hygiene (not injection) |
| **Security Monkey** | "an extension of Conformity Monkey. It finds security violations or vulnerabilities, such as improperly configured AWS security groups, and terminates the offending instances" — also checks SSL and DRM certificate validity | Security conformance |
| **10-18 Monkey** | "detects configuration and run time problems in instances serving customers in multiple geographic regions, using different languages and character sets" | Localisation/internationalisation |
| **Chaos Gorilla** | "similar to Chaos Monkey, but simulates an outage of an entire Amazon availability zone" | Availability-zone failure |

Three corrections this roll-call forces, all of which appear in circulating summaries:

1. **Not everything called a "Monkey" injects faults.** Doctor Monkey, Janitor Monkey, Conformity Monkey, Security Monkey and 10-18 Monkey *detect and remediate*; they are resilience **tooling**, not chaos **experiments**. Only Chaos Monkey, Latency Monkey and Chaos Gorilla inject failure in the original post.
2. **Chaos Kong — the region-failure tool — is not in the 2011 post.** ⚠ I searched for a primary Netflix announcement and could not retrieve a datable primary source for Chaos Kong in this pass; treat "Chaos Kong" as a *later, well-known* member of the family whose introduction date I am **not** asserting. The region-failure concept is covered in this guide's taxonomy (§5.7) on its own merits.
3. **The post explicitly describes the Army as partly aspirational** — "Parts of the Simian Army have already been built, but much remains an aspiration -- waiting for talented engineers to join the effort and make it a reality" ✅. Anyone citing the 2011 post as evidence that Netflix ran an eight-tool fault-injection platform in July 2011 has read it backwards. It is a *roadmap with a philosophy attached*, and the philosophy is the part that lasted.

There is also a date trap adjacent to this one. **Chaos Monkey the tool predates the Simian Army post** — the post says "This was our philosophy when we built **Chaos Monkey**", in the past tense ✅ — but I did **not** find a datable primary source for the tool's own first release in this pass, and I decline to assert a year. ⚠ The commonly-quoted figure for Chaos Monkey's inception could not be verified today; if you need it, go to the Netflix Tech Blog archive directly.

### 2.3 The Principles of Chaos — the Current Text and a Correction

**A correction to this repository's own sibling guide, stated plainly and up front.** [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §7.2 presents the Principles of Chaos as **four** principles and omits **'Minimize Blast Radius'** from its numbered list. **The current canon has five.** I extracted [principlesofchaos.org](https://principlesofchaos.org/) live on **17 September 2026** and the "ADVANCED PRINCIPLES" section carries exactly five headings, of which **"Minimize Blast Radius" is the fifth** ✅. The sibling guide has condensed a five-principle document into four and, worse, dropped the one principle whose absence is the single most common cause of chaos-programme failure in a regulated estate. This guide restores it, and §8 treats it as the load-bearing safety principle. This is the "sibling summarised, this guide verifies" correction this repository values.

**Document identity and revision history.** The page carries the line **"Last Update: 2019 March"** ✅ with a link to the specific change — `github.com/chaos-eng/chaos-eng.github.io/pull/23/files` ✅ — which is the repository that generates the site (`chaos-eng/chaos-eng.github.io`, Hugo-generated, 438 stars, 159 commits at the time of my extraction on 17 September 2026 ✅). So:

- **Ownership:** the document is **community-owned**, not Netflix-owned. It lives in a `chaos-eng` organisation repository ✅. The site itself invites the reader into "the ongoing discussion of the Principles of Chaos and their application in the **Chaos Community**" with a link to the `chaos-community` Google Group ✅.
- **Revision status:** the last substantive update was **March 2019** ✅ — which means the canon's text is now **more than seven years old** while the tooling landscape around it has churned violently (§6 documents multiple abandoned or absorbed projects). The principles are still quotable and still largely right; they are also **not maintained**, and treating an unmaintained 2019 document as a live standard is a category error worth naming.
- **Translation:** the repository documents a mechanism for adding translations via ISO 639-1 two-letter content files ✅, so the document has a broader reach than a single English page.

**The definition, verbatim** ✅: *"Chaos Engineering is the discipline of experimenting on a system in order to build confidence in the system's capability to withstand turbulent conditions in production."*

**The experimental method, verbatim** — the page describes four steps under "Chaos in Practice" ✅. Note that this is a *different four* from the five advanced principles, which is the source of the four-versus-five confusion in circulation:

1. "Start by defining 'steady state' as some measurable output of a system that indicates normal behavior."
2. "Hypothesize that this steady state will continue in both the control group and the experimental group."
3. "Introduce variables that reflect real world events like servers that crash, hard drives that malfunction, network connections that are severed, etc."
4. "Try to disprove the hypothesis by looking for a difference in steady state between the control group and the experimental group."

**The five advanced principles, verbatim** ✅ — and the site is explicit that these "describe an ideal application of Chaos Engineering", with "the degree to which these principles are pursued strongly correlates to the confidence we can have in a distributed system at scale":

| # | Principle (verbatim heading) | The document's own justification (abridged, quoted) |
| --- | --- | --- |
| 1 | **Build a Hypothesis around Steady State Behavior** | "Focus on the measurable output of a system, rather than internal attributes of the system… Chaos verifies that the system does work, rather than trying to validate how it works." |
| 2 | **Vary Real-world Events** | "Chaos variables reflect real-world events. Prioritize events either by potential impact or estimated frequency." |
| 3 | **Run Experiments in Production** | "Systems behave differently depending on environment and traffic patterns… sampling real traffic is the only way to reliably capture the request path." |
| 4 | **Automate Experiments to Run Continuously** | "Running experiments manually is labor-intensive and ultimately unsustainable." |
| 5 | **Minimize Blast Radius** | "Experimenting in production has the potential to cause unnecessary customer pain. While there must be an allowance for some short-term negative impact, it is the responsibility and obligation of the Chaos Engineer to ensure the fallout from experiments are minimized and contained." |

Note that principle 3 and principle 5 are **in direct tension within the same document**, and the document knows it — principle 5 opens by conceding that "there must be an allowance for some short-term negative impact" ✅. That concession is the crack that §10's regulated-institution analysis widens into a structural problem.

### 2.4 The Book Lineage

The standard work is **"Chaos Engineering: System Resiliency in Practice"** by **Casey Rosenthal and Nora Jones**, published by **O'Reilly Media**, **6 April 2020**, **308 pages**, **ISBN 1492043834 / 9781492043836** ✅. That bibliographic record comes from the **Google Books catalogue entry** for the title ✅ — a bibliographic database, and therefore better evidence than a publisher marketing page but not the physical book itself. The same record notes the authors "pioneered the discipline while working together at Netflix" ✅ and that the volume is partly a **multi-author contributed collection** — chapters from practitioners at Google, Microsoft, Slack and LinkedIn among others, organised into parts including "Principles in Action", "Human Factors", "Business Factors" and "Evolution" ✅. The "Human Factors" part is the one a banking reader should go to first: it is where the book stops being a tooling manual.

Two adjacent works are commonly cited alongside it. **A caveat first:** my searches for their bibliographic records **returned empty result sets** on this pass ✅ (an empty search, not a negative finding), so I mark both ⚠ rather than asserting publisher dates:

- ⚠ **"Learning Chaos Engineering"** by **Russ Miles** (O'Reilly) — widely cited as the practical, hands-on companion. I could **not** verify its publication year or ISBN today; the searches returned nothing.
- ⚠ **"Chaos Engineering"** as a **standalone earlier O'Reilly short-form/report title** by Casey Rosenthal — I saw the title in secondary listings but could not retrieve a datable primary publication record for a distinct volume separate from the 2020 book. If you need this for a citation, go to the O'Reilly catalogue directly; I am explicitly **not** asserting it exists as a separate book.

**What the lineage actually shows.** The canon crosses three genres and each adds something the previous lacked: the **2011 blog post** gave the practice its *philosophy and its origin myth*; the **Principles document (origin commonly dated 2015 ⚠ — I could not verify the first-publication year at a primary source this pass; last revised March 2019 ✅)** gave it a *definition and a claim to being a discipline* rather than an engineering habit; and the **2020 book** gave it *human factors, business justification and organisational evolution*. A programme that quotes only the first has a stunt. One that quotes only the second has a checklist. The third is the one that answers "why should the board fund this", which is §10's question.

### 2.5 From Internal Culture to Market

The practice crossed from one company's internal culture to a market through three identifiable vectors, and it is worth being precise about which is which:

1. **The document itself.** The Principles site is community-owned and open to translation contributions ✅, and links into an open discussion forum ✅. That is the *normative* vector — a definition anyone can adopt.
2. **Practitioner migration.** The clearest verified example: **Nora Jones** — co-author of the standard work — left Netflix for **Slack** and **Jet.com**, keynoted **AWS re:Invent in November 2017** on the subject "helping kick off the Chaos Engineering movement we see today", and went on to found **Jeli** and the **learningfromincidents.io** movement ✅ (all from the Google Books author biography of the 2020 book ✅). The practice travelled in people, not in code.
3. **Tooling commercialisation.** The commercial platforms in §6.2 — Gremlin, Harness, Steadybit, and the managed cloud services in §6.4 — are the *market* vector, and they are exactly where the source-discipline war of §14 begins: a vendor blog is a **vendor's claim about its own tool** ✅, not evidence about the world. The most striking datum in this guide's tooling section is that **Harness renamed its Chaos Engineering module to "Resilience Testing"** and folded load testing and DR testing into it ⚠ (third-party summary of a vendor change) — which is the market's own admission that standalone chaos tooling does not sell as a category on its own.

**The honest read on "adoption".** I could not verify any credible, methodologically-sound measurement of chaos-engineering adoption rates in this pass, and I am not going to cite a vendor survey as if it were one. ⚠ Treat every "X% of enterprises now practise chaos engineering" figure you encounter as a **vendor claim about the market** unless it names its sample and method.

## 3. The Principles, Examined

The canon is a good document and I am not going to pretend otherwise. But reciting five headings is not a guide — the value is in knowing which of them have aged well, which are contested, and which rest on an assumption that may not hold in your estate at all. This section takes them in the order the document lists them.

### 3.1 The Five, One at a Time

**Principle 1 — "Build a Hypothesis around Steady State Behavior." Aged well; arguably the whole discipline.** The document's sharpest sentence is here: "Chaos verifies that the system *does work*, rather than trying to validate *how it works*" ✅. That is a behavioural, black-box framing, and it is why chaos engineering survives architecture changes that invalidate white-box tests. The contested edge is *what counts as measurable output*. The document suggests "throughput, error rates, latency percentiles" ✅ — all of which are **user-visible aggregate** signals. In a bank, the steady state you actually care about is often a **business** signal (settlement completion, payment instruction acceptance, reconciliation break count), not an infrastructure signal. If your steady state is defined at the infrastructure layer and your risk lives at the business layer, you can pass every experiment and still be one batch window from a regulatory breach. My rule: define steady state at the **highest layer you can actually measure continuously**.

**Principle 2 — "Vary Real-world Events." Aged moderately; changed by the estate it lands in.** The document says to "prioritize events either by potential impact or estimated frequency" ✅ and to consider hardware failures, software failures and non-failure events like traffic spikes ✅. This is sound but **incomplete for a regulated estate**, where the events that matter most are frequently *operational and human* (§5.9) — a bad change, a certificate that nobody rotated, an expired credential in a third-party integration, a batch job that overran its window. The canon's event list is a *hardware-centric 2015 list*. Widen it deliberately.

**Principle 3 — "Run Experiments in Production." Contested. The discipline's distinctive claim and its biggest liability.** Analysed separately in §3.2.

**Principle 4 — "Automate Experiments to Run Continuously." Aged worst of the five in a risk-averse context.** The document's justification is purely economic: "Running experiments manually is labor-intensive and ultimately unsustainable" ✅. That is true. But the argument quietly assumes that the *marginal cost of a permanent fault injector* is near zero and the *marginal risk* is constant. Both assumptions fail as the blast radius grows. A continuously automated experiment that was safe when it ran against 1% of a stateless service is not automatically safe after the service acquired a shared database dependency — and a continuously running injector is precisely the thing that will not notice. Analysed in §3.3.

**Principle 5 — "Minimize Blast Radius." The most important principle in the document, and the one most often dropped.** Its drafting is unusually frank: "While there must be an allowance for some short-term negative impact, it is the responsibility and obligation of the Chaos Engineer to ensure the fallout from experiments are minimized and contained" ✅. Two things follow. First, "responsibility **and obligation**" is the strongest normative language in the whole document — this is not advice, it is a duty. Second, the concession "there must be an allowance for some short-term negative impact" ✅ is the exact sentence that makes a regulated estate uncomfortable, because in a regulated estate short-term negative impact may be an **operational incident** with reporting duties attached (§10.5). This is the principle the sibling guide omitted; it is the one that should be printed above the desk.

### 3.2 Run in Production — the Claim That Collides with Regulation

The document's argument for production is genuinely strong and should not be waved away ✅:

- "Systems behave differently depending on environment and traffic patterns."
- "Since the behavior of utilization can change at any time, sampling real traffic is the only way to reliably capture the request path."
- "To guarantee both authenticity of the way in which the system is exercised and relevance to the current deployed system, Chaos strongly prefers to experiment directly on production traffic."

Each of those sentences is correct about *engineering*. The problem is that each is also **correct in a way that conflicts with a control regime**:

1. **"Sampling real traffic is the only way to reliably capture the request path."** In a bank, "real traffic" is customer payment instructions. You are proposing to route real, irrevocable, regulated financial instructions through a path you have deliberately degraded. That is not a philosophical objection; it is a **settlement-risk** objection. Where a payment is irrevocable on acceptance, a failed request path is not a latency blip — it is a reconciliation break with a counterparty.
2. **"Systems behave differently depending on environment and traffic patterns."** True, and it implies the honest corollary the document does not state: **staging can prove almost nothing about systemic resilience**, only about mechanism correctness (§8.3). A bank that tests only in staging should say so plainly rather than presenting staging results as resilience assurance.
3. **"Relevance to the current deployed system."** Production is current by definition; staging drifts the moment it is provisioned. This is the strongest argument for production and it is why a "staging-only" chaos policy quietly decays into a fiction.

**The tension, stated precisely:** chaos engineering's third principle is a *probabilistic* claim (real traffic captures the real path better than a rehearsed one) and the regulated estate's constraint is a *deontic* one (a regulated process must be controlled, evidenced, and sometimes notified before it is executed). These are different kinds of statements — one about accuracy, one about permission — and no amount of engineering skill makes them commensurable. §10.5–10.6 is where this is resolved rather than argued.

### 3.3 Automate Continuously — the Cost and the Permanent Fault Injector

The case for continuous automation is real and I accept it at small blast radius: manual experiment execution does not scale, and the discipline's benefit compounds when experiments run on every deploy. The case against it in a regulated estate is three-part, and only the first is obvious:

- **The risk is not constant.** A permanent injector is a permanent production dependency. It has its own failure mode: the injector that fails *while injecting* is an experiment that cannot be aborted (§8.2's exact scenario).
- **The failure is silent when it matters.** An automated experiment produces a pass/fail signal. The thing that changes — a new shared dependency, a new synchronous call, a reduced replica count — does not necessarily trip the pass/fail signal on the day it changes; it trips it three weeks later when the injector's normal fault finally interacts with it. This is a **drift-blindness** problem, and it is why the automated experiment must be **re-scoped on every architecture change**, which is a human review, not an automation.
- **Continuous injection erodes the "event" status of the experiment.** A one-off game day is a visible, scheduled, communicated event with a change record. A continuous injector is background infrastructure. In an institution where §10.7 shows that the *evidence trail* is the deliverable, degrading an experiment from event to background process may destroy exactly the artefact the regulator wants to see.

**Defensible middle ground:** continuous automation on the *low* end of the fault spectrum (instance-level, high-redundancy, no state), and scheduled, witnessed, recorded experiments on everything higher. That is not a compromise of the principle; it is the principle applied with principle 5 intact.

### 3.4 The Load-Bearing Assumption — Do You Have a Steady State?

The whole method rests on an assumption the canon never defends: **that you can trustworthily define "normal" for your system.** §7 makes it a prerequisite; here is why it is a philosophical problem, not a tooling one.

- **What if the system is already degraded?** If your baseline captures a system that is silently running on a degraded path — a failover that never failed back, a replica that has been dead for a month, a cache that has been a miss for a week — then your "steady state" is the *degraded* state, and your experiment proves that you can survive a fault **on top of** an existing fault. It will pass. That pass is worthless, and worse, it is false assurance of the kind §10.7 warns about.
- **What if the steady state is a fiction the dashboards tell?** This is the harder version. If your availability metric is computed from a health endpoint that the load balancer also uses, you are measuring the load balancer's opinion, not the customer's experience. If your error rate excludes the retries the client library performed silently, you are measuring the server's optimism. If your latency percentile is computed over *successful* requests only, a partial outage can *improve* your p99.
- **What if the steady state is only knowable through a business proxy with a lag?** In banking this is the common case: reconciliation breaks, settlement status, end-of-day batch completion. These are the true steady-state signals and they arrive *after* the experiment window has closed. An experiment whose only meaningful signal arrives at 02:00 the next day is an experiment whose abort condition cannot use that signal.

**The practical consequence — and this is the section's real deliverable:** before you run a single experiment, you must **audit the steady-state definition itself**. Concretely, five questions: name the signal; name who computes it; name what it excludes; name the lag between cause and signal; and state whether it is a *system* signal or a *proxy for a system* signal. If you cannot answer those five questions for a candidate system, that system is not ready for chaos engineering, and running the experiment anyway is anti-pattern #3 in §13.

## 4. The Experiment Design

This is the section that separates the discipline from the stunt. Everything here is craft, and craft is transferable: the same nine steps work for a 1%-of-pods kill in a Kubernetes cluster and for a 200 ms latency injection against a downstream payment gateway.

### 4.1 The Nine Steps of an Experiment

| Step | What you produce | The failure mode if you skip it |
| --- | --- | --- |
| **1. Frame the hypothesis** | A falsifiable sentence naming the variable, the expected outcome, and the property under test. | You run a fault and learn nothing structured — anti-pattern #1. |
| **2. Define the steady state** | The metric(s), the source, the baseline band, the observation window. | You cannot tell effect from noise — §3.4, anti-pattern #3. |
| **3. Scope the variables** | Exactly one variable (or one coherent bundle), exactly which targets, exactly which environment. | Two variables change; you cannot attribute the result. |
| **4. Size the blast radius** | A percentage, a target set, a duration cap, and the "worst realistic case" written down. | The blast radius is whatever happens, rather than whatever you chose. |
| **5. Define abort conditions and the kill switch** | Machine-checkable thresholds and the tested stop mechanism. | §8.2's scenario: an experiment you cannot stop. |
| **6. Run pre-start safety checks** | The checklist in §8.5, signed. | You discover a missing dependency while injecting into it. |
| **7. Observe through the window** | Screens, the control room, the named watchers, the timestamped notes. | The findings are recollections rather than records. |
| **8. Analyse against the hypothesis** | Confirm/refute, with the evidence attached and the confounders named. | You report "it seemed fine", which is not a finding. |
| **9. Feed findings to remediation** | A ticket per finding, with an owner and a due date; then re-run the experiment. | The programme produces knowledge nobody acts on — anti-pattern #4. |

Note the shape: **steps 1–6 are all preparation.** In a mature programme, more than half the effort is spent before anything is injected, and that ratio is the single best indicator of whether a programme is real.

### 4.2 The Steady-State Definition and Its Tolerance

"Define steady state" is the instruction most often discharged badly. A usable definition has four parts and, critically, a **tolerance band** — because "unchanged" is not achievable against real traffic and demanding it makes every experiment fail for the wrong reason.

1. **The signal.** Named precisely, with its unit and its source system. Example: *payment instruction acceptance rate, as computed by the API gateway over 1-minute buckets.*
2. **The baseline.** A measured pre-experiment window, not a remembered number. **Minimum 30 minutes of clean data immediately preceding** the experiment, with the sample size stated.
3. **The tolerance band.** E.g. *acceptance rate ≥ 99.5% and p99 latency ≤ 400 ms*, established from the baseline plus the service's own SLO — and note here the cross-reference: the SLO and error-budget machinery is owned by [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2; **the experiment's tolerance should be derived from the error budget, not invented for the experiment.** If the experiment consumes more of the error budget than the budget permits, the experiment is not permitted.
4. **The lag caveat.** How long after the cause the signal actually moves. For synchronous request/response signals this is seconds; for batch and reconciliation signals it may be hours, which caps what the experiment can prove within its window (§3.4).

**The band matters more than the value.** An experiment with a band can *fail informatively* — you learn that the system deviated outside tolerance, which is a finding. An experiment without a band can only produce an opinion.

### 4.3 Abort Conditions and the Kill Switch

Two distinct things, routinely conflated, and conflating them is how experiments become incidents.

| | **Abort condition** | **Kill switch** |
| --- | --- | --- |
| **What it is** | A threshold that says *stop now* | The mechanism that *does* the stopping |
| **Who decides** | Nobody — it is machine-evaluated or a pre-agreed trip line | A human, or the automation, executing a pre-rehearsed action |
| **Examples** | Error rate > 2× baseline for 60 s; p99 latency > 1.5× tolerance; any 5xx on a specific critical endpoint; a single reconciliation break; the customer-impacting alert firing | A CLI command, a feature flag flip, a Kubernetes annotation removal, a `Ctrl-C` on the injector, a rollback of the experiment manifest |
| **Tested how** | Reviewed against the metric definitions before the run | **Rehearsed on a passing experiment before it is ever needed on a failing one** |

**The rule that follows:** if you have never *executed* your kill switch, you do not have a kill switch — you have an intention. §8.2 makes this concrete. And note the abort condition's most important property: it must be **checkable by someone who is not the experiment's author**, because the author has a stake in the experiment completing.

### 4.4 The Worked Experiment Design — an Adaptable Template

Below is a complete, generic-but-concrete design. Take it as a template; every field is meant to be filled with your own specifics. It uses two variants so you can see how the same frame flexes.

**EXPERIMENT CE-007 — pod termination (instance-level, stateless service)**

- **Hypothesis.** *"If 25% of the `payment-instruction-api` pods in one availability zone are terminated simultaneously, the service will maintain its steady state, because the remaining pods in three zones absorb the traffic within the 30-second connection-drain window."* Property under test: **instance-loss absorption without customer-visible degradation.**
- **Steady state.** Signal: instruction acceptance rate (API gateway, 1-min buckets) **and** p99 end-to-end latency (service mesh, per-service). Baseline: 30-minute pre-window, minimum 20,000 instructions sampled. Tolerance: **acceptance ≥ 99.5%**; **p99 ≤ 400 ms**; **zero** 5xx on the settlement-confirmation endpoint. Lag caveat: synchronous — signal moves within ~10 s.
- **Variables.** Exactly one: pod termination. Not a node drain, not an AZ loss, not a simultaneous latency injection. Target set: pods labelled `app=payment-instruction-api` in a single zone, chosen explicitly (not at random) so the blast radius is knowable in advance.
- **Blast radius.** 25% of pods in one zone (target: 3 of 12 pods). Duration: single termination event, no repeat for 15 minutes. Worst realistic case written down: **if the remaining pods' autoscaling is misconfigured, up to 25% of that zone's traffic fails for up to 90 s**, impacting an estimated **X% of total instruction volume** — the X must be computed from real traffic distribution, not guessed.
- **Abort.** Trip if acceptance rate < 99.5% for **60 consecutive seconds**, or p99 > 600 ms for 60 s, or any 5xx on settlement-confirmation, or the on-call engineer calls it. **Auto-abort** configured in the injector where the tool supports it (§6); manual abort always available in parallel.
- **Kill switch.** Removal of the chaos manifest / annotation; rehearse it on a **0% experiment** before this one. Named operator, named backup.
- **Pre-start checks.** §8.5, all green: replicas healthy across ≥ 3 zones, no in-flight deployment, no active change freeze, no month-end/quarter-end window, autoscaling policy verified as *enabled and warm*, PDB/anti-affinity verified present, control room staffed, notification sent.
- **Observation window.** 30 minutes total: 10 min pre-baseline, 15 min post-injection observation, 5 min cooling and confirmation.
- **Analysis.** Compare each metric against the band, control group vs experimental group where the topology permits; record every confounder (a concurrent deploy, a batch run, an unrelated alert). Explicit verdict: **hypothesis confirmed / refuted / inconclusive**, and — the field people skip — **what we now believe that we did not believe before.**
- **Artefacts.** The design document (this artifact), the injector configuration at a pinned commit, the dashboards (exported, timestamped), the abort log with timestamps, the findings register entries with owners, the re-run result.

**EXPERIMENT CE-008 — downstream latency injection (dependency-level)**

Same nine steps, two fields differ and both matter: the **variable** becomes *a 200 ms added latency on the egress path to the settlement gateway for a single caller*, and the **hypothesis** must name which degradation mechanism you expect to engage — *"the client timeout of 300 ms and the circuit breaker at 50% error rate will both engage, and the caller will degrade gracefully to the queued path rather than failing."* Property under test: **graceful degradation**, not absorption. Note the differing abort trip: here the risk is not volume loss but **timeout-induced retry storms**, so the abort condition should watch *outbound request rate* as well as acceptance rate — a retry storm shows up as a **rise in outbound calls** while acceptance is still nominally fine. That single extra metric is the difference between a latency experiment that is safe and one that takes out a shared dependency through retry amplification.

**Template, condensed** — copy this block into your own design document:

```
EXPERIMENT ID / TITLE:  |  PROPERTY UNDER TEST: absorption | degradation | detection | recovery
HYPOTHESIS (one falsifiable sentence):
STEADY STATE:  signal | source | baseline window | tolerance band | lag
VARIABLE:      exactly one, named, with its magnitude and duration
BLAST RADIUS:  % / target set / duration cap / worst realistic case + exposure estimate
ABORT:         machine-checkable trips, with dwell times, + human abort
KILL SWITCH:   mechanism | operator | backup | LAST REHEARSED ON:
PRE-START:     checklist ID and result, signed
WINDOW:        pre | observation | cooling, with timestamps
NOTIFICATION:  who was told, when, through which channel
FINDINGS:      register IDs, owners, due dates  |  VERDICT: confirmed | refuted | inconclusive, + belief delta
ARTEFACTS:     design doc | pinned config | exported dashboards | abort log | evidence pack
```

## 5. The Failure Taxonomy

A catalogue of faults worth injecting. For each: the **mechanism**, the **realistic blast radius**, what it **actually proves** that nothing else does, and the **tooling** that can inject it (§6 for the tool statuses, §6.1 for the dead-project rule). Two notes on reading this table. First, "proves" is deliberately narrow — a fault class that proves nothing a cheaper test already proves does not belong in your programme. Second, this taxonomy is ordered roughly by increasing nastiness in a regulated estate, which is also the order I recommend adopting it.

| # | Fault class | Mechanism | Realistic blast radius | What it PROVES that nothing else does | Injectors |
| --- | --- | --- | --- | --- | --- |
| 1 | **Instance / host termination** | Kill a VM, pod or container; drain a node | One instance (small) → 25–33% of a service's capacity in one zone (medium) | That your redundancy, connection draining, health-check eviction and autoscaling actually work *together*, at the moment of loss, with real in-flight requests. A static replica count proves none of this. | Chaos Monkey, Chaos Mesh (`PodChaos`), LitmusChaos pod-delete, Gremlin, cloud-native APIs |
| 2 | **Network latency** | Add delay on an egress/ingress path (e.g. +200 ms) | One caller→dependency edge (small) → a whole service's outbound traffic (large, via retry amplification) | That your **timeouts, retry budgets and circuit breakers are tuned to the latency you actually get**, not the latency you assumed. Latency is the fault class that finds retry storms. | Toxiproxy, Chaos Mesh (`NetworkChaos`), ChaosBlade, Gremlin, AWS FIS network actions |
| 3 | **Packet loss / corruption** | Drop or corrupt a % of packets | One path (small–medium) | That TCP-level retransmission and your application's own error handling do not compound into a stall. Loss is qualitatively different from latency: loss *lengthens* effective latency unpredictably. | Toxiproxy, Chaos Mesh, ChaosBlade |
| 4 | **Network partition** | Sever connectivity between two sets of nodes | Two islands (large — this is the fault that exposes split brain) | That your system's **quorum and consistency choices hold under partition** — the CAP question, tested rather than argued. Most partition tests produce a *correctness* finding, not an availability one. | Chaos Mesh (`NetworkChaos` partition), LitmusChaos, ChaosBlade, TC/iptables directly |
| 5 | **DNS failure** | Break resolution, or serve stale/wrong records for one name | Anything that depends on that name (potentially the whole estate) | That your resolution path has **no single point of failure and a sane TTL/cache story**. DNS is the fault class with the largest blast-radius-to-effort ratio and the one most often excluded from "safe" experiments by mistake. | Chaos Mesh (DNSChaos), ChaosBlade, resolver-level manipulation |
| 6 | **Dependency / downstream failure** | Return errors, malformed responses, or refuse connections from a named dependency | The caller and everything upstream of it (medium–large) | That **graceful degradation** engages: the fallback path, the cached path, the queued path. This is the single most valuable experiment class for a bank, because banks are integration-heavy and every integration is a dependency (§11). | Toxiproxy, WireMock fault injection, Chaos Mesh, the dependency's own stub |
| 7 | **Resource exhaustion — CPU** | Pin CPU / saturate cores | One container → one node (small–medium) | That your **latency SLOs hold at the CPU level you can actually reach in a busy period** — not at your 30% steady-state utilisation. Finds the headroom assumption. | Chaos Mesh (`StressChaos`), LitmusChaos, Gremlin |
| 8 | **Resource exhaustion — memory** | Allocate to a limit, trigger OOM | One container (small, unless the container is shared) | That your **OOM behaviour is the behaviour you designed** — a clean restart with connection re-establishment, rather than a silent partial failure. Also proves that your memory limits are set where you think. | Chaos Mesh (`StressChaos`), LitmusChaos, Gremlin |
| 9 | **Resource exhaustion — disk / file descriptors** | Fill a volume; exhaust FDs | One node or one process (small), but with **high tail risk for stateful systems** | That your logging, spooling and connection handling **fail loudly and recover**. Disk-full is the classic silent corruptor: a system can appear up while writing nothing. | Chaos Mesh (`IOChaos`, disk fill), LitmusChaos, Pumba, Gremlin |
| 10 | **State faults — data corruption** | Write garbage into a cache, a replica, a spool file | **Unbounded if you get it wrong.** Cache-only corruption is contained; primary-store corruption is not an experiment. | That your **checksums, validation-on-read and cache-invalidation are real**. This class has the worst blast-radius-to-control ratio in the taxonomy and should be attempted last, in a staging environment first, with restoration proven. | Chaos Mesh (`IOChaos`), custom injectors, application-level toggles — **never** against an unreplicated primary |
| 11 | **State faults — stale cache** | Serve an old value for a known key | Contained to the cache's consumers (small–medium) | That your **cache-invalidation story works and your reads tolerate staleness within the business's actual tolerance** — which is a business question (is a stale balance acceptable for 5 seconds? 5 minutes?) more than a technical one. | Mock/queue-level injectors, cache TTL manipulation, dependency stubs |
| 12 | **State faults — split brain** | Partition a leader from its replicas; allow both to accept writes | **Large and dual-write dangerous** | That your **fencing, leases and quorum actually prevent concurrent writers** — or that they do not. If a bank has one experiment that must never be improvised, it is this one (§8, §11). | Chaos Mesh partition, cluster-level partition, network isolation |
| 13 | **TIME faults — clock skew** | Skew the system clock on one or more nodes (±seconds to ±minutes) | Contained to the affected nodes but **wide in effect** (see below) | Nothing else in this table. Clock skew is the uniquely nasty class because it is invisible to application logic that assumes monotonic, agreed time. | LitmusChaos time chaos, `libfaketime`, NTP manipulation, Chaos Mesh `TimeChaos` |
| 14 | **Zone failure** | Remove or isolate an entire availability zone | 1/N of capacity (large) | That your **multi-AZ topology is genuinely symmetric** — that the load balancer, the database failover and the autoscaling groups all have the same view of the surviving zones. Chaos Gorilla's stated purpose in the 2011 post was exactly this: to "verify that our services automatically re-balance to the functional availability zones without user-visible impact or manual intervention" ✅. | Cloud provider zone-failure actions, Chaos Gorilla (historically), managed FIS services |
| 15 | **Region failure** | Lose a whole region | **Half the estate or more.** | That your **DR posture is not aspirational** — but note carefully: a region failure "experiment" is almost always a *DR drill* in disguise (§1.2), because you will fail over by design. It proves failover mechanics, not absorption. | DR tooling, DNS/global traffic management, provider region-failure actions |
| 16 | **Certificate / credential expiry** | Expire a TLS cert, rotate a credential, revoke a token | Everything terminating or authenticating through it (potentially the whole estate, **all at once**) | That your **rotation and renewal are automated and monitored**, rather than depending on a calendar reminder. This is the fault class most likely to cause a real incident with no chaos programme at all — which is exactly why it is worth rehearsing. | Expiry injection, short-lived cert issuance, credential revocation, mTLS test harnesses |
| 17 | **Accidental / human faults** | Bad config push, a bad change window, a mis-scoped permission, an overran batch | Whatever the change touched — **historically the largest outages in the industry** | That your **change control and rollback are real** — that you can detect a bad change, attribute it, and reverse it quickly. This is the class the canon's event list barely mentions (§3.1, principle 2) and the one that matters most in a bank. | Deployment rollback drills, permission reduction, feature-flag flips, canary-with-intent-to-revert |

**Why clock skew deserves its own paragraph.** Time faults sit at the intersection of three things that each independently assume a global, monotonic, agreed clock: **authentication** (TLS certificate validity windows; JWT `exp`/`nbf`/`iat` claims — a few minutes of skew can reject every token in the estate, or accept an expired one), **distributed tracing** (a trace whose spans are not causally ordered is not a trace; you lose the observability that §7 makes a prerequisite for the whole practice, so a clock-skew experiment can blind the experimenter), and **financial timestamps** (value dates, cut-off times, audit-trail ordering, and batch-window boundaries are *business semantics* built on wall-clock time). It is also the least-frequently-injected class in most programmes, because it produces confusing symptoms rather than clean failures. In a bank that is precisely why it is worth injecting: **a fault whose symptom is confusing is a fault you want to have met before your customers do.**

**Two closing rules for the taxonomy.** (1) **Adopt in order.** The numbered order above is a defensible maturity sequence for a regulated estate; jumping to #10 or #12 in production is how programmes get shut down permanently. (2) **Every row needs a steady state and an abort trip** — and for rows 10–12 that means the abort trip must watch a *correctness* signal (a checksum, a reconciliation count, a consistency check), not an availability signal, because the failure mode of a state fault is a system that is **up and wrong**. §7 explains why availability-only observability makes these experiments un-runnable.

## 6. The Tooling Landscape

### 6.1 How to Read This Section — and the Dead-Project Rule

Every entry below was checked on **17 September 2026** unless stated otherwise, and each carries its **maintenance status, the evidence for that status, and the date**. The evidence is the thing to look at, not the label: for repositories I read the **last commit date, the latest release, the licence file and the presence of a build system that still exists** — because a project's marketing page and its commit history routinely tell different stories.

**The dead-project rule, stated plainly: a dead or archived project must never be presented as a live option.** This field is unusually full of abandoned projects whose blog posts and documentation sites remain authoritative-looking years after the last commit — and a bank that adopts one of them inherits an unmaintained fault injector with production access. Three signs I treat as red flags regardless of what the docs site says: **(a)** a CI badge pointing at a CI service that has been discontinued; **(b)** no tagged release in more than two years; **(c)** a repository path that 404s because the original organisational home was dissolved. All three appear in the table below — in real entries.

**Source discipline for this section.** A vendor's product page is a **vendor's claim about its own tool** ✅ and is evidence only about what the vendor wants you to believe. A **repository page is primary evidence** — commit dates, releases, licences and archive banners are facts about the artefact. Where I could only find a **third-party directory listing** (funding trackers, company-profile aggregators), I say so and downgrade the claim accordingly: those sites are notoriously incomplete about funding and stale about status.

### 6.2 The Commercial Platforms

| Platform | What it injects | Deployment | Licensing | Status (verified) |
| --- | --- | --- | --- | --- |
| **Gremlin** | Host- and container-level attacks: resource (CPU/memory/disk), network (latency/loss/DNS), state (time-travel, process kill), plus dependency discovery and DR validation. Now positioned as "Enterprise Reliability Management & Resilience Testing" with a "Foresight AI" (Gremlin.ai) product and aggregate "reliability scores". | Agent-based (installed on hosts) plus a central control plane; covers bare metal, on-prem, multi-cloud and serverless per its own page. | Commercial SaaS, subscription. | **Active** ✅ — product page live and repositioned as of 17 Sep 2026; site claims "4 of the 5 largest US banks" and names RBC, Prudential, Capital One, Visa among customers (all **vendor claims about its own customer base** ⚠). ⚠ Funding/ownership: third-party trackers report **$26.8M across 2 rounds** ⚠ — a directory figure I could **not** corroborate against a primary filing, and which looks low for a company at this stage, so treat any funding number here as **unverified**. ⚠ I also could not verify any acquisition; no acquisition is asserted. |
| **Harness Chaos Engineering** | Kubernetes, container, network, HTTP, JVM and infrastructure faults, now bundled with load testing and DR testing. | Delivered inside the Harness software-delivery platform; supports Kubernetes-native and cloud targets. | Commercial SaaS (freemium tier historically); the platform has an open-source lineage but the chaos module is a commercial product. | **Active, but repositioned** ⚠ — Harness has **renamed and absorbed its Chaos Engineering module into "Resilience Testing"**, integrating chaos, load and disaster-recovery testing into a unified continuous approach ⚠ (this comes from a third-party summary of a vendor change, and the vendor's own release-notes site is the primary evidence for the current module name). ⚠ I could not verify the exact rename date. **Interpretation, clearly labelled as mine:** the market leader folding "chaos engineering" into a broader category is the strongest available evidence that chaos tooling does not survive as a standalone category — see §2.5. |
| **Steadybit** | Reliability experiments against services and infrastructure; automated issue discovery. | Agent/extension-based, self-hosted or SaaS. | Commercial. | **Active** ✅/⚠ — company website live and product marketed as of 17 Sep 2026 ✅; third-party funding data reports **€13M across 2 rounds, most recent a Series A in March 2024, "data as of 13 Jul 2026"** ⚠ — a **third-party summary**, so treat the figures as indicative. ⚠ I could not verify current headcount, ownership, or whether the self-hosted edition is still offered. |
| **Verica** | Continuous verification / chaos engineering with a security framing; historically associated with the practitioners who founded the discipline. | SaaS. | Commercial. | ⚠ **Status unverified — treat as a flagged negative finding.** The company domain (`verica.io`) **failed to resolve for automated extraction on 17 September 2026** ✅ — a retrieval failure that is *consistent with* a site that is down, parked or blocked, but is **not proof** of any of those. Third-party directories still list it with **$12.0M raised, latest a Series A in January 2022** ⚠. **Do not assume Verica is a live option without checking the domain yourself.** This is precisely the failure mode §6.1 warns about, and I am deliberately leaving it as a flagged negative rather than guessing. |
| **AWS / Azure / GCP managed services** | See §6.4 — treated separately because their status is a property of the cloud provider, not of a startup. | — | — | — |

### 6.3 The Foundation and Open-Source Projects

| Project | What it injects | Deployment | Licence | Status (verified 17 Sep 2026) |
| --- | --- | --- | --- | --- |
| **LitmusChaos** | A large experiment library (pod delete, container kill, network, CPU/memory stress, IO, DNS, time and more) published at ChaosHub. | Kubernetes-native; operator + CRDs, plus a ChaosCenter control plane. | Apache-2.0. | **Active** ✅ — **CNCF: accepted 25 June 2020, moved to Incubating on 11 January 2022** ✅ (verified on the CNCF project page). Repository shows **latest release 3.32.0 on 17 September 2026** ✅, **3,189 commits**, **121 releases**, **286 contributors**, **5.6k stars** ✅, and an adopter list including Motorola Solutions and Canonical ✅. Commit activity within the last 48 hours of my check ✅. **This is the healthiest open-source entry in this guide.** |
| **Chaos Mesh** | Kubernetes-native `Chaos` CRDs: `PodChaos`, `NetworkChaos`, `StressChaos`, `IOChaos`, `TimeChaos`, `DNSChaos`, `HTTPChaos`, and workflow orchestration. | Kubernetes-native operator + CRDs, with a dashboard UI. | Apache-2.0 ✅. | **Active** ✅ — repository shows **2,002 commits**, **7.9k stars**, **1.0k forks**, 121 watchers, and a `ROADMAP.md` ✅; **Chaos Mesh is a trademark of The Linux Foundation** ✅ per the repository itself. ⚠ **The PingCAP relationship I could not verify today** — my searches on current stewardship and maintainers **returned empty result sets**, and I am therefore not asserting who owns or primarily maintains it now, nor whether PingCAP remains the principal backer. The repository carries a `MAINTAINERS.md` and `GOVERNANCE.md`; **read those directly** rather than trusting any secondary account of the project's ownership. |
| **ChaosBlade** | A broad chaos experiment toolkit spanning OS, JVM, container and cloud layers. | CLI-driven, with a `chaosblade-box` management platform. | Apache-2.0 ✅. | **Active, and changing shape** ✅ — repository shows **6.5k stars**, **343 commits**, **42 releases**, **46 contributors**, and a **latest release `blade-ai-v0.7.0` on 25 August 2026** ✅ with the last commit **28 July 2026** ✅. Origin: the README describes it as "**An Alibaba open source** experimental injection tool" ✅. ⚠ **Important observation from the primary evidence:** the language breakdown is now **Python ~74%, TypeScript ~19%, Go ~3.6%** ✅ — a dramatic shift from a Go-based injection tool, tracking the recent `blade-ai` work (GitHub Actions workflows publishing to **npm and PyPI**, a TUI, an "AI" mode) ✅. **Interpretation:** the project is alive and actively being re-pointed at an AI-agent-flavoured use case. If you adopt it, pin a version and read the changelog — the velocity is real but the direction is moving. |
| **Chaos Toolkit** | Platform-agnostic experiments via drivers ("extensions") for Kubernetes, cloud providers, containers and more; YAML/JSON experiment definitions. | CLI (`chaos run experiment.json`), Python-based, works in CI/CD. | Apache-2.0 ✅. | **Active** ✅ — **2.0k stars**, **452 commits**, **206 forks** ✅. The current README documents installation via `uv tool install chaostoolkit` and requires **Python 3.12+**, and includes a modern `runtime.python.dependencies` feature where an experiment can declare its own Python version and dependencies and run isolated ✅ — evidence of maintenance well past the tool's original era. ⚠ I did **not** read the last-commit timestamp, only the current README state; treat "active" as resting on documentation evidence rather than commit-date evidence. |
| **Other Kubernetes-native projects** | Various narrow scopes. | — | — | ⚠ **Not individually verified this pass.** My searches for several candidate projects returned empty result sets. I am naming none of them rather than listing unverified names. |

### 6.4 The Cloud Providers' Managed Fault Injection

| Provider | Service | What it does | Status (verified 17 Sep 2026) |
| --- | --- | --- | --- |
| **AWS** | **AWS Fault Injection Service (FIS)** | "A fully managed service for running fault injection experiments to improve an application's performance, observability, and resilience"; explicitly "provides the controls and guardrails that teams need to run experiments in production, such as automatically rolling back or stopping the experiment if specific conditions are met"; pre-built scenarios from an "FIS Scenario Library"; **part of AWS Resilience Hub**. | **Generally available and active** ✅ — the AWS product page is live and describes the service in the present tense. ⚠ **Naming caveat, from primary evidence:** the page is titled **"AWS Fault Injection Service"** ✅ while the linked launch blog's URL slug still reads **`aws-fault-injection-simulator-...`** ✅ — i.e. **the service was formerly named "Fault Injection Simulator" and has been renamed**, and the old name persists in URL slugs across the web. Both names refer to the same service; use the current one in documentation. ⚠ I could not independently verify the rename date (my dedicated search returned an empty result set). |
| **Microsoft Azure** | **Azure Chaos Studio** | Managed chaos service. Two resource models: **Workspaces + Scenarios** (the current model — resource discovery, preconfigured Scenarios like *Compute Zone Down* and *DNS Outage*, and structured **Scenario reports**) and **Experiments (classic)** (the legacy model with **service-direct** faults via management API and **agent-based** in-guest faults). | **Split status — and this is the single most important nuance in this table** ✅. Per Microsoft's own documentation (page **last updated 09/06/2026** ✅): **Workspaces and Scenarios are in PUBLIC PREVIEW** — "Microsoft provides this preview 'as is'… it isn't covered by service-level agreements… This preview isn't meant for production use" ✅. **Experiments (classic) is the generally available model**, but Microsoft "**no longer develops features for Experiments (classic)** and considers only critical fixes, such as security updates, for backport" ✅. **Read that carefully before choosing Azure for a regulated programme:** the GA path is feature-frozen and the actively developed path is a preview explicitly not recommended for production. Also present: a **Chaos Studio AI plugin (`startchaos`)** usable as an **MCP server** for autonomous agents ✅. |
| **Google Cloud** | ⚠ **Not verified.** | — | ⚠ **Flagged negative finding.** I could **not** establish in this pass whether Google Cloud offers a managed fault-injection service, what it is called, or its status. My search queries on this returned **empty result sets**, and I am **not** going to name a product from memory. If you are building on GCP, verify this yourself against the current Google Cloud documentation before writing it into a design document or a regulatory submission. |

**A note on what a managed fault-injection service actually buys you.** Both verified services above expose the two things a bank needs and struggles to build: **pre-built, provider-maintained fault scenarios** and **provider-native stop conditions / rollback** ✅✅. What they do **not** do is resolve §10.8's question — when the provider runs the chaos on your behalf, the *scope, method and results* live in the provider's console under the provider's telemetry retention. **You own the regulatory obligation; the evidence is in someone else's account.** Azure's own documentation goes further than most, explicitly listing "**Compliance evidence**: Use Scenario reports to help support evidence requirements for operational resilience frameworks such as DORA" ✅ — which is a vendor claiming its artefact helps with a regulatory obligation, and §10.7 explains why you must test that claim against what your supervisor actually asks for rather than accepting it.

### 6.5 Proxy and Process-Level Tools

| Tool | What it injects | Deployment | Licence | Status (verified 17 Sep 2026) |
| --- | --- | --- | --- | --- |
| **Toxiproxy** (Shopify) | **TCP-level** faults via a proxy: latency, bandwidth, timeout, sliced transfers, and a `toxicity` parameter to make any toxic probabilistic. Works at the connection layer, so it is protocol-agnostic — which is why it is excellent for database and message-broker dependency testing. | A standalone proxy binary/container you point clients at; deterministic tampering under an HTTP API. | **MIT** ✅. | **Active** ✅ — **12.3k stars**, **752 commits**, **511 forks**, 505 watchers ✅. Primary datable fact from the README: **"We've been successfully using it in all development and test environments at Shopify since October, 2014"** ✅ — a ten-year-plus production record, which is the strongest maintenance signal in this whole section. Note the honest scope limit the README states itself: it is "made specifically to work in **testing, CI and development** environments" ✅ — **it is not designed for production injection**, which makes it a staging tool in a regulated programme. |
| **Pumba** | Container-level chaos for **Docker, containerd and Podman**: `kill`, `stop`, `pause`, `rm`, `restart`, plus network chaos via `netem`/`iptables` and resource stress via `stress-ng`. | CLI/docker-container; documented Kubernetes DaemonSet and OpenShift deployments. | **Apache-2.0** ✅. | **Active** ✅ — **3.2k stars**, **713 commits**, 216 forks, 46 watchers ✅, with a current README documenting containerd and Podman support and an explicit statement that **Windows is not supported and not planned** ✅ (a healthy, maintained-scope decision rather than an omission). ⚠ I did not read a release tag or last-commit date; the README's currency is the evidence. |
| **Chaos Monkey** (Netflix) | Terminates VMs and containers in production. | **Requires Spinnaker** — the README states "You must be managing your apps with Spinnaker to use Chaos Monkey to terminate instances" ✅. | **Apache-2.0** ✅. | ⚠ **NOT archived — but effectively dormant. This is the most important status call in the table.** The repository is **public and not archived** ✅ — 17.1k stars, 1.3k forks, **165 commits**, Go, with an **Apache-2.0 licence and an `OSSMETADATA` file** ✅. **But:** it carries **no releases** ✅, its README badges point at **Travis CI** (`travis-ci.com/Netflix/chaosmonkey`) ✅ and its support link is a **Google Group** ✅ — and Travis CI is defunct for open-source projects. **Netflix's own repository also carries the "NetflixOSS Lifecycle" badge** ✅, which is the project's own lifecycle signal. **Verdict: ⚠ treat Chaos Monkey as historical and educational, not as a production option for a 2026 programme.** It is a *fifteen-year-old tool with 165 commits*, and it needs Spinnaker — a dependency most banks do not have and would not adopt for this purpose. Its value now is as the origin document's artefact (§2), not as infrastructure. |
| **Blockade** | Network partition and failure testing: arbitrary partitions between containers, flaky networks (packet drop), slow networks (latency with distribution), duplicate packets. Explicitly designed for leader-election and quorum testing — "you could partition the leader away from the other nodes and ensure that the leader steps down" ✅. | Docker containers plus host-level `ip`/`tc`; YAML configuration; a CLI. | **Apache-2.0** ✅. | ❌/⚠ **Effectively abandoned — a flagged negative finding, and a case study in the dead-project rule.** The canonical repository path **`github.com/cloudfoundry-incubator/blockade` returns a hard 404** ✅ — the organisational home has been dissolved. The surviving code lives at **`github.com/worstcase/blockade`** ✅ (911 stars, 239 commits, 70 forks) — **a personal account, not a foundation or company** — and its README carries: a **Travis CI badge** pointing at `travis-ci.org` ✅ (a discontinued service), an **unfinished-work checklist file committed into the repo**, no listed releases ✅, and **"Blockade was originally developed by the Dell Cloud Manager (formerly Enstratius) team"** ✅ in the past tense. **Do not adopt this.** It is the exact archetype §6.1 warns about: a tool with real conceptual merit (it is one of the few purpose-built partition injectors) whose repository tells you it is over. |
| **WireMock** | Fault injection at the **HTTP API** layer: stub latency, malformed responses, error codes, connection resets, and stateful sequencing — the standard way to make a dependency misbehave on demand. | JVM library, standalone JAR, container, or cloud-hosted. | **Apache-2.0** ✅. | **Active** ✅ — **7.4k stars**, **4,851 commits**, **196 tags**, 1.5k forks ✅, with a **commit on 15 September 2026** ✅ and a current README including an AGENTS.md/Claude Code breaking-change report skill ✅ — i.e. actively maintained into the present. **Scope note:** WireMock is a *mock server*, not a chaos platform; it earns its place here because HTTP-level dependency fault injection is taxonomy class #6 and WireMock is the most maintained way to do it. |

### 6.6 Notable In-House Implementations

Verified examples of chaos capability built inside an organisation rather than bought:

- **Netflix (the originator).** The Simian Army is the archetype: an internally built tool family, announced publicly in July 2011 ✅, with the tools open-sourced under NetflixOSS ✅ and, in Chaos Monkey's case, now largely historical (§6.5). Two things about this model are worth copying and one is worth avoiding. **Copy:** the tools were built *by the teams that owned the production systems*, and the practice was institutionalised in the deployment platform (Spinnaker) rather than bolted alongside it ✅. **Avoid:** assuming that the open-sourced artefact is the capability. The capability was the culture around the tool, and §9 is about that.
- **Alibaba (ChaosBlade).** An in-house tool open-sourced and then maintained externally ✅ — currently being re-pointed at AI-agent use cases (§6.3). This is the "in-house becomes open source becomes something else" lifecycle in one repository.
- **Cloud providers.** AWS FIS and Azure Chaos Studio (§6.4) are in-house capability productised for customers — which is a different thing again: the provider runs chaos against its own services so that you do not have to build an injector, but the evidence stays in the provider's account (§10.8).
- ⚠ **Named bank implementations: not verified.** I am deliberately **not** naming any specific financial institution's internal chaos platform. My searches did not surface a verifiable primary account of a named bank's in-house chaos tooling, and this is exactly the kind of claim that circulates in vendor case studies without a checkable source. **If you want one, ask the vendor for a reference — and ask what the reference's regulator saw.**

### 6.7 The Verdict, Condensed

Every status above carries its evidence inline; this is the one-line read. **Adopt with confidence (active, foundation- or vendor-maintained, verified 17 Sep 2026):** LitmusChaos ✅, Chaos Mesh ✅, ChaosBlade ✅ (pin the version), Chaos Toolkit ✅, Pumba ✅, Toxiproxy ✅ (test/CI scoped by design), WireMock ✅, AWS FIS ✅, Gremlin ✅, Steadybit ✅/⚠. **Adopt with an explicit caveat:** Azure Chaos Studio ✅/⚠ (the GA model is feature-frozen and the actively developed model is a preview not intended for production), Harness ⚠ (renamed; check what you are actually buying). **Do not adopt:** Chaos Monkey ⚠ (dormant, Spinnaker-dependent — it is history, not infrastructure) and Blockade ❌ (abandoned; its original repository home no longer exists). **Do not assume:** Verica ⚠ (domain did not respond) and any Google Cloud equivalent ⚠ (not established this pass). **The rule that produces this list, restated because it is the point of the section:** a dead or archived project must never be presented as a live option, and the way you find out is to read the repository — the last commit, the last release, the licence, and whether the CI badge points at something that still exists — not the documentation site.

## 7. The Observability Prerequisite

You cannot run an experiment whose outcome you cannot observe. This is not a preference; it is a **precondition**, and it is the reason a chaos programme in a bank must be sequenced *after* observability, not alongside it. Four requirements, in the order they bite.

**1. A trustworthy steady-state signal (§3.4).** The metric, its computation, its exclusions, its lag. The five-question audit in §3.4 is the gate. **A system with no steady-state signal is not a candidate for chaos engineering; it is a candidate for instrumentation work**, and saying this out loud protects the programme from being blamed for findings it cannot produce evidence for.

**2. The three signal types, each for a different question.** **Metrics** tell you *whether* the steady state moved (aggregate, cheap, always-on — the abort-condition substrate). **Traces** tell you *where* it moved along the request path, which is what turns "latency rose" into "latency rose at hop 4 because the connection pool exhausted" — the finding, rather than the symptom. **Logs** tell you *why*, in the application's own words, but they are the slowest to search and the last to arrive. For every experiment in §5's taxonomy, name in advance which of the three will detect a deviation, and for the state-fault classes (10–12) add a **fourth, non-negotiable** signal: a **correctness check** — a checksum, a reconciliation count, an invariant assertion. Availability-only observability makes taxonomy rows 10–12 literally un-runnable, because the failure mode there is a system that is up and wrong.

**3. The false-positive problem — and this is the part practitioners under-plan.** You are **deliberately causing the symptom your alert watches for.** Three defensible strategies, each with a cost, and you must choose one *per experiment* and write it down:

| Strategy | What you do | Cost | When it is right |
| --- | --- | --- | --- |
| **Suppress** | Silence the specific alert for the experiment's duration and scope | You blind yourself to the *real* version of that failure during the window; needs a compensating watcher | Low ambiguity, high-volume alerts, mature alert hygiene |
| **Verify** | Leave alerts live and treat firing as a *desired* outcome — the experiment validates detection as well as resilience | The on-call queue gets real pages for a planned event; needs a loop-in before, not after | When **detection** is itself a property under test — the most valuable variant in a regulated estate |
| **Correlate** | Leave alerts live, but annotate the monitoring system with the experiment's start/end so responders can attribute | Requires the annotation to actually reach the responder | Default for most experiments |

**The honest recommendation:** prefer **verify** for anything customer-facing, because a resilience property you cannot *detect* is a resilience property you do not have — and because the alert-firing evidence is exactly the artefact §10.7 says a supervisor wants. Never suppress without a compensating watcher and a named person watching the suppressed signal.

**4. The control room, and rollback observability.** Name who is watching, in what channel, and with what dashboard open — before the experiment starts (§8.5). Two things people forget: the watcher must be **someone who did not design the experiment** (the author has a stake in completion; §4.3), and you must be able to observe the **rollback** as clearly as the injection. An experiment ends when you can *see* that the system returned to its pre-experiment state, not when you stop injecting.

**Cross-reference, not re-derivation.** The SLO and error-budget machinery — the definitions of SLI, SLO, error budget and the error-budget policy — is owned by [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2.3, and the instrumented availability metrics by §6 of the same guide. This section assumes those definitions and does not restate them. **⚠ Note:** I found no dedicated observability guide in `technology/` in this pass; if one exists elsewhere in the repository, treat it as the primary cross-reference and this section as the chaos-specific overlay.

## 8. Safety and the Guardrails

### 8.1 Blast-Radius Control

Principle 5 (§2.3) is a **duty**, and blast-radius control is how the duty is discharged. Four independent dials; you should be able to state the setting of all four in one sentence before proceeding:

1. **Scope** — which targets, named explicitly (a label selector, a host list, a single caller), *never* "all instances of X" and never a random pool whose worst case you cannot enumerate.
2. **Percentage** — the proportion of capacity or traffic exposed. Ramp it: **0% (rehearsal) → 1% → 5% → 25%**, holding each level until a full observation window passes clean. The 0% run is not a formality; it is the kill-switch rehearsal (§8.2).
3. **Duration** — a hard cap, enforced by the injector where the tool supports it, not by a human remembering to stop.
4. **Containment** — what the fault *cannot* reach: the shared database, the settlement path, the messaging backbone, the DR site. Written as a **negative** list ("this experiment cannot touch X"), because the negative list is what an auditor asks about.

**The blast-radius arithmetic you must do, and most programmes skip:** translate the percentage into a **business exposure** — *how many customer transactions, and of what value, could fail in the worst realistic case?* If you cannot compute that number, your blast radius is not controlled; it is merely bounded by optimism.

### 8.2 The Kill Switch as a Designed Mechanism

The kill switch is a **mechanism**, not a good intention, and the difference is testability. A designed kill switch has five properties: **(a)** it is *reachable by any participant*, not just the experiment's operator; **(b)** it is *independent of the injector* — if the injector is wedged, the stop path still works (this is why "the injector has a stop button" is not a kill switch); **(c)** it *reverses* the fault, not merely halts new injection — the fault already applied must be removed; **(d)** its effect is *observable* (§7); and **(e)** **it has been executed at least once, and that execution is recorded.**

**How you actually prove it works — the only method that does:** run the experiment at **0% scope** and press the kill switch. Then run it at **1%** and press the kill switch mid-window. Record both: who pressed it, when, the injector's response time, and the observed restoration of the steady state. **A kill switch whose rehearsal date is blank in the experiment record is an untested assertion, and the record should say so.** Add the failure mode everybody forgets: **what happens if the kill switch is needed because the injector itself has failed?** For every tool in §6, ask that question about that tool's failure modes — the answer is usually "a second, independent stop path" (e.g. a network rule at the host, a feature flag in the application, a DNS override), and it must be documented before the experiment, not discovered during it.

### 8.3 Stakeholders, Change Management and Environment Choice

**Notification.** Distinguish three audiences and notify each differently: the **control room** (continuous, in-channel, before and during), the **adjacent system owners** whose systems may see the fault's secondary effects — the messaging backbone, the reconciliation engine, the downstream counterparty gateway (before, with an abort window they can object within), and **service management / incident management** (before, so that a page is attributed rather than escalated — this is the item that prevents a chaos experiment from triggering a major-incident declaration at 03:00). Where the institution runs a formal change process, **the experiment should have a change record**; it is a controlled change to production behaviour, and the record is also §10.7's evidence.

**Environment choice — the honest staging-versus-production trade-off.** Both sides, plainly:

| | **Staging can prove** | **Staging cannot prove** |
| --- | --- | --- |
| Mechanism correctness | Does the circuit breaker open? Does the fallback return the cached value? Does the pod restart cleanly? | — |
| Tooling and process | That the injector works, the abort trips, the kill switch stops it, the runbook reads correctly | — |
| Systemic resilience | — | Whether the **interaction** of real traffic, real concurrency, real data volumes and real third-party behaviour degrades gracefully |
| Tuned parameters | — | Whether the timeouts, retry budgets and thread pools are set correctly **for production's actual latency and load** |
| Data and state | — | Anything requiring production data shape, volume or skew — the majority of a bank's interesting failure modes |
| Currency | — | Anything about the **currently deployed** version, because staging drifts |

**The position I would defend:** use staging for **familiarisation and mechanism validation**, production for **the confidence the discipline is actually claiming** — and be explicit about which of the two each experiment's result belongs to. A programme that reports staging results as resilience assurance is committing the error §10.7 calls false assurance.

### 8.4 The Paradox — When the Experiment Is the Incident

**State it plainly: a chaos experiment can cause a real incident.** This is not a risk that better tooling removes; it is intrinsic to the practice, because the practice deliberately degrades production. The canon itself concedes the possibility in principle — principle 5 exists only because "experimenting in production has the potential to cause unnecessary customer pain" ✅ — and every experienced practitioner knows the class of failure: an abort condition set too loose; a kill switch that turned out to depend on the component being tested; a blast radius larger than the arithmetic suggested because a dependency was not in the asset inventory; a fault that interacted with an unrelated in-flight change.

**On citations, and being honest about a gap.** I searched specifically for documented, datable cases of chaos experiments that caused real production outages — including the widely-repeated genre of "chaos engineering gone wrong" stories. **My searches returned empty result sets** ✅. I am therefore **not** citing a specific incident, and specifically I am **not** attributing any named outage (including any cloud provider outage) to a chaos experiment, because I could not verify that attribution at a primary source. **What I can say without inventing a citation:** the *class* of failure is acknowledged by practitioners and the risk is conceded in the canon's own text ✅; the mitigation is engineering discipline, not denial. ⚠ Treat every specific "chaos caused this outage" anecdote you read as **unverified** until it names a primary source and a date. **If you are a practitioner and it happened to you, write it down** — this is the single most under-documented area of the discipline.

**The consequence for a regulated institution** — and this is the sentence to carry into a risk committee: because a failed experiment may itself be a reportable operational incident (§10.5), the experiment's *governance* must be as strong as the incident it might cause. Which is what the checklist below is for.

### 8.5 The Pre-Flight Checklist

A real checklist — every line must be answerable, in writing, with a name against it. If any line is "no", the experiment does not start.

```
IDENTITY AND APPROVAL
[ ] Experiment ID, title, author, and the window (start AND hard end)
[ ] Property under test named: absorption | degradation | detection | recovery
[ ] Hypothesis written as ONE falsifiable sentence
[ ] Approval recorded: change record raised, business owner and system owner named
[ ] Risk assessment: business exposure computed (transactions, value) — not "low"

STEADY STATE AND OBSERVABILITY
[ ] Steady-state signal named, with source system, unit, baseline window, sample size
[ ] Tolerance band set, and derived from the service's SLO / error budget
[ ] Signal lag stated (how long after the cause does the signal move?)
[ ] Correctness signal present for state-fault experiments (checksum / reconciliation / invariant)
[ ] Dashboards open, shared and timestamped — rollback visible as well as injection
[ ] Alert-handling strategy chosen per experiment: suppress / verify / correlate — recorded

BLAST RADIUS
[ ] Target set named EXPLICITLY (selector / host list) — never "all of X"
[ ] Percentage set, and ramp level recorded (0% rehearsal done? yes/no)
[ ] Hard duration cap configured IN THE INJECTOR
[ ] Containment written as a NEGATIVE list: what this cannot reach
[ ] Shared dependencies, settlement paths and third parties identified and excluded or accepted

STOP MECHANISM
[ ] Abort conditions declared before the run: thresholds, dwell times, machine-checkable
[ ] Kill switch exists and is INDEPENDENT of the injector
[ ] Kill switch rehearsed at 0% and at 1%, with the rehearsal recorded (date, operator, restore time)
[ ] Second, independent stop path documented (host rule / feature flag / DNS override)
[ ] Named operator AND named backup, both in the control room, both able to abort

PEOPLE AND TIMING
[ ] Control room staffed, channel open, watcher NOT the experiment's author
[ ] Adjacent system owners notified, with an objection window that has now closed
[ ] Service management / incident management notified, so a page is attributed not escalated
[ ] Timing: not month-end, not quarter-end, not a batch window, not a reporting deadline
[ ] No deployment, migration or change in flight; no change freeze; no unrelated major incident

DATA AND STATE
[ ] Data touched by the experiment identified; production data protection rules applied
[ ] Restoration proven where the experiment can alter state (restore test, not a restore plan)
[ ] Reconciliation and outbound-integration effects considered and accepted

AFTER
[ ] Observation and cooling window scheduled, with a named observer
[ ] Findings register open, remediation owner named BEFORE the run
[ ] Artefacts list agreed and pinned (config commit, dashboard exports, abort log)
[ ] Re-run scheduled, so the remediation is verified rather than assumed
```

## 9. The Practice and the Culture

### 9.1 The Game Day

A game day is a **scheduled, facilitated exercise** in which one or more experiments run with defined roles, a control room and a written record. It is the mechanism by which a chaos programme becomes an *event* rather than a background hazard — and, per §3.3, the event-ness is a feature, not overhead.

**Design.** Pick a **scenario drawn from the institution's own severe-but-plausible list** — and cross-reference here rather than re-deriving: [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §6 owns the severe-but-plausible scenarios, and its §11 owns the testing and assurance framework. A game day that tests a scenario no regulator, risk function or board member would recognise is a hobby.

**Roles** — five, and each must be filled by a named person:

| Role | Responsibility | The failure if unfilled |
| --- | --- | --- |
| **Facilitator** | Runs the agenda, keeps time, asks "what did we just see?", protects the psychological safety of the room | The exercise drifts into a demo |
| **Injector operator** | Executes the fault, holds the abort, presses kill if needed | Ad-hoc injection; no clean control |
| **Observers** (≥ 2, not the designer) | Watch the dashboards and the signals; write timestamped notes | Findings become recollections |
| **Scribe / evidence owner** | Captures the record: timeline, decisions, aborts, artefacts | Nothing survives to satisfy §10.7 |
| **Business / service owner** | Holds the authority to stop on commercial grounds and owns the customer impact assessment | An engineering-only exercise with no business accountability |

**Cadence.** Start at **one game day per quarter**, one critical service, one fault class. Earn the right to increase frequency by producing remediations that closed. **Monthly is the realistic maximum for a large bank** without dedicated staff; continuous automation is a different programme (§3.3) and should be justified separately.

### 9.2 The Champion Model and the Psychology

**The practice runs on a blameless culture, and this is not a soft point — it is a structural dependency.** An experiment that finds a weakness finds *a person's* weakness: the timeout someone set, the fallback someone never wrote, the replica count someone reduced. In a blame culture, running an experiment is **career risk**, and the rational engineer's response is therefore to run only experiments that will pass — which is anti-pattern #1 (chaos theatre) arrived at through self-preservation rather than dishonesty. **You cannot manage your way to honest experiments; you have to build the culture that makes honesty safe.** That means: findings are attributed to *systems and conditions*, never to individuals; the first experiment's report is reviewed by leadership for *learning*, not for fault; and the programme's own metric is remediations closed (§9.4), not findings raised.

**The champion model.** A **resilience champion** per critical service — an engineer who knows the service, has run the experiments, and owns the findings register for that domain — is the only scaling mechanism that works, because chaos engineering requires service-specific knowledge that no central team can hold for a whole bank. The central function's job is: the framework, the guardrails, the checklist, the tooling, the evidence pack, and the training. The champions' job is the experiments. A programme that tries to centralise both fails at the second service.

**Leadership sponsorship.** Three things leadership must provide, and none of them is money: **(a)** an explicit statement that a failed experiment is not a discipline matter; **(b)** the authority to spend an error budget on experiments (which requires the budget to exist — cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2.3); and **(c)** the willingness to keep the programme after the first experiment finds something embarrassing. **Item (c) is the real test**, and §13's last anti-pattern is what happens when it fails.

### 9.3 The Maturity Progression

| Stage | What it looks like | What unlocks the next stage |
| --- | --- | --- |
| **1. Manual drill** | A hand-run experiment in a staging environment, single fault, facilitator present, generous observation window | The team can define a steady state and an abort condition correctly |
| **2. Scheduled exercise** | Game days in production, one critical service, ramped blast radius, full evidence pack, findings register with owners | Findings are actually remediated and re-run; the kill switch has been rehearsed |
| **3. Recurring practice** | Game days across several services; champions per domain; experiments tied to the change calendar; findings feeding the backlog as a standing item | Blast radius and abort conditions are reliable enough to raise frequency |
| **4. Continuous automated verification** | Low-severity experiments automated, running on deploys or on a schedule, with auto-abort and alerting | Architecture-change review that re-scopes every automated experiment (§3.3) |

**Cross-reference, deliberately:** this is the same model that [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) **§7.5 ("Building the Practice — the Blast-Radius Ramp")** presents, and the two sections should be read together — that guide owns the summary version of this progression inside the availability context; this is the same ramp applied to a whole programme, with the regulation-facing consequences added in §10. Note the honest limit: **most large banks will reach stage 3 and stop**, and that is a legitimate destination rather than a failure (§11).

### 9.4 The Honest Metrics

**Metrics that measure the wrong thing, and are nonetheless the ones everyone reports:** *number of experiments run* (rewards activity), *number of instances killed* (rewards violence), *services covered* (rewards breadth over depth), *chaos days held* (rewards the event, not the outcome). Every one of these can be maximised by a programme that changes nothing about the bank's resilience — which is the definition of a vanity metric.

**Metrics that actually indicate a working practice:**

| Metric | Why it is the right one |
| --- | --- |
| **Experiments that produced a remediation** (count, and % of experiments) | The only metric that ties the programme to a changed system. A rate near zero means the programme is theatre |
| **Mean time to remediate a finding** | Measures whether the organisation can *act* on resilience knowledge — the actual constraint in most banks |
| **Findings closed and re-verified** (with the re-run result attached) | Distinguishes "we fixed it" from "we believe we fixed it" |
| **Repeat findings** (same weakness found twice) | A rising count is the clearest possible signal that the findings loop is broken |
| **Experiments that tripped their abort condition** | Not a failure metric — a **control-effectiveness** metric. A programme where abort never trips may have abort conditions that are too loose |
| **Kill-switch rehearsals completed** | Direct evidence the safety mechanism is real, not intended |
| **Blast radius as business exposure** (worst-case transactions/value, trended down) | Shows the programme is growing in confidence *and* in control simultaneously |

**Report these to leadership; report the vanity metrics to no one.** And note the honest framing for the board: the programme's output is not "resilience" (unmeasurable in the general case) but a **trend in verified resilience evidence** — which is precisely the currency §10.7 shows a supervisor trades in.

## 10. The Regulated-Institution Angle

Every finding below was verified on **17 September 2026** and each is dated. This area moves, so treat any statement here without a date as expired. Where I could not reach a primary source, it is marked as an explicit negative finding rather than filled in from memory.

### 10.1 The Drivers — DORA and the EU TLPT Regime

**The instrument.** **Regulation (EU) 2022/2554** on digital operational resilience for the financial sector (DORA), "of the European Parliament and of the Council of **14 December 2022**", amending several sectoral regulations ✅. **On the source channel, stated honestly:** the article text below was read through a **full-text mirror of the regulation** (`digital-operational-resilience-act.com`, which reproduces the articles verbatim) because **EUR-Lex refused automated extraction on 17 September 2026** ✅ — the same access failure the sibling guide recorded, and re-confirmed by this guide's own check. So the *wording* is regulation text, but the *channel* is a third-party mirror, not the Official Journal: treat this as **P (via T mirror)** and go to EUR-Lex in a browser for anything you are putting in a regulatory submission. The adopting formula in that text reads "Done at Strasbourg, 14 December 2022" ✅. **The application date — verified against the article text:** DORA **Article 64 ("Entry into force and application")** states plainly, "**It shall apply from 17 January 2025**" ✅. The sibling guide [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §2 records the same date, and this pass re-verified it against the article text itself.

**The testing provision — Article 26, "Advanced testing of ICT tools, systems and processes based on TLPT"** ✅. The provisions that matter to a chaos engineer:

| Element | What Article 26 actually says ✅ |
| --- | --- |
| **Who** | Financial entities other than those referred to in Article 16(1) first subparagraph, and **other than microenterprises**, which are **identified in accordance with paragraph 8** — i.e. a defined subset, not everyone |
| **Cycle** | "at least **every 3 years**"; the competent authority "may, where necessary, request the financial entity to **reduce or increase** this frequency" |
| **Scope** | Each TLPT "shall cover **several or all critical or important functions**"; the entity identifies underlying ICT systems, processes and technologies supporting those functions; the entity assesses which functions need coverage and "the result of this assessment shall determine the precise scope of TLPT and **shall be validated by the competent authorities**" |
| **THE CRITICAL ONE** | "shall be performed on **live production systems** supporting such functions" |
| **Third parties** | Where ICT third-party service providers are in scope, the entity must ensure their participation and "shall retain at all times **full responsibility** for ensuring compliance"; **pooled testing** across several entities is expressly contemplated, with the provider contracting an external tester directly and the test still counting as TLPT |
| **Risk controls** | Entities, with the cooperation of providers and testers, must "apply effective risk management controls to mitigate the risks of any potential impact on data, damage to assets, and disruption to critical or important functions, services or operations" — note "at the financial entity itself, its counterparts or to the financial sector" |
| **Identification criteria (para 8)** | Competent authorities identify entities by reference to: (a) **impact-related factors** — the extent to which the entity's services impact the financial sector; (b) **possible financial stability concerns**, including systemic character at Union or national level; (c) **specific ICT risk profile, level of ICT maturity, or technology features involved** |
| **Internal vs external testers** | Entities shall contract testers per Article 27; when using **internal** testers they "shall contract **external testers every three tests**". **Credit institutions classified as significant under Article 6(4) of Regulation (EU) No 1024/2013 shall only use external testers** |
| **Authority designation** | Member States **may designate a single public authority** for TLPT matters nationally; failing that, a competent authority may delegate tasks to another national authority |
| **Attestation and mutual recognition** | Authorities provide an attestation that the test was performed per requirements "**in order to allow for mutual recognition** of threat led penetration tests between competent authorities"; the entity notifies the authority of the attestation, the summary of findings and the remediation plans — while remaining "**fully responsible for the impact of the tests**" |

**Article 27 — "Requirements for testers for the carrying out of TLPT"** ✅ sets out the tester conditions: testers must be "of the highest suitability and reputability"; possess technical and organisational capabilities and **demonstrate specific expertise in threat intelligence, penetration testing and red team testing**; be **certified by an accreditation body in a Member State or adhere to formal codes of conduct or ethical frameworks**; provide **independent assurance or an audit report** on risk management including protection of confidential information and redress; and be "**duly and fully covered by relevant professional indemnity insurances, including against risks of misconduct and negligence**" ✅. For **internal testers**, additional conditions apply ✅: the use must be **approved by the competent authority** (or the designated single public authority); the authority must have **verified sufficient dedicated resources and that conflicts of interest are avoided** throughout design and execution; and **the threat intelligence provider must be external to the entity** ✅. Contracts with external testers must require sound management of TLPT results across their whole lifecycle ✅.

**The operationalising RTS.** The joint Regulatory Technical Standards on TLPT are the instrument that turns Article 26 into a method. Verified on the **European Banking Authority's own single-rulebook page** ✅: status **"In force"**, **application date 08/07/2025** ✅, published in the Official Journal and linked to **CELEX 32025R1190** — i.e. **Commission Delegated Regulation (EU) 2025/1190** ✅. The EBA page states the RTS specify: the **criteria for identifying** entities required to perform TLPT; the **requirements and standards governing the use of internal testers**; the requirements on **scope, testing methodology and approach for each phase**, and the **results, closure and remediation stages**; and the **type of supervisory cooperation** needed, including for **mutual recognition** ✅. Article 26(11) itself required the **ESAs, in agreement with the ECB**, to develop these RTS **in accordance with the TIBER-EU framework**, and to submit the drafts to the Commission **by 17 July 2024** ✅ — which is why TIBER-EU (the ECB's threat-intelligence-based ethical red-teaming blueprint) is the methodological ancestor of EU TLPT, not a separate regime.

**What this means for the discipline, in one sentence:** the EU has **mandated, in law, the injection of realistic adversarial failure into live production systems of significant financial entities on a three-year cycle, under supervisory approval, using accredited external testers, with the results reported to the supervisor** — which is the closest thing in existence to a regulatory charter for the *spirit* of chaos engineering, while being methodologically a **red-team exercise**, not a chaos experiment (§10.5).

### 10.2 The UK Position — PRA/FCA and CBEST

**Operational resilience.** The PRA/FCA operational-resilience regime — impact tolerances and the set of **severe but plausible** scenarios against which firms must test their important business services — is owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §2, §4–§6, and I do not re-derive it here. The relevant point for this guide is the **shape** of the requirement: the UK regime tests against *scenarios the firm self-identifies and the supervisor scrutinises*, which is **scenario-based testing**, not adversarial injection and not continuous chaos (§10.6).

**CBEST.** The **Bank of England** operates **CBEST**, described on its own pages as a **security assessment framework** implemented by the Bank, the PRA and the Bank's Financial Market Infrastructure Directorate, with an **implementation guide for CBEST participants** ✅. Its method is **threat-intelligence-led assessment** — i.e. real threat intelligence shapes the test — and it is conducted in **"'live' corporate environments"** ✅ (the Bank's own 2025 CBEST thematic page, which exists and reviews findings from recent threat-led penetration testing across banks, insurers, asset managers and financial market infrastructures ✅). **Status: active, with an annual thematic publication** ✅ — the **2025 CBEST thematic** is published on the Bank of England site ✅, and a **2026 third-party commentary** discusses that 2025 thematic ⚠ (third-party summary). **CBEST is the UK's cousin to EU TLPT** and shares TIBER's intellectual lineage — threat intelligence, red team, live environment, supervisory visibility. ⚠ I did **not** verify CBEST's own framework version history or its current mandated population at a primary source in this pass; read the Bank's implementation guide for those.

### 10.3 The Singapore Position — MAS and AASE

**Business continuity.** The **MAS revised Business Continuity Management (BCM) Guidelines were issued on 6 June 2022** ✅, confirmed against the **MAS's own published PDF** ("BCM-Guidelines-June-2022") ✅ and corroborated by multiple law-firm and advisory summaries ✅. The revised guidelines **supersede the previous June 2003 version** ⚠ (per third-party summaries; the MAS PDF's own preface introduces the framework but I did not read a supersession clause directly). The sibling guide [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §2 also covers MAS BCM Guidelines 2022, MAS TRM notices, outsourcing requirements 2023–24 and incident reporting 2025–26 — cross-ref rather than re-derive, and note that those latter items are **that guide's findings**, which I did not independently re-verify this pass ⚠.

**Adversarial testing — the verified Singapore mechanism.** In **2018** the **Association of Banks in Singapore**, **with support from the MAS**, released the **Adversarial Attack Simulation Exercises (AASE) Guidelines** — commonly called the **"Red Teaming" guidelines** ✅ — confirmed against the **ABS's own published PDF media release dated 14 November 2018** ✅ and the **ABS guidelines document itself** ✅. The ABS describes AASE as "**sanctioned, planned, risk-managed and objective-driven cyber security assessments that simulate highly sophisticated targeted attacks against an organisation**" ✅. Note the four adjectives — *sanctioned, planned, risk-managed, objective-driven* — because they are the industry's own answer to the safety problem of §8, and note also that **this is industry-guideline-based, not a statutory testing mandate of the DORA kind**; its binding force comes through MAS supervision and the TRM expectations rather than through a dedicated testing regulation ⚠.

### 10.4 The BCBS Material

**BCBS "Principles for Operational Resilience" — March 2021** ✅. Verified at the **Bank for International Settlements' own publication page** (URL `bis.org/publications/202103-guidelines-principles-operational-resilience`) ✅ and the published PDF, which carries the line "**Principles for Operational Resilience March 2021**" ✅. The BIS states the Committee "**seeks to promote a principles-based approach to improving operational resilience**" ✅. Corroborating datable evidence: the **FSB** dates the principles **31 March 2021** ✅, and the **HKMA** wrote to banks noting that "**on 31 March 2021 the Basel Committee on Banking Supervision ('BCBS') issued two sets of principles, viz., the Principles for Operational Resilience ('POR') and the Revised Principles for Sound Management of Operational Risk**" ✅. So the **March 2021 / 31 March 2021** date is confirmed across three independent primary-or-near-primary sources. **The relevance to testing is directional rather than prescriptive:** the BCBS principles set expectations about resilience capability and testing **without prescribing a specific testing method, cycle or authority-approval gate** — which is precisely why DORA's Article 26 is the outlier and the most consequential instrument in this section.

### 10.5 The Structural Tension

Here is the analysis no vendor blog will give you. DORA Article 26 mandates testing **"on live production systems"** ✅. The Principles of Chaos mandate experimenting **"in production"** ✅. On that single point, the regulator and the canon agree. **The agreement ends immediately**, and the divergence is not a matter of degree — the two practices differ in **purpose, method, ownership and evidence**:

| Dimension | **Chaos engineering** | **TLPT / CBEST / AASE** |
| --- | --- | --- |
| **Purpose** | Build confidence in systemic resilience; find unknown weakness | Test the firm's ability to **detect and respond** to a sophisticated adversary |
| **Adversary model** | Non-adversarial failure (hardware, network, dependency, time) | **A motivated human adversary** with threat intelligence and intent |
| **Hypothesis** | Mandatory and falsifiable | Not hypothesis-driven; **objective-driven** |
| **Who runs it** | The firm's own engineers (usually) | **Accredited external testers** (mandatory for significant EU credit institutions; every third test otherwise) |
| **Approval** | Internal change control | **Supervisory validation of scope**; competent-authority approval for internal testers ✅ |
| **Cycle** | Continuous or per-change | **At least every 3 years** ✅ |
| **Output** | Findings → remediation → re-run | **Findings + remediation plans + attestation**, notified to the supervisor ✅ |
| **Mutual recognition** | N/A | **Explicitly designed for** across competent authorities ✅ |
| **Who bears the risk** | The firm, internally | The firm — "**fully responsible for the impact of the tests**" ✅ |

**The irony, stated plainly: a chaos experiment in a regulated firm may itself be a reportable operational incident.** The chain is not hypothetical. An experiment deliberately degrades production; principle 5 concedes "there must be an allowance for some short-term negative impact" ✅; in a DORA-scoped firm, a material operational or security incident affecting ICT services triggers **incident-reporting obligations** (the sibling guide records the incident-reporting RTS as **2024/1772 and 2025/301** — ⚠ **that attribution is the sibling guide's finding and I did not independently re-verify those RTS numbers this pass**); and a firm that reports its experiment as an incident has, in effect, **generated a regulatory event with its own resilience tooling**. Whether a *planned and controlled* experiment triggers a reporting duty depends on materiality thresholds and on how the firm's incident-classification policy handles planned change — which is a question **your compliance function must answer in writing before your first production experiment**, not after. **Getting that answer is item one of the Cymbal Bank programme in §12.3.**

**The second-order tension: continuous automation versus supervisory visibility.** §3.3 argued that continuous injection erodes the experiment's status as an event. In a regulated estate this is not merely a documentation inconvenience: a continuous injector means that **live production reliability is permanently conditioned on a fault injector being correctly scoped** — and a supervisor assessing "are you in control of your ICT systems?" will find "a system continuously injects failure into production by design" a materially harder answer to evidence than "we conducted a controlled exercise on a documented date."

### 10.6 The Resolutions Institutions Actually Reach

The tension is real but it is not a deadlock, and the resolution is a **portfolio**, not a choice. Five instruments, each doing something the others cannot:

1. **Scenario-based testing and tabletops.** Cheap, safe, and aligned with the PRA/FCA *severe-but-plausible* framing. Proves **decision-making, escalation and communication** under stress — which is where most real incidents are actually lost. **Cannot** prove a timeout is correct.
2. **The DR exercise and failover test.** Proves **recovery mechanics** and validates RTO/RPO claims against the institution's own documented procedures; cross-ref [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §8 for DR and [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §29 for platform DR drills. **Cannot** prove absorption without failover (§1.2, §5 row 15).
3. **The regulator-scoped adversarial test (TLPT / CBEST / AASE).** Proves the firm can **detect and respond to a motivated adversary** — and is the instrument with the strongest external evidentiary status because it produces an **attestation** ✅. **Cannot** substitute for resilience testing: it is adversary-shaped, not failure-shaped, and it runs on a three-year cycle.
4. **Production testing with controls.** The chaos experiments of this guide, run with §8's guardrails — ramped blast radius, rehearsed kill switch, defined abort, control room, change record. Proves the **tuned-parameter and systemic-interaction** properties that nothing else in the portfolio reaches. **Cannot** be run without the prior three: the scenario work tells you what matters, the DR work tells you the recovery path, and the adversarial work tells you the threat model.
5. **The change window and the notification.** The instrument that makes (4) admissible in the estate: schedule experiments inside the institution's existing change and maintenance framework, notify service management so pages are attributed rather than escalated, and record the experiment as a controlled change.

**The staged maturity sequence a real institution follows** — and note that it starts with the cheapest and least risky, which is also the correct order for building credibility: **tabletop → the institution's existing DR/failover test → staging chaos (mechanism validation) → production chaos at 1% on a non-customer-facing path → production chaos on a customer-facing path with a full evidence pack → (much later, if ever) continuous automation.** DORA TLPT sits **alongside** this sequence rather than inside it: it is a separate, supervisory-scoped, externally-tested exercise that consumes the same estate and the same risk appetite.

### 10.7 The Evidence Question

**What a supervisor actually asks for** — extrapolated from the instruments verified above, and stated as the shape of the ask rather than a quote: *what did you test, over what scope, using what method, with what result, what did you find, what did you do about it, who approved it, and what does the board know?* The DORA text is unusually concrete about the last two ✅: at the end of testing, "the financial entity and, where applicable, the external testers shall provide to the authority … a **summary of the relevant findings, the remediation plans and the documentation demonstrating that the TLPT has been conducted in accordance with the requirements**"; the authority then issues an **attestation**; and the entity "shall **notify** the relevant competent authority of the attestation, the summary of the relevant findings and the remediation plans" ✅.

**The audit trail, therefore, must show:** scope and why that scope (the criticality reasoning) · method and the standard it conforms to · the **controls applied to manage risk during the test** ✅ · results — including **negative and inconclusive results** · findings with severity · remediation plans with owners and dates · approval records (internal and, where applicable, supervisory) · and board/committee reporting.

**Where chaos engineering's own artefacts help.** The experiment design of §4.4 is **strikingly close to what the audit trail wants**: the hypothesis, the steady-state definition, the scoped variables, the **blast radius as business exposure**, the **abort conditions**, and the **kill-switch rehearsal record** are exactly the "controls applied to manage risk" evidence. The findings register with owners and due dates maps directly onto "remediation plans". The pinned configuration and exported dashboards map onto "documentation demonstrating the test was conducted in accordance with requirements". **A well-run chaos programme is, almost by accident, generating DORA-shaped evidence** — which is the single strongest business case for the practice in a regulated firm, and the argument I would lead with.

**Where they fail to satisfy it.** Three gaps, honestly:
- **Authority.** No internal chaos experiment produces a **supervisory attestation** ✅. Mutual recognition exists only for TLPT ✅. A chaos programme's findings are internal evidence, not an attested test — so it **supplements** the regulator-scoped test and never substitutes for it.
- **Tester independence.** Article 27's tester conditions ✅ — accreditation, professional indemnity insurance, independent assurance — are not met by an in-house team, and internal testers require supervisory approval plus an external threat-intelligence provider ✅. In-house chaos engineering cannot be presented as TLPT.
- **Scope validation.** TLPT scope "shall be validated by the competent authorities" ✅. A chaos programme chooses its own scope. That is a feature for engineering, a gap for evidencing.

**The synthesis:** chaos engineering is **how you generate the engineering facts**; TLPT/CBEST is **how you obtain the supervisory attestation**; and a mature firm uses the first to be ready for the second, and uses the second's findings as input to the first's backlog.

### 10.8 The Third-Party and Cloud Dimension

**Is injecting failure into a provider's managed service permitted?** ⚠ **I could not verify the current terms of service of AWS, Azure or Google Cloud regarding customer-initiated fault injection against their managed services in this pass** — my searches returned empty result sets, and I am **not** going to paraphrase cloud acceptable-use terms from memory. **Therefore: this is an explicit flagged negative finding, and it is a question you must resolve with your provider and your legal function in writing before injecting into any managed service.** The general shape of the risk, stated as a question rather than an assertion: a customer-initiated fault against a managed service can affect **other tenants' resources in shared infrastructure**, can appear to the provider as **abuse or an attack**, and can void **service-level commitments** — the specific answers live in each provider's acceptable-use policy, its SLA exclusions, and its security-testing policy. Treat any experiment that touches a provider's managed control plane as **requiring written authorisation**.

**The provider's own fault-injection service, and who owns the evidence.** Both verified services (§6.4) let the provider execute fault injection **on the provider's infrastructure, under the provider's guardrails, with results in the provider's console** ✅. Three consequences:
1. **You own the obligation; the provider holds the evidence.** DORA Article 26(3) makes this explicit for TLPT: where an ICT third-party provider is in scope, the entity "shall take the necessary measures and safeguards to ensure the participation of such ICT third-party service providers" and "shall retain at all times **full responsibility** for ensuring compliance" ✅. The same logic applies to any resilience evidence held by a provider: if the supervisor asks, **you** must produce it.
2. **Retention and exportability become contractual matters.** Ask, in writing: how long are scenario reports retained; can they be exported in a form that can be placed in an audit file; can they be shared with a supervisor; and what happens to them if the relationship ends.
3. **Pooled testing is the regulatory answer to the multi-tenant problem** — Article 26(4) ✅ expressly contemplates a provider contracting an external tester directly to run **pooled TLPT** for several financial entities at once, "under the direction of one designated financial entity", with the test still counting as TLPT carried out by the participating entities ✅, and with the number of participants "duly calibrated taking into account the complexity and types of services involved" ✅. **If your institution is a significant DORA-scoped user of a major cloud provider, pooled testing is the mechanism that exists specifically for you, and it is worth raising with your provider now.**

## 11. The Banking Engineering Reality

Bounded and cross-referenced — the estate is described elsewhere in this repository and I do not re-derive it.

**Where the practice has to work.** Three realities shape every experiment: **(a) always-on payment rails** — the estate that cannot be inside a maintenance window at all, described in [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) and [../banking/nets_singapore_guide.md](../banking/nets_singapore_guide.md); **(b) the core and its batch windows** — [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), where the "steady state" is only partly a request/response signal and partly an end-of-day completion; **(c) the middleware estate** — [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) and [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md), which is where taxonomy class #6 (dependency failure) and class #2 (latency) will actually be run. The payments hub ([../banking/payments_hub_guide.md](../banking/payments_hub_guide.md)) and the ISO 20022/IPBS connectivity ([../banking/ibps_payment_connect_guide.md](../banking/ibps_payment_connect_guide.md)) are the systems with the least tolerance for a badly-scoped experiment in the whole estate. The 24-7 payments switch and its availability requirements are owned by [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §9.

**The safe places to start.** Back-office and internal-facing services first: statement generation, notification dispatch, internal reporting, non-customer-facing admin APIs. These share the platform, the observability stack and the deployment pipeline with the critical rails — so an experiment there **exercises the same mechanisms** its critical siblings depend on, while the worst realistic case is a delayed internal report rather than a failed payment.

**The sequence a risk-averse institution follows** (the same ramp as §10.6, expressed as a plan): an internal-facing service → a non-customer-facing path of a customer-facing service → a customer-facing path in a low-traffic window → a customer-facing path in a normal window → a critical rail, only after the evidence pack form is proven and the supervisor conversation has happened.

**The calendar constraints, and they are harder than the technical ones.** Month-end and quarter-end (batch and reporting peaks); the regulatory reporting deadlines; the settlement and cut-off calendars of the relevant schemes; the batch window itself; the change freeze periods; and the firm's own peak trading or peak-transaction days. **An experiment scheduled on a day the operations team already considers risky will be cancelled, and after two cancellations the programme has lost its slot.** Build the calendar into the design, not into the negotiation.

**The relationship to the DR drill and failover test the bank already runs.** They are complementary and must be **distinguished in the record**, or the programme will be credited with evidence it did not produce: the DR drill proves recovery (RTO/RPO to a target), the chaos experiment proves absorption (survival without failover) — §1.2 and §5 row 15. **Practical recommendation:** run the chaos experiment **after** a successful DR drill on the same service, because the DR drill confirms the recovery path exists (so the chaos experiment's abort has somewhere to go) and because the DR drill's own exercise record establishes the baseline maturity.

**How much of the SRE-style ideal a large bank can realistically adopt — the honest statement.** A large bank can adopt: the hypothesis discipline; the steady-state definition; the taxonomy's first six classes; ramped blast radius; rehearsed kill switches; game days; champions; findings-to-backlog; and a credible evidence pack. A large bank will struggle to adopt: **continuous automated injection against critical rails** (the change-control and notification burden, and §10.5's supervisory-visibility problem, are real constraints, not timidity); **production experiments in the core** (batch-coupled steady states with hours of lag make abort conditions hard to construct honestly); and **the cultural precondition** (§9.2) in institutions where an adverse finding is still career risk. **Realistic target: stage 3 of §9.3, with staged and carefully-evidenced excursions into customer-facing paths.** That is not a failure of ambition — it is the honest ceiling, and a programme that claims more than this without the evidence is anti-pattern #4.

## 12. The Cymbal Bank Worked Example

**Cymbal Bank is a fictional institution, and every figure in this section is explicitly illustrative/fictional.** No number here is a benchmark, a survey result, or a claim about any real bank. The purpose is to show the *shape* of a programme that a resilience function could actually get approved.

### 12.1 Scoping by Criticality

The function begins by taking the institution's existing business-impact analysis and criticality ratings — **do not re-do the BIA**; [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §4–5 owns it — and lining services up against the taxonomy of §5. The scoping table they build (illustrative):

| Service | Criticality | Steady state measurable continuously? | Fault classes available | Blast radius tolerance | Verdict |
| --- | --- | --- | --- | --- | --- |
| Internal statement-generation service | Low | Yes — job success rate, 1-min buckets | #1, #7, #8 | Internal reports delayed up to 30 min | **Start here** |
| Notification dispatch (SMS/email) | Low | Yes — dispatch rate | #1, #6, #2 | Customer notifications delayed | **Second** |
| Card authorisation API (non-settlement path) | High | Yes — approval rate, p99 | #1, #2, #6 | Declines for a **bounded** set of transactions | **Third, later** |
| Payment instruction API (settlement path) | Critical | Yes, but settlement confirmation lags to EOD | #1, #2, #6 | **None acceptable at launch** | **Deferred** |
| Core ledger batch | Critical | **No** — EOD completion only, hours of lag | #7, #9 | None | **Deferred — steady state not constructible (§3.4)** |

**Note what the scoping exercise does:** it eliminates two services on *observability* grounds before any risk discussion happens, which is the §7 precondition doing real work. That is a finding in itself, and the function reports it as one.

### 12.2 The First Three Experiments, Designed in Full

**(All figures illustrative/fictional.)**

**CE-001 — Internal statement generation: pod termination.**
- **Hypothesis:** *"If 25% of the pods serving statement generation in one zone are terminated, the remaining pods will absorb the work within the 30-second drain window and the job success rate will remain ≥ 99.5%."*
- **Steady state:** job success rate (scheduler, 1-min buckets) **and** p99 job duration. Baseline: 30 min, ≥ 400 jobs. Tolerance: success ≥ 99.5%; p99 ≤ the service SLO's threshold; zero jobs stuck > 15 min. Lag: ~30 s.
- **Blast radius:** 3 of 12 pods, one zone, single event, no repeat for 15 min. Worst realistic case: **up to 25% of one zone's statement jobs fail and are retried by the scheduler**, exposing an estimated **8% of daily statement volume** to delay, with **zero customer-facing impact** (statements are delivered to the customer portal on a schedule).
- **Abort:** success rate < 99.0% for 60 s; any stuck job > 15 min; operations lead calls it. Auto-abort configured.
- **Kill switch:** removal of the chaos annotation; **rehearsed at 0% and 1% first**, recorded.
- **Window:** 10 min baseline, 15 min observation, 10 min cooling. Alert strategy: **verify** — the pod-restart alert firing is a desired outcome.
- **Findings (illustrative):** (1) the service's connection pool did not recover after the pods returned, leaving throughput at ~80% for 40 minutes — **this is exactly the class of finding nothing else produces**; (2) an alert that should have fired did not, because the threshold was set against a 5-minute average that the 30-second dip never breached. **Two remediations, both with named owners.**
- **Artefacts:** design doc, pinned manifest, exported dashboards, abort log, findings register entries, evidence pack.

**CE-002 — Notification dispatch: downstream fault injection.**
- **Hypothesis:** *"If the SMS gateway returns HTTP 503 for 100% of calls from the dispatch service for 90 seconds, the dispatch service will degrade gracefully — queueing messages to the retry topic and reporting healthy — rather than failing calls or retrying synchronously."*
- **Steady state:** dispatch success rate **and** the **outbound call rate to the gateway**. Baseline: 30 min. Tolerance: success ≥ 99.0%; **outbound call rate must not exceed 1.5× baseline** — the retry-storm trip from §4.4.
- **Blast radius:** one caller, one dependency, 90 s. Worst realistic case: **a batch of customer notifications is delayed by up to 15 minutes**, no message loss (the retry topic is durable — verified before the experiment).
- **Abort:** outbound call rate > 1.5× baseline for 30 s (**retry-storm trip**); success rate < 98% for 60 s; gateway owner or on-call calls it.
- **Kill switch:** stop the Toxiproxy toxic / WireMock fault; independent second path = feature flag forcing the service onto the queued path directly.
- **Window:** 10 / 15 / 10. Alert strategy: **correlate** (the gateway monitoring team is notified; the annotation reaches their responder).
- **Findings (illustrative):** (1) the circuit breaker **did** open as hypothesised — the designed mechanism worked; (2) the **fallback path had never been exercised in production and wrote a malformed payload to the retry topic**, which the downstream consumer rejected. **One remediation: an automated contract test on the fallback path, plus a re-run of CE-002 as the verification.** This finding is the single most valuable output of the whole programme — a silent, untested fallback is exactly what a chaos experiment exists to find.

**CE-003 — Card authorisation API: 200 ms latency on a downstream scoring dependency.**
- **Hypothesis:** *"If 200 ms of latency is injected on the egress path to the scoring service for one caller, the 300 ms client timeout and the circuit breaker at 50% error rate will both engage, and authorisations will degrade to the rules-only decision path with no increase in decline rate."*
- **Steady state:** authorisation approval rate, p99 end-to-end latency, **and decline rate**. Baseline: 30 min within a **low-traffic window**, with the traffic distribution confirmed representative. Tolerance: approval rate within 0.2 pp of baseline; p99 ≤ 250 ms; **decline rate must not rise at all** — a rise in declines is a customer-visible harm, so the tolerance is zero movement, not a band.
- **Blast radius:** one caller, one dependency, 200 ms for 5 minutes. Worst realistic case: **an estimated 0.4% of the window's authorisations are declined that would have been approved**, if the fallback misclassifies — which is why the experiment runs in a low-traffic window with an **explicitly capped number of transactions** in scope (illustrative: a maximum of 2,000 authorisations), enforced by the injector's target scoping rather than by hope.
- **Abort:** decline rate rises by **any** measurable amount for 60 s; approval rate falls > 0.5 pp; p99 > 400 ms for 60 s; the duty operations manager calls it — **and the business owner's authority to stop on commercial grounds is explicit** (§9.1).
- **Kill switch:** remove the injected latency; independent second path = a routing toggle at the API gateway that bypasses the scoring call.
- **Window:** 15 / 10 / 15, deliberately shorter than the first two, with a scheduled re-run only after a clean pass. Alert strategy: **verify** — and this is the experiment that also tests whether the institution would *detect* a real degradation of the scoring dependency.
- **Findings (illustrative):** the fallback engaged, but **the rules-only path had a materially different decline behaviour** for a small class of transactions — discovered by the zero-movement decline tolerance doing its job. **Remediation: recalibrate the rules-only path's thresholds, with a re-run as verification.** **No experiment #4 is scheduled until CE-001 through CE-003's remediations are closed and re-verified.**

### 12.3 Environment, Timing, Tooling, Notification

**Environment.** CE-001 and CE-002 run in production (internal-facing services, §11's safe start). CE-003 runs in production in a **low-traffic window** with a capped transaction scope. **Staging is used only for rehearsal** — the 0% and 1% kill-switch runs — and those results are recorded as mechanism validation, never as resilience assurance (§8.3).

**Timing.** All three scheduled **outside** month-end, quarter-end, batch windows, regulatory reporting deadlines and any change freeze. CE-003 additionally scheduled away from scheme cut-off times and peak card-usage hours.

**Tooling selection, with the maintenance-status reasoning (§6).** The function chooses **Chaos Mesh for CE-001** (Kubernetes-native `PodChaos`, Linux Foundation trademark, active ✅), **Toxiproxy and WireMock for CE-002** (both actively maintained ✅✅; Toxiproxy is *test/CI-scoped by design*, so it is used for the staging rehearsal and WireMock for the production HTTP fault), and the provider's **managed fault-injection service (§6.4) for the latency injection in CE-003** where the dependency is a managed service and customer-initiated injection is therefore a **terms-of-service question first** (§10.8). **Deliberately not chosen, and the reasoning recorded:** Chaos Monkey (dormant, Spinnaker-dependent ⚠), Blockade (abandoned ❌), and any tool whose status the function could not verify — **item: Verica ⚠, whose status the programme marks as "verify the domain responds before any procurement conversation."**

**Notification and change control.** Every experiment gets a **change record** (it is a controlled change to production behaviour). Adjacent owners are notified with an objection window: the messaging backbone owner for CE-002, the gateway monitoring team for CE-002, and the duty operations team for CE-003. **Item one, done before CE-001: the compliance function answers in writing whether a planned, controlled, successful experiment falls within the incident-reporting obligation, and how a *failed* experiment would be classified (§10.5).** The programme does not run a production experiment until that written answer exists.

### 12.4 Findings, Remediation and Regulator-Facing Evidence

**Findings and the remediation they forced** (illustrative): a connection-pool recovery defect (CE-001), an alert-threshold defect that hid a real degradation (CE-001), a malformed fallback payload (CE-002), and a fallback decision-threshold defect with customer-visible decline implications (CE-003). **Four findings, four remediations, four re-runs scheduled.** The re-run is the deliverable — §9.4's honest metrics measure closed-and-re-verified, not found.

**The regulator-facing evidence produced:** for each experiment — the design document (scope and the criticality reasoning behind it) · the controls applied to manage risk, including the **abort conditions** and the **kill-switch rehearsal record** · the pinned injector configuration · exported, timestamped dashboards showing steady state, injection, and rollback · the findings with remediation plans, owners and dates · the change records and notification evidence · and a board/committee report. That is §10.7's audit trail in the institution's own format — and the programme's explicit position is that this **supplements** a DORA TLPT/CBEST exercise and **never substitutes** for it (§10.7's authority gap).

### 12.5 Cost, Risk Register, and What the Programme Still Cannot Prove

**Operating cost (all figures illustrative/fictional, annualised, stated in relative terms rather than currency):** roughly **0.5–1.0 FTE** of champion time across the participating services; **~0.3 FTE** of central framework and evidence-pack ownership; **tooling** at either the open-source cost (engineering time to operate) or a mid-band commercial subscription; **~8 scheduled game-day slots per year**; and the **estimated remediation engineering** implied by four findings. The programme's honest business case is not cost avoidance (unquantifiable) but **evidence production** for a supervisor, plus the specific defects the first three experiments found.

**Risk register (illustrative):**

| Risk | Likelihood | Impact | Control |
| --- | --- | --- | --- |
| Experiment causes a customer-visible incident | Low | High | Ramped blast radius; rehearsed kill switch; independent second stop path; business owner's stop authority; low-traffic windows for customer-facing paths |
| Experiment is misreported as an incident | Medium | Medium | Service management notified in advance; change record; annotation reaching responders (§7) |
| Finding is not remediated and the programme loses credibility | Medium | High | Remediation owner named **before** the run; findings closed and **re-verified**; §9.4 metrics reported to leadership |
| Blast radius under-estimated due to an unknown dependency | Medium | High | Negative containment list; dependency discovery; capped transaction scope on customer-facing paths |
| A tool goes dormant mid-programme (§6) | Medium | Medium | Pin versions; prefer foundation-governed projects; re-check maintenance status at each annual review |
| First adverse finding kills the programme | Medium | High | §9.2's leadership sponsorship; blameless framing of the first report |

**What the programme STILL CANNOT PROVE, stated honestly:** that the institution would **survive a real region-level failure** (never tested, and CE-001–003 do not approach it); that the **core ledger** is resilient (deferred — no constructible steady state); that **payment settlement** paths degrade safely (deferred pending a supervisor conversation); that the **fallback paths on every service** are correct (only two were tested); that the institution can **detect** a slow-onset degradation (only CE-003 touches detection, and only in one form); and that the institution is **in control** in the sense a supervisor means when it asks the question — because the programme's evidence is four experiments, not an estate-wide assurance. **The programme's one-page summary to the board says exactly this**, which is why the board keeps funding it.

## 13. The Anti-Patterns

Each with **symptom → cause → guardrail**. The names are mine; the failures are universal.

1. **Chaos theatre.** *Symptom:* an experiment with no written hypothesis, no abort condition, no finding, run for a demo or a slide. *Cause:* the practice was adopted as a *brand* rather than a *method*, usually after a conference talk. *Guardrail:* no experiment starts without a hypothesis and an abort condition (§4.1); the artefacts are mandatory, and an experiment with no artefacts did not happen.
2. **The unmonitored experiment.** *Symptom:* a fault injected into a system nobody was watching, producing a finding nobody can describe. *Cause:* tooling-first adoption, taking the injector's success as the result. *Guardrail:* §7 — no observability, no experiment; the control room is staffed by a non-author (§9.1).
3. **Injecting into a system whose steady state was never defined.** *Symptom:* the result is an opinion ("it seemed fine"). *Cause:* skipping §3.4's five-question audit. *Guardrail:* the audit is a gate; a system that fails it goes to the instrumentation backlog instead.
4. **The checkbox compliance programme.** *Symptom:* "we do chaos engineering" + one experiment per year, repeated unchanged, findings closed as "no impact". *Cause:* the programme exists to produce an artefact for a regulator or an audit, not to find anything. *Guardrail:* §9.4's metrics — the % of experiments producing a remediation, and repeat findings. **A programme with zero findings after three experiments is not a safe bank; it is a badly designed programme** (see #5).
5. **Findings with no remediation owner.** *Symptom:* a findings register that grows; engineering improvement that never happens; the same weakness found again next year. *Cause:* the programme has the authority to *test* but not to *change* — the actual organisational constraint in most banks. *Guardrail:* the remediation owner is named **before** the experiment runs (§8.5), and repeat findings are escalated as a trend (§9.4).
6. **Confusing a DR drill or a load test with chaos engineering.** *Symptom:* confident resilience claims sourced from a failover test or a peak-volume test. *Cause:* the §1.2 distinctions were never drawn, usually because a single undifferentiated "resilience testing" label (as the market is now promoting, §6.2) encourages the blur. *Guardrail:* state in the record, per exercise, whether it proves **recovery**, **capacity** or **absorption**.
7. **Buying the tooling before building the practice.** *Symptom:* a licence, a control plane, an onboarding call, and no experiments. *Cause:* procurement as a substitute for the harder work of steady-state definition and blast-radius discipline. *Guardrail:* the §6.1 maintenance check *and* the §12.1 scoping exercise run first — and note that the open-source options verified in §6.3 are entirely sufficient for stages 1–3 of §9.3.
8. **The experiment that became the incident.** *Symptom:* §8.4. *Cause:* an unrehearsed kill switch; an abort condition set too loose; a blast radius larger than the arithmetic suggested. *Guardrail:* §8.1's business-exposure computation, §8.2's rehearsal, and the pre-flight checklist enforced without exception — **including when the schedule is tight.** The pressure to skip the checklist comes precisely when the risk is highest.
9. **The culture failure where the first adverse finding kills the programme.** *Symptom:* experiment #1 finds a real defect; the defect's owner is embarrassed; the programme is quietly not resourced for the next cycle. *Cause:* §9.2 — a blame culture in which an adverse finding is a career event. *Guardrail:* leadership sponsorship stated in writing before experiment #1, findings attributed to systems not people, and — the practical version — the first game day's report framed around **remediation value**, not around the defect. **This is the anti-pattern that most often ends programmes, and the one most often omitted from the lists.**

**Cross-reference rather than duplicate:** the repository's wider reliability and failure-culture literature covers post-incident review and the blameless-postmortem mechanics — this section deliberately keeps to the anti-patterns specific to *this* practice and cross-refs [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) for the resilience-design patterns whose absence those experiments often expose.

## 14. The Claims Audit

Source quality is graded: **P** = primary document (regulation, specification, repository, publication record, official regulator page); **V** = a vendor's claim about its own tool; **T** = third-party summary (news, consultancy explainer, directory listing). All dates are the date of verification, which for this pass is **17 September 2026** unless noted.

| Claim | Verdict | Source | Quality | Verified |
| --- | --- | --- | --- | --- |
| "The Netflix Simian Army" (19 Jul 2011, Izrailevsky & Tseitlin) is the origin document | ✅ | Netflix Tech Blog; text retrieved via an archived mirror citing the original URL; date corroborated by the mirror's own slug | P (via T mirror) | 17 Sep 2026 |
| The 2011 post named **exactly eight** simians: Chaos Monkey, Latency, Conformity, Doctor, Janitor, Security, 10-18, Chaos Gorilla — of which only three inject faults | ✅ | The post's own text | P | 17 Sep 2026 |
| "Chaos Kong" appeared in the 2011 post | ❌ rejected | Not present in the post's text | P | 17 Sep 2026 |
| The Principles of Chaos has **five** advanced principles, incl. "Minimize Blast Radius"; last updated 2019 March; community-owned, not Netflix-owned | ✅ | principlesofchaos.org + the `chaos-eng/chaos-eng.github.io` repository | P | 17 Sep 2026 |
| The sibling guide's **four**-principle list is correct | ❌ rejected | Same source — this guide's §2.3 carries the correction | P | 17 Sep 2026 |
| *Chaos Engineering: System Resiliency in Practice*, Rosenthal & Jones, O'Reilly, 6 Apr 2020, 308pp, ISBN 9781492043836 | ✅ | Google Books catalogue record | P (bibliographic DB) | 17 Sep 2026 |
| LitmusChaos: CNCF Incubating (per CNCF page, joined 25 Jun 2020, Incubating 11 Jan 2022); active (release 3.32.0, 17 Sep 2026). Chaos Mesh: active (2,002 commits), Apache-2.0, Linux Foundation trademark | ✅ | cncf.io project page + both repositories | P | 17 Sep 2026 |
| ChaosBlade: active, Alibaba-originated, Apache-2.0; release `blade-ai-v0.7.0` 25 Aug 2026, last commit 28 Jul 2026. Chaos Toolkit and Pumba: active, Apache-2.0 | ✅ | Repository releases, commits and READMEs | P | 17 Sep 2026 |
| Toxiproxy: active, MIT, Shopify production use "since October, 2014" | ✅ | Repository README | P | 17 Sep 2026 |
| Chaos Monkey's repository is **archived** | ❌ rejected | Public and unarchived — **but dormant**: 165 commits, Travis CI badges, zero releases, Spinnaker required | P | 17 Sep 2026 |
| Blockade is maintained | ❌ rejected | `cloudfoundry-incubator/blockade` returns a hard 404; the surviving personal-account repo carries a discontinued-CI badge and an unfinished-work file | P | 17 Sep 2026 |
| AWS FIS is live, part of AWS Resilience Hub, and was formerly named "Fault Injection Simulator"; Azure Chaos Studio's GA model is feature-frozen while its active model is in **public preview**, not intended for production | ✅ | AWS product page (current name + launch-blog URL slug); Microsoft Learn, page updated 06 Sep 2026 | P | 17 Sep 2026 |
| Verica is a live commercial option; Gremlin funding = $26.8M; Steadybit funding = €13M; Harness renamed Chaos Engineering → "Resilience Testing" | ⚠ flagged | `verica.io` failed extraction; third-party funding trackers; a third-party summary of a vendor change — none corroborated by a primary source | T | 17 Sep 2026 |
| DORA = Regulation (EU) 2022/2554, adopted 14 Dec 2022, **applies from 17 January 2025** | ✅ | DORA Article 64 text, read via a full-text mirror (`digital-operational-resilience-act.com`); **EUR-Lex blocked automated extraction 17 Sep 2026** | P (via T mirror) | 17 Sep 2026 |
| TLPT: at least every 3 years; covers critical or important functions; performed **on live production systems**; scope validated by the competent authority; external testers every three tests; significant credit institutions external only | ✅ | DORA Article 26 text, via the same full-text mirror | P (via T mirror) | 17 Sep 2026 |
| Tester conditions: highest suitability, threat-intel/pen-test/red-team expertise, accreditation, independent assurance, professional indemnity insurance; internal testers need supervisory approval and an external threat-intelligence provider | ✅ | DORA Article 27 text, via the same full-text mirror | P (via T mirror) | 17 Sep 2026 |
| TLPT RTS: in force, application date 08/07/2025, OJ CELEX 32025R1190; drafted per TIBER-EU with ESAs + ECB, drafts due 17 Jul 2024 | ✅ | EBA single-rulebook page + DORA Article 26(11) | P | 17 Sep 2026 |
| CBEST: BoE's threat-intelligence-led security assessment framework, conducted in "live corporate environments"; 2025 thematic published | ✅ | Bank of England pages | P | 17 Sep 2026 |
| MAS revised BCM Guidelines issued **6 June 2022**; MAS BCM 2022 reportedly supersedes the June 2003 version ⚠ | ✅/⚠ | MAS published PDF ("BCM-Guidelines-June-2022"); supersession from third-party summaries only | P / T | 17 Sep 2026 |
| ABS **AASE** ("Red Teaming") Guidelines, **14 November 2018**, with MAS support | ✅ | ABS PDF media release + the ABS guidelines document | P | 17 Sep 2026 |
| BCBS **Principles for Operational Resilience, March 2021** (issued 31 Mar 2021) | ✅ | bis.org publication page + PDF; FSB and HKMA corroborate the date | P | 17 Sep 2026 |
| A specific chaos experiment caused a named production outage; a named bank runs an in-house chaos platform; cloud providers' terms permit customer-initiated fault injection; any adoption-rate statistic | ⚠ flagged | Searches returned **empty result sets** for the first three; every adoption figure encountered was a vendor claim without a stated sample or method | — / V | 17 Sep 2026 |

**The highest-risk fact classes, and how this pass treated them:** (1) **tool maintenance status** — the single most likely claim to be stale, and the reason every entry in §6 was checked against the *repository*, not the docs site; (2) **the principles' exact wording and count** — the reason one sibling correction was made; (3) **origin dates** — the reason the Simian Army date was corroborated twice rather than taken from a secondary summary; (4) **regulatory provisions** — the reason Article 26 was read as article text rather than paraphrased, and the reason the RTS was confirmed on the regulator's own page.

## 15. What Could Not Be Verified

Stated plainly, because a negative finding is a finding:

- **A specific, datable case of a chaos experiment causing a real production outage.** Multiple targeted searches returned **empty result sets** ✅. No incident, vendor, or provider outage has been attributed to a chaos experiment anywhere in this guide.
- **Cloud provider acceptable-use / security-testing terms on customer-initiated fault injection.** Not retrieved. §10.8 treats this as an open question requiring written authorisation, not a settled answer.
- **Whether Google Cloud offers a managed fault-injection service, and its name/status.** Not established; no product named.
- **Verica's current operating status, Gremlin's funding/ownership, and the date of Harness's Chaos Engineering → Resilience Testing rename.** The Verica domain did not respond to extraction — a retrieval failure, not proof the company has ceased operations; the Gremlin figure is a third-party tracker value; the Harness rename is a third-party summary.
- **Chaos Mesh's current stewards and its relationship to PingCAP.** The repository's `MAINTAINERS.md` and `GOVERNANCE.md` are the place to look; my searches returned nothing.
- **Chaos Monkey's original release date** (as distinct from the 19 July 2011 Simian Army post).
- **Exact last-commit and latest-release dates for Chaos Toolkit, Pumba and Chaos Mesh** — release tags and READMEs were read instead of commit timestamps.
- ***Learning Chaos Engineering* (Russ Miles) publication data**, and whether a separate pre-2020 O'Reilly *Chaos Engineering* title by Rosenthal exists.
- **The DORA incident-reporting RTS numbers** cited by the sibling guide (2024/1772, 2025/301), and its **MAS TRM / outsourcing / incident-reporting items 2023–2026** — recorded as attributed, not independently re-verified here. Likewise **CBEST's framework version history and current mandated population.**
- **EUR-Lex as a direct source for the DORA text.** EUR-Lex refused automated extraction on 17 September 2026 (all scraping engines failed) — re-confirmed by this guide's own check as well as recorded by the sibling guide. The DORA articles in §10.1 were therefore read through a full-text mirror, and any provision you intend to rely on in a regulatory submission should be read on EUR-Lex in a browser.
- **The Principles of Chaos document's first-publication year.** Only its last revision (March 2019) is verified; the commonly quoted 2015 origin could not be confirmed at a primary source this pass.
- **Any verified adoption-rate statistic for chaos engineering.** Every figure encountered was a vendor claim without a stated sample or method, and none is cited here.
- **A named bank's in-house chaos tooling**, and **whether a planned, controlled experiment falls inside a firm's incident-reporting obligation** — the latter is a legal and compliance question, which §12.3 treats as a mandatory written determination rather than an engineering assumption.
- **Any 2026-dated status for the Simian Army tools other than Chaos Monkey.** Latency Monkey, Chaos Gorilla and the rest were not published as maintained products under those names, and I make no claim about any repository.

## 16. Glossary

| Term | Definition |
| --- | --- |
| **Chaos engineering** | "The discipline of experimenting on a system in order to build confidence in the system's capability to withstand turbulent conditions in production" — principlesofchaos.org ✅ |
| **Steady state** | A measurable output indicating normal behaviour; express as a band with a baseline, tolerance, and stated lag |
| **Hypothesis** | The falsifiable claim that the steady state will hold in both control and experimental groups while a variable is introduced |
| **Control group / experimental group** | The comparison structure the method requires — identical conditions, one differing variable |
| **Blast radius** | The set of users, transactions and systems that can be affected if the experiment goes wrong; best expressed as *business exposure*, not just a percentage |
| **Fault injection** | The mechanical act of introducing a variable (killing an instance, adding latency). The means; not the method |
| **Abort condition / kill switch** | A pre-declared, machine-checkable threshold that stops the experiment, and the *tested* mechanism that executes the stop and reverses the fault (which must be independent of the injector) |
| **0% run / blast-radius ramp** | The 0% run applies the fault to nothing, to rehearse the kill switch and observation; the ramp is the staged progression 0% → 1% → 5% → 25%, holding each level until an observation window passes clean |
| **Game day** | A scheduled, facilitated exercise with defined roles, a control room, and a written record |
| **Chaos theatre** | Running an experiment for the appearance of the practice, with no hypothesis, abort or finding |
| **Simian Army (Chaos Monkey, Latency Monkey, Conformity, Doctor, Janitor, Security, 10-18, Chaos Gorilla)** | Netflix's family of resilience tools announced July 2011; eight named in the original post, of which only Chaos Monkey, Latency Monkey and Chaos Gorilla inject faults. **Chaos Gorilla** simulates an availability-zone outage |
| **Chaos Kong** | A later, widely-cited region-failure tool; **not** named in the 2011 post |
| **Kubernetes-native frameworks (LitmusChaos, Chaos Mesh)** | LitmusChaos is a CNCF Incubating platform with a ChaosHub experiment library; Chaos Mesh is a Kubernetes-native `Chaos` CRD operator and a Linux Foundation trademark |
| **ChaosBlade, Chaos Toolkit** | An Alibaba-originated toolkit (currently re-pointing toward AI-agent features) and a Python, driver-based, platform-agnostic framework |
| **Proxy / container injectors (Toxiproxy, Pumba, WireMock)** | Toxiproxy is Shopify's TCP proxy for deterministic network faults, test/CI scoped by design; Pumba is container-level chaos for Docker/containerd/Podman, Linux only by design; WireMock is an HTTP mock server used for API-level dependency fault injection |
| **Cloud managed services (AWS FIS, Azure Chaos Studio)** | AWS Fault Injection Service (formerly Fault Injection Simulator), part of AWS Resilience Hub; Azure Chaos Studio has Workspaces/Scenarios (preview) and Experiments (classic, frozen) |
| **Commercial platforms (Gremlin, Harness, Steadybit, Verica)** | Gremlin positions as enterprise reliability management; Harness's chaos module has been renamed and absorbed into "Resilience Testing"; Steadybit markets self-hosted/SaaS experiments; Verica's status is unverified §6.2 |
| **TLPT** | Threat-Led Penetration Testing — DORA Article 26's advanced testing regime |
| **CBEST** | The Bank of England's threat-intelligence-led security assessment framework |
| **AASE** | Adversarial Attack Simulation Exercises — the ABS (Singapore) "red teaming" guidelines |
| **DORA** | Regulation (EU) 2022/2554; applies from 17 January 2025 |
| **TIBER-EU** | The ECB's threat-intelligence-based ethical red-teaming framework; the basis for the DORA TLPT RTS |
| **Pooled testing / attestation** | A service provider running a joint TLPT for several financial entities (Art. 26(4)); and the supervisory confirmation that a TLPT met the requirements, enabling mutual recognition |
| **Impact tolerance** | The maximum tolerable disruption to an important business service (PRA/FCA framing) — covered by the sibling operational-resilience guide |
| **False assurance / drift blindness** | Passing an experiment because the steady state was already degraded or the signal cannot see the failure; and an automated experiment that stays green while the architecture beneath it changes |

## 17. Cross-References and Further Reading

**Within this repository.** The summary this guide expands: [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) **§7** (lines 420–461 — the Chaos Monkey, the Principles of Chaos, GameDays, the chaos table, the blast-radius ramp), whose §7.2 four-principle list **this guide corrects to five** (§2.3). That guide also owns the availability math (§2), the SLO/error-budget machinery (§2.3), the deployment strategies (§3), the resilience patterns (§8) and the 24-7 payments switch (§9) — all cross-referenced, none re-derived. The regulatory mechanics: [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §2 (landscape), §4–5 (BIA and impact tolerances), §6 (severe-but-plausible scenarios), §8 (DR), §11 (testing and assurance). Platform chaos and DR drills: [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) §28–§29. Netflix at cloud scale: [architecture/billion_user_system_arch.md](architecture/billion_user_system_arch.md) §5.3. The estate: [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md), [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md), [../banking/nets_singapore_guide.md](../banking/nets_singapore_guide.md), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/ibps_payment_connect_guide.md](../banking/ibps_payment_connect_guide.md), [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md), [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md).

**Agent-side fault injection.** I found **no dedicated LLM/agent fault-injection guide** in `ai_llm/` in this pass. The nearest neighbours are [ai_llm/llm_agents_failures_production_guide.md](ai_llm/llm_agents_failures_production_guide.md) (how agents fail in production) and [ai_llm/ai_red_teaming_guide.md](ai_llm/ai_red_teaming_guide.md) (adversarial testing of AI systems) — both are adjacent to, not substitutes for, agent-side chaos. ⚠ If an agent fault-injection guide exists elsewhere, treat it as the primary cross-reference.

**Primary sources cited in this guide:** principlesofchaos.org (extracted live, 17 Sep 2026) and its generating repository `chaos-eng/chaos-eng.github.io`; the Netflix Simian Army post (19 July 2011) via an archived mirror of `techblog.netflix.com/2011/07/netflix-simian-army.html`; the O'Reilly/Google Books record for *Chaos Engineering: System Resiliency in Practice*; the live repository pages of Netflix/chaosmonkey, chaos-mesh/chaos-mesh, litmuschaos/litmus, chaosblade-io/chaosblade, chaostoolkit/chaostoolkit, alexei-led/pumba, Shopify/toxiproxy, wiremock/wiremock and worstcase/blockade; cncf.io/projects/litmus; the AWS Fault Injection Service product page; Microsoft Learn's Azure Chaos Studio overview (page updated 06 Sep 2026); gremlin.com; Regulation (EU) 2022/2554 Articles 26, 27 and 64; the EBA single-rulebook page for the joint TLPT RTS (OJ CELEX 32025R1190); the Bank of England CBEST implementation-guide and 2025-thematic pages; the MAS Business Continuity Management Guidelines (June 2022) and the ABS AASE guidelines (14 November 2018); and the BIS publication page for the BCBS Principles for Operational Resilience (March 2021).

**Further reading, in the order I would read it:** the Principles document in full (it is one page — read it rather than a summary of it); the 2011 Simian Army post (for the framing, which is more cautious than its reputation); the *Chaos Engineering* book's "Human Factors" part; the DORA Article 26 and Article 27 text (the only place the testing obligation is stated authoritatively); and the experiment records of whatever programme you are joining — because the artefacts of a real programme are more informative about its health than any framework document.

## 18. Closing Summary

Chaos engineering is a small discipline with a large claim: that the only way to know whether a system survives failure is to make it fail, deliberately, under observation, with a hypothesis. That claim is correct, and it is why the practice outlived the tools that founded it — Netflix's Simian Army has largely gone dormant, most of the original tooling is history, and the canon's own defining document has not been revised since March 2019, yet the method is in better shape than ever because the method was always the portable part. What travels is not Chaos Monkey; it is the nine steps of §4, the taxonomy of §5, the guardrails of §8, and the culture of §9.

Three conclusions I would defend. **First, the discipline's hardest problem is not tooling but the steady state** — a programme cannot find what it cannot see, and §3.4's five-question audit is more consequential than any injector choice. **Second, in a regulated institution the practice's value is evidence as much as resilience** — the artefacts a well-run experiment produces are, almost by accident, shaped like what a supervisor asks for, and that is the business case I would lead with, while being honest that a chaos programme can never produce the supervisory *attestation* that a DORA TLPT or a CBEST exercise does. **Third, the ceiling is real and should be stated out loud** — for a large bank that means stage 3 of the maturity ramp, carefully-scoped excursions beyond it, a rehearsed kill switch every time, and a board report that says plainly what the programme cannot yet prove.

And the discipline's own test is unforgiving, which is why it is worth doing: an experiment that finds nothing, changes nothing and is repeated unchanged is not evidence of a resilient system. It is evidence of a programme that has learned to avoid finding anything. The only honest measure is the remediation you can point at and the re-run that proves it worked — which is the difference between breaking things on purpose and simply breaking things, and the whole distance between a stunt and a discipline. What the practice is really buying is a system you can trust precisely because you have seen it fail on purpose, and the thing it protects is the steady state.
