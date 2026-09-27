# Work Experience with Business Development: A Comprehensive Guide — the Discipline, the Role, and the Lived Experience

> **A deep-dive reference on business development as a *job you do* and a *function you work with*: what the title actually means across organisations (established from real role definitions rather than career advice), what the work consists of day to day, why business development is not sales and where the two genuinely blur, the partnership and alliance motions and what usually goes wrong with each, the partner lifecycle, the contested problem of metrics and attribution, the skills and the ladder, the Asia context, the evidence-graded AI-era question, what a technical counterpart must give a BD function and must demand from it, a Cymbal Bank worked example of a partnership that is qualified, negotiated, declined in one case, and then owned — plus a claims audit, an explicit "could not be verified" section, and a glossary.**
>
> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Professional Development / Management & Leadership Series
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026

---

> **A note on verification and labelling.** Every substantive claim in this guide carries a marker. **✅** = verified in this pass against a named source. **⚠** = flagged: reported or widely repeated but not verified in this pass, or an estimate whose provenance is stated. **🔧** = the guide's own construction: analysis, taxonomy, or template that is *explicitly not* sourced to literature. **⛔** = rejected, or could not be verified. Where a definition of a role, a motion or a metric is given, the guide states **who published the definition** and **when**; where the publisher is also the seller of the thing being defined, that is said in the same sentence. The distinction between *documented*, *practitioner convention*, and *constructed* is maintained in every section, and §15 audits the load-bearing claims.
>
> **A note on the sources of this pass.** This guide was written with a search tool that **returned empty result sets for several queries** and a page-extraction tool that worked on most, but not all, primary pages. ✅ marks a claim verified against a source this guide could actually read in this pass — in practice: reference works, employer career pages, the firms' own partner-programme documentation, and one national statistics office. Pages that **failed to extract** and are therefore cited at a weaker grade include `salesforce.com/partners`, `partner.microsoft.com` (the marketing front door), `bls.gov` (both the Occupational Outlook Handbook and the OES tables), `strategicaccounts.org` (SAMA), and the Airwallex careers posting. **An empty search or a blocked page is recorded as a tool limitation, never as evidence that something does not exist.** §16 lists what could not be established.
>
> **What this guide is.** It owns **the business-development discipline and role**: the instability of the title, what the work actually consists of, the argument that business development is not sales, the partnership and alliance motions, the deal lifecycle, the metrics-and-attribution problem, the skills and the ladder, the regional context, the AI-era question, and the relationship between a BD function and the technical people it depends on.
>
> **What this guide is not.** It is not a sales-qualification guide ([meddicc_guide.md](meddicc_guide.md) owns MEDDPICC and deal review), not a survey of sales methodologies ([../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) owns that), not a post-sales guide ([post_sales_customer_facing_experience_guide.md](post_sales_customer_facing_experience_guide.md) owns what happens after the signature), not a business-case guide ([business_case_development_guide.md](business_case_development_guide.md) owns the construction of the case), not a revenue-product guide ([ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) owns revenue-line design), and not an advisory-career guide ([consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) owns the advisory-versus-delivery distinction). The boundary table in §1.2 says precisely which neighbour owns each object.
>
> **What to read first.** If you are considering the work, read **§2** (the title problem), **§3** (what the work is) and **§9** (the ladder and the routes in). If you are in an adjacent technical role and a BD function has just appeared in your life, read **§12** first, then **§7** and **§14**. If you own or run a partnership, read **§5**, **§6** and **§14**. If you want the worked example, it is **§13**.

---

## Table of Contents

1. [How This Guide Relates to the Repository, the Scope and the Decoder](#1-how-this-guide-relates-to-the-repository-the-scope-and-the-decoder)
2. [The Title Problem, Established from Real Role Definitions](#2-the-title-problem-established-from-real-role-definitions)
3. [What the Work Actually Consists Of](#3-what-the-work-actually-consists-of)
4. [Business Development Versus Sales](#4-business-development-versus-sales)
5. [The Motions and the Deal Types](#5-the-motions-and-the-deal-types)
6. [The Partner Lifecycle](#6-the-partner-lifecycle)
7. [The Metrics and the Attribution Problem](#7-the-metrics-and-the-attribution-problem)
8. [The Skills, and What Separates Good BD from Bad](#8-the-skills-and-what-separates-good-bd-from-bad)
9. [The Ladder, the Market and the Routes In](#9-the-ladder-the-market-and-the-routes-in)
10. [The Singapore and Asia Context](#10-the-singapore-and-asia-context)
11. [The AI-Era Effect on the Role](#11-the-ai-era-effect-on-the-role)
12. [Working with BD from a Technical Role](#12-working-with-bd-from-a-technical-role)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. How This Guide Relates to the Repository, the Scope and the Decoder

### 1.1 Two readings, both in scope

This guide deliberately treats **two different experiences** as its subject, and says so up front rather than pretending they are one thing.

**Reading (i) — the practitioner's experience of working *in* business development.** You hold the title, or a title that carries the work. Your week is partner calls, internal meetings, a pipeline that will not close this quarter and was never going to, and a spreadsheet of ecosystem relationships in various states of decay. You are measured — somehow — on things you do not fully control.

**Reading (ii) — the experience of working *with* a business-development function from an adjacent technical or architecture role.** You are a solution architect, engineer, product person or security reviewer. Someone in BD has promised something to a partner, and the first you hear of it may be a calendar invite, a design review, or a press release. Your job is to decide whether the integration is *wanted*, whether it is *feasible* on the timeline promised, and whether anyone will own it after launch.

Both readings are legitimate subjects and both are covered throughout. §12 is written specifically for reading (ii); §13's worked example is written as a two-sided scene with engineering and BD on opposite sides of the table.

🔧 *The two-readings framing is this guide's own construction, offered so a reader can locate themselves. It is not a distinction the industry draws.*

### 1.2 The boundary: what this guide owns, and what it does not

| Neighbouring guide | What it owns | What this guide does with it |
|---|---|---|
| [meddicc_guide.md](meddicc_guide.md) | **Sales qualification methodology** — MEDDICC/MEDDPICC, its lineage, deal review, forecasting, failure modes. | **Pointer only.** This guide never re-derives qualification. §4 and §7 refer to "qualification" as an inherited instrument and move on; §6's "qualify" stage means *partner* qualification, which is a different question and is labelled as such. |
| [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) | **The sales-methodology landscape** (note: this file lives in `technology/`, not `management/`). | **Pointer only.** §4 arguments about BD-versus-sales do not re-survey sales frameworks. |
| [post_sales_customer_facing_experience_guide.md](post_sales_customer_facing_experience_guide.md) | **The customer-facing experience after the sale** — lifecycle, commercial mechanics, retention arithmetic, metrics critique, enablement. | **Named cross-reference.** §6 uses it for post-launch discipline; §7 points at it for the metrics-critique *method* without re-deriving it. |
| [consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) | **The structural template of this series**, and **advisory-versus-delivery** (§5 there). | **Template + named cross-reference.** §12 points at its §5 and states only how BD differs from advisory work; it does not repeat it. |
| [business_case_development_guide.md](business_case_development_guide.md) | **Business-case construction** — how the case for an investment is built and defended. | **Pointer only.** §5 and §13 note that a partnership needs a case and do not build one. |
| [ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) | **Revenue-line design** — what is packaged, priced and monetised. | **Pointer only.** §5 describes the *motion* by which revenue moves through a partner, not the product or pricing design. |
| [vendor_management_guide.md](vendor_management_guide.md) | **The buy side** — how an enterprise selects, contracts with, governs and exits a supplier. | **Named cross-reference.** A partnership has a buy side too; where §6 and §12 touch it, they point here. |
| [reverse_job_search_guide.md](reverse_job_search_guide.md) | **Discoverability mechanics** — how a professional becomes findable. | **Named cross-reference only** in §9. |
| [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) | **The customer-embedded technical role** (FDE). | **Named cross-reference.** §12's "technical counterpart" is adjacent to the FDE archetype; its guide owns the role. |

🔧 *The "one owner per object of study" framing is this guide's own construction, offered as a reading aid. The industry does not organise itself this way — which is precisely the subject of §2.*

### 1.3 The decoder: the vocabulary of a BD job description, a partner pitch, or a partner-programme page

Business development has a worse vocabulary problem than most functions because its words are borrowed from **sales**, from **corporate finance**, from **distribution and channel marketing**, and from **legal structure**. The table below is the guide's own construction, written to be used against real postings, real programme pages and real partnership proposals. Definitions marked ⚠ follow common industry usage — that is, the usage is real and observable, but **no single authority defines the term**; the guide says so rather than dressing convention up as standard.

| Term | What it means in practice | What it is *not* | Marker |
|---|---|---|---|
| **Business development (the function)** | A function whose work is the analytical preparation of growth opportunities and the support and monitoring of their implementation (the Palgrave/Sørensen formulation, quoted in the reference work). | Not a defined profession with a single scope; §2 shows the title covering at least five materially different jobs. | ✅ definition as quoted; ⚠ as an occupational standard |
| **The partnership** | A durable commercial relationship with another organisation, usually not a new legal entity. | Not a synonym for "vendor" or "customer" — a partner is usually neither, or is both. | ⚠ industry usage |
| **The alliance** | A cooperative agreement between independent organisations pursuing agreed objectives while remaining independent; usually falls short of a legal partnership or affiliate relationship. | Not a joint venture (a new entity) in the definitions that exclude JVs, and *is* one in the definitions that include them — the literature genuinely disagrees. | ✅ definitions; ⚠ the disagreement is real |
| **The channel** | The route by which a product reaches a customer when it is not sold direct — distributors, resellers, agents. "Place" in the marketing mix. | Not a relationship; it is a *path*. A partner is an occupant of a channel. | ✅ |
| **The ecosystem** | The set of organisations around a platform or product whose offerings interact with it — complements, implementers, distributors, technology partners. | Not a managed asset by default, and not a synonym for "our partners"; most of an ecosystem you have no agreement with at all. | 🔧 the distinction |
| **Co-sell** | Two organisations jointly pursuing the *same* opportunity — the partner and the vendor's own sellers working one deal, sharing pipeline visibility and, often, funding. | Not reselling. In co-sell the vendor sells; in resell the partner sells. | ✅ (vendor programme pages) |
| **Resell** | A partner buys or transacts the product and sells it on to the end customer, owning the customer relationship in the process. | Not co-sell; and in practice the two are often run in the same programme by the same partner. | ✅ (vendor programme pages) |
| **OEM** | A company that produces parts or equipment that may be marketed by another company. The reference work states plainly that the term is **ambiguous** and confused with original-design and original-brand manufacturing. | Not a synonym for "integration" or "reseller", even though it is routinely used as one. Ask which meaning is intended. | ✅ ambiguity; 🔧 the advice |
| **Integration (as a motion)** | A partnership whose consideration is technical: the partner builds against your API, or you build against theirs, and the joint offering is the thing being sold. | Not the same as an integration *project* — the motion is commercial, the project is engineering. | 🔧 the distinction |
| **Referral** | One party passes a lead to the other, typically for a fee, a margin, or reciprocity, with no joint selling and no product change. One major vendor's own documentation describes the mechanism concretely: a published business profile listed where internal sellers search, leads delivered into the partner portal, with **required response time frames** attached. | Not co-sell, though programmes often route referrals through the same tooling. | ✅ (Microsoft Learn) |
| **The joint venture** | Two or more distinct firms combining a portion of their resources to form a **separate, jointly-owned entity**. | Not a strategic alliance in the definitions that exclude JVs; the reference work documents both definitional camps. | ✅ |
| **The pipeline** | The flow of potential clients a company has started developing, in which each opportunity carries an estimated probability of success and a projected volume. | Not a forecast. The weighted average of a pipeline is a staffing aid, and a bad one when the probabilities are guesses. | ✅ definition |
| **The qualified opportunity** | An opportunity that has met a defined threshold. The load-bearing point: **the threshold is set by whoever runs the programme**, not by a standard. One vendor's programme requires, among other things, a minimum number of launched opportunities and a minimum number of qualified opportunities within a rolling twelve months, and a minimum recognised revenue figure, before a partner may join. | Not a synonym for "real deal". Compare thresholds across programmes before comparing counts. | ✅ (vendor programme page) |

**One structural observation before anything else.** ⚠ *Unlike post-sales roles, business development has an actual professional body with an actual handbook and actual certifications* — the Association of Strategic Alliance Professionals (ASAP), a nonprofit which states it began in **1998** and describes itself as dedicated to advancing the practice of alliance and partnership management, offering **CA-AM** and **CSAP** certifications and publishing *The ASAP Handbook of Alliance Management* (the 5th edition announced as coming in fall 2026 on its own site) ✅. **But that body owns "alliance management", not "business development"** — and its own framing of the alliance lifecycle is a seven-phase cycle running from alliance-specific strategy through analysis and selection, value-creating negotiation, operational planning, structuring and governance, launching and managing, to transform/innovate/exit ✅. That gap — a real profession with a handbook for *alliances*, and no equivalent for *BD* — is the whole of §2.

---

## 2. The Title Problem, Established from Real Role Definitions

**This is the guide's opening finding and its central hazard.** "Business development" is not one job. It is a **title that organisations attach to at least five materially different roles**, and the divergence is visible in the employers' own definitions — not in career-advice content, which this guide does not treat as evidence.

The reference work states the problem in almost the same words, and it is the most quotable sentence available on the subject: the responsibilities associated with business development **vary across industries and countries**, often including tasks performed by IT programmers, specialised engineers, advanced marketers, key account managers, and professionals involved in sales and relationship management — and as a result **"it has become challenging to clearly define the unique characteristics of the business development function and to determine whether these activities directly contribute to profitability"** ✅ *(source: the reference work's "business development" article, retrieved this pass)*.

### 2.1 The five variants, with who actually defines them

| # | Variant | What the role is | Whose definition, and when | Grade |
|---|---|---|---|---|
| **A** | **New-partner / channel hunting** | Find organisations that could carry, resell or build on your product; sign them; hand them to a manager. Hunting, not farming. | **Vendor partner-programme documentation** (AWS Partner Paths: Software, Hardware, Services, Training and Distribution paths, each with an enrolment route and a qualification requirement ✅; AWS partner-programme pages retrieved this pass). The *job* of recruiting these partners is described in programme and path pages rather than by any occupational standard. | ✅ programme structure; ⚠ the role as a job description |
| **B** | **Alliance / partner management (farming)** | Own an existing relationship: governance cadence, joint planning, enablement, joint pipeline, renewal of the partnership itself. | **ASAP**, the professional body: seven-phase alliance lifecycle, CA-AM/CSAP certification, *Handbook of Alliance Management* ✅ (ASAP site, retrieved this pass; 3rd edition dated **January 2013** per the reference work's citation, ISBN 978-0-9882248-1-0). | ✅ |
| **C** | **Corporate development (M&A-adjacent)** | A different function that the word "development" pulls toward BD: planning and executing strategy **primarily through mergers, acquisitions or divestitures** — strategic planning, market and competitor mapping, phasing markets in or out, **arranging strategic alliances or partnerships or joint ventures**, identifying and acquiring companies, securing funding, divesting. | **The reference work's "corporate development" article** ✅ — which itself carries a *needs more citations* maintenance tag dating from **August 2012** ⚠, so treat it as a decent sketch clouded by weak sourcing. Corroborating employer usage: an Airwallex posting titled **"Director, Corporate Development & Investor Relations"** whose responsibilities begin with developing a global corporate development strategy and identifying, assessing and cultivating **a pipeline of actionable M&A opportunities including tuck-in acquisitions and larger transformational deals** ⚠ *(read from the search-result description of the employer's own careers page; the page itself could not be extracted in this pass)*. | ✅ function exists; ⚠ employer detail |
| **D** | **Strategic sales support / enterprise deal support** | Attached to the largest deals: structuring, commercial terms, partner involvement, the internal case. Carries a number, or is judged by the numbers it assists. | **Employer postings** — the observable pattern; **no professional body defines this** as a discrete occupation. | ⚠ employer usage only |
| **E** | **A rebrand of ordinary sales** | The same job as a sales role, with the letters "BD" in the title because it recruits better or prices differently. | **Google's own careers site** is the cleanest exhibit available: the posting is titled **"Business Development Manager, New Business Sales (English, Mandarin)"**, based in Shenzhen; its minimum qualifications ask for **"2 years of industry experience in advertising, consultative sales, or business development"**, and its responsibilities are *identify and expand new clients*, *formulate and implement go-to-market strategies*, *lead the entire business cycle from lead generation and initial engagement to agreement execution*, and *win clients through precise outreach, marketing and presentations* ✅ *(source: Google Careers, retrieved this pass; the employer publishing its own definition)*. Google also lists a **"Business Development Consultant, New Business Sales"** title ⚠. | ✅ |

**Conclusion, stated as the finding:** **"business development" names a FAMILY of roles rather than one profession.** ✅ This is not a rhetorical flourish; it is what the definitions show. A Google "Business Development Manager, New Business Sales" and an ASAP-certified alliance manager share a title fragment and almost nothing else — one is running a quota-bearing acquisition motion against advertisers, the other is running a governance cadence across two companies' organisations and reporting to a steering committee. A third person with the same title in a different building is evaluating an acquisition target.

### 2.2 Which variant dominates where — and how thin the evidence is

This is where the guide must be honest about the limits of what it can support.

- **In large technology vendors and cloud platforms:** the **programme-mediated** variants (A and B) dominate, because the vendor's business model *is* the ecosystem. This is inferable ✅ from the existence and scale of the partner programmes themselves — AWS describes its partner network as spanning **198 countries** ✅, and runs distinct paths and dozens of named programmes ✅. **But the inference that internal headcount is allocated this way is the guide's, not a source's** 🔧.
- **In startups and scale-ups:** the same title most often denotes **sales with a partner-adjacent flavour** (variant E or D) ⚠ — *practitioner-consensus observation, and the guide flags it as such. No source located in this pass quantifies the distribution.*
- **In listed corporates and banks with an M&A function:** **variant C** exists as a separate team, and the title "corporate development" is the one that is actually used for it ✅ — the reference work treats corporate development as its own subject with its own article, and the employer postings found this pass pair it with investor relations rather than with partnerships ⚠.
- **In pharma and biotech:** the reference work states that the business-development function **"seems to be more utterly matured"** in high-tech and **especially in pharma and biotech** ✅ *(as the reference work puts it — note the hedge in its own wording, which this guide preserves rather than hardens)*. In those industries "BD" is routinely the team that licenses, in-licenses and partners compounds — closer to variant C than to variant A. ⚠ *This guide did not verify a pharma BD job description in this pass.*
- **Where the evidence is thinnest, and the guide says so:** there is **no occupational classification located in this pass that has "business development" as a discrete code with a published scope note**, and the two national sources attempted — the US Bureau of Labor Statistics pages (`bls.gov`, both the Occupational Outlook Handbook entry for sales managers and the OES table) — **failed to load through the extraction tool** ⛔. Their absence from this guide is a **tool limitation**, not evidence that BD is unclassified in them. This is recorded again in §16.
- **The honest summary:** the *variant existence* is documented (variants A, B, C, E at ✅-grade from named publishers, D at ⚠). The *relative frequency* of the variants by organisation type is **not documented anywhere this guide could find, and is therefore offered only as inference, marked 🔧, where it is offered at all.**

### 2.3 What this means if you are reading a title

🔧 *Everything in this subsection is the guide's construction — a reading procedure, not a sourced finding.*

1. **Do not answer "what does BD do?" — ask "what is this BD team measured on?"** The answer separates A/B/C/D/E faster than any definition.
2. **Ask what sits on the other side of the table.** A hunting role (A) talks about *targets* and *logos*; a farming role (B) talks about *cadence*, *governance* and *joint roadmap*; corporate development (C) talks about *thesis*, *valuation* and *diligence*; deal support (D) talks about *the deal*; a rebranded sales role (E) talks about *quota*, *pipeline coverage* and *territory*.
3. **Ask which organisation publishes the definition you have been given.** If the answer is "our head of BD", that is a house definition and it is fine — but it is not a standard, and it will not transfer.
4. **Expect the title to be unstable inside one company over three years.** 🔧 *Inference from the observed practice of renaming functions; not sourced.*

**And the counter-hazard:** the fact that the title is unstable does **not** mean the *work* is incoherent. §3 argues the opposite — that beneath the label there is a recognisable bundle of activities, and it is the bundle, not the word, that you should be able to recognise.

---

## 3. What the Work Actually Consists Of

This section describes the day-to-day of the practitioner. It is written for reading (i). Where a claim is the guide's observation of how the work is normally described rather than a sourced fact, it is marked 🔧; where a mechanism is documented in a vendor programme or a professional body's own framework, it is marked ✅.

### 3.1 The six things the week is actually made of

🔧 *This taxonomy is the guide's construction. It is offered as a practical decomposition, not as a sourced model of the occupation.*

**1. The partner pipeline.** The list of organisations in some state of conversation — prospect, contacted, interested, evaluating, piloting, signed, dormant, dead. It is maintained in a CRM, in a spreadsheet, or in the practitioner's head plus a spreadsheet. The reference work's own definition of the general pipeline applies: a flow of potential clients, each carrying an estimated probability of success and a projected volume, used to project staffing ✅. **The honest difference from a sales pipeline:** a partner pipeline's stages are largely *unfalsifiable in a quarter*. "Interested" is not verifiable until it becomes "piloting", and "piloting" can last nine months without either party being able to say it has failed.

**2. Ecosystem mapping.** Building and maintaining the picture of who is around your platform: who integrates, who competes, who would integrate if asked, who used to integrate and stopped. Vendor programme structures make this mapping *concrete* rather than philosophical — AWS's Partner Paths (Software, Hardware, Services, Training, Distribution) are effectively a public ontology of the ways an organisation can relate to a platform ✅, and the "ecosystem" as used in vendor materials is the population of organisations occupying those paths plus those who don't. **The mapping is almost always stale, because the motion that updates it — a review meeting — is the first thing cancelled in a busy month.** 🔧

**3. Discovery conversations that are not sales calls.** The practitioner's most distinctive hour. In a sales call the objective is a next step toward a purchase. In partner discovery the objective is to find out whether the two organisations' interests *actually* overlap, which requires the counterpart to describe their own strategy honestly — something they will not do if they believe they are being sold to. 🔧 *Consequence, and it is the guide's analysis: a BD conversation that is run as a sales call answers the wrong question, because it optimises for a commitment instead of for information.* It is also the reason BD and sales people so often misjudge each other (§4.3).

**4. Negotiation of commercial terms with a counterpart who is also a competitor.** This is **coopetition**, and it is documented as a concept rather than invented here: firms engaging in both cooperation and competition simultaneously; a portmanteau of "cooperation" and "competition"; rooted in game theory (von Neumann and Morgenstern, 1944) and popularised in business use by the 1996 book of that title by Brandenburger and Nalebuff ✅ *(reference work, retrieved this pass; note that the reference work's illustrative examples carry "clarification needed" tags ⚠)*. The documented shape: companies collaborating in R&D, standard-setting or supply chain while competing on product ✅. **What the practitioner experiences:** a meeting in which the same organisation is both the person you need to close for the quarter and the person whose roadmap announcement will damage you next quarter. Terms are therefore written to survive the relationship cooling — exclusivity windows, most-favoured-customer constructs, data-sharing limits, termination triggers. 🔧 *The term list is the guide's; the concept is documented.*

**5. The internal advocacy load.** *This is the part that is systematically under-reported in published role descriptions and is the single most honest thing that can be said about the job.* 🔧 The guide's position, stated plainly: **in most organisations a majority of a BD person's working hours are spent persuading colleagues, not partners.** The work that must be done internally, for one partnership, includes: getting the partnership onto a roadmap that is owned by someone else; getting engineering to estimate the integration before it is promised; getting legal to accept terms that do not match the standard paper; getting finance to model a revenue line nobody believes; getting the partner's account team to actually talk to your account team; and getting a named owner appointed after launch. None of that is external work, and none of it is optional. **The guide cannot source a proportion for this** — no study located in this pass measures the internal/external ratio of BD time ⛔ — **so no number is printed.** 🔧 *What is asserted here is only the direction and the claim that the load is structurally large.*
   - **Corroborating evidence that internal work is real and formally recognised:** ASAP, the professional body, runs a member programme titled **"Aligning for Success: Internal Cohesion in Strategic Alliances"**, described as giving members "a forum for discussion-based learning focused on **building cohesion within their team and across their internal organization**" ✅ *(source: ASAP's own site, retrieved this pass)*. A professional body does not build programming around internal cohesion unless the internal side is where the difficulty is. **Note the grade honestly:** that ASAP runs the session is ✅; the inference that internal alignment is *the* principal difficulty is the guide's 🔧.

**6. The maintenance work after launch, and the long cycle.** The partnership is signed. Then: the integration is built (or was promised and is now being built under pressure), the enablement material is written (or not), the partner's sellers are trained (or not), the joint pipeline is reviewed monthly (or stops after two months), and the relationship owner is named (or not). §6 covers where this dies and cross-references the post-sales guide for the discipline. **The cycle length is the defining feature of the experience:** a partnership can take 6–18 months from first conversation to first joint revenue, and the practitioner has to remain credible to their own leadership throughout. ⚠ *The 6–18 month figure is the guide's characterisation of commonly described practice, NOT a sourced statistic — no source located in this pass measures partner-cycle duration, and this figure should not be quoted as one.*

### 3.2 A week in the life, honestly proportioned

🔧 *Illustrative week, constructed by this guide to show the shape. The proportions are not measurements and are not presented as typical of any particular organisation.*

| Day | External work | Internal work |
|---|---|---|
| Mon | Partner call: quarterly business review with an existing alliance; the partner has a new account team that has never heard of the joint offering | Rewriting the joint value proposition because the partner's new team does not understand it |
| Tue | Discovery call with a prospective channel partner | Chasing engineering for the answer to "can the API even do this?" — the question the prospect asked a week ago |
| Wed | Conference call at an unsuitable hour because the partner is in another timezone | Internal pipeline review where the partnership's "pipeline" is asked to convert to a number this quarter |
| Thu | Renegotiating a term that the partner's procurement team unilaterally reinterpreted | Escalating to secure a named owner on your side for the partner's support tickets |
| Fri | Drafting the partner announcement with marketing, who want it earlier than the integration is ready | Explaining to a sceptical product manager why this partner deserves roadmap attention |

**Read across that table: five of ten blocks are internal.** That is the honest answer to "what is the job" that §3.1(5) is making, and it is the reason a BD practitioner who cannot sell internally will fail even with excellent partners. 🔧

### 3.3 What the work is *not*, even though the job description may say so

- **It is not deal-closing at volume.** A BD practitioner with a partner-logo target may close two partnerships in a year and call it a good year. 🔧
- **It is not generally quota-carrying in the same way as sales** — and §7 examines what happens when it is made so anyway, because organisations do it constantly.
- **It is not marketing.** Marketing produces demand; BD produces *routes*. They are frequently conflated, and marketing is frequently asked to staff the announcement that BD made a promise to deliver. 🔧
- **It is not corporate development**, even where the title says "development" — one is a relationship function, the other is a transaction function (§2.1 variant C). ✅ *Distinction supported by the two functions having separate reference-work articles.*

---

## 4. Business Development Versus Sales

**This is the analytical core of the guide.** The distinction is real, it has consequences, and the boundary genuinely blurs — all three are true, and a guide that asserts only the first is selling something.

In one line: **BD builds the venue; sales works it.** 🔧 *That formulation is this guide's construction, and it is developed, with its consequences, below.*

### 4.1 The venue-building and venue-working distinction, with its consequences

A sales person arrives in a market that exists: buyers with budgets, a product they can describe, competitors they can name. Their skill is *converting* an available opportunity. A business-development person is often **constructing the conditions under which a market becomes available at all**: a channel that can reach buyers who will not take a direct call; an integration that makes the product usable in a segment where it is currently unusable; a co-sell agreement that puts your product in front of a partner's existing customers. Both are commercial work. They are not the same work, and the differences are structural — not cultural. 🔧 *What follows is the guide's analysis; where a mechanism is documented it is marked.*

| Dimension | Sales (venue-working) | Business development (venue-building) | Why the difference is structural, not stylistic |
|---|---|---|---|
| **Cycle length** | A quarter, a month, sometimes a week. Opportunity → proposal → close, repeatedly. | Multi-quarter to multi-year. First conversation to first joint revenue commonly spans 6–18 months ⚠ *(guide's characterisation, unsourced — §3.1)*. | A purchase decision is *made by* a buyer; a partnership decision is *co-created with* a counterpart organisation that must also move its own internal machinery. Two decision processes in series, not one. 🔧 |
| **Unit of work** | The opportunity. | The relationship, plus the mechanism (channel, integration, programme membership) by which revenue will flow. | An opportunity can be won or lost. A mechanism that works can be used by people who have never met you. That asymmetry is the whole economic case for BD. 🔧 |
| **What the number measures** | Revenue in a period, attributed to a defined deal. | Something that does not exist in the accounting: an influence, an enablement, a set of relationships that have not produced revenue *yet*. (§7 is the audit.) | Accounting can attribute a deal. It cannot attribute the venue. ⚠ *Practitioner-consensus claim; the guide found no source that resolves it, and §7 refuses to invent one.* |
| **Cadence** | Driven by the buyer's decision cycle and the quarter boundary. | Driven by the cadence both organisations agree to — and it decays the moment one side stops attending. | Documented in mechanism: ASAP's alliance lifecycle includes an explicit **"Launching and Managing"** phase and a **"Transform, Innovate, or Exit Gracefully"** phase ✅, which is a body telling its members that a partnership requires management *and* an end. Vendor referral programmes make the cadence explicit too, by imposing **required response time frames** on leads ✅ (Microsoft Learn). |
| **Internal-selling load** | Moderate: pricing approval, discount exception, legal review at the end. | High and continuous: the partnership must be re-sold internally every time an owner changes, a roadmap shifts, or a quarter goes badly. 🔧 *And see §3.1(5) for the evidence-grade on this.* | A sales person's internal asks are *transactions* (approve this), which have a queue. A BD person's internal asks are *resource commitments* (build this, own this), which have a competitor: everything else engineering is doing. 🔧 |
| **Compensation shape** | Commission on closed revenue, usually the majority of variable pay. | Commonly a mix: relationship or partnership objectives, programme metrics, and some revenue influence — but this **varies so much by organisation that the guide will not generalise.** §9 explains why no figure is printed. | Variable pay is an instrument for something measurable. BD's output is measurable only late, which is why organisations reach for proxies — and §7 argues the proxies are contested. |
| **Failure mode** | Loses the deal. Visible, fast, survivable. | Builds a relationship that never produces revenue; or produces revenue that nobody can attribute to it. Slow, invisible until an audit, and it costs the BD person their credibility rather than the company a deal. 🔧 |

### 4.2 Where the boundary genuinely blurs — admitted, not smoothed over

A guide that stopped at §4.1 would be dishonest. The following are places where the distinction collapses in practice. 🔧 *All five are the guide's analysis of observed practice; they are marked as such.*

1. **The overlay and the deal-support role.** Where BD is attached to enterprise deals (variant D, §2.1), a BD person may be on a live opportunity with a close date, working the same CRM opportunity as a quota-carrying seller. Functionally that is sales with a different business card, and everybody in the room knows it.
2. **The partner-sourced deal.** When a partner brings a deal, the BD person who owns the partner *is* in the room, and the question "who sells it" becomes procedural. This is exactly where attribution disputes live (§7).
3. **The founder/early-stage case.** In a company too small to have functions, the same person does both, and the sequencing — discovery, relationship, contract, deployment, expansion — is one continuous motion. All the "BD versus sales" arguments are arguments about *large* organisations. 🔧
4. **The rebranded sales role.** §2.1 variant E is documented at an employer's own careers site: Google's **"Business Development Manager, New Business Sales"** ✅, with a business-cycle-ownership remit and consultative-sales minimum qualifications. **When the employer's own definition is a sales job, the distinction is not blurred — it is absent, by design.**
5. **The post-signature expansion motion.** Once a partnership is live, growing revenue from it looks like account management or sales and is frequently performed by whichever team is available. Where that hand-off happens is organisational, not conceptual.

### 4.3 Why the two functions so often misjudge each other

🔧 *This subsection is analysis with stated reasons, as required — not assertion. It is the guide's construction.*

**The asymmetry that produces the friction, stated precisely:** *BD is measured on relationships that have not yet produced revenue; sales is measured on a quarter that closes in nine weeks.* Both people are rational. Their disagreement is structural.

- **The instrument mismatch.** Sales has an instrument (the deal) that converts effort into a verifiable number within a period. BD has instruments that are either late (revenue) or soft (relationships, pipeline influence). When two functions are measured by instruments of different resolutions, each perceives the other as either "not accountable" or "short-term". 🔧
- **The time-horizon mismatch, with a real consequence for the BD person.** A sales person who is asked to invest a quarter of their time in a partnership with no near-term deal is being asked to damage their own measured performance. Their resistance is not ignorance; it is arithmetic. Stated the other way: **a BD person who asks for seller time without understanding the seller's quota clock is asking a rational person to act irrationally.** 🔧
- **The vocabulary mismatch.** "Pipeline" means two different things (§1.3): for sales, opportunities with close dates and probabilities; for BD, partner conversations with neither. When the two use the same word for different objects in the same meeting, the meeting cannot conclude. 🔧
- **The framing mismatch — the deep one.** Sales is trained to make a commitment the counterpart must respond to. BD's most valuable output in the first months is *information*, which requires not making a commitment. A sales-trained BD person will over-commit early (this is the source of §12's central friction and §14's first anti-pattern); a BD-trained person in a sales room will be experienced by the seller as vague and slow. 🔧
- **The incentive to overpromise is not symmetrical.** A partnership promise is cheap to make (the BD person's cost of promising is a conversation; the engineer's cost of delivering is a quarter), and its cost lands on a different team and a later date. Stated as a rule: **the person who makes the promise is rarely the person who pays for it.** 🔧 *This is the mechanism behind the technical-counterpart friction in §12.*
- **The organisational consequence.** Because the two functions cannot easily read each other's instruments, organisations resolve the tension by *merging them* — which produces the "BD" titles that are sales jobs (§2.1 E) and destroys the venue-building capacity the BD function existed to provide. 🔧 *The guide's inference.*

### 4.4 Where to read the qualification machinery — and what this guide does not do

Business development inherits *qualification* from sales but the two qualify different objects:
- **Sales qualifies an opportunity** — is this deal real, winnable, and forecastable? The methodology for that (MEDDPICC and its lineage, deal review, forecasting, failure modes) is owned by **[meddicc_guide.md](meddicc_guide.md)**, and the wider landscape of sales methodologies is owned by **[../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md)**. This guide does not re-derive either.
- **BD qualifies a partner** — should this relationship exist at all, will the counterpart organisation actually move, and can we support it? §6.2 sets out that question. **It is a different question, and the tools are not interchangeable: a partner that passes every deal-qualification criterion can still be a partnership that should never have been signed.** ✅ *Supported at the level of documented mechanism — the vendor programmes' own entry requirements (a marketplace listing, a validated status, minimum launched and qualified opportunity counts, a minimum recognised revenue figure ✅ AWS ISV Accelerate requirements) are a *partner*-qualification instrument that no deal-qualification framework supplies.*

**The thesis, restated as the section's conclusion:** *Sales works the venue; business development builds it.* The consequences follow from that sentence and not from any culture claim: different cycle, different unit of work, different cadence, different internal-selling load (§4.1). **And business development is not sales** — except in the specific, documented cases where an employer has decided the word means exactly that (§4.2.4).

---

## 5. The Motions and the Deal Types

A "motion" is the commercial route by which a partnership produces revenue. This section is mechanism-level. **Where a vendor's own programme documentation defines a motion, that definition is used and cited; where the guide assembles a comparison, it is marked 🔧.**

### 5.1 The seven motions, taken from how the programmes themselves describe them

| Motion | What it is for | What the partner gets | What usually goes wrong | Grade |
|---|---|---|---|---|
| **Referral** | Passing a lead from one organisation to another where the selling party is the one with the relationship. The cleanest, cheapest motion and the one most often mistaken for a partnership. | Lead flow; sometimes a fee or margin. One vendor's documentation is concrete: the partner publishes a business profile that is listed wherever the vendor's own sellers search for partners, and leads arrive in the partner portal with **required response time frames** attached. | Referral volume is a function of *the vendor's* internal sellers remembering to look. Both a formalised mechanism and a fragile one. | ✅ (Microsoft Learn referrals documentation, **last updated 28 May 2025**) |
| **Resell** | A partner transacts the product and sells it to the end customer, owning that customer relationship. | Margin, and control of the customer. | The vendor loses sight of the customer and the partner's incentives diverge from the vendor's roadmap. Distributor tiers add a layer that both sides resent. | ✅ (Microsoft's **Cloud Solution Provider** programme is described by Microsoft as enabling partners "to resell Microsoft cloud services to customers"; indirect resellers work through a distributor that **handles billing with Microsoft while the reseller owns the customer relationship** — Microsoft Learn, **last updated 29 May 2026**; AWS runs *Resell with AWS Distributors*, *AWS Solution Provider Program* and *Channel Partner Private Offers* ✅) |
| **Co-sell** | Two organisations pursue the **same** opportunity together — the partner and the vendor's own sellers, sharing pipeline visibility and often funding. | Access to the vendor's sales force, incentives, funding, and a visibility mechanism. | Requires that the vendor's own sellers care. Programme design is therefore all about incentives and scoring — AWS states that **AWS Sellers are compensated more to co-sell with ISV Accelerate Partners**, and that partners boost a **"co-sell recommendation score"** by sharing opportunities ✅. | ✅ (AWS's own definition: "Co-selling is a collaborative approach between AWS and Partners to deliver comprehensive cloud solutions" — AWS co-sell page, retrieved this pass) |
| **OEM / embedded** | The partner's product incorporates yours, sold under the partner's brand. | A product feature and a supply relationship; the consideration is often price and support, not margin. | **The word is genuinely ambiguous** and is routinely confused with original-design and original-brand manufacturing ✅. The failure mode is commercial rather than technical: you become an input cost to someone else's margin, and your roadmap becomes their roadmap. | ✅ ambiguity (reference work); 🔧 the failure mode |
| **Integration (integration-led partnership)** | The consideration is technical: an API-level connection that makes the joint offering usable in a segment where neither product alone is. | A capability they could not build, and access to your customers. | **This is the motion that consumes engineering time and produces the least revenue**, because the commercial case is usually assumed rather than built. §12 and §14 are largely about this. | 🔧 |
| **Joint venture** | A separate, jointly-owned entity — documented as two or more distinct firms combining a portion of their resources, formed for reasons including market access, scale efficiency, shared risk, or access to skills ✅. | Equity in a new entity, and the strategic control that comes with it. | The entity has its own governance, its own cost base and its own board. Exiting is a legal project, not a conversation. The reference work's own article on JVs carries multiple maintenance warnings ⚠. | ✅ definition; ⚠ source quality |
| **Strategic investment** | Minority equity placed to make a partnership possible or to secure priority. | Capital, and the strategic signal. | The investment becomes the reason the partnership survives after the commercial rationale has gone. 🔧 *The guide's observation; no source located that quantifies this.* | 🔧 |

**A note on what is *not* here.** 🔧 *The guide has deliberately not invented a "seven types of partnership" taxonomy and presented it as standard, because no retrieved source presents one as standard.* Where taxonomy exists it is **programme-specific** — AWS's Partner Paths (Software, Hardware, Services, Training, Distribution) and its programme catalogue (reseller, services, managed services, technology solutions, business outcome, public sector programmes) are a real, published ontology of motions ✅, and any organisation building its own should treat it as a *sample*, not a universal model.

### 5.2 The qualification thresholds are published, and they are a design choice

The most useful thing a practitioner can learn from a vendor's programme page is that **"qualified" is a number somebody chose.** AWS publishes the entry requirements for its ISV co-sell programme, and they are specific and countable: a product listed as generally available in the marketplace; eligibility in the opportunity-sharing programme; a validated or differentiated status; a payee account; **a minimum of five launched opportunities in the past twelve months**; **a minimum of fifteen qualified opportunities in the past twelve months**; at least one individual having completed a named e-learning module; and **recognised revenue of at least $2,000** at enrolment ✅ *(AWS ISV Accelerate requirements, retrieved this pass)*.

Two observations that follow directly and are documented rather than argued:
1. **The threshold is the incentive.** A partner who needs fifteen qualified opportunities in twelve months will generate fifteen records, and the vendor knows it — which is why the same programme also requires *launched* opportunities, i.e. records that reached a next state. ✅ *The dual threshold is itself the anti-gaming design.*
2. **Comparability between programmes is illusory.** Two partners each reporting "twenty qualified opportunities" may be operating to different definitions. Ask for the definition before the number. 🔧 *The advice is the guide's; the variability is documented.*

### 5.3 What the vendor publishes about the results — and how to read it

Vendor pages carry outcome statistics, and they are of two kinds, which must be separated:

- **Commissioned analyst findings, published by the vendor.** AWS's co-sell page states that **eighty percent of partners identify AWS Marketplace as integral to their co-sell strategy**, citing a **Canalys** report ✅, and its ISV programme page states that **51% of partners report higher average revenue growth as a result of co-sell motions** and **65% close deals faster** ✅, again citing the Canalys reprint it hosts. **Grade: ✅ that the vendor published these figures with that attribution; ⚠ as evidence**, because the study is a survey commissioned in the context of the vendor's programme, the underlying methodology and sample are not on the page, and the vendor is a party to the result. **This guide does not use any of these numbers as a fact about partnerships in general.**
- **The firm's own services-revenue-per-dollar claim.** AWS's programmes page headlines a figure for services revenue generated by partners per US$1 of AWS technology sold, and a percentage of partners delivering AI as part of AWS transformation delivery ✅ *(as published; the underlying Omdia whitepaper is linked from the same page)*. **Grade: ⚠ vendor-published, analyst-sourced, not independently verified in this pass.**

**The rule this guide applies from here on:** a number that appears on the page of a party to the transaction is evidence *that the party says so* ✅ and is **not** evidence of the underlying fact ⚠. Every figure in §7 is treated this way.

---

## 6. The Partner Lifecycle

### 6.1 Two published lifecycles, and why both are used here

There is a genuine, documented professional lifecycle for this work — and it belongs to the *alliance* profession rather than to "BD" (the §2 gap again).

- **ASAP's seven-phase alliance management lifecycle**, published on its own site as the frame for its practice: **Alliance-Specific Strategy → Analysis and Selection → Building Trust and Value-Creating Negotiations → Operational Planning → Alliance Structuring and Governance → Launching and Managing → Transform, Innovate, or Exit Gracefully** ✅ *(ASAP site, retrieved this pass)*.
- **ASAP's three-state alliance life cycle**, published on the page for its own book: **startup, steady state, and wind-down** ✅, alongside the book's own named concept — **Value Inflection Points (VIPs)**, defined by that publisher as "moments when what happens next can significantly affect the value of the partnership" ✅, with an entire chapter devoted to **the first dispute** as a VIP ✅. The book's authors are named with their titles, and one of them — an Eli Lilly senior director — carries the title **"Alliance Management and Corporate Business Development"** ✅, which is a small but direct piece of evidence for the §2.1 B/C adjacency.
- **The reference work's alliance life cycle**, for general strategic alliances: **analysis and selection → formation → operation → alliance structuring and governance → end/development** ✅, with separate sections on common mistakes, success factors and risks ✅ *(the article carries a list-format maintenance tag ⚠)*.

**Why both are used:** the seven-phase form is a *process* view and is the more useful checklist; the three-state form is a *temperature* view and is the more useful diagnostic — a partnership is either starting, running, or ending, and mistaking which state you are in is itself a failure mode. 🔧 *The "process versus temperature" framing is the guide's.*

### 6.2 The eight stages as this guide runs them, and where arrangements most often die

The stage names below are the guide's 🔧; where a stage maps to a published phase, the mapping is noted.

**1. Identify.** Find organisations whose strategy makes overlap plausible. Sources: your ecosystem map, your own sellers' encounters with partners, partner-programme directories, conferences, and — in practice most often — someone's existing relationship. *Maps to ASAP's "Alliance-Specific Strategy" ✅ in the sense that the search should follow a stated strategy rather than precede it.* **Where it dies:** never; identification is easy and is the stage organisations over-invest in, because it feels like progress. 🔧

**2. Qualify.** The BD-specific qualification question (§4.4), and it is not a deal-qualification question. 🔧 *The guide's four-part test, offered as a construction:*
   - **Does the counterpart have a reason that survives our departure?** A partnership that exists because one enthusiastic person on their side likes your team will not survive that person's promotion.
   - **Will they commit a named person?** Not a name in a slide — a name in a calendar invitation.
   - **Can we support them?** Enablement, documentation, support hours, and a route for their escalations. A partner who cannot be supported is a future complaint.
   - **Does this scale, or is it one deal in disguise?** If the honest answer is "it is one deal", it is a deal and should be run as one.
   **Where it dies:** most often here, and this is the *right* place for arrangements to die. **The documented finding that supports this:** when BDO asked alliance professionals what contributed to alliance underperformance, the top three answers were **lack of internal alignment within at least one partner; failure to understand differences in goals and priorities between partners; and lack of sufficiently robust joint governance** — while the *lowest*-reported contributor was **"selected the wrong partner"** ✅ *(BDO, "The State of Alliance Management," based on data gathered in 2021 and 2022 from 183 alliance and account management professionals across 11 industries; BDO is a professional-services firm that also sells alliance advisory work ⚠)*. **Read that carefully: professionals do not blame partner selection, they blame alignment and governance** — which means the failures that look like stage-2 failures are usually stage-5 and stage-7 failures arriving late.

**3. Pilot.** A bounded piece of the relationship, deliberately small, whose purpose is to test whether the counterpart organisation can actually move. *Not a published stage in either lifecycle — this is the guide's insertion 🔧, and the justification is that both lifecycles presume a decision to proceed, while in vendor practice the pilot is how the decision is de-risked.* **Where it dies:** the pilot that has no exit criterion. A pilot with no defined end either becomes a permanent unowned workload (§12.4) or is quietly abandoned with the relationship still nominally alive (§6.3).

**4. Negotiate.** Commercial terms, and — from ASAP's own book coverage — **contract design** and the handling of **the first dispute** ✅. The coopetition problem (§3.1.4) is negotiated here: exclusivity, data, competitive independence, most-favoured terms, termination. **Where it dies:** in the gap between what the business developer agreed in principle and what legal will accept as paper. 🔧 *This gap is the single most common source of the technical-counterpart problem in §12, because it is where "we'll figure it out" becomes a signed obligation.*

**5. Launch.** *The published phase is "Launching and Managing" ✅.* **Where it dies:** almost always from **enablement failure**, and the evidence is structural rather than statistical: a partner who has signed but whose sellers have not been trained, whose support route is unknown and whose press release has already gone out has been *launched as a marketing event* rather than as an operating capability. 🔧 *For the post-launch discipline itself — adoption, escalation, the operating cadence, what to do when the partner goes quiet — this guide does not re-derive it: the owner is [post_sales_customer_facing_experience_guide.md](post_sales_customer_facing_experience_guide.md), and the reader should go there.*

**6. Scale.** The partner sells the thing repeatedly, to customers you did not introduce. **Where it dies:** it usually does not die, it *stalls* — a handful of joint deals per year, achieved by two individuals who happen to get along, which is a friendship with an invoice rather than a channel. 🔧

**7. Govern.** *Both published frameworks have this stage ✅ (ASAP: "Alliance Structuring and Governance" and its book's three governance chapters; the reference work: "alliance structuring and governance").* Joint steering, escalation, review cadence, renewals of the arrangement itself. **Where it dies:** the governance forum that becomes a status meeting, and the partner with **no owner** on the other side. **Cross-reference for the *buy-side* mirror of this problem** — how an enterprise governs a supplier relationship, and how it exits one — is [vendor_management_guide.md](vendor_management_guide.md); §12.4 states the practitioner-side consequence and does not repeat the governance mechanics.

**8. Exit.** *Published and explicitly named by ASAP: "Transform, Innovate, or Exit Gracefully" ✅; the reference work: "end/development" ✅.* **Where it dies:** it does not die — it is simply never performed, and the relationship persists as an unattended cost until somebody audits the partner list. This is anti-pattern 6 in §14.

### 6.3 Where arrangements most often *actually* die

Three failure shapes recur across the stages, and this guide states them as diagnosis rather than as measured rates:

1. **The partner that signs and never sells.** 🔧 *Cause: the signature was the objective on at least one side, and neither side built enablement, incentive or a first joint account.* The BD person's own measure — a signed partner — was achieved, which is precisely the measurement problem §7 examines.
2. **The launch with no enablement.** 🔧 *Cause: the announcement date was set by marketing before the capability existed.* Corollary: the technical counterpart learns of the commitment from the press release (§12.4).
3. **The relationship with no owner after signature.** 🔧 *Cause: the BD role that created the partnership ends at signature unless the organisation explicitly assigns stewardship. The professional literature treats "Launching and Managing" as a phase for exactly this reason ✅, and BDO's respondents ranked weak joint governance among the top three causes of underperformance ✅ — which is the closest thing to evidence this guide found for the pattern. It is not a measurement.*

**No failure rate is printed here.** §7.5 explains why, and §16 records the specific widely-repeated figure this guide refused to use.

---

## 7. The Metrics and the Attribution Problem

**Read this section as an anatomy of an unresolved problem, not as a solution.** The task the section is given — state the metrics problem honestly and do not solve it with an invented metric — is the honest task, because the problem is not solved in practice.

### 7.1 Why a quota fits sales and fits BD badly

Sales quota works because of four properties the sales motion has and the BD motion lacks. 🔧 *The analysis is the guide's; the definition of the pipeline it relies on is documented.*

| Property a quota needs | Sales has it | BD has it? |
|---|---|---|
| **A countable unit whose existence is verifiable** | The opportunity, then the closed deal. | No. The unit of BD work is a relationship and a mechanism, neither of which is countable at the moment it is created. |
| **An attribution rule that is not seriously disputed** | The seller closed it. | No — see §7.3. |
| **A period short enough to be actionable** | The quarter. | No. Partnership cycles run multi-quarter to multi-year, so a quarterly quota forces either under-performance or fabrication. |
| **Control over the proximate cause** | The seller controls the conversations that produce the close. | No. The BD person does not control the partner's sellers, the partner's priorities, or their own engineering team's roadmap. |

**The documented illustration of how different the instruments are.** BDO's survey reports that **alliances have driven on average one-third of company revenue over the past five years**, that the share ranged **from 27% to 36% by industry** among its most-represented industries, and that **80% of companies expect to increase multilateral alliances** ✅ *(BDO, 2024 publication, data gathered 2021–2022 from 183 professionals across 11 industries)*. **Note what this metric is and is not:** it is a *share of revenue attributed by surveyed professionals to alliances* — a self-reported attribution, from a sample drawn from a professional-services firm's own network, published by a firm that sells alliance advisory services ⚠. It is **not** a measurement of any individual's or function's performance, and it could not be used as a quota. **This guide prints it because the publisher and method are stated, and it flags it because the method has limits.** 🔧 *The critique is the guide's.*

### 7.2 What BD functions are actually measured on — only where a source says so

This subsection is deliberately short because **the honest answer is that no retrieved source establishes a standard.** What can be said:

- **ASAP, the professional body for the adjacent alliance profession, teaches measurement as a discipline rather than publishing a metric.** Its own book coverage is described as addressing governance, contract design, disputes, value inflection points and international alliances, and the body runs programming on the topic ✅ *(ASAP site, retrieved this pass)*. Its *Handbook* is the profession's reference work ✅. **A profession that has a handbook and certification and still treats "demonstrating the value of an alliance" as a taught problem is telling you the measurement question is not settled.** 🔧 *That inference is the guide's.*
- **McKinsey published a piece titled "Measuring alliance performance"** whose reported position is that measurement **should go well beyond cash-flow metrics** to include transfer-pricing benefits, benefits outside the deal's scope (for instance sales of related products), the value of options the alliance creates, and start-up and ongoing management costs ⚠ *(read from the search-result description of McKinsey's own page — **the page itself failed to extract in this pass**, so neither its date nor its full content was verified; cite with caution and see §16)*. **What this supports, and only this:** a management consultancy treats alliance measurement as a multi-line problem rather than a single number ⚠.
- **Vendor programmes measure partner contribution procedurally rather than financially.** AWS's co-sell machinery is built on *sharing opportunities into a common pipeline and having that sharing affect a visibility/recommendation score* ✅, and its programme entry rules count *launched* and *qualified* opportunities over a rolling twelve months ✅. **That is a real, published, operational answer to "how do you measure a partner" — and note that it measures activity and pipeline participation, not realised revenue.** ✅/🔧.
- **What the guide could not find, and therefore does not assert:** no retrieved source establishes how *business-development functions* (as distinct from alliance-management functions or partner programmes) are measured, and none establishes a typical mix of relationship objectives, pipeline-influence measures and revenue targets. ⛔ **Recorded in §16.**

### 7.3 The attribution dispute over a partner-sourced deal

The mechanism of the dispute, set out plainly: a deal closes. Four claims can be made about it, all defensible, and they conflict. 🔧 *The scenario is constructed; the vocabulary is industry usage.*

1. **The sales person's claim:** they ran the opportunity, the pricing, the legal review and the close. Without them there is no revenue.
2. **The BD person's claim:** the partner introduced the buyer; without the partnership the buyer had no route to the vendor, and the whole segment was unreachable.
3. **The partner's claim:** it is *their* customer, and the vendor's direct contact is arguably a channel-conflict problem.
4. **The finance function's claim:** none of the above is attributable, because revenue would likely have arrived by another route.

**The industry's vocabulary for the contested boundary is real and is used by the vendors of partner-management software:** the distinction between **partner-sourced** revenue (a deal the partner originated) and **partner-influenced** revenue (a deal the partner touched but did not originate), with the widely-published warning that programmes **blend the two into a single attributed total**, that **minor touches get counted as meaningful influence**, and that **the same deal can appear in more than one total** ⚠ — *sourced from vendor content marketing published by firms that sell partner-attribution and partner-programme software (multiple such pages were returned this pass); this is ⚠ industry usage documented by interested parties, not an independent finding. Note the grade honestly: the guide treats the **existence of the distinction** as well established by usage, and the **magnitude of the problem** as unverified.*

**The one hard fact the guide can add:** where a vendor runs a co-sell programme, **the attribution is procedural, not analytical** — the opportunity is either registered in the programme's pipeline or it is not, and the programme's own thresholds decide what counts ✅ (AWS's ACE/ISV Accelerate rules). *Which means the answer to "was this partner-sourced?" is frequently "whoever registered it in the system first."* 🔧

### 7.4 The counterfactual problem in long-cycle relationship work

This is the deeper reason BD measurement fails, and it is a genuine methodological problem rather than an organisational one. 🔧 *The framing is the guide's.*

To evaluate a partnership you must compare what happened to what would have happened without it. **Nobody has the counterfactual.** Specifically:
- The partner's introductions may have reached customers who would have found you anyway — or customers who would never have called. Both are plausible, and no data on the page distinguishes them.
- A partnership that produces no revenue may have *prevented* a competitor's partnership, which is a real benefit that appears in no column. Likewise, a partnership that produces revenue may have done so at a margin that a direct sale would have beaten.
- The relevant test for a long-cycle relationship is not "did this produce revenue this period" but "is the *mechanism* working", and the mechanism's health is not in the ledger at all.

**Consequence, stated as the section's analytical conclusion:** *a metric applied to long-cycle relationship work will be either late, partial, or counterfactual-dependent — and organisations resolve that tension by choosing a proxy and then forgetting it is a proxy.* §14's third anti-pattern is what happens next.

### 7.5 What this guide will not do, and why no failure rate is printed here

**This guide does not propose a metric.** An invented metric — "partner-sourced pipeline per active partner", "ecosystem coverage score", "partnership ROI index" — would read authoritatively and would have no evidence behind it. 🔧 *Relabelled as the guide's construction it would still be worthless to a reader who then used it, because the reader would be measuring their business against a number this guide made up.* **So it prints none.**

**On partnership failure rates — the classic widely-repeated figure.** This guide **does not repeat an unattributed "most partnerships fail" statistic**, because the number circulates without a consistent denominator (fail *how* — commercially? operationally? dissolved? underperforming against an unrealistic objective?), and the denominators differ across the sources that do exist. **What can be attributed properly is this:** BDO's own survey presents a chart headed **"More than half of partnerships fail to fully achieve their objectives"** and states that **alliance failure rates from 1996 to the time of writing have remained relatively constant**, finding that firms *expecting* alliance revenue to rise had a *higher* failure rate than those expecting it to fall (which the authors interpret as higher-usage organisations being more willing to experiment), that failures are no more likely in alliances than in internal R&D once organisational maturity is controlled for, and that the **highest failure rates occurred in platform and ecosystem partnerships** ✅ *(BDO, "The State of Alliance Management," 2024 publication; data gathered 2021–2022; 183 alliance and account management professionals across 11 industries)*. **Read that with three flags:** the source is a consultancy selling alliance services ⚠; the respondents are alliance professionals drawn from that firm's network, so a survivorship and selection bias toward organisations that already run alliances professionally is likely ⚠; and **"fail to fully achieve their objectives" is not the same as "fail"** — the headline is a measurement of ambition shortfall, and the guide repeats it as exactly that and no more.

**On compensation — no figure is printed anywhere in this guide.** See §9.4.

### 7.6 Four questions to ask before accepting anyone's BD metric

🔧 *The guide's construction. These are questions, not a metric, and the difference matters — a question cannot be mistaken for a measurement.*

1. **What is the denominator?** "Partner-sourced revenue" is meaningless without stating what total it is a share of, over what period, for what cohort of partners.
2. **What does a false positive look like?** If the metric can be improved by relabelling a deal, it will be.
3. **Who bears the cost when it is wrong?** If the answer is "a different team, next quarter", the metric is generating a promise that somebody else will pay for (§4.3).
4. **What would make us abandon this partnership?** If nothing appears in answer to that question, the metric is not governing anything — the relationship is already permanent, and §6.1's exit phase has been quietly deleted.

**The honest conclusion of §7:** **measurement in this field is contested and practice varies.** ✅ *That is what the sources actually support — a professional body that teaches value measurement without standardising it, a consultancy that measures alliance revenue shares by survey while flagging failure, a vendor that measures partner contribution procedurally, and an attribution vocabulary owned by the sellers of attribution software.* Any statement that "BD is measured on X" is a statement about one organisation.

---

## 8. The Skills, and What Separates Good BD from Bad

### 8.1 The documented skills first

Two sources define a skill set for this work, and they disagree in an informative way.

**The general business-development skill mixture** ✅ — the reference work states that skill sets and experience for business-development specialists "usually consist of a mixture of the following (depending on the business requirements)": **sales; finance; marketing; mergers and acquisitions; legal; strategic management; proposal management or capture management; cultural agility** — and that BD professionals frequently have earlier experience in **sales, financial services, investment banking or management consulting**, with some arriving via operations management ✅. *Note the inclusion of legal, M&A and finance: the reference work's list is wide enough to cover the corporate-development variant of the title (§2.1 C), which is itself evidence for the title problem.*

**The alliance profession's skill set, as a professional body defines it** ✅ — ASAP describes the professionals it serves as "new, mid-career, and senior/executive-level alliance and partnership professionals", explicitly including "people doing alliance work — whether or not 'alliance' is in their title", and offers two certifications, **CA-AM** and **CSAP** ✅, with the **CA-AM** track associated with a programme now titled *Applied Alliance Management* ✅. The body's own book coverage names the competencies it considers central: **governance structure and practice, contract design, dispute handling, value inflection points, and international alliances** ✅.

**Corporate development's documented credentials** ✅ — the reference work states that corporate-development executives focused on product or financial issues often hold **MBA, CFA or CPA** credentials, that advanced technical degrees are sought, and that teams contract or hire contributors from **legal or investment-banking backgrounds** ✅. *Grade note: this sits in an article carrying a long-standing citations warning ⚠.*

**The documented gap.** Neither source publishes a validated competency framework with assessment criteria. ASAP certifies its members ✅; the reference work lists skill areas ✅; **no retrieved source provides a task-level competency model for "business development" as such, and this is consistent with §2's finding that the title is a family rather than a profession** ⛔ *for the framework itself*.

### 8.2 The differentiators — the guide's analysis, plainly labelled

🔧 *Everything in 8.2 and 8.3 is the guide's construction: the analysis of what distinguishes good from bad practice in a job whose outputs are late and whose measures are contested. It is offered as reasoning, with the reasons given, not as findings.*

**1. Patience, and the ability to distinguish patience from inertia.** Good BD work is unrewarded for long stretches. The skill is not endurance; it is the ability to keep investing in a relationship whose evidence of progress is currently qualitative *while simultaneously* being willing to stop. The two halves are usually split between two different people — which is why organisations pair a relationship-holder with a sceptic when they run the function well. 🔧

**2. Internal credibility — the actual scarce resource.** Because [§3.1](§3) established that most of the work is internal, the practitioner's productivity is bounded not by the number of external meetings they can take but by **how many internal commitments they can obtain**. Credibility is earned by making a small number of promises and keeping them, and it is spent by making promises on behalf of engineering. **Stated as the guide's rule: a BD practitioner's most valuable asset is a track record of asking for less than they get, and their fastest route to oblivion is asking for more than can be delivered.** 🔧 *This is the direct bridge to §12.*

**3. The ability to say no to a partnership that will not scale.** This is the skill that is least taught and most decisive, and it has an uncomfortable property: **saying no is indistinguishable from saying yes in any monthly report, because both produce zero.** The evidence available supports the direction: BDO's respondents ranked **"selected the wrong partner"** as the *lowest* contributor to underperformance ✅ — which cuts both ways, and the guide reads it carefully. Either partner selection really is rarely the problem, **or** professionals systematically do not record their own selection errors. 🔧 *The guide leans to the second reading, and states the reason: a function that never declines a partnership has no way to generate the evidence that declining matters.* **What is documented is only that a professional body runs programming on internal cohesion ✅ and a consultancy's survey ranks internal alignment and weak governance above partner choice ✅. The claim that *declining* is the decisive skill is the guide's, and it is unsourced.**

**4. The failure of treating BD as a numbers game.** 🔧 The pattern: an organisation sets a partner-recruitment target ("sign twelve partners this year"), the function signs twelve, the revenue does not follow, and the function is then judged to have failed at the only thing it was told to do. **Three reasons given, because the claim needs them:** (i) partnership value is a function of mechanism quality, not partner count; (ii) a large partner roster multiplies support and enablement obligations linearly while revenue grows sub-linearly; (iii) the recruit-to-target incentive actively produces the "signs and never sells" partner of §6.3. **The consequence for the practitioner:** a recruitment target is a *measurable* objective, and measurable objectives displace immeasurable ones — that is §7.5's proxy problem in person.

**5. The network compounds.** 🔧 A BD practitioner's value grows through a network whose nodes are other professionals who move between organisations. A conversation with a partner contact is worth more in three years, when that person runs a different company's partnerships, than it is today. **Two consequences:** (i) short-term extraction from a relationship destroys a multi-year asset, which is why the practitioners who survive are the ones who do not treat counterparties instrumentally; (ii) the practitioner's market value is partly *their book of relationships*, which is portable in a way that a quota-attainment record is not — and which organisations recognise by treating BD hiring as a network-hiring exercise. 🔧 *Note honestly: this is the guide's analysis of a widely-described career pattern; no retrieved source measures network value.*

### 8.3 What the skills literature implies you should look for when hiring

🔧 *The guide's construction, derived from 8.1's documented skills plus 8.2's analysis.*

- **For a hunting role (variant A) and an alliance role (variant B), the interview questions are different, and asking the wrong ones is the most common hiring error.** A hunting role needs evidence of *origination under rejection*. An alliance role needs evidence of *holding a relationship through a dispute* — and the profession's own literature names **the first dispute** as a defining moment ✅, which suggests a good question: *describe a partnership that went wrong, what you did, and what the other side did.*
- **Ask for a partnership that was declined, and why.** §8.2(3) argues this is the decisive skill; the question is the only way to test it.
- **Ask what they promised internally that they could not deliver, and what they did next.** §4.3's promise-accountability asymmetry means this question probes the exact failure mode that damages engineering relationships (§12).
- **Do not test for product knowledge and call it BD aptitude.** The documented competency lists include finance, legal, strategic management and cultural agility ✅ — only one of the eight is "sales".

---

## 9. The Ladder, the Market and the Routes In

### 9.1 The levels, as organisations describe them

**Honest statement first: no retrieved source publishes a levelling framework for business development.** ⛔ *What exists is (a) a professional body's level description for the adjacent alliance profession, and (b) employer practice visible in titles.* Both are given, and neither is presented as a standard.

- **The professional body's level description** ✅ — ASAP states it serves "new, mid-career, and senior/executive-level alliance and partnership professionals", and that it has been advancing the profession for **28 years** as of its own 2026 content, having begun in **1998** ✅ *(ASAP site, retrieved this pass)*. **Read that carefully:** the profession's own level language is *three coarse bands*, and its currency is **certification (CA-AM, CSAP) and a handbook** rather than a job architecture. ✅
- **Employer practice, from titles observed in this pass** ⚠ — the pattern visible across the postings and pages retrieved is a three-to-four rung ladder of *Manager → Senior Manager → Director → Head/VP*, with **"Corporate Development & Investor Relations"** appearing as a single combined function at a growth-stage fintech ⚠ *(Airwallex posting, read from the search-result description of the employer's own careers page; the page did not extract)*, and with **"VP, Global Head of Alliance Management"** and **"Senior Director, Alliance Management and Corporate Business Development"** as titles held by the named authors of the alliance profession's own book ✅ *(author biographies published by ASAP)*. **That last pair is a genuinely useful data point: in large pharma, alliance management and corporate business development appear in one title at senior-director level, and alliance management reaches VP — which is evidence that this work has a real senior ceiling where the organisation takes it seriously.** ✅
- **The guide's inference, labelled:** 🔧 in organisations that treat partnerships as a channel, the BD ladder tops out below the sales ladder; in organisations whose business model *is* the ecosystem, the partner/alliance function reaches the executive level. The second half of that claim is supported by the existence of a "Chief Partner Officer" title published on the professional body's own site ✅ *(ASAP quotes an executive with that title)*; the first half is the guide's inference and is unsourced.

### 9.2 The routes in — documented

✅ The reference work states that business-development professionals **frequently have earlier experience in sales, financial services, investment banking or management consulting, and delivery**, while **some find their route in by climbing the corporate ladder in functions such as operations management** ✅. For the corporate-development variant specifically, it adds **legal or investment-banking backgrounds** and **MBA/CFA/CPA credentials** ✅ ⚠ *(article-level sourcing caveat)*.

**The route table, with the transition the guide would expect from each.** 🔧 *The "what transfers / what does not" column is the guide's analysis; the routes themselves are documented.*

| Route in | What transfers | What does not transfer |
|---|---|---|
| **From sales** | Deal instinct, commercial vocabulary, the ability to hold a conversation with a buyer. | The instinct to close. Partner discovery requires *not* asking for commitment (§3.1.3), and a sales-trained BD person over-commits (§4.3) — the most common documented-by-practice failure. |
| **From consulting / advisory** | Structured problem-solving, executive communication, the ability to write a case. | The engagement mindset. Advisory work has a defined scope and an end date; a partnership has neither. **See [consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) §5 for advisory-versus-delivery, which this guide does not re-derive — and note the difference that matters here: an adviser can leave, and a partner cannot.** ✅ *Accessible to the guide by name rather than re-derived from that guide's content.* |
| **From product** | Roadmap literacy, feasibility judgement, credibility with engineering. | The tolerance for a negotiation you do not control the terms of. |
| **From a technical / architecture role** | The highest-value transfer of all, in this guide's view 🔧: feasibility judgement, the ability to say "yes, but differently", and instant credibility in a joint architecture conversation. | Patience with the commercial pace, and the willingness to be measured on something late and soft. |
| **From an operator background** | Domain credibility — you know the partner's problem because you have run it. | Network. The operator route starts with no relationships, which for a network-compounding role (§8.2.5) is the longest ramp. |
| **Via operations management** ✅ | Process discipline, cross-functional navigation — the closest analogue to §3's internal-advocacy load. | External origination. |

### 9.3 The market — what is documented, and what is not

- ✅ **Documented:** alliances are described as "mainstream", contributing around a third of revenue on average across the surveyed organisations, with expected growth in contribution and in multilateral alliances, and with technology companies expecting most growth from **ecosystem and reseller partners** ✅ *(BDO, 2024 publication; 2021–2022 data; 183 respondents; consulting firm as publisher ⚠)*. **This is the strongest market-context evidence retrieved in this pass, and it is a survey of a consultancies' network, not a market study.**
- ⚠ **Flagged:** the professional body's homepage headlines four figures — that **1/3 of company revenue across industries is driven by alliances**; that **$100T in global economic output could come from ecosystem-driven businesses by 2030**; that **70%+ revenue growth is projected from partnerships over the next five years**; and that **94% of organizations believe their partner ecosystem will drive future growth**. ✅ *All four are verified as published by that body on its own site in this pass; ⚠ none carries a cited source, sample or methodology on the page, and the body's purpose is to advance the practice it is describing.* **The guide therefore reports them as the body's own claims and uses none of them as a fact.**
- ⛔ **Not established:** market size for a "business development services" or "partnership management software" category; headcount growth or shrinkage in BD roles anywhere; any regional BD-job-volume figure. The searches attempted for market data returned vendor content and no primary source, and `bls.gov` did not load.

### 9.4 Compensation — the absence, recorded as the finding

**No compensation figure appears anywhere in this guide, and that is a deliberate result, not an omission.** The reasoning, stated in full because the reasoning *is* the finding:

1. **The only sources located in this pass that publish BD pay figures are recruitment and salary-survey content**, typically search-optimised pages from staffing firms and career sites. **Those are marketing documents.** Their purpose is to attract candidates and clients; their samples are self-selected; their figures are frequently unexplained as to geography, seniority, base-versus-total, currency, and date. **This guide will not launder a recruitment firm's marketing page into a salary figure.** ⚠
2. **A defensible instrument does exist, and it is named.** Singapore's Ministry of Manpower publishes an **Occupational Wage Survey** whose 2025 tables were released **30 June 2026**; the page states the coverage and method on its own terms — data pertain to **full-time resident employees**, are given as **basic and gross wages excluding bonuses**, are compiled from a survey of **private-sector establishments with at least 25 employees** supplemented by administrative records, cover **over 500 occupations**, and the statistical department's data-collection processes have been **assessed by Ernst & Young Advisory Pte Ltd** ✅ *(MOM Labour Market Statistics and Publications, retrieved this pass)*. A companion file, "**List of Occupations and Industries for which Wage Data are Published**," is published as a spreadsheet alongside the tables ✅.
3. **But the load-bearing step was not taken in this pass.** This guide did **not** download and open that occupation list, and therefore **cannot state whether "business development" appears in it as an occupation with published wages, or under what classification it might sit.** ⛔ *Consequence: even the defensible instrument yields no figure here, because the guide did not verify that the occupation is covered.* **This is recorded as a research gap in §16 and as the honest reason no number is printed.**
4. **Why this matters more than it looks.** §2 established that the title covers at least five different jobs. A BD "salary" would therefore be an average across a channel recruiter, an alliance manager, an M&A analyst and a quota-carrying advertiser — a number whose meaning is nil. **The absence is arguably the correct output: the most useful statement a guide can make about BD pay is that the title is too unstable for a single figure to mean anything.** 🔧

**What a reader should do instead:** take the occupation, not the title, to a named national wage instrument in their own jurisdiction, check that the instrument's coverage statement includes the occupation, and ask for the *level*. That is a procedure, not a number, and it is the only responsible thing this guide can offer. 🔧

---

## 10. The Singapore and Asia Context

**What follows is small on purpose.** This guide could not find Asia-specific research on the business-development or alliance profession, and it says so rather than padding the section with regional assertion.

### 10.1 What is documented about the region's commercial geography

- **The regional-headquarters pattern.** ✅ A **Bloomberg Intelligence** report **published 21 February 2024** is reported to have found that **Singapore hosted regional headquarters for 4,200 multinational firms in 2023**, against **1,336 in Hong Kong**; and to have attributed the shift to Singapore's relations with the West, a broader talent pool, a diversified economy and targeted tax incentives, while noting that **Hong Kong's standard corporate tax rate is 16.5% while Singapore's 17% can be reduced to 13.5% or less through programmes for some activities** ✅ **as reported** *(source: Channel NewsAsia's report of the Bloomberg Intelligence findings, published 22 February 2024, retrieved this pass; the underlying Bloomberg Intelligence report itself was not read, and the same figures were reported across multiple outlets — the guide's grade is ✅ for "these figures were published by these outlets on that date" and ⚠ for the figures themselves).*
- **The same report names corporate examples of the pattern** — including technology, logistics and consumer names — **and this guide deliberately does not repeat them as evidence about anything.** They are reported examples of headquarters location; **no company named in that report is asserted here to be a partner, a buyer, a competitor or a player in anything.** That refusal is deliberate: headquarters location is a fact about real estate and tax, and it says nothing about a firm's partnership behaviour.
- **What the regional-HQ pattern *does* imply for the discipline, marked as inference.** 🔧 If a region hosts a large concentration of multinational regional headquarters, then the *partner-facing* counterparties in that region are disproportionately regional offices of global organisations — which changes the work in two ways the guide can only reason about: (i) the decision authority for a partnership may sit in another continent while the relationship is run locally, so the practitioner's internal-selling problem (§3.1.5) exists *twice*, once in their own firm and once in the counterpart's; and (ii) regional partnership staff are often the first to be cut and the last to be consulted when a global programme changes. **Both are the guide's reasoning 🔧 and neither is sourced.**
- **Talent and language:** the practitioner's daily working language in the region's business districts is English, and cross-border partnership work routinely requires a second language for the partner's local teams. ⚠ *Practitioner-consensus observation; the guide verified no source for it, and it is offered as such.* (A single corroborating detail from an employer's own posting: Google's Shenzhen-based business-development posting for new business sales **requires business fluency in English and Mandarin** ✅ — one data point about what one employer asks for in one market, and no more.)

### 10.2 Sector patterns — what one source actually says

The only sector-level finding this guide could source is from BDO's alliance survey, and it is a global survey rather than an Asian one. Reported as such:

- **Technology companies expect most revenue growth to come from ecosystem and reseller partners**, with partnership marketplaces coalescing around core systems in some software categories, while **reselling relationships between technology providers and the firms that advise customers on systems architecture remain important** ✅.
- **Financial-services firms expect customer partnerships to be their most significant revenue driver**, with banks working closely with traditional and new suppliers, and commercial lenders working with lending customers and a supply chain of systems and data providers to build risk tools ✅.
- **Life sciences respondents** were most likely to expect R&D partnerships to drive revenue (over 50%), with platform partnerships also high (44%), and healthcare respondents rating ecosystem alliances and supplier/reseller partnerships as significant ✅.
- Life sciences was the industry with the **lowest** share of revenue from alliances in the surveyed set **and** the **highest** expected increase over the next five years, as well as the **lowest** prevalence of multilateral alliances (19%, against healthcare's 42%) ✅.
- *All of the above: BDO, "The State of Alliance Management," 2024 publication, data gathered 2021–2022, 183 respondents across 11 industries, published by a firm that sells alliance advisory services ⚠. **Note also that the survey's most-represented industries are life sciences, healthcare, financial services and technology — so the sector findings above are the survey's own centre of gravity, not a global sample.*** ✅

- **Logistics, and Asia-specific sector patterns: not established.** ⛔ No source was retrieved this pass on partnership behaviour in Asian logistics, on the region's channel/distribution structures, or on the supply-chain functions that would make the comparison possible. **Recorded in §16.**

### 10.3 What could not be established about Asia, stated plainly

- ⛔ **No Asia-specific or Singapore-specific research on the business-development or alliance profession** was located. The professional body for alliance management is global, and its published events in this pass were in Boston, London and online — **its programming calendar as retrieved shows no Asia event** ⚠ *(observation about a snapshot of one organisation's events page, not a claim about its coverage).*
- ⛔ **No regional BD-job-volume, hiring-trend or compensation data**, and — as §9.4 records — the one defensible Asian wage instrument located (**Singapore MOM's Occupational Wage Survey 2025**, methodology stated ✅) was **not opened to the occupation level**, so the guide cannot say whether BD is covered by it.
- ⛔ **No Asian professional body, certification or competency standard for business development** was located, distinct from the alliance-management body.
- **The honest summary of §10:** the region's *commercial* context is documented (headquarters concentration, tax competition, sector expectations from one global survey); the region's *BD profession* is not documented at all in anything this guide could read. **A reader in Singapore should treat this section as a statement of absence and use the general sections, not as regional guidance.** 🔧

---

## 11. The AI-Era Effect on the Role

**Structure of this section, as required: documented fact first, then vendor claim, then the guide's own reasoning — separated, never averaged.**

### 11.1 Documented fact

- ✅ **The alliance profession's own body has deployed an AI assistant.** ASAP's site, retrieved this pass, presents **"Ally: ASAP's AI assistant for alliance pros"** as a live offering ✅. **What this establishes, precisely:** a professional body for this discipline judged an AI assistant for its members worth building and announcing. **What it does not establish:** any effect on how alliance work is performed, any productivity figure, or any headcount effect. The guide asserts none of those.
- ✅ **The profession is treating AI as a contracts-and-IP problem as well as a tooling question.** ASAP's published member programming includes a session titled **"Incorporating How AI is Impacting Inventorship and Contract Terms"** ✅ *(ASAP events page, retrieved this pass)*. **Documented inference available here:** in a discipline whose output is agreements, the arrival of AI lands first in the *document* — inventorship, IP ownership, contract terms — rather than in the relationship. 🔧 *The inference is the guide's; that the session exists is ✅.*
- ✅ **Vendor documents about AI in partner ecosystems exist, are numerous, and are dated.** Retrieved in this pass: an **Impartner** "AI Partner Playbook" (published **August 2025**) and a later Impartner ecosystem playbook (published **September 2026**), both PDFs from a partner-relationship-management software vendor ✅; an AI channel-partner ecosystem analysis PDF (dated **October 2025**) ✅; and vendor/consultancy blog and contributor pieces dated **2025 and 2026** ✅. **Their existence and dates are fact. Their content is 11.2.**

### 11.2 Vendor claim — and how it is labelled

- ⚠ **Claim:** that AI and automation applied to partner ecosystems "streamline operations from recruitment to performance analysis" and thereby "allow channel managers to focus on strategy and relationship building rather than administrative tasks" ✅ *as published by a partner-programme software vendor (retrieved this pass)*; and that AI is transforming partner programmes "from onboarding and enablement to co-marketing, sales support, and ecosystem strategy" ✅ *as published in a vendor playbook*.
- ⚠ **Quality labelling, stated in one sentence:** **all of the above are marketing documents.** Every source in this group sells partner-management or partner-automation software, or is a contributor piece published under a vendor-friendly council banner; none presents a methodology, a control group, or a measured outcome; and the claim *"AI lets you focus on relationships"* is a claim about the vendor's own product category. **The guide reports the claims because they define what is being promised to buyers of this work — not because they are true.** ✅ *as "the vendor says"*, ⚠ *as evidence of an outcome.*
- ⚠ **One further claim worth isolating because it will be made to you:** that partnerships are becoming more important *specifically because of* AI — ecosystems as "a fundamental requirement for market leadership" rather than a peripheral channel ✅ *(as published in a dated ecosystem-analysis document from a channel-focused publisher)*. **This is a claim; the guide found no evidence for or against it, and does not adjudicate it.** ⛔

### 11.3 The guide's own reasoning

🔧 *Everything in this subsection is the guide's construction. It is reasoning, offered with its reasons, and marked as such because it is not sourced.*

**The method: ask which parts of the work have a text artefact and an existing dataset, and which require authority and trust. The first set is automatable; the second is not.**

| Part of the work (§3) | Automatable? | Why, in one line |
|---|---|---|
| **Partner research and ecosystem scanning** — who exists, what they integrate with, what they announced | **Mostly yes** | The inputs are public documents; the output is a list. This is the most clearly automatable task in the job. 🔧 |
| **Ecosystem mapping as a maintained artefact** | **Yes, as maintenance; no, as judgement** | A machine can keep a map current; deciding which relationships *matter* is a strategic judgement with no training data. 🔧 |
| **Drafting** — the partner brief, the joint value proposition, the announcement, the first-draft agreement, the QBR pack | **Largely yes** | Text from templates with structured inputs. Expect the drafting burden to fall sharply and the *review* burden to rise. 🔧 |
| **Discovery conversation** | **No** | The task is to induce a counterpart to describe their own strategy honestly. That is a trust problem, not an information problem. 🔧 |
| **Negotiation of terms with a competitor-counterpart** | **No** | Requires the authority to commit, and the ability to read what a counterpart will accept. Both are personal. 🔧 |
| **Internal advocacy** | **No — and it is the least automatable part of the job** | The output is a colleague's *commitment of resources*. No tool can obtain one. If §3.1(5) is right that this is the majority of the work, then the majority of the work is structurally out of reach of the tools being sold. 🔧 |
| **Post-launch maintenance** | **Partly** | Enablement materials and progress reporting can be largely automated; *noticing that the partner has gone quiet* requires someone to care. 🔧 |

**Three consequences, each with its reason:**
1. **The automatable share may be the share that was never the differentiator.** If a BD function's value is §8.2's patience, internal credibility and the willingness to decline, then AI compresses the cost of the *support work* around that value without touching the value itself. 🔧
2. **The vendor content in 11.2 is therefore aimed at the wrong layer, or at an honest layer.** ⚠ *The guide cannot decide which, and says so: vendor tooling for partner programme operations is genuinely useful for partner programme operations. It is not evidence that BD judgement is being automated.*
3. **The most likely near-term change is a shift in what the practitioner is doing, not in how many practitioners there are.** The drafting, research and reporting load should fall, and the freed time goes to relationships and internal work — which is the *opposite* of a headcount effect and is a prediction the guide explicitly does not have evidence for. 🔧

**And the discipline this section requires, stated as a rule:** **this guide asserts NO headcount effect on business-development roles in either direction, because no source located in this pass supports one.** ⛔ *Any statement that "AI is replacing/freeing/expanding BD roles" that you read elsewhere is, as far as this pass can establish, unsupported by anything with a method.*

---

## 12. Working with BD from a Technical Role

*This is reading (ii): the section a solution architect, engineer, product manager or security reviewer needs when a business-development function enters their life. It is largely the guide's construction 🔧, and it is written to be checked against your own organisation rather than quoted at anybody.*

### 12.1 What BD genuinely needs from a technical counterpart

🔧 *The guide's construction, derived from §3's work model and §4.3's asymmetry.*

1. **Credibility in the room, on demand.** A partner's technical evaluators will decide within one meeting whether the joint offering is real. BD cannot supply that judgement, and a BD person bluffing technical feasibility is the single most damaging thing that can happen to a partnership. **The ask is bounded and concrete: one hour with a named engineer, in front of the partner, saying what is true.**
2. **Feasibility judgement with a range, not a number.** "Yes, about two quarters, if nothing else moves" is more useful to BD than "yes, Q3", because the first can be re-planned and the second becomes a promise (§4.3).
3. **A prototype that is honest about its limits.** A demo that works because a human is driving it, presented as if it were a product, transfers a false belief into a signed agreement. **The useful technical contribution is a demo whose presenter states, unprompted, what it does not do.**
4. **An early, roughly accurate view of the support burden.** Every integration produces support tickets and every partner escalates them. Whether your team can absorb that is a technical-operations question, and it is asking a great deal to have it answered after signature.
5. **A named technical owner for the partnership** — the same thing BD must supply on its side of the table (§12.2.3).

### 12.2 What a technical counterpart needs from BD

🔧 *Same grade. These are the four demands, in order of how often they are violated.*

1. **Advance notice.** Not approval rights — notice. The commitment should be visible to engineering **before** it is made, not after. This is the whole of §12.3's first friction point.
2. **No unkeepable promises.** A promise is unkeepable when it is made about work the promiser does not control, on a timeline set by a marketing calendar. The test: *if you have to check with engineering before you can say it, you cannot say it yet.*
3. **A named owner on the partner side.** An integration with an unnamed counterpart organisation is an integration with a person who may leave. **This is a BD deliverable and it is reasonable to demand it in writing** — BDO's respondents ranked lack of internal alignment in at least one partner among the top three causes of underperformance ✅, and "at least one partner" includes the other one.
4. **A decision on whether the integration is even wanted.** The most valuable thing BD can do for engineering is *decline* the partnership (§8.2.3). A BD function that never brings engineering a "no" is not filtering; it is queueing.

**The reciprocity, stated because the section would be one-sided without it:** engineering owes BD a *fast* answer, including a fast "no". A technical function that takes three weeks to say "we can't do that" has made the promise inevitable, and will then be blamed for the promise. 🔧

### 12.3 The friction points, named plainly — with the practice that prevents each

🔧 *All four are the guide's construction: the friction pattern plus the practice that prevents it. They are written as testable propositions — if the practice is absent in your organisation, the friction is not a personality problem.*

| # | Friction | What it looks like | The practice that prevents it |
|---|---|---|---|
| **1** | **The commitment made before engineering sees it** | A partner tells your architect "we're building this together"; the architect has never heard of the partner. Sometimes the first signal is a **press release**. | **The feasibility gate.** No partnership may be described externally as involving technical delivery until a named engineer has recorded, in writing, (a) what is in scope, (b) what is explicitly out, and (c) roughly when. **The gate protects BD, not engineering** — it is the mechanism by which BD avoids making a promise it cannot keep. Pair it with §12.2.1's notice right. |
| **2** | **The pilot that becomes a permanent unowned workload** | A proof of concept for one partner was funded by nobody, built by an engineer "as a favour", and is now required to keep working forever, with no owner, no monitoring and no on-call. | **A pilot exit clause, agreed before the pilot starts.** The three questions: *who owns it at the end; does it end unless someone signs up to own it; and what happens to the data and the endpoints when it does.* The first question can only be answered by a name. **This is §6.2 stage 3's exit criterion expressed in engineering terms.** |
| **3** | **The "strategic" partnership that consumes a quarter of engineering time and returns nothing** | A partner with executive sponsorship, a jointly branded announcement, three workshops, an integration effort, and no revenue a year later. Engineering's opportunity cost is invisible to the partnership's sponsors. | **Cost the engineering time explicitly, and report it beside the partnership's revenue.** The practice has two parts: (i) engineering time on partnership work is logged as partnership work rather than absorbed; (ii) the partnership's review includes that number. **No metric is being invented here** (§7.5) — one existing fact (engineering hours) is being placed next to another existing fact (revenue), and the judgement is left to the reader. |
| **4** | **The support and escalation vacuum after launch** | The partner's tickets arrive in a general queue with no distinguishing label, no owner and no service expectation, and are triaged against customers who pay. | **A named owner, a labelling rule and a stated response expectation, recorded at launch.** *This is post-launch discipline and it is owned elsewhere:* the operating cadence, escalation handling and adoption mechanics are the subject of [post_sales_customer_facing_experience_guide.md](post_sales_customer_facing_experience_guide.md), and the **buy-side** mechanics of governing and exiting a supplier relationship are the subject of [vendor_management_guide.md](vendor_management_guide.md). **This guide does not re-derive either** — the practice is named, the destination is given. |

### 12.4 The one structural observation that makes all four frictions predictable

🔧 **All four friction points share one cause: the person who makes a partnership commitment and the person who pays for it are different people, on different timescales, measured on different instruments.** §4.3 states it as the promise-accountability asymmetry. In engineering terms it means: *the cost of an over-promise lands on a roadmap that was already committed, and the benefit of making it lands on a function whose numbers improve when the announcement goes out.*

**Two consequences for the technical counterpart, offered as advice rather than as finding:**
1. **Ask for the mechanism, not the intention.** "Who owns it after launch?" and "what happens if it doesn't work?" are the questions that distinguish a partnership from an announcement. §6.1's published lifecycle names **"Launching and Managing"** and **"Transform, Innovate, or Exit Gracefully"** as phases ✅ precisely because the profession has learned that the signature is not the finish line.
2. **Note the difference from advisory work, and where this guide stops.** A consultant's engagement has a scope, a deliverable and an end date; a partnership has a mechanism, an owner and no natural end. The advisory-versus-delivery distinction — how those two modes differ, and what each demands of the client — is owned by [consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) §5, and is not re-derived here. **What differs for the technical counterpart is one thing only: you can finish an advisory engagement, and you cannot finish a partnership without performing §6.2's exit stage.** 🔧

---

## 13. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is not a real institution and no event described here happened. **Every figure in this section is labelled illustrative** and none is sourced, benchmarked or derived from any real organisation. The purpose is to put the §4, §6, §7 and §12 machinery into one continuous narrative with two parties on opposite sides of a table. Where a practice is named in the fiction that corresponds to a sourced or constructed finding earlier in the guide, the section cross-references it. 🔧

### 13.1 The setting

Cymbal Bank's payments division has a business-development function of four people. Their mandate, as written, is *"identify and secure strategic partnerships that extend the payments proposition."* The bank's platform engineering group is 60 people and already committed for the year. The two functions interact roughly monthly, and the relationship is not good — engineering believes BD makes promises; BD believes engineering says no by default.

A technology partner — call them **Vendor T** — approaches Cymbal's BD team offering an **integration-led partnership**: T's fraud-scoring service would be embedded in Cymbal's payment acceptance flow, sold jointly to merchants, with co-sell support from T's field sellers. **All figures illustrative.**

### 13.2 The qualification — the four questions, asked properly

BD runs §6.2's four-part partner-qualification test 🔧, deliberately separating it from deal qualification ([meddicc_guide.md](meddicc_guide.md) owns that; the two are not the same question, §4.4).

1. **Does T have a reason that survives a personnel change?** Yes — T's own distribution economics depend on being embedded in acquirers' flows, so the motive is structural rather than relational — which is what makes it durable. **Illustrative but conforms to §6.2's first test.**
2. **Will T commit a named person?** T offers a "partnership lead" in a slide. BD's second meeting asks for a name in a calendar invitation for a recurring meeting. **T provides one.** *This is the ask §12.2.3 demands of BD, applied to the counterpart.*
3. **Can Cymbal support it?** This is where it nearly ends. T's service would generate merchant queries that land in Cymbal's support desk, and Cymbal's support function has no agreement to take partner-originated volume. **Illustrative issue; structurally it is §12.3 friction 4 arriving at qualification time rather than after launch — which is the point of qualifying properly.**
4. **Does it scale, or is it one deal in disguise?** Unclear. T has three acquirer integrations of which one is materially transacting. **BD's honest answer is "we don't know yet", which converts stage 2 into a bounded stage 3 pilot rather than a launch.**

### 13.3 What BD promises versus what engineering can deliver

The scene the guide is describing is the ordinary one, and its mechanics are the section's payload.

- **What BD's external materials say (draft, before engineering input):** "embedded fraud scoring in the acceptance flow", "joint go-to-market with co-sell support", "available to merchants in the next two quarters." **Illustrative.**
- **What engineering's assessment actually produces:** the fraud-scoring call can be inserted in the *authorisation* path, but only synchronously, which adds latency to every transaction unless it is made asynchronous; making it asynchronous requires a scoring model variant T has not built; the merchant-facing reporting requires an event stream Cymbal does not currently emit; and the change touches a component under a change-control regime that requires a two-week review window and a named accountable owner.
- **The honest translation of the above into a commitment:** *authorisation-path screening in a staging environment in roughly one quarter, using T's current synchronous API, with a stated latency budget; production and merchant-facing reporting out of scope until a second review.* **This is §12.1.2's "range, not a number" and §12.1.3's honest prototype in one line.**
- **What BD does with it, and why this is the section's real subject:** BD does **not** hide the range. It rewrites the external materials to match — which costs BD a quarter in the announcement calendar and saves both organisations the failure in §14's first anti-pattern. **In the fiction, BD's willingness to take that loss is the moment the engineering relationship changes**, because the architect learns that the range they gave was used rather than rounded up. 🔧 *The guide's construction — but note that this is exactly what §12.2.1's advance notice and §12.3's feasibility gate exist to produce.*

### 13.4 The partnership Cymbal declines

A second conversation is running in parallel — **Vendor U**, a payments data broker offering a **reseller partnership**: U would resell Cymbal's merchant acceptance product into a market segment Cymbal cannot reach directly. **Illustrative.**

**The decline arises from §6.2 stage 2, and the reasons are stated because the skill is §8.2.3:**
1. **U cannot name an owner.** Three meetings, three different U attendees, no named partnership lead, and no recurring calendar invitation. §6.2's second test fails, and the guide's reading of BDO's finding is that this is the shape of an internal-alignment failure waiting to happen ✅/🔧.
2. **U's commercial ask is structurally wrong for the motion.** U wants reseller margin *and* exclusivity in the segment. **The exclusivity is the disqualifier:** it converts a distribution arrangement into a restraint on Cymbal's own direct motion, and Cymbal's answer is that it will not restrict its own channel to obtain a partner it is not sure can sell.
3. **The support obligation is unbounded.** U's proposal contains no support model — every merchant query would land on Cymbal. **Same defect as in 13.2, unaddressed.**
4. **The volume claim does not survive the first question.** U projects volumes "based on its pipeline". Asked for the definition of a pipeline opportunity, U's answer is a list of companies it has emailed. **This is §1.3's qualified-opportunity problem and §7.3's attribution problem arriving before signature rather than after — which is the only cheap time to discover them.**

**BD declines, in writing, with the reasons.** *The guide's note on why this is the section's most instructive act: declining costs BD nothing on any report that exists (§8.2.3: both "yes" and "no" produce zero), and it is the single act most likely to build the internal credibility that §8.2.2 calls the scarce resource. An organisation that cannot see its BD function decline partnerships has no way to know it is filtering at all.*

### 13.5 The commercial and support terms for Vendor T

**All figures illustrative. No benchmark is claimed.**

- **Commercial shape:** revenue share on T-attributed merchant volume rather than a flat licence, because the parties cannot agree on a volume forecast and a share-based term fails gracefully if the forecast is wrong. **Structural note:** the term was chosen because **attribution is contested (§7.3)** — a share-based term requires a written attribution rule, and the negotiation of that rule is what forces both sides to define "partner-sourced" before money is at stake. 🔧
- **The attribution rule, in the agreement:** volume counts as T-sourced where T registered the merchant in the shared pipeline **before** Cymbal's first contact, verified by timestamp; Cymbal-sourced merchants introduced to T for scoring are excluded; disputed items go to a named pair of individuals, one per side, on a two-week clock. **This mirrors, at contract scale, what vendor co-sell programmes do procedurally ✅ — and note that it was written *before* the deal, not reconstructed after.**
- **Support terms:** T operates first-line support for its own service; Cymbal operates first-line for the payment flow outside it; a joint escalation path is named with contacts on both sides and a stated response expectation. **This is §12.3 friction 4's practice, agreed at contract time.**
- **Latency and availability:** a stated latency budget in the authorisation path, and a stated behaviour when T's service is unavailable — including the decision that Cymbal's flow degrades to an unscreened path rather than failing. **That single clause is the engineering-visible commercial decision, and it could only have been written with engineering in the room.** 🔧
- **Exit:** either party on stated notice, with a defined wind-down period and obligations about data return. *This is §6.2 stage 8, written in advance, and §6.1's published "Exit Gracefully" phase ✅ applied to paper.*
- **No exclusivity.** Both sides retain the right to work with others. **Deliberate, and it is what makes the arrangement survive T's or Cymbal's strategy changing** — §3.1.4's coopetition terms written to outlive the relationship's warmth. 🔧

### 13.6 Enablement and ownership after launch

- **Ownership:** a named owner inside Cymbal engineering (a staff engineer, 10% allocation, recorded in the component's ownership file) and a named owner on T's side (the partnership lead from 13.2), plus a named commercial owner in Cymbal BD. **Three names, written down, before launch.** §6.2 stage 5's enablement failure and stage 7's no-owner problem are thereby addressed in advance rather than discovered.
- **Enablement:** T's field sellers cannot sell the joint offering until two things exist — a joint one-page proposition stating what the integration does *and does not* do, and a named Cymbal technical contact for their pre-sales questions. **The "does not do" column is the enablement artefact that prevents the next over-promise.** 🔧
- **Cadence:** a monthly operating call and a quarterly review that reports three things side by side — joint transacting volume, T-registered pipeline, and **Cymbal engineering hours consumed**. **Placing the third beside the other two is §12.3 friction 3's practice, and no new metric is invented (§7.5).**
- **The pilot's exit criterion, discharged:** the staging integration ends unless production is approved at the quarterly review on stated criteria. *This is §6.2 stage 3's exit criterion and §12.3 friction 2's named owner in one mechanism.*
- **Where the guide stops:** the post-launch operating discipline — adoption, health, escalation handling, the review mechanics themselves — is owned by [post_sales_customer_facing_experience_guide.md](post_sales_customer_facing_experience_guide.md). What §13.6 establishes is only who owns what and by when.

### 13.7 How the two functions renegotiate their way to a workable arrangement

**This is the section's conclusion, and it is a process change rather than a friendship.** 🔧 *The guide's construction.*

Four things change at Cymbal as a result of running one partnership through this way — and none of them is "better communication" in the abstract:

1. **The feasibility gate becomes a standing meeting, not a favour.** BD brings every partnership with a technical component to a fortnightly 30-minute slot with two named engineers, before anything is promised externally. **Effect:** §12.3 friction 1 stops happening, because there is no longer a path by which a commitment can be made without one.
2. **The external materials are drafted *from* the engineering assessment, not corrected afterwards.** **Effect:** BD loses roughly a quarter of announcement lead time and gains the ability to say "this is what we can do" to partners without checking.
3. **The engineering-hours figure appears in BD's own quarterly report, written by BD.** This is the single most important change, and it is a change of *ownership of a bad number*: BD reports the cost of its own programmes rather than leaving engineering to discover it. **Effect:** friction 3's confrontation becomes an internal one that BD performs on itself, which is the only version of it that survives a change of leadership.
4. **BD is measured on a stated mix rather than a revenue number it does not control** (§7.2 — *and note the guide is not proposing a metric here; it is describing the fictional outcome of §7.6's four questions being asked*): signed and **operating** partnerships (distinguishing §6.3's "signs and never sells" outcome from a success), joint transacting volume where the partner registered the pipeline, and the number of partnerships **declined with reasons**. **The third is the instructive one:** a function rewarded for declines has to generate declines and justify them, and the justification is where §8.2.3's decisive skill becomes visible instead of invisible.

**The unresolved remainder, stated because the fictional example must not end tidily.** Vendor U is declined, and U's regional head escalates to a Cymbal executive who had met him at a conference. **The executive asks why Cymbal is "saying no to growth".** BD's answer is the four disqualifiers from §13.4, in writing, with the support obligation and the exclusivity named as the reasons. **In the fiction the executive accepts the answer. In the guide's honest reading of the real world, this is the point at which BD's internal credibility (§8.2.2) is either present or absent — and it is decided by something that happened long before the escalation.** 🔧 That, and not any metric, is why the discipline is hard.

---

## 14. The Anti-Patterns

**Nine named patterns, each symptom → cause → guardrail.** 🔧 *All nine are the guide's construction; the causes are the mechanisms established in the referenced sections. Offered as diagnostic checks for a reader's own organisation, not as researched findings. The six required patterns are 1–6; 7–9 were added because they recur in the material of §7 and §11.*

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **1** | **The partnership announced before anything is built** | The press release is out; the integration exists as a slide; the partner's sellers are already pitching it | The announcement date was set by a marketing calendar and the technical assessment came afterwards (§12.3 friction 1; §4.3's promise-accountability asymmetry) | **The feasibility gate (§12.3):** no external description of technical delivery until a named engineer has recorded scope, exclusions and a rough timeline. Accept the cost — the guide's fictional BD lost a quarter of lead time and gained the ability to promise anything at all (§13.7) |
| **2** | **The signed agreement with no owner** | The partnership is real, the contract is executed, and nobody can say who is responsible for it on either side | A BD role whose mandate ends at signature; no stage-5 hand-off assigned (§6.2 stage 5, stage 7) | **A name per side, written down before launch** — commercial owner, technical owner, counterpart owner. If the partner cannot name one, that is a stage-2 qualification failure (§12.2.3; §13.6) |
| **3** | **The BD function measured on revenue it does not control** | The function reports a revenue number it cannot influence; the number is met by relabelling or missed and the function is judged to have failed; BD staff learn to claim credit for deals sales closed | A measurable objective displacing an unmeasurable one (§7.5); the absence of any settled instrument (§7.2) | **Do not solve this with a metric (§7.5).** Ask the four questions of §7.6; measure a stated mix including declines; and **never report partner revenue without the engineering cost and the attribution rule beside it** (§13.7 change 3) |
| **4** | **The technical counterpart who learns of the commitment from the press release** | An architect's first notification of a partnership is a link to the announcement | No notice right and no gate; the promise was made upstream of the people who pay for it (§4.3; §12.3 friction 1) | **Advance notice as a right, not a courtesy** (§12.2.1); the gate in #1; and reciprocally, **engineering owes a fast answer including a fast "no"** (§12.2) |
| **5** | **The "strategic" alliance that consumes engineering and returns nothing** | Twelve months in: three workshops, a joint logo, a materially built integration, no transacting volume, and nobody can state the engineering hours consumed | Executive sponsorship substitutes for a commercial case; engineering time is absorbed rather than attributed (§12.3 friction 3) | **Log partnership engineering time as partnership work, and report it beside revenue** (§12.3 friction 3; §13.7 change 3). Then apply §6.2's exit stage — and note that a function which cannot decline (§8.2.3) cannot exit either |
| **6** | **The ecosystem map that is never revisited** | An ecosystem map produced with effort at a strategy offsite; eighteen months later nobody has opened it, and two partners on it have been acquired | Mapping is treated as an artefact rather than a maintenance process; the review meeting is the first thing cancelled in a busy month (§3.1.2) | **Assign the map an owner and a cadence, and put it on the same review as the pipeline.** If it cannot earn a recurring slot, it should not be produced — a stale map is worse than none, because it is trusted |
| **7** | **The partner roster grown for its own sake** | Twelve partners signed this year; revenue flat; support and enablement load up substantially | A recruitment target is measurable and partnership value is not (§7.5); the recruit-to-target incentive actively produces §6.3's "signs and never sells" partner | **Set the target on operating partnerships, not signed ones** — and define "operating" narrowly (transacting, with a named owner and a working support path). Refuse the count-based target in the first place if you can |
| **8** | **The pilot with no exit criterion** | A proof of concept that has been running for two years, funded by nobody, owned by a departed engineer, kept alive by the fear of breaking it | No stage-3 end condition; the same defect as the technical hangover in §12.3 friction 2 | **Three questions before the pilot starts: who owns it at the end; does it end unless someone signs up to own it; what happens to the data and endpoints.** A pilot without an end date is a production system with no owner (§12.3; §13.6) |
| **9** | **The attribution argument conducted after the money moves** | Two functions each report the same deal; finance cannot reconcile the totals; the partner disputes the share; the argument is settled by seniority | The sourced/influenced vocabulary is industry usage not a standard (§7.3); attribution in co-sell is procedural and first-registrant-wins (§7.3) | **Write the attribution rule before the deal**, verified by timestamp, with a named pair of individuals and a clock for disputes — which is what a share-based term forces anyway (§13.5). Adopt the answer from §7.6: what is the denominator, and who bears the cost when it is wrong |

---

## 15. The Claims Audit

**How to read these tables.** *Status* is **Verified** (checked this pass against a named source), **Flagged** (widely repeated but not verified, or an estimate whose provenance is stated), **Rejected** (could not be verified, or sources conflict), or **Constructed** (this guide's own analysis, offered as such). *Source quality* is one of **tier-1 primary** (the organisation making the claim about itself, a standard, a regulator, an employer's own careers page, a professional body's own pages), **employer page**, **reference work**, **professional body**, **vendor content / marketing document**, **industry press**, or **the guide's own construction**. Retrieval labels of "September 2026" mean the page was read during the writing of this guide in that month.

### 15.1 The title problem — every role definition traced to its publisher

| # | Claim | Status | Who published the definition, and when | Quality |
|---|---|---|---|---|
| 1 | Business development **varies across industries and countries**, often including work performed by IT programmers, specialised engineers, advanced marketers, key account managers and sales/relationship professionals, such that **"it has become challenging to clearly define the unique characteristics of the business development function"** | ✅ Verified as published | Reference work, "Business development"; retrieved September 2026 | Reference work |
| 2 | Business development is defined as **the tasks and processes concerning the analytical preparation of potential growth opportunities, and the support and monitoring of the implementation of growth opportunities, but not decisions on strategy and implementation** | ✅ Verified **as quoted**; ⚠ the underlying attribution (Palgrave Encyclopedia of Strategic Management / Sørensen) was read **through** the reference work, not at the Palgrave entry | Reference work quoting that encyclopedia; retrieved September 2026 | Reference work |
| 3 | For larger, well-established companies **especially in technology-related industries**, "business development" often means **setting up and managing strategic relationships and alliances with third-party companies** | ✅ Verified as published | Same reference work; retrieved September 2026 | Reference work |
| 4 | **Variant A — new-partner/channel hunting** is structured in vendor programme documentation: **AWS Partner Paths** (Software, Hardware, Services, Training, Distribution), each with its own enrolment route and qualification requirement | ✅ Verified as documented | AWS Partner Network, `aws.amazon.com/partners/paths/`; retrieved September 2026 | Tier-1 primary (vendor, describing itself) |
| 5 | **Variant B — alliance/partner management** has a professional body with a published lifecycle, handbook and certifications: ASAP, a nonprofit **which states it began in 1998**; seven alliance phases (alliance-specific strategy → analysis and selection → value-creating negotiations → operational planning → structuring and governance → launching and managing → transform, innovate, or exit gracefully); certifications **CA-AM** and **CSAP**; **5th edition of the Handbook announced for fall 2026** | ✅ Verified as published | Association of Strategic Alliance Professionals (ASAP), own site; retrieved September 2026 (the reference work's citation places the **3rd edition at January 2013**, ISBN 978-0-9882248-1-0 ✅) | Professional body (primary, self-describing) |
| 6 | **Variant C — corporate development** is a distinct function defined as planning and execution of strategy **primarily through M&A or divestitures**, and *including* "arranging strategic alliances or partnerships or joint ventures" among its activities | ✅ Verified as published; ⚠ the article carries a **"needs more citations" tag dated August 2012** | Reference work, "Corporate development"; retrieved September 2026 | Reference work |
| 7 | **Variant C corroborated by an employer:** "Director, Corporate Development & Investor Relations" responsibilities include developing global corporate development strategy and **identifying, assessing and cultivating a pipeline of actionable M&A opportunities including tuck-in and transformational deals** | ⚠ Flagged | Airwallex careers page (read from the **search-result description**; the page failed to extract this pass). A separate **MyCareersFuture** listing for the same title exists in Singapore but did not render its content ⛔ | Employer page (unverified body text) |
| 8 | **Variant C adjacency to B exists inside single job titles:** ASAP's own author biographies list a **"Senior Director, Alliance Management and Corporate Business Development"** and a **"VP, Global Head of Alliance Management"** | ✅ Verified as published | ASAP, author biographies on its "Alliance Value" page; retrieved September 2026 | Professional body |
| 9 | **Variant D — strategic/enterprise deal support:** the role exists in employer practice; **no professional body defines it as a discrete occupation** | ⚠ Flagged (employer usage only) | No publisher located | Practitioner observation |
| 10 | **Variant E — a rebrand of ordinary sales, established from an employer's own definition:** Google posts **"Business Development Manager, New Business Sales (English, Mandarin)"** (Shenzhen), minimum qualifications **"2 years of industry experience in advertising, consultative sales, or business development"**, responsibilities covering client acquisition, go-to-market strategy, and **"the entire business cycle from lead generation and initial engagement to agreement execution"** | ✅ Verified as published | **Google Careers**, posting page; retrieved September 2026 | Tier-1 primary (employer, describing its own role) |
| 11 | Google also lists a **"Business Development Consultant, New Business Sales"** title | ⚠ Flagged | Google Careers, via search-result listing; the individual posting was not extracted | Employer page (unverified) |
| 12 | **Pharma/biotech and high-tech have the most mature BD function:** the function "seems to be more utterly matured in high-tech, and especially the pharma and biotech industries" | ✅ Verified **as the reference work's hedged wording**; ⚠ no pharma BD job description was retrieved to corroborate | Reference work; retrieved September 2026 | Reference work (hedged) |
| 13 | **"Business development" names a family of roles rather than one profession** | ✅ **Supported** by items 1, 4–8, 10 — the conclusion is drawn from named publishers, not asserted | — | Guide's conclusion from verified items |
| 14 | **Distribution of the variants by organisation type** (e.g. "startups usually mean sales") | ⚠ **Flagged inference**; no source quantifies it | None located | Guide's construction 🔧, marked as such |
| 15 | No occupational classification with "business development" as a discrete code and published scope note was located; `bls.gov` (Occupational Outlook Handbook and OES) **failed to load** | ⛔ **Not verified — tool limitation** | Failed pages: `bls.gov/ooh/management/sales-managers.htm`, `bls.gov/oes/current/oes112021.htm` | Tool limitation; see §16 |

### 15.2 The BD-versus-sales analysis

| # | Claim | Status | Basis and quality |
|---|---|---|---|
| 16 | **Sales works the venue; business development builds it** | 🔧 **Constructed** — the guide's own formulation, developed into the consequences below | Guide's construction |
| 17 | The consequence table: cycle length, unit of work, what the number measures, cadence, internal-selling load, compensation shape, failure mode | 🔧 **Constructed** as a unified analysis. **Individual cells that rest on a source are marked in place**: cycle length is ⚠ guide characterisation (no source measures partner-cycle duration); cadence is ✅ where it rests on ASAP's published lifecycle phases and Microsoft's published required response time frames for referrals | Mixed — see §4.1 |
| 18 | **"In most organisations a majority of a BD person's working hours are spent persuading colleagues"** | 🔧 **Constructed and explicitly unsourced.** **No proportion is printed anywhere in this guide** | Guide's construction; see §3.1(5) |
| 19 | **Internal cohesion is a real and formally recognised problem in partnership work** — supported by (a) ASAP running member programming titled **"Aligning for Success: Internal Cohesion in Strategic Alliances"**, described as building cohesion "across their internal organization" ✅; and (b) BDO's survey ranking **"lack of internal alignment within at least one partner"** as the **top** contributor to alliance underperformance ✅ | ✅ Verified as published | ASAP own site and BDO survey, both retrieved September 2026 |
| 20 | **Why BD and sales misjudge each other** — instrument mismatch, time-horizon mismatch, vocabulary mismatch, framing mismatch, asymmetric promise cost | 🔧 **Constructed analysis with stated reasons**; documented supports: the pipeline definition the vocabulary argument rests on ✅ (reference work), and the "conflicting metrics" dynamic ✅ (BDO: 53% expecting growth from two-to-three partnership types; expectation-driven differences) | Guide's construction over verified inputs |
| 21 | **A partner that passes deal qualification can still be a partnership that should never be signed** | ✅ Supported at the level of documented mechanism — vendor programmes' partner-entry requirements count launched and qualified opportunities and recognised revenue, an instrument no deal-qualification framework supplies | AWS ISV Accelerate requirements page; retrieved September 2026 |
| 22 | **Where the boundary blurs** — the overlay/deal-support role, the partner-sourced deal, the founder/early-stage case, the rebranded sales role, the post-signature expansion motion | 🔧 Constructed, **except** the rebranded-sales case, which is ✅ documented at item 10 | Mixed |

### 15.3 Motions, programmes and lifecycle

| # | Claim | Status | Publisher and date | Quality |
|---|---|---|---|---|
| 23 | **Co-sell** is "a collaborative approach between AWS and Partners to deliver comprehensive cloud solutions that drive customer success"; benefits include funding, collaboration and lead generation; AWS Sellers are **compensated more** to co-sell with ISV Accelerate partners, and partners boost a **"co-sell recommendation score"** by sharing opportunities | ✅ Verified as published | AWS co-sell page; retrieved September 2026 | Tier-1 primary |
| 24 | **Resell:** Microsoft's Cloud Solution Provider programme "enables partners to resell Microsoft cloud services"; indirect resellers work through a distributor which **handles billing with Microsoft, while the reseller owns the customer relationship**; the **Microsoft Partner Agreement** must be accepted, and a distributor relationship is required to transact | ✅ Verified as published | Microsoft Learn, CSP indirect-reseller page, **last updated 29 May 2026** | Tier-1 primary |
| 25 | **Referral:** partners publish a business profile listed wherever customers and internal seller-agents search; leads arrive through the Partner Center **Referrals** feature; **response time frames are required** to keep receiving leads | ✅ Verified as published | Microsoft Learn, referrals page, **last updated 28 May 2025** | Tier-1 primary |
| 26 | **Programme catalogue as a real ontology of motions:** reseller programmes (distributors, Solution Provider Program, Channel Partner Private Offers), services programmes, managed-services programmes, technology-solutions programmes (ISV Accelerate, ISV Workload Migration, Well-Architected, Marketplace List & Sell), business-outcome and public-sector programmes | ✅ Verified as published | AWS Partner Programs; retrieved September 2026 | Tier-1 primary |
| 27 | **OEM is an ambiguous term**, confused and used interchangeably with original-design and original-brand manufacturing | ✅ Verified as published | Reference work, "Original equipment manufacturer"; retrieved September 2026 | Reference work |
| 28 | **Joint venture:** formed when two or more distinct firms combine a portion of their resources to form **a separate, jointly-owned entity**; differs from M&A; reasons include market access, scale efficiencies, shared risk, access to skills | ✅ Verified as published; ⚠ article carries multiple maintenance tags | Reference work, "Joint venture"; retrieved September 2026 | Reference work |
| 29 | **Strategic alliance:** an agreement between two or more parties to pursue agreed objectives **while remaining independent**; usual fall short of legal partnership/affiliate status; **definitions disagree on whether JVs are included** | ✅ Verified as published; ⚠ list-format tag | Reference work, "Strategic alliance"; retrieved September 2026 | Reference work |
| 30 | **Coopetition** — firms both cooperating and competing; a portmanteau; game-theory roots (von Neumann and Morgenstern, 1944); popularised in business use by the 1996 book by Brandenburger and Nalebuff | ✅ Verified as published; ⚠ some illustrative examples carry "clarification needed" tags | Reference work, "Coopetition"; retrieved September 2026 | Reference work |
| 31 | **Channel/distribution:** distribution is making a product available to the consumer or business user; **"place" in the marketing mix**; direct or indirect channels; intensive/selective/exclusive approaches; push versus pull | ✅ Verified as published | Reference work, "Distribution (marketing)"; retrieved September 2026 | Reference work |
| 32 | The **partner lifecycle** — ASAP's seven phases ✅; ASAP's three states (**startup, steady state, wind-down**) ✅; **Value Inflection Points (VIPs)** defined by that publisher as "moments when what happens next can significantly affect the value of the partnership", with a chapter on **the first dispute** ✅; the reference work's five-stage alliance life cycle with sections on common mistakes, success factors and risks ✅ | ✅ Verified as published | ASAP own site; reference work "Strategic alliance"; retrieved September 2026 | Professional body + reference work |
| 33 | The guide's **eight-stage partner lifecycle** (identify → qualify → pilot → negotiate → launch → scale → govern → exit) | 🔧 **Constructed** — stage names are the guide's; the mapping to published phases is noted in place; **"pilot" is the guide's insertion** | — | Guide's construction |
| 34 | The guide's **four-part partner-qualification test** | 🔧 **Constructed** | — | Guide's construction |

### 15.4 Metrics, attribution, organisational and market figures

| # | Claim | Status | Publisher, date, method | Quality |
|---|---|---|---|---|
| 35 | **Alliances have driven on average one-third of company revenue over the past five years**; the share spanned **27%–36%** across the most-represented industries; **62%** reported most/great deal of innovation from third-party collaboration; **33%** of alliances were multilateral (healthcare 42%, life sciences 19%); **80%** expect to increase multilateral alliances; **3-year revenue growth was 27% for the most alliance-reliant vs flat for the least**; a 2020 study on coopetition found **73% more revenue growth** where >75% of future success was expected from external assets | ✅ Verified as published; ⚠ **evidence-grade caveat**: method stated only as **data gathered in 2021 and 2022 from 183 alliance and account management professionals across 11 industries**, plus "more than 25 years' worth of consulting" — **no sampling frame, response rate or questionnaire published**; the publisher (BDO) **sells alliance advisory services**; respondents are the firm's own network | BDO, "The State of Alliance Management in 2024 and Beyond," **published 2024** | Industry/consultancy survey — vendor-interest, self-selected sample |
| 36 | **"More than half of partnerships fail to fully achieve their objectives"**; alliance failure rates from 1996 "remained relatively constant"; firms *expecting* alliance revenue growth had a **higher** failure rate; failures no more likely than internal R&D once maturity is controlled for; **highest failure rates in platform and ecosystem partnerships** | ✅ Verified as published; ⚠ **the guide repeats this with its caveats in place**: the headline measures **ambition shortfall**, not dissolution or commercial failure; consultancy publisher; network sample | Same BDO publication, 2024 | Industry/consultancy survey. **This is the only partnership-failure figure in this guide**, deliberately |
| 37 | **Top three contributors to alliance underperformance:** (1) lack of internal alignment within at least one partner; (2) failure to understand differences in goals and priorities between partners; (3) lack of sufficiently robust joint governance. **Lowest:** "Selected the wrong partner" | ✅ Verified as published; ⚠ same survey caveats; and the guide notes the ambiguity that item 19 and §8.2(3) discuss | Same BDO publication, 2024 | Industry/consultancy survey |
| 38 | ASAP's homepage figures: **1/3 of company revenue driven by alliances**; **$100T global economic output from ecosystem-driven businesses by 2030**; **70%+ revenue growth projected over the next five years**; **94% of organizations believe their partner ecosystem will drive future growth** | ✅ Verified as published on that page; ⚠ **no source, sample or methodology is shown on the page**, and the publisher's purpose is to advance the practice it describes | ASAP own site; retrieved September 2026 | **Marketing document — market figures. Reported as claims; used as none** |
| 39 | **AWS co-sell outcome figures:** **80%** of partners identify AWS Marketplace as integral to co-sell strategy; **51%** report higher average revenue growth from co-sell; **65%** close deals faster | ✅ Verified as published on AWS's pages; ⚠ **methodology not on the page**; the figures are attributed to a **Canalys** report reprint hosted by AWS (the file path carries "24-", suggesting **2024**, unverified) | AWS pages, citing Canalys; retrieved September 2026 | **Vendor content citing a commissioned analyst survey** |
| 40 | **Vendor-published customer quotes** as evidence of co-sell outcomes (e.g. named partner executives reporting percentage increases in customer wins, marketplace contract value and co-sell engagement) | ✅ Verified as published; ⚠ **self-reported by the vendor's own partners, published by the vendor**; not evidence of a general effect | AWS ISV Accelerate page; retrieved September 2026 | Vendor content |
| 41 | **The partner-sourced vs partner-influenced distinction, and its documented distortions** — deals sourced by a partner vs merely touched by one; programmes **blend the two into one attributed total**; **minor touches counted as meaningful influence**; **the same deal appearing in more than one total** | ⚠ **Flagged — industry usage documented by interested parties.** Multiple vendor/partner-programme content pages assert this (retrieved September 2026), each published by a firm selling attribution or partner-management software | Multiple vendor content pages | **Marketing documents. The guide treats the *existence* of the distinction as established by usage and the *magnitude* of the problem as unverified** |
| 42 | **Attribution in a co-sell programme is procedural** — the opportunity is either registered in the programme's pipeline or it is not, and programme thresholds decide what counts | ✅ Supported | AWS ACE/ISV Accelerate rules; retrieved September 2026 | Tier-1 primary |
| 43 | **McKinsey's position that alliance measurement must go beyond cash-flow metrics** to include transfer-pricing benefits, benefits outside the deal's scope, the value of options created, and start-up and ongoing management costs | ⚠ **Flagged** — read from the **search-result description only**; **the McKinsey page failed to extract**, so its **date was not established** and its full argument was not read | McKinsey, "Measuring alliance performance"; **date not established in this pass** | Consultancy insight page (unread) |
| 44 | **"Measurement in this field is contested and practice varies"** | ✅ **Supported as the conclusion of items 35–43** — a professional body that teaches value measurement without standardising it; a consultancy measuring revenue shares by network survey; a vendor measuring contribution procedurally; an attribution vocabulary owned by software sellers | — | Guide's conclusion from verified items |
| 45 | **Four questions to ask before accepting anyone's BD metric** | 🔧 **Constructed** — questions, not a metric, and the distinction is deliberate (§7.5) | — | Guide's construction |
| 46 | **The guide proposes no metric and prints no partnership failure rate of its own** | ✅ Compliance statement — §7.5. The single failure figure used (item 36) carries its publisher, date, method statement and caveats | — | — |

### 15.5 The AI-era claims

| # | Claim | Status | Publisher and date | Quality |
|---|---|---|---|---|
| 47 | **ASAP has deployed an AI assistant for alliance professionals ("Ally")** | ✅ Verified as published | ASAP own site; retrieved September 2026 | Professional body |
| 48 | **ASAP runs member programming on AI's effect on inventorship and contract terms** | ✅ Verified as published | ASAP events listing; retrieved September 2026 | Professional body |
| 49 | Vendor documents on AI in partner ecosystems exist and are dated: **Impartner "AI Partner Playbook" (August 2025)**; a later Impartner ecosystem playbook (**September 2026**); an **AI channel-partner ecosystem analysis (October 2025)**; multiple vendor blog and contributor pieces (**2025–2026**) | ✅ Verified as documents, with those dates | Vendor PDFs and pages; retrieved September 2026 | **Marketing documents** |
| 50 | **Vendor claim:** AI/automation "streamline operations from recruitment to performance analysis" and let channel managers "focus on strategy and relationship building rather than administrative tasks"; AI transforms partner programmes "from onboarding and enablement to co-marketing, sales support, and ecosystem strategy" | ⚠ **Flagged — reported as the vendor's claim, not as evidence.** No methodology, sample or measured outcome is presented; every publisher sells the software category being claimed about | Vendor playbooks/blogs, 2025–2026; retrieved September 2026 | **Marketing documents** |
| 51 | **Claim that partner ecosystems have become "a fundamental requirement for market leadership" because of AI** | ⚠ Flagged; guide does not adjudicate | Channel-focused publisher, document dated October 2025 | Marketing/analysis hybrid |
| 52 | **The guide's routing of automation by task** (partner research/scanning and drafting most automatable; discovery, negotiation, internal advocacy least) | 🔧 **Constructed reasoning, with its method stated** ("which parts have a text artefact and an existing dataset") | — | Guide's construction |
| 53 | **Any headcount effect of AI on BD roles** | ⛔ **No source located; the guide asserts none in either direction** | — | **Recorded as absent, per requirement** |

### 15.6 Compensation and wages — recorded as absent

| # | Claim | Status | Publisher and date | Quality |
|---|---|---|---|---|
| 54 | **Singapore's Ministry of Manpower publishes an Occupational Wage Survey**, whose **2025 tables were released 30 June 2026**, covering **over 500 occupations**, for **full-time resident employees**, as **basic and gross monthly wages excluding bonuses**, compiled from a survey of **private-sector establishments with at least 25 employees** supplemented by administrative records, with data-collection processes **assessed by Ernst & Young Advisory Pte Ltd**; a companion spreadsheet lists the occupations and industries for which wage data are published | ✅ Verified as published | **Singapore MOM**, Labour Market Statistics and Publications, Occupational Wages page; retrieved September 2026 | **Tier-1 primary (national statistics office)** |
| 55 | **Whether "business development" appears in that occupation list, and under what classification** | ⛔ **Not verified** — the guide did **not** download or open the occupation-list spreadsheet this pass. **Consequence: no wage figure is printed.** See §16 | — | **Research gap** |
| 56 | **Any business-development compensation figure** | ⛔ **Not printed anywhere in this guide.** The only sources located publishing BD pay figures are **recruitment/salary-survey pages, i.e. marketing documents**, with self-selected samples and unexplained treatment of geography, seniority, base-versus-total and date | — | **Absent by design — §9.4** |
| 57 | The reasoning that a single title-averaged BD salary would be meaningless because the title covers five materially different roles | 🔧 Constructed, resting on item 13 ✅ | — | Guide's construction |

### 15.7 Rejected or not verified — consolidated

| # | Item | Status |
|---|---|---|
| 58 | Any **Asia-specific or Singapore-specific BD/alliance profession research** | ⛔ **Not located** |
| 59 | Any **Asian professional body, certification or competency standard for business development** distinct from the alliance body | ⛔ **Not located** |
| 60 | **A task-level competency framework for "business development" as such** | ⛔ **Not located** — the reference work lists skill areas, ASAP certifies alliance managers; neither publishes an assessment framework for BD |
| 61 | **How BD functions are measured** (as distinct from alliance functions or partner programmes) | ⛔ **Not established** — §7.2 |
| 62 | **A typical internal-versus-external time split for BD work** | ⛔ **Not established; no proportion printed** |
| 63 | **Typical partner-cycle duration** | ⛔ **Not established.** The guide's 6–18 month characterisation is labelled as its own and should not be quoted as a statistic |
| 64 | **BD job-volume, hiring-trend or market-size figures** | ⛔ **Not established** — the searches returned vendor content and no primary source; `bls.gov` failed to load |
| 65 | **Salesforce's partner programme** (a named programme of a major firm) | ⛔ **Not read** — `salesforce.com/partners` and `/partners/overview/` both failed to extract. **Absence from this guide is a tool limitation, not a claim about the programme.** The guide instead used AWS and Microsoft, whose programme pages did load |
| 66 | **Microsoft's public partner marketing front door** (`partner.microsoft.com`) | ⛔ **Not read** — extraction failed. Microsoft's **documentation** pages (Microsoft Learn) did load and are the source of items 24–25 |
| 67 | **SAMA (Strategic Account Management Association)** | ⛔ **Not read** — `strategicaccounts.org` failed to extract. This may matter, since strategic-account management is adjacent §2.1 variant D territory; recorded as a gap |
| 68 | **The McKinsey alliance-measurement page** | ⛔ **Not read** — extraction failed; cited only at search-description grade with its **date unknown** (item 43) |
| 69 | **The Bloomberg Intelligence report itself** | ⛔ **Not read** — the figures in §10.1 are as reported by secondary outlets on a named date |
| 70 | **US BLS occupational classification of business development** | ⛔ **Not verified** — `bls.gov` pages failed to load (item 15) |

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What could not be verified, and why

**Stated as the guide's honest inventory, because an absence is a finding.**

1. **Tool limitations, which are not evidence of absence.** The **web search tool returned empty result sets for several queries** attempted during this pass, and the **page-extraction tool failed on a number of primary pages**, including `salesforce.com/partners`, `partner.microsoft.com` (the marketing front door), `bls.gov` (both the Occupational Outlook Handbook entry for sales managers and the OES occupation table), `strategicaccounts.org` (SAMA), `mckinsey.com` (the alliance-performance page) and the Airwallex careers posting. **Every one of these is recorded as a tool limitation.** Where a claim rests on such a page, the guide either declined it (§15.7) or graded it down to ⚠ and said so in place (items 7, 43).
2. **The compensation absence.** No BD pay figure is printed. §9.4 gives the three-part reason: the only pay publishers located are recruitment/salary marketing documents; the one defensible instrument located (**Singapore MOM's Occupational Wage Survey 2025**, methodology stated ✅) was **not opened to the occupation level**; and a title that covers five different jobs (item 13 ✅) cannot be averaged into a meaningful figure.
3. **The measurement absence.** §7.2 could establish what a *professional body*, a *consultancy* and a *vendor programme* each do about partner measurement — it could **not** establish how business-development functions are typically measured. §7 therefore concludes that measurement is contested and practice varies ✅, which is what the sources support.
4. **The market absence.** No market-size, headcount or hiring figure for BD was located or printable — see §15.7 item 64. §9.3 gives the market context that *is* sourced (BDO's survey) and labels it ⚠.
5. **The regional absence.** Nothing Asia-specific about the BD profession was located (§10.3). The regional *commercial* context is documented (Bloomberg Intelligence figures as reported, February 2024 ⚠); the *profession* is not.
6. **The framework absence.** No task-level competency framework for BD exists in anything retrieved (item 60). Both published skill sources are lists, not models.
7. **Two definitional sources are second-hand in this guide, and this is said plainly.** The Palgrave/Sørensen definition of business development (item 2) was read **through** the reference work rather than at the encyclopedia entry; and the Airwallex corporate-development responsibilities (item 7) were read **from a search-result description** because the page would not load. Each is graded accordingly.
8. **The 6–18 month partner cycle and the "majority of time is internal" claim are the guide's characterisations, not sourced findings.** Both are marked 🔧 at every appearance and neither should be quoted as a statistic. They are recorded here again because they are the two places a careless reader is most likely to lift a number.

### 16.2 Glossary

🔧 *Definitions are the guide's, written for a reader of §1.3's decoder; where a definition rests on a source it is marked in §1.3 and in §15.*

| Term | Definition |
|---|---|
| **Alliance** | A cooperative agreement between independent organisations pursuing agreed objectives while remaining independent; usually short of a legal partnership. Whether a joint venture counts is genuinely disputed in the literature. |
| **Attribution** | The rule that decides which party's numbers a deal appears in. In partner work, contested by construction, and in co-sell programmes it is procedural (first registrant wins). |
| **Business development (BD)** | In this guide, the function whose work is the analytical preparation of growth opportunities and the support of their implementation — and, honestly, a title covering at least five different jobs. |
| **Channel** | The route by which a product reaches a customer when it is not sold direct. A path, not a relationship. |
| **Coopetition** | Simultaneous cooperation and competition between the same organisations; the condition under which most technology partnerships are actually negotiated. |
| **Co-sell** | Two organisations pursuing the same opportunity together, sharing pipeline visibility and often funding. The vendor sells. |
| **Corporate development** | The function that plans and executes strategy primarily through M&A and divestitures. Shares the word "development" with BD and little else. |
| **Ecosystem** | The organisations whose offerings interact with a platform or product. Most of it you have no agreement with. |
| **Ecosystem map** | The maintained artefact recording who is in the ecosystem and in what state. Its value decays with every month it is unopened. |
| **Enablement** | The material, training and support that makes a partner able to sell or support the joint offering. Its absence is the most common cause of a launch that produces nothing. |
| **Feasibility gate** | 🔧 The guide's name for the practice of requiring an engineer to record scope, exclusions and a rough timeline before any external description of technical delivery is made. |
| **Integration (as a motion)** | A partnership whose consideration is technical. The motion that consumes engineering time and most often lacks a built commercial case. |
| **Joint venture** | A separate, jointly-owned entity formed by two or more firms combining resources. |
| **Motion** | The commercial route by which a partnership produces revenue — referral, resell, co-sell, OEM, integration, joint venture, strategic investment. |
| **OEM** | A company producing parts or equipment marketed by another. Ambiguous by the reference work's own account; ask which meaning is intended. |
| **Partner** | A counterpart organisation in a durable commercial relationship; usually neither a customer nor a supplier, and frequently also a competitor. |
| **Partner-sourced / partner-influenced** | The industry's distinction between a deal a partner originated and one it merely touched. Usage, not standard; documented mostly by vendors of attribution software. |
| **Pipeline** | The flow of potential clients under development, each carrying an estimated probability and volume. Not a forecast. |
| **Qualified opportunity** | An opportunity meeting a threshold that whoever runs the programme set. Always ask for the definition before the count. |
| **Referral** | Passing a lead for a fee, margin or reciprocity, with no joint selling and no product change. |
| **Resell** | A partner transacts the product and sells it on, owning the customer relationship. |
| **Value inflection point (VIP)** | 🔧 *Term from the alliance profession's own book:* a moment when what happens next can significantly affect the value of the partnership — a disagreement, a major decision, a change in circumstances. The first dispute is the archetype. |
| **Venue** | 🔧 The guide's term of art: the set of conditions — routes, integrations, agreements, relationships — under which a market becomes reachable at all. BD builds it; sales works it. |

### 16.3 Cross-references — what each neighbouring guide owns, by name

| Guide | Owns | How this guide uses it |
|---|---|---|
| [meddicc_guide.md](meddicc_guide.md) | Sales qualification methodology (MEDDPICC), deal review, forecasting, failure modes in the deal cycle | **Pointer only.** §4.4 draws the partner-versus-opportunity qualification distinction; it never re-derives the methodology |
| [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) | The sales-methodology landscape (note: this file lives in `technology/`) | **Pointer only** — §4.4 |
| [post_sales_customer_facing_experience_guide.md](post_sales_customer_facing_experience_guide.md) | The customer-facing experience after the sale: lifecycle, commercial mechanics, retention arithmetic, metrics critique, enablement efficacy | **Named cross-reference** — §6.2 stage 5, §12.3 friction 4 and §13.6 hand off the post-launch operating discipline to it |
| [consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) | The structural template of this series, and advisory-versus-delivery (§5 there) | **Template, plus a named cross-reference** — §9.2 and §12.4 state how BD differs from advisory work (a partnership has no end date) and do not repeat §5 |
| [business_case_development_guide.md](business_case_development_guide.md) | Business-case construction | **Pointer only** — §5 and §13 note that a partnership needs a case and do not build one |
| [ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) | Revenue-line design — packaging, pricing, monetisation | **Pointer only** — §5 treats the motion, not the product design |
| [vendor_management_guide.md](vendor_management_guide.md) | The buy side: selecting, contracting with, governing and exiting a supplier | **Named cross-reference** — §6.2 stage 7 and §12.3 friction 4 |
| [reverse_job_search_guide.md](reverse_job_search_guide.md) | Discoverability and search mechanics | **Named cross-reference only** — §9 |
| [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) | The customer-embedded technical role (FDE) | **Named cross-reference** — §1.2 |

### 16.4 Closing summary

**Seven things this guide can say with evidence behind them:**

1. **"Business development" names a family of roles rather than one profession** ✅ — and the evidence is employers' and professional bodies' own definitions, not career advice: a nonprofit alliance profession with a handbook, certifications and a seven-phase lifecycle (ASAP, begun **1998**) ✅; a corporate-development function defined around M&A and divestitures ✅; a cloud vendor's published ontology of partner paths ✅; and an employer calling a plainly sales role "Business Development Manager, New Business Sales" ✅. **The reference work's own words are the finding: the responsibilities vary so much that defining the function's unique characteristics has become difficult** ✅.
2. **Business development is not sales, and the boundary genuinely blurs.** The distinction is structural — cycle length, unit of work, cadence, internal-selling load, what the number can measure — and it is developed as analysis with reasons (§4.1), not asserted. The blurring is admitted in five named places (§4.2), one of which is documented at an employer's own site ✅.
3. **Most of the job is internal.** The practitioner's productivity is bounded by internal commitments rather than external meetings (§3.1.5, §12.1.4). **No proportion is printed**, because none is sourced — but the direction is corroborated twice: the professional body runs programming on internal cohesion ✅, and the consultancy survey ranks internal alignment as the **top** contributor to underperformance ✅.
4. **The measurement problem is unresolved, and the guide refuses to solve it with an invention.** A quota needs four properties BD lacks (§7.1); what BD functions are measured on is not established by any source located (§7.2); attribution is contested and procedurally decided in co-sell programmes (§7.3); the counterfactual is unavailable (§7.4); and the guide proposes four *questions* instead of a metric (§7.6) — explicitly so a reader cannot mistake the questions for a measurement.
5. **The lifecycle has published phases and the deaths happen in three predictable places** (§6.3): the partner that signs and never sells; the launch with no enablement; the relationship with no owner. The professional literature treats *launching, managing and exiting* as phases ✅ for exactly this reason, and one survey ranks weak joint governance among the top three causes of underperformance ✅.
6. **The technical counterpart's interests are specific, mutual, and testable** (§12): a gate before promises, a pilot that names its owner, engineering time reported beside revenue, and a fast "no" owed in both directions. **All four frictions reduce to one mechanism — the person who makes the promise is rarely the person who pays for it** 🔧.
7. **The guide asserts no AI headcount effect, and no compensation figure, and says why.** Partner research, ecosystem mapping and drafting are the plausibly automatable parts; relationship-building and internal advocacy are not (§11.3) — and the vendor claims are labelled as marketing documents (§11.2). **No market-size, hiring or salary number appears anywhere in this guide**, because the sources that publish them are recruitment marketing or interested surveys (item 38, 39, 56).

**What the reader is left with, in one sentence each.** The word is unstable, so read the mandate and the measure, not the title. The work is mostly internal, so your credibility is the instrument. The lifecycle has phases, so insist on the ones that arrive after signature. The measurement is contested, so refuse the number that cannot state its denominator. And the discipline is real, because someone has to build the conditions in which selling becomes possible at all:

**sales works the venue; business development builds it.**
