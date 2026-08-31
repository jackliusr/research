# Product Management Frameworks: A Verified Library for Discovery, Prioritisation, and Strategy — A Solution Architect's Guide

> **Author:** Jack Liu Shurui, Solution Architect — Cymbal Bank, Singapore
> **Context:** the product-management deep-dive of the management series — how a solution architect selects and runs the product-management toolkit: discovery (Jobs-to-be-Done, the Value Proposition Canvas, the Lean Canvas, the problem interview), prioritisation (RICE, Kano, ICE, the effort/impact matrix, MoSCoW), strategy and metrics (the North Star metric, OKRs, HEART, AARRR), and product-market fit (the Sean Ellis test). It cross-references [Workshop Methodology](../management/workshop_methodology_guide.md) (facilitation mechanics, the double diamond, MoSCoW, and the Cymbal Bank worked-example convention), [Conflict Management](../management/conflict_management_guide.md) (the house format for claims audits and honest gaps), [E-Commerce Experience](../management/ecommerce_experience_guide.md) (AARRR), [3D Managerial Effectiveness](../management/3d_managerial_effectiveness_guide.md) and the leadership guides (OKRs and strategy), [MBA Body of Knowledge](../management/mba_body_of_knowledge_guide.md) (Porter/strategy), and the banking series [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) and [DBS Bank](../banking/dbs_bank_guide.md) (the regulatory and banking-product angle). It does *not* re-derive MoSCoW, the double diamond, AARRR, or OKR mechanics.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass ran on a pre-verified research bundle (primary-source captures compiled with live web access: the MIT Sloan Management Review full text of "Finding the Right Job for Your Product" (Christensen, Anthony, Berstell, Nitterhouse, 2007 — the canonical home of the milkshake article; the HBR 2007 URL 404s ✅); HBR's "Marketing Malpractice: How to Fix It" (Christensen, Cook, Hall, December 2005 ✅) and "Know Your Customers' Jobs to Be Done" (September 2016 ✅); the Internet Archive catalog record of *Competing Against Luck* (Christensen, Hall, Dillon, Duncan, HarperBusiness, 2016 ✅); Strategyzer's own Value Proposition Canvas page ✅; Ash Maurya's own "Why Lean Canvas versus Business Model Canvas" ✅; the Wikipedia record of the Kano model with the full 1984 citation (Kano, Seraku, Takahashi, Tsuji, "Attractive Quality and Must-be Quality", April 1984, *Hinshitsu* 14(2):39–48, 12th Annual Meeting of the JSQC ✅); Sean Ellis's own "The Startup Pyramid" plus its 2009 Wayback captures (the 40% "very disappointed" product-market-fit rule ✅); Janna Bastow's own account of inventing Now-Next-Later and the ProdPad FAQ ✅; and the Wayback capture of the original Intercom RICE post (Sean McBride, 2016, "last August [2015] we began developing" ✅). HEART is verified at Google Research (Rodden, Hutchinson, Fu, CHI 2010 ✅). Anything the bundle could not settle is flagged ⚠ in [The Claims Audit](#13-the-claims-audit) and [What Could Not Be Verified](#14-what-could-not-be-verified). Nothing in this guide is fabricated; unverified items are marked, not asserted.
> **Last Updated:** August 2026

---

### How to Use This Guide

**What this guide is.** A framework library for product work: what each framework is, its concise mechanics, its *verified* origin with source, when to use it, and its limitations — organised into the four families every product team works in: discovery, prioritisation, strategy and metrics, and product-market fit. It closes with a selection guide, a banking-product angle, a Cymbal Bank worked example, and the house claims audit.

**How to use it.** Read §1 first — the landscape and the verification discipline every later section assumes. §2–§3 are discovery; §4–§6 are prioritisation; §7–§8 are strategy and metrics; §9 is product-market fit. §10 is the selection guide (stage × question × framework); §11 is the banking angle; §12 is the worked example; §13 is the claims audit; §14 the honest gaps; §15 the glossary. Sections that fully derive a framework carry a source line; sections that merely apply one (MoSCoW, the double diamond, AARRR, OKRs) cross-reference their home guides and do not re-derive them.

**What this guide is not.** It is not a workshop-facilitation manual (that lives in `workshop_methodology_guide.md` — problem-interview and discovery-session mechanics are cross-referenced, not re-derived), not a strategy textbook (that lives in `mba_body_of_knowledge_guide.md`), and not a banking-regulations manual (that lives in `mas_regulations_guidelines_guide.md`). It is also not a catalogue of every PM framework that exists; it is the curated, origin-verified set a solution architect actually reaches for.

**The one-sentence thesis:** *a framework is a tool for making a decision legible — and the discipline is not in the framework you pick but in knowing whose idea it was, what it assumes, and when it is the wrong tool for the question.*

---

## Table of Contents

1. [The PM-Framework Library: The Landscape](#1-the-pm-framework-library-the-landscape)
2. [Discovery: Jobs-to-be-Done](#2-discovery-jobs-to-be-done)
3. [Discovery: The Value Proposition Canvas, the Lean Canvas, and the Problem Interview](#3-discovery-the-value-proposition-canvas-the-lean-canvas-and-the-problem-interview)
4. [Prioritisation: RICE](#4-prioritisation-rice)
5. [Prioritisation: The Kano Model](#5-prioritisation-the-kano-model)
6. [Prioritisation: ICE, the Effort/Impact Matrix, and MoSCoW](#6-prioritisation-ice-the-effortimpact-matrix-and-moscow)
7. [Strategy and Metrics: The North Star Metric and OKRs](#7-strategy-and-metrics-the-north-star-metric-and-okrs)
8. [Strategy and Metrics: HEART and AARRR](#8-strategy-and-metrics-heart-and-aarrr)
9. [Product-Market Fit: The Sean Ellis Test](#9-product-market-fit-the-sean-ellis-test)
10. [The Framework Selection Guide](#10-the-framework-selection-guide)
11. [The Banking-Product Angle](#11-the-banking-product-angle)
12. [The Worked Example: Cymbal Bank](#12-the-worked-example-cymbal-bank)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [The Glossary](#15-the-glossary)

---

## 1. The PM-Framework Library: The Landscape

### 1.1 What a product-management framework is

A product-management framework is a named, repeatable way of answering one of the four questions every product team keeps asking:

| Family | The question it answers | Frameworks in this guide |
|---|---|---|
| **Discovery** | What should we build, and for whom? | Jobs-to-be-Done (§2), Value Proposition Canvas, Lean Canvas, problem interview (§3) |
| **Prioritisation** | What do we build first, and why? | RICE (§4), Kano (§5), ICE, effort/impact matrix, MoSCoW (§6) |
| **Strategy & metrics** | How do we know it is working? | North Star metric, OKRs (§7), HEART, AARRR (§8) |
| **Product-market fit** | Have we built something people must have? | The Sean Ellis test (§9) |

The frameworks are deliberately heterogeneous: some are research-derived theories (Kano, HEART), some are consulting practice that became public doctrine (JTBD, the canvases), some are working tools published by the teams that invented them (RICE, Now-Next-Later). They are not interchangeable; they answer different questions at different stages. The selection guide in §10 maps stage × question × framework.

### 1.2 The verification discipline

The house rule of this repository: never present an unverified claim as fact. Every load-bearing origin claim in this guide carries a status — ✅ (verified this pass at a primary or authoritative source, source given), ⚠ (single-source, held from the author's knowledge base, or not re-verified), ❌ (disputed or false as stated). Two consequences follow:

- **Origins matter, so they are checked.** "Who coined this?" is not trivia: it tells you the context the framework was built for. Kano's model was built for quality management in 1984; RICE was built for a SaaS product team in 2015–2016; the Sean Ellis test was built for startup growth around 2009. Using a framework outside its home context is the most common misuse.
- **Unverifiable claims are flagged, not asserted.** Where this pass could not pin an origin to a primary source — the exact coinage date of the ICE score, the exact authorship of the Amplitude North Star Playbook, who first wrote "Jobs-to-be-Done" as a phrase — the guide says so, in the section, in the audit (§13), and in the honesty section (§14).

### 1.3 The honest origin story at a glance

The table below is the one-line version of the audit; each row is argued in its home section.

| Framework | Attributed origin | Status | Home |
|---|---|---|---|
| Jobs-to-be-Done | Christensen lineage: *The Innovator's Solution* (2003), HBR "Marketing Malpractice" (2005), MIT Sloan "Finding the Right Job for Your Product" (2007), *Competing Against Luck* (2016); parallel Tony Ulwick outcome-driven-innovation lineage; Moesta/Spiek popularisation | ✅/⚠ mixed (see §2.3) | §2 |
| Value Proposition Canvas | Osterwalder & Pigneur, from *Business Model Generation* (2010), formalised in *Value Proposition Design* (2014) | ✅ | §3 |
| Lean Canvas | Ash Maurya, adaptation of the Business Model Canvas | ✅ | §3 |
| RICE | Intercom, Sean McBride, 2016 | ✅ | §4 |
| Kano model | Kano, Seraku, Takahashi, Tsuji, 1984 | ✅ | §5 |
| ICE score | Sean Ellis / GrowthHackers | ⚠ | §6 |
| Effort/impact matrix | General practice; no single verifiable origin | ⚠ (honestly: none) | §6 |
| MoSCoW | DSDM lineage | ✅ (cross-ref) | §6 |
| North Star metric | Sean Ellis / GrowthHackers practice; Amplitude North Star Playbook (2018) | ⚠ | §7 |
| OKRs | John Doerr, from Andy Grove's MBO practice | ✅ (cross-ref) | §7 |
| HEART | Google Research: Rodden, Hutchinson, Fu, CHI 2010 | ✅ | §8 |
| AARRR | Dave McClure (2007) | ⚠ (cross-ref) | §8 |
| Sean Ellis test | Sean Ellis, "The Startup Pyramid", startup-marketing.com, ~July 2009 | ✅ | §9 |
| Now-Next-Later roadmap | Janna Bastow (ProdPad) | ✅ | §12 |

### 1.4 How to read a framework

Every framework in this guide can be read in four layers, and the craft is matching the layer to the situation:

1. **The claim it makes about the world** — Kano claims features have different satisfaction dynamics; JTBD claims customers hire products for jobs; RICE claims four factors capture a project's value per unit of time.
2. **The instrument it provides** — a survey (Kano, Sean Ellis), a score (RICE, ICE), a canvas (VPC, Lean Canvas), a roadmap format (Now-Next-Later).
3. **The context it was built in** — Kano in 1984 Japanese quality management; RICE in a 2015–2016 SaaS product team; the Sean Ellis test in 2009 startup growth. The context tells you what the framework's designers were optimising for, and where it will strain.
4. **The failure mode it assumes** — each framework exists because its designers saw a specific failure: RICE assumes teams cannot compare hard-to-compare ideas; Kano assumes teams spend on delighters while must-be's are missing; Now-Next-Later assumes teams promise dates they cannot keep.

| Framework | The claim | The instrument | Built for | The failure it assumes |
|---|---|---|---|---|
| JTBD | Customers hire products for jobs | Interview method, situation cases | New-product discovery | Feature- and demographic-driven ideation |
| Value Proposition Canvas | Value must be built from the customer side | Two-block canvas | Value-proposition design | Vague value propositions |
| Lean Canvas | Startups fail on assumptions, not plans | Nine-box canvas | Early-stage risk surfacing | Business-model optimisation before validation |
| RICE | Four factors capture value per time worked | Score | Backlog triage | Incomparable, indefensible priorities |
| Kano | Features have different satisfaction dynamics | Two-question survey | Feature categorisation | Treating all features as linear satisfiers |
| ICE | Experiments can be ranked fast | Score | Experiment backlogs | Slow, heavy prioritisation of cheap tests |
| Sean Ellis test | Fit is measurable | One-question survey | The scale decision | Scaling before fit |
| Now-Next-Later | Confidence, not dates, is the honest axis | Roadmap format | Roadmap communication | Broken date promises |

The layer most often misread is the third: a framework's home context is the strongest hint about where it is *not* at home — which is why §11 spends so much effort mapping the banking boundary.

---

## 2. Discovery: Jobs-to-be-Done

### 2.1 What it is

Jobs-to-be-Done (JTBD) is the proposition that customers do not buy products for the products' sake — they "hire" products to get a *job* done in a given situation, and they "fire" them when the job changes or a better candidate appears. The unit of analysis is therefore not the customer (demographics, psychographics) and not the product (category, price), but the **job**: the fundamental problem a customer needs to resolve in a given situation, with functional, emotional, and social dimensions. In the canonical formulation of the MIT Sloan article: "Customers just find themselves needing to get things done. When customers find that they need to get a job done, they 'hire' products or services to do the job."

### 2.2 The mechanics

- **The job is the unit of analysis.** Segment markets by job, not by product category or customer profile. Job-defined markets are generally much larger than product-category markets, because the real competition is every other candidate for the job — including "boredom".
- **Hiring criteria.** Customers compare "job candidates" on the experiences required to do the job completely: functional (does it work?), emotional (how does it make me feel?), social (what does using it say about me?). The marketing mix — promotion, product, price, placement — falls out of the job once it is understood.
- **Finding the job.** The article's hierarchy: (1) your current customer base — they are usually hiring the product for different jobs than you intended; (2) customers buying competing products for the job; (3) nonconsumers constrained from good solutions by complexity or cost (the disruption seam). Methods: interviews and surveys (when the job is knowable), observation of compensating behaviours (when no product exists yet), empathic participation (when the situation is murky — Hill-Rom's researchers worked as hospital orderlies; P&G's Swiffer came from watching people use a dustpan as a workaround), and coevolution (when neither side can articulate the job — GM OnStar).
- **Synthesising.** Distil interactions into "situation cases" — the chronological trail of events and thoughts that led to purchase. About 25 situation cases constitute critical mass; group them by similarity of situation; for each group describe the job, its frequency, the competing job candidates, the hiring criteria, and the "help wanted" signs (deficiencies and constraints that block the job).

### 2.3 The honest lineage — who actually coined what

The bundle settled the paper trail; the public debate about the *phrase* is real and is presented honestly:

| Claim | Status | Evidence |
|---|---|---|
| The milkshake case appears in HBR's "Marketing Malpractice: How to Fix It" by Christensen, Cook & Hall (December 2005) | ✅ | HBR December 2005 issue verified this pass; the article is the other JTBD anchor besides the Sloan piece |
| "Finding the Right Job for Your Product" — the fuller treatment with the milkshake, IKEA, OnStar, and Hill-Rom cases — is an MIT Sloan Management Review article by Christensen, Anthony, Berstell & Nitterhouse (2007) | ✅ | Full text at sloanreview.mit.edu/article/finding-the-right-job-for-your-product/ (authors listed in the article's About the Authors); the often-cited HBR 2007 URL 404s — the Sloan piece is the real home |
| The milkshake example is *disguised*: footnote 1 states "The descriptions of the product and company in this example have been disguised" | ✅ | Sloan article, reference 1 — so any claim that it is "McDonald's" goes beyond the primary source |
| The term "jobs to be done" was popularised in Christensen & Raynor, *The Innovator's Solution* (Harvard Business School Press, 2003) | ⚠ | Held from the author's knowledge base; the book was not extracted this pass. Widely documented, but flagged for honesty |
| *Competing Against Luck: The Story of Innovation and Customer Choice* (Christensen, Hall, Dillon & Duncan, HarperBusiness, 2016) | ✅ | Internet Archive catalog record (archive.org/details/competingagainst0000chri): 2016, HarperBusiness |
| HBR, "Know Your Customers' Jobs to Be Done" (September 2016) | ✅ | hbr.org/2016/09/know-your-customers-jobs-to-be-done, extracted this pass |
| Bob Moesta and Chris Spiek (ReWired Group) popularised JTBD as an applied practice, working with Christensen on the milkshake research | ⚠ | Held from the author's knowledge base; the bundle did not capture a primary Moesta/Spiek source this pass |
| Parallel lineage: Tony Ulwick's Outcome-Driven Innovation (ODI) — the "jobs to be done" concept as a structured innovation method (Ulwick, *What Customers Want*, 2005; *Jobs to Be Done: Theory to Practice*, 2016); Ulwick has publicly claimed to have coined the phrase in the early 1990s | ⚠ | Held from the author's knowledge base; not re-verified at a primary source this pass |
| The who-coined-what debate | ⚠ | The honest summary: Christensen's HBS lineage made "jobs to be done" famous through the milkshake; Ulwick's ODI school claims the phrase and a systematic method; Moesta/Spiek made it an interview practice. The bundle verifies the *papers* (rows above) but cannot arbitrate the coinage dispute, which is contested between living practitioners |

The section's working position, stated plainly: **the concept is Christensenian in popular form; the phrase's coinage is disputed; the papers above are the verified anchors.**

### 2.4 When to use it — and its limitations

- **Use it** when defining a new product, repositioning an existing one, finding the real competition, or understanding why a product that "should" sell does not.
- **Limitations.** JTBD is a discovery lens, not a prioritisation formula — it tells you the job, not the business case. It is interview- and observation-intensive (the "situation case" method is slow and qualitative). It does not by itself size the market. And the term is now a marketing label attached to many methods that share little but the name — verify which lineage a consultant or tool actually implements. For the facilitation mechanics of running JTBD-style discovery sessions, cross-reference the [Workshop Methodology](../management/workshop_methodology_guide.md) guide's double-diamond discovery workshops rather than re-deriving them here.

### 2.5 The JTBD interview kit

The Sloan article's method is observation-plus-interview ("It requires watching, participating, writing and thinking"), and its signature move is asking about the *situation*, not the customer: "Excuse me, but could you please tell me what job you were needing to get done for yourself when you came here to hire that milkshake?" The practitioner-standard question kit that descends from this (⚠ — standard interview practice, not a verified citation) clusters into five stems:

| Question stem | What it is hunting for |
|---|---|
| "Tell me about the last time you [did the thing]…" | The situation, narrated as a story — the raw material of a situation case (§2.2) |
| "What were you doing right before that? What happened next?" | The chronological trail — the trigger and the follow-through |
| "What did you try before you hired this?" | The real job candidates — including the non-obvious ones (bagels, bananas, boredom) |
| "What almost stopped you? What was hard about it?" | The "help wanted" signs — constraints and deficiencies that block the job |
| "What would you do if this didn't exist?" | Compensating behaviours — the workaround reveals the job underneath |

The listening discipline is as important as the stems: record the *functional* needs but hunt for the *emotional and social* dimensions (the Sloan article: "Products don't engender emotions. Situations do."); never ask "would you buy…?" (future opinion) when you can ask "what did you do…?" (past behaviour); and watch for compensating behaviours — the article's core discovery moves (P&G's dustpan users and the Swiffer; the 12-Minute Games inventor) all came from workarounds, not from stated preferences.

---

## 3. Discovery: The Value Proposition Canvas, the Lean Canvas, and the Problem Interview

### 3.1 The Value Proposition Canvas

**What it is.** A one-page tool that forces the value proposition to be built from the customer side: two blocks — the **Customer Profile** (jobs the customer is trying to get done, the pains they face doing them, the gains they perceive from getting them done) and the **Value Map** (the products and services on offer, the pain relievers, the gain creators). Fit is achieved when the value map demonstrably relieves the pains and creates the gains that matter for the jobs in the profile. Strategyzer's own page frames the three uses: precisely define customer profiles, visualise the value you create, and adjust the value proposition on customer evidence to achieve product-market fit.

**Verified origin.** The Strategyzer page (strategyzer.com/library/the-value-proposition-canvas) — "This method from the bestselling innovation book *Value Proposition Design* is applied in leading organizations and start-ups worldwide" ✅. The lineage: Osterwalder & Pigneur's *Business Model Generation* (2010) introduced the Business Model Canvas (the parent tool); the Value Proposition Canvas was formalised in *Value Proposition Design* (2014) ✅ for the book's existence (via the Strategyzer page), ⚠ for the exact 2014 publication details (held from knowledge, not extracted).

**When to use it.** When you must design or rework a value proposition around evidence — typically alongside problem interviews, before committing to build. It is the discovery tool that makes "the customer" concrete enough to test.

**Limitations.** It is a structuring and communication tool; it does not generate the customer evidence itself — the evidence must come from interviews and observation (§3.3). The two blocks can be filled with wishful thinking faster than any other canvas, which is why the fit test (evidence per customer, not words per slide) is the whole game.

### 3.2 The Lean Canvas

**What it is.** Ash Maurya's adaptation of Osterwalder's Business Model Canvas, rebuilt for the risk profile of a startup: it replaces the BMC's four "business model" boxes with problem-centric boxes. The nine boxes: Problem (top-3 problems), Solution, Key Metrics, Unique Value Proposition, Unfair Advantage, Channels, Customer Segments, Cost Structure, Revenue Streams. The "problem-first" layout is the point: a lean startup's first job is to find a problem worth solving, not to optimise a business model.

**Verified origin.** Ash Maurya's own blog, "Why Lean Canvas versus Business Model Canvas" (blog.leanstack.com) ✅ — extracted this pass; in it he explains why he "created a different adaptation from the original Business Model Canvas by Alex Osterwalder". The book *Running Lean* (O'Reilly) that popularised it is ⚠ on exact edition/date (held from knowledge; the 2012 second edition is the commonly cited one).

**When to use it.** At idea stage, to get the riskiest assumptions onto one page before any code or capital is committed; it pairs naturally with the problem interview (§3.3) and with customer-development practice.

**Limitations.** One page, nine boxes — the discipline is that everything on it is a hypothesis. It does not prioritise, does not measure, and its "unfair advantage" box is famously the one nobody can fill honestly. For a bank's context, the Lean Canvas also predates regulatory reality: compliance, conduct, and capital constraints do not fit its boxes and must be handled separately (§11).

### 3.3 The problem interview

**What it is.** The customer-development interview that tests whether a stated problem is real, frequent, and painful enough to build for — before solutioning. The canonical shape is three parts: (1) the introduction (set the frame: "we're researching, not selling"), (2) collecting the story (have the customer narrate the last time the problem happened — past behaviour beats future opinion), and (3) qualifying (frequency, current workarounds, what they tried, what it cost them). The signal you want is not enthusiasm but *evidence of existing behaviour*: workarounds, money already spent, and a willingness to be contacted again.

**Origin and status.** The practice descends from Steve Blank's customer development (*The Four Steps to the Epiphany*, 2005) and the lean-startup movement; the bundle verifies Blank's *Four Steps* existence via Sean Ellis's own citation of it in "The Startup Pyramid" ("In Steve's book *Four Steps to the Epiphany* he writes: 'Customer Validation proves that you have found a set of customers and a market who react positively to the product'") ✅ for the quote and book; the exact interview script mechanics are practitioner-standard material ⚠ (held from knowledge).

**When to use it.** Before building anything; every time the team is about to commit to a solution for an unvalidated problem.

**Limitations.** Interviews are cheap to run and expensive to interpret — they are the most biased instrument in the toolkit unless the interviewer actively resists leading questions and solution talk. One-on-one interview craft (listening, questioning, the facilitator's hat) is derived in the [Workshop Methodology](../management/workshop_methodology_guide.md) and [Facilitation Skills](../management/facilitation_skills_guide.md) guides; this section cross-references rather than re-derives them.

### 3.4 The three canvases compared

The Value Proposition Canvas, the Lean Canvas, and their parent Business Model Canvas are a family, and knowing the family tree prevents grabbing the wrong one:

| | Business Model Canvas | Lean Canvas | Value Proposition Canvas |
|---|---|---|---|
| **Focus** | The whole business model — 9 blocks covering customers, value, channels, relationships, revenue, resources, activities, partners, costs | The startup's riskiest assumptions — 9 boxes, problem-first | The fit between one customer segment and one value proposition |
| **Origin** | Osterwalder & Pigneur, *Business Model Generation* (2010) ⚠ (book not extracted this pass) | Ash Maurya, "Why Lean Canvas versus Business Model Canvas" ✅ | Osterwalder & Pigneur, *Value Proposition Design* ✅ (via Strategyzer's page) |
| **Relationship** | The parent | A startup adaptation of the parent (Maurya's own framing: "a different adaptation from the original Business Model Canvas by Alex Osterwalder") | A zoom-in on the parent's two central blocks (value proposition × customer segments) |
| **Best for** | Redesigning an established business's model | Early-stage assumption surfacing and iteration | Designing and testing a specific value proposition |
| **The banking note** | Useful for a bank's new-business line (e.g., a digital-banking sub-brand) | Useful for a bank's innovation lab — but the compliance and capital boxes must be overlaid, as they are not in the canvas | The natural pair for JTBD discovery on a banking journey (§11.3) |

The rule of thumb: if the question is "is the whole business coherent?" use the BMC; if it is "which of our assumptions will kill us?" use the Lean Canvas; if it is "does this offering fit this customer?" use the VPC. In a bank, all three get the §11 overlay: regulatory obligations are a block that no canvas carries natively.

---

## 4. Prioritisation: RICE

### 4.1 What it is

RICE is a scoring system for prioritising project ideas, published by Intercom's product team. It combines four factors into a single score that measures "total impact per time worked":

**RICE = (Reach × Impact × Confidence) / Effort**

| Factor | What it measures | The original scale (Intercom, 2016) |
|---|---|---|
| **Reach** | How many people each project will affect within a given period — for Intercom's team, "how many customers will this project impact over a single quarter?" | Number of people/events per time period |
| **Impact** | How much each affected person is moved | 3 = massive, 2 = high, 1 = medium, 0.5 = low, 0.25 = minimal (multiple-choice, to avoid decision paralysis) |
| **Confidence** | How sure you are about the reach and impact estimates | 100% = high, 80% = medium, 50% = low; anything below is a "total moonshot" |
| **Effort** | Total time the project requires from product, design, and engineering | Person-months |

### 4.2 The verified origin

The bundle contains the full text of the original post, captured by the Wayback Machine at 17 March 2016 (blog.intercom.io/rice-simple-prioritization-for-product-managers/), and the author is verified from page metadata:

- **Author:** Sean McBride — the current intercom.com page metadata contains "Sean McBride" eight times (verified this pass via the page's structured data) ✅.
- **The development story, from the post's own words:** "Prioritization is a perennial challenge when building a product roadmap. How do you decide what to work on first?" … "So, last August, we began developing our own scoring system for prioritization from first principles. After lots of testing and iteration, we settled on four factors, and a method for combining them." — i.e., development began August 2015 ✅. The author later notes, "I've used RICE to score over 100 project ideas during the past six months" ✅.
- **The date:** the earliest Wayback capture is 17 March 2016; six months back from the capture is ~September 2015, and "last August" is August 2015 — the post therefore dates to the 2015–2016 window, and the canonical public record treats RICE as published **January 2016**. The exact day is ⚠ (not independently confirmable from the bundle); the year 2016 and the August-2015 development start are supported by the primary text ✅.
- **A dating wrinkle worth knowing:** the *current* Intercom page (intercom.com/blog/rice-simple-prioritization-for-product-managers/) carries a datePublished of 2018-01-05 and dateModified of 2025-01-03 in its structured data — that is a republication, not the original date. Anyone citing "2018" for RICE is citing the republish, not the invention.

### 4.3 How to run it

1. List the candidate initiatives (problems, not features — see the Now-Next-Later discipline in §12).
2. For each, estimate reach (use real product-metric measurements where possible, per the original post's usage notes), pick impact and confidence from the fixed scales, and estimate effort in person-months.
3. Compute the score, sort, and review: the original post's usage notes explicitly recommend re-examining the list — "Are there projects where the score seems too high or too low?" — before committing.
4. Use the scored list to "decide between hard-to-compare ideas" and to "defend those decisions to others".

### 4.4 When to use it — and its limitations

- **Use it** when a backlog has grown past gut-feel size and decisions must be comparable and defensible — the exact problem the post describes ("Prioritization is a perennial challenge when building a product roadmap").
- **Limitations.** The author's own caveat, verbatim from the original post: "Of course, RICE scores shouldn't be used as a hard and fast rule. There are many reasons why you might work on a project with a lower score first. One project may be a dependency for another project, so it needs to happen first, or another feature might be 'table stakes' to sell to certain customers." Beyond that: the scores inherit every estimation error (garbage-in-garbage-out); confidence can be gamed to move pet projects up; effort in person-months is a coarse proxy that hides team composition; and the 3/2/1/0.5/0.25 impact scale compresses genuinely different magnitudes into five buckets. RICE ranks *candidate ideas*; it does not decide dependencies, regulatory obligations, or table stakes — a point that matters acutely in banking (§11).

---

## 5. Prioritisation: The Kano Model

### 5.1 What it is

The Kano model is a theory of customer satisfaction developed in the 1980s by Noriaki Kano: product features do not contribute to satisfaction on a single linear scale; they belong to categories with different satisfaction dynamics. Knowing which category a feature falls into tells you whether to *build it* (must-be), *compete on it* (one-dimensional), *delight with it* (attractive), *drop it* (indifferent), or *avoid it* (reverse).

| Category | When fulfilled | When absent | Example (from the Kano literature) |
|---|---|---|---|
| **Must-be Quality** | Customers are neutral — it is expected, taken for granted | Customers are very dissatisfied — it is the price of entry | A functioning brake in a car; a clean room in a hotel |
| **One-dimensional Quality** | Satisfaction rises with performance — "the spoken attributes, the ones companies compete on" | Dissatisfaction | A milk package claiming "10% more milk" delivers satisfaction if true, dissatisfaction if it under-delivers; call-centre resolution time |
| **Attractive Quality** | Satisfaction — unexpected delight, usually unspoken | No dissatisfaction | A thermometer on a milk carton; proactive escalation and instant resolution of a call-centre issue |
| **Indifferent Quality** | Neither satisfaction nor dissatisfaction — suppress it and save cost | Same | The thickness of the wax coating on a milk carton |
| **Reverse Quality** | Dissatisfaction — not all customers are alike; some prefer the basic model | Satisfaction for the segment that prefers it | Heavy jargon or excessive scripts in a call centre; over-engineered feature sets |

### 5.2 The verified origin

- **Full citation (verified this pass via the Wikipedia record's references):** Noriaki Kano, Nobuhiko Seraku, Fumio Takahashi, Shinichi Tsuji, "Attractive Quality and Must-be Quality" (「魅力的品質と当り前品質」), April 1984, *Journal of the Japanese Society for Quality Control* (*Hinshitsu*), 14(2), 39–48; the work was presented at the 12th Annual Meeting of the Japanese Society for Quality Control ✅.
- **Lineage:** the satisfaction-drivers table in the literature explicitly links Kano's categories to Herzberg et al.'s (1959) hygiene/motivator distinction and to later reformulations (Cadotte & Turgeon 1988; Brandt 1988; Brandt & Scharioth 1998; Llosa 1997/1999) ✅.

### 5.3 The mechanics

Kano proposed a standardised two-question-per-feature survey, measured implicitly rather than by direct asking:

| Question | Response scale |
|---|---|
| **Functional** — "How would you feel if the product had …?" / "…there was more of …?" | I like it / I expect it / I am neutral / I can tolerate it / I dislike it |
| **Dysfunctional** — "How would you feel if the product *did not* have …?" / "…there was less of …?" | Same five options |

Each respondent's answer pair maps to a category:

| Functional answer | Dysfunctional answer | Category |
|---|---|---|
| I expect it | I dislike it | Must-be |
| I like it | I dislike it | One-dimensional |
| I like it | I am neutral | Attractive |
| I am neutral | I am neutral | Indifferent |
| I dislike it | I expect it | Reverse |

Illogical pairs (e.g., "I like it" for both) are set aside as "questionable". Across respondents, the common aggregations are discrete analysis (modal category), continuous analysis, and satisfaction coefficients (see §12 for a worked classification).

### 5.4 When to use it — and its limitations

- **Use it** when triaging features before commitment — especially when the team is about to spend money on "delighters" while must-be's are still missing. It pairs naturally with requirements workshops and quality function deployment (QFD), which uses Kano categories to avoid mixing must-be characteristics into correlation matrices where they distort weights.
- **Limitations.** Two questions per feature makes the survey expensive at scale; categories are not stable — attributes migrate over time from attractive to one-dimensional to must-be as expectations rise (the model's own dynamic, flagged as unsourced in the Wikipedia record); results vary by segment and culture; and the model deliberately ignores cost and effort — it tells you the *category* of a feature, not its business case. In practice Kano classifies, and RICE prioritises: run Kano to understand the feature, RICE to rank the work.

### 5.5 Satisfaction coefficients — the continuous analysis

Beyond the modal category, the standard continuous aggregation computes two coefficients per feature from the counts of attractive (A), one-dimensional (O), must-be (M), and indifferent (I) responses (⚠ — the standard practitioner aggregation, consistent with the "satisfaction coefficients" approach named in the Kano literature; the formula itself is presented as standard practice, not as a verified citation):

- **Better (satisfaction impact)** = (A + O) / (A + O + M + I) — how much the feature raises satisfaction when present.
- **Worse (dissatisfaction impact)** = −(O + M) / (A + O + M + I) — how much the feature lowers satisfaction when absent (the minus sign marks it as a dissatisfier).

Plotting every feature on Worse (x-axis) against Better (y-axis) gives the classic Kano chart, and the build rule falls out of the quadrants: features with high Worse are must-be's (do them first — absence is pain); features with low Worse and high Better are attractive (do them last — presence is delight); features low on both are indifferent (drop them). The coefficients also quantify the §5.4 dynamics: a feature migrating toward must-be shows its Worse rising over successive surveys — which is exactly what regulators do to a banking feature's Worse score when they make it mandatory (§11.3).

---

## 6. Prioritisation: ICE, the Effort/Impact Matrix, and MoSCoW

### 6.1 ICE: Impact × Confidence × Ease

**What it is.** The growth-marketing experiment scorer: rank each candidate experiment on three 1–10 scales — Impact (how much the experiment moves the metric), Confidence (how sure you are), Ease (how easy it is to run) — and take the product (usually divided by 10 to keep scores in single digits). High-velocity teams score the whole experiment backlog in a session and work down the list.

**The verified origin — honestly flagged.** The ICE score is pervasively attributed to **Sean Ellis**, founder of GrowthHackers and author of *Hacking Growth* (with Morgan Brown, 2017). Sean Ellis himself is thoroughly verified this pass: his own blog identifies him as "author of Hacking Growth" ✅, and his blog's October 2013 update points to "our new project at GrowthHackers.com" ✅ (bundle file 04). But the *specific origin of the ICE score* — the canonical post or page that first defined Impact × Confidence × Ease, and the date — could not be pinned to a primary source this pass: the previous research pass was mid-way through checking GrowthHackers' pages when it timed out, and the search-engine and CDX routes failed. **Status: ⚠ on the exact origin and date; ✅ on the Sean Ellis / GrowthHackers association.** The guide therefore says "attributed to Sean Ellis / the GrowthHackers community" and flags the precise coinage as unverified (§13, §14).

**When to use it — and its limitations.** ICE is built for high-volume, low-cost, reversible experiments where ranking speed matters more than precision — the growth-team context. It has no reach factor (an experiment that moves a metric for 10 users scores the same as one that moves it for 10,000), the Ease axis is inverse-scaled and consistently misread, and the 1–10 scales are pure judgement, so scores are easy to game. For customer-impact decisions with reach differences — a bank's product roadmap, say — RICE (§4) is the more appropriate instrument; ICE is best confined to experiment backlogs.

### 6.2 The effort/impact matrix

**What it is.** The 2×2 that maps candidate initiatives by value (impact) against cost (effort), producing four archetypes: **quick wins** (high impact, low effort — do first), **big bets / major projects** (high impact, high effort — plan and sequence), **fill-ins** (low impact, low effort — do when spare capacity exists), and **time sinks / thankless tasks** (low impact, high effort — kill or re-scope). It is the visual workhorse of prioritisation workshops because it takes five minutes, needs no arithmetic, and makes disagreement visible.

**The origin — honestly stated: none.** The effort/impact matrix is general practice. Unlike RICE (a published 2016 Intercom post), Kano (a 1984 paper), or the Sean Ellis test (a 2009 blog post), the value-vs-effort quadrant has no single verifiable originator — it is the standard 2×2 prioritisation pattern that appears across agile, lean, and consulting practice in dozens of near-identical variants. **Status: ⚠ — presented as general practice with no single verifiable origin**, exactly as the evidence warrants.

**When to use it — and its limitations.** Use it as the shared visual for a scoping conversation, especially with stakeholders who will not trust a score they did not help produce. Its weaknesses are the mirror of its simplicity: no confidence axis (estimates are treated as fact), no reach dimension, coarse buckets, and no way to express dependencies or "table stakes" — a feature can be a high-effort/low-impact "time sink" and still be legally mandatory (in banking it usually is, §11). The matrix triages; RICE or Kano then do the heavy lifting.

### 6.3 MoSCoW — cross-reference, not re-derived

MoSCoW (Must-have, Should-have, Could-have, Won't-have-this-time) is the requirements-prioritisation scheme of the DSDM lineage: it sorts the *already-scoped* requirements of a delivery into four priority buckets so that a fixed-time, fixed-cost delivery can drop Could-haves instead of slipping the date. Its full mechanics — the DSDM history, the bucket definitions, the "Must-have means the release fails without it" test, and its use inside the double diamond — are derived in the [Workshop Methodology](../management/workshop_methodology_guide.md) guide, which carries the verified DSDM/MoSCoW lineage. This guide deliberately does not re-derive them; the relevant point here is placement: MoSCoW operates on requirements *inside* a committed delivery, while RICE, Kano, and the effort/impact matrix operate on candidate initiatives *before* commitment. A team that uses MoSCoW to decide which initiatives enter the backlog is using it outside its home context — the same misuse warning as §1.2.

---

## 7. Strategy and Metrics: The North Star Metric and OKRs

### 7.1 The North Star metric

**What it is.** One metric, chosen to sit at the top of the measurement system, that best captures the *core customer value* a product delivers — the single number that, if it grows, means the product is delivering on its promise. The discipline is not the metric itself but the filtering it imposes: every other metric becomes a driver of the North Star (or a guardrail against gaming it), and every roadmap conversation is anchored to it. Canonical examples — Airbnb's nights booked, Spotify's time listened, Facebook's daily active users — are commonly cited, but the bundle did not verify a canonical list, so the examples are given here as illustration, not as verified claims.

**The verified origin — partly flagged.** The North Star metric is a practice-origin concept: it is associated with Sean Ellis and the GrowthHackers community (verified person: Sean Ellis, GrowthHackers founder, author of *Hacking Growth* — see §6.1 ✅), and the canonical public artifact is **Amplitude's North Star Playbook (2018)**. This pass extracted amplitude.com/north-star directly: the playbook exists ✅, and its page quotes John Cutler ("Former Amplitude Product Evangelist, co-author of The North Star Playbook") ✅ — the co-authorship the previous pass was mid-verifying is now confirmed. **Status: ✅ on the playbook's existence and John Cutler's co-authorship; ⚠ on the exact coinage of the term and on the playbook's exact publication date (the page shows no date; "2018" rests on the public record).** A First Round Review interview commonly cited for Ellis on the North Star metric ("The Only Thing That Matters") returns 404 and could not be verified ⚠.

**When to use it — and its limitations.** Use it when the organisation needs one shared definition of value to align roadmap, growth, and incentives — the antidote to metric soup and to teams optimising different numbers. Limitations: a single metric can be gamed (hence guardrail metrics — churn, support load, compliance); choosing it is itself a strategic bet that must be revisited; and it is a *destination* metric, not a *diagnostic* one — when the North Star is flat, the North Star alone will not tell you why. That is the job of the framework families below: HEART (§8.1) for experience diagnosis, AARRR (§8.2) for funnel diagnosis, and OKRs (§7.2) for converting the North Star into owned, time-boxed commitments.

**Choosing a North Star — the selection checklist.** The Amplitude playbook's own framing (verified this pass at amplitude.com/north-star ✅) is that the framework exists to tie three languages together — the language of the customer (needs, goals, experiences), the language of the product (features, workflows, releases), and the language of the business (vision, revenue, growth) — and that the metric is what connects them. The working checklist that descends from this practice (⚠ where noted — standard practice):

1. **Does it capture delivered value, not activity?** A login is activity; a completed payment is value. If the metric can grow while customers get nothing done, it is an activity metric (§12.3's rejected candidates show the test).
2. **Is it a leading indicator of the business?** It should correlate with the business outcomes the organisation ultimately cares about (revenue, retention, share), so that optimising it is not a detour.
3. **Can it be gamed, and what guardrails watch it?** Every single metric can be gamed; the discipline is naming the guardrails (complaints, fraud loss, cost-to-serve — §12.3) at the same meeting.
4. **Is it one metric, not a dashboard?** If the team cannot say it in one line, it is not a North Star; the diagnosis happens elsewhere (HEART, AARRR), not inside the star.
5. **Is the team ready to change it?** The playbook's own chapter list includes changing the North Star; a metric that survives contact with new strategy by inertia is a relic, not a star.

### 7.2 OKRs — cross-reference, not re-derived

OKRs (Objectives and Key Results) are the goal-setting system in which an *Objective* states a qualitative direction and 3–5 *Key Results* state measurable outcomes that define done; they are set quarterly, cascaded (loosely — alignment, not coercion), and reviewed weekly. The full mechanics — objective writing, KR quality tests, cadence, scoring, and the common failure modes — are derived in the [3D Managerial Effectiveness](../management/3d_managerial_effectiveness_guide.md) guide and the leadership guides of this repository, and this guide does not re-derive them. Two placement notes matter here:

- **OKRs are the bridge between the North Star and the roadmap.** The North Star says *what value*; OKRs say *who moves it, how far, by when*; the roadmap (Now-Next-Later, §12) says *what work*. The Now-Next-Later material itself frames initiatives as delivering "measurable progress toward an OKR" — the three instruments are designed to chain.
- **Origin, flagged honestly:** OKRs are popularly attributed to John Doerr — who brought them from Andy Grove's management-by-objectives practice at Intel to Google in 1999 and wrote *Measure What Matters* (2018) — but this pass did not verify that origin at a primary source; it is held from the author's knowledge base and flagged ⚠ in the audit, with the mechanics safely cross-referenced to the guides that derive them.

---

## 8. Strategy and Metrics: HEART and AARRR

### 8.1 HEART

**What it is.** Google's user-experience measurement framework for web applications: five dimensions — **Happiness** (attitudes, often via survey), **Engagement** (depth of involvement: frequency, intensity, duration), **Adoption** (new users of a feature or product), **Retention** (repeat use over time), and **Task Success** (efficiency, effectiveness — the classic UX metrics: completion rate, time on task). Each dimension is populated through the **goals–signals–metrics** cascade: define the goal for the product change, choose the signal that indicates the goal is met, then pick the metric that captures the signal. The framework's value is that it makes the *category* of user experience explicit before any number is chosen — you decide whether a change is an adoption play or a task-success play, and only then select the metric.

**Verified origin.** The framework is verified at Google Research: Kerry Rodden, Hilary Hutchinson, and Xin Fu, "Measuring the User Experience on a Large Scale: User-Centered Metrics for Web Applications", CHI 2010 (the ACM CHI conference on Human Factors in Computing Systems) — verified via the Google Research record this pass ✅.

**When to use it — and its limitations.** Use it whenever a product change is meant to improve the *experience* of existing users — exactly the territory where raw usage numbers lie (a feature can be heavily "used" because it is annoying, or unused because it is invisible). Limitations: it is a measurement taxonomy, not a target-setting system (it will not tell you how much happiness is enough); survey-based dimensions (happiness) are expensive and noisy; and its five dimensions map awkwardly onto non-consumer products — a banking payments system's "users" are often other systems, not people (see §11).

### 8.2 AARRR — cross-reference, not re-derived

AARRR ("Pirate Metrics") is the acquisition–activation–retention–revenue–referral funnel: the five-stage lifecycle lens that tells you *where* in the journey value leaks, so growth effort can be aimed at the leak. It is one of the most widely used startup metric frameworks and is traditionally attributed to Dave McClure (the "Startup Metrics for Pirates" presentation, ~2007) — an attribution this pass did not verify at a primary source and therefore flags ⚠ in the audit. The full mechanics — stage definitions, per-stage metrics, and the funnel-diagnosis workflow — are derived in the [E-Commerce Experience](../management/ecommerce_experience_guide.md) guide (which carries the AARRR treatment), and this guide does not re-derive them.

Placement notes for this guide:

- **AARRR diagnoses the funnel; HEART diagnoses the experience; the North Star sets the destination.** The three are complementary, not competing: AARRR answers "where do users drop off?", HEART answers "how do users feel and perform?", the North Star answers "what does success mean?". A typical review cycle runs them in that order.
- **The banking caveat.** In banking, "acquisition" and "activation" are heavily regulated processes (onboarding and KYC are not free-form funnels you can redesign at will — §11), and "referral" collides with customer-data and inducement rules. The funnel lens still works, but several of its levers are unavailable to a bank; the e-commerce guide's treatment assumes the unregulated context and the banking guide's constraints must be applied on top.

---

## 9. Product-Market Fit: The Sean Ellis Test

### 9.1 What it is

The Sean Ellis test is a one-question survey that measures product-market fit by asking *existing users* how they would feel if they could no longer use the product. The rule of thumb: **if at least 40% say they would be "very disappointed", the product has achieved product-market fit**. The question is deliberately behavioural — it does not ask users whether they like the product, it asks them to forecast their own loss — and it is deliberately run only against people who have actually used the product.

### 9.2 The verified origin — the Startup Pyramid

The test comes from Sean Ellis's blog post **"The Startup Pyramid"** on startup-marketing.com, and the bundle contains both the current page and the 13 December 2009 Wayback capture:

- **The question, verbatim (bundle file 07):** "I've tried to make the concept less abstract by offering a specific metric for determining product/market fit. I ask existing users of a product how they would feel if they could no longer use the product. In my experience, achieving product/market fit requires at least 40% of users saying they would be 'very disappointed' without your product. Admittedly this threshold is a bit arbitrary, but I defined it after comparing results across nearly 100 startups. Those that struggle for traction are always under 40%, while most that gain strong traction exceed 40%." ✅
- **Dating:** the Wayback CDX record verified this pass shows captures from 31 July 2009 (302→200) and 18 September 2009 onward, and the post's own image asset path reads "uploads/2009/07" — the post is therefore from **~July 2009**, with the exact day ⚠ (bundle files 04/07) ✅.
- **The pyramid itself:** product/market fit is the foundation; the pre-scale steps above it are *Promise* (highlight the benefits described by the "must have" users), *Economics* (the business model that allows profitable acquisition), and *Optimize* (a repeatable, scalable acquisition process) — with the claim that executing these "often improves the conversion rate to transactions by 5X or more" ✅ (Ellis's own claim, presented as his claim).
- **The intellectual anchors Ellis cites in the post** (verified as cited ✅): Paul Graham — "make things people want"; Steve Blank's *Four Steps to the Epiphany* — "Customer Validation proves that you have found a set of customers and a market who react positively to the product"; Marc Andreessen's 2007 blog post — "the life of any startup can be divided into two parts – before product/market fit and after product/market fit" (noted in the post as removed from his blog and accessible via archive.org).

### 9.3 The mechanics

1. Survey existing users (post-signup, post-value — the sample must be people who have used the product).
2. Ask the single question: "How would you feel if you could no longer use the product?" with the canonical three options — *Very disappointed / Somewhat disappointed / Not disappointed*. **The question stem and the 40% threshold are ✅ verified; the exact wording of the three option labels is ⚠** (held from the author's knowledge base; the bundle verifies the stem and the "very disappointed" category, not the full option list).
3. Compute the percentage of "very disappointed" responses among a meaningful sample (Ellis's own practice compares across user cohorts; his later "must have" work — see §9.4 — segments by use case).
4. Below 40%: do not scale — keep burn low, engage users face-to-face, and iterate on the "must have" experience. At/above 40%: the pyramid's Promise → Economics → Optimize sequence is the next job.

### 9.4 The "must have" extension — and the survey-tool lineage

The same bundle (file 04) captures Ellis's later elaboration on startup-marketing.com: the goal is a "must have" product experience — "the key to sustaining rapid growth is understanding your 'must have' experience and then aligning the entire business around that experience," learned across startups including Dropbox, LogMeIn, Lookout, and Xobni. That post also documents the productisation of the survey approach (MustHaveScore, later the Qualaroo survey product) — the practical lineage of the PMF survey ✅ for the post's existence and claims-as-claims.

### 9.5 When to use it — and its limitations

- **Use it** when a product has real users and the team needs a defensible, comparable signal of whether it is ready to scale — the exact decision the pyramid's foundation exists for.
- **Limitations.** Ellis himself calls the threshold "a bit arbitrary". It only works on *existing users* — it is meaningless pre-launch and near-meaningless in low-usage samples. It measures current fit, not future fit (the "must have" experience shifts with every release and every competitor). It does not diagnose *why* fit is missing — when the number is 25%, the test tells you to go talk to users, not what to change. And in banking, the "existing users" condition collides with the fact that a bank cannot quietly let a cohort of customers "lose" the product to measure their disappointment — the test must be run as an honest survey, not an experiment (§11).

### 9.6 Running the test properly — a short checklist

The test is one question, but its validity lives in the sampling and the reading (⚠ where noted, standard survey practice):

1. **Sample existing users only** — people who have used the product past the value moment. A post-signup survey that catches users before value inflates the "somewhat disappointed" bucket and deflates the signal.
2. **Get a meaningful n** — a few dozen responses will swing on noise; the 40% rule was calibrated on comparisons across nearly 100 startups (Ellis's own description), so the team should aim for a sample that a 5-point swing would not change the decision on.
3. **Segment the reading** — Ellis's later "must have" practice (bundle file 04) segments responses by use case, because a product can be a must-have for one job and a nice-to-have for another; the 40% rule is the aggregate gate, the use-case split is the diagnosis.
4. **Re-run at milestones** — the "must have" experience shifts with every release and every competitive move; the test is a gate, not a certificate.
5. **Banking compliance** — the survey itself must respect PDPA consent and conduct expectations, and the "very disappointed" signal must come from honest survey responses, never from a degraded-experience experiment (§11.2, §11.4).

---

## 10. The Framework Selection Guide

### 10.1 Stage × question × framework

The mapping below is the working answer to "which framework do I reach for now?" — stage of the product lifecycle on one axis, the question being asked on the other.

| Stage | The question being asked | Reach for | Why |
|---|---|---|---|
| **Idea / pre-product** | What problem is real and worth solving? | Problem interview (§3.3), Lean Canvas (§3.2) | Both are built for unvalidated assumptions; the interview generates evidence, the canvas holds it |
| **Idea / pre-product** | What job is the customer hiring a solution for? | JTBD discovery (§2) | Job-level framing beats feature-level ideation; finds the real competition |
| **Value-proposition design** | Does the proposed offering relieve pains and create gains? | Value Proposition Canvas (§3.1) | The fit test is the whole point — evidence per customer |
| **Backlog triage** | Which candidate initiatives first? | RICE (§4) | Scored, comparable, defensible; the original post's "decide between hard-to-compare ideas" |
| **Backlog triage** | Which features delight vs. which are price-of-entry? | Kano (§5) | Category knowledge before commitment; prevents spending on delighters while must-be's are missing |
| **Experiment backlog** | Which growth experiments run this week? | ICE (§6.1) | Built for high-volume, low-cost, reversible experiments |
| **Scoping conversation** | Which quadrant does this initiative fall into? | Effort/impact matrix (§6.2) | Five-minute shared visual; makes disagreement visible |
| **Inside a committed delivery** | Which requirements must ship vs. can drop? | MoSCoW (§6, cross-ref) | The DSDM scheme for fixed-date, fixed-scope deliveries |
| **Strategy setting** | What does success mean for the whole product? | North Star metric (§7.1) | One definition of value to align roadmap, growth, incentives |
| **Quarterly goal-setting** | Who moves what, how far, by when? | OKRs (§7.2, cross-ref) | The bridge from the North Star to owned commitments |
| **Experience change** | Did the change improve how users feel and perform? | HEART (§8.1) | Goals–signals–metrics; the framework that names the dimension first |
| **Growth diagnosis** | Where in the journey do users leak? | AARRR (§8.2, cross-ref) | The funnel lens for acquisition-to-referral |
| **Scale decision** | Have we achieved product-market fit? | Sean Ellis test (§9) | The 40% "very disappointed" threshold on existing users |
| **Roadmap communication** | What are we doing now, next, later — honestly? | Now-Next-Later (§12) | Confidence horizons instead of false dates |

### 10.2 The chaining discipline

Frameworks are designed to run in sequence, not in isolation. The canonical chain this guide teaches:

1. **Discover** with problem interviews and JTBD — establish the job and the evidence for it (§2–§3).
2. **Shape** the value proposition with the canvases until it fits the evidence (§3).
3. **Prioritise** with RICE (ranked) and Kano (categorised), remembering that dependencies and table stakes override scores (§4–§5).
4. **Measure** with the North Star + OKRs for direction and HEART/AARRR for diagnosis (§7–§8).
5. **Gate** scaling on the Sean Ellis test (§9).
6. **Communicate** the whole thing on a Now-Next-Later roadmap so the confidence levels stay honest (§12).

### 10.3 The wrong-tool test

Three misuse patterns recur, and they are worth naming: (1) **using a ranking tool where a categorising tool belongs** — RICE-scoring features that are really must-be compliance items, when Kano's must-be category (or a regulator) has already decided them; (2) **using a diagnosis tool as a target** — setting an OKR on an AARRR stage metric without the North Star above it, which is how teams optimise funnels into the ground; (3) **using a commitment tool for exploration** — putting Later-stage bets on a dated roadmap (the exact failure Now-Next-Later exists to fix, §12). When a framework feels like it is fighting the situation, the situation usually wins: the frameworks are lenses, not laws.

---

## 11. The Banking-Product Angle

### 11.1 The same frameworks, a different physics

Every framework in this guide was born outside banking — most of them in consumer software. Applying them to banking products (cards, lending, payments, digital onboarding) works, but the physics change in three ways that matter more than the mechanics:

1. **The customer is often not the only user.** A payments system's "users" include merchants, counterparty banks, and other systems; a card product's "user" includes the scheme (Visa/Mastercard) and the acquirer. JTBD's "situation" is still findable — but so is the scheme's job, the regulator's job, and the operations team's job, and they all compete for the same roadmap.
2. **Table stakes are not a metaphor.** In the consumer world, "table stakes" is a figure of speech; in banking it is a regulatory obligation. RICE's own author concedes that "another feature might be 'table stakes' to sell to certain customers" — in banking, the must-be category of the Kano model is populated by statute: AML/CFT screening, capital and liquidity reporting, data-protection consent, auditability. A RICE score cannot outrank a Notice.
3. **Experimentation is constrained by law, not just by ethics.** The A/B-test culture that ICE and RICE assume — ship a variant, measure, keep the winner — runs into the regulatory stack the moment the test touches price, eligibility, or customer data.

### 11.2 The regulatory stack that bounds experiments

Cross-referencing the [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) guide (the repo's authoritative MAS treatment), the binding floor for a Singapore bank is the Banking Act 1970 plus the notices (the 600-series — e.g., Notice 626 AML/CFT, Notice 649 liquidity, Notice 658 outsourcing) and the FSMA tech-risk notices; the supervisory ceiling is the guidelines (TRMG, BCM, Outsourcing, IAC, FEAT). For product experimentation, the constraints that bite are:

| Constraint | What it means for product experiments | Anchor in the MAS guide |
|---|---|---|
| **Fair dealing / FEAT** | Differential treatment of customers — including in experiments — must be fair, explainable, and justifiable; "fairness" is a named FEAT principle (Fairness, Ethics, Accountability, Transparency, 2018). A bank cannot A/B-test a less favourable interest rate on a random cohort and call it a growth experiment | §4.6 FEAT — 2018 |
| **Customer data (PDPA)** | Using customer data to run experiments requires a lawful purpose and consent/legitimate basis; profiling and personalisation sit under the PDPA's purpose-limitation and notification obligations, which MAS enforces as a conduct matter | PDPA treatment in the MAS guide |
| **KYC/AML (Notice 626)** | The onboarding funnel is not a free-form AARRR activation canvas: verification steps are obligations, not drop-off points to optimise away. You may optimise the *experience* of KYC, not the *fact* of it | Notice 626 AML/CFT |
| **Technology risk (TRMG)** | Changes to customer-facing systems — including experiment variants — go through the bank's change-management and testing discipline; TRMG expects controlled rollouts, rollback, and operational-resilience proof, not permanent canary experiments | §4.2 TRMG — 2013/2021 |
| **Accountability (IAC)** | Named senior managers are accountable for the products and their conduct outcomes; a failed experiment is an accountable event with a paper trail, not a silently reverted flag | §4.5 IAC — 2020 |
| **The sanctioned experiment lane** | MAS operates a FinTech Regulatory Sandbox: the legitimate way to test innovative products with real customers under relaxed or suspended specific requirements, with agreed boundaries | The sandbox treatment in the MAS guide |

The practical consequence: **a bank's experimentation space is (a) non-price, (b) non-eligibility, (c) consented-data, and (d) change-managed.** What remains is plenty — UX variants, communications, feature gating, workflow redesign, service-channel routing — but the framework user must draw the boundary before scoring, not after.

### 11.3 How the frameworks translate

| Framework | Banking translation |
|---|---|
| **JTBD (§2)** | The job for a digital-onboarding product is not "open an account" — it is "get paid / get my salary in without visiting a branch / prove who I am once". The [DBS Bank](../banking/dbs_bank_guide.md) guide's §10.1 journey — Onboarding → KYC → Account Opening → PayLah! → Investment — is a job-sequence worth reading as a JTBD map, with every step carrying an obligation |
| **Kano (§5)** | Must-be = regulatory and scheme compliance (screening, limits, audit, chargeback handling); one-dimensional = price, speed, limits, coverage; attractive = the delighters (instant card issuance, spending insights, rounding-up savings); indifferent/reverse = the features that cost more than the segment values. Regulators keep repopulating must-be — Kano's dynamic where attractive decays into must-be is accelerated by every new Notice |
| **RICE (§4)** | Score the *discretionary* backlog; keep a parallel "obligations backlog" that no score can defer. Reach for a payments feature is "transactions per quarter", not "customers" — the original post's own definition ("number of people/events per time period") fits banking's event-heavy metrics well |
| **Sean Ellis test (§9)** | Run it as a survey on existing digital-banking users; the 40% threshold is meaningful for app engagement ("very disappointed if you could no longer use the app") but must never be proxied by quietly degrading service for a cohort |
| **HEART (§8.1)** | Task Success is the banker's dimension: onboarding completion, payment success, statement retrieval — the measurement framework maps cleanly onto digital-banking journeys; Happiness surveys must respect PDPA consent and conduct expectations |
| **North Star (§7.1)** | A bank's North Star candidate: "digitally active customers" (with guardrails: complaints, fraud loss, cost-to-serve) — chosen in the worked example (§12) |

### 11.4 What a bank cannot A/B-test freely — the honest list

- **Price and fees** (interest rates, charges, FX spreads) — fair-dealing and FEAT territory; differential pricing across test cohorts needs a justification that survives regulatory scrutiny.
- **Eligibility and credit decisions** — lending decisions are governed by responsible-lending expectations; an A/B test that approves a riskier cohort "to see what happens" is not an experiment, it is a conduct breach.
- **KYC/AML steps** — verification is an obligation; you can test *how* the step is presented, not *whether* it happens.
- **Customer data uses** — every experimental use of customer data must sit inside the PDPA purpose-limitation frame.
- **Anything customer-visible without change management** — TRMG's discipline applies to variants as much as to releases.

This is the boundary every framework section above has been pointing at: in banking, the frameworks rank and categorise and measure *within* the regulatory frame — they never override it.

### 11.5 The four domains, framework by framework

| Domain | Typical JTBD (§2) | Must-be (Kano + regulation) | Attractive (Kano) | RICE reach metric (§4) | Notes / cross-refs |
|---|---|---|---|---|---|
| **Cards** | "Pay anywhere without carrying cash or thinking about it" | Scheme compliance, chargeback handling, fraud alerts, limit controls | Instant virtual card issuance, spending insights, rounding-up savings | Card transactions per quarter (event-heavy) or new applications per quarter | The [DBS Bank](../banking/dbs_bank_guide.md) §10.1 journey shows the issuance-to-investment sequence a card product feeds |
| **Lending** | "Get the money I need now, sized to my life" | Responsible-lending checks, disclosure, cooling-off, conduct rules | Instant pre-approval, automated top-ups, transparent rate explanations | Loan applications per quarter | Eligibility and price are off-limits for A/B testing (§11.4); Kano's one-dimensional axis here is speed and clarity, not rate |
| **Payments** | "Move money and know it arrived" | Settlement integrity, AML screening (Notice 626), auditability, scheme rules | Real-time status notifications, payment links, request-to-pay | Transactions per quarter — RICE's own "people/events per time period" definition fits best here (§4.1) | The payments-modernisation programme in the workshop guide §11 is the architectural context |
| **Digital onboarding** | "Prove who I am once, and open everything" | KYC/AML verification (Notice 626), PDPA consent, data-retention rules | Single-ID reuse across products, video-verification UX, progress transparency | New customers onboarded per quarter | The funnel is regulated (§11.2): optimise the *experience* of KYC, never the *fact* of it |

Two cross-cutting observations. First, **the attractive category is where a bank's differentiation actually lives** — every bank has the same must-be floor and similar one-dimensional axes, so the delighters (instant issuance, round-up pots, payment status) are the only Kano category with real competitive headroom; the worked example's I3 shows how the frameworks protect that insight from the RICE ranking. Second, **the reach metric should be chosen per domain** — customer counts for onboarding, event counts for payments and cards — because the original RICE definition ("number of people/events per time period") was built precisely for this flexibility, and a payments backlog scored in "customers" quietly undervalues every high-frequency initiative.

---

## 12. The Worked Example: Cymbal Bank

### 12.0 The convention

**Cymbal Bank is the repository's recurring fictional bank persona** (see the [Workshop Methodology](../management/workshop_methodology_guide.md) guide's §11 worked example and its audit note): a Singapore retail-and-SME bank running a payments-modernisation programme on a 15-year-old stack. The cast, dates, and numbers in this section are pedagogical constructions in that persona — the *method* is the point, nothing here describes a real bank (audit #19). The example runs one quarter of product triage for the digital-banking team: four candidate initiatives across the cards, lending, payments, and onboarding domains, worked through RICE, Kano, the North Star choice, and a Now-Next-Later roadmap.

### 12.1 The RICE prioritisation — arithmetic shown

The team's four candidate initiatives for the quarter:

| # | Initiative (framed as a problem, not a feature) | Domain | Reach (per quarter) | Impact (3/2/1/0.5/0.25) | Confidence (100/80/50) | Effort (person-months) |
|---|---|---|---|---|---|---|
| I1 | "Get a working card in the customer's hands the day they apply" | Cards | 90,000 new card applications | 2 (high) | 80% | 3 |
| I2 | "Know my payment actually arrived, without calling the bank" | Payments | 1,200,000 customers making payments | 1 (medium) | 80% | 2 |
| I3 | "Save a little without thinking about it" (round-up pots) | Deposits | 400,000 app users in the target segment | 0.5 (low) | 50% (unvalidated demand) | 4 |
| I4 | "SME credit line that grows with my sales" (automated top-up) | Lending | 8,000 SME customers | 3 (massive) | 50% | 6 |

The scores, per the original RICE formula **RICE = (Reach × Impact × Confidence) / Effort**:

| # | Arithmetic | Score | Rank |
|---|---|---|---|
| I1 | (90,000 × 2 × 0.8) / 3 = 144,000 / 3 | **48,000** | 2 |
| I2 | (1,200,000 × 1 × 0.8) / 2 = 960,000 / 2 | **480,000** | 1 |
| I3 | (400,000 × 0.5 × 0.5) / 4 = 100,000 / 4 | **25,000** | 3 |
| I4 | (8,000 × 3 × 0.5) / 6 = 12,000 / 6 | **2,000** | 4 |

The team then runs the original post's review step — "Are there projects where the score seems too high or too low?" — and the caveat clause. Three judgements come out of the review:

- **I2's reach needs a reality check.** 1.2M is "customers who made a payment" — the post's usage notes say to use real product-metric measurements; the team re-runs the number against the payments event log and confirms ~1.14M, score 456,000 — still rank 1. The re-check is the discipline, not the exact number.
- **I4's score is low but the project is a strategic bet.** RICE does not capture dependencies or strategy: I4 is the lending domain's answer to a board-level SME-growth objective and is a dependency for a later treasury initiative. Per the author's own caveat ("there are many reasons why you might work on a project with a lower score first"), I4 is kept in the plan — placed in *Later* on the roadmap (§12.4) and linked to an OKR, not killed by its 2,000.
- **The obligations backlog is not scored at all.** The compliance team's quarterly items — a Notice 626 screening-rule update touching the onboarding flow, and a TRMG-required failover test for the payments hub — get no RICE score. They ship in *Now* regardless (§11.2). The table above is the *discretionary* backlog; the obligations backlog runs beside it.

### 12.2 The Kano classification exercise

The team runs a functional/dysfunctional survey (two questions per feature, the five-point scale of §5.3) on four features with a panel of 60 existing customers. The modal answer pairs:

| Feature | Modal functional answer | Modal dysfunctional answer | Kano category (per the §5.3 table) |
|---|---|---|---|
| 24/7 fraud alerts on card transactions | I expect it | I dislike it | **Must-be** |
| Instant virtual card issuance | I like it | I dislike it | **One-dimensional** |
| Round-up savings pots (I3) | I like it | I am neutral | **Attractive** |
| Loyalty-points dashboard | I am neutral | I am neutral | **Indifferent** |

The lesson the exercise teaches the team:

- **Fraud alerts are must-be.** No RICE score, no MoSCoW debate — their absence is a dissatisfier and (given the bank's obligations) a compliance exposure. They go into the obligations backlog.
- **Instant issuance is one-dimensional** — the market competes on it (the DBS digibank journey in [DBS Bank](../banking/dbs_bank_guide.md) §10.1 shows what the benchmark looks like) — which is why I1's RICE rank 2 is defensible: it is a spoken, competed-on attribute.
- **Round-up pots are attractive** — unspoken delight. The Kano dynamic (§5.4) says the delight will decay into expectation over time, and RICE already ranked it 3rd, so the team *deliberately* builds it after the must-be and one-dimensional work — a decision that only makes sense because the two frameworks were read together: RICE ranked the work, Kano explained the rank.

### 12.3 The North Star choice

Candidates considered, with the guardrail question applied to each:

| Candidate | Captures core value? | Gameable? | Verdict |
|---|---|---|---|
| Monthly active app users | No — an inactive "active" user is common in banking apps (a login to check a balance is not value) | Yes, trivially | Rejected |
| Successful payment count | Partially — value, but only for the payments domain | Pushable via micro-payments | Rejected (too narrow) |
| NPS | Attitudinal, not behavioural | Survey noise | Rejected (as a *star*; kept as a guardrail) |
| **Digitally active customers** (a customer performing ≥1 value-adding digital interaction per month — payment, card transaction, deposit, loan drawdown — excluding pure balance-check logins) | Yes — it is the bank's core promise: primary-banking done digitally | Harder — the definition excludes vanity logins | **Chosen** |

The chosen North Star: **digitally active customers**, with guardrail metrics — complaints per 1,000 customers, fraud loss rate, and cost-to-serve — because a single metric can be gamed (§7.1). It chains to the quarter's OKR (cross-ref §7.2): *Objective* — "Make Cymbal Bank the primary digital bank for our customers"; *KR1* — digitally active customers from 1.2M to 1.5M; *KR2* — onboarding completion rate 68% → 80%; *KR3* — complaints per 1,000 customers ≤ 0.8. Note that KR2 is exactly the kind of activation metric AARRR would diagnose (cross-ref §8.2) and HEART's Task Success would measure (cross-ref §8.1): the chain of §10.2, run on one quarter of a bank's roadmap.

### 12.4 The Now-Next-Later roadmap

The roadmap is drawn as three confidence horizons (Janna Bastow's format, §5 of the bundle; mechanics cross-referenced from ProdPad's own material ✅), with initiatives framed as problems, not features:

| **Now** — in motion, well understood | **Next** — being validated | **Later** — strategic bets |
|---|---|---|
| I2 payment-status notifications (build) | I1 instant virtual card issuance (solution discovery) | I3 round-up savings pots (demand validation) |
| Notice 626 screening-rule UX rework (obligations backlog) | KYC re-verification flow redesign (discovery; must not weaken verification — §11.4) | I4 SME automated credit top-up (strategic bet, linked to the SME OKR) |
| TRMG failover test for the payments hub (obligations backlog) | Loyalty-dashboard evidence review (Kano: indifferent — likely killed) | Payments-hub event-backbone exploration (feeds the modernisation programme, [Workshop Methodology](../management/workshop_methodology_guide.md) §11) |

Reading the roadmap as designed (bundle file 05): Now holds only validated work — stakeholders can commit to it without date anxiety; Next is where discovery narrows the options (I1's solution exploration, the KYC flow); Later holds the big bets that may change or disappear — I4's 2,000 RICE score did not kill it, it placed it where confidence is honestly lowest. The roadmap *is* the communication of §10.2's chaining: RICE ranked, Kano categorised, the North Star aimed, and Now-Next-Later made the confidence levels visible. The dates are gone; the priorities are not.

### 12.5 The debrief — what the example is built to teach

- **Frameworks disagree on purpose, not by accident.** RICE ranked I4 last; strategy kept it. Kano called round-up pots attractive; RICE ranked them 3rd. The disagreement is the information — each framework answered the question it was built for, and the team's job was to read them together.
- **Banking changes two columns.** The obligations backlog is not a prioritisation output, and "Later" bets in banking still carry regulatory watching-briefs — the sandbox lane (§11.2) exists precisely because a bet must be testable *somewhere* legitimate.
- **The confidence horizons are the honesty.** The team could not promise I1 for a quarter because it had not validated the solution; the roadmap says so instead of pretending.

### 12.6 The quarter's decision, one table

The whole example, collapsed into the artefact the team actually walks into the steering committee with:

| Initiative | RICE score (rank) | Kano category | Roadmap horizon | Decision |
|---|---|---|---|---|
| I2 — payment-status notifications | 480,000 (1) | One-dimensional | **Now** | Build; the quarter's flagship |
| Notice 626 screening-rule UX rework | not scored (obligation) | Must-be | **Now** | Ship regardless of score; obligations backlog |
| TRMG payments-hub failover test | not scored (obligation) | Must-be | **Now** | Ship regardless of score; obligations backlog |
| I1 — instant virtual card issuance | 48,000 (2) | One-dimensional | **Next** | Validate the solution, then commit |
| KYC re-verification flow redesign | not scored (discovery) | Must-be (experience of an obligation) | **Next** | Discovery; verification itself untouched (§11.4) |
| Loyalty-dashboard | not scored | Indifferent | **Next** | Evidence review; likely killed |
| I3 — round-up savings pots | 25,000 (3) | Attractive | **Later** | Demand validation before commitment |
| I4 — SME automated credit top-up | 2,000 (4) | One-dimensional (for SMEs) | **Later** | Strategic bet; kept on the OKR, not killed by the score |

Read vertically, the table is the whole method in one view: the scores ranked the discretionary work, the Kano categories explained the ranks and caught the must-be's, the obligations backlog sat outside scoring, the North Star aimed the OKRs the initiatives feed, and the Now-Next-Later horizons told the committee exactly how much confidence each row deserved. That is the product-management framework library, run as one system rather than a menu.

---

## 13. The Claims Audit

The house rule of this repository: never present an unverified claim as fact. This audit lists every load-bearing factual claim in the guide, with its verification status. **✅ = verified this pass at a primary or authoritative source** (evidence given); **⚠ = flagged** (single-source, not re-verified, or held from the author's knowledge base); **❌ = disputed or false as stated** (correction given in the text).

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | JTBD lineage — MIT Sloan "Finding the Right Job for Your Product" (Christensen, Anthony, Berstell, Nitterhouse, 2007) is the real home of the milkshake article; the often-cited HBR 2007 URL 404s | ✅ | Full text extracted at sloanreview.mit.edu/article/finding-the-right-job-for-your-product/ (About the Authors section); hbr.org/2007/09/… returns Page Not Found |
| 2 | JTBD lineage — "Marketing Malpractice: How to Fix It" (Christensen, Cook, Hall) is HBR December 2005 | ✅ | HBR December 2005 issue page verified this pass (hbr.org/2005/12/marketing-malpractice-…) |
| 3 | JTBD lineage — *Competing Against Luck* (Christensen, Hall, Dillon, Duncan, HarperBusiness, 2016) | ✅ | Internet Archive catalog record archive.org/details/competingagainst0000chri |
| 4 | JTBD lineage — HBR "Know Your Customers' Jobs to Be Done" (September 2016) | ✅ | hbr.org/2016/09/know-your-customers-jobs-to-be-done extracted this pass |
| 5 | JTBD lineage — *The Innovator's Solution* (2003) popularised the term; Moesta/Spiek popularisation; Ulwick's ODI parallel lineage and the coinage dispute | ⚠ | Held from the author's knowledge base; the bundle verifies the papers (rows 1–4) but no primary Moesta/Spiek or Ulwick source was captured this pass; the coinage dispute is between living practitioners and is presented as such (§2.3) |
| 6 | The milkshake example's company is *disguised* ("The descriptions of the product and company in this example have been disguised") | ✅ | Sloan article, reference 1 |
| 7 | Value Proposition Canvas: Strategyzer's own page; method from *Value Proposition Design* | ✅ | strategyzer.com/library/the-value-proposition-canvas ("This method from the bestselling innovation book Value Proposition Design") |
| 8 | VPC lineage — *Business Model Generation* (Osterwalder & Pigneur, 2010) as the parent canvas | ⚠ | Held from the author's knowledge base; the book was not extracted this pass |
| 9 | Lean Canvas: Ash Maurya's adaptation of the Business Model Canvas | ✅ | Maurya's own blog post "Why Lean Canvas versus Business Model Canvas" (blog.leanstack.com) extracted this pass |
| 10 | *Running Lean* (O'Reilly) as the book that popularised the Lean Canvas | ⚠ | Held from the author's knowledge base; exact edition/date not extracted |
| 11 | RICE: Intercom, author Sean McBride, original post "RICE: Simple prioritization for product managers" | ✅ | Wayback capture 2016-03-17 of blog.intercom.io/rice-simple-prioritization-for-product-managers/ (full text in bundle); "Sean McBride" appears 8 times in the current page's structured data |
| 12 | RICE: "last August, we began developing our own scoring system… from first principles" (development began August 2015); "I've used RICE to score over 100 project ideas during the past six months"; formula (Reach × Impact × Confidence) / Effort; scales 3/2/1/0.5/0.25, 100/80/50, person-months; "not a hard and fast rule" caveat | ✅ | Verbatim from the original post (bundle file 08) |
| 13 | RICE publication date: canonical record treats it as January 2016; exact day not confirmable; current intercom.com page's 2018-01-05 datePublished is a republication | ✅/⚠ | Earliest Wayback capture 17 Mar 2016; "last August" = Aug 2015 ✅; exact day ⚠; current-page metadata (datePublished 2018-01-05, dateModified 2025-01-03) read this pass ✅ |
| 14 | Kano: full 1984 citation — Kano, Seraku, Takahashi, Tsuji, "Attractive Quality and Must-be Quality", April 1984, *Journal of the Japanese Society for Quality Control* (*Hinshitsu*), 14(2), 39–48, presented at the 12th Annual Meeting of the JSQC | ✅ | Wikipedia Kano-model record's references (verified this pass); Herzberg 1959 lineage table also ✅ |
| 15 | ICE score attributed to Sean Ellis / GrowthHackers | ✅/⚠ | Sean Ellis verified at his own blog ("author of Hacking Growth"; GrowthHackers.com referenced in his Oct 2013 update) ✅; the *specific origin and date of the ICE score* ⚠ (no primary source pinned; growthhackers.com/glossary/ice-score blocked automated retrieval) |
| 16 | Effort/impact matrix has no single verifiable origin — general practice | ⚠ | Honest negative: no originator found; presented as general 2×2 practice (§6.2) |
| 17 | North Star metric: Amplitude North Star Playbook exists; John Cutler co-author; Sean Ellis association | ✅/⚠ | amplitude.com/north-star extracted this pass — "John Cutler, Former Amplitude Product Evangelist, co-author of The North Star Playbook" ✅; the playbook's exact 2018 publication date ⚠ (page shows no date); the term's coinage ⚠; First Round Review "The Only Thing That Matters" 404s ⚠ |
| 18 | HEART: Rodden, Hutchinson, Fu, "Measuring the User Experience on a Large Scale", CHI 2010 | ✅ | Google Research record, verified this pass |
| 19 | AARRR attributed to Dave McClure ("Startup Metrics for Pirates", ~2007) | ⚠ | Held from the author's knowledge base; not verified at a primary source this pass; mechanics cross-referenced to the e-commerce guide |
| 20 | OKRs attributed to John Doerr / Andy Grove lineage | ⚠ | Held from the author's knowledge base; mechanics cross-referenced to the 3D-effectiveness and leadership guides |
| 21 | Sean Ellis test: the 40% "very disappointed" rule; the question stem; the ~July 2009 Startup Pyramid post | ✅ | startup-marketing.com "The Startup Pyramid" (current page + Wayback 2009-12-13 capture, bundle files 04/07); CDX captures from 2009-07-31; post image path "uploads/2009/07" → ~July 2009, exact day ⚠ |
| 22 | Sean Ellis test: the exact wording of the three survey option labels (Very / Somewhat / Not disappointed) | ⚠ | Stem and "very disappointed" category ✅ (bundle); full option list held from knowledge |
| 23 | Now-Next-Later roadmap invented by Janna Bastow (ProdPad co-founder) as an alternative to timeline roadmaps | ✅ | ProdPad FAQ ("created by ProdPad co-founder Janna Bastow") + her own account "What Is a Now-Next-Later Roadmap? Why I Invented It" (bundle files 05/06) |
| 24 | MoSCoW is the DSDM requirements-prioritisation scheme | ✅ (cross-ref) | DSDM/MoSCoW lineage verified in the workshop-methodology guide (cross-referenced, not re-derived here) |
| 25 | Banking constraints: FEAT 2018, TRMG 2013/2021, IAC 2020, Notice 626 AML/CFT, Notice 658 outsourcing, the MAS FinTech Regulatory Sandbox, PDPA data limits | ✅ (cross-ref) | The MAS guide's verified treatment (itself written from mas.gov.sg primary extracts); anchors cited in §11.2 |
| 26 | DBS digibank journey: Onboarding → KYC → Account Opening → PayLah! → Investment (§10.1 of the DBS guide) | ✅ (cross-ref) | dbs_bank_guide.md §10.1 heading verified in-repo |
| 27 | Cymbal Bank worked example: cast, dates, initiatives, scores | ⚠ | Pedagogical constructions consistent with the repo's Cymbal Bank persona (workshop guide §11 convention); not facts about any real bank |

---

## 14. What Could Not Be Verified

The honesty section — the items this pass could not pin down, and exactly why:

- **The exact origin and date of the ICE score.** The *association* is solid — Sean Ellis is verified as GrowthHackers founder and *Hacking Growth* author, and his blog points to GrowthHackers (audit #15). But the canonical definitional source for Impact × Confidence × Ease, and its date, could not be captured: the previous research pass timed out mid-way through the GrowthHackers check, and this pass's direct retrieval of growthhackers.com/glossary/ice-score was blocked. The guide therefore attributes ICE to "Sean Ellis / the GrowthHackers community" and flags the coinage as unverified (§6.1).
- **The North Star metric's coinage and the playbook's exact date.** The Amplitude North Star Playbook is verified to exist with John Cutler as co-author (the page itself quotes him: "co-author of The North Star Playbook" — audit #17). The page shows no publication date, so "2018" rests on the public record rather than the page; the term's coinage — who first said "North Star metric" and when — remains a practice-origin claim, not a documented one. The First Round Review interview commonly cited for Sean Ellis on the North Star ("The Only Thing That Matters") returns 404 and could not be used.
- **The JTBD coinage dispute.** The papers are verified (audit #1–#4): Sloan 2007, HBR December 2005, HBR September 2016, *Competing Against Luck* 2016. What could not be settled is the *phrase*: *The Innovator's Solution* (2003) details, the Moesta/Spiek popularisation, and Tony Ulwick's claimed coinage and ODI method are all held from the author's knowledge base with no primary source captured this pass. §2.3 presents the dispute as a dispute, which is the honest state of the public record.
- **Book details not extracted.** *Business Model Generation* (2010) as the VPC's parent, *Running Lean*'s exact edition, and *Measure What Matters* as the OKR book are all ⚠ — the bundle verifies the pages and posts that surround them (Strategyzer's page, Maurya's blog), not the books themselves.
- **Small-date precision.** The exact day of RICE's publication (January 2016 is the canonical date; the earliest Wayback capture is 17 March 2016 — audit #13), the exact day of the Startup Pyramid post (~July 2009, from the CDX captures and the image path — audit #21), and the exact wording of the Sean Ellis survey's three option labels (audit #22) could not be pinned to the day or to the word.
- **AARRR and OKR origins.** Both are traditional attributions (Dave McClure ~2007; Doerr/Grove lineage) that this pass did not verify at primary sources — the mechanics are safely cross-referenced to the guides that derive them, so the ⚠ sits on the origin stories, not on the frameworks' use.
- **The worked-example details.** The Cymbal Bank quarter — its initiatives, its scores, its roadmap — is explicitly pedagogical (§12.0, audit #27). Nothing in it describes a real bank's roadmap; Cymbal Bank is this repository's recurring fictional persona.

---

## 15. The Glossary

| Term | Definition |
|---|---|
| **Jobs-to-be-Done (JTBD)** | The view that customers "hire" products to get a job done in a given situation; the job, not the customer or the product category, is the unit of analysis (§2, Sloan 2007 ✅) |
| **Job** | The fundamental problem a customer needs to resolve in a given situation, with functional, emotional, and social dimensions (§2.1) |
| **Hiring criteria** | The experiences, features, and functions a customer weighs when choosing between job candidates (§2.2) |
| **Situation case** | The chronological record of events and thoughts that led to a purchase; ~25 cases constitute critical mass for job discovery (§2.2, Sloan 2007 ✅) |
| **Value Proposition Canvas** | Osterwalder & Pigneur's two-block tool: Customer Profile (jobs, pains, gains) × Value Map (products & services, pain relievers, gain creators); fit is the goal (§3.1, ✅) |
| **Lean Canvas** | Ash Maurya's problem-first nine-box adaptation of the Business Model Canvas for startups (§3.2, ✅) |
| **Problem interview** | The customer-development interview that tests whether a problem is real, frequent, and painful before solutioning (§3.3) |
| **RICE** | (Reach × Impact × Confidence) / Effort — Intercom's scored prioritisation, "total impact per time worked" (§4, ✅, Sean McBride 2016) |
| **Kano categories** | Must-be / one-dimensional / attractive / indifferent / reverse — the five satisfaction dynamics of features (§5, ✅, Kano et al. 1984) |
| **Satisfaction coefficients** | A continuous-analysis aggregation of Kano survey responses across participants (§5.3) |
| **ICE score** | Impact × Confidence × Ease — the growth-experiment scorer attributed to Sean Ellis / GrowthHackers (⚠ on exact origin, §6.1) |
| **Effort/impact matrix** | The 2×2 (value vs effort) triage visual: quick wins, big bets, fill-ins, time sinks; general practice, no single origin (§6.2, ⚠) |
| **MoSCoW** | Must / Should / Could / Won't-have-this-time — DSDM's requirements prioritisation inside a committed delivery (§6.3, cross-ref) |
| **North Star metric** | The single metric that best captures core customer value; aims the whole organisation (§7.1, ⚠ coinage, ✅ Amplitude playbook) |
| **Guardrail metric** | A metric that watches for gaming of the North Star (complaints, fraud loss, cost-to-serve) (§7.1, §12.3) |
| **OKR** | Objectives and Key Results — quarterly goal-setting bridging the North Star and the roadmap (§7.2, cross-ref, ⚠ origin) |
| **HEART** | Google's five-dimension UX framework: Happiness, Engagement, Adoption, Retention, Task Success, with goals–signals–metrics (§8.1, ✅ Rodden/Hutchinson/Fu, CHI 2010) |
| **AARRR (Pirate Metrics)** | Acquisition–Activation–Retention–Revenue–Referral — the funnel diagnosis lens (§8.2, cross-ref, ⚠ McClure attribution) |
| **Sean Ellis test** | The one-question existing-user survey; ≥40% "very disappointed" indicates product-market fit (§9, ✅, ~July 2009) |
| **Product-market fit** | The state in which a product is a "must have" for a large enough market — the pyramid's foundation (§9, Ellis's framing) |
| **Now-Next-Later roadmap** | Janna Bastow's confidence-horizon roadmap: Now (building), Next (validating), Later (strategic bets) — no dates (§12.4, ✅) |
| **Confidence horizon** | A roadmap column defined by certainty rather than calendar time (§12.4) |
| **Obligations backlog** | The banking parallel queue of regulatory must-dos that no prioritisation score can defer (§11.3, §12.1) |
| **Table stakes** | The must-be baseline — in consumer products a metaphor, in banking a statutory category (§11.1, RICE's own caveat ✅) |

**The closing discipline:** the frameworks in this guide are lenses, not laws — each was built for a specific question in a specific context, and the verified origin is the best clue to what that context was. Read the audit before you repeat a claim, flag what you cannot verify, and let the framework that answers the question — not the one that sounds most impressive — inform the product decision.




