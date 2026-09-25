# a16z Big Ideas 2026 — Reading a Prediction List as a Position

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research) · **Series:** Technology Research Guides · **Version:** 1.0 · **Date:** September 2026
> **Context:** Technology Research — the a16z "Big Ideas 2026" edition read properly: the edition and its structure (three newsletter parts, December 9–11 2025), the full inventory of 47 ideas with their bylined authors, the authorship and what it reveals, the genre read (of what class of artefact this list is), the comparison against the 2025 omnibus and the 2023/2024 editions, the accountability question and why no hit-rate can honestly be computed, the enterprise and regulated-institution read, the self-interest map (sourced and dated), a practical method for reading any venture firm's annual list, a critical reading of five of the edition's ideas, adjacent artefacts, anti-patterns, a claims audit, an explicit unverified list, and a glossary with cross-references

> **Companion guides (cross-reference, do not re-derive):** the VC-framing and VC-terminology genre is already owned by [ai_native_companies_guide.md](ai_native_companies_guide.md) — its §2.2 covers the coinage and provenance of "AI-native" and already notes that "a16z, Sequoia, and Y Combinator all used AI-native framing in this window," flagging first use as unverified; this guide therefore does **not** re-derive that terminology thread and treats it as cited prior work. The enterprise-adoption discipline is in [ai_adoption_strategies_guide.md](ai_llm/ai_adoption_strategies_guide.md); the enterprise-AI platform discipline in [enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) and [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md); the agent-execution discipline in [agentic_workflows_guide.md](agentic_workflows_guide.md); the evaluation discipline in [llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md); the agent-era security discipline in [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md); the unit-economics discipline in [finops_guide.md](finops_guide.md); the capacity-planning discipline in [capacity_sizing_guide.md](capacity_sizing_guide.md); and the data-platform discipline in [data_fabric_guide.md](data/data_fabric_guide.md).

---

**Table of Contents**

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder)
2. [The Edition and Its Structure](#2-the-edition-and-its-structure)
3. [The 2026 Ideas, by Vertical, with Authors](#3-the-2026-ideas-by-vertical-with-authors)
4. [The Authorship and What It Reveals](#4-the-authorship-and-what-it-reveals)
5. [What This Artefact Actually Is — The Genre Read](#5-what-this-artefact-actually-is--the-genre-read)
6. [The Comparison with the Prior Edition](#6-the-comparison-with-the-prior-edition)
7. [The Accountability Question](#7-the-accountability-question)
8. [The Enterprise and Regulated-Institution Read](#8-the-enterprise-and-regulated-institution-read)
9. [The Self-Interest Map](#9-the-self-interest-map)
10. [How to Read a Venture Firm's Annual List](#10-how-to-read-a-venture-firms-annual-list)
11. [The 2026 Ideas Read Critically](#11-the-2026-ideas-read-critically)
12. [The Adjacent Artefacts](#12-the-adjacent-artefacts)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary, the Cross-References and the Closing Summary](#16-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Identity and the Decoder

**Thesis in one line: a venture firm's prediction list is not a forecast; it is a position — which does not make it useless, it makes it something specific, and reading it as what it is, is the whole job.**

That sentence is doing real work, so it is worth unpacking before the guide proceeds.

A **forecast** is a claim about the world that can be graded against the world. A **position** is a claim about where the claimant has placed resources. The two can use identical grammar — "AI-native infrastructure becomes table stakes in 2026" reads like a prediction — but they are answered by different evidence. A forecast is answered by whether infrastructure became table stakes. A position is answered by whether the speaker's portfolio, hiring, funds and public commitments were already concentrated there. a16z's annual Big Ideas list is legible as both, and the edition itself never separates them. This guide is the separation.

The reframing matters because it changes what a reader can usefully do with the artefact. If you read it as a forecast, you will try to grade it, will fail (there is no method that grades 47 one-line theses across seven verticals on a one-year horizon — see §7), and will end up either credulous or dismissive. If you read it as a position, you can do something concrete with it: you can learn what one of the best-capitalised investors in technology is publicly committed to, you can treat it as a demand-side inventory of what the market will spend the next year selling, and you can re-read it deliberately against outcomes later. Those are modest, defensible uses. The grandiose use — treating it as a market call from an oracle — is not available, and this guide argues it never was.

### 1.1 The decoder

Read the list with these terms held distinctly. Almost every misreading in the wild comes from collapsing two rows of this table.

| Term | What it literally denotes | What to watch for when reading |
| --- | --- | --- |
| **Omnibus edition** | A single a16z article page containing every vertical's idea for that year — the 2023, 2024 and 2025 format | Looks like a magazine feature; one URL, one date, one byline ("a16z editorial") |
| **Newsletter series** | Multiple separate a16z newsletter pages published on consecutive days, together comprising the year's edition — the 2026 format | No omnibus page exists; dates differ per part; each page has its own navigation, promo cards and subscribe prompts |
| **Vertical** | The firm's own organisational practice grouping under which ideas are filed — Infrastructure, Growth, Bio + Health, Speedrun, American Dynamism, Apps, crypto in 2026 | Verticals are org units, not market segments (§4); a "vertical" heading in the artefact is a management chart, not an industry taxonomy |
| **Contributor / partner byline** | The individual a16z person (or guest) whose one idea it is, named directly under the idea title | The page byline is "a16z New Media"; the **idea** byline is a person. Both are true and they are not the same thing |
| **Thesis** | The claim the contributor makes about what builders will tackle, or what partners observe, in the year ahead | Demand-side signal about what will be sold; not a statement about what any institution should buy (§8) |
| **Portfolio** | The set of companies a16z funds have invested in | A firm's own investment list is published at `a16z.com/investments/`; ideas and holdings overlap (§9) |
| **Anti-portfolio** | Companies the firm considered and passed on, or lost the deal for | Never published in these editions; its absence is a structural feature of the genre (§5) |
| **Fund** | A pooled investment vehicle managed by the firm, raised from LPs, deployed into portfolio companies | The artefact is published by the adviser to those vehicles; the fund is the economic unit the list ultimately serves (§5, §9) |
| **LP** | A limited partner — an institution or individual that commits capital to a fund | The list's secondary audience; a public thesis statement is also LP communication (§5) |
| **"Big idea" as a unit of content** | One contributor, one title, one short essay, one bio block — a deliberately bounded publishing unit | The bound is editorial, not analytical: an idea is one partner's view at one moment, never a firm-wide position (§13) |

### 1.2 The boundary with the sibling guides

This guide owns the **artefact**: what a16z published, when, by whom, in what structure, and how to read it as a genre. It does **not** re-derive:

- the **coinage and provenance of "AI-native"** as a term — owned by [ai_native_companies_guide.md](ai_native_companies_guide.md, "AI-Native Companies"), whose §2.2 already covers the term's use by a16z, Sequoia and Y Combinator in this window and flags first use as unverified;
- the **enterprise adoption maturity and organisational models** — owned by [ai_adoption_strategies_guide.md](ai_llm/ai_adoption_strategies_guide.md);
- the **enterprise-AI platform architecture** — owned by [enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) and [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md);
- the **agent-execution and agent-security disciplines** — owned by [agentic_workflows_guide.md](agentic_workflows_guide.md) and [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md);
- the **evaluation discipline** — owned by [llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md);
- the **model-serving unit economics** — owned by [finops_guide.md](finops_guide.md) and [capacity_sizing_guide.md](capacity_sizing_guide.md);
- the **data-platform substrate** the Infrastructure ideas presuppose — owned by [data_fabric_guide.md](data/data_fabric_guide.md).

Where this guide touches those subjects it names the sibling and stops. The one substantive crossover the guide does take on is §8, which asks which of the 47 ideas are *decision-relevant* to an enterprise technology function — a question about the artefact's usefulness to a reader, not about the technology itself.

---

## 2. The Edition and Its Structure

### 2.1 What was published, when, and where

The 2026 edition of a16z's Big Ideas package is **not a single article**. It is a three-part newsletter series, published on three consecutive days in December 2025:

| Part | Title | Page byline | Post date | URL | Ideas |
| --- | --- | --- | --- | --- | --- |
| 1 | Big Ideas 2026: Part 1 | a16z New Media | **December 9, 2025** | `a16z.com/newsletter/big-ideas-2026-part-1/` | 14 |
| 2 | Big Ideas 2026: Part 2 | a16z New Media | **December 10, 2025** | `a16z.com/newsletter/big-ideas-2026-part-2/` | 16 |
| 3 | Big Ideas 2026: Part 3 | a16z New Media | **December 11, 2025** | `a16z.com/newsletter/big-ideas-2026-part-3/` | 17 |
| — | **Total** | | December 9–11, 2025 | | **47** |

All three pages carry the byline "a16z New Media" and the date stamp. Each part also carries a trailing "About the Contributor / a16z New Media" block, which is the **page** byline and is not an idea entry — a distinction that matters for counting (§3.8).

### 2.2 The verticals and how the parts divide them

| Part | Pages states its teams as | Verticals appearing as headings | Ideas |
| --- | --- | --- | --- |
| 1 | "our Infrastructure, Growth, Bio + Health, and Speedrun teams" | Infrastructure (6), Growth (4), Bio + Health (1), Speedrun (3) | 14 |
| 2 | "our American Dynamism and Apps teams" | American Dynamism (6), Apps (10) | 16 |
| 3 | "various a16z crypto partners (plus a few guest contributors)" | No vertical headings; 17 ideas in a single sequence | 17 |

Seven team labels in total: Infrastructure, Growth, Bio + Health, Speedrun, American Dynamism, Apps, crypto. Note that Part 3 is structurally different from Parts 1 and 2: it is a flat numbered-in-order sequence with **no internal vertical headings at all**, because all 17 come from the same practice (the crypto team, plus guests). The heading `<h4>`-level "Apps" in Part 2 is placed above David Haber's entry — that boundary was verified in the cached page text.

### 2.3 The method statement each part gives

Each part states its own method, and the three statements are not identical. Quoting exactly:

**Part 1 (December 9, 2025):**

> "Our job as investors is to immerse ourselves in the ins-and-outs of every corner of the tech industry in order to understand where things are moving next. So every December, we ask our investing teams to share one big idea they think tech builders will tackle in the year to come."

Part 1 adds: "Today, we're sharing ideas from our Infrastructure, Growth, Bio + Health, and Speedrun teams."

**Part 2 (December 10, 2025):**

> "Yesterday, we shared the first installment of our Big Ideas package, which included the problems that our Infrastructure, Growth, Bio + Health, and Speedrun partners think startups will tackle in 2026."

and

> "We're back with part 2 of this package today, featuring contributions from our American Dynamism and Apps teams. Stay tuned for ideas from our Crypto team tomorrow."

**Part 3 (December 11, 2025):**

> "Over the past two days, we've shared the ideas our Infrastructure, Growth, Bio + Health, Speedrun, Apps, and American Dynamism teams think builders will tackle in 2026."

and — the wording that matters most —

> "Today, we're sharing are 17 things that various a16z crypto partners (plus a few guest contributors) observe about what's ahead — on topics ranging from agents and AI; stablecoins, tokenization, and finance; privacy and security; to prediction markets, SNARKs, and other applications… to how we'll build."

Three specific things to notice about that last sentence, because they are load-bearing:

1. It says **"observe"**, not "predict" — and not "will tackle". The crypto part deliberately uses weaker epistemic framing than Parts 1 and 2.
2. It says **"17 things"**, not "17 ideas" — again a different unit word.
3. It says **"(plus a few guest contributors)"** — acknowledging that Part 3 is not purely internal staff. Four of the 17 Part-3 contributors have bios that identify them as founders or executives of outside organisations (see §4.2).

Part 3 also closes with a forward promise: "And then tomorrow, we'll finish the week with a special announcement and an invitation from a16z that you won't want to miss." That is a statement about the artefact's function as a sequence, not about its content (§5, §12).

### 2.4 Primary-source provenance

This guide was written from three cached full texts of the edition's pages, read directly with text tools: `/tmp/a16z_2026_part1.txt` (Part 1, 32,766 characters, 465 lines), `/tmp/a16z_2026_part2.txt` (Part 2, 34,077 characters, 520 lines) and `/tmp/a16z_2026_part3.txt` (Part 3, 45,144 characters, 509 lines). Plus the prior-year comparison text, `/tmp/a16z_2025_omnibus.txt` (72,684 characters, 682 lines), and the web-extractor caches under `/home/ubuntu/.hermes/cache/web/` for the 2023, 2024 and 2025 omnibus pages, the three 2026 newsletter pages, the firm's public investment list, and a Wikipedia page used for one §7 check.

Every idea title, author name and quotation in §3, §4, §5, §6, §9 and §11 is traceable to one of those files. Nothing in this guide is reconstructed from memory of the 2026 edition.

### 2.5 Method and sources — and the tool limitations, stated plainly

This guide was produced in an environment with specific, verified limitations, and it would be dishonest to present the result without them.

- ✅ **`web_extract` worked against a16z.com.** The 404 on the omnibus URL, the three newsletter pages and their dates were re-verified directly in this pass, and full clean page text was cached to disk.
- ⚠️ **`web_search` returned empty result sets on every query attempted in this pass.** No search-engine-sourced reporting, commentary, analyst note, or critic's response to the 2026 edition was available to this guide. Every claim here comes from the publisher's own pages, the firm's own investment list, or a named general reference (Wikipedia) for one checkable fact.
- ⚠️ **The primary source was obtained as flattened page text, not as structured HTML.** Consequence: link targets inside the essays are not preserved in the cached text, and the extractor's rendering sometimes collapses list structure. Where an essay embeds an internal list (Jason Cui's entry, §11.5) that list is reproduced as it appears in the flattened text. Where a sentence contains a placeholder from the source's own editorial tooling (Andrew Hall's entry contains the literal string `<SC will link to our article here>`) that is a defect of the *published page*, not of this transcription, and is noted as such.
- ⚠️ **No independent body audits these lists** (§7). The absence of an audit is not this guide's finding about a16z specifically; it is a fact about the class of artefact.
- ⚠️ **Where this guide cannot verify something, §15 says so explicitly.** Absence of evidence was never treated as evidence of absence.

One more limitation worth stating in the method note rather than the appendix: the cached texts were captured at different times and the live pages are not frozen. The 2026 newsletter pages now render a "Table of Contents" navigation block that the flattened working copies do not contain; this is normal for a live CMS and does not affect the idea inventory, which was verified against both the flattened text and the live-extracted page.

---

## 3. The 2026 Ideas, by Vertical, with Authors

This is the content core. Everything below is read out of the three cached part texts. Titles are given exactly as published, including the edition's own punctuation and quotation marks. Part and date are given for every entry so the inventory can always be re-checked against the primary source.

### 3.1 Part 1 — December 9, 2025 (14 ideas)

| # | Vertical | Idea title | Bylined author |
| --- | --- | --- | --- |
| 1 | Infrastructure | Startups tame the chaos of multimodal data | Jennifer Li |
| 2 | Infrastructure | AI revives cybersecurity hiring | Joel de la Garza |
| 3 | Infrastructure | Agent-native infrastructure becomes table stakes | Malika Aubakirova |
| 4 | Infrastructure | Creative tools go multimodal | Justine Moore |
| 5 | Infrastructure | The AI-native data stack continues to evolve | Jason Cui |
| 6 | Infrastructure | The year we step inside video | Yoko Li |
| 7 | Growth | Systems of record lose ground | Sarah Wang |
| 8 | Growth | Vertical AI evolves from information retrieval and reasoning to multiplayer | Alex Immerman |
| 9 | Growth | Creating for agents, not humans | Stephenie Zhang |
| 10 | Growth | The end of the screen time KPI in AI applications | Santiago Rodriguez |
| 11 | Bio + Health | Healthy MAUs | Julie Yoo |
| 12 | Speedrun | World models take the spotlight in storytelling | Jonathan Lai |
| 13 | Speedrun | "The year of me" | Joshua Lu |
| 14 | Speedrun | The first AI-native university | Emily Bennett |

### 3.2 Part 2 — December 10, 2025 (16 ideas)

| # | Vertical | Idea title | Bylined author |
| --- | --- | --- | --- |
| 15 | American Dynamism | Building the AI-native industrial base | David Ulevitch |
| 16 | American Dynamism | The renaissance of the American factory | Erin Price-Wright |
| 17 | American Dynamism | The next wave of observability will be physical, not digital | Zabie Elmgren |
| 18 | American Dynamism | The electro-industrial stack will move the world | Ryan McEntush |
| 19 | American Dynamism | Autonomous labs accelerate scientific discovery | Oliver Hsu |
| 20 | American Dynamism | Data crusade in our critical industries | Will Bitsky |
| 21 | Apps | AI reinforces business models | David Haber |
| 22 | Apps | ChatGPT becomes the AI app store | Anish Acharya |
| 23 | Apps | Voice agents take up space | Olivia Moore |
| 24 | Apps | Prompt-free and proactive applications arrive | Marc Andrusko |
| 25 | Apps | AI will finally upgrade banking and insurance infrastructure | Angela Strange |
| 26 | Apps | Forward-deployed motions take AI to the 99% | Joe Schmidt |
| 27 | Apps | AI creates a new orchestration layer—and new roles—in the Fortune 500 | Seema Amble |
| 28 | Apps | Consumer AI shifts from "help me" to "see me" | Bryan Kim |
| 29 | Apps | New model primitives unlock previously impossible companies | Kimberly Tan |
| 30 | Apps | AI startups selling to other AI startups reach scale | James da Costa |

### 3.3 Part 3 — December 11, 2025 (17 ideas)

Part 3 has no vertical headings. All 17 are presented in the order below, with no grouping beyond the practice they come from.

| # | Idea title | Bylined author |
| --- | --- | --- |
| 31 | Privacy will be the most important moat in crypto | Ali Yahya |
| 32 | Prediction markets go bigger, broader, and smarter | Andrew Hall |
| 33 | Thinking about tokenization of real world assets, and stablecoins, in a more crypto-native way | Guy Wuollet |
| 34 | Trading as a way station, not the last stop, for crypto businesses | Arianna Simpson |
| 35 | From know your customer (KYC) to 'know your agent' (KYA) | Sean Neville |
| 36 | Better, more clever onramps/ offramps for stablecoins | Jeremy Zhang |
| 37 | Stablecoins unlock the bank ledger upgrade cycle — and new payment scenarios | Sam Broner |
| 38 | The (near) future of messaging isn't just quantum-resistant. It's decentralized | Shane Mac |
| 39 | From 'code is law' to 'spec is law' | Daejun Park |
| 40 | Crypto offers a new primitive for use beyond blockchains | Justin Thaler |
| 41 | We'll use AI for substantive research tasks | Scott Duke Kominers |
| 42 | The invisible tax on the open web | Elizabeth Harkavy |
| 43 | The rise of staked media | Robert Hackett |
| 44 | 'Secrets-as-a-service' | Adeniyi Abiodun |
| 45 | Wealth management for all | Maggie Hsu |
| 46 | The internet becomes the bank | Christian Crowley |
| 47 | Unleashing the full potential of blockchains… when legal architecture finally matches technical architecture | Miles Jennings |

### 3.4 Part 1 — neutral summaries

Each summary states what the entry claims, not whether it is true. Quotes are exact.

- **1. Startups tame the chaos of multimodal data — Jennifer Li.** Claims that unstructured, multimodal data is the binding constraint on enterprise AI, that the limiting factor is now "data entropy: the steady decay of freshness, structured, and truth inside the unstructured universe where **80% of corporate knowledge** now lives," and that platforms which clean, structure, validate and govern multimodal data are a generational opportunity. Names no companies.
- **2. AI revives cybersecurity hiring — Joel de la Garza.** Claims that the security hiring gap was largely self-inflicted by tooling that detects everything (so teams must review everything), that AI-native tooling will automate the level-1 drudgery, and that in 2026 AI "will break this cycle and close this hiring gap." Evidence offered: an unfilled-jobs statistic for 2013–2021 (see §11.2).
- **3. Agent-native infrastructure becomes table stakes — Malika Aubakirova.** Claims a shift from "human-speed traffic" with a 1:1 action-to-response ratio to "agent-speed" workloads that are "recursive, bursty, and massive," and that the existing enterprise backend is not architected for a single agentic goal to trigger a recursive fan-out. Presents infrastructure as the constraint, not the models.
- **4. Creative tools go multimodal — Justine Moore.** Claims the building blocks for AI storytelling now exist (generative voices, music, images, video) but that sustained creative control does not, and that tools which give creators director-like control across modalities are the open opportunity.
- **5. The AI-native data stack continues to evolve — Jason Cui.** Claims the "modern data stack" consolidated in the past year (citing the Fivetran/dbt merger and the continued rise of unified platforms like Databricks) but that a truly AI-native data architecture is still early. Contains an internal list headed **"A few ideas we're excited by:"** with three items: vector databases alongside structured data; agents navigating "the context problem"; and change in BI tools and spreadsheets as workflows become agentic.
- **6. The year we step inside video — Yoko Li.** Claims 2026 is when video "stops behaving like something we passively watch and starts feeling like a place we can actually step into," because video models can now hold temporal consistency, remember prior frames and react to user action.
- **7. Systems of record lose ground — Sarah Wang.** Claims the system of record "will finally start to lose primacy" in 2026 as models read, write and reason directly across operational data, turning ITSM and CRM systems into autonomous workflow engines rather than passive databases.
- **8. Vertical AI evolves from information retrieval and reasoning to multiplayer — Alex Immerman.** Claims vertical AI has moved through a retrieval phase and a 2025 reasoning phase, and that the next phase is "multiplayer" — multi-party, multi-agent workflows inside vertical domains. Cites sector growth ($100M+ ARR reached within a few years in healthcare, legal, housing).
- **9. Creating for agents, not humans — Stephenie Zhang.** Claims that as people interface with the web through agents, the signals optimised for human consumption (Google ranking, Amazon placement, the TL;DR lead) stop being the signals that matter, and the web must be re-authored for agent consumption.
- **10. The end of the screen time KPI in AI applications — Santiago Rodriguez.** Claims screen time has been the value indicator for consumer and business applications for fifteen years and that outcome-based pricing will retire it, citing personal examples of value captured with almost no screen time (named examples in the essay: Abridge, Cursor, Hebbia).
- **11. Healthy MAUs — Julie Yoo.** Introduces a healthcare customer segment — the "healthy MAUs" — positioned against the system's traditional "sick MAUs" and "sick DAUs," arguing the healthy cohort is an under-served and economically decisive group for 2026.
- **12. World models take the spotlight in storytelling — Jonathan Lai.** Claims AI world models will drive new interactive storytelling formats and digital economies, naming technologies already available — "Marble (World Labs) and Genie 3 (DeepMind)" — which generate full 3D environments from text prompts, and anticipating a "generative Minecraft."
- **13. "The year of me" — Joshua Lu.** Claims 2026 is when products stop being mass-produced and start being made for the individual, with education as the lead example (AI tutors adapting to each student's pace). Names Alphaschool.
- **14. The first AI-native university — Emily Bennett.** Claims 2026 will see the birth of an institution built from the ground up around intelligent systems — an "adaptive academic organism" — with assessment shifting from detection and prohibition to evaluation of how AI is used.

### 3.5 Part 2 — neutral summaries

- **15. Building the AI-native industrial base — David Ulevitch.** Claims America is rebuilding energy, manufacturing, logistics and infrastructure, and that the decisive shift is industrial companies that are "truly AI native and software-first," starting from simulation, automated design and AI-driven operations. Ends with a direct invitation to founders: "If that's you, let's talk." (On that sentence as a genre feature, see §5.)
- **16. The renaissance of the American factory — Erin Price-Wright.** Claims a rebirth of the American factory "with software and AI at its heart," acknowledging prior deindustrialisation as partly an "intentional, society-wide failure to build."
- **17. The next wave of observability will be physical, not digital — Zabie Elmgren.** Claims software observability's pattern — logs, metrics, traces making systems transparent — is being applied to the physical world, citing "more than a billion networked cameras and sensors already deployed across U.S. cities."
- **18. The electro-industrial stack will move the world — Ryan McEntush.** Claims the next industrial revolution happens inside machines, via convergence of electrification, materials and AI, producing machines that "sense, learn, and act on their own."
- **19. Autonomous labs accelerate scientific discovery — Oliver Hsu.** Claims improving model capabilities and robotic manipulation will enable closed-loop autonomous labs spanning hypothesis, experiment design, execution, and iteration.
- **20. Data crusade in our critical industries — Will Bitsky.** Claims the 2025 zeitgeist was compute and data-centre constraints while 2026 will be data constraints, and that critical industries are the next frontier because every truck roll, meter read and production run is latent unstructured data.
- **21. AI reinforces business models — David Haber.** Claims the best AI startups amplify the customer's existing economics rather than only cutting costs, using contingency-fee law as the exemplar (naming Eve) where AI predicts case success and thus increases the firm's revenue.
- **22. ChatGPT becomes the AI app store — Anish Acharya.** Claims consumer product cycles need new technology, behaviour and distribution, that the AI wave had the first two but no native channel, and that the OpenAI Apps SDK, Apple's mini-app support and ChatGPT's distribution now supply the third.
- **23. Voice agents take up space — Olivia Moore.** Claims AI voice agents moved from science fiction to production in 18 months across SMBs and enterprises for scheduling, bookings, surveys and intakes, and that the next move is into more complex, higher-stakes interactions.
- **24. Prompt-free and proactive applications arrive — Marc Andrusko.** Claims 2026 is "the death of the prompt box for mainstream users," with applications observing context and proposing actions for review. Gives IDE, CRM and design-tool examples.
- **25. AI will finally upgrade banking and insurance infrastructure — Angela Strange.** Claims institutions have applied AI on top of legacy systems but that real transformation requires rebuilding the infrastructure beneath, that in 2026 the risk of not modernising will outweigh the risk of failure, and that large institutions will let legacy vendor contracts lapse for AI-native alternatives. Predicts new category winners "10x the size of the older incumbents."
- **26. Forward-deployed motions take AI to the 99% — Joe Schmidt.** Claims AI startup benefits have accrued mostly to the Silicon Valley 1% and that forward-deployed go-to-market — engineers and sellers embedded with mainstream customers — is how the rest is reached.
- **27. AI creates a new orchestration layer—and new roles—in the Fortune 500 — Seema Amble.** Claims enterprises will move from isolated AI tools to coordinated multi-agent systems behaving like digital teams, requiring new organisational structures, new roles and new mechanisms for context flow. Names AskLio and HappyRobot as early examples.
- **28. Consumer AI shifts from "help me" to "see me" — Bryan Kim.** Claims 2026 shifts major consumer AI products from productivity to connectivity — AI used to see oneself and build relationships rather than to do work — enabled by multimodal context windows and falling inference costs, while acknowledging many social AI products have launched and failed.
- **29. New model primitives unlock previously impossible companies — Kimberly Tan.** Claims 2026 brings companies whose core capability is enabled by new model primitives in reasoning, multimodality and computer use, rather than improved incumbent products. Gives billing-dispute adjudication, latent video data extraction for physical-industry cameras, and desktop-software automation as examples.
- **30. AI startups selling to other AI startups reach scale — James da Costa.** Claims that because incumbents are adopting AI, a powerful and underrated distribution route for startups is serving companies at their formation — greenfield businesses that will standardise on the tooling they use from day one.

### 3.6 Part 3 — neutral summaries

- **31. Privacy will be the most important moat in crypto — Ali Yahya.** Claims privacy is the missing feature for onchain finance and that it creates chain lock-in ("a privacy network effect"), because "bridging tokens is easy, bridging secrets is hard," producing a winner-take-most dynamic that "a handful of privacy chains could own most of crypto."
- **32. Prediction markets go bigger, broader, and smarter — Andrew Hall.** Claims prediction markets will expand in contract count, use and sophistication as they intersect with crypto and AI, and that scaling requires new truth-alignment mechanisms. Lists concrete open problems: resolution of disputed cases (cites "the Zelensky suit market and the Venezuelan election market"), decentralised governance and LLM oracles, AI agents as traders and as oracles (cites "Prophet Arena"), and complementarity with — not replacement of — polling. Contains a visible source-editing placeholder, `<SC will link to our article here>`, in the published text.
- **33. Thinking about tokenization of real world assets, and stablecoins, in a more crypto-native way — Guy Wuollet.** Claims the current tokenisation of traditional assets is "skeuomorphic" — a wrapper on the old concept — and that crypto-native instruments such as perpetual futures unlock deeper liquidity and better uses.
- **34. Trading as a way station, not the last stop, for crypto businesses — Arianna Simpson.** Observes that "every crypto company that's doing well today, outside of stablecoins and some core infrastructure, has pivoted to or is pivoting to trading," warns of cannibalised mindshare and few winners, and argues trading should be a transitional position.
- **35. From know your customer (KYC) to 'know your agent' (KYA) — Sean Neville.** Claims the agent economy's bottleneck has moved from intelligence to identity, that "non-human identities" outnumber human financial-services employees "96-to-1," and that agents will need cryptographically signed credentials linking agent, principal, constraints and liability. The industry "has just months to figure out KYA."
- **36. Better, more clever onramps/ offramps for stablecoins — Jeremy Zhang.** Claims stablecoins did an estimated "46 trillion dollars in transaction volume last year" — "more than 20x the volume of PayPal; close to 3x the volume of Visa" — and that the unsolved problem is connecting digital dollars to the rails people already use. Describes three ongoing approaches by a "new generation of startups."
- **37. Stablecoins unlock the bank ledger upgrade cycle — and new payment scenarios — Sam Broner.** Claims banks still run decades-old core ledgers (citing Temenos' GLOBUS and InfoSys' Finacle as second-generation core banking software, and mainframe/COBOL/batch interfaces as the current state), that adding real-time payments can take months to years, and that stablecoins and tokenised instruments let institutions build new products without rewriting those systems.
- **38. The (near) future of messaging isn't just quantum-resistant. It's decentralized — Shane Mac.** Claims every major messenger depends on trusting a private server run by a single organisation, that post-quantum encryption does not help if that server can be shut down or coerced, and that decentralised, open-protocol messaging with no private server is the answer.
- **39. From 'code is law' to 'spec is law' — Daejun Park.** Claims recent DeFi hacks on battle-tested, audited protocols show security practice is "still largely heuristic and case-by-case," and argues it must move to design-level properties and principled approaches, pre-deployment and post.
- **40. Crypto offers a new primitive for use beyond blockchains — Justin Thaler.** Claims SNARKs have been effectively blockchain-only because proving overhead was extreme, and that in 2026 zkVM provers will reach a cost at which proof-based verification is practical outside blockchains.
- **41. We'll use AI for substantive research tasks — Scott Duke Kominers.** A mathematical economist's own observation that within a year models went from unable to understand his workflow to handling abstract instructions and "sometimes return novel and correctly executed answers," and that AI use for research tasks is broadening.
- **42. The invisible tax on the open web — Elizabeth Harkavy.** Claims AI agents extract data from ad-supported sites (the "Context layer") while bypassing the revenue streams that fund them (the "Execution layer"), imposing an unpriced cost on the open web's economic foundation.
- **43. The rise of staked media — Robert Hackett.** Claims audiences increasingly respect perspectives that disclose their stakes, and that media participation is shifting toward operators and practitioners who speak with visible interests rather than claimed objectivity.
- **44. 'Secrets-as-a-service' — Adeniyi Abiodun.** Claims most data pipelines are opaque, mutable and unauditable, that finance and healthcare require verifiable privacy, and that this blocks institutions from tokenising real-world assets — implying a service market for verifiable secrecy.
- **45. Wealth management for all — Maggie Hsu.** Claims tokenisation of asset classes plus crypto rails plus AI advice can deliver personalised, cheaply rebalanced portfolios historically reserved for high-net-worth clients.
- **46. The internet becomes the bank — Christian Crowley.** Claims that as agents transact autonomously, value must move "as fast and freely as information does today," and that value movement — not user clicking — becomes the primitive.
- **47. Unleashing the full potential of blockchains… when legal architecture finally matches technical architecture — Miles Jennings.** Claims a decade of US legal uncertainty forced engineers behind lawyers and produced distortions (avoided transparency, arbitrary token distributions, governance as theater), and that crypto market-structure legislation — "closer to passing than it's ever been" — could eliminate those distortions. Notes that "After GENIUS, the proliferation of stablecoins has exploded."

### 3.7 Cross-cutting counts

| Dimension | Count | Note |
| --- | --- | --- |
| Ideas total | 47 | 14 + 16 + 17 ✅ |
| Distinct bylined contributors | 47 | No contributor appears twice in the 2026 edition ✅ |
| Parts | 3 | December 9, 10, 11 2025 ✅ |
| Pages with a "a16z New Media" page byline | 3 | The page byline, not an idea byline ✅ |
| Verticals named as headings | 6 | Infrastructure, Growth, Bio + Health, Speedrun, American Dynamism, Apps — Part 3 has none ✅ |
| Team labels referenced in prose | 7 | the six above plus crypto ✅ |
| Ideas whose text names at least one company | 12+ | Named companies include Abridge, Cursor, Hebbia, Alphaschool, Marble (World Labs), Genie 3 (DeepMind), Eve, AskLio, HappyRobot, Fivetran, dbt, Databricks, Temenos, InfoSys, Prophet Arena, PayPal, Visa, Circle (via a contributor bio) |

### 3.8 The methodological caution: entries versus surrounding cards

The three pages are not clean lists. Each page interleaves the edition's entries with **navigation and promotional cards** — "More From this Contributor" strips, "Related Content" pointers to the other parts, "Recommended For You" modules, newsletter subscribe prompts, and a recurring card for an item titled "You can just read 25 books." Some of those cards carry titles that read exactly like plausible Big Ideas entries.

**The audit rule used in this guide.** A **real Big Ideas entry** has all four of:

1. an idea **title**, followed by
2. a **bylined author name** on its own,
3. a **body essay** of prose paragraphs, and
4. an **"About the Contributor"** block with the author's name repeated and a bio sentence beginning "is a…" / "was an…" / "is the…".

A **card** has (1) and (2) only — a title line and a name line, with no body and no bio block. Cards also carry a different visual frame: they sit under headings such as "More From this Contributor" or "Related Content," and often begin with "Investing in…", which is the firm's announcement format.

Applied to the cached files, the audit produces a clean result and confirms the count: **14 + 16 + 17 idea entries, plus exactly one "About the Contributor / a16z New Media" page-byline block per part.** The extra bio block at the end of each page is the page's own byline, not a 15th/17th/18th entry. Total idea entries: **47** ✅.

Cards that match the entry *shape* by title but fail the test — and are therefore **not** listed as ideas anywhere in this guide — include, as found in the cached 2026 page text:

- **"The Cinderella 'Glass Slipper' Effect: Retention Rules in the AI Era"** — Malika Aubakirova
- **"Governance FAQs"** — Andrew Hall
- **"AI Micro Dramas, Generative Media, and the Future of Creativity"** — Justine Moore
- **"The Palantirization of everything"** — Marc Andrusko
- **"The Incumbents Are Coming"** — Seema Amble

Each of those appears as a title-plus-name card with no body essay and no "About the Contributor" bio in the 2026 texts. A reader who scrapes the pages and counts title-like strings will overcount; the ~47 figure depends on applying the test above. This caution is repeated in §14 and §15.

---

## 4. The Authorship and What It Reveals

### 4.1 The structural point

**The list is organised the way the firm is organised.** The seven team labels used in the 2026 edition — Infrastructure, Growth, Bio + Health, Speedrun, American Dynamism, Apps, crypto — are not a market taxonomy that a neutral analyst would produce. They are the firm's own **practice groups**, and several of them are explicitly described in the bios as practices with named leaders. The headings in the artefact are therefore closer to a management chart than to an industry map, and that has three consequences a careful reader should carry:

1. **Coverage is a function of headcount, not of importance.** A vertical with more investing partners contributes more ideas. Growth contributes 4 and Bio + Health contributes 1 — not because growth-stage AI matters four times as much as healthcare, but because that is how the teams are staffed. Any reading of the edition as a weighted ranking of sectors is unsupported by the artefact's construction.
2. **Verticals can appear, disappear or be renamed without a market event.** "Apps" and "Speedrun" are 2026 labels; "Consumer Tech" and "Games" were 2025 labels (§6). A label change is an organisational fact first and a market signal second.
3. **Absence of a vertical is not a claim about that vertical.** Part 3 note: crypto is a practice with its own newsletter and its own framing language; it is not labelled with a "vertical" heading because it constitutes the whole of that page.

### 4.2 The bylines and the bios, as the source gives them

Every one of the 47 entries is individually bylined. The following bios are quoted or closely paraphrased from the "About the Contributor" blocks in the cached part texts. Where the cached text gives no role, this guide supplies no role — it does not infer a title from a person's other public appearances.

| Contributor | Idea (short) | Role as stated in the "About the Contributor" block |
| --- | --- | --- |
| Jennifer Li | multimodal data | "is a general partner at Andreessen Horowitz, where she leads infrastructure investments with an eye on data systems, developer tools and AI" |
| Joel de la Garza | cybersecurity hiring | "is a partner focused on information security related investments and other chaos adjacent businesses" |
| Malika Aubakirova | agent-native infrastructure | "**was** an investor on the AI Infrastructure team at a16z" — past tense (see §4.3) |
| Justine Moore | creative tools | "is a partner on the investing team at Andreessen Horowitz, where she focuses on AI — both foundation models and applications" |
| Jason Cui | data stack | "is a partner at Andreessen Horowitz, where he invests in infrastructure and AI" |
| Yoko Li | video | "is a partner at Andreessen Horowitz, where she focuses on developer tools, infrastructure, AI, and creative tools" |
| Sarah Wang | systems of record | "is a general partner on the Growth team at Andreessen Horowitz, where she leads growth-stage investments across AI, enterprise applications, and infrastructure" |
| Alex Immerman | vertical AI | "is a partner on the Growth team at Andreessen Horowitz, where he focuses on fintech, consumer, enterprise, and crypto/web3 companies" |
| Stephenie Zhang | creating for agents | "is a partner on the Growth investing team, focused on enterprise technology companies" |
| Santiago Rodriguez | screen-time KPI | "is a partner on the Growth investing team, focused on fintech, consumer, and crypto technology companies" |
| Julie Yoo | Healthy MAUs | "is a general partner on the Bio + Health team, where she leads investments into companies that are transforming how we access, pay for, and experience healthcare" |
| Jonathan Lai | world models | "is a general partner at Andreessen Horowitz, focused on a16z speedrun. He invests in early-stage teams building tomorrow's AI x creative landscape…" |
| Joshua Lu | "The year of me" | "is an Investment Partner at Andreessen Horowitz for a16z speedrun" |
| Emily Bennett | AI-native university | "is an investing partner for a16z speedrun" |
| David Ulevitch | AI-native industrial base | "is a general partner at Andreessen Horowitz, where he leads the firm's American Dynamism practice and invests in enterprise and SaaS software" |
| Erin Price-Wright | American factory | "is a general partner at Andreessen Horowitz where she invests in companies that promote American Dynamism, with an emphasis on teams building AI for the physical world" |
| Zabie Elmgren | physical observability | "is a partner at Andreessen Horowitz where she backs founders building companies that strengthen critical infrastructure, public safety, and more" |
| Ryan McEntush | electro-industrial stack | "is an investing partner at a16z, focusing on companies that promote American Dynamism" |
| Oliver Hsu | autonomous labs | "is a partner on the American Dynamism investing team at a16z, where he focuses on emerging computing platforms and applications for the physical world" |
| Will Bitsky | data in critical industries | "is an investing partner supporting a16z's American Dynamism team" |
| David Haber | AI reinforces business models | "is a general partner at Andreessen Horowitz, where he focuses on technology investments in B2B software and financial services" |
| Anish Acharya | AI app store | "is an entrepreneur and general partner at Andreessen Horowitz… focuses on consumer investing, including AI-native products…" |
| Olivia Moore | voice agents | "is a partner on the investing team at Andreessen Horowitz, where she focuses on AI" |
| Marc Andrusko | prompt-free applications | "is a partner at Andreessen Horowitz, where he focuses on B2B AI applications and fintech" |
| Angela Strange | banking/insurance infrastructure | "is a general partner at Andreessen Horowitz, where she focuses on financial services, insurance, and B2B software (with AI)" |
| Joe Schmidt | forward-deployed | "is a partner at Andreessen Horowitz, where he focuses on software, fintech, and insurtech investments" |
| Seema Amble | orchestration layer | "is a partner at Andreessen Horowitz, where she focuses on investments in B2B software and fintech" |
| Bryan Kim | "see me" | "is a partner at Andreessen Horowitz, where he invests primarily in consumer AI applications" |
| Kimberly Tan | new model primitives | "is an investing partner at Andreessen Horowitz, where she focuses on SaaS and AI investments" |
| James da Costa | AI selling to AI | "is a partner at Andreessen Horowitz, where he focuses on investing in B2B software and financial services" |
| Ali Yahya | privacy moat | "is a general partner at a16z crypto" |
| Andrew Hall | prediction markets | "is a research advisor to the a16z crypto team" |
| Guy Wuollet | tokenisation, crypto-native | "is a general partner at Andreessen Horowitz, focusing on infrastructure and application layer investments across crypto" |
| Arianna Simpson | trading as way station | "is a general partner at a16z crypto" |
| Sean Neville | KYC → KYA | "**is the cofounder of Circle and architect of USDC; CEO of Catena Labs**" — a guest contributor (see §4.4) |
| Jeremy Zhang | on/off ramps | "is a Full Stack Web Developer on a16z's Crypto team" |
| Sam Broner | bank ledger upgrade | "is an investing partner for the a16z crypto team" |
| Shane Mac | decentralised messaging | "**is the co-founder and CEO, XMTP Labs**" — a guest contributor |
| Daejun Park | spec is law | "is a Senior Blockchain Security Engineer at a16z crypto, developing formal methods and tools for web3 security…" |
| Justin Thaler | crypto primitives beyond blockchains | "is Research Partner at a16z and an Associate Professor in the Department of Computer Science at Georgetown University" |
| Scott Duke Kominers | AI for research | "is a Research Partner at a16z crypto" |
| Elizabeth Harkavy | invisible tax | "is a partner on the a16z crypto investment team" |
| Robert Hackett | staked media | "is an Operating Partner and Head of Content and Editorial for a16z crypto, helping to build a media operation centered on web3" |
| Adeniyi Abiodun | secrets-as-a-service | "**is the chief product officer and co-founder, Mysten Labs**" — a guest contributor |
| Maggie Hsu | wealth management | "is Head of Go-to-Market for a16z crypto" |
| Christian Crowley | internet becomes the bank | "is a business development partner for a16z crypto" |
| Miles Jennings | legal architecture | "is General Counsel & Head of Decentralization for a16z crypto" |

### 4.3 Two neutral footnotes on the bios

- **Malika Aubakirova's bio is in the past tense.** It reads "was an investor on the AI Infrastructure team at a16z" — not "is". That is a factual detail about the published page: the bio, as it appears in the December 2025 text, describes the role historically. This guide does not speculate about why, and does not assert any current role for this contributor. It is worth a footnote precisely because the artefact's own bio block is the only authoritative statement available here.
- **Two contributors are scientists, not investors.** Justin Thaler is described as a research partner *and* a Georgetown computer-science associate professor; Andrew Hall is described as a *research advisor*. The crypto part therefore mixes full-time investing partners, operating partners, engineers, a general counsel, a head of go-to-market — and outside academics. The word "partners" in the page framing is doing loose work; the bios are more precise than the framing.

### 4.4 The guest contributors

Part 3's framing explicitly flags "a few guest contributors". The bios support that: **Sean Neville** (cofounder of Circle, architect of USDC, CEO of Catena Labs), **Shane Mac** (cofounder and CEO, XMTP Labs) and **Adeniyi Abiodun** (chief product officer and cofounder, Mysten Labs) are described by their outside-company roles rather than as a16z personnel. A fourth, **Justin Thaler**, is described partly by an academic role but also as an a16z research partner, so he straddles the line. This is a real difference between Part 3 and Parts 1–2: **Parts 1 and 2 are internal staffing bylines; Part 3 includes external voices.**

Note what follows from that, and what does not. It follows that Part 3 is not purely one firm talking about itself. It does **not** follow that the guest entries are independent of the firm — two of the three named guests lead companies in categories the firm's crypto practice invests in, and one of those companies (Catena Labs) appears as the subject of an "Investing in…" card in the same cached page text (see §9). Both statements are facts of the document. This guide draws no conclusion about motive.

### 4.5 What authorship reveals about the editorial process

Three observations that can be made safely from the bylines alone:

- **One contributor, one idea, one short essay.** The unit is deliberately small. There is no attempt to reconcile Infrastructure's six ideas with one another, and Jason Cui's entry contains an internal list of three sub-ideas rather than a single claim — the "one big idea" framing is a publishing convention, not a content constraint.
- **The firm does not assign ideas to its best-known names.** Marc Andreessen, Ben Horowitz and several of the firm's most prominent general partners do not appear as 2026 idea authors. The bylines are practice-level investors and specialists. (Ben Horowitz appears in a *card* for "The Machine Age Fund" in Part 2, not as an idea author — see §9.)
- **Bylines are attributed to people, never to practices.** Even though the verticals are org units, no idea is signed "the Infrastructure team". This makes individual accountability possible in principle and firm-level accountability impossible in practice — a tension taken up in §7.

---

## 5. What This Artefact Actually Is — The Genre Read

This is the analytical centre of the guide. It argues a single claim: the Big Ideas package is best understood as **four things at once**, and the reader's job is to notice which one an individual sentence is doing.

### 5.1 Four simultaneous functions

**(i) An investment-thesis statement.** Every entry says, in effect, "here is a category we believe will matter, and here is what would have to be built in it." Jennifer Li does not merely observe that multimodal data is messy; she names the shape of the company she expects ("Startups that build the platform that extracts structure from documents, images, and videos; reconciles conflicts; repairs pipelines; or keeps data fresh and retrievable"). Angela Strange does not merely note legacy bank infrastructure; she predicts large institutions will "let their legacy vendor contracts lapse." These are theses in the ordinary investment sense: falsifiable-in-principle claims about where value will accrue.

**(ii) A deal-flow instrument.** A published thesis is how the founders who fit it find you. This is not an inference in the 2026 text — the text does it. David Ulevitch's entry ends: **"If that's you, let's talk."** Anish Acharya's entry points at a distribution channel and invites building on it. Erin Price-Wright's entry addresses founders directly. Part 1's framing says the team asks itself what "tech builders will tackle" — the audience is named in the framing. A thesis published in December, at the moment founders are planning the following year, is a recruitment notice for deal flow as much as it is an analysis.

**(iii) A brand and LP-communication product.** The pages carry newsletter subscribe prompts, an "Expert News by a16z" block, "See All Newsletters", cross-promotion to the other parts, and a two-line house tagline ("Software Is Eating the World"). Part 3 closes with a promise of "a special announcement and an invitation from a16z". Part 2's cards link a podcast episode and a fund page. This is a content operation with distribution objectives, and it is published by the adviser to investment vehicles whose investors read it.

**(iv) Positions already held.** The theses are not speculative in the sense of being untested by the firm's own capital. This is covered as fact, with sources and dates, in §9.

### 5.2 The publisher's own statement of the genre

The genre has a precedent and the publisher has described it. In the 2023 omnibus (`a16z.com/big-ideas-in-tech-for-2023-an-a16z-omnibus/`, byline "a16z editorial", posted **December 15, 2022**), a16z wrote:

> "This time of year, the floodgates open to a deluge of best-of lists and end-of-year recaps. But we at a16z have always been more interested in what's to come. We asked dozens of partners across the firm to spotlight one big idea that startups in their fields will tackle in 2023."

That is the publisher characterising both the surrounding genre (the year-end list) and its own deliberate divergence (a forward-looking list). It is the single most useful sentence in the corpus for §5, because it establishes the artefact as **intentionally prospective** — and it establishes the audience as "startups in their fields".

The later editions restate the same construction in the same register. 2024: "We asked over 40 partners across a16z to preview one big idea they believe will drive innovation in 2024." 2025: "We asked 50 a16z partners to preview one big idea that will spur innovation in 2025." 2026, Part 1: "every December, we ask our investing teams to share one big idea they think tech builders will tackle in the year to come." Four consecutive years, one construction, one cadence.

### 5.3 Observable genre features

These are properties of the class, verified against the 2026 text and the prior editions. None of them requires any claim about anyone's intent.

| Feature | Observation | Evidence in the artefact |
| --- | --- | --- |
| **No negative predictions** | Across all 47 entries, there is no entry whose claim is that something will *fail*, *shrink* or *stop*. The closest is Sarah Wang's "systems of record lose ground" — but the loss is to a successor the firm invests in, and the sentence is about re-architecture rather than decline | Reviewed all 47 entries in the cached parts; no entry predicts a category's failure |
| **No idea that would undercut the firm's own positions** | No entry argues against a category the firm is publicly invested in | Every entry argues for a build opportunity; see §9 for documented holdings in the same themes |
| **Organisational grouping** | Ideas are filed by the firm's practice groups, not by an external taxonomy | §4.1 |
| **Annual cadence** | Published every December, four years running (2022-12-15, 2024 edition, 2025 edition, 2025-12-09/10/11) | §2, §6 |
| **Bounded unit** | One contributor, one title, one short essay, one bio | §4.5 |
| **No method section** | No edition states how ideas were selected, how many were considered, or what threshold a claim must clear | §7 |
| **No scoring or follow-up** | No edition revisits a prior edition's claims | §7 |
| **Disclosure by template** | The conflict disclosure is a standing footer, identical or near-identical across editions, not entry-specific | §9.1 |

### 5.4 The two structural asymmetries worth naming

**(a) The list has no obligation to be wrong-able.** A thesis that cannot be marked wrong cannot be marked right either. The construction "X will become table stakes", "the year we step inside video", "creative tools go multimodal" resists adjudication on a one-year horizon because the boundary conditions are never specified. This is a property of the genre — a year-end thought-leadership artefact — not a defect unique to this publisher, and it does not require any assumption about editorial honesty to observe. The consequence for the reader is practical: **the artefact is usable as a map of attention and unusable as a scorecard** (§7, §10).

**(b) The disclosure is present but generic.** The pages do carry a substantial investment disclaimer (§9.1). It is a footer, not an entry-level disclosure: a reader is told that the firm may hold positions in anything mentioned, in a block of legal text, rather than beside the idea that mentions it. Both halves of that sentence are facts of the document.

**One thing this section explicitly does not say.** It does not allege bad faith, deception, manipulation or undisclosed conflict of any kind, and it attributes no motive to any named person. Everything above describes how a genre is built, what a genre does structurally, and what a reader can and cannot extract from it. The genre features listed are shared with sell-side research, with industry-consortium roadmaps, and with academic position papers; they are properties of prospective publishing, not evidence of anything about a16z's intentions, which this guide has no way to assess and does not purport to.

---

## 6. The Comparison with the Prior Edition

### 6.1 The format change, as verified

The most consequential difference between 2026 and every prior edition is not a topic — it is the container.

| Edition | Published as | URL pattern | Page byline | Date | Ideas |
| --- | --- | --- | --- | --- | --- |
| 2023 | Single omnibus article | `a16z.com/big-ideas-in-tech-for-2023-an-a16z-omnibus/` | a16z editorial | **December 15, 2022** | "40+ builder-worthy pursuits", "dozens of partners" |
| 2024 | Single omnibus article | `a16z.com/big-ideas-in-tech-2024/` | (page) | 2024 edition | "over 40 partners" |
| 2025 | Single omnibus article | `a16z.com/big-ideas-in-tech-2025/` | (page) | 2025 edition | "50 a16z partners" |
| 2026 | **Three-part newsletter series** | `a16z.com/newsletter/big-ideas-2026-part-{1,2,3}/` | **a16z New Media** | **December 9, 10, 11 2025** | **47** (14 + 16 + 17) |

The decisive evidence that the container changed rather than merely being supplemented: **the omnibus URL for the 2026 edition does not exist.** `a16z.com/big-ideas-in-tech-2026/` returns "Page not found | Andreessen Horowitz" — re-verified directly with the web extractor in this pass. The three newsletter URLs exist and carry the dates and the "a16z New Media" byline. So the format change is not an addition to a continuing series; it is a **replacement** of the omnibus with a three-day newsletter run.

Why that matters practically:

- **Discoverability.** There is no single URL that contains the 2026 edition. A reader who guessed the URL pattern from 2023–2025 finds a 404.
- **Citation.** A citation must name the part number and the date, because each part is a separate page with a separate date.
- **Per-part framing.** Each part writes its own introduction, and the three introductions are not identical in epistemic register (§2.3).
- **Packaging.** Each part re-renders the site's promotional furniture: related-content pointers, contributor cards, newsletter prompts, "Recommended For You". The 2026 edition is therefore embedded in a *newsletter product* in a way the omnibus editions were not.

### 6.2 Verticals present and absent

| Vertical label | 2025 omnibus | 2026 series |
| --- | --- | --- |
| American Dynamism | ✅ | ✅ |
| Bio + Health | ✅ | ✅ |
| Consumer Tech | ✅ | ❌ absent as a label |
| Crypto | ✅ | ✅ (as the whole of Part 3, without a heading) |
| Enterprise + Fintech | ✅ | ❌ absent as a label |
| Games | ✅ | ❌ absent as a label |
| Growth-Stage Tech | ✅ | ✅ (as **Growth**) |
| Infrastructure | ✅ | ✅ |
| **Apps** | ❌ | ✅ new label |
| **Speedrun** | ❌ | ✅ new label |

The 2025 omnibus names eight verticals; the 2026 series names six heading labels plus crypto. Three labels disappear (**Consumer Tech, Games, Enterprise + Fintech**) and two appear (**Apps, Speedrun**), while "Growth-Stage Tech" is shortened to "Growth".

Two things follow, and only two. First, **label churn is at least as large as topic churn** — a reader tracking "Consumer Tech" across editions has to notice that the label vanished while consumer-facing ideas did not (Bryan Kim's "help me" to "see me" is squarely consumer). Second, **the enterprise surface area shrank as a label and grew as content**: "Enterprise + Fintech" is gone, but Part 3 of the 2026 edition is heavily financial-infrastructure content and Part 2's Apps vertical is full of B2B (David Haber, Joe Schmidt, Seema Amble, James da Costa). Counting labels would mislead.

### 6.3 The topic shift, read from the publisher's own teaser

The 2025 omnibus opens with its own teaser line, which is the cleanest available baseline for what the firm was emphasising a year earlier:

> "An external 'AI brain.' Big swings in biopharma. Infinite games. A nuclear resurgence. 'Faceless' creators. Google search challengers. Battlefield AI. We asked 50 a16z partners to preview one big idea that will spur innovation in 2025."

Set against the 2026 distribution:

| 2025 emphasis | 2026 status in the edition |
| --- | --- |
| "An external 'AI brain.'" | Reframed: 2026 has no single equivalent; the closest analogues are agent-native infrastructure (Malika Aubakirova), prompt-free applications (Marc Andrusko) and creating for agents (Stephenie Zhang). The "brain" framing is replaced by an *interfaces-and-infrastructure* framing |
| "Big swings in biopharma." | **Contracting.** Bio + Health drops to a single idea (Julie Yoo's "Healthy MAUs"), and it is a healthcare-services idea rather than a biopharma one |
| "Infinite games." | **Absent as a Games vertical.** Game-adjacent content survives only obliquely, via Jonathan Lai's world-models entry ("generative Minecraft") |
| "A nuclear resurgence." | **Absent as a 2026 entry.** Energy appears only inside the industrial-base framing (David Ulevitch, Erin Price-Wright, Ryan McEntush). See §7 for what happened to the 2025 nuclear entry |
| "'Faceless' creators." | **Transformed.** 2026's creative content is about control and multi-modality (Justine Moore, Yoko Li), not anonymity |
| "Google search challengers." | **Absent as a headline**, but structurally present: Stephenie Zhang's entry argues the whole optimise-for-humans playbook (Google ranking explicitly named) stops working for agent consumption |
| "Battlefield AI." | **Absent as an explicit entry.** Defense-adjacent content sits inside American Dynamism as industrial capacity (Erin Price-Wright), physical observability (Zabie Elmgren) and critical-industry data (Will Bitsky) |

And what is new in 2026 relative to the 2025 teaser: **crypto moves from one of eight verticals to a third of the entire edition** (17 of 47 ideas, a full day); **industrial/physical AI becomes the largest single theme** (6 American Dynamism + 6 Infrastructure entries touching physical data and compute); and **agents stop being a topic and become an assumption** — agents appear as a premise inside Infrastructure, Growth, Apps and crypto entries rather than as a standalone prediction.

### 6.4 What each edition states as its method — and what is absent from both

| Edition | Method statement, verbatim | What it does **not** say |
| --- | --- | --- |
| 2023 | "We asked dozens of partners across the firm to spotlight one big idea that startups in their fields will tackle in 2023" | No selection criteria; no count of ideas considered |
| 2024 | "We asked over 40 partners across a16z to preview one big idea they believe will drive innovation in 2024" | No selection criteria; "over 40" is imprecise |
| 2025 | "We asked 50 a16z partners to preview one big idea that will spur innovation in 2025" | No selection criteria; the number is precise, the choice of *those* 50 is unexplained |
| 2026 (P1) | "every December, we ask our investing teams to share one big idea they think tech builders will tackle in the year to come" | No selection criteria; no mention of how many contributors declined; the internal phrase "tech builders" defines the audience but not the evidentiary bar |
| 2026 (P3) | "17 things that various a16z crypto partners (plus a few guest contributors) **observe** about what's ahead" | No selection criteria; **the epistemic verb changes** and is not reconciled with Parts 1–2 |

The 2026 series does state a comparable method to prior years — Part 1's "every December we ask our investing teams" is a direct restatement of the 2023/2024/2025 construction, and is the strongest continuity claim the edition can make. But **Part 3 breaks the construction** without saying so: it switches from "one big idea… will tackle" to "17 things… observe", drops the vertical structure, and admits guest contributors. A reader who takes Part 1's method sentence as describing the whole edition would be reading Part 3 wrongly. The edition supplies three framings and does not reconcile them; that unreconciled seam is the single most interesting editorial fact in the 2026 package.

---

## 7. The Accountability Question

### 7.1 The finding is that the method cannot support a hit-rate

The obvious question about any annual prediction list is: *how often was it right?* This guide's answer is that **the question cannot be answered, and that is itself the finding.**

Four reasons, each independent and each sufficient:

1. **No independent body audits these lists.** Nothing in the corpus — not the 2023, 2024, 2025 omnibus pages, not the three 2026 parts — is accompanied by, or subject to, any third-party assessment. There is no auditor, no scoring rubric, no published post-mortem. This is a fact about the class of artefact (a venture firm's marketing content), not a finding about a16z.
2. **The firm has no obligation to revisit them.** Nothing in any edition commits the publisher to reviewing the prior year's claims. Four editions (2023, 2024, 2025, 2026) were published; **not one contains a retro section.** This guide searched the cached texts and found no scoring, no self-assessment and no follow-up.
3. **A multi-year trend prediction is not falsifiable on a one-year horizon.** "Creative tools go multimodal", "the electro-industrial stack will move the world", "the internet becomes the bank" are directional claims whose truth conditions are unspecified. There is no date by which they are wrong, only a direction in which they are more or less true. A claim with no failure condition cannot be scored.
4. **Attribution would be impossible even with a longer horizon.** Compound outcomes (a category matured, or did not) cannot be attributed to one firm's published thesis, because thousands of actors publish overlapping theses. Nothing here licenses a causal claim.

So: **this guide does not compute a hit-rate, does not imply one, and states that any published hit-rate for this class of artefact is unsupported by the available evidence.**

### 7.2 What can honestly be done instead

The honest alternative is a small, labelled sample with an explicit status for each. Five ideas from the 2023, 2024 and 2025 editions, chosen because they have *some* checkable reference point — not because the sample is representative. The labels are: **(i) too broad to be wrong**, **(ii) genuinely still open**, **(iii) documented**.

#### (i) Too broad to be wrong

**2023 — "Small modular reactors advance the nuclear renaissance" — Michelle Volz (American Dynamism team).**
The claim's own hedging is visible in the text: "SMRs and the broader nuclear industry are likely **poised for growth** in the year ahead," with the concession that "there is still a way to go in reforming the regulatory frameworks". There is no threshold, no date-specific outcome, and no entity. The theme then **recurred**: the 2025 omnibus leads its American Dynamism section with "The Resurgence of Nuclear" (David Ulevitch). A theme that appears across three editions without an adjudication point is the definition of too broad to be wrong. **Status: (i).** No source needed — the claim itself lacks a failure condition.

**2025 — "Super Staffing for Healthcare" — Julie Yoo (Bio + Health).**
The claim is that healthcare faces "the mother of all clinical staffing crises" and an administrative staffing paradox, and that AI-enabled "super staffing" will address both. This is a structural claim about a multi-decade labour market. It cannot be marked wrong in 2026, 2027 or 2030 without a specified counterfactual. **Status: (i).** Note the same author appears again in 2026 with "Healthy MAUs" — a different but adjacent healthcare-thesis framing, which is what a firm's practice does over time.

#### (ii) Genuinely still open

**2025 — "Every White-Collar Worker Will Have an AI Copilot" — James da Costa (Enterprise + Fintech).**
The claim: "I predict 2025 will be the year of AI copilots — eventually, every white-collar role will have one." Evidence offered: a study by OpenAI and the University of Pennsylvania finding that with LLM access about 15% of US worker tasks could be completed significantly faster at equal quality, rising to 47–56% with software and tooling on top of LLMs. The claim contains a short-horizon and a long-horizon element, and only the long-horizon part is assessable — and it is not yet decided. Adoption of copilots is uneven across roles, and the 2026 edition's own entries treat agent-mediated work as an *arriving* condition (Marc Andrusko's "death of the prompt box"; Seema Amble's orchestration layer; Malika Aubakirova's agent-speed workloads), not a completed one. **Status: (ii) — genuinely still open.** This guide states no outcome.

#### (iii) Documented

**2024 — "New Applications for Computer Vision and Video Intelligence" — Kimberly Tan.**
What is documented here is a fact about the document, not an outcome in the world: the 2024 entry names two companies as examples of the trend — **"Companies like Flock Safety and Ambient, for example, are both leveraging computer vision in the physical world"** — and **both names appear on a16z's own public investment list** (`a16z.com/investments/`; both strings verified by direct grep of the cached list, this pass). This is a dated, documented instance of an idea text naming companies that appear on the firm's own published holdings list. It is a fact of the document set; this guide attributes no motive and asserts no causal relationship between the idea's publication and any investment. **Status: (iii) documented fact — of the documents, not of the world.**

**2025 — "The Resurgence of Nuclear" — David Ulevitch (American Dynamism).**
This is the one case in the sample where an external, retrievable source exists. The 2025 entry states that hyperscale data centres "are already reviving decommissioned nuclear plants, including Pennsylvania's once-infamous Three Mile Island, slated to come back online in 2028." Checked against the Wikipedia article on the Three Mile Island Nuclear Generating Station (retrieved this pass): Unit 1 is owned by **Constellation Energy**; on **September 20, 2024**, Constellation "announced plans to reopen Three Mile Island Unit 1 in 2028"; the plant "will be renamed the **Christopher M. Crane Clean Energy Center**"; **Microsoft** entered a **20-year agreement** to purchase power from the facility, supporting its data-centre energy demand; Constellation will invest **$1.6 billion** in upgrades, pending regulatory approval; the article's infobox lists Unit 1's official name as "Crane Clean Energy Center" with one unit decommissioned and one "restart planned".

Now the honest reading, because it cuts both ways:

- **The claim checks out as a statement of fact.** Three Mile Island Unit 1 is indeed slated to return, with a named buyer and a stated date.
- **But the fact pre-dates the prediction.** The restart announcement is dated **September 20, 2024**; the 2025 omnibus page is a year-end artefact published in December 2024. The essay's headline evidence was therefore **already public when the essay was written** — it describes something already happening rather than forecasting it.
- **And the 2026 edition dropped the theme.** Energy returns only as a component of the industrial-base framing; nuclear is not a 2026 entry.

**Status: (iii) documented outcome — with the caveat that the documented thing pre-dates the page.** Note also the operative word in the entry: *slated*. The restart is planned, not completed; the 2028 date is a plan, and plans can slip. This guide does not claim the restart has happened.

### 7.3 The result of the exercise

Of five checked ideas: two are too broad to be wrong, one is genuinely still open, one is documented only as a fact about the documents, and one is documented in the world — and in that single case the documented fact was already public before the page that predicted it was published.

**A 5-item sample with this distribution is not a hit-rate and must not be read as one.** No selection rule was applied beyond "has some checkable reference point", the sample is deliberately biased toward checkability, and the labels are qualitative. What the exercise does establish is the shape of the accountability problem: for the overwhelming majority of the 47 ideas in any given edition, there is no artefact against which to check them, and for the minority where there is, the check tends to reveal a description of an already-public development rather than a forecast.

### 7.4 What would change the picture

Three things would, and none is available here: (a) a pre-registered scoring rubric published before the year begins, so outcomes can be graded without hindsight; (b) a named threshold per idea — a number, a date, a company class — so failure is definable; (c) a stated follow-up commitment from the publisher. None of the four editions contains any of the three. Their absence is the structural reason the accountability question has no empirical answer, and saying so is the answer.

---

## 8. The Enterprise and Regulated-Institution Read

### 8.1 The framing claim

**A venture firm's idea list is a demand-side inventory.** It is a good guide to what will be *sold* into enterprises over the next several years, because the firms publishing it are the ones funding the sellers. It is a poor guide to what an institution should *buy*, because it contains no evaluation of any specific product, no cost model, no integration reality and no regulatory analysis — and it is written to identify opportunity, not to advise procurement.

That asymmetry is not a criticism of the list. It is a statement about which question the list answers. A reader in an enterprise technology function can extract real value from it if they use it as a market map — *this is the vocabulary my vendors will use this year, these are the problems they will claim to solve, these are the categories that will be pitched* — and no value at all if they use it as a shopping list.

This section names **no bank and no institution as a user of anything**. The cross-references are to this repository's own technology-evaluation guides by filename, which are the places an institution's decision procedure actually lives.

### 8.2 Which 2026 ideas are decision-relevant to an enterprise technology function

| Idea | Author | Decision-relevant to an enterprise technology function? | Why / why not |
| --- | --- | --- | --- |
| Startups tame the chaos of multimodal data | Jennifer Li | **Yes — highest** | Directly about the data substrate that enterprise AI depends on; the claim ("data entropy" as the limiting factor) maps to a real internal constraint. Cross-reference [data_fabric_guide.md](data/data_fabric_guide.md) |
| Agent-native infrastructure becomes table stakes | Malika Aubakirova | **Yes — highest** | A capacity, concurrency and architecture claim; the "agent-speed" fan-out argument is a planning claim. Cross-reference [capacity_sizing_guide.md](capacity_sizing_guide.md) and [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) |
| The AI-native data stack continues to evolve | Jason Cui | **Yes** | The internal list names vector stores, the context problem and agentic BI — all live internal decisions. Cross-reference [data_fabric_guide.md](data/data_fabric_guide.md) |
| AI revives cybersecurity hiring | Joel de la Garza | **Yes** | Security operations staffing and tooling-sprawl are first-order enterprise concerns; also a claim about the firm's own portfolio category. Cross-reference [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) |
| Systems of record lose ground | Sarah Wang | **Yes** | A direct claim about ITSM/CRM architecture and vendor strategy — decision-relevant even if wrong, because the vendor landscape will argue it |
| AI creates a new orchestration layer—and new roles—in the Fortune 500 | Seema Amble | **Yes** | Organisational design and multi-agent coordination are enterprise deployment questions. Cross-reference [ai_adoption_strategies_guide.md](ai_llm/ai_adoption_strategies_guide.md) and [agentic_workflows_guide.md](agentic_workflows_guide.md) |
| Prompt-free and proactive applications arrive | Marc Andrusko | **Partly** | The product-interaction claim is relevant to internal tool selection; the KPI/UX framing is vendor-side |
| The end of the screen time KPI in AI applications | Santiago Rodriguez | **Partly** | Relevant to how AI value is measured and priced internally; the essay is aimed at vendors' pricing models |
| Vertical AI evolves from information retrieval and reasoning to multiplayer | Alex Immerman | **Partly** | Relevant to vertical-software procurement; the claim is about vendor strategy |
| AI will finally upgrade banking and insurance infrastructure | Angela Strange | **Partly — as a market claim only** | Directly about a regulated industry's vendor decisions, but framed as an investment thesis with a "10x" winner claim; no procurement guidance. See §11.3 |
| Forward-deployed motions take AI to the 99% | Joe Schmidt | **Partly** | Explains a vendor go-to-market pattern an institution will encounter; not itself a capability claim |
| New model primitives unlock previously impossible companies | Kimberly Tan | **Partly** | Useful as a capability map (reasoning, multimodality, computer use); the claim is about startup formation |
| Creating for agents, not humans | Stephenie Zhang | **No — venture-market** | A claim about the web's economics and start-up opportunity, not about institutional systems |
| The year we step inside video / Creative tools go multimodal / Healthy MAUs | Yoko Li / Justine Moore / Julie Yoo | **No — consumer, creative and healthcare-market** | Creative-tooling theses and a customer-segmentation thesis inside one industry's business model |
| World models take the spotlight in storytelling / "The year of me" / The first AI-native university | Jonathan Lai / Joshua Lu / Emily Bennett | **No — consumer and education** | Speedrun theses about new product categories |
| All six American Dynamism ideas | Ulevitch, Price-Wright, Elmgren, McEntush, Hsu, Bitsky | **No — industrial/defence-market** | Notable that "observability" appears here as *physical* infrastructure monitoring; it is a different discipline from software observability |
| The remaining Apps ideas (AI app store, voice agents, consumer "see me", AI-selling-to-AI) plus all 17 crypto ideas | various | **Mixed / mostly no** | The remaining Apps entries are consumer or vendor-strategy claims. Of the crypto entries, three carry enterprise consequence: Sean Neville's KYA (identity for non-human agents, which any institution deploying agents will meet), Sam Broner's core-ledger/stablecoin argument (which describes a banking infrastructure reality), and Miles Jennings's legal-architecture claim (a regulatory-condition claim). The rest are crypto-market theses |

### 8.3 The honest summary of the enterprise read

- **Roughly a third of the 47 ideas have direct enterprise consequence** — the infrastructure and data entries, the security-hiring entry, the orchestration entry, and parts of the Apps vertical.
- **A second third has indirect consequence** as a description of what vendors will be selling.
- **The final third has none** — consumer, creative, education, industrial and crypto-market theses.
- **Nothing in the edition is a procurement recommendation**, and no entry could be turned into one without the missing pieces: cost models, integration constraints, failure modes, regulatory position, and evidence about a specific product. Cross-references for that work are the repository's own: [llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md) for evaluation, [finops_guide.md](finops_guide.md) for unit economics, [capacity_sizing_guide.md](capacity_sizing_guide.md) for sizing, [ai_adoption_strategies_guide.md](ai_llm/ai_adoption_strategies_guide.md) for organisational readiness, and [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) for the agent-era security surface.

### 8.4 One short worked example, naming no institution

*An abstract illustration only — no bank, no real institution, no named vendor.*

Suppose an enterprise technology function is planning its next year and a business unit arrives with a vendor proposal justified by "agent-native infrastructure" and "agent-speed workloads". The 2026 edition's Malika Aubakirova entry is useful here in exactly one way: it supplies the *question* the proposal should answer — does the platform's backend survive a single agentic goal triggering a recursive fan-out of thousands of calls, or was it built for a 1:1 human action-to-response ratio? That question is a genuine planning question and it comes, unmodified, from a published venture thesis.

The entry is useless for the next steps. It does not say what fan-out factor is realistic, what latency envelope matters, what the failure modes are, or what it costs. Those come from capacity sizing, load testing and unit-economics work — the disciplines owned by [capacity_sizing_guide.md](capacity_sizing_guide.md), [agentic_workflows_guide.md](agentic_workflows_guide.md) and [finops_guide.md](finops_guide.md) in this repository. The correct use of the idea list is to harvest the questions; the answers must come from elsewhere.

---

## 9. The Self-Interest Map

This section is short, sourced and dated on purpose. **It is explicitly not a claim about anyone's intent.** Everything below is either the firm's own published words or a verifiable string in a cached page, and each item is marked with what was verified in this pass.

### 9.1 The firm's own disclaimer, quoted

a16z's standard disclosure footer appears on **all four cached 2026-era pages** (the three newsletter parts) **and on the 2025, 2024 and 2023 omnibus pages**. Verbatim fragments, all verified by direct grep of the cached page text:

> "Any investments or portfolio companies mentioned, referred to, or described are not representative of all investments in vehicles managed by a16z… A list of investments made by funds managed by a16z is available here: https://a16z.com/investments/."

> "As for its investments in any cryptocurrency or token project, a16z is acting in its own financial interest, not necessarily in the interests of other token holders."

And, present in the **2023 and 2024** editions (cached text verified for 2023; the 2025/2024 pages carry the longer variant):

> "This content is provided for informational purposes only, and should not be relied upon as legal, business, investment, or tax advice."

Other fragments present in the footers of the cached 2025 and 2024 omnibus pages include: "The posts are not directed to any investors or potential investors, and do not constitute an offer to sell — or a solicitation of an offer to buy — any securities, and may not be used or relied upon in evaluating the merits of any investment." and "All content speaks only as of the date indicated."

**This is the firm's own published statement of the conflict.** The guide cites it as such and adds nothing to it: the disclaimer exists, it is on every page analysed here, it is a footer rather than an entry-level disclosure, and it explicitly names the cryptocurrency/token conflict in the firm's own words.

| Disclaimer fragment | 2023 | 2024 | 2025 | 2026 P1–P3 |
| --- | --- | --- | --- | --- |
| "not representative of all investments in vehicles managed by a16z" | ✅ found | ✅ found | ✅ found | ✅ found (all three parts) |
| "acting in its own financial interest, not necessarily in the interests of other token holders" | ❌ not found in the cached 2023 text | ✅ found | ✅ found | ✅ found (all three parts) |
| "This content is provided for informational purposes only… should not be relied upon as legal, business, investment, or tax advice" | ✅ found | ❌ not found in cached text | ❌ not found in cached text | ❌ not found in cached text |
| "not directed at nor intended for use by any investors or prospective investors" | ✅ found | ❌ not found in cached text | ❌ not found in cached text | ❌ not found in cached text |
| "The posts are not directed to any investors or potential investors" | variant not found | ❌ not found in cached text | ✅ found in cached text | ✅ found (all three parts) |

⚠️ **Note the honesty requirement on that table.** Every "❌ not found" cell means exactly that — the string was not found in the cached page text available to this pass. **It is not evidence the sentence is absent from the live page.** The footers differ by edition and by capture format: the 2023 page's cached footer carries the "informational purposes only / not directed at nor intended" wording; the 2024 and 2025 cached footers carry the longer "Views expressed in 'posts'…" block, which contains the "not directed to" phrasing; the 2026 pages carry the "Views expressed in 'posts'" block plus the investments/token sentences. A reader should treat the footer wording as **variable across editions and across capture formats** and should read the live page for the current version. The one row that is a genuine cross-edition observation rather than a capture artefact is the second: the token sentence was found on 2024, 2025 and 2026 pages and **not** on the cached 2023 page. See §15.

### 9.2 The same pages carry the firm's investment announcements

The 2026 pages publish the ideas **and**, in adjacent cards, disclose the firm's investments in the same themes. Verified strings in the cached part texts (each card is a title line plus a byline line; each was checked by direct grep):

| Card string (cached 2026 page) | Names on the card | Theme overlap with an idea on the same page |
| --- | --- | --- |
| "Investing in Vals" | Jennifer Li, Yoko Li, Raghu Raghuram, and Shangda Xu | Adjacent to Jennifer Li's multimodal-data and Yoko Li's video entries (both bylined on the idea side) |
| "Investing in Exa" | Sarah Wang, Jennifer Li, Stephenie Zhang, and Jason Cui | Adjacent to the data/infrastructure entries by those same contributors |
| "Investing in Neo" | Zane Lackey and Joel de la Garza | Security — the category of Joel de la Garza's cybersecurity-hiring entry |
| "Investing in Keycard" | Zane Lackey, Yoko Li, Joel de la Garza, and Malika Aubakirova | Security/agents — the category of Malika Aubakirova's agent-native entry |
| "Investing in Highstock" | Olivia Moore and Justine Moore | Creative/AI tooling — Justine Moore's creative-tools entry |
| "Investing in Westmag" | Erin Price-Wright and Oliver Hsu | American Dynamism — the same practice as both contributors' entries |
| "Investing in Endra" | Joe Schmidt, David Haber, Caroline Goggins, and Zabie Elmgren | Adjacent to three idea authors on the same page |
| "Investing in Jito" | Ali Yahya | Crypto infrastructure — the author of the privacy-moat entry |
| "Investing in Seismic" | Ali Yahya and Carra Wu | Crypto privacy — the same theme as the privacy-moat entry |
| "Investing in Catena Labs" | (card present in Part 3 cache) | Catena Labs is the company led by Part-3 guest contributor Sean Neville |
| "Investing in Mirendil" · "Investing in Town" · "Investing in Glif" · "Investing in Netris" · "Investing in Inferact" · "Investing in Runta" · "Investing in Gimlet" · "Investing in Temporal" · "Investing in Kavak" | (cards present in Part 1 cache; the last four are fintech/enterprise and creative-tooling items) | Same part as the Infrastructure/Growth entries |
| "Investing in Ulysses" · "Investing in Chariot Defense" · "Investing in Heron Power" · "Investing in Westmag" | (cards present in Part 2 cache; Westmag listed in the table above) | American Dynamism themes — the same vertical as four of the six Part 2 entries |
| "Investing in Probook" · "Investing in Convey" · "Investing in Sphere" · "Investing in Lightfield" | (cards present in Part 2 cache) | Apps/enterprise themes |
| "Investing in Hilbert" · "Investing in Oboe" · "Investing in Ethos" · "Investing in Petual" · "Investing in Stitch" | (cards present in Part 2 cache) | Fintech/consumer |
| "Investing in Kairos" · "Investing in Babylon" · "Investing in Daylight" · "Investing (again) in Dynamic Labs" · "Investing in Blackbird" | (cards present in Part 3 cache) | Crypto — the same practice as the whole of Part 3 |

**What this establishes, factually:** the same page that publishes a thesis discloses, in a neighbouring module, the firm's investment in companies in the same category — sometimes with the same person's name on both. That is a factual observation about the page structure, verifiable by anyone who reads the cached text.

**What it does not establish, and what this guide does not claim:** anything about intent, causality, sequencing or disclosure adequacy. The firm publishes a public investment list; the entries and cards are consistent with that list; the legal footer disclaims representativeness. No motive is attributed to any person named above.

### 9.3 A prior-edition case, dated

The 2024 edition's American Dynamism idea **"New Applications for Computer Vision and Video Intelligence"** (Kimberly Tan) names two companies as examples: **Flock Safety** and **Ambient**. Both names appear on a16z's own published investment list (verified by grep of the cached list this pass; `Flock` appears twice, `Ambient.ai` once).

**This is a dated, documented case of an idea text naming companies that appear on the firm's own investment list.** Date of the idea: the 2024 edition. Date of the observation: this pass. It is stated as a fact of the documents, with no motive attributed and no causal claim made.

### 9.4 The list is repackaged across the firm's own channels

Verified strings in the cached 2026 part texts:

- Part 2 carries a card for a podcast episode titled **"Big Ideas 2026: Physical AI and the Industrial Stack"** (Erin Price-Wright, Ryan McEntush, Zabie Elmgren, and Will Bitsky) — the same four contributors whose American Dynamism entries are in Part 2, repackaged in audio. The card appears twice in the cached Part 2 text.
- Part 2 carries a card for **"The Machine Age Fund"** (Ben Horowitz, Martin Casado, Raghu Raghuram, David Ulevitch, and David George) — a fund page, not an idea.
- Part 3 closes with: "And then tomorrow, we'll finish the week with a special announcement and an invitation from a16z that you won't want to miss."
- Every part carries a card titled **"You can just read 25 books"** bylined "a16z New Media", and newsletter subscribe prompts ("This first appeared in the a16z News newsletter. Subscribe to stay on top of the latest news."), plus an "Expert News by a16z" block and a "See All Newsletters" link.

**What this establishes:** the Big Ideas edition is one artefact inside a content operation with multiple distribution surfaces — newsletter, podcast, fund page, and a promised follow-up announcement. These are facts about the artefact's function as publishing and as communication with the firm's audience. They are not evidence of anything about the truth of any idea, and this guide draws no such inference.

### 9.5 What was omitted from this section, and why

| Claim considered | Disposition | Reason |
| --- | --- | --- |
| "The ideas are chosen to promote the portfolio" | **Not asserted** | No evidence available to this pass could establish intent, and absence of evidence is not evidence. The observable facts are stated in §9.2–9.3 without the inference |
| Any statement about fund performance, returns, or fundraising amounts | **Omitted** | Not present in the cached sources; not verifiable here |
| Any claim about the firm's positions in specific companies beyond the two grep-verified names in §9.3 and the card titles in §9.2 | **Omitted** | Only named what the sources name |
| Any claim about a competitor firm's comparable conflicts | **Omitted** | Not in scope and not verifiable here |
| The word "conflict of interest" applied to any person | **Not used** | The disclosure exists, is published, and is the firm's own statement; the guide reports it rather than characterising it |

---

## 10. How to Read a Venture Firm's Annual List

A practical method, in five uses, each with its honest limit.

### 10.1 Use it as a demand-side inventory

**Do:** read the list as a forecast of what will be *pitched* to you, not of what will be *true*. Every entry names a category that will have funded companies in it within eighteen months. If a business unit brings you a proposal using the year's vocabulary, you now know where the vocabulary came from and what the vendor's investors believe.

**Limit:** it tells you what will be sold, not what works. A funded category is not a validated one; several categories in the list will be sold hard and produce little.

### 10.2 Use it as a map of where a well-capitalised competitor is placing attention

**Do:** compare the verticals and the ideas against the same firm's prior editions. Three things become visible and are not visible from a single year: which themes recur (nuclear, agents, vertical AI), which labels disappear (Games, Consumer Tech, Enterprise + Fintech in 2026), and where the editorial investment went (2026 moved crypto from one-eighth to one-third of the edition). These are signals about where a firm with very large funds is directing attention.

**Limit:** attention is not capital. A published idea and a deployed dollar are different objects, and the list does not tell you the size of any position. Use the firm's public investment list as a separate artefact, and cross-check.

### 10.3 Use it as one sophisticated firm's model of the next few years

**Do:** read the essays for the *mechanism* — the causal story inside the claim. Jennifer Li's mechanism is that data entropy caps agent reliability; Malika Aubakirova's is that agent fan-out breaks a 1:1 concurrency assumption; Sarah Wang's is that models reading operational data demote the system of record. Even when you reject the prediction, the mechanism is a testable proposition about how systems behave, and mechanisms are more durable than predictions.

**Limit:** one firm's model, selected for presentability. A firm publishes ideas that are (a) strategically comfortable, (b) attractively framed and (c) statable in 400 words. Ideas that are true but strategically uncomfortable, or true but commercially uninteresting, are not in the list (§5.3).

### 10.4 Use it as a deliberate re-read, one year later

**Do:** save the edition, note today's date, and re-read it in twelve months *against what actually happened* — with the discipline of §7. Grade mechanisms, not slogans. Note which named companies still exist, which categories acquired a standard vendor, which claims quietly vanished. This is the single highest-value use, and it costs nothing but the note file.

**Limit:** the re-read will grade your own judgement as much as the firm's, because most claims cannot be graded at all. Plan for a small number of gradable items and a large number of "still open" — that is the real shape of the exercise, not a failure of the method.

### 10.5 Use it as a vocabulary source, correctly

**Do:** take the terms the edition introduces into circulation — "data entropy", "agent-native infrastructure", "prompt-free", "forward-deployed", "KYA" — and treat them as *market vocabulary* that will appear in vendor decks, analyst notes and internal slide packs for the next year. Understanding a term's origin and its intended claim is how you avoid arguing with a slogan.

**Limit:** a term in circulation is not a definition. "Agent-native" and "data entropy" have no agreed technical content; if a proposal depends on one of them, the first task is to force a definition, not to evaluate the claim.

### 10.6 The one-page method

The five uses, compressed into four steps (the first two merged):

| Step | Action | Artefact produced |
| --- | --- | --- |
| 1 | Read the list as demand-side inventory, then diff it against the prior edition's verticals, teaser and entries | A list of categories your vendors will sell into next year, plus a short note on what appeared, vanished and moved |
| 2 | Extract the *mechanisms*, not the predictions | A testable claim per idea you care about |
| 3 | Cross-check named companies against the firm's public investment list | A factual conflict map, dated |
| 4 | Diary a re-read for twelve months hence, with a rubric written **now** | A gradable sample, and an honest count of the ungradable remainder |

The rubric written *now* is the whole trick. A rubric written after the fact is how prediction lists acquire imaginary hit-rates.

---

## 11. The 2026 Ideas Read Critically

Five of the edition's ideas, read with the same four questions:

1. **What is claimed?** — the proposition, stated in the entry's own terms.
2. **What evidence does the piece offer?** — and what kind: a statistic, a named example, an analogy, or nothing.
3. **What would have to be true for it to be right?** — the conditions, not the conclusion.
4. **What kind of statement is it?** — a prediction, a description of something already happening, or a thesis about where to invest.

Quotes are exact and attributed to the named author.

### 11.1 Jennifer Li — "Startups tame the chaos of multimodal data"

**Claimed:** Unstructured, multimodal data is enterprise AI's binding constraint, and the limiting factor is "data entropy: the steady decay of freshness, structured, and truth inside the unstructured universe where 80% of corporate knowledge now lives." Enterprises need continuous clean/structure/validate/govern systems, and startups building that platform "hold the key to the kingdom of enterprise knowledge and process."

**Evidence offered:** a **quantitative claim without a citation** — the "80% of corporate knowledge" figure. The entry also offers an enumerated list of use cases (contract analysis, onboarding flows, claims handling, compliance, support, procurement, engineering search, sales enablement, analytics pipelines) and a stated consequence chain (messier inputs → RAG hallucination → subtle expensive agent failure → remaining human QA). The kind of evidence is: one uncited statistic, plus an argument by consequence. No company is named, no benchmark is cited, no cost figure is given.

**What would have to be true:** (a) that the "80%" figure is roughly right and stable; (b) that data entropy is a *causal* limiter rather than a co-symptom of poor process design; (c) that structuring it is a platform opportunity rather than a services market that gets absorbed by the systems that generate the data; (d) that a continuous-governance product can be monetised above the cost of the model calls it consumes. The entry argues (a) and asserts (b)–(d).

**Kind of statement:** a **description of a present condition plus an investment thesis**. The condition — enterprises have messy multimodal data and it breaks agent reliability — is not a 2026 forecast; it is the observable present. The forecast is the claim about *who captures the value*.

**Reading:** the strongest part of this entry is its mechanism, and the mechanism is not the 80% number. The claim that messy inputs cap agent reliability at a level no model improvement fixes is the testable proposition; it predicts that agent failure rates will track input quality rather than model quality. That is checkable inside any enterprise pilot, and it is the claim worth carrying. The 80% figure should be treated as **rhetorical scaffolding until a source is supplied** — ⚠️ flagged, see §14.

### 11.2 Joel de la Garza — "AI revives cybersecurity hiring"

**Claimed:** The security hiring shortage was caused by the industry's own tooling: security teams hire skilled technicians to do level-1 work created by products that "detect everything," and AI-native tooling will break this cycle, so that "In 2026, AI will break this cycle and close this hiring gap." Freed-up staff will "chase down bad guys, build new systems, and fix vulnerabilities."

**Evidence offered:** a **dated statistic with no source line** — "From 2013 to 2021, the number of unfilled cybersecurity jobs grew from under 1M to 3M" — plus a causal explanation (detect-everything tooling creates the labour demand) and a personal-credibility appeal ("Anyone who has ever spent time working on a large security team knows that half the work is easily solved with automation"). The kind of evidence is: one trend statistic, one mechanism, one appeal to practitioner experience.

**What would have to be true:** (a) that the unfilled-jobs figure counts genuine unmet demand rather than posted roles and definitional inflation (a well-known problem with this specific statistic class); (b) that the level-1 work is genuinely automatable to the required reliability; (c) that the freed capacity is redeployed rather than absorbed into expanded detection scope — which is precisely what the entry says happened the first time; (d) that "close this hiring gap" is a 2026 event rather than a direction.

**Kind of statement:** a **prediction of direction with a weak time bound**. The underlying mechanism is a version of induced demand, which is plausible and has been observed in adjacent operations fields. The 2026 date is the soft part: the entry gives no mechanism by which a staffing gap closes within a single year.

**Reading:** note what the entry implicitly concedes. If security teams created the drudgery by buying tools that detect everything, then the AI-native tools it recommends are the same class of purchase. The failure mode the entry diagnoses — buy a detection layer, create review labour — is not ruled out for the successor tool; the entry's own logic implies that "AI-native" must be evaluated on whether it *reduces total detection scope*, not whether it automates triage. That is a genuinely useful procurement test, and it is derived from the entry, not imported into it. The structural point for §9: this is an entry by a partner who "focuses on information security related investments", and the category it recommends is the category in which that partner's practice invests.

### 11.3 Angela Strange — "AI will finally upgrade banking and insurance infrastructure"

**Claimed:** Institutions have bolted AI onto legacy systems; real transformation requires rebuilding the infrastructure beneath. In 2026, "the risk of not modernizing to take full advantage of AI will outweigh the risk of failure," and "we'll see large financial institutions let their legacy vendor contracts lapse and start implementing newer, AI-native alternatives." The new platforms "centralize, normalize, and enrich underlying data from legacy systems and external sources." Consequences: workflows "dramatically streamlined and parallelized"; categories merge (KYC + transaction monitoring "could now sit together in a single risk platform"); and "The new winners of these categories will be **10x the size of the older incumbents**: the categories are much bigger, and the software market is eating labor."

**Evidence offered:** an architectural argument (bolt-on vs rebuild) plus a derived market-size claim ("10x"), justified by two assertions — that the categories are larger and that "software is eating labor". The entry offers a concrete workflow illustration (a mortgage loan-origination system's hundreds of tasks, with agents completing the mundane ones). The kind of evidence is: an architectural argument, one workflow example, and an uncited market-size multiplier. No institution is named as a buyer, and no contract is named as lapsing.

**What would have to be true:** (a) that large regulated institutions can change core vendors within the stated window — against a very long historical cycle for core replacement; (b) that the risk asymmetry genuinely inverts, which is a claim about board-level decision-making the entry asserts rather than demonstrates; (c) that the merged-category platform is achievable without the regulatory separation that created the seams; (d) that the "10x" multiplier is a prediction rather than a framing device.

**Kind of statement:** primarily a **thesis about where to invest**, dressed as a prediction. The prediction is the 2026 timing; the substance is the 10x claim about category structure. That is a claim about the returns of the companies the entry's authors would like to fund, and it is framed as a market-structure observation.

**Reading:** this is the most directly relevant entry in the edition to a regulated institution's vendor strategy, and simultaneously the one with the least institution-facing evidence. It tells you the firm expects institutions to replace core systems and that it is funding the replacements; it does not tell you which replacement, at what cost, under what regulatory constraint, or with what migration path. For an institution, the useful operation is to invert it: if the thesis is right, the incumbent vendors will spend the next two years arguing that it is wrong, and the procurement question is what evidence either side can produce. The entry supplies none — it is a claim about a market, and the reader should treat the "10x" as the marketing surface it is. (It should also be noted that the essay's own framing contradicts a strict "2026" reading: it describes institutions integrating "document ingestion and AI voice agents on top of their legacy systems" as the already-existing condition, and core replacement as the *next* step.)

### 11.4 Andrew Hall — "Prediction markets go bigger, broader, and smarter"

**Claimed:** Prediction markets have gone mainstream and will become "bigger, broader, and smarter as they intersect with crypto and AI," while posing new problems for builders.

**Evidence offered:** this entry offers the **most concrete problem list** in the edition, which is what makes it worth reading critically. Specifically:

- More contracts will be listed, moving beyond "major elections or geopolitical events" to "in-the-weeds outcomes and complex, intersecting events." Raising (the entry says) societal questions about transparency and auditability.
- Resolution is the bottleneck: "Centralized platform resolution (did a given event actually happen? how do we confirm it?) is important, but disputed cases like the **Zelensky suit market** and the **Venezuelan election market** show the limits."
- Proposed directions: "new kinds of decentralized governance and LLM oracles can help determine truth for contested outcomes."
- AI beyond oracles: agents "can scour the world for signals that help provide short-term trading edge"; the entry cites "**Projects like Prophet Arena** already hint at the excitement in this space."
- On polling: "Do prediction markets replace polling? **No**; they make polling better (and polling information can be fed into prediction markets)."

The kind of evidence is: named real-world disputed markets, one named project, and an explicit falsification-adjacent statement about polling. **This entry carries a visible editorial defect**: the published text contains the literal string `<SC will link to our article here>` — an unresolved placeholder in the source. That is worth recording because it is a fact about the artefact's production standards.

**What would have to be true:** (a) that listing many more contracts does not degrade the liquidity and informativeness that make the existing ones useful — thin markets are noisy ones; (b) that LLM oracles can be made trustworthy for contested outcomes, which requires the oracle itself to be adjudicable (a recursion the entry names but does not resolve); (c) that agent trading adds information rather than extracting it; (d) that the transparency/auditability problem is addressable rather than intrinsic to the product.

**Kind of statement:** closest to a **description plus an open-problems list** — and notably, the author's bio says "is a research advisor to the a16z crypto team", not an investing partner. The entry reads like an academic's research agenda for a field rather than a capital-allocation thesis. It is the least promotional entry in Part 3 for exactly that reason.

**Reading:** of all 47, this is the entry that most repays a second read, because it states what would make its own subject fail. It names disputed cases as limits *of the mechanism itself* rather than as teething problems. That is a rarer move in this genre than it should be (§5.3).

### 11.5 Miles Jennings — "Unleashing the full potential of blockchains… when legal architecture finally matches technical architecture"

**Claimed:** US legal uncertainty was the decade's central barrier to blockchain networks; it distorted behaviour ("Founders were told to avoid transparency. Token distributions became legally arbitrary. Governance became theater"), and market-structure legislation — "which the government is closer to passing than it's ever been" — could remove those distortions by "incentiviz[ing] transparency, creat[ing] clear standards, and replac[ing] 'enforcement roulette' with more clear, structured paths for fundraising, token launches, and decentralization."

**Evidence offered:** a historical narrative of distortion, a conditional claim about pending legislation ("has the potential to eliminate all of these distortions **next year**"), and one supporting data point: "After **GENIUS**, the proliferation of stablecoins has exploded." The kind of evidence is: a narrative, a conditional, and one named legislative precedent. No bill number, no vote count, no timeline.

**What would have to be true:** (a) that the legislation in question passes in the stated window — the entry itself uses "closer than it's ever been" and "if passed", so it already conditions the claim; (b) that the distortions were caused by legal uncertainty rather than by the incentive to maximise token value, which legal clarity would not remove; (c) that clarity produces "open, autonomous, composable, credibly neutral" networks, rather than simply more compliant corporate structures with tokens attached.

**Kind of statement:** an **explicitly conditional prediction about a regulatory outcome**, used as the premise of a thesis about where to invest. Of the five read here, it is the most honestly hedged: it says "if passed" and it says "has the potential to". Its weakness is not the hedge but the causal step from legal clarity to the specific properties (credible neutrality, decentralisation) the author wants.

**Reading:** the entry's own GENIUS→stablecoins observation is the one part with a dated referent, and it is worth noticing what it establishes: an analogy from a prior legislative change to an expected behavioural response. That is a reasonable argumentative form and a weak evidentiary one — it shows the mechanism can operate, not that this instance will. Note the byline: "is General Counsel & Head of Decentralization for a16z crypto." The entry is a legal-strategy thesis written by the firm's own counsel, published in the firm's own ideas package, in a year the firm's crypto practice received a third of the space. All four of those are facts; the guide draws no conclusion from their conjunction beyond the observation that they are conjoined.

### 11.6 What the five reads have in common

Across the five: Jennifer Li offers one uncited statistic plus a consequence chain; Joel de la Garza one uncited trend statistic plus a mechanism; Angela Strange an architectural argument, a workflow example and an uncited "10x" multiplier; Andrew Hall named disputed markets and an explicit non-claim about polling; Miles Jennings a narrative, a conditional and one legislative analogy. Three of the five state no failure condition at all, one states a partial one (Hall names limits of the mechanism), and one states a condition explicitly (Jennings's "if passed"). **That distribution is the empirical content of §7's finding, arrived at from the other direction: reading the essays for falsifiability produces the same conclusion as attempting to grade them.**

### 11.7 One internal-list entry worth flagging separately

**Jason Cui — "The AI-native data stack continues to evolve"** is the only entry in the 2026 edition containing an explicit internal list, headed **"A few ideas we're excited by:"**. Its three items are: data flowing "into performant vector databases alongside traditional structured data"; AI agents navigating "the context problem" by "continuously accessing the right data context and semantic layers"; and change in "traditional BI tools and spreadsheets" as "data workflows become more agentic and automated."

Two observations. First, this is a **structural deviation** — one "big idea" entry contains three subordinate ideas, which is why counting "ideas" is a softer operation than the edition's framing suggests. Second, the entry's evidence base is **named industry events rather than statistics**: the Fivetran/dbt merger and the continued rise of Databricks. That is a different evidentiary register from Jennifer Li's uncited 80% or Angela Strange's uncited 10x — it points at things a reader can look up. It is the most checkable entry in the edition, and deserving of the flag for that reason rather than because its claim is stronger.

---

## 12. The Adjacent Artefacts

Briefly, and only what can be verified. This section names no competitor's "big ideas" page, because this pass could not verify that any exists — `web_search` returned empty result sets (see §2.5 and §15), so no other firm's equivalent artefact could be confirmed.

### 12.1 What is verified to exist

| Artefact | Status | Evidence |
| --- | --- | --- |
| **a16z, "Big Ideas in Tech for 2023: An a16z Omnibus"** | ✅ exists | byline "a16z editorial", posted **December 15, 2022**; framing: "This time of year, the floodgates open to a deluge of best-of lists and end-of-year recaps. But we at a16z have always been more interested in what's to come." |
| **a16z, "Big Ideas in Tech" 2024** | ✅ exists | single omnibus page; framing: "We asked over 40 partners across a16z to preview one big idea they believe will drive innovation in 2024" |
| **a16z, "Big Ideas in Tech" 2025** | ✅ exists | single omnibus page; framing: "We asked 50 a16z partners to preview one big idea that will spur innovation in 2025"; teaser names eight verticals and previews "An external 'AI brain.' Big swings in biopharma. Infinite games. A nuclear resurgence. 'Faceless' creators. Google search challengers. Battlefield AI." |
| **a16z, "Big Ideas 2026" (Parts 1, 2, 3)** | ✅ exists | three newsletter pages, dated Dec 9/10/11 2025, byline "a16z New Media" |
| **The 2026 omnibus URL** | ❌ does not exist | `a16z.com/big-ideas-in-tech-2026/` returns "Page not found \| Andreessen Horowitz" — re-verified this pass |
| **a16z public investment list** | ✅ exists | `a16z.com/investments/`, cited by the firm's own disclaimer footer |
| **a16zcrypto's annual "things we're excited about" series** | ✅ referenced from the cached 2026 text | Part 3 carries a card: **"A Few of the Things We're Excited About in Crypto (2024)"** — Andrew Hall, Miles Jennings, Scott Duke Kominers, Eddy Lazzarin, Daren Matsuoka, Sam Ragsdale, Daniel Reynaud, Arianna Simpson, Carra Wu, and Ali Yahya. Also present: a card for **"Watch: 11 Ideas for AI x Crypto"** — Robert Hackett, Sam Broner, Scott Duke Kominers, Elizabeth Harkavy, Carra Wu, Jay Drain, and Matt Gleason |
| **a16z's site-wide card inventory** | ✅ present in the cached 2026 text | "How to Hire a Chief of Staff" (Cassady Churchill and Jackie McQuiston), "How Big Bank Fees Could Kill Fintech Competition (July 2025 Fintech Newsletter)" (James da Costa, Alex Rampell, Angela Strange, and David Haber), "How Enterprise AI Startups Are Rewriting the Benchmarks (July 2025 Enterprise Newsletter)" (a16z Enterprise Team), "CFO roundtable: AI growth, pricing, and forecasting (June 2025 Fintech Newsletter)" (Ivan Makarov and James da Costa), "The State of Generative Media 2026" (Jennifer Li and Justine Moore) |

### 12.2 What follows from the adjacent artefacts, and what does not

**Follows:** the Big Ideas package is one node in a larger publishing graph. The crypto practice runs its own annual list ("Things We're Excited About in Crypto"), the fintech and enterprise practices run monthly newsletters, the enterprise team publishes benchmark-facing content, and the media/creative practices publish state-of-the-industry pieces. A reader who wants the firm's full view of a category should read the practice newsletter, not just the December package.

**Follows:** the crypto annual list is a **second, separate annual-list artefact from the same firm**, published on its own cadence, with overlapping contributor sets (Andrew Hall, Miles Jennings, Scott Duke Kominers, Arianna Simpson and Ali Yahya appear on both the crypto series card and the 2026 Part 3 idea list). Anyone trying to count "a16z's big ideas" must decide which artefact they mean.

**Does not follow:** anything about other firms' artefacts. This pass could not verify that any competitor publishes a comparable annual list, so this guide asserts nothing about one. Stating the absence honestly is the correct output (§15).

---

## 13. The Anti-Patterns

Seven failure modes, each with the symptom, the cause and the guardrail. These are observed in circulation; they are the recurring errors in how this class of artefact is discussed.

| Anti-pattern | Symptom | Cause | Guardrail |
| --- | --- | --- | --- |
| **13.1 Treating a VC list as a market forecast** | "a16z predicts X in 2026" appears in a slide, a strategy memo or a vendor deck as if it settled a question about the market | The list's grammar is predictive and its publisher is authoritative; the position-framing is invisible unless you go and look for the firm's holdings and funds | Restate every claim with the speaker attached: "a16z's infrastructure team believes…" not "the market is moving toward…". Then ask what evidence the entry itself supplied — usually less than the restatement implied |
| **13.2 Citing a prediction as evidence something is happening** | A prediction is cited as proof of the trend it predicts, producing a closed loop where the list justifies the list | A published thesis by a well-known firm functions as social proof; the citation chain rarely reaches a primary measurement | Separate the claim from its referent. A prediction has one piece of evidence — the prediction — and it is not evidence of the world. Ask for the underlying measurement or example, and if there is none (§11), record the entry as unevidenced |
| **13.3 Reading a thesis without noticing the firm is a holder of it** | A category is described as "validated by a16z" when the validation is a thesis by a partner whose practice invests in that category, published on a page that also carries the firm's investment announcements | The idea text and the investment cards sit in different visual modules of the same page; the disclaimer is a footer | Before quoting the thesis, open the firm's public investment list and check the named companies (§9.3). Record the check — and record it as a check, not as a finding about intent |
| **13.4 Extrapolating one partner's essay into a firm-wide position** | "a16z thinks healthcare will…" from a single Bio + Health entry, or "a16z is betting on nuclear" from one American Dynamism essay | The package reads as a single document, so its parts are read as a single position; the vertical groupings reinforce the impression | Attribute to the person, not the firm (§4.5). Note that the edition contains no reconciliation between entries and no firm-level statement; where entries sit in tension — Infrastructure's six ideas are not sorted into a priority order — the tension is unresolved on the page |
| **13.5 Taking a 2025 idea's status as settled because the 2026 list still mentions the theme** | "Agents were the big 2025 story and a16z confirms it in 2026", which silently converts a recurring theme into a confirmed outcome | Continuity feels like confirmation. It is not: a firm can carry a theme for years without the world adjudicating it (§7.2, nuclear) | Separate continuity from adjudication. A theme appearing in two editions is evidence about the *publisher's* attention, not about the theme. Only an external measurement can settle a theme, and those are rare |
| **13.6 Reproducing a list without its dates** | A list of "a16z's 2026 predictions" appears with no publication date, no part numbers, and no indication that the edition is three separate pages | The artefact invites paraphrase; the flat list of titles is portable and the dates are not | Cite Part 1/2/3 with December 9/10/11 2025. Note that the page byline is "a16z New Media" and the idea bylines are individuals. Note also the separate crypto annual list (§12) — an undated "a16z big ideas" citation is ambiguous between two different artefacts |
| **13.7 Reporting more than 47 ideas (specific to this edition)** | A scraped inventory reports more than 47 ideas, or attributes an idea to a contributor who did not write one | The pages interleave real entries with promotional and related-content cards, some of which carry plausible-sounding titles (e.g. "The Cinderella 'Glass Slipper' Effect: Retention Rules in the AI Era"; "The Palantirization of everything") | Apply the §3.8 audit rule: title + bylined author + body essay + "About the Contributor" bio block. A card has title + name only. The extra "About the Contributor / a16z New Media" block at the end of each part is the page byline, not an entry |

---

## 14. The Claims Audit

Three separated tables, as required: (i) what the edition says, (ii) what is documented about the firm, (iii) what this guide concludes. Every idea title and author is traceable to a cached part text; every verification date is this pass unless stated otherwise; quality is marked ✅ verified / ⚠️ flagged / ❌ rejected.

### 14.1 What the edition says

| # | Claim | Source | Date | Quality |
| --- | --- | --- | --- | --- |
| 1 | The 2026 edition is three newsletter parts, not an omnibus | cached parts 1–3; live pages re-extracted | Posted Dec 9 / 10 / 11, 2025 | ✅ verified |
| 2 | Page byline is "a16z New Media" on all three parts | cached parts 1–3 | Dec 2025 | ✅ verified |
| 3 | Part 1 contains 14 ideas across Infrastructure (6), Growth (4), Bio + Health (1), Speedrun (3) | cached part 1 | Dec 9, 2025 | ✅ verified by title+author+bio audit |
| 4 | Part 2 contains 16 ideas across American Dynamism (6) and Apps (10) | cached part 2 | Dec 10, 2025 | ✅ verified (§3.8) |
| 5 | Part 3 contains 17 ideas with no vertical headings | cached part 3 | Dec 11, 2025 | ✅ verified |
| 6 | Total: 47 ideas, 47 distinct bylined contributors | all three cached parts | Dec 2025 | ✅ verified |
| 7 | Part 1's method sentence: "every December, we ask our investing teams to share one big idea they think tech builders will tackle in the year to come." | cached part 1 | Dec 9, 2025 | ✅ verified verbatim |
| 8 | Part 3's framing uses "observe", says "17 things", and adds "plus a few guest contributors" | cached part 3 | Dec 11, 2025 | ✅ verified verbatim |
| 9 | Part 3 promises "a special announcement and an invitation from a16z" for the following day | cached part 3 | Dec 11, 2025 | ✅ verified verbatim |
| 10 | Jennifer Li's entry claims "80% of corporate knowledge" lives in unstructured data | cached part 1 | Dec 9, 2025 | ⚠️ flagged — the figure is uncited in the entry; this guide does not endorse or dispute the number, only records that no source is given |
| 11 | Joel de la Garza's entry claims unfilled cybersecurity jobs grew "from under 1M to 3M" over 2013–2021 | cached part 1 | Dec 9, 2025 | ⚠️ flagged — dated statistic with no source line in the entry |
| 12 | Angela Strange's entry claims new category winners will be "10x the size of the older incumbents" | cached part 2 | Dec 10, 2025 | ⚠️ flagged — uncited market-size multiplier |
| 13 | Andrew Hall's entry names the Zelensky suit market and the Venezuelan election market as disputed-resolution cases, and Prophet Arena as an example project | cached part 3 | Dec 11, 2025 | ⚠️ flagged — named in the entry; not independently checked this pass, because `web_search` was empty and the named markets were not extracted |
| 14 | Andrew Hall's entry contains the literal placeholder `<SC will link to our article here>` | cached part 3 | Dec 11, 2025 | ✅ verified — a defect present in the published text |
| 15 | Jeremy Zhang's entry claims stablecoins did "46 trillion dollars" in transaction volume "last year", "more than 20x the volume of PayPal; close to 3x the volume of Visa" | cached part 3 | Dec 11, 2025 | ⚠️ flagged — figures attributed to the entry; not independently verified this pass |
| 16 | Sean Neville's entry claims non-human identities outnumber human financial-services employees "96-to-1" | cached part 3 | Dec 11, 2025 | ⚠️ flagged — uncited in the entry |
| 17 | Jason Cui's entry contains an internal list headed "A few ideas we're excited by:" with three items | cached part 1 | Dec 9, 2025 | ✅ verified verbatim |
| 18 | David Ulevitch's entry ends "If that's you, let's talk." | cached part 2 | Dec 10, 2025 | ✅ verified verbatim |
| 19 | Malika Aubakirova's bio is in the past tense: "was an investor on the AI Infrastructure team at a16z" | cached part 1 | Dec 9, 2025 | ✅ verified verbatim; no current role asserted by this guide |
| 20 | Every one of the 47 entries is individually bylined | cached parts 1–3 | Dec 2025 | ✅ verified |

### 14.2 What is documented about the firm

| # | Claim | Source | Date | Quality |
| --- | --- | --- | --- | --- |
| 21 | `a16z.com/big-ideas-in-tech-2026/` returns "Page not found \| Andreessen Horowitz" | live extraction, this pass | this pass | ✅ verified |
| 22 | `a16z.com/newsletter/big-ideas-2026-part-{1,2,3}/` all exist with the stated dates and byline | live extraction, this pass | this pass | ✅ verified |
| 23 | The 2023 omnibus exists, byline "a16z editorial", posted December 15, 2022, and contains the "floodgates open to a deluge of best-of lists… more interested in what's to come" framing | cached 2023 page | Dec 15, 2022 | ✅ verified verbatim |
| 24 | The 2024 page exists and frames itself as "We asked over 40 partners across a16z…" | cached 2024 page | 2024 edition | ✅ verified verbatim |
| 25 | The 2025 page exists and frames itself as "We asked 50 a16z partners…", with the teaser naming eight verticals including Games and Consumer Tech | cached 2025 page | 2025 edition | ✅ verified verbatim |
| 26 | The 2025 editorial content contains "including Pennsylvania's once-infamous Three Mile Island, slated to come back online in 2028" | cached 2025 page | 2025 edition | ✅ verified verbatim |
| 27 | Three Mile Island Unit 1: Constellation Energy announced a 2028 reopening on September 20, 2024; renamed the Christopher M. Crane Clean Energy Center; Microsoft signed a 20-year power purchase agreement; $1.6bn upgrade investment | Wikipedia, "Three Mile Island Nuclear Generating Station", retrieved this pass | announcement dated Sep 20, 2024; page retrieved Sep 2026 | ✅ verified — used only as a check on a claim, not as a claim about a16z |
| 28 | The firm's disclaimer footer carries "Any investments or portfolio companies mentioned… are not representative of all investments" and "as for its investments in any cryptocurrency or token project, a16z is acting in its own financial interest, not necessarily in the interests of other token holders" | cached 2026 parts and cached 2024/2025/2023 pages (per §9.1 table) | as published | ✅ verified verbatim |
| 29 | The 2026 pages carry "Investing in …" cards disclosing investments, including cards bylined by the same contributors whose ideas appear on the same page | cached parts 1–3; each card string grepped | Dec 2025 | ✅ verified (specific cards listed in §9.2) |
| 30 | The 2024 edition's computer-vision idea names Flock Safety and Ambient; both names appear on a16z's public investment list | cached 2024 page; cached investment list | 2024 edition; list retrieved this pass | ✅ verified by grep |
| 31 | The Part 2 cards link a podcast, "Big Ideas 2026: Physical AI and the Industrial Stack", and a "The Machine Age Fund" page | cached part 2 | Dec 10, 2025 | ✅ verified |
| 32 | Every part carries a "You can just read 25 books" card bylined "a16z New Media" | cached parts 1–3 | Dec 2025 | ✅ verified |
| 33 | Part 3's cards reference an a16zcrypto annual series, "A Few of the Things We're Excited About in Crypto (2024)", and a "Watch: 11 Ideas for AI x Crypto" item | cached part 3 | Dec 11, 2025 | ✅ verified |
| 34 | a16z's public investment list contains, among others, Exa, Keycard, Highstock, Neo, Vals, Runta, Inferact, Netris, Glif, Town, Hedra, World Labs, Kalshi, Dynamic Labs, Flock, Ambient.ai, Abridge, Databricks and Cursor | cached investment list, each name grepped individually | list retrieved this pass | ✅ verified — presence on the list only; no claim about any position's size, date or terms |
| 35 | The five non-entry cards ("The Cinderella 'Glass Slipper' Effect", "Governance FAQs", "AI Micro Dramas…", "The Palantirization of everything", "The Incumbents Are Coming") each appear as a title line plus a name line with no body and no bio block | cached parts 1–3, each pair grepped | Dec 2025 | ✅ verified |
| 36 | The firm's investment list is cited in its own disclaimer as `https://a16z.com/investments/` | cached footers | as published | ✅ verified |

### 14.3 What this guide concludes

| # | Conclusion | Basis | Quality |
| --- | --- | --- | --- |
| 37 | The 2026 edition is best read as a position, not a forecast | §5, §9, §11 collectively | ✅ argued from sources; the conclusion is analytical, not a fact claim |
| 38 | No hit-rate for this class of artefact can be honestly computed | §7.1 four independent reasons; §11.6 corroborates from the other direction | ✅ the finding |
| 39 | No independent audit of these lists exists, and no edition contains a retro section | search of all four editions' cached text; absence of any third-party assessor in the corpus | ✅ verified absence within the available corpus — see §15 for the limit |
| 40 | The list is organised the way the firm is organised, so coverage tracks headcount | §4.1; bios naming named practice leaders | ✅ argued from sources |
| 41 | "Apps" and "Speedrun" are new 2026 labels; "Consumer Tech", "Games" and "Enterprise + Fintech" are absent as labels | §6.2, comparing cached vertical headings | ✅ verified |
| 42 | Part 3's framing differs epistemically from Parts 1–2 and is unreconciled | §2.3, §6.4, both quoted | ✅ verified verbatim |
| 43 | Roughly a third of the 2026 ideas are directly decision-relevant to an enterprise technology function; nothing in the edition is a procurement recommendation | §8 | ⚠️ flagged — a judgement call, not a measurement; the classification in §8.2 is this guide's and is offered as such |
| 44 | The edition carries five-card promotional furniture, investment-announcement cards and newsletter prompts throughout | §9.2, §9.4, §12.1 | ✅ verified strings |
| 45 | This guide asserts no motive, alleges no bad faith, and attributes no intent to any named person | entire document | ✅ by construction — every §9 item is either a quotation or a grepped string |

### 14.4 What was rejected

| # | Claim encountered | Disposition | Reason |
| --- | --- | --- | --- |
| 47 | "a16z published a 2026 omnibus like the 2023–2025 editions" | ❌ rejected | The omnibus URL 404s; the edition is a three-part series |
| 48 | "The 2026 edition has more than 47 ideas" | ❌ rejected | Would require counting promotional/related-content cards as entries; the §3.8 audit produces exactly 47 |
| 49 | "The Cinderella 'Glass Slipper' Effect" / "Governance FAQs" / "The Palantirization of everything" / "The Incumbents Are Coming" / "AI Micro Dramas, Generative Media, and the Future of Creativity" are Big Ideas 2026 entries | ❌ rejected (all five) | Each appears as a title line plus a name line only — Malika Aubakirova, Andrew Hall, Marc Andrusko, Seema Amble and Justine Moore respectively — with no body essay and no "About the Contributor" block |
| 50 | Any specific hit-rate or percentage-correct figure for any edition | ❌ rejected as unsupportable | §7.1; no audit, no failure conditions, no attribution model |
| 51 | "a16z predicted nuclear power's resurgence and it happened" | ❌ rejected as stated | The relevant fact (Three Mile Island's restart announcement) was public before the page was published (Sep 20, 2024 vs December 2024), so it is a description, not a successful forecast — §7.2 |
| 52 | "a16z's ideas cause the investments they name" | ❌ rejected | No evidence available; this guide asserts no causality in either direction |
| 53 | Any statement about a competing firm's equivalent annual list | ❌ rejected | `web_search` returned empty result sets this pass; nothing verified, so nothing asserted |

---

## 15. What Could Not Be Verified

This section exists to be read. Everything here is a limit of this pass, not a finding about the subject.

### 15.1 Tool limitations, stated specifically

| Limitation | What it means for this guide |
| --- | --- |
| **`web_search` returned EMPTY result sets on every query attempted in this pass** | No search-engine-sourced reporting, commentary, analyst note, criticism or rebuttal of the 2026 edition was available. Every claim in this guide therefore rests on the publisher's own pages, the firm's own investment list, or one named general reference. No claim here has been cross-checked against independent journalism |
| **`web_extract` DID work against a16z.com this pass** | The 404 on `a16z.com/big-ideas-in-tech-2026/` and the existence and dates of the three newsletter parts were verified directly, and clean page text was cached. The format-change finding is therefore independently verified, not merely inherited |
| **The primary source was obtained as flattened page text** | Link targets inside the essays are not preserved. Where an essay appears to reference an external source (Jeremy Zhang's volume figures, Andrew Hall's disputed markets, Joel de la Garza's statistic), the guide records the number as the entry states it and marks it ⚠️ rather than pretending to have followed a link it does not hold |
| **The cached texts were captured at different times and are not frozen** | The live 2026 pages now render a "Table of Contents" navigation block absent from the working copies. Footer wording differs by edition and capture format (§9.1). The idea inventory itself was verified against both the flattened text and the live-extracted page and is stable |
| **No independent body audits these lists, and none was found** | Searching the available corpus for an auditor, a scoring rubric or a post-mortem produced nothing. **This is an absence of evidence within the available corpus, not evidence of absence** — a third-party assessment may exist that this pass could not reach |
| **The publisher's follow-up items were not followed** | Part 3 promises "a special announcement and an invitation from a16z" for December 12, 2025. This guide did not verify what that announcement was, because the pass had no search capability with which to locate it and did not hold its URL |
| **No competitor artefacts verified** | §12 names none, because none could be confirmed. That is a statement about this pass's reach, not about the market |

### 15.2 Specific items left unverified

| Item | Status | What would settle it |
| --- | --- | --- |
| The "80% of corporate knowledge" figure (Jennifer Li) | ⚠️ unverified — no source given in the entry | The originating study, if one exists |
| The 2013–2021 cybersecurity unfilled-jobs trend (Joel de la Garza) | ⚠️ unverified — no source given in the entry | The industry survey the figure derives from, and its definition of "unfilled" |
| The "10x the size of the older incumbents" claim (Angela Strange) | ⚠️ unverified — no source given in the entry | A category-sizing analysis, which the entry does not provide |
| The stablecoin volume figures (Jeremy Zhang) | ⚠️ unverified — no source given in the entry | The volume dataset and its methodology |
| The "96-to-1" non-human identity ratio (Sean Neville) | ⚠️ unverified — no source given in the entry | The survey or registry the ratio derives from |
| The disputed prediction-market cases (Andrew Hall) | ⚠️ unverified this pass — named in the entry | Reporting on the specific resolution disputes |
| Andrew Hall's cited project "Prophet Arena" | ⚠️ unverified this pass — named in the entry | Direct inspection of the project |
| The recurrence of the token-conflict sentence in the **2023** footer | ⚠️ not found in the cached 2023 text (found in 2024, 2025 and 2026) | Reading the live 2023 page's footer — see §9.1 for why this is likely a capture artefact rather than a real difference |
| The content of a16z's December 12, 2025 announcement | ⚠️ unverified | The follow-up page |
| Whether any third party has assessed any edition's accuracy | ⚠️ none found in the available corpus | Any audit, if it exists |
| Whether any other firm publishes a comparable annual list | ⚠️ unverified | A working search capability |
| Which of the 2026 ideas came true | ⚠️ **not assessed, and not assessable** | A pre-registered rubric written before the year began (§7.4). The 2026 calendar year is not complete at the time of writing (September 2026), so no idea with a 2026 horizon can yet be graded even in principle |

### 15.3 The one thing this section refuses to do

It refuses to convert any "not verified" into "false", and any "absent from the sources" into "absent from the world". A guide that treats its own reach as the world's boundary is worse than useless — it manufactures confidence out of search gaps. Every ⚠️ above is an open question with a stated route to closure, and the reader is invited to close them.

---

## 16. The Glossary, the Cross-References and the Closing Summary

### 16.1 Glossary — the decoder terms

| Term | Definition as used in this guide |
| --- | --- |
| **Omnibus edition** | The 2023–2025 format: one a16z article page containing every vertical's idea for that year, one byline, one date |
| **Newsletter series** | The 2026 format: three separate newsletter pages published on consecutive days, comprising the year's edition, with no single containing page |
| **Vertical** | A heading in the edition corresponding to one of the firm's practice groups (Infrastructure, Growth, Bio + Health, Speedrun, American Dynamism, Apps, crypto) — an org unit, not a market taxonomy |
| **Contributor byline** | The individual named under an idea title; distinct from the page byline ("a16z New Media") |
| **"Big idea" as a unit of content** | One contributor, one title, one short essay, one bio block — an editorial bound, not an analytical one. Jason Cui's entry contains three subordinate ideas inside a single unit |
| **Portfolio** | The companies a16z funds have invested in; published by the firm at `a16z.com/investments/` |
| **Anti-portfolio** | Deals the firm lost or passed on; never published in these editions, and its absence is a structural feature of the genre |
| **Fund** | A pooled vehicle managed by the firm, raised from LPs and deployed into portfolio companies |
| **LP** | A limited partner committed to a fund; a secondary audience for a published thesis statement |
| **Thesis** | The claim a contributor makes about what builders will tackle, or what partners observe, in the year ahead |
| **Position** | The firm's actual holdings and commitments, as distinguishable from its published claims — the distinction this guide's title rests on |
| **Demand-side inventory** | The reading of the list as a forecast of what will be *sold* into enterprises, not of what is *true* (§8.1) |
| **Mechanism** | The causal story inside an idea (e.g. "agent fan-out breaks 1:1 concurrency assumptions") — more durable and more testable than the prediction it supports (§10.3) |
| **Pre-registered rubric** | A scoring rule written *before* the year begins, so outcomes can be graded without hindsight bias; the missing ingredient in all four editions (§7.4) |

### 16.2 Glossary — terms the ideas themselves use

Only terms the edition's text actually uses are defined here.

| Term | Use in the edition | Plain reading |
| --- | --- | --- |
| **RAG** | Used by Jennifer Li ("causes RAG systems to hallucinate") | Retrieval-augmented generation — a pattern where a model answers using documents fetched at query time. The entry's point is that retrieval quality is capped by the quality of the retrieved data |
| **Data entropy** | Coined/used by Jennifer Li ("the steady decay of freshness, structured, and truth inside the unstructured universe") | The ongoing degradation of usable structure and accuracy in an organisation's unstructured corpus. An evocative term; not a measured quantity |
| **Agent-native** | Used by Malika Aubakirova ("Agent-native infrastructure becomes table stakes") | Infrastructure designed on the assumption that machine agents, not humans, are the primary callers — recursive, bursty, high-concurrency workload shapes |
| **Agent-speed / "the context problem" / data-in-action** | Used by Malika Aubakirova and Jason Cui | "Agent-speed" workload is characterised by recursive fan-out (Aubakirova's example: a single goal triggering thousands of calls), as distinct from human-speed traffic. The "context problem" is Cui's name for the difficulty of continuously supplying agents the right data context and semantic definitions from multiple systems of record |
| **World model** | Used by Jonathan Lai | A model that generates and maintains a persistent 3D environment — the entry names Marble (World Labs) and Genie 3 (DeepMind) as examples |
| **Electro-industrial stack** | Used by Ryan McEntush ("the rise of the electro-industrial stack") | The convergence layer of electrification, materials and AI-driven control inside industrial machines |
| **MAU / DAU / YAU** | Used by Julie Yoo ("healthy MAUs", "sick MAUs", "sick DAUs", "healthy YAUs") | Monthly / daily / yearly active users. The entry repurposes marketing metrics as healthcare customer segments |
| **Forward-deployed** | Used by Joe Schmidt ("Forward-deployed motions take AI to the 99%") | An engineering and sales model where vendor staff embed with the customer; borrowed from the defence and consulting lexicon |
| **Prediction market / LLM oracle** | Used by Andrew Hall | A market whose contracts pay out on the outcome of a future event, so prices act as probability estimates. An LLM oracle is a model used to determine whether a contract's condition was met — i.e. to resolve truth, not to trade |
| **SNARK / zk / zkVM** | Used by Justin Thaler ("SNARKs — cryptographic proofs that let you verify computation without re-executing it"; "zkVM provers will hit roughly…") | Succinct non-interactive argument of knowledge: a short proof that a computation was performed correctly, verifiable far faster than re-running it. A zkVM is a virtual machine that emits zero-knowledge proofs of its own execution; Thaler's claim is about falling proving overhead |
| **Stablecoin** | Used by Jeremy Zhang, Sam Broner, Maggie Hsu, Christian Crowley, Miles Jennings | A token designed to hold a stable value, typically pegged to a fiat currency; the edition treats it as a settlement layer |
| **Tokenization / perps (perpetual futures)** | Used by Guy Wuollet, Adeniyi Abiodun, Maggie Hsu | Representing an asset as a token on a blockchain. Wuollet distinguishes "skeuomorphic" tokenisation (wrapping the old asset concept) from crypto-native instruments, naming perpetual futures — contracts with no expiry, funded periodically |
| **KYA ("know your agent")** | Coined/used by Sean Neville | Extending KYC to non-human actors: cryptographically signed credentials binding an agent to its principal, constraints and liability |
| **Bridging** | Used by Ali Yahya ("Bridging tokens is easy, bridging secrets is hard") | Moving assets between blockchains. The entry's argument is that privacy breaks the fungibility bridging depends on |
| **GENIUS / credible neutrality / composability** | Referenced by Miles Jennings ("After GENIUS, the proliferation of stablecoins has exploded"; "open, autonomous, composable, credibly neutral, and decentralized") | GENIUS is the US stablecoin legislation the entry treats as a precedent for a market-structure bill. The trailing properties are what Jennings hopes legal clarity yields: no privileged operator, verifiable behaviour, and the ability for other systems to build on a network without permission |

Terms deliberately **not** defined because the edition does not use them: any term this guide could only define from general knowledge rather than from the source. Where the edition uses a term loosely, the glossary says so rather than supplying a technical definition the publisher did not intend.

### 16.3 Cross-references

Within this repository, by filename (all confirmed present at the time of writing):

| Guide | Owns |
| --- | --- |
| [ai_native_companies_guide.md](ai_native_companies_guide.md) | The AI-native-company concept; **§2.2 owns the coinage and provenance of "AI-native"** and already notes that "a16z, Sequoia, and Y Combinator all used AI-native framing in this window", flagging first use as unverified. This guide cross-references that thread and does not re-derive it |
| [ai_adoption_strategies_guide.md](ai_llm/ai_adoption_strategies_guide.md) · [enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) · [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) | The enterprise AI strategy-and-platform discipline: adoption maturity, phases and organisational models; enterprise AI platform architecture; platform engineering for AI systems |
| [agentic_workflows_guide.md](agentic_workflows_guide.md) · [llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md) | Agent execution, orchestration and workflow patterns; evaluation methodology — the discipline that turns a thesis into a test |
| [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) | The agent-era enterprise security surface (the repo's agent-security guide; `agent_security_guide.md` was checked for and does not exist, hence this filename) |
| [finops_guide.md](finops_guide.md) | Inference and cloud unit economics |
| [capacity_sizing_guide.md](capacity_sizing_guide.md) | Capacity planning and sizing — the discipline §8.4 points at |
| [data_fabric_guide.md](data/data_fabric_guide.md) | The data-platform substrate the Infrastructure ideas presuppose |

### 16.4 Closing summary

Ten things this guide established, in the order they matter:

1. **The 2026 edition is a three-part newsletter series**, published **December 9, 10 and 11, 2025**, bylined "a16z New Media", with **47 ideas in total (14 + 16 + 17)** across seven team labels. There is no omnibus page: `a16z.com/big-ideas-in-tech-2026/` 404s, verified directly this pass.
2. **Every idea is individually bylined, and the verticals are the firm's own practice groups.** The list is organised the way the firm is organised, so coverage tracks headcount and label churn is at least as large as topic churn ("Apps" and "Speedrun" in; "Games", "Consumer Tech" and "Enterprise + Fintech" out as labels).
3. **The edition contains three framings and does not reconcile them.** Part 1 restates the 2023–2025 method ("every December we ask our investing teams"). Part 3 switches register entirely: "17 things… **observe**… (plus a few guest contributors)". That unreconciled seam is the edition's most interesting editorial fact.
4. **The list is legible as four things at once** — an investment-thesis statement, a deal-flow instrument (Ulevitch's "If that's you, let's talk"), a brand and LP-communication product, and a disclosure of positions already held. The publisher itself described the genre in 2022: "we at a16z have always been more interested in what's to come."
5. **A reader must distinguish entries from cards.** Title + bylined author + body essay + "About the Contributor" bio = an entry; title + name alone = a promotional or related-content card. Applying that rule produces exactly 47 and excludes five plausible-looking card titles.
6. **The accountability question has no empirical answer, and saying so is the answer.** No independent body audits these lists, no edition contains a retro section, and a multi-year trend prediction is not falsifiable on a one-year horizon. **This guide computes no hit-rate and states that any published hit-rate for this class of artefact is unsupported.**
7. **Where checking was possible, the check cut both ways.** Of five sampled ideas across 2023, 2024 and 2025: two are too broad to be wrong, one is genuinely still open, one is documented only as a fact about the documents, and one is documented in the world — and in that single case (Three Mile Island) the documented fact was public before the page that "predicted" it was published.
8. **The conflict is disclosed, by the firm, in its own words, on every page.** "Any investments or portfolio companies mentioned… are not representative of all investments"; "as for its investments in any cryptocurrency or token project, a16z is acting in its own financial interest." The same pages carry "Investing in …" cards in the same categories as the ideas, sometimes bylined by the same contributors. This guide reports those facts, alleges no motive, and names no intent.
9. **For an enterprise technology function the artefact is a demand-side inventory** — a good map of what will be *sold*, a poor guide to what should be *bought*. Roughly a third of the 47 ideas are directly decision-relevant; none is a procurement recommendation. Harvest the questions; the answers live in the evaluation, capacity, security and unit-economics disciplines this repository documents separately.
10. **The practical method is five uses, compressed into four steps**: read it as demand-side inventory and diff it against the prior edition; extract the mechanisms rather than the predictions; cross-check named companies against the firm's public investment list; and diary a re-read twelve months out **with the rubric written now**. A rubric written after the fact is how prediction lists acquire imaginary hit-rates.

A venture firm's prediction list is not a forecast; it is a position — which does not make it useless, it makes it something specific, and reading it as what it is, is the whole job. **a prediction is a position.**
